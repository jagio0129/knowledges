MySQLバグレポート
===

`INSERT INTO ... ON DUPLICATE KEY UPDATE ... `は以降IODKUと表記

## tl;dr

- 5.7.26未満ではIODKUでデッドロック起きないが、レプリケーションで不具合が発生する可能性があった
- 問題を解消するために以下のコミットで以前取り込まれたbugifxをrevertした
  - https://github.com/mysql/mysql-server/commit/066b6fdd433aa6673622341f1a2f0a3a20018043
- 5.7.26以降の挙動はバグでなく正当な仕様である

## 概要

https://bugs.mysql.com/bug.php?id=98324

この投稿では、ユーザが5.7.26にあげたことでデッドロックが増えたといっている。その再現コードも[deadlock.sh](https://bugs.mysql.com/file.php?id=29341&bug_id=98324)で展開されている。

結論として、5.7.26以降のこの動作は正常であると述べている。

### レポートの動向

Thanh Nguyenが、2セッションのIODKUが5.7.26以上でデッドロックを引き起こすと報告。([再現コード](https://bugs.mysql.com/file.php?id=29341&bug_id=98324))

同氏の分析によると、

5.7.26でIODKUに以下の変更が入った。

> Two sessions concurrently executing an INSERT ... ON DUPLICATE KEY UPDATE operation generated a deadlock. During partial rollback of a tuple, another session could update it. The fix for this bug reverts fixes for Bug #11758237, Bug #17604730, and Bug #20040791. (Bug #25966845)
>
> 2つのセッションが同時にINSERT ... ON DUPLICATE KEY UPDATE操作を実行すると、デッドロックが発生しました。タプルの部分的なロールバック中に、別のセッションがタプルを更新してしまうという問題がありました。このバグの修正は、Bug #11758237、Bug #17604730、Bug #20040791の修正をrevertしています。(バグ#25966845)

詳細な変更を見ると

- https://github.com/mysql/mysql-server/commit/066b6fdd433aa6673622341f1a2f0a3a20018043
  - このコミットでリリースノートに記載されているbug fixコミットをrevertした
  - 「重複キー更新時に "on duplicate key update "句のためにタプルの部分的なロールバックを行った場合、保存性を維持していなかったため、行で待機している別の接続が更新されてしまい、誤った結果を引き起こす可能性がありました。」
- https://github.com/mysql/mysql-server/commit/c93b0d9a972cb6f98fd445f2b69d924350f9128a
  - ↑でrevertされたbugfixの一つ
  - 「INSERT ... ON DUPLICATE UPDATE ステートメントが実行されると、まず、レコードはクラスタ化されたインデックスに挿入され、続いてセカンダリインデックスが挿入されます。重複エラーが発生しても処理を停止しません。 すべてのユニークなセカンダリインデックスの処理を継続し、必要なギャップロックを配置します。重複エラーが発生すると、レコードは実際には挿入されず、ギャップロックのみが配置されます。」

要約によると、

- 5.7.4 (2014)では、レプリケーションの問題を解決するために長いロックを適用しました。
- 5.7.26 (2019)では、レプリケーションのためのより良いアプローチが見つかったので、余分なロックは削除されました。

ここから以下の質問を投げている

> Why 5.7.4 implementation has more aggressive locking but the deadlock happens more frequently when the change get reverted?
Before 5.7.4, the lock is extended for a longer period => the two batch transactions are fully isolated. In 5.7.26, the lock periods get shorter => 2 batch transactions are intervened and cause deadlock
>
> 5.7.4の実装ではロックがより積極的になっていますが、変更を元に戻すとデッドロックが頻繁に発生するのはなぜですか？5.7.4以前の実装では、ロック期間が長くなりました。5.7.26では、ロック期間が短くなりました。
>

[5.7.26のドキュメント](https://dev.mysql.com/doc/refman/5.7/en/innodb-locks-set.html)の

> "INSERT ... ON DUPLICATE KEY UPDATE differs from a simple INSERT in that an exclusive lock rather than a shared lock is placed on the row to be updated when a duplicate-key error occurs. An exclusive index-record lock is taken for a duplicate primary key value. An exclusive next-key lock is taken for a duplicate unique key value.

がアップデートされていないのでは？と発言。

## Jakub Lopuszanski の返答

(MySQLコントリビュータ、オラクル社員ぽい)

↓なぜ#25966845でrevertすることになったのか

> In particular the Bug #25966845 INSERT ON DUPLICATE KEY GENERATE A DEADLOCK has a title complaining about a deadlock, but deadlocks are a fact of life and not a bug per se.
However, when investigating what exactly went wrong, we've discovered that implementation does not provide enough locks which was much more serious problem!
>
> 特に Bug #25966845 INSERT ON DUPLICATE KEY GENERATE A DEADLOCK にはデッドロックについての不満が書かれていますが、デッドロックは日常茶飯事であり、それ自体がバグではありません。しかしながら、このfixの何が間違っていたのかを調査したところ、実装が十分なロックを提供していないことが判明しました。

> It could happen that knowing what queries transactions performed and in which order they committed was no longer enough to determine the final state of the DB,
which means among other things that replication could fail to produce exact copy of DB.
Therefore the fix Bug #25966845 is actually about bringing back serializability guarantees by adding more locks.
And more locks can (as expected) cause more deadlocks in some scenarios.
>
> トランザクションがどのクエリを実行し、どの順番でコミットしたかを知るだけでは、DBの最終的な状態を判断するのに十分ではなくなってしまう可能性がありました。
これは、レプリケーションがDBの正確なコピーの生成に失敗する可能性があることを意味します。
そのため、Bug #25966845 の修正は、実際にはより多くのロックを追加することでシリアライズ性の保証を復活させることを目的としています。
また、ロックを増やすと（予想通り）いくつかのシナリオでデッドロックが発生する可能性があります。

以降は、**revert前の並列IODKUが非決定論的(つまり実行ごとに結果が異なる)だったことが判明**した説明。

```sql
create table t1(
  f1 int auto_increment primary key,
  f2 int unique key,
  f3 int
);

-- and a single row in it: (1, 10, 100)

-- And that there are two parallel connections:

-- con1:
begin;
insert into t1 values(2, 10, 200) on duplicate key update f3 = 120;
commit;

-- con2:
begin;
insert into t1 values(2, 20, 300) on duplicate key update f3 = 500;
commit;
```

上の場合で、con1がcon2が開始される前に挿入しようとした場合、f2=10で競合しUPDATEが発生。その後con2は競合せずINSERTされるので、DBの状態は次のようになる。

(1,10,120)
(2,20,300)


しかし、レプリケーションの最中に、con2がcon1よりも先にコミットしたことを知っていた場合、逆の順序で実行することになります。

最初の con2 の挿入ではコンフリクトは発生せず、(2,20,300) を挿入します。


そして、con1は競合を見ますが、今回はf1=2で、(↑で先に見たf2=10の場合とは対照的に)競合している行は(2,20,300)であると判断し、(2,20,120)に更新するので、レプリカの終了状態は(2,20,120)になります。

(1,10,100)
(2,20,120)

直列化可能性(レプリケーションでも整合性が担保されるbinlog順と推定)を提供するためには、「決定を行うために見たすべてのものをロックする」必要があることが明らかになりました。

con1 が、最初に f1=2 の行がないことを観察することで、競合が f2 上にあることを決定したことを観察してください（f1 とは対照的に）。PRIMARY INDEXにf1=2のレコードを一時的に作成して、それが成功したことを確認しました。後になって初めて、f2で競合が見つかり、レコードを削除することにしました。

レコードを削除することで、con1はf1での競合を見なかったという「証拠」の一部を削除しました。そのため、con2はf1=2の行を挿入することができ、後にレプリケーションの問題が発生しました。

「証拠を保存する」ための正しい方法は、con1がコミットするまでf1=2が挿入されたギャップがロックされたままであることを確認することです。これは、一時的な行に明示的なロックを作成し、削除時にそれを継承させることで実現できます。このような明示的ロックは、いわゆる暗黙から明示への変換によって作成することができます。

con1はすでにレコードに対して暗黙のロックを持っていました。しかし、この暗黙のロックはレコードを物理的に削除するとすぐに消えてしまうので、明示的なロックとしてメモリに保存する必要があります。

## Jakub Lopuszanskiの実験

MySQL8系のほうが詳しいらしく8系(8.0.21-tr)で行っている。deadlock.shを用いる。

```shell=
# deadlock.sh

create_table='
 CREATE DATABASE IF NOT EXISTS dbtest;
 USE dbtest;
 DROP TABLE IF EXISTS ttable;

 CREATE TABLE `ttable` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `foo` int(11) NOT NULL,

  PRIMARY KEY (id),
  UNIQUE KEY `index_unique` (`foo`)
)
'

transaction1='INSERT INTO ttable (`foo`) VALUES (699422), (699421) ON DUPLICATE KEY UPDATE foo = VALUES (foo)'
transaction2='INSERT INTO ttable (`foo`) VALUES (699439), (699439) ON DUPLICATE KEY UPDATE foo = VALUES (foo)'

mysql -u root -e "$create_table"

for i in {1..100}; do
       mysql -u root dbtest -e "$transaction1" &
       mysql -u root dbtest -e "$transaction2"
done
```

実行したテーブルは以下のようになる。

```sql
mysql> select * from dbtest.ttable;
+----+--------+
| id | foo    |
+----+--------+
|  4 | 699421 |
|  3 | 699422 |
|  1 | 699439 |
+----+--------+
3 rows in set (0.00 sec)
```

トランザクション2がfoo:699439を2回挿入しているため、id=2の行が欠落しており、それによってautoincが2回インクリメントされています。

deadlock.sh のループの最初の繰り返しの中の最初の二つのクエリのタイミングによっては、例えば全く異なる値を見ることになるかもしれません。

```sql
mysql> select * from dbtest.ttable;
+----+--------+
| id | foo    |
+----+--------+
|  3 | 699421 |
|  2 | 699422 |
|  1 | 699439 |
+----+--------+
3 rows in set (0.00 sec)
```

8系では `innodb_autoinc_lock_mode=2`がデフォルトである。

以降はテーブルの状態が後者としてデッドロックがどのように起きるのか見る。

`lock_wait_check_candidate_cycle()` (デッドロックの可能性が高いサイクルが見つかったとき、ロックシステムの活動を妨げるラッチ(デッドロックの原因)を取得する直前)にブレークポイントを置いて、このスレッドをフリーズし、他のすべてのスレッドの一時停止を解除し、別のクライアントから `performance_schema.data_locks` をチェックすると、以下のようになります。

```sql
mysql> SELECT ENGINE_TRANSACTION_ID,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA FROM performance_schema.data_locks;
+-----------------------+-------------+--------------+-----------+--------------------+-------------+------------------------+
| ENGINE_TRANSACTION_ID | OBJECT_NAME | INDEX_NAME   | LOCK_TYPE | LOCK_MODE          | LOCK_STATUS | LOCK_DATA              |
+-----------------------+-------------+--------------+-----------+--------------------+-------------+------------------------+
|                 12298 | ttable      | NULL         | TABLE     | IX                 | GRANTED     | NULL                   |
|                 12298 | ttable      | index_unique | RECORD    | X                  | GRANTED     | 699439, 1              |
|                 12298 | ttable      | PRIMARY      | RECORD    | X,REC_NOT_GAP      | GRANTED     | 1                      |
|                 12298 | ttable      | PRIMARY      | RECORD    | X                  | GRANTED     | supremum pseudo-record |
|                 12298 | ttable      | PRIMARY      | RECORD    | X,INSERT_INTENTION | WAITING     | supremum pseudo-record |
|                 12297 | ttable      | NULL         | TABLE     | IX                 | GRANTED     | NULL                   |
|                 12297 | ttable      | index_unique | RECORD    | X                  | GRANTED     | 699422, 2              |
|                 12297 | ttable      | PRIMARY      | RECORD    | X,REC_NOT_GAP      | GRANTED     | 2                      |
|                 12297 | ttable      | PRIMARY      | RECORD    | X                  | GRANTED     | supremum pseudo-record |
|                 12297 | ttable      | PRIMARY      | RECORD    | X,INSERT_INTENTION | WAITING     | supremum pseudo-record |
+-----------------------+-------------+--------------+-----------+--------------------+-------------+------------------------+
10 rows in set (0.00 sec)
```

| header                | value               | 説明                                                                                                                                                                                                          |
| --------------------- |:------------------- |:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ENGINE_TRANSACTION_ID |                     | 読み取り専用ではないことがわかっているトランザクションの場合、これはロックを要求した trx の ID です。(まだ読み取り専用の可能性があるトランザクションの場合は、内部的に割り当てられた一時的な値が表示されます) |
| INDEX_NAME            | NULL                | テーブル全体のロック                                                                                                                                                                                          |
|                       | index_unique        | ユニークインデックス                                                                                                                                                                                          |
|                       | PRIMARY             | クラスタインデックス                                                                                                                                                                                          |
| LOCK_MODE             | X,REC_NOT_GAP       | レコードロック                                                                                                                                                                                                |
|                       | X                   | ネクストキーロック                                                                                                                                                                                            |
|                       | X,INSERT_INTENTION  | 挿入したいギャップの競合するロックが解放されるまで待つトランザクションで待ち受けるロック                                                                                                                      |
| LOCK_STATUS           | GRANTED             | トランザクションはロックを所有しています。                                                                                                                                                                    |
|                       | WAITING             | トランザクションは競合するロックが解放されるまで待機しています。                                                                                                                                              |
| LOCK_DATA             | for secondary index | インデックス定義で明示的に指定されたカラムの値に続いて、残りの主キーカラムの値を指定します。                                                                                                                  |
|                       | for primary index   | 主キーの全列                                                                                                                                                                                                  |
|                       | NULL                | 明らかにテーブルロックのために、もしくはバッファプールページが他のスレッドで使用されていたためにカラムの値を読むことができなかった場合はperformance_schema                                                 |

複数のトランザクションが supremum pseudo-record(テーブルの最大レコードを超えたレコード帯(疑似的なもの))に X ロックを持つことができます。なぜなら

1. supremum pseudo-recordは常にギャップロックのみである
    - このロックは実際にある最後のレコード以降のレコードをプロテクトします。
2. ギャップへのINSERTを防止することを目的としているため、1つ以上のトランザクションが任意のギャップにGRANTEDロックを持つことができます。
    - そのため複数のトランザクションが同じことが起こらないようにしたい場合は、競合はありません。

どちらのトランザクションも複数行のINSERTを実行します。

ISERNT INTO文の2行目(699439)をINSERTしようとするときこのトランザクションはページの最後にある隙間にそれを入れようとしますが、これは前述のsupremum pseudo-recordのXロックによって保護されています。。そのためロック待ちが発生する。これは、両方のトランザクションが両方ともガードしているギャップにX,INSERT_INTENTIONを必要とするため、デッドロックにつながります。

つまり、問題は(あるとしても)最初に挿入された行の処理にあるのではなく、むしろ2番目の行が挿入されないようになっている(ロックによって)ことにあると言えます。
実際に deadlock.sh を修正して各接続から一つの行だけを挿入してもデッドロックは発生しません。

### PRIMARY インデックスの supremum pseudo-recordの X ロック

PRIMARYインデックスのsupremum pseudo-recordのXロックの挙動は以下の通りです。テーブルは以下の状態とします。

```sql
mysql> select * from dbtest.ttable;
+----+--------+
| id | foo    |
+----+--------+
|  3 | 699421 |
|  2 | 699422 |
|  1 | 699439 |
+----+--------+
3 rows in set (0.00 sec)
```

このテーブルに対して以下の処理を行う

```sql
INSERT INTO ttable (`foo`) VALUES (699422) ON DUPLICATE KEY UPDATE foo = VALUES (foo)
```

ロックに関連した出来事の時系列を追う。

#### 1. テーブルにインテンションロックを作成するために`lock_table_create(...,LOCK_IX,...)`が呼び出される。
- これは任意のトランザクションがその行を変更していることを、テーブル全体をロックしようとしている他のスレッドに教えるためにある

#### 2. `lock_rec_insert_check_and_lock(...,rec,..,{name:"PRIMARY"})`は`PAGE_HEAP_NO_SUPREMUM` の直前に `rec` がある場合に呼び出されます。
- この時点ではロックを保持している他のトランザクションが存在しないため、supremum pseudo-recordの前のギャップにINSERTするためこのリクエストは、高速に付与されています。
  - これは、明示的なX,INSERT_INTENTIONロックを作成する必要はありません - スレッドは単に行を挿入するために移動します。
  - この操作は他のスレッドからの干渉を防ぐために行われ、その後は行内に格納されているTRX_ID(スレッドID)に依存しています。

#### 3. `lock_sec_rec_read_check_and_lock(lock_duration_t::AT_LEAST_STATEMENT,...,rec,{name:"index_unique"},...,LOCK_X,LOCK_ORDINARY)`は`row_ins_scan_sec_index_for_duplicate()`の実行中に呼ばれる
- `row_ins_scan_sec_index_for_duplicate()`はセカンダリインデックスをスキャンして重複がないかどうかを確認する関数です。
- `LOCK_ORDINARY`はネクストキーロックを取りたいことを意味する
-  `rec`はheap_no==3で、`(rec[1]*256+rec[2])*256+rec[3]`は699422、`rec[11]`は2である。
-  これがセカンダリインデックスの <foo:699422,id:2> エントリと一致していることに注目してください。
    1. これは `lock_protect_locks_till_statement_end()` を呼び出して、低い分離レベル(READ COMMITTED と READ UNCOMMITTED)でも、このステートメントの間、作成されたロックを確実に保持するようにしています。
        - これらの分離レベルは通常ギャップロックを全く使用していないにもかかわらず、レコードの物理的な削除時に、次のレコードまでのギャップロックを継承する。
    2. `lock_rec_lock(..,SELECT_ORDINARY,LOCK_X,...,heap_no=3,{name: "index_unique"},...)` の呼びだしに成功し、以下に相当するNext-keyロックを作成します。

```shell=
|                 12297 | ttable      | index_unique | RECORD    | X                  | GRANTED     | 699422, 2              |
```

#### 4. `lock_rec_convert_active_impl_to_expl(...,rec,..,{name:"PRIMARY"},heap_no=5)` は`trx_rollback_to_savepoint()`内で`row_mysql_handle_errors()`がDB_DUPLICATE_KEY errorを出したときに呼ばれる
- `rec[6]*256+rec[7]`は590です。これは一時的な行にオートインクリメントで割り当てられたidです。
- heap_no:5は一般的に、ヒープ割り当てやディール割り当ての履歴を知らずに解釈すると、複雑な方法で再利用されてしまうので注意が必要。
-  しかし、「単純な」履歴では、heap_no:0 は infimum、heap_no:1 は supremum、heap_no:2 は smalles record (今の場合は {id:1})、heap_no:3 は {id:2}、heap_no:4 は {id:3} となります。
-  したがって、heap_no:5は、ページ上でより大きな物理的レコードを扱うという我々の期待と一致しています。
    1. `lock_rec_add_to_queue(LOCK_REC | LOCK_X | LOCK_REC_NOT_GAP,...,heap_no=5,{name: "PRIMARY"})` を呼び出してレコードロックを作成します。 残念ながら、そのクエリは後で実行されたため、上記の performance_schema レポートでは見ることができません。
        -  しかし、この時点でこのスレッドをフリーズさせ、他のスレッドの一時停止を再開し、別の接続を使用して同様のクエリを実行することができます。

```shell
mysql> SELECT ENGINE_TRANSACTION_ID,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA FROM performance_schema.data_locks;
+-----------------------+-------------+--------------+-----------+---------------+-------------+-----------+
| ENGINE_TRANSACTION_ID | OBJECT_NAME | INDEX_NAME   | LOCK_TYPE | LOCK_MODE     | LOCK_STATUS | LOCK_DATA |
+-----------------------+-------------+--------------+-----------+---------------+-------------+-----------+
|                 12490 | ttable      | NULL         | TABLE     | IX            | GRANTED     | NULL      |
|                 12490 | ttable      | index_unique | RECORD    | X             | GRANTED     | 699422, 2 |
|                 12490 | ttable      | PRIMARY      | RECORD    | X,REC_NOT_GAP | GRANTED     | NULL      |
+-----------------------+-------------+--------------+-----------+---------------+-------------+-----------+
3 rows in set (0.00 sec)
```

- LOCK_DATA の NULL は performance_schema がレコードデータを取得できなかったことを意味します (私たちのスレッドはレコードのあるバッファプールページにラッチを持っているので、これは理解できます)。
- したがって、カラムの正確な値を報告することができません。

```shell=
mysql> SELECT ENGINE_LOCK_ID,INDEX_NAME,LOCK_MODE FROM performance_schema.data_locks;
+------------------------------------+--------------+---------------+
| ENGINE_LOCK_ID                     | INDEX_NAME   | LOCK_MODE     |
+------------------------------------+--------------+---------------+
| 2772975965448:1077:2772943755656   | NULL         | IX            |
| 2772975965448:20:5:3:2772943752744 | index_unique | X             |
| 2772975965448:20:4:5:2772943753096 | PRIMARY      | X,REC_NOT_GAP |
+------------------------------------+--------------+---------------+
3 rows in set (0.00 sec)
```

-  上記のように少しごまかすことで、これが実際にはヒープ番号 5 のロックであることがわかります。
    - ENGINE_LOCK_IDの読み方は`(...:space_id:page_id:heap_no:...)`です。
    - ENGINE_LOCK_ID==2772975965448:20:4:5:2772943753096はheap_no部が5です

#### 5. `lock_rec_inherit_to_gap(...,.,heir_heap_no=1,heap_no=5)` は、`lock_update_delete()` の一部として呼ばれ、行が物理的に削除されたときにロックを更新します (単に削除マークを付けるだけではなく)。
- 物理的に行を削除できるのは、ドキュメントにあるように `row_undo_ins()` の途中にあるからです。
  - 「テーブルへの行の新規挿入(fresh insert)を元に戻します。新規挿入とは、同じクラスタリングされたインデックスのユニークキーが、挿入時には削除マークが付いていても、レコードを持っていなかったことを意味します。 InnoDB はロールバックに熱心です: インデックスレコードがpurgeで削除されることがわかった場合、ロールバックで削除します。」
- これで、アクティブな読み込みビューではこの行を必要としないことがわかりました。
- 前述の通り、heap_no 1 は常にページ最上位のsupremum pseudo-recordを意味するので、削除される行 (heap_no=5) から最上位のsupremum pseudo-record (heir_heap_no=1) にロックを継承するように求められています。
-  これは、それが物理レコードで最大であったことを考えると、完全に理にかなっている。
-  先に `lock_protect_locks_till_statement_end()` が呼び出されていたことを考えると、分離度が低い場合でも、この操作により、
    - `lock_rec_add_to_queue(LOCK_REC | LOCK_GAP | LOCK_X,...,heap_no:1,{name:"PRIMARY"},...)`
    -  これは、Supremum上のロックは常にギャップロックであると仮定されているため、InnoDBで使用される慣例ではheap_no:1のLOCK_GAPフラグは無視されます。
    -  この時点で、ロックも状態は以下のようになっています。

```shell
mysql> SELECT ENGINE_TRANSACTION_ID,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA FROM performance_schema.data_locks;
+-----------------------+-------------+--------------+-----------+-----------+-------------+------------------------+
| ENGINE_TRANSACTION_ID | OBJECT_NAME | INDEX_NAME   | LOCK_TYPE | LOCK_MODE | LOCK_STATUS | LOCK_DATA              |
+-----------------------+-------------+--------------+-----------+-----------+-------------+------------------------+
|                 12490 | ttable      | NULL         | TABLE     | IX        | GRANTED     | NULL                   |
|                 12490 | ttable      | index_unique | RECORD    | X         | GRANTED     | 699422, 2              |
|                 12490 | ttable      | PRIMARY      | RECORD    | X         | GRANTED     | supremum pseudo-record |
+-----------------------+-------------+--------------+-----------+-----------+-------------+------------------------+
3 rows in set (0.00 sec)
```

- 2772975965448:20:4:5:2772943753096のロックは消え、次に新たな2772975965448:20:4:1:2772943753448のロックが生成される

#### 6. lock_sec_rec_read_check_and_lock(lock_duration_t::REGULAR,...,rec,{name: "index_unique"},...,LOCK_X,LOCK_REC_NOT_GAP)` が呼び出される。
- `rec`はheap_no==3であり、`(rec[1]*256+rec[2])*256+rec[3]`は699422であり、`rec[11]`は2です。
-  なぜまたここに来たのかと思うかもしれませんが、このロックはすでにステップ3でとられなかったのでしょうか？
    - それは別のロックモードで、ステップ3は一意性チェック中に行われたもの。修正して書き戻す前にセカンダリインデックスの値を読む
- つまり、現在は "UPDATE"の部分を処理しており、セカンダリインデックスを更新したい。
  - InnoDBでは最初にセカンダリインデックスのエントリを取得する必要がある。
- 実際には行はまったく変更されていないのに（新しい `foo` と古い `foo` は同じです）、これはちょっと無駄なことのように思えます。
- しかし、このコードは "foo=VALUES(foo) on conflict on foo" の場合にはあまり最適化されていないようです。
- また、この時点では古い競合する行の値を知らないので、何とかして知る必要がある: 古い競合する行を取得するには、PRIMARYインデックスを調べる必要があります。
-  しかし、検索すべきIDを知るためには、まずセカンダリインデックスを参照しなければなりません。
-  予想通り、この関数はすでにさらに強力なロックを持っていると結論づけ、何もしません。

#### 7. `lock_clust_rec_read_check_and_lock(lock_duration_t::REGULAR,...rec,{name: "PRIMARY"},...,LOCK_X,LOCK_REC_NOT_GAP...)` は、"書き込む前に行を読む必要がある"の一部として呼び出されます。
- 再び  `rec[6]*256+rec[7]` is 2

1. この場合、`lock_rec_convert_impl_to_expl()` のチェックを行う必要があります。 どうして前のステップで暗黙のロックをチェックしなかったのかの説明は以下です。
    1. `lock_table_create` テーブルのロックはレコードロックとは全く別のものです。
    2. `lock_rec_insert_check_and_lock` はレコード自体のロックを気にしません。
    3. `lock_sec_rec_read_check_and_lock` は、ページの最大trx idを読めば誰も触っていないことが分かる。
    4. `lock_rec_convert_active_impl_to_expl`は暗黙のロックを保持しているアクティブなtrxであると仮定しています (これが名前の由来です)。
    5. `lock_rec_inherit_to_gap`は興味深いです。私の記憶が正しければ、継承する前にimplicitからexplicitに変換すべきかどうかの議論がありました。(TODO: これは調査する必要がある)
    6. `lock_sec_rec_read_check_and_lock`は項目4と同じです。

    - このチェックは、行がアクティブなトランザクションによって変更されていないと結論づけているので、変換は必要ありません。
2. 続いて、`lock_rec_lock(..,SELECT_ORDINARY,...LOCK_REC_NOT_GAP|LOCK_X,heap_no=3,{name: "PRIMARY"})`を呼び出し、別のロックを作成することに成功しました

```shell=
mysql> SELECT ENGINE_TRANSACTION_ID,OBJECT_NAME,INDEX_NAME,LOCK_TYPE,LOCK_MODE,LOCK_STATUS,LOCK_DATA FROM performance_schema.data_locks;
+-----------------------+-------------+--------------+-----------+---------------+-------------+------------------------+
| ENGINE_TRANSACTION_ID | OBJECT_NAME | INDEX_NAME   | LOCK_TYPE | LOCK_MODE     | LOCK_STATUS | LOCK_DATA              |
+-----------------------+-------------+--------------+-----------+---------------+-------------+------------------------+
|                 12490 | ttable      | NULL         | TABLE     | IX            | GRANTED     | NULL                   |
|                 12490 | ttable      | index_unique | RECORD    | X             | GRANTED     | 699422, 2              |
|                 12490 | ttable      | PRIMARY      | RECORD    | X,REC_NOT_GAP | GRANTED     | 2                      |
|                 12490 | ttable      | PRIMARY      | RECORD    | X             | GRANTED     | supremum pseudo-record |
+-----------------------+-------------+--------------+-----------+---------------+-------------+------------------------+
4 rows in set (0.00 sec)
```

これでロックに関連した全ての手順が完了しました。

#### 5.7.26未満での挙動

まず第一に、deadlock.shはデッドロックを起こしません。

5.7.26以上との差分を見ていきます。

ステップ3では `lock_sec_rec_read_check_and_lock`はまだlock_duration_tをサポートしていないので、 `lock_protect_locks_till_statement_end`は呼ばれません。しかし、制約チェックのために作成されたロックがいずれにせよ継承されるようにする方法は他にもありました（もっとバグが多いけど）。

ステップ4はありません。`lock_rec_convert_active_impl_to_expl` はまだ実装されていないので、物理的に削除しようとしているレコードの暗黙のロックを明示的なロックに変換しないようにしています。

そこで、ステップ5に進みます。これは残念ながら `heap_no==5` に対して明示的なロックが行われていません。
(`lock_rec_convert_active_impl_to_expl`を呼んでいないため)なので、継承するものがないと判断します (ups!)


この時点では
```shell=
+-----------------------+-------------+--------------+-----------+-----------+-------------+-----------+
| ENGINE_TRANSACTION_ID | OBJECT_NAME | INDEX_NAME   | LOCK_TYPE | LOCK_MODE | LOCK_STATUS | LOCK_DATA |
+-----------------------+-------------+--------------+-----------+-----------+-------------+-----------+
|                  2346 | ttable      | NULL         | TABLE     | IX        | GRANTED     | NULL      |
|                  2346 | ttable      | index_unique | RECORD    | X         | GRANTED     | 699422    |
+-----------------------+-------------+--------------+-----------+-----------+-------------+-----------+
2 rows in set (7.46 sec)
```

ステップ6.は似ています(ただし、`duration`引数はありません)。

ステップ7.は全く同じです。

ご覧のように、動作の違いは明確にある。古いコードでは、一時的に挿入された行の暗黙のロックを明示的なロックに変換するのを忘れていた。したがって、ロックが継承されず、主キーに競合がなかったという証明することができません。

(オートインクリメントカラムの場合、そのような証明やギャップロックは必要ないと主張することができるかもしれませんが、InnoDBのような複雑なシステムでは、可能な限り一般的な方法で物事を行うことを好みます。
そのため、クラスタ化されたインデックスレコードのIDがユーザから与えられたものであっても（f1=2の場合のように！）、自動インクリメントシーケンス生成器から与えられたものであっても、同じルールを適用するだけです。)

### セカンダリインデックスにネクストキーロックを作成していることの証明

両バージョンのコードがセカンダリインデックスにネクストキーロックを作成していることをさらに証明するため、 rev a6c1dd2c99f4a のコードを検査してみました。

ステップ3.のセカンダリインデックスのロックが（レコードだけでなく）ギャップもカバーしているという判断は、以下の通りである。

```
const ulint lock_type =
    index->table->skip_gap_locks() ? LOCK_REC_NOT_GAP : LOCK_ORDINARY;
```

Xロック(排他ロック)であることは次のようになる

```
  } else if (allow_duplicates) {
```

最新のMySQLバージョンでもこのロジックは同じです。

そのため、バグフィックスでは、セカンダリインデックスのために取られたロックの種類は、私が知っている方法と変わりませんでした。しかし、変更されたのは`row_ins_duplicate_error_in_clust()`で使用されるロックの種類で、PRIMARYインデックス(a.k.a. クラスタインデックス)での競合をチェックします。

バグ修正前はレコードとギャップの両方をカバーするロック(Next Key Lock)を使用していましたが、修正後はレコードだけになりました。行を削除しなければならない場合には、ギャップロックとしてロックを適切に継承し、削除しなければギャップを保護する必要がないことに気付き、このような変更にした。これより並列性が向上する。

今回の変更は、一時的に挿入された行を削除する前に、ロックが暗示的なものから明示的なものに変換されているかどうかを適切に確認することで実現しました。

もしバグフィックス前のバージョンの`row_ins_duplicate_error_in_clust()` 関数が行とギャップの両方をカバーする X ロックを作成していたとしたら、そもそもなぜ "f1=2 ギャップロック" に問題があったのでしょうか？

カーソルが競合する行を見つけなかった場合、この関数は呼ばれません。つまりこの関数は、競合の証明となる挿入前に既に存在していた行のみをロックしますが、競合していないことの証明となるギャップはロックしません。

## Jakub Lopuszanskiの考え

> one might try to figure out a new, improved design, in which we try to handle a situation where autoincrement is involved in some other way, to exploit the intuitive notion that autoincrement values can not cause duplication and thus we don't have to preserve evidence of the lack of conflict for them.
>
> オートインクリメントが他の方法で関与している状況を処理するために、オートインクリメントの値が重複を引き起こすことができないという直感的な概念を利用して、オートインクリメントのために競合がないという証拠を保存する必要がないという新しい改良されたデザインを考え出すかもしれません。
>

> I estimate that it would be highly non-trivial to implement correctly, and IMHO classifies as a feature request (as opposed to bug).
>
> 私は、このロックを正しく実装することは非常に困難であると推定しており、個人的には機能要求（バグとは対照的）として分類しています。

> "Deadlock happens" is not a bug - deadlocks should be handled by a properly written application, as they are one of possible outcomes of each transaction.
>
> "デッドロックが起こる "というのはバグではありません - デッドロックは各トランザクションで起こりうる結果の一つであるため、適切に書かれたアプリケーションによって処理されるべきです。
>

> "Deadlocks happen more often than before" is also not a bug (Yes, it might be a symptom of a bug if the change was a surprising result of some unrelated change we haven't thought through. But in our case, the deadlocks happen more often because we lock more often, and this was an intended change)
>
> "デッドロックが以前よりも頻繁に起こる"というのもバグではありません (もしその変更が私たちの考慮漏れによる無関係な変更の予想外な結果であった場合はバグの症状かもしれません。しかし、これまで述べたような場合は、ロックする回数が増えたことでデッドロックがより頻繁に起こるようになりました。これは意図した変更だったのです)
>

> "I'd like deadlocks to happen less often" is a feature request in my opinion, unless one can point out where we make a mistake by taking a lock.
>
> "デッドロックの発生頻度を減らしてほしい "というのは、ロックを取ることでどこでミスをしているのかを指摘されない限り、私の考えでは機能的な要望だと思います。

## 解決方法

サイト後半のほう、いくつか回避方法を提示している

1. 一行ずつ処理する
    - 遅すぎる
2. INSERT...ON DUPLICATE KEY UPDATE ... "をトランザクションBEGINで置き換えてみてください
    - セカンダリインデックスで競合するSELECT
    - 競合する行をUPDATE
    - 新企行はINSERT
    - アプリケーションレイヤーでINSERT/UPDATEの判別ロジックが必要になる
3. Run server with `--innodb-autoinc-lock-mode=0`.
    - DB性能が落ちる
4. トランザクション先頭に`SELECT MAX(id) FROM ... FOR UPDATE;`
    - MAX(id)だけロックを取るようになる
    - 2の解決方法のほうがまだ良さそう
5. READ COMMITTED分離レベルに下げる
    - レプリケーションが安全でなくなる
6. 行の一意性を担保するユニークキーを設定する
    - PRIMARY IDとpackage_nameでユニークを担保しているので今障害ではよろしくない
7. 単一スレッドからリクエストするようにする
    - concurencyが実現できないならおそすぎるので却下
8. 挿入するスレッドに PRIMARY KEY の不連続なセットを事前に割り当ててみてください。
    - ある種のスキームや外部サービスを使用して、不連続であることが保証された id 値を取得し (MySQL/Redis/Memcached であれば何でも)、各インサータから PRIMARY KEY の不連続なフラグメントに挿入することができます。
    - ちょっとここはよくわからない
9. テーブルをシャードして、各インサータが特定のシャードで作業するようにします。
    - これは良いアイデアだと思います。modulo N と --auto-increment-increment=N を使ってシャードすることもできますし、それぞれのシャードに別々の範囲の ID を割り当てることもできます (例えば、一貫したハッシュのようなスキームで)。
    - シェアードナッシング実現するのに工数がかかりすぎる

> I'm hope it doesn't sound like I believe that any of the above is trivial. I just think each of above would be easier than changing the server code in this area.
>
> 上記のどれもが些細なことだと思っているように聞こえなければいいのですが。ただ、このあたりのサーバーコードを変更するよりも、上記のそれぞれの方が簡単だと思います。

> If you want an improvement on the server side, then motivating, real-world examples of queries are welcomed, as this could be used as input for a design of a new feature.
>
> サーバー側の改善を望むのであれば、モチベーションの高い、実際のクエリの例を歓迎します。

MySQLのEXPLAIN
===

- [MySQLのEXPLAINを徹底解説](http://nippondanji.blogspot.com/2009/03/mysqlexplain.html)を参考


```shell=
mysql> EXPLAIN SELECT * FROM Country, (SELECT * FROM City WHERE Population > 1000000) AS C1 WHERE Country.Code = C1.CountryCode;
+----+-------------+------------+--------+---------------+---------+---------+----------------+------+-------------+
| id | select_type | table      | type   | possible_keys | key     | key_len | ref            | rows | Extra       |
+----+-------------+------------+--------+---------------+---------+---------+----------------+------+-------------+
|  1 | PRIMARY     | <derived2> | ALL    | NULL          | NULL    | NULL    | NULL           |  237 |             |
|  1 | PRIMARY     | Country    | eq_ref | PRIMARY       | PRIMARY | 3       | C1.CountryCode |    1 |             |
|  2 | DERIVED     | City       | ALL    | NULL          | NULL    | NULL    | NULL           | 4079 | Using where |
+----+-------------+------------+--------+---------------+---------+---------+----------------+------+-------------+
```

### 1. id

- 実行順序。同数字の場合は同時に実行されている。


### 2. select_type

select_typeはクエリの種類を表す。クエリによって表示が変わるので以下参照。

#### JOINの場合

クエリがJOINのみから構成される場合は「SIMPLE」と表示される。SIMPLEであっても単純という意味ではない

#### サブクエリの場合

|key|説明|
|---|---|
|PRIMARY|外部クエリを示す|
|SUBQUERY|相関関係のないサブクエリ。実際に呼ばれるのは最初の一回だけ。それ以降はキャッシュされた結果が返る|
|DEPENDENT SUBQUERY|相関関係のあるサブクエリ。|
|UNCACHEABLE SUBQUERY|実行する度に結果が変わる可能性のあるサブクエリ。|
|DERIVED|FROM句で用いられているサブクエリ。これだけサブクエリ => 外部クエリの順で実行される|

#### UNIONの場合

クエリにUNIONが含まれている場合は次の5つ



| key | 説明 |
| -------- | -------- |
| PRIMARY     | UNIONにおいて最初にフェッチされるテーブル     |
|UNION|2番目以降にフェッチされるテーブル|
|UNION RESULT|UNIONの実行結果|
|DEPENDENT UNION|DEPENDENT SUBQUERYがUNIONになっている場合|
|UNCACHEABLE UNION|UNCACHEABLE SUBQUERYがUNIONになっている場合|



### 3. type

対象のテーブルに対してどのような方法であアクセスするかを示す。致命的なクエリだとここを見ればひと目で分かる。

|key|説明|
|---|---|
|const|PRIMARY KEYまたはUNIQUEインデックスの夜アクセス。最速|
|eq_ref|JOINにおいてPRIMARY KEYまたはUNIQUE KEYが利用されるときのアクセスタイプ|
|ref|ユニークでないインデックスを使って等価検索(WHERE key = val)を行ったときに使われるアクセスタイプ。|
|range|indexを用いた範囲検索|
|index|インデックス全体をスキャンする必要があるのでとても遅い|
|ALL|フルテーブルスキャン。一番遅い|

indexまたはALLだったらチューニング必須

### 4. possible_keys

オプティマイザがテーブルのアクセスに利用可能なインデクスの候補としてあげたキーの一覧。

### 5. key_len

選択されたキーの長さ。短いほうが高速

### 6. key

オプティマイザによって選択されたキー。possible_keysの中から実際に使用したキーとなる。

### 7. rows

そのターブルからゲッチされる行数の大まかな見積もり。正確な値ではないので注意が必要。

DERIVEDテーブルのときは正確な行数が返る

extra列がUsing whereのときは、フェッチした行からWhere条件に一致する行だけが返ることに注意。

JOINの場合、WhereがなければJOINするテーブルのrowsの積として考えることができる。レコードタイプがeq_refの場合、rowsは１。refやrangeになっている場合は、多数のテーブルがJOINされてしまうので注意。

### 8. extra

オプティマイザがクエリを実行するためにどのような戦略を選択したのかが示される。

|key|説明|
|---|---|
|Using where|WHERE句に検索条件が指定されており、なおかつインデックスを見ただけではWHERE句の条件をすべて適用することができない場合に示される|
|Using index|クエリがインデックスだけを用いて解決できるていることを示す。|
|Using filesort|クイックソートを行っていることを示す。改善の余地があるため[こちら](http://nippondanji.blogspot.com/2009/03/using-filesort.html)を参照すること。かなり良くない。
|Using temporary|JOINの結果をソートしたり、DISTINCTによる重複削除をおこなう場合など、クエリの実行にテンポラリテーブルが必要なことを示す。あまり良くない。|
|Using index for group-by|MIN()/MAX()がGROUP BY句と併用されているときに、クエリがインデクスだけを用いて解決できることを示す|
|Range checked for each record (index map: N|JOINにおいてrangeまたはindex_mergeが利用される場合に表示される。|
|Not exists|LEFT JOINにおいて、左側のテーブルからフェッチされた行にマッチする行が右側のテーブルに存在しない場合、右側のテーブルはNULLとなるが、右側のテーブルがNOT NULLとして定義されたフィールドでJOINされている場合にはマッチしない行を探せば良い・・・ということを示す。
|Using index condition|MySQL5.6から導入された。ICP(Index Condition Pushdown)最適化と呼ばれ、クラスタ化インデックスから取得するレコードが減るので高速になる。|

### EXPLAIN利用時の注意点

- 実データを利用することが非常に大事
  - テーブルの行数やインデックスの分散具合によって実行計画に違いが生じる
  - 本番では100万行なのに100行しかないテーブルでクエリの最適を行ってもあまり意味がない。


フェッチしていくる行数をへらすことに注力する

#### 参考サイト
- [MySQLのEXPLAINを徹底解説!!](http://nippondanji.blogspot.com/2009/03/mysqlexplain.html)
- [MySQLのexplainとかについてしらべたときのメモ](https://qiita.com/lastcat_/items/de7b530a94fbcf9ba646#%E3%81%9D%E3%82%82%E3%81%9D%E3%82%82order-by%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%AA%E3%81%84%E3%81%AE%E3%81%ABusing-filesort%E3%81%8C%E5%87%BA%E3%81%A6%E3%81%84%E3%82%8B%E7%82%B9%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)
- [SQLチューニング～お手軽に使えるヒント句まで](https://qiita.com/kite_999/items/05136fadebc4048e3bb6)
- [初心者はこれだけ覚えておけ！ SQLチューニング観点 10選](https://bebee5.com/%E5%88%9D%E5%BF%83%E8%80%85%E3%81%AF%E3%81%93%E3%82%8C%E3%81%A0%E3%81%91%E8%A6%9A%E3%81%88%E3%81%A6%E3%81%8A%E3%81%91%EF%BC%81-sql%E3%83%81%E3%83%A5%E3%83%BC%E3%83%8B%E3%83%B3%E3%82%B0%E8%A6%B3/)
- [MySQL データベースの負荷対策/パフォーマンスチューニング備忘録 インデックスの基礎〜実践](https://qiita.com/marnie_ms4/items/576055abc355184c51a1)
- [はじめてのMySQL。意外と知らない3つのTips](https://developers.gnavi.co.jp/entry/mysql-tips)
- [8.8.2 EXPLAIN 出力フォーマット](https://dev.mysql.com/doc/refman/5.6/ja/explain-output.html)
- [ヤフー社内でやってるMySQLチューニングセミナー大公開](https://www.slideshare.net/techblogyahoo/mysql-58540246)
- [MySQL with InnoDB のインデックスの基礎知識とありがちな間違い](https://techlife.cookpad.com/entry/2017/04/18/092524)
- [SQL実行計画の疑問解決には「とりあえずEXPLAIN」しよう](https://thinkit.co.jp/article/9658)

[元記事](https://www.infoq.com/jp/articles/Starting-With-MongoDB/)より、かいつまんで記載

## 認証なしでMongoDBサーバを立ち上げてしまった

- MongoDBはデフォルトで認証機能なし。
    - ローカルだと問題ないけど場合によっては[インジェクションの危険性](https://lockmedown.com/securing-node-js-mongodb-security-injection-attacks/)がある
- ユーザID/パスワードによる資格確認がインストールも管理も簡単

## MongoDBの攻撃可能面の対策を忘れていた

- [セキュリティチェックリスト](https://www.mongodb.com/docs/manual/administration/security-checklist/)は一読すること。
    - ネットワークへの侵入やデータ侵害のリスクを低減するためのアドバイスが紹介されている。
- デフォルトでは設定ファイルで`javascriptEnalbed:false`をセットしておくのが良い。

## スキーマの設計に失敗した

- MongoDBではスキーマ定義は必須ではないが必要なという意味ではない。
    - 一貫性ない場合、保存は極めて高速だが取得で[大変なことになる](https://www.compose.com/articles/mongodb-with-and-without-schemas/)

## コレーション(ソート順)のことを忘れていた

- MongoDBデフォルトで[バイナリコレーション](https://jira.mongodb.org/browse/SERVER-1920)(大文字小文字を区別する)
    - おおよそ区別すべきではない
    - [MySQLもBINARYをつけなければ区別しない](https://dev.mysql.com/doc/refman/8.0/ja/case-sensitivity.html)

## 大きなドキュメントでコレクションを生成してしまった

- 16MBまでの大容量ドキュメントをコレクションに収容可能
    - 可能なだけで、最大付近が望ましいという意味ではない。
- 数キロバイトぐらいで保つのが最も効果的
    - 大きなドキュメントは[パフォーマンス上の問題](https://www.reddit.com/r/mongodb/comments/573fqr/question_mongodb_terrible_performance_for_a/)を発生させる

## 大きな配列でドキュメントを作成してしまった

- 配列を含める際は、要素数を4桁未満にすることがベスト
- 要素が追加されて、格納されているドキュメントを超えると[ディスク上のロケーションを変更する必要が生じて](https://www.mongodb.com/docs/manual/core/data-model-operations/#document-growth)、結果的に[すべてのインデックスを更新しなければならない](https://www.mongodb.com/docs/manual/core/write-performance/#document-growth)
- 大きな配列を持つドキュメントでは、別々の[インデックスエントリが配列の全要素に存在する](https://www.mongodb.com/docs/manual/core/index-multikey/)ため、インデックスの書き換えが何度も発生することになる。
    - 同じようなインデックス更新は、ドキュメントの挿入や削除でも発生する。

## 集約でのステージ順が重要であることを忘れていた

> これは原文読みにいったほうが良い

## 高速書き込みを使用した

> 説明を見る限り、WiredTigerストレージエンジンを使用するMongo4系以上ではあまり関係なさそう。

- 永続性が低い場合、MongoDBを高速書き込み設定にしてはいけない
    - ファイア・アンド・フォーゲットモードモードでは、実際に書き込みが行われる前にコマンドが返ってくるので、速いように見える一方、ディスクに書き込まれる前にクラッシュするとデータが失われる
    - WiredTigerではジャーナリングを使ってこれを防止している。

## インデックスなしでソートした
- 単一でも複合でも、検索・集計のためのソートするときはindexを考慮する。
    - [How MongoDB indexes Work](https://studio3t.com/knowledge-base/articles/mongodb-index-strategy/)
- indexなしでもソートできるが、ソート操作は[全ドキュメントの合計サイズが32MBまで](https://docs.mongodb.org/manual/reference/limits/#Sort-Operations)という制限がある。
    - この制限に達するとエラーが生成されるか場合によっては[空のレコードセットが返される](https://www.sitepoint.com/7-simple-speed-solutions-mongodb/)

## サポートインデックスのないルックアップ

- ルックアップ(lookup)は、SQL joinと同じような機能を実行する。
    - 十分なパフォーマンスを得るには、外部キーとして使用するキー値にインデックスが必要。

## マルチアップデートを使用しなかった

- ドキュメントを一度に複数更新する場合は`db.collection.update()`メソッドを利用するが、optionで[`multi`](https://www.mongodb.com/docs/manual/reference/method/db.collection.update/#std-label-update-multi)パラメータを使用しないと予期せぬ挙動になる。
    - true: 検索条件を満たす複数のドキュメントを更新
    - false: 1つのドキュメントを更新。(デフォルト)

## ハッシュオブジェクト内のキー順序の重要性を忘れていた

- JSONは順序のないコレクションで構成される。
- MongoDBで使用するBSONでは、[検索時に順序が重要な意味を持つ](https://devblog.me/wtf-mongo)
    - 例えば`{ firstname: "Phil", surname: "factor" }`と`{ surname: "factor", firstname: "Phil" }`は**一致しない**
        - 名前と値のペアを確実に見つけるためには、ドキュメント内での順序を維持する必要がある

## nullとundeifnedを混同した

- JSON標準では`undefined`は有効ではないが、JavaScriptでは使用されている
- さらにBSONでは非推奨である上に、$nullに変換されるがこれは必ずしも満足できる解決策ではない。
    - [MongoDBでundefinedを使うのはやめよう](https://github.com/meteor/meteor/issues/1646#issuecomment-29682964)

## `$sort()`を使わずに`$limit()`を使用した

- クエリから返される結果のサンプルだけ欲しい場合に`$limit()`を使うことがよくあるが、ソートしない場合には順保証されないので注意
    - 確実な動作のためには「決定論的」、すなわち実行するたびに同じ結果が得られなくてはいけない
    - `$sort()`を使わない`$limit()`は非決定論的であるので、バグの温床になりかねない。

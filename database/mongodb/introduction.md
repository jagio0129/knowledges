MongoDB 入門
===

## 概要
ドキュメント指向データベースのNoSQL。

[公式ドキュメント](https://www.mongodb.com/)

![](https://camo.qiitausercontent.com/bf7c78b1687aa1227c4cff65472b3d5b6f57efb1/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3232313735392f34363031376264642d656461362d363430622d356335662d3664316235343738663961632e706e67)

## アーキテクチャ
![](https://camo.qiitausercontent.com/5ecbc5be1c8fd8f65f9b3df78e40899c18379956/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3232313735392f64623039653335332d366230312d346462342d656439352d3763666637343233396233632e706e67)

- 「ドキュメント」と呼ばれるデータをJOSNライクな形式で表現し、ドキュメントの集合を「コレクション」として管理する
- コレクションがスキーマレスなドキュメントで格納され、任意のフィールドをすきなときに追加できる
- ドキュメントに複雑な階層構造をもたせることができ、その構造に含まれるフィールドを指定したクエリやインデックス生成も簡単な指定によって行える
- KVSでは苦手なValue検索をができ、ソート・集計を実現できる

## 機能
- レプリカ
- シャーディング
- GridFSと呼ばれるプロトコルを使用することで大きなファイルをデータベースに格納・取得することができる
- インメモリ機能により、高速なREADを実現するためにpageという単位でデータをメモリに保持している
  - メモリにない場合はpage faultとなり、ディスクからロードする

## シェア
[DB-Engines](https://db-engines.com/en/ranking)によると、21/08/02時点で5位。redisより上。

## 導入コスト
- 様々なプラットフォーム、言語に対応
- レプリカの仕組みがRDBMSに比べてかんたん
- シャーディングもRDBMSに比べて楽

## 注意点
### トランザクションが弱い
MongoDBはトランザクションを基本的にサポートしていない。

MongoDB 4.0からマルチドキュメントトランザクションがサポートされ、RDBMSのような一貫性のある処理を行うことができるが、以下の条件が必要となる。

- レプリカセット構成あるいはシャーディング構成
- Feature Compatibility Version (FCV) 4.0以降
- WiredTigerストレージエンジン
  - 3.2からWiredTigerがデフォルト

複数のトランザクションを必要としたクエリが必要になる場合、MongoDBは適していない

### スキーマレス
何でも入るということは何が入っているかわからない。データ構造がはっきりしていないので、型のないRubyで扱う際はしっかりチェックする必要がありそう

### その他
- ドキュメントサイズは16MBまで
- インデックスを作りすぎるとInsertが遅くなる(これはRDBMSも同じ)
- 2.2からデータベース単位でロックが掛かるようになった
  - 書き込みクライアントがいると読み込みすらできない


## 使ってみる

> Webでサクッと試したい場合は[こちら](https://docs.mongodb.com/manual/tutorial/getting-started/)

### docker-compose

```yaml
# https://hub.docker.com/_/mongo/
version: '3.1'

services:

  mongo:
    image: mongo
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    ports:
      - 27017:27017
    volumes:
      - ./db:/data/db
      - ./configdb:/data/configdb

  # データが見れるGUIサービス
  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example
```

> mongo-expressの概要については[こちら](https://qiita.com/koshilife/items/dcc2b17e4c880d65ba78)

### 基本コマンド

```shell
# アクセス
docker-compose exec mongo bash
mongo admin -u root -p

# データベース確認
show databases

# データベース選択
use <dbname>
```

#### INSERT
```shell
#  コレクション「col1」に対して
#  データ形式はJSON
db.col1.insert({'name': 'mongo'})

# 変数も使える
data = {"string": "hello", "array": ["sun", "mon", "tue"]}
db.col1.insert(data)

# JavaScriptの制御機構が使える
for(var i=0; i<=10; i++) db.col1.insert({x:4, j:i})
```

#### SELECT
```shell
# 上記INSERTを実行済みとして
doc = db.col1.findOne()
  # {
  #         "_id" : ObjectId("610be3da86c87b06196b3734"),
  #         "string" : "hello",
  #         "array" : [
  #                 "sun",
  #                 "mon",
  #                 "tue"
  #         ]
  # }

printjson(doc['_id'])
  # ObjectId("610be3da86c87b06196b3734")


# カーソルが使える
var c = db.col1.find()
while (c.hasNext()) printjson(c.next())
  # {
  #         "_id" : ObjectId("610be3da86c87b06196b3734"),
  #         "string" : "hello",
  #         "array" : [
  #                 "sun",
  #                 "mon",
  #                 "tue"
  #         ]
  # }
  # { "_id" : ObjectId("610be42386c87b06196b3735"), "x" : 4, "j" : 0 }
  # { "_id" : ObjectId("610be42386c87b06196b3736"), "x" : 4, "j" : 1 }
  # { "_id" : ObjectId("610be42386c87b06196b3737"), "x" : 4, "j" : 2 }
  # { "_id" : ObjectId("610be42386c87b06196b3738"), "x" : 4, "j" : 3 }
  # { "_id" : ObjectId("610be42386c87b06196b3739"), "x" : 4, "j" : 4 }
  # { "_id" : ObjectId("610be42386c87b06196b373a"), "x" : 4, "j" : 5 }
  # { "_id" : ObjectId("610be42386c87b06196b373b"), "x" : 4, "j" : 6 }
  # { "_id" : ObjectId("610be42386c87b06196b373c"), "x" : 4, "j" : 7 }
  # { "_id" : ObjectId("610be42386c87b06196b373d"), "x" : 4, "j" : 8 }
  # { "_id" : ObjectId("610be42386c87b06196b373e"), "x" : 4, "j" : 9 }
  # { "_id" : ObjectId("610be42386c87b06196b373f"), "x" : 4, "j" : 10 }
```

### MySQLと比較したクエリ

以下の記事が参考になる

[MongoDBのクエリを使いこなそう](https://gihyo.jp/dev/serial/01/mongodb/0003?page=2)

## 参考サイト
- [やってみようNoSQL MongoDBを最速で理解する](https://qiita.com/Brutus/items/8a67a4db0fdc5a33d549)
- [MongoDBの利点8つ｜MongoDBの概要や特徴についても解説](https://www.fenet.jp/infla/column/engineer/mongodb%E3%81%AE%E5%88%A9%E7%82%B98%E3%81%A4%EF%BD%9Cmongodb%E3%81%AE%E6%A6%82%E8%A6%81%E3%82%84%E7%89%B9%E5%BE%B4%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6%E3%82%82%E8%A7%A3%E8%AA%AC/)
- [mongoDBを使う時気を付けていること](https://sitest.jp/blog/?p=9580)
- [MongoDBの性能 - MongoDBでゆるふわDB体験](https://gihyo.jp/dev/serial/01/mongodb/0013)
- [第3回 MongoDBのクエリを使いこなそう - MongoDBでゆるふわDB体験](https://gihyo.jp/dev/serial/01/mongodb/0003)

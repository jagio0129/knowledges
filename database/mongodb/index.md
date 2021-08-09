インデックス
===

インデックスがなければ、MongoDBはコレクションスキャン。つまりコレクション内のすべてのドキュメントをスキャンして、クエリ分にマッチするドキュメントを選ぶ必要がある。クエリに適切なインデックスがあればMongoDBはそのインデックスを使って検査すべきドキュメンtの数を制限することができる。

インデックスには、特定のフィールドまたはフィールドのセットの値が、そのフィールドの値の順に格納される。インデックスエントリの順序は、効率的な等値性のマッチングや範囲ベースのクエリ操作をサポートする。更にMongoDBはインデックスの順序を使ってソートされた結果を返すことができる。

次の図はインデックスを使って一致するドキュメントを選択肢、順序付けするクエリを示している。

![](https://docs.mongodb.com/manual/images/index-for-sort.bakedsvg.svg)

基本的にはMongoDBのインデックスは他のデータベースシステムのインデックスと似ている。MongoDBはコレクションインデックスを定義しており、MongoDBコレクション内のドキュメントのあらゆるフィールドやサブフィールドに対するインデックスをサポートしている。

## Default `_id` index

MongoDBはコレクション作成時に`_id`フィールドに一意のインデックスを作成する。_idインデックスはクライアントが_idフィールドに同じ値をもつ2つのドキュメントを挿入することをふせぐ。この_idフィールドのインデックスを削除することはできない。

> シャードクラスタにおいて_idフィールドをシャードキーとして使用しない場合、アプリケーションはエラーを防ぐために_idフィールドの値の一意性を確保する必要がある。これには標準的な自動生成されたObjectIdを使用することが多い。

## Indexの作成

```
db.collection.create>index()
# nameフィールドにindexを貼る場合
db.collection.createIndex({ name: -1})
```

MongoDBのインデックスはB-tree構造

## Indexの名前

インデックスのデフォルトの名前は、インデックスのキーとインデックス内での書くキーの方向(1,-1)をアンダースコアをセパレータにして連結したものになる。例えば{item: 1. quantity: -1}に対して作成されたインデックスの名前は、`item_1_quantity_-1となる。

任意のインデックス名をつけることもできる。

```
db.products.createIndex(
  { item: 1, quantity: -1 } ,
  { name: "query for inventory" }
)
```

インデックスの名前はdb.collection.getIndexes()メソッドで確認できる。一度作成したインデックスの名前を変更することはできない。代わりにインデックスを削除して新しい名前で作成する必要があります。

## Indexの種類

### 単一インデックス

MongoDBで定義されている_idインデックスに加えてMongoBDはドキュメントの単一のフィールドに対してユーザ定義の照準/降順インデックスを作成することをサポートしている。

![](https://docs.mongodb.com/manual/images/index-ascending.bakedsvg.svg)

シングルフィールドのインデックスとソート操作では、インデックスキーのソート順(昇順か降順)は重要ではない。MongoDBはインデックスをどちらの方向にでも横断できる。

詳細は[Single Filed Indexes](https://docs.mongodb.com/manual/core/index-single/)と[Sort with a Signle Field index](https://docs.mongodb.com/manual/tutorial/sort-results-with-indexes/#std-label-sort-results-single-field)を参照。

### 複合インデックス

複合インデックスに記載されるフィールドの順番には意味がある。{userid:. score: -1}でインデックスが構成されている場合はこのインデックスはまずuseridでソートされ、次に書くuseridの値の中でscoreでソートされる。

![](https://docs.mongodb.com/manual/images/index-compound-key.bakedsvg.svg)

### マルチキーインデックス

配列の値を保持するフィールドをインデックス化すると、MongoDBは配列の各要素に対して個別のインデックスエントリを作成する。このマルチキーインデックスを使うと、クエリで配列を含むドキュメントを選択する際に、配列の要素にマッチさせることができる。

![](https://docs.mongodb.com/manual/images/index-multikey.bakedsvg.svg)

### 地理空間インデックス

地理空間座標データの効率的なクエリをサポートするために、MongoDBは2角特別なインデックスを提供している。結果を返すときに平面幾何学を使う2dインデックスと、球形幾何学を使う2dsphereインデックスである。詳細については[2d index internals](https://docs.mongodb.com/manual/core/geospatial-indexes/)

### テキストインデックス

省略

### ハッシュインデックス

省略

## インデックスプロパティ

### ユニークインデックス

ユニークプロパティを設定すると、MongoDBはインデックス対象のフィールドの重複した値を拒否する。

### パーシャルインデックス

省略

### スパースインデックス

省略

### TTL インデックス

省略

### Hidden インデックス

省略

## インデックスの利用

MongoDBはクエリオプティマイザによってインデックスの読み取り操作の効率を向上させることができる。[Analyze Query Performance](https://docs.mongodb.com/manual/tutorial/analyze-query-plan/)チュートリアルでは、インデックスが有る場合とない場合のクエリの実行統計の例を示している。

詳細については[クエリオプティマイザ](https://docs.mongodb.com/manual/core/query-plans/#std-label-read-operations-query-optimization)を参照

## 参考
- [Index](https://docs.mongodb.com/manual/indexes/#compound-index)

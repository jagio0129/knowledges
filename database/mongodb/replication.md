レプリケーションについて
===

## レプリケーションのメリット
mongoDBはマスター/スレーブ方式を採用している

### 可用性の向上
マスターノードがスレーブノードと動悸しているため、マスタ似障害が起きてもデータロストやサービス停止時間を最小限にすることが可能。DR(ディザスタリカバリ)対応のため地理的に離れたデータセンターにノードを分散させるケースも多い。

### 保守性の向上
負荷の高い操作をスレーブノードで実行できることによりメンテナンス性を貯めることができる。例えばバックアップをスレーブで行い、不要な負荷をマスタにかけないようにするなど。他にも負荷の高い大規模なインデックス構築なども、まずはスレーブノードで構築しスレーブを既存のマスタと交代させ、その後新たなスレーブでもう一度インデックスを構築するといった方法も実現できる。

### 負荷分散による性能向上
リードレプリカを構築することも可能。Readの負荷をスレーブに分散させることでシステムをスケールさせることが可能。

## レプリケーション概要
レプリケーションは一つのMongoDBにあるデータを別のMongoDBに複製できる機能。従来のRDB担い機能として、MongoDBは自動フェイルオーバーを実装している。マスタに障害が発生しても自動的にフェイルオーバーとリカバリが可能。「書き込み保証」や「読み取り負荷分散」はスケーラブルに設計されたMongoDBの特徴的な機能である。

![](https://docs.mongodb.com/manual/images/replica-set-read-write-operations-primary.bakedsvg.svg)

セカンダリ(スレーブ)はプライマリ(マスタ)が利用できない場合、資格のあるセカンダリが自分自身を新しいプライマリに選出する投票を行う。セカンダリノードの詳細については[レプリカセットのセカンダリメンバー](https://docs.mongodb.com/manual/core/replica-set-secondary/)を参照。

各ノードは互いにHeartbeatして生存確認している。

![](https://docs.mongodb.com/manual/replication/)

コストの関係などでセカンダリを複数持てない場合、レプリカセットにアービターというノードを設けることができる。アービターは投票券を持つが、データの保持はしない。そのため最小スペックのノードでも問題なくなる。アービターの詳細は[レプリカセット アービター](https://docs.mongodb.com/manual/core/replica-set-arbiter/)を参照。

![](https://docs.mongodb.com/manual/images/replica-set-primary-with-secondary-and-arbiter.bakedsvg.svg)

## 非同期でのレプリケーション
セカンダリはプライマリの[オプログ](https://docs.mongodb.com/manual/core/replica-set-oplog/)を複製し、非同期にデータセットに操作を適用する。複数のノードに障害が発生してもレプリカセットは継続して機能する。

レプリケーションの仕組みの詳細については「[レプリカセットのオプログ](https://docs.mongodb.com/manual/core/replica-set-oplog/#std-label-replica-set-oplog)」と「[レプリカセットのデータ同期](https://docs.mongodb.com/manual/core/replica-set-sync/#std-label-replica-set-sync)」参照。

### スローオペレーション
バージョン4.2から、セカンダリメンバーが、しきい値よりも遅く適用されたoplogエントリを記録するようになった。レプリケーションのスローログみたいなもの。

### レプリケーション遅延とフロー制御
レプリケーション遅延とは、プライマリで行われた書き込み操作をセカンダリに複製するのに時間がかかること。ラグが大きくなるとプライマリのキャッシュが圧迫されるなどの重大な問題が発生する。

4.2移行、管理者はプライマリが書き込みを行う速度を制限できる。その目的は、コミットされた遅延の大部分を設定可能な最大値(flowControlTargetLagSecouds)以下に抑えること。この機能はデフォルトでON。

注意としては、レプリカセット/シャドークラスタの[FearueCompabilityVersion(FCV)](https://docs.mongodb.com/manual/reference/command/setFeatureCompatibilityVersion/#std-label-view-fcv)が4.2で、[majority read concern](https://docs.mongodb.com/manual/reference/configuration-options/#mongodb-setting-replication.enableMajorityReadConcern)が有効になっていること

フロー制御を有効にすると、ラグがflowContolTargetLagSecoundsに近づくにつれて、プライマリへの書き込みはロックをかけて書き込みを行う前にチケットを取得する必要がある。1病患に発行されるチケットの数を制限することで、フロー制御機構はラグを目標地位会に抑えようとする。

詳細は「[レプリケーションの遅延](https://docs.mongodb.com/manual/tutorial/troubleshoot-replica-sets/#std-label-replica-set-replication-lag」)と「[フロー制御の確認](https://docs.mongodb.com/manual/tutorial/troubleshoot-replica-sets/#std-label-flow-control)」

## 参考サイト
- [mongoDB Documentation - Replication](https://docs.mongodb.com/manual/replication/)
- [MongoDBのレプリケーションを構築してみよう](https://gihyo.jp/dev/serial/01/mongodb/0004)

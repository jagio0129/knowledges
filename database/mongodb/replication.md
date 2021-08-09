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

## 自動フェイルオーバー

プライマリが設定された期間(electionTimeoutMillis: default 10s)を超えて他のメンバーと通信がない場合、歯科雨のあるセカンダリが自分を新しいプライマリとなるために投票を要求する。

![](https://docs.mongodb.com/manual/images/replica-set-trigger-election.bakedsvg.svg)

投票が正常に終了するまでレプリカセットは書き込み処理できない。プライマリがオフラインのときにリードレプリカ設定している場合は、レプリカセットは読み取りクエリを継続して実行できる。

クラスタが新しいプライマリを選出するまでの時間の中央値は、レプリカの設定がデフォルトの場合、数条12秒を超えないようにすること。これにはプライマリを利用できないとマークし、投票を呼びかけて完了するまでの時間が含まれる。この時間を調整するには、setting.electionTimeoutMillisを変更する。ネットワークの遅延などにより、レプリカセットの投票完了までの時間が長くなることがあり、プライマリがいない状態でクラスタが動作する時間に影響する。これらの要因はクラスタのアーキテクチャによって異なる。

electionTimoutMillisオプションをデフォルトの10sから下げると、プライマリの障害をより早く検出することができる。しかし、プライマリが正常であっても、一時的なネットワーク遅延などの要因により、クラスタがより頻繁に選挙を行う必要がある。その結果、w:1の書き込み操作のロールバックが増加する可能性がある。

> `w` option
> `w`オプションは、書き込み操作が指定した数のmongodインスタンスや、指定したタブを持つmongodインスタンスに伝搬したことの確認を要求する
> https://docs.mongodb.com/manual/reference/write-concern/#std-label-wc-w

アプリケーションの接続ロジックには、自動フェイルオーバーとそれに続く選挙への体制をもたせる必要がある。MongoDB3.6以降、MongoDBドライバはプライマリが失われたことを検知して特定の書き込み操作を自動的に一度だけ再試行することができるようになり、自動フェイルオーバーや投票への対応が追加で組み込まれた。

- MongoDB4.2+ 互換性のあるドライバはデフォルトで再施工可能な書き込みを有効にする
- MongoDB4.0及び3.6互換のドライバではretryWrites=trueを接続文字列に含めて明治的に再施工可能な書き込みを有効にする必要がある。

4.4以降、MongoDBはミラーリングされた読み込みを提供し、投票で選ばれたセカンダリメンバーのキャッシュを直前まで持っている。セカンダリのキャッシュを予め持っておくことで、投票後の復帰を早くすることができる。

フェイルオーバーのプロセスについてのより多くの詳細は以下から。

- [Replica Set Elections](https://docs.mongodb.com/manual/core/replica-set-elections/#std-label-replica-set-elections)
- [Retryable Writes](https://docs.mongodb.com/manual/core/retryable-writes/#std-label-retryable-writes)
- [Rollbacks During Replica Set Failober](https://docs.mongodb.com/manual/core/replica-set-rollbacks/#std-label-replica-set-rollback)

## 参考サイト
- [mongoDB Documentation - Replication](https://docs.mongodb.com/manual/replication/)
- [MongoDBのレプリケーションを構築してみよう](https://gihyo.jp/dev/serial/01/mongodb/0004)

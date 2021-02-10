EC2(Elastic Compute Cloud)
===

仮想サーバ機能を提供するクラウドサービス。詳細は[公式のご説明](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/concepts.html)

## インスタンスの作成
以下の手順でインスタンスを作成します

1. AMIの選択
2. インスタンスタイプの選択
3. インスタンスの設定
4. ストレージの追加
5. タグの追加
6. セキュリティグループの設定

### AMIの選択
AMI は、インスタンスの作成に必要なソフトウェア構成 (OS、アプリケーションサーバー、アプリケーション) を含むテンプレート。

UbuntuとかAmazonLinuxとか選べる。自分でAMIを作成する事も可能っぽい(マイ AMI)

### インスタンスタイプの選択
インスタンスタイプはさまざまなCPU、メモリ、ストレージ、ネットワークキャパシティの組み合わせによって構成されている

### インスタンスの設定

- インスタンス数
  - 同じ構成を複数台(複数インスタンス)作れる

- 購入オプション(スポットインスタンスのリクエスト)
  - https://recipe.kc-cloud.jp/archives/321

- ネットワーク
  - VPCを指定する

- サブネット
  - サブネットを指定する

- 自動割り当てパブリック IP
  - DHCPみたいに割当たる

### セキュリティグループの設定
セキュリティグループはAWSのファイアウォール機能の一つ。インスタンスごとに適応することができる。ホワイトリストしか明示的に設定できない。

- インバウンドルール
  - そのセキュリティグループに関連付けられたインスタンスにアクセスできるトラフィックを規制するルール

- アウトバウンドルール
  - そのセキュリティグループに関連付けられたインスタンスからどの送信先にトラフィックを送信できるか(トラフィックの送信先と送信先ポート)を制御するルール

## インスタンスの起動

## インスタンスの停止
[インスタンスのライフサイクル](https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html#lifecycle-differences)

## インスタンスの削除
終了が削除にあたる。アクションからインスタンスを終了すると、terminatedにステータスが代わり、しばらくダッシュボードに残る。しばらくすると消える。


## 用語
### インスタンス
一台のサーバ

### EBS(Elastic Block Store)
サーバのハードディスクに相当する仮想ディスク

### AMI(Amazon Machine Image)
サーバにインストールするOS・各種ミドルウェア・アプリのイメージ

### [VPC(Amazon Virtual Private Network)](https://docs.aws.amazon.com/ja_jp/vpc/latest/userguide/what-is-amazon-vpc.html)
定義した仮想ネットワーク内でAWSリソースを起動できる

### Elastic IP
IPアドレスを一意に固定できるサービス

インスタンス作成時に割り当てられるIPアドレスは再起動したりすると変更される場合がある

- 料金発生条件
  - 一つのインスタンスに一つのElasticIPを割り当てていて、起動している場合は**発生しない**
  - 発生するのは以下の場合
    - 割り当て先のインスタンスが停止している
    - ElasticIPを取得しているのにインスタンスに割り当てていない場合
    - 2つ目以降のElasticIPを習得している場合
    - etc

- 料金形態
  - https://dev.classmethod.jp/articles/cost-of-eip/

## Amazon Routes 53
AmazonのEC2用DNSみたなやつ

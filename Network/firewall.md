ファイアウォール
===

![](https://image.itmedia.co.jp/ait/articles/0203/01/r20_fw1.jpg)

- 不正なアクセスなどから守るためのセキュリティ機能

## 種類

![](https://www.rworks.jp/wp-content/uploads/2020/09/FW-2.png)

### パケットフィルタリング型

事前に、通信を許可する送信元情報(IPアドレスやポート)や、宛先情報をルールとして設定することで、ルールに反する通信を遮断する。

仕組みがシンプルなので、パケットを見ただけでは攻撃されているか判別つかない。

### アプリケーションゲートウェイ型

HTTP・FTP・SMTPなどアプリケーションプロトコルごとに、通信を制御する仕組みを準備し、外部ネットワークとのやり取りを許可したり遮断したりする。

高精度なアクセス制限を設けることが可能。反対に、通信内容の検査処理に時間がかかったり、構築までに高い高ストが発生する。

### サーキットレベルゲートウェイ型

トランポー塗装レベルの通信を監視・制御する方式。コネクション単位の制御も可能で、パケットフィルタリングよりも設定や管理がかんたん。

コネクション単位で通信可否を制御するため、パケットフィルタリングでは防げない送信元IPアドレスの偽装(IPスプーフィング)を防ぐことが可能。またどのアプリケーションプロトコルでも汎用的に使用できるのもメリット。

## 機能

- 発信元IPや宛先IPに応じた通史の許可・遮断
- 通信プロトコルに応じた通信の許可・遮断
- 監視・管理

## iptables
Linuxに搭載されているパケットフィルタ。

### 設定確認

```
iptables -L

  Chain INPUT (policy DROP)
  target     prot opt source      destination
  ACCEPT     all  --  anywhere    anywhere
  ACCEPT     all  --  anywhere    anywhere      state RELATED,ESTABLISHED
  LOG        all  --  anywhere    anywhere      LOG level warning prefix "drop_packet: "

  Chain FORWARD (policy DROP)
  target     prot opt source      destination

  Chain OUTPUT (policy ACCEPT)
  target     prot opt source      destination
```

### Chain

ネットワークから受け取ったパケットは次のような手順で処理される

![](https://eng-entrance.com/wp-content/uploads/2016/07/chain.png)

このINPUT, OUTPUT, FORWARDの3つの経路に対して、それぞれ通過させるパケットのルールを設定する。この3つの経路をChainという

### 基本ポリシー

Chainには基本ポリシーを割り当てる。どのルールにも当てはなまらない場合、これらが基本設定ですよというルールになっている。

- DROP
  - ルールが指定されていな場合はすべて拒否
- ACCEPT
  - すべて許可

### それぞれの項目

Chainの下にある部分

```
  target     prot opt source      destination
```

- target
  - 条件にマッチするパケットの行き先をどうするか？という意味
- prot
  - プロトコルの種類
- opt
  - optはオプションが記述される
- source
  - どこから来たのか
- destination
  - どこへ向かうのか

## iptabelsの適応順番

iptablesは上から順番にルールを適応する。例えば下記のこの項目をまっさきに設定するとすべてのパケットログが取られてしまう。

```
LOG        all  --  anywhere    anywhere      LOG level warning prefix "drop_packet: "
```

## 最も一般的なフィルタリングテーブル

```
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

すべてアクセプトになっている。これに対して設定を追加していくのだが、基本的には次のような設定にする。

|チェイン|ターゲット|
|---|---|
|INPUT|DROP|
|FOWARD|DROP|
|OUTPUT|ACCEPT|

これは**すべての通信を遮断した上で必要な通信だけ許可する**のがセキュリティの基本となるため。

## iptablesの書式

```
iptables <テーブル> <コマンド> <マッチ> <ターゲット>
iptables -t filter -A INPUT -p tcp -j ACCEPT # 例
```
## 主なテーブルとコマンド

- テーブル
  - filter: 一般的なフィルタテーブル
  - nat: マスカレードなどを記述するテーブル
  - mangle: このテーブルを使うとQuality of Serviceなどが設定可能

- チェイン
  - INPUT: 入ってくるパケットに関して
  - OUTPUT: 出ていくパケットに関して
  - FORWARD: パケットの転送
  - PREROUTING: 受診時にアドレスを変換
  - POSTROUTING: 送信時にアドレスを変換

- ターゲット
  - ACCEPT: パケットを許可
  - DROP: パケットを拒否
  - REJECT: パケットを拒否して制御メッセージを送信
  - LOG: パケットのログを記録

- コマンド
  - -F: 何も指定されていない場合すべてのフィルタルールを削除する
  - -X: 何も指定されていない場合デフォルト以外のすべてのチェインを削除する
  - -A: ルールの追加
  - -D: ルールの削除
  - -P: ポリシーの指定

- その他パラメータ
  - -s: パケットの送信元
  - -d: パケットの送信先
  - -p: パケットのプロトコル指定
  - -i: 入ってくるインターフェイス指定(eth0など)
  - -o: 出ていくインターフェイス指定
  - -j: ターゲットの指定

- 拡張パラメータ
  - -m state -state
    - NEW: 新しい接続
    - ESTAVLEISHED: すでに許可されている接続
    - RELATED: 新しくかつ許可された接続
    - INVALID: 無効な接続
  - -p
    - --sport: 送信側のポート指定
    - --dport: 受信側のポート指定
    - --tcp-flags: SYN, ACK, FINなどカンマで区切り複数指定可能

## スクリプト例

```

#!/bin/bash

# 設定をクリア
iptables -F
iptables -X
# ポリシー設定
iptables -P INPUT   DROP
iptables -P FORWARD DROP
iptables -P OUTPUT  ACCEPT

# ほかACCEPTやDROPなどユーザによる設定

#ローカルループバックの接続を許可する。
iptables -A INPUT -i lo -j ACCEPT
#こちらから求めたパケットは許可する。
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
#それ以外はログを残す。
iptables -A INPUT -j LOG --log-prefix "drop_packet:"
```

実行はroot権限かつsshで接続している場合、sshを許可する設定の記述を忘れないこと

```
iptables -A INPUT -m state NEW -m tcp -p tcp -dport 22 -j ACCEPT
```

## 参考サイト
- [linuxのファイアウォール iptables](https://eng-entrance.com/linux-firewall)

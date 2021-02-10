多段SSH
===

SSHした先でさらに違うサーバにSSHする

イメージ図
![](https://wakuwakubank.imgix.net/S3Doc/dc/13/60/8b65ac3b0c33effb469311b778260e5b37aada6b4559fc94541ca5c575)

上記構成で以下情報とする

**EC2(public)**

|項目|説明|
|---|---|
|public IP| xxx.xxx.xxx.xxx|
|private IP|10.0.1.186|

**EC2(private)**

|項目|説明|
|---|---|
|private IP|10.0.3.236|

EC2(public)からのみアクセスできる

**RDS(private)**

|項目|説明|
|---|---|
|private IP|10.0.3.236|
|DB USER|sampleuser|
|DB PASSWORD|samplepass|

EC2(public)からのみアクセスできる

## => EC2(public) => EC2(private)

```sh
ssh -i [EC2(public)の鍵] \
-o ProxyCommand='ssh -i <EC2(private)の鍵> <ユーザー名>@<EC2(public)のアドレス> -W %h:%p' \
[ユーザー名]@[EC2(public)のアドレス]
```

## => EC2(public) => RDS  (ポートフォワーディング)

```sh
ssh -fNC \
-L [ローカルホストのポート]:[リモートホスト]:[リモートホストのポート] \
-i [EC2(public)の鍵] \
[ユーザー名]@[EC2(public)のアドレス]
```

|オプション|概要|
|---|---|
|-f|バックグラウンド実行|
|-N|接続先サーバでシェルを起動させない|
|-C|圧縮して送る|
|-L|ポートフォーワーディング設定|

通信が確立した後は以下のようにmysqlクライアントからRDSに接続することができる

```sh
mysql -u[DBユーザー名] -p[DBユーザーパスワード]  -h 127.0.0.1 --port=[ローカルホストのポート]
```

## 参考

- [多段SSH, ポートフォワーディングの方法](https://www.wakuwakubank.com/posts/681-ssh-portforward-multistage/)
- [踏み台の向こうにあるMySQLサーバに一発アクセスする](https://qiita.com/lighttiger2505/items/ea33291639a8656d50b4#%E8%B8%8F%E3%81%BF%E5%8F%B0%E3%82%B5%E3%83%BC%E3%83%90%E7%B5%8C%E7%94%B1%E3%81%A7db%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9)
- [つながるSSHトンネルが俺の力だ！](http://note.crohaco.net/2017/ssh-tunnel/)

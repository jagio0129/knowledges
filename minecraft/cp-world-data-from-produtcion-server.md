稼働サーバからワールドデータをコピーする方法
===

ここではワールドデータのみを稼働サーバからコピーする手順を記載する

## 前提事項
- 稼働サーバにSSHアクセスできる環境から行うこと
- 稼働サーバのマインクラフトが停止していること

## 手順

```sh
# 稼働サーバにて

# バックアップの作成
cp -Rp /opt/minecraft/ /home/minecraft/minecraft_yyyymmdd

# SSH元環境にて

# cp minecraft server data to local
ssh -r [ユーザ名]@[旧サーバアドレス]:/opt/minecraft_server [新サーバ側コピー先ディレクトリ]
cd [新サーバ側コピー先ディレクトリ]

# 所有者の確認
ls -l [新サーバ側コピー先ディレクトリ]

# ユーザ・グループの変更
#   minecraft server実行者に変更する
sudo chown -R [実行ユーザ]:[実行グループ] /opt/minecraft/

# minecraft server起動
java -Xmx1024M -Xms1024M -jar minecraft_server.1.17.1.jar nogui

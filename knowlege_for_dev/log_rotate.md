ログローテート
===

環境はUbuntu18。logrotateコマンドで実現する。

```
logrotate --version # => 3.11.0
```

```
# 設定ファイル構成
/etc/
├── logrotate.conf # メインの設定ファイル
├── logrotate.d # 各サービスごとの設定ファイル
│   ├── dracut
│   ├── iscsiuiolog
│   ├── mcollective
│   ├── mysql
│   ├── syslog
│   ├── yum
```

logrotate.confにすべての設定を記載してもいいが、logrotate.d配下もincludeされるのでサービスごとに設定ファイルを作成するのが望ましい。

### 設定ファイルの作成例

```
/var/log/sample-service/sample.log { # 対象のログファイル
    ifempty            # ログファイルが空でもローテーションする
    dateformat .%Y%m%d # dateフォーマットを任意のものに変更する
    missingok          # ログファイルがなくてもエラーを出さない
    compress           # 圧縮する
    daily              # 毎日ローテートする
    rotate 10          # 10世代分古いログを残す
    postrotate         # ローテート後にsyslogを再起動
        /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
    endscript
}
```

### ローテート確認

```sh
# dry-run
sudo ogrotate -dv /etc/logrotate.conf

# 本実行
/usr/sbin/logrotate /etc/logrotate.conf

# 確認
cat /var/lib/logrotate.status

  logrotate state -- version 2
  "/var/log/yum.log" 2017-1-13
  "/var/log/dracut.log" 2017-1-13
  "/var/lib/mysql/mysqld.log" 2017-1-13 # 対象のログがrotateされる時間が表示される
```

### 参考サイト
- https://qiita.com/Esfahan/items/a8058f1eb593170855a1
- https://www.itmedia.co.jp/help/tips/linux/l0291.html

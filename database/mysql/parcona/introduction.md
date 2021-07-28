percona-tool-kit
===
## What's this?
MySQLスロークエリとかの分析ができるpt-query-digestとかが入ったパッケージ

## Install for Ubuntu 18
```sh
# Download site
#   https://www.percona.com/downloads/percona-toolkit/LATEST/
wget https://downloads.percona.com/downloads/percona-toolkit/3.3.1/binary/debian/bionic/x86_64/percona-toolkit_3.3.1-1.bionic_amd64.deb
sudo apt-get install libdbd-mysql-perl libdbi-perl libio-socket-ssl-perl libnet-ssleay-perl libterm-readkey-perl
sudo dpkg -i percona-toolkit_3.3.1-1.bionic_amd64.deb
```

## Usage: pt-query-digest
```sh
sudo pt-query-digest /path/to/slow.log
```

以下のような結果が見れる
```
# 3.1s user time, 20ms system time, 38.36M rss, 101.36M vsz
# Current date: Sun Jul 15 15:37:37 2018
# Hostname: conoha
# Files: /var/log/mysql/slow.log
# Overall: 21.81k total, 31 unique, 0.12 QPS, 0.00x concurrency __________
# Time range: 2018-07-12T18:41:46 to 2018-07-14T21:25:01
# Attribute          total     min     max     avg     95%  stddev  median
# ============     ======= ======= ======= ======= ======= ======= =======
# Exec time            86s     1us      6s     4ms     3ms    89ms    98us
# Lock time          504ms       0   128ms    23us    31us     1ms       0
# Rows sent         18.11k       0     100    0.85    0.99    7.23       0
# Rows examine       8.78M       0   9.77k  421.97    0.99   1.87k       0
# Query size         1.70M      10 387.41k   81.88   62.76   3.67k   31.70
```

### 行の意味
|カラム|意味|
|---|---|
|Exec time|実行時間|
|Lock time|ロック待ち時間|
|Row sent|送信回数|
|Rows exeamine|フェッチした行数|
|Query size|SQLの値|

## その他の情報について
- [Percona Toolkitをインストールし、pt-query-digestでスロークエリログを解析する手順](https://nishinatoshiharu.com/percona-slowquerylog/)

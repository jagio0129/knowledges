vnstatまとめ
===
ネットワークトラフィックモニター

- お手軽に導入できる
- NICごとにIO(rx: 受信, tx: 送信)が確認できる
- DBに測定値を保存する
- 月/日/時でグラフ(png)をチュつ力してくれる
- リアルタイム計測もできる
  - vnstat起動してからのネットワーク量とか

## Install
パッケージで入れると古いものが降ってくるので最新のを入れる。1系と2系で見方と操作が結構変わる。

```sh
# 最新バージョンはhttp://humdi.net/vnstatのDawnloadsから確認
export VNSTAT_VERSION=2.6

wget http://humdi.net/vnstat/vnstat-$VNSTAT_VERSION.tar.gz
tar -xvf vnstat-$VNSTAT_VERSION
cd vnstat-$VNSTAT_VERSION

# makeの詳細はINSTALLファイルのtl;drに記載されている
sudo su
./configure --prefix=/usr --sysconfdir=/etc && make && make install

# start daemon
cp -v examples/systemd/vnstat.service /etc/systemd/system/
systemctl enable vnstat
systemctl start vnstat

# 確認(5分おきに更新されるのでdaemon起動直後はすべてNot enough data available yet.)
vnstat
                      rx      /      tx      /     total    /   estimated
  br-1c57b89f6509:
        Aug '20           0 B  /       810 B  /       810 B  /     --
          today           0 B  /       810 B  /       810 B  /     --

  br-3830e10f5584: Not enough data available yet.
  br-5ec8fc58768d: Not enough data available yet.
  br-634393e0dab0: Not enough data available yet.
  br-7b64447e8d6b: Not enough data available yet.
  br-c98c5a0c16d5: Not enough data available yet.
  br-de2e241457fc: Not enough data available yet.
  br-faa8bf0bc5d3: Not enough data available yet.
  docker0: Not enough data available yet.
  enp0s3:
        Aug '20    269.20 KiB  /  119.78 KiB  /  388.98 KiB  /     --
          today    269.20 KiB  /  119.78 KiB  /  388.98 KiB  /     703 KiB

  enp0s8:
        Aug '20         243 B  /   14.98 KiB  /   15.22 KiB  /     --
          today         243 B  /   14.98 KiB  /   15.22 KiB  /      27 KiB

  enp0s9:
        Aug '20      5.14 MiB  /   11.49 MiB  /   16.63 MiB  /   45.98 MiB
          today      5.14 MiB  /   11.49 MiB  /   16.63 MiB  /   30.13 MiB

  veth06ae626:
        Aug '20           0 B  /    1.26 KiB  /    1.26 KiB  /     --
          today           0 B  /    1.26 KiB  /    1.26 KiB  /     --

```

### 使い方
基本的な使い方は[公式](https://humdi.net/vnstat/)に記載されているので省略。

ここでは--liveのmodeだけ記載しておく。

```sh
# --liveで使えるmodeは0,1
vnstat -l 2 -i enp0s3
  Error: Invalid mode parameter "2" for -l / --live.
  Valid parameters:
    0 - show packets per second (default)
    1 - show transfer counters
```

## 参考
- [vnStat - a network traffic monitor for Linux and BSD - Humdi.net](https://humdi.net/vnstat/)
- [vnstatで手軽にtrafiicをモニタリング](https://qiita.com/kooohei/items/d4b29be4541cec5018c6)
- [Error: Failed to open database "/var/lib/vnstat/vnstat.db" in read-only mode.](https://github.com/vergoh/vnstat/issues/134)

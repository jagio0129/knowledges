---
title: JupyterでRubyを有効にする
tags: []
# excerpt: ローカルマシンで定期的にスクリプトを動かしたいと気に使える。cronのようなもの。
---
# JupyterでRubyを有効にする
Jupyter導入はpython/jupter.mdに記載

## 必要パッケージのインストール
```
sudo apt install libtool libffi-dev ruby ruby-dev make
sudo apt install git libzmq-dev autoconf pkg-config
git clone https://github.com/zeromq/czmq
cd czmq
./autogen.sh && ./configure && sudo make && sudo make install
gem install cztop iruby
iruby register --force
```

- <http://localhost:8888>がブラウザで開くとNewからRubyが選べるようになる。
- 選んだ後、Inのところにソースコードを貼り付けRunを押すとグラフが表示される

## 参考サイト
- [SciRuby/iruby](https://github.com/SciRuby/iruby)
- https://qiita.com/genya0407/items/b01f101d2f2725f77374
- https://qiita.com/tdrk/items/cdf54dcc55724b75e2f9
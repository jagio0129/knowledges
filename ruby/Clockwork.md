---
title: Clockwork
tags: ["gem"]
excerpt: ローカルマシンで定期的にスクリプトを動かしたいと気に使える。cronのようなもの。
---
# Clockwork
ローカルマシンで定期的にスクリプトを動かしたいときに使える。cronのようなもの。

## Install
gemなので`gem install clockwork`で導入。

## メソッド
メソッド                                                      | 概要
--------------------------------------------------------- | -------------------------------------------------------------
every                                                     | 定期実行するジョブを定義する
handler                                                   | ジョブ実行時に呼び出される処理を定義するメソッド
Numeric#seconds,Numeric#minutes,Numeric#hour,Numeric#day, | 時間指定
configure                                                 | Clockworkの初期状態に呼び出され、Clockworkのマルチスレッド/ロギング/タイムゾーンに関する設定を定義する

## サンプルソース
```ruby
require 'clockwork'

module Clockwork
  handler do |job|
    puts "Running #{job}"
  end

  every(10.seconds, 'frequent.job')
  every(3.minutes, 'less.frequent.job')
  every(1.hour, 'hourly.job')

  every(1.day, 'midnight.job', :at => '00:00')
end
```

## 基本
- handlerにジョブ実行時の処理を定義する。
- everyでジョブを定義する。
- everyでジョブの実行タイミングを指定する際にNumeric#secounds, Numeric#minutes, Numeric#hour, Numeric#dayを使う。
- moduke Clockworkはhandler,everyを呼びやすくするためのもので、特別な事情が泣けrばそのままで良い。

## 実行
### フォアグランド実行
```
clockwork clock.rb
```
puts等を記述していればその処理も表示される。Ctrl+Cで終了。

### バックグラウンド実行
- フォアグランドのままではターミナルを閉じると定期実行も終了してしまうので、 `clockworkd -c hoge.rb start --log`
- clockworkdコマンドを使うにはdaemonsが必要なので先に `gem install daemons`しておく。
- オプションに`--log`を付けてスタートすることで、コマンドを実行したカレントディレクトリにtmpフォルダが作成されログが吐かれる。
- 終了は`cloclworkd -c hoge.rb stop`

## 参考URL
- [公式リファレンス](https://github.com/tomykaira/clockwork)
- [clockwork について](http://www.ownway.info/Ruby/clockwork/about)
- [Rubyでcronのような定期実行を実現するclockwork](https://blog.piyo.tech/posts/2014-02-17-222721/)
- [clockworkでRubyスクリプトを定期実行しよう](http://qiita.com/giiko_/items/7e7c91a50f66bb351c89)
- [clockwork gemの使い方](http://d.hatena.ne.jp/riocampos+tech/20130625/p1)

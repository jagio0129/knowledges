Rubyでマルチスレッド
====

Rubyでマルチスレッディングする際の方法や注意点

## マルチプロセス・マルチスレッドの違い

![](https://imokuri123.com/assets/static/2013-12-process-thread_mini.679a5b3.56ee6d6f4dc526bebcca1a4d05332e3a.jpg)

- マルチプロセスはメモリを完全にコピーする
- マルチスレッドは同一プロセス内に存在し、メモリを共有する

## スレッドセーフ

- マルチスレッドではメモリを共有して並列処理が行われる。
  - 同じメモリ領域に書き込みが発生するとデータの整合性が取れなくなってしまう。

- マルチスレッドにおいて、互いのスレッドが悪影響を及ぼさないような実装を**スレッドセーフ**な実装と呼ぶ

## Rubyのスレッドセーフ担保の仕組み GVL

- Ruby(MRI)ではスレッドセーフを担保するGVL(Global VM Lock)
  - Rubyではマルチスレッド処理を実装してもGVLによって同時実行数は1になる
- I/O待ちが発生するとGVLが開放され複数のスレッドが同時に動くことになる

- 関連記事
  - [Rubyのスケール時にGVLの特性を効果的に活用する（翻訳）](https://techracho.bpsinc.jp/hachi8833/2020_05_27/92042)

## gem Parallel

- Rubyで配列を実現するのであれば、Kernel#fork, ThreadやFiberクラスを使うよりParallelを利用するほうが簡単
- [公式](https://github.com/grosser/parallel)
- [Rubyで並列処理を行うparallel gemの使い方と勘所](https://www.xmisao.com/2018/07/22/how-to-use-ruby-parallel-gem.html)はParallelの使い方、注意点など有益な情報がたくさん乗っている
- Parallelを利用する際に注意するのは以下
  - 例外処理
  - ActiveRecord
  - スレッドアンセーフになりやすいのでなるべく短く簡潔に
  - 引数でスレッドかプロセスか明示する際にキーワード引数をtypoするとマルチプロセス(数はCPUコア数に応じて自動設定)されるので注意。

### 他の並列処理方法

- Sidekiq
  - RedisにJobをキューイングしSidekiqのプロセスが随時ワーカーを起動してJobを取りに行く
  - スレッドモデルを採用しているのでIO待ちが発生するような処理に効果を発揮する
    - アプリのPush通知を送る
    - メールを送る
    - DBのレコードを更新
- Resque
  - Sidekiqと似ているがこちらはプロセスモデル。
  - メモリ余分に食いがち

- 並列処理に関しては以下の記事が参考になる
  - [Rubyで並列処理をやっていく #AdventCalendar](https://ainame.hateblo.jp/entry/2016/12/01/103708)

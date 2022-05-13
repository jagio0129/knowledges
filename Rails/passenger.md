Spawn methods explained - for Ruby developers
===

> [原文](https://www.phusionpassenger.com/library/indepth/ruby/spawn_methods/#spawn-methods-explained-for-ruby-developers)

Passenger は、HTTP プロキシおよびプロセスマネージャです。アプリケーションプロセスを生成し、入ってきた HTTP リクエストをそのうちの 1 つに転送します。

これは単純に聞こえるかもしれませんが、アプリケーションプロセスを生成する方法は1つだけではありません。Passengerは、いわゆる「スマートスポーン」をサポートしています（現在はRubyアプリケーションのみ）。これは、オペレーティングシステムの仮想メモリのコピーオンライト・セマンティクスを利用して、全体のメモリ使用量を削減するものです。

この記事では、Passenger が利用できるさまざまなスポーニング方法と、それらの比較、およびスマート・スポーニングの動作について説明します。

Rubyを使用していることを前提としています。Rubyを使用していない場合、Passengerは常にダイレクトスポーニングを使用します。

## The most straightforward and traditional way: direct spawning

Passenger は新しい Ruby プロセスを作成し、アプリケーションコードとともに Web フレームワーク (例: Rails) をロードすることができます。このプロセスはその後、リクエスト処理のメインループに入ります。

これはプロセスを生成する最も簡単な方法で、各プロセスはアプリケーションコードと Web フレームワークの完全なコピーをメモリ内に含んでいます。

これはうまく機能しますが、各プロセスがウェブフレームワークと同様にアプリケーションコードの独自のプライベートコピーを持っているため、効率的ではありません。これは、メモリと起動時間を浪費します。

![direct_spwan](https://www.phusionpassenger.com/library/indepth/spawn_methods/direct_spawning-7fd82545.png)

> アプリケーションプロセスとダイレクトスポーン 各プロセスはアプリケーションコードとWebフレームワークコードの独自のプライベートコピーを持ちます。

## The smart spawning method

アプリケーションとウェブフレームワークのコードが占有するメモリを、異なるプロセスで共有させることが可能である。これは、最新のオペレーティングシステムにおける仮想メモリシステムの、いわゆるコピーオンライトのセマンティクスを利用することで実現されています。副次的な効果として、起動時間も短縮されます。この技術は、Passengerのスマートスポーンメソッドで利用されています。

Rubyアプリの場合、spawnメソッドのデフォルトはsmartです。この機能は、Unicorn の preload_app true 機能に似ている。

### How it works

スマートスポーンメソッドを使用する場合、Passengerはまず、いわゆる「プリローダー」プロセスを作成します。このプロセスは、config.ru ファイルを読み込むことで、Web フレームワークとともにアプリケーション全体を読み込みます。プリローダプロセスはリクエスト処理には関与しません。

そして、Passenger が新しいアプリケーションプロセスを必要とするときはいつでも、プリローダーに作成するように指示します。そして、プリローダは (fork() システムコールを使って) 子プロセスを生成します。オペレーティングシステムは、この子プロセスが自分自身の正確な仮想コピーであることを保証しています。したがって、この子プロセスは、すでにアプリケーションコードとWebフレームワークのコードをメモリ内に持っています。

このようなプロセスの作成は非常に高速で、Rubyアプリケーションとフレームワークを一から読み込むよりも10倍ほど速くなります。さらに、OSは「コピーオンライト」と呼ばれる最適化も適用します。これは、子プロセスが変更していないメモリは、すべて親プロセスと共有されることを意味します。

![smart_spawn](https://www.phusionpassenger.com/library/indepth/spawn_methods/smart_spawning-45966b9d.png)

> アプリケーションプロセスとスマートスポーンメソッド プリローダーと同様に、すべてのプロセスは、同じアプリケーションコードとWebフレームワークのコードを共有します。

しかし、Ruby がこの copy-on-write 最適化を利用できるのは、ガベージコレクタが copy-on-write にフレンドリーである場合のみです。これは Ruby 2.0.0 以降のバージョンに限られ、それ以前のバージョンではコピーオンライトの最適化を利用することはできません。

プリローダプロセスには、アプリケーションプロセスと同じようにアイドルタイムアウトがあることに注意してください。プリローダは、しばらく何もするように指示されていない場合、メモリを節約するためにシャットダウンされます。このアイドルタイムアウトは、設定可能です。

### Summary of benefits

PassengerがRails 4.2.0を使うアプリケーション用のプロセスを必要とするとします。

スマートスポーン方式が使われ、このアプリケーション用のプリローダがすでに実行されている場合、プロセス生成時間は直接スポーンするよりも約10倍速くなります。このプロセスは、アプリケーションとRailsフレームワークのコードメモリもプリローダと共有し、同じプリローダによって生成された他のプロセスも共有します。

実際には（Ruby 2.0.0以降を使用していると仮定して）、スマート・スポーン方式は平均で約33%のメモリを節約します。

もちろん、スマート・スポーンに注意点がないわけではありません。しかし、注意点を理解すれば、簡単にスマート・スポーンのメリットを享受することができます。

## Smart spawning caveats

### Ruby >= 2.0.0 required

Ruby は、ガベージコレクタがコピーオンライトに対応している場合のみ、このコピーオンライトの最適化を利用することができます。これは Ruby 2.0.0 以降のバージョンで、それ以前のバージョンでは copy-on-write の最適化を利用することができません。

スマート・スポーンは以前の Ruby のバージョンでも動作します。エラーは発生しません。古いバージョンの Ruby でスマート・スポーンを使っても、スポーンにかかる時間は短縮されます。ただし、メモリ使用量は減りません。

### Unintentional file descriptor sharing

アプリケーションプロセスはプリローダプロセスからフォークして作成されるため、プリローダプロセスによって開かれたすべてのファイルディスクリプタを共有することになります。(これは、Unix の 'fork()' システムコールのセマンティクスの一部です。慣れないうちはググった方がいいかもしれません)。ファイルディスクリプタは、オープンされたファイル、オープンされたソケット接続、パイプなどのハンドルである。このようなファイルディスクリプタに対して、異なるアプリケーションプロセスが同時に書き込みを行うと、書き込みコールが[インターリーブ](https://e-words.jp/w/%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%BC%E3%83%AA%E3%83%BC%E3%83%96.html)されることになり、潜在的に問題が発生する可能性があります。

この問題は、意図せずに共有されているソケット接続によく発生します。この問題は、Passenger が新しいアプリケーションプロセスを作成するときに接続を閉じて再確立することで解決できます。Passenger はそのために PhusionPassenger.on_event(:starting_worker_process) という API コールを提供しています（Smart spawning hooks も参照してください）。そのため、config.ru に以下のコードを挿入することができます。

```ruby
if defined?(PhusionPassenger)
  PhusionPassenger.on_event(:starting_worker_process) do |forked|
    if forked
      # We're in smart spawning mode.
      ... code to reestablish socket connections here ...
    else
      # We're in direct spawning mode. We don't need to do anything.
    end
  end
end
```

なお、Passengerは新しいアプリケーションプロセスを作成する際に、自動的にActiveRecordのプライマリデータベース接続を再確立するため、通常、スマートスポーンモードを使用してもデータベースの問題は発生しません。

#### Example 1: Memcached connection sharing (harmful)

Railsアプリケーションがあり、そのconfig/application.rbでMemcachedサーバに接続するコードを呼び出しているとします。config/application.rbはconfig.ruから読み込まれるため、下図のようにプリローダーがMemcachedサーバへのソケット接続（ファイルディスクリプタ）を持つようになります。

```
 +--------------------+
 | Preloader          |-----------[Memcached server]
 +--------------------+
 ```

 次に、PassengerがHTTPリクエストを処理するための新しいアプリケーションプロセスを作成するとします。その結果は次のようになります。

```
 +--------------------+
 | Preloader          |------+----[Memcached server]
 +--------------------+      |
                             |
 +--------------------+      |
 | App process 1      |-----/
 +--------------------+
```

fork() はプロセスの（仮想的な）完全なコピーを作成するので、そのすべてのファイルディスクリプタもコピーされます。ここでわかるのは、Preloader と App process 1 の両方が Memcached への同じコネクションを共有しているということです。

ここで、あなたのサイトに突然大きなトラフィックが発生し、Passengerが別のプロセスを生成することにしたとします。その際、PassengerはPreloaderをフォークします。その結果、以下のようになります。

```
 +--------------------+
 | Preloader          |------+----[Memcached server]
 +--------------------+      |
                             |
 +--------------------+      |
 | App process 1      |-----/|
 +--------------------+      |
                             |
 +--------------------+      |
 | App process 2      |-----/
 +--------------------+
 ```

ご覧のように、Appプロセス1とAppプロセス2は同じMemcachedコネクションを持っています。

例えば、Joe と Jane というユーザーが同時にあなたの Web サイトを訪れたとします。Joe のリクエストはアプリプロセス 1 で処理され、Jane のリクエストはアプリプロセス 2 で処理されます。どちらのアプリケーション・プロセスも、Memcached から何かを取得したいと考えています。そのために、両方のハンドラが Memcached に `FETCH`コマンドを送信する必要があるとします。

しかし、Appプロセス1が`FE`を送信しただけで、コンテキストスイッチが発生し、Appプロセス2もMemcachedに`FETCH`コマンドを送信するようになったとします。Appプロセス2が`F`という1バイトの送信に成功した場合、Memcachedは`FEF`で始まるコマンドを受信することになり、認識できないコマンドになります。つまり、両方のハンドラからのデータが混在してしまうのです。そのため、Memcachedはこれをエラーとして処理せざるを得なくなります。

この問題は、フォークした後に Memcached への接続を再確立することで解決できます。

```
 +--------------------+
 | Preloader          |------+----[Memcached server]
 +--------------------+      |                   |
                             |                   |
 +--------------------+      |                   |
 | App process 1      |-----/|                   |
 +--------------------+      |                   |  <--- created this
                             X                   |       new
                                                 |       connection
                             X <-- closed this   |
 +--------------------+      |     old           |
 | App process 2      |-----/      connection    |
 +--------------------+                          |
           |                                     |
           +-------------------------------------+
```

アプリプロセス2は、Memcachedと独自の独立した通信チャネルを持つようになりました。config.ruのコードは以下のような感じです。

```ruby
if defined?(PhusionPassenger)
  PhusionPassenger.on_event(:starting_worker_process) do |forked|
    if forked
      # We're in smart spawning mode.
      reestablish_connection_to_memcached
    else
      # We're in direct spawning mode. We don't need to do anything.
    end
  end
end
```

#### Example 2: log file sharing (not harmful)

また、意図しないファイルディスクリプタの共有が有害でない場合もあります。そのようなケースの1つが、ログファイルのファイルディスクリプタの共有です。2つのプロセスが同時にログ・ファイルに書き込んだとしても、最悪の場合、ログ・ファイルのデータがインターリーブされることになります。

ログファイルに書き込まれたデータが決してインターリーブされないことを保証するためには、ファイルロックなどのプロセス間同期メカニズムによって書き込みアクセスを同期させる必要があります。Memcached の例で行ったように、ログファイルを再度開くことは役に立ちません。

### The need to revive threads

fork()システムコールのセマンティクスのもう一つの部分は、fork呼び出しの後にスレッドが消えるという事実です。そのため、environment.rb でスレッドを作成した場合、そのスレッドは新しく作成されたアプリケーションプロセスではもう実行されません。新しいプロセスが作成されたときに、それらを復活させる必要があります。Passenger が提供する :starting_worker_process イベントを利用して、以下のようにします。

```ruby
if defined?(PhusionPassenger)
  PhusionPassenger.on_event(:starting_worker_process) do |forked|
    if forked
      # We're in smart spawning mode.
      ... code to revive threads here ...
    else
      # We're in direct spawning mode. We don't need to do anything.
    end
  end
end
```

## Smart spawning hooks

次のようにして、スマート・スポーン機構にフックすることができます。

プリローダーが子プロセスをフォークする前にコードを実行したい場合、そのコードを config.ru で呼び出します (あるいは config/application.rb などの config.ru の読み込み中に呼び出されるコードから呼び出します)。

プリローダーが子プロセスをフォークした後に、その子プロセスのコンテキストでコードを実行させたい場合は、Passenger が提供する :starting_worker_process フックを使用します。以下のコードを config.ru (または config/application.rb など config.ru を読み込んでいる間に呼び出されるコード) に記述してください。

```ruby
if defined?(PhusionPassenger)
  PhusionPassenger.on_event(:starting_worker_process) do |forked|
    if forked
      # We're in smart spawning mode.
      ... your code here ...
    else
      # We're in direct spawning mode. We don't need to do anything.
    end
  end
end
```

forked 引数は Passenger がスマートスポーンモードでアプリをスポーンした場合のみ true になります。

このフックはUnicornの`after_fork`フックに匹敵します。

## Smart spawning FAQ

### ある問題に遭遇しています。スマートスポーンが原因なのでしょうか？

スマート・スポーンを無効にしてみてください（つまりダイレクト・スポーンを使っている）。スマート・スポーンを無効にしても問題が解決しない場合は、スマート・スポーンとは無関係であることは100%確実です。

スマートスポーンを無効にすることで解決した場合は、スマートスポーンに関する注意事項をご確認ください。

スマートスポーンは、以下の手順で無効にすることができます。

| module     | settings                       |
| ---------- | ------------------------------ |
| Nginx      | passenger_spawn_method direct; |
| Apache     | PassengerSpawnMethod direct    |
| Standalone | --spawn-method direct          |

### プリローダーが子プロセスをフォークした後、データベース接続を再確立するのは私の責任でしょうか？

依存します。Passengerは自動的にActiveRecordのプライマリデータベース接続を再確立します。Railsの大半は、他のデータベースライブラリではなくActiveRecordのみを使用し、1つのデータベース接続のみを使用します。

もしあなたのアプリケーションがActiveRecord以外のデータベースライブラリを使用していたり、ActiveRecordを複数のデータベース接続で使用している場合は、フォーク後にすべての接続を再確立する責任があります。スマート・スポーンフックを使ってください。

## 参考サイト
- [Multiprocessing: forkとspawnの違いを理解する](https://itsuka-naritai.com/2021/04/18/multiprocessing-fork%E3%81%A8spawn%E3%81%AE%E9%81%95%E3%81%84%E3%82%92%E7%90%86%E8%A7%A3%E3%81%99%E3%82%8B/)

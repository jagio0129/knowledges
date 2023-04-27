Phusion Passenger Design and Architecture
===

> [原文](https://www.phusionpassenger.com/documentation/Design%20and%20Architecture.html)

このガイドでは、Phusion Passengerの設計とアーキテクチャについて詳しく説明しています。このガイドによって、コントリビューターがPhusion Passengerのコードベースを素早く見つけることができるようになることを期待しています。

このガイドは、Phusion Passengerの使い方やNginxやApacheに慣れていること、[コントリビューターガイド](https://github.com/phusion/passenger/blob/master/CONTRIBUTING.md)や[デベロッパークイックスタート](https://github.com/phusion/passenger/blob/master/doc/DeveloperQuickstart.md)を読んでいることを前提にしています。

## 1. Introduction
### 1.1. Web application models and the role of the application server

Phusion Passengerについて説明する前に、WebアプリケーションをWebサーバーに接続しようとする人の視点から、典型的なWebアプリケーションがどのように動作するかを理解することが重要です。

典型的な孤立したWebアプリケーションは、あるI/OチャネルからHTTPリクエストを受け取り、それを内部で処理し、HTTPレスポンスを出力し、クライアントに送り返します。これは、アプリケーションが終了するように命令されるまで、ループで実行されます。これは、WebアプリケーションがHTTPを直接話すことを必ずしも意味するわけではありません。

![](https://www.phusionpassenger.com/documentation/images/typical_isolated_web_application.png)

Webアプリケーションには、HTTPプロトコルで直接アクセスできるものもあれば、そうでないものもあります。これは、Webアプリケーションが構築されている言語とフレームワークに依存します。例えば、Ruby（Rack/Rails）やPython（WSGI）のWebアプリケーションは、通常、HTTPプロトコルで直接アクセスできません。一方、Node.jsのウェブアプリケーションは、HTTPプロトコルでアクセスできる傾向があります。この理由は歴史的なものですが、このガイドの範囲外です。

#### 1.1.1. Common models

ここでは、一般的に使用されているモデルを紹介します。

第一に、Webアプリケーションは、アプリケーションサーバーに含まれています。このアプリケーションサーバーは、複数のウェブアプリケーションを格納できる場合もあれば、そうでない場合もあります。管理者は、アプリケーションサーバーを何らかのプロトコルで Web サーバーに接続します。このプロトコルは、HTTP、FastCGI、SCGI、AJP などの種類があります。Webサーバーはアプリケーションサーバーにリクエストを送信（転送）し、アプリケーションサーバーはWebアプリケーションが理解できるフォーマットで、正しいWebアプリケーションにリクエストを送信します。逆に、Webアプリケーションから出力されたHTTPレスポンスは、アプリケーションサーバーに送られ、アプリケーションサーバーからWebサーバーに送られ、最終的にHTTPクライアントに送られます。

代表的なモデルに以下のような者があります。

- Tomcatアプリケーションサーバーに含まれるJ2EEアプリケーションは、Apache Webサーバーの後ろにリバースプロキシされます。Tomcatは、1つのTomcatインスタンスに複数のWebアプリケーションを含めることができます。
- Phusion Passenger以外のほとんどのRubyアプリケーションサーバー（Thin、Unicorn、Goliathなど）。これらのアプリケーションサーバーは、1インスタンスにつき1つのRubyウェブアプリケーションしか格納できません。これらのサーバーは、Webアプリケーションを独自のプロセスにロードし、Webサーバー（Apache、Nginx）の背後にリバースプロキシで配置されます。
- Python（WSGI）アプリケーションサーバーであるGreen Unicornのリバースプロキシ設定。
- FPM（FastCGIプロセスマネージャ）により、NginxリバースプロキシでPHPウェブアプリケーションが生成されます。

第二に、Webアプリケーションは、Webサーバーに直接含まれています。この場合、Webサーバーはアプリケーションサーバーのように動作します。代表的な例としては、以下のようなものがあります。

- mod_php を通して Apache 上で動作する PHP Web アプリケーション。
- mod_uwsgi または mod_python を通して Apache 上で動作する Python (WSGI) ウェブアプリケーション。

これは、WebアプリケーションがWebサーバーと同じプロセス内で実行されることを必ずしも意味しないことに注意してください：それは単にWebサーバーがアプリケーションを管理することを意味します。mod_phpの場合、PHPはApacheのワーカープロセスの中で直接実行されますが、mod_uwsgiの場合、Pythonプロセスはアウトオブプロセスで実行するように設定することができます。

Phusion Passenger for ApacheとPhusion Passenger for Nginxはこのモデルを実装し、Webサーバプロセスの外側でアプリケーションを実行します。

第3に、WebアプリケーションはWebサーバーであり、HTTPリクエストを直接受け付けることができます。このモデルの例は以下のような者があります。

- ほぼすべてのNode.jsとMeteor JSのWebアプリケーション。
- スタンドアロンサーバーで動作する、バグトラッキングソフトウェア「Trac」。

ほとんどのセットアップでは、管理者はHTTPリクエストを直接受け付けるのではなく、ApacheやNginxなどの実際のWebサーバーの後ろにリバースプロキシ構成を配置します。

Phusion Passenger Standaloneはこのモデルを実装しています。しかし、Phusion Passenger Standaloneは内部でNginxを使用しているため、インターネットに直接公開することができます。

第4に、Webアプリケーションは直接HTTPを話さず、何らかの通信アダプタを介してWebサーバに直接接続する。CGI、FastCGI、SCGIなどがその代表的な例です。

上記４つのモデルは、PHP、Django、J2EE、ASP.NET、Ruby on Rails、その他をベースとした、ほとんどすべてのWebアプリケーションの動作方法をカバーしています。これらのモデルはすべて同じ機能を提供します。つまり、どのモデルも別のモデルができないことはできません。もし、ウェブサーバー、アプリケーションサーバー、ウェブアプリケーションなどの組み合わせを単一のエンティティ、いわばブラックボックスとみなすなら、これらのモデルはすべて最初の図で説明したものと同じであることに、批判的な読者は気付くでしょう。

また、これらのモデルは、特定のI/O処理の実装を強制するものではないことに注意する必要があります。ウェブサーバ、アプリケーションサーバ、ウェブアプリケーションなどは、I/Oをシリアルに（つまり一度に1つのリクエスト）処理することも、1つのスレッドでI/Oを多重化することも（例えば、select（2）やpoll（2）を使用）、複数のスレッドや複数のプロセスでI/Oを処理することも可能である。それは、実装に依存する。

もちろん、多くのバリエーションが可能である。例えば、ロードバランサーを使用することも可能です。しかし、それはこのドキュメントの範囲外である。

#### 1.1.2. The rationale behind reverse proxying

このように、管理者は、WebアプリケーションやそのアプリケーションサーバーがすでにHTTPを話す場合でも、リバースプロキシ設定で実際のWebサーバーの背後に置くことがよくあります。これは、適切で安全な方法でHTTPを実装するには、単にプロトコルを話すだけでは不十分なためです。公共のインターネットは、クライアントが任意のデータを送信でき、任意のI/Oパターンを示すことができる、敵対的な環境である。I/O処理を適切に実装しなければ、パーサーの脆弱性やサービス拒否攻撃にさらされる可能性があります。

ApacheやNginxのようなウェブサーバーは、すでに世界トップクラスのI/Oや接続処理のコードを実装しており、車輪の再発明をするのは無駄なことです。結局のところ、リバースプロキシの設定にアプリケーションを置くことで、システム全体がより堅牢になり、より安全になることが多いのです。これが、リバースプロキシがグッドプラクティスと言われる所以です。

典型的な問題は、遅いクライアントに対処することです。これらのクライアントは、HTTPリクエストをゆっくり送信し、HTTPレスポンスをゆっくり読み取るため、おそらく作業を完了するのに何秒もかかるでしょう。HTTPリクエストを読み、処理し、HTTPレスポンスを送るというループを繰り返す素朴なシングルスレッドHTTPサーバーの実装では、I/Oを待つ時間が長くなり、実際の作業にほとんど時間をかけられなくなってしまうかもしれません。さらに悪いことに、クライアントが悪意を持っていて、ソケットを開いたままにしてHTTPレスポンスを読まなかったとすると、サーバーはクライアントを永遠に待ち続けることになり、それ以上のリクエストを処理することができなくなります。この原則に基づく現実の攻撃が、[Slowloris](https://www.f5.com/ja_jp/glossary/slowloris)です。

**An example of a naive HTTP server implementation**

```ruby
while true
    client = accept_next_client()
    request = read_http_request(client)
    response = process_request(request)
    send_http_response(client, response)
end
```

この問題を解決する方法はたくさんあります。クライアントごとに1つのスレッドを使う、I/Oタイムアウトを実装する、イベント型I/Oアーキテクチャを使う、専用のI/Oスレッドやバッファのリクエストとレスポンスを処理する、などなど。ポイントは、これらすべてを適切に実装することは非自明であるということです。アプリケーションサーバーごとにこれらを何度も再実装するのではなく、実際のWebサーバーにすべての詳細を任せ、アプリケーションサーバーとWebアプリケーションは、それぞれの得意分野であるコアビジネスロジックに専念するのがよいでしょう。

### 1.2. Phusion Passenger architecture overview

![](https://www.phusionpassenger.com/documentation/images/passenger_architecture_overview.png)

Phusion Passengerは、単一のモノリシックな存在ではありません。その代わり、複数のコンポーネントとプロセスで構成され、連携して動作します。Phusion Passengerがこのように分割されている理由の一つは、技術的に必要だからです（他に実装する方法がない）。しかし、もう一つの理由は、安定性と堅牢性です。個々のコンポーネントはクラッシュする可能性があり、互いに独立して再起動させることができます。もし、すべてを単一のプロセス内に置くとしたら、クラッシュはPhusion Passengerのすべてをダウンさせることになります。

したがって、Passengerのコアがクラッシュしても、アプリケーションプロセスがクラッシュしても、Webサーバーの安定性に影響を与えることなく、どちらも再起動することができるのです。

#### 1.2.1. Web server module

HTTPクライアントがリクエストを送信すると、Webサーバー（NginxまたはApache）がそれを受信する。ApacheもNginxも、モジュールで拡張することができます。Phusion Passengerはそのようなモジュールを提供します。このモジュールはNginx/Apacheにロードされます。このモジュールは、リクエストがPhusion Passengerが提供するウェブアプリケーションによって処理されるべきかどうかをチェックし、もしそうなら、リクエストをPassengerコアに転送します。この転送の際に使用される内部ワイヤプロトコルは、SCGIを改良したものです。

NginxモジュールとApacheモジュールは、全く異なるコードベースを持っています。コードベースはそれぞれ ext/nginx と ext/apache2 にあります。両モジュールは、ほとんどのロジックをPassengerのコアに委託しており、また共通のライブラリ（ext/common）を利用しているため、比較的小規模です。これにより、多くのことを二度書きすることなく、NginxとApacheの両方をサポートすることができます。

#### 1.2.2. Passenger core

Passengerのコアは、ほとんどの処理が行われる場所です。コアは、現在どのアプリケーションプロセスが存在するかを追跡し、ロードバランシングルールを使用して、リクエストをどのプロセスに転送すべきかを決定します。また、コアはアプリケーションの生成も行います。より多くのアプリケーションプロセスを持つことが必要または有益であると判断した場合、それを実現します。コアは、ユーザが設定した最大値を超えるプロセスを生成することはありません。

また、コアは監視と統計収集の機能も備えています。アプリケーションのメモリ使用量、処理したリクエストの数などを常に記録しています。この情報は、後で管理ツールから照会することができます。また、アプリケーションのプロセスがクラッシュした場合、コアはそれを再起動させます。

コアは、システムで最も大きく、最も複雑な部分ですが、それ自体がいくつかの小さなサブシステムで構成されています。コアアーキテクチャの章の大半は、コアの説明に費やされている。

#### 1.2.3. UstRouter

このコアは、UstRouterと連携しています。この後者は(This latter isのいい訳が見つからなかった)監視用ウェブサービスであるUnion Stationにデータを送信する役割を担っています。もし、Phusion PassengerにUnion Stationにデータを送信するよう明示的に指示しなかった場合、UstRouterはアイドル状態になり、リソースを消費することはない。

#### 1.2.4. Watchdog

coreとUstRouterは複雑なロジックを含んでいるので、それらをクラッシュさせるバグを含む可能性があります。そこで、安全対策として、両者はウォッチドッグによって監視されています。どちらかがクラッシュした場合、ウォッチドッグによって再起動されます。このような設定により、何があってもシステムが稼働し続けることを保証しています。

ウォッチドッグがクラッシュしたらどうするんだろう？ウォッチドッグは別のウォッチドッグに監視されるべきではないだろうか？その可能性も考えましたが、ウォッチドッグは非常にシンプルで、2012年以降、ウォッチドッグがクラッシュしたという報告は一度もありませんし、クラッシュさせることもできていません。そこで、コードベースをできるだけシンプルに保つために、複数のWatchdogを導入しないことにしました。

#### 1.2.5. Command line tools

最後に、Phusion Passengerをサポートするコマンドラインツールの数々を紹介します。インストーラ - passenger-install-*-module - は、Phusion Passengerのインストールを担当します。passenger-status や passenger-memory-stats などの管理ツールもあります。その他にもたくさんあります。これらのツールの中には、エージェントの1つと通信するものもあります。例えば、passenger-statusは、コアが収集した情報をコアに問い合わせます。この通信がどのように行われるかは、「5. Instance state and communication」で説明されています。

#### 1.2.6. Passenger Standalone

Phusion Passenger Standaloneがダイアグラムに含まれていないことにお気づきかもしれません。では、どのようにしてこのアーキテクチャに組み込むのでしょうか？Phusion Passenger Standaloneは、実はNginx用のPhusion Passengerなのです。passenger startコマンドは、Phusion Passenger NginxモジュールがロードされたNginx Webサーバー（WebHelperと呼んでいます）をセットアップするだけです。

### 1.3. Build system and source tree

Phusion Passengerは、そのほとんどがC++とRubyで書かれています。Webサーバーモジュール、core、UstRouter、WatchdogはC++で書かれています。ほとんどのコマンドラインツールはRubyで書かれています。各コンポーネントはこちらでご覧いただけます：

- Webサーバモジュールは、ext/apache2およびext/nginxに含まれています。
- Passengerコア、UstRouter、Watchdogはext/common/agentに収録されています。
- コマンドラインツールはbinに、そのコードの一部はlibに含まれています。

より詳しい情報は、コントリビューターガイドに記載されています。このガイドでは、Phusion Passengerのコンパイル方法についても説明しています。

## 2. Initialization

![](https://www.phusionpassenger.com/documentation/images/startup_sequence.png)

Phusion Passengerは以下のように初期化されます。

** 1 **

まず、Webサーバーを起動することから始めます。例えば、sudo service apache2 startやsudo service nginx startを実行することで行います。あるいは、OSが自動的にWebサーバを起動するように設定されている場合もあり、その場合はユーザは何もする必要がありません。Phusion Passenger Standaloneの場合、ユーザーはpassenger startを実行し、それによってNginxが起動します。

** 2 **

Nginx/Apache内のPhusion Passengerモジュールは、Watchdogの起動を進めます。これは、以下で実装されています。

- ext/nginx/ngx_http_passenger_module.c, function start_watchdog().
- ext/apache2/Hooks.cpp の Hooks クラスのコンストラクタ内。
- ext/common/AgentsStarter.h と AgentsStarter.cpp です。ウォッチドッグの起動に関わるロジックのほとんどは、このファイルにあります。

** 3 **

Watchdogは最初に "インスタンスディレクトリ"(5章参照)を初期化します。これは、このPhusion Passengerインスタンスのライフタイム中に使用されるファイルを含む一時ディレクトリです。例えば、このディレクトリにはUnixドメインソケットファイルが含まれており、異なるPhusion Passengerプロセスが互いに通信できるようになっています。Watchdogは、ext/common/agent/Watchdog/Main.cppで実装されています。

** 4 **

Watchdogは、PassengerコアとUstRouterを同時に起動します。それぞれが独自の初期化を行う。

** 5 ** 

コアの初期化が終わると、ウォッチドッグに「終わったよ」というメッセージを送り返します。UstRouterも似たようなことをする。Watchdog が両方の確認メッセージを受信したら、初期化を終了する。Watchdog は、エージェントの1つが確認メッセージを送らずに終了したことに気づいたら、エラー状態になる。

** 6 ** 

ウォッチドッグは、Nginx/Apache内で動作しているPhusion Passengerモジュールに、起動の成功を報告します。あるいは、初期化が成功しなかった場合、Watchdogはエラーを報告します。Nginx/Apache内のPhusion Passengerモジュールは、そのエラーをログに記録します。

初期化後、Phusion Passengerはリクエストを受信し、処理する準備が整います。

## 3. Passenger core architecture

![](https://www.phusionpassenger.com/documentation/images/passenger_core_architecture.png)

Passengerコアは、2つのサブシステムで構成されています。ひとつは、リクエスト処理サブシステム。もうひとつは、プロセス管理の大部分を行うApplicationPoolサブシステムです。また、コアは多くのサポートライブラリを使用しています。最大のサードパーティサポートライブラリを図に示します。これ以外にも多くの内部サポート・ライブラリーが使用されていますが、図では省略しています。これらの内部サポート・ライブラリは、ext/common/Utilsディレクトリにあります。

### 3.1. Request handling

リクエストはまずウェブサーバから受け取られることを思い出してください。Web サーバはリクエストを SCGI フォーマットに少し修正したものにシリアライズし、コアの RequestHandler に送ります。RequestHandler はいくつかの作業を行い、最終的に通常の HTTP 応答を送り返します。WebサーバーはRequestHandlerの応答を解析し、元のHTTPクライアントに応答を送信します。

RequestHandler は、Unix ドメイン ソケット ファイルでリッスンします。このUnixドメインソケットファイルはrequestと呼ばれ、インスタンスディレクトリに配置されています。

#### 3.1.1. One client per request

Webサーバーは、リクエストごとにコアへの新しい接続を作成します。したがって、RequestHandlerの視点から見ると、そのクライアントはWebサーバーです。クライアントが接続するたびに（つまり新しいリクエストが転送されるたびに）、RequestHandlerはそのリクエストを表す新しいClientオブジェクトを作成します。すべてのリクエスト固有の状態は、Clientの中に保存されます。RequestHandlerはリクエストの処理を終えると、クライアントソケットを閉じます。

この図では、ClientはRequestHandlerと0または1個の関連性を持っていることに注意してください。これは、クライアントが切断されると、関連するRequestHandlerへのポインタがNULLに設定されるからです。クライアントへのポインタを持つバックグラウンド操作が残っている可能性があります。これらのバックグラウンド操作が終了するとすぐに、クライアントがRequestHandlerへの有効なポインタを持っているかどうかをチェックします。もしそうなら、その作業をコミットし、そうでないなら、その作業を破棄します。クライアントは、関連するすべてのバックグラウンド操作が終了すると破棄されます。

#### 3.1.2. Forwarding to the application

RequestHandlerは、このリクエストを処理するために適切なアプリケーションプロセスを選択するように、非同期にApplicationPoolサブシステムに要求します。ApplicationPoolは、適切なプロセスがあるかどうかをチェックし、ない場合は、そのプロセスをスポーンしようとします。もしかしたら、設定されたリソース制限のため、今すぐにはスポーンできないかもしれないので、待つ必要がある。いずれにせよ、ApplicationPoolはすべての厄介な詳細と帳簿を管理し、最終的にSessionオブジェクトか例外でRequestHandlerに返事を返します。

Sessionオブジェクトは、特定のアプリケーションプロセスとの1つのリクエスト/レスポンスサイクルを表します。RequestHandlerは、このSessionオブジェクトの情報を使って、そのプロセスとの接続を確立し、アプリケーションが好み、RequestHandlerがサポートするプロトコルを使って、リクエストを転送します。プロセスは作業を実行し、HTTPレスポンスで返信します。RequestHandlerは、応答を解析して後処理し、Webサーバーに応答を返送します。

ApplicationPoolが例外で応答した場合、RequestHandlerはエラー応答を送り返す。

#### 3.1.3. I/O model

RequestHandlerは、イベント型I/Oモデルを使用します。これは、RequestHandlerが、1つのプロセス内で1つのスレッドを使用して、同時に多くのクライアント（要求）を処理することを意味します。これは、OSが提供するI/Oイベント多重化機構を使用することで可能となります。そのようなメカニズムの例として、select()、poll()、epoll()、kqueue()システムコールがあります。しかし、これらのメカニズムは非常に低レベルでOS固有のものであるため、RequestHandlerはその違いを抽象化してより高レベルのAPIを提供する2つのライブラリ、libevとlibuvを使用します。

イベント型I/OモデルはNginxでも使用されています。これは、1プロセスあたり1クライアントを処理するシングルスレッド型のマルチプロセスモデル（Apacheのprefork MPMで使用）や、1スレッドあたり1クライアントを処理するマルチスレッドモデル（Apacheのworker MPMで使用）とは対照的なモデルです。イベント I/O と異なる I/O モデルについては、以下のリソースで詳しく知ることができます。

- [Event-Driven I/O: A hands-on introduction](https://www.slideshare.net/marc.seeger/seeger-aysnc-io) — Marc Seeger, 2010
- [Event Driven IO And Blocking vs NonBlocking](https://stackoverflow.com/questions/5807246/event-driven-io-and-blocking-vs-nonblocking) — Stack Overflow
- [How does event driven I/O allow multiprocessing?](https://stackoverflow.com/questions/3231018/how-does-event-driven-i-o-allow-multiprocessing) — Stack Overflow
- [The C10K problem](http://www.kegel.com/c10k.html) — an overview of the different I/O models used in different servers; Dan Kegel

### 3.2. The ApplicationPool subsystem

> 文脈からAppPreloaderだと思う

ApplicationPoolサブシステムは、次のことを担当する。

- どのような処理が存在するのか把握すること
- spawn process
- リクエストを適切なプロセスへルーティングすること。これは、プロセス間のリクエストをロードバランスすることも意味する。
- プロセス（CPU使用率、メモリ使用率など）を監視します。
- リソースの制限を強制する。プロセスが多く生成されないようにする、メモリを多く使用するプロセスを確実にシャットダウンする、など。
  - この機能たぶん有料版
- オンデマンドでプロセスを再起動する（restart.txtのタイムスタンプが変更された場合など）。
- クラッシュしたプロセスを再起動させる。
- リクエストをキューイングし、同時実行を制限する。各プロセスは、処理可能な同時リクエスト数をApplicationPoolに通知します。プロセスが処理できると言った数より多くの同時リクエストが来た場合、過剰なリクエストはApplicationPoolサブシステム内でキューに入れられます。同様に、プロセスが生成されている間にリクエストが来た場合、それらのリクエストはプロセスが生成され終わるまでキューに入れられます。

サブシステムへの主なインタフェースは、asyncGet()メソッドを持つPoolクラスです。RequestHandlerは、checkoutSession()メソッドの中でpool->asyncGet(options, callback)のようなものを呼び出します。asyncGet()はSession、または例外で返事をします。

Pool クラスは、サブシステムの中核です。これは、高レベルのプロセス管理ロジックを含みますが、プロセスをスポーンする詳細などの低レベルの詳細ではありません。コードはさらに以下のクラスに分割され、それぞれのクラスはそれぞれのドメインを管理するコア・コードを含んでいます。

**Group**

アプリケーションを表します。同じアプリケーションに属する複数のプロセスを含むことができる。

**Process**

OSのプロセスを表し、あるアプリケーションのインスタンスである。プロセスには複数のサーバーソケットがあり、リクエストを受け付けることができる。プロセスクラスは、現在開いているセッションの数など、さまざまな帳簿情報を保持する。また、基礎となるOSプロセスとの通信チャネルも含まれます。プロセス・オブジェクトは、ApplicationPool Spawnerサブシステム(3.3. The Spawner subsystem)を通じて作成されます。

**Socket**

1つのサーバーソケットを表し、その上でプロセスがリクエストを待ち受ける。セッションオブジェクトは、Socketを通じて作成されます。Socketは、その特定のソケットで現在開いているセッションの数に関するブックキーピング情報を保持する。

**Session**

特定のプロセスに対する1回のリクエスト/レスポンスサイクルを表現する。生成・破棄時に、各種帳簿情報を更新する。

**Options (not shown in diagram)**

Pool::asyncGet()メソッド用の設定オブジェクトです。


この図を見ると、GroupとProcessはすべて、その含むクラスと0or1個の関連付けを持っていることがわかります。NULLの関連付けを持つオブジェクトは、無効であるとみなされ、使用されるべきではありません。このように関連付けがNULLになるのは、メモリ管理方式の詳細です。

### 3.3. The Spawner subsystem

Spawnerサブシステムは、ApplicationPool内のサブシステムである。これは、実際にアプリケーション・プロセスをスポーンし、その中に正しい情報を持つProcessオブジェクトを作成する責任があります。

Spawner インターフェイスは、すべての低レベルのプロセス・スポニング・ロジックをカプセル化します。Poolは、他のアプリケーション・プロセスを生成する必要がある場合、Spawnerを呼び出します。

Phusion Passengerが複数のスポーンメソッドをサポートしていることを思い出してください。例えば、smart spawnメソッドは、中間プリローダープロセスを通してプロセスを起動し、コピーオンライトを利用することができます。これについては、Phusion Passengerのマニュアルの[Spawn methods explained](https://www.phusionpassenger.com/library/indepth/ruby/spawn_methods/)で詳しく説明されています。各スポーンメソッドは、Spawnerインターフェイスの異なる実装に対応しています。以下の実装が利用可能です。

- DirectSpawner — implements the direct spawn method.
- SmartSpawner — implements the smart spawn method.
- DummySpawner (not shown in diagram) — only used in unit tests.

スポーン・メソッドは、OptionsオブジェクトのspawnMethodフィールドを通じてユーザーが設定可能です。スポナー実装の選択ロジックでPoolのコードが複雑になるのを避けるため、Poolが使用するSpawnerFactoryクラスも用意されています。

スポーニング・プロセスの詳細は、次章で説明されています。

## 4. Application spawning and loading

アプリケーションプロセスは、Passengerのコアプロセスからスポーンされます。プロセスの起動には、通信経路の設定、現在の作業ディレクトリの設定、環境変数の設定など、多くの準備作業が必要です。この準備作業は、Spawnerオブジェクトと、様々なサポート実行ファイルによって行われます。

準備が完了したら、アプリケーションのエントリーポイントを何らかの方法でロードする必要があります。このロードは、言語固有のローダー・プログラムによって行われます。ローダープログラムは、先に設定した通信経路でSpawnerと通信し、言語固有の環境を初期化し、サーバーを立ち上げ、Spawnerに報告します。この通信は、あるプロトコルによって行われる。

### 4.1. Preparation work

![](https://www.phusionpassenger.com/documentation/images/spawning_preparation_work.png)

#### 4.1.1. Basic setup and forking

スポーンは、Spawn()メソッドがSpawnerオブジェクト上で呼び出されることで開始されます。Spawnerは、プロセスがどのユーザーとして実行されるべきかを決定し、いくつかの通信チャネル（匿名のUnixドメインソケットのペア）をセットアップし、プロセスをフォークします。親プロセスは、子プロセスが終了するか、通信チャネル上で何らかの応答を返すまで待機する。

この通信チャネルは、（プリ）ローダーから見ると、実際にはstdin、stdout、stderrだけである！Spawnerが作成する匿名のUnixドメインソケットのペアは、子プロセスのstdin、stdout、stderrファイルディスクリプタにマッピングされます。したがって、Spawnerはstdinに書き込むことで（プレ）ローダーにデータを送り、（プレ）ローダーはstdoutまたはstderrに書き込むことでSpawnerにデータを返します。

#### 4.1.2. Loading SpawnPreparer, possibly through bash

Passengerのコアはマルチスレッド化が進んでいるため、Spawnerによってフォークされた子プロセスは、非同期・シグナルセーフの操作しか行えません。

> "プロセスは、単一のスレッドで作成されなければならない。マルチスレッドプロセスがfork()を呼び出した場合、新しいプロセスには、呼び出したスレッドのレプリカとそのアドレス空間全体、場合によってはミューテックスや他のリソースの状態も含まれていなければならない。その結果、エラーを回避するために、子プロセスは、exec関数のいずれかが呼び出されるまで、非同期シグナルセーフの操作のみを実行することができる。フォークハンドラは、fork()呼び出しにまたがってアプリケーションの不変性を維持するために、pthread_atfork()関数によって確立することができます。"
>
> fork() man page — The Open Group's POSIX specification

この意味がわからなくても心配しないでください。要は、この段階でフォークされたプロセスが安全にできることはほとんど何もない、ということです。そこで、残りの準備作業のほとんどを外部の実行ファイルであるSpawnPreparerにアウトソースします。SpawnPreparerは、安全にコードを実行できるクリーンな環境からスタートします。

SpawnPreparerを実行するために、子プロセスは以下のコマンドのいずれかを実行します。

- 対象ユーザーのシェルがbashで、passenger_load_shell_envvarsオプションがオンになっている場合
  - `bash -l -c '/path-to/SpawnPreparer /path-to-loader-or-preloader'`
  - これにより、bashはbashrcやprofileなどの起動ファイルを読み込み、その後、与えられたパラメータでSpawnPreparerを実行します。
  - なぜこのようなことをするかというと、多くのユーザーがbashrcに環境変数を設定しようとし、その環境変数がPhusion Passengerによって起動されたアプリケーションに拾われることを期待しているからです。
  - 残念ながら環境変数はそのように動作しませんが、使い勝手が良いので、私たちはこの方法をサポートします。
- それ以外の場合は、bashを使わずにSpawnPreparerが直接実行されます。
  - `/path-to/SpawnPreparer /path-to-loader-or-preloader`

path-to-loader-or-preloader の決定方法については、「4.4 The AppTypes registry」に記載されています。

#### 4.1.3. SpawnPreparer further sets up the environment

SpawnPreparerは、特定の環境変数、現在の作業ディレクトリ、およびその他のプロセス環境条件を設定する役割を担っています。SpawnPreparerが終了すると、ローダまたはプリローダを実行する。


#### 4.1.4. Executing the loader or preloader

passenger_spawn_methodがsmart（デフォルト）に設定されており、アプリケーションのプログラミング言語で利用可能なプリローダーがある場合、このステップでは言語固有のプリローダーを実行します。前の条件のいずれかが満たされていない場合（したがって、passenger_spawn_methodは自動的にdirectに強制される）、このステップは言語固有のローダを実行します。

すべての（プリ）ローダーは、ソースツリーのhelper-scriptsディレクトリに配置されています。以下に、使用される(pre)loaderの一部を紹介します：

|Language/Framework	| Loader | Preloader|
|---|---|---|
|Ruby Rack and Rails|rack-loader.rb|rack-preloader.rb|
|Python|wsgi-loader.py|-|
|Node.js and bundled Meteor|node-loader.js|-|
|Unbundled Meteor|meteor-loader.rb|-|

AppTypesレジストリは、利用可能な（プレ）ローダーのリストと、それらがどの言語に属しているかを保持しています。

ローダーが何をするかは、Loadersに記述されています。同様に、プレローダーはPreloadersで説明されています。

### 4.2. Loaders

ローダーは4段階で初期化されます。

1. まず、ハンドシェイクを行い、スポナーが通信チャネルで送信したパラメータを読み取ります。
1. そして、アプリケーションをロードします。このステージの動作は、受け取ったパラメータによってカスタマイズすることができます。
1. サーバーをセットアップし、このアプリケーションプロセスがリクエストをリッスンする。
1. 初期化が成功したかどうか、成功した場合はこのアプリケーションプロセスがリクエストを受け付けるソケットの場所をSpawnerに伝えます。

初期化されると、ローダーはメインループに入り、終了を告げるシグナルを受け取るまでリクエストを処理し続けます。

基本セットアップとフォークで説明したように、ローダーが使用する通信チャネルは、単なる古いstdinとstdoutです。どのプログラミング言語でも、これらのチャンネルからの読み書きをサポートしています。これは、ローダーを実行し、ターミナルにメッセージを入力するだけで、ローダーを簡単にテストできることも意味します。

しかし、stdoutは通常の出力を印刷するためにも使うことができます。Spawnerは、どのようにして制御メッセージと、表示すべき通常のメッセージを区別するのでしょうか？その答えは、制御メッセージは `!> ` で始まり（末尾の空白を含む）、改行で終わらなければならないからです。Spawnerはメッセージを一行ずつ読み、`!> `で始まる行を処理し、そのマーカーで始まらない行を表示する。

#### 4.2.1. Handshake
プロトコルのバージョンハンドシェイクから始まります。ローダーは「!!!> I have control 1.0」という行を印字する。次にSpawnerは「You have control 1.0」を送信し、ローダーはこれをチェックする。もしローダーがバージョンハンドシェイクが期待に沿わないことを観察した場合、エラーで中止する。

Spawnerはまた、キーと値のペアのリストを送信し、それは空の改行で終了される。空の改行を受信すると、Spawnerはアプリケーションのロードを続行する。

```
!> I have control 1.0
                            You have control 1.0
                            passenger_root: ...
                            passenger_version: 4.0.45
                            ruby_libdir: /Users/hongli/Projects/passenger/lib
                            generation_dir: /tmp/passenger.1.0.2082/generation-0
                            gupid: 1647ad4-ovJJMiPkAAt
                            connect_password: jXGaSzo8vRX5oGe2uuSv5tJsf1uX7ZgIeEH2x0nfOEa
                            app_root: /Users/hongli/Sites/rack.test
                            startup_file: config.ru
                            process_title: Passenger RackApp
                            log_level: 3
                            environment: development
                            base_uri: /
                            ...
                            (empty newline)
```

#### 4.2.2. Application loading

アプリケーションをどのようにロードするかは、プログラミング言語によって異なります。以下にいくつかの例を示します。

- Ruby Rack ローダーは、起動ファイル（デフォルトでは config.ru）を load()-ing することで行います。
- Python ローダーは imp.load_source('passenger_wsgi', 'passenger_wsgi.py') を呼び出すことで実行します。
- Node.js（およびバンドルされているMeteor）ローダーは、デフォルトではapp.jsであるスタートアップファイルをrequire()することで実行します。
- バンドルされていないMeteorローダーは、meteor runコマンドを実行することでこれを行います。

エラーが発生しない場合、ローダーはサーバーのセットアップを進めます。そうでない場合は、エラーを報告します。

#### 4.2.3. Setting up a server

ローダーは、アプリケーションがリクエストを受け付けるサーバーをセットアップします。Spawnerは、これがどのように行われるのか、このサーバーがどのように動作するのか、あるいはその同時性がどの程度なのかを気にすることはない。Spawnerが気にするのは、サーバーにどうやってコンタクトするかだけです。ですから、ローダーはこのステップで完全な自由を得ます。

セクション「リクエスト処理」とサブセクション「アプリケーションへの転送」で説明したように、RequestHandlerは、アプリケーションが好むプロトコルでアプリケーションプロセスと会話することができます。RequestHandlerは2つのプロトコルをサポートしています：

- セッションプロトコルと呼ばれるPhusion Passengerの内部プロトコル。このプロトコルはRubyローダーとPythonローダーによって使用されます。このプロトコルの説明はこのドキュメントの範囲外ですが、どのように見え、どのように振る舞うかに興味があれば、RubyとPythonのローダーのソースコードや、ext/common/Utils/MessageIO.hを研究してみてください。

- HTTPプロトコルです。このプロトコルは、Node.jsとMeteorのローダーで使用されています。新しいローダーを書く場合は、このプロトコルと、ローダーのターゲット言語で利用可能なHTTPライブラリを使用するのが最も簡単でしょう。

通常、サーバーは生成ディレクトリのbackendsサブディレクトリにあるUnixドメインのソケットファイルをリッスンするように設定されています。生成ディレクトリへのパスは、ハンドシェイク中に渡されます。しかし、サーバーはTCPソケットでリッスンすることもできます。

サーバーがセットアップされると、ローダーは準備完了を報告することができます。

#### 4.2.4. Reporting readiness

サーバーがセットアップされると、loaderは!> Readyレスポンスを返し、その後にサーバーソケットがどこでリッスンしているか、どんなプロトコルを期待しているかの情報を返します。レスポンスは `!> ` 行で終了します (末尾に空白があることに注意してください。これは必須です)。

サーバソケットのリッスン先に関する情報は、4つのタプルである：

- 名前です。これは常にメインでなければなりません。
- アドレス。Unixドメインソケットの場合、unix:/path-to-socketという形式をとります。TCPソケットの場合は、tcp:/127.0.0.1:PORTという形式をとります。
- プロトコルを指定します。sessionまたはhttp_sessionのどちらかでなければなりません。
- サーバーがサポートする同時接続の最大数。ApplicationPoolは、プロセスがこの数以上の同時リクエストを受信しないようにします。値0は、同時接続が無制限であることを意味します。

以下は、Node.jsローダーがレスポンスとして送信する内容の例です。

```
!> Ready
!> socket: main;unix:/path-to-generation-dir/backends/node.1234-5677;session;0")
!>
```

#### 4.2.5. Error reporting

いずれかのステージで何か問題が発生した場合、ローダーは2つの方法でエラーを報告することができます：

1. 通常通りエラーメッセージをstdoutに書き出し、!> Readyメッセージを表示せずに中止します。Passengerコアは、ローダーがstdoutに書き込んだ内容をすべて読み、それをエラーメッセージとして使用します。このエラーメッセージは、プレーンテキストとみなされます。
1. 特別な !> エラーメッセージを表示した後に中止します。ローダーは、メッセージがHTMLであることをシグナリングすることができます。RequestHandlerは、エラーメッセージをHTMLとしてフォーマットします。

#### 4.2.6. Main loop and termination

ローダーのメインループの仕事は、stdinで1バイトを受信するまで待つことである。バイトが受信されない限り、ローダーは終了せず、リクエストの処理を続ける必要があります。バイトが受信されたとき、次の条件が真であることが保証される：

- この特定のプロセスのすべてのクライアントが切断された。
- この特定のプロセスに対するクライアントはすべて切断された。

この保証は、RequestHandlerとApplicationPoolによって強制されます。したがって、ローダーは複雑なシャットダウンを実行する必要はありません。ただ、プロセスを終了させることができます。

サーバーがUnixドメインソケットファイルでリッスンしていた場合、ローダーはファイルを削除する必要さえない。ApplicationPoolはすでにそれを世話する。

#### 4.2.7. Stdout and stderr forwarding

loader が stdout に書き込んだ行のうち、接頭辞に `!> ` が付いていないものは、Passenger のコアによって自身の stdout に転送されます。同様に、loaderがstderrに書き込んだ行は、接頭辞に `!> ` が付いていようがいまいが、Passengerコアのstdoutに転送されます。

Spawnerがまだ作業をしている間、この転送はSpawner自身が行う。Spawnerが作業を終えると、この作業を2つのPipeWatcherオブジェクトに委託し、それぞれがこの目的のためにバックグラウンドスレッドを生成します。

### 4.3. Preloaders

プリローダーはローダーの中でも特殊なもので、スポーン時間の短縮やコピーオンライトを活用するために使われます。これについては、Spawning methods explainedで詳しく解説しています。

プリローダーはローダーとよく似ていますが、動作は若干異なります。また、Spawnerとの通信にstdin、stderr、stdoutを使用します。プロトコルは非常に似ています。

プリローダーは4つのステージで初期化されます。

1. まず、ローダーのハンドシェイクと同じハンドシェイクを行います。
1. ローダーと同じようにアプリケーションをロードします。
1. Spawnコマンドを受け付けるサーバーをセットアップします。
1. Spawnerに応答を送り返す。これはローダーが行う方法と似ていますが、アプリケーションがリクエストをリッスンする場所をSpawnerに伝える代わりに、プリローダープロセスがスポーンコマンドをリッスンする場所をSpawnerに伝えています。

初期化されると、プリローダはメインループに入り、終了すべきシグナルを受け取るまでスポーンコマンドを処理し続ける。

spawnコマンドを受信すると、プリローダーは子プロセス（すでにアプリケーションがロードされている）をフォークオフし、子プロセスのPIDをSpawnerに報告します。また、Spawnerと子プロセスの間に通信チャネルを設定します。

### 4.4. The AppTypes registry

Webサーバーがリクエストを受信すると、その中のPhusion Passengerモジュールがリクエストの属するアプリケーションの種類を自動検出します。これは、ファイルシステムを調べ、どのスタートアップファイルが存在するかをチェックすることで行います。例えば、config.ruが存在すれば、Rubyアプリと判断します。また、app.jsが存在すれば、Node.jsのアプリであると判断します。Phusion Passengerモジュールは、推測されたアプリケーションタイプをPassengerコアに転送します。

アプリケーションタイプを指定すると、関連するローダーとプリローダーを検索することができます。

サポートされるアプリケーションタイプ、スタートアップファイル、ローダー、プリローダーに関する情報は、以下の場所で定義されています：

- ファイル ext/common/ApplicationPool2/AppTypes.cpp の定数 appTypeDefinitions は、サポートされる言語のリストを保持しています。また、各言語に属するデフォルトのスタートアップファイル名も指定する。
- ファイル ext/common/ApplicationPool2/Options.h のメソッド getStartCommand() は、各言語で使用されるべきローダーを定義する。
- ファイル ext/common/ApplicationPool2/SpawnerFactory.h の tryCreateSmartSpawner() メソッドが、各言語で使用するプリローダを定義しています。
- ファイル lib/phusion_passenger/standalone/app_finder.rb のメソッド looks_like_app_directory? は、サポートするスタートアップファイルのリストを保持します。これはPassenger Standalone内でのみ使用されます。

## 5. Instance state and communication

Phusion Passengerを起動するたびに、新しいインスタンスが作成されます。すべてのインスタンスは、一緒に動作する複数のプロセス（Watchdog、Passengerコア、UstRouter、アプリケーションプロセス）で構成されています。これらのプロセスはすべて、互いに通信できる必要があります。また、それらのプロセスは、他のインスタンスに属するプロセスと通信してはならない。例えば、Apache+PassengerとNginx+Passengerを起動した場合、Apacheから起動したPassengerコアがNginxから起動したUstRouterを利用するようなことがあってはなりません。

明らかに、プロセスは特定のTCPポートをリッスンして通信することはできません。また、固定のUnixドメインソケットのファイル名でリッスンすることもできません。

そこで、インスタンス・ディレクトリが登場します。すべてのPhusion Passengerインスタンスは、独自の一時ディレクトリを持っています。そのディレクトリは、インスタンスが停止したときに削除されます。このディレクトリには、プロセスがリッスンするUnixドメインソケットファイルが含まれています。Phusion Passengerに関連するすべてのプロセスは、自分自身のインスタンスディレクトリがどこにあるかを知っており、したがって、同じインスタンスに属する他のプロセスとの通信方法を知っています。インスタンスディレクトリは、ext/common/InstanceDirectory.hで実装されています。

passenger-statusのような管理ツールは、インスタンスディレクトリを使って情報を問い合わせる。まず、システム上にどのインスタンス・ディレクトリが存在するかを確認する。もし1つしか見つからなければ、その唯一のインスタンスディレクトリ内のソケットに問い合わせを行う。そうでない場合は、エラーで中断し、問い合わせるインスタンスを具体的に選択するようにユーザーに要求する。

## 6. Appendix A: About Rack

RubyのWebアプリケーションのデファクトスタンダードインタフェースがRackです。Rackは、Webアプリケーション開発者が実装するためのプログラミングインターフェイスを規定しています。このインターフェースは、HTTPリクエストとレスポンスの処理をカバーし、特定のアプリケーションサーバーに依存しない。Rackに準拠したアプリケーションサーバーであれば、Rackの仕様を実装し、Rackに準拠したすべてのWebアプリケーションで動作させることができるというものです。

![](https://www.phusionpassenger.com/documentation/images/rack.png)

遠い昔、RubyのWebフレームワークはそれぞれ独自のインターフェイスを持っていたため、アプリケーションサーバーはそれぞれのWebフレームワークのサポートを明示的に追加する必要がありました。現在では、アプリケーションサーバーはRackをサポートするだけです。

![](https://www.phusionpassenger.com/documentation/images/many_web_framework_protocols.png)

Ruby on Railsはバージョン3.0から完全にRackに準拠しています。Rails 2.3は部分的にRackに準拠していましたが、それ以前のバージョンはRackにまったく準拠していませんでした。Phusion PassengerはRails 1.xと2.xのすべてのバージョンと同様にRackをサポートしています。

## 7. Appendix B: About Apache

Apache Webサーバーは、動的モジュールシステムとプラグイン可能なI/Oマルチプロセッシング（同時に1つ以上のHTTPクライアントを処理する能力）アーキテクチャを備えています。特定のマルチプロセッシング戦略を実装するApacheモジュールは、マルチプロセッシングモジュール（MPM）と呼ばれます。以前はシングルスレッドのマルチプロセスPrefork MPMが主流でしたが、最近ではマルチスレッドとマルチプロセスのハイブリッドWorker MPMが性能とスケーラビリティに優れているため、ますます人気が高まっています。さらに、Apache 2.4では、イベントMPMが導入され、イベント型/マルチスレッド型/マルチプロセス型のハイブリッドMPMとして、さらにスケーラビリティを向上させることができるようになりました。

prefork MPMは、mod_phpと相性の良い唯一のMPMであるため、現在も広く使用されています。

prefork MPM は、複数のワーカーの子プロセスを生成します。HTTP リクエストは、まず制御プロセスで受け付けられ、次にワーカープロセスのひとつに転送されます。次の章では、prefork MPM のアーキテクチャを示す図を示します。

## 8. Appendix C: About Nginx

Nginxは、普及が進んでいる軽量なWebサーバです。イベントドI/Oアーキテクチャにより、Apacheよりも小型・軽量でスケーラブルであることが知られています。とはいえ、NginxはApacheよりも柔軟性に欠けます。例えば、動的モジュールシステムを持たないため、すべてのモジュールはNginxに静的にコンパイルする必要があります。

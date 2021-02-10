# webmockについて
外部へのHTTPリクエストをstubするgem。Net::HTTPだけでなく、TCPSocketを使った場合もstubしてくれる。

## できること

HTTPリクエストをリクエストホストに対して投げる前にフックして、あたかも処理されたようにレスポンスを返すことができる。
つまり、StubやMockのようなことができる。
しかも、Net::HTTPだけでなくTCPSocketも対応してくれる。

## 使い方

### 1. `gem 'webmock'`を追加してbundle install

### 2. 事前に自分のほしいレスポンスを返却できるようにスタブを登録する。

```rb
require 'webmock'

WebMock.enable!

# GETリクエストのスタブ登録
WebMock.stub_request(:get, "http://www.example.com").to_return(
  body: File.read("#{Rails.root}/test/fixtures/stub_api_response.json"),
  status: 200,
  headers: { 'Content-Type' =>  'application/json' })

# PUTリクエストのスタブ登録  
WebMock.stub_request(:put, "http://www.example.com").to_return(
  body: '{"key": "value"}',
  status: 200,
  headers: { 'Content-Type' =>  'application/json' })

# 同じように他HTTPメソッドのリクエストもスタブ登録できます
```

これでスタブの設定はできたので、該当のAPIを呼び出すと設定したレスポンスデータが返却されるのですが、WebMockはデフォルト設定だとスタブ登録していないAPIのアクセスを許可していません。
スタブ化していないAPIのアクセスを許可するには以下の記述を追加します。

```rb
require 'webmock'
WebMock.allow_net_connect!(:net_http_connect_on_start => true)
```

### 3. RSpec
テストでも外部APIへのリクエストをスタブ化します。同じようにスタブ登録するだけです。
```rb
require 'rails_helper'
require 'webmock/rspec'

describe ApiTest
  before do
    WebMock.enable!

  end

  describe '#get api' do
    context '200 status' do
      before do
        # ここでテスト対象コードの中に実装されている外部API呼び出しをstub登録します
        WebMock.stub_request(:get, "http://www.example.com").to_return(
          body: File.read("#{Rails.root}/test/fixtures/stub_api_response.json"),
          status: 200,
          headers: { 'Content-Type' =>  'application/json' })
      end

      it 'returns 200 status' do
        # ここにテストコードが入ります
      end
    end
  end
end     
```

### 4. アプリケーションへの組み込み
次はRailsアプリケーションへWebMockを組み込む方法を説明します。
WebMockをアプリケーションコードに組み込むときに以下のような制約があるとします。
- development, test環境のみ使用する。staging、production環境ではWebMockをインストールしたくない
- development環境でも、WebMockを使用する場合と使用しない場合で使い分けたい
- 外部API呼び出しの実装はModule化している

この場合のWebMockの組み込みの一例を書いてみます

まず、Gemfileのdevelopment, test環境にのみ以下の記述を追加します。
require falseのオプションを追加して明示的に読み込むようにします。

```rb
group :development, :test do
  gem 'webmock', require: false
end
```

次はdevelopment.rbとtest.rbにWebMockを読み込むように記述します。
今回は以下のように設計します。
- development環境でWebMockを使用する場合は環境変数に値を設定する
- test/stub/api_stub.rbファイルに外部API呼び出しをstub登録するコードを実装する

```rb
# development.rb
if ENV["APIMODE"] == "stub" # 環境変数の有無を判定します
  require 'webmock'　# ここで明示的に読み込みます
  WebMock.allow_net_connect!(:net_http_connect_on_start => true)
  require_relative '../../test/stub/api_stub'
end
```

```rb
# test.rb
require 'webmock'
WebMock.allow_net_connect!(:net_http_connect_on_start => true)
```

stub登録の実装はtestディレクトリのファイルなので、ここで環境変数が設定されていない場合や、staging、production環境の場合は読み込まれません。
stub登録の実装は今までの説明と同じ内容ですが、stub登録のコードを実行後に本来の外部API呼び出しのコードを実行するため、最後にsuperの記述を追加しています。

```rb
# api_stub.rb
module ApiStub
  WebMock.enable!

  # アプリケーションコードに`call_get_api`という名称の外部API呼び出し実装を含むメソッドがあることして、ここで同名のメソッドを作成してstub登録します。  
  def call_get_api
    WebMock.stub_request(:get, "http://www.example.com").to_return(
      body: File.read("#{Rails.root}/test/fixtures/stub_api_response.json"),
      status: 200,
      headers: { 'Content-Type' =>  'application/json' })　　

    super # アプリケーションコードの実装を呼び出す
  end
end
```
次はアプリケーションコードからこのstub登録をするファイルを読み込むように設定します。
WebMockの使用有無を判定する環境変数とRAILS_ENVの二つを確認してstub登録の実装を読み込んでいます。

```rb
module ApiModule
  prepend ApiStub if ENV["APIMODE"] == "mock" && Rails.env.development?

  def call_get_api
    # ここに本来の外部API呼び出しの実装が入る
  end
end
```

ここではprependでスタブ実装のmoduleを読み込んでいます。そのためこのモジュールを読み込んだクラスの継承ツリーは以下のようになります。
1. ApiModule(外部API呼び出し実装)
1. ApiStub(stub登録コード)
1. モジュールを読み込んだクラス
stub登録の実装で最後にsuperと記述しているのは、この継承ツリーが作られることを前提に実装しているからです。
最後に外部API呼び出しのmoduleを使用するクラスですが、通常通りmoduleをincludeするだけです。

```rb
# api_client.rb
class ApiClient
  include ApiModule
  # ここに実装が入ります
end
```

## 参考サイト
- [webmock使ってみた](http://qiita.com/ogawatti/items/58bd4fe1180a0acfdd5e)
- [Rails開発でWebMockを使ってAPIアクセスをスタブ化する](http://dev.classmethod.jp/server-side/rails-development-with-webmock/)
- [Webmock RSpecのWebアクセスのモックを作成してくれる超便利Gem](http://morizyun.github.io/blog/webmock-rspec-gem-ruby-rails/)
- [外部に HTTP アクセスする機能の単体テストで WebMock 使ってみた](http://tnakamura.hatenablog.com/entry/2012/12/21/193832)
- [webmockでHTTPリクエストをstubしてファイルの中身を返す](http://xoyip.hatenablog.com/entry/2014/01/17/210605)
- [Webmock でリダイレクトが絡む HTTP 通信をスタブ化する](http://blog.kymmt.com/entry/webmock_stubbing_redirect)
  - loginを要するサイトでも、スクリプトでloginしてスクレイピングを可能にする。

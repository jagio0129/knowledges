Capistrano
==========================  
サーバ操作およびデプロイの自動化ツール  
詳細はここから(http://labs.gree.jp/blog/2013/12/10084/)  

# Capistranoの導入
## 1. インストール
RubyGemsでインストールする場合  
`$ gem install capistrano`  

Bundlerを使う場合はGemfileに  
`gem 'capistrano', '~> 3.0.1'`  
を記述してbundle install

## 2. 設定ファイルのひな形作成
`$ mkdir test-project //設定ファイルの配置場所作成`  
`$ cd test-project`  
`$cap install`

## 3. サンプル設定ファイルの完成イメージ
### config/deploy/test.rb  
* 作業対象サーバの設定を記述  

### config/deploy.rb  
* Capistranoデフォルトタスクの消去
* タスク「ソースコードの取得」の定義
* タスク「ビルドとアーカイビング・パッケージング」の定義
* タスク「アプリケーションのビルドとインストール」「アプリケーションの起動と停止」の定義

## 4. Capistranoデフォルトタスクの消去
Capistranoが推奨するデプロイ仕様を把握し既存のデプロイ手順をそれに合わせて変更してからCapistranoへ移行するのは手間がかかる。なので先に移行してしまってから必要に応じてCapistranoの推奨スタイルへ徐々に寄せていくためにCapistranoのデフォルトタスクを一度消す  
```
framework_tasks = [ :starting, :started, :updating, :updated, :publishing, :published, :finishing, :finished]

framework_tasks.each do |t|
  Rake::Task["deploy:#{t}"].clear
end

Rake::Task[:deploy].clear
```
## 5. config/deploy/ステージ名.rbを書く
ステージ名.rb(今回はtest.rb)には、ステージごとに異なる設定を書く。
* そのステージで作業対象サーバ  
  * ホスト名
  * サーバロール
  * ログインユーザ
  * SSH設定
  * その他、そのサーバに紐づく任意の設定  


* そのステージだけで実行するタスク  

## 6. config/deploy.rbを書く
config/deploy.rbにはステージ間で共通の設定を書く
* アプリケーション名
* レポジトリ名
* タスク
* それぞれのタスクで実行するコマンド  

これらの設定はDSLで記述する

### 設定値の変更と取得
アプリケーション名やレポジトリ名などの設定値は`set 名前, 値`で設定し、`fetch 名前`で取り出す
一度setした値はdeploy.rbやステージ.rbの全域で取り出すことができる  
```
set :repo_url. 'git@github.com:mumoshu/fignale_sample_app'  

fetch :repo_url
#=> "git@github.com:mumoshu/finagle_sample_app"
```










## 参考サイト
* 入門 Cpaistrano 3 ~すべての手作業を生まれる前に消し去りたい  
  * http://labs.gree.jp/blog/2013/12/10084/

*

loggerを使うことでコントローラー、モデル、ビューなどからロガーで設定されたログ出力先にログを出力することができる。

## ログを出力する

- 5つのメソッドが用意されている
  ```rb
  logger.fatal  # 致命的なエラー情報
  logger.error  # エラー情報
  logger.warn   # 警告情報
  logger.info   # お知らせ情報
  logger.debug  # デバッグ情報
  ```

- ログレベルの使い分けは次のようになっている。
  ```rb
  fatal # プログラムがクラッシュしたなどエラーハンドリングできないエラー
  error # エラーハンドリングできるエラー
  warn  # 警告
  info  # システム操作に対する一般的な役に立つ情報
  debug # 開発者向けの詳細な情報
  ```

- Railsでログレベルを設定するには以下のファイルに書く
  - `config/development.rb`
  - `config/test.rb`
  - `config/production.rb`

- 設定したログレベルより低い(fatal>error>warn>info)ログは出力されなくなる
  ```rb
  # :fatal, :error, :warn, :info, :debugが設定可能
  config.log_level = :warn  # logger.infoとlogger.debugの内容に出力されない
  ```

- もしくは、上記のinitializersイア外のログレベルrを変更したい場合は次のようにする。
  ```rb
  Rails.logger.level = 0  # :debug
  # 4 :fatal, 3 :error, 2 :warn, 1 :info, 0 :debug
  ```

## 新たにロガーを作成する
- Railsでロガーを作成するには、`config/development.rb`、`config/test.rb`、`config/production.rb`内で次のように記載
  ```rb
  # log/developmetn.logに出力するロガーを作成。週ごとにログファイルがローテートされる
  config.logger = Logger.new('log/development.log', 'weekly')

  # log/developmetn_special.logに出力する２つ目のロガーを作成。日ごとにログファイルがローテートされる
  config.special_logger = Logger.new('log/development_special.log', 'daily')
  ```

- コントローラー、モデル、ビューなどで上記で作成したログを使うことができる
  ```rb
  logger.debug "log/development.logに出力します"

  # 新たに定義したロガーは次のような指定をしないといけない
  Rails.application.config.special_logger.debug "log/development_special.logに出力される"
  ```

## ログのフォーマットを設定
- Railsでロガーを作成するには`config/development.rb`、`config/test.rb`、`config/production.rb`内でformatterを設定する。デフォルトのフォーマッターは次のように設定する。
  ```rb
  config.logger.formatter = ::Logger::Formatter.new

  # ログ内容です。
  I, [2014-12-18T22:50:46.439420 #42595]  INFO -- : Started GET "/users" for 127.0.0.1 at 2014-12-18 22:50:46 +0900
  I, [2014-12-18T22:50:46.585090 #42595]  INFO -- : Processing by UsersController#index as HTML
  ```

- ログフォーマットをカスタマイズすることもできる
  ```rb
  config.logger.formatter = proc do |severity, datetime, progname, msg|
    "[#{severity}]#{datetime}: #{progname} : #{msg}\n"
  end

  # ログ内容です。
  [INFO]2014-12-18 22:48:40 +0900:  : Started GET "/users" for 127.0.0.1 at 2014-12-18 22:48:40 +0900
  [INFO]2014-12-18 22:48:40 +0900:  : Processing by UsersController#index as HTML
  ```

## 参考文献
- [Railsでのログ/ロガーまとめ（ログ出力、ログレベル、ロガー作成、ログフォーマット）](http://ruby-rails.hatenadiary.com/entry/20150110/1420863998)

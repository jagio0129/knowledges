# RubyでTwitter
[公式リファレンス](http://www.rubydoc.info/gems/twitter)

TwitterのAPIを利用するには
- CONSUMER KEY
- CONSUMER SECRET
- ACCESS TOKEN
- ACCESS TOKEN SECRET
- が必要
-
## gemのインストール
`gem install twitter`

## Twitter developersへログイン
[デベロッパーサイト](https://dev.twitter.com/)よりアカウントを作成。APIが使用できるのは自分だけのため、TwitterUserは使用したいアカウント名を記入すること

## アプリケーションの作成
事前にアカウントと電話番号のヒモ付が必要。
websiteは(http://127.0.0.1)でおｋ

## アクセスレベルの変更
Application TypeをRead and Writeにしておく

## アクセストークンの取得
webサイトなどで使う場合は、都度アクセストークンが発行されるので必要ないが、ボットなどを作成する場合はTwitter認証を行わない代わりにアクセストークンの取得が必要となる。

## SSL証明書のインストール
SSL証明書をインストールしてRubyにわかるようにしないと例外を吐く
[ここ](https://curl.haxx.se/docs/caextract.html)から証明書の内容をコピペし、任意のファイル(cacert.pemとか)に貼り付ける。

スクリプト内でRubyに証明書の場所を教えてやるには

```rb
ENV["SSL_CERT_FILE"] = PEM_PATH
```

してやればよい。

## 参考サイト
- [Rubyのtwitter-gem(v5)でTwitter-APIを叩く](http://qiita.com/tkss_lab/items/137d47816ef2a51afe4e)
- [Ruby を用いて userstream に対応した Twitter bot を作る](http://qiita.com/owata/items/fb25faf71124eaa3cb14)
- [Twitter Gem を使って、気になるあの人の生活リズムを覗き見たい...! #loupestudy](http://lo-upe.hatenablog.com/entry/20150113/1421150990)

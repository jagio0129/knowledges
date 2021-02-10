# OAuth認証
OAuthは以下の特徴をもつ「認可情報の委譲」のための使用。
- あらかじめ信頼関係を構築したサービス感で
- ユーザの同意の元
- セキュアにユーザの権限を受け渡しする

具体的に言うと、
- あるサービスAの機能に対し
- ユーザの許可を得た他のサービスBがサービスAの提供するAPIにアクセスできる。

この場合、サービスAはprovider、サービスBはConsumerと呼ばれる。
UserがConsumerに許可を与えれば、ConsumerはProviderのAPIを使用することができる。

## OAuth認証の流れ
1. ConsumerはProviderにアプリケーションの登録を行い、Consumer KeyとConsumer Secretを取得する。
1. Consumer KeyとConsumer Secretをもとにrequestトークンを取得する
1. 取得したrequestトークンをもとに、ユーザがAPI認可するためのURLを生成する
1. ユーザは生成されたAPI認可のページアクセスし、ConsumerがAPIを利用してProvierにアクセスすること許可(または拒否)する
1. (以下許可されたと仮定)認可されたrequestトークンを元に、access token, access secretを取得する
1. access token, access secrete, consumer keyを元にAPIにアクセスする。

**ConsumerにはユーザのProviderのアカウント情報がわたっていないこと**

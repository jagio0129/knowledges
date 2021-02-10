暗号技術 用語集
===

## PKI(Public Key Infrastructure) 公開鍵暗号基盤

電子商取引では以下のリスクが伴う
- なりすまし
  - 他人が当事者の振りをして取引を行う
- 改ざん
  - 取引データ等の内容を書き換えてしまう
- 盗聴
  - 取引情報を他人（第三者）が盗み見る
- 否認
  - 上記によって「他人が行ったこと」と否定しなければいけないリスク

PKIは、上記のような想定されるリスクの回避に使用できる安全・信頼向上のための認証技術。PKIは認証局（CA）、登録局、リポジトリなどから構成されている。認証局が証明書を発行するが、その証明局が信頼できるかをチェックするためにほかの(上位の)認証局が必要になる。最終的に登って行った先がルート認証局

## SSL

## TLS

## SSLサーバ証明書
サイバートラスト社に代表される信頼された認証局が発行する。以下2つの機能を有する
- 通信先サーバサイトの運営組織が実在していることを証明する。
- サーバ・クライアント間の通信を暗号化し、情報が第三者から干渉されないようにする。

### 実在証明がなぜ必要か
- サイト運営者が悪意を持っていた場合、情報の安否は保証できない
- そのため認証局が、証明書を発行する際にその運営団体の審査を行い存在を確認する

## クライアント証明書
- SSLサーバ証明書がサーバにインストールされ、ウェブサイトの所有者の実在性を認証する
- クライアント証明書は、サービス等を利用するユーザのデバイスの正当性を証明する

![cert](https://jp.globalsign.com/images/service/clientcert/knowledge/ill_index09.gif)

## ルート証明書
- ルート認証局が発行した証明書

## 検証
- TLSサーバ証明書をクライアント側で検証する際に何をやっているのか？
  - クライアントがサーバにアクセスした際、サーバ証明書が降ってくる
  - その証明書のSubjectとIssueフィールドに着目
  - 下位証明書のIssueが上位証明書のSubjectに一致するように並べ替えられる
    - 上位の証明書をたどっていく
    - SubjectとIssueが一致するものを自己署名証明書といい、これ以上上位はいない
  - 下位証明書の本文と署名フィールドの組み合わせを、上位証明書の公開鍵で検証する。あっていれば合格
- より詳細は[こちら](https://qiita.com/n-i-e/items/35cba71d04b9123e676c)

## X.509
ISO規格で広く使われている証明書の形式。詳細は[電子証明書](https://www.ipa.go.jp/security/pki/033.html)を読むと良い。

## PKCS
同じく証明書の形式。RSA社によって策定。PKCS で定められた標準は、現在、1番から 15番まで存在する。詳細は[電子証明書](https://www.ipa.go.jp/security/pki/033.html)を読むと良い。

## ASN.1
X.509証明書や関連技術のフォーマットの表記方法。構造化された複雑なデータ形式を表現するためのデータ構造。より詳細は[ASN.1](http://e-words.jp/w/ASN.1.html)

実用的な話は[ASN.1 データ生成/解析の事始](https://www.soum.co.jp/misc/individual/asn1/)

ASN.1 記法で FooProtocol のデータ構造を定義したものが以下。

```
FooProtocol DEFINITIONS ::= BEGIN

    FooQuestion ::= SEQUENCE {
        trackingNumber INTEGER,
        question       VisibleString
    }

    FooAnswer ::= SEQUENCE {
        questionNumber INTEGER,
        answer         BOOLEAN
    }

END
```

データ型の説明等は[抽象記法](http://www5d.biglobe.ne.jp/stssk/asn1/basic.html)に沿っている

OID(Object ID)一覧は[こちら](http://www.umich.edu/~x509/ssleay/asn1-oids.html)。OIDの符号化のアルゴリズムは[こちら](https://docs.microsoft.com/ja-jp/windows/win32/seccertenroll/about-object-identifier?redirectedfrom=MSDN)

## DES(Data Encryption Statndard)
暗号化アルゴリズムの一つ。アメリカで1977年に採用された。

3DESなどは同じ暗号化を3回繰り返す

単純なDESでは、ある鍵で同一の平分を暗号化すると同一の暗号文になってしまう。そのためモードが必要になった。メジャーどころは以下

- DES-ECB
- DES-CBC

## AES(Advanced Encryption Standard)
暗号化アルゴリズムの一つ。アメリカで2001年に採用された。DESには弱点がありそれを克服したものである。主にWPA2(無線LAN通信を保護する規格)

- 参考
  - https://ja.wikipedia.org/wiki/%E6%9A%97%E5%8F%B7%E5%88%A9%E7%94%A8%E3%83%A2%E3%83%BC%E3%83%89
  - https://it-trend.jp/encryption/article/64-0070

## RSA
桁数が大きい合成数の素因数分解問題が困難であることを安全性の根拠とした公開鍵暗号の一つ。RSA Security社がライセンスを持っていたが今は誰でも使える。

セキュリティ強度的には2048以上なら無難レベルっぽい。([参考](https://qiita.com/wnoguchi/items/a72a042bb8159c35d056))


## ECDSA
RSA同様デジタル署名の暗号化方式。楕円曲線をアルゴリズムに用いているよう。アルゴリズムの詳細は[ここあたり](https://qiita.com/angel_p_57/items/f2350f2ba729dc2c1d2e)を読むのがよさそう

## EdDSA
RSA同様デジタル署名の暗号方式。ECDSAよりも強力。アルゴリズムの詳細は[ここあたり](https://qiita.com/angel_p_57/items/f2350f2ba729dc2c1d2e)

各暗号化アルゴリズムの比較まとめは[こちら](https://dnsops.jp/event/20180627/20180627-ed25519.pdf)が参考になる。

## SHA
Secure Hash Algorithmシリーズの暗号学的ハッシュ関数。今はSHA512とかある。

## MD5
暗号学的ハッシュ関数のひとつ。脆弱性があるので使うべきではない。生成はSHA1より早いらしい

## DER
鍵や証明書をASN.1というデータ構造で表し、それをシリアライズしたバイナリファイル。DERはバイナリエンコードなので、そのままでは文字列化できない。

## PEM
鍵や証明書のエンコーディングの一つ。ファイルの先頭に -- BEGIN... という行があるのをみたら「PEMだな」と思えば良い。opensslコマンドのデフォルトのエンコーディング。ssh-keygenで生成される秘密鍵とかもこれ。中身はbase64でエンコードされてBEGIN/ENDで挟まれている

```
-----BEGIN CERTIFICATE-----
<base64でエンコードされたなにか>
-----END CERTIFICATE-----
```

CERTIFICATEと書かれている場合、内容はX509である。CERTIFICATEの部分に部分がPKCS7になっている場合はPKCS#7

## CSR(Certificate Signing Request) : 証明書署名要求
証明局に対して、SSLサーバ証明書への署名を申請する内容を表す。

CSRには以下の情報が記載されている

- 公開鍵
- 公開鍵の所有者情報
- 申請者が対応する秘密鍵を持っていることを示すための申請者の署名

証明局は証明書にその所有者情報を署名することで、所有者の存在を証明している。

CSRファイルは以下のようなコードになっている。
```
-----BEGIN CERTIFICATE REQUEST-----
MIIBpDCCAQ0CAQAwZDELMAkGA1UEBhMCSlAxDjAMBgNVBAgTBVRva3lvMRMwEQYD
:
pj10tZdLyYDNraCNYi6nO87P1l62oFa+tckDi8wmATdsS4T5GJY5DA==
-----END CERTIFICATE REQUEST-----
```

## SCEP(Simple Certificate Enrollment Protocol)
PKI操作に使用するプロトコル。主な特徴は以下。

- リクエスト/レスポンスモデルはHTTPに基づく
- RSAベースの暗号化のみサポート
- 証明書リクエスト形式としてPKCS#10を使用
- 暗号キー/暗号化されたメッセージを伝えるためにPKCS#7を使用
- 要求者による定期ポーリングを使った、サーバでの非同期の許可をサポート
- 証明書思考リスト(CRL)検索サポートを限定
- オンライン証明書執行はサポートしない
- サーバと供給者の間だけで共有する必要がある、証明書署名要求(CSR)内に**チャレンジパスワードフィールド**の使用が必要

SCEPの登録と使用方法は一般に次のワークフロー

1. CA証明書のコピーを取得し、検証
2. CSRを生成しCAに安全に送信する
3. 証明書が署名したかどうか確認するためにSCEPサーバにポーリング
4. 現在の証明書の有効期限前に新しい証明書を取得すために必要に応じて再登録する
5. 必要に応じてCRLを再取得する

より詳細な情報は[Simple Certificate Enrollment Protocol の概要](https://www.cisco.com/c/ja_jp/support/docs/security-vpn/public-key-infrastructure-pki/116167-technote-scep-00.html)

## 参考
- [SSLサーバ証明書の基礎知識](https://www.cybertrust.ne.jp/sureserver/basics/ssl1.html)
- [rubyで暗号技術入門](https://qiita.com/mitswku/items/9f479765dbc8ec043f56)
- [RSA鍵、証明書のファイルフォーマットについて](https://qiita.com/kunichiko/items/12cbccaadcbf41c72735)
- [デジタル証明書の仕組み](http://www.fc-lab.com/network/server/pki/certificate.html)
- [電子証明書](https://www.ipa.go.jp/security/pki/033.html)
- [証明書のファイル形式について](http://moca.wide.ad.jp/moca_guide/about_fileformat.html)

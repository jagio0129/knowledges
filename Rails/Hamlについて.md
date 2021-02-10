# Hamlについて

RailsではView（html出力）のために標準でerbが使用される。
しかし、erbは凡庸的なテンプレートであり、HTML以外のファイルにRubyのコードを埋め込むことができる。
そのためHTML作成という点で見ると冗長な箇所がある。

より生産的にHTMLを作成するためにRailsでは

- Haml
- Slim

という２つのHTML用テンプレートエンジンがよく使われる。

## Hamlの基礎

```HTML
[erbファイル]

<!DOCTYPE html>
<html>
<head>
  <title>SlimTest</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>
</head>
<body>

<header id="header">
  <h1 class="title logo">Slim Test</h1>
</header>

<%= yield %>

</body>
</html>
```

```html
[hamlファイル]

!!! 5
%html
  %head
    %title HamlTest
    = stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true
    = javascript_include_tag 'application', 'data-turbolinks-track' => true
    = csrf_meta_tags

  %body

  = yield
```

### Hamlの簡単な覚え方

1. <、>、<%、%>をタグから削除し。&をタグの直前につける。また、閉じタグは全て削除する
1. each、ifなどのロジック部分の先頭には、-を記載する（endは必要ない)
1. <%= ... >は=にする
1. class属性やid属性は、%p.fieldsや%p#contentsなどにする。タグがdivのときはdivを省略し#contentと記載できる
1. class属性やid属性以外のhrefやimgなどの属性を追加したい場合は、ハッシュで記載する（例：%a{:href => "#"} リンクテキスト）
1. コメントは、/ このコメントはHTMLに変換後に表示されないで記載する


## RailsにHamlを導入する

Gemfileに以下を追加

```rb
gem 'haml-rails'
```
q
RailsにHamlが導入されたので、xxx.html.hamlという拡張子でViewファイルを作成することでHamlとしてRailsが読み込んでくれる。
**erbとhamlの同名ファイルがある場合、erbが優先して読みこまれるため、erbファイルは削除すること**
**haml-rails gemを導入後は、rails generatorでscaffoldやcontrollerを作成すると、Viewファイルはすべて.erbの代わりに.hamlで作成される。**

## Hamlに変換するポイント

- h1, tableなどのタグの<と>を削除し、%をタグに付け、閉じタグは削除する
- id属性、class属性、その他の属性を追加する場合は、{}でハッシュ形式でタグの直後に記載する
- ifやeachなどロジック部分の行には、先頭に-を記載する
- 上記をインデントに気をつけながら記載する

---
title: golang
tags: [golang]
excerpt: golangとは
---

# Go言語、golang
- 2009年にgoogleより発表された言語

# 1.特徴
- シンプルな言語仕様で学習が比較的容易
- 巨大なコードでも高速にコンパイルできるため大規模開発にも向いている
- Win、OSX、Linuxなどの環境に合わせた実行ファイルを生成するクロスコンパイルの仕組みがあるため、作成したプログラムを容易に配布できる。
- 並行処理のサポートも充実しており、ミドルウェアの開発などにも適している。
- CやC++などが使用されるシステムプログラミング領域で、より効率よくプログラムを書くことを目的に作られた。

## 言語をシンプルに保つ
他の言語が持つような機能の多くを削り、言語をシンプルに保っている。

**最小限の構文**
- 繰り返し構文はfor文しかない。
- 三項演算子はない

## ツールのサポート
Goには開発を支援するコマンドが同梱されており、様々なツールを使用できる

コマンド|用途
---|---
go build|プログラムのビルド
go fmt|Goの規約に合わせてプログラムを整形
go get|外部パッケージの取得
go install|プログラムのビルドとインストール
go run|プログラムのビルドと実行
go test|テストやベンチマークの実行
go tool yacc|パーサをGoで出力するGo実装のyacc(パーサジェネレータ)
godoc|ソースからドキュメントの生成

## クロスコンパイル
例えば、Linux64ビット用の実行ファイルを、OSXのマシンで生成したい場合は、OSX上でLinux64ビット用のコンパイラをインストールするために、事前に次のようにGOOSとGOARCHという変数を指定してバッチファイルを実行する必要がある

```
$ cd go/src # Goのインストールディレクトリのsrc
$ GOOS=linux GOARCH=amd64 ./make.bash --no-clean
```

準備が終われば以下のようにして目的の実行ファイルが生成できる

```
$ GOOS=linux GOARCH=amd64 go build hello.go
```

## プロジェクト構成とパッケージ
プロジェクト内はbin,pkg,srcディレクトリを作成するのが一般的
```
myproject
├── bin # go install時の格納先
├── pkg # 依存パッケージのオブジェクトファイル
└── src # プログラムのソースコード
```

### 環境変数GOPATHの指定
```
$ cd myproject
$ export GOPATH=`pwd` # myprojectをGOPATHに登録
```

### パッケージの作成
Goでは1つのパッケージは1つのディレクトリに格納する。

### buildとrun
正しくGOPATHが設定された状態でgo runコマンドでmain.goを実行すると、gosampleパッケージの場所が正しく解決され、プログラムを実行できる。
```
$ cd $GOPATH/src/main
$ go run main.go
hello world
```

次にビルドして一つの実行形式のファイルを生成する。go buildコマンドを用いてその場に実行ファイルを作ることもできるが、go installコマンドを用いると、生成されたファイルが$GOPATH/binに自動的に格納される。
```
$ cd $GOPATH/src/main
$ go install
```

build後のプロジェクト内は次のようになる。
```
myproject
├── bin
│ └── main
├── pkg
│ └── darwin_amd64
│ └── gosample.a
└── src
    └── gosample
    ｜  └── gosample.go
    └── main
        └── main.go
```
# Gemの作り方
```
bundle gem test_gem -t
```
`-t`はテストも作成

## gem名について
`_(アンダースコア)`はディレクトリ構造に変換される。例えば`test_gem`という名前で作成すると、`lib/test_gem`だったところが`lib/test/gem.rb`のような構造になる。

## gemspec
gemのメタデータが書かれている。

```rb
# coding: utf-8
lib = File.expand_path('../lib', __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require 'test_gem/version'

Gem::Specification.new do |spec|
  spec.name          = "test_gem"
  spec.version       = TestGem::VERSION
  spec.authors       = ["HOGE"]
  spec.email         = ["hoge@example.com"]
  spec.description   = %q{TODO: Write a gem description}
  spec.summary       = %q{TODO: Write a gem summary}
  spec.homepage      = ""
  spec.license       = "MIT"

  spec.files         = `git ls-files`.split($/)
  spec.executables   = spec.files.grep(%r{^bin/}) { |f| File.basename(f) }
  spec.test_files    = spec.files.grep(%r{^(test|spec|features)/})
  spec.require_paths = ["lib"]

  spec.add_development_dependency "bundler", "~> 1.3"
  spec.add_development_dependency "rake"
  spec.add_development_dependency "rspec"
end
```

TODOを埋め、spec.homepageにはgithub urlを入れる。

このファイルで重要なのは依存するgemの設定。`add_dependency`と`add_development_dependency`を使って必要なgemを追加する。Gemfile自信はこのファイルを参照しているので変更する必要はない。

## Gemの実行方法
```rb
require 'test_gem/version'
module TestGem
  def self.greet
    'Hello World!'
  end
end
```

```
# gem のインストール
bundle install
# rubyの対話型インタプリタを起動
bundle exec irb                                                                              [2.1.4]
irb(main):001:0> require 'test_gem'
=> true
irb(main):002:0> TestGem.greet
Hello World!
```

irbでなくpryが使いたいときはgemspecファイルに
```rb
spec.add_development_dependency "rspec"
```
を追加して`bundle exec pry`としてやればよい。

## ビルドやリリース
ビルドやリリースはすべてrakeタスクが用意されている。`rake -T`で確認できる。

リリース前は`lib/test_gem/version.rb`を手動で編集してバージョン番号を挙げないといけない。

実際に試してみたいときは、**対象ファイルをコミットして**
```
rake install
```
でインストールしないと実行できない。

# 参考
[Gemの作り方まとめ](http://masarakki.github.io/blog/2014/02/15/how-to-create-gem/)
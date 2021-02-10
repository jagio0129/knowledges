---
title: メソッドいろいろ
tags: []
excerpt: Rubyの様々なメソッドについてのまとめ
---
## [const_set](http://ref.xaio.jp/ruby/classes/module/const_set)
- `Class.const_set(name, value)`
- クラスやモジュールに定数を設定する。

```ruby
class Product
end

Product.const_set(:TAX_RATE, 0.05)
p Product::TAX_RATE # => 0.05
```

## [send](http://ref.xaio.jp/ruby/classes/object/send)
- `obj.send(name, [arg, ...])`
- レシーバの持っているメソッドを呼び出す
- nameにメソッド名をシンボルか文字列で指定
- メソッドの引数を指定したい場合は、第2引数以降に引数を並べる

```ruby
class Cat
  def hello(n = nil)
    n ? Array.new(n, "meow!") : "meow!"
  end

  private
  def sleep
    "zzz..."
  end
end

cat = Cat.new
p cat.send(:hello)    # => "meow!"
p cat.send(:hello, 3) # => ["meow!", "meow!", "meow!"]
p cat.send(:sleep)    # => "zzz..."
```

## eval
- 文字列を式として評価する

## [define_method](http://ref.xaio.jp/ruby/classes/module/define_method)
- クラスやモジュールにメソッドを定義します。
- defによるメソッド定義を使わずにメソッドを定義できます。
- 引数にはメソッド名を渡し、メソッドの本体はブロックで記述します。
- define_methodは渡されたブロックからProcオブジェクトを作成します。
- 戻り値はそのProcオブジェクトになります。

## [module Enumerable](http://ref.xaio.jp/ruby/classes/enumerable)
- 外列やハッシュなど集合を表すクラスに数え上げや検索などのメソッドを提供する
- Enumerableモジュールのメソッドはすべて、オブジェクトのeachメソッドを呼び出す。
- 自作のクラスにEnumerableモジュールをインクルードするには、eachメソッドを実装する必要がある。

## [tap](http://ref.xaio.jp/ruby/classes/object/tap)
- ブロック変数にレシーバ自身を入れてブロックを実行する。
- 戻り値はレシーバ自身
- メソッドチェーンの中にtapメソッドを挟み込み、ソースコードを完結にする目的で使われる。

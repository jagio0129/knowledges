---
title: 例外処理の基礎
tags: []
excerpt: begin, resque, raiseや例外の補足などの基礎
---
## Rubyにおける例外処理の基本(begin rescue end)
```ruby
class ExceptionTest
  def test
    begin
      # 0での除算でエラーを発生させる
      1/0
    rescue ZeroDivisionError => ex
      puts "ZeroDivisionError"
    end
  end
end

obj = ExceptionTest.new
# 例外発生
obj.test # => ZeroDivisionError
```

### rescue StandardError => exを追加
```ruby
class ExceptionTest
  def test
    begin
      # 0での除算でエラーを発生させる
      1/0
    rescue StandardError => ex
      puts "StandardError"
    rescue ZeroDivisionError => ex
      puts "ZeroDivisionError"
    end
  end
end

obj = ExceptionTest.new
# 例外発生
obj.test # => StandardError
```
- 発生した例外(今回ならZeroDivisionError)の親クラスであるStandardErrorkクラスのresucue節が実行される。

### raiseの追加
- railseは例外を発生させるKernelモジュールのインスタンスメソッド

```ruby
class ExceptionTest
  def test
    begin
      # 0での除算でエラーを発生させる
      1/0
    rescue ZeroDivisionError => ex
      puts "ZeroDivisionError"
      raise
    rescue StandardError => ex
      puts "StandardError"
    end
  end
end

obj = ExceptionTest.new
# 例外発生
obj.test

# Error内容
=begin
ZeroDivisionError
exception_test5.rb:6:in `/': divided by 0 (ZeroDivisionError)
        from exception_test5.rb:6:in `test'
        from exception_test5.rb:18:in `<main>'
=end
```

- **raiseが発生した時点で実行中のrescue節が終了し、呼び出し元に戻る**
- rubyは例外が発生した場合、発生した地点から例外が補足されるまでスタックを辿り続ける。

呼び出し元に例外補足処理を加えてみる。
```ruby
class ExceptionTest
  def test
    begin
      # 0での除算でエラーを発生させる
      1/0
    rescue ZeroDivisionError => ex
      puts "ZeroDivisionError"
      raise
    rescue StandardError => ex
      puts "StandardError"
    end
  end
end

obj = ExceptionTest.new
# 例外発生
begin
  obj.test
rescue => ex
  puts "Other"
  puts "class: #{ex.class}"
end

=begin
ZeroDivisionError
Other
class: ZeroDivisionError
=end
```
- しっかりとステックを辿り、例外が補足されていることがわかる。
- rescue節内で実行されるraiseメソッドに引数を省略した場合、**対象のrescue節の引数と同じオブジェクトが渡される。**

### ensureの追加
```ruby
class ExceptionTest
  def test
    begin
      # 0での除算でエラーを発生させる
      1/0
    rescue StandardError => ex
      puts "StandardError"
      raise
    rescue ZeroDivisionError => ex
      puts "ZeroDivisionError"
    ensure
      puts "Ensure"
    end
  end
end

obj = ExceptionTest.new

begin
  obj.test
rescue => ex
  puts "Other"
  puts "class: #{ex.class}"
end

=begin
ZeroDivisionError
Ensure
Other
class: ZeroDivisionError
=end
```
- ensure節を追加したことにより処理が行われているが、**raiseにより発生した例外補足の前にensure節が実行されている**

## まとめ
```ruby
  begin
　　例外が起きそうな通常処理
　rescue
　　例外が起きた場合の処理
　else
　　例外が発生しなかった場合の処理
　ensure
　　例外が発生してもしなくても行う処理
　end
```

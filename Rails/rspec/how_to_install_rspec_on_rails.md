RSpecをRailsに導入する方法
===

- Rails v6.0.0
- Ruby v2.6.3

```
# Gemfile
group :development, :test do
  gem 'rspec-rails', '~> 3.6'
end

# install
bundle install

# RSpec 用の初期ファイルをgenerate
bundle exec rails generate rspec:install
```

## Spring を使った RSpec の導入

```
# Gemfile
group :development, :test do
  gem 'spring-commands-rspec'
end

# install
bundle install

# RSpec の stub ファイルを作成
bundle exec spring binstub rspec
```

### 参考サイト
- [Ruby on Rails のテストフレームワーク RSpec 事始め](https://qiita.com/tatsurou313/items/c923338d2e3c07dfd9ee)
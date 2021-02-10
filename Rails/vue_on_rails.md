# 既存のRailsプロジェクトにvueを導入
対象環境はRails 5.1.4

## 手順

1. nodeのインストール
```
sudo apt install -y nodejs npm
sudo npm install n -g
sudo n stable
sudo npm install yarn -g
```
2. Gemfileに`gem 'webpacker', github: 'rails/webpacker'`を追記して`bundle install`



Hugo
===

## reference
https://gohugo.io/

## env
ubuntu 18.04

## setup
```
# wget current .deb 
cd
wget https://github.com/gohugoio/hugo/releases/download/v0.62.2/hugo_0.62.2_Linux-64bit.deb
sudo apt install ./hugo_0.62.2_Linux-64bit.deb
hugo version
  Hugo Static Site Generator v0.62.2-83E50184 linux/amd64 BuildDate: 2020-01-05T18:51:38Z
```

## usage
```
# create skelton folder
hugo new site myblog

# build
hugo

# server up(vagrant env)
hugo serve -b http://192.168.33.10 --bind 0.0.0.0
```

## themes
https://themes.gohugo.io/

## 参考サイト
- https://computingforgeeks.com/how-to-install-hugo-on-ubuntu-debian/
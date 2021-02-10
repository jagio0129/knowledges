Goの導入
===

Ubuntu18.04で構築。rbenvとだいたい同じ流れ。

```sh
# install goenv
git clone https://github.com/syndbg/goenv.git ~/.goenv

# bashrcに記載
export GOENV_ROOT=$HOME/.goenv
export PATH=$GOENV_ROOT/bin:$PATH
eval "$(goenv init -)"
# logout

# show install list
goenv install -l

# install
goenv install <version>

# GOPATHの設定
mkdir ~/go
export GOPATH=$HOME/go
PATH=$PATH:$GOPATH/bin
```

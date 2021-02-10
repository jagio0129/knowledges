# Python 3.6.0(Ubuntu 16.04 LTSの場合)
```
sudo apt-get update
sudo apt-get install build-essential libncursesw5-dev libgdbm-dev libc6-dev zlib1g-dev libsqlite3-dev tk-dev libssl-dev openssl libbz2-dev libreadline-dev

# pyenvのインストール
git clone https://github.com/yyuu/pyenv.git ~/.pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.profile
echo 'eval "$(pyenv init -)"' >> ~/.profile
source ~/.profile

# Python3.6.0のインストール
pyenv install 3.6.0

# globalに設定
pyenv global 3.6.0
python -V
```

- https://qiita.com/Fendo181/items/912b65c4fcc3d701d53d
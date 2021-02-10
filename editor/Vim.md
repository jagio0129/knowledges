Vim導入とカスタマイズ
========================
### Vimのバージョン確認
`yum list installed | grep vim`

### Vim最新インストールためのパッケージ導入
```
$ yum install mercurial
$ yum install ncurses-devel
$ yum install make
$ yum install gcc
```

### Vimインストール
```
$ cd /usr/local/src
$ sudo hg clone https://bitbucket.org/vim-mirror/vim vim
```
念のため最新をPull
```
$ cd vim
$ sudo hg pull
pulling from https://bitbucket.org/vim-mirror/vim
searching for changes
no changes found
```
make(コンパイル)
```
$ sudo ./configure --with-features=huge --enable-multibyte --disable-selinux
$ sudo make
$ sudo make install
```
(おまけ)luaを入れる場合
```
$ sudo yum install mercurial ncurses-devel lua lua-devel
$ cd /usr/local/src
$ ./configure \
    --with-features=huge \
    --enable-multibyte \
    --enable-luainterp=dynamic \
    --enable-gpm \
    --enable-cscope \
    --enable-fontset
$ make
$ make install
```

### Vimrcカスタマイズ
.vimrcのみに設定を書いていくと行数が多すぎるので幾つかに分ける。  

ユーザホーム（`$ echo $HOME`で確認）下にdotfilesフォルダを作成し、GitHub等で管理。他の環境のvimに適応できるようにする
```
$ cd
$ mkdir dotfiles
```
dotfilesフォルダに  
- .vim(ディレクトリ)
- .vimrc
- .vimrc.basic (基本設定)
- .vimrc.neobundle (追加するプラグイン)
- .vimrc.plugin (プラグイン設定)
を作成
filetype plugin indent onを設定している場合は
- .vim/filetype.vim
- .vim/ftplugin/
も作成

[.vimrc]
```
if filereadable(expand('$HOME/dotfiles/.vimrc.neobundle')) " ファイルが読み込み可能かチェック
  source $HOME/dotfiles/.vimrc.neobundle " .vimrcファイル読み込み
  if filereadable(expand('$HOME/dotfiles/.vimrc.plugin'))
    source $HOME/dotfiles/.vimrc.plugin
  endif
endif
if filereadable(expand('$HOME/dotfiles/.vimrc.basic'))
  source $HOME/dotfiles/.vimrc.basic
endif
```
とすればファイルを分けて設定できる。
filetypeに関しても似たような設定を行う
[.vim/filetype.vim]
```
augroup filetypedetect
  au BufRead,BufNewFile *.rb setfiletype ruby
  au BufRead,BufNewFile *.php setfiletype php
  au BufRead,BufNewFile *.swift setfiletype swift
augroup END
```
などとし、ftplugin内にruby.vimのようなファイルに言語別で設定を記述すれば良い

#### NeoBundleのインストール
NeoBundleの導入はgit cloneではなくgit add submoduleで追加する必要がある。  
.vimファルダの下にbundleフォルダを作成後,
```
$ git submodule add https://github.com/Shougo/neobundle.vim .vim/bundle/neobundle.vim
```

シンボリックリンクの作成。違う環境にdotfilesを設定したい時にも必要なので、シェルスクリプトで半自動化。dotfilesの中にdotfilesLink.shファイルを作成。
```
#! /bin/bash
ln -s ~/dotfiles/.vimrc ~/.vimrc
ln -s ~/dotfiles/.gvimrc ~/.gvimrc
ln -s ~/dotfiles/.bashrc ~/.bashrc
ln -s ~/dotfiles/.bash_profile ~/.bash_profile
ln -s ~/dotfiles/.vim ~/.vim
ln -s ~/dotfiles/.gitconfig ~/.gitconfig
ln -s ~/dotfiles/.gitignore_global ~/.gitignore_global
```

権限変更
```
$ chmod +x dotfilesLink.sh
$ ./dotfilesLink.sh
```

git submoduleで追加したneobundle以外はgitの管理対象外にしておく
```
.vim/bundle/*
!.vim/bundle/neobundle.vim
```

GitHubにpush
```
$ cd dotfiles
$ git init
$ git add .
$ git commit -m "initial commit"
$ git remote add origin git@github.com:youraccountname/dotfiles.git
$ git push -u origin master
```

#### 別環境でクローンして使うには
dotfilesを$HOMEにクローン
`$ git clone git@github.com:youraccountname/dotfiles.git`

dotfilesで管理するので、シンボリックリンクの作成(例.vimrc)
```
$ cd dotfiles
$ git submodule init
$ git submodule update
$ ./dotfilesLink.sh
```

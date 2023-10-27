```shell
# 導入
brew install anyenv
anyenv init
set -x PATH $HOME/.anyenv/bin $PATH
exec $SHELL -l # シェルを再起動
anyenv install --init

# anyenv-updateプラグイン
#   anyenvで入れた*env系を一括でアップデートしてくれる
mkdir -p ~/.anyenv/plugins
git clone https://github.com/znz/anyenv-update.git ~/.anyenv/plugins/anyenv-update
git clone https://github.com/znz/anyenv-git.git ~/.anyenv/plugins/anyenv-git
# 操作は以下
anyenv git pull # anyenvでインストールした全ての**env系とインストールされているプラグインのアップデート
anyenv git gc # ガーベージコレクション（お掃除）
anyenv git remote -v # 全てのリモートリポジトリを表示
anyenv git status # gitのステータス表示

# installできる*envたち確認
anyenv install -l

# rbenvのinstall
anyenv install rbenv
# vim ~/.config/fish/config.fish
set -x RBENV_ROOT $HOME/.anyenv/envs/rbenv
set -x PATH $PATH $RBENV_ROOT/bin
set -gx PATH $RBENV_ROOT/shims $PATH
set -gx RBENV_SHELL fish

# nodenv
anyenv install nodenv
nodenv install 19.9.0
set -x NODENV_ROOT $HOME/.anyenv/envs/nodenv
set -x PATH $PATH $NODENV_ROOT/bin
eval (nodenv init - | source)

# goenv
anyenv install goenv
nodenv install 1.21.3
set -x GOENV_ROOT $HOME/.anyenv/envs/goenv
set -x PATH $PATH $GOENV_ROOT/bin
eval (goenv init - | source)
```

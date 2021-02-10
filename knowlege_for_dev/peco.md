peco
===

## Install(Ubuntu)
```
cd 

# 以下のページから最新バージョンをwget
# https://github.com/peco/peco/releases
wget https://github.com/peco/peco/releases/download/v0.5.7/peco_linux_386.tar.gz
tar xzvf peco_linux_386.tar.gz
cd peco_linux_386
# PATHの通っている所にpecoを配置
sudo cp peco /usr/local/bin
peco --version
```

## Ctrl + r でpecoを使う
```
# bash
function peco-select-history() {
    local tac
    which gtac &> /dev/null && tac="gtac" || \
        which tac &> /dev/null && tac="tac" || \
        tac="tail -r"
    READLINE_LINE=$(HISTTIMEFORMAT= history | $tac | sed -e 's/^\s*[0-9]\+\s\+//' | awk '!a[$0]++' | peco --query "$READLINE_LINE")
    READLINE_POINT=${#READLINE_LINE}
}
bind -x '"\C-r": peco-select-history'

# fish(fisher)
# fisher add oh-my-fish/plugin-peco
function fish_user_key_bindings
  bind \cr peco_select_history
end
```

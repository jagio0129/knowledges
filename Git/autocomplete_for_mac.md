MacでGitの補完を有効にする
===

デフォルトだと補完が効かないので設定する

```sh
# for bash
curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -o ~/.git-completion.bash

# add ~/.bash_profile
if [ -f ~/.git-completion.bash ]; then
  . ~/.git-completion.bash
fi

chmod +x ~/.git-completion.bash
source ~/.bash_profile
```
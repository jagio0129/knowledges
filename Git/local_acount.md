リポジトリ毎にアカウント情報を変える
===

```
vim ~/.gitconfig

[includeIf "gitdir:<対象ディレクトリ>"]
  path = ~/.gitconfig_private

vim ~/.gitconfig_private

[user]                                                                              name = local_user_name
  email = local_user_name@mail.address

```

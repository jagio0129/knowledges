Terafform 導入
===

[5分で分かるTerraform（Infrastructure as Code）](https://www.lac.co.jp/lacwatch/service/20200903_002270.html)

## tfenv install
Terraformのバージョンを切り替えられるツール。rbenvのようなもの。

https://beyondjapan.com/blog/2019/02/tfenv-change-version/

```sh
git clone https://github.com/tfutils/tfenv.git ~/.tfenv
# for bash
echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bash_profile
# for fish
echo 'set -x PATH $PATH $HOME/.tfenv/bin' >> ~/.config/fish/config.fish

# relogin

# 確認
tfenv --version
```

## Terraform install

```sh
# show installavle versions
tfenv list-remote

# install 0.13.1
tfenv install 0.13.1

# choose version
tfenv use 0.13.1

# 作成しておくとterraformバージョンを指定できる
touch .terraform-version
echo "0.13.1" > .terraform-version
```

## クレデンシャル情報のロード

Terraformでは以下のコードでアカウント情報を設定する

```
# main.tf
provider "aws" {
  access_key = "ACCESS_KEY_HERE"
  secret_key = "SECRET_KEY_HERE"
  region = "ap-northeast-1"
}
```

しかしコード中にクレデンシャル情報を含めてコミットすることはできない。

Terraformでは環境変数に情報をセットすることでコード中に含めることを回避する

```sh
set -Ux AWS_ACCESS_KEY_ID "key_id"
set -Ux AWS_SECRET_ACCESS_KEY "access_key"
set -Ux AWS_DEFAULT_REGION "ap-northeast-1"
```

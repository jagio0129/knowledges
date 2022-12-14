AWS CLI
===

## AWS アカウント作成
基本的には以下記事に沿って作成

- https://note.com/mc_kurita/n/n6d93b62fee28

練習がてらrootユーザでの操作は行わず、IAMでメインの操作を行うユーザを新規作成する。

- https://qiita.com/kzykmyzw/items/ca0c3276dfebb401f7d8

## インストール(Ubuntu 18.04)

```sh
# https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2-linux.html
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin
aws --version
  #=> aws-cli/2.0.23 Python/3.7.4 Linux/4.14.133-113.105.amzn2.x86_64 botocore/2.0.0
```

## Configure
```sh
aws configure
  AWS Access Key ID [None]: <access_key_id>
  AWS Secret Access Key [None]: <secret_access_key>
  Default region name [None]: ap-northeast-1
  Default output format [None]: json

# 確認
aws ec2 describe-vpcs  # VPC設定がjsonで取得できるはず
```

### autocomplete
```sh
# https://www.karakaram.com/aws-cli-getting-started/

# ~/.bashrc
complete -C '/usr/local/bin/aws_completer' aws

# ~/.config/fish/config.fish
complete --command aws --no-files --arguments '(begin; set --local --export COMP_SHELL fish; set --local --export COMP_LINE (commandline); aws_completer | sed \'s/ $//\'; end)'
```

## git-secrets install

gitでクレデンシャルな情報をコミットしようとすると警告が出るようになる

```sh
# install
cd
git clone https://github.com/awslabs/git-secrets.git
cd git-secrets
sudo make install

# apply git-secrets
cd YOUR_REPOSITORY
git secrets --install
git secrets --register-aws
```

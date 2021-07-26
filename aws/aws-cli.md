AWS CLI
===

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
# ~/.bashrc
complete -C '/usr/local/bin/aws_completer' aws

# ~/.config/fish/config.fish
complete --command aws --no-files --arguments '(begin; set --local --export COMP_SHELL fish; set --local --export COMP_LINE (commandline); aws_completer | sed \'s/ $//\'; end)'
```

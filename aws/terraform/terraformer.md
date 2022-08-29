Terraformer
===

手動で作成されたAWSリソースをTerraformに落とし込むためのツール

- 環境
  - Ubuntu18.04

### Installation

```shell
export PROVIDER=aws
curl -LO https://github.com/GoogleCloudPlatform/terraformer/releases/download/$(curl -s https://api.github.com/repos/GoogleCloudPlatform/terraformer/releases/latest | grep tag_name | cut -d '"' -f 4)/terraformer-${PROVIDER}-linux-amd64
chmod +x terraformer-${PROVIDER}-linux-amd64
sudo mv terraformer-${PROVIDER}-linux-amd64 /usr/local/bin/terraformer

terraformer --version
  version v0.8.21
```

### 使い方

```shell
# 適当にフォルダ作成
mkdir path/to/terraformer-sample && cd path/to/terraformer-sample

# 今回はDocumentDBをterraform化してみる
mkdir docdb cd && docdb
touch versions.tf

vim versions.tf
  terraform {
    required_version = ">= 0.1" # terraform1系はサポート外らしい
    required_providers {
      aws = {
        source = "hashicorp/aws"
        version = "3.63.0"
      }
    }
  }

terraform init

# import
#   AWSリソース一覧はここから(https://github.com/GoogleCloudPlatform/terraformer/blob/master/docs/aws.md)
terraformer import aws --resources=docdb --regions=ap-northeast-1

# generated配下にズラっとできる
```

### 参考サイト

- [GoogleCloudPlatform/terraformer](https://github.com/GoogleCloudPlatform/terraformer)
- [手で作られた AWS リソースをできるだけ簡単に Terraform に落とし込む](https://developers.cyberagent.co.jp/blog/archives/33331/)

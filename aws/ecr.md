ECR(Elastic Container Registory)
===

- Dockerイメージを保存するDocker Hubみたいなもの。
- Fargateへデプロイするコンテナは基本的にここに登録されたものを使う

## ECRへイメージを登録
- aws-cliを使う
- 環境にDockerが入っていること

```sh
export ECR_URI="account_id.dkr.ecr.ap-northeast-1.amazonaws.com"
export REGION="ap-northeast-1"
export ACCOUTN_ID="account_id"
export REPOSITORY="hoge_repository"

cd <your directory containing the Docker files>

# 認証
aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $ACCOUTN_ID.dkr.ecr.$REGION.amazonaws.com
  # => Login Succeeded

# Dockerfileからタグをつけてイメージを生成
sudo docker build -t $REPOSITORY .
# プッシュするイメージにタグ付け
docker tag $REPOSITORY:latest $ECR_URI/$REPOSITORY:latest
# push
docker push $ECR_URI/$REPOSITORY:latest
```

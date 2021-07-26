AWS CLI コマンド
===

## EC2
### 状態確認
```sh
# https://dev.classmethod.jp/articles/awscli-tips-ec2-start-stop/

aws ec2 describe-instance-status --instance-ids <instance_id>
```

### 起動
```sh
aws ec2 start-instances --instance-ids <instance_id>

# 起動完了まで結果を待つ
aws ec2 start-instances --instance-ids <instance_id> && aws ec2 wait instance-running --instance-ids <instance_id>

# 停止している全EC2インスタンス起動
aws ec2 start-instances --instance-ids $(aws ec2 describe-instances | jq -r '[.Reservations[].Instances[] | select(.State.Name == "stopped") | .InstanceId] | join(" ")')
```

### 停止
```sh
aws ec2 stop-instances --instance-ids <instance_id>

# 停止確認
aws ec2 describe-instances --instance-ids <instance_id> | jq '.Reservations[].Instances[] | {InstanceId, InstanceState: .State.Name}'

# 停止完了を待つ
aws ec2 stop-instances --instance-ids <instance_id> && aws ec2 wait instance-stopped --instance-ids <instance_id>

# 起動している全EC2インスタンス停止
aws ec2 stop-instances --instance-ids $(aws ec2 describe-instances | jq -r '[.Reservations[].Instances[] | select(.State.Name == "running") | .InstanceId] | join(" ")')
```

## RDS
https://devlog.arksystems.co.jp/2018/12/04/6469/

### 起動

```sh
aws start-db-instance --db-instance-identifier <インスタンス名>
```

### 停止
```sh
aws stop-db-instance --db-instance-identifier <インスタンス名>
```

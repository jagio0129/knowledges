Module
===

moduleとはなんぞやとか作り方とかは以下がわかりやすい

- [モジュールの使い方 - Terraformのきほんと応用](https://zenn.dev/sway/articles/terraform_biginner_modules)


terraformはすべての定義をmain.tfに記載することができる。が、見通しが悪くなるため別ファイルに分離することが望ましい。

```
- main.tf
- outputs.tf
- providers.tf
- variables.tf
```

これでも見通しが悪い場合はリソースごとに定義ファイルを分ける

```
- aws_routes.tf
- aws_vpc.tf
- aws_subnet.tf
- outputs.tf
- providers.tf
- variables.tf
```

ファイル数は増えるがメンテナンス性は高い。しかしこの構造ではそのうち破綻してしまう。そこでモジュールに分離する。

```
-- modules/
  |-- ecs_service/
  |-- iam_role/
  |-- rds/
  |-- security_group/
```

最初のうちは「modules」ディレクトリを作って育てていくことが推奨。別リポジトリへの分離は「責務が明確」「枯れていて変更頻度が低い」「設定を柔軟にオーバーライドできる」ようなモジュールに限定すべき。

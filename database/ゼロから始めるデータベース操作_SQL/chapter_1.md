第1章
===
- RDBでは必ず行単位でデータの読み書きを行う

## SQLの文と種類
- DDL(Data Defenition Language:データ定義言語)
  - DBやテーブルの作成・削除など
  - CREATE
  - DROP
  - ALTER: DBやテーブルなどの構成を変更する

- DML(Data Manipulation Language:データ操作言語)
  - テーブルの行を検索したり変更したりする
  - SELECT
  - INSERT
  - UPDATE
  - DELETE

- DCL(Data Control Language:データ制御言語)
  - テーブルに対して行った変更を確定したり取り消したりする
  - COMMIT:DBに対しておこなった変更を確定する
  - ROLLBACK:DBに対しておこなった変更を取り消す
  - GRANT:ユーザに操作の権限を与える
  - REVOKE:ユーザから操作の権限を奪う

第2章 SELECT
===

- SELECTは「テーブルにあるデータから必要なものだけを取り出す」


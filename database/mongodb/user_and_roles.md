ユーザ管理
===

デフォルトだとユーザ認証はoffになっている。有効にしたい場合は設定ファイル等で有効にしてあげる必要がある

```
security:
   authorization: enabled
```

## ユーザ作る場合

### 管理者ユーザの追加
```sh
# mongoシェルログイン
mongo

# ユーザ管理者を作成
> use admin
> db.createUser({
    user: 'admin',
    pwd: 'password',
    roles: [{
        role: 'userAdminAnyDatabase',
        db: 'admin'
    }]
})

> show users
{
        "_id" : "admin.admin",
        "userId" : UUID("2e366f67-e29b-4576-a682-e4a7756dd8c9"),
        "user" : "admin",
        "db" : "admin",
        "roles" : [
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                }
        ],
        "mechanisms" : [
                "SCRAM-SHA-1",
                "SCRAM-SHA-256"
        ]
}

```

### 通常ユーザの追加

```sh
# まずはユーザ管理者でログイン
mongo
> use admin
> db.auth("admin", "password")

# DBを選択してユーザ作成
#   アプリケーションはrole: "readWrite"で事足りる
use test
db.createUser({
    user: "user1",
    pwd: "password",
    roles: [{
        role: "readWrite", db: "test"
    }]
})
```

## ロール

MongoDBは、ロールベースの認証によってデータやコマンドへのアクセスを許可する。また、[任意のロールを定義する](https://www.mongodb.com/docs/manual/core/security-user-defined-roles/#std-label-user-defined-roles)こともできる。

MongoDB の組み込みロールは、そのロールのデータベースにあるすべての非システムコレクションへのアクセスをデータベースレベルで定義し、すべてのシステムコレクションへのアクセスをコレクションレベルで定義する。

MongoDBでは組み込みの「データベースユーザ」と「データベース管理者」を提供し、それ以外のroleをすべてadminデータベースにのみ提供する。


| 種別                          | ロール名             | 概要                                                                                                                                  |
| ----------------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Database User Roles           | read                 | システム系のDBやコレクション以外のデータを読む権限                                                                                    |
|                               | readWrite            | システム系のDBやコレクション以外のデータを読み書きする権限                                                                            |
| Database Administration Roles | dbAdmin              | データベースの管理権限                                                                                                                |
|                               | dbOwner              | システム全体の管理権限。readWrite、dbAdmin、userAdminロールと関連付いている。                                                         |
|                               | userAdmin            | ユーザやロールの管理者                                                                                                                |
| Cluster Administration Roles  | clusterAdmin         | クラスタの管理権限。clusterManager、clusterMonitor、hostManagerロールと紐付く。                                                       |
|                               | clusterManager       | クラスタの運用管理権限                                                                                                                |
|                               | clusterMonitor       | クラスタのモニタリングツールに対する参照権限                                                                                          |
|                               | hostManager          | サーバの運用管理権限                                                                                                                  |
| Backup and Restore Roles      | backup               | データのバックアップ権限                                                                                                              |
|                               | restore              | バックアップデータからのリストア権限                                                                                                  |
| ALL-Database Roles            | readAnyDatabase      | 全てのDBの読み取り権限                                                                                                                |
|                               | readWriteAnyDatabase | 全てのDBの読み書き権限                                                                                                                |
|                               | userAdminAnyDatabase | 全てのDBのユーザ管理権限                                                                                                              |
|                               | dbAdminAnyDatabase   | 全てのDBのデータベース管理権限。dbAdminと同等。                                                                                       |
| Superuser Roles               | root                 | システム全体の管理権限。readWriteAnyDatabase、dbAdminAnyDatabase、userAdminAnyDatabase、clusterAdmin roles、restore、backupが紐付く。 |
| Internal Role                 | \_\_system           | 内部処理用の権限。割り当ててはいけない。                                                                                              |

> ※ 詳細は[Built-In Roles](https://www.mongodb.com/docs/manual/reference/built-in-roles/#database-user-roles)

## 参考サイト

- [Manage Users and Roles - MongoDB](https://www.mongodb.com/docs/manual/tutorial/manage-users-and-roles/)
- [MongoDB の アクセス制御 (ユーザー認証) を 有効化する 方法](https://garafu.blogspot.com/2017/01/enable-mongodb-auth-control.html#mongodbconfig)

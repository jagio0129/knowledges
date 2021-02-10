デッドロックの確認方法
===

## ロック原因を特手出来るようにする

```sh
# vi /etc/my.cnf

# デッドロック関連のログをエラーログに出力させる
innodb_print_all_deadlocks=ON

# ロックモニターの有効化：SHOW ENGINE INNODB STATUSでロック原因を特定できるようにする
innodb_status_output=ON
innodb_status_output_locks=ON
```

`SHOW ENGINE INNODB STATUS`で確認できる。dockerなら`docker exec -it DBNAME mysql -u USERNAME -pPASSWORD -e "SHOW ENGINE INNODB STATUS"`で直接取れる。

## ログの見方

[ここ](https://qiita.com/h-oikawa/items/91e2401fad5d93262f6f)の説明で充分そうなので書略

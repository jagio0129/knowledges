サンプルデータを使う方法
===

## Samples
- [Other MySQL Documentation](https://dev.mysql.com/doc/index-other.html)
- [MySQL Sample Database](https://www.mysqltutorial.org/mysql-sample-database.aspx)

基本的には.sqlファイルを落としてくればよい

```sh
wget https://www.mysqltutorial.org/wp-content/uploads/2018/03/mysqlsampledatabase.zip
unzip mysqlsampledatabase.zip
```

## dockerで使う

以下のようなdocker-composeを想定

```yml
ersion: '3'

services:
  mysql:
    image: mysql:5.7.21
    container_name: mysql_host
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: root
      TZ: 'Asia/Tokyo'
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    volumes:
      - ./docker/db/data:/var/lib/mysql
      - ./docker/db/my.cnf:/etc/mysql/conf.d/my.cnf
      - ./docker/db/sql:/docker-entrypoint-initdb.d
    ports:
      - 3306:3306
```

docker/db/sql/に.sqlファイルを設置して起動すれば良い。もし設置する前に起動してしまった場合はdocker/db/data/をすべて削除してから再起動すれば適応される。

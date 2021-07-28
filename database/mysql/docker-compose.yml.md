MySQL docker-compose
===

```yml
version: '3'

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

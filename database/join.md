内部結合・外部結合
===

凡例テーブル

**Company**
id|name
---|---
1|Amazon
2|Apple
3|Facebook
4|Google

**User**
id|name|company_id
---|---|---
1|Steve Jobs|2
2|Jeff Bezos|1
3|Stephen Gary Wozniak|2
4|Tim Cook|2
5|Mark Zuckerberg|3
6|Larry Page|4
7|Sergey Brin|4
8|Bill Gates|5


## 内部結合

![inner_join](https://sql-oracle.com/sqlserver/wp-content/uploads/2018/03/inner-join-%E3%83%99%E3%83%B3%E5%9B%B3.jpg)

```ruby
User.joins(:company)
```

```sql
SELECT * FROM users
INNER JOIN companies
ON users.company_id = companies.id;
```

- ON節で結合の条件を指定する
  - ↑の場合ではユーザが持つcompany_idとcompany.idが同じもの(つまり関連付けされている)レコードを結合する

id|name|company_id|id|name
---|---|---|---|---
1|Steve Jobs|2|2|Apple
2|Jeff Bezos|1|1|Amazon
3|Stephen Gary Wozniak|2|2|Apple
4|Tim Cook|2|2|Apple
5|Mark Zuckerberg|3|3|Facebook
6|Larry Page|4|4|Google
7|Sergey Brin|4|4|Google

- 内部結合の場合、ベースとなるテーブル(User)から条件にマッチするレコードがないものは削除される。
  - user.id = 8は条件(`ON users.company_id = companies.id`)に該当するCompanyが存在しないため除外されている。

### ユースケース

## 外部結合
- joinsの内部結合に加え、どちらかのテーブルにしか存在しないものも取得できる

### LEFT OUTER JOIN
![left_outer_join](https://sql-oracle.com/sqlserver/wp-content/uploads/2018/03/left-join-%E3%83%99%E3%83%B3%E5%9B%B3.jpg)


### RIGHT OUTER JOIN
![right_outer_join](https://sql-oracle.com/sqlserver/wp-content/uploads/2018/03/right-join-%E3%83%99%E3%83%B3%E5%9B%B3.jpg)


### joins
```sql
Company.joins(:users)
  Company Load (1.2ms)  SELECT `companies`.* FROM `companies` INNER JOIN `users` ON `users`.`company_id` = `companies`.`id`
```
- 内部結合してくれる
  - ２つのテーブルをガッチャンコ
- ベースとなるテーブルから、条件にマッチするレコードがないものは削除される
  - Company - Userが1対Nで紐付いていて`Company.joins(:users)`とすれば、ユーザの紐付いている企業だけ取れる
- マッピングされたレコードが返るのでテーブルよりレコード数が多く帰ってくる場合がある
  - `Company.joins(:users)`だとCompanyに紐づくユーザ数の数だけ返ってくる
- joins先のテーブルのカラムの値でしぼりこんだりできる
  - `Company.joins(:users).where(users: {id: [*1..3]})`

### includes
```sql
Company.includes(:users)
  Company Load (1.0ms)  SELECT `companies`.* FROM `companies`
  User Load (1.1ms)  SELECT `users`.* FROM `users` WHERE `users`.`company_id` IN (1, 2)
```
- まずCompanyをロードしてそのCompanyに紐づくUserすべてロード。データの先読み
- referencesを付けることでLEFT OUTER JOINになる
```
Company.includes(:users).references(:users).where('users.id < ?', 3)
  SQL (1.1ms)  SELECT `companies`.`id` AS t0_r0, `companies`.`name` AS t0_r1, `companies`.`code` AS t0_r2, `companies`.`guid` AS t0_r3, `companies`.`created_at` AS t0_r4, `companies`.`updated_at` AS t0_r5, `users`.`id` AS t1_r0, `users`.`name` AS t1_r1, `users`.`email` AS t1_r2, `users`.`user_id` AS t1_r3, `users`.`guid` AS t1_r4, `users`.`company_id` AS t1_r5, `users`.`provider` AS t1_r6, `users`.`created_at` AS t1_r7, `users`.`updated_at` AS t1_r8 FROM `companies` LEFT OUTER JOIN `users` ON `users`.`company_id` = `companies`.`id` WHERE (users.id < 3)
```
  - この場合はユーザIDが3以下のユーザが紐づく企業のみが返ってくる
- where句が文字列でない場合はreferences付けなくてもLEFT OUTER JOINとなる
```
Company.includes(:users).where(users: {id: [*1..3]})
```
- includes + joinsという使い方もできる
```
Company.includes(:users).joins(:users).where(:users: {id: [*1..3]})
```

### 外部結合

INSERT INTO ... ON DUPLICATE KEY UPDATE ...
===

## 基本構文

```sql
INSERT INTO <table> (columns...) VALUES (values...) ON DUPLCIATE KEY UPDATE duplicate_keys
```

なければINSERT、あればUPDATEをワンステートで行うことができる構文。

新規行の場合はそのままINSERTされる。UNIQUEインデクスまたはPRIMARYキーに重複した値を挿入しようとした場合、古い行がUPDATEされる


## 利用に際して気をつけるとこ

いくつか気をつける事がある

### ギャップロックの誘発

`ON DUPLICATE KEY UPDATE`構文に限らず、INSERTで複数レコードを追加する場合に発生する可能性がある

https://ichirin2501.hatenablog.com/entry/2015/12/24/164916

### インクリメントカラムがある場合飛び値が発生する

https://ameblo.jp/stylagy/entry-12434472506.html

これに伴いインクリメントカラムの最大値に達してしまう場合もある(BIGINTなら4000京ぐらいまで大丈夫)

### レプリケーションで整合性が取れなくなる可能性

行ベースレプリケーション(RBR)なら起きない。MIXEDならRBR扱いなので問題ない。ステートメントベースレプリケーション(SBR)だと不整合が起きる

### 謎のデッドロック

UPDATEステートが常に発生する実行で謎のデッドロックが起きる。インデックス最大値以上のロックを取るギャップロックによるデッドロックっぽいが明言できない。innodb_auto_inc_mode=0にすると解消するがAUTO_INCREMENTロックが厳しくなるので性能が落ちる

## 参考サイト

- [13.2.5.3 INSERT ... ON DUPLICATE KEY UPDATE 構文](https://dev.mysql.com/doc/refman/5.6/ja/insert-on-duplicate.html)

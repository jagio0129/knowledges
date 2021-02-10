MySQL クエリチューニング
===

## 問題の発見方法【TBD】
- スロークエリ出力するように設定する
- デッドロックを検知できるようにする
- peroconaで解析
- show full processlist
- zibbixなどで負荷を監視
    - golangが入っている環境なら[ps-top](https://blog.vtryo.me/entry/ps-top-use)とかおすすめ
- ScoutAPMなどでパフォーマンスを確認

手元のMySQLで各種ログを出したい場合はmy.confに設定すればよい。参考は[こちら](https://www.yuulinux.tokyo/10010/#i-3)

## チューニング方法あれこれ

- EXPLAINで実行計画を確認
  - EXPLAINの見方は下記に記載

- indexが張られているか。使われているか。
  - pk以外のカラムに張る場合は[カーディナリティ](https://qiita.com/hmatsu47/items/2d44c173a9114fd06853)の高いカラムに貼ること
  - indexをむやみやたらと貼ればいいというものではない
      - 参照は早くなるが、書き込みは遅くなる。

- 複合キーで検索かける場合は複合indexを張ること
  - 複合indexは順番通りに条件句を記載すること
  - `CREATE UNIQUE INDEX employee_pk ON EMPLOYEES (employee_id, department_id)`なら`SELECT employee_id, department_id`の順で。逆にするとindexが効かない
- `select * from ~`は必要なカラムだけに絞れないか
  - ARのpluck検討
- join
  - 必要のないテーブルを結合していないか
      - where句で肉抜きせずにONで結合のタイミングで肉抜きできないか考える
      - [テーブル結合についての備忘録](https://qiita.com/mounntainn/items/c6ca4184acc4839f9679)
      - 外部結合でのJOIN ONは危険なのでちゃんと確認すること
  - MySQLのjoinアルゴリズムはNestedLoopということを念頭に置く
    - いわゆる多重ループ
    - ORDERBYとともに使うと思わぬ遅さにつながる【TBD】
    - https://qiita.com/yuku_t/items/208be188eef17699c7a5

- IN句は左側からヒットしそうなキーを並べる
  - ヒットしなければ何度も検索され続けてしまう
- 「in句」で実装されている部分を「exists句」で実装できるかどうか検討してみる
  - exists句はすべてのレコードを検索する必要が無いので速度改善できることがある。
  - distinctも同様にexists, not existsに変更できるかも

- WHERE句の左辺には算術演算子を使わない
  - `WHERE  total_price - 1000 > 10000` : ☓
  - ` WHERE  total_price  > 10000 + 1000` : ◎

- 「MySQLサブクエリは遅い」は一概には言えない
  - ほんとに遅いのはDEPENDENT SUBQUERYだけ
  - MySQL5.6以上からオプティマイザが進化して、DEPENDENT SUBQUERYにならない可能性もある。
      - [MySQL のサブクエリって、ほんとに遅いの？](https://dev.classmethod.jp/articles/mysql-subquery-delay/)
      - [開発スピードアクセル全開ぶっちぎり！日本よ、これがMySQL 5.6だッ！！](http://nippondanji.blogspot.com/2012/10/mysql-56.html)

#### 参考サイト
- [MySQLのEXPLAINを徹底解説!!](http://nippondanji.blogspot.com/2009/03/mysqlexplain.html)
- [MySQLのexplainとかについてしらべたときのメモ](https://qiita.com/lastcat_/items/de7b530a94fbcf9ba646#%E3%81%9D%E3%82%82%E3%81%9D%E3%82%82order-by%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%AA%E3%81%84%E3%81%AE%E3%81%ABusing-filesort%E3%81%8C%E5%87%BA%E3%81%A6%E3%81%84%E3%82%8B%E7%82%B9%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6)
- [SQLチューニング～お手軽に使えるヒント句まで](https://qiita.com/kite_999/items/05136fadebc4048e3bb6)
- [初心者はこれだけ覚えておけ！ SQLチューニング観点 10選](https://bebee5.com/%E5%88%9D%E5%BF%83%E8%80%85%E3%81%AF%E3%81%93%E3%82%8C%E3%81%A0%E3%81%91%E8%A6%9A%E3%81%88%E3%81%A6%E3%81%8A%E3%81%91%EF%BC%81-sql%E3%83%81%E3%83%A5%E3%83%BC%E3%83%8B%E3%83%B3%E3%82%B0%E8%A6%B3/)
- [MySQL データベースの負荷対策/パフォーマンスチューニング備忘録 インデックスの基礎〜実践](https://qiita.com/marnie_ms4/items/576055abc355184c51a1)
- [はじめてのMySQL。意外と知らない3つのTips](https://developers.gnavi.co.jp/entry/mysql-tips)
- [8.8.2 EXPLAIN 出力フォーマット](https://dev.mysql.com/doc/refman/5.6/ja/explain-output.html)
- [ヤフー社内でやってるMySQLチューニングセミナー大公開](https://www.slideshare.net/techblogyahoo/mysql-58540246)
- [MySQL with InnoDB のインデックスの基礎知識とありがちな間違い](https://techlife.cookpad.com/entry/2017/04/18/092524)
- [SQL実行計画の疑問解決には「とりあえずEXPLAIN」しよう](https://thinkit.co.jp/article/9658)
- [SQL Joinサンプル集　Joinで遅いSQLの原因を調べる方法](https://style.potepan.com/articles/14926.html)

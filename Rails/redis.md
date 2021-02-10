# redis
redisはインメモリにデータを展開し読み書きを高速化していますが、電源が切れるなどするとデータは揮発してしまいます。

そこで変更を随時追記専用ファイルに書き出すことで復旧することができます。

```
appendonly yes
```

以降、Redis はデータセットを変更するコマンド(e.g. SET)を受け付ける都度、AOF に追記します。Redis を再起動したときは、AOF ファイルをリプレイして状態を再構成します。

https://redis-documentasion-japanese.readthedocs.io/ja/latest/topics/persistence.html
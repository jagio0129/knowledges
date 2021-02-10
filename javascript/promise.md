---
title: Promise
tags: ["javascript", "coffeescript", "非同期処理"]
excerpt: 非同期処理
---

macのターミナルでjsを動かすために以下を実行する

```
cd /usr/local/bin
ln -s /System/Library/Frameworks/JavaScriptCore.framework/Versions/A/Resources/jsc ./

jcs <file.js>
```

## 非同期処理
通常プログラム上から下へ順番に一つずつ処理する。これを同期処理という。

```javascript
print (1);
print (2);
print (3);

1
2
3
```

しかし、途中で時間が掛かる処理が走るとそれ以降の処理はその処理が終わるまで実行されない。

```javascript
function sleep(waitMsec) {
  var startMsec = new Date();
  while (new Date() - startMsec < waitMsec);
  print(2)
}

print (1);
sleep(5000);
print (3);

1
// 5s wait
2
3
```

このような欠陥を解消するために「**非同期処理**」という仕組みが用意されている。非同期処理であれば、結果を待たずして次の処理が行われる。


## 参考サイト
- [OSXのターミナルでJavascriptを動かす](https://qiita.com/wnoooo/items/ef88433e84b79e362a8d)
- [【JavaScript入門】誰でも分かるPromiseの使い方とサンプル例まとめ！](https://www.sejuku.net/blog/52314)
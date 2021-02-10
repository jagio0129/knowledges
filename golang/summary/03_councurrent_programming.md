---
title: 並行プログラミング
tags: [golang]
excerpt: golangの並行プログラミングについて
---

# 並行プログラミングの基本
複数の処理を効率よく行うために、Goは言語自体が並行処理に必要な機能をサポートしている。

# ゴルーチン
Goにはゴルーチン(Goroutine)という軽量スレッドの仕組みがある。main()関数も1つのゴルーチンの中で実行されている。go構文を用いて、任意の関数を別のゴルーチンとして起動することで、処理を並行して走らせることができる。

## ゴルーチンを使わない場合
例として、3つのサイトにアクセスし、そのステータスコードを表示するプログラムを考える。
```go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func main() {
    urls := []string{
        "http://example.com",
        "http://example.net",
        "http://example.org",
    }
    for _, url := range urls {
        res, err := http.Get(url)
        if err != nil {
            log.Fatal(err)
        }
        defer res.Body.Close()
        fmt.Println(url, res.Status)
    }
}
```
http.Get()は同期処理であるので、この方法では別々のサイトにリクエストしているにも関わらず、前のレスポンスが帰らないと次のリクエストに進むことができない

![image](http://image.gihyo.co.jp/assets/images/dev/feature/01/go_4beginners/0005/001.png)

## ゴルーチンを使った場合
```go
func main() {
    urls := []string{
        "http://example.com",
        "http://example.net",
        "http://example.org",
    }
    for _, url := range urls {
        // 取得処理をゴルーチンで実行する
        go func(url string) {
            res, err := http.Get(url)
            if err != nil {
                log.Fatal(err)
            }
            defer res.Body.Close()
            fmt.Println(url, res.Status)
        }(url)
    }
    // main()が終わらないように待ち合わせる
    time.Sleep(time.Second)
}
```
ここではmain()が実行されたときに内部で3つのゴルーチンを起動しているが、起動が終わってもゴルーチン内で処理が行われている間もmain()は先に進んでしまうため、待ち合わせのためにtime.Sleep()を読んで1秒間main()を止めている。

![image](http://image.gihyo.co.jp/assets/images/dev/feature/01/go_4beginners/0005/002.png)

## sync.WaitGroup
time.Sleep()でmain()を1秒間待たせていたが、実際に待ちたいのはhttp.Get()を行っている全てのゴルーチンの終了。そこでsync.WaitGroupを利用する。

sync.WaitGroupはAdd()でカウントを増やしDone()でカウントを減らし、Wait()でカウントがゼロになるまで待ち合わせをする。

```go
func main() {
    wait := new(sync.WaitGroup)
    urls := []string{
        "http://example.com",
        "http://example.net",
        "http://example.org",
    }
    for _, url := range urls {
        // waitGroupに追加
        wait.Add(1)
        // 取得処理をゴルーチンで実行する
        go func(url string) {
            res, err := http.Get(url)
            if err != nil {
                log.Fatal(err)
            }
            defer res.Body.Close()
            fmt.Println(url, res.Status)
            // waitGroupから削除
            wait.Done()
        }(url)
    }
    // 待ち合わせ
    wait.Wait()
}
```

## チャネル
**複数のゴルーチン間でやり取りをしたい場合**、組み込みのチャネル(channel)という機能を用いることで、メッセージパッシング(情報をメッセージとして送受信する)によってデータを送受信できる。チャネルはmake()関数に型を指定して生成することで、その型のデータの書き込みと読み出しができる。

```go
// stringを扱うチャネルを生成
ch := make(chan string)

// チャネルにstringを書き込む
ch <- "a"

// チャネルからstringを読み出す
message := <- ch
```

今回の場合は、ゴルーチンないで取得したステータスコードをチャネルに書き込みそれをmain()のゴルーチンで読み出すことで、ゴルーチン感でデータを受け渡すことができる。
```go
func main() {
    urls := []string{
        "http://example.com",
        "http://example.net",
        "http://example.org",
    }
    statusChan := make(chan string)
    for _, url := range urls {
        // 取得処理をゴルーチンで実行する
        go func(url string) {
            res, err := http.Get(url)
            if err != nil {
                log.Fatal(err)
            }
            defer res.Body.Close()
            statusChan <- res.Status
        }(url)
    }
    for i := 0; i < len(urls); i++ {
        fmt.Println(<-statusChan)
    }
}
```

ゴルーチンの中でstatusChanに値が書き込まれるまで、main()の中では値を読み出すことができない。この場合、main()内ではstatusChanの読み出しが３回完了するまで処理がブロックされるため、waitGroupのような待ち合わせの処理は必要ない。

![image](http://image.gihyo.co.jp/assets/images/dev/feature/01/go_4beginners/0005/003.png)

## チャネルを返すパターン
これまではmain()ないの匿名関数でHTTPのGETを行っていたが、この処理を別の関数にし、関数が内部で生成したチャネルを返すように実装してみる。
```go
func getStatus(urls []string) <-chan string {
    // 関数でチャネルを生成
    statusChan := make(chan string)
    for _, url := range urls {
        go func(url string){
            res, err := http.Get(url)
            if err != nil {
                log.Fatal(err)
            }
            defer res.Body.Close()
            statusChan <- res.Status
        }(url)
    }
    return statusChan // チャネルを返す
}

func main(){
    urls := []string{
        "http://example.com",
        "http://example.net",
        "http://example.org",
    }
    statusChan := getStatus(urls)

    for i := 0; i< len(urls); i++{
        fmt.Println(<-statusChan)
    }
}
```
まず、getStatus()内で結果を渡すためのstatus Chanを生成する。次に非同期に行う処理を匿名関数にし、リクエストをそれぞれ別のゴルーチンで実行する。関数自体はstatusChanを返して終了し、起動されたゴルーチンが内部でstatusChanに結果を書き込んでいる。

main()は、関数を呼び出すと同時に結果を受信するチャネルを受取、それをfor文内で読み出す。これにより、main()が非常にスッキリと記述でき、ロジックの大半はgetStatus()に隠蔽できる。

また、このときgetStatus()はmain()がチャネルに値を書き込むことを想定していない。こうした場合は、getStatus()の戻り値を<-chan stringと読み出し専用のチャネルにすることで、main()がこのチャネルに値を書き込むことを型レベルで防ぐことができる。

このパターンはチャネルを用いる場合によく使うので覚えておくこと。

## select文を用いたイベント制御
複数のチャネルに対する読み出しや書き込みを同時に制御するためにはselect文を用いる。select分はfor文と組み合わせて使う場合は多くある。

### case
複数の操作をselect文のcaseに指定しておくと、いずれかのcaseの操作が実行されたときに、該当する処理が実行される。どれか一つcaseが実行されない限りは、select文はブロックする。
```go
ch1 := make(chan string)
ch2 := make(chan string)
for{
    select{
        case c1 := <-ch1 :
            // ch1からデータを読み出したときに実行される
        case c2 := <-ch2:
            // ch2からデータを読み出したときに実行される
        case c3 := <-ch3:
            // ch2にデータを書き込んだときに実行される
    }
}
```

### default
caseの最後にdefaultを記述すると、実行するcaseがなかった場合はdefaultが実行される。defaultの実行が終わるとselect文の処理が終わるため、select文がブロックされなくなる。

```go
ch1 := make(chan string)
ch2 := make(chan string)
for {
    select {
    case c1 := <-ch1:
        // ch1からデータを読み出したときに実行される
    case c2 := <-ch2:
        // ch2からデータを読み出したときに実行される
    case ch2 <- "c":
        // ch2にデータを書き込んだときに実行される
    default:
        // caseが実行されなかった場合に実行される
    }
}
```

### タイムアウト
for/select文とbreakを用いて実装する代表的な例はタイムアウト処理
```go
func main() {
    // 1秒後に値が読み出せるチャネル
    timeout := time.After(time.Second)
    urls := []string{
        "http://example.com",
        "http://example.net",
        "http://example.org",
    }
    statusChan := getStatus(urls)

LOOP: // ラベル名は任意
    for {
        select {
        case status := <-statusChan:
            fmt.Println(status) // 受信したデータを表示
        case <-timeout:
            break LOOP // このfor/selectを抜ける
        }
    }
}
```

timeパッケージにあるtime.After()関数は、時間指定するとその時間後にデータを書き込むチャネルを返す関数。このチャネルの読み出しをselect文に登録することで、タイムアウト処理を実現できる。

先程のstatusChanの読み出しを無限forループ内のselect文で受け取るようにする。ステータスを受け取った場合はそれが表示されるが、1秒後にtimeoutから値が読み出せると、そこでfor/select文を抜けてHTTPリクエストが全て終わっている/いないにかかわらず、プログラムを終わらせる事ができる。

注意点として、caseの中でbreakを呼ぶと、select文は抜けられるが、その外側のfor文は抜けられない。そこでfor分にラベルを付け、breakでそのラベルを指定することで、caseからfor/select文を抜けることができる。returnで関数毎抜けることもできるが、ラベルのbreakもよく使うパターンなので覚えておくこと。

## チャネルバッファ
make()でチャネルを生成するときに、バッファを指定できる。バッファとは、そのチャネルが内部に保持できるデータの数。

### バッファなしチャネル
バッファを指定せずにmake()で生成したチャネルは、内部に値を保持しておくことができない。

次の場合はmain()内でチャネルの読み出す側に先に到達するが、ゴルーチン内で値が書き込まれるまでそこで1秒間ブロックする。
```go
func main() {
    ch := make(chan string)
    go func() {
        time.Sleep(time.Second)
        ch <- "a" // 1秒後にデータを書き込む
    }()
    <-ch // 1秒後にデータが書き込まれるまでブロック
}
```

逆に次の場合は、main()内でチャネルに書き込む側に先に到達するが、ゴルーチン内でその値が読み出されるまでそこで1秒間ブロックする
```go
func main(){
    ch := make(chan string)
    go func(){
        time.Sleep(time.Second)
        <-ch // 1病後にデータを読み出す
    }()
    ch <- "a" //1秒後にデータが読み出されるまでブロック
}
---
title: 言語仕様
tags: [golang]
excerpt: golangの言語仕様について
---

# 2.言語仕様
## インポート
```go
import(
    "fmt"
)

func main(){
    fmt.Println("hello world")
}
```
インポートしたパッケージ内には、`パッケージ名.Hoge()`でアクセス。

### オプションの指定
```go
import(
    f "fmt"
    _ "github.com/hoge/fuga"
    . "strings"
)

func main(){
    // fmt.Println() => f.Println()
    // strings.ToUpper() => ToUpper()
    f.Println(ToUpper("hello world")) // => "HELLO WORLD"
}
```

- `_`はインポートしたパッケージをコンパイルしないことを意味する。
- インポートしたパッケージを使用していないとエラーとなる。

## 組み込み型

型|説明
---|---
uint8|8ビット符号なし整数
int8|8ビット符号あり整数
float32|32ビット浮動小数
comlex64|64ビット複素数
byte|uint8のエイリアス
rune|Unicodeのコードポイント
uint|32か64ビットの符号なし整数
int|32か64ビットの符号あり整数
uintptr|ポインタ値用符号なし整数
error|エラーを表すインターフェイス

- runeは他の言語でいうchar
- stringが""、runeは''、複数行ヒアドキュメントは``

## 変数
```go
var message string = "hello world" // `var 変数 型`が基本

// 複数宣言
var (
    a string = "aaa"
    b = "bbb"
    c = "ccc"
)
```

### 関数内部での宣言と初期化
関数内部では、varと型宣言を`:=`で省略できる

```go
func main(){
    // var message string = "hello world"
    message := "hello world"
}
```

## 定数
```go
func main(){
    const Hello string = "hello"
    Hello = "bye"  // cannot assign to Hello
}
```

## ゼロ値
明示的に値を初期化しなかった場合の値。型ごとに異なる。

型|ゼロ値
---|---
整数型|0
浮動小数点型|0.0
bool|false
string|""
配列|各要素がゼロ値の配列
構造体|各フィールドがゼロ値の構造体
その他の型|nil

## if
goではif条件部に`()`はいらない

```go
func main() {
    a, b := 10, 100
    if a > b {
        fmt.Println("a is larger than b")
    } else if a < b {
        fmt.Println("a is smaller than b")
    } else {
        fmt.Println("a equals b")
    }
}
```

## for
goではfor条件部に`()`はいらない

```go
func main() {
    for i := 0; i < 10; i++ {
        fmt.Println(i)
    }
}
```

## whileもforで
```go
n := 0
for n < 10 {
    fmt.Printf("n = %d\n", n)
    n++
}
```

## 無限ループ
```go
for {
    doSomething()
}
```

## break, continue
- ループの終了 => break
- 最初から処理を再開 => continue

```go
func main() {
    n := 0
    for {
        n++
        if n > 10 {
            break // ループを抜ける
        }
        if n%2 == 0 {
            continue // 偶数なら次の繰り返しに移る
        }
        fmt.Println(n) // 奇数のみ表示
    }
}
```

## switch
```go
func main() {
    n := 10
    switch n {
    case 15:
        fmt.Println("FizzBuzz")
    case 5, 10:
        fmt.Println("Buzz")
    case 3, 6, 9:
        fmt.Println("Fizz")
    default:
        fmt.Println(n)
    }
}
```
- どのcaseにも該当しなかったらdefaultが実行される
- 他の言語と違い、**breakを書かなくても該当case内しか実行されない**
    - 次のcaseに処理を進めたいときは`fallthrough`と明示する

# 関数
- `func`で始まる
```go
// 引数なし
func hello(){}

// 引数あり
func hello(i, j int){}

//　引数・戻り値あり
func hello(i, j int) int {}
```

## 複数の値を戻り値とする
```go
func swap(i,j int)(int, int){
    return j, i
}

func main(){
    x,y := 3,4
    x,y = swap(x,y)
    fmt.Println(x,y) // => 4,3

    x = swap(x,y)   // コンパイルエラー

    x, _ = swap(x,y) //第二戻り値を無視
    fmt.Println(x)  // => 3
}
```

## エラーを返す関数
関数が多値を返せることを利用して、内部で発生したエラーを戻り値で表現する。**関数の処理に成功した場合はエラーはnil**にし、**異常があった場合はエラーだけに値が入り、他方はゼロ値**になる。

```go
func main(){
    file, err := os.Opne("hello.go")
    if err != nil {
        // エラー処理
        // returnなどで処理を別の場所に抜ける
    }
    // fileを用いた処理
}
```
- **複数の値を返す場合もエラーを最後にする慣習がある**
- 以上を戻り値で表現できない場合については、パニックとリカバリ

## 名前付き戻り値
- 戻り値にあらかじめ名前をつけることができる

```go
func div(i,j int)(result int, err error)
```
- 関数内ではゼロ値で初期化された変数として扱うことができる
- returnを明示する必要はない

```go
func div(i, j int) (result int, err error) {
    if j == 0 {
        err = errors.New("divied by zero")
        return // return 0, errと同じ
    }
    result = i / j
    return // return result, nilと同じ
}
```
- 戻り値に名前を付けても、returnのあとに戻す値を明示することも可能

## 関数リテラル
- 関数リテラルを用いると、無名関数を作ることができる
```go
func main(){
    func(i,j int){
        fmt.Println(i+j)
    }(2,4)
}
```

- 関数を変数に代入したり、関数の引数に渡すことができる。
```go
var sum func(i,j int) = func(i,j int){
    fmt.Println(i+j)
}

func main(){
    sum(2,4)
}
```

# 配列
```go
// 変数 [添字]型
var arr [4]string
arr[0] = "a"
fmt.Println(arr[0]) // => "a"

// 同時に宣言と初期化
arr := [4]string{"a","b","c","d"}
// sizeを暗黙的に指定
arr := [...]string{"a","b","c","d"}
```

# スライス
- スライスは可変長配列として扱うことができる
```go
// スライスの宣言
var s []sring

// 宣言と初期化
s := []string{"a","b"}
```

## append()
- スライスの末尾に値を追加吸う場合に使用する
```go
var s []string
s = append(s, "a")
s = append(s, "b","c")
fmt.Println(s)  // => [a b c]
```

## range
- 配列やスライスに格納された値を、**先頭から順番に処理するような場合**は、添字のアクセスの代わりにrangeを使用する。
```go
var arr [4]string

arr[0] = "a"
arr[1] = "b"
arr[2] = "c"
arr[3] = "d"

for i, s := range arr {
    // i = 添字, s = 値
    fmt.Println(i, s)
}
/* =>
0 a
1 b
2 c
3 d
*/
```

## 値の切り出し
- string，配列，スライスから，値を部分的に切り出すことができる
```go
s := []int{0, 1, 2, 3, 4, 5}
fmt.Println(s[2:4])      // [2 3]
fmt.Println(s[0:len(s)]) // [0 1 2 3 4 5]
fmt.Println(s[:3])       // [0 1 2]
fmt.Println(s[3:])       // [3 4 5]
fmt.Println(s[:])        // [0 1 2 3 4 5]
```

## 可変長引数
```go
func sum(nums ...int) (result int) {
    // numsは[]int型
    for _, n := range nums {
        result += n
    }
    return
}

func main() {
    fmt.Println(sum(1, 2, 3, 4))  // 10
}
```

# マップ
- 値をkey-valueの対応で保存するデータ構造
```go
var month map[int]string = map[int]string{}

month[1] = "January"
month[2] = "February"
fmt.Println(month)      // => map[1:January 2:February]
```

## マップの操作
```go
jan := month[1]
fmt.Println(jan) // January
```

- 第二戻り値を受け取ると、指定したキーがこのマップに格納されているかをboolで返す
```go
_, ok := month[1]
if ok {
    // データがあった場合
}
```

- マップからデータを消す場合は`delete()`
```go
delete(month, 1)
fmt.Println(month) // map[1:January]
```

- スライス同様、rangeを用いるとfor文でkey-valueをそれぞれ受け取りながら処理をすすめることができる。
- ただし**マップの順番を保証されない**
```go
for key, value := range month {
    fmt.Printf("%d %s\n", key, value)
}
```

# ポインタ
```go
func callByValue(i int) {
    i = 20 // 値を上書きする
}

func callByRef(i *int) {
    *i = 20 // 参照先を上書きする
}

func main() {
    var i int = 10
    callByValue(i) // 値を渡す
    fmt.Println(i) // 10
    callByRef(&i) // アドレスを渡す
    fmt.Println(i) // 20
}
```

# defer
ファイル操作などを行う場合、使用後のファイルは必ず閉じる必要がある。次の例では関数の最後にファイルクローズ処理を記述しているが、その前に関数を抜ける処理があったり、後述するパニックが起こってしまうとClose()まで到達しない場合が発生してしまう。
```go
func main() {
    file, err := os.Open("./error.go")
    if err != nil {
        // エラー処理
    }
    // 正常処理
    file.Close()
}
```

こうした処理はdeferを用いて記述できる。先の例ではfile.Close()の関数呼び出しをdeferの後ろに記述すると、この処理が**main()を抜ける直前に必ず実行されるようになる**
```go
func main() {
    file, err := os.Open("./error.go")
    if err != nil {
        // エラー処理
    }
    // 関数を抜ける前に必ず実行される
    defer file.Close()
    // 正常処理
}
```

# パニック
エラーは戻り値によって表現するのが基本だが，そうではない場合もありる。たとえば配列やスライスの範囲外にアクセスした場合や，ゼロ除算をしてしまった場合など。こうした処理はエラーを返すことができないため，代わりにパニックという方法でエラーが発生する。

このパニックで発生したエラーはrecover()という組込み関数で取得し，そこでエラー処理を実施できる。recover()をdeferの中に書くことで，パニックで発生したエラーの処理を実施してから，関数を抜けることができる。
```go
func main() {
    defer func() {
        err := recover()
        if err != nil {
            // runtime error: index out of range
            log.Fatal(err)
        }
    }()

    a := []int{1, 2, 3}
    fmt.Println(a[10]) // パニックが発生
}
```

## panic()
- パニックは組み込み関数panic()を用いて自分で発生させることもできる。
```go
a := []int{1, 2, 3}
for i := 0; i < 10; i++ {
    if i >= len(a) {
        panic(errors.New("index out of range"))
    }
    fmt.Println(a[i])
}
```

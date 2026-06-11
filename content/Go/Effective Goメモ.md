---
title: Effective Goメモ
date: 2026-01-10
modified: 2026-01-10
tags:
  - go
  - tips
---
https://go.dev/doc/effective_go

<hr>

## Formatting
- 行の長さはないが、行が長すぎると感じた場合は改行してタブを追加しインデントする
## Names
- 利用者は常にパッケージ名で中身を呼び出すため、パッケージ内で公開（エクスポート）される要素の名前は、パッケージ名との重複を避けるように設計する
- `ring.Ring`型を作成する関数（コンストラクタ）を例に挙げると、通常は`NewRing`としたくなるが、そのパッケージが`ring`という名前であれば、単に`New`と命名するのが慣習。利用者はこれを`ring.New`として参照するので十分に意味が通じる
## Getters
- Goの慣習ではGetterに`Get`というプレフィックスはつけない。Setterには`Set`プレフィックスをつける
```go
// ownerを取得する際は GetOwner() ではなく Owner() と呼ぶ
owner := obj.Owner()

// 条件に応じてSetterを呼ぶ
if owner != user {
    obj.SetOwner(user)
}
```
## MixedCaps
- キャメルケースを使用するのが慣習
## Semicolons
- 正式な文法では、文を修了するためにセミコロンを使用するがソースコード上ではほとんど現れない。コンパイル時にレクサー（字句解析機）がセミコロンを自動挿入しているため
## Control structures
**Type switch**
- インターフェース変数の動的な型を判別するために使用される。`switch t := t.(type) { ... }` という構文を使い、各ケース内で変数はその型として扱われる

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```
## Functions
**Named result parameters**
- 戻り値には、入力パラメータと同様に名前を付けることができる
  - ドキュメントとしての役割
    - 戻り値に名前を付けることで、コードが短くなるだけでなく、どの戻り値が何を意味するかが明確になり、ドキュメントとしての価値が高まる
  - 初期化と「裸の戻り値」
      - 名前付き戻り値は関数開始時にその型のゼロ値で初期化される。また、関数内で値を更新した後、引数なしの `return`ステートメントを実行すると、その時点の変数の値が返される
```go
func ReadFull(r Reader, buf []byte) (n int, err error) { // 戻り値が初期化される
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```
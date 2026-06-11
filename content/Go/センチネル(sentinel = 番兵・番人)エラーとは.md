---
title: センチネル(sentinel = 番兵・番人)エラーとは
date: 2025-09-26
modified: 2025-10-21
tags:
  - go
  - errorhandling
---
- 「現在の状態には問題があり、処理を続行できない」ことを知らせる事前定義されたエラー値
```go
// io packageで定義されている
var EOF = errors.New("EOF")

// io packageを使用
import "io"

// Is関数でerrを第二引数の値と同一か判定ができる
if errors.Is(err, io.EOF) {
    // 読み込みデータがなくなった場合の処理
}
```
- 参考：[entropy/センチネル値](https://scrapbox.io/entropy/センチネル値)

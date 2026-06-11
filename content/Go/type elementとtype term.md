---
title: type elementとtype term
date: 2025-09-26
modified: 2025-10-20
tags:
  - go
  - interface
  - type
---
## Declare a type constraint
- 型制約をインタフェースとして宣言できる
  - これを`type element`と呼ぶ
```go
type Number interface {
    int64 | float64 // ←がtype term（type elementの構成要素）
}
```
  参考：https://go.dev/doc/tutorial/generics#declare_type_constraint

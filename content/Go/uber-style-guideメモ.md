---
title: uber-style-guide-jaメモ
date: 2026-01-10
modified: 2026-01-11
tags:
  - go
  - tips
---
迷ったら一旦、[[Effective Goメモ]]か[uber-style-guide](https://github.com/knsh14/uber-style-guide-ja?tab=readme-ov-file)を見よう！

<hr>

## Channel Size is One or None
- `channel`のサイズは普段は1もしくはバッファなしのものにするべき
  - デフォルトでは`channel`はバッファなしでサイズが0になっていて、 それより大きいサイズにする場合はよく考える必要がある
  - どのようにしてサイズを決定するのか、チャネルがいっぱいになり処理がブロックされたときにどのような挙動をするかよく考える必要がある
## Use "time" to handle time
- 時間を正く扱うのは非常に難しい！！ので、時間を扱う場合は常に`time`パッケージを使用する
**Use time.Time for instants of time**
- 時刻を扱う（比較や足し引き）時は`time.Time`型のメソッドを使用する
```go
func isActive(now, start, stop time.Time) bool {
  return (start.Before(now) || start.Equal(now)) && now.Before(stop)
}
```
**Use time.Duration for periods of time**
- 期間を扱う時には`time.Duration`型を使用する
```go
func poll(delay time.Duration) {
  for {
    // ...
    time.Sleep(delay)
  }
}

poll(10*time.Second)
```
- カレンダー上で次の日の同じ時刻にしたい場合は`time.AddDate`メソッドを使用する。もしその時刻から正確にxx時間後にしたい場合は`time.Add`メソッドを使う
```go
newDay := t.AddDate(0 /* years */, 0, /* months */, 1 /* days */)
maybeNewDay := t.Add(24 * time.Hour)
```
## Style
**Avoid overly long lines**
- 横にスクロールしたり、たくさん首をふるような長過ぎるコードは避ける
- 横幅は99文字を推奨している。書く側はこれを超えると改行したほうが良いが、絶対ではない
## Unnecessary Else
- `if-else` のどちらでも変数に代入する場合、条件に一致した場合に上書きするようにする
```go
a := 10
if b {
  a = 100
}
```
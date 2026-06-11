---
title: Go1.25新機能 testing/synctest で高速＆確実な並列テストを実現する方法
date: 2025-09-26
modified: 2025-09-28
tags:
  - conference
  - go
---
- CANARYの人
- 課題
  - 非同期的なテストは難しい
  - テスト時間を短くしつつ、テストを落ちないようにする両立が難しいため
    - 非同期な処理が終わることを担保したのちに、チェックを実行する必要があること
    - 待ち時間は出来るだけ短くしたい
- `testing/synctest`
  - `Wait`
    - 非同期処理を担保する
  - `Test`
    - 待ち時間を短くする
  - bubble とdurably bolockedが重要
    - testing/synctest#Timeに記述されている
      - fake clockを使用している
      - 初期時刻はUTC 2000-01-01で設定されている
      - 全てのTargetがdurably blockになったら、bubble内の時刻が進む
    - testing/synctest#Blocking
      - time.Sleepはゴルーチンをduraby blockされる
  - synctest.TestでSleepしているテストをラップするだけで待ち時間を短くできる
  - ゴルーチンが複数ある場合は、synctest.Waitを使用する
    - 自分を呼んでいる ゴルーチン以外を観察してそれがdurably blockされるまでblockされる
  - タイムアウト60.221から0256
  - すぐに対応できる
    - 実務では現在のテストは残してsynctestを追加するのが良さそう
  - GoDoc読むの大切

<hr>

## 資料

- https://gocon.jp/2025/talks/958977/
- https://zenn.dev/canary_techblog/articles/ec8a96b4541685
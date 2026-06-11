---
title: サプライチェーン攻撃に学ぶmoduleの仕組みとセキュリティ対策
date: 2025-09-26
modified: 2025-09-30
tags:
  - go
  - conference
---
- kuroさんの発表
- 下記は自分が簡単にまとめたもの。詳しくは資料のスライドを確認

<hr>

## 発覚した事件
- 2025年2月、Socket Securityが報告
- 下記の悪意のあるパッケージが3年以上潜伏していた
  - `github.com/boltdb-go/bolt`
  - リモートコード実行のバックドアが仕込まれていた
    - 感染したマシンを外部から操作きるようにする仕組み
- タイポスクワッティングを悪用していた
## 攻撃手法
- 正式名称`github.com/boltdb/bolt`に酷似した上記、悪意のあるパッケージを作成
  - 微妙な命名の違いで、誤植や手違いを起こし悪意のあるパッケージを選択させるように設計されている
- Go Module Proxyにバックドアが仕込まれたバージョンをキャッシュさせる
  - 悪意のあるコードがプロキシサーバにキャッシュされる
- Github上でタグをクリーンな別バージョンに書き換える
  - Github上では痕跡を隠蔽されているので、安全なコードに見える
## なぜ防げなかったのか
- Goのセキュリティが機能する前の攻撃であったため
## 対策
- 依存パッケージを手書きしない
  - goplsを使用する
    - ただし、補完自体が間違えている可能性もある
- lintで検知
  - importが見やすくなるのでレビューでタイポを確認しやすくなる
- 下記でpackage調べてヘッダーでコピーも対策になりそう
  - https://pkg.go.dev/

<hr>

## 資料
- https://gocon.jp/2025/talks/939638/
- https://speakerdeck.com/kuro_kurorrr/understanding-module-through-the-lens-of-supply-chain-attacks
- https://socket.dev/blog/malicious-package-exploits-go-module-proxy-caching-for-persistence

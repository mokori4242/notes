---
title: 今日から始めるpprof
date: 2025-09-26
modified: 2025-10-01
tags:
  - go
  - conference
---
- ymotongpooさんのワークショップ
- 下記は自分が簡単にまとめたもの。詳しくは資料のスライドを確認

<hr>

## profileとは
- プログラムのどこが遅いかを調査するための手法
- 統計的な数値を取得する
## pprofの基本機能
- プログラムからのプロファイルデータの取得
- プロファイルデータの可視化
  - Top
  - Graph
  - Flame Graph
  - Peek
  - Source
  - Disassemble
- `go tool pprof -http :9999 cpu.sample.prof`でWebUIが9999ポートで立ち上がる
  - `-http`オプションなしでインタラクティブモードで立ち上げることもできる
    - プロファイルデータを確認することできるが見づらめ
  - WebUIを立ち上げるには`graphviz`のインストールが必須
## pprofをプログラムに組み込む（計装）
- `runtime/pprof`で明示的にプロファイルの取得を宣言することで特定の処理のプロファイルを取得
```go
// create pprof report to record
report, _ := os.Create("cpu.sample.prof")
defer report.Close()
_ = pprof.StartCPUProfile(report)
defer pprof.StopCPUProfile()
```
- `net/http/pprof`はimportのみでOK
## pprofの読み方を理解する
- Flame Graph
  - 長い = リソース消費が大きい = 悪い
- Top
  - リソース消費順に表示
  - Flatが遅い場合にはアルゴリズムに問題がある
  - Cumが遅い場合には外部関数の呼び出し方に問題がある
- Graph
  - 箱が大きいとリソース消費が大きい
  - 色はプロファイルを比較した時の指標
    - 緑（リソース消費減）〜灰色（変化なし）〜赤（リソース消費増）のグラーデション
    - 一つのプロファイルを見るときは箱の大きさが重要
- Source
  - 行レベルでのリソース消費を表示
## 改善にあたっての結果の見方
- 基本、自分が書いたものに原因があると思え
  - 標準パッケージや著名なパッケージはベンチマークテストが入念に行われているので、パフォーマンスは良いものと考える
- Flame Graph、Graphでボトルネックの関数の見当をつけて、Sourceで深掘りする
- 改善結果は計測によってのみ判断される
  - XXXミリ秒処理が早くなった
  - X%ヒープサイズが小さくなった
- 改善結果を客観的に共有するには図表が効果的
  - pprofで差分を表示するには、`go tool pprof -http :9999 -base cpu.sample.prof diff.sample.prof`
![[gopproflow.png]]
![[gopproffigure.png]]
## テストの一環としてのプロファイル

- `_test.go`ファイルにベンチマーク関数を記載して、`go test -bench=. -cpuprofile=cpu.sample.prof`のオプションでプロファイルも出力できる
  - ヒープを取得する場合は-`memprofile=mem.sample.prof`

## 継続的プロファイル

- CNCFのオブザービリティ白書で重要なテレメトリとして京座奥的プロファイルを挙げている
  - https://github.com/cncf/tag-observability/blob/main/whitepaper.md#observability-signals
- 本番環境で動作しているものの継続的プロファイルを取る
  - スパイクした段階でプロファイルを取っても、スパイク段階のプロファイルは取れないため
- 継続的プロファイルの仕組みを自分で作ることもできるが、サービスとして提供しているSaaSも多くなってきている
![[observability.png]]
## ワークショップでの有り難い小話
- SAMPLESはほぼ使わない
  - データがおかしい時にSAMPLESを確認する（数が少ない場合が多い）
- パフォーマンス改善では複数のコンテキストをもとに解決することが多い
  - 実務ではレビュー時にGraphを貼ったりしてコンテキスト集める場合がある
- sysCall = カーネルとユーザーランドのやり取りで重い処理になる
- 特定のユースケースをAIエージェントに理解させるのにプロファイルデータが適している

<hr>

## 資料
- https://gocon.jp/2025/workshops/1018925/
- https://t.co/9mnMktVykv
- https://pkg.go.dev/bufio
- https://ja.wikipedia.org/wiki/Cloud_Native_Computing_Foundation
- https://www.cncf.io/blog/2022/05/31/what-is-continuous-profiling/

---
title: Goで体感するMultipath TCP ― Go 1.24 時代の MPTCP Listener を理解する
date: 2025-09-26
modified: 2025-09-28
tags:
  - go
  - conference
---
- さくらインターネットに所属の人
- MPTCPとは
  - 複数のTCPコネクションを束ねて通信の帯域を拡張し、冗長性を高めることができる
  - シームレスなハンドオーバー
    - モバイル体験が改善される
  - 帯域の集約
  - アプリから1つのTCP socketに見える

<hr>

## 資料
- https://gocon.jp/2025/talks/958952/
- https://speakerdeck.com/takehaya/go-conference-2025-godeti-gan-surumultipath-tcp-go-1-dot-24-shi-dai-no-mptcp-listener-woli-jie-suru

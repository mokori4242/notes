---
title: 全てGoで作るP2P対戦ゲーム入門
date: 2025-09-26
modified: 2025-09-27
tags:
  - go
  - conference
---
- Ebitengineを使って画面描画
  - Goの2Dゲームライブラリ
  - https://github.com/hajimehoshi/ebiten
  - 更新と描画を繰り返せばゲームになる
- マッチング
  - websocketserverをgoで実装
    - 双方向通信ができる
  - Queue管理
  - ビルドインパッケージで実現
- P2P通信
  - ブラウザ対ブラウザでやり取り
  - サーバコスト0で遅延が少ない
  - WebRTCを使用
    - https://en.wikipedia.org/wiki/WebRTC
    - goではpion/webrtcが有名
    - https://github.com/pion/webrtc
  - P2P通信にはsignalingが必要
    - https://github.com/OpenAyame/ayameを使用
- Ebitengineとゲームロジックの橋渡しにchannelを使う

<hr>

## 資料
- https://gocon.jp/2025/talks/956553/
- https://speakerdeck.com/ponyo877/quan-tegodezuo-rup2pdui-zhan-gemuru-men
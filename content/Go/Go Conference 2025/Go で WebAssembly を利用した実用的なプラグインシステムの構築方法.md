---
title: Go で WebAssembly を利用した実用的なプラグインシステムの構築方法
date: 2025-09-26
modified: 2025-09-27
tags:
  - go
  - conference
---
- WASMでプラグイン作る
  - Sandbox機構により安全に実行できる
  - WASMランタイムがあればどこでも動く
  - Guetで実行する処理をHostで制限することができる
  - GuestがクラッシュしてもHostが巻き込まれない
- Shared Library（公式）
  - GusetがクラッシュするとHostもクラッシュする
  - Guestで動くコードに制限がない
  - GoとGo Modulesを揃えないと動かない
- WASM以外（RPC）
  - hasicorp/go-plugin

　WASI

　	Host側でアクセスを細かく制御できる

　	0.2までリリース済

　	Goは0.1まで実装済

　	アプリケーションにより、必要なものが異なるのでコンパイラかインタプリタか選ぶ

　Tiny GoとGo、どちらでWASMプラグインを作るか精査する必要がある

　wazero

　	wasi-go

　	dispatchrun/net

　		上記二つを併せてNetwork Socket対応の実装を利用できる

<hr>

## 資料

- https://gocon.jp/2025/talks/958532/
- https://docs.google.com/presentation/d/e/2PACX-1vTaC6MpbfGMO72_gLj4YYagXhBNs5cux7nJAS4LJZBG5i9zPJnBQSlwGlwmNasov8vCVpqEIm-U6Ra1/pub?start=false&loop=false&delayms=3000
- [entropy/WebAssembly](https://scrapbox.io/entropy/WebAssembly)

---
title: バージョン管理ツールについて（anyenv・asdf・mise）
date: 2025-10-24
modified: 2025-12-16
tags:
  - tips
  - versioncontrol
---
## anyenv
- 複数の`**env`形式の環境マネージャー（rbenv、pyenv、nodenvなど）を一元管理するためのラッパーツール
- あくまでラッパーツールのため各`**env`インストール後は、各ツール毎にコマンド使用・プロジェクト毎のバージョン管理を行わなければならない
- 後発のasdfやmiseが登場した影響か、2025年10月時点での最終リリースが2022年8月

<hr>

https://github.com/anyenv/anyenv

## asdf
- 単一のCLIコマンド`asdf`で複数の言語ランタイムバージョンを管理できるバージョン管理ツール
- `.tool-versions`ファイルでプロジェクト毎のバージョンを管理できる
- 多数のプラグインを導入することができる
 
<hr>

-  https://asdf-vm.com/ja-jp/guide/introduction.html
- https://github.com/asdf-vm/asdf
## mise
- asdfからの移行を容易にするために、`.tool-versions`ファイルの互換性を維持している
- asdf同様、単一のCLIコマンド`mise`で複数の言語ランタイムバージョンを管理できるバージョン管理ツール
- `mise.toml`ファイルでプロジェクト毎のバージョン・タスクランナー・環境変数を管理できる
  - タスクランナー
    - `mise.toml`ファイルで下記のような記述し`mise run タスク名`で実行可能
```toml
[tasks.build]
description = "Build the CLI"
run = "ls"

[tasks.run]
description = "Run the CLI"
run = "ls -a"
```
    ・ 既存コマンドと重複しない場合、runを省略することができる
    ・mise runでタスク一覧表示も可能
    
![[miserun.png]]

<hr>

- https://mise.jdx.dev/
- https://github.com/jdx/mise
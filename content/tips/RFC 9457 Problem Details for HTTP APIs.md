---
title: RFC 9457 Problem Details for HTTP APIs
date: 2025-10-29
modified: 2025-10-29
tags:
  - http
  - rfc
  - tips
---
https://www.rfc-editor.org/rfc/rfc9457.html

<hr>

## 問題

- HTTP APIのエラーレスポンスはこれまでバラバラで、プロジェクトによって設計を考えなければならなかった
  - 例えば、400エラーの場合
```json
// Aプロジェクト
{"error": "Invalid input"}

// Bプロジェクト 
{"code": "INVALID_INPUT", "message": "入力が不正です"}
```
## RFC 9457での改善
- RFC 9457ではJSONやXMLで統一的に表す**Problem Details**形式を定義する
  - content-typeを`application/problem+json`や`application/problem+xml`で宣言する
    - 以下、標準項目でエラーレスポンスを定義する。なお、全てのフィールドはオプショナルである
      - `type`
        - 問題の種類を示すURI
        - 相対パスだとAPIのベースURIによって意味が変わってしまうため、一意性・互換性を保つために絶対URIを推奨
        - 必ずしもアクセス可能である必要はない
        - 空の場合はデフォルト値の`about:blank`になる
          - 古いAPIや単純なケースではtypeが必要でないケースがある
      - `status`
        - HTTPステータスコードとステータスコードを必ず一致させる
      - `title`
        - 人間が読める問題の簡潔な説明
      - `detail`
        - 人間が読める発生した問題の詳細
      - `instance`
        - 問題の発生箇所を特定するURI
        - ログやサポートで特定するために使用する。一意に識別できればよく、アクセス可能である必要はない
        - IDなど含める場合は外部からは解読できないUUID等で対処する
        - 必ずしもアクセス可能である必要はない
      - その拡張項目
```json
HTTP/1.1 403 Forbidden
Content-Type: application/problem+json
Content-Language: en

{
 "type": "https://example.com/probs/out-of-credit", // クレジットのエラーとわかるURI
 "status": 403,
 "title": "You do not have enough credit.",
 "detail": "Your current balance is 30, but that costs 50.",
 "instance": "/account/12345/msgs/abc", // 誰の取引でエラーが起こったかわかるURI
 "balance": 30, // 拡張項目
 "accounts": ["/account/12345", // 拡張項目
              "/account/67890"]
}
```

```json
HTTP/1.1 422 Unprocessable Content
Content-Type: application/problem+json
Content-Language: en

{
 "type": "https://example.net/validation-error",
 "title": "Your request is not valid.",
 "errors": [ // 拡張項目
             {
               "detail": "must be a positive integer",
               "pointer": "#/age"
             },
             {
               "detail": "must be 'green', 'red' or 'blue'",
               "pointer": "#/profile/color"
             }
          ]
}
```

## これから
- 現状、statusがProposed Standardなので実際に採用しても良い段階である！
  - Internet Standardではないので国際標準ではない

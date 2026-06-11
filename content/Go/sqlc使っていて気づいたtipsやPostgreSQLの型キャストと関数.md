---
title: sqlc使っていて気づいたtipsやPostgreSQLの型キャストと関数
date: 2025-11-19
modified: 2025-11-19
tags:
  - go
  - tips
  - db
---
## [sqlc](https://github.com/sqlc-dev/sqlc)とは
> sqlc はSQLから型安全なコードを生成します。その仕組みは以下のとおりです。
> クエリは SQL で記述します。
> sqlc を実行して、これらのクエリに対する型安全なインターフェースを持つコードを生成します。
> 生成されたコードを呼び出すアプリケーション コードを記述します。
### メリット
- 他ORMを使用すると、生成されるSQLを確認して実行計画の確認をすることが多い。また、生成されるSQLが不明瞭で知見を溜めることも難しい。その為、最初からSQLで実行できるコードを生成できたら良いじゃん！
- 複雑な条件になってくると生のSQLに頼ることも出てくる。その為、最初からSQLで実行できるコードを生成できたら良いじゃん！
- SQLの変更をコンパイル時に検知できる
### デメリット
- ある程度SQLに知見がないと辛い
## 型キャスト構文
- 下記、どちらも等価な構文
```sql
 CAST ( expression AS type )
 expression::type
```
  - `CAST`構文はSQLの仕様に準拠したもの（らしい）
  - `::`を使用する構文はpostgresで伝統的に使用されている方法
ドキュメント：https://www.postgresql.jp/docs/9.4/sql-expressions.html
## 関数
  - 1対多の関係で一つのカラムに集約したいケース、あるある
    - 例えば、店舗の休日が複数存在する場合
      - 下記SQLの流れ
```sql
COALESCE(ARRAY_AGG(p.method), '{}')::int[] AS payment_methods,
```
        1. `ARRAY_AGG`関数で引数のカラムを配列に集約
	        ・ 入力行が存在しない場合は空配列ではなく`NULL`を返す
		2. `ARRAY_AGG`の戻り値が`NULL`の場合に`COALESCE`関数で空配列を返す
		3. `int`配列で型キャストする。面倒に感じるが、sqlcで集約関数を使用する場合、明示的に型キャストしてあげないと`interface{}`でコードが生成されてしまう
```sql
 -- PaymentMethods interface{}で生成される
	COALESCE(ARRAY_AGG(p.method), '{}') AS payment_methods,
 
 -- PaymentMethods []int32で生成される
	COALESCE(ARRAY_AGG(p.method), '{}')::int[] AS payment_methods,
```
      ・集約対象のカラムが`NULL`許容の場合や`LEFT JOIN`の`ON`句で`NULL`が発生する可能性がある場合は、`FILTER`関数を使うとフィルター後の結果を集約できる
```sql
COALESCE(ARRAY_AGG(p.method) FILTER (WHERE p.method IS NOT NULL), '{}')::text[] AS payment_methods,
```
        ・参考：https://www.postgresql.jp/docs/17/tutorial-agg.html
      ・集約対象のカラムが重複する場合は`DISTINCT`キーワードを使うと重複を防ぐことができる
```sql
COALESCE(ARRAY_AGG(DISTINCT p.method), '{}')::text[] AS payment_methods,
```
        ・参考：https://stackoverflow.com/questions/26363742/how-to-remove-duplicates-which-are-generated-with-array-agg-postgres-function
      ・`ARRAY_AGG`関数以外にも下記、集約関数あるので用途に応じて使い分けると良いかも（ドキュメントを参照してください）
        ・`JSON_AGG`関数
        ・`JSON_OBJECT_AGG`関数
        ・`STRING_AGG`関数
        ・`XMLAGG`関数
  - ドキュメント：https://www.postgresql.jp/docs/9.4/functions-aggregate.html
## tips
　sqlcで動的に絞り込みを実現する

```sql
	 WHERE
       (sqlc.narg(store_name)::text IS NULL OR s.name ILIKE '%' || sqlc.narg(store_name)::text || '%')

```
  - 絞り込み引数が`NULL`であれば絞り込みは行われない
  - 参考：https://www.reddit.com/r/golang/comments/1c9vaxe/sqlc_is_goated/

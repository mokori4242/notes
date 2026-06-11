---
title: panicと(PHPの)exception
date: 2025-11-20
modified: 2025-12-28
tags:
  - go
  - php
  - tips
  - errorhandling
---
## panic
- 組み込みの関数
- `panic`が起こるとそれ以降の処理は行わず、スタックを巻き戻し、アプリケーションをクラッシュさせる
  - この際、`defer`があればLIFOで実行していく
    - `defer`内に`recover`がある場合は`panic`を捕捉し、アプリケーションを回復させる
      - `recover`した時点でパニック状態は解除されるが、その関数の残りの`defer`（スタックに残っているもの）は通常通り実行されてから、呼び出し元に戻る
```go
func main() {
	recovery()
	fmt.Println("Hello, 世界")
}

// panicを起こして回復させるまでの関数
func recovery() {
	defer fmt.Println("1番")
	defer func() {
		// recover()が組み込み関数
		if r := recover(); r != nil {
			fmt.Println("2番 - panic捕捉:", r)
		}
	}()
	defer fmt.Println("3番")
	panic("panic!!")
	// 下は処理されない
	fmt.Println("4番")
}

// output
// 3番
// 2番 - panic捕捉: panic!!
// 1番
// Hello, 世界
```
## exception
- 基底例外クラスの`Exception`がある
  - 基底例外クラスを継承したカスタム例外クラスが、特定の処理の際に投げられる（ことが多い）
- 例外が投げられると例外クラスのオブジェクトが生成され、スタックを巻き戻す
  - `catch`可能な場所がない場合はアプリケーションがクラッシュする
    - `catch`
```php
 // case1
 <?php
 
 function main() {
     recovery();
 }
 
 function recovery() {
     try {
         throw new Exception('例外!!');
     } catch (Exception $e) {
         echo '3番' . "\n";
         echo "2番 - 例外出力: {$e->getMessage()}" . "\n";
         echo '1番' . "\n";
     }
     
     echo 'Hello, 世界';
 }
 
 main();
 
 // output: 3番 2番 - 例外出力: 例外!! 1番 Hello, 世界
```

```php
 // case2
 <?php
 
 class CustomException extends Exception
 {
 }
 
 function main() {
     recovery();
 }
 
 function recovery() {
     try {
         throw new CustomException('カスタム例外！！');
         
         echo 'ここは実行されない' . "\n";
     } catch (CustomException $e) {
         echo '3番' . "\n";
         echo "2番 - 例外出力: {$e->getMessage()}" . "\n";
         echo '1番' . "\n";
     } catch (Exception $e) {
         echo '予期せぬ汎用例外をキャッチ' . "\n";
     }
     
     echo 'Hello, 世界'; 
 }
 
 main();
 
 // output: 3番 2番 - 例外出力: カスタム例外！！ 1番 Hello, 世界
```
- Handler
```php
<?php

class CustomException extends Exception
{
}

class Handler
{
    public static function catchUncaught($e): void
    {
        if ($e instanceof CustomException) {
            echo '3番' . "\n";
            echo '2番' . "\n";
            echo '1番' . "\n";
        } else {
            echo '予期せぬ汎用例外をキャッチ' . "\n";
        }
    }
}

function main() {
    set_exception_handler(['Handler', 'catchUncaught']);
    
    recovery();
    
    echo 'この行は実行されない'; 
}

function recovery() {
    throw new CustomException('ハンドラでキャッチ！！');
}

main();

// output: 3番 2番 - 例外出力: ハンドラでキャッチ！！ 1番
```
## 共通点
- スタックを巻き戻す
- 何もしないとアプリケーションがクラッシュする
- 関数シグネチャに現れない
## 相違点
- `panic`は
  - `any`（どんな型でも）を渡せる
  - スタックを巻き戻しつつ、機械的にLIFOで`defer`を実行する。型の比較はしない
    - スタックトレースでエラー発生箇所はわかるが、具体的に何が起こったかわかりづらい場合が多い
  - アプリケーションの回復(`recover`)は`defer`内でのみ可能
- exceptionは
  - 投げれるのは`Throwable`インタフェースを実装したクラス
    - 基底例外`Exception`クラスが準備されているので、開発者は`Exception`を継承したカスタム例外クラスを使用することが多い
      - `Error`クラスも`Throwable`インタフェースを実装しているが、PHPの内部エラーで使用するので開発者の使用は避けたい
  - スタックを巻き戻しつつ、型の比較を行い`catch`可能な場所を探索する
    - `catch`可能な場所が見つかると、そのブロックの処理が実行される
    - 型により、どのようなエラーが発生したかわかりやすい
  - アプリケーションの回復は`catch`ブロックで可能
    - `catch`ブロックの配置場所に特に制限はない
    - `catch`ブロックの実行後、ブロックの直後から処理を再開する
  - 例外Handlerをセットする仕組みが言語レベルで準備されている
    - `catch`で捕捉しきれなかった（しなかった）例外を一元的に捕捉する仕組み
    - 予期せぬエラー時のログ出力など行い、アプリケーションをクラッシュすることができる
## 結論
- goでは基本的に`error`を使用する
  - `error`は関数シグネチャに現れるのでエラーハンドリングを忘れ難い
  - `error`は文脈を付与しやすい
  - `error`はラップされたエラーに対する処理がシンプルに実装できるようになっている
    - `errors.Is`や`errors.As`
  - `panic`を使用すると、パスが増えてコードのシンプルさが無くなってしまう

<hr>

## 参考
- https://www.oreilly.co.jp/books/9784814401192/
- https://go.dev/wiki/CodeReviewComments#dont-panic
- https://go.dev/blog/defer-panic-and-recover
- https://www.php.net/manual/ja/class.error.php
- https://www.php.net/manual/ja/class.exception.php
- https://go-tour-jp.appspot.com/methods/19

# バッチフレームワーク


## バッチ処理としてほしい機能？  
JP1、cronなどから`.sh`がキックされてGoが呼ばれる流れとしたときにバッチフレームワークとしてほしい機能  
* コマンドラインの引数のパース  
引数のチェック機構などはなくてもよい？  

* 終了時の終了コード設定


## 標準パッケージでの実装

コマンドライン引数は標準パッケージの`flag`で取得可能。  
終了コードは`os.Exit()`で実装可能。

```go
package main

import (
	"flag"
	"fmt"
	"os"
)

func main() {
	// 引数の取得
	flag.Parse()
	args := flag.Args()

	for _, s := range args {
		fmt.Println(s)
	}

	// 終了コードの設定
	os.Exit(1)
}
```

実行  
`go run main.go 0123 4567`と指定した場合
```
> go run main.go 0123 4567
0123
4567
exit status 1
```



## 外部パッケージ調査
外部パッケージをいくつか見たが、できることはサブコマンドを定義したり、必須のパラメータを定義できたりといったレベルなので、上記で記載したJP1→.sh→.goの流れであれば標準パッケージで十分だと感じる。  

* cliパッケージ
  * [urfave/cli](https://github.com/urfave/cli)


## 標準パッケージを使用したサンプル
[sample-batch](https://github.com/temp-go-dev/sample-batch)


## 参考
[golang でシェルの Exit code を扱う](https://tellme.tokyo/post/2018/04/02/golang-shell-exit-code/)


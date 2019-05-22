# パッケージ管理（利用）

## 目的
外部パッケージの利用方法を調査・検討していく。  
具体的にはJavaでいうところの、`Maven`、`Gradle` の依存関係の解決あたりの機能について
GoLang界隈には何があるのか、何を使うべきなのかを検討する。

## 調査事項

調査時のGoLangのバージョン：1.12.5

現状、GoLangでのパッケージ管理（依存関係管理）は以下が存在する。

|#|ソフト|最盛期|導入時期|
|:---:|:---|:---|:---|
|1|go get||
|2|[glide](https://github.com/Masterminds/glide)|||
|3|[dep](https://github.com/golang/dep)|||
|4|[vgo](https://github.com/golang/vgo)|||
|5|[Modules](https://github.com/golang/mod)||1.11（試験的）1.13（正式予定）|

それぞれを掘り下げても仕方がないので、  
結論としては「`Modules`」を利用するのが今後のデファクトである模様。

### Modules

- 参考  
https://budougumi0617.github.io/2019/02/15/go-modules-on-go112/

#### Modulesの２つのモード
`Modules`には２つのモードが混在している。
- GOPATH mode
  - バージョン1.10までと同じ挙動
  - 標準パッケージ以外を全部GOPATH以下のディレクトリで管理する
  - パッケージ管理は最新リビジョンのみが対象となる :thumbsdown:
- module-aware mode
  - 標準パッケージ以外を全部モジュールとして管理する :thumbsup:
  - モジュールの管理やビルドが任意のディレクトリで可能になる :thumbsup:
  - モジュールはリポジトリのバージョンタグ/リビジョン毎に管理する :thumbsup:

#### 切り替え方法
`GO111MODULE` という環境変数で切り替える

|設定値|挙動|
|:---|:---|
|on|module-aware|
|off|GOPATH|
|auto|GOPATH配下では `GOPATH mode`、それ以外では `module-aware mode`|

基本 `on` で開発を行った方がよい。

#### 使い方

使い方をいくつかメモ。  
以下はすべて `module-aware mode`

##### プロジェクト新規作成
```bash
$ cd 任意の場所
$ mkdir project
$ cd project
$ go mod init github.com/temp-go-dev/sample-api-crud
$ cat go.mod
module github.com/temp-go-dev/sample-api-crud

go 1.12
```
これで、`go.mod` というファイルが作成される。

##### 利用ライブラリの追加（依存関係への追加）
```bash
$ go get -u github.com/labstack/echo
```
モジュールがダウンロードされる。  
ダウンロードされる場所は、`$GOPATH/pkg/mod`  
GOPATHはデフォルトだと、`$HOME/go` なので、Windowsでは`C:\Users\{user}\go`  
上記の場合だと、`$GOPATH\pkg\mod\github.com\labstack\echo@v3.3.10+incompatible` が作成される。  
自分は `.m2` ぽい印象を受けた。

__オプション__
|||
|:--|:---|
|-u|常にリポジトリの最新のコミットをダウンロードする。このオプションを指定しない場合はローカルに存在しないリポジトリのみダウンロードする|
|-t|パッケージのダウンロード後、ビルド前にユニットテストの実行を行う。テストに失敗した場合はビルドを行わずに終了|
|-d|ダウンロードのみを行い、ビルドを行わない|
|-v|詳細表示|

#### 開発する構成

`Modules` - `module-aware mode` を利用する場合のローカルのディレクトリ構成
Windows前提で記載する

```
C:\
 |-- Users\{user}\go …GOPATH
 |     |-- pkg\mod
 |     |       |-- 各種パッケージ
 |
 |-- {任意} 
 |     |-- {my-module}
 |     |       |-- go.mod
 |     |       |-- go.sum
 |     |       |-- main.go
 |     |       `-- etc...
```

`my-module` の中で別パッケージを作成して利用する場合の実装例

- main.go
```go
package main

import (
	"net/http"
	"github.com/labstack/echo"
	"github.com/temp-go-dev/sample-api-crud/service" // 自分のモジュールの別パッケージ
)

func main() {
	e := echo.New()
	e.GET("/", func(c echo.Context) error {
		return c.String(http.StatusOK, service.GetMessage())
	})
	e.Logger.Fatal(e.Start(":1323"))
}
```

- service/service.go
```go
package service

// GetMessage メッセージを返却
func GetMessage() string {
	return "Hellow, World !!!"
}
```

- go.mod
```go
module github.com/temp-go-dev/sample-api-crud

go 1.12

// omitted
```
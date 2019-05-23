# 利用するIDE

Date: 2019-05-23

## 目的
プロジェクトで利用するIDEを決定する。


## 調査結果

### 調査したIDE
* Atom  
* Visual Studio Code  
* GoLand(一部有料)
* Eclipse

### ほしい機能の観点
* コード補完・ジャンプ機能・自動整形
* ビルド
* デバッカー
* Gitとの連携

上記4つを調査した結果、やれることに差はなさそうなため、好みの問題と思われる。
比較的情報が多く、設定が簡単なため、VSCodeがよさそう。  


# VSCode

[VScode](https://code.visualstudio.com/)をインストール  
### 拡張機能の設定
* マーケットで「Go」で検索して出てきたもの(Go for Visual Studio Code)を追加  
* コマンドパレットで `Go: Install/Update tools`
出てきたものをすべて選択し、インストール/アップデート  
`
gocode
gopkgs
go-outline
go-symbols
guru
gorename
dlv
gocode-gomod
godef
goreturns
golint
gotests
gomodifytags
impl
fillstruct
goplay
godoctor
`  

### 所感
必要な機能は一通り整っており、簡単に導入できる。  
Gitとの連携もGUIで行えるため簡単。  
デバッグ、メソッドジャンプや下記の静的解析も行える。  
動作が軽い
* go imports  
使用していないimportを削除したり追加したりする
* go lint  
構文上微妙なものを警告表示してくれる


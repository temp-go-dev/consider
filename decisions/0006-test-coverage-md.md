# カバレッジ確認

## 方法は大体４種類見つかった

### 1. 直接go testのオプションを使って確認する  

コマンド例：go test -cover
 
 - メリット  
   - コマンド叩くだけでカバレッジを確認できる

- デメリット
  - どこが通ってないのがわからない

### 2. go testとgo toolでファイルを生成して確認  

コマンド例： go test -coverprofile=cover.out ./sample/  
　　　　　　 go tool cover -html=./sample/cover.out -o cover.html

- メリット
  - html生成して、どこが足りないのが一目瞭然

-デメリット
  - ファイルいちいち生成し、管理が面倒くさい
  - Linux環境だと生成したhtmlの画面崩れの可能性がある

### 3. vscodeの拡張機能GOを使う  

使い方：ソース画面右クリック→GO: Toggle Test Coverage In Current Package

- メリット
  - 操作が簡単で、カバレッジの確認がすぐできる

- デメリット
  - ソース全体が赤と緑に塗りつぶさる

### 4. vscodeの拡張機能のGo Coverage Viewer

使い方：ソース画面でctrl+shift+P同時押してGO Coverage: Display Resultsを入力して選択（２回目以降はそのまま選択できるようになる）

- メリット
  - 2と同じファイルで確認できるからわかりやすい

-  デメリット
    - 試したところで、開かない場合がある
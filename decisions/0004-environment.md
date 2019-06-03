# 環境変数（依存値）管理

## 環境変数（依存値）とは
プログラムのロジックは基本的に静的である。
環境によって動的に変化する値、これを環境変数（依存値）と呼んでいる。

## GoLangでの扱い方（標準）
`os`パッケージを利用する。  
利用方法は簡単であるが、これ以上でも以下でもない。

```go
sampleEnvValue := os.Getenv("SAMPLE_ENV_VALUE")
```

Javaでいうところの、`System.getEnv("Key")` 相当。
これは、多くの環境変数を一気にロードしたりするには向いていない。とはいえ、

## GoDotEnv
[joho/godotenv](https://github.com/joho/godotenv)  
`.env`という拡張子のファイルを用意してLoad処理を行うと、読み込んで環境変数にExportしてくれる。


## envconfig
[kelseyhightower/envconfig](https://github.com/kelseyhightower/envconfig)  
こちらは、構造体と環境変数をマッピングして読み込み保持してくれる。

## どう使うか
`GoDotEnv`と`envconfig`の両方を利用して、Javaの`Properties`およびSpringの`ConfigurationProperties`の組み合わせと等しい使い勝手の良さが出るのではないか？
まだガッツリ利用していないので、サンプル開発にて利用してみる。

少し気にする部分としては、`GoDotEnv`がDocker利用時に本当に必要になるのか？という部分。デプロイ時には、Docker起動の`--env-file`で`key=value`形式のファイルを指定すれば機能としては重複するはずなので考えもの

## 他の可能性
こんなものもある。

[build時にtags指定でビルド対象のソースファイルを切り替える](https://qiita.com/ueokande/items/fac0d1219dbbc8f44db7#build-tags%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E3%83%93%E3%83%AB%E3%83%89%E3%81%AE%E5%88%87%E3%82%8A%E5%88%86%E3%81%91)

## 結果

上記調査時に上がったGoDotEnvによる環境変数読み込みは  
envfileがイメージに内蔵されてしまうため、環境変更時などにimageの再生成が必要となってしまう。  
そのため下記方法で読み込む方法とする。  

* 環境変数の読み込み  
docker 起動時に`.env`ファイルを読み込む  
`.env`ファイルは別のリポジトリを作成し、submoduleとして取り込む

* コード上での環境変数利用方法  
envconfigを利用し、構造体とのマッピングを行う。  
利用方法は[サンプル参照](https://github.com/temp-go-dev/sample-api-gin/blob/master/config/config.go)

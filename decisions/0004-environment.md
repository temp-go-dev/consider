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

## 結果

サンプル開発と議論の結果を後で書く
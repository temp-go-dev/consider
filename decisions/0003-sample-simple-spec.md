# WEBサンプル実装のシンプル仕様

Date: 2019-05-24

## 概要

- REST-API
- 認証なし（今のところ）
- RDB接続あり
- REST-APIによるRDBレコードのCURD操作を行う
- RDBはMySQLとする

## RDB仕様

DB定義は適当に作りましたが、以下にあります。  
[sample-db](https://github.com/temp-go-dev/sample-db)

## API仕様
API仕様を文章で作成するのが面倒なので、OpenAPIのYAMLで記載する。

- [sample-simple-spec.yaml](files/sample-simple-spec.yaml)

とりあえず、便利そうだったので以下のVSCode Pluginを入れておいた。


|Plugin|何をしてくれるか（簡易）|
|:---|:---|
|Swagger Viewer|Swaggerv2/OAS3のYAMLをSwaggerUIとして見れるようにしてくれるもの。多分普通にローカルでSwaggerEditorを立ち上げちゃってる。内容としては https://editor.swagger.io/# と変わらない|
|openapi-lint|Editor上でSwaggerV2/OAS3の構文チェックをしてくれる|

話が変わってきているが、SwaggerV2の構文に慣れきっているのであれば、[コンバーター](https://mermade.org.uk/openapi-converter) を使うのも早い。
このコンバーターは、Jarでも提供されている。
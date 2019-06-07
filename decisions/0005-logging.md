
# logging



## ロギングパッケージ

* [zap](https://github.com/uber-go/zap)   
  実績があるzapを調査。  
  標準ライブラリより高速に動作するとのことでよく使われているらしい。


## ログローテーション

ロギングのパッケージではローテーションまで行えるものがないため、
ロギングとローテーションは別で考えたほうがよい。  
Linuxの`logrotate`を利用するか、ローテーション用のパッケージを利用するかの手法が存在する。

* [lumberjack](https://github.com/natefinch/lumberjack)  
設定がシンプル  
ただし、複数プロセスからの書き込みに対応していないそうなので、1サーバ１プロセス構成でなければ使用できない。  
zapとの連携方法 https://github.com/uber-go/zap/blob/master/FAQ.md#usage  


## lumberjack

バックアップファイル名の指定はできない  
オプションが必要最低限しかない

```go
	w := zapcore.AddSync(&lumberjack.Logger{
		Filename:   "/var/log/app.log",  // ファイル名
		MaxSize:    100,     // MB
		MaxBackups: 3,     
		MaxAge:     28,    // days
		Compress:   true,  // 圧縮
		LocalTime:  true,  // バックアップファイル名の時間
	  })

```

* バックアップされたファイルの形式  
  ```
  /var/log/app.log
  /var/log/app-2019-06-06T14-57-05.818.log.gz
  /var/log/app-2019-06-06T14-57-05.836.log.gz
  ```

## zapとlumberjackを使用した実装例

### 構造化メッセージ
下記のように構造化したメッセージの出力が可能。  
Errorの出力もzap.Errorで構造化してくれる。  

* 実装
  ```
  logger.Error("errors", zap.String("key", "value"), zap.Error(err))
  ```

* 出力ログ
  ```
  2019-06-06T20:27:54.131+0900	ERROR	errors	{"key": "value", "error": "Error 1062: Duplicate entry '232156e6-763a-4a68-a410-885bb94d1a95' for key 'PRIMARY'"}
  ```




### ログフォーマット

* ログ出力の実装 (詳細はsample-api-ginを参照)  
  ```
  logger.Error("エラー発生")
  ```

* 出力ログ
  ```
  2019-06-06T16:43:37.015+0900	ERROR	エラー発生
  ```


zap.ConfigからBuildする場合、実行行の出力設定などできるが、出力先の指定が標準出力かファイルしか対応しておらず、`lumberjack`などのWriterを使用できない。  
そのため、Zap.newでLoggerを生成するため、フォーマット面で切るのは上記のレベルまで。


# 開発環境バージョン管理

Date: 2019-05-22

## 目的
Windowsでの開発環境を想定し、Go言語のバージョン管理方法を調査・検討する。

## 調査結果
バージョン管理方法として下記が見つかった。
* Chocolatey  
最新のバージョンがすぐに利用できるわけではなさそう。

* [Go Version Manager for Windows](https://github.com/danielkermode/gvm)  
使用者の記事などが見つからなかったため、現実的ではないように思える。

現時点ではChocolateyを利用するのが良いかと思われる

## Chocolatey利用方法

### Chocolateyのインストール  
#### コマンドプロンプトから下記コマンドでインストール  
`@powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin`

### コマンド
* インストール  
バージョン指定が可能  
`choco install {packageName} -Version {Verison}`

* アップデート  
`choco update {packageName}`  

### ChocolateyのGolangサイト
現時点の最新版は存在する(1.12.5)  
https://chocolatey.org/packages/golang


## 結果
最新のバージョンをリリース直後に利用できるわけではなさそうであるが、  
chocolateyを利用したGoLangのバージョン管理を行う。
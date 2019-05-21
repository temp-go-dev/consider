# 検討リポジトリ

## 検討の進め方

### 作業フロー
ADRを用いた検討の進め方は以下のフローとします。  
作業フロー内に出てくるそれぞれのワードに関しては、後述します。

|#|内容|作業概要|Board対応|
|:---|:---|:---|:---|
|1|検討事項のタスク（Issue）を作成|`issue`、`feature branch`作成、[ADR](#ADR)を作成|ToDo|
|2|担当者が調査・検討・必要あらば検証を行う|`feature branch` への `comimt & push` |In Progress|
|3|チーム内議論を行う(PullRequest)|`PullRequest`の作成|In Discuss|
|4|合意|`PullRequest`の`Mearge`、`Issue`のクローズ|Done|

### ADR
検討の経緯は残しておくことが重要であるため、[Architecture Decision Record](http://thinkrelevance.com/blog/2011/11/15/documenting-architecture-decisions) を用いて管理します。  
簡潔に言いますと、「検討事項のメモを `Mardown` できちんと残しておく」というだけです。  
画像を入れる場合は、別で作成してリンクになると思います。

#### adr-toolsのインストール
ADRを残すための支援ツールとして、[adr-tools](https://github.com/npryce/adr-tools) が提供されています。  
[インストール手順](https://github.com/npryce/adr-tools/blob/master/INSTALL.md) を参考にしてインストールする。  
Mac / Windows10 / Linux に対応していますので、大抵の環境下で利用可能です。  

#### 使い方
1. `temp-go-dev/consider`をclone
1. ルート階層にて `adr new {some title}` を実行。`{some title}` は任意の文字列
1. `####_some_titme.md` が作成される
1. テンプレートが適用された雛形が作成されますが、そこにはあまりこだわらず記載する


## タスク管理

### Issue
`temp-go-dev/consider` の `Issue` にて作成します。  
Issueテンプレートを用意しているので、沿って作成を行います。

[こちら](https://github.com/temp-go-dev/consider/issues/new/choose) からIssueの作成が行えます。  
利用するIssueテンプレートは「`considers`」です。
記載後はプロジェクトに「`consider`」を指定します。

### 進捗状況 
GitHubの [プロジェクトのBoard](https://github.com/orgs/temp-go-dev/projects/1) 機能を利用して確認ができます。  
`Issue` のステータス遷移に従ってある程度自動的に `Board` が動いてくれますが、  
完全に自動化にはなっていないため、所々操作が必要になります。

|Column|意味|操作の必要性|
|:---|:---|:---|
|ToDo|やらなければならないこと。未着手の状態|-|
|In Progress|着手した状態|ToDoに着手する人が移動|
|In Discuss|ADRのPullRequestが作成され討議している状態|PullRequestを作成した人が移動|
|Done|討議が完了し合意されている状態|-|

その他、作成した `PullRequest` についても  [プロジェクトのBoard](https://github.com/orgs/temp-go-dev/projects/1) 機能に表れます。  
こちらは操作を行う必要はありません。

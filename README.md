# 検討リポジトリ

## 検討の進め方

検討の進め方には、


## タスク管理

### Issue
`temp-go-dev/consider` の `Issue` にて作成します。  
Issueテンプレートを用意しているので、沿って作成を行います。

[こちら](https://github.com/temp-go-dev/consider/issues/new/choose) からIssueの作成が行えます。  
利用するIssueテンプレートは「`considers`」です。

### 進捗状況 
GitHubの [プロジェクトのBoard](https://github.com/orgs/temp-go-dev/projects/1) 機能を利用して確認ができます。  
`Issue` や `PullRequest` のステータス遷移に従ってある程度自動的に `Board` が動いてくれますが、  
完全に自動化にはなっていないため、所々操作が必要になります。

|Column|意味|操作の必要性|
|:---|:---|:---|
|ToDo|やらなければならないこと。未着手の状態|-|
|In Progress|着手した状態|ToDoに着手する人が移動|
|In Discuss|ADRのPullRequestが作成され討議している状態|PullRequestを作成した人が移動|
|Done|討議が完了し合意されている状態|-|

その他、作成した `PullRequest` についても  [プロジェクトのBoard](https://github.com/orgs/temp-go-dev/projects/1) 機能に表れます。  
こちらは操作を行う必要はありません。

# WEBアプリの例外処理


## Golangの例外ハンドリング
* `try~catch`はない。
* 関数の戻り値の最後の値として[Errorインターフェース](https://golang.org/pkg/builtin/#error)の返却して例外ハンドリングを行う。



## [pkg/errors](https://godoc.org/github.com/pkg/errors)  
[`pkg/errors`](https://godoc.org/github.com/pkg/errors)を使用することにより、エラーの生成、スタックなどが行えるようになる。  


* エラーの生成  
エラーを新たに生成する場合、下記を使用する。スタックされず、新たなエラーとなる。  
  * `pkg/errors.New`  
  * `Errorf`  

* エラー付与  
エラーをスタックしたい場合、wrapを使用する。  
  * `pkg/errors.Wrap`  
  * `Wrapf`  


* スタックトレースの出力  
スタックトレースを出力する  
  * `pkg/errors.Cause`  
  * `%+v `



### 実装例
CreateTodo内でエラーが発生し、そのエラーをWrapで上位に伝播させることでスタックトレースを出すことができる。  

service.go
```go
// CreateTodosTranNest TransactNestを使用した実装
func (s TodoService) CreateTodosTranNest(todos model.Todos) ([]string, error) {
	db := db.GetDB().Begin()
	todoID := []string{}

	// Transactにトランザクションを行いたい処理を実装した無名関数を渡す
	_, err := TransactNest(db, true, func(tx *gorm.DB) (interface{}, error) {
		// ↓↓↓ トランザクション対象の処理を記載 ↓↓↓

		// insert001 ネストしたトランザクション処理 Begin済みのDBを渡す
		todoID, _ = insert001(tx, todos)

		// 一意制約でエラーにする
		uuid := uuid.New()
		uuidStr := uuid.String()

		for _, todo := range todos.Todo {
			todo.ID = uuidStr
			errEvent := CreateTodo(tx, todo)    //★★★★★★★★★★ エラー発生個所
			if errEvent != nil {
				return nil, errors.Wrap(errEvent, "DBアクセス処理でエラー")
			}
			todoID = append(todoID, uuidStr)
		}
		return todoID, nil
		//↑↑↑ トランザクション対象の処理を記載 ↑↑↑
	})
	if err != nil {
		return nil, errors.Wrap(err, "トランザクション処理でエラー")
	}
	return todoID, nil
}
```
<details><summary>controller.go</summary><div>

```go
// Create1 action: Create /todos
func (pc TodoController) Create1(c *gin.Context) {
	logger := config.GetLogger()

	todos := model.Todos{}
	if err := c.BindJSON(&todos); err != nil {
		logger.Error("Jsonパースエラー",  zap.Error(err))
		c.JSON(http.StatusBadRequest, gin.H{"status": "BadRequest"})
		return
	}

	if len := len(todos.Todo); len == 0 {
		// 0件の場合エラー
		c.JSON(http.StatusBadRequest, gin.H{"status": "BadRequest"})
		return
	}

	var s service.TodoService
	logger.Info("todo作成を実行します")
	p, err := s.CreateTodosTranNest(todos)
	if err != nil {
		logger.Error("error", zap.String("E001", "CreateTodosTranNest処理でエラー発生"), zap.Error(err))
		c.JSON(http.StatusInternalServerError, gin.H{"status": "Internal Server Error", "errMessage": err})
	} else {
		c.JSON(http.StatusOK, gin.H{"createdTodoId": p})
	}
}

```
</div></details>

<details><summary>出力されるログ</summary><div>

```bash
2019-06-10T16:14:32.492+0900	INFO	todo作成を実行します
2019-06-10T16:14:32.559+0900	ERROR	error	{"E001": "CreateTodosTranNest処理でエラー発生", "error": "トランザクション処理でエラー: DBアクセス処理でエラー: sql: transaction has already been committed or rolled back", "errorVerbose": "sql: transaction has already been committed or rolled back\nDBアクセス処理でエラー\ngithub.com/temp-go-dev/sample-api-gin/service.TodoService.CreateTodosTranNest.func1\n\tC:/Users/awaduharatk/go/src/github.com/sample-api-gin/service/todo_service.go:150\ngithub.com/temp-go-dev/sample-api-gin/service.TransactNest\n\tC:/Users/awaduharatk/go/src/github.com/sample-api-gin/service/todo_service.go:210\ngithub.com/temp-go-dev/sample-api-gin/service.TodoService.CreateTodosTranNest\n\tC:/Users/awaduharatk/go/src/github.com/sample-api-gin/service/todo_service.go:136\ngithub.com/temp-go-dev/sample-api-gin/controller.TodoController.Create1\n\tC:/Users/awaduharatk/go/src/github.com/sample-api-gin/controller/todo_controller.go:75\ngithub.com/gin-gonic/gin.(*Context).Next\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/context.go:124\ngithub.com/temp-go-dev/sample-api-gin/server.sampleMiddleware.func1\n\tC:/Users/awaduharatk/go/src/github.com/sample-api-gin/server/server.go:52\ngithub.com/gin-gonic/gin.(*Context).Next\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/context.go:124\ngithub.com/gin-gonic/gin.RecoveryWithWriter.func1\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/recovery.go:83\ngithub.com/gin-gonic/gin.(*Context).Next\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/context.go:124\ngithub.com/gin-gonic/gin.LoggerWithConfig.func1\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/logger.go:240\ngithub.com/gin-gonic/gin.(*Context).Next\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/context.go:124\ngithub.com/gin-gonic/gin.(*Engine).handleHTTPRequest\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/gin.go:389\ngithub.com/gin-gonic/gin.(*Engine).ServeHTTP\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/gin.go:351\nnet/http.serverHandler.ServeHTTP\n\tC:/Go/src/net/http/server.go:2774\nnet/http.(*conn).serve\n\tC:/Go/src/net/http/server.go:1878\nruntime.goexit\n\tC:/Go/src/runtime/asm_amd64.s:1337\nトランザクション処理でエラー\ngithub.com/temp-go-dev/sample-api-gin/service.TodoService.CreateTodosTranNest\n\tC:/Users/awaduharatk/go/src/github.com/sample-api-gin/service/todo_service.go:158\ngithub.com/temp-go-dev/sample-api-gin/controller.TodoController.Create1\n\tC:/Users/awaduharatk/go/src/github.com/sample-api-gin/controller/todo_controller.go:75\ngithub.com/gin-gonic/gin.(*Context).Next\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/context.go:124\ngithub.com/temp-go-dev/sample-api-gin/server.sampleMiddleware.func1\n\tC:/Users/awaduharatk/go/src/github.com/sample-api-gin/server/server.go:52\ngithub.com/gin-gonic/gin.(*Context).Next\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/context.go:124\ngithub.com/gin-gonic/gin.RecoveryWithWriter.func1\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/recovery.go:83\ngithub.com/gin-gonic/gin.(*Context).Next\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/context.go:124\ngithub.com/gin-gonic/gin.LoggerWithConfig.func1\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/logger.go:240\ngithub.com/gin-gonic/gin.(*Context).Next\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/context.go:124\ngithub.com/gin-gonic/gin.(*Engine).handleHTTPRequest\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/gin.go:389\ngithub.com/gin-gonic/gin.(*Engine).ServeHTTP\n\tC:/Users/awaduharatk/go/pkg/mod/github.com/gin-gonic/gin@v1.4.0/gin.go:351\nnet/http.serverHandler.ServeHTTP\n\tC:/Go/src/net/http/server.go:2774\nnet/http.(*conn).serve\n\tC:/Go/src/net/http/server.go:1878\nruntime.goexit\n\tC:/Go/src/runtime/asm_amd64.s:1337"}

```
</div></details>



## panic
* `panic`が生成されるとその場で処理を中断し、呼び出し元までたどって終了する。ランタイムエラー。  

* 存在しないスライスへのアクセス、初期化されていないポインタへのアクセスなどで発生することがある。  
→Golangでは事前にnilチェックなどを行い、panic/recoverを使用しないようにコーディングする方針となっている。  
ネストが深くなりすぎて処理を戻すのが大変になっている場合などに使用されているらしい。

## recover
`defer`とセットで使用する。  
`recover`を呼び出すと発生した`panic`の内容をトレースできる。　　

### 実装サンプル  
txFunc(tx)で発生した`panic`を`defer`内で`recover`して後処理を行っている。  
```go
// Transact トランザクション実行
func Transact(db *gorm.DB, txFunc func(*gorm.DB) (interface{}, error)) (data interface{}, err error) {
	tx := db.Begin()
	if tx.Error != nil {
		return nil, tx.Error
	}
	defer func() {
		if p := recover(); p != nil {
			tx.Rollback()
			panic(p)
		} else if err != nil {
			tx.Rollback()
		} else {
			err = tx.Commit().Error
		}
	}()
	// 無名関数にBeginしたDBを渡して実行する
	data, err = txFunc(tx)
	return
}
```



# memo
[Golangのエラーハンドリングの基本](https://qiita.com/shoichiimamura/items/13199f420ebaf0f0c37c#%E3%82%A8%E3%83%A9%E3%83%BC%E3%81%AB%E5%BF%9C%E3%81%98%E3%81%A6%E3%82%B9%E3%83%86%E3%83%BC%E3%82%BF%E3%82%B9%E3%82%B3%E3%83%BC%E3%83%89%E3%82%92%E9%81%B8%E3%81%B6)


[【Golang Error handling】エラーの種類によって処理を分けるBESTな方法](https://qiita.com/yoshinori_hisakawa/items/15bf0307245744deb4fc)


[logging](https://re-engines.com/2018/11/05/go%E8%A8%80%E8%AA%9E%E3%81%AE%E3%82%A8%E3%83%A9%E3%83%BC%E3%83%8F%E3%83%B3%E3%83%89%E3%83%AA%E3%83%B3%E3%82%B0%E3%81%A8%E3%83%AD%E3%82%B0%E3%83%AD%E3%83%BC%E3%83%86%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3/)

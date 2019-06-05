# データベースアクセス

## データベースアクセスとは
RDBへGoのアプリケーションからアクセスすることを指している。  
データベースへのアクセスはGoLang自体のオリジナルパッケージでも可能となる。  
ドライバは別途。  

## ORMapperパッケージを利用するかどうか
GoLangには、`database/sql` パッケージが存在する。  
このパッケージで自分でデータベースアクセスを細かく記載することは可能となる。なお、ConnectionPool関連の機能はこのパッケージに機能を有する。

- [本家](http://go-database-sql.org/connection-pool.html)
- [日本語訳](https://golang.shop/post/go-databasesql-10-connection-pool-ja/)

例えば、DBから複数行のレコードをトランザクション制御なしで取得するコードは以下となる。

```go
db, err := sql.Open("mysql", "user:password@tcp(localhost:3306)/sampledb")
if err != nil {
    fmt.Println(err.Error())
    return []model.User{}, err
}
defer db.Close()

var (
    id         string
    firstName  string
    lastName   string
    email      string
    passsword  string
    phone      string
    userStatus int
    version    int
)

var users []model.User

rows, err := db.Query("SELECT id, first_name, last_name, email, password, phone, userStatus, version FROM user ORDER BY id")
if err != nil {
    fmt.Println(err)
    return []model.User{}, err
}
defer rows.Close()
for rows.Next() {
    err := rows.Scan(
        &id, &firstName, &lastName, &email, 
        &passsword, &phone, &userStatus, &version)
    if err != nil {
        fmt.Println(err)
    }
    user := model.User{}
    user.Id = id
    user.FirstName = firstName
    user.LastName = lastName
    user.Email = email
    user.Password = passsword
    user.Phone = phone
    user.UserStatus = userStatus
    user.Version = version
    users = append(users, user)
}
```

## ORMを利用すると、どう楽になるのか  

### GORMを使用した実装例

* 行数が少なく済む、構造体へのマッピングが楽  
* 生SQLの記載が可能。  
GORM特有の書き方もできるが、TBL結合などが発生した場合の記法が特殊なので生SQLで統一してしまったほうが良いと思われる。  


main.go

```go
// GetAllTodo ユーザIDに紐づくTODOを取得
func (s TodoService) GetAllTodo(uid string) ([]model.Todo, error) {
	db := db.GetDB()
	todos := []model.Todo{}

	// SELECT実行
        err := db.Raw("SELECT * FROM todo where user_id = ?", uid).Scan(&todos).Error

        // GORM Find を利用したSELECT
	// tx.Table("todo").Where("user_id = ?", uid).Find(&todos)
	if err != nil {
		return nil, err
	}
	return todos, nil
}

```

model.go

```go
// Todo TodoTBLの構造体
type Todo struct {
	ID          string    `json:"id" gorm:"column:id"`
	UserID      string    `json:"owner" gorm:"column:user_id"`
	Title       string    `json:"title" gorm:"column:title"`
	Contents    string    `json:"contents" gorm:"column:contents"`
	Start       time.Time `json:"start" gorm:"column:start"`
	Due         time.Time `json:"due" gorm:"column:due"`
	ActualStart time.Time `json:"actualStart" gorm:"column:actualStart"`
	ActualEnd   time.Time `json:"actualEnd" gorm:"column:actualEnd"`
	Status      int       `json:"status" gorm:"column:status"`
	Version     int       `json:"version" gorm:"column:version"`
}
```

## GORMについて


* コネクションプールなどの設定が可能。`database/sql`をラップしている。  
// SetMaxIdleConnsはアイドル状態のコネクションプール内の最大数を設定  
db.DB().SetMaxIdleConns(10)  

* Timeout値などはDNSパラメータとして設定可能  [その他の設定可能値](https://github.com/go-sql-driver/mysql#dsn-data-source-name)    

* gorm.open時のパラメータとして`?parseTime=true`が必要。  
設定することでMySQL側でGOで扱える形式に変換してから取得できるようになる。  
設定しない場合は`time.Time`型へのパース時にエラーが発生する。


* 今回はMySQLドライバを使用したが、`database/sql`のオブジェクトであれば外部のドライバも使用可能らしい。[参考](https://github.com/jinzhu/gorm/issues/1009)




db.go
```go
package db

import (
	"time"

	_ "github.com/go-sql-driver/mysql"
	"github.com/jinzhu/gorm"
	"github.com/temp-go-dev/sample-api-gin/config"
)

var (
	db  *gorm.DB
	err error
)

// Init DB初期化
func Init() {
	prop := config.GetProperties()

	// parseTime=trueを指定しないとdatetime→time.Timeへの変更でエラーが発生する。
	CONNECT := prop.User + ":" + prop.Pass + "@" + prop.Protocol + "/" + prop.Dbname + "?parseTime=true" + "&readTimeout=10s"
	db, err = gorm.Open(prop.Dbms, CONNECT)

	if err != nil {
		//　err発生時の処理要検討
		panic(err.Error())
	}
	// DBデバッグログの出力設定
	db.LogMode(true)
	// SetMaxIdleConnsはアイドル状態のコネクションプール内の最大数を設定
	db.DB().SetMaxIdleConns(10)
	// SetMaxOpenConnsは接続済みのデータベースコネクションの最大数を設定
	db.DB().SetMaxOpenConns(100)
	// SetConnMaxLifetimeは再利用され得る最長時間を設定
	db.DB().SetConnMaxLifetime(time.Hour)

}

// GetDB is called in models
func GetDB() *gorm.DB {
	return db
}

```



## MySQLドライバー

* [go-sql-driver/mysql](https://github.com/go-sql-driver/mysql#requirements)  
MariaDBなどにも対応している




## トランザクション制御

Begin → Rollback or Commitの操作が必要。  
内包した関数を[実装](https://github.com/temp-go-dev/sample-api-gin/commit/859bc6e9aeb2a95bcf1cae6ad5209c1e3e9f9039)

ネストさせる場合、関数の外でBeginして持ちまわすように実装を行うことで一応可能
[[ネスト実装]](https://github.com/temp-go-dev/sample-api-gin/commit/63eb5f4897dc1c576c8f8dfaf7cb03810ff854d3#diff-f38dafae52d1032820799499212f8868R145)


```go
// CreateTodos Todoの作成
func (s TodoService) CreateTodos(todos model.Todos) (string, error) {
	db := db.GetDB()

	// //トランザクション開始
	tx := db.Begin()
	if tx.Error != nil {
		return "", tx.Error
	}

	// Create実行
	for _, todo := range todos.Todo {
		err := tx.Table("todo").Create(&todo).Error
		if err != nil {
			fmt.Println("DB error")
			// ロールバックして終了
			tx.Rollback()
			return "", err
		}
	}
	// コミットして終了
	tx.Commit()
	return "", nil
}
```



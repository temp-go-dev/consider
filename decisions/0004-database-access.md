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

// TODO：似た機能をGORMで実装

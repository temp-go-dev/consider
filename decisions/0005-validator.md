# Validator

## validatorパッケージ
* [go-playground/validator](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=3&cad=rja&uact=8&ved=2ahUKEwiOt_XB3OPiAhXFw7wKHd1xCa0QFjACegQIBBAB&url=https%3A%2F%2Fgodoc.org%2Fgopkg.in%2Fgo-playground%2Fvalidator.v9&usg=AOvVaw29EU5F7DLBg7UTOctWxmfW)  
* [asaskevich/govalidator](https://github.com/asaskevich/govalidator)  
 githubのスター数3.4kで`go-playground/validator`と同じぐらい使われている。  
 Readmeを流し読みした感じは上記と変わらないように感じたため今回は触っていない。  

Gin(v1.4.0)は内部で[https://github.com/go-playground/validator/tree/v8.18.2](https://github.com/go-playground/validator/tree/v8.18.2)を使用している。


## 実装(go-playground/validator)

* 構造体のタグに指定する。  
`v-post:"required"`  
POSTやPUTリクエストで同じ構造体を使用しているため、POST用のタグとして`v-post`を定義している  
使用可能なタグは[ここ](https://godoc.org/gopkg.in/go-playground/validator.v8#hdr-Baked_In_Validators_and_Tags)を参照
  ```go
    // User ユーザTBLの構造体
  type User struct {
  	ID         string `json:"id" gorm:"column:id" v-post:"required"`
  	Name       string `json:"username" gorm:"column:name" v-post:"required"`
  	FirstName  string `json:"firstName" gorm:"column:first_name"`
  	LastName   string `json:"lastName" gorm:"column:last_name"`
  	Email      string `json:"Email" gorm:"column:email"`
  	Password   string `json:"Password" gorm:"column:password"`
  	Phone      string `json:"Phone" gorm:"column:phone"`
  	UserStatus int    `json:"UserStatus" gorm:"column:userStatus"`
  	Version    int    `json:"Version" gorm:"column:version"`
  }
  ```


* validationの実行

  ```go
  // Create action: Create /users
  func (pc UserController) Create(c *gin.Context) {
  	logger := config.GetLogger()
  	u := model.User{}

  	fmt.Println("create")
  	if err := c.BindJSON(&u); err != nil {
  		fmt.Println("vali Error")
  		c.JSON(http.StatusBadRequest, gin.H{"status": "BadRequest"})
  		return
  	}

 	// タグ v-post にもとづき validation
	config := &validator.Config{TagName: "v-post"}
	validate := validator.New(config)
	if err := validate.Struct(u); err != nil {
		fmt.Println("v-post vali Error")

		for _, err := range err.(validator.ValidationErrors) {
			fmt.Printf("errField:%s ", err.Field)
			fmt.Printf("errType:%s\n", err.Tag)
		}

		logger.Error("error", zap.Error(err))
		c.JSON(http.StatusBadRequest, err)
		return
	}

  	var s service.UserService
  	p, err := s.CreateUser(u)
  	if err != nil {
  		c.AbortWithStatus(404)
  		fmt.Println(err)
  	} else {
  		c.JSON(200, p)
  	}
  }
  ```

上記実装でチェックエラーとなるようにリクエストを投げると下記の構造体が返却される。  
このままではリクエストした側からすると何のエラーなのかわからない。  
validatorから返却されるエラーからフィールド情報などを取り出してエラーメッセージを生成する部分を作りこむ必要がありそう。

```bash
$ curl -X POST "http://localhost:8080/users" -H "accept: */*" -H "Content-Type: application/json" -d "{\"id123\":\"1234\",\"username123\":\"string\",\"firstName\":\"string\",\"lastName\":\"string\",\"email\":\"string\",\"password\":\"string\",\"phone\":\"string\",\"userStatus\":0,\"version\":2}"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   506  100   345  100   161  10147   4735 --:--:-- --:--:-- --:--:-- 14882
{"User.ID":{"FieldNamespace":"User.ID","NameNamespace":"ID","Field":"ID","Name":"ID","Tag":"required","ActualTag":"required","Kind":24,"Type":{},"Param":"","Value":""},"User.Name":{"FieldNamespace":"User.Name","NameNamespace":"Name","Field":"Name","Name":"Name","Tag":"required","ActualTag":"required","Kind":24,"Type":{},"Param":"","Value":""}}
```

* printした結果
  ```
  errField:ID errType:required
  errField:Name errType:required
  [GIN] 2019/06/12 - 18:56:14 |[90;43m 400 [0m|     45.0351ms |             ::1 |[97;46m POST    [0m /users
  ```



## 参考
[go-playground/validator リクエストパラメータ向けValidationパターンまとめ](https://qiita.com/RunEagler/items/ad79fc860c3689797ccc#%E3%83%90%E3%83%AA%E3%83%87%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E5%86%85%E5%AE%B9%E3%82%92%E8%87%AA%E4%BD%9C%E3%81%99%E3%82%8B%E5%A0%B4%E5%90%88)



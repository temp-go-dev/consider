# テストフレームワーク

## パッケージ

### testing(標準)

　 テストコードのルール  
テストコードを実装する際、3つのルールがある。

パッケージ　：testingパッケージをインポートする  
ファイル名　：xxx_test.go  
テスト関数名：func TestXxx (t *testing.T)

　　このルールに則ることで、コードをビルドする時にテストコードが除外され、テスト実行コマンドでテストコードのみがビルドされるようになる。

  ```go
  func TestSub1(t *testing.T) {
  	a := Saga(111, "fff")
  	b := "sa"
  	if a != b {
  		t.Fatal("fatal")
  	}
  }
  ```

TableDrivenTests（公式よりこういう書き方を推薦する）
```go
type atoi64Test struct {
  in  string
  out int64
  err error
}

var atoi64tests = []atoi64Test{
  {"", 0, ErrSyntax},
  {"0", 0, nil},
  {"-0", 0, nil},
  {"1", 1, nil},
  {"-1", -1, nil},
}


func TestParseInt64(t *testing.T) {
  for i := range atoi64tests {
    test := &atoi64tests[i]
    out, err := ParseInt(test.in, 10, 64)
    if test.out != out || !reflect.DeepEqual(test.err, err) {
      t.Errorf("Atoi64(%q) = %v, %v want %v, %v",
        test.in, out, err, test.out, test.err)
    }
  }
}
``` 

特別な書き方
1. Examples  
　　Exampleから始まる特別の書き方で、出力を`//OutPut:`で出力もののテストができる
　　
  ```go
  func ExampleA() {
  	fmt.Println(Vs(1, "2"))
      //OutPut:"結果1"
  }
  ```

2. Unordered output  
 順不同な結果に対してマッチできる。出力を`Unordered output:`で指定する
  ```go
    func ExampleUnordered() {
  	  fmt.Println(Vs(1, "2"))
  	  fmt.Println(Vs(1, "1"))
  	  //Unordered output:hello
  	  //hey!!
 　 }
  ```

  **vscodeの場合、gotestsを使って、テストケースが自動生成できる。  
  手順：関数をダブルクリック→右クリック→GO:Generate Unit Tests For Functionをクリック**

## mock(insterface)

下記３種のやり方がある。

  1.testingパッケージのみ使う  
  
- メリット
  - 標準パッケージのみでいける。
- デメリット  
  - 実装の置き換えしかできない 
  - 呼ばれた回数などのテストをする場合mockインターフェースの定義が複雑になる

sampleコード 
```go
//実装

type Gist struct {
	Rawurl string
}

//GistsのAPIにリクエストするインターフェース
type Doer interface {
	doGistRequest(user string) (io.Reader, error)
}

//List APIを扱うためのクライアント実装
type Client struct {
	Gister Doer
}

type Gister struct{}

//ここをモックかする
func (g *Gister) doGistRequest(user string) (io.Reader, error) {
	fmt.Println("wawawawawawawa")
	return nil, nil
}

func (c *Client) ListGists(user string) (io.Reader, error) {
	r, err := c.Gister.doGistRequest(user)
	if err != nil {
		return nil, err
	}
	return r, nil
}
```

```go
//test

//Doerのダミー
type dummyDoer struct{}

//doGistRequestのダミー実装
func (d *dummyDoer) doGistRequest(user string) (io.Reader, error) {
	return nil, nil
}

func TestGetGists2(t *testing.T) {

    //dummyDoerはDoerの実装なので、Clientに渡せる。
	c := &Client{&dummyDoer{}}
	urls, err := c.ListGists("test")
	if err != nil {
		t.Fatal("args")
	}
	if urls != nil {
		t.Fatal("aaaaa")
	}
}
```

　2.gomockパッケージを使う  
- メリット
  - コマンド１発でモックが生成でき、生成したモックを使ってテストできる。(使用コマンド例:  mockgen -source sample.go -destination mock_sample/)

- デメリット  
  - ファイルが増えるため、管理がしにくくなる恐れがある。



  3.testifyパッケージのみ使う  

- メリット
  - mockのほかにassert,Suiteなどかもあるため、テストが多少簡易化できる
  - 実行回数なども簡単に確認できる

- デメリット  
  - mockの実装は１とあんまり変わらない
  - 公式ではない
  
```go
//実装
// お天気クライアントのinterface
type WeatherClient interface {
	RequestWeather() string
}

// お天気クライアントの実態の構造体
type WeatherClientImpl struct{}

// お天気をリクエストする関数
func (c WeatherClientImpl) RequestWeather() string {
	fmt.Println("天気をリクエストするクライアント")
	// 本当は天気をリクエストする処理とかがここに入る
	return "晴れ"
}

// お天気を扱うサービスの構造体(interfaceの実態)
type WeatherService struct {
	client WeatherClient
}

// お天気取得関数
func (w WeatherService) GetWeather() string {
	fmt.Println("天気を取得する")
	return w.client.RequestWeather()
}
```
```go
//test
// モック生成
type weatherClientMock struct {
	mock.Mock
}

// モック関数定義
func (mock *weatherClientMock) RequestWeather() string {
	fmt.Println("モック実行")
	result := mock.Called() // 呼び出し
	return result.String(0) // モックの返却値 Returnの引数を返却する
}

// くもりの時のテスト
func TestRequestWeather1(t *testing.T) {
	assert := assert.New(t)

	// モック準備
	weatherClientMock := new(weatherClientMock)
	weatherClientMock.On("RequestWeather").Return("くもり") // このテストではモックに"くもり"を返却するように指定している

	// テスト実行
	weatherService := WeatherService{client: weatherClientMock}
	weather := weatherService.GetWeather()

	// 検証
	assert.Equal(weather, "くもり") // 天気サービスが"くもり"を返却しているか検証
}
```




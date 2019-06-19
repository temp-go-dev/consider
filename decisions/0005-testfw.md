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

TableDrivenTests
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

  vscodeの場合、gotestsを使って、テストケースが自動生成できる。（関数をダブルクリック→右クリック→GO:Generate Unit Tests For Function)
---
title:  "백엔드 시리즈 5. Golang Unit Test 작성하기"
date:   2021-08-03 12:27:19 +0900
permalink: ":categories/backend/:title"
---

> 원문 출처: [Techschool](https://www.youtube.com/watch?v=rx6CPDK_5mU&list=PLy_6D98if3ULEtXtNSY_2qN21VCKgoQAE&ab_channel=TECHSCHOOL "Tech School")

### 백엔드 시리즈 5. Golang Unit Test 작성하기

앞선 시리즈에서 만든 CRUD operation 코드에 대한 유닛 테스트를 작성해봅시다.

#### CreateAccount 유닛 테스트

1. `./sqlc/` 폴더에 테스트 코드가 들어갈 `account_test.go`파일을 하나 만들어 줍니다. `go`언어에서 `best practice` 또는 `convention`은 특정 코드의 테스트 파일은 동일한 디렉토리에 위치하는 것입니다.
1. 그리고 테스트 템플릿을 작성해 줍니다

    ```go
      package db

      import "testing"

      // #1
      func TestCreateAccount(t *testing.T) {

      }

    ```

    1. `go`의 유닛 테스트 function signature는 `Test`로 시작해야 합니다. 그리고 `testing.T` 객체를 인풋으로 받습니다. 이 `T` 객체를 통해 테스트의 상태를 관리할 수 있습니다.

이제 `account.sql.go` 파일로 가서 테스트를 진행할 function을 살펴 보도록 합시다.
  
  ```go
    func (q *Queries) CreateAccount(ctx context.Context, arg CreateAccountParams) (Account, error) {
      row := q.db.QueryRowContext(ctx, createAccount, arg.Owner, arg.Balance, arg.Currency)
      var i Account
      err := row.Scan(
        &i.ID,
        &i.Owner,
        &i.Balance,
        &i.Currency,
        &i.CreatedAt,
      )
      return i, err
    }
  ```

  `CreateAccount()`는 `Queries` 객체의 메쏘드로 정의가 되어있기 때문에 database와 연결이 되어있어야 합니다. 따라서 테스트를 진행하기 위해 db와 연결과 `Queries` 객체 세팅부터 진행해야 합니다. 이를 진행하기 위해 `main_test.go`파일을 만들어 줍니다.

#### main_test.go 파일 추가하기

  ```go
    package db
    
    import (
      "database/sql"
      "log"
      "os"
      "testing"
    )

    var testQueries *Queries // #1

    const ( // #3
      dbDriver = "postgres"
      dbSource = "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable"
    )

    func TestMain(m *testing.M) { // #2
      conn, err := sql.Open(dbDriver, dbSource) // #4
      if err != nil { // #5
        log.Fatal("cannot connect to db:", err)
      }
      
      testQueries = New(conn) // #6
      os.Exit(m.Run()) // #7
    }
  ```

1. `testQueries`라는 전역 변수를 정의해줍니다.
1. `TestMain` function은 현재 작업중인 `db` 패키지의 유닛테스트 시작 포인트입니다.
1. `dbDriver`와 `dbSource`는 우선 `const`에 정의해 줍니다.
1. `sql.Open()`을 사용하여 `db`와의 연결을 시작합니다.
1. `err`가 `nil`이 아닌경우엔 문제가 발생했음을 로깅합니다.
1. `err`가 `nil`인 경우 `./sqlc/db.go`에서 `sqlc`가 자동으로 만든 `New()` function에 `conn`을 넣어 `testQueries` 객체를 생성해 줍니다.
1. `m.Run()`을 사용하여 유닛 테스트를 시작합니다. 이 function은 `exit code`를 리턴하는데 테스트가 성공인지 실패인지 여부를 알려줍니다. 그리고 `os.Exit` 커멘드를 사용하여 `test runner`에게 테스트 결과를 알려줍니다.

이제 `run test`를 진행해 봅니다.

결과:
  
  ```zsh
    2021/08/07 00:55:58 cannot connect to db:sql: unknown driver "postgres" (forgotten import?)
  ```

실패가 떴습니다. 메세지는 "db와 연결할 수 없습니다. 알 수 없는 driver입니다." 를 말하고 있습니다. 이 에러가 나타나는 이유는 `database/sql` 패키지가 SQL 데이터베이스의 기본적이 인터페이스만을 제공하기 때문입니다. 특정 `db driver` (우리의 경우 `postgres`)를 `db engine`으로 사용하기 위해 새로운 디펜던시를 설치해 주어야 합니다. 우리는 `lib/pq` 드라이버를 사용하도록 하겠습니다.

#### lib/pq 설치하기

[lib/pg](https://github.com/lib/pq)를 커맨드 라인을 사용하여 설치해 보도록 하겠습니다. 터미널을 열고 패키지를 설치해 줍니다.

입력:

  ```zsh
    go get github.com/lib/pq
  ```

출력:

  ```zsh
    go: downloading github.com/lib/pq v1.10.2
    go get: added github.com/lib/pq v1.10.2
  ```

`./go.mod`파일이 새로 생성된걸 확인할 수 있습니다. 그리고 파일을 클릭하여 들어가보면 새로운 디펜던시가 설치된걸 볼 수 있습니다.

  ```go
    require github.com/lib/pq v1.10.2 // indirect
  ```

`indrect`라고 표시가 되어있고 워닝이 뜨는 이유는 아직 해당 패키지를 사용하지 않았기 떄문입니다. 패키지를 사용하기 위해 `github.com/lib/pq`를 복사하고 `main_test.go`파일로 돌아가 `import`에 추가해 줍니다.

  ```go
    import "github.com/lib/pq"
  ```

하지만 이렇게 그대로 내버려 둔다면 코드에서 직접적으로 `lib/pq`의 기능을 사용하지 않기때문에 `go formatter`가 해당 `import`문을 삭제하게 됩니다. 계속 `"github.com/lib/pq"`문을 유지하기 위에 앞에 `_`를 추가해 줍니다.

  ```go
      import (
      "database/sql"
      "log"
      "os"
      "testing"

      _ "github.com/lib/pq" 
)
  ```

이제 문서를 저장하고 `run test`를 다시 시도해봅니다.

  ```zsh
    ok  github.com/mannerism/simplebank/db/sqlc 0.454s [no tests to run]
  ```

테스트가 정상적으로 완료된걸 확인할 수 있습니다.

이제 터미널에 `go mod tidy`를 실행시키면 `go.mod`파일에 `indirect`문이 사라진걸 확인할 수 있습니다.

  ```go
    require github.com/lib/pq v1.10.2
  ```

#### testify 설치하기

[testify](https://github.com/stretchr/testify)를 커맨드라인을 사용하여 설치해 줍니다. `testify`를 사용하면 테스트 결과를 `if-else`구문 보다 간단하고 쉽게 확인할 수 있습니다. 터미널에 다음의 커멘드를 입력해 줍니다.

  입력:
  
  ```zsh
    go get github.com/stretchr/testify
  ```
  
  출력:

  ```zsh
    go: downloading github.com/stretchr/testify v1.7.0
    go: downloading github.com/pmezard/go-difflib v1.0.0
    go: downloading github.com/davecgh/go-spew v1.1.0
    go: downloading github.com/stretchr/objx v0.1.0
    go: downloading gopkg.in/yaml.v3 v3.0.0-20200313102051-9f266ea9e77c
    go get: added github.com/stretchr/testify v1.7.0
  ```

`account_test.go`파일로 돌아가 `testify/require`패키지를 `import`해 줍니다.

  ```go
  import "github.com/stretchr/testify/require"
  ```

#### `CreateAccount()` 유닛 테스트 계속

이제 세팅은 완료가 되었습니다. 다시 `account_test.go`파일로 돌아가서 첫번째 유닛테스트를 작성해 보도록 하겠습니다.

```go
  func TestCreateAccount(t *testing.T) {
    // #1
    arg := CreateAccountParams{
      Owner:    "tom",
      Balance:  100,
      Currency: "USD",
    }

    // #2
    account, err := testQueries.CreateAccount(context.Background(), arg)

    // #3
    require.NoError(t, err)
    require.NotEmpty(t, account)

    require.Equal(t, arg.Owner, account.Owner)
    require.Equal(t, arg.Balance, account.Balance)
    require.Equal(t, arg.Currency, account.Currency)

    require.NotZero(t, account.ID)
    require.NotZero(t, account.CreatedAt)
  }    
```

1. 테스트에 사용할 `CreateAccountParams`를 새로 만들어줍니다.
1. `CreateAccount()` function을 테스트해 봅니다. `Background()` context와 `#1`에서 생성한 `arg`를 인풋으로 넣어줍니다.
1. `testify/require` 패키지에 포함 된 `NoError`, `NotEmpty`등과 같은 function을 사용하여 테스트 결과를 확인합니다.

>주의: 테스트를 정상적으로 진행하기위해 `docker`가 돌아가고 있어야 합니다.

이제 `account_test.go`에서 `run test`를 진행합니다.

결과:

```zsh
  ok  github.com/mannerism/simplebank/db/sqlc (cached)
```

 테스트가 정상적으로 돌아갑니다.

 `main_test.go`파일 맨 위줄에 보이는 `run package test`를 클릭하면 본 패키지에 포함된 모든 유닛테스트를 실행하게 되며, 테스트가 완료 된 개별 function은 초록색으로 색깔이 칠해집니다. 그리고 코드 커버리지역시 콘솔에서 확인할 수 있습니다.

```zsh
  ok  github.com/mannerism/simplebank/db/sqlc 0.477s coverage: 6.5% of statements
```

`TablePlus`에 있는 `simple_bank` `db`에도 테스트 값이 정상적으로 입력된걸 확인할 수 있습니다.

#### 랜덤 값으로 유닛 테스트 진행

`TestCreateAccount()`에 `arg`로 추가한 `CreateAccountParams`의 값으로 저희가 직접 값을 입력을 했지만, 이 값을 자동 생성하면 코드가 깔끔해지고 써야하는 코드가 적어진다는 장점이 있습니다. 그리고 테스트하는 값이 랜덤으로 생성되기 때문에 테스트간 충돌을 방지할 수도 있습니다.`simplebank/`폴더 안에 새로운 `util`폴더를 만들어줍니다. 그리고 `util`폴더 안에 `random.go`파일은 한개 생성해 줍니다.

먼저 `init()` function 을 작성해 줍니다. 이 function은 `util`패키지가 처음 사용될 때 실행됩니다.

  ```go
    package util

    func init() {
      rand.Seed(time.Now().UnixNano())
    }
  ```

`rand.Seed()`을 사용하여 seed 값을 입력해 줍니다. 보통 seed값은 현재 시간을 입력해 줍니다. `rand.Seed()`는 `int64`로 인풋값을 받기 때문에 우리는 현재 시간인 `time.Now()`를 `UnixNano()`를 사용해 `int64`로 변환해 줍니다.

이렇게 되면 매번 다른 seed 값을 사용하게 되며 생성되는 값 역시 매번 달라지게 됩니다. 만약 `rand.Seed()`를 사용하지 않는다면 이 랜덤값 생성기는 seed값을 `1`이라 간주하고 매번 똑같은 값을 생성하게 됩니다.

이제 random integer를 생성하는 function을 만들어 봅시다.

  ```go
    func RandomInt(min, max int64) int64 {
      return min + rand.Int63n(max-min+1)
    }
  ```

`RandomInt()` function은 두 개의 `int64`숫자인 `min`, `max`를 입력 받고 `min`과 `max`사이의 랜덤 `int64` 숫자를 리턴합니다.

`rand.Int63n(n)` function은 `0`과 `n-1`사이의 랜덤 `int64` 숫자를 리턴 합니다. 따라서 `rand.Int63n(max-min+1)`은 `0`과 `max - min`사이의 랜덤 숫자를 리턴합니다.

따라서, `min`을 마지막에 더해주게 되면 `min`과 `max`사이의 숫자가 리턴되게 됩니다.

다음으로 `n`개의 `character`를 갖게되는 랜덤 스트링을 제공하는 function을 만들어 보겠습니다.

```go
  const alphabet = "abcdefghijklmnopqrstuvwxyz"   // #1
  func RandomString(n int) string {               // #2
    var sb strings.Builder                        // #3
    k := len(alphabet)                            // #4

    for i := 0; i < n; i++ {                      // #5
      c := alphabet[rand.Intn(k)]                 // #6
      sb.WriteByte(c)                             // #7
    }
    return sb.String()                            // #8
  }
```

1. random string을 생성할 26 character 알파벳을 정의해줍니다.
1. `string builder` 객체를 생성해 줍니다.
1. 1번에서 정의한 총 알파벳 갯수를 `k` 변수에 저장합니다.
1. `for-loop`을 사용하여 `n`개의 알파벳을 생성합니다.
1. `rand.Intn(k)`를 사용하여 `0` 와 `k-1`사이의 랜덤 포지션 값을 생성해 줍니다. 그리고 해당 포지션의 `character`를 `c`변수에 저장합니다.
1. 매 `loop iteration`마다 랜덤으로 선택된 `character` `c`를 `sb.WriteByte()`를 사용하여 string builder에 추가합니다.
1. string builder에 추가된 `c`값들을 `sb.ToString()`를 사용하여 `string`으로 변환 후 리턴합니다.

`RandomString()` function을 사용하여 랜덤 `owner`명을 만들어주는 function도 만들어 줍니다. `RandomOwner()`는 6개의 character를 갖는 랜덤 `owner`명을 생성합니다.

```go
  func RandomOwner() string {
    return RandomString(6)
  }    
```

 그리고 `RandomInt()` function을 사용하여 랜덤 `money`를 만들어주는 function을 만들어 줍니다. `RandomMoney()`는 0 과 1000 사이의 랜덤 `int64`를 생성합니다.

```go
  func RandomMoney() int64 {
    return RandomInt(0, 1000)
  }
```

랜덤 `currency`를 생성하는 function도 하나 만들어 줍니다.

```go
  func RandomCurrency() string {
    currencies := []string{"EUR", "USD", "CAD"} // #1
    n := len(currencies)                        // #2
    return currencies[rand.intn(n)]             // #3
  }
```

1. `currencies` list를 `string slices` 형태로 만들어 줍니다.
1. `len()`을 사용하여 `currencies`의 `length`를 `n`변수에 저장해 줍니다.
1. `rand.intn()`을 사용하여 0 부터 n사이 랜덤 인덱스를 사용하여 1개의 통화를 `string`값으로 리턴해 줍니다.

다시 `account_test.go`파일로 돌아가서 `CreateAccountParams`값을 방금 만든 랜덤값으로 변경해 줍니다.

```go
  func TestCreateAccount(t *testing.T) {
    arg := CreateAccountParams{
      Owner:    util.RandomOwner(),
      Balance:  util.RandomMoney(),
      Currency: util.RandomCurrency(),
    }

    ...
  }
```

이제 다시 유닛 테스트를 실행하고 `TablePlus`를 새로고침하면 새롭게 랜덤값이 `simple_bank` `db`에 입력된것을 확인할 수 있습니다.

이제 보다 쉽게 유닛테스트를 진행하기 위해 `Makefile`에 유닛 테스트 커맨드를 추가해 주겠습니다. 커멘드는 비교적 간단합니다. `go test` 커멘드를 사용하고 `-v`옵션을 추가하여 `verbose logs`를 활성화 시킵니다. 그리고 `-cover`옵션을 사용하여 `code coverage`를 활성화 합니다.

```makefile
  test:
    go test -v -cover ./...
```

본 프로젝트에는 1개 이상의 패키지가 존재할 것이기 때문에 `./...` `argument`를 사용하여 모든 유닛테스트를 실행할 수 있습니다.

터미널에 `make test`를 진행하면 `verbose`로그를 프린트하고 코드 커버리지를 콘솔에 보여주는걸 확인할 수 있습니다. 다시 `TablePlus`로 돌아가 새로운 테스트에 대한 기록이 잘 보이는지 확인해 줍니다.

이제 나머지 `CRUD` operation에 대한 유닛 테스트 작성을 해 봅니다.

#### GetAccount 유닛 테스트

`GetAccount()`를 테스트하기 위해서는 우선 `CreateAccount()`가 선행되어야 합니다. 방금 전에 `CreateAccount()`를 테스트하며 새로운 `account`가 생겼지만, 모든 유닛테스트는 각각의 유닛 테스트로부터 독립된 테스트를 수행하는것이 가장 바람직합니다. 이는 프로젝트가 커지면서 수백개, 심지어 수천개의 유닛테스트를 진행하게 될 것인데 개별 유닛테스트가 서로의 결과에 영향을 주게되면 코드를 관리하기가 힘들뿐더러 유닛테스트 자체의 `integrity`, 즉 `신뢰도`가 매우 떨어지기 때문입니다.

따라서 우리가 작성하는 각 테스트마다 새로운 `account`기록을 만들어 주겠습니다. 이를 위해 이미 작성된 `TestCreateAccount()` 코드를 따로 빼어 `createRandomAccount()` function으로 이동시켜 줍니다.

```go
  func createRandomAccount(t *testing.T) Account {
    arg := CreateAccountParams{
      Owner:    util.RandomOwner(),
      Balance:  util.RandomMoney(),
      Currency: util.RandomCurrency(),
    }

    account, err := testQueries.CreateAccount(context.Background(), arg)
    require.NoError(t, err)
    require.NotEmpty(t, account)

    require.Equal(t, arg.Owner, account.Owner)
    require.Equal(t, arg.Balance, account.Balance)
    require.Equal(t, arg.Currency, account.Currency)

    require.NotZero(t, account.ID)
    require.NotZero(t, account.CreatedAt)

    return account
}
```

`createRandomAccount()`는 앞에 `Test`가 붙지 않기때문에 유닛 테스트를 실행하여도 실행이되지 않습니다. 또한, `c`가 소문자이기 때문에 `account_test.go` 파일 내부에서만 사용함을 알려줍니다. 추가로 `Account`를 리턴해 주도록하여 보다 다양한 유닛테스트 operation을 할 수 있도록 합니다.

`TestCreateAccount()`역시 다음과 같이 수정해 줍니다.

```go
  func TestCreateAccount(t *testing.T) {
    createRandomAccount(t)
  }
```

이제 `createRandomAccount()`를 사용하여 `TestGetAccount()` 작성을 진행해 보겠습니다.

```go
  func TestGetAccount(t *testing.T) {
    account1 := createRandomAccount(t) // #1
    account2, err := testQueries.GetAccount(context.Background(), account1.ID) // #2
    require.NoError(t, err) // #3
    require.NotEmpty(t, account2) // #4

    require.Equal(t, account1.ID, account2.ID) // #5
    require.Equal(t, account1.Owner, account2.Owner) // #6
    require.Equal(t, account1.Balance, account2.Balance) // #7
    require.Equal(t, account1.Currency, account2.Currency) // #8
    require.WithinDuration(t, account1.CreatedAt, account2.CreatedAt, time.Second) // #9
}  
```

1. `createRandomAccount()`를 사용하여 `TestGetAccount()`에서 사용할 `account`를 만들어 줍니다. 이를 `account1` 변수에 저장합니다.
1. `account2` 변수에 `GetAccount()`를 사용하여 리턴받은 `account`를 저장합니다. 이때 우리는 이미 1번에서 만들어 놓은 `account.ID`를 사용하여 `GET`을 진행합니다.
1. `err`가 없는지 확인합니다.
1. `account2`값이 비어있지 않은지 확인합니다.
1. `account1.ID`와 `account2.ID`가 동일한지 확인합니다.
1. `account1.Owner`와 `account2.Owner`가 동일한지 확인합니다.
1. `account1.Balance`와 `account2.Balance`가 동일한지 확인합니다.
1. `account1.Crrency`와 `account2.Currency`가 동일한지 확인합니다.
1. `WithinDuration()`을 사용하여 `account1.CreatedAt` 값이 `account2.CreatedAt`값과 차이가 `time.Second` 즉 1초 내에 이루어 졌는지 확인합니다.

#### UpdateAccount 유닛 테스트

비슷한 방법으로 작성해 줍니다.

```go
  func TestUpdateAccount(t *testing.T) {
    account1 := createRandomAccount(t)

    // #1
    arg := UpdateAccountParams{
      ID:      account1.ID,
      Balance: util.RandomMoney(),
    }

    account2, err := testQueries.UpdateAccount(context.Background(), arg)
    require.NoError(t, err)
    require.NotEmpty(t, account2)

    require.Equal(t, account1.ID, account2.ID)
    require.Equal(t, account1.Owner, account2.Owner)
    require.Equal(t, arg.Balance, account2.Balance) //#2
    require.Equal(t, account1.Currency, account2.Currency)
    require.WithinDuration(t, account1.CreatedAt, account2.CreatedAt, time.Second)
}
```

1. `UpdateAccount()`를 하기 위해 update를 진행할 `UpdateAccountParams`를 적어줍니다.
1. 업데이트 될 `arg.Balance`가 `account2.Balance`에 잘 반영이 되었는지 확인해 줍니다.

#### DeleteAccount 유닛 테스트

```go
  func TestDeleteAccount(t *testing.T) {
    account1 := createRandomAccount(t)
    err := testQueries.DeleteAccount(context.Background(), account1.ID)
    require.NoError(t, err) // #1

    account2, err := testQueries.GetAccount(context.Background(), account1.ID) // #2
    require.Error(t, err)  // #3
    require.EqualError(t, err, sql.ErrNoRows.Error()) // #4
    require.Empty(t, account2) // #5
}  
```

1. `DeleteAccount()` 진행이 정상적으로 이루어졌고 `err`가 없는지 확인합니다.
1. 1번에서 삭제된 `account.ID`로 `GetAccount()`를 진행해 봅니다.
1. 에러가 나는지 확인합니다.
1. 에러의 종류가 `sql,ErrNoRows.Error()`가 맞는지 확인합니다.
1. `GetAccount`에 실패했기 떄문에 `account2`가 비어있는지 확인합니다.

#### ListAccounts 유닛 테스트

```go
  func TestListAccounts(t *testing.T) {
    // #1
    for i := 0; i < 10; i++ {
      createRandomAccount(t)
    }

    // #2
    arg := ListAccountsParams{
      Limit:  5,
      Offset: 5,
    }

    // #3
    accounts, err := testQueries.ListAccounts(context.Background(), arg)
    require.NoError(t, err)
    require.Len(t, accounts, 5)

    // #4
    for _, account := range accounts {
      require.NotEmpty(t, account)
    }
}
```

1. `ListAccount()`를 테스트하기 위해 1개 이상의 `account`가 필요합니다. 이 부분에 저희는 10개의 `account`를 `for-loop`을 사용하여 만들어 줍니다.
1. 테스트를 진행할 `ListAccountsParams`를 `arg`에 저장합니다. `Limit: 5`는 리턴값을 5개로 제한한다는 조건입니다. `Offset: 5`은 순서대로 처음 5개를 건너뛰라는 조건입니다.
1. 1번에서 총 10개의 `account`를 만들고 2번에서 5개의 `account` 리스트만 돌려달라는 조건으로 `ListAccounts()`를 실행시킵니다. 결과를 확인하기 위해 첫번째로 `err`가 없는지 확인합니다. 두번째로 리턴받은 `account` 리스트 갯수가 5개임을 확인합니다.
1. 리턴받은 `account`리스트가 비어있지 않은지 확인 합니다.

#### `entry_test.go`

```go
package db

import (
  "context"
  "testing"
  "time"

  "github.com/mannerism/simplebank/util"
  "github.com/stretchr/testify/require"
)

func createRandomEntry(t *testing.T, accountId int64) Entry {
  arg := CreateEntryParams{
    AccountID: accountId,
    Amount:    util.RandomMoney(),
  }

  entry, err := testQueries.CreateEntry(context.Background(), arg)
  require.NoError(t, err)
  require.NotEmpty(t, entry)

  require.Equal(t, arg.AccountID, entry.AccountID)
  require.Equal(t, arg.Amount, entry.Amount)

  require.NotZero(t, entry.ID)
  require.NotZero(t, entry.CreatedAt)

  return entry
}

func TestCreateEntry(t *testing.T) {
  account := createRandomAccount(t)
  createRandomEntry(t, account.ID)
}

func TestGetEntry(t *testing.T) {
  account := createRandomAccount(t)
  entry1 := createRandomEntry(t, account.ID)
  entry2, err := testQueries.GetEntry(context.Background(), entry1.ID)

  require.NoError(t, err)
  require.NotEmpty(t, entry2)

  require.Equal(t, entry1.ID, entry2.ID)
  require.Equal(t, entry1.AccountID, entry2.AccountID)
  require.Equal(t, entry1.Amount, entry2.Amount)
  require.WithinDuration(t, entry1.CreatedAt, entry2.CreatedAt, time.Second)
}

func TestListEntries(t *testing.T) {
  account := createRandomAccount(t)
  // Creating 10 random entries before testing `ListEntries`
  for i := 0; i < 10; i++ {
    createRandomEntry(t, account.ID)
  }

  arg := ListEntriesParams{
    AccountID: account.ID,
    Limit:     5,
    Offset:    5,
  }

  entries, err := testQueries.ListEntries(context.Background(), arg)

  require.NoError(t, err)
  require.NotEmpty(t, entries)
  require.Len(t, entries, 5)

  for _, entry := range entries {
    require.NotEmpty(t, entry)
  }
}
```

#### `transfer_test.go`

```go
package db

import (
  "context"
  "testing"
  "time"

  "github.com/mannerism/simplebank/util"
  "github.com/stretchr/testify/require"
)

func createRandomTransfer(t *testing.T, account1 Account, account2 Account) Transfer {
  arg := CreateTransferParams{
    FromAccountID: account1.ID,
    ToAccountID:   account2.ID,
    Amount:        util.RandomMoney(),
  }

  transfer, err := testQueries.CreateTransfer(context.Background(), arg)

  require.NotEmpty(t, account1)
  require.NotEmpty(t, account2)

  require.NoError(t, err)
  require.NotEmpty(t, transfer)

  require.Equal(t, arg.FromAccountID, transfer.FromAccountID)
  require.Equal(t, arg.ToAccountID, transfer.ToAccountID)
  require.Equal(t, arg.Amount, transfer.Amount)
  require.NotEmpty(t, transfer.ID)
  require.NotEmpty(t, transfer.CreatedAt)

  return transfer
}

func TestCreateTransfer(t *testing.T) {
  account1 := createRandomAccount(t)
  account2 := createRandomAccount(t)
  createRandomTransfer(t, account1, account2)
}

func TestGetTransfer(t *testing.T) {
  account1 := createRandomAccount(t)
  account2 := createRandomAccount(t)
  transfer1 := createRandomTransfer(t, account1, account2)
  transfer2, err := testQueries.GetTransfer(context.Background(), transfer1.ID)

  require.NoError(t, err)
  require.NotEmpty(t, transfer2)

  require.Equal(t, transfer1.ID, transfer2.ID)
  require.Equal(t, transfer1.FromAccountID, transfer2.FromAccountID)
  require.Equal(t, transfer1.ToAccountID, transfer2.ToAccountID)
  require.Equal(t, transfer1.Amount, transfer2.Amount)
  require.WithinDuration(t, transfer1.CreatedAt, transfer2.CreatedAt, time.Second)
}

func TestListTransfer(t *testing.T) {
  account1 := createRandomAccount(t)
  account2 := createRandomAccount(t)

  for i := 0; i < 10; i++ {
    createRandomTransfer(t, account1, account2)
  }

  arg := ListTransfersParams{
    FromAccountID: account1.ID,
    ToAccountID:   account2.ID,
    Limit:         5,
    Offset:        5,
  }

  transfers, err := testQueries.ListTransfers(context.Background(), arg)

  require.NoError(t, err)
  require.NotEmpty(t, transfers)
  require.Len(t, transfers, 5)

  for _, transfer := range transfers {
    require.NotEmpty(t, transfer)
  }
}
```

끝

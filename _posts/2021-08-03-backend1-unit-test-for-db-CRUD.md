---
title:  "백엔드 시리즈 5. Golang Unit Test 작성하기"
date:   2021-08-03 12:27:19 +0900
permalink: ":categories/backend/:title"
---

> 원문 출처: [Techschool](https://www.youtube.com/watch?v=rx6CPDK_5mU&list=PLy_6D98if3ULEtXtNSY_2qN21VCKgoQAE&ab_channel=TECHSCHOOL "Tech School")

### 백엔드 시리즈 5. Golang Unit Test 작성하기 (미완성)

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

#### lib/pq 디펜던시 설치하기

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

#### 본격적인 유닛 테스트 작성

이제 세팅은 완료가 되었습니다. 다시 `account_test.go`파일로 돌아가서 첫번째 유닛테스트를 작성해 보도록 하겠습니다.

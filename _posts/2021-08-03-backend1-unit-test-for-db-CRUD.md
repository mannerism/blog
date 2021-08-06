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

      #1
      func TestCreateAccount(t *testing.T) {

      }

    ```

    1. `go`의 유닛 테스트 function signature는 `Test`로 시작해야 합니다. 그리고 `testing.T` 객체를 인풋으로 받습니다. 이 `T` 객체를 통해 테스트의 상태를 관리할 수 있습니다.

`account.sql.go` 파일로 가서 테스트를 진행할 function을 살펴 보도록 합시다.
  
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
    
    var testQueries *Queries // #1

    const (
      dbDriver = "postgres"
      dbSource = "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable"
    )

    func TestMain(m *testing.M) {
      conn, err := sql.Open(dbDriver, dbSource)
      if err != nil {
        log.Fatal("cannot connect to db", err)
      }
      
      testQueries = New(conn)

      m.Run()
    }
  ```

1. `testQueries`라는 전역 변수를 정의해줍니다.
1. `TestMain` function은 1개의 golang 패키지의 유닛테스트 시작 포인트입니다.
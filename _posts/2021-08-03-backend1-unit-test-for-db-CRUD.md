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

      #1
      func TestCreateAccount(t *testing.T) {

      }

    ```

    1. `go`의 유닛 테스트 function signature는 `Test`로 시작해야 합니다. 그리고 `testing.T` 객체를 인풋으로 받습니다. 이 `T` 객체를 통해 테스트의 상태를 관리할 수 있습니다.
1. `account.sql.go` 파일로 가서 테스트를 진행할 function을 살펴 보도록 합시다.
  
    ```go
      func (q *Queries) UpdateAccount(ctx context.Context, arg UpdateAccountParams) (Account, error) {
        row := q.db.QueryRowContext(ctx, updateAccount, arg.ID, arg.Balance)
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

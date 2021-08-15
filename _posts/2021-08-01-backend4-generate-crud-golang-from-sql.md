---
title:  "백엔드 시리즈 4. CRUD golang 코드 작성하기"
date:   2021-08-01 12:27:19 +0900
permalink: ":categories/backend/:title"
---

> 원문 출처: [Techschool](https://www.youtube.com/watch?v=rx6CPDK_5mU&list=PLy_6D98if3ULEtXtNSY_2qN21VCKgoQAE&ab_channel=TECHSCHOOL "Tech School")

### 백엔드 시리즈 4. CRUD golang 코드 작성하기

#### CRUD 란?

서버에서 REST API를 구축할 시 고려해야하는 4가지 기본 기능입니다.

1. **C**: Create, DB에 새로운 값을 넣는 기능을 수행합니다.
1. **R**: Read, DB에 기록된 값을 불러오는 기능을 수행합니다.
1. **U**: Update, DB에 기록된 값을 업데이트하는 기능을 수행합니다.
1. **D**: Delete, DB에 기록된 값을 지우는 기능을 수행합니다.

#### Golang으로 CRUD 구현하는 방법

1. `Database/SQL` [Golang 기본 라이브러리](https://pkg.go.dev/database/sql#DB.QueryContext)
    * `QueryRowContext()`를 사용하여 raw `SQL` query 스크립트를 작성할 수 있습니다.
    * 장점: 매우 빠르며, 코드 작성이 직관적입니다.
    * 단점: 수동으로 SQL fields를 변수와 매핑을 해 주어야 합니다.상당히 지루한 프로세스이며 `runtime`까지 실수를 확인할 수 없기 때문에 다소 귀찮아 질 수 있습니다.
1. [GORM](https://gorm.io/index.html): Golang's high-level, object-relational-mapping 라이브러리
    * 장점: CRUD 가 이미 구현되어 있으며, 작성해야 하는 코드가 매우 적음
    * 단점1: `GORM`에서 정해준대로 query를 작성해야함
    * 단점2: 복잡한 query를 진행하는 경우 직접 query를 작성하는것 보다 5-6배 느릴 수 있음
1. [SQLX](https://github.com/jmoiron/sqlx): `1`번과 `2`번의 중간 단계의 라이브러리
    * 장점: 스탠다드 라이브러리 만큼 빠르고 간편함
    * 단점1: 그래도 작성해야 하는 코드 양이 제법 많음
    * 단점2: 실수를 `runtime`까지 모름
1. [SQLC](https://github.com/kyleconroy/sqlc): 우리의 초이스
    * 장점1: 빠르기와 간편하기가 대단함
    * 장점2: 가장 훌륭한 기능은, query를 작성하면 알아서 `Database/SQL` 스탠다드 라이브러리를 사용하여 `golang CRUD`코드를 생성해 줌. 따라서 `runtime`전에 에러 발견 가능
    * 단점: `2020.7.13`일 기준 `Postgres`밖에 지원을 안함

#### SQLC 설치

1. [SQLC](https://github.com/kyleconroy/sqlc) 로 이동합니다.
1. `Installation` 페이지로 이동합니다.
1. `macos`의 경우 터미널을 열고 `brew install sqlc` 입력합니다. 이 시리즈의 경우 버전 `1.8.0`을 사용합니다.
1. 터미널에서 `simplebank` 프로젝트 폴더로 이동하여 `sqlc init`을 입력합니다. 그러면 `sqlc.yaml` config파일이 생성됩니다.
1. [Config](https://docs.sqlc.dev/en/latest/reference/config.html) 페이지로 이동하여 내용 복사한 후 `VScode`를 열고 `sqlc.yaml`에 붙여넣기 해 줍니다.
  
    ```yaml
      version: "1"
      packages: # 1
        - name: "db" #2
          path: "./db/sqlc" #3
          queries: "./db/query " #4
          schema: "./db/migration" #5
          engine: "postgresql" #6
          emit_prepared_queries: false #7
          emit_interface: false #8
          emit_exact_table_names: false #9
          emit_empty_slices: false #19
          emit_json_tags: true #11
          json_tags_case_style: "snake" #12
    ```

    설명:

      1. `sqlc`에 1개 이상의 `Go package`를 생성하도록 할 수 있습니다. 이번 시리즈는 1개의 `Go package`만 생성하도록 합니다.
      1. 생성 될 `Go package`의 이름을 정의합니다.
      1. 코드 파일이 저장 될 `path`를 지정해 줍니다. `db` 폴더안에 `sqlc`폴더를 하나 만들어주고 위치로 지정해 줍니다.
      1. `sqlc`에서 사용할 `SQL queries`파일이 저장된 위치를 알려줍니다. `db` 폴어안에 `query`폴더를 하나 만들어주고 위치로 지정해 줍니다.
      1. `schema` 또는 `migration`파일이 저장된 위치를 알려줍니다.
      1. 사용하는 `DB Engine`을 적어줍니다.
      1. `prepared queries`를 지원하도록 합니다. 아직은 퍼포먼스 최적화 단계전이기 때문에 `false`로 해 줍니다.
      1. 생성된 패키지 안에 `Querier` 인터페이스를 제공하도록 합니다. `mock`을 사용하여 `higher-level function`을 테스트할 때 도움이 될 수 있지만 지금은 `false`로 해 줍니다.
      1. 생성된 `struct`이름을 테이블 이름과 일치하도록 합니다. `false`인 경우 복수형 이름을 단수형으로 바꿔줍니다.
      1. `:many` queries를 사용하여 전달받은 `slices`들이 `nil`이 아닌 비어있는 채로 생성됩니다.
      1. 생성된 `structs`에 `JSON tags`를 추가하도록 합니다.
      1. `JSON tags`의 스타일을 정의합니다.
1. `Makefile`에 새로운 `sqlc`커멘드를 추가해 줍니다.

    ```Makefile
      sqlc: 
      sqlc generate

      .PHONY: postgres createdb dropdb opendb migrateup migratedown sqlc
    ```

#### SQL Query 작성 (Create)

1. `./db/query`폴더 안에 새로운 `account.sql`파일을 생성합니다.
1. [sqlc getting started](https://docs.sqlc.dev/en/latest/tutorials/getting-started.html)로 이동하여 `CreateAuthor` 템플릿을 복사, 붙여넣기 합니다.

    ```sql
      -- name: CreateAuthor :one #1
      INSERT INTO accounts (
        name, bio
      ) VALUES (
        $1, $2
      )
      RETURNING *;
    ```

    설명:
      1. 코멘트는 `sqlc`에게 어떤 `signature`로 Golang function을 만들지 알려줍니다. 위 코멘트의 경우 `CreateAccount()`라는 function을 만들것이고 `:one` 이란 키워드로 1개의 `account object`를 리턴할 것입니다.

1. [시리즈1](https://mannerism.github.io/blog/develop/backend/backend1-DB-Schema-and-SQL-code)에서 생성한 `accounts` 테이블을 보고 템플릿을 수정해 줍니다.
    `accounts` 테이블:

    ```sql
      CREATE TABLE "accounts" (
        "id" bigserial PRIMARY KEY,
        "owner" varchar NOT NULL,
        "balance" bigint NOT NULL,
        "currency" varchar NOT NULL,
        "created_at" timestamptz NOT NULL DEFAULT (now())
      );
    ```

    템플릿 수정본:

    ```sql
      -- name: CreateAuthor :one
      INSERT INTO accounts (
        owner, #1
        balance, #1
        currency #1
      ) VALUES (
        $1, $2, $3 #2
      )
      RETURNING *; #3
    ```

    설명:

      1. `accounts` 테이블을 보면 `id`와 `created_at`은 자동으로 부여되는 값이기 때문에, `owner`, `balance`, `currency`에 대한 값만 제공해 주면 됩니다.
      2. 3개의 column이 있기 때문에 `$1`, `$2`, `$3` 이렇게 세 개의 `argument`를 입력 받을것이라고 알려줍니다.
      3. `RETURNING *`를 사용하여 `postgresql`에 새로운 값이 `DB`에 쓰여질 때 모든 column값을 리턴하라고 알려줍니다. 이 부분이 매우 중요한 이유는 `account`가 새로 생성될 때마다 `id`를 클라이언트에게 제공해 주길 원하기 때문입니다.

1. 이제 터미널을 열고 `make sqlc` 커멘드를 입력합니다.
    입력:

      ```zsh
        make sqlc
      ```

    출력:

      ```zsh
        sqlc generate
      ```

    `VScode`로 돌아가보면 `sqlc` 폴더 안에 `go` 코드 파일들(`accounts.sql.go`, `db.go`, `models.go`)이 자동으로 생성된걸 확인할 수 있습니다.

#### sqlc를 사용하여 생성된 파일 살펴보기

이제 `sqlc`를 사용하여 생성된 `go`파일들을 하나씩 살펴보겠습니다.

##### models.go

```go
  // Code generated by sqlc. DO NOT EDIT.
  package db

  import (
    "time"
  )

  type Account struct {
    ID        int64     `json:"id"`
    Owner     string    `json:"owner"`
    Balance   int64     `json:"balance"`
    Currency  string    `json:"currency"`
    CreatedAt time.Time `json:"createdAt"`
  }

  type Entry struct {
    ID        int64 `json:"id"`
    AccountID int64 `json:"accountID"`
    // can be positive or negative
    Amount    int64     `json:"amount"`
    CreatedAt time.Time `json:"createdAt"`
  }

  type Transfer struct {
    ID            int64 `json:"id"`
    FromAccountID int64 `json:"fromAccountID"`
    ToAccountID   int64 `json:"toAccountID"`
    // must be positive
    Amount    int64     `json:"amount"`
    CreatedAt time.Time `json:"createdAt"`
  }
```

이 파일은 `Account`, `Entry`, `Transfer`, 총 3개의 모델에 대한 `struct`를 정의합니다. 앞서 `sqlc.yaml`의 `emit_json_tags`를 `true`로 지정한대로 `JSON tags` 정보를 포함하고 있습니다. 또한 시리즈1 `DB schema`에 추가한 코멘트들까지 반영된걸 확인할 수 있습니다.

##### db.go

```go
  package db

  import (
    "context"
    "database/sql"
  )

  #1
  type DBTX interface {
    ExecContext(context.Context, string, ...interface{}) (sql.Result, error)
    PrepareContext(context.Context, string) (*sql.Stmt, error)
    QueryContext(context.Context, string, ...interface{}) (*sql.Rows, error)
    QueryRowContext(context.Context, string, ...interface{}) *sql.Row
  }

  #2
  func New(db DBTX) *Queries {
    return &Queries{db: db}
  }

  type Queries struct {
    db DBTX
  }

  #3
  func (q *Queries) WithTx(tx *sql.Tx) *Queries {
    return &Queries{
      db: tx,
    }
  }
```

1. `DBTX` 인터페이스를 보여줍니다. 이 인터페이스에선 `sql.DB`와 `sql.Tx` 객체가 가지고 있는 일반적인 4개의 메쏘드를 정의하고 있습니다. 이 인터페이스에서 정의된 메쏘드를 가지고 `db`나 `transaction`를 사용하여 `query`를 진행할 수 있습니다.
1. `New()`는 `DBTX`를 input으로 받으며, `Queries`객체를 리턴합니다. 따라서 우린 `sql.DB`나 `sql.Tx` 객체를 인풋으로 사용할 수 있습니다. 1개의 Query를 진행하던지, 1개의 transaction에 포함된 다수의 Query를 진행하던지 할 수 있습니다.
1. `WithTx()`을 사용해 `Queries` 인스턴스를 `transaction`과 연관 지을 수 있습니다.

##### account.sql.go

```go
  // Code generated by sqlc. DO NOT EDIT.
  // source: account.sql
  
  #1
  package db

  import (
    "context"
  )
  
  #2
  const createAccount = `-- name: CreateAccount :one
  INSERT INTO accounts (
    owner,
    balance,
    currency
  ) VALUES (
    $1, $2, $3
  )
  RETURNING id, owner, balance, currency, created_at
  `
  #3
  type CreateAccountParams struct {
    Owner    string `json:"owner"`
    Balance  int64  `json:"balance"`
    Currency string `json:"currency"`
  }

  #4
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

1. `sqlc.yaml`에서 정의한대로 package명은 `db`로 되어있습니다.
1. 본 시리즈 `SQL Query 작성`에서 작성한 sql query와 내용이 전부 같습니다만 return 부분에 `RETURNING *`를 모든 Column의 이름으로 바꾸어 놓았습니다. 즉 추가로 작성되는 모든 Column의 정보를 다시 클라이언트에게 내 보내는 역할을 수행하는 것입니다.
1. `CreateAccountParams`는 새로운 `account`를 만들 시 클라이언트에서 값을 입력할 수 있는 세 개의 column을 포함하고 있습니다.
1. `CreateAccount()`는 `q *Queries`에서 보여지는대로 `Queries` 객체의 메쏘드로 정의 되어있습니다. 이 메쏘드는 `context.Context`와 `CreateAccountParams` 타입을 input으로 받습니다. 그리고 `Account` 모델 객체와 `error`를 리턴합니다.

#### Go module initialize 하기

>자신의 컴퓨터에 `go`가 깔려있지 않다면 터미널에 `brew install go`를 진행하여 설치를 해 줍니다. [go official website](https://golang.org/doc/install)

`VScode`에서 `account.sql.go`파일에 빨간줄로 에러가 떠있을 수 있습니다. 이는 우리가 아직 module을 initialize하지 않았기 때문입니다. 터미널을 열고 module을 initialize해 줍니다.

입력:

  ```zsh
    go mod init go mod init github.com/mannerism/simplebank (자신의 github프로젝트 링크)
  ```

출력:

  ```zsh
    go: creating new go.mod: module github.com/mannerism/simplebank
    go: to add module requirements and sums:
    go mod tidy
  ```

에러가 사라진걸 확인할 수 있습니다.

#### SQL Query 작성 (Read)

1. [sqlc getting started](https://docs.sqlc.dev/en/latest/tutorials/getting-started.html)로 이동하여 `GetAuthor`과 `ListAuthors` 템플릿을 `./db/account.sql`파일에 복사 붙여넣기 합니다.

    ```sql
      -- name: GetAuthor :one
      SELECT * FROM authors
      WHERE id = $1 LIMIT 1;

      -- name: ListAuthors :many
      SELECT * FROM authors
      ORDER BY name;
    ```

1. 우리 목적에 맞게 이름을 수정해 줍니다.

    ```sql
      #1
      -- name: GetAccount :one
      SELECT * FROM accounts
      WHERE id = $1 LIMIT 1;

      #2
      -- name: ListAccounts :many
      SELECT * FROM accounts
      ORDER BY id;
      LIMIT $1
      OFFSET $2;
    ```

    1. `sqlc`를 통해 메쏘드가 생성될 function signature를 코멘트에 작성해 줍니다. 이 경우 메쏘드명은 `GetAccount`가 됩니다. 코멘트 뒤에 추가된 `:one`으로 1개의 값이 리턴될 것이라고 알려줍니다. `accounts`테이블에서 `SELECT`를 진행하고 `id` 값을 인풋으로 받아서 `LIMIT`을 사용하여 1개의 값만 찾도록 지정해 줍니다.
    1. 코멘트 메쏘드명을 `ListAccounts`로 수정해 줍니다. 코멘트 뒤에 `:many`를 추가하여 다수의 값을 리턴할 것을 알려줍니다. 그리고 1번과 동일하게 `accounts` 테이블에서 `SELECT`하도록 하며, `ORDER BY` `id`를 사용하여 `id`순으로 값을 리턴하도록 해 줍니다. 리턴받는 값의 수를 인풋받아 `LIMIT`에 전달해주며 `OFFSET` 값도 인풋으로 제공할 수 있도록 해 줍니다.

1. `account.sql`에 입력한 Queries를 저장하고 터미널을 열어서 `make sqlc`를 진행해 줍니다.
1. `account.sql.go` 파일에 `GetAccount()`와 `ListAccounts()` 메쏘드가 추가된 것을 확인할 수 있습니다. 보다 자세한 `golang` syntax는 차근차근 알아가 보도록 해 봅시다.

#### SQL Query 작성 (Update)

1. [sqlc update](https://docs.sqlc.dev/en/latest/howto/update.html?highlight=update)로 이동하여 `UpdateAuthor`템플릿을 `./db/account.sql`파일에 복사 붙여넣기 합니다.

    ```sql
      -- name: UpdateAuthor :exec
      UPDATE authors SET bio = $2
      WHERE id = $1;
    ```

1. Query를 수정해 줍니다.

    ```sql
      -- name: UpdateAccount :one #1
      UPDATE accounts
      SET balance = $2 #2
      WHERE id = $1
      RETURNING *; #1
    ```
  
    1. 별도 값을 리턴하지 않는 `Update`의 경우 `:exec`키워드를 추가합니다. 하지만 `Update`를 하는 경우에도 업데이트가 되는 `account`의 정보를 알면 도움이 되므로 여기서도 `:one` 키워드를 사용하겠습니다. 그리고 마지막에 `RETURNING *`를 추가해 줍니다.
    1. account의 `owner`와 `currency`는 클라이언트에서 업데이를 할 필요가 없기 때문에 `balance` 값만 인풋으로 받아 특정 `id`를 가지는 `account`의 `balance`를 업데이트 해 줍니다.

1. `account.sql`에 입력한 Queries를 저장하고 터미널을 열어서 `make sqlc`를 진행해 줍니다.
1. `account.sql.go` 파일에 `UpdateAccount()` 메쏘드가 추가된 것을 확인할 수 있습니다.

#### SQL Query 작성 (Delete)

1. [sqlc delete](https://docs.sqlc.dev/en/latest/howto/delete.html?highlight=Delete)로 이동하여 `DeleteAuthor`템플릿을 `./db/account.sql`파일에 복사 붙여넣기 합니다.

    ```sql
      -- name: DeleteAuthor :exec
      DELETE FROM authors WHERE id = $1;
    ```

1. Query를 수정해 줍니다.

    ```sql
      -- name: DeleteAccount :exec
      DELETE FROM accounts 
      WHERE id = $1;
    ```

1. `account.sql`에 입력한 Queries를 저장하고 터미널을 열어서 `make sqlc`를 진행해 줍니다.
1. `account.sql.go` 파일에 `DeleteAccount()` 메쏘드가 추가된 것을 확인할 수 있습니다.

비슷한 방법으로 `entries`와 `transfers`도 진행해 줍니다.

#### `Entries` SQL Query 작성(Create, Read)

1. `./db/query` 에 `entry.sql`파일 생성
1. SQL Query입력

    ```sql
      -- name: CreateEntry :one
      INSERT INTO entries (
        account_id,
        amount
      ) VALUES (
        $1, $2
      )
      RETURNING *;

      -- name: GetEntry :one
      SELECT * FROM entries
      WHERE id = $1 
      LIMIT 1;

      -- name: ListEntries :many
      SELECT * FROM entries
      WHERE account_id = $1
      ORDER BY id
      LIMIT $2
      OFFSET $3;
    ```

1. `make sqlc` 커멘드 진행
1. `./db/sqlc`에 `entry.sql.go`파일 생성 확인

#### `Transfers` SQL Query 작성(Create, Read)

1. `./db/query` 에 `transfer.sql`파일 생성
1. SQL Query입력

    ```sql
      -- name: CreateTransfer :one
      INSERT INTO transfers (
        from_account_id,
        to_account_id,
        amount
      ) VALUES (
        $1, $2, $3
      )
      RETURNING *;

      -- name: GetTransfer :one
      SELECT * FROM transfers
      WHERE id = $1 LIMIT 1;

      -- name: ListTransfers :many
      SELECT * FROM transfers
      WHERE
        from_account_id = $1 OR
        to_account_id = $2
      ORDER BY id
      LIMIT $3
      OFFSET $4;
    ```

1. `make sqlc` 커멘드 진행
1. `./db/sqlc`에 `transfer.sql.go`파일 생성 확인

끝.

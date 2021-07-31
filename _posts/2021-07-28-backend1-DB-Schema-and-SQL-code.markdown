---
title:  "백엔드 시리즈 1. DB Schema와 SQL 코드"
date:   2021-07-28 21:07:19 +0900
permalink: ":categories/backend/:title"
---

> 원문 출처: [Techschool](https://www.youtube.com/watch?v=rx6CPDK_5mU&list=PLy_6D98if3ULEtXtNSY_2qN21VCKgoQAE&ab_channel=TECHSCHOOL "Tech School")

### 백엔드 시리즈 1. DB Schema 와 SQL 코드 만들기

이번 백엔드 시리즈에서는 `Simple Bank`라는 프로젝트를 만들어 보겠습니다. `Simple Bank`는 다음과 같은 기능을 수행합니다:

1. 은행 계좌를 만들고 관리합니다.
1. 계좌 변동 사항을 기록합니다.
1. 계좌 간 이체를 진행할 수 있습니다.

이 시리즈에서 사용할 기술 스택:

1. PostgreSQL
1. Docker
1. Go

먼저 [DBdiagram](https://dbdiagram.io)를 클릭하여 사이트로 이동합니다. 그리고 우측 상단에 있는 `Go To App`을 클릭합니다. 그러면 좌측에 DB Struct를 작성할 수 있고, 이 작성된 코드를 기반으로 우측에 다이어그램이 자동으로 생성됩니다. DB Struct 작성이 완료 되면 상단에 있는 `Export` 메뉴를 사용하여 SQL Code를 생성할 수 있습니다.

이제 좌측 상단에 `Untitled Diagram`로 되어있는 이름을 `Simple bank`로 수정해 줍니다.

본격적인 DB Struct 작성을 시작해 봅시다.

---

#### DB Struct 작성

1. 계좌 정보를 담게 되는 `Account`를 만듭니다.

    ```sql
    Table accounts as A { // #1
      id bigserial [pk] // #2
      owner varchar [not null] // #3
      balance bigint [not null] // #4
      currency varchar [not null]
      created_at timestamptz [not null, default: `now()`] // #5
    }
    ```

    설명:

    1. `as` 키워드를 사용하여 `account`의 `Alias`를 `A`로 정의합니다.
    1. 모든 계좌마다 유니크한 ID를 부여하기 위해서 사용하는 type으로, auto-increment, 즉 자동으로 값이 올라가는 type을 사용합니다. PostgreSQL에서는 8-byte (64-bit, 1 에서 9223372036854775807까지) 사이즈의 `bigserial`이란 type을 사용할 수 있습니다. 자동으로 하나씩 올라가는 큰 숫자라고 이해하시면 됩니다. 그리고 `[pk]`를 사용하여 이 필드가 `Primary Key`라는 것을 알려줍니다.
    1. `varchar`을 사용하여 계좌 이름을 저장할 수 있게 해 줍니다.
    1. 8-byte (64-bit, -9223372036854775808 에서 +9223372036854775807 까지) 사이즈의 `bigint` 를 사용합니다. 실제로는 소숫점을 사용하는 경우가 많기 때문에 `decimal` type을 사용할 수도 있습니다.
    1. 계좌가 언제 생성되었는지 여부를 확인하기 위해 `timestamp` type을 사용합니다. 뒤에 `tz`를 추가하여 `time zone`을 트랙할 수 있는 type인 `timestamptz`를 사용합니다. `[default:`now( )`]`를 사용하여 db에서 자동으로 계좌 생성 시간을 입력할 수 있도록 해 줍니다.
    1. `id`필드를 제외한 모든 필드에 `[not null]`을 추가하여 필드가 `null`이 될 수 없음을 명확히 표시해 줍니다.

2. 계좌에 남아있는 금액의 정보를 기록하고 추적할 수 있는 `entries`를 만듭니다.

    ```sql
    Table entries {
      id bigserial [pk]
      account_id bigint [not null, ref: > A.id] // #1
      amount bigint [not null, note: 'can be positive or negative.'] //#2
      created_at timestamptz [not null, default: `now()`]
    }
    ```

    1. `account_id`라는 외부 키를 정의해 줍니다. `[ref: > A.id]`를 사용하여 이 값은 1번에서 정의한 `account`의 `id`를 참조한다는 것을 알려 줍니다. 이 필드를 통해서 1개의 계정에서 다수의 `entries`를 작성할 수 있게 됩니다.
    1. `note`를 추가하여 이 필드를 사용시 알아야하는 추가 정보를 입력해 줍니다.

3. 계좌간 이체내역 정보를 담는 `transfers`를 만듭니다.

    ```sql
    Table transfers {
      id bigserial [pk]
      from_account_id bigint [not null, ref: > A.id]
      to_account_id bigint [not null, ref: > A.id]
      amount bigint [not null, note: 'must be positive.'] // #1
      created_at timestamptz [not null, default: `now()`]
    }
    ```

    1. `entries`와 다르게 `trasfers`의 amount는 `-` 값을 가질 수 없습니다.

4. 추후 필드 검색을 용이하게 하기위해 `Indexes`를 추가해 줍니다.

    ```sql
    Table accounts as A {
      ...
      Indexes {
        owner
      }
    }

    Table entries {
      ...
      Indexes {
        account_id
      }
    }

    Table transfers {
    ...
      Indexes {
        from_account_id
        to_account_id
        (from_account_id, to_account_id)
      }
    }
    ```

완성된 DB Schema:
![DB Schema](https://raw.githubusercontent.com/mannerism/blog/gh-pages/_posts/images/DB-Schema.png "DB Schema")
*Figure 1. Simple Bank DB Schema*

---

#### PostgreSQL 코드로 Export 하기

`dbdiagram` 웹사이트 중간 상단 `Export` > `Export to PostgreSQL`를 클릭하여 작성된 DB Schema를 SQL code 파일로 다운받아 줍니다.

다운로드 된 PostgreSQL 코드:

```sql
CREATE TABLE "accounts" (
  "id" bigserial PRIMARY KEY,
  "owner" varchar NOT NULL,
  "balance" bigint NOT NULL,
  "currency" varchar NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

CREATE TABLE "entries" (
  "id" bigserial PRIMARY KEY,
  "account_id" bigint NOT NULL,
  "amount" bigint NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

CREATE TABLE "transfers" (
  "id" bigserial PRIMARY KEY,
  "from_account_id" bigint NOT NULL,
  "to_account_id" bigint NOT NULL,
  "amount" bigint NOT NULL,
  "created_at" timestamptz NOT NULL DEFAULT (now())
);

ALTER TABLE "entries" ADD FOREIGN KEY ("account_id") REFERENCES "accounts" ("id");

ALTER TABLE "transfers" ADD FOREIGN KEY ("from_account_id") REFERENCES "accounts" ("id");

ALTER TABLE "transfers" ADD FOREIGN KEY ("to_account_id") REFERENCES "accounts" ("id");

CREATE INDEX ON "accounts" ("owner");

CREATE INDEX ON "entries" ("account_id");

CREATE INDEX ON "transfers" ("from_account_id");

CREATE INDEX ON "transfers" ("to_account_id");

CREATE INDEX ON "transfers" ("from_account_id", "to_account_id");

COMMENT ON COLUMN "entries"."amount" IS 'can be positive or negative';

COMMENT ON COLUMN "transfers"."amount" IS 'must be positive';
```

시리즈1 끝.
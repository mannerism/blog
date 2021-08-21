---
title:  "백엔드 시리즈 6. DB Transaction 작성하기"
date:   2021-08-03 12:27:19 +0900
permalink: ":categories/backend/:title"
---

> 원문 출처: [Techschool](https://www.youtube.com/watch?v=rx6CPDK_5mU&list=PLy_6D98if3ULEtXtNSY_2qN21VCKgoQAE&ab_channel=TECHSCHOOL "Tech School")

### 백엔드 시리즈 6. DB Transaction 작성하기(미완성)

이번 시리즈에서는 db transaction을 작성해 보겠습니다.

#### DB Transaction이란?

1개 또는 1개 이상의 데이터 베이스 작업의 묶음(Unit of work)을 말합니다.

예를 들어서 설명해 보겠습니다. `simple bank`프로젝트에서 우리는 `10 USD`를 `account1`에서 `account2`로 보내고자 하는경우 총 5개의 `db operation`이 필요합니다:

1. `10`의 값을 가진 `transfer` 레코드를 하나 만듭니다.
1. `account1`에서 돈이 빠져나가기 때문에 `-10`값을 갖는 `account1`의 `entry` 레코드를 만들어 줍니다.
1. `account2`에 돈이 들어오기 떄문에 `+10`값을 갖는 `account2`의 `entry`레코드를 만들어 줍니다.
1. `account1`의 `balance`에 `10`을 빼줍니다.
1. `account2`의 `balance`에 `10`을 더해줍니다.

이 `db operation`의 묶음, 즉 가장 작은 단위의 작업을 `transaction`이라고 합니다.

#### transaction이 필요한 이유

1. 시스템이 제대로 동작하지 않더라도 작업 수행에 있어 명확한 기준을 제시해 줍니다.
1. DB에 동시적으로 접근하는 다양한 프로그램간을 분리할 수 있습니다.

#### ACID 요소

위의 두 가지 목표를 이루기 위해 database transaction은 `ACID`요소를 충족시켜야 합니다.

- `A`: `Atomicity` 원자성. 작업의 실패와 성공의 명확한 분리라고 이해하시면 됩니다. 1개의 transaction에 포함돼 있는 다양한 `db operation`이 전부 성공을 하던지 아니면 1개의 `db operation`이라도 실패할 경우 모든 작업이 취소되고 작업을 실행하기 이전의 상태로 돌아가게되는것을 말합니다.

- `C`: `Consistency` 일관성. DB의 상태는 `transaction`이 완료된 후에 모두 유효해야 합니다. 좀 더 정확히 설명하자면, DB에 기록된 모든 데이터는 초기에 정해진 룰들 (constraints, cascades, triggers 등)에 항상 부합해야 합니다.

- `I`: `Isolation` 분리. 동시다발적으로 이루어지는 `transaction`들은 서로 영향을 주지않도록 분리 되어있어야 합니다. `transaction`간 분리는 다양한 레벨로 나뉘어 지는데 언제 특정 `transaction`이 일으킨 변화가 외부에 공개 될지는 추후 천천히 알아보도록 합시다.

- `D`: `Durability` 내구성. 성공적인 `transaction`을 통해 기록된 데이터는 시스템이 멈추더라고 계속 유지되어야 합니다.

#### SQL DB Transaction 작성 순서

1. `BEGIN` 문을 사용하여 `transaction`을 시작합니다.
1. `SQL Queries`를 작성해 줍니다.
1. 모든 작업이 성공이면 `COMMIT`문을 사용하여 영구적으로 기록합니다. 그러면 DB는 새로운 상태를 갖게 됩니다.
1. 만약에 query가 실패한 경우, `ROLLBACK`문을 사용하여 수행된 모든 queries를 취소하고 이전의 상태로 돌려줍니다.

#### DB Transaction을 `Go`로 작성하기

`store.go`라는 새로운 파일을 `db/sqlc`폴더에 만들어 줍니다. 그리고 `Store` struct를 작성해 줍니다.

```go
  type Store struct {

  }
```

이 `Store`는 db queries를 수행하는 모든 function을 담게 돕니다.

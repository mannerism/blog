---
title:  "백엔드 시리즈 3. DB 마이그레이션"
date:   2021-07-31 12:27:19 +0900
permalink: ":categories/backend/:title"
---

> 원문 출처: [Techschool](https://www.youtube.com/watch?v=rx6CPDK_5mU&list=PLy_6D98if3ULEtXtNSY_2qN21VCKgoQAE&ab_channel=TECHSCHOOL "Tech School")

### 백엔드 시리즈 3. DB 마이그레이션

이번 시리즈에는 DB 마이그레이션을 진행해 보겠습니다. 백엔드 개발을 하다보면 비지니스 로직이 바뀌고 DB가 바뀌어야 하는 경우가 종종있는데 이를 위한 강의라고 보시면 됩니다.

---

#### Golang migrate 라이브러리 설치

이번 시리즈의 DB migration을 위해서 `golang migrate` 라이브러리를 사용해 보겠습니다. [golang-migrate](https://github.com/golang-migrate/migrate) 깃허브에서 [CLI Documentation](https://github.com/golang-migrate/migrate/tree/master/cmd/migrate)에 설명된 대로 설치를 시작해 보겠습니다.

`homebrew` 패키지 매니저를 통한 설치 방법:

입력:

  ```zsh
    brew install golang-migrate
  ```

출력:

  ```zsh
    ...
    ==> Pouring golang-migrate--4.14.1.big_sur.bottle.tar.gz
    🍺  /usr/local/Cellar/golang-migrate/4.14.1: 5 files, 37.6MB
  ```

설치가 완료되면 제대로 설치가 되었는지 한번 확인해 봅시다.

입력:

  ```zsh
    migrate -version
  ```

출력:

  ```zsh
    v4.14.1
  ```

설치된 버전이 정상적으로 뜨는것을 보니 제대로 설치가 된걸 알 수 있습니다.

---

#### Simple Bank 프로젝트 폴더 생성

이제 본격적으로 본 시리즈의 메인 프로젝트 폴더를 생성하여 하나씩 만들어 나가보겠습니다. 프로젝트 폴더의 위치는 개인의 기호에 맞게 알아서 진행하면 됩니다. 배경화면에 둬도 되고 사용자 폴더안에 둬도 됩니다. 저같은 경우 `~/dev` 폴더를 만들어 두고 그 안에 모든 프로젝트 폴더를 정리해 두고 있습니다. 커멘드 라인을 사용하여 `simplebank` 폴더를 만들어 줍니다.

입력:

  ```zsh
    mkdir simplebank
  ```

추가로 `simplebank` 폴더 안에서 `db/migration` 폴더를 만들어 줍니다.

입력:

  ```zsh
    mkdir -p db/migration
  ```

---

#### 마이그레이션 파일 생성

`simplebank` 폴더에서 이전에 설치한 `migrate` 을 사용해 마이그레이션 파일을 새로 만들어 줍니다.

마이그레이션 파일 생성 커멘드 syntax
> migrate create [-ext E] [-dir D] [-seq] [-digits N] [-format] NAME

입력:

  ```zsh
    migrate create -ext sql -dir db/migration -seq init_schema
  ```

설명: `create` 커멘드를 사용하여 마이그레이션 파일을 생성해 주며, `extension`은 `sql`로 지정해 줍니다. 디렉토리는 이전에 만든 `db/migration`으로 지정해 줍니다. `-seq` 플래그를 사용하여 마이그레이션을 진행할때마다 순차적으로 버전이 하나씩 올라가도록 정해줍니다. 그리고 이번 마이그레이션의 이름으로 `init_schema`를 적어줍니다.

출력:

  ```zsh
    /Users/sukyu/dev/simplebank/db/migration/000001_init_schema.up.sql
    /Users/sukyu/dev/simplebank/db/migration/000001_init_schema.down.sql
  ```

두 개의 `up` `down` 마이그레이션 파일이 만들어졌습니다. 각 파일명 앞에 버전 `00001`이 붙어있는걸 확인할 수 있습니다. `up`과 `down`파일로 분리를 해서 진행하는 것이 가장 바람직한 DB 마이그레이션 방법입니다.

1. `00001_init_schema.up.sql` 스크립트는 `DB Schema`를 변경하는데 사용합니다. `up` 스크립트가 돌 때는 버전이 한개씩 올라가게 됩니다.
1. `00001_init_schema.down.sql` 스크립트는 `00001_init_schema.up.sql`에서 변경되 내용을 취소하고 다시 뒤로 돌리는데 사용합니다. `down` 스크립트가 돌 때는 버전이 하나씩 내려가게 됩니다.

이제 [백엔드 시리즈 1. DB Schema와 SQL코드](https://mannerism.github.io/blog/develop/backend/backend1-DB-Schema-and-SQL-code) 마지막에 생성한 `Simple Bank.sql` 스크립트를 복사하여 방금 만든 `00001_init_schema.up.sql` 파일에 복사 붙여넣기를 해 줍니다.

>이번 시리즈의 `Text Editor`는 `VSCode`를 사용합니다. [VSCode 다운로드](https://code.visualstudio.com/)

그리고 `00001_init_schema.down.sql` 파일에 다음과 같이 적어줍니다:

```sql
DROP TABLE IF EXISTS accounts cascade;
DROP TABLE IF EXISTS entries cascade;
DROP TABLE IF EXISTS transfers cascade;
```

이번 마이그레이션의 이름에서도 알 수 있듯이 새로운 DB를 시작하는것 입니다. 따라서 이 마이그레이션의 `down`은 아무것도 없는 빈 `DB Schema`가 되기 때문에 `down`파일에는 존재하는 모든 테이블을 지워주는 스크립트를 적어 줍니다.

---

#### 추가 Docker 커멘드

마이그레이션을 진행하기 전에 우선 `Postgres` `docker container`가 정상적으로 돌 고 있는지 확인을 해 줍니다.

입력:

  ```zsh
    docker ps
  ```

출력:

  ```zsh
    CONTAINER ID   IMAGE                COMMAND                  CREATED        STATUS        PORTS                                       NAMES
    29bb6c666adc   postgres:12-alpine   "docker-entrypoint.s…"   20 hours ago   Up 20 hours   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres12
  ```

정상적으로 돌고 있음이 확인 됩니다.

추가로 활용할 수 있는 `docker`커멘드 몇가지를 살펴 보겠습니다.

##### 현재 돌고 있는 컨테이너를 멈추기

  ```zsh
    docker stop postres12(컨테이너 이름)
  ```

이후 `docker ps`로 돌고 있는 컨테이너를 확인해 보면 아무것도 안뜨는걸 알 수 있다.

입력:

  ```zsh
    docker ps
  ```

출력:

  ```zsh
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
  ```

##### 중지 된 컨테이너와 중지되지 않은 컨테이너 모두 확인하기

입력:

  ```zsh
    docker ps -a
  ```

출력:

  ```zsh
    CONTAINER ID   IMAGE                COMMAND                  CREATED        STATUS                          PORTS     NAMES
    29bb6c666adc   postgres:12-alpine   "docker-entrypoint.s…"   21 hours ago   Exited (0) About a minute ago             postgres12
  ```

모든 컨테이너를 확인하기 위한 방법으로 `-a` 플래그를 추가한다. 컨테이너 `status`가 `Exited`로 표시되는걸 확인할 수 있다.

##### 컨테이너 완전 삭제하기

!!주의!! 아직 이 커멘드는 실행시키지 마세요.

입력:

  ```zsh
    docker rm postgres12
  ```

컨테이너를 완전히 삭제하는 커멘드 입니다.

##### 컨테이너 다시 시작하기

입력:

  ```zsh
    docker start postgres12
  ```

컨테이너가 다시 시작되고, `ps`로 확인해보면

입력:

  ```zsh
    docker start postgres12
  ```

출력:

  ```zsh
    CONTAINER ID   IMAGE                COMMAND                  CREATED        STATUS         PORTS                                       NAMES
    29bb6c666adc   postgres:12-alpine   "docker-entrypoint.s…"   21 hours ago   Up 3 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres12
  ```

`Postgres12` 컨테이너가 다시 정상적으로 돌고 있음이 확인됩니다.

##### Postgres 서버 Shell 접근하기

다음의 커멘드를 사용하여 현재 돌고 있는 `Postgres12` 서버의 `Sh Shell`에 접근할 수 있습니다. 이 시리즈에서 `postgres alpine`이미지를 사용하기 때문에 `ubuntu`에서 처럼 디폴트로 `bash shell`이 설치되어 있지 않습니다. 따라서 `sh shell`에 접근하여 사용해 보겠습니다.

입력:

  ```zsh
    docker exec -it postgres12 /bin/sh
  ```

출력:

  ```zsh
    / # 
  ```

접근이 완료 되어 `postgres12` 서버를 `standard linux` 커멘드로 조작할 수 있게 됩니다.

##### Postgres 서버에 커멘드로 DB 생성하기

`createdb` 커멘드를 사용하여 DB를 생성할 수 있습니다.

입력:

  ```zsh
    createdb --username=root --owner=root simple_bank
  ```

`--username` 옵션을 사용하여 `root`를 지정해 주고 `--owner` 옵션을 사용하여 이 또한 `root`로 지정해 줍니다. 마지막은 DB의 이름을 `simple_bank`로 지정해 줍니다.

DB가 정상적으로 생성되었는지 확인하기 위해 `psql` 커멘드를 사용할 수 있습니다.

입력:

  ```zsh
    psql simple_bank
  ```

출력:

  ```zsh
    psql (12.7)
    Type "help" for help.

    simple_bank=# 
  ```

DB가 생성되고 접근 가능한것을 확인할 수 있습니다.

이제 `\q`를 입력하여 `simple_bank`의 접근을 중지합니다.

입력:

  ```zsh
    simple_bank=# \q
  ```

출력:

  ```zsh
    / # 
  ```

다시 `postgres12` 서버 `sh shell` 메인으로 돌아왔습니다.

##### Postgres 서버에 커멘드로 DB 삭제하기

`dropdb` 커멘드를 사용하여 생성된 DB를 삭제할 수 있습니다.

입력:

  ```zsh
    dropdb simple_bank
  ```

그리고 이를 확인하기 위해 다시 `psql`커멘드를 사용합니다.

입력:

  ```zsh
    psql simple_bank
  ```

출력:

  ```zsh
    psql: error: FATAL:  database "simple_bank" does not exist
  ```

존재하지 않는 DB라고 보여지니 정상적으로 DB가 지워졌음을 알 수 있습니다.

##### 컨테이너 쉘에서 나가기

`Postgres12` 쉘에서 벗어나기 위해 `exit` 커멘드를 입력합니다.

입력:

  ```zsh
    exit
  ```

출력:

  ```zsh
    simplebank git:(master)
  ```

다시 로컬 환경으로 돌아온것을 확인할 수 있습니다.

##### 컨테이너에 접근하지 않고 로컬환경에서 바로 Postgres12 서버에 DB 생성하기

굳이 컨테이너 서버에 접근을 하지않고 바로 로컬에서 `Postgres12` 서버에 DB를 생성할 수 있습니다.

입력:

  ```zsh
    docker exec -it postgres12 createdb --username=root --owner=root simple_bank
  ```

이전과 동일한 방법으로 `psql` 커멘드를 사용하여 DB가 정상적으로 만들어졌는지 확인할 수 있습니다.

입력:

  ```zsh
    docker exec -it postgres12 psql -U root simple_bank
  ```

출력:

  ```zsh
    psql (12.7)
    Type "help" for help.

    simple_bank=#
  ```

##### Makefile을 사용하여 자주 사용하는 커멘드 쉽게 쓰기

`Makefile`을 사용하면 위에서 배운 `createdb`, `dropdb`와 같은 커멘드를 쉽게 사용할 수 있습니다.

먼저 `simplebank` 프로젝트 폴더안에 새로운 파일을 추가하고 파일명을 `Makefile`이라고 적어 줍니다.

그리고 생성된 `Makefile` 파일에 `postgres`, `createdb`, `dropdb` 커멘드를 다음과 같이 추가해 줍니다:
>`postgres` 커멘드는 [백엔드 시리즈 2. Docker와 PostgreSQL 설치하기](https://mannerism.github.io/blog/develop/backend/backend2-Install-Docker-and-PostgreSQL)에서 진행한 `Docker Container 시작하기` 커멘드 라인으로 `postgres12 container`를 만들때 사용한 커멘드 라인입니다.

  ```Makefile
    postgres:
      docker run --name postgres12 -p 5432\:5432 -e POSTGRES_USER=root -e POSTGRES_PASSWORD=secret -d postgres:12-alpine

    createdb:
      docker exec -it postgres12 createdb --username=root --owner=root simple_bank

    dropdb:
      docker exec -it postgres12 dropdb simple_bank

    opendb:
      docker exec -it postgres12 psql -U root simple_bank

    .PHONY: postgres createdb dropdb opendb
  ```

이제 DB를 새로 생성하기위해 `docker exec -it postgres12 createdb --username=root --owner=root simple_bank` 커멘드를 다 쓸 필요 없이 다음과 같이 진행하면 됩니다. `.PHONY`에서 지정한 대로 `createdb`가 자동완성 되는걸 확인할 수 있습니다.

  ```zsh
    make createdb
  ```

이제 본격적으로 `Makefile`에 새로 추가한 커멘드들이 제대로 동작하는지 확인을 해 보겠습니다.

1. `docker ps`로 현재 돌고 있는 컨테이너를 확인합니다.
1. 컨테이너가 돌고 있다면 `docker stop postgres12` 커멘드로 컨테이너를 중지시킵니다.
1. `docker ps -a` 커멘드를 사용하여 모든 컨테이너 리스트를 확인합니다.
1. `docker rm postgres12` 커멘드를 사용하여 중지되어있는 `postgres12` 컨테이너를 완전히 삭제해 줍니다.
1. `docker ps -a` 커멘드를 사용하여 `postgres12` 컨테이너가 완전히 삭제된 걸 확인합니다.
1. `make postgres` 커멘드를 입력합니다.
1. `docker ps` 커멘드를 입력하면 새로운 컨테이너가 생성되어 돌고있다는것을 확인할 수 있습니다.
1. `make createdb` 커멘드를 입력하여 새로 생성된 `postgres12` 서버안에 db를 생성합니다.
1. `make opendb` 커멘드를 입력하여 DB가 정상적으로 생성되었는지 확인합니다.
1. `exit`을 입력하여 빠져나옵니다.

이 스텝을 전부 따라했다면 새로운 `postgres12` 서버가 만들어 지고, `simple_bank` DB가 생성되어 있을겁니다.

---

#### DB 마이그레이션

이제 본격적인 DB 마이그레이션을 시작해 보겠습니다.

1. `Table Plus`를 열고 [백엔드 시리즈 2. Docker와 PostgreSQL 설치하기](https://mannerism.github.io/blog/develop/backend/backend2-Install-Docker-and-PostgreSQL)의 `Table Plus 설치 및 세팅하기`에서 진행한 대로 연결되어있는 `postgres12`를 열어줍니다.
1. 중앙 상단 `SQL` 좌측에 보이는 `DB`아이콘을 클릭합니다.
1. `Open database`라는 창이 뜨면 `simple_bank` DB를 선택하고 `open`을 누릅니다.
1. 좌측 패널에 `root` DB와 `simple_bank` DB가 생겨난걸 확인할 수 있으며, `simple_bank` DB는 비어있는걸 확인할 수 있습니다.

이제 이 비어있는 `simple_bank` DB에 이전에 다운받아 설치한 `migrate` library를 사용하여 DB를 옯겨 보겠습니다.

터미널로 돌아가서 다음과 같이 입력해 줍니다.

입력:

  ```zsh
    migrate -path db/migration -database "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable" -verbose up
  ```

1. `-path` 플래그를 사용하여 마이그레이션 파일의 위치를 알려줍니다.
1. `-database` 플래그를 사용하여 DB의 URL을 알려줍니다. 우리는 `postgres`를 사용하기 때문에 드라이버의 이름을 `postgresql://`로 사용합니다. 그리고 사용자명 `root`를 입력해주고 비밀번호 `secret`을 입력해 줍니다. `postgres`서버가 돌고 있는 `localhost:5432`주소를 입력해주고 뒤에 DB명인 `simple_bank`를 입력해줍니다. 현재로서는 `postgres`서버가 디폴트로 `ssl`을 활성화하지 않기 떄문에 `sslmode=disable`파라미터를 추가해 줍니다.
1. `-verbose` 플래그를 추가하여 보다 상세한 로그를 보여줄 수 있도록 합니다.
1. 마지막으로 `up`을 추가하여 이번 마이그레이션이 `up`마이그레이션인 것을 알려줍니다.

출력:

  ```zsh
    2021/08/01 02:34:46 Start buffering 1/u init_schema
    2021/08/01 02:34:46 Read and execute 1/u init_schema
    2021/08/01 02:34:46 Finished 1/u init_schema (read 11.027243ms,   ran 35.780731ms)
    2021/08/01 02:34:46 Finished after 55.252024ms
    2021/08/01 02:34:46 Closing source and database
  ```

다시 `Table Plus`로 돌아가 `simple_bank` DB를 선택하고 `cmd` + `r`을 누르면 마이그레이션이 정상적으로 이루어진 4개의 테이블을 확인할 수 있습니다.

`Makefile`에 `migrateup`과 `migratedown` 커멘드를 추가해 줍니다.

  ```Makefile
    migrateup:
      migrate -path db/migration -database "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable" -verbose up

    migratedown:
      migrate -path db/migration -database "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable" -verbose down

    .PHONY: postgres createdb dropdb opendb migrateup migratedown
  ```

`make migratedown`를 진행해 봅니다.

입력:

  ```zsh
    make migratedown
  ```

출력:

  ```zsh
    migrate -path db/migration -database "postgresql://root:secret@localhost:5432/simple_bank?sslmode=disable" -verbose down
    2021/08/01 02:59:38 Are you sure you want to apply all down migrations? [y/N]
    y
    2021/08/01 02:59:59 Applying all down migrations
    2021/08/01 02:59:59 Start buffering 1/d init_schema
    2021/08/01 02:59:59 Read and execute 1/d init_schema
    2021/08/01 02:59:59 Finished 1/d init_schema (read 10.885675ms, ran 17.12311ms)
    2021/08/01 02:59:59 Finished after 20.687695639s
    2021/08/01 02:59:59 Closing source and database
  ```

`Table Plus`로 돌아가서 `cmd` + `r`을 눌러 새로고침을 해 주면 `simple_bank` DB의 테이블이 `migrate down`되어 삭제된 것을 확인할 수 있습니다. 앞으로 마이그레이션은 `migrate up`, `migrate down`으로 비교적 쉽게 진행할 수 있겠습니다.

사리즈3 끝.

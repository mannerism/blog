---
title:  "백엔드 시리즈 3. DB 마이그레이션"
date:   2021-07-31 12:27:19 +0900
permalink: ":categories/backend/:title"
---

> 원문 출처: [Techschool](https://www.youtube.com/watch?v=rx6CPDK_5mU&list=PLy_6D98if3ULEtXtNSY_2qN21VCKgoQAE&ab_channel=TECHSCHOOL "Tech School")

### 백엔드 시리즈 1. DB 마이그레이션

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

두 개의 `up` `down` 마이그레이션 파일이 만들어졌습니다. 각 파일명 앞에 버전 `00001`이 붙어있는걸 확인할 수 있습니다.

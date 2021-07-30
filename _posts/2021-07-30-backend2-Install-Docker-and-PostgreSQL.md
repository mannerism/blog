---
title:  "백엔드 시리즈 2. Docker와 PostgreSQL 설치하기"
date:   2021-07-30 12:27:19 +0900
permalink: ":categories/backend/:title"
---

> 원문 출처: [Techschool](https://www.youtube.com/watch?v=rx6CPDK_5mU&list=PLy_6D98if3ULEtXtNSY_2qN21VCKgoQAE&ab_channel=TECHSCHOOL "Tech School")

### 백엔드 시리즈 2. Docker와 PostgreSQL 설치하기

이번 시리즈 2에서 할 일:

1. `Docker Desktop`을 설치
2. `Postgres` 를 `docker container`에 실행
3. `Table Plus` 설치
4. DB Schema 생성

---

#### Docker Desktop 설치

[Docker Offical Website](https://www.docker.com/products/docker-desktop)로 들어가서 자신의 OS에 맞는 `Docker Desktop`을 설치 해 줍니다. `Brew`를 사용해서 설치할 수도 있고 그냥 앱을 다운받아서 설치하는 방법이 있으니 각자 기호에 맞는 방법으로 설치를 진행 해줍니다. 설치가 완료되면 `docker`를 실행 시켜 줍니다. Mac의 경우 `Status Bar`에 `docker`가 실행됨이 표시 되며, 아이콘을 클릭하면 프로그램이 정상적으로 실행되고 있음을 표시하는 초록불이 표시 됩니다.

이제 터미널을 열어주고 `docker`를 이것저것 만져 봅니다.

>참고: docker 커멘드 문서: [Docker Commands](https://docs.docker.com/engine/reference/commandline/ps/)

터미널에 다음과 같이 입력후 엔터를 칩니다.

입력:

  ```zsh
    docker ps
  ```

출력:

  ```zsh
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
  ```

현재 돌고 있는 컨테이너가 없기 떄문에 아무것도 뜨지 않습니다. 이미지도 한번 확인해 보겠습니다.

입력:

  ```zsh
    docker images
  ```

출력:

  ```zsh
    REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
  ```

이미지 역시 아직 없기 때문에 아무것도 나오지 않습니다.

---

#### Docker image 생성하기

이 시리즈에서는 `PostgreSQL`을 DB Engine으로 사용하기 때문에 [Postgres Docker Hub](https://hub.docker.com/_/postgres)에들어가서 입력하여 사용할 이미지 버전을 확인해 줍니다. 그리고 다시 터미널을 열고 다음과 같이 입력을 하여 이미지를 불러 옵니다.

`docker`에서 이미지를 가져오는 커멘드 syntax:
  >`docker pull <image>:<tag>`

입력:

  ```zsh
    docker pull postgres:12-alpine
  ```

출력(이미지 불러오기 완료):

  ```zsh
    12-alpine: Pulling from library/postgres
    5843afab3874: Pull complete 
    525703b16f79: Pull complete 
    86f8340bd3d9: Pull complete 
    7627208d44d3: Pull complete 
    055fcb3625a8: Pull complete 
    b8f670b8a8c0: Pull complete 
    586264c967a3: Pull complete 
    dcd1f4239533: Pull complete 
    Digest: sha256:eeec7282ab2a0b6d87be5e8d35cb8fe2b77e479b3560430b2d4d75fe254389e5
    Status: Downloaded newer image for postgres:12-alpine
    docker.io/library/postgres:12-alpine
  ```

이 시리즈에서는 비교적 가벼운 `postgres 12-alpine`을 사용하여 이미지를 가져오겠습니다. 

이제 다시 `docker` 이미지를 확인해 보겠습니다.

입력:

  ```zsh
    docker images
  ```

출력:

  ```zsh
    REPOSITORY   TAG         IMAGE ID       CREATED       SIZE
    postgres     12-alpine   7d6f672908a4   4 weeks ago   190MB
  ```

방금 추가한 `Postgres` 이미지가 보여집니다.

---

#### `Docker` Container 시작하기

이제 컨테이너를 만들어서 추가된 `docker` 이미지를 실행시킬 수 있습니다. 1개의 이미지를 다양한 컨테이너에 추가하여 실행시킬 수 있습니다.

컨테이너를 시작하기위해서 사용하는 커멘드라인 기본 syntax:

  >docker run --name<컨테이너 이름> -e<환경 변수> -d <이미지>:<태그>

`docker` 컨테이너는 새로운 가상환경에서 실행되기 때문에 현재 저희 컴퓨터에서 접속하는 `host network`와 분리되어 있습니다. 따라서 `docker` 컨테이너가 돌고 있는 port 5432로 연결하기 위해서는  환경변수 플래그인 `-e`앞에 `-p` 플래그를 추가하여 `Port Mapping`을 진행해 주어야 합니다.

터미널에 입력해 봅니다.

입력:

  ```zsh
     docker run --name postgres12 -p 5432:5432 -e POSTGRES_USER=root -e POSTGRES_PASSWORD=secret -d postgres:12-alpine
  ```

출력:

  ```zsh
    29bb6c666adc4f271c64e2a6b73816431595da5c29253d741bdcb4eb4bb01643
  ```

컨테이너가 생성되고, 컨테이너의 `unique id`가 출력 됩니다. 이제 컨테이너가 정상적으로 실행되고 있는지 확인해 봅시다.

입력:

  ```zsh
    docker ps
  ```

출력:

  ```zsh
    CONTAINER ID   IMAGE                COMMAND                  CREATED          STATUS          PORTS                                       NAMES
    29bb6c666adc   postgres:12-alpine   "docker-entrypoint.s…"   19 minutes ago   Up 19 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   postgres12
  ```

새로운 컨테이너가 정상적으로 작동하고 있는것을 확인할 수 있습니다.

이제 `Postgres` 서버도 정상적으로 실행되고 있으니, 직접 해당 서버에 접속해 보도록 하겠습니다.

컨테이너안에 1개의 커맨드를 실행시키는 syntax:
  >`docker exec -it <컨테이너 이름 또는 id> <커멘드> [args]

입력:

  ```zsh
    docker exec -it postgres12 psql -U root
  ```

여기서 우리는 `postgres12`라는 이름의 컨테이너안에 `psql` 커멘드를 사용하여 해당 `Postgres`서버의 콘솔에 접근을 할 것이며, `-U` 플래그를 사용하여 `root`유저로 접근을 희망한다는 명령을 보낼 수 있습니다.

출력:

  ```zsh
    psql (12.7)
    Type "help" for help.

    root=# 
  ```

접근이 완료된 모습을 볼 수 있습니다. 한가지 명심해야할 것은, 컨테이너를 실행시킬 때 환경 변수로 `-e POSTGRES_PASSWORD=secret`를 입력해 주었지만 방금 컨테이너 콘솔에 접근을 할 시에는 비밀번호를 물어보지 않았다는 점입니다. 이유는 [Postgres Docker Hub](https://hub.docker.com/_/postgres) `Environment Variables`에 적힌바와 같이 `localhost`에서 접근할 시에는 암호를 물어보지 않습니다.

  >Note 1: The PostgreSQL image sets up trust authentication locally so you may notice a password is not required when connecting from localhost (inside the same container). However, a password will be required if connecting from a different host/container.

테스트로 몇가지 커멘드를 입력해 봅니다.

입력:
  
  ```zsh
    root=# select now();
  ```

출력:

  ```zsh
                  now              
    -------------------------------
    2021-07-30 07:11:10.090379+00
    (1 row)
  ```

현재 시간을 가지고 오는 커멘드를 테스트로 돌려보니 잘 되는걸 확인할 수 있습니다. 이제 접근을 종료하겠습니다.

접근 종료 커멘드 입력:

  ```zsh
    root=# \q
  ```

추가로 컨테이너의 로그를 확인하는 방법을 알아보겠습니다.

컨테이너 로그를 확인하는 커멘드 syntax:
  >docker logs <컨테이너 이름 또는 id>

한번 실행해 봅니다.

입력:

  ```zsh
    docker logs postgres12
  ```

출력:

  ```zsh
    The files belonging to this database system will be owned by user "postgres".
    ...

  Success. You can now start the database server using:

    pg_ctl -D /var/lib/postgresql/data -l logfile start
    ...
  
    2021-07-30 06:26:57.518 UTC [35] LOG:  database system is shut down
    done
    server stopped

    PostgreSQL init process complete; ready for start up.
  ```

로그 정보가 제대로 출력되는것을 확인할 수 있습니다.

---

#### Table Plus 설치하기

Table Plus 프로그램을 사용하여 DB 관리를 좀 쉽게 진행해 보겠습니다. 2021년 7월 30일 기준 몇번 사용하면 유료 구매를 권장하는 팝업이 좀 거슬리지만 한번 프로그램을 껏다가 키면 사라지니, 이번 시리즈에는 좀 참고 사용해 봅시다.

1. [Table Plus Official 사이트](https://tableplus.com/)로 들어가서 각자 운영체제에 맞는 프로그램을 다운받고 프로그램을 설치해 줍니다.
1. 설치가 완료 되면 프로그램을 실행 시키고 가운데 하단에 `Create a new connection`이란 조그만 버튼(왜 저렇게 작게 만들었는지 의문)을 클릭합니다.
1. `PostgreSQL`을 선택하고 `create` 버튼을 누릅니다.
1. `PostgreSQL Connection`이란 창이 뜨게 되고 다음과 같이 입력해 줍니다.

    - `name` = postgres12
    - `User` = root
    - `Password` = secret
    - 그 외의 필드는 그대로 둡니다.

1. `Test` 버튼을 클릭하여 입력한 필드가 초록색으로 변하는지 확인 합니다.
1. `Connect` 버튼을 클릭하여 연결을 완료합니다.

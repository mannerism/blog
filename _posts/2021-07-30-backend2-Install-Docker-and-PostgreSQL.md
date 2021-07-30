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

> 참고: docker 커멘드 문서: [Docker Commands](https://docs.docker.com/engine/reference/commandline/ps/)

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

이 시리즈에서는 `PostgreSQL`을 DB Engine으로 사용하기 때문에 [Docker Hub](https://hub.docker.com/)에서 검색창에 `PostgreSQL`을 입력하여 맨 처음에 올라온 official 이미지 버전을 확인해 줍니다. 그리고 다시 터미널을 열고 다음과 같이 입력을 하여 이미지를 불러 옵니다.

입력:

  ```zsh
    docker pull postgres:12-alpine
  ```

출력:

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

이 시리즈에서는 비교적 가벼운 `postgres 12-alpine`을 사용하여 이미지를 가져오겠습니다. `docker`에서 이미지를 가지고 오는 커멘드 syntax는 `docker pull <image>:<tag>` 입니다.

---
title: docker-compose 작성 시 환경 변수 지정
author: Jandari
date: 2022-03-24 18:46:00 -0500
categories: [Docker, docker-compose]
tags: [Docker, Postgres]
math: true
mermaid: true
---

## 개요

docker-compose는 IT 관리자가 여러 개의 컨테이너를 한번의 명령으로 실행 시키기 위해 `정의해 놓은 파일을 실행시키는 툴`입니다.

이때 IT 관리자가 정의 되어 있는 파일을 수정하지 않고 `환경 변수를 통해 재사용` 할 수 있습니다.

이때 환경변수를 정의하기 위해 사용하는 파일이 `환경 변수 파일` 입니다.

#### 설명

아래와 같은 `Postgres` 컨테이너를 띄우기 위한 `docker-compose` 파일이 있다.

```yml
services:
  postgres:
    image: postgres:13
    container_name: postgres
    ports:
      - '5432:5432'
    environment:
      - POSTGRES_PASSWORD=pw1234
```

여기서 `Postgres`의 암호는 `pw1234`로 정의 되어 있다.

하지만 비밀번호를 변경하기 위해서는 docker-compose.yml 파일을 수정해야 한다.

위에는 서비스가 1개여서 복잡하지 않지만 서비스가 굉장히 많을 때는 실수가 생기기 마련이다.

이 때 `.env` 파일을 생성하여 환경변수로 비밀번호를 지정 할 수 있다.

docker-compose와 동일 레벨에 있을 때는 따로 옵션을 주지 않아도 되지만 파일 경로를 지정하고 싶을 때는 `--env-file` 옵션을 사용하면 된다.

#### 환경변수 파일 사용

아래와 같이 파일을 구성한다.

```
postgres(Directory)
└─.env
└─docker-copmose.yml
```


아래와 같이 docker-compose.yml 파일에 환경변수를 사용하도록 수정한다.


```yml
services:
  postgres:
    image: postgres:13
    container_name: postgres
    ports:
      - '5432:5432'
    environment:
      - POSTGRES_PASSWORD=${PASSWD}
```

그리고 아래와 같이 `.env` 파일을 작성한다.

```
PASSWD=pw1234
```

그리고 `docker-compose up`로 실행시키면 된다.

#### 예제

[example.zip](/assets/img/post/2022-03-24-Environment/example.zip)
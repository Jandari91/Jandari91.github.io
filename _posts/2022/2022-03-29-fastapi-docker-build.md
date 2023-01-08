---
title: FastAPI Docker 이미지 제작
author: Jandari
date: 2022-03-29 15:00:00 -0500
categories: [FastAPI]
tags: [FastAPI, Python, Docker]
math: true
mermaid: true
---

## 소개

이번 포스트는 FastAPI 서비스를 Docker Image로 만들어 띄우는 방법에 대해 소개합니다.

## main.py 생성

main.py 파일을 생성하여 기본 코드를 작성합니다.

```py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def root():
    return {"hello root"}

@app.get("/world")
def world():
    return {"hello world"}
```

해당 코드는 `/`로 접속하면 `[hello root]`가 출력 되고 `/world`로 접속하면 `[hello world]`가 출력 되는 간단한 프로그램입니다.

## pip 모듈 정의

pip는 필요 모듈을 따로 정의하여 자동으로 패키지 추가가 가능합니다.

`requirements.txt` 파일을 생성하고 필요한 패키지를 정의 합니다.

```
uvicorn
fastapi
```

FastAPI는 `uvicorn`과 `fastapi` 모듈이 기본적으로 필요합니다.

## Docker Image 생성

Docker Image를 만들기 위해서는 Dockerfile을 정의하고 Docker build를 통해 자신만의 이미지를 생성하면 됩니다.

#### Dockerfile 정의

`Dockerfile`을 생성합니다.

```
FROM python:latest

WORKDIR /app/

COPY ./main.py /app/
COPY ./requirements.txt /app/

RUN pip install -r requirements.txt

CMD uvicorn --host=0.0.0.0 --port 8000 main:app
```

* `FROM python:latest` : 베이스가 되는 Docker Image로 python 이미지를 사용합니다.
* `WORKDIR /app/` : 처음 실행 시 사용 되는 경로 정보 입니다.
* `COPY ./main.py /app/` : 현재 경로의 main.py를 /app 경로로 복사합니다.
* `COPY ./requirements.txt /app/` : 현재 경로의 requirements.txt를 /app 경로로 복사합니다.
* `RUN pip install -r requirements.txt`: 복사 된 requirements.txt를 사용하여 pip로 패키지를 추가합니다.
* `CMD uvicorn --host=0.0.0.0 --port 8000 main:app` : uvicorn을 사용하여 main.py의 app을 실행시킵니다.

#### Docker Build

`Dockerfile`이 있는 경로에서 `docker build` 명령어를 실행합니다.

```
> docker build --tag fastapi:0.1 .
```

* `--tag` : 생성한 이미지의 이름을 설정합니다. 설정하지 않으면 None으로 생성됩니다.

## Docker Container 생성

`Dockerfile`의 정의를 보면 기본 포트는 `8000`으로 되어 있습니다.

Container를 실행 할 때 Port 포워딩을 통해 원하는 포트를 지정하여 실행 할 수 있습니다.

```
> docker run --rm -p 8080:8000 fastapi:0.1

INFO:     Started server process [8]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

* `--rm` : 이미지가 stop 되면 자동으로 Container가 삭제 됩니다.
* `-p 8080:8000` : FastAPI의 기본 포트인 8000과 자신의 로컬 포트 8080을 포워딩 합니다.

[http://localhost:8080](http://localhost:8080)

```
["hello root"]
```

[http://localhost:8080/world](http://localhost:8080/world)


```
["hello world"]
```

## docker-compose 추가

docker-compose는 여러 컨테이너를 한번에 띄워야 할 때 하나의 파일로 정의하여 관리하기 위해 사용합니다.

`docker image` 생성 부터 실행까지 자동으로 할 수 있습니다.

`docker-compose.yml` 파일을 생성합니다.

```yml
services:
  fastapi:
    build:
      context: .
      dockerfile : .
    ports:
      - 8080:8000
```

* `services` : service를 정의합니다.
* `fastapi`: service의 이름입니다.
* `build` : docker image를 생성 할 build에 대해 정의합니다.
* `context` : build를 실행 할 경로 입니다.
* `dockerfile` : dockerfile의 경로 입니다.
* `ports` : 포트포워딩 정보 입니다.

`docker-compose.yml` 파일이 있는 곳에서 아래의 명령어를 실행합니다.

```
> docker-compose up

Creating src_fastapi_1 ... done
Attaching to src_fastapi_1
fastapi_1  | INFO:     Started server process [8]
fastapi_1  | INFO:     Waiting for application startup.
fastapi_1  | INFO:     Application startup complete.
fastapi_1  | INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

## 예제

[fastapi_docker.zip](/assets/img/post/2022-03-29-fastapi-docker-build/fastapi_docker.zip)
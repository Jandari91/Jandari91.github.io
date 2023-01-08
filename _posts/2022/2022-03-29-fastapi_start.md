---
title: FastAPI 실행하기
author: Jandari
date: 2022-03-29 14:00:00 -0500
categories: [FastAPI]
tags: [FastAPI, Python]
math: true
mermaid: true
---

## 소개

이번 포스트는 `FastAPI`를 python 코드로 실행하고 확인하는 방법에 대해 설명합니다.

## 시작하기

`FastAPI`는 실행 방법이 두가지 있습니다.

python에서 uvicorn 모듈을 사용하여 직접 실행하는 방법과 디버깅을 위해 main 함수에 uvicorn 실행 코드를 넣는 방법 입니다.

#### 직접 실행

아래와 같이 `main.py` 파일에 코드를 입력 합니다.

```py
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```

위 코드는 FastAPI의 간단한 코드 입니다.

VS Code에서 `F5`를 눌러 실행시키면 서비스가 실행 되지 않고 바로 종료가 됩니다.

실행 시키기 위해서는 main.py를 uvicorn을 통해 실행 시켜야 합니다.

```
> python.exe -m uvicorn main:app --reload

INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [26272] using watchgod
INFO:     Started server process [15080]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

#### 디버깅

위 직접 실행의 경우 디버깅이 불가능 하기 때문에 main 함수에 uvicorn을 호출하여 실행하는 방법이 있습니다.

```py
import uvicorn
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def root():
    a = "a"
    b = "b" + a
    return {"hello world": b}


if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

위와 같이 선언하면 바로 실행이 가능합니다.

```
INFO:     Started server process [504]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

## 살펴보기

아래와 같이 `main.py`를 만들어 실행 시킵니다.

```py
import uvicorn
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def root():
    return {"hello root"}

@app.get("/world")
def world():
    return {"hello world"}


if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```



#### RestAPI 호출

위에서 실행시킨 코드를 살펴보면 `8000` 포트로 접속하였을 때 FastAPI로 실행시킨 서비스에 접근이 가능합니다.

`/`로 접근하였을 때는 `hello root`라는 리턴값이 반환 되고 `/world`로 접근하였을 때는 `hello world`라는 리턴값이 반환 됩니다.

[http://localhost:8000](http://localhost:8000)

```
["hello root"]
```

[http://localhost:8000/world](http://localhost:8000/world)


```
["hello world"]
```

#### Swagger

FastAPI는 Swagger를 기본적으로 지원합니다.

[http://localhost:8000/docs](http://localhost:8000/docs)

![image](/assets/img/post/2022-03-29-fastapi_start/1.jpg)

Swagger를 통해 Web UI에서 RestAPI들을 테스트 할 수 있습니다.

#### Redocs

OpenAPI의 경우 서비스를 사용하는 곳에 스펙에 대해 문서로 전달해야 하는 경우가 많습니다.

이 때 Redocs를 사용하게 되면 자동으로 API와 동기화가 이뤄지는 문서를 배포 가능합니다.

[http://localhost:8000/redoc](http://localhost:8000/redoc)

![image](/assets/img/post/2022-03-29-fastapi_start/2.jpg)

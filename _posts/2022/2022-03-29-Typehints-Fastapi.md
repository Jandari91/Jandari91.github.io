---
title: FastAPI의 Type hits
author: Jandari
date: 2022-03-29 10:00:00 -0500
categories: [FastAPI]
tags: [FastAPI, Python]
math: true
mermaid: true
---

* [https://fastapi.tiangolo.com/ko/python-types/](https://fastapi.tiangolo.com/ko/python-types/)

## Python Types란

FastAPI는 Python의 `type hints`를 사용합니다.

python은 기본적으로 동적(dynamic)프로그래밍 언어로써 인터프리터(interpreter)가 코드를 실행하면서 Type을 추론하여 체크하기 때문에 Type이 고정되어 있지 않습니다.

FastAPI에서는 type을 선언하여 사용하므로 Editor와 Tools에서 디버깅에 대해 더 나은 경험을 제공 할 수 있습니다.

## 인텔리센스(IntelliSense) 지원

```py
def get_full_name(first_name, last_name):
    full_name = first_name.title() + " " + last_name.title()
    return full_name

print(get_full_name("john", "doe"))
```

```
John Doe
```

위의 예제는 `first_name`과 `last_name`을 가져오고 각 문자의 첫 글자를 대문자로 변환 후 출력하는 간단한 예제입니다.


#### Type을 추가 했을 경우

위 프로그램은 간단한 코드이기 때문에 금방 쓰게 되었지만 만약 처음부터 쓰고 있다고 했을 때 아래와 같이 인텔리젠스에서 힌트를 얻을 수가 없습니다.

![image](/assets/img/post/2022-03-29-Typehints-Fastapi/1.jpg)

```
first_name, last_name
```

매개변수에 아래와 같이 유형을 추가하여 줍니다.

```
first_name: str, last_name: str
```

![image](/assets/img/post/2022-03-29-Typehints-Fastapi/2.jpg)

`type hints`를 추가하는 것 만으로도 코드를 작성 할 때 인텔리전스에서 어떤 것을 사용 할 수 있는지 지원이 가능합니다.

## 런타임 전 에러 체크

#### VS Code Type Check 활성화

VS Code에서 Python Type Check를 사용하기 위해서는 아래의 포스트를 참고해 주시기 바랍니다.

[VS Code에서 Python Type Check 기능 활성화](/posts/python-typecheck-vscode/)

#### 코드 추가

```py
def get_name_with_age(name: str, age: int) -> str : 
    name_with_age = name + " is this old " + age
    return name_with_age

get_name_with_age("Jhon", 19)
```
위와 같이 코드를 추가하고 실행을 시킵니다.

하지만 age에서 `TypeError`가 발생하게 됩니다.

VS Code에서 런타임 전 아래와 같이 오류를 미리 볼 수 있습니다.

![image](/assets/img/post/2022-03-29-Typehints-Fastapi/3.jpg)

## 표준 Type 지원

Python의 표준 타입 뿐만 아니라 모든 타입을 정의 할 수 있습니다.

* int
* float
* bool
* bytes

```py
def get_items(item_a: str, item_b: int, item_c: float, item_d: bool, item_e: bytes):
    return item_a, item_b, item_c, item_d, item_d, item_e
```

## 변수를 포함한 Type

`dict`, `list`, `set`, `tuple` 등 내부 Type을 정의하여 사용하는 데이터 구조도 내부 유형에 대해 정의가 가능합니다.

#### list

* python3.9+


```py
def process_items(items: list[str]):
    for item in items:
        print(item)
```

list 안의 타입이 str이라고 정의 하여 사용 할 수 있습니다.

#### tuple & set
* python3.9+

```py
def process_items(items_t: tuple[int, int, str], items_s: set[bytes]):
    return items_t, items_s
```

#### dict

* python3.9+

```py
def process_items(prices: dict[str, float]):
    for item_name, item_price in prices.items():
        print(item_name)
        print(item_price)
```

#### Union

* python3.10+

```py
def process_item(item: int | str):
    print(item)
```

Union의 경우 변수가 여러 유형 중 하나가 있다는 것을 정의 할 수 있습니다.

위 예제는 item의 Type이 int 이거나 str 일 때 사용합니다.

## 사용자 정의 DataType

```py
class Person:
    def __init__(self, name: str):
        self.name = name


def get_person_name(one_person: Person):
    return one_person.name
```

![image](/assets/img/post/2022-03-29-Typehints-Fastapi/4.jpg)

사용자 정의 DataType도 위 그림과 같지 인텔리센스를 사용 할 수 있습니다.

## Pydantic Models

`Pydantic`은 `Data Type`에 속성이 있을 경우 사용합니다.

`Pydantic`은 데이터를 검증하고 설정들을 관리하는 라이브러리로 런타임 환경에서 타입을 강제하고 타입이 유효하지 않을 때 에러를 발생시켜 줍니다.

```py
from datetime import datetime

from pydantic import BaseModel


class User(BaseModel):
    id: int
    name = "John Doe"
    signup_ts: datetime | None = None
    friends: list[int] = []


external_data = {
    "id": "123",
    "signup_ts": "2017-06-01 12:22",
    "friends": [1, "2", b"3"],
}
user = User(**external_data)
print(user)
# > User id=123 name='John Doe' signup_ts=datetime.datetime(2017, 6, 1, 12, 22) friends=[1, 2, 3]
print(user.id)
# > 123
```
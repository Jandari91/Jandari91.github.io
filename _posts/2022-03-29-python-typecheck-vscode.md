---
title: VS Code에서 Python Type Check 기능 활성화
author: Jandari
date: 2022-03-29 11:00:00 -0500
categories: [VSCode]
tags: [VSCode, Python]
math: true
mermaid: true
---

## 소개

python은 기본적으로 동적(dynamic)프로그래밍 언어로써 인터프리터(interpreter)가 코드를 실행하면서 Type을 추론하여 체크하기 때문에 Type이 고정되어 있지 않습니다.

하지만 `Python 3.5` 부터 Python에서도 Type을 체크할 수 있도록 `Type Hints`라는 기능이 도입되어 ㅆ습니다.

데이터형에 주석을 붙여 사용하게 됩니다.

```py
def greeting(name:str) -> str:
    return 'Hello' + name
```

* `name:str` : 인수 name의 Type이 str이라는 것을 어노테이션합니다.
* `-> str` : 함수 반환값의 Type이 str이라는 것을 어노테이션 합니다.

#### VSCode Type Check 활성화

VS Code에서 아래와 같이 코딩을 합니다.


```py
def get_name_with_age(name: str, age: int) -> str : 
    name_with_age = name + " is this old " + age
    return name_with_age

get_name_with_age("Jhon", 19)
```

위 코드를 동작시키면 age의 형변환을 하지 않아 오류가 생기기 됩니다.

하지만 VS Code에서는 따로 Run Time 전에 따로 인텔리전스가 되지 않습니다.

해당 기능을 활성화 시키기 위해서는 `파일 -> 기본설정 -> 설정`으로 이동하거나 `Ctrl + ,`를 사용합니다.

검색에서 `Type Checking Mode`를 검색합니다.

![image](/assets/img/post/2022-03-29-python-typecheck-vscode/1.jpg)

위와 같이 `Type Checking Mode`가 `Off` 상태 인것을 알 수 있습니다.

`basic`으로 상태를 변경하여 줍니다.

![image](/assets/img/post/2022-03-29-python-typecheck-vscode/2.jpg)

`VS Code`에서 런타임 전에 오류를 체크해주는 것을 확인 할 수 있습니다.
---
title: 5장. 객체 지향 프로그래밍
author: Jandari
date: 2022-05-02 23:30:00 +09:00
categories: [DDD, 클린 아키텍처 소프트웨어 구조와 설계의 원칙, 2부 벽돌부터시작하기:프로그래밍 패러다임]
tags: [DDD, CleanArchitecture]
math: true
mermaid: true
---

# 소개

![image](/assets/img/post/2022-05-01-PPPCleanArchitecture_ch5/1.jpg)

클린아키텍처: 소프트웨어 구조와 설계의 원칙 책을 읽고 정리하며 소감을 적는 포스트입니다.

# 객체 지향 프로그래밍

좋은 아키텍처를 만드는 일은 객체지향(OO) 설계 원칙을 이해하고 응용하는 데서 출발한다. 그럼 OO란 무엇인가?

"데이터와 함수의 조합"의 답은 o.f()가 왼지 f(o)와 다르다는 의미를 내포하고 있어 별로다.

"시렞 세계를 모델링하는 새로운 방법"의 답은 얼버무리는 수준이다.

OO의 본질을 설명하기 위해 캡슐화(encapsulation), 상속(inheritance), 다형성(polymorphism)이 세 가지 개념을 적절하게 조합한 것이거나, 또는 OO 언어는 최소한 세 가지 요소를 반드시 지원해야 한다고 말하는 부류들이 있다.

## 캡슐화?

데이터와 함수가 응집력 있게 구성된 집단을 서로 구분 짓는 선을 그을 수 있다.

구분선 바깥에서 데이터는 은니고디고, 일부 함수만이 외부에 노출된다.

사실 C 언어에서도 완벽한 캡슐화가 가능하다.

>point.h
```c
struct Point;
struct Point* makePoint(double x, double y);
double distance (struct Point *p1, struct Point *p2);
```

>point.c
```c
#include "point.h"
#include <stdlib.h>
#include <math.h>

struct Point {
    double x,y;
}

struct Point* makepoint(double x, double y) {
    struct Point* p = malloc(sizeof(struct Point));
    p->x = x;
    p->y = y;
    return p;
}

double distance(struct Point* p1, struct Point* p2){
    double dx = p1 -> x - p2 -> x;
    double dy = p1 -> y - p2 -> y;
    return sqrt(dx*dx+dy*dy);
}
```

point.h를 사용하는 측에서는 `struct Point`의 멤버에 접근할 방법이 전혀 없다.

이것이 바로 완벽한 캡슐화이며, 보다시피 OO가 아닌 언어에서도 충분히 가능하다.

Java와 C#은 헤더와 구현체를 분리하는 방식을 모두 버렸고, 이로 인해 캡슐화는 더웃 심하게 훼손되었다. 이들 언어에서는 클래스 선언과 정의를 구분하는게 아예 불가능하다.

OO 프로그래밍은 프로그래머가 충분히 올바르게 행동함으로써 캡슐화된 데이터를 우회해서 사용하지 않을 거라는 믿음을 기반으로 하지만 실제로는 C 언어에서 누렸던 완벽한 캡슐화를 약화시켜 온 것이 틀림없다.

## 상속?

상속이란 단순히 어떤 변수와 함수를 하나의 유효 범위로 묶어서 재정의하는 일에 불괗다.

> namedPoint.h
```c
struct NamedPoint;

struct NamedPoint* makeNamedPoint(double x, double y, char* name);
void setName(struct NamedPoint* np, char* name);
char* getName(struct NamedPoint* np);
```

> namedPoint.c
```c
#include "namedPoint.h"
#include <stdlib.h>

struct NamedPoint {
    double x,y;
    char *name;
};

struct NamedPoint* makeNamedPoint(double x, double y, char* name) {
    struct NamedPoint* p = malloc(sizeof(struct NamedPoint));
    p->x = x;
    p->y = y;
    p->name = name;
    return p;
}

void setName(struct NamedPoint* np, char* name) {
    np->name = name;
}

char* getName(struct NamedPoint* np) {
    return np -> name;
}
```

> main.c

```c
#include "point.h"
#include "namedPoint.h"
#include <stdio.h>


int main(int ac, char** av) {
    struct NamedPoint* origin = makeNamedPoint(0.0, 0.0, "origin");
    struct NamedPoint* upperRight = makeNamePoint(1.0, 1.0, "upperRight");

    printf("distance=%f\n", distance(
        (struct Point*) origin,
        (struct Point*) upperRight
    ));
}
```

main을 보면 NamedPoint 데이터 구조가 마치 Point 데이터 구조로 부터 파생된 구조인 것처럼 동작한다는 사실을 볼 수 있다.

NamedPoint는 Point의 가면을 쓴 것처럼 동작 할 수 있는데, 이는 NamedPoint가 순전히 Poiont를 포함하는 상위 집합으로, Point에 대응하는 멤버 변수의 순서가 그대로 유지되기 때문이다.

## 다형성?

```c
#include <stdio.h>

void copy() {
    int c;
    while ((c=getchar()) != EOF)
        putchar(c);
}
```

getchar() 함수는 STDIN에서 문자를 읽는다. 그러면 STDIN은 어떤 장치인가?

putchar()함수는 STDOUT으로 문자를 쓴다. 그런데 STDOUT은 또 어떤 장치인가?

이러한 함수는 다형적(polymorphic)이다. 즉, 행위가 STDIN과 STDOUT의 타입에 의존한다.

FILE 데이터 구조는 열기(open), 닫기(close), 읽기(read), 쓰기(write), 탐색(seek)의 표준함수들을 포함한다.

```c
struct FILE {
    void (*open)(char* name, int mode);
    void (*close)();
    int (*read)();
    void (*write)(char);
    void (*seek)(long index, int mode);
}

```

콘솔용 입출력 드라이버에서는 이들 함수를 아래와 같이 정의하며, FILE 데이터 구조를 함수에 대한 주소와 함께 로드한다.

```c
#include "file.h"
void (*open)(char* name, int mode) {/*...*/}
void (*close)  {/*...*/};
int (*read)  {/*...*/}
void (*write)(char)  {/*...*/}
void (*seek)(long index, int mode)  {/*...*/}

struct FILE console = {open, close, read, write, seek};
```

이제 STDIN을 FILE*로 선언하면, STDIN은 콘솔 데이터 구조를 가리키므로, getchar()는 아래와 같은 방식으로 구현할 수 있다.

```c
extern struct FILE* STDIN;

int getchar() {
    return STDIN->read();
}
```

함수를 가리키는 포인터를 응용한 것이 다형성이라는 점을 말하고 있다.

1940년대 후반 폰 노이만(Von Neumann) 아키텍처가 처음 구현된 이후 프로그래머는 다형적 행위를 수행하기 위해 함수를 가리키는 포인터를 사용해 왔다. 따라서 OO가 새롭게 만든 것은 전혀 없다.

하지만 OO 언어는 다형성을 좀 더 안전하고 더욱 편리하게 사용할 수 있게 해준다.

함수에 대한 포인터를 직접 사용하여 다형적 행위를 만드는 이 방식은 문제가 있는데, 함수 포인터가 위험하다는 사실이다. 만약 프로그래머가 특정 관례를 지키지 못한다면 버그가 발생하고, 이러한 버그는 찾아내고 없애기가 지독히 힘들다.

## 다형성이 가진 힘

다형성이 가진 매력의 진가를 알아보기 위해 복사 프로그램 예제를 다시 살펴본다.

새로운 입출력 장치가 생긴다면 프로그램에는 어떤 변화가 생기는가? 새로운 장비에서도 복사 프로그램이 동작하도록 만들려면 어떻게 수정해야 하는가?

아무런 변경도 필요치 않다! 복사 프로그램 소스 코드는 입출력 드라이버의 소스 코드에 의존하지 않기 때문이다.

다시 말해 입출력 드라이버가 복사 프로그램의 플러그인(plugin)이 된것이다.

## 의존성 역전

다형성을 안전하고 편리하게 적용할 수 있는 메커니즘이 등장하기 전에는 main 함수가 고수준 함수를 호출하고, 고수준 함수는 다시 중간 수준 함수를 호출하며, 중간 함수는 다시 저수준 함수를 호출했다.

이러한 호출 트리에서는 의존성의 방향이 반드시 제어흐름(flow of control)을 따르게 된다.

![image](/assets/img/post/2022-05-01-PPPCleanArchitecture_ch5/2.jpg)
> 소스 코드 의존성 vs. 제어 흐름

제어흐름은 시스템의 행위에 따라 결정되며, 소스 코드 의존성은 제어흐름에 따라 결정된다.

하지만 다형성이 끼어들면 무언가 특별한 일이 일어난다.

![image](/assets/img/post/2022-05-01-PPPCleanArchitecture_ch5/3.jpg)
> 의존성 역전

ML1과 I 인터페이스 사이의 소스 코드 의존성(상속 관계)이 제어흐름과는 반대인 점을 주목해야 한다.

이는 의존성 역전(dependency inversion)이라고 부르며, 소프트웨어 아키텍트 관점에서 이러한 현상은 심오한 의미를 갖는다.

예를 들어 업무 규칙이 데이터베이스와 사용자 인터페이스(UI)에 의존하는 대신에, 시스템의 소스 코드 의존성을 반대로 배치하여 데이터베이스와 UI가 업무 규칙에 의존하게 만들 수 있다.

![image](/assets/img/post/2022-05-01-PPPCleanArchitecture_ch5/4.jpg)
> 데이터베이스와 사용자 인터페이스가 업무 규칙에 의존한다.

즉, UI와 데이터베이스가 업무 규칙의 플러그인이 된다는 뜻이다. 다시 말해 업무 규칙의 소스 코드에서는 UI나 데이터베이스를 호출하지 않는다.

따라서 업무 규칙을 UI와 데이터베이스와 독립적으로 배포할 수 있다. UI나 데이터베이스에서 발생한 변경사항은 업무 규칙에 일절 영향을 미치지 않는다.

즉, 이들 컴포넌트는 개별적이며 독립적으로 배포 가능하다.

다시 말해 특정 컴포넌트의 소스코드가 변경되면, 해당 코드가 포함된 컴포넌트만 다시 배포하면 된다. 이것이 `배포 독립성(independent deployability)`다.

시스템의 모듈을 독립적으로 배포할 수 있게 되면, 서로 다른 팀에서 각 모듈을 독립적으로 개발할 수 있다. 그리고 이것이 `개발 독립성(independent developability)`이다


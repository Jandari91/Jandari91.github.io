---
title: 9장. LSP, 리스코프 치환 원칙
author: Jandari
date: 2022-05-05 23:30:00 +09:00
categories: [DDD, 클린 아키텍처 소프트웨어 구조와 설계의 원칙]
tags: [DDD, CleanArchitecture]
math: true
mermaid: true
---

# 소개

![image](/assets/img/post/2022-05-05-PPPCleanArchitecture_ch9/1.jpg)

클린아키텍처: 소프트웨어 구조와 설계의 원칙 책을 읽고 정리하며 소감을 적는 포스트입니다.

# LSP : 리스코프 치환 원칙

1988년 바바라 리스코프(Barbara Liskov)는 하위 타입(subtype)을 아래와 같이 정의 했다.

```
S 타입의 객체 o1에 각각에 대응하는 T 타입 객체 o2가 있고, T 타입을 이용해서 정의한 모든 프로그램 P에서 o2의 자리에 o1을 치환하더라도 P의 행위가 변하지 않는다면, S는 T의 하위 타입이다.
```

## 상송을 사용하도록 가이드하기

아래의 그림과 같이 License라는 클래스가 있다고 했을 때, License는 PersonalLicense와 BusinessLicense라는 두 가지 '하위 타입'이 존재한다.

![image](/assets/img/post/2022-05-05-PPPCleanArchitecture_ch9/2.jpg)
> License와 파생 클래스는 LSP를 준수한다.

이 설계는 LSP를 준수하는데 Bulling 애플리케이션의 행위가 License 하위 타입 중 무엇을 사용하는지에 전혀 의존하지 않기 때문이다. 이들 하위 타입은 모두 License 타입을 치환할 수 있다.

## 정사각형/직사각형 문제

LSP를 위반하는 전형적인 문제로는 유명한 정사각형/직사각형 문제가 있다.

![image](/assets/img/post/2022-05-05-PPPCleanArchitecture_ch9/3.jpg)
> 악명 높은 정사각형/직사각형 문제

이 예제에서 Square는 Rectangle의 하위 타입으로는 적합하지 않다.

Rectangle의 높이와 너비는 서로 독립적으로 변경될 수 있는 반면, Square의 높이와 너비는 반드시 함께 변경되기 때문이다.

```
Rectangle r = ...
r.setW(5);
r.setH(2);
assert(r.area() == 10)
```

... 코드에서 Square를 생성한다면 assert문은 실패하게 된다.

if문 등을 이용해서 Square인지 검사하는 메커니즘을 User에 추가해야 하기 때문에 User는 행위가 사용하는 타입에 의존하게 되므로, 결국 타입을 서로 치환할 수 없게 된다.

## LSP 아키텍처

잘 정의 된 인터페이스와 그 인터페이스의 구현체끼리의 상호 치환 가능성에 기대는 사용자들이 존재하기 때문에 많은 상황에 LSP를 적용 할 수 있다.

아키텍처 관점에서 LSP를 이해하는 최선의 방법은 이 원칙을 어겼을 때 시스템 아키텍처에서 무슨 일이 일어나는지 관찰하는 것이다.

## LSP 위배 사례

LSP를 위배하여 개발을 하게 된다면 고객의 요구사항에 대해 예외 사항을 처리하는 로직을 추가해야 한다.

만약 택시 파견 서비스를 통합하는 애플리케이션을 만들고 있다고 해본다.

퍼플캡이라는 회사에서 아래와 같이 URL을 통해 파견 정보를 호출한다.

```
purplecab.com/driver/Bob
	/pickupAddress/24 Maple St.
    /pickupTime/153
    /destination/ORD
```

또 애크미라는 택시업체가 있고 그 업체의 프로그래머들은 `destination`을 `dest`로 축약하여 사용하고 있다.

그런데 이 둘 회사가 합쳐진다고 한다면 모든 파견 명령어에 아래와 같이 if 문장을 추가해야 한다.
```
if (driver.getDispatchUri().startsWith("acme.com").....
```

실력 있는 아키텍트라면 당연히 시스템을 이런 식으로 구성하는 것을 용납하지 않는다.

아키텍트는 이 같은 버그로 부터 시스템을 격리해야 한다.

이때 파견 URI를 키로 사용하는 설정용 데이터베이스를 이용하는 파견 명령 생성 모듈을 만들어야 할 수도 있다.

|URI|	Dispatch Format|
|-|-|
|`Acme.com`|/pickupAddress/%s/pickupTime/%s/dest/%s|
|`*.*`| /pickupAddress/%s/pickupTime/%s/destination/%s|

또한 아키텍트는 REST 서비스들의 인터페이스가 서로 치환 가능하지 않다는 사실을 처리하는 중요하고 복잡한 매커니즘을 추가해야 한다.

## 결론

LSP는 아키텍처 수준까지 확장할 수 있고, 반드시 확장해야만 한다.

치환 가능성을 조금이라도 위배하면 시스템 아키텍처가 오염되어 상당량의 별도 메커니즘ㅇ르 추가해야 할 수 있기 때문이다.

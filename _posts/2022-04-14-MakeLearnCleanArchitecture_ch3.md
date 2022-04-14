---
title: 코드 구성하기
author: Jandari
date: 2022-04-14 22:20:00 +09:00
categories: [DDD, 만들면서 배우는 클린 아키텍처]
tags: [DDD, CleanArchitecture]
math: true
mermaid: true
---
## 소개

![image](/assets/img/post/2022-04-14-MakeLearnCleanArchitecture_ch3/1.jpg)

만들면서 배우는 클린 아키텍처 책을 읽고 정리하며 소감을 적는 포스트입니다.

자바로 이뤄어진 코드는 C#으로 변경합니다.

## 코드 구성하기

코드를 보는 것만으로도 아키텍처가 파악 된다면 굉장히 좋을 것입니다.

이번 포스트는 BuckPal 예제 코드를 구조화 하기 위해 육각형 아키텍처를 직접 레이아웃을 구성하도록 하겠습니다.

아래는 사용자가 본인의 계좌에서 다른 계좌로 돈을 송금할 수 있는 `송금하기` 유스케이스를 살펴보겠습니다.

### 계층으로 구성하기

코드를 구조화하는 첫 번째 접근법은 계층을 이용하는 것입니다.

```
buckpal
├─────── domain
│         ├──── Account
│         ├──── Activity
│         ├──── IAccountRepository
│         └──── AccountService
│
├─────── persistence
│         └──── AccountRepository
│
└─────── web
          └──── AccountController
```

웹 계층(web), 도메인 계층(domain), 영속성 계층(persistence)로 구분하였습니다.

domain 패키지에 IAccountRepository 인터페이스를 추가하고, persistence 패키지에 AccountRepository 구현체를 둠으로써 의존성을 역전시켰습니다.

그러나 적어도 세 가지 이유로 이 패키지 구조는 최적의 구조가 아닙니다.

#### 첫번째

애플리케이션 기능 조각(functional slice)이나 특성(feature)을 구분 짓는 패키지 경계가 없습니다.

추가적인 구조가 없다면, 아주 빠르게 서로 연관되지 않은 기능들끼리 예상하지 못한 부수효과를 일으킬 수 있는 클래스들의 엉망진창 묶음으로 변모할 가능성이 큽니다.

#### 두번째

애플리케이션이 어떤 유스케이스들을 제공하는지 파악 할 수 없습니다.

AccountService와 AccountController가 어떤 유스케이스를 가지고 있는지 소스를 보기 전에는 알수 없습니다.

#### 세번째
패키지 구조를 통해 목표로하는 아키텍처를 파악할 수 없습니다.

육각형 아키텍처를 따랐다고 하지만 어떤 기능이 웹 어댑터에서 호출되는지, 영속성 어댑터가 도메인 계층에 어떤 기능을 제공하는지 한눈에 알아볼 수 없다.


### 기능으로 구성하기

`계층으로 구성하기` 방법의 몇 가지 문제를 해결해보겠습니다.

```
buckpal
└─────── account
          ├──── Account
          ├──── AccountController
          ├──── IAccountRepository
          ├──── AccountRepository
          └──── SendMoneyService
```

가장 본질적인 변경은 계좌와 관련된 모든 코드를 최상위의 account 패키지에 넣었다는 점이다 계층 패키지들도 없앴다.

AccountService의 책임을 좁히기 위해 SendMoneyService로 클래스명을 바꿨다. 이제 `송금하기` 유스케이스를 구현한 코드는 클래스명만으로도 찾을 수 있게 됐다.

애플리케이션의 기능을 코드를 통해 볼 수 있게 만드는 것을 가리켜 로버트 마틴이 `소리치는 아키텍처(screaming architecture)`라도 명명한 바 있다.

그러나 기능에 의한 패키징 방식은 사실 계층에 의한 패키징 방식보다 아키텍처의 `가시성을 훨씬 더 떨어뜨린다.`

어댑터를 알아볼 수 없고, 심지어 도메인 코드와 영속성 코드 간의 의존성을 역전시켜서 SendMoneyService가 AccountRepository 인테페이스만 알고 있고 구현체는 알 수 없도록 했음에도 도메인 코드가 실수로 영속성 코드에 의존하는 것을 막을 수 없다.

### 아키텍처적으로 표현력 있는 패키지 구조

육각형 아키텍처에서 구조적으로 핵심적인 요소는 엔티티, 유스케이스, 인커밍/아웃고잉 포트, 인커밍/아웃고잉(혹은 주도하거나 주도되는) 어댑터다.

```
buckpal
└─────── account
          ├──── adapter
          │      ├──── in
          │      │     └──── web
          │      │           └──── AccountController
          │      │
          │      └──── out
          │            └──── persistence
          │                  ├──── AccountPersistenceAdapter
          │                  └──── SpringDataAccountRepository
          │       
          ├──── domain
          │      ├──── Account
          │      └──── Activity
          │
          └──── application
                 ├──── SendMoneyService
                 └──── port
                        ├──── in
                        │     └──── SendMoneyUseCase
                        └──── out
                              ├──── LoadAccountPort
                              └──── UpdateAccountStatePort
                        
```

구조의 각 요소들은 패키지 하나씩에 직접 매핑된다.

* domain
  * 도메인 모델 (Account, Activity)
* application
  * 도메인 모델을 둘러싼 서비스 계층 (SendMoneyService)
  * 인커밍 포트 인터페이스 (SendMoneyUseCase)
  * 아웃고잉 포트 인터페이스 (LoadAccountPort, UpdateAccountStatePort)
* adapter
  * 어플리케이션 계층의 인커밍 포트를 호출하는 인커밍 어댑터 (AccountController)
  * 어플리케이션 계층의 아웃고잉 포트에 대한 구현을 제공하는 아웃고잉 어댑터 (AccountPersistenceAdapter, SpringDataAccountRepository)

이러한 패키지 구조는 다양한 장점이 있는데 첫번째로 `아키텍처-코드 갭(architecture-code gap)` 혹은 `모델-코드 갭(model-code gap)`을 효과적으로 다룰 수 있다.

또한 패키지간의 접근을 제어 할 수 있다.

adpater 패키지의 모든 클래스는 application 패키지 내의 포트 인터페이스를 통해 외부에서 호출되기 때문에 dapter는 모두 package-private 접근 수준으로 둬도 된다.

그러므로 어플리케이션 계층에서 어댑터로 향하는 우발적 의존성은 없어진다.

하지만 application 패키지와 domain 패키지늬 일부 클래스는 의도적으로 어댑터에서 접근 가능해야 하므로 public으로 지정해야 한다.

또한 도메인 클래스는 서비스, 그리고 잠재적으로 어댑터에서도 접근 가능하도록 public이여야 하며 서비스는 인커밍 포트 인터페이스 뒤에 숨을 수 있기 때문에 public일 필요는 없다.

### 의존성 주입의 역할

클린 아키텍처의 본질적인 요건은 `어플리케이션이 인커밍/아웃고잉 어댑터에 의존성을 갖지 않아야 한다.`

육각형 아키텍처에서 는 애플리케이션 계층에 인터페이스를 만들고 어댑터에 해당 인터페이스 구현체를 두는데 여기서 인터페이스가 포트 역할을 하게 된다.

그런데 인터페이스를 구현한 객체를 누가 애플리케이션에 제공해야 할까?? 애플리케이션 계층에 어댑터에 대한 의존성을 추가하고 싶지 않을 수 있다.

이럴 때 의존성 주입을 활용 할 수 있다. 모든 계층에 의존성을 가진 중립적인 컴포넌트를 하나 도입하는 것이다.

이 컴포넌트는 아키텍처를 구성하는 대부분의 클래스를 초기화하는 역할을 한다.

![image](/assets/img/post/2022-04-14-MakeLearnCleanArchitecture_ch3/2.jpg)

위 그림에서 중립적인 의존성 주입 컴포넌트는 AccountController, SendMoneyService, AccountPersistenceAdapter 클래스의 인스턴스를 만들어 주입하게 된다.

그렇게 되면 Account-Contoller는 SendMoneyUseCase 인터페이스만 알면 되기 때문에 자신이 어떤 구현체의 객체를 가지게 되는지 몰라도 된다.

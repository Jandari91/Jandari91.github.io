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


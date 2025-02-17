---
title: 8장. OCP, 개방-폐쇄 원칙
author: Jandari
date: 2022-05-05 21:00:00 +09:00
categories: [DDD, 클린 아키텍처 소프트웨어 구조와 설계의 원칙, 3부 설계 원칙]
tags: [DDD, CleanArchitecture]
math: true
mermaid: true
---

# 소개

![image](/assets/img/post/2022-05-05-PPPCleanArchitecture_ch8/1.jpg)

클린아키텍처: 소프트웨어 구조와 설계의 원칙 책을 읽고 정리하며 소감을 적는 포스트입니다.

# OCP: 개방-폐쇄 원칙

개방-폐쇄 원칙(OCP)이라는 용어는 1988년에 버트란트 마이어(Bertrand Meyer)가 만들었는데, 다음과 같다.

`소프트웨어 개체(artifact)는 확장에는 열려 있어야 하고, 변경에는 닫혀 있어야 한다.`

다시 말해 소프트웨어 개체의 행위는 확장할 수 있어야 하지만, 이때 개체를 변경해서는 안 된다.

소프트웨어 아키텍처를 공부하는 가장 근본적인 이유이기도 하다. 만약 요구사항을 살짝 확장하는 데 소프트웨어를 엄청나게 수정해야 한다면 그 소프트웨어 시스템을 설계한 아키텍트는 엄청난 실패에 맞닥뜨린 것이다.

대다수 개발자들은 OCP가 클래스와 모듈을 설계할 때 도움되는 원칙이라고 알고 있지만 아키텍처 컴포넌트 수준에서 OCP를 고려할 때 훨씬 중요한 의미를 가진다.

## 사고 실행

제무제표를 웹 페이지로 보여주는 시스템이 있다고 가정한다.

웹 페이지에 표시되는 데이터는 스크롤할 수 있으며, 음수는 빨간색으로 출력한다.

하지만 이해관계자가 동일한 정보를 보고서 형태로 변환해서 흑백 프린터로 출력해 달라고 요청했다고 해보자.

* 페이지 번호가 있어야 한다.
* 페이지 마다 적절한 머리글과 바닥글이 있어야 한다.
* 표의 각 열에는 레이블이 있어야 한다.
* 음수는 괄호로 감싸야 한다.

소프트웨어 악키텍처가 훌륭하다면 변경되는 코드의 양이 가능한 한 최소화 될 것이다. 이상적인 변경량은 0이다.

서로 다른 목적으로 변경되는 요소를 적절하게 분리하고(단일 책임 원칙SRP), 이들 요소 사이의 의존성을 체계화함으로써(의존성 역전 원칙DIP) 변경량을 최소화 할 수 잇다.

![image](/assets/img/post/2022-05-05-PPPCleanArchitecture_ch8/2.jpg)
> SRP 적용하기

여기서 가장 중요한 영감은 보고서 생성이 두 개의 책임으로 분리된다는 사실이다.

이 처럼 웹과 프린터로 분리가 되어 있다면 두 책임 중 하나에서 변경이 발생하더라도 다른 하나는 변경되지 않도록 소스 코드 의존성도 확실히 조직화해야 한다.

또한, 새로운 조직화한 구조에서는 행위가 확장 될 때 변경이 발생하지 않음을 보장해야 한다.

![image](/assets/img/post/2022-05-05-PPPCleanArchitecture_ch8/3.jpg)
> 처리 과정을 클래스 단위로 분할하고, 클래스는 컴포넌트 단위로 분리한다.

위 그림에서 좌측 상단의 컴포넌트는 Controller이고 우측 상단은 Interractor 컴포넌트를, 우측 하단에서는 Database 컴포넌트를 볼 수 있다. 좌측 하단에는 Presenter와 View를 담당하는 네 가지 컴포넌트가 위치한다.

여기서 주목할 점은 모든 의존성이 소스 코드 의존성을 나타낸다는 사실이다. FinancialDataMapper는 구현 관계를 통해 FinancialDataGateway를 알고 있지만, FinancialDataGateway는 FinancialDataMapper에 대해 아무것도 알지 못한다.

![image](/assets/img/post/2022-05-05-PPPCleanArchitecture_ch8/4.jpg)
> 컴포넌트 관계는 단방향으로만 이루어진다.

모든 컴포넌트 관계는 단방향으로 이루어진다. 이들 화살표는 변경으로 부터 보호하려는 컴포넌트를 향햐도록 그려진다.

A 컴포넌트에서 발생한 변경으로 부터 B 컴포넌트를 보호하려면 반드시 A 컴포넌트가 B 컴포넌트에 의존해야 한다.

* Presenter에서 발생한 변경으로부터 Controller를 보호하고자 한다.
* View에서 발생한 변경으로부터 Presenter를 보호하고자 한다.
* Interactor는 다른 모든 것에서 발생한 변경으로부터 보호하고자 한다.

Interator는 OCP를 가장 잘 준수할 수 있는 곳에 위치한다. Database, Controller, Presenter, View에서 발생한 어떤 변경도 Interator에 영향을 주지 않는다.

Interactor는 업무 규칙을 포함하기 때문에 애플리케이션에서 가장 높은 수준의 정책을 포함해야 한다.

아키텍트는 기능이 어떻게(how), 왜(why), 언제(when) 발생하는지에 따라서 기능을 분리하고, 분리한 기능을 컴포넌트의 계층구조로 조직화한다.

컴포넌트 계층구조를 이와 같이 조직화하면 저수준 컴포넌트에서 발생한 변경으로부터 고수준 컴포넌트를 보호 할 수 있다.

## 방향성 제어

![image](/assets/img/post/2022-05-05-PPPCleanArchitecture_ch8/3.jpg)

위 다이어그램은 컴포넌트 간 의존성이 제대로 된 방향으로 향하고 있음을 보여준다.

예를 들어 FinancialDataGateway 인터페이스는 FinancialReportGenerator와 FinancialDataMapper 사이에 위치하는데, 이는 의존성을 역전시키기 위해서다.

## 정보은닉

![image](/assets/img/post/2022-05-05-PPPCleanArchitecture_ch8/3.jpg)

FinancialReportRequester 인터페이스는 방향성 제어와는 다른 목적을 가진다. 이 인터페이스는 FinancialReportController가 Interactor 내부에 대해 너무 많이 알지 못하도록 막기 위해서 존재한다.

만약 이 인터페이스가 없다면 Controller는 FinancialEntities에 대해 추이 종속성(transitive dependency)를 가지게 된다.

추이 종속성을 가지게 되면, 소프트웨어 엔티티는 `자신이 직접 사용하지 않는 요소에는 절대로 의존해서는 안 된다.`는 소프트웨어 원칙을 위반하게 된다.

## 결론

OCP는 시스템의 아키텍처가 떠받치는 원동력 중 하나다. OCP의 목표는 시스템을 확장하기 쉬운 동시에 변경으로 인해 시스템이 너무 많은 영향을 받지 않도록 하는 데 있다.

이러한 목표를 달성하려면 시스템을 컴포넌트 단위로 분리하고, 저수준 컴포넌트에서 발생한 변경으로부터 고 수준 컴포넌트를 보호 할 수 있는 형태의 의존성 계층 구조가 만들어지도록 해야 한다.

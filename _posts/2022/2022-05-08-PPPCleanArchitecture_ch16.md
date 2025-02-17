---
title: 16장. 독립성
author: Jandari
date: 2022-05-08 21:00:00 +09:00
categories: [DDD, 클린 아키텍처 소프트웨어 구조와 설계의 원칙, 5부 아키텍처]
tags: [DDD, CleanArchitecture]
math: true
mermaid: true
---

# 소개

![image](/assets/img/post/2022-05-08-PPPCleanArchitecture_ch16/1.jpg)

클린아키텍처: 소프트웨어 구조와 설계의 원칙 책을 읽고 정리하며 소감을 적는 포스트입니다.

# 독립성

좋은 아키텍처는 다음을 지원해야 한다.

* 시스템의 유스케이스
* 시스템의 운영
* 시스템의 개발
* 시스템의 배포

## 유스케이스

첫 번째 주요 항목인 유스케이스의 경우, 시스템의 아키텍처는 시스템의 의도를 지원해야 한다는 뜻이다.

아키텍트의 최우선 관심사는 유스케이스이며, 아키텍처에서도 유스케이스가 최우선이다. 아키텍처는 반드시 유스케이스를 지원해야 한다.

좋은 아키텍처가 행위를 지원하기 위해 할 수 있는  중에서 가장 중요한 사항은 행위를 명확히 하고 외부로 드러내며, 이를 통해 시스템이 지닌 의도를 아키텍처 수준에서 알아볼 수 있게 만드는 것이다.

장바구니 애플리케이션이 좋은 아키텍처를 갖춘다면 이 애플리케이션은 장바구니 애플리케이션처럼 보일 것이다. 해당 시스템의 유스케이스는 시스템 구조 자체에서 한눈 에 들어날 것이다.

이들 행위는 일급 요소(first-class)이며 시스템의 최상위 수준에서 알아볼 수 있으므로, 개발자가 일일이 찾아 헤매지 않아도 된다.

## 운영

시스템의 운영 지원 관점에서 볼 때 아키텍처는 더 실질적이며 덜 피상적인 역할을 맡는다. 아키텍처는 요구와 관련 된 각 유스케이스에 걸맞은 처리량와 응답시간을 보장해야 한다.

이러한 형태를 지원한다는 말은 시스템에 따라 다양한 의미를 지닌다. 어떤 시스템에서는 시스템의 처리 요소를 일련의 작은 서비스들로 배열하여 서로 다른 많은 서버에서 병렬로 실행할 수 있게 만들어야 함을 의미한다.

만약 시스템이 단일체(monolith)로 작성되어 모노리틱 구조를 갖는다면, 다중 프로세스, 다중 스레드, 또는 마이크로서비스 형태가 필요해질 때 개선하기 어렵다.

그에 비해 아키텍처에서 각 컴포넌트를 적걸히 격리하고 유지하고 컴포넌트 간 통신 방싱을 특정 형태로 제한하지 않는다면, 시간이 지나 운영에 필요한 요구사항이 바뀌더라도 스레드, 프로세스, 서비스로 구성된 기술 스펙트럼 사이를 전환하는 일이 훨씬 쉬워질 것이다.

## 개발

아키텍처는 개발환경을 지원하는 데 있어 핵심적인 역할을 수행한다.

콘웨이(Conway)의 법칙이 작용하는 지점이 바로 여기이다. 코누에이의 법칙은 다음과 같다.

`시스템을 설계하는 조직이라면 어디든지 그 조직의 의사소통 구조와 동일한 구조의 설계를 만들어 낼 것이다.`

많은 팀으로 구성되며 관심사가 다양한 조직에서 어떤 시스템을 개발해야 한다면, 각 팀이 독립적으로 행동하기 편한 아키텍처를 반드시 확보하여 개발하는 동안 팀들이 서로를 방해하지 않도록 해야한다.

이러한 아키텍처를 만들려면 잘 격리되어 독립적으로 개발 가능한 컴포넌트 단위로 시스템을 분할 할 수 있어야 한다.

그래야만 이들 컴포넌트를 독립적으로 작업할 수 있는 팀에 할당할 수 있다.

## 배포

아키텍처는 배포 용이성을 결정하는 데 중요한 역할을 한다. 이때 목표는 `즉각적인 배포(immediate deployment)`다.

좋은 아키텍처는 수십 개의 작은 설정 스크립트나 속성 파일을 약간씩 수정하는 방식을 사용하지 않는다.

좋은 아키텍처라면 시스템이 빌드된 후 즉각 배포할 수 있도록 지원해야 한다.
'다시 말하지만, 이러한 아키텍처를 만들려면 시스템을 컴포넌트 단위로 적절하게 분할하고 격리시켜야 한다.

## 선태사항 열어놓기

좋은 아키텍처는 컴포넌트 구조와 관련된 이 관심사들 사이에서 균형을 맞추고, 각 관심사 모두를 만족시킨다.

말은 쉽지만 현실에서는 이러한 균형을 잡기가 매우 러렵다. 대부분의 경우 우리는 모든 유스케이스를 알 수는 없으며, 운영하는 데 따르는 제약사항, 팀 구조, 배포 요구사항도 알지 못하기 때문이다.

더 심각한 문제는 이러한 사항들을 알고 있더라도, 시스템이 생명주기의 단계를 하나씩 거쳐감에 따라 이 사항들도 반드시 변해간다는 사실이다.

하지만 몇몇 아키텍처 원칙을 구현하는 비용은 비교적 싸지 않으며, 관심사들 사이에서 균형을 잡는데 도움이 된다. 이들 원칙은 시스템을 제대로 격리된 컴포넌트 단위로 분할 할 때 도움이 되며, 이를 통해 선택사항을 가능한 한 많이, 그리고 가능한한 오랫동안 열어 둘 수 있게 해준다.

좋은 아키텍처는 선택사항을 열어 둠으로써, 향후 시스템에 변경이 필요할 때 어떤 방향으로든 쉽게 변경할 수 있도록 한다.

## 계층 결합 분리

아키텍트는 필요한 모든 유스케이스를 지원할 수 있는 시스템 구조를 원하지만, 유스케이스 전부를 알지는 못한다.

하지만 아키텍트는 시스템의 기본적인 의도는 분명히 알고 있다. 그 시스템이 장바구니 시스템인지, 자재 명세서 시스템인지 안다는 뜻이다.

따라서 아키텍트는 단일 책임 원칙과 공통 폐쇄 원칙을 적용하여, 그 의도의 맥락에 따라서 다른 이유로 변경되는 것들은 분리하고, 동일한 이유로 변경되는 것들은 묶는다.

사용자 인터페이스가 변경되는 이유는 업무 규칙과는 아무런 관련이 없다. 만약 유스케이스가 두 가지 요소를 모두 포함한다면, 유스케이스에서 UI 부분와 업무 규칙 부분을 서로 분리하고자 할 것이다.

업무 규칙은 그 자체가 애플리케이션과 밀접한 관련이 있거나, 혹은 더 범용적일 수 있다. 예를 들어 입력 필드 유효성 검사는 애플리케이션 자체와 연관 된 업무 규칙이다.

반대로 계좌의 이자 계산 같은 경우는 업무 도메인에 더 밀접하게 연관된 엄무 규칙이다.

이들 규칙은 서로 분리하고, 독립적으로 변경할 수 있도록 만들어야만 한다.

데이터베이스, 쿼리 언어, 스키마 조차도 업무 규칙이나 UI와는 아무런 관련이 없다. 결론적으로 아키텍트는 이들을 시스템의 나머지 부분으로부터 분리하여 독립적으로 변경할 수 있도록 해야만 한다.

이제 우리는 서로 결합되지 않은 수평적인 계층으로 분리하는 방법을 알게 되었다.

이러한 계층의 예로는 UI, 애플리케이션에 특화된 업무 규칙, 애플리케이션과는 독립적인 업무 규칙, 데이터베이스 등을 들 수 있다.

## 유스케이스 결합 분리

서로 다른 이유로 변경되는 것에는 또 무엇이 있을까? 바로 유스케이스 그 자체가 있다!

주문 입력시스템에서 주문을 추가하는 유스케이스는 주문을 삭제하는 유스케이스와는 틀림없이 다른 속도로, 그리고 다른 이유로 변경된다. 유스케이스는 시스템을 분할하는 매우 자연스러운 방법이다.

이와 같이 결합을 분리하려면 주문 추가 유스케이스의 UI와 주문 삭제 유스케이스의 UI를 분리해야 한다. 이런식으로 시스템의 맨 아래 계층까지 수직으로 내려가며 유스케이스들이 각 계층에서 서로 겹치지 않게 한다.

여기에서 패턴을 볼 수 있다. 시스템에서 서로 다른 이유로 변경되는 요소들의 결합을 분리하면 기존 요소에 지장을 주지 않고도 새로운 유스케이스를 계속해서 추가할 수 있다.

유스케이스가 UI와 데이터베이스의 서로 다른 관점(aspect)를 사용하게 되면, 새로운 유스케이스를 추가하더라도 기존 유스케이스에 영향을 주는 일은 거의 없을 것이다.

## 결합 분리 모드

이렇게 결합을 분리하면 두 번째 항목인 운영 관점에서 어떤 의미가 있는지 살펴본다.

![image](/assets/img/post/2022-05-08-PPPCleanArchitecture_ch16/2.jpg)
> 수직, 수평 계층 분할 예시

UI와 데이터베이스가 업무 규칙과 분리되어 있다면, UI와 데이터베이스는 업무 규칙과는 다른 서버에서 실행될 수 있다. 높은 대역폭을 요구사흔ㄴ 유스케이스는 여러 서버로 복제하여 실행할 수 있따.

분리된 컴포넌트를 서로 다른 서버에서 실행해야 하는 상황이라면, 이들 컴포넌트가 단일 프로세서의 동일한 주소 공간에 함께 상주하는 형태가 만들어져서는 안 된다.

분리된 컴포넌트는 반드시 독립된 서비스가 되어야 하고, 일정의 네트워크를 통해 서로 통신해야 한다.

많은 아키텍트가 이러한 컴포넌트를 `서비스(service)` 또는 `마이크로서비스(micro-service)`라고 하는데, 실제로 서비스에 기반한 아키텍처를 흔히들 `서비스 지향 아키텍처(service-oriented architecture)`라고 부른다.

여기서 이야기 하고자 하는 핵심은 우리는 때때로 컴포넌트를 서비스 수준까지도 분리해야 한다는 것이다.

## 개발 독립성

컴포넌트가 완전히 분리되면 팀 사이의 간섭은 줄어든다. 업무 규칙이 UI를 알지 못하면 UI에 중점을 둔 팀은 업무 규칙에 중점을 둔 팀에 그다지 영향을 줄 수 없다.

기능팀, 컴포넌트 팀, 계층 팀, 혹은 또 다른 형태의 팀이라도, 계층과 유스케이스의 결합이 분리되는 한 시스템의 아키텍처는 그 팀 구조를 뒷받침해 줄 것이다.

## 배포 독립성

유스케이스와 계층의 결합이 분리되면 배포 측면에서도 고도의 유연성이 생긴다. 실제로 결합을 제대로 분리했다면 운영 중인 시스템에서도 계층과 유스케이스를 교체(hot-swap) 할 수 있다.

## 중복

소프트웨엉에서 중복은 일반적으로 나쁜 것이다. 우리는 중복된 코드를 좋아하지 않는다. 코드가 진짜로 중복되었다면, 우리는 전문가로서의 명예를 걸고 중복을 줄이거나 제거해야 한다.

하지만 중복에도 종류가 있다. 그 중하나는 진짜 중복이다. 이 경우는 인스턴스가 변경되면, 동일한 변경을 그 인스턴스의 모든 복사본에 반드시 적용해야 한다.

또 다른 중복은 거짓된 또는 우발적 중복이다. 중복으로 보이지만 두 코드 영역이 각자의 경로로 발전한다면, 즉 서로 다른 속도와 다른 이유로 변경된다면 이 두 코드는 진짜 중복이 아니다.

유스케이스를 수직으로 분리할 때 이러한 문제와 마주칠 테고, 이들 유스케이를 통합하고 싶다는 유혹을 받게 될 것이다. 하지만 조심해야 한다. 중복이 진짜 중복인지 확인해야 한다.

## 결합 분리 모드(다시)

다시 결합 분리 모드로 돌아간다. 계층과 유스케이스의 결합을 분리하는 방법은 다양하다. 

### 소스 수준 분리 모드 

소스 코드 모듈 사이의 의존성을 제어할 수 있다. 이를 통해 하나의 모듈이 변하더라도 다른 모듈을 변경하거나 재컴파일하지 않도록 만들 수 있다(예, 루비 Gem).

이 모드에서는 모든 컴포넌트가 같은 주소 공간에서 실행되고 서로 통신할 때는 간단한 함수 호출을 사용한다.

이러한 구조를 모노리틱 구조라고 부른다.

### 배포 수준 분리 모드

jar 파일, DLL, 공유 라이브러리와 같은 배포 가능한 단위들 사이의 의존성을 제어할 수 있다. 이를 통해 한 모듈의 소스 코드가 변하더라도 다른 모듈을 재빌드하거나 재배포하지 않도록 만들 수 있다.

이 모드의 중요한 특징은 결합이 분리된 컴포넌트가 jar 파일, Gem 파일, DLL과 같이 독립적으로 배포할 수 있는 단위로 분할되어 있다는 점이다.

### 서비스 수준 분리 모드

의존하는 수준을 데이터 구조 단위 까지 낮출 수 있고, 순전히 네트워크 패킷을 통해서만 통신하도록 만들 수 있다.

이를 통해 모든 실행 가능한 단위는 소스와 바이너리 변경에 대해 서로 완전히 독립적이게 된다.

<br/>
<br/>

시간이 흐르면 시스템에서 운영 요구사항은 감소할 수 있다. 한때는 결합을 서비스 수준까지 분리해야 했던 것들이 이제 배포 수준, 심지어 소스 수준의 결합 분리만으로 충분할 수도 있다.

좋은 아키텍처는 시스템이 모노리틱 구조로 태어나서 단일 파일로 배포되더라도, 이후에는 독립적으로 배포 가능한 단위들의 집합으로 성정하고, 또 독립적인 서비스나 마이크로서비스 수준까지 성정할 수 있도록 만들어져야한다.

또한 좋은 아키텍처라면 나중에 상황이 바뀌었을 때 이 진행 방향을 거꾸로 돌려 원래 형태인 모노리틱 구조로 되돌리 수도 있어야 한다.

좋은 아키텍처는 이러한 변경으로부터 소스 코드 대부분을 보호한다. 좋은 아키텍처는 결합 분리 모드를 선택사항으로 남겨두어서 배포 규모에 따라 가장 적합한 모드를 선택해 사용할 수 있게 만들어 준다.

## 결론

물론 이렇게 하기느 ㄴ까다롭다. 그리고 결합 분리 모드를 변경하기가 설정 값하나 바꾸듯 쉬워야 한다는 뜻이 아니다.

시스템의 결합 분리 모드는 시간이 지나면서 바뀌기 쉬우며, 뛰어나 아키텍트라면 이러한 변경을 예층하여 큰 무리 없이 반영할 수 있도록 만들어야 한다는 점이다.

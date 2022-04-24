---
title: 10장. 아키텍처 경계 강제하기
author: Jandari
date: 2022-04-23 11:50:00 +09:00
categories: [DDD, 만들면서 배우는 클린 아키텍처]
tags: [DDD, CleanArchitecture]
math: true
mermaid: true
---

## 소개

![image](/assets/img/post/2022-04-23-MakeLearnCleanArchitecture_ch10/1.jpg)

만들면서 배우는 클린 아키텍처 책을 읽고 정리하며 소감을 적는 포스트입니다.

## 아키텍처 경계 강제하기

일정 규모 이상의 모든 프로젝트에서는 시간이 지나면서 아키텍처가 서서히 무너지게 된다.

계층 간의 경계가 약화되고, 코드는 점점 더 테스트하기 어려워지고, 새로운 기능을 구현하는 데 점점 더 많은 시간이 든다.

### 경계와 의존성

![image](/assets/img/post/2022-04-23-MakeLearnCleanArchitecture_ch10/2.jpg)
> 아키텍처 경계를 강제한다는 것은 의존성이 올바른 방향을 향하도록 강제하는 것을 의미한다. 아키텍처에서 허용되지 않은 의존성을 점선 화살표로 표시했다.

위 그림은 육각형 아키텍처의 요소들이 클린 아키텍처 방식과 유사항 4개의 계층에 어떻게 흩어져 있는지 보여준다.


가장 안쪽의 계층에는 도메인 엔티티가 있고 애플리케이션 계층은 애플리케이션 서비스 안에 유스케이스를 구현하기 위해 도메인 엔티티에 접근한다.

어댑터는 인커밍 포트를 통해 서비스에 접근하고, 반대로 서비스는 아웃고잉 포트를 통해 어댑터에 접근한다. 마지막으로 설정 계층은 어댑터와 서비스 객체를 생성할 팩터리를 포함하고 있고, 의존성 주입 매커니즘을 제공한다.

### 접근 제한자

경계를 강제하기 위해 자바에서 제공하는 가장 기본적인 도구인 접근 제한자(visibility modifier)부터 시작한다.

대부분의 사람들은 public, protected, private 제한자만 알고 있고, 거의 대부분이 package-private(혹은, default) 제한자를 모른다.

그럼 왜 `package-private` 제한자가 중요할까?? 바로 클래스들을 응집적인 `모듈`로 만들어 주기 때문이다.

이런 모듈 내에 있는 클래스들은 서로 접근 가능하지만, 패키지 바깥에서는 접근 할 수 없다. 그럼 모듈의 진입점으로 활용될 클래스들만 골라서 public으로 만들면 된다. 이렇게 하면 의존성이 잘못된 방향으로 가리켜서 의존성 규칙을 위반할 위험이 줄어든다.

```
buckpal
└─────── account
          ├──── adapter
          │      ├──── in
          │      │     └──── web
          │      │           └──── O AccountController
          │      │
          │      └──── out
          │            └──── persistence
          │                  ├──── O AccountPersistenceAdapter
          │                  └──── O SpringDataAccountRepository
          │       
          ├──── domain
          │      ├──── + Account
          │      └──── + Activity
          │
          └──── application
                 ├──── O SendMoneyService
                 └──── port
                        ├──── in
                        │     └──── + SendMoneyUseCase
                        └──── out
                              ├──── + LoadAccountPort
                              └──── + UpdateAccountStatePort
                        
```
> 접근 제한자를 추가한 패키지 구조

persistence 패키지에 있는 클래스들은 외부에서 접근 할 필요가 없기 때문에 package-private(위에서 O)으로 만들 수 있다.

영속성 어댑터는 자신이 구현하는 출력 포트를 통해 접근된다. 

의존성 주입 메커니즘은 일반적으로 리플렉션을 이용해 인스턴스를 만들기 때문에 package-private이라도 여전히 인스턴스로 만들 수 있다.

`package-private` 제한자는 몇 개 정도의 클래스로만 이뤄진 작은 모듈에서 가장 효과적이다. 특정 개수를 넘어가기 시작하면 하나의 패키지에 너무 많은 클래스를 포함하는 것이 혼란스러워지게 된다.

### 컴파일 후 체크

클래스에 public 제한자를 쓰면 아키텍처 상의 의존성 방향이 잘못되더라도 컴파일러는 다른 클래스들이 이 클래스를 사용하도록 허용한다.

한 가지 방법은 컴파일 후 체크(post-compile check)를 도입하는 것이다. 다시 말해, 코드가 컴파일된 후에 런타임에 체크한다는 뜻이다.

이러한 체크를 도와주는 자바용 도구로 ArchUnit이라는 도구가 있다. ArchUnit은 의존성 방향이 기대한 대로 잘 설정돼 있는지 체크할 수 있는 API를 제공한다. 의존성 규칙 위반을 발견하면 예외를 던진다.

아래의 예제는 도메인 계층에서 바깥쪽의 애플리케이션 계층으로 향하는 의존성이 없다는 것을 체크할 수 있다.

```java
class DependencyRuleTests {

	@Test
	void testPackageDependencies() {
		noClasses()
				.that()
				.resideInAPackage("io.reflectoring.reviewapp.domain..")
				.should()
				.dependOnClassesThat()
				.resideInAnyPackage("io.reflectoring.reviewapp.application..")
				.check(new ClassFileImporter()
						.importPackages("io.reflectoring.reviewapp.."));
	}
}
```

ArchUnit API를 이용하면 적은 작업만으로도 육각형 아키텍처 내에서 관련된 모든 패키지를 명시할 수 있는 일종의 도메인 특화 언어(DSL)을 만들수 있고, 패키지 사이의 의존성 방향이 올바른지 자동으로 체크할 수 있다.

```java
class DependencyRuleTests {

	@Test
	void validateRegistrationContextArchitecture() {
		HexagonalArchitecture.boundedContext("io.reflectoring.buckpal.account")

				.withDomainLayer("domain")

				.withAdaptersLayer("adapter")
				.incoming("in.web")
				.outgoing("out.persistence")
				.and()

				.withApplicationLayer("application")
				.services("service")
				.incomingPorts("port.in")
				.outgoingPorts("port.out")
				.and()

				.withConfiguration("configuration")
				.check(new ClassFileImporter()
						.importPackages("io.reflectoring.buckpal.."));
	}
}
```

위 예제에서 먼저 바인드디 컨텍스트의 부모 패키지를 지정한다(단일 바운디드 컨텍스트라면 애플리케이션 전체에 해당한다). 그런 다음 도메인, 어댑터, 애플리케이션, 설정 계층에 해당하는 하위 패키지들을 지정한다.

마지막 호출하는 check()는 몇가지 체크를 실행하고 패키지 의존성의 의존성 규칙을 따라 유효하게 설정됐는지 검증한다.

컴파일 후 체크는 리팩터링에 취약하기 때문에 항상 코드와 같이 유지보수 해야 한다.

### 빌드 아티팩트

빌드 아티팩트는 (아마도 자동화된) 빌드 프로세스의 결과물이다. Java에서 인기있는 빌드도구는 메이븐(Maven)과 그레이들(Gradle)이다.

빌드 도구의 주요한 기능 중 하나는 의존성 해결(dependency resolution)이다. 어떤 코드베이스를 빌드 아티팩트로 변환하기 위해 빌드 도구가 가장 먼저 할 일은 코드베이스가 의존하고 있는 모든 아티팩트가 사용 가능한지 확인하는 방법이다.

이를 활용하여 모듈과 아키텍처의 계층 간의 의존성을 강제할 수 있다.(따라서 경계를 강제하는 효과가 생긴다.)

각 모듈 혹은 계층에 대해 전용 코드베이스와 빌드 아티팩트로 분리된 비륻 모듈(JAR 파일)을 만들 수 있다. 각 모듈의 빌드 스크립트에서는 아키텍처에서 허용하는 의존성만 지정한다. 클래스들이 클래스패스에 존재하지 않아 컴퍼일 에러가 발생하기 때문에 개발자들은 더이상 실수로 잘못된 의존성을 만들 수 없다.

![image](/assets/img/post/2022-04-23-MakeLearnCleanArchitecture_ch10/3.jpg)
> 잘못된 의존성을 막기 위해 아키텍처를 여러 개의 빌드 아티팩트로 만드는 여러가지 방법

위 그림의 핵심은 모듈을 더 세불화할수록, 모듈 간 의존성을 더 잘 제어할 수 있게 된다는 것이다. 하지만 더 작게 분리할수록 모듈간에 매핑을 더 많이 수행해야 한다.

이 밖에도 빌드 모듈로 아키텍처 경계를 구분하는 것은 패키지로 구분하는 방식과 비교했을 때 몇 가지 장점이 있다.

첫 번째로, 빌드 도구가 순환 의존성(circular dependency)를 극도로 싫어한다는 것이다. 순환 의존성은 하나의 모듈에서 일어나는 변경이 잠재적으로 순환 고리에 포함된 다른 모든 모듈을 변경하게 만들며, 단일 책임 원칙을 위배하기 때문에 좋지 않다. 그러므로 빌드 도구를 이용하면 빌드 모듈 간 순환 의존성이 없음을 확신 할 수 있다.

두 번째로, 빌드 모듈 방식에서는 다른 모듈을 고려하지 않고 특정 모듈의 코드를 격리한채로 변경 할 수 있다. 특정 어댑터에서 컴파일 에러가 생기는 애플리케이션 계층을 리팩터링하고 있다고 상상해본다.

만약 애플리케이션 계층이 같은 빌드 모듈에 있다면 어느 한쪽 계층의 컴파일 에러 때문에 빌드가 실패할 것이다. 그러므로 여러 개의 빌드 모듈은 각 모듈을 격리한 채로 변경할 수 있게 해준다.

심지어 각 모듈은 자체 코드 리포지토리에 넣어 서로 다른 팀이 서로 다른 모듈을 유지보수하게 할 수도 있다.

마지막으로, 모듈 간 의존성이 빌드 스크립트에 분명하게 선언돼 있기 때문에 새로 의존성을 추가하는 일은 우연이 아닌 의식적인 행동이 된다. 어떤 개발자가 당장은 접근 할 수 없는 특정 클래스에 접근해야 ㅎ라 일이 생기면 빌드 스크립트에 이 의존성을 추가하기에 앞서 정말로 이 의존성이 ㅍ리요한 것인지 생각할 여지가 생긴다.

하지만 이런 장점에는 빌드 스크립트를 유지보수하는 비용을 수반하기 때문에 아키텍처를 여러 개의 빌드 모듈로 나누기 전에 아키텍처가 어느 정도 안정된 상태여야 한다.
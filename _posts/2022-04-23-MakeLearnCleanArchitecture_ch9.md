---
title: 애플리케이션 조립하기
author: Jandari
date: 2022-04-23 10:50:00 +09:00
categories: [DDD, 만들면서 배우는 클린 아키텍처]
tags: [DDD, CleanArchitecture]
math: true
mermaid: true
---

## 소개

![image](/assets/img/post/2022-04-23-MakeLearnCleanArchitecture_ch9/1.jpg)

만들면서 배우는 클린 아키텍처 책을 읽고 정리하며 소감을 적는 포스트입니다.

## 애플리케이션 조립하기 

유스케이스, 웹 어댑터, 영속성 어댑터를 구현해봤으니, 이제 이것들을 동작하는 애플리케이션으로 조립하는 차례이다.

### 왜 조립까지 신경 써야 할까?

필요할때 마다 인스턴스화 시켜서 사용하지 않는 것은 의존성이 올바른 방향을 가리키게 하기 위해서다.

애플리케이션의 도메인 코드 방향으로 향해야 도메인 코드가 바깥 계층의 변경으로부터 안전하다.

유스케이스가 영속성 어댑처를 호출해야 하고 스스로 인스턴스화한다면 코드 의존성이 잘못된 방향으로 만들어진 것이다.

그럼 객체 인스턴스를 생성할 책임은 누구에게 있을까?? 


![image](/assets/img/post/2022-04-23-MakeLearnCleanArchitecture_ch9/2.jpg)
> 중립적인 설정 컴포넌트는 인스턴스 생성을 위해 모든 클래스에 접근할 수 있다.

위 그림과 같이 아키텍처에 대해 중립적이고 인스턴스 생성을 위해 `모든` 클래스에 대한 의존성을 가지는 설정 컴포넌트(configuration component)가 있어야 하는 것이다.

`설정 컴포넌트`는 아래와 같은 역할을 수행해야 한다.

* 웹 어댑터 인스턴스 생성
* HTTP 요청이 실제로 웹 어댑터로 전달되도록 보장
* 유스케이스 인스턴스 생성
* 웹 어댑터에 유스케이스 인스턴스 제공
* 영속성 어댑터 인스턴스 생성
* 유스케이스에 영속성 어댑터 인스턴스 제공
* 영속성 어댑터가 실제로 데이터베이스에 접근 할 수 있도록 보장

하지만 위와 같이 책임(`"변경할 이유"`)이 굉장히 많다. 이것은 단일 책임 원칙을 위반하는 게 아닐까?? 위반하는 게 맞다.

### 평범한 코드로 조립하기

의존성 주입 프레임워크의 도움 없이 애플리케이션을 만들고 있다면 아래와 같이 평범한 코드로 컴포넌트를 만들 수 있다.

```java
package com.book.cleanarchitecture.buckpal;

import com.book.cleanarchitecture.buckpal.account.adapter.in.web.SendMoneyController;
import com.book.cleanarchitecture.buckpal.account.adapter.out.persistence.AccountPersistenceAdapter;
import com.book.cleanarchitecture.buckpal.account.application.port.in.SendMoneyUseCase;
import com.book.cleanarchitecture.buckpal.account.application.service.SendMoneyService;

public class Application {
    public static void main(String[] args) {
        AccountRepository accountRepository = new AccountRepository();
        ActivityRepository activityRepository = new ActivityRepository();

        AccountPersistenceAdapter accountPersistenceAdapter = new AccountPersistenceAdapter(accountRepository, activityRepository);

        SendMoneyUseCase sendMoneyUseCase = new SendMoneyService(
                accountPersistenceAdapter,
                accountPersistenceAdapter
        );

        SendMoneyController sendMoneyController = new SendMoneyController(sendMoneyUseCase);
        
        startProcessingWebRequests(sendMoneyController);
    }
}
```

위와 같은 방식으로 애플리케이션을 조립하게 되면 아래와 같이 몇가지 단점이 존재한다.

1. 위 코드는 웹 컴트롤러, 유스케이스, 영속성 어댑터가 단 하나씩만 있는 애플리케이션을 예로 들고 있지만 앤터프라이즈 애플리케이션은 엄청나게 많은 코드가 들어가야한다.
2. 각 클래스가 속한 패키지 외부에서 인스턴스를 생성하기 때문에 이 클래스들은 전부 public이어야 한다.


### 스프링의 클래스패스 스캐닝으로 조립하기

스프링의 클래스패스 스캐닝으로 클래스패스에서 접근 가능한 모든 클래스를 확인해서 @Component 애너테이션이 붙은 클래스를 찾는다.

```java
@RequiredArgsConstructor
@PersistenceAdapter
class AccountPersistenceAdapter implements
		LoadAccountPort,
		UpdateAccountStatePort {

	private final SpringDataAccountRepository accountRepository;
	private final ActivityRepository activityRepository;
	private final AccountMapper accountMapper;

	@Override
	public Account loadAccount(
					AccountId accountId,
					LocalDateTime baselineDate) {
        //...
	}

	@Override
	public void updateActivities(Account account) {
		//...
	}
}
```

클래스패스 스캐닝 방식을 이용하면 아주 편리하게 애플리케이션을 조립할 수 있다. 적절한 곳에 @Component 애너테이션을 붙이고 생성자만 잘 만들어 두면 된다.

이 어노테이션 덕분에 코드를 읽는 사람들은 아키텍처를 더 쉽게 파악할 수 있지만 단점도 있다.

첫 번째로, 클래스에 프레임워크에 특화된 애너테이션을 붙여야 한다는 점에서 침투적이다.

라이브러리나 프레임워크를 만드는 입장에서는 사용하지 말아야 하는 방법이다.

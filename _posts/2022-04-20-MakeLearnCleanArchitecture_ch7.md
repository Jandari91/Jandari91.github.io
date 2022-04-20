---
title: 아키텍처 요소 테스트하기
author: Jandari
date: 2022-04-20 23:20:00 +09:00
categories: [DDD, 만들면서 배우는 클린 아키텍처]
tags: [DDD, CleanArchitecture]
math: true
mermaid: true
---

## 소개

![image](/assets/img/post/2022-04-20-MakeLearnCleanArchitecture_ch7/1.jpg)

만들면서 배우는 클린 아키텍처 책을 읽고 정리하며 소감을 적는 포스트입니다.

## 아키텍처 요소 테스트하기

많은 프로젝트의 자동화된 테스트는 규칙에 따라 작성되지만 테스트 전략을 물었을 때 제대로 답변하는 이가 없었다.

이번 포스트에서는 육각형 아키텍처에서의 테스트 전략에 대해 이야기 합니다.

### 테스트 피라미드

![image](/assets/img/post/2022-04-20-MakeLearnCleanArchitecture_ch7/2.jpg)
> 테스트 피라미드에 따르면 비용이 많이 드는 테스트는 지양하고 비용이 적게 드는 테스트를 많이 만들어야 한다.

기본 전제는 만드는 비용이 적고, 유지보수하기 쉽고, 빨리 실행되고, 안정적인 작은 크기의 테스트들에 대해 높은 커버리지를 유지해야 한다.

단위와 단위를 넘는 경계, 아키텍처의 경계, 시스템의 경계를 결합하는 테스트는 만드는 비용이 더 비싸지고, 실행이 더 느려지며 깨지기 쉬워진다.

단위 테스트는 피라미드의 토대에 해당한다. 일반적으로 하나의 클래스를 인스턴스화하고 해당 클래스의 인터페이스를 통해 기능들을 테스트 한다. 만약 테스트 중인 클래스가 다른 클래스에 의존한다면 의존되는 클래스들은 인스턴스화 하지 않고 테스트하는 동안 필요한 작업들을 흉내 내는 목(mock)으로 대체한다.

통합테스트는 연결된 여러 유닛을 인스턴스화하고 시작점이 되는 클래스의 인터페이스로 데이터를 보낸 후 유닛들의 네트워크가 기대한 대로 잘 동작하는지 검증한다.

두 계층 간의 경계를 걸쳐서 테스트 할 수 있기 때문에 객체 네트워크가 완전하지 않거나 어떤 시점에는 목을 대상으로 수행해야 한다.

시스템 테스트는 애플리케이션을 구성하는 모든 객체 네트워크를 가동시켜 특정 유스케이스가 전 계층에서 잘 동작하는지 검증한다.

### 단위 테스트로 도메인 엔티티 테스트하기

Account 엔티티를 테스트하기 위해 상태틑 과거 특정 시점의 계좌 잔고(baselineBalance)와 그 이후의 입출금 내역(activity)으로 구성돼 있다.

[github code](https://github.com/wikibook/clean-architecture/blob/main/src/test/java/io/reflectoring/buckpal/account/application/domain/AccountTest.java#L31)

```java
class AccountTest{
    @Test
    void withdrawalSucceeds(){
        AccountId accountId = new AccountId(1L);
        Account account = defaultAccount()
            .withAccountId(accountId)
            .withBaselineBalance(Money.of(555L))
            .withActivityWindow(new ActivityWindow(
                defaultActivity()
                    .withTargetAccount(accountId)
                    .withMoney(Money.of(999L)).build(),
                defaultActivity()
                    .withTargetAccount(accountId)
                    .withMoney(Money.of(1L)).build()))
            .build();

        boolean success = account.withdraw(Money.of(555L), new AccountId(99L));

        assertThat(success).isTrue();
        assertThat(account.getActivityWindow().getActivities()).hasSize(3);
        assertThat(account.calculateBalance()).isEqualTo(Money.of(1000L));
    }
}
```

특정 상태의 Account를 인스턴스화하고 withdraw()메서드를 호출해서 출금이 성공했는지 검증하고, Account 객체의 상태에 대해 기대되는 부수효과들이 잘 일어났는지 확인하는 단순한 단위 테스트다.

단위 테스트는 이해하기 쉽고, 아주 빠르게 실행된다. 또한 도메인 엔티티에 녹아 있는 비즈니스 규칙을 검증하기에 가장 적절한 방법이다.

### 단위 테스트로 유스케이스 테스트 하기

계층의 바깥쪽으로 나가서, 다음으로 테스트 할 것이 아키텍처의 유스ㅔ이스다.

`SendMoneyService`의 테스트를 살펴보면, SenMoney 유스케이스는 출금 계좌의 잔고가 다른 트랜잭션에 의해 변경되지 않도록 락(lock)을 건다. 출금 계좌에서 돈이 출금되고 나면 똑같이 입금 계좌에 락을 걸고 돈을 입금시킨다.

[github code](https://github.com/wikibook/clean-architecture/blob/main/src/test/java/io/reflectoring/buckpal/account/application/service/SendMoneyServiceTest.java#L63)

```java
class SendMoneyServiceTest{
    // 필드 선언은 생략

    @Test
    void transactionSucceeds() {
        Account sourceAccount = givenSourceAccount();
        Account targetAccount = givenTargetAccount();

        givenWithdrawalWillSucceed(sourceAccount);
        givenDepositWillSucceed(targetAccount);

        Money money = Money.of(500L);

        SendMoneyCommand command = new SendMoneyCommand(
            sourceAccount.getId(),
            targetAccount.getId(),
            money);

        boolean success = sendMoneyService.sendMoney(command);

        assertThat(success).isTrue();

        AccountId sourceAccountId = sourceAccount.getId();
        AccountId targetAccountId = targetAccount.getId();

        then(accountLock).should().lockAccount(eq(sourceAccountId));
        then(targetAccount).should().withdraw(eq(money), eq(targetAccountId));
        then(accountLock).should().releaseAccount(eq(sourceAccountId));

        then(accountLock).should().lockAccount(eq(targetAccountId));
        then(targetAccount).should().deposit(eq(money), eq(sourceAccountId));
        then(accountLock).should().releaseAccount(eq(targetAccountId));

        thenAccountsHaveBeenUpdated(sourceAccountId, targetAccountId);
    }

    // 헬퍼 메서드 생략
}

```
테스트의 가독성을 높이기 위해 행동-주도 개발(behavior driven development)에서 일반적으로 사용하는 방식대로 given/when/then 섹셩으로 나눳다.

`given` 섹션에서는 출금 및 입금 Account의 인스턴스를 각각 생성하고 적절한 상태로 만들고 SendMoneyCommand 인스턴스도 만들어서 유스케이스의 입력으로 사용한다.

`when` 섹션에서는 유스케이스를 실행하기 위해 sendMoeny() 메서드를 호출한다.

`then` 섹션에서는 트랜잭션이 성공적이었는지 확인하고, 출금 및 입금 Account, 그리고 계좌에 락을 걸고 해제하는 책임을 가진 AccountLock에 대해 특정 메서드가 호출됐는지 검증한다.

서비스가 (모팅된) 의존 대상의 특정 메서드와 상호작용햇는지 여부를 검증하면, 테스트가 코드의 `행동` 변경 뿐만 아니라 코드의 `구조` 변경에도 취약해진다.

그렇기 때문에 테스트에서 어떤 상호작용을 검증하고 싶은지 신중하게 생각해야 한다.

### 통합 테스트로 웹 어댑터 테스트하기

한 계층 더 바깥으로 나가면 어댑터에 도착한다. 웹 어댑터를 테스트 한다.

[github code](https://github.com/wikibook/clean-architecture/blob/main/src/test/java/io/reflectoring/buckpal/account/adapter/in/web/SendMoneyControllerTest.java)

```java
@WebMvcTest(controllers = SendMoneyController.class)
class SendMoneyControllerTest {

	@Autowired
	private MockMvc mockMvc;

	@MockBean
	private SendMoneyUseCase sendMoneyUseCase;

	@Test
	void testSendMoney() throws Exception {

		mockMvc.perform(post("/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}",
				41L, 42L, 500)
				.header("Content-Type", "application/json"))
				.andExpect(status().isOk());

		then(sendMoneyUseCase).should()
				.sendMoney(eq(new SendMoneyCommand(
						new AccountId(41L),
						new AccountId(42L),
						Money.of(500L))));
	}

}
```

MockMvc 객체를 이용해 모킹했기 때문에 실제로 HTTP 프로토콜을 통해 테스트 한 것은 아니다. 프레임워크가 HTTP 프로토콜에 맞게 모든 것을 적절히 잘 변환한다고 믿는 것이다.

이 테스트는 왜 단위 테스트가 아니라 통합 테스트 인가?? 이 테스트에서는 하나의 웹 컨트롤러 클래스만 테스트 한 것처럼 보이지만, 사실 보이지 않는 곳에서 다 많은 일들이 벌어지고 있다.

웹 컨트롤럴가 스프링 프레임워크에 강하게 묶여 있기 때문에 격리된 상태로 테스트하기 보다는 이 프레임워크와 통합된 상태로 테스트 하는 것이 합리적이다.

### 통합 테스트로 영속성 어댑터 테스트하기

영속성 어댑터의 테스트는 단위 테스트보다는 통합 테스트를 적용하는 것이 합리적이다.

[github code](https://github.com/wikibook/clean-architecture/blob/main/src/test/java/io/reflectoring/buckpal/account/adapter/out/persistence/AccountPersistenceAdapterTest.java)

```java
@DataJpaTest
@Import({AccountPersistenceAdapter.class, AccountMapper.class})
class AccountPersistenceAdapterTest {

	@Autowired
	private AccountPersistenceAdapter adapterUnderTest;

	@Autowired
	private ActivityRepository activityRepository;

	@Test
	@Sql("AccountPersistenceAdapterTest.sql")
	void loadsAccount() {
		Account account = adapterUnderTest.loadAccount(new AccountId(1L), LocalDateTime.of(2018, 8, 10, 0, 0));

		assertThat(account.getActivityWindow().getActivities()).hasSize(2);
		assertThat(account.calculateBalance()).isEqualTo(Money.of(500));
	}

	@Test
	void updatesActivities() {
		Account account = defaultAccount()
				.withBaselineBalance(Money.of(555L))
				.withActivityWindow(new ActivityWindow(
						defaultActivity()
								.withId(null)
								.withMoney(Money.of(1L)).build()))
				.build();

		adapterUnderTest.updateActivities(account);

		assertThat(activityRepository.count()).isEqualTo(1);

		ActivityJpaEntity savedActivity = activityRepository.findAll().get(0);
		assertThat(savedActivity.getAmount()).isEqualTo(1L);
	}

}
```

이 테스트는 데이터베이스를 모킹하지 않았다는 점이 중요하다. 테스트가 실제로 데이터베이스에 접근한다. 데이터베이스를 모킹했더라도 테스트는 여전히 같은 코드 라인 수만큼 커버해서 똑같이 높은 커버리지를 보여줬을 것이다.

기본적으로 인메모리(in-memory) 데이터베이스를 테스트에서 사용한다. 아뭇것도 설정할 필요 없이 곧바로 테스트할 수 있으므로 아주 실용적이다.

### 시스템 테스트로 주요 경로 테스트하기

시스템 테스트는 전체 애플리케이션을 띄우고 API를 통해 요청을 보내고, 모든 계층이 조화롭게 잘 동작하는지 검증한다.

[github code](https://github.com/wikibook/clean-architecture/blob/main/src/test/java/io/reflectoring/buckpal/SendMoneySystemTest.java)

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class SendMoneySystemTest {

	@Autowired
	private TestRestTemplate restTemplate;

	@Autowired
	private LoadAccountPort loadAccountPort;

	@Test
	@Sql("SendMoneySystemTest.sql")
	void sendMoney() {

		Money initialSourceBalance = sourceAccount().calculateBalance();
		Money initialTargetBalance = targetAccount().calculateBalance();

		ResponseEntity response = whenSendMoney(
				sourceAccountId(),
				targetAccountId(),
				transferredAmount());

		then(response.getStatusCode())
				.isEqualTo(HttpStatus.OK);

		then(sourceAccount().calculateBalance())
				.isEqualTo(initialSourceBalance.minus(transferredAmount()));

		then(targetAccount().calculateBalance())
				.isEqualTo(initialTargetBalance.plus(transferredAmount()));

	}

	private Account sourceAccount() {
		return loadAccount(sourceAccountId());
	}

	private Account targetAccount() {
		return loadAccount(targetAccountId());
	}

	private Account loadAccount(AccountId accountId) {
		return loadAccountPort.loadAccount(
				accountId,
				LocalDateTime.now());
	}


	private ResponseEntity whenSendMoney(
			AccountId sourceAccountId,
			AccountId targetAccountId,
			Money amount) {
		HttpHeaders headers = new HttpHeaders();
		headers.add("Content-Type", "application/json");
		HttpEntity<Void> request = new HttpEntity<>(null, headers);

		return restTemplate.exchange(
				"/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}",
				HttpMethod.POST,
				request,
				Object.class,
				sourceAccountId.getValue(),
				targetAccountId.getValue(),
				amount.getAmount());
	}

    //일부 헬퍼 메서드는 생략
}
```

시스템 테스트라고 하더라도 언제나 서드파티 시스템을 실행해서 테스트할 수 있는 것은 아니기 때문에 결국 모킹을 해야 할 때도 있다. 육각형 아키텍처는 이러한 경우 몇 개의 출력 포트 인터페이스만 모킹하면 되기 때문에 아주 쉽게 이 문제를 해결할 수 있다.

### 얼마만큼의 테스트가 충분할까??

테스트가 코드의 80%를 커버하면 충분할까? 아니면 그보다 높아야 할까??

라인 커버리지(line coverage)는 테스트 성공을 측정하는 데 있어서 잘못된 지표다.

나는 얼마나 마음 편하게 소프트웨어를 배포할 수 있느냐를 테스트의 성공 기준으로 삼으면 된다고 생각한다.

처음 몇 번의 배포에는 믿음의 도약이 필요하다. 그렇지만 `프로덕션의 버그를 수정하고 이로부터 배우는 것` 우선순위로 삼으면 제대로 가고 있는 것이다.

아래는 육각형 아키텍처에서 사용하는 테스트 전략이다.

* 도메인 엔티티를 구현 할 때는 단위 테스트로 커버하자
* 유스케이스를 구현할 때는 단위 테스트로 커버하자
* 어댑터를 구현할 때는 통합 테스트로 커버하자
* 사용자가 취할 수 있는 중요 애플리케이션 경로는 시스템 테스트로 커버하자

`구현할 때는`이라는 문구가 중요한데, 만약 테스트가 기능 개발 후가 아닌 개발 중에 이뤄진다면 하기 싫은 귀찮은 작업이 아니라 개발 도구로 느껴질 것이다.


---
title: 4장. 유스케이스 구현하기
author: Jandari
date: 2022-04-17 22:20:00 +09:00
categories: [DDD, 만들면서 배우는 클린 아키텍처]
tags: [DDD, CleanArchitecture]
math: true
mermaid: true
---

## 소개

![image](/assets/img/post/2022-04-17-MakeLearnCleanArchitecture_ch4/1.jpg)

만들면서 배우는 클린 아키텍처 책을 읽고 정리하며 소감을 적는 포스트입니다.

## 유스케이스 구현하기

이전 포스트에서 만든 육각형 아키텍처에서 애플리케이션, 웹, 영속성 계층이 현재 아키텍처에서 아주 느슨하게 결합돼 있기 때문에 필요한 대로 도메인 코드를 자유롭게 모델링할 수 있다.

DDD를 할 수 있고, 풍부하거나(rich), 빈약한(anemic) 도메인 모델을 구현할 수도 있다.

### 도메인 모델 구현하기

한 계좌에서 다른 계좌로 송금하는 유스케이스를 구현해 본다.

입금과 출금을 할 수 있는 Account 엔티티를 만들고 출금 계좌에서 돈을 출금해서 입금 계좌로 돈을 입금하는 것이다.

```java
package buckpal.account.domain;

@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Account {

	@Getter private final AccountId id;

	@Getter private final Money baselineBalance;

	@Getter private final ActivityWindow activityWindow;

	public static Account withoutId(
					Money baselineBalance,
					ActivityWindow activityWindow) {
		return new Account(null, baselineBalance, activityWindow);
	}

	public static Account withId(
					AccountId accountId,
					Money baselineBalance,
					ActivityWindow activityWindow) {
		return new Account(accountId, baselineBalance, activityWindow);
	}

	public Optional<AccountId> getId(){
		return Optional.ofNullable(this.id);
	}

	public Money calculateBalance() {
		return Money.add(
				this.baselineBalance,
				this.activityWindow.calculateBalance(this.id));
	}

	public boolean withdraw(Money money, AccountId targetAccountId) {

		if (!mayWithdraw(money)) {
			return false;
		}

		Activity withdrawal = new Activity(
				this.id,
				this.id,
				targetAccountId,
				LocalDateTime.now(),
				money);
		this.activityWindow.addActivity(withdrawal);
		return true;
	}

	private boolean mayWithdraw(Money money) {
		return Money.add(
				this.calculateBalance(),
				money.negate())
				.isPositiveOrZero();
	}

	public boolean deposit(Money money, AccountId sourceAccountId) {
		Activity deposit = new Activity(
				this.id,
				sourceAccountId,
				this.id,
				LocalDateTime.now(),
				money);
		this.activityWindow.addActivity(deposit);
		return true;
	}

	@Value
	public static class AccountId {
		private Long value;
	}

}
```

계좌에 대한 모든 입금과 출금은 Activity 엔티티에 포착 된다.

한 계좌에 대한 모든 활동(activity)들은 항상 메모리에 한꺼번에 올리는 것은 현명한 방법이 아니기 때문에 Account 엔티티는 ActivityWindow 값 객체(value object)에서 포착한 지난 며칠 혹은 몇 주간의 범위에 해당하는 활동만 보유한다.

#### Account 엔티티

실제 계좌의 현재 스냅샷을 제공

#### ActivityWindows
한 계좌에 대한 모든 활동(activity)들을 항상 메모리에 한꺼번에 올리는 것은 현명하지 않으므로 지난 며칠 혹은 몇 주간의 범위 활동만 보유

#### baselineBalance

첫번째 활동 전의 잔고

현재 총 잔고는 기준 잔고(baseBalance)에 활동 창의 모든 활동들의 잔고를 합한 값

#### withdraw (입금), deposit(출금)

새로운 활동을 활동창에 추가

출금 전에는 잔고를 초과하는 금액을 출금할 수 없도록 비즈니스 규칙 검사

### 유스케이스 둘러보기

일반적으로 유스케이스는 아래와 같은 단계를 따른다.

1. 입력을 받는다.
1. 비즈니스 규칙을 검증한다.
1. 모델 상태를 조작한다.
1. 출력을 반환한다.

유스케이스 코드가 도메인 로직에만 신경 써야 하고 입력 유효성 검증으로 오염되면 안된다. 그래서 입력 유효성 검증은 다른 곳에서 진행해야 한다.

유스케이스는 `비즈니스 규칙(business rule)`을 검증할 책임이 있다. 그리고 도메인 엔티티와 이 책임을 공유한다.

아래는 `송금하기` 유스케이스를 구현한 것이다.

```java
package buckpal.account.application.service;

@RequiredArgsConstructor
@Transactional
public class SendMoneyService implements SendMoneyUseCase {

	private final LoadAccountPort loadAccountPort;
	private final AccountLock accountLock;
	private final UpdateAccountStatePort updateAccountStatePort;

	@Override
	public boolean sendMoney(SendMoneyCommand command) {
		// TODO: 비즈니스 규칙 검증
		// TODO: 모델 상태 조작
		// TODO: 출력 값 반환
	}
}
```

![image](/assets/img/post/2022-04-17-MakeLearnCleanArchitecture_ch4/2.jpg)
> 하나의 서비스가 하나의 유스케이스를 구현하고, 도메인 모델을 변경하고, 변경된 상태를 저장하기 위해 아웃고잉 포트를 호출한다.

서비스는 인커밍 포트 인터페이스인 `SendMoneyUseCase`를 구현하고, 계좌를 불러오기 위해 아웃고잉 포트 인터페이스인 `LoadAccountPort`를 호출한다.

그리고 데이터베이스의 계좌 상태를 업데이트하기 위해 `UpdateAccountStatePort`를 호출한다.

### 입력 유효성 검증

애플리케이션 계층해서 입력 유효성을 검증해야 하는 이유는, 그렇게 하지 않을 경우 애플리케이션 코어의 바깥쪽으로부터 유효하지 않은 입력값을 받게 되고, 모델의 상태를 해칠 수 있다.

따라서 애플리케이션 계층에서 유효성을 검증해야 하지만 유스케이스 클래스에서는 하면 안된다.

아래는 `SendMoneyCommand`클래스의 생성자 내에서 입력 유효성을 검증한다.

```java
@Getter
public class SendMoneyCommand {

    private final AccountId sourceAccountId;
    private final AccountId targetAccountId;
    private final Money money;

    public SendMoneyCommand(
            AccountId sourceAccountId,
            AccountId targetAccountId,
            Money money) {
        this.sourceAccountId = sourceAccountId;
        this.targetAccountId = targetAccountId;
        this.money = money;
        requireNonNull(sourceAccountId);
        requireNonNull(targetAccountId);
        requireNonNull(money);
        requireGraterThan(money, 0);
    }
}
```

`SendMoneyCommand`는 유스케이스 API의 일부이기 때문에 인커밍 포트 패키지에 위치한다. 그러므로 유효성 검증이 애플리케이션의 코어(육각형 아키텍처의 육각형 내부)에 남아 있지만 신성한 유스케이스 코드를 오염시키지는 않는다.

### 생성자의 힘

클래스는 불변이기 때문에 생성자의 인자 리스트에는 클래스의 각 속성에 해당하는 파라미터들이 포함돼 있다.

그뿐만 아니라 생성자가 파라미터의 유효성 검증까지 하고 있기 때문에 유효하지 않은 상태의 객체를 만드는 것은 불가능하다.

빌더 패턴을 사용하면 긴 파라미터 리스트를 받아야하는 생성자를 private으로 만들고 빌더의 build() 메서드 내부에 생성자를 숨길 수 있다.

```java
new SendMoneyCommandBuilder()
		.sourceAccountId(new AccountId(41L))
		.targetAccountId(new AccountId(42L))
		// ... 다른 여러 필드를 초기화
		.build();
```

유효성 검증 로직은 생성자에 그대로 둬서 빌더가 유효하지 않은 상태의 객체를 생성하지 못하도록 막을 수 있다.

그러나 빌더 패턴의 경우 코드에 새로운 필드를 추가하는 것을 잊었을 경우, 컴파일러가 유효하지 않은 불변 객체를 만들려는 시도에 대해 경고해주지는 못한다.

그래서 단위테스트 혹은 런타임에서 유효성 검증 로직이 동작해서 누락된 파라미터에 대한 에러를 던질 것이다.

하지만 생성자를 직접 사용한다면 컴파일 에러에 따라 나머지 코드에 즉각 변경사항을 반영 할 수 있다.

### 유스케이스마다 다른 입력 모델

`계좌 등록하기`와 `계좌 정보 업데이트하기`의 경우 거의 똑같은 계좌 상세 정보가 필요하다.

그래서 두 유스케이스에서 같은 입력 모델을 공유할 수도 있다.

그러나 같은 입력 모델을 공유하게 된다면 `계좌 등록하기`에서는 소유자 ID에 null 값을 허용해야만 한다.

불변 커맨드 객체의 필드에 대해서 null은 유효한 상태로 받아들이는 것은 그 자체로 코드 냄새(code smell)다.

### 비즈니스 규칙 검증하기

입력 유효성 검증과 비즈니스 규칙 검증은 분명히 유스케이스 로직의 일부이다.

그런데 언제 입력 유효성을 검증하고 언제 비즈니스 규칙을 검증해야 할까??

입력 유효성 검증은 `구문상의(syntactical)` 유효성으 검증하는 것이라고도 할 수 있고, 반면 비즈니스 규칙은 유스케이스 맥락 속에서 `의미적인(semantical)` 유효성을 검증하는 일이라고 할 수 있다.

예를 들면 `"출금 계좌는 초과 출금되어서는 안된다"`는 모델의 현재 상태에 접근하여 출금, 입금 계좌의 존재 유무를 판단해야 하기 때문에 `비즈니스 규칙`이다.

반대로 `"송금되는 금액은 0보다 커야한다."`라는 규칙은 모델의 현재 상태와 상관없이 검증 될 수 있기 때문에 `입력 유효성 검증`이다.

비즈니스 규칙의 가장 좋은 위치는 도메인 엔티티 안에 넣는 것이다.

```java
public class Account {
    // ...

    public boolean withdraw(Money money, AccountId targetAccountId) {
            if (!mayWithdraw(money)){
                    return false;
            }
            // ...
    }
}
```

만약 도메인 엔티티에서 비즈니스 규칙을 검증하기가 여의치 않다면 유스케이스 코드에서 도메인 엔티티를 사용하기 전에 해도 된다.

```java
@RequiredArgConstructor
@Transactional
public class SendMoneyService implements SendMoneyUseCase {
    // ...
    
    @Override
    public boolean sendMoney(SendMoneyCommand command){
                requireAccountExists(command.getSourceAccountId());
                requireAccountExists(command.getTargetAccountId());
    }
}
```

### 풍부한 도메인 모델 vs. 빈약한 도메인 모델

DDD 철학을 따르는 `풍부한 도메인 모델(rich domain model)`과 `빈약한 도메인 모델(anemic domain model)`의 구현에 대해 자주 논의 되는 사항이다.

풍부한 도메인 모델에서는 애플리케이션의 코어에 있는 엔티티에서 가능한 한 많은 도메인 로직이 구현된다. 엔티티들은 상태를 변경하는 메서드를 제공하고, 비즈니스 규칙에 맞는 유효한 변경만 허용한다.

또한 많은 비즈니스 규칙이 유스케이스 구현체 대신 엔티티에 위치하게 된다.

빈약한 도메인 모델에서는 엔티티 자체가 굉장히 얇다. 일반적으로 엔티티는 상태를 표현하는 필드와 이 값을 읽고 바꾸기 위한 getter, setter 메서드만 포함하고 어떤 도메인 로직도 가지고 있지 않는다.

이 말은 즉, 도메인 로직인 유스케이스 클래스에 구현돼 있다는 것이다. 비즈니스 규칙을 검증하고, 엔티티의 상태를 바꾸고, 데이터베이스 저장을 담당하는 아웃고잉 포트에 엔티티를 전달할 책임 역시 유스케이스 클래스에 있다.

 ### 유스케이스 마다 다른 출력 모델

 유스케이스가 할 일을 다하고 나면 호출자에게 무엇을 반환해야 할까?

 입력과 비슷하게 출력도 가능하면 각 유스케이스에 맞게 구체적일수록 좋다. 출력은 호출자에게 꼭 필요한 데이터만 들고 있어야 한다.

 `송금하기` 유스케이스 코드에서는 boolean 값 하나만 반환했다. 업데이트 된 Account를 통째로 반환하고 싶을 수도 있다.

 이부분은 정답이 없다. 그러나 유스케이스를 가능한 한 구체적으로 유지하기 위해서는 계속 질문해야 한다. 만약 의심스럽다면 가능한 한 적게 반환하자.

 유스케이스들 간에 같은 출력 모델을 공유하게 되면 유스케이스들도 강하게 결합된다. 단일 책임 원칙을 적용하고 모델을 분리해서 유지하는 것은 유스케이스의 결합을 제거하는 데 도움이 된다.

 같은 이유로 도메인 엔티티를 출력 모델로 사용하고 싶은 유혹도 견뎌야 한다.

 ### 읽기 전용 유스케이스는 어떨까?

 `계좌 잔고 보여주기` 같은 특정 유스케이스의 경우에도 다른 유스케이스와 비슷한 방식으로 구현해야 한다.

 ```java
package buckpal.application.service;

@RequiredArgsConstructor
class GetAccountBalanceService implements GetAccountBalanceQuery {
	private final LoadAccountPort loadAccountPort;

	@Override
	public Money getAccountBalance(AccountId, accountId){
		return loadAccountPort.loadAccount(accoutId, LocalDateTime.now()).calculateBalance();
	}
}
 ```

 쿼리 서비스는 유스케이스 서비스와 동일한 방식으로 동작한다. `GetAccountBalanceQuery`라는 인커밍 포트를 구현하고, 데이터베이스로부터 실제로 데이터를 로드하기 위해 `LoadAccountPort`라는 아웃고잉 포트를 호출한다.

 이처럼 읽기 전용 쿼리는 쓰기가 가능한 유스케이스(또는 커맨드)와 코드 상에서 명확하게 구분된다.

 이러한 방식은 CQS(Command-Query Separation)나 CQRS(Command-Query Responsibility Segregation) 같은 개념과 아주 잘 맞는다.


> CQRS란? 흔히 말하는 CRUD(Create, Read, Update, Delete)에서 CUD(Command)와 R(Query)를 구분하자는 것. <br/>
> `CQRS의 장점` <br/>
> Read와 CUD 각각에 더 최적화된 Database 구성을 통해서 성능을 더 향상시킬 수 있다. <br/>
> Read와 CUD에서 필요한 데이터 형식이 다를 수 있고, 특히 Read는 aggregation(집계 함수) 등의 부가적인 attribute들이 Entity에 필요하게 될 수 있다. <br/>
> R과 CUD를 분리함으로써 R로 인해 Entity의 구조가 변경되는 것을 막을 수 있다. <br/>
> R과 CUD를 분리함으로써 과도하게 복잡한 모델을 덜 복합하게 만듦으로서 시스템 복잡도를 줄일 수 있다. <br/>
> 출처: Log.bluayer


### 유지보수 가능한 소프트웨어를 만드는 데 어떻게 도움이 될까?

이 책의 아키텍처에서는 도메인 로직을 우리가 윈하는 대로 구현할 수 있도록 허용하지만, 입력 출력 모델을 독립적으로 모델링한다면 원치 않는 부수효과를 피할 수 있다.

물론 유스케이스 간에 모델을 공유하는 것보다는 더 많은 작업이 필요하다.

꼼꼼한 입력 유효성 검증, 유스케이스별 입출력 모델은 지속 가능한 코드를 만드는 데 큰 도움이 된다.
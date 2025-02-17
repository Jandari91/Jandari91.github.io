---
title: 한번 듣고 평생 써먹는 코드 리뷰 노하우
author: Jandari
date: 2022-04-21 22:00:00 +09:00
categories: [Code Review]
tags: [CodeReview]
math: true
mermaid: true
---

## 소개

이번 포스트는 [`한번 듣고 평생 써먹는 코드 리뷰 노하우`](https://www.youtube.com/watch?v=TAPviNhFuSg)를 시청하고 정리하는 포스트입니다.

# 목차

* 왜 코드 리뷰를 해야 하나?
  * 우리가 살고 있는 시대
  * 개발생산성 / SW 공학의 특성 / 장인정신
  * 코드 리뷰의 정의 / 목적
* 코드 리뷰의 절차
* 왜 코드 리뷰가 어려운가
* 코드 리뷰의 기법들
  * 효율적인 PR 방법
  * 효율적인 리뷰 방법
  * 피드백 방법
  * 교착상태 시
  * 추가적인 사례
  * 코드 리뷰를 하는 아주 재밌는 방법

# 왜 코드 리뷰를 해야하나?

## 우리가 살고 있는 시대
> `"Software is eating the world"` <br />

* 소프트웨어에 의해 운영되는 제품과 서비스들의 영역이 늘어나고 있음
* ICT의 융합으로 이뤄지는 차세대 산업 혁명
  * Big Data, AI, Robot 공학, IoT, 무인 운송 수단, 3D Printing, 나노 기술과 같은 7대 분야에서의 새로운 기술 혁신
* Build 2021 Keynote에서 Global GDP에서의 IT 기술의 비율이 2020년 5%에서 2030: 10%까지 갈것으로 예측했습니다.
* Non-Tech영역에서 개발자 수 증가속도가 Tech 영역에서보다 가파르게 증가한다.

### VUCA란?

* `V`olatiliy(변동성) : 변화의 속도가 빠르고 다양하게 전개 될 것
* `U`ncertainty(불확실성) : 미래 상황에 변수가 많아 예측하기 어려울 것
* `C`omplexity(복잡성) : 인과관계가 단순하지 않고 다양한 요인이 작용될 것
* `A`mbiguity(모호성) : 뚜렷한 현상이 없어 판별하기 어려울 것

`"불확실하고, 복잡하고, 모호하며 변화가 많은 세상이 될것!"`

* 비즈니스 성공을 위한 개발의 역할
  * 시장 : VUCA
  * 비즈니스 : 더 빨리 혁신해야 함
  * 개발 DRFR
* `개발 조직의 성능(생산성)`이 중요해짐


## 개발 생산성

아래의 그림들은 클린아키텍처에서 말하는 표입니다.

![image](/assets/img/post/2022-04-21-CodeReview/1.jpg)

X축 : 출시 회차, Y축 : 개발자 수 

출시를 하면 할 수록 필요한 개발자의 수가 늘어 난다.

![image](/assets/img/post/2022-04-21-CodeReview/2.jpg)

X축 : 출시 회차, Y축 : 생산성

출시 회차가 늘어 날 수록 생산성이 줄어듭니다.



![image](/assets/img/post/2022-04-21-CodeReview/3.jpg)
>설계 체력 가설 

X축 : 시간, Y축 : 추가 된 기능 횟수

시간이 가면 갈 수록 설계 수익선을 지나면 기능의 수가 줄어듭니다.

설계가 좋지 못하면 새로운 기능을 추가하기 위해 시간을 쓰지 못하고 기존의 문제를 해결하기 위해 시간을 많이 소비하게 된다.

## SW 공학의 특성

왜 SW 공학은 시간이 갈 수록 생산성이 줄어드나?

### 건축 공학

* 공학 = 설계(Design) + 빌드(Build)
  * 설계 : 예측하기 어렵고, 급여가 비싸고 창의적인 사람들을 필요
  * 빌드 : 좀 더 예측하기 쉬움
* 설계와 빌드가 `분리`됨
* `빌드 비용`이 비쌈(건축 90%)
* `유지보수` 비용이 상대적으로 낮음
* 공학 활동의 최종 목적
  * 빌드 할 수 있는 어떤 종류의 `재생산` 가능한(Reproducible) `문서`

### SW 공학

* 설계 = 완전한 `소스 코드`
* SW 빌드 : 컴파일
* 좋은 설계 ~= 클린코드
* SW 엔지니어 : 설계를 잘하는 사람 -> 코드를 잘 작성하는 사람

### 클린 코드의 중요성

* SW의 진정한 비용 ~= 유지보수(전체의 80% 이상)
* 한번 작성한 코드는 10번 이상 읽음. 작성보다 이해에 10배의 노력 소요
* 90% 이상의 시간을 어떤 코드를 이해하는데 사용함

### SW 개발의 단순한 진리

* "The only way to go fast, is to go well" - Rovert C. Martin
* 시간이 흘러도 생산성 저하, 비용 증가를 막을 수 있는 유일한 방법
* SW 품질에 신중해야
  * `SW의 비용과 품질의 관계`는 비정상적, 비직관적
  * 향후 변경 비용을 낮춤으로서 익숙한 트레이드오프를 역전 시킴
    * "높은 품질의 제품은 비싸다"
  * 개발자의 `전문가 정신? 비즈니스 혁신?` : 이런 걸로는 경영진을 설득 할 수 없다.

## SW 장인정신

* 장인정신
  * 지식과 경험의 `공유`만이 `전문성`을 갖춘 개발자 육성
  * 후배들에게 지식과 경험을 공유(도제관계)

* 코드 리뷰
  * `개발자가 지금부터 당장 행할 수 있는 공유 활동`
  * Code SNS 댓글 놀이
  * `배움을 주고 받으며 지속가능한 SW 개발자가 될 수 있는 실천법`
* 코드리뷰의 정의
  * `"코드 리뷰는 한명 또는 여러사람이 스스로 소스코드 일부를 보고 읽는 방식으로 프로그램을 확인하는 소프트웨어 경진 활동으로서 구연 후에 하거나 또는 구현을 잠깐 중단 시킨 후 한다."`
  * Peer Reviews, Pull Requests, Merge Request라고도 한다.

## 목적

* `주목적` : `품질 문제 검수`(버그, 장애)
* 더 나은 코드 `품질` : 아키텍처 속성 개선을 위한 코드 개선(`향후 변경 비용 개선`)
* `학습 및 지식 전달` : 코드, 해결책 등과 관련된 지식 `공유`에 기여
  * 공유(`주고 받는` 학습)를 통한 역량 증대 및 성장
  * 참여한 모든 사람들의 배움의 기회
  * 대개의 경우 `리뷰어들도` 리뷰 과정에서 지식을 얻게 됨(하드스킬, 소프트스킬)
  * `동기부여` : 다른사람이 잘하는 사람이 있으면 그 사람처럼 잘 하려고 한다.

> 하드스킬 : 개발의 구체적인 영역들(실제 코드 기술) <br/>
> 소프트스킬 : 시니어가 주니어에게 쉽게 설명하는 방법(친절, 공감능력 등..) <br/>

* `상호 책임감` 증대
  * 집단 코드 오너십 및 `결속` 증대
  * 내가 하고 있는 일에 `관심`을 가져주는 것
  * 팀에서 일어나는 일 `공유`. 내 동료는 무엇을 하나? `팀웍`
* `개발 문화 개선`
* `설계 개선 제안`


# 코드 리뷰의 절차

![image](/assets/img/post/2022-04-21-CodeReview/4.jpg)

* 저자(Author)
  * 코드 작성, 리뷰 요청
* 리뷰어
  * 코드를 읽고
  * 머지 가능한지 결정
* 변경 내역(Change List, PR)
  * 리뷰 시작 전에 작성
  * 저자가 머지를 원하는 소스 코드에 대한 일련의 변경(잘 한것, 아쉬운 것, 눈여겨 볼 것)에 대해 기술

## 좋은 Pull Request의 예

* `저자가 고생해서 리뷰어의 시간을 아껴줘야 한다!!`

![image](/assets/img/post/2022-04-21-CodeReview/5.jpg)

# 왜 코드 리뷰가 어려운가?

* 저자
  * 본인 생각에 멋지다고 생각하는 PR을 보냄
* 리뷰어
  * 왜 멋지지 않은지에 대한 장황한 이유를 작성
* `코드에 대한 비판을 자신에 대한 비판으로 이해`
* 코드 리뷰는
  * 지식 /공학적 `결정을 공유`하는 기회
  * `공유`(잘한것, 아쉬운 것)를 통해 서로의 `지식/경험`을 나누며 `상호 학습`을 통한 `역량 증대 수단`
  * 코드 토의를 `개인적 공격`으로 받아들이면 물거품
* 생각을 `글로 전달하는 것에 대한 어려움`
  * 오해의 위험이 큼(음성 톤, 표정의 부재)
* 피드백을 조심스럽게 표현하는 것이 더 중요
  * `파일 Handle 닫는게 빠졌어요!` -> `파일 Handle을 닫는 것 깜빡하다니! 넌 바보야!`로 받아드리는 경우가 있다.

## Git의 등장
* 금요일 오후 svn commit을 조사하며 리뷰
* 불러서 깨기, 불화/갈등
* Git의 등장
  * Local branch에 commit 단위 리뷰할 수 있도록
  * SNS 댓글 놀이

# 코드 리뷰의 기법들

## 효율적인 PR 방법

### 지루한 작업은 컴퓨터로 처리

코드를 읽는 것은 `인지적 부담`이 되는 `고수준의 집중`이 요구되는 작업

* `컴퓨터가 할 수 있는 일`에 이런 노력을 낭비하지 말라
* 심지어 기계가 더 잘 할 수 있는 일에 리뷰어의 시간을 낭비시키지 말라.
* Formatting Tool
  * 공백, 들여쓰기 오류 등
  * `별도의 커밋/PR로 분리`. 리뷰 불필요를 기술해서 리뷰를 생략 할 수 있도록

### 스타일 가이드를 통해 스타일 논쟁을 해소

스타일에 대한 논쟁은 리뷰에서 시간 낭비

* 유명한 코딩 스타일을 차용하거나 자신들의 코딩 스타일을 표준화 해야한다.

### PR을 올릴 때 주석 달기

PR을 저자가 먼저 읽어봐야 한다.

* 리뷰어들을 위한 설명을 커멘트로 남겨서 리뷰어들의 시간을 절약해야 한다.

### 모두를 포함하라

* 많은 사람들이 볼 수록 버그를 더 잘 찾아낼 수 있다.
* 많은 사람들이 본다는 것을 알면 사람들은 대개 더 잘 하려는 경향이 있다.(코드, PR 작성)

### 의미있는 커밋으로 분리
* 혼자하는 코드리뷰
* 커밋을 의미 있게 분리하면 혼자서 코드 리뷰를 할 수 있다.
* 2주 후에 봐도 이해가 될 수 있도록 해야한다.


## 효율적인 리뷰 방법

### 리뷰는 즉시 시작
* 코드 리뷰를 `높은 우선순위`로
  * 저자는 리뷰 종료될 때까지 대기(Blocked)함
* 리뷰를 바로 시작하면 선순환됨
  * 코드를 읽고 피드백을 줄 때는 시간을 가지고 진행해도 되지만 `시작은 바로 해라`
  * 이상적으로 수분 내에
* 리뷰 라운드의 최대 시간은 하루
  * 우선순위 높은 업무로 1일 내 불가하면 다른 리뷰어 지정
  * 월 1회 이상 재지정을 해야한다면 속도를 줄여서 건강한 개발 관습(Practices)을 유지할 수 있어야함

#### Agile Pull requests by Mark Seemann

* "괴로우면 더 자주해라!"
* 아침 스탠드 미팅에 익숙
  * 매일 아침 30분, 점심 식사 후 30분을 `리뷰를 위해 미리 확보`
* PR에 포함된 `변경이 적도록` 노력
  * `반나절 정도` 작업한 양 정도
  * 모든 팀원들이 하루에 두번 작은양의 PR을 리뷰할 수 있고 최대 4시간 안에 리뷰가 완료 ㅗ딜 것
* 근본적인 문제는 사람들이 `리뷰할 시간이 없다고 느낀다`는 것임
  * 당신의 `개인 기여`로만 `평가`를 받고 있다면, 팀을 돕기 위해 수행하는 모든 일은 시간 낭비처럼 보임
  * 이것은 리뷰를 하는 것의 문제라기보다는 `조직적인 문제`이다.

#### Pull Requests vs Pair Programming
* 트레이드오프 : Latency or throughput?
* 내성적, 사색, 비동기 : Pull Requests
* 외향적, 친밀한 개인적 상호 작용 : Pair Programming
* 절대답은 없음. 앙상블 방식도 채택 할 수 있다. : 시간을 잡아서 같이 리뷰한다.

### 리뷰를 할 때는 고수준으로 시작 저수준으로 내려가라

* 리뷰 라운드에서 `많은 의견을 남길 수록, 저자가 당황`할 위험 커짐
* 하나의 라운드에 20~50개 정도의 의견은 위험의 시작
* 초기 라운드에서는 `고수준 피드백으로 제한`
  * 버그, 장애, 성능, 보안 등
  * Extract Method, Composed Method, Invert-if(복잡도) 등
* 고수준의 피드백이 처리된 후에 `저수준 이슈`를 처리
  * (선택적인) 설계 개선
  * 변수명 변경, 주석을 명확하게 하는 것 등

### 예제 코드 제공에 관대해라
* 저자를 기분 좋게 하기 위한 방법
  * 리뷰 중에 `선물` 주기(코드 예제)
* 너무 긴 예제는 관대한 것이 아니라 `억압적`으로 보임
* 라운드당 2~3개의 코드 예제로 제한
  * 모든 PR에 예제를 제공하면 저자가 코드를 작성 할 수 없다고 생각한다는 신호

### 리뷰의 범위를 존중하라

* 자주 보이는 Anti-Pattern
  * PR 근처의 코드를 보고 저자에게 수정을 요청
* Rule of thumb
  * PR에 포함되지 않은 라인은 리뷰 범위가 아님
* 예외 : PR이 둘러싼 코드에 영향을 미칠 때

### 태그를 활용

* [Nit]
  * `고치면 좋지만 아니어도 그만`을 의미
  * 그래도 보통은 고침
  * 리뷰어는 `항상 더 개선할 수 있는 의견을 자유롭게 남길 수 있어야함`
  * 중요치 않다면 `Nit` 태그로 남겨서 저자가 무시할 수 있도록 할 수 잇음
  * 교육적인 목적이나 개발자들이 지속적으로 기술을 연마하는 것을 돕는 목적
  * 예) nit: null 대신 Optional을 쓰면 어떨까요?, OCP 준수를 위해 Strategy 도입은 어떨까요?

### 한두 등급만 코드 레벨을 올리는 것을 목표로
* D 등급의 PR을 받으면 저자가 C나 B 등급을 받도록 도와라
  * Letter Grade
* `완전하지는 않아도 충분히 좋은 코드`가 되도록
* F
  * 기능적으로 틀렸거나
  * 너무 복잡해서 정합성에 확신이 없는 상태
* `승인을 보류하는 유일한 이유`
  * 수 차례의 리뷰 라운드 후에도 코드가 F 상태인 경우


## 피드백 방법
### 절대 "너"라고 하지마라

* 리뷰의 핵심
  * `"무엇이 코드를 나아지게 하는가?"`
  * "누가 그런 아이디어(잘못)를 냈는지"가 아님
* 저자의 방어 유발을 최소화하는 방법으로 피드백
  * `비판의 대상은 코드.` 저자가 아님
  * "너" : 저자의 주의를 코드에서 자신으로 바꿈
* "너"만 빼라(저자에 대한 판단 -> 단순한 정정)
  * I Message 대화법 : 행동 - 결과 - 감정
* ~하는 것을 제안합니다. `하는게 어떨까요?` : 오픈 커뮤니케이션

### 건설적인 피드백을 하라

* 동료들 간의 코드 리뷰
  * not 경쟁 유발 but 팀의 생산성을 높이는 것
* 코드 리뷰를 자신의 코드에 대해 `비판`이 아니라 `학습의 과정`으로 인지하면 전체적인 프로젝트의 성공을 기여함
* `건설적인 피드백`은 개발자들이 그들의 실수에서 배우고 역량을 증대하도록 `동기부여`함
* 건설적인 피드백을 못하겠으면 차라리 아무말을 하지마라.

### 진정한 칭찬을 해라

* 대부분의 리뷰어가 `잘못된 부분에만 집중`
  * 하지만 리뷰는 긍정적 행위 강화를 위한 값진 기회이기도 함
* PR에서 좋은 변경이 있을 때마다
  * "오 이런 API가 있나요. 정말 유용해요"
  * "정말 좋은 해결책이네요. 생각도 못 했네요"
  * "함수를 분리하는 것은 좋은 생각이에요. 훨씬 단순해졌어요."
* 저자가 `주니어 혹은 신규 입사자`라면 리뷰에 매우 `민감`하고 `방어적`일 수 있음
* `진심어란 칭찬`은 리뷰어가 잔인한 감시자가 아니라 `도와주려는 팀동료`라는 것을 보여서 이런 긴장감을 낮춤

### 피드백은 명령이 아니라 요청으로 표현해라

* `일상에서 동료에게 명령하지 않음`
  * 명령형(How) : "12번 테이블 자리가 비어있습니다. 우리 가족은 저기에 앉을 것입니다."
  * 요청(What) : "네명 앉을 자리 부택해요" <- `선언형. Tell, Don't Ask`
* 하지만 리뷰에서는 강압적인 명령이 종종 발견됨
  * ex. 이 클래스를 별도의 파일로 `분리하라`
  * -> 이 클래스를 별도의 파일로 `분리할 수 있을까요?`
  * or 이 클래스는 너무 커지는 것 같은데 `괜찮을까요?` <- 나의 걱정

### 의견이 아니라 원칙에 기반하여 피드백하라

* 저자에게 의견을 줄 때는
  * `"제안하는 변경"`과 "변경의 `이유`"를 모두 설명하라
  * ex) 이 클래스를 2개로 분리해야 해요
  * -> 지금 이 클래스는 파일 다운로드와 파싱의 `2가지 책임`을 가지고 있어요. 다운로더와 파서 2개의 클래스로 분리하여 `SRP`를 준수하는 것이 어떨까요?
* SW는 과학인 동시에 예술
  * 항상 원칙에 기반하여 정확히 뭐가 잘못 되었는지 언급할 수 있는 것은 아니다.
  * 단지 그냥 보기 싫거나 `직관적이지 않을 수 있다.`
  * 무엇을 할 수 있을지 `객관적으로 설명`하라
  * ex) 이 코드는 혼란스럽네요(너?) -> `나는` 이 코드를 이해하기 어렵네요(I Message)

### 반복적인 패턴에 대해서 피드백을 제한하라
* 저자의 실수가 `동일한 패턴`임을 식별 했다면 모든 경우를 언급하지는 말라
* 동일 패턴에 대해서 2~3개 정도의 예를 언급하라.
* 그 이상은 저자에게 개별 사례가 아니라 패턴에 대해서 수정을 요구하라
* 10개가 있으면 1개만 말하고 10개를 모두 찾아서 고치라고 하지 말라

## 교착상태 시

### 교착상태를 적극적으로 처리해라
* 교착상태로 향하는지 나타내는 표식
  * 토론의 `톤`이 점차 팽팽해지고 `공격적`으로 됨
  * 라운드당 `커멘트가 줄어들지 않는 경향`을 보임
  * 너무 많은 커멘트에 `저항`이 보임
* 코드 리뷰의 `최악의 결과`는 교착상태(Stalemate)
  * 커멘트를 반영하지 않으니 승인 거부
  * 저자는 커멘트 반영을 거부
* `만나서 얘기하라`
  * 화상 혹은 만나서 논의(특히 복잡한 리뷰)
  * `텍스트 기반` 의사소통은 `상태가 인간`이라는 것을 잊게 함

### 교착상태를 적극적으로 처리해라

* 인정하거나 Escalate하라
  * 교착상태가 길어지면 `관계가 나빠짐(퇴사)`
  * 그냥 승인하는 비용(`Agree to disagree` - 갈등 해결책)
  * 저수준 코드를 무심코 승인하면 `SW품질`이 낮아질 수 있음
  * 동료와 너무 다퉈서 `함께 일하지 않게 된다면` 고수준의 품질을 얻을 수 없음
* 인정이 불가한 경우
  * 저자에게 논의를 팀장이나 테크 리더에게 `Escalation`
  * `다른 리뷰어에게 할당`
* 교착상태로 부터 회복
  * 상황을 관리자와 논의 하라
  * 휴식을 가져라. 가능하다면 안정될 때까지 PR을 서로 보내지 마라
  * 갈등 해결책을 학습하라
* 설계 리뷰를 고려하라
  * 코드 리뷰 때 설계 리뷰 때 논의되었아야 할 사항을 논쟁하는가?
  * 설계 리뷰는 있었나?
* 아주 심각하지 않다면
  * 그냥 인정하고 좋은 관계로 동료와의 협업을 지속해라
  * Agree to disagree

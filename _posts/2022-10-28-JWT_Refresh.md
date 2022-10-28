---
title: JWT의 Access Token과 Refresh Token
author: Jandari
date: 2022-10-28 13:30:00 +09:00
categories: [보안, 인증]
tags: [보안, 인증, Token, JWT]
math: true
mermaid: true
---

# 소개

JWT를 사용하여 로그인을 구현하게 되면 아래와 같은 문제점에 부딪히게 됩니다.

Token 방식의 인증은 제 3자에게 탈취 당할 경우 `보안에 매우 취약합니다.`

보안 상의 이유로 Access Token의 만료기간을 짧게 가진다면 사용자는 Token이 만료 될 때마다 매번 로그인을 새롭게 해야합니다.

만료기간을 늘리면, Token을 탈취 당했을 때 보안에 더 취약해집니다.

`유효기간을 짧게 하면서 좋은 방법이 있을까?`라는 질문에 답은 바로 `Refresh Token`을 사용하는 방법 입니다.



## Access Token과 Refresh Token의 재발급 원리

#### 1. 로그인을 하게 되면 Access Token과 Refresh Token을 모두 발급합니다.

Refresh Token은 서버쪽에서 DB에 저장하며, 클라이언트 쪽에서는 Access Token과 Refresh Token을 쿠키 또는 웹 스토리지에 저장합니다.

#### 2. 사용자가 인증이 필요한 API에 접근한다면 아래와 같이 유효성을 검사 합니다.

||Case|동작|
|-|-|-|
|1|Access Token과 Refresh Token 모두 만료가 된 경우|다시 로그인 하도록 유도하고 Access/Refresh Token 모두 새로 발급 합니다.|
|2|Access Token은 만료 됐지만, Refresh Token은 유효 할 경우|Refresh Token을 검증하여, Access Token을 재발급 합니다.|
|3|Access Token은 유효하지만, Refresh Token이 만료 됐을 경우|Access Token을 검증하여 Refresh Token을 재발급 합니다.|
|3|Access Token과 Refresh Token 모두 유효 할 경우|정상적으로 사용 가능 합니다.|


#### 3. 로그아웃을 하게 되면 Access/Refresh Token을 모두 만료시킵니다.

## Refresh Token 인증 과정

#### 정상적일 때

![image](/assets/img/post/2022-10-28-JWT_Refresh/1.jpg)

#### Access Token이 만료 되었을 때

![image](/assets/img/post/2022-10-28-JWT_Refresh/2.jpg)

## 참고

* https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-Access-Token-Refresh-Token-%EC%9B%90%EB%A6%AC-feat-JWT
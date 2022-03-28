---
title: ASUS 보안 부팅 해제
author: Jandari
date: 2021-03-25 11:04:00 -0500
categories: [DeepLearning]
tags: [CUDA, Setup]
math: true
mermaid: true
---

이 글은 RHEL(Red Hat Enterprise Linux)에 Tensorflow를 설치하기 위한 글입니다.

먼저 NVIDIA를 Linux에 설치하기 위해서는 BIOS에서 보안부팅을 해제해야 합니다.

## ASUS SECURE BOOT 해제

1. 서버 부팅 시 검은 화면에서 `[DEL]` 키 혹은 `[F2]` 키를 연타하게 되면 BIOS 부팅 메뉴에 진입 할 수 있습니다.
2. 아래와 같은 BIOS 메인 메뉴에서 마우스로 우측 하단의 [Advanced Mode(F7)]를 클릭 하거나 `F7` 키를 누릅니다.

![image](/assets/img/post/2022-03-25-asus_secure_boot/1.jpg)

3. `[Boot] or [부팅]`을 클릭 합니다.

![image](/assets/img/post/2022-03-25-asus_secure_boot/2.jpg)

4. `[Secure Boot] or [보안 부팅 메뉴]`를 클릭합니다.

![image](/assets/img/post/2022-03-25-asus_secure_boot/3.jpg)


5. `[Secure Boot State] or [안전 부팅 상태]`를 보시면 활성화가 되어 있는 것을 확인 할 수 있습니다.

6. `[Key Management] or [키 관리]`를 클릭합니다.

![image](/assets/img/post/2022-03-25-asus_secure_boot/4.jpg)

7. 키 관리 메뉴에서 `[Clear Secure Boot Keys] or [안전 부팅 키 지우기]` 메뉴를 클릭 후 `[Yes]`를 클릭합니다.

![image](/assets/img/post/2022-03-25-asus_secure_boot/5.jpg)

8. `[Secure Boot State] or [안전 부팅 상태]`를 확인 후 `[F10]`키를 눌러 `[Save & reset]` 창이 뜨면 `[OK]`를 클릭 합니다.

![image](/assets/img/post/2022-03-25-asus_secure_boot/6.jpg)

## 이미지 출처

* [https://blog.naver.com/tolikeu/222292344943](https://blog.naver.com/tolikeu/222292344943)
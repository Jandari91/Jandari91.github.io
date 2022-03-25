---
title: RHEL(RedHat Enterprise Linux) 설치
author: Jandari
date: 2021-03-25 15:04:00 -0500
categories: [DeepLearning]
tags: [RHEL, Setup, Linux, Cuda]
math: true
mermaid: true
---

이 글은 RHEL(Red Hat Enterprise Linux)에 Tensorflow를 설치하기 위한 글입니다.

먼저 이전에 작성한 블로그를 참고하여 BIOS에서 보안부팅을 해제해야 합니다.

## RHEL iso 다운로드

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/1.png)

[https://developers.redhat.com/products/rhel/download](https://developers.redhat.com/products/rhel/download)

RHEL은 기존에는 한 사용자당 한 대의 소프트웨어 개발 목적으로만 한정되어 있었지만, 앞으로는 최대 16개의 프로덕션 환경에서 사용 할 수 있다.

RHEL을 무료로 사용하려면 Individual Developer Subscription에 등록해야 하고 등록 비용은 발생하지 않는다.

지금까지 개인 개발자만 대상으로 Red Hat Developer 프로그램을 제공하였으나 앞으로는 고객의 시스템을 개발하는 개발팀도 등록 가능하다.

위 링크에서 iso 파일을 다운 받습니다.

## 부팅 USB 만들기

저는 `Rufus`를 사용하여 부팅 USB를 만들도록 하겠습니다.

[https://rufus.ie/ko/](https://rufus.ie/ko/)

위 링크에서 다운 받은 후 실행 합니다.


![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/2.png)

1. 장치에서 자신이 원하는 USB를 선택합니다.


![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/3.png)

2. 선택 버튼을 클릭하여 다운로드 받은 iso 파일을 선택합니다.

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/4.png)

3. 시작 버튼을 클릭합니다.

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/5.png)

## RHEL(Red Hat Enterprise Linux) 설치

서버에 부팅 USB를 연결하고 전원 버튼을 누릅니다.

저의 경우 메인보드가 ASUS 이기 때문에 `F8`을 누르면 부팅 메뉴가 나타납니다.

여기서 자신의 USB를 선택 합니다.

1. `Install Red Hat Enterprise Linux 8.5`를 선택 합니다.

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/6.png)

2. 저는 영어로 설치하기 위해 `English`로 선택 했습니다.

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/7.png)

3. 설치 메인 메뉴를 확인 할 수 있다.

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/8.png)

4. `Time & Date` 메뉴에서 서울로 시간을 변경합니다.

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/9.png)

5. 먼저 Network를 활성화 하기 위해 `Network & Host Name`을 클릭하여 활성화 시킨다.

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/10.png)

6. `Root Password` 메뉴에서 패스워드를 설정한다.

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/11.png)

7. `Create User` 메뉴에서 아이디와 암호를 지정한다. 또한 `Make this user administrator`를 체크하여 일반 계정으로 admin 권한을 사용 할 수 있도록 지정한다.

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/12.png)

8. `Installation Destination` 메뉴를 선택하여 파티션을 나눕니다.

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/13.png)

`RHEL`에서 파티션을 나누는 방법은 3가지가 있습니다.

|종류|설명|
|-|-|
|Standard Partition|물리적은 볼륨을 여러개의 논리 드라이브로 나누는 리눅스의 일반적인 파티셔닝 방식|
|LVM|디스크를 표준파티션보다 더 효율적이고 유연하게 관리할 수 있는 방식으로 필요에 따라 사용자가 파티션 용량을 줄이거나 늘리는게 간단하고 여러 물리적 볼륨을 하나의 논리적 볼륨으로 묶을 수 있는 방식|
|LVM Thin Provisioning|디스크 저장공간의 낭비를 해결하기위해 나온 방식으로 볼륨에 특정 용량을 할당을 하더라도 사용하는 사용자가 할당받은 그 용량을 전부 사용하지 않고 일정부분 남겨둔다면 그 만큼 낭비가 발생하는 것이기 때문에 볼륨에 10gb의 용량을 할당을 한다고 하면 실제 할당한 크기는 10gb가 아니라 현재 사용자가 사용하는 만큼의 용량이고 디스크에 파일을 저장을 해서 용량이 늘어나면 그때마다 유동적으로 저장공간 크기가 늘어남|


저는 Standard Patition으로 생성하도록 하겠습니다

9. 전 총 20GB이기 때문에 아래와 같이 나눴습니다.

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/14.png)


|종류|설명|
|-|-|
|root|모든 리눅스 파일체제의 최상위 디렉토리|
|boot|부팅에 필요한 커널, 모듈 등이 존재하는 디렉토리|
|home|사용자의 홈 디렉토리가 위치하는 디렉토리|
|var|로그 및 사용자 데이터, 가변적인 데이터들이 저장되는 공간|
|swap|메모리 용량이 부족할 때 하드디스크 일정공간을 스왑메모리로 사용|
|biosboot|GPT로 장치를 부팅하는데 필요한 파티션|


끝난 후 좌측 상단의 `Done`을 클릭하여 끝냅니다.


10. `Installation Source`의 경우 설치 시 Software를 Network로 설치하거나 iso에 포함 된 Software를 설치하거나 선택하는데 전 iso 포함이므로 설정하지 않겠습니다.

11. `Software Selection` 버튼을 클릭하여 원하는 소프트웨어를 선택 합니다.

전 Server용도에 FTP Server만 있으면 되기 때문에 FTP만 선택하겠습니다.

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/15.png)

12. 이제 모든 셋팅이 끝났고 메인 메뉴에서 `Begin Installation` 버튼으로 설치를 시작합니다.

설치가 끝나면 `Rebbot System`으로 시작합니다.

![image](/assets/img/post/DeepLearning/2022-03-25-RHEL_setup/16.png)






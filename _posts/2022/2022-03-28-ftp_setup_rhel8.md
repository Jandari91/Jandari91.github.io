---
title: RHEL8.5 FTP 설치
author: Jandari
date: 2022-03-28 17:21:00 -0500
categories: [Infra, FTP]
tags: [FTP, RHEL8]
math: true
mermaid: true
---

## vsftpd 패키지 설치

먼저 vsftpd가 실행 중인지 확인 합니다.

```
$ ps -ax | grep vsftpd

  46688 pts/1    S+     0:00 grep --color=auto vsftpd
```

grep 명령어만 뜨는 걸 봐서는 설치가 되어 있지 않습니다.

dnf 명령어를 통해 설치 합니다.

```
$ sudo dnf -y install vsftpd
```

## FTP 서버 설정

config 파일을 설정하여 줍니다.

```
$ sudo vi /etc/vsftpd/vsftpd.conf
```

아래 설정 3개를 변경하여 줍니다.

```
anonymous_enable=NO 
local_enable=YES
write_enable=YES
local_umask=022
anon_upload_enable=YES
dirmessage_enable=YES
xferlog_enable=YES
xferlog_file=/var/log/xferlog
connect_from_port_20=YES
listen=YES
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
```

`chroot_list`에 허용할 아이디 리스트를 입력하여 줍니다.

```
$ sudo vi /etc/vsftpd/chroot_list
```

```
mirero
```

## 방화벽 설정

기본적으로 FTP는 21번 포트를 사용 합니다.

`firwall-cmd` 명령어로 방화벽을 해제 하여 줍니다.

```
$ sudo firewall-cmd --permanent --add-service=ftp
success
$ sudo firewall-cmd --permanent --add-port=21/tcp
success
$ sudo firewall-cmd --reload
success
```

## SELINUX 해제

`SELINUX`는 보안을 강화해주는 커널입니다.

`zero-day` 공격 및 `buffer overflow` 등 어플리케이션 취약점으로 인한 해킹을 방지해주는 핵심 구성요소 입니다.

```
$ sudo vi /etc/selinux/config
```

`SELINUX=disabled`로 변경한다.


## 서비스 시작

```
$ sudo systemctl restart vsftpd 
```
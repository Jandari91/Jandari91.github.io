---
title: Docker Container Exit Code
author: Jandari
date: 2022-03-31 11:15:00 -0500
categories: [Docker]
tags: [Docker]
math: true
mermaid: true
---


## 소개

이번 포스팅에서는 Docker Container의 Exit Code에 대해 알아봅니다.

Docker를 사용하다보면 예기치 않게 Container가 내려가거나 내려가 있어도 무슨 원인으로 내려갔는지 모를 경우가 많습니다.

이때 Exit Code를 확인하여 원인을 알아 볼 수 있습니다.

## Docker 종료 된 Container 목록 확인

```
> docker ps -a --filter "status=exited"
```

`docker ps -a`로 모든 컨테이너 목록을 가져오고 `--filter` 옵션으로 exited 된 컨테이너 목록만 가져 올 수 있습니다.

## Exit Code 종류

#### Exited (0)

`0` 코드는 가장 일반적인 경우로, 컨테이너 내부의 `init process`가 자신의 역할을 끝낸 후 정상적으로 종료 되었을 때, 또는 docker stop 명령어로 종료 시켰을 때 해당됩니다.

#### Exited (125)
`125` 코드는 `docker run` 명령어의 실패로 실제 docker container가 띄워지지 않았을 때 발생합니다.

#### Exited (126)

`126` 코드는 Container 내부의 명령어를 실행하지 못하는 경우 발생합니다.

권한, 접근, 파일, 옵션 등등이 발생 가능합니다.

#### Exited (127)

`127` 코드는 Container 내부의 명령어 자체가 없을 경우 발생합니다.

#### Exited (128 + n)

`128 + n`는 Linux Signal에 의해 종료 되었을 때 발생합니다.

n의 의미즌 Linux Signal Number를 의미합니다.

#### Exited (130)

`128 + 2` 코드는 컨테이너가 실행되었을 때, 인터럽트 루틴(Control + C) 에 의해 init 프로세스가 전달된 경우이다. 인터럽트 신호인 SIGINT의 코드가 2이므로 종료 코드는 130이 된다. 사실상 개발자가 의도적으로 컨테이너를 종료시키는 경우가 아니면 거의 볼 수 없다. 

mysql과 같은 컨테이너를 interactive하게 (-it) 생성하고 Control + C를 연속적으로 입력하면 이 코드를 인위적으로 만들어 낼 수 있다.

#### Exited (137)

`128 + 9`로 Linux Signal num 9인 SIGKILL이 발생하였을 경우입니다. 자주 사용하는 kill -9로 인해 종료되었을 경우를 의미합니다. docker kill, docker rm -f를 사용했을때 발생 가능합니다.


#### Exited (143)

`128 + 15`로 Linux Signal num 15인 SIGTERM이 발생하였을 경우입니다.


#### Exited (255)
`255` 코드는 종료 코드가 범위를 벗어나는 경우 발생합니다.
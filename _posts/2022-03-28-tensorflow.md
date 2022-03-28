---
title: RHEL8.5 Python 및 Tensorflow 설치
author: Jandari
date: 2021-03-28 10:00:00 -0500
categories: [DeepLearning]
tags: [CUDA, Setup, Linux, Python, Tensorflow]
math: true
mermaid: true
---

이 글은 RHEL(Red Hat Enterprise Linux)에 Tensorflow를 설치하기 위한 글입니다.

먼저 이전에 작성한 블로그를 참고하여 CUDA를 설치하여야 합니다.

## Python 설치

RHEL에 python을 설치하기 위해서는 `yum` 또는 `dnf`를 통해 쉽게 설치가 가능합니다.

```
$ sudo yum -y install python3
$ sudo dnf -y install python3
```

하지만 `2022-03-28` 기준 최신 버전인 `3.10.4`를 설치하도록 하겠습니다.

#### Python Source 설치 전 패키지 설치

```
$ sudo dnf -y install gcc openssl-devel bzip2-devel libffi-devel
```

해당 패키지는 소스 파일의 `Makefile`을 컴파일하기 위해 필요한 패키지 입니다/

패키지를 설치 하지 않으면 `make`파일을 컴파일 할 때 아래와 같은 에러가 발생할 수 있습니다.

```
Python-3.10.4/Modules/_ctypes/_ctypes.c:107:10: fatal error: ffi.h: No such file or directory
 #include <ffi.h>
          ^~~~~~~
compilation terminated.
```

#### Python 소스파일 다운로드

먼저 Python Download 사이트로 들어가 해당 버전의 url를 가져옵니다.

[https://www.python.org/downloads/source/](https://www.python.org/downloads/source/)

![image](/assets/img/post/2022-03-28-tensorflow/1.jpg)

해당 url로 들어간 후 가장 아래로 스크롤을 내리면 `Files`라고 보입니다.

OS에 맞춰 파일을 다운 받을 수 있는데 저는 `tar.xz` 파일을 받도록 하겠습니다.

![image](/assets/img/post/2022-03-28-tensorflow/2.jpg)

해당 부분에서 마우스 오른쪽 키로 링크를 복사 한후 wget으로 다운 받습니다.

```
$ sudo wget https://www.python.org/ftp/python/3.10.4/Python-3.10.4.tar.xz
$ ls
Python-3.10.4.tar.xz
```

#### Python 설치

위에서 받은 압축을 풀어 줍니다.

```
$sudo tar xvf Python-3.10.4.tar.xz
```

압축이 풀린 폴더로 들어가 `Makefile` 파일을 만들기 위해 `configure`를 실행합니다.

```
$ cd Python-3.10.4
$ ./configure --enable-optimizations
$ ls | grep Makefile
Makefile
Makefile.pre
Makefile.pre.in
```

`Makefile`을 컴파일 및 설치를 진행합니다.

```
$ sudo make altinstall
$ which python3.10
/usr/local/bin/python3.10

$ python3.10

Python 3.10.4 (main, Mar 28 2022, 09:47:54) [GCC 8.5.0 20210514 (Red Hat 8.5.0-4)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

설치가 정상적으로 되었고 실행도 정상적으로 가능합니다.


## Tensorflow 설치

#### 가상환경 설정

Tensorflow에서는 Python을 가상환경으로 설치하여 사용하는 것을 권장한다.

그래서 아래와 같은 명령어로 가상환경을 설치하여 Tensorflow를 설치하겠습니다.

```
$ python -m venv --system-site-packages ~/python/venv
$ source ~/python/venv/bin/activate
```

가상 환경을 활성화하게 되면 프롬프트가 `(venv)`로 변경됩니다.

```
(venv) $ pip install --upgrade pip
(venv) $ pip install --upgrade tensorflow
(venv) $ python -c "import tensorflow as tf;print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
```


#### 가상환경 python 기본으로 설정

하지만 `python3.10`이라는 명령으로 사용해야 하기 때문에 `alias`를 변경하여 `python`으로만 실행이 가능하도록 설정합니다.

현재 가상환경은 `~/python/venv`에 설정이 되어 있습니다.

```
$ vi ~/.bashrc
```

가장 아래에 `alias python=""`를 추가합니다.

```
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# User specific environment
if ! [[ "$PATH" =~ "$HOME/.local/bin:$HOME/bin:" ]]
then
    PATH="$HOME/.local/bin:$HOME/bin:$PATH"
fi
export PATH

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions

alias python="$HOME/python/venv/bin/python3.10" # 추가
```

이제 설정 파일을 적용합니다.

```
$ source ~/.bashrc
$ python

Python 3.10.4 (main, Mar 28 2022, 09:47:54) [GCC 8.5.0 20210514 (Red Hat 8.5.0-4)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>import tensorflow
>>>
```

#### pip 오프라인 설치

`pip`를 사용 할 때 아래와 같은 에러가 뜰 경우가 있다.

이것은 사내에서 인트라넷을 사용하는 경우 pip 패키지에서 신뢰 할 수 없어 패키지 다운로드가 안되는 것입니다.

이런경우 오프라인으로 설치가 가능합니다.

기존의 pip로 패키지를 다운받아 사용 중이던 PC에서 모듈 리스트를 추출합니다.

```
$ pip freeze > requirements.txt
```

requirements.txt 파일을 열어보면 아래와 같이 설치되어 있는 패키지의 목록이 나오게 됩니다.

```
absl-py==1.0.0
astunparse==1.6.3
(... 생략 ...)
tensorboard==2.8.0
tensorboard-data-server==0.6.1
tensorboard-plugin-wit==1.8.1
tensorflow==2.8.0
tensorflow-datasets==4.5.2
tensorflow-io-gcs-filesystem==0.24.0
tensorflow-metadata==1.7.0
tensorflow-serving-api==2.8.0
termcolor==1.1.0
```

해당 목록의 패키지를 다운로드 받습니다.

```
$ pip download -r requirements.txt
```

그럼 `.whl`이나 `.tar.gz`으로 현재 경로에 다운로드가 받아집니다.

이제 모든 파일을 오프라인으로 설치하고 싶은 PC로 이동시킵니다.

```
$ python -m pip install --no-index --find-links="./" -r requirements.txt
```

requirements.txt에 있는 목록이 일괄 설치 되게 됩니다.

```
python -m pip install --no-index --find-links="./" schedule-0.6.0-py2.py3-none-any.whl
```

위와 같이 특정 모듈을 지정하여 설치도 가능합니다.




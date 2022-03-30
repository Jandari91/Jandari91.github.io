---
title: NUMA node read from SysFS had negative value -1
author: Jandari
date: 2021-03-30 08:30:00 -0500
categories: [DeepLearning]
tags: [CUDA, Setup, Linux, TroubleShooting]
math: true
mermaid: true
---

## 환경

* OS : RHEL 8.5
* GPU : Nvidia GeForce GTX 1060 6GB
* CUDA : 11.6


## 문제발생

Tensorflow를 실행 시킬 때 아래와 같은 로그가 뜨면서 GPU 동작을 하지 않고 CPU로만 동작합니다.

```
Found 3670 files belonging to 5 classes.
Using 2936 files for training.
2022-03-29 16:21:12.425272: I tensorflow/stream_executor/cuda/cuda_gpu_executor.cc:936] successful NUMA node read from SysFS had negative value (-1), but there must be at least one NUMA node, so returning NUMA node zero
Skipping registering GPU devices...
2022-03-29 16:21:12.448855: I tensorflow/core/platform/cpu_feature_guard.cc:151] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX2 FMA
To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
Found 3670 files belonging to 5 classes.
Using 734 files for validation.
```

일단 위 로그를 확인해보면 아래와 같이 해석이 됩니다.

```
NUMA node 정보가 올바르지 않지만, 너 적어도 한 개의 NUMA node 있다.
그러니 일단 되게끔 해 보겠다.
```

#### NUMA (Non-Uniformed Memory Access) 란?


> 불균일 기억 장치 접근(Non-Uniform Memory Access, NUMA)는 멀티프로세서 시스템에서 사용되고 있는 컴퓨터 메모리 설계 방법중의 하나로, 메모리에 접근하는 시간이 메모리와 프로세서간의 상대적인 위치에 따라 달라진다. NUMA구조에서 프로세서는 자기의 로컬 메모리에 접근할 때가 원격 메모리에 접근할 때보다 더 빠르다. 원격 메모리는 다른 프로세서에 연결되어 있는 메모리를 말하고 로컬 메모리는 자기 프로세서에 연결되어 있는 메모리를 말한다.

즉, 하나의 메인보드에서 여러 프로세서를 사용하면서 메모리 접근 효율을 높이기 위한 기술로 특정 프로세서가 메모리를 다 잡게되면 버스를 자기 혼자 독점하고있으니 다른 프로세서는 놀아야하는 상황이 발생하기 때문에 각 프로세서마다 메모리 구역을 나눠주고 ‘여기만 접근해’ 라고 지정하고 그걸 NUMA 노드라고 함.


## 해결방법

```
$ lspci | grep -i nvidia

01:00.0 VGA compatible controller: NVIDIA Corporation GP106 [GeForce GTX 1060 6GB] (rev a1)
01:00.1 Audio device: NVIDIA Corporation GP106 High Definition Audio Controller (rev a1)
```

첫번째 줄 VGA 호환 장치, NVIDIA Geforce 의 주소가 01:00.0 으로 나온다. 이는 각자 다를 것이므로 이 부분을 잘 변경해 주자.

#### NUMA 설정 값 확인 및 변경

```
$ cd /sys/bus/pci/devices/
$ ls

0000:00:00.0  0000:00:14.2  0000:00:1b.0  0000:00:1f.0  0000:00:1f.5  0000:01:00.1
0000:00:01.0  0000:00:16.0  0000:00:1c.0  0000:00:1f.3  0000:00:1f.6  0000:04:00.0
0000:00:14.0  0000:00:17.0  0000:00:1d.0  0000:00:1f.4  0000:01:00.0

```

위에서 확인한 01:00.0 이 보인다. 다만 앞에 0000:이 붙어있다.

연결되어있는지 다시 확인해 본다.

```
$ cat /sys/bus/pci/devices/0000\:01\:00.0/numa_node

-1

```

-1은 연결이 안되있다는 의미, 0이 잘 되어있다는 의미다. 즉, 안되어있다.

아래의 명령어로 고쳐 준다.


```
$ echo 0 | sudo tee -a /sys/bus/pci/devices/0000\:01\:00.0/numa_node
$ cat /sys/bus/pci/devices/0000\:01\:00.0/numa_node

0
```


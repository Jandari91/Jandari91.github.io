---
title: Could not load dynamic library 'libcudnn.so.8'
author: Jandari
date: 2021-03-30 08:50:00 -0500
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
2022-03-29 16:21:12.448227: W tensorflow/stream_executor/platform/default/dso_loader.cc:64] Could not load dynamic library 'libcudnn.so.8'; dlerror: libcudnn.so.8: cannot open shared object file: No such file or directory; LD_LIBRARY_PATH: :/usr/local/cuda-11.6/lib64
2022-03-29 16:21:12.448246: W tensorflow/core/common_runtime/gpu/gpu_device.cc:1850] Cannot dlopen some GPU libraries. Please make sure the missing libraries mentioned above are installed properly if you would like to use GPU. Follow the guide at https://www.tensorflow.org/install/gpu for how to download and setup the required libraries for your platform.
Skipping registering GPU devices...
2022-03-29 16:21:12.448855: I tensorflow/core/platform/cpu_feature_guard.cc:151] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX2 FMA
To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
Found 3670 files belonging to 5 classes.
Using 734 files for validation.
```

일단 위 로그를 확인해보면 아래와 같이 해석이 됩니다.

```
Could not load dynamic library 'libcudnn.so.8'; dlerror: libcudnn.so.8: cannot open shared object file: No such file or directory; LD_LIBRARY_PATH: :/usr/local/cuda-11.6/lib64
```

> 동적라이브러리 `libcudnn.so.8`을 로드할 수 없습니다. 해당 파일 또는 디렉토리가 없어 공유 객체를 열수 없습니다. 경로는 LD_LIBRARY_PATH: :/usr/local/cuda-11.6/lib64 입니다.

이건 CuDNN을 설치하지 않았거나 잘 못 설치가 되었을 때 생깁니다.

## CuDNN 설치

[https://developer.nvidia.com/rdp/cudnn-archive](https://developer.nvidia.com/rdp/cudnn-archive)

해당 사이트에서 tar로 되어 있는 파일로 설치하거나 rpm으로 받아 설치하는 방법이 있습니다.

![image](/assets/img/post/2022-03-28-cuda/1.jpg)

저는 tar 파일을 다운받아 설치하도록 하겠습니다.

#### tar 설치

`Local Installer for Linux x86_64 (Tar)` 파일을 다운 받아 리눅스 서버로 전송합니다.

`wget`으로는 리눅스에서 직접 받아지지 않습니다.

```
$ tar -xvf cudnn-linux-x86_64-8.3.2.44_cuda11.5-archive.tar.xz
$ cd cudnn-linux-x86_64-8.3.2.44_cuda11.5-archive/
$ sudo cp include/cudnn*.h /usr/local/cuda-11.6/include
$ sudo cp lib/libcudnn* /usr/local/cuda-11.6/lib64
sudo chmod a+r /usr/local/cuda-11.6/include/cudnn*.h /usr/local/cuda-11.6/lib64/libcudnn*
```


---
title: TorchServe 맛보기
author: Jandari
date: 2021-03-23 17:04:00 -0500
categories: [PyTorch, TorchServe]
tags: [PyTorch, TorchServe]
math: true
mermaid: true
---
## TorchServe 맛보기

TorchServe를 간단히 시작하기 위한 정리입니다.
[pytorch/serve](https://github.com/pytorch/serve)에서 제공해주는 학습 모델을 사용하여 RestApi로 추론하는 방법입니다.
Torchserve는 Docker 이미지를 사용합니다.

#### 참고
[https://github.com/pytorch/serve/tree/master/examples/image_classifier/densenet_161](https://github.com/pytorch/serve/tree/master/examples/image_classifier/densenet_161)


## 모델압축

모델을 학습하게 되면 로컬에 pth 파일이 생성이 됩니다. 이 모델 파일을 그대로 TorchServe에서는 사용 할 수 없기 때문에 먼저 Training이 끝난 모델을 압축해야 합니다.

[densenet161 학습 모델 다운로드](https://download.pytorch.org/models/densenet161-8d451a50.pth)

[model.py](https://github.com/pytorch/serve/blob/master/examples/image_classifier/densenet_161/model.py)


TorchServe에서 제공해주는 모델을 다운로드 받습니다.

저는 `D:\save_model` 경로에 넣어두겠습니다.

로컬 경로와 `pytorch/torchserve` 컨테이너의 볼륨을 -v 옵션으로 마운트 후 컨테이너를 실행 시킵니다.

```
> docker run --rm -it -v D:\save_model:/models/model pytorch/torchserve bash
```

|옵션|설명|
|-|-|
|--rm | 컨테이너가 내려가면 바로 컨테이너 삭제|
|-it | -i와 -t를 동시에 사용해 컨테이너 터미널에 붙어 명령어를 사용가능하다.|
|-v | 호스트의 로컬과 컨테이너의 디렉터리를 연결한다(마운트)|
|bash | -it를 사용하기 위해 마지막 bash 명령을 실행시킵니다.|


컨테이너를 실행 후 컨테이너 안에서 아래의 명령어를 실행합니다.

```
$ torch-model-archiver --model-name densenet161 --version 1.0 --model-file /models/model/model.py --serialized-file /models/model/densenet161-8d451a50.pth --export-path /models/model/ --handler image_classifier
```

|옵션|설명|
|-|-|
|--model-name|모델이름 모델이름으로 mar 파일이 생성됩니다.|
|--version 1.0|버전 정보|
|--model-file|state_dict을 포함하는 .pth의 경우 사용합니다.|
|--serialized-file|.pt 또는 .pth 파일 위치|
|--export-path|mar 파일 생성 위치|
|--handler|data가 들어왔을 때 전처리/추론/후처리를 관리하는 파일 커스텀으로 생성이 가능합니다.|

`D:\save_model`에 `densenet161.mar` 파일이 생성됐다면 준비는 다 끝난겁니다.

## TorchServe 실행

이제 Inference Serving을 하기 위해 다시 컨테이너를 띄웁니다.

```
> docker run --rm -p 8080:8080 -p 8081:8081 -v D:\save_model\:/tmp/models pytorch/torchserve torchserve --model-store=/tmp/models/ --models /tmp/models/densenet161.mar
```

```
Torchserve version: 0.5.3
TS Home: /home/venv/lib/python3.8/site-packages
Current directory: /home/model-server
Temp directory: /home/model-server/tmp
Number of GPUs: 0
Number of CPUs: 16
Max heap size: 6384 M
Python executable: /home/venv/bin/python
Config file: config.properties
Inference address: http://0.0.0.0:8080
Management address: http://0.0.0.0:8081
Metrics address: http://0.0.0.0:8082
Model Store: /tmp/models
Initial Models: /tmp/models/densenet161.mar
Log dir: /home/model-server/logs
Metrics dir: /home/model-server/logs
Netty threads: 32
Netty client threads: 0
Default workers per model: 16
Blacklist Regex: N/A
Maximum Response Size: 6553500
Maximum Request Size: 6553500
Limit Maximum Image Pixels: true
Prefer direct buffer: false
Allowed Urls: [file://.*|http(s)?://.*]
Custom python dependency for model allowed: false
Metrics report format: prometheus
Enable metrics API: true
Workflow Store: /tmp/models
Model config: N/A
2022-03-22T01:53:50,124 [INFO ] main org.pytorch.serve.ModelServer - Loading initial models: /tmp/models/densenet161.mar
```

위와 같은 로그가 찍히면 정상적으로 컨테이너 실행이 된 상태입니다.

[http://localhost:8080/ping](http://localhost:8080/ping)

```json
{
  "status": "Healthy"
}
```

위 경로에서 추론 서버의 상태 체크가 가능합니다.

[http://localhost:8080/predictions/densenet161](http://localhost:8080/predictions/densenet161)

```json
{
  "code": 503,
  "type": "InternalServerException",
  "message": "Prediction failed"
}
```

모델 경로로 들어가면 예측이 실패 했지만 정상적으로 동작하는 것을 확인 할 수 있습니다.

## REST API 추론

[추론 할 이미지 다운로드](https://github.com/pytorch/serve/blob/master/examples/image_classifier/kitten.jpg)

![](https://images.velog.io/images/jandari91/post/c7adb1e9-c988-4591-81c1-a909d7677abb/image.png)

해당 이미지를 추론하게 되면 `0` 이 나와야 합니다.

```py
import json
import requests
import numpy as np
import cv2
from PIL import Image
from io import BytesIO


def preprocess(img_path_or_buf):
    # Check whether image is a path or a buffer
    raw_image = (
        Image.fromarray(cv2.imread(img_path_or_buf))
        if isinstance(img_path_or_buf, str)
        else img_path_or_buf
    )

    # If buffer was np.array instead of PIL.Image, transform it
    if type(raw_image) == np.ndarray:
        raw_image = Image.fromarray(raw_image)

    raw_image = raw_image.convert("RGB")

    raw_image_bytes = BytesIO()
    raw_image.save(raw_image_bytes, format="PNG")
    raw_image_bytes.seek(0)

    return raw_image_bytes.read()

data  = preprocess('test_data/torch_dataset/kitten.jpg')
headers = {"content-type": "image/png"}
url = "http://localhost:8080/predictions/densenet161"


response = requests.post(url, data=data, headers=headers)

predictions = json.loads(response.text)
print('실제 값 : {} / 예측 값 : {}'.format('0', np.argmax(predictions)))
```

해당 파이썬 코드는 로컬의 이미지를 읽어 byte로 변경해 RESTAPI 경로로 보내 예측값을 가져오는 코드입니다.

```
실제 값 : 0 / 예측 값 : 0
```
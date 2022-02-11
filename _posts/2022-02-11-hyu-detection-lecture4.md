---
layout: post
title:  "[HYU 2022] Detector Practice"
date:   2022-02-11 09:00:01
categories: HYU-Jan-2022
---



# About This Lecture

이 강의는 한양대학교 대학원생들을 대상으로 "Deep Learning based Object Detection(딥러닝 기반 객체 검출)"이라는 주제로 강의를 합니다. 강의 순서는 다음과 같습니다.

1. **Introduction to Object Detection**: Object Detection에 대한 전반적인 소개를 합니다.
2. **Two-Stage Detectors**: Two-Stage Detector 모델들을 소개하고 실습해봅니다.
3. **One-Stage Detectors**: One-Stage Detector 모델들을 소개하고 실습해봅니다.
3. **Transformer-based Detectors**: Transfomer 기반으로 구현된 Detector 모델들을 소개합니다.

이 강의를 통해서 여러분들이 배울 수 있는 것은 다음과 같습니다.

- 객체 검출 모델의 발전 역사
- CNN기반의 classification + regression이 혼합된 지도 학습 방법
- 학습된 모델을 이용한 객체 검출 방법 (실습)



# Object Detection Practice (Feat. Colab x TF-Hub)

객체 검출 기술을 테스트 해보기 위해서는 이를 구동할 수 있는 시스템이 필요하다. 윈도우보다는 리눅스가 딥러닝을 활용하기에 수월하다. 기본적으로 파이썬과 딥러닝 프레임워크 하나는 설치돼있어야 한다. CPU로도 구동이 가능하지만 결과를 확인하는데 오래걸리므로 GPU가 있어야 한다. 특히 학습은 GPU가 필수다. GPU를 활용하려면 CUDA도 설치해야한다. 그래서 딥러닝을 위한 시스템 하나 세팅하기 위해서는 돈도 많이 들고 시간도 들고 관련 지식도 필요하다.  

간단히 교육/학습 목적으로 딥러닝을 써보고 싶은데 저렇게 시간과 돈을 들이기 어렵다. 이럴 때 적합한 시스템이 구글에서 제공하는 클라우드 서비스인 **Google Colaboratory** (이하 colab)이다. **무료**로 GPU를 사용할 수 있으며 파이썬, 텐서플로(Tensorflow)도 이미 설치돼있다. 사용방식은 이미 많은 사용자들이 쓰고 있는 Jupyter Notebook과 비슷하다.  

시스템이 갖춰졌더라도 딥러닝을 써보려면 어디선가 소스 코드도 받아야하고 학습된 모델 파일도 받아야하고 소스 코드마다 사용법도 배워야 한다. 적당한 소스 코드를 찾는 것도 어려운 일이다. 그래서 텐서플로에서는 TF-Hub, 파이토치(Pytorch)에서는 Model Zoo를 통해 미리 학습된 모델을 제공하고 있다. 여기서는 Colab에서 텐서플로와 **TF-Hub** 모델을 이용하여 실습을 진행한다.



## 1. 시스템 확인

딥러닝 모델을 받기 전에 시스템 하드웨어를 먼저 확인해보자. 리눅스 명령어를 쓰려면 앞에 **!** 를 붙여서 실행하면 된다.

```
!echo "===== OS 확인 ====="
!cat /etc/issue.net
!echo "===== CPU 확인 ====="
!cat /proc/cpuinfo | grep "model name"
!echo "===== 메모리 확인 ====="
!cat /proc/meminfo | grep "Mem"
!echo "===== 스토리지 확인 ====="
!df -h | grep "/dev/sda1"
!echo "===== 현재 경로 확인 ====="
!pwd
!echo "===== 현재 경로의 내용 확인 ====="
!ls
!echo "===== GPU 모델 확인 ====="
!nvidia-smi --query-gpu=name --format=csv,noheader
!echo "===== GPU 상태 확인 ====="
!nvidia-smi
```

> ```
> ===== OS 확인 =====
> Ubuntu 18.04.5 LTS
> ===== CPU 확인 =====
> model name	: Intel(R) Xeon(R) CPU @ 2.30GHz
> model name	: Intel(R) Xeon(R) CPU @ 2.30GHz
> ===== 메모리 확인 =====
> MemTotal:       13302912 kB
> MemFree:        10317044 kB
> MemAvailable:   12229152 kB
> ===== 스토리지 확인 =====
> /dev/sda1        86G   47G   40G  55% /opt/bin/.nvidia
> ===== 현재 경로 확인 =====
> /content
> ===== 현재 경로의 내용 확인 =====
> sample_data
> ===== GPU 모델 확인 =====
> Tesla K80
> ===== GPU 상태 확인 =====
> Fri Feb 11 18:02:56 2022       
> +-----------------------------------------------------------------------------+
> | NVIDIA-SMI 460.32.03    Driver Version: 460.32.03    CUDA Version: 11.2     |
> |-------------------------------+----------------------+----------------------+
> | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
> | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
> |                               |                      |               MIG M. |
> |===============================+======================+======================|
> |   0  Tesla K80           Off  | 00000000:00:04.0 Off |                    0 |
> | N/A   55C    P8    31W / 149W |      0MiB / 11441MiB |      0%      Default |
> |                               |                      |                  N/A |
> +-------------------------------+----------------------+----------------------+
> +-----------------------------------------------------------------------------+
> | Processes:                                                                  |
> |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
> |        ID   ID                                                   Usage      |
> |=============================================================================|
> |  No running processes found                                                 |
> +-----------------------------------------------------------------------------+
> ```

설치된 소프트웨어도 확인해보자.

```
!echo "===== 파이썬 버전 확인 ====="
!python --version
!echo "===== 텐서플로 버전 확인 ====="
!python -c "import tensorflow as tf; print(tf.__version__)"
```

> ```
> ===== 파이썬 버전 확인 =====
> Python 3.7.12
> ===== 텐서플로 버전 확인 =====
> 2.7.0
> ```

모든 모델에서 사용하는 공통 패키지도 미리 import 해두자.

```python
import tensorflow as tf
import tensorflow_hub as hub
import matplotlib.pyplot as plt
import numpy as np
import cv2
```



## 1. Faster R-CNN





```
img_array_np = cv2.imread('/content/data/beatles01.jpg')
img_array = img_array_np[np.newaxis, ...]
print(img_array_np.shape, img_array.shape)

detector = hub.load("https://tfhub.dev/tensorflow/faster_rcnn/inception_resnet_v2_640x640/1")
detector_output = detector(image_tensor)
class_ids = detector_output["detection_classes"]
```
























---
layout: post
title:  "[HYU] Paper Reivew"
date:   2020-01-14 09:00:03
categories: HYU-Deep-Learning
---



# About This Lecture

이 강의는 "Deep Learning based Viusal Odometry and Depth Estimation" (DL-VODE, or simply VODE) 라는 주제로 강의를 합니다. 강의순서는 다음과 같습니다. 

1. **Prerequisites for VODE**: VODE를 이해하기 위한 사전지식을 공부합니다.
2. **Understanding DL-VODE**: VODE의 학습 원리를 이해하고 핵심 과정을 실습합니다.
3. **Paper Review**: SfmLearner 등 최신 VODE 관련 논문을 리뷰합니다.



# Paper Review

VODE 관련 주요 논문들을 소개합니다. 2017년부터 VODE의 초기 발전과정을 주로 다룹니다.



## 1. SfmLearner

<table>
<colgroup>
<col width="10%" />
<col width="90%" />
</colgroup>
<thead>
<tr class="header">
<th>제목</th>
<th>Unsupervised Learning of Depth and Ego-Motion from Video</th>
</tr>
</thead>
<tbody>
<tr>
<td markdown="span">저자</td>
<td markdown="span">Tinghui Zhou, Matthew Brown, Noah Snavely, David G. Lowe (google)</td>
</tr>
<tr>
<td markdown="span">출판</td>
<td markdown="span">CVPR, 2017</td>
</tr>
</tbody>
</table>


| Mono VO | Depth Prediction | Learning   | Absolute Scale | Open source |
| ---- | ---------------- | ---------- | -------------- | ------------- |
| O    | O               | Unsupervised | X           | O          |

**github**: <https://github.com/tinghuiz/SfMLearner> (tensorflow)



### 특징

아마도 **최초로 Unsupervised learning으로 Monocular Visual Odometry와 Depth Estimation을 동시에 학습**한 논문일 것이다. Monocular image sequence로만 학습하기 때문에 실제 스케일은 알지 못한다.  

이 모델은 두 개의 네트워크가 있는데 한 장의 이미지로부터 depth를 추정하는 Depth network와 두 장의 이미지로부터 둘 사이의 상대적인 pose를 추정하는 Pose network이 있다. 여기서도 소스 이미지(source image)를 변환(warping)하여 타겟 이미지(target image) 만들어내고 실제 타겟 이미지와의 차이(photometric error)를 손실(loss)로 사용한다. 

![sfmlearner1](../assets/2019-06-27-vode-survey/sfmlearner1.png)



특정 타겟 이미지 픽셀 ($$p_t$$)에 해당하는 소스 이미지 픽셀 ($$p_s$$)을 계산하는 식은 다음과 같다. 

![sfmlearner3](../assets/2019-06-27-vode-survey/sfmlearner3.png)

식을 해석하면 타겟 픽셀 ($$p_t$$)에 inverse projection과 depth를 곱하여 타겟 좌표계에서의 3차원 좌표를 계산하고 ($$\hat{D}_t(p_t)K^{-1}$$), 이를 소스 좌표계의 3차원 좌표로 변환하고 ($$\hat{T}_{t \to s}$$), 다시 이를 이미지 좌표로 projection ($$K$$) 한다는 뜻이다. 모든 타겟 픽셀에 대해 위 연산을 하면 소스 이미지 전체를 타겟 좌표계에서 찍은 것처럼 합성(synthesize)할 수 있다. 이론적으로 depth map과 pose가 잘 추정이 되었다면 합성된 이미지가 타겟 이미지와 동일해야 한다.   

구체적인 네트워크 구조는 아래와 같다. Depth network는 encoder-decoder 구조이다. Encoder에서는 공간적인 해상도(spatial resolution)을 줄여가면서 feature의 채널 수를 늘리고 다시 Decoder에서는 반대로 공간적인 해상도는 늘리면서 채널 수는 줄인다. Encoder와 Decoder 사이에는 skip connection을 두는데 decoder에서 convolution을 여러번 거친 high-level feature와 encoder에서 더 정확한 공간적인 정보를 담고 있는 low-level feature를 합쳐 더욱 풍성한 정보를 통해 Depth를 추론하도록 한다.  

Pose network은 Explanability network와 feature layer를 공유한다. 마지막 feature에서 몇 번의 convolution을 더 거친 후 **global average pooling으로 pose를 출력**하고, deconvolution을 붙여 이미지와 크기가 같은 explainability map을 만든다. Explainability map은 photometric error를 구할 때 weight 역할을 한다. occlusion이나 moving object 들로 인해 한 장의 이미지에서 depth를 추정하기 어려운 영역들에 낮은 weight를 주고자 만든 것이다.   

![sfmlearner2](../assets/2019-06-27-vode-survey/sfmlearner2.png)



### Loss

photometric loss는 다음과 같다. 특정 타겟 이미지($$I_t(p)$$)에서 나온 depth map으로 주변 여러 이미지($$I_1,...,I_N \to \hat{I}_s(p)$$)를 변환하여 모든 픽셀에 대해 오차를 더한다. 이때 explainability ($$\hat{E}_s$$)가 곱해진다.  

![sfmlearner4](../assets/2019-06-27-vode-survey/sfmlearner4.png)



단순 photometric loss는 이미지에서 변화가 별로 없는(low gradient, textureless) 영역에서 학습 효과가 없다. 그래서  global한 관점에서 depth를 추정할 수 있도록 depth network를 가운데가 좁은 encoder-decoder 구조로 설계하고 loss도 multi scale로 계산한다.

전체 loss 함수는 multi-scale에 대해서 photometric loss + smoothness  loss + explainability에 대한 regularization term으로 계산한다.

![sfmlearner5](../assets/2019-06-27-vode-survey/sfmlearner5.png)





## 2. UnDeepVO

<table>
<colgroup>
<col width="10%" />
<col width="90%" />
</colgroup>
<thead>
<tr class="header">
<th>제목</th>
<th>UnDeepVO: Monocular Visual Odometry through Unsupervised Deep Learning</th>
</tr>
</thead>
<tbody>
<tr>
<td markdown="span">저자</td>
<td markdown="span">Ruihao Li, Sen Wang, Zhiqiang Long and Dongbing Gu</td>
</tr>
<tr>
<td markdown="span">출판</td>
<td markdown="span">ICRA, 2017</td>
</tr>
</tbody>
</table>


### 특징

DeepVO의 후속작으로 MVO와 depth prediction을 Unsupervised learning으로 동시에 학습한 논문이다. SfmLearner와 비슷한데 **차이점은 Stereo 영상을 학습에 사용하여 실제 스케일까지 학습할 수 있다**는 것이다. 학습에는 두 개의 영상이 들어가서 두 영상의 depth map과 둘 사이의 pose (translation + rotation)가 나온다. 가장 중요한 학습 원리는 왼쪽 영상을 pose와 depth 정보를 이용해 오른쪽 카메라에서 찍은 것처럼 image reconstruction을 하고 원래 오른쪽 영상과 비교하여 차이(photometric error)가 줄어들도록 학습시키는 것이다. 이 학습을 temporal image sequence에만 적용하면 (시간 상 k와 k+1 이미지로 학습) 스케일을 알 수가 없다. 하지만 stereo image pair에도 적용하여 두 카메라 사이의 알려진 실제 거리를 통해 실제 스케일을 알도록 depth를 학습시킬 수 있다. temporal sequence 학습에서도 pose는 depth와 스케일이 맞아야 하니 pose도 실제 스케일에 맞춰지게 된다. 이런 오묘한 원리를 이용해 pose나 depth에 대한 ground truth가 없어도 unsupervised learning으로 스케일까지 정확한 VODE(Visual Odometry and Depth Estimation) 학습이 가능해진다.  

![UnDeepVO1](../assets/2019-06-27-vode-survey/UnDeepVO1.png)

네트워크 구조는 depth estimation을 위한 encoder-decoder 구조 하나, pose estimation을 위한 double-head 구조가 하나있다. Translation과 rotation을 하나의 출력으로 학습시키면 둘 사이의 weight 조절이 필요한데 이처럼 가지를 나눠서 학습시키면 그럴 필요가 없다.  

<img src="../assets/2019-06-27-vode-survey/UnDeepVO2.png" alt="undeepvo" width="300"/>



### Loss

#### Losses from stereo image pair

스테레오 이미지를 이용한 학습은 둘 사이의 pose는 알려져 있으므로 depth map을 disparity map으로 쉽게 변환할 수 있다. 이를 통해 왼쪽 이미지를 오른쪽 이미지로부터 복원 가능하고 그 반대도 가능하다.

1. Photometric consistency: 오른쪽 이미지를 카메라에서 찍은것 처럼 변환한 이미지($$I'_l$$) 와 원래 왼쪽 이미지($$I_l$$) 사이의 차이를 SSIM과 L1 distance의 조합으로 구성

   ![UnDeepVO3](../assets/2019-06-27-vode-survey/UnDeepVO3.png)

2. Disparity consistency: 왼쪽 depth map으로 만든 disparity map $$D^l_{dis}$$과 오른쪽 depth map으로 만든 disparity map $$D^r_{dis}$$는 서로 역방향 관계에 있으므로 $$D^r_{dis}$$를 반대방향으로 뒤집은 $$D^{l'}_{dis}$$는 $$D^l_{dis}$$과 같게 나와야한다. 이 둘 사이의 차이를 loss로 학습한다.

   ![UnDeepVO4](../assets/2019-06-27-vode-survey/UnDeepVO4.png)

3. Pose consistency: 같은 시각 k와 k+n 사이에 오른쪽과 왼쪽 image sequence에서 pose는 서로 같아야 한다. (논문에서는 그렇게 썼지만 사실 동일하지는 않다.) 그래서 양쪽 이미지 sequence에서 추정한 pose의 차이를 loss로 사용한다.

   ![UnDeepVO5](../assets/2019-06-27-vode-survey/UnDeepVO5.png)



#### Losses from temporal image sequences

Temporal image sequences에서는 pose estimator에서 추정한 pose와 depth map을 이용해 disparity map을 구한다. dispartiy map은 역시 반대쪽 이미지 복원에 사용하여 photometric error를 계산한다.

1. Photometric consistency: 시간 k+1의 이미지를 시간 k의 포즈에서 찍은 것처럼  변환한 이미지($$I'_l$$)와 원래 시간 k의 이미지와 ($$I_l$$) 사이의 차이를 SSIM과 L1 distance의 조합으로 구성

   ![UnDeepVO6](../assets/2019-06-27-vode-survey/UnDeepVO6.png)

2. 3D geometric registration: 시간 k+1의 point cloud ($$P_{k+1}$$)를 시간 k의 포즈로 변환한  point cloud ($$P'_k$$)와 원래 시간 k의 point cloud ($$P_k$$) 사이의 차이를 L1 distance로 계산

   ![UnDeepVO7](../assets/2019-06-27-vode-survey/UnDeepVO7.png)





## 3. Deep-VO-Feat

<table>
<colgroup>
<col width="10%" />
<col width="90%" />
</colgroup>
<thead>
<tr class="header">
<th>제목</th>
<th>Unsupervised Learning of Monocular Depth Estimation and Visual Odometry with Deep Feature Reconstruction</th>
</tr>
</thead>
<tbody>
<tr>
<td markdown="span">저자</td>
<td markdown="span">Huangying Zhan, Ravi Garg, Chamara Saroj Weerasekera, Kejie Li, Harsh Agarwal, Ian M. Reid</td>
</tr>
<tr>
<td markdown="span">출판</td>
<td markdown="span">CVPR, 2018</td>
</tr>
</tbody>
</table>


| Mono VO | Depth Prediction | Learning   | Absolute Scale | Open source |
| ---- | ---------------- | ---------- | -------------- | ------------- |
| O    | O               | Unsupervised | O          | O          |

**github**: <https://github.com/Huangying-Zhan/Depth-VO-Feat> (caffe)



### 특징

CVPR 2018부터 SfmLearner와 비슷한 목적을 가진 논문들이 여러 편씩 나오기 시작했다. UnDeepVO처럼 스테레오 영상을 이용해 실제 스케일의 depth와 pose를 학습하는 논문이다. 기존 연구는 이미지만 다른 좌표계로 합성해서 원래 영상과 합성된 영상 사이의 차이를 줄이는 방향으로 학습했지만, 여기서는 **CNN feature도 합성해서 원래 feature와 합성된 feature 사이의 차이도 loss**에 들어간다. 즉 이미지의 지역적 특성이 같다면 CNN feature도 같을 것이고 따라서 feature도 이미지처럼 다른 좌표계로 합성하면 원래 그곳에서 나온 feature와 동일해야 한다는 논리다. (feature reconstruction loss)   

같은 물체를 어느 방향에서 찍어도 같은 화소값이 나온다고 가정하는 것을 Lambertial assumption이라 하는데 항상 성립하는 것은 아니므로 시점에 따라 같은 위치의 RGB 값이 달라질 수 있고 이는 photometric consistency loss를 오염시킨다. 그래서 photometric consistency에만 의존하지 않고 이보다 더 밝기에 강인하다고 생각되는 CNN feature를 복원하는 loss를 만든 것이다.  

네트워크 구성은 UndeepVO와 유사하다. Depth CNN과 Odometry CNN으로 구성되어 있고 이를 이용해 어떤 이미지를 다른 pose에서 찍은 것처럼 합성한 이미지를 만들어 loss를 계산한다. 학습에 스테레오 이미지를 활용하여 실제 스케일을 학습할 수 있다. 차이점은 Depth CNN이 depth를 바로 추정하지 않고 inverse depth를 추정한다는 것이다. Odometry CNN은 FC layer를 통해 pose를 6차원 벡터($$se(3)$$)로 출력한다.

![deepvofeat1](../assets/2019-06-27-vode-survey/deepvofeat1.png)





### Loss

![deepvofeat2](../assets/2019-06-27-vode-survey/deepvofeat2.png)



논문에 학습과정을 위와 같이 그렸는데 해석하면 다음과 같다.

1. 기준 이미지 ($$I_{L,t2}$$)를 Depth CNN에 넣어 depth map($$D_{L,t2}$$)을 만든다.
2. 기준 이미지 ($$I_{L,t2}$$)와 이전 시간 이미지 ($$I_{L,t1}$$)을 Odometry CNN에 입력하여 좌표계 이동($$T_{t2 \to t1}$$)을 추정한다. 
3. Depth map($$D_{L,t2}$$)과 좌표계 이동($$T_{t2 \to t1}$$)을 이용하여 이전 이미지($$I_{L,t1}$$)를 현재 좌표계로 합성한 $$I'_{L,t1}$$을 만든다.
4. Depth map($$D_{L,t2}$$)과 스테레오 변환($$T_{L \to R}$$)을 이용하여 오른쪽 이미지($$I_{R,t2}$$)를 왼쪽 좌표계로 합성한 $$I'_{R,t2}$$을 만든다.
5. Depth map($$D_{L,t2}$$)과 좌표계 이동($$T_{t2 \to t1}$$)을 이용하여 이전 이미지에서 나온 CNN feature($$F_{L,t1}$$)를 현재 좌표계로 합성한 $$F'_{L,t1}$$을 만든다.
6. Depth map($$D_{L,t2}$$)과 스테레오 변환($$T_{L \to R}$$)을 이용하여 오른쪽 CNN feature($$F_{R,t2}$$)를 왼쪽 좌표계로 합성한 $$F'_{R,t2}$$을 만든다.
7. 합성한 이미지와 기준 이미지 ($$I_{L,t2}$$), 합성한 feature와 기준 feature ($$F_{L,t2}$$)의 차이로 loss를 계산한다.

전체 loss는 세 가지 요소로 구성되어 있다.

- 이미지 합성 함수와 image reconstruction loss

  ![deepvofeat3](../assets/2019-06-27-vode-survey/deepvofeat3.png)

  ![deepvofeat4](../assets/2019-06-27-vode-survey/deepvofeat4.png)

- Feature 합성 함수와 feature reconstruction loss

  ![deepvofeat5](../assets/2019-06-27-vode-survey/deepvofeat5.png)

  ![deepvofeat5](../assets/2019-06-27-vode-survey/deepvofeat6.png)

- Edge-aware smootheness loss: 이미지 변화($$\partial_x I_{m,n}$$)가 낮은 곳에서 depth 변화($$\partial_x D_{m,n}$$)가 높으면 loss가 커짐

  ![deepvofeat7](../assets/2019-06-27-vode-survey/deepvofeat7.png)





## 4. GeoNet

<table>
<colgroup>
<col width="10%" />
<col width="90%" />
</colgroup>
<thead>
<tr class="header">
<th>제목</th>
<th>GeoNet: Unsupervised Learning of Dense Depth, Optical Flow and Camera Pose</th>
</tr>
</thead>
<tbody>
<tr>
<td markdown="span">저자</td>
<td markdown="span">Zhichao Yin and Jianping Shi</td>
</tr>
<tr>
<td markdown="span">출판</td>
<td markdown="span">CVPR, 2018</td>
</tr>
</tbody>
</table>


| Mono VO | Depth Prediction | Learning   | Absolute Scale | Open source |
| ---- | ---------------- | ---------- | -------------- | ------------- |
| O    | O               | Unsupervised | O          | O          |

**github**: <https://github.com/yzcjtr/GeoNet> (tensorflow 1.1)



### 특징

GeoNet은 Visual Odometry, Depth Prediction 외에도 Optical Flow까지 추정하는 모델이다. 세 가지 일을 하므로 거기에 해당하는 세 가지 서브 네트워크가 있다.

- DepthNet: 한 장의 이미지로부터 depth map 출력한다. "globally high-level and locally detailed" 정보를 동시에 전달하기 위해 encoder-decoder 구조에 skip connection을 추가하였고 multi-scale 결과를 출력한다.
- PoseNet: image sequence를 한번에 받아 타겟 이미지를 기준으로 다른 이미지들의 상대 pose 출력한다. 마지막에 global average pooling으로 translational vector와 euler angle을 포함한 6차원 벡터를 출력한다.
- ResFlowNet: depth map과 pose로부터 계산된 rigid flow에서 움직이는 물체로부터 발생하는 추가적인 flow 출력한다. 입력으로는 두 이미지 ($$I_s, I_t$$), rigid flow ($$f^{rig}_{t \to s}$$), rigid flow로 합성된 이미지 ($$\tilde{I}^{rig}_s$$), 합성된 이미지와 원본 이미지와의 오차가 채널로 합쳐서 들어간다. DepthNet과 동일하게 encoder-decoder 구조에 skip connection을 추가하였고 multi-scale 결과를 출력한다.



GeoNet은 아래 그림처럼 두 단계로 구성된다. 첫 번째 단계에서는 DepthNet과 PoseNet에서 출력한 depth map과 pose를 이용하여 rigid flow를 아래 식으로 계산한다. 움직이는 물체를 고려하지 않고 정적 환경의 rigid transformation에 의한 optical flow만 계산한 것이다.  그림을 보면 forward, backward 두 방향의 flow를 모두 계산한다.

![geonet2](../assets/2019-06-27-vode-survey/geonet2.png)

두 번째 단계는 ResFlowNet에서 움직이는 물체에 의해 발생하는 residual flow를 출력한다. Residual flow를 보면 움직이는 물체를 찾아낼 수 있다. 최종적인 flow는 두 가지 flow를 합쳐서 구한다: $$ f^{full}_{t \to s}=f^{rig}_{t \to s}+f^{res}_{t \to s} $$  이때도 역시 forward, backward 두 방향 모두 계산한다.



![geonet](../assets/2019-06-27-vode-survey/geonet1.png)



### Training and Loss

네트워크 구성처럼 학습도 두 단계로 진행된다. 먼저 DepthNet과 PoseNet을 학습시킨 다음, 두 네트워크를 고정시킨채 ResFlowNet을 학습시킨다. 

전체 loss는 다음과 같이 다섯개 항목으로 구성되는데 한번에 여러 pose를 출력하므로 <t, s>가 여러가지가 되고 multi-scale이기 때문에 스케일 별로 loss를 계산하여 더한다.

![geonet3](../assets/2019-06-27-vode-survey/geonet3.png)

1. Rigid warping loss $$\mathcal{L}_{rw}$$ : 합성한 이미지와 타겟 이미지의 차이를 L1 norm과 SSIM의 조합으로 계산한다.

   ![geonet4](../assets/2019-06-27-vode-survey/geonet4.png)

2. Edge-aware depth smoothness loss $$\mathcal{L}_{ds}$$ : 이미지 변화($$\nabla I(p_t)$$)가 낮은 곳에서 depth 변화($$\nabla D(p_t)$$)가 높으면 loss가 커짐

   ![geonet5](../assets/2019-06-27-vode-survey/geonet5.png)

3. Full flow warping loss $$\mathcal{L}_{fw}$$ : full flow ($$ f^{full}_{t \to s}$$)로 합성한 이미지와 타겟 이미지의 차이를 Rigid warping loss처럼 계산

4. Flow smoothness loss $$\mathcal{L}_{fs}$$ : depth smoothness loss처럼 flow flow에 대한 smoothness 계산

5. Geometric consistency loss $$\mathcal{L}_{gc}$$ : forward flow와 backward flow 사이에 역관계성을 측정한다. 역관계는 occluded region(한 쪽 영상에서만 보이는 부분)에서는 성립할 수 없으므로 그 부분을 제외하기 위해 $$[\delta(p_t)]$$를 곱한다. $$[\delta(p_t)]$$는 forward-backwad 오차가 작은 곳에서는 1이고 큰 곳에서는 0이 되는 함수다.

   ![geonet6](../assets/2019-06-27-vode-survey/geonet6.png)

   ![geonet7](../assets/2019-06-27-vode-survey/geonet7.png)



---

DL-VODE는 이후에도 다양한 논문들이 나오고 있다. 최근에는 depth, pose 뿐만 아니라 optical flow나 카메라의 intrinsic parameter도 동시에 학습하는 논문들도 나오고 기능이 더욱 다양화되는 추세이다.


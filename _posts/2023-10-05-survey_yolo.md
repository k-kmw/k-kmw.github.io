---
layout: post
title: YOLO(You Only Look Once) 기술의 원리와 버전 별 비교
author: MinWook, Kim
tags: engineering wrting, yolo, object detection
---

> 이 글에서는 YOLO(You Only Look Once)기술을 소개하고, 초기 YOLO에서 YOLOv6까지의 주요 변경 사항을 알아본다.

## 서론
실시간 객체 탐지 모델로 YOLO(You Only Look Once)가 주목을 받고 있다. YOLO는 2016년 6월 27일, 미국 라스베이거스에서 열린 컴퓨터 비전 학회인 CVPR 2016에서 처음 공개되었고 객체 감지를 실시간으로 가능하다는 점에서 주목을 받았다. 이것이 가능한 이유로 사물이 어디 있는지 위치 정보를 나타내는 localization과 사물이 어떤 것인지 분류하는 classification을 한 번에 처리하는 1-stage-detection방법을 사용하기 때문이다. 

## 본론
### Original YOLO 추론 과정
#### Bounding Box Regression 
각 그리드 셀마다 해당 셀을 포함하는 2개의 바운딩 박스를 생성한다. 앞 두개의 5열은 각 바운딩 박스 중심의 x좌표, y좌표, 너비, 높이와 바운딩 박스내에 객체가 존재할 확률Pc인 신뢰도를 나타낸다. 여기서 𝑃𝑐=𝑃𝑟(𝑜𝑏𝑗𝑒𝑐𝑡)∗𝐼𝑜𝑈(𝑡𝑟𝑢𝑡ℎ, 𝑝𝑟𝑒𝑑), 𝑃𝑟(𝑜𝑏𝑗𝑒𝑐𝑡)은 박스에 물체가 존재하면 1, 존재하지 않으면 0을 나타내고 𝐼𝑜𝑈(𝑡𝑟𝑢𝑡ℎ, 𝑝𝑟𝑒𝑑)은 예측한 박스가 실제 정답과 얼마나 겹치는지를 나타낸다.

![]({{site.baseurl}}/images/20231005/img1.png)

그림1 7x7 grid cell 

![]({{site.baseurl}}/images/20231005/img2.png)

그림2 class score

이후 바운딩 박스의 class scores를 구한다. P_r (class_i│object)*P_r (object)*IoU(truth, pred)=P_r (class_i )*IoU(true, pred). 즉, 그리드 셀의 클래스 레이블 score * 바운딩 박스 내에 객체가 존재할 확률이다. Class scores는 클래스가 바운딩 박스에 등장할 확률과 객체가 예측된 박스에 얼마나 잘 맞는지를 나타낸다.

![]({{site.baseurl}}/images/20231005/img3.png)

그림3 class scores 구하는 과정

그림에서 7x7개의 셀이 존재하여 각 그리드 셀마다 2개의 바운딩 박스를 그리므로, 총 98개의 바운딩 박스에 대한 클래스 scores를 생성한다.

![]({{site.baseurl}}/images/20231005/img4.png)
![]({{site.baseurl}}/images/20231005/img5.png) ![]({{site.baseurl}}/images/20231005/img6.png)

그림4 class scores 생성

#### Classification
1. 앞서 구한 class score가 confidence threshold보다 낮을 경우 0으로 만든다. 
2. Class score에 대해 내림차순으로 정렬한다.
3. NMS알고리즘을 적용하여 중복되는 바운딩 박스에 대한 해당 클래스 score를 0으로 만든다.

![]({{site.baseurl}}/images/20231005/img7.png)

그림5 classification 과정

<br>
**NMS**

하나의 그리드 셀에 대하여 두 개의 바운딩 박스를 예측하므로 같은 객체를 나타내는 여러 중복되는 바운딩 박스가 나타날 수 있다. NMS는 이러한 같은 객체를 나타내는 중복되는 바운딩 박스를 제거하는 과정이다.

![]({{site.baseurl}}/images/20231005/img8.png)

그림6 NMS

NMS에 IoU개념이 사용된다. IoU는 바운딩 박스의 교집합/합집합 비율로 두 바운딩 박스가 얼마나 겹치는지에 대한 값을 나타낸다.

![]({{site.baseurl}}/images/20231005/img9.png)

그림7 IoU

앞서 Classification과정에서 Class score에 대하여 내림차순 정렬하였으므로 첫 번째 바운딩 박스 score가 최대(max)가 된다. 
1.	Max와 다른 모든 바운딩 박스의 해당 클래스 socre를 비교한다.
2.	만약 두 바운딩 박스의 IoU를 비교하여 IoU threshold보다 클 경우 해당 score를 0으로 설정한다. 이 과정을 거치면서 같은 객체를 가리키는 바운딩 박스가 제거된다.
3.	다음으로 큰 바운딩 박스 score를 max로 설정한 후 위 과정을 반복한다.
4.	이 과정을 모든 클래스에 대해 반복한다.
5.	각 바운딩 박스의 class scores에서 최대값을 가지는 class로 분류하고 바운딩 박스를 그린다. 

![]({{site.baseurl}}/images/20231005/img10.png)

그림8

![]({{site.baseurl}}/images/20231005/img11.png)

그림9

### YOLO 버전별 비교
**YOLOv2**
1. 고해상도 분류기: YOLOv1과 마찬가지로 224 x 224 크기의 ImageNet으로 모델을 사전 훈련 시키고, 448 x 448 해상도로 ImageNet에서 10호의 epoch동안 파인튜닝을 수행하여 고해상도 입력에 대한 성능을 향상시켰다.
2. Fully Convoluitonal: dense layer를 제거하고 Fully Convolutional 구조를 사용
3. Anchor Box 사용: 미리 정의된 모양을 가진 앵커 박스를 사용하여 바운딩 박스를 예측하는 방식을 사용하였다. 각 그리드 셀마다 여러 앵커 박스가 정의되며 시스템은 각 앵커 박스마다 좌표를 예측한다. 


**YOLOv3**
1. 바운딩 박스 예측 방식: 로지스틱 회귀를 사용하여 각 바운딩 박스에 대한 객체 존재 여부 점수를 예측한다. 점수는 ground truth와 가장 높은 겹침을 가진 Anchor Box에 대해 1이고 나머지는 0이다. 
2. 클래스 예측: 분류에 대해 소프트맥스를 사용하는 대신, 로지스틱 분류기를 훈련하기 위해 이진 교차 엔트로피를 사용하였다. 이 변경으로 동일한 박스에 여러 라벨을 할당할 수 있다.
3. 새로운 백본: v3에서는 53개의 합성곱 레이어로 구성된 Darknet-53을 사용한다.

**YOLOv4**
1. 개선된 아키텍처 (CSPDarknet53): v3에서 사용된 Darknet-53에 cross-stage partial connections(CSPNet)과 Mish 활성함수를 사용하도록 수정하였다. 
2. 개선된 학습: 기존의 무작위 밝기, 대비, 크기 조정, 자르기 및 회전을 이용한 증강 외, 4개의 이미지를 하나의 이미지로 결합하여 일반적이지 않은 맥락에서 학습하는 Mosaic기법을 도입하였다. 이로 인해 학습시 배치 크기를 줄일 수 있다.
3. 유전 알고리즘을 사용하여 하이퍼마리미터 최적화: 훈련에 사용된 최적의 하이퍼파라미터를 찾기 위해 유전 알고리즘을 사용하였다.

**YOLOv5**
1. 아키텍처 수정: v4에서 사용된 CSPDarknet-53에서 메모리 및 계산 비용을 줄이기 위해 큰 window size를 가지는 stem으로 시작하도록 수정하였다.
2. 다섯가지의 규모의 버전: YOLOv5n (nano), YOLOv5s (small), YOLOv5m (medium), YOLOv5l (large), and YOLOv5x (extra-large)로 여러 하드웨어 및 소프트웨어 요구사항에 맞게 convolution modules의 깊이와 너비를 다양하게 하였다.

**YOLOv6**
1. 새로운 백본 도입: RepVGG를 기반으로 한 새로운 백본인 EfficientRep을 도입하여 이전 버전의 백본보다 더 높은 병렬성을 활용한다. 
2. 새로운 분류 및 회귀 손실 함수를 사용: 분류에는 VariFocal손실을 사용하고 회귀에는 SloU/GloU회귀 손실을 사용한다.


![]({{site.baseurl}}/images/20231005/table1.png)

표 1 YOLO 버전 별 주요 변경 점

결론
YOLO는 사물을 분류하는 classification과 위치를 나타내는 localization을 한 번에 하는 1-stage-detection방식을 사용하여 실시간 객체 탐지한다.
original YOLO에서 발생하는 문제를 개선하여 여러 버전이 출시되었다. 각 버전 별 YOLO 구조와 성능을 요약하면 표 2와 같다.

![]({{site.baseurl}}/images/20231005/table2.png)

표 2 YOLO 버전 별 특징

표 2의 성능 지표(AP)에서 YOLO와 YOLOv2의 경우 VOC2007 데이터셋을 사용하여 평가되었고, 나머지는 COCO2017 데이터셋을 사용하여 평가하였다.

## 참고문헌
Juan Terven, Diana Cordova-Esparza, “A Comprehensive Review of YOLO: From YOLOv1 and Beyond”, 2023
https://docs.google.com/presentation/d/1aeRvtKG21KHdD5lg6Hgyhx5rPq_ZOsGjG5rJ1HP7BbA/pub

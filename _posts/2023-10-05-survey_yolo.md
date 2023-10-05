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
![7x7 grid cell](https://k-kmw.github.io/images/20231005/img1.png)

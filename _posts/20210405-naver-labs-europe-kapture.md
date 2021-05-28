---
title: Kapture + CVPR2020 Visual localization challenge (NAVER LABS Europe)
date: 2021-04-03 16:09:03
mathjax: true
tags: 
- NAVER LABS
- Visual localization
- Kapture
categories: 
- [SLAM, 공부 자료]
excerpt: 2020년 NAVER LABS Europe Kapture 블로그 아티클 + VisLocOdomMapCVPR2020 블로그 아티클 정리
---


- [kapture – A unified data format to facilitate visual localization and structure from motion.아티클 원본](https://europe.naverlabs.com/blog/kapture/)
- [One method, one pipeline: NAVER LABS Europe ranks high across three visual localization challenges at CVPR 2020 아티클 원본](https://europe.naverlabs.com/blog/one-method-one-pipeline-naver-labs-europe-ranks-high-across-three-visual-localization-challenges-at-cvpr-2020/)
---

## Kapture

- 많은 Visual localization 데이터셋들이 다른 데이터 format을 가지고 있다.
- **Kapture**는
  1. 이런 데이터를 호환되게 convert 시켜줄 수 있다.
    - Sensor parameters (i.e. intrinsic/extrinsics)
    - Raw sensor data (i.e. camera images / LiDAR data)
    - Other sensor data (GPS/WiFi)
    - Intermediate data
      - 2d local features
      - 2D-2D match
      - global feature
      - 3D map
  2. 다양한 pipeline을 제공한다.
    - COLMAP + SIFT
    - R2D2 + AP-GeM
  3. Visual localization 실험을 위한 데이터셋을 준비해놨다.
    - ECCV 2020 Visual localization challenge에 쓰인 데이터셋 모두 포함
      - Aachen Day-Night
      - Inloc
      - RobotCar Seasons
      - Extended CMU-Seasons
      - SILDa
      - Time of Day

Visual localization 입문을 쉽게 만들어주는 소프트웨어 정도로 생각하면 좋을 것 같다.

&nbsp;

---

## CVPR 2020 workshop

- CVPR 2020의 Joint Workshop on Long-Term Visual Localization, Visual Odometry and Geometric and Learning-based SLAM에서는 3가지 visual localization 챌린지를 열었다.
  1. 자율주행 자동차에서 visual localization하기
  2. Handheld device에서 visual localization하기
  3. Long-term localization을 위한 local feature 찾기.
- NAVER LABS Europe팀은 1번 챌린지에서 1등, 2번 챌린지에서 4등, 3번 챌린지에서 2등을 했다고 한다.


- 사용한 기술은 APGeM이라는 image retrieval 기술과 R2D2라는 local feature이다.
- 이 둘을 Kapture에 담은 visual localization pipeline을 이용했다.


&nbsp;

### 사용한 기술 - RD2D + APGeM

- 글의 초반은 [이전 글](changh95.github.io/20210405-naver-labs-europe-kapture)과 겹치는 내용이 많아 생략한다.


- R2D2
  - Sparse keypoint detector & descriptor
  - Synthetic image 기반으로 학습됨.
  - Detection과 Description을 동시에 추론 가능.
    - Keypoint reliability와 repeatability를 따로 계산함.
- 너무 큰 Large-scale인 경우에는 3D reconstruction을 할 수 없음. 그러므로 image retrieval을 이용해서 place recognition 방식을 사용함.
- APGeM
  - Generalized meanpooling (GeM) 레이어를 이용해서 feature map을 정해진 길이의 컴팩트한 형태로 바꿈.
  - Mean average precision (mAP) 값을 이용해서 모델을 학습함. (Metric learning)
    - Google Landmakrs dataset으로 학습.
- COLMAP을 이용해서 structure-from-motion (SfM)을 수행하고, geometric verification을 수행.


- 파이프라인을 보면...
  - 처음에 Training images + poses를 기반으로 APGeM을 적용한다.
  - 이렇게 뽑은 similar 이미지들에 R2D2를 적용하여 keypoint를 뽑고 매칭을 한다.
  - COLMAP으로 SfM을 수행해서 3D point cloud를 만든다.
  - Query image에 APGeM을 적용한다.
  - 이렇게 뽑은 similar image에 존재하는 2D-3D correspondence를 만들어서, query-reference-3Dmap correspondence를 만들고, geometric verification (i.e. PnP+RANSAC)으로 pose 를 추정한다.


- 조금 이해가 안가는 부분은, training images는 어떻게 결정하는지이다. APGeM으로 Top k개의 similar image들을 뽑은거를 training image라고 한걸까?


&nbsp;

### 결과

- 이미지에 다양한 요소가 포함되어있음에도 굉장히 잘 generalize했다.
  - time of day
  - season of the year
  - Outdate reference representation
  - Occlusion
  - Motion blur
  - Extreme viewpoint change
  - Low texture area
- 1등을 한 챌린지인 '자율주행에서의 visual localization'에서는 motion sequence가 주어지면서, 이로부터 정확한 위치를 찾을 수 있었다.
- 4등을 한 챌린지인 'Handheld 기기에서의 visual localization'에서는 1장의 query image로부터 정확한 위치를 찾는 것이였다.
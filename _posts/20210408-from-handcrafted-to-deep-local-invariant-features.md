---
title: Csurka 2018 - From Handcrafted to Deep Local Invariant Features
date: 2021-04-08 17:17:35
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- Local features
- Deep Learning
- Visual localization
- NAVER LABS
categories: 
- [SLAM, 논문 리뷰]
excerpt: Gabriela Csurka와 Martin Humenberger의 2018년도 Local feature 서베이 논문 정리.
---

경험상 나는 ASIFT, ORB, AKAZE가 잘 된다는 걸 알고있다. (OpenCV에도 다 구현되어있어서 그냥 가져다가 써봤는데 잘 됬다).
Deep local feature는 아직 잘 써보진 못했지만, 기존의 방식들이 범접하지 못할정도로 잘 된다고 한다.
물론 traditional과 deep 방식의 장단점이 있겠다만, 아무래도 미래는 deep local feature가 훨씬 밝지 않을까 생각이 들기 때문에 공부할 필요가 있다고 생각한다.

이번 서베이를 통해서 어떤 방법론들이 있고 어떤 목적으로 만들어졌는지 간단하게 파악한 후 사용해보려고 한다.

&nbsp;

---

## Introduction

- Local feature = **Keypoint + local descriptor**
  - **Keypoint**: 이미지 속 두드러지는 부분
    - Detection 과정을 거쳐서 keypoint를 정확하게 검출해야함.
    - 다른 시점에서 보았을 때도 동일한 3D 위치에서 검출할 수 있어야하며 (**geometric invariance**), 조명이 변해도 동일한 위치에서 검출할 수 있어야함 (**photometric invariance**)
    - Interest point, anchor point라고 불리기도 함.
  - **Local descriptor**: 이미지 속 특정 부분의 특징을 표현하는 방법
    - Keypoint detection을 한 후, 해당 위치에서 local descriptor를 뽑는 경우가 많음.
    - 픽셀 값들을 저장하거나, 히스토그램을 사용해서 저장함.
- Local feature들을 사용해서 다음과 같은 작업을 수행할 수 있음.
  - Face/object detection & recognition, image retrieval, motion detection & tracking, depth-map generation, image stitching, camera calibration, 3D reconstruction, structure-from-motion (SfM), visual odometry, visual-SLAM.


- Descriptor는 3가지 방법으로 사용할 수 있음.
  - 해당 local feature가 어떤 의미를 가지는지 표현 (i.e. semantics)
    - 인공위성 이미지에서 line을 추출했다면, 혹시 그 line이 도로를 의미한다던지
    - 불량품 검사 이미지에서 blob이 추출됬을 때, 혹시 그 blob이 불량한 부분이라던지
  - **다른 시간, 다른 위치에서 찍은 이미지에서도 동일하게 keypoint들을 검출해서 이미지간의 motion 또는 geometric relationship을 계산하기 위해**
    - 최대한 동일한 물체를 표현하는 local feature끼리 매칭이 잘 되야함.
  - 여러개의 local feature 정보들을 군집화하여 하나의 큰 global descriptor를 표현.
    - Bag of visual words, Fisher vectors, VLAD를 참고하면 좋음.


- 좋은 local feature는...
  - **시점 변화, 조명 변화에도 강인함**
    - 이미지 노이즈, quantization으로 인한 precision loss, 이미지 압축 등으로부터 나타나는 노이즈에도 강인함.
  - **Distinctive한 descripto**r들을 가지고 있어야함.
    - 여러개의 local feature들을 가지고 있을 때, 동일한 위치를 표현하는 keypoint들의 descriptor를 매칭할 수 있어야함.
      - 이를 위해서 descriptor는 **충분한 정보량**을 가지고 있어야함.
      - 하지만 **메모리 효율성과 속도**를 고려했을 때, 적당히 compact하기도 해야함.

{% asset_img "pipeline.png" "Pipeline" %}

&nbsp;

---

## Handcrafted local features

---

### Keypoint detectors

#### 초기 detector 방법 - corners / image gradient 이용

- Corner 방식은 curve를 찾아서 maxima를 찾거나, edge에 대한 tangent를 찾아서 급격한 방향 변화를 탐지하는 방법
- Image gradient의 변화를 이용하는 방법
  - **Hessian detector** 계열
    - [Zuniaga and Haralick 1983]으로 시작
    - **SURF** 알고리즘에서는 Hessian matrix를 box filter를 이용해서 빠르게 계산 [Bay 2006]
  - **Harris detector** 계열
    - [[Harris and Stephens 1988]](http://changh95.github.io/20201215-Harris-corner/)으로 시작
  - **SIFT** 계열
    - [Lowe 2004]로 시작
    - Difference of Gaussian (DoG)를 이용해서 Laplacian of a Gaussian에 근사함.

#### Segmentation 기법

- Homogeneous region을 가르는 boundary를 찾는 방법 [Liu and Tsai 1990]
- Homogeneous region을 찾는 방법 [Corso and Hager 2005]
  - **MSER** 알고리즘의 경우 wide-baseline stereo에서 disparity를 계산하기 위해 만들어짐 [Matas 2002]

#### Mathematical morphology

- SUSAN detector는 이미지 속 모든 픽셀에 대해, 해당 픽셀과 주변 픽셀 값을 비교해서 밝은지/어두운지를 비교해서 keypoint 검출을 한다. [Smith and Brady 1997]
- **FAST**는 SUSAN을 개량한 버전으로써, 해당 픽셀에 원을 그려서 해당 원 위에 위치한 픽셀들과 값을 비교, 그 후 밝거나 어두운 픽셀들이 몇개나 연속해있는지를 판단하여 keypoint 검출을 한다. [Roesten and Drummond 2006]
  - 엄청 빠르다
  - 비슷한 위치에서 keypoint가 다수 발견될 경우, 하나로 묶어주는 Non-maximum suprression 알고리즘도 포함되어있다.
- [Lepetit and Fua 2006] 논문에서는 FAST에 Laplacian function을 이용해서 scale selection 기능을 추가하였다.

#### Saliency

- 'Keypoint는 주변 픽셀과 비교하였을 때 unpredictable local attribute를 가지고 있다는 점'을 이용해서 keypoint를 검출하는 방식이다.
- [Kadir 2004] 논문에서는 주변 픽셀들과 비교하였을 때 gray-value histogram에서 entropy 값의 변화를 이용하였다.
- Multi-resolution에서 안정적으로 keypoint를 추정하기 위해 wavelet transformation을 사용하는 방법도 제안되었다 [Sebe 2003]

#### Edge information

- 노이즈에 취약한 이미지 밝기 값을 사용하기보다는, 좀 더 안정적인 edge 정보를 사용하는 방법도 있다.
- 두개의 edge가 서로 perpendicular할 때, 그 두개의 edge와의 거리가 동일해지는 (i.e. equidistant) edge focus 위치를 keypoint로 사용하는 EdgeFoci 방법도 제안되었다 [Zitnick and Ramnath 2011]

&nbsp;

---

### Real descriptors

> 여기서 real은 real number를 의미한다. 0/1로 이루어진 binary descriptor와는 다르다.

#### SIFT 계열

- **SIFT**는 SIFT detector로 keypoint를 뽑은 후 해당 위치로부터 1,2,3,4분면에 4x4 grid에 local gradient histogram을 계산해서 descriptor로 사용함. [Lowe 2004]
  - 이 밑으로 전부 SIFT에서 발전한 방법들임.
- GLOH는 **grid를 log-polar 형태로 나누고** gradient들의 평균 값으로 변환해서 사용함. (**4x4 grid는 네모난 패치인데, 네모난 형태는 좌우대각선 거리가 일정하지 않아서 모든 방향으로 거리가 일정한 원 형태의 log-polar grid로 바꾼 것**) [Mikolajczyk and Schmid 2003]
- CSIFT는 colour invariant characteristics를 이용함 [Abdel-Hakim and Farag 2006]
- DSP-SIFT는 여러 scale로부터 SIFT descriptor pooling을 수행함 [Dong and Soatto 2015]
  - 기존의 SIFT에서 사용된 $\sigma$로 분리되었던 scale-space를 이용해서 local maxima를 찾지만, DSP-SIFT는 image pyramid를 만들며 생긴 size-space에서 local maxima를 찾음. (Detector 알고리즘)
  - 기존의 SIFT는 8x8 grid에서 image gradient를 계산해서 histogram으로 만들지만, DSP-SIFT는 location과 scale에서 모두 뽑고, concatenate시킴.
    - 즉, 차원 수가 훨씬 많다는 것.
- Scale-less SIFT는 다중 scale에 적용되는 subspace SIFT representation을 이용함 [Hassner 2017]
- **DSP-SIFT**, **Scale-less SIFT**는 최신 논문이니 직접 읽어보는 것도 좋을 듯.

#### SIFT보다 빠른 계열

- SURF는 integral image 기법으로 계산한 Haar filter response를 descriptor로 사용함 [Bay 2006]
- KAZE는 nonlinear scale space를 만드는 detector / descriptor임 [Alcantarialla 2012]
- **AKAZE**는 KAZE의 pyramidal framework에 fast explicit diffusion을 사용해서 계산을 가속시킨 방법임. [Alcantarilla 2013]
  - 꽤 정확한 편. 보통 ORB~AKAZE~SIFT로 정확도/속도를 비교함.

&nbsp;

---

### Binary descriptors

> 실시간 프로세싱 + 적은 메모리를 요구하는 경우에는 (e.g. Visual SLAM) binary feature descriptor를 사용하는게 훨씬 효율적이다. Binary feature descriptor들간의 거리를 구할 때는 **hamming distance**를 사용한다.

{% asset_img "binary_descriptors.png" "binary descriptors" %}
(Courtesy: Yukyung Choi)

- LBP와 census transform 방식은 keypoint가 해당하는 픽셀과 주변 픽셀들의 밝기를 비교함. [Ojala 1996, Pietikainen 2011] [Zabih 1994]
  - Keypoint 픽셀이 밝으면 1, 어두우면 0을 저장함.
- **BRIEF**는 keypoint 픽셀 주변의 픽셀들이 pair로 지정되어서 랜덤한 패턴을 가지고 있음. 이 pair 픽셀끼리 밝기를 비교해서 descriptor로 사용함. [Calonder 2010]
  - 이 때 저자들이 만들어둔 (잘되는) 패턴이 몇개 있음.
- **Oriented-FAST Rotated-BRIEF (ORB)**는 FAST detection을 하면서 image moment를 구해얻은 방향 정보를 BRIEF에 추가함으로써 회전에 대한 invariance를 얻음 [Rublee 2011]
- **BRISK**는 keypoint 픽셀 주변으로 여러개의 원형 region에서 각각 픽셀 샘플링을 하고 밝기 값을 비교함. [Leutenegger 2011]
  - 방향을 미리 정해두기 때문에 Rotation invariance가 있음.
- **FREAK**은 multi-resolution 환경에서 pair를 뽑아 밝기 값을 비교함. [Alahi 2012]
  - 사람 눈의 방식을 따라했다고 함
- BIO는 LIOP descriptor의 영향을 받아서 패치 안의 밝기 값들에 대해 order와 shape 정보를 저장해둠. [Choi 2014] [Wang 2011, 2016]

&nbsp;

---

## Machine learning 기반의 local feature





&nbsp;

---

## 후기

내가 읽기 어려워하는 바텀업 형태의 논문 ㅜㅜ...
엄청 많은 정보를 쫙 다 늘어놓고나서 '우리는 이 중에서 2번만 볼것이다' 같은건 너무 힘듭니다...
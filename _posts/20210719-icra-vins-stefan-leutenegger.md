---
title: ICRA 2021 - Visual-Inertial SLAM and Spatial AI for Mobile Robotics (Prof. Stefan Leutenegger) 
date: 2021-07-19 21:14:31
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- ICRA
- Imperial College London
- SLAMCore
- Deep Learning
- TUM
- VIO
- Robotics
- Stefan Leutenegger
categories: 
- [SLAM, 학회 발표 리뷰]
excerpt: ICRA 2021 Workhop on Visual-Inertial Navigation Systems 에서 Stefan Leutenegger 교수님의 토크.
---

@2:23:50
https://youtu.be/xlDbuw6skag

&nbsp;

----

## Stefan Leutenegger 교수님은 누구?

- 취리히 공대 학석박, Roland Siegwart 교수님 랩실에서 박사학위
- 2014년부터 영국 ICL에서 lecturer
- Andrew Davison 교수님과 함께 SLAMCore의 Cofounder
- 주요 논문:
  - BRISK
  - OKVIS
  - ElasticFusion
  - SemanticFusion

&nbsp;

----

## Levels in Spatial AI

{% asset_img "level_of_performance.png" "level of performance" %}

- Robotic vision, spatial understanding... 결국 무엇을 하고싶은 것인가?
- 가장 뻔한 기능부터 하나씩 발전시켜나간다고 해보자
  - 가장 낮은 레벨에서는 '**Motion tracking**'이라도 잘 하고 싶을 것이다. 적어도 내가 어디있는지는 알아야할거 아닐까?
    - **Sparse VINS**면 충분하지 않을까?
    - 이를 통해 자기 자신의 자세를 조정할 수 있을 것이다 (i.e. **position control**)
    - OKVIS, OKVIS 2
      - OKVIS 2는 만들고있다고 한다.
  - 그 다음 레벨은 '**주변 환경에 대한 기본적인 정보**'를 얻고 싶을 것이다. 이게 뭔지는 몰라도 무언가가 있다/없다 정도는 구분할 수 있어야할 것이다.
    - **Dense mapping**이 필요할 것이다.
    - 이를 통해 **Motion planning**과 **collision avoidance**를 할 수 있을 것이다.
    - ElasticFusion
  - 그 다음 레벨에서는 **'주변이 어떤 것들로 이뤄졌는지'** 알고 싶을 것이다.
    - **Semantic mapping**이 필요할 것이다.
    - 이를 통해 주위 환경과 **interaction**을 구현할 수 있을 것이다.
  - 그 다음 레벨에는 '**주변에 어떤 물건이 어디에 어떤 상태로 있는지**' 알고싶을 것이다.
    - **Object-level/ dynamic mapping**이 필요할 것이다.
    - 이쪽은 아직 연구가 많이 진행되지 않았다.

&nbsp;

---

## SLAMCore

{% asset_img "slamcore.png" "level of performance" %}

- 예에 회사홍보
- SLAMCore는 영국에 있는 로보틱스 비전을 위한 소프트웨어 회사이고, Spatial AI를 구현하기 위해 힘쓰고 있습니다~~~

&nbsp;

---

## Sparse VINS problem

{% asset_img "sparse_vins.png" "level of performance" %}

- Visual SLAM은 쉽다
- Camera pose와 3D landmark의 위치를 최적화 하는 것 뿐이다.
  - 최적화를 위한 error term은 3D landmark를 camera pose에 대해 2D 이미지로 투영하였을 때 나타나는 센서값과의 재투영 오차 (i.e. reprojection error)이다.
- 수식 설명
  - 기호 설명
    - $u$는 센서 값 (i.e. 뽑힌 피쳐의 위치)
    - $\pi$는 재투영 수식
    - $T$는 카메라 포즈
    - $l$은 랜드마크의 위치를 의미한다.
  - $T*l$은 3D landmark의 위치를 카메라 프레임에서 표현한 것
  - $\pi{(T*l)}$는 카메라 프레임에서 표현된 3D landmark를 2D 이미지로 투영했을 때의 픽셀 위치
  - $u - \pi{(T*l)}$$는 실제 이미지에서 뽑힌 visual keypoint의 위치와 2D 이미지로 투영된 3D landmark의 픽셀 위치 오차를 의미한다.
- 위 수식을 Factor graph 형태로 표현하면 아래와 같다. 

{% asset_img "visual.png" "visual-only" %}

- 각각의 pose들은 따로 떨어져있다.
  - 실제로 Visual SLAM은 graph optimisation을 할 때 모든 연속된 프레임을 다 사용하지 않아도 된다.
  - 키프레임만 사용하는 경우가 대다수이다.
- 이는, Visual SLAM에서는 temporal 정보는 없어도 된다는 것이다.
- 위 수식에서 $I$는 멀티카메라 셋팅일 경우 카메라의 수를 뜻하고, $K$는 포즈의 수, $J$는 모든 projection의 수를 뜻한다.
  - $e^{T}We$는 least squares optimisation을 표현한 것이다.

{% asset_img "vins.png" "vins" %}

- IMU를 사용하게 되면 카메라 프레임들 사이마다 IMU 값들을 얻게 되고, 이를 통해 카메라 프레임들을 measurement로 이음으로써 temporal 정보를 얻게 된다.
  - VINS가 잘되는 이유는 temporal 정보까지 함께 최적화하기 때문이라고도 볼 수 있다.
  - IMU는 종종 preintegration을 거친 형태를 가진다.
- 하지만 IMU 센서의 특성을 고려했을 때, IMU 센서의 빠른 bias 변화를 예측할 수 있어야 정확한 temporal 정보도 얻을 수 있기 때문에 최적화 해야하는 term과 constraint가 많아진다.
  - 이 때문에 VINS는 Visual SLAM보다 기본적으로 계산량이 많다는 것을 알 수 있다.

&nbsp;

---

## OKVIS

- 시간이 지날수록 최적화 해야하는 term들은 많이 쌓인다. 
  - 실시간성을 유지하기 위해 늘어나는 계산량을 어떻게 관리할 것인지가 중요해진다.
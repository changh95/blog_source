---
title: SLAM에서 사용할 수 있는 센서의 종류 
date: 2021-02-28 23:38:19
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- Sensors
- Camera
- LiDAR
- RADAR
- Ultrasonic
- IMU
- GPS
- GNSS
categories: 
- [SLAM, 50가지 CV/SLAM 기술면접 질문들]
excerpt: Camera, LiDAR, RADAR, Ultrasonic, IMU, GNSS, Wheel encoder 등 센서들의 특징과 장단점에 대해 설명
---

> 모바일로 글을 보신다면 '데스크탑 버전으로 보기'를 누르시면 좀 더 보기 쉽습니다

# 예상 면접 질문: 
1. 자율주행 (또는 드론, 또는 VR/AR)에서 쓸 수 있는 센서에는 어떤게 있을까요?
2. 다양한 센서의 값들을 어떻게 섞을 수 있나요?
3. Camera와 LiDAR의 차이점을 설명해주세요.

<br>

---

# 답

## 센서의 종류

- Preprioceptive sensors: 주변 환경의 상태와 관련없이 로봇 자기 자신의 state 값을 읽을 수 있는 센서
  - IMU, Wheel encoder
- Exteroceptive sensors: 주변 환경에 대한 상태를 읽는 센서
  - GNSS, Camera, LiDAR, RADAR, Ultrasonic

<br>

---

## Preprioceptive sensor 설명

### IMU - Inertial measurement units

- Linear accelerator (선형가속도 측정) 와 Angular gyroscope (각속도 측정)를 포함하는 센서
- [Spring-damper system](https://en.wikipedia.org/wiki/Mass-spring-damper_model)의 원리를 이용하여 만든 시스템.
  - 소형화를 위해 실제로 spring-damper 를 사용하는건 아님
    - Optical system - 차량용 IMU
    - MEMS - 칩기반 IMU (스마트폰 탑재)
- 장점:
  - 저렴한 consumer grade 제품
  - 높은 sensitivity
  - 높은 FPS (100Hz, 200Hz, 400Hz, 800Hz 제품 등 다양하게 있음)
- 단점:
  - 시간이 지남에 따라 bias에 의한 drift가 생김
    - 이러한 drift를 해소하기 위해 camera나 LiDAR를 함께 사용해서 drift 보정을 수행함.
  - 높은 정확도 + 낮은 drift를 가진 tactical grade IMU는 엄청 비쌈
    - 이런 IMU는 1시간에 1도 미만의 drift를 가짐.
    - 하지만 가격이 억대임 ㄷㄷ 
  - 낮은 가격의 consumer grade는 에러가 높고 drift도 높음.

{% asset_img "imu.png" "imu" %}

<br>

---

### Wheel encoder

- 바퀴의 회전량을 (RPM)과 이동량을 (바퀴의 회전량 * 바퀴의 둘레) 측정하는 센서
  - brush, optical, magentic, inductive, capacitative... 만드는 방법은 여러가지임
- 장점:
  - 센서가 쌈
- 단점:
  - Drift에 약함
  - 바퀴가 헛도는 경우 잘못된 센서 값이 생길 수 있음
  - (바퀴의 회전량 x 바퀴의 둘레) 를 계산할때... 바퀴의 둘레가 사실 주행 중 자주 바뀜
    - 마찰열로 인해 타이어 팽창
    - 바람빠짐 등등

{% asset_img "wheel.png" "wheel" %}

<br>

---

## Exteroceptive 센서 종류

### GNSS - Global navigation satellite system

- 비콘 기반의 위치추정 센서.
  - 보통 1. 위치추정, 2. 루트 플래닝 작업에서 많이 사용함.
  - 다수의 비콘에 대한 통신 시간에 대한 차이를 (i.e. time delays) 이용하여 비콘-로봇의 거리 값을 구하고, 삼각측량을 통해서 localization 수행.
- GPS (US), GLONASS (Russia), BeiDou (China), Gallileo (Europe) 등 각각의 나라/대륙에서 관리하는 시스템이 있음.
- 장점:
  - 싸고 구하기 쉬움
- 단점:
  - 부정확함 (10~20m 정도의 위치 오차)
    - Camera나 LiDAR를 사용해서 drift 보정을 하기도 함.
  - RTK-GPS, DGPS, AGPS 등을 이용하면 오차가 cm 단위로 내려오면서 훨씬 정확해지나, 가격도 굉장히 높아짐
  - 고층빌딩 사이에서 [multi-path 문제](https://argustracking.zendesk.com/hc/en-us/articles/333757037696-GPS-Accuracy-Bouncing-Multipath-)로 인해 오차가 생길 수 있음
  - 실내/지하에서는 신호를 아예 받을 수 없음
    - 이런 경우에는 카메라나 라이다를 사용한 map 기반 localization을 수행해야함.

{% asset_img "gnss.png" "gnss" %}

<br>

---

### LiDAR - Light detection and ranging sensors

- 적외선 레이저를 쏘아 반사되어 돌아오는 시간을 측량하여 거리를 잴 수 있는 센서
  - 시간을 측량하는데에는 3가지 방식이 있음;
    - Pulse를 이용한 Time-of-flight
    - Phase shift
    - Frequency modulation
- 3D point cloud 형태로 데이터를 받기 때문에, 주변환경의 3D 구조를 바로 알 수 있음.
  - Point cloud 를 얻는데에 2가지 방식이 있음:
    - Scanning LiDAR (Rolling shutter 방식과 유사함)
      - Mechanical Scanning
        - Macro-mechanical
        - Micro-motio
        - Prisms
      - Solid State Scanning
        - MEMS
        - Electro-optical
        - Optical phased array
    - Flash LiDAR (Global shutter 방식과 유사함)
  - (LiDAR 공부하기 좋은 자료 [#1](https://www.ti.com/lit/ml/slyp665/slyp665.pdf?ts=1610183879406), [#2](http://www.epnc.co.kr/news/articleView.html?idxno=82099))
- 장점:
  - 센서가 추정한 3D 구조가 타 센서에 비해 훨씬 정확한 편임.
  - 센서마다 다르지만, 자율주행용 LiDAR의 경우 ~100m 정도 거리까지 커버가 가능함.
  - 특정 적외선 파장을 사용하기 때문에 빛의 파장 간섭등이 잘 일어나지 않음
    - (동일한 device를 여러곳에서 사용하는 경우 제외)
  - Reflectance (i.e. 쏜 레이저와 돌아온 레이저의 신호 세기를 비교하여 반사량을 계산 가능) 
    - reflectance를 이용하여 자율주행에서는 차선이나 횡단보도 패턴을 읽을 수도 있음
- 단점:
  - 비쌈 (~몇천만원 단위)
    - 물론 가격은 지속적으로 낮아지고 있지만, 아직 카메라, RADAR, Ultrasonic 센서에 비해 비쌈.
  - Resolution의 한계
    - LiDAR 이미징 센서는 아직 카메라와 비교하였을 때 센서 resolution이 많이 부족함.
  - 날씨 영향을 받음
    - 눈이 오거나 비가 올때 레이저가 눈/비에 막히기 때문에 작동하기 어려움
  - Solid-state의 경우 360도 방향을 커버하기 위해 여러개의 센서가 필요함

{% asset_img "lidar.png" "lidar" %}
{% asset_img "lidar_multipath.png" "multipath" %}

<br>

---

### RADAR - Radio detection and ranging sensor

- 전파 (raidiowave)를 이용해서 주변 환경에 반사되어 돌아오는 전파를 측정하여 radial 거리를 재는 센서.
  - 돌아오는 전파의 각도 + 돌아오는데 걸리는 시간을 이용해서 주변 환경에 대한 거리를 추정할 수 있음.
  - [Doppler 효과](https://en.wikipedia.org/wiki/Doppler_effect)를 이용해서 물체의 속도를 측정 가능.
  - 전파의 종류를 바꿈으로써 near-range와 long-range를 정할 수 있음.
- 장점:
  - 날씨의 영향을 잘 받지 않음
  - 타 센서에서 얻지 못하는 '속도' 값을 얻을 수 있음.
- 단점:
  - 작은 물체들은 detection에 실패할 수 있음
  - LiDAR보다 resolution이 낮음
  - 주변에 벽이나 바닥에 대해 multipath 문제가 있을 수 있음

{% asset_img "radar.jpg" "Radar" %}

<br>

---

### Ultrasound

- RADAR와 작동방식이 거의 동일하나, 초음파를 이용함.
- 장점:
  - 저렴함
  - Near-range에서 잘 작동함
- 단점:
  - Geometry를 잘 추정하는 편은 아님.
    - 다중 센서를 이용해서 geometry 추정을 할 수는 있지만... 좋은 편은 아님.
  - 노이즈가 많음

<br>

---

### Camera

- 이미징 센서를 통해 빛의 정보를 받아드리고, RGB 패턴에 대해 [debayering](https://en.wikipedia.org/wiki/Demosaicing)을 통해 이미지를 만듬.
- 필터를 사용하지 않으면 Monochrome 카메라, 또는 RGB가 아닌 다른 특정 파장으로 특수 카메라 (e.g. 열화상 카메라, 적외선 카메라)를 만들 수 있음.
- 장점:
  - 사람이 보는 시야와 가장 유사한 데이터를 뽑음
  - 저렴한 센서임에도 texture가 충만한 dense한 정보를 가짐.
  - FPS가 높은 편 (30fps, 60fps)
  - 렌즈를 바꿈으로써 시야각을 바꿀 수 있음
- 단점:
  - Depth 정보가 없음
    - Stereovision을 통해 Depth를 구현할 수 있지만, 계산량이 적지 않음
  - 조명에 대한 영향을 많이 받음
    - 어두운 곳에서는 좋은 데이터가 많이 안들어옴
    - 너무 밝거나 빛반사가 카메라에 바로 오는 경우 작동하지 않을 수 있음.

{% asset_img "camera.png" "camera" %}

<br>

---

### Microphones

- 공기의 진동을 transducer를 통해 전기 신호로 변환시켜 소리를 읽는 센서.
- 여러개의 마이크 신호를 조합하여 소리의 근원에 대한 위치 및 방향을 계산할 수 있음.
- 장점:
  - 유일하게 소리에 대한 정보를 사용하는 센서
  - 가격이 저렴함
- 단점:
  - Geometry가 부정확함
  - 잡음이 심함

{% asset_img "mic.png" "mic" %}

<br>

---

## 센서를 섞는 방법

센서 값을 섞는다는 것은, 다양한 센서 값들을 기반으로 '로봇과 주변 환경의 state를 추정'하는 작업이다.
이는 [Maximum-a-Posteriori estimation](https://en.wikipedia.org/wiki/Maximum_a_posteriori_estimation)과 유사하다.
MAP는 optimisation 방식으로 (e.g. least squares optimisation, bundle adjustment) 풀 수도 있고, filter 방식으로도 (e.g. Kalman filter, Particle filter) 풀 수 있다.

{% asset_img "bayes.png" "Bayes" %}

Maximum-a-posteriori estimation을 진행하는데에는 Tightly-coupled 방식과 Loosely-coupled 방식이 있다.
Tightly-coupled vs Loosely-coupled 방식에 대해 이야기가 많지만, 두 방식 중 어떤 방식이 항상 더 좋다라고 단정할 수 없다.

### Tightly-coupled 방식

- 각각의 센서들에서 나온 데이터들에 대해 cost function을 정의하고, **모든 센서 데이터** 기반으로 optimal state를 추정하는 것
  - 예를 들어서, '$0$번째 프레임부터 $k$ 번째 프레임까지의 센서 값들을 전부 가지고 있을 때, 매 프레임마다 로봇이 어떻게 움직였고 주변 환경이 어떻게 생겨야 모든 센서 데이터 값들이 말이 될까?' 를 푸는 것이라고 볼 수 있다.
    - 여기에 [least squares optmisation](https://en.wikipedia.org/wiki/Least_squares)이 적용되는 것이라면, residual $r$ 값을 제곱한 $r^2$이 최소화되는 state의 값을 구하는 식이 된다. 이 때, 센서마다 $r$ 값을 구하는 식이 다를 것이다.
- 정확도 측면에서 loosely-coupled 방식보다 정확하다~라는 인식이 있다.
  - 하지만 항상 모든 케이스에서 그런 것은 아니다.

### Loosely-coupled 방식

- 센서들마다 state를 추정하고, 추정된 state들의 정보를 합성해서 optimal state를 추정하는 것
  - Camera로만 추정한 state, IMU로만 추정한 state, LiDAR로만 추정한 state들이 있을 때, 이 3개의 state들의 확률을 조합하여 1개의 optimal state를 구하는 것이다.
- State들에 대한 정보만 있다면 optimal state를 추정하는 시스템을 확장시켜나가는 것이 어렵지 않다.
  - 즉, 엔지니어링 측면에서 센서를 추가하거나 제거하는 것이 tightly-coupled 방식보다 훨씬 쉽다.
  - 또, 특정 조건에 대해서 특정 센서들에 더 가중치를 둘 수도 있기 때문에, 엔지니어링 측면에서 자유도가 높다고 볼 수 있다.


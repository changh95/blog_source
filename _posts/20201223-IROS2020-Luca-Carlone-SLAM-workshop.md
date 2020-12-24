---
title: IROS 2020 - Past, Present, and Future of SLAM 튜토리얼 노트 (Prof. Luca Carlone 발표)
date: 2020-12-23 20:38:19
tags: 
- SLAM
- Visual-SLAM
- CV
categories: 
- [SLAM]
excerpt: IROS 2020 학회에서 Luca Carlone 교수님께서 발표해주신 튜토리얼 영상의 노트입니다.
---

# 이번 토크의 주제는?

이번 토크에서는 3개의 주제를 훑는다.
- Spatial Perception and SLAM
- Solving SLAM via nonlinear optimization
- Open Problems

Luca Carlone 교수님의 랩실이 주로 연구하는 분야로 발표를 이끌어가신다.
이 랩실에서는 Spatial AI로는 [Kimera](https://youtu.be/-5XxXRABXJs), nonliear optimization으로는 [IMU-preintegration](https://ieeexplore.ieee.org/document/7557075) 등 큰 획을 긋는 연구를 진행하였다.

---

# SLAM이 쓰이는 곳은?

Robotics 기술은 정말 다양한 필드에 많이 쓰인다.
Robotics 기술자로써 굉장히 좋은 시기이다.

Ground : Transportation, domestics, goods/manufacturing, Mining
Air : Monitoring, disaster response, precision agriculture
Space : Exploration, Science
plus... medical applications, environmental monitoring (underwater?)

로봇은 자신이 돌아다니는 환경을 이해할 줄 아는게 중요하다.
이러한 능력을 Spatial Perception (공간 인지)이라고 한다.

---

# Spatial Perception이란?

{% asset_img "spatial_perception.png" "Spatial perception" %}

Spatial perception: 센서를 통해 얻은 실제 세상의 정보를 프로세싱을 통해 우리가 사용할 수 있는 representation으로 바꾸는 능력.

Perception을 통해 우리는 map, robot pose, poses of objects/people/other robots 등을 얻을 수 있다.
Map을 얻는 작업은 Mapping, Pose를 얻는 작업은 localization이라고 한다.

---

# SLAM이란?

SLAM 기술을 통해 sensor measurement로부터 Map의 정보와 Robot trajectory를 얻을 수 있다. 

Chicken-and-egg 문제:
- Map이 있을 때 Localization은 쉽고
- Robot pose가 있을 때 Mapping은 쉽지만
- 둘 다 없을때는 어렵다.
  - SLAM은 둘 다 없을 때 하는 것.

--- 

# 3D Map을 표현하는 방법은?

- Point cloud
- Voxel
  - 3D version of occupancy grid map
- Geometries (segments, lines, planes)
- triangular/polygonal mesh.

Point cloud나 Geometry 기반의 map을 먼저 만들경우, 이 정보로부터 Voxel이나 Mesh 기반을 만들기 어렵지 않다. 

---

# 센서 데이터 타입 #1 - Proprioceptive measurement - Odometry

**자기 자신에 대한 정보를 읽을 수 있는 센서**를 **proprioceptive** 센서라고 함.

그 중 로보틱스에서 많이 사용되는 정보는 odometry 정보임.
**Odometry**: **Relative motion between consecutive time**
센서 값을 읽을 때 마다 그 사이의 상대적인 움직임 (위치 값, 방향 값).
이 위치, 방향 값들을 state로 다룰 수 있음.

{% asset_img "odometry.png" "odometry" %}

Wheel odometry로 얻을 수도 있지만,
Visual, LiDAR 등으로 얻을 수도 있음.

{% asset_img "motion_model.png" "motion model" %}

- 센서로 추정한 이번 프레임의 state 값이 $x_{k}^{r}$ (아는 정보)
- 센서로 추정한 지난 프레임의 state 값이 $x_{k-1}^{r}$이고 (아는 정보)
- 로봇의 실제 이동량이 (**odometry**) $u_{k}$ 고 (모르는 정보)
- 노이즈가 $\epsilon_{u}$ 일 때... (대충 아는 정보)

$f$는 어떤 함수여야 정확한 odometry $u_{k}$ 값을 알 수 있는가?
Motion model은 뭐 여러 실험을 통해 얻어낼 수 있다.
Motion model을 한번 얻어내고 나면, 동일한 motion model을 사용해서 $x_{k-1}^{r}$값과 $x_{k}^{r}$을 가지고 (+$\epsilon_{u}$ 값도 있지만... 얘는 안변한다는 전제하에...) 로봇의 실제 이동량인 $u_{k}$를 추정할 수 있는 것이다.

위와 같은 방식으로 이동량을 추정하면 결과에 소량의 에러가 포함된다.
에러에는 다음과 같은 이유가 있다. 
- system model error (== motion model 함수를 정확하게 추론하지 못해 나오는 에러)
- wheel slip, gear backlash (예상치 못한 물리적인 현상)
- measurement noise (센서 노이즈)
- sensor, processor quantization error (아날로그 -> 디지털 신호 변환을 위해 생기는 어쩔 수 없는 에러)

---

# Odometry drift - Solve by extroceptive measurements

Motion model을 통해 두개의 프레임간의 이동량 (odometry)를 추정을 할 수 있다.

그러면
프레임 -> 프레임 -> 프레임 -> 프레임 -> 프레임 -> 프레임 -> 프레임 -> 프레임 ...
이 매 프레임간의 이동량을 계속 누적해나가면, trajectory를 구할 수 있을 것이다.

하지만 위의 이론은 실제에서는 작동하지 않는데, 이는 에러도 함께 누적이 되기 때문이다.
즉, 시간이 지날수록 에러가 점점 커지고 쓸 수 없게 된다.

이렇게 위치추정 값을 누적해나가는 방법을 **Dead-reckoning**이라고 한다.
실제로 항공/해양 쪽에서 쓴다고 하는데, 이 경우에는 엄청나게 비싸고 좋은 IMU센서와 GPS를 결합한 INS 시스템을 사용하는 경우가 많다.

로봇에서는 값싼 MEMS IMU나 휠 인코더 등을 사용하면 당연히 금방 오차가 쌓여버린다.

사람의 눈을 가리고 걷게하면 odometry라고 볼 수 있다.
사막에서 사람의 눈을 가리고 걷게하면 일자로 걷지 못할 것이다.
눈을 뜨게 해준다면 주위 환경을 보고 자신의 위치를 보정하며 (**relocalize**) 일자로 걸을 수 있다. 
로봇도 똑같이 주변 환경에 대한 정보를 이용할 수 있게 해주면, 제대로 자신의 위치를 추정할 수 있을 것이다.

즉, 주위 환경에 위치한 움직이지 않는 landmark를 이용하면 된다. 

---

# Extroceptive measurements

{% asset_img "extroceptive_measurement.png" "extroceptive measurement" %}

**외부 환경에 대한 정보를 읽는 센서**를 **extroceptive sensor**라고 함.
자주 보이는 센서로는 카메라, LiDAR / 레이저 스캐너, 레이더 등이 있다.

레이저 스캐너는 센서와 주위 환경 (e.g. 벽)과의 거리를 재는 센서이다.
위의 사진에서는 벽과의 거리를 재는 수식, 즉 measurement model의 (observation model이라고도 부른다) 정보를 가지고 있다.
센서로 재는 것이다 보니 어쩔 수 없디 $\epsilon_z$가 들어가긴 한다. 

{% asset_img "measurement_model.png" "measurement_model" %}

---

# SLAM as state estimation

{% asset_img "slam_state_estimation.png" "slam as state estimation" %}

- Map에 있는 landmark들의 정보
- Robot의 pose (Map 안에서의 위치+방향 정보)

이 두개의 정보를 **동시에** 추정해야한다.
이 정보들은 모두 각각의 state로 표현할 수 있다.
Landmark의 state (x,y,z)... Robot pose의 state (tx,ty,tz, rx,ry,rz)...
이 모든 state를 한번에 계산하는 것이 SLAM이 하는 것이라고 교수님은 말씀하신다. 
State의 정보는 어떤 센서에 따라 달라질 수 있다.

여기서 State의 값을 어떻게 추정하는게 좋을까?
하나의 값을 계산하는 방법이 있고, 어떠한 위치에 있을 수 있는 확률을 추정하는 방법이 있겠다.
우리는 노이즈가 존재하는 센서 데이터를 통해 계산을 하기 때문에, 단일 state 값을 계산하는 것은 위험할 수 있다. 그러므로, 가장 높은 확률을 가진 값을 추정하고, 그 값이 정확한 state일 확률을 함께 표현해주는 것이 좋다.

{% asset_img "belief.png" "belief" %}

Proprioceptive measurement $u_{1:k}$와 Extroceptive measurement $z_{1:k}$가 있을 때, $x_{k}$는 어떤 확률 분포를 가질까?
이 질문을 쉽게 표현하면 $x_{k}$의 사후확률 (Posterior probability)은 무엇인지 묻는 것이다.

---

# Brief history of SLAM

기본적인 것은 이제 알았으니, 지금까지의 SLAM 방법론의 변화에 대해 알아보자.

1990~2000 - [EKF SLAM](https://journals.sagepub.com/doi/abs/10.1177/027836498600500404)
- Extended Kalman filter 기반.

2000-2007 - [Particle Filter SLAM](https://ieeexplore.ieee.org/document/1241885) 쓰룬 형...
- Particle filter 기반

2007-PRESENT - [MAP Estimation](https://link.springer.com/article/10.1023/A:1008854305733)
- EKF and PF에 있는 단점을 보완하기위해 나타난 방법.
- 2005년 이전에는 실시간으로 작동할 수 없었다.

---

# MAP Estimation (or full smoothing)

**MAP** Estimation = **Maximum A Posteriori** Estimation

다른 이름으로는
- Factor graph optimization
- Full smoothing
- Graph-SLAM
- Smoothing and Mapping
- Bundle adjustment (Structure from Motion)

MAP Estimation은 사실 다른 분야에서도 많이 사용되는 estimation 기법이다.

---

# MAP Estimation 이해하기

{% asset_img "measurements.png" "measurements" %}

Proprioceptive measurement가 $u_{1:k}$, 즉 1번 프레임부터 $k$ 번째 프레임까지의 데이터를 가지고 있다고 해보자.
이 measurement를 측정하는 에러 $\epsilon_{u}$가 작다면, 사실상 $\epsilon_{u}$를 무시해도 될 것이다.
그렇다면 그림에 나온대로 $x_k - f(x_{k-1}, u_k) \simeq 0$ 이 된다.

Exteroceptive measurement도 비슷한 방식으로 $z_{k,j} - h(x_k,l_j) \simeq 0$이 될 수 있다.

여기서부터는 굉장히 쉽다.

$\min _{\boldsymbol{x}_{0}, \ldots, \boldsymbol{x}_{k}} \sum_{i=1}^{k}\left\|\boldsymbol{x}_{i}-f\left(\boldsymbol{x}_{i-1}, \boldsymbol{u}_{i}\right)\right\|^{2}+\sum_{(i, j) \in \mathcal{E}}\left\|\boldsymbol{z}_{i, j}-h\left(\boldsymbol{x}_{i}, \ell_{j}\right)\right\|^{2}$

Proprioceptive measurement와 Exteroceptive measurement의 에러 값들이 최소값이 되게 만들려면 $x$와 $z$가 각각 어떤 값이 되야하는가? 를 묻는 것이 MAP Estimation의 기본이라고 할 수 있다.
즉, 비선형 최적화 프로그래밍인 것이다.

제곱이 들어가는 이유는, 최적화 과정에서 에러값이 큰 경우 더 크게 페널티를 주기 위해서이다. 
예를 들어, $x$ 값이 $f(x_{i-1},u_{i})$에 비해 큰 오차를 가지고 있다면, 이 $x$ 값은 정답에서 많이 먼 값이다. 반대로, $x$ 값이 $f(x_{i-1},u_{i})$에 비해 작은 오차를 가지고 있다면, 이 $x$ 값은 정답에서 가까운 값이다.
우리는 최적화 프로그래밍을 통해, 정답에서 먼 값에서 가까운 값으로 점진적으로 구하고 싶기 때문에, 오차카 큰 값들은 크게 페널티를 주고, 오차가 작은 값들은 작게 페널티를 줘야한다.
이 때, 이 오차 값에 제곱을 시키면, 큰 값들은 크게 페널티를 주고, 작은 값들은 작게 페널티 줄 수 있다.
이런 방식을 보통 least squares 최적화 방법이라고 한다.

위와 같이 '가장 작은 에러를 가지는 state의 값들'을 추정하는 최적화 프로그래밍은, 사실 반대로 생각하면 '가장 높은 정확도/확률을 가지는 state의 값들'이 될 수 있다.

이러한 방식을 수식으로 표현하면, 아래와 같은 Maximum-a-posteriori (MAP) estimation 문제로 표현할 수 있다.
1번 프레임부터 $k$번째 프레임들(즉, 지금까지 쌓여온 모든 프레임들)의 센서 값들로 ($u$, $z$) 추정했을 때, 가장 확률적으로 말이 되는 state는 무엇인가?를 찾는 것이다.

$\max _{\boldsymbol{x}_{0}, \ldots, \boldsymbol{x}_{k} \atop \ell_{1}, \ldots, \ell_{m}} \mathbb{P}\left(\boldsymbol{x}_{0}, \ldots, \boldsymbol{x}_{k}, \ell_{1}, \ldots, \ell_{m} \mid \boldsymbol{u}_{1: k}, \boldsymbol{z}_{1: k}\right)$

{% asset_img "map_estimation.png" "MAP estimation" %}

---

# MAP Estimation - 에시

















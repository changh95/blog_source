---
title: IROS 2020 - Past, Present, and Future of SLAM (Prof. Luca Carlone 발표)
date: 2020-12-23 20:38:19
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- CV
- Kimera
- Certifiable algorithm
- MAP estimation
categories: 
- [SLAM, 학회 발표 리뷰]
excerpt: IROS 2020 학회에서 Luca Carlone 교수님께서 발표해주신 튜토리얼 영상의 노트입니다. SLAM 기술 소개, Maximum-a-posteriori (MAP) estimation, 그리고 미래에 풀어야할 숙제들에 대해 설명합니다.
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

{% asset_img "eq1.png" "eq1" %}

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

{% asset_img "eq2.png" "eq2" %}

{% asset_img "map_estimation.png" "MAP estimation" %}

---

# MAP Estimation - 예시

1개의 landmark를 가진 환경에서, 자동차가 landmark의 위치를 추정하면서 정사각형 경로를 그리며 이동하는 시나리오를 생각해보자.
자동차는 proprioceptive 데이터로 odometry 값을 추정할 수 있다. (휠 인코더 등으로...)

{% asset_img "example_1.png" "Example - before optimisation" %}

우선 Odometry (control 값)만 가지고 추정을 했을 때, 이동 경로는 이렇게 추정 될 수 있다.
Odometry 값에는 노이즈가 어느정도 껴있기 때문에, 이 odometry 값을 쌓았을 때 정사각형 모양이 나오지 않는다.

여기서 extreroceptive 결과를 보았을 때, 말이 되지 않는 결과가 나온다.
즉, proprioceptive 센서 값과 exteroceptive 센서 값이 서로 동의하지 않는 상황이 온다는 것이다.

Exteroceptive 센서가 정확하게 읽었다는 전제를 깔면, proprioceptive 센서 값으로 이동량을 추정하는 식, 즉 motion model에 에러가 있다는 것을 알 수 있다.

이 에러를 최소화하기 위해 gradient descent 방식을 사용해서 최적화를 한다. (딥러닝에서 사용하는 adam, SGD 등이 아닌, Gauss-Newton이나 Levenberg-Marquardt 방식이 사용된다.)

<br>

{% asset_img "example_2.png" "Example - after optimisation" %}

이렇게 Motion model과 Measurement model의 에러를 최소화 할 수 있는 $x$와 $l$의 state를 찾아낸다면, proprioceptive 센서와 exteroceptive 센서가 모두 동의할 수 있는 결과를 얻게 된다.

이 결과가 '가장 확률적으로 말이 되는 state'가 되겠으며, 이 값을 유도하는 과정이 Maximum-a-posteriori (MAP) Estimation이 되겠다.

---

# Factor graph

{% asset_img "factor_graph.png" "Factor graph" %}

방금 전 예제에서는 1개의 landmark와 robot pose 트랙킹에 대한 시나리오를 보았지만, 실제 SLAM에서는 다수의 landmark를 사용한다.
하지만 그렇다고 MAP 과정이 달라지지는 않는다.
단순히 고려해야할 변수의 수가 많아지는 것 뿐이다.

변수가 많아지면, 각각의 robot pose와 landmark관의 관계를 서술하기가 복잡해진다.
예를 들어, pose1 에서는 landmark1,2,3이 보일 수 있고, pose2 에서는 landmark 2,4,1042 등등이 보일 수도 있는 것이다.
최적화 프로그래밍을 할 때는 이 관계가 모두 정확하게 정의되어야하는데, 이 때 graph를 사용해서 각각 변수의 관계를 표현해주면 쉽다.

각각의 node는 pose나 landmark position을 의미하고,
edge (또는 factor)는 센서값을 포함한다.

---

# 오픈소스 라이브러리

이러한 최적화 프로그래밍 문제를 푸는 solver 라이브러리로는
- GTSAM (조지아 공대)
- g2o (프라이버그 대학)
- ceres-solver (구글)
등이 있다.

실제로 이 라이브러리들을 쓴거를 보려면 오픈소스 구현체들을 보면 좋다.
ORB-SLAM, Cartographer, ROS 구현체들이 추천되었다.

{% asset_img "large_scale_optimisation.png" "Large Scale Optimisation" %}

사진에 보이는 것 처럼 몇천~몇만개의 factor가 있는 그래프도 solver 라이브러리를 통해 몇초만에 계산할 수 있다.

---

# Sub-field of SLAM

1. Visual-SLAM
   - 카메라를 써서 하는 SLAM.
     - ORB-SLAM
     - DSO 등등
2. Visual-Inertial SLAM
   - 카메라 + IMU 데이터를 써서 하는 SLAM
     - Kimera
     - OpenVINS
     - VINS-mono 등등

... 라이다 슬램은 따로 얘기 안해주시는 것 같다.

---

# Open problems

현재 SLAM이 가지는 약점들을 (i.e. 잘 안되는 부분) 단점이라고 생각할 수도 있고, 어떻게보면 연구를 할 수 있는 기회라고 볼 수도 있다.
연구자의 입장으로는 긍정적으로 생각했으면 좋겠다.

---

# Problem 1 - Robustness, tuning

Robustness for outliers, robustness for tuning 이 필요하다.

온라인에 공개된 오픈소스 라이브러리들을 다운받아서 쓰면, Small-scale의 잘 컨트롤 된 환경에서는 잘 된다.

근데 그 이상으로 넘어가려고 하면, 엄청나게 세심하게 파라미터 튜닝을 해야하고, 새로운 환경으로 갈 경우 또 다시 파라미터 튜닝을 해야한다.

연구 단계에서는 데이터셋을 찍고 튜닝을 하고, 몇번이고 테스트하면서 튜닝을 해주면 된다.

근데 안전이 고려되어야하는 제품에서는 어떨까? 
사람이 계속해서 감독해주고 튜닝을 해줘야하는 이러한 과정은 자율주행차 등등에서 안전 측면에서 보았을 때 굉장히 위험하다.
갈길이 멀다!

{% asset_img "robust_perception.png" "Robust perception" %}

테슬라에서 만든 드라이빙 어시스턴스 시스템이 인명사고를 내는 경우도 있고,

이미 잘 되는 것 처럼 보이는 Google street view 등에서도 사람을 고용해서 잘못된 부분을 직접 수정하게 하는 작업을 하기도 한다.

{% asset_img "certifiable_algorithms.png" "certifiable_algorithms" %}

이러한 튜닝이 필요한 이유는, 각각의 환경에서 나타나는 특정 현상들에 대해 대비하기 위해 파라미터를 맞추는 것이다.
보통 우리가 예상하지 못한 현상들을 outlier 데이터라고 하는데, 이 데이터들을 잘 걸러내서 우리에게 유용한 데이터만 사용해서 SLAM 계산을 하는 것이 중요하다.

여기서 Luca Carlone 교수님의 랩실에서 연구하는 주제인 Certifable algorithms 연구에 대해 소개가 된다.
이 기술로 풀려고 하는 문제는 '수많은 Outlier에 강인한 알고리즘을 만들 수 있는가?' 이다.
이번 RAL & ICRA 2020에서 best paper finalist에 올라갔다고 한다. 

---

# Problem 2 - High-level understanding

SLAM 기술의 발전은 대부분 수학/알고리즘의 발전에 집중해왔으며, 상당수의 연구가 1. 실시간 환경에서 작동하게 만들거나, 2. 정확하게 geometry를 딸 수 있게 만드는데에 집중해왔다.

하지만 사람이 공간을 인지하는데에는 geometry만 있는 것이 아니다.
사람은 point cloud나 geometric primitives로 세상을 보지 않는다.
사람은 물체의 semantic 정보를 읽을 수 있고, context도 읽을 수 있다.
이 부분에 있어서 SLAM과 인간의 spatial perception에는 큰 차이가 있다.
이 갭이 좁혀져야한다.

High-level understanding을 처음 적용한 논문은 ICL의 Andrew Davison 교수님께서 밀고 계시는 'Spatial AI'의 개념이다.
이 랩실에서 2013년에 나온 [SLAM++](https://youtu.be/tmrAh1CqCRo)이라는 논문에서, depth image로부터 object-level reconstruction을 수행하는 것을 보여준다.
Map은 그냥 덩그러니 포인트 클라우드가 아닌, Map을 만들 때 어떤 오브젝트가 들어가있는지 이해하면서 하나씩 오브젝트를 맵에 추가해내간다는 것이다. 
영상을 보면 의자와 책상을 하나씩 map에 추가해내가는 것을 볼 수 있다.

Luca Carlone 교수님은 본인의 랩실에서 발표한 [Kimera](https://youtu.be/-5XxXRABXJs)를 소개한다.

{% asset_img "kimera.png" "Kimera" %}

1. CPU에서 돌 수 있는 
2. Metric scale을 가졌고
3. Semantic understanding을 수행할 수 있는
4. SLAM
이라고 하신다.

스테레오 카메라 + IMU를 사용해서 3D semantic metric mesh model이 나온다고 한다.

---

# Problem 3 - 소형화

SLAM 알고리즘들이 많이 '최적화' 되고 하드웨어가 발전하면서, 최근에는 정말로 가벼운 모바일 하드웨어에서도 SLAM을 돌릴 수 있게 되었다.

여기서 이야기하는 '정말로 가벼운 모바일 하드웨어'는 사실 >10W 의 전자기기... 보통 SBC 등을 이야기 하는 것이다.
과거에 비해서는 엄청나게 나아진 것이지만, 미래를 보았을 때 갈길이 멀다는 것을 알 수 있다.
왜냐면 결국 우리는 더 가벼운 플랫폼, <5W나 <200mW 등의 초소형 기기에서도 돌리고 싶을 것이기 때문이다.

이는 사람의 시각 능력보다도 훨씬 좋은 성능이여야하는 것인데...
아래 사진을 보면 아직 우리의 알고리즘은 사람의 시각 능력보다 훨씬 비효율적이고, 미래에 갈 길이 멀다는 것을 알 수 있다.

{% asset_img "human_vs_machine.png" "Humans vs Machines" %}

---

# 공부해야할 것

이론:
- Geometry
- Estimation
- Probability
- Optimization

구현:
- Real-time
- Robustness
- Impact

---

# 추천 공부 방식

이걸로 시작하면 좋다고 하신다.

{% asset_img "recommendations.png" "Recommendation" %}















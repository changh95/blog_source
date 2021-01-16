---
title: Deep Direct Visual SLAM (Prof. Daniel Cremers 발표)
date: 2020-12-25 20:38:19
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- Direct SLAM
- Deep Learning
- CV
- LSD-SLAM
- DSO
- DVSO
- D3VO
- GN-Net
categories: 
- [SLAM, 학회]
excerpt: CVPR 2020 학회에서 Joint Workshop on Long-Term Visual Localization, Visual Odometry and Geometric and Learning-based SLAM 워크샵 중 Daniel Cremers 교수님께서 발표해주신 Deep Direct Visual SLAM 영상의 노트입니다. 
---

---

# 시작하기 전...

{% asset_img "avengers.png" "어벤저스 ㄷㄷ" %}

소개되는 내용에 참여신 분들은 TUM의 Cremers 교수님 랩실의 어벤저스 이다.

Nan Yang - DVSO와 D3VO를 성공시킨 떠오르는 딥러닝 슬램 스타,
Lukas von Stumberg - GN-Net, LM-Reloc 등 기존 다이렉트 방식의 한계를 깨부수는 딥러닝 기법을 만든 연구자,
Rui Wang - 수많은 DSO의 베리에이션을 만들었고, 최신 Cremers 교수님 랩실의 굵직한 연구에는 다 참여하는 연구자,
Jörg Stückler - 막스 플랑크...,
Jakob Engel - Direct 방식을 띄워준 장본인인 LSD-SLAM과 DSO의 1저자,
그리고 Cremers 대장님

숨만 쉬어도 새 논문이 나올 것 같은 조합 ㄷㄷ

기대 만땅으로 공부를 시작합니다

--- 

# Feature-based SLAM vs Direct SLAM

SLAM의 목적은 Camera motion + 3D Structure를 구해내는 방법이다.

Erwin Kruppa라는 오스트리아 수학자/과학자가 SLAM의 기초 아이디어를 처음 제안하였다.
1913년에 (무려 100년도 전에... ㄷㄷ) "두개의 view로부터 5개의 correspondence가 있으면, 그 view들 사이의 motion과 3D 구조를 유한한 해로 추측할 수 있다'라는 것을 증명했다고 한다.

<br>

{% asset_img "keypoint_vs_direct.png" "Keypoint vs Direct" %}

<br>

## Feature-based SLAM

Kruppa의 방식을 그대로 따른 방법이 Feature-based (Keypoint-based) SLAM이다.
Kruppa가 이야기한대로, 2개의 이미지로부터 correspondence match를 만들어준다.
이 때, feature detector / descriptor를 이용한다.
또, 매치가 잘 안될 경우를 고려해서 RANSAC 등의 테크닉을 함께 사용한다.
Bundle adjustment를 수행하면 camera motion과 3D structure를 얻을 수 있다.

<br>

## Direct SLAM

Cremers 교수님은 다른 방법 (Direct method)를 제안한다.
카메라 이미지로부터는 엄청난 양의 데이터가 들어온다.
이 데이터로는 색, 밝기 등등에 대한 정보가 많이 있는데 (dense data), Point feature를 뽑는 과정에서 우리는 이걸 모두 버리게 된다 (sparse data).
심지어 Point feature를 뽑아도 이 point가 motion / structure 추정에 도움이 되는 point인지 알기 어렵다.
그렇기에 좋지 않은 point가 계산에 들어가게 될 수도 있고, 이 경우에는 motion / structure 추정에도 좋지 않은 결과를 내게 된다. (RANSAC이 어느정도 처리를 해주긴 하지만, RANSAC 파라미터 튜닝이 굉장히 귀찮은 작업이다.)
이렇게 point feature를 뽑지 않고, 카메라 이미지 자체를 사용하는 방식이 Direct 방식이다.
Direct 방식의 장점은, Raw data를 그대로 이용해서 기존의 feature detector / descriptor 방식의 에러들을 제거할 수 있다는 것이다.

두 방식의 다른 점을 뽑으라면, 
Feature-based SLAM은 reprojection error를 최소화하는 reconstruction을 목적으로 최적화 프로그래밍 (즉, motion과 structure가 서로 동의할 수 있는 추정치를 가질 때 까지)을 수행하는 것이고,
Direct SLAM은 이미지 밝기 값에 대한 에러인 photometric error / brightness consistency 값이 최소화 되는 최적화 프로그래밍을 한다 (이 과정은 optical flow등을 생각하면 굉장히 유사하다).

---

# 초기 Direct SLAM - LSD-SLAM

[LSD-SLAM](https://vision.in.tum.de/research/vslam/lsdslam)은 Jakob Engel이 ECCV 2014 학회에서 발표한 최초의 Large scale 환경을 커버할 수 있는 direct 기반 SLAM 이다.

{% asset_img "lsd_slam.png" "LSD-SLAM" %}

LSD-SLAM은 다음과 같은 방식으로 작동한다.
영상이 들어오면 Tracking과 Depth map estimation 두개의 작업이 동시에 이뤄진다.
그리고 keyframe에서 추출된 Depth map들은 Point cloud 형태의 Global map으로 추가된다. 
Large-scale에서 작동할 수 있는 이유는 오른쪽 상단의 Similarity optimisation 방식이 정확하게 계산을 해주기 때문이다.
이 방식은 원래 LiDAR SLAM에서 사용하던 것인데, trajectory를 효과적으로 계산하는 것을 보고 채용해왔다.

Cremers 교수님 말씀으로는, 대부분의 연구/작업은 Jakob Engel이 진행하였고, 본인은 LSD-SLAM이라는 이름을 만들어서 인기있게 만든 것 뿐이라고 하신다 (LSD는 마약 이름이기도 하니까, 노이즈마케팅이라고 하신다 ㅋㅋㅋ...)

<br>

{% asset_img "lsd_tracking.png" "LSD-SLAM Tracking" %}

LSD-SLAM이 Direct SLAM인 이유에 대해서 잠시 설명하자면...
Keyframe 이미지와 카메라에서 방금 들어온 최신 이미지가 있을 때, 
이 이미지를 어떤 rigid body motion (i.e. 3d rotation + 3d translation)으로 warp해야, keyframe 이미지와 똑같이 생기는가? 라는 문제를 푸는 것이다.
여기서 keyframe 이미지와 현재 이미지에서의 밝기를 (intensity) 비교하기 때문에, Direct 방식이라고 할 수 있다.
그리고 이 과정은 단순히 6개의 파라미터 (i.e. xyz rotation, xyz translation)만 추정하기 때문에 계산량이 적고, 그렇기에 CPU 속 1개의 코어에서 실시간으로 돌아간다고 한다.
나머지 코어에서는 다른 부분인 Depth map estimation과 Global realignment / pose graph optimisation을 수행한다고 한다.

이러한 방식으로 LSD-SLAM은 Large-scale에서도 굉장히 작은 drift을 가지게 된다고 한다.
[영상](https://youtu.be/GnuQzP3gty4)으로 볼 수 있다.

LSD-SLAM에는 단점이 하나 있다.
LSD-SLAM 은 camera trajectory를 계산하고나서 map에 대한 계산을 이어간다.
하지만 사실 이러한 계산은 단일 과정으로 jointly하게 풀어내는게 훨씬 더 좋다.
trajectory에 에러가 생긴만큼, map 계산에 그 에러가 전파되기 때문이다.
물론 jointly하게 계산하는 방식은 방식은 계산해야할 파라미터의 수가 훨씬 많아져서 iterative하게 풀어낼 수 밖에 없다.
그리고 iterative하게 풀기 위해서는 어쩔 수 없이 photometric bundle adjustment라는 방식을 만들어내야한다.
이 당시에는 photometric bundle adjustment라는 방식이 만들어지지 않았었다.

이 단점을 해소하기 위해 Jakob Engel은 그 다음 방법인 DSO를 제안한다.

---

# Direct SLAM - DSO (Direct Sparse Odometry)

DSO는 굉장히 robust하다.
[데모 영상](https://youtu.be/C6-xwSOOdqQ)을 보면 (약 10~15초 쯤), 주변에 사람들이 걸어다니는데도 그들을 무시하면서 잘 트랙킹을 한다.
SLAM은 원래 static environment에서만 잘된다.
하지만 DSO는 움직이는 객체가 있어도 잘 되는 것 처럼 보인다.

DSO는 7개의 연속된 keyframe 이미지로부터 camera motion과 structure를 동시에 계산하는 (i.e. joint optimisation) photometric bundle adjustment를 수행하면서, semi-dense reconstruction을 수행한다. 

DSO는 pose graph optimisation이라던지 loop closure와 같은 global optimisation을 수행하지 않는다.
그렇기 때문에 오랜기간동안 트랙킹을 진행할 경우 어쩔 수 없이 drift가 쌓일 수 밖에 없다.
그래서 DSO도 'SLAM' 이라는 이름 대신 'Odometry'라는 이름을 채택하였다.
하지만 그럼에도 drift가 굉장히 작은 것을 볼 수 있다 - 몇백 m를 움직였는데도 drift는 2-3m 밖에 나타나지 않는데, 그만큼 DSO가 정확하다는 것을 볼 수 있다.

여기서 우리는 SLAM에서 '정확도'라는 개념에 대해서 조금 생각을 해보게 된다.
Large-scale에서 정확도는 어떻게 재야할까?
Small-scale에서 정밀하게 트랙킹하며 GT 데이터로 사용되는 모션 캡처 카메라는 large-scale에서 사용할 수 없다.
GPS, IMU 등은 오차가 +- x m단위로 부정확해서 논외이다.
시뮬레이션으로 만든 방식으로는 정확한 GT를 얻어낼 수 있다.
하지만 시뮬레이션에서 잘 작동하는 SLAM 방식을 만들어도 종종 실제 환경에서 잘 작동하지 않는 경우가 있다.
그러므로 시뮬레이션 GT도 사용하기 애매하다.

Jakob Engel이 택한 방법은, real-life scene에 대한 큰 데이터셋을 구축하는 것이다.
Indoor, outdoor 환경에서 perspective, fisheye 등의 다양한 카메라 셋팅을 사용해서 약 50개의 시퀀스를 구비해놨다.
모든 시퀀스는 끝나갈 때 쯤에 시퀀스가 처음 시작한 위치로 돌아오는데, 이 때 최종 위치와 초기 위치를 align시켜 정확한 위치를 찾고, DSO가 추정한 위치 값을 비교하여 translation, rotation, scale 등에 대한 '정확도'를 추정하였다.

{% asset_img "dso.png" "DSO performance" %}

DSO의 정확도 벤치마크 결과이다.
DSO가 ORB-SLAM (잘되는 feature-based SLAM 모듈)보다 훨씬 정확하다는 결과가 나온다.
DSO에는 pose graph optimisation이나 loop closure가 없는데도 말이다.
Cremers 교수님은 '이미지를 직접 다루는 Direct 방식이기 때문에' 이런 정확도가 나올 수 있다고 이야기한다. 

<br>

---

# Deep Neural Networks

딥러닝 방식은 현재 Classical한 방식이 만드는 reconstruction 정확도를 이기지 못한다고 알려져있다.
물론 여기에는 정말 다양한 이유가 있지만, 이번 발표의 주제가 아니기 때문에 깊게 들어가지 않는다.
그럼에도 딥러닝은 지속적으로 연구할만한 가치가 있다.

<br>

{% asset_img "semantic.png" "Semantic map" %}

SLAM에 딥러닝을 섞었을 때, 현재 단계에서는 Semantic understanding을 통해 가장 효과적으로 가치를 창출할 수 있다. 

단순히 3D point cloud map을 만드는게 아니라, 실제로 어떤 object가 어디에 있는지도 알려주는 것이다.
위의 사진은 Cremers 교수님의 스타트업인 Artisense에서 만든 3D semantic map이다.
하늘색은 건물, 초록색은 식물, 주황색은 도보, 파란색은 자동차 등등이 라벨링이 되어있는 것을 볼 수 있다.
이는 여러대의 자동차에 각각 센서 시스템을 구축하고, 각각의 차에서 얻어진 맵을 하나로 합친 결과이다. 
Artisense의 목적은 저렴한 센서 시스템을 구축해서 자율주행을 위한 정확하고 쓸모있는 3D semantic map을 구축하는 것이 목표라고 한다.

<br>

{% asset_img "deep.png" "Deep learning slam" %}

실제로 딥러닝을 SLAM 파이프라인에 집어넣는 연구는 2017년부터 시작되었다고 볼 수 있다.

딥러닝을 이용해서 직접적으로 카메라 포즈를 구해내고 3D reconstruction을 수행하는 것이다.
이러한 연구들은 이런 계산이 '가능하다는 것'을 보여주었다.
하지만 아직 State-of-the-Art 성능을 보여주지는 못했다.

이러한 이유 중 하나를 이야기하자면, 많은 연구가 SLAM에서 필요로 하는 과정을 정확하게 이해하지 못하고 overly-ambitious + blind하게 end-to-end로 풀어내려는 것이라고 생각한다. 
현재의 기술로는, 뛰어난 성능의 추론능력을 보여주는 딥러닝 네트워크를 기존의 SLAM 프레임워크에 부분적으로 사용하는 방식이 가장 효과적이라고 본다.

---

# 딥러닝 SLAM - DVSO (Deep Virtual Stereo Odometry)

{% asset_img "dvso.png" "Deep Virtual Stereo Odometry" %}

Cremers 교수님 랩실에서 제일 먼저 성공적으로 만든 딥러닝 기반 SLAM은  ECCV 2018 학회에서 공개한 [Deep Virtual Stereo Odometry](https://youtu.be/sLZOeC9z_tw)이다. 

CVPR 2017 학회에서 [Kuznietsov 2017](https://arxiv.org/abs/1702.02706) 논문에서 발표된 Single image depth estimation 네트워크의 개선하여 DVSO에서는 새로운 StackNet이라는 네트워크를 제안하였다.
위에서 언급하였던 Nan Yang, Rui Wang, Jörg Stückler가 연구를 리드하였다. 
Depth estimation의 성능만 보았을 때 StackNet은 기존의 Kuznietsov의 연구보다 더 좋은 성능을 보였다.
하지만 DVSO 논문의 포인트는 그게 아니다.
DVSO는 이 StackNet을 기존의 SLAM 프레임워크에 포함시켜, Classical 방식의 SLAM보다 더 정확한 SLAM을 만들어냈다. 
StackNet을 이용해서 Keyframe에 대한 초기 depth map을 만들었다는 점과, Direct SLAM 파이프라인을 고려해서 reconstruction되는 3D 구조와 StackNet이 추론하는 depth map이 서로 동의할 수 있게 + brightness consistency를 고려한 새로운 loss function을 사용했다는 점이 이 논문의 포인트이다.
또, 이 방식은 Self-supervised learning으로 트레이닝 할 수 있다.

<br>

{% asset_img "dvso_comp.png" "DVSO Performance" %}

DVSO는 기존의 Classical 방식들과 비교했을 때, 월등히 좋은 성능을 보여준다.
특히, State-of-the-Art인 ORB-SLAM 2와 Stereo-DSO와 비교했을 때도 더 좋은 성능을 보여준다. 놀라운 점은 이 두가지 방식은 정확한 맵핑을 위해 Stereovision을 사용했음에도 불구하고, Mono camera를 사용한 DVSO가 더 좋은 성능을 보여줬다는 것이다.
또 다른 놀라운 점은 scale drift에 있다.
보통 Monocular SLAM을 사용할 때는 scale drift가 나타난다.
하지만 DVSO는 scale drift가 거의 없다.

딥러닝을 잘 사용하기만 하면 하드웨어의 한계도 뛰어넘을 수 있다는 결과가 나타난 것이기 때문에, Artisense가 추구하는 '저렴한 센서 시스템으로 정확한...' 부분과 잘 맞아떨어진다고 볼 수 있다.

---

# 딥러닝 SLAM - D3VO (3 - Depth, Pose, Uncertainty)

DVSO가 진화한 버전이 D3VO라고 볼 수 있다.

<br>

{% asset_img "d3vo.png" "What is D3VO" %}

CVPR 2020 학회에 발표된 이 [논문](https://arxiv.org/abs/2003.01060)의 주 저자는 DVSO의 주 저자인 Nan Yang이다. 
DVSO가 딥러닝 기술로 depth를 추정하고 direct slam 파이프라인에 포함시킨거였다면, D3VO는 depth, camera pose, uncertainty를 모두 딥러닝으로 추정하고 SLAM 파이프라인에 포함시키는 것이다.

<br>

{% asset_img "affine_brightness.png" "How D3VO works" %}

D3VO가 작동하는 방법은 다음과 같다.

우선, $I_t$ (시간 t에 얻는 이미지 I)를 single image depth estimation 네트워크에 넣어서 depth map을 추정한다.

동시에, $I_t$ 와 그 다음 프레임인 $I_{t'}$를 pose estimation 네트워크에 넣어서 relative motion을 (i.e. 3D rotation + 3D translation) 구한다.
이 3D transformation 값으로 이미지를 align시켜서, 전체 네트워크를 $L_{self} = r(I_{t}, I_{t'->t})$ loss로 self-supervised training을 할 수 있다.
이 값이 brightness consistency가 되겠다. 

D3VO에서 새롭게 내보이는 방식인 딥러닝을 통한 affine brightness correction도 이 단계에서 계산한다.
실제 환경에서 카메라를 가지고 이미지를 얻을 때 aperture가 (i.e. 카메라 조리개 값 == 빛을 받는 양) 변한다면 이미지의 밝기가 변한다.
즉, 동일 위치를 바라보고 있는 상태에서도 두 이미지의 밝기가 다르다는건데, 이는 기존의 Direct 방식이 풀 수 없는 조건이였다.
이 부분을 딥러닝 네트워크를 통해 affine brightness correction 값을 학습하여, 동일 위치를 바라볼 때 direct 방식으로 이미지를 비교할 수 있도록 조명의 밝기를 맞춰준다.

<br>

{% asset_img "uncertainty.png" "Aliatoric uncertainty" %}


이 affine brightness correction은 사실 빛의 반사 모델 중 하나인 [lambertian model](https://en.wikipedia.org/wiki/Lambertian_reflectance)에 기반한 방법이다. 
하지만 실제 세상에서는 사실 금속이나 유리처럼 특별히 반사가 잘 되거나 안되는 재질이 있다.
이러한 재질에서는 어떤 값이 나올지 모르기 때문에, 이에 해당하는 uncertainty를 할당해줘야하는데, 이를 위해서 [Kendall 2017 논문](https://papers.nips.cc/paper/2017/file/2650d6089a6d640c5e85b2b88265dc2b-Paper.pdf)과 [Klodt 2018 논문](https://www.robots.ox.ac.uk/~vgg/publications/2018/Klodt18/klodt18.pdf)에서 제안된 aliatoric uncertainty 기법을 사용하였다. 
이 uncertainty가 높은 부분에서는 아무래도 brightness consistency가 낮을 것이라고 예상을 할 수 있기 때문에, 이 부분에 uncertainty 값을 이용해서 최적화 과정에 weight를 줄 수 있다.
사진에 보이는 것 처럼, 금속이나 유리 재질 뿐만이 아니라 나무 꼭대기처럼 motion이 생겨서 높은 brightness consistency를 가지는 부분은 높은 uncertainty를 가지는 것 처럼 보이게 된다.

그 후, 모든 Depth, Uncertainty, Pose, Affine brightness correction을 Frontend tracking과 Backend optimisation에 넣는 것이다.
Frontend에서는 nonlinear factor graph를 사용해서 brightness consistency를 계산하고, Backend에서는 reconstruction된 모델이 pose와 depth에 대해 추론한다 (추론 모델은 이 둘에 대해 사전 트레이닝 되어있다).

{% asset_img "d3vo_depth.png" "Depth estimation" %}

결과를 보았을 때, 우선 depth estimation의 성능을 보면 sota 모델 중 하나인 MonoDepth2보다 더 좋은 성능을 냈으며, 굉장히 좋은 generalization을 보여줬다.
반사가 잘 되는 재질에서 aliatoric uncertainty가 굉장히 잘 나타나는 것을 볼 수 있다.

{% asset_img "d3vo_vo.png" "Visual odometry" %}

SLAM에 대한 결과를 보면, classical method나 deep learning 기반 기술 양쪽 모두에서 State-of-the-art를 달성한 것을 볼 수 있다.
특히, Stereo Visual-inertial odometry인 [Basalt](https://arxiv.org/abs/1904.06504)와 동일한 성능을 내는 것을 볼 수 있는데, 여기서 다시 한번 D3VO는 monocular visual odometry인 것을 생각하면 엄청나게 성능이 좋다는 것을 알 수 있다.

{% asset_img "d3vo_recon.png" "Reconstruction" %}

Mono DSO와 D3VO의 reconstruction 결과를 비교했을 때도, D3VO의 결과가 훨씬 깨끗한 점을 알 수 있다.

---

# 딥러닝 SLAM - GN-Net (Multi weather relocalization)

{% asset_img "vis_loc.png" "Visual localization under lighting and weather" %}

이쯤되면 Direct 방식의 가장 큰 문제점을 알 수 있다.
조명이 바뀌어버리면, photometric 최적화를 할 수 없다는 점이다.
그렇다면, 동일한 장소라도 다른 시간대에 온 경우에는 (낮/밤, 여름/겨울) 조명이 다르니 동일 장소로 인식할 수 없을 것이다.
위에 있는 사진을 예시로 들면, 여름에는 비가와서 물이 고였고 잔디가 파릇파릇하게 자랐고, 겨울에는 눈이 덮혔다.
이 때문에 이미지 속 밝기 값이 많이 다르게 보이고, 이전에 봤던 위치랑은 다르게 인식이 될 것이다.

하지만 이러한 high-level representation은 딥러닝 기반 기술이 제일 잘 푸는 문제 중 하나이다.
이 문제를 풀어보기위해 데이터셋을 제작하고, [GN-Net](https://arxiv.org/abs/1904.11932)이라는 기술을 von Sturmberg가 발표하였다.
GN-Net은 high-dimensional feature space에서 intensity transformation을 하는데, 사실 이러한 방법은 이전에 다른 네트워크에서도 많이 제안되었던 방법이다.
이 논문이 다른 점은, Gauss-Netwon 기반 최적화를 하는 SLAM에서 좋은 결과를 낼 수 있도록 학습했다는 점이다.

<br>

{% asset_img "gn-net.png" "GN-Net" %}

여기 이미지에서 왼쪽 아래 빨간 박스에 쳐진 이미지는 현재 보고있는 이미지, 오른쪽 아래 박스에 쳐진 이미지는 트레이닝 데이터셋에서 보았던 가장 비슷한 이미지이다.
그리고 회색 포인트 클라우드는 이전에 만들었던 포인트 클라우드이고, 파란색 포인트 클라우드는 새로 만들어지는 포인트 클라우드이다.
그래서 사진을 보았을 때, 회색에는 없지만 파란색에는 있는 차가 있지만, 바닥에 그려진 차선은 정확하게 일치하는 것을 볼 수 있다.

GN-Net은 GPS 센서 없이 direct 방식만으로도 relocalization을 할 수 있다는 것을 증명하였다.
또, 지금까지의 direct 방식에서는 odometry 방식으로 SLAM을 했왔는데, 이제는 large-scale에서 필수적인 relocalization을 할 수 있다는 것을 증명하였다.

---

# 발표에 대한 질문

## Photometric error를 사용할 때의 단점이 있다면 무엇인가요?

Classical한 방식으로 Brightness consistency는 특정 상황에서 깨지게 됩니다.
그래서 이러한 단점을 해결하기 위해 딥러닝 기법을 적용하였습니다.
이번 토크에서는 3가지 방법을 언급하였습니다.

- Affine brightness change
- Aliatoric uncertainty
- Brightness variation (multi-weather)

네트워크가 brightness variation을 이해하는것이 중요합니다.

## 다른 센서와 퓨전한다면 어떤 것과 해야할까요?

이 질문은 연구보다는 실무에 계신 분들이 하실 것 같습니다.
이번 토크에서는 Visual-SLAM만을 특정하여 이야기 하였고, 카메라만 가지고 어느정도까지 뽑아낼 수 있는지에 대해 이야기하였습니다.

하지만 실제 환경에서 자율주행차나 로봇에서 사용할 것이라면, 가능한 센서 조합을 많이 테스트해보고 제일 정확한 방법을 사용하는 것이 맞습니다.
저희가 테스트 했을 때는 fast rotation 등이 있었을 때 IMU가 굉장히 도움이 됬었습니다. 
GPS (RTK GPS)가 있다면 global reference도 얻을 수 있구요.
LiDAR나 RADAR도 굉장히 좋구요.

하지만 제가 드리고 싶은 조언은, 센서 퓨전을 할것이라면 정확히 '어떤 목적을 위해서 센서 퓨전을 하는지' 아는 것이 중요하다고 생각합니다.
왜 이 센서를 퓨전하는 것이고, 어떤 방식으로 퓨전을 할 것이고를 충분히 이해해야 센서들의 장점들을 섞을 수 있는 것이고, 제대로 이해하지 못한다면 센서들의 단점만 섞게 될 수도 있습니다.

Artisense에서 사용하는 Stereo Visual-inertial 시스템은 3km 정도를 주행하고 왔을 때 약 1 m 정도 에러가 나타나는데, 이는 센서 퓨전을 잘 수행했기 때문에 에러를 줄일 수 있었던 것으로 봅니다. 

## Robustness와 Accuracy를 위해서 End-to-End를 공부하는게 좋을까요? 아니면 Hybrid를 공부하는게 좋을까요?

지금 상황에서는 답변하기 어려운 질문입니다.
Artisense에서는 둘 다 연구를 하고 있습니다.
딥러닝은 correspondence estimation, single image depth estimation 등의 기술에 굉장히 효과적입니다만, classical 방식에서 optimisation 기법 역시 굉장히 효과적인 방법입니다.
우리가 알고 있는 것도 잘 쓸줄 알아야하죠 - 카메라의 작동 원리라던가, 이미지 투영 방식, rigid body motion과 perspective projection을 통해서 이미지가 만들어지는 과정이라던지, epipolar geometry라던지... 우리는 정말 잘 알고 있습니다. 
딥러닝은 효과적인 방법이긴 하지만, 잘 될거라는 확신을 가지고 깊게 이해하지 못한 채 막 쓰는 것은 좋지 않고, 기존의 classical 방식을 통한 인사이트를 기반으로 어떤 목적으로 사용하고 어떻게 적용 될지 정확히 이해했을 때 쓰는 것이 좋습니다. 

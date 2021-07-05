---
title: 3DGV 2021 - Representations and Computational Patterns in Spatial AI (Prof. Andrew Davison)
date: 2021-04-07 22:14:31
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- Deep Learning
- Deep SLAM
- Rearrangement
- Embodied AI
- Spatial AI
- RLBench
- iMAP
categories: 
- [SLAM, 학회 발표 리뷰]
excerpt: 3DGV 세미나 중 Prof. Andrew Davison 교수님 세미나 정리. Rearrangement, RLBench, iMAP 연구 소개.
---

{% youtube ERCqnJnLH9Y %}

토크의 상당 부분은 [이전 세미나](https://changh95.github.io/20201226-CVPR-2020-SLAM-workshop-Davison/)와 내용이 겹칠 것으로 예상된다. 이번 글에서는 새로 업데이트 된 내용만 커버하기로...

---

## Spatial AI - Perception for Embodied Intelligence

- To enable embodied AI, a Spatial AI system should build **a persistent and understandable scene representation** which is close to metric 3D geometry, at least locally.
  - Map 처럼 생겼다던가...
  - 쉽게 update 가능한다던가...
  - Planning 같은게 쉽게 만들어졌다던가...
- Generality...
  - 다른 도메인에 사용되더라도 Spatial AI는 공통적으로 가지는 특성이 있을 것이다.
    - 드론, 로봇, AR... 상관 없다.

&nbsp;

---

## Rearrangement: A Challenge for Embodied AI

- 다양한 분야의 전문가들과 함께 만든 discussion 논문.
- **"Spatial perception, navigation, interaction을 위해 어떤 challenge를 푸는게 좋을까?"에 대한 전문가들의 답변.**
- 여러가지 질문들
  - 하나의 dominant task를 수행하기 위해 subtask는 어떻게 알아낼 것인가?
  - 각각의 agent가 얼마나 task를 잘 수행하고 있는지 알 수 있을까?
    - Task specification, settings, evaluation
    - 로봇이 방을 치우는 task라면, tidy-state라는 것은 무엇인가?
      - 정확한 3D coordinate을 줄까?
      - 대충 이미지 하나를 던져줄까?
      - Exploration을 통해 알아서 학습하게 할까? (강화학습?)
    - 다양한 '정확도'로 방이 치워질 수 있는데, 이 때 '방이 깔끔해진 정도'를 어떤 metric으로 결정해야할까?
      - 테이블 위에 접시를 놓고, 접시 위에 칼을 놔야한다면...
      - 접시는 테이블 위 아무데나 있어도 되지만, 칼은 더 좁은 공간인 접시 위에 있어야한다. 이 경우 어떻게 score를 매길 것인가?
- 교수님이 이 연구에서 관심을 가지셨던것은 '**어떤 방식으로 scene representation을 만들어야 real-time, persistent, updatable할 수 있을까?**' 라고 하셨다.
- 이런 문제를 풀기 위해서 end-to-end로 푼다는 것은 아직 상상도 못하겠다.
  - 어떠한 intermediate representation이 꼭 필요할 것이라고 생각한다.

&nbsp;

---

## RLBench

- Rearrangement 논문에서 참고된 시뮬레이션 소프트웨어.
  - Robot learning을 위해 만듬.
- Prof Andrew Davison은 원래 벤치마크를 별로 안좋아하심
  - 근데 이건 교수님께서 직접 만드셨네...? ㄷㄷ 뭐지

&nbsp;

---

## Creating map representations...

- [이전 세미나](https://changh95.github.io/20201226-CVPR-2020-SLAM-workshop-Davison/)와 내용이 많이 겹침
- MonoSLAM
- ElasticFusion
- SemanticFusion
- Semantic SLAM Computation Graph
- SLAM meets deep learing
  - End-to-End algorithm <-> Fully human-designed algorithm
- CodeSLAM, SceneCode, DeepFactors

&nbsp;

---

## iMAP - Implicit Mapping and Positioning in Real-Time

{% youtube c-zkKGArl5Y %}

- Uses **implicit neural representations**, like how [NeRF](https://arxiv.org/abs/2003.08934) is using the thing
  - 하지만 이 논문에서는 RGB-D를 사용하기 때문에 훨씬 쉽다고 교수님이 얘기하심.
- Multi-layer perceptron 내부에서 처리되는 값들만을 가지고 실시간으로 scene representation을 표현.
  - 사실상 **실시간으로 multi-layer perceptron을 학습**하면서 값들을 사용하는 것.
  - **xyz coordinate을 인풋으로 주면, scene속에 있는 어떤 3D point의 occupancy와 colour 값을 아웃풋으로 내뱉는 네트워크**임.
- 카메라가 어디있는지 알고 있고, 센서 인풋으로 raw colour + depth 이미지가 들어올 때, 우리는 **MLP 네트워크가 내뱉는 아웃풋이 raw colour + depth 이미지에 매칭되게 optimise**할 수 있다.
  - 즉, RGB+Depth 이미지는 optimizing cost function을 위해서만 사용되고, 실제로 SLAM 계산에는 사용되지 않음? 
- 이러한 결과물을 SLAM 시스템으로 변환하기 위해서는, RGB+Depth render 값들이 어떠한 하나의 map representation으로 수렴할 수 있어야한다.
  - 여기서 **Keyframe 구조**를 사용해서 맵을 딴다.
  - 교수님께서는 PTAM을 예시로 들면서 tracking / mapping이 쓰레드로 따로 도는 것을 설명하셨다.
  - 영상에서 빨간색 피라미드 형태로 나타나는 것이 keyframe이다.
- Keyframe은 DB를 만들어서 관리한다.
  - Mapping thread에서는 이전에 취득한 키프레임과 가장 최신의 프레임을 이용해서 정보를 공유한다.
  - 모든 키프레임을 다 쓰는거는 아니다. 그러면 너무 느릴것이다.
  - **'가장 유용한 키프레임'**을 고르고, 그 후 그 키프레임에서 **'가장 유용한 point'**를 뽑아 최적화한다.
    - 이런 방식으로 **계산해야할 포인트의 수를 줄여서 실시간 작동**을 구현할 수 있었다.
    - 2Hz (ㅋㅋ 겨우 실시간)으로 실시간 맵핑을 할 수 있었다.
- 진짜 신기한건, 아직 보지도 않은 공간의 데이터도 대충 채워넣을 수 있다는 것이다.
  - 딥러닝 없이 RGB-D SLAM으로 했을 경우에는, 보지 못한 공간은 구멍이 숭숭 뚫려있다.
- 결국에 그냥 SLAM한거랑 뭐가 다른거지...? 라는 생각이 들긴했지만,
  - 교수님 말씀으로는 '**하나의 네트워크를 가지고 tracking + mapping을 다 할 수 있는 neural representation을 만든 것이다**' 라고 하심
  - 그리고 그 네트워크는 **단지 1mb의 작은 모델**이다. (호우,,,)


&nbsp;

---

## Object-based SLAM

- [이전 세미나](https://changh95.github.io/20201226-CVPR-2020-SLAM-workshop-Davison/)와 내용이 많이 겹침
- SLAM++
- MoreFusion
- Fusion++
- NodeSLAM

&nbsp;

---

## Processor for Spatial AI

- [이전 세미나](https://changh95.github.io/20201226-CVPR-2020-SLAM-workshop-Davison/)와 내용이 많이 겹침
- SLAMBench
- Graphcore IPU
- Futuremapping
- Bundle Adjustment on a Graph Processor 

&nbsp;

---

## Research plans for Gaussian Belief Propagation & Factorized Computation

- Dynamic, self-abstracting graph map: Object-based SLAM?
- General dense front-end image processing
  - Unifying flow estimation and segmentation
  - Smart cameras
- Multi-device distributed mapping
  - Towards a robot web?
- GBP learning
  - Continual, self-supervised convergent

&nbsp;

---

## 질문

Q. iMAP에서 implicit neural representation을 쓴 이유는 무엇인가요? 그냥 mesh에 optimisation하면 안되나요?
A. Mesh optimisation은 굉장히 어렵다. Topology 자체가 굉장히 복잡한데, 그 위에 새로운 measurement가 생길때마다 editing을 해야한다면 끔찍하다. 솔직히 어떻게 해야할지 모르겠다. 3D representation에는 여러 방법이 있고 각각의 장단점이 있다.

Q. iMap을 large-scale로 가져가려면 어떤 난관이 있을까요?
A. Large-scale을 고려하고 실험한적은 아직 없다. 아마 더 큰 네트워크가 필요하지 않을까? Reprsentation을 더 풍부하게 하고 싶다면 네트워크는 커져야할것이다.

Q. iMap의 경우 online learning이 된다는 것이 굉장히 특이하고 신기하지만, 기존의 Deep SLAM 방법론들은 미리 학습해온 모델을 사용하는 경우가 많습니다. 이 두가지를 혼용할 경우 정확도는 당연히 떨어지게 될 것으로 보이고, 이는 두가지 방법론이 적대적으로 보이기도 하는데, 어떻게 이 두가지 연구방법론을 유지하면서 정확도/성능을 높일 생각이신가요? (Stefan Leutenegger 교수님 질문 ㄷㄷ, Davison 교수님 당황 ㅋㅋ)
A. 이 방법론 둘 다 해야한다. 하지만 우리는 두 방법이 각각 장단점을 가지고 있다는 것을 안다. Pre-learned된 방법들은 적당히 좋은 성능을 보이지만, 새로운 환경에 가져갔을 때 항상 성능이 애매해지는 것을 볼 수 있다. iMAP 같은 방법은 새로운 환경에서도 바닥부터 새로 트레이닝하면서 좋은 결과를 만들 수 있지만, 매번 새롭게 트레이닝 해야한다는게 매우 귀찮을 것이다. 미래의 로봇은 사실 이 두개를 동시에 다 해야할것이다. 로봇을 샀으면 키자마자 뭔가를 해야하지 않을까? (ㅋㅋ) 하지만 새로운 환경인만큼 새로운 무언가를 배우긴 해야할 것이다. 예를 들어 기본적인 맵핑 알고리즘들은 탑재했지만, semantic 정보를 한 10~30분정도 학습하는건 어떨까? Bayesian의 생각으로 이야기할 때, 결국 **우리는 새로운 것을 배우는 방법**과 **이전에 배운 것 (prior)을 잘 활용하는 방법들**을 **잘 밸런스** 할 수 있지 않을까 생각한다.

Q. 이 두 방법을 밸런스해야할까요? 아니면 두개를 병렬처리해야할까요?? (ㄷㄷㄷ)
A. 

Q. Unified representation이 정말 필요할까요?
A. 당연히 상황에 따라서 여러 파라미터가 조정되어야한다. Rearrange task의 경우에는 semantic class / instance도 잘 알아야할 것이고, 정확도도 중요할 것이다. 단순히 장애물을 회피하는 로봇이라면 굳이 그런걸 알 필요는 없다. Manipulation 로봇이라면 mm 단위의 정확도가 필요할수도 있고, 아닐수도 있다 (suction cup같은걸 쓰면 mm단위는 필요없다). Representation에 대해 연구하면서 여러가지 고민을 새롭게 하게 된다.

Q. iMAP이 맵 하나를 implicit representation에 넣는 것은 잘 보았다. Object-level SLAM을 할 때도 implicit representation이 될 것 같은가? 아니면 graph라던지 explicit representation밖에 없는것인가? (Angela Dai 교수님 질문)
A. 아직 잘 모르겠다. 하지만 iMAP을 만들면서 본 것이, 매 프레임마다 맵 자체가 굉장히 흔들리는 것을 볼 수 있다. 이 흔들림은 사실 SGD 등을 통해 네트워크를 최적화하면서 모델 weight가 바뀌는데, 이 때 특정 object (map안에 있는 축구공이라던지)를 자세히 보면, 이 흔들림이 object단위로 균일하게 흔들리는 것을 보았다. 아마 이 object에 대한 weight 파라미터는 그리 많지 않을 것이다. 나는 이를 통해 네트워크 안에 implicit한 형태로 object에 대한 representation이 들어있다고 보고있다. 이러한 representation을 code로 뽑아낸다던지... 등은 아직 잘 모르겠다.

Q. 그렇다면 몇개의 파라미터만 바꿔서 object를 다룰 수 있게 된다면...  Object를 마음대로 옮긴다던지, 또는 scene deformation을 만들어서 loop closure라던지 구현할 수 있을까요? 아무래도 SLAM 특성 상 이런 방향으로 가고싶을 것 같은데요.(Stefan Leutenegger 교수님 질문)
A. 나도 이런 방향으로 연구하고싶다 (ㅋㅋ) 몇개의 파라미터만 조정해서 loop closure를 만들 수 있을까? 그게 될지는 잘 모르겠다. 진짜 잘 모르겠음 (이 말 하시고 고민에 빠지심 ㅋㅋ)

Q. Rearrangement를 할 때, indirect한 effect가 있다면 이것은 implicit representation이 되야할까? explicit representation이 되야할까? 
A. 

Q. CAD-model 없이 Object-level SLAM이 가능할까요?
A. 어후... 어려울거같다. Class만 가지고 모든 instance의 geometry를 추정할 수 있을까? 사람은 가방을 보았을 때 어디에 포켓이 있고 어디에 지퍼가 있고 등등 잘 알고있지만, 딥러닝은 아직 그정도까지 깊게 이해하고 있지는 않는 것 같다. 간단한 컵/접시 모양은 할 수 있겠다만, 이것 이상의 복잡한 물체들은 아직 많이 어렵다고 본다.
코멘트: 그게 된다면 그게 Spatial AI일듯 ㅋㅋ (Angela Dai교수님 ㅋㅋ)
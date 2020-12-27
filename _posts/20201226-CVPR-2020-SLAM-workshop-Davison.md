---
title: CVPR 2020 - SLAM 워크샵 From SLAM to Spatial AI 발표 노트 (Prof. Andrew Davison 발표)
date: 2020-12-26 20:38:19
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- Spatial AI
- Deep Learning
- CV
- MonoSLAM
- DTAM
- KinectFusion
- ElasticFusion
- SemanticFusion
- CodeSLAM
- SceneCode
- DeepFactors
- SLAM++
- MoreFusion
- Fusion++
- NodeSLAM
- FutureMapping
- Graphcore
- IPU
- Graph processor
- Andrew Davison
categories: 
- [SLAM]
excerpt: CVPR 2020 학회에서 Joint Workshop on Long-Term Visual Localization, Visual Odometry and Geometric and Learning-based SLAM 워크샵 중 Andrew Davison 교수님께서 발표해주신 From SLAM to Spatial AI 영상의 노트입니다. Andrew Davison 교수님 랩실의 연구를 주로 커버합니다.
---

# Visual-SLAM

로봇, 드론, 모바일/스마트폰 AR 소프트웨어, VR/AR 헤드셋 등에서 Visual-SLAM을 사용한다.
이는 카메라가 작고 싸기 떄문에 제품화하기 좋기 때문이다.
또, 이미지를 통해 정말 다양한 기능을 이용할 수 있기 때문이다.

Visual-SLAM을 사용하는 이유는 대부분 positioning 또는 어느정도의 sparse/semi-dense mapping을 수행하기 위해서이다.
sparse / semi-dense mapping을 넘어, dense / semantic mapping을 사용하는 제품들도 점차 나타나기 시작했다.

<br>

{% asset_img "level_of_performance.png" "Levels of performance in SLAM" %}

Davison 교수님이 멤버로 계신 SLAMCORE라는 스타트업에서 정의한 SLAM 기술의 단계는 위와 같다.

SLAM은 앞으로 Spatial AI의 일부로 들어갈 것이다.
Spatial AI는 컴퓨터 비전을 사용하여 실시간으로 디바이스가 주변 환경과 상호작용을 할 수 있게 해주는 AI 시스템이다. 

SLAM의 목적은 localization 한번 하는 것, mapping 한번 하는 것으로 끝나는게 아니다.
SLAM의 성능은 제품에 모듈로써 포함될 수 있는 정도가 되어야한다.
실시간으로 작동하고 인터랙션을 할 수 있는 데모를 볼 수 있어야한다.
직접 데모로 보여주던가, 제품을 내던가, 아니면 오픈소스 프로젝트로 오픈할 수 있어야한다.
논문에서 데이터셋을 이용한 벤치마크 성능을 보여주는 것 보다 이것이 더 중요하다.

---

# Spatial AI, Spatial IA가 풀어야 할 숙제

## Spatial AI가 풀어야 할 숙제

대량생산될 청소 로봇을 만든다고 생각해보자.
이 로봇은 복잡한 구조의 방과 물건들 사이에서 닦고 쓸줄 알아야한다.
이 로봇을 설계할 때, 우리는 이 로봇의 가격, 디자인, 크기, 안전, 소비전력 등을 고려해야한다.
그리고, 이 모든게 보통의 사용자가 쓸 수 있는 범위 안에 있어야한다.

지금 SLAM의 단계로는 복잡한 구조의 방과 물건에서 정확하게 localization / mapping에서 어려울 수 있다.

<br>

## Spatial IA가 풀어야 할 숙제

우선 Spatial IA는 Intelligence Augmentation을 줄인 말이다.
간단하게 생각하면 증강현실을 이용해서 주변 환경과 사물에 대해 인터랙션을 추가한 것이라고 볼 수 있다.

증강현실 안경을 만든다고 생각해보자.
우선 이 안경의 크기는 보통의 안경과 같은 사이즈여야한다.
약 65g 정도의 무게를 가지고, 배터리는 하루종일 돌아갈 수 있어야한다.
동시에, 실시간으로 정확하게 정보를 증강시킬 수 있어야하며, 증강되는 정보는 주변 사물, 환경, 사람 등에 대한 정보를 모두 포함한다.

지금 [MS 홀로렌즈](https://www.microsoft.com/en-us/hololens) 정도면 사전에 등록해놓은 정보들을 실시간으로 증강시키고, 사람의 손과 눈 정보는 읽을 수 있다.
하지만 이 경우, 무게는 약 600g에 배터리는 몇시간 남짓하지 않고, 가격은 5백만원 대이다.

<br>

{% asset_img "gap.png" "Product gap" %}

여기까지 생각하면 몇가지 질문이 생긴다.
어떻게 만들어야할까?
만들 수 있기는 한걸까?
만든다면 누가 만들것인가?

물론 아직 아무도 만들어본 적이 없기 때문에, 어떻게 만드는지는 아무도 모른다.

대학과 기업 연구실에서 많은 연구를 하고 있지만, Spatial AI까지의 길은 아직 멀고도 멀어보인다.
특히, Robustness와 Efficiency에서 많이 발전해야할 것 같다.

<br>

Davison 교수님께서 
1. Spatial AI가 풀어야할 문제들과 
2. 이 문제를 다 풀어낸 Spatial AI의 모습은 어떨지 
구상을 하시고 적으신 [논문](https://arxiv.org/pdf/1803.11288.pdf)이 있다.

교수님이 구상하시는 Spatial AI의 특징을 두개만 꼽으면 아래와 같다.

1. 3D geometry의 정보를 내포하고 있는 어떠한 map에 대한 정보가 있어야한다.
2. 튜닝 파라미터의 수가 적어야한다 + 몇개의 파라미터를 변경하는 것으로 여러 도메인의 spatial AI를 넘나들 수 있어야한다 (e.g. 로봇 -> AR).

---

# SLAM의 역사와 발전 방향

SLAM이 미래에 어떻게 될지 보기 위해, 과거로부터 거슬러 올라가보자.

## 1. MonoSLAM

{% asset_img "monoslam.png" "MonoSLAM" %}

가장 초기의 SLAM으로 2003년에 만들어진 [MonoSLAM](https://www.robots.ox.ac.uk/ActiveVision/Publications/davison_iccv2003/davison_iccv2003.pdf)을 소개하셨다.
Hand-held 카메라를 사용해서 feature detection + tracking을 수행하고, Extended Kalman filter를 통해 camera pose와 map point의 위치를 jointly하게 계산해냈다.
2003년의 노트북으로 30FPS의 성능이 나왔고, 성공적으로 실시간 데모를 할 수 있었다.

물론 단점도 있었다.

1. feature의 수가 작았고, 
2. robust하지 않았다.

그렇기에 feature가 많은 공간에서 천천히 움직여야만 작동했다.

<br>

---

## Dense / Direct Visual SLAM의 시작

## DTAM

{% asset_img "denseslam.png" "Dense SLAM" %}

2009년에 GPGPU를 사용해서 실시간 dense reconstruction을 성공시켰다.
당시 Davison 교수님 랩실에서 박사과정을 하고 있던 Richard Newcombe이 연구를 주도하였다.
이 연구를 발전시켜 2011년에 PTAM의 dense버전인 [DTAM](https://youtu.be/Df9WhgibCQA)이 발표되었다.
PTAM은 sparse map밖에 쓸 수 없었지만, DTAM은 이를 dense map으로 확장을 했다고 볼 수 있다.
이 연구를 통해 sparse map에 비해 dense map이 가지는 장점을 확실하게 알 수 있었다.

## KinectFusion

조금 다른 방향의 연구가 진행된 것도 있다.
Microsoft Kinect와 같은 RGB-D 카메라를 사용한 연구이다.
2010년에는 [KinectFusion](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/ismar2011.pdf)이 발표되었다.
이 연구 역시 Richard Newcombe의 작품이다.

DTAM의 경우는 monocular 카메라를 사용해서 dense map을 만드는 계산 방법을 만들어내는데에 많은 고민을 했다고 한다.
하지만 KinectFusion의 경우 하드웨어의 힘을 받아 dense map을 만드는 작업을 손쉽게 성공시켰다.
그리고 next step이라고 생각했던 '좀 더 큰 맵'을 만드는 과정과 volumetric representation 기술을 연구할 수 있게 되었다고 한다.

<br>

## ElasticFusion, SemanticFusion

{% asset_img "semanticfusion.png" "Semantic Fusion" %}

2016년에는 loop-closure가 가능한 [ElasticFusion](https://youtu.be/XySrhZpODYs)을 발표하였다.
이 방식은 기존의 KinectFusion에서 진화한 버전이라고 볼 수 있다.
그리고 2017년도에는 ElasticFusion으로 만들어내는 Map 정보에 딥러닝 CNN 기술로 semantic 정보를 추가한 [SemanticFusion](https://youtu.be/_aqP9rumkgQ)이 발표된다.
이 연구를 통해 dense semantic SLAM, deep-SLAM 연구가 시작되었다.

---

## Deep SLAM에 필요한 계산 설계도

SLAM 기술이 발전되면서 몇가지 트렌드가 극명하게 드러났다.
1. 정확도의 향상을 위해 점점 더 많은 양의 계산이 요구되었다.
2. 실시간성 유지를 위해 병렬처리를 요구하게 되었으며, 이에 따른 하드웨어의 수도 늘어났다.
3. 새로운 기능들이 추가하면서 점점 더 많은 데이터의 타입을 저장하게 되었다.

SLAM 시스템은 점점 더 복잡하게 바뀌어갔다.
과거와 비교했을 때, SemanticFusion과 같은 deep SLAM의 설계 구조를 보면 굉장히 복잡했다.
그럼에도 SLAM은 계속 발전해나가야한다.
하지만 이 트렌드가 유지된다면, Spatial AI 단계에 다다랐을 때 시스템이 요구하는 설계는 겉잡을 수 없을정도로 복잡할 것이다.

아래는 간단한 Spatial AI가 수행해야하는 연산의 관계와 데이터의 타입들을 그래프 형태로 정리한 것이다.

<br>

{% asset_img "computation_graph.png" "Spatial AI computation graph" %}

수많은 다양한 종류의 계산과 데이터가 오고가는 것을 볼 수 있다.
이 뜻은, 어떠한 계산에 최적화가 들어가도 해당 계산만 최적화가 될 것이라는거다.
즉, 최적화를 해도 전체 시스템의 성능에 큰 영향을 주지 못할 수 있다.
하드웨어 디펜던시도 걸려있다.
딥러닝 추론과 같은 계산은 병렬처리에 특화되어 GPU와 같은 하드웨어에서 잘 돌지만 CPU에서 잘 돌지 못할것이다.
반대로 최적화 프로그래밍 계산은 고성능 클럭을 가진 CPU에서만 잘된다.
하드웨어 간의 통신도 고려해야하며, 종종 큰 데이터가 이동해야할 때 효율성이 급격하게 떨어질 수도 있다 (e.g. CPU->GPU 정보이동).
정리하자면, 전체 시스템의 최적화을 하기에는 수많은 제약이 걸려있으며, 최적화는 굉장히 어려울 것이다.

고성능 고효율의 spatial AI를 만들기 위해서는 이 그래프 전체를 최적화해야한다.
Davison 교수님은 두가지 방법을 제안하셨다.
1. Representation을 개선하기
2. 하드웨어를 개선하기

--

# CodeSLAM, SceneCode, DeepFactors - 딥러닝 keyframe 정보 압축

## CodeSLAM

{% asset_img "codeslam.png" "Code SLAM" %}

Visual-SLAM에서 keyframe 정보는 굉장히 많이 사용되는 정보 중 하나이다.
Visual-SLAM에서는 추출된 keyframe들로부터 3D reconstruction을 수행한다.
최근에는 Keyframe들로부터 depth map이나 semantic label map 등을 만들기도 한다.

Davison 교수님 랩실에서 최근 [CodeSLAM](https://arxiv.org/abs/1804.00874)과 [SceneCode](https://arxiv.org/abs/1903.06482), [DeepFactors](https://arxiv.org/abs/2001.05049)라는 연구를 발표되었다.
위의 사진은 SceneCode에서 가져왔다.

가장 초기 연구였던 CodeSLAM에서는 딥러닝 기법인 variational autoencoder를 사용해서 keyframe 이미지로부터 추출하는 depth map을 압축할 수 있다는 것을 보여주었다.
Variational autoencoder는 encoder 부분에서 차원을 축소하면서 컴팩트한 데이터 형태로 바꿀 수 있다.
Decoder를 사용해서 다시 복원시킬 수도 있지만, 이 연구에서는 '압축'에 집중하기에 decoder는 고려하지 않는다.
Variational autoencoder를 SLAM에 적용했을 때, keyframe에서 얻을 수 있는 depth map을 컴팩트한 데이터 형태인 'code'로 변환할 수 있게 된다.
이 code는 약 32개의 파라미터를 가지며, 이는 기존의 방식에 비해 굉장히 작다는 것을 알 수 있다.
또, 이 code의 파라미터를 조정함으로써 값을 변화시킬 수도 있다.

## SceneCode

후속 연구인 SceneCode에서는 depth map과 semantic map의 code를 이용해 joint optimisation을 수행하는 방법을 제안한다.
Depth map의 code화는 CodeSLAM에서 보여준 방식과 동일하다.
Semantic map의 code화는 새롭게 제안하는 방식이지만, CodeSLAM과 크게 차이가 나지는 않는 것 같다.
여러 keyframe들로터 depth map code와 semantic map code가 있을 때, 이 code 들의 값을 조정하면서 joint optmisation을 수행할 수 있다.
Code가 포함하는 파라미터의 수가 굉장히 작기 때문에, joint optimisation의 계산량도 많지 않다고 한다.

## DeepFactors

<br>

{% asset_img "deepfactors.png" "DeepFactors" %}

DeepFactors에서는 위 방식을 실제 Factor graph optmisation에 포함시켜 [실시간 데모](https://youtu.be/htnRuGKZmZw)를 만들었고, [오픈소스 프로젝트](https://github.com/jczarnowski/DeepFactors)로 공개하였다.
CodeSLAM에서는 하나의 방법론을 탐색하였다면, DeepFactors에서는 전체 SLAM 시스템에서 작동하는 수준으로 끌어올린 것이다.
DeepFactors에서는 camera motion부터 dense scene geometry까지 joint optmisation을 수행한다.

...여기까지 만들었지만, 정작 Davison 교수님은 이 방법에 대해 아직 의구심이 남아있으신 것 같다.
Depth map이나 3D scene에서 바뀔 수 있는 정보들을 고작 32개의 파라미터에 모두 담을 수 있을까?
한계가 있을 것이라고 보시는 것 같다.

---

# SLAM++, Hierarchical map (Live Maps, Dynamic scene graph), MoreFusion, Fusion++, NodeSLAM

SLAM에서 가장 효과적인 Map의 representation은 무엇일까?
Robotics, AR과 같은 분야에서는 objects를 다루는 경우가 많고, 각각의 objects에 정보를 저장해야한다.
Spatial AI에서 결국 우리가 가장 집중하게 되는 것은 'objects'이지 않을까 생각한다. 
하지만 현재의 파이프라인에서는 reconstructed map에서부터 objects를 추출해야하는데, 굉장히 비효율적이다.
"SLAM에서 사용하는 map을 object단위로 만들 수 있을까?"라는 질문이 생긴다.

## SLAM++

<br>

{% asset_img "slamplusplus.png" "SLAM++" %}

2013년에 [SLAM++](https://youtu.be/tmrAh1CqCRo)이라는 연구를 발표했다.
SLAM++은 뎁스카메라를 사용하면서 프론트엔드 단계에서 object recognition 기능을 추가하였다.
생성된 map은 object instance끼리 연결된 graph형태로 포현된다.
각각의 object instance는 3D 공간상의 위치+방향 정보와 해당하는 object class의 3D 모델 정보를 가지고 있었다.
이 방법의 단점이라면, 3D 공간 안에 어떤 object가 있어야하는지 미리 알고있어야한다는 것이다.

<br>

{% asset_img "map_hierarchy.png" "Hierarchical Map" %}

현재 활발하게 연구되고 있는 분야는 Hierarchical map 구조이다.
이 구조는 SLAM++에서 영감을 받았다고 할 수 있다.
Facebook Reality Labs의 [Live Maps](https://youtu.be/JTa8zn0RNVM)나 MIT의 Luca Carlone 교수님 랩실에서 나온 [3D Dynamic Scene Graph](https://youtu.be/JTa8zn0RNVM)
SLAM++은 object level만 다룬다고 하면, 이 방식들은 여러 level로 그래프를 구성한다는 것이다. 
SLAM++에서 수행하는 작업이 Hierarchical map 속 하나의 level이 될 수 있다는 것이다.

## MoreFusion

<br>

{% asset_img "morefusion.png" "MoreFusion" %}

[MoreFusion](https://youtu.be/6oLUhuZL4ko)은 CVPR 2020 학회에서 선보인 object-level SLAM과 6D pose estimation 기법이다.
MoreFusion은 뎁스카메라 이미지와 사전에 3D CAD 데이터가 있는 물체들을 기반으로 map을 만들려고 한다.
이 점이 SLAM++와 굉장히 유사하다.

<br>

{% asset_img "morefusion2.png" "MoreFusion2" %}

MoreFusion은 object가 겹쳐올려진 상태에서도 정확한 맵을 복원하여 로봇팔을 통해 물체들과 상호작용을 할 수 있다.
보통 Object가 겹쳐 올려진 상태에서는 서로가 서로를 가리기 때문에 6D pose를 정확하게 구할 수 없다.
MoreFusion은 volumetric reasoning를 (object끼리 서로 통과할 수 없는 constraint) 사용하여 정확하게 6D pose를 구할 수 있다.
하지만 SLAM++이 방 크기를 스캔할 수 있던데에 비해 MoreFusion은 훨씬 작은 테이블탑 크기에서만 사용할 수 있다. 

## Fusion++

<br>

{% asset_img "fusionplusplus.png" "Fusion++" %}

SLAM++이나 MoreFusion처럼 object에 대한 사전정보가 없으면 어떻게 할까?
[Fusion++](https://youtu.be/2luKNC03x4k)은 [Mask-RCNN](https://arxiv.org/abs/1703.06870)이라는 딥러닝 object intance segmentation 기술을 이용해, 그 자리에서 object를 검출하고 volumetric reconstruction을 수행하여 object-level SLAM을 수행한다.

## NodeSLAM

<br>

{% asset_img "nodeslam.png" "NodeSLAM" %}

[NodeSLAM](https://arxiv.org/abs/2004.04485)은 SLAM++와 Fusion++의 중간점이라고 볼 수 있다.
SLAM++은 검출하려는 object들의 모양을 알고 있어야하고, Fusion++은 아예 이러한 정보가 아예 없는 상황에서 만들어내는 과정이다.
NodeSLAM은 검출될 object들의 neural shape descriptor (i.e. differentiable shape feature)를 사전에 딥러닝 autoencoder로 학습해두고, 검출 단계에서 이 정보를 사용한다. 
보통 shape descriptor들은 정확한 shape을 읽어내려는 것이 아니라 object의 class를 구분하는데에 쓰인다.

<br>

{% asset_img "nodeslam_2.png" "NodeSLAM_2" %}

여기서 neural shape descriptor가 학습되는 과정은, 아까 설명되었던 CodeSLAM에서 했던 것 처럼 autoencoder의 encoder를 통해 압축된 code가 추출된다.
그리고 DeepFactors에서 이 code를 조정함으로써 camera pose와 map정보를 jointly optimise할 수 있던 것 처럼, NodeSLAM에서도 object shape와 camera pose를 jointly optimise할 수 있다.

<br>

{% asset_img "nodeslam_3.png" "NodeSLAM_3" %}

Object shape와 camera pose이 joint optimisation을 거쳐 얼마나 정확해지냐면...
MoreFusion이 3D CAD 모델을 가지고 할 수 있었던 똑같은 작업인 Robot manipulation을 할 수 있을 정도로 정확해진다.

---

# 하드웨어 개선 - Graphcore IPU

<br>

{% asset_img "processors.png" "Processors" %}

SLAM에서 쓸 수 있는 하드웨어는 여러가지 종류가 있다.
지난 PAMELA 프로젝트를 통해서 여러 하드웨어에 대해 SLAM 성능 벤치마크를 진행했었다.
다양한 SLAM 알고리즘을 단 하나의 칩으로 정확도, 속도, 효율성 모두 잡을 수는 없었다.
SLAM을 위한 하드웨어가 있어야한다는 생각도 들었다.
하지만 Sparse mapping, Dense mapping, CNN, Semantic mapping 등등 기술 트렌드가 너무나도 빨리 발전하기 때문에, 단 하나의 플랫폼에 엮이면 안된다는 생각도 들었다.
SLAM의 하드웨어 연구는 아직 계속 진행되어야한다.

<br>

{% asset_img "ipu.png" "Graphcore IPU" %}

요즘 눈여겨보고 있는 프로세서는 Graph processor이다.
영국의 Graphcore라는 업체에서 IPU라는 이름으로 판매하고있기도 하다.

IPU는 그래프 연산에 최적화된 설계를 가지고 있으며, AI 계산을 빠르게 수행하기 위해 개발되었다.
CPU는 보통의 연산에 효과적이지만, AI 계산에서는 좋지 않다.
GPU는 기존에는 그래픽스 렌더링을 위해 제작되어있지만, 현재는 AI 계산을 할 수 있도록 많이 기능이 추가되었다.
하지만 GPU 역시 비효율적인 부분이 있다.
IPU는 이 중 가장 효율적이게 작동할 수 있을 것이다.

<br>

{% asset_img "ipu_memory.png" "IPU Memory" %}

IPU 역시 GPU와 비슷하게 병렬처리에 특화되어있다.
GPU는 모든 코어가 동일한 작업을 할 때 효과적인 연산을 할 수 있다.
그에 비해 IPU는 각각의 코어가 다른 작업을 하면서도 효과적으로 작업할 수 있다.

IPU는 GPU에 비해서 메모리 공간이 GPU에 비해 훨씬 크다.
이 덕분에 프로세서 코어 당 더 많은 메모리를 사용할 수 있다.
또, 각 코어끼리 메모리에 저장된 내용을 공유할 수도 있다.
이를 통해, IPU는 각각의 코어에서 계산을 한 후 주변 코어에 결과 값을 넘겨주는 연산에 특화된 것을 알 수 있다.
딥러닝에서 사용하는 뉴럴넷 계산이 이러한 구조 덕에 훨씬 빠르게 계산될 수 있다.

<br>

{% asset_img "ipu_vis.png" "Visualization of IPU processes" %}

위 그림은 AlexNet이라는 CNN 아키텍처를 가진 네트워크를 사용할 때 필요한 연산을 IPU에 탑재했을 때, 각각의 코어에서 담당하는 작업을 시각화 한 것이다.
IPU의 특성 상 가까이 있는 코어들끼리 비슷한 연산을 함께 할 수 있는 것을 볼 수 있다.

<br>

{% asset_img "ipu_spatial_ai.png" "IPU for Spatial AI" %}

오늘 토크의 주제였던 Spatial AI를 달성하기 위해 IPU를 사용한다면, 프로세서의 구조가 어떻게 나눠질지 예상해본 그림이다.
위의 AlexNet 이미지와 어느정도 유사함을 보인다.

IPU가 Spatial AI에 특화될 수 있는 또 하나의 이유는 map을 표현하는 그래프 구조와 큰 연관이 있다.
Map 위에 있는 위치들이 가까울 수록 실제 IPU에서도 가깝게 위치시킬 수 있다.
멀리 있는 위치들끼리는 코어끼리의 거리 상 멀리 떨어트려 놓을 수 있다.

<br>

{% asset_img "ipu_graph.png" "Gaussian belief propagation within graph in IPU" %}

2019년 [FutureMapping2](https://arxiv.org/abs/1910.14139)라는 논문에서는 State estimation에서 쓰이는 Gaussian belief propagation을 Graph processor에서 수행하는 것을 제안하였다.
Gaussian belief propagation은 옛날에 사용되었던 방식이고, 현재 CPU에서 돌린다면 Gaussian belief propagation보다 많은 solver 라이브러리에 탑재된 Bundle adjustment가 훨씬 빠르게 작동한다.
하지만 CVPR 2020 학회에서 발표한 [Bundle adjustment on a Graph Processor](https://arxiv.org/abs/2003.03134) 논문에 따르면, Gaussian belief propagation 기반으로 bundle adjustment를 구현하여 IPU에 올렸을 때 CPU에서 돌린 bundle adjustment보다 20~40배는 더 빠르게 계산을 수행한 것을 확인할 수 있었다.

---

# 발표 질문들

## Spatial AI의 설계 그래프에는 여러 모듈이 있는데, 그중 특볋히 발전시키기 어려운 모듈이 있을까요?

딱 하나만 꼽기는 어렵고... 
Robustness 문제가 생기는 모듈들은 전부 어려운 것 같다.

예를 들어서, Semantic label을 만들어주는 모듈의 경우에는 하나의 데이터셋에서는 잘되지만 처음 보는 데이터셋에서는 굉장히 안될 수 있다.
이런 문제는 딥러닝 관점에서 보았을 때 unsupervised learning으로 풀어야하지 않을까 생각한다.
또는, 로봇이 처음 보는 환경에 있을 때, 그 자리에서 학습을 할 수 있어야한다고 생각한다.

## IPU 칩이 Long-term SLAM을 할 수 있을까요?

IPU 칩에서는 factor graph를 통해서 in-place computation을 하면 좋다. 
하지만 물론 당연히 map이 커지고, 더 많은 level의 hierarchical map을 만든다면, 칩이 감당할 수 없을 수 있다.
이런 경우에는 incremental abstraction을 써야한다.
예를 들어, 현재 보고 있는 이미지에서 포인트가 100개가 뽑혔다고 해보자.
그런데, 약 5초정도 후 보았을 때, 그 포인트가 알고보니까 어떤 하나의 plane에서 나왔다고 해보자.
이런 경우에는 포인트 100개가 아니라 plane을 저장하면 훨씬 메모리를 적게 사용할 수 있을 것이다.
이런식으로 세밀한 데이터 node들이 모여서 super-node를 만들어가고... 이 과정이 반복될 수 있지 않을까 생각한다.





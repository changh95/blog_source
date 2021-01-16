---
title: Deep Visual SLAM Frontends - SuperPoint, SuperGlue and SuperMaps (Tomasz Malisiewicz 발표)
date: 2020-12-27 20:38:19
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- Deep Learning
- Deep SLAM
- CV
- Tomasz Malisiewicz
- AR
- Magic Leap
- SuperPoint
- DeepChArUco
- SuperPointVO
- SuperGlue
- SuperMaps
categories: 
- [SLAM, 학회]
excerpt: CVPR 2020 학회에서 Joint Workshop on Long-Term Visual Localization, Visual Odometry and Geometric and Learning-based SLAM 워크샵 중 Tomasz Malisiewicz께서 발표해주신 Deep Visual SLAM Frontends - SuperPoint, SuperGlue and SuperMaps의 발표 노트입니다. Magic Leap의 최근 연구에 대해 설명합니다.
---

# 시작하기 전...

Tomasz Malisiewicz는 토마스 말리세비치라고 읽는다.

이번 토크는 3가지 기술을 커버한다.
1. SuperPoint
   - Magic Leap에서 딥러닝을 SLAM에 적용하기 위한 시도라고 소개한다.
2. SuperGlue
   - Graph neural network와 Attention을 이용하여 feature matching의 성능을 높인 기술이다.
3. SuperMaps
   - 아직 완성되지는 않았지만, SLAM에서 pairwise matching을 제거하고 완전한 end-to-end 방식으로 넘어가기 위한 로드맵을 소개한다.

---

# SuperPoint - SIFT를 딥러닝으로 대체하기

(SIFT는 SLAM에서 잘 안쓰지 않나...? ORB가 더 말이 될 것 같지만, Tomasz 본인이 SIFT를 대체하는 기술이라고 적었다.)

## SLAM Frontend란?

<br>

{% asset_img "slam.png" "SLAM in 2 parts" %}

Visual SLAM은 두가지 단계로 나눠진다.
Frontend는 keypoint extraction과 keypoint matching 작업을 수행한다.
Backend는 bundle adjustment와 같은 nonlinear least squares optimisation 작업을 수행하며 카메라 위치와 주변 환경 구조를 최적화한다.

Frontend는 이미지만을 다루기 때문에, 최근 classification, detection, segmentation 등 2D 이미지를 다루는 딥러닝 CNN 기술을 frontend에도 사용해볼 수 있다.

## SuperPoint 특징

<br>

{% asset_img "superpoint_simple.png" "SuperPoint in a nutshell" %}

[SuperPoint](https://arxiv.org/abs/1712.07629)는 CVPR 2018 학회에서 발표된 연구이다. 
SuperPoint를 간단하게 설명하면, 이미지 속 2D keypoint의 위치와 해당 feature의 descriptor 정보를 내뱉는 CNN 네트워크이다.

SuperPoint는 Keypoint의 위치와 descriptor 정보를 jointly 하게 계산할 수 있다.
이는 기존의 keypoint descriptor 알고리즘들이 keypoint 주변의 patch 정보를 사용했던 점과 확연히 다른 점이라고 볼 수 있다.

논문에서는 [VGG](https://arxiv.org/pdf/1409.1556.pdf) 네트워크를 backbone으로 사용하였다.
하지만 다른 backbone을 쓰지 못할 이유는 없다.

SuperPoint는 GPU에서 실시간으로 돌 수 있게 설계되었다고 한다.
이는 Keypoint location과 descriptor 계산이 ~90%의 weight 정보를 공유할 수 있게 만듬으로써 가능했다고 한다.
또, backbone 네트워크가 크지 않아서 가능했다고 한다.

이렇게 나온 SuperPoint를 가지고 Backend에 그대로 넣어 SLAM을 할 수 있다.

<br>

{% asset_img "decoder.png" "Keypoint decoder" %}

SuperPoint는 어떻게 Keypoint를 추출하는걸까?
SuperPoint의 코어 컴포넌트 중 하나인 keypoint decoder는, 어떠한 픽셀이 keypoint인지 아닌지 classification 문제를 푼다고 한다.
SuperPoint는 우선 8x8 크기의 패치 안에는 무조건 하나의 keypoint만 있을 수 있다고 가정한다.
8x8 크기의 패치 안에는 keypoint가 될 수 있는 64개의 후보 픽셀들이 있다.
여기에 1개의 'dustbin'이라는 후보를 추가한다.
SuperPoint의 목적은, 이 65개의 후보들에 대한 distribution을 생성하는 것이다.
Distribution을 쓰면 이 모든 후보들에 대한 heatmap을 얻을 수 있다.

## SuperPoint 학습 과정

<br>

{% asset_img "training.png" "Training" %}

SuperPoint는 다수의 이미지 페어들에서 좋은 keypoint matching이 되는 것을 목적으로 만들었다.
즉, matching이 잘 되는 keypoint만을 추출하는 것이 중요하다.
우선 Keypoint matching을 하기 위해서는 2D location을 추출해주는 'keypoint detector'와 keypoint의 매칭을 위한 정보를 추출해주는 'keypoint descriptor'가 필요하다.
여기서, 같은 keypoint를 다른 각도에서 바라보아도 비슷한 keypoint descriptor가 뽑히며, 다른 keypoint의 descriptor와는 차이가 나타나야한다.
이러한 점을 학습하기 위해, SuperPoint의 학습과정에서는 ground truth 데이터로 keypoint matching이 잘 된 이미지 페어 (i.e. 2장)이 필요했다.
또, 2장의 이미지로부터 효과적으로 학습하기 위해 [Siamese training](https://youtu.be/6jfw8MuKwpI) 기법으로 학습되었다.

2장의 이미지는 다음과 같이 생성되었다.
원본 이미지를 [homography warp](https://www.learnopencv.com/homography-examples-using-opencv-python-c/)를 통해 또 다른 이미지를 생성한다.
Homography warp를 하면, 원본 이미지에서의 픽셀 위치를 새롭게 생성된 warp 이미지에서도 찾을 수 있다.
즉, 원본 이미지에서의 keypoint 위치를 다른 이미지에서도 똑같이 찾을 수 있다는 것이다.

Keypoint detector는 keypoint label을 supervised learning으로 학습하였다.

그리고, keypoint detector로 찾은 keypoint location에서 descriptor를 추출하여 비교한 후 descriptor 학습을 수행하였다.
Keypoint descriptor 학습은 최근 많이 사용되는 [contrastive loss](https://youtu.be/yIdtx3pQkdg)를 이용한 [metric learning](https://youtu.be/M0EjrFQH49o) 기법을 사용하였다.

<br>

{% asset_img "natural_image.png" "Natural images" %}

아까 위에서 Keypoint detector는 keypoint label을 기반으로 학습을 진행했다고 하였다.
그러면 이 label은 어디서 오는 것일까?
가장 생각하기 쉬운 방법은 SIFT, SURF, ORB 등의 방식에서 뽑아서 label 데이터를 만드는 것이다.
하지만 이 방식으로 트레이닝을 하면, SuperPoint의 최대성능이 SIFT, SURF 정도밖에 되지 않기 때문에, 사용할 수 없다.
그러면 다른 방법으로 keypoint label을 만들어야한다.
일단, 사람이 직접 label을 추가할 수는 없다.

이러한 문제를 해결하기 위해, [Self-supervised learning](https://hoya012.github.io/blog/Self-Supervised-Learning-Overview/) 방식을 도입하였다.

<br>

{% asset_img "self_supervised_training.png" "Render" %}

존경하는 hoya012 블로그에서는 Self-supervised learning을 다음과 같이 설명하였다.
"Network로 하여금 만든 pretext task를 학습하게 하여 데이터 자체에 대한 이해를 높일 수 있게 하고, 이렇게 network를 pretraining 시킨 뒤 downstream task로 transfer learning을 하는 접근 방법"

SuperPoint에서는 pretext task로써 네트워크가 간단한 환경에서 keypoint를 찾는 기능을 학습하게 하였다.
이 과정은 2017년 발표된 SuperPoint 이전의 논문인 [Towards Geometric Deep SLAM](https://arxiv.org/abs/1707.07410) 논문에서 소개하는 MagicPoint에 해당한다.
가상 렌더링을 사용하여 이미지 데이터와 렌더링된 모델의 vertex 위치 데이터를 기반으로 네트워크를 학습하였다.
Robustness를 높이기 위해  가상 이미지 데이터는 다양한 조명과 노이즈도 첨가되어있다.
이를 통해 네트워크는 keypoint의 특성을 학습하게 되었다.

가상 이미지로 학습한 결과를 Real-world 데이터셋으로 옮기기 위해 (transfer), [MS-COCO 데이터셋](https://cocodataset.org/#home)에 다시 학습을 하였다.
MS-COCO 데이터셋은 keypoint에 대한 label이 없다.
이 데이터셋에서 keypoint label을 만들기 위해 homographic adaptation이라는 기법을 사용했다.
Homographic adaptation 기법에는 추후에 설명한다.

<br>

{% asset_img "magicpoint.png" "MagicPoint Performance" %}

지금까지 소개한 부분인 MagicPoint는 기존의 FAST, Harris, Shi-Tomasi keypoint detector에 비해 월등히 좋은 성능을 가진다.
특히, 노이즈가 있는 상황에서 더 잘 작동하는 것을 볼 수 있다.
이는 MagicPoint가 noise가 있는 상황에서도 잘 작동할 수 있게 학습되었기 때문이다.

<br>

{% asset_img "homographic_adaptation.png" "Homographic adaptation" %}

아까 부족했던 Homographic adaptation에 대해 설명하겠다.
Homographic adaptation의 목적은 [homography 기법](https://docs.opencv.org/master/d9/dab/tutorial_homography.html)을 이용해 planar camera motion을 만들어내는 것이다.
두 이미지 사이의 camera motion을 알 수 있다면, 하나의 이미지 위의 keypoint 위치들을 다른 이미지 위에 그대로 옮길 수 있다.
즉, self-labelling 기술이 된다는 것이다.
Homograpic adaptation을 통한 self-labelling을 통해 우리는 두가지 효과를 기대할 수 있다.
1. 잘못된 keypoint detection을 피할 수 있다
2. Keypoint detection의 repeatability (i.e. 다른 각도에서 보아도 동일한 위치에서 keypoint가 검출되는 능력)을 향상시킬 수 있다.

Label이 없는 원본 이미지에 homography를 적용하여 여러 각도에서 바라보는 이미지들을 생성한다.
그 후, 각각의 이미지에 MagicPoint를 적용하여 keypoint를 추출한다.
이 keypoint들을 homography 변환으로 원본 이미지에 다시 옮겨주면, keypoint들의 'superset'이 생기는 것이다.
(SuperPoint의 'super'가 여기서 왔다고 한다).

## SuperPoint 성능 비교

<br>

{% asset_img "superpoint_eval.png" "Evaluation of SuperPoint" %}

SuperPoint, [LIFT](https://arxiv.org/abs/1603.09114), SIFT, ORB를 비교한 결과이다.
LIFT는 또 다른 딥러닝 기반 keypoint detector + descriptor 이다.
SuperPoint가 매칭된 keypoint의 수도 가장 높고, keypoint들의 위치도 이미지에 고루고루 퍼져있다.
그에 비해, Visual-SLAM에서 많이 사용되는 ORB feature는 좋지 않은 성능을 보여준다.
특히, 대부분의 keypoint가 한곳에 밀집되어있고, 그 keypoint끼리 생김새가 비슷할 때 매칭의 성능이 많이 떨어진다.
이런 경우에는 camera motion이 정확하게 계산될 수 없다.

<br>

{% asset_img "superpoint_eval2.png" "Evaluation of SuperPoint 2 " %}

두번째 비교이다.
위에서 설명한 ORB의 단점이 잘 보이는 또 다른 예시이다.

<br>

{% asset_img "superpoint_eval3.png" "Evaluation of SuperPoint 3 " %}

세번째 비교이다.
LIFT에 비해 회전에 좀 더 강인한 모습을 보인다.
하지만 SuperPoint도 완전하게 rotation-invariant하지 않다는 점을 인지해야한다.
Backbone으로 사용한 VGG 네트워크는 rotation-invariant하지 않기 때문이다.
그렇기 때문에 SuperPoint도 최대 15~30도 정도의 회전만을 고려했다고 한다.

<br>

{% asset_img "3d.png" "3D Performance" %}

SuperPoint의 학습과정는 전부 2D 이미지에서 학습한 것을 볼 수 있다.
그러면 3D에서 잘 작동할지 궁금해진다.
결과만 이야기하면, 잘 작동한다.

SuperPoint를 직접 사용해보고 싶으면 이 [Github 링크](https://github.com/magicleap/SuperPointPretrainedNetwork)를 참조하면 좋다.
PyTorch 기반으로 작동한다고 한다.

---

# DeepChArUco - SuperPoint의 성능을 보여주는 다른 예시

<br>

{% asset_img "deepcharuco.png" "DeepChArUco" %}

SuperPoint의 robustness를 보여주는 프로젝트를 하나 소개하려고 한다.

ChArUco라는 패턴이 있다.
이 패턴은 체스보드 패턴에 [ArUco](https://www.uco.es/investiga/grupos/ava/node/26) 마커를 임베딩한 패턴이다.
ArUco 마커들은 각각의 ID를 가지고 있으며, 이 ID를 기반으로 fiducial marker detection을 할 수 있다.
또, 체스보드와 ArUco 마커 코너 점들을 이용하여 2D-3D correspondence matching을 통해 pose estimation도 수행할 수 있다.

DeepChArUco 연구에서는 기존의 Corner detector와 ArUco 마커 ID detection 알고리즘을 새롭게 변형하였다.
Corner detector는 SuperPoint로 대체되었다.
그리고 ArUco 마커 ID detection도 딥러닝 기반 classifier로 변형되었다.

<br>

{% asset_img "deepcharuco_dark.png" "DeepChArUco in dark environment" %}

DeepChArUco 방식은 사람도 구분할 수 없는 어두운 상황에서도 정확하게 마커를 찾을 수 있다.
[데모 영상](https://youtu.be/Smorg9dffc0)을 보면, 어두워도, 그림자가 져도 찾을 수 있다.

딥러닝 기반 Local feature detection의 성능이 뛰어나다는 것을 보여주는 좋은 예시라고 볼 수 있다.

<br>

---

# SuperPointVO - SuperPoint로 Visual Odometry를 할 수 있을까?

이 연구는 2018년 발표된 [Self-improving visual odometry](https://arxiv.org/abs/1812.03245) 연구를 소개한다.

위에서 보여준 것 처럼, SuperPoint로 얻은 keypoint는 3D 환경에서도 트랙킹이 잘 되는 것을 볼 수 있다.
그리고 실제로 optmisation 프로그래밍을 추가하여 visual odometry 시스템을 만들었을 때도 성공적으로 3D reconstruction을 수행할 수 있었다.
SuperPoint가 visual odometry에서도 쓰일 수 있다는 점이 입증되었으나, visual odometry를 위해 최적화되어있진 않다.

이번 연구는 visual odometry를 수행하여 나오는 데이터를 기반으로 네트워크를 학습한다.
이로써 오랜 시간이 지나도 keypoint correspondence를 만들 수 있게 네트워크를 학습할 수 있다.
또, 어떤 keypoint가 오랜 시간이 지나도 stable하게 매칭이 되는지 학습할 수 있다.

<br>

{% asset_img "superpointvo.png" "SuperPointVO" %}

학습 과정은 다음과 같다.
우선 Monocular 이미지 시퀀스들로부터 SuperPoint를 추출하고, correspondence를 만든다.
그 후, [SfM (Structure-from-Motion)](https://youtu.be/i7ierVkXYa8)을 이용하여 3D reconstruction을 수행한다.
(어차피 학습은 비실시간에서 수행하기 때문에 SfM이라는 단어를 사용하였다. 이론상 VO backend나 SfM이나 크게 다르지 않다)

<br>

{% asset_img "reproj_error.png" "Stable / Unstable points" %}

이렇게 생긴 3D map을 각각의 이미지에 재투영하고 (reprojection), [reprojection error](https://stackoverflow.com/questions/29095349/how-is-the-reprojection-error-calculated-in-matlabs-triangulate-function-sadly)를 계산한다.
이 때 3d map point의 평균 reprojection error가 1 픽셀보다 낮다면, 어떤 방향에서 보던지 3D map을 정확하게 만들어내는 keypoint 가 되겠다.
반대로, 평균 reprojection error가 5 픽셀보다 높으면, 3D map을 정확하게 만들지 못하는 keypoint가 되겠다.
중간에 어중간한 keypoint들은 무시한다.

이 정보들을 기반으로 CNN 네트워크를 학습한다.

<br>

{% asset_img "superpointvo_training.png" "Training SuperPointVO" %}

SuperPointVO의 학습과정은 SuperPoint를 학습하는 과정과 유사하다.
Siamese training 구조로 keypoint loss와 descriptor loss를 구한다.

<br>

## 결과

{% asset_img "superpointvo_result1.png" "SuperPointVO result 1" %}

1초 정도의 프레임 간격에는, SuperPointVO의 성능이 크게 부각되지 않는다.
이는, 바라보고 있는 방향이 비슷할 때는 왠만한 keypoint detector는 좋은 성능을 보인다는 것이다.

<br>

{% asset_img "superpointvo_result2.png" "SuperPointVO result 2" %}

하지만 3초정도 프레임 간격에는 카메라 위치와 방향에 큰 차이가 나게 된다 (i.e. wide baseline이 생긴다).
이런 경우에 SuperPointVO의 성능이 부각된다.

<br>

---

# SuperGlue - Graph neural network + Self-attention 기반 keypoint 매칭

<br>

{% asset_img "superglue.png" "SuperGlue" %}

SuperGlue는 Graph Neural Network에 Optimal transport procedure를 추가한 알고리즘이다.

SuperGlue의 목적은 wide baseline에서 SuperPoint를 매칭하는 것이며, motion model을 사용하지 않고도 기존의 motion-guided matching 기법보다 더 좋은 성능을 내는 것이다.
이는, SLAM을 진행하면서 종종 잘못된 motion model을 사용했을 때 feature matching의 성능이 급격하게 감소하는 것을 방지하기 위함이다.

SuperGlue는 GPU에서 실시간으로 돌아가면서 State-of-the-Art 성능을 보여준다.
SuperGlue는 SuperPoint와 쓸 수도 있고, SIFT와 쓸 수도 있다.

<br>

{% asset_img "superglue_architecture.png" "SuperGlue Architecture" %}

(원래 영상에서는 깊게 설명하지 않지만, 다른 SuperGlue 영상에서 내용을 가져와서 첨부하였습니다)

SuperGlue는 2가지 단계로 나눠져있다.
첫번째는 Attention을 이용한 graph neural network이다.
이 부분은 Keypoint들의 location 정보와 descriptor 정보를 attention 메커니즘을 사용해서 contextual cue에 대한 정보를 학습한다.
두번째는 기존의 (i.e. 딥러닝을 쓰지 않는) 최적화 프로그래밍을 수행하며, 사용한 알고리즘은 sinkhorn 알고리즘이다.
Sinkhorn 알고리즘을 통해 differentaible solver를 만듬으로써, domain knowledge에 대한 constraint를 만든다.

## Keypoint encoder

<br>

{% asset_img "superglue_front.png" "SuperGlue Architecture - creating representation" %}

SuperGlue의 가장 앞단에서는, 매칭을 수행할 이미지 2장으로부터 추출된 keypoint들을 모두 attention 메커니즘에 input 데이터로 사용한다.
이런 방법은 기존의 매칭 방법과 큰 차이를 보인다.
기존의 매칭 방법은, 각각의 이미지에서 따로 descriptor 등의 계산을 거친 후 [nearest neighbour 방식](https://github.com/mariusmuja/flann) 등으로 매칭을 수행하였다. 
하지만 비슷한 local feature descriptor가 많이 있을 때, 매칭 candidate가 너무 많기 때문에 이런 경우에는 매칭에 실패하기도 한다.

SuperPoint를 사용했을 경우를 가정해보고 설명한다.
빨간색이 1번 이미지에서 나온 정보, 파란색이 2번 이미지에서 나온 정보이다.
두 이미지에 각각 SuperPoint를 검출한 후 keypoint location 정보를 keypoint encoder 네트워크로 보내 섞는다.
이 encoded keypoint location 정보는 local visual appearance 정보와 (i.e. visual descriptor) multi-layer perceptron 레이어을 거치며 합쳐진다.
이 결과 keypoint representation이 만들어진다.

## Attentional aggregation (Attentional Graph Neural Network update)

<br>

{% asset_img "superglue_graph.png" "SuperGlue Architecture - Attentional aggregation" %}

Keypoint representation은 Attentional graph neural network 구조 속 다른 keypoint 정보들로부터 iterative하게 업데이트 된다.

업데이트가 되는 방식이 두개가 있는데, self-edge 업데이트와 cross-edge 업데이트가 있다.
하나의 keypoint 정보가 동일한 이미지의 다른 keypoint 정보로부터 업데이트가 되는 것이 self-edge update이다.
그리고 keypoint 정보가 다른 이미지의 다른 keypoint 정보로부터 업데이트가 되는 것이 cross-edge udpate이다.
이 keypoint 정보들을 node로 표현하고 update 관계를 edge로 표현하면, self-edge와 cross-edge로 이뤄진 하나의 graph 형태가 나타난다.
이 업데이트 과정은 graph neural network의 한 종류인 Message passing neural network를 사용하며, 각각의 message는 self-attention 또는 cross-attention을 이용하여 계산된다.
이 Self-attention은 [Transformer](https://arxiv.org/abs/1706.03762)에서 영감을 받았다.

Self/cross-attention을 통한 message 계산은 database retrieval과 비슷하다고 한다.
Query 데이터 $q_i$와 비슷한 Key 데이터 $k_j$를 찾고, 해당하는 value 데이터 $v_j$를 찾는다고 한다.
이 때, 평균 value 값에 query~key 데이터의 similarity 값을 weight로 곱해준 값을 message 값으로 사용한다고 한다.
이 때문에, 이 업데이트 과정을 attentional aggregation이라고도 한다.

<br>

{% asset_img "attentional_aggregation.png" "Query to Key and Values" %}

위 과정을 조금 더 쉽게 설명하겠다.
$x_i$ keypoint 정보로 Query를 했을 때, Key 값으로 여러 keypoint candidate가 나온다.
이 $x_i$ 값은 keypoint location과 visual appearance 정보를 encode 하고 있기 때문에, $k_j$들로 spatial neighbour, self-similarity, salient keypoint를 매칭할 수 있다.
 
<br>

{% asset_img "self_cross.png" "Self/Cross attention" %}

Self-attention으로는 동일한 이미지에서 다른 keypoint들로부터 self-similarity 정보를 효과적으로 얻어낼 수 있다.
Cross-attention은 이미지 간 candidate match로 찾을 수 있다.

## Optimal matching

<br>

{% asset_img "superglue_optimal_matching.png" "Optimal matching layer" %}

Attentional aggregation이 끝나면 최종 matching descriptor 정보가 각각의 이미지로부터 나온다.
Matching descriptor 정보를 dot product하면, 두 keypoint descriptor 정보간의 밀접도를 (i.e. affinity) 표현하는 score matrix가 생긴다.
종종 occlusion의 이유로 매칭이 될 수 없는 keypoint가 있는 것을 대비하여, score matrix에 dustbin score를 추가시켰다.

[Sinkhorn 알고리즘](https://michielstock.github.io/OptimalTransport/)을 사용하여 partial assignment 최적화 문제를 GPU에서 잘 돌 수 있도록 바꿔주었다.

SuperGlue의 네트워크 최앞단부터 partial assignment matrix까지 전부 ground truth correspondence 데이터를 통해 end-to-end 트레이닝이 가능하다.

<br>

{% asset_img "superglue_eval.png" "Evaluation of SuperGlue" %}

기존의 Nearest neighbour 매칭 방식에 비교했을 때, SuperGlue가 훨씬 더 많은 매칭 결과를 보여준다.
특히나, 바닥에서 반복되는 패턴이 있음에도 굉장히 매칭이 잘된다.

<br>

{% asset_img "superglue_eval2.png" "Evaluation of SuperGlue 2" %}

MagicLeap 팀이 지금까지 테스트 해본 결과 SuperPoint + SuperGlue 조합이 제일 잘 되는 것을 확인하였다.

SuperGlue는 [오픈소스](https://github.com/magicleap/SuperGluePretrainedNetwork)로 공개되어있다.
GPU가 달린 노트북/데스크탑으로 VGA 이미지로부터 약 512개의 keypoint를 추출했을 때, 15FPS 정도로 작동한다고 한다.

Tomasz의 조언에 따르면, 데모가 굉장히 잘된다고 한다.
논문을 읽고나서 이 기술을 쓸지 말지 결정하는데는 몇시간이 걸릴 수도 있지만, 그냥 웹캠 하나 꽂고 돌려보면 바로 써봐야겠다는 결정을 할거라고 한다.
본인도 이 데모의 failure case를 만들어보기 위해 카메라 포커스도 바꿔보고, occlusion도 만들어봤지만 그냥 너무 잘된다고 한다.

실제로 CVPR 2020 학회에서 진행된 3개의 대회 (Image matching challenge, Local features for visual localization, Visual localization for handheld devices) 대회에서 모두 우승했다고 한다.

<br>

---

# SuperMaps - SuperPoint + SuperGlue의 미래...?

SuperPoint + SuperGlue의 다음은 뭘까?

SuperPoint + SuperGlue 콤보는 이미지 pair를 사용할 수 있다.
SuperMaps는 여러장의 이미지를 쓸 수 있지 않을까?
여러장의 이미지는 보통 3D Map을 의미하기도 한다.
3D Map 끼리 섞는것도 가능하지 않을까?

SuperPoint + SuperGlue 콤보는 keypoint matching만 해주고 정작 제일 중요한 camera pose estimation은 하지 않는다.
SuerperMaps는 camera pose estimation도 할 수 있지 않을까?

SuperPoint + SuperGlue 콤보는 loop closure 기능을 내포하지 않는다.
그리고 SuperGlue의 계산량은 꽤 높은 편이다.
SuperGlue를 사용하기 어렵기 때문에, loop closure를 구현하기도 쉽지는 않다.
SuperMaps는 loop closure를 위한 keyframe embedding 기능이 있어야한다.
SuperPoint들을 (i.e. local keypoints) 모아서 하나의 global descriptor를 만드는 것도 하나의 방법이 될 것 같다.

SuperPoint와 SuperGlue는 각각 트레이닝되야한다.
SuperMaps는 가능하면 한번에 end-to-end 트레이닝이 되면 좋을 것 같다.
필수는 아니다.

SuperPoint는 CNN 기반이고, SuperGlue는 GNN 기반이다.
그렇기 때문에 두 네트워크의 receptive field 개념이 굉장히 다르다.
SuperMaps는 공통된 receptive field가 있으면 좋을 것 같다.

---

# 딥러닝 SLAM에서 풀어야 할 숙제

- Multi-user SLAM
  - 다수의 agent를 통해서 representation/map을 만드는 방법이 있어야한다.
- Object recognition이 SLAM frontend로 들어가야한다
  - Semantics 정보를 다룰 수 있어야한다
- Life-long SLAM
  - 시간이 지날수록 map도 함께 바뀌어가며 진화해야한다


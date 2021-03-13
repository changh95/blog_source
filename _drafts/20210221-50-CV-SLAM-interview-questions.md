---
title: 50가지 CV/SLAM 기술면접 질문들 (정리중)
date: 2021-02-21 20:38:19
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- Interview
- Questions
categories: 
- [SLAM, 50가지 CV/SLAM 기술면접 질문들]
excerpt: 나름대로 정리해본 50가지 CV/SLAM 기술면접 질문들입니다.
---

### 1. "Image projection 과정에 대해서 최대한 자세하게 설명해주세요"

- 답
    {% asset_img "projection1.png" "projection" %}

    3D 공간에서 2D 이미지로 변환하는 과정을 image projection이라고 합니다. 3D 공간 위의 어떤 위치를 $[X,Y,Z,1]^T$로 표현하고, 2D 이미지 위의 픽셀 위치를 $[u,v,1]^T$ 로 표현하겠습니다. $X,Y,Z$는 3D coordinate고, $u,v$는 픽셀 coordinate 위의 column, row를 의미하며, 1은 euclidean space가 아닌 projective space 에서의 변환을 고려하기 위한 homogeneous coordinate 표기법 때문에 붙었습니다. 

    카메라가 이 3D 포인트를 바라보고 있을 때, 카메라의 optical centre와 3D 포인트 사이의 직선을 그리면 image plane 을 통과할 것입니다. 이 때, 카메라의 optical centre에 위치한 카메라의 coordinate frame과 World의 coordinate frame의 회전과 이동 (rotation / translation)을 Euclidean space에서의 $SE(3)$로 표현하면 3x4 매트릭스 형태를 가집니다. 이 매트릭스를 Extrinsic matrix라고 부르며, $[X,Y,Z,1]^T$ 에 이 매트릭스를 곱하면, 카메라의 coordinate frame에서 $[X,Y,Z,1]^T$ 의 위치를 알 수 있습니다. 

    이후, Intrinsic matrix를 이용해서 $[X,Y,Z,1]^T$를 2차원 평면에 투영시킬 수 있습니다. Intrinsic matrix는 Focal Length (fx, fy)와 Principal point (cx, cy) 정보를 가지고 있습니다. Focal length는 normalized scale에서 pixel scale로 변환을 할 수 있게 해줍니다. cx와 cy는 보통 중앙 픽셀의 위치 값을 가지는데, 이는 coordinate frame을 optical centre (또는 이미지 중앙)이 아닌, 좌측 상단에 올려놓기 위함입니다. 하지만 카메라가 실제로 만들어지면서 완벽하게 중앙에 위치하지 않기 때문에, 몇픽셀의 차이는 있을 수 있습니다. Intrinsic matrix의 3번 row는 $[0, 0, 1]$인데, 이는 homogeneous coordinate를 맞추려는게 아니고, 3차원→2차원 축소를 위함입니다.

    이 계산이 끝나면 $[su, sv, s]^T$이 결과 값으로 나오는데, $s$는 homogeneous coordinates의 영향으로 나온 scale 값입니다. 그러므로 $s * [u, v, 1]$로 normalize를 시켜주면 pixel coordinate 값이 나오게 됩니다.

    - 추가 이미지

        {% asset_img "projection2.png" "projection" %}

### 2. Monocular SLAM의 장점과 단점은 무엇인가요?

- 답
    - 장점
        - 카메라를 한대만 쓰기 때문에 소비전력과 구성 비용이 싸다.
        - 다중 카메라 시스템에 비해 처리해야하는 이미지 양이 적다.
        - 수많은 제품이 1개의 카메라를 사용한다 (e.g. 스마트폰... 요즘엔 좀 더 많아지긴 하지만)
        - 다중 카메라에서 필요한 캘리브레이션 과정이 축약된다. Intrinsic 캘리브레이션 한번만 하면 된다.

    - 단점
        - 알고 있는 geometry를 사용하지 않는 이상, Epipolar geometry 계산 시 Scale을 구할 수 없다. 이 경우 metric scale 복원이 불가능하다.
        - Monocular image에서 depth 추정이 불가능하다. 상대적 거리를 알기 위해서는 무조건 multiple view reconstruction이 필요하다.
        - 360 카메라나 어안렌즈와 같은 특수한 렌즈 구성을 사용하지 않는 이상 field of view가 작다.

    - 단점 타파 방법
        - Scale과 depth 를 구하기 위해서는 스테레오 카메라 또는 다중 카메라 시스템을 구축하면 된다.
        - Scale을 구하기 위해 IMU를 함께 사용하는 방법이 있다. IMU는 metric scale 복원이 가능하다.
        - 마커와 같이 이미 알고있는 랜드마크 물체를 사용해 scale 추정을 할 수 있다.
        - 최근 딥러닝 기반 monocular depth estimation, 또는 single image monocular depth estimation 기술을 사용해 depth 정보를 추출하기도 한다. 아직 엄청나게 정확한 편은 아니지만, CNN-SLAM의 경우 depth estimation 프론트엔드에 LSD-SLAM 백엔드를 사용해서 SLAM 시스템을 만들기도 했다.

### 3. LiDAR, RADAR, 카메라의 장단점을 이야기하세요. 이들을 함께 쓰려면 어떻게 해야할까요?

- 답
    - LiDAR:
      - 가격이 (아직) 비싼편
      - 레이저 반사의 TOF를 이용해서 카메라보다 훨씬 먼 거리임에도 정확한 뎁스 추정이 가능. 하지만 눈이나 비, 안개가 낀 상황에 성능이 급격하게 떨어짐.
      - LiDAR SLAM이 가능
      - 딥러닝 기술을 이용하여 object detection도 가능

    - RADAR:
      - Doppler 효과를 이용해서 움직이는 장애물의 속도 추정 가능.
      - 데이터에 잡음이 엄청 많음
      - 안 좋은 날씨에는 라이다나 카메라보다 잘 됨

    - 카메라:
      - 가격이 싼 편
      - 딥러닝 기반 detection 등을 이용해서 장애물을 구분한다던지, 차선을 본다던지, 신호등 신호를 구분할 수 있음.
      - 다중 카메라로 꾸밀 경우 depth 정보 추출 가능하나, 단안 카메라로는 불가능함. 다중 카메라를 사용해도 라이다보다 훨씬 거리가 작음.
      - 어둡거나, 눈/비/안개 등으로 인해 보이지 않을 경우 성능이 급격하게 떨어짐.

    안전을 위해서는 모두 다 쓸 것 같음. GPS와 IMU도 쓸 것 같음. 카메라가 60Hz, 라이다가 10Hz, 레이더가 100Hz, IMU가 200Hz, GPS가 5Hz라서 속도가 다 다를텐데, Extended Kalman Filter 등을 사용해서 퓨전을 할 것 같음. 물론 센서들의 특성을 분석하고 퓨전을 할 듯. 

### 4. RANSAC에 대해 설명해주세요. RANSAC의 장점과 단점도 설명해주세요.

- 답

    RANSAC은 Fischer와 Balls의 예엣날 논문으로써 Random sample consensus의 약자이다. 모델 추정을 할 때 outlier가 끼어있으면 정확한 모델 추정이 불가능한데, RANSAC을 통해 수많은 데이터로부터 outlier를 제거하고 (확률적으로) 올바른 모델을 찾을 수 있다.

    RANSAC이 진행되는 방법은 다음과 같다. 우선 모델을 구성할 수 있는 minimal set의 데이터를 무작위로 뽑는다. 예를 들어, homography의 경우 4개의 feature match를, p3p인 경우 3개를 고르는 것과 같다. 뽑은 데이터로부터 모델을 만들고, 이 모델을 사용할 때 다른 데이터들이 얼마나 에러를 가지는지를 구한다. 이 때, 이 에러가 지금까지 찾은 모델들보다 낮다면 (i.e. best model이라면), 그 정보를 저장한다 (물론 첫 iter에서는 항상 best model이다). 그 다음에 또 다시 minimal set의 데이터를 무작위로 뽑고, 그 모델이 best model인 경우 모델 정보를 저장, 그게 아니라면 정보를 버린다. 이렇게해서 정해놓은 iteration, 또는 정해놓은 error threshold에 도달할 때 까지 돌리는데, 랜덤한 성격을 가지고 있다보니 예정보다 빨리 끝날 때도 있고, 주어진 시간 안에 해결하지 못할 때도 있다.

    RANSAC의 장점은 통계적으로 대충 몇 iteration이면 몇%의 확률로 좋은 모델이 나올 지 추정을 할 수 있다는 것 이다. 그렇기에 알고리즘 플래닝을 할 때 유용하다. 또, 수많은 경우의 수에서 빠르게 모델을 추정할 수 있다. 개량된 RANSAC (e.g. Lo-RANSAC이나 PROSAC) 등을 이용할 경우, 더욱 빠르게 답을 찾을 수 있다.

    RANSAC의 단점은 많이 있는 것 같다. 
    첫째는, 랜덤하게 샘플을 뽑기 때문에, 거의 대부분의 경우 매번 다른 모델이 추정된다. 이 때문에 SLAM 알고리즘의 정확도에 대해 계산할 때, 매번 다른 결과가 나오게 된다. Deterministic test도 믿기 조금 어려울 때가 있다 (random seed를 고정해도, 그냥 그 seed가 안좋아서... 운이 안좋아서 결과가 잘 안나오는건지, 아니면 진짜 내 알고리즘이 좋지 않아서 안나오는건지 구분하기 어려울 때가 있다). 
    둘째는, error threshold를 넘겨도 local minimum 모델이 나올 수 있다. local minimum 모델이 나올때는 두가지 경우가 생길 수 있다. 첫째는 모델 자체가 완전히 잘못된 경우 geometric verification에 실패할 수 있다. 둘째는 모델이 global minimum에 가까운 local minimum인 경우인데, 이 경우 error가 쌓이게 누적되게 된다. 이러한 단점을 해결하기 위해서는 100% 확률로 global minimum 모델을 찾는 방법을 써야하는데, 이 경우 탐색해야하는 경우의 수도 너무 많고 수많은 최적화 계산이 들어가기 때문에 실시간으로 작동할 수 없다. 실시간으로 작동해야하는 경우에는 RANSAC이 적합하지만, 비실시간으로 돌아도 괜찮다면 최적화 manifold를 쉽게 만들어주는 convex relaxation을 거친 후 MAXCON 방식으로 풀어가는 방식이 종종 사용되기도 한다. 
    셋째는 데이터에서 여러 모델이 나올 수 있는 경우에도 단 하나의 모델만 찾을 수 있다.

### 5. Loop closure가 무엇인가요? 어떻게 하는건가요?

- 답

    SLAM을 진행하면서 로봇이 이전에 왔던 공간에 다시 도달 했을 때, 경로에 loop가 생기게 됩니다. 이는 geometric constraint가 생긴다는 것을 의미하며, graph optimisation이 가능하게 합니다. Graph optimisation을 수행하면 모든 카메라 pose와 map point 들에 대해서 최적화를 진행함으로써, 가장 에러가 적어지는 pose와 map point들의 위치를 구할 수 있습니다.

    Loop closure를 수행하기 위해서는 우선적으로 loop가 생겼음을 인지해야합니다. Visual-SLAM의 경우에는 image retrieval 기술을 통해, 현재 내가 보고있는 이미지와 이전에 저장해두었던 keyframe 이미지의 유사도를 계산하고 loop closure detection을 수행합니다. 이 부분에 있어서 Bag-of-visual-words나 VLAD 기법을 사용합니다. 최근에는 NetVLAD나 DELF와 같은 딥러닝 기반 image retrieval 기능들도 주목을 받고 있습니다. LiDAR SLAM의 경우에는 아마 3D feature들을 벡터화 시킨 것들의 유사도로 loop closure detection을 하지 않을까 추측합니다.

    Loop가 있다면, 두 이미지간의 transformation 계산을 함으로써 geometric constraint를 만듭니다. 이 두 constraint를 hard constraint로 잡고, loop에 있는 카메라 pose나 map point들을 최적화 합니다. 최적화의 경우, 모든 카메라 pose와 map point를 최적화 할 수도 있고, 때와 목적에 따라서 motion-only BA를 할 수도 있습니다.

    Feature-based SLAM의 경우 reprojection error 기반 최적화인 Bundle adjustment를 사용합니다. Direct method의 경우에는 잘 모르겠습니다 ㅠㅠ (L-DSO 논문에 나와있는걸 찾아볼 것 같습니다).

### 6. Bundle adjustment에 대해서 설명해주세요.

- 답

    카메라의 optical center와 카메라가 바라보고 있는 map point를 연결하면 하나의 light ray가 만들어집니다. 카메라가 여러개의 map point를 보고 있으면 여러개의 light ray가 있겠구요. 그리고 여러개의 이미지들이 여러개의 map point들을 보고 있다면 다수의 light ray가 있습니다. 이 때, 이것을 'bundle of light rays'라고 합니다.

    Bundle adjustment는 말 그대로 이 light ray들을 가장 정확한 위치에 갈 수 있게 '조정해주는' 것입니다. 정확한 위치에 도달하기 위해서는 reprojection error 가 작아야합니다. Reprojection error는, 추정된 map point가 2D 이미지 위를 통과할 때, 통과되는 위치가 map point에 해당하는 2D feature의 위치와의 픽셀 거리를 이야기합니다. 완벽한 light ray의 경우, 정확하게 이 2D feature를 통과할 것입니다.

    하지만 카메라 모션 추정에서 생긴 에러라던지, 이미지 속에서의 에러 등으로 항상 에러는 있으며, 시간이 지날수록 누적됩니다. Bundle adjustment는 시간에 관계없이 다수의 데이터를 한꺼번에 최적화시킴으로써, 이 에러들이 최소화되는 지점을 찾고, 이로써 에러를 분산시키는 효과도 가집니다. 

    Bundle adjustment의 수학적인 내용으로는, Graph-optimisation 기법을 통해 reproejction error라는 cost function을 가지고 non-linear optimisation을 수행합니다. 이 때, 최적화하는 것은 가장 정확한 카메라 pose들과 map point의 위치입니다. 보통 카메라 pose는 빠르게 정답을 찾아가는 것에 비해, map point는 훨씬 느리게 정답을 찾아가게 됩니다.

    {% asset_img "ba.png" "ba" %}
    

### 7. Gradient descent 방식, Newton-Raphson 방식, Gauss-Newton 방식, Levenberg-Marquardt 방식 최적화 기법은 어떻게 다른가요?

### 8. Kalman filter와 Particle filter에 대해서 설명해주세요

### 9. Visual SLAM과 Visual-Inertial Odometry는 무슨 차이가 있나요?

- 답

    Visual SLAM은 카메라만을 센서로 이용해서 SLAM을 하는 방식입니다. SLAM이기 때문에 단순히 odometry가 아닌 global consistency를 가져야 하며, 이 때문에 loop closure 등의 기능을 가지고 있습니다.

    Visual-Inertial odometry는 카메라와 IMU를 센서로 이용하며 카메라의 궤적을 추정하는 방식입니다. Loosely-coupled 방식과 Tightly-coupled 방식으로 나뉘며, 디자인 방식에 따라 장단점이 달라지기도 합니다. Loosely-coupled 방식에서는 각각 센서의 추정치를 합치는 것이고, Tightly-coupled 방식은 두 센서의 추정치를 동시에 이용합니다. IMU의 속도가 카메라의 속도보다 월등히 빠르기 때문에, inter-frame motion을 추정하기 좋으며, 종종 IMU 값으로 motion model을 완전히 대체하기도 합니다. 카메라로부터 오는 이미지에 feature가 충분하지 않거나, blur 등으로 카메라에서 제대로 정보를 얻지 못할 때, IMU 추정치로 대체하기도 합니다. 하지만, IMU의 특성 상 발산하는 성격을 가지고 있기 때문에 카메라가 빨리 돌아와야합니다. 다중 센서를 사용하다보니 캘리브레이션도 필요합니다.

### 10. Tight-coupling과 Loose-coupling의 차이를 설명해주세요.

- 답 (추가 필요)

    Tight-coupling은 여러개의 센서의 값을 동시에 사용해서 단일 추정치를 만드는 방법이고, loose-coupling은 각각의 센서로부터의 추정치를 합치는 것입니다. 

### 11. Map point를 만드는 방식들 중 생각나는 것들을 다 설명해보세요.

- 답
    - Initialization을 할 때 사용하는 triangulation
        - 두개의 이미지로부터 feature matching을 하고, 7-point / 8-point / 5-point 알고리즘 등을 통해 R|t를 구한 후, triangulation을 통해 초기 map point를 만드는 것
    - Map point를 추가할 때 쓰이는 triangulation
        - 기존의 map에 새로운 map point를 추가할 때... 다수의 이미지들로부터 feature matching을 하고, 가장 정확한 map point를 만드는 two-view를 찾음. 그 후 이 값을 초기 값으로 잡아서 n-view optimization을 진행함으로써 최대한 정확한 map point를 찾고 추가함.

### 12. 실내 SLAM과 실외 SLAM은 어떤점들이 다를까요?

- 답
    - 조명 / 환경
        - 실내 SLAM의 경우 조명이 일정한 편입니다 → 설치된 조명이 많기 때문에 시간에 따라 조명이 크게 변하지 않습니다. 비, 눈, 안개가 없습니다.
        - 실외 SLAM은 낮과 밤으로 조명이 바뀝니다. 비, 눈, 안개가 있으며 센서 값에 큰 영향을 줄 수 있습니다.
    - 오브젝트
        - 실내의 경우 움직이는 오브젝트가 있을 수도 있고, 없을 수도 있습니다 → Deformable object가 있을 수 있고, moving object가 있을 수도 있습니다.
        - 실외의 경우 건물이나 도로의 모습들은 일정할 수 있으나, 시간에 따라 모습이 바뀔 수도 있습니다. 예를 들어서, 봄/여름에는 나뭇잎이 파란색이고, 가을에는 갈색, 겨울에는 나뭇잎이 없을 수 있습니다. 자동차와 같이 형태는 일정하나 움직이는 object가 있을 수 있고, 보행자와 같이 형태가 변하며 움직이는 deformable object도 있습니다.
    - 로봇의 속도
        - 대부분 실내 SLAM의 경우 벽들이 많기 때문에 빠르게 움직이는 로봇 플랫폼이 많이 없습니다.
        - 실외의 경우, 드론의 속도 또는 자동차 자율주행을 고려했을 때 빠른 이동속도도 고려할 수 있습니다.
    - 센서
        - 실내에서는 GPS가 잘 안됩니다. 하지만 와이파이나 비콘 설치가 용이할 수 있습니다.
        - 실외에서는 GPS가 실내보다는 잘 되는 편이지만, 고층빌딩 사이나 터널 안에서는 잘 안될수도 있습니다.

### 13. RANSAC 말고 어떤 다른 Outlier removal 기법이 있나요? 아는 방식이 있다면 설명해주세요.

- 답 (추가 필요)

    Robust estimator (M-estimator)가 있습니다.

### 14. $SO(3)$와 $SE(3)$는 어떻게 다르나요?

- 답 (추가 필요)

    $SO(3)$는 Special Orthogonal Group으로써 3x3 Rotation matrix를 표현합니다. $SO(3)$ 매트릭스의 inverse는 transpose matrix와 동일합니다.

    $SE(3)$는 Special Euclidean Group으로써 4x4 transformation matrix를 표현합니다. $SO(3)$가 내부에 포함되어있습니다. $SE(3)$를 inverse하기 위해서는 아래의 방법을 사용해야합니다.

### 15. Lie Group과 Lie Algebra는 왜 쓰이나요?

- 답

    3x3 rotation matrix (i.e. $SO(3)$)에 대해서 small movement를 계산하기 위해 → 즉, 미분을 하기 위해 있습니다. 

    Rotation matrix에서 미분이 가능해진다는 것은, $SO(3)$ space에서의 manifold를 구축할 수 있다는 것이고, 이는 즉 optimisation을 할 수 있다는 것을 의미합니다.

    Camera pose가 최적화되야하는 과정 (e.g. bundle adjustment, local optimisation) 에서 많이 사용됩니다.

### 16. PTAM, ORB-SLAM, SVO의 차이점을 이야기해주세요.

- 답

    PTAM은 2개의 프로세스 쓰레드를 가진 형태의 feature-based SLAM이며, 1번 쓰레드는 tracking, 2번 쓰레드는 mapping 목적을 가집니다. Tracking 쓰레드에서는 실시간 frontend를 담당하며, FAST 코너를 뽑고 매칭을 함으로써 이미지 프레임간의 변환을 계산합니다. 동시에, 키프레임 선정을 담당하는데, 선정된 키프레임은 mapping thread로 넘어갑니다. Mapping thread에서는 키프레임들로부터 bundle adjustment를 통해서 map point를 생성합니다. 전체적으로 Loop closure 기능이 없기 때문에, table top 사이즈 정도에서 사용하기 적당합니다.

    ORB-SLAM은 3개의 프로세스 쓰레드를 가진 형태의 feature-based SLAM이며, 1번 쓰레드는 tracking, 2번 쓰레드는 local mapping, 3번 쓰레드는 loop closure / global optimization을 담당합니다. Tracking 쓰레드에서는 실시간 frontend를 담당하며, ORB feature를 뽑고 매칭을 하고 이미지 프레임간의 변환을 계산합니다. 동시에 키프레임 선정을 담당하며, 선정된 키프레임은 local mapping 쓰레드로 넘어갑니다. Local mapping 쓰레드에서는 지난 몇개의 키프레임들을 이용해 bundle adjustment를 함으로써 카메라 포즈 및 map point 최적화를 진행해주고, 새로운 map point를 추가하기도 합니다. 동시에, ORB-SLAM만의 특성인 covisibility graph / essential graph를 관리하기도 합니다. Loop closure / global optimization 쓰레드에서는 loop detection을 진행하고, 비실시간성 작업인 loop closure / global optimization을 bundle adjustment를 사용하여 진행합니다. ORB-SLAM은 버전 1, 2, 3까지 나왔는데, 버전1은 monocular SLAM, 버전2는 Stereo/RGB-D SLAM, 버전3은 Visual-Inertial SLAM입니다.

    SVO는 feature-based와 direct method를 섞은 hybrid 방식입니다. SVO는 우선 fast corner feature를 뽑고 direct 기반의 feature tracking 방식을 사용합니다. 빠른 카메라를 사용하기 때문에 photometric 변화가 크게 없어 빠르게 direct 기반 tracking 방식이 수렴합니다. 트랙킹 된 feature들을 기반으로 sliding window bundle adjustment를 진행함으로써 빠르게 카메라 포즈를 정확하게 찾아낼 수 있습니다. 하지만 loop closure 기능이 없기 때문에 global consistency를 기대할 수 없으며, 그 때문에 SLAM이라고 부르기보다는 odometry 시스템이라고 부르는 것이 더 적합하다고 생각합니다. 

### 17. Essential Matrix와 Fundamental Matrix에 대해 설명해주세요.

### 18. 카메라와 IMU 캘리브레이션은 어떻게 하나요?

### 19. Feature detector와 Feature descriptor의 다른 점을 설명하세요.

- 답

    Feature detection은 이미지로부터 interest point, 또는 keypoint라는 것을 추출해내는 작업입니다. Interest point는 이미지에 투영되는 geometric property의 특성을 이야기하는데, corner나 blob등이 주로 사용되는 방식입니다. Corner의 경우 Harris corner detection, FAST corner detection 등이 주로 사용되고, Blob의 경우 SIFT, SURF, MSER 등이 대표되는 방식입니다. 하나의 물체를 두개의 다른 시야에서 바라볼 때, 우리는 두 이미지에서 Feature detction을 수행하면 공통되는 feature가 나타날 것을 기대하고 사용합니다.

    Feature detection을 수행하고나면 추출된 interest point들의 2D pixel location 값들이 나옵니다. 카메라 포즈 연산이나 feature 비교 등의 연산을 하기 위해서는, feature matching을 수행해야합니다. Feature descriptor는 해당 interest point의 주변 모습이 어떻게 생겼는지에 대한 정보를 벡터화해서 저장하고, 비교할 수 있는 distance 척도 시스템을 가지고 있습니다. 예를 들어서, SIFT descriptor 의 경우 주변 16 x 16 패치에 대해서 gradient histogram을 추출하고, floating-point container에 저장합니다. SIFT descriptor 2개의 정보를 비교했을 때 동일하다면 같은 3D 위치를 뜻하는 feature이고, 크게 다르다면 다른 3D 위치를 뜻하는 feature라고 인식할 수 있겠습니다. Floating-point descriptor 말고 Binary descriptor도 있는데, ORB feature에 들어간 BRIEF 등이 이런 방식을 사용합니다. 이러한 feature들은 floating-point가 아닌 binary 수 (0,1)들을 벡터화시키기 때문에 훨씬 적은 메모리를 사용하고, hamming distance를 이용한 비교를 하기 때문에 빠르게 비교가 가능합니다.

### 20. Feature의 invariance는 어떤 것을 의미하나요?

- 답

    Invariance는 '불변성'을 의미하며, feature invariance는 어떠한 상황에도 물체에서 동일한 feature를 찾을 수 있게 해주는 해당 feature detection / descriptor 방식의 고유한 특성을 뜻한다. Invariance에는 scale invariance, rotation invariance, viewpoint invariance, illumination invariance 등이 있다.

    Scale invariance는, 가까이서 찍은 이미지에서나 멀리서 찍은 이미지에서나 동일하게 feature가 뽑히는 것을 의미한다. Scale invariance를 구현하기 위해서 보통 image pyramid 기법을 사용한다.

    {% asset_img "fi1.png" "fi1" %}

    Rotation invariance는 이미지가 회전해도 같은 feature가 뽑히는 것을 의미한다. 보통 dominant direction 등을 계산해서 주 방향 축을 잡고, 그 방향축에 대해서 align 후 descriptor 값을 비교한다. SIFT의 경우에도 dominant direction을 잡고, ORB-feature의 경우에는 Image moment를 이용한 dominant direction 계산을 한다.

    {% asset_img "fi2.png" "fi2" %}

    Viewpoint invariance는 rotation invariance와 다르게 단순히 2D 이미지 회전이 아닌, 3D 오브젝트 회전의 경우에도 강인하게 동일한 feature를 뽑을 수 있다는 것을 의미한다.

    {% asset_img "fi3.png" "fi3" %}

    Illumination invariance는 어두운 곳에서나, 밝은 곳에서나, 다른 색의 조명이 있는 경우에도 강인하게 동일한 feature를 뽑는 것을 이야기한다. 

    {% asset_img "fi4.png" "fi4" %}

### 21. Homography matrix가 무엇인가요?

- 답

    Homography matrix는 projective space에서 두개의 2D plane들간의 변환 관계를 뜻한다. 

    $s\left[\begin{array}{l}
    x^{\prime} \\
    y^{\prime} \\
    1
    \end{array}\right]=H\left[\begin{array}{l}
    x \\
    y \\
    1
    \end{array}\right]=\left[\begin{array}{lll}
    h_{11} & h_{12} & h_{13} \\
    h_{21} & h_{22} & h_{23} \\
    h_{31} & h_{32} & h_{33}
    \end{array}\right]\left[\begin{array}{l}
    x \\
    y \\
    1
    \end{array}\right]$

    Homography matrix는 2D-2D correspondence가 있을 때 유추해낼 수 있다. Homography를 유추해내기 위해서는 최소 4개의 correspondence가 있어야한다. Homography matrix를 SVD로 분해하면 Rotation과 Translation을 구할 수 있다.

    이러한 특성 덕분에 아래와 같은 용도로 사용된다.

    1. 같은 물체를 바라보고 있는 두개의 image plane들간의 변환관계 (i.e. rotation / translation).
    
    {% asset_img "hmat.png" "hmat" %}

    2. Planar surface를 바라보고 있는 두개의 카메라 view들에 대한 변환관계 추정. 마커기반 위치추정법들이 이 방식을 많이 사용함.

    {% asset_img "hmat2.png" "hmat2" %}

    3. 회전 모션으로 파노라마 이미지 생성 시 이미지 프레임들간의 변환관계 추정

    {% asset_img "hmat3.png" "hmat3" %}

### 22. Perspective-n-Point 문제가 무엇인가요?

- 답

    Perspective-n-Points 문제는 2D-3D correspondence가 존재할 때, 카메라에서부터 world space에 대한 변환관계를 알아내려고 하는 문제이다. 간단하게 이야기해서, perspective (물체를 바라보고있을 때), n-point (n개의 2D-3D point correspondence가 있다면)로 생각할 수 있다.

    보통 2D 정보는 image feature detection에서 나오고, 3D 정보는 3D reconstruction 단계에서 저장된 point cloud이다. 2D image feature의 경우 feature descriptor를 가지고 있고, 3D point cloud도 각각의 포인트마다 descriptor를 저장하고 있기 때문에 매칭이 가능하다. 물론 다른 방법으로도 매칭할 수 있다.

    PnP 문제를 풀기위해 필요한 최소 correspondence 수는 3개이다. 이를 사용하는 알고리즘이 P3P인데, 종종 방향에 대해 ambiguity가 생기기 때문에 하나를 더 추가해서 사용하기도 한다. 이 외로 EPnP 또는 UPnP와 같은 방법들도 있다.

    위에서 방금 소개했던 방식들은 closed-form으로 계산을 하기 때문에, 2D 정보나 3D 정보에 노이즈가 껴있는 경우 정확한 값이 나오지 않거나, 완전히 다른 값이 나타날 수 있다. 이 때문에 정확한 값을 얻기 위해서는 더 많은 correspondence들을 추출해낸 후 RANSAC을 돌리는 방법이 많이 사용된다. 또 다른 방법으로는 optimization을 사용해서 2D정보와 3D 정보를 변형하는 방법도 있다.

    OpenCV에서 사용하고 있는 PnP 솔버 함수에는 *solvePnP()*와 *solvePnPRANSAC()*이 있다. 후자의 경우가 minimal set 방식을 이용해서 푸는 방식이다. 전자의 경우에는 RANSAC을 사용하지 않고, 문제를 non-linear optimization 방식으로 바꿔버린다. 그렇기 때문에, 얻어진 3D pose로 reprojection을 했을 때 정확하게 작동하지 않을 수 있고, 또 outlier correspondence가 있을 때 위험할 수도 있다. 

    {% asset_img "pnp.png" "pnp" %}

### 23. Inverse depth parameterization이 무엇인가요?

### 24. Feature-based method와 Direct method의 다른 점을 설명하고, 두 방식의 장단점을 설명하세요.

- 답

    Feature-based method는 feature detection + descriptor 추출을 통해서 이미지 속 keypoint들을 sparse하게 찾아냅니다. 이 keypoint들을 매칭을 함으로써 기반으로 카메라의 움직임이나 오브젝트의 움직임과 같은 기하학적인 관계를 추론할 수 있습니다. Bundle adjustment를 통해 최적화를 진행할때는 reprojection error 기반 cost function을 사용합니다.

    - 장점
        - 뽑힌 feature들을 사용해서 camera pose 및 map point 계산 뿐만이 아니라, loop closure detection이나 relocalization 등에도 사용할 수 있습니다.
        - 대부분 사용되는 feature들은 grayscale에서 뽑히기 때문에 간단한 조명변화에도 강인합니다.
        - Motion이 커도 descriptor 매칭을 통해서 정확한 카메라 포즈 + map point 추정이 가능합니다.
    - 단점
        - Feature를 뽑는데에 시간이 들어가는데, feature를 뽑는 행위 자체는 SLAM의 코어 요소가 아닙니다.
        - Feature가 없는 환경에서는 작동하지 않습니다 (e.g. 아무것도 없는 벽, 바닥)
        - Sparse map만 만들 수 있습니다. Dense map을 만들려면 새로운 추정 과정이 필요합니다.

    Direct method는 feature를 찾지 않고, 이미지 전체의 픽셀 값들을 사용합니다. 그렇기 때문에 feature-based 방식보다 데이터의 양이 좀 더 많고, 이 덕분에 이미지 artefact가 없을 때 최적화에서 노이즈 outlier의 효과를 줄일 수 있습니다. 트랙킹 과정에도 최적화 과정이 들어가며, photommetric error 기반 cost function을 사용합니다.

    - 장점:
        - Feature가 없는 곳에서도 잘 작동합니다.
        - Feature를 안 뽑기 때문에 시간이 절약됩니다.
        - Semi-dense 또는 Dense map을 만들 수 있습니다.
    - 단점:
        - sparse / semi-dense를 사용하는 경우 blur에 살짝 취약합니다.
        - 카메라 셋팅을 잘 잡아야하고, photommetric calibration도 제대로 해야합니다.
        - 카메라 모션이 작아야합니다. 천천히 움직이거나, 카메라가 빨라야합니다. 모션이 크면 잘못된 값으로 수렴할 수 있기도 하고, 더 오래 걸리기 때문입니다.
        - 조명이 빠르게 바뀌는 곳에서는 작동하지 못합니다.

### 25. Marginalization이 무엇인가요?

### 26. Floating-point descriptor와 Binary descriptor는 무슨 차이가 있나요?

- 답

    Floating-point descriptor는 Descriptor patch에 대한 정보를 벡터화 할 때 floating point (i.e. 소수점)으로 저장하는 방식을 뜻 합니다. 정확한 vector 값을 저장할 수 있지만, 메모리를 많이 차지하는게 단점입니다. Floating-point descriptor들끼리 비교할 때는 L2 Norm이 사용됩니다. SIFT, SURF descriptor 등이 이러한 방식을 사용합니다. 매칭에 있어서는 FLANN 라이브러리에서 Approximate nearest neightbour 방식 중 Locally Sensitive Hashing (LSH)를 많이 사용하는 편입니다.

    Binary descriptor는 descriptor patch에 대한 정보를 벡터화 할 때 binary number (i.e. 0,1)로 저장하는 방식을 뜻합니다. 이 때 binary number는 보통 정해진 descriptor 패턴에서 두 픽셀을 비교했을 때 밝기 차이 또는 순서를 통해 0 또는 1을 저장합니다. Floating-point descriptor에 비해 메모리를 훨씬 덜 차지하고, 읽는 속도도 빠릅니다. Binary descriptor 들끼리 비교할 때는 Hamming distance가 사용됩니다. BRIEF, BRISK 등이 이러한 방식을 사용합니다. 매칭에 있어서는 FLANN 라이브러리에 들어있는 Multi-probe LSH나 kd-tree 방식을 사용합니다.

### 27. IMU Pre-integration이 무엇인가요?

### 28. SLAM을 하는 도중 움직이는 물체가 있을 때 어떻게 해야할까요?

### 29. Graph optimization이 무엇이고, 왜 쓰나요?

### 30. Direct method와 Optical flow가 어떻게 다르나요?

### 31. Sparse mapping과 Dense mapping의 차이와 장단점을 설명하세요.

### 32. Extended Kalman filter와 Bundle adjustment의 차이를 설명하세요.

### 33. Line / Edge 추출 방식들을 설명하고 각각의 장단점을 설명하세요.

### 34. Epipolar constraint가 무엇이고 어떻게 쓰일 수 있을까요?

### 35. SLAM에서 Keyframe이 무엇일까요?

### 36. Scale drift는 왜 나타나는걸까요?

### 37. 여러개의 map을 하나로 뭉치려면 어떻게 해야할까요?

### 38. Relocalization이란 무엇인가요?

### 39. BoW와 VLAD에 대해서 설명해주세요.

### 40. Kalman filter와 Extended Kalman filter의 차이를 설명해주세요.

### 41. Patch similarity를 구하는 방법 중 SSD, SAD, NCC를 비교해주세요.

### 42. Feature descriptor distance는 어떤 방식으로 구해지나요?

### 43. 이미지에서 노이즈를 줄이는 방식에 대해 소개해주세요.

### 44. 반복되는 패턴을 바라보고 있는 2장의 이미지가 있습니다. 이 때 두 이미지 사이의 정확한 변환관계를 어떻게 구하나요?

### 45. FLANN, LSH, Multi-probe LSH, kd-tree에 대해서 설명해주세요.

### 46. 정확한 Feature matching을 만드는 전략에는 어떤 방법들이 있을까요?

### 47. Image pyramid에 대해서 설명해주세요.

### 48. MLE와 MAP는 어떤게 다른가요?

### 49. DLT (Direct Linear Transform)에 대해서 설명해주세요.

### 50. Visual-LiDAR fusion 방식에 대해서 아는대로 얘기해주세요.
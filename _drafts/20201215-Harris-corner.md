---
title: Harris Corner (Harris & Stephens 1988) 논문 리뷰
date: 2020-12-15 20:38:19
tags: 
- SLAM
- Feature Detector
- Visual Feature
- Visual-SLAM
- CV
categories: 
- [SLAM]
excerpt: Harris & Stephens 1988 - A Combined Corner and Edge Detector 논문 리뷰입니다
---

[***Harris corner 논문 + 노트필기***](https://drive.google.com/file/d/1FQoRwpzDBjhaz63mldfn71ydHF_dztM4/view?usp=sharing)를 첨부합니다.

<br>

# 논문 소감
---

Harris corner는 굉장히 오래된 기술이고, 현재 시점에서는 거의 사장된 기술이다.

ORB와 같이 더 빠르고 강인한 feature detector가 선호되기 때문이다.

종종 ORB에서 FAST score 대신 Harris corner score를 계산하지만, 이 또한 하나의 휴리스틱일 뿐이다.

<br>

Harris corner를 통해 우리는 distinctive point로써 corner의 장점과 기하학적 특성을 배운다.

그리고 보통 곧바로 더 성능이 좋은 SIFT, FAST 를 공부한다.

<br>

하지만 이번에 논문을 직접 읽어보면서 몇가지 새롭게 알게 된 사실이 있다.

1. Harris detector는 corner뿐만이 아닌 edge도 검출할 수 있다 (그것도 꽤 강인하게).

2. 논문은 사실 edge tracking에 초점을 두고, corner detection은 사실 부산물이다.

3. 논문이 추구했던 Edge tracking 방법론들은 현재 시점에서 모두 사장되었다.

우리가 현재 휴리스틱으로 사용하고 있는 방법들도, 언젠가는 사장되어 다른 기발한 기술이 대체하겠지? 라는 생각을 해본다.

<br>

# Abstract
---

- 이미지 시퀀스에서 3D 공간이 어떻게 변화하는지 이해하기 위해서는, edge filtering이 굉장히 중요하다. <br><br>
- 이 논문에서는 auto-correlation을 이용해 corner와 edge를 둘 다 검출 할 수 있는 detector 알고리즘을 제안한다.

<br>

Abstract를 통해 다음과 같은 생각이 든다.

Edge filtering이 왜 3D 공간 변화를 이해하는데에 도움이 될까?

Auto-correlation이란?


<br>

# Introduction
---

- Image feature를 이용해서 3D 공간 또는 3D 공간 속 움직임을 이해할 수 있다. <br><br>
- Image feature에는 corner와 edge가 있다.
  - Corner에는 connectivity가 없기 때문에 higher level description을 얻기 어렵다.
    - 여기서 higher level description이란, 면 (surface)이나 물체 (object) 정보를 뜻한다.
  - Edge는 connectivity 정보가 있다.

<br>

Feature matching을 통해 3D 공간 속 움직임 정보를 추출해낼 수 있다는 것을 유추할 수 있다.

여기서 point feature (i.e. corner)로 부터 higher level description을 얻을 수 없다고 하였지만, 요즘에는 descriptor 정보를 추출하고 clustering함으로써 물체를 구분할 수 있다 (e.g. Bag of Visual Words).

전체적으로 저자는 edge에 좀 더 관심이 있어보인다.

<br>

# The Edge Tracking Algorithm
---

- Edge matching은 epipolar geometry를 통해 할 수 있다.
  - 하지만, camera motion을 모르는 경우에는 aperture problem이 있기 때문에 매칭을 할 수 없다. 
  - 이 경우, motion을 다른 방법으로 풀어내면 된다. <br><br>
- Edge matching을 통해 edge tracking을 풀 수 있다.
  - Edge tracking을 통해 edge의 3D 공간 속 위치를 알아낼 수 있다. <br><br>
- Edge connectivity를 이용해서 공간이나 물체의 wireframe 모델을 재구축하거나, 3D surface 정보를 추출할 수 있다. <br><br>
- Canny edge operator를 통해 edge를 추출할 수 있다.
  - Detection threshold를 잘못 설정하면 edge의 생김새가 달라지기 때문에, 일정한 edge를 지속적으로 추출하기 어렵고 매칭 또한 어렵다.
    - 이 문제를 해결하기 위해서, 이미지에서 corner를 추출하고, 이 코너점에 edge끼리 겹칠 수 있도록 threshold를 조정해야했다. 
    - Corner detector에 대해 설명하기 위해 (다음 목차에서) Moravec 코너 추출법에 대해 소개한다.

<br>

이 기술이 edge를 추출하기 위한 목적으로 설계된 것이 여기서 보인다.

1988년도 당시에는 object나 3D scene을 wireframe model 등으로 이해하려는 시도가 많았다.

본문에서 Canny와 corner detector를 함께 사용해서 안정적인 edge 추출 방식에 대해 이야기하였다.

하지만 이 방법은 굉장히 오래걸리는 계산이였을 것 이라고 추출한다.

이유는... Canny 자체가 굉장히 무거운 알고리즘인데, edge 정보와 corner정보가 일치할 때 까지 threshold를 바꿔가며 Canny 계산을 해야하기 때문이다.

<br>

# Moravec Revisited
---

- Moravec 코너 추출법은, 이미지 상 로컬 윈도우를 하나 만들고, 한픽셀씩 우, 우/상, 상, 좌/상 으로 움직인다.
  - 이 때, 윈도우 내부의 평균 밝기 값을 기록한다. 수식으로는 다음과 같이 표현된다.
  $E(x, y) = \sum_{u,v} (w_{u,v} | I_{x+u, y+v} - I_{u,v})^2$.
    - $E$ - 평균 밝기 값의 변화량
    - $w$ - 윈도우 내부의 평균 픽셀 값을 계산하는 행동
    - $u,v$ - 픽셀 위치 변화
    - $x,y$ - 이미지 속 픽셀 위치 (window 중앙 픽셀의 위치)
    - $I$ - 이미지 밝기 값  <br><br>
- 이 평균 밝기 값을 통해 corner인지, edge인지, 아무것도 아닌지를 평가한다.
  - Edge도 corner도 아닌 경우: 평균 밝기 값에 아주 작은 변화만 있을 것이다.
  - Edge의 경우:  평균 밝기 값이 한쪽 방향으로 이동할 때만 큰 변화가 생긴다. 예를 들어, 좌/우로 움직일 때는 변화가 크기만, 상/하로 움직일 때는 변화가 작을 것이다.
  - Corner의 경우: 어떤 방향으로 움직여도 평균 밝기 값이 크게 변화한다.
  - 평균 밝기 값이 작게, 또는 크게 변했다는 것을 결정하기 위해 임의의 Threshold 값을 사용했다.

<br>

왜 우, 우/상, 상, 좌/상 일까?

굉장히 이상한 움직임인데, 이 부분은 아직까지 이해가 가지 않는다.

[Moravec의 박사논문](https://www.semanticscholar.org/paper/Obstacle-avoidance-and-navigation-in-the-real-world-Moravec/93b376bd451db8ed94a18c556da16f25a3e7961b?p2df)은 1980년도에 나왔으며, 직접 개발한 corner detector 알고리즘과 feature matching을 통한 motion 추정 방식을 소개한다. 

가장 초기의 corner detector 알고리즘인데, Harris corner가 실질적으로 그 아이디어를 계승한다고 볼 수 있다 (수식도 굉장히 비슷하다).

이 논문은 박사논문이다보니 굉장히 길어서 읽기 힘든데, Harris corner 논문에서 쉽게 정리해주는게 너무 감사하다.

또 다른 신기한 점은, 왠만한 Harris corner 블로그 글들을 보면 실제 논문에서 사용하는 $u,v$와 $x,y$ 노테이션을 반대로 사용한다는 것이다.

<br>

# Auto-Correlation Detector
---

- Moravec 알고리즘에는 몇가지 단점이 있다. <br><br>
- Sliding window가 실제로 좌/우/상 등으로만 움직이기 때문에, 코너 response가 anisotropic하다 (즉, 특정 방향만의 값으로 계산되었다)
  - 이 문제를 해결하기 위해, 논문에서 제안하는 알고리즘은 image gradient를 계산함으로써 정확하게 intensity shift 방향을 알아내었다.
  - Image gradient는 Taylor expansion을 통해 1차 미분으로 근사되었고, 이 1차 미분 값은 sobel operator로 구할 수 있다.
    - Taylor Expansion: 
    $E_{x,y} = \sum_{u,v}(w_{u,v}[I_{x+u,y+v} - I_{u,v}])^2 \simeq \sum_{u,v}(w_{u,v}[xX + yY + O(x^2,y^2)])$
    - Sobel operator: 
    $ X = I \circledast (-1, 0, 1) \simeq \delta I / \delta x$, 
    $ Y = I \circledast (-1, 0, 1)^T \simeq \delta I / \delta y$
    - Small shift approximation:
    $ E(x,y) = Ax^2 + 2Cxy + By^2 $
    , where
    $ A = X^2 \circledast w$
    $ B = Y^2 \circledast w$
    $ C = (XY) \circledast w$
  - X방향, Y방향, XY방향에 대한 정보를 2x2 매트릭스에 담아서 auto-correlation matrix를 구성하였다.
    - 논문 버전으로는...
    $\begin{bmatrix}
        A & C \\
        C & B 
    \end{bmatrix}$
    - 모던 버전으로는...
    $E(x,y)=\left[\begin{array}{ll}x & y\end{array}\right]\left[\begin{array}{ll}\sum\left(\frac{\partial I}{\partial x}\right)^{2} & \sum \frac{\partial I}{\partial x} \frac{\partial I}{\partial y} \\ \sum \frac{\partial I}{\partial x} \frac{\partial I}{\partial y} & \sum\left(\frac{\partial I}{\partial y}\right)^{2}\end{array}\right]\left[\begin{array}{l}x \\ y\end{array}\right]$ <br><br>
- Sliding window가 노이즈의 영향이 있었다.
  - 기존의 sliding window는 그냥 평균 값을 사용했지만, Harris corner의 window는 Gaussian blur를 적용하였다. 또, sliding window의 모양을 원형으로 바꿨다.
  $ w_{u,v} = exp -(u^2 + v^2) / 2 \sigma ^2 $ <br><br>
- 단순히 Threshold 값보다 높으면 됬기 때문에, edge에도 잘 반응하였다.
    - Auto-correlation 매트릭스는 해당 sliding window 속 이미지의 shape을 표현하기도 하였다. 즉, 이 매트릭스의 Eigenvalue 들을 구할 수 있으면, 해당 shape이 corner인지, edge인지 구분할 수 있었다.
    - 두개의 Eigenvalue 중 둘 다 작으면 flat, 하나만 크면 edge, 둘 다 크면 corner로 판단할 수 있다. 
      - 여기에 적용된 논리는 위의 Moravec 알고리즘에 적용된 논리와 비슷하다.

<br>

Harris corner의 알고리즘을 설명해준다.

사실 Moravec과 크게 다른 바가 없다.

Image gradient를 사용했다는 점, 그리고 노이즈를 줄여줬다는 점을 제외하고는 실질적으로 Moravec의 기존 아이디어를 그대로 사용한다고 볼 수 있다.

<br>

# Corner/Edge response function
---

- Eigenvalue를 계산해내어 shape을 추정하는 것이 정확하겠지만, 계산량이 너무 많다.
  - Tr(M)과 Det(M)을 사용하여 대신 계산한다. k 는 사용자가 정할 수 있다.
  $Tr(M) = \alpha + \beta = A + B $
  $Det(M) = \alpha\beta = AB - C^2 $
  Corner response $R = Det(M) - kTr(M)^2$  <br><br>

- Sliding window가 상하좌우대각 8 방향 모두 response 값이 local maximum이 나타난다면 corner로 인식한다. 
- Sliding window가 x나 y방향으로 local minima이고 음수라면 edge로 인식한다.
- Edge hysteresis를 적용해서 edge continuity를 보강할 수 있다.

<br>

논문에서 corner response는 alpha와 beta를 구해내서 얻어낸 Determinant와 Trace 값을 사용한다.

이는 그때 당시 컴퓨터로 eigenvalue decomposition을 하기엔 너무 오래걸렸기 때문이다.

하지만 요즘 컴퓨터는 빠르고, 2x2 매트릭스의 eigenvalue decomposition은 금방할 수 있다.

그렇기에 더욱 정확한 값을 얻어내기 위해, OpenCV 구현 코드 등에서는 eigenvalue를 구하고, 

$Det(M) = \lambda_{1}\lambda_{2}$
$Tr(M) = \lambda_{1} + \lambda_{2}$

...를 사용한다.

<br>

# 코드 구현
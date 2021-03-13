---
title: Image projection의 원리
date: 2021-02-21 20:38:19
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- Image projection
- Pinhole camera
categories: 
- [SLAM, 50가지 CV/SLAM 기술면접 질문들]
excerpt: Image projection의 원리에 대해 설명합니다.
---

# 예상 면접 질문: 
1. Image projection 과정에 대해서 설명해주세요
2. 카메라는 어떻게 작동하나요?
3. 3D 공간이 2D 이미지로 투영되는 방법에 대해 설명해주세요
4. Projection matrix에 대해 설명해주세요.

# 답

{% asset_img "projection1.png" "projection" %}

{% asset_img "projection2.png" "projection2" %}

**3D 공간의 정보를 2D 이미지로 투영하는 과정**을 image projection이라고 합니다.

3D 공간 위의 어떠한 점를 $[X,Y,Z,1]^T$로 표현하고, 2D 이미지 위의 픽셀 위치를 $[u,v,1]^T$ 로 표현하겠습니다. $X,Y,Z$는 World frame으로부터의 3D coordinate이고, $u,v$는 Camera frame으로부터의 pixel coordinate를 의미합니다. $[X,Y,Z,1]^T$와 $[u,v,1]^T$ 모두 밑에 1이 붙는데, 이 1은 homogeneous coordinate 표기법으로 위치를 표현하기 위함입니다. Homogeneous coordinate로 표현함으로써 3차원-2차원 변환 관계를 scale 차원 정보를 포함해서 계산하는 projective geometry를 통해 계산할 수 있게 되는데, 이 scale 차원 정보가 1일 때는 우리가 익숙한 euclidean space에서 계산할 수 있습니다.

카메라가 $[X,Y,Z,1]^T$의 3D 포인트를 바라보고 있을 때 카메라의 optical centre와 3D 포인트 사이의 직선을 그릴 수 있습니다. 이 직선은 image plane을 통과하며, 이는 3D 포인트가 2D 이미지에 투영될 수 있다는 것입니다. 투영을 수행하기 위해서는 우선 3D 포인트를 world coordinate frame 기준이 아닌 camera coordinate frame 기준에서 표현해야 하고, 이를 위해 world->camrea frame 변환이 필요합니다. 이를 위해 Camera coordinate frame과 World coordinate frame의 회전과 이동 (rotation / translation)에 대한 변환 관계를 계산해야하고, 이 변환 관계는 Euclidean space에서의 $SE(3)$로 표현하면 3x4 매트릭스 형태를 가집니다. (보통의 $SE(3)$ 매트릭스는 4x4 형태이지만, 수식을 간단하게 표현하기 위해 euclidean space로 만들어 아랫단의 [0, 0, 0, 1]을 제거하여 3x4 매트릭스로 표현하였습니다. [0, 0, 0, 1]을 붙혀 4x4로 계산해도 같은 결과가 나옵니다.) 이 3x4 매트릭스를 Extrinsic matrix라고 부르며, $[X,Y,Z,1]^T$ 에 이 매트릭스를 곱하면, 카메라의 coordinate frame에서부터 $[X,Y,Z,1]^T$ 의 상대적인 R|t를 알 수 있습니다. 

이후, **Intrinsic matrix**를 이용해서 $[X,Y,Z,1]^T$를 2차원 평면에 투영시킬 수 있습니다. Intrinsic matrix는 **Focal Length** (fx, fy)와 **Principal point** (cx, cy) 정보를 가지고 있습니다. Focal length는 normalized scale에서 pixel scale로 변환을 할 수 있게 해줍니다. cx와 cy는 보통 중앙 픽셀의 위치 값을 가지는데, 이는 coordinate frame을 optical centre (또는 이미지 중앙)이 아닌, 좌측 상단에 올려놓기 위함입니다. 하지만 카메라가 실제로 만들어지면서 완벽하게 중앙에 위치하지 않기 때문에, 몇픽셀의 차이는 있을 수 있습니다. Intrinsic matrix의 3번 row는 $[0, 0, 1]$인데, 이는 homogeneous coordinate를 맞추는 것이 아니고, 3차원→2차원 축소를 위함입니다.

이 계산이 끝나면 $[su, sv, s]^T$이 결과 값으로 나오는데, $s$는 homogeneous coordinates의 영향으로 나온 scale 값입니다. 우리는 euclidean space에서의 값을 원하기 때문에 $s * [u, v, 1]$로 normalize를 시켜주면 pixel coordinate 값, 즉 2차원의 좌표값이 나오게 됩니다.

전체적으로, 3차원 공간에 존재하는 하나의 점을 2차원 이미지의 픽셀 위치로 투영하였고, 이 과정을 image projection이라고 합니다. **Projection matrix**는 intrinsic matrix와 extrinsic matrix를 곱하여 $s * [u, v, 1]^T = P * [X, Y, Z, 1]^T$ 형태로 바꾸었을 때의 매트릭스를 뜻합니다.
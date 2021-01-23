---
title: Visual-SLAM developer roadmap [1] - 컴퓨터 비전 입문 
date: 2021-01-21 21:38:19
mathjax: true
tags: 
- SLAM
- Github
- Roadmap
categories: 
- [SLAM, Visual-SLAM 공부 로드맵]
excerpt: Visual-SLAM 공부 로드맵 - 완전 입문자 편 입니다.
---

원본 Github 링크: [Visual-SLAM roadmap repository](https://github.com/changh95/visual-slam-roadmap)

---

# 컴퓨터 비전 입문

![컴퓨티 비전 입문](https://raw.githubusercontent.com/changh95/visual-slam-roadmap/main/img/beginner.png)

기초적인 프로그래밍, 수학, 영상처리 개념을 익히는 단계입니다.
프로그래밍과 수학 공부를 우선 하신 후, 그 다음 영상처리를 공부하시는 것을 추천합니다.

<br>

---

## 프로그래밍

Visual-SLAM을 하기 위해서는 **C++ 언어**를 익혀야합니다.
C++에서 간단한 pointer 사용과 class 설계 및 이해를 할 수 있으면 충분하다고 생각합니다.
C++이 처음이시라면 다음 링크를 참조하셔서 공부하시면 좋습니다.
- [홍정모의 따배씨](https://www.inflearn.com/course/following-c-plus)
- [Modern C++ for Computer Vision](https://youtube.com/playlist?list=PLgnQpQtFTOGRM59sr3nSL8BmeMZR9GCIA)
- [Cherno C++](https://youtube.com/playlist?list=PLlrATfBNZ98dudnM48yfGUldqGD0S4FFb)
- [freecodecamp C++](https://youtu.be/vLnPwxZdW4Y)

<br>

---

## 수학

고등학교 수학에서 요구하는 것 보다 아주 살짝 더 필요합니다.

- **확률과 통계 기초**
  - 우선 가우시안 분포까지만 이해하셔도 됩니다. SLAM에서는 가우시안 형태의 센서 데이터를 많이 사용합니다.
- **로그와 지수**
  - 추후 Lie algebra를 이해하기 위해 필요합니다. 
- **선형대수 기초**
  - 공간에 대한 계산을 하기 위해 전반적인 이해가 필요합니다.
    - 깊게 알수록 더 좋습니다.
  - 국문 강의는 [이상엽의 선형대수 강의](https://youtube.com/playlist?list=PL127T2Zu76FuVMq1UQnZv9SG-GFIdZfLg)를 추천합니다.
  - 영문 강의는 [Gilbert Strang 선형대수 강의](https://youtube.com/playlist?list=PLE7DDD91010BC51F8) 를 추천합니다.
  - 또는 [여기](http://localhost:4000/20210114-graduate-school-math-linear-algebra/)에도 정리가 되어있습니다.
- **미적분 기초**
  - 최적화 계산과 근사치 계산에서 필요합니다. 편미분, Jacobian 등은 지금 알면 좋습니다. Taylor expansion은 꼭 알고 가셔야합니다.

<br>

---

## 영상처리

이미지를 통해 정보를 추출하는 방법들을 공부합니다.

### Projective geometry

아마 입문 단계에서 제일 어려운 부분일 겁니다.

- **Image projection**에 대해 이해해야합니다.
  - 3D scene이 2D 이미지로 투영되는 과정에 대해 이해합니다.
  - [Stachniss 교수님의 렉처](https://youtu.be/uHApDqH-8UE)를 추천합니다.
    - 이를 통해 Visual-SLAM에서 다루는 데이터에 대해 이해할 수 있습니다.
- **3D 공간에서의 강체 운동 (rigid body motion) 및 공간 변환 관계**를 이해합니다.
  - Homogeneous coordinates, homogeneous transformation (i.e. 4x4 매트릭스)에 대해 익숙해져야합니다.
  - [Stachniss 교수님의 렉처](https://youtu.be/MQdm0Z_gNcw)를 추천합니다.
    - 이를 통해 카메라의 움직임, 물체의 움직임을 수식적으로 표현할 수 있게 됩니다.

### 카메라 이해하기

Visual-SLAM에서 사용되는 카메라의 특성에 대해 이해합니다.

- **렌즈의 특성**에 대해 이해합니다.
  - [Mathworks 페이지](https://www.mathworks.com/help/vision/ug/camera-calibration.html)에 설명이 잘 되어있습니다.
- **카메라 센서**에 대해 이해합니다.
  - 다양한 조명에서 기본적인 카메라 셋팅을 할 수 있어야합니다.
    - [사진 튜토리얼 - 1,2,3편](https://youtu.be/RX9DtIZLHbg?t=189)을 보시면 좋습니다.
  - 다양한 카메라 셋팅 (stereo / RGB-D)등을 이해합니다.
    - [SLAM KR 스터디 발표](https://youtu.be/hKsdfWFAjQY)를 추천합니다.
  - 두개의 2D 이미지로부터 3D 공간을 이해하는 epipolar geometry 방법을 이해합니다.
    - [Stachniss 교수님의 렉처](https://youtu.be/cLeF-KNHgwU)를 추천합니다.

### 이미지 데이터 이해하기

- **이미지 데이터의 특성**에 대해 이해합니다.
- **SLAM에서 왜 grayscale 이미지를 사용하는지** 이해합니다.
  - Grayscale 이미지에서 유용한 정보를 뽑아내는 다양한 방법을 이해합니다.

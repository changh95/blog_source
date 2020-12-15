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

[Harris corner 논문 + 노트필기](https://drive.google.com/file/d/1FQoRwpzDBjhaz63mldfn71ydHF_dztM4/view?usp=sharing)를 첨부합니다.

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

# 
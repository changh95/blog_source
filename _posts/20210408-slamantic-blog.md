---
title: SLAMANTIC – Leveraging semantics to improve VSLAM in dynamic environments (NAVER LABS Europe)
date: 2021-04-08 15:44:28
mathjax: true
tags: 
- NAVER LABS
- SLAMANTIC
- Semantic SLAM
- Visual SLAM
- SLAM
categories: 
- [SLAM, 공부 자료]
excerpt: 2021년 04월 03일 NAVER LABS Europe 블로그 아티클 정리
---

[원본 글 링크](https://europe.naverlabs.com/blog/slamantic-leveraging-semantics-to-improve-vslam-in-dynamic-environments-2/)

네이버랩스 유럽의 블로그를 통해 Visual localization 기술을 공부하다가 SLAMANTIC에 대한 블로그 아티클을 보게 되었다. 서울에서 열린 ICCV 2019 학회에서 이 연구를 처음 보았지만, 당시에는 잘 이해하지 못해서 그냥 '오 딥슬램!' 정도로 생각하고 넘어갔었다.

최근에는 semantic 정보를 정말 잘 활용할 수 있는 SLAM이 필요하겠다라는 생각이 들었는데, 블로그 탐방을 하다가 이 논문을 보게 되면서 다시 관심을 가지게 되었다.

&nbsp;

---

## Visual SLAM

- SLAM은 원래 static한 환경을 전제로 하며, 연속된 측량을 통해 map을 계속 이어가는 방식임.
  - 여러 observation을 기반으로 triangulation을 하며 3D measurement를 만들어감.
- 측량중에 환경이 움직여버리다면 (i.e. 움직이는 물체가 있다면), static한 환경이라는 전제가 깨져버리기 때문에 SLAM이 잘 되지 않음.
  - 움직여버린다면 triangulation이 잘 되지 않음.


&nbsp;

---

## SLAMANTIC

- Semantic segmentation을 통해서 각각의 object class마다 (i.e. car, person, building etc) 생겨난 map point에 confidence measure와 detection consistency를 계산한다.
- 높은 confidence를 가진 point를 이용해서 낮은 confidence를 가진 point에 대해 verification을 수행하고, 최종적으로 pose와 map point를 계산한다.

&nbsp;

---

아 이거도 블로그 글이 너무 짧아서 논문으로 제대로 읽어야겠네
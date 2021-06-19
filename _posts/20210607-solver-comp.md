---
title: Juric 2021 - A Comparison of Graph Optimization Approaches for Pose Estimation in SLAM
date: 2021-06-07 13:50:21
mathjax: true
tags: 
- SLAM
- Visual-SLAM
- Solver
- GTSAM
- Ceres-solver
- g2o
- SE-Sync
categories: 
- [SLAM, 논문 리뷰]
excerpt: Juric 2021 - A Comparison of Graph Optimization Approaches for Pose Estimation in SLAM 논문 리뷰입니다. g2o, ceres-solver, GTSAM, SE-Sync의 성능을 비교합니다.
---

[Juric 2021 - A Comparison of Graph Optimization Approaches for Pose Estimation in SLAM 논문](https://lamor.fer.hr/images/50036607/2021-ajuric-comparison-mipro.pdf)의 리뷰입니다.

&nbsp;

---

## 목적

- 다양한 SLAM solver 프레임워크들의 성능을 비교합니다.
  - g2o, ceres-solver, GTSAM, SE-Sync
  - 이 solver 라이브러리들에 대한 설명은 [Graph-based SLAM 입문 + Solver 프레임워크 소개 (Ceres-solver, g2o, GTSAM, SE-Sync)](https://changh95.github.io/20210607-solvers/) 글 참조.

&nbsp;

---

## 실험환경

- 인텔 8코어 i7-6700HQ. 2.60GHz
  - 16GB 램
  - Ubuntu 20.04
- g2o, ceres-solver, GTSAM은 Levenberg-Marquardt (L-M) solver 사용
- SE-Sync는 Riemannian trust-region 기법 (RTR) 사용 (L-M 미지원)
- 100 iteration만 가능
  - Early stop
    - L-M의 경우 $10^{-5}$ 까지 떨어지면 early-stop.
    - RTR의 경우 $10^{-2}$ 까지 떨어지면 early-stop.

&nbsp;

---

## 데이터셋

- Real-world
  - INTEL, MIT, Garage, Cubicle, Rim
- Simulation
  - M3500 (a,b,c), Sphere-a, Torus, Cube

아래는 각각의 데이터셋이 가지고 있는 node와 edge들 (i.e. constraints들의 수)

{% asset_img "datasets.png" "datasets" %}

&nbsp;

---

## 실험 결과

{% asset_img "result.png" "result" %}

- 빨간색: 제일 빠른 프레임워크
  - SE-Sync가 제일 많은 빨간색을 가짐
- 초록색: 가장 정확한 프레임워크
  - SE-Sync가 제일 많은 초록색을 가짐
- 파란색: 값은 잘 나오지만, 시각화해서 보면 완전 잘못된 결과 (i.e. local minima converge)
  - Ceres와 GTSAM이 파란색이 많은 편

&nbsp;

---

## 결론

- SE-Sync
  - **가장 빠른 프레임워크**
    - 다른 프레임워크에 비해 몇십배~몇백배 빠른 경우도 있었다.
    - 추가적인 verification step을 거치면 global optimality도 얻을 수 있다.
- g2o
  - 제일 오래 걸리는 편
  - 간단한 2D 데이터셋에서는 잘 됨
- Ceres
  - 모든 면에서 딱 중간 - **적당히 빠르고, 적당히 정확함**.
- GTSAM
  - **SE-Sync랑 성능이 동급**
  - 하지만 **노이즈가 섞여있을 때 local minima에 빠짐**.

> **기본적으로 SE-Sync를 쓰는게 제일 좋을듯**. 
> 탄탄한 frontend를 기반으로 **좋은 initial guess가 보장된다면 GTSAM을 사용**해도 무방.
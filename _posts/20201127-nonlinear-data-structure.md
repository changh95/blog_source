---
title: C++ 비선형 자료 구조
date: 2020-11-27 09:25:13
mathjax: true
tags: 
- C++
- Data structures
- STL
categories: 
- [C++, 언어 공부]
- [SW, 알고리즘/자료구조]
excerpt: 비선형 자료 구조 정리
---

## Nonlinear data structure

- `std::vector`, `std::array`, `std::forward_list`, `std::list`, `std::deque` 등과 같은 linear data structure에서는 iterator를 통해 빠르게 데이터를 훑을 수 있었다.
- 비선형 자료구조는 좀 더 복잡한 형태의 데이터를 다룬다.

## Nonlinearity

- Hierarchical problem
  - 각각의 데이터가 계층에 대한 종속 관계를 가진다면 어떻게 할 것인가?
  - e.g. 조직도

![](https://t1.daumcdn.net/cfile/tistory/99A91A3E5D404C2410)

- Cyclic dependency
  - 데이터 끼리의 관계도를 그릴 때 순환되어 돌아오는 구조라면?

![](https://www.researchgate.net/profile/Alberto-Paoluzzi/publication/2402984/figure/fig4/AS:667857369186306@1536241013277/Directed-a-cyclic-graph-used-to-represent-a-set-of-coordinated-activities-Labels-of-arcs.png)
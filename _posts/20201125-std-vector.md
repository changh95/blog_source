---
title: std::vector
date: 2020-11-25 16:16:13
mathjax: true
tags: 
- C++
- Data structures
- STL
categories: 
- [C++, STL]
excerpt: std::vector 정리
---

## std::array랑 다른 점

- `std::array`의 고정된 크기랑은 다르게 dynamic한 크기를 가짐.

## 특징

- 새로운 데이터 원소를 채워넣는데에는 $O(1)$만큼 소요.
- `size`와 `capacity`를 가지고 있음.
  - size는 실제로 데이터가 채워진 메모리의 양.
  - capacity는 현재 vector에 할당된 메모리의 양.
    - capacity가 다 차게 되면, 2*size만큼 새로 capacity를 할당.
    - 새로 할당된 메모리로 기존의 정보를 모두 이동 (i.e. 복사 이동)
    - 이 경우, $O(n)$만큼 소요.
- `insert()`는 $O(n)$만큼 소요.
- 

## 사용 예제

```cpp
std::vector<int> vec; // 빈 vector
std::vector<int> vec = {1,2,3,4,5};
std::vector<int> vec(10); // 크기가 10인 벡터 선언
std::vector<int> vec(10, 0); // 크기가 10이고 0으로 초기화된 벡터 선언

vec.insert(int.begin(), 0); // 중간에 추가
vec.push_back(10); // 맨뒤에 추가
vec.insert(std::find(vec.begin(), vec.end(), 10), 9); // 10 앞에 9 추가.

std::vector<Image> vecImg;
vecImg.emplace(vecImg.begin()+1, img());
vecImg.emplace_back(img()); // emplace_back()은 push_back()과 다르게 복사 연산이 들어가지 않음. 큰 구조체를 넣을 때 constructor와 함께 사용하면 좋음.

vec.pop_back();
vec.erase(vec.begin()+5); // 6번째 원소 지우기
vec.erase(vec.begin() +2, vec.begin() +4) // 2번째 부터 4번째 원소까지 지우기

vec.clear() // 전부 비우기.
vec.reserve(100) // capacity를 100만큼 할당하기.
vec.shrink_to_fit() // 현재 사용중인 메모리에 효율적이도록 빈 메모리를 해제하기.

```
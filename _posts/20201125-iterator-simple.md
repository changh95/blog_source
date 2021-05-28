---
title: Iterators (simple)
date: 2020-11-25 16:25:13
mathjax: true
tags: 
- C++
- Data structures
- STL
categories: 
- [C++, 언어 공부]
- [SW, 알고리즘/자료구조]
excerpt: Iterators 정리
---

## Iterators

{% asset_img "iterators.png" "iterators" %}
- [이미지 출처](https://www.cplusplus.com/reference/iterator/)


- Iterator의 종류는 사용하는 컨테이너에 따라 결정된다.
- `std::vector`, `std::array`는 contiguous data structure이기 때문에 특정 위치의 데이터 원소에 곧바로 접근 가능.
  - Random access iterator
- `std::forward_list`는 역방향이 존재하지 않음.
  - Forward iterator
- `std::list`는 양방향 움직일 수 있음.
  - Bidirectional iterator
- `advance()`, `next()`, `prev()`로 움직일 수 있음.

```cpp

std::vector<int> vec = {1,2,3,4,5};

auto it = vec.begin();
it += 3;
std::advance(it, -2);
std::cout << *it << std::endl;

std::forward_list<int> lst = {1,2,3,4,5};
auto it lst.begin();
// it += 3; // Error, since std::forward_list does not support random access
std::advance(it, 3);
// std::advance(it, -1); // Error, since std::forward_list does not support  backward iteration

```
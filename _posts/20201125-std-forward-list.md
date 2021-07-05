---
title: std::forward_list
date: 2020-11-25 16:18:13
mathjax: true
tags: 
- C++
- Data structures
- STL
categories: 
- [C++, STL]
excerpt: std::forward_list 정리
---

## Contiguous data structure의 단점

- 데이터 중간에 자료를 추가하거나 뺄 때 비효율적.

## std::forward_list 특징

- Linked list 자료구조에 간편한 기능들을 추가한 클래스.
- 최앞단 원소만 바로 접근 가능.
  - `push_front()` 또는 `insert_after()`을 사용해서 $O(1)$으로 추가 가능.
- `remove(x)`를 통해 list 내부에 x라는 값을 가진 원소를 모두 없앰.
  - list 내부에 있는 데이터들이 `==operator`가 안된다면 컴파일러 에러.
- `remove_if(comp)`를 사용해서 list 내부에 comp 함수의 조건을 맞춘 원소를 모두 없앰.
  - 람다 함수를 이용해서 만들면 좋음.
- `remove()`와 `remove_if()`는 전체를 순회하기 때문에 $O(n)$이 걸림.
- `sort()`를 쓸 수 있지만, `std::vector` 등에서 쓰는 sort와 다름.
  - `<operator`를 통해 사용하거나 커스텀 cmp 함수를 통해 사용 가능.
  - $O(n log(n))$ 만큼 소요됨.
- `reverse()`로 순서를 뒤집을 수 있음. $O(n)$

## 사용 예제

```cpp

std::forward_list<int> lst = {1,2,3};

lst.push_front(0); // {0,1,2,3}
lst.insert_after(lst.begin(), 0); // 맨 처음 원소 뒤에 0 추가. {0,0,1,2,3}

lst.pop_front() // {0,1,2,3}
lst.erase_after(lst.begin()); // {1,2,3}
lst.erase_after{lst.begin() + 1, lst_end()} // {1}

std::forward_list<Image> lstImg;
lst.emplace_front(Image());
lst.emplace_after(lst.begin(),Image());

lst.reverse(); // {3,2,1,0}

std::forward_list<int> lst1 = {0,0,0,0,1,2,3};
lst.unique(); // {0,1,2,3}
lst.unique([](int, a, int b){return (b-a) <2;}); // {}. 특정 원소가 바로 앞 원소보다 2 이상 크지 않으면 삭제.

std::forward_list<int> lst2 = {100, 50, 70, 0, -100};
lst2.sort(); // {-100, 0, 50, 70, 100}
lst2.sort(std::greater<int>()); // {100, 70, 50, 0, -100};


```
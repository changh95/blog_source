---
title: Professional C++ - C++ 컨테이너 개요 (Ch17, Part 1)
date: 2021-03-28 15:05:38
mathjax: true
tags: 
- C++
categories: 
- [C++, Professional C++ 책]
excerpt: Marc Gregoire의 "Professional C++"의 챕터17 파트1 정리
---

## Introduction

- STL에 있는 컨테이너를 이용하면 C 스타일 배열을 사용하지 않아도 된다.
- 아래와 같은 컨테이너가 있다.
  - 순차 컨테이너
    - `std::vector`
    - `std::deque`
    - `std::list`
    - `std::forward_list`
    - `std::array`
  - 연관 컨테이너
    - `std::map`
    - `std::multimap`
    - `std::set`
    - `std::multiset`
  - 비정렬 연관 컨테이너 (Hash table)
    - `std::unordered_map`
    - `std::unordered_multimap`
    - `std::unordered_set`
    - `std::unordered_multiset`
  - 컨테이너 어댑터
    - `std::queue`
    - `std::priority_queue`
    - `std::stack`

## 17.1.1 원소에 대한 요구사항

- STL 컨테이너 특징
  - STL 컨테이너는 기본적으로 value semantics (i.e. pass by value)를 통해 원소를 생성한다.
    - 즉, 원소의 복제본을 저장한다.
  - 원소에 대한 레퍼런스나 포인터를 사용할 수도 있다. (i.e. reference semantics, pass by pointer)
    - 컨테이너에 `std::reference_wrapper`를 사용해도 좋다.
    - `std::unique_ptr` 또는 `std::shared_ptr`를 쓰는 것이 좋다.
- Allocator
  - STL 컨테이너는 allocator를 통해 원소에 대한 메모리를 할당하거나 해제할 수 있다.
- Comparator
  - `std::map`과 같은 일부 컨테이너는 comparator를 쓸 수 있다.
- Allocator + Comparator를 사용하는 컨테이너의 원소가 갖춰야할 요구사항은 다음과 같다.
  - Default constructor
  - Copy constructor
  - Move constructor
  - Destructor
  - Assignment operator
  - Move assignment operator
  - Operator ==
  - Operator <





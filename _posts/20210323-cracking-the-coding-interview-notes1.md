---
title: Cracking the coding interview 노트 - 기술면접 준비
date: 2021-03-22 13:38:19
mathjax: true
tags: 
- Cracking the coding interview
- Algorithms
- Data structure
categories: 
- [  SW, 알고리즘/자료구조]
excerpt: Cracking the coding interview 노트 - ch 7 기술면접 준비
---

## 기본적으로 알아야하는 것들

- 자료구조
  - Linked lists
  - Trees, Tries & Graphs
  - Stacks & Queues
  - Heaps
  - Vectors / ArrayLists
  - **Hash tables**
- 알고리즘
  - Breadth-First Search
  - Depth-First Search
  - Binary Search
  - Merge Sort
  - Quick Sort
- 컨셉
  - Bit Manipulation
  - Memory (Stack vs Heap)
  - Recursion
  - Dynamic Programming
  - Big O Time & Space

Practice implementing the data structures and algorithms (first on paper, then on a computer).

&nbsp;

## 문제를 푸는 방법

문제를 풀면서 계속 이야기해야한다. 면접관은 우리가 어떻게 문제에 접근하는지 보고싶어한다.

1. Listen
   - 문제가 정의하는 것들을 자세히 들여다보기.
     - **Mentally record any unique information in the problem**
   - 보통 Optimal code를 작성하기 위해 모든 정보를 사용해야함.
2. Example
   - Example을 하나 만들어보자.
     - 보통 Example은 심플하고, 특수 케이스가 아니여야한다.
     - Example은 specific하고 적당히 커야한다.
   - 디버깅을 해보자.
3. Brute force
   - 최대한 빠르게 brute-force 코드를 작성하자.
   - 효율적인 코드가 아니여도 된다! 일단 간단한걸 만들고 그 다음에 optimise한다.
     - Time & space complexity가 어떻게 되는지 알아보고, 더 좋은 방법을 모색한다.
4. Optimise
   - **BUD Optimisation**을 사용해서 최적화된 코드를 짜자.
     - Bottlenecks
     - Unnecessary Work
     - Duplicated Work
   - 문제에서 쓰지 않은 내용이 있는가 확인하자.
   - Example 케이스에 대해 풀어보자.
     - 풀린다면, 새로운 Example 케이스도 만들어서 풀어보자!
   - 틀린 케이스로 한번 풀어보자. 이 이슈를 해결할 수 있는가?
   - Precomputation이 있는가? (e.g. sorting이 이미 되어있다던지...)
     - 이 정보를 잘 이용하면 훨씬 효율적이게 코드를 짤 수 있다.
   - Time vs space tradeoff를 고려하자. Hash table은 굉장히 좋은 툴이다!
5. Walk through
   - Optimal solution이 만들어졌다면, 한번 머릿속으로 돌려보면서 모든 디테일이 확실하게 커버되는지 확인하자.
   - Whiteboard coding도 좋다!
6. Implement
   - 깔끔한 코드를 작성하자!
   - Modularisation은 필수다.
   - Error check를 까먹지 말자! 필요하다면 TODO로 놔두고, 대신 말로 어떤걸 체크할지 이야기하자.
   - Variable name은 깔끔하게 하자. 
7. Test
   - 'Submit'하기 전에 무조건 테스트는 꼭 하자!
   - Conceptual test. Code review하듯이 코드를 다시 읽자.
   - Non-standard code를 잡아내자.
   - 자주 틀리는 곳들 - recursion, integer divison, binary tree의 null node등을 체크하자.
   - 작은 테스트 케이스를 돌려보자.
   - 특수 케이스 코드를 돌려보자. 

&nbsp;

## 최적화 기법 - BUD 찾기


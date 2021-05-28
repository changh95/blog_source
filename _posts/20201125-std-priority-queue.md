---
title: std::priority_queue (container adapter)
date: 2020-11-25 18:25:13
mathjax: true
tags: 
- C++
- Data structures
- STL
categories: 
- [C++, 언어 공부]
- [SW, 알고리즘/자료구조]
excerpt: std::priority_queue 정리
---

## std::priority_queue

- Priority queue, 또는 Heap이라고도 불림.
- 가장 작거나 (또는 가장 큰) 원소에 빠르게 접근 할 수 있는 자료구조.
- 최소/최대 원소에 접근하는데는 $O(1)$
- 원소 삽입/제거는 $O(log (n))$
  - 원소 제거는 최소/최대 원소에 대해서만 가능.
- `std::vector`를 기본 컨테이너로 사용해서 구현.
- 기본적으로 `std::less`를 사용해서 max heap이 구현되어있음

## 사용 예제

```cpp
// Creates a max heap
priority_queue <int> pq;
pq.push(5);
pq.push(1);
pq.push(10);
pq.push(30);
pq.push(20);

// One by one extract items from max heap
// 30 20 10 5 1 
while (pq.empty() == false)
{
    std::cout << pq.top() << " ";
    pq.pop();
}


// Creates a min heap
priority_queue <int, vector<int>, greater<int> > pq1;
pq1.push(5);
pq1.push(1);
pq1.push(10);
pq1.push(30);
pq1.push(20);

// One by one extract items from min heap
// 1 5 10 20 30 
while (pq1.empty() == false)
{
    std::cout << pq1.top() << " ";
    pq1.pop();
}
```
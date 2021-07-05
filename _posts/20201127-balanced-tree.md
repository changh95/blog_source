---
title: Balanced tree (AVL, red-black tree)
date: 2020-11-27 15:25:13
mathjax: true
tags: 
- C++
- Data structures
- STL
categories: 
- [SW, 알고리즘/자료구조]
excerpt: Balanced tree 구조 정리
---

## BST 의 단점

![](https://qph.fs.quoracdn.net/main-qimg-eccb472654af69385afe691e9958a713.webp)

- 데이터가 sort된 상태에서 가장 작은 숫자를 root node로 만들고, 그 후 작은 순서대로 insert를 하게 된다면, 한쪽으로 치우쳐진 tree가 생길 것이다.
  - 비슷한 예제로, 역방향 sort + 가장 큰 숫자를 root node로 만듬 + 큰 순서대로 insert를 하면 역시 또 한쪽으로 치우쳐진 tree가 생긴다.
- `find()`함수의 시간 복잡도는 $O(height)$이다.
- 그렇기에 tree의 높이는 최적화 되야한다.
  - 이 작업을 'balancing'이라고 한다.
  - 이 작업을 자동으로 하면 'self-balancing'이 된다.
- self-balancing tree의 종류에는 AVL tree와 red-black tree가 있다.
  - `std::map`이 red-black tree 형태로 구현되어있다.
  - 왜 red-black tree로 되어있을까?
    - AVL tree의 rebalancing rotation은 $O(log(n))$
    - Red-black tree의 rebalancing rotation은 $O(1)$이다.
    - ["Why is std::map implemented as a red-black tree?"](https://stackoverflow.com/questions/5288320/why-is-stdmap-implemented-as-a-red-black-tree)

---
title: 대학원 들어가기 전 알아야 할 수학 [0] - 선형대수
date: 2021-01-14 20:38:19
mathjax: true
tags: 
- Maths
- Linear algebra
categories: 
- [Maths]
excerpt: Thomas A. Garrity의 "All the Mathematics You Missed - But Need to Know for Graduate School"의 선형대수 파트 정리입니다.
---

{% asset_img "book.jpg" "The book" %}

케임브리지 연구실에 방문연구원 생활이 끝나면서 기념품을 하나 사고싶었다.
케임브리지 로컬 서점에 들어가서 둘러보는데, 이 책이 눈에 띄였고 그 자리에서 바로 구매했다.
내 책의 뒷면에 £30 이라는 가격표와 CAMBRIDGE UNIVERSITY PRESS라고 적힌걸 보면... 아마 그 서점은 케임브리지 출판사 전용 서점이였나보다 ㅋㅋ
어쨋든 이 기념품은 내 책장에 몇년동안 그대로 방치되었다가, 지난 주 쯤 페이스북에서 지인이 이 책에 대해 질문을 하시길래 다시 흥미가 생겨 읽게되었다.

아직 첫장밖에 읽지 않았지만, 이 책에는 두가지 특성이 있는 것 같다.
- **수학을 학문으로써 공부하는 학생 입장**을 잘 고려한다.
  - 그만큼 응용분야의 입장은 크게 고려하지 않는다 (e.g. 머신러닝, 컴퓨터 비전...)
- 개별적인 테크닉이나 방법론보다는, 최대한 그 **학문의 본질을 파악**할 수 있게 한다.

첫장은 선형대수였고, 이 책은 '선형대수는 무엇을 하는 학문인가?' 라는 질문을 던진다.

<br>

---

# 선형대수의 본질

**선형대수의 본질**은 **System of linear equations 문제를 푸는 것**이다.
그리고 우리는 이 문제를 풀기 위해 matrix를 이런저런 방법으로 다루는 방법을 공부한다.

조금 과장하자면, 모든 수학문제는 선형대수 문제로 변환할 수 있어야만 풀 수 있다. (라고 한다)
선형대수는 어떤 수학적인 특성을 공유하는 object들을 하나의 vector space로 다룰 수 있게 해준다.
또, linear transformation을 통해 이 vector space들간의 개념들을 잇는 관계들을 표현할 수 있다.

선형대수로 우리는 **n개의 linear equation들과 n개의 unknown이 있을 때**, **solution이 있는지 없는지** 알아낼 수 있는 다양한 방법들을 제공한다.

<br>

---

# Vector space

- 가장 본질적인 vector space는 $\mathbb{R}^n$로써 모든 real number의 집합이다.
$$ \{ (x_1,...,x_n): x_i \in \mathbb{R} \} $$

- $\mathbb{R}^n$은 왜 vector space일까?
  - $\mathbb{R}^n$에 속하는 두개의 값을 더했을 때 $\mathbb{R}^n$에 속하는 다른 값이 나온다.
    - $ (x_1, ... , x_n) + (y_1, ... , y_n) = (x_1 + y_1, ... , x_n + y_n)  $
  - $\mathbb{R}^n$에 속하는 값과 scalar 값을 곱했을 때 $\mathbb{R}^n$에 속하는 다른 값이 나온다.
    - $ \lambda(x_1, ..., x_n) = (\lambda x_1, ..., \lambda x_n) $

<br>

- $\mathbb{R}^n$ 에서 $\mathbb{R}^m$으로의 변환 관계, 즉 n차원에서 m차원으로의 맵핑은 매트릭스로 표현할 수 있다.

$$ A=\left(\begin{array}{cccc}a_{11} & a_{12} & \ldots & a_{1 n} \\\\ \vdots & \vdots & \vdots & \vdots \\\\ a_{m 1} & \ldots & \ldots & a_{m n}\end{array}\right) $$

$$
A \mathbf{x}=\left(\begin{array}{cccc}
a_{11} & a_{12} & \ldots & a_{1 n} \\\\
\vdots & \vdots & \vdots & \vdots \\\\
a_{m 1} & \ldots & \ldots & a_{m n}
\end{array}\right)\left(\begin{array}{c}
x_{1} \\\\
\vdots \\\\
x_{n}
\end{array}\right)=\left(\begin{array}{c}
a_{11} x_{1}+\ldots+a_{1 n} x_{n} \\\\
\vdots \\\\
a_{m 1} x_{1}+\ldots+a_{m n} x_{n}
\end{array}\right)
$$


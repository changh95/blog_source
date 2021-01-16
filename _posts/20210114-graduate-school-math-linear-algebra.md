---
title: 대학원 들어가기 전 알아야 할 수학 [1] - 선형대수
date: 2021-01-14 20:38:19
mathjax: true
tags: 
- Maths
- Linear algebra
categories: 
- [Maths, 대학원 들어가기 전 알아야할 수학 시리즈]
excerpt: Thomas A. Garrity의 "All the Mathematics You Missed - But Need to Know for Graduate School"의 선형대수 파트 정리입니다.
---

첫장은 선형대수였고, 이 책은 '선형대수는 무엇을 하는 학문인가?' 라는 질문을 던진다.

<br>

---

# 선형대수의 본질

{% asset_img "systems.png" "Systems of linear equations" %}

<br>

**선형대수의 본질**은 **System of linear equations 문제를 푸는 것**이다.
그리고 이 문제를 풀기위해 다양한 matrix를 다루는 방법들을 공부한다.

조금 과장하자면, 모든 수학문제는 선형대수 문제로 변환할 수 있어야만 풀 수 있다. (라고 한다)
선형대수를 통해 공통된 특성을 공유하는 object들의 관계를 하나의 vector space로 다룰 수 있게 해준다.
또 이 vector space들 사이의 관계를 linear transformation으로 표현할 수 있다.

선형대수는 **n개의 linear equation들과 n개의 unknown이 있을 때**, **solution이 있는지 없는지** 알아낼 수 있는 다양한 방법들을 제공한다.

<br>

---

# 가장 기초적인 Vector space: $\mathbb{R}^n$

- 가장 본질적인 vector space는 $\mathbb{R}^n$로써 모든 real number의 집합이다.
$$ \\{(x_1,...,x_n): x_i \in \mathbb{R}\\} $$

<br>

- $\mathbb{R}^n$은 왜 vector space일까?
  - **Vector addition**이 가능하다.
    - 즉, $\mathbb{R}^n$에 속하는 두개의 값을 더했을 때 $\mathbb{R}^n$에 속하는 다른 값이 나온다.
      - $ (x_1, ... , x_n) + (y_1, ... , y_n) = (x_1 + y_1, ... , x_n + y_n)  $
  - **Scalar multiplication**이 가능하다.
    - 즉, $\mathbb{R}^n$에 속하는 값과 scalar ($\lambda$) 값을 곱했을 때 $\mathbb{R}^n$에 속하는 다른 값이 나온다.
      - $ \lambda(x_1, ..., x_n) = (\lambda x_1, ..., \lambda x_n) $

<br>

- $\mathbb{R}^n$는 n차원의 [real number (i.e. 정수)](https://ko.wikipedia.org/wiki/정수)를 가진 벡터를 의미한다.
  - 차원의 수를 늘리거나 줄일 수 있을까?
    - $\mathbb{R}^n$ 에서 $\mathbb{R}^m$으로의 변환 관계, 즉 **n차원에서 m차원으로의 맵핑은 매트릭스로 표현**할 수 있다.

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

<br>

- 아래와 같은 **system of linear equations**이 있다고 해보자.
  - $ b_1 , ... , b_m $과 $ a_{11} , ... , a_{mn} $이 주어졌고, 우리는 $ x_1 , ... x_n $을 구하고싶다고 해보자.

{% asset_img "systems.png" "Systems of linear equations" %}

- 많은 선형대수 문제는 위와 같은 system of linear equations 문제로 요약될 수 있다.
  - 적은 수의 equation만 있다면 [**Gaussian elimination (i.e. 가우스 소거법)**](https://ko.wikipedia.org/wiki/가우스_소거법) 등의 방식을 통해 손으로 계산할 수 있다.
  - 하지만 **많은 수의 equation + unknowns를 풀어야한다면 계산량이 무지막지하게 많아진다**.
    - 이론적으로 어렵지 않으나, 수많은 숫자들을 트랙킹하는게 골치아프다...
- 우선 위와 같은 형태의 문제들을 아래와 같은 방식으로 요약할 수 있다.
  - $x$와 $b$를 벡터화, $A$를 매트릭스화 한다.
  - 그러면 문제를 $\mathbf{Ax = b}$와 같이 보기 편한 형태로 바꿀 수 있다.

<br>

$$ Ax = b $$

$$\quad A=\left(\begin{array}{cccc}a_{11} & a_{12} & \ldots & a_{1 n} \\\\ \vdots & \vdots & \vdots & \vdots \\\\ a_{m 1} & \ldots & \ldots & a_{m n}\end{array}\right), \mathrm{x}=\left(\begin{array}{c}x_{1} \\\\ \vdots \\\\ x_{n}\end{array}\right), \mathrm{b}=\left(\begin{array}{c}b_{1} \\\\ \vdots \\\\ b_{m}\end{array}\right)$$

<br>

- $m > n$일 때 (i.e. equation의 수가 unknown의 수 보다 많을 때) unknown은 구할 수 없다 (i.e. **No solution**).
  - [Over-determined](https://en.wikipedia.org/wiki/Overdetermined_system) 문제라고 하며, 이 경우는 least squares optimisation을 통해 최소 에러값을 가지는 Unknown value를 구할 수 있다.
  - 컴퓨터 비전에서 자주 나타나는 문제이다.
- $m < n$일 때 (i.e. unknown의 수가 equation의 수 보다 많을 때)는 **무한히 많은 solution**이 존재한다.
  - [Under-determined](https://en.wikipedia.org/wiki/Underdetermined_system) 문제라고 한다.
- **선형대수에서 가장 많이 집중하는 케이스**는 $m = n$인 상황이다.
  - 이 경우 $A$는 $n \times n$ 매트릭스의 형태를 띄우겠고, $Ax = b$ 문제를 풀어서 $(x_1, ..., x_n)$의 값을 알아내려고 한다.
  - x를 풀기 위해서는 $x = A^{-1}b$가 성립되어야한다.
    - 이를 위해서는 A의 inverse, 즉 $A^{-1}$가 존재해야한다.
    - **선형대수의 상당한 부분은 $A^{-1}$가 존재하는지에 대해 알아내는 방법**을 포함한다.

---

# Vector spaces와 Linear Transformation

## Vector space의 정의

- $V$라는 set은 아래와 같은 맵핑 조건을 충족한다면 정수에 대한 vector space가 될 수 있다.
  - $\mathbb{R} \times V \rightarrow V$
    - 어떠한 정수와 어떠한 $V$의 Element를 곱했을 때 나오는 결과가 $V$ set에 포함되어있는 경우
  - $V \times V \rightarrow V$
    - $V$ set에 포함되어있는 두 element를 더했을 때 나오는 결과가 $V$ set에 포함되어있는 경우

<br>

- Vector space $V$의 element는 편히 **vector**라고 부른다.
- Vector space가 존재하는 정수 element는 **scalar**라고 부른다.
  - 사실 정수 뿐만이 아니라 다른 수의 체계 (e.g. 정수, 허수, 자연수...)도 사용 가능하다.

<br>

- 이 때, 아래의 추가조건을 달성해야한다.
  - 0 vector 가 있어야한다 (i.e. 모든 vector에 대해서 $0 + v = v$ 라는 계산이 가능해야한다). (i.e. **Null space**)
  - 모든 vector에 대해 상응하는 역수가 있어야한다 (i.e. $v + (-v) = 0$이 가능해야한다). (i.e. **Inverse element**)
  - 모든 vector에 대해 $v + w = w + v$가 성립해야한다. (i.e. **Commutative**)
  - 모든 vector에 대해 상응하는 항등수가 있어야한다. (i.e. $1 \cdot v = v$) (i.e. **Neutral element**)
  - 모든 vector와 어떠한 scalar 값 $a$에 대해, $a(v+w) = av + aw$가 성립해야한다 (i.e. **Distributivity**)
  - 모든 vector와 어떠한 두개의 scalar 값 $a$ $b$에 대해, $a(bv) = (a \cdot b)v$가 성립해야한다 (i.e. **Scalar associativity**).
  - 모든 vector와 어떠한 두개의 scalar 값 $a$ $b$에 대해, $(a+b)v = av + bv$가 성립해야한다 (i.e. **Scalar distributivity**)

<br>

## Linear transformation 정의

- **Vector space간의 맵핑 관계는 Linear transformation으로 표현된다**.
- $V$ vector space에서 $W$ vector space로의 변환 관계를 표현하는 함수인 T가 있다.
  - 수식으로는 $T:V \rightarrow W$ 로 표현된다.
  - T가 linear transformation이기 위해선 $T(a_1 v_1 + a_2 v_2) = a_1T(v_1) + a_2T(v_2)$ 조건이 충족되어야한다.

<br>

## Vector subspace 정의

- Vector space $V$에 속한 vector subspace $U$도 하나의 vector space이다.
- Vector subspace의 존재함에는 몇가지 조건이 충족되어야한다.
  - $U$ vector subspace 내부에서도 **addition 및 scalar multiplication에 대한 closure**가 있어야한다.
    - 즉, $U$ 내부의 모든 vector 사이의 addition 및 scalar multiplication 연산은 $U$ 내부의 또다른 vector가 되어야한다는 것이다.

<br>

## Linear transformation과 Vector subspace의 관계

- $V$ 에서 $W$로의 linear transformation $T$가 있다. (i.e. $T:V \rightarrow W$)
  - $T$의 kernel은 $ker(T) = \\{ v \in V : T(v)  = 0 \\}$이다
  - $T$의 image는 $Im(T) = \\{ w \in W : \text{ there exists a } v \in V \text{ with } T(v) = w \\}$

{% asset_img "kernel_image.png" "Kernel and image" %}

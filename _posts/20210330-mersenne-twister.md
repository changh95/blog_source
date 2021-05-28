---
title: 메르센 트위스터 (std::mt19937 / std::mersenne_twister_engine)
date: 2021-03-30 00:17:11
mathjax: true
tags: 
- C++
- SW
- Random number generator
categories: 
- [C++, 언어 공부]
excerpt: C++ STL에서 사용할 수 있는 random number generator 중 하나인 Mersenne twister에 대해 알아봅니다.
---

> std::mt19337과 std::mersenne_twister_engine은 C++에서 제공하는 random number generator이다. 기존에 많이 사용되던 rand의 대체제로 쓸 수 있다.

&nbsp;

---

## 왜 rand 대신 써야하는가?

rand의 문제점은 아래와 같다.

- 생성된 random number의 분포가 그리 고르지 않다.
- 반복 주기는 $2^{32}$로 다른 pseudo random number generator 에 비해 상대적으로 짧다.
- 전역 함수이기 때문에 프로그램 전체에서 시드를 공유한다.

시뮬레이션 계산을 할 때 random number에 bias가 있다면 그 시뮬레이션 계산은 믿을 수 없다.

Mersenne Twister는 pseudorandom number generator로써 rand의  $2^{32}$ 보다 **훨씬 긴 $2^{19937} -1$의 주기**를 가지고 있다. Pseudorandom number generator이기 때문에 완전 random한 결과를 주는 것은 아니지만, 주기가 굉장히 길기 때문에 작은 샘플링 수에서는 충분히 random하다고 판단할 수 있다 (긴 주기가 random-ness를 보장하진 않지만, 해당 테스트에서 rand의 $2^{32}$ 주기는 random의 형태가 무너지는 문제가 나타난다). 

이에 대한 증거로 Diehard 테스트와 다수의 TestU01 테스트를 통과했다.

&nbsp;

---

## Mersenne Twister는 무엇인가?

![https://upload.wikimedia.org/wikipedia/commons/b/b5/Mersenne_Twister_visualisation.svg](https://upload.wikimedia.org/wikipedia/commons/b/b5/Mersenne_Twister_visualisation.svg)

많은 휴리스틱 알고리즘을 이용해서 random number를 생성한다.

주기가 $2^{19937} -1$ 인 이유는 624개의 integer (32-bit) ⇒ $624 * 32 = 19936$ 이기 때문이다.

C++에는 <random>에 구현되어있으며 아래와 같은 구성을 가지고 있다.

- 커스텀 Mersenne Twister를 만들 때 사용되는 std::mersenne_twister_engine

```cpp
template<
    class UIntType,
    size_t w, size_t n, size_t m, size_t r,
    UIntType a, size_t u, UIntType d, size_t s,
    UIntType b, size_t t,
    UIntType c, size_t l, UIntType f
> class mersenne_twister_engine;
```

- 1998년도에 나온 32-bit MT를 구현하는 std::mt19937

```cpp
typedef mersenne_twister_engine<unsigned int, 32, 624, 397, 31, 0x9908b0df,
    11, 0xffffffff, 7, 0x9d2c5680, 15, 0xefc60000, 18, 1812433253> mt19937;
```

- 2000년도에 나온 64-bit MT를 구현하는 std::mt19937_64

```cpp
typedef mersenne_twister_engine<unsigned long long, 64, 312, 156, 31,
    0xb5026f5aa96619e9ULL, 29,
    0x5555555555555555ULL, 17,
    0x71d67fffeda60000ULL, 37,
    0xfff7eee000000000ULL, 43,
    6364136223846793005ULL> mt19937_64;
```

&nbsp;

> 이론에 관심 있으신 분들은 오리지널 논문을 참조바랍니다. 
> ["Mersenne twister: a 623-dimensionally equidistributed uniform pseudo-random number generator"](https://dl.acm.org/doi/10.1145/272991.272995)

&nbsp;

---

## Mersenne Twister의 장점

- C++에 이미 구현되어있고 라이센스가 풀려있다.
- Stats 테스트를 많이 통과했다 (e.g. Diehard test, many of TestU01 test)
- $2^{19937} -1$ 의 주기로 왠만하면 random-ness를 유지해준다.
- distribution이 일정한 편이다.
- 빠르다! → 2017년에 나온 논문에 따르면 True random 방식의 SIMD 구현보다 20배 빠르게 64-bit floating point random number를 생성한다고 한다.

## Mersenne Twister의 단점

- State vector가 큰 편이다 (2.5 kB)
  - 임베디드에서 문제가 될 수 있는데, 이는 TinyMT를 사용해서 해결할 수도 있다.
  - [Tiny Mersenne Twister (TinyMT)](http://www.math.sci.hiroshima-u.ac.jp/~m-mat/MT/TINYMT/index.html)
- SIMD 버전을 쓰지 않으면 다른 방법들에 비해서 빠른건 아니다.
  - SIMD 버전은 SFMT가 2배 더 빠르다고 한다. 하지만 Intel SSE2가 있어야한다.
  - [SIMD-oriented Fast Mersenne Twister (SFMT)](http://www.math.sci.hiroshima-u.ac.jp/~m-mat/MT/SFMT/)
- 암호학적으로 안전한 방법이 아니다. std::mt19937의 경우, 624개의 random number generation 값을 보면, 추후 어떤 값들이 나올지 전부 추측이 가능하다.
  - 하지만 실험용으로는 전혀 문제없다 😈

&nbsp;

---

## 내가 사용하는 방법

```cpp
#include <random>

std::random_device randDev;
std::mt19937 mtGenerator(randDev());

// ... //

std::vector<int32_t> someData;
std::shuffle(someData.begin(), someData.end(), mtGenerator); // 랜덤 셔플

```

mt19937을 이용해서 랜덤 데이터 셔플을 한다던지, RANSAC을 돌린다던지, random seed 를 만든다던지 할 때 쓰면 좋다.

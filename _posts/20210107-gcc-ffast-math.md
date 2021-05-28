---
title: C++ 프로그램 속도를 빠르게 만들어주는 gcc 컴파일러 플래그
date: 2021-01-07 20:38:19
mathjax: true
tags: 
- C++
- gcc
categories: 
- [C++, 개발]
excerpt: ffast-math, -fassociative-math, -Ofast 플래그에 대해 알아보자!
---

C++로 프로그램을 만드는 이유 중 하나는 **빠른 실행 속도**이다.
gcc 컴파일러를 사용해서 C++ 프로그램을 컴파일 할 때, 평소보다 **훨씬 더 빠르게 프로그램이 작동하도록 컴파일하는 방법**이 있다.
[온라인 도큐먼트에 다양한 컴파일 옵션](http://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)들이 잘 설명되어있다. 

<br>

{% asset_img "tldr.gif" "tldr" %}

**근데 이걸 언제 다 읽어!**
간단하게 중요한 몇개의 컴파일 플래그만 알아보도록 한다.

--- 

# ffast-math 플래그

`ffast-math`는 기존의 IEEE에서 정의한 C++ 컴파일 방식을 따르지 않고, **더 효율적인 방식으로 프로그램을 컴파일**한다.

예를 들면,

```cpp
x = x*x*x*x*x*x*x*x;
```
와 같은 코드가 있다고 해보자.
이 방식은 7 번의 곱셈 계산이 필요할 것이다.

<br>

``` cpp
x *= x;
x *= x;
x *= x;
```

`ffast-math` 플래그를 사용하면 위 연산은 이렇게 바뀐다.
동일한 연산을 3번의 곱셈 연산으로 바꾸었다.
이론적으로, 이 연산은 약 **7/3배 빠르다**.

<br>

## 사용할 때 유의할 점 1

[Goldberg 논문](https://dl.acm.org/doi/10.1145/103162.103163)에 나온 것 처럼 floating-point의 계산은 associative하지 않다.
그러므로 **`ffast-math` 연산 방식에서는 실제 값에 오류를 포함**할 수 밖에 없다.
이러한 점 때문에 `ffast-math` 방식은 IEEE에서 정의한 방식을 따르지 못한다.

위와 같은 특징 때문에, 정확한 값을 계산해야하는 것이라면 `ffast-math`를 사용하면 안된다.
**하지만 대충 어림잡아서 맞는 값을 원하는 것이라면?**
예를 들어서 계산 값이 10.1923845 mm가 나오는데, 우리가 원하는게 대충 10.2 mm 정도인지 아는 것으로 충분한 것이라면 그 뒤 소수점으로는 오류가 있어도 상관없다.
이런 경우에는 `ffast-math`가 효과적일 것이다.

**실시간 컴퓨터 비전에서는 이 방식이 효과적**일 수 있다.
**어차피 센서 값에서 오류를 포함**하고 있을테니, 정도껏만 정확하기만 해도 충분하기 때문이다.

<br>

## 사용할 때 유의할 점 2

사실 위의 것 외로도 다른 주의할 점들이 있다.
`ffast-math` 플래그는 아래와 같은 컴파일 옵션들을 사용한다.

- fno-math-errno
- funsafe-math-optimizations 
- ffinite-math-only
- fno-rounding-math
- fno-signaling-nans
- fcx-limited-range
- fexcess-precision=fast

이 각각의 옵션들의 한계를 잘 확인하고, 내 코드에서 사용할 수 있는지 확인해봐야한다.

예를 들어, `ffinite-math-only` 플래그와 `fno-signaling-nans` 플래그는 `NaN`을 사용하는 연산은 사용할 수 없게 만든다.

---

# fassociative-math 플래그

`fassociative-math` 플래그는 **`ffast-math`의 순한맛 버전**...?의 느낌이다.

`ffast-math`에서 보여준 arithmetic re-ordering 예시를 그대로 수행한다.
이 때문에 NaN과 관련된 문제도 나타나게 된다.
하지만 이것 외로는 다른 제약은 없다.

내 코드의 성능을 올리기 위해, `fassociatve-math` 플래그와 `ffast-math` 플래그를 하나씩 시도해보는 것도 좋다고 생각한다.

---

# Ofast 플래그

**가장 강력한 최적화 플래그**이다.
IEEE고 정확도고 뭐고 다 비켜!!!!!!
속도가 짱짱이시다!!! 같은 느낌...

{% asset_img "ggf.png" "gotta go fest" %}

`ffast-math`와 `fallow-store-data-races` 플래그를 포함하기 때문에, IEEE 규칙을 무시한 것은 물론이고 `ffast-math`에서 주의해야할 점들도 모두 가지고 있다.
그 외로, 아래에 보이는 것 처럼 control flow 관련 최적화 플래그도 모두 포함한다.

- fgcse-after-reload 
- fipa-cp-clone
- floop-interchange 
- floop-unroll-and-jam 
- fpeel-loops 
- fpredictive-commoning 
- fsplit-loops 
- fsplit-paths 
- ftree-loop-distribution 
- ftree-loop-vectorize 
- ftree-partial-pre 
- ftree-slp-vectorize 
- funswitch-loops 
- fvect-cost-model 
- fvect-cost-model=dynamic 
- fversion-loops-for-strides

---

# 결론

내가 짠 **코드의 성능을 단 하나의 플래그로 뻠핑하고싶다**면, 위의 플래그들을 사용해보면 좋다. 
순서대로 **`fassociative-math` -> `ffast-math` -> `Ofast` 를 시도해보자**.
근데 안될수도 있으니 테스트를 잘 해보자.
---
title: (번역) 실시간 컴퓨터 비전을 위한 C++ 최적화 (12) - 필요없는 조건문은 없애줍시다!
date: 2020-12-16 20:38:19
tags: 
- C++
- CV
- Coding Interviews
- Optimisation
categories: 
- [Programming, C++, 실시간 컴퓨터 비전을 위한 C++ 최적화 시리즈]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "Having fun with Palindrome words`"을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "Case study: filter a Point Cloud faster"을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/palindrome/)를 봐주세요.

<br>

이번 글은 다른 글들에 비해서 컴퓨터 비전과 크게 관련은 없을 수 있습니다.

그래도 적는 이유는... 조건 분기문을 없앰으로써 얼마나 코드가 빨라질 수 있는지 보여줄 수 있는 좋은 예시라고 생각되서입니다.

`if`문은 보통 엄청나게 빠르기 때문에 런타임 cost를 보통 생각하지 않아도 됩니다. 하지만 종종 특수 케이스에서는 실제로 눈에 보일만큼 성능 향상이 나타날 때도 있습니다.

<br>

## 코딩 인터뷰
---

제가 이 글을 적고 있었을 때는 (역자 코멘트: 제가 아니라, 원글의 저자입니다!), 저는 엄청 활발하게 이직할 회사를 찾고 있었습니다. 그러다보니 자연스럽게 코딩 인터뷰도 보게 되었지요.

코딩 인터뷰는 왠만하면 할만합니다.

언젠가 하루는 이런 질문을 받았습니다.

> String이 [palindrome](https://en.wikipedia.org/wiki/Palindrome)인지 확인할 수 있는 함수를 짜주시겠어요? (역자 코멘트: palindrome은 거꾸로 뒤집어도 똑같은 단어를 의미합니다. [로꾸거](https://youtu.be/vJfYJhtdcKE) 를 참조하세요.)

자연스럽게 이런 생각이 들더군요

{% asset_img "fizzbuzz.jpg" "fizzbuzz" %}

나랑 말장난하자는거군!

이런 질문을 하는게 잘못된건 아닙니다 ㅎㅎ. 아이스 브레이킹 용도로 딱 좋은 질문인 것 같아요. 하지만 저는 좀 더 real-world code를 적는 걸 예상했는데 말이죠.

어찌되었건, 이렇게 답을 했습니다.

```C++

bool IsPalindrome(const std::string& str)
{
    const size_t N = str.size();
    const size_t N_half = N / 2;
    for(size_t i=0; i<N_half; i++)
    {
        if( str[i] != str[N-1-i])
        {
            return false;
        }
    }
    return true;
}
```

<br>

이 정도 문제는 껌이지요 :)

너무 쉽습니다!

여기에 최적화를 끼얹어서 재미를 좀 봅시다!

<br>

## 더 빠른 IsPalindrome()
---

오리지널 버전도 나쁘진 않았습니다.

- 복사가 없었구요
- 찾는 즉시 loop를 멈췄구요
- 모든 코너 케이스에 대해 대응할 수 있었습니다.

하지만 여기서 더 빠르게 만들 수 있는 방법이 떠올랐습니다. `if`가 반복되는 수를 줄일 수 있죠.

한 글자씩 비교하는게 아니라, 'word'끼리 비교를 하는겁니다.

`uint32_t`에 word를 저장해봅시다. `uint32_t`는 4바이트를 한번에 다룰 수 있습니다.

구현하면 아래 코드처럼 되겠습니다.

```C++
#include <byteswap.h>

inline bool IsPalindromeWord(const std::string& str)
{
    const size_t N = str.size();
    const size_t N_half = (N/2);
    const size_t S = sizeof(uint32_t);
    // number of words of size S in N_half
    const size_t N_words = (N_half / S);

    // example: if N = 18, half string is 9 bytes and
    // we need to compair 2 pairs of words and 1 pair of chars

    size_t index = 0;

    for(size_t i=0; i<N_words; i++)
    {
        uint32_t word_left, word_right;
        memcpy(&word_left, &str[index], S);
        memcpy(&word_right, &str[N - S - index], S);

        if( word_left != bswap_32(word_right))
        {
            return false;
        }
        index += S;
    }
    // remaining bytes.
    while(index < N_half)
    {
        if( str[index] != str[N-1-index])
        {
            return false;
        }
        index++;
    }
    return true;
}
```

<br>

여기서 우리는 string을 4바이트의 '블럭'으로 쪼갤겁니다. 좌,우 순서대로 쪼갤거고, 중간에 남는 값들은 그대로 str로 둘겁니다.

블럭끼리 비교는 ***단 한번***만 하면 됩니다.

뒤에서오는 블럭은 순서가 거꾸로 뒤집어져야 합니다. 우리는 이 방법을 위해 STL에 있는 [bswap_32](https://man7.org/linux/man-pages/man3/bswap_32.3.html)라는 방식을 사용하겠습니다.  

<br>

```C++
inline uint32_t Swap(const uint32_t& val)
{
  union {
    char c[4];
    uint32_t n;
  } data;
  data.n = val;
  std::swap(data.c[0], data.c[3]);
  std::swap(data.c[1], data.c[2]);
  return data.n;
}
```

<br>

# 벤치마크
---

{% asset_img "palindrome_benchmark.png" "벤치마크" %}

긴 string을 사용했을 때에 (8바이트보다 컸을 때) 성능은 약 ***50%*** 정도, 그리고 짧은 string에서는 약 ***150***% 정도 성능 향상이 있었습니다.

실제로 엄청나게 긴 string의 경우에는, 128이나 256 비트를 사용할수도 있겠습니다. 이 경우에는 [SIMD](https://stackoverflow.blog/2020/07/08/improving-performance-with-simd-intrinsics-in-three-use-cases/)를 사용할 수 있겠네요. 하지만 SIMD는 이번 글의 목적이 아닙니다.

# 정리하자면...

이번 글에서 나온 코드는 '심플하고 읽기 좋은 코드를 적고, 최적화에 너무 집중하지 마세요'라는 말의 완전한 반대를 실천합니다.

이런 경우가 사실 좋다고 할 수는 없습니다. 가독성 좋고 유지보수가 가능한 코드를 짜는 것은 중요하지요.

여기 나온 예제는 대체적으로 'for fun'용도로 짠 것이고, 생각하기 어려운 ***특정 케이스***에서도 최적화의 가능성이 있다는 것을 보여줍니다.

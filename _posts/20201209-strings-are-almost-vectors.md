---
title: (번역) 실시간 컴퓨터 비전을 위한 C++ 최적화 (5) - String은 vector처럼 써야합니다
date: 2020-12-09 20:38:19
tags: 
- C++
- CV
- Optimisation
categories: 
- [SLAM]
- [C++]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "Strings are (almost) vectors"을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "Strings are (almost) vectors"을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/strings_are_vectors/)를 봐주세요.

<br>

# String에 대해 진지하게 고민할 필요가 있나요?
---

`std::string`은 엄청나게 좋습니다. ***C***언어에서 사용하는 raw pointer와 길이 값과 코드를 짜면서 생기는 온간 세상 근심걱정에 비해서요.

농담입니다. C 개발자들, 사랑해요!
> ... 하지만 그래도 어느정도 불쌍하다고 생각이 드는건 어쩔 수 없습니다.

생각해보면, string은 사실 `std::vector<char>`랑 목적 면에서 크게 다른 점이 없습니다. 그냥 `std::string`에는 몇가지 쓰기 좋은 툴이 더 있다는 것만 다른 점이겠죠.

하지만 우리가 생각하지 못했던 ***또 다른 점***이 있습니다. "Small string Optimisation (SSO)"가 가능하다는 점이죠.

SSO에 대해서 [여기](https://changh95.github.io/20201210-small-string-optimisation/) 글에 적어두었습니다.

제가 말씀드리고 싶은 점은, `std::string`도 다른 memory allocation이 필요할 수도 있는 자료구조를 다루는 것 처럼 비슷한 방식으로 다룰 수 있어야 효율적으로 사용할 수 있다는 겁니다.

<br>

## ToString
---

```c++
enum Color{
    BLUE,
    RED,
    YELLOW
};

std::string ToStringBad(Color c)
{
    switch(c) {
    case BLUE:   return "BLUE";
    case RED:    return "RED";
    case YELLOW: return "YELLOW";
    }
}

const std::string& ToStringBetter(Color c)
{
    static const std::string color_name[3] ={"BLUE", "RED", "YELLOW"};
    switch(c) {
    case BLUE:   return color_name[0];
    case RED:    return color_name[1];
    case YELLOW: return color_name[2];
    }
}
```

여기 예제에서 보이는 것 처럼, 가능하다면 최대한 새로운 string을 만들고 또 만드는 행위는 피해야합니다. 

{% asset_img "tostring.png" "To String..." %}

<br>

## Temp string을 재사용합시다
---


아래 예시를 보시면 이전에 할당한 메모리를 재사용 할 수***도*** 있는 예시를 보실 수 있습니다. 

두번째 함수가 첫번째 함수보다 항상 빠를 것이라고 단정할 수는 없습니다. 하지만 왠만하면 빠르지 않을까 싶어요.

```c++
// 첫번째 방법은 매번 새로운 string을 만드는겁니다.
static std::string ModifyString(const std::string& input)
{
    std::string output = input;
    output.append("... indeed");
    return output;
}
// 이미 할당한 string을 다시 사용합니다. 이 string이 우리가 필요한 메모리 공간을 이미 충분히 가지고 있길 빌어야죠. 
// (아닐 수도 있구요... 그러면 느릴수도 있어요)
static void ModifyStringBetter(const std::string& input, std::string& output)
{
    output = input;
    output.append("... indeed");
}
```

벤치마크를 돌렸더니, 예상한대로 나왔습니다.

{% asset_img "modifystring.png" "Modify string" %}

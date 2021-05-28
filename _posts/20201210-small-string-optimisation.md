---
title: (번역) 실시간 컴퓨터 비전을 위한 C++ 최적화 (6) - String 만들어서 쓰기 vs String 내용 복사해서 쓰기
date: 2020-12-10 20:38:19
tags: 
- C++
- CV
- Optimisation
categories: 
- [C++, 실시간 컴퓨터 비전을 위한 C++ 최적화 시리즈]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "Small string optimization"을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "Small string optimizations"을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/small_strings/)를 봐주세요.

<br>

# Small String 최적화 (small string optimisation - SSO)
---

제가 "`std::string`은 결국 `std::vector<char>`와 다를게 많이 없다고 했던 것을 기억하시나요.

훌륭하신 분들께서, 이미 할당된 메모리 안에서 string을 사용하는 방법을 고안하셨습니다. 

`std::string`의 크기는 64-bit 플랫폼에서 ***24 바이트***입니다 (데이터 포인터, size, capacity 등을 저장합니다). 근데 여기서 우리가 메모리를 할당하기 전 까지 ***'static'***하게 23바이트의 메모리를 저장하는 방법이 있어요.

이 방법을 사용하면 성능이 엄청나게 좋아집니다!

{% asset_img "relax_sso.jpg" "Relax, SSO!" %}

이 방법을 **Small String Optimisation (SSO)** 라고 합니다. 이 방법의 구현 방법이 궁금하신 분들을 위해 링크를 준비해놨습니다.

- [SSO-23](https://github.com/elliotgoodrich/SSO-23)
- [CppCon 2016: “The strange details of std::string at Facebook"](https://www.youtube.com/watch?v=kPR8h4-qZdk)

주의하셔야할 점은, 사용하는 컴파일러의 버전에 따라서 사용할 수 있는 크기가 23바이트보다 작을 수도 있습니다. 23바이트는 이론적으로 가능한 최대 크기에요.


<br>

## Example
---

```C++
const char* SHORT_STR = "hello world";

void ShortStringCreation(benchmark::State& state) {
  // String을 계속해서 만드는 예제입니다.
  // SHORT_STR 정도면 23바이트 안에 들어갑니다.
  // 즉 SSO가 돌아요!
  for (auto _ : state) {
    std::string created_string(SHORT_STR);
  }
}

void ShortStringCopy(benchmark::State& state) {
  // 여기 예제에서는 string을 한번 만들고, 내용을 계속 복사합니다.
  // ... 카피가 계속 만드는거보다 빠를 줄 알았는데, 왜 ShortStringCreation보다 느릴까요?
  // 알고보니, 컴파일러가 생각보다 많이 똑똑한거였습니다. SSO 최고,, 
  std::string x; // create once
  for (auto _ : state) {
    x = SHORT_STR; // copy
  }
}

const char* LONG_STR = "this will not fit into small string optimization";

void LongStringCreation(benchmark::State& state) {
  // LONG_STR 정도의 긴 string은 분명히 memory allocation을 트리거 할겁니다.
  for (auto _ : state) {
    std::string created_string(LONG_STR);
  }
}

void LongStringCopy(benchmark::State& state) {
  // String 내용을 복사할 때는 원래 이렇게 빨라야합니다.
  // 긴 string 을 쓸 때는 무조건 한번 만들고 recycle 해야해요.
  std::string x;
  for (auto _ : state) {
    x = LONG_STR;
  }
}
```

우리가 보통 알고 있는 "매번 새롭게 String을 만들면 느릴꺼야!" 라는 생각이 여기서 깨지게 됩니다. String이 짧다면, 복사를 하는 것 보다 더 빠를 수 있어요.
물론 긴 string을 만드는 것은 예상했던대로 memory allocation이 들어가서 느립니다. 이 경우는 한번만 만들고 내용을 계속 복사해주는 게 더 빨라요.

{% asset_img "sso_in_action.png" "SSO 실행" %}



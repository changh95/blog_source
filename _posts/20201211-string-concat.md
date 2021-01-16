---
title: (번역) 실시간 컴퓨터 비전을 위한 C++ 최적화 (7) - String 이어 붙일 때 '+' 쓰지 마세요!
date: 2020-12-11 20:38:19
tags: 
- C++
- CV
- Optimisation
categories: 
- [Programming, C++, 실시간 컴퓨터 비전을 위한 C++ 최적화 시리즈]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "String concatenation the false sense of security of `operator+`"을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "String concatenation the false sense of security of `operator+`"을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/strings_concatenation/)를 봐주세요.

<br>

# String 이어붙이기
---

이 글을 시작하기 전에 룰 #1을 설명하고 시작하겠습니다.

룰 #1 : ***"Profiler로 봤을 때 오버헤드가 있는 경우에만 최적화를 하세요"***

이전 글에서도 얘기했듯이, string은 `std::vector<char>`과 다를바가 없습니다. 그렇기 때문에 데이터를 저장하기 위해 heap allocation를 해야할 수도 있죠.

C++에서 string을 이어붙이는 것은 (i.e. concatenate) 굉장히 쉽습니다. 하지만 조심해야할 점이 있습니다. 

<br>

## 많이 사용하는 concatenation 방법
---

우리가 자주 쓰는 방법은 아래와 같습니다.

```C++
std:string big_string = first + " " + second + " " + third;

// Where...
// std::string first("This is my first string.");
// std::string second("This is the second string I want to append.");
// std::string third("This is the third and last string to append."); 
```

우리가 많이 썼던 방식이지만... 최적화 글을 몇개 읽으신 당신에게는 아마 이제 뭔가 쎄한 느낌이 들겁니다. 

> '이렇게 적으면 Heap allocation에 문제가 있을텐데...??"

{% asset_img "spider_senses.png" "string sense가 팅글링한다~~" %}

<br>

쎄한 느낌에 최적화 센스가 팅글링합니다.

Heap allocation을 고려해서 다시 적어보겠습니다.

```C++
std:string big_string = (((first + " ") + second) + " ") + third;
```

이 정도 길이의 string을 이어붙이려면, heap allocation이 많이 수행되어야하고, 매번 이전 메모리에서 신규 메모리로 복사가 되어야합니다.

`std::string`이 `std::vector::reserve()`와 같은 기능이 있었으면 좋겠는데 말이죠... :(

...는 실제로 [이 기능](https://en.cppreference.com/w/cpp/string/basic_string/reserve)이 있었네요? 띠용용

<br>

## 무작정 가져다 붙이기
---

`reserve`를 사용해서 heap allocation의 횟수를 한번으로 줄여봅시다.

`big_string` 안에 들어가는 글자의 수를 미리 세어두고 `reserve`를 사용하면 이렇게 될겁니다.

```C++
    std::string big_one;
    big_one.reserve(first_str.size() + 
                    second_str.size() + 
                    third_str.size() + 
                    strlen(" ")*2 );

    big_one += first;
    big_one += " ";
    big_one += second;
    big_one += " ";
    big_one += third;
```

이 코드를 보시는 여러분들이 무슨 생각을 하시는지 다 압니다. 

{% asset_img "feel_bad.jpg" "어우야 대체 왜 제발 좀" %}

으... 이 더러운 코드는 뭐야...

라고 생각하게 되지만, 실제로 Profiler를 돌려보면 이 코드는 기존의 방법보다 ***2.5배*** 나 더 빠릅니다. 

속도는 좋은데, 진짜 가독성 어떻게하죠 이거?

<br>

## 더 나은 방법? - Variadic concatenation
---

좀 더 빠르고, 재사용하기 좋고, 읽기 쉬운 string concat 방법이 없을까요?

Modern C++을 사용해야하는 이유가 여기에 있습니다. ***Variadic templates***를 사용해봅시다.

Variadic template가 뭔지 모르신다면, 여기 템플릿에 대해 설명을 잘 해주는 굉장히 좋은 [링크](https://arne-mertz.de/2016/11/more-variadic-templates/)가 있습니다.


```C++
//--- String의 전체 크기를 구해주는 함수 ---
size_t StrSize(const char* str) {
  return strlen(str);
}

size_t StrSize(const std::string& str) {
  return str.size();
}

template <class Head, class... Tail>
size_t StrSize(const Head& head, Tail const&... tail) {
  return StrSize(head) + StrSize(tail...);
}

//--- String append 를 해주는 함수 ---
template <class Head>
void StrAppend(std::string& out, const Head& head) {
  out = head;
}

template <class Head, class... Args>
void StrAppend(std::string& out, const Head& head, Args const&... args) {
  out += head;
  StrAppend(out, args...);
}

//--- String concat을 해주는 함수 ---
template <class... Args> 
std::string StrCat(Args const&... args) {
  size_t tot_size = StrSize(args...);
  std::string out;
  out.reserve(tot_size);

  StrAppend(out, args...);
  return out;
}
```

복잡한 코드가 좀 많았죠? 

그래도 우리는 이제 아래와 같이 굉장히 읽기 좋고 편한 방법으로 string concat을 할 수 있습니다.

```C++
std:string big_string = StrCat(first, " ", second, " ", third );
```

그럼 이제 얼마나 빨라졌는지 봅시다.

{% asset_img "string_concatenation.png" "string concat - benchmark" %}

이 Variadic template 방식이 위의 '더러운' 코드보다 느린 이유는...

사실 잘 모르겠습니다!

대신, variadic template 방식은 기존의 방식보다 2배나 빠르고 또 읽기 쉽다는건 확신합니다.

<br>

## 제 코드를 복붙하시기 전에...
---

제가 구현한 `StrCat`은 사실 그렇게 사용성이 좋지 않습니다. 제가 이 코드를 만든 이유는, 단순히 기존의 string concat보다 이 방식이 더 빠르다는걸 보여드리고 싶었어요.

실제로 구현 하실 때는, 위의 코드도 다 잊어버리시고 그냥 [{fmt} 라이브러리](https://github.com/fmtlib/fmt)를 사용하세요.

fmt 라이브러리는 쓰기도 엄청 쉽고, 도큐먼트도 잘 만들어져있고, 그리고 string formatting 할 때 ***엄청나게 빠릅니다***.

그리고 [C++20 std::format](https://en.cppreference.com/w/cpp/utility/format)의 구현체이기도 하지요.

이즉슨, {fmt}를 쓰시거나 C++20을 쓰시면 읽기 쉽고 빠른 string 코드를 쓰실 수 있다는 겁니다.






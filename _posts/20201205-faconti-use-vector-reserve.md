---
title: (번역) 실시간 컴퓨터 비전을 위한 C++ 최적화 (1) - std::vector<>::reserve()를 쓰세요!
date: 2020-12-05 12:44:26
tags: [C++, CV, Optimisation]
categories: 
- [Programming, C++, 실시간 컴퓨터 비전을 위한 C++ 최적화 시리즈]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "Use std::vector<>::reserve by default"을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "Use std::vector<>::reserve by default"을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/reserve/)를 봐주세요.

<br>

# std::vector 최고...
---

`std::vector<>`는 다른 자료구조에 비해 엄청난 장점이 있습니다. 메모리 상에서 vector element가 바로 옆에 따닥따닥 붙어있다는 점입니다. 

`std::vector<>`의 이런 특성이 왜 성능향상을 시키냐, 를 설명하려면 이 글이 꽤 길어질겁니다. 프로세서가 어떻게 메모리를 처리하는지 설명을 해야할테니까요. 

좀 더 자세히 알고싶으신 분들은, "C++ cache aware programming"이라고 구글링 해보시거나, 아니면 아래 자료를 보시는걸 추천드립니다.

- [CPU Caches and why you Care](https://www.aristeia.com/TalkNotes/codedive-CPUCachesHandouts.pdf)
- [Writing cache friendly C++ (video)](https://www.youtube.com/watch?v=Nz9SiF0QVKY) 

`std::vector<>`의 element에 대해 iterator를 돌리는건 엄청나게 빠릅니다. 또 vector의 끝단 element를 지우거나 추가할 때도 잘 되죠.

<br>

# `reserve`를 써야할 때
---

`reserve()`를 왜 써야할까요? 

이 질문에 답을 하려면 `std::vector<>`가 어떻게 작동하는지 이해해야합니다.

비어있거나, 꽉 차있는 `std::vector<>`에 새로운 element를 넣을 때, 우리는...

- 새로운 메모리 블록을 할당해야하거나
- 이전 메모리 블록에 저장되어있는 모든 element 정보를 새로운 블록으로 이동해야합니다.

어떤 방법을 택하든 엄청 비싼 작업입니다. 그리고 비싼 작업은 지양해야합니다. 뭐, 가끔은 그냥 눈물을 머금고 해야하지만요...

이 새로운 메모리 블록은 보통 ***이전 블록의 2배 크기***를 가집니다. 그렇기에, 우리의 vector가 `size()`도 `capacity()`도 100개의 element를 가지고 있을 때 우리가 `push_back()`을 해버리면, 새로운 메모리 블록은 200개의 element를 가질 수 있게 할당됩니다.

200개의 메모리 블록을 가지는게 잘못됬다는게 아닙니다. 다만 이 allocation 작업이 반복해서 나타난다면 엄청나게 느려지겠죠. 이런 allocation 과정을 피하기 위해 우리는 메모리 블록을 ***'미리 할당' (i.e. reserve)*** 할 수 있습니다. 

아래 코드를 한번 봅시다.

```C++
static void NoReserve(benchmark::State& state) 
{
  for (auto _ : state) {
    // create a vector and add 100 elements
    std::vector<size_t> v;
    for(size_t i=0; i<100; i++){  v.push_back(i);  }
  }
}

static void WithReserve(benchmark::State& state) 
{
  for (auto _ : state) {
    // create a vector and add 100 elements, but reserve first
    std::vector<size_t> v;
    v.reserve(100);
    for(size_t i=0; i<100; i++){  v.push_back(i);  }
  }
}


static void ObsessiveRecycling(benchmark::State& state) {
  // create the vector only once
  std::vector<size_t> v;
  for (auto _ : state) {
    // clear it. Capacity is still 100+ from previous run
    v.clear();
    for(size_t i=0; i<100; i++){  v.push_back(i);  }
  }
}
```
{% asset_img "vector_reserve.png" "vector reserve" %}

차이를 한번 보세요! Element가 100개 밖에 안되는데 이렇게 차이가 납니다.

Element의 수가 몇개가 되냐에 따라서 reserve의 성능 향상치가 달라질 수 있습니다. 하지만 하나 확실한 점은, ***reserve를 쓰면 무조건 빨라진다는 겁니다***.

그리고 또, `ObsessiveReclycing` 함수를 자세히 봅시다. 작은 vector에서는 성능차이가 있는데, 큰 vector에서는 별 차이가 없습니다.

아, 물론 `ObsessiveRecycling`이 더 빠릅니다. 그냥 어떤 사이즈를 쓰냐에 따라서 '눈에 띄게 달라지는지'는 조금 차이가 날 수 있다는거죠.

<br>

## Profiler에서 `std::vector<>` 때문에 생기는 문제인지 바로 알아보는 법

아래 사진은 어플리케이션 runtime에서 측정된 메모리 할당량입니다 (Heaptrack이라는 프로파일러를 사용했습니다.)

{% asset_img "growing_vector.png" "사이즈가 점점 커지는 vector" %}

그냥 딱 봐도, 뭔가 메모리가 계속 두배씩 커지네요. 대체 뭐 때문일까요? 아, vector 때문이겠네요. 다른 자료구조는 linear 하게 커지니까요.

이렇게 Profiler로 보면, 종종 `std::vector<>` index에 관련된 버그를 잡을 수 있습니다. 100개의 element를 담는 vector를 만들었는데, 메모리가 2배 늘어나는게 보인다면 분명히 100개를 초과했을테니까요. 이렇게 버그를 잡을 수 있습니다. 

<br>
<br>
게시글 공유:{% share_post twitter %}, {% share_post facebook %}, and {% share_post linkedin %}
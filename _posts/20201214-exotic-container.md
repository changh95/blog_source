---
title: (번역) 실시간 컴퓨터 비전을 위한 C++ 최적화 (10) - STL에 없는 새로운 컨테이너
date: 2020-12-14 22:38:19
tags: 
- C++
- CV
- Boost
- Data Structure
- Optimisation
categories: 
- [SLAM]
- [C++]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "Optimizing using an exotic associative container`"을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "Optimizing using an exotic associative container"을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/boost_flatmap/)를 봐주세요.

<br>

# 문제
---

지난주에 제가 사용하는 모듈 중 CPU를 많이 잡아먹는 친구를 뜯어보았습니다.

정확히 어떤 모듈인지 설명드릴 수는 없지만, 대충 로우레벨 하드웨어 인터페이스로 저희 회사 로봇 제품의 모터를 제어하는 코드입니다.

저는 이 코드가 CPU 코어 하나를 다 차지할정도로 이 소프트웨어가 무거우면 안되는 것을 잘 알고 있습니다. 하지만, 안타깝게도 지금의 코드는 그런 상태이죠.

언제나와 같이, 저는 제 무기인 **Hotspot**을 꺼내들었고, CPU가 코드의 어떤 부분에서 시간을 제일 많이 보내는지 알아봤습니다.

{% asset_img "motor_profile1.png" "모터 프로파일링 1" %}

좀 지저분하지요? 어쩔 수 없습니다.

[Flamegraph](http://www.brendangregg.com/flamegraphs.html)를 처음 보시는 분들에게 간단하게 설명을 드리자면, caller function이 제일 밑에 있고, 점차 위로 callee function이 쌓여가는거라고 보시면 됩니다.

여기서 제가 발견한 것은, 제 CPU의 30%가 `std::unordered_map<>::operator[]`에 쓰이고 있다는 것입니다.

{% asset_img "motor_profile2.png" "모터 프로파일링 2" %}

오른쪽에는 엄청나게 큰 블록이 있고, 왼쪽에는 작은 블록이 여러개 많이 있습니다.

오른쪽 블록은 caching으로 쉽게 풀 수 있고, 왼쪽은 좀 풀기 어렵죠.

문제가 되는 코드는 아래와 같습니다.

```C++
// 간단한 코드
std::unordered_map<Address, Entry*> m_dictionary;

struct Address{
    int16_t index;
    unt8_t subindex;
};
```
<br>

# 해결 방법
---

코드를 좀 더 뜯어봤습니다.

```C++
// 간단하게 적은 코드
bool hasEntry(const Address& address) 
{
    return m_dictionary.count(address) != 0;
}

Value getEntry(const Address& address) 
{
    if( !hasEntry(address) {
        throw ...
    }
    Entry* = m_dictionary[address];
    // ... 나머지 코드
}
```

<br>


{% asset_img "two_lookups.jpg" "두번 룩업하네요!" %}

<br>

***두번이나 값을 찾네요***! 

누가 짠거야 이 코드!

아래처럼 코드를 바꿔줬습니다.

```C++
// 한번만 값을 찾도록 바꿈
Value getEntry(const Address& address) 
{
    auto it = m_dictionary.find(address);
    if( it ==  m_dictionary.end() ) {
        throw ...
    }
    Entry* = it->second;
    // ... 나머지 코드
}
```

<br>

이렇게 코드를 바꿔주는 것 만으로도 `std::unordered_map`의 오버헤드를 반이나 줄일 수 있었습니다.

이제 오른쪽, 왼쪽 블록의 문제를 해결해봅시다.

<br>

## 오른쪽 블록: 캐싱을 합시다! (i.e. 따로 저장해둡시다!)
---

우선 짚고 넘어가야하는 점은, 이 `std::unordered_map` 형태의 dictionary는 한번 만들어지면 run-time에서 절대 바뀌지 않는다는 점 입니다.

이 점을 이용해서 우리는 flamegraph의 오른쪽에서 엄청 큰 블록에 최적화를 할 수 있습니다. 바로 캐싱을 이용하면 됩니다. 

Map에서 값을 찾을 때 우리는 `Address`를 통해 `Entry*`를 찾습니다. 하지만 이 pair가 절대로 변하지 않는다면, 그냥 `Entry*`만 저장해도 되지 않을까요?

그 결과, `std::unordered_map`를 사용한 오른쪽 블록의 큰 오버헤드가 사라졌고 , 약 15%의 CPU만 사용하는 효율적인 코드가 되었습니다.

이렇게 따로 값을 저장하는 LUT를 만들고 사용하는 것은 굉장히 효과적입니다. 좀 더 자세히 알고싶으신 분들은 [지난 글](https://changh95.github.io/20201211-dont-compute-twice/)을 참조하시길 바랍니다.

<br>

## 왼쪽 블록: `boost::container_flat_map`로 광명찾기
---

이제 왼쪽의 코드를 봅시다. 

왼쪽의 경우, 모두 `std::unordered_map`을 사용하기는 하지만, 사용하는 코드가 다 제각각입니다. 이 모든 코드를 하나하나 다 LUT로 바꿔주기는 굉장히 귀찮습니다.

하지만 그렇다고 가만히 둘 수도 없죠. 'Death by 1000 papercuts'가 딱 이 케이스입니다.

그리고 제 머릿속을 스쳐가는 무언가가 있었습니다.

`std;:has<Address>`는 왜 이렇게 오래걸리는걸까요? 이게 딱 그 hash table이 잘 작동하지 못하는 (Big O() 노테이션이 작동하지 않는) 몇개의 ***레어 케이스*** 인걸까요?

`std::unordered_map` 룩업은 O(1) 이여야 할 터입니다. 

분명 잘 되야할텐데, 기가차고 코가찰 노릇입니다.

<br>

### Boost 라이브러리 - flat_map
---

이 문제를 풀려면 꽤나 머리를 싸매야 할 것 같습니다.

한번 다른 시각에서 문제를 바라보겠습니다. O(logn)을 쓰면서, hash function의 단점이 없는 마법같은 컨테이너는 없는걸까요?

자 신사숙녀 여러분, [boost::container_flat_map](https://www.boost.org/doc/libs/1_74_0/doc/html/container/non_standard_containers.html#container.non_standard_containers.flat_xxx)를 소개합니다.

위의 링크에서 소개하는 내용을 반복하지는 않겠습니다. 도큐먼트가 상당히 설명을 잘 해줍니다. 

아주 짧게 설명하면, map을 기존의 ordered vector처럼 사용하겠다는 겁니다. [이전 글](https://changh95.github.io/20201206-do-you-need-map/)에서 제가 설명했던 방식과 굉장히 유사합니다.

결과는 놀랍습니다.

오버헤드가 그냥 줄은게 아니라, ***아예 사라져버렸습니다***.

`flat_map<>::operator[]`의 비용은 거의 없더군요.

단순하게 `flat_map`으로 바꿔주는 것 만으로도 위의 모든 내용이 한줄의 코드로 바뀌면서 오버헤드가 모두 사라졌습니다.

---
title: 컴퓨터 비전에서 std::vector 초기화 vs reserve 성능비교
date: 2020-12-16 20:38:19
tags: 
- C++
- CV
- Optimisation
categories: 
- [C++]
excerpt: std::vector<> vec(1000) vs vec.reserve(1000)의 차이를 알아보고, 컴퓨터 비전에서의 적용 예시를 적었습니다.
---

# `std::vector`의 문제
---

`std::vector`가 꽉 찬 상황에서 새로운 element를 추가한다고 해보자.

이미 메모리가 다 찼기 때문에, 새롭게 메모리를 할당해야한다. 이 때, vector는 Heap 속 새로운 위치에 메모리를 할당하고, 모든 데이터를 새 메모리 위치로 복사한다.

보통 새롭게 할당되는 메모리는 기존 메모리의 2배 크기가 된다.

이 메모리가 또다시 다 차게 되면, 위의 과정을 반복하게 된다.

<br>

이 과정은
1. Heap 메모리 할당 + 기존 메모리를 복사 작업에 시간이 들어가고
2. 복사되는 메모리의 크기가 크면 복사 시간이 늘어나기 때문에
   
`std::vector`를 자주 사용하는 실시간 컴퓨터 비전 입장에서는 꽤나 골치아픈 문제이다.

<br>

# 해결법
---

우리가 미리 해당 vector에 들어갈 최대 element 수를 알고 있다면, 그만큼 미리 메모리를 할당해둠으로써 추가적인 heap allocation을 피할 수 있다.

이 때, 우리는 두가지 방법: `reserve()`를 쓰는 방식과, 초기화 방식으로 메모리를 할당할 수 있다.

각 방식의 구현은 아래 코드를 참조하자.

```C++
#include <iostream>
#include <vector>

int main()
{
    // Reserve 방식
    std::vector<int> vec;
    vec.reserve(1000);

    for (const auto& elem : vec)
        std::cout << elem << std::endl;

    std::cout << vec.size() << std::endl;
    std::cout << vec.capacity() << std::endl;


    // Initialization 방식
    std::vector<int> vec2(1000);

    for (const auto& elem : vec2)
        std::cout << elem << std::endl;

    std::cout << vec2.size() << std::endl;
    std::cout << vec2.capacity() << std::endl;
}
```

<br>

## `reserve()` 방식 vs 초기화 방식 차이점
---

### 1. 저장되는 데이터  

- `reserve()` 방식은 size = 0, capacity = 1000 을 출력한 것에 비해,
- 초기화 방식은 size = 1000, capacity = 1000 을 출력하였다.

즉, `reserve()` 방식은 메모리만 할당하는 것, 그리고 초기화 방식은 실제로 어떤 값을 만들어서 넣는다는 것이다. 

실제로 확인해봤을 때, `reserve()` 방식은 어떠한 값도 `elem`에서 출력되지 않았고, 초기화 방식은 `elem`으로부터 0이 1000개가 출력되었다.

0이 중요한 정보로 사용된다면 (e.g. distance 값, true/false), `reserve()` 방식을 사용하는 것이 좀 더 안전할 것 같다.   

또, 아래와 같은 방식으로 체크를 한다면 `reserve()` 방식이 유리하다.

```C++
bool FeatureDetector::detectAndCompute(std::vector<VisualFeature>& features)
{
    // ... some code
}

int main()
{
    std::vector<VisualFeature> features;
    features.reserve(2000);

    if (!FeatureDetector.detectAndCompute(features))
        return;

    // code...
}

```

<br>

### 2. 생성되는 속도

그렇다면, 어떤 방식을 사용하는 것이 더 빠를까? 

아래의 코드로 테스트 해봤다.

```C++
#include <iostream>
#include <string>
#include <vector>
#include <chrono>

int main()
{
    std::chrono::system_clock::time_point startTime = std::chrono::system_clock::now();
    std::chrono::system_clock::time_point endTime = std::chrono::system_clock::now();
    std::chrono::microseconds t = std::chrono::duration_cast<std::chrono::microseconds>(endTime - startTime);
    
    // Reserve 방식
    startTime = std::chrono::system_clock::now();
    
    std::vector<int> vec;
    vec.reserve(1000);
    
    endTime = std::chrono::system_clock::now();
    
    t = std::chrono::duration_cast<std::chrono::microseconds>(endTime - startTime);
    
    std::cout << "Time to create vector in us : " << t.count() << " microseconds" << std::endl;

    // Initialization 방식

    startTime = std::chrono::system_clock::now();

    std::vector<int> vec2(1000);

    endTime = std::chrono::system_clock::now();
    
    t = std::chrono::duration_cast<std::chrono::microseconds>(endTime - startTime);
    
    std::cout << "Time to create vector in us : " << t.count() << " microseconds" << std::endl;
}
```

<br>

1000 개의 element로 테스트를 해봤을 때, 
- `reserve()` 방식은 43 마이크로초, 
- 초기화 방식은 2 마이크로초 정도가 걸렸다.

이렇게 보면 초기화 방식의 승리라고 볼 수 있다.

하지만 100,000개의 element로 테스트를 해봤을 때, 
- `reserve()` 방식은 65 마이크로초, 
- 초기화 방식은 390 마이크로초 정도가 걸렸다.

<br>

## 3. 작은 Element를 채워넣는 속도

```C++
#include <iostream>
#include <string>
#include <vector>
#include <chrono>

int main()
{
    std::chrono::system_clock::time_point startTime = std::chrono::system_clock::now();
    std::chrono::system_clock::time_point endTime = std::chrono::system_clock::now();
    std::chrono::microseconds t = std::chrono::duration_cast<std::chrono::microseconds>(endTime - startTime);
    
    // Reserve 방식
    
    std::vector<int> vec;
    vec.reserve(1000);

    startTime = std::chrono::system_clock::now();
    
    int zero = 0;
    for (uint32_t i = 0; i < vec.size(); i++)
        vec.push_back(zero);

    endTime = std::chrono::system_clock::now();
    
    t = std::chrono::duration_cast<std::chrono::microseconds>(endTime - startTime);
    
    std::cout << "Time to fill a vector in us : " << t.count() << " microseconds" << std::endl;


    // Initialization 방식

    std::vector<int> vec2(100000);

    startTime = std::chrono::system_clock::now();

    for (uint32_t i = 0; i < vec2.size(); i++)
        vec2[i] = zero;
        
    endTime = std::chrono::system_clock::now();
    
    t = std::chrono::duration_cast<std::chrono::microseconds>(endTime - startTime);
    
    std::cout << "Time to fill a vector in us : " << t.count() << " microseconds" << std::endl;
}
```

<br>

그렇다면 Element를 채워넣는 속도는 얼마나 차이가 날까?

1000개로 실험해봤을 때, 
- `reserve()`는 0 us, 
- 초기화 방법은 1 us가 나왔다.

별 차이가 없다고 볼 수 있다.

100,000개로 실험해봤을 때, 
- `reserve()`는 0 us, 
- 초기화 방법은 246 us가 걸렸다. 

<br>

## 4. 큰 Element를 채워넣는 속도
---

```C++
#include <iostream>
#include <string>
#include <vector>
#include <chrono>

// 큰 데이터
struct someBigData
{
    std::vector<long long> data = std::vector<long long>(1000);
    std::vector<long long> data2 = std::vector<long long>(1000);
    std::vector<long long> data3 = std::vector<long long>(1000);
};

int main()
{
    std::chrono::system_clock::time_point startTime = std::chrono::system_clock::now();
    std::chrono::system_clock::time_point endTime = std::chrono::system_clock::now();
    std::chrono::microseconds t = std::chrono::duration_cast<std::chrono::microseconds>(endTime - startTime);
    
    // Reserve 방식
    startTime = std::chrono::system_clock::now();
    
    std::vector<someBigData> vec;
    vec.reserve(1000);
    
    someBigData data;
    // startTime = std::chrono::system_clock::now();

    for (uint32_t i = 0; i < vec.size(); i++)
        vec.push_back(data);

    endTime = std::chrono::system_clock::now();
    
    t = std::chrono::duration_cast<std::chrono::microseconds>(endTime - startTime);
    
    std::cout << "Time to create vector in us : " << t.count() << " microseconds" << std::endl;

    // Initialization 방식
    startTime = std::chrono::system_clock::now();

    std::vector<someBigData> vec2(1000);

    // startTime = std::chrono::system_clock::now();
    for (uint32_t i = 0; i < vec2.size(); i++)
        vec2[i] = data;
        
    endTime = std::chrono::system_clock::now();
    
    t = std::chrono::duration_cast<std::chrono::microseconds>(endTime - startTime);
    
    std::cout << "Time to create vector in us : " << t.count() << " microseconds" << std::endl;
}
```

큰 Element를 채워넣는 속도 비교로 실험을 마친다.

`someBigData`라는 구조체를 만들고, 이를 기반을 vector를 만들고 element를 채워넣어보았다.

- `reserve()` 방식은 생성과정까지 포함하면 50 us, 
  - 포함하지 않고 push_back만 보았을 때는 0 us가 걸렸다.

- 초기화 방식은 생성과정까지 포함하면 14521 us, 
  - 포함하지 않고 index 접근만 하면 4032 us가 걸렸다.


<br>

# Discussion 및 결론
---

본 실험 결과를 다음과 같이 평가하였다.

- `0`이 vector element로써 중요한 의미로 사용된다면, `reserve()`를 사용하는 방식이 안전하다.
- Empty vector를 체크하는데에는 `reserve()`가 유리하다.
- 적은 양의 (~1000개) 작은 element를 가진 vector를 다룰 때에는 초기화 방법이 유리하다. 생성 속도는 `reserve()` 방식에 비해 훨씬 빠르고, element를 채워넣는 속도는 별 차이가 없다.
- 그 외 많은 양의 element를 가진 vector (~100,000개), 또는 큰 데이터를 담는 vector의 경우에는 `reserve()` 방식이 더 빠르다.

작은 element를 담아야하는 상황이 어떤 것이 있을까? Feature matching을 하고 hamming distance 등을 저장할 때 사용할 수 있을 것 같다.

큰 데이터 element를 담아야하는 상황은 어떤 것이 있을까? Feature 정보를 전부 담아둘 때 사용할 수 있을 것 같다. `std::pair<featureLocation, featureDescriptor>`와 같이 말이다.

이 두 방법의 속도가 이렇게 차이나는 이유가 무엇일까? 

특히나 큰 element를 저장할 때 이렇게 속도가 차이나는 이유가 궁금해진다.

Cache locality 라고 예전에 어디서 들어본 것 같은데, 이 키워드를 중점으로 조금 더 찾아봐야겠다.
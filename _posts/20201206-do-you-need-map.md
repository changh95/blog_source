---
title: (번역) 실시간 컴퓨터 비전을 위한 C++ 최적화 (3) - std::map을 써야할까요?
date: 2020-12-06 20:38:19
tags: 
- C++
- CV
- Optimisation
categories: 
- [C++, 실시간 컴퓨터 비전을 위한 C++ 최적화 시리즈]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "Do you actually need to use std::map?"을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "Do you actually need to use std::map?"을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/dont_need_map/)를 봐주세요.

<br>

# 어... 진짜로 std::map을 써야하나요?
---

`std::map`은 C++에서 가장 많이 사용하는 associative container 형태의 자료구조입니다. 시간이 지나면서 조금 인기가 떨어지긴 했어두요.

Associative container는 우리가 **Key / Value 페어 값**을 가지고 있고, 우리가 Key 값이 있을 때 해당하는 Value 값을 찾기 위해 사용합니다.

하지만, `std::map`은 생성될 때 [red-black tree](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree)가 생성되는 방식을 따릅니다. 그렇다보니, `std::list`와 비슷한 이유로 선호되지 않기도 하죠. (i.e. 메모리 레이아웃이 cache 메모리와 잘 맞지 않을 뿐더러, insertion 등을 할 때 memory allocation의 방식 등등...)

`std::map`을 쓰시기 전에, 우선 아래와 같은 질문을 한번 해봅시다.:
- 모든 데이터 페어가 key값에 대해 ***정렬*** 될 필요가 있는가?
- 이 컨테이너 내부에 있는 모든 값들에 대해 자주 iterate를 해야하는가?

첫번째 질문의 답이 '아니오' 였다면, `std::unordered_map` 으로 바꾸시는걸 추천드립니다.

제 벤치마크에서는 `std::unordered_map`을 사용하면 항상 성능이 올랐습니다. 이론적으로 상황에 따라 `std::map`이 더 좋은 성능을 낼 수 있지만, 저는 아직 그런 케이스를 본 적이 없네요.
(역자 코멘트: `std::map`은 밸런스 트리 구조, `std::unordered_map`은 해쉬 테이블을 사용합니다. 그렇기 때문에, 1. 저장하는 데이터의 양이 많고, 2.겹치는 데이터가 많을 수록 pigeonhole principle에 따라 `std::unordered_map`에서 hash collision이 나타나고 더 느려지게 됩니다)

두번째 질문의 답이 '네'라면... 문제가 굉장히 흥미로워집니다.

잠시 제 최적화 썰을 들어보시지요.

<br>

## Velodyne 드라이버 최적화 썰
---

여기 제가 꽤 자부심을 가지고 있는 Pull Request가 있습니다.

[Avoid unnecessary computation in RawData::unpack](https://github.com/ros-drivers/velodyne/pull/194)

작은 변화가 아주 큰 성능 개선을 만들었지요.

우선 이 드라이버가 어떤 기능을 하는지 설명드리겠습니다.

{% asset_img "velodyne.png" "Velodyne LiDAR 드라이버" %}

**벨로다인 (Velodyne)** LiDAR 센서는 매 초 마다 수많은 포인트 값을 계산합니다 (i.e. 장애물과의 거리 값). 

수 많은 자율주행 자동차에서 이 센서를 사용합니다.

벨로다인 드라이버는 polar coordinates에서 얻어진 관측 값을 3D cartesian coordinates (i.e. PointCloud) 형태로 바꿔줍니다.

저는 이 벨로다인 드라이버를 **Hotspot**이라는 profiler를 통해 분석해봤습니다.

그리고... `std::map::operator[]`라는게 CPU를 많이 잡아먹는다는 것을 보았죠.

코드를 뜯어봤고, 아래와 같은 점을 찾았습니다.

```C++
 std::map<int, LaserCorrection> laser_corrections;
```
`LaserCorrection`이라는 객체는 레이저 센서 값을 보정하기 위한 calibration 정보를 담고 있습니다.

Map에 들어있는 `int` key는 [0, N-1]의 range를 가지고 있습니다. 벨로다인 센서의 채널 수를 뜻하는데, 16, 32, 64, 128 등이 있죠.

그리고 `laser_corrections`는 딱 한번만 만들어지고 (insertion도 딱 한번만 합니다), 그 후에는 계속 루프에서 재사용됩니다.

```C++
 // 원본 코드를 조금 요약해서...
 for (int i = 0; i < BLOCKS_PER_PACKET; i++) {
    //some code
    for (int j = 0; j < NUM_SCANS; j++) 
    {   
        int laser_number = // omitted for simplicity
        const LaserCorrection &corrections = laser_corrections[laser_number];
        // some code
    }
 }
```
{% asset_img "that_is_logn.jpg" "저거 log_n인데" %}

흠, 이런 순수한 코드에...

```C++
laser_corrections[laser_number];
```

red-black tree 속 탐색을 하는 더러운 코드(?) 가 있었다니!

여기서 주의하셔야할 점은, index값은 랜덤값이 아닙니다. 이 값은 언제나 0~N-1이며, N은 작은 수에요.

그래서 다음와 같이 수정해서 제안을 했고, 아래와 같은 피드백을 받았습니다.

```C++
 std::vector<LaserCorrection> laser_corrections;
```

{% asset_img "quote.png" "답장" %}

요약하자면, 여기에는 associative containe가 애초부터 필요하지 않았습니다. 그냥 `std::vector`의 포지션 값을 사용해도 됬죠.

벨로다인 개발자들이 잘못했다는게 아닙니다. 이러한 문제는 회고를 할 때만 보이는게 정상이고, 또 실제로 프로파일러를 돌려보면서 시간을 재봐야 쓸데없는 오버헤드를 찾을 수 있기 때문이죠.

개발자의 입장에서는 대부분의 코드가 엄청나게 복잡하고 어려운 수학 계산을 하기 때문에, bottleneck이 수학쪽 계산에서 나타난다고 생각하기 쉽습니다. 하지만 종종 진짜 bottleneck은 `std::map`과 같은 기본적인 곳에서 나타나기도 하며, 이런 bottleneck을 알아차리기는 정말로 쉽지 않습니다.

<br>


## 한발짝 더 나아가서... [key,value]를 담는 vector는 어떨까요?
---

이번 예시는 사실 조금 특이한 케이스였습니다. 우선 Key 값이 참 편리하게도 int로 되어있었고, 또 0과 N 사이의 작은 값이였죠.

뭐 그래도, 저는 가끔 '진짜' associative container 대신 이런 코드를 짜기도 합니다.

```C++
std::vector< std::pair<KeyType, ValueType> > my_map;
```

이러한 자료구조의 장점은 ***모든 element에 iteration을 자주 돌려야할 때 엄청난 효율성***을 보여줍니다.

대부분의 다른 자료구조는 이 방법의 속도를 이길 수가 없어요!

<br>

> "어,, 근데 저는 element들이 ordered인 형태가 되야하는데... 그래서 제가 `std::map`을 쓴 거거든요..."

Ordering이 필요하시면... ordering 하시면 됩니다!

```C++
std::sort( my_map.begin(), my_map.end() ) ;
```

<br>

> "어,, 근데 저는 map에서 element를 찾아야하는걸요"

그 경우에는, [std::lower_bound](http://www.cplusplus.com/reference/algorithm/lower_bound/) 함수를 사용해서 ***ordered vector***로부터 key값을 이용해 value를 얻어낼 수 있습니다. `std::lower_bound`와 `std::upper_bound`의 complexity는 ***O(log n)***입니다. `std::map`이랑 같지만, iteration은 훨씬 더 빠르게 돌 수 있죠.

<br>

# 정리하자면...
---

- 데이터에 어떻게 접근을 하고싶은지 충분히 고민을 해야합니다.
- Insertion/deletion을 자주할지, 자주하지 않을지 제대로 파악해야합니다.
- Associative container를 사용함으로써 생기는 cost를 무시하시면 안됩니다.
- `std::unordered_map`을 먼저 써보세요... 아니면 `std::vector`를 써보세요!
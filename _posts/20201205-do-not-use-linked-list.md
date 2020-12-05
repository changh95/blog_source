---
title: (번역) 실시간 컴퓨터 비전을 위한 C++ 최적화 (2) - Linked List 쓰지 마세요!
date: 2020-12-05 20:38:19
tags: [C++, CV, Optimisation]
categories: [SLAM, C++]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "I have learnt linked-lists at university, should I use them?" Nooope."을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "I have learnt linked-lists at university, should I use them?" Nooope."을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/no_lists/)를 봐주세요.

<br>

# `std::list`를 쓰고 계신다면, 당장 그만두세요.
---


{% asset_img "linked_list.png" "Linked List를 왜 씁니까?" %}

사실 linked list의 효율성에 대한 벤치마크는 이미 이전에도 다른 사람들이 많이 했었습니다. 제가 똑같은 벤치마크를 다시 돌릴 필요는 없다고 봅니다.


- [std::vector vs std::list benchmark](https://baptiste-wicht.com/posts/2012/11/cpp-benchmark-vector-vs-list.html)

- [Are lists evil? Bjarne Stroustrup](https://isocpp.org/blog/2014/06/stroustrup-lists)

- [Video from Bjarne Stroustrup keynote](https://www.youtube.com/watch?v=YQs6IC-vgmo)

"그래도 내 경우에는 무조건 linked list를 써야할 것 같은데?" 라고 생각하실 수도 있는데, **장담하건데 linked list 안 쓰셔도 됩니다**.

STL 자료구조에는 이미 `std::list` 보다 훨씬 더 훌륭한 자료구조가 있습니다. [`std::deque<>`](https://es.cppreference.com/w/cpp/container/deque) 라는 친구가 99%의 경우 당신의 문제를 해결해줄 겁니다.

아니면, 우리의 영원한 친구 `std::vector`가 해결해줄 수도 있죠. 종종 `std::vector`는 list보다 더 효율적이게 작동합니다.

나는 너무 힙하고 멋져야하고 남들보다 뛰어나야하니 `std::vector`나 `std::deque` 같은건 절대로 쓸 수 없다, 라고 하신다면 [plf::colony](https://plflib.org/colony.htm)를 보시는걸 추천드려요.

***근데 진짜로, 그냥 `std::vector`나 `std::deque` 쓰세요.***
 
 <br>

## 실제로 쓰인 예시: Intel RealSense 카메라 드라이버의 성능 끌어올리기
---

제가 [Intel RealSense](https://github.com/IntelRealSense) 코드에 컨트리뷰트 했던 Pull Request에 대해 얘기해보려고 합니다.

{% asset_img "realsense.png" "영롱한 RealSense" %}

대체 무슨 생각으로 그랬는지는 모르겠지만, RealSense의 코드에서는 `std::list<>`를 쓰고 있었습니다. Intel에서 왜 아직도 이런걸 쓰고 있었는지 정말 이해가 안되네요.

***농담입니다, 인텔 개발자들 사랑해요 알러뷰!***

여기 제가 건의했던 PR의 링크입니다. [Considerable CPU saving in BaseRealSenseNode::publishPointCloud()](https://github.com/IntelRealSense/realsense-ros/pull/1097)

요약하자면, 이 PR은 간단한 내용 2개를 수정했습니다.

```C++
// 여기 매 카메라 프레임마다 생성되는 list가 있습니다.
std::list<unsigned> valid_indices;

// 이 list를 vector로 바꿨습니다. 이 vector는 class member로써, 사용 전에 clear됩니다.
std::vector<unsigned> _valid_indices;
```

추가로, 매 프레임마다 `sensor_msgs::PointCloud2 msg_pointcloud`라는 큰 오브젝트가 생성됬습니다. 이 큰 오브젝트를 class member로 변환되어 재사용함으로써 더 빨라졌지요.

이렇게 수정해줌으로써, 약 20~30% 정도의 속도를 향상시켰습니다.

코드 몇줄 바꿔서 이 정도 속도 향상을 얻는건 엄청난거죠.


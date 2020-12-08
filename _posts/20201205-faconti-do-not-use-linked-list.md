---
title: (번역) 실시간 컴퓨터 비전을 위한 C++ 최적화 (2) - Linked List 쓰지 마세요!
date: 2020-12-05 20:38:19
tags: [C++, CV, Optimisation]
categories: 
- [SLAM]
- [C++]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "I have learnt linked-lists at university, should I use them?" Nooope."을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "I have learnt linked-lists at university, should I use them?" Nooope."을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/no_lists/)를 봐주세요.

역자 코멘트: 원작자의 요청으로 최대한 원글의 뉘앙스를 그대로 옮겨왔습니다. 저는 `std::list`가 글에서 표현하는 것 처럼 쓸모없다고 생각하지는 않습니다. 그랬다면 이미 C++ 언어에서 사라졌겠지요 ㅎㅎ... 적시적기에 사용한다면 `std::list` 역시 타 자료구조보다 좋은 성능을 내겠습니다. 하지만, 실시간 컴퓨터 비전의 용도에서 그런 상황이 자주 나타나지 않는 점에도 동의합니다. 이 점 유의하시고 읽어주시길 부탁드립니다 :)

<br>

# `std::list`를 쓰고 계신다면, 당장 그만두세요.
---


{% asset_img "linked_list.png" "Linked List를 왜 씁니까?" %}

사실 linked list의 효율성에 대한 벤치마크는 이미 이전에도 다른 사람들이 많이 했었습니다. 제가 똑같은 벤치마크를 다시 돌릴 필요는 없다고 봅니다.


- [std::vector vs std::list benchmark](https://baptiste-wicht.com/posts/2012/11/cpp-benchmark-vector-vs-list.html)
  - (역자 코멘트: 위의 링크는 잘못 첨부된 그래프가 몇개 포함되어있습니다. 가장 중요한 random remove 관련 그래프가 잘못 올라와있더라구요. 그러니 정확한 벤치마크 결과를 위해서는 위 링크보다는 여기 링크: [C++ benchmark – std::vector VS std::list VS std::deque](https://baptiste-wicht.com/posts/2012/12/cpp-benchmark-vector-list-deque.html)를 참조하시길 바랍니다.)

- [Are lists evil? Bjarne Stroustrup](https://isocpp.org/blog/2014/06/stroustrup-lists)
  - (역자 코멘트: 이 링크 내용을 읽어보시면 C++의 창시자인 Bjarne Stroustrup는 list가 완전 잘못된건 아니라고 하고 있습니다. 정확하게 이해하고 쓰면 list도 다른 자료구조보다 좋은 성능을 낼 수 있습니다.)

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

---

## 역자 추가 내용

[C++ benchmark – std::vector VS std::list VS std::deque](https://baptiste-wicht.com/posts/2012/12/cpp-benchmark-vector-list-deque.html) 링크를 보시면 다음과 같은 내용이 나옵니다.

- 대부분의 경우, `std::vector`와 `std::deque`가 `std::list`에 비해 더 빠르다.
- `std::list`가 `std::vector`와 `std::deque`에 비해 더 빠른 경우는 아래와 같다.
  - Random insert (중간에 새 element를 insert) 작업에서, insert 되는 객체의 크기가 큰 경우
  - Random insert (중간에 새 element를 insert) 작업에서, insert 되는 객체가 non-trivial인 경우
  - Random remove (중간에 element를 erase) 작업
  - push_front (최앞단에 새 element를 insert) 작업 
  - sort 작업에서, element 객체가 큰 경우

즉, 위와 같은 작업에서는 `std::list`를 써야하는 것이 맞으며, 이는 원글이 주장하는 내용과 다릅니다.

원글이 `std::list`를 쓸 필요가 없다고 주장한 부분은, 아무래도 ***위와 같은 경우가 실시간 컴퓨터 비전에서 자주 나타나지 않는 상황***이기에 그런 것 같습니다.

우선적으로, 실시간 컴퓨터 비전에서는 센서로부터 오는 데이터를 순차적으로 저장하는 경우가 많기 때문에, '중간에 새 element를 insert' 하는 경우가 많이 없습니다. 

또, 새 데이터가 들어오면 가장 뒷단에 저장하는 경우가 많기 때문에, `push_front()` 같은 작업도 자주 나타나지 않죠.

'중간에서 element를 erase'하는 작업도 보통 잘 나타나지 않습니다. Raw 센서 데이터에서 '최대 정확도를 낼 수 있는 최소한의 데이터'만 남기는 것이 메모리와 계산량의 효율성에 큰 영향을 줍니다. 그렇기에 Raw 데이터에서 전처리를 통해 남는 데이터가 많지 않습니다. 보통 기존의 컨테이너에서 element를 지우기보다는 새로운 컨테이너에 전처리를 통과한 데이터를 저장하고, raw 데이터는 지우는 방식을 사용합니다.

Sort는 자주 사용되는 기법이지만, 경우에 따라서 비교해야하는 element 객체가 크거나 작을 수 있습니다. 여기에 있어서 많은 기법들이 element 객체를 직접 비교하는게 아닌, 어떠한 distance metric을 사용하여 비교합니다. L2 Norm, Hamming distance 등이 있습니다. 예시로, Visual feature matching을 위해 descriptor distance를 구하는데, 각각의 descriptor를 직접 비교하는 것이 아닌 floating distance / hamming distance 등으로 metric distance값을 계산하여, 매치의 정확도 값을 따로 저장하고, 그 정확도를 기점으로 sort하는 경우가 많습니다. 

하지만 컴퓨터 비전 쪽 계산에는 정말로 다양한 방식으로 계산을 하고 데이터를 저장해야합니다. 제가 생각하지 못한 부분에서 위와 같은 경우가 나타날 수 있고, 그런 경우에는 `std::list`가 진가를 보여줄 수 있는 부분이 있을 것이라고 생각합니다. 이 글을 읽으시는 분들께는 `std::list`를 사용하실 때 `std::list`, `std::vector`, `std::deque` 중 어떤 컨테이너를 사용하는게 더 효율적일지 한번 더 고려하실 수 있는 기회가 되길 바랍니다.
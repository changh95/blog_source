---
title: (번역) 실시간 컴퓨터 비전을 위한 C++ 최적화 (4) - 사이즈가 작은 vector를 쓸 때...
date: 2020-12-08 20:38:19
tags: 
- C++
- CV
- Optimisation
categories: 
- [Programming, C++, 실시간 컴퓨터 비전을 위한 C++ 최적화 시리즈]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "Small vector optimization"을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "Small vector optimization"을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/small_vectors/)를 봐주세요.

<br>

# Small 벡터 최적화
---

여기까지 읽으셨다면... 여러분들께서는 이제 우리가 associative container가 정말로 필요하지 않는 이상 왠만하면 `std::vector`를 사용해봐야 하는 것을 이해하실 겁니다.

`std::vector`를 사용하면서 사이즈가 늘어날 때 마다 비싼 heap allocation를 방지하기 위해 우리는 `reserve`를 사용했었습니다. 하지만, 이 방법을 쓰면 ***무조건 처음에 단 한번***은 heap allocation을 해야하죠. 이 점이 굉장히 아쉽습니다. 좋은 방법이 없을까요?

사실 여기에도 방법이 있습니다. 추후에 적을 [Small String Optimisation (SSO)](https://changh95.github.io/20201210-small-string-optimisation/) 글에도 이 방법을 사용할겁니다.

<br>

# "Static" vector와 "Small" vector
---

우리가 사용할 vector의 크기가 작고, 최악의 상황에도 작은 크기로 있을 것을 알 때가 있습니다. 이 경우에는 모든 element들을 heap가 아니라 stack에 넣어둘 수 있습니다. Heap allocation은 비싸거든요.

사이즈가 작은 vector를 쓴다니, 이런 경우가 얼마나 자주 있겠나~ 라고 생각하실 수도 있지만, 생각보다 굉장히 자주 나타납니다. 당장 2주 전만해도, 제가 사용하는 라이브러리에서 이러한 패턴을 발견했습니다. 이 vector의 크기는 0에서 최대 8까지 element를 가집니다.

30분 정도 리팩토링을 했고, 20%정도의 성능 향상을 얻었습니다!

리팩토링의 결과를 요약하자면... 
```C++
std::vector<double> my_data; // 이 vector를 사용하려면 heap allocation이 필요합니다.
```
위 코드를 이런 방식으로 바꿨죠.
```C++
double my_data[MAX_SIZE]; // Heap allocation이 없는 형태 
int size_my_data;
```
이 `StaticVector`의 간단한 구현을 한번 함께 봅시다.

```C++
#include <array>
#include <initializer_list>

template <typename T, size_t N>
class StaticVector
{
public:

  using iterator       = typename std::array<T,N>::iterator;
  using const_iterator = typename std::array<T,N>::const_iterator;

  StaticVector(uint8_t n=0): _size(n) {
    if( _size > N ){
      throw std::runtime_error("SmallVector overflow");
    }
  }

  StaticVector(const StaticVector& other) = default;
  StaticVector(StaticVector&& other) = default;

  StaticVector(std::initializer_list<T> init)
  {
    _size = init.size();
    for(int i=0; i<_size; i++) { _storage[i] = init[i]; }
  }

  void push_back(T val){
    _storage[_size++] = val;
    if( _size > N ){
      throw std::runtime_error("SmallVector overflow");
    }
  }

  void pop_back(){
    if( _size == 0 ){
      throw std::runtime_error("SmallVector underflow");
    }
    back().~T(); // call destructor
    _size--;
  }

  size_t size() const { return _size; }

  void clear(){ while(_size>0) { pop_back(); } }

  T& front() { return _storage.front(); }
  const T& front() const { return _storage.front(); }

  T& back() { return _storage[_size-1]; }
  const T& back() const { return _storage[_size-1]; }

  iterator begin() { return _storage.begin(); }
  const_iterator begin() const { return _storage.begin(); }

  iterator end() { return _storage.end(); }
  const_iterator end() const { return _storage.end(); }

  T& operator[](uint8_t index) { return _storage[index]; }
  const T& operator[](uint8_t index) const { return _storage[index]; }

  T& data() { return _storage.data(); }
  const T& data() const { return _storage.data(); }

private:
  std::array<T,N> _storage;
  uint8_t _size = 0;

```
StaticVector는 겉으로 보기에는 `std::vector`와 굉장히 유사하지만... 결과는 매우 서프라이징하네요.

{% asset_img "inconceivably.jpg" "아 예 써프라이징 성능~~" %}

종종 우리는 vector 컨테이너가 최대 ***N***개의 element를 가질 확률이 높다는건 알지만, 항상 그럴것인지 확신하지 못할 때가 있죠.

그런 경우에 우리는 ***Small vector (작은 벡터)***를 사용할 수 있습니다. 작은 vector는 첫 N개의 element는 stack에 미리 저장한 메모리 값을 사용하고, 그리고 그 벡터가 더 확장해야할 때***만*** 새로운 heap allocation을 통해 사이즈를 확장할 수 있습니다.

<br>

## StaticVector와 SmallVector를 실제로 쓰려면... 
---

찾아봤더니 이런 트릭은 실제로 많은 라이브러리에서 이미 구현이 되어있더라구요. 예시로는 아래의 라이브러리들이 있습니다.

- [Boost::container](https://www.boost.org/doc/libs/1_73_0/doc/html/container.html). (이런 트릭이 존재한다는건, Boost에 이미 있다는 뜻입니다 ㅋㅋ)
- [Abseil](https://github.com/abseil/abseil-cpp/tree/master/absl/container). 여기서는 `fixed array` 또는 `inlinec_vector`라고 합니다. 
- 제대로 공부하고 싶으신 분들은, 여기 링크를 타고 가시는 것을 추천드립니다. [SmallVector used internally by LLVM](https://github.com/llvm/llvm-project/blob/master/llvm/include/llvm/ADT/SmallVector.h)

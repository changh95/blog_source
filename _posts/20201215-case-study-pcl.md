---
title: (번역) 실시간 컴퓨터 비전을 위한 C++ 최적화 (11) - PCL 라이브러리 최적화
date: 2020-12-15 20:38:19
tags: 
- C++
- CV
- PCL
- Robotics
- Optimisation
categories: 
- [C++, 실시간 컴퓨터 비전을 위한 C++ 최적화 시리즈]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "Case study - filter a Point Cloud faster`"을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "Case study: filter a Point Cloud faster"을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/pcl_filter/)를 봐주세요.

<br>

[Point Cloud Library (PCL)](https://pointclouds.org/)는 로보틱스, 자율주행, 3D 인지 쪽에서 굉장히 유명한 라이브러리죠. 이 라이브러리는 위 분야들에 ***엄청난 기여***를 했습니다. 

{% asset_img "pcl.jpg" "PCL library" %}

PCL은 [12,000개가 넘는 커밋과 5천개가 넘는 Github star](https://github.com/PointCloudLibrary/pcl)를 가진 거대 오픈소스 프로젝트입니다.

약 [400명이 넘는 컨트리뷰터](https://github.com/PointCloudLibrary/pcl/graphs/contributors)가 있는데, 이런 경우에는 이 분야 고수들이 기능을 만드느라 최적화를 할 기회가 없다고 생각하실 수도 있겠습니다.

그리고 여기 제가 있습니다.

나: "ㅋㅋㅋ"

<br>

## ConditionalRemoval 필터
---

PCL을 조금 써보신 분들이라면 [pcl::ConditionalRemoval](https://pcl-tutorials.readthedocs.io/en/latest/remove_outliers.html) 기능을 한번쯤은 써보셨을 것 같습니다. 

[공식 튜토리얼](https://pcl-tutorials.readthedocs.io/en/latest/remove_outliers.html)에서는 해당 기능을 다음과 같은 방식으로 사용하라고 알려줍니다:

```C++
// 읽기 쉽게 조금 코드를 바꿨습니다
using namespace pcl;

auto range_cond  = std::make_shared<ConditionAnd<PointXYZ> ();
range_cond->addComparison ( 
    std::make_shared<FieldComparison<PointXYZ>("z", ComparisonOps::GT, 0.0));
range_cond->addComparison (
    std::make_shared<FieldComparison<PointXYZ>("z", ComparisonOps::LT, 1.0)));

// 필터 생성
ConditionalRemoval<PointXYZ> condition_removal;
condition_removal.setCondition (range_cond);
condition_removal.setInputCloud (input_cloud);
// 필터 적용
condition_removal.filter (*cloud_filtered);
```

> 이 코드는 간단하게 Point cloud에 필터를 적용하는데, 필터를 통과할 수 있는 point의 특성에 대한 조건을 걸어주는 겁니다. 여기서 그 조건은 다음과 같습니다. 0.0보다 크고 (Greater than == GT) 1.0보다 작은 경우 (Less than == LT). 

<br>

PCL 사용에 익숙하지 않으신 분들을 위해 조금 더 쉽게 정리하겠습니다.

- 필터 조건을 하나 만듭니다. 첫번째 조건은 'Point의 xyz좌표 중 Z 값이 0.0보다 커야한다' 입니다.
- 또 다른 필터 조건을 하나 만듭니다. 두번째 조건은 'Point의 xyz좌표 중 Z 값이 1.0보다 작아야한다' 입니다. 
- 이 조건들은 `ConditionAnd`에 추가됩니다.
- `ConditionalRemoval`을 사용해 이 두개의 조건들을 Point cloud에 적용할겁니다.
- 필터가 적용되면, 이 조건을 통과하는 point들만 point cloud에 남게 됩니다. 

<br>
보통의 Point cloud는 몇천개~몇만개의 포인트를 가지고 있습니다.

한번 생각해봅시다: 

{% asset_img "think_about_it.jpg" "생각 좀 해보고" %}
 
Point cloud는 사실상 ***vector***에 point 들을 담아놓은 데이터가 아니겠습니까?

각각의 point들은 아래처럼 생겼겠죠.

<br>

```C++
// 엄청 간단하게 축약한겁니다.
struct PointXYZ{
  float x;
  float y;
  float z; 
};
```

<br>

필터를 통과하고 난 후의 point cloud를 새로 만든다고 생각해봅시다. 필터 조건은 아래와 같습니다.
```C++
      0.0 < point.z < 1.0
```

<br>

제가 어떻게 코드를 바꿀껀지 물어보신다면, 저는 우선 이렇게 할 것 같습니다.

```C++
auto cloud_filtered = std::make_shared<PointCloud<PointXYZ>();

for (const auto& point: input_cloud->points) 
{
  if( point.z > 0.0 && point.z < 1.0 )
  {
    cloud_filtered->push_back( point );
  }
}
```

<br>

굉장히 간단한 **"naive filter"** 입니다.

실제 벤치마크를 보여드리기 전에, 우선 말씀드려야 할 점은... `pcl` 필터는 사실 이거보다 더 많은 체크를 하긴 합니다. 포인트 클라우드는 종종 이상한 레어 케이스가 있는데, 그걸 방지하는 체크가 많이 내장되어있습니다.

하지만 우선 저희가 아래와 같은 필터 조건을 건 것은 기억해주세요.

```C++
pcl::FieldComparison<pcl::PointXYZ> ("z", pcl::ComparisonOps::GT, 0.0)));
pcl::FieldComparison<pcl::PointXYZ> ("z", pcl::ComparisonOps::LT, 1.0)));
```

<br>

생각해보면, 어딘가 'parser'가 하나 있어야할겁니다.

이 parser를 짠다면 아무래도 `switch` 문을 써서 만들었을 것 같기도 하지만... 몇천~몇만개의 포인트마다 switch를 쓸 사람은 없을 것 같습니다.

...라고 생각했지만. 제엔장!! [진짜 switch 썼네요!](https://github.com/PointCloudLibrary/pcl/blob/pcl-1.11.0/filters/include/pcl/filters/impl/conditional_removal.hpp#L98-L127)

말도 안됩니다 . 몇천~몇만개의 포인트를, 각각 하나마다 switch문을 그것도 2개나 돌리고 있군요.

요약하자면 - 이 함수들은 이해하기 쉽게 만들기 위해서 ***속도를 버리고 있습니다***.

이렇게 짜인건 어쩔 수 없죠.

대신 우리가 쓸 때 바꿔주면 됩니다 ;)

<br>

## 빠르고 읽기 쉬운 코드를 짜봅시다!
---

수많은 pcl 함수들이 `switch`문을 쓰면서 성능이 박살났습니다. 

우리가 직접 pcl 함수들을 짜봅시다. `pcl::ConditionBase`는 어떨까요?

```C++
template <typename PointT>
class GenericCondition : public pcl::ConditionBase<PointT>
{
public:
  typedef std::shared_ptr<GenericCondition<PointT>> Ptr;
  typedef std::shared_ptr<const GenericCondition<PointT>> ConstPtr;
  typedef std::function<bool(const PointT&)> FunctorT;

  GenericCondition(FunctorT evaluator): 
    pcl::ConditionBase<PointT>(),_evaluator( evaluator ) 
  {}

  virtual bool evaluate (const PointT &point) const {
    // just delegate ALL the work to the injected std::function
    return _evaluator(point);
  }
private:
  FunctorT _evaluator;
};
```

<br>
딱 이 코드만 있어도 됩니다.

저는 단순히 `pcl::ConditionBase`속 `std::function<bool>(const PointT&)`를 랩핑해준겁니다. 다른건 없어요.

이렇게 바뀐 코드를 적용해봅시다.

<br>

**예전 코드**는 아래와 같습니다. 

```C++
auto range_cond  = std::make_shared<ConditionAnd<PointXYZ> ();
range_cond->addComparison ( 
    std::make_shared<FieldComparison<PointXYZ>("z", ComparisonOps::GT, 0.0));
range_cond->addComparison (
    std::make_shared<FieldComparison<PointXYZ>("z", ComparisonOps::LT, 1.0)));
```

<br>

그리고 여기 **새 코드** 입니다.

```C++   
auto range_cond = std::make_shared<GenericCondition<PointXYZ>>(
  [](const PointXYZ& point){ 
      return point.z > 0.0 && point.z < 1.0; 
  });
```

나머지 코드는 바뀐게 없습니다!

{% asset_img "beautiful.jpg" "아름답습니다..." %}

아름답습니다...

<br>

## 벤치마크 결과
---

[여기](https://github.com/facontidavide/CPP_Optimizations_Diary/tree/master/cpp/pcl_conditional_removal.cpp) 제가 짜둔 코드가 있습니다. 이 코드를 그대로 가져가셔서 테스트 해보셔도 됩니다.

여기 아래 샘플 포인트 클라우드에 4개의 필터를 돌린 벤치마크가 있습니다.

```
-------------------------------------------------------------
Benchmark                   Time             CPU   Iterations
-------------------------------------------------------------
PCL_Filter            1403083 ns      1403084 ns          498
Naive_Filter           107418 ns       107417 ns         6586
PCL_Filter_Generic     668223 ns       668191 ns         1069
```

물론 결과 값은 필터 조건이 몇개인지에 따라, 포인트 클라우드의 크기에 따라 달라질 수 있습니다.

<br>

# 배워갈 점

- "naive" 필터도 여러 케이스에서 사용해도 됩니다. 엄청 빠릅니다.
- `pcl::ConditionalRemoval`는 계속 써도 됩니다. 대신 기존의 `pcl::Conditions`는 너무 느리니까 가능하면 쓰지 마세요. 대신 `GenericCondition`을 쓰면 더 읽기 쉽고 빠르게 작동할 수 있습니다. 

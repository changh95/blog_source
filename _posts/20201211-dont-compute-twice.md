---
title: (번역) 실시간 컴퓨터 비전을 위한 C++ 최적화 (8) - 두번 계산하지 마세요! (당연하게도)
date: 2020-12-12 20:38:19
tags: 
- C++
- CV
- Robotics
- Optimisation
categories: 
- [Programming, C++, 실시간 컴퓨터 비전을 위한 C++ 최적화 시리즈]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "Don't compute it twice`"을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "don't compute it twice"을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/2d_transforms/)를 봐주세요.

<br>

# 두번 계산하지 마세요!
---

의미가 없는 계산을 하지 않는 것은 최적화 측면에서 엄청나게 당연한겁니다.

문제는 우리가 보통 코드를 짤 때 '의미가 없는 계산'인지 모르고 짤 때가 많다는 거죠. 이런 문제는 코드 리뷰 또는 회고를 할 때만 보입니다.

사실 이런 문제는 (심지어 같은 목적의 같은 맥락에서 사용되는 코드를) 여러 오픈소스 프로젝트 코드에서 종종 보이곤 합니다.

몇 백개의 Github star를 뽐내는 프로젝트들에서도 이런 부분이 최적화가 안 된 경우도 많습니다.

아래 좋은 예제를 가져왔습니다. Velodyne 라이다를 이용한 LOAM (라이다 오도메트리) 프로젝트의 ['Speed up improvement for laserOdometry and scanRegister (20%)'](https://github.com/laboshinl/loam_velodyne/pull/20) PR 입니다.

<br>

## 제일 자주 보이는 실수: LUT를 사용합시다.
---

아래 코드를 한번 봅시다.

```C++
double x1 = x*cos(ang) - y*sin(ang) + tx;
double y1 = x*sin(ang) + y*cos(ang) + ty;
```

그래픽스나 로보틱스를 공부하신 분들에게는 이 코드가 어떤건지 바로 알아보실겁니다. 

2D point의 Affine transformation이죠.

이 코드에는 최적화가 될 수 있는 부분이 뻔하게 보입니다.

```C++
const double Cos = cos(angle);
const double Sin = sin(angle);
double x1 = x*Cos - y*Sin + tx;
double y1 = x*Sin + y*Cos + ty;
```

Trigonometic function은 생각보다 계산량이 꽤 높습니다. 이 함수로 동일한 값을 두번이나 계산하는 것은 효율적이지 않습니다. 실제로, 아래의 개선된 코드가 위의 코드보다 두배 빠릅니다. 

이런 affine transformation 코드에서는 다양한 `angle`값이 사용될 수 있습니다. 특정 floating point의 값이 사용될 수도 있고, 아니면 고정 integer 값이 사용될 수도 있죠. 

우리가 사용할 `angle`값이 많지 않을 때는, 해당 값들을 미리 계산해서 따로 저장해두는 ***look-up table (LUT)***를 만들어두면 좋습니다.

아까 링크에 적어둔 PR이 풀려고 하는 문제가 딱 이 케이스였습니다. Laser scan data가 polar coordinate에서 Cartesian coordinate로 변환되는 코드 부분이였습니다.

{% asset_img "laser_scan_matcher.png" "Laser scan matching" %}

이 코드를 아래에 간단하게 정리해뒀습니다. 매 point마다 trigonometric function이 적용되며, 이 계산은 1초에 몇 천번 수행되죠.

```c++
// 비효율적인 코드...!
std::vector<double> scan_distance; // 인풋
std::vector<Pos2D> cartesian_points; // 아웃풋

cartesian_points.reserve( scan_distance.size() );

for(int i=0; i<scan_distance.size(); i++)
{
    const double dist = scan_distance[i];
    const double angle = angle_minimum + (angle_increment*i);
    double x = dist*cos(angle);
    double y = dist*sin(angle);
    cartesian_points.push_back( Pos2D(x,y) );
}
```

<br >

여기 우리 잠깐 멈추고 생각해봅시다: 

 - **최소 angle 값**과 **매 angle이 바뀌는 값 (increment value)**는 일정합니다.
 - **scan_distance**의 size도 항상 일정합니다. 
 
 이런 경우가 우리가 LUT를 써서 성능을 향상시킬 수 있는 좋은 예시입니다.
 
<br>

```C++
 
//------ 여기 부분을 한번만 계산하고 LUT를 만듭니다 -------
std::vector<double> LUT_cos;
std::vector<double> LUT_sin;

for(int i=0; i<scan_distance.size(); i++)
{
    const double angle = angle_minimum + (angle_increment*i);
    LUT_cos.push_back( cos(angle) );
    LUT_sin.push_back( sin(angle) );
}

// ----- 그리고 여기 conversion에서 LUT값을 불러와 효울적으로 계산을 수행합니다. ------
std::vector<double> scan_distance;
std::vector<Pos2D> cartesian_points;

cartesian_points.reserve( scan_distance.size() );

for(int i=0; i<scan_distance.size(); i++)
{
    const double dist = scan_distance[i];
    double x = dist*LUT_cos[i];
    double y = dist*LUT_sin[i];
    cartesian_points.push_back( Pos2D(x,y) );
}
```
 
<br>

# 정리하자면...
---

위에서 우리는 간단한 예제를 보았습니다. 

여기서 우리가 알아야할것은 비싼 계산이 들어갈 때마다 (e.g. SQL 쿼리, state를 거치지 않는 계산) 우리는 미리 저장된 값을 사용하거나 LUT를 사용하는 것을 고려해봐야합니다.

하지만 항상 그렇듯이, 최적화를 적용하기 전에 현재 코드가 얼마나 비효율적인지 ***실제로 측정***해보고, 그 후에 최적화가 의미가 있을 지 정하는 것이 제일 중요합니다.


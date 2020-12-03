---
title: (번역) 컴퓨터 비전을 위한 C++ 최적화 (0) - Const reference를 쓰세요!
date: 2020-12-03 12:44:26
tags: [C++, CV, Optimisation]
categories: [SLAM, C++]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "Value Semantics vs references"을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "Value Semantics vs references"을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/prefer_references/)를 봐주세요.

---

이번 글의 내용은 사실 어느정도 C++ 개발을 해보신 분들은 이미 잘 아는 내용일 것이라고 믿습니다.

그래도 저는 종종 이런 코드를 짜는 사람들을 봅니다.

```C++
bool OpenFile(str::string filename);

void DrawPath(std::vector<Points> path);

Pose DetectFace(Image image);

Matrix3D Rotate(Matrix3D mat, AxisAngle axis_angle);

```

위의 함수들은 제가 방금 만들어낸 것이긴 하지만, 실제로 제품으로 배포되는 코드에서도 종종 보이는 코드들입니다. 

위 코드들은 모두 함수 인자를 ***passing by value*** 형태로 넘깁니다.

즉, 이 함수를 호출할 때 마다, 이 함수의 인자들은 복사된 후에 함수로 넘겨집니다.

{% asset_img "why_copy.png" "왜 복사하는건데 대체" %}

복사 작업은 종종 굉장히 무거운 작업이 될 수 있습니다. 작은 객체를 복사할 때는 그리 무겁지 않지만, 엄청 큰 객체를 복사하면 무거울 수 있죠. 동적 heap memory allocation이 들어간다면 더 무거울 수도 있겠습니다. 

위의 예제들 중 아마 무시할 수 있을 정도의 굉장히 작은 양의 overhead가 나타나는 객체도 있습니다. `Matrix3D` 나 `AngleAxis` 같은 객체는 heap allocation이 없이도 복사가 됩니다. 

하지만 overhead가 작아도, 굳이 CPU 사이클을 이런 의미없는 복사에 낭비할 필요가 있을까요? 복사를 안하면 되지 않나요?

아래 예제를 한번 봅시다.


```C++
bool OpenFile(const str::string& filename); // string_view is even better

void DrawPath(const std::vector<Points>& path);

Pose DetectFace(const Image& image);

Matrix3D Rotate(const Matrix3D& mat, const AxisAngle& axis_angle);

```

이번 예제에서 우리는 ***reference semantic***이라는 기법을 사용했습니다.

이 방법 대신에 ***C-style***의 포인터를 사용해서 같은 결과를 얻을 수도 있겠습니다. 다만 포인터와 비교했을 때 reference semantic 기법이 다른 점이라면, reference semantic은 컴파일러에 다음과 같은 의미를 전달합니다.

- 함수 인자들이 **Constant** 하다는 점. 함수를 호출한 쪽에서도, 함수 내부에서도 이 인자 값을 수정하지 않겠다는 점을 명시합니다. 
- 함수 인자들이 **Reference**라는 점. 이미 값이 존재하는 인자들을 'refer' 하는 것입니다. C-style 포인터는 `nullptr` 가 값이 될 수 있지만, reference인 경우는 안되겠네요.
- 포인터를 사용하는게 아니니, 우리는 이 객체의 **ownership**을 넘겨주는게 아닙니다. 

어쨋든 이 방법을 사용하면 아래에서 보는 것 처럼 계산량이 엄청나게 줄어듭니다.

```C++
size_t GetSpaces_Value(std::string str)
{
    size_t spaces = 0;
    for(const char c: str){
        if( c == ' ') spaces++;
    }
    return spaces;
}

size_t GetSpaces_Ref(const std::string& str)
{
    size_t spaces = 0;
    for(const char c: str){
        if( c == ' ') spaces++;
    }
    return spaces;
}

const std::string LONG_STR("a long string that can't use Small String Optimization");

void PassStringByValue(benchmark::State& state) {
    for (auto _ : state) {
        size_t n = GetSpaces_Value(LONG_STR);
    }
}

void PassStringByRef(benchmark::State& state) {
    for (auto _ : state) {
        size_t n = GetSpaces_Ref(LONG_STR);
    }
}

//----------------------------------
size_t Sum_Value(std::vector<unsigned> vect)
{
    size_t sum = 0;
    for(unsigned val: vect) { sum += val; }
    return sum;
}

size_t Sum_Ref(const std::vector<unsigned>& vect)
{
    size_t sum = 0;
    for(unsigned val: vect) { sum += val; }
    return sum;
}

const std::vector<unsigned> vect_in = { 1, 2, 3, 4, 5 };

void PassVectorByValue(benchmark::State& state) {
    for (auto _ : state) {
        size_t n = Sum_Value(vect_in);
    }
}

void PassVectorByRef(benchmark::State& state) {
    for (auto _ : state) {
        size_t n = Sum_Ref(vect_in);
        benchmark::DoNotOptimize(n);
    }
}

```

{% asset_img "const_reference.png" "const reference가 최고입니다" %}


명백하게, passing by reference의 성능이 더 좋습니다.
Clearly, passing by reference wins hands down.

## Const reference를 쓰면 안되는 예시

> "오옹... 앞으로는 `const&`로 내 코드를 도배해야겠다~"

이런 생각을 하기 전에, 다음 예제도 한번 봅시다.

```C++
struct Vector3D{
    double x;
    double y;
    double z;
};

Vector3D MultiplyByTwo_Value(Vector3D p){
    return { p.x*2, p.y*2, p.z*2 };
}

Vector3D MultiplyByTwo_Ref(const Vector3D& p){
    return { p.x*2, p.y*2, p.z*2 };
}

void MultiplyVector_Value(benchmark::State& state) {
    Vector3D in = {1,2,3};
    for (auto _ : state) {
        Vector3D out = MultiplyByTwo_Value(in);
    }
}

void MultiplyVector_Ref(benchmark::State& state) {
    Vector3D in = {1,2,3};
    for (auto _ : state) {
        Vector3D out = MultiplyByTwo_Ref(in);
    }
}
```

{% asset_img "multiply_vector.png" "const& 써도 차이없음" %}

의외로, 이번에는 `const&`를 써도 아무 차이가 없습니다.

우리가 heap allocation이 필요없을만큼 작은 (그냥 몇 바이트 정도로 작은 데이터) 객체를 복사하는거면, passing by reference를 써도 거의 차이가 없습니다. 하지만 그렇다고 더 느려지는것도 아니죠. 그러니까 `const&`를 써야할지 말지 모르겠을때는, 일단 써보면 좋을 것 같습니다. 

다만 문제가 되는 경우도 있습니다.

Primitive types를 const reference로 복사할 때는 컴파일러가 추가적인 instruction을 더 만든다고 하지만 (여기서 테스트 해봤습니다 https://godbolt.org/z/-rusab), 하지만 컴파일 옵션에 `-O3` 를 추가하면 이 문제 역시 해결됩니다. 하지만 매번 이 옵션을 넣어주긴 귀찮으니, ***저는 보통 '8 바이트 정도 되는 인자, 또는 그것보다 작은 인자 (e.g. int, double, chars, long)`은 절대로 pass by reference를 쓰지 않습니다***. Primitive type에 const, & 등을 사용하면, 성능 향상은 하나도 되지 않으면서, 동시에 코드가 더 헷갈려보이고 못생겨집니다.

```C++
void YouAreTryingTooHardDude(const int& a, const double& b); // 이건 하지 맙시다
```

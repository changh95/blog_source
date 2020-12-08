---
title: C++에서 raw pointer 쓰는 방법
date: 2021-12-03 12:44:26
tags: [C++]
categories: 
- [C++]
excerpt: C++14의 unique_ptr<>로 공부를 시작했기 때문에 raw pointer를 직접 다뤄본적이 없습니다. 근데 코딩 테스트에서는 대부분 raw pointer 사용을 요구하더라구요? 그래서 공부하게 되었습니다.
---

저는 C++을 저희 팀원들의 코드를 통해 배웠습니다. 저희 팀의 C++ Standard는 C++17이였고, 모두가 raw pointer 대신 smart pointer를 사용하였습니다. 정말로 감사하게도, 저는 덕분에 smart pointer로 공부를 시작했습니다. 

Modern C++ 프로그래밍을 익힌 것은 좋았지만, 아직 여러 코딩 테스트에서는 raw pointer의 사용을 요구하더라구요. 그래서 저도 raw pointer를 공부하게 되었습니다. 

이번 글은 raw pointer 사용 법에 대한 공부 노트입니다.

<br>

# 포인터와 동적 메모리

동적 메모리를 이용하면 컴파일 시간에 크기를 확정할 수 없는 데이터를 다룰 수 있다.

<br>

## 스택 vs 힙

C++에서 사용하는 메모리는 ***스택(stack)***과 ***힙(heap)***으로 나뉜다.

스택에는 `main()` 함수 스코프에서부터 현재 실행중인 함수 스코프까지 차례로 쌓여있다.

예를 들어, 현재 `foo()`라는 함수가 실행된 상태에서 `bar()`라는 다른 함수를 호출하면, 최상단 스택 프레임은 `bar()`가 된다. 이 때, `foo()`에서 `bar()`로 전달되는 매개변수는 모두 `foo()`의 스택 프레임에서 `bar()`의 스택 프레임으로 복제된다.

스택 프레임에는 각각의 함수마다 독립적인 메모리 공간이 있다. 그렇기 때문에, 함수의 실행이 끝나면 해당 스택 프레임이 삭제되면서, 해당 함수 안에서 선언된 변수 역시 삭제되며 더 이상 메모리 공간을 차지하지 않는다.

힙은 스택과 완전히 독립적인 메모리 공간이다. 함수가 끝나도 그 안에서 사용하던 변수를 계속 쓰고싶다면 힙에 저장하면 된다.

대신 힙에 할당된 메모리는 자동으로 해제시켜주는 기능이 없기 때문에 (스마트 포인터 제외. 스마트 포인터가 이렇게 편하다~~), 사용자가 직접 해제해주어야한다.


<br>

# 포인터 사용법

1. Initiaisation
```C++
int* p_i; // declaration - uninitialized pointer variable

int* p_i = nullptr; // initialise with nullptr

int i = 8;
int* p_i = &i; // p_i is a pointer variable that points to the address of a variable called i, which contains 8 as value.
```

2. Deleting variables
```C++
delete p_i; // Release memory
p_i = nullptr // Make sure that the variable does not hold anything
```

3. Conditional variables
```C++
if(!p_i) // if p_i is nullptr, then p_i == false
{
    // code...
}
```

4. Pointer of a class / struct
```C++
// ugly method
Klasse* a_Klasse = getKlasse(); // Klasse is 'class' in German lol
std::cout << (*a_Klasse).memberVariable << std::endl;

// beaut method
Klasse* a_Klasse = getKlasse();
std::cout << a_Klasse->memberVariable << std::endl;
```

<br>

# 동적 배열 (Dynaimc allocation)

배열을 동적으로 할당할 때 힙을 사용한다. 이 때 new[] operator를 사용한다.

굉장히 불편하다.
```C++
int arraySize = 8;
int* array = new int[arraySize];

array[3] = 2;

delete[] array;
array = nullptr;
```

<br>

# 스마트 포인터 (smart pointer)

스마트 포인터로 지정한 객체는 스코프를 벗어나면 메모리가 자동으로 해제되기 때문에, 기존의 C 스타일 포인터가 겪는 메모리 관련 문제를 스마트 포인터를 사용함으로써 방지할 수 있다.

보통 `std::unique_ptr<>`과 `std::shared_ptr<>`이 많이 사용된다. 이 둘 다 `<memory>` 헤더에 정의되어있다.

<br>

## unique_ptr

`std::unique_ptr<>`은 객체가 1. 스코프를 벗어나거나, 2. 삭제되면, 해당 객체를 위해 하당된 메모리가 자동으로 삭제된다. 이 점을 제외하면 기존의 포인터와 다른게 없다.

`std::unique_ptr<>`을 생성하려면 `std::make_unique<>`를 써야한다. C++14 이전에는 다른 방식으로 추가할 수 있었지만, 이 방법은 추천되지 않는다.

`std::make_unique<>`로 `std::unique_ptr<>`을 만들때는 `auto` 키워드를 이용한 자동 타입 인퍼런싱을 사용하면 좋다. 왜냐면 클래스 등의 타입 인퍼런싱을 하면 코드가 너무 길어지기 때문이다.

```C++
auto p_Klasse = std::make_unique<Klasse>();

std::unique_ptr<Devices::Camera::CalibParams::FocalLength> p_focalLengths = std::make_unique<Devices::Camera::CalibParams::FocalLength>(); // 겁나리 길어지니 auto를 쓰자.
```

`std::unique_ptr`로 C 스타일 배열을 저장할 수도 있다.

```C++
auto klasse = std::make_unique<Klasse[]>(10);
std::cout << klasse[0]->memberVariable << std::endl;
```

<br>

## shared_ptr

`std::shared_ptr`는 데이터를 공유할 수 있는 스마트 포인터다. `std::shared_ptr`을 사용할 때 마다 reference count가 하나씩 증가한다. `std::shared_ptr`이 스코프를 벗어나면 이 reference ount가 줄어들며, reference count가 0이 되면 메모리를 해제한다.

`std::shared_ptr`을 만드는 방법은 `std::unique_ptr`과 비슷하다.

```C++
auto klasse = std::make_shared<Klasse>();
```
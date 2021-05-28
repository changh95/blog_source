---
title: Professional C++ - C++ 기초 정리 (Ch1, Part 1)
date: 2021-03-21 20:38:19
mathjax: true
tags: 
- C++
categories: 
- [C++, Professional C++ 책]
excerpt: Marc Gregoire의 "Professional C++"의 챕터1 파트1 정리 - 쉬운 내용이니까 중요한 내용만...
---

## 1.1.1 완전 기초 - Hello World 내부에 봐야하는 것

- 주석
- 전처리 지시자 (Pre-processor)
  - `#include [file]`
  - `#define [key] [value]`
  - `#ifdef [key], #endif`
  - `#ifndef [key], #endif`
  - `#pragma [xyz]`
- main() 함수
  - `int main(int argc, char* argv[])`
- I/O 스트림
  - `std::cout`, `std::cerr`
  - `\n`, `\r`, `\t`, `\\`, `\"`
  - `std::cin`

```cpp
#include <iostream>

// A simple hello world program
int main(int argc, char* argv[])
{
  std::cout << "Hello world!" << std::endl;
}

```

&nbsp;

## 1.1.2 Namespace

- `using namespace std;`는 쓰지 말자
- 헤더파일 안에서는 using 쓰지 말자
  - 그러면 헤더파일을 include하는 모든 파일에서 using문으로 지정한 방식으로 호출해야한다.
- C++17에서는 **Nested namespace**를 사용할 수 있다.
- Namespace alias도 잘 쓰자!

```cpp

namespace Library{
    namespace Module{
        namespace Submodule{
            // ... 
        }
    }
}

namespace Library::Module::Submodule{
    // ...
}

// namespace alias
namespace MySubmodule = Library::Module::SubModule;

```

&nbsp;

## 1.1.3 Literals

- 십진수, 8진수, 16진수, 이진수
- 신기하게 자릿수도 적을 수 있음 ㅋㅋ (e.g. 123'456'789 )

&nbsp;

## 1.1.4 변수

- signed int, short, long, long long
- unsigned int, short, long, long long
- float, double, long double
- char, char16_t, char32_t, wchar_t
- bool
- (C++17) **std::byte**
  - 기존에 쓰던 char나 unsigned char 대신 쓸 수 있음
- 캐스팅 방식에는 3가지 방법이 있음. **`static_cast<>`를 많이 쓰자!**

```cpp

int i1 = (int)myVal;
int i1 = int(myVal);
int i1 = static_cast<int>(myVal);

```

&nbsp;

## 1.1.5 연산자

- =, !, +, -, *, /
- % Modulus, 나머지 연산자
- ++ 사전증가/사후증가, -- 사전감소,사후감소
- +=, -=, *=, /=, %=
- &, &= AND 연산
- |, |= OR 연산
- <<, >>, <<=, =>> Bitwise shift
- ^, ^= XOR 연산

&nbsp;

## 1.1.6 타입

- Enum type
  - Enum type의 값에 int를 더해버리면 완전 다르게 될 수 있다.

```cpp
const int TypeA = 0;
const int TypeB = 0;
const int TypeC = 0;
const int TypeD = 0;

// same as ...
enum Type { TypeA, TypeB, TypeC, TypeD };
```

- 그러니 **Strong type** (또는 Type-safe하게) 만들자!
  - **Enum class**
  - 이 방식을 쓰는 것이 1. 안전하고, 2. 이해하기 더 쉽다.

```cpp

enum class Type : unsigned long
{
    TypeA = 1,
    TypeB,
    TypeC = 10,
    TypeD
};

// use it as
if (Type::TypeA == 1) {...}

```

- Struct
  - 미니 class 처럼 쓰면 된다. 대신 private 항목이 없는...

```cpp

struct Employee{
    char mNameInitial;
    char mSurnameInitial;
    int mEmployeeNumber;
    int mSalary;
}

// use as
int main()
{
    Employee employee;
    employee.mNameInitial = "H";
    employee.mSurnameInitial = "C";
    employee.mEmployeeNumber = 100;
    employee.mSalary = 80'000'000;
}

```

&nbsp;

## 1.1.7 조건문

- if/else문
  - **(C++17) Initialiser를 써서 if문 만들기**

```cpp

// C++17 Initilaiser if/else
if (Employee employee = GetEmployee(); employee.salary > 1000) {...}

```

- Switch 문
  - **Fallthrough model**도 있음
    - 버그에 취약하기때문에 C++17 방식을 사용하면 좋음

```cpp

switch (cases){
    case A:
        // case a
        break;
    case B:
        // case b
        break;
    default:
        // Error message
        break;
}

// Fallthrough model with C++17 initializer + [[fallthrough]]
switch (Case case){
    case A:
        // case a
        [[fallthrough]];
    case B:
        // case b
        break;
    case C:
        // case c
        [[fallthrough]];
    default:
        // Error message
        break;
}

```

- 조건 연산자
  - ```std::cout << ((i>2) ? "i is higher than 2" > "i is lower than 2");``` 

&nbsp;

## 1.1.9 함수

- 함수 선언 - 헤더파일
- 함수 구현 - 소스파일
- 함수 리턴 형에 `auto` 키워드 사용 가능.
- `__func__`에는 현재 함수 이름이 담겨있다.

&nbsp;

## 1.1.10 C 스타일 배열

- C 스타일 배열은 별로 안 좋아함...
- `int myArray[3] = {0};`처럼 만들 수 있음
  - 2차원은 `int myArray[3][3] = {0};`으로 만듬.
- (C++17)`int arraySize = std::size(myArray)` 
  - C++17를 지원하지 않으면 `unsigned int arraySize = sizeof(myArray) / sizeof(myArray[0])`;

&nbsp;

## 1.1.11 std::array

- C 스타일 배열이지만 iterator 쓰기 쉽게 만든 것
- ``` std::array<int,3> arr = {9,8,7}; ```

&nbsp;

## 1.1.12 std::vector

- 동적 컨테이너
- ```std::vector<int> vec = {11, 12};```

&nbsp;

## 1.1.13 Structural binding

```cpp

std::array<int,3> values = {11, 12, 13};
auto [x,y,z] = values;

```

&nbsp;

## 1.1.14 반복문

- for
- range based for
- while
- do-while

&nbsp;

## 1.1.15 Initalizer list

```cpp
#include <initializer_list>

int a = makeSum({1,2,3});
int a = makeSum({1,2,3,4,5,6});

int makeSum(std::initializer_list<int> lst)
{
    int total =0;
    for (int value: lst){
        total += value;
    }
    return total;
}

```

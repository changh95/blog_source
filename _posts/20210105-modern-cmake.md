---
title: Modern CMake
date: 2021-01-05 20:38:19
mathjax: true
tags: 
- CMake
- C++
categories: 
- [C++]
excerpt: Modern Cmake - An introduction 영상을 보고 공부한 노트입니다
---

Joshua Lospinoso의 '[Modern Cmake: An introduction](https://youtu.be/bDdkJu-nVTo)' 영상을 보고 공부한 노트입니다.
추천하는 책은 '[Professional CMake: A Practical Guide](https://crascit.com/professional-cmake/)'

---

# 가장 쉬운 인트로

## Executable 프로그램 만들기

``` cpp
#include <iostream>

int main()
{
    std::cout << "Hello, world!\n";
}
```

위와 같은 형태의 .cpp 파일이 있다고 해보자.
우리는 이 파일을 기반으로 binary executable이나 library를 만들고싶다.
이를 위해 우리는 CMake를 사용한다.

``` cmake
cmake_minimum_required(VERSION 3.15)
project(HelloWorld)

add_executable(App main.cpp)
```

- CMake 버전은 3.xx 이후로 사용한다면 어떤걸 사용하던지 크게 상관없다.
- 하나의 `CMakeLists.txt`는 단 1개의 project만 설정할 수 있다.
  - 그 안에서 여러개의 library와 executable (i.e. binary 프로그램)을 생성할 수 있다.
  - 위의 예시는 `main.cpp`라는 소스파일을 사용해서 `App`이라는 binary 프로그램을 만드는 것이다.
- 여기서 `App`에 해당하는 것을 target이라고 한다.

``` bash
$ ls -1
   CMakeLists.txt
   main.cpp
$ mkdir build && cd build/
$ cmake .. -GNinja
$ ninja
$ ./App
Hello, world!
```

터미널에서 다음과 같은 커맨드를 사용하면 프로젝트를 생성하고 실행할 수 있다.
`-GNinja` 라는 generator flag는 추후에 기술한다.
ninja 대신 make를 사용해도 된다 (Lospinoso는 ninja를 더 좋아한다고 한다).

<br>

### cmake 커맨드 & ninja 커맨드 설명

{% asset_img "cmake_run.png" "CMake run" %}

`cmake .. -GNinja` 커맨드를 실행했을 때, 위와 같은 파일들이 생성된다.
이 중 제일 중요한 것은 `CMakeCache.txt`인데, 이 파일 안에 빌드에 필요한 캐쉬 값들이 저장된다.

{% asset_img "ninja_run.png" "ninja run" %}

이 후 `ninja` 커맨드를 실행하면 빌드를 위해 ninja 패키지가 사용하는 intermediate object와 log, dependency 등이 생성된다.
그리고 제일 중요한 빌드의 결과물인 `build/App` executable이 생성된다.

<br>

### add_executable에 사용 가능한 옵션

``` cmake
add_executable (<name> [WIN32] [MACOSX_BUNDLE]
                [EXCLUDE_FROM_ALL]
                [source1] [source2 ...])
```

- `<name>`은 executable의 이름
  - Globally unique 해야함
- `WIN32`와 `MACOSX_BUNDLE`은 OS에 따른 빌드 방식 선언
- `EXCLUDE_FROM_ALL`은 `make all`과 같은 상위레벨 빌드 명령에서 이 target 빌드를 제외하는 플래그
-  헤더 파일은 보통 자동으로 찾아지고, cpp 소스파일만 넣어준다.

``` cmake
set(CMAKE_EXECUTABLE_SUFFIX .bin)
```

- Windows에서 target은 `App.exe`로 나온다
- Linux에서 target은 suffix가 없이 `App`으로 나온다.
- 이러한 셋팅은 위 `set` 명령어로 오버라이드 할 수 있다.
  - 위의 예제로는 `App.bin`이 나오게 된다.

---

## Library 만들기

``` cmake
add_library(Cat cat.cpp)
```

- Executable을 만드는 방식과 비슷하다.
- `Cat`이 target이고, `cat.cpp`는 소스파일이다.

<br>

### add_library에 사용 가능한 옵션

``` cmake
add_library(<name> [STATIC | SHARED | MODULE | OBJECT]
            [EXCLUDE_FROM_ALL]
            [source1] [source2 ...])
```

- `STATIC`이나 `SHARED` library를 고를 수 있다.
- 필요에 따라 `MODULE` (i.e. plug-in) 라이브러리를 만들 수도 있다.
  - C++20의 'module'과는 다르다.
  - 보통 Mac에서 사용하는 것 같다.
- Library type 정보를 넣지 않으면, `BUILD_SHARED_LIBS`라는 boolean variable에 저장된 정보로 static 또는 shared를 결정한다.

---

## Executable에 Library 링크하기.

`App`이 `Cat` 라이브러리를 사용한다면...

``` cmake
target_link_libraries(App PUBLIC Cat)
```

- `PUBLIC`은 `Cat` 라이브러리가 사용하는 모든 symbol이 App에서 노출된다는 것이다.
  - `PRIVATE`, `INTERFACE` 등도 있다. 추후에 기술한다.
- `target_link_libraries` 명령어는 target이 생성된 후에 사용되어야한다.
  - `App`과 `Cat`이 만들어진 다음에 쓸 수 있다는 뜻이다.

<br>

### target_link_libraries에 사용 가능한 옵션

``` cmake
target_link_libraries(<target>
                      <PRIVATE|PUBLIC|INTERFACE> <item> ...
                      [<PRIVATE|PUBLIC|INTERFACE> <item> ...] ...)                      
```

- 동시에 다수의 라이브러리를 링크할 수 있다.
  - 라이브러리마다 각각의 방식으로 링크할 수 있다.

{% asset_img "target_link_libraries.png" "target_link_libraries" %}

큰 프로젝트에서는 위와 같은 구조도 만들 수 있다.

---

## 파일/디렉토리 추가

``` cmake
add_subdirectory(dir1)
include(dir2/HelperScript.cmake)
```

- Directory를 추가하고 싶을 때는 보통 `add_subdirectory` 명령어를 사용한다.
- CMake script를 직접 추가하고 싶을 때는 `include` 명령어를 사용한다.

<br>

### add_subdirectory에 사용 가능한 옵션

``` cmake
add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
```

- `binary_dir`은 보통 잘 안쓰인다. 정말 필요한 경우가 아닌 경우에는 추천 안한다.

<br>

### include에 사용 가능한 옵션

``` cmake
include(<file|module> [OPTIONAL] [RESULT_VARIABLE <var>] [NO_POLICY_SCOPE]) 
```

- `OPTIONAL`이 있다면, 파일이나 모듈을 찾지 못했을 때 에러를 반환하지 않는다.
- `RESULT_VARIABLE`은 파일을 찾으면 full filename을 주고, 못찾으면 NOTFOUND를 준다.
- `NO_POLICY_SCOPE`는 외부 파일에서 CMake policy를 사용하고 싶을 때 사용한다.

---

## CMake 디버깅하기

``` cmake
message([<mode>] "message text" ...)
```

- `message`에 `<mode>` 파라미터를 설정함으로써 다양한 디버깅을 할 수 있다.
- 사용 가능한 `<mode>` 파라미터에는`TRACE`, `DEBUG`, `VERBOSE`, `STATUS`, `NOTICE`, `DEPRECATION`, `AUTHOR_WARNING`, `WARNING`, `SEND_ERROR`, `FATAL_ERROR`가 있다.
  - 이 중에 가장 많이 사용하는 것은 `STATUS`와 `FATAL_ERROR`이다.

### 간단한 디버깅 예시

``` cmake
message(STATUS a)
message(ERROR b)
message(FATAL_ERROR c)
message(STATUS a)
```

위와 같은 CMakeLists.txt를 작성했다면,

``` bash
$ cmake -GNinja ..
...
-- a
ERRORb
CMake Error at CMakeLists.txt:5 (message):
 c
 -- Configuring incomplete, errors occured!
```

이런 결과가 나온다.
`ERRORb`가 나온 이유는, `ERROR`는 `message`가 사용할 수 있는 `<mode>`가 아니기 때문에 string으로 취급되기 때문이다.

### message mode를 variable로 선언하기

``` cmake
set(MODE TRACE CACHE STRING "Message mode/level")
message(${MODE} "message(${MODE})")
```

- Cache variable을 통해 mode 값을 조정할 수 있다.
  - Cache variable에 대해서는 추후에 설명한다.

``` bash
$ cmake .. -GNinja -DMODE:STRING=STATUS
-- message(STATUS)
$ cmake .. -GNinja -DMODE:STRING=NOTICE
message(NOTICE)
```

### CMake의 모든 message 보기

``` bash
cmake --log-level=TRACE ...
```

위와 같은 방식으로 cmake를 돌렸을 때, 리눅스 환경에서는
- `STATUS`는 stdout으로
- `NOTICE`는 stderr로 출력된다.
  - cmake-gui를 쓴 경우에는 빨간색으로 출력된다.

---

# Variables

## Variables 사용하기

``` cmake
set(NormalVariable "A normal variable")
set(CacheVariable "A cache variable" CACHE STRING "Some description")
```

CMake에서 쓸 수 있는 variable에는 두가지가 있다.

- 보통의 variable
  - CMakeLists 내부에서만 사용 가능
- Cache variable
  - Project 외부에서 선언 가능함 (e.g. terminal에서...) 

### 사용 예시

``` bash
-- NormalVariable = A normal variable
-- CacheVariable = A cache variable
```

실제로 돌렸을 때는 위와 같이 나온다.

``` bash
$ grep -IrE "Variable" -C1 CMakeCache.txt
CMakeCache.txt-//Some description
CMakeCache.txt:CacheVariable:STRING=A cache variable
CMakeCache.txt-
```

Cache variable만 `CmakeCache.txt`에 저장된다.

### 같은 이름의 variable들

``` cmake
set(Variable "A normal variable")
set(Variable "A cache variable" CACHE STRING "Conflicts")
message(STATUS "CacheVariable = ${CacheVariable})

if (Variable STREQUAL "A cache variable")
    message(STATUS "Variable (${Variable}) is Cache")
else()
    message(STATUS "Variable (${Variable}) is Normal")
endif()
```

위와 같은 형태로 CMakeList를 만들었다고 해보자.
동일한 이름의 normal variable과 cache variable을 만들었다.

이 경우 작동은 하지만, 아래와 같은 참사가 나타난다.

``` bash
$ cmake ..
-- CacheVariable = A cache variable
-- Variable (A cache variable) is Cache

$ cmake ..
-- CacheVariable = A cache variable
-- Variable (A normal variable) is Normal
```

처음 돌렸을 때는 Cache variable이고,
두번째 돌렸을 때는 Normal variable로 나온다.

위와 같은 방식을 쓰지 말자!
스크립트를 돌릴 때 마다 매번 같은 결과가 나오는 좋은 스크립트를 짜도록 하자.

### set에 사용할 수 있는 옵션

``` cmake
# Set a normal variable
set(<variable> <value>... [PARENT_SCOPE])

# Set a cache variale (cache entry)
set(<variable> <value>... CACHE <type> <docstring> [FORCE])
# (Where <type> is one of BOOL, FILEPATH, PATH, STRING, or INTERNAL)

# Set an environment variable directly
set(ENV{<variable>} [<value>])
```

### Cache variable의 특징

Cache variable은 한번 만들고나면 `CMakeLists.txt`를 통해서 수정이 불가능하다.
왜냐하면 Cache variable 정보가 이미 `CMakeCache.txt`에 저장이 되어버렸기 때문에, 해당 파일을 수정하거나 지우지 않는 이상 cache variable 정보에 접근할 수 없다.

``` bash
$ cmake .. -DCacheVariable:STRING="Permanent override"
-- CacheVariable = Permanent override

$ cmake ..
-- CacheVariable = Permanent override

$ rm CMakeCache.txt

$ cmake ..
-- CacheVariable = A cache variable
```

### CMakeCache.txt의 특징

`CMakeCache.txt` 파일은 최상위 빌드 디렉토리에 있으며, 빌드에 대한 모든 정보를 담고 있다.

이 파일을 새로 만들거나 수정한다면, `configure`나 `generate`를 할 때 모든 `CMakeLists.txt` 파일이 다시 처리되어야한다.

---

## 다양한 variable 사용하기

- Variable의 종류에는 string, list, boolean이 있다.
  - List는 string들이 `;`로 분리되어있는 형태이다.

### String 사용하기

{% asset_img "string1.png" "string1" %}

{% asset_img "string2.png" "string2" %}

{% asset_img "string3.png" "string3" %}

### List 사용하기

{% asset_img "list1.png" "list1" %}

{% asset_img "list2.png" "list2" %}

---

# Control flow

CMake에 control flow 함수는 많지 않다.

``` cmake
if (A)
# ...
endif()
```

`x(...)`라는 control flow 함수가 있다면,
그 함수는 끝날 때 `endx()`로 끝나게 된다.

## Control flow 함수

- `if()` ; `else()` ; `elseif()` ; `endif()`
- `while()` ; `endwhile()`
- `foreach()` ; `endforeach()`
- `function()` ; `endfunction()`
- `macro()`; `endmacro()`
- `break()` ; `continue()` - 루프 컨트롤

## if문

``` cmake
if(<condition>)
    <commands>
elseif(<condition>)
    <commands>
else()
    <commands>
endif()
```

- 리턴 타입은 
  - True: 1, ON, YES, TRUE, Y, 또는 non-zero number
  - False: 0, OFF, NO, FALSE, N, IGNORE, NOTFOUND, 빈 스트링

   
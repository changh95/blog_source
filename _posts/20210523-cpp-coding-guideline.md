---
title: Modern C++ Coding Guideline
date: 2021-05-23 12:44:26
tags: [C++, Modern C++, cpp, Coding guideline]
categories: 
- [C++, 개발]
excerpt: 효율적으로 동작하는 / 협업에 효과적인 / C++ good practices를 가진 모던 C++ 코딩 가이드라인.
---

## 주의점

- 모든 edge case를 다 커버하지 않습니다.
- 특정 라이브러리에서 요구하는 특수 문법 (e.g. CUDA)로 인해 부분적으로 최적화가 되지 않는 부분이 생길 수 있습니다.
  - 개인적으로 사용할 때는, 해당 라이브러리 사용에 맞는 컨벤션을 추가하는 것이 좋습니다.
  - 협업을 할 때는, 팀원들과 상의를 통해 최대한 모두가 코드를 이해하고 리뷰할 수 있는 컨벤션을 만드는 것이 좋습니다.

&nbsp;

---

## 개발 환경

### 개발 환경 - IDE

- [**Visual Studio C++**](https://visualstudio.microsoft.com/ko/)
  - **가장 효과적인 디버깅**을 할 수 있습니다.
- [CLion](https://www.jetbrains.com/ko-kr/clion/)
  - 크로스 플랫폼 개발에 효과적입니다
  - CMake를 사용할 수 있어야합니다.
- [Visual Studio Code](https://code.visualstudio.com/)
  - 소프트웨어가 가볍습니다.
  - 언제까지 MS가 무료로 풀어줄지는 잘 모르겠습니다.
- QT
- Eclipse
- Android Studio
- [Vi(m)](https://www.vim.org/)
  - 모바일 로봇과 같이 SSH를 사용해서 원격 코딩해야할 때 좋습니다.

&nbsp;

### 개발 환경 - OS

- Windows
  - Visual Studio C++
  - MS HoloLens 프로그램이 할 때 좋습니다. (Windows SDK + C++ + C#)
- Linux
  - CLion or Vim
  - Visual Studio Code
    - 가벼운 에디터를 선호하거나
    - 또는 기타 다른 언어들도 하나의 에디터에서 사용해야할 때 (e.g. Python, Rust, Go, Scala)
  - 임베디드 / 로봇 포팅할 때 좋습니다.
  - ROS 프로그래밍 할 때 좋습니다.
- MacOS
  - 가능하면 피하세요!

&nbsp;

### 개발 환경 - Toolchain

- Windows
  - [Visual Studio C++](https://visualstudio.microsoft.com/ko/)를 설치하면서 기본적인 C++을 위한 툴체인 설치.
  - 크로스 플랫폼 개발을 위한 [CMake](https://cmake.org/)
  - [Resharper C++](https://www.jetbrains.com/ko-kr/resharper-cpp/)를 구매 및 설치해서 intellisense 성능 강화
- Linux
  - (**추천!**) [cv-learn's cpp-essential package](https://raw.githubusercontent.com/changh95/cpp-cv-project-template/main/scripts/cpp_tool_chains/install_essentials.sh)를 사용해서 `gcc`/`clang` 컴파일러 설치 및 `git`, `build-essential`, `cmake`, `cppcheck` 패키지 설치
  - (**추천!**) [Ignacio Vizzo's llvm toolchain install script](https://raw.githubusercontent.com/changh95/cpp-cv-project-template/main/scripts/cpp_tool_chains/install_llvm_toolchain.sh)를 사용해서 `clang-tools`, `clang-tidy`, `libclang`, `clang-format`, `python3-clang`, `clangd`, `OpenMP`, `libc++`, `lldb` 설치.

### 개발 환경 - Linter & Formatter

- `clang-format`을 사용해서 코드 작성 중 '저장'을 누를 때 마다 자동으로 줄바꿈, space 공백처리, tab 처리 등등 교정.
  - clang-format을 직접 만들고싶다면 [clang-format configurator](https://zed0.co.uk/clang-format-configurator/)을 사용해서 직접 만듬.
  - (**추천!**) 또는 [cv-learn's clang-format](https://raw.githubusercontent.com/changh95/cpp-cv-project-template/main/.clang-format)을 사용.
- Linux 환경에서는 `clangd`를 통해 static_analysis 수행
  - 코드에서 이상한 부분을 알려주거나, 헤더파일이 링크가 안된 부분을 상시 리포트 해줌.
  - Windows 환경에서는 Visual studio가 다 해줌.

&nbsp;

---

## 코드

### 코드 - 파일 이름

- `.cpp`와 `.hpp` 사용을 권장.
- Linux-only 코드인 경우 `.cc`, `.hh`도 가능.
  - 하지만 `.cpp`, `.hpp`와 혼용하지 않는 것을 적극 권장.

&nbsp;

### 코드 - include

- **include 자동 정렬은 비활성화**한다.
  - 특정 외부 라이브러리를 사용할 때 내부 생성 매크로로 인해 include 순서에 따라서 빌드 성공/실패 여부가 생길 수 있기 때문.
- 순서는 가능하면 **먼 순서에서 가까운 순서대로 선언**함. (i.e. 외부 라이브러리 -> 프로젝트 내부 소스 파일)
- "" 대신 <>를 사용하기.

```cpp
#include <opencv2/opencv.hpp>
#include <myHeader.hpp>
#include <vector>
```

&nbsp;

### 코드 - namespace

- double namespace는 최대한 피하기
  - 위 보다 아래가 더 좋은 방식

```cpp
namespace Vehicle
{
    namespace Dynamics
    {

    }
}
```

```cpp
namespace VehicleDynamics
{
    
}
```

&nbsp;

### 코드 - literal 값 표현 방식

- **int의 경우 정확한 비트 수를 포함하여 선언**하는 것이 크로스 컴파일 시 오류가 나타나지 않음.
  - 이 때문에 `char`, `short`, `size_t`, `int` 와 같은 타입은 잘 사용하지 않음.
- **float의 경우 f를 적어줘서 정확하게 값을 표현**해줌
  - `float num = 123.456`은 안됨.

```cpp
// int / uint.
int8_t a; // uint8_t a;
int16_t b;
int32_t c;
int64_t d;

// float
float num1 = 123.456f;
float num2 = 123.0f; // 'f' 이전에 0을 적어준다.
float num3 = 0.1234f;
```

&nbsp;

### 코드 - Class 설계

- 클래스 이름은 대문자로 시작하며 CamelCase로 작성한다.
- 클래스 멤버함수와 멤버변수들에 대한 설명은 [**doxygen**](https://www.doxygen.nl/index.html)을 사용하면 좋다.
  - 코드 내부에서도 통일된 방식으로 설명을 볼 수 있다.
  - 또, doxygen 빌드를 하면 자동으로 모든 함수/변수에 대한 설명이 적힌 웹사이트가 생성된다. (i.e. 개발자 문서 페이지를 쉽게 만들 수 있다)
- 한줄로 작성될 수 있는 코드는 한줄로 작성한다. (아래의 `getParam()`, `setParam()` 참조)
- 멤버변수에 접근하는 방법은 2가지 방법이 있다.
  - **Getter / Setter 함수를 사용**하는 방법
    - `int getParam() const;` 을 통해 클래스 외부에서 멤버변수 값을 읽는다.
    - `void setParam(int param);` 을 통해 클래스 외부에서 멤버변수 값을 수정한다.
  - **Public struct를 통해 값을 직접 접근**하여 설정한다.
  - `int& param() {return mParam;}`과 같이 **Reference 직접 접근하는 방식은 절대 사용하지 않는다** (엄청 위험하다)

```cpp
#include <vector>

namespace ORBSLAM
{

class FeatureDetector
{
public:
    using Keypoints = std::vector<Keypoint>;

    struct Data
    {
        int mIntParam = 0;       ///< 변수 설명
        double mDoubleParam = 0; ///< 변수 설명
    }

    /**
    * @brief 간단한 함수 설명
    * @param param 매개변수 설명
    * @return 리턴값 설명
    */
    bool detect(const Image& img);

    int getParam() const {return mParam};

    void setParam(int param) { mParam = param; };

private:
    int mParam = 0;  ///< 변수 설명
    Keypoints mKpts; ///< 변수 설명
    Data mData;      ///< 변수 설명
};
}
```

&nbsp;

### 코드 - 함수

- **함수의 이름은 해당 함수의 목적과 과정을 정확하게 담아낸다**.
  - `void removeOutliers()` 대신 `void removeOutliersbyReprojectionError()`가 더 좋다.
  - 함수의 이름이 길어져도 괜찮다.
- 함수 이름은 소문자로 시작하며, 이후 CamelCase로 작성한다.
- 함수가 한줄로 작성될 수 있는 경우에는, 함수 이름에서 대괄호를 이어서 작성한다.
  - `int getParam() const { return mParam };`
- 함수가 여러 줄로 작성될 때는, 함수 이름을 작성 후 그 다음줄에 대괄호를 적으며 작성한다.
- **`const`가 필요하지 않은 곳 외의 모든 곳에 `const`를 사용**한다.
  - `const`를 적음으로써 'const-함'을 표현하는 것이 아닌, **`const`를 사용하지 않음으로써 해당 함수의 특성을 표현**한다고 생각하자.
  - 이러한 특성이 두드러지게 표현해야 내 코드의 목적이 잘 전달된다. (코드 리뷰에서는 이러한 점이 중요하다)
  - `double evaluateModel(const Model& model, const Data& data);`
  - `const Images& getImages(const Images& imgs) const;`
- namespace, class, type 등과 겹치는 함수 이름은 피한다.
- `if`, `for`, `while`을 사용할 때 내용이 1줄만 사용된다면 대괄호를 사용하지 않는다.


&nbsp;

### 코드 - 함수 argument

- Pass by reference / pointer / value의 차이를 정확하게 인지하고 사용해야한다.
- **Pass by reference**
  - 복사 작업이 수행되지 않는다.
  - 그러므로, **클래스 인스턴스**를 읽거나 **큰 데이터를 읽을 때**는 **무조건 pass by reference를 사용**한다.
    - 원본 데이터를 유지하고 싶을 때는 `void foo(const Image& a)`
    - 원본 데이터를 수정하고 싶을 때는 `void foo(Image& a)`
- Pass by pointer
  - 데이터 내부의 특정 포인터 위치를 넘길 때 사용한다.
  - 또, 클래스 인스턴스를 옵셔널하게 넘길 때 사용한다. (`nullptr`이 넘겨진다면 빈 값이 넘겨지는 것)
    - `void foo(int a, Data* ptr = nulltpr)`
    - 옵셔널하게 클래스 인스턴스를 넘기는 상황이 아닌 경우에는 pass by reference를 사용하는 것을 권장한다.
- Pass by value
  - 복사 작업을 수행해야하는 경우에 사용한다.
    - `std::move`를 사용해서 `void foo(Data data)`를 쓸 수 있다. 하지만 권장하지 않음.
  - **Primitive 값들을 넘길 때 사용**한다.
    - `void foo(int a, double b)`

```cpp
// Pass by reference
void gaussianFilter(const Image& inputImage, Image& outputImage, const KernelMatrix& kernel); // outputImage는 이번 함수를 통해 수정될 값이라는 것을 강조. 동시에, 다른 매개변수는 수정되지 않을 것이라는 것을 강조.

// Pass by pointer
double evaluateModel(const Model& model, const Data* data = nullptr) // data는 optional한 값이라는 것을 강조. 
{
  if (data == nullptr) // 데이터가 들어오지 않은 경우에는 함수 조기 종료
    return std::numeric_limits<double>::max();

  ... // data에 어떤 값이 들어있는 거라면 제대로 된 model evaluation 수행
}

// Pass by value
void detectObjects(const Image& inputImage, int numObjects) // Primitive type은 거의 모든 경우 pass by value 사용.
```

&nbsp;

### 코드 - 변수

- 변수 이름은 소문자로 시작하며 CamelCase로 작성한다.
- 전역 변수 (global variable)은 global-을 뜻하는 `g-` prefix를 가진다. (e.g. gParam)
  - 전역변수를 사용하는 것은 추천하지 않는다.
- **클래스 멤버변수**는 member-를 뜻하는 **`m-` prefix**를 가진다. (e.g. mParam).
- Struct 멤버변수는 member-를 뜻하는 `m-` prefix를 가진다. (e.g. mParam).
- **Constant 변수**는 constant k value-를 뜻하는 **`k-` prefix**를 가진다. (e.g. kPi = 3.141592654;).

```cpp
class Object
{
public:
  ...

  void setParam(int intParam, double doubleParam)
  {
    mIntParam = intParam; // 'm-' prefix를 사용해 매개변수와 멤버변수를 구분. 거의 동일한 이름 사용 가능.
    mDoubleParam = doubleParam;
  }

private:
  int mIntParam; // 클래스 멤버 변수는 'm-' prefix를 사용
  double mDoubleParam;
}
```

&nbsp;

### 코드 - 템플릿

- 템플릿 코드를 작성할 때는 설명을 정확하게 작성하는 것이 중요하다.
- 템플릿 / 타입이름은 대문자로 작성한다.

```cpp
template <typename PRECISION = double, typename ESTIMATOR, typename CAMERAMODEL>
class Camera
```

&nbsp;

### 코드 - 에러 처리

- 임베디드 환경에서 성능 최적화를 위해 에러처리는 디버깅 단계에서 전부 확인해야한다.
  - 이 때문에 try & catch보다 **`assert()`를 사용**한다.

```cpp
void gaussianFilter(const Image& inputImage, Image& outputImage)
{
  // 이 함수는 inputImage에 저장된 값이 실제 이미지여야만 작동함.
  // assert를 이용해서 빈 데이터는 작동하지 않게 코드를 작성.
  assert(!inputImage.empty()); 
  
  if (outputImage.size() != inputImage.size())
    outputImage.setSize(ingputImage.size());

  ...
}
```

&nbsp;

### 코드 - Unit test

- **구현되는 모든 코드에 대해 unit test를 작성**한다.
  - [GTest](https://github.com/google/googletest) 프레임워크를 이용한다.
- 너무 뻔한 테스트는 생략해도 괜찮다. (int를 반환하는 함수가 정말로 int를 반환했는지 확인한다던지...)
- **Edge cases와 failure case를 가능한 많이 추가**한다.
- Unit test를 작성하는데에 **시간을 충분히 할애**한다.
  - Leaky unit test를 작성했다가 추후 unit test는 통과하는데 릴리즈 코드가 작동하지 않는다면... 해당 버그를 찾아서 해결하는 시간이 훨씬 더 오래 걸릴 것이다.

&nbsp;

### 코드 - 코드 최적화

- **처음부터 완벽한 코드를 작성하려고 하지 말자**.
- **코드를 작성하는 3단계**를 지킨다. (rough->fine->perfect)
  - 1차: **'1개의 케이스에서 잘 돌아가는 코드를 만들자'**
    - 유닛테스트 작성 + 코드 리뷰
  - 2차: **'고객이 요구하는 모든 케이스들 + edge 케이스에서 잘 돌아가는 코드를 만들자'**
    - 유닛테스트 작성 + 코드 리뷰
  - 3차: **'성능이 최적화된 코드를 만들자'**
    - 유닛테스트 작성 + 코드 리뷰

&nbsp;

### 코드 - STL, Containers, 모던 C++ 문법

- 직접 구현하지 말고 **STL 함수가 있는지 꼭 먼저 찾아보자**.
  - 컴파일러들은 STL 함수에 대해 최적화가 정말 잘 되어있다.
  - 거의 모든 경우 내가 짠 코드보다 STL 함수가 더 최적화가 잘 되어있다.
- **`using namespace std`는 절대 쓰지 말자**. 
  - conflict 생길 수 있다.
- STL을 사용하면서 코드 가독성이 떨어진다면 주석으로 충분히 설명을 잘 해주자.
  - 가능하면 주석 없이도 이해할 수 있도록 코드를 쪼개주자.
- **목적에 맞는 컨테이너를 사용**하자.
  - `std::vector`, `std::unordered_map`, `std::map`...
  - `std::unordered_map`의 경우 컴파일러마다 구현 방법이 다르기 때문에 결과가 다를 수 있다. 가능하면 직접 hash-function도 만들어주자.

```cpp
std::vector<int> data;

// C++98 시대의 방법. 가독성이 떨어진다.
for (int i = 0; i < data.size(); i++)
    foo(data[i]);

// 모던 C++의 range-based for loop 방법. 가독성이 좋다.
for (const auto& d: data)
    foo(d);

// STL의 방법. 코드 효율성은 높아지나 가독성은 떨어진다.
std::for_each(data.begin(), data.end(), [this](const auto& i){ foo(i); }; )
```

&nbsp;

### 코드 - Enum class / strong types

- Type을 만들 때는 Strong type의 사용을 적극 권장한다 (i.e. **Enum class 사용을 권장**한다)
  - Enum만 사용하는 것은 권장하지 않는다.
- Enum class의 멤버들은 대문자와 `_`를 사용한다.

```cpp
/// Doxygen을 위한 enum class 설명
enum class DetectionStatus : std::int8_t
{
    NOT_DETECTED = 0; ///< 변수 설명
    SUCCESS = 1;      ///< 변수 설명
    FAIL = 2;         ///< 변수 설명
}
```

&nbsp;

### 코드 - 타입 캐스팅

- Explicit하게 타입 변화를 해주자.
  - `static_cast`, `dynamic_cast`

```cpp
int i = 10;
double j = (double)i; // 사용 금지
double j = static_cast<double>(i); // 권장
```

&nbsp;

### 코드 - Log

- `std::cout`은 굉장히 느린 편이므로 [**fmt**](https://github.com/fmtlib/fmt)를 사용한 로깅 라이브러리를 사용하는 것이 좋다.
  - e.g. fmt 기반으로 만들어진 [**spdlog**](https://github.com/gabime/spdlog)가 사용하기 좋다.

&nbsp;

### 코드 - 모던 C++

- 매크로
  - **`using DEFINITION = MACRO;`를 사용**한다.
    - C 방식의 `#define MACRO DEFINITION`은 사용하지 않는다.
- 출력
  - `std::cout`을 사용한다.
    - C 방식의 `printf`는 사용하지 않는다.
    - Log 목적이라면 위에 설명한 **spdlog 사용을 적극 권장**한다.
- 포인터
  - **`std::unique_ptr`, `std::shared_ptr`의 사용을 적극 권장**한다.
    - C 방식의 Raw pointer의 사용은 가능한 피한다.
    - C++ 방식의 `new`, `delete`의 사용도 가능한 피한다.
- String
  - **`std::string`의 사용을 권장**한다.
    - C 방식의 `const char*`는 사용하지 않는다.
    - 외부 라이브러리 사용을 위해 C 방식의 char가 필요하다면, std::string으로부터 `c_str()` 함수를 사용해서 바꾼다.
- Array (Stack)
  - **`std::array`의 사용을 권장**한다.
    - C 방식의 `int[]`는 사용하지 않는다.
- For loop
  - **Range-based for loop의 사용을 적극 권장**한다.

&nbsp;

### 코드 - 성능 측정 + 최적화

- 코드에서 생기는 성능 bottleneck은 profiler로 찾아서 제거한다.
  - Linux에서는 [Hotspot](https://github.com/KDAB/hotspot)을 자주 사용한다.
  - 크로스 플랫폼 + 멀티코어 환경에서는 [EasyProfiler](https://github.com/yse/easy_profiler)를 사용한다.
  - GPU 프로그래밍의 경우는 해당 GPU 벤더사에서 프로파일링 도구를 제공하기도 한다. (e.g. [NVIDIA nSight Visual profiler](https://developer.nvidia.com/nvidia-visual-profiler))

---

## 협업

### 협업 - 형상관리

- [Github](https://github.com/), [Gitlab](https://about.gitlab.com/), [Bitbucket](https://bitbucket.org/)과 같은 **Git 기반 온라인 코드 저장소를 사용하여 형상관리**를 한다.
  - Git에 대한 이해도가 있어야한다.
    - [Git 사용방법](https://git-scm.com/book/ko)을 보고 배우면 좋다.
  - [GitKraken](https://www.gitkraken.com/), [Github Desktop](https://desktop.github.com/)과 같은 GUI 앱을 사용해도 괜찮지만, **가능하면 위의 방법으로 CLI을 익힌다**.
    - CLI 기능 중 GUI에 구현되지 않은 기능들이 많기 때문이다.
- Repository 관리 전략
  - **main 브랜치는 릴리즈를 위한 브랜치**이다.
    - 고객이 보는 브랜치이기 때문에, 절대로 버그가 있어서는 안된다.
    - **admin을 제외한 그 아무도 merge를 수행할 수 없다**.
  - development 브랜치는 현재 팀이 개발중인 가장 최신의 브랜치이다.
  - feature / individual 브랜치는 개개인이 작업중인 브랜치이다.
    - 하나의 feature 브랜치에는 하나의 기능만 개발한다.
    - **Pull request를 통해서 development 브랜치로 작업물을 통**합할 수 있다.
      - Merge into branch 방식을 써서 통합할 수 있다.
      - Rebase branch 방식을 써서도 통합할 수 있다.

&nbsp;

### 협업 - 코드리뷰

- **코드 리뷰는 Pull request를 통해 진행**된다.
  - 해당 코드 개발과 관련된 인원, 또는 해당 코드를 리뷰해줄 수 있는 인원들로 **최소 2명에게 코드 리뷰**를 받는다.
  - 코드가 아직 준비되지 않은 상황이라면 draft Pull request로 만든다.
  - Pull request에는 **1. 해당 작업의 배경과 목적**, **2. 수행한 개발 내용 정리**, **3. 해당 코드를 통해 기대하는 효과**를 적는다.
    - 실험한 내용이 있다면 이 역시 첨부한다.
- 코드 리뷰의 내용은 다음과 같은 내용을 가지며 **1. 코드 퀄리티의 증진**, **2. 개발된 내용이 실제 목표를 달성하는지를 확인**하는데에 목적을 둔다.
  - 가독성 증진을 위한 함수/변수 이름 수정
  - assert 등의 조건문 추가
  - 코드 효율성 증가
  - 개발 내용 확인
  - 등등...
- **Pull request에서 제안하는 변화는 가능한 작고 짧게 만든다**.
  - **업데이트되는 코드가 너무 많고 복잡하면** 제대로된 리뷰를 하기가 어려워진다.
    - 이 때, 리뷰어는 **리뷰를 거부할 수 있다**.
- 리뷰는 코드를 작성하는것 만큼 정말 중요하다.
  - **업무 시간의 30~40%는 동료의 코드를 리뷰하는데에 사용**해도 된다.
- 두명 이상에게서 approve를 받는다면 merge를 진행한다.

&nbsp;

### 협업 - CI/CD

- [Jenkins](https://www.jenkins.io/), [TeamCity](https://www.jetbrains.com/teamcity/) 등과 같이 agent 할당 프레임워크를 이용해서 CI/CD 빌드 스트림을 만들어둔다.
- Pull request가 들어오면 CI/CD는 [Github Actions](https://github.com/features/actions)를 통해 변화를 감지하고 자동으로 빌드를 수행한다.
  - 크로스 플랫폼 개발의 경우 **여러 환경에 대한 빌드를 모두 성공해야 merge가 가능**하다.
  - 단일 플랫폼 개발의 경우 **다양한 컴파일러들에 대해 빌드를 수행**하고, 각각의 빌드마다 성능을 측정하여 **가장 좋은 성능을 보여주는 컴파일러 빌드를 사용하여 릴리즈** 할 수 있다.
    - 성능 벤치마크 스크립트를 작성하면 좋다.

{% asset_img "gh-action.png" "gh_actions" %}
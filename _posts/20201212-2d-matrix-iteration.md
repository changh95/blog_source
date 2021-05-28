---
title: (번역) 실시간 컴퓨터 비전을 위한 C++ 최적화 (9) - 이미지 속 픽셀 계산 빨리 하는 방법 (또는 2D matrix 계산 빨리 하는 방법)
date: 2020-12-12 22:38:19
tags: 
- C++
- CV
- Robotics
- Data Structure
- Optimisation
categories: 
- [C++, 실시간 컴퓨터 비전을 위한 C++ 최적화 시리즈]
excerpt: Davide Faconti의 CPP Optimization Diary 블로그 글 중 "Iterating over 2D matrix`"을 적당히 번역했습니다.
---

Davide Faconti의 [CPP Optimization Diary 블로그](https://cpp-optimizations.netlify.app/) 글 중 "Iterating over 2D matrix"을 적당히 번역했습니다. 원글 링크는 [여기](https://cpp-optimizations.netlify.app/2d_matrix_iteration/)를 봐주세요.

<br>

# 2D matrix iteration 하기
---

2D matrix는 로보틱스에서 엄청나게 자주 사용됩니다.

이미지, gridmap/costmap 등등을 2D 매트릭스로 표현하지요. 

최근에 저는 우리 팀에서 costmap 계산에서 사용되는 알고리즘이 꽤 느리다고 느꼈고, **Hotspot**을 이용해서 프로파일링을 해봤습니다.

두개의 costmap을 합쳐서 새로운 costmap을 만드는 과정에서 바틀넥이 있었습니다.

대충 이런 코드였죠.

```C++
// 대충 적은 코드입니다.
for( size_t y = y_min; y < y_max; y++ ) 
{
    for( size_t x = x_min; x < x_max; x++ ) 
    {
        matrix_out( x,y ) = std::max( mat_a( x,y ), mat_b( x,y ) ); 
    }
}
```

<br>

이해하기는 쉬운 코드였습니다.

잘 적었으니까요. 그쵸? 

하지만 이 계산은 실제로 돌 때 너무 많은 CPU 사이클을 잡아먹고 있었습니다.

어쩔수 없이 저는 최적화 여행을 또 떠나게 되었지요 (휴)

<br>

## C++에서 좋은 2D 매트릭스 만드는 방법
---

이런 코드를 적어본 적 있으시나요?


```C++
// 컴공 학생 때 교수님은 이런 코드를 적으셨는데요...
float** matrix = (float**) malloc( rows_count * sizeof(float*) );
for(int r=0; r<rows_count; r++) 
{
    matrix[r] = (float*) malloc( columns_count * sizeof(float) );
}
// element access는 이렇게 합니다
matrix[row][col] = 42;
```

<br>

이러면 망한겁니다.

이 코드를 본 이상, 어쩔 수 없이 이 코드를 USB에 담아서 [모르도르로](https://en.wikipedia.org/wiki/Mount_Doom) 가져가 용암 속에 던질 수 밖에 없어요.

{% asset_img "mordor.jpg" "이 코드는 불태워버려야합니다" %}


<br>

우선, 저희는 [Eigen 라이브러리](http://eigen.tuxfamily.org)를 사용해보는 것을 고려해봐야합니다. Eigen 라이브러리는 성능도 좋고, 이해하기 쉬운 API를 가지고 있어요.

실사용은 Eigen으로 한다지만, 공부를 하기 위한 목적으로는 물론 직접 짜봐야겠지요 (하지만... 진짜 그냥 Eigen 쓰시는게 좋아요. 진심으로.). 아래 코드에 예시가 있습니다.

<br>

```C++
template <typename T> class Matrix2D
{
public:
    Matrix2D(size_t rows, size_t columns):  _num_rows(rows)
    {
        _data.resize( rows * columns );
    }
    
    size_t rows() const
    { 
    	return _num_rows; 
    }
    
    T& operator()(size_t row, size_t col)  
    {
        size_t index = col*_num_rows + row; 
        return _data[index];
    }
    
    T& operator[](size_t index)  
    {
        return _data[index];
    }
    
    // 다른 method는 딱히 설명하지 않겠습니다.
private:
    std::vector<T> _data;
    size_t _num_rows;
};

// element access는 이렇게 하시면 됩니다.
matrix(row, col) = 42;

```

<br>

이 방법이 가장 캐시메모리 사용에 유용한 매트릭스 생성 방법입니다. Single memory allocation에, 데이터는 [column-wise](https://www.geeksforgeeks.org/row-wise-vs-column-wise-traversal-matrix/)방식으로 저장됩니다.

우리가 쉽게 이해하는 row/column 값을 vector의 index값으로 변환하기 위해서는, 단 한번의 곱셈연산과 덧셈연산이 필요합니다.

<br>

## 원래 문제로 돌아가서...
---

처음에 보여드렸던 제 코드로 돌아가봅시다.

우리는 여러번 iteration을 해야하는데, 매번 `(x_max-x_min)*(y_max-y_min)` 계산을 할 겁니다.

종종, 꽤 많은 pixel 또는 cell에 대한 계산이 들어가겠죠.

매 iteration마다 우리는 다음과 같은 방법으로 index 값을 세번 계산해야합니다.  

     size_t index = col*_num_rows + row;

곱셈 계산이 엄청나게 많군요! 좋지 않습니다.

<br>

아래와 같은 코드로 바꿔보았습니다.

```C++
// index를 직접 계산해봅시다.
for(size_t y = y_min; y < y_max; y++) 
{
    size_t offset_out =  y * matrix_out.rows();
    size_t offset_a   =  y * mat_a.rows();
    size_t offset_b   =  y * mat_b.rows();
    for(size_t x = x_min; x < x_max; x++) 
    {
        size_t index_out =  offset_out + x;
        size_t index_a   =  offset_a + x;
        size_t index_b   =  offset_b + x;
        matrix_out( index_out ) = std::max( mat_a( index_a ), mat_b( index_b ) ); 
    }
}
```
<br>

무슨 생각을 하고 계신지 압니다. ***제 눈도 아파요***.

코드가 엄청 더럽죠.

하지만 이로써 얻는 성능 개선은 무시할 수 없는 정도입니다.

사실 당연한 결과일 수도 있겠습니다. 곱셈연산의 수가 3배나 줄어들었으니까요.

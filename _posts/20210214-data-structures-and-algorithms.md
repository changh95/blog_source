---
title: 자료구조와 알고리즘 이해
date: 2021-02-14 20:38:19
mathjax: true
tags: 
- Programming
- Data structures
- Algorithms
categories: 
- [  SW, 알고리즘/자료구조]
excerpt: 자료구조와 알고리즘에 대해 전반적으로 이해하기.
---

# 왜 자료구조와 알고리즘을 배워야하는가?

자료구조: 효율적인 데이터 접근 및 수정 방법을 제공
알고리즘: 문제를 풀기 위한 효율적이고 빠른 방법을 제공

자료구조와 알고리즘을 알고 있다면 문제를 효율적인 방법으로 풀 수 있음.
자료구조와 알고리즘을 알지 못하는 프로그래머는 효율적인 방법으로 풀 수 없으며, 왜 본인의 코드가 비효율적인지 알지 못함.

<br>

---

# 알고리즘 효율성의 척도 - Big O

Array에 담긴 모든 숫자들의 합을 구하는 함수를 짰다고 해보자.

`number_list`에 10개의 값이 있다면, 이 함수가 끝나기 위해서는 10개의 값을 모두 탐색해야한다. 20개의 값이 있다면 20개의 값을 모두 탐색해야한다.


``` python

def find_sum(number_list):
    total_sum = 0
    for num in number_list:
        total_sum += num
    return total_sum

```

{% asset_img "sum.jpg" "find_sum" %}

이번에는 array의 첫번째 값만 읽는 함수를 짰다고 해보자.

이번에는 `number_list`의 크기와 상관없이 무조건 첫번째 값만 읽으면 된다.


``` python

def first_num(number_list):
    return number_list[0]

```

{% asset_img "first_elem.jpg" "first_num" %}


(물론 이 두 함수의 목적은 달랐지만) 이 두 함수는 다른 **Time complexity**를 가지고 있다. Time complexity는 **인풋 데이터의 크기가 커짐에따라 바뀌는 계산량의 관계**를 뜻한다. 우리는 이를 **Big O notation**을 통해 표현한다.

Big O notation에는 다음과 같은 종류가 있다. (효율적인거 부터 비효율적인 순서대로)

- O($1$)
- O($\log n$)
- O($n$)
- O($n \log n$)
- O($n^2$)
- O($2^n$)
- O($n!$)

방금 전 `find_sum()` 함수는 O($n$), `first_num()` 함수는 O($1$)이였다. 

실제로 인풋 데이터의 크기인 $n$값이 올라가면서 얼마나 데이터를 탐색해야하는지 비교를 해보면 알고리즘의 효율성에 대해 좀 더 직관적으로 볼 수 있다. n의 수가 작을 때에는 큰 효과를 보지 못하지만, n의 크기가 점점 커지면서 효과는 기하급수적으로 커진다. n = 5인 경우와 n = 50인 경우를 비교해보자.

{% asset_img "time_complexity.jpg" "time_complexity" %}


Big O notation을 사용하여 알고리즘을 비교할 때 우리는 상수 값은 보통 포함시키지 않는다. O($n$)이나 O($2n$)이나 둘 다 결국 linear하기 때문에, 다른 계수와 비교하는게 아닌 이상 큰 차이를 만들지 않기 때문이다.

Big O notation은 **Space complexity**를 표현하는데에도 사용 가능하다. Space complexity는 **인풋 데이터의 크기가 커짐에 따라 바뀌는 데이터 저장 공간 크기의 관계**를 뜻한다.

<br>

---

# Logarithm (+ Binary search 이진탐색)

Log는 수학에서 Exponential의 반대되는 개념이라고 볼 수 있다. Log가 갑자기 왜 알고리즘 / 자료구조에 나오는지 이해하기 어려울 수 있다. 이해를 돕기 위한 예시를 한번 들어보자.

종이 사전에서 단어를 찾을 때를 생각해보자. 가장 간단한 방법으로는 사전의 첫장부터 모든 단어 하나하나 살펴가며 내가 찾는 단어인지 비교하는 방법이 있다. 이 방법은 O($n$)이 걸릴 것이며, 종이사전이 10만개의 단어를 가진다면 10만번의 탐색을 해야할 것이다. 하지만 사람이라면 이렇게 하지 않는다. 

{% asset_img "linear_search.jpg" "linear_search" %}

좀 더 좋은 방법은, 종이사전의 반씩 쪼개는 방법이다. 내가 찾는 단어가 종이사전의 첫 50%에 있는가, 아니면 뒤 50%에 있는가? 첫 50%에 있다면 뒤 50%는 버리고, 첫 50%에서 또 반을 갈라서 고른다. 이 과정을 단어를 찾을 때 까지 반복한다. 이 방법은 O($\log n$)이 걸리며, 종이사전이 10만개의 단어를 가질 경우 약 17번 정도의 탐색 끝에 단어를 찾을 수 있다. 또, 이 경우 종이사전의 크기가 20만개로 늘어난다고 해도 탐색은 1번만 더 하면 된다.

{% asset_img "bst.jpg" "bst" %}

이 방법은 컴퓨터 과학에서는 **Binary search**라고 한다.

``` python

def binary_search(arr, target):
    start, end = 0, len(arr) - 1
    while start <= end:
        mid = (start+end)//2
        if target == arr[mid]:
            return mid
        elif target < arr[mid]:
            end = mid - 1
        else:
            start = mid + 1
    return -1

```

Binary search는 효율적이지만, 데이터가 사전에 **정렬**되어있어야한다.

<br>

---

# Sorting (정렬)

Sorting을 구현하는데에 가장 쉬운 방법은 **selection sort**이다. 이 방법은 모든 데이터를 나머지의 모든 데이터와 비교하며 정렬을 수행한다. 1. 모든 데이터를, 2. 모든 나머지 데이터들과 비교를 해야하니, O($n * n$)이 되며, 이는 O($n^2$)이 된다.

{% asset_img "selection_sort.jpg" "selection_sort" %}

``` python

def selection_sort(arr)
    for i in range(len(arr) -1):
        min_index = i

        for j in range(i+1, len(arr)-1):
            if arr[j] < arr[min_index]:
                min_index = j

        arr[i], arr[min_index] = arr[min_index], arr[i]

```

좀 더 효율적인 방법에는 divide & conquer 방식인 **Merge sort**가 있다 (코드는 [ratsgo님 블로그](https://ratsgo.github.io/data%20structure&algorithm/2017/10/03/mergesort/)를 참조했습니다). Merge sort는 array는 재귀적으로 반으로 계속 쪼갠 후, 쪼개진 값들을 constant time으로 비교하여 sort를 진행한다. 총 $\log n$개의 쪼갬 작업이 있고, 2개의 정렬된 리스트를 병합하는데에는 O($n$)만큼의 계산이 필요하므로, 총 O($n * \log n$)의 복잡도를 가진다.

부분적으로 sort하는게 아닌 이상 보통 merge sort가 많이 사용되며, O($n \log n$)을 sorting 의 복잡도로 생각한다.

{% asset_img "merge_sort.png" "merge sort" %}

``` python

def merge_sort(list):
    if len(list) <= 1:
        return list
    mid = len(list) // 2
    leftList = list[:mid]
    rightList = list[mid:]
    leftList = merge_sort(leftList)
    rightList = merge_sort(rightList)
    return merge(leftList, rightList)

def merge(left, right):
    result = []
    while len(left) > 0 or len(right) > 0:
        if len(left) > 0 and len(right) > 0:
            if left[0] <= right[0]:
                result.append(left[0])
                left = left[1:]
            else:
                result.append(right[0])
                right = right[1:]
        elif len(left) > 0:
            result.append(left[0])
            left = left[1:]
        elif len(right) > 0:
            result.append(right[0])
            right = right[1:]
    return result

```

유명한 sorting 방식중에는 Quick sort 도 있다. 평균 O($n \log n$)의 복잡도를 가지지만, worst case (이미 sorting된 케이스)에서는 O($n^2$)를 가진다.

{% youtube XE4VP_8Y0BU %}

<br>

---

# Arrays, Linked List

**Array**는 가장 간단한 자료구조라고 볼 수 있다. 데이터들을 이어지게 하나로 쭉 늘어놓은 형태를 뜻하며, 생성 시 크기를 정한다. 10개의 크기를 가진 array에 11번째 데이터를 넣는 것은 불가능하다.

{% asset_img "array.jpg" "array" %}

**Linked list**는 크기를 원하는대로 늘이고 줄일 수 있는 자료구조이다. 기본적으로 여러개의 node를 연결해둔 형태를 가지고 있고, 하나의 node에는 저장하고 싶은 데이터와 다음 node로의 메모리 주소 값을 저장한다. 쉽게 이해하려면, 하나의 하이퍼링크를 타고 갔더니 또 다른 하이퍼링크가 나오고, 그걸 타고 갔더니 또 하이퍼링크가 나오고, 또 타고 갔더니 또 나오는 형태로 보면 된다. 그러다가 나타나는 다음 node로의 메모리 주소가 NULL이라면 그것이 linked list의 마지막이라고 볼 수 있다.

{% asset_img "linked_list.jpg" "linked list" %}

Linked list에 새로운 값을 추가하려면 마지막 node에 다음 주소값만 적어주면 되기 때문에 constant time (i.e. O(1))이라고 볼 수 있다. 지우는 것도 하나의 주소값만 지워주면 되기 때문에 역시 O(1)이다. 이처럼 빠르게 보이지만, linked list는 특정 index의 값을 불러오기 위해서 모든 데이터를 거쳐야하기 때문에 O($n$)이 소요된다. 

이렇게 어떤 특정 index 값을 불러오는 것은 array가 O(1)로 더 빠르다. 하지만 array의 경우, 대부분의 프로그래밍 언어는 확장성을 추가한 `std::vector` 등의 형태로 구현하는데 (확장성이 없는 `std::array`도 있음), 이는 생성시 정해둔 크기를 초과하여 데이터를 추가할 때 기존의 크기의 2배가 되는 메모리를 새로 할당하고 기존의 데이터를 모두 복사해야한다. 이 경우 복사하는데에 시간이 많이 소요되지만, 언제 나타날 지 모르기 때문에 우리는 **평균 O(1)**을 가진다고 한다.

<br>

---

# Tree

**Tree**는 하나의 node가 다음 n개의 node로의 메모리 주소를 가지고 있다. 이는 linked list가 1개의 다음 node로의 메모리 주소를 가지는 점과 다르다고 볼 수 있다. 어떠한 tree node가 다음 2개의 node로의 메모리 주소를 가진다면 binary tree가 된다. 다음 node는 또 그 다음 2개의 메모리 주소를 가지는데, 이는 tree가 재귀적인 특성을 가지기 때문에 subtree라고 부르기도 한다. 다음 node로의 주소가 없는 경우는 leaf node라고 부른다.

**Binary search tree**는 자주 사용되는 Tree 구조로써, 하나의 node가 2개의 child node를 가지며, 좌측 child node는 더 작은숫자, 우측 child node는 더 크거나 같은 숫자를 가진다. Binary search tree에서 어떠한 값의 데이터를 찾을 때는 내가 현재 위차한 node보다 크거나 작은지만 보면 된다. 모든 branch의 높이가 같은 것은 아니며, 최악의 경우 branch의 높이만큼 이동해야한다. 그렇기 때문에 binary search tree는 1. 탐색, 2. 데이터 추가, 3. 데이터 삭제에 모두 O(H)만큼 소요되고, Height는 $log n$과 비례하기 때문에 O($log n$)이 된다. 

{% asset_img "bst2.jpg" "binary search tree" %}

종종 sorting이 완료된 값을 binary search tree로 넣을 때 (e.g. [1,2,3,4]) binary search tree의 법칙을 따랐음에도 linked list처럼 구현될 수 있다 (2는 1보다 크기 때문에 우측에, 3은 2와 1보다 크기 때문에 2의 우측에, 4는 3과 2와 1보다 크기 때문에 3의 우측에). 이렇게 되면 O($n$)이 되어버려 비효율적이게 되니, 자동으로 좌/우를 밸런싱해서 tree height가 O($log n$)으로 수렴하게 해주는 red-black tree와 같은 기법을 사용하면 좋다. 

{% asset_img "balanced_tree.jpg" "balanced tree" %}
 
**Heap, 또는 Priority Queue**는 parent node가 child node보다 더 중요하거나 동일한 우선도를 가지는 경우를 뜻한다. Heap의 root node는 항상 제일 높은 우선도를 가지는 값이다. Min heap의 경우 최소값이 root node로 가고, max heap의 경우 최대값이 root node로 간다. Root node의 값을 얻기위해서는 값을 1개만 보면 되기 때문에 O(1)이 소요된다. Heap에 새로운 값을 추가하거나 뺄 때는 tree의 가장 아랫단 빈 곳에 임의로 추가하고, tree를 올라가면서 priority를 비교하며 위치를 교환하는 방식이다. 이 방식은 O($log n$)이 소요된다. 이 방식의 단점은 root node를 제외한 나머지 node들이 sorting이 되지 않아서 나머지 정보를 사용하기 어렵다는 점이며, 이 때문에 탐색을 할 경우 unsorted array의 탐색과 똑같은 O($n$)만큼의 시간이 걸린다. 그렇기에 보통 root node만 중요할 때 이 자료구조를 사용하는 편이 많다.

<br>

---

# Depth-first search vs Breadth-first search (깊이탐색 vs 너비탐색)

Tree를 탐색하는데에는 두가지 방법이 있다. **Depth-first search**와 **Breadth-first search**이다. Tree를 찾는데에 처음부터 최대한 깊게 들어가서 최하레벨 element부터 찾는게 depth-first, 아니면 최대한 상위레벨에서 하나씩 보면서 천천히 들어가는게 breadth-first라고 볼 수 있다.

{% asset_img "depth_breadth.jpg" "depth vs breadth" %}

Depth-first는 **stack**이라는 list 자료구조 형태로 구현된다. Stack은 2가지 작업을 할 수 있는데, list의 끝단에 새 값을 추가 (i.e. push)하거나 값을 지우는 (i.e. pop) 작업을 할 수 있다. 이러한 방식을 **LIFO** - Last in, first out이라고 한다.

Breadth-first는 **queue**라는 자료구조 형태로 구현된다. Queue는 list 끝단에 새 값을 추가 (i.e. enqueue)와 list 최앞단 값을 지우는 (i.e. dequeue) 작업을 할 수 있다. 이러한 방식을 **FIFO** - first in, first out라고 한다.

<br>

---

# Graphs (그래프)

Graph는 데이터 값들을 node처럼 다루고, 이들을 edge로 연결해둔 형태를 뜻한다. 모든 tree는 graph지만, 모든 graph가 tree는 아니다. A->B 형태로 'A에서 B'를 구현할 수도 있고, A-B 형태로 'A에서 B 또는 B에서 A'를 구현할 수도 있다. Edge를 구현하는 방법 중에는 Adjacency list와 Adjacency matrix 가 유명하다. 

Graph는 node간의 거리를 표현하여 **최단거리 계산**을 하는데에 효과적이다. 

예를 들어, 내가 인맥을 통해 JYP 박진영을 알기위해 인맥의 몇단계를 건너야하는지를 생각해보자. 내가 우리 팀장님을 통해 업계 사람 4명을 거쳐 JYP를 알게되는 루트가 있다고 해보자. 그리고 내가 JYP 팬클럽에 가입한 친구를 통해 2명만 거쳐서 JYP를 알게되는 루트가 있다고 보자. 이 경우 최단거리로 내가 JYP와 이어진 거리는 나-친구-친구지인-JYP 이기 때문에 3이다. 

{% asset_img "jyp.jpg" "jyp" %}

종종 graph의 edge에는 weight가 있을 수 있다. 예를 들어서, 내가 출근을 하려고 하는데 버스를 탈 경우 한번에 20분이 걸려서 도착하고, 지하철을 탈 경우 한번 갈아타서 8분 + 8분이 걸린다고 해보자. 이 경우에는 Dijkstra 알고리즘 등으로 weight를 고려해서 풀 수 있다. 

{% asset_img "dijkstra.jpg" "dijkstra" %}

<br>

---

# Hash map (해쉬맵)

**Hash map**은 기존의 array구조에 key-value 구조를 추가함으로써, 데이터를 추가/삭제/탐색하는데에 평균 constant time O(1)로 데이터를 탐색할 수 있는 방법이다. Hash map은 hash function을 통해 key값에 대한 *유일한* hash code 값을 만들어낸다. 이 hash code 값을 index로 사용해서 value 값들을 저장해둘 수 있다. 당연하게도, 동일한 key 값에 대해 hash code는 동일하게 나와야한다.

{% asset_img "hash_map.png" "hash map" %}

위에 *유일한* hash code라고 적으며 강조를 했는데, 이는 종종 다른 key 값들로 hash code를 만들었는데, 동일한 hash code가 만들어지는 경우이다. 이 경우 동일한 hash code에 여러 value 값을 저장하는데, 이 때 linked list 형태로 저장해야하기 때문에 (linked list는 O(n)을 가지고 있으므로), O(1)으로 값을 받지 못하게 된다. 이 현상을 hash collision이라고 한다. 이 때문에 O(1)을 유지하고 싶으면 hash function을 잘 설계해야하며, 어쩔 수 없이 hash collision이 나타나게되면 O(1)이 유지가 되지 않으므로 '평균 O(1)'이라고 하는 것이다. 파이썬의 Dictionary, C++의 std::unordered_map이 hash map을 사용한다.

---

출처: [Tren Black 영상 - Data Structures and Algorithms in 15 Minutes](https://youtu.be/oz9cEqFynHU)


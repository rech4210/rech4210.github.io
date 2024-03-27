---
layout: post
title:  "[자료구조/ 알고리즘] Heap, Priority queue"
date: 2023-09-20 12:17:04 +0900
categories: Algorithm
tags : Algorithm, Data Structure

---

## Priority queue
우선순위를 가지는 큐이다.

array, linked list, heap으로 구성할 수 있다.
array의 경우 요소 변경에 어려움이 있다.
linked list의 경우 요소의 개수가 많아짐에 따라 탐색이 오래 걸린다.

10000명의 선수를 대상으로 일정 수의 사람을 선별한다고 할 때 head과 linked list의 차이는 확연히 벌어진다.

O(Nlog N)
O(Nlog N + N)으로 선별 + 우선순위 설정 과정이 추가된다.

그러므로 보통 heap을 통해 구성한다.


## Heap
우선순위 큐를 위해 만들어진 자료구조이다.
tree 구조를 사용하는데 이는 요소간의 관계성을 유지하기 위해서이다.


힙의 최소 조건
1. 완전이진트리의 조건
	1-1. 트리의 왼쪽 -> 오른쪽으로 데이터가 채워져야한다.
	1-2. 마지막 레벨을 제외하고 모든 노드가 채워져야한다.
3. 이진 트리일것.
4. Heap order Property
	4-1. parent >= children 속성을 만족해야한다.


###  Heap as an Array
heap의 관계성을 유지하면 이를 array에 대입할 수 있게 된다.

위의 내용을 보면 array로 구현시 요소 변경과 탐색에 어려움이 있다 적어놨는데...
사실은 heap의 내부 데이터는 배열로 저장되며 이는 꽤 효율적으로 작용된다.

그러니까 heap이라는 구조 안에 데이터 컨테이너를 array로 쓰는것으로 생각하면 될듯하다.


```cpp
class Heap{
    vector<int> v;
public:
    Heap(int default_size=10){
        v.reserve(default_size+1);
        v.push_back(-1);
    }
 };
```

Heap의 구조에 vector를 사용하는 까닭은 heap이 완전 이진트리의 모양으로 구성되기에 선형적이며, 일련된 데이터의 성질을 보장하기 때문이다.

이로써 삽입 삭제에 유동적이며 선형성이 보장되는 데이터 관리가 가능하다.

### Insertion
```cpp
// O(Log n) height of tree log n steps
    void push(int data){
        v.push_back(data);
        int idx = v.size()-1;
        int parent = idx/2;

        //swap Logic
        while (idx >1 and v[idx] < v[parent]){
            swap(v[idx], v[parent]);
            idx = parent;
            parent = parent/2;
        }
    }
```
가령 루트 노드를 i라고 한다면 그 자식들은 2i , 2i+1의 인덱스를 가지도록 배열할 수 있게 된다.

노드를 추가할 시 배열의 마지막에 접근하여 O(1) 상수 시간 처리가 가능하며 부모 노드와의 swap도 Idx/2 만큼 거슬로 올라가 찾을 수 있다.


###  Pop
```cpp
void Pop(){
        int idx = v.size()-1;
        swap(v[1],v[idx]);
        v.pop_back();
        //case : swap with parent
        Heapify(1);
    }

void Heapify(int i){
        int left = i*2;
        int right = i*2 + 1;

        int minIdx = i;
        //좌우 노드 값 비교
        if(left < v.size() and v[left] < v[i]){
            minIdx = left;
        }
        if(right < v.size() and v[right] < v[minIdx]){
            minIdx = right;
        }
        if(minIdx != i){
            swap(v[i], v[minIdx]);
            Heapify(minIdx);
        }
    }
```

Heap에서 루트 노드를 제거하면 Heap의 구조가 파괴될 우려가 있다.
크게 2가지 단계로 처리해야 하는데

1. 루트와 리프 노드의 값을 교체한다.
	1-1. 리프 노드의 값을 삭제한다.
2. 힙 조건 만족을 위해 위에서부터 비교 연산을 수행해 swap 한다.
	> 더 이상 변경점이 없다면 최소 힙 조건을 만족하게 된다.


###  Switch Min Max Heap
최대 힙, 최소 힙을 바꾸는 과정이다.
기존 메소드에서 bool 인자를 넣어 교체해주도록 하자.

```cpp
bool IsMin(int idx, int root){
        if(isMin){
            return idx<root;
        }
        else{
            return idx>root;
        }
    }

// Heapify()
if(left < v.size() and IsMin(v[left], v[i])){
    idx = left;
}
if(right < v.size() and IsMin(v[right], v[idx])){
    idx = right;
}

// Constructor
Heap(int default_size=10, bool isMin = true){
        this->isMin = isMin;
        v.reserve(default_size+1);
        v.push_back(-1);
}
// main
Heap  h = Heap(15,false);

result
10
-1 -> 10 ->
20
-1 -> 20 -> 10 ->
50
-1 -> 50 -> 10 -> 20 ->
40
-1 -> 50 -> 40 -> 20 -> 10 ->
30
-1 -> 50 -> 40 -> 20 -> 10 -> 30 ->
0
-1 -> 50 -> 40 -> 20 -> 10 -> 30 -> 0 ->
0
-1 -> 40 -> 30 -> 20 -> 10 -> 0 ->
0
-1 -> 30 -> 10 -> 20 -> 0 ->
0
-1 -> 20 -> 10 -> 0 ->
0
-1 -> 10 -> 0 ->
0
-1 -> 0 ->
```

### STL Priority queue

STL의 우선순위 큐를 알아보자

```cpp
int main()
{
    int arr[] = {10,15,20,13,6,90};
    int n = sizeof(arr)/sizeof(int);

    priority_queue<int> heap;
    //reversed Min heap
    // priority_queue<int, vector<int> , greater<int>> heap;
    for (int x : arr){
        heap.push(x);
    }

    while (!heap.empty()){
        cout<< heap.top() << endl;
        heap.pop();
    }
 }
```

STL의 기본 정렬은 Max Heap이다.


```cpp
priority_queue<int, vector<int> , greater<int>> heap;
```
우선순위 큐에는 다른 파라미터를 넣어 조건을 변경시킬 수 있는데, 해당 파라미터에 따라 Min, Max heap 을 만들 수 있다.

#### Custom Compare
이전에도 다뤄봤던것 같은데, 커스텀 비교기를 만들 수 있다.

```cpp
class Compare{
public:
    bool operator()(int a,int b){
        return a < b;
    }
};

priority_queue<int, vector<int> , Compare> heap;
```


비교 기준에 커스텀 비교기를 입력해주면 된다.

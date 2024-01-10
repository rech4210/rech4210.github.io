---
layout: post
title:  "[자료구조/ 알고리즘] 헤더파일과 템플릿, searching sorting 알고리즘"
date: 2023-10-01 15:32:01 +0900
categories: Algorithm
tags : Algorithm, Data Structure

---

## Header File & Template Classes

기본적으로 c , c++에서는 헤더파일을 만들어 소스코드를 사용할 수 있다.  
사용하고자 하는 파일을 name.h 형식으로 만든 뒤, 사용할 곳에서 #include "name.h"를 선언해주면 된다.  

```
#include "name.h"
```

```
template<typename  T>
```
c#에서 사용하던 그 generic이다.  
template을 사용함에 따라 사용할 매개변수 또한 제너릭 타입으로 변경시켜주어야 한다.
```cpp
template<typename T>
class vector
{
    int cs;
    int ms;
    T * arr;
public:
    vector(int max_size = 1)
    {
        cs = 0;
        ms =max_size;
        arr = new T[ms];
    }

    void push_back(const T value)
    {
        if(cs == ms)
        {
            T * oldArr = arr;
            ms = 2*ms;
            arr = new T[ms];

            for (T i = 0; i < cs; i++)
            {
                arr[i]= oldArr[i];
            }
            delete[] oldArr;
        }
        arr[cs] = value;
        cs++;
    }

    T front() const
    {
        return arr[0];
    }


    T operator [](const int i) const
    {
        return arr[i];
    }
};
```


## Inbulit Searching


```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;
int main()
{
    vector<int> arr {10,11,2,3,4,6,7,8};

    int key = 11;
    cin >> key ;

    //arr.end() is not points end of array. points next to end of sequence
    // linear search use in unsorted arrays
    vector<int>::iterator it = find(arr.begin(), arr.end(),key);

    if(it != arr.end())
        cout << "present index" << it - arr.begin();
    else
        cout << "element not founded";
}
```
c++에는 배열 탐색을 위한 함수들이 여러 존재한다.
그중 std:: find 는 algorithm 헤더파일에서 가져오며 시작점 begin 끝점 end를 설정하여 범위내 key를 찾도록 한다.

비슷한 함수로 std::search가 있다.
차이점으로는 find는 단일 element 탐색, search는 subSequence를 탐색한다.

## Sorting Complex Vector

복합적인 데이터를 정렬할 때가 있을 텐데, 그 때 어떻게 데이터를 처리해야할까?

아래와같이 string, vector <int>의 pair를 담는 vector container가 있다.
이 데이터를 총합이 높은 순으로 이름을 정렬하려면 아래와 같이 하면 된다.

```cpp
int calc(vector<int> marks)
{
    int sum = 0;
    for (auto &&i : marks)
        sum += i;
    return sum;
}

bool compare(pair<string,vector<int>> m1, pair<string,vector<int>> m2)
{
    return calc(m1.second) > calc(m2.second);
}
int main()
{
    vector<pair<string,vector<int>>> student_marks{
        {"Rohan",{10,20,11}},
        {"Prateek",{10,21,3}},
        {"Vivek",{4,5,6}},
        {"Rijul",{10,13,20}}
    };

    sort(student_marks.begin(), student_marks.end(), compare);

    for (auto s : student_marks)
        cout << s.first << " " << calc(s.second) <<endl;
}
Result:
Rijul 43
Rohan 41
Prateek 34
Vivek 15
```

위 코드에서 보이듯이
1.  sort 함수에 정렬 대상의 범위를 설정해준다.
2. sort 함수의 compare 인자에 비교기를 넣어준다.
3. 비교기의 기준에 따라 데이터를 정렬한다.

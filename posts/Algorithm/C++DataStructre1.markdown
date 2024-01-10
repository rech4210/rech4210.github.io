---
layout: post
title:  "[자료구조/ 알고리즘] Dynamic Array & Vector Container"
date: 2023-09-27 13:24:55 +0900
categories: Algorithm
tags : Algorithm, Data Structure

---
## 2D Dynamic Array

```cpp
int ** create2DArray(int row, int col)
{
    // int pointer that contains int pointer which point to int array
    int ** array = new int *[row];

    //allocate memory for loop
    for (int i = 0; i < row; i++)
    {
        array[i] = new int[col];
    }

    int value = 0;
    //set value
    for (int i = 0; i < row; i++)
    {
        for (int j = 0; j < col; j++)
        {
            array[i][j] = value++;
            // print array value
            cout<< array[i][j] << endl;
        }
    }
    return array;
}

int main()
{
    int row, col;
    cin >> row >> col;
    int ** test2DArray = create2DArray(row,col);

    cout << test2DArray<< endl; // 0x767fa0
}
result : 1,2
1
2
0
1
```
2차원 동적 배열을 만드는 코드이다.
프로세스를 살펴보자면
1. main 함수에서 create2DArray 메소드를 호출해 CallStack에 쌓는다.
2. create2DArray 함수에서 (Static) int**포인터와 (Heap) 2차원 동적 배열을 만들어준다.
3. 함수 값 반환 시 함수의 int 포인터와의 연결이 해제되고 main의 int** 변수가 2dArray에 연결된다.

int ** 타입??
> int 타입의 포인터로써 int * 즉 int * 타입 배열을 가리키는 포인터이다.


## Vector

### Vector vs Array
1. Vector는 Container를 통해 사이즈 변경이 동적으로 가능하다.
2. 메모리의 연속적인 요소 저장이 되며 접근속도가 O(1)이다.

### Vector doubling
벡터의 size가 초과할 때 확장이 일어나 공간이 증가한다.
증가하는 값은 정확히 기존 배열의 2배인데, 이 과정에서 2배만큼의 새로운 배열을 생성하고 기존 배열의 값을 복사하고 삭제하는 과정을 거친다.

vector push_back 함수를 보면 Complexity 가 Contant (amortized)라고 나오는데,
배열 공간에 요소 n만큼의 push_back 연산을 수행할 때 수행횟수가 n/n으로 나누어 상수로 떨어지기 때문이다.

아래 정리가 잘 된 글이 있어 참고함.
[amortized vector push_back](https://codingdog.tistory.com/entry/amortized-%EB%B3%B5%EC%9E%A1%EB%8F%84-average-complexity%EC%99%80-%EB%AD%90%EA%B0%80-%EB%8B%A4%EB%A5%B8%EA%B0%80%EC%9A%94)

## Using Vector Container

```cpp
int main()
{
    // vector Initialize
    vector<int> arr = {1,2,3,4,5};

    // vector Constructor
    vector<int> arr2(5,0);
    arr.pop_back();
    arr.push_back(15);

    for (int i = 0; i < arr.size(); i++)
    {
        cout << arr[i] << endl;
    }
}
```

초기 벡터 배열에 push back을 수행하면 size , capacity 는 5,5 -> 6, 10 으로 변한다.
capacity는 배열 크기를 초과하면 2배로 크기를 늘리기 때문.

### 2D Vector using STL

```cpp
int main()		
{
    vector< vector<int> > arr {1,2,3},{4,5,6},{7,8,9,10},{11,12};
    for (int i = 0; i < arr.size(); i++)
    {
        for (auto x : arr[i])
        {
            cout<< x <<",";
        }
        cout << endl;
    }
}
```

## Vector Class

vector container의 구성요소를 알아보자.
기본적으로 vector는 동적으로 크기가 증가하기에 push 에서 늘려주는 작업을 거친다.
```cpp
class vector
{
    int cs; // current
    int ms; // max
    int * arr;
    //vector constructor
    // 초기 배열 크기를 정해준다.
public:
    vector(int max_size = 1)
    {
        cs = 0;
        ms =max_size;
        arr = new int[ms];
    }
    void push_back(const int value)
    {
        if(cs == ms)
        {
            int * oldArr = arr;
            ms = 2*ms;
            arr = new int[ms];

            for (int i = 0; i < cs; i++)
            {
                arr[i]= oldArr[i];
            }
            delete[] oldArr;
        }
        arr[cs] = value;
        cs++;
    }
    void pop_back()
    {
        if(cs>=0)
        cs--;
    }

    bool isEmpty() const
        return cs==0;

    int front() const
        return arr[0];

    int back() const
        return arr[cs-1];

    int at(int i) const
        return arr[i];

    int size() const
        return cs;

    int capacity() const
        return ms;

    // if value not change, can use const
    int operator [](const int i) const
    {
        return arr[i];
    }
};

int main()
{
    vector v(5);
    v.push_back(1);
    v.push_back(2);
    v.push_back(3);
    v.push_back(4);
    v.push_back(5);

    v. pop_back();
    std::cout << v.size() << v.capacity()  << v[3]<< std::endl;
}
```

위를 보면 const가 작성된 것을 볼 수 있는데, 수정되지 않을 데이터를 다룰 때 컴파일러 측에 해당 상수를 알려 오류를 최소화 할 수 있다.

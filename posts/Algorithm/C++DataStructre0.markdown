---
layout: post
title:  "[자료구조/ 알고리즘] C++ 포인터와 레퍼런스"
date: 2023-09-24 11:54:15 +0900
categories: Algorithm
tags : Algorithm, Data Structure

---
# Pointer

```cpp
int(*p)[3];
// int[3] 짜리 배열을 가리키는 포인터 타입 변수

int  a[3] ={1,2,3};
p = &a;
///a 배열의 주소값을 포인터 p 변수에 저장한다.
for (int  i = 0; i < 4; i++)
{
	cout  << (*(p) + i) <<  " "; //주소값 출력
	cout  << *(*(p) + i) <<  " "; // dereference 역참조
}
result:
1 2 3 6487720
0x62fe98 0x62fea0 0x62fea8 0x62feb0
```

마찬가지로 포인터를 담는 포인터 타입 변수도 만들 수 있다.
```cpp
int x = 5;
int * intptr = &x;
int ** intptr_ptr = &intptr;


Result :
cout  <<  intptr   <<  endl; 0x62fea0
cout  << *intptr   <<  endl; 5

cout  <<  intptr_ptr   <<  endl; 0x62fe9c
// refrence of ** pointer

cout  << *intptr_ptr   <<  endl; 0x62fea0
cout  << **intptr_ptr   <<  endl; 5
```
이렇게 더블 포인터를 사용한 후 역참조를 실행하면 포인터의 주소 및 레퍼런스된 값을 가져올 수 있다.
더블 포인터 주소 **0x62fe9c** , 포인터 주소 **0x62fea0**간의 값을 빼면 4만큼의 공간이 남는데, 이것은 포인터의 저장 공간이 된다.

 null Ptr
 > nul을 가리키는 포인터


## Reference Variables

1. pointer *
2. reference &
이 두가지는 개별의 개념이다

reference(&)는 call by value 타입을 call by reference 로 바꾸어 사용할 수 있다.

아래의 예제를 보자.
```cpp
//call by reference type
void  Add(int  &value)
{
	value = value+1;
}

int  main()
{
	int  x = 10;
	int& y = x;

	y++;
	x++;

	cout  <<"value x:"  <<  x  <<  endl;
	cout  <<"value y:"  <<  y  <<  endl;

	Add(x);
	cout  <<"value x:"  <<  x  <<  endl;
	cout  <<"value y:"  <<  y  <<  endl;
}
Result
value x:12
value y:12
value x:13
value y:13
```
즉, Reference Value는 같은 메모리를 공유하는 변수이다.


## Reference by Pointer

다음은 포인터를 사용한 refrence 전달 방식이다.
```cpp
void Add(int * valuePtr)
{
	// dereference operator *
    *valuePtr = *valuePtr+1;
}

int main()
{
    int value = 5;
    // address of operator
    Add(&value);
    cout<< "value: "<< value << endl;
}
```

## Dynamic Memory Allocation
함수 및 지역변수 (포인터 변수를 포함하여) 는 Stack Memory에 저장되나
reference type의 포인터 실제 데이터 및 refernce 타입은  Heamp Memory (동적 메모리)에 저장된다.

또한 StackMemory의 call stack은 컴파일러에 의해 삭제되나 heap은 동적 메모리 할당 공간이기에 직접 관리를 해주어야 한다.

arr[2] = * (arr + 2)와 동일하다. 즉 배열의 메모리에 직접 접근하는것을 표현한다.

### Memory Leek
Heap에서 더이상 사용하지 않는 메모리를 점유하고 있는 누수 현상이다.
Stack 호출에서 동적으로 메모리를 할당하고 이에 대한 포인터를 만들 때, 포인터가 손실되는 경우가 예시이다.

c++의 경우 직접 메모리 해제를 해주어야 하나, java 같은 경우에도 만약 해당되는 메모리의 참조가 삭제되지 않는다면 (java는 참조하지 않은 개체를 가비지가 제거한다) 삭제를 수행하지 않는다.
  [참고링크](https://velog.io/@wonhee010/%EB%A9%94%EB%AA%A8%EB%A6%AC-%EA%B5%AC%EC%A1%B0-feat.-%EC%9E%AC%EA%B7%80-vs-%EB%B0%98%EB%B3%B5%EB%AC%B8)

## New & Delete 동적 생성 / 삭제

```cpp
{
    int n;
    cin >> n;
    //static 공간에 포인터와 heap 공간에 실제 동적 메모리를 할당
    int * arr = new int[n];

    for (int i = 0; i < n; i++)
    {
        arr[i] = i;
        cout <<  arr[i] << endl;
    }
    cout << arr << endl;
    // 이 시점에서 동적 메모리가 삭제됨
    delete [] arr;
} // main 함수 범위를 벗어나면 컴파일러에 의해 *arr 포인터 또한 삭제됨.
```
heap 공간에 메모리를 할당할 땐, new 키워드를 사용하여 동적으로 공간을 할당해준다.
반대로 delete를 사용하여 메모리 해제 (포인터 해제)를 시켜준다.

:warning: delete를 하고나서 array indexing (ex) arr[2]... 를 하게되는 경우 메모리 참조가 되는 경우가 있는데, 이는 컴파일러가 메모리 해제와 동시에 제거하지 않아 발생할 수 있다 (최적화의 문제임)

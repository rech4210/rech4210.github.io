---
layout: post
title:  "[자료구조/ 알고리즘] C++"
date: 2023-09-20 12:17:04 +0900
categories: Algorithm
tags : Algorithm, Data Structure

---

# Array 선언 및 정의
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/c889d99d-e531-4144-80a5-c8b15d7ec81c)
순서대로
<br>
<br>

1. 선언

초기화 하지 않았으므로 100개의 공간에는 가비지 값이 들어간다.

```cpp
int main()
{
    int marks[100];
    int n;
    cout << "Enter the num of students";
    cin >> n;

    for (int i = 0;i<n; i++)
    {
        cin >> marks[i];
        marks[i] *= marks[i];
    }
    for (int i = 0; i < 30 ; i++)
    {
        cout<< marks[i] <<" ,";
    }
    cout << endl;
    return 0;
}
```

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/8215cc69-9546-4a52-a593-b290da4e16d2)
<br>

2. 선언 후 초기화  

배열에 초기 값을 설정해주면 전부 같은 값으로 초기화

```cpp
int marks[100] = {0};
```
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/be7ae0c1-9780-4d41-86f4-4a10a6962a13)

<br>
<br>

3. 선언 후 특정 인덱스 초기화

```cpp
int marks[100] = {1,2,3,4};
```  
<br>

4. 인덱스 개수 만큼만 정의 및 초기화

```cpp
int makr[] = {1,2,3};
```

# Array Pass by reference (함수에 배열 사용)

### value와 reference 차이
c++에서 배열의 길이를 구하려면 sizeof를 사용한다.
> sizeof(array)/ sizeof(T) T는 타입으로 int, string, char 등등..

타입을 함수의 인자로 넘길 때 value, reference의 차이를 반드시 알아야한다.
배열은 reference 타입으로 함수의 인자로 넘길 때 해당 메모리 주소 포인터를 넘긴다.
<br>
```cpp
void PrintArray(int  arr[])
{
    //cout << "size of array: " << n << endl;
    for(int i=0 ; i< sizeof(arr)/ sizeof(int);i++)
    {
        arr[i] = i;
        cout<<  arr[i]<< " ";
    }
}

int main()
{
    int a[] = {1,2,3,4,5,6,7,8};
    //int n = sizeof(a)/sizeof(int);
    PrintArray(a);
}
---
result = 0 1;
```
arr 메모리 포인터의 sizeof(arr)가 8byte, sizeof(int)가 4byte 이므로 8/4 =2
이처럼 결과값을 2회만 출력한다.

<br>
<br>

### 원본 객체에 접근하여 배열의 길이 구하기
```cpp
void PrintArray(int arr[],int n)
{
    cout << "size of array: " << n << endl;
    for(int i=0 ; i< n;i++)
    {
        arr[i] = i;

        cout<<  arr[i]<< endl;
    }
}

int main()
{
    int a[] = {1,2,3,4,5,6,7,8};
    int n = sizeof(a)/sizeof(int);
    PrintArray(a,n);
}
```
결과적으로 PrintArray 함수에서 매개변수 int arr[]는 int a[]의 포인터 값을 가져오는 것이다.
> **func(Type  * array)** 는 **func(Type array[])** 와 같다고 한다. (배열의 주소 값을 넘겨주기 때문)

<br>
<br>
그렇기 때문에 인자를 전달받아 주소값을 참조하여 원본 값 변경 또한 가능해진다.

### 포인터를 통해 원본 값 변경

```cpp
void PrintArray(int * arr,int n)
{
    for(int i=0 ; i< n;i++)
    {
        arr[i] *= i;
        cout<<  arr[i]<< ",";
    }
    cout<< "함수내부: 주소에 접근하여 원본 값 변화"<<endl;
}

int main()
{
    int a[] = {1,2,3,4,5,6,7,8};
    int n = sizeof(a)/sizeof(int);
        for(int i =0 ; i < n; i++)
    {
        cout << a[i] << ",";
    }
    cout<< "원본 배열 값"<<endl;

    PrintArray(a,n);
    cout << endl;

    for(int i =0 ; i < n; i++)
    {
        cout << a[i] << ",";
    }
    cout<< "변경 후 배열 값"<<endl;
}

1,2,3,4,5,6,7,8,원본 배열 값
0,2,6,12,20,30,42,56,함수내부: 주소에 접근하여 원본 값 변화
0,2,6,12,20,30,42,56,변경 후 배열 값
```
### ❗ 하지만 Pointer(*)와 Reference(&)는 서로 다른것이라고 한다.
> 이 부분은 추후에 자세히 다뤄보도록 하자.

# Searching in Array (배열 탐색)

## Linear Search
배열의 선형 탐색으로 brutal force (모든 경우의 수 탐색) 이다.
```cpp
int Linear_Search(int * arr, int key,int length)
{
    for(int i = 0; i< length; i++)
    {
        if(arr[i] == key)
            return i;
    }
    return -1;
}

int main()
{
    int arr[] = {75,344,2,13,45};
    int key = 2;
    int length = sizeof(arr)/sizeof(int);
    cout<< Linear_Search(arr,key,length) <<endl;

    return 0;
}
```
워낙 기본중의 기본이니 자세한 설명은 생략
전체를 순회하며 탐색하기에 O(n)의 시간 복잡도를 가진다.

## Binary Search
시작 값과 끝 값을 기준으로 중간을 분할하여 탐색하는 방식이다.
Binary Search는 탐색 효율과 속도가 빠른데, log2^n^ 의 탐색속도를 가진다.
최악의 경우에서 n/2^k^  :arrow_right:  n = 2^k^  :arrow_right: k = log2^n^로  k번 나누기를 반복하기 때문.
다만 탐색 대상이 정렬되어있어야 사용 가능하며 단조적이어야 한다.

> Monotonic (단조적인) 배열이 {1,2,3,4} / {3,5,7,9}등 등차를 이루어 증가하는 경우.

로직을 나누어보자면
1. 배열의 시작과 끝 그리고 중간 값을 설정한다.
2. array[middle] 값과 key 값을 비교한다.
	- middle > key  :arrow_right: key
	> 값이 인덱스의 중간보다 **앞**에 위치 (start = middle +1)

	- middle < key  :arrow_right: key
	> 값이 인덱스의 중간보다 **뒤**에 위치 (end = middle -1)

	- middle이 key값과 대응
	> middle == key

3. start와 end의 인덱스가 0이 될때까지 반복
	> 만약, start값이 end값을 초과하면 not founded -1 반환

```cpp
int Binary_Search(int  arr[], int key,int length)
{
    int start,end,middle;
    start = 0;
    end = length-1;
    middle = 0;

    while(start <= end)
    {
        middle = (start+end)/2;
        if(arr[middle] > key)
        {
            return middle;
        }
        else if(arr[middle] < key)
        {
            start = middle +1;
        }
        else
        {
            end = middle -1;
        }
    }
    // 탐색 실패시 -1 반환
    return -1;
}

int main()
{
    int a[] = {10,20, 50, 70, 90};
    int key = 50;
    int length = sizeof(a)/sizeof(int);

    int result = Binary_Search(a,key,length);
    if(result == -1)
    {
        cout << key << " index not founded"<< endl;
    }
    else
    {
        cout<< result  << endl;
    }
    return 0;
}

Result : 2
```

## Reverse Array

배열의 인덱스를 서로 교체해 배열 요소를 전체적으로 반전시키는 방법이다.
```cpp
void reverseArray(int arr[], int n)
{
    int s = 0;
    int e = n-1;
    while(s != e)
    {
        //int temp = 0;

        //iostream에 정의된 swap() 함수
        swap(arr[s],arr[e]);
        //temp = arr[s];
        //arr[s] = arr[e];
        //arr[e] = temp;
        s++;
        e--;
    }

}

int main() {

    int arr[] = {10,20,30,40,50};
    int n = sizeof(arr)/sizeof(int);

    reverseArray(arr,n);

    for(int i = 0; i< n; i++)
    {
        cout << arr[i]<< endl;
    }


    return 0;
}
result : 50 40 30 20 10
```

<br>
c#의 경우

```csharp
int temp = 0;
temp = arr[s];
arr[s] = arr[e];
arr[e] = temp;
```
이와 같이 임시 변수를 만들어 값을 저장하고 swap 했다.
하지만 c++에는 swap() 함수가 있어 바로 교체가 가능하다.

## 예제 Sub Array 출력 후 가장 큰 값 찾기

```cpp
// brutal force
void PrintSubArray(int arr[], int n)
{
    int sum = 0;
    for(int i= 0; i < n ; i++){
        for(int j = i; j < n; j++){
            int subArraySum = 0;

            for(int k = i; k<= j ; k++){
                cout << arr[k]<<",";
                subArraySum += arr[k];
            }
            sum = max(sum,subArraySum);
            cout <<"합 :"<< subArraySum <<" ,인덱스 :"<< j-i << "\n";
            cout <<endl;}
        cout <<endl;}
    cout << "최대값은 : "<<sum<<endl;
}

int main() {
    int arr[] = {10,20,-5,30,-27,50};
    int n = sizeof(arr)/sizeof(int);
    PrintSubArray(arr,n);
    return 0;
}
10,합 :10 ,인덱스 :0
10,20,합 :30 ,인덱스 :1
10,20,-5,합 :25 ,인덱스 :2
10,20,-5,30,합 :55 ,인덱스 :3
10,20,-5,30,-27,합 :28 ,인덱스 :4
10,20,-5,30,-27,50,합 :78 ,인덱스 :5
.
.

-27,합 :-27 ,인덱스 :0
-27,50,합 :23 ,인덱스 :1

50,합 :50 ,인덱스 :0

최대값은 : 78
```
위의 max 함수로 최대값을 비교한다.
시간 복잡도는 3단계의 nested loop를 돌기에 O(n^3^) 이다

## 최적화하기

```
temp[i] = temp[i-1] + alphaValue;
```
하나의 배열을 만들어 인덱스를 탐색할 때, 인덱싱하며 더해가는 방식이다.
결과적으로 마지막 인덱스는 모든 인덱스들의 합이 된다.
```cpp
#include <iostream>
using namespace std;
int PrintSubArray(int arr[], int n)
{
    int sum = 0;

    for(int i= 0; i < n ; i++){
        int tempArr[n] = {0};
        tempArr[i] = arr[i];
        cout <<endl<<"초기 값 :"<< tempArr[i] << endl << endl;
        for(int j = i+1; j < n; j++)
        {
            tempArr[j] = tempArr[j-1] + arr[j];
            sum = max(sum, tempArr[j]);
            cout <<"더해질 값: " << arr[j] << " 연산 후: " << tempArr[j]<< " sum : "<<sum << endl;
        }
    }
    return sum;
}

int main() {

    int arr[] = {1,2,3,4,5,6};
    int n = sizeof(arr)/sizeof(int);
    cout <<PrintSubArray(arr,n)<< ": 최대 값";
    return 0;
}

<details>
<summary>결과</summary>
초기 값 :1

더해질 값: 2 연산 후: 3 sum : 3
더해질 값: 3 연산 후: 6 sum : 6
더해질 값: 4 연산 후: 10 sum : 10
더해질 값: 5 연산 후: 15 sum : 15
더해질 값: 6 연산 후: 21 sum : 21

초기 값 :2

더해질 값: 3 연산 후: 5 sum : 21
더해질 값: 4 연산 후: 9 sum : 21
더해질 값: 5 연산 후: 14 sum : 21
더해질 값: 6 연산 후: 20 sum : 21

초기 값 :3

더해질 값: 4 연산 후: 7 sum : 21
더해질 값: 5 연산 후: 12 sum : 21
더해질 값: 6 연산 후: 18 sum : 21

초기 값 :4

더해질 값: 5 연산 후: 9 sum : 21
더해질 값: 6 연산 후: 15 sum : 21

초기 값 :5

더해질 값: 6 연산 후: 11 sum : 21

초기 값 :6

21: 최대 값
</details>

```
시간 복잡도는 2단계의 nested loop를 돌기에 O(n^2^) 이다.
다만 위의 방법은 새로운 배열을 생성하기에 메모리의 낭비가 존재할 수 있다.

더욱 최적화를 한다면 정적으로 배열을 선언하고 초기화한다.
```cpp
fill_n(tempArr,n,0);
```

## Kadane's algorithm
SubArray의 최대값을 구할 때 사용하는 최적의 알고리즘이다.

1. 더하는 인자가 음수이고 더한 결과값이 0보다 작을 때 현재 합계를 0으로 바꾼다
2. 아니라면 현재 합계에 인덱스 값을 더해간다.
3. max() 함수를 이용해 결과값을 비교한다.

```cpp
int KadaneSubArray(int arr[], int n)
{
    int result = 0;
    int csum = 0;

    for(int i = 0; i < n ; i++)
    {
        csum = arr[i] + csum;
        if(csum < 0)
            csum = 0;

        result = max(result , csum);
    }
    return result;
}

int main()
{

    int arr[] = {-5,2,7,-6,-32,15};
    int n = sizeof(arr)/sizeof(int);
    cout <<KadaneSubArray(arr,n)<< ": 최대 값";
    return 0;
}
15: 최대 값
```
하나의 반복문을 배열 개수  n 만큼 순회하며 추가적인 배열 메모리 공간을 필요로 하지 않아 시간복잡도는 O(n) 이다.

## Vector Struct
t
c++ 배열과  Vector의 차이점

1. 배열의 성질을 동일하게 갖고있다.
	- 조회가 O(1).
	- 연속적이다.

차이점
2. 동적 배열이다. (사이즈 변경 가능)
3. 함수의 인자로 전달할때 값으로 전달한다.

```cpp
int main()
{
    vector<int> arr = {1,5,235,32,5};
    cout << arr.size() << endl;
     cout << arr.capacity() << endl;
     arr.push_back(16);

    cout << arr.size() << endl;
     cout << arr.capacity() << endl;
    return 0;
}
size = 5
capacity = 5
------
size = 6
capacity = 10
```
이처럼 인덱스 요소가 1개 증가할 때와 달리 벡터의 메모리 사이즈를 늘려야 할 땐 기존 메모리 공간의 두 배를 할당한다.

--------------
----------
----

# Sorting

## Bubble Sort

n^2^의 복잡도를 가지는 정렬로 인덱스를 한칸씩 n번 (n-1, n-2...) 비교하며 남은 인덱스동안 반복한다. 만약 정렬 대상이 존재하면 대상과 자신의 위치를 바꾸는 방식이다.

```cpp
void bubble_sort(int* arr, int n)
{
    for(int times = 1; times <= n; times++)
    {
        for(int j =0 ; j <= n-times; j++)
        {
		    // 해당 인덱스가 더 크면 스왑.
            if(arr[j] > arr[j+1])
                swap(arr[j+1],arr[j]);
        }
    }
}

int main()
{
    int arr[5] = {7,5,3,18,2};
    int n = sizeof(arr)/ sizeof(int);
    bubble_sort(arr, n);
    for(auto x : arr)
    {
        cout<< x << ",";
    }
    return 0;
}
```

## Bubble sort Optimization
버블 정렬 최적화

```cpp


```

## Insert Sort

n^2^의 복잡도를 가지며 인덱스 기준의 이전 배열을 순회하고, 정렬이 끝나게 되면 currentValue의 값을 마지막에 삽입하는 정렬이다.

```cpp
void insertion_sort(int* arr, int n)
{
    for(int i = 1; i <= n-1; i++)
    {
        int currentValue = arr[i];
        int prev = i -1;
        // 현재 값과 인덱스를 비교해가며 인덱스 감소 반복문 실행
        while(arr[prev]> currentValue && prev >= 0)
        {
            arr[prev+1] = arr[prev];
            prev = prev -1;
        }
        // 마지막 인덱스 초과범위에서 +1
        arr[prev+1] = currentValue;
    }
}

int main()
{
    int arr[5] = {7,5,3,18,2};
    int n = sizeof(arr)/ sizeof(int);

    insertion_sort(arr, n);
    for(auto x : arr)
    {
        cout<< x << ",";
    }
    return 0;
}
```

## Selection Sort
정렬 대상을 기준으로 순회하며 나온 결과 인덱스를 초기 인덱스와 swap 하여 정렬하는 방식이다. 복잡도는 n^2^이다.

```cpp
void selection_sort(int *a, int n)
{
    for(int pos = 0; pos < n-1; pos++)
    {
        int currentValue = a[pos];
        int minpos = pos;

        for(int j = pos; j < n; j++)
        {
            if(a[minpos] > a[j])
            {
                minpos = j;
            }
        }
        swap(a[pos],a[minpos]);
    }
}
int main()
{

    int a[] = {-2,3,4,-1,5,-12,6,1,3};
    int n = sizeof(a)/sizeof(int);
    selection_sort(a,n);

    for(auto x: a)
    {
        cout<< x << endl;
    }

    return 0;
}
```

## Inbulit sort & comparators
내장 알고리즘과 비교기
sort()는 #include <algorithm>라이브러리에 포함된 알고리즘 함수이고 복잡도는 Nlogn이다.

비교기는 bool타입 또는 결과값을 반환하는 수식 함수를 넣어주어 이를 함수처럼 쓸 수 있는 것이다.
~~~
sort(a,a+n, compare);
~~~
compare(a, b)와 같이 인자를 넣어주는 부분은 생략되었지만, 자동으로 함수의 파라미터를 입력하여 비교식을 수행한다. 비교기를 사용할 땐 괄호를 쓰지 않으므로 주의하자.

```cpp
#include <iostream>
#include <algorithm>
using namespace std;
bool compare(int a, int b)
{
    return a>b;
}

int main()
{

    int a[] = {-2,3,4,-1,5,-12,6,1,3};
    int n = sizeof(a)/sizeof(int);
    //         비교기
    sort(a,a+n,compare);
    //reverse(a,a+n);
    for(auto x: a)
    {
        cout<< x << endl;
    }

    return 0;
}
결과 : -12, -2, -1, 1, 3, 3, 4, 5, 6 return a>b;
결과 : 6, 5, 4, 3, 3, 1, -1, -2, -12 return a<b;
```
-  compare의 수식을 변경하면 정렬 기준을 변경할 수 있다.
그 외 **greater< int>()** 를 통해 정렬 기준을 정의할 수도 있다.

## Counting Sort
주어진 배열의 최대값을 기준으로 배열 크기를 만들고 순회하며 배열 값에 해당하는 인덱스에 count를 늘리는 방식이다.
반복도는 배열의 길이 + 순회 n으로 lange + N이다.

1. 최대값을 찾는다.
2. 최대값만큼 vector 배열 크기를 할당한다.
3. 배열을 순회하며 vecotr index에 a[i] 값 만큼 ++ 한다.
4. vector 배열을 순회하며 조건에 따라 a[j] = freq index, freq-- 를 수행한다.
```cpp
void counting_sort(int a[],int n)
{
    int largest = -1;
    for(int i = 0; i < n; i++)
    {
        largest = max(largest,a[i]);
    }
    vector<int> freq(largest+1,0);

    for(int i= 0; i < n; i++)
    {
        freq[a[i]]++;
    }

    int j = 0;
    // freq의 인덱스를 기준으로 순회
    for(int i = 0; i <= largest; i++)
    {
        while(freq[i] >0)
        {
            //배열의 인덱스와 수가 일치
            a[j] = i;
            freq[i]--;
            j++;
        }
    }
}

int main()
{
    int a[] = {88,97,10,12,15,1,5,6,12,5,8};
    int n = sizeof(a)/sizeof(int);
    counting_sort(a,n);

    for(auto x : a)
    {
        cout << x << ", ";
    }

    return 0;
}
```

# Character Array

문자열 배열은 다른 타입과 조금 다른점이 있다.
```cpp
// 가장 기본적인 선언으로 공간을 할당해줌
char a[100];
// 배열 내부의 값을 a, b, c로 초기화하나, 나머지가 초기화되지 않음
char a[100] = {'a','b','c'};
// 배열 내부 값을 초기화하고 나머지를 null로 초기화 해줌
char a[100] = {'a','b','c','\0'};
// abc 문자열을 넣으며 나머지 부분을 일괄 null로 초기화
char a[100] = "abc";
```
:warning: 두번째의 경우 잘못된 방법이다.
배열의 나머지 공간을 항상 null로 초기화해주자.

<br>

```cpp
int main()
{
    char a[] = "abcdefgi"; //{'a','b','c','d','\0'};
    cout << a << endl;

    char b[100];
    cin >> b;

    cout << b << endl;

    cout << strlen(a) << endl; //number of characters
    cout << sizeof(a) << endl; // +1 add null character
    return 0;
}
```
위의 내용에서 주목할것은 strlen, sizeof이다.
strlen은 문자열 배열의 길이이고 sizeof는 문자열 단위로 크기를 나타내는데, 이 둘의 값이 다르게 나온다.
null 문자열을 포함하기 때문.

## Cin.get

문자열 배열에서 std::cin을 사용하면 아래와 같은 문제가 발생한다.
~~~
input > hello my name is ggwa.
output > hello
~~~
cin함수가 output을 만나게 되면 맨처음 문자열부터 입력받아 공백 또는 널을 만나면 입력을 그만 받기 때문이다.

### cin.get
cin.get을 이용하면 공백, 줄바꿈을 포함하여 입력이 가능하다.

```cpp
#include <iostream>
#include <cstring>
using namespace std;

int main() {
    char temp = cin.get();
    int len = 0;
    char sentence[1000];
    while(temp != '#')
    {
        sentence[len++] = temp;
        temp = cin.get();
    }
    cout<< sentence;
    return 0;
}
```

## cin.getline()
긴 문장을 읽을 때 유용하게 사용할 수 있다.

~~~
cin.get();
cin.getline(string, stremaSize);
~~~
cin.get 함수와 달리 제약조건을 매개변수로 설정할 수 있으며, 문자를 추가해주지 않아도 된다.

```cpp
#include <iostream>
using namespace std;
    char sentence[100];

   cin.getline(sentence,100);

    cout << sentence << endl;
}
```
또한  오버로딩으로 제약조건을 설정할 수 있다.
```cpp
cin.getline(sentence,100, '?');
result:
I LIKE coding 코 딩 조 아
Would you wanna coding?
I LIKE coding 코 딩 조 아
Would you wanna coding?
```

## Find Displacement (예제)
문자열을 입력받아 원점으로부터 얼마만큼 떨어저있는지를 구하는 예제이다.

```cpp
#include<iostream>
using namespace std;

int main()
{
    char route[1000];
    cin.getline(route,1000);
    int x = 0;
    int y = 0;

    for (int i = 0; route[i] != '\0'; i++)
    {
        switch (route[i])
        {
        case 'N': y++;
            break;
        case 'S': y--;
            break;
        case 'E': x++;
            break;
        case 'W' :x--;
            break;
        default:
            break;
        }
    }

    cout << "final x and y is " << x << ","<< y << endl;
    if (x >=0 and y>= 0)
    {
        while (y--)
        {
            cout << "N";
        }
        while (x--)
        {
            cout << "E";
        }
    }
    else if(x<= 0 and y <= 0)
    {
        while (y++)
        {
            cout << "S";
        }
        while (x++)
        {
            cout << "W";
        }
    }
    return 0;
}
입력 : SSWSENNEWS
결과 : SS
```

##  String copy, compare, concat
```cpp
#include<iostream>
#include<cstring>
using namespace std;

int main()
{
    char a[1000] = "apple";
    char b[1000];
    //문자열의 길이를 계산합니다.
    cout << strlen(a) << endl;

    //Strcpy a의 값을 b에 복사합니다
    strcpy(b,a);
    cout << b << endl;

    //Compare 모두 같다면 0, 다르다면 양수 음수로 반환합니다.
    cout << strcmp(a,b)<< endl;

    //strcat 서로 다른 문자열들을 합칩니다.
    char web[] = "https://";
    char site[] = "rech4210.github.io/";
	cout  <<  strcat(web, site) <<  endl;
    strcpy(strcat(web,site),b);

    cout << strcmp(a,b) << endl;
    return 0;
}
Result :5, apple, 0 ,https://rech4210.github.io/ , -1
```

## 위의 함수들 도큐먼트에서 찾아보기

## Largest String
```cpp
#include<iostream>
#include<cstring>
using namespace std;

int main()
{
    int n;
    cin >> n;

    //cin.get없이 입력을 수행하면 n-1번만 입력을 받는다
    //이유는 cin>>n 이후의 \n의 영향을 받아 줄바꿈이 입력되기 때문.
    cin.get();
    char sentence[1000];
    char largestSentence[1000];

    int len = 0;
    int largestLen=0;
    while (n--)
    {
        cin.getline(sentence,1000);
        len = strlen(sentence);
        if(len> largestLen)
        {
            largestLen=len;
            strcpy(largestSentence, sentence);
        }
    }
    cout <<"largestLength :" << largestLen <<endl <<"LargestSentence : "<< largestSentence<<endl;

    return 0;
}
result :
3
hello world!
my name is ggwa
nice to meet you!
largestLength :17
LargestSentence : nice to meet you!
```

## String class
문자열을 다루는 방법은 크게 3가지로 나뉜다.
1. char[] 문자 배열타입
2. string 객체 타입
3. vector<string> 타입

```cpp
char s[];
// 문자 배열로 가변이 어려워 낮은 유연성.
string s;
// 단일 문자열 조작에 특화 (문자만 포함 가능, 연속적)
vector<string> s;
// 동적 문자열 컬렉션 관리 특화, 문자 객체 모두 포함 (연속적)
```
string과 vector<string> 둘은 유사하나  가장 큰 차이점은 사용 용도에 있다.

```cpp
#include<iostream>
#include<cstring>
#include<vector>
using namespace std;

int main()
{
    // char s[1000] = {'1','2','a'};
    string s = {"hello"};
    getline(cin,s,'.');

    for (char ch : s)
    {
        cout << ch << ',';
    }
    cout << s << endl;


	//multifle strings
    string temp;
    int n = 5;
    vector<string> sarr;
    while (n--)
    {
        getline(cin,temp);
        sarr.push_back(temp);
    }

    for (string ch : sarr)
    {
        cout<< ch << ",";
    }

    return 0;
}
```

## run length encodng
문자열 최적화 방법.
문자열 배열을 읽어올 때, 같은 문자가 여러개가 연속적으로 존재한다면 아래와같이 최적화 방법을 사용할 수 있을것이다.

```cpp
#include <iostream>
#include <cstring>
using namespace std;
string CompressStrings(string str)
{
    int n = str.length();
    string output;
    for(int i = 0; i < n; i++)
    {
        int count = 1;
        while(i < n-1 and str[i] == str[i+1])
        {
            count++;
            i++;
        }
        output += str[i] + to_string(count);
    }
    //최적화 전 후를 비교한다.
    if(output.length() > str.length())
    {
        return str;
    }
    return output;
}

int main() {

    string str = "aabbccddee";
    cout<< CompressStrings(str) << endl;
    return 0;
}
Result: a2b2c2d2e2
```

## Multi- dimensional Arrays

n차원의 배열은 실생활에도 쓰인다.
그 예시로 이미지를 볼 수 있다.
이미지는 행열로 구성되는 2차원의 데이터이다 (흑백 컬러라면)
다만 컬러 이미지의 경우 3차원 데이터가 되는데
R G B 각각의 데이터가 한 픽셀에 저장되어 컬러를 만들기 때문에 3차원 축은  R,G,B의 데이터 집합이 된다.

마찬가지로 비디오 데이터는 4차원 데이터인데, 3차원 컬러 이미지 데이터에 시간축 t가 곱해져 4차원이 되기 때문이다.

![macos - Visualizing n-dimensional arrays - Software Recommendations Stack  Exchange](https://i.stack.imgur.com/Tbe9W.png)

## 2D Array

2차원 배열 선언과 정의 이다.
```cpp
int arr[][]; // 2d 배열 선언
```
```cpp
#include<iostream>
using namespace std;

void print(int arr[][100], int n , int m)
{
    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < m; j++)
        {
            cout << arr[i][j] << " ";
        }
        cout << endl;
    }
}

int main()
{
    int arr[100][100];
    int n,m;  
    cin >>n >> m;

    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < m; j++)
        {
            cin >> arr[i][j];
        }
    }
    print(arr,n,m);
}

result
arrays : 3 4
1 2 3 4 5 6 7 8 9 10 11 12

1 2 3 4
5 6 7 8
9 10 11 12
```
여기서 중점으로 봐야할것은 배열의 메모리 공간이 연속적이란 것이다.

![Java Pass 2d Array To Method](https://scaler.com/topics/images/pass-column-size-in-arguments.webp)
>아키텍처에 따라 달라질 수 있다.

### char 배열에 문자열을 저장하기

```cpp
int main()
{
    int j = 0;
    char numbers[][10] ={
      "one",
      "two",
      "three",
      "foury",
      "eight",
      "hundred",
    };

    int number_int[4][15] ={
      {1,2,3,4},
      {452,3,4},
      234,
    };

    for (auto &&i : numbers)
    {
        cout<< i[j] << ", ";
        cout<< i << endl; // 문자행 출력
    }
    for (auto &&i : number_int)
    {
        cout<< i[j] << ", ";
        cout<< i << endl; // 메모리 주소 출력
    }
}
result :
o, one
t, two
t, three
f, foury
e, eight
h, hundred

1, 0x61fdc0
452, 0x61fdfc
234, 0x61fe38
0, 0x61fe74
```


## Sorted Array Search

미리 정렬된 배열을 탐색하는 방법 (최적화)
아래와 같은 2d 배열이 주어진다고 하면

```cpp
    int arr[][4] = {10,20,30,40},{15,25,35,45},{27,29,37,48},{32,33,39,50};

열 = N
행 = M
```

가장 기본적인 방법인 brutal force를 사용할 시 N x M의 탐색으로 O(n^2^) 가 나올 것 이다.

알고리즘을 최적화 할땐, 주어진 정보가 어느정도인지, 그 안에서 최대한의 정보를 활용하여야 한다.

행 데이터가 정렬되어 있으므로 Binary Search를 사용 할 수 있다.
 $$ 결과적으로  \log_n 2 의 복잡도를 지니게 된다. $$


한 가지 더, 위의 배열은 이미 열 또한 정렬되어 있으므로 모든 배열간의 상관관계를 명확히 나타낼 수 있다.



이를 이용해 계단식으로 내려가 탐색하는 방법을 사용하면, O(n) 만큼의 복잡도를 가진다.
```cpp
#include<iostream>
#include<cstring>
#include<vector>
using namespace std;

pair<int, int> staircaseSearch(int arr[][4],int n, int m, int key)
{
    if(key < arr[0][0] or key > arr[n-1][m-1])
    {
        return {-1,-1};
    }

    int i = 0;
    int j = m-1;
    while (i <= n-1 and j >=0)
    {
        if (arr[i][j] == key)
        {
            return {i,j};
        }
        else if(arr[i][j]> key)
        {
            j--;
        }
        else
        {
            i++;
        }
    }
    return {-1,-1};
}

int main()
{
    int arr[][4] = {10,20,30,40},{15,25,35,45},{27,29,37,48},{32,33,39,50};

    int n = 4;
    int m = 4;
    int key = 50;

    pair<int, int> pairs = staircaseSearch(arr,n,m,key);
    cout << pairs.first << "," << pairs.second << endl;
}
```

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

## Dinamic Memory Allocation
함수 및 지역변수 (포인터 변수를 포함하여) 는 Stack Memory에 저장되나
reference type의 포인터 실제 데이터 및 refernce 타입은  Heamp Memory (동적 메모리)에 저장된다.

또한 StackMemory의 call stack은 컴파일러에 의해 삭제되나 heap은 동적 메모리 할당 공간이기에 직접 관리를 해주어야 한다.

arr[2] = * (arr + 2)와 동일하다. 즉 배열의 메모리에

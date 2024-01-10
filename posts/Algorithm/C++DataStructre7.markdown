---
layout: post
title:  "[자료구조/ 알고리즘] Time & Space Complexity"
date: 2023-10-26 16:47:04 +0900
categories: Algorithm
tags : Algorithm, Data Structure

---
## Complexcity
시간과 공간의 복잡도를 다룬다
예를 들어 어떠한 알고리즘이 존재할 때, 추가적인 메모리 용량을 사용하고 더 빠른 속도를 낼 수 있거나
반대로 추가 메모리 사용 없이 문제를 해결하는 대신 느린 경우가 존재할 수 있다.

시간복잡도의 경우
Quadratic, Linear, Logarithmic ,Constant로 분류할 수 있다.
각각 n^2^, 선형 (n), Logn, 상수로 생각하면 된다.

### Theoretical Analysis

#### Nested Loop
```cpp
for (size_t i = 0; i < count; i++)
{
    for (size_t j = 0; j < count; j++)
    {
        /* code */
    }
}
```
위와 같은 코드가 있다고 했을 때, count 수에 따라 n, n-1, n-2 ..... 0를 반복한다.
이를 중첩 수행하므로 T = n(n+1)/2이다.
T= n^2^/2 + n/2이고 최고 복잡도를 기준으로 분모를 제거하면...
 O(n^2^)이 된다.

#### Loop Based
```cpp
for (size_t i = 0; i < count; i+j)
{
    for (size_t j = i+1; j < k; j++)
    {
        /* code */
    }
}
```
위의 경우엔 얼핏보기에 O(n^2^) 처럼 보이겠지만 실상 그렇지 않다.
worst case를 기준으로 i 는 i+k횟수만큼 증가하므로 수행횟수는
($\frac{N}{K}$)  x K (j의 worst case) 이므로 O(n)이 된다.

#### Binary Search
이전에서도 다뤘 듯, 이진탐색은 log2^N^ 만큼의 시간 복잡도를 가진다.
상수시간 K의 이진탐색 횟수에 따라 (N+($\frac{N}{2}$) + ($\frac{N}{4}$)+($\frac{N}{8}$)... ($\frac{N}{2^K}$) 의 시행횟수를 가진다.

($\frac{N}{2^K}$) = 1, (N = 2^k^) => (K = Log2^N^)의 값이 된다.

```cpp
int binarySearch(int arr[], int l, int r, int x)
{
    while (l <= r) {
        int m = l + (r - l) / 2;
        if (arr[m] == x)
            return m;

        if (arr[m] < x)
            l = m + 1;

        else
            r = m - 1;
    }
    return -1;
}
```
온라인 코딩테스트에서 시간 복잡도 제한이 있을 때, 기본적으로 1s이다.
1s => 10^8^ 만큼의 반복을 수행한다.

이때 입력수가 충분히 많이 (10^9^) 주어졌을때, Linear Search등을 사용할 경우 10^9^ => 10s 의 시간복잡도를 가지기에 Logarithmic한 알고리즘이 요구되는걸 알 수 있다.

#### Merge Sort
MergeSort는 기본적으로 분할 정복 과정을 거친다.

```cpp
void mergeSort(int array[], int const begin, int const end)
{
	// excute k times
    if (begin >= end)
        return;

    int mid = begin + (end - begin) / 2;
    mergeSort(array, begin, mid); // T(N/2)
    mergeSort(array, mid + 1, end); // T(N/2)
    merge(array, begin, mid, end); // n
}
```
T(N) =  K + T($\frac{N}{2}$) + T($\frac{N}{2}$) + Kn
=> K(n+1) + 2T($\frac{N}{2}$) 이므로..
T(N) = Kn +2T($\frac{N}{2}$) 이 된다.

 결과적으로 MergeSort는 입력 값에서 반을 나누고 (Log^n^) 합치는(N) 과정을 거치므로 NLog2^n^이 된다.

#### Recursion

##### Power-1
```cpp
int power(int a, int n)
{
    if() return 1;
    return a* power(a,n-1);
}
```
Recursion에서 n의 조건에 따라 n회만큼 재귀적 호출이 일어난다. (Call Stack에 n만큼 스택이 쌓인다.)
시간 복잡도는 O(n)

##### Power-2
```cpp
int power(int a, int n)
{
    if() return 1;
    int halfPowerSquare = power(a,n/2) * power(a,n/2);

    if(n%2 != 0)
        return a* halfPowerSquare;

    return halfPowerSquare;
}
```
실질적으로 1번의 경우와 같다
시간복잡도는 O(n), 공간 복잡도는 O(Log^n^)을 가진다.

수행 횟수는 Log^n^으로 1+2+4....+ 2^Logn^으로  2^Log n^ 이며 O(n)이 된다.
공간 복잡도는 Call Stack이 쌓임에 따라 늘어나는데, 분할된 Max Depth만큼 복잡도를 가지기에 Log^n^이 된다.

##### Power-3
```cpp
int power(int a, int n)
{
    if() return 1;
    int halfPower = power(a,n/2);
    int halfPowerSquare = halfPower * halfPower;

    if(n%2 != 0)
    {
        return a* halfPowerSquare;
    }
    return halfPowerSquare;
}
```
위 로직은 공간 및 시간 복잡도 모두 Log^n^을 가진다.
a^n^-> (a^n/2^)^2^.... 의 연산 횟수를 가진다.
T(n) = K +  T($\frac{N}{2}$)
T($\frac{N}{2}$) = K +  T($\frac{N}{4}$)
.... => KLog^n^을 가지기에 O(Log^n^)이 된다.

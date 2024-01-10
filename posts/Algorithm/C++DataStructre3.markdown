---
layout: post
title:  "[자료구조/ 알고리즘] Bit 연산"
date: 2023-10-03 15:14:53 +0900
categories: Algorithm
tags : Algorithm, Data Structure

---
## Bitwise Operators

bit 연산자를 통해 기존 연산자에 비한 빠른 연산이 가능하다.
bit shift는 각각 *2^n^ , / 2^n^ 이다.

```cpp
int main()
{
    int x = 0;
    cout << (x<<1) << endl;
    cout << (x>>1) << endl;
    cout << ~x << "" << endl;
}
```
:warning: 2진 비트의 ~bit 는 반전 뒤집기이지만, 숫자 1을 not 연산자 사용시 -1이 되므로 유의하자.

### Bit manipulation

비트 연산을 통해 다양한 기능을 수행할 수 있다.
아래와 같이 수행하는데, 비트 연산에서의 중요점은 2^n^ -1의 형태 2진수라는 것이다.
이것은 1 <<n 형태로 나타낼 수 있으며 이는 연산에 log 만큼이 든다.
```cpp
//선택한 인덱스 비트 값 가져오기
int getIthBit(int n, int i)
{
    return (n & (1<<i)) >0 ? 1 : 0;
}

// 선택한 인덱스의 비트 삽입
void setIthBit(int &n, int i)
{
    n |= (1<<i);
}
// 선택한 인덱스의 bit를 제거
void clearIthBit(int &n, int i)
{
    int mask = ~(1 << i);
    n &= mask;
}
// 비트 삽입, 삭제
void updateIthBit(int &n, int i, int v)
{
    clearIthBit(n,i);
    int mask = (v << i);
    n |= mask;
}
// i구간 이전 비트 삭제
void clearLastIBits(int &n, int i)
{
    n= n&(~0<<i);
}

// i와 j 사이 범위 비트 삭제
void clearBitsRange(int &n, int i, int j)
{
    //case 1
    if(j>i)
    {
        cout << "error it can't be bigger j than i" << endl;
    }
    else
    {
        int mask = (-1 << i+1);
        n =(n&mask)|(~(-1<<j));
    }
    //case 2
    int a = -1 << i+1;
    int b = (1<< j)-1;
    // 2^j -1
    int mask = a|b;
    n = n&mask;
}

// 공배수체크
void CheckCommonMultiple(int n)
{
    // 2의 배수는 2^n -1 과 and 연산시 항상 0이 나온다 이를 이용한 연산
    if(n&(n-1) == 0)
    {
        cout << "number is CommonMultiple "<<endl;
    }
    else
    {
        cout << "number is not CommonMultiple "<<endl;
    }
}

// 특정 인덱스 비트 반전 xor
int FlipBit(int &n,int i)
{
    n ^= (1<<i);
}

//1인 비트 개수 세기
int count_Bits(int n)
{
    int cnt = 0;
    while (n > 0)
    {
        cnt += n&1;

        n = n>>1;
    }

    return cnt;

}

//2n & 2n-1은 0이라는 것을 응용해 0이 될때까지 개수 추가
int count_Bits_Hack(int n)
{
    int cnt=0;
    while(n >0)
    {
         n = n&(n-1);
         cnt++;
    }
    return cnt;
}

// 제곱 계산
// n -> 2진수로 변환을 응용
int fastExpo(int a, int n)
{
    int ans = 1;
    while (n > 0)
    {
        if(n&1)
        {
            ans = ans * a;
        }
        a = a*a;
        n = n >>1;
    }
}

// 10의 자리를 2진수로 변환
int ConvertToBinary(int n)
{
    int ans = 0;
    int power = 1;
    while(n
> 0)
    {
        ans += (n&1) *power;
        power *= 10;
        n= n >>1;
    }
    return ans;
}
```

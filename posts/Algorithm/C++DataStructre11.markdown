---
layout: post
title:  "[자료구조/ 알고리즘] Stack"
date: 2023-02-18 21:01:13 +0900
categories: Algorithm
tags : Algorithm, Data Structure

---

### Stack

스택은 LIFO의 규칙을 따르는 자료구조이다.

내부적으로 Linked List와 Vector 컨테이너로 구성될 수 있으며 STL 에 지정되어 있는 자료구조이다.

아래에서는 Linked List와 Vector 컨테이너를 통해 stack을 구현해보겠다.


### Node

```cpp
template<typename T>  
class Node
{
public:
    T data;
    Node<T> *next;

    Node(T d)
        data = d;
};
```
Stack에 저장될 Node 클래스이다.
위를 보면 template으로 타입이 지정되었다.

### 템플릿

```cpp
template<typename T>  
```
템플릿은 C#의 제너릭과 비슷한 기능이다.

템플릿은 class, method 동일하게 적용이 가능하다.



#### 템플릿 특수화

```cpp
// template
template <typename T>
T sum(T args1, T args2)
{
    T result = args1 + args2;
    return result;
}

// template with other arguments
template <typename T, typename U>
T sum(T args1, U args2)
{
    T result = args1 + args2;
    return result;
}

// override template
/* template specialization */
template<>
string sum<string>(string a, string b)
{
    string result = a+b + " string type";
    return result;
}

---
int main()
{
	sum<string>(a,b);
	sum<int>(c,d);
	stack<int> s_int;
	stack<char> s_char;
}
```
이처럼 정의된 템플릿에 타입에 따라 처리해주려면 재정의 하면 된다.

>  ## template specialization
> ##### 특수 자료형을 다르게 처리해주기위해 사용한다.

사용할 때 템플릿의 자료형을 명시적으로 표시해주나, 때에 따라선 컴파일러가 해당 내요을 유추해준다.

[템플릿 특수화](https://blockdmask.tistory.com/45)


### Stack Using Linked List

```cpp
template<typename T>
class Stack_L
{
private:
    Node<T> * head;
public:
    Stack()
    {
        head = NULL;
    }
    //헤드가 추가되는 방식
    push(T data)
    {
        Node<T> * n = new Node<T>(data);
        n->next = head;
        head = n;
    }

    bool IsEmpty()
    {
        return head == NULL;
    }
    T Top()
    {
        return head->data;
    }

    Pop()
    {
        if(head != NULL)
        {
            Node<T> * temp = head;
            head = head->next;
            delete temp;
        }
    }
};
```

내부적으로 Linked List로 구현된 stack이다.

LIFO에서 FI 부분이 head가 된다.
이는 새로운 요소가 추가 될때마다 head가 갱신된다.

---

###  Stack Using Vector

```cpp
template<typename T>
class Stack_V
{
    vector<T> arr;

public:
    Push(T data)
    {
        arr.
        arr.push_back();
    }
    Pop()
    {
        arr.pop_back();
    }
    T Top()
    {
        // int lastidx = arr.size()-1;
        // return arr[lastidx];
        return arr.back();
    }
    bool IsEmpty()
    {
        return arr.size() == 0;
    }
};
```

vector 컨테이너를 활용한 stack이다.

vector 자체가 STL에 속해있다 보니까 기본적인 기능은 모두 구현되어있다.

---

###  Stack STL
STL은 C++에서 제공하는 기본 자료구조와 함수 템플릿이다.


```cpp
stack<T> Stack;
Stack.pop();
Stack.push();
Stack.empty();
Stack.size();
```
위와같이 STL에 포함된 stack 자료구조를 이용해 처음부터 구현하지 않아도 사용이 가능하다.

---

### Push Bottom

stack은 적층되는 형식이므로 맨 아래에 넣는 기능은 없다.

이를 함수로 구현해보자.

```cpp
// ##### Challenge : Instert at bottom recursion
void InsertAtBottom(stack<int> &s, int data)
{
    if(s.empty())
    {
        s.push(data);
        return;
    }
    int d =s.top();
    s.pop();


    InsertAtBottom(s,data);
    s.push(d);
}
```
1. d= s.top(); 가장 위의 요소를 저장하고 pop() 시킨다.
2. 재귀호출을 통해 이를 반복한다.
3. s.empty()에 다다르면 맨 아래에 data 를 넣어주고 역순으로 재귀를 호출한다.
4. s.push(d)를 역순으로 호출하여 기존 스택에 저장된다.


---

### Reverse Recursion
```cpp
// ##### Challenge : Reverse Stack Recursion
void Reverse(stack<int> &s)
{
    if(s.empty())
    {
        return;
    }

    int d = s.top();
    s.pop();

    Reverse(s);
    // 콜스택의 적재와 호출이 반대이므로,
    // 재귀호출마다 저장된 값을 맨아래에 적재한다.
    InsertAtBottom(s,d);
}

int main()
{
    stack<int> books;
    books.push(1);
    books.push(2);
    books.push(3);
    books.push(4);

    // InsertAtBottom(books,5);
    Reverse(books);
    while (!books.empty())
    {
        cout << books.top() << endl;
        books.pop();
    }

    return 0;
}
```
Reverse 기능은 꽤나 복잡해 보인다.

1. d= s.top(); 가장 위의 요소를 저장하고 pop() 시킨다.
2. 재귀호출을 통해 이를 반복한다.
3. s.empty()에 다다르면 맨 아래에 InsertAtBottom(s,d)를 수행한다.
4. 재귀의 스택 순서와 호출 순서는 반대이므로 가장 마지막으로 쌓인 순서대로 가장 아래부터 적층된다.

즉 1->2->3->4 순서대로 쌓였다면 InsertAtBottom(s,d)를 만나서
```
\0 InsertAtBottom(s,1)
1  InsertAtBottom(s,2)
2 -> 1  InsertAtBottom(s,3)
3 -> 2 -> 1  InsertAtBottom(s,4)
4 -> 3-> 2-> 1
```

호출순서대로 맨 아래에 적재를 하게된다.

----

### Stock Span Sol

```cpp
void StockPan(int * prices, int n, int * span)
{
    //indices of the days
    stack<int> s;
    s.push(0);
    span[0] = 1;

    for (int i = 1; i <= n-1; i++)
    {
        int currentPrice = prices[i];

        // topmost element higher than current prices
        while (!s.empty() and prices[s.top()] <= currentPrice)
        {
            s.pop();
        }

        //가장 높았던 날짜 인덱스 값과 현재날을 빼 저장
        if(!s.empty())
        {
            int prev_highest = s.top();
            span[i] = i - prev_highest;
        }
        else
        {
            span[i] = i+1;
        }
        //push this element into the stack
        s.push(i);
    }
}

int main()
{
    int prices[] ={100,80,60,70,60,75,85};
    int n = sizeof(prices) / sizeof(int);
    int span[100000] = {0};
    StockPan(prices,n,span);

    for (int i = 0; i < n; i++)
    {
        cout << span[i]<< " ";
    }
}
```

날짜별로 차트간 span을 구하는 함수이다

stack 에 최근 높았던 값을 적재하고, 다음 날짜의 span을 계산할 때 해당 날짜 값을 stack에 적재해 최근 높았던 값을 최신화 한다.

만약 4일차의 70의 경우 이전 적재된 60의 값은 쓸모가 없으므로 stack에서 pop 해버리고 80과 인덱스를 비교해 span을 구한다.

Stack은 LIFO인 대표적인 자료구조이다.

그 특성에 맞게 재귀와도 잘 어울리고, 등록별 요소의 추가 삭제 또한 자유롭다.

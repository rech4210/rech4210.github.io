---
layout: post
title:  "[자료구조/ 알고리즘] Linked List"
date: 2023-02-18 21:01:13 +0900
categories: Algorithm
tags : Algorithm, Data Structure

---

## Linked List
요소간의 link를 통해 구현된 리스트이며 data와 다음 인자에 대한 포인터를 가진다.
LinkedList에는 head와 tail이 있으며, 첫번째 요소가 head 마지막 인자가 tail이 된다.


### member List Initiallize
```cpp
Node(int d):data(d),next(NULL){}
```
생성자 뒤에 : 를 붙여 멤버를 초기화 한다.
대입이 아니라 초기화다.

멤버가 const거나 reference 타입일 경우 초기화를 반드시 해주어야 하며 메모리 순으로 초기화 되기에 순서를 지켜줘야 한다.

리스트 이니셜라이즈
[List Initiallize](https://dhshin94.tistory.com/46)

### friend class CustomList

```cpp
class Node
{
	friend class CustomList;
};
class CustomList{};
```

friend 클래스를 선언하면 다른 클래스의 보호수준에 상관없이 (private, protected 멤버)접근 허용을 한다.
즉 CustomList는 Node의 private 멤버에 접근권한을 얻는다.

### Push

```cpp
PushFront(int data)
{
    if(head == NULL)
    {

        Node * n = new Node(data);
        head = tail = n;
    }

    else
    {
        Node * n = new Node(data);
        //use when pointer or reference type
        n->next = head;
        head = n;
    }
}

PushBack(int data)
{
    if(head == NULL)
    {
        Node * n = new Node(data);
        head = tail = n;
    }
    else
    {
        Node * n = new Node(data);
        tail->next = n;
        tail = n;
    }
}
```

Linked List에 요소를 넣는 함수이다.

각각 요소의 맨 앞, 뒤에 넣는 작업을 하며 요소를 넣을 때 이미 해당 위치에 데이터가 존재하면 List의 Node 포인터 레퍼런싱을 변경하는 방식으로 요소간 연결을 변경한다.

---

### Destructor

개체가 파괴될 때 자동적으로 실행되는 멤버 함수 (종료자)이다.

```cpp
~CustomList()
{
    if(head != NULL)
    {
        delete head;
        head = NULL;
    }
}
```
Unity의 Ondestroyed와 같은 역할을 한다.

---

### Pop
```cpp
void Pop_front()
{
    Node * temp = head;
    head = head->next;
    // 같은 reference를 가리키므로 destructor을 고려하더라도 NULL로 바꾸어야함.
    temp->next = NULL;
    delete temp;
}

void Pop_Back()
{
    Node * temp = head;
    Node * n;
    while (temp->next->next != NULL)
    {
        temp = temp->next;
    }
    n = temp->next;
    temp->next = NULL;
    delete n;
}
```

Pop은 요소 제거에 사용된다.

- Pop_front : 맨 앞의 요소를 제거한다 destructor가 정의되어 있으므로 temp를 생성해 포인터를 바꿔주어야 한다. 그렇지 않으면 기존 리스트의 destructor가 연쇄적으로 일어나 모든 링크된 요소의 destructor가 실행된다.
- Pop_back : 맨 뒤의 요소를 제거한다 해당 되는 요소의 prev 요소는 NULL로 바꾸어주어야 한다.

---

### Insert
```cpp
Insert(int data, int pos)
{
    if(pos == 0)
        PushFront(data);

    Node * temp = head;
    for (int i = 0; i < pos-1; i++)
    {
        temp = temp->next;
    }
    Node * n = new Node(data);
    n->next = temp->next;
    temp->next = n;

}
```

특정 인덱스에 요소를 추가하는 함수.

인덱스의 이전 위치까지 이동한 후에 새로운 요소를 사이에 추가하고 링크를 추가된 요소 기준으로 변경한다.

![Insertion in Linked List - GeeksforGeeks](https://media.geeksforgeeks.org/wp-content/cdn-uploads/gq/2013/03/Linkedlist_insert_middle.png)

<br>

---

### Remove
```cpp
void Remove(int d)
{
    Node* temp = head;
    while (temp != NULL)
    {
        if(temp->next->data == d)
        {
            Node* n = temp->next;
            temp->next = n->next;
            n->next = NULL;
            delete n;
            break;
        }
        temp = temp->next;
    }
}
```

![Deletion in Linked List - GeeksforGeeks](https://media.geeksforgeeks.org/wp-content/uploads/20210211175616/Untitleddesign.png)

특성 데이터 요소를 삭제한다.
링크된 인덱스에 따른 요소를 삭제할 수도 있다.

Insert에서 추가된 요소 기준으로 링크가 재연결 되었으나, Remove에서는 기존 요소의 링크를 끊고 다음으로 연결시켜준다.

---

### Search

```cpp
bool Search(Node * head, int key)
{
    while (head != NULL)
    {
        if(head->data == key)
            return true;
        head = head->next;
    }
    return false;

}

bool SearchRecursive(Node * head, int key)
{
    //base case
    if(head == NULL)
        return false;

    if (head->data == key)
        return true;
    else
    {
        SearchRecursive(head->next, key);
    }
    //recursive case
}

요소가 존재하는지 탐색하는 함수이다.

아래는 재귀 방식으로 구현되었다.

---


int SearchingIndexRecursive(int key)
{
    int idx = SearchHelper(head, key);
    return idx;
}

private:
int SearchHelper(Node * start, int key)
{
    // base case
    if(start == NULL)
        return -1;

    if(start->next->data == key)
        return 0;

    //remaining part of the linked list recursion
    int subIndex = SearchHelper(start->next,key);
    if(subIndex == -1)
        return -1;


    return subIndex+1;
}
```

인덱스를 탐색하는 함수이다.
마찬가지로 재귀로 구현되었다.

재귀 호출을 계속 수행하며
```cpp
if(start->next->data == key)
    return 0;
```
해당 부분을 기점으로 재귀 호출 횟수만큼 subIndex+1; 를 반환한다.
마지막 subIndex+1 의 값이 대상 인덱스 위치이다.

---


### Reverse
```cpp
void ReverseList()
{
    Node * temp;
    Node * current = head;
    Node * prev = NULL;

    while (current != NULL)
    {
        temp = current->next;
        current->next = prev;
        prev = current;
        current = temp;
    }
    head = prev;
}
```
![Reverse a Linked List - GeeksforGeeks](https://media.geeksforgeeks.org/wp-content/cdn-uploads/RGIF2.gif)

리스트를 반대로 뒤집는 함수이다.


1. temp = current->next; 현재 요소의 링크를 저장해둔다.
2. current->next = prev; 현재 요소의 링크를 반대로 바꾼다.
3. prev = current;  현재 요소를 prev로 변경한다.
4. current = temp;  현재 위치를 다음 요소로 변경한다.

위의 과정을 반복적으로 실행하면 Reverse 함수가 수행된다.

마지막으로 head의 값을 마지막 prev 값으로 바꿔준다.

---

### 전체 코드

```cpp
#include<iostream>
#include<cstring>
// #include "CustomList.h"
using namespace std;

class Node
{
public:
    int data;
    Node * next;
    //Initaillize List Assingment
    Node(int d):data(d),next(NULL){}

    //종료자의 재귀적 호출
    ~Node()
    {
        if(next != NULL)
        {
            delete next;
        }
    }
    friend class CustomList;
};



class CustomList
{
public:
    Node * head;
    Node * tail;

    CustomList():head(NULL),tail(NULL){}

    PushFront(int data)
    {
        if(head == NULL)
        {

            Node * n = new Node(data);
            head = tail = n;
        }

        else
        {
            Node * n = new Node(data);
            //use when pointer or reference type
            n->next = head;
            head = n;
        }
    }

    PushBack(int data)
    {
        if(head == NULL)
        {
            Node * n = new Node(data);
            head = tail = n;
        }
        else
        {
            Node * n = new Node(data);
            tail->next = n;
            tail = n;
        }
    }

    Node * Begin()
    {
        return head;
    }

    Insert(int data, int pos)
    {
        if(pos == 0)
            PushFront(data);

        Node * temp = head;
        for (int i = 0; i < pos-1; i++)
        {
            temp = temp->next;
        }
        Node * n = new Node(data);
        n->next = temp->next;
        temp->next = n;

    }

    bool Search(Node * head, int key)
    {
        while (head != NULL)
        {
            if(head->data == key)
                return true;
            head = head->next;
        }
        return false;

    }

    bool SearchRecursive(Node * head, int key)
    {
        //base case
        if(head == NULL)
            return false;

        if (head->data == key)
            return true;
        else
        {
            SearchRecursive(head->next, key);
        }
        //recursive case
    }

    void Pop_front()
    {
        Node * temp = head;
        head = head->next;
        // 같은 reference를 가리키므로 destructor을 고려하더라도 NULL로 바꾸어야함.
        temp->next = NULL;
        delete temp;
    }

    void Pop_Back()
    {
        Node * temp = head;
        Node * n;
        while (temp->next->next != NULL)
        {
            temp = temp->next;
        }
        n = temp->next;
        temp->next = NULL;
        delete n;
    }

    void Remove(int idx)
    {
        Node* temp = head;
        while (temp != NULL)
        {
            if(temp->next->data == idx)
            {
                Node* n = temp->next;
                temp->next = n->next;
                n->next = NULL;
                delete n;
                break;
            }
            temp = temp->next;
        }

    }
    ~CustomList()
    {
        if(head != NULL)
        {
            delete head;
            head = NULL;
        }
    }
#pragma region //searching Index
    int SearchingIndexRecursive(int key)
    {
        int idx = SearchHelper(head, key);
        return idx;
    }

    void ReverseList()
    {
        Node * temp;
        Node * current = head;
        Node * prev = NULL;

        while (current != NULL)
        {
            temp = current->next;
            current->next = prev;
            prev = current;
            current = temp;
        }
        head = prev;
    }
private:
    int SearchHelper(Node * start, int key)
    {
        // base case
        if(start == NULL)
            return -1;

        if(start->next->data == key)
            return 0;

        //remaining part of the linked list recursion
        int subIndex = SearchHelper(start->next,key);
        if(subIndex == -1)
            return -1;


        return subIndex+1;
    }
#pragma endregion
};

int main()
{
    CustomList myList;
    myList.PushFront(0);
    myList.PushFront(1);
    myList.PushFront(4);
    myList.PushFront(5);
    myList.PushFront(3);

    myList.PushBack(2);
    myList.Insert(3,1);
    myList.Remove(3);
    myList.Pop_Back();

    Node * temp = myList.Begin();
    while (temp != NULL)
    {
        cout << temp->data << " -> ";
        temp = temp->next;
    }

    cout << endl;
    myList.ReverseList();
    temp = myList.Begin();

    while (temp != NULL)
    {
        cout << temp->data << " -> ";
        temp = temp->next;
    }

    return 0;
}
```

리스트의 레퍼런스를 항상 고려하자
-> 특히 pop 이용시 레퍼런스를 저장하는 temp 를 잘 써야한다.

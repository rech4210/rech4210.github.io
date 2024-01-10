---
layout: post
title:  "[자료구조/ 알고리즘] 객체지향 프로그래밍 OOP"
date: 2023-10-28 13:44:13 +0900
categories: Algorithm
tags : Algorithm, Data Structure

---
## OOP
객체지향 프로그래밍이다.
유니티 C#을 다루는 입장에선 꽤나 익숙한 개념으로
캡슐화, 상속, 모듈화등의 기능이 있다.


### Class
많이 사용하는 Class는 Object의 설계도를 담당한다.
 Class는 Data member, Function으로 구성되며 공개 설정에따라 public, private으로 표시된다.

Class는 메모리에 존재하지 않으며 Object로 인스턴스화 되며 메모리를 점유하게 된다.

```cpp
class cs
{
    public: cs()
        {
            value = 1;
            model_id = 3;
            name = "prototype";
        }
    private:
        int value;
        int model_id;
        string name;

    public:
        void Print()
        {
            cout << model_id << " : " << name << endl;
        }
};
int main()
{
    cs myCS;
    myCS.Print();
    return 0;
}
```

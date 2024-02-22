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

### Copy Constructor

Constructor는 오브젝트가 생성될때 딱 한번 실행되는 생성자란 것이다.
이러한 생성자는 다양한 방법으로 재정의 될 수 있는데, 그 중 Copy 방법이 있다.

```cpp
Product(int id, const char * n, int mrp, int selling_price)
{
    this->id = id;
    this->name = new char[strlen(n)+1];
    strcpy(name, n);
    this->mrp = mrp;
    this->selling_price = selling_price;
}
```
위의 내용을 수행하는 생성자를 만들었다고 하자.
이 생성자는 custom 생성자로 기본 생성자와 달리 개발자가 정한 방식대로 오브젝트를 생성시킨다.

이러한 생성자를 Copy 하는 방법이 있다.

```cpp
//Copy Constructor must needs & type
Product(Product &X)
{
    this->id = X.id;
    this->name = new char[strlen(X.name)+1]; // Deep Copy
    strcpy(name, X.name);
    this->mrp = X.mrp;
    this->selling_price = X.selling_price;
}
```
복사할 대상을 &(레퍼런스) 타입으로 가져온다.
그 후 수행할 내용을 정의해주면 된다.

한데, 한 가지 유의해야 할 점이 있다.
바로 reference 타입의 Shallow Copy, Deep Copy의 차이이다.


>### Shallow Copy, Deep Copy
얕은 복사와 깊은 복사이다.

얕은 복사
- 메모리에 대한 주소를 공유한다
- reference type의 경우 얕은 복사를 하게 된다.

깊은 복사
-   데이터 전체를 복제하여 독립적인 메모리에 할당한다.
-   value type의 경우 깊은 복사를 하게 된다.

<br>

[참고 shallow vs deep](https://velog.io/@ellyheetov/Shallow-Copy-VS-Deep-Copy)

```cpp
int main()
{
    Product Stuff(0,"stuff",0,0);
    Stuff.ShowInfo();

    Product Sword(Stuff);
    Sword.setMrp(5000);
    Sword.setPrice(2000);
    Sword.SetName("Sword"); //
    Sword.ShowInfo();
}
Sword|0|0
Sword|2000|5000
// Stuff와 Sword가 같은 name array를 공유하므로 Shallow Copy
// 얕은 복사를 통해 Suff마저 바뀌게 된다.
```
> 여기서 깊은 복사, 얕은 복사의 개념을 유의해야 한다.

<br>

그렇다면 reference 타입의 개체들을 Deep Copy  해줘야한다.
아래의 경우는 독자적인 메모리를 할당해준 후, 깊은 복사를 시켜준 예시이다.

```cpp
class Product
{
	// Product data....

    Product(int id, const char * n, int mrp, int selling_price)
    {
        this->id = id;
        this->name = new char[strlen(n)+1];
        strcpy(name, n);
        this->mrp = mrp;
        this->selling_price = selling_price;
    }

    void SetName(const char *n)
        //Shallow Copy
        strcpy(name, n);

    //Copy Constructor must needs & type
    Product(Product &X)
    {
        this->id = X.id;
        this->name = new char[strlen(X.name)+1];//Deep Copy
        strcpy(name, X.name);
        this->mrp = X.mrp;
        this->selling_price = X.selling_price;
    }
};

int main()
{
    Product Stuff(0,"stuff",0,0);

    Product Sword(Stuff);
    Sword.setMrp(5000);
    Sword.setPrice(2000);
    Sword.SetName("Sword");

    Product Staff = Stuff;
    Staff.SetName("Staff");
    Staff.setMrp(7000);
    Staff.setPrice(3500);
}
stuff|0|0
Sword|2000|5000
Staff|3500|7000
```
### Destructor
생성자가 있듯이 클래스가 생성된 scope를 지날 때 소멸자를 실행시켜준다.

소멸자는 개체가 할당된 메모리를 해제해주기 위해 생긴 개념으로, 개체의 역할이 끝날때 해당되는 일 (메모리 해제, reference type null 초기화, 메모리 leak 관리 등)을 수행한다.

#### memory leak
사용이 끝난 메모리 블록을 해제하지 않아 생긴 메모리 낭비.
<br>

이를 방지하기위해 destructor를 사용한다.
사용 방법은 아래와 같이 ~className 으로 사용한다.
```cpp
~Product()
{
    cout << "delete " << name << endl;
    if(name != NULL)
    delete[] name;
    name = NULL;
}
stuff|0|0      
Sword|2000|5000
Staff|3500|7000

delete Staff
delete Sword
delete stuff
```

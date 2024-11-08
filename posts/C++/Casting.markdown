---
layout: post
title: "캐스팅,탬플릿과 컴파일의 성능 관계"
description:
date: 2024-11-04 15:01:04 +0900
tags : C++
---
### C++의 캐스팅 개념

- 정수 to 정수 : long long -> int로 변환할 경우, 리틀 엔디안 개념에 따라 상위 4바이트를 버리며 나머지 하위 4바이트가 저장된다.
- 정수 to 실수 : 정수부의 부동소수점을 붙여준다 (컴파일러 기준 다름)
- 실수 to 정수 : 실수부의 정보를 버림으로 캐스팅된다

int -> char 의 경우도 마찬가지로 상위 3바이트를 버리게 된다.

>포인터의 명시적 표시의 이유?
>예를들어 int*의 경우 int가 4byte 이기에 최하위 비트 기준 4byte 만큼의 데이터를 가리킬 수 있음을 뜻한다.

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/6e23c35f-005c-4655-8fb1-c1a5c72567d9)



## Static, Dynamic, Reinterpret_cast, Const
- static_cast
	- 정적 캐스팅 방법으로 컴파일 타임에 타입을 체크한다. 정보의 손상을 감지하여 잘못된 캐스팅시 컴파일 타임 즉시 에러를 출력하며, 파생 클래스와 클래스간의 캐스팅을 주로 한다.

- dynamic_cast
	- 동적 캐스팅 방법, 런타임에 객체 타입을 체크하므로 안정성을 높일 수 있다. 정적 캐스팅과 다르게 객체의 타입을 런타임에 체크하므로 캐스팅이 유연하다 (파생관계가 아니여도 우선 캐스팅을 수행해보고, 실패시 null또는 예외를 반환하므로 예외 처리가 가능하다)

- reinterpret_cast
	- 동적 캐스팅 방법,  reinterpret_cast는 관련 없는 클래스 간 형 변환에도 사용할 수 있지만, 런타임에 검사나 변환을 수행하지 않음.   가장 큰 차이점은 reinterpret_cast는 가리키는 실제 객체가 대상 클래스와 실제로 일치하는지 확인하지 않고 항상 성공하여 한 객체 유형의 메모리 주소를 다른 객체 유형으로 변환한다는 것인데, 이는 원시 타입 포인터등에 사용될 수 있다.

	-  일부 데이터에 대한 원시 포인터를 반환하는 C 라이브러리 함수를 호출하는 경우.  (외부 라이브러리의 타입)
	-  관리되지 않는 코드(예: 어셈블리 언어)로 작업하고 있으며 메모리에 직접 액세스해야 하는 경우.

- const_cast
	- const로 선언된 개체의 const 속성을 잠시 해제하여 내부 데이터를 수정하기 위해 사용한다.   (왠만하면 절대 사용하지말자)
	- 임시 수정: 짧은 기간 동안 상수 객체를 수정해야 하지만 전체 객체를 상수가 아닌 것으로 만들고 싶지 않을 때가 있습니다. const_cast를 사용하면 이 작업을 수행할 수 있습니다.

	- 레거시 코드 호환성: const 정합성을 일관성 없이 사용하는 레거시 코드로 작업하는 경우, const_cast를 사용하면 이러한 이전 API에서 작동하도록 코드를 조정할 수 있습니다.



### 캐스팅

```cpp
#include <iostream>

class Animal {
public:
    virtual void sound() = 0;
};

class Dog : public Animal {
public:
    void sound() { std::cout << "Woof!" << std::endl; }
};

int main() {
    // Static Cast Example
    Animal* animalPtr = new Dog();
    Dog* dogPtr = static_cast<Dog*>(animalPtr);
    dogPtr->sound();  // Output: Woof!

    // Dynamic Cast Example
    // This will throw an exception at runtime!
    Animal* animal2Ptr = new Cat();  
    if (Dog* dog2Ptr = dynamic_cast<Dog*>(animal2Ptr)) {
        dog2Ptr->sound();
    } else {
        std::cout << "Not a Dog!" << std::endl;
    }

	// Reinterpret Cast Example
    Animal* animalPtr = new Dog();
    Dog* dogPtr = reinterpret_cast<Dog*>(animalPtr);
    dogPtr->sound();  // Output: Woof!

	// Reinterpret Cast Example 2 (Non exsist)
    Animal* animalPtr = new Dog();
    //can use undefined type
    //will not throw an exception at runtime but cause error maybe
    Tiger* tigerPtr = reinterpret_cast<Tiger*>(animalPtr);
    tigerPtr->sound();  // error

	//const cast
	//스택영역에 저장된 const로 readonly 제한 영역이 아님. 수정 o
	const int a = 3;
	const int* pa = &a;
	int* paa = const_cast<int*>(pa);
	*paa = 6;

	// readonly 영역이므로 쓰기 제한이 걸려버림.
	const char* c = "gfg";
	char* d = const_cast<char*>(c);
	*d = 'c';

    return 0;
}
```


#### 한줄 요악

* 수행 중인 작업에 대해 확실하고 컴파일 오류를 방지하려면 static_cast (statitc_cast 는 컴파일 타입에 이루어지므로 런타임 리소스를 아낄 수 있다.)
* 유연성이 더 필요하거나 다형성 객체로 작업하는 경우 dynamic_cast 사용.
* 자신이 하는 일을 정말 잘 알고 있고 잠재적인 문제를 신경 쓰지 않는다면 reinterpret_cast를 사용.

### 얼탱이 없는 경우

```cpp
class Parent {

};

class A :public Parent
{
public:
    void A_func() {  };
};

class B:public Parent {
public:
    void B_func() {  };
};


Parent* pa = new Parent();
A* a = static_cast<A*>(pa);

// A != B 원래 이게 되면 안되는데, 상속을 개발자가 잘못하면 이렇게 된다.
Parent* a_p = static_cast<Parent*>(a);
a_p = static_cast<B*>(a_p);


```

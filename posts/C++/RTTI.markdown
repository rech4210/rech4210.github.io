---
layout: post
title: "RTTI 런타임 타입 정보"
description:
date: 2024-11-04 15:01:04 +0900
tags : C++
---
## RTTI (런타임 타입 정보)

C++에서 `dynamic_cast`는 RTTI를 사용하여 객체의 실제 타입을 확인합니다. RTTI는 프로그램이 실행되는 동안 객체의 타입 정보를 제공하는 메커니즘입니다. 이를 통해 `dynamic_cast`는 객체가 특정 타입인지 확인할 수 있습니다.

### C++ 예제 코드

```cpp
#include
#include
using namespace std;

struct Base {
 virtual ~Base() {}
};

struct Derived : public Base {
 void doSomething() {
 cout << "Derived doing something!" << endl;
 }
};

void process(Base* b) {
 // typeid를 사용하여 객체의 실제 타입을 확인
 cout << "Type of b: " << typeid(*b).name() << endl;
 // dynamic_cast를 사용하여 Derived로 안전하게 캐스팅
 if (Derived* d = dynamic_cast(b)) {d->doSomething();}
 else {cout << "b is not of type Derived." << endl;}
}

int main() {
 Base* base = new Base();
 Derived* derived = new Derived();
 Base* base2 = new Derived();

 process(base); // Base 타입
 process(derived); // Derived 타입
 process(base2); // Derived 타입

 delete base;
 delete derived;
 delete base2;

 return 0;
}
```

이러한 과정을 통해  객체의 타입을 런타임에 확인할 수 있다.
RTTI는 가상함수가 있는 클래스를 반드시 필요로 한다.

## 2. 가상 함수와 가상 테이블

C++에서 클래스가 가상 함수를 포함하고 있으면, 컴파일러는 해당 클래스의 객체에 대한 가상 테이블(vtable)을 생성합니다. 이 vtable은 클래스의 가상 함수에 대한 포인터를 포함하고 있으며, 각 객체는 자신의 vtable에 대한 포인터를 가지고 있습니다. 이 포인터는 `객체의 실제 타입`을 나타내는 데 사용됩니다.


### 예제 1 : Dynamic_Cast의 RTTI 정보 확인 에러
```cpp
struct S {
 virtual ~S() {}
};
int main() {
 S* p = new S();
 void* v = dynamic_cast(p);
 S* p1 = dynamic_cast(v); // 이 부분에서 에러 발생
}
```

위 코드에서 `S* p1 = dynamic_cast(v);` 부분이 에러가 나는 이유는 `dynamic_cast`가 안전한 캐스팅을 위해 RTTI 정보를 필요로 하기 때문입니다. `dynamic_cast`는 포인터나 참조를 대상으로 사용할 수 있으며, 기본적으로 `dynamic_cast`는 다형성을 지원하는 클래스(즉, 가상 함수가 있는 클래스)에서만 작동합니다.

`void*` 타입은 RTTI 정보를 가지고 있지 않기 때문에, `dynamic_cast`를 사용할 수 없습니다. 따라서 `v`를 `void*`로 캐스팅한 후 다시 `S*`로 캐스팅하려고 하면 컴파일 에러가 발생합니다. `dynamic_cast`는 `void*`를 `S*`로 안전하게 변환할 수 없기 때문입니다.

## 3. `dynamic_cast`의 작동 방식

`dynamic_cast`가 호출되면, 다음과 같은 과정이 진행됩니다:

1. **타입 정보 확인**: `dynamic_cast`는 변환하려는 타입의 RTTI 정보를 사용하여 객체의 실제 타입을 확인합니다. 이 정보는 가상 함수가 있는 클래스에 대해 생성된 RTTI에 포함되어 있습니다.

2. **vtable 사용**: `dynamic_cast`는 객체의 vtable 포인터를 통해 해당 객체의 실제 타입을 확인합니다. 객체의 vtable 포인터는 객체가 어떤 클래스의 인스턴스인지를 나타내는 정보를 포함하고 있습니다.

3. **타입 비교**: `dynamic_cast`는 변환하려는 타입과 객체의 실제 타입을 비교합니다. 이 비교는 RTTI 정보를 기반으로 수행됩니다. 만약 두 타입이 일치하면, 변환이 성공하고 해당 포인터가 반환됩니다. 일치하지 않으면, `dynamic_cast`는 `nullptr`를 반환합니다 (포인터 타입의 경우) 또는 예외를 발생시킵니다 (참조 타입의 경우).


### 예제 2:  dynamic_cast의 작동 방식

```cpp
struct B {
 virtual void doSth() {cout << "hello" << endl;}
};

struct A : public B {
 void doSth() override { cout << "A says hello!" << endl;}
};

struct C : public B {
 void doSth() override {cout << "C says hello!" << endl;}
};

int main() {
 B* b = new A(); // B 타입의 포인터가 A 객체를 가리킴

 // dynamic_cast를 사용하여 A로 캐스팅
 if (A* a = dynamic_cast(b)) {a->doSth(); // A의 doSth 호출 }
 // dynamic_cast를 사용하여 C로 캐스팅
 if (C* c = dynamic_cast(b)) {c->doSth(); // 이 부분은 실행되지 않음 }

 delete b;
 return 0;
}
```

위 예제에서 `dynamic_cast`는 런타임에 `b`가 실제로 어떤 타입의 객체를 가리키고 있는지를 확인합니다. `b`는 `A` 타입의 객체를 가리키고 있으므로, `dynamic_cast(b)`는 성공적으로 `A*`로 캐스팅됩니다. 반면, `dynamic_cast(b)`는 실패하고 `nullptr`을 반환합니다.

이 과정에서 C++ 컴파일러는 각 객체에 대한 vptr(가상 함수 테이블 포인터)를 사용하여 해당 객체의 실제 타입을 확인합니다. 각 클래스는 vtable을 가지고 있으며, 이 vtable은 해당 클래스의 가상 함수에 대한 포인터를 포함합니다. `dynamic_cast`는 이 vptr을 통해 객체의 실제 타입을 확인하고, 올바른 타입으로 안전하게 캐스팅할 수 있는지를 판단합니다.

이러한 방식으로 RTTI는 C++에서 객체의 타입을 런타임에 안전하게 확인하고, 필요한 경우 적절한 타입으로 캐스팅할 수 있도록 도와줍니다.

---
layout: post
title: "생성자 (복사, 이동, 복사 대입)"
description:
date: 2024-11-04 15:01:04 +0900
tags : C++
---

## 생성자

생성자는 객체가 생성될 때 초기화되는 특수 멤버 함수이다.

1. 기본 생성자: 가장 기본적인 생성자로, 매개변수를 받지 않고 기본값으로 객체를 초기화합니다.  
2. 복사 생성자: 복사 생성자는 다른 기존 객체에서 모든 멤버를 복사하여 새 객체를 만듭니다.  
3. 생성자 이동: 이동 생성자는 메모리와 같은 리소스의 소유권을 한 객체에서 다른 객체로 이전합니다.


## 1. 생성자 (Constructor)
생성자는 객체가 생성될 때 호출되는 함수입니다. 객체의 초기화를 담당합니다.

#### 호출되는 상황:
- 객체가 선언될 때
- 객체가 동적으로 할당될 때 (`new` 연산자 사용)

#### 예제 코드:

```cpp
class MyClass {
public:
 int value;
 // 기본 생성자
 MyClass() : value(0) {
 std::cout << "Default Constructor called" << std::endl;
 }
 // 매개변수가 있는 생성자
 MyClass(int v) : value(v) {
 std::cout << "Parameterized Constructor called" << std::endl;
 }
};
```


---

## 2. 복사 생성자 (Copy Constructor)
복사 생성자는 객체가 다른 객체로부터 복사될 때 호출되는 함수입니다. 깊은 복사와 얕은 복사를 관리할 수 있습니다.

#### 호출되는 상황:
- 객체가 다른 객체로 초기화될 때
- 객체가 함수에 값으로 전달될 때
- 함수가 객체를 값으로 반환할 때

#### 사용하는 이유
- call by reference
- call by value
위의 상황을 분리해서 사용하기위해 만들어졌다.

#### 예제 코드:
```cpp
#include

class MyClass {
public:
 int value;

 // 기본 생성자
 MyClass(int v = 0) : value(v) {
 std::cout << "Constructor called" << std::endl;
 }

 // 정의된 복사 생성자
 MyClass(const MyClass& other) : value(other.value) {
 std::cout << "Copy Constructor called" << std::endl;
 }
};

void func(MyClass obj) {
 // 함수 인자로 객체가 전달될 때 복사 생성자 호출
}

int main() {
 MyClass obj1(10);
 MyClass obj2 = obj1; // 복사 생성자 호출
 func(obj1); // 복사 생성자 호출
 return 0;
}
```

---

## 3. 이동 생성자 (Move Constructor)
이동 생성자는 객체가 다른 객체로부터 이동될 때 호출되는 함수입니다. 자원의 소유권을 이전할 때 사용됩니다.

#### 호출되는 상황:
- 객체가 임시 객체로부터 초기화될 때
- 객체가 함수에서 임시 객체로 반환될 때

#### 사용하는 이유
- 소유권의 이전
- 크기가 큰 객체의 복사에 사용.

#### 예제 코드:
```cpp
#include
#include // std::move

class MyClass {
public:
 int* data;

 // 기본 생성자
 MyClass(int size = 0) : data(size ? new int[size] : nullptr) {
 std::cout << "Constructor called" << std::endl;
 }

 // 소멸자
 ~MyClass() {
 delete[] data;
 std::cout << "Destructor called" << std::endl;
 }

 // 복사 생성자
 MyClass(const MyClass& other) : data(other.data ? new int[*other.data] : nullptr) {
 std::cout << "Copy Constructor called" << std::endl;
 }

 // 이동 생성자 &&는 해당 값으로 r-value만 받는다는 의미이다.
 MyClass(MyClass&& other) noexcept : data(other.data) {
 other.data = nullptr;
 std::cout << "Move Constructor called" << std::endl;
 }
};

MyClass createObject() {
 MyClass temp(10);
 return temp; // 이동 생성자 호출
}

int main() {
 MyClass obj1 = createObject(); // 일반 생성자 호출
 MyClass obj2 = std::move(obj1); // 이동 생성자 호출
 return 0;
}
```

### 요약
- **생성자**는 객체가 생성될 때 호출됩니다.
- **복사 생성자**는 객체가 다른 객체로부터 복사될 때 호출됩니다.
- **이동 생성자**는 객체가 다른 객체로부터 이동될 때 호출됩니다.

### L-value와 R-value의 차이점

- **L-value (Left Value)**:
 - 메모리 주소를 가지고 있으며, 할당 연산자의 왼쪽에 올 수 있는 값입니다.
 - 수정 가능하며, 변수나 참조를 나타냅니다.
 - 예: `int x = 10;`에서 `x`는 L-value입니다.

- **R-value (Right Value)**:
 - 메모리 주소를 가지지 않으며, 일시적인 값을 나타냅니다.
 - 수정 불가능하며, 상수나 임시 객체를 나타냅니다.
 - 예: `int y = move(x + 5);`에서 `move(x + 5)`는 R-value입니다.
 - 이러한 임시변수는 스택 영역에 주로 저장됨.


#### 임시변수 소유권 이전 후 메모리 해제

- **소유권 이전**:
 - 이동 생성자가 호출되면, 원본 객체의 자원이 새로운 객체로 이동됩니다.
 - 원본 객체는 더 이상 그 자원을 소유하지 않으며, 일반적으로 원본 객체의 자원 포인터는 `nullptr`로 설정됩니다.

- **메모리 해제**:
 - 새로운 객체가 소멸될 때, 이동된 자원은 새로운 객체의 소멸자에 의해 해제됩니다.
 - 원본 객체는 자원을 소유하지 않으므로, 소멸자에서 자원을 해제하지 않습니다.

#### move semantics
move()는 소유권을 이동시켜주는 함수로, 컴파일러에게 해당하는 객체가 `r - value`라는것을 인지시켜준다. 이를 사용해 임수 변수 생성 및 메모리 복사에 대한 오버헤드 감소 이점을 얻는다.

forwarder 함수의 r l value


https://pppgod.tistory.com/7 move 함수



### 난이도: 쉬움

**문제 1: 기본 생성자**
```cpp
class MyClass {
public:
 int* data;
 MyClass() {
 data = new int(10);
 }
 ~MyClass() {
 delete data;
 }
};

int main() {
 MyClass obj;
 std::cout << *(obj.data) << std::endl;
 return 0;
}
```
위 코드에서 기본 생성자가 하는 역할을 설명하고, 메모리 할당과 해제에 대해 설명하세요.

**문제 2: 복사 생성자**
```cpp
class MyClass {
public:
 int* data;
 MyClass() {
 data = new int(10);
 }
 MyClass(const MyClass& other) {
 data = new int(*(other.data));
 }
 ~MyClass() {
 delete data;
 }
};

int main() {
 MyClass obj1;
 MyClass obj2 = obj1;
 std::cout << *(obj2.data) << std::endl;
 return 0;
}
```
위 코드에서 복사 생성자가 하는 역할을 설명하고, 왜 복사 생성자가 필요한지 설명하세요.

> call by refernce, value 깊은복사, 얕은복사
MyClass m2 = m ;  /  myClass m2(m)   복사대입연산자 vs 복사연산자
대입 연산자의 경우 m2 = m 했을땐 default 또는 복사 대입 연산자가 호출된다.
MyClass m2 = m = m_; << 대입연산자(m = m_), 복사생성자(m2 = m) 로 불려진다

### 난이도: 중간

**문제 3: 이동 생성자**
```cpp
class MyClass {
public:
 int* data;
 MyClass() {
 data = new int(10);
 }
 MyClass(MyClass&& other) noexcept {
 data = other.data;
 other.data = nullptr;
 }
 ~MyClass() {
 delete data;
 }
};

int main() {
 MyClass obj1;
 MyClass obj2 = std::move(obj1);
 std::cout << (obj1.data == nullptr) << std::endl;
 std::cout << *(obj2.data) << std::endl;
 return 0;
}
```
위 코드에서 이동 생성자가 하는 역할을 설명하고, 이동 생성자가 필요한 이유를 설명하세요.
> 소유권 이전을 한다. rvalue 처리를 위해 (복사 생성자가 필요없는 경우, 속도 상향을 위해 사용함).

**문제 4: 복사 생성자와 이동 생성자의 차이점**
```cpp
class MyClass {
public:
 int* data;
 MyClass() {
 data = new int(10);
 }
 MyClass(const MyClass& other) {
 data = new int(*(other.data));
 }

 // rvalue 타입 참조 (변환 아님 lvalue는 안받음)
 // &의 경우 lvalue
 MyClass(MyClass&& other) noexcept {
 data = other.data;
 other.data = nullptr;
 }
 ~MyClass() {
 delete data;
 }
};

int main() {
 MyClass obj1;
 MyClass obj2 = obj1; // 복사 생성자 호출

 //move 사실상 또 rvalue로 만들어주겠다.
 MyClass obj3 = std::move(obj1); // 이동 생성자 호출

 std::cout << *(obj2.data) << std::endl;
 std::cout << (obj1.data == nullptr) << std::endl;
 std::cout << *(obj3.data) << std::endl;
 return 0;
}
```
위 코드에서 복사 생성자와 이동 생성자가 각각 호출되는 시점을 설명하고, 이 둘의 차이점을 설명하세요.

### 난이도: 어려움

**문제 5: 메모리 관리와 성능**
```cpp
class MyClass {
public:
 int* data;
 MyClass() {
 data = new int(10);
 }
 MyClass(const MyClass& other) {
 data = new int(*(other.data));
 }
 MyClass(MyClass&& other) noexcept {
 data = other.data;
 other.data = nullptr;
 }
~MyClass(){
	};
};
//위 코드에서 `process` 함수 호출 시 복사 생성자와 이동 생성자가 각각 호출되는 상황을 설명하고, 메모리 관리와 성능 측면에서 어떤 차이가 있는지 설명하세요.

void process(MyClass obj) {
 std::cout << *(obj.data) << std::endl;
}

int main() {
 MyClass obj1;
 process(obj1); // 복사 생성자 호출 -> 함수 호출이기에 우선 call by value로 호출됨. lvalue
 process(std::move(obj1)); // 이동 생성자 호출 ->  원본 주소에 대하여 process를 통해 소유권을 바꾸게  됨 rvalue.
 return 0;
}
```


**문제 6: 복사 생성자와 이동 생성자의 활용**
```cpp
class MyClass {
public:
 int* data;
 MyClass() {
 data = new int(10);
 }
 MyClass(const MyClass& other) {
 data = new int(*(other.data));
 }
 MyClass(MyClass&& other) noexcept {
 data = other.data;
 other.data = nullptr;
 }
};
// 위 코드에서 `createObject` 함수가 반환할 때 어떤 생성자가 호출되는지 설명하고, 복사 생성자와 이동 생성자가 각각 호출될 때의 차이점을 설명하세요. 또한, RVO(Return Value Optimization)와 관련된 최적화 기법에 대해 설명하세요.
// RVO
MyClass createObject() {
 return MyClass(); << 일반 생성자 호출
}

MyClass&& createObject() {
 return MyClass(); << return에서 일반 생성자, rvalue로 이동 생성자 호출
}

//NRVO
MyClass createObject() {
 MyClass temp;
 return temp; << 이 부분을 최적화 하면 RVO
}

int main() {
 MyClass obj = createObject();
 std::cout << *(obj.data) << std::endl;
 return 0;
}
```

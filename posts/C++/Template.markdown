---
layout: post
title: "C++ 템플릿"
description:
date: 2024-11-04 15:01:04 +0900
tags : C++
---


## 템플릿
템플릿은 cpp에서 다형성을 위해 지원하는 기능으로,  c#의 제너릭과 유사한 기능을 제공한다. 여기서 주목할 점은 상속과 달리 템플릿은 컴파일 타임에 이를 정의하기에 더 빠르다.


### 객체지향 관점에서의 템플릿 사용 이유

우선 템플릿은 추상과 비슷해보이나, 이는 내부적으로나 적용적인 측면으로나 같지 않다. 다만 템플릿의 특성상 다형성의 성질을 띈다.

- **재사용성**: 템플릿을 사용해 재사용할 수 있습니다. 예를 들어, `std::vector`는 템플릿 클래스로, 어떤 타입의 요소도 저장할 수 있습니다.

- **유연성**: 템플릿을 사용하면 타입에 독립적인 코드를 작성할 수 있습니다. 이는 코드의 유연성을 높여주며, 다양한 타입을 처리할 수 있는 일반화된 함수를 만들 수 있습니다.

- **타입 안전성**: 템플릿을 사용하면 컴파일 타임에 타입 검사를 할 수 있어, 런타임 오류를 줄일 수 있습니다.

- **성능**: 템플릿은 컴파일 타임에 인스턴스화되므로, 런타임 오버헤드가 없습니다. 이는 성능 최적화에 유리합니다. ( c# 제너릭의 경우 컴파일 체크, 런타임에 인스턴스화 된다.)



### 1. 일반 템플릿

일반 템플릿은 다양한 타입에 대해 동작할 수 있는 일반화된 코드를 작성할 때 사용.

```cpp
#include

// 일반 템플릿
template <typename T>
class MyClass {
public:
 void display(T value) {
 std::cout << "General template: " << value << std::endl;
 }
};

int main() {
 MyClass intObj;
 intObj.display(42); // General template: 42

 MyClass doubleObj;
 doubleObj.display(3.14); // General template: 3.14

 return 0;
}
```

---

### 2. 특수화된 템플릿 (명시적 특수화)

특수화된 템플릿은 **특정 타입에 대해 다른 동작** 을 정의하고 싶을 때 사용됩니다.

```cpp
#include

// 일반 템플릿
template <class T>
class MyClass {
public:
 void display(T value) {
 std::cout << "General template: " << value << std::endl;
 }
};

// 특수화된 템플릿
template <>
class MyClass<char> {
public:
 void display(char value) {
 std::cout << "Specialized template for char: " << value << std::endl;
 }
};

int main() {
 MyClass<int> intObj;
 intObj.display(42); // General template: 42

 MyClass<char> charObj;
 charObj.display('A'); // Specialized template for char: A

 return 0;
}
```

---

### 3. 함수 템플릿

함수 템플릿은 함수의 동작을 다양한 타입에 대해 일반화할 때 사용됩니다.

```cpp
#include

// 함수 템플릿
template<typename T>
void display(T value) {
 std::cout << "Function template: " << value << std::endl;
}

int main() {
 display(42); // Function template: 42
 display(3.14); // Function template: 3.14
 display('A'); // Function template: A

 return 0;
}
```


### 4. 인자 2개를 이용한 템플릿 클래스 C++ 코드

인자 2개를 사용하는 템플릿 클래스를 작성할 수 있습니다. 아래는 두 개의 타입 인자를 받는 템플릿 클래스의 예제입니다.

```cpp
#include

template <typename T1,typename T2>
class Pair {
private:
 T1 first;
 T2 second;

public:
 Pair(T1 f, T2 s) : first(f), second(s) {}

 T1 getFirst() const {
 return first;
 }

 T2 getSecond() const {
 return second;
 }

 void print() const {
 std::cout << "First: " << first << ", Second: " << second << std::endl;
 }
};


//템플릿 부분 특수화
template <typename T>
class MyClass<T, int> {
public:
	void print() const {

		std::cout << "Partially specialized template for int" << std::endl;
	}
};

int main() {
 Pair p1(1, 2.5);
 p1.print(); // First: 1, Second: 2.5

 Pair p2("Hello", 'A');
 p2.print(); // First: Hello, Second: A

 return 0;
}
```

- **일반 템플릿**: 다양한 타입에 대해 동작할 수 있는 일반화된 코드를 작성할 때 사용됩니다.

- **특수화된 템플릿 (명시적 특수화)**: 특정 타입에 대해 다른 동작을 정의하고 싶을 때 사용됩니다.

- **함수 템플릿**: 함수의 동작을 다양한 타입에 대해 일반화할 때 사용됩니다.


### 5. CRTP

CRTP (Curiously Recurring Template Pattern)는 C++에서 자주 사용되는 디자인 패턴 중 하나로, 템플릿을 사용하여 상속 계층 구조를 정의할 때 자식 클래스가 부모 클래스의 템플릿 인자로 자신을 전달하는 패턴입니다. 이 패턴은 주로 정적 다형성을 구현하거나 컴파일 타임에 특정 동작을 결정할 때 사용됩니다.

CRTP의 기본 아이디어는 부모 클래스가 자식 클래스의 타입을 템플릿 인자로 받아들이고, 이를 통해 런타임 오버헤드 없이 다형성을 구현할 수 있습니다.

```cpp
#include

// CRTP 기반의 기본 클래스
template
class Base {
public:
 void interface() {
 // Derived 클래스의 구현을 호출
 static_cast(this)->implementation();
 }

 void implementation() {
 std::cout << "Base implementation" << std::endl;
 }
};



// Derived 클래스
class Derived1 : public Base {
public:
 // Derived 클래스에서 구현을 재정의
 void implementation() {
 std::cout << "Derived1 implementation" << std::endl;
 }
};

// 또 다른 Derived 클래스
class Derived2 : public Base {
public:
 // Derived 클래스에서 구현을 재정의
 void implementation() {
 std::cout << "Derived2 implementation" << std::endl;
 }
};

int main() {
 Derived1 d1;
 Derived2 d2;

 d1.interface(); // Derived1 implementation
 d2.interface(); // Derived2 implementation

 return 0;
}
```

이 예제에서 `Base` 클래스는 템플릿 클래스이며, `Derived` 타입을 템플릿 인자로 받습니다. `Base` 클래스는 `interface`라는 멤버 함수를 제공하며, 이 함수는 `Derived` 클래스의 `implementation` 함수를 호출합니다. `Derived1`과 `Derived2` 클래스는 `Base` 클래스를 상속받고, 각각의 `implementation` 함수를 재정의합니다.

`main` 함수에서 `Derived1`과 `Derived2` 객체를 생성하고 `interface` 함수를 호출하면, 각 객체의 `implementation` 함수가 호출됩니다. 이를 통해 런타임 오버헤드 없이 정적 다형성을 구현할 수 있습니다.

CRTP는 다음과 같은 상황에서 유용하게 사용될 수 있습니다:
1. **정적 다형성**: 런타임 오버헤드 없이 다형성을 구현할 때.
2. **코드 재사용**: 공통된 기능을 기본 클래스에 구현하고, 이를 여러 파생 클래스에서 재사용할 때.
3. **컴파일 타임 최적화**: 컴파일 타임에 타입 정보를 활용하여 최적화할 때.


[템플릿 참고](https://ddidding.tistory.com/36)


### 가변길이 템플릿

[가변길이 템플릿](https://modoocode.com/290)


```cpp
#include <iostream>

template <typename T>
void print(T arg) {
  std::cout << arg << std::endl;
}

template <typename T, typename... Types>
void print(T arg, Types... args) {
  std::cout << arg << ", ";
  print(args...);
}

int main() {
  print(1, 3.1, "abc");
  print(1, 2, 3, 4, 5, 6, 7);
}

```

[가변길이 폴드](https://jungwoong.tistory.com/103)
```cpp
template<typename First,typename... T>
void print(First f, T... t) {

	// 파라미터 팩 확장을 통해 처리한다.
	int a = (...+t); // fold expression need c++ 17
	int a2[] = { t... };
}

// 명시적 람다 탬플릿  need c++ 20
auto lambda = []<typename T>(T x, T y) {
	return x + y;
}(1, 2);
// 명시적 람다 탬플릿 폴드 방식 need c++ 20
auto lambda2 = []<typename... T>(T...x) {
	return x + y;
}(1, 2,3,4,5,6);
int main(){

	cout << []<typename... T>(T x, T y) {
		return x + y;
	}(1, 2);

	print(1, 2, 3, 4, 5);
}
```
재귀적으로 호출된다.
참 예쁘다.

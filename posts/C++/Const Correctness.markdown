---
layout: post
title: "상수 교정자"
description:
date: 2024-11-04 15:01:04 +0900
tags : C++
---

## const correctness (상수 교정자)

```cpp
const class ConstTest  // class에 붙은 const는 컴파일에 무시된다.
{
public:
	int global;
	// 함수의 const는 class 의 멤버변수 변경을 금지한다.

	ConstTest(ConstTest* other) {
		cout << "const 복사생성자";
	}
	void test()const { //인스턴스해도 const 함수가 멤버함수를 불러온
		global = a;
		a++; // const
		int local = a;
		cout << "const";
	}

	void test() {
		cout << "normal";
	}

	int value2 = 5;
	const int& ConstReturnRef() {
		return value2;
	}
	int& NoneConstReturnRef() {
		return value2;
	}
private:
};


#define NUMBER_TEST 5;

int main() {
	// 상수 교정자
	volatile const int a = 5; // 데이터 영역 read only로는 안들어감.
	int b = 6;
	int c = 6;

	const int* ap = &a; // 포인터 ap는 변경할 수 없는 개체 5를 가리킨다.
	int const* ap;
	//const 기준으로 결정

	int* const ap2 = &b; // 오른쪽에 붙는 const는 가리키는 대상을 수정 금지한다.
	int* const ap2; // const 기준 포인터 변경 금지

	const int* const ap3 = &b;
	int const * const ap3; // 왼쪽부터 가리키는 값 변경 금지 , * const 포인터 변경 금지

	int &const ref1 = b; // 변경 불가능한 대상을 가리키는 레퍼런스
	const int&  ref2 = b; //

	*ap = 5; // 변경 불가능한 개체 a를 수정하려 하는 *ap. a는 변경 불가능한 변수이며, ap는 이를 가리키는 const int 포인터이다.
	ap = &b; // 가리킬 대상의 변경은 자유롭다.

	*ap2 = 5; // 가리키는 대상의 수정이 자유롭다.
	ap2 = &c; // * const는 포인터의 변경을 금지한다.

	ap3 = &c; // 변경 불가능한 대상을 가리키는 포인터
	ap3 = new int(6); // 포인터의 변경을 금지

	ref1 = c;
	ref1++;


	ref2 = c;
	ref2++;


	constexpr int constexpr2 = NUMBER_TEST; //컴파일 상수로 런타임 변경을 금지함.

	const volatile constexpr int dsfdsffds = 5;

	ConstTest Ctest;
	const ConstTest Ctest2;

	Ctest.test();
	Ctest2.test(); // const 함수의 차이는 const 개체가 멤버 const 함수를 호출한다는 것이다.


	Ctest.ConstReturnRef()++; // 반환값 변경을 금지함
	Ctest.NoneConstReturnRef()++; // 반환값 변경이 허용된

	return 0;
}
```

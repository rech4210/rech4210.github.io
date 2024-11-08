---
layout: post
title: "구조체 패딩"
description:
date: 2024-05-05 12:17:04 +0900
tags : ComputerScience
---

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/9d6c464e-48d9-4a7f-b3df-d876ff7b6417)

struct, class 내부에 멤버들이 있을 때, 이 멤버들은 어떻게 메모리 공간을 차지하게 될까?

## struct Padding  
구조체를 정의할 때 컴파일러가 멤버 사이에 추가 byte를 삽입해 메모리 내 데이터의 정렬을 보장하는 메모리 액세스 및 정렬 최적화 기법이다.

프로세서는 한번에 정해진 만큼의 일을 할수있는데, 86, 64는 각각 4byte, 8byte의 1사이클을 돌아 메모리에 접근한다.

#### why 64x 86x architecture paddin is same?
> 64, 86 아키텍처 모두 데이터 정렬 요구사항이 비슷하며, 아키텍처 일관성 보장을 위해 패딩 결과가 비슷하게 나온다.

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/23021c62-8b3e-4f80-bdd2-7bb43f3af32e)

위와같이 정의된다고 했을 때, 구조체 패킹에 의해 결과값은 맨 위의 사진과 같이 컴파일러가 추가 byte를 삽입하고 c 변수의 공간을 마지막 4byte에 정의해줄 것이다.

만약 구조체 내의 멤버 변수가 순서를 바뀐 채 정의된다면 어떻게 될까?

### variable

```cpp
#include<iostream>
using namespace std;
struct abc {
	char a; // 1
	char b; // 1
	int c; // 4
};

struct bcd {
	char a; // 1
	int c; // 4
	char b; // 1
};

int main() {
	abc d;
	bcd e;
	printf("%d\n", sizeof(d));
	printf("%d\n", sizeof(e));
}
```

### result
```
8
12
```
순서만 바뀌었을 뿐인데,  메모리 공간이 늘어났다
<br>



![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/d1a3eaf3-5c5a-48ec-9373-69c9703c2802)

위의 그림을 보면 char가 정의되고 빈 공간만큼 구조체 패킹을 통해 나머지 4byte 공간에 int가 정의된다. 마지막 순서에 따라 char가 정의되며 총 12byte 공간을 가지게 된다 (4바이트 손해)

결과적으로 순서가 바뀌게 된다면 구조체 패킹에 의해 총 12byte의 공간을 차지하게 된다.
<br>

---


### method
구조체 내부의 함수는 코드 영역에 저장되므로 함수 자체의 크기는 구조체에 영향을 주지 않는다.
```cpp
struct virtulMethod {
	char a; // 1
	char b; // 1
	int c; // 4
	virtual void run() {}; // => ?
};
```

위는 구조체 내부에 가상함수가 선언되어 있다.
출력해보면 몇 byte의 공간을 차지하고 있을까?

### result
```
16
```

왜 8byte에서 16byte 공간이 차지하게 된 걸까?

이를 이해하려면 vtable에 대한 이해가 있어야 한다.


![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/6d2415ab-d9d0-45a9-b996-049e8eb2157e)

간단히 가상함수에 접근하려면 이 가상함수를 구현하는 개체들이 컴파일 타임에 가상함수의 정보를 가지는 vtable 포인터를 가지게 된다. 이 포인터는 8byte의 크기로 구조체 가장 처음 메모리에 적재되기 때문이다.

[vtable이란? ]()

---
layout: post
title: "IL2CPP, Mono, AOT, JIT"
date: 2023-10-06 14:36:41 +0900
description:
tags : Unity+
---
https://www.youtube.com/watch?v=-9X965jXrn8

유니티의 인터페이스 부분은 C#이다.

C#은 MS의 .net 프레임 워크 기반으로 개발되었다.
그런데 어떻게 다양한 플랫폼에 대응되도록 빌드를 수행시킬 수 있을까?

유니티를 mono에 빌드시키기 위해,  c#을 IL이라는 중간언어로 변환한다 (.net 어셈블리)

Mono가 이를 런타임 상에서 여러 플랫폼에 대응되도록 JIT (Just in time) 컴파일을 수행한다.

## IL2CPP?

IL2CPP는 C#에서 만든 IL언어를 C++로 변환하는 것이다.
C++로 변환시의 장점
1. 다양한 플랫폼에 빌드를 할 수 있다.
2. 좋은 성능을 가져갈 수 있다.

![image](https://user-images.githubusercontent.com/65288322/266649859-9782dec1-6d9d-43a9-a449-f4f0c58895c9.png)

Generic Sharing


C++의 템플릿은 컴파일시에 사용된 제너릭 타입을 수동으로 정의해준다. (사용할수록 정의된 내용이 늘어남)

### C#의 제너릭과 C++의 템플릿의 차이
Call by value vs Call by reference

c++은 다양한 방법으로 데이터를 넘겨줄 수 있다. (value, pointer, reference) 반면, c#은 자료타입에 따라 value, reference로 나뉜다.

제너릭 사용시 reference type은 공유 코드를 사용하여 제너릭 정의를 최소화 하지만 value의 경우는 코드를 확장하게 되므로 시간이 오래 걸리게 된다.

결국 데이터 타입과 빌드 방식 (AOT, JIT)에 따라 c++로의 변환에 매우 긴 시간이 걸리게 된다. (특히 generic sharing의 경우 매우 많은 제너릭 타입을 c++로 변환시 정의해야 한다)

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/312e9d2e-b525-4d35-90c2-8eb468bce832)
유니티에서는 ILL2CPP 빌드 방식에 2가지 방법을 제공한다.
1. Value 타입 제너릭 코드를 확장시킨다. (빌드 용량 증가 속도 감소)
2. value 타입을 rapping하여 공유코드를 사용한다 (빌드 속도는 상승하나, 런타임 성능이 떨어진다)

## Job System
데이터 기반 프로그래밍

C#에서는 멀티스레드 스레드세이프 하지않다. 특히 Unity API를 멀티스레드 환경에서 사용시 교착상태에 빠지거나 프로그램이 깨질 우려가 있는데, 이를 보완하기위한 시스템이다.



## Burst
IL을 바로 어셈블리로 바꿔주는 컴파일러로 SIMD에 최적화된 코드를 만들어줌.

#### SIMD  (single instruction multiple data)
> 하나의 연산으로 여러 데이터를 처리하는 아키텍처 (Neon SSE)
>  ex ) vector 타입을 한번에 묶어 연산처리
>
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/c66d4eb6-bccf-4dc0-95f7-df85af50d78a)

LLVM(컴파일 단계의 VM)이 IL언어를 IR로 변환하여 몇번의 사이클을 거치고 SIMD에 최적화된 어셈블리 코드를 생성해낸다. (Job 시스템과 잘 어울리는 이유이다)

다만, Burst의 효과를 잘 받기 위해선 NativeArray를 사용해야하고 메모리 관리를 직접 제어해주어야 한다. 마찬가지로 Delegate를 사용하지 못하고 funtion pointer를 사용하는 c++에 가까운 코딩을 해야한다.

그렇기에 Burst의 이점을 활용할 수 있는 부분에만 [BurstCompile] 어트리뷰트를 선언해 컴파일 대상을 지정할 수 있다.

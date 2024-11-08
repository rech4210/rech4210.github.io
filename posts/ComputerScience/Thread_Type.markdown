---
layout: post
title: "Thread의 종류"
description:
date: 2024-11-04 15:01:04 +0900
tags : ComputerScience
---
![image](https://github.com/user-attachments/assets/982ff559-2d32-4811-947d-ad485f4521fd)


## H/W Thread
![image](https://github.com/user-attachments/assets/2f12d67d-4b35-4b81-91fb-6fd22dabf785)
CPU core가 작업을 한 후 memory 작업을 하게될때 쉬게되는 구간이 존재하게 된다. 메모리에 접근하는 동안 독립적인 작업을 하기위해 제안되었다.

> OS 관점에서 가상의 코어로 인식된다 (멀티코어 -> 4개의 H/W Thread -> 4개의 논리 코어로 인식)

#### 하이퍼 스레딩
이처럼 H/W Thread의 기술을 하이퍼 스레딩이라 한다.

## OS Thread
CPU에서 실제로 실행되는 단위, CPU 스케쥴링의 단위
- OS 스레드의 컨텍스트 스위칭은 커널이 개입하므로 비용이 발생한다. (User -> Kernel)
- 사용자 코드와 커널 코드 모두 OS 스레드에서 실행된다.
	- OS Thread -> System call -> Kernel -> User

아래와 같이 불리기도 함
- 네이티브 스레드
- 커널 스레드*
- 커널-레벨 스레드
- OS-레벨 스레드

## User Thread
스레드 개념을 프로그래밍 레벨에서 추상화 한 것.

![image](https://github.com/user-attachments/assets/d19dbaa2-dcfb-4b1e-9531-798ff8f3126a)

즉 User mode에서의 User Thread를 OS Thread와 연결시킨 것이다.

### One-to-One model

- User와 Kernel 스레드가 1:1 매칭되는 상태 (스레드 관리를 OS에 위임)
- 멀티코어를 잘 활용할 수 있음
- 스레드간의 독립적임 (race condition 가능성)

### Many-to-One model

- User와 Kernel 스레드가 n:1 매칭되는 상태
- 컨텍스트 스위칭이 빠르다 (User 모드에서 컨텍스트 스위칭이 일어나기 때문)
- OS 레벨에서의 race condition 가능성이 적음 (User 레벨에서 가능성 존재)
- 멀티코어 활용을 못함
- 스레드간 연관적임 (block IO 경우 모든 스레드 블락 -> non block IO로 해결)

### Many-to-Many model (Go)
- one / many-one의 장점을 모아 만듬
- 구현이 어려움

### User thread
OS와는 독립적으로 유저 레벨에서 스케줄링 되는 스레드 (기술문서등에선 User단에서 작동하는 스레드만을 지칭하는 경우가 있음)

### Green thread
OS와는 독립적으로 유저 레벨에서 스케줄링되는 스레드


## Kernel thread
OS 커널의 역할을 수행하는 스레드 (커널의 역할을 하는 스레드와 커널단에서 작동하는 스레드, 이 두 가지 혼동에 유의)

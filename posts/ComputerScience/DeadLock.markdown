---
layout: post
title: "교착상태 DeadLock"
description:
date: 2024-04-15 19:25:59 +0900
tags : ComputerScience
---

# 교착상태

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/686963da-5d5a-40ce-9c8a-7b5319567e71)


### 교착상태란?
일어나지 않을 상황을 기다리며 진행이 멈춰버린 상태


#### 자원 할당 그래프
어떤 프로세스가 자원을 할당 받아 사용하는지 / 대기하는지 확인 가능
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/47a56ff1-1517-4d35-bf79-66ddf8f94083)

교착 상태가 일어난 자원 할당 그래프는 원형의 형태를 띈다.

#### 교착 상태의 발생 조건
1. 상호 배제
	: 한 프로세스가 사용하는 자원에 대한 접근 제한
2. 점유와 대기
	: 자원을 할당받은 상태에서 다른 자원을 할당 대기받는 상태
3. 비선점
	: 어떤 프로세스도 다른 프로세스의 자원을 강제로 뺏지 못하는 상태
4. 원형 대기
	: 프로세스들이 원의 형태로 자원을 대기하는 상태

위 4가지를 모두 만족하면 교착 상태가 발생할 수 있음.

#### 교착 상태 해결 : 예방, 회피, 검출 후 회복

1. 예방
위의 교착 상태 조건 일부를 없애 교착 상태를 사전에 예방한다
	- 점유와 대기 : 특정 프로세스에 자원을 모두 할당 또는 할당x => 자원의 활용율이 낮아진다.
	- 비선점 : 자원을 선점형으로 관리한다 =>  모든 자원을 선점형으로 사용할 수 있는것이 아니다 (프린터)
	- 원형 대기 : 자원별 우선순위를 부여한다  => 자원마다 호출이 다르므로 우선순위별 활용율이 차이난다.
데드락 프로파일러를 사용하여 미리 검증과정을 거친다.

2. 회피
	교착 상태를 무분별한 자원 할당으로 인한 발생이라 간주한다 (교착 상태가 발생하지 않을 만큼만 자원 배분)

- 안전 순서열 : 교착 상태 없이 안전하게 모든 프로세스들이 자원을 할당할 수 있는 순서
	- 안전 상태 : 교착 상태 없이 모든 프로세스가 자원을 할당 받고 종료될 수 있는 상태
		> 안전 순서열이 있는 상태이다

	- 불안정 상태 : 교착 상태가 발생할 수 있는 상태
		> 안전 순서열이 없는 상태이다

##### 안전 순서열을 유지하려면?
안전 상태에서 안전 상태로 움직이는 경우에만 자원 할당 허용

프로세스들에게 자원을 할당해줬을 때 할당받은 프로세스가 종료되고 나머지 프로세스에게 요구치 만큼의 자원을 할당해줄 수 있다면 안정 순서열이 있다고 할 수 있다.
ex) 은행원 알고리즘
<br>

3. 교착 상태 검출 후 회복
교착 상태의 발생을 인정하고 사후에 조치
- 프로세스가 자원을 요구하면 일단 할당, 교착 상태가 검출되면 회복
- 선점을 통한 회복
	- 교착 상태가 해결될 때까지 한 프로세스씩 자원을 몰아줌
- 프로세스 강제 종료를 통한 회복
	- 교착 상태에 놓인 프로세스 모두 강제 종료 (작업 내역)
	- 교착 상태가 해결될 때까지 한 프로세스씩 강제 종료( 오버헤드 발생)

4. 교착 상태 무시
희귀하게 발생하므로 그냥 무시함.

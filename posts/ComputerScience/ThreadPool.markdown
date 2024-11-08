---
layout: post
title: "Thread Pool"
description:
date: 2024-11-04 15:01:04 +0900
tags : ComputerScience
---

### Thread Per request Model
: 서버 사이드를 예로 들어 들어오는 requst 마다 하나의 thread가 담당하게 되는 1:1 관계이다.

> 만약 Thread Per request Model 방식이 요청마다 스레드를 새로 만들어 처리한다면 어떻게 될까?

1. 스레드의 생성을 위해서는 커널의 자원을 사용하게된다. 커널 레벨 접근에 의해 요청 처리가 오래 걸리게된다.
2. 스레드 수가 증가함에 따라 정해진 CPU 스케쥴링 정책에 따라 컨텍스트 스위칭이 더 자주 발생하게되며, CPU 오버헤드가 증가하게된다. (응답 불가 상태 가능성)
3. 스레드가 계속 생성됨에 따라 메모리가 더욱 고갈됨.

### Thread Pool
![image](https://github.com/user-attachments/assets/cb81d995-4c40-4000-9edd-36c5984eb3a9)

미리 스레드를 여러 개 만들어놓고 재사용한다
여러개의 작업을 동시에 처리하는 경우 (requst 모델, subtask 처리, 비동기 처리)

1. 스레드 생성 시간 절약이 가능
2. 제한된 개수의 스레드 운용에 따라 무제한 생성을 방지

### Thread pool 사용 팁

1. 스레드 풀에 몇 개의 스레드를 만들어 두는게 적절한가?
- CPU bound task 라면 코어의 개수 + (코어 개수에 영향을 받으므로 스레드 수가 제한됨)
-  I/O bound Task 라면 코어 개수보다 배수로 가능하나, 정확한 수치를 테스트 해보아야 함.


![image](https://github.com/user-attachments/assets/a05e107f-d4e7-4dd0-a5bd-80d64ec8cd6a)
1 . 스레드 풀에서 실행될 task 개수에 제한이 없다면, thread requst queue 사이즈를 봐야한다.
-  INT_MAX 값의 capacity를 가지면 거의 무제한의 requst 요청을 사이즈에 담게되고 메모리 고갈을 야기하게된다.

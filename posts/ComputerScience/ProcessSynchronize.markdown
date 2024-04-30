---
layout: post
title: "Process 동기화 방법"
description:
date: 2024-04-25 17:25:04 +0900
tags : ComputerScience
---

# 프로세스 동기화
동시다발적으로 실행되는 프로세스 (스레드) 들은 서로 협력하며 영향을 주고 받는데 이 과정에서 자원을 일관성있게 보장해야 한다.

## 동기화
프로세스들의 수행 시기를 맞추는 것으로 실행의 문맥을 갖는 모든 대상은 동기화 대상이기에 스레드 또한 동기화 대상이 된다.

### 모드에 따른 동기화 방법
유저 모드
- 크리티컬 섹션
- 스핀락 (cotext switching 부하 제거를 위해 무한 lock 획득 시도 반복)
- SRW lock

커널 모드
- 뮤텍스
- 세마포어
- 이름있는 뮤텍스
- 이벤트

---

### 실행 순서 제어
 - 프로세스를 올바른 순서대로 실행하기

#### reader writer problem
writer : 파일에 값을 저장하는 프로세스
reader : 파일에 저장된 값을 읽어들이는 프로세스
reader 프로세스는 읽어들이려는 파일 안에 값이 존재한다는 조건이 만족되어야만 실행이 가능하다 그렇기에 reader, writer 프로세스간 실행의 순서가 명백해야 한다.


### 상호 배제
- 동시에 접근해선 안되는 자원에 하나의 프로세스만 접근하게 하기

#### bank account problem (race condition)
한 번에 하나의 프로세스만 접근 가능한 크리티컬 섹션에 동시 사용 가능성이 발생해 값이 변경될 수 있는 상태.

만약 프로세스가 점유중인 자원을 다른 프로세스가 동시에 사용한다면 어떻게 될까?

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/f1b43026-049d-4a9d-a6b3-43f77fe7113b)

> 위 내용을 확실히 이해하려면 레지스터에 대한 이해가 필요할것이다.

위는 유명한 Producer & Customer 문제라고 한다. 위에서 알 수 있듯이 동일한 자원을 동기화 없이 관리하게 되면 값을 덮어쓰거나 context switching이 일어남에 따라 값을 예측할 수 없다. 마찬가지로 segmentation fault를 일으킬 가능성이 있다.

---

### shared resource / critical section
shared resource (공유 자원)
: 공유 자원으로 여러 프로세스 혹은 스레드가 공유하는 자원 (전역변수, 파일, IO 장치, 기억장치)

<br>

critical section (임계 구역)
 : 동시에 실행하면 문제가 발생하는 자원에 접근하는 **코드 영역** (위 예시의 총합 변수)
임계 구역에 진입하고자 하면 진입한 프로세스 이외에는 대기해야 한다.

<br>


##### critical section (유저 모드 동기화)
윈도우에서 유저 모드의 동기화 방법 중 하나이다.

뮤텍스는 커널에 대한 시스템 호출을 필요로 하므로 오버헤드가 생기기에 크리티컬 섹션이 더 빠르다. 크리티컬 섹션은 하나의 프로세스 내 쓰레드 간에서만 사용할 수 있지만 race condition이 발생할 경우 커널 모드로 전환되며, 비경합의 경우 훨씬 빠르다.

##### critical section 예제

```cpp
CRITICAL_SECTION cs; // 크리티컬 섹션 개체 생성
int glober_number = 0; //cs 영역에 해당될 변수

void threadTest(thread* id, string name) {
	chrono::system_clock::time_point start = chrono::system_clock::now();
	for (int i = 0; i < 5; ++i) {
		cout << "name : " << name << " id : "
    << id->get_id() << " count :  " << i << "번 " << endl;
		}
	chrono::duration<double> duration = chrono::system_clock::now() - start;
	cout << " 루프 시간 : " << duration.count() << "s name : " << name << endl;

	EnterCriticalSection(&cs);		//Critical Section 객체 시작지점 설정
	cout << " 글로벌 넘버 증가 " << ++glober_number << " name : "<< name << endl;
	//스레드가 크리티컬 섹션에 접근할때 선점된 개체 이외엔 glober number에 접근하지 못한다.
	LeaveCriticalSection(&cs);		//Critical Section 객체 끝지점 설정

	chrono::duration<double> duration2 = chrono::system_clock::now() - start;
	cout << " CS 완료 시간 : " << duration2.count() <<  "s name : " << name << endl;
}

int main()
{
	InitializeCriticalSection(&cs); //Critical Section 객체 초기화
	thread th1;
	th1 = thread(threadTest, &th1,"th1");
	thread th2;
	th2 = thread(threadTest, &th2, "th2");
	thread th3;
	th3 = thread(threadTest, &th3, "th3");

	// 스레드의 실행 완료 대기
	th1.join();
	th2.join();
	th3.join();

	DeleteCriticalSection(&cs);		//종료시점 Critical Section 객체 초기화
	return 0;
}

```

<br>

```cpp
InitializeCriticalSection(&cs);   //Critical Section 객체 초기화
EnterCriticalSection(&cs);		//Critical Section 객체 시작지점 설정

// variable

LeaveCriticalSection(&cs);		//Critical Section 객체 끝지점 설정
DeleteCriticalSection(&cs);		//종료시점 Critical Section 객체 초기화
```

<br>

- InitializeCriticalSection : 객체가 생성되면 lock count, 소유자 스레드, recursion 횟수 등 제어 변수와 내부 데이터에 메모리가 할당된다.   

초기화 이전
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/ffa65144-8d07-4e55-8d29-d36cf0a697ff)


초기화 이후
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/9dfa938d-55bb-4abd-823a-b152a6b3c34e)

<br>

- EnterCriticalSection: 스레드가 임계 영역에 진입하려 할때 해당 함수를 호출하면 접근 권한을 획득하게 된다. 이미 다른 스레드가 권한을 획득중이라면 호출 스레드가 차단된다. 내부적으로 정해진 횟수를 무한루프 돌며 접근권한을 얻고자 하는데, 이는 커널모드의 비싼 context switching 작업을 방지하기 위함이다.

<br>

- LeaveCriticalSection: 스레드가 작업을 완료하면 잠금을 해제한다.

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/780a155c-394b-4aab-b8a0-54a0c763c283)

<br>

- DeleteCriticalSection: 크리티컬 섹션에 대한 메모리를 초기화한다.

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/607112a3-2b52-46c7-85c8-9f84c8b3c2b3)


<br>

예제 1

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/eefd7d20-e5ad-4d04-8824-6de83216f2c1)
초기에 스레드 1,3이 실행되고 가장먼저 루프 완료한 th3가 임계영역에 대한 접근권한을 가진다. 만약 나머지 스레드들이 루프 작업을 끝냈다면 th3의 임계영역 작업을 완료할때까지 대기한다. 이때 CS영역에 대한 접근을 위해 무한 loop를 돌다 시간이 길어지면 context switching 작업을 위해 커널모드로 진입요청을 보낸다.


<br>

예제 2

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/fa01fcef-bf45-4248-a745-155f97f588b6)



##### 인터락
멀티스레드 환경에서 안전하게 변수값을 조작하는 함수를 인터락이라 한다.
++와 같은 증감연산자는 단순 한줄로 수행되지만, 어셈블리단에서는 레지스터의 값을 가져와 변경시켜주는 과정을 거친다. 이 과정에서 스위칭이 발생할시 bank account problem와 같은 문제가 발생하게 되는것이다.

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/d607854f-6928-43e4-99fc-b298e66d147c)
~~그럼 임계구역에 나올때 무슨 인터럽트를 주는걸까?~~

임계 구역에 동시에 접근하면 자원의 일관성이 깨질 수 있는데, 공유 자원을 두고 동시 사용할 때 발생하는 상황을 **race condition** 이라 한다.
즉, 경쟁 상태 중 저급 언어로 컴파일 등의 과정을 거치다 context switching이 발생하는 등 해당 자원에 여러 프로세스가 사용 시도 시 자원의 일관성이 깨질 수 있다.

### 동기화의 원칙

1. 실행 순서 동기화

2. 상호 배제 동기화
- 상호 배제 (mutual exclusion)
	- 한 프로세스가 임계 구역에 진입했다면 다른 프로세스의 접근은 배제된다.
- 진행 (progress)
	- 임계 구역에 어떤 프로세스도 진입하지 않았다면 프로세스의 진입을 허용한다.
- 유한 대기 (bounded waiting)
	- 한 프로세스가 임계 구역에 진입하고자 한다면 언젠가는 들어올 수 있어야 한다 (무한 대기 금지)


## 동기화 기법

### 뮤텍스 락 (상호 배제 mutual exclusion)
상호 배제를 위한 동기화 도구로 임계 영역 진입을 위한 키의 역할을 한다.

자물쇠 역할: 프로세스들이 공유하는 전역 변수 lock
임계 구역을 잠그는 함수 : acquire 함수, 임계구역 진입전까지 진입 시도 호출
임계 구역의 잠금을 해제하는 함수 : release 함수, 임계 구역 작업이 끝나고 호출

#### busy waiting
위에서 설명한대로 acquire 함수를 사용하는 프로세스가 임계 구역 진입을 위해 반복적으로 접근 시도하는 경우.
이럴 경우 CPU 사이클에 영향을 주게 된다.

### 세마포어
공유 자원이 여러 개 있는 경우에도 적용 가능
임계 구역 앞에서 멈춤 신호를 받으면 대기하다 진입 신호를 받고 진입
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/29d9b236-c122-4272-b04f-98d94c6e8203)

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/937c891c-d8b2-442d-a9ff-ac6bf65a0be1)

- 임계 구역에 진입할 수 있는 프로세스의 개수를 담은 전역변수 S
- 임계 구역에 진입 / 대기할지 알려주는 wait 함수
- 임계 구역 앞에서 기다리는 프로세스에게 진입 신호를 주는 signal 함수

#### Busy waiting을 해결하는 방법
잦은 진입확인은 CPU의 사이클을 낭비시키는데, 이를 해결하기위해
- 사용할 자원이 없을때 프로세스를 대기 상태로 만든다(해당 프로세스의 PCB를 대기 큐에 삽입)
- 사용할 자원이 생길 경우 대기 큐의 프로세스를 준비 상태로 만듬. (PCB를 대기 큐에서 꺼내 준비 큐에 삽입)

이처럼 잦은 cpu 사이클을 방지하기위해 프로세스를 대기 큐 / 준비 큐에 삽입해 이용한다.

#### 세마포어의 실행 순서 동기화
- 세마포어 변수 0으로 설정
- 먼저 실행할 프로세스 뒤에 signal 함수
- 다음에 실행할 프로세스 앞에 wait 함수를 붙인다.

wait 가 붙은 프로세스는 변수가 0 으로 설정됨에 따라 먼저 실행되더라도 대기 상태에 빠지게 되며 signal 함수가 붙은 프로세스가 우선적으로 실행된다.

### 모니터
사용자 친화적 동기화 방법
세마포어의 잘못된 임계구역 접근 지정으로 상호 배제 조건이 어긋나는 경우를 보완하기 위해 사용한다.

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/1898a491-9857-43db-b878-dfbef649dee6)

상호 배제를 위한 동기화
- 인터페이스를 위한 큐 (임계 구역에 해당하는 자원을 보호하기위해 인터페이스만 제공한다 마치 시스템콜과 같음)
- 공유 자원에 접근하고자 하는 프로세스를 큐에 삽임
- 큐에 삽입된 순서대로 공유 자원 이용

실행 순서를 위한 동기화
- 조건 변수 이용
	- 프로세스나 스레드의 실행 순서를 제어하기 위해 사용하는 특별한 변수
	- 조건변수 큐에 넣어 관리
	- 조건변수.wait() / 조건변수.signal()

모니터는 상호 배제 뿐 아니라 실행 순서 동기화도 제공하는데, 모니터 안에는 하나의 프로세스만 가지는것을 원칙으로 한다.
모니터의 실행 순서 동기화는 아래와 같다.

1. wait() 함수를 호출한 프로세스는 signal() 함수를 호출한 프로세스가 종료된 후에 수행을 재개한다.
2. signal()을 호출한 프로세스의 실행을 **일시 중단**하고 자신이 실행된 뒤 다시 signal()을 호출한 프로세스 수행을 재개.

정리하자면
- 뮤텍스는 임계 영역에 해당하는 영역을 소유하는 개념이다.
- 세마포어는 뮤텍스를 포함하지만 그 반대는 될 수 없다.
- 세마포어는 동기화 대상이 여러 개일 때 사용하며 내부적 기능은 동일하다. (뮤텍스는 하나)
- 모니터는 동일 프로세스 내의 다른 스레드 간 동기화 사용에 해당된다.

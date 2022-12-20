---
layout: post
title: "클래스 상속과 다형성"
date: 2022-11-22 20:08:04 +0900
tags : Unity
---

![Character, Player](https://user-images.githubusercontent.com/65288322/202504527-e07daa39-32dc-4b3d-8dc1-d9a276bcb1c3.png)

객체지향, 상속 등등 뭐라 말은 많지만 이게 사실 실제로 적용해보려면 너무 머리가 아팠다.<br>
내가 해맸던 이유와 상속에서 헷갈렸던점을 정리하고자 한다.<br>

오늘 아주 오래 삽질하며 알게 된 결과.<br>

1. monobehaviour의 유무 차이
1. base 키워드 사용방법
   2-1 override, virtual
1. Delegate는 상속이 되는가?
   3-1. Delegate부착시 상속받은 서브클래스가 instance로 받음.

<br>
설명하기에 앞서 내부 구조에 대해 간략히 설명한다.

![states](https://user-images.githubusercontent.com/65288322/202505775-547a33b6-be06-4da2-a7fa-c192bbe45f96.png)

위 내용은 States 스크립트에 move, shot, character 등등의 인스턴스를 만들고 각각의 인스턴스에 접근해 States의 메소드인 ChangeEnum, ReturnEnum을 델리게이트에 부착시킨 내용이다.<br>

왜 굳이 이렇게 했느냐면
<br>

각 스크립트에서 메소드가 실행될 때, States로 연결되는 커플링을 최대한 줄이고 enum 타입을 통해서 메소드 발동시 enum 값을 쉽게 바꾸고, 바뀐 enum 값으로 애니메이션 setBool 처리를 하기 위함이였다.

여기서 플레이어와 AI의 다형성이 필요해졌는데, 이때 부모 (Character)의 상속을 받은 Player,AI는 같은 States 기능을 공유한다.
<br>

### 1. monobehaviour는?<br>
상속에서 monobehaviour의 유무 차이는 뭘까?
기본적으로 유니티의 모든 개체는 mobonehaviour를 상속받아야 컴포넌트로 부착이 가능하다.
그말은 즉슨.

<br>
![image](https://user-images.githubusercontent.com/65288322/202512293-aa09ca39-8c79-4031-9c98-e30f73538f8f.png)
<br>
이렇게 상속을 받는다 쳤을때,  rabbit, cat, fox , russian blue 등등 모두 컴포넌트로써 부착이 가능하다는것.


<br>

마찬가지로 컴포넌트의 역할이기에 동적으로 생성하면 안된다.<br>
만약 동적으로 개체를 생성할시 유니티가 경고한다. mobonehaviour를 상속받는 개체를 동적으로 정의하려 한다고.

![image](https://user-images.githubusercontent.com/65288322/202512317-98c543ae-7628-4bb9-82af-f33b48e28fd0.png)


반대로 mobonehaviour를 상속받지 않을 경우, 개체는 부착이 불가능하며 접근이 불가하다.<br>
사용하는 방법은 알다싶이 인터페이스를 담당할 스크립트 MakeAnimal.cs 가 있다고 치면

Cat _mycat = new Cat();   //기본 생성자를 사용한다고 가정한다.<br>
russian_blue_bluecat= new russian_blue(); //자식 개체 생성<br>
Cat _blue_bluecat= new russian_blue(); // 자식을 부모로 업캐스팅<br>

하여 생성 및 정의 가능한것이다.

### 2. Base<br>
base의 사용법이다.

![Character, Player](https://user-images.githubusercontent.com/65288322/202504527-e07daa39-32dc-4b3d-8dc1-d9a276bcb1c3.png)<br>

이 사진을 보면 Player.cs 에서는 Die 메소드를 base.Die()로 쓰고있다.<br>
이 뜻은 부모 개체에 있는 Die 라는 메소드를 자식 스크립트에서 base 키워드로 실행하겠다는 것.<br>
다음으로 생성자를 base로 사용할 수 도 있다.<br>


![image](https://user-images.githubusercontent.com/65288322/202511317-580b94e8-8857-4155-94ea-a560920d05af.png)


이렇게 말이다.
생성자 실행 순서는.
1. 부모의 생성자 실행
3. 자식의 생성자 실행
4. 자식의 종료자 실행
5. 부모의 종료자 실행
이 순서대로 흘러간다.

그 다음으로는 override와 virtual이다.
이건 꽤나 헷갈렸다.

간단히 얘기하자면 virtual은 부모단에서 override는 부모의 virtual로 지정된 메소드나 프로퍼티등을 내 입맛에 맞게 내용을 바꾸는 것이다.

![image](https://user-images.githubusercontent.com/65288322/202512835-e21010d4-79ec-47b3-bb7d-4dda2db95277.png)

![image](https://user-images.githubusercontent.com/65288322/202512929-8367db6f-a82b-4fb5-8021-c67377d08e64.png)

위의 사진처럼 virtual로 선언된 getDamage를 Player 스크립트에서 override로 선언하고 몸통 부분을 바꾼것과 같다.

<br>
이렇게 되면 자식에서 getDamaged() 실행 시 부모의 메소드를 사용하는것이 아닌 자식 별개의 메소드로 사용된다.

<br>
물론 부모의 getDamage를 사용하고 싶다면 base.getDamage를 쓰면 된다.
유용하게 쓰자.

<br>
세번째
과연 델리게이트 또한 상속이 될까?
두번째로 상속받은 각 개체는 states의 인스턴스로 인식되어 delegate가 부착되어 제대로 실행하는가였다.

<br>
해본 결과.<br>
Player를 부착시키고 게임 실행시 정상적으로 작동했다.<br>

이 말의 의미는 States 클래스의 Charater 인스턴스를 가져오는 과정이 Player 스크립트가 업캐스팅 되면서 처리되었다는 뜻일것이다. 결과적으로 부모의 delegate를 사용하는것이 가능했다.


어쩌면 당연한 얘기겠지만, 삽질하기 전까진 이해가 잘 안되었다.<br>

<br>

### 인터페이스를 상속받는 인터페이스 만들기<br>
+ 델리게이트도 인터페이스로부터 상속받을 수 있을까?<br>
옵저버 패턴으로 변경시켜보기<br>


코드 줄이기 결과<br>
이렇게 한 이유<br>
-> 구현함에 있어서 States 클래스가 옵저버에 모든 상태 변화 메세지를 뿌려주고 각 클래스는 독자적인 행동을 하는 Notify() 함수를 만들고 싶었다.<br>

전<br>
![states](https://user-images.githubusercontent.com/65288322/202639955-b654867d-3db4-40d7-9f66-3ff802198b51.png)<br>
states.cs

![move](https://user-images.githubusercontent.com/65288322/202640140-20019836-515b-4771-85c0-8c8d57acfc91.png)<br>
move.cs

![timer](https://user-images.githubusercontent.com/65288322/202640190-75949073-acce-495d-8a86-f933b27ccd61.png)<br>
timer.cs

---

후<br>
![image](https://user-images.githubusercontent.com/65288322/202639520-2520640b-7226-4daf-81e3-b83fa4dd3901.png)<br>
<br>
states.cs

![image](https://user-images.githubusercontent.com/65288322/202640598-795b160d-8bdb-4474-9808-49512c51cf12.png)<br>
추가된 IstateObserver

![image](https://user-images.githubusercontent.com/65288322/202640665-4aa9b57a-3925-492e-b285-58e10ad70c04.png)<br>
<br>
States.cs의 옵저버 구현부분

<br>

![image](https://user-images.githubusercontent.com/65288322/202640758-b5423f1b-19d1-4605-bc5d-5de8c75ebf64.png)<br>
<br>
IObserver.cs

![image](https://user-images.githubusercontent.com/65288322/202640512-e8cd4cab-352f-4576-b803-ccd542d009b8.png)<br>
move.cs

![image](https://user-images.githubusercontent.com/65288322/202640489-03f79534-919a-4e76-9b94-927f8d292931.png)<br>
<br>
timer.cs<br>


해당 내용처럼 states 클래스를 기준으로 상태 변화시 감지하는 옵저버 패턴을 구현하였다.

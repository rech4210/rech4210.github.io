---
layout: post
title: "유니티 C# 델리게이트"
date: 2022-11-04 19:57:04 +0900
tags : Unity,C#
---
## 델리게이트
C#의 기능중 델리게이트라는 것이 있다.

함수의 값을 참조해서 저장하고 호출시 저장된 함수를 실행시키는 대리자라고 한다.

기본적인 형식은 이러하다.
> public delegate 반환형 델리게이트이름(매개변수 인자);

```c#
public delegate float ReturnFloatDele(float f_number);

```

이처럼 델리게이트에 함수를 저장하고 호출시키는 용도로 사용한다.

반환형과 매개변수는 델리게이트의 특성을 나타내므로 저장될 함수와 반드시 같아야 한다.

예를들어
```c#
    public delegate float ReturnFloatDele(float f_number);
    public delegate void FloatDele(float f_number);
    public delegate int IntDele(int number);

    public ReturnFloatDele returnf_Dele;
    public FloatDele f_Dele;
    public IntDele i_Dele;
```
델리게이트를 선언한 후 형식에 맞게 작성하여 사용하면 된다.

사용 방법은 각 델리게이트 변수에 함수를 추가하는 방식이다.

여기 미리 정의해둔 변수가 있다

```c#

    void hi()
    {
        Debug.Log("hi" + name);
    }
	
    void bye()
    {
        Debug.Log("bye" + name);
    }

    void floatCalc(float f_number)
    {
        Debug.Log(f_number);
    }
	
    float getfloatCalc(float f_number)
    {
        return f_number;
    }
	
    int getIntCalc(int number)
    {
        return number;
    }
```

자 이제 이걸 각 델리게이트에 추가해보자.

```java
void Start()
    {
        returnf_Dele = getfloatCalc;  float 타입 반환 델리게이트
        f_Dele = floatCalc;  float 형 매개변수 델리게이트
        i_Dele = getIntCalc; int 형 반환 델리게이트
    }
```
#### 부착될 메소드들은 서로간의 반환타입, 매개변수가 동일해야만 한다.


허나 한가지 예외가 있는데...


### Multicast
> 멀티캐스트는 대리자에게 다중 메소드를 부착시키는 방법이다.

다만 매개변수와 반환타입 모두 void 형식이여야 가능하다는 점을 주의하자.

``` java
void Start()
    {
        multi = hi;
        multi += bye; + 연산자를 통해 추가가 가능하다.
        multi();
    }

 void hi()
    {
        Debug.Log("hi" + name);
        
    }
    void bye()
    {
        Debug.Log("bye" + name);
    }

```
![delegate](https://user-images.githubusercontent.com/65288322/199981011-c972e1bb-23df-41b0-9e65-e10d28650038.png)

음... 잘 작동한다.

그 외 람다식, 익명 메소드를 이용해 델리게이트에 부착하는것 또한 가능하다.

```java
        Multicast multicast = new Multicast(delegate() { Debug.Log("it's me"); });
        multicast();

        ReturnFloatDele f_returndele = f_number => { return f_number; }; //람다식
        ReturnFloatDele f_returndele_2 = delegate(float f_number) { return f_number = 5; }; // 익명 메소드
        f_Dele = f_number => { Debug.Log(f_number); };
```

그래서 이걸 어떻게 쓸건데?

델리게이트의 존재 의의는 다커플링, 분리, 병목처리이다.
한쪽에서 다른쪽의 존재를 알 필요가 없는 경우. 또는 구현되지 않았을 경우가 그 예시이다.

여기 캐릭터의 스탯과 스탯을 증가시키는 스크립트가 있다.

### Player.CS
``` c#
public class Player : MonoBehaviour
{
    public delegate int intDele(int number);
    public intDele intdele;

    int _hp;
    int _damage;

    public int hp { get { return  _hp;} set { _hp = value; Debug.Log("현재 체력 : "+_hp); } }
	
    public int damage { get { return _damage; } set { _damage = value; Debug.Log("현재 공격력 : "+_damage); } }

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            intdele(5); 
        }
    }

}

```

위 스크립트는 캐릭터의 체력, 데미지의 정보를 가지고 있으며 프로퍼티로 값 변경을 출력한다.

업데이트 문에서 스페이스를 누르면 자동으로 캐릭터의 공격력 또는 체력이 증가하게 된다.


### Buff.CS
```c#
public class Buff : MonoBehaviour
{
    Player intDelegate;
    void Start()
    {
        Player = gameObject.GetComponent<Player>();
        Player.intdele = upHp;
    }
    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Alpha1)) intDelegate.intdele = upDamage;
        if (Input.GetKeyDown(KeyCode.Alpha2)) intDelegate.intdele = upHp;
    }
    int upHp(int number)
    {
        return intDelegate.hp += number;
    }
    int upDamage(int number)
    {
        return intDelegate.damage += number;
    }
}
```

이 스크립트는 플레이어 스크립트가 가진 델리게이트 객체에 접근하여 upHP, upDamage를 키 1번,2번과 매핑시켜 준다.

매핑이 완료되면 플레이어 스크립트의 intdele 객체에 포인터로 부착되며 각 메소드가 실행된다.

![결과](https://user-images.githubusercontent.com/65288322/199996234-1fc95250-417c-4ee1-b9df-be1b912fd634.png)

잘 된다.

끝!
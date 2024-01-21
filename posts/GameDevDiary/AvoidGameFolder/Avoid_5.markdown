---
layout: post
title: 공격 로직 수정
description:
date: 2024-01-20 19:31:12 +0900
tags :
---

## 문제점
공격 개체들이 늘어남에 따라 Generate 클래스의 역할과 타입 판단이 늘어남.

만약 공격 spawner의 종류가 50개라면 50개의 비교 스크립트 또는 타입을 관리하려면 여간 불편한게 아닐것이다.  
현재 분리된 데이터들을 관리할 필요성이 생겼다  

이처럼 통합관리를 위해 제너릭을 사용하고자 한다.

<br>


## 수정 전
### AbstractAttackCard .cs
---
```csharp
public abstract class AbstractAttackCard : MonoBehaviour, ISetCardInfo,ICardSkill
{
	// Datas....
	// Other Methods...
    public void Skill()
    {
        switch (attackInfo.attackCardEnum)
        {
            case AttackCardEnum.skill_1:
                attackGenerator.UseSkill(attackStatus, "skill_1");
                break;
            case AttackCardEnum.skill_2:
                attackGenerator.UseSkill(attackStatus, "skill_2");
                break;
            case AttackCardEnum.skill_3:
                attackGenerator.UseSkill(attackStatus, "skill_3");
                break;
            default:
                break;
        }
    }
}
```
- AbstractAttackCard : 공격 타입에 대한 정보를 가진 카드 클래스이며 카드 사용으로 효과를 발동시킨다 (추상 클래스)

카드마다 스킬을 가지고 있을 때, 이를 사용하려면 Generater가 UseSkill이 사용될 타겟을 알아야 한다.  
또한 현재 Invoke를 활용하여 스킬을 발동시키는데, 스트링 타입을 넘기는것 자체가 하드코딩을 불러올 수 있고 수정 또한 번거롭다.  
<br>

### AtkObjStat .cs
---
```csharp
public class AtkObjStat : MonoBehaviour
{
	// Datas....
	// Other Methods...
    public float Point { get { return point;} }
    public void GetAtkObjPoint(AttackStatus attackStatus)
    {
        this.point = attackStatus.point;
        this.speed = attackStatus.speed;
    }
}
```

- AttackObj : 공격 개체들이 가질 기본 기능을 구현한다 (추상 클래스)

각 공격 개체마다 스탯의 구성이 다르며 인자의 개수 또한 다르기에 초기화 함수를 설정하고 타입에 따라 다양한 초기화를 제공 할 필요가 있다.

예를 들어 Trap 공격의 경우 speed가 필요없거나, 스킬이 구현되어 있지 않을 경우 알맞은 초기화를 제공하여야 한다.

<br>

### AttackSpwaner .cs
---
```csharp
public abstract class AttackSpwaner : MonoBehaviour, IUseSkill
{
	// Datas....
	// Other Methods...

    public abstract void Skill_1();
    public abstract void Skill_2();
    public abstract void Skill_3();

}
```
- AbstractSpwaner : 공격 개체 생성을 담당하는 Spwaner이다

AttacGenerator에서 일반화 사용을 위해 generic을 사용해줘야한다.


첫번째 문제는 IUseSkill 를 받아 각 스포너 개체마다 Skill을 구현하도록 지시하고 있다는 것.  
Spawner에서 스킬을 가지도록 하기에 [스포너 - 공격 ]개체가 분리된 현재 코드에서는 올바르지 않다.  
그렇기에 인터페이스를 제거하고 통합하여 사용할 수 있는 함수로 변경하도록 한다.  

두번째 문제로 Skill 발현 제어가 어렵다는 것이다.

```csharp
// AttackGenerater.cs
public void Skill()
    {
        switch (attackInfo.attackCardEnum)
        {
            case AttackCardEnum.skill_1:
                attackGenerator.UseSkill(attackStatus, "skill_1");
                break;
            case AttackCardEnum.skill_2:
                attackGenerator.UseSkill(attackStatus, "skill_2");
                break;
            case AttackCardEnum.skill_3:
                attackGenerator.UseSkill(attackStatus, "skill_3");
                break;
            default:
                break;
        }
    }
```
bool 함수를 사용하지 않고 AbstractAttackCard.cs에서 해당 카드가 가진 카드 정보를 읽어와 swich 문을 수행한다.  
이로 인해 스킬 제어가 어려워진다.  

<br>

### AttackGenerator .cs
---

- AttackGenerater : 공격 카드로 부터 정보를 가져와 공격 개체를 생성 및 관리한다.

```csharp
public class AttackGenerator : MonoBehaviour
{
	// Datas....
	// Other Methods...
	private List<AttackSpwaner> objectsComponent = new List<AttackSpwaner>();

    public void Generate(AttackStatus status)
    {
        var obj = Instantiate(attackObjectPrefab[(int)status.attackType],
        RandomPose(),transform.rotation);   

        var component = obj.GetComponent<AttackSpwaner>();
        component._Player = attackTarget;
        component._AttackStatus = status;

		attackObjects.Add(obj);
        objectsComponent?.Add(component);
    }

    public void IncreaseTargetStat(AttackStatus status, AttackCardInfo info)
    {
        foreach (var obj in objectsComponent)
        {
            if (obj._AttackType == status.attackType)
            {
                obj.GetComponent<AttackSpwaner>()?.CalcStat(status,info);
            }
        }
    }

    public void UseSkill(AttackStatus status, string skill)
    {
        foreach (var obj in objectsComponent)
        {
            if (obj._AttackType == status.attackType)
            {
                obj.GetComponent<AttackSpwaner>()?.Invoke(skill,0);
            }
        }
    }
}
```

하나의 AttackSpwaner 로 업캐스팅하여 관리하고있다.

```csharp
//업캐스팅 된 개체를 담는 list
private List<AttackSpwaner> objectsComponent = new List<AttackSpwaner>();
```
AttackSpwaner

- Trap
- Guided
- Laser
- Bullet

이는 분리된 4개의 리스트로의 관리를 힘들게 만든다.

결과적으로, 타입을 유추하지 못해 foreach문을 돌리며 각 요소마다 attacType과 비교하여 실행하고 있다.  
이는 list 요소가 삭제될 때 foreach 문제 치명적인 문제를 일으킬 수 있다.  

UseSkill 함수는 각 Spwaner의 조건을 검사하고 Invoke를 통해 skill 함수를 발동 시키는데, 이는 매우 좋지 않은 방식이다  
하드 코딩이며 변경에 대처하기 어렵다.  

<br>

## 변경 후

### AbstractAttackCard.cs
---
```csharp
public abstract class AbstractAttackCard : MonoBehaviour, ISetCardInfo, ICardSkill
{
   	// Datas....
	// Other Methods...

    public void Skill<T>() where T : AttackSpwaner<T>
    {
        switch (attackCardInfo.attackCardEnum)
        {
            case AttackCardEnum.skill_1:
                attackGenerator.SetSkillActive<T>(attackStatus, 1);
                break;
            case AttackCardEnum.skill_2:
                attackGenerator.SetSkillActive<T>(attackStatus, 2);
                break;
            case AttackCardEnum.skill_3:
                attackGenerator.SetSkillActive<T>(attackStatus, 3);
                break;
            default:
                break;
        }
    }
}
```

Skill 함수 사용시 개체 식별을 위해 제너릭 타입을 적용시켰으며 int 값 인자를 넘겨주어 bool값 판단을 하도록 하였다.  


<br>

### AttackSpwaner.cs
---
```csharp
public abstract class AttackSpwaner<T> : MonoBehaviour, ITimeEvent where T : AttackSpwaner<T>
{
   	// Datas....
	// Other Methods...
    protected bool sk_1 = false;
    protected bool sk_2 = false;
    protected bool sk_3 = false;

    public void SetSkillBool(int i)
    {
        switch (i)
        {
            case 0: sk_1 = true; break;
            case 1: sk_2 = true; break;
            case 2: sk_3 = true; break;
            //case 3: sk_4 = true; break;
            //case 4: sk_5 = true; break;
            default:
                break;
        }
    }
}
```
- AbstractAttackCard : 공격 타입에 대한 정보를 가진 카드 클래스이며 카드 사용으로 효과를 발동시킨다 (추상 클래스)

- 제너릭 타입을 적용하였다.
- 상속받는 자식들이 공통으로 사용할 수 있는 함수로 변경하여 attackGenerator와의 참조관계를 끊고 매개변수를 전달하도록 변경하여 모듈화에 집중하였다.
- 기존 Skill 함수에 사용된 (string 타입, attackStatus) 매개변수 대신 bool 타입을 사용하여 유연성을 확보하였다.

<br>

### AtkObjStat.cs
---
```csharp
public abstract class AtkObjStat<T> : MonoBehaviour  where T : AtkObjStat<T>
{
   	// Datas....
	// Other Methods...
    public float Point { get { return point;} }
    protected void GetAtkObjPoint(AttackStatus attackStatus)
    {
        this.point = attackStatus.point;
        this.speed = attackStatus.speed;
    }

    public virtual void Initialize(AttackStatus attackStatus)
    { GetAtkObjPoint(attackStatus); }
    public virtual void Initialize(AttackStatus attackStatus, bool skill_1)
    { GetAtkObjPoint(attackStatus); }
    public virtual void Initialize(AttackStatus attackStatus, bool skill_1, bool skill_2)
    { GetAtkObjPoint(attackStatus); }
    public virtual void Initialize(AttackStatus attackStatus, bool skill_1, bool skill_2, bool skill_3)
    { GetAtkObjPoint(attackStatus); }
    public virtual void Initialize(AttackStatus attackStatus, bool skill_1, bool skill_2, bool skill_3, bool skill_4)
    { GetAtkObjPoint(attackStatus); }
}
```
- AttackObj : 공격 개체들이 가질 기본 기능을 구현한다 (추상 클래스)

- 제너릭 타입으로 교체하였다. (제너릭 타입을 사용하지 않을 경우 부모의 virtual 함수를 사용한다)
- Initialize 함수 오버로딩을 제공하여 공격 개체에 따라 다양한 유형의 초기화를 제공하였다.

virtual 타입으로 사용 한 이유는 abstract로 구현 시 해당하는 모든 Initialize 함수 강제 구현을 피하기 위함이다.

<br>
AtkObjStat<T> 개체들은 제너릭 타입을 적용하였는데, Spwaner에서 Instanciate와 동시에 Initialize 함수를 호출한다.  
그렇기에 제너릭 타입을 사용하지 않으면 부모의 virtual 함수를 사용하게 되는 경우가 발생한다.  

(물론 이 경우 Instanciat 후 함수호출 대신 Start 함수에서 사용해주어도 될 것이다.)

아래는 이를 상속받은 개체들이다.

```csharp
public class BulletObj : AtkObjStat<BulletObj>,IUseSkill
{
    public override void Initialize(AttackStatus attackStatus, bool skill_1)
    {
        GetAtkObjPoint(attackStatus);
        if (skill_1)
            Skill();
    }
}

public class TrapObj : AtkObjStat<TrapObj>, IUseSkill
{
    public override void Initialize(AttackStatus attackStatus)
    {
        GetAtkObjPoint(attackStatus);
    }
}
```

이로써 오버로딩된 Initialize를 수행한다.

<br>

### AttackGenerator.cs
---
```csharp
public class AttackGenerator : MonoBehaviour
{
   	// Datas....
	// Other Methods...
    private List<LaserTurret> laserList = new List<LaserTurret>();
    private List<TrapTurret> trapList = new List<TrapTurret>();
    private List<GuidedTurret> guidedList = new List<GuidedTurret>();
    private List<BulletTurret> bulletList = new List<BulletTurret>();

    public void Generate<T>(AttackStatus status, AttackCardInfo info) where T : AttackSpwaner<T>
    {
        var obj = Instantiate(attackObjectPrefab[(int)status.attackType],
        RandomPose(),transform.rotation,StageManager.Instance.GetCurrentStagePos());

        var component = obj? obj.GetComponent<AttackSpwaner<T>>() : null
        component.Initalize(status,info,attackTarget);
        component.DeadAction(() =>
        {
            GetTurretList<T>(status).Remove(component as T);
            obj.SetActive(false);
        });
        StoreList(component);
    }

    private void StoreList<T>(AttackSpwaner<T> AttackSpwaner) where T : AttackSpwaner<T>
    {
        GetTurretList<T>(AttackSpwaner._AttackStatus).Add(AttackSpwaner as T);
    }

    public void IncreaseTargetStat<T>(AttackStatus status, AttackCardInfo info) where T : AttackSpwaner<T>
    {
        var list = GetTurretList<T>(status);
        for (int i = 0; i < list.Count; i++)
        {
            list[i].CalcStat(status, info);
        }
    }

    public void SetSkillActive<T>(AttackStatus status, int skillNum) where T : AttackSpwaner<T>
    {
        var list = GetTurretList<T>(status);
        for (int i = 0; i < list.Count; i++)
        {
            list[i].SetSkillBool(skillNum);
        }
    }

    private List<T> GetTurretList<T>(AttackStatus status) where T : AttackSpwaner<T>
    {
        switch (status.attackType)
        {
            case AttackType.laser:
                return laserList as List<T>;
            case AttackType.guided:
                return guidedList as List<T>;
            case AttackType.bullet:
                return bulletList as List<T>;
            case AttackType.trap:
                return trapList as List<T>;
            default:
                return null;
        }
    }
}
```
- AttackGenerator : 공격 카드로부터 정보를 가져와 공격 개체를 생성 및 관리한다.

- T 타입별로 만든 리스트를 통해 각 개체를 관리할 수 있게 되었다.
- GetTurretList T 함수를 통해 각 개체와 List T 를 매칭시켜줌.
- SetSkillActive T, IncreaseTargetStat T 함수에서 더 이상 개체 확인을 위해 반복문을 사용하지 않음.
- SetSkillActive T 함수를 이용해 Invoke를 사용할 때 보다 유연하고 변경에 대처하기 쉬워졌다.
- foreach문을 사용하지 않아 데이터 요소 삭제로부터 자유로움.

<br>

```csharp
//각 개체를 담는 list
private List<LaserTurret> laserList = new List<LaserTurret>();
private List<TrapTurret> trapList = new List<TrapTurret>();
private List<GuidedTurret> guidedList = new List<GuidedTurret>();
private List<BulletTurret> bulletList = new List<BulletTurret>();

// 해당하는 list를 반환하는 T타입 함수
private List<T> GetTurretList<T>(AttackStatus status) where T : AttackSpwaner<T>
{
    switch (status.attackType)
    {
        case AttackType.laser:
            return laserList as List<T>;
        case AttackType.guided:
            return guidedList as List<T>;
        case AttackType.bullet:
            return bulletList as List<T>;
        case AttackType.trap:
            return trapList as List<T>;
        default:
            return null;
    }
}

// 객체의 action 부착. T 타입개체 판단으로 객체 제거시의 액션을 전달하기 쉬워졌다.  
// 게다가 리스트에 직접 접근하지 않아도 된다는 것이 큰 장점!  
component.DeadAction(() =>
{
    GetTurretList<T>(status).Remove(component as T);
    obj.SetActive(false);
});
```

이와 같이 공격에 해당하는 스크립트들에 Generic을 적용하여 관리를 용이하게 만들었다.  
다만, 이해하기 어려운 감이 꽤 있다.  
제너릭이 일반화시켜 편하게 관리할 수 있다지만 협업하는 입장에서는 꽤나 난처한 문법이라 생각한다.  

사실 가장 좋은 방법은 이러한 구조를 짜지 않고도 관리하기 편하고 가독성도 좋은 코드를 짜는게 아닐까 싶다.  

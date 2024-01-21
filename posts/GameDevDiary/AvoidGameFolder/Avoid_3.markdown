---
layout: post
title: 버프 디버프 카드 시스템 작업
description:
date: 2024-01-20 13:55:04 +0900
tags :
---
## 문제점

현재 로직에서는  StatusEffect를 상속받는 Debuff와 Buff가 있고  이들을 BuffManager에서 관리한다.  

여기서 문제가 생길수 있는데, 이 둘을 나눴기 때문에 리스트로 관리하기 힘들어 지는 부분이 생긴다는거다.  
예컨데 카드를 선택할 때 해당 개체가 buff인지 debuff인지 확인하는 과정이 추가로 필요해진다.


```csharp
//통합 관리가 어려워짐에 따라 생길 수 있는 문제.
void UseBuff(Buff buff);
void UseDebuff(Debuff debuff);

List<Buff> buff;
List<Debuff> debuff;

//관리해야 할 데이터 구조도 나뉘게 된다.
BuffData buffData;
DebuffData debuffData;
```

<br>

## 수정 전
```csharp
public abstract class StatusEffect : MonoBehaviour
{
    protected BuffManager buffManager;
    public int rank;

    public abstract void Init();
    public abstract void OnChecked();
    protected void FindBuffManager(BuffManager buff)
    {
        try
        {
            if (GameObject.FindWithTag("BuffManager")
            .TryGetComponent<BuffManager>(out buff))
            {
                this.buffManager = buff;
            }
        }
        catch (System.NullReferenceException e)
            throw e;
    }
}

public abstract class Buff : StatusEffect
{
    public int point;
    public abstract void BuffUse();
    public abstract void BuffUp();
    public abstract void BuffDown();
    public abstract void RankUp(int rank);
}

public abstract class DeBuff : StatusEffect
{
    public int point;
    public abstract void DeBuffUse();
    public abstract void DeBuffUp();
    public abstract void DeBuffDown();
    public abstract void RankUp(int rank);
}
```

<br>


또한 새로운 기능 (데이터를 변경 이외에 특정한 기능)을 넣게 될 경우 결국, buffmanager가 버프 카드의 정보와 공격 개체의 정보를 동시에 등록해야하는 중복되는 커플링이 생길 수 있다는것.

ex) 카드를 사용하면 버프매니저를 거쳐 AttackGenerater를 통해 공격 개체에 특수 스킬을 사용한다.

결국 요구사항과 구조에 대해 깊게 고민해 볼 필요가 있게 되었다.


## 변경 후

### StatusEffect .cs
---
```csharp
public abstract class StatusEffect : MonoBehaviour, ISetCardInfo
{
    char buffCode;
    protected BuffManager buffManager;
    CardInfo cardInfo;
    BuffStat buffStat;

    public char _BuffCode { get { return buffCode; } set { buffCode = value; } }
    public CardInfo _CardInfo { get { return cardInfo; } set { cardInfo = value; } }
    public BuffStat _BuffStat { get { return buffStat; } set { buffStat = value; } }

    public abstract void OnChecked();
    protected void FindBuffManager(BuffManager buff)
    {
        try
        {
            if (GameObject.FindWithTag("BuffManager")
            .TryGetComponent<BuffManager>(out buff))
            {
                this.buffManager = buff;
            }
        }
        catch (System.NullReferenceException e)
        {
            Debug.LogError($"에러 대상:{this.name }$에러 내용: {e.Message}");
            throw e;
        }
    }
    public virtual void GetRandomCodeWithInfo(BuffData data)
    { _BuffCode = data.buffCode; _CardInfo = data.cardInfo; _BuffStat = data.stat; }

    public virtual void SetCardInfo()
    {
        //set cardInfo
    }
}
```
해결법
- Buff와 Debuff의 구분을 없애고 CardInfo, BuffData 타입으로 통합시켜 관리하도록 하였다.
- Buff 데이터 구조를 변경시켰다.
- 차별화된 기능을 구현시 interface를 활용하도록 계획.

### BuffData.cs
---
```csharp
public class BuffData
{
    public char buffCode;
    public CardInfo cardInfo;
    public BuffStat stat;

    public void Print()
    {
        UnityEngine.Debug.Log($"code:{buffCode},{cardInfo.cardName},{stat.point}");
    }
    public BuffData(char buffCode, BuffStat stat, CardInfo cardInfo)
    {
        this.buffCode = buffCode;
        this.stat = stat;
        this.cardInfo = cardInfo;
    }
}
```

기존 구조에서는 Buff와 Debuff에 대한 정보를 나누어 관리하였다면,
이젠 하나의 개체로 관리하되 버프와 디버프의 구분이 필요하다면 interface를 활용하거나 코드로 식별하는 방법을 사용하면 된다.

처음부터 구조를 확정지으면 나중에 수정할 일이 꼭 생기는 것 같다.
데이터를 관리할 땐 * **인터페이스가 어떻게 구성될 것인지 구상** * 해두어야 데이터 타입의 관리도 수월한 것 같다.

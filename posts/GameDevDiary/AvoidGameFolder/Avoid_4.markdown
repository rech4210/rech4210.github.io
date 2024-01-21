---
layout: post
title: Json 파싱 제어 변경 / 데이터 타입 변경
description:
date: 2024-01-20 17:42:04 +0900
tags :
---

## 문제점
카드의 정보를 Json 파일에 저장해두고 Parsing을 통해 불러오고 있다.

card를 buff와 attack 개체로 나누어두어 Json 파일을 분리하여 관리하기에 Parsing 부분에서 타입이 다르기에 비슷한 내용의 함수를 반복적으로 구현해야하는 불편함이 생겼다.
<br>

### BuffDataJson.cs
---
```csharp
{
    "buff": [
        {
            "buffCode": 0,
            "cardInfo": {
                "buffType": 3,
                "cardName": "1",
                "bGImage": "1",
                "fRImage": "1",
                "information": "1",
                "description": "1"
            },
            "stat": {
                "rank": 1,
                "point": 1,
                "useValue": 1,
                "upValue": 1
            }
        }
     ]
}
```
<br>

### AttackData.json
---
```csharp
{
    "attackDatas": [
        {
            "attackCode": 0,
            "attackStatus": {
                "attackType": 0,
                "rank": 1,
                "point": 1,
                "duration": 2,5,
                "scale": 1.0,
                "speed": 1.0
            },
            "attackInfo": {
                "attackCardEnum": 0,
                "attackName": "Laser Attack",
                "bGImage": "Laser",
                "fRImage": "Laser_f",
                "information": "Spawn Laser Object",
                "description": "In time the Spark explosed"
            }
        }
     ]
 }
```

이를 사용하는 스크립트에서도 BuffData, AttackData에 따른 함수를 여럿 구현해주어야 하며
BuffManager 역할 취지에 부합하지 않는다고 판단, Parsing 기능을 DataManager에서 처리할 수 있도록 변경해 주어야한다.

<br>

## 수정 전


### BuffManager.cs
---
```csharp
//Other Properties
//Other Member
//Other Method
private void Awake()
{
    JsonParsing();
    JsonAttackParsing();
}

public void JsonParsing()
{
    path = Path.Combine(Application.dataPath + "/Json/", "BuffData.json");

    string jsonData = File.ReadAllText(path);
    Debug.Log(jsonData);
    Structure _buffData = JsonUtility.FromJson<Structure>(jsonData);

    for (int i = 0; i < buffCounts; i++)
    {
        allBuffStatArchive.Add(_buffData.buff[i].buffCode, _buffData.buff[i].stat);
        allCardInfoArchive.Add(_buffData.buff[i].buffCode, _buffData.buff[i].cardInfo);
    }
}

public void JsonAttackParsing()
{
    path = Path.Combine(Application.dataPath + "/Json/", "AttackData.json");

    string jsonData = File.ReadAllText(path);
    Debug.Log(jsonData);
    AttackStructure _attackStruct = JsonUtility.FromJson<AttackStructure>(jsonData);
    for (int i = 0; i < 10; i++)
    {
        allAttackStatArchive.Add(_attackStruct.attackDatas[i].attackCode, _attackStruct.attackDatas[i].attackStatus);
        allAttackCardInfoArchive.Add(_attackStruct.attackDatas[i].attackCode, _attackStruct.attackDatas[i].attackInfo);
    }
}
```

- JsonParsing()
- JsonAttackParsing()

함수명만 보더라도 불필요한 일을 반복하고 있다.

이런 구조는 전혀 좋지 않다.
심지어 함수 내부를 보면 중복되는 내용이 다수이다.  
만약 다른 개체의 데이터를 관리한다면 또 이렇게 함수를 여럿 만들어 둘 것인가?

Parsing 부분을 보자.

각 함수를 수행할 때마다 for문을 돌며 dictionary에 요소를 저장하고 있다.  
이를 DataManager에서 수행하도록 할 수도 있지만, 완성된 데이터를 받아오는것이 좀 더 가시성이 좋을것 같다.
굳이 Parsing이 어떻게 이루어지는지 코드를 늘리면서 까지 Manager에서 보여줄 이유가 없을 테니 말이다.

이를 통합적 관리하도록 하자.
<br>

### StructDatas.cs / Buff 데이터 부분
---
```csharp
[Serializable]
public class Structure
{
    public BuffData[] buff;

    public Structure()
    {
        buff = new BuffData[100];
    }
}

[Serializable]
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
[Serializable]
public class CardInfo
{
   // CardInfo
}
[Serializable]
public struct BuffStat
{
    //BuffStatInfo
}
```
<br>

### StructDatas.cs / Attack 데이터 부분
---
```csharp
[Serializable]
public class AttackStructure
{
    public AttackData[] attackDatas;

    // 배열 길이 수정할것
    public AttackStructure()
    {
        attackDatas = new AttackData[10];
    }
}

[Serializable]
public class AttackData
{
    public char attackCode;
    public AttackStatus attackStatus;
    public  AttackCardInfo attackInfo;

    public AttackData(char attackCode, AttackStatus status ,AttackCardInfo info)
    {
        this.attackCode = attackCode;
        attackStatus = status;
        attackInfo = info;
    }
}
[Serializable]
public struct AttackStatus
{
    //AttackStatus
}
[Serializable]
public class AttackCardInfo
{
    //AttackCardInfo
}
```

Parsing시 깔끔하게 정리하려면 어떻게 하는게 좋을까?  
위의 for문을 Parsing 단에서 내부 함수를 구현하여 처리하는 것이 가장 보기 좋을것이다.
<br>

## 변경 후

### DataManager .cs
---
```csharp
public class DataManager : Manager<DataManager>
{
    //Other Properties
  	//Other Member
  	//Other Method

    private void Awake()
    {
        var buffDataAsset  = Resources.Load<TextAsset>("Json/BuffData");
        var attackDataAsset = Resources.Load<TextAsset>("Json/AttackData");

        buffsArchive = LoadJson<BuffStructure, int, BuffData>(buffDataAsset).MakeDict();
        attackArchive = LoadJson<AttackStructure, int, AttackData>(attackDataAsset).MakeDict();
    }

    Loader LoadJson<Loader, Key, Value>(TextAsset asset) where Loader : IDataLoader<Key, Value>
    {
        Loader data = JsonUtility.FromJson<Loader>(asset.ToString());
        return data;
    }
}
```
- DataManager : 데이터를 통합적으로 관리하는 매니저 스크립트.

- 반복되는 작업을 통합시킴
- DataManager에서 직접 수행하던 작업을 MakeDict 함수를 통해 내부적으로 처리함.

<br>

```csharp
// MakeDict() 함수를 통해 일괄 삽입
var buffDataAsset  = Resources.Load<TextAsset>("Json/BuffData");
    buffsArchive = LoadJson<BuffStructure, int, BuffData>(buffDataAsset).MakeDict();

Loader LoadJson<Loader, Key, Value>(TextAsset asset) where Loader : IDataLoader<Key, Value>
    {
        Loader data = JsonUtility.FromJson<Loader>(asset.ToString());
        return data;
    }
```
Parsing 부분을 압축한 부분이다.

꽤 복잡해 보이지만 간단하다.  
Loader 타입을 반환하는 함수 LoadJson은 <Loader, key, Value>의 제너릭 값을 가지며 제약 조건으로 Loader는 IDataLoader<Key, Value>를 구현하는 개체이다.

TextAsset 를 매개변수로 받아 FromJson<Loader>를 수행한다.
<br>
### StructDatas.cs , IDataLoader<Key,Value>
---
```csharp
public interface IDataLoader<Key,Value>
{
    Dictionary<Key, Value> MakeDict();
}

[Serializable]
public class BuffStructure : IDataLoader<int, BuffData>
{
    public BuffData[] buffDatas;
    public BuffStructure()
    {
        buffDatas = new BuffData[100];
    }

    public Dictionary<int, BuffData> MakeDict()
    {
        Dictionary<int, BuffData> dict = new();
        foreach (var item in buffDatas)
        {
            if(item != null)
                dict.Add(item.buffCode, item);

        }
        return dict;
    }
}
```
<br>

Manager에서 수행했던 작업을 StructDatas.cs 의 BuffStructure : IDataLoader<int, BuffData> 형태로 인터페이스를 구현.  
** MakeDict() ** 함수를 통해 dictionary를 반환하며 내부적으로 처리할 수 있도록 변경시켰다.

---

이렇게 Json 파싱 부분을 수정해보았다.  
Json 파일을 다루면서 느낀것이 참 많다.  

- Json 문법이 하나만 어긋나도 하드코딩이 된다.
- 데이터의 수정이 자주 일어난다면 이 방식이 꽤나 불편하다는것.
- JsonUtillity의 경우 Enum의 값이 숫자로만 들어가진다 (NewtonSoftJson은 괜찮다는 듯)

아무래도 추후에 엑셀 시트를 가져오도록 해야 할 것같다.

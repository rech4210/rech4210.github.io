---
layout: post
title: 초기 게임 시스템 전체 구조 작성
description:
date: 2024-01-19 21:00:04 +0900
tags :
---
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/6fdf303c-9c2a-41b4-995c-3af8efe274e9)

# 게임 시스템 전체 구조

게임을 구성하는 전체 구조를 간략하게 정리한다.



## 플레이어 제어

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/9f85f3ec-e015-4d58-b006-8cde04a2e7aa)

플레이어 관련 스크립트부터 작성해 볼것이다.

- BuffManager : Buff 카드 데이터를 저장, 관리하며 Player Stat에 카드 효과를 적용시켜준다.
- Player Stat : 플레이어의 스탯과 현재 상태를 저장한다.
- Player : 플레이어의 이동과 충돌 관리를 설정한다.

### Player.cs
---
```csharp
public class Player : MonoBehaviour
{
    Vector3 direction;
    Vector3 desVector;
    Vector3 refValue = Vector3.zero;
    Rigidbody rigid;

    private float velocityLimit = 0.3f;
    private float walkDeaccelerationOnX;
    private float walkDeaccelerationOnZ;
    private float walkDeacceleration = 3f;

    [Range(5, 30)] private float movePower = 13;

    void Start()
    {
        rigid = GetComponent<Rigidbody>();
    }

    void Update()
    {
        direction = new Vector3(Input.GetAxis("Horizontal"), 0, Input.GetAxis("Vertical")).normalized * movePower;
    }
    private void FixedUpdate()
    {
        rigid.velocity = new Vector3(Mathf.SmoothDamp(direction.x, 0, ref walkDeaccelerationOnX,walkDeacceleration),0,
                                     Mathf.SmoothDamp(direction.z, 0,ref walkDeaccelerationOnZ,walkDeacceleration));
        if (rigid.velocity.magnitude < velocityLimit)
        {
            rigid.velocity = Vector3.zero;
            rigid.angularVelocity = Vector3.zero;
        }
    }
}
```
플레이어의 이동과 충돌을 담당하는 스크립트이다.

- Player : 플레이어의 이동과 충돌 관리를 설정한다.

해당 내용을 수행하도록 플레이어 스탯과 분리해 모듈화를 시켰다.
부드러운 움직임을 위해 smoothDamp를 이용했다.

### BuffManager .cs
---
```csharp
public class BuffManager : MonoBehaviour
{

    private Dictionary<char, BuffData> allBuffStatArchive = new();
    private Dictionary<char, BuffStat> containBuffDict = new();

    Player player = null;
    private void Start()
    {
        DataManager.Instance.ReturnDict(allBuffStatArchive);
    }

    public void AddorUpdateDictionary(char buffCode)
    {
        if (containBuffDict.ContainsKey(buffCode))
        {
            containBuffDict[buffCode] = BuffUp(containBuffDict[buffCode]);
            Debug.Log($"이미 존재합니다 버프 랭크 증가");
        }
        else
        {
            containBuffDict.Add(buffCode, allBuffStatArchive[buffCode].stat);
            Debug.Log("없는 버프 추가 : " + allBuffStatArchive[buffCode].cardInfo.cardName + " " + "현재 버프 갯수:"+ containBuffDict.Count);
        }
        Debug.Log($"대상 버프 : {allBuffStatArchive[buffCode].cardInfo.cardName}, " +
            $"스탯 상승 : {containBuffDict[buffCode].point}, " +
            $"랭크 : {containBuffDict[buffCode].rank}");
    }

    private BuffStat BuffUp(BuffStat buffStat)
    {
        buffStat.rank++;
        buffStat.point += buffStat.upValue;
        return buffStat;
    }

    public void RemoveSomthing(char buffCode)
    {
        if (containBuffDict.ContainsKey(buffCode))
        {
            containBuffDict.Remove(buffCode);
        }
    }

    public BuffStat ReturnBuff(char buffCode)
    {
        return containBuffDict[buffCode];
    }

    public Dictionary<char, BuffStat> ContainStatToGenerate()
    {
        return containBuffDict;
    }
}

```

BuffManager 에서는 * ScreenPointToRay(Input.mousePosition) * 를 통해 가져오고 카드의 함수를 발동시킨다 (OnChecked)
카드들은 BuffManager를 참조하여 각 함수를 거쳐 리스트에 저장한다.


### PlayerStat.cs
---
```csharp
public struct PlayerStatStruct
{
    public float health;
    public float defence;
    public float speed;
    public float will;

    public PlayerStatStruct(float health, float defence, float speed, float will, float damage, float avoid, float block)
    {
        this.health = health;
        this.defence = defence;
        this.speed = speed;
        this.will = will;
        this.damage = damage;
        this.avoidness = avoid;
        this.blockness = block;
    }
    public float damage;
    public float avoidness;
    public float blockness;
}
public struct PlayerAbilityStruct
{
    int indurance;
    int luck;
    int agility;
    int wisdom;
    int faith;
}
public class PlayerStat : MonoBehaviour
{
    PlayerStatStruct playerStat;
    public void GetDamaged(float dmg)
    {
        playerStat.health -= dmg;
        Debug.Log(playerStat.health);
        if (playerStat.health <= 0f)
        {
            isLive = false;

            DeadEvents.Instance.ExecuteEvent();
        }
    }
    bool isLive = true;

    void Start()
    {
        playerStat = new PlayerStatStruct(50,3,15,0,0,0,0);
    }

    private void OnTriggerEnter(Collider other)
    {
        if (other.gameObject.CompareTag("Background")) { }
        else if(isLive)
        {
            var attackObject = other.gameObject.GetComponent<AtkObjStat>();
            GetDamaged(attackObject.Point);
            attackObject.OnHitTarget();
        }
    }
}
```
플레이어의 상태 변화와 초기화 데이터를 관리하는 클래스이다.

플레이어의 스탯에 관한 정보를 enum으로 정리하였다.
그 외 플레이어의 충돌에 따른 로직, 피격시 판정을 이곳에서 처리하게 된다.

## 버프 생성 카드 관리
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/0e2546c9-7524-4c8b-aff2-bfa74857460b)

### StatusEffect.cs && Buff.cs

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
버프와 디버프 모두 StatusEffect 를 상속받는다.

상속받은 개체들은 함수를 구현하고 각자의 역할을 하도록 설정해두었다.
구현할 함수로는

- 사용: 처음 사용시 카드의 효과를 높게 설정
-  재사용: 동일한 카드 사용시 효과를 감소
-  감소: 카드 능력을 감소시킨다 (랭크)

카드가 처음 발동될 때와 두번 이상 발동되는 경우 차별점을 주고 싶어 내부적으로 함수를 구현해두었다.



## 공격

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/667b9c7b-71e2-4027-b732-98a34e7e7bfd)

이번엔 공격 개체 생성과 구조를 작성해볼것이다.
우선 공격 개체들을 생성시킬 매니저를 만든다.

### AttackGenerater .cs
---
```csharp
public class AttackGenerator : MonoBehaviour
{
    [SerializeField]
    private GameObject[] attackObjectPrefab;

    private Dictionary<char, AttackData> allAttackStatArchive = new();
    private Dictionary<char, AttackStatus> containAttackDict = new();

    private GameObject attackTarget;
    private List<GameObject> attackObjects = new List<GameObject>();
    private List<AttackFunc> objectsComponent = new List<AttackFunc>();

    public void AddorUpdateAttackDictionary(char attackCode, AttackStatus attackStatus)
    {
        if (containAttackDict.ContainsKey(attackCode))
        {
            containAttackDict[attackCode] = attackStatus;
            Debug.Log($"랭크 증가");
        }
        else
        {
            containAttackDict.Add(attackCode, allAttackStatArchive[attackCode].attackStatus);
            Debug.Log("없는 공격 추가 : " + allAttackStatArchive[attackCode].attackInfo.attackName + " " + "현재 버프 갯수:" + containAttackDict.Count);
        }
        Debug.Log($"대상 공격 : {allAttackStatArchive[attackCode].attackInfo.attackName}, " +
            $"스탯 상승 : {containAttackDict[attackCode].point}, " +
            $"랭크 : {containAttackDict[attackCode].rank}");
    }
    public Dictionary<char, AttackStatus> ContainAttackStatToGenerate()
    {
        return containAttackDict;
    }
    private void Start()
    {
        DataManager.Instance.ReturnDict(allAttackStatArchive);
        FindPlayer();
    }

    protected void FindPlayer()
    {
        try
        {
            if (GameObject.FindWithTag("Player").TryGetComponent<Player>(out Player player))
            {
                // 게임오브젝트 컴포넌트 자체를 가져오려고 생긴 문제였음.
                attackTarget = player.gameObject;
                Debug.Log(attackTarget + "공격 대상");
            }
        }
        catch (NullReferenceException e)
        {
            throw e;
        }
    }

    public void Generate(AttackStatus status)
    {
        Debug.Log(((int)status.attackType).ToString());
        var obj = Instantiate(attackObjectPrefab[(int)status.attackType], RandomPose(),this.transform.rotation);
        attackObjects.Add(obj);

        var component = obj.GetComponent<AttackFunc>();
        component._Player = attackTarget;
        component._AttackStatus = status;

        objectsComponent?.Add(component);
    }

    public void IncreaseTargetStat(AttackStatus status, AttackCardInfo info)
    {
        foreach (var obj in objectsComponent)
        {
            if (obj._AttackType == status.attackType)
            {
                obj.GetComponent<AttackFunc>()?.CalcStat(status,info);
            }
        }
    }

    public void UseSkill(AttackStatus status, string skill)
    {
        foreach (var obj in objectsComponent)
        {
            if (obj._AttackType == status.attackType)
            {
                obj.GetComponent<AttackFunc>()?.Invoke(skill,0);
            }
        }
    }

}
```

- AttackGenerater : 공격 카드로 부터 정보를 가져와 공격 개체를 생성 및 관리한다.

<br>

### AbstractAttackCard .cs
```csharp
public abstract class AbstractAttackCard : MonoBehaviour, ISetCardInfo,ICardSkill
{
    protected int skillCheckNum = 100;

    protected AttackGenerator attackGenerator;
    protected BuffManager buffManager;

    protected char attackCode;
    AttackType attackType;
    AttackStatus attackStatus;
    AttackCardInfo attackInfo;
    public AttackCardInfo _AttackCardInfo { get { return attackInfo; }}
    public AttackStatus _AttackStatus { get { return attackStatus; }}

    public abstract void OnChecked();

    protected void FindBuffWithAttackGenerator()
    {
        try
        {
            if (GameObject.FindWithTag("AttackGenerator")
            .TryGetComponent(out AttackGenerator attack))
            {
                this.attackGenerator = attack;
            }
            if (GameObject.FindWithTag("BuffManager")
            .TryGetComponent(out BuffManager buff))
            {
                this.buffManager = buff;
            }
        }
        catch (System.NullReferenceException e)
        {
            Debug.LogError($"에러 대상:{this.name}$에러 내용: {e.Message}");
            throw e;
        }
    }

    public virtual void GetRandomCodeWithInfo(char _attackCode, AttackCardInfo _cardInfo, AttackStatus _attackStatus)
    { this.attackCode = _attackCode; attackInfo = _cardInfo; this.attackStatus = _attackStatus; }

    public virtual void CalcAttackStatus(float calcNum, string statType)
    {
        this.attackStatus.rank++;
        switch (statType)
        {
            case "duration":
                this.attackStatus.duration *= calcNum; break;
            case "point":
                this.attackStatus.point += (int)calcNum; break;
            case "scale":
                this.attackStatus.scale *= calcNum; break;
            default:
                break;
        }
    }

    public virtual void SetCardInfo()
    {
        // get cardInfo
    }

    public void Skill()
    {
        switch (_AttackCardInfo.attackCardEnum)
        {
            case AttackCardEnum.skill_1:
                attackGenerator.UseSkill(_AttackStatus, "skill_1");
                break;
            case AttackCardEnum.skill_2:
                attackGenerator.UseSkill(_AttackStatus, "skill_2");
                break;
            case AttackCardEnum.skill_3:
                attackGenerator.UseSkill(_AttackStatus, "skill_3");
                break;
            default:
                break;
        }
    }
}
```
- AbstractAttackCard : 공격 타입에 대한 정보를 가진 카드 클래스이며 카드 사용으로 효과를 발동시킨다 (추상 클래스)

아래는 이를 상속받은 스포너 클래스이다.

<br>

```csharp
public class BulletSpwaner : AbstractAttackCard
{
    public void Start()
    {
        FindBuffWithAttackGenerator();
    }
    // 온체크시 버프 매니저에게 영향을 줘야함.수정? 어택 제너레이터에서 해야할듯.
    public override void OnChecked()
    {
        if ((int)_AttackCardInfo.attackCardEnum > skillCheckNum)
        {
            Skill();
        }
        else if (_AttackCardInfo.attackCardEnum == AttackCardEnum.generate)
        {
            attackGenerator?.Generate(_AttackStatus);
        }

        else if ((int)_AttackCardInfo.attackCardEnum < skillCheckNum)
        {
            attackGenerator.IncreaseTargetStat(_AttackStatus, _AttackCardInfo);
        }
        attackGenerator.AddorUpdateAttackDictionary(attackCode, _AttackStatus);
        this.gameObject.SetActive(false);
    }

    public override void SetCardInfo()
    {
        base.SetCardInfo();
    }
}
```
공격 개체가 하나 씩 늘어날수록 작성해야 할 부분이 많아지고 다양하게 대처해야 하기에 추상 클래스를 상속받아 처리하고자 하였다.

<br>

### AtkObjStat.cs
```csharp
public class AtkObjStat : MonoBehaviour
{
    protected float speed;
    protected float point;
    protected float duration;
    protected float scale;

    public float Point { get { return point;} }
    public void GetAtkObjPoint(AttackStatus attackStatus)
    {
        this.point = attackStatus.point;
        this.speed = attackStatus.speed;
    }

    public virtual void OnHitTarget()
    {
        Destroy(gameObject);
    }
}
```

- AttackObj : 공격 개체들이 가질 기본 기능을 구현한다 (추상 클래스)

<br>

## 카드 생성

### GenerateCard .cs
---
```csharp
public class GenerateCard : MonoBehaviour
{
    private char buffCode;
    private Dictionary<char, BuffStat> statGenerateDic;
    private Dictionary<char, CardInfo> infoGenerateDic;
    public GameObject cardPrefab;
    PowerUp powerUp;

    public int cardCount;

    GraphicRaycaster graphicRaycaster;
    PointerEventData pointerEventData;

    [SerializeField]
    EventSystem eventSystem;
    private void Start()
    {
        graphicRaycaster = GetComponent<GraphicRaycaster>();

        if (GameObject.FindWithTag("BuffManager")
            .TryGetComponent<BuffManager>(out BuffManager buffManager))
        {
            this.statGenerateDic = buffManager.StatToGenerate();
            this.infoGenerateDic = buffManager.InfoToGenerate();
        }
        Generate();
    }

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Mouse0))
        {
            pointerEventData = new PointerEventData(eventSystem);
            pointerEventData.position = Input.mousePosition;

            List<RaycastResult> raycastResults = new List<RaycastResult>();

            graphicRaycaster.Raycast(pointerEventData, raycastResults);

            foreach (var result in raycastResults)
            {
                if(result.gameObject.transform.parent.TryGetComponent<PowerUp>(out PowerUp component))
                {
                    powerUp = component;
                    powerUp.OnChecked();
                }
                else
                {
                    Debug.Log("찾을수 없는 컴포넌트 대상");
                }
            }
        }
    }
    void Generate()
    {
        for (int i = 0; i < cardCount; i++)
        {
            var cardObj = Instantiate(cardPrefab,this.transform);
            var _buffcode = (char)Random.Range(1, statGenerateDic.Count + 1);

            cardObj.GetComponent<PowerUp>()
                .GetRandomCodeWithInfo(_buffcode, infoGenerateDic[_buffcode]);
        }
    }
}
```
GenerateCard 는 딕셔너리에 저장된 데이터를 가져온 후 버프와 공격 개체 카드를 생성한다.
우선 버프에 관한 함수만 작성해두었다.

이곳에서 화면에 띄워줄 카드를 생성한다.

<br>

## 메인 시스템 싱글톤
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/7477eeaf-067e-49b1-bdbf-e9c9467b7974)
다음으로 메인 시스템에 해당하는 싱글톤을 만들것이다.

static 클래스와 싱글톤중 고민을 해봤는데, 아래와 같은 이유로 싱글톤을 하는게 더 이득이 클 것 같다.

- 싱글톤은 분리 가능한 의존성으로 연결되어 있지만 정적 클래스는 하드코딩에 가깝다.
- 객체 지향 관점에서 싱글톤이 더 유리하다 (내부 개체 변경 및 유연한 설계)

<br>

### Events<T>.cs
---
```csharp
public abstract class Events<T> :MonoBehaviour where T : Events<T>
{
    protected void Start()
    {
        DontDestroyOnLoad(gameObject);
        Execute();
    }

    private static  T instance;
    public static T Instance {
        get
        {
            if(instance == null)
            {
                instance = Object.FindObjectOfType(typeof(T)) as T;
            }

            return instance;
        }
    }
    public System.Action OnExecute;

    protected abstract void Execute();
    public virtual void ExecuteEvent()
    {
        OnExecute?.Invoke();
    }
}
```

나는 싱글톤을 만들 되 싱글톤 역할을 수행할 개체를 제너릭으로 만들고자 하였다.

추상클래스를 상속받고 공통되는 action 부분을 구현하여 어느 개체에서나 접근하고 역할을 액션에 넘겨주고 호출할 수 있도록 말이다.

아래는 Events를 상속받는 싱글톤 클래스이다.

<br>

### ClearEvent : Events<ClearEvent>
---
```csharp
public class ClearEvent : Events<ClearEvent>
{
    protected override void Execute()
    {
        if (OnExecute?.Method == null)
        {
            OnExecute += Clear_1;
            OnExecute += Clear_2;
        }
    }

    private void Clear_1()
    {
        Debug.Log("clear_1");
    }
    private void Clear_2()
    {
        Debug.Log("clear_2");
    }
}
```
이벤트 함수를 상속받아 Instance 개체로 접근이 가능하며, 액션에 부착된 역할을 수행하도록 한다.
부모에서 이미 action이 정의되어 있기 때문에 clear 또한 부모의 자원을 활용 할 수 있다.

<br>

```csharp
// 호출 방법
ClearEvent.Instance.ExecuteEvent();
```

> ### 왜 제너릭인가?
제너릭 타입을 쓴 이유는 관리의 용이성을 위해서이다.
여러 싱글톤을 만들게 되었을 때, 클래스마다 그 기능을 중복해서 구현하는것에 불편함이 있으며 공통된 부분을 제어하기 편하기 때문이다.

마찬가지로 일반화를 통해 각 개체마다 같은 호출에 다른 기능을 수행할 수 있으니 꽤나 매력적이지 않은가?

물론 처음부터 필요로 하지 않고 필요할 때 생성시킬 거라면 여기에 Lazy Singleton 처리를 해주면 될 것이다.

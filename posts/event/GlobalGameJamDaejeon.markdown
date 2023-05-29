---
layout: post
title: 글로벌 게임잼 대전
tags : Game Jam
description:
---

![KakaoTalk_20230530_032358459_01](https://github.com/rech4210/rech4210.github.io/assets/65288322/04ed768d-a107-4c9c-a10b-d994766dd437)  
뿌리가 주제
이번에 참여한 게임잼은 글로벌 게임잼으로 전세계에서 동시에 열리는 큰 행사이다.(두근두근)    

저 화면을 통해 전세계 게임잼 사람들이 같은 주제를 가지고 게임을 제작한다고 한다.  
이번년도의 주제는 "뿌리" 말 그대로 뿌리이다.  
뿌리라는 단어를 듣자마자 아이디어가 생각날 정도로 재밌는 주제였던것 같다.


우선 아이디어와 팀원을 구상한 후 자리 정리를 하였다.  
나의 팀은 나와 프로그래머 1명.  
그러니까 총 2명의 프로그래머가 게임을 제작하게 된 것이다.  

![KakaoTalk_20230530_032358459_02](https://github.com/rech4210/rech4210.github.io/assets/65288322/d4e3ca43-85f7-47a6-8eea-21a0bc2c7f3f)  
내가 가져온 개발도구

우선 크게 포탑, 적으로 나누어 내가 적을 담당하기로 했다.  

진행하기전 포탑에 대한 구조를 설계하였는데, 간략히 Node가 부모로 존재하고 하위 오브젝트인 Turret, Barrier 등이 상속받아 같은 기능을 가지도록 하였다.  
각 노드들은 말 그대로 포탑을 자식으로 가질수 있다. (상속의 자식이 아니라 진짜 자식이 생긴다)  
그러면 각 노드에서 자식을 만들면 그 노드는 부모가 되며, 아래 노드를 자르거나 고칠수 있다.  

생성된 자식들은 부모의 능력을 유전받아 더 강해지는 컨셉도 추가되었다.  

가장 중요한것은 리프노드는 부모노드의 정보를 가지고 있어, 지키지 못하면 부모 노드들이 다음 순서로 공격받고 뿌리 노드가 부셔지면 게임이 종료된다는 것이다.  

참 재밌지 않은가? ㅎㅎㅎㅎ  

<br>
<br>
<br>


다음으로 적에 대한 구현이다.
대략적으로 적이 공격하는 시나리오는 이러하다.  

1. 턴 또는 시간에 따라 적이 쳐들어온다.
1. 적은 정해진 위치에서 spawner를 통해 생성된다.
1. 생성될때 적의 스탯은 동적으로 생성된다 (강한 개체, 약한 개체)
1. 적은 리프노드를 향해 공격을 개시한다. -> 리프노드 탐색
1. 적이 리프노드에 닿을경우 해당 오브젝트에 공격한다. (방어벽, 노드, 또는 포탑)
1. 적이 노드를 공략하면 적의 체력이 회복되고 개체수가 증가한다.
1. 적이 루트 노드에 닿으면 게임이 종료된다.  

적을 구현하는데에 최대한 책임을 분리하기위해 EnemyData에 적의 정보를 담고 ScriptableObject 형식으로 데이터를 가져와 Enemy.cs에서 사용하도록 하였다.
이번 프로젝트를 수행하며 가장 중점으로 두었던것은, 책임분리이다.  

그렇기에 EnemySpwaner, EnemyData, Enemy 로 서로간의 역할을 나누어 책임을 분할시켰고, 파괴해야할 노드들의 데이터를 GameManager와 Spawner가 받아 좀비들에게 뿌려주기로 하였다.  

```cs
EnemyData.cs
    [SerializeField]
    private string enemyName;
    public string EnemyName { get { return enemyName; } }
    /*--------------------------------------------------*/

    [SerializeField]
    private int hp;
    public int Hp { get { return hp; } }
    /*--------------------------------------------------*/

    [SerializeField]
    private float attackSpeed;
    public float AttackSpeed { get { return attackSpeed; } }

    [SerializeField]
    private int damage;
    public int Damage { get { return damage; } }
    /*--------------------------------------------------*/

    [SerializeField]
    private float moveSpeed;
    public float MoveSpeed { get { return moveSpeed; } }
    /*--------------------------------------------------*/
```  

## 좀비의 초기값 설정  
이 데이터를 받아온 Enemy는 초기값을 설정하게된다.  

```cs  
Enemy.cs
public void Initialize(EnemyData enemyData, UnityAction<Enemy> onDie)
  {
      canMoving = false;

      hp = enemyData.Hp;
      damage = enemyData.Damage;
      attackSpeed = enemyData.AttackSpeed;
      moveSpeed = enemyData.MoveSpeed;
      enemyMoney = enemyData.EnemyMoney;
      enemyAudio = enemyData.EnemyAudio;
      enemyName = enemyData.EnemyName;

      this.onDie = onDie;
  }
```

## 에네미 스포너  
에네미 스포너에서는 좀비들을 코루틴에 따라 생성하고 난이도를 관리한다.
```cs  
EnemySpwaner.cs

[SerializeField] private List<EnemyData> enemydata = new List<EnemyData>();
// 에네미 데이터를 담은 리스트
private Dictionary<int, int> waveLevelDictionary = new Dictionary<int, int>();
// 난이도를 담아놓은 딕셔너리 난이도, 좀비 숫자를 키와 값 쌍으로 해두었다.

// 좀비들이 추적해야할 리프노드 (끝노드)들을 담아놓은 리스트다.
// 리프노드가 제거될때마다 좀비들에게 새로 리프노드를 탐색하도록 명령한다.
public void SetLeafNodeList(List<Node> leafNodes)
    {
        leafNodeList = leafNodes;
        if (enemiseList?.Count != 0)
        {
            for (int i = 0; i < enemiseList.Count; i++)
            {
                enemiseList[i]?.SetFisrtNode(leafNodeList[Random.Range(0, leafNodeList.Count)]);
            }
        }
    }


Enumerator GenerateEnemy()
    {
        Debug.Log(leafNodeList.Count);
        while (true)
        {
            var _randomNum = System.Enum.GetValues(typeof(EnemyType)).Length;
            var _randomPosX = Random.Range(-1f, 1f);
            var _randomPosY = Random.Range(-1f, 1f);

            var pos = new Vector3(transform.position.x + _randomPosX, transform.position.y + _randomPosY);
            var enemyData = enemydata[Random.Range(0,enemydata.Count-1)];
            var enemy = Instantiate(enemyData.EnemyPrefab, pos, Quaternion.identity).GetComponent<Enemy>();
            enemy.Initialize(enemyData, OnEnemyDie);

            enemiseList.Add(enemy);

            // 딕셔너리에 저장된 좀비의 숫자만큼 생성한다.
            if (enemiseList.Count >= waveLevelDictionary[waveLevel])
            {
                SoundManager.PlaySound(AudioEnum.StartSound);
                yield return new WaitForSeconds(2f);
                for (int i = 0; i < enemiseList.Count; i++)
                {
                    enemiseList[i].SetFisrtNode(leafNodeList[Random.Range(0, leafNodeList.Count)]);
                    enemiseList[i].StartMove();
                }
                Debug.Log(enemiseList.Count);
                yield break;
            }
            yield return wait;
        }
    }


```  


이 적들은 크게 3가지 동작을 하는데, 노드 탐색 및 이동, 공격, 피격이다.  

### 노드의 탐색 및 이동
```cs
private void MoveToNode()
    {
        if (currentNode == null)
        {
            return;
        }

        var dir = (transform.position - currentNode.transform.position);
        var moveDelta = Time.deltaTime * moveSpeed * TempSpeedValue;
        unitedPos = new Vector2((currentNode.transform.position.x + gap), currentNode.transform.position.y);

        transform.position = Vector3.MoveTowards(this.transform.position, unitedPos, moveDelta);

        Debug.Log(currentNode + "+" + currentNode.transform.position);
        float angle = Mathf.Atan2(dir.y, dir.x) * Mathf.Rad2Deg;
        transform.rotation = Quaternion.Euler(0, 0, angle + 90);

        if (dir.sqrMagnitude <= attackRange * attackRange)
        {
            isAttacking = true;
            lastAttackTime = Time.time;
        }
        else
        {
            tempShakeIntensity = 0;
        }
    }
```

### 공격
```cs
private void AttackNode()
{
    if (lastAttackTime + attackSpeed < Time.time)
    {
        SoundManager.PlaySound(AudioEnum.EnemyAttack);
        lastAttackTime = Time.time;

        currentNode.OnDamage(damage);

        Shake();
    }

    if ((transform.position - currentNode.transform.position).sqrMagnitude > (attackRange * 2) * (attackRange * 2))
    {
        isAttacking = false;
    }
}
```

### 죽음 + 자원 반환  

```cs
private void Die()
    {
        onDie?.Invoke(this);

        switch (enemyName)
        {
            case "normal":
                DataManager.GetMoney(enemyMoney);
                SoundManager.PlaySound(enemyDeadAudio);
                break;
            case "strong":
                DataManager.GetMoney(enemyMoney);
                SoundManager.PlaySound(enemyDeadAudio);
                break;
            case "range":
                DataManager.GetMoney(enemyMoney);
                SoundManager.PlaySound(enemyDeadAudio);
                break;
            default:
                break;
        }
   }
```  

여기에 DataManager.GetMoney라고 보일것이다.  
데이터를 관리할 대상이 많아지니 머리가 정말 아파졌다...  
어떻게 하는게 가장 좋은 방법일까?

생각하던 찰나, 싱글톤을 사용하기로 했다.  
다만 호출이 있기 전까진 생성되지 않고, 늦게 생성되는 Lazy 싱글톤이란걸 한번 사용해보기로 했다.  
지금 아니면 언제 사용해보겠나?  

### 제너릭 타입 매니저
```cs
public class Manager<T> where T : new()
{
    protected static T instance;

    public static T GetInstance()
    {
        if (instance == null)
        {
            instance = new T();
        }
        return instance;
    }
}

```  
정말 간단하게 매니저 하나를 만들어주었다.  
각 매니저의 호출이 있으면, GetInstance 함수가 발동되면서 새로운 매니저가 인스턴스화 될것이다.  

많은 매니저가 있지만 중요한 DataManager를 한번 살펴보겠다.  

### DataManager (자원 관리)
DataManager의 역할은 좀비 처치시 얻을 자원을 계산해주고 UI와 연동하는 역할을 한다.  
또한 건물을 지을때 필요한 자원을 체크하는 역할도 한다.  
```cs  
public class DataManager : Manager<DataManager>
{
    public Resourse resourse;
    private Resourse startResourse = new(200,0);
    // 초기 돈 200원을 준다

    public DataManager()
    {
        resourse = startResourse;
    }
}

```  


### DataManager의 함수들  
보통 자원에 관한 내용으로 CheckResource() 함수는 가스를 주는 적에 따라 오버로딩을 했다.  
이곳은 돈을 얻거나, 건물을 지을 때 자원을 체크하는 함수들이다.
```cs  
public static void GetMoney(int money)
{
    GetInstance().resourse.money += money;
    SoundManager.PlaySound(AudioEnum.Money);
    GetInstance().PrintResource();

    GameManager.instance.UpdatePanelUI();
}
public static void GetGas(int gas)
{
    GetInstance().resourse.gas += gas;
    GetInstance().PrintResource();

}
public static bool CheckResource(int money)
{
    var instance = GetInstance();

    if (instance.resourse.money >= money)
    {
        instance.resourse.money -= money;
        instance.PrintResource();

        GameManager.instance.UpdatePanelUI();
        return true;
    }
    else
    {
        instance.SendError();
        return false;
    }
}
public static bool CheckResource(int money, int gas)
{
    var instance = GetInstance();

    if ((instance.resourse.money > money) && (instance.resourse.gas > gas))
    {
        instance.resourse.money -= money;
        instance.resourse.gas -= gas;
        instance.PrintResource();

        return true;
    }
    else
    {
        instance.SendError();

        return false;
    }
}

private void SendError()
{
    Debug.Log("자원이 부족합니다");
}
private void PrintResource()
{
    Debug.Log($"현재 자원 {resourse.money} / {resourse.gas}");
}
```  


### 학습시간  
![KakaoTalk_20230530_032358459_03](https://github.com/rech4210/rech4210.github.io/assets/65288322/54503365-aed9-4c1d-82c8-40cad1d0adb8)    
자고 일어나니 모두 사이좋게 학습을 듣고있는 모습이다.  

![KakaoTalk_20230530_032358459_05](https://github.com/rech4210/rech4210.github.io/assets/65288322/e670201f-c90f-4418-8823-180d4dbf63da)  
배도 고픈데 밥은 먹고 해야지? ㅎㅎ  

![KakaoTalk_20230530_032358459_04](https://github.com/rech4210/rech4210.github.io/assets/65288322/4487ec84-99bd-43f2-a2b8-09f64492a98c)  
주위를 둘러보니 다들 힘들어보인다.  
확실히 이틀 밤 새거나 하면 체력적으로 절대 못 버틴다...  

조금만 더 힘내서 해보자.  

마지막은 GameManager를 살펴보면 될 듯 하다.  
### GameManager  
GameManager에는 초기 UI 팝업과 노드, DataManager가 공유하는 함수들이 있다.  
그중 내가 작성한것만 올리도록 하겠다.  
```cs  

// 리프노드의 위치를 정렬하는 함수
private void SetSpawnerLeafNodeList()
    {
        foreach (var spawner in enemySpawner)
        {
            spawner.SetLeafNodeList(tree.GetLeafNodes());
            spawner.transform.position = new Vector2(tree.treeArea.xMax + distanceFromNode, spawner.transform.position.y);
        }
    }


//웨이브의 시작을 spawn에 전달하는 함수.
public void WaveStart()
    {
        if (wave < 10)
        {
            foreach (var spawner in enemySpawner)
            {
                spawner.AttackStart();
            }
            Wave++;
            panelUI.UpdateWaveText(Wave, 10);

        }
    }


//여기 DataManager.CheckResource 함수가 사용된다.  
    private void OnClickRepairButton(Node node)
    {
        if (!DataManager.CheckResource(balance.RepairCost))
        {
            return;
        }

        node?.Repair();;
    }
```  

이렇듯 전체적인 구조는 최대한 공유하지 않되, 게임 내적으로 연동시켜야하는 재화, UI, 사운드 등은 싱글톤을 이용해 처리하였다.  
구조를 짤 때 시간을 꽤 많이 들였는데, 이전 프로젝트에 비해 깔끔하게 나온것 같아 나름 만족스럽다.  

![KakaoTalk_20230530_032358459_06](https://github.com/rech4210/rech4210.github.io/assets/65288322/d02c78aa-5caa-42a8-aa42-9a6e11841863)  
이제 마지막 밥을 먹을 시간이다.  
사람들도 이제 다 마무리를 한 듯 준비를 하는 모습이다.  
이미 끝난 사람들은 메이플 BGM 틀어놓고 쉬고있었음 ㅋㅋ  


![KakaoTalk_20230530_032358459_07](https://github.com/rech4210/rech4210.github.io/assets/65288322/a4f9290d-9598-4326-b779-732f4a76d91d)  
루트투 노드 완성!  
드디어 우리 팀도 완성했다.  
게임이 궁금하다면 아래 유튜브를 확인해보시면 될듯하다!  

[유튜브](https://youtu.be/p6PsjPhFjFc)

관심이 있다면 글로벌 게임잼 사이트에서 다운받아 해보시길... ㅎㅎ..   
[게임잼 사이트](https://globalgamejam.org/2023/games/roottonode-7)

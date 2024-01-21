---
layout: post
title: Events & Manager 역할 분리
description:
date: 2024-01-21 01:23:04 +0900
tags :
---

## 문제점
Events, 매니저간 역할 분담이 모호함

action을 수행할 Events  
싱글톤 역할로 데이터 관리를 수행할 Manager 간의 역할을 분리해야한다.  

메인 시스템 개체의 생명 주기 제어가 어렵다.

<br>

## 수정 전

### Events.cs
---
```csharp
public abstract class Events<T> where T : Events<T>
{
    public static T instance;
    public Events()
    {
        if (instance != (T)this || instance == null)
        {
            instance = (T)this;
        }
    }

    public static System.Action<T> OnExecute;
    protected abstract void Execute();
    public virtual void ExecuteEvent()
    {
        Execute();
        OnExecute?.Invoke((T)this);
    }
}

// 매니저이지만 이벤트와 동일한 내용을 상속받는다.
public class DeadEvents : Events<DeadEvents>
public class DataManager : Events<DataManager>

```

이벤트에 관련된 내용을 매니저와 공유하고 있다.
매니저에서 Action이 필요가 없기에 이를 수정해야한다.

<br>

## 변경 후

### Events.cs
---
```csharp
public abstract class Events<T> :MonoBehaviour where T : Events<T>
{
    protected void Awake()
    {
        if(instance != this && instance != null)
        {
            Destroy(this.gameObject);
        }
        ActionInitiallize();
    }

    private static  T instance;
    public static T Instance {
        get
        {
            if(instance == null)
            {
                instance = FindObjectOfType(typeof(T)) as T;
            }
            return instance;
        }
    }
    public System.Action OnExecute;

    protected abstract void ActionInitiallize();
    public virtual void ExecuteEvent()
    {
        OnExecute?.Invoke();
    }
}
```
Event 클래스는 애초 목적에 맞게 다른 클래스에서 접근하여 액션을 부착할 수 있도록 설정해두었다.

<br>

### Manager.cs
---
```csharp
public abstract class Manager<T> : MonoBehaviour where T : Manager<T>
{
    protected void Start()
    {
        if (instance != this && instance != null)
        {
            Destroy(gameObject);
        }
    }
    private static T instance;
    public static T Instance
    {
        get
        {
            if (instance == null)
            {
                instance = FindObjectOfType(typeof(T)) as T;
            }
            return instance;
        }
    }
}
```

<br>

### GameManager .cs
---
```csharp
public class GameManager : Manager<GameManager>
{
    [SerializeField] protected GameObject player;
    Transform playerTransform;
    public Transform _PlayerTransform { get { return _PlayerTransform; } }

    private void Awake()
    {
        Time.timeScale = 1f;
        var obj = Instantiate(player, new Vector3(0,.5f,0),Quaternion.identity);
        obj.SetActive(true);
        playerTransform = obj.transform;
    }
}
```

Manager와 event의 역할을 분리할 스크립트를 작성하였다.  
- Manager의 초기 목적에 맞게 데이터의 통합 관리 및 생명 주기 제어에 초점을 맞춰 수정하였다.  
- DontdestoryOnLoad 제거

DontdestoryOnLoad 제거함으로써 생명주기 제어를 직접적으로 하게 되었다.

### 왜 DontdestoryOnLoad를 사용하지 않는가?

[참고 문서 - 프로젝트가 확장될 때 코드 설계 방법](https://unity.com/kr/how-to/how-architect-code-your-project-scales#clean-and-controlled-shutdown)
> LoadSceneMode.Single을 사용하지 말고 대신 LoadSceneMode.Additive를 사용하는 것이 좋습니다.
>씬을 언로드할 때는 명시적 언로드를 사용해야 합니다. 결국에는 씬을 전환할 때 일부 오브젝트를 유지해야 하기 때문입니다.
>
>DontDestroyOnLoad도 사용하지 않는 것이 좋습니다. 이를 사용하면 오브젝트의 수명을 제어할 수 없습니다. 실제로 LoadSceneMode.Additive를 사용해 로딩할 경우, DontDestroyOnLoad를 사용할 필요가 없습니다. 대신 오래 사용해야 하는 오브젝트를 오랜 기간 유지되는 특별 씬에 넣는 것이 더 좋습니다.

플레이어가 죽을 시 DontDestroyOnLoad를 통해 생성한 매니저의 생명주기를 제어할 수 없다는 것이 큰 이유였다.  
그대로 놔둬도 현재 플레이에 문제는 없었지만, 이후 스테이지, 맵이 분리되거나 통합될 때 이를 관리한다보면 분명 문제가 생길것이다.  
차라리 Manager 개체들을 모두 다른 씬에 넣어두고 위의 설명처럼 Additive를 사용하는 것도 하나의 방법이 될 것이다.  

생명주기 다이어그램을 그릴때만 해도 난해하고 이해가 잘 안되었는데, 제어할 일이 생기니 꼭 넘겨짚고 가야 할 부분이었다.  

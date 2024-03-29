---
layout: post
title: "미니게임 제작일지"
description: 과녁들을 UI위에 일괄 생성하도록 하고 타겟을 상속으로 구현하였다
date: 2022-12-30 22:29:04 +0900
tags : Unity,DevDiary
---
```
1차
과녁들을 UI위에 일괄 생성하도록 하고 타겟을 상속으로 구현하였다
```

### 오브젝트를 UI 위에 배치하기  

레일 위에 표적을 생성하기위해서는 별도의 카메라가 필요하다  
## *URP 기준입니다!  

<br>
![UI카메라](https://user-images.githubusercontent.com/65288322/210081424-4f5aa18c-f656-4b8e-94a0-a4d80ab805af.png)  
메인 카메라와 별개의 게임 오브젝트를 담당할 카메라를 생성한다.  
<br>

<br>
![카메라스태킹](https://user-images.githubusercontent.com/65288322/210081450-b95b5b12-ae9e-4364-9fec-794b05cd8948.png)  
메인 카메라는 base로 설정하고 ui위에 출력할 대상을 담은 카메라는 Overlay를 설정한다.  

<br>
<br>
![컬링마스크](https://user-images.githubusercontent.com/65288322/210081483-4fd81378-0a4d-4e87-98df-271de0ae5b57.png)  
레이어를 하나 만든다 이름은 마음대로!  

만들었으면 Overlay 대상 카메라 설정 중 Culling Mask의 대상을 해당 레이어만 체크하도록 설정한다.  

<br>
<br>
![개체를 OverUI로](https://user-images.githubusercontent.com/65288322/210081583-37fe38fb-96fa-4715-bdfe-ea50205a9996.png)  
마지막으로 UI위에 출력할 개체들의 레이어를 모두 OverUI로 설정하면 된다.
<br>

### 타겟을 구현하는 스크립트  
```cs
private void Awake()
{
    int randomInt = rand.Next(0, slots.Count);

    for (int i = 0; i < conveyorBelts; i++)
    {
        var gameobj = transform.GetChild(i);
        if (gameobj.GetChild(i) != null ? true : SendNullError())
        {
            for (int j = 0; j < slotsInBelts; j++)
            {
                var _slot = gameobj.GetChild(j).gameObject;
                slots.Add(_slot);
                setedTarget.Add(Instantiate(targets[UnityEngine.Random.Range(0, targets.Length - 1)], _slot.transform));
            }
        }
    }
}

```
![타겟 초기설정](https://user-images.githubusercontent.com/65288322/210076836-966fea6f-d6c8-491a-9d39-31cb84c67a92.gif)  

짠!  
모든 과녁들이 타일들 위에 생성되는데, 해당 위치들을 모두 조절해주기위해 이러한 구조를 사용하였다.  

![image](https://user-images.githubusercontent.com/65288322/210076985-7cbd88e8-7270-482b-93a6-981acaf0ca80.png)  
컨베이어 벨트 아래에는 3개의 레일 구조가 있다.  
그 아래 레일에는 총 5개의 게임오브젝트가 있는데, 이 각각이 표적이 생성될 위치를 담고있다.  

이렇게 구상한 이유는 다름이아닌 과녁의 갯수가 증가하거나 레일의 개수가 증가하는 경우에도 유연하게 대처하기 위함이다!  


![image](https://user-images.githubusercontent.com/65288322/210076429-f60caec3-d651-4867-988a-e5211c42d8f4.png)
Midium, high, low 타겟 각각 생성
이 3가지는 Target을 부모로 상속받음으로써 공통된 기능을 묶었다.  
![하이과녁](https://user-images.githubusercontent.com/65288322/210077457-589af4a1-0a74-43b5-91d6-e831216e36e3.png)  
![로우과녁](https://user-images.githubusercontent.com/65288322/210077527-eac71462-62ca-47b7-815f-b78a5b60ee5a.png)  

이처럼 과녁들은 모두 Target을 부모로 받고있도록 하여 점수의 차이를 두거나 새로운 기능을 나누어 추가하기 위함이다. :thumbsup:  
<br>  
<br>

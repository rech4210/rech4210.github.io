---
layout: post
title: "미니게임 제작일지"
description: 표적 타격 구현, 랜덤 타이머 설정, 표적 자동 복구 설정, 레이캐스트 사용
date: 2022-12-30 22:29:04 +0900
tags : Unity,DevDiary
---
```
2차
표적 타격 구현, 랜덤 타이머 설정
표적 자동 복구 설정
레이캐스트 사용
```



![타겟 간격 설정](https://user-images.githubusercontent.com/65288322/210077842-1994d58b-8cfd-4e34-acb1-3c97ad819a6d.gif)  
간단한 스크립트를 이용해서 타겟 간격을 설정하도록 하였다.  

만약 타겟의 갯수가 늘어나게될때 간격이 얼마정도 되어야 적절할까 고민하다가 수치 때려맞추기보단 이렇게 하는게 편하지 않을까 해서 제작했다...  

대신 게임 시작을 해야 테스트 할 수 있다는 점이 불편했는데 그러다 보니 찾은게 있다!!!  

## **[ExecuteInEditMode](https://docs.unity3d.com/kr/530/ScriptReference/ExecuteInEditMode.html)**

플레이를 누르지 않아도 에딧모드에서도 실행이 된다!! 엄청 신기했다.  

<br>  

그 다음으로 표적을 제거하기위한 로직을 구상했다.  
우선 레이캐스트를 활용하여 타겟을 가져오는것 부터 해보자  
```cs
if (Input.GetMouseButtonDown(0))
        {
            ray = Camera.main.ScreenPointToRay(Input.mousePosition);
            Debug.DrawRay(ray.origin, ray.direction * 50f, Color.red);

            if (Physics.Raycast(ray, out hitinfo, 50f))
            {
                try
                {
                    hitinfo.collider.GetComponent<Target>().OnRayHit();
                    Debug.Log(hitinfo.collider.gameObject.name);

                }
                catch (Exception ex)
                {
                    var st = new System.Diagnostics.StackTrace(ex, true);
                    var line = st.GetFrame(0).GetFileLineNumber();
                    Debug.LogError(ex.Message + $" 에러가 발생된 줄: {line}\n" + $"레이저 충돌대상 : {hitinfo.collider.name}");
                }
            }
        }
```

[ScreenPointToRay](https://docs.unity3d.com/ScriptReference/Camera.ScreenPointToRay.html)  
카메라로부터 스크린 포지션으로까지의 ray를 반환.

마우스 버튼을 누르면 ScreenPointToRay함수의 값으로 ray를 반환해 가져온다.  

그리고 Physics.Raycast로 충돌처리를 해주면 끝!  
간단하게 예외처리도 해주었다.  

![레이 적용](https://user-images.githubusercontent.com/65288322/210077860-a3d7596b-0b8a-4f14-8cf9-5222ffbef035.gif)  

ray를 생성하는건 완료되었으니 이제 표적을 제거하면 끝난다!  
<br>  

표적이 제거되는 스크립트는 Target.cs에 작성해두었는데, 카메라에서 발사된 ray가 물체를 인식하면 메소드가 발동되도록 작성해두었다.  
<br>

```cs  
public void OnRayHit()
    {
        if (isHited)
        {
            isHited = false;
            ReturnScore(score);
        }
    }
```  
<br>
다음으로 표적이 발생시킬 함수다.  
```cs
// 표적 적중시 발동
int ReturnScore(int score)
{
    Debug.Log($"반환 점수: {score}");
    scoreManager.ScoreUp(score);
    Destroy(gameObject, 0.5f);
    generateTarget.Generation(transform.parent);
    return score;
}
```
삭제됨과 동시에 Generation메소드를 이용해 자신의 위치에 새로운 과녁을 생성하도록 구현하였다.  


이제 플레이 해보면..!  
<br>
![레이로 타겟 제거하기](https://user-images.githubusercontent.com/65288322/210077853-cd56046a-fc5b-4bb3-9062-2b1d9d7ec82c.gif)  

예외처리를 포함하여 잘 작동하는 것 같다 ^-^  

![랜덤시간](https://user-images.githubusercontent.com/65288322/210084688-a09c4e14-d5b0-46e7-b628-fbc1d2805274.png)  

아 그리고 각 과녁은 점수가 다른 만큼 생성 시간에도 랜덤을 주었다.  
이로써 2차 일지는 끝.

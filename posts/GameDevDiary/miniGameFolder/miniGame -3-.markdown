---
layout: post
title: "미니게임 제작일지"
description: 레일의 스크롤, OnBecameInvisible을 이용해 벗어날 때 위치 수정 처리
date: 2022-12-30 22:29:04 +0900
tags : Unity,DevDiary
---
```
3차
레일의 스크롤
OnBecameInvisible을 이용해 벗어날 때 위치 수정 처리
```

맞으면 표적이 넘어지는 기능을 넣어보자.  
해당 기능은 표적이 맞으면 자동으로 발동한다!  

```cs
IEnumerator FallDown()
    {
        float u = 0;
        while(u <= 90)
        {
            u += Mathf.Cos(25) * 1;
            transform.eulerAngles = new Vector3(u,0,0);
            yield return null;
        }
    }
```
1프레임마다 호출하여 부드럽게 넘어지도록 구현하였다.  
#### 어디까지나 해당 함수는 넘어지는 애니메이션을 넣기 전에 사용할 대용이다!  

<br>
그 다음 레일 이동 기능을 추가하면 자연스레 이동하게 될 것 이다. 이 부분은 간단하니 생략  
<br>
<br>
하지만 가장 중요한 화면 바깥으로 나갔을때의 처리가 필요하다.  

화면을 나갔을때 어떻게 처리할 수 있을까... 고민을 하다가 결국 Screen의 x와 y값을 기준으로 판단하는 수 밖에 없나? 하는 순간!  
<br>
### [OnBecameInvisible](https://docs.unity3d.com/kr/530/ScriptReference/Renderer.OnBecameInvisible.html)  
이라는 콜백 메소드를 찾았다!!!  
> 오브젝트가 어떤 카메라에도 렌더링 되지 않으면 발동하는 메소드

와 무슨 유니티 없는게 없냐 신기하다 ㄹㅇ..  

<br>
그런데 이게 참 애매한게 **어떤 카메라에도** 이기 때문에 scene 탭과 game 탭을 같이 열어두면 발동 안되는 경우도 있다. ㅎ;; 이 점 참고하길 바라며  

<br>
마지막으로 점수를 텍스트와 연동해주면 끝이다  

![미니게임 완성](https://user-images.githubusercontent.com/65288322/210077855-eba2e284-a622-4e8e-8cad-59191f8587d1.gif)  
<br>
표적이 카메라 화면을 벗어나면 레일 회전 방향의 반대편으로 나오도록 설정 완료되었다.  

확실히 조잡해보인다 아직까지는..  
나머지 부분은 아트 에셋이 구해지는대로 수정을 하면 그래도 볼 만 한 수준까지 되지않을까?  

그 전까진 이 게임은 잠시 보류해두고 새로운 미니게임을 제작할 것이다.  

미니게임을 한.. 4종류 정도 더 개발하고 나서 하나의 게임의 미니게임으로 활용할 생각이다 그 날이 빨리 오길 바라며 열심히 공부해야지!  

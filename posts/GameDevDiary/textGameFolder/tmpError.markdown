---
layout: post
title: "TMP 텍스트 메쉬 프로 이상하게 뜰 때"
date: 2022-11-15 23:48:04 +0900
description: TMP 오류 해결방법
tags : Unity,C#,TMP
---
*맛있는 사람이 되자.*
<br><br><br>

텍스트 기반 게임을 만들던 중 난관에 부딪혔다.<br>
바로 텍스트가 제대로 출력되지 않는 현상이 나타난것..<br><br>

![tmp 네모 오류](https://user-images.githubusercontent.com/65288322/201338450-564ffe46-688b-44a6-9531-0df36cf3fd36.png)<br>
화면에 보이는것 처럼 상태가 아주 메롱이다.<br><br><br>

![TMP 서브메쉬뜸](https://user-images.githubusercontent.com/65288322/201338789-16602503-c1fa-4438-979d-c1e825685867.png)<br>
그리고 왠.. 이상한 녀석도 생성된다. TMP 서브메쉬? 이게 뭐지.<br>


찾아해멘 결과 문제는 두가지였다.<br>

텍스트 개체에 들어가는 텍스트의 양이 많은것이 1차적인 문제<br>
그리고 두번째로 TMP 대상이 되는 텍스트의 아틀라스 크기가 작다는 것 이다.<br>
<br><br>
### TMP설정
![tmp 설정 컴포넌트 아틀라스](https://user-images.githubusercontent.com/65288322/201343906-060ae734-1965-4b9f-b443-9f54be418f0e.png)<br>
TMP 컴포넌트 설정으로 들어가면 Atlas값을 바꿀 수 있다.
<br>
<br>
### 아틀라스가 2048일 때
![아틀라스 2048](https://user-images.githubusercontent.com/65288322/201343503-b0ff335a-1c9a-4ab2-9aa7-c0503ca1d931.gif)<br>
:angry: 정말 답답하다 이게 무슨 ㅡㅡ 간간히 텍스트가 깨지는것도 보인다.<br>
<br>
<br>

### 아틀라스가 4096일 때
![아틀라스 4096](https://user-images.githubusercontent.com/65288322/201343546-30b41759-bdb4-4ab0-9eea-1ec6652c642e.gif)<br>
:clap: 아주 잘 된다. :clap:<br>
이처럼 TMP서브메쉬가 뜨거나 텍스트가 깨지는경우 atlas값을 확인해보자.<br>

사실 근본적인 문제는 텍스트에 들어가는 양이다.<br>

거의 900줄이 넘어가는 텍스트가 존재하니 당연히 버벅일 수 밖에.<br>

그러니 만약 텍스트의 양이 많아진다면 따로 저장할 데이터셋을 마련해두거나 넉넉히 아틀라스 크기를 키워두자.<br>

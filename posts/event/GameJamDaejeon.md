---


---

<hr>
<h2 id="layout-posttitle-게임잼-대전descriptiondate-2023-08--091904-0900tags--unitygamejam-event">layout: post<br>
title: “게임잼 대전”<br>
description:<br>
date: 2023-08- 09:19:04 +0900<br>
tags : Unity,GameJam, Event</h2>
<h1 id="gen-ai-대전-게임잼">Gen AI 대전 게임잼</h1>
<p><img src="https://github.com/rech4210/rech4210.github.io/assets/65288322/eaba1062-ea20-4d26-9ee5-80c35b38cbc9" alt="image"></p>
<p>저번 글로벌 게임잼을 이후로 두번째로 대전 게임잼을 참가하게 되었다.<br>
<br><br><br><br></p>
<p>주제는 Gen AI 로 나온 결과물로 게임을 만드는것!</p>
<p>예전 Novel AI 때 잠깐이나마 diffusion 생성 모델을 다뤄본적이 있었는데, 그때와 비교하면 굉장히 발전이 된듯하다.</p>
<p>총 12명으로 3명의 팀으로 나뉘어 게임을 제작하였는데, 우리팀은 골동품 가게 컨셉을 살려 AI 이미지를 최대한 활용해보기로 했다.</p>
<p>물론 주제를 정하기 전에 가장 중요한것은 팀원들간의 합의가 아니겠는가?</p>
<p>서로 골동품이라는 컨셉을 가지로 어떻게 게임을 진행시킬지 아이디어를 의논했고 크게 두 가지 결론이 나왔다.</p>
<ol>
<li>상인에게 아이템을 주선하고 아이템을 제작 및 판매<br>
(아이템 제작에 초점을 맞춘 메이커 게임)</li>
<li>중개인으로 모험가(NPC)의 물건을 상인과 중개 판매하고 금액에 따라 NPC의 던전 탐사율이 올라가는 탐사 게임.<br>
(NPC의 탐사를 돕는 매니지먼트)</li>
</ol>
<br>    
1번째 방향성은 기획자님이
2번째 방향성은 내가 제시하였다.
<p>첫번째는 중고품 판매로 우산금지와 비슷하며.<br>
두번째는 매니지먼트와 제작이 혼합된 게임으로 비슷한 예시인  잭스미스에 가깝다고 볼 수 있다.</p>
<p><img src="https://github.com/rech4210/rech4210.github.io/assets/65288322/bd276768-4114-4f0e-800a-dcdf3ac5b101" alt="image"></p>
<blockquote>
<p>전당포가 주를 이루는 게임 우산금지</p>
</blockquote>
<p><img src="https://github.com/rech4210/rech4210.github.io/assets/65288322/a0f27e85-27a7-47fb-a710-8788788ef0a6" alt="image"></p>
<blockquote>
<p>NPC에게 제작한 무기를 건내주는 잭스미스</p>
</blockquote>
<br>
우리팀은 한 10분가량 서로 머리를 맞대고 어떤것이 더 좋을지 의논을 진행하였고 결과는 1번이 되었다.   
<br>  
<br>
<p>나는 개인적으로 2번을 주장하였는데<br>
가장 큰 이유는 우선순위 기준을 "재미"로 판단했기 때문이다.<br>
(매니지먼트가 혼합되면 더 재밌을것이라 생각해서였다)<br>
<br></p>
<p>허나, 의논을 거친 결과 게임잼 시간 안에 매니지먼트의 재미요소 부분까지 구현이 힘들것이라 판단하여 1번째를 선택하였다.</p>
<p>결국 시간안에 구현하는것이 현 상황에 가장 중요하니 말이다.<br><br>
그도 그럴것이 <strong>기획 개발 아트</strong> 가 각각 1:1:3으로 나 혼자 개발해야 할 상황이였기 때문…<br>
(물론 별개로 기획자님의 아이디어도 꽤 흥미로웠다)</p>
<p>그렇게 기획이 잡혀가고 즉시 개발을 시작했다.<br>
<br><br>
<br></p>
<p>코드를 작성하기 전 우선 구현 대상의 구조부터 생각했다.<br>
아이템 <strong>제작</strong>이 초점이기에 다루어야 할 데이터가 많은 것이다. 간단히 생각해보아도.</p>
<ol>
<li>아이템 이름</li>
<li>아이템 레시피 (재료)</li>
<li>아이템 판매가격</li>
<li>아이템 이미지</li>
<li>아이템 식별 코드<br>
<br></li>
</ol>
<p>등이 필요하니 말이다.<br>
그러니 요구사항을 정리해볼 필요가 있다.<br>
<br></p>
<ol>
<li>관리해야 할 데이터가 많다.
<blockquote>
<p>sJson 파일로 데이터를 관리하자</p>
</blockquote>
</li>
<li>접근이 용이해야한다.
<blockquote>
<p>개체에 접근하기 쉽도록 제너릭 싱글톤을 사용</p>
</blockquote>
</li>
<li>데이터 추가 및 수정이 편리해야한다.
<blockquote>
<p>Json 데이터 관리가 쉽도록 스크립트를 제어</p>
</blockquote>
</li>
<li>아이템의 확장성을 고려하여야 한다. (레시피 조합 때문)
<blockquote>
<p>아이템의 생성 로직을 분리하고 추상 팩토리를 사용</p>
</blockquote>
</li>
</ol>
<br>
<br>
<p>몇 가지 요구사항을 기반으로 작성을 시작했다.<br><br>
게임잼 중후반에 갈수록 코드가 엉망으로 쌓여 도저히 손도 대기 힘든 코드가 되어버린 경험이 있었기에, 더욱 핵심 구조는 철저히 잡기로 하였다.<br>
<br><br>
<br></p>
<p><img src="https://github.com/rech4210/rech4210.github.io/assets/65288322/ae103dc0-5b87-41df-acfd-ef5b677c8b42" alt="20230805_214442"></p>
<p>옆에선 아트님들이 단련된 근력으로 열심히 AI를 갈구시는 중이다.<br>
<br><br>
<br><br>
<img src="https://github.com/rech4210/rech4210.github.io/assets/65288322/235c5024-f5ba-4079-bfe9-4563573e4cf3" alt="20230804_205229"><br>
중간에 맛있는 피자도 먹고 ㅎㅎ,<br>
맛있게 잘 먹었습니다 PD님 ^- ^<br>
<br><br></p>
<p><img src="https://github.com/rech4210/rech4210.github.io/assets/65288322/f5384143-a84d-4db8-a741-f9a2d8be4d90" alt="20230805_214423"><br>
참고로 이번 게임잼에서는 개발 장비가 모두 세팅되어 있었다.</p>
<p>개인 노트북을 하나 가져왔는데 역시 개발은 장비빨이 중요하다 세삼스레 느낀다.<br>
<br><br>
다만 모듈 단위로 역할을 많이 분리해놔서 그런지 모니터 3개가 부족할 지경이였다.<br>
<br></p>
<p>데이터들의 관리는 DataManager를 싱글톤으로 만들어 관리하였다.<br>
하지만 데이터 관리에 미흡했던 탓에 꽤나 곤혹을 치르게 되었다.</p>
<pre class=" language-cs"><code class="prism  language-cs">{
            "code": 11,
            "price": 150,
            "honor": 2,
            "grade": 2,
            "name": "어둠의 창",
            "Image": "DarknessSpear",
            "description": "",
            "recipe": 
            {
                "recipeCode": 11,
                "ingredientArray": [
                    {
                        "code": 0,
                        "needCount": 1
                    },
                    {
                        "code": 1,
                        "needCount": 2
                    },
                    {
                        "code": 3,
                        "needCount": 1
                    }]
	        }
},
</code></pre>
<blockquote>
<p>무기 아이템의 데이터이다.</p>
</blockquote>
<p>무엇보다 아이템 조합식과 그에 맞는 재료들을 매칭시켜 비교하는 과정에서 데이터 관리가 꽤 힘들었다.<br>
마찬가지로 상인 / 재료 의 데이터도 모두 관리하기로 하였으니 가끔씩 생기는 배열의 인덱스 에러가 날 괴롭혔다…</p>
<pre class=" language-cs"><code class="prism  language-cs">&lt;Manager.cs&gt;
using UnityEngine;

public abstract class Manager&lt;T&gt; : MonoBehaviour where T : new()
{
    private static T _instance;
    
    public static T Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = FindObjectOfType(typeof(T)) as T;
                Debug.Log("인스턴스 생성");
            }
            return _instance;
        }
    }
}

</code></pre>
<p>관리할 것 들이 많아지다보니 위의 요구사항대로 제너릭 타입의 싱글톤을 사용하였는데, 크게 Item / Data / Game / Resource 으로 제너릭을 나누었다.<br>
<br><br>
DataManager에서 Json 데이터 파싱과 관리를 담당하고 Item에서는 생성된 아이템 (인벤토리 정보) 등을 저장하였다.</p>
<p>처음 구조를 설계할때는 데이터와 실제 아이템을 분리해야하는것이 옳다고 생각하였음에도, 막상 코드를 어느정도 작성하고 나니 무의식적으로 혼용해서 사용한것 같다.<br>
<br><br><br>
옆에서 개발을 하던분꼐서 하신 말씀이 생각난다.</p>
<blockquote>
<h4 id="연관관계와-데이터를-어떻게-관리할지-잘-구상하는것이-중요하다.">연관관계와 데이터를 어떻게 관리할지 잘 구상하는것이 중요하다.</h4>
</blockquote>
<p>이 말을 듣고 고개를 한 5번은 끄덕인 것 같다.<br>
격투게임과 마찬가지로 맞고나서야 실감하게 되는 것 … 아닐까?</p>
<p><br><br></p>
<p><a href="https://github.com/rech4210/rech4210.github.io/assets/65288322/8ac554df-4098-4fb0-a2e4-7cb7463dac42">바빠보이는 척…</a><br>
이것 저것 열심히 찾아다니는 모습<br>
<br></p>
<p><img src="https://github.com/rech4210/rech4210.github.io/assets/65288322/b89ea926-a1e4-4657-be07-dce26684a15a" alt="20230806_004219"></p>
<p>새벽까지 하다가 배가 고파 팀원들끼리 치킨을 먹었다.<br>
역시 밤새며 먹는 치킨은 치트키가 아닐까 싶을 정도의 맛. 😢<br>
<br><br>
<br><br></p>
<p><img src="https://github.com/rech4210/rech4210.github.io/assets/65288322/bfeb1d2f-3deb-4d60-8696-7eaa7c22a45f" alt="20230806_133024"><br>
코딩 주머니를 담당할 국밥 😄 😄 😄<br>
<br><br><br><br><br><br></p>
<p><img src="https://github.com/rech4210/rech4210.github.io/assets/65288322/0b3a902f-5394-411f-8131-2b3c1623a5dd" alt="20230806_160440"><br>
마지막날에는 AI 멘토분이 오셔서 생성형 인공지능에 대해 설명해주셨다.</p>
<br>
예상외의 부분이 네이버 TTS / 음성생성 기술이 전세계적인 수준이라고...
캐릭터마다 음성을 생성해서 적용시킬 수 있다면 인디게임의 저변도 강력해지지 않을까? 하는 생각이 들었다.  
<p>AI의 상용화가 이미 이루어지고 있고 특히 SD 모델의 성과가 계속 보이는걸로 보아 매우 기대가 된다.<br>
<br><br><br><br></p>
<p>이번 게임잼은 내게 꽤 큰 의미가 되지않았나 싶다.<br>
노벨AI가 유출되고 난 후 잠깐 다뤄보았지만 그때 당시에는 모델의 부족함이 너무 많았다. (인체의 뒤틀림, 손가락의 기형 등)</p>
<p>하지만 현재 Gen2의 기술이라던지, MJ - gen2 runaway를 사용한 실시간 비디오 기술은 정말 놀라울 정도다…<br>
이번 기회를 통해 AI 리소스를 활용하는 방법을 고안해봐야겠다.</p>
<br>
<br>
<p>아래는 게임잼 기간동안 제작한 게임의 몇몇 스크린샷을 가져왔다.<br>
현재 빌드에 약간 문제가 생겨 게임 플레이 영상은 추후에 추가하도록 해야겠다.</p>
<p><a href="https://user-images.githubusercontent.com/65288322/259400598-9e735c83-6523-463c-b51b-fb6406530bac.mp4">에디터 영상</a></p>
<p><img src="https://github.com/rech4210/rech4210.github.io/assets/65288322/a2cdcd22-2475-4f7c-afc8-74c5631c828e" alt="2023-08-08 08;11;37"></p>
<p><img src="https://github.com/rech4210/rech4210.github.io/assets/65288322/ee500c1d-05e0-4b13-b4ac-6cf6c8baaba5" alt="2023-08-08 08;12;15"></p>
<p><img src="https://github.com/rech4210/rech4210.github.io/assets/65288322/a00c77b1-490d-4886-8978-e078c46290a3" alt="2023-08-08 08;11;49"></p>

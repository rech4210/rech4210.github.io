---
layout: post
title: 버닝비버 게임잼
tags : Game Jam
description:
---
## 11월 25일 ~ 11월 27일
### 장소 영등포 올댓마인드


처음으로 참가해보는 오프라인 게임잼으로 온라인까지 합치면 이번이 두번째.<br>
입구에서부터 엄청나게 많은 개발자들이 있다..<br>
![게임잼 입장사진](https://user-images.githubusercontent.com/65288322/208680075-3c990a89-73be-47ec-a8a1-e5ef1700b3b5.jpg)<br><br>
시작하기 전 간단한 아이스 브레이킹과 이번 게임잼의 주제에 대해 얘기한 후 자유롭게 팀 빌딩하는 시간을 가졌다.
<br><br><br>
내가 선택했던 팀은 버닝비버를 주제로 한 땅따먹기 게임을 만들기로 했는데, 이 팀이 정말 황금 밸런스였다.<br>
꿈에 그리던 3프밍 1아트 1기획 삼위일체의 조합.. 정말 든든했다. <br><br>
한편 걱정도 좀 됐는데, 게임의 메인 로직을 담당하고 싶다고 호기롭게 선언했지만.<br>
만약 내가 잘못하면 게임이 엎어질게 뻔하기 때문에 꽤 두려웠다. 하지만 그런 부분을 이해해주시고 "이번이 아니면 언제 해보겠어?" 라고 다들 말해주셨을때 정말 감동 그 자체였지... <br>
그렇게 나는 게임의 메인 로직인 타일 생성과 키입력으로 변화하는 과정을 맡았다.<br>

특히 나를 제외한 개발자 두 분이 너무 잘하셔서 이번에 정말 많이 배운 기회가 된 것 같다.<br>
평소 쉐이더와 개발에 관심이 많았던 나에게 두 분은 최고의 선생님이 아니었나 싶다. 정말로<br>
<br>
<br>
![핫식스](https://user-images.githubusercontent.com/65288322/208680081-8065f3a1-1777-4202-a34b-8582ffba7006.jpg)<br><br>
먹거리는 물론 다양한 부식류와 개발자에겐 없어선 안될 몬스터가 구비되어있다.<br>
저 많이 쌓여있는게 사실 한 번 이미 리필 된 양이라는것...<br>
평소 에너지드링크를 마시는 편은 아닌데 이번 게임잼에서 1년치를 몰아 마신것 같다 :smile: <br>
<br>


![개발환경](https://user-images.githubusercontent.com/65288322/208680066-65524373-371d-45ab-9e68-b1feb7e17e60.jpg)<br>
<br>
▲ 게임잼 당시 내가 사용했던 컴퓨터와 모니터 사진<br>

설마 나 이외에도 본체를 가져온 사람이 있을까 싶었는데 역시나 나 뿐이더라 ㅎ..<br><br>
개발을 시작하기전에 앞서 첫 날 인터넷 환경이 고르지않아 개발에 약간 차질이 있었으나, 그래도 생각보다 스토브 측에서 개발환경과 편의에 대해 많이 생각해준듯 하다. 너무 감사할 따름 :clap: :clap:<br>

[개발 영상](https://user-images.githubusercontent.com/65288322/208913378-2beb96ab-8499-4281-a0bc-ba0f44ee863c.mp4)<br><br>
1차 프로토타입을 영상으로 담아보았다.<br>
게임의 기본적인 뼈대만을 완성한 프로토타입으로 타일 위치 설정 및 동적 생성, 랜덤 타일 변화, 랜덤 알파벳 생성등이 추가되었다.<br>
<br>
보기와는 다르게 꽤나 로직이 복잡했는데, 기본적으로 변화하지 않은(땅 색깔) '랜덤'의 타일을 가져오기 때문에 랜덤 타일을 hit 했는지 miss 했는지에 따라 오류가 계속 생겨 골치거리 였다.<br>

이를 해결한건 생각보다 간단했다.<br>
바로 2차원 리스트에 요소들을 모두 담은 후, 두 개의 1차원 리스트 List_A, List_B를 만들었다.<br>

순서는 이러했다.<br>
1. 모든 타일 요소들을 동적으로 생성하고 시작시 2차원 리스트에 저장해두는 컨테이너를 만든다.
1. 컨테이너의 모든 타일들을 변하기 전 List_A에 넣는다. 그 후 List_A 내부 요소 타일들을 랜덤 변화시킨다.
1. 사용자가 타일을 누르면 그 타일을 변화된 List_B에 저장한다. ListA 를 모두 탐색후 변화 할 타일이 없는 경우 잠시 대기한다.

```cs
public void GenerateMap(int width, int height)
   {
       tileContainer =  new Tile[width,height];

       float deafultGap = (frameSize/mapWidth)* ((width * 0.5f)- 0.5f);

       for (int i = 0; i < height; i++)
       {
           for (int j = 0; j < width; j++)
           {
               var a = Instantiate(prefabTile, transform);
               unsetedTileList.Add(a.GetComponent<Tile>());

               a.transform.GetChild(0).GetComponent<SpriteRenderer>().sortingOrder = -i - 1;

               a.transform.position = new Vector3(frameSize/(float)mapWidth * j- deafultGap, frameSize/(float)mapHeight * i- deafultGap+0.5f, 5.0f);
               a.transform.localScale = new Vector3((frameSize / (float)mapWidth) * setFrameSizeFloat, (frameSize / (float)mapHeight) * setFrameSizeFloat, 0);
               // 포지션과 사이즈 설정
               tileContainer[i, j] = a.GetComponent<Tile>();
           }
       }
   }
```

이렇게 타일이 랜덤하게 선택될경우 miss 되는 경우의 수를 제거했다.!<br>  
이 외에도 키 입력과 타일간의 키매핑, 캐릭터간의 타일을 다르게 출력하는 설정도 포함하여 작성하였다.

```cs
public void drawTargetAlphabetOrFrame()
    {
        TileHotKey = (char)Random.Range(97, 123);
        int offset = TileHotKey - 97;
        currentKey = (KeyCode)((int)KeyCode.A + offset); // 대응 키, 핫키 설정
        spwanedTime = Time.timeSinceLevelLoad;
        spwaningTime = MapGenerater.S.GetRandomSpawningTime();
        switch (tileType)
        {
            case "beaver":
                // 격자의 알파값 설정
                tileHotkeyTextMesh.gameObject.SetActive(false);
                tileFrame.gameObject.SetActive(true);
                break;
            case "neutrality":
                //격자, 텍스트 알파값 설정
                tileHotkeyTextMesh.gameObject.SetActive(true);
                tileFrame.gameObject.SetActive(true);
                break;
            case "human":
                tileHotkeyTextMesh.gameObject.SetActive(true);
                tileFrame.gameObject.SetActive(false);
                tileHotkeyTextMesh.text = tileHotKey.ToString().ToUpper();
                // 텍스트 알파값 설정.
                break;
            case "desert":
                // 황무지로 설정
                tileHotkeyTextMesh.gameObject.SetActive(false);
                tileFrame.gameObject.SetActive(false);
                break;
            default:
                break;
        }

        u = 0;

    }
/--------------------------------------------------------------------------------/
public void SetTile(char tileHotKey, KeyCode currentKey, Sprite tileCurrentSprite, float spwaningTime,string tileType)
    {
        u = 0;
        spwanedTime = Time.timeSinceLevelLoad;
        isSetTiled = false;
        TileHotKey = tileHotKey;
        this.tileCurrentSprite = tileCurrentSprite;
        this.tileTargetSprite.sprite = tileCurrentSprite;
        this.spwaningTime = spwaningTime;
        this.tileType = tileType;
        this.currentKey = currentKey;
    }
```  
지금 생각해보니 타일을 설정하는 함수에 인자가 너무 많이 들어가있다..  
책임을 분리하여 기능별로 함수를 만들어두는게 더 낫지 않았을까 싶다.  

![치킨](https://user-images.githubusercontent.com/65288322/208680055-9dfd0c3b-7446-4d52-89f5-f38faf53e5b2.jpg)<br>
간간히 야식도 챙겨주셨다. 치킨 너무 맛있었어요!!! <br><br><br><br><br><br>

[개발 영상2](https://user-images.githubusercontent.com/65288322/208913489-5341de77-c4d7-4195-9492-750a57c037de.mp4)<br><br>
어느정도 뼈대가 잡히고 아트적인 부분도 추가 된 영상이다.<br>
확실히 게임 느낌이 사는듯. 아트의 위대함을 다시금 느껴본다.<br><br>
1차 영상과는 다르게 활성화된 타일을 상대방이 누르면 황무지로 변해 클릭이 안되도록 바뀌고, 일정 시간이 지나면 활성화 되도록 기획이 바뀌었다. <br>

기존 타일 생성과 로직 자체는 동일하니 변경 자체는 매우 쉬웠다!<br>



![중식](https://user-images.githubusercontent.com/65288322/208680087-5e9d6710-99d6-4d91-8bfb-493312aa503a.jpg)<br>
그렇게.. 개발의 시간을 지나 마지막 날 마무리 후 중식을 먹었다.<br>
게임잼 2박3일 동안 거의 기름진 음식만 먹은 것 같은데, 이건 분명 누군가의 취향이 반영된것 같다<br>

![망가진 패드](https://user-images.githubusercontent.com/65288322/208680090-d0b3cc1a-5097-41dd-b034-9ddf61b3260a.jpg)<br><br>
게임잼 기간동안 나의 마우스 패드가 되어준 박스조각에게 감사를 표한다 :cry:
<br>
<br>
사실 이번 게임잼에 참가함에 있어서 너무 생각없이 행동한게 아닌가 싶었다.<br>
왜냐면 나에겐 이동형 개발장비가 없었기 때문.<br>
<br>
결국 내가 생각해낸 방법은 본체와 모니터를 들고 대구 ↔ 서울을 왕복하는 방법을 선택 했는데,<br>
돌이켜보면 정말 무모했던 짓인것 같다. 40kg이 되는 짐을 가방과 함께 계속 끌고 다녔으니..<br>

하지만 되려 생각해보면 이번 게임잼 참가가 나에게 있어 큰 반환점이 된 듯하다.<br>
인디게임 개발자들과 친목도 다지며 내 실력이 어느정도인지 가늠할 수 있는 최고의 기회가 아니었나 싶다.<br>
다음에 또 다시 참여하고 싶을 정도로!<br>
<br>
<br>
![stoveIndie](https://user-images.githubusercontent.com/65288322/208687058-da88121d-8cf3-4dc1-920e-3f0acf75f0ca.png)<br>
<br>
스마게 인디게임에 우리팀이 제작한 게임이 올라갔다!!!!
다만 댓글 반응을 보아 2인게임이라 플레이하기엔 약간 제한이 있나 보다
<br>
추가로 게임잼때 공부한 내용을 따로 TIL에 정리해두었다. 미래의 나에게 꼭 도움이 되길 ^-^ <br>
[비버 게임잼 공부 기록](../TIL/2022/11.html)

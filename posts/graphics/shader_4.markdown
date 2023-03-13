---
layout: post
title:  "HLSL 프로그래밍_2"
date:   2023-03-13 10:30:10 +0900
categories: shader
tags : Shader, Graphic
---
## blinn phong <br>
![image](https://user-images.githubusercontent.com/65288322/224728101-93d2245a-e591-47b8-ab97-436b9c78eda3.png)<br>
두 요소를 합성하는 방식의 쉐이더 모델로
1. 빛을 향하는 물체의 측면을 비추는 확산 조명을 계산한다.
2. 오브젝트에 빛 또는 하이라이트의 specular 조명을 연산해 생기 부여

```cs
#if UNITY_VERSION >= 202120
        return UniversalFragmentBlinnPhong(lightingInput,surfaceInput);
    #else
        return UniversalFragmentBlinnPhong(lightingInput,surfaceInput.albedo
        ,float4(surfaceInput.specular,1),surfaceInput.smoothness,0,surfaceInput.alpha);
```

위 내용은 UniversalFragmentBlinnPhong 함수로 블린 퐁 라이트 연산 모델을 수행한다.<br> ShaderLibrary/Lighting.hlsl 패키지에 포함되어있음.

위를 보면 #if로 되어있는데 이는 해시태그로 유니티의 버전에 따라 다른 쉐이더 형식이 작동되도록 짠것이다.

<br>
여기 들어가는 인자는 아래와 같다.<br>
```cs
InputData lightingInput = (InputData)0;
// 현재 fragment에서 메쉬의 위치와 방향에 대한 정보를 저장
SurfaceData surfaceInput = (SurfaceData)0;
// material의 물리적 속성들을 가지고 있음.

struct SurfaceData {
    half3 albedo;
    half3 specular;
    half  metallic;
    half  smoothness;
    half3 normalTS;
    half3 emission;
    half  occlusion;
    half  alpha;

	// And added in v10 :
    half  clearCoatMask;
    half  clearCoatSmoothness;
};

struct InputData {
    float3  positionWS;
    half3   normalWS;
    half3   viewDirectionWS;
    float4  shadowCoord;
    half    fogCoord;
    half3   vertexLighting;
    half3   bakedGI;
    float2  normalizedScreenSpaceUV;
    half4   shadowMask;
};
```

<br>
<br>
빛의 연산을 위한 normal 추가<br>
```cs
struct Attribute
{
    float3 positionOS: POSITION;
    float3 normalOS:NORMAL; // 라이트 연산에 필요한 노말
    float2 uv : TEXCOORD0;
};
struct Varyings
{
    float4 posCS: SV_POSITION;
    float3 normalWS: TEXCOORD1; //TEXCOORD
태그를 쓰는 이유는 래스터라이저에서 TEXCOORD로 태그된 모든 필드를 보간하기 떄문임.
    float2 uv : TEXCOORD0;
};
```

정점의 노말이 정점 조립 단계에서 추출되어 Attribute로 들어온다.<br>
들어온 노말은 normalOS로 vertex funtion에서 처리된다.<br>

<br>
![image](https://user-images.githubusercontent.com/65288322/223446968-b27c361c-d84b-4b53-bf19-6f1e13310c0f.png)<br><br>

좌표공간을 변경한 노말은 래스터라이저의 대상이 되어 보간이 진행된다.<br>
여기서 서로 반대 방향을 가진 노말의 사이를 보간 진행될 때, 아래 그림처럼 vector의 값이 변경된다.<br>
![image](https://user-images.githubusercontent.com/65288322/223447305-f87e94bd-6162-4e33-bbda-f0894726b6fb.png)<br><br>

라이트 연산에서는 조명의 쿼리티를 위해 모든 벡터를 정규 벡터 (길이가 1)로 만들어주는 노말라이즈 과정을 거쳐 부드러운 라이트를 만들어 내 준다 (제곱근 계산이 있어 꽤 무거운 연산이라고 한다)<br>
![image](https://user-images.githubusercontent.com/65288322/223447943-c6e6f8b2-bfb2-49ec-98d0-0ad255dd43c9.png)<br><br>
정규화가 진행된 벡터

specular<br>

: 광원을 직접적으로 반사하면서 발생하는 표면의 하이라이트를 뜻한다.<br> 즉 강도가 높을수록 빛을 강하게 반사한다.<br> 맵 중에서 gloss map 이란게 있는데, 검은색은 0 흰색은 1인 alpha 맵으로 1에 가까울수록 반사 강도가 높아진다
![image](https://user-images.githubusercontent.com/65288322/224002360-5181fa13-d26c-40d0-a0d8-2723235df56e.png)<br>

<br>
smoothness
: 표면의 부드러움을 나타내며 0에 가까울수록 거친 표면이 된다.<br> 이 값은 specular reflect의 산란 방식에 영향을 미친다.<br> 쉽게 말해 하이라이트가 얼마나 집중적으로 나타날지, 퍼져나갈지를 조절하는 값이다.<br> 즉 값이 1에 가까우면 거울같은 (정반사) 효과가 난다는 것.
![image](https://user-images.githubusercontent.com/65288322/224002833-3a08e235-7dc3-4a73-aa2d-29437fe1e288.png)

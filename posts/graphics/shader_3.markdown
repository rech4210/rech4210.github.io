---
layout: post
title:  "HLSL 프로그래밍_1"
date:   2023-03-13 10:30:10 +0900
categories: shader
tags : Shader, Graphic
---
렌더링 파이프라인 단계에서 정점 조립은 프로그래밍 불가 영역이다.

![image](https://user-images.githubusercontent.com/65288322/224725633-0843530d-ef8d-4e1b-ac28-7c169ab2ed93.png)<br><br>
이 단계에서 각 정점의 데이터를 정점 구조에 Semantics을 통해 위치, 노말, UV등의 정보를 가져온다.
<br> (POSITION, SV_POSITION, SV_TARGET, TEXCOORD0)


### struct **VertexPositionInputs**<br>
![image](https://user-images.githubusercontent.com/65288322/224723960-462d076d-07af-4999-a5bc-ac320c360961.png)<br>
=> 미리 정의된 구조체로 positionWS, VS, CS ,NDS등 좌표에 대한 정보를 담은 구조체로 사용된다.<br>
각각 월드, 뷰, 클립, NDS로 처리된다.<br>
버텍스 쉐이더에서는 클립 과정까지 처리한다.

### 버텍스 쉐이더의 역할
1. 행렬 연산을 통한 정점 위치 변환
1. 입력된 정점에 대한 텍스처 위치 계산 (텍스처 매핑에 사용)
1. 각 정점에 대한 조명 연산 수행
1. 각 정점에 대한 색상 계산 (색칠이 아님)

## 버텍스 구조체
```cs
struct Varyings
{
    float4 posCS: SV_POSITION; // CS의 버텍스 위치를 포함하고 있다는 Syntax
    float2 uv : TEXCOORD0;
};
```

<br>
일반적으로 프로퍼티를 통해 인스펙터에서 사용자의 값을 받아올 수 있다.<br>
```cs
 Properties
    {
        [MainTexture] _MainTex ("map here", 2D) = "white" {}
        [MainColor]_BaseColor ("Here",Color) =(1,1,1,1)
    }
```
<br>

물론 이 값이 쉐이더로 바로 전달되는건 아니고, 쉐이더에서 따로 유니폼 변수로 만들어줘서 넣어줘야한다.<br>

```cs
float4 _BaseColor;
TEXTURE2D(_MainTex);
SAMPLER(sampler_MainTex);
// 텍스쳐를 불러올지에 대한 매크로 왼쪽은 샘플러 오른쪽을 텍스쳐
float4 _MainTex_ST;
// ST 타입에 유니티가 자동적으로 값을 저장해줌

#define MACRO(argument) argument + 2 // 매크로
```

<br>
<br>
기존 유니티에서는 샘플러와 텍스쳐를 동시에 불러왔다
```cs
Sampler2D _MyTexture;
float4 _MyTexture_ST;

float4 color = tex2D(_MyTexture,uv);
```
이런 방식이였다면

HLSL에서는

```cs
TEXTURE_2D(_MyTexture);
SAMPLER(sampler_MyTexture);

float4 _MyTexture_ST
```
이런 식으로 텍스쳐를 가져올 수 있다.<br>
샘플러를 여러개 설정하고 텍스처 또한 하나의 쉐이더에서 여러개를 쓸수 있기에 장점.

<br>
<br>
찾아보니 이  **TEXTURE_2D** 매크로는 GPU칩셋에 텍스처와 샘플링 변수를 선언해주는 메크로라고 한다.
<br>
<br>
<br>
## 프래그먼트 쉐이더
![image](https://user-images.githubusercontent.com/65288322/224718144-9766ed50-6c17-4d6f-81d6-af6aa05cb1f9.png)
마지막으로 프래그먼트 쉐이더에서 **SAMPLE_TEXTURE2D** 매크로를 사용해주면 된다.
<br>
```cs
float4 colorSample = SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.uv);
//인자는 각각 텍스쳐 이름, 샘플러 이름, UV 이다
```

---
layout: post
title:  "HLSL 프로그래밍 Wave 구현해보기"
date:   2023-05-28 13:45:10 +0900
categories: shader
tags : Shader, Graphic
---
## 오늘의 주제

이번 과제는 전에 배웠던 개념들을 모두 혼합하여 사용하고자 Gestner Wave를 사용한 바다를 만들어보기로 하였다.  


## [제작된 Shader 보기](https://www.youtube.com/watch?v=XL_UecVhJT4)

## 구현해야 할 부분은?  
1. 파도의 움직임을 구현하기위해 sin, cos을 통해 position 값을 제어해주기
> Gestner 웨이브 유사 공식을 사용할 예정이다.  
1. UV Scroll / Gradient 활용하여 파도의 색깔 값 제어하기
1. UniversalFragmentPBR 함수를 사용하여 BDRF 구현하기
1. Normal Map을 활용하여 파도의 표면 부분 변경 및 라이트 반영하기
1. 왜곡효과 주기

전체 내용을 기술할경우 너무 길어질듯하니 중요한 부분만 기술하도록 하겠다.  
## **전체 코드를 바로 보고싶다면 맨 아래로 내리시면 되겠다.**



## 0. Shader 파일 만들기
우선 가장 기본적인 shader 파일을 만들어 주자  
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/0420f500-b082-4276-a3da-10d9912a05a3)  
HLSLPROGRAM부터 ENDHLSL 사이에 쉐이더 코드들이 들어가니 정의된 명령어를 제외하곤 모두 안에 쓰면 된다.  
<br><br><br><br><br><br><br><br>  

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/94752079-1a99-4576-b74e-c047115e57d9)  
마찬가지로 hlsl 파일도 만들어 기본적인 내용을 정의해주자  
vertex 함수의 내부를 보면 VertexPositionInputs, VertexNormalInputs 두 구조체가 있는데 이 둘은 URP 내부에 정의된 구조체로 PBR 함수의 필수요소이다.  
정점의 포지션을 담을 VertexPositionInputs, Tangent Coordinate를 담을 VertexNormalInputs를 선언하고 정의해주자.  

각 구조체에 들어갈 값은 GetVertex....() 함수를 통해 구할 수 있다.  

```cs  
Varyings vert(Attributes input_Att)
{
    Varyings v_output;

    //정점의 포지션을 담을 VertexPositionInputs
    VertexPositionInputs posinput = GetVertexPositionInputs(input_Att.positionOS);
    v_output.posCS = posinput.positionCS;
    v_output.posWS = posinput.positionWS;

    //Tangent Coordinate를 담을 VertexNormalInputs
    VertexNormalInputs normalInput = GetVertexNormalInputs(input_Att.normalOS, input_Att.tangentOS);
    v_output.normalWS = normalInput.normalWS;
    v_output.tangentWS= float4(normalInput.tangentWS,input_Att.tangentOS.w);
    v_output.bitangentWS = normalInput.bitangentWS;
}
```  

쉐이더 내부의 함수와 프로퍼티를 사용하기위해 패키지 include 하는것을 잊지 말자.  
```cs
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"  
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"  
```  
간단한 내용이니 전혀 걱정할 필요없다.  

## 1. sin, cos 함수를 이용하여 wave 만들기
이제 삼각함수를 이용하여 wave를 구현해볼것이다.  
두가지 sin, cos 값을 이용하여 웨이브를 구현할것인데, 간단히 정점의 position값에 삼각함수의 값을 곱해주는 것이다.  
![Forces_in_Trochoidal_wave](https://github.com/rech4210/rech4210.github.io/assets/65288322/4ce441d1-62c8-4bb0-ac64-812788210898)  
> 이처럼 삼각함수의 파형을 wave로 변환한다고 생각하면 된다.  

아래 코드에는 sin, cos 함수의 값을 vertex의 uv.x, uv.y 의 값으로 곱하여 위치를 변경한다.  
```cs
v_output.posCS.x += sin(v_output.uv.x+ _Time.w);
v_output.posCS.y += cos(v_output.uv.y+ _Time.w);
v_output.posCS.y += cos(v_output.uv.x+ _Time.w);
```
순서대로 실행해 보면.  
![2023-05-29 22;52;09](https://github.com/rech4210/rech4210.github.io/assets/65288322/930432b0-be2c-4a1f-9ff2-96139be460cc)  
uv의 값을 기준으로 파형을 그리며 위치값이 변경된다.  
잘 적용 된 모습인데, 그대로만 있으면 wave 값을 조절하기가 불편할것이다.
여기에 property를 추가하여 값을 조절해보자.  

1. shader 파일에 프로퍼티 인터페이스를 선언해준다.
1. hlsl 파일에 프로퍼티를 정의해준다.

```cs
Shader ""
{
  Properties
  {
    waveTexture("Wave Texture", 2D) = "white"{}
    waveValue("Wave uv Value",Range(0.0,10.0)) = 0
    waveDuration("Wave Duration",Range(0,10)) = 0
    waveAmplitude("Amplitude",Range(0.0,10.0)) = 0
    waveSurfacePow("surface pow", Range(0,10)) = 0
  }
}

/---------------------------------------------------/
//hlsl file

struct Attributes
{
  //Atrributes
};

struct Varyings
{
  //Varyings
};

// 많이 사용되는 데이터들을 담아두는 컨테이너로 상수 버퍼를 사용하는 매크로이다.
CBUFFER_START(UnityPerMaterial)
  float waveValue;
  float waveDuration;
  float waveSurfacePow;
  float waveSurfaceDistort;
  float waveAmplitude;
CBUFFER_END

Varyings vert(Attributes input_Att)
{
    //--------------------------  Wave  -----------------------------------//
    v_output.posCS.x += waveAmplitude * sin(v_output.uv.x * waveValue  + _Time.w * waveDuration );
    v_output.posCS.y += waveAmplitude * cos(v_output.uv.y * waveValue + _Time.w * waveDuration *1.3);
    v_output.posCS.y += waveAmplitude * cos(v_output.uv.x * waveValue + _Time.w * waveDuration);

    return v_output;
}
```  

삼각함수 내부에 들어간 속성중 waveValue는 uv tilling, _Time.w * waveDuration은 진동횟수 마지막으로 waveAmplitude는 진폭을 조절하기 위해 추가하였다.  
![2023-05-29 22;57;31](https://github.com/rech4210/rech4210.github.io/assets/65288322/9cd54198-33ea-4e36-bdf1-fb05fcdbe59b)  
property들을 통해 원하는 값을 제어할 수 있게 되었다.    

---------
<br><br><br>


## 2. UV Scroll  
이번엔 UV 스크롤와 Gradient를 활용하여 파도의 색을 입힐것이다.  
1. UV를 스크롤 하기위해 vertex 함수에 _Time.y 값을 곱한다.  
1. 프로퍼티를 정의한다.  
1. frgament 함수에서 output 변수를 만들어 SAMPLE_TEXTURE2D 함수의 반환형을 받는다  

```cs  

struct Attributes
{
  //Atrributes
};

struct Varyings
{
  //Varyings
};

// 텍스쳐 타입의 변수 정의
TEXTURE2D(waveTexture);
// 샘플링 방식 정의
SAMPLER(sampler_waveTexture);

CBUFFER_START(UnityPerMaterial)
// float waveValue;
// float waveDuration;
// float waveSurfacePow;
// float waveSurfaceDistort;
// float waveAmplitude;

// 샘플링 된 텍스쳐 좌표 및 크기의 정보를 담는 변수이다.
float4 waveTexture_ST;
float4 MainColor;

CBUFFER_END

Varyings vert(Attributes input_Att)
{
    // 각 uv 방향으로 시간 만큼 UV를 스크롤한다.
    v_output.uv.y += _Time.y *0.25;
    v_output.uv.x += _Time.y *0.15;
    return v_output;
}

half4 frag(Varyings input_Vry) : SV_TARGET
{
     float4 output;
     // SAMPLE_TEXTURE2D는 주어진 텍스쳐와 샘플링 방식을 통해 UV를 기준으로 텍스쳐의 값을 가져오는 함수이다.
     output = SAMPLE_TEXTURE2D(waveTexture,sampler_waveTexture, input_Vry.uv) * MainColor;
     return output;
}
```

![2023-05-30 16;50;01](https://github.com/rech4210/rech4210.github.io/assets/65288322/e2261145-af4c-450f-919a-5bb847bfec12)  

무사히 텍스쳐가 적용되었다.  

하지만 우리가 원하는것은 검은 표면이 위로 오는것이 아니다.  
이를 해결하기위해 바다의 물결 부분은 1 (밝게)에 가깝도록 만들고 아닌 부분은 0 (어둡게)만들 필요가 있다.  

그렇다면 어떻게 해야할까?  

### 제곱과 반전
고민해본 결과, 물결이 치는 부분과 아닌 부분의 대비를 강하게 주면 될 것이라 생각한다.  
1. 텍스처의 값을 제곱(pow)하여 대비를 강하게 주자. 사용할 값은 waveSurfacePow로 미리 선언해두었다.  
1. 여기에 1-output 으로 밝은 부분과 어두운 부분의 값을 서로 뒤집어주자.  

```cs  
half4 frag(Varyings input_Vry) : SV_TARGET
{
     // float4 output;
     // // SAMPLE_TEXTURE2D는 주어진 텍스쳐와 샘플링 방식을 통해 UV를 기준으로 텍스쳐의 값을 가져오는 함수이다.
     // output = SAMPLE_TEXTURE2D(waveTexture,sampler_waveTexture, input_Vry.uv) * MainColor;
     output = pow(output, waveSurfacePow);
     output.rgb = 0.1+(1- output.rgb);
     return output;
}
```  
![2023-05-30 17;14;35](https://github.com/rech4210/rech4210.github.io/assets/65288322/eb855bbf-8ec7-4545-a6b7-e0f176cdcf66)  
> waveSurfacePow의 값을 이동시켜보면 확연하게 차이를 확인할 수 있다.  

음 잘 적용된 모습이다.  

## 3. Gradient 활용하여 파도의 색깔 값 제어하기    
다음으로는 Gradient를 입혀볼것이다.  
Gradient를 입히기 위해선 2가지 컬러, 시작지점 컬러와 끝지점 컬러가 필요하다.
그리고 이 둘을 lerp 시키면 그라데이션을 만들 수 있다.  

허나 문제점이 하나 발생했다.  
이유인 즉슨, lerp값으로 보간하면 uv를 기준으로 _Time 값이 계속 진행되기 때문에 UV의 스크롤이 멈추지 않고 결국 끝 지점 컬러로 모두 바뀌어버리는 것이다.  

```cs  
vert()
{
   //아래 코드를 활성화하면 바로 한 지점 값으로 고정된다.
   v_output.uv.x += _Time.y *0.15;
}
half4 fragment() ....  
{
  output += lerp(MainColor,DestColor,input_Vry.uv.x);
}
```  

![2023-05-30 17;31;01](https://github.com/rech4210/rech4210.github.io/assets/65288322/0f86db95-ec14-40bc-8081-1b1b32b9f959)  
>이 문제로 한동안 머리가 아팠다..  

이 문제를 어떻게 해결해야할지 고민하다가 생각해낸 두 가지 경우가 있었다.

1. if 조건문을 넣어 처리.
1. 기존 UV를 remap한 새로운 Gradient_UV를 만들어 사용하기.

이를 해결하고자 처음에는 if 조건문을 넣어 처리하였다.  


UV 값을 scroll 시킬 때 UV.x 또는 y 값이 일정 이상이 되면 초기화하는 방법으로 유지하는 방법을 생각하였는데, 해결되지 않았을 뿐 더러 if 조건문을 남용하는것은 gpu 내부에서 dynamic branching을 방해하고 작업을 무겁게 한다고 한다.  


다른 방법을 찾아보자.  
앞선 방법으로 고민하던 중 컬러와 scroll을 같이 연산하게 될 경우 결국 제어할 수 없다고 판단하여, 이를 분리하면 어떨까 생각하였다.  

간단히 말해 UV Scroll은 기존 UV 값을 사용하고 컬러는 UV를 remap 시켜 새로만든 Gradient_UV에 가져오는것이다.  

1. Gradient UV를 만들어 uv를 remap시켜 값을 가져온다.
1. Gradient UV의 x,y값에 각각 컬러를 넣고 이 둘을 합쳐준다.
1. output 결과값에 더해준다.


```cs
Varyings vert(Attributes input_Att)
{
    // lerp를 사용하지 않아도 되지만, 그라데이션 범위 변경을 고려하였다.
    v_output.GradientUV.x = lerp(0,1,v_output.uv.x);
    v_output.GradientUV.y = lerp(0,1,v_output.uv.y);
    // v_output.uv = v_output.uv * 1.5;
    return v_output;
}

half4 frag(Varyings input_Vry) : SV_TARGET
{
    //----------------------------  Main Color  --------------------------------------//
    // float4 output;
    // output = SAMPLE_TEXTURE2D(waveTexture,sampler_waveTexture, input_Vry.uv);
    // output= pow(output, waveSurfacePow);
    // output = 0.5- output;

    // uv를 따로 만들어 (remap하여) 컬러값을 더해줘 스크롤시의 문제점을 해결했다.
    float4 finalColor = lerp(MainColor,DestColor,input_Vry.GradientUV.x);
    float4 finalColor_2 = lerp(MainColor,DestColor,input_Vry.GradientUV.y);
    output +=  0.5 * (finalColor + finalColor_2);
}
```  

결과는...??  
<br><br>
![2023-05-30 17;43;16](https://github.com/rech4210/rech4210.github.io/assets/65288322/889e4dd6-5647-4700-9441-9aa14fb16b2b)  

잘 작동하는 모습이다.  
이제 UV scrolling과 관계없이 Gradient 값을 마음대로 바꿀 수 있다.  
UV, 컬러 작업을 마쳤으니 다음은 밋밋한 마테리얼을 손봐야할듯 하다.  
BDRF를 제공하는 PBR 마테리얼으로 출력하자.  

BDRF가 무엇인지 궁금하다면?  
[BDRF](https://en.wikipedia.org/wiki/Bidirectional_reflectance_distribution_function)  

----

## 4. UniversalFragmentPBR 함수를 사용하여 BDRF 구현하기  

URP에는 PBR 마테리얼을 다루는 함수 UniversalFragmentPBR가 있다.  
UniversalFragmentPBR 함수에는 두가지 인자값이 들어가는데, InputData와 SurfaceData 두 구조체 값이다.  
1. InputData는 Light 연산을 하기위한 정점의 데이터를 가진다.  
1. SurfaceData는 재질 표면에 맺히는 색, 투명도, 반사율등의 값을 가진다.

이걸 이용해서 라이팅 연산 및 노말맵을 다뤄볼것이다.  

```cs
Shader ""
{
    Properties
    {
        // waveTexture("Wave Texture", 2D) = "white"{}
        // waveValue("Wave uv Value",Range(0.0,10.0)) = 0
        // waveDuration("Wave Duration",Range(0,10)) = 0
        // waveAmplitude("Amplitude",Range(0.0,10.0)) = 0
        // waveSurfacePow("surface pow", Range(0,10)) = 0

        [Header(BRDF)]
        _SpecCular("Specular",Range(0,1)) = 0
        _Smoothness("Smooth", Range(0, 1)) = 0

        // MainColor("Start color",Color) = (1,1,1,1)
        // DestColor("Destination Color", Color) = (0.1,0.5,1,1)
    }
}

//------------------------------------------------------------//

half4 frag(Varyings input_Vry) : SV_TARGET
{
    // float4 finalColor = lerp(MainColor,DestColor,input_Vry.GradientUV.x);
    // float4 finalColor_2 = lerp(MainColor,DestColor,input_Vry.GradientUV.y);
    // output +=  0.5 * (finalColor + finalColor_2);

    InputData lightingInput = (InputData)0;
    lightingInput.normalWS = input_Vry.normalWS;
    lightingInput.positionWS = input_Vry.posWS;
    lightingInput.viewDirectionWS = GetWorldSpaceNormalizeViewDir(lightingInput.positionWS);
    lightingInput.shadowCoord = TransformWorldToShadowCoord(input_Vry.posWS);
    lightingInput.positionCS = input_Vry.posCS;
    lightingInput.tangentToWorld = 0;

    SurfaceData surfaceInput = (SurfaceData)0;
    surfaceInput.albedo = output.rgb;
    surfaceInput.alpha = output.a;
    surfaceInput.specular = _SpecCular;
    surfaceInput.smoothness = _Smoothness;
    surfaceInput.emission = 0;
    surfaceInput.metallic = 0;
    surfaceInput.normalTS = 0;

    return UniversalFragmentPBR(lightingInput, surfaceInput);
}
```

![2023-05-30 20;44;21](https://github.com/rech4210/rech4210.github.io/assets/65288322/9b445c6c-170f-4e17-911a-4962a52020e6)  

Smooth는 표면의 매끄러운 정도를 나타낸다.  
값을 0에서 1로 바꾸면 난반사 - 정반사의 변화를 나타낼 수 있는데, 1에 가까우면 유리와 같이 매끄러운 정반사 재질을 나타낼 수 있다.  

Specular는 반사율의 값인데 해당 함수에서는 1로 고정된다.  
Specular와 Smooth가 궁금하다면?  

[Blinn Phong, Specular](https://rech4210.github.io/posts/graphics/shader_4.html)  

이렇게 PBR 마테리얼을 적용해보았다.  
생각보다 너무 쉽지 않은가? :clap:

## 5. Normal Map을 활용하여 파도의 표면 부분 변경 및 라이트 반영하기
앞서 PBR 마테리얼을 적용함으로써 라이트 연산을 처리할수있게 되었다.  
하지만 과연 바다에서 저런 동그란 모양의 태양이 보일까?  
때론 물결에 부딪히거나 표면끼리 산란되며 모양이 달라질것이다.  

여기서 필요한것이 바로 표면의 값을 처리할 수 있는 Normal Map이다.  
노말맵은 법선 벡터의 값을 2D 맵으로 표현한것으로, 기존 정점의 값에 추가 정보를 주어 표면의 돌출을 나타낼 수 있는 방법중 하나이다.  

노말맵에 대해 궁금하다면?  
[Normal Map Tangent Coordinate](https://rech4210.github.io/posts/graphics/shader_10.html)  

자세한 내용은 위 링크를 참고하면 될 듯하니 이제 순서대로 노말맵을 적용해보자.  


1. 노말맵 샘플링 결과 값을 가져와 디코딩한다.
1. normal, tangent, bitangent 값을 계산하여 TBN 변환행렬을 구한다.
1. TBN 행렬을 이용하여 Tangent space의 normal vector를 world space로 변환한다.  
  - Tangent Space의 값을 World Space로 가져오는 이유는 Light 연산이 World Space를 기준으로 하기 때문이다.  
1. 변환한 world space 값을 InputData, SurfaceDatad에 대입한다.  


```cs
Shader ""
{
    Properties
    {
        // waveTexture("Wave Texture", 2D) = "white"{}
        // waveValue("Wave uv Value",Range(0.0,10.0)) = 0
        // waveDuration("Wave Duration",Range(0,10)) = 0
        // waveAmplitude("Amplitude",Range(0.0,10.0)) = 0
        // waveSurfacePow("surface pow", Range(0,10)) = 0

        // [Header(BRDF)]
        // _SpecCular("Specular",Range(0,1)) = 0
        // _Smoothness("Smooth", Range(0, 1)) = 0

        [Header(Normal property)]
        [NoScaleOffset][Normal]waveNormalTexture("Normal Map",2D) = "bump"{}
        NormalTextureValue("Normal Appear Value",Range(0,1)) = 0
        _NormalOffset_X("Normal X_Offset",Range(0,2)) =0
        _NormalOffset_Y("Normal Y_Offset",Range(0,2)) =0
        _NormalAddValue("Normal Add",Range(0,5)) = 0

        // MainColor("Start color",Color) = (1,1,1,1)
        // DestColor("Destination Color", Color) = (0.1,0.5,1,1)
    }
}

//------------------------------------------------------------//


half4 frag(Varyings input_Vry) : SV_TARGET
{

    //---------------------------  Normal Map  -------------------------------------//
    // 노말맵 샘플링
    float3 normalTS = UnpackNormalScale(SAMPLE_TEXTURE2D(waveNormalTexture, sampler_waveNormalTexture, input_Vry.normalUV),NormalTextureValue);
    // 노말맵 값의 범위를 remap 시킨다.
    normalTS = lerp(0.5,1, normalTS);
    // TBN 행렬 계산
    float3x3 tangentToWorld = CreateTangentToWorld(input_Vry.normalWS, input_Vry.tangentWS,input_Vry.tangentWS.w);
    //
    input_Vry.normalWS= normalize(TransformTangentToWorld(normalTS + _NormalAddValue , tangentToWorld));

    //----------------------------  Main Color  --------------------------------------//
    //Code....

    //----------------------------  Value  --------------------------------------//
    InputData lightingInput = (InputData)0;
    // lightingInput.normalWS = input_Vry.normalWS;
    // lightingInput.positionWS = input_Vry.posWS;
    // lightingInput.viewDirectionWS = GetWorldSpaceNormalizeViewDir(lightingInput.positionWS);
    // lightingInput.shadowCoord = TransformWorldToShadowCoord(input_Vry.posWS);
    // lightingInput.positionCS = input_Vry.posCS;
    lightingInput.tangentToWorld = tangentToWorld;

    SurfaceData surfaceInput = (SurfaceData)0;
    // surfaceInput.albedo = output.rgb;
    // surfaceInput.alpha = output.a;
    // surfaceInput.specular = _SpecCular;
    // surfaceInput.smoothness = _Smoothness;
    // surfaceInput.emission = 0;
    // surfaceInput.metallic = 0;
    surfaceInput.normalTS = normalTS;

    // return UniversalFragmentPBR(lightingInput, surfaceInput);
}
```  

![2023-05-31 00;07;16](https://github.com/rech4210/rech4210.github.io/assets/65288322/300abee5-9530-4f36-9d50-57199af6f42f)  
노말맵이 깔끔하게 적용 된 모습이다.  


다음으로는 투명도 값을 설정해보자.  
알다싶이 물은 투명하다, 그렇기에 alpha 값이 적용되는 transparent 개체로 바꿔야 한다.  

```cs  
Shader "Custom/wave_Shader"
{
    Properties
    {
        [Header(Wave property)]

        [Header(BRDF)]

        [Header(Normal property)]

        [Header(Colors)]
        AlphaColor("Alpha", Range(0,1)) = 1
        // MainColor("Start color",Color) = (1,1,1,1)
        // DestColor("Destination Color", Color) = (0.1,0.5,1,1)
    }
}

//---------------------------------  hlsl  -----------------------------------//

half4 frag(Varyings input_Vry) : SV_TARGET
{
    //---------------------------  Normal Map  -------------------------------------//
    //----------------------------  Main Color  --------------------------------------//

    // SurfaceData surfaceInput = (SurfaceData)0;
    // surfaceInput.albedo = output.rgb;
    // PBR 마테리얼에 알파값을 곱해줘 투명도를 만들어준다.
    surfaceInput.alpha = output.a * AlphaColor;
    // surfaceInput.specular = _SpecCular;
    // surfaceInput.smoothness = _Smoothness;
    // surfaceInput.emission = 0;
    // surfaceInput.metallic = 0;
    // surfaceInput.normalTS = normalTS;

    return UniversalFragmentPBR(lightingInput, surfaceInput);
}  
```  
코드 설정이 완료되었으면 Render Queue를 Transparent로 바꿔주자.  
Transparent는 렌더링 순서 Geometry의 뒷 단계로 반투명 오브젝트를 렌더하기위해 사용한다.  
![2023-06-05 15;53;25](https://github.com/rech4210/rech4210.github.io/assets/65288322/7bdee6eb-7a1e-403e-b2bc-bc78e5ff9611)  

간단히 정점을 순서대로 출력하고 겹치는 정점 부분을 Blend 하는 것이다.  
자세히 알고싶다면? 아래를 참고하길 바란다.    
[Blend](https://rech4210.github.io/posts/graphics/shader_7.html)  




## 6. 왜곡효과 주기  

거의 다 완성되었다.  
하지만 중요한점이 있다.  
파도가 변칙적이지 않고 똑같이 반복만된다면 어색할 것 같다.  
마침 최근 Radial shear를 배워봤으니 이를 활용해보자.  
방사형 전단(radial shader)는 방사형으로 나타나는 이미지 왜곡 기법이다.

1. 전단이 수행될 중심점을 선택한다.
1. 각 픽셀과 중심점과의 거리를 계산한다.
1. 강도를 선택한다.

```cs  
Varyings vert(Attributes input_Att)
{
    // VARYINGS V_OUTPUT;

    // //--------------------------  GRADIENT UV -----------------------------------//

    // //--------------------------  WAVE  -----------------------------------//


   //---------------------------- Shear  ----------------------------------//
    float2 delta = v_output.uv - _ShearCenter;
    float delta2 = dot(delta.xy, delta.xy);
    // 두 벡터 dot 연산
    float2 delta_offset = delta2 * _ShearStrength * (_CosTime.w *0.1); // 휘어짐
    float2 shear = v_output.uv
    + (float2(delta.y, -delta.x) * delta_offset) + (_ShearOffset); //Shear Offset은 uv 스크롤

    v_output.uv = shear ;

    //---------------------------- UV Scroll  ----------------------------------//
}
```  
![2023-06-05 16;01;14](https://github.com/rech4210/rech4210.github.io/assets/65288322/0faf5f28-1c4a-4c13-ad65-f172603dd5a8)  

이렇게해서 대략적으로 완성된듯 한데, 교차지점에 하이라이트를 주는 쉐이더를 추가로 제작해보려 한다.  
Intersection 쉐이더는 이후 포스팅에서 다뤄보도록 하겠다.  


## wave Shader  
![wave shader](https://github.com/rech4210/rech4210.github.io/assets/65288322/fd8a1817-0b5b-40fa-b6e7-35cf255187da)  

## wave Shader.hlsl  
![wave hlsl code](https://github.com/rech4210/rech4210.github.io/assets/65288322/93c7086b-253b-4878-969f-67beb24de03c)  

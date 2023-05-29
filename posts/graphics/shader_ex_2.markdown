---
layout: post
title:  "HLSL 프로그래밍 Wave 구현해보기"
date:   2023-05-28 13:45:10 +0900
categories: shader
tags : Shader, Graphic
---
## 오늘의 주제

이번 과제는 전에 배웠던 개념들을 모두 혼합하여 사용하고자 Gestner Wave를 사용한 바다를 만들어보기로 하였다.  


## > [제작된 Shader 보기](https://www.youtube.com/watch?v=XL_UecVhJT4)

## 구현해야 할 부분은?  
1. 파도의 움직임을 구현하기위해 sin, cos을 통해 position 값을 제어해주기
> Gestner 웨이브 유사 공식을 사용할 예정이다.  
1. UV Scroll / Gradient 활용하여 파도의 색깔 값 제어하기
1. UniversalFragmentPBR 함수를 사용하여 BDRF 구현하기
1. Normal Map을 활용하여 파도의 표면 부분 변경 및 라이트 반영하기
1. 왜곡효과 주기

전체 내용을 기술할경우 너무 길어질듯하니 중요한 부분만 기술하도록 하겠다.  
## **전체 코드를 바로 보고싶다면 맨 아래로 내리시면 되겠다.**



### 0. Shader 파일 만들기
우선 가장 기본적인 shader 파일을 만들어 주자  
HLSLPROGRAM부터 ENDHLSL 사이에 쉐이더 코드들이 들어가니 정의된 명령어를 제외하곤 모두 안에 쓰면 된다.  
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/0420f500-b082-4276-a3da-10d9912a05a3)

마찬가지로 hlsl 파일도 만들어 기본적인 내용을 정의해주자
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/94752079-1a99-4576-b74e-c047115e57d9)  

쉐이더 내부의 함수와 프로퍼티를 사용하기위해 패키지 include 하는것을 잊지 말자.  
```cs
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"  
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"  
```  
간단한 내용이니 전혀 걱정할 필요없다.  

### 1. sin, cos 함수를 이용하여 wave 만들기
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

### 2. sin, cos 함수를 이용하여 wave 만들기



## wave Shader  
```cs
Shader "Custom/wave_Shader"
{
    Properties
    {
        [Header(Wave property)]
        waveTexture("Wave Texture", 2D) = "white"{}
        waveValue("Wave uv Value",Range(0.0,10.0)) = 0
        waveDuration("Wave Duration",Range(0,10)) = 0
        waveAmplitude("Amplitude",Range(0.0,10.0)) = 0
        waveSurfacePow("surface pow", Range(0,10)) = 0

        [Header(BRDF)]
        _SpecCular("Specular",Range(0,1)) = 0
        _Smoothness("Smooth", Range(0, 1)) = 0

        [Header(Normal property)]
        [NoScaleOffset][Normal]waveNormalTexture("Normal Map",2D) = "bump"{}
        NormalTextureValue("Normal Appear Value",Range(0,1)) = 0
        _NormalOffset_X("Normal X_Offset",Range(0,2)) =0
        _NormalOffset_Y("Normal Y_Offset",Range(0,2)) =0
        _NormalAddValue("Normal Add",Range(0,5)) = 0


        [Header(Colors)]
        AlphaColor("Alpha", Range(0,1)) = 1
        MainColor("Start color",Color) = (1,1,1,1)
        DestColor("Destination Color", Color) = (0.1,0.5,1,1)


        [HideInInspector] _ZWrite("ZWrite",Float) = 0
        [HideInInspector] _Cull("Culling",Float) = 0


        [Header(Wave Distort)]
        waveSurfaceDistort("Distort", Range(0,10)) = 0

        _ShearCenter("Shear CenterValue",Range(-1,1)) =0
        _ShearStrength("Shear Strenth",Range(-5,5)) = 0
        _ShearOffset("Shear Offset",Range(-1,1)) = 0
    }

    SubShader
    {
        Tags
        {"RenderPipeline"="UniversalPipeline" "RenderType" = "Transparent"}
        Pass
        {
            Tags{"LightMode" = "UniversalForward"}

            Blend SrcAlpha OneMinusSrcAlpha

            HLSLPROGRAM

            #if UNITY_VERSION >= 202120
            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS

            #endif
            #pragma multi_compile_fragment _ _SHADOWS_SOFT
            #pragma vertex vert
            #pragma fragment frag
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            #include "wave_Shader.hlsl"

            ENDHLSL
        }

    }
}
```

## wave Shader.hlsl  
```cs
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"
struct Attributes
{
    float3 positionOS : POSITION;
    float3 normalOS : NORMAL;
    float4 tangentOS : TANGENT;

    float2 uv : TEXCOORD0;
};

struct Varyings
{
    float3 posWS : TEXCOORD1;
    float4 posCS : SV_POSITION;

    float3 normalWS : NORMAL;
    float4 tangentWS : TEXCOORD2;
    float3 bitangentWS : TEXCOORD4;

    float2 uv : TEXCOORD0;
    float2 normalUV : TEXCOORD5;
    float2 GradientUV : TEXCOORD6;
};


TEXTURE2D(waveTexture);
SAMPLER(sampler_waveTexture);

TEXTURE2D(waveNormalTexture);
SAMPLER(sampler_waveNormalTexture);

CBUFFER_START(UnityPerMaterial)
float waveValue;
float waveDuration;
float waveSurfacePow;
float waveSurfaceDistort;
float waveAmplitude;

float _Smoothness;
float _SpecCular;

float NormalTextureValue;
float _NormalOffset_X;
float _NormalOffset_Y;
float _NormalAddValue;


float4 waveTexture_ST;
float4 waveNormalTexture_ST;

//float _rotateValue;

float4 MainColor;
float4 DestColor;
float AlphaColor;


float _Reveal;
float _ShearStrength;
float _ShearOffset;
float _ShearCenter;

CBUFFER_END


Varyings vert(Attributes input_Att)
{
    Varyings v_output;

    VertexPositionInputs posinput = GetVertexPositionInputs(input_Att.positionOS);
    v_output.posCS = posinput.positionCS;
    v_output.posWS = posinput.positionWS;

    VertexNormalInputs normalInput = GetVertexNormalInputs(input_Att.normalOS, input_Att.tangentOS);
    v_output.normalWS = normalInput.normalWS;
    v_output.tangentWS= float4(normalInput.tangentWS,input_Att.tangentOS.w);
    v_output.bitangentWS = normalInput.bitangentWS;

    v_output.uv = TRANSFORM_TEX(input_Att.uv.yxy, waveTexture);

    //--------------------------  Gradient UV -----------------------------------//
    v_output.GradientUV.x = lerp(0,1,v_output.uv.x);
    v_output.GradientUV.y = lerp(0,1,v_output.uv.y);
    v_output.uv = v_output.uv * 1.5;

    //--------------------------  Wave  -----------------------------------//
    v_output.posCS.x += waveAmplitude * sin(v_output.uv.x * waveValue  + _Time.w * waveDuration );
    v_output.posCS.y += waveAmplitude * cos(v_output.uv.y * waveValue + _Time.w * waveDuration *1.3);
    v_output.posCS.y += waveAmplitude * cos(v_output.uv.x * waveValue + _Time.w * waveDuration);

   //---------------------------- Shear  ----------------------------------//
    float2 delta = v_output.uv - _ShearCenter;
    float delta2 = dot(delta.xy, delta.xy);

    float2 delta_offset = delta2 * _ShearStrength * (_CosTime.w *0.1);
    float2 shear = v_output.uv
    + (float2(delta.y, -delta.x) * delta_offset) + (_ShearOffset);

    v_output.uv = shear ;

    //---------------------------- UV Scroll  ----------------------------------//
    v_output.uv.y += _Time.y *0.25;
    v_output.uv.x += _Time.y *0.15;

    v_output.normalUV.x = v_output.uv.x + (cos(_Time) * _NormalOffset_X);
    v_output.normalUV.y = v_output.uv.y + (sin(_Time) * _NormalOffset_Y);

    return v_output;
}


half4 frag(Varyings input_Vry) : SV_TARGET
{

    //---------------------------  Normal Map  -------------------------------------//
    float3 normalTS = UnpackNormalScale(SAMPLE_TEXTURE2D(waveNormalTexture, sampler_waveNormalTexture, input_Vry.normalUV),NormalTextureValue);
    float3x3 tangentToWorld = CreateTangentToWorld(input_Vry.normalWS, input_Vry.tangentWS,input_Vry.tangentWS.w);
    input_Vry.normalWS= normalize(TransformTangentToWorld(normalTS + _NormalAddValue , tangentToWorld));


    //----------------------------  Main Color  --------------------------------------//
    float4 output;
    output = SAMPLE_TEXTURE2D(waveTexture,sampler_waveTexture, input_Vry.uv) * MainColor;
    output= pow(output, waveSurfacePow);
    output = 0.5- output;

    float4 finalColor = lerp(MainColor,DestColor,input_Vry.GradientUV.x);
    float4 finalColor_2 = lerp(MainColor,DestColor,input_Vry.GradientUV.y);
    output +=  0.5 * (finalColor + finalColor_2);



    InputData lightingInput = (InputData)0;
    lightingInput.normalWS = input_Vry.normalWS;
    lightingInput.positionWS = input_Vry.posWS;
    lightingInput.viewDirectionWS = GetWorldSpaceNormalizeViewDir(lightingInput.positionWS);
    lightingInput.shadowCoord = TransformWorldToShadowCoord(input_Vry.posWS);
    lightingInput.positionCS = input_Vry.posCS;
    lightingInput.tangentToWorld = tangentToWorld;

    SurfaceData surfaceInput = (SurfaceData)0;
    surfaceInput.albedo = output.rgb;
    surfaceInput.alpha = output.a * AlphaColor;
    surfaceInput.specular = _SpecCular;
    surfaceInput.smoothness = _Smoothness;
    surfaceInput.emission = 0;
    surfaceInput.metallic = 0;
    surfaceInput.normalTS = normalTS;


    return UniversalFragmentPBR(lightingInput, surfaceInput);
}
```

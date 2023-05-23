---
layout: post
title:  "HLSL 프로그래밍 clip 활용하기"
date:   2023-05-12 15:24:33 +0900
categories: shader
tags : Shader, Graphic
---

```cs
Shader "Custom/clip"
{
    Properties
    {
        _MainTexture("Main Texture", 2D) = "white"{}
        _BaseColor ("Base Color",Color) =(1,1,1,1)
        [NoScaleOffset]_dissolveTexture("Dissolve Texture",2D) = "white"{}

        Dissolveness("This is Dissolve Value",Range(-2,2)) = 0
        DustColor("DustColor",Range(0,1))= 0
        FadeSpeed("FadeSpeed",Range(0,3)) = 0
    }

    SubShader
    {
        Tags { "RenderType" = "Transparent" "Queue" = "Transparent" }
        Pass
        {

            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "hlsl_portfolio.hlsl"

            ENDHLSL
        }
    }
}
```

```cs
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

struct Attributes
{
    float3 positionOS : POSITION;
    float2 uv : TEXCOORD0;
    float2 maskUV : TEXCOORD3;
};

struct Varyings
{
    float4 positionCS : SV_POSITION;
    float3 positionWS : TEXCOORD1;
    float2 uv : TEXCOORD0;
    float2 maskUV : TEXCOORD3;
};

TEXTURE2D(_MainTexture);
SAMPLER(sampler_MainTexture);

TEXTURE2D(_dissolveTexture);
SAMPLER(sampler_dissolveTexture);


CBUFFER_START(UnityPerMaterial)

float4 _dissolveTexture_ST;
float4 _MainTexture_ST;
float4 _BaseColor;

float DustColor;
float Dissolveness;
float FadeSpeed;

CBUFFER_END


void Clipping(float4 color,float4 dissolve)
{
    clip(color.a - (dissolve.a));
}



Varyings vert(Attributes v_in)
{
    Varyings v_out;
    v_out.positionCS = TransformObjectToHClip(v_in.positionOS);
    v_out.uv = TRANSFORM_TEX(float2(v_in.uv),_MainTexture);

    return v_out;
}


half4 frag(Varyings f_in) : SV_TARGET
{
    float2 uv = f_in.uv;
    float4 color = SAMPLE_TEXTURE2D(_MainTexture, sampler_MainTexture, uv);
    float dissolveValue =
    SAMPLE_TEXTURE2D(_dissolveTexture, sampler_dissolveTexture, uv).r;

    dissolveValue += (Dissolveness + _SinTime.w ) *FadeSpeed;

    if(color.a <= dissolveValue) Clipping(color,(dissolveValue));

    else if(color.a - (dissolveValue) <=  0.1)
    color = smoothstep(0.1,1,DustColor);
    else if (color.a - (dissolveValue) <=  0.2)
    color = float4(1,0,0,1);
    else if(color.a - (dissolveValue) <=  0.3)
    color = float4(1,1,0,1);
    else if(color.a - (dissolveValue) <=  0.5)
    color = lerp(0,1, color.a - (dissolveValue));

    else
    {
        return color;
    }

    return pow(color,3);
}
```

![2023-05-23 16;31;58](https://github.com/rech4210/rech4210.github.io/assets/65288322/6b1dcffe-a4f3-4162-92ad-4710aa828340)

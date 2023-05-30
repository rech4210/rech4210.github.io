---
layout: post
title:  "HLSL 프로그래밍_8 PBR 마테리얼 Rendering Debugger"
date:   2023-04-28 13:12:47 +0900
categories: shader
tags : Shader, Graphic
---
## PBR
>물리 기반 렌더링 (Physically Based Rendering)의 약자로 양방향 반사율 분포 함수를 통해 사실적인 조명을 시뮬레이션 함.

이미 유니티에서 PBR이 구현되어있기 때문에 그냥 함수를 사용하면 된다.  

```cs
//in Fragment
return UniversalFragmentPBR(lightingInput, surfaceInput);
```  

또한
자체적으로 Specular lighting이 내장되어 있다.  
그 말은  **#define _SPECULAR_COLOR** 키워드를 삭제해도 된다는 뜻

![2023-04-24 14;20;36](https://user-images.githubusercontent.com/65288322/233906445-a3f9a2cf-8ace-4b2a-a247-381cbda3feda.gif)  
하이라이트가 꽤나 사실적이네요  
이것이 BRDF이다.  

## Debugging View  
frame debugger 는 변수의 값을 확인할 수 없는데, 그렇다면 어떻게 쉐이더 내부 변수들이 값을 확인할 수 있을까?  
방법중 하나는 색으로 표현하는 것이다. 알다싶이 vector 또한 컬러값으로 나타낼 수 있다 그렇기에 이를 이용해 해당 정점의 vector 값을 가져와 컬러로 변환한다.  

fragment 단에서 해당 픽셀의 값을 remap시켜주면 된다  
> remap은 -1,1 사이의 정점 값을 [0,1] 사이로 변환시켜주는 과정이다.
```
#if normalWS <= 0
           normalWS = (normalWS +1) *0.5;
#endif
           return float4(normalWS,1);
```  

![2023-04-24 14;48;00](https://user-images.githubusercontent.com/65288322/233909988-85ee1ebc-bd06-4076-b413-04f0db8951d2.gif)  
remap을 통해 나타낸 정점의 컬러 값이다.  
normal의 값이 컬러로 나타난 모습.  

컬러이외에도 UV를 확인할 수 있는데, normalWS 정점을 가져오는게 아닌 uv의 값을 바로 반환해주면 된다.
```cs
return float4(uv,0,0);  
//두번째 인자는 w에 해당하며 z축 컬러값을 각 정점마다 더해준다.  
// float4 타입을 쓰는 이유는 frgament 함수의 반환 타입이 float4이기 때문임.  
```
![image](https://user-images.githubusercontent.com/65288322/233926960-851a8f66-a172-403f-812c-e04f3f4e0a56.png)  
>UV의 매핑이 이렇게 나온다  

## Rendering Debugger  
![2023-04-24 16;50;17](https://user-images.githubusercontent.com/65288322/233932940-1a2c01ee-0c31-4bfb-ab13-c0aac6eb5ce2.gif)  
Rendering Debug를 사용하면 현재 사용된 property를 시각화하여 볼 수 있음.  2020버전은 안 될 수도?..  
![image](https://user-images.githubusercontent.com/65288322/233933219-c55e23bb-a536-49f8-ba10-b36aee0abc84.png)  
다양한 프로퍼티를 확인할 수 있음.  

## Normal Mapping   
> 노멀맵은 노말 벡터를 수정하여 diffuse lighting 강도를 결정한다.  

```cs
float4 normalSample = SAMPLE_TEXTURE_2D(_NormalMap, samper_NormalMap, uv);
float3 normalWS = UnpackNormal(normalSample);
return float4 ((normalWS +1) * 0.5, 1);
```
### UnpackNormal  
> 노말 맵 텍스쳐를 샘플링하여 디코딩하는 함수이다.  
> 디코딩 하는 이유는 유니티가 normal 맵을 압축해서 저장하기 때문이라 한다.  


## Tangent Coordinate  
![image](https://user-images.githubusercontent.com/65288322/234154980-cb84262a-b5df-4c4a-a348-a63a19df960c.png)  
> 각각 의 X,Y,Z는 탄젠트 - 비탄젠트 - 노말과 대응된다.    

tangent, bitangent, normal vector로 구성된다.  
normal vector를 기준으로 tangent와 bitangent는 항상 동일한 방향으로 정렬됨.  

렌더링에 사용되는 조명, 카메라 등의 데이터는 WS를 기준으로 하기 때문에, lighting이 수행되기 위해서는 이 Tangent 좌표계 값을 WS로 가져 올 필요가 있다.
마찬가지로 tangent 값을 사용해야 하는데, 이는 normal map이 모델 표면의 높이 값을 저장하기 때문에, 저장된 normal vector와 모델의 표면을 연결시켜주는 역할을 한다.

![image](https://user-images.githubusercontent.com/65288322/234156937-bece3aa0-b499-4a3d-beb5-cd2d1d3c160c.png)  
> tangent space를 WS로 바꿔주는 과정이 필요하다.  

정리하자면 normal 맵의 lighting 연산을 위해서는  
1. normal, tangent, bitangent 값을 계산한다.
2. normal vector 값을가져온다.
3. TBN 행렬을 이용하여 Tangent space의 normal vector를 world space로 변환한다.


![image](https://user-images.githubusercontent.com/65288322/234157741-6fd73a9f-3595-40a9-b5b7-13208ddab3f9.png)  
노말과 똑같이 정점 조립단계에서 Tangent, bitangent 벡터 값을 가져올 수 있다.  
이제 tangent vector 값을 WS로 가져와 이를 조명연산에 사용해보자.  

우선 노말맵을 받아올 2D 프로퍼티를 하나 선언한다.  
```cs
[NoScaleOffset][Normal] _NormalMap("Normal", 2D) = "bump" {}
//[NoScaleOffset]는 tilling offset 속성을 제한하는 플래그임.
//[Normal]의 경우 텍스쳐의 인코딩을 확인하는 플래그.
```  
-----  
그다음 컬러맵과 동일하게 TEX2D 변수를 생성하고 샘플링을 한다.  
```cs
TEXTURE2D(_ColorMap); SAMPLER(sampler_ColorMap);
TEXTURE2D(_NormalMap); SAMPLER(sampler_NormalMap);
```  

-----

Tangent 값을 이용하기위해 LightPass에 tangent 변수와 시맨틱을 선언한다.  
tangent space를 WS로 바꾸기 위해 tangentWS로 바꿔주자.  
```cs
struct Attributes {
	float4 tangentOS : TANGENT;
};

struct Interpolators {
	float4 tangentWS : TEXCOORD3;
};

aryings vert(Attribute input)
{
    Varyings output;

    VertexPositionInputs posinput = GetVertexPositionInputs(input.positionOS);

    output.posCS = posinput.positionCS;
    output.posWS = posinput.positionWS;

    VertexNormalInputs normInput = GetVertexNormalInputs(input.normalOS,input.tangentOS);
    // 탄젠트, 비탄젠트, 노말을 가진 좌표계 구조체
    // 비탄젠트의 경우 TANGENT 시맨틱의 w 프로퍼티에 저장된다.

    output.normalWS = normInput.normalWS;
    output.tangentWS = float4(normInput.tangentWS,input.tangentOS.w);
    // 구조체에는 float3로 저장되어서 w 프로퍼티를 따로 넣어준다.

    output.uv = TRANSFORM_TEX(input.uv,_MainTex);
    return output;
}
```
-----   
다음 본격적으로 tangent를 이용해 tangent space의 정점 값을 가져와 이를 World 좌표로 바꿔주자.  
이를 위해선 UnpackNormalScale, CreateTangentToWorld 두 가지의 메소드가 필요하다.  

```cs
float3 normalTS = UnpackNormalScale(SAMPLE_TEXTURE2D(_NormalMap,sampler_NormalMap,uv),_TestValue);
    float3x3 tangentToWorld = CreateTangentToWorld(normalWS, input.tangentWS,input.tangentWS.w);
```  
**UnpackNormalScale**은 노멀맵을 디코딩하기위해 사용하는 함수이다.  
노말 맵의 법선 벡터는 텍스처 좌표를 통해 계산되고, 유니티에서는 노멀맵을 특수한 형태로 압축하여 사용하기에 이를 풀기위해 사용한다.  

**CreateTangentToWorld** 함수의 반환타입은  float3로 각각 tangent, bitangent, normal의 vector3 타입 3가지를 가지고 있어서이다 좀 더 자세히 살펴보자.  

### CreateTangentToWorld
```cs
real3x3 CreateTangentToWorld(real3 normal, real3 tangent, real flipSign)
{
    // For odd-negative scale transforms we need to flip the sign
    real sgn = flipSign * GetOddNegativeScale();
    real3 bitangent = cross(normal, tangent) * sgn;

    return real3x3(tangent, bitangent, normal);
}
```
이전 코드에서 tangenWS를 만들때 tangentOS.w 파라미터를 넣어줬을 것이다.  

이 w 파라미터는 CreateTangentToWorld 함수의 flipSign에 해당되며 이는 tangent와 normal을 외적해 bitangent를 구할때의 부호를 유도하는 역할을 한다.  
> return unity_WorldTransformParams.w >= 0.0 ? 1.0 : -1.0;  bitangent의 부호 유도    
이로써 vector3 타입의 3x3 World기준 tangent 값을 얻을 수 있게 된것이다.

최종적으로 normal map에서 추출한 normal vector를 tangent space에서 world space로 변환하여 연산하기 위해 **TransformTangentToWorld** 함수를 사용해주자.  
```cs  
// tangentToWorld is the matrix representing the transformation of a normal from tangent to world space
real3 TransformTangentToWorld(float3 normalTS, real3x3 tangentToWorld, bool doNormalize = false)
{
    // Note matrix is in row major convention with left multiplication as it is build on the fly
    real3 result = mul(normalTS, tangentToWorld);
    if (doNormalize)
        return SafeNormalize(result);
    return result;
}
```  
함수의 내용으로 normalTS와 TangentToWorld행렬을 곱하여 나온 결과값을 최종 normalWS로 사용하는 것이다.  

![image](https://user-images.githubusercontent.com/65288322/235592827-5f6a1ee3-716d-4b19-978f-5f35e51ed951.png)  
나무 재질의 텍스쳐를 넣어주도록 하겠다.  
텍스쳐를 사용하기전에 반드시 Normal Map으로 변환해주어야 에러가 안생긴다.  

![2023-05-02 15;14;04](https://user-images.githubusercontent.com/65288322/235592394-ab6808ca-4f1f-4a8e-9dbe-daa8472502ac.gif)  
normal map이 잘 적용됐다!  
말 그대로 normal map의 값을 추출해 normal vector로 사용하고, 바뀐 normal vector의 값을 기준으로 tangent bitangent를 가져와 WS로 변환후 lighting 연산에 사용한 것이다!  

### Rendering debugger 확인하기  
normal map이 적용되었으니 간단한 작업을 통해 이제 렌더링 디버거를 통해 확인해볼 수 있다.  
litPass의 InputData, SurfaceData에 해당 구문을 추가하자.  
```cs
lightingInput.tangentToWorld = tangentToWorld;

surfaceInput.normalTS = normalTS;
```
------  
다음으로 shader 파일에 NORMALMAP을 정의해준다.  
이 쉐이더가 NORMALMAP을 지원한다는 신호를 디버거에 전달한다.  
```cs
#define _NORMALMAP
```
![2023-05-03 10;55;09](https://user-images.githubusercontent.com/65288322/235819779-41fc3108-7bb3-4690-9e65-8ab31156fb96.gif)  
*디버거를 통해 노말맵을 확인할 수 있다!*

## Metallic  
> Metallic은 금속과 비금속에 대한 프로퍼티이다.  
이전 Specular, smooth값을 정의하고 조절하던것 처럼 Metallic 또한 프로퍼티를 정의하여 값을 조절할 수 있다.  

URP 내부의 InputData와 SurfaceData에서 이미 기능이 구현되어있으니 해당 부분에 값을 넣어주기만 하면 된다.  

<h5 a><strong><code>clipMacro.hlsl</code></strong></h5>  

```cs  
float4 _MainTex_ST;
float _Smoothness;
float _SpecColor;
float4 _BaseColor;
float _Cutoff;
float _Emmision;
float _TestValue;
float _Metallic;
```  
----  

<h5 a><strong><code>Lit.hlsl</code></strong></h5>  

```cs  
surfaceInput.metallic = _Metallic;  
```  
----  
![2023-05-03 15;12;57](https://user-images.githubusercontent.com/65288322/235843554-f4c95007-72dd-4b53-b579-715ed6e1ea34.gif)  
>[0,1] 범위로 값이 높을수록 금속체를 띈다.  

### Metalness Masks  
> 텍스쳐를 적용하여 일부분만 금속성질을 띄게 한다.
```cs
surfaceInput.metallic = SAMPLE_TEXTURE2D(_MetalMap,sampler_MetalMap,uv).r * _Metallness;
```  
해당 구문 하나만 작성해주면 된다.  
샘플링 결과값의 r 채널을 가져오는 이유는 Metallic이 Texture의 r 채널의 값을 사용하기 때문이다.  

> 메탈릭 텍스쳐의 rgba 중에서 a는 Smoothness가 되며 GB 채널은 무시된다.  
![image](https://user-images.githubusercontent.com/65288322/235861759-29e1bd03-2db0-439f-b99e-b20093165a5d.png)  
> 선택된 Metallic Map  


![2023-05-03 17;01;08](https://user-images.githubusercontent.com/65288322/235861596-b2e0337c-7d6f-4053-a040-fc81c1b14ab3.gif)  
*선택된 텍스쳐에 따라서 metallic mask 되는걸 알 수 있다!*

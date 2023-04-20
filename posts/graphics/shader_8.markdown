---
layout: post
title:  "HLSL 프로그래밍_6 Alpha CutoutMode"
date:   2023-04-18 10:45:10 +0900
categories: shader
tags : Shader, Graphic
---

## Alpha Cutouts
![image](https://user-images.githubusercontent.com/65288322/231665739-1fad74d7-e9a7-4f09-ba27-6ab432b7fff5.png)  
Opaque와 Transparent를 결합한 방식으로 Alpha Clipping, Alpha Testing등으로 불린다.  
이 기능을 통해서 texture의 원하는 부분만 렌더링 할 수 있다.


부분마다 Opaque와 Transparent가 존재하므로 혼합모드가 아니기 때문에 부분마다의 ZWrite와 Shadow Cast 및 Blend에 신경을 쓰지 않아도 된다.  


이 Alpha Cutouts은 Render Queue의 순서중 Geometry를 사용하지 않고 AlphaTest를 사용한다. 왜일까?


> 일반적으로 최적화를 위해 Depth Sorting을 이용하는데, Alpha Cutout은 Geometry 단계보다 더 많은 비용이 들기에 후순위 렌더를 수행한다.  

여기서 Alpha를 적용하기 위해 Custom Inspector에 TransparentCutout case를 추가해준다.

```cs
  case SurfaceType.TransparentCutout:
                material.renderQueue = (int)RenderQueue.AlphaTest;
                material.SetOverrideTag("RenderType", "TransparentCutout");
                break;
```   


기존 Opaque와 동일하게 Cutout은 Shadow나 ZWrite를 그대로 적용해도 되므로 큰 수정은 없다.  

다음으로 cutout을 수행하기위해 clip함수를 사용해보자.  



```cs
// 텍스쳐의 Alpha와 Color의 Alpha 값을 곱하여 -0.5를 한다.
// 값이 0.5보다 작다면 해당 fragment를 자른다.
clip(ColorSample.a * _Color.a - 0.5);
```  

해당 함수를 사용하면 fragment 결과물을 반환하지 않고 모두 삭제시킨다.   
Depth buffer나 render target으로 사용하지 않기에 호출하지 않은것과 동일한 결과가 나온다.  


![2023-04-13 16;37;30](https://user-images.githubusercontent.com/65288322/231688106-d45c4a1d-0344-46ec-ab25-b9b2b1827d86.gif)   

보다싶이 AlphaCutout의 값이 증가할수록 나머지 부분이 Cutout된다.  

말 그대로 이 값보다 alpha값이 작으면 fragment를 clip(discard) 해버린다는 얘기다.

한가지 문제점이 있는데, Cutout이 그림자에는 적용되지 않는것이다.  

ShadowCast.hlsl 파일에도 clip 함수를 선언하면 되지만, 프로퍼티는 셰이더 전체에 정의되므로 공통으로 사용할 수 있는 파일에 만들어주는게 좋을듯 하다.  

새로운 hlsl 파일을 만들어주자.  
```cs
ClipMacro.cs
TEXTURE2D(_MainTex);
SAMPLER(sampler_MainTex);

CBUFFER_START(UnityPerMaterial)
float4 _MainTex_ST;
float _Smoothness;
float _SpecColor;
float4 _BaseColor;
float _Cutoff;
CBUFFER_END


void TestAlphaClip(float4 colorSample)
{
    #ifdef _ALPHA_CUTOUT
        clip(colorSample.a * _BaseColor.a - _Cutoff);
    #endif
}
```    

이처럼 기존 hlsl 파일에 있던 프로퍼티와 텍스쳐 변수들을 한 곳에 묶었다.  
이 파일을 다른 hlsl에서 사용하기 위해서는 #include를 넣어주어야 한다.

> 이 과정에서 이전에 선언해두었던 변수들을 재정의하게되면 에러가 발생하니 주의하자. 이를 막기위해 Guard Keyword를 사용한다.

### Guard Keyword
> HLSL파일의 오류를 방지하기 위해  전체 코드를 #ifndef keyword 형식으로 묶어버리는것을 뜻함.  

```  
#ifndef MY_SHADOWCASTER_INCLUDED
#define MY_SHADOWCASTER_INCLUDED

//code

#endif
```  
해당 과정을 거치되면, #include시 중복되는 프로퍼티 선언을 방지할 수 있다.  

![2023-04-13 16;45;27](https://user-images.githubusercontent.com/65288322/231690045-9e6e2efc-bb55-446c-9580-9a123497d23c.gif)  

ShadowCaster Pass에 그림자 Alpha가 적용된 모습이다.

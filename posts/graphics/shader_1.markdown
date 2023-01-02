---
layout: post
title:  "쉐이더란?"
date:   2022-07-01 19:52:50 +0900
categories: post
description: 쉐이더의 기초, hlsl 문법을 배우자
tags : Shader, Graphic
---
*천리길도 한 걸음 부터*  
* * *

## 쉐이더란 무엇일까?  
![쉐이더](https://cdn2.unrealengine.com/Unreal+Engine%2Fblog%2Fjobs-in-unreal-engine---technical-artist%2FStore_AutomotiveMaterialPack-1920x1080-fe768f6ea44e10a78c1ee1159da5d2680ead2827.jpg)

쉐이더는 그래픽스 처리과정중 수학적인 연산과정을 거쳐 렌더링을 하는 방식을 말한다.   

필자는 유니티를 활용하여 HLSL과 쉐이더 그래프를 이용해 쉐이더를 작성할 예정이다.  
시작하기 앞서 쉐이더를 제작하는 방법에 대해 간단하게 알아보자.  

### 쉐이더 언어의 종류
쉐이더 언어는 CG, GLSL, HLSL로 나뉜다.  
CG는 Computer for Graphics로, 이를 기반으로 나온 다음 언어는 HLSL, GLSL 크게 두 갈래가 나뉜다.

> #### GLSL 그리고 HLSL  
> GLSL : OpenGL 기반의 Shader Language이다. 호환성이 매우! 좋아 실행 환경에 종속되는 경우가 적다.  

> HLSL : High Level Shader Language로 마이크로소프트사가 개발한 그래픽스 언어이다.  
윈도우 플랫폼에 종속적이기에 호환성이 처참하다 다만 반대로 말해 최적화 성능만큼은 뛰어나다!.  

필자는 유니티 엔진을 다루기에 HLSL에 대해 중점적으로 포스팅 하고자 한다.  
우선 HLSL의 구조를 알아보자.  

```hlsl
Shader ShaderFolder/MyCustomShader // 쉐이더를 모은 폴더와 쉐이더 이름
{
	// 유니티 프로퍼티에서 조절할 인터페이스를 만드는 곳!
	Properties
	{
	}
	SubShader // 쉐이더 코드가 들어가는 곳
	{
		tags // 렌더 타입이나 렌더링 파이프라인 설정 등등을 하는곳
		{
		}
		Pass
		{
			#pragma vertex vert // vertex를 vert로 지시하는 지시어
			#prama fragment frag

			HSLSPROGRAM // 시작

			#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
			// HLSL에서 사용되는 매크로와 함수가 포함되어 있어요

			// Attributes의 구조체를 버텍스 쉐이더의 입력 구조체로 이용합니다
			struct Attributes
			{
				 float4 positionOS   : POSITION;
				 // 오브젝트 스페이스의 버텍스 위치값 입니다
			};


			struct Varyings
			{
				float4 positionHCS  : SV_POSITION;
				// 이 구조체의 포지션 변수는 반드시 SV_POSITION 시멘틱을 가져요
			};

			// 버텍스 쉐이더의 타입은 반드시 출력 대상 구조체와 타입이 일치해야함
			Varyings vert(Attributes IN)
			{
				Varyings Out;
				Out.positionHCS = TranformObjectToHClip(IN.positionOS.xyz)  
				// 오브젝트 스페이스 정점을 클립 스페이스로 변환

				return Out;
			}

			half4 () : SV_Target
			{
				half4 customColor = half4(0.5,0,0,1); // 순서대로 RGBA
				return customColor;
			}
			ENDHLSL // 종료

		}

	}
}
```

위에 보이는 SV_POSITION는 시멘틱의 일종이다  
> #### 시멘틱이란?  
> HLSL에서 변수의 입력과 출력을 확인 또는 역활에 대한 의미를 부여하는 행위.  
> ex) : POSITION, : SV_Target  

근데 이 부분이 좀 헷갈린다.. SV_POSITION과 POSITION의 차이가 뭘까?  

일단 찾아본 결과로는 SV는 시멘틱으로 버텍스의 OUT이나 프래그먼트의 IN등 사용되는 매개변수에 의미를 부여하는 것이라고 한다.  

그러니까.. 이 구조체의 매개변수는 프래그먼트의 POSITION에 대응한다.. 이런 뜻인데 이를 만든 이유는 다렉의 호환성 때문이란다.  

> #### Properties  
> 유니티 인스펙터에서 인터페이스를 연결하는 부분  

* 인터페이스를 정의하는 방법은 다음과 같다  

```js
[옵션] 변수이름("뭐 하는건지 설명적기", 어떤 타입) = 초기변수값  

MyTexture("텍스쳐를 받을 인터페이스", 2D) = "white"{}
Myvector("벡터 인자값을 설정하는 인터페이스", Vector) = (1,1,1,1)  
Myfloat("슬라이더가 포함된 테스트 플롯", Range(0.0,1,0)) = 0  
Myfloat("일반 플롯", Float) = 0 등으로 사용할 수 있다.  

기본적으로 vector는 color의 값으로도 사용할 수 있다.  
```  


> #### SubShader  
> tag를 사용하여 렌더링할 방법과 시기를 지정하는 구문이다. 문법은 아래와 같다.  
```hlsl
Tags {"RenderType = Opaque}
```



> #### Pass  
> 전처리구문을 포함하는 영역이다. Pass에 정의된 일련의 쉐이더 연산을 수행하며 렌더 상태를 설정한다  
ex) Cull, ZTest, ZWrite  

Attributes와 Varyings는 각각 vertexshader의 IN & OUT에 해당하며 이 값이 fragment shader로 전달된다.

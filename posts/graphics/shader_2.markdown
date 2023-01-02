---
layout: post
title:  "쉐이더 제작하기"
date:   2022-07-04 10:30:10 +0900
categories: post
description: 쉐이더의 uv에 대해 알아보자
tags : Shader, Graphic
---
*힘든 일에는 용기를 가지고 나아가자*  
* * *

쉐이더를 직접 제작해보자  
오늘은 texture를 입히는 방법을 알아보겠다.  

기본적으로 쉐이더에 texture를 적용하기 위해선 UV가 필요하다.  

> #### UV란?
> UV는 3D 모델링 물체가 2D로 표현되었을때의 일종의 전개도이다.  
![UV 전개도](https://understandingmaya.files.wordpress.com/2015/07/cube_representative_uv_unwrapping.png?w=700&h=467)  
수학시간에 사각형의 전개도를 그린적이 있는데 딱 그것과 비슷하다!  

이름이 왜 uv인가 하니, 좌표계의 xy가 각각 uv에 대응된다 이 uv는 [0-1]의 값을 가지는데, 텍스쳐의 해상도 값에 상관없이 비율로 나눠준다.  
어떤 화질의 텍스쳐를 입힐지는 모르지만, 그 텍스쳐를 [0-1]범위로 축척시키는 것이다.  

![uv컬러](https://t1.daumcdn.net/cfile/tistory/99CF9A3F5B440AC90F)  
여길 보자 이게 uv의 범위를 나타낸것인데, 색깔이 꽤 다채롭다.  

uv [0,0]부터 U축으로 R, V축으로 G값이 증가한다 그리고 [1,1]에 이르러서는 노란색이 나타나는데, 앞서 말했듯이 UV가 x,y에 대응되기 때문이다. 다르게 말해 RG = XY = UV로 매핑된다. 결과적으로 1에 가까울수록 R+G의 값인 Y가 나오는것이다.

> #### UV 입히기  

```hlsl
Shader "Custom/UVtexture"
{
    Properties
    {
        _MyBaseMap("여기 맵을 넣어주세요", 2D) = "white"{}
    }

    SubShader
    {
        Tags { "RenderType" = "Opaque" "RenderPipeline" = "UniversalPipeline" }

        pass
        {
            HLSLPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            struct vertexInput
            {
                float4 position : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct vertexOutput
            {
                float4 position : SV_POSITION;
                float2 uv : TEXCOORD0;
            };

            TEXTURE2D(_MyBaseMap);
            SAMPLER(sampler_MyBaseMap);

            CBUFFER_START(UnityPerMaterial)
                float4 _MyBaseMap_ST;
            CBUFFER_END

            vertexOutput vert(vertexInput i)
            {
                vertexOutput o;
                o.position = TransformObjectToHClip(i.position.xyz);
                o.uv = TRANSFORM_TEX(i.uv, _MyBaseMap);

                return o;
            }

            half4 frag(vertexOutput i) : SV_TARGET
            {

                half4 o = SAMPLE_TEXTURE2D(_MyBaseMap,sampler_MyBaseMap, float2(i.uv.x, i.uv.y)+float2(i.uv));
                return  o;
            }
            ENDHLSL
        }
    }
}
```  



![image](https://user-images.githubusercontent.com/65288322/178305468-eeca00ac-a11b-4f1b-a8c6-c62b12ca8513.jpg)  
짠! 보다싶이 UV에 텍스쳐가 매핑된 모습을 볼 수 있다.  

참고로 uv를 포함한 대부분의 변수는 형식을 맞춰주면 서로 연산해도 된다. 덤으로 Tilling Offset 기능도 한번 다뤄보자.  

> #### Tilling & Offset  

![offset](https://user-images.githubusercontent.com/65288322/178306881-5416778e-8c05-4403-9d3c-dbd4d9ceb21c.png)  
오프셋은 uv를 해당 방향만큼 이동시키는 것이고, 타일링은 같은 모양을 격자 형식으로 만드는 것이다.

오프셋은 SAMPLE_TEXTURE2D에서 각 UV에 숫자 값을 더해주면 된다.  
![offset](https://user-images.githubusercontent.com/65288322/178308249-4ef51b1f-7011-4084-844c-d901cfaa7aa3.png)  
해당 코드 대로면 uv 1을 기준으로 0.5니까 반 정도 이동한 화면이 나타날 것이다.  

![offset](https://user-images.githubusercontent.com/65288322/178308043-1e1c0e13-300c-4837-bb6a-68b39b68a273.png)   
![Tilling](https://user-images.githubusercontent.com/65288322/178313919-9afcf0dc-fc2d-425e-b2bd-9751b91a5881.png)  
타일링의 경우는 uv 값에 각각 곱해주거나 동일한 값을 한번에 곱해주면 된다.  

![Tilling](https://user-images.githubusercontent.com/65288322/178313871-a3bc118d-4a8f-4fb6-b9d8-cd27f61841eb.png)  


정말 간단하지 않은가??! :clap:  

마지막으로 _Time 기능을 사용해보자 한다.  

_Time은 자체적으로 내장된 빌트인 쉐이더 변수라고 한다, 시간의 변화를 주거나 쉐이더의 부드러운 움직임을 위해 사용한다.  

![time](https://user-images.githubusercontent.com/65288322/178311991-06d49860-a3ff-4c3f-b9c8-cd7abfd11016.png)  
아까 uv식에서 x축(u)에는 _Time.y, 그리고 y축(v)엔 _SinTime.z*3 를 입력해줬다.  

각각의 뜻은 x축으로 오프셋을 시간변화에 따라 y속도 만큼, y축을 sin함수에 비례하여 이동시켰다.  
![timegif](https://user-images.githubusercontent.com/65288322/178311427-9bedc377-1e8e-4159-9346-c99e8acdfb97.gif)  
마치 울렁거리는 느낌이 든다 :smile:

참고로 _Time.xyz에 해당하는 xyz들은 축이 아니라 _Time의 시간속도다.  
[유니티 Built-in shader variables _Time](https://docs.unity3d.com/Manual/SL-UnityShaderVariables.html)

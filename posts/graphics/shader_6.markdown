---
layout: post
title:  "HLSL 프로그래밍_4 ShadowCascade"
date:   2023-04-11 09:30:10 +0900
categories: shader
tags : Shader, Graphic
---
## shadow cascade and soft shadow

### [쉐도우 케스케이드](https://docs.unity3d.com/kr/2020.3/Manual/shadow-cascades.html)

![image](https://user-images.githubusercontent.com/65288322/227157658-bd0642ff-86c7-4b00-a853-5bfce0233324.png)
> 원근 앨리어싱

렌더링 공간에서 directional light는 마치 위치를 가지고 있는것처럼 보이지만, 사실 이는 태양을 본따 만든것이다. 그렇기에 실제 장면으로부터의 거리는 무한대로 떨어져있다.  

이로 인해 원근 앨리어싱이 생기며 전체 장면을 포함하는 퀄리티 좋은 shadow map은 만들기가 어려워진다.

![원근 앨리어싱](https://user-images.githubusercontent.com/65288322/227157823-c7ed1c3c-873c-4e63-98a3-303f0ee8ab86.png)

원근 앨리어싱이 발생하는 이유로는 카메라 위치에 따라 원근이 발생하며 이로 인해 섀도우 맵이 불균형하게 스케일되기 때문이다. 위 그림처럼, 가까운 곳은 픽셀4개, 먼곳은 픽셀 20개로 쉐도우 맵이 만들어지지만 렌더링 화면에는 같은 크기로 나타나기에 상대적으로 카메라에 가까울수록 해상도 품질에 영향을 준다.

### Shadow cascade의 작동 방식
소프트 섀도우를 사용하거나, 높은 해상도의 섀도우 맵을 생성하는 방법이 있지만, 많은 메모리와 대역폭을 사용한다.  
반면, 섀도우 캐스케이드를 사용하면

![image](https://user-images.githubusercontent.com/65288322/227162125-044f2d6b-e385-4ca6-a68b-ddf236b1ff46.png)

이러한 점으로 Unity 는 cascade를 고려하여 쉐도우 맵을 샘플링한다.  
여러 쉐도우 맵을 렌더링하고 가장 세부적인 맵을 샘플링하는것이 그 방법이다.  


이처럼 쉐도우 케스케이드는 카메라로부터 떨어진 거리에 따라 절두체 영역을 분할한다.
이때 분할된 Cascaded shadow map은 다른 크기의(픽셀수) 동일한 해상도를 가진다.

![image](https://user-images.githubusercontent.com/65288322/227161834-042969ec-3cc6-4eef-a858-6ee60648f2b5.png)

> 4단계의 케스케이드를 적용한 모습이다.
케스케이드를 많이 사용할수록 원근 앨리어싱에 영향을 덜 받지만, 당연히 렌더링 오버헤드가 증가한다.

근데 이렇게 해도 쉐도우맵 해상도를 올리는것보다 훨씬 싸다고? 한다


-----
## FrameDebugger

![image](https://user-images.githubusercontent.com/65288322/227228056-dc835ee3-3c1e-465b-bd78-f58e5edc4526.png)

Window - analysis - frame debugger를 통해 렌더링 순서등 렌더에 관한 모든 정보를 표시해준다.  
또한 이곳에서 soft shadow 설정이나, Main Light를 제거했을때, 키워드가 변경되는것을 확인할 수 있다.  

## Depth Buffer
Shadow Caster Pass에서 연산을 수행할 때 광원으로부터의 거리에 따라 shadow map이 생성되는데, 이 과정은 render 시스템이 자동으로 수행해준다.  
마찬가지로 Raterizer 과정에서  Clip Position은 depth value로 치환되는데,  Rasterizer 보간이 수행될때 각각의 fragment 정점에 구조체 타입인 depth buffer 값이 저장된다.

이 깊이값은 Opaque인 대상들의 overdraw를 방지하는데 사용된다.


urp에서는  shadow caster pass의 depth buffer 결과값을 shadow map으로 재사용 하는데, 대부분의 Pass들은 각자의 depth buffer를 가지고 있다.

-----
## shadow ance
Shadow Cast Pass에서 그림자를 렌더링할 때 발생하는 문제이다.  

단순히 **depth value**와 **shadow map**에 저장된 값을 비교하게 될 때, 어느 것을 아래에 그려야 할지 렌더링 시스템이 헷갈려하기 때문에 발생하는 문제이다.

이를 해결하기 위해서는

![image](https://user-images.githubusercontent.com/65288322/228268309-aa6ab6de-81dd-4fdc-b7eb-418925d48168.png)

1. bias라는 아주 작은값을 이용해 광원 가까이에 배치한다.

![image](https://user-images.githubusercontent.com/65288322/228268341-ee7e17d1-a7d3-493c-8f19-6e33d6d8864f.png)

2.  nomal offset 값을 조절해 offset 값 만큼 노말 방향으로 이동시키는 방법이다.

```cs
float3 _LightDirection;
// urp에서 제공하는 광원까지의 거리 프로퍼티, 전역번수처럼 쓸수 있음.

//shadow acne를 제어하기위해 필요한 normal offset과 bias를 만드는 함수
float4 GetShadowCasterPositionCS(float3 positionWS, float3 normalWS)
{
    float3 lightDirectionWS = _LightDirection;

    // 마치 광원 거리월드pos와 월드normal이 필요함
    float4 positionCS = TransformWorldToHclip(ApplyShadowBias(positionWS,normalWS,lightDirectionWS));

// 역버퍼일 경우 가장 낮은 depth value(가장 멀리있는)의 값을 넣어준다.
    #if UNITY_REVERSED_Z
        positionCS.z = min(positionCS.z,UNITY_NEAR_CLIP_VALUE);
    #else
        positionCS.z = max(positionCS.z,UNITY_NEAR_CLIP_VALUE);
    #endif
        return positionCS;
}
```

<br>
### 역버퍼란?
> UNITY_REVERSED_Z는 depth buffer를 반전시킨 것으로 이를 역버퍼 값이다 부른다.
흔히 Z-fighting 또는 깊이 인식 문제를 줄이기 위해 사용한다.

기존 렌더링 시스템에서는 가까운 개체부터 먼 개체까지의 depth value를 정렬하고 가까운 값부터 렌더링 한다 참고하자.

```cs
Pass
//ShadowCaster의 Pass 구문
{
    // 이 패스는 ShadowCaster라는 패스로, 그림자를 연산하고 화면에 그리도록 하는 패스이다.
    Name "ShadowCaster"

    Tags{"LightMode" = "ShadowCaster"} // shadowCaster라는 태그를 사용하여 그림자임을 알림.

    // shadow caster가 depth buffer만 사용할때 color는 쓸 필요가 없으니 꺼버린다
    ColorMask 0

    HLSLPROGRAM

    #pragma vertex vert
    #pragma fragment frag

    #include "myShadowCaster.hlsl"

    ENDHLSL
}

```

---
layout: post
title:  "HLSL 프로그래밍_3 ShadowMapping"
date:   2023-04-06 09:30:10 +0900
categories: shader
tags : Shader, Graphic
---

## Shadow Mapping
쉐도우 매핑은 쉐도우를 그리는 알고리즘이다.
URP에서는 어떻게 오브젝트의 쉐도우 매핑을 실시할까?

우선 그림자에 해당하는 면적을  확인해야 한다.

![image](https://user-images.githubusercontent.com/65288322/225001466-4235d22e-3085-41b6-9d93-73899bdeaffd.png)

단순 접근법을 사용하면 정점의 WS와 Light WS사이에 물체의 여부를 판단하는 것이다.
이를 위해 셰이더가 레이캐스트를 모든 오브젝트에 반복해야하므로 느려진다.

![image](https://user-images.githubusercontent.com/65288322/225001831-2fa8d62f-c875-478b-99dd-ed872235ff70.png)  
두번째 매핑 방법으로, 일직선의 광선을 발사하며 다른 표면과 교차하여 검사하는 방식.
1. 빛에서 시작하는 직선의 광선을 발사해 픽셀과 다른 표면을 교차하도록 한다.
1. 광선의 가까운 표면에만 불이 켜지면, 해당 정점이 최소거리보다 큰지 테스트하면 된다.

이렇게하면 빛으로부터 가장 가까운 표면의 값을 구하는 과정이 줄어든다.


기본적으로 뎁스맵을 그릴 때 광원은 하나의 카메라가 되어 이를 렌더링한다.

포인트 라이트, 스폿 라이트, 다이렉셔널 라이트 등등 모두가 하나의 광원 카메라가 되어 오브젝트에 Model - Light - Projection matrix 일련의 매트릭스 연산을 수행하는것과 같이 연산과정을 거쳐 Depth Map을 렌더링하게 된다.  
(실제로 Directional light는 orthograpic이다 그러니 투영은 없다.)


우리가 오브젝트를 렌더링할때 카메라로부터 가장 가까운 표면을 그린다.
이때 거리를 색으로, 카메라를 라이트로 변환하는 방식으로 뎁스맵을 이용할 수 있다.


색상을 어떻게 거리로 나타낼까?
결국 색상과 벡터 모두 숫자에 불과하다.
그렇기에 RGB 중 R 채널 내부에 거리에 따른 값을 색상으로 저장할 수 있다.

![image](https://user-images.githubusercontent.com/65288322/225016626-3fe3f902-848d-46c0-9e8f-d5d529b79c05.png)
> 거리에 따라 R 채널 값을 나타낸 모습이다.

Shadow Mapping 시스템은 색상을 렌더링 하기전에 카메라를 main light의 원근과 일치시키도록 변환한다.
그러고 나서  depth 값을 그리기위해 다른 쉐이더 패스인 shadow caster pass를 이용하여 각 픽셀마다 depth값을 그려준다.

하지만 우리는 이 depth map (shadow map)을 실제로 본적이 없을것이다.
그 이유는 depth map이 그려지고 난 후, 렌더링 과정에서 presentation과정을 훔쳐 sceen 에 이 맵을 그려주는게 아닌 render target이라는 특수 텍스쳐에 이를 그려주기 때문인데,
이 렌더 타겟은 light의 거리에 따른 값을 포함하고 있는데, 이를 shadow map 이라 부르게 된다.

다시 코드로 돌아와서

```
lightInput.shadowCoord = TransformWorldToShadowCoord(i.posWS);
// 정점의 WS를 쉐도우 맵 좌표로 변환하는데 사용되는 함수.
```
<br>

### 정점이 쉐도우 좌표로 변환되는 대략적인 과정은 이러하다.  
1. 정점의 WS와 V_MATRIX를 곱하여 (VS) 광원의 시야 공간으로 변환한다. (V_MATRIX는 광원의 위치, 방향으로 구성됨)    
2. 정점을 shadow map의 2D 좌표로 변환하기위해 VS에 P_MATRIX를 곱한다. (P_MATRIX는 그림자 맵의 수치와 FOV(화각)으로 구성된다. 3D 공간의 점을 절두체에 매핑하면 뷰어의 거리에 따라 쉐도우 2D 좌표 공간에 매핑되는데 , 범위가 -1, 1이 된다)  

![image](https://user-images.githubusercontent.com/65288322/225310278-d57864d1-0d38-4289-b823-15c2aadd15ef.png)

> 해당 결과인 shadowCoord는 NDC좌표에서 픽셀의 위치를 나타내지만, z축은 픽셀의 깊이값으로 표현된다.

3. 결과 좌표를 벡터 W 성분으로 나누어 동차 좌표계로 나타낸다 (shadowCoord /= shadowCoord.w;)  
이러한 결과를 바탕으로 쉐도우 맵을 샘플링 할 수 있는데 코드로 보면 이러하다.

```cs
float shadowDepth = texture(shadowMap, shadowCoord.xy).r;
// shadowMap은 shadow map의 샘플러이고 shadowCoord.xy는 현재 계산될 내 정점의 uv좌표이다.
//shadow map의 r값을 가져와서 깊이 값으로 사용하겠다는 뜻이다.

shadowDepth = shadowDepth * 2.0 - 1.0;
// 기본적으로 perspective projection 을 거치고나면 uv 값의 범위가 [-1,1]이다.
//하지만 현재 shadowCoord는 [0,1]로 매핑시켰으므로 값 비교를 위해다시 [-1,1] 범위로 바꿔준다

float depthBias = 0.005;
float currentDepth = shadowCoord.z; // 현재 내 정점의 깊이 값
float depthDifference = currentDepth - shadowDepth + depthBias;
// 현재 내 정점의 깊이 값 - 쉐도우 맵에 저장된 깊이값 + bias 값을 수행해준다.

// 내 정점의 위치값이 shadow map 보다 가까이 있으면 그림자가 지지않는다.
if (depthDifference > 0.0) {
    // Not in shadow
} else {
    // In shadow
}

```

-----
## bias  
여기서 bias 의 값은 아주 작은 상수 (0.005 등) 값이며, shadow 맵과 각 정점의 값을 비교할 때 그림자 아티팩트를 줄이고 시각적 품질을 향상 시킬 수 있기에 사용한다.

즉, 간단히 얘기하자면 광원으로부터 정점의 WS를 가져와 Shadow map을 만들어 저장한다. 그러면 Shadow map에 z값은 깊이값을 나타내게 된다.  
그럼 이제 이 깊이값과 그림자 계산을 할 대상인 정점간의 거리를 계산해주면 된다.  

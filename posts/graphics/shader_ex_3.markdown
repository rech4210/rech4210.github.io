---
layout: post
title:  "HLSL 프로그래밍 polar coordinat (극좌표 활용)"
date:   2023-05-23 15:53:06 +0900
categories: shader
tags : Shader, Graphic
---


atan2 를 이용하여 각도의 값을 극 좌표계로 나타내는 방식이다.

```cs
  float2 radial = atan2(input_Vry.uv.y ,input_Vry.uv.x) / (3.14);
```

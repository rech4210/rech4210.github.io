---
layout: post
title:  "MACRO"
date:   2023-04-20 14:27:10 +0900
categories: shader
tags : Shader, Graphic
---

> Macro와 함수의 차이점  

[Macro VS function](https://forum.unity.com/threads/macros-vs-function.1008748/)  

1. scope 의 범위가 다름 (함수는 정의된 값 또는 내부의 값, 파라미터만 가능)
2. Macro는 호출 지점에 직접 삽입되는 코드로 해당 범위에 있는 값(local 등)에 액세스 가능.
3. Macro는 호출 이름을 수식으로 바꾼것과 같음.
4. 세미콜론을 붙이지 않음.
5. 기존 Macro를 재정의 할 수 있음.  

```cs
#define MACRO(value) value *= 2

half4 frag(Varyings i) : SV_TARGET
{
    float foo = 1.0;
    MACRO(foo);
}

// 매크로 재정의
#define MY_MACRO(value) value*=1

#undef MY_MACRO
#define MY_MACRO(value) value *=2
```  

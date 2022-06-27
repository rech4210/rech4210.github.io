---
layout: post
title:  "렌더링 파이프라인"
date:   2022-06-26 10:05:04 +0900
categories: post
tags : Shader, Graphic
---
*위대한 일은 하루 아침에 이루어지지 않는다*  
* * *
3차원 이미지를 우리가 보는 평면 모니터에 2차원 래스터 이미지로 표현하기 위한 단계를 뜻함.


## 렌더링 과정
```
물체는 각 단계의 좌표 행렬을 매트릭스 연산을 통한 변환을 거쳐 
최종적으로 그려진다.
```
1. 오브젝트 데이터 받아오기
1. 버텍스 쉐이더
1. 래스터라이저
1. 프래그먼트 쉐이더

-----------------------------------------------------
> ## 목차
1. **정점 데이터 받기**
1. **버텍스 쉐이더**
  - [Tessellation](#tessellation)
  - [Geometry Shader](#geometry-shader)
1. **래스터라이저**
  - [Clipping](#clipping)
  - [Perspective divide](#perspective-divide)
  - [culling](#culling)
  - [Viewport transform](#viewport-transform)
  - [Scan conversion](#scan-conversion)
1. **프래그먼트 쉐이더 / 픽셀 쉐이더**
  - [Z 버퍼 테스트](#z-버퍼-테스트)
  - [stencil test](#stencil-test)
  - [color mask](#color-mask)
  
-----------------------------------------------------


## 정점 데이터 받기
```
버텍스 쉐이더에 입력할 수 있는 정점 값을 받아오는 과정.
```

정점 데이터는 GPU에 의해 **병렬** 처리되는데, 이 과정을 수행하도록 CPU의 **그래픽스 API** 를 초기화 명령이 필요하다.  
> #### 정점이 뭐에요?
> 물체는 삼각형의 메쉬로 구성되는데, 이 삼각형을 이루는 각각의 점을 정점이라고 한다!

![vertex](https://miro.medium.com/max/1050/1*iXasM6kAXxMEO1adrJedtQ.jpeg)  

그래픽스 API의 구성으로는 **GPU 디바이스, 커맨드 큐, 커맨드 버퍼** 로 구성된다.  
CPU는 GPU 디바이스와 커맨드 큐를 생성하여 커맨드 큐에 커맨드 버퍼를 쌓게 되는데, 커맨드 큐는 GPU 디바이스와 연결되어 정점 데이터를 넘겨준다. 이때 데이터는 CPU의 RAM에서 GPU의 VRAM으로 업로드된다.

> #### 왜 이런 방식을 사용하나요?
> GPU가 CPU에 의해 실행 지연되는 경우를 최소화하기 위해 버퍼를 쌓아두는 방식을 이용한다!

렌더링 파이프라인에 사용될 속성들을 가진 **렌더링 파이프라인 상태 오브젝트**는 여러개로 생성되며, 렌더링 파이프라인 서술자가 정점 데이터, 버텍스쉐이더, 프래그먼트 쉐이더를 포함하고 있다.  

> #### 드로우 콜이란?  
> CPU가 GPU로 전달할 명령을 생성하는 단계, 오브젝트를 그리는데 필요한 명령들의 집합을 말함!


정점은 구조체로 [**위치, 컬러, 노말, 텍스처의 좌표**]등을 포함하여 구성되는데, 전달받은 정점 데이터가 전달될 때는 구조체가 아닌 스트링 버퍼 형식으로 데이터를 전송시킨다.  

이 스트링 버퍼는 정점 조립과정을 거쳐 버텍스 쉐이더에 인자로 넣어진다.

-----------------------------------------------------
## 버텍스 쉐이더
```
전달받은 정점 인자를 이용하여 오브젝트를 그려질 화면의 위치로 전달하는 과정.
```
버텍스 쉐이더는 위치 변환과정을 거칠때 행렬 연산을 수행하여 좌표를 변환한다.

| 모델         |  곱하기 행렬           | 결과 |
|:-------------|:------------------|:------|
| 오브젝트 좌표| 모델 행렬 | 월드 좌표  |
| 월드 좌표 | 카메라 행렬   | 카메라 좌표  |
| 카메라 좌표| 투영 행렬      | 투영 좌표   |

> #### 행렬 연산은 한번에 수행! 
> 그려야 할 삼각형이 100개라면, 300개의 정점을 가지고 있다. 이때 행렬 연산을 순차적으로 진행하게 될 시 300 * 월드 행렬 * 카메라 행렬 * 투영 행렬 만큼의 정점 연산을 수행하게 된다!  
> 반면, 하나의 행렬로 묶어 연산한다면 현저히 줄어듬!

### 행렬 수행 과정

![모델 행렬연산 과정](data:image/png;base64,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){: width="600" height="400"}

:sleeping: 이게 무슨말인가 싶겠지만 천천히 확인해보자

#### 1.오브젝트 공간
모델의 피봇이 원점인 상태, 스스로가 세상의 중심인 공간
#### 2.월드 공간
모델이 모델 행렬 연산을 통해 변환된 공간, 절대 좌표 위치를 기준으로 모델의 위치가 정해짐
#### 3.뷰/ 카메라 공간
월드 공간에 카메라 행렬을 연산한 공간, 월드상의 정점 위치를 카메라의 상대적인 위치로 옮김  
시야와 원근감이 존재하지 않음

  - #### 클리핑 공간 
  카메라의 시점을 반영한 공간으로 카메라 이외 영역을 잘라내는 클리핑이 실행됨.   
  4차원을 동차 좌표계를 이용하여 계산
  
#### 4.투영 공간
카메라 위치 기준 공간을 카메라 시야(화면) 기준으로 변환된 공간으로 원근감이 존재함.

  - #### 정규화 공간  
  클리핑이 실행된 후 공간 범위를 [Perspective divide](#perspective-divide)을 거쳐 w로 정규화된 3차원 좌표계  
  뷰포트 장치에 전송하기위해 정규화를 한 상태.
  
-----------------------------------------------------
## Tessellation  
![tessellation](https://t1.daumcdn.net/cfile/tistory/99758D4D5C8053B419)
렌더링이 가능한 최소 단위로 분할하는 작업. 메쉬를 형성할때 삼각형으로 쪼개진 모습을 볼수 있다. tessellation은 이처럼 큰 patch를 수많은 primitive 조각들로 쪼개는 과정이다.  
1. Tessellation Controll Shader  
patch를 primitive로 쪼갤때의 수준을 결정한다.  
제어 포인트 위치 및 기타속성, 사용자 정의를 포함해 생성함.

2. Fixed-funtion tessellation Engine  
patch를 더 작은 primitive로 분배하는 방법을 결정함.

3. Tessellation Evaluation Shader  
tessellation 연산의 결과를 받고, 보간할 위치를 연산하는 과정.

-----------------------------------------------------
## Geometry Shader  
![geometry-shader](https://dexint.files.wordpress.com/2018/02/geom.jpg)  
vertex shader를 인자값으로 사용하는 셰이더로, 하나의 primitive에 대응하는 연산을 수행한다.  
입력받은 primitive에 대한 위치 변환, 생성, 삭제가 가능하며 프로그래머블한 쉐이더!  

:cat: geometry shader를 이용하면 동물의 털 같은 재질을 쉽게 표현할수 있다 :dog:

-----------------------------------------------------
## Clipping  
![clipping](https://developer.download.nvidia.com/CgTutorial/elementLinks/fig4_4.jpg){: width="800" height="400"}  
```
뷰 좌표에 Projection Matrix 연산을 수행하는 과정에서
clip space 이외 영역 물체를 그리지 않는 과정.  
```
clipping은 clip space 공간에서 실행되는데, 쉽게 말해 카메라로 사진을 찍을때 화면 밖의 물체는 어차피 안보이므로 그려줄 필요가 없기에 잘라내는 것이다.  

또한, clipping은 **동차좌표**를 이용한 4차원 (x,y,z,w)좌표에서 실행된다. 어차피 3차원 좌표로 할수 있는데 왜 굳이 4차원 좌표계를 사용할까?  
 
![orthographics-and-persfective](https://www.researchgate.net/profile/Juan-Liu/publication/265693452/figure/fig2/AS:392209707880480@1470521489497/Projection-modes-a-orthographic-projection-and-b-perspective-projection.png)
여기엔 몇가지 이유가 있는데 첫번째는 투영이 포함되기 때문이다.

원근감은 w의 값을 기준으로 0이면 Orthographic 직교 투영, 아닐경우 Perspective 원근 투영으로 결정된다. 위 사진을 보면 물체가 카메라에 투영될때 원근감의 차이를 확인할 수 있다. 그리고 기본적으로 정점 변환을 위한 연산은 매트릭스 행렬의 곱셈으로 나타내지는데 가령.  

#### 1. 변환의 이점  
정점을 이동시킬때 행렬은 벡터 정보를 사용한다. 벡터는 1x4 matrix의 형태인데, 여기 곱해질 행렬이 4x4 matrix 가 아닐경우 행렬의 곱셈이 적용되지 않는다. 이렇듯 계산의 편의성을 위해 3차원에 w를 포함한 동차좌표계를 사용한다.  

#### 2. point와 vecotr 
w의 값이 필요한 이유는 또 한가지 벡터의 속성을 판단하기 쉽기 때문이다. (x,y,z)인 3차원 좌표계의 경우 point인지 vector인지 판단하기 어렵기에 w가 0일때 vector를 1일때 point를 나타낸다.  
* w = 0 vector (확대, 축소, 방향변화)
* w = 1 point (이동)  

#### 3. 원근 표현  
w값은 [0~1]의 범위를 가지는데 0일 경우 직교투영, 1일경우 원근투영이다.  
만약, w의 값이 n이라면 모든 요소의 값을 n으로 나누어주는 과정을 거치며 정규화가 된다. 정규화는  perspective divide 작업으로 NDCS(정규화 좌표)가 생성된다.

-----------------------------------------------------
## Perspective divide  
![perspectivedivide](/assets/img/NDCS.jpg){: width="800" height="400"}  
원근법을 구현하는 과정으로 뷰 좌포계에 투영 행렬을 곱해 NDCS (정규좌표계)가 만들어진다.  
분할은 w 좌표계를 기준으로 전체 요소를 나누어 원근분할을 수행.

-----------------------------------------------------
## culling  
최적화를 위해 보이지 앟는 부분을 제거하는 기능으로, 와인딩 순서 알고리증을 사용한다.  
>와인딩이란?  
>다각형을 구성하는 버텍스간의 순서를 와인딩 순서하고 한다!  

 ![culling](https://i.stack.imgur.com/NYP5Q.png)
 
 
* 시계방향 와인딩 CW  
보이는 면을 결정하는 방식이다.  
다각형을 이루는 정점을 정면에서 바라볼때 시계방향 와인딩으로 그릴시, 물체를 그려야 할 면이라 판단하여 그리게 된다.  
그려진 면의 노말벡터는 카메라의 노말벡터 방향과 반대이다. 이 말의 뜻은 정점 노말벡터가 모두 카메라를 향하는 앞면에 해당한다는 뜻이다
 
* 반시계방향 와인딩 CCW  
보이지 않는 면을 결정하는 방식으로, 정면을 기준으로 했을때 뒷면은 반시계 방향이 되므로 은면처리하여 그리지 않는 부분이 된다.

-----------------------------------------------------
## Viewport transform  
![뷰포트 변환](https://static.javatpoint.com/tutorial/computer-graphics/images/computer-graphics-window-to-viewport-co-ordinate-transformation.png)  
뷰포트 변환은 NDCS(정규화좌표) 결과를 각각의 출력 장치에 매핑하는 단계이다.  
 
모니터 기준으로 width가 x, height가 y에 영향을 받으며, 평면에 그려질 도형의 순서는 [Z 버퍼 테스트](#z-버퍼-테스트)를 통해 결정되며 3D 공간에서 View volume에 존재하는 객체들을 Projection Plane에 투영하여 모니터에 출력한다. 

뷰포트 변환에서는 윈도우와 윈도우 내에 출력 공간을 지정할 viewport가 필요하다.    
* 윈도우와 viewport 크기가 대부분 동일하나, 그렇지 않은 경우도 존재함.  

-----------------------------------------------------
## Scan conversion  
![주사변환](https://www.cs.princeton.edu/courses/archive/fall99/cs426/lectures/pipeline/img068.gif)  
주사변환은 래스터화라고도 한다. 내가 그릴 그래픽스 객체를 디스플레이에 표시하기 위해 픽셀 데이터로 변환하는 과정을 뜻한다.

-----------------------------------------------------
## Z 버퍼 테스트
```
그려야 할 물체의 픽셀들이 다른 물체보다 앞에 있는지 판단하기 위한 테스트 기법  
```
![z버퍼테스트](https://larranaga.github.io/Blog/imagenes/z-buffer.png)  

Z read와 Z write로 나뉜다.  
* Z buffer : 시점으로부터 물체까지의 Z값을 저장해두는 기억 장치.  
* Z read : Z 버퍼에 기존 z 값 보다 앞에 있는지 판단한다.  
* Z write : read과정에서 새로운 버퍼가 Pass 되면 자신의 z 값으로 덮어씌워 그린다.  

-----------------------------------------------------
## stencil test

-----------------------------------------------------
## color mask


---
layout: post
title:  "HLSL 프로그래밍_7 DoubleSided과 Culling"
date:   2023-04-20 09:30:10 +0900
categories: shader
tags : Shader, Graphic
---
## Double Sided Rendering  
게임에서 카메라를 움직여 가까이 가져갈 때, 내부를 통과할시 안쪽이 투명하게 보이는걸 볼 수 있는데, 그 이유가 바로 이것이다.   
![image](https://user-images.githubusercontent.com/65288322/232411095-ca1c21a6-e9b8-42f0-83f0-cc9386739a2f.png)


일반적으로 한 면만 렌더링 하기 때문에 양면 렌더링을 실시할 경우 내부도 보이게 된다.  


### Face Culling  
컬링은 최적화 기법으로 카메라에 보여질 면을 선택하고 그 외는 렌더링 하지 않는 방법을 뜻한다.  
```cs  
//CullMode를 Enum 타입으로 가져온다.
[Enum(UnityEngine.Rendering.CullMode)] _cull("Culling",FLOAT) = 1;

Cull[_cull]
```  
![2023-04-17 16;17;20](https://user-images.githubusercontent.com/65288322/232412484-a0957de3-6b41-4a9f-96dc-068f3358b2c6.gif)  

culling이 적용된 걸 확인할 수 있다.  


### Normal 방향 뒤집기   
![image](https://user-images.githubusercontent.com/65288322/232410010-3ad46545-28c9-4735-84df-f5c0619ba070.png)  

면의 안쪽을 보면 내부가 어둡게 나타난다.  
이는 면의 방향 (normal vector)가 뒤집힌 상태이기 때문에 나타나는 결과이다.  
이를 해결하기 위해선 해당 정점이 뒤집혔는지 확인해야 한다.  

![image](https://user-images.githubusercontent.com/65288322/232410512-8f822be3-6aa0-478b-9b1c-fea4f0a5d9c9.png)  

> Fragment 단에서 해당 픽셀이 보고 있는 면 방향을 알려주는 시멘틱 FRONT_FACE_SEMANTIC  

그리고 normalWS에 해당 정점의 정보를 가져와 뒤집어준다. (-1 또는 1을 곱해서)  

```cs
lightingInput.normalWS = normalize(i.normalWS)  * IS_FRONT_VFACE(frontFace,1,-1);
```

![image](https://user-images.githubusercontent.com/65288322/232408680-6df5783d-dc47-446e-8fab-36269a14f552.png)   

내부에 조명이 적용되는걸 볼 수 있다.

그런데 한 가지 문제점이 또 생겼다.  
바로 shadow ance가 발생한것.


------

### Back face Shadow ance 제거

![image](https://user-images.githubusercontent.com/65288322/232408680-6df5783d-dc47-446e-8fab-36269a14f552.png)


![image](https://user-images.githubusercontent.com/65288322/232409483-b301ea9c-5ddf-4c3d-b23e-17ce66d92347.png)  
이를 해결하기 위해선 해당 그림처럼 normal vector와 viewDir(광원 방향)간의 각도를 기준으로 90도 이상일시 뒤집어줘야 한다.  

기존 LitPass에서는 **FRONT_FACE_TYPE frontFace : FRONT_FACE_SEMANTIC** 시멘틱을 통해 뒤집혀진 정점을 가져올 수 있었지만, Shadow pass에서는 vertex 단계에서 shadow position을 계산하기 때문에 fragment 단의 **FRONT_FACE_SEMANTIC** 시멘틱을 사용할 수 없다.  

결국 직접 뒤집힌 정점을 구한 뒤, 해당 정점 위치에 대응되도록 Bias 값을 적용시킬 수 밖에 없다.  


```cs
float3 FlipNormalBaseOnViewDir(float3 normalWS, float3 positionWS)
{
    float3 viewDirWS = GetWorldSpaceNormalizeViewDir(positionWS);
    // 카메라와 정점간의 거리 (viewPos - positionPos)

    return normalWS * (dot(normalWS,viewDirWS) < 0 ? -1 : 1);
    //dot 연산 결과가 0보다 작으면 각도가 90도 이상이므로 면을 뒤집어 줘야한다. -1값을 넣어준다.
    //결과를 바탕으로 normalFace를 뒤집은 후 발생한 shadow acne를 처리하기 위해
    //shadow bias 값을 반전시키기 위함이다.
}

normalWS = FlipNormalBaseOnViewDir(normalWS,positionWS);
//FlipNormalBaseOnViewDir() 함수를 통해 shadow depth에 사용됨

```

---


![image](https://user-images.githubusercontent.com/65288322/232408716-048aec50-50ba-403d-b365-86c78df327fc.png)  
>shadow bias를 반전시키고 난 후.

이제 양면 렌더링을 적용시켰을 때 오버헤드를 줄여야한다.  

이전 surface와 같이 FaceRenderType enum을 만들어 ShaderGUI단에서 선택할 수 있도록 해야한다.  
아래와 같이 enum을 만들고 shader 파일에 있는 _FaceRenderingMode 프로퍼티와 연결하자  

```cs
public override void OnGUI(MaterialEditor materialEditor, MaterialProperty[] properties)
    {
var faceProp = BaseShaderGUI.FindProperty("_FaceRenderingMode",properties, true);

faceProp.floatValue = (int)(FaceRenderType)
            EditorGUILayout.EnumPopup("FaceRendering Mode", (FaceRenderType)faceProp.floatValue);
}

FaceRenderType faceRenderingMode =
(FaceRenderType)material.GetFloat("_FaceRenderingMode");


        if (faceRenderingMode == FaceRenderType.FrontOnly)
            material.SetInt("_Cull", (int)CullMode.Back);
        else
            material.SetInt("_Cull", (int)CullMode.Off);

        if (faceRenderingMode == FaceRenderType.DoubleSided)
            material.EnableKeyword("_DOUBLE_SIDED_NORMALS");
        else
            material.DisableKeyword("_DOUBLE_SIDED_NORMALS");
```

-----


다음으로 shader 파일에  _FaceRenderingMode 프로퍼티 선언과 shader_feature 명령어를 정의해주자   

```cs
[HideInInspector] _FaceRenderingMode("Face Render Mode",Float) = 0

SubShader
{
    Pass
    {
        Name "ForwardLit"
        #pragma shader_feature_local _DOUBLE_SIDED_NORMALS
    }
    Pass
    {
        Name "ShadowCaster"
        #pragma shader_feature_local _DOUBLE_SIDED_NORMALS
    }
}
```

shader 파일에 명령어 정의가 완료되면 ForwardLit 파일과 ShadowCaster파일에 아래와 같이 오버헤드가 발생할 부분에 명령어 #ifdef와 #endif 처리를 해주면 된다.  

![image](https://user-images.githubusercontent.com/65288322/233002687-766a776e-f13a-44fb-8486-e501ca4b2963.png)

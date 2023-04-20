---
layout: post
title:  "HLSL 프로그래밍_5 Transparent"
date:   2023-04-14 11:27:10 +0900
categories: shader
tags : Shader, Graphic
---

## Transparency  
기존에는 Opaque 재질만 다뤄봤다면 이제는 투명도 Tranparency를 다뤄볼 차례이다.  
우선 투명도를 사용하기 위해서는 Alpha Blending 명령어를 사용해야 하는데, 이는 알파 채널을 사용하겠다는 의미이다.  

```c#
  SubShader
{
        Tags {"RenderPipeline" = "UniversalPipeline"}
        Pass {
            Name "ForwardLit"
            Tags{"LightMode" = "UniversalForward"}

            Blend SrcAlpha OneMinusSrcAlpha
         }
}
```
> ### Blend SrcAlpha OneMinusSrcAlpha
> Blend 명령어는 rasterizer가 대상 화면에 있는 컬러와 fragment 결과물을 어떻게 혼합시킬지에 대한 역할을 수행한다.

즉, fragment 함수에서 반환되는 색상을 source color, 렌더 대상에 저장된 색상을 destination이라 칭한다.

![image](https://user-images.githubusercontent.com/65288322/230815088-6b2906e0-f76e-4ddc-a74e-13714e305274.png)

이 과정을 수행할 때, rasterizer는 source color와 destnaton 컬러에 각각 숫자를 곱한 후, 결과물을 서로 더하여 결과물을 덮어씌운다.

![image](https://user-images.githubusercontent.com/65288322/230824695-acc22f1f-9013-4c30-b6fa-ec52d6cb32cb.png)

당연히 유니티에서는 SrcAlpha와 OneMinusSrcAlpha를 통해서 소스 컬러의 투명도를 제공해준다.  

너무 고마운 유니티쨩 ^-^  
한번 실행시켜보도록 하자


![image](https://user-images.githubusercontent.com/65288322/230829382-e833a0ed-34df-4412-8f98-1c3f57f76e61.png)
>보이긴 하는데 뭔가 좀 이상한...

앞서 나온 내용대로 소스 컬러와 목표 컬러를 혼합하면 제대로 나와야할텐데.. 렌더링이 이상하게 된다.  

그 이유는 rasterizer가 depth buffer에 반투명 오브젝트의 위치값을 저장하게 되면서 뒤에 있는 물체는 fragment를 수행하지 않기 때문이다.

당연하게도 fragment가 수행되지 않은곳은 렌더링 자체가 되지 않으니 섞을수가 없는것이다!

이를 수정하기 위해서는 **ZWrite Off**를 사용해야 한다.  

```cs
 Pass
        {
            Name "ForwardLit"
            Tags{"LightMode" = "UniversalForward"}

            Blend SrcAlpha OneMinusSrcAlpha
            ZWrite Off
         }
```


이 명령어는 rasterizer가 해당 Pass의 데이터를 depth buffer에 저장하지 않도록 제어해준다.  

![image](https://user-images.githubusercontent.com/65288322/230835364-92bd7f17-a800-4958-a7ca-eeb9425dd9a4.png)  
만약 이렇게 나타난다면 Render Queue를 확인해보자  

![image](https://user-images.githubusercontent.com/65288322/230835679-5ccd65c3-1bff-40bd-9a70-9b3a3e6834cf.png)  

![image](https://user-images.githubusercontent.com/65288322/230834695-33f028e5-a53d-478a-8c36-50c06d0c3b9e.png)  
Transparency로 바꿔주면 skyBox가 제대로 렌더링 되는걸 볼 수 있다.
와우 잘 된다!! 👍

## Render Queues  
렌더큐를 통해 물체가 그려질 순서를 정할 수 있다.
![image](https://user-images.githubusercontent.com/65288322/230836853-861e6392-fb34-4891-9c44-1e4f733cec8d.png)
> 픽셀을 혼합하기 위해 Transparent 개체가 가장 마지막에 그려진다

또한 shader 내부에서 이를 처리할 수 있는데,  
![image](https://user-images.githubusercontent.com/65288322/230854920-097fb9c2-7ff4-43d3-af3d-6dc87d507b0a.png)  

SubShader 내부에 Tags를 지정할 때, 렌더 타입과 Queue를 설정해줄수 있다.  


##Custom Inspector
마테리얼 속성을 Opaque와 Transparent로 전환시킬때,  render queue, texture, shadow 등등 바꿔야 할것이 많아진다.  
물론 일일이 바꿔줄 수 있지만 한번에 전환시킬 커스텀 인스펙터를 만들어 이를 처리할 수 있다.  
이를 위해선 C# 스크립트와 Shader 파일을 수정해주어야 한다.  


우선 shader파일에 ZWrite, RenderType, Blend를 바꿔주기위한 변수를 생성해야한다.  
```cs
Properties
{
        [HideInInspector] _SourceBlend("Source blend",Float) = 0
        [HideInInspector] _DestBlend("Destination blend",Float) = 0
        [HideInInspector] _ZWrite("ZWrite",Float) = 0

        [HideInInspector] _SurfaceType("Source type",Float) = 0
}

{
        //프로퍼티의 값을 가져와 명령을 수행한다.
        Blend[_SourceBlend][_DestBlend]
        ZWrite[_ZWrite]
}
```


다음으로 C# 스크립트를 작성해준다.  
``` cs
public class ShaderCustomInspector : ShaderGUI
{
    public enum SurfaceType
    {
        Opaque,
        Transparent
    }
    public override void OnGUI(MaterialEditor materialEditor, MaterialProperty[] properties)
    {
        //MaterialEditor을 통해 현재 타겟 material을 가져올 수 있음.
        Material material = materialEditor.target as Material;

        //string과 이름이 같은 변수를 properties[]에서 가져옴.
        var surfaceProp = BaseShaderGUI.FindProperty("_SurfaceType", properties, true);

        //Begin ~ EndChange는 해당 블록안 내용에 대해서만 GUI의 변경에 대해 확인한다.
        EditorGUI.BeginChangeCheck();

        //property중 surfaceType의 value를 GUIEnumPopup의 값으로 변경시켜줌.
        surfaceProp.floatValue = (int)(SurfaceType)
            EditorGUILayout.EnumPopup("Surface_Type", (SurfaceType)surfaceProp.floatValue);


        //코드블럭 내부의 GUI가 변경되는 경우 아래의 코드를 수행함.
        if (EditorGUI.EndChangeCheck())
        {
            UpdateSurfaceType(material);
        }
        //OnGUI 렌더링 default설정
        base.OnGUI(materialEditor, properties);
    }


    //Shader File Edit
    private void UpdateSurfaceType(Material material)
    {
        //material 내부의 프로퍼티 _SurfaceType를 가져와 SurfaceType Enum값으로 적용.
        SurfaceType surface = (SurfaceType)material.GetFloat("_SurfaceType");

        switch(surface)
        {
            case SurfaceType.Opaque:
                material.renderQueue = (int)RenderQueue.Geometry;

                //태그를 재정의 할 수 있음
                material.SetOverrideTag("RenderType", "Opaque");

                //이미 정의되어있는 BlendMode Enum
                material.SetInt("_SourceBlend",(int)BlendMode.One);
                material.SetInt("_DestBlend",(int)BlendMode.Zero);
                material.SetInt("_ZWrite",1);

                //SetShaderPassEnabled를 통해 label pass를 제어할 수 있다.
                material.SetShaderPassEnabled("ShadowCaster",true);
                break;

            case SurfaceType.Transparent:
                material.renderQueue = (int)RenderQueue.Transparent;
                material.SetOverrideTag("RenderType", "Transparent");
                material.SetInt("_SourceBlend", (int)BlendMode.SrcAlpha);
                material.SetInt("_DestBlend", (int)BlendMode.OneMinusSrcAlpha);
                material.SetInt("_ZWrite", 0);
                material.SetShaderPassEnabled("ShadowCaster", false);
                break;
        }
    }

}
```

마지막으로 Shader파일 내부에 CustomEditor 명령어를 넣어준다.  
![image](https://user-images.githubusercontent.com/65288322/231376716-a7eeee23-d815-43b5-b967-23133481ad5a.png)  

결과를 확인해보면?..

![2023-04-12 16;01;41](https://user-images.githubusercontent.com/65288322/231377749-e94a4dc6-af3d-4bb7-a42c-079b46f33327.gif)  
Enum을 바꿔주면 GUI의 변경을 Begin ~ EndChange 블럭에서 판단하여 switch문을 수행한다.  


쉐이더를 새로 할당해주었을 때 변경되지 않을 수 있는데, 이럴 땐 **AssignNewShaderToMaterial** 함수를 이용해 Material이 할당될 때 처리를 할 수 있다.  

```cs
  public override void AssignNewShaderToMaterial
      (Material material, Shader oldShader, Shader newShader)
      {
          base.AssignNewShaderToMaterial(material, oldShader, newShader);

          if (newShader.name == "Custom/KMSshader")
          {
              UpdateSurfaceType(material);
          }
      }
```

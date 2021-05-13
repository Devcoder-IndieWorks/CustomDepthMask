# CustomDepth 기반 Masking 영역 만들기

----------------------------------------------------------------------------------------------------------------------------------------------------------------

### 개요

![](https://github.com/Devcoder-IndieWorks/CustomDepthMask/blob/master/ScreenShots/CustomDepth_Masking_Scene.png)

빨간 박스 영역내 검은색으로 된 부분이 UE4 CustomDepth 기능을 이용하여 전체 씬 영역에서 마스킹 영역으로 사용할 부분을 만든 것이다.

Composure의 Final Compositing Material에서 마스킹 영역에 필요한 미디어 데이터를 넣을 수 있어, 다양한 용도로 사용 가능하다. 또한 마스킹 영역에 해당 되는 오브젝트의 뒷쪽에 위치하는 오브젝트와 겹치는 부분을 Blur Edge로 처리 하여 마스킹 영역과 합성시 좀 더 자연스러움 처리가 가능하다.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

### 구현

UE4 CustomDepth 기능을 이용하기 때문에 Material 기반으로 구현.

화면 영역에서 마스킹 될 영역을 만들기 위해 **Post Process Material**인 **PP_CustomDepthMask Material**을 만든다.

우선 Post Process Material 관련 설정을 다음 그림과 같이 설정 해 준다.

![](https://github.com/Devcoder-IndieWorks/CustomDepthMask/blob/master/ScreenShots/CustomDepthMask_PostProcessMaterial.png)

마스킹 될 영역에 해당하는 오브젝트보다 앞에 존재하며, 카메라쪽으로 가깝게 있는 오브젝트에 의해 마스킹 될 영역은 가려져야 하므로 이 가려짐 여부를 0 ~ 1 사이값으로 구한다.

![](https://github.com/Devcoder-IndieWorks/CustomDepthMask/blob/master/ScreenShots/CustomDepthMask_Occluded.png)

마스킹 영역의 가장 자리를 블러처리 하기 위해 **MF_SpiralBlur Material Function**을 호출하여 블러링 된 CustomDepth 값을 얻어와 0 ~ 1 사이의 값으로 만든다.

![](https://github.com/Devcoder-IndieWorks/CustomDepthMask/blob/master/ScreenShots/CustomDepthMask_BlurEdge.png)

이렇게 구해진 값들(**가려짐 여부, Blur Edge된 CustomDepth 값**)을 곱하기 합산하여 최종적인 가장자리가 블러링된 마스킹 영역을 구한다.

![](https://github.com/Devcoder-IndieWorks/CustomDepthMask/blob/master/ScreenShots/CustomDepthMask_Output.png)

Blur Edge처리를 하는 MF_SpiralBlur Material Function 구현은 다음 그림과 같다.

![](https://github.com/Devcoder-IndieWorks/CustomDepthMask/blob/master/ScreenShots/MF_SpiralBlur_Material_Function.png)

Custom Node에 들어갈 실제 Bluring 처리에 대한 Shader Code는 다음과 같다.

<pre>
    <code>
 float2 nUV = UV;
 int i=0;
 float StepSize = Detail / (int) DetailIterations;
 float CurDetail=0;
 float2 CurOffset=0;
 float3 CurColor=0;
 float SubOffset = 0;
 float TwoPi = 6.283185;
 float accumdist=0;
 int TexIndex = 13;
 if (DetailIterations < 1)
 {
   return SceneTextureLookup(UV, TexIndex, false);
 }
 else
 {
   while (i < (int) DetailIterations)
   {
     CurDetail += StepSize;
     for (int j = 0; j < (int) RadialSteps; j++)
     {
       SubOffset +=1;
       CurOffset.x = cos(TwoPi*(SubOffset / RadialSteps));
       CurOffset.y = sin(TwoPi*(SubOffset / RadialSteps));
       nUV.x = UV.x + CurOffset.x * CurDetail;
       nUV.y = UV.y + CurOffset.y * CurDetail;
       float distpow = pow(CurDetail, KernelPower);
       CurColor += ceil(SceneTextureLookup(nUV, TexIndex, false))*distpow;
       accumdist += distpow;
     }
     SubOffset +=RadialOffset;
     i++;
   }
   CurColor = CurColor;
   CurColor /=accumdist;
   return CurColor;
 }
    </code>
</pre>

Scene Texture에 대한 참조는 UE4 엔진 코드에 정의된 SceneTextureLookup() 함수를 통해 접근해서 입력된 UV 좌표에 해당하는 Pixel 정보를 얻어온다.

UV 좌표에 대한 값도 Custom Node에 Shader Code를 작성해서 구한다.

<pre>
    <code>
    return GetDefaultSceneTextureUV(Parameters, 14);
    </code>
</pre>

다음은 마스킹 영역을 구해, 다른 레이어에서 구해진 이미지들과 합성 하기 위해 마스킹 영역에 대한 이미지만 캡쳐해서 처리 할 수 있는 레이어 Blueprint Actor를 구현 한다.

이 Blueprint Actor가 BP_CgMaskCaptureCompElement Actor이다. 이 Actor에 마스킹 영역을 구하기 위해 서로 관련이 있는 Level Object 목록이 주어지면, 현재 카메라 위치와 마스킹 영역을 나타내는 오브젝트 사이에 존재하는 Level Object들만 추려서 캡쳐 목록을 다시 만들어 SceneCaptureComponent를 통해 캡쳐해서 원하는 마스킹 영역을 구한다.

![](https://github.com/Devcoder-IndieWorks/CustomDepthMask/blob/master/ScreenShots/Msking_Result.png)


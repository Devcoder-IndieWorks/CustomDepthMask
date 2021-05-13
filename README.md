# CustomDepth 기반 Masking 영역 만들기

----------------------------------------------------------------------------------------------------------------------------------------------------------------

### 개요

![](https://github.com/Devcoder-IndieWorks/CustomDepthMask/blob/master/ScreenShots/CustomDepth_Masking_Scene.png)

빨간 박스 영역내 검은색으로 된 부분이 UE4 CustomDepth 기능을 이용하여 전체 씬 영역에서 마스킹 영역으로 사용할 부분을 만들 것이다.

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




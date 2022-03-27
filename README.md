THE PBR GUIDE - PART 1
===

THE PBR GUIDE BY ALLEGORITHMIC - PART 1
---


# Light and Matter : The theory of Physically-Based Rendering and Shading
## TABLE OF CONTENTS
* 광선(Light Rays)
* 흡수(Absorption)와 산란(Scattering)[^scattering_diffuse_difference] - 투명도(Transparency)와 반투명(Translucency)
* 확산(Diffuse)과 정반사(Specular Reflection) - 미세면 이론(Microfacet Theory)
* 색상(Color)
* 양방향 반사도 분포 함수(BRDF)
* Energy Conservation
* Fresnel Effect - F0 (Fresnel Reflectance at 0 Degrees)
* Conductors and Insulators - Metals and Non Metal
* Linear Space Rendering
* Key characteristics of PBR


# THE THEORY OF PHYSICALLY-BASED RENDERING AND SHADING
 빛은 파동과 입자의 속성을 모두 나타낼 수 있기 때문에 복잡한 현상이다. 결과적으로 그런 현상을 설명하기 위해 다양한 모델이 만들어졌다.

 텍스처 아티스트로서 우리는 빛과 물질의 상호 작용을 설명하는 빛의 광선 모델(ray model of light)에 관심이 있다. 우리의 임무는 서페이스를 설명하는 텍스쳐를 만드는 것이기 때문에 광선이 서페이스 물질과 상호 작용하는 방식을 이해하는 것이 중요하다. 우리가 만든 텍스쳐와 재질은(textures and materials) 가상 세계에서 빛과 상호 작용한다. 빛이 어떻게 작용하는지 더 많이 이해할수록 텍스처가 더 좋아 보일 것 이다.

 이 가이드에서는 물리 기반 렌더링(PBR) 모델의 물리학에 대한 이론을 논의한다. 우리는 광선의 행동양식을 조사하는 것으로 시작하여 PBR의 주요 특성을 정의하는 작업을 할 것이다.


## Light Rays
 빛의 광선 모델은 광선이 공기와 같은 균질한 투명 매질(homogeneous transparent medium)[^homogeneous_medium]에서 직선의 궤적을 갖는다는 것을 나타낸다. 광선 모델은 또한 광선이 불투명한 물체(opaque objects)와 같은 서페이스를 만날 때 또는 공기에서 물을 지나는 것과 같이 다른 매질[^매질]을 통과할 때 예측 가능한 방식으로 행동한다고 말한다.

 이를 통해 광선이 시작지점에서 열과 같은 다른 형태의 에너지로 변하는[^conversion] [^albedo] 다른 지점으로 이동할 때 광선의 경로를 시각화할 수 있다.

 |albedo table||
 |----|----|
 |Material|Albedo|
 |snow|80-90%|
 |cloud|60-90%|
 |desert|30%|
 |meadow|20%|

 서페이스에 닿는 광선을 입사광선이라고 하고 그것이 닿는 각도를 입사각이라고 한다(Figure01).

<!--
이미지 기본형
![figure01](/img/figure01.png)

가운데 정렬하기(캡션 포함)
<img> 태그로 이미지를 첨부하고, 이미지 태그를 p태그로 감싼다.
-->
<p align="center">
  <img src="/img/figure01.png" alt="figure01" width="75%" height="75%" /> 
  <p align="center">
  Figure 01: 입사각(Angle of incidence), 입사광선과 반사광선(incident and reflected rays)
  </p>
</p>

 광선은 두 매질 사이의 평면 경계면에 입사한다. 광선이 서페이스에 닿으면 다음 이벤트 중 하나 또는 모두가 발생할 수 있다.

 1. 광선은 서페이스에서 반사되어 다른 방향으로 이동한다. 이때 광선은 "반사각은 입사각(반사광)과 같다"는 반사의 법칙을 따른다.

 2. 광선은 직선(굴절광)의 궤적으로 한 매질에서 다른 매질로 통과한다.

 이 지점에서 광선은 반사와 굴절(reflection and refraction)의 두 방향으로 나뉜다. 서페이스에서 광선은 반사되거나 굴절되며 결국 두 매질에 흡수(Absorption)될 수 있다. 그러나 물질의 표면에서는 흡수가 일어나지 않는다.

[^homogeneous_medium]: homogeneous medium : 균질 매질. 설탕물, 표준대기상태의 공기로 채워진 용기와 같이 혼합물 전체가 같은 조성을 가지는 매질.(장소와 관계가 없는 매질.)

[^매질]: 매질(medium) : 힘이나 파동 등의 물리현상을 전달하는 작용을 하는 물질 또는 공간. 장(선형/비선형), 공간(균질/비균질), 방향성(등방성/<이방성, 비등방성>), 탄성(탄성/비탄성) 의존성에 따라 성질이 구분된다. 매질의 물리적 성질에 따라 서로 다른 매질의 경계면에서는 반사(Reflection), 굴절(Refraction), 산란(Scattering), 투과(Transmission), 흡수(Absorption) 등의 현상이 일어난다. 

[^conversion]: 서페이스에서는 빛의 반사(reflection) 뿐만 아니라 흡수(Absorption, absorb)도 일어난다. 빛 에너지는 몸체의 분자에 흡수되어 운동 에너지로 변환되고, 분자의 움직임이 증가하면 주변으로 복사되는 열이 발생한다.

[^albedo]: 알베도(albedo) : 이러한 흡수 원리는 일상 생활으로 옮겨볼 수 있다. 검은색 티셔츠는 흰색 티셔츠보다 햇빛을 더 많이 흡수한다. 이것이 우리가 여름에 검은색 셔츠를 입고 땀을 더 많이 흘리는 이유이다. 몸체의 알베도는 흡수된 햇빛의 비율에 영향을주어 다양한 스펙트럼 범위에서 재질의 반사 정도를 측정한다 . 100%의 알베도는 흡수가 일어나지 않음을 나타낸다. 따라서 0%의 알베도는 반사가 없음을 나타낸다. 테이블 참조.

[^scattering_diffuse_difference]: 산란(Scattering)과 확산(Diffuse)의 차이 : 산란은 "어떤 매질에 빛이 입사되어 흡수되고 흡수된 매질의 원자를 파원으로 하여 사방으로 빛이 재방출되는 현상"을 의미한다.
확산은 "어떤 매질에 일정 방향에서 입사한 빛이 반사 또는 투과된 후, 모든 방향으로 퍼지면서 진행하는 현상"이다. 확산은 난반사(Diffuse Reflection)를 일으킨다.


## Absorption and Scattering (Transparency and Translucency)
 불균일한 매질(inhomogeneous medium)이나 반투명 물질로 이동할 때 빛은 다음과 같이 흡수되거나 산란될 수 있다.

 빛이 흡수되면(absorbed) 다른 형태의 에너지(보통 열)로 바뀌면서 빛의 강도가 감소한다. 파장을 근거로하여 흡수되는 빛의 양에 따라 색이 변하지만 광선의 방향은 변하지 않는다.

 빛이 산란되면(scattered) 광선 방향이 무작위로 바뀌고 편차의 정도는 재질에 따라 다르다. 산란은 빛의 방향을 무작위로 지정하지만 강도는 변화시키지 않는다. 신체부위중 귀는 이 현상의 좋은 예이다. 귀가 얇아서(흡수율이 낮음) Figure 02와 같이 귀 뒤쪽에서 산란광이 방출되는 것을 볼 수 있다.

 산란이 없고 흡수가 낮으면 광선이 표면을 직접 통과할 수 있다. 이것은 유리의 경우이다. 예를 들어 깨끗한 수영장에서 수영하고 있다고 할때, 사람은 눈을 뜨고 맑은 물을 통해 먼 거리를 볼 수 있다. 그러나 같은 수영장이 상대적으로 더러우면 먼지 입자가 빛을 산란시켜 물의 투명도를 낮추고 결과적으로 볼 수 있는 거리가 줄어든다.

 그러한 매질/물질(medium/material)에서 더 많은 빛이 이동할수록 더 많이 흡수 및/또는 산란된다. 따라서 물체의 두께는 빛이 얼마나 흡수되거나 산란되는지에 큰 역할을 한다. 두께 맵(thickness map)은 Figure 03과 같이 셰이더에 오브젝트의 두께를 설명하는 데 사용할 수 있다.

<p align="center">
  <img src="/img/figure02.jpg" alt="figure02" width="75%" height="75%" /> 
  <p align="center">
  Figure 02: 귀 뒤쪽으로부터 방사되는 산란광
  </p>
</p>

<p align="center">
  <img src="/img/figure03.png" alt="figure03" width="75%" height="75%" /> 
  <p align="center">
  Figure 03: 섭스턴스 페인터 내에서 서페이스 아래 산란과 함께 사용되는 두께 맵
  </p>
</p>


## Diffuse and Specular Reflection
 정반사(Specular Reflection)란 광선(Light Rays) 섹션에서 논의한 것처럼 서페이스에서 반사된 빛을 말한다. 광선은 서페이스에서 반사되어 다른 방향으로 이동한다. 그것은 반사의 법칙을 따른다. 즉, 완벽하게 평평한 표면에서 반사각은 입사각과 같다. 그러나 대부분의 표면은 불규칙하고 반사 방향은 표면 거칠기(surface roughness)에 따라 무작위로 달라진다. 이 경우 빛의 방향이 바뀌지만 빛의 강도는 일정하게 유지된다.

 더 거친 서페이스는 더 크고 더 흐릿하게 보이는 하이라이트를 가질 것이다. 매끄러운 표면은 정반사의 초점을 유지하고 적절한 각도에서 볼 때 더 밝거나 강렬하게 보인다. 그러나 두 경우 모두 동일한 총 광량이 반사된다(Figure 04).

<p align="center">
  <img src="/img/figure04.jpg" alt="figure04" width="75%" height="75%" /> 
  <p align="center">
  Figure 04: 반사 방향은 표면 거칠기에 따라 무작위로 달라진다.
  </p>
</p>

 굴절(Refraction)은 광선 방향의 변화이다. 빛이 한 매질에서 다른 매질로 이동할 때 속도와 방향이 바뀐다. 굴절률 또는 IOR(Index Of Refraction)은 광선이 진행하는 방향의 변화를 설명하는 광학 측정이다. 기본적으로 IOR 값은 광선이 한 매체를 통해 다른 매체로 통과할 때 구부러지는 정도를 결정하는 데 사용된다. 예를 들어, 물의 IOR은 1.33인 반면 판유리의 IOR은 1.52이다. Figure 05에서 물 한 컵에 담긴 빨대의 렌더링을 볼 수 있다. 빨대는 빛이 다른 매질(공기, 물, 유리)을 통과할 때 굴절로 인해 구부러진 것처럼 보인다.

<p align="center">
  <img src="/img/figure05.jpg" alt="figure05" width="75%" height="75%" /> 
  <p align="center">
  Figure 05: 빨대가 굴절로 인해 구부러진 것처럼 보인다.
  </p>
</p>

 확산 반사(Diffuse Reflection)는 굴절된 빛이다. 광선은 한 매질에서 다른 매질로 전달된다. 광선이 오브젝트에 들어간다고 할 때, 빛은 이 오브젝트 내에서 여러 번 산란된다. 그것은 마침내 물체 밖으로 다시 굴절되어 처음에 들어갔던 거의 같은 지점에서 원래의 매질로 되돌아간다(Figure 06).

 확산 물질은 흡수성이다. 굴절된 빛이 그러한 물질에서 너무 오래 이동하면 완전히 흡수될 수 있다. 빛이 이 물질을 빠져나가면 진입 지점에서 아주 작은 거리만 이동했을 가능성이 크다.

 따라서 들어가는 지점과 나오는 지점간의 거리는 무시할 수 있다. 전통적인 셰이딩에서 확산 반사에 사용되는 램버트(Lambertian) 모델은 서페이스 거칠기를 고려하지 않는다. 그러나 오렌-나야르(Oren-Nayar) 모델과 같은 다른 확산 반사 모델은 이러한 거칠기를 반영한다.

 높은 산란과 낮은 흡수(high scattering and low absorption)를 모두 갖는 재료를 참여 매질(participating media) 또는 반투명 물질(translucent materials)이라고 한다. 예를 들어 연기, 우유, 피부, 옥 및 대리석이 있다. 광선의 들어오고 나가는 지점 사이의 차이가 더 이상 무시할 수 없는 것으로 간주되는 피하산란(subsurface scattering)의 추가 모델링으로 우유, 피부, 옥 및 대리석과 같은 렌더링이 가능할 수 있다. 연기 또는 안개와 같이 매우 다양하고 산란 및 흡수가 매우 낮은 매질을 정확하게 렌더링하려면 몬테카를로 시뮬레이션(Monte Carlo simulations)[^Monte]과 같은 훨씬 더 느린 방법이 요구될 수 있다.

<p align="center">
  <img src="/img/figure06.png" alt="figure06" width="75%" height="75%" /> 
  <p align="center">
  Figure 06: 한 매질에서 다른 매질로 이동하는 광선이 물체 내부에서 산란
  </p>
</p>

[^Monte]: 몬테카를로 시뮬레이션(Monte Carlo simulations) : 반복되는 랜덤 샘플링을 사용하여 발생하는 결과 범위의 가능성을 얻을 수 있는 계산 알고리즘의 한 유형. 몬테카를로 메서드 또는 다중 확률 시뮬레이션이라고도 하는 몬테카를로 시뮬레이션은 불확실한 사건의 가능한 결과를 추정하는 데 사용되는 수학적 기법이다.

### Microfacet Theory
 이론적으로 확산 반사(diffuse)와 정반사(specular reflection) 모두 광선이 매질과 교차하는 표면의 불규칙성(surface irregularities)에 따라 달라진다. 그러나 실제로는 난반사(diffuse reflection)에 대한 거칠기의 영향은 물질 내부에서 발생하는 산란 때문에 덜 가시적이다. 결과적으로 광선이 반사되어 나가는 방향은 표면 거칠기와 입사 방향에 상당히 독립적이다.[^diffuse_ref] 확산 반사에 대한 가장 일반적인 모델(Lambertian)은 거칠기를 완전히 무시한다.

 이 가이드에서는 이러한 표면 불규칙성을 표면 거칠기(surface roughness)[^roughness]라고 한다. 표면 불규칙성은 사용 중인 PBR 워크플로우에 따라 거칠기, 부드러움, 광택 또는 미세 표면(roughness, smoothness, glossiness or micro-surface)을 비롯한 여러 다른 이름을 가질 수 있다. 이 모든 용어는 서브 텍셀[^texel](sub-texel)의 기하학적 세부사항인 서페이스의 동일한 측면(surface irregularities)을 설명한다.

 이러한 표면 불규칙성은 사용 중인 워크플로에 따라 거칠기(roughness) 또는 광택맵(glossiness map)에서 작성된다. 물리 기반 BRDF(physically-based BRDF)[^BRDF]는 표면이 미세면(microfacet)이라고 하는 다양한 방향의 소규모 평면 세부 표면(small-scaled planar detail surfaces of varying orientation)으로 구성되어 있다고 가정하는 미세면 이론(microfacet theory)을 기반으로 한다. 이 작은 평면 각각은 법선(normal)을 기반으로 한 방향(single direction)을 향해 빛을 반사한다(Figure 07).

 표면 법선(surface normal)이 조명 방향과 보기 방향(light direction and view direction) 사이의 정확히 중간 방향인 미세면은 가시광선을 반사한다. 그러나 미세표면 노말과 하프 노말이 동일한 경우 그림 07에서와 같이 일부 미세면이 그림자(빛 방향, light direction) 또는 마스킹(보기 방향, view direction)에 의해 차단되므로 모든 미세면이 기여하는 것은 아니다.

 미세한 수준(microscopic level)의 표면 불규칙성은 광 확산을 유발한다. 예를 들어, 흐릿한 반사는 산란된 광선으로 인해 발생한다. 광선은 평행하게 반사되지 않으므로 정반사를 흐리게 인식한다(Figure 08).

<p align="center">
  <img src="/img/figure07.png" alt="figure07" width="75%" height="75%" /> 
  <p align="center">
  Figure 07: 미세면 이론에 기반한 Physically-based BRDF
  </p>
</p>
<p align="center">
  <img src="/img/figure08.png" alt="figure08" width="75%" height="75%" /> 
  <p align="center">
  Figure 08: 산란된 광선에 의한 흐릿한 반사
  </p>
</p>

[^diffuse_ref]: 디퓨즈 리플렉션은 물질 내부에서 발생하는 산란으로 인해 표면의 거칠기를 무시 가능한 수준으로 무작위적인 방향으로 반사된다.

[^roughness]: Roughness : 또한 대부분의 대중적인 렌더러에서 표면에 대한 거칠기를 Roughness라는 이름으로 나타낸다. 이때 일반적으로 거칠기가 증가하면 더 많은 확산으로 인해 부드러운 셰입의 반사를 얻을 수 있고, 거칠기가 감소하면 선명하고 날카로운 셰입의 반사를 얻을 수 있다.

[^texel]: Texel : 텍셀, 텍스처 엘리먼트 또는 텍스처 픽셀은 텍스처 맵의 기본 단위이다. 텍스처는 이미지가 픽셀 배열로 표현되는 것처럼 텍스처 공간을 나타내는 텍셀 배열로 표현된다.

[^BRDF]: BRDF : 양방향 반사도 분포 함수(Bidirectional Reflectance Distribution Function)는 빛이 불투명한 표면에서 어떤 방식으로 반사되는지를 정의하는 4차원 함수이다.


## Color

 표면의 가시적인 색상은 광원에서 방출되는 파장에서 비롯된다. 이러한 파장은 물체에 흡수되고, 정반사 및 확산 반사된다. 그 흡수되는 파장 외 나머지 반사 파장을 우리가 색상으로 보는 것이다.

 예를 들어, 사과의 피부는 대부분 붉은 빛을 반사한다. 빨간색 파장만 사과 껍질 외부로 다시 산란되고 나머지 파장은 흡수된다(Figure 09).

 사과는 또한 전기를 전도하지 않는 물질(유전체, dielectrics)로서(사과 껍질과 같이) 정반사가 파장에 거의 독립적이기 때문에 광원과 동일한 색상의 밝은 정반사 하이라이트를 갖는다. 이러한 물질의 경우 정반사에 색상이 지정되지 않는다. 이후 섹션에서 다양한 유형의 물질(금속 및 유전체)에 대해 논의할 것이다.

 Substance PBR 셰이더는 GGX 미세면 분포(GGX microfacet distribution)를 사용한다.

<p align="center">
  <img src="/img/figure09.png" alt="figure09" width="75%" height="75%" /> 
  <p align="center">
  Figure 09: 적색 파장이 눈으로 반사된다.
  </p>
</p>


## BRDF

 양방향 반사율 분포 함수(BRDF)는 표면의 반사율 속성을 설명하는 함수이다. 컴퓨터 그래픽에는 다양한 BRDF 모델이 있으며 그 중 일부는 물리적으로 설득력있지 않다. BRDF가 물리적으로 그럴듯해지기 위해서는 에너지를 보존(energy conserving)하고 상호성을 나타내야 한다. 상호성은 BRDF의 결과에 영향을 미치지 않고 들어오는 광선과 나가는 광선이 서로의 반전으로 간주될 수 있다는 헬름홀츠 상호성 원칙(Helmholtz Reciprocity Principle)를 나타낸다.

 섭스턴스의 PBR 쉐이더에서 사용하는 BRDF는 Disney의 principled reflectance model을 기반으로 한다. 이 모델은 GGX 미세면 분포(GGX microfacet distribution)를 기반으로 한다. GGX는 반사 분포 측면에서 더 나은 솔루션 중 하나를 제공한다. 하이라이트에서 더 짧은 피크와 폴오프에서 더 긴 꼬리를 사용하여 더 사실적으로 보인다(Figure 10).























test line-----------------

    code block

test line-----------------

## reference
* <https://www.fis.uni-bonn.de/en/recherchetools/infobox/professionals/what-remote-sensing/reflection-and-absorption>
* <https://knowledge.autodesk.com/support/maya-lt/learn-explore/caas/CloudHelp/cloudhelp/2020/ENU/MayaLT-LightingShading/files/GUID-9158B659-5CA0-413B-8AD6-194E35A10FD8-htm.html>
* <https://www.scienceall.com/%EB%A7%A4%EC%A7%88medium-%E5%AA%92%E8%B3%AA/>
* <http://www.ktword.co.kr/test/view/view.php?m_temp1=6336>
* <https://ko.strephonsays.com/dispersion-and-vs-diffusion-5788>
* <https://optical-engineering.tistory.com/entry/%EB%B9%9B%EC%9D%98-%EC%82%B0%EB%9E%80-Scattering-%EA%B3%BC-%ED%99%95%EC%82%B0-Diffusing%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90>
* <https://www.scienceall.com/%ED%88%AC%EA%B3%BCtransmission/>
* <https://www.ibm.com/kr-ko/cloud/learn/monte-carlo-simulation>
* <https://ko.wikipedia.org/wiki/%EB%AA%AC%ED%85%8C%EC%B9%B4%EB%A5%BC%EB%A1%9C_%EB%B0%A9%EB%B2%95>
* <https://substance3d.adobe.com/tutorials/courses/the-pbr-guide-part-1>
* <https://en.wikipedia.org/wiki/Texel_(graphics)>
* <https://en.wikipedia.org/wiki/Bidirectional_reflectance_distribution_function#:~:text=The%20bidirectional%20reflectance%20distribution%20function,and%20in%20computer%20vision%20algorithms.>
* 

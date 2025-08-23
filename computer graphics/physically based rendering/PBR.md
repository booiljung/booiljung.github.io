# 물리 기반 렌더링(Physically Based Rendering, PBR)



물리 기반 렌더링(PBR, Physically Based Rendering)은 재질 표면에 따른 빛의 반사 정도가 물리적 원리에 기반하여 어떻게 이루어지는지를 시뮬레이션하고, 이를 시각적으로 표현하는 기술이다. 이 기법은 표면이 가진 고유한 특성을 렌더링 과정에 포함하는 방식으로, 물리 기반 셰이딩(PBS, Physically Based Shading)이라고도 불린다.1

기존의 렌더링 방식은 아티스트가 경험과 감각에 의존하여 재질의 색상, 그림자, 반사광을 수동으로 그려 넣는 작업이었다.3 예를 들어, `디퓨즈` 맵에 `앰비언트 오클루전`이나 `스페큘러` 정보를 함께 포함시키는 경우가 많았으며, 이로 인해 조명 환경이 바뀌면 재질의 모습을 일일이 재조정해야 하는 비효율성이 발생했다.1

그러나 PBR은 재질의 물리적 속성(예: 거칠기, 금속성)을 `파라미터`로 제공함으로써, 빛의 조건이 변화하더라도 일관되고 예측 가능한 결과물을 보장하는 혁신적인 접근법을 제시하였다.6 이는 아티스트가 기술적인 측면보다는 창의적인 아이디어 구현에 집중할 수 있는 지속 가능한 워크플로우를 가능하게 한다.6 PBR이 도입한 `직관적인 인터페이스`는 재질 제작 과정에서 불확실성을 크게 줄였고, 이는 현대 3D 그래픽스 파이프라인의 표준으로 자리 잡는 주요 원동력이 되었다.3

PBR의 본질에 대한 흔한 오해는 그 결과물이 무조건 사실적이어야 한다는 것이다.3 PBR의 시초는 사실 픽사/디즈니와 같은 비사실적 렌더링(NPR, Non-Photorealistic Rendering)을 위한 기술이었다.3 이들은 사실적인 기법을 활용해 만화적 표현을 더욱 풍부하게 만들고자 했으며, `몬스터 대학교`의 차량 페인트나 마이크 캐릭터의 피부 질감 표현 등이 그 사례다.9 따라서 PBR의 핵심은 포토리얼리즘 그 자체에 있다기보다, 재질의 물리적 특성을 `직관적인 파라미터`(예: 0과 1 사이의 값)로 표현하여 어떤 조명 환경에서도 일관적이고 예측 가능한 결과물을 만들도록 돕는 `방법론`에 있다.3 이러한 특성 덕분에 PBR은 현실적인 그래픽뿐만 아니라, 오버워치나 디스아너드 2와 같이 스타일리시한 비주얼을 추구하는 게임에서도 성공적으로 활용될 수 있다.9


물리적 기반으로 간주되는 PBR 셰이딩 모델은 다음 세 가지 조건을 반드시 충족해야 한다.10 이 원칙들은 PBR을 퐁(Phong)이나 블린-퐁(Blinn-Phong)과 같은 기존 경험적 모델과 구분 짓는 핵심적인 요소이다.

1. **마이크로패싯(Microfacet) 표면 모델에 기반할 것:** 모든 표면은 육안으로는 매끄러워 보일지라도, 미세한 관점에서는 무수히 작은 반사면(미세면)들로 구성되어 있다는 이론을 채택해야 한다.10
2. **에너지를 보존할 것:** 물체 표면에서 반사되거나 굴절되어 나가는 빛의 총량은 입사된 빛의 총량을 초과할 수 없다는 물리 법칙을 준수해야 한다.11
3. **물리 기반 BRDF를 사용할 것:** 빛의 입사각과 반사각에 따라 표면의 반사 특성을 정의하는 함수(BRDF)가 물리적 원리에 기반해야 한다.11

아래 표는 PBR이 기존 렌더링 모델과 어떻게 다른지 주요 특징을 정리한 것이다.

| **특징**          | **PBR(물리 기반 렌더링)**                  | **기존(레거시) 렌더링**                                |
| ----------------- | ------------------------------------------ | ------------------------------------------------------ |
| **기반 원리**     | 물리 법칙 기반 (미세면, 에너지 보존, BRDF) | 아티스트의 경험적 근사                                 |
| **재질 파라미터** | 러프니스, 메탈니스 등 직관적 파라미터 사용 | 스페큘러 강도, 신(Shininess) 등 비직관적 파라미터 사용 |
| **조명 반응**     | 어떤 조명 환경에서도 일관된 결과물 보장    | 조명 환경이 바뀌면 재조정이 필요함                     |
| **재질 정보**     | 순수한 재질 속성만 담은 맵 사용 (알베도)   | 광 반사, 음영 정보 등이 포함된 맵 사용 (디퓨즈)        |
| **대표 모델**     | 쿡-토랜스, GGX                             | 퐁(Phong), 블린-퐁(Blinn-Phong)                        |



모든 PBR 기술은 표면이 미시적인 수준에서 무수히 많은 작은 반사 거울, 즉 `미세면(Microfacets)`들로 이루어져 있다는 가설에 기반을 둔다.10 이 미세면들은 표면의 거칠기(Roughness)에 따라 그 정렬 상태가 달라진다.10

- **매끄러운 표면:** `러프니스` 값이 낮으면 미세면들이 거의 평평하게 정렬되어 입사된 빛이 예측 가능한 한 방향으로 강하게 반사된다. 이는 좁고 선명한 하이라이트를 생성한다.6
- **거친 표면:** `러프니스` 값이 높으면 미세면들이 불규칙하게 배열되어 입사된 빛이 여러 방향으로 흩어진다. 이로 인해 넓고 흐릿한 하이라이트가 나타난다.6

이 이론은 `러프니스`라는 단일 파라미터가 어떻게 표면의 광택과 반사광의 모양을 물리적으로 타당하게 결정하는지 설명하는 근간이 된다.10 PBR 셰이더는 이 미세면들의 정렬 상태를 통계적으로 근사하여 렌더링한다.10


빛이 물체 표면에 닿았을 때 반사되어 나가는 빛의 총 에너지는 입사된 빛의 총 에너지를 초과할 수 없다. 이것이 에너지 보존 법칙의 핵심이다.12 PBR은 이 원칙을 준수함으로써 현실과 같은 자연스러운 재질 표현을 가능하게 한다.11

입사된 빛 에너지는 크게 두 가지 경로로 나뉜다.

1. **반사광(Specular):** 표면에서 직접 반사되어 나가는 빛이다. 매끈한 표면에서 더 강하게 반사되며, 거친 표면에서는 더 희미하게 보인다.12
2. **굴절광(Diffuse):** 표면 안으로 흡수되었다가 재료 내부에서 산란되어 다시 표면 밖으로 나오는 빛이다.10

에너지 보존 법칙을 적용하면, 반사광의 강도가 강해질수록 굴절광의 강도는 감소해야 한다.10 PBR 모델은 이를 $k_D = 1.0 - k_S$와 같은 방식으로 구현하는데, 여기서 $k_D$는 굴절된 빛의 비율, $k_S$는 반사된 빛의 비율을 나타낸다.10


프레넬 효과는 빛이 물체 표면에 입사하는 각도에 따라 반사되는 양이 달라지는 현상을 의미한다. 빛이 표면에 수직으로 입사할 때 반사율이 가장 낮고, 옆에서 비스듬히 볼수록(입사각이 90도에 가까울수록) 반사율이 급격히 증가하여 거의 100% 반사하게 된다.15

이 효과는 모든 재질에 적용되지만, 그 정도는 재질의 종류에 따라 다르다.17 예를 들어, 플라스틱이나 나무와 같은 절연체(Dielectric) 재질은 정면에서 볼 때 2~5% 정도의 낮은 반사율을 갖는다.15 멀리서 물웅덩이를 보면 마치 거울처럼 반사되지만, 발밑을 내려다보면 물속 바닥이 보이는 것이 프레넬 효과의 좋은 예시이다.15

일부 소프트웨어에서는 이와 별개로, 광원이나 환경에 의존하지 않고 객체의 가장자리에 색상화된 빛을 내는 비물리적인 시각 효과를 "프레넬 효과"라고 부르기도 한다.18 이는 PBR의 근간을 이루는 물리적 프레넬 효과와는 다른 개념이므로, 용어를 혼동하지 않도록 유의해야 한다.15


BRDF(Bidirectional Reflectance Distribution Function)는 빛이 한 지점에서 특정 방향($w_i$)으로 입사했을 때, 다른 방향($w_o$)으로 얼마나 많이 반사되는지를 수학적으로 정의하는 함수이다.10 PBR은 이 BRDF를 기반으로 최종 렌더링 결과를 계산하며, 이는 `반사율 방정식(Reflectance Equation)`의 핵심 요소다.12

반사율 방정식은 다음과 같이 정의된다.
$$
L_o(p, \omega_o) = \int_{\Omega} f_r(p, \omega_i, \omega_o) L_i(p, \omega_i)(n \cdot \omega_i) d\omega_i
$$
여기서 $L_o$는 관찰자에게 보이는 빛의 양, $L_i$는 표면에 입사하는 빛의 양, $f_r$은 BRDF 함수, 그리고 $\int$는 표면의 반구($\Omega$) 내의 모든 방향에서 들어오는 빛을 합산하는 적분 기호다.10 PBR은 이 복잡한 적분 방정식을 실시간으로 계산하기 위해 `쿡-토랜스(Cook-Torrance) 모델`과 같은 근사 모델을 사용한다.11

쿡-토랜스 BRDF 모델은 확산광 항과 반사광 항으로 구성되며, 반사광 항은 다시 세 가지 주요 함수로 나뉜다.11
$$
f_r = k_d \frac{c_{diff}}{\pi} + k_s \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}
$$

- **정규 분포 함수 (D; Normal Distribution Function):** 미세면들의 정렬 상태를 통계적으로 나타낸다. 이 함수의 모양은 `러프니스` 값에 따라 변하며, `GGX(Generalized Trowbridge-Reitz)` 분포가 긴 꼬리(long tail)를 가진 하이라이트를 생성하여 거친 표면의 반사를 사실적으로 표현하는 데 널리 사용된다.16
- **기하 함수 (G; Geometry Function):** 거친 표면에서 미세면들이 서로를 가려 빛의 반사를 방해하는 `그림자(shadowing)` 및 `가리기(masking)` 효과를 모델링한다. `러프니스`가 클수록 이 효과는 강해진다.11
- **프레넬 항 (F; Fresnel Function):** 빛이 표면에 입사하는 각도에 따라 반사되는 빛의 양을 계산한다. `슐릭의 근사(Schlick's approximation)`를 사용하여 계산하는 것이 일반적이다.17



PBR은 `메탈릭/러프니스(Metallic/Roughness)`와 `스페큘러/글로시니스(Specular/Glossiness)`라는 두 가지 주요 워크플로우로 구현된다.8 이 둘은 물리적으로 타당한 결과를 내지만, 재질 파라미터를 정의하는 방식과 아티스트의 작업 방식에서 차이가 있다.

| **특징**           | **메탈릭/러프니스 워크플로우**                               | **스페큘러/글로시니스 워크플로우**                           |                                                              |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **주요 맵**        | Albedo(Base Color), Metallic, Roughness                      | Diffuse, Specular, Glossiness                                |                                                              |
| **특성**           | `Metallic` 맵을 사용해 재질이 금속인지 비금속인지 이분법적으로 정의. 비금속의 스페큘러는 4%로 고정. | `Specular` 맵을 사용해 비금속의 반사율(F0)을 직접 조절.      |                                                              |
| **장점**           | - 아티스트에게 매우 직관적임 (Metallic = 0 or 1)- 필요한 텍스처 채널이 적어 메모리 효율적임 (Albedo(RGB) + Metallic(G) + Roughness(G))- 실시간 렌더링에 적합하여 업계 표준으로 널리 채택됨 8 | - Specular 맵으로 비금속의 반사율을 세밀하게 제어 가능- 유색 비금속 등 독특한 재질 표현에 용이함- 오프라인 렌더링이나 VFX 분야에서 여전히 선호됨 25 |                                                              |
| **단점**           | - `Diffuse`와 `Specular` 색상이 다른 특수한 재질 표현에 제약이 있음 28 | - 금속과 비금속의 경계에 아티팩트가 발생할 수 있음 25        | - 파라미터가 덜 직관적임 (Glossiness = 1 - Roughness)- 필요한 텍스처 채널이 많아 메모리 비효율적임 (Diffuse(RGB) + Specular(RGB) + Glossiness(G))- 아티스트 실수로 에너지 보존 법칙을 위반할 위험이 있음 8 |
| **대표 사용 엔진** | 언리얼 엔진(Unreal Engine), 어도비 서브스탠스                | 유니티(Unity), 크라이엔진, 일부 오프라인 렌더러 25           |                                                              |

`메탈릭/러프니스` 워크플로우는 `직관성`과 `효율성`을 강점으로 실시간 렌더링을 지향하는 게임 산업의 표준으로 자리 잡았다.24 아티스트는 `메탈니스` 값을 0(비금속) 또는 1(금속)로 설정하고, `러프니스` 값만으로 표면의 질감을 조절할 수 있다.25 반면 `스페큘러/글로시니스` 워크플로우는 `스페큘러 맵`을 통해 비금속 재질의 반사율(F0)을 세밀하게 제어할 수 있는 `유연성`을 제공한다.25 이는 높은 제어력이 필요한 오프라인 렌더링이나 특정 시각 효과 작업에서 여전히 유용하게 활용된다.25

이처럼 PBR이라는 큰 틀 안에서도 다양한 구현 방식이 공존하는데, 이는 `효율성과 직관성` 대 `유연성과 통제력` 사이의 근본적인 트레이드오프를 보여준다.


PBR 워크플로우에서 재질의 물리적 속성은 여러 `텍스처 맵`을 통해 정의되며, 이 맵들은 렌더링 엔진에서 셰이더로 전달되어 빛의 상호작용을 계산하는 데 사용된다.

| **맵 이름**                        | **색상 형식**      | **주요 기능**                                                | **대표적 활용 사례**                               |
| ---------------------------------- | ------------------ | ------------------------------------------------------------ | -------------------------------------------------- |
| **알베도(Albedo)**                 | RGB                | 물체의 순수한 기본 색상 정의. 조명/그림자 정보 없음.         | 나무의 나뭇결, 사과의 붉은색 등 물체 고유의 색상 1 |
| **노멀(Normal)**                   | RGB                | 법선 벡터를 조작하여 표면의 미세한 깊이감을 표현. 실제 모델 변형 없음. | 돌벽의 울퉁불퉁한 표면, 가죽의 주름 등 1           |
| **범프(Bump)**                     | Grayscale          | 밝기 값으로 표면의 간단한 요철을 표현. 깊이감 한계, 가장 짧은 렌더링 시간. | 자잘한 돌기나 미세한 흠집 등 1                     |
| **디스플레이스먼트(Displacement)** | Grayscale          | 실제 3D 모델의 형태를 물리적으로 변형시킴. 가장 긴 렌더링 시간 소요. | 바위의 큰 굴곡, 나무껍질, 공룡 피부 등 거친 표면 1 |
| **러프니스(Roughness)**            | Grayscale          | 표면의 거칠기 정도를 정의. 흰색은 거침, 검은색은 매끄러움.   | 무광택 플라스틱(흰색), 유리(검은색) 등 1           |
| **메탈니스(Metalness)**            | Grayscale          | 재질이 금속인지 비금속인지 정의. 흰색(1)은 금속, 검은색(0)은 비금속. | 구리, 철(흰색)과 나무, 플라스틱(검은색) 구분 1     |
| **스페큘러(Specular)**             | RGB 또는 Grayscale | 비금속 재질의 반사광 색상/강도 조절. `Metallic` 맵과 함께 사용되지 않음. | 플라스틱의 색이 있는 하이라이트 등 1               |
| **앰비언트 오클루전(AO)**          | Grayscale          | 빛이 닿기 어려운 틈새와 구석을 어둡게 표현하여 깊이감 향상.  | 벽돌 사이의 홈, 오브젝트 접합부의 그림자 등 1      |
| **에미시브(Emissive)**             | RGB                | 물체 스스로 빛을 내는 효과를 표현.                           | 형광등, LED 스크린, 불꽃 등 1                      |
| **오파시티(Opacity)**              | Grayscale          | 물체의 투명도와 범위를 조절. 흰색은 불투명, 검은색은 투명.   | 식물의 잎, 유리, 얇은 천 등 1                      |
| **리프랙션(Refraction)**           | Grayscale          | 투명한 물체를 통과하는 빛의 굴절 정도를 조절.                | 물속의 물체, 유리 뒤의 사물 등 1                   |



PBR은 최신 `AAA급 게임`의 사실적인 그래픽을 구현하는 데 필수적인 기술로 자리 잡았다.5 재질의 물리적 특성을 정확하게 묘사함으로써, 다양한 조명 환경에서도 일관적이고 현실적인 모습을 유지할 수 있다.29 한편, PBR은 `오버워치`나 `디스아너드 2`와 같이 `스타일리시한` 비주얼을 추구하는 게임에서도 활용되어, 만화 같은 외형을 유지하면서도 재질의 물리적 표현을 강화하여 시각적 완성도를 높이는 데 기여했다.9


PBR의 시초라 할 수 있는 `디즈니의 원칙 기반 BRDF(principled BRDF)`는 비실사 애니메이션에서 사실적인 질감을 표현하기 위해 개발되었다.3 이 BRDF는 아티스트에게 직관적이고 적은 수의 파라미터를 제공하는 것을 핵심 원칙으로 삼아, `몬스터 대학교`와 같은 애니메이션 영화의 재질 표현에 활용되었다.9 최근에는 

`언리얼 엔진`과 같은 게임 엔진이 `실시간 글로벌 조명`(`Global Illumination`)과 `노드 기반 재료 편집기`를 PBR과 결합하여 영화 제작의 `사전 시각화(pre-visualization)` 단계를 혁신하고 있다.30 감독과 제작진은 가상 환경에서 실시간으로 조명과 카메라를 조정하며 `가상 위치 스카우팅`, `스토리보드` 제작 등의 작업을 수행함으로써 시간과 비용을 획기적으로 절약할 수 있게 되었다.30


D5 Render`와 같은 실시간 렌더링 솔루션은 `PBR 재질`과 `정확한 조명`을 결합하여 건축 시각화 분야에서 전문가 수준의 결과물을 빠르게 만들어낸다.31 건축 시각화 전문가는 PBR 텍스처(예: `Rustic Stone Wall` 텍스처)를 활용해 실제와 흡사한 재질감을 표현하며, 실시간 렌더링을 통해 디자인을 즉각적으로 검토할 수 있다.32 이를 통해 고객 프레젠테이션 및 대회 출품작 제작 시간을 획기적으로 단축할 수 있다.31



PBR은 `사실적인` 표현에 매우 탁월하지만, 모든 상황에서 완벽한 만능 기술은 아니다.3 PBR의 엄격한 물리적 제약은 때때로 아티스트가 의도적으로 `과장되거나`, `비현실적인` 표현을 하는 것을 어렵게 만들 수 있다.3 또한, 높은 품질의 PBR 텍스처를 제작하는 데에는 상당한 `시간`과 `노력`이 소모되며, 고품질 렌더링은 여전히 많은 `컴퓨팅 파워`를 요구하는 단점이 있다.5


PBR과 `레이 트레이싱(Ray Tracing)`은 서로 완전히 별개의 기술이지만, 최신 3D 그래픽스 렌더링에서는 이 둘을 **동시에** 사용하는 것이 일반적이다.2 PBR은 재질의 물리적 특성을 정의하는 `셰이딩 모델`이며, 레이 트레이싱은 광선의 반사와 굴절을 추적하여 물리적으로 정확한 `빛의 전송(light transport)`을 계산하는 `렌더링 기술`이다.2 PBR 재질을 기반으로 레이 트레이싱을 적용하면, 빛의 물리적 상호작용이 더욱 정확하게 시뮬레이션되어 PBR의 사실적인 효과를 극대화할 수 있다.7 GPU 기술의 발전으로 `실시간 레이 트레이싱`이 가능해지면서, 오프라인 렌더링과 실시간 렌더링의 경계가 모호해지고 있으며, PBR은 이러한 변화의 중심에 있다.7


최근 `AI` 기술은 PBR 워크플로우를 혁신적으로 간소화하는 방향으로 발전하고 있다.29 예를 들어, D5 Render와 같은 솔루션은 AI 기반 최적화 및 적응형 조명을 통해 복잡한 장면의 라이팅 작업을 단순화한다.29

가장 주목할 만한 발전은 **AI 텍스처 생성**이다.29 AI는 하나의 `베이스 컬러 맵`만으로 `노멀`, `러프니스`, `높이` 맵 등 필요한 모든 PBR 텍스처 맵을 자동으로 생성할 수 있다.29 이는 고품질 텍스처 제작에 소요되는 시간을 획기적으로 단축하여 아티스트의 작업 효율성을 크게 향상한다.29

이러한 현상은 PBR이 제공하는 `규칙`과 `표준`이 AI가 학습하고 예측하기에 이상적인 환경을 조성한다는 점을 보여준다. PBR이 재질의 물리적 특성을 `파라미터`로 명확하게 정의해 놓았기 때문에, AI는 이러한 표준화된 데이터 구조를 기반으로 새로운 텍스처 맵을 생성하고 렌더링을 최적화할 수 있다. 이는 PBR이 단순한 렌더링 기술의 발전을 넘어, AI 기술이 3D 그래픽스 제작에 통합될 수 있는 견고한 토대 역할을 하고 있음을 의미한다.


1. [Texture] PBR Texture / PBR텍스쳐란 무엇인가?, 8월 14, 2025에 액세스, https://caresser.tistory.com/36
2. 물리 기반 렌더링 - 나무위키, 8월 14, 2025에 액세스, [https://namu.wiki/w/%EB%AC%BC%EB%A6%AC%20%EA%B8%B0%EB%B0%98%20%EB%A0%8C%EB%8D%94%EB%A7%81](https://namu.wiki/w/물리 기반 렌더링)
3. PBR에 대한 흔한 오해 - 사실적인 렌더링 - Hybrid3D, 8월 14, 2025에 액세스, https://blog.hybrid3d.dev/2018-04-12-misunderstandings-in-pbr
4. [Graphics] 3D 모델링을 위한 OBJ & MTL 파일 구조와 PBR 재질 정리, 8월 14, 2025에 액세스, https://mvje.tistory.com/83
5. PBR vs Non-PBR: 재질 표현의 대결! 🖌️ - 재능넷, 8월 14, 2025에 액세스, https://www.jaenung.net/tree/5214
6. PBR(물리 기반 렌더링)이란? - Adobe Substance 3D, 8월 14, 2025에 액세스, https://www.adobe.com/kr/products/substance3d/discover/pbr.html
7. 물리 기반 렌더링(PBR): 디지털 재질의 사실성 - Render Farm, 8월 14, 2025에 액세스, https://garagefarm.net/ko-blog/physically-based-rendering-pbr-realism-in-digital-materials
8. Adobe Learn - Learn Substance 3D Designer The PBR Guide - Part 2, 8월 14, 2025에 액세스, https://www.adobe.com/learn/substance-3d-designer/web/the-pbr-guide-part-2
9. 현실적이지 않은 PBR : r/gamedev - Reddit, 8월 14, 2025에 액세스, https://www.reddit.com/r/gamedev/comments/64ons2/non_realistic_pbr/?tl=ko
10. Learn OpenGL - PBR : Theory - Gyutae Lee - 티스토리, 8월 14, 2025에 액세스, https://gyutts.tistory.com/187
11. PBR(물리 기반의 렌더링) 기본 정리 - OptIn의 삽질노트, 8월 14, 2025에 액세스, https://bbtarzan12.github.io/PBR/
12. Theory - LearnOpenGL, 8월 14, 2025에 액세스, https://learnopengl.com/PBR/Theory
13. OpenGL 정리 - 29. 물리 기반 렌더링 (Physically Based Rendering ..., 8월 14, 2025에 액세스, https://surkim.tistory.com/m/65
14. PBR(Physically Based Rendering) - BRDF - WinCNT - 티스토리, 8월 14, 2025에 액세스, https://wincnt-shim.tistory.com/304
15. 서브스턴스 PBR 가이드 파트 1_7 프레넬 효과(Fresnel Effect) - YouTube, 8월 14, 2025에 액세스, https://www.youtube.com/watch?v=_XKKlRJQ5wE
16. [번역] [물리적 렌더링 (PBR) 백서] (3) 디즈니 원칙의 BRDF 및 BSDF 요약 - techartnomad, 8월 14, 2025에 액세스, https://techartnomad.tistory.com/35
17. Cook-Torrance Shading | portfolio - Wix.com, 8월 14, 2025에 액세스, https://garykeen27.wixsite.com/portfolio/cook-torrance-shading
18. Fresnel 효과 - Azure Remote Rendering - Microsoft Learn, 8월 14, 2025에 액세스, https://learn.microsoft.com/ko-kr/azure/remote-rendering/overview/features/fresnel-effect
19. PBR (Physically Based Rendering), 8월 14, 2025에 액세스, https://shubidumdu.github.io/posts/pbr
20. Article - Physically Based Rendering - Cook–Torrance - Coding Labs, 8월 14, 2025에 액세스, http://www.codinglabs.net/article_physically_based_rendering_cook_torrance.aspx
21. 6. BRDF 구현하기(Implementing BRDF) - Technical Art / 계속 공부하는 블로그 - 티스토리, 8월 14, 2025에 액세스, https://prooveyourself.tistory.com/6
22. Confused with all the different names of the microfacet model's distribution and masking-shadowing functions : r/GraphicsProgramming - Reddit, 8월 14, 2025에 액세스, https://www.reddit.com/r/GraphicsProgramming/comments/1btcfz7/confused_with_all_the_different_names_of_the/
23. Physically Based Rendering - More Accurate Microsurface Distribution Function GGX | by UWA | Medium, 8월 14, 2025에 액세스, https://medium.com/@uwa4d/physically-based-rendering-more-accurate-microsurface-distribution-function-ggx-3968fc09fa48
24. An Intro to Physically Based Rendering Material Workflows and Metallic/Roughness, 8월 14, 2025에 액세스, https://blog.turbosquid.com/2023/07/27/an-intro-to-physically-based-rendering-material-workflows-and-metallic-roughness/
25. Which is better, more used and why, Metal or Specular workflow? : r/vfx - Reddit, 8월 14, 2025에 액세스, https://www.reddit.com/r/vfx/comments/118id8d/which_is_better_more_used_and_why_metal_or/
26. Why did U4 use roughness/metallic vs Specular/Glossiness? - Unreal Engine Forums, 8월 14, 2025에 액세스, https://forums.unrealengine.com/t/why-did-u4-use-roughness-metallic-vs-specular-glossiness/9804
27. Metallic Roughnes VS Specular Glossiness / Shading Basics - YouTube, 8월 14, 2025에 액세스, https://www.youtube.com/watch?v=IRaxcECNYkM
28. PBR 기초 이론과 사용되는 맵들 Vol.3 - SlideShare, 8월 14, 2025에 액세스, https://www.slideshare.net/slideshow/pbr-vol3/250566173
29. PBR이란 무엇인가: 사실적 렌더링을 위한 가이드 - D5 Render, 8월 14, 2025에 액세스, https://www.d5render.com/ko/posts/what-is-pbr-in-computer-graphics
30. 언리얼 엔진과 같은 게임 엔진이 VFX 제작에 미치는 영향 - Synapse Studio, 8월 14, 2025에 액세스, [https://synapsestudiovn.com/ko/%EC%96%B8%EB%A6%AC%EC%96%BC-%EC%97%94%EC%A7%84%EA%B3%BC-%EA%B0%99%EC%9D%80-%EA%B2%8C%EC%9E%84-%EC%97%94%EC%A7%84%EC%9D%B4-vfx-%EC%A0%9C%EC%9E%91%EC%97%90-%EB%AF%B8%EC%B9%98%EB%8A%94-%EC%98%81%ED%96%A5/](https://synapsestudiovn.com/ko/언리얼-엔진과-같은-게임-엔진이-vfx-제작에-미치는-영향/)
31. 6가지 건축 스타일 설명: 최신 렌더링 팁 포함 - D5 Render, 8월 14, 2025에 액세스, https://www.d5render.com/ko/posts/6-architectural-styles-modern-rendering-virtual-tour-tips
32. Rustic Stone Wall PBR - SUPER SHADERS - 티스토리, 8월 14, 2025에 액세스, https://aesource.tistory.com/entry/RusticStoneWall-sbsar
33. 건축가를 위한 최고의 레이 트레이싱 소프트웨어: 2025 구매자 가이드 - D5 Render, 8월 14, 2025에 액세스, https://www.d5render.com/ko/posts/the-best-ray-traced-3d-software




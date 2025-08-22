---
layout: page
title: 2D Gaussian Splatting (2DGS) 기반 TDOM 생성에 대한 고찰
permalink: /computer graphics/gaussian splatting/2DGS 기반 TDOM 생성
---


최근 3D 재구성 및 신규 시점 합성 분야에서 Gaussian Splatting (GS)은 고품질 및 빠른 렌더링 속도를 제공하는 유망한 해결책으로 부상하였다.1 특히, 2D Gaussian Splatting (2DGS)은 다중 시점 이미지로부터 기하학적으로 정확한 복사장(radiance fields)을 모델링하고 재구성하는 새로운 접근 방식이다.2 2DGS의 핵심 아이디어는 3D 볼륨을 2D 방향성 평면 가우시안 디스크(oriented planar Gaussian disks) 집합으로 축소하여 씬을 표현하는 것이다.2

한편, True Digital Orthophoto Map (TDOM)은 도시 계획, 인프라 관리, 환경 모니터링 등 다양한 응용 분야에서 높은 기하학적 정밀도와 조밀한 이미지 특징으로 인해 수요가 매우 높다.5 그러나 전통적인 TDOM 생성 방법은 Digital Surface Models (DSM) 및 오클루전 감지(occlusion detection)와 같은 복잡하고 계산 비용이 많이 드는 프로세스를 필요로 하며, 이 과정에서 오류가 발생할 가능성이 존재한다.6 본 보고서는 2DGS가 이러한 전통적인 TDOM 생성 방법의 한계를 극복하고 고품질의 공간 재구성 및 정사영상 생성에 어떻게 기여하는지 심층적으로 고찰할 것이다.

본 보고서는 2DGS 기반 TDOM 생성 기술의 개념, 작동 원리, 장점, 한계 및 최신 연구 동향을 포괄적으로 분석하는 것을 목적으로 한다. 2DGS가 TDOM 생성 분야에 가져오는 혁신과 그 잠재력을 탐구하고, 관련 기술적 과제를 명확히 제시할 것이다.

2DGS의 본질적인 강점과 TDOM의 요구사항 사이에는 강력한 시너지가 존재한다. 2DGS는 3D 볼륨을 2D 방향성 평면 가우시안 디스크 집합으로 축소하여 시점 일관된 기하학을 제공하며 표면을 내재적으로 모델링한다.2 이러한 특성은 잡음 없고 상세한 기하학적 재구성을 가능하게 한다.2 TDOM은 높은 기하학적 정밀도와 조밀한 이미지 특징을 요구하며, 정확한 공간 데이터를 제공해야 한다.5 2DGS의 시점 일관된 기하학 및 표면 내재적 모델링 능력은 TDOM이 요구하는 높은 기하학적 정밀도 및 조밀한 이미지 특징과 직접적으로 부합한다. 특히, 2DGS가 명시적인 DSM 및 오클루전 감지 없이 TDOM을 생성할 수 있다는 점은 전통적인 방법의 높은 계산 비용과 오류 발생 가능성이라는 한계를 직접적으로 해결함을 의미한다.6 이는 단순한 기술 적용을 넘어, 2DGS가 TDOM 생성 패러다임을 근본적으로 변화시킬 잠재력을 가짐을 시사한다. 2DGS는 표면 재구성에 특화되어 있어, 3DGS가 겪는 다중 시점 불일치 문제 3를 해결함으로써 TDOM의 핵심 요구사항인 기하학적 정확도를 더욱 효과적으로 충족시킨다. 결과적으로 2DGS는 TDOM의 정확성과 효율성이라는 두 가지 핵심 가치를 동시에 제공하는 시너지를 창출한다.



2DGS는 다중 시점 이미지로부터 기하학적으로 정확한 복사장을 모델링하고 재구성하는 새로운 접근법이다.2 이 방법의 핵심 아이디어는 3D 볼륨을 2D 방향성 평면 가우시안 디스크(oriented planar Gaussian disks) 집합으로 축소하는 것이다. 3D 가우시안과 달리, 2D 가우시안은 표면을 내재적으로 모델링하면서 시점 일관된 기하학을 제공한다.2

얇은 표면을 정확하게 복구하고 안정적인 최적화를 달성하기 위해, 2DGS는 광선-스플랫 교차(ray-splat intersection) 및 래스터화(rasterization)를 활용하는 원근 정확한 2D 스플랫팅 프로세스를 도입한다.2 이러한 미분 가능한 렌더러는 잡음 없고 상세한 기하학적 재구성을 가능하게 하며, 경쟁력 있는 외관 품질, 빠른 훈련 속도 및 실시간 렌더링을 유지한다.2 2DGS는 3DGS의 다중 시점 불일치 문제를 해결하고 기하학적 표현 및 렌더링 메커니즘을 향상시켜 표면 재구성 품질과 원근 일관성을 높인다.6 또한, 2D 가우시안 프리미티브의 내재적 표면 법선(surface normals)은 법선 제약(normal constraints)을 통한 직접적인 표면 정규화를 가능하게 한다.2


2DGS의 장점을 명확히 이해하기 위해서는 기존의 3D 장면 표현 및 렌더링 기술인 3DGS(3D Gaussian Splatting) 및 NeRF(Neural Radiance Fields)와의 비교가 필수적이다.


3DGS는 고품질 신규 시점 합성 및 빠른 렌더링 속도로 3D 재구성 분야를 혁신하였으나 3, 3D 가우시안의 다중 시점 불일치 특성으로 인해 표면을 정확하게 표현하지 못하는 한계가 존재한다.3 반면 2DGS는 3DGS의 이러한 한계를 극복하고 2D 서펠(surfels)을 사용하여 얇은 표면을 근사함으로써 우수한 기하학적 재구성 품질을 보여준다.8 2D 디스크는 복잡하고 얇은 표면에 잘 정렬될 수 있으며, 3D 가우시안 프리미티브의 다중 시점 투영으로 인한 불일치 문제를 완전히 해결하여 재구성의 기하학적 정확도를 크게 향상시킨다.8 그러나 2DGS는 광택 있는 표면(glossy surfaces)을 처리할 때 구멍(holes)이 발생하거나 깊이 편향(depth bias) 문제가 발생할 수 있으며 8, 3DGS에 비해 훈련 병렬화가 적어 훈련 시간이 길어질 수 있다는 의견도 존재한다.9 하지만 최적화된 2DGS 구현은 3DGS보다 빠른 훈련 시간을 달성할 수 있음을 보여주는 연구 결과도 있다.8


NeRF 기반 방법은 일반적으로 컴팩트한 신경망을 사용하여 암시적 연속 매핑을 생성하며, 복잡한 이미지 세부 정보를 보존하고 이미지 압축 및 초해상도와 같은 응용 분야에 새로운 가능성을 제시한다.1 그러나 NeRF는 긴 훈련 시간과 느린 렌더링 속도 문제로 인해 신규 시점 합성의 실제 적용 가능성을 크게 제한하는 단점이 있다.7 2DGS는 INR(Implicit Neural Representations) 기반 방법보다 작은 이미지 피팅에서 유사한 품질과 더 높은 효율성을 달성하며, 만족스러운 신호 대 잡음비(SNR)를 유지한다.1


일반적인 2D 가우시안 함수는 평균 $\mu$ (또는 중심)와 공분산 행렬 $\Sigma$로 정의된다.11 2D에서 공분산 행렬 $\Sigma$는 2x2 행렬이며, 가우시안의 타원형 모양을 정의하는 기하학적 의미를 갖는다.11 2DGS의 핵심은 3D 공간의 2D 방향성 평면 가우시안 디스크를 사용하여 장면을 표현하는 것이다.2

렌더링 과정에서 3D 가우시안은 화면에 2D 가우시안으로 투영된다. 이 과정에서 3x3 공분산 행렬 $\Sigma$는 회전 및 스케일링 변환을 통해 표준 가우시안을 타원형으로 변형시키는 역할을 한다. 추가적인 변환 $V$가 적용될 경우, 공분산 행렬은 $\Sigma' = V \Sigma V^\top$ 로 업데이트된다. 이는 3D 공분산을 2D 공간으로 투영하는 데 중요하다.11 2DGS는 광선-스플랫 교차 및 래스터화(rasterization)를 활용하는 원근 정확한 2D 스플랫팅 프로세스를 도입한다.2

렌더링 방정식 (Rendering Equation)은 각 픽셀의 색상 $C$를 계산하기 위해 가우시안 프리미티브의 색상 $c_i$와 불투명도 $\alpha_i$를 누적하는 방식으로 표현될 수 있다 12:
$$
C = \sum_{i \in \text{sorted Gaussians}} c_i \alpha_i \prod_{j=1}^{i-1} (1 - \alpha_j)
$$
여기서 $c_i$는 $i$번째 가우시안의 색상, $\alpha_i$는 $i$번째 가우시안의 불투명도, 그리고 $\prod_{j=1}^{i-1} (1 - \alpha_j)$는 이전 가우시안들의 누적 투과율(accumulated transmittance)을 나타낸다. 2DGS는 최적화 과정에서 깊이 왜곡(depth distortion) 및 법선 일관성(normal consistency) 항을 도입하여 재구성 품질을 더욱 향상시킨다.2 깊이 왜곡 항은 광선을 따라 2D 프리미티브가 좁은 범위 내에 집중되도록 하여 가우시안 간의 거리가 무시되는 렌더링 프로세스의 한계를 해결한다.2

2DGS의 기하학적 정확도 향상 메커니즘은 매우 중요하다. 3DGS는 다중 시점 불일치로 인해 표면을 정확히 표현하지 못하는 근본적인 한계를 가진다.3 이는 3D 가우시안이 볼륨을 표현하여 표면의 얇은 특성을 제대로 반영하지 못하고, 다른 시점에서 볼 때 기하학적 형태가 달라지는 불일치를 야기하기 때문이다. 반면 2DGS는 3D 볼륨을 2D 평면 디스크로 붕괴시킴으로써 2, 표면 자체에 더 가깝게 모델링한다. 이러한 접근 방식은 TDOM과 같이 높은 기하학적 정밀도와 정확한 깊이 정보 5가 필수적인 응용 분야에서 2DGS가 3DGS보다 훨씬 유리한 이유를 설명한다. 특히, 명시적인 광선-스플랫 교차를 통한 원근 정확한 스플랫팅 2은 렌더링 과정에서 발생하는 기하학적 오류를 최소화하여 최종 결과물의 정확도를 높이는 핵심 요소이다. 2DGS의 기하학적 정확도 향상은 단순히 더 나은 시각적 품질을 제공하는 것을 넘어, TDOM과 같은 정량적이고 측정 가능한 공간 데이터를 요구하는 분야에서 2DGS가 신뢰할 수 있는 기반 기술이 될 수 있음을 의미한다. 이는 3D 재구성 기술이 단순한 시각화 도구를 넘어 실제 산업 및 공학적 응용에서 활용될 수 있는 문을 연다. 즉, 2DGS는 3DGS가 제공하지 못했던 기하학적 신뢰성이라는 새로운 차원의 가치를 제공한다.

2DGS는 빠른 훈련 속도와 실시간 렌더링을 유지한다고 언급되지만 2, 3DGS에 비해 훈련 병렬화가 적어 훈련 시간이 길어질 수 있다는 의견도 존재한다.9 이는 수많은 가우시안 포인트 상황에서 대규모 이미지 피팅에 대한 도전 과제로 이어진다.1 D2GV(Deformable 2D Gaussian Splatting for Video Representation)와 같은 비디오 표현 방식에서는 고정 길이 GoP(Group of Pictures)가 빠른 장면 전환에 비효율적일 수 있다는 점이 지적된다.13 이러한 점들은 2DGS의 빠른 훈련 속도가 상대적인 개념일 수 있으며, 특정 대규모 데이터셋에서는 훈련 효율성이 저하될 수 있음을 시사한다. 이는 고해상도 TDOM 생성 시 중요한 병목 지점이 될 수 있다. D2GV의 고정 GoP 크기 한계는 동적 또는 대규모 환경에서 2DGS 기반 시스템의 유연성과 최적 성능을 저해할 수 있다. 즉, 2DGS의 기본 개념은 효율적이지만, 실제 대규모 응용에서는 여전히 최적화 및 확장성 문제가 존재하며, 이를 해결하기 위한 추가적인 방법론(예: LIG의 Level-of-Gaussian, D2GV의 GoP 전략)이 필요하다.1 2DGS가 기하학적 정확도에서 우위를 점하지만, 규모와 동적 변화를 다루는 데 있어서는 여전히 최적화될 여지가 있다는 점이 드러난다. 이는 2DGS가 단순히 3DGS의 대체재가 아니라, 특정 응용 분야의 요구사항에 맞춰 지속적인 개선과 변형이 필요한 진화하는 기술임을 의미한다. 특히, TDOM과 같은 대규모 지리 공간 데이터 처리에서는 이러한 훈련 및 렌더링 효율성, 그리고 동적 환경 처리 능력이 핵심적인 성공 요인이 될 것이다.

아래 표는 2DGS와 3DGS 및 NeRF의 주요 특징을 비교한다.

**표 1: 2DGS와 3DGS 및 NeRF 비교**

| 특징                | 2D Gaussian Splatting (2DGS)                           | 3D Gaussian Splatting (3DGS)                | Neural Radiance Fields (NeRF)                           |
| ------------------- | ------------------------------------------------------ | ------------------------------------------- | ------------------------------------------------------- |
| **표현 방식**       | 2D 방향성 평면 가우시안 디스크 집합 2                  | 3D 가우시안 집합 3                          | 암시적 신경망 기반 연속 매핑 1                          |
| **기하학적 정확도** | 우수 (시점 일관성, 표면 내재적 모델링) 2               | 한계 (다중 시점 불일치, 표면 표현 부정확) 3 | 우수 (복잡한 이미지 세부 정보 보존) 1                   |
| **렌더링 속도**     | 빠름 (실시간 렌더링) 2                                 | 매우 빠름 (실시간 렌더링) 3                 | 느림 (실제 적용 제한) 7                                 |
| **훈련 속도**       | 빠름 (경쟁력 있음) 2                                   | 매우 빠름 3                                 | 매우 느림 (수 시간 소요) 10                             |
| **주요 장점**       | 기하학적 충실도, 얇은 표면 재구성, 깊이 맵 직접 도출 2 | 고품질 신규 시점 합성, 빠른 렌더링 1        | 고품질 이미지 표현, 압축, 초해상도 가능성 1             |
| **주요 단점**       | 광택 표면 구멍/깊이 편향, 대규모 데이터 최적화 과제 1  | 표면 표현 부정확, 메싱 어려움 3             | 높은 훈련/추론 비용, 제한된 확장성, 낮은 해석 가능성 10 |
| **주요 적용 분야**  | TDOM 생성, 표면 재구성 5                               | 3D 재구성, 신규 시점 합성, VR 1             | 이미지/비디오 압축, 초해상도 1                          |

이 표는 복잡한 정보를 구조화하고 시각적으로 명확하게 제시하여 각 기술의 상대적 위치와 장단점을 한눈에 파악할 수 있도록 한다. 이는 독자의 학습 효율성을 극대화하고, 보고서의 전문성을 높이는 데 기여한다. 이러한 비교는 특정 응용 분야, 예를 들어 높은 기하학적 정밀도를 중요시하는 TDOM 생성에 어떤 기술이 가장 적합한지 판단하는 데 필요한 핵심 정보를 제공한다.



TDOM은 높은 기하학적 정밀도와 조밀한 이미지 특징을 특징으로 하는 지도이다.5 TDOM은 도시 계획, 인프라 관리, 환경 모니터링, 재난 평가 등 다양한 분야에서 정확한 공간 데이터에 대한 수요가 증가함에 따라 그 중요성이 부각되고 있다.5 일반적인 디지털 정사영상 지도(DOM)는 지형 및 비스듬한 촬영으로 인한 투영 왜곡을 보정하기 위해 Digital Elevation Model (DEM)과 이미지를 결합하여 생성된다.7 그러나 TDOM은 건물 가장자리와 같은 복잡한 지형 및 얇은 구조에서 발생하는 왜곡을 더욱 정밀하게 보정하여 실제 3D 지형을 정확히 반영하는 데 중점을 둔다.7


전통적인 TDOM 생성 방법은 Digital Surface Models (DSM) 및 오클루전 감지(occlusion detection)와 같은 정교하고 계산 비용이 많이 드는 프로세스를 필요로 하며 오류 발생 가능성이 존재한다.6 Z-버퍼(Z-buffer) 방식과 같은 초기 가시성 분석 알고리즘은 원리는 단순하지만 고정밀 DSM에 강하게 의존하는 특성을 가진다.6 최근에는 학습 기반(learning-based) TDOM 생성 방법도 등장하였으나, 이들 역시 제한된 일반화 능력과 LiDAR 데이터에 대한 높은 의존성이라는 단점을 가진다.7

이러한 기존 방법들은 고품질 DSM 또는 DBM(Digital Building Model)에 크게 의존하며, 이는 획득 비용이 높다. 또한, 정밀한 가장자리 감지(edge detection), 자연스러운 텍스처 보정(texture compensation), 그리고 효율적인 워크플로우 측면에서 여전히 어려움을 겪는다.7 특히, 대규모 TDOM을 생성할 때 이러한 어려움은 더욱 두드러진다.7

TDOM의 진정한 요구사항과 기존 기술의 근본적인 한계를 고찰할 필요가 있다. TDOM은 높은 기하학적 정밀도와 조밀한 이미지 특징을 요구하며, 특히 건물 가장자리가 연속적이고 선명해야 하고, 입면이 보여서는 안 된다.7 또한 전력탑이나 난간과 같은 얇은 구조물의 복원도 매우 중요하다.7 그러나 기존 방법은 DSM 및 오클루전 감지에 의존하며, 이로 인해 계산 비용이 높고 오류 발생 가능성이 존재한다.6 또한 정밀한 가장자리 감지와 자연스러운 텍스처 보정에 어려움을 겪는다.7 TDOM의 핵심은 3D 공간의 실제 기하학을 2D 평면에 왜곡 없이 투영하는 것이다. 이는 특히 수직 구조물(건물)의 입면이 보이지 않아야 하고, 얇은 구조물이 정확하게 표현되어야 한다는 점에서 기존의 DSM 기반 방식이 가지는 근본적인 한계를 드러낸다. DSM은 지표면의 높이 정보를 제공하지만, 건물과 같은 객체의 수직면이나 복잡한 얇은 구조를 정확히 모델링하는 데는 한계가 있다. 오클루전 감지 또한 복잡한 계산을 필요로 하며, 부정확할 경우 TDOM의 품질을 저하시킨다. 즉, 기존 방법은 TDOM의 진정한 3D 기하학적 충실도 요구사항을 충족시키기에는 본질적인 제약이 있다. TDOM의 진정한 가치는 단순한 2D 지도 생성을 넘어, 실제 3D 환경의 디지털 트윈으로서의 역할을 수행하는 데 있다. 기존 방법의 한계는 이러한 진정한 디지털 트윈 구축을 어렵게 만들었다.



2DGS는 기존 TDOM 생성 방법의 한계를 극복하는 대안적인 기술로 제시된다. Wang et al.의 연구 5는 명시적인 DSM 및 오클루전 감지 없이 TDOM을 생성하는 새로운 방법을 제안한다.

이 연구의 주요 기여는 다음과 같다:

- 기하학적 충실도 및 다중 시점 일관성을 보장하는 효율적인 미분 가능 2D 가우시안 렌더러 기반 TDOM 생성 프레임워크를 제안한다.6
- 렌더링 과정에서 직접 깊이 맵을 도출하는 기하학 인식 깊이 추정 모듈을 도입한다.6
- 제한된 컴퓨팅 자원 하에서 효율적인 대규모 장면 재구성 및 고해상도 TDOM 렌더링을 위한 분할 정복(divide-and-conquer) 전략을 활용한다.5

세부 프로세스를 살펴보면, 먼저 점진적 데이터 분할을 수행한 후 병렬 훈련을 진행한다.7 훈련된 가우시안은 정사 투영(orthographic projection) 방식으로 이미지 평면에 투영되며 7, 최종적으로 배치 래스터화(batch rasterization)를 통해 완전한 TDOM이 렌더링된다.7

2DGS를 TDOM 생성에 적용함으로써 얻을 수 있는 이점은 명확하다. 2DGS는 표면 재구성 품질 및 원근 일관성을 향상시키며 6, 법선 정보를 활용하여 표면 구조를 정교화함으로써 고품질 3D 재구성을 가능하게 한다.6 또한, 렌더링 과정에서 깊이 맵을 직접 추출하여 3D 구조를 정확하게 표현할 수 있다.6


2DGS 기반 TDOM 생성 방법은 여러 면에서 우수한 성능과 장점을 보여준다.


이 방법은 건물 경계 및 오클루전 관계를 우수하게 재구성한다.7 기존 방법에서 흔히 발생하던 건물 입면 노출, 흐릿한 가장자리, 불규칙한 경계, 정렬 불량 등의 문제가 해결됨을 확인하였다.7 특히, 전력탑 지지대나 에어컨 난간과 같은 얇은 구조물의 세부 사항이 강한 명암 대비 조건에서도 선명하게 복원되는 것을 관찰할 수 있다.7


2DGS 기반 접근 방식은 대규모 장면 재구성 및 고정밀 지형 모델링에서 높은 효율성을 입증한다.5 제한된 컴퓨팅 자원으로도 고해상도 TDOM 렌더링이 가능하며 5, 전통적인 DSM 또는 오클루전 감지 프로세스가 필요 없어 계산 비용을 크게 절감할 수 있다.6


이 방법은 정확한 공간 데이터를 제공하여 도시 계획 및 의사 결정 지원과 같은 다양한 응용 분야에 기여한다.5 또한, 기존 학습 기반 방법의 단점이었던 LiDAR 데이터에 대한 높은 의존성을 낮추는 이점도 있다.7

2DGS 기반 TDOM 생성은 정교함과 실용성을 동시에 확보하는 중요한 전환점이다. 기존 TDOM 방법의 주요 병목은 DSM 생성의 복잡성과 오클루전 감지의 부정확성, 그리고 이로 인한 높은 계산 비용이었다. 2DGS는 이러한 중간 단계를 제거하고, 렌더링 과정에서 직접 깊이 정보를 얻어냄으로써 워크플로우를 간소화한다. 이는 높은 기하학적 정확도와 얇은 구조 복원이라는 정교함을 유지하면서도, 계산 효율성과 자원 절감이라는 실용성을 동시에 확보하는 핵심적인 전환점이다. 특히, 분할 정복 전략은 대규모 지리 공간 데이터 처리의 고질적인 문제인 자원 제약을 해결하여, 이론적 가능성을 실제 적용 가능한 수준으로 끌어올린다. 2DGS 기반 TDOM 생성은 단순히 기존 방법의 개선을 넘어, TDOM이 요구하는 정교함과 실용성이라는 상충될 수 있는 두 가지 목표를 동시에 달성하는 새로운 패러다임을 제시한다. 이는 TDOM의 적용 범위를 확장하고, 도시 계획, 자율 시스템 등 고정밀 공간 데이터가 필수적인 분야에서 2DGS가 핵심 기술로 자리매김할 수 있는 기반을 마련한다. 즉, 2DGS는 TDOM 생성의 비용-효과 곡선을 근본적으로 개선한다.



2DGS는 TDOM 생성에 있어 많은 장점을 제공하지만, 여전히 해결해야 할 몇 가지 한계와 도전 과제가 존재한다.

- **깊이 편향 및 광택 표면 문제:** 2DGS는 광택 있는 표면을 처리할 때 구멍이 발생하거나 깊이 편향 문제가 발생할 수 있다.8 이는 깊이 연속성을 강제하는 데 사용되는 투과율 가중치가 불충분하기 때문일 수 있으며, 실제 표면이 광선-가우시안 교차점의 중간 지점으로 간주되어 깊이가 편향될 수 있다는 분석이 제시된다.8
- **대규모 데이터 처리의 복잡성:** 대규모 이미지를 처리할 때 수많은 가우시안 포인트로 인한 최적화의 어려움이 발생할 수 있다.1 D2GV(Deformable 2D Gaussian Splatting for Video Representation)와 같은 비디오 표현 방식에서는 고정된 GoP(Group of Pictures) 길이가 빠른 장면 전환에 비효율적일 수 있다는 점이 지적된다.13
- **데이터 전처리 및 최적화:** 2DGS의 훈련 병렬화가 3DGS보다 적어 훈련 시간이 길어질 수 있다는 의견도 존재하며 9, 이는 대규모 데이터셋 처리 시 중요한 고려 사항이 된다.
- **텍스처 표현의 한계:** 2DGS는 각 스플랫에 단색을 사용하며, 가우시안 분포로 불투명도를 조절하여 표현력이 제한적일 수 있다.15 이는 복잡하고 미세한 텍스처를 가진 표면을 정확하게 재구성하는 데 어려움을 초래할 수 있다.


이러한 한계점들을 극복하고 2DGS의 적용 범위를 확장하기 위한 활발한 연구가 진행 중이다.

- **향상된 가우시안 표현:**
  - **Gaussian-Hermite Splatting (2DGH):** 양자 물리학에서 영감을 받아 가우시안-헤르미트 커널을 새로운 프리미티브로 사용하여 표현력을 강화하는 연구가 진행된다.16 이 새로운 커널은 타원형을 넘어 변형될 수 있으며, 날카로운 경계를 더 잘 표현하여 미세하고 복잡한 구조 및 불연속적인 가장자리 재구성에 탁월한 성능을 보인다.16
  - **Gaussian Billboards:** 2DGS에 공간적으로 변화하는 색상을 추가하기 위해 스플랫별 텍스처 보간을 사용하는 Gaussian Billboards는 2DGS의 표현력을 높이는 데 기여한다.15 이는 2DGS의 견고한 장면 최적화 능력과 텍스처 매핑의 표현력을 효과적으로 결합한 결과이다.15
- **깊이 정확도 개선:** 깊이 편향 문제를 해결하기 위해 교차하는 가우시안 수와 광선을 따른 누적 불투명도를 모두 고려하는 새로운 기준인 깊이 보정(depth correction)이 도입되어 깊이 정확도를 개선한다.8
- **대규모 및 동적 장면 처리:**
  - **Large Images are Gaussians (LIG):** 수많은 가우시안 포인트 상황에서 대규모 이미지 피팅을 위한 2DGS 표현 변형 및 CUDA 커널 재구현이 이루어졌다.1 Level-of-Gaussian 메커니즘을 통해 저주파 구조 초기화 후 고주파 세부 사항을 위한 2단계 피팅을 도입하여 효율성을 높인다.1
  - **Deformable 2D Gaussian Splatting for Video Representation (D2GV):** 비디오 표현을 위한 변형 가능한 2D 가우시안 스플랫팅이 제안되었다.10 이 방법은 GoP(Group of Pictures) 수준에서 작동하여 수렴 및 확장성을 개선하고, 복잡도를 선형 증가로 감소시킨다.10 또한 CUDA 기반 래스터화를 통해 400+ FPS의 빠른 디코딩이 가능하다.10
  - **TRAN-D:** 희소 시점 투명 객체 깊이 재구성을 위한 2DGS 기반 방법인 TRAN-D는 객체별 가우시안 최적화 및 물리 기반 시뮬레이션을 통한 장면 업데이트를 통해 객체 제거 및 연쇄 반응 움직임을 효과적으로 처리한다.22
- **디지털 트윈 및 자율 시스템 통합:** Gaussian Splatting은 디지털 트윈 공간을 혁신할 잠재력을 가진다.14 드론, 감시 카메라, 자율 시스템이 가우시안 스플랫을 생성하거나 점진적으로 업데이트하고, 스플랫 내에서 자체 위치를 파악하여 객체 감지 등 ML 출력을 통합할 수 있다.14 단일 가우시안 스플랫이 여러 자율 차량이 SLAM 시스템을 통해 업데이트하는 "진실의 원천(source of truth)" 역할을 할 수 있으며 14, XR(확장 현실) 장치와 가우시안 스플랫의 직접 정렬 가능성도 제기된다.14 이러한 발전은 팩토리, 창고, 물류 센터 등 자율/반자율 환경에 큰 변화를 가져올 것으로 예상된다.14

2DGS의 초기 한계점들은 연구 커뮤니티의 활발한 노력으로 빠르게 극복되고 있음을 보여준다. 다양한 변형(2DGH, Gaussian Billboards)과 확장(LIG, D2GV, TRAN-D)은 2DGS가 특정 문제에 대한 맞춤형 해결책으로 진화하고 있음을 의미한다. 이는 2DGS가 단순한 고정된 기술이 아니라, 다양한 응용 분야의 복잡한 요구사항을 충족시키기 위해 지속적으로 적응하고 발전하는 플랫폼으로서의 잠재력을 가짐을 시사한다. 특히, TDOM 생성과 같은 고정밀 공간 재구성 분야에서 이러한 진화는 2DGS의 장기적인 유효성과 적용 가능성을 보장하는 핵심 요소가 된다.


2D Gaussian Splatting (2DGS)은 3D 장면 재구성 및 신규 시점 합성 분야에서 혁신적인 접근 방식으로 등장하였으며, 특히 True Digital Orthophoto Map (TDOM) 생성에 있어 상당한 잠재력을 가진다. 2DGS는 3D 볼륨을 2D 방향성 평면 가우시안 디스크로 축소하여 시점 일관된 기하학을 제공하고 표면을 내재적으로 모델링함으로써, 기존 3DGS의 다중 시점 불일치 문제를 해결하고 NeRF 기반 방법의 느린 훈련 및 렌더링 속도 한계를 개선한다. 이러한 특성은 TDOM이 요구하는 높은 기하학적 정밀도와 조밀한 이미지 특징을 명시적인 DSM이나 오클루전 감지 없이 달성할 수 있게 하여, 전통적인 TDOM 생성 방법의 계산 비용 및 오류 발생 가능성이라는 주요 한계를 극복한다.

그러나 2DGS 역시 광택 있는 표면 처리 시 깊이 편향 및 구멍 발생 문제, 대규모 데이터 처리 시 최적화의 어려움, 그리고 텍스처 표현의 한계와 같은 도전 과제를 안고 있다. 이러한 한계점들은 Gaussian-Hermite Splatting (2DGH)을 통한 표현력 강화, Gaussian Billboards를 통한 텍스처 통합, 깊이 보정을 통한 정확도 개선, 그리고 Large Images are Gaussians (LIG), Deformable 2D Gaussian Splatting for Video Representation (D2GV), TRAN-D와 같은 확장 연구를 통해 활발히 극복되고 있다.

결론적으로 2DGS는 TDOM 생성의 정교함과 실용성을 동시에 확보하는 새로운 패러다임을 제시하며, 도시 계획, 인프라 관리, 자율 시스템을 위한 디지털 트윈 구축 등 고정밀 공간 데이터가 필수적인 분야에서 중추적인 역할을 할 것으로 기대된다. 2DGS는 단순한 기술 개선을 넘어, 다양한 응용 분야의 복잡한 요구사항을 충족시키기 위해 지속적으로 적응하고 발전하는 플랫폼으로서의 잠재력을 보여준다. 향후 연구는 이러한 진화를 더욱 가속화하여 2DGS 기반 TDOM 기술의 적용 범위를 넓히고, 그 성능을 지속적으로 향상시킬 것이다.


1. arxiv.org, 8월 2, 2025에 액세스, https://arxiv.org/html/2502.09039v1
2. arxiv.org, 8월 2, 2025에 액세스, https://arxiv.org/html/2403.17888v1
3. 2D Gaussian Splatting for Geometrically Accurate Radiance Fields - Hugging Face, 8월 2, 2025에 액세스, https://huggingface.co/papers/2403.17888
4. [2403.17888] 2D Gaussian Splatting for Geometrically Accurate Radiance Fields - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/abs/2403.17888
5. [2503.19703] High-Quality Spatial Reconstruction and Orthoimage Generation Using Efficient 2D Gaussian Splatting - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/abs/2503.19703
6. High-Quality Spatial Reconstruction and Orthoimage Generation Using Efficient 2D Gaussian Splatting - ResearchGate, 8월 2, 2025에 액세스, https://www.researchgate.net/publication/390176414_High-Quality_Spatial_Reconstruction_and_Orthoimage_Generation_Using_Efficient_2D_Gaussian_Splatting
7. High-Quality Spatial Reconstruction and Orthoimage Generation Using Efficient 2D Gaussian Splatting - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2503.19703v1
8. arxiv.org, 8월 2, 2025에 액세스, https://arxiv.org/html/2503.06587v1
9. 2DGS vs 3DGS : r/GaussianSplatting - Reddit, 8월 2, 2025에 액세스, https://www.reddit.com/r/GaussianSplatting/comments/1iefikz/2dgs_vs_3dgs/
10. arxiv.org, 8월 2, 2025에 액세스, https://arxiv.org/html/2503.05600v1
11. How to Render a Single Gaussian Splat? - Shi's blog, 8월 2, 2025에 액세스, https://shi-yan.github.io/how_to_render_a_single_gaussian_splat/
12. PS-GS: Gaussian Splatting for Multi-View Photometric Stereo - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2507.18231v1
13. D2GV: Deformable 2D Gaussian Splatting for Video Representation ..., 8월 2, 2025에 액세스, https://arxiv.org/pdf/2503.05600
14. Gaussian splatting is such an impressive technique. I wish it finds ..., 8월 2, 2025에 액세스, https://news.ycombinator.com/item?id=40721275
15. Gaussian Billboards: Expressive 2D Gaussian Splatting with Textures - YouTube, 8월 2, 2025에 액세스, https://www.youtube.com/watch?v=OuK3sxPw5BQ
16. arxiv.org, 8월 2, 2025에 액세스, https://arxiv.org/html/2408.16982v1
17. 2DGH: 2D Gaussian-Hermite Splatting for High-quality Rendering and Better Geometry Reconstruction - ResearchGate, 8월 2, 2025에 액세스, https://www.researchgate.net/publication/383648555_2DGH_2D_Gaussian-Hermite_Splatting_for_High-quality_Rendering_and_Better_Geometry_Reconstruction
18. [Literature Review] 2DGH: 2D Gaussian-Hermite Splatting for High ..., 8월 2, 2025에 액세스, https://www.themoonlight.io/en/review/2dgh-2d-gaussian-hermite-splatting-for-high-quality-rendering-and-better-geometry-reconstruction
19. 2DGH: 2D Gaussian-Hermite Splatting for High-quality Rendering and Better Geometry Reconstruction - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2408.16982
20. [Literature Review] Gaussian Billboards: Expressive 2D Gaussian ..., 8월 2, 2025에 액세스, https://www.themoonlight.io/en/review/gaussian-billboards-expressive-2d-gaussian-splatting-with-textures
21. arxiv.org, 8월 2, 2025에 액세스, https://arxiv.org/abs/2502.09039
22. arxiv.org, 8월 2, 2025에 액세스, https://arxiv.org/html/2507.11069v1
23. (PDF) TRAN-D: 2D Gaussian Splatting-based Sparse-view Transparent Object Depth Reconstruction via Physics Simulation for Scene Update - ResearchGate, 8월 2, 2025에 액세스, https://www.researchgate.net/publication/393723907_TRAN-D_2D_Gaussian_Splatting-based_Sparse-view_Transparent_Object_Depth_Reconstruction_via_Physics_Simulation_for_Scene_Update


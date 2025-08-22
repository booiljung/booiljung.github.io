---
layout: page
title: R3LIVE SLAM 기술에 대한 심층 분석
permalink: /slam/RXLIVE/R3LIVE
---



동시적 위치추정 및 지도작성(Simultaneous Localization and Mapping, SLAM) 기술은 자율 로봇 공학의 핵심으로 자리 잡았다. 이 기술의 발전은 주로 사용되는 센서의 종류에 따라 두 가지 주요 흐름으로 나뉘어 왔다: 시각 SLAM(Visual SLAM)과 LiDAR SLAM. 각 기술은 뚜렷한 장점을 가지고 있지만, 동시에 명백한 한계점 또한 내포하고 있어 현대 로봇 응용 분야에서 '완벽한' 단일 센서 솔루션은 존재하지 않는다는 딜레마에 봉착했다.

시각 SLAM은 저렴하고 크기가 작으며 전력 소모가 적은 카메라를 주 센서로 사용한다.1 카메라는 환경의 풍부한 색상과 텍스처 정보를 포착하여 인간이 직관적으로 이해하기 쉬운 지도를 생성할 수 있다. 그러나 시각 SLAM은 본질적으로 빛에 의존하기 때문에 조명 변화에 매우 취약하다. 극단적으로 어둡거나 밝은 환경, 혹은 급격한 조도 변화는 특징점 추적을 실패하게 만들어 시스템 전체를 마비시킬 수 있다.1 또한, 단안 카메라의 경우 깊이 정보를 직접 측정할 수 없어 스케일 드리프트(scale drift) 문제가 발생하며, 이는 시간이 지남에 따라 추정된 궤적과 지도의 크기가 실제와 달라지는 현상을 야기한다. 텍스처가 부족한 넓은 벽이나 복도 같은 환경에서는 특징점 자체가 부족하여 성능이 급격히 저하되는 문제도 고질적이다.1 스테레오 카메라나 RGB-D 카메라를 사용하면 깊이 정보 문제를 일부 완화할 수 있지만, 측정 거리나 정확도 면에서 여전히 한계가 있다. 결과적으로 시각 SLAM으로 생성된 지도의 기하학적 정밀도는 LiDAR SLAM에 비해 떨어지는 경향이 있다.1

반면, LiDAR SLAM은 레이저를 이용하여 주변 환경까지의 거리를 직접, 그리고 매우 정밀하게 측정한다. 이를 통해 얻어지는 3차원 포인트 클라우드는 환경의 기하학적 구조를 높은 정확도로 표현하며, LiDAR는 자체 광원을 사용하므로 조명 변화에 거의 영향을 받지 않는 강인함을 보인다.1 하지만 LiDAR SLAM 역시 만능은 아니다. 긴 복도, 넓은 평면 벽, 텅 빈 광장과 같이 기하학적 특징이 부족한(geometrically-degenerated) 환경에 직면하면, 연속된 스캔 간의 정합(registration)에 필요한 충분한 제약 조건을 찾지 못해 쉽게 실패할 수 있다.1 특히 시야각(Field of View, FoV)이 좁은 고정형 LiDAR(solid-state LiDAR)를 사용할 경우 이러한 문제는 더욱 심각해진다.4 근본적으로 LiDAR는 환경의 기하학적 정보만을 제공할 뿐, 색상 정보를 획득할 수 없어 생성된 지도는 현실감이 떨어지고, 인간의 시각적 해석이나 특정 응용 분야에 활용하기에는 정보가 부족하다.1

이러한 각 센서의 명백한 한계는 자연스럽게 다중 센서 융합(Multi-sensor Fusion)이라는 필연적인 결론으로 이어진다. LiDAR의 정밀하고 조명에 강인한 기하 구조 측정 능력과, 카메라의 풍부하고 저렴한 색상 및 텍스처 정보 획득 능력을 결합하면 각 센서가 단독으로 사용될 때 발생하는 성능 저하 시나리오를 상호 보완할 수 있다.1 예를 들어, 기하학적으로 평탄한 복도에서 LiDAR가 헤매고 있을 때, 벽의 미세한 텍스처나 포스터 같은 시각 정보가 위치 추정을 도울 수 있다. 반대로, 조명이 갑자기 꺼져 카메라가 무용지물이 되어도 LiDAR는 안정적으로 주변 구조를 측정하여 시스템이 중단되는 것을 막을 수 있다. 이처럼 상호 보완을 통해 SLAM 시스템의 전반적인 강인성(robustness)과 정확도(accuracy)를 극대화하는 것이 현대 SLAM 연구의 핵심 과제가 되었고, R3LIVE는 바로 이 지점에서 출발한다.6


R3LIVE는 LiDAR, 관성 측정 장치(Inertial Measurement Unit, IMU), 그리고 시각(카메라) 센서를 긴밀하게 결합(Tightly-coupled)한 프레임워크로, 그 이름 자체가 설계 철학을 명확하게 드러낸다. R3LIVE는 **R**obust, **R**eal-time, **R**GB-colored의 세 가지 핵심 목표를 동시에 달성하기 위해 탄생했다.4

첫째, **강인성(Robustness)**은 R3LIVE의 가장 중요한 설계 목표다. 앞서 언급했듯이 LiDAR나 카메라 센서는 특정 환경에서 성능이 급격히 저하되는 '열화(degeneration)' 현상을 겪는다. R3LIVE는 고주파의 운동 정보를 제공하는 IMU를 센서 융합의 중심으로 삼아, LiDAR나 카메라로부터의 정보가 일시적으로 불안정하거나 유실되더라도 안정적인 상태 추정을 이어갈 수 있도록 설계되었다.12 한 센서의 약점을 다른 센서의 강점으로 보완하는 긴밀 결합 방식을 통해, 기존 단일 센서 기반 SLAM 시스템들이 실패했던 도전적인 환경에서도 멈추지 않고 동작하는 것을 목표로 한다.8

둘째, **실시간성(Real-time)**은 로봇 응용 분야에서 필수적인 요구사항이다. 자율 로봇이 주변 환경을 인식하고 경로를 계획하며 제어하기 위해서는 자신의 위치와 자세를 지연 없이 알아야 한다. R3LIVE는 계산적으로 매우 효율적인 알고리즘 구조를 채택하여, 전체 시스템이 실시간으로 동작하도록 보장한다.4 이는 복잡한 최적화 문제를 여러 개의 작은 문제로 나누어 푸는 아키텍처와, 계산량을 획기적으로 줄인 칼만 필터 업데이트 방식 등을 통해 달성된다.

셋째, **고품질 RGB 컬러 맵(High-quality RGB-colored Map)** 생성은 R3LIVE를 단순한 위치 추정 시스템 이상으로 만드는 핵심 기능이다. R3LIVE는 LiDAR와 IMU를 통해 정밀한 3차원 기하 구조를 구축하는 동시에, 카메라 이미지를 이용해 이 구조에 생생한 색상을 입힌다.4 그 결과로 생성되는 정밀하고 밀도 높은 3D RGB 컬러 맵은 단순한 로봇 내비게션을 넘어, 측량, 맵핑, 가상현실(VR)/증강현실(AR), 디지털 트윈 구축 등 다양한 고부가가치 응용 분야에 직접적으로 활용될 수 있는 잠재력을 가진다.9

결론적으로 R3LIVE는 어느 한 가지 목표를 위해 다른 것을 희생하는 것이 아니라, 강인성, 실시간성, 고품질 맵핑이라는 세 마리 토끼를 모두 잡기 위한 종합적인 해결책으로 제안되었다. 이는 센서들의 물리적 특성과 알고리즘적 장점을 최적으로 조합한 결과물이라 할 수 있다.


R3LIVE는 홍콩과기대(HKUST) MARS 연구실에서 수행된 일련의 연구 결과물로, 그 기술적 뿌리는 이전 연구인 R2LIVE에 두고 있다. R3LIVE와 R3LIVE++로 이어지는 진화 과정은 단순한 성능 개선을 넘어, SLAM 기술의 목표가 어떻게 심화되고 있는지를 보여주는 중요한 지표다.

R3LIVE는 R2LIVE를 기반으로 개발되었지만, 특히 시각-관성 오도메트리(VIO) 서브시스템의 아키텍처를 완전히 재설계하여 성능과 강인성을 한 단계 끌어올렸다.4 R2LIVE가 시각 정보를 융합하는 초기 시도였다면, R3LIVE는 이를 보다 체계적이고 강인한 방식으로 통합하여 'RGB-colored'라는 목표를 본격적으로 실현했다. 이는 기하학적으로 정확한 지도를 만들고 그 위에 색을 칠하는 '기하학 우선(geometry-first)' 접근법으로 볼 수 있다.4

여기서 한 걸음 더 나아간 것이 **R3LIVE++**이다. 이 시스템은 단순한 RGB 색상 값을 맵에 입히는 것을 넘어, 환경에 존재하는 각 지점의 물리적 속성인 **'Radiance(광휘)'**를 모델링하고 재구성하는 것을 목표로 한다.15 Radiance는 관찰 시점이나 조명에 따라 변하는 픽셀 밝기와 달리, 표면의 고유한 방사 특성을 나타내는 물리량이다. 이를 정확하게 추정하기 위해 R3LIVE++는 카메라 센서의 물리적 특성까지 고려한다. 예를 들어, 카메라의 비선형 응답 함수(non-linear response function, 밝기에 따라 센서가 비선형적으로 반응하는 특성), 렌즈 주변부가 어두워지는 비네팅(vignetting) 효과와 같은 광도계(photometric) 왜곡을 모델링하고 온라인으로 보정한다. 또한, 이미지마다 달라지는 카메라의 노출 시간을 실시간으로 추정하여 Radiance 계산에 반영한다. 이러한 정교한 물리 모델링을 통해 R3LIVE++는 기존 R3LIVE 대비 위치 추정 및 맵핑의 정확도를 극적으로 향상시켰다.1

이러한 기술적 진화는 SLAM의 패러다임 전환을 시사한다. R3LIVE가 '나는 지금 어디에 있는가?'라는 위치 추정과 '주변이 어떻게 생겼는가?'라는 기하학적 맵핑에 집중했다면, R3LIVE++는 여기서 더 나아가 **'센서와 무관하게 이 세상이 물리적으로 어떻게 보이는가?'**라는 근본적인 질문에 답하고자 한다. 이는 SLAM의 목표가 단순한 항법 보조 도구를 넘어, 현실 세계를 매우 높은 충실도(high-fidelity)로 복제하는 **디지털 트윈(Digital Twin)** 생성으로 확장되고 있음을 보여준다. 실제로 R3LIVE++의 결과물은 고다이나믹레인지(HDR) 이미징, 가상 환경 탐사, 3D 비디오 게임 등 사진처럼 사실적인(photorealistic) 표현이 필수적인 응용 분야에 직접 활용될 수 있다.16 이처럼 R2LIVE에서 R3LIVE++로의 발전은 센서 융합 SLAM 기술이 얼마나 더 정교하고 야심 찬 목표를 향해 나아가고 있는지를 명확히 보여주는 사례다.


R3LIVE의 핵심적인 강점과 실시간 성능은 그 독특한 시스템 아키텍처에서 비롯된다. 이 시스템은 하나의 거대한 단일체(monolithic) 구조가 아니라, 명확하게 역할이 분담된 두 개의 하위 시스템, 즉 LiDAR-Inertial Odometry (LIO)와 Visual-Inertial Odometry (VIO)가 병렬적으로 동작하는 구조를 채택하고 있다.8 이 두 시스템은 분리되어 있는 것처럼 보이지만, 실제로는 에러 상태 반복 칼만 필터(Error-State Iterated Kalman Filter, ESIKF)의 상태 벡터를 공유하며 긴밀하게 상호작용한다.


R3LIVE의 아키텍처는 '관심사의 분리(separation of concerns)'라는 공학적 원칙을 충실히 따른다. 시스템의 전체 상태(위치, 자세, 속도, 바이어스 등)는 ESIKF라는 단일 필터 내에서 관리된다. 이 필터를 중심으로 LIO와 VIO라는 두 개의 독립적인 업데이트 모듈이 동작한다.

- **LIO 서브시스템**은 LiDAR와 IMU 데이터를 받아 시스템의 상태를 예측하고, LiDAR 측정값을 이용해 상태를 업데이트한다. 이 과정에서 환경의 기하학적 뼈대를 구축하는 역할을 담당한다.
- **VIO 서브시스템**은 카메라와 IMU 데이터를 받아, LIO가 구축한 기하학적 맵 위에 색상을 입히고, 이 과정에서 발생하는 광도 오차를 이용해 시스템의 상태를 추가적으로 미세 조정(refine)한다.

두 서브시스템은 병렬적으로 실행되지만, 데이터 흐름은 유기적으로 연결되어 있다. LIO가 추정한 최신 상태와 맵 정보는 VIO가 색상을 입히고 상태를 보정하는 데 사용된다. 그리고 VIO에 의해 보정된 상태는 다시 ESIKF의 상태 벡터에 반영되어, 다음 LIO 업데이트 시 더 정확한 초기값으로 활용된다. 이러한 구조는 각 센서의 장점을 극대화하면서도 시스템 전체의 복잡도를 관리하는 데 효과적이다.

이러한 분리된 아키텍처는 강인성과 실시간 성능 확보를 위한 전략적인 선택이다. LVI-SAM과 같이 모든 센서 측정값을 하나의 거대한 요인 그래프(factor graph) 최적화 문제로 푸는 방식과 대조된다.19 R3LIVE의 구조는 몇 가지 중요한 장점을 가진다. 첫째, **실패 분리(Failure Isolation)**가 가능하다. 예를 들어, 카메라가 가려져 시각 정보가 들어오지 않아도(visual degradation), LIO 서브시스템은 독립적으로 계속 동작하여 시스템 전체의 안정성을 유지한다. 반대로, LiDAR가 특징 없는 벽을 향하고 있어 기하학적 정보가 부족할 때(geometric degradation), VIO 서브시스템이 광도 정보를 기반으로 한 보정 업데이트를 제공하여 드리프트를 억제할 수 있다. 둘째, 

**계산 효율성**이 높다. 기하학적 정합 문제(LIO)와 광도 오차 최소화 문제(VIO)를 분리된 단계에서 해결함으로써, 하나의 거대하고 복잡한 비선형 최적화 문제를 푸는 것보다 계산 부하를 줄일 수 있다. 이는 R3LIVE가 실시간 성능을 달성하는 데 결정적인 역할을 한다.9 셋째, **모듈성(Modularity)**이 뛰어나다. LIO(FAST-LIO 기반)와 VIO는 개념적으로 명확히 구분되는 모듈이므로, 시스템을 이해하고 디버깅하며, 향후 특정 모듈만 업그레이드하기에 용이하다. 이는 일괄 최적화(batch optimization) 방식이 제공할 수 있는 잠재적인 전역 정확도보다 실시간 강인성을 우선시한 의도적인 공학적 트레이드오프의 결과라 할 수 있다.


R3LIVE의 LIO 서브시스템은 전체 시스템의 중추적인 역할을 담당하며, 그 기반은 홍콩대 MARS 연구실의 이전 연구 성과인 FAST-LIO 21 및 FAST-LIO2 23에 두고 있다.8 LIO의 주된 임무는 LiDAR 포인트 클라우드와 IMU 데이터를 긴밀하게 결합하여 시스템의 상태(위치, 자세, 속도 등)를 고주파로 추정하고, 이를 바탕으로 전역 맵의 정밀한 기하학적 구조, 즉 3D 포인트들의 위치를 구축하는 것이다.4

LIO 서브시스템의 가장 큰 특징 중 하나는 전통적인 LiDAR SLAM 방식에서 흔히 사용되는 **특징점 추출(feature extraction) 과정을 생략**한다는 점이다. LOAM 계열의 알고리즘들은 포인트 클라우드에서 코너(edge)나 평면(planar)과 같은 명확한 특징점을 먼저 추출하고, 이 특징점들만을 이용해 정합을 수행한다. 이 방식은 계산량을 줄이는 데 효과적이지만, 특징점 추출 과정에서 많은 정보를 버리게 되며, 특징이 뚜렷하지 않은 환경에서는 성능이 저하된다. 반면, R3LIVE의 LIO는 FAST-LIO2의 접근법을 따라, LiDAR 스캔에서 얻은 **원시 포인트(raw points)를 직접** 전역 맵에 등록한다.23 이 방식은 환경에 존재하는 미세한 기하학적 특징들까지 모두 활용할 수 있게 하여, 결과적으로 더 높은 정확도와 강인성을 달성하게 해준다.

또한, LIO 서브시스템은 맵을 효율적으로 관리하고 검색하기 위해 **ikd-Tree**라는 동적 KD-Tree 데이터 구조를 사용한다.9 전통적인 KD-Tree는 한 번 생성되면 구조를 변경하기 어렵기 때문에, 새로운 포인트가 계속 추가되는 SLAM 응용에는 비효율적이다. ikd-Tree는 새로운 포인트를 증분적으로 추가(incremental update)하고, 트리의 균형을 동적으로 재조정(dynamic re-balancing)하는 기능을 지원하여, 맵이 계속 커지더라도 빠른 최근접 이웃 탐색(k-Nearest Neighbor search, kNN) 성능을 유지한다. 이는 실시간 맵핑과 위치 추정을 위한 핵심 기술이다.


VIO 서브시스템은 LIO가 구축한 정밀한 기하학적 뼈대 위에 색과 생명을 불어넣는 예술가와 같은 역할을 한다. VIO의 주된 임무는 LIO가 만든 3D 포인트 클라우드 맵에 카메라 이미지로부터 얻은 RGB 색상 정보를 렌더링하여, 사실적인 3D 컬러 맵을 완성하는 것이다.4

R3LIVE의 VIO가 특별한 이유는 시각 정보를 활용하는 방식에 있다. 대부분의 전통적인 VIO 시스템(예: VINS-Mono, ORB-SLAM)은 특징점 기반(feature-based) 접근법을 사용한다. 즉, 이미지에서 코너나 블롭(blob)과 같은 특징점을 검출하고, 연속된 프레임 간에 이 특징점들을 추적/매칭하며, 재투영 오차(reprojection error)를 최소화하여 카메라의 움직임을 추정한다. 이 방식은 정확도가 높지만, 텍스처가 부족한 환경에서는 특징점 검출 자체가 실패하여 시스템 전체가 멈출 수 있다.

R3LIVE는 이러한 문제를 해결하기 위해 **직접법(direct method)**에 기반한 독창적인 VIO 아키텍처를 채택했다. 이 시스템은 명시적인 특징점 추출 및 매칭 과정 없이, **'프레임-맵 간 광도 오차(frame-to-map photometric error)'**를 직접 최소화한다.4 이 과정은 다음과 같이 이루어진다:

1. LIO가 구축한 전역 맵에는 각 3D 포인트의 위치와 함께, 이전에 관측된 이미지를 통해 추정된 RGB 색상 값이 저장되어 있다.
2. 새로운 카메라 이미지가 들어오면, 맵에 있는 3D 포인트들을 현재 추정된 카메라 자세를 이용해 이미지 평면에 투영(project)한다.
3. 그 다음, 각 맵 포인트에 저장되어 있던 '기대 색상 값'과, 이미지에 투영된 위치의 실제 '픽셀 색상 값' 사이의 차이(즉, 광도 오차)를 계산한다.
4. 시스템은 이 광도 오차의 총합이 최소가 되도록 ESIKF의 상태 벡터(특히 카메라의 자세)를 미세하게 보정한다.

이 직접법 기반의 접근 방식은 R3LIVE가 텍스처가 거의 없는 환경에서도 강인하게 동작할 수 있는 핵심 비결이다. 명확한 코너나 특징점이 없더라도, 벽면의 미세한 명암 변화나 색상 그라데이션만 존재한다면 VIO는 이를 활용하여 카메라의 자세를 보정할 수 있는 정보를 얻을 수 있다. 이는 특징점 기반 방식의 근본적인 취약점을 극복하고, R3LIVE가 설계 목표로 하는 극한 환경에서의 강인성을 달성하는 데 결정적인 기여를 한다.4


R3LIVE 시스템 내에서 데이터는 두 개의 하위 시스템을 통해 유기적으로 흐르며 상태를 지속적으로 동기화하고 정제한다. 전체 프로세스는 ESIKF를 중심으로 이루어진다.

1. **IMU 전파 (Propagation):** 모든 프로세스의 기본은 IMU 데이터다. 200Hz 이상의 고주파로 수신되는 IMU 데이터(가속도, 각속도)는 ESIKF의 운동 모델에 따라 시스템의 상태(자세, 위치, 속도)를 지속적으로 예측(전파)한다. 이는 LIO와 VIO 양쪽 모두에게 다음 센서 측정값이 들어오기 전까지의 움직임에 대한 사전 정보(prior)를 제공한다.
2. **LIO 업데이트 (Update):** LiDAR 스캔(일반적으로 10Hz)이 수신되면 LIO 서브시스템이 활성화된다. LIO는 IMU 전파를 통해 예측된 현재 상태를 초기값으로 사용하여, 들어온 LiDAR 포인트들을 전역 맵에 정합한다. 이 과정에서 발생하는 포인트-평면 간의 기하학적 오차를 최소화하여 ESIKF의 상태 벡터를 업데이트한다. 이 업데이트를 통해 시스템의 상태는 더욱 정확해지고, 전역 맵의 기하학적 구조 또한 새로운 포인트들로 갱신된다.
3. **VIO 업데이트 (Update):** 카메라 프레임(일반적으로 20-30Hz)이 수신되면 VIO 서브시스템이 활성화된다. VIO는 LIO에 의해 가장 최근에 업데이트된 ESIKF의 상태 추정치와 전역 맵 구조를 가져온다. 이 정보를 바탕으로 맵 포인트들을 현재 카메라 이미지에 투영하고, 앞서 설명한 '프레임-맵 간 광도 오차'를 계산한다. VIO는 이 광도 오차를 최소화하는 방향으로 ESIKF의 상태 벡터를 한 번 더 보정한다. 이 과정은 LIO가 제공한 기하학적 추정치를 시각 정보로 다시 한번 검증하고 미세 조정하는 역할을 한다.
4. **상태 피드백 루프:** VIO에 의해 보정된 상태는 ESIKF의 최종 상태로 확정되며, 이는 다시 다음 LiDAR 스캔이 들어왔을 때 LIO가 사용할 초기 추정치가 된다. 이처럼 LIO -->> VIO -->> LIO로 이어지는 피드백 루프는 시스템의 상태 추정치가 시간이 지남에 따라 발산하지 않고, 두 종류의 센서 정보(기하학, 광도)에 의해 지속적으로 정제되도록 보장한다.

이러한 데이터 흐름과 상태 동기화 메커니즘은 R3LIVE가 서로 다른 특성과 주기를 가진 세 개의 센서(LiDAR, IMU, Camera)를 효과적으로 융합하여, 강인하고 정확하며 실시간으로 동작하는 SLAM 시스템을 구현하는 핵심적인 기반이 된다.


R3LIVE의 성능은 정교한 수학적 모델링에 기반한다. 시스템은 LIO와 VIO라는 두 축을 중심으로 동작하며, 이 둘은 에러 상태 반복 칼만 필터(ESIKF)라는 공통된 프레임워크 위에서 상태를 공유하고 업데이트한다. 각 서브시스템의 수학적 원리를 깊이 있게 살펴보자.


LIO 서브시스템은 FAST-LIO2의 철학을 계승하여, ESIKF를 통해 LiDAR와 IMU 데이터를 긴밀하게 결합한다.


시스템이 추정하고자 하는 모든 변수는 하나의 상태 벡터 $\mathbf{x}$로 정의된다. 이 상태 벡터는 단순한 유클리드 공간이 아닌, 회전 그룹 $SO(3)$와 유클리드 공간 $\mathbb{R}^n$이 결합된 매니폴드(manifold) 상에 존재한다. FAST-LIO2를 기준으로, R3LIVE의 상태 벡터는 다음과 같이 구성된다.23
$$
\mathbf{x} \triangleq \begin{pmatrix} {}^G\mathbf{R}_I \\ {}^G\mathbf{p}_I \\ {}^G\mathbf{v}_I \\ \mathbf{b}_\omega \\ \mathbf{b}_a \\ {}^G\mathbf{g} \\ {}^I\mathbf{R}_L \\ {}^I\mathbf{p}_L \end{pmatrix}
$$
각 항목의 의미는 다음과 같다:

- ${}^G\mathbf{R}_I \in SO(3)$: 전역 좌표계(Global frame, G)에 대한 IMU 좌표계(I)의 회전 행렬.
- ${}^G\mathbf{p}_I \in \mathbb{R}^3$: 전역 좌표계에서 바라본 IMU의 위치 벡터.
- ${}^G\mathbf{v}_I \in \mathbb{R}^3$: 전역 좌표계에서 바라본 IMU의 속도 벡터.
- $\mathbf{b}_\omega \in \mathbb{R}^3$: 자이로스코프의 바이어스.
- $\mathbf{b}_a \in \mathbb{R}^3$: 가속도계의 바이어스.
- ${}^G\mathbf{g} \in \mathbb{R}^3$: 전역 좌표계에서의 중력 벡터. 시스템 초기에는 알 수 없으므로 추정 대상에 포함된다.
- ${}^I\mathbf{R}_L \in SO(3)$: IMU 좌표계에 대한 LiDAR 좌표계(L)의 회전 행렬 (외인성 파라미터).
- ${}^I\mathbf{p}_L \in \mathbb{R}^3$: IMU 좌표계에서 바라본 LiDAR의 위치 벡터 (외인성 파라미터).

이 상태 벡터는 전체 SO(3)×R15×SO(3)×R3 매니폴드에 속하며, 총 24 자유도(degrees of freedom)를 가진다. R3LIVE는 이 외인성 파라미터들을 온라인으로 추정하거나, 사전에 정밀하게 캘리브레이션된 값을 사용할 수 있다.


새로운 IMU 측정값(‘ωm,am‘)이 들어올 때마다, 시스템은 상태 벡터의 동역학을 기술하는 연속 시간 운동학 모델(continuous-time kinematic model)을 이용해 상태를 다음 시간으로 전파(propagate)시킨다. 이는 칼만 필터의 예측(prediction) 단계에 해당한다.23
$$
\begin{aligned}
{}^G\dot{\mathbf{R}}_I &= {}^G\mathbf{R}_I (\boldsymbol{\omega}_m - \mathbf{b}_\omega - \mathbf{n}_\omega)^\wedge \\
{}^G\dot{\mathbf{p}}_I &= {}^G\mathbf{v}_I \\
{}^G\dot{\mathbf{v}}_I &= {}^G\mathbf{R}_I (\mathbf{a}_m - \mathbf{b}_a - \mathbf{n}_a) + {}^G\mathbf{g} \\
\dot{\mathbf{b}}_\omega &= \mathbf{n}_{b_\omega} \\
\dot{\mathbf{b}}_a &= \mathbf{n}_{b_a}
\end{aligned}
$$
여기서 $\boldsymbol{\omega}_m$과 $\mathbf{a}_m$은 각각 자이로와 가속도계의 측정값이며, $\mathbf{n}$으로 시작하는 항들은 각 측정과 바이어스에 포함된 백색 가우시안 노이즈(white Gaussian noise)를 의미한다. 바이어스는 랜덤 워크(random walk) 프로세스로 모델링된다. 연산자 $(\cdot)^\wedge$는 3차원 벡터를 3x3 반대칭 행렬(skew-symmetric matrix)로 변환하는 연산으로, 회전 운동학을 표현하는 데 사용된다. 이 미분 방정식들을 IMU 샘플링 시간 $\Delta t$ 동안 적분하여 다음 시점의 상태와 공분산을 예측한다.


LiDAR 스캔이 들어오면 칼만 필터의 업데이트 단계가 수행된다. FAST-LIO 계열의 핵심은 각 LiDAR 포인트를 독립적인 측정값으로 보고, 이 포인트가 맵 상의 지역 평면(local plane)에 속해야 한다는 기하학적 제약을 이용하는 것이다.

LiDAR 스캔의 j번째 포인트 L_p_j가 있다고 하자. 이 포인트는 현재 추정된 상태 $\mathbf{x}_k$에 포함된 변환 행렬들(${}^G\mathbf{T}_{I_k} = ({}^G\mathbf{R}_{I_k}, {}^G\mathbf{p}_{I_k})$와 ${}^I\mathbf{T}_L = ({}^I\mathbf{R}_L, {}^I\mathbf{p}_L)$)을 통해 전역 좌표계로 변환될 수 있다.
$$
{}^G\mathbf{p}_j = {}^G\mathbf{R}_{I_k}({}^I\mathbf{R}_L {}^L\mathbf{p}_j + {}^I\mathbf{p}_L) + {}^G\mathbf{p}_{I_k}
$$
이제, 전역 맵(ikd-Tree로 구현됨)에서 변환된 포인트 ${}^G\mathbf{p}_j$와 가장 가까운 5개의 포인트를 찾는다. 이 5개의 포인트들은 지역적으로 하나의 작은 평면을 형성한다고 가정할 수 있다. 주성분 분석(PCA) 등을 통해 이 평면의 법선 벡터(normal vector) ${}^G\mathbf{u}_j$와 평면 위의 한 점(보통 5개 포인트의 중심점) ${}^G\mathbf{q}_j$를 계산한다.

이상적으로, 참 값인 포인트 ${}^G\mathbf{p}_j$는 이 평면 위에 정확히 놓여야 한다. 따라서, 포인트와 평면 사이의 거리(point-to-plane distance)는 0이 되어야 한다. 이 거리가 바로 측정 잔차(measurement residual) $z_j$로 정의된다.23
$$
z_j = {}^G\mathbf{u}_j^T \left( ({}^G\mathbf{R}_{I_k}{}^I\mathbf{R}_L {}^L\mathbf{p}_j + {}^G\mathbf{R}_{I_k}{}^I\mathbf{p}_L + {}^G\mathbf{p}_{I_k}) - {}^G\mathbf{q}_j \right)
$$
LIO의 목표는 모든 유효한 LiDAR 포인트 $j=1, \dots  , m$에 대한 이 잔차들의 가중 제곱 합, 즉 비용 함수 $\sum_{j=1}^{m} ||z_j||^2_{\mathbf{R}_j^{-1}}$를 최소화하는 상태 $\mathbf{x}_k$를 찾는 것이다. 여기서 $\mathbf{R}_j$는 측정 노이즈의 공분산이다.


이 비선형 최소 제곱 문제를 풀기 위해 R3LIVE는 ESIKF를 사용한다. ESIKF는 상태 벡터 자체를 직접 다루는 대신, 현재 추정치 주변의 작은 '오차 상태(error state)' $\delta\mathbf{x}$를 추정한다. 오차 상태는 매니폴드의 접선 공간(tangent space)에 정의되므로 일반적인 벡터 연산이 가능하다.

업데이트 과정은 다음과 같이 요약된다:

1. **잔차 계산:** 현재 상태 추정치 $\hat{\mathbf{x}}_k$를 사용하여 모든 포인트에 대한 잔차 $z_j$를 계산한다.

2. **자코비안 계산:** 잔차 $z_j$를 오차 상태 $\delta\mathbf{x}$에 대해 편미분하여 자코비안 행렬 $\mathbf{H}_j$를 구한다.

3. **칼만 게인 계산:** 예측된 공분산 $\mathbf{P}_k$, 자코비안 $\mathbf{H}_k =^T$, 측정 노이즈 공분산 $\mathbf{R}_k$를 이용하여 칼만 게인 $\mathbf{K}$를 계산한다.
   $$
   \mathbf{K} = (\mathbf{H}_k^T \mathbf{R}_k^{-1} \mathbf{H}_k + \mathbf{P}_k^{-1})^{-1} \mathbf{H}_k^T \mathbf{R}_k^{-1}
   $$
   FAST-LIO의 핵심 기여 중 하나는 이 $\mathbf{K}$를 계산하는 공식을 변형하여, 측정값의 수 `m`(수천 개에 달할 수 있음)이 아닌 상태 벡터의 차원(24)에만 의존하도록 만들어 계산 복잡도를 $\mathcal{O}(m)$에서 $\mathcal{O}(1)$ 수준으로 획기적으로 줄인 것이다.22

4. **상태 업데이트:** 계산된 칼만 게인과 잔차를 이용해 최적의 오차 상태 $\delta\hat{\mathbf{x}}$를 추정하고, 이를 현재 상태 추정치에 더하여 상태를 업데이트한다. 회전과 같은 매니폴드 변수는 덧셈 대신 특별한 $\boxplus$ 연산(retraction)을 사용한다.
   $$
   \hat{\mathbf{x}}_k^{new} = \hat{\mathbf{x}}_k \boxplus \mathbf{K} \mathbf{z}_k
   $$
   **반복:** 업데이트된 상태 $\hat{\mathbf{x}}_k^{new}$를 새로운 추정치로 삼아 1-4 과정을 잔차가 수렴할 때까지 여러 번 반복(iterate)한다. 이는 측정 모델의 비선형성이 강할 때 더 정확한 해를 찾는 데 도움을 준다.21


VIO 서브시스템은 LIO가 제공한 상태와 맵을 기반으로, 시각 정보를 이용해 상태를 한 번 더 정제하는 역할을 한다.


VIO의 핵심은 직접법(direct method)에 기반한 광도 오차 최소화다. LIO가 구축한 전역 맵에는 3D 포인트 $\mathbf{P}_i$의 위치와 함께, 이전에 관측된 이미지를 통해 추정된 해당 포인트의 RGB 색상 값 `C(\mathbf{P}_i)`이 저장되어 있다.

새로운 카메라 이미지 $\mathcal{I}_k$가 들어오면, 맵에 있는 포인트 $\mathbf{P}_i$를 현재 카메라 자세 $\mathbf{T}_{cw}$와 카메라 내부 파라미터(intrinsic matrix) $\mathbf{K}_{cam}$을 이용해 이미지 평면에 투영한다. 이 투영 함수를 $\pi(\cdot)$라고 하면, 이미지 상의 픽셀 좌표 $\mathbf{u}_i$는 다음과 같다.

이때, 광도 오차 잔차 `r_photo`는 맵 포인트에 저장된 기대 색상 `C(\mathbf{P}_i)`와, 이미지의 투영된 위치에서 실제로 측정된 픽셀 색상 $\mathcal{I}_k(\mathbf{u}_i)$의 차이로 정의된다.4
$$
r_{\text{photo}, i} = C(\mathbf{P}_i) - \mathcal{I}_k(\pi(\mathbf{K}_{cam} \mathbf{T}_{cw} \mathbf{P}_i))
$$
VIO는 이 광도 오차의 가중 제곱 합 $\sum_{i} ||r_{\text{photo}, i}||^2$를 최소화하도록 ESIKF의 상태 벡터(주로 카메라 자세 $\mathbf{T}_{cw}$에 영향을 주는 ${}^G\mathbf{R}_I, {}^G\mathbf{p}_I$)를 추가적으로 보정한다. 이 과정은 LIO 업데이트와 유사하게 ESIKF 프레임워크 내에서 잔차와 자코비안을 계산하고 상태를 업데이트하는 방식으로 이루어진다.


R3LIVE++는 이 모델을 한 단계 더 발전시켜 물리적으로 더욱 의미 있는 Radiance를 추정한다.15 단순히 RGB 값을 비교하는 대신, 카메라의 이미지 생성 과정을 모델링한다. 이미지 센서가 기록하는 픽셀 강도 $I$는 실제 세계의 Radiance $L$, 카메라의 비선형 응답 함수 $g(/)$, 렌즈 비네팅 효과 $V(u)$, 그리고 노출 시간 $t$의 복합적인 결과물이다.

R3LIVE++는 이 관계를 역으로 이용하여, 측정된 픽셀 강도 $I(u)$로부터 미지의 Radiance $L$과 다른 파라미터들(응답 함수 $g$, 비네팅 $V$, 노출 시간 $t$)을 추정한다. 이 파라미터들은 ESIKF 상태 벡터에 포함되어 온라인으로 최적화된다. 비용 함수는 맵 포인트의 추정된 Radiance와 카메라 파라미터들로부터 예측된 픽셀 값과, 이미지에서 실제로 측정된 픽셀 값의 차이를 최소화하는 방향으로 구성된다.1 이 정교한 광도 보정 모델 덕분에 R3LIVE++는 다양한 조명과 카메라 설정 하에서도 일관되고 정확한 맵을 생성하고, 더 높은 위치 추정 정확도를 달성할 수 있다.


R3LIVE의 가치는 이론적 정교함뿐만 아니라, 실제 환경에서의 성능과 강인성을 통해 증명된다. 특히, 단일 센서 시스템이 실패하기 쉬운 열악한 환경(degenerated environment)에서 얼마나 안정적으로 동작하는지가 R3LIVE의 핵심적인 성능 지표다.


R3LIVE의 아키텍처는 센서 열화 상황에 대처하기 위해 전략적으로 설계되었다. 각 센서의 약점을 다른 센서가 보완하는 상호 보완적 융합은 시스템의 전반적인 강인성을 극대화한다.5

- **LiDAR 열화 상황:** 긴 복도, 넓은 공터, 안개가 낀 환경 등 주변에 뚜렷한 기하학적 특징이 부족하여 LiDAR 스캔만으로는 움직임을 추정하기 어려운 시나리오가 있다. 전통적인 LiDAR SLAM은 이러한 환경에서 큰 드리프트를 겪거나 실패한다. R3LIVE에서는 LIO 서브시스템의 성능이 저하되더라도, VIO 서브시스템이 계속해서 동작한다. 벽면의 미세한 텍스처, 바닥의 무늬, 멀리 있는 건물의 윤곽 등 시각 정보를 이용한 광도 오차 최소화를 통해 드리프트를 억제하고 안정적인 자세 추정을 돕는다.9 즉, 시각 정보가 기하학 정보의 부족을 메우는 보조적인 역할을 수행한다.
- **카메라 열화 상황:** 반대로, 극도로 어둡거나 밝은 환경, 빠른 움직임으로 인한 심한 모션 블러, 텍스처가 전혀 없는 순백색의 벽면 등 시각 정보가 거의 없는 상황에서는 VIO의 성능이 저하된다. 특징점 기반 VIO는 이러한 환경에서 즉시 실패하지만, R3LIVE의 직접법 VIO도 한계에 부딪힐 수 있다. 그러나 이때는 LIO 서브시스템이 핵심적인 역할을 한다. LiDAR는 조명과 무관하게 안정적으로 주변의 기하학적 구조를 측정하고, IMU는 고주파로 움직임을 추정한다. 이 둘의 긴밀한 결합을 통해 시스템은 시각 정보 없이도 정확한 위치 추정을 이어나갈 수 있다.4
- **동시 열화 상황:** R3LIVE의 진정한 강인성은 LiDAR와 카메라가 동시에 열화되는 최악의 시나리오에서 드러난다. 예를 들어, 텍스처 없는 긴 복도를 빠르게 움직이는 경우, LiDAR는 기하학적 특징 부족으로, 카메라는 텍스처 부족과 모션 블러로 인해 각각 성능이 저하된다. R3LIVE는 이러한 극한 상황에서도 IMU를 융합의 중심으로 삼아, 남아있는 미약한 기하학적 정보와 광도 정보를 최대한 활용하여 시스템의 안정성을 유지하고 실패를 방지한다. 개발팀은 논문과 공개된 데이터셋을 통해 이러한 동시 열화 환경에서도 R3LIVE가 강인하게 동작함을 실험적으로 증명했다.8


알고리즘의 성능을 객관적으로 평가하고 재현 가능성을 높이기 위해, 개발팀은 자체적으로 `R3LIVE-Dataset`이라는 대규모 데이터셋을 구축하여 공개했다.8 이는 단순히 알고리즘을 발표하는 것을 넘어, 커뮤니티의 발전에 기여하고 자신들의 연구 성과를 공정하게 평가받기 위한 노력의 일환이다.

이 데이터셋은 홍콩과기대(HKUST)와 홍콩대(HKU) 캠퍼스 내외의 다양한 환경에서 수집되었다. 데이터셋은 총 13개의 시퀀스로 구성되며, 총 주행 거리는 8.4km, 총 기록 시간은 2.4시간에 달한다.25 여기에는 잘 구조화된 건물 내부, 복잡한 나무와 수풀이 우거진 공원, 넓은 야외 공간, 아침, 점심, 저녁 등 다양한 시간대와 조명 조건이 포함되어 있어 알고리즘의 전반적인 성능을 종합적으로 평가할 수 있다.

특히 주목할 만한 것은 의도적으로 센서 열화 상황을 연출한 시퀀스들이다. `degenerate_seq_00`, `degenerate_seq_01`, `degenerate_seq_02` 시퀀스는 장비를 일부러 벽이나 바닥으로 향하게 하여 LiDAR나 카메라(혹은 둘 다)의 정보가 부족해지는 상황을 재현한다.25 이러한 데이터는 R3LIVE가 내세우는 '강인성'이라는 특성을 정량적으로 검증하는 데 매우 중요한 역할을 한다.

이처럼 자신들의 알고리즘이 강점을 보이는 특정 시나리오를 포함하는 벤치마크 데이터셋을 직접 제작하고 공개하는 것은, 연구의 영향력을 확대하고 커뮤니티 내에서 표준적인 평가 기준으로 자리 잡게 하려는 전략적인 움직임으로 해석될 수 있다. 코드 8, 데이터셋 25, 심지어 하드웨어 설계도 8까지 모두 공개함으로써, 연구의 투명성을 높이고 다른 연구자들이 쉽게 접근하여 검증하고 확장할 수 있는 생태계를 구축하고 있다. 이는 R3LIVE가 학계와 산업계에 빠르게 확산되는 데 큰 기여를 했다.


R3LIVE는 강인성뿐만 아니라 높은 수준의 정확도와 실시간성을 동시에 달성한다. ICRA 2022에 발표된 R3LIVE 논문에 따르면, 자체 데이터셋 중 하나에서 1.5km를 주행한 후 최종 위치 오차는 0.16m, 최종 회전 오차는 3.9도에 불과한 매우 높은 정확도를 보였다고 보고했다.4 이는 전체 주행 거리에 대한 오차율이 약 0.01% 수준임을 의미한다.

후속 연구인 R3LIVE++는 정확도를 한층 더 끌어올렸다. R3LIVE++ 논문에서는 공개 벤치마크 데이터셋인 NCLT-dataset을 이용해 다른 최신(State-of-the-art, SOTA) SLAM 시스템들과 성능을 비교했다. 그 결과, LVI-SAM, LIO-SAM, FAST-LIO2와 같은 강력한 경쟁자들을 제치고 R3LIVE++가 가장 높은 전반적인 정확도를 달성했다고 주장한다.1 이는 Radiance 맵과 정교한 광도 보정 모델을 도입한 것이 단순한 맵 품질 향상을 넘어, 위치 추정의 정확도에도 직접적으로 기여했음을 보여준다.

실시간 성능은 R3LIVE의 또 다른 핵심적인 장점이다. 시스템의 모든 구성 요소(LIO, VIO, 맵핑)는 실시간으로 동작하도록 설계되었다.4 이는 계산 효율적인 ESIKF 프레임워크와 ikd-Tree와 같은 최적화된 자료 구조, 그리고 FAST-LIO의 혁신적인 칼만 게인 계산법 덕분이다. 이러한 실시간성은 R3LIVE가 이론적인 연구에 그치지 않고, 자율주행 드론이나 로봇과 같은 실제 하드웨어에 탑재되어 즉시 활용될 수 있음을 의미한다.


R3LIVE의 기술적 위치와 장단점을 명확히 이해하기 위해서는, 동시대의 다른 주요 SLAM 시스템들과의 비교 분석이 필수적이다. 특히 LiDAR-Visual-Inertial 융합 분야에서 널리 알려진 LIO-SAM, VINS-Fusion과 비교하면 R3LIVE의 설계 철학과 아키텍처적 특징이 더욱 뚜렷하게 드러난다.


| **항목 (Criteria)** | **R3LIVE / R3LIVE++**                                        | **LIO-SAM**                                                  | **VINS-Fusion**                                    |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------- |
| **핵심 아키텍처**   | 두 개의 병렬 서브시스템 (LIO, VIO)을 ESIKF로 결합 8          | LIO와 VIO가 요인 그래프(Factor Graph)에 정보를 제공하는 통합 구조 19 | VIO가 중심이며, 다른 센서(LiDAR)가 보조하는 구조   |
| **백엔드 최적화**   | 오차 상태 반복 칼만 필터 (ESIKF) 23                          | 요인 그래프 최적화 (Factor Graph Optimization) 19            | 슬라이딩 윈도우 기반 비선형 최적화                 |
| **융합 전략**       | 긴밀 결합 (Tightly-coupled) 필터 방식                        | 요인 그래프를 통한 긴밀 결합 최적화 방식                     | 긴밀 결합 최적화 방식                              |
| **시각 정보 역할**  | 맵 텍스처링 및 광도 오차(Photometric Error)를 통한 상태 보정 4 | 특징점 기반 재투영 오차(Reprojection Error)를 요인으로 추가, 시각 기반 루프 클로저 27 | 주된 상태 추정을 위한 특징점 기반 재투영 오차      |
| **LiDAR 정보 역할** | 주된 상태 추정 및 기하학적 맵 구축 4                         | 주된 상태 추정 및 맵 구축, 요인 그래프의 핵심 요인           | (LiDAR 버전) 깊이 정보 제공 및 맵 구축             |
| **강인성 전략**     | LIO와 VIO의 상호 보완. 직접법 VIO로 텍스처 부족 환경 극복 4  | 요인 그래프 내 다중 센서 제약조건 활용, 루프 클로저          | IMU와의 긴밀 결합, 루프 클로저                     |
| **핵심 장점**       | 센서 열화 환경에서의 극강의 강인성, 실시간 고품질 컬러/Radiance 맵 생성 | 전역적 일관성(Global Consistency), 루프 클로저를 통한 장기적 드리프트 보정 | 경량화, 높은 정확도의 VIO 성능, 폭넓은 플랫폼 지원 |


R3LIVE와 LIO-SAM은 LiDAR-Visual-Inertial 융합 SLAM 분야에서 가장 대표적인 두 가지 접근법, 즉 필터 기반 방식과 최적화 기반 방식을 각각 상징한다.

**R3LIVE**는 **필터 기반(filter-based)** 접근법의 정점에 있다. 시스템의 핵심에는 ESIKF가 자리 잡고 있으며, 새로운 센서 측정값이 들어올 때마다 상태 벡터와 공분산을 점진적으로 업데이트한다.27 이 방식의 가장 큰 장점은 

**계산 효율성**과 **실시간성**이다. 과거의 모든 측정값을 저장하고 최적화할 필요 없이, 현재 상태와 공분산이라는 요약된 정보만을 유지하기 때문에 계산 비용이 상대적으로 낮고 일정하게 유지된다. 이는 제한된 연산 자원을 가진 로봇 플랫폼에 매우 유리하다. 하지만 단점도 명확하다. 필터는 과거 정보를 '요약'하는 과정에서 일부 정보를 잃게 되므로, 시간이 지남에 따라 누적된 오차를 완벽하게 보정하기 어렵다. 특히, 출발점으로 다시 돌아왔을 때(루프 클로저) 발생하는 불일치를 소급하여 전체 궤적을 수정하는 데에는 한계가 있어, 전역적인 일관성(global consistency) 측면에서는 최적화 기반 방식에 비해 불리할 수 있다.

반면, **LIO-SAM**은 **최적화 기반(optimization-based)**, 특히 **요인 그래프(Factor Graph) 최적화**를 사용하는 대표적인 시스템이다.19 이 방식은 일정 시간 동안의 주요 프레임(keyframes)들과 그 사이의 센서 측정값(IMU 전적분, LiDAR 오도메트리, 시각 오도메트리 등)을 모두 '요인(factor)'으로 하는 거대한 그래프를 구축한다. 그리고 이 그래프 전체의 오차를 최소화하도록 모든 키프레임의 자세를 동시에 최적화한다. 이 방식의 최대 장점은 

**전역적 일관성**이다. 루프 클로저가 발생하면, 새로운 제약 조건을 요인 그래프에 추가하여 과거의 모든 궤적을 일관성 있게 수정할 수 있어 장기적인 드리프트 문제를 효과적으로 해결한다. 하지만 이는 상당한 계산 비용을 수반한다. 키프레임이 늘어날수록 그래프의 크기가 커져 최적화에 더 많은 시간과 메모리가 필요하다. LIO-SAM은 슬라이딩 윈도우(sliding window) 기법과 같은 방법으로 이 문제를 완화하지만, 본질적으로 필터 기반 방식보다 계산 부하가 크다.

R3LIVE와 LVI-SAM(LIO-SAM의 시각 기능 강화 버전)은 IMU 전적분, VIO, LIO의 추정치를 융합한다는 점에서 개념적으로 유사하지만, 그 융합이 일어나는 장소가 다르다. R3LIVE는 ESIKF라는 단일 **필터** 내에서 실시간으로 융합이 이루어지는 반면, LVI-SAM은 **포즈 그래프**라는 백엔드에서 융합이 이루어진다.28 이는 실시간 반응성과 전역 정확도 사이의 근본적인 트레이드오프를 보여주는 대표적인 사례다.


R3LIVE와 VINS-Fusion은 시각 정보를 활용하는 철학에서 근본적인 차이를 보인다.

**VINS-Fusion**은 그 이름에서 알 수 있듯이 **VIO(Visual-Inertial Odometry)가 시스템의 중심**이다. 시각 정보, 즉 이미지에서 추출한 특징점(feature points)과 IMU 데이터를 긴밀하게 결합하여 시스템의 자세를 추정하는 것이 핵심이다. 여기서 시각 정보는 위치 추정의 주된 역할을 담당한다. VINS-Fusion의 LiDAR 버전에서는 LiDAR 데이터가 특징점의 깊이(depth)를 더 정확하게 초기화해주거나, 추가적인 제약 조건을 제공하는 보조적인 역할을 수행한다. 즉, '시각'이 주연이고 'LiDAR'가 조연인 셈이다.

반면, **R3LIVE**에서는 **LIO(LiDAR-Inertial Odometry)가 시스템의 중심**이다. LiDAR와 IMU가 결합된 LIO가 시스템의 주된 위치 추정과 기하학적 맵 구축을 담당한다. 여기서 시각 정보의 역할은 두 가지로 요약된다. 첫째는 LIO가 만든 맵에 **색을 입히는(texturing)** 것이고, 둘째는 이 과정에서 발생하는 **광도 오차를 이용해 LIO가 추정한 자세를 미세하게 보정(refining)**하는 것이다.4 R3LIVE의 VIO는 시각 정보 자체만으로 독립적인 위치 추정을 시도하지 않는다. 대신, 이미 LIO에 의해 매우 정확하게 추정된 기하 구조 위에서 동작하며, 이를 통해 안정성과 강인성을 확보한다. 즉, 'LiDAR'가 주연이고 '시각'이 신뢰도 높은 조연인 셈이다.

이러한 차이는 각 시스템의 강점과 약점으로 이어진다. VINS-Fusion은 경량화되어 있고 VIO 자체의 성능이 매우 뛰어나 다양한 플랫폼에서 널리 사용된다. 하지만 텍스처가 부족한 환경에서는 특징점 추적 실패로 인해 성능이 저하될 수 있다. R3LIVE는 LiDAR를 중심으로 하기 때문에 기하학적으로 구조적인 환경에서 매우 안정적이며, 직접법 기반의 VIO 덕분에 텍스처가 부족한 환경에서도 시각 정보를 보조적으로 활용하여 강인성을 유지한다. 이는 각 시스템이 목표로 하는 주된 응용 시나리오와 설계 철학의 차이를 반영한다.


R3LIVE와 그 후속 연구인 R3LIVE++는 단순한 학술적 성과를 넘어, 다양한 실제 산업 및 응용 분야에 직접적인 영향을 미칠 수 있는 강력한 잠재력을 가지고 있다. 이 시스템들이 제공하는 실시간성, 강인성, 그리고 고품질 3D 맵핑 능력은 기존 기술의 한계를 뛰어넘는 새로운 가능성을 열어준다.


R3LIVE는 다재다능한(versatile) 시스템으로 설계되어, 여러 분야에서 즉시 활용될 수 있다.14

- **실시간 로봇 응용:** R3LIVE의 가장 직접적인 응용 분야는 자율 로봇의 항법 시스템이다. 자율주행차, 무인 항공기(드론), 자율 이동 로봇(AMR) 등은 주변 환경을 실시간으로 인식하고 자신의 위치를 정확하게 파악해야 한다. R3LIVE는 GPS가 수신되지 않는 실내나 도심 협곡과 같은 환경에서도 강인하고 정밀한 6-DOF(6-Degree-of-Freedom) 상태 추정을 제공한다. 또한, 실시간으로 생성되는 3D 맵은 로봇이 장애물을 회피하고 안전한 경로를 계획하는 데 필수적인 정보를 제공한다.3
- **측량 및 맵핑:** 전통적인 측량 및 맵핑 작업은 고가의 장비와 많은 시간, 인력을 필요로 했다. R3LIVE는 상대적으로 저렴한 센서 조합을 사용하여, 사람이 휴대하거나 로봇에 장착하여 이동하는 것만으로도 대규모 환경의 정밀하고 밀도 높은 3D RGB 컬러 맵을 신속하게 구축할 수 있다.11 이는 건설 현장의 진행 상황 모니터링, 대규모 인프라(다리, 터널 등)의 정밀 안전 진단, 지리 정보 시스템(GIS) 데이터베이스 구축, 산림 자원 관리 등 다양한 분야에서 비용과 시간을 획기적으로 절감할 수 있는 솔루션이 될 수 있다.
- **고품질 3D 에셋 생성:** R3LIVE는 단순히 포인트 클라우드 맵을 생성하는 데 그치지 않고, 재구성된 맵을 후처리하여 3D 메쉬(mesh)로 변환하고 그 위에 고품질 텍스처를 입히는 오프라인 유틸리티를 함께 제공한다.9 이를 통해 현실 세계의 공간과 객체를 매우 사실적인 3D 모델로 손쉽게 변환할 수 있다. 이렇게 생성된 3D 에셋은 가상현실(VR) 및 증강현실(AR) 콘텐츠, 영화 및 광고 제작을 위한 특수효과(VFX), 고도로 현실적인 비디오 게임 환경, 자율주행 시뮬레이터 개발 등 디지털 콘텐츠 제작 산업 전반에 걸쳐 폭넓게 활용될 수 있다.


R3LIVE++가 도입한 Radiance Map 개념은 응용의 범위를 한 차원 더 높은 수준으로 확장시킨다.16 Radiance는 관찰 시점이나 조명과 무관한 표면의 고유한 물리적 속성이므로, 이를 재구성한다는 것은 단순한 3D 모델링을 넘어 현실 세계의 빛을 디지털로 복제하는 것을 의미한다.

- **고다이나믹레인지(HDR) 이미징:** 일반 카메라는 한 번에 담을 수 있는 밝기의 범위(dynamic range)가 제한적이다. R3LIVE++는 서로 다른 노출 시간으로 촬영된 여러 장의 이미지를 Radiance Map으로 통합하는 과정에서, 매우 밝은 부분과 매우 어두운 부분의 정보를 모두 보존할 수 있다. 따라서 재구성된 Radiance Map으로부터 어떤 시점에서든 HDR 이미지를 생성해낼 수 있다.16
- **가상 환경 탐사 및 재조명(Relighting):** 물리적으로 정확한 Radiance Map이 있다면, 가상 공간에서 새로운 시점으로 장면을 렌더링하는 것(novel view synthesis)이 가능하다. 더 나아가, 가상의 광원을 추가하거나 기존 광원의 위치와 색상을 변경하여 장면 전체에 빛이 어떻게 반사되고 그림자가 생기는지를 물리적으로 시뮬레이션하는 재조명(relighting) 작업도 가능해진다. 이는 건축 시뮬레이션, 인테리어 디자인, 가상 쇼룸, 몰입형 교육 콘텐츠 등에서 매우 높은 수준의 현실감과 상호작용을 제공할 수 있다.16


R3LIVE는 많은 발전을 이루었지만, 여전히 해결해야 할 과제와 향후 연구를 위한 방향성이 존재한다.

- **동적 객체 처리:** 대부분의 현존하는 SLAM 시스템과 마찬가지로, R3LIVE는 기본적으로 주변 환경이 정적(static)이라고 가정한다.15 하지만 실제 환경은 움직이는 사람, 차량 등 수많은 동적 객체로 가득 차 있다. 이러한 동적 객체들은 LiDAR 포인트 클라우드에 '잔상'을 남기거나 잘못된 기하학적 제약 조건을 만들어 위치 추정의 정확도를 심각하게 저해하는 요인이 된다.29 향후 연구는 딥러닝 기반의 시맨틱 분할(semantic segmentation) 기술을 융합하여 동적 객체를 실시간으로 탐지하고 맵핑 과정에서 제거하거나, 그 움직임을 별도로 추적하는 방향으로 나아가야 한다.30
- **장기적 맵 관리 및 유지보수:** R3LIVE는 주어진 시간 동안의 맵을 정밀하게 구축하는 데 초점을 맞춘다. 하지만 로봇이 같은 공간을 오랜 기간 동안 반복적으로 운용될 경우, 시간이 지남에 따라 변하는 환경(예: 가구 배치 변경, 계절에 따른 식생 변화, 조명 조건 변화)에 대응해야 한다. 또한, 맵 데이터가 무한정 커지는 것을 방지하고, 변화된 부분을 효율적으로 탐지하고 업데이트하는 장기적인 맵 관리(long-term map management) 전략이 필요하다.
- **의미론적(Semantic) SLAM으로의 확장:** 현재 R3LIVE가 생성하는 맵은 기하학적 구조와 색상 정보만을 담고 있다. 여기에 딥러닝 기술을 결합하여 맵의 각 요소가 무엇인지(예: '문', '의자', '도로')를 이해하는 의미론적 정보를 추가할 수 있다. 시맨틱 SLAM은 로봇이 단순히 공간을 '보는' 것을 넘어 '이해'하게 만들어, "문으로 가서 손잡이를 잡아라"와 같은 더 높은 수준의 임무 수행을 가능하게 할 것이다. 이는 R3LIVE의 풍부한 3D 컬러 맵을 기반으로 발전시킬 수 있는 유망한 연구 방향이다.


R3LIVE는 오픈 소스로 공개되어 있어 연구자나 개발자가 직접 설치하고 활용해볼 수 있다. 성공적인 설치와 운용을 위해서는 시스템 요구사항을 정확히 맞추고, 특히 센서 캘리브레이션에 주의를 기울여야 한다.


R3LIVE를 원활하게 구동하기 위한 하드웨어 및 소프트웨어 요구사항은 다음과 같다.

- **소프트웨어 요구사항** 8:

  - **운영체제:** Ubuntu 16.04, 18.04, 20.04
  - **ROS (Robot Operating System):** ROS Kinetic, Melodic, Noetic. 버전에 맞는 추가 패키지(cv_bridge, tf, message_filters 등) 설치가 필요하다.
  - **OpenCV:** 버전 3.3 이상이 필요하며, **매우 중요한 점**은 R3LIVE 코드를 컴파일할 때 사용한 OpenCV 버전과 ROS가 사용하는 시스템의 OpenCV 버전이 일치해야 한다는 것이다. 버전 불일치는 런타임 에러의 주된 원인이므로 각별한 주의가 필요하다.8
  - **기타 라이브러리:** CGAL (Computational Geometry Algorithms Library), PCL (Point Cloud Library) 등이 필요하다.

- **하드웨어 요구사항** 8:

  - **LiDAR:** R3LIVE는 특히 Livox 사의 LiDAR(예: Avia)에 최적화되어 개발되었다. Livox LiDAR의 비반복적인 스캔 패턴은 더 밀도 높은 맵을 구축하는 데 유리하다.12 하지만 소스 코드의 프론트엔드 부분(

    `LiDAR_front_end.cpp`)을 수정하면 Ouster와 같은 전통적인 회전형 LiDAR도 지원 가능하다.8

  - **카메라:** 일반적인 산업용 USB 또는 GigE 카메라를 사용할 수 있다.

  - **IMU:** 높은 주파수(최소 200Hz)의 6축 IMU가 필요하다. Livox LiDAR에 내장된 IMU를 사용하는 것이 시간 동기화 측면에서 편리하다.12

  - 개발팀은 이 센서들을 통합한 자체 제작 핸드헬드 장비의 3D 프린팅 가능한 기구 설계도와 회로도를 오픈 소스로 공개하여 사용자들이 동일한 하드웨어 구성을 재현할 수 있도록 지원하고 있다.8


R3LIVE의 설치 및 빌드 과정은 표준적인 ROS 패키지 빌드 절차를 따른다.8

1. **의존성 설치:** `apt-get`을 이용해 ROS, OpenCV, PCL, CGAL 등 필요한 모든 라이브러리를 설치한다.

2. **livox_ros_driver 설치:** Livox LiDAR를 사용하는 경우, 공식 드라이버를 설치해야 한다. R3LIVE 개발팀은 LiDAR, IMU, 카메라 데이터 간의 타임스탬프를 정밀하게 동기화하기 위해 수정한 버전의 드라이버를 별도로 제공하며, 자체 데이터 수집 시 이 버전을 사용할 것을 권장한다.8

3. **R3LIVE 소스 코드 다운로드:** `git clone` 명령어를 사용해 R3LIVE의 공식 GitHub 저장소에서 소스 코드를 `catkin_ws/src` 폴더로 다운로드한다.

   ```Bash
   cd ~/catkin_ws/src
   git clone https://github.com/hku-mars/r3live.git
   ```

4. **빌드:** `catkin_ws` 폴더로 이동하여 `catkin_make` 또는 `catkin build` 명령어로 전체 패키지를 컴파일한다.

   ```Bash
   cd ~/catkin_ws
   catkin_make
   ```

5. **환경 설정:** 빌드가 완료되면, 터미널에서 R3LIVE 노드를 실행할 수 있도록 `source devel/setup.bash` 명령어로 환경을 설정한다.


R3LIVE와 같은 긴밀 결합 다중 센서 융합 시스템에서 성능을 좌우하는 가장 결정적인 요소 중 하나는 바로 **센서 간의 외인성 파라미터(extrinsic parameter) 캘리브레이션**이다. 외인성 파라미터란 한 센서 좌표계를 다른 센서 좌표계로 변환하는 회전(Rotation) 및 이동(Translation) 관계를 의미한다. LiDAR-Camera, LiDAR-IMU, Camera-IMU 간의 이 상대적인 위치와 자세가 정확하지 않으면, 시스템은 데이터를 잘못 융합하여 심각한 오차를 유발하거나 아예 발산해버릴 수 있다.8

R3LIVE는 이 캘리브레이션의 중요성을 인지하고 있으며, 사용자들이 정밀한 캘리브레이션을 수행할 수 있도록 별도의 툴 사용을 권장한다. 특히 LiDAR와 카메라 간의 캘리브레이션을 위해, 같은 연구실에서 개발한 `livox_camera_calib`이라는 오픈 소스 툴을 추천한다.9 이 툴은 별도의 체커보드와 같은 타겟 없이, 자연 환경에서 수집한 데이터만으로 LiDAR와 카메라 간의 정밀한 외인성 파라미터를 추정할 수 있는 강력한 기능을 제공한다. 캘리브레이션을 통해 얻어진 변환 행렬 값들은 R3LIVE의 설정 파일(`r3live_config.yaml`)에 정확하게 입력되어야 시스템이 올바르게 동작한다.


R3LIVE를 성공적으로 빌드했다면, 개발팀이 제공하는 예제 데이터셋을 통해 쉽게 테스트해볼 수 있다.

1. **예제 데이터셋 다운로드:** `R3LIVE-Dataset` 저장소에 안내된 링크(Google Drive 또는 Baidu Netdisk)를 통해 rosbag 형식의 예제 파일을 다운로드한다.8

2. **예제 실행:** 새로운 터미널을 열고, R3LIVE 런치 파일을 실행한다.

   ```Bash
   roslaunch r3live r3live_bag.launch
   ```

   다른 터미널에서 다운로드한 rosbag 파일을 재생한다.

   ```Bash
   rosbag play YOUR_DOWNLOADED_BAGFILE.bag
   ```

   모든 것이 정상이라면, Rviz와 같은 시각화 툴에 실시간으로 생성되는 3D 컬러 포인트 클라우드 맵과 추정된 궤적을 확인할 수 있다.

- **맵 저장 및 후처리** 8:

  - 실행 중 R3LIVE의 제어판(Control panel) 창을 활성화하고 's' 키를 누르면 현재까지 구축된 맵이 ${HOME}/r3live_output` 폴더에 `.pcd` 파일 형식으로 저장된다.

  - 맵 저장이 완료된 후, 제공되는 유틸리티를 사용하여 포인트 클라우드를 3D 메쉬로 변환하고 텍스처를 입힐 수 있다.

    ```Bash
    roslaunch r3live r3live_reconstruct_mesh.launch
    ```

  - 생성된 포인트 클라우드(`rgb_pt.pcd`)는 `pcl_viewer`로, 텍스처가 입혀진 메쉬(`textured_mesh.ply`)는 `meshlab`과 같은 툴로 시각화하여 결과를 확인할 수 있다.

자체 센서 장비를 사용하여 R3LIVE를 구동하기 위해서는, 위에서 언급한 정밀한 센서 캘리브레이션을 수행하고, 모든 센서 데이터(LiDAR, IMU, Camera)가 동일한 시간 기반(예: ROS time)으로 동기화되어 rosbag 파일로 기록되어야 한다. 이 과정을 거치면 자체 데이터를 이용한 R3LIVE의 강력한 맵핑 성능을 직접 체험할 수 있다.


1. R-LIVE++: A Robust, Real-Time, Radiance Reconstruction Package With a Tightly-Coupled LiDAR-Inertial-Visual State Estimator, accessed July 23, 2025, https://www.computer.org/csdl/journal/tp/2024/12/10669796/206qdCn3c88
2. LVI-Fusion: A Robust Lidar-Visual-Inertial SLAM Scheme - MDPI, accessed July 23, 2025, https://www.mdpi.com/2072-4292/16/9/1524
3. 3D LiDAR SLAM: A survey - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/profile/Pengcheng-Shi-12/publication/381290917_3D_LiDAR_SLAM_A_survey/links/666b0619b769e769193363c4/3D-LiDAR-SLAM-A-survey.pdf
4. A Robust, Real-time, RGB-colored, LiDAR-Inertial-Visual tightly-coupled - Jiarong Lin, accessed July 23, 2025, https://jiaronglin.com/uploads/r3live.pdf
5. R 3 LIVE: A Robust, Real-time, RGB-colored, LiDAR-Inertial-Visual tightly-coupled state Estimation and mapping package | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/361955988_R_3_LIVE_A_Robust_Real-time_RGB-colored_LiDAR-Inertial-Visual_tightly-coupled_state_Estimation_and_mapping_package
6. A Compact Handheld Sensor Package with Sensor Fusion for Comprehensive and Robust 3D Mapping - PMC - PubMed Central, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11055049/
7. LIV-GaussMap: LiDAR-Inertial-Visual Fusion for Real-time 3D Radiance Field Map Rendering - arXiv, accessed July 23, 2025, https://arxiv.org/html/2401.14857v2
8. hku-mars/r3live: A Robust, Real-time, RGB-colored, LiDAR-Inertial-Visual tightly-coupled state Estimation and mapping package - GitHub, accessed July 23, 2025, https://github.com/hku-mars/r3live
9. R$^3$LIVE: A Robust, Real-time, RGB-colored, LiDAR-Inertial-Visual tightly-coupled state Estimation and mapping package | Jiarong Lin, accessed July 23, 2025, https://jiaronglin.com/publication/paper_r3live/
10. R Live: A Robust, Real-Time, Rgb-Colored, Lidar-Inertial-Visual Tightly-Coupled State Estimation and Mapping Package | PDF - Scribd, accessed July 23, 2025, https://www.scribd.com/document/567434306/R3live-shuai
11. R3LIVE: A Robust, Real-time, RGB-colored, LiDAR-Inertial-Visual tightly-coupled state Estimation and mapping package | Papers With Code, accessed July 23, 2025, https://paperswithcode.com/paper/r3live-a-robust-real-time-rgb-colored-lidar
12. R3LIVE Real-time Robust Tightly Coupled System by MaRS Laboratory, HKU - Livox, accessed July 23, 2025, https://www.livoxtech.com/showcase/livox-r3live
13. R-LVIO: Resilient LiDAR-Visual-Inertial Odometry for UAVs in GNSS-denied Environment, accessed July 23, 2025, https://www.mdpi.com/2504-446X/8/9/487
14. R3LIVE: A Robust, Real-time, RGB-colored, LiDAR-Inertial-Visual tightly-coupled state Estimation and mapping package - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/354649365_R3LIVE_A_Robust_Real-time_RGB-colored_LiDAR-Inertial-Visual_tightly-coupled_state_Estimation_and_mapping_package
15. R 3 LIVE++: A Robust, Real-time, Radiance Reconstruction Package with a Tightly-coupled LiDAR-Inertial-Visual State Estimator | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/383879863_R_3_LIVE_A_Robust_Real-time_Radiance_Reconstruction_Package_with_a_Tightly-coupled_LiDAR-Inertial-Visual_State_Estimator
16. [2209.03666] R$^3$LIVE++: A Robust, Real-time, Radiance reconstruction package with a tightly-coupled LiDAR-Inertial-Visual state Estimator - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2209.03666
17. R$^3$LIVE++: A Robust, Real-time, Radiance reconstruction package with a tightly-coupled LiDAR-Inertial-Visual state Estimator | Papers With Code, accessed July 23, 2025, https://paperswithcode.com/paper/r-3-live-a-robust-real-time-radiance
18. R 3 ^3 LIVE++: A Robust, Real-time, Radiance reconstruction package with a tightly-coupled LiDAR-Inertial-Visual state Estimator - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/363403813_R3LIVE_A_Robust_Real-time_Radiance_reconstruction_package_with_a_tightly-coupled_LiDAR-Inertial-Visual_state_Estimator
19. LIVE++: A Robust, Real-time, Radiance reconstruction package with a tightly-coupled LiDAR-Inertial - Jiarong Lin, accessed July 23, 2025, https://jiaronglin.com/uploads/r3live++.pdf
20. R 3LIVE++: A Robust, Real-Time, Radiance Reconstruction Package With a Tightly-Coupled LiDAR-Inertial-Visual State Estimator - PubMed, accessed July 23, 2025, https://pubmed.ncbi.nlm.nih.gov/39250361/
21. A Fast, Robust LiDAR-Inertial Odometry Package by Tightly-Coupled Iterated Kalman Filter, accessed July 23, 2025, https://app.scinito.ai/article/W3133648073
22. FAST-LIO: A Fast, Robust LiDAR-inertial Odometry Package by Tightly-Coupled Iterated Kalman Filter | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/349909094_FAST-LIO_A_Fast_Robust_LiDAR-inertial_Odometry_Package_by_Tightly-Coupled_Iterated_Kalman_Filter
23. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - GitHub, accessed July 23, 2025, https://raw.githubusercontent.com/hku-mars/FAST_LIO/main/doc/Fast_LIO_2.pdf
24. [2109.07982] R3LIVE: A Robust, Real-time, RGB-colored, LiDAR-Inertial-Visual tightly-coupled state Estimation and mapping package - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2109.07982
25. ziv-lin/r3live_dataset - GitHub, accessed July 23, 2025, https://github.com/ziv-lin/r3live_dataset
26. R$^3$LIVE: A Robust, Real-time, RGB-colored, LiDAR-Inertial-Visual tightly-coupled state Estimation and mapping package | Jiarong Lin, accessed July 23, 2025, https://jiaronglin.com/project/proj_r3live/
27. Sensor Fusion, accessed July 23, 2025, https://gggliuye.github.io/Study/PaperRead/sensor_fusion/
28. RELEAD: Resilient Localization with Enhanced LiDAR Odometry in Adverse Environments, accessed July 23, 2025, https://arxiv.org/html/2402.18934v1
29. A Review of Dynamic Object Filtering in SLAM Based on 3D LiDAR - MDPI, accessed July 23, 2025, https://www.mdpi.com/1424-8220/24/2/645
30. Multi-sensor Fusion for 3D Reconstruction of Urban Pipeline Interior Using the Enhanced R 3 LIVE Algorithm - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/389666523_Multi-sensor_Fusion_for_3D_Reconstruction_of_Urban_Pipeline_Interior_Using_the_Enhanced_R_3_LIVE_Algorithm
31. SLAM-Rep/r3live - Gitee, accessed July 23, 2025, https://gitee.com/slam_-rep/r3live?skip_mobile=true
32. h-wata/r3live_tools: Preparation for r3live - GitHub, accessed July 23, 2025, https://github.com/h-wata/r3live_tools


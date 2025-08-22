---
layout: page
title: AF-RLIO 기술 심층 분석 보고서
permalink: /slam/AF-RLIO/AF-RLIO
---



AF-RLIO의 전체 명칭은 **"Adaptive Fusion of Radar-LiDAR-Inertial Information for Robust Odometry in Challenging Environments"**이다.1 이 명칭은 시스템의 핵심 철학을 명확하게 보여준다. AF-RLIO의 근본적인 목표는 자율 로봇이나 차량이 기존의 센서 시스템으로는 안정적인 운용이 어려운 환경, 즉 '인식적으로 열악한(perceptually-degraded)' 환경에서 강건하고(robust) 신뢰성 있는(reliable) 주행 거리 측정(odometry) 및 위치 추정(localization)을 달성하는 것이다.2 이러한 열악한 환경은 연기가 자욱한 재난 현장, 기하학적 특징이 부족한 긴 터널, 악천후(안개, 폭우, 폭설) 등을 포함하며, 이러한 조건에서 정확한 위치 정보는 자율 시스템의 안정성과 안전을 보장하는 데 필수적이다.1

이 목표를 달성하기 위해 AF-RLIO는 단일 센서의 한계에 의존하는 대신, 4D 밀리미터파 레이더(4D millimeter-wave radar), 라이다(LiDAR), 관성 측정 장치(IMU), 그리고 GPS와 같은 다종의 센서를 통합한다. 핵심 전략은 각 센서가 특정 환경에서 나타내는 상호 보완적인 강점을 최대한 활용하는 '적응형 융합(adaptive fusion)' 접근법을 채택하는 것이다.1


사용자 질의의 핵심 키워드인 'AF-RLIO'에서 'AF'는 공식적으로 발표된 논문 제목과 저자의 프로젝트 설명에 따르면 명백히 **'Adaptive Fusion'**을 의미한다.1 이는 시스템이 환경 변화에 따라 동적으로 센서 활용 전략을 바꾸는 핵심 기능을 지칭한다.

하지만 초기 연구 버전의 URL이나 일부 자료에서는 'Anchor-Free'라는 표현이 관찰되었다.2 이는 단순한 표기 오류일 수도 있으나, 시스템의 아키텍처적 특성을 암시하는 중요한 단서일 가능성이 있다. 로보틱스 SLAM(Simultaneous Localization and Mapping) 분야에서 '앵커(Anchor)'는 종종 키프레임(keyframe)이나 사전 정의된 랜드마크와 같은 고정된 참조점을 의미한다.4 따라서 'Anchor-Free'는 이러한 고정된 참조점에 얽매이지 않는, 보다 유연한 데이터 처리 방식을 채택했음을 시사할 수 있다. 이 개념에 대해서는 본 보고서의 3장에서 심층적으로 분석할 것이다.


기존의 최첨단(State-of-the-Art, SOTA) 라이다-관성 주행 거리 측정(LIO) 시스템들은 일반적인 환경에서는 매우 높은 정확도를 제공한다.5 그러나 특정 조건에서는 치명적인 한계를 드러낸다. 라이다 기반 융합 알고리즘은 연기, 안개, 먼지와 같은 부유 입자에 의해 레이저 빔이 산란되거나 흡수되어 스캔 데이터의 품질이 심각하게 저하된다.2 이는 포인트 클라우드 정합(point cloud registration) 과정에 근본적인 오류를 야기하며, 라이다와 IMU에만 의존하는 시스템의 정확도를 급격히 떨어뜨린다.2

또한, 터널이나 고층 빌딩이 밀집한 도심 협곡과 같은 환경에서는 GPS 신호가 차단되거나 다중 경로 오차로 인해 심각한 이상치(outlier)를 포함하게 된다.2 이러한 부정확한 GPS 정보는 전역 최적화 과정에서 오히려 전체 궤적을 왜곡시켜 시스템 실패로 이어질 수 있다.

이러한 배경에서 AF-RLIO의 필요성이 대두된다. AF-RLIO는 단순히 기존 LIO 시스템의 정확도를 점진적으로 개선하는 것을 넘어, 기존 시스템이 완전히 실패하는 '치명적 실패 모드(catastrophic failure mode)'에 대한 근본적인 해결책을 제시한다. 그 논리적 근거는 다음과 같다. 첫째, SOTA LIO 시스템(예: LIO-SAM, FAST-LIO2)은 정상 환경에서는 이미 높은 성능을 달성했다.5 둘째, 연기나 안개 같은 악천후 상황은 알고리즘의 문제가 아니라 라이다 센서 자체의 물리적 한계로 인해 데이터 획득이 불가능해지는 상황이다. 이는 '성능 저하'를 넘어 '데이터 손실' 및 '시스템 붕괴'로 이어진다.2 셋째, 밀리미터파 레이더는 파장이 길어 이러한 입자들을 투과할 수 있으므로 악천후 환경에 본질적으로 강건하다.2 따라서 AF-RLIO의 다중 센서 '적응형' 융합 전략은 단순히 정확도를 소폭 향상시키는 것이 아니라, 기존 시스템이 작동을 멈추는 시나리오에서 시스템의 생존성을 보장하고 임무를 지속하게 만드는 핵심적인 패러다임 전환이다. 이는 자율주행, 재난 구조, 광산 탐사 등 실제 필드 로보틱스 응용을 위한 필수 불가결한 기술적 진보라 할 수 있다.8


AF-RLIO는 계층적 구조를 가지며, 크게 세 가지 핵심 모듈로 구성된다: **전처리(Pre-processing)**, **동적 인지 다중모드 주행 거리계(Dynamic-aware Multimodal Odometry)**, 그리고 **적응형 요인 그래프 최적화(Adaptive Factor Graph Optimization)**.2 이 구조는 실시간 처리를 담당하는 프론트엔드와 전역 일관성을 확보하는 백엔드로 나뉘는 현대 SLAM 시스템의 표준적인 설계를 따른다.


전처리 모듈의 주된 역할은 후속 처리 단계의 부담을 줄이고 입력 데이터의 품질을 높이는 것이다. 이를 위해 레이더의 고유한 장점을 적극적으로 활용한다.

첫째, 4D 밀리미터파 레이더는 거리, 방위각, 고도각뿐만 아니라 객체의 상대 속도를 직접 측정하는 도플러(Doppler) 정보를 제공한다.2 AF-RLIO는 이 도플러 정보를 이용해 포인트 클라우드에서 동적 객체(움직이는 차량, 보행자 등)와 정적 배경을 분리한다. 분리된 동적 객체 정보는 라이다 포인트 클라우드에서 해당 영역을 필터링하는 데 사용된다.2 대부분의 SLAM 알고리즘은 정적 환경을 가정하기 때문에, 동적 객체는 위치 추정의 정확도를 저해하는 주요 노이즈 요인이다. 이를 사전에 제거함으로써 시스템의 강건성을 크게 향상시킬 수 있다.7

둘째, 이 모듈은 라이다 데이터의 품질을 실시간으로 분석하여 현재 환경이 라이다 센서에 적합한지, 아니면 성능이 저하된(degraded) 상태인지를 판단한다.2 이 판단은 다음 단계인 프론트엔드 오도메트리에서 어떤 센서 조합을 사용할지를 결정하는 중요한 트리거(trigger) 역할을 한다.


프론트엔드 모듈은 고주파의 실시간 포즈 추정을 담당하며, AF-RLIO의 핵심인 **'적응형 스위칭(adaptive switching)'** 기능이 구현되는 곳이다. 환경 조건에 따라 두 가지 주요 작동 모드 사이를 전환한다.

- **LiDAR-Inertial Odometry (LIO) 모드:** 주변 환경이 양호하여 라이다 데이터가 신뢰할 수 있다고 판단될 때 활성화된다. 이 모드에서는 전처리 모듈을 거쳐 동적 객체가 제거된 라이다 포인트 클라우드를 IMU 측정값과 **강결합(tightly-coupled)**하여 로봇의 포즈를 정밀하게 추정한다.2 이 강결합 과정은 **반복 오차 상태 칼만 필터(Iterative Error State Kalman Filter, IESKF)**를 기반으로 수행된다.2 IESKF는 비선형성이 큰 시스템의 상태를 효과적으로 추정하는 기법으로, FAST-LIO2와 같은 고성능 LIO 시스템에서 그 효율성과 정확성이 입증된 바 있다.2
- **Radar-Inertial Odometry (RIO) 모드:** 연기, 안개, 폭설 등의 악천후로 인해 라이다 데이터의 품질이 심각하게 저하되면, 시스템은 자동으로 RIO 모드로 전환한다. 이 모드에서는 라이다 대신 레이더 포인트 클라우드를 IMU 측정값과 강결합하여 오도메트리 추정을 중단 없이 이어간다.2 이는 라이다 센서의 일시적 또는 지속적 실패 상황에서도 시스템이 완전히 발산(divergence)하는 것을 방지하고, 최소한의 위치 추적 능력을 유지하게 해주는 핵심적인 안전장치다.

환경이 다시 양호해져 라이다 데이터의 신뢰성이 회복되면, 시스템은 다시 LIO 모드로 복귀하여 고정밀 위치 추정을 재개한다.2


백엔드 모듈은 프론트엔드에서 생성된 고주파의 오도메트리 추정치와 저주파의 전역 측정값(GPS 등)을 통합하여, 전체 궤적에 걸쳐 누적된 오차(drift)를 최소화하고 전역적인 일관성을 확보하는 역할을 한다.2 이 최적화 문제는 **요인 그래프(Factor Graph)**라는 수학적 도구를 사용하여 모델링된다.2 요인 그래프에서 각 노드(node)는 특정 시점의 로봇 상태(포즈, 속도, 바이어스 등)를 나타내고, 노드들을 연결하는 요인(factor)은 센서 측정값에 의해 부과되는 상태 간의 제약 조건을 의미한다.8

AF-RLIO의 백엔드는 단순한 후처리 과정이 아니라, 프론트엔드와 마찬가지로 또 다른 '적응형' 메커니즘을 내장하고 있다. 그 이유는 다음과 같다. 첫째, GPS는 LIO나 RIO에서 발생하는 누적 오차를 보정하는 데 매우 효과적인 전역 위치 정보를 제공한다.8 둘째, 그러나 앞서 언급했듯이 GPS는 터널이나 도심 환경에서 신호가 끊기거나 큰 오차를 포함하는 이상치를 발생시키는 본질적인 약점을 가지고 있다.2 셋째, AF-RLIO는 GPS 측정값을 맹목적으로 신뢰하지 않는다. 대신, 레이더로 추정한 속도 정보와 프론트엔드 오도메트리 결과를 바탕으로 현재 수신된 GPS 데이터의 신뢰도를 실시간으로 평가한다.2 넷째, 만약 특정 GPS 측정값이 이상치로 판단되면, 해당 요인이 최적화에 미치는 영향을 동적으로 줄인다. 이는 요인 그래프에서 해당 GPS 요인의 가중치(정보 행렬 값)를 낮추는 방식으로 구현된다.2 이처럼 백엔드 최적화 과정 자체에 강건함(robustness)을 부여하는 이 기능은 AF-RLIO의 중요한 혁신 중 하나다. 프론트엔드가 '외부 환경의 변화'에 적응한다면, 백엔드는 '측정 데이터 품질의 변화'에 적응하는 셈이다.


앞서 언급했듯이, AF-RLIO의 'AF'는 공식적으로 'Adaptive Fusion'을 의미하지만, 초기 버전에서 사용된 'Anchor-Free'라는 용어는 시스템의 설계 철학을 이해하는 데 중요한 실마리를 제공한다. 이 개념을 이해하기 위해 먼저 다른 분야에서의 의미를 살펴보고, 이를 LIO/SLAM의 맥락으로 확장하여 AF-RLIO와의 연관성을 분석한다.


컴퓨터 비전, 특히 객체 탐지(object detection) 분야에서 '앵커(anchor)'는 이미지 내에 객체가 존재할 만한 위치에 미리 정의된 다양한 크기와 종횡비의 참조 박스(anchor box)를 의미한다. '앵커 기반(anchor-based)' 방식은 이 수많은 앵커 박스들을 기준으로 객체의 위치와 크기를 미세 조정하는 방식으로 작동한다. 이 방법은 효과적이지만, 앵커 박스의 크기, 비율, 개수 등 수많은 하이퍼파라미터를 사전에 잘 설계해야 하는 부담이 있고, 객체의 형태가 매우 다양할 경우 비효율적일 수 있다.13

이에 대한 대안으로 등장한 '앵커 프리(Anchor-free)' 방식은 이러한 사전 정의된 박스 없이, 객체의 중심점(center point)이나 모서리점(corner point)과 같은 핵심적인 키포인트를 직접 예측하여 객체를 탐지한다. 이 접근법은 하이퍼파라미터 의존성을 줄이고, 다양한 형태의 객체에 대해 더 유연하고 강건한 성능을 보이는 것으로 평가받는다.13


LIO/SLAM의 맥락에서 '앵커'라는 용어는 명시적으로 정의되어 사용되지는 않지만, 유사한 개념으로 **키프레임(keyframe)**과 **랜드마크(landmark)**를 들 수 있다. 특히 키프레임 기반 SLAM은 전체 센서 데이터 스트림에서 정보량이 풍부하거나 움직임이 특정 임계값을 넘는 프레임만을 선별하여 '키프레임'으로 지정한다. 이후의 모든 최적화와 맵 구성은 이 선별된 키프레임들을 중심으로 이루어진다.5 LIO-SAM과 같은 최적화 기반의 많은 LIO 시스템들이 이 방식을 채택하고 있으며, 이는 전체 데이터를 처리하는 데 따르는 계산 부하를 극적으로 줄여 실시간성을 확보하는 효과적인 전략이다.8

그러나 키프레임 방식은 어떤 프레임을 선택할지에 대한 전략(heuristic)에 따라 성능이 좌우되며, 선택되지 않은 프레임에 포함된 유용한 정보가 손실될 수 있다는 단점이 있다.15 즉, 시스템의 성능이 '앵커' 역할을 하는 키프레임의 선택과 품질에 크게 의존하게 된다.


'Anchor-Free'라는 용어를 '키프레임에 의존하지 않는' 또는 '키프레임의 제약에서 자유로운' 방식으로 해석한다면, AF-RLIO의 아키텍처는 이와 깊은 관련이 있다. 이는 AF-RLIO가 채택한 것으로 보이는 **직접 방식(Direct Method)**과의 연관성을 통해 설명될 수 있다.

그 논리적 흐름은 다음과 같다. 첫째, 키프레임 없이 오도메트리를 수행하는 대표적인 방식은 '직접 방식'이다. 이 방식은 LOAM이나 LIO-SAM처럼 평면이나 엣지 같은 기하학적 특징점(feature)을 추출하는 과정을 생략한다.17 대신, 원본(raw) 포인트 클라우드 또는 균일하게 다운샘플링된 포인트를 직접 맵에 정합(registration)하여 포즈를 추정한다.10

둘째, FAST-LIO2는 특징점 추출 없이 원본 포인트를 직접 맵에 등록하는 대표적인 직접 방식 LIO 시스템이다. 이 시스템은 선택된 키프레임으로 맵을 구성하는 대신, ikd-Tree라는 동적 자료구조를 사용하여 모든 유효한 포인트로 맵을 지속적으로 갱신한다.5

셋째, AF-RLIO의 프론트엔드는 FAST-LIO2와 동일한 IESKF를 사용하며, 'scan-to-map' 매칭을 수행한다고 명시되어 있다.2 특히 논문 초록에서 "특징 추출 없이(without feature extraction)"라는 표현을 사용하여, 직접 방식을 채택했음을 강하게 시사한다.2

결론적으로, AF-RLIO가 전통적인 특징점 기반의 키프레임-슬라이딩 윈도우 방식(예: LIO-SAM)이 아니라, 들어오는 모든 포인트 정보를 활용하여 지속적으로 맵을 갱신하고 포즈를 추정하는 직접 방식을 채택했다면, 이는 '앵커(키프레임)에 얽매이지 않는다'는 의미에서 'Anchor-Free'로 불릴 기술적 타당성을 가진다. 비록 최종 논문명이 'Adaptive Fusion'으로 확정되었지만, 'Anchor-Free'는 개발 초기 단계에서 시스템의 핵심적인 데이터 처리 철학을 반영하는 이름이었을 가능성이 매우 높다.


AF-RLIO의 성능은 정교한 수학적 모델링에 기반한다. 시스템은 IMU를 중심으로 상태를 정의하고, 프론트엔드에서는 칼만 필터를, 백엔드에서는 요인 그래프 최적화를 통해 상태를 추정한다.


시스템은 IMU 중심(IMU-centric)으로 설계되었으므로, 추정하고자 하는 핵심 상태 벡터 $x$는 IMU의 운동학적 상태와 센서 내부 파라미터로 구성된다.2 일반적인 LIO 시스템의 상태 벡터 정의를 참고하면 20, AF-RLIO의 상태 벡터는 다음과 같이 표현될 수 있다:
$$
x =^T
$$
여기서 각 항목은 다음과 같다:

- ${}^W R_I \in SO(3)$: 월드(World) 좌표계에 대한 IMU 좌표계의 회전 행렬.
- ${}^W p_I \in \mathbb{R}^3$: 월드 좌표계에서 측정한 IMU의 위치 벡터.
- ${}^W v_I \in \mathbb{R}^3$: 월드 좌표계에서 측정한 IMU의 속도 벡터.
- $b_a \in \mathbb{R}^3$: 가속도계의 바이어스(bias).
- $b_g \in \mathbb{R}^3$: 자이로스코프의 바이어스.


프론트엔드는 IESKF를 사용하여 상태를 추정한다.2 IESKF는 참 상태(true state) 

$x$를 공칭 상태(nominal state) $\hat{x}$와 작은 오차 상태(error state) $\delta x$의 합으로 모델링한다 ($x = \hat{x} \boxplus \delta x$). 필터는 직접적으로 상태를 추정하는 대신, 이 오차 상태 $\delta x$의 평균(0으로 가정)과 공분산을 추정한다.20

1. 예측 (Prediction) 단계:

이전 LiDAR 스캔 시점 $k-1$부터 현재 시점 $k$까지 수집된 고주파의 IMU 측정값($u_i =^T$)을 순차적으로 적분하여 공칭 상태 $\hat{x}_k$와 오차 공분산 $P_k$를 전파(propagate)한다.20
$$
\hat{x}_{i+1} = f(\hat{x}_i, u_i, 0)
\\
P_{i+1} = F_i P_i F_i^T + G_i Q G_i^T
$$
여기서 $f$는 IMU 운동학 모델, $F_i$와 $G_i$는 각각 상태 전파와 노이즈에 대한 자코비안 행렬, $Q$는 IMU 측정 노이즈의 공분산 행렬이다.

2. 갱신 (Update) 단계:

새로운 LiDAR(또는 Radar) 스캔이 들어오면, 갱신 단계가 수행된다. 각 포인트 $p_j$를 현재 추정된 포즈 $\hat{x}_k$를 이용해 월드 좌표계로 변환하고, 맵에서 가장 가까운 평면을 찾는다. 이때 포인트와 평면 사이의 수직 거리를 잔차(residual) $r_j$로 정의한다.

IESKF는 이 잔차들의 제곱 합을 최소화하는 오차 상태 $\delta x$를 찾기 위해 반복적으로 최적화를 수행한다. 비용 함수는 다음과 같이 구성된다 21:
$$
\min_{\delta x} \sum_j ||r_j(\hat{x}_k \boxplus \delta x)||^2_{\Sigma_r} + ||\delta x||^2_{P_k^{-1}}
$$
첫 번째 항은 측정 모델(LiDAR/Radar)에 대한 오차를, 두 번째 항은 예측 모델(IMU)로부터의 편차를 나타낸다. 이 최적화 문제는 여러 번의 반복을 통해 선형화되어 풀리며, 최종적으로 칼만 게인 $K$를 계산하여 상태 $\hat{x}_k$와 공분산 $P_k$를 갱신한다. FAST-LIO 계열의 핵심은 이 칼만 게인 계산 과정을 수학적 트릭(Sherman-Morrison-Woodbury formula)을 통해 매우 효율적으로 수행하는 데 있으며, AF-RLIO 또한 유사한 접근법을 취할 것으로 강력히 추정된다.5


백엔드는 슬라이딩 윈도우 또는 전체 궤적에 걸쳐 상태 변수 집합 $\mathcal{X} = \{x_1, x_2,..., x_n\}$에 대한 최대 사후 확률(Maximum a Posteriori, MAP) 추정 문제를 푼다.11 이는 베이즈 정리에 의해 비선형 최소 제곱 문제로 변환될 수 있다.23

전체 비용 함수 $J(\mathcal{X})$는 시스템에 존재하는 모든 측정 정보(요인)로부터 발생하는 오차(잔차)의 마할라노비스 거리(Mahalanobis distance)의 제곱 합으로 표현된다.25
$$
J(\mathcal{X}) = \sum ||r_{prior}||^2_{\Sigma_{prior}} + \sum_{i} ||r_{imu}(x_i, x_{i+1})||^2_{\Sigma_{imu}} + \sum_{k} ||r_{lio/rio}(x_k)||^2_{\Sigma_{lio/rio}} + \sum_{k} ||r_{gps}(x_k)||^2_{\Sigma_{gps}}
$$
각 요인(factor)의 의미는 다음과 같다:

- **사전 요인 (Prior Factor):** 첫 상태 $x_0$에 대한 초기 추정치와 같은 사전 정보를 제공하는 요인이다.
- **IMU 예비적분 요인 (IMU Preintegration Factor):** 연속된 두 포즈 $x_i$와 $x_{i+1}$ 사이의 상대적인 움직임을 IMU 측정값을 통해 제약하는 요인이다. LIO-SAM에서 핵심적으로 사용되는 강력한 제약 조건이다.8
- **LIO/RIO 요인 (LIO/RIO Factor):** 프론트엔드 오도메트리에서 계산된 각 포즈 $x_k$에 대한 측정값으로, 일종의 절대 포즈 제약 역할을 한다.
- **GPS 요인 (GPS Factor):** 특정 포즈 $x_k$에 대한 절대 위치 측정값을 제공한다. AF-RLIO의 핵심적인 혁신은 이 요인의 신뢰도를 나타내는 공분산 행렬 $\Sigma_{gps}$가 고정된 값이 아니라는 점이다. 시스템은 GPS 측정값이 이상치로 판단되면 이 행렬의 값을 조정하여(가중치를 낮춰) 해당 측정값이 전체 최적화에 미치는 영향을 동적으로 제어한다.2


AF-RLIO의 성능은 특히 기존 시스템이 어려움을 겪는 열악한 환경에서 두드러질 것으로 예상된다. 아직 공개된 벤치마크 결과는 없지만, 시스템 아키텍처와 설계 목표를 통해 주요 LIO 프레임워크와의 성능을 비교 분석할 수 있다.


AF-RLIO의 가장 큰 강점은 설계 목표 자체에서 명시하듯 연기, 터널과 같은 열악한 환경에서의 강건성이다.2

- **연기/안개/먼지 환경:** 이러한 환경에서는 라이다 센서의 레이저 빔이 공기 중 입자에 의해 산란되어 유효한 거리 측정이 거의 불가능해진다. 기존 LIO 시스템은 이 경우 실패할 수밖에 없다. 반면, AF-RLIO는 라이다 데이터의 저하를 감지하고 자동으로 RIO 모드로 전환한다. 밀리미터파 레이더는 이러한 입자를 투과하는 특성이 있으므로, RIO 모드는 안정적인 오도메트리 추정을 지속하여 시스템의 생존성을 보장한다.2
- **터널 환경:** 긴 터널은 기하학적 특징이 반복되거나 부족하여(feature-less) 라이다 오도메트리가 방향을 잃는 '축퇴(degeneracy)' 현상이 발생하기 쉽다.28 동시에 터널 내부에서는 GPS 신호 수신이 불가능하다. 이런 상황에서 레이더는 라이다보다 더 먼 거리를 감지할 수 있고, 도플러 속도 정보를 제공하므로 IMU와의 강결합을 통해 누적 오차(drift)를 효과적으로 억제하는 데 중요한 역할을 할 수 있다.2


AF-RLIO는 LIO-SAM과 FAST-LIO2 같은 선도적인 LIO 시스템들의 장점을 결합하고 한계를 극복하려는 시도로 볼 수 있다. 각 시스템의 아키텍처를 비교하면 그 차이점이 명확해진다.

**Table 1: AF-RLIO vs. LIO-SAM vs. FAST-LIO2 아키텍처 비교**

| 항목 (Feature)  | AF-RLIO                                              | LIO-SAM 5                       | FAST-LIO2 5                               |
| --------------- | ---------------------------------------------------- | ------------------------------- | ----------------------------------------- |
| **핵심 방법론** | 하이브리드 (IESKF Front-end + Factor Graph Back-end) | 최적화 기반 (Factor Graph)      | 필터 기반 (Iterated EKF)                  |
| **센서 융합**   | 적응형 다중모드 (LiDAR/Radar/IMU/GPS)                | 강결합 (LiDAR/IMU/GPS)          | 강결합 (LiDAR/IMU)                        |
| **강건성 확보** | 센서 스위칭, 동적 객체 제거(Radar), GPS 이상치 처리  | 루프 클로저, GPS 팩터           | 특징점 불필요(Direct), 빠른 상태 업데이트 |
| **처리 방식**   | 직접 방식(Direct Method) 추정 (특징점 추출 없음)     | 특징점 기반 (Feature-based)     | 직접 방식 (Direct Method)                 |
| **키프레임**    | 의존성 낮음 (맵 직접 업데이트)                       | 핵심 요소 (Keyframe-based)      | 사용 안 함 (Map-centric)                  |
| **주요 장점**   | 열악한 환경에서의 강건성                             | 전역적 일관성, 다중 센서 확장성 | 계산 효율성, 높은 정확도                  |

이 비교를 통해 AF-RLIO가 FAST-LIO2의 효율적인 필터 기반 프론트엔드와 LIO-SAM의 유연한 최적화 기반 백엔드를 결합한 하이브리드 구조임을 알 수 있다. 여기에 레이더를 활용한 적응형 로직을 추가하여 두 시스템이 가지지 못한 환경 대응 능력을 확보한 것이 가장 큰 차별점이다.


AF-RLIO 논문은 2025년 ICRA 학회 제출 예정으로 3, 아직 KITTI와 같은 공개 벤치마크 순위에 등록된 공식 결과는 없다. 특히 KITTI 데이터셋은 IMU 데이터의 주파수가 낮아 LIO-SAM과 같은 고주파 IMU를 필요로 하는 시스템의 성능을 제대로 평가하기 어렵다는 문제가 지적된 바 있다.31

그럼에도 불구하고, 각 시스템의 아키텍처와 기존 연구 결과를 바탕으로 성능을 유추해 볼 수 있다. LIO-SAM과 FAST-LIO2는 여러 논문에서 비교 대상으로 사용되며, 일반적으로 KITTI Odometry 벤치마크에서 1% 내외의 매우 낮은 절대 궤적 오차(Absolute Trajectory Error, ATE)를 기록하는 것으로 알려져 있다.32

AF-RLIO는 FAST-LIO2와 유사한 프론트엔드와 LIO-SAM과 유사한 백엔드를 가졌기 때문에, **정상적인 환경**에서는 이들과 비슷하거나 백엔드 최적화로 인해 약간 더 계산 부하가 높은 대신 동등하거나 더 나은 정확도를 보일 것으로 예상된다.

그러나 AF-RLIO의 진정한 가치는 **도전적인 환경**에서 드러난다. 동적 객체가 많거나, GPS 이상치가 빈번하거나, 라이다 성능 저하가 발생하는 시나리오에서는 레이더 기반 동적 객체 제거 및 적응형 센서 스위칭 기능 덕분에 LIO-SAM과 FAST-LIO2를 압도하는 안정성과 정확도를 보일 것이라 강력하게 추정할 수 있다. 이는 AF-RLIO의 설계 목표와 정확히 일치하는 결과다.2

**Table 2: 공개 벤치마크 데이터셋 기반 성능 비교 (추정치 포함)**

| 알고리즘      | KITTI Odometry (ATE %) | UrbanLoco (ATE RMSE m)                     | 열악 환경 (정성 평가)     |
| ------------- | ---------------------- | ------------------------------------------ | ------------------------- |
| **AF-RLIO**   | N/A (직접 비교 어려움) | N/A                                        | **매우 우수** (설계 목표) |
| **LIO-SAM**   | ~1% 내외 (참고치) 33   | ID-LIO(개선판)가 0.67*LIO-SAM 5            | **취약** (LiDAR/GPS 의존) |
| **FAST-LIO2** | ~1% 내외 (참고치) 32   | Faster-LIO(개선판)가 FAST-LIO2보다 빠름 34 | **취약** (LiDAR 의존)     |


AF-RLIO는 의심할 여지 없이 강건한 LIO 기술 분야에서 중요한 진전을 이루었지만, 동시에 몇 가지 잠재적인 한계와 향후 연구 과제를 안고 있다.


- **시스템 복잡성 증가:** 4종류의 센서(LiDAR, Radar, IMU, GPS)를 융합하고, 적응형 스위칭 로직, IESKF 필터, 요인 그래프 최적화를 모두 포함하는 구조는 시스템의 복잡도를 필연적으로 높인다. 이는 수많은 파라미터를 튜닝하고 시스템을 디버깅하는 과정을 매우 어렵게 만들 수 있다.
- **레이더 데이터의 본질적 한계:** 4D 레이더가 악천후에 강건한 것은 사실이지만, 라이다에 비해 포인트 클라우드가 훨씬 희소(sparse)하고 노이즈가 많으며, 특히 높이(z축) 정보의 정확도가 떨어진다는 단점이 있다.2 따라서 RIO 모드로 장시간 운용될 경우, LIO 모드에 비해 누적 오차가 더 클 수밖에 없다. RIO는 어디까지나 LIO의 실패를 막는 보조 수단이며, 그 자체로 LIO 수준의 정밀도를 장시간 유지하기는 어렵다.
- **모드 전환의 안정성:** LIO 모드와 RIO 모드 간의 전환이 발생할 때, 상태 추정치에 불연속성이나 불안정성이 발생할 위험이 있다. 시스템이 이 전환을 얼마나 '부드럽게(smoothly)' 처리하고, 두 모드에서 추정된 맵 정보를 어떻게 일관성 있게 통합하는지가 전체 시스템 성능의 중요한 관건이 될 것이다.


AF-RLIO가 제시한 방향성을 바탕으로, 향후 강건한 LIO 기술은 다음과 같은 방향으로 발전할 것으로 전망된다.

- **연속 시간 궤적 최적화 (Continuous-Time Trajectory Optimization):** AF-RLIO를 포함한 대부분의 현재 LIO 시스템은 이산적인 시간(discrete-time) 스텝에서 상태를 추정한다. 반면, '연속 시간' 패러다임은 로봇의 궤적 자체를 B-스플라인(B-spline)이나 가우시안 프로세스(Gaussian Process)와 같은 연속 함수로 모델링하고, 이 함수의 파라미터를 직접 최적화한다.35 이 접근법은 IMU와 같이 고주파로 비동기적으로 들어오는 센서 데이터를 예비적분(pre-integration)과 같은 전처리 과정 없이 그 측정 시점에 맞춰 자연스럽게 융합할 수 있는 강력한 장점을 가진다.35 특히 로봇의 움직임이 매우 빠르고 역동적인 상황에서 더 정확한 모션 보간(motion interpolation)을 가능하게 하여, LIO의 성능과 강건성을 한 단계 더 끌어올릴 핵심 기술로 주목받고 있다.39
- **학습 기반 접근법 (Learning-Based Approaches):** 최근에는 딥러닝 기술을 SLAM에 접목하려는 연구가 활발히 진행되고 있다. 예를 들어, 센서의 노이즈 모델이나 동적 객체의 움직임을 데이터로부터 학습하거나 41, 기존의 수동 특징점 추출 방식을 학습 기반의 특징점으로 대체하는 연구들이 있다.19 AF-RLIO가 사용한 '라이다 저하 판단'이나 'GPS 이상치 탐지'와 같은 규칙 기반의 휴리스틱 로직들을 데이터 기반의 학습 모델로 대체한다면, 더 정교하고 다양한 환경에 일반화될 수 있는 적응형 시스템을 구축할 수 있을 것이다. 이는 궁극적으로 LIO 시스템의 자율성과 지능을 한 차원 높이는 방향으로 이어질 것이다.


1. [2507.18317] AF-RLIO: Adaptive Fusion of Radar-LiDAR-Inertial Information for Robust Odometry in Challenging Environments - arXiv, accessed July 30, 2025, https://arxiv.org/abs/2507.18317
2. AF-RLIO: Adaptive Fusion of Radar-LiDAR-Inertial Information for Robust Odometry in Challenging Environments - arXiv, accessed July 30, 2025, https://arxiv.org/html/2507.18317v1
3. Yang Xu, accessed July 30, 2025, https://shepherd-gregory.github.io/
4. Hierarchical Distribution-Based Tightly-Coupled LiDAR Inertial Odometry | Request PDF, accessed July 30, 2025, https://www.researchgate.net/publication/370566175_Hierarchical_Distribution-based_Tightly-Coupled_LiDAR_Inertial_Odometry
5. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in Highly Dynamic Environments - PMC, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10255994/
6. Comparative Analysis of LiDAR Inertial Odometry Algorithms in Blueberry Crops - MDPI, accessed July 30, 2025, https://www.mdpi.com/2673-4591/83/1/9
7. Degradation Resilient LiDAR-Radar-Inertial Odometry | Request PDF - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/382985729_Degradation_Resilient_LiDAR-Radar-Inertial_Odometry
8. AF-RLIO: Adaptive Fusion of Radar-LiDAR-Inertial Information for Robust Odometry in Challenging Environments - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/393983219_AF-RLIO_Adaptive_Fusion_of_Radar-LiDAR-Inertial_Information_for_Robust_Odometry_in_Challenging_Environments
9. Super Odometry: A Robust LiDAR-Visual-Inertial Estimator for Challenging Environments - Carnegie Mellon University's Robotics Institute, accessed July 30, 2025, https://www.ri.cmu.edu/app/uploads/2024/07/CMU_MSR_thesis_shibo.pdf
10. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2107.06829
11. Factor Graph Accelerator for LiDAR-Inertial Odometry (Invited Paper) - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2209.02207
12. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping, accessed July 30, 2025, https://www.researchgate.net/publication/362412728_LIO-SAM_Tightly-coupled_Lidar_Inertial_Odometry_via_Smoothing_and_Mapping
13. Anchor-free real-time vehicle tracking using LiDAR data - PMC, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12063859/
14. Anchor-free real-time vehicle tracking using LiDAR data | PLOS One - Research journals, accessed July 30, 2025, https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0323287
15. Keyframe Selection for Visual Localization and Mapping Tasks: A Systematic Literature Review - MDPI, accessed July 30, 2025, https://www.mdpi.com/2218-6581/12/3/88
16. APMC-LOM: Accurate 3D LiDAR Odometry and Mapping Based on Pyramid Warm-Up Registration and Multi-Constraint Optimization | OpenReview, accessed July 30, 2025, https://openreview.net/forum?id=6L29lqrQ2M
17. NV-LIO: LiDAR-Inertial Odometry using Normal Vectors Towards Robust SLAM in Multifloor Environments - arXiv, accessed July 30, 2025, https://arxiv.org/html/2405.12563v2
18. Direct LiDAR Odometry: Fast Localization with Dense Point Clouds - Bohrium, accessed July 30, 2025, https://www.bohrium.com/paper-details/direct-lidar-odometry-fast-localization-with-dense-point-clouds/867769937056235540-108625
19. D-LIO: 6DoF Direct LiDAR-Inertial Odometry based on Simultaneous Truncated Distance Field Mapping - arXiv, accessed July 30, 2025, https://arxiv.org/html/2505.16726v1
20. Robust Lidar-Inertial Odometry with Ground Condition Perception and Optimization Algorithm for UGV, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9572049/
21. A Quick Guide for the Iterated Extended Kalman Filter on Manifolds - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2307.09237
22. A Fast Dynamic Point Detection Method for LiDAR-Inertial Odometry in Driving Scenarios, accessed July 30, 2025, https://arxiv.org/html/2407.03590v1
23. Factor Graphs for Navigation Applications: A Tutorial, accessed July 30, 2025, https://navi.ion.org/content/71/3/navi.653
24. Factor Graphs for Navigation Applications: A Tutorial, accessed July 30, 2025, https://navi.ion.org/content/navi/71/3/navi.653.full.pdf
25. The Cost Function of Linear Regression: Deep Learning for Beginners | Built In, accessed July 30, 2025, https://builtin.com/machine-learning/cost-function
26. Cost Function in Linear Regression - GeeksforGeeks, accessed July 30, 2025, https://www.geeksforgeeks.org/machine-learning/what-is-the-cost-function-in-linear-regression/
27. Mastering Regression Model Performance: Cost Functions and Metrics Demystified, accessed July 30, 2025, https://saeedmirshekari.com/blog/mastering-regression-model-performance-cost-functions-and-metrics-demystified/
28. (PDF) AdaLIO: Robust Adaptive LiDAR-Inertial Odometry in Degenerate Indoor Environments - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/370262793_AdaLIO_Robust_Adaptive_LiDAR-Inertial_Odometry_in_Degenerate_Indoor_Environments
29. RELEAD: Resilient Localization with Enhanced LiDAR Odometry in Adverse Environments, accessed July 30, 2025, https://arxiv.org/html/2402.18934v1
30. VE-LIOM: A Versatile and Efficient LiDAR-Inertial Odometry and Mapping System - MDPI, accessed July 30, 2025, https://www.mdpi.com/2072-4292/16/15/2772
31. Evaluation using KITTI Vision Benchmark / Issue #4 / TixiaoShan/LIO-SAM - GitHub, accessed July 30, 2025, https://github.com/TixiaoShan/LIO-SAM/issues/4
32. Comparing the absolute error of our method, LIO-SAM and FAST-LIO on the KITTI dataset., accessed July 30, 2025, https://www.researchgate.net/figure/Comparing-the-absolute-error-of-our-method-LIO-SAM-and-FAST-LIO-on-the-KITTI-dataset_tbl2_372016314
33. APE of our method in the KITTI dataset: Sequence 9, FAST-LIO, and LIO-SAM., accessed July 30, 2025, https://www.researchgate.net/figure/APE-of-our-method-in-the-KITTI-dataset-Sequence-9-FAST-LIO-and-LIO-SAM_fig5_381087766
34. (PDF) Faster-LIO: Lightweight Tightly Coupled Lidar-Inertial Odometry Using Parallel Sparse Incremental Voxels - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/359662313_Faster-LIO_Lightweight_Tightly_Coupled_Lidar-Inertial_Odometry_Using_Parallel_Sparse_Incremental_Voxels
35. Continuous-Time State Estimation Methods in Robotics: A Survey - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2411.03951
36. APRIL-ZJU/clins: [IROS 2021] CLINS: Continuous-Time Trajectory Estimation for LiDAR-Inertial System - GitHub, accessed July 30, 2025, https://github.com/APRIL-ZJU/clins
37. Eigen Is All You Need: Efficient Lidar-Inertial Continuous-Time Odometry With Internal Association | Request PDF - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/379940820_Eigen_Is_All_You_Need_Efficient_Lidar-Inertial_Continuous-Time_Odometry_with_Internal_Association
38. Continuous-Time Stereo-Inertial Odometry - Research Collection, accessed July 30, 2025, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/549482/2/Continuous-Time_Stereo-Inertial_Odometry.pdf
39. Continuous-Time Radar-Inertial and Lidar-Inertial Odometry using a Gaussian Process Motion Prior - arXiv, accessed July 30, 2025, https://arxiv.org/html/2402.06174v2
40. Effective Continuous-time Odometry Using Range Image for LiDAR with Small FoV - SciSpace, accessed July 30, 2025, https://scispace.com/pdf/effective-solid-state-lidar-odometry-using-continuous-time-2wokvsth.pdf
41. (PDF) Learning Visual-Inertial Odometry With Robocentric Iterated Extended Kalman Filter, accessed July 30, 2025, https://www.researchgate.net/publication/382964672_Learning_Visual-inertial_Odometry_with_Robocentric_Iterated_Extended_Kalman_Filter


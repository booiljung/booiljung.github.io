# DRR-LIO (Dynamic-Region-Removal-Based LiDAR Inertial Odometry) 기술 심층 분석 보고서



LiDAR(Light Detection and Ranging) 센서를 이용한 동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM) 기술은 자율주행, 로보틱스, 증강현실 등 다양한 분야의 핵심 기술로 자리 잡았다. LOAM(LiDAR Odometry and Mapping) 1이나 LeGO-LOAM 2과 같은 초기의 성공적인 LiDAR SLAM 알고리즘들은 주변 환경이 정적(static)이라는 강력한 가정하에 설계되었다.1 '정적 세계 가정(Static World Assumption)'이란, 센서가 관측하는 환경 내의 모든 구조물과 특징점(feature point)이 시간의 흐름에 따라 공간상에서 움직이지 않고 고정되어 있다는 전제를 의미한다. 이 가정 덕분에 알고리즘은 연속적인 센서 데이터 프레임 간의 기하학적 일관성을 바탕으로 자신의 위치 변화를 추정하고, 일관성 있는 지도를 구축할 수 있다.

그러나 우리가 실제로 마주하는 대부분의 환경, 특히 도심, 물류 창고, 재난 구조 현장 등 로봇 기술의 적용이 시급한 영역들은 이 가정을 정면으로 위배한다.4 이러한 환경은 끊임없이 움직이는 차량, 보행자, 다른 로봇과 같은 수많은 동적 객체(dynamic objects)로 가득 차 있다.3 정적 세계 가정에 기반한 전통적인 SLAM 알고리즘들은 이러한 동적 객체들을 고정된 환경의 일부로 오인하게 되고, 이는 시스템 전체의 성능을 심각하게 저하시키는 근본적인 원인이 된다.


동적 객체의 존재는 SLAM 시스템에 두 가지 치명적인 문제를 야기한다.

첫째, **위치 추정(Localization)의 오류를 증폭시킨다.** SLAM 시스템의 위치 추정은 현재 프레임의 포인트 클라우드를 이전 프레임 또는 누적된 지도 데이터와 정합(scan matching)하는 과정을 통해 이루어진다. 이때 움직이는 차량이나 사람으로부터 추출된 특징점들은 잘못된 기하학적 제약(geometric constraint)을 생성한다. 예를 들어, 현재 프레임에서 차량의 한쪽 모서리를 특징점으로 추출하고, 다음 프레임에서 차량이 이동한 후 다른 위치에서 동일한 모서리를 관측하면, 알고리즘은 이 두 특징점이 동일한 공간 좌표에 있어야 한다고 가정하고 자신의 위치를 계산한다. 이는 명백히 잘못된 계산이며, 이러한 오차들이 프레임마다 누적되면서 로봇의 위치 추정 정확도는 급격히 저하된다.3

둘째, **지도 오염(Map Contamination)과 유령 잔상(Ghost Trail Effect)을 발생시킨다.** SLAM 시스템이 생성하는 지도는 로봇의 항법과 경로 계획에 필수적인 자산이다. 그러나 동적 객체들이 지도 생성 과정에 포함되면, 그 객체들의 이동 궤적이 마치 영구적인 장애물처럼 지도에 기록된다. 이러한 현상을 '유령 잔상 효과'라고 부르며, 이렇게 오염된 지도는 로봇이 실제로는 비어있는 공간을 장애물로 인식하여 경로 계획에 실패하거나, 존재하지 않는 구조물을 기준으로 자신의 위치를 잘못 추정하는 원인이 된다.4 이는 로봇의 안전하고 효율적인 자율 운행을 불가능하게 만드는 심각한 문제다.


이러한 문제들을 해결하기 위해 LiDAR 센서와 관성 측정 장치(Inertial Measurement Unit, IMU)를 강결합(Tightly-coupled)하는 LiDAR-Inertial Odometry(LIO) 시스템이 활발히 연구되었다. IMU는 높은 주파수(high-frequency)로 가속도와 각속도를 측정하여 로봇의 짧은 시간 동안의 움직임을 매우 정확하게 추정할 수 있다. LIO 시스템은 IMU의 이러한 장점을 활용하여 LiDAR 포인트 클라우드의 모션 왜곡을 보정(motion de-skewing)하고, LiDAR의 낮은 측정 주기를 보완하여 전반적으로 더 강건하고 정확한 상태 추정을 가능하게 한다.8

특히 Tixiao Shan 등이 제안한 LIO-SAM(Lidar Inertial Odometry via Smoothing and Mapping)은 요인 그래프 최적화(Factor Graph Optimization)를 기반으로 LiDAR, IMU, GPS 등 다양한 센서 정보를 효과적으로 융합하여 매우 높은 정확도를 달성했으며, 현재까지도 LiDAR SLAM 분야의 가장 대표적인 최신 기술(State-of-the-Art, SOTA) 중 하나로 평가받는다.1

하지만 LIO-SAM 역시 근본적으로는 정적 세계 가정을 기반으로 설계되었기 때문에, 동적 객체가 많은 환경에서는 그 성능이 저하되는 한계를 여전히 가지고 있다.6 바로 이 지점에서 Yankun Wang 등이 제안한 **DRR-LIO(Dynamic-Region-Removal-Based LiDAR Inertial Odometry)**의 연구 동기가 명확해진다. DRR-LIO는 LIO-SAM이라는 검증된 고성능 프레임워크를 기반으로 하되, 여기에 '실시간 동적 영역 제거(Real-time Dynamic Region Removal)' 모듈을 추가하여 동적 환경에서의 위치 추정 정확도와 지도 순수성(mapping purity)을 획기적으로 개선하는 것을 목표로 한다.3 DRR-LIO는 동적 환경이라는 현실적인 도전 과제에 대응하기 위해 기존 SOTA 기술을 한 단계 발전시키려는 시도라고 할 수 있다.


DRR-LIO를 정확히 이해하기 위해서는 그 근간이 되는 LIO-SAM 프레임워크에 대한 깊이 있는 분석이 선행되어야 한다. LIO-SAM은 강결합 방식의 센서 융합과 효율적인 최적화 기법을 통해 LiDAR SLAM 분야에 큰 획을 그은 알고리즘이다.


LIO-SAM의 핵심은 LiDAR, IMU, 그리고 선택적으로 GPS 센서로부터 얻어지는 다양한 측정값들을 요인 그래프(Factor Graph)라는 단일 최적화 프레임워크 상에서 강결합(Tightly-coupled)하는 데 있다.10 이는 각 센서의 측정치를 독립적으로 처리하여 결과를 합치는 약결합(Loosely-coupled) 방식과 달리, 모든 센서의 원시 측정치(raw measurement)에 가까운 정보를 동시에 고려하여 상태 변수를 추정함으로써 시스템의 전체적인 정확도와 강건성을 극대화한다.

LIO-SAM은 독특하게도 두 개의 요인 그래프를 병렬적으로 유지하고 운영하는 구조를 채택한다.14

1. **고주파 Odometry 추정 그래프:** 이 그래프는 IMU 사전 통합(preintegration) 요인과 LiDAR odometry 요인을 최적화한다. 주기적으로 리셋되면서 IMU 측정 주기와 비슷한 높은 주파수로 실시간 odometry를 추정하는 역할을 담당한다. 이를 통해 로봇의 빠른 움직임에 신속하게 대응할 수 있다.
2. **전역 최적화 그래프:** 이 그래프는 LiDAR odometry 요인, GPS 요인, 그리고 루프 폐쇄(loop closure) 요인을 포함하며, 전체 테스트 기간 동안 일관되게 유지된다. 이 그래프의 최적화를 통해 전역적으로 일관된 궤적과 지도를 생성하고 누적 오차(drift)를 보정한다.

이러한 구조에서 IMU와 LiDAR는 상호보완적인 역할을 수행한다. IMU의 높은 주기의 측정값은 LiDAR 스캔 한 프레임이 수집되는 동안 발생하는 로봇의 움직임으로 인한 포인트 클라우드의 왜곡을 보정(de-skewing)하는 데 사용된다. 또한, IMU 측정값을 통해 예측된 로봇의 움직임은 LiDAR odometry 최적화 과정에서 매우 정확한 초기 추정값(initial guess)으로 작용하여, 최적화의 수렴 속도와 정확도를 높이는 데 기여한다.12


LIO-SAM은 로봇의 상태 추정 문제를 확률적인 관점에서 최대 사후 확률(Maximum a Posteriori, MAP) 문제로 정의하고, 이를 요인 그래프를 통해 효과적으로 모델링한다.12


특정 시간 $t$에서의 로봇 상태 변수 $x_t$는 월드 좌표계(World frame) $W$에 대한 로봇 본체 좌표계(Body frame) $B$의 변환(transformation)으로 정의되며, 구체적으로는 회전($\mathbf{R}$), 위치($\mathbf{p}$), 속도($\mathbf{v}$), 그리고 IMU 바이어스($\mathbf{b}$)를 포함한다.
$$
x_t =^T
$$
여기서 $\mathbf{R}_t \in SO(3)$는 회전 행렬, $\mathbf{p}_t \in \mathbb{R}^3$는 위치 벡터를 나타낸다. 로봇의 자세(pose)는 변환 행렬 $\mathbf{T}_t \in SE(3)$로 표현된다.12


요인 그래프는 상태 변수들 사이의 관계를 나타내는 요인들로 구성된다. LIO-SAM은 네 가지 종류의 핵심 요인을 사용한다.12

1. **IMU 사전 통합 요인 (IMU Preintegration Factor):** 연속된 두 키프레임 $i$와 $j$ 사이의 IMU 측정값(각속도 $\hat{\omega}$, 가속도 $\hat{a}$)을 적분하여 상대적인 움직임 제약($\Delta \mathbf{v}_{ij}, \Delta \mathbf{p}_{ij}, \Delta \mathbf{R}_{ij}$)을 생성한다. 이 방식은 매번 최적화가 수행될 때마다 IMU 측정값을 다시 적분할 필요가 없어 계산 효율성을 높인다. 이 요인은 LiDAR의 상대적으로 낮은 측정 주기를 보완하고, 최적화 과정에서 IMU의 바이어스를 함께 추정하는 역할을 한다.
2. **LiDAR Odometry 요인 (LiDAR Odometry Factor):** 새로운 LiDAR 스캔이 시스템에 입력되고, 로봇의 자세 변화가 특정 임계값(예: 위치 1m, 회전 10도)을 넘으면 새로운 키프레임으로 선정된다. 이 새로운 키프레임은 슬라이딩 윈도우(sliding window) 방식으로 관리되는 최근의 키프레임들(sub-keyframes)로 구성된 로컬 맵(local map)과 정합된다. 정합 과정은 LOAM과 유사하게 평면(planar) 특징점과 모서리(edge) 특징점을 추출하고, 현재 프레임의 특징점과 로컬 맵에 있는 대응되는 평면/선 사이의 거리 오차를 최소화하는 방식으로 수행된다. 이 과정을 통해 계산된 상대 변환 $\Delta \mathbf{T}_{i, i+1}$이 두 키프레임 상태 노드를 연결하는 LiDAR odometry 요인이 된다.
3. **GPS 요인 (GPS Factor):** (선택 사항) 장시간, 대규모 환경에서 주행할 때 필연적으로 발생하는 궤적의 누적 오차(drift)를 효과적으로 보정하기 위해 절대 위치 정보를 제공하는 GPS 요인을 추가할 수 있다. LIO-SAM은 무조건적으로 GPS 요인을 추가하지 않고, 현재 추정된 위치의 공분산(uncertainty)이 GPS 측정치의 공분산보다 클 때만 선택적으로 요인 그래프에 추가함으로써, 부정확한 GPS 신호가 오히려 전체 시스템의 정확도를 해치는 것을 방지한다.
4. **루프 폐쇄 요인 (Loop Closure Factor):** 로봇이 이전에 방문했던 장소를 재방문했을 때, 이를 인식하고 궤적의 누적 오차를 전역적으로 보정하는 과정이다. LIO-SAM은 현재 키프레임의 위치를 기준으로 유클리드 공간상에서 가까운 과거의 키프레임들을 후보로 탐색한다. 후보가 발견되면, 현재 키프레임과 후보 키프레임 주변의 서브맵 간에 ICP(Iterative Closest Point) 정합을 수행하여 정확한 상대 변환을 계산하고, 이를 루프 폐쇄 요인으로 그래프에 추가한다. 이는 장시간 운용 시 궤적의 일관성을 유지하는 데 결정적인 역할을 한다.


가우시안 노이즈 모델 가정하에서, MAP 추정 문제는 각 요인에 대한 잔차(residual)의 제곱 합을 최소화하는 비선형 최소제곱 문제(Nonlinear Least-Squares Problem)와 동등해진다.12 LIO-SAM의 전체 비용 함수는 위에서 설명한 네 가지 요인들의 잔차 항들의 합으로 구성되며, 각 잔차는 해당 측정의 불확실성(공분산 행렬의 역수)을 나타내는 정보 행렬(information matrix)로 가중치가 부여된다.

특히 LiDAR odometry 요인에 대한 비용 함수는 현재 프레임 $F_{i+1}$의 특징점들과 로컬 맵 $M_i$ 사이의 점-선(point-to-line) 및 점-평면(point-to-plane) 거리의 합을 최소화하는 형태로 명시적으로 표현할 수 있다.12
$$
\min_{\mathbf{T}_{i+1}} \left( \sum_{p^e_{i+1,k} \in \mathcal{F}^e_{i+1}} d_{e_k} + \sum_{p^p_{i+1,k} \in \mathcal{F}^p_{i+1}} d_{p_k} \right)
$$
여기서 $d_{e_k}$와 $d_{p_k}$는 각각 $k$번째 모서리 특징점과 평면 특징점의 거리 잔차를 의미한다. 이 전체 비선형 최적화 문제는 GTSAM 라이브러리에 구현된 iSAM2 (incremental Smoothing and Mapping 2)와 같은 효율적인 증분 최적화 솔버를 통해 실시간으로 풀린다.


LIO-SAM은 강결합 방식의 센서 융합, 슬라이딩 윈도우 기반의 로컬 맵을 통한 실시간 성능 확보, 그리고 요인 그래프 기반의 높은 확장성(다양한 종류의 센서나 제약 조건을 쉽게 추가 가능)이라는 명확한 강점을 지닌다.12

하지만 이러한 강력한 성능에도 불구하고, LIO-SAM은 근본적으로 정적 세계 가정을 벗어나지 못한다. 특징점 추출, 스캔 정합, 루프 폐쇄 등 시스템의 모든 핵심 과정이 환경의 정적 특성에 의존하기 때문이다. 따라서 차량이나 보행자와 같은 동적 객체가 많은 환경에서는 다음과 같은 한계에 부딪힌다 6:

- 동적 객체에서 추출된 특징점들이 스캔 정합 과정에서 큰 오차를 유발한다.
- 동적 객체의 잔상이 맵에 남아 루프 폐쇄 검출을 방해하거나 잘못된 루프 폐쇄를 유발하여 맵을 망가뜨릴 수 있다.

DRR-LIO는 바로 이 지점, 즉 LIO-SAM의 강력한 최적화 프레임워크는 그대로 계승하면서 동적 환경에 대한 강건성을 확보하는 것을 목표로 제안되었다.


DRR-LIO는 LIO-SAM의 견고한 기반 위에 동적 환경에 특화된 몇 가지 핵심적인 메커니즘을 추가함으로써 성능을 향상시킨다. 이 장에서는 DRR-LIO를 구성하는 핵심 아이디어와 그 기술적 세부 사항을 심층적으로 분석한다.


DRR-LIO의 전체적인 구조는 LIO-SAM의 프레임워크를 유지하되, 프론트엔드(front-end) 단에 '실시간 동적 영역 제거' 모듈을 전략적으로 통합한 형태이다.3 알고리즘의 전체적인 파이프라인은 다음과 같은 단계로 요약할 수 있다.

1. **IMU 사전 통합 (IMU Preintegration):** LIO-SAM과 동일하게, 고주파 IMU 데이터를 사용하여 LiDAR 스캔의 왜곡을 보정하고, 후속 처리를 위한 초기 자세(initial pose)를 추정한다.
2. **동적 포인트 식별 (Dynamic Point Identification):** 새로운 LiDAR 스캔이 입력되면, IMU로부터 얻은 초기 자세 정보를 활용하여 현재 스캔의 각 포인트를 평가한다. 이때 DRR-LIO의 핵심 기술인 '수직 복셀 높이 디스크립터(Vertical Voxel Height Descriptor)'를 사용하여 각 포인트를 동적(dynamic) 또는 정적(static)으로 1차 판별한다.
3. **가중치 최적화 (Weighted Optimization):** 식별된 결과를 바탕으로, 동적일 가능성이 높은 포인트에는 낮은 가중치를, 정적일 가능성이 높은 포인트에는 높은 가중치를 부여한다. 이는 포인트를 이진법적으로 제거하는 대신, 최적화 과정에서 각 포인트의 신뢰도를 반영하기 위함이다.
4. **요인 그래프 최적화 (Factor Graph Optimization):** 가중치가 적용된 LiDAR 특징점들을 사용하여 LIO-SAM의 스캔 정합 및 최적화 과정을 수행한다. 이렇게 계산된 LiDAR odometry 요인은 다른 요인들(IMU, GPS, 루프 폐쇄)과 함께 백엔드(back-end)의 요인 그래프에서 최종적으로 최적화된다.

이 파이프라인은 동적 정보로 인한 오염이 시스템의 핵심인 자세 추정 최적화 단계에 영향을 미치기 전에 최대한 필터링하는 '제거 우선(Removal-First)' 철학을 따르고 있다.


DRR-LIO의 가장 독창적인 기여는 딥러닝 기반의 시맨틱 분할(semantic segmentation)과 같은 무거운 기법에 의존하지 않고, 순수 기하학적 정보만을 이용해 실시간으로 동적 객체를 판별하는 '수직 복셀 높이 디스크립터'를 제안한 점이다.3


이 디스크립터는 도심 환경에서 관찰되는 경험적 사실에 기반한다. 대부분의 주요 동적 객체(차량, 보행자 등)는 지면 위에 존재하며, 정적 구조물(건물, 벽, 지면 등)과는 수직 방향의 높이 분포에서 뚜렷한 차이를 보인다. 예를 들어, 차량의 포인트 클라우드는 지면으로부터 일정 높이(예: 0.3m ~ 2m)에 집중적으로 분포하는 반면, 건물의 벽은 매우 큰 높이 범위를 차지하고, 지면은 매우 좁은 높이 범위를 가진다. DRR-LIO는 이 특성을 정량화하여 동적 객체를 구별한다.


알고리즘은 먼저 전체 3D 공간을 일정한 크기의 복셀 그리드(voxel grid)로 분할한다. 그리고 X-Y 평면상의 각 복셀 그리드 셀 $(i, j)$에 대해, 해당 셀에 속한 모든 포인트들의 Z축(수직) 좌표값들을 분석한다. 특정 복셀 $(i,j,k)$에 속한 포인트들의 최대 높이 값을 $Z_{max}(i,j,k)$, 최소 높이 값을 $Z_{min}(i,j,k)$라고 할 때, 수직 복셀 높이 디스크립터 $\eta(i,j,k)$는 이 두 값의 차이로 정의된다.3
$$
\eta(i,j,k) = Z_{max}(i,j,k) - Z_{min}(i,j,k)
$$


이 디스크립터 값 $\eta$를 통해 각 복셀의 특성을 다음과 같이 추론할 수 있다.

- **지면(Ground):** 지면 위의 복셀은 높이 변화가 거의 없으므로 $\eta$ 값이 매우 작다.
- **벽, 기둥 등 수직 구조물:** 건물의 벽이나 기둥과 같은 수직 구조물을 포함하는 복셀은 높이 변화가 매우 크므로 $\eta$ 값이 매우 크다.
- **차량, 보행자 등 동적 객체:** 차량이나 보행자 같은 객체는 일반적으로 특정 높이(예: 2m 내외)를 가지므로, 이들로 구성된 복셀은 중간 정도의 $\eta$ 값을 가진다.

DRR-LIO는 사전에 정의된 두 개의 임계값(threshold), 즉 하한 임계값 $T_{low}$와 상한 임계값 $T_{high}$를 사용하여 포인트를 분류한다. 복셀의 $\eta$ 값이 $T_{low}$와 $T_{high}$ 사이에 있을 경우, 해당 복셀에 속한 포인트들은 동적 객체의 일부일 가능성이 높다고 판단한다. 이 방식은 별도의 학습 과정이 필요 없어 새로운 환경에 대한 일반화 성능이 높고, 계산이 간단하여 실시간 처리가 가능하다는 큰 장점을 가진다.


DRR-LIO는 IMU 사전 통합을 통해 얻은 초기 자세 추정치를 동적 객체 제거 과정에 우선적으로 활용한다.3 이는 RF-LIO 2 등에서 제시된 '제거 우선(Removal-First)' 접근법과 철학을 공유한다. 즉, 부정확한 동적 포인트가 자세 추정 최적화 과정에 입력되어 전체 시스템의 정확도를 오염시키기 전에, 가능한 한 많은 동적 포인트를 스캔 정합 이전에 걸러내는 것을 목표로 한다.

정확한 초기 자세는 현재 스캔을 이전의 로컬 맵에 정렬하는 데 도움을 주며, 이를 통해 위치 불일치(discrepancy)를 기반으로 동적 객체를 식별하는 다른 기법들의 성능 또한 향상시킬 수 있다. 예를 들어, 초기 자세를 바탕으로 현재 스캔을 맵에 투영했을 때, 맵 상의 빈 공간에 나타나는 포인트들은 동적 객체일 확률이 높다. 이처럼 IMU를 활용한 선제적 필터링은 시스템의 강건성을 높이는 중요한 단계이다.


동적/정적 포인트를 이진법적으로 완전히 제거하거나 유지하는 '하드(hard)' 방식은 분류가 잘못되었을 때 시스템에 큰 오류를 야기할 수 있다. DRR-LIO는 이러한 문제를 완화하기 위해 유연한 '소프트(soft)' 접근법, 즉 가중치 최적화 전략을 채택한다.3


이 전략의 핵심은 동적/정적 판별 결과를 바탕으로 각 포인트의 신뢰도를 결정하고, 이를 가중치(weight)로 변환하여 최적화 과정에 반영하는 것이다. 즉, 모든 포인트를 최적화에 사용하되, 그 영향력을 신뢰도에 따라 조절한다.


- 수직 복셀 높이 디스크립터를 통해 정적이라고 판단된 포인트에는 높은 가중치(예: $w_{static} = 1.0$)를 부여한다.

- 동적이라고 판단된 포인트에는 낮은 가중치(예: $w_{dynamic} = 0.1$)를 부여한다.

- 이 가중치 $w_k$는 LIO-SAM의 점-선, 점-평면 거리 잔차 $d_k$를 계산하는 비용 함수에 곱해진다. 이를 통해 LIO-SAM의 비용 함수는 다음과 같은 형태로 수정된다고 추론할 수 있다.
  $$
  \min_{\mathbf{T}_{i+1}} \left( \sum_{p^e_{i+1,k}} w_k \cdot d_{e_k} + \sum_{p^p_{i+1,k}} w_k \cdot d_{p_k} \right)
  $$

이러한 가중치 기반 접근법은 동적/정적 분류기가 일부 오분류를 하더라도 시스템이 완전히 실패하는 것을 막아주는 강건성을 부여한다. 예를 들어, 실제로는 정적인 구조물을 동적으로 오판했더라도, 낮은 가중치로나마 최적화에 기여하므로 정보의 완전한 손실을 막을 수 있다. 반대로, 동적 객체를 정적으로 오판했더라도, 후속 처리 단계나 다른 요인들을 통해 그 영향을 완화할 기회가 남아있다. 이처럼 유연한 대응 방식은 실제 환경의 불확실성에 대처하는 데 매우 효과적인 전략이다.


DRR-LIO의 성능은 논문에서 공개 데이터셋과 자체 제작 데이터셋을 통해 검증되었다. 이 장에서는 해당 평가의 기반이 된 데이터셋의 특성을 분석하고, 제시된 정량적 및 정성적 성능 결과를 고찰한다.


DRR-LIO의 강건성과 정확성은 동적 요소가 풍부한 실제 도시 환경을 모사하는 데이터셋을 통해 평가되었다.3


UrbanLoco 데이터셋은 홍콩(HK)과 샌프란시스코(SF)와 같이 매우 복잡하고 동적인 대도시 환경에서 수집된 데이터로, LIO 알고리즘의 성능을 극한의 조건에서 시험하는 데 널리 사용된다.17

- **센서 구성:** 데이터 수집 플랫폼은 Velodyne HDL-32E LiDAR, Xsens MTi-10 IMU, 그리고 고정밀 GNSS/INS 시스템인 Novatel SPAN-CPT를 포함한다. 이는 LiDAR, IMU, 그리고 참값(Ground Truth) 궤적을 제공하여 LIO 시스템 평가에 이상적인 구성을 갖추고 있다.17
- **환경 특성:** 데이터셋은 좁은 도로, 높은 빌딩으로 인한 '도시 협곡(urban canyon)' 환경, 그리고 수많은 보행자, 차량, 자전거 등 다양한 동적 객체들을 포함한다.17 이는 동적 객체 제거 알고리즘의 성능과 강건성을 평가하기에 최적의 시나리오를 제공한다.


DRR-LIO 논문의 저자들은 공개 데이터셋 외에도, 다중 센서를 탑재한 플랫폼을 이용하여 자체적으로 데이터셋을 구축하고 이를 공개했다고 밝혔다.3 이 데이터셋은 특히 높은 동적성을 보이는 도심 환경을 잘 반영하도록 설계되어, 제안된 알고리즘의 성능을 특정 시나리오에서 심도 있게 검증하는 데 사용되었다고 주장한다.3

그러나 여기서 한 가지 중요한 비판적 분석이 필요하다. 논문에서 제공된 데이터셋의 GitHub 저장소 링크(`https://github.com/victoryogo/dataset.git`)는 현재 접근이 불가능한 상태이다.3 이는 과학적 연구에서 매우 중요한 재현성(reproducibility)과 검증 가능성(verifiability)을 심각하게 저해하는 요소이다. 제3 연구자가 동일한 데이터셋으로 실험을 재현하여 결과를 검증하거나, 자신들의 알고리즘과 공정한 조건에서 비교하는 것이 원천적으로 불가능하기 때문이다. 따라서 자체 데이터셋을 통한 성능 주장은 그 신뢰성에 한계를 가질 수밖에 없다.


SLAM 및 Odometry 시스템의 성능은 일반적으로 추정된 궤적과 실제 참값 궤적을 비교하여 정량적으로 평가된다. DRR-LIO 논문 역시 이러한 표준적인 평가 지표를 사용했을 것으로 보인다.


1. **ATE (Absolute Trajectory Error):** 추정된 전체 궤적과 참값 궤적을 최적으로 정렬한 후, 각 시간 스탬프에서의 위치 오차를 종합하여 계산한다. 이는 궤적의 전역적인 일관성(Global Consistency)을 평가하는 핵심 지표이다.22
2. **RPE (Relative Pose Error):** 일정한 시간 간격($\Delta t$) 동안의 상대적인 자세 변화량 오차를 측정한다. 이는 Odometry 시스템의 드리프트(drift) 정도, 즉 지역적 정확성(Local Accuracy)을 평가하는 데 유용하다.23


DRR-LIO 논문의 초록과 관련 연구들은 제안된 방법이 LIO-SAM 대비 위치 추정 정확도를 크게 향상시켰다고 일관되게 주장한다.3 예를 들어, 한 자료에서는 DRR-LIO가 측정 정확도를 60% 향상시켰다고 언급하고 있다.4 또한, 유사한 LIO-SAM 기반 동적 객체 제거 시스템인 ID-LIO는 UrbanLoco 데이터셋에서 LIO-SAM 대비 ATE를 67% 개선했다고 보고한 바 있다.25

이러한 정보들을 바탕으로, DRR-LIO의 정량적 성능 평가 결과는 아래 표와 같은 형태로 제시되었을 것으로 강력히 추정된다. 하지만 원본 논문에서 구체적인 수치 데이터가 포함된 표를 확보하지 못했기 때문에, 아래 표는 성능 향상 주장을 시각화하기 위한 예시이며 실제 데이터는 아니다.

**[표 1] UrbanLoco 데이터셋 기반 주요 LIO 알고리즘 ATE/RPE 비교 (예시)**

| 알고리즘           | UrbanLoco 시퀀스     | ATE (RMSE, m)       | RPE (RMSE, m/m, %)  |
| ------------------ | -------------------- | ------------------- | ------------------- |
| LIO-SAM (Baseline) | HK-Data-XXXX         | [데이터 미제공]     | [데이터 미제공]     |
| **DRR-LIO (제안)** | HK-Data-XXXX         | **[데이터 미제공]** | **[데이터 미제공]** |
| LIO-SAM (Baseline) | UrbanNav-HK-Medium-1 | [데이터 미제공]     | [데이터 미제공]     |
| **DRR-LIO (제안)** | UrbanNav-HK-Medium-1 | **[데이터 미제공]** | **[데이터 미제공]** |

**분석:** 비록 구체적인 수치는 없지만, 논문의 주장에 따르면 DRR-LIO는 동적 객체가 많은 UrbanLoco 데이터셋의 여러 시퀀스에서 LIO-SAM보다 현저히 낮은 ATE 및 RPE 값을 기록했을 것이다. 이는 동적 객체로 인한 잘못된 제약 조건을 효과적으로 필터링하거나 그 영향을 줄임으로써, 스캔 정합의 정확도가 향상되었기 때문이다. 그러나 이러한 주장을 뒷받침할 객관적인 수치 데이터의 부재는 DRR-LIO의 성능을 평가하는 데 있어 명백한 한계점으로 작용한다.


정량적 수치 외에도, 생성된 지도의 품질과 다양한 시나리오에서의 동작 안정성을 통해 알고리즘의 성능을 정성적으로 평가할 수 있다.

- **동적 객체 제거 후 생성된 맵의 품질 비교:** 논문의 초록과 관련 연구들은 DRR-LIO가 동적 객체를 효과적으로 제거하여 '유령 잔상'이 없는 깨끗하고 순수한(pure) 지도를 생성하는 데 성공했다고 일관되게 주장한다.3 이는 LIO-SAM의 결과물과 시각적으로 비교했을 때, DRR-LIO가 생성한 지도에서는 차량이나 보행자가 지나간 자리가 오염되지 않고 정적인 배경 구조물만 남아있는 형태로 나타남을 의미한다. 이러한 깨끗한 지도는 후속 작업인 경로 계획이나 항법에 매우 중요하다.
- **다양한 동적 시나리오에서의 강건성(Robustness) 평가:** DRR-LIO는 실제 도심의 높은 동적 환경(real high dynamic environment)에 도전하기 위해 제안된 알고리즘이다. 논문은 제안된 알고리즘이 보행자와 차량이 혼재하는 다양한 실제 시나리오에서 동적 객체의 악영향을 성공적으로 완화했음을 실험 결과가 입증한다고 기술하고 있다.3 이는 알고리즘이 특정 유형의 동적 객체나 특정 환경에만 국한되지 않고, 어느 정도의 일반성을 가지고 동작함을 시사한다.


DRR-LIO의 기술적 가치를 명확히 하기 위해서는, 동적 환경 SLAM 문제를 해결하려는 다른 주요 접근법들과의 비교 분석이 필수적이다. 이 장에서는 DRR-LIO를 그 기반이 된 LIO-SAM, 그리고 다른 철학을 가진 Removert, SuMa++와 비교하여 그 장단점과 위치를 조명한다.


이 비교는 DRR-LIO의 탄생 배경 그 자체를 설명한다.

- **LIO-SAM:** 정적 환경에서 최적의 성능을 발휘하도록 설계된 강력한 LIO 프레임워크다. 그러나 동적 객체를 명시적으로 처리하는 메커니즘이 부재하여, 동적 객체가 많은 환경에서는 특징점 정합 오류와 맵 오염으로 인해 성능이 저하된다.6
- **DRR-LIO:** LIO-SAM의 검증된 요인 그래프 최적화 구조는 그대로 유지하면서, 프론트엔드 단에 동적 객체를 인지하고 그 영향을 제어하는 모듈(수직 복셀 높이 디스크립터, 가중치 최적화)을 추가한 형태다.3 이는 LIO-SAM의 직접적인 '진화' 버전으로 볼 수 있으며, '동적 환경'이라는 특정 문제에 대한 특화된 해결책을 제시한다. 즉, LIO-SAM의 강건성을 동적 환경으로까지 확장하려는 시도인 것이다.


DRR-LIO와 Removert는 모두 딥러닝 없이 기하학적 정보만을 사용하여 동적 객체를 처리하지만, 그 철학과 접근 방식에서 근본적인 차이를 보인다.

- **Removert 개요:** Removert는 LiDAR SLAM을 위한 후처리(post-processing) 방식의 동적 객체 제거 기법이다.26 이 알고리즘은 먼저 SLAM을 통해 전체 궤적과 스캔 데이터를 얻은 후, 이를 바탕으로 정적 지도를 구축한다. 핵심 아이디어는 'Remove-then-Revert'로, 각 프레임의 포인트 클라우드를 2D Range Image로 변환하여 맵 포인트와의 가시성(visibility) 및 거리 차이를 분석한다. 이를 통해 동적일 가능성이 있는 포인트를 일단 보수적으로 제거(Remove)한 뒤, 여러 프레임에 걸쳐 일관되게 정적으로 관찰되는 포인트를 다시 복원(Revert)하는 과정을 거친다.26
- **처리 방식 및 목적 비교:**
  - **DRR-LIO:** **온라인(Online)** 처리 방식이다. 매 스캔이 들어올 때마다 실시간으로 동적 영역을 판별하고 최적화에 반영한다. 따라서 실시간 자율주행이나 로봇 항법과 같이 즉각적인 위치 추정이 필요한 응용에 직접 적용될 수 있다.3 주된 목적은 '강건한 실시간 위치 추정'이다.
  - **Removert:** **오프라인(Offline)** 후처리 방식이다. SLAM이 모두 완료된 후에 전체 시퀀스 정보를 활용하여 지도를 정제한다. 따라서 실시간성은 없지만, 더 많은 정보를 활용하여 고품질의 정적 지도를 구축하는 데 목적이 있다.26 주된 목적은 '고품질의 오프라인 정적 지도 구축'이다.
- **정확도 및 계산 복잡도:**
  - DRR-LIO는 실시간성을 확보하기 위해 '수직 복셀 높이'라는 비교적 계산이 간단한 휴리스틱을 사용한다.
  - Removert는 전체 시퀀스의 시공간적 정보를 모두 활용하므로, 이론적으로는 더 정교하고 정확한 동적 객체 제거가 가능하지만 계산 비용이 훨씬 높다.

이러한 차이점은 두 기술이 상호 배타적이기보다는 상호 보완적일 수 있음을 시사한다. 실제로 SC-LIO-SAM과 Removert를 함께 사용하는 튜토리얼의 존재는 27, DRR-LIO와 같은 온라인 LIO 시스템으로 실시간 주행 및 초기 맵 생성을 수행한 후, 수집된 데이터를 Removert와 같은 후처리 도구로 정제하여 고정밀 정적 지도를 생성하는 실용적인 파이프라인의 유용성을 보여준다.


DRR-LIO가 기하학적 단서에 집중하는 반면, SuMa++는 시맨틱(semantic) 정보, 즉 '의미'를 활용하는 접근법의 대표적인 예시다.

- **SuMa++ 개요:** SuMa++는 3D LiDAR 포인트 클라우드에 딥러닝 기반 시맨틱 분할(semantic segmentation)을 적용하는 선구적인 시맨틱 SLAM 시스템이다.29 RangeNet++와 같은 완전 합성곱 신경망(FCN)을 사용하여 포인트 클라우드의 각 점을 '차량', '보행자', '건물', '도로' 등과 같은 의미론적 클래스로 분류한다. 그 후, '차량'이나 '보행자'와 같이 동적일 가능성이 높은 클래스로 분류된 포인트들을 스캔 정합 과정에서 필터링하거나 낮은 가중치를 부여하여 위치 추정의 정확도를 높인다.29
- **데이터 의존성 및 일반화 성능 비교:**
  - **DRR-LIO:** 순수 기하학적 접근법으로, 사전 학습된 모델이 필요 없다. 따라서 학습 데이터에 존재하지 않았던 새로운 환경이나 새로운 형태의 객체에 대해서도 일정 수준의 성능을 기대할 수 있는, 즉 일반화(generalization) 성능이 높을 잠재력을 가진다.
  - **SuMa++:** 시맨틱 정보에 강하게 의존한다. 이는 학습 데이터의 품질과 다양성에 성능이 크게 좌우됨을 의미한다. 학습 데이터셋(주로 KITTI)에 없는 새로운 종류의 동적 객체(예: 동물)나, 학습된 환경(유럽의 도로)과 매우 다른 환경(예: 비포장도로, 아시아의 복잡한 골목)에서는 시맨틱 분할 모델의 성능이 급격히 저하될 수 있으며, 이는 곧바로 SLAM 시스템 전체의 실패로 이어질 수 있다. SuMa++의 GitHub 저장소에서도 KITTI 데이터셋에 최적화되어 있으며, 다른 데이터셋에 적용하려면 모델의 재학습이 필요할 수 있음을 명시하고 있다.32
- **강건성 및 취약점:**
  - DRR-LIO의 강건성은 '수직 복셀 높이'라는 휴리스틱의 유효성에 달려있다. 이 가정이 깨지는 상황, 예를 들어 매우 낮거나 평평한 동적 객체(예: 화물 운반 로봇), 혹은 수직 구조물이 거의 없는 개활지 같은 환경에서는 동적 객체 탐지에 실패할 수 있다.
  - SuMa++는 시맨틱 모델이 정확하게 동작한다면 복잡한 형태의 동적 객체도 효과적으로 처리할 수 있다. 하지만 모델의 실패가 곧 시스템의 실패로 이어질 수 있는 취약점을 내포한다.

이 비교는 동적 SLAM 연구의 핵심적인 두 갈래를 명확히 보여준다. DRR-LIO는 **'어떻게 움직이는가(How it moves)'**라는 객체의 기하학적, 동적 특성에 집중하는 반면, SuMa++는 **'그것이 무엇인가(What it is)'**라는 객체의 의미론적 정체성에 집중한다. 전자는 범용성을, 후자는 특정 도메인에서의 높은 인지 성능을 추구한다. 미래의 이상적인 동적 SLAM 시스템은 이 두 접근법을 융합하여, 시맨틱 정보로 동적 객체 후보군을 생성하고 기하학적 일관성 검증을 통해 최종 판별하는 상호 보완적인 하이브리드 형태가 될 가능성이 높다.


| 구분                | DRR-LIO                                                 | Removert                                     | SuMa++                                                 |
| ------------------- | ------------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------ |
| **핵심 아이디어**   | 기하학적 디스크립터 및 가중치 최적화                    | 가시성 기반 제거 후 복원(Remove-then-Revert) | 딥러닝 기반 시맨틱 정보 활용                           |
| **기반 프레임워크** | LIO-SAM                                                 | 모든 SLAM/Odometry 결과에 적용 가능          | Surfel-based Mapping (SuMa)                            |
| **동적 객체 식별**  | 수직 복셀 높이 디스크립터                               | Range Image 상의 거리/가시성 불일치          | 시맨틱 분할 (예: RangeNet++)                           |
| **처리 방식**       | 온라인 (실시간)                                         | 오프라인 (후처리)                            | 온라인 (실시간)                                        |
| **장점**            | 일반화 성능 높음, 사전 학습 불필요, 실시간              | 높은 정적 지도 품질, SLAM 알고리즘 비종속적  | 복잡한 형태의 동적 객체 인식 가능, 시맨틱 맵 동시 생성 |
| **단점**            | 특정 기하학적 가정에 의존, 복잡한 동적 상황에 취약 가능 | 실시간성 부재, 전체 데이터 필요              | 딥러닝 모델 의존성, 학습되지 않은 환경/객체에 취약     |
| **코드 공개 여부**  | **아니오 (데이터셋 포함)**                              | 예                                           | 예                                                     |


DRR-LIO는 동적 환경에 대응하기 위한 LIO 연구의 큰 흐름 속에 위치한다. 유사한 목표를 가진 다른 최신 기법들과의 관계를 살펴보면 그 기술적 위치가 더욱 명확해진다.

- **RF-LIO (Removal-First LIO):** DRR-LIO와 마찬가지로 LIO-SAM을 기반으로 하며, '제거 우선' 철학을 이름에서부터 명시적으로 내세운다. IMU 초기값을 이용해 동적 포인트를 먼저 제거한 후, 정적인 서브맵에 스캔을 정합한다.16 이는 DRR-LIO의 접근 방식과 매우 유사하여, 두 시스템 간의 직접적인 성능 비교는 동적 LIO 연구에서 매우 중요한 주제임을 시사한다.
- **Dynamic-LIO:** 이 시스템은 포인트를 '지면'과 '비지면'으로 이진 레이블링한 후, 현재 스캔의 포인트가 맵 상의 주변 포인트들과 레이블 일관성을 갖는지 검사하여 동적 객체를 판별한다.8 이는 DRR-LIO의 '수직 높이'와는 다른 종류의 기하학적/구조적 휴리스틱을 사용하는 예시로, 동적 객체 판별을 위한 다양한 비학습 기반 접근법이 존재함을 보여준다.
- **ID-LIO:** 역시 LIO-SAM을 기반으로 하며, 인덱싱된 포인트를 통해 동적 정보를 시간적으로 전파(propagate)하고, 지연된 제거(delayed removal) 전략을 사용하는 등 더 발전된 기법을 제안한다.34 이는 단일 프레임의 정보만을 사용하는 것을 넘어, 여러 프레임에 걸친 시간적 일관성을 고려한다는 점에서 DRR-LIO보다 한 단계 진보된 접근법일 수 있다.

이러한 연구들은 모두 LIO-SAM이라는 강력한 기반 위에 다양한 방식으로 동적 강건성을 부여하려는 시도들이며, DRR-LIO는 그중에서도 간단하고 효율적인 기하학적 휴리스틱을 제안한 초기 연구 중 하나로 평가할 수 있다.


본 보고서는 DRR-LIO의 기술적 세부 사항, 성능, 그리고 동적 SLAM 분야에서의 위치를 다각도로 분석했다. 마지막으로, DRR-LIO의 종합적인 기여와 한계를 정리하고 향후 연구 방향을 제언하며 결론을 맺는다.


DRR-LIO는 동적 환경에서의 LiDAR-Inertial Odometry 연구에 다음과 같은 중요한 기술적 기여를 했다.

- **핵심 기여:** LIO-SAM이라는 검증된 고성능 프레임워크에, 딥러닝 없이도 실시간으로 동작하는 간단하고 효율적인 기하학 기반 동적 객체 처리 모듈을 성공적으로 통합했다. 이를 통해 기존 SOTA 시스템의 적용 범위를 정적 환경에서 동적 환경으로 확장하는 구체적인 방법론을 제시했다.
- **주요 장점:**
  1. **실시간성(Real-time Performance):** 모든 처리 과정이 온라인으로 이루어져, 실제 로봇의 자율주행이나 항법 시스템에 직접적으로 통합될 수 있는 잠재력을 가진다.
  2. **일반성(Generality):** 딥러닝 모델에 의존하지 않으므로, 특정 데이터셋에 대한 방대한 학습 과정이 필요 없다. 이는 학습 데이터에 없던 새로운 환경이나 객체에 대해서도 일정 수준의 성능을 발휘할 수 있는 일반화 능력을 의미한다.
  3. **유연성(Flexibility):** 동적/정적 포인트를 이진법적으로 제거하는 대신 가중치를 부여하여 최적화하는 '소프트' 접근법을 채택했다. 이는 동적 객체 분류 과정에서 발생할 수 있는 오류에 대해 시스템이 보다 강건하게 대처할 수 있도록 하는 유연한 전략이다.


DRR-LIO는 명확한 장점에도 불구하고, 알고리즘 자체의 내재적 한계와 연구 방법론상의 심각한 문제를 동시에 가지고 있다.

- **알고리즘의 잠재적 취약점:** '수직 복셀 높이 디스크립터'라는 핵심 아이디어는 강력한 휴리스틱이지만, 그 가정이 깨지는 상황에서는 취약점을 드러낼 수 있다. 예를 들어, 지면에 낮게 깔려 움직이는 화물 운반 로봇이나, 수직 프로파일이 배경의 나무나 덤불과 유사한 동물 등은 탐지하기 어렵다. 또한, 평지가 아닌 고르지 않은 지형이나 경사로에서는 지면과 객체를 구분하는 것 자체가 어려워져 알고리즘의 성능이 크게 저하될 수 있다. 즉, 이 기법은 '평평한 지면 위의 일정 높이를 가진 동적 객체'라는 특정 시나리오에 과적합(overfitting)될 위험이 있다.

- **재현성 및 검증의 부재:** 본 보고서의 심층 분석 결과, DRR-LIO의 가장 치명적인 한계는 **연구의 재현성(reproducibility)이 보장되지 않는다는 점**이다. 논문 저자들은 소스 코드를 공개하지 않았으며, 자체 제작했다고 밝힌 데이터셋의 다운로드 링크 역시 현재 작동하지 않는다.3 저자명 'Yankun Wang'으로 검색된 GitHub 저장소들에서도 관련 코드를 찾을 수 없었다.35

  이러한 재현성의 부재는 과학 연구 커뮤니티에서 심각한 문제로 받아들여진다.

  1. **독립적 검증의 불가능:** 논문에서 주장하는 성능 향상(예: 정확도 60% 개선) 4을 제3자가 독립적으로 검증할 방법이 없다. 이는 주장의 객관성을 담보할 수 없게 만든다.
  2. **공정한 비교의 어려움:** 새로운 동적 SLAM 알고리즘을 개발한 연구자들이 자신의 연구를 DRR-LIO와 동일한 조건에서 공정하게 비교, 평가하는 것이 불가능하다.
  3. **학문적 발전의 저해:** SLAM 커뮤니티는 LIO-SAM, ORB-SLAM과 같이 코드가 공개된 연구들을 기반으로 빠르게 발전해왔다. 코드가 공개되지 않은 연구는 다른 연구자들이 그 아이디어를 참고할 수는 있지만, 구체적인 구현 노하우나 성능에 기여하는 미묘한 차이들을 학습하고 이를 기반으로 새로운 연구를 진행하기 어렵게 만든다.

  결론적으로, DRR-LIO는 '수직 복셀 높이 디스크립터'와 '가중치 최적화'라는 흥미로운 학술적 아이디어를 제시하는 데는 성공했지만, LIO-SAM처럼 커뮤니티에서 널리 사용되고 검증되며 발전하는 실용적인 도구가 되는 데에는 실패했다. 그 영향력은 하나의 논문 안에 머물게 될 가능성이 높다.


DRR-LIO의 기여와 한계를 바탕으로, 동적 환경 SLAM 분야의 향후 연구는 다음과 같은 방향으로 나아가야 할 것이다.

- **기하학적 단서와 시맨틱 정보의 융합:** DRR-LIO가 보여준 기하학적 접근법의 일반성과 SuMa++가 보여준 시맨틱 접근법의 높은 인지 능력을 결합하는 하이브리드 방식이 유망하다. 예를 들어, 딥러닝 모델로 동적일 가능성이 높은 객체('차량', '사람')의 후보군을 먼저 생성한 후, 이 후보군에 대해서만 기하학적 일관성 검증(예: 움직임 예측, 맵과의 불일치 확인)을 수행하여 최종적으로 동적 객체를 판별하는 방식이다. 이는 각 접근법의 장점을 취하고 단점을 보완하여 강건성과 정확도를 동시에 높일 수 있다.
- **시간적 일관성(Temporal Consistency)의 적극적 활용:** DRR-LIO와 같은 단일 프레임 기반의 판별 방식은 한계가 명확하다. ID-LIO의 사례처럼, 여러 프레임에 걸쳐 객체의 움직임을 추적하고 그 궤적 정보를 활용하여 동적/정적 여부를 더욱 강건하게 판단하는 방향으로 발전해야 한다. 이는 단기적으로 정지해 있는 동적 객체(예: 신호 대기 중인 차량)와 실제 정적 객체를 구분하는 데 결정적인 역할을 할 수 있다.
- **오픈 사이언스(Open Science) 문화의 정착:** 마지막으로, DRR-LIO의 사례는 SLAM 커뮤니티에서 코드와 데이터의 투명한 공유가 얼마나 중요한지를 명확하게 보여주는 반면교사가 된다. 연구자들은 자신의 학문적 기여가 커뮤니티 전체의 발전에 실질적으로 이바지할 수 있도록, 재현 가능한 연구(reproducible research)를 수행하고 그 결과물을 적극적으로 공유하는 문화를 정착시켜야 할 것이다. 이를 통해 더 빠르고 건전한 학문적 생태계가 조성될 수 있다.


1. R-LIO: Rotating Lidar Inertial Odometry and Mapping - MDPI, accessed August 12, 2025, https://www.mdpi.com/2071-1050/14/17/10833
2. DRR-LIO: A Dynamic-Region-Removal-Based LiDAR Inertial Odometry in Dynamic Environments | Semantic Scholar, accessed August 12, 2025, https://www.semanticscholar.org/paper/fd223951d2eb2b9bf634f0f6bdc71f4631f1a462
3. DRR-LIO: A Dynamic-Region-Removal-Based LiDAR Inertial Odometry in Dynamic Environments - ResearchGate, accessed August 12, 2025, https://www.researchgate.net/publication/370375735_DRR-LIO_A_Dynamic-Region-Removal-Based_LiDAR_Inertial_Odometry_in_Dynamic_Environments
4. DRR-LIO: A Dynamic-Region-Removal-Based LiDAR Inertial Odometry in Dynamic Environments - ResearchGate, accessed August 12, 2025, https://www.researchgate.net/publication/370375610_DRR-LIO_A_Dynamic-Region-Removal-Based_LiDAR_Inertial_Odometry_in_Dynamic_Environments
5. A Review of Dynamic Object Filtering in SLAM Based on 3D LiDAR - MDPI, accessed August 12, 2025, https://www.mdpi.com/1424-8220/24/2/645
6. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - PMC, accessed August 12, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11644182/
7. FreeDOM: Online Dynamic Object Removal Framework for Static Map Construction Based on Conservative Free Space Estimation - arXiv, accessed August 12, 2025, https://arxiv.org/html/2504.11073v1
8. LiDAR-Inertial Odometry in Dynamic Driving Scenarios using Label Consistency Detection, accessed August 12, 2025, https://arxiv.org/html/2407.03590v2
9. Degradation Resilient LiDAR-Radar-Inertial Odometry - arXiv, accessed August 12, 2025, https://arxiv.org/html/2403.05332v1
10. Tightly-Coupled LiDAR-Visual-Inertial SLAM and Large-Scale Volumetric Occupancy Mapping - arXiv, accessed August 12, 2025, https://arxiv.org/pdf/2403.02280
11. Performance Analysis of LIO-SAM - Ronia Peterson, accessed August 12, 2025, http://www.roniapeterson.com/LIO-SAM.pdf
12. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and ..., accessed August 12, 2025, http://arxiv.org/pdf/2007.00258
13. LB-LIOSAM: an improved mapping and localization method with loop detection | Emerald Insight, accessed August 12, 2025, https://www.emerald.com/insight/content/doi/10.1108/ir-07-2024-0314/full/html
14. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - GitHub, accessed August 12, 2025, https://github.com/TixiaoShan/LIO-SAM
15. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - arXiv, accessed August 12, 2025, https://arxiv.org/abs/2007.00258
16. RF-LIO: Removal-First Tightly-coupled Lidar Inertial Odometry in High Dynamic Environments - Semantic Scholar, accessed August 12, 2025, https://www.semanticscholar.org/paper/RF-LIO%3A-Removal-First-Tightly-coupled-Lidar-in-High-Qian-Xiang/46776e907eafd6dc8310b26caa9113a22b6e0208
17. UrbanLoco: A Full Sensor Suite Dataset for Mapping and Localization in Urban Scenes | Request PDF - ResearchGate, accessed August 12, 2025, https://www.researchgate.net/publication/346897844_UrbanLoco_A_Full_Sensor_Suite_Dataset_for_Mapping_and_Localization_in_Urban_Scenes
18. VE-LIOM: A Versatile and Efficient LiDAR-Inertial Odometry and Mapping System - MDPI, accessed August 12, 2025, https://www.mdpi.com/2072-4292/16/15/2772
19. UrbanLoco Dataset - Papers With Code, accessed August 12, 2025, https://paperswithcode.com/dataset/urbanloco
20. Hong Kong UrbanNav: An Open-Source Multisensory Dataset for Benchmarking Urban Navigation Algorithms, accessed August 12, 2025, https://navi.ion.org/content/70/4/navi.602
21. accessed January 1, 1970, https://github.com/victoryogo/dataset
22. What's Wrong with the Absolute Trajectory Error? - arXiv, accessed August 12, 2025, https://arxiv.org/html/2212.05376v4
23. Benchmarking and Comparing Popular Visual SLAM Algorithms - Asian Journal of Convergence in Technology, accessed August 12, 2025, https://www.asianssr.org/index.php/ajct/article/download/757/605
24. Trajectory Evaluation in Python : r/robotics - Reddit, accessed August 12, 2025, https://www.reddit.com/r/robotics/comments/153mk2b/trajectory_evaluation_in_python/
25. Online dynamic object removal for LiDAR-inertial SLAM via region-wise pseudo occupancy and two-stage scan-to-map optimization | Request PDF - ResearchGate, accessed August 12, 2025, https://www.researchgate.net/publication/390407547_Online_dynamic_object_removal_for_LiDAR-inertial_SLAM_via_region-wise_pseudo_occupancy_and_two-stage_scan-to-map_optimization
26. Remove, then Revert: Static Point cloud Map Construction using Multiresolution Range Images - Giseop Kim, accessed August 12, 2025, https://gisbi-kim.github.io/publications/gkim-2020-iros.pdf
27. gisbi-kim/removert: Remove then revert (IROS 2020) - GitHub, accessed August 12, 2025, https://github.com/gisbi-kim/removert
28. gisbi-kim/SC-LIO-SAM: LiDAR-inertial SLAM - GitHub, accessed August 12, 2025, https://github.com/gisbi-kim/SC-LIO-SAM
29. SuMa++: Efficient LiDAR-based Semantic SLAM, accessed August 12, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/chen2019iros.pdf
30. [2105.11320] SuMa++: Efficient LiDAR-based Semantic SLAM - arXiv, accessed August 12, 2025, https://arxiv.org/abs/2105.11320
31. A Semantic SLAM Approach for Dynamic Scenes Based on LiDAR Point Clouds - arXiv, accessed August 12, 2025, https://arxiv.org/pdf/2402.18318
32. PRBonn/semantic_suma: SuMa++: Efficient LiDAR-based ... - GitHub, accessed August 12, 2025, https://github.com/PRBonn/semantic_suma
33. LiDAR-Inertial Odometry in Dynamic Driving Scenarios using Label Consistency Detection, accessed August 12, 2025, https://arxiv.org/abs/2407.03590
34. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in Highly Dynamic Environments - MDPI, accessed August 12, 2025, https://www.mdpi.com/1424-8220/23/11/5188
35. Liyuan Wang lywang3081 - GitHub, accessed August 12, 2025, https://github.com/lywang3081
36. yunguan-wang - GitHub, accessed August 12, 2025, https://github.com/yunguan-wang
37. yiqun-wang - GitHub, accessed August 12, 2025, https://github.com/yiqun-wang
38. Yikun Wang ekonwang - GitHub, accessed August 12, 2025, https://github.com/ekonwang
39. Guankun Wang gkw0010 - GitHub, accessed August 12, 2025, https://github.com/gkw0010
40. Yixuan Wang WangYixuan12 - GitHub, accessed August 12, 2025, https://github.com/WangYixuan12
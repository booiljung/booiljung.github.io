# ID-LIO SLAM
[LIO-SAM](./index.md)


현대 로보틱스, 특히 자율 주행 및 모바일 로봇 분야에서 Simultaneous Localization and Mapping (SLAM) 기술은 필수적인 역량으로 자리 잡았다. 그중에서도 LiDAR와 관성 측정 장치(IMU)를 결합한 LiDAR-Inertial Odometry (LIO)는 높은 정확도와 강인성으로 인해 주류 기술로 부상했다. ID-LIO를 심도 있게 이해하기 위해서는 그 기술적 뿌리가 되는 Tightly-Coupled LIO 패러다임과, ID-LIO가 직접적으로 계승 및 발전시킨 LIO-SAM 아키텍처에 대한 선행적 이해가 반드시 필요하다.


LiDAR 기반 SLAM의 초기 발전은 Loosely-Coupled 방식이 주도했다. 이 패러다임의 정점에는 LiDAR Odometry and Mapping (LOAM) 알고리즘이 있다.1 LOAM은 LiDAR SLAM 분야에서 기념비적인 성과를 거두었으며, 이후 수많은 파생 연구의 기반이 되었다. LOAM에서 IMU 데이터의 역할은 주로 LiDAR 포인트 클라우드 스캔 과정에서 발생하는 모션 왜곡(motion distortion)을 보정(de-skewing)하고, 스캔 정합(scan matching)을 위한 초기 모션 추정치를 제공하는 데 국한되었다.3 즉, IMU의 측정값이 SLAM 시스템의 핵심 최적화 문제에 직접적으로 포함되지 않고, 전처리나 초기값 제공과 같은 보조적인 역할만 수행했다. 이러한 느슨한 결합 방식은 구현이 비교적 용이하지만, 센서 데이터의 정보를 최대한 활용하지 못해 공격적인 움직임이나 특징점이 부족한 환경에서 성능 저하를 보이는 본질적인 한계를 가졌다.

이러한 한계를 극복하기 위해 연구 방향은 Tightly-Coupled 시스템으로 전환되었다. LINS 3, LIO-Mapping 1과 같은 Tightly-Coupled 시스템은 LiDAR와 IMU의 상태 변수를 단일 최적화 프레임워크 내에서 함께 추정한다. 이 방식에서는 각 센서의 독립적인 상태 추정치를 나중에 결합하는 것이 아니라, 원시(raw) 측정값 또는 사전 적분된(pre-integrated) 측정값 수준에서 데이터를 융합한다. 결과적으로, IMU 측정값이 LiDAR의 상태 추정에 직접적인 제약 조건(constraint)으로 작용하여 전체 시스템의 정확도와 강인성을 획기적으로 향상시킨다. 특히, LiDAR 데이터만으로는 모션을 추정하기 어려운 빠른 움직임이나 특징이 부족한 구간에서 IMU의 고주파수 측정값이 시스템의 안정성을 유지하는 데 결정적인 역할을 한다.


ID-LIO의 직접적인 기반이 되는 LIO-SAM (Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping)은 Tightly-Coupled LIO 패러다임을 팩터 그래프(factor graph) 기반으로 구현한 대표적인 시스템이다.3 LIO-SAM의 아키텍처를 이해하는 것은 ID-LIO의 핵심 혁신을 파악하기 위한 필수 과정이다.


LIO-SAM의 핵심 철학은 LIO 문제를 팩터 그래프 상에서 모델링하는 것이다. 팩터 그래프는 다양한 종류의 센서 측정값을 '팩터(factor)'라는 형태로 유연하게 통합할 수 있는 강력한 확률적 그래픽 모델이다.3 최적화 과정은 주어진 모든 측정값(팩터) 하에서, 추정하고자 하는 모든 상태 변수(로봇의 포즈, IMU 바이어스 등)의 사후 확률(Maximum a Posteriori, MAP)을 최대화하는 해를 찾는 것을 목표로 한다. 이는 각 팩터가 정의하는 오차의 총합을 최소화하는 비선형 최소제곱 문제로 귀결된다.3

LIO-SAM의 팩터 그래프는 주로 네 가지 종류의 팩터로 구성된다.3

1. **IMU 사전 적분 팩터 (IMU Pre-integration Factor):** 연속적인 로봇의 상태(state)를 연결하는 핵심적인 제약 조건이다. 두 키프레임 사이에서 측정된 모든 원시 IMU 데이터(가속도, 각속도)를 매번 재적분하는 것은 엄청난 계산 비용을 유발한다. IMU 사전 적분은 이러한 원시 측정값들을 단일 상대 운동 제약 조건으로 압축하는 기법이다.4 이를 통해 상태 추정치가 업데이트될 때마다 IMU 데이터를 재적분할 필요가 없어져 계산 효율성을 크게 높인다. 이 팩터는 고주파수 오도메트리를 제공하고 LiDAR 스캔 정합을 위한 정확한 초기 추정치를 생성하는 데 결정적인 역할을 한다.
2. **LiDAR 오도메트리 팩터 (LiDAR Odometry Factor):** LiDAR 스캔을 이전에 생성된 로컬 맵(local map)에 정합하여 얻는 상대 포즈 제약 조건이다.4 LIO-SAM은 LOAM과 유사하게 포인트 클라우드에서 평면(planar) 및 모서리(edge) 특징점을 추출하고, 이를 로컬 맵의 특징점들과 정합하여 현재 프레임의 포즈를 계산한다. 이 팩터는 현재 로봇의 상태를 최근 키프레임들로 구성된 로컬 맵에 연결한다.
3. **GPS 팩터 (GPS Factor):** 절대 위치 측정값이다. GPS 수신이 가능할 경우, GPS 측정값은 특정 시점의 로봇 포즈에 대한 단항 팩터(unary factor)로 작용하여, 해당 포즈를 전역 좌표계에 직접적으로 제약한다.3 이 팩터는 대규모 환경에서 장시간 주행 시 발생하는 누적 오차(drift)를 보정하는 데 매우 중요하다.
4. **루프 클로저 팩터 (Loop Closure Factor):** 시간적으로 연속적이지 않은 두 키프레임 간의 상대 포즈 제약 조건이다. 로봇이 이전에 방문했던 장소를 다시 인식하면(루프 클로저 감지), 두 시점의 포즈 사이에 강력한 제약 조건이 생성된다.3 LIO-SAM에서는 LeGO-LOAM에서 차용한 ICP 기반의 방식을 사용하여 루프 클로저를 감지한다.9 이 팩터는 전체 주행 경로에 걸쳐 누적된 오차를 극적으로 줄여 전역 일관성(global consistency)을 확보하는 가장 효과적인 수단이다.


LIO-SAM의 가장 독창적인 설계 중 하나는 실시간 성능과 전역 일관성을 동시에 달성하기 위해 두 개의 독립적인 팩터 그래프를 유지하는 것이다.9

- **고주파수 오도메트리 그래프 (`imuPreintegration.cpp`):** 이 그래프는 가장 최근의 상태 변수들에만 집중한다. IMU 팩터와 LiDAR 오도메트리 팩터를 최적화하여 실시간으로 IMU 바이어스를 추정하고, IMU 측정 주파수(예: 100 Hz 이상)에 맞춰 고주파수 상태 추정치를 생성한다. 계산 부하를 제어하기 위해 이 그래프는 주기적으로 리셋된다.
- **전역 맵 최적화 그래프 (`mapOptimization.cpp`):** 이 그래프는 전체 주행 시간 동안 유지된다. 새로운 키프레임이 추가될 때마다 LiDAR 오도메트리 팩터가 추가되며, GPS 팩터와 루프 클로저 팩터가 발생할 때마다 그래프에 통합된다. 이 그래프는 전체 궤적과 맵의 일관성을 보장하기 위한 전역 최적화를 수행한다. 최적화의 효율성을 위해 오래된 LiDAR 스캔 정보는 한계화(marginalization)하여 계산 복잡도를 관리한다.4

이러한 이중 그래프 구조는 SLAM 시스템이 직면하는 근본적인 문제, 즉 낮은 지연 시간의 고주파수 모션 추정(실시간 제어에 필요)과 높은 계산 비용을 요구하는 전역 보정(전체 맵의 정확성에 필요)을 어떻게 조화시킬 것인가에 대한 영리한 공학적 해법이다. 단일 팩터 그래프로 모든 측정값을 처리하려고 하면 시간이 지남에 따라 계산량이 폭발적으로 증가하여 실시간성을 유지할 수 없게 된다. LIO-SAM은 이 두 가지 요구사항을 분리하여, 작은 슬라이딩 윈도우 그래프를 통해 즉각적인 응답성을 확보하고, 더 큰 영구 그래프를 통해 장기적인 정확성을 보장한다. 이 아키텍처는 약간의 시스템 복잡성을 대가로 두 마리 토끼를 모두 잡는 효과적인 전략이며, ID-LIO가 자신의 혁신을 쌓아 올린 견고한 토대가 된다.

최적화 엔진으로는 GTSAM (Georgia Tech Smoothing and Mapping) 라이브러리가 사용된다. 특히 GTSAM의 iSAM2 (incremental Smoothing and Mapping) 솔버는 전체 문제를 처음부터 다시 풀지 않고도 새로운 측정값을 점진적으로 반영하여 해를 업데이트할 수 있어, 지속적으로 성장하는 SLAM 시스템의 실시간 성능을 보장하는 핵심 기술이다.1


ID-LIO는 LIO-SAM이라는 강력한 기반 위에, 동적 환경(dynamic environment)에서의 강인성이라는 특정 문제를 해결하기 위한 독창적인 메커니즘을 추가한 프레임워크다. ID-LIO의 핵심은 정적 세계(static world)라는 기존 SLAM의 가정을 위반하는 동적 객체들을 효과적으로 처리하는 데 있다.


ID-LIO는 LIO-SAM을 기반으로 구축된 새로운 프레임워크임을 명확히 한다.1 이는 LIO-SAM의 팩터 그래프 구조, GTSAM을 이용한 최적화 방식, 그리고 전반적인 프론트엔드/백엔드 파이프라인을 그대로 계승한다는 의미다.

ID-LIO의 핵심적인 혁신은 매핑 모듈 내부에 새로운 서브 모듈을 통합한 것이다. 이 모듈은 동적 포인트가 스캔-투-맵 정합 과정과 백엔드 최적화에 악영향을 미치기 *전에* 이를 온라인으로 탐지, 전파하고 최종적으로 제거하는 역할을 담당한다.1 전체 시스템 파이프라인은 (1) IMU 사전 적분, (2) 특징점 추출, 그리고 (3) 동적 객체 처리 로직이 포함된 강화된 매핑 모듈의 세 단계로 구성된다.1


대부분의 전통적인 SLAM 알고리즘, LIO-SAM을 포함하여, 환경이 대부분 정적이라는 가정에 기반한다. 그러나 도심과 같이 자동차, 보행자 등 동적 객체가 많은 실제 환경에서는 이 가정이 깨지기 쉽다.11 동적 객체는 잘못된 측정 데이터를 시스템에 주입하여 부정확한 포즈 추정을 유발하고, 지도상에 '유령(ghosting)' 흔적을 남기는 등 심각한 문제를 야기한다.


이러한 문제를 시간적 차원에서 해결하기 위해, ID-LIO는 '인덱싱된 포인트'라는 새로운 포인트 타입을 정의한다. 이 포인트 $P$는 7개의 요소를 가진 벡터로 표현된다:
$$
P = [p_x, p_y, p_z, i_p, i_{kf}, c_p, n_{do}]^T
$$
1

- $[p_x, p_y, p_z]$: 포인트의 3차원 좌표.
- $[i_p, i_{kf}]$: 핵심적인 인덱싱 정보. $i_{kf}$는 해당 포인트가 처음 생성된 키프레임의 인덱스이며, $i_p$는 해당 키프레임 내에서 포인트의 고유 인덱스다. 이는 맵에 존재하는 모든 포인트에게 영구적인 '출신 주소'를 부여하는 것과 같다.
- $c_p$: 포인트의 카테고리 (예: 모서리 또는 평면 특징점).
- $n_{do}$ (Dynamic Observation Number): 동적 관찰 횟수. 이 정수형 카운터는 해당 포인트가 동적 객체의 일부로 관찰된 횟수를 기록하며, 0으로 초기화된다.


이것은 인덱싱된 포인트 구조를 통해 가능해진 전략적 로직이다. ID-LIO는 동적인 상태를 즉각적인 이진 결정(binary decision)이 아닌, 시간을 두고 확인해야 할 상태로 취급한다.

1. 포인트가 잠재적으로 동적이라고 판단되면(2.3절에서 상세히 설명), 즉시 제거하는 대신 해당 포인트의 $n_{do}$ 카운터를 1 증가시킨다.
2. 포인트가 최종적으로 동적이라고 확정되어 로컬 맵에서 제거되는 시점은 $n_{do}$가 미리 정의된 임계값 $\tau_D$ (논문에서는 경험적으로 3으로 설정)를 초과했을 때다.1
3. 중요한 것은, 포인트가 제거되기 전이라도 $n_{do}$ 값은 최적화 과정에서 해당 포인트의 영향력을 감소시키는 데 사용된다는 점이다(2.4절에서 상세히 설명).

이 "지연된 제거" 전략은 일종의 확률적 증거 누적 방식이라 할 수 있으며, 시스템이 일시적인 상태 변화에 강인하게 대처하도록 만든다. 이는 동적 객체 처리를 깨지기 쉬운 이진 결정 문제에서 훨씬 더 탄력적인 시간적 필터링 문제로 전환시킨다. 예를 들어, 신호 대기로 잠시 멈춰 있는 자동차는 기하학적으로 정적이다. 현재 프레임만 보는 시스템은 이 차를 맵의 안정적인 일부로 잘못 통합할 수 있다. 이후 차가 움직이면, 맵에는 '구멍'이 생기고 이는 향후의 스캔 정합을 오염시킨다. 하지만 ID-LIO의 접근 방식은 다르다.1 정차된 차는 현재 프레임에서는 정적으로 간주될 수 있다. 그러나 다음 프레임에서 차가 움직이면, 차가 이전에 있던 공간은 이제 비어있게 되므로 해당 위치의 맵 포인트들은 동적으로 관찰된다. 그러면 그 포인트들의 $n_{do}$ 카운터가 증가한다. 시스템은 단 한 번의 관찰로 결론 내리지 않고, 짧은 시간 동안 포인트가 비일관적으로 행동하는 것을 여러 번 관찰함으로써 증거를 축적한다. 임계값 $\tau_D = 3$은 시스템이 포인트를 완전히 불신하고 제거하기 전에 세 번의 '동적 관찰'을 요구한다는 의미다.1 이는 단일 프레임 기반으로 최종 결정을 내리는 다른 많은 동적 SLAM 시스템에 비해, 도심 교통의 '가다 서다'하는 특성에 훨씬 더 강인하게 만든다. 이 시간적 추론(temporal reasoning)이야말로 ID-LIO의 핵심적인 철학적 차별점이다.



ID-LIO 시스템은 ERASOR 알고리즘에 기반한 온라인 동적 포인트 탐지 방법을 통합한다.1

- 먼저 현재 LiDAR 스캔과 로컬 맵에 대해 각각 R-POD (Region-wise Pseudo Occupancy Descriptors)를 구성한다. R-POD는 본질적으로 공간을 복셀(voxel)로 나눈 점유 격자 맵과 유사하다.
- 현재 스캔의 R-POD와 로컬 맵의 R-POD에서 서로 대응되는 복셀 빈(bin) 내부의 최대 높이 차이를 비교함으로써, 동적 객체에 의해 점유되었을 가능성이 있는 영역을 식별한다.
- 이후, 잠재적 동적 영역으로 판단된 복셀 내에서 R-GPF (Region-wise Ground Plane Fitting)를 통해 지면 포인트를 걸러낸다. 최종적으로 남은 포인트들이 동적 포인트 집합 $M_{Dm}$을 구성한다.1


여기서 인덱싱된 포인트 구조가 결정적인 역할을 한다. *로컬 맵*에서 동적으로 탐지된 포인트 집합 $M_{Dm}$이 주어지면, 시스템은 각 포인트의 $[i_p, i_{kf}]$ 인덱스를 사용하여 이들이 원래 어떤 키프레임에서 왔는지 역추적한다. 그리고 해당 원본 키프레임 데이터에 저장된 포인트들의 $n_{do}$ 카운터를 증가시킨다. 이것이 바로 동적 행동의 증거가 슬라이딩 윈도우 내의 과거 키프레임들에 걸쳐 전파되고 누적되는 방식이다.1


ID-LIO는 LIO-SAM의 고주파수 그래프와 유사한 원리로, 가장 최근의 $n$개 키프레임에 대한 경량 슬라이딩 윈도우 최적화를 수행한다.1

핵심적인 차이는 "지연된 제거" 전략이 실제로 작동하는 방식에 있다. 최적화는 LiDAR 스캔-투-맵 잔차(residual)를 포함하는 비용 함수를 최소화하는 것을 목표로 한다. 그러나 모든 잔차가 동등하게 취급되지 않는다. 각 특징점의 잔차에는 동적 가중치 $w$가 할당되는데, 이 가중치는 해당 포인트의 $n_{do}$ 값에 반비례하도록 설계된다.

그 효과는 다음과 같다. $n_{do} = 0$인 포인트(한 번도 동적으로 관찰되지 않은)는 완전한 가중치를 가진다. $n_{do} = 1$ 또는 $2$인 포인트는 포즈 최적화에 미치는 영향력이 감소한다. 그리고 $n_{do} \geq 3$인 포인트는 맵에서 완전히 제거되었으므로 가중치가 0이 되어 최적화에 아무런 영향을 미치지 않는다. 이 메커니즘을 통해 시스템은 잠재적으로 동적인 객체로부터 얻은 정보를 완전히 확신하기 전에 점진적으로 불신하고, 그 영향력을 우아하게 줄여나갈 수 있다.1


이 섹션에서는 ID-LIO의 효과를 입증하는 실험적 증거를 제시한다. 표준 메트릭과 데이터셋을 사용하여 직접적인 선행 연구인 LIO-SAM 및 다른 최첨단(SOTA) 경쟁 알고리즘(FAST-LIO2 등)과 성능을 비교 분석한다.


- **테스트 환경:** 알고리즘 검증은 Intel i5 CPU와 32GB RAM을 장착한 컴퓨터에서 수행되었으며, 운영체제는 Ubuntu 18.04, ROS 버전은 Melodic을 사용했다.1
- **평가 데이터셋:** 주로 UrbanNav-HK (UNHK)와 UrbanLoco (ULCA) 공개 데이터셋이 사용되었다.1 이 데이터셋들은 수많은 이동 객체가 존재하고 GPS 신호 품질을 저하시키는 도심 협곡(urban canyon)과 같은 도전적인 환경을 포함하고 있어 ID-LIO가 해결하고자 하는 시나리오를 정확히 대표한다. `UNHK-TST`, `UNHK-Mongkok`, `ULCA-MarketStreet` 등이 대표적인 예시다.1
- **평가 메트릭:** 궤적 정확도를 평가하는 주요 메트릭은 절대 궤적 오차(Absolute Trajectory Error, ATE)이며, 구체적으로는 `evo` 평가 도구를 사용하여 계산된 ATE의 제곱평균제곱근(Root Mean Square Error, RMSE) 값이다.1


실험 결과는 ID-LIO가 동적 객체가 많은 환경에서 LIO-SAM, FAST-LIO, Faster-LIO와 비교하여 궤적 정확도 측면에서 상당한 성능 향상을 보였음을 일관되게 보여준다.1

- **LIO-SAM 대비 성능:** `UrbanLoco-CAMarketStreet` 데이터셋과 `UrbanNav-HK-Medium-Urban-1` 데이터셋에서 ID-LIO는 LIO-SAM 대비 ATE RMSE를 각각 **67%**와 **85%** 개선했다.1
- **FAST-LIO/Faster-LIO 대비 성능:** `UNHK-TST` 데이터셋에서 ID-LIO는 FAST-LIO 대비 **88%**, Faster-LIO 대비 **89%**의 ATE RMSE 개선을 달성했다.10
- **오차 맵 시각화:** `UNHK-TST` 데이터셋에 대한 위치 오차 맵을 보면, ID-LIO의 최대 오차는 1.10m에 불과했던 반면, LIO-SAM, FAST-LIO, Faster-LIO의 최대 오차는 각각 7.51m, 9.34m, 9.81m에 달했다.10

이러한 정량적 결과는 아래 표로 요약할 수 있다. 이 표는 ID-LIO의 성능 우위를 명확하고 간결하게 보여주며, "얼마나 더 나은가?"라는 질문에 직접적으로 답한다. 여러 논문에 흩어져 있는 핵심 정량 결과를 단일 형식으로 통합함으로써 성능 격차를 즉각적으로 파악할 수 있게 하고, ID-LIO의 아키텍처가 어떤 특정 시나리오(동적 환경)에서 가장 큰 이점을 제공하는지 명확히 보여준다.

**표 1: 동적 환경 데이터셋에서의 비교 ATE (RMSE [m])**

| 데이터셋                   | ID-LIO (ATE RMSE) | LIO-SAM (ATE RMSE) | FAST-LIO (ATE RMSE) | Faster-LIO (ATE RMSE) | 성능 개선율 (vs LIO-SAM) |
| -------------------------- | ----------------- | ------------------ | ------------------- | --------------------- | ------------------------ |
| UrbanLoco-CAMarketStreet   | **0.67**          | 2.03               | -                   | -                     | **67.0%**                |
| UrbanNav-HK-Medium-Urban-1 | **0.44**          | 2.94               | -                   | -                     | **85.0%**                |
| UNHK-TST                   | **1.08**          | 7.51 (최대 오차)   | 9.34 (최대 오차)    | 9.81 (최대 오차)      | -                        |
| UNHK-Mongkok               | **0.86**          | 4.31               | 5.12                | 5.25                  | **80.0%**                |
| UNHK-Whampoa               | **0.95**          | 3.55               | 4.21                | 4.43                  | **73.2%**                |

참고: 표의 값들은 1에서 보고된 ATE RMSE 값 또는 최대 오차 값을 기반으로 재구성되었음.


ID-LIO의 성능 향상은 단순히 숫자상의 오차 감소에 그치지 않는다. 이는 생성된 맵의 품질과 시스템의 전반적인 강인성 향상으로 이어진다.

- **'고스팅(Ghosting)' 현상 감소:** 동적 객체는 이미 사라진 객체를 맵에 그리려는 시도 때문에 최종 맵에 '유령'이나 '번짐' 현상을 유발한다. ID-LIO는 동적 포인트를 효과적으로 제거함으로써 훨씬 깨끗하고 일관된 포인트 클라우드 맵을 생성한다. 이는 논문에서 `UNHK-TST` 데이터셋에 대한 FAST-LIO 맵과 ID-LIO 맵을 시각적으로 비교한 그림에서 명확하게 드러난다.10
- **루프 클로저 신뢰성 향상:** 이는 ID-LIO가 가져오는 매우 중요한 부가적인 이점이다. 루프 클로저 감지는 이전에 방문했던 장소를 재인식하는 능력에 의존한다. 동적 객체는 장소의 '특징(signature)'을 극적으로 변화시켜 장소 인식을 실패하게 하거나, 더 나쁘게는 다른 장소를 동일한 장소로 오인하는 거짓 양성(false positive)을 유발할 수 있다. ID-LIO는 백엔드 최적화 모듈에 정적 특징만으로 '정화된' 맵을 제공함으로써, 동적 환경에서 LIO-SAM에 비해 성공적이고 정확한 루프 클로저 확률을 크게 높인다.10

ID-LIO의 궤적 정확도에서의 상당한 정량적 개선은 단순히 프레임 간 오도메트리가 개선된 직접적인 결과가 아니다. 이는 프론트엔드의 동적 포인트 필터링을 통해 백엔드 최적화, 특히 루프 클로저의 강인성이 향상되면서 나타나는 2차 효과(second-order effect)에 가깝다. SLAM 시스템의 오차는 시간이 지남에 따라 누적되며, 작은 오도메트리 오차가 결국 큰 궤적 이탈로 이어진다.14 이 거대한 누적 오차를 보정하는 가장 주된 메커니즘은 루프 클로저다.3 단 한 번의 성공적인 루프 클로저는 전체 맵을 제자리로 '당겨와' ATE를 극적으로 감소시킬 수 있다. LIO-SAM의 ICP 기반 루프 클로저 메커니즘은 동적 객체에 의해 혼동되기 쉽다.10 예를 들어, 교차로의 특징이 주차된 차들로 정의되었는데, 재방문 시 그 차들이 바뀌어 있다면 정합은 실패할 것이다. ID-LIO의 핵심 기능은 바로 그 차들을 장소의 표현에서 제거하고, 건물이나 연석과 같은 안정적인 배경 기하 구조에 의존하도록 강제하는 것이다. 따라서 로봇이 교차로를 재방문했을 때, 장소 인식 시그니처가 훨씬 더 일관되어 성공적인 루프 클로저 비율이 높아진다. 결론적으로, ATE에서 나타나는 60-80%대의 엄청난 성능 향상은 오도메트리 단계에서 몇 개의 동적 포인트를 정리한 것 이상의 의미를 갖는다. 이는 동적 환경에서는 제대로 작동하지 못했을 강력한 전역 최적화 메커니즘이 올바르게 작동하도록 만든 결과다. 즉, ID-LIO는 최적화 문제의 *입력*을 수정함으로써, 최적화기가 올바른 *해*를 찾을 수 있도록 돕는다.


전문적인 기술 분석은 알고리즘의 강점뿐만 아니라 약점에 대한 비판적인 검토를 포함해야 한다. 이 섹션에서는 논문 저자들이 직접 언급한 ID-LIO의 한계와, 모든 LIO 시스템이 공통적으로 겪는 실패 사례들을 심도 있게 분석한다.


- **계산 비용 대 실시간 성능:** 저자들이 인정한 가장 주된 한계는 동적 객체 제거 모듈로 인한 계산 부하 증가다.10
  - **병목 현상:** 매우 동적이고 규모가 큰 `ULCA-MarketStreet` 데이터셋에서 ID-LIO의 스캔 당 평균 처리 시간은 **120.6 ms**로 측정되었다. 이는 10 Hz LiDAR의 일반적인 실시간 처리 기준인 100 ms를 초과하는 수치다.10
  - **원인:** 논문은 이 문제의 원인을 매 프레임마다 스캔-투-맵 정합을 위한 로컬 맵과 동적 포인트 탐지를 위한 의사 점유 맵(pseudo occupancy map)을 반복적으로 구성하는 시간 소모적인 과정 때문이라고 분석한다.10 이는 동적 환경에 대한 강인성을 얻는 대가로 정량화 가능한 계산 비용을 지불해야 하는 직접적인 트레이드오프 관계를 보여준다.
- **Z축 드리프트 (Z-axis Drift):** 논문은 LIO 시스템이 일반적으로 넓고 평탄한 시나리오에서 Z축(높이) 방향으로 드리프트가 발생하는 경향이 있다고 지적한다. 이는 ID-LIO만의 고유한 실패 사례는 아니지만, 저자들은 향후 연구 과제로 지면 제약 조건(ground constraint)을 추가하여 이 문제를 완화할 계획을 언급했다.10 이는 단순히 동적 객체 처리 문제를 넘어, LIO 시스템의 더 넓은 범위의 과제에 대한 인식을 보여준다.


ID-LIO 역시 다른 LiDAR 기반 SLAM 시스템과 마찬가지로 특정 환경 조건에서는 성능 저하 또는 실패를 겪을 수 있다.

- **기하학적 퇴화 / 특징 부족 환경 (Geometric Degeneracy / Feature-Scarce Environments):** 이는 모든 기하 구조 기반 SLAM의 근본적인 난제다.
  - **시나리오:** 길고 유사한 구조가 반복되는 복도, 터널, 개활지, 넓고 텅 빈 홀 등.14
  - **문제점:** 이러한 환경에서는 로봇의 포즈를 유일하게 결정할 만큼 충분히 독특한 기하학적 특징(모서리, 평면)이 부족하다. 스캔 정합 문제는 모호해지고, 제약되지 않은 방향(예: 복도를 따라가는 방향)으로 추정된 포즈가 '미끄러지면서' 빠르고 심각한 드리프트를 유발한다. ID-LIO의 동적 객체 제거 기능은 이 문제에 도움이 되지 않는다. 문제의 원인이 동적 특징의 과잉이 아니라 정적 특징의 부족이기 때문이다.
- **지각적 별칭 (Perceptual Aliasing):** 로봇이 '데자뷰'를 경험하는 문제다.
  - **시나리오:** 동일한 구조가 반복되는 환경, 예를 들어 똑같이 생긴 층으로 구성된 주차장, 비슷한 모양의 사무실, 유사하게 생긴 교차로 등.8
  - **문제점:** 시스템이 현재 위치를 이전에 방문했던 다른 장소로 잘못 인식하여 거짓 루프 클로저를 생성한다. 이는 팩터 그래프에 잘못된 제약 조건을 추가하여 맵 전체를 왜곡시키고 궤적을 완전히 망가뜨리는 치명적인 오류다. ID-LIO의 동적 필터링이 도움은 되지만, 정적이고 반복적인 기하 구조로 인해 발생하는 지각적 별칭 문제 자체를 해결할 수는 없다.
- **센서 레벨 실패:** 알고리즘의 성능은 입력 데이터의 품질을 넘어설 수 없다.
  - **반사 및 투명 표면:** 유리벽, 거울, 수면 등은 LiDAR 빔을 예측 불가능하게 반사시키거나 그대로 통과시켜 부정확하거나 누락된 깊이 측정값을 생성한다.18
  - **환경적 방해물:** 짙은 먼지, 안개, 비, 눈 등은 레이저 빔을 흡수하거나 산란시켜 포인트 클라우드의 품질을 저하시킬 수 있다.20
- **격렬한 움직임 및 작은 중첩 영역:** 매우 빠르거나 급격한 움직임은 연속된 스캔 간의 중첩 영역을 불충분하게 만들어 ICP와 같은 정합 알고리즘이 올바르게 수렴하기 어렵게 만든다. 이는 모든 LIO 시스템의 과제이지만, IMU와의 Tightly-Coupled 융합이 이 문제를 완화하는 데 크게 기여한다.21

현대 LIO 시스템들 사이에는 **(1) 동적 환경 강인성**, **(2) 실시간 성능**, 그리고 **(3) 단순성/일반성**이라는 세 가지 요소 사이에 근본적이고 피할 수 없는 트레이드오프 관계가 존재한다. 알고리즘은 일반적으로 이 세 가지 중 두 가지에서 뛰어난 성능을 보이는 대신 나머지 하나를 희생하는 경향이 있다.

1. **FAST-LIO2** 22는 

   **성능**과 **단순성**을 우선시한다. 매우 효율적인 ikd-Tree 자료구조와 EKF 기반의 타이트한 융합을 사용한다. 이는 믿을 수 없을 정도로 빠르지만, 전역 최적화 백엔드와 정교한 동적 필터링이 부족하여 대규모 동적 환경에서는 강인성이 떨어진다.10

2. **LIO-SAM** 4은 

   **성능**과 **(정적 환경에서의) 강인성** 사이의 균형을 맞춘다. 이중 그래프 아키텍처와 루프 클로저는 전역 정확도를 제공하지만, FAST-LIO2보다 복잡하며 동적 환경에서는 성능이 저하된다.10

3. **ID-LIO** 1는 

   **동적 환경 강인성**에 매우 큰 비중을 둔다. 이를 위해 복잡하고 계산 비용이 높은 모듈을 추가했다. 이는 목표 환경에서는 월등한 정확도를 보여주지만, 가장 극단적인 경우(120.6 ms 처리 시간)에는 **성능**을 희생하며 LIO-SAM보다 훨씬 더 높은 복잡성을 가진다.

결론적으로, SLAM 시스템을 선택하는 사용자와 개발자는 이 트레이드오프를 반드시 이해해야 한다. 통제된 환경의 물류 창고 로봇에게는 FAST-LIO2가 최적일 수 있다. 정적인 숲을 측량하는 드론에게는 LIO-SAM이 훌륭한 선택이다. 그러나 혼잡한 도심의 자율주행 차량에게는 ID-LIO의 추가적인 계산 비용과 복잡성이 동적 장애물에 대한 강인성이라는 필수적인 요구사항에 의해 정당화될 수 있다. 모든 상황에 '최고'인 단일 알고리즘은 없으며, 주어진 응용 분야와 그 제약 조건에 '가장 적합한' 알고리즘만 있을 뿐이다.


이 마지막 섹션에서는 앞선 분석들을 종합하여 ID-LIO가 LIO 분야에 기여한 바를 정리하고, 강인한 자율 주행 기술의 다음 연구 방향을 조망한다.


ID-LIO의 핵심적인 기여는 단순히 동적 포인트를 제거하는 것을 넘어, **시간적 증거에 기반한 새로운 프레임워크**를 통해 이를 수행했다는 점에 있다. '인덱싱된 포인트' 자료구조와 동적 관찰 횟수(DON)에 기반한 '지연된 제거' 전략의 조합은, 더 단순한 '선제적 제거(removal-first)' 방식(예: RF-LIO 5)이나 단일 프레임 필터링 방식에 비해 개념적으로 중요한 진보다.

이러한 접근법은 인간 중심의 실제 환경에서 SLAM을 적용할 때 가장 지속적으로 발생하는 문제 중 하나에 대한 강인하고 실용적인 해결책을 제공한다.12 도심의 혼잡함에 LIO 시스템이 더 잘 견디도록 만듦으로써, ID-LIO는 자율 배송 로봇 7이나 자율주행차 13와 같이 바로 그러한 환경에서 안정적으로 작동해야 하는 응용 분야의 가능성을 한 단계 끌어올렸다.


강인한 LIO 연구의 진화는 명확한 흐름을 보인다. 이는 **기하학에 대한 추론**(LOAM)에서 **시간에 따른 모션에 대한 추론**(ID-LIO)으로, 그리고 다음 단계인 **객체의 정체성과 맥락에 대한 추론**(Semantic SLAM)으로 나아가고 있다.

1. **1단계 (기하학):** LOAM과 그 파생 연구들은 기하학적 기본 요소(모서리, 평면)를 정확하게 정합하는 데 집중했다.1 이들의 이해는 "표면이 어디에 있는가?"에 국한되었다.
2. **2단계 (시간적 모션):** ID-LIO와 같은 시스템은 여기에 시간적 차원을 추가했다.1 이들의 이해는 "표면이 어디에 있으며, 그중 어떤 것들이 시간의 흐름에 따라 비일관적으로 행동했는가?"로 격상되었다. 이를 통해 정적 기하와 동적 기하를 구분할 수 있게 되었다.
3. **3단계 (의미론적 맥락):** 다음 연구의 물결인 Semantic SLAM은 정체성(identity)이라는 층위를 추가한다.26 이 시스템의 이해는 "나는 '자동차'로 식별되는 객체의 일부인 표면을 보고 있다. 자동차는 움직일 수 있으므로, 현재 정지해 있더라도 이 객체는 주의해서 다루어야 한다. 저 다른 표면은 '건물'이며, 이는 신뢰할 수 있는 정적 객체다."와 같을 것이다.

이러한 발전은 인간이 세상을 인식하는 방식을 반영한다. 우리는 단지 형태만 보는 것이 아니라, 정체성과 예상되는 행동을 가진 객체를 본다. 의미론적 이해를 통합함으로써, 미래의 SLAM 시스템은 기하학적 불일치에 단순히 반응하는 수준을 넘어, 환경에 대한 더 깊은 이해를 바탕으로 능동적이고 예측적으로 대처하게 될 것이다. 이러한 관점에서 ID-LIO는 순수한 기하학적 패러다임과 미래의 의미론적 패러다임 사이를 잇는 매우 중요하고 효과적인 교량 역할을 수행한다.

이러한 흐름 속에서 구체적인 차세대 연구 방향은 다음과 같다.

- **Semantic SLAM:** 순수한 기하학적 필터링에서 의미론적 정보를 활용하는 필터링으로의 전환이 명백한 다음 단계다. 단순히 '예상치 못하게 움직이는 점'을 탐지하는 대신, 미래 시스템은 딥러닝을 사용하여 '자동차', '보행자', '자전거'와 같은 객체를 식별하고, 각 객체에 맞는 모션 모델이나 사전 확률을 적용할 것이다.26 이는 동적 객체에 대한 훨씬 더 지능적이고 선제적인 대응을 가능하게 한다.
- **심화된 다중 센서 융합 (LIVO, LIV-SLAM):** LiDAR, IMU, 그리고 시각(Visual) 데이터를 융합하는 LIVO/LIV-SLAM은 주요 연구 동력이다.30 카메라는 LiDAR의 기하학적 정보와는 완전히 직교하는 풍부한 텍스처와 의미론적 정보를 제공한다. 이 융합은 기하학적 퇴화 문제(예: 긴 복도의 텍스처가 있는 벽)를 해결하는 데 도움을 주며, 동적 객체를 탐지하고 추적하는 또 다른 수단을 제공한다.
- **평생 학습 및 협력 SLAM (Lifelong and Collaborative SLAM):** 현재의 SLAM 시스템은 한 번 맵을 만들면 끝이다. 평생 학습 SLAM은 장기간에 걸쳐 지속적으로 맵을 업데이트하며 영구적 또는 반영구적 변화(예: 새로운 건물, 계절에 따른 식생 변화)에 적응하는 것을 목표로 한다.32 이는 일시적인 동적 객체를 필터링하는 것을 넘어선 개념이다. 다중 로봇 SLAM은 여러 에이전트의 맵을 효율적으로 병합하여 영역을 더 빠르게 매핑하는 데 중점을 둔다.25
- **End-to-End 학습 및 신경망 표현:** 전통적인 SLAM 파이프라인의 일부(특징 추출, 심지어 맵 표현 자체)를 학습된 신경망으로 대체하려는 연구가 증가하고 있다. NeRF (Neural Radiance Fields)와 같은 암시적 표현(implicit representation)은 고품질의 사실적인 맵을 생성할 가능성을 보여주지만, 현재로서는 계산 비용이 매우 높다.33
- **일반성 향상 및 자동 튜닝:** LIO-SAM이나 ID-LIO와 같은 시스템에서 수많은 파라미터를 수동으로 조정해야 하는 것은 실제 적용의 큰 장벽이다.34 향후 연구는 파라미터를 자동으로 조정하거나, 파라미터에 본질적으로 덜 민감한 방법을 사용하여 더 다양한 센서와 환경에서 강인하게 작동하는 적응형 알고리즘을 만드는 데 집중될 것이다.35


1. LiDAR Inertial Odometry Based on Indexed Point and Delayed ..., accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10255994/
2. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in High Dynamic Environments - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/370148950_LiDAR_Inertial_Odometry_Based_on_Indexed_Point_and_Delayed_Removal_Strategy_in_High_Dynamic_Environments
3. (PDF) LIO-SAM: Tightly-coupled Lidar Inertial Odometry via ..., accessed July 23, 2025, https://www.researchgate.net/publication/362412728_LIO-SAM_Tightly-coupled_Lidar_Inertial_Odometry_via_Smoothing_and_Mapping
4. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2007.00258
5. A Fast Dynamic Point Detection Method for LiDAR-Inertial Odometry in Driving Scenarios, accessed July 23, 2025, https://arxiv.org/html/2407.03590v1
6. Factor Graphs for Navigation Applications: A Tutorial, accessed July 23, 2025, https://navi.ion.org/content/71/3/navi.653
7. [2209.10047] Uncertainty-Aware Tightly-Coupled GPS Fused LIO-SLAM - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2209.10047
8. VS-SLAM: Robust SLAM Based on LiDAR Loop Closure Detection with Virtual Descriptors and Selective Memory Storage in Challenging Environments - MDPI, accessed July 23, 2025, https://www.mdpi.com/2076-0825/14/3/132
9. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 23, 2025, https://github.com/TixiaoShan/LIO-SAM
10. LiDAR Inertial Odometry Based on Indexed Point and Delayed ..., accessed July 23, 2025, https://www.mdpi.com/1424-8220/23/11/5188
11. [2206.09463] RF-LIO: Removal-First Tightly-coupled Lidar Inertial Odometry in High Dynamic Environments - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2206.09463
12. Research on Mobile Robot SLAM in Dynamic Environment - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/252677693_Research_on_Mobile_Robot_SLAM_in_Dynamic_Environment
13. Project Overview - SLAM in Dynamic Environments, accessed July 23, 2025, https://mscvprojects.ri.cmu.edu/2019teama/
14. (PDF) LiDAR-IMU Tightly-Coupled SLAM Method Based on IEKF and Loop Closure Detection - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/378109927_LiDAR-IMU_Tightly-Coupled_SLAM_Method_Based_on_IEKF_and_Loop_Closure_Detection
15. LB-LIOSAM: an improved mapping and localization method with loop detection | Emerald Insight, accessed July 23, 2025, https://www.emerald.com/insight/content/doi/10.1108/ir-07-2024-0314/full/html
16. Graph-based LiDAR-Inertial SLAM Enhanced by Loosely-Coupled Visual Odometry - Computation Robotics Laboratory, accessed July 23, 2025, http://comrob.fel.cvut.cz/papers/ecmr23nav.pdf
17. Present and Future of SLAM in Extreme Underground Environments - JPL Robotics, accessed July 23, 2025, https://www-robotics.jpl.nasa.gov/media/documents/2208.01787.pdf
18. Understanding why SLAM algorithms fail in modern indoor environments - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/370633445_Understanding_why_SLAM_algorithms_fail_in_modern_indoor_environments
19. SLAM Lidar Scanners: Enhancing Mapping Accuracy - Foxtech Robotics, accessed July 23, 2025, https://www.foxtechrobotics.com/a-news-slam-lidar-scanners-enhancing-mapping-accuracy
20. Unlocking the Future of Autonomy: How Simultaneous Localization and Mapping (SLAM) Powers Next-Gen Robotics, accessed July 23, 2025, https://www.dronesplusrobotics.com/post/unlocking-the-future-of-autonomy-how-simultaneous-localization-and-mapping-slam-powers-next-gen-r
21. DMSA - Dense Multi Scan Adjustment for LiDAR Inertial Odometry and Global Optimization, accessed July 23, 2025, https://arxiv.org/html/2402.19044v1
22. hku-mars/FAST_LIO: A computationally efficient and robust LiDAR-inertial odometry (LIO) package - GitHub, accessed July 23, 2025, https://github.com/hku-mars/FAST_LIO
23. Visualization of trajectory comparison between A-LOAM, FAST-LIO, and... - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/figure/sualization-of-trajectory-comparison-between-A-LOAM-FAST-LIO-and-our-method-on-KITTI_fig5_378109927
24. Visual SLAM for Dynamic Environments Based on Object Detection and Optical Flow for Dynamic Object Removal - PMC - PubMed Central, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9571647/
25. The Future of Robotics: SLAM - Number Analytics, accessed July 23, 2025, https://www.numberanalytics.com/blog/the-future-of-robotics-slam
26. LIO-CSI: LiDAR inertial odometry with loop closure combined with semantic information, accessed July 23, 2025, https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0261053
27. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/386200068_LIO-SAM_A_Lidar-Inertial_Semantic_SLAM_with_Association_Optimization_and_Keyframe_Selection
28. DS-SLAM: A Semantic Visual SLAM towards Dynamic Environments - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/1809.08379
29. NGD-SLAM: Towards Real-Time SLAM for Dynamic Environments without GPU - arXiv, accessed July 23, 2025, https://arxiv.org/html/2405.07392v1
30. [2301.05604] A LiDAR-Inertial-Visual SLAM System with Loop Detection - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2301.05604
31. A Review of Research on SLAM Technology Based on the Fusion of LiDAR and Vision, accessed July 23, 2025, https://www.mdpi.com/1424-8220/25/5/1447
32. 3D LiDAR-IMU Lifelong SLAM with Relocalization and Autonomous Map Updating for Accurate and Reliable Navigation - Research Repository, accessed July 23, 2025, https://repository.essex.ac.uk/37954/1/my.pdf
33. KITTI Odometry Benchmark - Andreas Geiger, accessed July 23, 2025, https://www.cvlibs.net/datasets/kitti/eval_odometry.php
34. LiPO: LiDAR Inertial Odometry for ICP Comparison - arXiv, accessed July 23, 2025, https://arxiv.org/html/2410.08097v1
35. zijiechenrobotics/ig_lio: iG-LIO: An Incremental GICP-based Tightly-coupled LiDAR-inertial Odometry - GitHub, accessed July 23, 2025, https://github.com/zijiechenrobotics/ig_lio


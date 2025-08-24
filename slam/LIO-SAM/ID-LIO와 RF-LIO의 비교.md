[LIO-SAM](./index.md)
# ID-LIO와 RF-LIO의 비교



전통적인 동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM) 알고리즘은 대부분 환경이 정적(static)이라는 기본 가정하에 설계되었다.1 그러나 도심과 같은 실제 환경은 보행자, 차량 등 수많은 동적 객체로 가득 차 있으며, 이로 인해 정적 세계 가정은 쉽게 깨진다.3 동적 객체의 존재는 LiDAR SLAM 시스템에 심각한 문제를 야기한다. 움직이는 객체에서 반사된 포인트 클라우드는 프레임 간 정합(registration) 과정에서 잘못된 기하학적 제약 조건을 생성하여, 결과적으로 부정확한 위치 추정(pose estimation)과 동적 객체의 궤적이 그대로 남아있는 오염된 지도(polluted map)를 생성하게 된다.2 따라서 강건한(robust) 자율주행 및 로봇 내비게이션 기술을 구현하기 위해서는 동적 객체를 효과적으로 탐지하고 그 영향을 최소화하는 고도화된 LiDAR-Inertial Odometry (LIO) 시스템이 필수적이다.1


본고에서 비교 분석할 ID-LIO와 RF-LIO는 모두 LIO-SAM(Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping)이라는 강력한 프레임워크를 기반으로 개발되었다.1 LIO-SAM은 요인 그래프(Factor Graph) 최적화 기법을 사용하여 LiDAR와 IMU(Inertial Measurement Unit) 센서 데이터를 강결합(Tightly-coupled) 방식으로 융합하는 것이 특징이다.10

요인 그래프는 시스템의 상태(로봇의 위치, 속도, IMU 바이어스 등)와 측정값 사이의 확률적 관계를 그래프 구조로 표현한 것이다. LIO-SAM의 요인 그래프는 LiDAR odometry 요인, IMU pre-integration 요인, 그리고 선택적으로 GPS 요인과 루프 클로저(loop closure) 요인을 통합하여 전역적으로 일관된 최적의 해를 찾는다.10 특히 IMU pre-integration은 높은 주기로 수집되는 IMU 측정값을 적분하여 LiDAR 스캔 간의 상대적인 움직임을 예측한다. 이는 LiDAR 포인트 클라우드에서 발생하는 모션 왜곡(motion distortion)을 효과적으로 보정하고, 계산 비용이 높은 LiDAR odometry 최적화를 위한 정확한 초기 추정값(initial guess)을 제공하는 핵심적인 역할을 수행한다.10

LIO-SAM은 그 자체로 매우 뛰어난 성능을 보이지만, 동적 객체에 대한 명시적인 처리 메커니즘은 부재하다. 요인 그래프에 입력되는 LiDAR odometry 요인이 동적 객체로 인해 오염될 경우, 아무리 정교한 최적화 과정을 거치더라도 최종 결과의 정확성은 저하될 수밖에 없다. 즉, 입력 데이터의 품질이 전체 시스템의 성능을 결정하는 근본적인 한계를 내포하고 있다. ID-LIO와 RF-LIO는 바로 이 '오염된 입력' 문제를 해결하기 위해 등장했으며, 이는 단순한 기능 추가를 넘어 LIO-SAM의 근본적인 한계를 극복하려는 두 가지 상이한 철학적 접근법의 결과물로 이해해야 한다.



RF-LIO(Removal-First Tightly-coupled Lidar Inertial Odometry)의 핵심 철학은 이름에서 알 수 있듯이, 위치 추정을 위한 포인트 클라우드 정합 및 최적화 이전에 동적 객체를 '먼저 제거'하는 'Removal-First' 전략에 있다.1 이 접근법은 최적화 단계에 입력되는 데이터 자체를 사전에 정제함으로써, 동적 객체로 인해 발생하는 오차 요인이 요인 그래프에 포함되는 것을 원천적으로 차단한다. 이는 전체 SLAM 파이프라인을 개념적으로 명확하고 직관적으로 유지하는 장점을 가진다.


RF-LIO의 전체 파이프라인은 LIO-SAM을 기반으로 구성되지만, 프론트엔드 단에 동적 객체 제거 모듈이 명시적으로 추가된 형태를 띤다.1 시스템의 데이터 흐름은 다음과 같다.

1. **동적 객체 제거**: 입력된 원본 LiDAR 스캔에 대해 동적 객체 제거 모듈을 적용하여 정적 포인트(static points)로만 구성된 깨끗한 포인트 클라우드를 생성한다.
2. **특징 추출 및 Odometry 계산**: 정제된 정적 포인트 클라우드를 LIO-SAM의 표준 파이프라인에 입력하여 특징점(edge/planar features)을 추출하고, 이를 기반으로 LiDAR odometry를 계산한다.
3. **요인 그래프 최적화**: 계산된 (깨끗한) LiDAR odometry 요인을 IMU pre-integration 요인 및 기타 요인들과 함께 요인 그래프 상에서 최적화하여 최종적인 로봇의 상태를 추정한다.


RF-LIO는 동적 객체 제거를 위해 'Removert' (Remove-then-Revert)라는 알고리즘을 채택했다.13 Removert는 IROS 2020 컨퍼런스에서 발표된 연구로, 다중 해상도 Range Image를 사용하는 가시성 기반(visibility-based) 접근법이다.14 이 방법의 핵심은 현재 스캔과 이전에 구축된 맵 포인트 간의 가시성 차이를 이용하여 동적 객체를 탐지하는 것이다.4


Removert는 현재 쿼리 스캔($P^Q_k$)과 로컬 맵($P^M$)을 동일한 시점의 구면 좌표계(spherical coordinate system)로 투영하여 두 개의 Range Image, $I^Q_k$와 $I^M_k$를 생성한다. 각 이미지의 픽셀은 해당 방향에서 관측된 가장 가까운 포인트의 거리(range) 값을 저장한다.15 수학적으로, 픽셀 $(i, j)$에 해당하는 지도 Range Image 값 $I^M_{k,ij}$는 다음과 같이 정의된다 15:
$$
I^M_{k,ij} = \min_{p \in P^M_{ij}} r(p)
$$
여기서 $r(\cdot)$은 포인트의 거리 값을, $P^M_{ij}$는 픽셀 $(i, j)$에 투영되는 지도 포인트들의 집합을 의미한다. 동일한 방식으로 쿼리 스캔의 Range Image $I^Q_k$도 생성된다.

두 Range Image 간의 픽셀별 차이($I_{Diff}^k = I^Q_k - I^M_k$)를 계산함으로써 불일치를 탐지한다. 만약 특정 픽셀 $(i,j)$에서 $I_{Diff}^k(i,j)$ 값이 양의 임계값($\tau_D$)보다 크다면, 이는 현재 스캔의 포인트가 이전에 관측된 지도 포인트보다 '뒤에' 있다는 것을 의미한다. 즉, 이전에 그 자리를 가리고 있던 지도 포인트($p^M_{k,ij}$)가 현재는 사라졌음을 시사하므로, 해당 지도 포인트를 동적으로 판단하는 것이다.15
$$
P^D_k = \{p^M_{k,ij} | I_{Diff}^k(i,j) > \tau_D\}
$$


가시성 기반 방법은 LiDAR의 위치 추정(등록) 오차에 매우 민감하다는 본질적인 약점을 가진다.15 로봇의 자세가 약간만 틀어져도, 실제로는 정적인 벽면 포인트가 이전에 관측된 위치와 달라져 동적으로 오인될 수 있다. 'Removert'는 이 문제를 정면으로 다루기 위해 독창적인 2단계 메커니즘을 제안한다.

1. **Remove (제거 단계)**: 처음에는 가장 높은 해상도(finest resolution)의 Range Image를 사용하여 매우 보수적으로(conservatively) 동적 포인트를 식별하고 제거한다. 이 단계에서는 위치 추정 오차로 인해 일부 정적 포인트가 동적으로 잘못 분류될 수 있다 (False Positives).15
2. **Revert (복원 단계)**: 그 후, 점차적으로 낮은 해상도(coarser resolution)의 Range Image를 사용하여 이전에 동적으로 분류되었던 포인트들을 재검토한다. 낮은 해상도의 픽셀은 더 넓은 각도 범위를 포괄하므로, 작은 위치 추정 오차는 같은 픽셀 내에 흡수될 가능성이 커진다. 이로 인해, 1단계에서 위치 오차 때문에 동적으로 잘못 제거되었던 정적 포인트들이 다시 '정적'으로 올바르게 판단되어 복원(revert)될 기회를 얻는다. 이는 위치 추정의 불확실성을 암묵적으로 보상하는 매우 효과적인 전략이다.15

이 과정은 단일 스캔의 폐색(occlusion)이나 제한된 시야(FOV) 문제를 극복하기 위해, 여러 스캔에 걸쳐 각 맵 포인트가 '정적' 또는 '동적'으로 관측된 횟수를 누적하여 투표(voting)하는 방식으로 확장된다. 이를 통해 더욱 강건하게 동적 객체를 제거하고 깨끗한 정적 맵을 구축할 수 있다.15



ID-LIO(LiDAR inertial odometry-based on indexed point and delayed removal strategy)는 RF-LIO와는 다른 철학을 가진다. 동적 객체 정보를 별도의 전처리 단계에서 완전히 분리하는 대신, 메인 최적화 프레임워크 내에서 직접 관리하고 통합한다.9 이 접근법은 동적인지 정적인지 여부가 불분명한 '회색 지대'의 정보를 무조건 버리지 않고, 그 불확실성을 가중치(weight) 형태로 최적화 과정에 반영함으로써 가용 정보를 최대한 활용하려는 시도이다.


ID-LIO는 LIO-SAM의 슬라이딩 윈도우(sliding window) 기반 최적화 구조를 근간으로 하되, 동적 객체를 처리하기 위한 여러 모듈이 파이프라인 전반에 걸쳐 유기적으로 결합되어 있다.9 이는 동적 객체 처리가 단순한 전처리 과정이 아니라, 상태 추정 및 맵 관리와 긴밀하게 연결된 통합 프로세스임을 의미한다.



ID-LIO는 현재 스캔에서 동적일 가능성이 있는 포인트를 1차적으로 탐지하기 위해 공간적 차원에서의 '유사 점유(pseudo occupancy)' 방법을 사용한다.11 이는 특정 공간(예: 복셀 또는 그리드)이 이전 프레임들에서는 비어있었는데 현재 스캔에서 점유되거나, 반대로 이전에는 점유되었는데 현재는 비어있는지를 비교하는 원리에 기반한다. 이는 Ray-tracing 기반 방법과 유사한 아이디어지만, 실시간성을 확보하기 위해 더 단순화된 모델을 사용할 것으로 추정된다.5


'인덱스 포인트'는 ID-LIO의 가장 핵심적인 개념이다. 이는 한번 동적으로 탐지된 포인트의 '상태'를 시간적으로 전파하고 추적하기 위한 메커니즘이다.9 RF-LIO가 매 프레임마다 독립적으로 동적 객체를 '탐지'하는 데 초점을 맞춘다면, ID-LIO는 한 걸음 더 나아가 객체의 동적 '상태'를 시간 축으로 '추적'한다. 예를 들어, 프레임 

$k$에서 특정 차량에 속한 포인트들이 동적으로 식별되면, 이 포인트들은 '동적'이라는 레이블과 함께 고유한 인덱스를 부여받는다. 다음 프레임 $k+1$에서 이 포인트들이 다시 관측될 경우, 시스템은 이들의 인덱스를 통해 과거 이력을 즉시 인지하고 이 정보를 최적화에 활용한다. 이러한 접근법은 신호 대기 등으로 일시적으로 멈춘 차량(외견상 정적이지만 잠재적으로는 동적인 객체)을 처리하는 데 있어, 매 프레임을 독립적으로 판단하는 방식보다 훨씬 유리하다.


ID-LIO는 탐지된 동적 포인트를 단순히 제거하는 대신, 슬라이딩 윈도우 기반 최적화 과정에서 해당 포인트들이 생성하는 측정 오차(measurement error)에 낮은 가중치(weight)를 부여하여 그 영향을 줄인다.13 LIO-SAM의 기본 비용 함수(cost function)가 각 오차항의 제곱합($E = \sum ||r_i||^2_{\Sigma_i}$)이라면, ID-LIO는 동적 포인트에서 발생하는 오차항에 동적 가중치 $w_d$ ($0 \le w_d < 1$)를 곱하여 그 영향을 조절한다.
$$
E_{ID-LIO} = \sum_{p_j \in \text{Static}} ||r_j||^2_{\Sigma_j} + \sum_{p_k \in \text{Dynamic}} w_d \cdot ||r_k||^2_{\Sigma_k}
$$
여기서 가중치 $w_d$는 해당 포인트가 동적일 확률에 따라 결정될 수 있다. 이는 정보를 완전히 버리지 않는 '소프트(soft)' 결정 방식으로, 불확실성을 자연스럽게 최적화에 통합하는 방법이다.


로컬 맵(local map)을 구성하는 핵심 프레임인 키프레임(keyframe)에서 동적 포인트를 처리할 때, ID-LIO는 즉시 제거하지 않고 '지연된 제거(Delayed Removal)' 전략을 사용한다.9 키프레임의 포인트들은 미래의 위치 추정을 위한 중요한 기준점이 되므로, 한 프레임에서 어떤 포인트가 동적으로 오인되어 즉시 제거된다면 이는 영구적인 정보 손실로 이어진다. '지연된 제거' 전략은 이러한 성급한 결정을 방지하는 안전장치 역할을 한다. 특정 포인트가 여러 프레임에 걸쳐 일관되게 동적으로 판단될 때만 맵에서 최종적으로 제거함으로써, 일시적인 오탐지(false positive)로 인해 중요한 정적 환경 정보가 유실되는 것을 막고 맵의 장기적인 일관성을 향상시킨다.



RF-LIO와 ID-LIO는 동적 객체 문제에 대해 근본적으로 다른 패러다임을 채택한다.

- **RF-LIO (선제적 필터링)**: 파이프라인이 직관적이고 모듈화되어 있다. 동적 객체 제거 모듈의 성능이 전체 시스템의 성능을 직접적으로 좌우한다. 동적으로 판단된 포인트는 후속 단계에서 전혀 고려되지 않는 '하드(hard)' 결정 방식을 취한다.
- **ID-LIO (통합적 관리)**: 더 복잡하지만 유기적인 시스템이다. 동적 여부의 불확실성을 최적화 과정에 '소프트(soft)'하게 반영하여 정보를 최대한 보존하려 한다. 잠재적으로 더 많은 정보를 활용하지만, 최적화 문제 자체가 더 복잡해지고 관리해야 할 상태 변수가 늘어난다.


두 시스템의 탐지 메커니즘은 각각의 철학을 반영하며 뚜렷한 차이를 보인다.

- **RF-LIO (가시성 기반)**: 기하학적 원리에 기반하여 직관적이고 별도의 학습 데이터가 필요 없다. 하지만 폐색(occlusion) 상황에 취약할 수 있다. 예를 들어, 거대한 트럭이 뒤쪽의 건물을 완전히 가릴 경우, 트럭을 동적으로 판단할 가시성 기반의 근거가 사라져 탐지에 실패할 수 있다.4 또한 위치 추정 오차에 민감하지만, 'Revert' 메커니즘을 통해 이를 완화하려 시도한다.15
- **ID-LIO (Pseudo-Occupancy + Temporal)**: 공간적 불일치와 시간적 추적을 결합한 하이브리드 방식이다. 특히 '인덱스 포인트'를 통한 시간적 정보 활용은 이 시스템의 가장 큰 차별점이다. 이는 일시적으로 정지한 동적 객체(예: 신호 대기 중인 차량)나 간헐적으로 나타나는 동적 객체를 처리하는 데 더 강건한 성능을 보일 수 있다.


맵 관리 전략과 계산 효율성 측면에서도 차이가 존재한다.

- **RF-LIO**: 정제된 포인트만 맵에 추가되므로, 동적 객체 제거 모듈이 정확하게 작동한다면 매우 깨끗한 정적 맵을 실시간으로 생성할 수 있다. 계산 부하는 Removert 알고리즘의 효율성에 크게 의존하며, 본래 오프라인 후처리 방식이었던 Removert를 실시간 LIO 시스템에 맞게 최적화한 것이 핵심 기술 중 하나이다.14
- **ID-LIO**: '지연된 제거' 전략 덕분에 성급한 포인트 제거를 방지하여 장기적인 맵의 일관성을 유지하는 데 유리하다. 반면, 모든 포인트의 동적 상태와 인덱스를 지속적으로 추적하고 관리해야 하므로 추가적인 메모리 및 계산 오버헤드가 발생할 수 있다.


두 시스템의 장단점은 특정 시나리오에서 더욱 명확하게 드러난다.

- **시나리오 1: 혼잡한 교차로 (다수의 동적 객체)**
  - RF-LIO는 다수의 객체를 효과적으로 사전 제거하면 높은 성능을 보일 것이다. 하지만 객체 간 폐색이 심하게 발생하면 탐지 성능이 저하될 수 있다.
  - ID-LIO는 가중치 기반 접근법과 시간적 추적을 통해 일부 탐지 실패가 있더라도 전체적인 위치 추정의 안정성을 유지할 가능성이 높다.
- **시나리오 2: 잠시 멈췄다 출발하는 버스**
  - RF-LIO는 버스가 멈춰있는 동안 이를 정적 객체로 오인하여 맵에 포함시킬 수 있다. 이후 버스가 움직이면, 맵에 남은 잔상(ghost)이 위치 추정 오차를 유발하는 원인이 될 수 있다.
  - ID-LIO는 '인덱스 포인트' 덕분에 버스가 이전에 동적이었던 이력을 기억하고 있을 가능성이 높다. 따라서 버스가 멈춰있더라도 낮은 가중치를 부여하여 그 영향을 최소화함으로써 더 강건하게 대처할 수 있다.


두 시스템은 모두 공개 데이터셋인 UrbanLoco와 UrbanNav에서 성능을 평가했으며, 베이스라인인 LIO-SAM 대비 상당한 정확도 향상을 보고했다.1 특히 동적 객체가 많은 도심 환경 데이터셋에서의 성능 비교는 두 시스템의 실효성을 가늠하는 중요한 척도이다.

아래 표는 주요 동적 데이터셋에서의 절대 궤적 오차(Absolute Trajectory Error, ATE)를 비교한 것이다.

| 알고리즘    | UrbanLoco-CAMarketStreet (ATE RMSE [m]) | UrbanNav-HK-Medium-Urban-1 (ATE RMSE [m]) | 비고                                                |
| ----------- | --------------------------------------- | ----------------------------------------- | --------------------------------------------------- |
| LIO-SAM     | Baseline                                | Baseline                                  | 동적 객체 처리 기능 부재                            |
| RF-LIO      | LIO-SAM 대비 ~70% 향상 1                | LIO-SAM 대비 ~70% 향상 1                  | 다른 연구에서 ID-LIO보다 오차가 큰 것으로 나타남 21 |
| ID-LIO      | LIO-SAM 대비 67% 향상 9                 | LIO-SAM 대비 85% 향상 9                   | 다른 연구에서 RF-LIO보다 우수한 성능으로 언급됨 21  |
| Dynamic-LIO | -                                       | -                                         | RF-LIO와 ID-LIO를 능가한다고 주장 21                |

정량적 수치상으로는 ID-LIO가 RF-LIO보다 약간 더 나은 성능을 보이는 경향이 있다. 특히 UrbanNav 데이터셋에서 85%라는 높은 향상률을 기록한 점이 주목할 만하다.9 한편, 'Dynamic-LIO'라는 또 다른 시스템이 RF-LIO와 ID-LIO를 모두 능가한다고 주장하는 연구21도 존재하는데, 이는 동적 환경 LIO 분야의 연구가 여전히 활발하게 진행 중이며 두 시스템 모두 개선의 여지가 있음을 시사한다.



ID-LIO와 RF-LIO는 동적 환경이라는 동일한 문제에 대해 서로 다른 철학적 해법을 제시하는 대표적인 LIO 시스템이다.

- **RF-LIO**는 '선-제거'라는 명확하고 모듈화된 접근법을 사용한다. 파이프라인이 직관적이지만, 성능이 전적으로 동적 객체 제거 모듈의 정확성에 의존하는 구조이다. 위치 추정 오차에 대한 강건성을 확보하기 위해 'Revert'라는 독창적인 메커니즘을 도입했다.
- **ID-LIO**는 최적화 과정에 동적 정보를 통합하는 복합적이고 유기적인 접근법을 취한다. 시간적 일관성을 고려하고 불확실성을 가중치로 다루는 데 강점이 있어, 복잡하고 애매한 동적 시나리오에 더 강건할 수 있다. 다만 시스템의 전체적인 복잡도는 더 높다.


어떤 시스템을 선택할지는 적용 분야와 연구 목적에 따라 달라질 수 있다.

- **RF-LIO 추천 시나리오**:
  - 시스템의 각 모듈을 독립적으로 분석하고 개선하는 등, 시스템의 모듈성과 해석 용이성이 중요할 때.
  - 동적 객체가 명확하게 구분되고 객체 간 폐색이 비교적 적은 환경.
  - 강력한 동적 객체 제거 모듈을 별도로 개발하거나 튜닝할 수 있는 리소스가 있을 때.
- **ID-LIO 추천 시나리오**:
  - 일시적으로 정지하는 객체 등 애매한 상태의 동적 객체가 빈번하게 발생하는 매우 복잡한 도심 환경.
  - 최적화 프레임워크 내에서 모든 정보를 최대한 활용하여 시스템의 강건성을 극대화하는 것이 최우선 목표일 때.
  - 증가하는 시스템의 복잡도를 감수할 수 있을 때.


두 시스템은 동적 환경 LIO 연구에 중요한 이정표를 제시했지만, 여전히 발전의 여지가 많다. 향후 연구는 다음과 같은 방향으로 진행될 수 있다.

- **하이브리드 접근법**: 두 시스템의 장점을 결합하는 연구가 유망하다. 예를 들어, RF-LIO의 Removert와 같은 강력한 기하학 기반 탐지기를 사용하여 동적 객체 후보군을 생성하고, ID-LIO의 가중치 기반 최적화와 시간적 추적 메커니즘으로 최종 결정을 내리는 하이브리드 방식은 두 접근법의 장점을 모두 취할 수 있다.
- **딥러닝 기반 의미론적 정보 융합**: 딥러닝 기반의 의미론적 분할(semantic segmentation) 기술을 각 파이프라인에 통합하여 '자동차가 동적일 가능성이 높다'와 같은 사전 지식을 활용하면 탐지 정확도를 획기적으로 높일 수 있다.3 이는 기하학적 정보만으로는 해결하기 어려운 애매한 상황을 해결하는 데 큰 도움이 될 것이다.


1. [2206.09463] RF-LIO: Removal-First Tightly-coupled Lidar Inertial Odometry in High Dynamic Environments - arXiv, accessed July 30, 2025, https://arxiv.org/abs/2206.09463
2. Moving target removal for lidar SLAM - SPIE Digital Library, accessed July 30, 2025, https://www.spiedigitallibrary.org/conference-proceedings-of-spie/11763/117637B/Moving-target-removal-for-lidar-SLAM/10.1117/12.2587440.pdf
3. A Coarse-to-Fine LiDAR-Based SLAM with Dynamic Object Removal in Dense Urban Areas | PolyU, accessed July 30, 2025, [https://www.polyu.edu.hk/aae/ipn-lab/us/publications/Conference/A%20Coarse-to-Fine%20LiDAR-Based%20SLAM%20with%20Dynamic%20Object%20Removal%20in%20Dense%20Urban%20Areas.pdf](https://www.polyu.edu.hk/aae/ipn-lab/us/publications/Conference/A Coarse-to-Fine LiDAR-Based SLAM with Dynamic Object Removal in Dense Urban Areas.pdf)
4. LIDAR-based SLAM system for autonomous vehicles in degraded point cloud scenarios: dynamic obstacle removal | Emerald Insight, accessed July 30, 2025, https://www.emerald.com/insight/content/doi/10.1108/ir-01-2024-0001/full/pdf?title=lidar-based-slam-system-for-autonomous-vehicles-in-degraded-point-cloud-scenarios-dynamic-obstacle-removal
5. Lidar SLAM Algorithm Based on Online Point Cloud Removal in Pseudo Occupied Area, accessed July 30, 2025, https://www.researching.cn/articles/OJc3e8f940b2d19b39
6. Observation Time Difference: an Online Dynamic Objects Removal Method for Ground Vehicles - arXiv, accessed July 30, 2025, https://arxiv.org/html/2406.15774v1
7. A Review of Dynamic Object Filtering in SLAM Based on 3D LiDAR - PMC - PubMed Central, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10821332/
8. A Review of Dynamic Object Filtering in SLAM Based on 3D LiDAR - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/377547031_A_Review_of_Dynamic_Object_Filtering_in_SLAM_Based_on_3D_LiDAR
9. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in Highly Dynamic Environments - MDPI, accessed July 30, 2025, https://www.mdpi.com/1424-8220/23/11/5188
10. RF-LIO: Removal-First Tightly-coupled Lidar Inertial Odometry in High Dynamic Environments - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/361457103_RF-LIO_Removal-First_Tightly-coupled_Lidar_Inertial_Odometry_in_High_Dynamic_Environments
11. SR-LIO++: Efficient LiDAR-Inertial Odometry and Quantized Mapping with Sweep Reconstruction | Request PDF - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/390354696_SR-LIO_Efficient_LiDAR-Inertial_Odometry_and_Quantized_Mapping_with_Sweep_Reconstruction
12. LiDAR Odometry Survey: Recent Advancements and Remaining Challenges - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2312.17487
13. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in High Dynamic Environments - Preprints.org, accessed July 30, 2025, https://www.preprints.org/manuscript/202304.0527/v1
14. gisbi-kim/removert: Remove then revert (IROS 2020) - GitHub, accessed July 30, 2025, https://github.com/gisbi-kim/removert
15. Remove, Then Revert: Static Point Cloud Map Construction Using ..., accessed July 30, 2025, http://ras.papercept.net/images/temp/IROS/files/0855.pdf
16. ‪Giseop Kim‬ - ‪Google 학술 검색‬, accessed July 30, 2025, https://scholar.google.com/citations?user=9mKOLX8AAAAJ&hl=ko
17. Remove, then Revert: Static Point cloud Map Construction using Multiresolution Range Images - Giseop Kim, accessed July 30, 2025, https://gisbi-kim.github.io/publications/gkim-2020-iros.pdf
18. Mapping the Static Parts of Dynamic Scenes from 3D LiDAR Point Clouds Exploiting Ground Segmentation, accessed July 30, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/arora2021ecmr.pdf
19. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in High Dynamic Environments - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/370148950_LiDAR_Inertial_Odometry_Based_on_Indexed_Point_and_Delayed_Removal_Strategy_in_High_Dynamic_Environments
20. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in Highly Dynamic Environments - PMC, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10255994/
21. UrbanLoco: A Full Sensor Suite Dataset for Mapping and Localization in Urban Scenes | Request PDF - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/346897844_UrbanLoco_A_Full_Sensor_Suite_Dataset_for_Mapping_and_Localization_in_Urban_Scenes
22. LIO-CSI: LiDAR inertial odometry with loop closure combined with semantic information, accessed July 30, 2025, https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0261053


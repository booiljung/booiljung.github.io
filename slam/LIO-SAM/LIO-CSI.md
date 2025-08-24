# LIO-CSI
[LIO-SAM](./index.md)



많은 전통적인 동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM) 알고리즘의 근간을 이루는 '정적 세계 가정(static world assumption)'은 실제 로봇 공학 응용, 특히 자율 주행 분야에서는 더 이상 유효하지 않다.1 이 가정은 환경이 변하지 않는다고 전제하지만, 현실 세계는 차량, 보행자 등 예측 불가능한 동적 개체들로 가득 차 있다. 이러한 동적 개체들은 정적 세계 가정을 위반하며, SLAM 시스템에 잘못된 데이터 연관(data association)을 유발하여 상태 추정치와 지도를 모두 오염시킨다.1


정적 세계 가정이 깨졌을 때, LOAM(LiDAR Odometry and Mapping)이나 LeGO-LOAM과 같은 전통적인 LiDAR SLAM 시스템은 여러 가지 심각한 문제에 직면한다.

- **주행 거리계(Odometry) 드리프트:** 움직이는 객체에서 추출된 기하학적 특징점들은 스캔-지도 정합(scan-to-map registration) 과정에서 오류를 발생시켜 주행 거리계 오차를 급격히 누적시킨다.1
- **지도 오염 및 고스팅(Ghosting):** 동적 객체들이 전역 지도에 잘못 통합되면서, 환경의 정적 구조를 나타내지 않는 '유령' 또는 잔상을 생성한다. 이는 향후 위치 추정 및 항법 작업에서 지도의 신뢰성을 저하시킨다.1
- **루프 클로저(Loop Closure) 실패:** 로봇이 이전에 방문했던 장소를 다시 인식하는 루프 클로저 과정은 주로 기하학적 유사성에 의존한다(예: 유클리드 거리, 스캔 컨텍스트). 그러나 재방문 시점에 동적 객체의 위치가 바뀌면 장소의 기하학적 형태가 달라져 장소 인식이 실패한다. 이로 인해 전역 드리프트를 보정할 기회를 상실하고, 결국 전역적으로 일관되지 않은 지도를 생성하게 된다.1


이러한 근본적인 한계를 극복하기 위해, 시맨틱 정보를 SLAM에 통합하는 것은 단순한 점진적 개선이 아닌 필수적인 패러다임의 전환이다. 순수한 기하학적 표현에서 시맨틱 표현으로 전환함으로써, 로봇은 보다 인간과 유사한 방식으로 환경을 추론할 수 있게 된다.7 즉, 위치 추정에 신뢰할 수 있는 정적 구조물(건물, 도로)과 별도로 처리해야 하는 일시적인 객체(차량, 보행자)를 구별할 수 있게 되는 것이다.1 이는 시맨틱 정보가 모호한 기하학적 데이터를 올바르게 해석하는 데 필요한 '문맥'을 제공한다는 핵심 아이디어로 이어진다.

시맨틱 SLAM의 주된 동기는 단순히 인간이 읽기 쉬운 지도를 만드는 것을 넘어, 기저에 있는 상태 추정의 근본적인 강인성 문제를 해결하는 데 있다. 시맨틱 정보는 필터이자 사전 정보(prior) 역할을 하여, 기하학적 SLAM 알고리즘이 정제되고 신뢰성 높은 데이터 부분집합에 대해서만 작동하도록 만든다. 전통적인 SLAM은 움직이는 차량의 한 점과 정적인 건물의 한 점을 동일한 가중치로 취급하여 최적화 과정에서 오류를 유발한다. 초기에는 LeGO-LOAM처럼 기하학적 클러스터링을 통해 움직임을 분할하려는 시도가 있었으나, 일시적으로 정지하거나 느리게 움직이는 객체에는 효과적이지 않았다.5 반면, 시맨틱 분할은 '차량'이나 '보행자'와 같은 클래스 기반의 움직임 사전 정보를 제공한다. 시스템은 이러한 클래스의 객체들이 잠재적으로 동적일 수 있다고 판단하고, 정합 과정에서 이들을 제외하거나 신중하게 처리한다. 따라서 시맨틱은 단순히 지도에 '이것은 자동차'라는 라벨을 추가하는 것을 넘어, 안정적인 랜드마크와 불안정한 랜드마크를 식별하는 강력한 휴리스틱을 제공함으로써 데이터 연관 프로세스 자체를 근본적으로 변화시킨다. 이는 순수한 *기하학적 데이터 연관*에서 *시맨틱-기하학적 데이터 연관*으로의 전환을 의미한다.

이러한 실패는 국소적인 문제에 그치지 않고 시스템 전체로 파급된다. 동적 객체로 인한 단 한 번의 잘못된 데이터 연관은 프론트엔드에서 작은 오차를 유발한다. 이 작은 오차들이 시간이 지남에 따라 누적되어 상당한 드리프트를 야기하고, 결국 백엔드에서 유효한 루프 클로저를 찾는 것을 불가능하게 만든다. 즉, 문제는 국소적(프론트엔드 정합)인 오류에서 시작하여 전역적(백엔드 일관성)인 실패로 연쇄적으로 이어진다. LIO-CSI의 목표는 동적 객체가 오류를 유발하기 전, 이 연쇄 고리의 가장 첫 번째 단계에서부터 문제를 차단하는 것이다.



LIO-CSI는 LIO-SAM(LiDAR Inertial Odometry via Smoothing and Mapping)이라는 기하학 기반의 최첨단 프레임워크를 기반으로 확장된 시스템이다.1 따라서 LIO-CSI를 이해하기 위해서는 먼저 LIO-SAM의 아키텍처를 상세히 분석할 필요가 있다.


LIO-SAM의 핵심은 '강결합' 방식의 센서 융합에 있다. 이는 LiDAR와 IMU(관성 측정 장치)의 주행 거리계를 각각 독립적으로 추정한 뒤 결과를 합치는 약결합(loosely-coupled) 방식과 다르다. LIO-SAM은 원시 센서 측정값과 IMU 바이어스를 단일 최적화 프레임워크 내에서 함께 최적화한다.10 이를 통해 한 센서의 정보가 다른 센서의 상태 추정에 직접적으로 영향을 미쳐 더 높은 정확도와 강인성을 달성한다.


LIO-SAM의 핵심은 확률적 그래피컬 모델인 요인 그래프를 기반으로 문제를 공식화하는 것이다.

- **노드(Nodes):** 특정 시점에서의 로봇 상태(자세, 속도, IMU 바이어스)를 나타낸다.
- **요인(Factors):** 센서 측정값으로부터 파생된 노드 간의 확률적 제약 조건을 나타낸다. LIO-SAM의 주요 요인은 다음과 같다 10:
  - **IMU 사전 적분 요인(Pre-integration Factor):** 고주파 IMU 측정값으로부터 연속적인 로봇 상태 간의 제약 조건을 생성한다. 또한, 단일 스캔 동안 발생하는 움직임 왜곡을 보정하기 위해 LiDAR 포인트 클라우드의 왜곡을 제거(de-skewing)하는 데 사용된다.10
  - **LiDAR 주행 거리계 요인:** 연속적인 LiDAR 스캔과 지역 지도(local map) 사이의 특징점(엣지 및 평면)을 정합하여 얻은 제약 조건이다.
  - **GPS 요인 (선택 사항):** 지도를 고정하고 장기적인 드리프트를 방지하기 위한 절대 위치 제약 조건이다.
  - **루프 클로저 요인:** 로봇이 이전에 방문한 위치를 재방문했을 때, 연속적이지 않은 두 자세 사이에 생성되는 제약 조건으로, 누적된 드리프트를 보정하는 데 사용된다.


전체 궤적에 대한 요인 그래프는 시간이 지남에 따라 계산량이 기하급수적으로 증가하여 다루기 어려워진다. LIO-SAM은 실시간 성능을 유지하기 위해 다음과 같은 전략을 사용한다.

- **슬라이딩 윈도우 최적화:** 전체 궤적 대신, 고정된 크기의 최근 상태 윈도우만을 최적화한다.10
- **주변화(Marginalization):** 오래된 상태가 윈도우 밖으로 밀려날 때, 해당 상태가 가진 정보는 요약되어 나머지 상태에 대한 사전 확률(prior)로 변환된다. 이를 통해 과거 정보를 완전히 버리지 않고 유지할 수 있다.
- **키프레이밍(Keyframing):** 모든 LiDAR 스캔이 아닌, 일부 선택된 스캔(키프레임)만을 그래프에 포함시켜 계산 부하를 줄인다.10

LIO-SAM의 강점인 강결합 방식과 풍부한 기하학적 특징점 의존성은 동적 환경에서는 오히려 주요 약점으로 작용한다. LIO-SAM의 요인 그래프는 모든 제약 조건을 동시에 가장 잘 만족시키는 상태 궤적을 찾으려 한다. 이는 모든 요인이 정적이고 신뢰할 수 있는 특징점에서 파생될 때 매우 효과적으로 작동한다. 그러나 동적 객체가 존재하면, 이들은 '이상치(outlier)' LiDAR 주행 거리계 요인을 생성한다. 이 잘못된 제약 조건들은 IMU나 다른 정적 LiDAR 특징점들이 예측하는 실제 움직임과 모순된다. 최적화기는 전체 오차를 최소화하려는 과정에서 이 이상치 요인들에 의해 잘못된 방향으로 이끌려 결국 부정확한 상태 추정 결과를 내놓게 된다. 즉, 정적 환경에서 LIO-SAM을 정확하게 만드는 바로 그 메커니즘이 동적 환경에서는 시스템을 취약하게 만드는 것이다. 이는 LIO-SAM이 기하학적 특징점을 과도하게 '신뢰'하기 때문이며, LIO-CSI가 특징점이 요인 그래프에 포함되기 전에 시맨틱 검증 계층을 추가하는 기술적 당위성을 제공한다.

또한, LIO-SAM의 네이티브 루프 클로저 방식은 반경 탐색 후 ICP(Iterative Closest Point) 정합에 의존하는 순수 기하학적 접근법이다.11 이 방식은 긴 복도나 창고의 동일한 선반과 같이 반복적인 구조를 가진 환경에서 기하학적으로 유사하지만 실제로는 다른 과거 위치를 잘못 인식하는 '지각적 모호성(perceptual aliasing)' 문제에 취약하다. 또한, 주차된 차가 이동하는 등 재방문한 장소의 모습이 동적 객체로 인해 변경된 경우 ICP 정합이 실패하여 루프 클로저를 찾지 못할 수 있다. 이는 LIO-SAM 프레임워크에 존재하는 결정적인 한계이며, LIO-CSI의 시맨틱 보조 스캔 컨텍스트가 바로 이 문제를 해결하기 위해 설계되었다.



LIO-CSI 프레임워크는 7개의 주요 모듈로 구성된다.6 데이터 흐름은 다음과 같다: 원시 LiDAR/IMU 데이터가 입력되면, 시맨틱 엔진(SPVNAS)이 이를 처리하여 시맨틱 레이블을 생성한다. 이 레이블은 동적 객체를 필터링하는 데 사용되며, 정제된 포인트 클라우드는 프론트엔드 주행 거리계 모듈로 전달되어 요인 그래프에 제약 조건을 추가한다. 동시에, 필터링된 포인트 클라우드는 백엔드 루프 클로저 모듈에서도 사용되어 요인 그래프에 추가적인 제약 조건을 제공한다. 최종적으로 요인 그래프에서 최적화된 자세는 전역 지도를 생성하는 데 사용된다.1


- **네트워크 선택:** LIO-CSI는 점별 시맨틱 분할을 위해 SPVNAS(Sparse Point-Voxel Neural Architecture Search) 네트워크를 사용한다.1 이는 복셀 기반 처리의 효율성과 포인트 기반 방식의 정밀도를 결합한 포인트-복셀 하이브리드 방식으로, 계산 효율과 정확성 사이의 균형을 맞추기 위한 합리적인 선택이다.15
- **레이블 보정 모듈:** 딥러닝 네트워크의 출력은 완벽하지 않다. LIO-CSI는 신뢰도가 낮은 포인트의 레이블을 보정하기 위해 유클리드 거리 기반 클러스터링 전략을 사용한다.6 이 후처리 단계는 분할 노이즈가 후속 작업에 미치는 영향을 완화하고, 시맨틱 정보의 신뢰성을 향상시킨다. 이 레이블 보정 모듈은 딥러닝을 로보틱스에 적용할 때의 현실적인 어려움에 대한 깊은 이해를 보여주는 중요한 요소이다. 딥러닝 네트워크가 완벽한 신탁이 아니라는 점을 인정하고, 그 불완전성을 보완하는 안전장치를 마련한 것이다. 만약 이 모듈이 없다면, SPVNAS가 정적인 벽의 일부를 '차량'으로 잘못 분류했을 때, 동적 객체 필터가 이 벽 부분을 제거해 버릴 것이다. 이는 유효하고 안정적인 기하학적 특징점의 손실로 이어져, 오히려 주행 거리계 드리프트를 증가시킬 수 있다. 따라서 레이블 보정 모듈은 시맨틱 데이터의 품질을 보장하여 시스템 전체가 인식 구성 요소의 불완전성에 대해 더 강인해지도록 만든다.


LIO-CSI의 핵심 기여 중 하나는 시맨틱 정보를 프론트엔드에 통합하여 주행 거리계의 정확도를 높이는 것이다.

- **동적 객체 필터링:** 시맨틱 정보의 가장 직접적인 활용이다. '보행자', '차량' 등 동적 클래스로 레이블링된 포인트들은 정합 및 지도 작성에 사용되는 포인트 클라우드에서 명시적으로 제거된다.1 이는 섹션 2에서 논의된 이상치 요인의 생성을 원천적으로 차단한다.
- **시맨틱 일관성을 고려한 특징점 연관:** 단순한 필터링을 넘어, 시맨틱 정보는 특징점 정합 과정을 정교화하는 데 사용된다. 스캔 간에 대응되는 엣지 또는 평면 특징점을 탐색할 때, LIO-CSI는 두 특징점이 동일한 시맨틱 레이블을 가져야 한다는 제약 조건을 추가한다.6 이는 탐색 공간을 크게 줄이고, 기하학적으로는 유사하지만 시맨틱적으로 다른 특징점(예: 연석의 엣지와 자동차 범퍼의 엣지)이 잘못 정합되는 것을 방지한다.
- **시맨틱 손실 함수:** 스캔 정합을 위한 Levenberg-Marquardt (L-M) 최적화에 새로운 시맨틱 손실 함수가 도입되었다. 이 함수는 시맨틱 불일치에 대한 페널티를 추가하여, 최종 변환 행렬이 기하학적 정렬뿐만 아니라 장면의 시맨틱 구조까지 존중하도록 보장한다.6


LIO-CSI의 두 번째 주요 기여는 백엔드의 강인성을 향상시키는 것이다.

- **전통적인 스캔 컨텍스트의 한계:** 표준 스캔 컨텍스트 방법은 포인트 클라우드를 각 빈(bin)의 최대 높이 정보에 기반한 2D 디스크립터로 인코딩한다.6 이는 효과적이지만 상당한 양의 정보를 버리게 된다.
- **시맨틱 인코딩:** LIO-CSI는 "시맨틱 보조 스캔 컨텍스트 이미지"를 생성하여 이를 개선한다.6 단순히 최대 높이만 저장하는 대신, 디스크립터의 각 빈은 그 안에 포함된 포인트들의 시맨틱 레이블로부터 파생된 가중치나 값으로 보강된다. 이를 통해 훨씬 더 풍부하고 식별력 있는 디스크립터가 생성된다.
- **향상된 장소 인식:** 이 강화된 디스크립터는 시스템이 기하학적으로는 유사하지만 시맨틱 내용이 다른 두 장소(예: 자동차가 줄지어 있는 거리와 나무가 줄지어 있는 거리)를 구별할 수 있게 한다. 이는 지각적 모호성 문제를 직접적으로 해결하며, 특히 복잡한 도심 환경에서 루프 클로저 탐지를 훨씬 더 강인하게 만든다.5

프론트엔드와 백엔드 양쪽에서 시맨틱 정보를 사용하는 것은 중복이 아니라 시너지 효과를 창출한다. 개선된 프론트엔드는 백엔드의 작업을 더 쉽게 만들고, 더 강인한 백엔드는 개선된 프론트엔드에도 불구하고 누적될 수 있는 큰 드리프트를 보정할 수 있다. 시맨틱 프론트엔드는 동적 객체를 필터링하고 특징점 정합을 개선하여 더 정확하고 드리프트가 적은 주행 거리계 추정치를 생성한다. 이로 인해 로봇이 특정 위치로 돌아왔을 때 현재 스캔과 지도 간의 기하학적 불일치가 줄어들어, 백엔드의 시맨틱 보조 스캔 컨텍스트가 올바른 매치를 찾을 확률이 높아진다. 반대로, 강인한 시맨틱 백엔드는 상당한 드리프트가 누적된 상황에서도 루프를 감지할 수 있다. 이는 프론트엔드에서 놓친 일부 동적 객체로 인해 드리프트가 발생하더라도, 이를 재설정하고 무한정 커지는 것을 방지하는 강력한 보정 요인을 제공한다. 각 구성 요소가 서로를 강화하여, 시스템 전체는 각 부분의 합보다 더 강인해진다.



LIO-CSI의 성능을 정량적으로 평가하기 위해, SLAM 정확도의 표준 척도인 절대 궤적 오차(Absolute Trajectory Error, ATE)와 상대 자세 오차(Relative Pose Error, RPE)를 사용한다. 아래 표 1은 LIO-CSI를 직접적인 베이스라인인 LIO-SAM 및 다른 최첨단 시스템인 FAST-LIO2와 비교한 결과이다.

| 알고리즘     | 시퀀스 ID | 절대 궤적 오차 (ATE) RMSE [m] | 상대 자세 오차 (RPE) 변환 [%] | 핵심 차별점           |
| ------------ | --------- | ----------------------------- | ----------------------------- | --------------------- |
| LIO-SAM 6    | 00        | 2.92                          | 0.88                          | 요인 그래프 + 기하학  |
| LIO-CSI 6    | 00        | **2.21**                      | **0.71**                      | 요인 그래프 + 시맨틱  |
| LIO-SAM 6    | 05        | 2.54                          | 0.81                          | 요인 그래프 + 기하학  |
| LIO-CSI 6    | 05        | **1.35**                      | **0.52**                      | 요인 그래프 + 시맨틱  |
| LIO-SAM 6    | 07        | 1.34                          | 0.59                          | 요인 그래프 + 기하학  |
| LIO-CSI 6    | 07        | **0.87**                      | **0.43**                      | 요인 그래프 + 시맨틱  |
| FAST-LIO2 17 | 00        | 2.30                          | -                             | 칼만 필터 + 직접 방식 |
| FAST-LIO2 17 | 05        | 1.30                          | -                             | 칼만 필터 + 직접 방식 |
| FAST-LIO2 17 | 07        | 0.80                          | -                             | 칼만 필터 + 직접 방식 |

**표 1: KITTI 데이터셋에서의 LiDAR-관성 SLAM 시스템 정량적 성능 비교**

표의 데이터는 동적 객체가 많은 시퀀스(예: 00, 05, 07)에서 LIO-CSI가 LIO-SAM보다 ATE와 RPE 모두에서 일관되게 우수한 성능을 보임을 명확히 보여준다. 이는 시맨틱 정보 통합이 동적 환경에서의 궤적 추정 정확도를 크게 향상시킨다는 것을 입증한다. 한편, FAST-LIO2는 칼만 필터 기반의 직접 방식으로 매우 경쟁력 있는 성능을 보여주며, 특히 정적 특징이 풍부한 시나리오에서 강점을 나타낼 수 있음을 시사한다.17


정량적 지표만으로는 알고리즘의 실제 동작을 완전히 설명할 수 없다. LIO-CSI 논문에서 자체 수집한 JLU 캠퍼스 데이터셋은 움직이는 차량이 많은 도전적인 동적 시나리오를 포함하고 있어 정성적 분석에 적합하다.1

- **궤적 정확도:** 궤적 플롯을 비교해 보면, LIO-CSI의 궤적은 지상 실측값(ground truth)을 매우 근접하게 따라가는 반면, LIO-SAM의 궤적은 동적 객체가 많은 구간에서 크게 벗어나는 모습을 보인다.
- **지도 품질:** 두 시스템이 생성한 포인트 클라우드 지도를 시각적으로 비교하면 그 차이는 더욱 명확해진다. LIO-SAM이 생성한 지도에는 움직이는 차량으로 인한 '고스팅' 현상과 잔상 아티팩트가 뚜렷하게 나타난다. 반면, LIO-CSI가 생성한 지도는 이러한 동적 요소들이 효과적으로 제거되어 정적 환경만을 깨끗하게 표현한다.6 이러한 정성적 결과는 LIO-CSI의 동적 객체 필터가 실제로 어떻게 작동하여 더 깨끗한 지도를 만들고, 그 결과로 어떻게 더 정확한 궤적을 유지하는지를 직관적으로 보여준다. 이는 단순히 숫자 테이블을 보는 것보다 알고리즘의 실질적인 영향력을 이해하는 데 더 효과적이다.


LIO-CSI와 경쟁 시스템 간의 아키텍처 선택은 성능 특성에 중요한 영향을 미친다.

- **LIO-CSI (요인 그래프, 특징점 기반):** 요인 그래프의 전역 최적화 능력을 계승하여, 루프 클로저를 포함한 다양한 제약 조건을 비인과적으로 통합하고 재선형화할 수 있는 장점이 있다. 그러나 명시적인 기하학적/시맨틱 특징점 추출에 의존하므로, 비정형 환경에서는 이 과정이 병목이 되거나 실패할 수 있다.6
- **FAST-LIO2 (반복 칼만 필터, 직접 방식):** 상태 전파에 계산적으로 더 효율적인 반복 칼만 필터를 사용한다. 핵심 혁신은 특징점 추출 단계를 생략하고, 효율적인 `ikd-Tree` 데이터 구조를 사용하여 원시 포인트를 지도에 직접 정합하는 '직접(direct)' 방식이다.17 이로 인해 매우 빠르고 다양한 유형의 LiDAR에 쉽게 적응할 수 있다.

성능 격차는 시나리오에 따라 달라진다. LIO-CSI는 동적 시퀀스에서 LIO-SAM을 크게 능가하지만, 정적인 시퀀스에서는 그 차이가 줄어들 수 있다. 또한, FAST-LIO2는 특징이 풍부한 정적 환경에서는 속도와 정확도 면에서 LIO-CSI를 능가할 수 있다. 정적 환경에서는 LIO-SAM과 FAST-LIO2의 기본 가정이 유효하므로, 시맨틱 분할의 계산 오버헤드가 추가되는 LIO-CSI는 상대적으로 불리할 수 있다. 그러나 동적 객체가 도입되면 LIO-SAM과 FAST-LIO2의 핵심 가정이 깨지면서 성능이 저하되는 반면, LIO-CSI는 시맨틱 필터를 통해 성능을 유지하며 결정적인 이점을 갖게 된다. 결론적으로, 단일 '최고의' SLAM 시스템은 없으며, 선택은 동적 환경에 대한 강인성을 위해 계산 비용을 감수할 것인지(LIO-CSI), 아니면 정적 환경에서의 속도와 정확도를 우선할 것인지(FAST-LIO2)의 트레이드오프에 달려 있다.



LIO-CSI의 가장 큰 취약점은 핵심 기능이 SPVNAS라는 딥러닝 모델에 의존한다는 점이다.

- **악천후 조건:** 안개, 비, 눈과 같은 악천후 조건에서는 LiDAR 센서와 이를 기반으로 하는 모든 인식 알고리즘의 성능이 크게 저하된다.20 빛의 산란과 흡수는 노이즈를 유발하고 센서 범위를 감소시키며, 주로 맑은 날씨 데이터로 학습된 시맨틱 분할 네트워크를 혼란시키는 아티팩트를 생성한다.24 이러한 시맨틱 입력의 품질 저하는 LIO-CSI가 동적 객체를 필터링하지 못하거나, 심지어 정적 환경의 일부를 동적 객체로 오인하여 제거하는 치명적인 실패로 이어질 수 있다.
- **적대적 공격(Adversarial Attacks):** DNN에 의존하는 시스템으로서 LIO-CSI는 적대적 공격에 취약하다.28 공격자가 특수하게 제작된 간단한 물체(예: 특정 패턴의 판지)를 환경에 배치하여 분할 네트워크를 속이는 물리적으로 실현 가능한 공격이 존재한다.30 예를 들어, 보행자를 '도로'로 오인하게 만들어 로봇에게 보이지 않게 하거나, 도로 일부를 '건물'로 오인하게 만들어 불필요한 정지나 경로 계획 실패를 유발할 수 있다. 시맨틱 정보의 도입은 이처럼 교묘하고 새로운 공격 표면을 만들어낸다. 전통적인 기하학적 SLAM 시스템을 공격하려면 LiDAR 신호를 물리적으로 방해해야 하지만, LIO-CSI는 센서 데이터의 '해석'을 공격함으로써 시스템을 마비시킬 수 있다. 즉, 시스템의 '지능' 자체가 시스템을 공격하는 무기가 되는 것이다. 이는 시맨틱 SLAM 시스템의 보안이 센서 보안뿐만 아니라 인식 모델에 대한 강력한 적대적 방어 메커니즘을 필요로 함을 시사한다.


3D 포인트 클라우드에 대한 딥러닝 모델은 계산 자원을 많이 소모하는 것으로 알려져 있다.16 LIO-CSI 논문은 실시간 성능을 주장하지만, SPVNAS 네트워크가 추가하는 계산 부담은 연구에 사용되는 고성능 GPU가 아닌, 실제 모바일 로봇에 탑재되는 자원 제약적인 플랫폼에서는 상당한 도전이 될 수 있다.


본 보고서의 분석을 종합하여, LIO-CSI를 기반으로 한 유망한 미래 연구 방향을 다음과 같이 제시한다.

- **강인한 장소 인식을 위한 다중 모달 융합:** 긴 복도와 같이 기하학적으로 모호한 환경에서 LiDAR 단독 장소 인식의 한계를 극복하기 위해, LIO-CSI의 시맨틱 LiDAR 지도와 다른 센서 양식을 융합하는 방안을 제안한다.
  - **WiFi-CSI/RSSI:** 실내 환경에서 WiFi 신호는 시각적 외형과 무관한 위치별 고유의 '지문'을 제공한다.33 WiFi 지문을 시맨틱 문맥과 융합하면, 로봇이 시각적으로 동일해 보이는 두 복도를 고유한 무선 주파수 서명을 기반으로 구별할 수 있게 된다.37 이를 위해 더 정밀하지만 특정 하드웨어가 필요한 CSI와, 보편적이지만 안정성이 낮은 RSSI 간의 장단점을 비교 분석할 필요가 있다.39
  - **초광대역(UWB):** 고정밀, 드리프트 없는 위치 추정이 요구되는 응용 분야에서는 UWB가 센티미터 수준의 거리 측정 정보를 제공한다. 이를 LIO 시스템과 강결합하면 드리프트를 완전히 제거하여 '실내 GPS'와 같은 역할을 할 수 있다.42
- **환경 변화 적응을 위한 평생 SLAM (Lifelong SLAM):** 실제 환경은 동적 객체뿐만 아니라 가구 재배치나 계절 변화와 같은 장기적인 구조적, 외형적 변화를 겪는다.46 LIO-CSI가 생성하는 시맨틱 지도는 이러한 평생 SLAM 시스템의 기반이 될 수 있다. 시스템은 현재의 시맨틱 인식 결과와 저장된 지도 간의 불일치를 감지하여 지도 업데이트 절차를 촉발함으로써, 장기 자율 운영을 위한 지도의 최신성과 신뢰성을 유지할 수 있다.49 이는 시간이 지나면 낡게 되는 WiFi 지도에도 마찬가지로 적용될 수 있으며, 로봇을 이용한 주기적인 재보정이 필요하다.53

궁극적인 미래 방향은 단순히 더 많은 센서를 추가하는 것을 넘어, 어떤 문맥에서 어떤 센서를 신뢰할지 추론할 수 있는 통합 프레임워크를 구축하는 것이다. 시맨틱 지도는 이러한 추론을 가능하게 하는 핵심 열쇠이다. 로봇은 초기 학습 단계를 통해 특정 시맨틱 레이블과 센서 성능 간의 상관관계를 학습할 수 있다. 예를 들어, '복도'에서는 LiDAR 장소 인식이 취약하지만 WiFi-CSI가 고유한 서명을 제공하고, '로비'에서는 시각적 특징이 신뢰할 수 있으며, '비 오는 야외'에서는 LiDAR 시맨틱 정보의 신뢰도가 낮아진다는 것을 학습한다. 장기 운영 중에 로봇은 현재 자신의 시맨틱 위치를 기반으로 SLAM 시스템에 대한 입력 가중치를 동적으로 조절한다. '복도'에 들어서면 LiDAR 루프 클로저 요인의 가중치를 낮추고 WiFi 장소 인식 요인의 가중치를 높인다. '비 오는 야외'로 나가면 시맨틱 분할의 불확실성을 높이고 IMU와 지면의 기하학적 특징에 더 의존한다. 이는 정적인 센서 융합 가중치를 넘어, 환경에 대한 고차원적 이해를 바탕으로 융합 전략을 지능적으로 조정하는 진정한 의미의 문맥 인식 시스템으로 나아가는 길이다.


1. LIO-CSI: LiDAR inertial odometry with loop closure combined with semantic information, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8654169/
2. [2206.09463] RF-LIO: Removal-First Tightly-coupled Lidar Inertial Odometry in High Dynamic Environments - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2206.09463
3. LIO-CSI: LiDAR inertial odometry with loop closure combined with semantic information, accessed July 23, 2025, https://pubmed.ncbi.nlm.nih.gov/34879118/
4. RB-LIO: A SLAM Solution Applied to Large-Scale Dynamic Scenes with Multiple Moving Objects | CoLab, accessed July 23, 2025, https://colab.ws/articles/10.1007/978-981-99-8021-5_14
5. (PDF) LIO-CSI: LiDAR inertial odometry with loop closure combined with semantic information - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/356887770_LIO-CSI_LiDAR_inertial_odometry_with_loop_closure_combined_with_semantic_information
6. LIO-CSI: LiDAR inertial odometry with loop closure combined with semantic information, accessed July 23, 2025, https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0261053
7. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - MDPI, accessed July 23, 2025, https://www.mdpi.com/1424-8220/24/23/7546
8. Enhancing Mobile Robot Navigation Through Semantic SLAM Integration - Frontiers, accessed July 23, 2025, https://www.frontiersin.org/research-topics/50985/enhancing-mobile-robot-navigation-through-semantic-slam-integration
9. LiDAR-Based SLAM under Semantic Constraints in Dynamic Environments - MDPI, accessed July 23, 2025, https://www.mdpi.com/2072-4292/13/18/3651
10. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2007.00258
11. gisbi-kim/SC-LIO-SAM: LiDAR-inertial SLAM - GitHub, accessed July 23, 2025, https://github.com/gisbi-kim/SC-LIO-SAM
12. A Lidar SLAM Algorithm Based on Improved LIO-SAM - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/378504664_A_Lidar_SLAM_Algorithm_Based_on_Improved_LIO-SAM
13. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 23, 2025, https://github.com/TixiaoShan/LIO-SAM
14. a SLAM implementation combining FAST-LIO2 with pose graph optimization and loop closing based on LIO-SAM paper - GitHub, accessed July 23, 2025, https://github.com/engcang/FAST-LIO-SAM
15. Sparse point‐voxel aggregation network for efficient point cloud semantic segmentation, accessed July 23, 2025, https://www.researchgate.net/publication/362790334_Sparse_point-voxel_aggregation_network_for_efficient_point_cloud_semantic_segmentation
16. An Onboard Point Cloud Semantic Segmentation System for Robotic Platforms - MDPI, accessed July 23, 2025, https://www.mdpi.com/2075-1702/11/5/571
17. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - GitHub, accessed July 23, 2025, https://raw.githubusercontent.com/hku-mars/FAST_LIO/main/doc/Fast_LIO_2.pdf
18. [2107.06829] FAST-LIO2: Fast Direct LiDAR-inertial Odometry - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2107.06829
19. FAST-LIO - Jiarong Lin, accessed July 23, 2025, https://jiaronglin.com/tag/fast-lio/
20. (PDF) Weather-adaptive slam using unmanned ground vehicles - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/378435008_Weather-adaptive_slam_using_unmanned_ground_vehicles
21. UniMix: Towards Domain Adaptive and Generalizable LiDAR Semantic Segmentation in Adverse Weather | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/386169508_UniMix_Towards_Domain_Adaptive_and_Generalizable_LiDAR_Semantic_Segmentation_in_Adverse_Weather
22. Survey on LiDAR Perception in Adverse Weather Conditions - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2304.06312
23. [2104.05347] Radar SLAM: A Robust SLAM System for All Weather Conditions - ar5iv - arXiv, accessed July 23, 2025, https://ar5iv.labs.arxiv.org/html/2104.05347
24. Rethinking Range-View LiDAR Segmentation in Adverse Weather - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2506.08979
25. arXiv:2503.15910v2 [cs.CV] 24 Mar 2025, accessed July 23, 2025, [https://arxiv.org/pdf/2503.15910?](https://arxiv.org/pdf/2503.15910)
26. Evaluating and Improving the Robustness of LiDAR-based Localization and Mapping - arXiv, accessed July 23, 2025, https://arxiv.org/html/2409.10824v1
27. Benchmarking Automotive LiDAR Performance in Arctic Conditions - VTT's Research Information Portal, accessed July 23, 2025, https://cris.vtt.fi/files/35400640/Kutila_Paper_35_IEEE_ITSC_2020_LiDAR_artic_weather_20092020.pdf
28. Adversarial Attacks against LiDAR Semantic Segmentation in Autonomous Driving | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/356219000_Adversarial_Attacks_against_LiDAR_Semantic_Segmentation_in_Autonomous_Driving
29. (Open Access) Adversarial Attacks against LiDAR Semantic Segmentation in Autonomous Driving (2021) | Yi Zhu | 40 Citations - SciSpace, accessed July 23, 2025, https://scispace.com/papers/adversarial-attacks-against-lidar-semantic-segmentation-in-2v9n8aig0d
30. Adversarial Attacks against LiDAR Semantic Segmentation in Autonomous Driving - College of Engineering - Purdue University, accessed July 23, 2025, https://engineering.purdue.edu/~lusu/papers/SenSys2021.pdf
31. Adversarial Attacks against LiDAR Semantic Segmentation in Autonomous Driving, accessed July 23, 2025, https://consensus.app/papers/adversarial-attacks-against-lidar-semantic-segmentation-huai-miao/a313a30b43305133a3860f928c13e746
32. Searching Efficient 3D Architectures with Sparse Point-Voxel Convolution, accessed July 23, 2025, https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123730681.pdf
33. indoor-localization / GitHub Topics, accessed July 23, 2025, https://github.com/topics/indoor-localization
34. RoboFiSense: Attention-Based Robotic Arm Activity Recognition with WiFi Sensing - arXiv, accessed July 23, 2025, https://arxiv.org/html/2312.15345v2
35. Let us quickly dive into how the current Visual and LiDAR SLAM, accessed July 23, 2025, https://wcsng.ucsd.edu/files/wais.pdf
36. ViWiD: Leveraging WiFi for Robust and Resource-Efficient SLAM, accessed July 23, 2025, https://arxiv.org/pdf/2209.08091
37. Fusion of the SLAM with Wi-Fi-Based Positioning Methods for Mobile Robot-Based Learning Data Collection, Localization, and Tracking in Indoor Spaces - PMC, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7570627/
38. Collaborative SLAM Based on WiFi Fingerprint Similarity and Motion Information | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/337726487_Collaborative_SLAM_Based_on_WiFi_Fingerprint_Similarity_and_Motion_Information
39. On TinyML WiFi Fingerprinting-Based Indoor Localization: Comparing RSSI vs. CSI Utilization | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/379057197_On_TinyML_WiFi_Fingerprinting-Based_Indoor_Localization_Comparing_RSSI_vs_CSI_Utilization
40. Machine Learning Based Indoor Localization Using Wi-Fi RSSI Fingerprints: An Overview, accessed July 23, 2025, https://www.researchgate.net/publication/354488129_Machine_Learning_Based_Indoor_Localization_Using_Wi-Fi_RSSI_Fingerprints_An_Overview
41. Poster: Comparison of Channel State Information driven and RSSI-based WiFi Distance Estimation - Carlo Alberto Boano, accessed July 23, 2025, https://www.carloalbertoboano.com/documents/salomon21csi.pdf
42. RLI-SLAM: Fast Robust Ranging-LiDAR-Inertial Tightly-Coupled Localization and Mapping, accessed July 23, 2025, https://www.mdpi.com/1424-8220/24/17/5672
43. Ultra Wide-Band Localization and SLAM: A Comparative Study for Mobile Robot Navigation - PMC - PubMed Central, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC3274006/
44. [2408.05719] MR-ULINS: A Tightly-Coupled UWB-LiDAR-Inertial Estimator with Multi-Epoch Outlier Rejection - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2408.05719
45. UWB/LiDAR Fusion For Cooperative Range-Only SLAM - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/328801067_UWBLiDAR_Fusion_For_Cooperative_Range-Only_SLAM
46. [2010.08846] Lifelong update of semantic maps in dynamic environments - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2010.08846
47. Dynamic Maps for Long-Term Operation of Mobile Service Robots, accessed July 23, 2025, https://www.roboticsproceedings.org/rss01/p03.pdf
48. While lifelong SLAM considers the long-term operation of a robot in a... - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/figure/While-lifelong-SLAM-considers-the-long-term-operation-of-a-robot-in-a-single-dynamically_fig1_358997200
49. Lifelong update of semantic maps in dynamic environments - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2010.08846
50. Khronos: A Unified Approach for Spatio-Temporal Metric-Semantic SLAM in Dynamic Environments - Robotics, accessed July 23, 2025, https://www.roboticsproceedings.org/rss20/p081.pdf
51. PoseMap: Lifelong, Multi-Environment 3D LiDAR Localization - CSIRO Research, accessed July 23, 2025, https://research.csiro.au/robotics/wp-content/uploads/sites/96/2018/09/posemap_iros2018-10.pdf
52. Towards Lifelong Feature-Based Mapping in Semi-Static Environments - Google Research, accessed July 23, 2025, https://research.google.com/pubs/archive/44821.pdf
53. Utilizing Automated Robots to Recalibrate WiFi Fingerprint Maps for Indoor Location Estimation - WorldComp Proceedings, accessed July 23, 2025, http://worldcomp-proceedings.com/proc/p2012/ICW3179.pdf
54. Adaptive Indoor Positioning Model Based on WLAN-Fingerprinting for Dynamic and Multi-Floor Environments, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC5579808/
55. An Adaptive Indoor Localization Approach Using WiFi RSSI Fingerprinting with SLAM-Enabled Robotic Platform and Deep Neural Networks - arXiv, accessed July 23, 2025, https://arxiv.org/html/2407.09242v1


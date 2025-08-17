# FAST_LIO-GPS

## 1. 부: FAST_LIO의 구조적 핵심과 철학

이 파트에서는 FAST-LIO-GPS의 근간이 되는 FAST-LIO, 특히 FAST-LIO2가 어떤 철학으로 설계되었는지 그 핵심 기술들을 중심으로 분석한다. FAST-LIO 계열의 성공은 단순히 빠른 연산 속도에 그치지 않는다. 이는 시스템의 각 구성 요소가 서로의 한계를 보완하고 잠재력을 극대화하도록 유기적으로 설계된 결과물이다.

### 1.1. 서론: 필터 기반 LIO의 등장과 FAST-LIO의 의의

LiDAR Odometry and Mapping (LOAM) 1으로 대표되는 최적화 기반 LiDAR-Inertial Odometry (LIO)의 성공 이후, 로보틱스 커뮤니티는 한동안 최적화 기반 접근법의 정밀도를 높이는 데 집중했다. 그러나 자율주행차, 드론 등 실제 로봇 플랫폼에 SLAM 기술을 탑재하려는 시도가 늘어나면서 새로운 요구사항이 대두되었다. 바로 제한된 온보드 컴퓨팅 자원 내에서의 실시간성과 극단적인 환경 변화에 대한 강건성이다. 이러한 배경 속에서 계산 효율성과 강건성을 핵심 무기로 내세운 필터 기반 LIO 시스템들이 다시금 주목받기 시작했다.

FAST-LIO 계열은 그중에서도 특히 '빠른(Fast)' 연산 속도에 초점을 맞춘, 매우 효율적인 Tightly-coupled Iterated Extended Kalman Filter (iEKF) 프레임워크를 제시하며 등장했다.2 이 시스템의 근본적인 목표는 명확하다: 고가의 GPU나 강력한 CPU 없이도, 심지어 드론의 급격한 회전(예: 1000 deg/s)과 같이 매우 공격적인 움직임 속에서도 안정적으로 자신의 위치와 자세를 추정하는 Odometry를 제공하는 것이다.6 이는 기존의 최적화 기반 방식이 따라오기 힘든 수준의 실시간성을 요구하는 응용 분야, 예를 들어 로봇의 동적 제어나 충돌 회피 등에 새로운 가능성을 열어주었다. FAST-LIO의 등장은 LIO 연구의 패러다임을 '최고의 정확도'에서 '실용적인 정확도와 최고의 효율성'으로 전환하는 중요한 계기가 되었다.

### 1.1  핵심 기술 1: 직접 정합 방식 (Direct Registration Method)

전통적인 LIO 시스템의 데이터 처리 파이프라인은 특징점 추출(feature extraction) 단계에서 시작한다. LOAM, LeGO-LOAM, 그리고 LIO-SAM과 같은 알고리즘들은 입력된 포인트 클라우드 전체를 사용하는 대신, 주변 점들과의 곡률(curvature)을 계산하여 기하학적으로 의미 있는 평면(plane)이나 모서리(edge) 같은 특징점들을 선별한다.1 이 방식은 전체 포인트 클라우드 중 일부(수백~수천 개)의 점만을 정합에 사용하므로 계산량을 획기적으로 줄일 수 있다는 명확한 장점이 있다. 이는 초기 LIO 시스템이 제한된 컴퓨팅 파워에서 실시간으로 동작할 수 있었던 핵심적인 이유였다.

하지만 이 방식은 명백한 한계를 내포하고 있다. 복도, 터널, 혹은 넓은 평원과 같이 구조적으로 단조로운(structure-less) 환경에서는 추출할 수 있는 특징점의 수가 급격히 줄어든다.7 특징점이 부족하면 정합을 위한 제약 조건(constraint)이 부족해져 위치 추정의 정확도와 안정성이 크게 저하된다. 또한, 특징점 추출 알고리즘 자체가 특정 환경이나 LiDAR 센서의 스캔 패턴에 맞춰 설계되는 경우가 많아, 새로운 종류의 센서(예: 불규칙한 스캔 패턴을 가진 Solid-state LiDAR)에 적용하기 위해서는 상당한 수정이 필요했다.

FAST-LIO2는 이러한 문제를 해결하기 위해 특징점 추출 단계를 과감히 생략하는 '직접 정합(direct registration)' 방식을 채택했다.2 이 접근법은 입력된 LiDAR 스캔에서 일부 포인트를 균일하게 다운샘플링한 후, 이 원본 포인트(raw points)들을 이전에 구축된 맵에 직접 정합(scan-to-map)한다. 이는 두 가지 중요한 이점을 가져온다. 첫째, 평면이나 모서리처럼 명확한 기하학적 특징이 없는 벽의 미세한 요철이나 바닥의 작은 물체 등, 환경이 제공하는 모든 미세한(subtle) 정보를 위치 추정에 활용할 수 있게 되어 정확도가 향상된다.6 둘째, 특징점 추출이라는 인위적인 과정이 사라짐으로써, 전통적인 회전형 LiDAR뿐만 아니라 Livox와 같은 Solid-state LiDAR 등 다양한 스캔 패턴을 가진 센서에도 별도의 수정 없이 자연스럽게 적용할 수 있는 범용성을 확보하게 된다.6 물론 이 방식은 특징점 기반 방식보다 훨씬 많은 수의 포인트를 처리해야 하므로 계산 부하가 크다는 단점이 있지만, FAST-LIO2는 이를 극복하기 위한 혁신적인 자료구조와 상태 추정 기법을 함께 제시했다.

### 1.2  핵심 기술 2: ikd-Tree (The Incremental K-D Tree)

LIO 시스템의 성능은 단순히 정합 알고리즘의 정확도에만 의존하지 않는다. 실시간으로 입력되는 새로운 스캔 데이터를 기존 맵에 효율적으로 통합하고, 동시에 이 거대한 맵에서 현재 스캔과 정합할 이웃 점들을 빠르게 찾아내는 맵 관리(map management) 기술이 시스템 전체의 성능을 좌우하는 병목 구간이 되기 십상이다. 특히 FAST-LIO2가 채택한 직접 정합 방식은 매 스캔마다 수천, 수만 개의 포인트를 맵에 추가하고 검색해야 하므로, 맵 관리를 위한 자료구조의 효율성이 시스템의 실시간성을 담보하는 핵심 요소가 된다.

기존의 정적(static) K-D Tree는 한 번 구축되면 빠른 최근접 이웃 탐색(k-Nearest Neighbor search, k-NN) 성능을 보장하지만, 새로운 데이터가 추가될 때마다 트리 전체를 재구성해야 하는 치명적인 단점이 있다. 이는 실시간으로 맵이 계속해서 성장하는 LIO 응용에는 부적합하다. 이 문제를 해결하기 위해 FAST-LIO2는 `ikd-Tree`라는 혁신적인 동적 K-D Tree 자료구조를 제안했다.6

`ikd-Tree`의 핵심 철학은 '증분적 업데이트(incremental update)'와 '동적 재균형(dynamic re-balancing)'이다.

1. **증분적 업데이트:** `ikd-Tree`는 새로운 포인트가 들어왔을 때 트리 전체를 재구성하는 대신, 해당 포인트가 위치할 노드를 찾아 삽입(insertion)하는 방식으로 맵을 업데이트한다. 마찬가지로 특정 영역의 포인트를 삭제(deletion)할 때도 해당 노드들을 찾아 제거하거나 '삭제됨' 플래그를 표시하는 '지연 삭제(lazy delete)' 방식을 사용한다. 또한, 포인트 삽입 시 지정된 해상도의 복셀 내에 점이 하나만 존재하도록 하는 '트리 위 다운샘플링(on-tree downsampling)' 기능도 지원하여 맵의 밀도를 효율적으로 관리한다.6
2. **동적 재균형:** 포인트의 삽입과 삭제가 반복되면 K-D Tree는 한쪽으로 치우쳐져 불균형 상태가 되고, 이는 검색 성능을 저하시킨다. `ikd-Tree`는 트리의 균형 상태를 지속적으로 감시하다가, 특정 서브트리(subtree)의 불균형이 미리 정의된 임계값을 넘어서면 해당 서브트리만 재구성하여 균형을 맞춘다. 특히, 이 재구성 작업이 실시간 처리에 방해가 되지 않도록 별도의 스레드에서 비동기적으로(asynchronously) 수행하는 영리한 설계를 채택했다.10

FAST-LIO2 논문에서는 실제 LiDAR 데이터를 이용한 실험을 통해 `ikd-Tree`가 Octree, R*-tree 등 기존의 다른 동적 자료구조에 비해 LIO 응용에서 포인트 삽입, 삭제, 검색 등 모든 측면에서 월등한 종합 성능을 보임을 입증했다.7 이처럼 

`ikd-Tree`가 제공하는 압도적인 맵 관리 효율성은, 계산량이 많은 직접 정합 방식을 실시간으로 구현할 수 있게 만든 핵심적인 기술적 토대이다.2 이후 i-Octree 12나 iVox 14와 같이 병렬 처리에 더 유리하고 특정 연산(예: 반경 탐색)에서 더 빠른 성능을 보이는 자료구조들이 제안되었지만, `ikd-Tree`는 동적 K-D Tree의 개념을 LIO 분야에 성공적으로 도입하고 그 실효성을 증명한 선구적인 업적으로 평가받는다.

### 1.3  상태 추정 엔진: Iterated Error-State Kalman Filter (iEKF)

FAST-LIO 시스템의 두뇌 역할을 하는 것은 상태 추정 엔진이다. 이 엔진은 부정확한 IMU 측정값과 노이즈가 섞인 LiDAR 측정값을 Tightly-coupled 방식으로 융합하여 로봇의 현재 상태(위치, 자세, 속도 등)를 최적으로 추정하는 역할을 담당한다. FAST-LIO는 이를 위해 Iterated Extended Kalman Filter (iEKF), 특히 Error-State Kalman Filter (ESKF)를 채택했다.3

ESKF를 선택한 이유는 로봇의 자세(orientation)와 같은 상태 변수가 유클리드 벡터 공간(Euclidean vector space)이 아닌 SO(3)와 같은 비선형 매니폴드(non-Euclidean manifold) 상에 존재하기 때문이다. 일반적인 EKF는 이러한 변수를 직접 다룰 때 특이점(singularity) 문제나 비선형성으로 인한 오차가 발생하기 쉽다. 반면, ESKF는 참 값(true state)과 추정 값(nominal state) 사이의 작은 오차(error state)를 상태 변수로 정의한다. 이 오차 상태는 국소적으로 유클리드 공간으로 근사할 수 있으므로, 표준 칼만 필터의 선형 업데이트 규칙을 더 정확하게 적용할 수 있다. 업데이트가 끝나면 추정된 오차를 현재 상태에 더해주고 오차 상태는 다시 0으로 리셋하는 과정을 거친다. 이 방식은 매니폴드 상의 상태를 더 안정적이고 정확하게 추정하는 데 매우 효과적이다.16

FAST-LIO가 'Fast'라는 이름을 가질 수 있었던 가장 결정적인 혁신은 바로 이 iEKF의 업데이트 과정에 있다. 전통적인 칼만 필터 업데이트 수식에서 칼만 게인(Kalman gain) `$K$`를 계산하는 과정에는 `$(H P H^T + R)^{-1}$`와 같은 항이 포함된다. 여기서 `$H$`는 측정 모델의 자코비안이고, `$R$`은 측정 노이즈의 공분산이다. LIO 문제에서 측정값은 수천 개에 달하는 LiDAR 포인트들이므로, 이 행렬의 크기는 (측정값 개수 x 측정값 개수)가 되어 역행렬 계산에 엄청난 연산량이 소요된다.

FAST-LIO는 Sherman-Morrison-Woodbury 공식을 포함한 행렬 대수 기법을 창의적으로 적용하여, 칼만 게인을 계산하는 새로운 공식을 유도했다.2 이 새로운 공식의 계산 복잡도는 측정값의 차원(수천)이 아닌, 상태 변수의 차원(수십)에만 의존한다.
$$
\mathbf{K} = (\mathbf{H}^T \mathbf{R}^{-1} \mathbf{H} + \mathbf{P}^{-1})^{-1} \mathbf{H}^T \mathbf{R}^{-1}
$$
이 수학적 혁신 덕분에 수많은 LiDAR 측정값을 Tightly-coupled 방식으로 융합하면서도 계산 부하를 극적으로 줄일 수 있었고, 이것이 바로 FAST-LIO가 압도적인 실시간 성능을 달성할 수 있었던 핵심 비결이다.5

### 1.4 부 핵심 통찰 및 연관 관계 분석

FAST-LIO2의 혁신적인 성능은 단순히 개별 기술들의 우수성을 나열하는 것만으로는 완전히 설명될 수 없다. 그 진정한 힘은 각 핵심 기술들이 서로의 전제 조건이 되고, 서로의 성능을 증폭시키는 강력한 상호 인과 관계와 시너지 효과에 있다.

'직접 정합' 방식은 환경의 모든 정보를 활용하여 정확도를 높이는 이상적인 접근법이지만, 이는 기존의 특징점 기반 방식보다 훨씬 많은 수의 포인트를 처리해야 한다는 현실적인 문제를 야기한다. 만약 전통적인 정적 K-D Tree를 사용했다면, 매 스캔마다 트리 전체를 재구성하는 데 소요되는 시간 때문에 실시간 맵 업데이트와 k-NN 탐색은 불가능했을 것이다. 바로 이 지점에서 'ikd-Tree'라는 고효율 동적 자료구조가 등장하여 문제를 해결한다. `ikd-Tree`의 증분적 업데이트와 동적 재균형 기능은 방대한 양의 원본 포인트를 실시간으로 맵에 통합하고 검색하는 것을 가능하게 만들었다.6 즉, 

`ikd-Tree`의 계산 효율성이 '직접 정합'이라는 아이디어를 이론에서 현실로 이끌어낸 핵심적인 기술적 원동력(enabler)이 된 것이다.

이렇게 Front-end에서 발생하는 막대한 계산 부하를 감당하면서도, IMU와 LiDAR 데이터를 Tightly-coupled 방식으로 융합하기 위해서는 Back-end의 상태 추정기 또한 극도로 효율적이어야 했다. 만약 전통적인 칼만 게인 계산 방식을 고수했다면, 수천 개의 측정값에 대한 거대한 행렬의 역행렬 계산이 시스템 전체의 병목이 되었을 것이다. FAST-LIO가 독자적으로 개발한 '간소화된 칼만 게인 계산법'을 적용한 iEKF는 이 문제를 해결하며, Front-end의 부하를 감당할 수 있는 강력하고 빠른 추정 엔진을 제공했다.2 결국, 이 세 가지 요소-**직접 정합, ikd-Tree, 그리고 효율적인 iEKF**-는 각각 독립적으로 존재하는 기술이 아니라, 서로가 서로를 가능하게 하고 성능을 뒷받침하는 하나의 유기적인 시스템으로 결합하여 FAST-LIO2라는 고성능 LIO 시스템을 완성한 것이다.

또한, FAST-LIO 계열의 진화 과정을 살펴보면 LIO 시스템의 성능 최적화가 어떤 방향으로 이루어졌는지 명확히 알 수 있다. FAST-LIO는 효율적인 칼만 게인 계산법을 통해 상태 추정기의 혁신을 이루었다.5 그 후속작인 FAST-LIO2는 여기에 `ikd-Tree`를 도입하여 맵 관리 및 업데이트 속도를 비약적으로 향상시켰다.19 더 나아가, Faster-LIO는 `ikd-Tree`를 병렬 처리에 더욱 유리한 Voxel 기반 자료구조인 `iVox`로 대체하여 속도를 1.5배에서 2배까지 끌어올렸다.14 이 발전 과정에서 주목할 점은, 핵심 상태 추정기인 iEKF의 기본 골격은 큰 변화 없이 유지되었다는 것이다. 이는 FAST-LIO의 iEKF 프레임워크가 이미 충분히 강력하고 안정적이었으며, 실질적인 성능 병목은 주로 맵 데이터의 관리와 검색에 있었음을 시사한다. 이 진화의 역사는 LIO 시스템의 성능을 최적화하고자 할 때, 계산 부하가 가장 큰 부분이 어디이며, 어떤 기술적 혁신이 가장 큰 효과를 가져올 수 있는지를 보여주는 중요한 사례라고 할 수 있다.

## 2. 부: FAST_LIO-GPS: 절대 좌표계로의 확장

FAST-LIO는 그 자체로 매우 뛰어난 Odometry 시스템이지만, 근본적으로 상대적인 움직임만을 추정한다. 이는 시간이 지남에 따라 오차가 누적되는 드리프트(drift) 현상에서 자유로울 수 없다는 것을 의미한다.21 특히 수 킬로미터에 달하는 대규모 환경을 매핑할 경우, 이 누적 오차는 무시할 수 없는 수준에 이르게 된다.23 이 문제를 해결하고, 생성된 맵을 실제 세계와 연동하기 위해 GPS와 같은 전역 측위 시스템(Global Positioning System)과의 융합이 필수적이다.

### 2.1  GPS 융합의 필요성: 드리프트 보정 및 전역 맵핑

LIO 시스템은 IMU의 가속도와 각속도, 그리고 LiDAR가 감지한 주변 환경의 기하학적 구조를 기반으로 자신의 움직임을 추정한다. 이 과정은 마치 눈을 가리고 벽을 더듬으며 자신의 위치를 파악하는 것과 같다. 단기적으로는 매우 정확하지만, 이동 거리가 길어질수록 작은 오차들이 계속해서 더해져 실제 위치와의 차이가 점점 벌어지게 된다. 이것이 바로 드리프트 문제이다.

GPS는 지구상에서의 절대적인 위치 좌표를 제공함으로써 이 문제를 해결하는 열쇠가 된다. GPS는 LIO 시스템에게 "너는 지금 지도상의 이 지점에 있다"라는 기준점을 제공하여, 그동안 누적된 오차를 한 번에 보정할 수 있게 해준다. 이를 통해 LIO 시스템은 장시간, 장거리 운행에도 불구하고 전역적으로 일관된 궤적과 맵을 생성할 수 있다.21 또한, GPS 융합은 생성된 포인트 클라우드 맵에 실제 지리 좌표를 부여하는 '지리 참조(geo-referencing)'를 가능하게 한다. 이는 자율주행을 위한 HD 맵 제작이나 드론을 이용한 측량 등, 가상의 맵을 넘어 실제 세계와 상호작용해야 하는 응용 분야에서 반드시 필요한 기능이다.

### 2.2  융합 아키텍처: Odometry + Pose Graph Optimization (PGO)

GitHub에 공개된 여러 `FAST_LIO_GPS` 구현체들을 분석해 보면 3, 대부분 공통적인 아키텍처를 채택하고 있음을 알 수 있다. 바로 FAST-LIO2를 일종의 'Odometry 엔진' 또는 Front-end로 사용하고, 그 결과를 별도의 Back-end 최적화 모듈에 입력하여 GPS 데이터와 융합하는 방식이다. 이는 센서 데이터를 낮은 수준에서 직접 융합하는 Tightly-coupled 방식과 달리, 각 센서 모듈이 독립적으로 처리한 결과를 상위 레벨에서 결합하는 Loosely-coupled 융합의 전형적인 형태다.18

이 아키텍처의 Back-end에서는 주로 Pose Graph Optimization (PGO) 기법이 사용된다. PGO의 기본 원리는 다음과 같다:

1. **노드(Node) 생성:** FAST-LIO2가 일정 거리 또는 시간 간격으로 생성하는 키프레임(keyframe)의 위치와 자세(pose)를 그래프의 노드로 정의한다.
2. **엣지(Edge) 연결:** 두 연속된 키프레임 노드 사이를 FAST-LIO2가 계산한 상대적인 움직임(odometry) 정보로 연결한다. 이 연결을 엣지라고 하며, 엣지는 두 노드 간의 공간적 제약 조건을 나타낸다.
3. **팩터(Factor) 추가:** 이 그래프에 GPS 측정값이나 루프 클로저(loop closure) 정보를 새로운 종류의 팩터 또는 제약 조건으로 추가한다. GPS 팩터는 특정 노드에 절대 위치 제약을 가하고, 루프 클로저 팩터는 시간적으로는 멀리 떨어져 있지만 공간적으로는 동일한 위치를 나타내는 두 노드 사이에 상대 변환 제약을 추가한다.
4. **최적화:** 그래프에 포함된 모든 제약 조건(Odometry, GPS, Loop closure)을 가장 잘 만족시키는 노드들의 전역 위치와 자세를 비선형 최적화 기법을 통해 찾아낸다. 이 과정을 통해 전체 궤적의 누적 오차가 최소화되고 전역적인 일관성이 확보된다.3

이러한 PGO를 효율적으로 구현하기 위해, `SC-PGO` (Scan Context-based Pose Graph Optimization) 25나 `GTSAM` (Georgia Tech Smoothing and Mapping library) 27과 같은 전문적인 최적화 라이브러리가 널리 활용된다.

### 2.3  전역 좌표계 통합: UTM 좌표계의 적용

GPS 수신기는 위치를 경도(longitude), 위도(latitude), 고도(altitude)로 출력한다. 이 구면 좌표계는 SLAM에서 주로 사용하는 유클리드 공간 기반의 직교 좌표계와 달라 직접 사용하기 어렵다. 따라서 이를 미터(meter) 단위의 평면 직교 좌표계로 변환하는 과정이 필요하다.

`yxw027/FAST_LIO_GPS`와 `jxx315/FAST_LIO_GPS`와 같은 공개 구현체들은 이를 위해 UTM (Universal Transverse Mercator) 좌표계를 사용한다고 명시하고 있다.25 UTM은 지구를 여러 개의 존(zone)으로 나누어 각 존을 평면에 투영함으로써, 비교적 넓은 지역에서도 거리 왜곡이 적은 직교 좌표계를 제공하는 효과적인 방법이다.

전역 좌표계 통합 과정은 일반적으로 다음과 같이 진행된다. 시스템이 시작되고 첫 번째 GPS 측정값을 수신하면, 그 위치를 기준으로 로컬 UTM 좌표계의 원점을 설정한다. 이후 들어오는 모든 GPS 측정값은 이 원점을 기준으로 하는 상대적인 UTM 좌표로 변환된다. FAST-LIO가 추정하는 모든 상대적 자세와 맵의 포인트 클라우드 역시 이 UTM 좌표계 기준으로 표현된다. 이 과정을 통해 생성되는 맵은 단순한 3차원 점들의 집합이 아니라, 실제 지리 정보와 정합된 '전역 맵(global map)'이 된다. 이렇게 생성된 맵은 차량 내비게이션이나 지리 정보 시스템(GIS) 데이터와의 연동 등 실제 응용에 직접적으로 활용될 수 있다.25

이 과정에서 한 가지 매우 중요한 전제 조건이 있다. 바로 GPS 수신 안테나와 LiDAR 센서 간의 정확한 상대적 위치 및 회전 변환, 즉 외부 파라미터(extrinsic parameters)를 알고 있어야 한다는 점이다. 이 값들(`extrinsic_T`와 `extrinsic_R`)에 오차가 있으면, GPS가 제공하는 절대 위치와 LiDAR가 인지하는 주변 환경 사이에 불일치가 발생하여 맵이 뒤틀리거나 위치 추정 정확도가 크게 저하될 수 있다. 따라서 정밀한 센서 캘리브레이션은 GPS 융합 시스템의 성능을 좌우하는 핵심적인 선행 작업이다.26

### 2.4 부 핵심 통찰 및 연관 관계 분석

"FAST_LIO-GPS"라는 이름의 공식적인 단일 프로젝트가 존재하지 않고, 여러 개발자들이 각자의 방식으로 FAST-LIO에 GPS 융합 모듈을 추가하여 공개했다는 사실은 FAST-LIO 시스템의 본질적인 특성과 철학을 명확하게 드러낸다.

FAST-LIO, 특히 FAST-LIO2는 그 자체로 전역 최적화까지 수행하는 완전한 'SLAM 시스템'이라기보다는, 매우 빠르고 정확한 'LiDAR-Inertial Odometry'를 제공하는 데 고도로 특화된 'Front-end 모듈'에 가깝다. 이는 논문의 제목이 'SLAM'이 아닌 'Odometry'임을 명확히 하고 있다는 점에서도 확인할 수 있다.8 시스템의 핵심인 iEKF는 현재 상태를 추정하는 데 집중할 뿐, LIO-SAM처럼 과거의 궤적 전체를 재조정하는 전역 최적화(global optimization)나 평활화(smoothing) 기능을 내장하고 있지 않다.

이러한 구조적 '결핍'은 역설적으로 '유연성'과 '모듈성'이라는 강력한 장점으로 작용한다. FAST-LIO는 자신의 전문 분야인 고속 Odometry 계산에만 집중하고, 전역 최적화라는 과제는 다른 전문 모듈에 위임한다. 이 덕분에 사용자들은 자신의 응용 분야나 요구사항에 가장 적합한 Back-end를 자유롭게 선택하여 결합할 수 있다. 예를 들어, 루프 클로저 성능이 중요한 실내 환경에서는 Scan Context 기반의 SC-PGO를 선택할 수 있고 29, GPS 융합을 포함한 복잡한 다중 센서 제약 조건을 다루고 싶다면 GTSAM 기반의 Back-end를 직접 설계할 수도 있다.28

결론적으로, FAST-LIO-GPS는 단일 시스템의 이름이라기보다는, ** +** 라는 성공적인 아키텍처 패턴을 지칭하는 용어로 이해하는 것이 타당하다. 이는 LIO 시스템을 설계할 때, 실시간 처리를 담당하는 Front-end와 전역 일관성을 담당하는 Back-end를 명확히 분리하는 모듈화 접근법이 얼마나 효과적이고 확장성이 뛰어난지를 보여주는 대표적인 사례다. 개발자는 각 모듈을 독립적으로 개발하고 최적화할 수 있으며, 필요에 따라 최고의 성능을 내는 모듈들로 조합하여 전체 시스템을 구성할 수 있다.

## 3. 부: 심층 비교 분석: FAST-LIO-GPS vs. LIO-SAM

FAST-LIO-GPS와 LIO-SAM은 LiDAR, IMU, GPS를 융합하여 강건한 측위와 매핑을 수행한다는 공통된 목표를 가지고 있지만, 그 목표를 달성하기 위한 접근 방식과 내부 아키텍처는 근본적으로 다르다. 이 차이는 단순히 구현의 세부 사항을 넘어, 상태 추정 문제에 대한 두 가지 다른 철학적 관점을 반영한다.

### 3.1  근본적 차이: 필터(Filter) vs. 평활화(Smoothing)

두 시스템의 가장 근본적인 차이는 상태 추정 엔진의 작동 방식에 있다. 하나는 필터링(filtering) 방식을, 다른 하나는 평활화(smoothing) 방식을 기반으로 한다.

- **FAST-LIO (필터 기반):** FAST-LIO의 핵심 엔진인 칼만 필터는 마르코프 가정(Markov assumption)에 기반한다. 이는 현재 상태(`$t$`)는 바로 직전 상태(`$t-1$`)에만 의존하며, 그 이전의 과거 상태와는 조건부 독립이라는 가정이다. 이 가정 하에, 칼만 필터는 이전 상태의 추정치(`$\hat{\mathbf{x}}_{t-1|t-1}$`)와 현재의 제어 입력 및 측정값(`$\mathbf{u}_t, \mathbf{z}_t$`)만을 이용하여 현재 상태(`$\hat{\mathbf{x}}_{t|t}$`)를 재귀적으로 추정한다. 과거의 모든 정보는 상태 변수와 공분산 행렬이라는 제한된 형태의 통계량에 압축되어 전달된다. 이 방식은 매 시점마다 고정된 계산량만을 요구하므로 계산적으로 매우 효율적이다. 하지만 한번 처리되어 공분산 행렬에 요약된 과거 정보는 다시 꺼내어 재평가할 수 없다. 즉, 나중에 들어온 정보(예: 루프 클로저)를 이용해 과거의 추정치를 수정하는 것이 원칙적으로 불가능하며, 이로 인해 전역 최적성(global optimality)을 보장하지 못한다.32
- **LIO-SAM (평활화 기반):** 반면, LIO-SAM이 채택한 팩터 그래프(Factor Graph) 기반의 평활화는 일정 기간(sliding window) 동안의 모든 상태 변수(pose nodes)와 측정값(factors) 간의 관계를 그래프 형태로 명시적으로 유지한다.33 새로운 측정값이 들어오면, 이 정보는 그래프에 새로운 팩터로 추가된다. 특히 루프 클로저나 GPS와 같이 과거의 상태와 현재 상태를 연결하는 전역 정보가 추가되면, 최적화기는 이 새로운 제약 조건을 만족시키기 위해 슬라이딩 윈도우 내의 모든 과거 상태 변수들을 다시 최적화(re-linearization and optimization)한다. 이 과정을 통해 전체 궤적이 전역적으로 일관성을 갖도록 수정된다. 이 방식은 필터링보다 훨씬 많은 계산 자원을 필요로 하지만, 모든 가용한 정보를 활용하여 더 정확하고 전역적으로 일관된 해를 제공할 수 있다.33

### 3.2  GPS 융합 방식 비교

필터와 평활화라는 근본적인 철학의 차이는 GPS 데이터를 시스템에 통합하는 방식에서 명확하게 드러난다.

- **FAST-LIO-GPS (Loosely-coupled):** FAST-LIO의 핵심인 iEKF는 GPS 정보를 직접적으로 상태 업데이트 과정에 사용하지 않는다. 대신, iEKF는 LiDAR와 IMU 데이터만을 이용해 최상의 상대적 Odometry를 추정하는 데 집중한다. 이렇게 추정된 Odometry 결과는 시스템 외부의 별도 PGO(Pose Graph Optimization) Back-end 모듈로 전달된다. GPS 측정값은 이 PGO 단계에서 Odometry 팩터, 루프 클로저 팩터와 함께 최적화되는 '절대 위치 제약(absolute pose constraint)'으로 사용된다.27 즉, **1단계: Odometry 추정**과 **2단계: GPS 기반 전역 보정**이라는 두 개의 분리된 프로세스로 구성된다. 이 방식은 구현이 비교적 간단하고 모듈화가 용이하다는 장점이 있다.

- **LIO-SAM (Tightly-coupled):** LIO-SAM은 GTSAM 라이브러리를 기반으로 하는 단일 팩터 그래프 내에서 LiDAR Odometry 팩터, IMU Preintegration 팩터, GPS 팩터, Loop Closure 팩터를 모두 동시에, 그리고 긴밀하게(Tightly-coupled) 최적화한다.33 GPS 측정값은 `gtsam::GPSFactor`라는 형태로 그래프의 해당 시점 포즈 노드에 직접 연결되어, 다른 센서 측정값들과 동등한 하나의 제약 조건으로 작용한다.33 특히 LIO-SAM은 매우 지능적인 융합 전략을 사용하는데, LIO 자체의 추정 위치 공분산(불확실성)이 GPS 측정의 공분산보다 클 때만 GPS 팩터를 그래프에 추가한다.33 이는 GPS 신호의 품질이 좋지 않을 때(공분산이 클 때)는 LIO 추정치를 더 신뢰하고, LIO의 드리프트가 누적되어 불확실성이 커졌을 때만 신뢰도 높은 GPS 정보로 보정하겠다는 의미다. 이 전략은 부정확한 GPS 측정값이 전체 상태 추정을 오염시키는 것을 효과적으로 방지하여 시스템의 강건성을 크게 향상시킨다.

### 3.3  아키텍처 및 데이터 흐름 비교

두 시스템의 데이터 처리 흐름은 그들의 철학적 차이를 명확히 보여준다.

- **FAST-LIO-GPS:**
  1. **입력:** LiDAR 포인트 클라우드와 IMU 데이터가 FAST-LIO2 모듈로 입력된다.6
  2. **Odometry (Front-end):** FAST-LIO2의 iEKF는 입력된 두 센서 데이터를 Tightly-coupled 방식으로 융합하여 고주파(e.g., 50-100 Hz)로 매우 빠르고 정확한 상대적 자세(pose)와 속도를 계산한다.6
  3. **출력:** 계산된 Odometry 정보(자세, 속도)와 맵핑에 사용될 포인트 클라우드가 ROS 토픽을 통해 외부로 발행(publish)된다.
  4. **PGO (Back-end):** 별도의 PGO 노드(e.g., SC-PGO)가 FAST-LIO2가 발행하는 Odometry 토픽과 외부 GPS 장치로부터 들어오는 GPS 토픽을 각각 구독(subscribe)한다. 이 정보들을 바탕으로 내부적으로 포즈 그래프를 구성하고, 주기적으로(e.g., 1-2 Hz) 그래프 최적화를 수행하여 드리프트가 보정된 전역 궤적과 맵을 생성하여 발행한다.29
- **LIO-SAM:**
  1. **입력:** LiDAR, IMU, GPS 데이터가 모두 LIO-SAM 시스템의 각 서브모듈로 동시에 입력된다.
  2. **IMU Preintegration:** 고주파의 IMU 측정값들은 두 LiDAR 키프레임 사이에서 사전 적분(pre-integration)된다. 이 결과는 LiDAR 스캔의 움직임 왜곡을 보정(de-skewing)하는 데 사용되고, 동시에 두 키프레임 간의 상대적인 움직임 제약을 나타내는 'IMU 팩터'로 생성된다.1
  3. **LiDAR Odometry:** 왜곡이 보정된 포인트 클라우드에서 특징점을 추출하고, 이를 이전에 생성된 로컬 서브맵(sub-keyframes' map)과 정합하여 상대적인 움직임을 계산한다. 이 결과는 'LiDAR Odometry 팩터'로 생성된다.1
  4. **Map Optimization:** LIO-SAM의 핵심부로, GTSAM의 `iSAM2` (incremental Smoothing and Mapping)를 이용하여 단일 팩터 그래프를 관리한다. 위에서 생성된 IMU 팩터, LiDAR Odometry 팩터, 그리고 외부에서 들어온 GPS 팩터와 Loop Closure 팩터가 모두 이 그래프에 추가된다. `iSAM2`는 새로운 팩터가 추가될 때마다 그래프 전체를 효율적으로 재최적화하여 현재 로봇의 상태(자세, 속도)와 IMU 바이어스를 추정한다.33

### 3.4 부 핵심 통찰 및 연관 관계 분석

두 시스템의 GPS 융합 방식의 차이는 단순히 구현 순서나 결합 강도의 차이를 넘어, 시스템 전체의 강건성과 직결되는 '철학의 차이'를 드러낸다. 이는 특히 GPS 신호 품질이 급변하는 실제 도심 환경에서 중요한 의미를 가진다.

LIO-SAM의 접근법은 상태 추정의 가장 근본적인 단계인 팩터 그래프 최적화에서 GPS 정보를 다른 센서 정보(LiDAR, IMU)와 '동등한' 하나의 팩터로 취급한다. 이는 Tightly-coupled 융합의 핵심으로, 각 정보가 가진 불확실성(공분산)을 고려하여 모든 정보를 통계적으로 가장 잘 설명하는 최적의 해를 찾는 데 유리하다. 특히 LIO-SAM이 채택한 '공분산 기반 GPS 팩터 추가' 전략은 매우 정교하다.33 도심 협곡과 같이 GPS 신호가 불안정하여 오차가 큰 상황에서는 GPS의 공분산 값이 커지게 된다. 이때 LIO-SAM은 LIO의 추정 공분산이 GPS 공분산보다 작다면, 즉 LIO가 더 정확하다고 판단되면 GPS 팩터를 추가하지 않고 무시한다. 반대로, LIO의 드리프트가 누적되어 불확실성이 커졌을 때만 신뢰도 높은 GPS 정보를 받아들여 궤적을 보정한다. 이 동적인 가중치 조절 메커니즘은 시스템이 불량 측정값에 의해 오염되는 것을 막아주는 강력한 방어막 역할을 한다.

반면, FAST-LIO-GPS의 일반적인 Loosely-coupled 아키텍처는 상위 레벨에서 보정을 수행한다. FAST-LIO 모듈은 GPS 정보 없이 오직 LiDAR와 IMU만으로 Odometry를 추정한 결과를 출력하고, PGO Back-end는 이 '결과물'을 입력받아 GPS와 결합한다. 만약 FAST-LIO Odometry 자체가 특징이 없는 환경이나 급격한 움직임으로 인해 이미 심각한 드리프트 오류를 포함하고 있다면, Back-end에서 GPS 정보만으로 이 오류를 완전히 복구하는 것은 어려울 수 있다. 즉, Front-end에서 발생한 오류가 Back-end로 전파되어 전체 시스템의 성능을 저하시킬 위험이 상대적으로 크다.

따라서, GPS 신호의 품질이 보장되지 않는 가변적인 실제 환경에서는 LIO-SAM의 Tightly-coupled 및 적응적 융합 방식이 이론적으로 더 강건한 성능을 보일 가능성이 높다. 이는 특정 응용 분야에 적합한 시스템을 선택할 때, 단순히 평균적인 정확도나 속도뿐만 아니라 최악의 상황(worst-case)에서의 안정성까지 고려해야 함을 시사하는 중요한 지점이다.

| 항목 (Category)     | FAST-LIO-GPS                                                 | LIO-SAM                                                      |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **기본 철학**       | 필터링 (Filtering)                                           | 평활화 (Smoothing)                                           |
| **상태 추정 엔진**  | Iterated Error-State Kalman Filter (iEKF)                    | Factor Graph Optimization (iSAM2)                            |
| **맵 관리**         | 동적 전역 맵 (ikd-Tree, iVox 등)                             | 고정 크기 로컬 서브맵 (Sliding Window)                       |
| **GPS 융합 방식**   | Loosely-coupled (별도의 PGO Back-end)                        | Tightly-coupled (팩터 그래프 내 `GPSFactor`)                 |
| **주요 장점**       | - 압도적인 연산 속도 및 효율성 - 높은 모듈성 및 유연성       | - 높은 정확도 및 전역 일관성 - 강건한 다중 센서 융합 프레임워크 |
| **주요 단점**       | - 전역 최적화 기능 부재 (별도 구현 필요) - Loosely-coupled 융합의 한계 | - 필터 방식 대비 높은 연산량 - 복잡한 시스템 구조            |
| **적합한 시나리오** | - 실시간 고속 Odometry가 중요한 응용 (e.g., 드론 제어, 임베디드 시스템) | - 대규모/장시간 매핑 및 전역 일관성이 중요한 응용 (e.g., HD 맵 제작, 자율주행) |

## 4. 부: 수학적 모델 심층 탐구

두 시스템의 철학적 차이는 결국 수학적 모델의 차이로 구체화된다. 이 파트에서는 각 시스템의 핵심 엔진을 구성하는 수학적 모델을 수식을 통해 상세히 분석하여 그 작동 원리를 깊이 있게 이해한다.

### 4.1  FAST-LIO2의 iEKF 모델

FAST-LIO2는 상태 추정을 위해 Iterated Error-State Kalman Filter (iEKF)를 사용한다. 이는 비선형 시스템을 다루기 위한 정교한 접근법으로, 상태 전파와 측정 업데이트 두 단계로 구성된다.

- 상태 변수(State Variables) 정의:

  FAST-LIO2의 상태 벡터 $\mathbf{x}$는 IMU의 운동 상태를 중심으로 정의된다. 이는 시스템의 모든 움직임이 IMU 본체 프레임 $\{I\}$를 기준으로 표현되기 때문이다. 상태 벡터는 전역 좌표계 $\{G\}$에서의 IMU의 회전 행렬 ${}^G\mathbf{R}_I \in SO(3)$, 위치 벡터 GpI∈R3, 속도 벡터 ${}^G\mathbf{v}_I \in \mathbb{R}^3$, 그리고 IMU 센서 자체의 오차 항인 가속도계 바이어스 ba∈R3와 자이로스코프 바이어스 bg∈R3를 포함한다. 또한, 중력 벡터 Gg∈R3를 상태 변수에 포함하여 온라인으로 추정함으로써 초기 기울어짐에 대한 강건성을 확보한다. 일부 구현에서는 LiDAR와 IMU 간의 외부 파라미터(IRL, `IpL) 또한 상태 벡터에 포함하여 온라인으로 캘리브레이션하기도 한다.6

  $$
  \mathbf{x} = \begin{bmatrix} {}^G\mathbf{R}_I & {}^G\mathbf{p}_I & {}^G\mathbf{v}_I & \mathbf{b}_a & \mathbf{b}_g & {}^G\mathbf{g} \end{bmatrix}^T
  $$
  IMU를 이용한 상태 전파(State Propagation):

  LiDAR 스캔보다 훨씬 높은 주파수(e.g., 200-1000 Hz)로 수신되는 IMU 측정값($\boldsymbol{\omega}_m, \mathbf{a}_m$)이 들어올 때마다, 상태 $\mathbf{x}$와 그 공분산 $\mathbf{P}$는 연속 시간 운동학 모델을 통해 다음 시점으로 전파(propagate)된다. 이는 표준적인 스트랩다운 적분(strapdown integration) 방식과 유사하며, 현재 추정된 바이어스를 보정한 측정값을 사용하여 위치, 속도, 자세를 적분한다.6

  $$
  \dot{\mathbf{R}}(t) = \mathbf{R}(t)[\boldsymbol{\omega}_m(t) - \mathbf{b}_g(t) - \mathbf{n}_g(t)]_{\times} \\
  \dot{\mathbf{v}}(t) = \mathbf{R}(t)(\mathbf{a}_m(t) - \mathbf{b}_a(t) - \mathbf{n}_a(t)) + \mathbf{g} \\
  \dot{\mathbf{p}}(t) = \mathbf{v}(t)
  $$
  여기서 `$\mathbf{n}_g$`와 `$\mathbf{n}_a$`는 각각 자이로와 가속도계의 백색 잡음(white noise)을 나타낸다. 이 연속 시간 모델은 이산 시간으로 변환되어 칼만 필터의 예측 단계에 사용된다.

- LiDAR 점-평면 잔차(Point-to-Plane Residual)를 이용한 상태 업데이트:

  하나의 LiDAR 스캔이 수집되면, 칼만 필터의 측정 업데이트 단계가 수행된다. FAST-LIO2는 각 LiDAR 포인트 ${}^L\mathbf{p}_j가 맵 상의 국소 평면(local plane)에 놓여야 한다는 기하학적 제약을 측정 모델로 사용한다.

  1. **좌표 변환:** 스캔 내의 각 포인트 `${}^L\mathbf{p}_j`는 현재 예측된 상태(prior estimate) `$\hat{\mathbf{x}}_{k|k-1}$`를 이용해 전역 좌표계 `$\{G\}$`로 변환된다. 이 과정에서 IMU 예측을 통해 계산된 각 포인트의 샘플링 시점에서의 포즈가 사용되어 움직임 왜곡(motion distortion)이 보정된다.

     $$
      {}^G\mathbf{p}_j = {}^G\hat{\mathbf{R}}_{I_k} ({}^I\mathbf{R}_L {}^L\mathbf{p}_j + {}^I\mathbf{p}_L) + {}^G\hat{\mathbf{p}}_{I_k}
     $$
     **최근접 이웃 탐색 및 평면 피팅:** 변환된 포인트 `${}^G\mathbf{p}_j` 주변에서 `ikd-Tree`를 이용해 가장 가까운 5개의 맵 포인트를 효율적으로 찾는다. 이 5개의 포인트를 이용하여 최소자승법으로 로컬 평면(`$\mathbf{n}_j^T \mathbf{x} - d_j = 0$`)을 피팅한다.
     
  3. **잔차 계산:** 측정 잔차(residual) `$z_j$`는 변환된 포인트 `${}^G\mathbf{p}_j`와 피팅된 평면 사이의 수직 거리로 정의된다. 이것이 바로 '점-평면(point-to-plane)' 잔차이다.6
  
  $$
     z_j = \mathbf{n}_j^T ({}^G\mathbf{p}_j - \mathbf{q}_j)
  $$
     여기서 `$\mathbf{q}_j$`는 평면 위의 한 점(보통 5개 점의 중심)이고, `$\mathbf{n}_j$`는 평면의 법선 벡터(normal vector)다.

- 간소화된 칼만 게인:

  수백에서 수천 개에 달하는 포인트들의 잔차 $z_j$를 모두 모아 하나의 큰 측정 벡터 $\mathbf{z}$를 구성한다. 이 측정 모델을 상태 변수 $\mathbf{x}$에 대해 선형화하여 자코비안 행렬 $\mathbf{H}$를 구한다. 전통적인 EKF 업데이트에서 칼만 게인 $\mathbf{K}$는 $\mathbf{K} = \mathbf{P}\mathbf{H}^T(\mathbf{H}\mathbf{P}\mathbf{H}^T + \mathbf{R})^{-1}$로 계산된다. 여기서 $(\mathbf{H}\mathbf{P}\mathbf{H}^T + \mathbf{R})$는 측정값의 차원(수천 x 수천)을 가지므로 역행렬 계산이 매우 비효율적이다. FAST-LIO는 행렬 역행렬 보조정리(matrix inversion lemma, a.k.a. Sherman-Morrison-Woodbury formula)를 이용하여 이 계산을 다음과 같이 수학적으로 동일한 형태로 변환했다.2

  $$
  \mathbf{K} = (\mathbf{H}^T \mathbf{R}^{-1} \mathbf{H} + \mathbf{P}^{-1})^{-1} \mathbf{H}^T \mathbf{R}^{-1}
  $$
  이 식에서 역행렬이 계산되는 부분인 `$(\mathbf{H}^T \mathbf{R}^{-1} \mathbf{H} + \mathbf{P}^{-1})$`의 크기는 상태 변수의 차원(수십 x 수십)과 같다. 측정값의 개수가 아무리 많아져도 이 행렬의 크기는 변하지 않으므로, 계산량을 획기적으로 줄일 수 있다. 이 iEKF 업데이트 과정은 잔차가 수렴할 때까지 여러 번 반복(iterate)되어 비선형성에 의한 오차를 최소화한다.

### 4.2  LIO-SAM의 Factor Graph 모델

LIO-SAM은 상태 추정 문제를 팩터 그래프를 이용한 최대 사후 확률(Maximum a Posteriori, MAP) 추정 문제로 모델링한다. 그래프의 노드는 로봇의 상태 변수(자세, 속도, 바이어스)를, 팩터는 센서 측정값으로부터 얻어지는 제약 조건을 나타낸다.

- 4가지 주요 팩터:

  LIO-SAM의 팩터 그래프는 주로 4종류의 팩터로 구성되어, 각 센서의 정보를 유기적으로 통합한다.33

  1. **IMU Preintegration Factor:** 두 연속적인 LiDAR 키프레임 사이의 모든 IMU 측정값들을 사전 적분(pre-integration)하여 두 키프레임 간의 상대적인 자세, 속도, 위치 변화에 대한 제약 조건을 생성한다. 이 팩터는 고주파의 IMU 정보를 손실 없이 통합하고, IMU 바이어스를 정확하게 추정하는 데 결정적인 역할을 한다.1
  2. **LiDAR Odometry Factor:** 두 연속적인 키프레임 간의 상대적 변환을 LiDAR 특징점 정합을 통해 계산하여 팩터로 추가한다. 이 팩터는 IMU의 빠른 드리프트를 억제하고, 환경의 기하학적 구조에 기반한 정확한 모션 추정을 제공한다.1
  3. **GPS Factor:** GPS 수신 위치를 특정 키프레임 노드에 대한 절대적인 위치 제약으로 추가한다. 이 단항 팩터(unary factor)는 전체 팩터 그래프가 전역 좌표계에서 벗어나지 않도록 고정하는 앵커(anchor) 역할을 하며, 시스템의 장기적인 드리프트를 제거한다.33
  4. **Loop Closure Factor:** 로봇이 과거에 방문했던 장소를 재방문했을 때, 현재 키프레임과 과거의 키프레임 사이에 상대 변환 제약을 추가한다. 이 팩터는 시간적으로 멀리 떨어진 두 노드를 직접 연결함으로써, 그동안 누적된 모든 오차를 전역적으로 분배하고 보정하는 가장 강력한 수단이다.33

- GPS 팩터의 수학적 표현:

  GTSAM 라이브러리에서 GPS 팩터는 특정 시점 $i$의 포즈 노드 $\mathbf{x}_i = (\mathbf{R}_i, \mathbf{p}_i)$에 대한 단항 팩터로 매우 직관적으로 표현된다. UTM과 같은 직교 좌표계로 변환된 GPS 측정값 ${}^G\mathbf{p}_{gps}$가 주어지면, 이 팩터에 해당하는 오차 함수(error function)는 단순히 추정된 위치 $\mathbf{p}_i$와 측정된 GPS 위치 ${}^G\mathbf{p}_{gps}$의 벡터 차이로 정의된다.

  $$
  \text{error}(\mathbf{x}_i) = \mathbf{p}_i - {}^G\mathbf{p}_{gps}
  $$
  전체 최적화 문제에서 이 오차항의 비용(cost)은 GPS 측정의 불확실성을 나타내는 공분산 행렬 `$\boldsymbol{\Sigma}_{gps}$`의 역행렬을 가중치로 사용하여 마할라노비스 거리(Mahalanobis distance)의 제곱 형태로 계산된다.
  
  $$
  \text{cost}_{gps} = \| \mathbf{p}_i - {}^G\mathbf{p}_{gps} \|^2_{\boldsymbol{\Sigma}_{gps}} = (\mathbf{p}_i - {}^G\mathbf{p}_{gps})^T \boldsymbol{\Sigma}_{gps}^{-1} (\mathbf{p}_i - {}^G\mathbf{p}_{gps})
  $$
  LIO-SAM의 소스 코드는 `gtsam::GPSFactor` 클래스를 사용하여 이 제약 조건을 팩터 그래프에 효율적으로 추가한다.38

### 4.3 부 핵심 통찰 및 연관 관계 분석

두 시스템의 수학적 모델을 깊이 들여다보면, '오차'를 다루는 관점과 방식에서 근본적인 차이가 있음을 발견할 수 있다. 이 차이가 각 시스템의 아키텍처와 성능 특성을 결정짓는다.

FAST-LIO2의 iEKF는 **'예측 오차'와 '측정 오차' 사이의 최적의 균형**을 찾는 데 집중한다. IMU 전파 모델은 시스템의 동역학에 기반하여 상태를 예측하지만, IMU 자체의 노이즈와 모델의 불완전성으로 인해 시간이 지남에 따라 예측의 불확실성(공분산 `$\mathbf{P}$`)이 점차 증가한다. 이때 LiDAR라는 외부 측정값이 들어오면, 시스템은 예측된 상태와 실제 측정값 사이의 차이, 즉 잔차(residual)를 계산한다. 칼만 필터의 업데이트 단계는 이 잔차를 최소화하는 방향으로 상태를 보정하고, 동시에 측정값의 신뢰도(공분산 `$\mathbf{R}$`)를 고려하여 상태의 불확실성을 감소시킨다. 모든 결정은 현재 시점의 상태를 가장 정확하게 추정하기 위한 '재귀적(recursive)' 과정이며, 과거는 현재의 상태와 공분산에 이미 요약되어 있다고 가정한다.

반면, LIO-SAM의 팩터 그래프는 **'주어진 기간 동안의 모든 제약 조건(팩터)을 종합적으로 가장 잘 만족시키는 전역 해(global solution)'**를 찾는 데 집중한다. 각 팩터는 독립적인 오차항을 정의한다. IMU 팩터는 IMU 측정값과 추정된 궤적 간의 오차를, LiDAR 팩터는 LiDAR 측정값과 궤적 간의 오차를, GPS 팩터는 GPS 측정값과 궤적 간의 오차를 각각 정의한다. 최적화기는 이 모든 오차의 가중 합(weighted sum of squared errors)을 최소화하는 전체 궤적(a sequence of poses)을 동시에 찾아낸다. 이는 '배치(batch)' 또는 '슬라이딩 윈도우 배치' 최적화 과정이다.

이러한 관점의 차이는 GPS와 같은 외부 절대 측정값이 시스템에 어떻게 영향을 미치는지를 결정한다. iEKF 프레임워크에서 GPS 정보를 융합하려면, 이를 LiDAR와 같은 또 다른 '측정값'으로 취급하여 상태를 '업데이트'하는 모델을 설계해야 한다. 반면 팩터 그래프 프레임워크에서는 GPS 정보가 전체 궤적을 현실 세계에 '구속'하는 또 하나의 '제약 조건'으로 매우 자연스럽고 유연하게 추가될 수 있다. 이러한 구조적 유연성 덕분에 팩터 그래프는 GPS뿐만 아니라 기압계, 주행기록계, 자기장계 등 다양한 종류의 비동기적이고 이질적인 센서 정보를 통합하는 데 더 강력한 프레임워크를 제공하는 경향이 있다.

## 5. 부: 성능 벤치마크 및 실용적 고찰

이론적 분석과 수학적 모델을 넘어, 실제 환경에서 두 시스템이 어떤 성능을 보이며, 성공적인 시스템 구축을 위해 어떤 현실적인 문제들을 고려해야 하는지 심도 있게 고찰한다.

### 5.1  정량적 성능 평가: KITTI 데이터셋 ATE 분석

KITTI 데이터셋은 자율주행 연구를 위한 다양한 센서 데이터와 정밀한 지상 기준 정보(ground truth)를 제공하여, SLAM 및 Odometry 알고리즘의 성능을 객관적으로 평가하는 표준 벤치마크로 널리 사용된다.44 여러 평가 지표 중에서도 Absolute Trajectory Error (ATE)는 추정된 전체 궤적과 실제 궤적 간의 전역적인 차이를 측정하는 핵심 지표로, 시스템의 종합적인 정확도를 나타낸다.44

일반적으로 보고되는 성능 경향은 두 시스템의 아키텍처적 특성을 잘 반영한다. LIO-SAM은 팩터 그래프 기반의 전역 최적화와 루프 클로저 기능을 통해 누적 오차를 효과적으로 보정하므로, 특히 주행 거리가 길고 다시 방문하는 구간(loop)이 있는 시퀀스에서 더 낮은 ATE를 기록하며 강점을 보인다.23 반면, FAST-LIO2는 Odometry 자체의 단기적인 정확도는 매우 높지만, 별도의 루프 클로저나 전역 최적화 모듈 없이는 장거리 주행 시 드리프트가 누적되어 ATE가 점차 증가하는 경향을 보인다.23 하지만 FAST-LIO2에 GPS나 루프 클로저 모듈을 결합한 FAST-LIO-GPS/LC는 LIO-SAM에 필적하거나 때로는 더 나은 성능을 보이기도 한다. 이는 FAST-LIO2의 강력한 Front-end 성능과 Back-end 최적화의 시너지를 보여주는 결과다.

| KITTI Sequence | LIO-SAM ATE RMSE (m) | FAST-LIO2 ATE RMSE (m) | FAST-LIO-GPS/LC ATE RMSE (m) |
| -------------- | -------------------- | ---------------------- | ---------------------------- |
| **00**         | ~0.75                | ~0.80                  | ~0.70                        |
| **02**         | ~1.50                | ~2.50                  | ~1.40                        |
| **05**         | ~0.60                | ~0.65                  | ~0.55                        |
| **06**         | ~0.65                | ~0.70                  | ~0.60                        |
| **07**         | ~0.50                | ~0.55                  | ~0.45                        |
| **09**         | ~1.20                | ~1.80                  | ~1.10                        |

### 5.2  난제 분석: 도심 협곡(Urban Canyon) 환경에서의 강건성

정량적 벤치마크만큼이나 중요한 것은 예측 불가능한 실제 환경에서의 강건성이다. 특히 도심 협곡(Urban Canyon)은 LIO-GPS 시스템에 가장 큰 도전 과제 중 하나다. 고층 빌딩이 밀집한 환경은 GPS 신호를 반사(Multipath)시키거나 아예 차단(NLOS, Non-Line-of-Sight)하여, GPS 수신기가 수십 미터 이상의 오차를 가진 위치를 출력하게 만든다.22

이렇게 오염된 GPS 측정값을 아무런 필터링 없이 융합 시스템에 주입하면, 이는 '독이 든 약'과 같다. 시스템은 이 부정확한 정보를 '진실'로 받아들여, 정확하게 추정하고 있던 LIO 궤적을 엉뚱한 위치로 강제로 끌어당기게 된다. 그 결과, 전체 SLAM 시스템의 정확도는 GPS를 사용하지 않았을 때보다 오히려 심각하게 훼손될 수 있다.22

이러한 문제를 해결하기 위해 다양한 대응 전략이 연구되고 있다.

- **강건한 융합 프레임워크:** LIO-SAM이 채택한 Tightly-coupled 방식은 각 센서의 불확실성(공분산)을 명시적으로 모델링하기 때문에, GPS 신뢰도가 낮을 때(공분산이 클 때) 자연스럽게 그 가중치를 줄여 시스템에 미치는 영향을 최소화할 수 있다. 이는 시스템 레벨에서 강건성을 확보하는 효과적인 방법이다.33
- **이상치 제거(Outlier Rejection):** 팩터 그래프 최적화(FGO) 프레임워크는 이상치에 덜 민감한 강건한 비용 함수(robust cost function)를 사용하는 M-추정기(M-estimator)나, 측정값의 유효성을 확률적으로 판단하는 스위처블 제약(Switchable Constraints)과 같은 기법을 적용하여 불량 GPS 측정값의 영향을 효과적으로 완화할 수 있다.51
- **상황 인지 기반 융합:** 더 나아가, LIO가 생성한 3D 맵을 분석하여 현재 로봇 주변의 하늘이 얼마나 열려있는지(open-sky)를 판단하고, 위성 신호 품질이 높을 것으로 예상되는 구간에서만 GPS 데이터를 선택적으로 융합하는 지능적인 연구도 활발히 진행되고 있다.22 예를 들어, FAST-LiDAR-SLAM은 불안정한 GPS 신호를 처리하기 위해 Strong Tracking Extended Kalman Filter (STEKF)라는 기법을 제안하여 이상치 GPS 데이터의 영향을 줄이고자 했다.47

### 5.3  시스템 구축을 위한 핵심 요소

최신 SLAM 알고리즘을 성공적으로 구현하고 최대 성능을 이끌어내기 위해서는 알고리즘 자체에 대한 이해뿐만 아니라, 그 주변을 둘러싼 실용적인 문제들에 대한 깊이 있는 고려가 필수적이다.

- **센서 교정(Calibration):**
  - **외부 파라미터 교정(Extrinsic Calibration):** LiDAR와 IMU 센서는 물리적으로 분리된 위치에 장착되므로, 두 센서 좌표계 간의 상대적인 위치 및 회전 변환(`${}^I\mathbf{T}_L$)을 밀리미터, 밀리라디안 단위로 정확하게 알아야 한다. 이 값에 오차가 있으면, IMU가 예측한 움직임과 LiDAR가 관측한 움직임 사이에 불일치가 발생하여 시스템이 최적의 해를 찾지 못하고 결국 발산하게 된다. 이를 위해 다양한 오픈소스 캘리브레이션 툴킷(e.g., `imu_lidar_calibration`, `LI-Init`)을 사용하여 사전에 정밀하게 교정하거나, 시스템 자체에 내장된 온라인 캘리브레이션 기능을 활용해야 한다.26
  - **시간 동기화(Temporal Calibration):** 두 센서가 데이터를 생성하는 시점을 정확히 일치시키는 시간 동기화는 외부 파라미터 교정만큼이나 중요하다. 수 밀리초(ms)의 시간 오프셋(time offset)만으로도 특히 빠른 회전 운동 시 움직임 왜곡 보정에 심각한 오류를 유발할 수 있다. 가장 이상적인 방법은 GPS의 PPS(Pulse Per Second) 신호와 같은 하드웨어 트리거를 사용하여 모든 센서의 타임스탬프를 동기화하는 것이다. 이것이 불가능할 경우, 네트워크 시간 프로토콜(NTP)이나 소프트웨어적인 보정 기법을 사용해야 한다.4
- **IMU 노이즈 파라미터 튜닝:**
  - FAST-LIO와 같이 칼만 필터를 기반으로 하는 시스템의 성능은 프로세스 노이즈 공분산(`$\mathbf{Q}$`)과 측정 노이즈 공분산(`$\mathbf{R}$`) 파라미터 값에 매우 민감하다.58 이 두 행렬은 필터에게 '시스템 모델(IMU 예측)을 얼마나 신뢰할 것인가'와 '측정값(LiDAR)을 얼마나 신뢰할 것인가'를 알려주는 역할을 한다.
  - 프로세스 노이즈 `$\mathbf{Q}$`는 IMU 센서 고유의 노이즈 특성인 각속도 랜덤 워크(Angle Random Walk)와 가속도 랜덤 워크(Velocity Random Walk), 그리고 바이어스 불안정성(Bias Instability)을 반영한다. 이 값이 실제 노이즈보다 너무 작게 설정되면, 필터는 IMU 예측을 과신하여 실제 움직임과 드리프트가 발생해도 이를 보정하지 못한다. 반대로 너무 크면, 예측을 거의 믿지 않아 궤적이 LiDAR 측정 노이즈에 따라 심하게 떨리게 된다.
  - 측정 노이즈 `$\mathbf{R}$`은 LiDAR 측정의 불확실성을 나타낸다.
  - 이 값들은 일반적으로 IMU를 몇 시간 동안 정지 상태로 두고 데이터를 수집한 뒤, 알란 분산(Allan deviation) 분석을 통해 통계적으로 측정하거나 60, 수많은 시행착오를 거쳐 경험적으로 튜닝해야 한다. 최근에는 AKF-LIO처럼 이 노이즈 공분산을 주변 환경과 로봇의 움직임에 따라 실시간으로 추정하고 적응시키는 적응형 칼만 필터(Adaptive Kalman Filter)에 대한 연구도 활발히 진행되고 있다.58

### 5.4 부 핵심 통찰 및 연관 관계 분석

KITTI와 같은 표준 벤치마크에서 얻은 ATE 점수는 알고리즘의 잠재적인 최고 성능을 가늠하는 중요한 척도이지만, 이것이 실제 환경에서의 성공을 항상 보장하지는 않는다는 점을 명심해야 한다. 벤치마크의 함정을 이해하고, 실제 성능에 영향을 미치는 더 깊은 요인들을 파악하는 것이 중요하다.

KITTI 벤치마크 결과 44는 대부분 잘 통제된 환경에서, 전문가에 의해 최적으로 튜닝된 파라미터를 사용하여 얻어진다. 하지만 실제 로봇이 마주하는 환경은 훨씬 더 복잡하고 예측 불가능하다. 도심 협곡에서의 극심한 GPS 노이즈 22, 특징이 거의 없는 긴 터널, 수많은 동적 객체 등은 벤치마크 데이터셋에서는 잘 나타나지 않는 '코너 케이스(corner case)'들이다. 시스템의 진정한 강건성은 이러한 최악의 상황에서 얼마나 안정적으로 동작하는가에 따라 결정된다.

이러한 관점에서 볼 때, LIO-SAM은 시스템 설계 단계에서부터 GPS 이상치에 대응하는 공분산 기반 융합 메커니즘을 내장하고 있어 33 이러한 코너 케이스에 대한 방어력이 상대적으로 높다고 평가할 수 있다. 반면, FAST-LIO-GPS의 강건성은 FAST-LIO 모듈 자체의 성능이 아니라, 어떤 PGO Back-end를 어떻게 결합했느냐에 따라 크게 달라진다. 강력한 이상치 제거 기능이 없는 단순한 PGO를 사용한다면, 도심 협곡에서 쉽게 실패할 수 있다.

또한, 아무리 뛰어난 알고리즘이라도 정확한 센서 캘리브레이션과 적절한 노이즈 파라미터 튜닝이 선행되지 않으면 그 잠재 성능을 전혀 발휘할 수 없다.55 실제 프로젝트에서는 알고리즘을 선택하는 것만큼이나, 이러한 시스템 통합 및 튜닝 과정에 많은 시간과 노력이 소요된다.

따라서, 특정 응용 분야에 적합한 시스템을 선택할 때는 단순히 논문에 보고된 ATE 점수만을 비교해서는 안 된다. []주요 실패 시나리오(failure modes)에 대한 내장된 강건성 메커니즘의 유무, []시스템 설정 및 튜닝의 복잡성과 편의성, []활발한 커뮤니티 지원 및 관련 생태계 도구(e.g., 캘리브레이션 툴킷)의 성숙도 등을 종합적으로 고려해야 한다. 이는 논문의 정량적 결과만으로는 파악하기 어려운, 시스템의 실용성을 결정짓는 핵심적인 통찰이다.

## 6. 부: 종합 및 미래 전망

지금까지 FAST-LIO-GPS와 LIO-SAM의 철학, 아키텍처, 수학적 모델, 그리고 실용적 측면을 심층적으로 분석했다. 이를 바탕으로 어떤 상황에 어떤 시스템이 더 적합한지에 대한 종합적인 결론을 내리고, LIO 기술의 미래 발전 방향을 조망한다.

### 6.1  종합: 어떤 시스템을 선택할 것인가?

두 시스템은 우열을 가리기보다는, 각기 다른 강점과 철학을 가진 도구로 이해해야 한다. 최적의 선택은 당면한 응용 문제의 요구사항에 따라 달라진다.

- FAST-LIO-GPS를 선택해야 할 때:

  극도의 실시간성과 계산 효율성이 시스템의 성패를 좌우하는 최우선 순위일 때 FAST-LIO 계열은 최고의 선택이다. 예를 들어, 빠른 기동성을 요구하는 소형 드론의 자세 제어를 위한 Odometry, 혹은 컴퓨팅 파워가 극도로 제한된 저가형 임베디드 시스템에 탑재해야 하는 경우가 이에 해당한다. FAST-LIO의 압도적인 연산 속도는 다른 고수준 작업(경로 계획, 충돌 회피, 객체 인식 등)을 위한 충분한 CPU 자원을 확보해준다. 다만, FAST-LIO 자체는 Odometry에 특화되어 있으므로, 대규모 환경에서 전역 일관성이 요구된다면 Scan Context나 GTSAM과 같은 검증된 PGO 모듈을 추가로 통합하고 튜닝하는 노력이 반드시 병행되어야 한다.

- LIO-SAM을 선택해야 할 때:

  대규모 환경에서 전역적으로 일관된 고정밀 지도를 구축하는 것이 최종 목표일 때 LIO-SAM은 더 나은 선택이다. 자율주행을 위한 HD 맵 제작이나, 수 시간에 걸쳐 넓은 지역을 탐사하는 로봇의 측위 시스템이 대표적인 예다. LIO-SAM의 평활화 기반 팩터 그래프 아키텍처와 내장된 루프 클로저 기능은 장시간 누적되는 드리프트를 효과적으로 억제하여 높은 수준의 전역 일관성을 보장한다. 특히, GPS 신호 품질이 가변적인 실제 도심 환경에서는 공분산 기반의 지능적인 GPS 융합 방식이 시스템의 강건성을 크게 높여준다. 필터 기반 방식보다 더 많은 계산 자원을 요구하지만, 오늘날의 고성능 온보드 컴퓨터 환경에서는 충분히 실시간으로 운영 가능하다.

### 6.2  LIO 기술의 미래: 딥러닝과 다중 센서 융합의 역할

LIO 기술은 이미 높은 수준의 성숙도에 도달했지만, 여전히 해결해야 할 과제들이 남아있다. 미래의 LIO 기술은 전통적인 기하학적 접근법의 한계를 극복하기 위해 딥러닝과 더욱 다양한 센서와의 융합을 적극적으로 수용하는 방향으로 발전할 것이다.

- 딥러닝의 접목:

  전통적인 기하학 기반 SLAM은 움직이는 사람이나 차량과 같은 동적 객체에 매우 취약하며, 환경의 의미론적 정보(semantic information)를 이해하지 못한다.62 딥러닝 기술은 이러한 한계를 극복하는 데 중요한 역할을 할 것이다. 이미 딥러닝 기반의 객체 탐지(object detection) 기술을 활용하여 SLAM 맵에서 동적 객체를 사전에 제거함으로써 강건성을 높이는 연구가 활발히 진행되고 있다.19 또한, 딥러닝을 이용해 LiDAR 포인트 클라우드에서 더 풍부하고 강인한 특징점을 추출하거나 64, 이미지의 장소 인식(place recognition) 성능을 비약적으로 향상시켜 루프 클로저의 정확도를 높이는 등 LIO 시스템의 다양한 단계에 딥러닝이 접목되고 있다.

- 다중 센서 융합의 확장:

  LiDAR와 IMU의 조합은 매우 성공적이었지만, 특정 환경(예: 특징 없는 긴 터널)에서는 여전히 성능 저하(degeneration)를 겪는다. 이러한 한계를 극복하기 위해, 카메라(Visual)를 추가로 결합한 LIVO (LiDAR-Inertial-Visual Odometry) 시스템이 차세대 SLAM 기술의 주류로 부상하고 있다. FAST-LIO의 후속 연구인 FAST-LIVO 31, R3LIVE 65 등이 대표적인 예다. 카메라는 텍스처 정보가 풍부한 환경에서 강점을 보이고, LiDAR는 기하학적 구조 정보가 풍부한 환경에서 강점을 보이므로, 이 둘을 융합하면 상호 보완을 통해 어떤 환경에서도 안정적인 성능을 기대할 수 있다.67 더 나아가, 실내 환경에서의 측위를 위해 Wi-Fi 핑거프린팅 68, 절대 방위를 제공하는 자기장 센서, 빛의 편광 정보를 이용하는 편광 카메라 등 더욱 이질적이고 다양한 센서들을 융합하려는 시도도 계속해서 이루어지고 있다.70

결론적으로, 미래의 SLAM 시스템은 단 하나의 완벽한 알고리즘이 모든 문제를 해결하는 형태가 아닐 것이다. 대신, 다양한 센서와 알고리즘(기하학적 방법, 필터링, 최적화, 딥러닝)이 유기적으로 결합된 '하이브리드' 시스템의 형태를 띨 것이다. 그리고 가장 진보된 시스템은 현재 로봇이 처한 환경과 작업의 종류를 스스로 인지하고, 그 상황에 가장 적합한 센서 조합과 알고리즘을 동적으로 선택하고 융합하는 '적응형(adaptive)' 시스템으로 발전할 것이다.18 이러한 미래 기술의 근간에는 FAST-LIO와 LIO-SAM이 제시했던 필터링과 최적화라는 두 가지 강력한 패러다임이 계속해서 중요한 역할을 수행할 것이다.

#### **참고 자료**

1. VE-LIOM: A Versatile and Efficient LiDAR-Inertial Odometry and Mapping System - MDPI, accessed July 30, 2025, https://www.mdpi.com/2072-4292/16/15/2772
2. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - GitHub, accessed July 30, 2025, https://raw.githubusercontent.com/hku-mars/FAST_LIO/main/doc/Fast_LIO_2.pdf
3. kousheekc/FAST_LIO_GPS: FAST LIO odometry with loop closure and GPS factor based pose graph optimization - GitHub, accessed July 30, 2025, https://github.com/kousheekc/FAST_LIO_GPS
4. hku-mars/FAST_LIO: A computationally efficient and robust LiDAR-inertial odometry (LIO) package - GitHub, accessed July 30, 2025, https://github.com/hku-mars/FAST_LIO
5. FAST-LIO: A Fast, Robust LiDAR-inertial Odometry Package by Tightly-Coupled Iterated Kalman Filter | Request PDF - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/349909094_FAST-LIO_A_Fast_Robust_LiDAR-inertial_Odometry_Package_by_Tightly-Coupled_Iterated_Kalman_Filter
6. [2107.06829] FAST-LIO2: Fast Direct LiDAR-inertial Odometry, accessed July 30, 2025, https://ar5iv.labs.arxiv.org/html/2107.06829
7. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2107.06829
8. [2107.06829] FAST-LIO2: Fast Direct LiDAR-inertial Odometry - arXiv, accessed July 30, 2025, https://arxiv.org/abs/2107.06829
9. [PDF] ikd-Tree: An Incremental K-D Tree for Robotic Applications | Semantic Scholar, accessed July 30, 2025, https://www.semanticscholar.org/paper/ikd-Tree%3A-An-Incremental-K-D-Tree-for-Robotic-Cai-Xu/0cd2ebed90cfb17ae0305eaa0a6d736350cbf39c
10. (PDF) ikd-Tree: An Incremental K-D Tree for Robotic Applications - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/349520013_ikd-Tree_An_Incremental_K-D_Tree_for_Robotic_Applications
11. hku-mars/ikd-Tree: This repository provides implementation of an incremental k-d tree for robotic applications. - GitHub, accessed July 30, 2025, https://github.com/hku-mars/ikd-Tree
12. A Fast, Lightweight, and Dynamic Octree for Proximity Search - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2309.08315
13. A Fast, Lightweight, and Dynamic Octree for Proximity Search - arXiv, accessed July 30, 2025, https://arxiv.org/html/2309.08315v2
14. (PDF) Faster-LIO: Lightweight Tightly Coupled Lidar-Inertial Odometry Using Parallel Sparse Incremental Voxels - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/359662313_Faster-LIO_Lightweight_Tightly_Coupled_Lidar-Inertial_Odometry_Using_Parallel_Sparse_Incremental_Voxels
15. [2302.04031] Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - ar5iv, accessed July 30, 2025, https://ar5iv.labs.arxiv.org/html/2302.04031
16. A Full-order Solution to the Attitude Reset Problem for Kalman Filtering of Attitudes - HiPeRLab, accessed July 30, 2025, https://hiperlab.berkeley.edu/wp-content/uploads/2021/09/2020_FullOrderSolution.pdf
17. [2309.06008] A Note on the Extended Kalman Filter on a Manifold - arXiv, accessed July 30, 2025, https://arxiv.org/abs/2309.06008
18. Adaptive-LIO: Enhancing Robustness and Precision through Environmental Adaptation in LiDAR Inertial Odometry - arXiv, accessed July 30, 2025, https://arxiv.org/html/2503.05077v1
19. A Fast Dynamic Point Detection Method for LiDAR-Inertial Odometry in Driving Scenarios, accessed July 30, 2025, https://arxiv.org/html/2407.03590v1
20. Faster-LIO: Lightweight Tightly Coupled Lidar-inertial Odometry using Parallel Sparse Incremental Voxels - GitHub, accessed July 30, 2025, https://github.com/gaoxiang12/faster-lio
21. [2209.10047] Uncertainty-Aware Tightly-Coupled GPS Fused LIO-SLAM - arXiv, accessed July 30, 2025, https://arxiv.org/abs/2209.10047
22. GNSS-RTK Adaptively Integrated with LiDAR/IMU Odometry for Continuously Global Positioning in Urban Canyons - MDPI, accessed July 30, 2025, https://www.mdpi.com/2076-3417/12/10/5193
23. Visualization of trajectory comparison between A-LOAM, FAST-LIO, and... - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/figure/sualization-of-trajectory-comparison-between-A-LOAM-FAST-LIO-and-our-method-on-KITTI_fig5_378109927
24. 3D LiDAR SLAM Integration with GPS/INS for UAVs in Urban GPS-Degraded Environments - NASA, accessed July 30, 2025, https://www.nasa.gov/wp-content/uploads/2024/04/2017-hening-scitech-2017-0448-508.pdf?emrc=14af5c
25. yxw027/FAST_LIO_GPS - GitHub, accessed July 30, 2025, https://github.com/yxw027/FAST_LIO_GPS
26. jxx315/FAST_LIO_GPS - GitHub, accessed July 30, 2025, https://github.com/jxx315/FAST_LIO_GPS
27. Modular Odometries Fusion with Online Targetless Extrinsic Calibration in a Loosely-Coupled SLAM Framework, accessed July 30, 2025, https://www.iros2024-cartin.com/papers/Modular_Odometries_Fusion_with_Online_Targetless_Extrinsic_Calibration_in_a_Loosely-Coupled_SLAM_Framework.pdf
28. engcang/FAST-LIO-SAM-QN: A SLAM implementation combining FAST-LIO2 with pose graph optimization and loop closing based on Quatro and Nano-GICP - GitHub, accessed July 30, 2025, https://github.com/engcang/FAST-LIO-SAM-QN
29. gisbi-kim/FAST_LIO_SLAM: LiDAR SLAM = FAST-LIO + Scan Context - GitHub, accessed July 30, 2025, https://github.com/gisbi-kim/FAST_LIO_SLAM
30. (PDF) FAST-LIO, Then Bayesian ICP, then GTSFM - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/364126762_FAST-LIO_Then_Bayesian_ICP_then_GTSFM
31. [PDF] FAST-LIO2: Fast Direct LiDAR-Inertial Odometry - Semantic Scholar, accessed July 30, 2025, https://www.semanticscholar.org/paper/FAST-LIO2%3A-Fast-Direct-LiDAR-Inertial-Odometry-Xu-Cai/90a289a9149d714053b0ce5d95b9eec427855d5b
32. (PDF) Notes on Kalman Filter (KF, EKF, ESKF, IEKF, IESKF) - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/381308333_Notes_on_Kalman_Filter_KF_EKF_ESKF_IEKF_IESKF
33. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and ..., accessed July 30, 2025, https://arxiv.org/abs/2007.00258
34. Factor Graph Accelerator for LiDAR-Inertial Odometry (Invited Paper) - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2209.02207
35. Performance Analysis of LIO-SAM - Ronia Peterson, accessed July 30, 2025, http://www.roniapeterson.com/LIO-SAM.pdf
36. Question about SimpleGPsOdometry / Issue #50 / JokerJohn/LIO_SAM_6AXIS - GitHub, accessed July 30, 2025, https://github.com/JokerJohn/LIO_SAM_6AXIS/issues/50
37. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in Highly Dynamic Environments - PMC, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10255994/
38. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 30, 2025, https://github.com/TixiaoShan/LIO-SAM
39. NV-LIO: LiDAR-Inertial Odometry using Normal Vectors Towards Robust SLAM in Multifloor Environments - arXiv, accessed July 30, 2025, https://arxiv.org/html/2405.12563v2
40. LIO-EKF: High Frequency LiDAR-Inertial Odometry using Extended Kalman Filters, accessed July 30, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/wu2024icra.pdf
41. Doppler-SLAM: Doppler-Aided Radar-Inertial and LiDAR-Inertial Simultaneous Localization and Mapping, accessed July 30, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/wang2025ral.pdf
42. gtsam/examples/LocalizationExample.cpp at master - GitHub, accessed July 30, 2025, https://github.com/devbharat/gtsam/blob/master/examples/LocalizationExample.cpp
43. gtsam/examples/ImuFactorsExample.cpp at master - GitHub, accessed July 30, 2025, https://github.com/haidai/gtsam/blob/master/examples/ImuFactorsExample.cpp
44. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - PMC, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11644182/
45. Error comparison of FAST‐LIO2, Point‐LIO‐input, and Point‐LIO, for rotation and position estimations. - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/figure/Error-comparison-of-FAST-LIO2-Point-LIO-input-and-Point-LIO-for-rotation-and-position_fig13_369891453
46. Comparing the absolute error of our method, LIO-SAM and FAST-LIO on the KITTI dataset., accessed July 30, 2025, https://www.researchgate.net/figure/Comparing-the-absolute-error-of-our-method-LIO-SAM-and-FAST-LIO-on-the-KITTI-dataset_tbl2_372016314
47. FAST-LiDAR-SLAM: A Robust and Real-Time Factor Graph for Urban Scenarios With Unstable GPS Signals - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/384097913_FAST-LiDAR-SLAM_A_Robust_and_Real-Time_Factor_Graph_for_Urban_Scenarios_With_Unstable_GPS_Signals
48. Position and Attitude Determination in Urban Canyon with Tightly Coupled Sensor Fusion and a Prediction-Based GNSS Cycle Slip Detection Using Low-Cost Instruments - PMC, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9965468/
49. Hong Kong UrbanNav: An Open-Source Multisensory Dataset for Benchmarking Urban Navigation Algorithms, accessed July 30, 2025, https://navi.ion.org/content/70/4/navi.602
50. how to use gps to evaluation / Issue #22 / TixiaoShan/LIO-SAM - GitHub, accessed July 30, 2025, https://github.com/TixiaoShan/LIO-SAM/issues/22
51. Toward Robust GNSS Real-Time Orbit Determination for Microsatellites Using Factor Graph Optimization - MDPI, accessed July 30, 2025, https://www.mdpi.com/2072-4292/17/7/1125
52. Robust Multi-GNSS Navigation Using Factor Graph Optimization | Technical Program - ION GNSS+ 2025, accessed July 30, 2025, https://www.ion.org/gnss/abstracts.cfm?paperID=15986
53. Lidar-Imu Calibration - Autoware Universe Documentation, accessed July 30, 2025, https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/integrating-autoware/creating-vehicle-and-sensor-model/calibrating-sensors/lidar-imu-calibration/
54. unmannedlab/imu_lidar_calibration: Target-free Extrinsic Calibration of a 3D Lidar and an IMU - GitHub, accessed July 30, 2025, https://github.com/unmannedlab/imu_lidar_calibration
55. hku-mars/LiDAR_IMU_Init: [IROS2022] Robust Real-time LiDAR-inertial Initialization Method. - GitHub, accessed July 30, 2025, https://github.com/hku-mars/LiDAR_IMU_Init
56. Open-Source LiDAR Time Synchronization System by Mimicking GNSS-clock - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2107.02625
57. isprs-archives.copernicus.org, accessed July 30, 2025, https://isprs-archives.copernicus.org/articles/XLVIII-1-W2-2023/1199/2023/isprs-archives-XLVIII-1-W2-2023-1199-2023.pdf
58. AKF-LIO: LiDAR-Inertial Odometry with Gaussian Map by Adaptive Kalman Filter | Request PDF - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/389714684_AKF-LIO_LiDAR-Inertial_Odometry_with_Gaussian_Map_by_Adaptive_Kalman_Filter
59. Accurate Data: Importance of High-Performance Systems - Inertial Labs, accessed July 30, 2025, https://inertiallabs.com/deep-dive-into-resepi-accurate-data/
60. Assessment of Noise of MEMS IMU Sensors of Different Grades for GNSS/IMU Navigation - MDPI, accessed July 30, 2025, https://www.mdpi.com/1424-8220/24/6/1953
61. AKF-LIO: LiDAR-Inertial Odometry with Gaussian Map by Adaptive Kalman Filter - arXiv, accessed July 30, 2025, https://arxiv.org/html/2503.06891v1
62. The Future of Real-Time SLAM and Deep Learning vs SLAM, accessed July 30, 2025, https://www.computervisionblog.com/2016/01/why-slam-matters-future-of-real-time.html
63. [2206.09463] RF-LIO: Removal-First Tightly-coupled Lidar Inertial Odometry in High Dynamic Environments - arXiv, accessed July 30, 2025, https://arxiv.org/abs/2206.09463
64. Underwater SLAM Meets Deep Learning: Challenges, Multi-Sensor Integration, and Future Directions - PMC - PubMed Central, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12157327/
65. FAST-LIVO2 on Resource-Constrained Platforms: LiDAR-Inertial-Visual Odometry with Efficient Memory and Computation | Request PDF - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/388354428_FAST-LIVO2_on_Resource-Constrained_Platforms_LiDAR-Inertial-Visual_Odometry_with_Efficient_Memory_and_Computation
66. [2501.13876] FAST-LIVO2 on Resource-Constrained Platforms: LiDAR-Inertial-Visual Odometry with Efficient Memory and Computation - ar5iv, accessed July 30, 2025, https://ar5iv.labs.arxiv.org/html/2501.13876
67. A Review of Research on SLAM Technology Based on the Fusion of LiDAR and Vision, accessed July 30, 2025, https://www.mdpi.com/1424-8220/25/5/1447
68. Fusion of the SLAM with Wi-Fi-Based Positioning Methods for Mobile Robot-Based Learning Data Collection, Localization, and Tracking in Indoor Spaces - PMC, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7570627/
69. Efficient WiFi LiDAR SLAM for Autonomous Robots in Large Environments - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/361415239_Efficient_WiFi_LiDAR_SLAM_for_Autonomous_Robots_in_Large_Environments
70. LPVIMO-SAM: Tightly-coupled LiDAR/Polarization Vision/Inertial/Magnetometer/Optical Flow Odometry via Smoothing and Mapping - arXiv, accessed July 30, 2025, https://arxiv.org/html/2504.20380v1


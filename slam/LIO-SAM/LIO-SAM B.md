# LIO-SAM

## 1. 강결합 LiDAR-관성 SLAM 서론

### 0.1  SLAM 문제와 센서 모달리티

동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM)은 자율 로봇, 자율 주행차 등 지능형 시스템의 핵심 기술로, 미지의 환경에서 자신의 위치를 실시간으로 추정하면서 동시에 주변 환경의 지도를 구축하는 문제를 다룬다.1 이 복잡한 문제를 해결하기 위해 다양한 센서가 활용되며, 그중 가장 대표적인 것이 LiDAR(Light Detection and Ranging)와 카메라다.

**LiDAR SLAM**은 레이저 펄스를 발사하고 빛이 물체에 반사되어 돌아오는 시간(Time of Flight, ToF)을 측정하여 주변 환경과의 거리를 정밀하게 계산한다. 이를 통해 3차원 포인트 클라우드(point cloud) 형태의 정확한 기하학적 정보를 직접적으로 획득할 수 있다.3 LiDAR는 조명 변화에 강인하고, 희미한 텍스처 환경에서도 안정적인 성능을 보장하므로 자율 주행 및 실외 측량과 같이 높은 정밀도가 요구되는 분야에서 선호된다.7 하지만 센서 가격이 비싸고, 터널과 같이 특징이 없는 반복적인 구조의 환경이나 유리 같은 반사면에 취약하다는 단점이 있다.1

**Visual SLAM (vSLAM)**은 카메라를 사용하여 환경의 시각적 특징점(visual features)을 추적함으로써 움직임을 추정하고 지도를 생성한다.8 카메라는 저렴하고 가벼우며, 풍부한 색상과 텍스처 정보를 제공하여 의미론적(semantic) 정보 추출에 유리하다.11 그러나 조명 변화, 모션 블러, 텍스처가 부족한 환경에서는 성능이 급격히 저하되는 등 환경에 매우 민감하다는 한계를 지닌다.1

이처럼 각 센서는 명확한 장단점을 가지므로, 하나의 센서만으로는 모든 환경에서 강인한 성능을 보장하기 어렵다. 따라서 현대 SLAM 연구의 주된 흐름은 각 센서의 단점을 상호 보완하고 장점을 극대화하는 다중 센서 융합(multi-sensor fusion) 기술로 나아가고 있다.1

### 0.2  진화의 과정: 약결합에서 강결합 융합으로

LiDAR와 관성 측정 장치(Inertial Measurement Unit, IMU)를 융합하는 방식은 크게 약결합(loosely-coupled)과 강결합(tightly-coupled)으로 나뉜다. 이 둘의 차이는 SLAM 시스템의 정확성과 강인성에 결정적인 영향을 미친다.

**약결합 시스템 (예: LOAM, LeGO-LOAM)**은 각 센서 데이터를 독립적이고 순차적으로 처리한다. 예를 들어, IMU 데이터는 주로 LiDAR 스캔 과정에서 발생하는 움직임 왜곡(motion distortion)을 보정하는 전처리 단계에서만 사용된다.2 즉, IMU의 상태(가속도, 각속도 등)가 SLAM의 핵심 최적화 문제에 직접 포함되지 않는다. 이 방식은 구현이 비교적 간단하지만, 한 센서 시스템에서 발생한 오차가 다른 시스템으로 그대로 전파되어 전체 성능을 저하시킬 수 있다는 근본적인 한계를 가진다.

**강결합 시스템 (예: LIO-SAM, FAST-LIO)**은 LiDAR와 IMU의 원시 측정값(또는 사전 적분된 측정값)을 단일 최적화 프레임워크 내에서 통합적으로 처리한다.2 시스템의 상태 변수(위치, 속도, 자세, IMU 바이어스 등)를 동시에 추정함으로써 센서 간 상호 보완을 극대화한다. 예를 들어, LiDAR 측정값은 IMU의 누적 오차(drift)와 바이어스(bias)를 보정하는 데 사용되고, IMU 측정값은 LiDAR 포인트 클라우드 정합(registration)을 위한 정확한 초기 추정치를 제공한다. 이 방식은 구조가 더 복잡하지만, 일반적으로 더 높은 정확도와 강인성을 달성할 수 있다.2

### 0.3  LIO-SAM의 등장 배경: 선행 연구의 한계를 넘어서

LIO-SAM(Lidar Inertial Odometry via Smoothing and Mapping)은 2020년 Tixiao Shan 등에 의해 제안되었으며, LiDAR SLAM 분야에 큰 영향을 미친 LOAM(Lidar Odometry and Mapping) 프레임워크의 한계를 극복하기 위해 설계되었다.2

**LOAM의 한계점:**

- **장거리 주행 시 누적 오차:** LOAM은 기본적으로 스캔 정합(scan-matching)에 기반하기 때문에, 대규모 환경에서 장시간 주행 시 오차가 누적되어 상당한 드리프트가 발생한다.2
- **전역 최적화의 부재:** 전역 복셀 맵(global voxel map)에 데이터를 저장하는 구조로 인해, 주행 경로가 겹치는 루프 폐쇄(loop closure)를 감지하고 이를 통해 누적 오차를 보정하기가 어렵다. 또한 GPS와 같은 절대 위치 측정값을 통합하여 전역적인 오차를 수정하는 기능이 부족하다.2
- **약결합 융합:** LiDAR와 IMU를 약결합 방식으로 융합하여 두 센서 간의 시너지를 완전히 활용하지 못한다.2

이러한 한계들을 해결하기 위한 LIO-SAM의 핵심적인 기여는 다음과 같다.

- **강결합 LiDAR-관성 오도메트리 프레임워크:** LIO-SAM은 팩터 그래프(factor graph) 기반의 최적화 기법을 도입하여 LiDAR와 IMU 데이터를 강결합 방식으로 융합한다.2 이는 LIO-SAM의 가장 근본적인 아키텍처 변화로, 강인한 다중 센서 융합과 전역 최적화를 가능하게 하는 토대가 된다.
- **효율적인 지역적 슬라이딩 윈도우 기반 스캔 정합:** 기존의 스캔 대 전역 맵(scan-to-global-map) 정합 방식 대신, 새로운 키프레임(keyframe)을 고정된 크기의 이전 키프레임 집합("sub-keyframes")으로 구성된 지역 맵(local map)에 정합한다.2 이 방식은 최적화 문제의 복잡도가 전체 지도의 크기와 무관하게 유지되므로, 실시간 성능을 획기적으로 향상시킨다.
- **다양한 제약 조건의 통합:** 팩터 그래프 구조 덕분에 루프 폐쇄 제약과 GPS 제약 같은 다양한 상대 및 절대 측정값을 시스템에 쉽게 통합할 수 있다. 이를 통해 장기적인 누적 오차를 효과적으로 억제하고 전역적으로 일관된 지도를 생성할 수 있다.2

LIO-SAM의 등장은 VIO(Visual-Inertial Odometry) 분야에서 활발히 연구되던 강결합 센서 융합 및 최적화 기법이 LiDAR SLAM 분야에 성공적으로 적용되고, 고성능 오픈소스 형태로 공개되었다는 점에서 중요한 의미를 가진다. 이는 로보틱스 커뮤니티에 더욱 진보된 융합 방법론을 널리 보급하는 계기가 되었으며, LIO-SAM을 기반으로 한 수많은 후속 연구를 촉발시켰다.17 LIO-SAM은 시각 SLAM과 LiDAR SLAM 커뮤니티 간의 방법론적 간극을 메우는 중요한 다리 역할을 했다.

## 1.  LIO-SAM의 아키텍처 설계

### 1.1  시스템 개요 및 데이터 흐름

LIO-SAM 시스템은 3D LiDAR, 9축 IMU, 그리고 선택적으로 GPS로부터 센서 데이터를 입력받아 로봇의 궤적을 추정하고 3차원 지도를 생성한다.2 전체 시스템은 크게 4개의 핵심 모듈로 구성되며, 각 모듈은 별도의 스레드에서 병렬적으로 처리되어 실시간 성능을 보장한다. 데이터는 포인트 클라우드 전처리, IMU 사전 적분, LiDAR 오도메트리, 그리고 지도 최적화의 단계를 거치며 처리된다.

### 1.2  이중 팩터 그래프 전략: 실시간 성능의 핵심

LIO-SAM의 가장 독창적인 설계 특징 중 하나는 두 개의 독립적인 팩터 그래프를 유지하여 상충되는 두 가지 요구사항, 즉 고주파 실시간 오도메트리 추정과 저주파 전역 지도 일관성 유지를 동시에 만족시킨다는 점이다.21

**고주파 오도메트리 그래프 (`imuPreintegration.cpp`):**

- 이 그래프는 최근의 상태(state)들로 구성된 작은 크기의 슬라이딩 윈도우(sliding window)만을 최적화한다.
- IMU 사전 적분 팩터와 LiDAR 오도메트리 팩터를 강결합하여 융합한다.
- 주요 목표는 IMU 측정 빈도에 맞춰 고주파의 실시간 위치 및 자세 추정치를 생성하고, IMU 바이어스를 정확하게 추정하는 것이다.
- 결정적으로, 이 그래프는 주기적으로 초기화되어 계산량을 한정시키고 실시간 성능을 보장한다.21

**저주파 전역 그래프 (`mapOptimization.cpp`):**

- 이 그래프는 시스템이 동작하는 전체 시간 동안 유지된다.
- LiDAR 오도메트리 팩터, GPS 팩터, 루프 폐쇄 팩터 등 전역적인 일관성을 위한 제약 조건들을 통합한다.
- 주요 목표는 누적된 오차를 보정하고 전역적으로 일관된 궤적과 지도를 생성하는 것이다.
- 선별된 키프레임만을 대상으로 동작하기 때문에 업데이트 주기가 훨씬 길며, 이를 통해 전역 최적화의 계산 부담을 현실적인 수준으로 유지한다.21

이러한 이중 그래프 아키텍처는 '일관성 대 효율성'이라는 SLAM의 고전적인 딜레마에 대한 실용적인 해법을 제시한다. 로봇의 제어 루프가 필요로 하는 즉각적인 실시간 위치 추정 기능과, 장기적인 항법에 필수적인 전역적으로 정확한 지도를 유지하는 작업을 효과적으로 분리한 것이다. 로봇은 한 모듈로부터 빠르고 반응성 좋은 위치 정보를 얻고, 다른 모듈로부터 주기적으로 보정되는 전역적으로 일관된 지도를 얻음으로써 두 마리 토끼를 모두 잡을 수 있다. 이러한 관심사의 분리(separation of concerns)는 높은 정확도와 실시간 성능을 동시에 달성할 수 있게 한 핵심적인 아키텍처 설계 결정이다.

### 1.3  모듈별 상세 분석

#### 1.3.1  모듈 1: 포인트 클라우드 전처리 (`imageProjection.cpp`)

이 모듈은 LiDAR 센서로부터 받은 원시 포인트 클라우드를 후속 처리에 적합한 형태로 가공하는 역할을 한다.

- **레인지 이미지 투영:** 3차원 포인트 클라우드를 2차원 레인지 이미지(range image)로 투영한다. 이는 각 포인트의 이웃을 효율적으로 탐색하기 위한 기법이다.23
- **움직임 왜곡 보정 (Motion De-skewing):** 회전하는 기계식 LiDAR는 한 번의 스캔을 완료하는 데 시간이 걸리며, 이 시간 동안 로봇이 움직이면 포인트 클라우드가 왜곡된다. LIO-SAM은 IMU 사전 적분 모듈로부터 받은 고주파의 IMU 측정값을 이용해 각 포인트가 측정된 시점의 로봇 자세를 보간하고, 이를 통해 왜곡된 포인트 클라우드를 스캔 시작 시점 기준으로 보정한다.15 이는 정확한 특징 추출을 위한 필수적인 전처리 과정이다.
- **특징점 추출:** LOAM과 유사하게, 각 포인트의 주변 곡률(curvature)을 계산하여 특징점을 추출한다. 곡률이 큰 지점은 '엣지(edge)' 특징점으로, 곡률이 작은 지점은 '평면(planar)' 특징점으로 분류한다.17

#### 1.3.2  모듈 2: IMU 사전 적분 (`imuPreintegration.cpp`)

이 모듈은 고주파의 IMU 원시 데이터를 처리하여 유용한 모션 제약으로 변환한다.

- 두 개의 연속된 LiDAR 키프레임 사이에서 수집된 모든 IMU 측정값(가속도, 각속도)을 적분하여 단일 상대 변환 제약(relative motion constraint)을 생성한다. 이를 'IMU 사전 적분 팩터'라고 한다.2
- '사전 적분' 기법을 사용하는 이유는 최적화 과정에서 로봇의 과거 상태가 업데이트될 때마다 매번 IMU 측정값을 다시 적분하는 막대한 계산 비용을 피하기 위함이다.
- 이 모듈에서 계산된 모션 추정치는 모듈 1의 움직임 왜곡 보정에 사용되며, 모듈 3의 LiDAR 오도메트리 최적화를 위한 초기 추정치(initial guess)로도 활용된다.2

#### 1.3.3  모듈 3: LiDAR 오도메트리 및 스캔 대 지역 맵 정합

이 모듈은 지역적이고 실시간적인 위치 추정의 핵심이다.

- LIO-SAM은 새로운 키프레임을 전역 맵이 아닌, **고정된 크기의 슬라이딩 윈도우 내에 있는 이전 키프레임들로 구성된 지역 맵**에 정합한다.2
- 이 '스캔 대 지역 맵(scan-to-local-map)' 정합 전략은 정합 문제의 복잡도가 전체 지도의 크기에 따라 증가하지 않기 때문에 LIO-SAM의 효율성과 실시간 성능을 보장하는 핵심 요소다.2
- 정합 결과로 현재 키프레임과 지역 맵 간의 상대적인 자세 변환이 계산되며, 이는 'LiDAR 오도메트리 팩터'로서 팩터 그래프에 추가된다.

#### 1.3.4  모듈 4: 지도 최적화 (`mapOptimization.cpp`)

이 모듈은 전역 팩터 그래프를 관리하고 최적화하여 전역적인 일관성을 유지한다.

- 새로운 키프레임과 그에 해당하는 LiDAR 오도메트리 팩터를 전역 그래프에 추가한다.
- **루프 폐쇄 감지:** 로봇이 이전에 방문했던 장소를 다시 지나갈 때, 시스템은 이를 감지한다(초기 버전은 반경 탐색과 ICP 기반). 루프가 감지되면 현재 위치와 과거 위치 사이에 새로운 제약 조건(루프 폐쇄 팩터)을 추가하여, 멀리 떨어진 두 노드를 연결한다.15
- **GPS 융합:** GPS 데이터가 사용 가능한 경우, 절대 위치 정보를 제공하는 'GPS 팩터'를 그래프에 추가한다. 이 팩터는 지도를 전역 좌표계에 고정시키고 오차가 무한정 누적되는 것을 방지한다.2
- GTSAM 라이브러리를 사용하여 주기적으로 이 전역 그래프를 최적화하고, 모든 키프레임의 자세를 미세 조정하여 전역적으로 일관된 궤적과 지도를 생성한다.16

## 2.  프레임워크의 심장: 팩터 그래프 최적화

### 2.1  최대 사후 확률(MAP) 기반 문제 정의

LIO-SAM은 SLAM 문제를 확률적인 관점에서 접근한다. 즉, 모든 센서 측정값이 주어졌을 때, 가장 가능성이 높은 로봇의 상태(위치, 자세, 속도, 바이어스 등) 시퀀스를 찾는 문제로 정의한다. 이는 통계학적으로 최대 사후 확률(Maximum a Posteriori, MAP) 추정 문제에 해당한다.2 팩터 그래프는 이러한 MAP 문제를 표현하고 해결하는 데 매우 적합한 그래픽 모델이다. 그래프의 노드(node)는 추정하고자 하는 미지의 로봇 상태를 나타내고, 팩터(factor)는 센서 측정값으로부터 파생된 확률적 제약 조건을 나타낸다.2

### 2.2  네 가지 핵심 제약 팩터 분석

LIO-SAM의 팩터 그래프는 네 가지 주요 팩터를 통해 시스템의 상태를 제약한다.2

**IMU 사전 적분 팩터:**

- 연속된 두 로봇 상태 노드를 연결한다.
- 두 상태 사이의 고주파 IMU 측정값을 적분하여 얻은 상대적인 움직임을 제약 조건으로 표현한다.2
- 여기서 강결합의 진가가 드러나는데, LiDAR 오도메트리 최적화의 결과가 다시 IMU 바이어스를 추정하고 보정하는 데 사용된다.2 이는 IMU가 LiDAR를 돕고, LiDAR가 다시 IMU를 돕는 상호 보완적인 피드백 루프를 형성한다.

**LiDAR 오도메트리 팩터:**

- 로봇 상태 노드를 지도에 연결한다.
- 현재 LiDAR 스캔의 특징점(엣지, 평면)을 지역 맵에 정합함으로써 얻어지는 기하학적 제약 조건을 나타낸다.2
- 이 팩터의 오차는 포인트와 라인 간의 거리, 그리고 포인트와 평면 간의 거리 합으로 계산된다.17 최적화 과정은 이 오차를 최소화하여 스캔을 맵에 정밀하게 정렬한다.

**GPS 팩터:**

- 선택 사항이지만, 누적 오차를 억제하는 데 매우 강력한 역할을 하는 절대 위치 제약 조건이다.
- 단일 로봇 상태 노드를 월드 좌표계의 고정된 위치에 직접 연결한다.2
- 이 단일(unary) 팩터는 전역적인 앵커(anchor) 역할을 하여, 궤적이 무한정 표류하는 것을 막고 생성된 지도를 UTM과 같은 실제 전역 좌표계에 정렬시킨다. `poseCovThreshold`와 같은 파라미터를 통해 GPS 측정값에 대한 신뢰도를 조절할 수 있다.16

**루프 폐쇄 팩터:**

- 물리적으로는 동일한 위치에 해당하지만, 시간적으로는 멀리 떨어진 두 로봇 상태 노드를 연결한다.
- 로봇이 이전에 방문했던 장소로 돌아와 루프가 감지되면, 현재 위치와 과거 위치 간의 상대적인 변환이 계산된다(주로 ICP 알고리즘 사용).19
- 이 변환 관계가 팩터로 추가되면, 최적화 과정에서 궤적의 양 끝을 서로 끌어당겨 루프 전체에 걸쳐 누적된 오차를 분산시키고 보정한다.2 부정확한 루프 폐쇄의 영향을 줄이기 위해 코시(Cauchy)와 같은 강인한 커널 함수(robust kernel)가 사용될 수 있다.19

### 2.3  GTSAM의 역할

LIO-SAM은 팩터 그래프 최적화 문제를 해결하기 위해 조지아 공과대학교에서 개발한 GTSAM(Georgia Tech Smoothing and Mapping) 라이브러리에 의존한다.16 GTSAM은 효율적인 증분식 평활화 및 매핑(incremental smoothing and mapping) 알고리즘인 iSAM2를 제공한다. iSAM2는 새로운 팩터가 추가될 때마다 전체 문제를 처음부터 다시 푸는 대신, 변화가 생긴 부분만 효율적으로 업데이트하여 해를 구한다. 이는 특히 지속적으로 노드와 팩터가 추가되는 전역 그래프의 최적화에서 실시간 성능을 유지하는 데 결정적인 역할을 한다.17

팩터 그래프는 단순히 계산 도구가 아니라, LIO-SAM을 매우 유연하고 확장 가능하게 만드는 핵심적인 추상화 계층이다. LIO-SAM 프레임워크의 진정한 힘은 새로운 센서나 제약 조건을 기존 시스템 아키텍처를 변경하지 않고도 새로운 '팩터'로 쉽게 추가할 수 있다는 점에 있다. 예를 들어, SC-LIO-SAM은 기존의 루프 폐쇄 방식을 더 강인한 Scan Context로 대체했는데, 이는 시스템 전체를 재작성하는 것이 아니라 더 신뢰도 높은 루프 폐쇄 팩터를 생성하는 모듈로 교체한 것에 불과하다.19 편광 카메라와 광학 흐름을 추가하는 LPVIMO-SAM은 Z축 드리프트를 억제하기 위한 '높이 팩터'와 '속도 팩터'라는 완전히 새로운 유형의 제약을 기존 그래프에 추가했다.1 이처럼 팩터 그래프는 센서 융합을 위한 보편적인 '플러그 앤 플레이' 인터페이스 역할을 한다. 이러한 모듈성은 LIO-SAM 변종들이 폭발적으로 증가하게 된 근본적인 이유이며, 원본 알고리즘의 특정 구성 요소보다 훨씬 더 중요한 기여라고 할 수 있다.

## 3.  이론에서 현실로: 구현 및 시스템 요구사항

LIO-SAM을 성공적으로 구현하고 활용하기 위해서는 알고리즘 자체에 대한 이해뿐만 아니라, 하드웨어 및 소프트웨어 요구사항, 그리고 핵심 설정 파라미터에 대한 정확한 인지가 필수적이다.

### 3.1  하드웨어 요구사항: 타협할 수 없는 조건들

LIO-SAM의 성능은 입력되는 센서 데이터의 품질에 절대적으로 의존한다.

**LiDAR:** 회전하는 기계식 LiDAR가 필요하며, 각 포인트마다 다음 두 가지 정보를 반드시 제공해야 한다.

- **포인트 타임스탬프 (Point Timestamp):** 한 스캔 내에서 각 포인트의 상대적인 시간 정보 (예: 10Hz LiDAR의 경우 0 ~ 0.1초). 이 정보는 움직임 왜곡 보정에 필수적이다.21
- **링 번호 (Ring Number):** 각 포인트가 LiDAR의 몇 번째 채널에서 측정되었는지를 나타내는 ID. 이는 포인트 클라우드를 레인지 이미지로 효율적으로 변환하는 데 사용된다.21

**IMU:** 가장 중요하고 종종 오해를 불러일으키는 요구사항이다.

- LIO-SAM은 **9축 IMU**를 사용하도록 설계되었다. 9축 IMU는 가속도와 각속도뿐만 아니라, 지자기 센서를 융합하여 절대적인 롤(roll), 피치(pitch), 요(yaw) 자세 추정치를 제공한다.21

- 롤과 피치는 시스템 초기화 시 중력 방향을 정렬하는 데 사용되며, 요는 GPS와 함께 사용할 때 초기 진행 방향을 설정하는 데 활용된다.

- 많은 센서에 내장된 일반적인 **6축 IMU**(가속도, 각속도만 제공)를 사용하면 표준 LIO-SAM 구현에서는 실패한다. 이는 저자에 의해 직접 확인된 사실이다.28

  `liorf`와 같은 일부 파생 프로젝트에서 6축 IMU 지원이 추가되었지만, 원본 시스템은 9축을 요구한다.21

- **높은 주파수:** IMU 데이터 전송률은 높을수록 좋으며, 최소 200Hz, 원본 논문에서는 500Hz 사용을 권장한다.21 데이터 주파수가 높을수록 사전 적분과 왜곡 보정의 정확도가 향상되기 때문이다.

### 3.2  소프트웨어 의존성

LIO-SAM은 다음의 소프트웨어 스택 위에서 동작한다.

- **ROS (Robot Operating System):** 원본 구현은 ROS1(Kinetic, Melodic, Noetic에서 테스트됨)을 기반으로 하며, ROS2를 지원하는 브랜치도 존재한다.16
- **GTSAM (v4.0 이상):** 핵심 최적화 라이브러리.16
- **PCL (Point Cloud Library):** 포인트 클라우드 데이터 구조 및 처리에 사용된다.16

### 3.3  설정 심층 분석: `params.yaml`

이 YAML 파일은 사용자가 조정할 수 있는 모든 파라미터를 담고 있으며, 특정 센서 구성에 맞게 시스템을 튜닝하는 데 매우 중요하다.

- **센서 토픽:** `lidarTopic`, `imuTopic`, `gpsTopic` 등 ROS 토픽 이름을 정확하게 설정해야 한다.
- **외부 파라미터 보정 (Extrinsic Calibration):** `extrinsicTrans`와 `extrinsicRot` (또는 `extrinsicRPY`)는 IMU와 LiDAR 좌표계 간의 상대적인 위치 및 회전 변환을 정의한다. **강결합 융합이 제대로 동작하기 위해서는 이 값이 매우 정확해야 한다.** 저자들은 사용한 IMU의 가속도계와 자이로스코프/자세계의 좌표계가 달라 두 개의 다른 외부 파라미터가 필요했다고 언급했다.21
- **키프레임 관리:** `surroundingKeyframeSearchRadius`, `loopClosureFrequency`와 같은 파라미터는 그래프의 밀도와 정확도-계산 부하 간의 트레이드오프를 제어한다.
- **GPS 튜닝:** `gpsCovThreshold`, `poseCovThreshold`는 시스템이 GPS 측정값을 얼마나 신뢰하고 융합할지를 제어한다.22

------

**표 1: LIO-SAM 하드웨어 및 소프트웨어 요구사항**

| 구성 요소         | 요구 사양 / 명세                   | 필요한 이유 / 참고사항                                       |
| ----------------- | ---------------------------------- | ------------------------------------------------------------ |
| **LiDAR**         | 회전하는 기계식 LiDAR              | 원본 알고리즘은 기계식 LiDAR의 스캔 패턴을 가정함.21         |
|                   | 포인트별 타임스탬프 제공           | 움직임 왜곡 보정(motion de-skewing)에 필수적임.21            |
|                   | 포인트별 링 번호 제공              | 포인트 클라우드를 레인지 이미지로 효율적으로 변환하는 데 사용됨.21 |
| **IMU**           | 9축 (가속도, 각속도, 지자기)       | 초기 자세(롤, 피치)를 중력 방향에 정렬하고, GPS와 함께 초기 방향(요)을 설정하는 데 필요함.21 |
|                   | 데이터 전송률: 200Hz 이상 권장     | 높은 주파수는 IMU 사전 적분 및 왜곡 보정의 정확도를 향상시킴.21 |
| **GPS (선택)**    | NMEA 또는 `nav_msgs/Odometry` 형식 | 절대 위치 정보를 제공하여 전역 누적 오차를 보정하고 지도를 고정시킴.2 |
| **컴퓨팅 플랫폼** | Intel i5/i7 CPU, 16GB+ RAM         | 실시간 처리를 위한 적절한 계산 능력이 필요함.17              |
| **소프트웨어**    | ROS (ROS1 Melodic/Noetic 권장)     | 시스템의 통신 및 실행 환경.16                                |
|                   | GTSAM (v4.0 이상)                  | 팩터 그래프 최적화를 위한 핵심 라이브러리.16                 |
|                   | PCL (Point Cloud Library)          | 포인트 클라우드 데이터 처리.16                               |

------

이러한 엄격한 하드웨어 요구사항, 특히 IMU에 대한 조건은 LIO-SAM이 범용 알고리즘이라기보다는 특정 종류의 하드웨어에 최적화된 전문 시스템임을 보여준다. LIO-SAM의 성능은 센서 입력의 품질과 근본적으로 연결되어 있으며, 이는 알고리즘 자체만큼이나 센서 선택과 보정이 중요함을 의미한다. 강결합 시스템의 특성상, 한 센서(예: 노이즈가 많고 주파수가 낮은 IMU)의 결함은 전체 최적화 과정에 직접적이고 부정적인 영향을 미친다. 따라서 성공적인 LIO-SAM 배포를 위한 엔지니어링 노력의 상당 부분은 코드를 실행하기 전 단계, 즉 센서 선택, 장착, 그리고 정밀한 외부 파라미터 보정 과정에서 이루어진다. 이는 단순한 소프트웨어 문제가 아닌, 시스템 엔지니어링의 과제이다.

## 4.  성능 분석 및 벤치마킹

### 4.1  SLAM의 핵심 평가 지표

SLAM 시스템의 궤적 추정 정확도를 정량적으로 평가하기 위해 주로 사용되는 표준 지표는 다음과 같다.5

- **절대 궤적 오차 (Absolute Trajectory Error, ATE):** 궤적의 전역적인 일관성을 측정한다. 추정된 궤적과 실제 참값(ground truth) 궤적을 정렬한 후, 각 시간 지점에서의 위치 오차에 대한 평균 제곱근 오차(Root Mean Square Error, RMSE)를 계산한다. 이는 시스템의 전체적인 누적 오차를 나타내는 좋은 지표다.29 ATE는 다음 수식으로 계산된다:


$$
\text{ATE} = \frac{1}{N} \sum_{i=1}^{N} \| p_i^\text{groundtruth} - p_i^\text{estimate} \| ^2
$$

- **상대 위치 오차 (Relative Pose Error, RPE):** 궤적의 지역적인 일관성, 즉 오도메트리의 드리프트를 측정한다. 고정된 시간 간격이나 이동 거리에 대한 상대적인 자세 변환 오차를 계산한다. 이는 시스템이 단위 시간 또는 단위 거리당 얼마나 표류하는지를 평가하는 데 유용하다.5

### 4.2  공개 데이터셋에서의 성능

LIO-SAM은 다양한 플랫폼에서 수집된 여러 공개 데이터셋을 통해 광범위하게 검증되었다.15

- **KITTI 데이터셋:** 자율 주행 연구를 위한 표준 벤치마크로, LIO-SAM은 종종 이 데이터셋에서 다른 알고리즘과 비교되는 기준선(baseline)으로 사용된다.31 공식 리더보드에 직접 이름이 올라가 있지는 않지만, 수많은 논문에서 LIO-SAM을 기준으로 자신들의 알고리즘 성능을 평가하며, LIO-SAM이 준수한 성능을 보이지만 최신 특화 알고리즘에는 미치지 못함을 보여준다.
- **UrbanLoco & UrbanNav-HK:** 매우 동적인 도심 환경을 담고 있는 도전적인 데이터셋이다. 이 시나리오에서는 동적 객체를 처리하는 기능이 추가된 확장 버전들이 LIO-SAM의 성능을 크게 향상시키는 것으로 나타났다. 예를 들어, RF-LIO는 ATE를 70% 개선했고 18, ID-LIO는 67-85% 개선했다.17
- **MulRan 데이터셋:** 로보틱스 연구를 위한 대규모 다중 모달 데이터셋이다. SC-LIO-SAM은 이 데이터셋에서, 특히 다리 위와 같이 구조가 명확하지 않고 움직이는 차량이 많은 환경에서 강인한 성능을 보여주었다.19

### 4.3  실시간 성능 및 계산 부하

- LIO-SAM은 적절한 하드웨어에서 실시간보다 최대 10배 빠르게 실행되도록 설계되었다.21
- 시스템의 실행 시간은 팩터 그래프의 노드 수보다는 특징점 지도의 밀도에 더 큰 영향을 받는다.20 이는 스캔 대 지역 맵 정합 방식의 직접적인 결과이다.
- 시스템을 가속화하려는 노력도 있었다. `LIO-SAM-GPU-ScanToMapOpt`와 같은 프로젝트는 가장 계산 비용이 높은 부분인 특징점 정합을 위한 최근접 이웃 탐색(k-NN search)을 CUDA를 이용해 GPU로 오프로딩하여 프레임당 처리 시간을 크게 단축시켰다.37

------

**표 2: 공개 데이터셋에서의 LIO-SAM 성능 (정성적 및 비교 분석)**

| 데이터셋                        | 환경 유형                        | LIO-SAM 성능 요약                                            | 비교 대상 / 성능 초과 알고리즘 | 핵심 시사점                                                  |
| ------------------------------- | -------------------------------- | ------------------------------------------------------------ | ------------------------------ | ------------------------------------------------------------ |
| **KITTI**                       | 자율 주행 (도심, 시골, 고속도로) | 강력한 기준선 역할을 하지만, 긴 시퀀스에서는 누적 오차가 발생할 수 있음.31 | LIO-SAM++ 31, LIO-CSI 34       | 전반적으로 우수하나, 의미론적 정보나 동적 객체 필터링을 추가하면 성능이 향상됨. |
| **UrbanLoco / UrbanNav-HK**     | 매우 동적인 도심 환경            | 많은 수의 동적 객체로 인해 정확도가 저하되는 경향이 있음.17  | RF-LIO 18, ID-LIO 17           | 동적 환경에 대한 강인성은 LIO-SAM의 주요 개선 영역임.        |
| **MulRan**                      | 대규모 복합 환경 (도심, 교외)    | 효과적인 루프 폐쇄 덕분에 강인한 성능을 보이지만, 더 정교한 루프 폐쇄 기법으로 개선 가능.19 | SC-LIO-SAM 19                  | 대규모 환경에서는 루프 폐쇄의 품질이 전역 일관성에 결정적임. |
| **실내 / 복도 (사용자 데이터)** | 특징이 부족하고 반복적인 구조    | 구조적으로 반복되는 직선 구간에서 위치 추정에 실패하거나 큰 오차가 발생할 수 있음.38 | NV-LIO 23 (실내 특화)          | 기하학적으로 퇴화된(degenerate) 환경은 LIO-SAM의 주요 취약점임. |

------

LIO-SAM의 성능은 단일한 수치로 평가되기보다는 하나의 '성능 범위(performance envelope)'로 이해되어야 한다. 이 알고리즘은 높은 품질의 기준선을 설정했지만, 연구 커뮤니티에서 LIO-SAM의 진정한 가치는 동적인 환경이나 대규모 누적 오차와 같은 특정하고 도전적인 조건에서 체계적으로 개선되어야 할 벤치마크로서의 역할에 있다. 수많은 후속 연구들이 LIO-SAM을 기반으로 특정 실패 사례를 해결함으로써 분야 전체의 발전을 이끌고 있다는 사실이 이를 증명한다. 따라서 LIO-SAM의 역할은 '최첨단(state-of-the-art)'에서 '표준 잣대(standard yardstick)'로 진화했다고 볼 수 있다.

## 5.  경쟁 구도: LIO-SAM과 동시대의 알고리즘들

### 5.1  LIO-SAM vs. LeGO-LOAM: 강결합의 혁명

- **핵심 차이:** 융합 전략. LeGO-LOAM은 약결합, LIO-SAM은 강결합 방식이다.2
- **LeGO-LOAM:** IMU를 왜곡 보정과 스캔 대 스캔(scan-to-scan) 정합을 위한 초기 추정치 제공에만 사용한다. 지상 차량에 최적화하기 위해 지면 분할(ground segmentation) 기능을 포함하고 있으며, 별도의 오도메트리 스레드와 매핑 스레드를 가진다.23
- **LIO-SAM:** IMU 사전 적분을 팩터 그래프 최적화에 직접 통합하여 LiDAR와 IMU가 상호 보완적으로 서로를 보정하게 한다. 스캔 대 스캔 오도메트리 대신 더 강인한 스캔 대 지역 맵(scan-to-local-map) 방식을 채택했다.24
- **시사점:** LIO-SAM의 아키텍처는 격렬한 움직임에 본질적으로 더 강인하며 센서 바이어스를 더 잘 추정할 수 있어 전반적으로 더 높은 정확도를 제공하지만, 그 대가로 더 엄격한 센서 요구사항을 가진다.

### 5.2  LIO-SAM vs. FAST-LIO 계열: 특징점 기반 vs. 직접 방식, 최적화 vs. 필터링

이는 현대 LiDAR-관성 오도메트리(LIO) 분야의 가장 근본적인 방법론적 비교다.

- **프론트엔드 (정합 방식):**

  - **LIO-SAM:** **특징점 기반(Feature-based)**. LOAM에서 계승한 방식으로, 포인트 클라우드에서 엣지와 평면 특징점을 추출하여 이 희소한 특징점들을 정합한다.23
  - **FAST-LIO2:** **직접 방식(Direct method)**. 원시 포인트 클라우드를 다운샘플링하여 밀집된(또는 반-밀집된) 포인트들을 지도에 직접 정합한다.23 이는 특징점 추출 단계를 생략하여 계산 병목이나 특징점이 잘 정의되지 않는 환경에서의 실패 가능성을 줄여준다.

- **백엔드 (상태 추정 방식):**

  - **LIO-SAM:** **최적화 기반(Optimization-based)**. 팩터 그래프와 평활화(smoothing) 기법(GTSAM)을 사용하여 과거의 여러 자세를 포함하는 윈도우를 최적화한다. 과거 상태를 다시 선형화(re-linearization)함으로써 더 높은 정확도를 얻을 수 있다.40
  - **FAST-LIO 계열:** **필터 기반(Filter-based)**. 반복 확장 칼만 필터(Iterated Extended Kalman Filter, iEKF)를 사용하여 현재 상태를 재귀적으로 업데이트한다. 독창적인 칼만 이득(Kalman gain) 계산 방식 덕분에 극도의 계산 효율성으로 유명하다.23

- **지도 관리:**

  - **LIO-SAM:** 키프레임과 연관된 포인트 클라우드로 구성된 슬라이딩 윈도우를 사용한다.

  - **FAST-LIO2/Faster-LIO:** 지도를 위해 `ikd-Tree` 23나 

    `iVox` 23와 같은 매우 효율적인 증분식 데이터 구조를 사용한다. 이는 포인트의 삽입, 삭제, 검색을 매우 빠르게 처리하여 고속의 스캔 대 맵 정합을 가능하게 한다.

- **시사점:** FAST-LIO2는 종종 더 빠르고 계산적으로 효율적이다. 반면, LIO-SAM의 팩터 그래프 백엔드는 루프 폐쇄나 GPS와 같은 전역 제약 조건을 통합하기에 더 유연한 프레임워크를 제공하여, 대규모 지도에서 더 나은 전역 일관성을 달성할 잠재력을 가진다. 이러한 장단점은 `FAST-LIO-SAM`과 같은 하이브리드 시스템의 등장을 이끌었다. 이 시스템은 FAST-LIO2의 빠른 프론트엔드와 LIO-SAM의 강인한 백엔드를 결합하여 두 세계의 장점을 모두 취하려 한다.26

### 5.3  LIO-SAM vs. VINS-Mono: LiDAR vs. Vision

- **센서 모달리티:** 가장 명백한 차이점이다. LIO-SAM은 기하학적 데이터를 위해 LiDAR를, VINS-Mono는 시각적 특징을 위해 단안 카메라를 사용한다.15
- **장단점:**
  - **LIO-SAM:** 뛰어난 기하학적 정확도, 조명 불변성. 하지만 기하학적으로 퇴화된 환경(긴 터널 등)에서 취약하다.
  - **VINS-Mono:** 텍스처가 풍부한 환경에서 강인하고, 가볍고 저렴하다. 하지만 저조도나 텍스처가 없는 환경에서는 실패하며, 적절한 초기화나 루프 폐쇄 없이는 스케일(scale)이 모호해질 수 있다.4
- **통합 (LVI-SAM):** `LVI-SAM`의 존재는 두 기술의 상호 보완성을 증명하는 궁극적인 증거다.14 LVI-SAM은 LIO-SAM과 VINS-Mono 시스템을 결합하여, LiDAR-관성 시스템(LIS)이 시각-관성 시스템(VIS)에 대해 정확한 깊이와 스케일을 포함한 강인한 초기화를 제공하도록 한다. 이는 각 개별 구성 요소보다 훨씬 더 강인한 시스템을 만들어낸다.

------

**표 3: LIO-SAM, LeGO-LOAM, FAST-LIO2 비교 분석**

| 속성           | LeGO-LOAM                          | LIO-SAM                             | FAST-LIO2                       |
| -------------- | ---------------------------------- | ----------------------------------- | ------------------------------- |
| **융합 전략**  | 약결합 (Loosely-Coupled)           | 강결합 (Tightly-Coupled)            | 강결합 (Tightly-Coupled)        |
| **프론트엔드** | 특징점 기반 (엣지/평면), 지면 분할 | 특징점 기반 (엣지/평면)             | 직접 방식 (다운샘플링된 포인트) |
| **백엔드**     | 2단계 최적화 (Odometry/Mapping)    | 팩터 그래프 평활화 (iSAM2)          | 반복 확장 칼만 필터 (iEKF)      |
| **지도 구조**  | 전역 복셀 맵                       | 슬라이딩 윈도우 키프레임 맵         | 증분식 k-d 트리 (ikd-Tree)      |
| **루프 폐쇄**  | 반경 탐색 기반                     | 반경 탐색 기반 (확장 가능)          | 기본적으로 없음 (추가 가능)     |
| **핵심 강점**  | 지상 차량에 최적화, 경량화         | 팩터 그래프를 통한 높은 전역 일관성 | 극도의 계산 효율성 및 속도      |
| **주요 약점**  | 약결합으로 인한 정확도 한계        | 특징점 추출 의존성, 계산 부하       | 전역 최적화 부재, 파라미터 튜닝 |

## 6.  한계 및 실패 사례에 대한 비판적 검토

LIO-SAM은 강력한 성능을 자랑하지만, 완벽한 알고리즘은 아니며 특정 조건에서는 한계를 드러낸다. 이러한 실패 사례를 이해하는 것은 알고리즘의 작동 원리를 더 깊이 파악하고 적절한 적용 분야를 선택하는 데 중요하다.

### 6.1  대규모 및 특징 부족 환경에서의 누적 오차

- LIO-SAM은 강인함에도 불구하고, 특히 대규모 환경에서는 작은 오차가 누적되어 드리프트로부터 자유롭지 못하다.2
- **Z축 드리프트:** 수직 방향(Z축)으로의 드리프트는 특히 흔한 문제다. 수직적인 특징이 거의 없는 평평한 지면을 장시간 주행할 때, 시스템은 수직 움직임을 제대로 제약하지 못해 지도가 시간이 지남에 따라 기울어지거나 위아래로 표류하는 현상이 발생한다.1 이 문제는 지면을 명시적으로 추출하여 제약 조건으로 추가하는 연구들을 촉발시켰다.49
- **기하학적 퇴화 (Geometric Degeneracy):** 특징점 기반 프론트엔드는 길고 균일한 복도나 터널과 같이 구조적으로 반복되거나 뚜렷한 기하학적 특징이 없는 환경에서 실패한다.13 이러한 경우, 최적화 문제가 제대로 제약되지 않아(poorly constrained) 큰 자세 오차를 유발하거나 완전히 실패할 수 있다.51 한 사용자는 실내 복도에서 로봇이 직선으로 주행한 후 위치 추정에 실패했다고 보고했다.38

### 6.2  동적 환경에 대한 민감성

- LIO-SAM의 핵심 가정은 대부분 정적인 환경이다. 비록 이전의 많은 알고리즘보다 강인하지만, 많은 수의 동적 객체(차량, 보행자 등)는 스캔 정합 과정에 잘못된 특징점을 유입시킬 수 있다.34
- 이러한 '나쁜' 특징점들은 LiDAR 오도메트리 팩터를 오염시켜 부정확한 자세 추정을 유발하고, 지도에 '유령(ghosting)' 현상을 남긴다.34
- 이러한 한계는 특징점 정합 이전에 의미론적 분할(semantic segmentation)을 통해 동적 객체를 식별하고 필터링하는 확장 연구(예: LIO-CSI, LIO-SAM++, RF-LIO)의 주요 동기가 되었다.18

### 6.3  '9축 IMU' 제약과 초기화 문제

- 앞서 4장에서 자세히 설명했듯이, 9축 IMU에 대한 강한 요구사항은 중요한 제약이다. 이는 Ouster LiDAR의 내장 IMU를 포함하여 많은 센서에서 흔히 볼 수 있는 6축 IMU의 사용을 막는다.21 이는 저자에 의해 LIO-mapping 데이터셋에서 실패한 원인으로 직접 확인되었다.28
- 시스템의 초기화는 IMU로부터 얻는 절대적인 롤/피치 값에 의존하여 중력 방향을 정렬한다. 부정확한 초기화는 시스템이 처음부터 발산하거나 기울어진 지도를 생성하는 원인이 될 수 있다.

### 6.4  루프 폐쇄의 취약성

- LIO-SAM의 기본적인 루프 폐쇄 메커니즘은 단순한 반경 탐색과 ICP에 의존하여 취약할 수 있다.

- 누적된 오차가 너무 커서 로봇이 과거 위치의 탐색 반경 내에 있다는 것을 인지하지 못하면 루프를 감지하지 못할 수 있다.48

- 반대로, 반복적인 구조의 환경에서는 잘못된 루프를 감지(false positive)하여 지도를 부정확하게 수정할 위험도 있다.

- 이는 `SC-LIO-SAM`의 Scan Context 19나 

  `LB-LIOSAM`의 BoW3D 48와 같이, 더 강인한 디스크립터(descriptor) 기반의 루프 폐쇄 방법을 개발하는 동기가 되었다.

LIO-SAM의 실패 모드들은 알고리즘의 핵심 가정들이 위반될 때 발생한다. 이 알고리즘의 우아함은 몇 가지 강력한 사전 가정(좋은 기하학적 특징, 정적인 세계, 고품질 9축 IMU)에 의존하는 데서 비롯된다. 이러한 가정 중 어느 하나라도 현실 세계에서 깨지면 시스템의 성능은 저하된다. 예를 들어, LIO-SAM은 안정적인 기하학적 특징을 가정하지만 24, 동적인 세계에서는 이 특징들이 움직여 잘못된 제약을 만들고 34, 퇴화된 환경에서는 특징 자체가 없어 약한 제약을 만든다.38 또한, 9축 IMU가 좋은 초기 방향과 고주파 모션 데이터를 제공한다고 가정하지만 21, 6축 IMU는 초기화에 필요한 절대 방향을 제공하지 못해 실패를 유발한다.28 따라서 LIO-SAM의 한계를 이해하는 것은 '버그'를 찾는 것이 아니라, 알고리즘의 작동 설계 범위의 경계를 이해하는 것이다. 8장에서 논의될 전체 확장 생태계는 이러한 가정이 유지되지 않는 환경에 대해 '패치'를 적용하려는 노력의 집합체로 볼 수 있다.

## 7.  LIO-SAM 생태계: 파생 및 개선 연구 동향

LIO-SAM의 오픈소스 공개는 전 세계 연구자들이 이를 기반으로 다양한 문제점을 해결하고 기능을 확장하는 활발한 생태계를 구축하는 계기가 되었다. 이 생태계는 크게 강인성 향상, 성능 증대, 그리고 기능 확장이라는 세 가지 방향으로 발전해왔다.

### 7.1  강인성 및 정확도 향상

- **동적 환경 대응:**
  - **RF-LIO:** '제거 우선(Removal-First)' 접근법을 채택하여, 스캔 정합에 앞서 움직이는 객체를 먼저 식별하고 제거한다. 이를 통해 매우 동적인 환경에서 ATE를 최대 70%까지 개선했다.18
  - **ID-LIO:** 인덱싱된 포인트와 지연된 제거 전략을 사용하여 동적 환경에서의 성능을 향상시켰으며, 도심 데이터셋에서 67-85%의 ATE 개선을 보였다.17
- **의미론적 SLAM (Semantic SLAM):**
  - **LIO-SAM++ / LIO-CSI:** 딥러닝 기반의 의미론적 분할을 통합하여 동적 객체를 필터링하고, 정합을 위한 추가적인 의미론적 제약 조건을 제공하며, 장면의 의미론적 변화에 따라 키프레임을 생성하는 등 자세 추정 정확도를 높였다.31
- **고급 루프 폐쇄:**
  - **SC-LIO-SAM:** 기존의 반경 탐색 기반 루프 폐쇄를 Scan Context로 대체했다. Scan Context는 큰 누적 오차에도 강인한 LiDAR 장소 인식 기법으로, 전역 일관성을 크게 향상시킨다.19
  - **LB-LIOSAM:** 텍스처가 부족하거나 반복적인 구조의 환경에서 더 나은 성능을 위해 BoW3D(Bag-of-Words for 3D) 루프 폐쇄 기법을 통합했다.48
  - **LIO-SAM-6AXIS-INTENSITY:** LiDAR의 강도(intensity) 값으로 생성한 이미지를 이용하여 시각 SLAM과 유사한 루프 폐쇄를 수행한다. 이는 실내 환경에서 효과적이지만 고해상도 LiDAR를 필요로 한다.55

### 7.2  성능 및 속도 증대

- **GPU 가속:**
  - **LIO-SAM-GPU-ScanToMapOpt:** 라인/평면 오도메트리(스캔 정합) 부분을 CUDA로 재구현하고, GPU 기반 해시 맵을 사용하여 k-NN 탐색을 가속화함으로써 프레임당 처리 시간을 획기적으로 줄였다.37
- **하이브리드 시스템:**
  - **FAST-LIO-SAM:** FAST-LIO2의 매우 효율적인 프론트엔드(직접 방식, iKD-Tree)와 LIO-SAM의 강인하고 전역 최적화가 가능한 백엔드(루프 폐쇄 기능이 있는 팩터 그래프)를 결합한 인기 있는 파생 프로젝트다. 이는 속도와 전역 정확도라는 두 마리 토끼를 모두 잡으려는 시도다.26

### 7.3  기능 확장

- **다중 센서 융합:**
  - **LVI-SAM:** LIO-SAM과 VINS-Mono를 강결합하여, 각 시스템의 단점을 상호 보완하는 LiDAR-시각-관성 시스템을 구축했다.46
  - **LPVIMO-SAM:** LiDAR, 편광 카메라, IMU, 지자기 센서, 광학 흐름 센서를 단일 팩터 그래프에 융합하여, 텍스처가 매우 부족하거나 LiDAR 성능이 저하되는 극한의 환경에 대응하기 위한 연구 프레임워크다.1
- **새로운 기능 추가:**
  - **LIO-SAM-Localization:** 핵심 알고리즘을 수정하여, 새로운 지도를 생성하는 대신 사전에 구축된 지도 내에서 위치를 추정하는 기능(localization)을 구현했다. 이는 실제 로봇 응용에 필수적인 기능이다.56
- **다중 로봇 SLAM:**
  - 직접적인 파생 프로젝트는 아니지만, LIO-SAM은 종종 다중 로봇 SLAM 연구에서 개별 로봇의 오도메트리 백본(backbone)으로 사용된다. 이 경우, 로봇 간의 장소 인식 및 지도 병합이 핵심 과제가 된다.57

------

**표 4: 주요 LIO-SAM 파생 프로젝트 요약**

| 파생 프로젝트 명         | 핵심 기여                   | 해결하고자 하는 문제                         |
| ------------------------ | --------------------------- | -------------------------------------------- |
| **SC-LIO-SAM**           | Scan Context 통합           | 취약한 루프 폐쇄 / 큰 누적 오차 19           |
| **RF-LIO**               | 동적 객체 선제적 제거       | 동적 환경에서의 부정확한 정합 18             |
| **LIO-SAM++**            | 의미론적 제약 조건 추가     | 동적 객체, 부정확한 특징점 연관 53           |
| **FAST-LIO-SAM**         | FAST-LIO2 프론트엔드 융합   | 특징점 기반 프론트엔드의 속도 한계 26        |
| **LVI-SAM**              | VINS-Mono 융합              | VIO/LIO 개별 시스템의 실패 사례 46           |
| **LIO-SAM-Localization** | 기존 지도 내 위치 추정 기능 | 매핑(Mapping) vs. 측위(Localization) 문제 56 |
| **LIO-SAM-GPU**          | GPU 가속 오도메트리         | k-NN 탐색에서의 CPU 병목 현상 37             |

## 8.  결론 및 향후 전망

### 8.1  LIO-SAM의 영향 종합

LIO-SAM은 고성능의 접근성 높은 오픈소스 구현을 통해 강결합, 팩터 그래프 기반의 LiDAR-관성 SLAM을 대중화시킨 결정적인 기여를 했다. 이 알고리즘은 수많은 학술 논문에서 표준 기준선으로 사용되고 있으며 17, 이를 기반으로 한 풍부한 확장 생태계가 형성되었다는 사실 자체가 LIO-SAM이 연구 커뮤니티에 미친 지대한 영향을 증명한다.48 LIO-SAM은 단순히 하나의 뛰어난 알고리즘을 넘어, LiDAR-관성 융합 연구를 위한 견고한 발판이자 플랫폼으로서의 역할을 성공적으로 수행했다.

### 8.2  남겨진 과제와 미해결 연구 문제

- **장기 자율성 (Long-Term Autonomy):** 많은 개선에도 불구하고, GPS 없이 매우 오랜 시간(수일, 수주) 동안 누적 오차 없이 강인하게 동작하는 것은 여전히 해결되지 않은 과제다. 이는 지도 유지보수, 변화하는 환경에 대한 적응, 그리고 계산량 확장성 문제를 포함한다.
- **극한 환경에서의 강인성:** 짙은 안개나 먼지와 같은 시야 방해물, 극심한 센서 진동, 또는 비구조적인 자연환경과 같은 열악한 조건에서 성능을 한계까지 끌어올리는 것은 계속 진행 중인 연구 분야다.51
- **의미론적 이해:** 단순한 기하학적 지도를 넘어 진정한 장면 이해로 나아가는 것이 중요하다. 의미론적 SLAM이 등장하고 있지만 52, 더 지능적인 데이터 연관(data association) 및 고수준 추론을 위해 의미론적 지식을 시스템에 깊이 통합하는 것이 미래의 핵심 연구 방향이 될 것이다.

### 8.3  LiDAR-관성 SLAM의 미래 궤적

- **더 깊은 융합:** 악천후에 강한 레이더(Radar)나 열화상 카메라 등 더 다양한 센서 모달리티와의 통합이 계속될 것이다.51
- **협력적 SLAM (Collaborative SLAM):** 단일 로봇으로는 불가능한 작업을 수행하기 위해, 강인하고 분산적이며 통신 효율적인 다중 로봇 SLAM 시스템을 개발하는 것이 중요한 차세대 연구 분야다.57
- **학습 기반 접근법:** LIO-SAM은 고전적인 모델 기반 접근법이지만, 미래에는 특징점 추출, 데이터 연관, 또는 퇴화 상황 감지와 같은 구성 요소를 딥러닝으로 강화하여 모델 기반 방식과 학습 기반 방식의 장점을 결합한 하이브리드 시스템이 주를 이룰 것으로 예상된다.

결론적으로, LIO-SAM은 LiDAR-관성 SLAM 분야에 하나의 이정표를 세웠다. 그 아키텍처와 아이디어는 현재까지도 수많은 연구에 영감을 주고 있으며, 그 한계점들은 오히려 미래 연구가 나아가야 할 방향을 명확히 제시해주고 있다. 앞으로의 SLAM 기술은 LIO-SAM이 다진 견고한 토대 위에서 더욱 강인하고, 지능적이며, 협력적인 방향으로 진화해 나갈 것이다.

#### **참고 자료**

1. LPVIMO-SAM: Tightly-coupled LiDAR/Polarization Vision/Inertial/Magnetometer/Optical Flow Odometry via Smoothing and Mapping - arXiv, accessed July 18, 2025, https://arxiv.org/html/2504.20380v1
2. Lio-sam: Tightly-coupled lidar inertial odometry via smoothing and mapping - arXiv, accessed July 18, 2025, http://arxiv.org/pdf/2007.00258
3. LiDAR SLAM vs. Visual SLAM: An In-depth Comparison-Geosun Navigation, accessed July 18, 2025, https://en.geosuntech.com/News/252.html
4. A Review of Multi-Sensor Fusion SLAM Systems Based on 3D LIDAR - MDPI, accessed July 18, 2025, https://www.mdpi.com/2072-4292/14/12/2835
5. Benchmarking and Comparing Popular Visual SLAM Algorithms - Asian Journal of Convergence in Technology, accessed July 18, 2025, https://www.asianssr.org/index.php/ajct/article/download/757/605
6. LiDAR SLAM vs. Visual SLAM: Key Differences for Smarter Mapping Decisions, accessed July 18, 2025, https://www.geosunlidar.com/news/lidar-slam-vs-visual-slam-key-differences-for-smarter-mapping-decisions-190902.html
7. Quantitative comparison of the proposed method, LIO-SAM and ..., accessed July 18, 2025, https://www.researchgate.net/figure/Quantitative-comparison-of-the-proposed-method-LIO-SAM-and-LeGO-LOAM-m_tbl2_366690779
8. Comparison of Visual and LiDAR SLAM Algorithms using NASA Flight Test Data, accessed July 18, 2025, https://ntrs.nasa.gov/api/citations/20220017949/downloads/Comparison_of_Visual_and_LiDAR_SLAM_Algorithms_using_NASA_Flight_Test_Data_FINAL.pdf
9. Visual vs Lidar SLAM : r/robotics - Reddit, accessed July 18, 2025, https://www.reddit.com/r/robotics/comments/1g8ik4r/visual_vs_lidar_slam/
10. A Framework for Reproducible Benchmarking and Performance Diagnosis of SLAM Systems - arXiv, accessed July 18, 2025, https://arxiv.org/html/2410.04242v1
11. SLAM without lidar sensor? : r/robotics - Reddit, accessed July 18, 2025, https://www.reddit.com/r/robotics/comments/1127uhn/slam_without_lidar_sensor/
12. A Review of Research on SLAM Technology Based on the Fusion of LiDAR and Vision, accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11902412/
13. M2UD: A Multi-model, Multi-scenario, Uneven-terrain Dataset for Ground Robot with Localization and Mapping Evaluation - arXiv, accessed July 18, 2025, https://arxiv.org/html/2503.12387v1
14. Tightly-Coupled LiDAR-Visual-Inertial SLAM and Large-Scale Volumetric Occupancy Mapping - arXiv, accessed July 18, 2025, https://arxiv.org/html/2403.02280v1
15. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2007.00258
16. LIO_SAM - Autoware Documentation, accessed July 18, 2025, https://autowarefoundation.github.io/autoware-documentation/pr-347/how-to-guides/creating-maps-for-autoware/open-source-slam/lio-sam/
17. LiDAR Inertial Odometry Based on Indexed Point and Delayed Removal Strategy in Highly Dynamic Environments - PMC, accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10255994/
18. [2206.09463] RF-LIO: Removal-First Tightly-coupled Lidar Inertial Odometry in High Dynamic Environments - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2206.09463
19. gisbi-kim/SC-LIO-SAM: LiDAR-inertial SLAM - GitHub, accessed July 18, 2025, https://github.com/gisbi-kim/SC-LIO-SAM
20. Performance Analysis of LIO-SAM - Ronia Peterson, accessed July 18, 2025, http://www.roniapeterson.com/LIO-SAM.pdf
21. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 18, 2025, https://github.com/TixiaoShan/LIO-SAM
22. README.md - LIO-SAM - GitLab, accessed July 18, 2025, https://gitlab.uni-hannover.de/esistoiamk/LIO-SAM/-/blob/master/README.md
23. NV-LIO: LiDAR-Inertial Odometry using Normal Vectors Towards Robust SLAM in Multifloor Environments - arXiv, accessed July 18, 2025, https://arxiv.org/html/2405.12563v2
24. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - MDPI, accessed July 18, 2025, https://www.mdpi.com/1424-8220/24/23/7546
25. LIO-SAM on Yonohub - by Ahmed Radwan - Medium, accessed July 18, 2025, https://medium.com/yonohub/lio-sam-on-yonohub-a163fa6bac49
26. a SLAM implementation combining FAST-LIO2 with pose graph optimization and loop closing based on LIO-SAM paper - GitHub, accessed July 18, 2025, https://github.com/engcang/FAST-LIO-SAM
27. Impact of 3D LiDAR Resolution in Graph-based SLAM Approaches: A Comparative Study, accessed July 18, 2025, https://arxiv.org/html/2410.17171v1
28. Problems with other dataset · Issue #2 · TixiaoShan/LIO-SAM - GitHub, accessed July 18, 2025, https://github.com/TixiaoShan/LIO-SAM/issues/2
29. LiDAR Based SLAM: A Comparative Evaluation with LeGO-LOAM and HDL-Graph SLAM - DergiPark, accessed July 18, 2025, https://dergipark.org.tr/en/download/article-file/3761251
30. LiDAR Based SLAM: A Comparative Evaluation with LeGO-LOAM and HDL-Graph SLAM, accessed July 18, 2025, https://dergipark.org.tr/en/pub/umagd/issue/90102/1444519
31. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - PMC, accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11644182/
32. Performance Assessment of Lidar Odometry Frameworks: A Case Study at the Australian Botanic Garden Mount Annan - arXiv, accessed July 18, 2025, https://arxiv.org/html/2411.16931v1
33. An Accurate LiDAR-Inertial SLAM Based on Multi-Category Feature ..., accessed July 18, 2025, https://www.mdpi.com/2072-4292/17/14/2425
34. (PDF) LIO-CSI: LiDAR inertial odometry with loop closure combined with semantic information - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/356887770_LIO-CSI_LiDAR_inertial_odometry_with_loop_closure_combined_with_semantic_information
35. LIO-CSI: LiDAR inertial odometry with loop closure combined with semantic information, accessed July 18, 2025, https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0261053
36. LIO-CSI: LiDAR inertial odometry with loop closure combined with semantic information, accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8654169/
37. qdLMF/LIO-SAM-GPU-ScanToMapOpt: A CUDA reimplementation of the line/plane odometry of LIO-SAM. A point cloud hash map (inspired by iVox of Faster-LIO) on GPU is used to accelerate 5-neighbour KNN search. - GitHub, accessed July 18, 2025, https://github.com/qdLMF/LIO-SAM-GPU-ScanToMapOpt
38. Reconstruct jump and error · Issue #433 · TixiaoShan/LIO-SAM - GitHub, accessed July 18, 2025, https://github.com/TixiaoShan/LIO-SAM/issues/433
39. How does this compare with LeGO-LOAM? · Issue #151 · TixiaoShan/LIO-SAM - GitHub, accessed July 18, 2025, https://github.com/TixiaoShan/LIO-SAM/issues/151
40. LiPO: LiDAR Inertial Odometry for ICP Comparison - arXiv, accessed July 18, 2025, https://arxiv.org/html/2410.08097v1
41. Voxel-SLAM: A Complete, Accurate, and Versatile LiDAR-Inertial SLAM System - arXiv, accessed July 18, 2025, https://arxiv.org/html/2410.08935v1
42. SR-LIO: LiDAR-Inertial Odometry with Sweep Reconstruction - arXiv, accessed July 18, 2025, https://arxiv.org/pdf/2210.10424
43. Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - arXiv, accessed July 18, 2025, https://arxiv.org/pdf/2302.04031
44. VINS-Mono: A Robust and Versatile Monocular Visual-Inertial State Estimator, accessed July 18, 2025, https://www.researchgate.net/publication/319122049_VINS-Mono_A_Robust_and_Versatile_Monocular_Visual-Inertial_State_Estimator
45. Visual-Inertial SLAM Comparison - Bharat Joshi - GitHub Pages, accessed July 18, 2025, https://joshi-bharat.github.io/projects/visual_slam_comparison/
46. LVI-SAM: Tightly-coupled Lidar-Visual-Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 18, 2025, https://github.com/TixiaoShan/LVI-SAM
47. (PDF) LVI-SAM: Tightly-coupled Lidar-Visual-Inertial Odometry via Smoothing and Mapping, accessed July 18, 2025, https://www.researchgate.net/publication/351063772_LVI-SAM_Tightly-coupled_Lidar-Visual-Inertial_Odometry_via_Smoothing_and_Mapping
48. LB-LIOSAM: an improved mapping and localization method with loop detection | Emerald Insight, accessed July 18, 2025, https://www.emerald.com/insight/content/doi/10.1108/ir-07-2024-0314/full/html
49. LIO-GC: LiDAR Inertial Odometry with Adaptive Ground Constraints - MDPI, accessed July 18, 2025, https://www.mdpi.com/2072-4292/17/14/2376
50. Ground-optimized SLAM with Hierarchical Loop Closure Detection in Large-scale Environment - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/378192838_Ground-optimized_SLAM_with_Hierarchical_Loop_Closure_Detection_in_Large-scale_Environment
51. [2403.05332] Degradation Resilient LiDAR-Radar-Inertial Odometry - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2403.05332
52. Semantic Lidar-Inertial SLAM for Dynamic Scenes - MDPI, accessed July 18, 2025, https://www.mdpi.com/2076-3417/12/20/10497
53. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/386200068_LIO-SAM_A_Lidar-Inertial_Semantic_SLAM_with_Association_Optimization_and_Keyframe_Selection
54. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - PubMed, accessed July 18, 2025, https://pubmed.ncbi.nlm.nih.gov/39686083/
55. LIO-SAM-6AXIS with intensity image loop optimization - GitHub, accessed July 18, 2025, https://github.com/JokerJohn/LIO-SAM-6AXIS-INTENSITY
56. harshalkataria/LIO-SAM-Localization: Estimate the odometry of a robot in pre-built map using LIO-SAM - GitHub, accessed July 18, 2025, https://github.com/harshalkataria/LIO-SAM-Localization
57. S3E: A Multi-Robot Multimodal Dataset for Collaborative SLAM - arXiv, accessed July 18, 2025, https://arxiv.org/html/2210.13723v8
58. Enhanced Multi-Robot SLAM System with Cross-Validation Matching and Exponential Threshold Keyframe Selection - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/384700240_Enhanced_Multi-Robot_SLAM_System_with_Cross-Validation_Matching_and_Exponential_Threshold_Keyframe_Selection
59. A flexible framework for accurate LiDAR odometry, map manipulation, and localization, accessed July 18, 2025, https://arxiv.org/html/2407.20465v1
60. Tightly Coupled 3D Lidar Inertial Odometry and Mapping | Request PDF - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/335143697_Tightly_Coupled_3D_Lidar_Inertial_Odometry_and_Mapping


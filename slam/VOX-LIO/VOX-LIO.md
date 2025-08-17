# VOX-LIO

## 서론

### 0.1  LiDAR-Inertial Odometry (LIO)의 중요성 및 당면 과제

자율주행 자동차, 로봇, 증강현실(AR)과 같은 현대 기술의 근간에는 주변 환경을 인식하고 자신의 위치와 자세를 정밀하게 추정하는 능력이 자리 잡고 있다. 이를 위한 핵심 기술이 바로 6자유도(6-DoF) 자세 추정 및 동시적 지도 작성(Simultaneous Localization and Mapping, SLAM)이다. 다양한 센서 조합 중, LiDAR(Light Detection and Ranging)와 관성 측정 장치(Inertial Measurement Unit, IMU)를 융합하는 LiDAR-Inertial Odometry(LIO)는 가장 유망한 해결책 중 하나로 부상했다. LiDAR는 주변 환경에 대한 정밀한 3차원 거리 정보를 제공하는 반면, IMU는 높은 시간 해상도로 가속도와 각속도 등 플랫폼의 동역학 정보를 제공하여 상호 보완적인 관계를 형성한다.1

하지만 LIO 시스템 개발은 **정확성(Accuracy)**, **강인성(Robustness)**, **효율성(Efficiency)**이라는 세 가지 목표 사이의 근본적인 상충 관계(trilemma)에 직면한다.4 특히, 로봇의 급격하고 공격적인 움직임(aggressive motion), 터널이나 복도처럼 기하학적 특징이 부족한 환경(feature-degenerate environments), 그리고 제한된 연산 자원을 가진 임베디드 플랫폼에서의 실시간 성능 확보는 LIO 기술이 풀어야 할 핵심적인 난제로 남아있다.5

### 0.2  VOX-LIO: 개요 및 연구 목표

이러한 문제들을 해결하기 위한 최신 접근법으로 **VOX-LIO**가 제안되었다. VOX-LIO는 효율적인 아키텍처와 매니폴드(manifold) 공간 상에서의 가중 융합(weighted fusion) 기법을 결합하여, 자원이 제한된 플랫폼에서도 실시간으로 고정밀 맵핑 및 위치 추정을 달성하는 것을 목표로 하는 LIO 시스템이다.5

본 보고서는 다음과 같은 목표를 가진다. 첫째, VOX-LIO를 구성하는 핵심 기술적 기여를 심층적으로 분석한다. 둘째, LIO-SAM, FAST-LIO2와 같은 최신 LIO 시스템과의 구조적, 방법론적 비교를 통해 VOX-LIO의 기술적 위치와 장단점을 규명한다. 셋째, KITTI와 같은 공개 벤치마크 데이터셋을 이용한 정량적 성능 평가 결과를 바탕으로 VOX-LIO의 실질적인 성능을 검증한다. 마지막으로, 이를 통해 LIO 기술의 전반적인 동향을 파악하고 미래 발전 방향을 조망한다.

## 1.  LiDAR-Inertial Odometry 기술 동향

### 1.1  센서 융합 프레임워크

LIO 시스템은 LiDAR와 IMU 데이터를 융합하는 방식에 따라 크게 강결합(Tightly-Coupled)과 약결합(Loosely-Coupled)으로 나뉜다.

- **강결합 (Tightly-Coupled)**: 이 방식은 LiDAR의 포인트 클라우드와 IMU의 각속도 및 가속도와 같은 원시 측정값(raw measurements)을 단일 최적화 문제 내에서 함께 처리한다. 센서 간의 상호 의존성을 직접 모델링하므로 상태 추정의 정확도와 강인성을 크게 높일 수 있으며, 센서 간의 외부 파라미터(extrinsic parameters)나 IMU 바이어스(bias)를 온라인으로 추정하는 데 유리하다. VOX-LIO를 포함하여 FAST-LIO, LIO-SAM 등 대부분의 최신 고성능 시스템이 이 방식을 채택하고 있다.7
- **약결합 (Loosely-Coupled)**: 이 방식은 LiDAR 모듈과 IMU 모듈이 각각 독립적으로 자세를 추정한 후, 그 결과를 후처리 단계에서 융합한다. 구조가 간단하고 구현이 용이하지만, 각 센서의 추정 과정에서 정보 손실이 발생할 수 있다. 이로 인해 한 센서의 성능 저하가 전체 시스템의 실패로 이어질 수 있으며, 일반적으로 강결합 방식보다 정확도가 떨어진다.11

### 1.2  상태 추정 기법

강결합 프레임워크 내에서 상태를 추정하는 방법론은 크게 필터 기반과 최적화 기반으로 양분된다.

- **필터 기반 (Filter-based) 접근법**: 주로 확장 칼만 필터(EKF)나 그 변형인 반복 확장 칼만 필터(Iterated EKF, iEKF)를 사용한다. 이 접근법은 시스템의 상태 변수와 그 불확실성을 가우시안 분포로 모델링하고, 새로운 측정값이 들어올 때마다 상태를 예측하고 갱신하는 재귀적인 구조를 가진다. **FAST-LIO** 계열이 대표적인 예시로, 이들은 상태 변수의 차원에만 의존하는 새로운 형태의 칼만 이득(Kalman Gain) 계산식을 도입하여 연산 효율을 극적으로 향상시켰다.10 필터 기반 방식은 실시간 성능이 매우 뛰어나지만, 비선형 시스템을 선형화하는 과정에서 발생하는 오차로 인해 전역적으로 최적의 해를 보장하지 못할 수 있다는 한계가 있다.13
- **최적화 기반 (Optimization-based) 접근법**: 주로 특정 시간 윈도우(sliding window) 내의 모든 측정값과 상태 변수 간의 관계를 요인 그래프(Factor Graph)로 모델링한다. 그 후, 전체 비용 함수(cost function)를 최소화하는 상태 변수들을 찾아 해를 구한다. **LIO-SAM**이 이 접근법의 대표적인 예시이며, IMU 사전통합(preintegration), LiDAR odometry, GPS, 루프 클로저(loop closure) 등 다양한 요인(factor)을 유연하게 통합하여 전역적으로 일관된 궤적을 추정할 수 있다.9 이 방식은 필터 기반 방식보다 더 정확한 해를 찾을 수 있는 잠재력을 가지지만, 일반적으로 더 많은 연산량을 요구한다.9

### 1.3  맵 표현을 위한 자료구조

LIO 시스템의 효율성은 맵을 표현하고 관리하는 자료구조에 크게 의존한다. 특히, 새로운 스캔을 기존 맵에 정합하기 위한 최근접 이웃 탐색(k-Nearest Neighbor, k-NN)은 가장 큰 연산 병목 중 하나이다. LIO 자료구조의 발전 과정은 탐색 효율성과 메모리 관리 사이의 근본적인 긴장 관계를 반영한다. 초기 k-d 트리 구조에서 시작하여 트리 유지 보수 오버헤드를 없앤 해시 기반 복셀, 그리고 해시 충돌마저 제거한 배열 기반 복셀로의 진화는 순수한 속도 향상을 위한 끊임없는 노력을 보여준다. 이러한 흐름 속에서 VOX-LIO가 서펠(surfel) 논리를 해시 복셀에 통합한 것은, 다음 단계의 발전이 단순히 더 빠른 구조가 아니라, 속도와 표현력을 모두 갖춘 '더 스마트한' 구조를 지향함을 시사한다.

- **K-D 트리 및 변형 (K-D Trees and Variants)**: 포인트 클라우드 검색에 전통적으로 사용되어 온 자료구조다. 특히 **FAST-LIO2**에서 제안된 **ikd-Tree**는 포인트의 증분적(incremental) 추가, 삭제, 그리고 동적 재조정이 효율적으로 이루어져 동적인 맵 관리에 큰 강점을 보인다.12

- **복셀 그리드 (Voxel Grids)**: 공간을 정육면체 셀(복셀)로 나누어 관리하는 방식으로, 최근 LIO 시스템에서 각광받고 있다.

  - **해시 기반 복셀 (Hash-based Voxels)**: **Faster-LIO**의 `iVox`가 대표적이다. 해시 테이블을 사용하여 맵을 관리하므로 트리의 재균형(rebalancing) 과정이 없어 삽입 및 삭제가 매우 빠르다.4 하지만 해시 충돌(collision) 처리를 위한 추가적인 오버헤드가 발생할 수 있다.4

    **VOX-LIO** 역시 적응형 해시 복셀을 기본 자료구조로 채택했다.5

  - **배열 기반 복셀 (Array-based Voxels)**: **FR-LIO**의 `RC-Vox`는 로봇 중심의 고정된 크기를 갖는 2계층 3D 배열을 사용한다. 모듈로(modulo) 연산을 통해 맵 인덱스를 계산하므로 해시 충돌이 원천적으로 없고 접근 속도가 극도로 빠르다. 그러나 맵의 크기가 고정되어 있어 메모리 관리의 유연성이 떨어진다는 단점이 있다.4

- **서펠 (Surfels)**: 점(point)을 단순히 3차원 좌표로 저장하는 것을 넘어, 해당 점 주변의 국소적인 표면 정보(법선 벡터, 곡률, 반경 등)를 함께 디스크 형태로 표현하는 방식이다. **VOX-LIO**는 복셀 내에 이러한 서펠 특징을 통합하여, 단순한 점대점(point-to-point) 또는 점대평면(point-to-plane) 정합보다 훨씬 풍부한 기하학적 정보를 활용함으로써 정합의 정확도와 강인성을 높인다.5

## 2.  VOX-LIO 시스템 심층 분석

### 2.1  시스템 아키텍처 개요

VOX-LIO는 자원 제약적 플랫폼에서도 강인성과 정확도를 동시에 달성하기 위해 설계된 강결합 LIO 시스템이다.8 시스템의 핵심은 크게 세 가지로 구성된다: (1) 평면성(planarity)과 서펠 특징을 활용하는 적응형 해시 복셀 기반 포인트 클라우드 매칭, (2) 매니폴드 공간 상에서 LiDAR와 IMU 측정치를 가중 융합하는 상태 추정, 그리고 (3) 루프 클로저를 통해 전역 맵의 일관성을 보장하는 전역 최적화 모듈 (VOX-LIOM)이다.5

### 2.2  핵심 기여 1: 적응형 서펠-복셀 매핑

VOX-LIO의 가장 독창적인 기여는 맵 관리 및 포인트 정합 방식에 있다.

- **적응형 해시 복셀 (Adaptive Hash Voxels)**: VOX-LIO는 공간을 해시 복셀로 관리하되, 각 복셀 내에 포함된 포인트들의 분포를 분석하여 평면성을 계산한다. 이 평면성 값을 기준으로 복셀을 **서펠 복셀(surfel voxels)**과 **일반 복셀(common voxels)**로 동적으로 분류한다.5 이는 평평한 벽이나 바닥과 같이 구조적 특징이 뚜렷한 영역과 그렇지 않은 영역을 구분하여 처리 전략을 달리함으로써, 효율성과 정확성을 동시에 추구하는 지능적인 접근법이다.

- **서펠 특징의 점진적 정제 및 재사용 (Incremental Refinement and Reuse of Surfel Features)**: '서펠 복셀'로 분류된 영역에서는 내부 포인트들의 공분산 행렬을 계산하여 해당 평면을 가장 잘 나타내는 서펠(평균 위치와 법선 벡터) 특징을 추출하고 저장한다. 이 서펠 정보는 시간이 지나면서 지속적으로 관측될 경우, 새로운 스캔을 정합할 때 재사용되어 최적화의 제약 조건으로 활용된다. 이는 매번 주변 포인트들을 찾아 평면 피팅을 수행하는 기존 방식의 계산 비용을 획기적으로 줄여준다.5 또한, 새로운 포인트가 해당 복셀에 추가되면 기존 서펠 특징을 점진적으로 업데이트(refine)하여 맵의 정확도를 지속적으로 향상시킨다.5

- **수학적 모델링**: 새로운 스캔에서 얻은 포인트 pi가 주어졌을 때, 이 포인트가 속한 복셀이 '서펠 복셀'로 분류되었다면, 해당 복셀에 저장된 서펠 특징(평균 $\boldsymbol{\mu}$와 법선 벡터 n)을 이용한 점대평면(point-to-surfel) 오차 잔차(residual) ri를 계산한다. 이 잔차는 전체 최적화 문제의 비용 함수에 포함되어 최소화의 대상이 된다.
  $$
  r_i = \mathbf{n}^T (\mathbf{p}_i - \boldsymbol{\mu})
  $$

### 2.3  핵심 기여 2: 매니폴드 상에서의 가중 융합

전통적으로 LIO 시스템은 빠른 속도를 중시하는 필터 기반 방식과 정확도를 중시하는 최적화 기반 방식으로 나뉘어 발전해왔다. 그러나 최근에는 이 두 패러다임의 경계가 모호해지며 아이디어가 융합되는 추세가 나타나고 있다. FAST-LIO2와 같은 반복 칼만 필터는 지역적인 최적화를 수행하며, LIO-SAM과 같은 슬라이딩 윈도우 최적화기는 지역 필터처럼 동작한다. VOX-LIO의 접근 방식은 이러한 융합의 대표적인 사례다. 프론트엔드 odometry를 위해 "매니폴드 상에서의 가중 융합"과 "진보된 최적화 기법"을 사용하는데, 이는 매니폴드 기하학의 엄밀함을 갖춘 지역적 비선형 최적화 문제로, 필터와 최적화의 특징을 모두 가진다. 여기에 VOX-LIOM 버전은 전역 요인 그래프 백엔드를 추가하여, 필터/지역 최적화기의 실시간 효율성과 그래프 기반 방식의 전역 일관성을 결합하는 것이 최적의 아키텍처임을 보여준다.

- **IMU 측정치를 이용한 모션 왜곡 보정 (Motion Distortion Correction)**: LiDAR 센서는 한 번의 스캔을 완료하는 데 시간이 걸리므로, 그동안 센서가 움직이면 포인트 클라우드가 왜곡(skew)되는 현상이 발생한다. VOX-LIO는 IMU의 높은 주파수(high-frequency) 출력을 이용해 각 포인트가 측정된 정확한 시점의 센서 포즈를 추정하고, 모든 포인트를 스캔 시작 또는 종료 시점의 단일 좌표계로 변환하여 이 왜곡을 효과적으로 보정한다.5

- **매니폴드 상에서의 상태 표현 및 최적화 (State Representation and Optimization on Manifold)**: 로봇의 상태(자세, 속도, IMU 바이어스)를 표현할 때, 특히 회전은 일반적인 유클리드 공간이 아닌 회전 그룹 $SO(3)$와 같은 매니폴드 공간에서 직접 다룬다. 이는 짐벌락(gimbal lock)과 같은 회전 표현의 특이점(singularity) 문제를 근본적으로 방지하고 최적화 과정의 수렴성을 향상시킨다. 시스템의 상태 변수 $\mathbf{x}$는 다음과 같이 표현될 수 있다.
  $$
  \mathbf{x} =^T
  $$
  여기서 $\mathbf{R}, \mathbf{p}, \mathbf{v}$는 각각 회전, 위치, 속도를 나타내며, ba,bg는 가속도계와 자이로스코프의 바이어스를 의미한다.

- **가중 융합 전략 (Weighted Fusion Strategy)**: LiDAR 측정값에서 계산된 기하학적 잔차와 IMU 사전통합(pre-integration)에서 계산된 동역학적 잔차를 융합할 때, 각각의 불확실성(uncertainty)에 기반한 가중치를 부여하여 전체 비용 함수를 구성한다. 이는 각 센서 데이터의 신뢰도에 따라 융합 과정에서의 기여도를 동적으로 조절하는 강인한(robust) 방식이다. 특히, 급격한 움직임으로 인해 IMU 측정값의 신뢰도가 일시적으로 떨어질 경우, 강인한 커널 함수(robust kernel function)를 적용하여 이상치(outlier)가 전체 추정 결과에 미치는 영향을 효과적으로 줄인다.5

### 2.4  핵심 기여 3: 전역 최적화 (VOX-LIOM)

VOX-LIO 시스템에 루프 클로저(loop closure) 모듈과 전역 포즈 그래프 최적화(global pose graph optimization) 기능을 통합한 확장 버전을 **VOX-LIOM**이라 칭한다.5 이는 장시간, 대규모 환경에서의 운용 시 발생하는 누적 오차(drift) 문제를 해결하기 위한 필수적인 구성 요소다.

- **루프 탐지 (Loop Detection)**: 새로운 키프레임(keyframe)이 시스템에 추가될 때마다, 이전에 저장된 과거의 키프레임들과의 기하학적 근접성을 기준으로 루프 후보를 탐색한다. 후보가 발견되면, 두 키프레임 간의 정밀한 스캔 정합(scan matching)을 수행하여 상대적인 변환 관계를 계산한다.
- **포즈 그래프 최적화 (Pose Graph Optimization)**: 성공적으로 검증된 루프 클로저는 포즈 그래프에 새로운 제약 조건(factor)으로 추가된다. 포즈 그래프는 시스템의 키프레임들을 노드(node)로, 이들 사이의 상대적 움직임(odometry factor, loop closure factor)을 엣지(edge)로 표현한 자료구조다. 새로운 루프 클로저 제약이 추가되면, 그래프 전체에 걸쳐 누적된 오차를 재분배하는 전역 최적화를 수행한다. 이 과정을 통해 궤적의 전역적인 일관성을 확보하고, 시간이 지남에 따라 쌓인 드리프트를 효과적으로 보정하여 정확한 맵을 생성할 수 있다.5

## 3.  주요 LIO 시스템과의 비교 분석

VOX-LIO의 기술적 특징과 위치를 명확히 하기 위해, 현재 LIO 분야를 대표하는 LIO-SAM, FAST-LIO2, Faster-LIO, FR-LIO 시스템과 비교 분석한다.

| 특성 (Feature)     | **VOX-LIO**            | **LIO-SAM**           | **FAST-LIO2**              | **FR-LIO**                 |
| ------------------ | ---------------------- | --------------------- | -------------------------- | -------------------------- |
| **핵심 추정 방식** | 가중 융합 최적화       | 요인 그래프 최적화    | 반복 확장 칼만 필터 (iEKF) | 반복 칼만 스무더 (ESKS)    |
| **자료구조**       | 서펠-해시 복셀         | 키프레임 기반 복셀 맵 | 증분 K-D 트리 (ikd-Tree)   | 로봇 중심 3D 배열 (RC-Vox) |
| **사용 특징**      | 서펠 / 평면 특징       | 에지 / 평면 특징      | 원시 포인트 (Raw Points)   | 원시 포인트 (Raw Points)   |
| **융합 전략**      | 강결합                 | 강결합                | 강결합                     | 강결합                     |
| **전역 최적화**    | 루프 클로저 (VOX-LIOM) | 루프 클로저, GPS      | -                          | -                          |

### 3.1  vs. LIO-SAM (최적화 기반 대표 주자)

LIO-SAM은 최적화 기반 LIO의 대표 주자로서 VOX-LIO와 많은 공통점을 가지지만, 핵심적인 차이점도 존재한다.

- **공통점**: 두 시스템 모두 강결합 방식을 채택하고, IMU를 이용한 왜곡 보정을 수행하며, 루프 클로저를 통한 전역 최적화 기능을 제공한다.
- **차이점**:
  - **프론트엔드 정합**: LIO-SAM은 포인트들의 곡률(curvature)을 계산하여 에지(edge)와 평면(planar) 특징점을 명시적으로 추출하고 이를 정합에 사용한다.9 반면, VOX-LIO는 평면성을 기준으로 복셀을 분류하고 암시적으로 서펠 특징을 활용한다. VOX-LIO의 방식은 구조화되지 않은 자연 환경 등에서 더 강인한 성능을 보일 수 있다.
  - **맵 관리**: LIO-SAM은 고정된 개수의 이전 키프레임들로 구성된 지역 서브맵(local sub-map)을 슬라이딩 윈도우 방식으로 관리하여 실시간성을 확보한다.9 이에 비해 VOX-LIO는 해시 기반의 전역 복셀 맵을 점진적으로 구축하고 업데이트하는 방식을 사용한다.5 LIO-SAM의 방식이 연산량 측면에서 유리할 수 있으나, 맵의 전역적 표현을 유지하는 데는 VOX-LIO가 더 직접적인 구조를 가진다.
  - **상태 추정**: LIO-SAM은 요인 그래프 라이브러리(GTSAM)를 활용하여 슬라이딩 윈도우 내의 모든 요소를 고려한 최대 사후 확률(MAP) 추정을 수행한다.9 VOX-LIO는 매니폴드 상에서 가중치를 적용한 잔차를 최소화하는 지역 최적화에 더 가깝다.

### 3.2  vs. FAST-LIO2 & Faster-LIO (필터 기반 대표 주자)

FAST-LIO 계열은 필터 기반 LIO의 정점으로, 극도의 효율성을 자랑한다.

- **공통점**: 두 시스템 모두 강결합 방식을 사용하며 높은 계산 효율성을 추구한다.

- **차이점**:

  - **상태 추정**: FAST-LIO 계열은 iEKF라는 필터 기반 방식을 사용하는 반면, VOX-LIO는 최적화 기반의 융합 방식을 사용한다.10 이론적으로는 최적화 방식이 더 정확한 해를 제공할 잠재력이 높다.

  - **특징 사용**: FAST-LIO2는 특징 추출 과정을 생략하고 원시 포인트(raw points)를 직접 맵에 등록하는 '직접 방식(direct method)'을 채택하여, 다양한 종류의 LiDAR 센서에 대한 범용성을 확보했다.12 VOX-LIO는 평면성을 기준으로 포인트를 분류하고 서펠 특징을 활용하므로, 환경의 기하학적 구조를 더 명시적으로 이용한다. 이는 구조적인 환경에서 더 높은 정확도를 기대할 수 있게 하지만, 특징이 없는 환경에서는 FAST-LIO2가 더 유리할 수 있다.

  - **자료구조 및 성능**: 자료구조는 세 시스템의 핵심적인 차별점이다. FAST-LIO2는 `ikd-Tree` 12, Faster-LIO는 

    `iVox` 15, VOX-LIO는 서펠-해시 복셀 5을 사용한다. Faster-LIO는 

    `iVox`를 통해 `ikd-Tree`보다 빠르다고 주장하는 반면 15, VOX-LIO는 Faster-LIO 대비 정확도에서 월등한 성능 향상을 보였다고 보고한다.5 이는 VOX-LIO의 서펠-복셀 전략이 단순한 속도 경쟁을 넘어, 정합의 '질' 자체를 높이는 데 기여했음을 강력히 시사한다.

### 3.3  vs. FR-LIO (고속 복셀맵 경쟁자)

FR-LIO는 VOX-LIO와 마찬가지로 복셀 기반 맵을 사용하며 공격적인 움직임에 대한 강인성을 목표로 한다.

- **공통점**: 두 시스템 모두 공격적인 움직임에 대한 강인성 및 극도의 계산 효율성을 추구한다.
- **차이점**:
  - **자료구조**: FR-LIO는 고정된 크기의 배열 기반 `RC-Vox`를 사용하여 이론적으로 가장 빠른 검색 속도를 보장한다.6 그러나 맵의 크기가 고정되어 있어 넓은 영역을 탐사할 때 맵 포인트를 순환시켜야 하는 오버헤드가 발생한다.4 반면, VOX-LIO의 동적 해시 맵은 필요에 따라 확장 가능하여 대규모 환경에 더 유연하게 대처할 수 있다.
  - **상태 추정**: FR-LIO는 반복 칼만 스무더(Iterated Error State Kalman Smoother, ESKS)를 사용하여 일정 시간 윈도우 내의 모든 측정값을 바탕으로 현재 상태를 추정한다.6 이는 iEKF보다 더 부드럽고 정확한 궤적을 생성할 수 있다. 두 시스템 모두 공격적인 움직임에 대응하기 위해 정교한 상태 추정기를 채택했다는 공통점을 가진다.

## 4.  성능 평가 및 벤치마크 분석

### 4.1  평가 데이터셋 및 지표

- **데이터셋**:
  - **KITTI Odometry**: 자율주행 시나리오의 표준 벤치마크로, 도심, 시골, 고속도로 등 다양한 환경을 포함한다. 정확한 지상 실측 정보(ground truth)를 제공하여 LIO 알고리즘의 정확도를 평가하는 데 널리 사용된다.5
  - **NCLT (North Campus Long-Term)**: 장기간, 대규모 환경에서의 SLAM 성능 평가에 특화된 데이터셋이다. 계절과 날씨 변화를 포함하고 있어 시스템의 장기적인 강인성을 테스트하는 데 유용하다.5
- **평가 지표**:
  - **ATE (Absolute Trajectory Error)**: 추정된 전체 궤적과 실제 궤적 간의 전역적인 차이를 측정하는 지표다. 시스템의 전반적인 정확도를 나타내며, 보통 RMSE(Root Mean Square Error)로 계산된다.18
  - **RPE (Relative Pose Error)**: 일정 시간 또는 거리 간격 동안의 상대적인 자세 변화 오차를 측정한다. Odometry 시스템 자체의 드리프트(drift) 정도를 평가하는 데 주로 사용된다.19

### 4.2  정량적 성능 비교

벤치마크 결과는 알고리즘의 성능을 객관적으로 보여주지만, 그 해석에는 신중함이 필요하다. 예를 들어, VOX-LIO 논문은 특정 KITTI 시퀀스에서 Faster-LIO 대비 60%가 넘는 경이적인 ATE 감소를 보고했지만 5, 다른 연구에서는 LIO-SAM과 FAST-LIO2 같은 최상위 알고리즘들 간의 ATE 차이가 KITTI 데이터셋에서 "유의미하지 않다"고 결론 내리기도 한다.21 이러한 상반된 결과는 벤치마크 포화(benchmark saturation) 현상을 시사할 수 있다. 즉, 평균적인 성능 지표는 특정 시나리오에서 발휘되는 알고리즘의 고유한 강점을 가릴 수 있다. VOX-LIO의 서펠 기반 접근법이 특정 구조적 환경에서 경쟁 알고리즘의 단순한 복셀 방식보다 월등한 성능을 발휘했기 때문에 큰 폭의 개선이 나타났을 가능성이 있다. 따라서 단일 벤치마크 점수에 의존하기보다는, 특정하고 도전적인 하위 시나리오에서의 성능을 심층 분석하는 것이 알고리즘의 진정한 강점을 이해하는 데 필수적이다.

| 알고리즘       | 데이터셋 | ATE (m) | RPE (%) | 출처 |
| -------------- | -------- | ------- | ------- | ---- |
| **Faster-LIO** | KITTI 05 | 0.748   | -       | 5    |
| **VOX-LIO**    | KITTI 05 | 0.270   | -       | 5    |
| **VOX-LIOM**   | KITTI 05 | 0.120   | -       | 5    |
| **LIO-SAM**    | KITTI 05 | -       | 0.6     | 21   |
| **FAST-LIO2**  | KITTI 05 | -       | 0.7     | 21   |

*표1: KITTI Odometry Benchmark Sequence 05 성능 비교. VOX-LIO 논문은 ATE를, 다른 연구는 RPE를 주로 보고하여 직접 비교에 한계가 있으나, VOX-LIO 계열의 압도적인 ATE 개선을 확인할 수 있다.*

| 알고리즘       | 처리 속도               | 플랫폼          | 출처 |
| -------------- | ----------------------- | --------------- | ---- |
| **VOX-LIO**    | "계산 비용 최소화"      | Intel i7-6700HQ | 5    |
| **Faster-LIO** | >100 Hz (32-line)       | Modern CPU      | 14   |
| **FAST-LIO2**  | ~100 Hz                 | Intel i7        | 12   |
| **FR-LIO**     | > Faster-LIO (parallel) | -               | 7    |

*표2: 주요 LIO 시스템의 연산 성능. VOX-LIO는 정확도 향상에 집중하면서도 효율성을 유지했다고 주장하며, 다른 시스템들은 극도의 속도를 강조한다.*

VOX-LIO 논문에 따르면, `kitti_05` 시퀀스에서 VOX-LIO는 Faster-LIO 대비 평균 ATE를 63.9% 감소시켰으며, 루프 클로저가 추가된 VOX-LIOM은 여기서 55.5%를 추가로 감소시켜 최종적으로 0.12 m 수준의 뛰어난 정확도를 달성했다.5 이는 서펠-복셀 전략과 전역 최적화의 시너지 효과를 극명하게 보여주는 결과다. 연산 성능 측면에서 VOX-LIO는 Intel i7-6700HQ CPU 환경에서 테스트되었으며 5, "계산 비용을 최소화"했다고 주장한다.8 이는 Faster-LIO나 FR-LIO와 같은 속도 중심 시스템과 비교할 때, VOX-LIO가 정확도 향상을 위해 어느 정도의 연산 비용을 감수하면서도 실시간성을 유지하는 균형점을 찾았음을 시사한다.

### 4.3  강인성 및 일반화 성능

- **공격적인 움직임**: VOX-LIO의 가중 융합 방식은 빠른 LiDAR 움직임 하에서 발생하는 모션 왜곡을 효과적으로 보상하도록 설계되었다.5 이는 움직임이 역동적인 4족 보행 로봇 플랫폼에서의 성공적인 테스트를 통해 검증되었다.5 이는 최대 1000 deg/s의 회전 속도를 감당하는 FAST-LIO2나 21.8 rad/s의 각속도를 처리하는 FR-LIO와 같은 다른 시스템들의 강인성 주장과 비교될 수 있다.4
- **장기 운용**: NCLT와 같은 대규모 데이터셋에서 VOX-LIOM의 루프 클로저 기능은 전역 맵의 일관성을 유지하고 드리프트를 억제하는 데 핵심적인 역할을 한다.5 이는 GPS와 같은 절대 위치 센서의 도움 없이도 장기적인 자율 운용을 가능하게 하는 중요한 능력이다.
- **다양한 플랫폼 적용**: VOX-LIO는 전통적인 바퀴형 로봇뿐만 아니라, 움직임이 더 불규칙한 4족 보행 로봇 플랫폼에서도 성공적으로 테스트되어 알고리즘의 높은 일반화 가능성을 입증했다.5 이는 시스템이 특정 플랫폼의 모션 모델에 과도하게 의존하지 않는 견고한 아키텍처를 가졌음을 보여준다.

## 5.  결론 및 미래 전망

### 5.1  VOX-LIO의 종합 평가

VOX-LIO는 (1) 서펠 특징을 융합한 적응형 해시 복셀 맵 관리, (2) 매니폴드 상에서의 강인한 가중 융합, 그리고 (3) 루프 클로저를 통한 전역 최적화라는 세 가지 핵심 기여를 통해 LIO 시스템의 고질적인 문제인 정확성, 강인성, 효율성 간의 균형을 성공적으로 달성한 우수한 시스템으로 평가된다.

기술적으로 VOX-LIO는 기존의 속도 중심의 자료구조 경쟁에서 한 걸음 나아가, '정보가 풍부한 스마트한 복셀'과 정교한 융합 기법을 결합하여 성능을 한 단계 끌어올렸다는 점에서 중요한 의의를 가진다. 그러나 한계점도 존재한다. 논문에서는 동적 객체(dynamic objects) 제거에 대한 명시적인 언급이 부족하다. LIO-Livox나 Dynamic-LIO와 같이 동적 환경에 특화된 시스템들과 비교할 때 22, 실제 복잡한 도심 환경에서는 추가적인 모듈 없이는 성능 저하를 겪을 수 있다.

### 5.2  LIO 기술의 미래 발전 방향

LIO 기술의 '강인성'에 대한 정의는 계속해서 확장되고 있다. 과거에는 특징이 부족한 환경을 극복하는 것이 주요 과제였지만, 이제는 공격적인 움직임과 수많은 동적 객체가 혼재하는 실제 환경에 대한 대응 능력이 중요해졌다.6 이러한 복합적인 문제들을 해결하기 위해서는 단일 알고리즘이 아닌, 모듈화된 SLAM 파이프라인이 필요하다. 예를 들어, 

`동적 객체 필터 -> LIO 코어 -> 전역 최적화기`와 같은 구조다. VOX-LIO는 이 구조에서 매우 강력한 'LIO 코어' 역할을 할 수 있지만, 동적 객체 필터의 부재는 이러한 모듈화 추세를 역설적으로 보여준다. 미래의 SLAM 개발은 각 모듈의 성능을 고도화하고 이들을 유기적으로 통합하는 방향으로 나아갈 것이다.

- **다중 센서 융합의 심화**: LiDAR, IMU에 더해 카메라(Visual), 레이더(Radar), GPS, 휠 인코더 등 성질이 다른 이종(heterogeneous) 센서 정보를 긴밀하게 통합하는 연구가 더욱 활발해질 것이다.25 이는 상호 보완을 통해 악천후나 GPS 음영 지역과 같은 극한 환경에서의 강인성을 극대화하는 열쇠가 될 것이다.
- **학습 기반 방법론의 통합**: 특징 추출, 장소 인식(place recognition), 루프 탐지 등 기존에 수동으로 설계되던 모듈에 딥러닝을 적용하여 그 한계를 극복하려는 시도가 증가할 것이다.28 예를 들어, 포인트 클라우드를 BEV(Bird's-Eye-View) 이미지로 변환하고 CNN을 통해 특징을 추출하여 루프 클로저의 정확도를 높이는 연구가 유망한 방향으로 제시되고 있다.29
- **시맨틱 정보의 활용**: 포인트 클라우드에 '자동차', '건물', '도로'와 같은 의미론적(semantic) 레이블을 부여하고, 이를 상태 추정과 맵핑에 적극적으로 활용하는 시맨틱 SLAM이 부상할 것이다.31 이는 동적 객체를 보다 근본적으로 식별하여 제거하고, 인간이 이해하기 쉬운 지도를 생성하여 맵의 활용도를 높이는 데 크게 기여할 것이다.
- **장기 자율성을 위한 맵 관리**: 시간이 지남에 따라 변화하는 실제 환경(예: 공사 구간, 계절 변화)에 대응하여 맵을 동적으로 업데이트하고 유지보수하는 라이프롱(lifelong) SLAM 기술이 장기 자율성의 핵심 과제가 될 것이다.

#### **참고 자료**

1. LiDAR Odometry Survey: Recent Advancements and Remaining Challenges - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/pdf/2312.17487
2. LiDAR odometry survey: recent advancements and remaining challenges, 8월 3, 2025에 액세스, https://snu.elsevierpure.com/en/publications/lidar-odometry-survey-recent-advancements-and-remaining-challenge
3. LiDAR odometry survey: recent advancements and remaining challenges - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/378106065_LiDAR_odometry_survey_recent_advancements_and_remaining_challenges
4. Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/pdf/2302.04031
5. VOX-LIO: An Effective and Robust LiDAR-Inertial Odometry System Based on Surfel Voxels, 8월 3, 2025에 액세스, https://www.mdpi.com/2072-4292/17/13/2214
6. [2302.04031] FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/abs/2302.04031
7. FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/368361497_FR-LIO_Fast_and_Robust_Lidar-Inertial_Odometry_by_Tightly-Coupled_Iterated_Kalman_Smoother_and_Robocentric_Voxels
8. VOX-LIO: An Effective and Robust LiDAR-Inertial Odometry System Based on Surfel Voxels, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/393224081_VOX-LIO_An_Effective_and_Robust_LiDAR-Inertial_Odometry_System_Based_on_Surfel_Voxels
9. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and ..., 8월 3, 2025에 액세스, http://arxiv.org/pdf/2007.00258
10. [2010.08196] FAST-LIO: A Fast, Robust LiDAR-inertial Odometry Package by Tightly-Coupled Iterated Kalman Filter - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/abs/2010.08196
11. RTLIO: Real-Time LiDAR-Inertial Odometry and Mapping for UAVs - PMC, 8월 3, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC8226800/
12. FAST-LIO2: Fast Direct LiDAR-inertial Odometry, 8월 3, 2025에 액세스, https://arxiv.org/abs/2107.06829
13. LiPO: LiDAR Inertial Odometry for ICP Comparison - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2410.08097v1
14. Faster-LIO: Lightweight Tightly Coupled Lidar-inertial Odometry using Parallel Sparse Incremental Voxels - GitHub, 8월 3, 2025에 액세스, https://github.com/gaoxiang12/faster-lio
15. (PDF) Faster-LIO: Lightweight Tightly Coupled Lidar-Inertial Odometry Using Parallel Sparse Incremental Voxels - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/359662313_Faster-LIO_Lightweight_Tightly_Coupled_Lidar-Inertial_Odometry_Using_Parallel_Sparse_Incremental_Voxels
16. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/abs/2007.00258
17. The KITTI Vision Benchmark Suite - Andreas Geiger, 8월 3, 2025에 액세스, https://www.cvlibs.net/datasets/kitti/
18. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - PMC, 8월 3, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11644182/
19. Performance Assessment of Lidar Odometry Frameworks: A Case Study at the Australian Botanic Garden Mount Annan - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2411.16931v2
20. Visualization of trajectory comparison between A-LOAM, FAST-LIO, and... - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/figure/sualization-of-trajectory-comparison-between-A-LOAM-FAST-LIO-and-our-method-on-KITTI_fig5_378109927
21. Evaluation of 3D LiDAR SLAM algorithms based on the KITTI dataset - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/370151983_Evaluation_of_3D_LiDAR_SLAM_algorithms_based_on_the_KITTI_dataset
22. Livox-SDK/LIO-Livox: A Robust LiDAR-Inertial Odometry for Livox LiDAR - GitHub, 8월 3, 2025에 액세스, https://github.com/Livox-SDK/LIO-Livox
23. LiDAR-Inertial Odometry in Dynamic Driving Scenarios using Label Consistency Detection, 8월 3, 2025에 액세스, https://arxiv.org/html/2407.03590v2
24. [2206.09463] RF-LIO: Removal-First Tightly-coupled Lidar Inertial Odometry in High Dynamic Environments - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/abs/2206.09463
25. LIWO: Lidar-Inertial-Wheel Odometry - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2302.14298v2
26. FAST-LIVO2: Fast, Direct LiDAR-Inertial-Visual Odometry - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2408.14035v2
27. R-LVIO: Resilient LiDAR-Visual-Inertial Odometry for UAVs in GNSS-denied Environment, 8월 3, 2025에 액세스, https://www.mdpi.com/2504-446X/8/9/487
28. [2010.09343] SelfVoxeLO: Self-supervised LiDAR Odometry with Voxel-based Deep Neural Networks - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/abs/2010.09343
29. BEV-LIO(LC): BEV Image Assisted LiDAR-Inertial Odometry with Loop Closure - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2502.19242v2
30. [Literature Review] BEV-LIO(LC): BEV Image Assisted LiDAR-Inertial Odometry with Loop Closure - Moonlight, 8월 3, 2025에 액세스, https://www.themoonlight.io/en/review/bev-liolc-bev-image-assisted-lidar-inertial-odometry-with-loop-closure
31. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - MDPI, 8월 3, 2025에 액세스, https://www.mdpi.com/1424-8220/24/23/7546


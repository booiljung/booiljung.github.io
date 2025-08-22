---
layout: page
title: FR-LIO
permalink: /slam/FAST-LIO/FR-LIO
---


자율 이동 로봇이 미지의 환경을 항해하기 위한 핵심 기술은 동시적 위치추정 및 지도작성(Simultaneous Localization and Mapping, SLAM)이다.1 이 중에서도 LiDAR(Light Detection and Ranging) 기반 SLAM은 빠르고 정확하며 밀도 높은 3차원 환경 복원을 제공하는 능력 덕분에 광범위하게 활용되고 있다.2 본 보고서는 SLAM의 핵심 하위 문제이자 시스템 전체 성능에 지대한 영향을 미치는 주행계(Odometry) 기술, 특히 LiDAR-관성 주행계(LIO)에 초점을 맞춘다.1

현대 LIO 시스템의 성능은 정확성(Accuracy), 강건성(Robustness), 그리고 효율성(Efficiency)이라는 세 가지 상호 연관된 축으로 평가된다.1 정확성은 포인트 클라우드 등록(registration)의 정밀도를 높이고 센서의 움직임으로 인해 발생하는 모션 왜곡(motion distortion) 보상 오차를 최소화함으로써 달성된다.2 강건성은 시스템이 다양한 코너 케이스(corner cases)에 대처하는 능력으로, 특히 구조적 특징이 부족한 퇴화 환경(scene degradation)과 격렬한 움직임(aggressive motion) 상황에서의 안정적인 추적 능력이 주요 난제로 꼽힌다.1 마지막으로 효율성은 실시간 처리를 위한 계산 자원의 효율적 사용 문제로, 특히 포인트 클라우드 맵을 관리하는 공간 데이터 구조(spatial data structure)의 선택이 시스템의 전반적인 속도에 가장 큰 영향을 미친다.1

이 세 가지 과제 중에서도 '격렬한 움직임'은 현대 LIO 기술의 가장 중요한 프론티어 중 하나이다. 대부분의 기존 최첨단(State-of-the-Art, SOTA) LIO 시스템들은 저속의 부드러운 움직임에서는 뛰어난 성능을 보이지만 6, 급격한 회전이나 가감속이 포함된 격렬한 움직임 상황에서는 심각한 성능 저하를 겪는다. 근본적인 원인은 관성 측정 장치(IMU)의 제한된 샘플링 주파수에 있다. IMU 측정 간격 사이에서 발생하는 고주파의 비선형적인 움직임(non-linear dynamics)을 포착하지 못하는 것이다.7 이는 단순히 로봇이 빠르게 움직이는 속도의 문제가 아니라, IMU 측정치 사이의 움직임을 선형적이라고 가정하는 기존 모션 왜곡 보상 모델의 근본적인 실패를 의미한다. 이 모델링 오류는 부정확한 왜곡 보상과 잘못된 초기 자세 추정으로 이어져, 결국 시스템의 추적 실패를 야기한다.1 실제로 최대 각속도가 21.8 rad/s에 달하는 움직임은 기존 SOTA 방법들이 안정적으로 처리하지 못하는 미지의 영역으로 남아 있었다.5

본 보고서에서 심층 분석할 'FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels' 논문은 바로 이 격렬한 움직임에 대한 강건성 문제를 정면으로 다루면서도, 최고 수준의 효율성과 경쟁력 있는 정확성을 동시에 달성하고자 하는 새로운 LIO 프레임워크를 제안한다.2 FR-LIO의 설계 철학은 세 가지 핵심 기여가 서로 유기적으로 연결되어 있다는 점에서 주목할 만하다. 첫째, 격렬한 움직임으로 인한 IMU 적분 오차 문제를 해결하기 위해 '적응형 서브프레임 분할' 기법을 도입한다. 둘째, 이로 인해 발생하는 새로운 문제, 즉 데이터 희소성으로 인한 등록 실패(degradation)를 막기 위해 '반복 칼만 스무더(ESKS)'를 채택한다. 셋째, 이 모든 과정을 실시간으로 처리할 수 있는 계산적 여유를 확보하기 위해 극도로 효율적인 '로보센트릭 복셀 맵(RC-Vox)'을 설계하였다. 이처럼 FR-LIO는 하나의 문제를 해결하기 위한 솔루션이 야기하는 부작용을 다음 솔루션이 보완하는 정교한 인과적 연쇄 구조를 통해 시스템 전체의 완성도를 높였다.


FR-LIO 시스템은 격렬한 움직임에 대응하기 위한 세 가지 핵심 구성요소, 즉 적응형 서브프레임 분할, 반복 오류 상태 칼만 스무더(ESKS), 그리고 로보센트릭 복셀 맵(RC-Vox)의 유기적인 결합으로 이루어져 있다.


FR-LIO의 첫 번째 혁신은 격렬한 움직임 동안 누적되는 IMU 적분 오차를 근본적으로 줄이는 데 있다.2 전통적인 LIO 시스템은 LiDAR 센서가 360도 회전하여 얻는 하나의 완전한 스캔(full scan)을 단일 처리 단위로 사용한다. 하지만 움직임이 격렬할 경우, 스캔 시작 시점과 종료 시점 사이의 시간 동안 큰 자세 변화가 발생하여 IMU 적분 오차가 커지고, 이는 포인트 클라우드의 왜곡 보상 실패로 이어진다.

이 문제를 해결하기 위해 FR-LIO는 LiDAR의 연속 스캐닝(continuous scanning) 특성을 적극적으로 활용한다. 하나의 전체 스캔을 처리하는 대신, 이를 여러 개의 작은 부분 스캔, 즉 '서브프레임(sub-frame)'으로 분할하여 처리 주기를 높인다.1 핵심은 이 분할 과정이 고정적이지 않고 동적이라는 점이다. 시스템은 IMU에서 측정된 가속도와 각속도의 표준편차를 실시간으로 분석하여 현재 움직임의 강도를 판단하고, 움직임이 격렬할수록 더 많은 수의 서브프레임으로 스캔을 나눈다.5 이는 평상시에는 불필요한 계산을 줄여 효율성을 유지하다가, 격렬한 움직임이 감지될 때만 처리 정밀도를 높이는 지능적인 전략이다.

서브프레임의 수 $n_{curr}$는 다음의 수학적 공식에 따라 결정된다 5:
$$
n_{curr} = \text{ceil} \left( n_{max} \cdot \text{max} \left( \frac{\sigma_{acc_{curr}}}{\sigma_{acc_{max}}}, \frac{\sigma_{gyr_{curr}}}{\sigma_{gyr_{max}}} \right) \right)
$$
여기서 $n_{max}$는 시스템이 분할할 수 있는 최대 서브프레임 수, $\sigma_{acc_{curr}}$와 $\sigma_{gyr_{curr}}$는 현재 스캔에 해당하는 가속도 및 각속도의 표준편차, 그리고 $\sigma_{acc_{max}}$와 $\sigma_{gyr_{max}}$는 사전에 통계적으로 결정된 최대 표준편차 값이다. 이 기법을 통해 각 포인트의 왜곡 보상을 위한 IMU 적분 구간이 짧아져 비선형 운동으로 인한 오차를 크게 줄일 수 있으며, 결과적으로 시스템의 유효 업데이트 주기를 높여 추적 정확도를 향상시킨다.6


적응형 서브프레임 분할은 왜곡 보상 문제를 해결하지만, 새로운 문제를 야기한다. 각 서브프레임이 담고 있는 포인트 수가 적어지기 때문에, 복도나 벽과 같이 기하학적 특징이 부족한 환경에서는 충분한 제약조건을 얻지 못해 위치 추정이 실패할 위험, 즉 '퇴화(degradation)' 문제가 발생할 수 있다.1

FR-LIO는 이 문제를 해결하기 위해 전통적인 순방향(forward-pass) 칼만 필터(EKF, ESKF)가 아닌, '스무더(smoother)'를 채택했다.2 칼만 필터는 과거와 현재의 측정값만을 사용하여 현재 상태를 추정하는 반면 9, 스무더는 특정 시간 구간 내의 모든 측정값(과거, 현재, 미래)을 활용하여 과거 상태를 다시 한번 보정하는 양방향(forward and backward pass) 방식을 사용한다.11 이는 더 높은 정확도와 부드러운 궤적을 제공하지만 계산 비용이 높다는 단점이 있다. FR-LIO는 반복 오류 상태 칼만 스무더(Iterated Error State Kalman Smoother, ESKS) 프레임워크를 통해 이 문제를 정면으로 돌파한다. 즉, 현재 서브프레임의 정보가 부족하더라도, 시간적으로 뒤따르는 미래 서브프레임들의 풍부한 정보를 역으로 활용하여 현재의 불확실성을 해결하고 궤적의 평활도(smoothness)를 보장하는 것이다.1

시스템의 상태 벡터는 전역 좌표계 `{G}`에 대한 IMU 본체 좌표계 `{b}`의 위치 ${}^{G}p_{b}$, 속도 ${}^{G}v_{b}$, 자세 ${}^{G}q_{b}$와 IMU의 가속도계 바이어스 $b_a$, 자이로스코프 바이어스 $b_g$, 그리고 중력 벡터 ${}^{G}g$로 구성된다. 상태 추정기는 이들의 공칭 상태(nominal state)와 오차를 나타내는 오류 상태(error state) $\delta x$를 추정한다.5
$$
\delta x \triangleq \begin{pmatrix} {}^{G}\delta p_{b}^{T} & {}^{G}\delta v_{b}^{T} & {}^{G}\delta \theta_{b}^{T} & \delta b_{a}^{T} & \delta b_{g}^{T} & {}^{G}\delta g^{T} \end{pmatrix}^{T} \in \mathbb{R}^{18}
$$
FR-LIO의 ESKS 상태 추정 파이프라인은 다음과 같은 3단계로 구성된다:

1. **순방향 전파 (Forward Propagation)**: 새로운 IMU 측정값이 들어올 때마다 공칭 상태와 오류 상태 공분산(covariance)을 다음 시간 단계로 예측한다.5
   $$
   \delta x_{k+1} = (I + F_k \Delta t) \delta x_k + (C_k \Delta t) w
   $$

   $$
   P_{k+1} = (I + F_k \Delta t) P_k (I + F_k \Delta t)^T + (C_k \Delta t) Q_k (C_k \Delta t)^T
   $$

   **반복적 업데이트 (Iterative Update)**: 새로운 LiDAR 서브프레임이 도착하면, FAST-LIO2와 유사한 직접법(direct method)을 사용하여 포인트-평면 간의 기하학적 잔차(residual)를 계산한다.5
   $$
   h_j \left( x_{\kappa_k} \right) = {}^{G}n_j^T \left( {}^{G}q_{\kappa_k}^b {}^{b}q_l l_{p_j} + {}^{b}p_l \right) + {}^{G}p_{\kappa_k}^b - {}^{G}c_j
   $$
   이 잔차를 측정값으로 사용하여 칼만 게인을 계산하고, 상태를 반복적으로 업데이트하며 최적의 해를 찾아간다.

2. **역방향 스무딩 (Backward Smooth)**: 하나의 전체 스캔에 해당하는 모든 서브프레임의 순방향 처리가 완료되면, 시스템은 Rauch-Tung-Striebel(RTS) 스무더 알고리즘을 적용하여 저장된 과거 상태들을 역방향으로 보정한다.5 이는 미래의 더 정확한 측정값을 바탕으로 과거의 불확실했던 추정치를 개선하는 과정으로, 퇴화된 서브프레임의 포즈를 효과적으로 보정한다.
   $$
   x_k^s = x_k + G_k (x_{k+1}^s - x_{k+1}^{-})
   $$
   여기서 $x_k^s$는 스무딩된 상태, $G_k$는 스무더 게인, $x_{k+1}^{-}$는 순방향 필터링으로 예측된 상태를 의미한다.

이러한 ESKS 프레임워크는 필터의 실시간성과 배치 최적화의 정확성 사이에서 효과적인 절충안을 제시한다. 전체 이력을 최적화하는 팩터 그래프(Factor Graph Optimization) 방식보다 가벼우면서도, 과거 상태를 버리는 단순 필터 방식보다 훨씬 정확하고 부드러운 궤적을 생성할 수 있다. 이는 지역적 정확도와 궤적 평활도가 중요한 고성능 주행계에 최적화된 설계 선택이라 할 수 있다.


앞서 설명한 적응형 분할과 ESKS는 계산 비용이 높은 작업이다. 이를 실시간으로 수행하기 위해서는 맵 관리, 즉 포인트 클라우드의 추가, 삭제, 검색 연산이 극도로 빨라야 한다. LIO 시스템에서 맵 관리는 가장 시간이 많이 소요되는 부분 중 하나이기 때문이다.1 기존의 동적 k-d 트리(`ikd-Tree`)는 증분 업데이트는 가능하지만 트리가 불균형해지면 부분 재구성이 필요하고 1, 해시 테이블 기반의 `iVox`는 해시 충돌과 복잡한 함수 계산이라는 오버헤드가 존재한다.5 이론적으로 가장 빠른 접근 속도를 가진 데이터 구조는 배열(array)이지만, 대규모 포인트 클라우드를 저장하기에는 막대한 메모리가 필요하여 LIO 시스템에는 부적합한 것으로 여겨져 왔다.2

FR-LIO는 '로보센트릭(robocentric)' 개념과 '고정 크기 2계층 3D 배열'이라는 두 가지 독창적인 아이디어를 결합하여 이 문제를 해결하고, 배열 구조의 장점을 극대화했다.1 이는 기존의 복잡한 자료구조를 더 빠르게 만드는 접근에서 벗어나, 가장 빠른 단순 자료구조인 배열을 LIO에 사용 가능하도록 만드는 발상의 전환을 보여준다.

- **로보센트릭 (Robocentric)**: 맵의 원점을 고정된 전역 좌표계에 두는 대신, 항상 로봇의 현재 위치에 고정시킨다. 로봇이 움직이면 맵도 함께 따라 움직이며, 로봇의 LiDAR 유효 범위를 벗어나는 오래된 맵 데이터는 자연스럽게 폐기된다. 이를 통해 맵의 크기가 무한정 커지는 것을 막고, 항상 일정한 크기의 로컬 맵만 유지하여 메모리 사용량을 획기적으로 줄인다.2
- **2계층 3D 배열 (Two-layer 3D Array)**: 메모리 효율성과 높은 해상도를 동시에 달성하기 위해 2계층 구조를 도입했다. 상위 계층 배열(TLA)은 미터 단위의 낮은 해상도를 가진 그리드로 구성되고, 하위 계층 배열(BLA)은 TLA의 각 그리드 내부에 포인트가 존재할 경우에만 동적으로 생성되는 센티미터 단위의 높은 해상도 복셀로 구성된다.5
- **모듈로 연산 (Modulo Operation)**: 로봇이 이동하여 로컬 맵이 고정된 크기의 TLA 배열 경계를 벗어나려 할 때, 모듈로 연산을 통해 해당 영역을 배열의 반대편으로 즉시 재매핑(remapping)한다. 이는 배열의 크기를 물리적으로 변경하지 않으면서도 무한한 공간을 표현하는 효과를 내는 핵심적인 기법이다.2

이러한 RC-Vox 구조 덕분에 FR-LIO는 포인트의 추가, 삭제, 그리고 최근접 이웃(k-NN) 검색을 상수 시간 복잡도에 가깝게 수행할 수 있다. 이는 ESKS와 같은 복잡한 상태 추정 알고리즘을 실시간으로 구동할 수 있는 계산적 여유를 제공하는 결정적인 요소로 작용한다.


FR-LIO의 기술적 위상을 명확히 이해하기 위해서는 당대의 주요 LIO 시스템들과의 비교 분석이 필수적이다. LIO 시스템은 크게 필터 기반 접근법과 최적화 기반 접근법으로 나눌 수 있으며, FR-LIO는 이 두 패러다임의 특징을 모두 포함하고 있다.


FAST-LIO 계열은 오류 상태 칼만 필터(ESKF)를 기반으로 하는, 빠르고 강인한 LIO 시스템의 대명사이다.

- **FAST-LIO2**: 이 시스템은 두 가지 핵심 혁신으로 LIO 분야에 큰 족적을 남겼다.13 첫째는 **직접 점 등록(Direct Point Registration)**으로, 전통적인 특징점(edge, plane) 추출 단계를 생략하고 원본 포인트(raw points)를 맵에 직접 등록하는 방식이다.17 이는 구조가 적은 환경에서도 미세한 기하학적 정보를 활용하여 정확도와 강건성을 높이는 중요한 역할을 한다. 둘째는 

  **ikd-Tree**라는 새로운 데이터 구조로, 증분 업데이트(삽입, 삭제)와 동적 재균형이 가능하여 기존 정적 k-d 트리에 비해 맵 업데이트 시간을 획기적으로 단축시켰다.1

- **Faster-LIO**: FAST-LIO2를 기반으로, 맵 데이터 구조를 `ikd-Tree`에서 **iVox(Incremental Sparse Voxels)**로 교체하여 처리 속도를 1.5~2배가량 향상시킨 시스템이다.22

  `iVox`는 해시 테이블과 LRU(Least Recently Used) 캐시를 사용하여 복셀을 관리하며, 병렬 처리 시 `ikd-Tree`의 성능을 능가한다.5 또한, 엄격한 k-NN 검색 대신 근사 k-NN 검색을 수행하여 속도를 극대화하는 전략을 취한다.23

- **FR-LIO와의 비교**: FR-LIO는 FAST-LIO 계열과 동일하게 ESKF 프레임워크와 직접 점 등록 방식을 공유하지만, 두 가지 결정적인 차이가 있다. 첫째, 상태 추정 방식에서 FR-LIO는 순방향 필터링에 더해 역방향 스무딩 과정이 포함된 ESKS를 사용하여 궤적의 정확도와 평활도를 더욱 높였다. 둘째, 데이터 구조에서 FR-LIO의 `RC-Vox`는 `ikd-Tree`나 `iVox`보다 근본적으로 더 빠른 접근 속도를 제공하여, 단일 스레드 환경에서도 병렬화된 SOTA 시스템을 능가하는 압도적인 효율성을 보인다.1


LIO-SAM(Lidar Inertial Odometry via Smoothing and Mapping)은 필터가 아닌 팩터 그래프 최적화(Factor Graph Optimization, FGO)를 기반으로 하는 대표적인 LIO 시스템이다.24

- **핵심 아키텍처**: LIO-SAM은 로봇의 포즈와 랜드마크를 노드(node)로, 그리고 IMU 사전적분, LiDAR 주행계, GPS, 루프 클로저 등의 측정값을 팩터(factor)로 표현하는 그래프 모델을 구축한다.24 새로운 키프레임이 추가될 때마다, 이 그래프 전체(또는 슬라이딩 윈도우 내)의 모든 제약조건을 만족시키는 최적의 해를 비선형 최적화 기법으로 찾는다.
- **FR-LIO와의 비교**: FR-LIO와 LIO-SAM의 비교는 단순히 수학적 기법의 차이를 넘어, 시스템 설계 목표의 근본적인 차이를 보여준다. FR-LIO는 최소한의 지연 시간으로 가장 정확한 지역적(local) 포즈를 추정하는 **프론트엔드 주행계**로서의 성능에 집중한다. 반면, LIO-SAM은 루프 클로저나 GPS와 같은 전역 제약조건을 통합하여 장시간 동안 누적되는 오차를 보정하고 전역적으로 일관된(globally consistent) 맵을 구축하는 완전한 **SLAM 시스템**을 목표로 한다.24 FR-LIO의 ESKS는 과거 상태를 점진적으로 요약(marginalize)하며 진행하는 재귀적 방식인 반면, LIO-SAM의 FGO는 과거 상태들을 유지하며 반복적으로 재선형화(re-linearization)하는 배치(batch) 방식이다. 이론적으로 FGO가 더 정확하지만, 계산 복잡도가 높을 수 있다.29 이는 이상적인 차세대 SLAM 시스템이 FR-LIO와 같은 빠른 프론트엔드와 LIO-SAM과 같은 강력한 백엔드를 결합한 하이브리드 형태가 될 것임을 시사한다.4


LIO 프론트엔드의 성능을 좌우하는 핵심은 데이터 구조이다. `ikd-Tree`, `iVox`, `RC-Vox`는 각각 다른 철학을 바탕으로 설계되었으며, 그 장단점은 다음과 같다.

| **특징 (Feature)**    | **ikd-Tree (FAST-LIO2)**                | **iVox (Faster-LIO)**                              | **RC-Vox (FR-LIO)**                                          |
| --------------------- | --------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| **기본 원리**         | 동적 k-차원 트리 (Dynamic k-d Tree)     | 증분 희소 복셀 (Incremental Sparse Voxel)          | 로보센트릭 2계층 배열 (Robocentric 2-Layer Array)            |
| **자료 구조**         | 트리 (Tree)                             | 해시 테이블 + LRU 캐시 (Hash Table + LRU Cache)    | 고정 크기 3D 배열 (Fixed-size 3D Array)                      |
| **업데이트 메커니즘** | 증분 삽입/삭제, 부분 재균형             | 해시 기반 삽입/삭제, LRU 기반 캐싱                 | 모듈로 연산을 통한 재매핑                                    |
| **k-NN 검색**         | 엄격한(Strict) k-NN 검색                | 근사(Approximate) k-NN 검색                        | 근사(Approximate) k-NN 검색                                  |
| **장점**              | 엄격한 검색 가능, 증분 업데이트 지원 17 | 극도로 빠른 업데이트/검색, 병렬화 용이 15          | 가장 빠른 접근 속도, 구조적 오버헤드 없음 2                  |
| **단점**              | 불균형 시 재구성 비용 발생 1            | 해시 충돌, 근사 검색으로 인한 정확도 손실 가능성 5 | 대규모 맵에 대한 메모리 비효율성 (로보센트릭으로 완화), 엄격한 검색 불가 5 |

이 표는 LIO 데이터 구조의 발전 방향을 명확하게 보여준다. 구조화된 정확성을 추구한 `ikd-Tree`에서, 병렬화와 속도에 집중한 해시 기반의 `iVox`로, 그리고 다시 근본적인 접근 속도를 위해 배열 구조를 채택하고 그 약점을 독창적으로 보완한 `RC-Vox`로의 진화는 프론트엔드 주행계가 '엄격한 정확성'보다는 '압도적인 속도'를 우선시하는 방향으로 발전하고 있음을 나타낸다.


FR-LIO의 성능은 4개의 공개 데이터셋(NTU VIRAL, M2DGR, UrbanNav, Newer College)과 자체 제작한 데이터셋을 포함한 총 27개의 시퀀스에서 광범위하게 검증되었다.5 평가는 절대 궤적 오차(ATE)와 상대 궤적 오차(RTE)를 지표로 사용하였다.5


- **정확도**: 일반적인 주행 환경에서 FR-LIO는 FAST-LIO2, Faster-LIO 등 다른 SOTA 시스템들과 경쟁력 있는(competitive) 정확도를 달성했다.5 특히 M2DGR 데이터셋의 

  `street 07` 시퀀스와 같이 회전이 많은 구간에서는 적응형 포인트 분할과 ESKS 스무딩 덕분에 오히려 더 높은 정확도를 보이기도 했다.5

- **강건성 (격렬한 움직임)**: 이 항목에서 FR-LIO의 진정한 가치가 드러난다. 최대 각속도가 21.8 rad/s에 달하는 자체 제작 데이터셋에서 실험한 결과, 비교 대상인 FAST-LIO2는 움직임을 감당하지 못하고 추적에 실패했지만, FR-LIO는 안정적인 추적을 유지하며 높은 품질의 맵을 성공적으로 재구성했다.2 이는 적응형 서브프레임 분할과 역방향 스무딩이 이론뿐만 아니라 실제 극한 상황에서도 효과적으로 작동함을 입증하는 강력한 증거이다. 정성적으로도, FR-LIO가 재구성한 포인트 클라우드는 FAST-LIO2의 결과물보다 훨씬 더 선명하고 뚜렷한 구조를 보여주었다.2

- **효율성**: M2DGR 데이터셋을 이용한 속도 테스트에서 FR-LIO는 가장 놀라운 결과를 보여주었다. FR-LIO의 단일 스레드 버전이 현재 가장 빠른 LIO 시스템으로 알려진 Faster-LIO의 병렬 가속 버전보다도 빠른 처리 속도를 기록한 것이다.2 이는 RC-Vox 데이터 구조가 기존의 트리나 해시 테이블 기반 구조에 비해 근본적으로 더 높은 효율성을 가짐을 명백히 증명한다.


- **핵심 기여 (Contributions)**: FR-LIO의 핵심적인 학술적 기여는 다음과 같이 요약할 수 있다.1
  1. 격렬한 움직임에 강건한 LIO 시스템을 구현하기 위해, 적응형 라이다 포인트 분할 기법과 이를 보완하는 반복 ESKS 기반 상태 추정 프레임워크를 최초로 제안했다.
  2. LIO 시스템의 효율성을 극대화하기 위해, 로보센트릭 개념과 2계층 배열 구조를 결합한 새로운 맵 구조인 `RC-Vox`를 개발했다.
  3. 광범위하고 도전적인 실험을 통해 제안된 시스템의 강건성과 효율성이 기존 SOTA 시스템을 능가함을 입증했다.
- **내재적 한계 및 향후 연구 방향**: 논문에서 명시적으로 한계점을 언급하지는 않았지만 5, 시스템 설계와 실험 결과를 바탕으로 다음과 같은 한계점과 향후 연구 방향을 추론할 수 있다.
  1. **전역 맵핑의 부재**: RC-Vox는 본질적으로 로보센트릭 '로컬 맵'이다. 따라서 장거리, 대규모 환경에서 전역적인 일관성을 유지하거나 루프 클로저를 수행하는 기능이 없다. 이는 순수 주행계로서의 한계이며, 완전한 SLAM 시스템으로 확장하기 위해서는 FR-LIO를 프론트엔드로 사용하고, 추정된 포즈와 로컬 맵을 전역 맵에 통합하는 별도의 백엔드 모듈(예: 팩터 그래프 최적화)을 결합하는 연구가 필요하다.16
  2. **구조적 퇴화 환경에서의 성능**: ESKS는 '움직임'으로 인한 제약조건 부족 문제를 효과적으로 완화하지만, 긴 터널이나 넓은 평원처럼 '환경' 자체가 기하학적 특징이 없는 LiDAR-degraded 환경에 대한 근본적인 해결책은 아니다.1 이러한 환경에서는 FR-LIO의 강건성을 더욱 높이기 위해 비전(vision) 센서 등 다른 종류의 센서와 융합하는 연구가 유망하다.
  3. **파라미터 민감도**: 적응형 분할의 최대 개수 $n_{max}$나 RC-Vox의 복셀 크기 등은 시스템 성능에 중요한 영향을 미칠 수 있는 하이퍼파라미터이다. 이러한 파라미터들의 민감도에 대한 심층적인 분석이 부족하며, 실제 다양한 환경에 적용하기 위해서는 이에 대한 튜닝 과정이 필요할 수 있다.22

FR-LIO의 강건성은 단순히 궤적을 부드럽게 만드는 스무딩 효과에서 비롯된 것이 아니다. 정보 이론적 관점에서 보면, 이는 시스템의 '일관성(consistency)'을 유지하는 메커니즘과 깊이 관련되어 있다. 순방향 필터는 특징이 부족한 상황에서 잘못된 추정치에 대해 과도하게 확신하는 경향이 있는데, 이는 EKF-SLAM의 관측 가능성(observability) 문제와 연결된다.38 ESKS의 역방향 스무딩 과정은 미래의 정보를 이용해 과거 상태의 불확실성을 재평가함으로써, 필터가 잘못된 해로 성급하게 수렴하는 것을 막고 추정치의 불확실성을 보다 정확하게 표현하도록 강제한다. 이는 시스템의 일관성을 높여 결과적으로 강건성 향상으로 이어진다.

또한, FR-LIO의 가장 큰 실용적 의의는 단일 스레드에서의 압도적인 효율성에 있다. 이는 고성능 LIO 기술을 고가의 대형 로봇이나 자율주행차를 넘어, 드론, 모바일 로봇, AR/VR 헤드셋과 같은 저전력 엣지 디바이스(edge device)에 탑재할 수 있는 길을 열었다는 것을 의미한다.40 이는 고성능 자율 항법 기술의 대중화를 앞당기는 중요한 전환점이 될 수 있다.


FR-LIO는 격렬한 움직임에 대한 강건성과 전례 없는 계산 효율성을 동시에 달성함으로써, LIO 기술의 새로운 지평을 연 최첨단 프레임워크로 평가할 수 있다. 이는 필터 기반 접근법의 속도와 스무딩/최적화 기반 접근법의 정확도라는 두 마리 토끼를 잡기 위한 영리한 하이브리드 설계의 성공적인 사례이다.

FR-LIO가 제시한 핵심 원칙들은 LIO 및 로보틱스 분야의 여러 미래 연구 동향과 깊은 관련을 맺고 있다.

- **동적 초기화 (Dynamic Initialization)**: 기존 LIO 시스템은 대부분 정지 상태에서 초기화해야 한다는 제약을 갖는다.6 하지만 고속도로 주행 중 재시작이나 긴급 구조 로봇 투입과 같이, 움직이는 상태에서 즉시 시스템을 가동해야 하는 현실적인 요구가 증가하고 있다.6 FR-LIO와 같이 움직임 자체에 매우 강건한 주행계는 이러한 동적 초기화 알고리즘을 개발하는 데 필수적인 기반 기술이 될 것이다.43
- **딥러닝과의 융합 (Fusion with Deep Learning)**: 전통적인 기하학 기반 SLAM은 동적 객체나 조명 변화와 같은 비기하학적 요인에 취약하다.46 딥러닝 기술은 의미론적 정보(semantic information)를 추출하여 이러한 문제를 해결할 강력한 잠재력을 가지고 있다.48 FR-LIO의 가볍고 빠른 아키텍처는 계산 비용이 큰 딥러닝 기반 객체 탐지 및 제거 모듈과 결합하기에 이상적인 프론트엔드이다. 이를 통해 계산 부담을 최소화하면서도 동적 환경에서의 강건성을 극대화하는 차세대 하이브리드 SLAM 시스템을 구축할 수 있다.
- **엣지 AI 및 하드웨어 가속 (Edge AI and Hardware Acceleration)**: 로보틱스의 미래는 클라우드 의존성을 줄이고, 엣지 디바이스에서 실시간으로 AI를 처리하는 방향으로 나아가고 있다.40 FR-LIO의 뛰어난 단일 스레드 효율성은 복잡한 LIO 알고리즘을 NVIDIA Jetson이나 FPGA와 같은 저전력 임베디드 시스템에 탑재할 수 있는 현실적인 가능성을 제시한다.40 이는 고성능 자율 항법 기술의 적용 범위를 크게 확장하는 중요한 열쇠가 될 것이다.
- **차세대 센서로의 확장 (Extension to Next-Generation Sensors)**: 이벤트 카메라(Event Camera)나 4D 이미징 레이더(4D Imaging Radar)와 같은 차세대 센서들은 기존 센서의 한계를 뛰어넘는 장점(높은 시간 해상도, 악천후 강건성 등)을 제공하지만, 데이터가 비동기적이거나 희소(sparse)하여 기존의 프레임 기반 처리 방식으로는 다루기 어렵다.52 FR-LIO의 ESKS와 같이 연속 시간(continuous-time)에 가깝게 상태를 추정하는 방식과 효율적인 맵 구조는 이러한 새로운 센서 데이터를 효과적으로 융합하는 미래 SLAM 프레임워크의 핵심 기반 기술로 활용될 수 있다.

결론적으로, FR-LIO는 단순히 또 하나의 LIO 알고리즘을 넘어, '강건성'과 '효율성'이라는 상충될 수 있는 두 목표의 한계를 동시에 밀어붙임으로써 차세대 자율 시스템이 요구하는 핵심 역량을 명확히 제시했다. 이는 향후 LIO 연구가 나아가야 할 방향, 즉 더 가혹한 환경에서, 더 제한된 자원으로, 더 안정적으로 작동하는 시스템을 향한 중요한 기술적 이정표라 할 수 있다.


1. [2302.04031] Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - ar5iv, 8월 3, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/2302.04031
2. FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/368361497_FR-LIO_Fast_and_Robust_Lidar-Inertial_Odometry_by_Tightly-Coupled_Iterated_Kalman_Smoother_and_Robocentric_Voxels
3. A 2.5D Map-Based Mobile Robot Localization via Cooperation of Aerial and Ground Robots, 8월 3, 2025에 액세스, https://www.mdpi.com/1424-8220/17/12/2730
4. What is SLAM? A Beginner to Expert Guide - Kodifly, 8월 3, 2025에 액세스, https://kodifly.com/what-is-slam-a-beginner-to-expert-guide
5. Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/pdf/2302.04031
6. Fast and Robust LiDAR-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/380301592_Fast_and_Robust_LiDAR-Inertial_Odometry_by_Tightly-Coupled_Iterated_Kalman_Smoother_and_Robocentric_Voxels
7. VE-LIOM: A Versatile and Efficient LiDAR-Inertial Odometry and Mapping System - MDPI, 8월 3, 2025에 액세스, https://www.mdpi.com/2072-4292/16/15/2772
8. Versatile LiDAR-Inertial Odometry with SE(2) Constraints for Ground Vehicles - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2404.01584v1
9. Kalman filter - Wikipedia, 8월 3, 2025에 액세스, https://en.wikipedia.org/wiki/Kalman_filter
10. A State Optimization Model Based on Kalman Filtering and Robust Estimation Theory for Fusion of Multi-Source Information in Highly Non-linear Systems - PMC - PubMed Central, 8월 3, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC6479802/
11. Unscented Rauch--Tung--Striebel Smoother - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/3033057_Unscented_Rauch--Tung--Striebel_Smoother
12. The Invariant Rauch-Tung-Striebel Smoother - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2403.00075v1
13. FAST-LIO2: Fast Direct LiDAR-inertial Odometry | Request PDF - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/353258051_FAST-LIO2_Fast_Direct_LiDAR-inertial_Odometry
14. Illustration of incremental k-d tree update and re-balancing. (a) - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/figure/llustration-of-incremental-k-d-tree-update-and-re-balancing-a-an-existing-k-d-tree_fig1_349520013
15. arxiv.org, 8월 3, 2025에 액세스, [https://arxiv.org/pdf/2302.04031#:~:text=The%20iVox%20structure%20proposed%20in,performance%20of%20the%20ikd%2DTree.](https://arxiv.org/pdf/2302.04031#:~:text=The iVox structure proposed in,performance of the ikd-Tree.)
16. ROG-Map: An Efficient Robocentric Occupancy Grid Map for Large-scene and High-resolution LiDAR-based Motion Planning | Request PDF - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/368879715_ROG-Map_An_Efficient_Robocentric_Occupancy_Grid_Map_for_Large-scene_and_High-resolution_LiDAR-based_Motion_Planning
17. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - GitHub, 8월 3, 2025에 액세스, https://raw.githubusercontent.com/hku-mars/FAST_LIO/main/doc/Fast_LIO_2.pdf
18. [2107.06829] FAST-LIO2: Fast Direct LiDAR-inertial Odometry - ar5iv, 8월 3, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/2107.06829
19. FAST-LIO2: Fast Direct LiDAR-inertial Odometry - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/pdf/2107.06829
20. ikd-Tree: An Incremental KD Tree for Robotic Applications - Bohrium, 8월 3, 2025에 액세스, https://www.bohrium.com/paper-details/ikd-tree-an-incremental-k-d-tree-for-robotic-applications/867748286671356153-108625
21. (PDF) ikd-Tree: An Incremental K-D Tree for Robotic Applications - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/349520013_ikd-Tree_An_Incremental_K-D_Tree_for_Robotic_Applications
22. Faster-LIO: Lightweight Tightly Coupled Lidar-inertial Odometry using Parallel Sparse Incremental Voxels - GitHub, 8월 3, 2025에 액세스, https://github.com/gaoxiang12/faster-lio
23. (PDF) Faster-LIO: Lightweight Tightly Coupled Lidar-Inertial Odometry Using Parallel Sparse Incremental Voxels - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/359662313_Faster-LIO_Lightweight_Tightly_Coupled_Lidar-Inertial_Odometry_Using_Parallel_Sparse_Incremental_Voxels
24. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - GitHub, 8월 3, 2025에 액세스, https://github.com/TixiaoShan/LIO-SAM
25. Lio-sam: Tightly-coupled lidar inertial odometry via smoothing and mapping - arXiv, 8월 3, 2025에 액세스, http://arxiv.org/pdf/2007.00258
26. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/362412728_LIO-SAM_Tightly-coupled_Lidar_Inertial_Odometry_via_Smoothing_and_Mapping
27. [PDF] LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping, 8월 3, 2025에 액세스, https://www.semanticscholar.org/paper/LIO-SAM%3A-Tightly-coupled-Lidar-Inertial-Odometry-Shan-Englot/e49902d212374c81b48d23c3cac1cee6e2339edd
28. Performance Analysis of LIO-SAM - Ronia Peterson, 8월 3, 2025에 액세스, http://www.roniapeterson.com/LIO-SAM.pdf
29. A Comparative Study of Factor Graph Optimization-Based and Extended Kalman Filter-Based PPP-B2b/INS Integrated Navigation - MDPI, 8월 3, 2025에 액세스, https://www.mdpi.com/2072-4292/15/21/5144
30. Comparison of Extended Kalman Filter and Factor Graph Optimization for GNSS/INS Integrated Navigation System | Request PDF - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/340859540_Comparison_of_Extended_Kalman_Filter_and_Factor_Graph_Optimization_for_GNSSINS_Integrated_Navigation_System
31. A Comparison of Graph Optimization Approaches for Pose Estimation in SLAM - Laboratory for Autonomous Systems and Mobile Robotics, 8월 3, 2025에 액세스, https://lamor.fer.hr/images/50036607/2021-ajuric-comparison-mipro.pdf
32. Monocular visual simultaneous localization and mapping: (r)evolution from geometry to deep learning-based pipelines - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2503.02955v1
33. (PDF) A Hybrid Visual-Based SLAM Architecture: Local Filter-Based SLAM with KeyFrame-Based Global Mapping - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/357408926_A_Hybrid_Visual-Based_SLAM_Architecture_Local_Filter-Based_SLAM_with_KeyFrame-Based_Global_Mapping
34. LOCUS 2.0: Robust and Computationally Efficient Lidar Odometry for Real-Time 3D Mapping - JPL Robotics, 8월 3, 2025에 액세스, https://www-robotics.jpl.nasa.gov/media/documents/2205.11784.pdf
35. Robocentric map joining: Improving the consistency of EKF-SLAM - CiteSeerX, 8월 3, 2025에 액세스, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=6e372bf6ca97cf6884257b22ff6181ff661ed211
36. Q&A: How are researchers optimizing AI systems for science? | Penn State University, 8월 3, 2025에 액세스, https://www.psu.edu/news/engineering/story/qa-how-are-researchers-optimizing-ai-systems-science
37. From Underground Mines to Offices: A Versatile and Robust ... - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2407.14797v1
38. Observability Analysis of SLAM Using Fisher Information Matrix | Request PDF, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/224392024_Observability_Analysis_of_SLAM_Using_Fisher_Information_Matrix
39. Generalized Analysis and Improvement of the Consistency of EKF-based SLAM - People, 8월 3, 2025에 액세스, https://people.csail.mit.edu/ghuang/paper/tr/TR_slam_genconsistency.pdf
40. Edge AI Revolution: Top 10 Game-Changing Hardware Devices Leading 2025 - Huebits, 8월 3, 2025에 액세스, https://blog.huebits.in/edge-ai-revolution-top-10-game-changing-hardware-devices-leading-2025/
41. Unlocking Robotics with SLAM Technology - Number Analytics, 8월 3, 2025에 액세스, https://www.numberanalytics.com/blog/unlocking-robotics-with-slam-technology
42. Hardware Acceleration for SLAM in Mobile Systems | Journal of Computer Science and Technology - JCST, 8월 3, 2025에 액세스, https://jcst.ict.ac.cn/en/article/pdf/preview/10.1007/s11390-021-1523-5.pdf
43. [2504.01451] Dynamic Initialization for LiDAR-inertial SLAM - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/abs/2504.01451
44. Dynamic Initialization for LiDAR-inertial SLAM | Request PDF - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/390439964_Dynamic_Initialization_for_LiDAR-inertial_SLAM
45. hku-mars/LiDAR_IMU_Init: [IROS2022] Robust Real-time LiDAR-inertial Initialization Method. - GitHub, 8월 3, 2025에 액세스, https://github.com/hku-mars/LiDAR_IMU_Init
46. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - MDPI, 8월 3, 2025에 액세스, https://www.mdpi.com/1424-8220/24/23/7546
47. Review on VSLAM based on deep learning - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/385516006_Review_on_VSLAM_based_on_deep_learning
48. Review of Visual Simultaneous Localization and Mapping Based on Deep Learning - MDPI, 8월 3, 2025에 액세스, https://www.mdpi.com/2072-4292/15/11/2740
49. A Review of Research on SLAM Technology Based on the Fusion of LiDAR and Vision, 8월 3, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11902412/
50. Deep Learning for Visual SLAM in Transportation Robotics: A review - Oxford Academic, 8월 3, 2025에 액세스, https://academic.oup.com/tse/article/1/3/177/5747766
51. Optimizing Edge AI for Effective Real-time Decision Making in Robotics, 8월 3, 2025에 액세스, https://www.edge-ai-vision.com/2025/03/optimizing-edge-ai-for-effective-real-time-decision-making-in-robotics/
52. Event-based Vision, Event Cameras, Event Camera SLAM - Robotics and Perception Group, 8월 3, 2025에 액세스, https://rpg.ifi.uzh.ch/research_dvs.html
53. 4D Radar-based Pose Graph SLAM with Ego-velocity Pre-integration Factor - ResearchGate, 8월 3, 2025에 액세스, https://www.researchgate.net/publication/372145764_4D_Radar-based_Pose_Graph_SLAM_with_Ego-velocity_Pre-integration_Factor
54. Towards Mobile Sensing with Event Cameras on High-mobility Resource-constrained Devices: A Survey - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2503.22943v1
55. Radar and Event Camera Fusion for Agile Robot Ego-Motion Estimation - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/html/2506.18443v1
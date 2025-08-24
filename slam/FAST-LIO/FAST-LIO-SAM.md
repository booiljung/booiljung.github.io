[FAST-LIO](./index.md)
# FAST-LIO-SAM



동시적 위치추정 및 지도작성(Simultaneous Localization and Mapping, SLAM)은 로봇 공학의 핵심 과제 중 하나로, 로봇이 미지의 환경에 대한 지도를 구축함과 동시에 그 지도 내에서 자신의 위치를 추정하는 문제를 다룬다.1 본질적으로 SLAM은 "닭이 먼저냐, 달걀이 먼저냐"와 같은 순환적 의존성을 내포한다. 즉, 정확한 지도를 생성하기 위해서는 정밀한 위치 정보가 필요하고, 역으로 정밀한 위치를 추정하기 위해서는 정확한 지도가 전제되어야 한다.4 이 문제의 해결은 매핑 오류와 위치추정 오류가 서로 상관관계에 있으며, 공동으로 최적화될 수 있다는 이해에서 출발한다.4

SLAM 문제는 주로 두 가지 형태로 구분된다. 첫 번째는 'Full SLAM'으로, 로봇의 전체 경로와 지도를 함께 추정하는 문제이다.1 이는 모든 과거 데이터를 사용하여 전역적으로 일관된 결과를 도출하는 것을 목표로 하며, 주로 후처리나 스무딩(smoothing) 기반의 최적화 기법에서 다룬다. 두 번째는 'Online SLAM'으로, 전체 경로가 아닌 현재 시점의 로봇 위치와 지도를 추정하는 데 초점을 맞춘다.1 이는 필터링(filtering) 기반의 접근법에서 주로 사용되며, 실시간성을 확보하는 데 유리하다. 이 두 접근법의 철학적 차이는 최신 SLAM 시스템의 구조를 이해하는 데 중요한 기반이 된다.


라이다-관성 오도메트리(LiDAR-Inertial Odometry, LIO)는 SLAM 기술의 정확성과 강인성을 높이기 위해 등장한 핵심적인 센서 융합 기법이다. LiDAR 센서는 주변 환경에 대한 정밀하고 밀도 높은 3차원 기하학 정보를 제공하지만, 복도나 평원처럼 특징이 부족한(feature-degenerate) 환경에서는 성능이 저하될 수 있다.6 반면, 관성 측정 장치(Inertial Measurement Unit, IMU)는 가속도와 각속도 등 고주파의 모션 데이터를 제공하여 빠른 움직임에 강하지만, 시간이 지남에 따라 오차가 누적되는 드리프트(drift) 현상이 심각하다.8

LIO는 이 두 센서의 장점을 결합하고 단점을 상호 보완한다. IMU 데이터는 LiDAR 스캔 사이의 움직임을 예측하여 포인트 클라우드의 왜곡(de-skewing)을 보정하고, 특히 급격한 움직임이 있을 때 스캔 정합(scan matching)을 위한 정확한 초기 추정치를 제공한다.10 동시에, LiDAR로부터 얻은 정밀한 기하학적 제약 조건은 IMU의 누적된 드리프트를 보정하고 바이어스(bias)를 추정하는 데 사용된다.12 이러한 융합 방식은 '강결합(Tightly-coupled)'과 '약결합(Loosely-coupled)'으로 나뉜다. 강결합 방식은 각 센서의 원시 측정값을 단일 최적화 프레임워크 내에서 함께 처리하여 상호작용을 극대화하는 반면, 약결합 방식은 각 센서 처리 파이프라인의 결과물을 나중에 융합한다. 현대의 고성능 LIO 시스템은 대부분 강결합 방식을 채택하여 최고의 정확도를 추구한다.12


LIO-SLAM을 포함한 현대 SLAM 기술의 발전은 근본적으로 필터링 기법의 실시간 효율성과 최적화 기법의 전역적 정확성 사이의 균형점을 찾는 과정이었다. 이 두 가지 접근 방식은 SLAM 문제를 해결하는 서로 다른 철학을 대표한다.

**필터링 접근법 (Online SLAM):** 이 접근법은 칼만 필터(Kalman Filter), 특히 확장 칼만 필터(Extended Kalman Filter, EKF)를 사용하여 상태를 재귀적으로 추정한다.4 시스템의 상태는 모션 모델을 통해 시간 순서대로 예측되고, 새로운 측정값이 들어올 때마다 보정된다. 이 방식의 핵심 특징은 과거 상태 정보를 '한계화(marginalize out)'하여 현재 상태 추정에만 집중한다는 점이다. 이로 인해 계산적으로 매우 효율적이어서 실시간 응용에 적합하지만, 과거의 오류를 수정할 수 없어 시간이 지남에 따라 비일관적인 지도를 생성할 수 있다.5

**최적화 접근법 (Full SLAM):** 그래프 기반 최적화는 SLAM 문제를 모든 로봇의 포즈와 랜드마크 위치에 대한 최대 사후 확률(Maximum a Posteriori, MAP) 추정 문제를 푸는 것으로 재정의한다.4 이는 팩터 그래프(factor graph)로 표현되는데, 노드(node)는 추정하고자 하는 상태(포즈, 랜드마크)를, 팩터(factor)는 센서 측정값(오도메트리, 루프 폐쇄)으로부터 파생된 제약 조건을 나타낸다.4 역사적으로 이 방식은 모든 데이터를 한 번에 처리하는 배치(batch) 방식으로 인해 계산 비용이 매우 높았지만, iSAM(incremental Smoothing and Mapping)과 같은 증분 최적화 기법의 등장은 최적화 기반 접근법을 실시간으로 활용할 수 있는 길을 열었다.10

이러한 배경 속에서 FAST-LIO-SAM과 같은 하이브리드 시스템이 등장하게 된 동기는 명확해진다. 이는 단순히 두 가지 철학 중 하나를 선택하는 것이 아니라, 고도로 효율적인 필터 기반 프런트엔드(front-end)를 지역적 오도메트리에 사용하고, 강력한 최적화 기반 백엔드(back-end)를 전역 지도 보정에 사용하여 역사적인 '효율성 대 정확성'의 딜레마에 대한 직접적인 해결책을 제시하는 것이다.


FAST-LIO-SAM의 하이브리드 아키텍처를 이해하기 위해서는 그 기술적 뿌리가 되는 선행 연구들의 계보를 추적하는 것이 필수적이다. 각 알고리즘은 이전 기술의 한계를 극복하고 새로운 패러다임을 제시하며 발전해왔다.


LiDAR Odometry and Mapping (LOAM)은 현대 LiDAR SLAM의 기틀을 마련한 기념비적인 연구다.17 LOAM의 핵심 혁신은 복잡한 SLAM 문제를 두 개의 병렬적이고 상호작용하는 알고리즘으로 분해하여 실시간성을 확보한 것이다.19

- **LiDAR Odometry:** 약 10 Hz의 높은 주기로 동작하며, 연속된 두 스캔 사이의 특징점을 정합하여 로봇의 움직임을 빠르게 추정한다. 이 모듈은 실시간 속도 추정치를 제공하고, 이를 이용해 LiDAR 스캔 과정에서 발생하는 포인트 클라우드의 왜곡을 보정하는 역할을 한다.18
- **LiDAR Mapping:** 약 1 Hz의 낮은 주기로 동작하며, 왜곡이 보정된 포인트 클라우드를 지금까지 생성된 전역 지도에 정합한다. 이 과정을 통해 오도메트리 단계에서 발생한 드리프트를 보정하고 더 정밀한 포즈 추정치를 생성한다.18
- **특징점 추출:** LOAM은 전체 포인트 클라우드를 사용하는 대신, 각 점의 주변 곡률(curvature)을 계산하여 특징점을 추출한다. 곡률이 큰 '모서리점(edge point)'과 곡률이 작은 '평면점(planar point)'을 구분하여 정합에 사용함으로써 계산 효율성을 크게 높였다.18

하지만 LOAM은 결정적으로 루프 폐쇄(loop closure) 기능이 없어, 긴 경로를 주행할 경우 누적된 오차를 보정할 방법이 없었다.2 따라서 완전한 의미의 SLAM 시스템이라기보다는 정밀한 오도메트리 및 매핑 시스템에 가까웠다.


LeGO-LOAM은 LOAM을 계승하면서, 특히 연산 능력이 제한된 지상 로봇을 위해 경량화에 초점을 맞춘 알고리즘이다.22

- **지면 분할:** 가장 큰 기여는 포인트 클라우드 처리 초기에 지면점을 분할하여 제거하는 것이다. 이는 처리해야 할 포인트의 수를 대폭 줄여 알고리즘의 속도를 크게 향상시켰다.22
- **2단계 최적화:** 6자유도(6-DOF) 포즈 추정을 두 단계로 분리했다. 먼저 평면 특징을 이용해 회전과 z축 이동을 추정하고, 그 다음 모서리 특징을 이용해 나머지 이동을 추정한다. 이 방식은 LOAM의 동시 최적화 방식보다 계산 시간을 단축시키는 효과를 가져왔다.22

LeGO-LOAM은 지능적인 전처리 및 최적화 전략을 통해 저전력 임베디드 시스템에서도 LiDAR SLAM을 실시간으로 구동할 수 있음을 입증했다.22


LIO-SAM은 LOAM 계열의 아이디어를 확장하여 강력한 백엔드를 통합한 강결합 LIO 프레임워크로, 완전한 SLAM 시스템을 구현했다.10

- **팩터 그래프 백엔드:** 핵심 혁신은 LIO 문제를 GTSAM(Georgia Tech Smoothing and Mapping) 라이브러리를 사용한 팩터 그래프 위에서 공식화한 것이다.11 이를 통해 다양한 센서 측정값과 제약 조건을 원활하게 통합할 수 있게 되었다.
- 주요 팩터 11:
  - **IMU 사전 통합 팩터(IMU Pre-integration Factor):** IMU 데이터로부터 연속된 포즈 간의 상대적인 움직임 제약을 생성한다. 이를 통해 로봇의 상태와 IMU 바이어스를 공동으로 최적화할 수 있다.
  - **LiDAR 오도메트리 팩터(LiDAR Odometry Factor):** 현재 스캔을 '서브 키프레임(sub-keyframes)'으로 구성된 지역적인 슬라이딩 윈도우 맵에 정합하여 얻은 제약이다.
  - **GPS 팩터(GPS Factor, 선택 사항):** GPS와 같은 절대 위치 측정값을 그래프에 추가하여 지도를 전역 좌표계에 고정시키고 장기적인 드리프트를 제거할 수 있다.
  - **루프 폐쇄 팩터(Loop Closure Factor):** 이전에 방문했던 장소를 다시 인식하면, 현재 포즈와 과거 포즈 사이에 제약 조건을 추가한다. 그래프 최적화기는 이 제약을 이용해 누적된 오차를 전역적으로 보정한다.
- **실시간 성능:** 오래된 스캔을 한계화하고 슬라이딩 윈도우 방식을 사용함으로써, 계속해서 커지는 전역 지도에 정합할 필요 없이 실시간 성능을 보장한다.12 증분 최적화 솔버인 iSAM2의 사용이 이러한 실시간 성능의 핵심이다.10


FAST-LIO 계열(FAST-LIO, FAST-LIO2)은 LOAM 스타일의 특징점 기반 프런트엔드에서 벗어난 새로운 패러다임을 제시했다.13

- **강결합 반복 칼만 필터:** 별도의 오도메트리/매핑 구조 대신, 강결합 반복 오차 상태 칼만 필터(iEKF)를 사용하여 원시 LiDAR 포인트와 IMU 데이터를 직접 융합한다.13
- **직접적인 스캔-맵 정합:** FAST-LIO2는 모서리/평면 특징을 명시적으로 추출하는 대신, 다운샘플링된 원시 포인트를 지도에 직접 정합하는 방식을 채택했다.27 이는 명확한 특징이 부족한 환경에서 더 강인할 수 있다.
- **계산 효율성:** 이름의 'FAST'는 칼만 게인(Kalman gain)을 계산하는 새로운 공식에서 유래했다. 이 공식은 계산 복잡도가 측정값의 차원(수천 개의 LiDAR 포인트)이 아닌 상태 변수의 차원(작고 고정됨)에 의존하도록 설계되어, 필터 업데이트 단계를 극도로 빠르게 만든다.13

LIO-SAM이 백엔드를 강화하는 방향으로 진화했다면, FAST-LIO2는 프런트엔드를 혁신하는 방향으로 발전했다. 이 두 가지 다른 진화의 흐름이 바로 FAST-LIO-SAM이라는 하이브리드 아키텍처의 탄생 배경이 된다. 아래 표는 이러한 기술적 진화 과정을 요약하여 보여준다.


| 기능              | LOAM (2014)                         | LeGO-LOAM (2018)              | LIO-SAM (2020)                 | FAST-LIO2 (2021)                    |
| ----------------- | ----------------------------------- | ----------------------------- | ------------------------------ | ----------------------------------- |
| **프런트엔드**    | 약결합 LiDAR 오도메트리             | 지면 최적화 오도메트리        | 강결합 LIO (IMU 사전 통합)     | 강결합 iEKF                         |
| **백엔드**        | 저주파 매핑                         | 저주파 매핑 + 루프 폐쇄       | 팩터 그래프 최적화 (iSAM2)     | 별도 백엔드 없음 (iEKF 매핑에 통합) |
| **포인트 정합**   | 모서리/평면 특징                    | 모서리/평면 특징              | 모서리/평면 특징               | 직접 포인트-맵 정합                 |
| **핵심 자료구조** | 특징점 지도                         | 특징점 지도                   | 서브 키프레임 기반 복셀 맵     | ikd-Tree                            |
| **루프 폐쇄**     | 없음                                | 지원 (거리 기반 + ICP)        | 지원 (팩터 그래프 기반)        | 없음 (오도메트리 집중)              |
| **주요 기여**     | 이중 알고리즘(오도메트리/매핑) 구조 | 임베디드 시스템용 경량 최적화 | 팩터 그래프를 통한 전역 일관성 | 계산 효율적인 iEKF 프런트엔드       |


FAST-LIO-SAM의 프런트엔드를 구성하는 FAST-LIO2의 성능은 두 가지 핵심 기술, 즉 반복 오차 상태 칼만 필터(iEKF)와 증분 k-d 트리(ikd-Tree)에 기반한다. 이 두 기술의 시너지는 고성능 직접 LIO 프런트엔드를 가능하게 했다.


FAST-LIO에서 사용하는 iEKF 프레임워크는 전통적인 EKF의 한계를 극복하고 강결합 센서 융합을 효율적으로 수행하기 위해 설계되었다.13

- **오차 상태 공식화:** 필터는 전체 상태 x를 직접 추정하는 대신, 참 상태(true state)를 명목 상태(nominal state)와 작은 오차 상태(error state)의 합(x=xˉ⊞x~)으로 모델링한다.27 필터는 이 작은 오차 상태 $\tilde{x}$에 대해 동작하는데, 이는 선형화 가정이 더 잘 성립하고 회전 표현에서 발생하는 특이점(singularity) 문제를 회피하는 데 유리하다.
- **예측 및 반복적 업데이트:** IMU 측정값은 명목 상태를 시간상으로 전파(예측 단계)하는 데 사용된다. LiDAR 스캔이 도착하면 업데이트 단계가 수행된다. '반복적(iterated)'이라는 의미는 단일 업데이트 단계 내에서 선형화 지점을 여러 번 재계산하여 최적의 해에 더 가깝게 수렴시키는 것을 말하며, 이는 표준 EKF보다 높은 정확도를 제공한다.
- **암시적 측정 모델(Implicit Measurement Model):** LiDAR 측정 모델은 상태 x를 측정값 z로 직접 매핑하는 함수 $z = h(x)$가 존재하지 않는 '암시적' 형태를 띤다. 대신, 완벽한 상태 추정치에 대해 0이 되어야 하는 잔차(residual) 조건을 통해 간접적으로 정의된다. 예를 들어, 변환된 LiDAR 포인트와 지도상의 해당 평면 사이의 거리가 0이 되어야 한다는 조건이 바로 그것이다.27
- **효율적인 칼만 게인:** FAST-LIO의 가장 핵심적인 혁신이다. 표준 칼만 게인 계산은 측정값(LiDAR 포인트)의 수에 비례하는 크기의 행렬 역산을 포함하는데, 이는 수천 개의 포인트에 대해 계산적으로 불가능하다. FAST-LIO는 이 거대한 행렬 역산을 피하는 새로운 공식을 적용하여, 계산 비용이 훨씬 작은 상태 변수의 차원에만 의존하도록 만들었다.13 이것이 수천 개의 포인트를 사용하면서도 실시간 성능을 달성하는 비결이다.


`ikd-tree`는 FAST-LIO2의 또 다른 주요 기여로, SLAM 매핑 과정의 동적인 특성에 맞춰 설계된 새로운 자료구조이다.27

- **배경 (k-d Tree):** 표준 k-d 트리는 k차원 공간의 점들을 효율적으로 구성하여 최근접 이웃 탐색(nearest-neighbor search)을 가속화하는 공간 분할 자료구조이다.27 하지만 이 구조는 정적이어서, 점을 추가하거나 삭제할 때마다 비용이 많이 드는 전체 또는 부분적인 재구축이 필요하다는 단점이 있다.
- **ikd-Tree의 혁신:** `ikd-tree`는 SLAM 매핑과 같이 점진적으로 데이터가 추가되고 삭제되는 동적인 환경에 최적화된 k-d 트리이다.27
- ikd-Tree의 주요 특징 27:
  - **증분 업데이트:** 단일 포인트 또는 박스 단위의 포인트 그룹을 효율적으로 추가하고 삭제할 수 있다.
  - **동적 리밸런싱:** 데이터가 변경될 때 트리를 처음부터 다시 만들지 않고, 필요한 부분만 동적으로 재조정(rebalance)한다. 이는 전통적인 k-d 트리 업데이트에 비해 4%의 시간만 소요될 정도로 엄청난 성능 향상을 가져온다.27
  - **병렬 리밸런싱:** 큰 서브 트리의 재조정이 필요할 경우, 병렬 처리를 통해 성능 저하 없이 빠르게 작업을 완료할 수 있다.
  - **내장 다운샘플링:** 트리 구조 자체를 활용하여 복셀 그리드 다운샘플링을 효율적으로 지원한다.

이러한 `ikd-tree`의 효율성은 FAST-LIO2가 대규모 포인트 클라우드 맵을 실시간으로 유지하고 쿼리할 수 있게 하는 핵심 요소이다. 이후 Faster-LIO의 `iVox`(해시 테이블과 LRU 캐시 기반)나 FR-LIO의 `RC-Vox`(고정 크기 2계층 3D 배열)와 같이 훨씬 더 빠른 자료구조들이 제안되었지만, `ikd-tree`는 동적 포인트 클라우드 관리에 대한 중요한 패러다임 전환을 제시했다.28

결론적으로, 효율적인 칼만 게인 공식과 `ikd-tree`의 조합은 LIO 프런트엔드에 대한 접근 방식을 근본적으로 바꾸었다. 이는 LOAM의 '특징 추출 후 정합' 철학에서 '직접 융합 및 효율적 매핑' 철학으로의 전환을 의미한다. 이로 인해 프런트엔드는 더 빠를 뿐만 아니라, LOAM이 의존하는 뚜렷한 기하학적 특징이 부족한 환경에서도 더 강인한 성능을 발휘할 수 있게 되었다.



'FAST-LIO-SAM'은 단일 논문으로 발표된 공식적인 알고리즘이 아니라, 오픈소스 커뮤니티를 중심으로 형성된 사실상의 표준(de-facto standard) 하이브리드 아키텍처이다.32 이 아키텍처는 두 세계의 최고 장점, 즉 FAST-LIO2의 고성능 프런트엔드와 LIO-SAM의 강력한 백엔드를 결합한 것이다.

- **프런트엔드 (Front-End):** **FAST-LIO2**의 고주파, 저드리프트, 필터 기반 오도메트리를 사용한다. 이는 매우 정확하고 강인한 실시간 포즈 추정치를 제공하며, `ikd-tree`를 사용하여 지역 지도를 효율적으로 관리한다.
- **백엔드 (Back-End):** **LIO-SAM**의 그래프 기반 최적화 프레임워크를 사용한다. 이는 포즈 그래프 최적화(PGO)와 루프 폐쇄를 통해 전역 지도의 일관성을 보장하는 메커니즘을 제공한다.
- **인터페이스:** 두 시스템의 연결은 직관적이다. FAST-LIO2 프런트엔드에서 생성된 저드리프트 포즈 추정치와 키프레임들이 LIO-SAM 백엔드의 팩터 그래프에 노드와 오도메트리 팩터로 입력된다.

이러한 분리된 설계는 SLAM 분야의 성숙을 보여주는 지표로, 각 도메인(프런트엔드와 백엔드)에서의 독립적인 혁신을 가능하게 한다. 프런트엔드는 실시간 상태 추정의 품질을 책임지고, 백엔드는 장기적인 지도 일관성을 책임지는 명확한 역할 분담이 이루어진다.


FAST-LIO-SAM이라는 개념은 실제 코드 구현을 통해 정의되므로, 주요 GitHub 저장소를 분석하는 것은 매우 중요하다.

- `GDUT-Kyle/faster_lio_sam` 32:

   아이디어를 결합한 초기 예시 중 하나이다. 융합 및 오도메트리 부분(`fusionOptimization.cpp`)에서는 IESKF와 iVox(Faster-LIO에서 유래)를 사용하고, IMU 바이어스 추정 및 오도메트리 팩터 생성(`imuPreintegration.cpp`)에는 iSAM2를 사용하여 프런트엔드와 백엔드의 개념을 명확히 분리했다.

- `engcang/FAST-LIO-SAM` 33:

   FAST_LIO를 git 서브모듈로 직접 사용하여 모듈식 접근 방식을 명확히 보여주는 보다 직접적인 결합이다.

- `engcang/FAST-LIO-SAM-QN` 34:

   매우 진보되고 모듈화된 구현체이다. FAST-LIO2를 PGO 모듈과 명시적으로 결합하며, 정교한 루프 폐쇄 기술을 도입했다.

  - **Quatro:** 루프 폐쇄를 위한 좋은 초기 추정치를 빠르고 강인하게 제공하는 전역 정합 알고리즘이다.
  - **Nano-GICP:** 루프 폐쇄 정렬을 미세 조정하기 위한 빠른 ICP(Iterative Closest Point) 변종이다.

- `engcang/FAST-LIO-SAM-SC-QN` 35:

   QN 변종을 한 단계 더 발전시킨 버전이다. 효율적인 루프 후보 탐지를 위해 전역 디스크립터인 **Scan Context**를 추가했다. 여기서 탐지된 후보는 Quatro와 Nano-GICP를 통해 검증된다. 이는 루프 폐쇄에 대한 계층적 접근 방식을 보여준다.

이러한 구현체들은 현대 SLAM 시스템이 "플러그 앤 플레이" 방식으로 발전하고 있음을 보여준다. 기본 오도메트리와 PGO 프레임워크는 유지하면서, 루프 폐쇄 모듈과 같은 특정 구성 요소를 교체하며 성능을 개선하는 모듈식 설계가 트렌드임을 알 수 있다.


| 저장소                       | 기본 오도메트리           | PGO 프레임워크         | 루프 탐지                         | 주요 의존성          | 라이선스           |
| ---------------------------- | ------------------------- | ---------------------- | --------------------------------- | -------------------- | ------------------ |
| `TixiaoShan/LIO-SAM`         | 특징점 기반 (LOAM 스타일) | GTSAM (iSAM2)          | 반경 탐색 + ICP                   | GTSAM                | Apache 2.0         |
| `hku-mars/FAST_LIO`          | iEKF + ikd-Tree           | 없음 (오도메트리 전용) | 없음                              | Eigen                | BSD 3-Clause       |
| `GDUT-Kyle/faster_lio_sam`   | IESKF + iVox              | GTSAM (iSAM2)          | 거리 기반 (외부)                  | GTSAM, PCL, OpenCV   | 명시되지 않음      |
| `engcang/FAST-LIO-SAM-QN`    | FAST-LIO2                 | GTSAM (iSAM2)          | Quatro + Nano-GICP                | GTSAM, Teaser++, tbb | CC BY-NC-SA 4.0 36 |
| `engcang/FAST-LIO-SAM-SC-QN` | FAST-LIO2                 | GTSAM (iSAM2)          | Scan Context + Quatro + Nano-GICP | GTSAM, Teaser++, tbb | CC BY-NC-SA 4.0    |

*참고: `FAST-LIO2`는 `hku-mars/FAST_LIO`의 후속 버전이며, 일반적으로 `FAST-LIO-SAM` 구현체들은 `FAST-LIO2`를 기반으로 한다.*


하이브리드 시스템의 백엔드에서 수행되는 PGO 과정은 다음과 같은 단계로 이루어진다.34

- **키프레임 감지:** 프런트엔드는 연속적으로 포즈를 생성하지만, 그래프의 계산 복잡도를 관리하기 위해 특정 조건(예: 이동 거리나 회전 각도 임계값 초과)을 만족하는 포즈만 '키프레임'으로 선택하여 그래프에 추가한다.37
- **팩터 추가:** 새로운 키프레임이 추가될 때마다, 이전 키프레임으로부터의 상대적인 움직임을 나타내는 오도메트리 팩터(FAST-LIO2에 의해 추정됨)가 그래프에 추가된다.
- **iSAM2 최적화:** GTSAM/iSAM2 솔버는 오도메트리나 루프 폐쇄 같은 새로운 팩터가 추가될 때마다 전체 포즈 그래프를 증분적으로 최적화하여, 지도가 전역적으로 일관성을 유지하도록 보장한다.32


장소 인식 및 루프 폐쇄는 SLAM의 누적 오차를 제거하는 데 결정적인 역할을 한다. 최신 FAST-LIO-SAM 구현체들은 다음과 같은 고급 모듈을 계층적으로 사용한다.

- Scan Context (SC) 3:

   LiDAR 포인트 클라우드를 위한 전역 디스크립터이다. 3D 스캔을 방사형 및 섹터 빈(bin) 내 포인트들의 높이 정보를 기반으로 2D 행렬로 표현한다.21 이는 대규모 키프레임 데이터베이스에서 루프 

  *후보*를 매우 빠르게 찾는 데 사용되지만, 정밀한 정렬에는 충분하지 않다.

- Quatro 34:

   빠른 전역 정합 알고리즘으로, SC 등을 통해 찾은 루프 후보 스캔과 현재 스캔 사이의 대략적인 초기 정렬 값을 제공하는 데 사용된다.

- ICP 및 변종 (Nano-GICP) 34:

   ICP는 포인트 클라우드를 정밀하게 정합하는 고전적인 알고리즘이다. Nano-GICP는 그 빠른 구현체로, 루프 폐쇄 제약 조건을 위한 최종 변환 행렬을 서브 복셀(sub-voxel) 수준의 정확도로 계산하는 데 사용된다.

가장 발전된 시스템(`FAST-LIO-SAM-SC-QN`)은 이러한 모듈을 계층적으로 결합한다. 1) Scan Context가 수많은 키프레임 중에서 빠르게 루프 후보를 찾고, 2) Quatro가 이 후보에 대한 강인한 초기 정렬을 제공하며, 3) Nano-GICP가 최종적으로 정밀하게 정렬을 완료한다. 이 조합은 속도와 정밀도를 모두 만족시키는 효과적인 전략이다.



SLAM 시스템의 궤적 정확도를 평가하기 위해 사용되는 표준 지표는 다음과 같다.

- **절대 궤적 오차 (Absolute Trajectory Error, ATE):** 궤도의 전역적인 일관성을 측정한다. 추정된 궤도와 실제 참값(ground truth) 궤도를 정렬한 후, 모든 포즈에 대해 직접적인 차이를 계산하며, 보통 제곱근 평균 오차(Root Mean Square Error, RMSE)로 보고된다.37 ATE는 전체적인 드리프트를 시각화하고 평가하는 데 유용하다.
- **상대 포즈 오차 (Relative Pose Error, RPE):** 고정된 시간 또는 거리 간격 동안의 지역적인 정확성 또는 드리프트를 측정한다. 포즈 쌍 사이의 상대적인 움직임 오차를 계산하므로, 오도메트리 자체의 성능을 평가하는 데 적합하다.39


KITTI Vision Benchmark Suite, 특히 오도메트리 벤치마크는 실제 주행 환경에서 수집된 LiDAR, IMU, 그리고 GPS/INS 참값 데이터를 제공하여 SLAM 알고리즘을 평가하는 표준으로 널리 사용된다.37

여러 연구에서 보고된 성능 분석 결과는 다음과 같다.

- KITTI 공식 리더보드에 따르면, LOAM이나 V-LOAM과 같은 최상위권 방법들은 약 0.5-0.6% 수준의 ATE를 달성하며, 이는 성능 비교의 기준점이 된다.42
- LIO-SAM 및 FAST-LIO 계열 알고리즘을 평가한 여러 논문들은 이들과 경쟁적이거나 더 우수한 성능을 보고했다.37
- 특히 LIO-SAM++ 논문은 여러 KITTI 시퀀스에서 LEGO-LOAM, LIO-SAM, FAST-LIO와 성능을 상세히 비교했으며, 이들 대비 평균 ATE가 8-19% 개선되었음을 보여주었다.37
- 시맨틱 Lidar-Inertial SLAM 논문은 동적 객체가 많은 UrbanNav 데이터셋에서 LIO-SAM의 RMSE를 13.069m에서 5.543m로 크게 줄이는 극적인 성능 향상을 보였다.46 이는 KITTI는 아니지만 시맨틱 필터링의 효과를 명확히 보여주는 사례이다.


FAST-LIO-SAM의 우수성은 그 하이브리드 특성에서 기인한다.

- **낮은 지역적 드리프트:** FAST-LIO2 프런트엔드로부터 극히 낮은 드리프트의 오도메트리 성능을 물려받았다. 이는 루프 폐쇄 사이의 구간이나 루프가 없는 긴 궤적에서 LIO-SAM의 네이티브 프런트엔드와 같은 특징점 기반 방식보다 오차 누적이 훨씬 느리다는 것을 의미한다.
- **전역적 일관성:** LIO-SAM으로부터 강력한 백엔드를 물려받았다. 루프가 감지되면 iSAM2 최적화기가 전체 궤적을 보정하여 누적된 드리프트를 제거할 수 있다.
- **강인성:** 필터 기반 프런트엔드는 급격한 움직임에 강인하며 28, 특징이 부족한 환경에서도 LOAM 스타일 프런트엔드보다 덜 민감할 수 있다. 백엔드는 장기적인 드리프트에 대한 강인성을 제공한다.

결론적으로, FAST-LIO-SAM은 지역적으로는 FAST-LIO2의 정확성을, 전역적으로는 LIO-SAM의 일관성을 모두 갖춤으로써, 각 구성 요소의 단점을 보완하고 장점을 극대화하는 매우 효과적인 SLAM 솔루션이다.



퇴화(degenerate) 또는 '특징 없는(structure-less)' 환경이란 긴 복도, 넓은 평원, 터널과 같이 LiDAR 데이터만으로는 6자유도 움직임을 모두 결정할 만큼 충분한 기하학적 제약 조건을 얻을 수 없는 공간을 의미한다.7 이러한 환경에서 LiDAR 단독 SLAM은 실패하기 쉽다.

이때 FAST-LIO 프런트엔드의 IMU와의 강결합이 결정적인 역할을 한다. LiDAR가 움직임을 관측할 수 없을 때에도 IMU는 회전과 같은 움직임을 감지할 수 있으므로, 오도메트리가 실패하거나 제어 불가능하게 드리프트하는 것을 막아준다.7 이는 LiDAR 전용 SLAM에 비해 LIO가 갖는 핵심적인 장점이다.


대부분의 기하학적 SLAM 알고리즘은 '정적인 세계 가정(static world assumption)'이라는 근본적인 약점을 가지고 있다.48 움직이는 자동차나 보행자와 같은 동적 객체는 잘못된 제약 조건을 생성하여 부정확한 포즈 추정을 유발하고 지도에 '유령(ghosting)' 현상을 남긴다.48 순수 기하학적 알고리즘은 움직이는 자동차와 정적인 벽을 구분할 수 없으며, 둘 다 그저 점들의 집합으로 인식할 뿐이다.

이 문제에 대한 해결책으로 딥러닝 모델을 활용한 시맨틱 분할(semantic segmentation) 기술을 통합하는 것이 새로운 트렌드로 부상하고 있다.37

- **방법론:** SPVNAS나 RangeNet++와 같은 신경망이 LiDAR 데이터에 대해 각 포인트에 '자동차', '보행자', '건물'과 같은 시맨틱 레이블을 할당한다.44 이후 동적일 가능성이 있는 레이블의 포인트들은 스캔 정합 및 매핑 과정에서 제외된다.
- **영향:** 이러한 동적 객체 필터링은 복잡한 도심 환경에서 정확도를 극적으로 향상시킨다. LIO-SAM++ 37 및 시맨틱 SLAM 관련 연구 46의 정량적 결과는 궤적 오차가 크게 감소함을 보여주며 그 효과를 입증한다.
- **고급 키프레임 선택:** 시맨틱 정보는 키프레임 선택 전략을 개선하는 데에도 사용될 수 있다. 단순히 이동 거리에 의해서가 아니라, 장면의 *시맨틱 내용*에 중요한 변화가 감지되었을 때 새로운 키프레임을 생성함으로써, 환경의 의미 있는 변화를 놓치지 않고 포착할 수 있다.37

이러한 발전은 SLAM 기술이 통제된 환경을 넘어 혼란스러운 실제 세계로 나아가기 위한 필연적인 진화 과정이다. 순수 기하학적 방법의 실패가 딥러닝 기반의 인식 기술을 SLAM 파이프라인에 통합시키는 직접적인 동인이 된 것이다.


- **센서 선택 및 설정:** 필수 센서로는 3D LiDAR(Velodyne, Ouster, Livox 등)와 9축 IMU가 있다.25 GPS는 선택 사항이지만 대규모 드리프트를 보정하는 데 매우 권장된다.11 LiDAR와 IMU 간의 외부 파라미터(extrinsic calibration) 보정과 타임스탬프 동기화의 정확성은 시스템 성능에 지대한 영향을 미치므로 매우 중요하다.25
- **소프트웨어 의존성:** 시스템 구축에는 ROS(Robot Operating System), PCL(Point Cloud Library), Eigen, GTSAM이 핵심적으로 요구된다.25 고급 변종의 경우 tbb, Teaser++ 등의 추가 라이브러리가 필요할 수 있다.34
- **설정:** `params.yaml`과 같은 설정 파일에서 센서 토픽, 외부 파라미터 변환 행렬, LiDAR 고유 파라미터(예: 스캔 라인 수) 등을 정확하게 설정해야 한다.25
- **구현체 선택:** 최고의 성능과 최신 기능을 원한다면 `FAST-LIO-SAM-SC-QN`과 같은 변종이 좋은 선택일 수 있다. 반면, 더 간단한 시작점을 원한다면 `faster_lio_sam`이 접근하기 쉬울 수 있다. 상업적 프로젝트의 경우, 일부 구현체의 비상업적 라이선스 36는 매우 중요한 고려 사항이다.


본 보고서는 FAST-LIO-SAM 아키텍처에 대한 심층적인 분석을 제공했다. 분석을 통해 도출된 핵심 결론은 다음과 같다.

첫째, FAST-LIO-SAM은 단일 알고리즘이 아니라, 두 가지 주요 LIO-SLAM 철학의 종합을 대표하는 강력하고 커뮤니티 주도적인 하이브리드 아키텍처이다. 이는 FAST-LIO2의 고효율 필터 기반 프런트엔드와 LIO-SAM의 전역적으로 일관된 최적화 기반 백엔드를 결합한 결과물이다.

둘째, 이 아키텍처는 지역적 정확성을 위한 저드리프트 오도메트리 프런트엔드와 전역적 일관성을 위한 강력한 포즈 그래프 백엔드를 활용함으로써 탁월한 성능을 보인다. FAST-LIO2 프런트엔드는 급격한 움직임과 특징이 부족한 환경에서도 강인한 실시간 포즈를 제공하며, LIO-SAM 백엔드는 루프 폐쇄와 같은 전역 제약 조건을 통해 누적 오차를 효과적으로 제거한다.

셋째, 이러한 시스템의 모듈식, 오픈소스 특성은 연구 개발을 가속화하여, 루프 폐쇄나 시맨틱 분할과 같은 최첨단 구성 요소들을 신속하게 통합할 수 있게 했다. 이는 SLAM 기술이 순수 기하학적 접근을 넘어, 동적이고 복잡한 실제 환경에 대응하기 위해 딥러닝 기반의 시맨틱 인식을 통합하는 방향으로 진화하고 있음을 시사한다.

결론적으로, FAST-LIO-SAM은 현재 자율 시스템을 위해 사용 가능한 가장 유능하고 다재다능한 오픈소스 SLAM 솔루션 중 하나로, 그 설계 철학과 성능은 현대 SLAM 기술의 발전 방향을 명확하게 보여준다.


1. 46. Simultaneous Localization and Mapping - CS@Columbia, accessed July 19, 2025, https://www.cs.columbia.edu/~allen/F19/NOTES/slam_paper.pdf
2. What Is SLAM (Simultaneous Localization and Mapping)? - MATLAB & Simulink, accessed July 19, 2025, https://www.mathworks.com/discovery/slam.html
3. Optimized Fast LiDAR Odometry and Mapping Using Scan Context - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2204.04932
4. What is SLAM? A Beginner to Expert Guide - Kodifly, accessed July 19, 2025, https://kodifly.com/what-is-slam-a-beginner-to-expert-guide
5. Square Root SAM - Frank Dellaert, accessed July 19, 2025, https://dellaert.github.io/files/Dellaert06ijrr.pdf
6. 786studio.tistory.com, accessed July 19, 2025, [https://786studio.tistory.com/6#:~:text=1.1.%20LIO%2DSAM%20%EC%9D%B4%EB%9E%80,%EC%88%98%20%EC%9E%88%EB%8A%94%20SLAM%20%EA%B8%B0%EB%B2%95%EC%9E%85%EB%8B%88%EB%8B%A4.](https://786studio.tistory.com/6#:~:text=1.1. LIO-SAM 이란,수 있는 SLAM 기법입니다.)
7. Strategies to integrate IMU and LiDAR SLAM for indoor mapping, accessed July 19, 2025, https://research.utwente.nl/en/publications/strategies-to-integrate-imu-and-lidar-slam-for-indoor-mapping
8. [2302.14298] LIWO: Lidar-Inertial-Wheel Odometry - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2302.14298
9. Improving SLAM Techniques with Integrated Multi-Sensor Fusion for 3D Reconstruction - PMC - PubMed Central, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11014387/
10. [Isaac Sim + ROS2] LIO-SAM (LiDAR Inertial Odometry SLAM) 구현, accessed July 19, 2025, https://786studio.tistory.com/6
11. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - MIT Senseable City Lab, accessed July 19, 2025, https://senseable.mit.edu/papers/pdf/20201020_Shan-etal_LIO-SAM_IROS.pdf
12. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and ..., accessed July 19, 2025, https://arxiv.org/abs/2007.00258
13. [2010.08196] FAST-LIO: A Fast, Robust LiDAR-inertial Odometry Package by Tightly-Coupled Iterated Kalman Filter - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2010.08196
14. [2407.02786] LiDAR-Inertial Odometry Based on Extended Kalman Filter - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2407.02786
15. iSAM: Fast Incremental Smoothing and Mapping with Efficient Data Association - Frank Dellaert, accessed July 19, 2025, https://dellaert.github.io/files/Kaess07icra.pdf
16. Real-time Smoothing and Mapping - Ananth Ranganathan, accessed July 19, 2025, http://www.ananth.in/Projects/Entries/2010/12/7_Real-time_Smoothing_and_Mapping.html
17. 라이다를 활용한 SLAM (LOAM, LeGO-LOAM, HDL GRAPH SLAM, LIO-SAM) - 자료실, accessed July 19, 2025, [http://www.lumisol.co.kr/sub/reference/lidar.asp?mode=view&bid=4&s_type=&s_keyword=&s_cate=&idx=212&page=1](http://www.lumisol.co.kr/sub/reference/lidar.asp?mode=view&bid=4&s_type&s_keyword&s_cate&idx=212&page=1)
18. Introduction to LiDAR SLAM: LOAM and LeGO-LOAM Paper and Code Explanation with ROS 2 Implementation - LearnOpenCV, accessed July 19, 2025, https://learnopencv.com/lidar-slam-with-ros2/
19. LOAM: Lidar Odometry and Mapping in Real-time - Carnegie Mellon ..., accessed July 19, 2025, https://www.ri.cmu.edu/pub_files/2014/7/Ji_LidarMapping_RSS2014_v8.pdf
20. 자율주행 로봇을 위한 라이다 기반 - Odometry 및 맵핑 기술 리뷰, accessed July 19, 2025, [https://icros.org/Newsletter/202304/data/5.%EB%A1%9C%EB%B4%87%EA%B8%B0%EC%88%A0%EB%A6%AC%EB%B7%B0.pdf](https://icros.org/Newsletter/202304/data/5.로봇기술리뷰.pdf)
21. An Improved Simultaneous Localization and Mapping Method Fusing LeGO-LOAM and Scan Context for Underground Coalmine - PMC - PubMed Central, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8778426/
22. LeGO-LOAM - OpenCV, accessed July 19, 2025, https://opencv.org/blog/lego-loam/
23. LeGO-LOAM-FN: An Improved Simultaneous Localization and Mapping Method Fusing LeGO-LOAM, Faster_GICP and NDT in Complex Orchard Environments - MDPI, accessed July 19, 2025, https://www.mdpi.com/1424-8220/24/2/551
24. LOAM | Wiki, accessed July 19, 2025, https://wiki.hanzheteng.com/algorithm/slam/loam
25. LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping - GitHub, accessed July 19, 2025, https://github.com/TixiaoShan/LIO-SAM
26. Performance Analysis of LIO-SAM - Ronia Peterson, accessed July 19, 2025, http://www.roniapeterson.com/LIO-SAM.pdf
27. [SLAM] FAST-LIO2 논문 리뷰 (+ IKF, ikd-tree) - A L I D A - 티스토리, accessed July 19, 2025, https://alida.tistory.com/102
28. Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2302.04031
29. A Fast Dynamic Point Detection Method for LiDAR-Inertial Odometry in Driving Scenarios, accessed July 19, 2025, https://arxiv.org/html/2407.03590v1
30. SR-LIO: LiDAR-Inertial Odometry with Sweep Reconstruction - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2210.10424
31. FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/368361497_FR-LIO_Fast_and_Robust_Lidar-Inertial_Odometry_by_Tightly-Coupled_Iterated_Kalman_Smoother_and_Robocentric_Voxels
32. GDUT-Kyle/faster_lio_sam: FASTER-LIO-SAM: A SLAM ... - GitHub, accessed July 19, 2025, https://github.com/GDUT-Kyle/faster_lio_sam
33. FAST-LIO-SAM/.gitmodules at master - GitHub, accessed July 19, 2025, https://github.com/engcang/FAST-LIO-SAM/blob/master/.gitmodules
34. engcang/FAST-LIO-SAM-QN: A SLAM implementation ... - GitHub, accessed July 19, 2025, https://github.com/engcang/FAST-LIO-SAM-QN
35. engcang/FAST-LIO-SAM-SC-QN: A SLAM implementation combining FAST-LIO2 with pose graph optimization and loop closing based on ScanContext, Quatro, and Nano-GICP - GitHub, accessed July 19, 2025, https://github.com/engcang/FAST-LIO-SAM-SC-QN
36. FAST-LIO-SAM/LICENSE at master - GitHub, accessed July 19, 2025, https://github.com/engcang/FAST-LIO-SAM/blob/master/LICENSE
37. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association ..., accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11644182/
38. gisbi-kim/SC-LIO-SAM: LiDAR-inertial SLAM - GitHub, accessed July 19, 2025, https://github.com/gisbi-kim/SC-LIO-SAM
39. Visualization of trajectory comparison between A-LOAM, FAST-LIO, and... - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/figure/sualization-of-trajectory-comparison-between-A-LOAM-FAST-LIO-and-our-method-on-KITTI_fig5_378109927
40. Performance Assessment of Lidar Odometry Frameworks: A Case Study at the Australian Botanic Garden Mount Annan - arXiv, accessed July 19, 2025, https://arxiv.org/html/2411.16931v1
41. The KITTI Vision Benchmark Suite - Andreas Geiger, accessed July 19, 2025, https://www.cvlibs.net/datasets/kitti/
42. KITTI Odometry Benchmark - Andreas Geiger, accessed July 19, 2025, https://www.cvlibs.net/datasets/kitti/eval_odometry.php
43. APE of our method in the KITTI dataset: Sequence 9, FAST-LIO, and LIO-SAM., accessed July 19, 2025, https://www.researchgate.net/figure/APE-of-our-method-in-the-KITTI-dataset-Sequence-9-FAST-LIO-and-LIO-SAM_fig5_381087766
44. LIO-CSI: LiDAR inertial odometry with loop closure combined with semantic information, accessed July 19, 2025, https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0261053
45. LIO-SAM++: A Lidar-Inertial Semantic SLAM with Association Optimization and Keyframe Selection - MDPI, accessed July 19, 2025, https://www.mdpi.com/1424-8220/24/23/7546
46. Semantic Lidar-Inertial SLAM for Dynamic Scenes - MDPI, accessed July 19, 2025, https://www.mdpi.com/2076-3417/12/20/10497
47. LiDAR-Visual-Inertial SLAM Robust in Structurally and Visually Degenerate Environments, accessed July 19, 2025, https://www.robot.t.u-tokyo.ac.jp/~yamashita/paper/E/E487Final.pdf
48. A Review of Dynamic Object Filtering in SLAM Based on 3D LiDAR - MDPI, accessed July 19, 2025, https://www.mdpi.com/1424-8220/24/2/645
49. A Coarse-to-Fine LiDAR-Based SLAM with Dynamic Object Removal in Dense Urban Areas | PolyU, accessed July 19, 2025, [https://www.polyu.edu.hk/aae/ipn-lab/us/publications/Conference/A%20Coarse-to-Fine%20LiDAR-Based%20SLAM%20with%20Dynamic%20Object%20Removal%20in%20Dense%20Urban%20Areas.pdf](https://www.polyu.edu.hk/aae/ipn-lab/us/publications/Conference/A Coarse-to-Fine LiDAR-Based SLAM with Dynamic Object Removal in Dense Urban Areas.pdf)
50. A Coarse-to-Fine LiDAR-Based SLAM with Dynamic Object Removal in Dense Urban Areas, accessed July 19, 2025, https://www.ion.org/publications/abstract.cfm?articleID=18083
51. LiDAR-Based SLAM under Semantic Constraints in Dynamic Environments - MDPI, accessed July 19, 2025, https://www.mdpi.com/2072-4292/13/18/3651

LIO-SAM - Autoware Documentation, accessed July 19, 2025, https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/integrating-autoware/creating-maps/open-source-slam/lio-sam/




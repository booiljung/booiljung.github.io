# SLAM(동시적 위치추정 및 지도작성)의 계보학적 탐구
[SLAM 동향](./index.md)

확률론적 기반에서 공간 AI까지


본 보고서는 동시적 위치추정 및 지도작성(Simultaneous Localization and Mapping, SLAM) 기술에 대한 포괄적인 계보학적 탐구를 제공한다. 1980년대 확률론적 추정과 컴퓨터 비전의 기초 개념에서부터 2020년대의 최첨단 기술에 이르기까지 그 지적 혈통을 추적한다. 본 연구의 핵심은 SLAM의 진화 과정을 시각적으로 구조화한 Mermaid 다이어그램이다. 필터 기반 접근법과 최적화 기반 접근법 사이의 주요한 분기점을 분석하고, 이후 시각 SLAM(Visual SLAM, VSLAM) 및 LiDAR SLAM과 같은 센서별 하위 분야로의 분화를 따라가며, 각 분기를 정의하는 대표적인 알고리즘들을 분석한다. 마지막으로, 딥러닝, 시맨틱 이해, 다중 로봇 협업이 이 분야의 새로운 지평을 재정의하며 SLAM을 진정한 공간 AI(Spatial AI)로 이끌고 있는 현대적 통합을 탐구한다.



SLAM은 알려지지 않은 환경의 지도를 구축함과 동시에 그 안에서 에이전트의 위치를 추적하는 계산적 문제를 해결하는 기술로 정의된다.1 이 문제는 본질적으로 지도를 알아야 위치를 추정할 수 있고, 위치를 알아야 지도를 작성할 수 있다는 순환적 의존성 때문에 종종 "닭이 먼저냐, 달걀이 먼저냐"의 문제에 비유된다.4


SLAM의 탄생은 단일 발명품이 아니라, 서로 다른 분야의 필요가 수렴한 결과물이다. 로봇 공학계는 항법 중 발생하는 불확실성을 다룰 방법이 필요했고, 컴퓨터 비전 분야는 이미지로부터 3차원 구조를 복원하는 기하학적 문제를 풀고 있었다. SLAM은 이 두 분야의 교차점에서 탄생했으며, 로봇 공학의 확률론적 추정 기법을 컴퓨터 비전의 기하학적 인식 문제에 적용함으로써 새로운 지평을 열었다.


SLAM의 이론적 초석은 1986년 Smith, Self, Cheeseman이 발표한 논문 "로봇 공학에서의 불확실한 공간 관계 추정(Estimating Uncertain Spatial Relationships in Robotics)"에서 찾을 수 있다.5 이 논문은 공간적 관계와 그 불확실성을 평균과 공분산 행렬로 표현하는 '확률론적 지도(Stochastic Map)'라는 개념을 제시했다.8 이는 로봇의 위치와 환경 내 객체(랜드마크) 간의 상대적 관계를 확률적으로 모델링하고, 새로운 관측이 들어올 때마다 이 관계를 점진적으로 갱신하는 방법을 제안한 것으로, SLAM의 확률론적 상태 추정 문제의 기틀을 마련했다.10


이러한 아이디어는 1986년 샌프란시스코에서 열린 IEEE 로보틱스 및 자동화 컨퍼런스(IEEE Robotics and Automation Conference)에서 Durrant-Whyte, Cheeseman과 같은 핵심 연구자들 사이에서 처음 논의되며 구체화되었다.12 이 학회는 확률론적 SLAM 문제의 개념이 잉태된 장소로 평가받는다.


같은 시기, 컴퓨터 비전 커뮤니티에서는 SLAM과 밀접하게 관련된 두 가지 기술이 독자적으로 발전하고 있었다. 첫째는 여러 이미지로부터 카메라의 상대적 자세와 3차원 구조를 복원하는 SfM(Structure from Motion)으로, 이는 시각 SLAM의 오프라인 버전으로 간주될 수 있다. 둘째는 NASA의 화성 탐사 로버 프로그램에서 바퀴 미끄러짐 문제를 해결하기 위해 개발된 시각 주행 거리 측정(Visual Odometry, VO) 기술이다.12 이 두 기술은 2000년대 이전까지 로봇 공학의 SLAM 연구와는 거의 교류 없이 개별적으로 발전했다.12


1990년대에 들어 Hugh Durrant-Whyte와 John J. Leonard와 같은 연구자들은 SLAM 문제의 수학적 구조를 정립하는 데 중추적인 역할을 했다.3 마침내 1995년 국제 로봇 연구 심포지엄(International Symposium on Robotics Research)에서 문제의 구조와 수렴성이 공식적으로 발표되었고, 'SLAM'이라는 약어가 처음으로 명명되었다.12

SLAM 문제의 정의 자체가 그 해결책의 구조를 규정했다. SLAM은 수학적으로 로봇의 전체 경로와 지도를 포함하는 결합 사후 확률 분포, 즉 $p(\text{경로}, \text{지도} | \text{측정값}, \text{제어입력})$을 추정하는 문제로 공식화된다.2 이 공식은 필연적으로 시간이 지남에 따라 상태 벡터의 크기가 계속 증가하고, 그에 따라 공분산 행렬의 크기가 제곱으로 커지는 계산적 문제를 내포한다. 따라서 SLAM의 전체 역사는 이 근본적인 확률 방정식을 효율적으로 근사하거나 인수분해하는 방법을 찾기 위한 여정으로 볼 수 있다.


SLAM 계보의 첫 번째 주요 분기는 확률론적 SLAM 문제를 해결하기 위한 두 가지 지배적인 수학적 패러다임, 즉 필터링과 최적화로부터 발생했다. 이 분기는 단순히 다른 알고리즘을 넘어서, SLAM 문제를 바라보는 두 가지 근본적으로 다른 관점을 반영한다.


필터링 접근법은 SLAM 문제를 '온라인(online)' 문제로 간주한다. 즉, 매 순간의 측정값을 이용해 현재의 상태(로봇의 자세와 지도)를 순차적으로 추정하고, 과거의 정보는 현재 상태에 요약되어 반영(marginalization)된다.2


확장 칼만 필터(Extended Kalman Filter, EKF) 기반 SLAM은 이 문제에 대한 최초의 광범위한 해결책이었다.11 EKF-SLAM은 로봇의 자세와 모든 랜드마크의 위치를 포함하는 단일 상태 벡터를 유지하고, 이를 하나의 다변량 가우시안 분포(평균과 공분산)로 표현한다.19 새로운 움직임이나 관측이 발생하면, 칼만 필터의 예측 및 업데이트 단계를 통해 이 가우시안 분포를 재귀적으로 갱신한다.21

그러나 EKF-SLAM은 세 가지 치명적인 한계를 가지고 있었다.

1. **계산 복잡도**: 상태 벡터의 공분산 행렬 업데이트 비용이 랜드마크 수(n)에 대해 $O(n^2)$의 복잡도를 가져 대규모 환경에서는 실용적으로 사용하기 어렵다.11
2. **불일치성(Inconsistency)**: 비선형적인 로봇의 움직임과 관측 모델을 선형화하는 과정에서 발생하는 오차는 필터가 자신의 불확실성을 과소평가하게 만든다. 이로 인해 추정치가 실제 값에서 벗어나는 '불일치' 현상이 발생하며, 이는 필터의 발산으로 이어질 수 있다.23 이 문제는 단순히 파라미터 조정으로 해결할 수 없는 구조적인 결함이다.18
3. **데이터 연관(Data Association)의 취약성**: 단일 가우시안 분포에 의존하기 때문에, 관측된 랜드마크를 지도상의 랜드마크와 잘못 연관시키면 필터 전체가 쉽게 붕괴될 수 있다.11


FastSLAM은 EKF-SLAM의 한계를 극복하기 위한 직접적인 대안으로 등장했다. 핵심 혁신은 '라오-블랙웰화된 파티클 필터(Rao-Blackwellized Particle Filter)'를 사용하는 것이다.26 이 기법은 SLAM 문제를 두 부분으로 분해한다. 즉, 파티클 필터를 사용해 로봇의 경로(비선형적 움직임을 잘 표현)를 추정하고, 각 파티클은 해당 경로를 조건으로 하는 독립적인 저차원 EKF들을 이용해 개별 랜드마크의 위치를 추정한다.28 이 분해 덕분에 FastSLAM은 비선형 모델과 비가우시안 분포를 더 잘 다룰 수 있으며 31, 계산 복잡도를 랜드마크 수에 대해 선형 또는 로그 시간으로 줄일 수 있었다.28 그러나 파티클 수가 제한되어 있어 루프 폐쇄(loop closure)와 같은 상황에서 '파티클 고갈(particle deprivation)' 문제를 겪으며, 시간이 지남에 따라 여전히 낙관적인 불확실성 추정치를 생성할 수 있다.26


최적화 접근법은 SLAM을 '전체(full)' 문제로 재구성한다. 이는 로봇의 전체 경로와 지도를 한 번에 추정하는 일괄 처리(batch) 방식이다.2 이 패러다임의 대표주자가 GraphSLAM이다.


GraphSLAM은 전체 SLAM 문제를 비선형 최소제곱 최적화 문제로 변환한다.17 시스템은 그래프로 표현되는데, 여기서 노드(node)는 로봇의 자세나 랜드마크 위치를 나타내고, 엣지(edge)는 로봇의 움직임(주행 거리계)이나 센서 관측으로부터 파생된 제약(constraint)을 나타낸다.31 목표는 그래프 내 모든 제약 조건의 오차를 최소화하는 노드들의 구성을 찾는 것이다.32 이 접근법은 1997년 Lu와 Milios에 의해 처음 제안되었으나 17, 희소 행렬을 효율적으로 다루는 선형대수 기법이 발전한 후에야 널리 보급되었다. GraphSLAM의 핵심은 SLAM 문제의 기저에 있는 정보 행렬이 매우 희소(sparse)하다는 사실을 활용하여 계산 효율성을 확보하는 것이다.31


필터링과 최적화의 대립은 SLAM 커뮤니티가 어느 한쪽을 선택하는 것으로 끝나지 않았다. 대신, 두 패러다임의 장점을 결합한 하이브리드 모델이 등장하며 새로운 통합을 이루었다. 필터링은 실시간으로 현재 상태를 추정하는 데 강점이 있지만, 과거 정보를 요약하는 과정에서 정보 손실이 발생하고 선형화 오차가 누적된다. 반면, 최적화는 전체 데이터를 재선형화하여 더 정확하고 일관된 결과를 제공하지만, 계산 비용이 높다.24

이러한 통합의 정점은 '키프레임(keyframe)' 기반 SLAM의 등장이다. PTAM 36과 이를 계승한 ORB-SLAM 38과 같은 현대적인 SLAM 시스템은 두 패러다임의 장점을 모두 취한다.

1. **추적(Tracking)**: 새로운 프레임이 들어올 때마다 로봇의 자세를 빠르게 추정한다. 이는 필터의 예측 단계와 유사한 온라인 작업이다.
2. **지역 매핑(Local Mapping)**: 소수의 최근 키프레임들로 구성된 '슬라이딩 윈도우' 내에서 번들 조정(Bundle Adjustment, BA)과 같은 지역적 최적화를 수행한다. 이는 작은 규모의 '전체 SLAM' 문제를 실시간으로 푸는 것과 같다.
3. **루프 폐쇄(Loop Closure)**: 로봇이 이전에 방문했던 장소를 다시 인식하면, 전체 경로에 대한 대규모 포즈 그래프 최적화를 수행하여 누적된 오차를 전역적으로 보정한다. 이는 대규모 '전체 SLAM' 문제에 해당한다.

이 하이브리드 아키텍처는 온라인 방식의 실시간 효율성과 일괄 처리 방식의 전역적 정확성을 결합함으로써, 필터링과 최적화 사이의 오랜 논쟁에 대한 실질적인 해답을 제시했다. 이는 SLAM 기술 발전의 중요한 변곡점이었다.


2장에서 논의된 기초적인 수학적 패러다임들은 특정 센서의 장단점에 맞춰 특화되면서 여러 하위 분야로 분화했다. 그중 가장 대표적인 것이 카메라를 사용하는 시각 SLAM(VSLAM)과 레이저 스캐너를 사용하는 LiDAR SLAM이다.


VSLAM은 저렴한 비용과 풍부한 환경 정보 획득 능력 덕분에 널리 연구되었지만, 조명 변화에 민감하고 스케일 모호성(scale ambiguity)과 같은 고유한 문제점을 안고 있다.40 VSLAM은 크게 간접 방식과 직접 방식으로 나뉜다.


이 방식은 먼저 이미지에서 코너나 블롭(blob)과 같은 희소한 특징점(feature)을 추출하고, 이들의 기하학적 관계를 이용해 자세를 추정한다.42

- **MonoSLAM (Davison, 2003-2007)**: 최초의 실시간 단안(monocular) SLAM 시스템으로, VSLAM의 가능성을 처음으로 입증했다.12 EKF 백엔드를 사용하여 희소한 이미지 패치들을 추적했다.

- **PTAM (Klein & Murray, 2007)**: VSLAM의 혁명으로 평가받는 알고리즘이다. 핵심 혁신은 추적(tracking)과 매핑(mapping)을 병렬 스레드로 분리한 것이다.36 이 구조 덕분에 실시간 추적을 유지하면서도, 매핑 스레드에서는 계산 비용이 높은 번들 조정(BA)을 수행할 수 있었다. 또한, 중복을 줄이고 효율적인 최적화를 가능하게 하는 

  **키프레임** 개념을 도입하여 현대 VSLAM 시스템의 표준을 제시했다.37

- **ORB-SLAM (Mur-Artal et al., 2015-2017)**: 특징점 기반 VSLAM의 집대성으로 여겨진다.39 PTAM의 아이디어를 기반으로, 추적, 매핑, 재인식, 루프 폐쇄 등 모든 작업에 ORB 특징점을 일관되게 사용한다.44 또한, 지역 매핑을 위한 공가시성 그래프(covisibility graph)와 강건한 장소 인식 및 루프 폐쇄를 위한 'Bag-of-Words' 모델을 도입하여 완결성 높은 시스템을 구축했다.38 이후 ORB-SLAM2는 스테레오 및 RGB-D 카메라로 확장되었고 38, ORB-SLAM3는 다중 지도 기능과 긴밀한 시각-관성 통합을 추가했다.46


이 방식은 특징점을 추출하는 대신 픽셀의 밝기 값(intensity)을 직접 사용하여, 기하학적 재투영 오차가 아닌 광도 오차(photometric error)를 최소화한다.40 특징이 부족한 환경에서도 이미지의 더 많은 정보를 활용할 수 있는 장점이 있다.42

- **LSD-SLAM (Engel et al., 2014)**: 준밀집(semi-dense) 지도를 생성하는 선구적인 직접 방식 알고리즘이다.49 키프레임 간의 직접적인 이미지 정렬을 수행하며, 변환 행렬을 

  sim(3) 공간에서 다루어 단안 카메라의 스케일 변화(scale drift) 문제를 명시적으로 처리했다.49

- **DSO (Engel et al., 2017)**: 직접 희소 주행 거리 측정(Direct Sparse Odometry)의 약자로, 직접 방식을 한 단계 발전시켰다. 특징점 검출/기술 과정 없이, 이미지 전체에서 밝기 변화가 큰 픽셀들을 희소하지만 균일하게 샘플링하여 모든 파라미터(기하 구조 및 카메라 움직임)를 공동으로 최적화한다.52 또한, 노출 시간, 렌즈 비네팅 등을 고려한 엄밀한 광도 보정 모델을 도입하여 높은 정확도를 달성했다.53


- **단안(Monocular)**: 가장 저렴하고 작지만, 절대적인 지도 크기를 알 수 없는 스케일 모호성 문제가 있으며, 초기화를 위해 상당한 움직임이 필요하다.41
- **스테레오(Stereo)**: 두 카메라 사이의 기준선(baseline) 덕분에 삼각 측량을 통해 깊이를 직접 추정할 수 있어 스케일 모호성 문제를 해결한다.41
- **RGB-D**: 컬러 이미지와 함께 픽셀별 깊이 정보를 직접 제공하여, 지도 초기화와 데이터 연관 문제를 크게 단순화시킨다.56


LiDAR SLAM은 직접적이고 정확한 거리 측정값을 제공하는 LiDAR를 주 센서로 사용한다. 이는 조명 변화에 강건하지만, 카메라에 비해 의미론적으로 풍부한 정보를 얻기는 어렵다.57


- **GMapping**: 2D 레이저 기반 SLAM을 위한 매우 영향력 있고 널리 사용되는 오픈소스 알고리즘이다.58 FastSLAM과 유사하게 라오-블랙웰화된 파티클 필터를 사용하여 2D 점유 격자 지도(occupancy grid map)를 학습한다.60 주행 거리계와 레이저 스캔 데이터에 의존하는 것이 핵심 특징이다.60
- **Google Cartographer**: 더 현대적인 그래프 기반의 2D/3D SLAM 시스템이다.63 핵심 아키텍처는 SLAM을 **지역 SLAM(Local SLAM)**과 **전역 SLAM(Global SLAM)**으로 분리한 것이다. 지역 SLAM은 실시간으로 지역적으로 일관된 서브맵(submap)을 만들고, 전역 SLAM은 백그라운드에서 이 서브맵들 간의 루프 폐쇄를 찾아 포즈 그래프를 최적화함으로써 전역적 일관성을 보장한다.65 이 구조는 높은 효율성과 확장성을 제공한다.


- **LOAM (Zhang & Singh, 2014)**: 3D LiDAR SLAM의 기념비적인 알고리즘이다.68 PTAM의 병렬 처리와 유사하게, LOAM은 문제를 두 개의 병렬 컴포넌트로 나눈다. 하나는 고속으로 동작하며 속도를 추정하고 포인트 클라우드의 왜곡을 보정하는 주행 거리 측정(odometry) 알고리즘이고, 다른 하나는 저속으로 동작하며 정밀한 매칭과 등록을 수행하는 매핑(mapping) 알고리즘이다.68 이 분해 구조가 실시간 성능과 낮은 오차 누적의 비결이다.
- **LeGO-LOAM 및 후속 연구**: LOAM의 영향력 있는 변종인 LeGO-LOAM(Lightweight and Ground-Optimized)은 지면을 분할하여 제거하고 다른 포인트들을 클러스터링함으로써 계산 부하를 크게 줄여 효율성을 높였다.71 이로 인해 임베디드 시스템에 더 적합하게 되었으며, A-LOAM, LIO-SAM 등 다양한 LOAM 기반 방법론의 등장을 촉발했다.74

VSLAM과 LiDAR SLAM의 발전 경로는 서로 다른 센서에서 출발했음에도 불구하고, 고수준 아키텍처에서 놀라운 수렴 진화(convergent evolution)를 보여준다. 두 분야 모두 문제를 고주파의 실시간 추적/주행 거리 측정 요소와 저주파의 계산 집약적인 매핑/최적화 요소로 분리하는 것이 실시간 성능과 높은 정확도를 동시에 달성하는 핵심임을 독립적으로 발견했다. VSLAM에서는 PTAM이 36, LiDAR SLAM에서는 LOAM이 68, 그리고 그래프 기반 SLAM에서는 Cartographer가 65 이러한 분리 아키텍처를 채택했다. 이는 우연이 아니라, 제어에 필요한 낮은 지연 시간의 자세 업데이트와 전역적 일관성을 위한 고비용의 일괄 최적화라는 SLAM의 근본적인 긴장 관계에 대한 최적의 해결책이기 때문이다.


| 알고리즘         | 대표 논문                                  | 패러다임    | 주 센서             | 지도 표현                    | 핵심 혁신                                   | 주요 강점                                  | 주요 약점                                |
| ---------------- | ------------------------------------------ | ----------- | ------------------- | ---------------------------- | ------------------------------------------- | ------------------------------------------ | ---------------------------------------- |
| **EKF-SLAM**     | Smith et al. 1986; Dissanayake et al. 2001 | 필터 기반   | 레이저, 초음파      | 희소 특징점                  | 최초의 통합된 확률론적 SLAM 해법            | 이론적 기반 제공                           | O(n2) 복잡도, 불일치성, 데이터 연관 취약 |
| **FastSLAM**     | Montemerlo et al. 2002                     | 필터 기반   | 레이저, 비전        | 희소 특징점                  | Rao-Blackwellized 파티클 필터로 문제 분해   | 비선형/비가우시안에 강건, 계산 효율성 향상 | 파티클 고갈, 장기적 불일치성             |
| **GraphSLAM**    | Lu & Milios 1997; Thrun et al. 2006        | 최적화 기반 | 레이저, 비전        | 포즈 그래프, 특징점          | 전체 SLAM 문제를 그래프 최적화로 재구성     | 높은 정확도와 일관성, 전체 경로 최적화     | 일괄 처리로 인한 높은 초기 계산 비용     |
| **MonoSLAM**     | Davison et al. 2007                        | 필터 기반   | 단안 카메라         | 희소 특징점                  | 최초의 실시간 단안 VSLAM                    | 실시간 단안 SLAM의 가능성 입증             | EKF 한계 계승, 소규모 환경에 국한        |
| **PTAM**         | Klein & Murray, ISMAR 2007                 | 하이브리드  | 단안 카메라         | 희소 특징점                  | 병렬 추적/매핑, 키프레임, 실시간 BA         | 실시간 성능과 높은 정확도 동시 달성        | 스케일 모호성, 루프 폐쇄 기능 제한적     |
| **ORB-SLAM**     | Mur-Artal et al., TRO 2015                 | 하이브리드  | 단안/스테레오/RGB-D | 희소 특징점, 공가시성 그래프 | 특징점 기반 VSLAM의 완성형 프레임워크       | 높은 정확도, 강건한 루프 폐쇄 및 재인식    | 동적 환경 및 텍스처 부족 환경에 취약     |
| **LSD-SLAM**     | Engel et al., ECCV 2014                    | 하이브리드  | 단안 카메라         | 준밀집(Semi-Dense)           | 직접 방식, sim(3) 포즈 추정으로 스케일 보정 | 텍스처 부족 환경에 강건, 더 밀집된 지도    | 광도 변화에 민감, 계산 비용 높음         |
| **DSO**          | Engel et al., PAMI 2017                    | 하이브리드  | 단안 카메라         | 희소 포인트                  | 특징점 없는 희소 직접 방식, 광도 보정       | 높은 정확도, 특징점 검출 불필요            | 초기화 어려움, 순수 회전에 취약          |
| **GMapping**     | Grisetti et al. 2007                       | 필터 기반   | 2D LiDAR            | 점유 격자 지도               | RBPF 기반의 효율적인 2D 그리드 매핑         | ROS 표준, 낮은 계산 자원, 강건성           | 3D 확장 불가, 오도메트리 의존성 높음     |
| **Cartographer** | Hess et al., ICRA 2016                     | 최적화 기반 | 2D/3D LiDAR, IMU    | 서브맵, 포즈 그래프          | 지역/전역 SLAM 분리 아키텍처                | 높은 확장성, 실시간 루프 폐쇄, 저지연      | 파라미터 튜닝 복잡, 높은 자원 요구       |
| **LOAM**         | Zhang & Singh, RSS 2014                    | 하이브리드  | 3D LiDAR            | 포인트 클라우드              | 고속/저속 병렬 오도메트리/매핑 분리         | 낮은 오차 누적, 실시간 3D 매핑             | 루프 폐쇄 부재 (초기 버전)               |
| **LeGO-LOAM**    | Shan & Englot, IROS 2018                   | 하이브리드  | 3D LiDAR            | 포인트 클라우드              | 지면 분할을 통한 경량화 및 최적화           | 저사양 하드웨어에서 실시간 성능            | 지면 존재 가정, 특정 LiDAR에 최적화      |


고전적인 기하학 기반 SLAM 파이프라인은 이제 새로운 기술들과 융합하며 그 한계를 넘어서고 있다. 현대 SLAM 연구는 단순히 '어디에 있는가'를 넘어 '무엇이 있는가', '어떻게 움직이는가'를 이해하는 방향으로 진화하고 있다.


대부분의 고전 SLAM 알고리즘은 환경이 정적이라는 근본적인 가정을 전제로 한다.47 그러나 현실 세계는 사람, 차량 등 움직이는 객체로 가득 차 있다. 이러한 동적 객체들은 SLAM 시스템에 잘못된 제약 조건을 추가하여 추적 실패를 유발한다.46 이 문제를 해결하기 위한 접근법은 크게 두 가지로 나뉜다.

- **기하학적 방법**: 연속된 프레임 간의 에피폴라 기하(epipolar geometry)나 RANSAC과 같은 방법을 사용하여 정적인 배경과 움직이는 객체를 분리한다.46
- **학습 기반 방법**: YOLO와 같은 객체 탐지나 시맨틱 분할(semantic segmentation) 딥러닝 모델을 사용하여 동적일 가능성이 높은 객체(예: 사람, 자동차)를 사전에 식별하고 마스킹하여 계산에서 제외한다.46

이러한 발전은 **시맨틱 SLAM**으로 이어진다. 시맨틱 SLAM은 기하학적 지도("벽이 어디에 있는가?")에서 한 걸음 더 나아가, 의미론적 지도("이 객체는 의자이고, 어디에 있는가?")를 구축하는 것을 목표로 한다. 딥러닝 기반 객체 탐지 및 분할 기술을 SLAM 파이프라인에 통합함으로써, 지도를 점이나 선이 아닌 '의자', '문', '책상'과 같은 의미 있는 객체 레이블로 채울 수 있다.78 이는 로봇이 환경을 더 깊이 이해하고, 더 복잡한 작업을 수행할 수 있게 하는 핵심 기술이다.


딥러닝은 SLAM 분야에 혁명적인 변화를 가져왔으며, 그 통합 수준에 따라 세 가지 단계로 분류할 수 있다.80

1. **모듈 강화형 접근**: 이 방식은 전통적인 SLAM 파이프라인의 특정 모듈을 딥러닝으로 대체하거나 성능을 향상시키는 것이다. 예를 들어, CNN을 사용하여 더 강건한 특징점을 추출하고 기술하거나 81, 단일 이미지로부터 깊이를 예측하거나(CNN-SLAM) 16, 더 신뢰성 있는 루프 폐쇄를 수행하는 데 사용된다.83
2. **End-to-End 학습**: 이 접근은 전통적인 SLAM 또는 VO 파이프라인 전체를 하나의 거대한 신경망으로 대체하려는 시도이다.81 예를 들어, DeepVO는 RNN을 사용하여 이미지 시퀀스로부터 직접 카메라 자세를 추정한다.81 이러한 시스템은 지도 데이터나 자세 레이블을 이용한 지도/비지도 학습을 통해 훈련되지만, 종종 새로운 환경에 대한 일반화 성능이 부족하고 전통적인 기하학 기반 방법의 엄밀함과 장기적인 일관성을 따라가지 못하는 한계가 있다.84
3. **암시적 신경 표현 (Implicit Neural Representation)**: 가장 최신의 연구 방향으로, NeRF-SLAM이 대표적이다. 이 방식은 점, 선, 격자 등 명시적인 지도 표현 대신, 환경 전체를 신경망의 가중치로 표현한다.85 예를 들어, NeRF(Neural Radiance Field)는 3차원 좌표와 시선 방향을 입력받아 해당 지점의 색상과 밀도를 출력하는 연속적인 함수를 학습한다. SLAM은 이 학습된 암시적 표현에 대해 카메라 자세를 최적화하는 방식으로 수행된다. iMAP, NICE-SLAM 등이 이 계열에 속하며, 조밀하고 사실적인 3차원 지도 생성을 가능하게 한다.85

이러한 발전은 SLAM의 목표가 순수한 기하학적 문제 해결에서 종합적인 '장면 이해(scene understanding)' 문제로 전환되고 있음을 시사한다. 전통적인 SLAM이 많은 응용 분야에서 "해결된 문제"로 간주되면서 86, 연구의 최전선은 환경에 대한 맥락과 의미를 부여하는 방향으로 이동하고 있다. 동적 SLAM, 시맨틱 SLAM, 딥러닝 기반 방법들은 단순한 점진적 개선이 아니라, SLAM의 목표 자체를 기하학적 모델 생성에서 포괄적이고 지능적인 '월드 모델(world model)' 생성으로 바꾸고 있다.


C-SLAM(Collaborative SLAM)은 여러 로봇을 사용하여 SLAM을 수행하는 기술이다. 이는 넓은 지역을 더 빠르게 매핑하고, 단일 로봇의 실패에 대한 강건성을 높이며, 다양한 시점의 정보를 융합하여 더 정확한 지도를 생성하는 장점을 가진다.5

C-SLAM 아키텍처는 주로 두 가지로 나뉜다.

- **중앙 집중식**: 모든 로봇이 자신의 데이터나 서브맵을 중앙 서버로 전송하면, 서버가 이를 융합하고 전역 최적화를 수행한다.87
- **분산형**: 로봇들이 P2P(peer-to-peer) 방식으로 통신하며, 각자 이웃 로봇의 정보를 바탕으로 자신의 지도를 유지하고 최적화한다.87

C-SLAM의 핵심 과제는 로봇들이 서로 같은 장소를 보고 있다는 것을 어떻게 인식하여 지도 간의 제약 조건을 설정할 것인가(장소 인식/랑데부), 각기 다른 좌표계의 지도를 어떻게 하나의 일관된 지도로 병합할 것인가(지도 융합), 그리고 제한된 대역폭의 통신 환경을 어떻게 효율적으로 사용할 것인가 등이다.88

5장: 미래 궤적: 지속적인 과제와 공간 AI로의 길


SLAM 기술은 눈부신 발전을 이루었지만, 여전히 해결해야 할 주요 과제들이 남아있다.

- **장기 자율성(Lifelong SLAM)**: 로봇이 수 주, 수 개월, 또는 수 년 동안 안정적으로 작동하면서 계절 변화나 공사와 같은 환경 변화에 적응하고, 지도가 무한정 커지거나 불일치해지지 않도록 하는 방법은 여전히 중요한 연구 주제이다.39
- **강건성과 실패 복구**: 센서 고장, 급격한 움직임, 조명이 어둡거나 텍스처가 없는 열악한 환경, 악천후 등에서도 안정적으로 작동하는 강건한 SLAM 시스템 개발이 필요하다.90
- **확장성과 효율성**: 저전력 임베디드 장치에서도 복잡한 SLAM 알고리즘을 실행할 수 있도록 계산 및 메모리 사용량을 줄이는 것이 중요하다.89


본 보고서에서 추적한 SLAM의 계보학적 진화는 궁극적으로 **공간 AI(Spatial AI)**라는 새로운 목표를 향하고 있다.92 공간 AI는 단순한 기하학적 매핑을 넘어, 기하학, 시맨틱, 객체 간의 관계, 물리 법칙, 그리고 행동 유도성(affordance)까지 포함하는 공간에 대한 총체적인 이해를 추구하는 시스템으로 정의할 수 있다.

미래의 SLAM 시스템은 단순히 지도를 생성하는 것을 넘어, 로봇이 복잡한 추론과 작업을 수행할 수 있도록 하는 예측 가능하고 상호작용적인 월드 모델을 구축할 것이다. 고전적인 기하학적 방법론과 현대적인 딥러닝 및 시맨틱 이해 기술의 융합이 이러한 변화를 이끄는 원동력이다.89 결국, SLAM 계보의 미래 분기점은 수학적 공식의 차이가 아니라, 매핑 과정에 얼마나 깊은 '이해'를 담아낼 수 있는지에 따라 정의될 것이다. 카메라는 단순한 센서를 넘어, 유연하고 대규모의 상황 인식을 가능하게 하는 "공간 AI 센서"로 변모하고 있다.92


1. Simultaneous localization and mapping - Wikipedia, accessed July 19, 2025, https://en.wikipedia.org/wiki/Simultaneous_localization_and_mapping
2. 46. Simultaneous Localization and Mapping - CS@Columbia, accessed July 19, 2025, https://www.cs.columbia.edu/~allen/F19/NOTES/slam_paper.pdf
3. SLAM explained | aijobs.net, accessed July 19, 2025, https://aijobs.net/insights/slam-explained/
4. Simultaneous localization and mapping (SLAM) | Robotics and Bioinspired Systems Class Notes | Fiveable, accessed July 19, 2025, https://library.fiveable.me/robotics-bioinspired-systems/unit-11/simultaneous-localization-mapping-slam/study-guide/PtQW9FKUtK8FAuiA
5. Overview of Multi-Robot Collaborative SLAM from the Perspective of Data Fusion - MDPI, accessed July 19, 2025, https://www.mdpi.com/2075-1702/11/6/653
6. Estimating Uncertain Spatial Relationships in Robotics 1 Introduction - MIT, accessed July 19, 2025, https://web.mit.edu/2.166/www/handouts/smith90stochastic.pdf
7. [1304.3111] Estimating Uncertain Spatial Relationships in Robotics - arXiv, accessed July 19, 2025, https://arxiv.org/abs/1304.3111
8. Estimating Uncertain Spatial Relationships in Robotics - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/301849501_Estimating_Uncertain_Spatial_Relationships_in_Robotics
9. I I I I I I I I I I I I I I I I I I I - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/1304.3111
10. Estimating Uncertain Spatial Relationships in Robotics - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/221405213_Estimating_Uncertain_Spatial_Relationships_in_Robotics
11. Simultaneous Localization and Mapping with Unknown Data Association Using FastSLAM - Sebastian Thrun, accessed July 19, 2025, http://robots.stanford.edu/papers/montemerlo.fastslam-dataassoc03.pdf
12. Brief Review on Visual SLAM: A Historical Perspective - Fan's Farm, accessed July 19, 2025, https://fzheng.me/2016/05/30/slam-review/
13. Simultaneous Localisation and Mapping (SLAM): Part I The Essential Algorithms - People @EECS, accessed July 19, 2025, https://people.eecs.berkeley.edu/~pabbeel/cs287-fa09/readings/Durrant-Whyte_Bailey_SLAM-tutorial-I.pdf
14. Hugh F. Durrant-Whyte - Wikipedia, accessed July 19, 2025, https://en.wikipedia.org/wiki/Hugh_F._Durrant-Whyte
15. John J. Leonard - Wikipedia, accessed July 19, 2025, https://en.wikipedia.org/wiki/John_J._Leonard
16. The Ultimate SLAM Guide - Number Analytics, accessed July 19, 2025, https://www.numberanalytics.com/blog/ultimate-guide-to-slam
17. A Tutorial on Graph-Based SLAM, accessed July 19, 2025, http://www2.informatik.uni-freiburg.de/~stachnis/pdf/grisetti10titsmag.pdf
18. Robocentric map joining: Improving the consistency of EKF-SLAM - CiteSeerX, accessed July 19, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=6e372bf6ca97cf6884257b22ff6181ff661ed211
19. EKF SLAM - Jihong Ju, accessed July 19, 2025, https://jihongju.github.io/2018/10/05/slam-05-ekf-slam/
20. EKF-SLAM A Very Quick Guide - IRI UPC, accessed July 19, 2025, [https://www.iri.upc.edu/people/jsola/JoanSola/objectes/curs_SLAM/SLAM2D/SLAM%20course.pdf](https://www.iri.upc.edu/people/jsola/JoanSola/objectes/curs_SLAM/SLAM2D/SLAM course.pdf)
21. EKF SLAM updates in O(n) with Divide and Conquer SLAM - CiteSeerX, accessed July 19, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=b8bddc8c3e320856b74147e228697859935d5a92
22. ekfSLAM - Perform simultaneous localization and mapping using extended Kalman filter - MATLAB - MathWorks, accessed July 19, 2025, https://www.mathworks.com/help/nav/ref/ekfslam.html
23. Consistency of the EKF-SLAM Algorithm - The University of Sydney, accessed July 19, 2025, https://www-personal.acfr.usyd.edu.au/tbailey/papers/ekfslam.pdf
24. Comparison of EKF based SLAM and optimization based SLAM algorithms - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/326053198_Comparison_of_EKF_based_SLAM_and_optimization_based_SLAM_algorithms
25. Comparison of EKF based SLAM and optimization based SLAM algorithms - SciSpace, accessed July 19, 2025, https://scispace.com/pdf/comparison-of-ekf-based-slam-and-optimization-based-slam-qes9i467fx.pdf
26. SLAM, Fast SLAM, and Rao-Blackwellization 1 About SLAM 2 Basic Approaches to SLAM, accessed July 19, 2025, https://www.cs.cmu.edu/~16831-f12/notes/F09/lec24/16831_lecture24.bwagenkn.pdf
27. rbpf-slam-tutorial-2007.pdf, accessed July 19, 2025, http://www2.informatik.uni-freiburg.de/~stachnis/pdf/rbpf-slam-tutorial-2007.pdf
28. FastSLAM: A Factored Solution to the Simultaneous Localization and Mapping Problem - Washington, accessed July 19, 2025, https://courses.cs.washington.edu/courses/cse590a/02sp/fastslam.pdf
29. FastSLAM: A Factored Solution to the Simultaneous Localization and Mapping Problem | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/224773156_FastSLAM_A_Factored_Solution_to_the_Simultaneous_Localization_and_Mapping_Problem
30. Consistency of the FastSLAM Algorithm - CiteSeerX, accessed July 19, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=998802f52b95b0f697805371ba4178dd90f84181
31. The Types of SLAM Algorithms - Medium, accessed July 19, 2025, https://medium.com/@nahmed3536/the-types-of-slam-algorithms-356196937e3d
32. (PDF) A tutorial on graph-based SLAM - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/231575337_A_tutorial_on_graph-based_SLAM
33. The GraphSLAM Algorithm with Applications to Large-Scale Mapping of Urban Structures - Sebastian Thrun, accessed July 19, 2025, http://robots.stanford.edu/papers/thrun.graphslam.pdf
34. Graph SLAM: From Theory to Implementation - Federico Sarrocco, accessed July 19, 2025, https://federicosarrocco.com/blog/graph-slam-tutorial
35. A brief introduction to GraphSLAM | by Shiva Chandrachary - Medium, accessed July 19, 2025, https://shivachandrachary.medium.com/a-brief-introduction-to-graphslam-4204b4fce2f0
36. PTAM (Parallel Tracking and Mapping) re-released under GPLv3. - GitHub, accessed July 19, 2025, https://github.com/Oxford-PTAM/PTAM-GPL
37. Parallel Tracking and Mapping for Small AR Workspaces - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/4334429_Parallel_Tracking_and_Mapping_for_Small_AR_Workspaces
38. ORB-SLAM Project Webpage, accessed July 19, 2025, https://webdiis.unizar.es/~raulmur/orbslam/
39. (PDF) ORB-SLAM: a versatile and accurate monocular SLAM system - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/271823237_ORB-SLAM_a_versatile_and_accurate_monocular_SLAM_system
40. Visual SLAM: What Are the Current Trends and What to Expect? - PMC, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9735432/
41. Monocular vs Stereo vs Monochrome Camera | by Abhishek Jain - Medium, accessed July 19, 2025, https://medium.com/@abhishekjainindore24/monocular-vs-stereo-vs-monochrome-camera-16612bf4358b
42. Feature-based or Direct: An Evaluation of Monocular Visual Odometry - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/316875491_Feature-based_or_Direct_An_Evaluation_of_Monocular_Visual_Odometry
43. A Comprehensive Survey of Visual SLAM Algorithms - MDPI, accessed July 19, 2025, https://www.mdpi.com/2218-6581/11/1/24
44. ORB-SLAM: Tracking and Mapping Recognizable Features - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/263746979_ORB-SLAM_Tracking_and_Mapping_Recognizable_Features
45. ORB-SLAM: A Versatile and Accurate Monocular SLAM System - SciSpace, accessed July 19, 2025, https://scispace.com/papers/orb-slam-a-versatile-and-accurate-monocular-slam-system-t5f1c0a74e
46. Optimization of visual SLAM in dynamic environments using object detection - SPIE Digital Library, accessed July 19, 2025, https://www.spiedigitallibrary.org/conference-proceedings-of-spie/13486/134860J/Optimization-of-visual-SLAM-in-dynamic-environments-using-object-detection/10.1117/12.3055716.full
47. PLDS-SLAM: Point and Line Features SLAM in Dynamic Environment - MDPI, accessed July 19, 2025, https://www.mdpi.com/2072-4292/15/7/1893
48. Combining Feature-based and Direct Methods for Semi-dense Real-time Stereo Visual Odometry - Autonomous Intelligent Systems, accessed July 19, 2025, https://www.ais.uni-bonn.de/papers/IAS-14_2016_Krombach.pdf
49. LSD-SLAM: Large-Scale Direct Monocular SLAM - Jakob Engel, accessed July 19, 2025, https://jakobengel.github.io/pdf/engel14eccv.pdf
50. LSD-SLAM: large-scale direct monocular SLAM | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/319770169_LSD-SLAM_large-scale_direct_monocular_SLAM
51. tum-vision/lsd_slam: LSD-SLAM - GitHub, accessed July 19, 2025, https://github.com/tum-vision/lsd_slam
52. Understanding how Direct Visual SLAM works | Kudan Global, accessed July 19, 2025, https://www.kudan.io/blog/direct-visual-slam/
53. Direct Sparse Odometry - Jakob Engel, accessed July 19, 2025, https://jakobengel.github.io/pdf/DSO.pdf
54. JakobEngel/dso: Direct Sparse Odometry - GitHub, accessed July 19, 2025, https://github.com/JakobEngel/dso
55. Visual SLAM - DSO: Direct Sparse Odometry - Computer Vision Group, accessed July 19, 2025, https://cvg.cit.tum.de/research/vslam/dso
56. Visual SLAM with RGB-D Camera - MATLAB & Simulink - MathWorks, accessed July 19, 2025, https://www.mathworks.com/help/vision/ug/visual-slam-with-an-rgbd-camera.html
57. DV-LOAM: Direct Visual LiDAR Odometry and Mapping - MDPI, accessed July 19, 2025, https://www.mdpi.com/2072-4292/13/16/3340
58. How to deploy Gmapping SLAM algorithms in mobile robots, accessed July 19, 2025, https://global.agilex.ai/blogs/news/how-to-deploy-gmapping-slam-algorithms-in-mobile-robots
59. Comparison of Various SLAM Systems for Mobile Robot in an Indoor Environment, accessed July 19, 2025, http://vigir.missouri.edu/~gdesouza/Research/MobileRobotics/Comparison_of_Various_SLAM_RG.pdf
60. GMapping | turtlebot2-tutorials - GitHub Pages, accessed July 19, 2025, https://dabit-industries.github.io/turtlebot2-tutorials/06-Gmapping.html
61. SLAM | Husarion, accessed July 19, 2025, https://husarion.com/tutorials/ros-tutorials/8-slam/
62. stdr_simulator/Tutorials/Create a map with gmapping - ROS Wiki, accessed July 19, 2025, [http://wiki.ros.org/stdr_simulator/Tutorials/Create%20a%20map%20with%20gmapping](http://wiki.ros.org/stdr_simulator/Tutorials/Create a map with gmapping)
63. Cartographer SLAM Method for Optimization with an Adaptive Multi-Distance Scan Scheduler - MDPI, accessed July 19, 2025, https://www.mdpi.com/2076-3417/10/1/347
64. Cartographer - Cartographer documentation, accessed July 19, 2025, https://google-cartographer.readthedocs.io/
65. Building Maps Using Google Cartographer and the OS1 Lidar Sensor | Ouster, accessed July 19, 2025, https://ouster.com/insights/blog/building-maps-using-google-cartographer-and-the-os1-lidar-sensor
66. IMPROVING GOOGLE'S CARTOGRAPHER 3D MAPPING BY CONTINUOUS-TIME SLAM, accessed July 19, 2025, https://isprs-archives.copernicus.org/articles/XLII-2-W3/543/2017/isprs-archives-XLII-2-W3-543-2017.pdf
67. Real-Time Loop Closure in 2D LIDAR SLAM - Google Research, accessed July 19, 2025, https://research.google.com/pubs/archive/45466.pdf
68. LOAM: Lidar Odometry and Mapping in Real-time, accessed July 19, 2025, https://www.ri.cmu.edu/publications/loam-lidar-odometry-and-mapping-in-real-time/
69. (PDF) LOAM : Lidar Odometry and Mapping in real-time - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/311570125_LOAM_Lidar_Odometry_and_Mapping_in_real-time
70. LOAM: Lidar Odometry and Mapping in Real-time - Carnegie Mellon University's Robotics Institute, accessed July 19, 2025, https://www.ri.cmu.edu/pub_files/2014/7/Ji_LidarMapping_RSS2014_v8.pdf
71. LeGO-LOAM: Lightweight and Ground-Optimized Lidar Odometry and Mapping on Variable Terrain - OpenCV, accessed July 19, 2025, https://opencv.org/blog/lego-loam/
72. LeGO-LOAM-FN: An Improved Simultaneous Localization and Mapping Method Fusing LeGO-LOAM, Faster_GICP and NDT in Complex Orchard Environments - MDPI, accessed July 19, 2025, https://www.mdpi.com/1424-8220/24/2/551
73. An Improved Simultaneous Localization and Mapping Method Fusing LeGO-LOAM and Scan Context for Underground Coalmine - PMC - PubMed Central, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8778426/
74. gogojjh/M-LOAM: Robust Odometry and Mapping for Multi-LiDAR Systems with Online Extrinsic Calibration - GitHub, accessed July 19, 2025, https://github.com/gogojjh/M-LOAM
75. LeGO-LOAM-BOR: optimizer Lidar Odometry and Mapping - GitHub, accessed July 19, 2025, https://github.com/koide3/LeGO-LOAM-BOR
76. Review of Visual SLAM in Dynamic Environment - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/363734777_Review_of_Visual_SLAM_in_Dynamic_Environment
77. Robust Visual SLAM in Dynamic Environment Based on Motion Detection and Segmentation | J. Auton. Veh. Sys. | ASME Digital Collection, accessed July 19, 2025, https://asmedigitalcollection.asme.org/autonomousvehicles/article/4/1/011001/1201322/Robust-Visual-SLAM-in-Dynamic-Environment-Based-on
78. Semantic Visual Simultaneous Localization and Mapping: A Survey - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2209.06428
79. SLAM-past-present-future.pdf - cs.wisc.edu, accessed July 19, 2025, https://pages.cs.wisc.edu/~jphanna/teaching/25spring_cs639/resources/SLAM-past-present-future.pdf
80. A Survey of Visual SLAM Based on RGB-D Images Using Deep Learning and Comparative Study for VOE - MDPI, accessed July 19, 2025, https://www.mdpi.com/1999-4893/18/7/394
81. Mastering Visual SLAM - Number Analytics, accessed July 19, 2025, https://www.numberanalytics.com/blog/mastering-visual-slam-techniques
82. Deep Learning for Visual SLAM: The State-of-the-Art and Future Trends - MDPI, accessed July 19, 2025, https://www.mdpi.com/2079-9292/12/9/2006
83. A Survey on Visual SLAM based on Deep Learning, accessed July 19, 2025, https://www.china-simulation.com/EN/10.16182/j.issn1004731x.joss.19-VR0466
84. A robust visual-inertial SLAM based deep feature extraction and matching - arXiv, accessed July 19, 2025, https://arxiv.org/html/2405.03413v2
85. 3D-Vision-World/awesome-NeRF-and-3DGS-SLAM - GitHub, accessed July 19, 2025, https://github.com/3D-Vision-World/awesome-NeRF-and-3DGS-SLAM
86. A Multisensor Data Fusion Approach for Simultaneous Localization and Mapping* - Zhekai Jin, accessed July 19, 2025, https://zhekaijin.github.io/docs/SLAM_IEEEITSC2019_FINAL.pdf
87. Collaborative Visual SLAM Framework for a Multi-Robot System, accessed July 19, 2025, https://www-sop.inria.fr/members/Philippe.Martinet/publis/2015/PPNIV15Nived.pdf
88. Towards Collaborative Simultaneous Localization and Mapping: a Survey of the Current Research Landscape - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2108.08325
89. The Future of Robotics with SLAM - Number Analytics, accessed July 19, 2025, https://www.numberanalytics.com/blog/future-of-robotics-with-slam
90. SLAM Algorithms: Unraveling the Foundations and Applications in Mobile Robotics, accessed July 19, 2025, https://morpheustek.com/slam-algorithms-unraveling-the-foundations-and-applications-in-mobile-robotics/
91. Unlocking the Future of Autonomy: How Simultaneous Localization and Mapping (SLAM) Powers Next-Gen Robotics, accessed July 19, 2025, https://www.dronesplusrobotics.com/post/unlocking-the-future-of-autonomy-how-simultaneous-localization-and-mapping-slam-powers-next-gen-r
92. Visual SLAM in the era of Deep Learning | AI Talks - YouTube, accessed July 19, 2025, https://www.youtube.com/watch?v=_UMNcNnyLiY


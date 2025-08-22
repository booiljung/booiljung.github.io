---
layout: page
title: GTSAM(Georgia Tech Smoothing and Mapping)
permalink: /sensors/filters smoothing/GTSAM Georgia Tech Smoothing and Mapping
---



로보틱스와 컴퓨터 비전 분야의 핵심적인 도전 과제 중 하나는 불확실하고 노이즈가 섞인 센서 데이터를 바탕으로 시스템의 잠재적인 상태 변수(latent variable)를 추정하는 것이다.1 여기서 상태 변수란 로봇의 위치 및 자세(pose), 주변 환경을 구성하는 랜드마크의 위치, 혹은 로봇의 속도와 같은 직접 관측할 수 없는 물리량을 의미한다. 이 문제는 본질적으로 확률적 추론 문제로 귀결되며, 주어진 측정값들을 바탕으로 가장 가능성이 높은 상태를 찾아내는 것이 최종 목표이다.5

역사적으로 이러한 상태 추정 문제는 주로 필터링(filtering) 기반의 접근법을 통해 해결되어 왔다. 특히 칼만 필터(Kalman Filter)와 그 비선형 시스템 확장판인 확장 칼만 필터(Extended Kalman Filter, EKF), 무향 칼만 필터(Unscented Kalman Filter, UKF) 등은 수십 년간 로보틱스 분야의 표준적인 해법으로 자리매김했다.3 필터링 방식은 현재 시점까지의 측정값만을 이용하여 현재 상태를 재귀적으로 갱신하기 때문에, 실시간 처리에 유리하다는 장점이 있다.


그러나 시스템의 비선형성이 강해지거나, 과거의 모든 정보를 활용하여 전역적인 일관성을 확보해야 하는 문제에서는 필터링 기반 접근법의 한계가 드러나기 시작했다. 필터링은 과거 상태를 한 번 추정하고 나면 다시는 되돌아보지 않고 주변화(marginalize out)하기 때문에, 비선형 시스템에서 발생하는 선형화 오류가 누적되어 추정치의 정확도를 저하시킬 수 있다.8

이러한 한계를 극복하기 위해 등장한 것이 바로 최적화(optimization) 기반의 접근법, 특히 스무딩 및 매핑(Smoothing and Mapping, SAM)이다.10 스무딩은 필터링과 달리, 특정 시점의 상태를 추정하기 위해 과거와 현재는 물론 미래의 측정값까지 포함한 전체 데이터셋을 고려한다.9 이는 전체 궤적(trajectory)과 맵을 동시에 최적화하는 배치(batch) 또는 전체 스무딩(full smoothing) 문제로 귀결된다. 이 접근법은 모든 정보를 활용하여 반복적으로 재선형화(re-linearization)를 수행하므로, 특히 비선형성이 강한 문제에서 필터링 방식보다 월등히 높은 정확도를 제공하는 것으로 입증되었다.1 동시적 위치추정 및 지도작성(Simultaneous Localization and Mapping, SLAM)과 모션으로부터 구조 복원(Structure from Motion, SfM)은 이러한 최적화 기반 패러다임이 가장 성공적으로 적용된 대표적인 응용 분야이다.2


GTSAM(Georgia Tech Smoothing and Mapping)은 바로 이러한 비선형 최적화 문제를 해결하기 위해 조지아 공과대학교(Georgia Institute of Technology)에서 개발된 C++ 라이브러리이다.10 GTSAM의 핵심 철학은 문제를 추상적인 희소 행렬(sparse matrix)로 변환하는 대신, 팩터 그래프(factor graph)와 베이즈 네트워크(Bayes Network)를 기본적인 계산 패러다임으로 사용하는 것이다.10 이러한 접근 방식은 복잡한 추정 문제를 훨씬 직관적이면서도 강력하게 모델링할 수 있는 틀을 제공한다.16

본 보고서는 GTSAM에 대한 심층적인 고찰을 목표로 한다. 이를 위해 먼저 GTSAM의 이론적 기반이 되는 팩터 그래프와 베이즈 추론의 원리를 탐구하고, 라이브러리의 내부 아키텍처와 핵심 기능을 심층 분석할 것이다. 나아가 SLAM 및 SfM과 같은 핵심 문제를 GTSAM을 통해 실제로 구현하는 방법을 코드 수준에서 살펴보고, 시각-관성 주행 거리 측정(Visual-Inertial Odometry, VIO)과 같은 고급 응용 사례를 다룬다. 마지막으로, g2o, Ceres Solver 등 경쟁 프레임워크와의 비교 분석을 통해 GTSAM의 장단점을 명확히 하고, 현재의 한계점과 최신 연구 동향을 조망함으로써 미래 전망을 제시하고자 한다.



확률적 그래피컬 모델(Probabilistic Graphical Models, PGM)은 확률 이론과 그래프 이론을 결합하여, 복잡한 확률 분포를 변수 간의 구조적 관계를 이용해 간결하게 표현하는 강력한 도구이다.5 PGM은 크게 방향성 모델과 비방향성 모델로 나뉜다.

대표적인 방향성 모델인 베이즈 네트워크(Bayesian Networks, BN)는 방향성이 있는 비순환 그래프(Directed Acyclic Graph, DAG)를 사용하여 변수 간의 인과 관계를 표현한다.5 각 노드는 확률 변수를 나타내고, 부모 노드에서 자식 노드로 향하는 엣지는 조건부 의존성을 의미한다. 이를 통해 전체 결합 확률 분포를 각 변수의 조건부 확률의 곱으로 인수분해(factorization)할 수 있다.

반면, 마르코프 랜덤 필드(Markov Random Fields, MRF)는 비방향성 그래프를 사용하여 변수 간의 직접적인 의존 관계를 표현하는 대표적인 비방향성 모델이다.5 MRF에서 두 변수는 그래프 상에서 직접 연결되어 있을 때만 직접적인 상호 의존성을 가지며, 전체 확률 분포는 그래프 내의 클리크(clique)에 정의된 포텐셜 함수(potential function)들의 곱으로 표현된다. 팩터 그래프는 이러한 PGM의 한 종류로, BN이나 MRF와는 다른 독특한 표현력을 가진다.5


팩터 그래프는 상태 추정 문제의 근간을 이루는 확률 분포의 인수분해 구조를 가장 명시적으로 드러내는 그래피컬 모델이다. 이는 두 종류의 노드로 구성된 이분 그래프(bipartite graph)로 정의된다: 하나는 추정하고자 하는 미지의 양을 나타내는 **변수 노드(variable node)**이고, 다른 하나는 변수들의 부분 집합에 대한 확률적 제약이나 함수를 나타내는 **팩터 노드(factor node)**이다.6 팩터 그래프에서 엣지는 항상 변수 노드와 팩터 노드 사이에만 존재하며, 이는 특정 팩터가 어떤 변수들에 의존하는지를 명확히 보여준다.

팩터 그래프의 핵심은 전역 함수, 예를 들어 사후 확률(posterior probability) $P(X|Z)$를 지역적인 팩터들의 곱으로 표현하는 것이다:1

$$
P(X|Z) \propto \prod_{i} f_i(X_i)
$$
여기서 $X$는 모든 변수들의 집합, $Z$는 모든 측정값들의 집합, $f_i$는 개별 팩터, $X_i$는 팩터 $f_i$에 연결된 변수들의 부분 집합을 의미한다.

이러한 구조는 다른 그래피컬 모델에 비해 몇 가지 중요한 장점을 가진다.20

- **비방향성 그래프(MRF)와의 비교:** 팩터 그래프는 MRF보다 더 풍부한 표현력을 가진다. 예를 들어, 세 변수 $x_1, x_2, x_3$가 서로 종속적인 관계에 있다고 가정하자. MRF에서는 이 세 변수가 하나의 클리크를 형성한다고 표현할 뿐, 그들 사이의 구체적인 함수 관계를 구분하지 못한다. 만약 이 관계가 두 개의 다른 함수, 즉 $f_a(x_1, x_2, x_3)$와 $f_b(x_2, x_3)$의 곱으로 표현된다면, MRF는 이 둘을 합친 하나의 포텐셜 함수로만 나타낼 수 있다. 반면, 팩터 그래프는 $f_a$와 $f_b$를 각각 별개의 팩터 노드로 명시적으로 표현할 수 있어, 결합 확률 분포의 인수분해 구조를 정확하게 드러낸다. 이처럼 하나의 MRF는 여러 다른 팩터 그래프로 표현될 수 있지만, 그 역은 성립하지 않으며, 이 과정에서 MRF는 세부 정보를 잃게 된다.20

- **방향성 그래프(BN)와의 비교:** 은닉 마르코프 모델(Hidden Markov Model, HMM)과 같은 동적 시스템을 생각해 보자. BN에서는 상태 전이 확률 $P(X_{t+1}|X_t)$과 측정 확률 $P(Z_t|X_t)$이 방향성 엣지로 표현된다.18 이를 팩터 그래프로 변환하면, 상태 전이 확률은 두 상태 변수 

  $X_t, X_{t+1}$에 연결되는 팩터로, 측정 확률(또는 우도 함수)은 상태 변수 $X_t$에 연결되는 팩터로 각각 모델링된다. 이 변환을 통해 모든 확률적 제약 조건들이 대칭적인 '팩터'로 취급되어 문제의 구조를 더 명확하게 파악할 수 있다.19

- **추론에서의 이점:** 그래프 기반 추론 알고리즘에서 '트리 구조'는 문제를 매우 단순하게 만들어주는 중요한 특성이다. 방향성 그래프나 비방향성 그래프가 순환(cycle) 구조를 가질 경우, 이를 적절한 팩터 그래프로 변환하면 순환 구조가 없는 트리 구조로 만들 수 있는 경우가 많다. 예를 들어, 방향성 그래프의 '머리 대 머리(head-to-head)' 구조는 비방향성 그래프로 변환 시 순환을 유발할 수 있지만, 이를 팩터 그래프로 변환하면 다시 트리 구조를 얻을 수 있다. 이처럼 팩터 그래프는 추론 문제 해결에 있어 구조적인 이점을 제공한다.20

이러한 장점들은 "보고 있는 것을 그대로 모델링한다(What You See Is What You Model)"는 GTSAM의 핵심 철학으로 이어진다. 로보틱스 연구자가 SLAM 문제를 구상할 때, 그들은 로봇의 시점별 자세(pose), 자세들 간의 상대적 움직임(odometry), 그리고 특정 자세에서 관측한 랜드마크(landmark sighting)라는 개념으로 생각한다. 이 개념적 모델은 팩터 그래프의 구조와 정확히 일치한다. 즉, 로봇의 자세는 변수 노드가 되고, 주행 거리 측정치는 두 자세 변수를 잇는 이진(binary) 팩터가 되며, 랜드마크 관측치는 자세 변수와 랜드마크 변수를 잇는 또 다른 이진 팩터가 된다.11 GTSAM에서 

`graph.add(BetweenFactor<Pose2>(...))`와 같은 코드는 연구자가 화이트보드에 그리는 팩터 그래프 다이어그램을 그대로 반영한다. 이는 문제를 거대한 단일 행렬이나 상태 전이 모델로 추상화하는 다른 접근법과 대조된다. 결과적으로 GTSAM은 강력한 최적화 도구일 뿐만 아니라, 사용자가 문제의 본질적인 희소성(sparsity)과 확률적 가정을 명확하게 이해하도록 돕는 훌륭한 교육적 도구이기도 하다.18


로보틱스에서의 상태 추정 문제는 크게 필터링과 스무딩이라는 두 가지 패러다임으로 나눌 수 있다.

- **필터링(Filtering):** 필터링은 시간 $t$까지의 모든 측정값 $z_{1:t}$을 이용하여 현재 시점 $t$의 상태 $x_t$에 대한 확률 분포 $p(x_t | z_{1:t})$를 추정하는 문제이다.7 이는 재귀적인 과정으로, 새로운 측정값이 들어올 때마다 이전 상태 추정치를 기반으로 현재 상태를 갱신한다. 칼만 필터 계열이 대표적인 필터링 알고리즘이다. 필터링은 온라인 추정에 계산적으로 효율적이지만, 과거 상태에 대한 정보를 주변화하여 버리기 때문에 비선형 시스템에서는 초기의 부정확한 선형화 오류를 되돌릴 수 없다는 단점이 있다.8
- **스무딩(Smoothing):** 스무딩은 시간 $t$의 상태를 추정하기 위해 가용한 모든 측정값, 즉 $t$ 시점 이후의 미래 측정값까지 포함한 전체 데이터셋 $z_{1:T}$ (단, $T \ge t$)를 사용하는 문제이다. 즉, 사후 확률 분포 $p(x_t | z_{1:T})$를 구하는 것이다.12 이는 모든 정보를 활용하여 전체 궤적을 한 번에 최적화하는 문제로, 반복적인 재선형화를 통해 필터링보다 훨씬 정확한 결과를 얻을 수 있다.8 스무딩의 가장 큰 단점은 전통적으로 계산 비용이 매우 높다는 점이었다. 시간이 지남에 따라 추정해야 할 변수와 제약 조건이 계속 늘어나기 때문이다.8
- **고정 지연 스무딩(Fixed-Lag Smoothing):** 이는 필터링과 전체 스무딩 사이의 절충안으로, 최근의 일정 시간 창(sliding window) 내의 상태들만 최적화하는 방식이다.8 계산 비용을 일정하게 유지할 수 있지만, 창 밖의 정보를 버리기 때문에 휴리스틱한 근사 해법에 해당한다.

GTSAM은 기본적으로 전체 스무딩 및 매핑(SAM) 프레임워크를 지향한다.10 GTSAM의 핵심 혁신 기술인 iSAM2 알고리즘은 전체 스무딩의 높은 계산 비용 문제를 해결하여, 이를 실시간 온라인 응용 프로그램에서도 사용할 수 있도록 만들었다. 이에 대한 자세한 내용은 다음 장에서 다룬다.



GTSAM은 로보틱스와 컴퓨터 비전 분야의 스무딩 및 매핑(SAM) 문제를 해결하기 위한 C++ 라이브러리로, 그 핵심 철학은 팩터 그래프를 기본 계산 패러다임으로 삼는 것이다.10 이는 문제를 단순히 희소 행렬 연산으로 추상화하는 대신, 문제의 고유한 그래피컬 구조를 명시적으로 활용하여 직관성과 성능을 모두 높이려는 시도이다.

GTSAM은 매우 모듈화되고 확장 가능하도록 설계되었다. 사용자는 자신의 특정 문제에 맞춰 새로운 변수 타입과 팩터 타입을 손쉽게 정의하고 추가할 수 있다.26 라이브러리의 기반은 고성능 C++ 라이브러리들로 구성되어 있는데, 특히 선형대수 연산을 위해 **Eigen** 라이브러리를 적극적으로 활용하며 28, 과거에는 다양한 유틸리티를 위해 **Boost** 라이브러리에 의존했다.10 최근에는 Boost 의존성을 줄이고 C++17 표준을 채택하려는 움직임이 있다.10


GTSAM 라이브러리는 여러 핵심 클래스와 기능으로 구성되어 있다.

- 핵심 클래스 26:

  - `Values`: 그래프 내 모든 변수 노드들의 현재 추정치를 담는 컨테이너 객체이다. 각 변수는 고유한 `Key` 값(주로 `gtsam::symbol` 함수로 생성)으로 식별된다.19
  - `FactorGraph` (및 `NonlinearFactorGraph`): 문제에 포함된 모든 팩터들을 담는 컨테이너 객체이다. 이 객체 자체가 확률 모델 $P(X|Z)$를 나타낸다.11
  - `Factor` 타입: 라이브러리는 일반적인 로보틱스 작업을 위한 풍부한 종류의 팩터들을 미리 구현하여 제공한다.27
    - `PriorFactor`: 단일 변수에 대한 사전 정보를 나타내는 단항(unary) 팩터.11
    - `BetweenFactor`: 주행 거리 측정(odometry)과 같이 두 변수 간의 상대적 제약을 나타내는 이진(binary) 팩터.11
    - `ProjectionFactor`, `BearingRangeFactor`: 랜드마크에 대한 센서 측정값을 모델링하는 팩터들.19

- 매니폴드 상에서의 최적화 (Optimization on Manifolds) 3:

  로보틱스에서 다루는 많은 변수들, 예를 들어 3차원 회전($SO(3)$)이나 3차원 자세($SE(3)$)는 단순한 유클리드 공간이 아닌 비선형 매니폴드(manifold) 상에 존재한다. GTSAM은 이러한 기하학적 타입을 일급 객체(first-class citizen)로 취급하며, 리트랙션(retraction)이나 지수 사상(exponential map)과 같은 개념을 사용하여 매니폴드 상에서 올바르게 최적화를 수행한다.32 이는 회전을 오일러 각(Euler angles)으로 다루는 순진한 접근법에서 발생할 수 있는 특이점(singularity) 문제를 근본적으로 방지하는 중요한 장점이다.

- 노이즈 모델 (Noise Models):

  GTSAM은 가우시안, 대각(Diagonal), 등방성(Isotropic) 등 다양한 노이즈 모델을 제공하여 측정값의 불확실성을 정밀하게 표현할 수 있도록 지원한다.11


전체 스무딩은 이론적으로 가장 정확한 해를 제공하지만, 새로운 측정값이 추가될 때마다 전체 문제를 처음부터 다시 풀어야 하므로 계산 비용이 매우 높다.8 iSAM2(incremental Smoothing and Mapping)는 이러한 단점을 극복하고 효율적인 온라인 스무딩을 가능하게 하는 GTSAM의 최첨단 알고리즘이다.1

iSAM2의 효율성은 **베이즈 트리(Bayes tree)**라는 특별한 자료 구조에서 비롯된다.9

베이즈 트리는 정보 행렬(information matrix)의 콜레스키 분해(Cholesky factorization) 결과를 확률적 그래피컬 모델 형태로 표현한 것이다. 기존의 스무딩 방식이 새로운 측정값이 추가될 때마다 거대한 정보 행렬 전체를 다시 분해해야 했던 것과 달리, iSAM2는 베이즈 트리를 직접 수정하는 방식을 사용한다. 새로운 정보가 추가되면, 그 정보에 의해 영향을 받는 그래프의 일부만 선택적으로 재선형화하고 베이즈 트리를 갱신한다. 이 접근법 덕분에 많은 응용 분야에서 점진적인 업데이트가 분할 상환 분석(amortized analysis) 관점에서 거의 상수 시간($O(1)$)에 가까운 성능을 보인다.8

그러나 iSAM2의 강력한 성능에는 중요한 전제 조건이 따른다. iSAM2의 내부 메커니즘은 정보 행렬($A^T A$)의 콜레스키 인수($R$ 행렬)를 유지하고 갱신하는 것에 의존한다.36 콜레스키 분해는 대상 행렬이 양의 준정부호(positive semi-definite)일 때만 정의된다. 이는 곧 최적화 문제가 "잘 제약되어(well-constrained)" 있어야 함을 의미한다.36 단안 카메라를 이용한 시각 주행 거리 측정(monocular visual odometry)에서 발생하는 절대적인 스케일(scale)이나 초기 자세(pose)의 모호성(gauge freedom)과 같이 관측 불가능한 변수가 존재하는 문제에서는 정보 행렬이 완전한 랭크(full rank)를 갖지 못하며, 따라서 콜레스키 분해가 불안정해진다.

결론적으로, iSAM2는 관측 불가능한 변수나 게이지 자유도가 있는 문제에 그대로 적용할 수 없다. 사용자는 iSAM2를 적용하기 전에 사전 정보(prior)를 추가하거나 특정 변수를 고정하는 등의 방법으로 문제가 적절히 제약되도록 보장해야 한다.36 이는 iSAM2의 매우 강력한 성능 이면에 숨겨진 실용적인 제약 조건으로, 문제의 관측 가능성(observability)에 대한 깊은 이해가 필요함을 시사한다. 이는 Ceres와 같은 다른 솔버들이 이러한 게이지 자유도를 더 유연하게 처리하는 것과 대조되는 지점이다.36


GTSAM을 사용하기 위해서는 몇 가지 사전 요구사항이 필요하다. 기본적으로 CMake, 최신 C++ 컴파일러, Boost 라이브러리가 필요하며, 성능 향상을 위해 선택적으로 Intel TBB(Threaded Building Blocks)나 MKL(Math Kernel Library)을 함께 설치할 수 있다.10 설치는 표준적인 `cmake`와 `make`를 이용한 빌드 과정을 따르며 10, Arch Linux의 AUR과 같은 일부 리눅스 배포판에서는 패키지 관리자를 통해 쉽게 설치할 수도 있다.29

GTSAM의 가장 큰 장점 중 하나는 **MATLAB 및 Python 래퍼(wrapper)**를 공식적으로 지원한다는 점이다.10 이 래퍼들은 신속한 프로토타이핑과 결과 시각화에 매우 유용하다. 래퍼의 API 구문은 C++ API와 매우 유사하게 설계되어 있어, 두 환경 간의 전환이 용이하다.19 이는 Python이나 MATLAB과 같은 고급 언어 환경에서 알고리즘을 개발하고 디버깅한 후, 성능이 중요한 부분만 C++로 구현하여 배포하는 강력한 개발 워크플로우를 가능하게 한다.




2D Pose-Graph SLAM은 로봇이 평면상에서 이동하며 주행 거리 측정(odometry)과 루프 클로저(loop closure)로부터 상대적인 자세(pose) 측정값을 얻는 상황을 모델링한다.41 루프 클로저는 로봇이 과거에 방문했던 장소를 다시 인식했을 때 얻어지는 제약 조건으로, 누적된 오차를 보정하는 데 결정적인 역할을 한다. 이 문제의 목표는 이러한 상대적 제약 조건들을 만족시키는 로봇의 전체 궤적, 즉 모든 시점의 자세들을 최적화하는 것이다.42


이 문제는 다음과 같이 팩터 그래프로 모델링된다.

- **변수(Variables):** 로봇의 각 시점별 자세는 `Pose2` 타입의 변수(2차원 위치 $x, y$와 방향 $\theta$)로 표현되며, 고유한 키(key)로 식별된다.
- **팩터(Factors):**
  - `PriorFactor<Pose2>`: 첫 번째 자세 변수($x_1$)에 추가되는 단항 팩터이다. 이는 맵의 기준이 되는 전역 좌표계를 고정하고, 전체 맵의 위치 및 방향 모호성을 제거하는 역할을 한다.
  - `BetweenFactor<Pose2>`: 연속된 두 자세 변수($x_i, x_{i+1}$) 사이에 추가되는 이진 팩터로, 휠 인코더나 LiDAR 스캔 매칭으로 얻은 주행 거리 측정값을 나타낸다.
  - `BetweenFactor<Pose2>`: 로봇이 이전에 방문했던 장소를 재인식했을 때, 연속적이지 않은 두 자세 변수($x_i, x_j$) 사이에 추가되는 이진 팩터로, 루프 클로저 제약 조건을 나타낸다.


C++를 이용한 2D Pose-Graph SLAM의 기본적인 구현은 다음과 같다.11

```C++
// 비선형 팩터 그래프 객체 생성
NonlinearFactorGraph graph;

// 첫 번째 자세 x1에 대한 가우시안 사전 정보(prior) 추가
Pose2 priorMean(0.0, 0.0, 0.0); // 평균값은 원점
noiseModel::Diagonal::shared_ptr priorNoise =
  noiseModel::Diagonal::Sigmas(Vector3(0.3, 0.3, 0.1)); // 노이즈 모델
graph.add(PriorFactor<Pose2>(1, priorMean, priorNoise));

// 주행 거리 측정(odometry) 팩터 추가
Pose2 odometry(2.0, 0.0, 0.0); // x축으로 2만큼 이동
noiseModel::Diagonal::shared_ptr odometryNoise =
  noiseModel::Diagonal::Sigmas(Vector3(0.2, 0.2, 0.1));
graph.add(BetweenFactor<Pose2>(1, 2, odometry, odometryNoise));
graph.add(BetweenFactor<Pose2>(2, 3, odometry, odometryNoise));
//... (루프 클로저 팩터 추가)...
```

이 코드는 먼저 `NonlinearFactorGraph` 객체를 생성한다. `PriorFactor`는 키 `1`로 식별되는 첫 번째 자세에 대한 사전 믿음을 정의하며, `BetweenFactor`는 키 `1`과 `2`, 그리고 `2`와 `3`으로 식별되는 자세들 간의 상대적 움직임을 모델링한다.

비선형 최적화는 초기 추정치(initial estimate)에 크게 의존하므로, `Values` 객체를 사용하여 모든 변수에 대한 초기값을 제공해야 한다.11 그 후, Levenberg-Marquardt와 같은 최적화기를 생성하고 실행하여 최적화된 `Values` 객체를 얻는다. 최적화 결과뿐만 아니라, 각 변수의 불확실성을 나타내는 주변 공분산(marginal covariance)도 계산할 수 있다.11

MATLAB/Python 래퍼를 사용한 구현은 C++ 코드와 거의 동일하며, 구문상의 차이만 존재한다.19 이는 프로토타이핑과 실제 구현 간의 간극을 크게 줄여준다.



Pose-Graph SLAM을 확장하여, 주변 환경에 존재하는 정적인 랜드마크들의 위치까지 명시적으로 추정하는 문제를 랜드마크 기반 SLAM이라 한다. 이 경우, 상태 벡터는 로봇의 자세뿐만 아니라 모든 랜드마크의 위치를 포함하게 된다.19


- **변수(Variables):** 로봇 자세를 위한 `Pose2` 변수와 랜드마크 위치를 위한 `Point2` 변수가 사용된다.
- **팩터(Factors):** 자세에 대한 `PriorFactor`와 `BetweenFactor` 외에, 센서 측정값을 모델링하는 팩터가 추가된다.
  - `BearingRangeFactor2D`: 로봇이 특정 자세에서 랜드마크까지의 방향(bearing)과 거리(range)를 측정했을 때 사용되는 이진 팩터이다. 이 팩터는 하나의 `Pose2` 변수와 하나의 `Point2` 변수를 연결한다.


MATLAB 예제 코드를 통해 랜드마크 기반 SLAM의 팩터 그래프 구성을 살펴보자.

```Matlab
% 그래프 컨테이너 생성
graph = NonlinearFactorGraph;

% 변수 키 생성 (x: 자세, l: 랜드마크)
i1 = symbol('x',1); i2 = symbol('x',2);
j1 = symbol('l',1);

%... (자세에 대한 Prior 및 Odometry 팩터 추가)...

% 랜드마크 측정 팩터 추가 (방향/거리)
brNoise = noiseModel.Diagonal.Sigmas([0.1; 0.2]); % 측정 노이즈
% 자세 i1에서 랜드마크 j1 관측
graph.add(BearingRangeFactor2D(i1, j1, Rot2(45*degrees), sqrt(8), brNoise));
% 자세 i2에서 랜드마크 j1 관측
graph.add(BearingRangeFactor2D(i2, j1, Rot2(90*degrees), 2, brNoise));
```

위 코드에서 `symbol('l', 1)`을 통해 랜드마크 변수를 위한 키를 생성하고, `BearingRangeFactor2D`를 사용하여 각 자세에서 랜드마크를 관측한 측정값을 그래프에 추가한다. 최적화 결과, 여러 번 관측된 랜드마크일수록 그 위치의 불확실성이 감소(공분산 타원이 작아짐)하는 것을 확인할 수 있다.19



SfM은 정렬되지 않은 여러 장의 2D 이미지로부터 3차원 환경의 구조(3D 점들)와 카메라의 3차원 자세를 동시에 복원하는 문제이다.17 이는 SLAM과 매우 유사한 문제이지만, 보통 시간 순서에 따른 제약이 없고 전역적인 최적화(bundle adjustment)에 더 초점을 맞춘다.


- **변수(Variables):** 카메라의 3차원 자세를 위한 `Pose3` 변수와 3차원 점의 위치를 위한 `Point3` 변수가 사용된다.
- **팩터(Factors):**
  - `PriorFactor<Pose3>`: 하나 이상의 카메라 자세에 사전 정보를 추가하여, 전체 시스템의 7자유도 모호성(3D 위치, 3D 회전, 스케일)을 제거한다.
  - `GenericProjectionFactor<Cal3_S2>`: SfM의 핵심이 되는 팩터이다. 이 팩터는 3D 점을 특정 카메라 자세로 투영했을 때 예측되는 2D 이미지 좌표와, 실제 이미지에서 관측된 2D 특징점 좌표 사이의 재투영 오차(reprojection error)를 계산한다. 이 팩터는 하나의 `Pose3` 변수와 하나의 `Point3` 변수를 연결한다.


C++ 및 MATLAB 예제를 보면, SfM 문제 역시 SLAM과 동일한 프레임워크 내에서 해결됨을 알 수 있다.30 사용자는 단지 문제에 맞는 변수 타입(`Pose3`, `Point3`)과 팩터 타입(`ProjectionFactor`)을 선택하여 그래프를 구성하기만 하면 된다. 이는 팩터 그래프 추상화가 얼마나 강력하고 모듈화되어 있는지를 보여주는 대표적인 사례이다.17

더 나아가, GTSAM을 기반으로 구축된 **GTSfM** 프로젝트는 특징점 검출/매칭과 같은 프론트엔드(`frontend`)부터 회전/이동 평균화(`averaging`), 최종적인 번들 조정(`bundle`)에 이르는 완전한 SfM 파이프라인을 제공한다.44 이는 GTSAM이 복잡한 시스템의 핵심 백엔드 최적화 엔진으로서 얼마나 중요한 역할을 하는지를 잘 보여준다.



VIO는 카메라의 시각 정보와 관성 측정 장치(Inertial Measurement Unit, IMU)의 관성 정보를 융합하여 로봇의 자세를 추정하는 기술이다. 이 문제의 핵심적인 어려움은 수백 Hz에 달하는 고주파의 IMU 데이터와 상대적으로 낮은 주파수(예: 30Hz)의 카메라 데이터를 효율적으로 결합하는 데 있다. 모든 IMU 측정값을 개별 팩터로 추가하는 것은 계산적으로 불가능에 가깝다.

이 문제를 해결하기 위해 **IMU 사전적분(IMU Preintegration)** 이라는 개념이 도입되었다.8

IMU 사전적분은 두 카메라 키프레임(i와 j) 사이의 수많은 IMU 측정값들을 하나의 상대적인 움직임 제약 조건으로 미리 적분해두는 기법이다. 이렇게 사전적분된 측정값은 키프레임 i와 j에서의 자세, 속도, 바이어스에만 의존하며, 그 사이의 모든 중간 상태와는 무관해진다. GTSAM은 이 사전적분을 단순한 유클리드 공간에서의 오일러 적분이 아닌, 회전의 비선형성을 올바르게 다루는 매니폴드 상에서(on the manifold) 수행하는 정교한 구현체를 제공한다. 이는 정확도 측면에서 매우 중요한 장점이다.10

GTSAM에서는 `CombinedImuFactor` (또는 구버전의 `PreintegratedImuMeasurements`)라는 팩터를 통해 IMU 사전적분을 손쉽게 사용할 수 있다. VIO 시스템의 팩터 그래프는 카메라의 재투영 오차를 모델링하는 `ProjectionFactor`와 IMU의 상대적 움직임을 모델링하는 `CombinedImuFactor`를 결합하여 구성된다. 다수의 연구 프로젝트들이 GTSAM을 VIO 백엔드로 사용하여 최첨단 성능을 달성한 바 있다.33


팩터 그래프 패러다임은 LiDAR, IMU, 카메라, GPS, 기압계 등 종류와 측정 주기가 각기 다른 다양한 센서 데이터를 융합하는 데 매우 적합하다.4 각 센서로부터 얻어지는 측정값은 그것이 제약하는 상태 변수들을 연결하는 하나의 팩터로 자연스럽게 모델링될 수 있기 때문이다.3

- GPS-IMU 융합 37:

   GPS 측정값은 특정 시점의 자세 변수에 대한 절대 위치 정보를 제공하는 `GPSFactor`(일종의 `PriorFactor`)로 모델링될 수 있으며, 이는 IMU 사전적분 팩터가 제공하는 상대적 움직임 정보와 결합되어 전역적인 드리프트를 억제한다.

- LiDAR-IMU 융합 46:

   LiDAR 스캔 매칭을 통해 얻은 상대적 자세 변환은 `BetweenFactor`로 모델링되며, 이는 IMU 사전적분 팩터와 함께 융합되어 더욱 강건한 주행 거리 측정을 가능하게 한다. 이 프레임워크는 센서 간의 외부 파라미터(extrinsic parameter)를 보정하는 데에도 활용될 수 있다.

이러한 고급 응용 사례들을 통해 GTSAM의 진정한 가치를 파악할 수 있다. GTSAM은 단순히 여러 센서를 융합할 수 있다는 사실을 넘어, IMU 사전적분과 같이 이론적으로 복잡하고 구현이 까다로운 핵심 요소들을 매우 강건하고 정확한 최첨단 방식으로 미리 구현하여 제공한다.10 이는 연구자나 엔지니어가 복잡한 최적화 백엔드를 처음부터 구현하는 수고를 덜고, 자신들의 핵심 분야인 새로운 센서 모델이나 프론트엔드 알고리즘 개발에 집중할 수 있도록 돕는다. 결과적으로 GTSAM은 고성능 다중 센서 융합 시스템의 개발을 극적으로 가속하는 '조력자(enabler)' 역할을 수행한다.49


로보틱스 및 컴퓨터 비전 분야의 비선형 최적화 문제 해결을 위해 GTSAM 외에도 g2o와 Ceres Solver가 널리 사용되고 있다. 각 프레임워크는 고유한 철학과 장단점을 가지고 있어, 문제의 성격과 개발 목표에 따라 적절한 도구를 선택하는 것이 중요하다.


- **GTSAM:** **팩터 그래프 중심적**이다. API 자체가 사용자가 문제를 그래피컬 모델의 관점에서 사고하도록 유도한다. 대부분의 일반적인 팩터들은 미리 구현되어 있지만, 새로운 팩터를 추가하려면 자코비안(Jacobian)을 직접 해석적으로 유도하여 제공해야 한다. 코드의 구조가 모델의 구조를 그대로 반영하는 것이 특징이다.10
- **g2o (General Graph Optimization):** **범용 그래프 최적화** 프레임워크이다. GTSAM과 마찬가지로 그래프 기반이지만, API는 정점(vertex)과 엣지(edge)를 정의하는 보다 일반적인 그래프 구조에 가깝다. 확장성은 뛰어나지만, 초기 설정이 복잡할 수 있다.13 사용자 커뮤니티는 Ceres에 비해 상대적으로 작다는 평가가 있다.36
- **Ceres Solver:** **범용 비선형 최소제곱 문제 해결기**이다. 본질적으로 '그래프' 라이브러리가 아니라, 사용자가 잔차 블록(residual block) 또는 비용 함수(cost function)를 정의하는 방식이다. 가장 큰 특징은 **자동 미분(automatic differentiation)** 기능으로, 사용자가 자코비안을 직접 계산할 필요 없이 라이브러리가 자동으로 계산해준다. 이는 새로운 모델을 신속하게 프로토타이핑할 때 압도적인 편의성을 제공한다.36 또한, 매니폴드 연산을 정의하는 API가 유연하여 다양한 제약 조건을 다루기 용이하다.55


세 프레임워크 모두 고도로 최적화되어 있어 매우 높은 성능을 보인다. 특정 문제에 대한 성능은 문제의 종류, 파라미터 튜닝, 하드웨어 등 다양한 요인에 따라 달라질 수 있다.51

- **GTSAM:** 증분 최적화 솔버인 **iSAM2**를 사용할 수 있는 문제(예: 잘 제약된 Pose-Graph SLAM)에서 특히 효율적이다.36 팩터 그래프 구조를 직접 활용하는 방식은 로보틱스 문제 해결에 매우 효율적인 해법을 제공할 수 있다.50
- **g2o:** 시각적 재구성(visual reconstruction)과 같은 특정 문제에 특화되어 있으며, 해당 분야에서 매우 효율적인 것으로 알려져 있다.36 ORB-SLAM과 같은 성공적인 오픈소스 프로젝트의 백엔드로 채택되어 그 성능을 입증했다.51
- **Ceres:** 매우 강건하며 널리 사용된다. 구글의 Cartographer SLAM 프레임워크 내에서 g2o와 직접 비교한 일부 실험에서는 Ceres가 수렴 속도와 정확도 측면에서 더 나은 성능을 보였다.13 구글의 풍부한 엔지니어링 자원을 바탕으로 지속적으로 튜닝되고 있다.


- **GTSAM:** 자세, 매니폴드, IMU 사전적분 등 로보틱스 표준 문제에 대한 풍부한 **사전 구현 팩터 라이브러리**를 제공하여 해당 문제에 대한 개발을 매우 용이하게 한다.56 MATLAB/Python 래퍼는 신속한 프로토타이핑과 디버깅에 큰 장점이다.19 다만, 복잡한 매니폴드 상에 새로운 커스텀 팩터를 정의해야 할 경우 학습 곡선이 가파를 수 있다.
- **g2o:** 초기 설정이 다소 복잡할 수 있다. 2D SLAM 예제를 30줄 미만의 코드로 구현할 수 있을 만큼 강력하지만 53, 초심자에게는 시스템 전체가 GTSAM보다 덜 직관적일 수 있다.
- **Ceres:** **자동 미분** 기능은 새로운 비용 함수를 개발할 때 사용 편의성 측면에서 막대한 이점을 제공한다.36 또한, 팀과 기존 코드베이스가 이미 Ceres에 기반을 두고 있다면 굳이 다른 프레임워크로 전환할 필요가 없다는 의견은 57, 그만큼 Ceres가 충분히 강력하고 범용적임을 시사한다. 게이지 자유도 문제도 비교적 잘 처리한다.36


- **GTSAM:** 조지아 공대를 중심으로 한 강력한 **학계 커뮤니티**를 보유하고 있다. 문서화와 예제가 잘 되어 있으며 56, ROS에 통합되어 있고 Ubuntu PPA를 통해 쉽게 설치할 수 있다.58
- **g2o:** Ceres보다 사용자 커뮤니티가 작지만 36, ORB-SLAM과 같은 유명 오픈소스 프로젝트에서 사용되어 참고할 만한 예제가 풍부하다.51 RTAB-Map에서도 기본 최적화기로 사용되지만, 다중 세션 매핑에서는 GTSAM이 더 강건하다는 평가도 있다.59
- **Ceres:** 구글이 지원하는 가장 큰 **산업계 및 학계 커뮤니티**를 자랑한다.36 문서화가 매우 뛰어나며, Android/iOS와 같은 모바일 플랫폼까지 지원하는 등 활발하게 발전하고 있다.36

다음 표는 세 가지 최적화 프레임워크의 특징을 한눈에 비교하여 요약한 것이다.

| 특징                   | GTSAM                                    | g2o                       | Ceres Solver                           |
| ---------------------- | ---------------------------------------- | ------------------------- | -------------------------------------- |
| **핵심 패러다임**      | 팩터 그래프 추상화                       | 범용 그래프 최적화        | 비선형 잔차 최소화                     |
| **자코비안 계산**      | 해석적 (사전 구현/사용자 정의)           | 해석적                    | 자동 미분                              |
| **핵심 강점**          | 증분 최적화(iSAM2), 매니폴드 최적화      | 시각 SLAM에서의 효율성    | 유연성, 자동 미분, 강건성              |
| **이상적인 사용 사례** | 실시간 로보틱스 SLAM/VIO                 | 배치 번들 조정, 시각 SLAM | 새로운 최적화 문제 프로토타이핑        |
| **사용 편의성**        | 표준 문제에 직관적, 커스텀 팩터는 어려움 | 중간-높음 수준의 복잡도   | 자동 미분 덕분에 새로운 문제에 용이    |
| **커뮤니티/지원**      | 강력한 학계 커뮤니티, 좋은 문서          | 작지만 확립된 커뮤니티    | 대규모 산업/학계 커뮤니티, 뛰어난 문서 |
| **게이지 자유도 처리** | 주의 필요 (특히 iSAM2 사용 시)           | 솔버가 처리               | 솔버가 잘 처리                         |



GTSAM은 강력한 도구이지만, 몇 가지 내재적인 한계와 도전 과제를 안고 있다.

- **이상치(Outlier)에 대한 민감성:** 최소제곱 기반 최적화기는 본질적으로 이상치에 매우 민감하다. 특히 SLAM에서 잘못된 데이터 연관(data association)으로 인해 발생한 거짓 루프 클로저(false positive loop closure)는 전체 최적화 결과를 완전히 왜곡시켜 발산하게 만들 수 있다.60 이는 GTSAM만의 문제가 아닌 최소제곱법의 근본적인 한계로, 백엔드 최적화기의 성능은 양질의 프론트엔드에 크게 의존할 수밖에 없다.17
- **iSAM2의 요구 조건:** 앞서 논의했듯이, iSAM2는 문제가 잘 제약되어 있을 것을 요구한다. 이는 게이지 자유도가 있는 문제에 iSAM2를 직접 적용하기 어렵게 만드는 요인이다.36
- **초기값 민감성:** 모든 비선형 최적화기가 그렇듯, GTSAM 역시 전역 최적해(global minimum)에 수렴하기 위해 합리적인 수준의 초기 추정치가 필요하다. 좋지 않은 초기값은 지역 최적해(local minimum)에 빠지게 할 위험을 높인다.42
- **커스텀 팩터 구현의 복잡성:** GTSAM의 확장성은 뛰어나지만, 새로운 팩터를, 특히 복잡한 매니폴드 상에서 정의하기 위해서는 미분 기하학에 대한 깊은 이해가 필요하다. `evaluateError` 함수와 자코비안을 올바르게 구현하는 것은 상당한 전문성을 요구하는 작업이다.27


이러한 한계들을 극복하고 새로운 영역으로 나아가기 위한 연구들이 활발히 진행되고 있다.

- **강건한 SLAM (Robust SLAM):** 백엔드 최적화기 자체가 이상치에 강건해지도록 만드는 연구가 진행 중이다. 예를 들어, 최적화 과정에서 잘못된 제약 조건을 스스로 식별하고 거부하는 메커니즘을 도입하려는 시도들이 있다.60
- **협력적 SLAM (Collaborative SLAM, C-SLAM):** 여러 로봇이 협력하여 공동의 맵을 구축하는 C-SLAM 시스템에서 GTSAM이 핵심 백엔드로 활용되고 있다.62 이는 분산 최적화, 데이터 공유, 로봇 간 루프 클로저 탐지 등 새로운 도전 과제를 제기한다. Hydra-Multi와 같은 프로젝트는 중앙 서버에서 GTSAM을 사용하여 여러 로봇의 정보를 융합하는 방식을 채택했다.63
- **이벤트 기반 비전 (Event-based Vision):** 연구자들은 고속 및 고동적 범위(High Dynamic Range, HDR) 환경에서 강점을 보이는 이벤트 카메라의 데이터를 처리하기 위해 GTSAM을 활용하고 있다. 이는 가우시안 프로세스(Gaussian Process) 등을 이용한 연속 시간 궤적 표현과, GTSAM을 엔진으로 하는 증분 MAP 추론을 결합하는 방식으로 이루어진다.64
- **학습과의 깊은 통합:** 딥러닝 기반의 프론트엔드(특징점 검출, 데이터 연관 등)와 GTSAM과 같은 고전적인 최적화 기반 백엔드를 결합하여, 학습의 강력한 표현력과 기하학 기반 최적화의 정밀성을 모두 취하려는 연구가 주류를 이루고 있다.65
- **제어 및 계획으로의 확장:** GTSAM의 창시자인 Frank Dellaert 교수의 최근 연구는 팩터 그래프의 개념을 최적 제어 및 모션 계획 문제로 확장하고 있다. 이는 '행동(action)' 자체를 일종의 추론 문제로 간주하는 새로운 시각을 제시하며, 팩터 그래프 패러다임의 적용 범위를 넓히고 있다.6

이러한 최신 연구 동향들은 GTSAM의 역할을 재조명한다. 최신 연구들은 GTSAM을 단순히 블랙박스 솔버로 사용하는 것을 넘어, GTSAM이 주창하는 **팩터 그래프라는 '언어'**를 사용하여 새롭고 복잡한 문제들을 '정의'하고 있다. C-SLAM은 거대한 분산 팩터 그래프 문제로, 이벤트 카메라는 비동기적 이벤트와 연속 시간 궤적을 잇는 팩터들로, 최적 제어는 비용 함수를 나타내는 팩터들로 모델링된다. 이처럼 팩터 그래프 패러다임은 현대 로보틱스가 마주한 다양한 추정 및 의사결정 문제들을 기술하는 공통되고 표현력 있는 '문법(grammar)'을 제공한다. 따라서 GTSAM의 가장 중요한 장기적 영향은 라이브러리 자체를 넘어, 로보틱스 문제 해결을 위한 통일된 사고방식으로서 팩터 그래프를 대중화했다는 점에 있을 수 있다. 이는 시공간에 걸쳐 이기종 정보를 복잡하고 동적인 방식으로 융합해야 하는 차세대 로보틱스 문제들을 해결하기 위한 강력한 정신적, 실용적 도구를 제공한다.



GTSAM은 로보틱스와 컴퓨터 비전 분야의 스무딩 및 매핑 문제를 해결하기 위해 탄생한 우아하고, 강력하며, 효율적인 C++ 라이브러리이다. 그 핵심적인 강점은 **팩터 그래프 패러다임**에 있으며, 이는 복잡한 로보틱스 문제를 직관적이면서도 이론적으로 견고하게 모델링할 수 있는 틀을 제공한다.16 특히 iSAM2를 통한 증분 최적화, 매니폴드 상에서의 IMU 사전적분과 같은 최첨단 알고리즘의 구현체는 GTSAM을 고성능 실시간 시스템 구축을 위한 필수적인 도구로 만들었다.10


GTSAM과 그 대안들 사이에서 선택해야 하는 실무자를 위해 다음과 같은 제언을 할 수 있다.

- **GTSAM을 선택해야 할 경우:**
  - SLAM, VIO, SfM과 같이, 사전 구현된 풍부한 팩터 라이브러리가 개발 시간을 크게 단축시켜 줄 수 있는 표준적인 로보틱스 문제를 다룰 때.
  - 문제가 잘 제약되어 있고, 실시간 증분 업데이트가 성능에 결정적일 때 (iSAM2의 활용).
  - 문제의 구조에 대한 깊고 직관적인 이해를 바탕으로 모델링하고 싶을 때.
  - MATLAB/Python을 이용한 신속한 프로토타이핑이 개발 워크플로우의 중요한 부분일 때.
- **대안(Ceres/g2o)을 고려해야 할 경우:**
  - 완전히 새로운 형태의 최적화 문제를 프로토타이핑하며, 자동 미분(Ceres) 기능이 개발 속도를 현저히 높여줄 수 있을 때.
  - 팀의 기존 기술 스택과 코드베이스가 이미 다른 생태계(예: Ceres)에 깊이 의존하고 있을 때.57
  - 게이지 자유도가 큰 문제를 다루면서 증분 최적화가 필요할 때, Ceres가 더 직관적인 해결 경로를 제공할 수 있다.


결론적으로 GTSAM은 단순한 최적화 솔버 그 이상이다. 그것은 로봇 인식 분야를 근본적으로 형성한 성공적인 연구 프로그램의 결정체이다. GTSAM의 미래는 대규모 협력 자율주행, 평생 학습 기반의 매핑(lifelong mapping), 그리고 고전 기하학과 딥러닝의 융합과 같이 새롭게 부상하는 도전 과제들에 팩터 그래프라는 우아한 언어를 통해 지속적으로 적용되는 데에 있다.35 GTSAM은 앞으로도 로보틱스 연구와 개발의 최전선에서 핵심적인 역할을 수행하며 그 가치를 증명해 나갈 것이다.


1. Factor Graphs for Robot Perception - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/319132170_Factor_Graphs_for_Robot_Perception
2. Rao-Blackwellized Particle Smoothing for Simultaneous Localization and Mapping - arXiv, accessed July 30, 2025, https://arxiv.org/abs/2306.03953
3. Factor Graphs for Navigation Applications: A Tutorial, accessed July 30, 2025, https://navi.ion.org/content/navi/71/3/navi.653.full.pdf
4. Sensors and Sensor Fusion Methodologies for Indoor Odometry: A Review - MDPI, accessed July 30, 2025, https://www.mdpi.com/2073-4360/14/10/2019
5. A brief introduction to Factor Graphs - UAL, accessed July 30, 2025, http://ingmec.ual.es/~jlblanco/papers/2020-introduction-factor-graphs_JLBlanco.pdf
6. What are Factor Graphs? - GTSAM, accessed July 30, 2025, https://gtsam.org/2020/06/01/factor-graphs.html
7. The Types of SLAM Algorithms - Medium, accessed July 30, 2025, https://medium.com/@nahmed3536/the-types-of-slam-algorithms-356196937e3d
8. Incremental Smoothing vs. Filtering for Sensor Fusion on an Indoor UAV - TU Chemnitz, accessed July 30, 2025, https://www.tu-chemnitz.de/etit/proaut/publications/ICRA13-ISF.pdf
9. Concurrent Filtering and Smoothing - Frank Dellaert, accessed July 30, 2025, https://dellaert.github.io/files/Kaess12fusion.pdf
10. borglab/gtsam: GTSAM is a library of C++ classes that ... - GitHub, accessed July 30, 2025, https://github.com/borglab/gtsam
11. Modeling Robot Motion - GTSAM 4.0.2 documentation, accessed July 30, 2025, https://gtsam-jlblanco-docs.readthedocs.io/en/latest/ModelingRobotMotion.html
12. Filtering vs Smoothing in Bayesian Estimation - Cross Validated - Stack Exchange, accessed July 30, 2025, https://stats.stackexchange.com/questions/243798/filtering-vs-smoothing-in-bayesian-estimation
13. g2o vs. Ceres: Optimizing Scan Matching in Cartographer SLAM - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/393587126_g2o_vs_Ceres_Optimizing_Scan_Matching_in_Cartographer_SLAM
14. On State Estimation in Multi-Sensor Fusion Navigation: Optimization and Filtering - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2401.05836
15. Factor Graphs for Robot Perception - CMU School of Computer ..., accessed July 30, 2025, https://www.cs.cmu.edu/~kaess/pub/Dellaert17fnt.pdf
16. Factor Graphs for Robot Perception - Now Publishers, accessed July 30, 2025, https://www.nowpublishers.com/article/Details/ROB-043
17. GTSAM | GTSAM is a BSD-licensed C++ library that implements sensor fusion for robotics and computer vision using factor graphs., accessed July 30, 2025, https://gtsam.org/
18. Factor Graphs and GTSAM: A Hands-on Introduction - GT Digital Repository, accessed July 30, 2025, https://repository.gatech.edu/server/api/core/bitstreams/b3606eb4-ce55-4c16-8495-767bd46f0351/content
19. Factor Graphs and GTSAM | GTSAM, accessed July 30, 2025, https://gtsam.org/tutorials/intro.html
20. 인자 그래프 - ML Note - 티스토리, accessed July 30, 2025, https://mldiary.tistory.com/73
21. gtsam::FactorGraph< FACTOR > Class Template Reference - the official ROS docs, accessed July 30, 2025, https://docs.ros.org/en/latest-available/api/gtsam/html/classgtsam_1_1FactorGraph.html
22. Welcome to Factor Graphs - Emma Benjaminson, accessed July 30, 2025, https://sassafras13.github.io/FactorGraphs/
23. Structure from Motion - CMSC426 Computer Vision, accessed July 30, 2025, https://cmsc426.github.io/gtsam/
24. Rao-Blackwellized particle smoothing for simultaneous localization and mapping - uu .diva, accessed July 30, 2025, https://uu.diva-portal.org/smash/get/diva2:1890402/FULLTEXT01.pdf
25. Tutorials - GTSAM 4.0.2 documentation - Read the Docs, accessed July 30, 2025, https://doc-gtsam.readthedocs.io/en/latest/Tutorials.html
26. gtsam/USAGE.md at develop - GitHub, accessed July 30, 2025, https://github.com/borglab/gtsam/blob/develop/USAGE.md
27. Creating new factor and variable types - GTSAM, accessed July 30, 2025, https://gtsam.org/doxygen/
28. gtsam: Getting started - ROS documentation, accessed July 30, 2025, http://docs.ros.org/en/melodic/api/gtsam/html/GettingStarted.html
29. Get Started | GTSAM, accessed July 30, 2025, https://gtsam.org/get_started/
30. gtsam/examples/SFMExample.cpp at master - GitHub, accessed July 30, 2025, https://github.com/devbharat/gtsam/blob/master/examples/SFMExample.cpp
31. Structure from Motion - GTSAM 4.0.2 documentation, accessed July 30, 2025, https://gtsam-jlblanco-docs.readthedocs.io/en/latest/StructureFromMotion.html
32. Factor Graphs for Robot Perception - Now Publishers, accessed July 30, 2025, https://www.nowpublishers.com/article/DownloadSummary/ROB-043
33. Publications - Frank Dellaert, accessed July 30, 2025, https://dellaert.github.io/publications/
34. 2.2. Pose2 iSAM - GTSAM by Example, accessed July 30, 2025, https://gtbook.github.io/gtsam-examples/Pose2ISAM2Example.html
35. From Square Root SAM to GTSAM: Factor Graphs in Robotics, accessed July 30, 2025, https://roboticsconference.org/2020/docs/keynote-TestOfTime-DellaertKaess.pdf
36. G2O vs GTSAM vs Ceres Solver from a programmer's perspective | by Jianzhu Huai, accessed July 30, 2025, https://medium.com/@jianzhuhuai0108/g2o-vs-gtsam-vs-ceres-solver-from-a-programmers-perspective-f45ac68a90fd
37. tussedrotten/sensor-fusion-example - GitHub, accessed July 30, 2025, https://github.com/tussedrotten/sensor-fusion-example
38. Docs | GTSAM, accessed July 30, 2025, https://gtsam.org/docs/
39. Advancing The Robotics Software Development Experience: Bridging Julia's Performance and Python's Ecosystem - arXiv, accessed July 30, 2025, https://arxiv.org/html/2406.03677v1
40. PoseSLAM - GTSAM 4.0.2 documentation - Read the Docs, accessed July 30, 2025, https://gtsam-jlblanco-docs.readthedocs.io/en/latest/PoseSLAM.html
41. 2. SLAM - GTSAM by Example, accessed July 30, 2025, https://gtbook.github.io/gtsam-examples/slam.html
42. 2D Pose SLAM in GTSAM - Piazza, accessed July 30, 2025, https://piazza.com/class_profile/get_resource/hbl3nsqea3z6uo/hf5dj0hcfey5fi
43. structure from motion | Reality Bytes, accessed July 30, 2025, https://realitybytes.blog/tag/structure-from-motion/
44. borglab/gtsfm: End-to-end SFM framework based on GTSAM - GitHub, accessed July 30, 2025, https://github.com/borglab/gtsfm
45. rahul-sb/VINS: Sensor Fusion using Extended Kalman Filter - GitHub, accessed July 30, 2025, https://github.com/rahul-sb/VINS
46. Motion Based Calibration of IMU and a Pose sensor - Google Groups, accessed July 30, 2025, https://groups.google.com/g/gtsam-users/c/91_DBJUcUYY
47. Sensor Fusion and Online Calibration of an Ambulatory Backpack System for Indoor Mobile Mapping - UC Berkeley EECS, accessed July 30, 2025, https://www2.eecs.berkeley.edu/Pubs/TechRpts/2016/EECS-2016-81.pdf
48. A Review of Multi-Sensor Fusion SLAM Systems Based on 3D LIDAR - MDPI, accessed July 30, 2025, https://www.mdpi.com/2072-4292/14/12/2835
49. robotics - IRIS, accessed July 30, 2025, https://iris.uniroma1.it/retrieve/e3835329-b25e-15e8-e053-a505fe0a3de9/Grisetti_Least-squares-optimization_2020.pdf
50. miniSAM: A Flexible Factor Graph Non-linear Least Squares Optimization Framework - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/1909.00903
51. g2o vs. Ceres: Optimizing Scan Matching in Cartographer SLAM - arXiv, accessed July 30, 2025, https://arxiv.org/html/2507.07142v1
52. A Comparison of Graph Optimization Approaches for Pose Estimation in SLAM - LAMoR, accessed July 30, 2025, https://lamor.fer.hr/images/50036607/2021-ajuric-comparison-mipro.pdf
53. (PDF) G2o: A general framework for graph optimization - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/224252449_G2o_A_general_framework_for_graph_optimization
54. Show HN: SymForce – Fast symbolic computation, code generation, and optimization | Hacker News, accessed July 30, 2025, https://news.ycombinator.com/item?id=31494328
55. A Practical Guide to Marginalization for Nonlinear Least Squares on Boxplus Manifolds - Evan Levine, accessed July 30, 2025, https://evanlev.github.io/marginalization.pdf
56. [Q] GTSAM vs G2O : r/robotics - Reddit, accessed July 30, 2025, https://www.reddit.com/r/robotics/comments/dm7cy0/q_gtsam_vs_g2o/
57. stereo vision SLAM, some questions. : r/SelfDrivingCars - Reddit, accessed July 30, 2025, https://www.reddit.com/r/SelfDrivingCars/comments/gldgh5/stereo_vision_slam_some_questions/
58. [Feature Request] Allow use of system libraries for 3rdparty codebases / Issue #292 / borglab/gtsam - GitHub, accessed July 30, 2025, https://github.com/borglab/gtsam/issues/292
59. rtabmap: g2o, GTSAM - Robotics Stack Exchange, accessed July 30, 2025, https://robotics.stackexchange.com/questions/94593/rtabmap-g2o-gtsam
60. Towards a Robust Back-End for Pose Graph SLAM - Niko Sünderhauf, accessed July 30, 2025, https://nikosuenderhauf.github.io/assets/papers/ICRA12-robustSLAM.pdf
61. Pose2SLAMExample_g2o and Pose3SLAMExample_g2o, the GN solver cannot proceed on some common benchmark datasets / Issue #846 / borglab/gtsam - GitHub, accessed July 30, 2025, https://github.com/borglab/gtsam/issues/846
62. Towards Collaborative Simultaneous Localization and Mapping: a Survey of the Current Research Landscape - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2108.08325
63. Collaborative Online Construction of 3D Scene Graphs with Multi-Robot Teams - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2304.13487
64. Asynchronous Optimisation for Event-based Visual Odometry - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2203.01037
65. A survey on real-time 3D scene reconstruction with SLAM methods in embedded systems - arXiv, accessed July 30, 2025, https://arxiv.org/pdf/2309.05349
66. SPRING 2023 GRASP On Robotics: Frank Dellaert, Georgia Tech - YouTube, accessed July 30, 2025, https://www.youtube.com/watch?v=6o8ATX3HQQE


---
layout: page
title: GICP 포인트 클라우드 정합
permalink: /sensors/point cloud/registrations/GICP
---


3차원(3D) 포인트 클라우드 정합(Point Cloud Registration)은 로보틱스, 자율 주행, 3D 모델 재구성, 의료 영상 분석 등 현대 공학 및 컴퓨터 과학의 여러 핵심 분야에서 필수불가결한 기술이다.1 이는 서로 다른 시점이나 시간에 취득된 복수의 3D 포인트 클라우드 데이터셋을 하나의 일관된 좌표계로 정렬하는 과정을 의미한다. 특히, LiDAR(Light Detection and Ranging) 센서의 보급이 확대되면서, 자율주행 차량이나 로봇이 자신의 위치를 추정하고 주변 환경의 지도를 동시에 작성하는 SLAM(Simultaneous Localization and Mapping) 기술과 주행 거리 및 경로를 추정하는 Odometry 기술에서 정합 알고리즘의 역할은 그 어느 때보다 중요해졌다.3

1992년 Besl과 McKay에 의해 제안된 ICP(Iterative Closest Point) 알고리즘은 그 단순성과 효율성으로 인해 지난 수십 년간 3D 정합 분야의 사실상 표준으로 자리매김했다.6 그러나 표준 ICP 알고리즘은 몇 가지 본질적인 한계를 내포하고 있다. 대표적으로, 정합을 시작하기 위한 초기 변환 추정치에 매우 민감하여 부정확한 초기값은 알고리즘을 국소 최적해(local minimum)에 빠뜨릴 위험이 크다.7 또한, 측정 노이즈나 이상치(outlier)에 취약하며, 두 포인트 클라우드가 부분적으로만 겹치는(partial overlap) 현실적인 상황에서는 성능이 급격히 저하될 수 있다.1

이러한 표준 ICP의 한계를 극복하고자 다양한 변형 알고리즘이 연구되었다. 그중 Point-to-Plane ICP는 오차 측정 방식을 개선하여 수렴 속도와 정확도를 크게 향상시킨 중요한 진보였다.6 그리고 2009년, Aleksandr Segal, Dirk Hähnel, Sebastian Thrun은 기존의 Point-to-Point 및 Point-to-Plane ICP를 하나의 통일된 확률적 프레임워크로 일반화한 Generalized-ICP(GICP)를 제안하였다.11 GICP는 포인트 클라우드에 내재된 불확실성과 국소적인 기하학적 구조를 명시적으로 모델링함으로써, 기존 방법들보다 훨씬 더 강건하고 정확한 정합을 가능하게 하였다.

본 보고서는 GICP 알고리즘의 핵심 원리, 특히 'Plane-to-Plane' 오차 측정 방식의 수학적 기반을 깊이 있게 분석하는 것을 목표로 한다. 이를 위해 먼저 ICP 알고리즘의 발전 과정을 살펴보고, GICP의 확률적 프레임워크와 목적 함수를 상세히 유도한다. 나아가 Point Cloud Library (PCL)와 같은 주요 라이브러리에서의 실제적 구현, 핵심 파라미터의 역할, 그리고 NDT(Normal Distributions Transform)와 같은 다른 주요 알고리즘과의 성능 비교 분석을 수행한다. 마지막으로, LiDAR SLAM을 포함한 주요 응용 분야와 Voxelized GICP(VGICP), Differentiable GICP(WGICP)와 같은 최신 연구 동향까지 망라함으로써, GICP에 대한 포괄적이고 심층적인 고찰을 제공하고자 한다.


GICP의 혁신성을 이해하기 위해서는 그 이전에 존재했던 ICP 알고리즘들의 원리와 한계를 명확히 파악하는 것이 선행되어야 한다. 이 장에서는 가장 기본적인 형태인 표준 ICP(Point-to-Point)부터 시작하여, 그 한계를 개선한 Point-to-Plane ICP까지의 발전 과정을 살펴본다.


표준 ICP 알고리즘, 또는 Point-to-Point ICP는 두 포인트 클라우드, 즉 소스 클라우드 $P = {p_i}$와 타겟 클라우드 $Q = {q_i}$를 정렬하기 위한 반복적인 접근법이다.1 알고리즘의 핵심 절차는 다음과 같이 두 단계로 구성된다.

1. **대응점 탐색 (Correspondence Search):** 현재의 변환 추정치 $(R_k, t_k)$를 사용하여 변환된 소스 클라우드의 각 점 pi′=Rkpi+tk에 대해, 타겟 클라우드 Q에서 유클리드 거리가 가장 가까운 점 qi를 찾아 대응점 쌍 $(p_i, q_i)$를 구성한다.14
2. **변환 추정 (Transformation Estimation):** 찾아낸 모든 대응점 쌍에 대해, 점들 간의 제곱 거리의 합을 최소화하는 새로운 강체 변환(Rigid Transformation) $(R_{k+1}, t_{k+1})$을 계산한다.6

이 과정에서 최소화하고자 하는 목적 함수는 다음과 같이 정의된다.
$$
E(R, t) = \sum_{i} \| (R p_i + t) - q_i \|^2
$$
이 목적 함수는 변환 행렬 R과 변환 벡터 t에 대해 닫힌 형태(closed-form)의 해를 가지며, 일반적으로 특이값 분해(Singular Value Decomposition, SVD)를 통해 효율적으로 풀 수 있다는 큰 장점이 있다.14

하지만 이러한 단순함은 명백한 한계를 동반한다. Point-to-Point 방식의 가장 큰 문제는 대응점을 찾는 과정에서 표면의 기하학적 구조를 전혀 고려하지 않는다는 점이다. 예를 들어, 평평한 벽면을 스캔한 두 포인트 클라우드를 정렬하는 경우를 생각해보자. 소스 클라우드의 한 점은 타겟 클라우드의 벽면 위 어떤 점과도 정렬될 수 있어야 한다. 그러나 Point-to-Point 방식은 오직 유클리드 거리가 가장 가까운 한 점만을 대응점으로 간주하기 때문에, 표면을 따라 자유롭게 미끄러지는(sliding) 움직임을 허용하지 않고 잘못된 방향으로 점들을 끌어당기게 된다. 이러한 특성은 특히 평면이나 원통과 같이 대칭적인 구조를 가진 환경에서 수렴 속도를 현저히 저하시키고, 부정확한 대응점 설정으로 인해 알고리즘을 국소 최적해에 빠뜨리는 주된 원인이 된다.6


Point-to-Point 방식의 한계를 극복하기 위해 Chen과 Medioni는 Point-to-Plane 이라는 개선된 오차 측정 방식을 제안했다.6 이 방법의 핵심 아이디어는 두 대응점 사이의 직접적인 유클리드 거리를 최소화하는 대신, 소스 포인트와 타겟 포인트에서의 접평면(tangent plane) 사이의 거리를 최소화하는 것이다.

구체적으로, 변환된 소스 포인트 pi′=Rpi+t와 그에 대응하는 타겟 포인트 qi가 주어졌을 때, 오차는 두 점의 차이 벡터 $(p'*i - q_i)$를 타겟 포인트 qi에서의 법선 벡터(normal vector) $n*{q_i}$에 투영한 거리로 정의된다.14 이를 통해 새롭게 정의된 목적 함수는 다음과 같다.
$$
E(R, t) = \sum_{i} \left( \left( (R p_i + t) - q_i \right) \cdot n_{q_i} \right)^2
$$
이 목적 함수는 기하학적으로 매우 중요한 의미를 가진다. 법선 벡터에 수직인 방향, 즉 접평면 방향으로의 오차는 목적 함수에 영향을 주지 않는다. 이는 알고리즘이 표면을 따라 자유롭게 미끄러지도록 허용하여, Point-to-Point 방식이 겪었던 문제를 효과적으로 해결한다. 그 결과, Point-to-Plane ICP는 특히 건물 내부나 가구와 같이 평면 구조가 많은 환경에서 Point-to-Point 방식에 비해 훨씬 빠른 수렴 속도와 높은 정합 정확도를 보인다.10

그러나 Point-to-Plane ICP 역시 완벽하지 않다. 첫째, 이 방법은 비대칭적(asymmetric)이다. 오직 타겟 포인트 클라우드의 법선 정보만을 사용하며, 소스 포인트 클라우드가 가진 기하학적 정보는 완전히 무시된다.13 만약 타겟 클라우드의 법선 추정이 노이즈로 인해 부정확하다면 정합 성능이 저하될 수 있다. 둘째, 목적 함수가 회전 행렬 

R에 대해 비선형적(non-linear)이므로, SVD와 같은 닫힌 형태의 해법을 사용할 수 없다. 대신 Levenberg-Marquardt(LM)나 Newton 방법과 같은 반복적인 비선형 최적화 기법을 사용해야 하며, 이는 각 반복 단계의 계산 복잡도를 증가시키는 요인이 된다.10 GICP는 바로 이러한 Point-to-Plane ICP의 한계를 극복하고, 두 방식의 장점을 통합하는 일반화된 프레임워크를 제시하면서 등장하게 된다.


GICP는 기존 ICP 변형들의 아이디어를 계승하면서도, 포인트 클라우드 정합 문제를 근본적으로 다른 시각, 즉 확률적 관점에서 재정의한다. GICP의 핵심은 각 포인트를 단순한 3차원 좌표가 아닌, 불확실성을 내포한 확률 분포로 모델링하고, 두 분포 간의 유사도를 최대화하는 변환을 찾는 것이다.11


GICP의 첫 단계는 포인트 클라우드의 각 점이 가진 불확실성과 그 주변의 국소적인 기하학적 구조를 정량적으로 모델링하는 것이다. LiDAR와 같은 3D 센서로 측정된 포인트들은 센서 자체의 노이즈, 표면의 반사율 변화, 샘플링 과정 등 여러 요인으로 인해 실제 표면과 약간의 오차를 가진다. GICP는 이러한 불확실성을 다루기 위해 각 점 pi와 그 주변의 국소적인 점 분포를 다변량 가우시안 분포(Multivariate Gaussian Distribution) $N(\mu, C)$로 근사한다.13

이 가우시안 분포의 공분산 행렬 C는 해당 점의 불확실성을 나타낼 뿐만 아니라, 그 점 주변의 기하학적 특징을 암시하는 매우 중요한 정보를 담고 있다.


GICP는 소스 클라우드 A의 각 점 ai와 타겟 클라우드 B의 각 점 bi에 대해 대칭적으로 공분산 행렬 $C_{a_i}$와 $C_{b_i}$를 계산한다.12 이 공분산 행렬은 일반적으로 해당 점의 k-최근접 이웃(k-Nearest Neighbors, k-NN)을 찾고, 이 이웃 점들의 공간적 분포를 분석하여 추정된다.19

이 공분산 행렬이 단순한 오차 범위를 넘어 국소 기하학의 인코더(Encoder)로 작동하는 방식은 다음과 같다.

1. 어떤 점의 k-NN 이웃 점들은 그 점 주변의 국소 표면(local surface) 위에 분포하게 된다.
2. 이 이웃 점들의 집합에 대해 주성분 분석(Principal Component Analysis, PCA)을 수행하면, 데이터가 퍼져있는 주된 방향과 그 정도를 나타내는 고유벡터(eigenvector)와 고유값(eigenvalue)을 얻을 수 있다. 이 정보가 바로 공분산 행렬에 담겨 있다.
3. 만약 국소 표면이 **평면(plane)**에 가깝다면, 이웃 점들은 평면을 따라 두 방향으로는 넓게 퍼져있고(큰 분산), 평면의 법선 방향으로는 매우 좁게 모여있을 것이다(작은 분산). 이는 공분산 행렬이 두 개의 큰 고유값(λ1,λ2)과 한 개의 매우 작은 고유값(λ3≈0)을 갖게 됨을 의미한다. 이때 가장 작은 고유값 λ3에 해당하는 고유벡터는 바로 그 국소 평면의 법선 벡터(normal vector)가 된다.
4. 만약 국소 표면이 **선(edge)**에 가깝다면, 이웃 점들은 선 방향으로만 길게 분포하고 나머지 두 방향으로는 거의 퍼져있지 않을 것이다. 이는 한 개의 큰 고유값과 두 개의 작은 고유값으로 나타난다.
5. 만약 점들이 구형으로 흩어져 있다면(예: 코너점), 세 개의 고유값이 비슷한 크기를 가질 것이다.

이처럼 공분산 행렬은 각 점의 불확실성뿐만 아니라, 그 점이 평면 위에 있는지, 선 위에 있는지, 혹은 복잡한 구조의 일부인지를 정량적으로 암호화하여 담고 있다. GICP는 이 풍부한 기하학적 정보를 정합 과정에 직접 활용함으로써 기존 방법들보다 훨씬 정교하고 강건한 성능을 달성한다.


GICP는 두 대응점 ai와 bi의 정합 오차를 계산할 때, 단순 유클리드 거리가 아닌 두 점의 확률 분포 간의 거리를 측정한다. 소스 클라우드의 점 ai는 가우시안 분포 $N(a_i, C_{a_i})$로, 타겟 클라우드의 점 bi는 $N(b_i, C_{b_i})$로 모델링된다.

강체 변환 T가 소스 클라우드에 적용되면, 변환된 점 Tai의 분포는 $N(T a_i, T C_{a_i} T^T)$가 된다. 이제 문제는 변환된 분포 $N(T a_i, T C_{a_i} T^T)$와 타겟 분포 N(bi,Cbi) 사이의 차이를 최소화하는 최적의 변환 T를 찾는 것이다.

두 분포 간의 차이, 즉 오차 벡터 di=bi−Tai는 두 독립적인 가우시안 분포의 차이이므로, 역시 가우시안 분포를 따른다. 그 분포는 $d_i \sim N(0, C_{b_i} + T C_{a_i} T^T)$이다. GICP는 이 오차 분포의 확률을 최대화하는 변환 T를 찾고자 하며, 이는 오차 벡터 di의 마할라노비스 거리(Mahalanobis distance) 제곱의 합을 최소화하는 것과 동일하다.20 따라서 GICP의 최종 목적 함수는 다음과 같이 정의된다.
$$
T^* = \arg\min_{T} \sum_{i} d_i^T (C_{b_i} + T C_{a_i} T^T)^{-1} d_i
$$
이 목적 함수는 GICP가 Point-to-Point와 Point-to-Plane ICP를 모두 포괄하는 일반화된 모델임을 명확히 보여준다.

- **Point-to-Point로의 환원:** 만약 모든 점의 불확실성을 등방성(isotropic)으로 가정하여 공분산 행렬을 모두 단위 행렬(Cai=Cbi=I)로 설정하면, (Cbi+TCaiTT)−1 항은 (I+TITT)−1=(I+I)−1=(1/2)I가 된다. 상수항을 무시하면 목적 함수는 ∑diTdi=∑∥bi−Tai∥2로 단순화되며, 이는 정확히 Point-to-Point ICP의 목적 함수와 일치한다.21
- **Point-to-Plane으로의 환원:** 만약 소스 클라우드는 이상적인 점(Cai=0)으로, 타겟 클라우드는 법선 방향으로만 불확실성이 매우 작은 평면($C_{b_i}$가 랭크-1에 가까운 행렬)으로 가정하면, 마할라노비스 거리는 법선 방향의 오차에만 매우 큰 가중치를 부여하게 되어 Point-to-Plane ICP와 유사한 효과를 낸다.
- **Plane-to-Plane의 구현:** GICP는 여기서 한 걸음 더 나아가, 소스와 타겟 양쪽 모두의 평면 구조 정보를 공분산 행렬에 반영한다. 최적화 과정은 소스 클라우드의 평면과 타겟 클라우드의 평면이 가장 잘 정렬되도록 유도하며, 이것이 바로 GICP가 'Plane-to-Plane' 정합으로 불리는 이유이다.11 이 대칭적인 접근 방식은 한쪽 스캔의 정보가 불완전하거나 노이즈가 많을 때 다른 쪽 스캔의 정보를 활용하여 정합의 안정성을 높이는 데 크게 기여한다.


GICP의 목적 함수는 변환 행렬 T의 회전 성분(보통 오일러 각도나 쿼터니언으로 매개변수화됨)에 대해 고도로 비선형적이다. 따라서 SVD와 같은 닫힌 형태의 해가 존재하지 않으며, 수치적 최적화 기법을 통해 반복적으로 해를 찾아야 한다.13

Segal의 원 논문과 PCL(Point Cloud Library) 구현에서는 주로 Newton 방법(정확히는 Gauss-Newton에 가까움)을 사용하여 이 비선형 최소 제곱 문제를 푼다.19 Newton 방법은 목적 함수의 1차 미분(자코비안)과 2차 미분(헤시안) 정보를 모두 사용하여 최적의 다음 단계를 결정한다. 2차 정보를 활용하기 때문에 수렴 영역에 들어서면 매우 빠른 속도로 최적해에 도달할 수 있다. 그러나 매 반복마다 헤시안 행렬을 계산하고 그 역행렬을 구하는 비용이 크다는 단점이 있다.

이러한 계산 비용 문제를 완화하기 위해 BFGS(Broyden–Fletcher–Goldfarb–Shanno)와 같은 유사-Newton(Quasi-Newton) 방법도 대안으로 사용될 수 있다.19 BFGS는 1차 미분 정보인 그래디언트만을 사용하여 헤시안 행렬을 근사적으로 갱신해 나간다. 이는 헤시안을 직접 계산하는 것보다 훨씬 효율적이며, 많은 경우 Newton 방법과 비슷한 성능을 보이면서도 더 안정적일 수 있다.20 PCL의 GICP 구현에서는 `useBFGS()` 함수를 통해 사용자가 이 두 최적화기 중에서 선택할 수 있도록 옵션을 제공한다.19


이론적 우수성에도 불구하고, 알고리즘의 실제 성능은 효율적인 구현과 주어진 데이터 및 응용에 맞는 적절한 파라미터 설정에 크게 좌우된다. 이 장에서는 가장 널리 사용되는 오픈소스 라이브러리인 PCL(Point Cloud Library)에서의 GICP 구현을 분석하고, 성능에 영향을 미치는 주요 파라미터들의 역할과 튜닝 전략을 논의한다. 또한, 실시간 성능을 목표로 하는 최적화된 라이브러리들을 소개한다.


PCL은 3D 포인트 클라우드 처리를 위한 포괄적인 기능을 제공하는 라이브러리로, GICP 알고리즘을 `pcl::GeneralizedIterativeClosestPoint` 클래스를 통해 표준화된 인터페이스로 제공한다.2 이 클래스는 PCL의 기본 정합 프레임워크인 `pcl::Registration`을 상속받아 일관된 사용법을 보장한다.23

PCL GICP의 내부 작동 흐름은 다음과 같다.

1. **입력 설정:** 사용자는 `setInputSource()`와 `setInputTarget()` 함수를 통해 소스 및 타겟 포인트 클라우드를 입력한다.
2. **공분산 계산:** 사용자가 외부에서 계산된 공분산 행렬을 `setSourceCovariances()`나 `setTargetCovariances()`로 제공하지 않으면, 클래스 내부에서 각 포인트의 k-NN을 찾아 공분산을 자동으로 계산한다.19
3. **반복 정합:** `align()` 함수가 호출되면, ICP의 주 루프가 시작된다.
   - **대응점 탐색:** 내부적으로 FLANN(Fast Library for Approximate Nearest Neighbors) 라이브러리에 기반한 k-d tree를 사용하여 변환된 소스 포인트에 대한 최근접 타겟 포인트를 효율적으로 탐색한다.22
   - **변환 최적화:** 찾아낸 대응점 쌍과 미리 계산된 공분산 행렬을 이용하여 GICP 목적 함수를 구성하고, 기본적으로 Newton 방법(또는 사용자가 BFGS를 선택한 경우)을 통해 목적 함수를 최소화하는 변환 업데이트량을 계산한다.19
   - **변환 갱신 및 수렴 확인:** 계산된 업데이트량을 현재 변환 행렬에 적용하여 갱신하고, 변환량의 크기가 미리 설정된 임계값(`TransformationEpsilon`)보다 작아지거나 최대 반복 횟수(`MaximumIterations`)에 도달하면 루프를 종료한다.
4. **결과 반환:** 최종적으로 수렴된 변환 행렬을 `getFinalTransformation()` 함수를 통해 얻을 수 있다.

이처럼 PCL은 GICP의 복잡한 수학적 세부 사항을 추상화하여 사용자가 몇 줄의 코드로 강력한 정합 기능을 활용할 수 있도록 지원한다.24


GICP의 이론적 모델과 실제 응용 사이의 간극을 메우는 것은 바로 사용자가 설정할 수 있는 파라미터들이다. 이 파라미터들은 알고리즘의 동작을 세밀하게 제어하여, 특정 데이터셋이나 응용 시나리오에 맞게 정확도, 속도, 강건성 간의 균형을 조절하는 역할을 한다. 따라서 이 파라미터들의 의미를 정확히 이해하고 튜닝하는 것은 GICP의 성능을 최대로 이끌어내는 데 매우 중요하다.

다음 표는 PCL GICP의 핵심 파라미터들과 그 역할, 그리고 일반적인 튜닝 전략을 정리한 것이다.

**Table 1: PCL GICP 주요 파라미터 가이드**

| 파라미터 (PCL 함수)                                          | 설명                                                         | 역할 및 튜닝 전략                                            | 관련 자료 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | --------- |
| `setCorrespondenceRandomness(int k)`                         | 공분산 계산 시 사용할 k-최근접 이웃의 수. 내부적으로 `k_correspondences_` 변수에 저장된다. | 이 값은 국소 표면의 기하학적 구조를 얼마나 넓은 범위에서 추정할지를 결정한다. `k`가 클수록 노이즈에 강하고 안정적인 공분산 추정이 가능하지만, 계산량이 증가하고 세밀한 구조가 뭉개질 수 있다. 반대로 `k`가 작으면 계산은 빠르지만 노이즈에 민감해진다. 일반적으로 포인트 클라우드의 밀도와 구조의 복잡성을 고려하여 10에서 30 사이의 값으로 설정한다. | 19        |
| `useBFGS()`                                                  | 기본 최적화기인 Newton 대신 BFGS를 사용하도록 설정.          | PCL 문서에서는 Newton 방법이 일반적으로 더 빠르고 정확하다고 언급하지만 19, 목적 함수가 매우 비선형적이거나 헤시안 행렬이 불안정한 경우 BFGS가 더 안정적인 수렴을 보일 수 있다.20 정합이 실패하거나 발산하는 경우, 최적화기를 변경하여 시도해 볼 가치가 있다. | 19        |
| `setMaximumOptimizerIterations(int max)`                     | 내부 최적화 단계(Newton/BFGS)의 최대 반복 횟수. `max_inner_iterations_`에 해당한다. | GICP의 각 반복(전체 ICP 루프) 내에서 수행되는 비선형 최적화의 최대 스텝 수를 제한한다. 높은 정밀도가 요구되고 계산 시간에 여유가 있다면 이 값을 늘릴 수 있고, 실시간성이 중요하다면 줄여서 타협할 수 있다. | 19        |
| `setRotationEpsilon(double eps)` / `setTransformationEpsilon(double eps)` | 전체 ICP 루프의 수렴 판단 기준. 연속된 변환 행렬 간의 차이가 이 값보다 작으면 수렴으로 간주한다. | 이 값은 정합의 정밀도와 계산 시간 사이의 트레이드오프를 결정한다. 너무 작은 값은 불필요한 반복을 유발하여 시간을 낭비하게 하고, 너무 큰 값은 최적해에 도달하기 전에 너무 일찍 수렴을 종료시켜 정밀도를 떨어뜨릴 수 있다. 보통 1e-5 ~ 1e-8 범위에서 설정된다. | 19        |
| `setSourceCovariances(...)` / `setTargetCovariances(...)`    | 외부에서 미리 계산된 공분산 행렬을 제공.                     | 공분산 계산은 GICP의 주요 계산 병목 중 하나이다. 만약 센서의 물리적 모델이나 다른 방법을 통해 더 정확하거나 빠르게 공분산을 계산할 수 있다면, 이 함수를 통해 외부에서 계산된 공분산 행렬을 주입하여 전체 정합 성능을 개선할 수 있다. | 19        |




표준 PCL의 GICP 구현은 개념 증명과 교육용으로는 훌륭하지만, 대부분 단일 스레드로 동작하기 때문에 수만 개 이상의 포인트를 가진 대규모 포인트 클라우드를 실시간으로 처리하기에는 성능적인 한계가 명확하다. 이러한 한계를 극복하기 위해 GICP의 성능을 극적으로 향상시킨 여러 최적화된 라이브러리들이 등장했다.

- **`fast_gicp`**: 이 라이브러리는 GICP의 실시간 적용을 목표로 개발되었으며, 다음과 같은 최적화 기법들을 적용한다.25
  - **멀티스레딩 (Multi-threading):** OpenMP를 사용하여 공분산 계산 및 기타 병렬화 가능한 부분들을 여러 CPU 코어에서 동시에 처리하여 속도를 높인다.
  - **복셀화 (Voxelization):** `fast_gicp`는 VGICP(Voxelized GICP)라는 변형을 구현한다. 이는 NDT처럼 공간을 복셀로 나누어 k-NN 탐색의 필요성을 없애고, 대신 복셀 내의 포인트 분포들을 집계하여 복셀의 대표 분포를 계산하는 방식이다. 이를 통해 k-NN 탐색이라는 가장 큰 병목을 제거한다.27
  - **GPU 가속:** CUDA를 활용하여 알고리즘의 많은 부분을 GPU에서 병렬로 실행하는 `FastVGICPCuda` 버전을 제공하여, CPU 기반 구현보다 훨씬 높은 처리량을 달성한다. `fast_gicp`는 CPU에서 약 70 FPS, GPU에서 약 120 FPS의 성능을 보고하기도 했다.25
- **`small_gicp`**: `fast_gicp`의 개발자가 PCL과 같은 무거운 라이브러리에 대한 의존성을 제거하고, 핵심 의존성을 Eigen 라이브러리 등으로 최소화하여 이식성과 사용 편의성을 높인 최신 라이브러리이다.29 헤더 전용(header-only)으로 제공되어 프로젝트에 쉽게 통합할 수 있으며, `fast_gicp`보다도 개선된 병렬 처리와 최적화를 통해 더 빠른 성능을 목표로 한다.

이러한 고성능 라이브러리들의 등장은 GICP가 더 이상 이론이나 오프라인 분석에만 머무르지 않고, 자율주행차나 로봇과 같이 실시간성과 계산 효율성이 극도로 중요한 실제 시스템에 탑재될 수 있는 길을 열어주었다는 점에서 큰 의의를 가진다.


GICP의 성능을 객관적으로 평가하기 위해서는 다른 주요 정합 알고리즘들과의 비교가 필수적이다. 이 장에서는 GICP를 그 직계 선행 기술인 Point-to-Plane ICP와, 그리고 완전히 다른 접근 방식을 사용하는 대표적인 알고리즘인 NDT(Normal Distributions Transform)와 비교하여 그 장단점을 심층적으로 분석한다.


GICP와 Point-to-Plane ICP는 모두 표면의 법선 정보를 활용하여 정합 성능을 높인다는 공통점이 있지만, 근본적인 철학에서 차이를 보인다. GICP는 소스와 타겟 양쪽 포인트 클라우드의 표면 구조를 모두 확률적으로 모델링하는 '대칭적(symmetric)' 접근 방식을 취하는 반면, Point-to-Plane ICP는 타겟 클라우드의 법선 정보만을 사용하는 '비대칭적(asymmetric)' 방식을 사용한다.12

이러한 대칭성 덕분에 GICP는 Point-to-Plane ICP보다 일반적으로 더 강건하고 정확한 결과를 제공한다.11 예를 들어, 한쪽 스캔 데이터에 노이즈가 심하거나, 특정 영역의 포인트가 희소하여 법선 추정이 불안정한 상황을 가정해보자. Point-to-Plane ICP는 이 부정확한 법선 정보에 의존하여 정합 오차를 계산하므로 성능이 저하될 수 있다. 반면, GICP는 양쪽 스캔의 정보를 모두 고려하므로, 한쪽의 정보가 불확실하더라도 다른 쪽의 더 깨끗하고 구조적인 정보를 활용하여 안정적인 정합을 수행할 수 있다. 이처럼 GICP의 확률적 모델은 데이터의 불확실성을 자연스럽게 통합하여 잘못된 대응점의 영향을 줄이고, 전반적인 정합의 신뢰도를 높인다.

**Table 2: ICP 계열 알고리즘 비교**

| 특성            | Point-to-Point ICP          | Point-to-Plane ICP             | Generalized-ICP (GICP)                   |
| --------------- | --------------------------- | ------------------------------ | ---------------------------------------- |
| **오차 측정**   | 점-점 유클리드 거리         | 점-평면 투영 거리              | 분포-분포 마할라노비스 거리              |
| **최적화**      | 선형 (SVD)                  | 비선형 (LM, Newton)            | 비선형 (Newton, BFGS)                    |
| **법선 필요**   | 아니오                      | 타겟 클라우드                  | 소스 & 타겟 클라우드 (내부 계산)         |
| **대칭성**      | 대칭                        | 비대칭                         | 대칭                                     |
| **강건성**      | 낮음                        | 중간                           | 높음                                     |
| **대표 사용처** | 간단한 형상, 최종 미세 조정 | 구조적 환경, 빠른 수렴 필요 시 | 노이즈가 많거나 복잡한 환경, 고정밀 SLAM |


GICP와 NDT는 둘 다 포인트 클라우드를 확률 분포의 집합으로 간주한다는 점에서 유사하지만, 그 분포를 모델링하는 방식에서 근본적인 차이를 보인다. 이 차이점은 두 알고리즘의 성능 특성을 결정하는 핵심 요인이 된다.

- **근본적 접근 방식의 차이:**
  - **GICP:** 각 **포인트**를 중심으로 그 주변의 k-NN 이웃들을 이용하여 국소적인 확률 분포(공분산)를 계산한다. 이는 데이터 자체의 구조를 따라가는 비구조적인(unstructured) 모델링 방식이다.
  - **NDT:** 공간을 일정한 크기의 **복셀(voxel) 그리드**로 먼저 분할하고, 각 복셀에 포함된 모든 포인트들을 대표하는 단 하나의 정규 분포를 계산한다.31 이는 공간을 먼저 구조화(structured)하고 데이터를 그 구조에 맞춰 요약하는 방식이다.

이러한 차이는 '구조화된 공간 분할(NDT)'과 '비구조화된 국소 기하 모델링(GICP)' 간의 대결로 볼 수 있다.

1. **NDT의 장단점:** NDT는 공간을 복셀로 나누기 때문에 대응점 탐색이 매우 빠르다. 특정 포인트가 어떤 복셀에 속하는지만 알면 그 복셀의 정규 분포와 바로 비교할 수 있기 때문이다. 이는 k-NN 탐색이라는 GICP의 가장 큰 계산 병목을 원천적으로 제거한다. 하지만 NDT는 복셀의 해상도(resolution)에 매우 민감하다는 치명적인 단점이 있다.27 해상도가 너무 크면 세밀한 기하학적 구조가 모두 뭉개져 버리고, 너무 작으면 각 복셀에 포함되는 포인트 수가 너무 적어져 안정적인 분포 추정이 불가능해진다. 또한, 복셀 경계에서 목적 함수가 불연속적으로 변할 수 있어 최적화 과정에 문제를 일으킬 수 있다.31
2. **GICP의 장단점:** GICP는 각 점을 중심으로 이웃을 탐색하므로, 복셀 해상도와 같은 인위적인 파라미터에 덜 민감하며, 각 점의 고유한 국소 기하학적 특징을 NDT보다 훨씬 정밀하게 모델링할 수 있다. 이는 일반적으로 더 높은 정확도로 이어진다. 그러나 모든 포인트에 대해 k-NN 탐색을 수행해야 하므로 계산 비용이 매우 높다는 것이 가장 큰 단점이다.27
3. **결론적 트레이드오프:** 결국 NDT는 '속도'를 얻기 위해 '정밀도'와 '파라미터 민감도'라는 대가를 치르는 경향이 있고, GICP는 '정밀도'를 얻기 위해 '속도'를 희생하는 경향이 있다. 이러한 상호 보완적인 특성 때문에, 어떤 상황에서는 NDT가, 다른 상황에서는 GICP가 더 나은 성능을 보인다.34 흥미롭게도, 최근의 연구 동향은 이 두 가지 접근법을 융합하는 방향으로 나아가고 있다. 앞서 언급된 VGICP는 NDT의 복셀 기반 탐색 속도와 GICP의 정밀한 분포 모델링 방식을 결합하여 두 알고리즘의 장점을 모두 취하려는 대표적인 시도이다.27

**Table 3: GICP와 NDT 비교 분석**

| 특성              | Generalized-ICP (GICP)                              | Normal Distributions Transform (NDT)                     |
| ----------------- | --------------------------------------------------- | -------------------------------------------------------- |
| **기본 단위**     | 포인트 (Point)                                      | 복셀 (Voxel)                                             |
| **분포 모델링**   | 각 포인트 중심의 국소 이웃 분포                     | 각 복셀 내 포인트들의 단일 분포                          |
| **주요 병목**     | k-최근접 이웃 탐색 (k-NN Search)                    | 복셀 내 공분산 계산, 그래디언트 계산                     |
| **주요 파라미터** | 이웃 수 (`k`)                                       | 복셀 해상도 (Voxel Resolution)                           |
| **장점**          | 높은 정확도, 해상도 비의존적, 국소 기하 정밀 모델링 | 빠른 대응점 탐색, 특징 없는 평면에서 강함, 미분 가능     |
| **단점**          | k-NN 탐색으로 인한 높은 계산 비용                   | 복셀 해상도에 매우 민감, 경계 불연속성, 구조 뭉개짐 위험 |


GICP의 강력한 성능은 다양한 실제 응용 분야, 특히 로봇 공학과 자율 주행 분야에서 그 가치를 입증하고 있다. 이 장에서는 GICP가 LiDAR SLAM 및 Odometry에서 수행하는 핵심적인 역할을 살펴보고, 실시간 성능을 위한 최적화 기법 및 학습 기반으로 발전하는 최신 연구 동향을 조망한다.


LiDAR SLAM 시스템은 크게 두 부분으로 구성된다: 로봇의 연속적인 움직임을 추정하는 프론트엔드(front-end)와, 누적된 오차를 보정하고 전역적으로 일관된 지도를 생성하는 백엔드(back-end)이다. GICP는 이 두 부분 모두에서 핵심적인 역할을 수행한다.

- **LiDAR Odometry (프론트엔드):** 로봇이 움직이면서 LiDAR 센서는 연속적으로 3D 스캔 데이터를 생성한다. LiDAR Odometry는 현재 스캔과 바로 이전 스캔(또는 최근 몇 개의 스캔으로 구성된 지역 지도)을 정합하여 그 사이의 상대적인 움직임(회전 및 이동)을 추정하는 과정이다. GICP는 그 높은 정확도와 강건성 덕분에 이 스캔-투-스캔(scan-to-scan) 또는 스캔-투-맵(scan-to-map) 정합의 핵심 엔진으로 널리 사용된다.34 GICP 기반 Odometry는 특히 지하 광산과 같이 GPS 신호가 없고 특징이 부족한 비구조적인 환경에서 안정적인 성능을 발휘한다.34
- **Loop Closure (백엔드):** 로봇이 장시간 이동하다 보면 필연적으로 오차가 누적되어 추정된 경로가 실제 경로에서 벗어나게 된다(drift). 이때, 로봇이 과거에 방문했던 장소를 다시 인식하게 되면(loop closure), 현재 스캔과 과거에 생성된 지도 부분을 정합하여 그동안 누적된 오차를 계산하고, 이를 전체 경로에 분배하여 보정할 수 있다. GICP는 이 loop closure 정합 과정에서 시간적으로 멀리 떨어진 두 데이터셋을 정확하게 정렬하는 데 사용되어, 전역적으로 일관된 지도를 구축하는 데 결정적인 기여를 한다.34


GICP의 가장 큰 실용적 장벽은 높은 계산 비용, 특히 k-NN 탐색에 소요되는 시간이다. 이 문제를 해결하기 위한 연구는 GICP를 실제 로봇에 탑재하기 위한 필수적인 과정이며, 크게 두 가지 방향으로 진행되고 있다.

1. **Voxelized GICP (VGICP):** 앞서 비교 분석에서 언급했듯이, VGICP는 GICP의 가장 큰 병목인 k-NN 탐색을 NDT의 복셀화 아이디어로 대체한 것이다.27 VGICP는 먼저 포인트 클라우드를 복셀 그리드로 나누고, 각 복셀에 속한 포인트들의 공분산들을 집계하여 복셀의 대표 공분산을 계산한다. 그 후, 대응 관계를 포인트 단위가 아닌 복셀 단위로 찾음으로써 탐색 공간을 크게 줄여 속도를 획기적으로 향상시킨다.27 이 방식은 GICP의 정확도를 최대한 유지하면서 NDT의 속도 이점을 결합한 하이브리드 접근법으로, 실시간 LiDAR SLAM에서 매우 유망한 기술로 평가받고 있다.

2. **GPU 가속:** GICP 알고리즘의 많은 부분, 예를 들어 모든 포인트에 대한 공분산 계산, 대응점 탐색, 목적 함수의 각 항 계산 등은 본질적으로 병렬 처리에 매우 적합하다. 이러한 특성을 활용하여 CPU의 여러 코어를 사용하는 멀티스레딩 25을 넘어, 수천 개의 코어를 가진 GPU(Graphics Processing Unit)를 사용하여 연산을 가속화하는 연구가 활발히 진행 중이다.28

   `fast_gicp`와 같은 라이브러리는 CUDA를 이용한 GPU 가속 버전을 제공하며, 이를 통해 일반적인 CPU 환경에서는 불가능했던 수준의 실시간 성능(예: 100 FPS 이상)을 달성할 수 있음을 보여주었다.25


최근 3D 비전 분야의 가장 큰 흐름은 전통적인 기하학 기반의 접근법과 딥러닝 기반의 학습 접근법을 융합하는 것이다. GICP 역시 이러한 흐름에 맞춰 진화하고 있으며, 이는 GICP의 패러다임을 '정밀한 기하학적 모델링'에서 '데이터 기반의 학습 가능한 강건성'으로 전환시키고 있다.

이러한 패러다임 전환의 핵심에는 **Differentiable GICP**라는 개념이 있다.

1. **GICP의 근본적 취약점:** GICP는 순수하게 기하학적 정보에만 의존하기 때문에, 기하학적으로 설명되지 않는 오차 요인, 예를 들어 도로 위의 다른 움직이는 차량, 보행자, 센서의 반사로 인한 허상(ghost points)과 같은 동적 객체나 심각한 이상치(outlier)에는 여전히 취약하다.
2. **미분 가능성(Differentiability)의 도입:** 이 문제를 해결하기 위해, GICP의 전체 정합 과정, 즉 입력 포인트 클라우드로부터 최종 변환 행렬을 출력하기까지의 모든 계산을 미분 가능한 함수로 근사하거나 재구성하려는 시도가 등장했다.39
3. **Weighted GICP (WGICP)의 작동 원리:** GICP가 미분 가능해지면, 각 입력 포인트가 최종 변환 추정 결과에 얼마나 영향을 미쳤는지(즉, 손실 함수에 대한 그래디언트)를 역전파(backpropagation)를 통해 계산할 수 있다. 이 그래디언트 정보를 이용하여, 어떤 포인트가 정합에 '중요한' 기여를 하고 어떤 포인트가 '방해'가 되는지를 판단하는 신경망을 훈련시킬 수 있다.39 이 훈련된 네트워크는 새로운 포인트 클라우드가 입력되었을 때 각 포인트에 대한 '신뢰도 가중치(confidence weight)'를 출력한다. GICP는 이 가중치를 목적 함수에 통합하여, 가중치가 높은(신뢰도 높은) 포인트의 오차는 크게 반영하고 가중치가 낮은(신뢰도 낮은, 예: 동적 객체 위의 점) 포인트의 오차는 작게 반영하여 최적화를 수행한다. 이를 **Weighted GICP (WGICP)**라고 한다.39
4. **전망:** WGICP는 단순히 이상치를 제거하는 기존의 휴리스틱한 방법을 넘어, 데이터로부터 직접 강건성을 학습하는 새로운 차원의 접근법이다. 이는 GICP가 정적인 환경뿐만 아니라, 사람이나 차량이 혼재하는 복잡하고 동적인 실제 환경에서도 높은 신뢰성을 가지고 작동할 수 있는 가능성을 열어준다. 또한, 가중치가 매우 낮은 포인트들을 아예 계산에서 제외함으로써 계산 효율성을 높이는 부수적인 효과도 얻을 수 있다.39


Generalized-ICP(GICP)는 3D 포인트 클라우드 정합 분야에서 중요한 이론적, 실용적 진보를 이룬 알고리즘이다. GICP는 기존의 Point-to-Point 및 Point-to-Plane ICP를 하나의 통일된 확률적 프레임워크로 통합하고 일반화하였다. 소스와 타겟 양쪽 스캔 데이터의 국소적 평면 구조를 공분산 행렬로 대칭적으로 모델링하고, 이들 분포 간의 마할라노비스 거리를 오차 측정 기준으로 사용함으로써, GICP는 측정 노이즈, 샘플링 밀도 변화, 그리고 불완전한 데이터에 대한 강건성을 획기적으로 향상시켰다. 이는 3D 정합 기술의 정확도와 신뢰성을 한 단계 끌어올린 결정적인 기여로 평가된다.

본 보고서는 GICP의 수학적 원리부터 시작하여, PCL과 같은 라이브러리에서의 실제 구현, 주요 파라미터의 역할, 그리고 NDT와 같은 다른 알고리즘과의 심층적인 성능 비교를 통해 GICP에 대한 다각적인 분석을 제공하였다. 또한, LiDAR SLAM 및 Odometry와 같은 핵심 응용 분야에서의 역할을 조명하고, VGICP, GPU 가속, 그리고 학습 기반의 WGICP와 같은 최신 연구 동향을 통해 GICP가 어떻게 진화하고 있는지를 살펴보았다.

GICP의 발전 과정은 포인트 클라우드 정합 기술의 연구 초점이 순수한 '정확도' 향상에서 '실시간성'과 '강건성' 확보로 이동하고 있음을 명확히 보여준다. 미래의 정합 기술은 VGICP와 같이 속도와 정확도의 균형을 맞춘 하이브리드 접근법과, WGICP처럼 딥러닝을 결합하여 환경의 의미론적 정보(semantic information)를 이해하고 동적 요소를 능동적으로 처리하는 방향으로 더욱 발전할 것이다. 궁극적으로 GICP와 그 후속 연구들은 기하학적 정합과 학습 기반 인식이 긴밀하게 융합된, 더욱 빠르고, 정확하며, 지능적인 차세대 3D 인식 시스템의 핵심 구성 요소로 자리매김할 것으로 전망된다.


1. What is Iterative Closest Point (ICP)? | Activeloop Glossary, accessed August 7, 2025, https://www.activeloop.ai/resources/glossary/iterative-closest-point-icp/
2. Registration with the Point Cloud Library - ais.uni-bonn.de, accessed August 7, 2025, https://www.ais.uni-bonn.de/papers/RAM_2015_Holz_PCL_Registration_Tutorial.pdf
3. LiDAR Point Cloud Generation for SLAM Algorithm Evaluation - MDPI, accessed August 7, 2025, https://www.mdpi.com/1424-8220/21/10/3313
4. 3D LiDAR SLAM – Scan Matching Explained - Ignitarium, accessed August 7, 2025, https://ignitarium.com/3d-lidar-slam-scan-matching-explained/
5. A Review of Simultaneous Localization and Mapping Algorithms Based on Lidar - MDPI, accessed August 7, 2025, https://www.mdpi.com/2032-6653/16/2/56
6. An Improved Large Planar Point Cloud Registration Algorithm - MDPI, accessed August 7, 2025, https://www.mdpi.com/2079-9292/13/14/2696
7. Robust iterative closest point algorithm based on global reference point for rotation invariant registration - PMC - PubMed Central, accessed August 7, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC5703502/
8. An Iterative Closest Points Algorithm for Registration of 3D Laser Scanner Point Clouds with Geometric Features - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/319072034_An_Iterative_Closest_Points_Algorithm_for_Registration_of_3D_Laser_Scanner_Point_Clouds_with_Geometric_Features
9. Generalized-ICP | Request PDF - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/371535553_Generalized-ICP
10. Linear Least-Squares Optimization for Point-to-Plane ICP Surface Registration ∑ - NUS Computing, accessed August 7, 2025, https://www.comp.nus.edu.sg/~lowkl/publications/lowk_point-to-plane_icp_techrep.pdf
11. [PDF] Generalized-ICP - Semantic Scholar, accessed August 7, 2025, [https://www.semanticscholar.org/paper/Generalized-ICP-Segal-H%C3%A4hnel/b352b3a7f1068b2d562ba12a446628397dfe8a77](https://www.semanticscholar.org/paper/Generalized-ICP-Segal-Hähnel/b352b3a7f1068b2d562ba12a446628397dfe8a77)
12. Generalized-ICP: Aleksandr V. Segal Dirk Haehnel Sebastian Thrun | PDF - Scribd, accessed August 7, 2025, https://www.scribd.com/document/514815771/Generalized-ICP
13. Generalized-ICP - Robotics, accessed August 7, 2025, https://www.roboticsproceedings.org/rss05/p21.pdf
14. Iterative Closest Point (ICP) for 3D Explained with Code, accessed August 7, 2025, https://learnopencv.com/iterative-closest-point-icp-explained/
15. Analysis of Iterative Closest Point - Neha Das, accessed August 7, 2025, https://neha191091.github.io/files/icp-final-poster.pdf
16. Tutorial: Iterative Closest Point (ICP) - land-of-kain, accessed August 7, 2025, http://land-of-kain.de/docs/icp/
17. Point-to-plane error between two surfaces. | Download Scientific Diagram - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/figure/Point-to-plane-error-between-two-surfaces_fig1_228571031
18. Generalized-ICP | Request PDF - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/221344436_Generalized-ICP
19. pcl::GeneralizedIterativeClosestPoint ... - Point Cloud Library (PCL), accessed August 7, 2025, https://pointclouds.org/documentation/classpcl_1_1_generalized_iterative_closest_point.html
20. Generalized ICP - Wiki, accessed August 7, 2025, https://wiki.hanzheteng.com/algorithm/slam/generalized-icp
21. avsegal/gicp: Generalized ICP reference implementation - GitHub, accessed August 7, 2025, https://github.com/avsegal/gicp
22. pcl/registration/gicp.h Source File - Point Cloud Library, accessed August 7, 2025, https://pointclouds.org/documentation/gicp_8h_source.html
23. Module registration - Point Cloud Library (PCL), accessed August 7, 2025, https://pointclouds.org/documentation/group__registration.html
24. GICP usage in PCL - c++ - Stack Overflow, accessed August 7, 2025, https://stackoverflow.com/questions/51871464/gicp-usage-in-pcl
25. koide3/fast_gicp: A collection of GICP-based fast point cloud registration algorithms - GitHub, accessed August 7, 2025, https://github.com/koide3/fast_gicp
26. fast_gicp - ROS Index, accessed August 7, 2025, https://index.ros.org/r/fast_gicp/
27. Voxelized GICP for Fast and Accurate 3D Point Cloud Registration, accessed August 7, 2025, https://staff.aist.go.jp/shuji.oishi/assets/papers/preprint/VoxelGICP_ICRA2021.pdf
28. Accelerating Nearest Neighbor Search in 3D Point Cloud Registration on GPUs | Request PDF - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/388877886_Accelerating_Nearest_Neighbor_Search_in_3D_Point_Cloud_Registration_on_GPUs
29. koide3/small_gicp: Efficient and parallel algorithms for point cloud registration [C++, Python], accessed August 7, 2025, https://github.com/koide3/small_gicp
30. small_gicp/README.md at master - GitHub, accessed August 7, 2025, https://github.com/koide3/small_gicp/blob/master/README.md
31. Evaluation of 3D Registration Reliability and Speed - A Comparison of ICP and NDT - CiteSeerX, accessed August 7, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=0185f6fb7a7a7859daf8ae326b485e9891a7461d
32. Comparative Analysis of 3D LiDAR Scan-Matching Methods for State Estimation of Autonomous Surface Vessel - MDPI, accessed August 7, 2025, https://www.mdpi.com/2077-1312/11/4/840
33. A Method Coupling NDT and VGICP for Registering UAV-LiDAR and LiDAR-SLAM Point Clouds in Plantation Forest Plots - MDPI, accessed August 7, 2025, https://www.mdpi.com/1999-4907/15/12/2186
34. Robust GICP-Based 3D LiDAR SLAM for Underground Mining Environment - MDPI, accessed August 7, 2025, https://www.mdpi.com/1424-8220/19/13/2915
35. Why ICP is much faster than NDT in front_end? / Issue #127 / koide3/hdl_graph_slam, accessed August 7, 2025, https://github.com/koide3/hdl_graph_slam/issues/127
36. Robust GICP-Based 3D LiDAR SLAM for Underground Mining Environment - ResearchGate, accessed August 7, 2025, https://www.researchgate.net/publication/334159341_Robust_GICP-Based_3D_LiDAR_SLAM_for_Underground_Mining_Environment
37. LOCUS 2.0: Robust and Computationally Efficient Lidar Odometry for Real-Time 3D Mapping - JPL Robotics, accessed August 7, 2025, https://www-robotics.jpl.nasa.gov/media/documents/2205.11784.pdf
38. GLIM: versatile and extensible point cloud-based 3D localization and mapping framework - GitHub, accessed August 7, 2025, https://github.com/koide3/glim
39. WGICP: Differentiable Weighted GICP-Based Lidar Odometry - arXiv, accessed August 7, 2025, https://arxiv.org/pdf/2209.09777
40. [2209.09777] WGICP: Differentiable Weighted GICP-Based Lidar Odometry - arXiv, accessed August 7, 2025, https://arxiv.org/abs/2209.09777
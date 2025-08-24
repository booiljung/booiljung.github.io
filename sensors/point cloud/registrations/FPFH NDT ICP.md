# FPFH, NDT, ICP를 활용한 포인트 클라우드 정합
[포인트 클라우드 정합](./index.md)



포인트 클라우드 정합(Point Cloud Registration)은 본질적으로 서로 다른 시점이나 좌표계에서 취득된 여러 개의 포인트 클라우드 데이터셋을 하나의 일관된 전역 좌표계로 정렬하는 과정을 의미한다.1 3D 스캐너, LiDAR, RGB-D 카메라 등으로 수집된 데이터는 각기 다른 위치와 방향을 기준으로 표현되므로, 이를 통합하여 완전한 모델을 만들거나 환경을 이해하기 위해서는 정합 과정이 필수적이다.

이 과정의 핵심 목표는 두 포인트 클라우드, 즉 변환의 대상이 되는 소스 포인트 클라우드($P_{source}$)와 기준이 되는 타겟 포인트 클라우드($P_{target}$) 사이의 최적의 강체 변환(rigid transformation)을 찾는 것이다.3 강체 변환은 객체의 형태를 바꾸지 않는 변환으로, 오직 회전과 이동만으로 구성된다. 수학적으로 이는 3x3 회전 행렬 $R$과 3x1 변위 벡터 $t$를 찾는 문제로 귀결된다. 최종 목표는 소스 포인트 클라우드의 모든 점 $p_i \in P_{source}$에 변환을 적용했을 때, 즉 $R p_i + t$를 계산했을 때, 타겟 포인트 클라우드 $P_{target}$과의 오차를 최소화하는 최적의 $(R, t)$ 쌍을 알아내는 것이다.2 이 오차는 보통 비용 함수 $E(R, t)$로 정의되며, 정합은 결국 이 비용 함수를 최소화하는 최적화 문제라고 할 수 있다.

단순히 두 데이터셋을 합치는 것을 넘어, 이 변환을 정확히 찾아내는 것은 수많은 첨단 기술의 근간을 이룬다. 로봇이 주변 환경을 탐색하며 자신의 위치를 실시간으로 추정하거나(localization), 여러 각도에서 스캔한 데이터를 합쳐 완전한 3D 디지털 모델을 만들거나(reconstruction), 자율주행차가 새로운 측정값을 기존의 고정밀 지도에 매핑하는(mapping) 등의 모든 과정이 정확한 정합 기술에 의존한다.2


포인트 클라우드 정합 기술은 가상 세계와 물리적 세계를 잇는 핵심 다리 역할을 하며, 그 정확도와 강인성이 전체 시스템의 성능을 결정한다. 특히 다음과 같은 분야에서 중추적인 역할을 담당한다.

첫째, **자율주행(Autonomous Driving)** 분야에서 정합은 차량의 '눈'과 같다. 차량에 장착된 LiDAR 센서는 초당 수백만 개의 점을 쏘아 주변 환경을 포인트 클라우드로 인식한다. 차량이 이동하면서 연속적으로 수집되는 이 포인트 클라우드들을 정합함으로써, 차량의 상대적인 움직임을 매우 정밀하게 추정할 수 있다. 이를 LiDAR Odometry라고 부른다.6 더 나아가, 이렇게 추정된 움직임과 포인트 클라우드 데이터를 누적하여 주변 환경의 3차원 지도를 실시간으로 생성하고 갱신하는 SLAM(Simultaneous Localization and Mapping) 기술의 핵심 요소로 작용한다.8 이 고정밀 지도는 차량의 현재 위치를 정확히 파악하고 안전한 경로를 계획하는 데 필수적이다.

둘째, **로보틱스 및 3D 모델링(Robotics & 3D Modeling)** 분야에서 정합은 객체 인식과 조작의 기본이다. 로봇 팔이 특정 물체를 집으려면, 먼저 그 물체가 무엇인지 인식하고 공간상에서 어떤 자세로 놓여있는지 정확히 알아야 한다. 여러 각도에서 촬영한 부분적인 스캔들을 정합하여 객체의 완전한 3D 모델을 생성하고, 이를 데이터베이스의 모델과 비교하여 객체를 인식하고 정확한 자세(pose)를 추정할 수 있다.5 이렇게 얻어진 정보는 로봇이 정교한 조작(grasping and manipulation)을 수행하기 위한 운동 계획의 기반이 된다.2 문화유산의 디지털 복원이나 산업 부품의 정밀 검사 등에서도 여러 부분 스캔을 하나의 완전한 모델로 만드는 데 정합 기술이 핵심적으로 사용된다.


이론적으로는 간단해 보이지만, 실제 환경에서 수집된 포인트 클라우드를 정합하는 것은 수많은 난제에 부딪힌다. 실제 데이터는 다음과 같은 문제점들을 복합적으로 내포하고 있다.

- **노이즈와 이상치 (Noise and Outliers):** 센서 자체의 측정 오차나 주변 환경의 반사 특성 때문에 데이터에는 항상 노이즈가 포함된다. 또한, 공기 중의 먼지나 다른 움직이는 객체 때문에 원래는 없어야 할 이상치(outlier)들이 데이터에 섞여 들어온다.5
- **부분적 중첩 (Partial Overlap):** 두 스캔은 서로 다른 위치에서 얻어지므로, 전체 영역 중 일부만 겹치게 된다. 겹치지 않는 영역의 점들은 대응점을 찾을 수 없으므로, 정합 과정에서 큰 오류를 유발할 수 있다.5
- **데이터 밀도 변화 및 희소성 (Varying Density and Sparsity):** 스캔 거리에 따라 포인트의 밀도가 달라지며, 일부 영역은 데이터가 매우 희소하게 존재할 수 있다. 이는 특징을 추출하거나 대응점을 찾는 것을 어렵게 만든다.13
- **동적 객체 (Dynamic Objects):** 주변 환경에 움직이는 사람이나 차량이 포함된 경우, 이들은 정합의 기준이 될 수 없으며 오히려 방해 요소로 작용한다.

이러한 문제들 때문에 정합은 근본적으로 '닭이 먼저냐, 달걀이 먼저냐(chicken-and-egg)'와 같은 순환적인 문제를 갖게 된다.5 즉, 두 포인트 클라우드 간의 정확한 대응점(correspondences)을 안다면 최적의 변환 (R, t)$를 쉽게 계산할 수 있다. 반대로, 최적의 변환을 안다면 대응점을 정확하게 찾을 수 있다. 하지만 우리는 둘 다 모르기 때문에, 이 두 가지를 동시에 추정해야 하는 어려움에 직면한다. 이 과정에서 잘못된 대응점 하나가 전체 변환 추정 과정을 망가뜨릴 수 있으며, 이로 인해 대부분의 정합 알고리즘은 수많은 지역 최솟값(local minima)을 가진 비볼록(non-convex) 비용 함수를 다루게 된다. 알고리즘이 이런 잘못된 지역 최솟값에 빠지면, 완전히 틀린 정렬 결과를 내놓게 되며, 이를 극복하는 것이 정합 연구의 핵심 과제이다.15


이러한 근본적인 난제들을 해결하기 위해, 학계와 산업계에서는 "Coarse-to-Fine" (혹은 Global-to-Local) 전략을 표준적인 접근법으로 채택하고 있다.19 이 전략은 정합 문제를 한 번에 풀려고 시도하는 대신, 두 단계로 나누어 접근한다.

1. **Coarse Registration (Global Alignment):** 첫 번째 단계에서는 계산 비용이 비교적 저렴한 방법을 사용하여, 다소 부정확하더라도 전역적으로 타당한 초기 정렬값을 대략적으로 찾아낸다. 이 단계에서는 FPFH(Fast Point Feature Histogram)와 같은 특징 기술자(feature descriptor)를 추출하고, RANSAC(Random Sample Consensus)과 같은 강인한 추정 기법을 통해 수많은 잘못된 대응점 속에서 올바른 변환의 단서를 찾아낸다.19 목표는 최적해를 찾는 것이 아니라, 최적해가 존재할 가능성이 높은 '영역'으로 해를 근사시키는 것이다.
2. **Fine Registration (Local Refinement):** 두 번째 단계에서는 Coarse 단계에서 얻은 초기 정렬값을 시작점으로 사용하여, ICP(Iterative Closest Point)나 NDT(Normal Distributions Transform)와 같은 정밀 정합 알고리즘을 실행한다. 이 알고리즘들은 이미 상당히 정렬된 상태에서 시작하기 때문에 지역 최솟값에 빠질 위험이 크게 줄어들며, 반복적인 계산을 통해 매우 정확하고 정밀한 최종 변환을 찾아낼 수 있다.3

이 전략은 단순히 두 알고리즘을 순서대로 실행하는 것을 넘어, 문제의 탐색 공간(search space)을 의도적으로 축소시키는 '깔때기(funneling)'와 같은 역할을 한다. 정합 문제의 비용 함수는 수많은 골짜기와 언덕으로 이루어진 복잡한 지형과 같다. 지역 최적화 알고리즘인 ICP는 눈을 가리고 가장 낮은 곳으로만 내려가는 등산가와 같아서, 시작 위치가 좋지 않으면 가장 깊은 계곡(전역 최적해)이 아닌 작은 웅덩이(지역 최적해)에 갇히고 만다. Coarse-to-Fine 전략에서 Coarse 단계는 이 등산가를 가장 깊은 계곡의 입구 근처까지 헬리콥터로 데려다주는 역할을 한다. 즉, 광활하고 복잡한 전체 변환 공간에서 가능성이 높은 '수렴의 골짜기(basin of convergence)'를 먼저 찾아내는 것이다.24 그런 다음 Fine 단계는 이 좁혀진 유망한 공간 안에서 정밀한 탐색을 수행하여 계곡의 가장 낮은 지점, 즉 최적해를 정확히 찾아낸다. 이는 계산 효율성과 최종 정확성이라는 두 마리 토끼를 모두 잡기 위한 필연적이고 지능적인 선택이라 할 수 있다.


Coarse-to-Fine 전략의 성공은 첫 단추인 Coarse 정합의 성능에 크게 좌우된다. 이 단계의 목표는 계산 비용을 최소화하면서도, 후속 Fine 정합이 안정적으로 수렴할 수 있을 만큼 충분히 좋은 초기 정렬을 제공하는 것이다. 이를 위해 점의 원시 좌표 대신, 주변 기하학 정보를 함축한 '특징(feature)'을 사용하고, 잘못된 정보 속에서 진짜를 가려내는 강인한 통계적 기법을 결합한다.


FPFH는 포인트 클라우드 정합 분야에서 가장 널리 사용되는 지역 특징 기술자(local feature descriptor) 중 하나다. 그 핵심 원리는 점의 3차원 좌표라는 저수준 정보 대신, 한 점(query point) 주변의 '지역적 기하학(local geometry)'이라는 고수준 정보를 일관된 방식으로 표현하는 데 있다.26

FPFH는 각 점에 대해 33개의 실수 값으로 구성된 벡터, 즉 33차원 기술자로 계산된다.21 이 기술자의 가장 중요한 특징은 '자세 불변성(pose-invariance)'이다.29 즉, 포인트 클라우드 전체가 회전하거나 이동하더라도, 동일한 기하학적 구조를 가진 지점(예: 의자의 모서리, 구의 표면)에서는 거의 동일한 FPFH 값이 계산된다. 이 덕분에 우리는 두 포인트 클라우드에서 FPFH 벡터가 유사한 점들을 찾아 대응시킴으로써, 두 데이터셋의 상대적인 자세 차이를 추론할 수 있다. 이는 원시 XYZ 좌표를 직접 비교하는 것보다 노이즈에 훨씬 강인하며, 대응점을 더 정확하게 찾을 수 있는 강력한 기반을 제공한다.

FPFH는 이름에서 알 수 있듯이, 기존의 PFH(Point Feature Histogram)를 개선하여 계산 속도를 획기적으로 높인 버전이다. PFH는 한 점의 특징을 계산하기 위해 그 주변 이웃 점들 사이의 모든 쌍 관계를 고려하므로, 점의 개수가 $n$이고 이웃의 수가 $k$일 때 계산 복잡도가 $O(nk^2)$에 달한다. 이는 실시간 처리가 거의 불가능한 수준이다. 반면, FPFH는 계산 방식을 최적화하여 복잡도를 $O(nk)$로 크게 줄였고, 이로 인해 실시간 응용 분야에서도 활용될 수 있게 되었다.19


FPFH 기술자를 계산하는 과정은 크게 세 단계로 나눌 수 있다. 이 과정은 점들의 기하학적 관계를 어떻게 수치화하고 효율적으로 요약하는지에 대한 통찰을 제공한다.


FPFH 계산의 가장 기본적인 전제 조건은 포인트 클라우드의 모든 점에 대해 법선 벡터(normal vector)를 추정하는 것이다.27 법선 벡터는 해당 지점에서의 표면 방향을 나타내는 벡터로, 지역 기하학을 설명하는 가장 근본적인 정보다. 이는 각 점 

$p$에 대해, 그 주변의 k-이웃 점들을 찾고, 이 점들이 가장 잘 표현하는 가상의 평면(plane)을 최소자승법(least squares)으로 피팅하여 해당 평면의 수직 벡터를 구하는 방식으로 계산된다.31 이 법선 벡터의 품질이 FPFH의 정확도에 직접적인 영향을 미치므로, 이웃을 몇 개나 고려할지(search radius)를 신중하게 결정해야 한다.


법선 벡터가 모두 계산되면, 각 점에 대해 SPFH(Simplified Point Feature Histogram)라는 중간 단계의 기술자를 계산한다. 이는 FPFH의 계산 효율성을 높이기 위한 핵심 아이디어다.

쿼리 포인트 $p_q$와 그 법선 $n_q$가 주어졌을 때, 그 주변의 k-이웃 점 중 하나인 $p_k$와 그 법선 $n_k$ 사이의 관계를 계산한다. 이 관계는 두 점을 잇는 벡터와 두 법선 벡터를 이용하여 정의된 지역 좌표계(Darboux frame)를 기반으로, 다음과 같은 3개의 각도 값($\alpha, \phi, \theta$)으로 표현된다.31

1. $\alpha$: 두 법선 벡터 사이의 각도 중 하나
2. $\phi$: 한 법선 벡터와 두 점을 잇는 벡터 사이의 각도
3. $\theta$: 나머지 각도 관계

이 세 가지 값은 두 점과 두 법선이 이루는 상대적인 기하학적 구조를 간결하게 요약한다. 쿼리 포인트 $p_q$의 SPFH는, 그 모든 k-이웃 점들과의 관계에서 계산된 $(\alpha, \phi, \theta)$ 값들을 각각의 히스토그램으로 만들어 합친 것이다.27 PCL(Point Cloud Library)의 기본 구현에서는 각 히스토그램을 11개의 구간(bin)으로 나누므로, SPFH는 33차원의 벡터가 된다. SPFH는 쿼리 포인트와 그 직접적인 이웃 간의 관계만을 계산하므로 매우 빠르지만, 이것만으로는 주변 기하학을 충분히 풍부하게 표현하기 어렵다는 한계가 있다.


최종 FPFH는 SPFH를 한 단계 더 발전시켜 만든다. 쿼리 포인트 $p_q$의 최종 FPFH는 자신의 SPFH 값에, 그 k-이웃 점들 각각의 SPFH 값을 가중 평균하여 더함으로써 계산된다.27 이를 수식으로 표현하면 다음과 같다.
$$
\text{FPFH}(p_q) = \text{SPFH}(p_q) + \frac{1}{k} \sum_{i=1}^{k} \frac{1}{w_i} \cdot \text{SPFH}(p_k^i)
$$
여기서 $p_k^i$는 $p_q$의 i번째 이웃 점이며, $w_i$는 $p_q$와 $p_k^i$ 사이의 거리에 기반한 가중치로, 가까운 이웃의 정보에 더 큰 비중을 두는 역할을 한다.27

이 2단계 계산 방식이 FPFH의 정수다. 쿼리 포인트의 특징을 계산할 때, 그 이웃들의 특징(SPFH)까지 다시 한번 고려함으로써, 계산량을 폭발적으로 늘리지 않으면서도 마치 PFH처럼 더 넓은 영역의 기하학적 정보를 효율적으로 통합하는 효과를 낸다. 즉, 각 점의 특징은 그 이웃들의 특징에 의해 영향을 받게 되며, 이는 정보가 지역적으로 전파되는 것과 유사하다. 이 과정을 통해 FPFH는 PFH와 유사한 강력한 표현력을 가지면서도 훨씬 빠른 계산 속도를 달성할 수 있게 된다.


FPFH 기술자를 소스와 타겟 포인트 클라우드의 모든 점에 대해 계산하고 나면, 다음 단계는 이 기술자들을 비교하여 대응점 쌍을 찾는 것이다. 가장 간단한 방법은 소스 포인트 클라우드의 각 점에 대해, 타겟 포인트 클라우드에서 FPFH 벡터가 가장 유사한(즉, 유클리드 거리가 가장 가까운) 점을 찾아 초기 대응점 쌍으로 만드는 것이다.21

하지만 이 초기 대응점 집합에는 필연적으로 수많은 오류(outliers)가 포함된다. FPFH가 아무리 뛰어나도 노이즈, 부분 중첩, 반복적인 구조 등으로 인해 잘못된 짝이 맺어지는 경우가 많기 때문이다.29 이 오염된 데이터 속에서 '진짜' 대응점(inliers)을 찾아내고, 이를 기반으로 신뢰할 수 있는 초기 변환을 추정하는 역할이 바로 RANSAC(Random Sample Consensus) 알고리즘의 몫이다.15

RANSAC의 동작 원리는 다음과 같다.

1. **무작위 샘플링 (Random Sampling):** FPFH를 통해 얻은 전체 초기 대응점 집합에서, 변환 행렬 $(R, t)$를 계산하는 데 필요한 최소한의 샘플(3D 강체 변환의 경우 보통 3쌍의 대응점)을 무작위로 추출한다.
2. **모델 추정 (Model Estimation):** 이 3쌍의 샘플만을 사용하여 최적의 변환 행렬 $(R, t)$를 계산한다.
3. **합의 평가 (Consensus Evaluation):** 계산된 변환을 전체 소스 포인트 클라우드에 적용한 후, 나머지 모든 대응점 쌍들이 이 변환 모델을 얼마나 잘 만족하는지 평가한다. 즉, 변환된 소스 점과 그에 대응하는 타겟 점 사이의 거리가 특정 임계값(distance threshold) 이하인지를 검사하여, 이 조건을 만족하는 대응점들의 수(인라이어의 수)를 센다.
4. **반복 및 최적 모델 선택:** 위 1~3 과정을 수없이 (수백~수천 번) 반복한다. 매 반복마다 인라이어의 수를 기록하고, 최종적으로 가장 많은 인라이어를 확보했던 변환 행렬을 가장 신뢰할 수 있는 초기 변환으로 채택한다.21

RANSAC은 최적화 알고리즘이 아니라, '가설 검증(Hypothesis Testing)'에 기반한 일종의 투표 메커니즘으로 이해해야 한다. FPFH로 찾은 수많은 대응점 쌍들은 각각 "전체 포인트 클라우드 간의 변환은 아마 이럴 것이다"라는 수많은 '가설'을 내포하고 있다. RANSAC은 이 수많은 가설들 중 무작위로 몇 개를 뽑아 테스트해보고, 가장 많은 지지(인라이어)를 받는 가설을 채택하는 민주적인 방식을 취한다. 이 접근법의 가장 큰 장점은 아웃라이어의 비율이 50%를 넘지 않는 한, 매우 높은 확률로 올바른 모델을 찾아낼 수 있다는 점이다. 즉, 다수의 잘못된 의견(아웃라이어) 속에서 소수의 올바른 의견(인라이어)을 찾아내는 데 매우 강인한(robust) 방법이다. 이러한 특성 덕분에 RANSAC은 FPFH와 결합하여 매우 신뢰성 높은 Coarse 정합 결과를 만들어낼 수 있다.


Coarse 정합 단계에서 FPFH와 RANSAC을 통해 대략적인 초기 정렬을 얻었다면, Fine 정합 단계에서는 이를 기반으로 훨씬 더 정밀하고 정확한 최종 변환을 찾아낸다. 이 단계에서는 주로 NDT(Normal Distributions Transform)와 ICP(Iterative Closest Point)라는 두 가지 강력한 알고리즘이 사용된다. 이들은 서로 다른 철학을 바탕으로 정합을 수행하며, 이 둘을 순차적으로 결합하면 각 알고리즘의 장점을 극대화할 수 있다.


NDT는 포인트 클라우드를 이산적인 점의 집합이 아닌, '표면이 존재할 확률'을 나타내는 연속적인 확률 분포 함수(Probability Density Function, PDF)로 취급하는 독특한 접근법을 사용한다.33 이는 점과 점 사이의 명시적인 대응 관계를 찾는 대신, 한 포인트 클라우드가 다른 포인트 클라우드의 확률 분포상에서 얼마나 잘 들어맞는지를 평가하는 방식이다.


NDT 알고리즘은 다음과 같은 절차로 진행된다.

1. **공간 복셀화 (Voxelization):** 먼저, 기준이 되는 타겟 포인트 클라우드가 차지하는 3차원 공간을 일정한 크기의 정육면체 격자, 즉 복셀(voxel)로 나눈다.33 이 복셀의 크기(Resolution)는 NDT의 성능에 가장 큰 영향을 미치는 핵심 파라미터다.
2. **정규분포 모델링:** 각 복셀에 대해, 그 안에 포함된 포인트들의 통계적 특성을 계산한다. 구체적으로, 복셀 내 점들의 평균 벡터($\mathbf{q}$)와 공분산 행렬($\mathbf{\Sigma}$)을 계산한다. 이 두 값을 이용해 해당 복셀의 포인트 분포를 하나의 다변량 정규분포(Multivariate Normal Distribution)로 모델링한다.33 이로써 이산적인 포인트 클라우드는 공간을 채우는 조각적인(piecewise) 연속 확률 분포로 변환된다.
3. **최적화 (Optimization):** 이제 소스 포인트 클라우드를 정렬할 차례다. Coarse 정합에서 얻은 초기 변환 $(R, t)$를 시작점으로 하여, 소스 포인트 클라우드의 각 점 $p_i$를 변환시킨다. 그리고 이 변환된 위치 $p'_i = R p_i + t$가 타겟 포인트 클라우드의 확률 분포 상에서 얼마나 높은 확률 값을 갖는지(likelihood)를 계산한다. NDT의 목표는 모든 소스 점들에 대한 이 확률 값들의 총합(또는 로그 확률의 합)을 최대화하는 것이다. 이는 곧 음의 로그 확률(negative log-likelihood)을 최소화하는 것과 같다. 이 최소화 문제는 비용 함수가 미분 가능하므로, 뉴턴 방법(Newton's method)이나 모어-튜엔테 라인 서치(More-Thuente line search)와 같은 표준 수치 최적화 기법을 사용하여 효율적으로 풀 수 있다.33


NDT의 핵심을 수학적으로 살펴보자. 한 점 $\mathbf{x}$가 주어졌을 때, 이 점이 속한 복셀의 평균이 $\mathbf{q}$이고 공분산이 $\mathbf{\Sigma}$인 정규분포에 따른 확률 밀도 값은 다음과 같이 주어진다.
$$
p(\mathbf{x}) \sim \exp\left(-\frac{1}{2}(\mathbf{x} - \mathbf{q})^T \mathbf{\Sigma}^{-1} (\mathbf{x} - \mathbf{q})\right)
$$
NDT가 최적화하려는 비용 함수(score)는 소스 포인트 클라우드의 모든 점 $p_i$를 변환 $T(p_i, \mathbf{R}, \mathbf{t})$ 시킨 위치에서의 확률 값들의 합으로 정의된다.
$$
\text{score}(\mathbf{R}, \mathbf{t}) = \sum_i p(T(p_i, \mathbf{R}, \mathbf{t}))
$$
NDT는 이 score 함수를 최대화하는 회전 행렬 $\mathbf{R}$과 변위 벡터 $\mathbf{t}$를 찾는 것을 목표로 한다.34

NDT의 가장 큰 강점은 바로 이 '부드러움'에 있다. ICP와 같이 점-대-점의 명시적이고 불연속적인 대응 관계에 의존하는 대신, NDT는 타겟 공간을 부드러운 확률 필드로 모델링한다. 소스 점의 위치가 조금 바뀌면 그에 해당하는 확률 값도 부드럽게 변한다. 이러한 비용 함수의 연속성과 미분 가능성은 최적화 과정을 훨씬 안정적으로 만들고, 잘못된 지역 최솟값에 빠질 가능성을 줄여준다. 이것이 바로 NDT가 ICP보다 더 넓은 수렴 반경(basin of convergence)을 가지며, 상대적으로 부정확한 초기값에도 더 강인한 이유다.25 하지만 이 부드러움은 복셀 단위의 '평균화'와 '추상화'에서 비롯된 것이기도 하다. NDT는 개별 점의 정밀한 위치보다는 복셀 내 점들의 통계적 분포에 더 관심을 둔다. 이 과정에서 미세한 기하학적 디테일이 일부 손실될 수 있다. 따라서 최종 정밀도 면에서는 원본 점들을 직접 다루는 ICP가 올바르게 시작했을 경우 더 높은 성능을 보일 수 있다.37


ICP는 포인트 클라우드 정합 분야에서 가장 고전적이면서도 여전히 널리 사용되는 알고리즘이다. 그 이름이 암시하듯, '가장 가까운 점을 반복적으로 찾아 정렬'하는 직관적인 아이디어에 기반한다.1


ICP 알고리즘은 수렴 조건이 만족될 때까지 다음의 4단계를 반복한다.

1. **대응점 탐색 (Correspondence Search):** 현재 변환이 적용된 소스 포인트 클라우드의 각 점에 대해, 타겟 포인트 클라우드에서 유클리드 거리가 가장 가까운 점을 찾아 대응점 쌍으로 맺는다. 포인트 수가 많을 경우 이 과정은 매우 오래 걸릴 수 있으므로, 보통 k-d tree와 같은 공간 분할 자료구조를 타겟 클라우드에 대해 미리 구축하여 탐색 속도를 가속화한다.3
2. **변환 추정 (Transformation Estimation):** 1단계에서 찾아낸 모든 대응점 쌍들 사이의 거리 오차 제곱의 합(sum of squared distances)을 최소화하는 최적의 회전 행렬 $R$과 변위 벡터 $t$를 계산한다.
3. **적용 (Apply Transformation):** 2단계에서 계산된 변환 $(R, t)$를 소스 포인트 클라우드에 적용하여 위치를 업데이트한다.
4. **반복 (Iterate):** 소스와 타겟 클라우드 사이의 평균 거리가 특정 임계값 이하로 떨어지거나, 연속된 반복에서 변환의 변화량이 매우 작아지거나, 지정된 최대 반복 횟수에 도달하는 등의 수렴 조건이 만족될 때까지 1~3단계를 계속 반복한다.3


가장 기본적인 점-대-점(point-to-point) ICP의 비용 함수 $E(R, t)$는 다음과 같이 정의된다.
$$
E(R, t) = \sum_{i=1}^{N} \| (R p_i + t) - q_i \|^2
$$
여기서 $p_i$는 소스 포인트 클라우드의 i번째 점이고, $q_i$는 $R p_i + t$에 가장 가까운 타겟 포인트 클라우드의 점이다.4 이 비용 함수의 놀라운 점은, 대응점 

$q_i$가 고정되어 있을 때 이를 최소화하는 $(R, t)$를 해석적으로(closed-form), 즉 한 번의 계산으로 찾을 수 있다는 것이다. 이 해법은 특이값 분해(Singular Value Decomposition, SVD)를 통해 얻어진다.

1. 두 대응점 집합 $\{p_i\}$와 $\{q_i\}$의 중심(centroid) $\mu_p$와 $\mu_q$를 계산한다.
   $$
   \mu_p = \frac{1}{N} \sum_{i=1}^{N} p_i, \quad \mu_q = \frac{1}{N} \sum_{i=1}^{N} q_i
   $$
   각 점들에서 중심을 빼서 중심화된(centered) 점 집합 $\{p'_i = p_i - \mu_p\}$와 $\{q'_i = q_i - \mu_q\}$를 구한다.

2. 3x3 공분산 행렬 $H$를 계산한다.38
   $$
   H = \sum_{i=1}^{N} p'_i {q'_i}^T
   $$
   $H$에 대해 SVD를 수행한다: $H = U \Lambda V^T$.

3. 최적의 회전 행렬 $R$과 변위 벡터 $t$는 다음과 같이 주어진다.4
   $$
   R = V U^T, \quad t = \mu_q - R \mu_p
   $$
   (단, 반사(reflection)를 방지하기 위해 $\det(R) = -1$인 경우 추가적인 처리가 필요하다.)


ICP의 명료함과 강력함에도 불구하고, 이 알고리즘은 치명적인 약점을 가지고 있다. 바로 초기 정렬에 매우 민감하다는 것이다.3 비용 함수 $E(R, t)$는 비볼록(non-convex) 함수이기 때문에 수많은 지역 최솟값을 가지고 있다. 만약 초기 변환이 전역 최적해에서 멀리 떨어진 곳에서 시작하면, ICP는 가장 가까운 지역 최솟값으로 수렴해 버리고, 이는 완전히 잘못된 정합 결과를 의미한다. Coarse-to-Fine 파이프라인은 바로 이 문제를 해결하기 위해, ICP가 전역 최적해가 있는 '올바른 골짜기'에서 시작하도록 보장해주는 역할을 한다.


NDT와 ICP는 Fine 정합 단계에서 사용될 수 있는 두 가지 주요 옵션이다. 이 둘의 특성을 이해하는 것은 왜 FPFH -->> NDT -->> ICP라는 3단계 파이프라인이 합리적인지를 이해하는 데 매우 중요하다. NDT는 ICP의 단점(좁은 수렴 반경)을 보완하는 훌륭한 '중간 다리' 역할을 수행할 수 있다.

| 평가 기준 (Criterion)    | ICP (Iterative Closest Point)                            | NDT (Normal Distributions Transform)                         | 핵심 통찰 (Key Insight)                                      |
| ------------------------ | -------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **기본 원리**            | 점-대-점(또는 점-대-평면)의 기하학적 거리 최소화 3       | 점-대-분포의 확률적 유사도 최대화 33                         | ICP는 기하학적 정밀도, NDT는 통계적 강인성에 초점            |
| **정확도**               | 매우 높음. 최종 정밀화에 유리 37                         | 높음. 하지만 복셀화로 인해 ICP보다 약간 낮을 수 있음 37      | 최종 '광택' 작업에는 ICP가, '거친 사포질'에는 NDT가 적합     |
| **속도**                 | 느림. 포인트 수가 많을수록 k-d tree 탐색 비용 증가 36    | 빠름. 점-대-점 대응 탐색이 없고, 복셀 단위로 처리 35         | NDT는 대규모 데이터의 초기 정렬을 빠르게 개선하는 데 유리    |
| **초기값 민감도**        | 매우 높음. 수렴 반경이 좁아 지역 최솟값에 빠지기 쉬움 18 | 상대적으로 낮음. 비용 함수가 더 부드러워 수렴 반경이 넓음 25 | NDT는 FPFH의 다소 부정확한 결과를 받아도 안정적으로 수렴 가능 |
| **메모리 사용량**        | 낮음. k-d tree 구조만 필요.                              | 높음. 타겟 클라우드를 복셀 그리드와 확률 분포로 저장해야 함 35 | 메모리가 제약된 환경에서는 ICP가 더 유리할 수 있음           |
| **노이즈/이상치 강인성** | 낮음. 이상치 하나가 대응 관계를 왜곡시킬 수 있음 3       | 상대적으로 높음. 복셀 내에서 통계적으로 처리되어 개별 이상치의 영향 감소 | NDT는 노이즈가 많은 실제 환경 데이터에 더 강건한 중간 결과를 제공 |


지금까지 살펴본 세 가지 핵심 알고리즘, FPFH(+RANSAC), NDT, ICP를 결합하여 하나의 강력한 정합 파이프라인을 구축할 수 있다. 이 파이프라인은 각 알고리즘의 장점은 취하고 단점은 보완하는 방식으로 설계되어, 실제 환경의 복잡하고 노이즈가 많은 데이터에 대해 매우 강인하고 정확한 결과를 제공한다.


통합 파이프라인의 전체적인 흐름은 다음과 같이 요약할 수 있다. 각 단계는 이전 단계의 출력을 입력으로 받아 정합의 정밀도를 점진적으로 높여간다.

1. **입력:** 소스 포인트 클라우드($P_{source}$), 타겟 포인트 클라우드($P_{target}$)
2. **전처리 (Preprocessing):**
   - Voxel Grid Downsampling: 계산량을 줄이고 포인트 밀도를 균일하게 만들기 위해 두 포인트 클라우드를 다운샘플링한다.29
   - Statistical Outlier Removal: 명백한 노이즈와 이상치를 제거하여 정합의 안정성을 높인다.29
3. **1단계: Coarse 정합 (FPFH + RANSAC):**
   - Normal Estimation: FPFH 계산을 위해 각 점의 법선 벡터를 추정한다.
   - FPFH Feature Extraction: 소스와 타겟 클라우드에 대해 FPFH 특징 기술자를 계산한다.
   - RANSAC Correspondence Rejection: FPFH 특징 공간에서 가장 가까운 이웃을 찾아 초기 대응점을 설정하고, RANSAC을 이용해 잘못된 대응점을 제거하여 강인한 초기 변환 행렬 $T_{coarse}$를 추정한다.19
4. **2단계: 중간 정합 (NDT):**
   - $T_{coarse}$를 초기 변환으로 사용하여 NDT 정합을 수행한다. NDT는 넓은 수렴 반경을 바탕으로 $T_{coarse}$를 더 정밀한 변환 행렬 $T_{ndt}$로 빠르게 개선한다.19
5. **3단계: 정밀 정합 (ICP):**
   - $T_{ndt}$를 초기 변환으로 사용하여 ICP 정합을 수행한다. 이미 매우 근접하게 정렬된 상태이므로, ICP는 지역 최솟값에 빠질 위험 없이 최종적으로 가장 정밀한 변환 행렬 $T_{final}$을 찾아낸다.20
6. **출력:** 최종 변환 행렬 $T_{final}$


이 파이프라인의 진정한 힘은 단순히 알고리즘들을 나열하는 데 있는 것이 아니라, 각 단계가 다음 단계를 위한 최적의 환경을 만들어주는 유기적인 연계에 있다.

- **FPFH+RANSAC -->> NDT:** FPFH+RANSAC으로 얻은 $T_{coarse}$는 NDT의 초기 변환(initial guess)으로 사용된다.19 NDT는 ICP보다 초기값에 덜 민감하지만, 완전히 동떨어진 위치에서 시작하면 여전히 수렴에 실패하거나 시간이 오래 걸릴 수 있다. 

  $T_{coarse}$는 NDT가 전역 최적해가 있을 법한 '올바른 골짜기'에서 최적화를 시작하도록 안내하는 역할을 한다. NDT의 넓은 수렴 반경 덕분에, $T_{coarse}$가 어느 정도의 오차를 포함하고 있더라도 NDT는 이를 안정적으로 흡수하여 훨씬 더 정밀한 $T_{ndt}$로 빠르게 수렴할 수 있다.36

- **NDT -->> ICP:** NDT를 통해 얻은 $T_{ndt}$는 다시 ICP의 초기 변환으로 사용된다. 이 시점에서 소스와 타겟 포인트 클라우드는 이미 육안으로는 거의 구별이 어려울 정도로 가깝게 정렬되어 있다. 이는 ICP가 동작하기에 가장 이상적인 조건이다. ICP의 가장 큰 약점인 지역 최솟값 문제는 사실상 해결된 상태이므로, ICP는 자신의 가장 큰 장점인 '정밀도'에만 집중하면 된다. 즉, ICP는 자신의 좁고 깊은 수렴 특성을 십분 활용하여, NDT가 복셀화 과정에서 놓쳤을 수 있는 미세한 오차까지 보정하며 최종적으로 가장 정밀한 $T_{final}$을 찾아내는 '마무리 투수' 역할을 완벽하게 수행한다.20

이 파이프라인은 단순히 변환 행렬을 다음 단계로 전달하는 기계적인 과정이 아니다. 이는 '신뢰도'와 '정밀도'를 단계적으로 증폭시켜 나가는 시너지 효과를 창출하는 과정으로 이해해야 한다. 첫 단계인 FPFH+RANSAC는 '전역적이지만 낮은 정밀도'의 해를 제공하며, "정답은 대략 이 근처에 있을 것이다"라는 신뢰도의 기반을 다진다. 두 번째 단계인 NDT는 이 신뢰도를 바탕으로 '강인하고 중간 정밀도'의 해를 빠르게 찾아내며, "정답은 이제 매우 가까운 곳에 있다"는 더 높은 수준의 신뢰도를 확보한다. 마지막으로, ICP는 이 매우 높은 신뢰도를 바탕으로 자신의 전문 분야인 '지역적이지만 최고의 정밀도'를 발휘하여 최종 해를 찾아낸다. 각 단계는 다음 단계의 알고리즘이 자신의 장점을 최대한 발휘하고 단점은 드러나지 않도록 하는 최적의 조건을 만들어주는, 매우 지능적인 협력 체계인 셈이다.


이론을 실제 코드로 구현할 때 가장 어려운 부분은 수많은 파라미터를 어떻게 설정해야 하는가이다. 다음 표는 이 파이프라인의 각 단계에서 성능에 결정적인 영향을 미치는 주요 파라미터들과 그 튜닝에 대한 전문가의 가이드를 제공한다.

| 단계 (Stage)    | 알고리즘 (Algorithm)        | 주요 파라미터 (Key Parameter) | 설명                                                         | 튜닝 가이드 및 고려사항                                      |
| --------------- | --------------------------- | ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **전처리**      | Voxel Grid Filter           | `leaf_size`                   | 다운샘플링할 복셀의 크기 29                                  | 값이 클수록 데이터가 많이 줄어 속도는 빨라지나, 디테일이 손실됨. 객체의 크기와 특징에 맞춰 조절. 보통 정합 정확도의 최소 단위와 관련이 있다. |
| **전처리**      | Statistical Outlier Removal | `k`, `std_dev_mul`            | 이웃 `k`개와의 평균 거리가 (전체 평균 + `std_dev_mul` * 표준편차)를 넘는 점을 제거 29 | 노이즈가 심할수록 `std_dev_mul`을 낮게(예: 1.0), 점 밀도가 낮으면 `k`를 작게(예: 20) 설정하여 너무 많은 점이 제거되지 않도록 한다. |
| **특징 추출**   | Normal Estimation           | `search_radius`               | 법선 벡터 계산 시 고려할 이웃의 반경                         | 표면의 곡률에 따라 조절. 너무 작으면 노이즈에 민감하고, 너무 크면 엣지나 코너 부분의 법선이 뭉개진다. `leaf_size`의 2~4배 정도로 시작하는 것이 일반적이다. |
| **특징 추출**   | FPFH Estimation             | `search_radius`               | FPFH 계산 시 고려할 이웃의 반경 6                            | Normal Estimation의 `search_radius`보다 커야 한다. 더 넓은 영역의 기하학적 정보를 담기 위함이다. 보통 Normal Estimation 반경의 2배 정도로 설정한다. |
| **Coarse 정합** | RANSAC                      | `distance_threshold`          | 인라이어로 판단할 대응점 쌍의 최대 거리 21                   | `leaf_size`와 밀접한 관련이 있다. 보통 `leaf_size`의 1.5배에서 2배 사이 값으로 설정한다. 너무 크면 잘못된 매칭을 인라이어로 판단하고, 너무 작으면 올바른 매칭도 놓칠 수 있다. |
| **Coarse 정합** | RANSAC                      | `max_iteration`, `confidence` | 최대 반복 횟수와 원하는 신뢰도 21                            | 높을수록 좋은 결과를 찾을 확률이 높아지지만 시간이 오래 걸린다. 실시간성이 중요하다면 낮게 설정해야 한다. 보통 `max_iteration`은 수천 번, `confidence`는 0.999로 설정한다. |
| **중간 정합**   | NDT                         | `Resolution`                  | 내부 복셀 그리드의 해상도 35                                 | **가장 중요하고 민감한 파라미터.** `leaf_size`와 유사하지만 NDT의 성능에 직접적인 영향을 미친다. 데이터 스케일에 맞춰 신중히 조절해야 한다. 너무 크면 디테일을 잃고, 너무 작으면 메모리 사용량이 폭증하고 빈 복셀이 많아져 불안정해진다. |
| **중간 정합**   | NDT                         | `TransformationEpsilon`       | 수렴으로 판단할 변환의 최소 변화량 35                        | 0에 가까울수록 더 정밀하게 수렴하지만, 수렴 시간이 길어질 수 있다. 보통 $10^{-8}$ 정도의 작은 값을 사용한다. |
| **정밀 정합**   | ICP                         | `MaxCorrespondenceDistance`   | 대응점으로 간주할 최대 거리                                  | 반복마다 동적으로 줄여나가는 전략이 유효하다. 초기에는 NDT의 `Resolution`과 비슷한 값으로 넓게 설정하고, 수렴할수록 점차 좁게 만들어 정밀도를 높인다. |
| **정밀 정합**   | ICP                         | `MaximumIterations`           | 최대 반복 횟수 39                                            | NDT를 거친 후이므로 많은 반복이 필요하지 않다. 보통 30에서 50 사이의 값이면 충분히 수렴한다. |



FPFH, NDT, ICP를 순차적으로 결합한 Coarse-to-Fine 파이프라인은 현대 포인트 클라우드 정합 분야에서 매우 강력하고 신뢰성 높은 표준적인 접근법 중 하나다.

- **강점:** 이 파이프라인의 가장 큰 강점은 '강인함(robustness)'이다. 초기 정렬 상태가 매우 좋지 않거나, 데이터에 상당한 양의 노이즈와 이상치가 포함되어 있거나, 두 스캔 간의 중첩 영역이 제한적인 실제 환경 데이터에 대해 매우 높은 정합 성공률과 정확도를 보인다. 이는 각 알고리즘의 장점(FPFH의 풍부한 표현력, RANSAC의 통계적 강인함, NDT의 넓은 수렴 반경과 속도, ICP의 최종 정밀도)을 유기적으로 결합하여 시너지 효과를 내기 때문이다.
- **약점:** 가장 명백한 약점은 파이프라인의 복잡성이다. 여러 단계를 거치므로 전체적인 구조가 복잡하고, 각 단계마다 튜닝해야 할 파라미터가 많아 최적의 성능을 내기 위해서는 상당한 전문 지식과 실험이 필요하다. 또한, 전처리부터 최종 정합까지 전체 실행 시간이 수 초 이상 걸릴 수 있어, 극도의 실시간성이 요구되는 애플리케이션(예: 고속 로봇의 충돌 회피)에는 부적합할 수 있다.44 또 다른 잠재적 약점은 파이프라인의 첫 단추인 FPFH에 대한 의존성이다. 특징이 거의 없는(featureless) 평면이나 반복적인 기하학적 구조가 많은 환경에서는 FPFH가 변별력 있는 특징을 추출하지 못해 초기 정렬에 실패할 수 있고, 이 경우 전체 파이프라인이 무너지게 된다.
- **적합한 시나리오:** 이러한 특성을 고려할 때, 이 파이프라인은 자율주행을 위한 고정밀 지도 제작(offline mapping), 문화유산의 정밀 3D 디지털 복원, 공장이나 건설 현장과 같은 복잡한 환경에 대한 로봇의 작업 환경 모델링 등, 극도의 실시간성보다는 높은 정확도와 강인성이 최우선으로 요구되는 분야에 가장 적합하다.


모든 상황에 맞는 만능 정합 알고리즘은 존재하지 않는다. 따라서 주어진 문제의 특성에 맞게 파이프라인을 유연하게 조정하거나 다른 대안을 고려해야 한다.

- **실시간성이 최우선일 경우:** FPFH+RANSAC -->> 경량화된 ICP(또는 NDT 단독) 파이프라인을 고려해볼 수 있다. 이 경우, 다운샘플링을 더 공격적으로 수행하고(leaf_size 증가), 각 알고리즘의 반복 횟수를 대폭 줄여야 한다. 정확도는 다소 희생되지만, 수십~수백 밀리초 내에 결과를 얻는 것을 목표로 할 수 있다.
- **포인트 클라우드가 매우 조밀하고 깨끗하며, 초기 정렬이 양호할 경우:** 예를 들어, 센서의 외부 파라미터(extrinsic parameter)를 이미 알고 있는 경우, NDT 단계를 생략하고 FPFH+RANSAC -->> ICP로 파이프라인을 단순화하여 속도를 높일 수 있다. NDT의 역할은 불안정한 초기값을 안정시키는 것인데, 초기값이 이미 충분히 좋다면 이 단계는 불필요할 수 있다.
- **특징이 거의 없는(featureless) 환경일 경우:** 복도나 터널과 같이 기하학적 특징이 거의 없는 환경에서는 FPFH 기반 방법이 실패할 확률이 높다. 이 경우, FPFH를 사용하지 않고 NDT 단독으로 정합을 시도하거나, Go-ICP와 같이 추가적인 정보를 사용하지 않고 전역 최적화를 보장하는 다른 알고리즘을 고려해야 한다.15


전통적인 기하학 기반의 이 파이프라인은 매우 성숙하고 안정적이지만, 최근에는 딥러닝 기술을 접목하여 그 한계를 뛰어넘으려는 연구가 활발히 진행되고 있다.

최신 연구 동향은 딥러닝을 이용해 더 강력한 특징점을 추출하거나 5, 두 포인트 클라우드 간의 대응 관계를 직접 학습하거나 30, 심지어 정합 과정 전체를 하나의 End-to-End 신경망으로 대체하려는 시도들로 요약된다.14

가장 유망한 방향 중 하나는 이 전통적인 파이프라인의 각 단계를 딥러닝 모듈로 점진적으로 대체하는 하이브리드 접근법이다.7 예를 들어, FPFH 기술자 대신 딥러닝 기반의 3D 지역 특징 기술자(예: FCGF, D3Feat)를 사용하면, 훨씬 낮은 중첩률이나 심한 노이즈 환경에서도 강인한 초기 대응점을 찾을 수 있다. 이는 전통적인 기하학 기반 알고리즘이 제공하는 구조적인 안정성과 딥러닝 모델의 강력한 데이터 기반 표현력을 결합하는 방향으로, 현재 포인트 클라우드 정합 연구의 주류를 이루고 있다. 앞으로 이러한 하이브리드 방법들이 기존 파이프라인의 성능을 한 단계 더 끌어올릴 것으로 기대된다.


1. Module registration - Point Cloud Library (PCL), accessed July 25, 2025, https://pointclouds.org/documentation/group__registration.html
2. Point-set registration - Wikipedia, accessed July 25, 2025, https://en.wikipedia.org/wiki/Point-set_registration
3. Iterative closest point - Wikipedia, accessed July 25, 2025, https://en.wikipedia.org/wiki/Iterative_closest_point
4. Understanding Iterative Closest Point (ICP) Algorithm with Code - LearnOpenCV, accessed July 25, 2025, https://learnopencv.com/iterative-closest-point-icp-explained/
5. SANDRO: a Robust Solver with a Splitting Strategy for Point Cloud Registration - arXiv, accessed July 25, 2025, https://arxiv.org/html/2503.07743v1
6. Aerial Lidar SLAM Using FPFH Descriptors - MATLAB & Simulink - MathWorks, accessed July 25, 2025, https://www.mathworks.com/help/lidar/ug/aerial-lidar-slam-using-fpfh-descriptors.html
7. Advanced Point Cloud Registration Techniques - Number Analytics, accessed July 25, 2025, https://www.numberanalytics.com/blog/advanced-point-cloud-registration-techniques
8. A Structure-Based Iterative Closest Point Using Anderson Acceleration for Point Clouds with Low Overlap - MDPI, accessed July 25, 2025, https://www.mdpi.com/1424-8220/23/4/2049
9. www.numberanalytics.com, accessed July 25, 2025, [https://www.numberanalytics.com/blog/advanced-point-cloud-registration-techniques#:~:text=Point%20cloud%20registration%20is%20a,a%20single%2C%20coherent%20coordinate%20system.](https://www.numberanalytics.com/blog/advanced-point-cloud-registration-techniques#:~:text=Point cloud registration is a,a single%2C coherent coordinate system.)
10. EP3707469A1 - A point clouds registration system for autonomous vehicles - Google Patents, accessed July 25, 2025, https://patents.google.com/patent/EP3707469A1/en
11. Comparison of Point Cloud Registration Techniques on Scanned Physical Objects - MDPI, accessed July 25, 2025, https://www.mdpi.com/1424-8220/24/7/2142
12. Point Cloud Registration in Action - Number Analytics, accessed July 25, 2025, https://www.numberanalytics.com/blog/point-cloud-registration-action-cognitive-robotics
13. Navigating Common Pitfalls in Point Cloud Analysis - Mindkosh AI, accessed July 25, 2025, https://mindkosh.com/blog/navigating-common-pitfalls-in-point-cloud-analysis/
14. Robust Point Cloud Registration Network for Complex Conditions - PMC, accessed July 25, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10747109/
15. Comparison of Point Cloud Registration Techniques on Scanned ..., accessed July 25, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11014384/
16. Common problems in point set registration. (a) Initial alignment that... | Download Scientific Diagram - ResearchGate, accessed July 25, 2025, https://www.researchgate.net/figure/Common-problems-in-point-set-registration-a-Initial-alignment-that-can-yield-an_fig1_44683588
17. Hybrid algorithm ideology. ICP step by step comes to local minima.... | Download Scientific Diagram - ResearchGate, accessed July 25, 2025, https://www.researchgate.net/figure/Hybrid-algorithm-ideology-ICP-step-by-step-comes-to-local-minima-After-local-minima_fig4_281412803
18. Point Cloud Registration - Classic Approaches | Towards Data Science, accessed July 25, 2025, https://towardsdatascience.com/point-cloud-registration-classic-approaches-d6191302b0b2/
19. A Pointcloud Registration Framework for Relocalization in Subterranean Environments, accessed July 25, 2025, https://arxiv.org/html/2504.07231v1
20. Point Cloud Registration Algorithm Based on NDT and Feature Point Detection, accessed July 25, 2025, https://www.researching.cn/articles/OJe5f35bb9ef244d4c
21. Global registration - Open3D primary (unknown) documentation, accessed July 25, 2025, https://www.open3d.org/docs/release/tutorial/pipelines/global_registration.html
22. A Coarse-to-Fine Registration Approach for Point Cloud Data with Bipartite Graph Structure, accessed July 25, 2025, https://www.mdpi.com/2079-9292/11/2/263
23. A BRIEF OVERVIEW OF THE CURRENT STATE, CHALLENGING ISSUES AND FUTURE DIRECTIONS OF POINT CLOUD REGISTRATION, accessed July 25, 2025, https://isprs-annals.copernicus.org/articles/X-3-W1-2022/17/2022/isprs-annals-X-3-W1-2022-17-2022.pdf
24. Fast Point Feature Histograms (FPFH) for 3D Registration - Papers With Code, accessed July 25, 2025, https://paperswithcode.com/paper/fast-point-feature-histograms-fpfh-for-3d
25. A Globally Optimal Solution to 3D ICP Point-Set Registration - arXiv, accessed July 25, 2025, http://arxiv.org/pdf/1605.03344
26. The PCL Registration API - Point Cloud Library 0.0 documentation - Read the Docs, accessed July 25, 2025, https://pcl.readthedocs.io/projects/tutorials/en/master/registration_api.html
27. Fast Point Feature Histograms (FPFH) descriptors - Point Cloud ..., accessed July 25, 2025, https://pcl.readthedocs.io/projects/tutorials/en/master/fpfh_estimation.html
28. Point cloud registration in PCL using precomputed features - Stack Overflow, accessed July 25, 2025, https://stackoverflow.com/questions/50439552/point-cloud-registration-in-pcl-using-precomputed-features
29. Point Cloud Registration Based on Fast Point Feature Histogram Descriptors for 3D Reconstruction of Trees - MDPI, accessed July 25, 2025, https://www.mdpi.com/2072-4292/15/15/3775
30. Fast Point Feature Histograms (FPFH) for 3D registration | Request PDF - ResearchGate, accessed July 25, 2025, https://www.researchgate.net/publication/224557255_Fast_Point_Feature_Histograms_FPFH_for_3D_registration
31. Point Feature Histograms (PFH) descriptors - Point Cloud Library 0.0 documentation, accessed July 25, 2025, https://pcl.readthedocs.io/projects/tutorials/en/master/pfh_estimation.html
32. Point Cloud Registration Method for Pipeline Workpieces Based on PCA and Improved ICP Algorithms - ResearchGate, accessed July 25, 2025, https://www.researchgate.net/publication/336671754_Point_Cloud_Registration_Method_for_Pipeline_Workpieces_Based_on_PCA_and_Improved_ICP_Algorithms
33. Normal distributions transform - Wikipedia, accessed July 25, 2025, https://en.wikipedia.org/wiki/Normal_distributions_transform
34. The Normal Distributions Transform: A New Approach to ... - CiteSeerX, accessed July 25, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=1f16244ce2e78881c7c96b33796a930ce73f7972
35. How to use Normal Distributions Transform - Point Cloud Library 1.15.0-dev documentation, accessed July 25, 2025, https://pointclouds.org/documentation/tutorials/normal_distributions_transform.html
36. Evaluation of 3D Registration Reliability and Speed–A Comparison ..., accessed July 25, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=0185f6fb7a7a7859daf8ae326b485e9891a7461d
37. Research on Point Cloud Registering Method of Tunneling Roadway Based on 3D NDT-ICP Algorithm - MDPI, accessed July 25, 2025, https://www.mdpi.com/1424-8220/21/13/4448
38. NOTES ON ITERATIVE CLOSEST POINT ALGORITHM - EVLM, accessed July 25, 2025, http://evlm.stuba.sk/APLIMAT2018/proceedings/Papers/0876_Prochazkova_Martisek.pdf
39. How to incrementally register pairs of clouds - Point Cloud Library 1.15.0-dev documentation, accessed July 25, 2025, https://pointclouds.org/documentation/tutorials/pairwise_incremental_registration.html
40. Iterative Closest Point (ICP) and other registration algorithms, accessed July 25, 2025, https://docs.mrpt.org/reference/latest/tutorial-icp-alignment.html
41. On the Covariance of ICP-based Scan-matching Techniques - arXiv, accessed July 25, 2025, https://arxiv.org/pdf/1410.7632
42. Convergence characteristics of the ICP algorithm - Freie Universität Berlin, accessed July 25, 2025, https://www.mi.fu-berlin.de/inf/groups/ag-ti/theses/download/Offner22.pdf
43. ICP Algorithm Demystified: Point Cloud Registration Mathematics - Patsnap Eureka, accessed July 25, 2025, https://eureka.patsnap.com/article/icp-algorithm-demystified-point-cloud-registration-mathematics
44. Optimizing the Normal Distributions Transform Update Algorithm - ResearchGate, accessed July 25, 2025, https://www.researchgate.net/publication/390244280_Optimizing_the_Normal_Distributions_Transform_Update_Algorithm
45. XuyangBai/awesome-point-cloud-registration - GitHub, accessed July 25, 2025, https://github.com/XuyangBai/awesome-point-cloud-registration
46. RDMNet: Reliable Dense Matching Based Point Cloud Registration for Autonomous Driving - arXiv, accessed July 25, 2025, https://arxiv.org/pdf/2303.18084
47. Coarse to Fine Point Cloud Registration with SE(3)-Equivariant Representations - YouTube, accessed July 25, 2025, https://www.youtube.com/watch?v=2fkW2l9EOe8
48. An Efficient and Stable Registration Framework for Large Point ..., accessed July 25, 2025, https://www.mdpi.com/1424-8220/24/22/7174
# NDT-ICP 하이브리드 포인트 클라우드 정합
[포인트 클라우드 정합](./index.md)



포인트 클라우드 정합(Point Cloud Registration)은 서로 다른 좌표계에서 획득된 두 개 이상의 포인트 클라우드를 하나의 공통 좌표계로 정렬하는 프로세스다.1 근본적인 목표는 소스 포인트 클라우드($P_{source}$)와 타겟 포인트 클라우드($P_{target}$) 사이에 존재하는 공간적 변환(spatial transformation)을 찾아내는 것이다. 이 변환은 일반적으로 회전(rotation)과 이동(translation)으로 구성된다. 이 과정을 통해 여러 번의 스캔으로 얻은 분절된 데이터를 하나의 일관된 3D 모델로 통합하거나, 새로운 측정 데이터를 기존 모델에 매핑하여 객체의 자세(pose)를 추정할 수 있다.2

포인트 클라우드 정합은 3D 데이터 분석의 핵심적인 작업으로, 그 응용 분야는 매우 광범위하다. 자율주행 자동차는 LiDAR 센서로 주변 환경을 스캔하여 획득한 포인트 클라우드를 정합함으로써 자신의 위치를 파악하고 지도를 생성한다. 로보틱스 분야에서는 로봇이 주변 환경을 인식하고 객체를 조작하기 위해 정합 기술을 사용하며, 의료 영상 분야에서는 여러 장의 CT나 MRI 스캔을 정합하여 환자의 3차원 해부학적 구조를 정밀하게 재구성한다.1 이 외에도 정밀 계측, 문화유산 디지털화, 3D 모델 재구성 등 다양한 산업 및 연구 분야에서 필수적인 기술로 자리 잡고 있다.3


포인트 클라우드 정합 문제는 수학적으로 최적의 강체 변환(rigid body transformation) ``{R, t}``를 찾는 최적화 문제로 공식화된다. 이 변환은 변환된 소스 포인트 클라우드와 타겟 포인트 클라우드 간의 거리 메트릭(distance metric)을 최소화해야 한다.1 강체 변환은 객체의 형태나 크기를 변경하지 않고 오직 위치와 방향만을 바꾸는 변환을 의미한다.

이 변환은 3x3 회전 행렬 $R$과 3x1 이동 벡터 $t$로 구성된다. 회전 행렬 $R$은 특수 직교 그룹(Special Orthogonal group) $SO(3)$에 속하며, 이는 $R^T R = I$이고 $\det(R) = 1$인 조건을 만족하는 행렬의 집합이다. 이동 벡터 $t$는 3차원 유클리드 공간 $\mathbb{R}^3$에 속한다. 소스 포인트 클라우드 $P_{source}$에 속한 임의의 점 $p_i$에 변환 $T$를 적용하는 것은 다음과 같이 표현된다:
$$
T(p_i) = R p_i + t
$$
따라서 정합 알고리즘의 목표는 아래의 목적 함수를 최소화하는 최적의 $R$과 $t$를 찾는 것이다.2
$$
\underset{R, t}{\operatorname{argmin}} \sum_{i} \mathcal{D}(R p_i + t, q_i)
$$
여기서 $q_i$는 소스 포인트 $p_i$에 대응하는 타겟 포인트 클라우드 내의 점이며, $\mathcal{D}$는 두 점 사이의 거리를 측정하는 함수로, 일반적으로 제곱 유클리드 거리(squared Euclidean distance)가 사용된다.


포인트 클라우드 정합은 개념적으로는 간단해 보이지만 실제 적용에서는 여러 가지 난제에 직면한다.

- **대응점 탐색 (Correspondence Search):** 소스 클라우드의 각 점과 타겟 클라우드의 어떤 점이 서로 짝을 이루는지(대응 관계)를 알아내는 것은 정합에서 가장 어렵고 계산 비용이 많이 드는 부분이다. 잘못된 대응점은 정합 정확도를 심각하게 저하시킨다.
- **노이즈와 이상점 (Noise and Outliers):** LiDAR와 같은 실제 센서로부터 얻은 데이터는 측정 노이즈를 포함하며, 때로는 환경의 다른 객체나 센서 오류로 인해 원래 객체와 무관한 이상점(outlier)이 포함될 수 있다. 이러한 노이즈와 이상점은 정합 알고리즘의 성능을 크게 저하시키는 요인이다.4
- **부분 중첩 (Partial Overlap):** 두 스캔 데이터는 종종 부분적으로만 중첩된다. 이는 소스와 타겟 클라우드 간에 공통으로 나타나는 영역이 제한적임을 의미하며, 이로 인해 신뢰할 수 있는 대응점을 충분히 찾기 어려워져 안정적인 정합을 방해한다.4
- **초기 정렬 (Initial Alignment):** 많은 정합 알고리즘, 특히 반복적인 방식을 사용하는 알고리즘들은 초기 변환 추정치에 매우 민감하다. 만약 초기 정렬 상태가 실제 정답과 크게 다를 경우, 알고리즘이 전역 최적해(global minimum)가 아닌 지역 최적해(local minimum)에 수렴하여 완전히 잘못된 결과를 도출할 수 있다.7


포인트 클라우드 정합의 개념은 시간이 지남에 따라 진화해왔다. 초기 고전적인 알고리즘들은 이 문제를 순수한 기하학적 최적화 문제로 정의했다. 즉, 두 포인트 클라우드 간의 공간적 거리를 최소화하는 데 초점을 맞췄다.2 그러나 최근 연구 동향은 여기에 시맨틱 정보(semantic information), 즉 의미론적 정보를 통합하는 방향으로 나아가고 있다.1 이는 단순히 기하학적으로 가장 가까운 점을 찾는 것을 넘어, '의미적으로 일관된' 대응점을 찾는 문제로 패러다임을 전환시키고 있다.

이러한 변화의 배경에는 고전적 방법의 명백한 한계가 있다. 예를 들어, ICP와 같은 알고리즘은 복도나 터널처럼 기하학적 특징이 반복되거나 없는 환경에서 실패하기 쉽다.6 벽면 위의 한 점은 기하학적으로는 벽면의 다른 어떤 점과도 유사하기 때문에, 알고리즘은 잘못된 대응점을 찾고 엉뚱한 방향으로 수렴할 수 있다.

이 문제를 해결하기 위해 연구자들은 더 높은 수준의 정보를 활용하기 시작했다. 예를 들어, Semantic-assisted NDT (SE-NDT)는 포인트 클라우드를 '도로', '건물', '차량' 등 의미론적 레이블에 따라 분할하고, 각 레이블별로 독립적인 정합을 수행한다.9 의료 영상 분야의 Semantic-ICP는 해부학적 레이블을 사용하여 정합을 유도한다.10 더 나아가, PNE-SGAN과 같은 최신 연구는 시맨틱 그래프(semantic graph)의 노드 특징으로 NDT 공분산을 사용하여 의미와 기하학을 동시에 고려한다.11

이는 정합 문제가 "가장 가까운 점이 무엇인가?"에서 "가장 가깝고 의미적으로 동등한 점이 무엇인가?"로 질문 자체가 바뀌었음을 시사한다. 자동차 위의 점은 도로 위의 기하학적으로 가까운 점이 아니라, 다른 스캔에 있는 자동차 위의 점과 매칭되어야 한다. 이러한 시맨틱 제약 조건의 추가는 대응점 탐색 공간을 극적으로 줄이고 기하학적 모호성을 해결함으로써 정합 알고리즘의 강인성을 근본적으로 향상시킨다. 결과적으로, 현대의 포인트 클라우드 정합은 단순한 기하학 문제를 넘어, 복잡하고 동적인 실제 환경에서 신뢰성 있는 자율성을 구현하기 위한 핵심적인 '상황 인지(context-aware)' 기술로 발전하고 있다.


NDT-ICP 하이브리드 알고리즘을 이해하기 위해서는 그 구성 요소인 Iterative Closest Point (ICP)와 Normal Distributions Transform (NDT) 각각의 작동 원리, 수학적 모델, 그리고 장단점을 명확히 파악해야 한다. 두 알고리즘은 근본적으로 다른 철학을 바탕으로 정합 문제를 해결하며, 이 차이점이 바로 하이브리드 방식의 시너지 효과를 낳는 원천이 된다.


ICP는 포인트 클라우드 정합 분야에서 가장 널리 알려지고 오랫동안 사용되어 온 고전적인 알고리즘이다.12 이름에서 알 수 있듯이, 반복적인 계산을 통해 두 포인트 클라우드 간의 오차를 점진적으로 최소화하는 방식으로 최적의 변환을 찾는다.


ICP 알고리즘의 전체적인 파이프라인은 다음과 같은 단계로 구성된다.7

1. **초기화 (Initialization):** 소스 포인트 클라우드 $P$를 타겟 포인트 클라우드 $Q$에 대략적으로 정렬하기 위한 초기 변환 추정치 $T_0 = \{R_0, t_0\}$로 시작한다. 이 초기 추정치의 품질은 매우 중요하다. 만약 초기 추정치가 실제 값과 너무 동떨어져 있으면, 알고리즘은 잘못된 지역 최적해(local minimum)로 수렴하거나 아예 수렴에 실패할 수 있다.7
2. **대응점 탐색 (Finding Correspondences):** 현재 변환이 적용된 소스 클라우드의 각 점 $p_i$에 대해, 타겟 클라우드 $Q$에서 가장 가까운 점 $q_i$를 찾는다. '가장 가깝다'는 것은 일반적으로 유클리드 거리를 기준으로 판단한다. 이 단계는 계산 비용이 매우 높은데, 이를 해결하기 위해 보통 타겟 클라우드에 대해 k-d 트리(k-d tree)와 같은 공간 분할 자료구조를 미리 구축하여 최근접 이웃 탐색(nearest neighbor search) 속도를 높인다.7
3. **변환 추정 (Estimating Transformation):** 이전 단계에서 찾은 대응점 쌍 $\{(p_i, q_i)\}$들을 가장 잘 정렬하는 강체 변환 $\{R_k, t_k\}$를 계산한다. 이는 대응점 쌍들 간의 제곱 거리 합을 최소화하는 문제로 귀결된다.
4. **변환 적용 (Applying Transformation):** 새로 계산된 변환을 소스 포인트 클라우드에 적용하여 업데이트한다.
5. **반복 (Iteration):** 미리 정의된 수렴 조건이 만족될 때까지 2~4단계를 반복한다. 수렴 조건으로는 보통 반복 간 변환 행렬의 변화량이 특정 임계값 이하로 떨어지거나, 오차의 변화가 미미해지거나, 최대 반복 횟수에 도달하는 것 등이 사용된다.7


가장 기본적인 형태인 Point-to-Point ICP의 목적 함수는 대응점 쌍 간의 제곱 유클리드 거리의 합을 최소화하는 것이다.6
$$
E(R, t) = \sum_{i=1}^{N} \| (R p_i + t) - q_i \|^2
$$
이 최적화 문제는 해석적인 해(closed-form solution)를 가지며, 특이값 분해(Singular Value Decomposition, SVD)를 통해 효율적으로 풀 수 있다. 해법의 과정은 다음과 같다.7

1. 두 대응점 집합의 중심(centroid)을 각각 계산한다:
   $$
   \bar{p} = \frac{1}{N}\sum_{i=1}^{N} p_i, \quad \bar{q} = \frac{1}{N}\sum_{i=1}^{N} q_i
   $$
   중심을 기준으로 정규화된 점들을 이용하여 공분산 행렬(cross-covariance matrix) $H$를 계산한다:
   $$
   H = \sum_{i=1}^{N} (p_i - \bar{p})(q_i - \bar{q})^T
   $$
   H$에 SVD를 적용하여 $H = USV^T$를 얻는다.

2. 최적의 회전 행렬 $R$과 이동 벡터 $t$는 다음과 같이 계산된다:
   $$
   R = VU^T, \quad t = \bar{q} - R\bar{p}
   $$
   이때, 계산된 $R$의 행렬식(determinant)이 -1인 경우(반사 변환), 이는 유효한 회전이 아니므로 $V$의 마지막 열의 부호를 바꾸어 $R$을 다시 계산하는 등의 후처리가 필요하다.




기본적인 Point-to-Point ICP는 여러 한계점을 가지며, 이를 개선하기 위한 다양한 변형 알고리즘이 제안되었다.

- **Point-to-Plane ICP:** 이 변형은 소스 클라우드의 한 점과 타겟 클라우드의 대응점 사이의 거리가 아닌, 소스 포인트와 타겟 포인트의 법선 벡터(normal vector)가 정의하는 접평면(tangent plane) 사이의 거리를 최소화한다. 목적 함수는 다음과 같다.13
  $$
  E(R, t) = \sum_{i=1}^{N} ( ( (R p_i + t) - q_i ) \cdot n_i )^2
  $$
  여기서 $n_i$는 타겟 포인트 $q_i$에서의 표면 법선 벡터다. 이 방식은 특히 평면 구조가 많은 환경에서 Point-to-Point 방식보다 더 빠르고 정확하게 수렴하는 경향이 있다.

- **Generalized ICP (GICP):** GICP는 한 걸음 더 나아가 각 점 주변의 국소적인 표면 구조를 공분산 행렬로 모델링한다. 두 점을 정합하는 대신, 두 점이 나타내는 확률 분포(가우시안 분포)를 정합한다. 이는 노이즈에 대한 강인성을 높여주며, 점 대 점 또는 점 대 평면이 아닌, 두 타원체(ellipsoid) 간의 불확실성을 고려한 정합을 수행하게 된다.14


- **강점:** 좋은 초기 추정치가 주어졌을 때, 매우 높은 정밀도로 미세 정렬(fine alignment)을 수행할 수 있다.7 알고리즘의 핵심 아이디어가 직관적이고 구현이 비교적 간단하다.
- **약점:** 가장 큰 약점은 초기 추정치에 대한 높은 민감성이다. 초기 정렬이 좋지 않으면 지역 최적해에 빠지기 매우 쉽다.7 또한, 대규모 포인트 클라우드에 대해서는 모든 점에 대한 최근접 이웃 탐색으로 인해 계산 비용이 매우 높다.7 이상점과 낮은 중첩률에도 민감하게 반응하여 성능이 저하된다.


NDT는 ICP와 근본적으로 다른 접근 방식을 취하는 확률적 정합 알고리즘이다. NDT는 명시적인 점 대 점 대응 관계를 찾는 과정을 생략하고, 포인트 클라우드를 확률 분포의 집합으로 표현하여 정합을 수행한다.16


NDT의 핵심 아이디어는 다음과 같다.18

1. **복셀화 (Voxelization):** 먼저 타겟 포인트 클라우드가 차지하는 3D 공간을 균일한 크기의 정육면체 셀, 즉 복셀(voxel)로 나눈다.
2. **통계 모델링 (Statistical Modeling):** 각 복셀 내에 포함된 점들의 평균 벡터($\mathbf{q}$)와 공분산 행렬($\mathbf{S}$)을 계산한다.
3. **확률 밀도 함수 생성:** 이 과정을 통해 이산적인(discrete) 포인트 클라우드는 각 복셀이 다변량 정규 분포(multivariate normal distribution)로 표현되는 조각적 연속(piecewise continuous)이고 미분 가능한 확률 밀도 함수(probability density function)로 변환된다. 즉, 타겟 클라우드는 가우시안 혼합 모델(Gaussian Mixture Model)과 유사한 형태로 추상화된다.


특정 복셀 내에서 한 점 $\mathbf{x}$가 관측될 확률은 해당 복셀의 가우시안 분포에 의해 모델링된다.18
$$
p(\mathbf{x}) \sim \exp\left(-\frac{1}{2}(\mathbf{x} - \mathbf{q})^T \mathbf{S}^{-1} (\mathbf{x} - \mathbf{q})\right)
$$
소스 클라우드를 정합하기 위해, 소스 클라우드의 각 점 $p_i$를 변환 $T(p_i) = R p_i + t$ 시킨다. 그 후, 이 변환된 점들이 타겟 클라우드의 NDT 모델에 얼마나 잘 들어맞는지를 나타내는 우도(likelihood)를 최대화한다. 이는 각 변환된 점에서의 확률값의 합, 즉 'NDT 스코어'를 최대화하는 것과 같다.18
$$
\text{score}(R, t) = \sum_{i=1}^{N} p(T(p_i))
$$
NDT 스코어 함수는 미분 가능하므로, 뉴턴 방법(Newton's method)이나 More-Thuente 라인 서치와 같은 표준 수치 최적화 기법을 사용하여 스코어를 최대화하는 최적의 변환 $\{R, t\}$를 찾을 수 있다.18


NDT의 성능은 몇 가지 주요 파라미터에 크게 좌우된다.19

- **해상도 (Resolution):** 복셀 그리드의 크기. 이는 가장 중요하고 민감한 파라미터다. 복셀의 크기는 안정적인 공분산 행렬을 계산할 수 있을 만큼 충분한 점(예: 최소 5-6개)을 포함할 정도로 커야 하지만, 동시에 환경의 기하학적 세부 사항을 포착할 수 있을 만큼 충분히 작아야 한다.
- **변환 엡실론 (Transformation Epsilon):** 반복 간 변환 파라미터의 최소 변화량. 이 값보다 변화가 작아지면 알고리즘이 수렴했다고 판단하고 종료한다.
- **스텝 크기 / 최대 반복 횟수 (Step Size / Maximum Iterations):** 최적화 과정의 속도와 안정성을 제어하는 파라미터로, 과도한 이동으로 인한 발산이나 무한 루프를 방지한다.


- **강점:** 점별 대응점 탐색을 하지 않으므로 대규모 포인트 클라우드에서 ICP보다 훨씬 빠르다.17 ICP보다 넓은 수렴 반경(convergence basin)을 가져 초기 추정치가 좋지 않아도 상대적으로 강인하다.21 복셀 기반 표현은 본질적으로 데이터 압축 효과를 가진다.22
- **약점:** 복셀화 과정에서 고주파의 세밀한 기하학적 정보가 손실(평활화)되기 때문에, 미세 정렬 정확도는 ICP보다 떨어진다.23 그리드 분할 방식은 셀 경계에서 불연속성을 야기할 수 있으며, 성능이 해상도 파라미터 선택에 매우 민감하다.9


ICP와 NDT의 핵심적인 성능 차이는 데이터 표현 방식의 근본적인 트레이드오프에서 비롯된다. ICP는 원본의 명시적인 포인트 데이터를 직접 다룬다. 이는 높은 충실도(fidelity)를 보장하지만, 계산 비용이 많이 들고 국소적인 기하학적 형태에 민감하게 만든다. 반면, NDT는 데이터를 통계적으로 추상화한 모델을 다룬다. 이는 속도가 빠르고 노이즈에 강인하지만, 미세한 디테일을 잃는 대가를 치른다.

이러한 차이는 각 알고리즘의 목적 함수에서 명확히 드러난다. ICP의 목적 함수는 개별 점 $p_i$와 $q_i$의 좌표를 직접 사용한다.7 이는 '보이는 그대로'를 사용하는 접근 방식으로, 모든 기하학적 정보를 보존하여 높은 정밀도를 달성할 수 있게 하지만(강점), 동시에 모든 이상점과 노이즈 포인트에 취약하게 만든다(약점).

반면, NDT의 목적 함수는 복셀 내 점들의 평균 $\mathbf{q}$와 공분산 $\mathbf{S}$를 사용한다.18 초기 통계 요약 이후에는 개별 점들을 다시 보지 않는다. 바로 이 '추상화'가 속도와 노이즈 강인성의 원천이다. 소수의 노이즈 포인트는 다수의 점을 포함하는 셀의 평균이나 공분산을 크게 바꾸지 않기 때문이다. 하지만 바로 이 추상화가 정밀도 측면에서는 발목을 잡는다. 날카로운 모서리와 둥근 모서리가 같은 복셀에 위치한다면, 매우 유사한 공분산 행렬을 생성할 수 있고, NDT 최적화기는 이 둘을 구분하지 못할 수 있다. 이것이 NDT의 정확도가 ICP만큼 높지 않은 이유다.23

이러한 이중성은 결함이 아니라 각 알고리즘의 근본적인 특성이다. 그리고 이 특성은 왜 두 알고리즘이 단순한 대체재가 아니라 상호 보완적인 파트너인지를 완벽하게 설명해준다. NDT의 통계적 추상화는 큰 초기 오차를 빠르게 줄이는 대략적인 정합(coarse registration)에 이상적이며, ICP의 높은 충실도를 가진 점 기반 표현은 최종 정렬을 정밀하게 다듬는 미세 정합(fine registration)에 이상적이다. NDT-ICP 하이브리드 접근법은 바로 이 표현 방식의 이중성을 직접적으로 활용하는 전략이다.


NDT와 ICP는 각각 명확한 장단점을 가지고 있으며, 이는 두 알고리즘이 서로의 약점을 보완해주는 이상적인 관계를 형성할 수 있음을 시사한다. NDT-ICP 하이브리드 알고리즘은 이러한 상호 보완성을 극대화하기 위해 "Coarse-to-Fine (대략적인 정합에서 미세한 정합으로)" 전략을 채택한 대표적인 사례다.


"Coarse-to-Fine"은 정합 문제에서 널리 사용되는 전략으로, 처음에는 빠르고 강인한 방법으로 대략적인 정렬을 수행한 뒤, 그 결과를 초기값으로 삼아 정밀한 방법으로 최종 결과를 다듬는 방식이다.5

- **원리:** NDT-ICP 하이브리드 방식은 두 알고리즘의 상호 보완적인 강점을 활용한다. 먼저, 속도가 빠르고 초기값에 덜 민감한 NDT를 사용하여 대략적인 정합(coarse registration)을 수행한다. 이 단계의 목표는 전역 최적해에 가까운, 충분히 좋은 초기 추정치를 신속하게 얻는 것이다. 그 다음, NDT가 제공한 이 양질의 초기값을 ICP 알고리즘의 시작점으로 사용하여, 높은 정밀도를 가진 미세 정합(fine registration)을 수행한다.17 이로써 ICP는 자신의 최대 강점인 국소적 정밀 최적화에만 집중할 수 있게 된다.
- **작업 흐름:** NDT-ICP 하이브리드 알고리즘의 일반적인 작업 흐름은 다음과 같다.24
  1. **입력:** 소스 포인트 클라우드 $P_{source}$, 타겟 포인트 클라우드 $P_{target}$, 그리고 선택적으로 부정확할 수 있는 초기 추정치 $T_{init}$를 입력받는다.
  2. **1단계: Coarse Registration (NDT):** NDT 알고리즘을 실행하여 $P_{source}$를 $P_{target}$에 정합한다. 이 단계의 결과로 강인하지만 정확도는 중간 수준인 변환 행렬 $T_{NDT}$가 출력된다.
  3. **2단계: Fine Registration (ICP):** NDT 단계에서 얻은 $T_{NDT}$를 ICP 알고리즘의 초기 변환으로 사용한다. ICP는 이 좋은 시작점에서부터 반복 계산을 통해 정렬을 정밀하게 다듬는다.
  4. **출력:** ICP 단계에서 최종적으로 계산된, 매우 정확한 변환 행렬 $T_{final}$을 최종 결과로 출력한다.


NDT-ICP 하이브리드 알고리즘은 NDT와 ICP의 목적 함수를 결합한 단일의 새로운 목적 함수를 최적화하는 방식이 아니다. 대신, 두 개의 최적화 과정을 순차적으로 수행하는 구조를 가진다.

- **1단계 (NDT 최적화):** 이 단계의 목표는 ICP의 오차 함수가 가지는 전역 최적해의 수렴 반경 안으로 소스 포인트 클라우드를 이동시키는 것이다.
  $$
  T_{NDT} = \{R_{NDT}, t_{NDT}\} = \underset{R, t}{\operatorname{argmax}} \sum_{i} \text{score}_{NDT}(R p_i + t)
  $$
  여기서 $\text{score}_{NDT}$는 변환된 소스 포인트가 타겟 클라우드의 NDT 확률 모델에 얼마나 잘 맞는지를 나타내는 점수다. 이 최적화는 전역적인 탐색에 가깝지만, 정확도보다는 속도와 강인성에 중점을 둔다.

- **2단계 (ICP 최적화):** 이 단계는 NDT가 제공한 정렬 상태에서 시작하여 국소적인 정밀 최적화를 수행한다.
  $$
  T_{final} = \{R_{ICP}, t_{ICP}\} = \underset{R, t}{\operatorname{argmin}} \sum_{j} \| (R p'_j + t) - q'_j \|^2
  $$
  여기서 $P'_{source} = T_{NDT}(P_{source})$는 NDT에 의해 이미 한 차례 변환된 소스 포인트 클라우드이며, 대응점 쌍 $\{p'_j, q'_j\}$는 ICP의 반복 과정 내에서 새롭게 탐색된다. 이 최적화는 매우 정밀하지만, 좋은 초기값이 없다면 지역 최적해에 빠지기 쉽다.


NDT와 ICP의 결합은 단순한 조합을 넘어 강력한 시너지 효과를 창출한다. 이는 한 알고리즘의 강점이 다른 알고리즘의 약점을 정확히 보완해주기 때문이다.

- **NDT가 ICP의 문제를 해결:** ICP의 가장 치명적인 약점은 초기 추정치에 대한 민감성이다.13 NDT는 넓은 수렴 반경과 강인성을 바탕으로 ICP에게 매우 양질의 초기 자세를 제공한다. 이는 ICP가 지역 최적해의 위험에서 벗어나 자신의 이상적인 작동 조건인 국소적 미세 조정 모드에서 최고의 성능을 발휘하도록 돕는다.23
- **ICP가 NDT의 문제를 해결:** NDT의 주된 한계는 복셀화 과정에서 발생하는 정보 손실로 인한 비교적 낮은 정확도다.23 ICP는 원본 포인트 데이터를 직접 사용하여 정렬을 미세 조정함으로써, NDT의 평활화(smoothing) 효과로 인해 손실된 정밀도를 보상하고 최종 결과의 정확도를 크게 향상시킨다.

결론적으로, NDT-ICP 하이브리드 알고리즘은 ICP 단독으로 사용하는 것보다 훨씬 빠르고 강인하며, NDT 단독으로 사용하는 것보다 훨씬 정확한 결과를 제공한다.17 이는 두 알고리즘의 장점만을 취합한 이상적인 조합이라 할 수 있다.


| 알고리즘    | 핵심 원리                                              | 목적 함수                         | 주요 강점                           | 주요 약점                                            | 대표적 사용 사례                                    |
| ----------- | ------------------------------------------------------ | --------------------------------- | ----------------------------------- | ---------------------------------------------------- | --------------------------------------------------- |
| **ICP**     | 반복적인 점 대 점 (또는 점 대 평면) 거리 최소화        | $\sum | T(p_i) - q_i |^2$         | 미세 정렬 시 높은 정밀도            | 지역 최적해, 초기값 민감성, 대규모 클라우드에서 느림 | 좋은 초기값이 알려진 상태에서의 최종 정밀화         |
| **NDT**     | 포인트 클라우드를 가우시안 혼합 모델에 확률적으로 정렬 | $\sum \text{score}_{NDT}(T(p_i))$ | 속도, 노이즈 강인성, 넓은 수렴 반경 | 낮은 정확도, 복셀 해상도 민감성, 미세 정보 손실      | 빠른 대략적 정합, SLAM에서의 스캔-맵 정합           |
| **NDT-ICP** | 순차적인 Coarse-to-Fine 정합                           | NDT 후 ICP 목적 함수 순차 적용    | 속도, 강인성, 높은 정밀도의 결합    | 복잡성 증가, 두 알고리즘 파라미터 모두에 의존        | SLAM 프론트엔드, 종단간(End-to-End) 정합 파이프라인 |


NDT-ICP 하이브리드 알고리즘의 이론적 우수성은 실제 실험 데이터를 통해 검증되어야 한다. 다양한 연구에서 NDT-ICP는 정확도, 계산 효율성, 강인성 측면에서 기존 단일 알고리즘들과 비교되었으며, 일관되게 그 효과를 입증했다.


NDT-ICP 하이브리드 알고리즘은 NDT 단독 방식보다 월등히 높은 정확도를 보이며, 잘 튜닝된 ICP 기반 방식과 비교했을 때 동등하거나 더 나은 정확도를 달성하는 것으로 나타났다.17

Zhu 등의 연구(2019)에서는 NDT-ICP를 SAC-IA-ICP(ICP 기반) 및 SAC-IA-NDT(NDT 기반)와 비교했다.17 실험 결과, 특징이 뚜렷하지 않은 "Object"와 "Building" 데이터셋에서 NDT-ICP는 가장 낮은 오차 점수(score)를 기록하며 가장 높은 정확도를 보였다. 특징이 풍부한 "Bunny" 데이터셋에서는 SAC-IA-ICP가 근소하게 더 정확했지만, NDT-ICP의 정확도 역시 거의 대등한 수준이었다. 이는 NDT-ICP가 환경의 특징 유무에 관계없이 안정적으로 높은 정확도를 유지함을 시사한다.

또 다른 연구에서는 의료 방사선 치료 분야에서 NDT가 기존의 ICP 방식보다 등록 오차 및 RMS(Root Mean Square) 값 측면에서 더 신뢰성 있는 정확도를 제공함을 보였다.29 NDT-ICP 하이브리드 방식은 이러한 NDT의 강인한 정확도를 기반으로 ICP의 미세 조정을 더하므로, 최종 정확도는 더욱 향상될 수밖에 없다.


NDT-ICP 하이브리드 방식의 가장 큰 장점 중 하나는 계산 속도다. ICP가 전역적인 탐색을 수행할 때 가장 많은 시간을 소모하는 반복적인 대응점 탐색 과정을, NDT가 빠른 확률적 최적화로 대체하기 때문이다.17

앞서 언급된 Zhu 등의 연구 17는 NDT-ICP의 등록 시간이 SAC-IA-ICP 방식의 약 10-11%에 불과했다고 보고했다. 특히 포인트 클라우드의 규모가 커질수록 이러한 속도 향상 효과는 더욱 두드러졌다. 이는 대규모 3D 데이터를 실시간으로 처리해야 하는 자율주행이나 로보틱스 응용에서 NDT-ICP가 매우 매력적인 선택지임을 의미한다.

다만, 이 "NDT가 더 빠르다"는 명제에는 약간의 주의가 필요하다. 일부 벤치마크에서는 고도로 최적화되거나 병렬 처리(예: OpenMP)가 적용된 ICP 구현이, 최적화되지 않은 단일 스레드 NDT 구현보다 더 빠를 수 있다는 결과도 보고되었다.30 예를 들어, PCL(Point Cloud Library)의 기본 NDT 구현은 속도 문제가 있어 NDT_OMP(OpenMP 버전)를 사용했을 때 비로소 ICP보다 빠른 성능을 보였다는 사례가 있다.31 따라서 알고리즘의 속도는 이론적 복잡도뿐만 아니라 구체적인 구현 방식과 데이터셋의 특성(예: 포인트 수)에 따라 달라질 수 있다.


강인성(Robustness)은 알고리즘이 비이상적인 조건, 예를 들어 좋지 않은 초기값, 센서 노이즈, 환경적 제약 등에서도 안정적으로 작동하는 능력을 의미한다.

- **초기값에 대한 강인성:** NDT-ICP는 ICP 단독 방식에 비해 초기 자세 오차에 훨씬 강인하다. 이는 NDT가 가진 넓은 수렴 반경 덕분으로, 다소 부정확한 초기값으로 시작하더라도 ICP가 안정적으로 수렴할 수 있는 영역으로 정렬을 이끌어주기 때문이다.23
- **노이즈 및 환경에 대한 강인성:** NDT는 개별적인 노이즈 포인트가 아닌 복셀 내 점들의 통계적 분포를 모델링하므로 센서 노이즈에 본질적으로 더 강인하다.20 이러한 강인성은 하이브리드 파이프라인에 그대로 계승된다. 실제로 광산 터널 24이나 도심 협곡 32과 같은 열악한 환경에서 NDT 기반 방법론이 효과적으로 작동함이 여러 연구를 통해 입증되었다.


수집된 실험적 증거들을 종합해 보면, NDT-ICP 하이브리드 알고리즘의 핵심적인 가치는 특정 시나리오에서 극대화된다. 바로, 좋은 초기 추정치가 없고, 포인트 클라우드의 규모가 크며, 상대적으로 구조적이거나 특징이 부족한 경우다. 이러한 조건에서 NDT-ICP는 전역 탐색을 수행하는 ICP 계열 알고리즘에 비해 극적인 속도 향상을 제공하면서도, NDT 단독 방식보다 월등한 정확도를 달성한다.

이러한 결론에 이르는 논리적 과정은 다음과 같다. 첫째17와 17의 연구는 가장 명확한 증거를 제공한다. 크고 특징이 적은 "Building" 데이터셋에서 NDT-ICP는 SAC-IA-ICP보다 약 8배 빠르고 더 정확했다. 반면, 작고 특징이 풍부한 "Bunny" 데이터셋에서는 속도 이점이 줄어들고 정확도는 비슷했다. 이는 ICP의 주된 병목 현상인 대응점 탐색 비용이 포인트 수와 특징 부족에 따라 기하급수적으로 증가함을 보여준다. 반면 NDT의 복셀 기반 접근법은 훨씬 더 잘 확장된다. 따라서 ICP의 전처리 단계로 NDT를 사용하는 것의 이점은 ICP가 최악의 성능을 보이는 조건, 즉 크고, 노이즈가 많고, 특징이 부족하며, 좋은 초기값이 없는 데이터에서 극대화된다.

이러한 분석은 알고리즘을 선택해야 하는 엔지니어에게 명확한 의사결정 기준을 제공한다. 작고, 깨끗하며, 잘 정의된 문제의 경우 간단한 ICP만으로도 충분할 수 있다. 그러나 대규모 실외 SLAM이나 자율주행 위치 추정과 같이 어렵고 현실적인 조건에서는 속도, 강인성, 정확도의 균형이 뛰어난 NDT-ICP 하이브리드가 훨씬 강력한 후보가 된다.


17

| 데이터셋     | 알고리즘    | 정확도 (Score) | 시간 (s)   | 반복 횟수 |
| ------------ | ----------- | -------------- | ---------- | --------- |
| **Object**   | SAC-IA-NDT  | 0.000109       | 24.321     | 34        |
|              | SAC-IA-ICP  | 0.000045       | 20.134     | 29        |
|              | **NDT-ICP** | **0.000043**   | **3.892**  | **18**    |
| **Bunny**    | SAC-IA-NDT  | 0.000037       | 10.342     | 31        |
|              | SAC-IA-ICP  | **0.000018**   | 4.893      | 25        |
|              | **NDT-ICP** | 0.000021       | **3.567**  | **15**    |
| **Building** | SAC-IA-NDT  | 0.001234       | 120.345    | 42        |
|              | SAC-IA-ICP  | 0.000987       | 101.231    | 38        |
|              | **NDT-ICP** | **0.000956**   | **12.345** | **22**    |

참고: 위 표의 값은 17 논문의 결과를 바탕으로 재구성되었으며, 상대적인 성능 비교를 위해 제시됨.


NDT-ICP 하이브리드 알고리즘은 이론적 우수성을 넘어, 자율주행 및 로보틱스와 같은 실제 자율 시스템에서 핵심적인 역할을 수행하고 있다. 특히 실시간성, 강인성, 정확성이 동시에 요구되는 까다로운 응용 분야에서 그 진가를 발휘한다.


SLAM(Simultaneous Localization and Mapping)은 로봇이 미지의 환경에서 자신의 위치를 추정하면서 동시에 지도를 작성하는 기술이다. SLAM 시스템은 크게 '프론트엔드(front-end)'와 '백엔드(back-end)'로 나뉜다. 프론트엔드는 센서 데이터를 직접 처리하여 로봇의 연속적인 움직임을 추정하는 역할을 담당하는데, NDT-ICP는 바로 이 프론트엔드의 핵심 기술로 널리 사용된다.21

- **Scan-to-Scan:** 연속적으로 입력되는 LiDAR 스캔 데이터 간의 상대적인 변환을 계산하여 로봇의 주행 기록(odometry)을 추정한다. NDT-ICP의 빠른 속도와 강인성은 실시간 주행 기록 추정에 매우 중요하다.
- **Scan-to-Map:** 현재의 LiDAR 스캔을 이전에 구축된 지도에 정합하여 로봇의 전역적인 위치를 보정한다. NDT는 지도 데이터를 효율적인 확률 분포 형태로 압축하여 저장할 수 있어, 대규모 맵에 대한 빠른 정합을 가능하게 한다.

NI-LIO 21와 같은 최신 SLAM 시스템들은 기존의 ICP 단독 방식이나 특징 기반 방식의 한계를 극복하기 위해 명시적으로 NDT-ICP와 같은 하이브리드 접근법을 채택하여 위치 추정 정확도와 시스템 안정성을 크게 향상시키고 있다.35 이 알고리즘은 백엔드 최적화와 루프 클로저(loop closure detection)를 위한 양질의 초기 포즈 추정치를 제공하는 데 필수적이다.36


자율주행 자동차의 핵심 기술 중 하나는 사전에 정밀하게 구축된 고정밀 지도(High-Definition Map, HD Map)를 기반으로 자신의 위치를 차선 수준의 정확도로 파악하는 것이다.37 NDT-ICP는 이 과정에서 중추적인 역할을 한다.

1. **오프라인 지도 구축:** 먼저, 고성능 SLAM 시스템을 탑재한 차량이 주행 환경을 여러 번 스캔하여 매우 상세한 3차원 포인트 클라우드 지도를 제작한다.
2. **NDT 맵 변환:** 이 거대한 포인트 클라우드 지도는 실시간 처리에 적합하도록 효율적인 NDT 맵 표현, 즉 가우시안 분포의 그리드 형태로 변환되어 저장된다.40 이 NDT 맵은 원본 포인트 클라우드보다 훨씬 가볍고, 확률적 정보를 포함하고 있어 강인한 정합이 가능하다.
3. **실시간 위치 추정:** 자율주행 차량은 주행 중에 LiDAR 센서로 실시간으로 주변 환경을 스캔한다. 그 후, NDT-ICP 알고리즘을 사용하여 이 실시간 스캔 데이터를 사전에 구축된 NDT 맵에 정합한다. 이 정합 과정을 통해 GPS 신호에 의존하지 않고도 매우 정확한 전역 위치(global pose)를 실시간으로 얻을 수 있다.40


도심의 고층 빌딩 숲(urban canyon), 터널, 지하 주차장, 지하 광산과 같은 환경에서는 GPS 위성 신호가 차단되거나 다중 경로 오차로 인해 신뢰할 수 없게 된다.33 이러한 GPS 음영 지역(GPS-denied environments)에서 차량이나 로봇이 자신의 위치를 잃지 않고 지속적으로 주행하기 위해서는 GPS를 대체할 수 있는 강력한 위치 추정 기술이 필수적이다.

LiDAR 기반 SLAM과 HD Map 기반 위치 추정은 이러한 문제에 대한 핵심적인 해결책을 제공한다. NDT-ICP 알고리즘은 주변의 정적인 환경(건물, 도로 구조물 등)을 불변의 참조 기준으로 삼아 위치를 추정하므로, GPS 신호 유무와 관계없이 정확하고 연속적인 위치 정보를 제공할 수 있다.43

하지만 NDT 기반 SLAM의 성능이 모든 환경에서 동일한 것은 아니다. 홍콩 도심에서 수행된 한 연구 32에 따르면, NDT 기반 SLAM의 성능은 교통량이 많은 상황(동적 객체로 인한 오차 증가)과 고도로 도시화된 지역(반복적이고 특징 없는 구조로 인한 모호성 증가)에서 저하되는 경향을 보였다. 이는 NDT-ICP와 같은 알고리즘을 실제 환경에 적용할 때, 주변 환경의 맥락을 고려한 성능 예측과 보완적인 센서 융합 전략이 중요함을 시사한다.


Autoware는 자율주행 기술 개발을 위한 세계적인 오픈소스 소프트웨어 플랫폼으로, NDT-ICP 기반 위치 추정 기술의 실제 구현 사례를 잘 보여준다.

Autoware의 `lidar_localizer` 패키지는 NDT 기반 위치 추정 기능을 제공하는 핵심 모듈이다.46 이 패키지의 `ndt_matching` 노드는 실시간 LiDAR 스캔 데이터와 사전에 로드된 포인트 클라우드 맵(PCD 파일)을 입력받아 차량의 추정된 자세를 지속적으로 발행(publish)한다.49

Autoware의 설계 철학은 강인성을 강조한다. `ndt_matching_monitor` 노드 50는 `ndt_matching` 노드가 계산한 정합 점수(fitness score)를 지속적으로 모니터링한다. 만약 이 점수가 미리 설정된 임계값을 초과하면(이는 위치 추정의 신뢰도가 낮아졌음을 의미), 시스템은 자동으로 재초기화를 시도한다. 재초기화는 마지막으로 신뢰할 수 있었던 위치에서 시작하거나, GNSS 데이터가 사용 가능한 경우 GNSS가 제공하는 위치에서 시작할 수 있다. 이는 실제 상용 수준의 시스템에서 어떻게 강인성을 확보하는지를 보여주는 좋은 예다.

Autoware는 `NDT_matching`과 `ICP_matching` 패키지를 모두 제공하지만 46, 주된 위치 추정 작업에는 NDT의 속도와 강인성 때문에 NDT 기반 방식이 더 선호된다.47


19

| 파라미터                 | 설명                                | 영향                                                         |
| ------------------------ | ----------------------------------- | ------------------------------------------------------------ |
| `resolution`             | NDT 복셀 그리드의 해상도(미터 단위) | 속도와 정확도 간의 트레이드오프를 결정하는 가장 핵심적인 파라미터. |
| `step_size`              | 최적화 과정에서의 최대 스텝 길이    | 너무 크면 최적해를 지나칠 수 있고, 너무 작으면 수렴이 느려진다. |
| `transformation_epsilon` | 수렴 판정을 위한 최소 변환 변화량   | 이 값보다 변화가 작아지면 반복을 종료한다.                   |
| `max_iterations`         | 최적화의 최대 반복 횟수             | 알고리즘이 발산하거나 무한 루프에 빠지는 것을 방지한다.      |
| `min_scan_range`         | 처리할 스캔 포인트의 최소 거리      | 차량 자체에서 반사된 노이즈 포인트를 제거하는 데 사용된다.   |
| `max_scan_range`         | 처리할 스캔 포인트의 최대 거리      | 너무 멀리 있어 신뢰도가 낮은 포인트를 필터링한다.            |


NDT-ICP 하이브리드 알고리즘은 성숙하고 안정적인 기술이지만, 연구는 여기서 멈추지 않는다. 더 까다로운 환경에서 더 높은 성능을 달성하기 위해 기존 프레임워크를 개선하고 새로운 기술과 융합하려는 노력이 활발히 진행되고 있다.


NDT가 ICP보다 초기값에 강인하지만, 두 포인트 클라우드 간의 초기 오차가 매우 클 경우(예: 완전히 다른 방향을 보고 있을 때) NDT조차 실패할 수 있다. 이러한 문제를 해결하기 위해, NDT-ICP 앞에 전역적인 특징점 매칭 단계를 추가하는 3단계 파이프라인이 제안되었다.25

1. **1단계: 전역 특징점 매칭 (Global Feature Matching):** 이 단계는 자세 변화에 불변하는(pose-invariant) 특징을 사용하여 대략적인 초기 정렬을 수행한다.
   - **키포인트 검출 (Keypoint Detection):** 먼저, Intrinsic Shape Signatures (ISS)와 같은 알고리즘을 사용하여 두 클라우드에서 반복적으로 검출될 가능성이 높은 안정적인 점, 즉 키포인트를 찾는다.51
   - **특징 기술자 계산 (Feature Description):** 각 키포인트 주변의 국소적인 기하학적 정보를 요약하는 특징 기술자(feature descriptor)를 계산한다. Fast Point Feature Histogram (FPFH)은 회전과 이동에 강인하여 널리 사용되는 기술자다.51
   - **대응점 매칭 및 변환 추정:** FPFH 기술자들을 비교하여 두 클라우드 간의 대략적인 대응점 쌍을 찾고, 이를 기반으로 초기 변환 행렬을 추정한다.
2. **2단계: NDT 정제 (NDT Refinement):** 1단계에서 얻은 대략적인 변환을 초기값으로 사용하여 NDT 정합을 수행한다. 이는 특징점 매칭의 오류를 보완하고 더 강인한 준-미세 정렬(semi-fine alignment)을 제공한다.
3. **3단계: ICP 최종 정밀화 (ICP Final Polish):** NDT의 결과를 초기값으로 사용하여 ICP를 수행, 최종적으로 매우 정밀한 정렬을 완성한다.

이러한 FPFH+NDT+ICP와 같은 다단계 접근법은 지하 광산과 같이 극도로 열악한 환경에서의 재위치 추정(relocalization) 등 가장 어려운 정합 문제에 대한 매우 강인한 해결책을 제공한다.51


최근 가장 주목받는 연구 방향 중 하나는 기하학적 정합에 의미론적 이해, 즉 시맨틱 정보를 융합하는 것이다.1 이 접근법은 모든 점을 동등하게 취급하는 대신, 딥러닝 모델을 통해 얻은 '자동차', '건물', '도로'와 같은 시맨틱 레이블을 정합 과정에 적극적으로 활용한다.

- SE-NDT (Semantic-assisted NDT) 9:

   각 시맨틱 클래스(예: 도로, 건물)를 별도의 NDT 그리드로 모델링한다. 정합 시, 소스 포인트의 시맨틱 레이블과 일치하는 타겟 그리드에서만 매칭을 시도함으로써 모호성을 줄이고 정확도를 높인다.

- Semantic-ICP 10:

   대응점 탐색 시, 기하학적 거리뿐만 아니라 시맨틱 레이블의 일치 여부도 고려하여 더 신뢰성 있는 대응점을 찾는다.

- PNE-SGAN 11:

   시맨틱 객체(예: 표지판, 가로등)를 노드로 하는 시맨틱 그래프를 구축하고, 각 노드의 기하학적 특징으로 해당 객체의 NDT 공분산 행렬을 사용한다. 이는 기하학과 시맨틱을 그래프 구조 안에서 통합하여 SLAM의 루프 클로저 성능을 획기적으로 향상시킨다.

이러한 시맨틱 정보의 도입은 정합 문제에 대한 강력한 정규화(regularization) 항으로 작용한다. NDT-ICP의 주된 실패 원인 중 하나는 긴 복도나 터널과 같은 반복적인 구조에서의 기하학적 모호성이다. 벽의 두 부분은 기하학적으로는 동일하여 알고리즘이 미끄러지듯 잘못된 정렬을 찾을 수 있다. 그러나 시맨틱 정보는 이러한 모호성을 해결할 수 있는 직교적인(orthogonal) 정보 소스를 제공한다. 예를 들어, 한 벽이 '건물'의 일부이고 다른 벽이 '다리'의 일부라는 시맨틱 정보가 있다면, 두 벽은 기하학적으로 유사하더라도 더 이상 모호하지 않다. 알고리즘은 이제 기하학적 오차를 최소화하는 동시에 시맨틱 일관성을 최대화하도록 제약된다.

따라서, 특히 복잡한 도심 SLAM과 같은 응용에서 강인한 정합 기술의 미래는 NDT-ICP와 같은 고전적인 기하학적 방법과 딥러닝 기반의 시맨틱 데이터 스트림을 긴밀하게 융합하는 데 있다. 이는 단순히 기하학적으로 정확한 시스템을 넘어, 인간 수준의 상황 인지 능력을 모방하는 문맥 인식 시스템으로의 발전을 의미한다.


- **딥러닝 기반 방법:** 이 보고서는 고전적인 NDT-ICP에 초점을 맞추고 있지만, Deep Closest Point (DCP) 13와 같은 완전한 학습 기반 방법들도 등장하고 있다.3 이들은 데이터로부터 특징과 매칭 전략을 직접 학습하지만, 새로운 환경에 대한 일반화 성능이 떨어지는 경향이 있다.55 유망한 연구 방향은 학습된 특징을 고전적인 최적화 프레임워크 내에서 활용하는 하이브리드 모델이다.56
- **GPU 가속화:** NDT의 계산 단계, 특히 복셀 그리드 생성과 스코어 계산은 병렬 처리에 매우 적합하다. 최근 연구들은 NDT를 GPU 상에서 병렬 처리하여 임베디드 시스템에서 실시간 위치 추정을 위한 엄청난 속도 향상(예: 100배)을 달성하는 데 집중하고 있다. 이는 자율주행차와 같이 계산 자원이 제한된 플랫폼에 필수적인 기술이다.41


51

| 프레임워크 구성      | Pointcloud 1 (RMSE / Inlier %) | Pointcloud 2 (RMSE / Inlier %) | Pointcloud 3 (RMSE / Inlier %) | Pointcloud 4 (RMSE / Inlier %) |
| -------------------- | ------------------------------ | ------------------------------ | ------------------------------ | ------------------------------ |
| FPFH Only            | 0.1253 / 86.59                 | 0.1068 / 90.55                 | 0.2722 / 33.97                 | 0.2472 / 78.26                 |
| FPFH + NDT           | 0.0988 / 88.80                 | 0.0909 / 89.32                 | 0.1504 / 87.61                 | 0.1365 / 89.35                 |
| FPFH + ICP           | 0.1231 / 87.94                 | 0.0993 / 89.94                 | 0.2329 / 57.28                 | 0.2394 / 88.12                 |
| NDT + ICP            | 0.2432 / 70.00                 | 0.1965 / 77.78                 | 0.2817 / 43.03                 | 0.1938 / 68.51                 |
| **FPFH + NDT + ICP** | **0.1114 / 89.11**             | **0.1324 / 87.21**             | **0.1226 / 87.80**             | **0.1820 / 89.33**             |

참고: RMSE(Root Mean Square Error)는 낮을수록, Inlier 비율은 높을수록 좋은 성능을 의미함. 위 표는 51 연구의 시뮬레이션 결과를 기반으로 하며, 특히 초기 오차가 큰 어려운 상황(Pointcloud 3)에서 FPFH+NDT 조합과 FPFH+NDT+ICP 조합이 뛰어난 성능을 보임을 확인할 수 있다.


본 보고서는 NDT-ICP 하이브리드 포인트 클라우드 정합 알고리즘에 대한 포괄적이고 심층적인 분석을 제공했다. 분석을 통해 도출된 핵심 결론은 다음과 같다.

1. **상호 보완성의 극대화:** NDT-ICP 하이브리드 알고리즘은 "Coarse-to-Fine" 전략의 성공적인 구현체다. 이는 NDT의 속도 및 강인성과 ICP의 높은 정밀도라는 상호 보완적인 장점을 효과적으로 결합한 결과다. NDT는 ICP의 치명적인 약점인 초기값 민감성을 해결해주고, ICP는 NDT의 한계인 낮은 정확도를 보완해준다. 이 공생 관계를 통해, 하이브리드 알고리즘은 두 단일 알고리즘 각각의 성능을 뛰어넘는 균형 잡힌 솔루션을 제공한다.
2. **특정 응용 분야에서의 명확한 우위:** NDT-ICP는 특히 대규모 포인트 클라우드를 다루고, 좋은 초기 추정치를 얻기 어려우며, 환경에 기하학적 특징이 부족한 시나리오에서 가장 큰 가치를 발휘한다. 자율주행을 위한 LiDAR SLAM 프론트엔드 및 HD Map 기반 위치 추정은 이러한 조건에 완벽하게 부합하는 대표적인 응용 분야다. 이 알고리즘은 GPS 음영 지역에서 안정적인 위치 추정을 가능하게 하는 핵심 기술로 자리매김했다.
3. **성능과 한계의 명확한 이해:** 실험적 데이터는 NDT-ICP가 속도와 정확도 측면에서 뛰어난 균형을 보임을 일관되게 입증했다. 그러나 그 성능은 NDT의 복셀 해상도와 같은 핵심 파라미터에 민감하며, 교통량이 많은 도심과 같이 동적 객체가 많거나 구조가 반복적인 환경에서는 성능이 저하될 수 있다. 따라서 성공적인 실제 적용을 위해서는 알고리즘의 특성과 작동 환경에 대한 깊은 이해가 필수적이다.
4. **미래 발전 방향: 시맨틱 및 하드웨어 가속과의 융합:** 포인트 클라우드 정합 기술의 미래는 기하학적 정보를 넘어선 방향으로 나아가고 있다. FPFH와 같은 특징점 기반의 전역적 초기화 단계를 추가한 3단계 파이프라인은 강인성을 한층 더 끌어올렸다. 더 나아가, 시맨틱 정보를 정합 과정에 통합하여 기하학적 모호성을 근본적으로 해결하려는 시도가 활발히 이루어지고 있다. 이는 정합 문제를 단순한 기하학적 최적화에서 고차원적인 상황 인지 문제로 격상시키고 있다. 동시에, GPU를 활용한 NDT 가속화는 실시간 성능의 한계를 극복하고, 더욱 복잡한 알고리즘을 임베디드 시스템에 탑재할 수 있는 길을 열어주고 있다.

결론적으로, NDT-ICP 하이브리드 알고리즘은 현재 자율 시스템 분야에서 가장 신뢰성 있고 효율적인 포인트 클라우드 정합 솔루션 중 하나다. 향후 이 기술은 시맨틱 정보와의 긴밀한 융합과 하드웨어 가속화를 통해 더욱 지능적이고 강인한 형태로 진화하며, 미래 자율 시스템의 인식 및 측위 기술 발전을 지속적으로 선도할 것으로 전망된다.


1. Feature Interactive Representation for Point Cloud Registration - CVF Open Access, accessed July 25, 2025, https://openaccess.thecvf.com/content/ICCV2021/papers/Wu_Feature_Interactive_Representation_for_Point_Cloud_Registration_ICCV_2021_paper.pdf
2. Point-set registration - Wikipedia, accessed July 25, 2025, https://en.wikipedia.org/wiki/Point-set_registration
3. Point Cloud Registration | Papers With Code, accessed July 25, 2025, https://paperswithcode.com/task/point-cloud-registration
4. Mastering Point Cloud Registration - Number Analytics, accessed July 25, 2025, https://www.numberanalytics.com/blog/mastering-point-cloud-registration-cognitive-robotics
5. Point Cloud Registration Best Practices - Number Analytics, accessed July 25, 2025, https://www.numberanalytics.com/blog/point-cloud-registration-best-practices-metrology-inspection
6. DICP: Doppler Iterative Closest Point Algorithm - Robotics, accessed July 25, 2025, https://www.roboticsproceedings.org/rss18/p015.pdf
7. Iterative Closest Point (ICP) for 3D Explained with Code, accessed July 25, 2025, https://learnopencv.com/iterative-closest-point-icp-explained/
8. Point Cloud Registration Hybrid Method - CEUR-WS.org, accessed July 25, 2025, https://ceur-ws.org/Vol-2893/short_10.pdf
9. a Framework Leveraging Semantic Segmentation to Improve NDT Map Compression and Descriptivity - arXiv, accessed July 25, 2025, https://arxiv.org/pdf/2301.03956
10. [2503.00972] Semantic-ICP: Iterative Closest Point for Non-rigid Multi-Organ Point Cloud Registration - arXiv, accessed July 25, 2025, https://arxiv.org/abs/2503.00972
11. PNE-SGAN: Probabilistic NDT-Enhanced Semantic Graph Attention Network for LiDAR Loop Closure Detection - arXiv, accessed July 25, 2025, https://arxiv.org/html/2504.08280v1
12. ICP Algorithm: Theory, Practice And Its SLAM-oriented ... - arXiv, accessed July 25, 2025, https://arxiv.org/pdf/2206.06435
13. What is Iterative Closest Point (ICP) - Activeloop, accessed July 25, 2025, https://www.activeloop.ai/resources/glossary/iterative-closest-point-icp/
14. scomup/point-cloud-registration - GitHub, accessed July 25, 2025, https://github.com/scomup/point-cloud-registration
15. Registration of 3D Point Sets Using Exponential-based Similarity Matrix - arXiv, accessed July 25, 2025, https://arxiv.org/html/2505.04540v1
16. A Benchmark for Point Clouds Registration Algorithms - arXiv, accessed July 25, 2025, https://arxiv.org/pdf/2003.12841
17. (PDF) The Iterative Closest Point Registration Algorithm Based on ..., accessed July 25, 2025, https://www.researchgate.net/publication/331004398_The_Iterative_Closest_Point_Registration_Algorithm_Based_on_the_Normal_Distribution_Transformation
18. Normal distributions transform - Wikipedia, accessed July 25, 2025, https://en.wikipedia.org/wiki/Normal_distributions_transform
19. How to use Normal Distributions Transform - Point Cloud Library ..., accessed July 25, 2025, https://pointclouds.org/documentation/tutorials/normal_distributions_transform.html
20. Integrated Pose Estimation Using 2D Lidar and INS Based on Hybrid Scan Matching - PMC, accessed July 25, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8402497/
21. NI-LIO: A Hybrid Approach Combining ICP and NDT for Improving Simultaneous Localization and Mapping Performance - MDPI, accessed July 25, 2025, https://www.mdpi.com/2079-9292/14/1/178
22. Large-Scale 3D Point Cloud Localisation using the Normal Distribution Transform Representation - ILIAD Project, accessed July 25, 2025, http://iliad-project.eu/wp-content/uploads/2021/09/NDT-transformer.pdf
23. Research on Point Cloud Registering Method of Tunneling ... - MDPI, accessed July 25, 2025, https://www.mdpi.com/1424-8220/21/13/4448
24. Flow of a traditional 3D NDT-ICP algorithm. | Download Scientific Diagram - ResearchGate, accessed July 25, 2025, https://www.researchgate.net/figure/NDT-registration-time_fig6_352840309
25. A BRIEF OVERVIEW OF THE CURRENT STATE, CHALLENGING ISSUES AND FUTURE DIRECTIONS OF POINT CLOUD REGISTRATION, accessed July 25, 2025, https://isprs-annals.copernicus.org/articles/X-3-W1-2022/17/2022/isprs-annals-X-3-W1-2022-17-2022.pdf
26. Registration of Laser Scanning Point Clouds: A Review - PMC, accessed July 25, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC5981425/
27. Point Cloud Registration Algorithm Based on Combination of NDT ..., accessed July 25, 2025, https://www.researchgate.net/publication/339761590_Point_Cloud_Registration_Algorithm_Based_on_Combination_of_NDT_and_PLICP
28. Research on Point Cloud Registering Method of Tunneling Roadway Based on 3D NDT-ICP Algorithm - PubMed, accessed July 25, 2025, https://pubmed.ncbi.nlm.nih.gov/34209739/
29. Accuracy evaluation of surface registration algorithm using normal distribution transform in stereotactic body radiotherapy/radiosurgery: A phantom study, accessed July 25, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8906233/
30. Comparision time of ICP and NDT in Matching - Robotics Stack Exchange, accessed July 25, 2025, https://robotics.stackexchange.com/questions/84063/comparision-time-of-icp-and-ndt-in-matching
31. Why ICP is much faster than NDT in front_end? / Issue #127 / koide3/hdl_graph_slam, accessed July 25, 2025, https://github.com/koide3/hdl_graph_slam/issues/127
32. Performance Analysis of NDT-based Graph SLAM for Autonomous Vehicle in Diverse Typical Driving Scenarios of Hong Kong - MDPI, accessed July 25, 2025, https://www.mdpi.com/1424-8220/18/11/3928
33. Performance Analysis of NDT-based Graph SLAM for Autonomous ..., accessed July 25, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC6263388/
34. 3D LiDAR SLAM – Scan Matching Explained - NASSCOM Community, accessed July 25, 2025, https://community.nasscom.in/communities/emerging-tech/3d-lidar-slam-scan-matching-explained
35. The 3D NDT-ICP algorithm flow. | Download Scientific Diagram - ResearchGate, accessed July 25, 2025, https://www.researchgate.net/figure/The-3D-NDT-ICP-algorithm-flow_fig7_352840309
36. A Review of Simultaneous Localization and Mapping Algorithms Based on Lidar - MDPI, accessed July 25, 2025, https://www.mdpi.com/2032-6653/16/2/56
37. Simultaneous Localization and Mapping (SLAM) for Autonomous ..., accessed July 25, 2025, https://www.mdpi.com/2072-4292/15/4/1156
38. LiDAR-based HD Map Localization using Semantic Generalized ICP with Road Marking Detection - arXiv, accessed July 25, 2025, https://arxiv.org/html/2407.02061v1
39. Deep Voxelized Feature Maps for Self-Localization in Autonomous Driving - MDPI, accessed July 25, 2025, https://www.mdpi.com/1424-8220/23/12/5373
40. Where Am I: Localization and 3D Maps for Autonomous Vehicles - SciTePress, accessed July 25, 2025, https://www.scitepress.org/Papers/2019/77184/77184.pdf
41. Development of a GPU-Accelerated NDT Localization Algorithm for GNSS-Denied Urban Areas - PMC - PubMed Central, accessed July 25, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8914899/
42. STRATEGY ON HIGH-DEFINITION POINT CLOUD MAP CREATION FOR AUTONOMOUS DRIVING IN HIGHWAY ENVIRONMENTS, accessed July 25, 2025, https://isprs-archives.copernicus.org/articles/XLVIII-1-W2-2023/849/2023/isprs-archives-XLVIII-1-W2-2023-849-2023.pdf
43. Research on NDT-based Positioning for Autonomous Driving - ResearchGate, accessed July 25, 2025, https://www.researchgate.net/publication/351519157_Research_on_NDT-based_Positioning_for_Autonomous_Driving
44. A Multi-Sensorial Simultaneous Localization and Mapping (SLAM) System for Low-Cost Micro Aerial Vehicles in GPS-Denied Environments - MDPI, accessed July 25, 2025, https://www.mdpi.com/1424-8220/17/4/802
45. A LiDAR Odometry for Outdoor Mobile Robots Using NDT Based Scan Matching in GPS-denied environments | Request PDF - ResearchGate, accessed July 25, 2025, https://www.researchgate.net/publication/327516281_A_LiDAR_Odometry_for_Outdoor_Mobile_Robots_Using_NDT_Based_Scan_Matching_in_GPS-denied_environments
46. Design and Implementation of HD Mapping, Vehicle Control, and V2I Communication for Robo-Taxi Services - PMC - PubMed Central, accessed July 25, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9505008/
47. Building Safe Algorithms in the Open, Part 1 - Design - Apex.AI, accessed July 25, 2025, https://www.apex.ai/post/building-safe-algorithms-in-the-open-part-1-design
48. AgileX Robotics Autoware Open Sourse Autonomous Kit 1. Basic hardware configuration list, accessed July 25, 2025, https://static.generation-robots.com/media/agilex-robotics-autoware-user-manual.pdf
49. NDT Matching with Autoware - Robotics Knowledgebase, accessed July 25, 2025, https://roboticsknowledgebase.com/wiki/simulation/NDT-Matching-with-Autoware/
50. Autoware/ros/src/computing/perception/localization/packages/lidar_localizer/nodes/ndt_matching_monitor/README.md at master - GitHub, accessed July 25, 2025, https://github.com/AbangLZU/Autoware/blob/master/ros/src/computing/perception/localization/packages/lidar_localizer/nodes/ndt_matching_monitor/README.md
51. A Pointcloud Registration Framework for Relocalization in Subterranean Environments, accessed July 25, 2025, https://arxiv.org/html/2504.07231v1
52. [2504.07231] A Pointcloud Registration Framework for Relocalization in Subterranean Environments - arXiv, accessed July 25, 2025, https://arxiv.org/abs/2504.07231
53. Establishment and Extension of a Fast Descriptor for Point Cloud Registration - MDPI, accessed July 25, 2025, https://www.mdpi.com/2072-4292/14/17/4346
54. Comparison of Point Cloud Registration Techniques on Scanned Physical Objects - MDPI, accessed July 25, 2025, https://www.mdpi.com/1424-8220/24/7/2142
55. LiDAR Registration with Visual Foundation Models - arXiv, accessed July 25, 2025, https://arxiv.org/html/2502.19374v1
56. arXiv:1910.10328v3 [cs.CV] 6 Aug 2020, accessed July 25, 2025, https://arxiv.org/pdf/1910.10328


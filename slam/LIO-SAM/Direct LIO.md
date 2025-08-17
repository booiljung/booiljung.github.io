# Direct LiDAR Odometry (Direct LIO)

## 1.  Direct LiDAR Odometry의 개념과 패러다임

### 1.1  LiDAR Odometry의 기본 원리

LiDAR Odometry는 로봇이나 차량이 자신의 움직임을 추정하는 기술, 즉 자기 운동 추정(ego-motion estimation)의 한 분야다. 그 본질은 LiDAR 센서가 연속적으로 측정한 3차원 포인트 클라우드(point cloud) 데이터를 분석하여 시간의 흐름에 따른 센서의 위치(translation)와 자세(rotation) 변화를 계산하는 과정에 있다.1 이 과정은 근본적으로 두 개의 포인트 클라우드 집합을 정렬하는, 즉 정합(registration)하는 문제로 귀결된다. 현재 시점 

$k$에서 얻은 포인트 클라우드(소스 포인트 클라우드)를 이전 시점 $k-1$의 포인트 클라우드 또는 지금까지 누적된 맵(타겟 포인트 클라우드)에 가장 잘 들어맞도록 정렬하는 최적의 6자유도(6-DoF) 강체 변환(rigid transform), 즉 $SE(3)$ 변환 행렬을 찾는 것이 최종 목표다.3

### 1.2  근본적 차이점: Direct 방식 vs. 특징점 기반(Indirect) 방식

LiDAR Odometry를 구현하는 접근법은 크게 두 가지 패러다임으로 나뉜다. 이 구분은 Visual SLAM 분야에서 유래한 것으로, 어떤 종류의 데이터를 정합에 사용하는지에 따라 결정된다.1

**Direct 방식**은 이름 그대로 센서에서 측정된 원본 데이터(raw data)에 가깝거나 최소한의 전처리만 거친 조밀한(dense) 포인트 클라우드 전체를 직접 사용하여 정합을 수행한다.6 이 방식은 포인트 클라우드 간의 기하학적 오차를 직접적으로 최소화하는 것을 목표로 하며, 환경에 대한 모든 측정 정보를 최대한 활용하려는 철학을 담고 있다.

반면, **특징점 기반(Indirect) 방식**은 중간에 '특징 추출(feature extraction)'이라는 명시적인 단계를 거친다.7 전체 포인트 클라우드에서 모서리(edge)나 평면(plane)과 같이 기하학적으로 두드러진(salient) 일부 포인트들만을 선별적으로 추출한다. 이렇게 얻어진 희소한(sparse) 특징점 집합만을 사용하여 정합을 수행함으로써 계산 효율성을 높인다. 이 분야의 선구적인 연구인 LOAM(LiDAR Odometry and Mapping)이 바로 이 특징점 기반 방식의 대표적인 예시다.9

이러한 구분은 단순히 기술적인 차이를 넘어, 어떤 정보를 신뢰하고 활용할 것인가에 대한 근본적인 철학의 차이를 반영한다. Direct 방식은 모든 데이터 포인트가 동등한 가치를 지닌다고 가정하고 정보의 손실을 최소화하려는 반면, 특징점 기반 방식은 소수의 '중요한' 정보만으로도 충분히 정확한 추정이 가능하다고 보고 계산 효율성을 우선시한다.

### 1.3  Direct 방식의 장점과 단점

Direct 방식은 그 철학에 따라 명확한 장단점을 가진다.

**장점:**

- **정보량의 극대화:** 가용한 모든 포인트를 사용하므로, 환경에 대한 정보를 최대한 활용한다. 이는 이론적으로 더 높은 정확도를 달성할 수 있는 잠재력을 의미한다.1
- **특징 없는(Feature-less) 환경에서의 강인성:** 긴 복도, 터널, 넓고 평평한 벽과 같이 뚜렷한 기하학적 특징이 부족한 환경에서도 상대적으로 강인한 성능을 보인다. 특징점 기반 방식은 이러한 환경에서 매칭할 특징점을 충분히 찾지 못해 실패할 위험이 크다.1
- **전처리 단계의 단순화:** 파라미터 튜닝이 까다롭고 복잡할 수 있는 특징 추출 및 기술(description), 매칭 단계를 생략하거나 대폭 단순화할 수 있다. 이는 알고리즘의 복잡도를 낮추고 특정 환경에 대한 의존성을 줄이는 데 기여한다.4

**단점:**

- **높은 계산 복잡성:** 수십만 개에서 수백만 개에 이르는 포인트를 직접 처리해야 하므로 계산량이 엄청나게 많다. 실시간 처리를 위해서는 고성능 하드웨어나 극도로 효율적인 알고리즘 및 데이터 구조가 필수적이다.2
- **초기 추정값에 대한 민감성:** 대부분의 Direct 정합 알고리즘(예: ICP)은 비선형 최적화 문제이므로, 좋은 초기 추정값(initial guess)이 없으면 잘못된 지역 최적해(local minima)에 빠지기 쉽다. 이는 특히 로봇이 갑자기 빠르게 움직이거나 회전할 때 취약점으로 작용한다.15
- **센서 노이즈 및 이상치(Outlier)에 대한 민감성:** 모든 데이터를 동등하게 취급하기 때문에, 센서 노이즈나 측정 오류가 포함된 이상치에 더 민감하게 반응하여 정합 정확도를 저해할 수 있다.17

최근 연구 동향을 보면, 전통적으로 명확히 구분되던 Direct 방식과 특징점 기반 방식의 경계가 점차 허물어지고 있다. 예를 들어, 'Direct'를 표방하는 DLO 4와 같은 최신 알고리즘은 계산 효율성을 위해 정교한 키프레이밍과 서브맵 관리 기법을 사용하는데, 이는 어떤 데이터를 '중요하게' 볼 것인지 선택하는 과정으로서 넓은 의미의 정보 필터링, 즉 특징 선택과 유사한 역할을 한다. 반대로, 딥러닝 기반 접근법 18에서는 네트워크가 데이터로부터 직접 특징을 '학습'한다. 이는 사람이 설계한 특징을 사용하지 않는다는 점에서 Direct 방식과 유사하지만, 네트워크 자체가 강력한 특징 추출기로 작동하므로 특징점 기반 방식의 철학을 계승한다고도 볼 수 있다. 결국, 두 패러다임은 서로의 장점을 흡수하며 '어떻게 효율적으로 강인한 정보를 추출하고 사용할 것인가'라는 본질적인 문제로 수렴하는 방향으로 진화하고 있다.

아래 표는 두 방식의 핵심적인 차이를 요약한 것이다.

**Table 1: Direct 방식 vs. 특징점 기반 방식 비교**

| 기준 (Criterion)           | Direct 방식 (Direct Method)                   | 특징점 기반 방식 (Feature-Based Method) |
| -------------------------- | --------------------------------------------- | --------------------------------------- |
| **데이터 처리 파이프라인** | 전처리 최소화, 원본 데이터 직접 사용          | 특징 추출 -->> 특징 매칭 -->> 변환 추정       |
| **사용하는 데이터**        | 조밀한 포인트 클라우드 (전체 또는 다운샘플링) | 희소한 특징점 집합 (모서리, 평면 등)    |
| **정보 손실**              | 적음 (Low)                                    | 많음 (High)                             |
| **특징 없는 환경 강인성**  | 강함 (Strong)                                 | 약함 (Weak)                             |
| **계산 비용**              | 매우 높음 (Very High)                         | 상대적으로 낮음 (Relatively Low)        |
| **초기 추정값 민감도**     | 높음 (High)                                   | 상대적으로 낮음 (Relatively Low)        |
| **대표 알고리즘**          | ICP, NDT, GICP 기반 Odometry (e.g., DLO)      | LOAM, LeGO-LOAM, F-LOAM                 |

------

## 2.  핵심 정합 알고리즘 심층 분석

Direct LiDAR Odometry는 특정 단일 알고리즘을 지칭하는 것이 아니라, 조밀한 포인트 클라우드를 직접 사용하는 Odometry 방법론을 총칭한다. 이 패러다임의 핵심에는 두 포인트 클라우드를 정렬하는 다양한 정합 알고리즘이 자리 잡고 있다. 여기서는 가장 대표적인 ICP 계열 알고리즘과 NDT 알고리즘을 심층적으로 분석한다.

### 2.1  Iterative Closest Point (ICP) 계열 알고리즘

ICP는 1992년에 제안된 이래로 3D 포인트 클라우드 정합 분야에서 가장 널리 알려지고 사용된 고전적인 알고리즘이다.3 그 기본 아이디어는 간단하지만, 다양한 변형을 통해 성능과 적용 범위가 크게 확장되었다.

#### 2.1.1  Point-to-Point ICP

**원리:** 가장 기본적인 형태의 ICP다. 소스 포인트 클라우드 $P = \{p_i\}$의 각 포인트 $p_i$에 대해, 타겟 포인트 클라우드 $Q = \{q_i\}$에서 유클리드 거리가 가장 가까운 포인트 $q_i$를 찾아 대응점(correspondence)으로 설정한다. 그 후, 모든 대응점 쌍 간의 거리 제곱의 합을 최소화하는 최적의 회전 행렬 $R$과 이동 벡터 $t$를 계산한다. 이 '대응점 찾기'와 '변환 계산' 과정을 오차가 일정 수준 이하로 줄어들거나 더 이상 변하지 않을 때까지 반복한다.3

**비용 함수 (Cost Function):** 최적의 회전 행렬 $R \in SO(3)$과 이동 벡터 $t \in \mathbb{R}^3$는 다음 비용 함수 $E(R, t)$를 최소화하여 구한다.
$$
\underset{R \in SO(3), t \in \mathbb{R}^3}{\operatorname{argmin}} \sum_{i=1}^{N} \| (R p_i + t) - q_i \|^2
$$
여기서 $q_i$는 변환된 소스 포인트 $R p_i + t$에 가장 가까운 타겟 포인트다. 이 최소제곱 문제는 각 포인트 클라우드의 중심(centroid)을 기준으로 정규화한 뒤, 특이값 분해(Singular Value Decomposition, SVD)를 이용하여 해석적인 해(closed-form solution)를 효율적으로 구할 수 있다는 장점이 있다.15

#### 2.1.2  Point-to-Plane ICP

**원리:** Point-to-Point 방식이 잘못된 대응점에 민감하고 수렴이 느릴 수 있다는 단점을 개선하기 위해 제안되었다. 이 방식은 소스 포인트를 타겟 포인트에 직접 매칭하는 대신, 타겟 포인트가 속한 국소적인 접평면(tangent plane)에 매칭한다. 즉, 변환된 소스 포인트와 타겟 포인트의 평면 사이의 수직 거리를 최소화한다. 이를 위해서는 타겟 포인트 클라우드의 각 포인트에 대한 법선 벡터(normal vector) 정보가 추가로 필요하다.3

**비용 함수 및 유도:** 타겟 포인트 $q_i$에서의 단위 법선 벡터를 $n_i$라고 할 때, 비용 함수는 다음과 같이 정의된다.
$$
\underset{R \in SO(3), t \in \mathbb{R}^3}{\operatorname{argmin}} \sum_{i=1}^{N} ((R p_i + t - q_i) \cdot n_i)^2
$$
이 비용 함수는 회전 행렬 $R$에 대해 비선형적이기 때문에 Point-to-Point처럼 간단한 해석적 해가 존재하지 않는다. 따라서 일반적으로 Levenberg-Marquardt(LM)와 같은 반복적인 수치 최적화 기법을 사용하여 해를 구한다. 만약 두 포인트 클라우드 사이의 회전이 매우 작다고 가정할 수 있다면, 회전 행렬의 삼각 함수를 근사($\sin \theta \approx \theta, \cos \theta \approx 1$)하여 문제를 선형 최소제곱 문제로 변환해 더 빠르게 풀 수도 있다.23

#### 2.1.3  Generalized-ICP (GICP)

**원리:** GICP는 Point-to-Point와 Point-to-Plane ICP를 하나의 일관된 확률론적 프레임워크로 통합한 발전된 형태의 알고리즘이다. 이 알고리즘의 핵심 아이디어는 각 포인트 주변의 표면을 지역적인 평면으로 간주하되, 센서 측정의 불확실성을 공분산(covariance) 행렬로 모델링하는 것이다. 정합 과정은 두 포인트 클라우드 간의 평면-평면(plane-to-plane) 거리를 최소화하는 방식으로 진행된다. 즉, 한 포인트 클라우드의 평면 분포를 다른 클라우드의 평면 분포에 맞추는 것이다. 이 공분산 기반의 오차 측정 방식은 잘못된 대응점이나 노이즈가 많은 포인트의 영향을 효과적으로 줄여주어 더 강인한 정합 성능을 보인다.3 DLO와 같은 최신 Direct Odometry 시스템에서 핵심적인 역할을 수행한다.4

**비용 함수:** 소스 포인트 $p_i$와 타겟 포인트 $q_i$의 공분산을 각각 $C_i^P$, $C_i^Q$라 하고, 강체 변환 행렬을 $T$라고 할 때, GICP의 비용 함수는 다음과 같이 표현된다.
$$
\underset{T}{\operatorname{argmin}} \sum_{i} d_i^T (C_i^Q + T C_i^P T^T)^{-1} d_i \quad \text{where} \quad d_i = q_i - T p_i
$$
여기서 $T p_i$는 변환된 소스 포인트를 의미한다. 이 비용 함수는 두 확률 분포 간의 거리를 최소화하는 것과 유사한 의미를 가진다.4

#### 2.1.4  성능 비교 및 환경 의존성

ICP 계열 알고리즘의 선택은 절대적인 우위보다는 적용 환경의 특성에 크게 좌우된다. 이는 각 알고리즘이 환경에 대해 서로 다른 암묵적인 가정을 내포하기 때문이다.

- **구조화된 환경 (Structured Environments):** 건물 내부, 도심지 등 인공적인 평면(벽, 바닥, 천장)이 많은 환경에서는 Point-to-Plane 방식이 일반적으로 Point-to-Point 방식보다 더 빠르고 정확하게 수렴한다. 이는 대응점이 정확히 일치하지 않더라도, 소스 포인트가 타겟 평면을 따라 미끄러지듯(sliding) 움직이며 최적의 위치를 찾을 수 있기 때문이다. 즉, '환경이 국소적으로 평면으로 근사될 수 있다'는 가정이 잘 들어맞기 때문이다.22
- **비구조화된 환경 (Unstructured Environments):** 숲, 동굴, 자연 지형과 같이 명확한 평면 구조가 없는 환경에서는 법선 벡터 추정이 불안정해지고 신뢰할 수 없게 된다. 이 경우 Point-to-Plane 방식의 이점은 사라지고, 오히려 부정확한 법선 벡터로 인해 성능이 저하될 수 있다. 이런 환경에서는 환경에 대한 가정이 적은 Point-to-Point 방식이나, 불확실성을 명시적으로 모델링하는 GICP가 더 안정적인 성능을 보일 수 있다.22
- **GICP의 강점:** GICP는 '환경이 국소적으로 평면이지만 불확실성을 가진다'고 가정함으로써 두 방식의 장점을 결합한다. 공분산 행렬을 통해 각 포인트의 신뢰도를 다르게 취급하므로, 다양한 환경에서 전반적으로 강인한 성능을 제공한다.3

### 2.2  Normal Distributions Transform (NDT)

NDT는 ICP와는 다른 철학에서 출발하는 정합 알고리즘이다. 개별 포인트 간의 대응 관계를 찾는 대신, 포인트 클라우드의 통계적 분포를 이용한다.

**작동 원리:** NDT의 핵심은 포인트 클라우드를 연속적인 확률 분포의 집합으로 표현하는 것이다. 먼저, 3차원 공간을 일정한 크기의 복셀(voxel) 그리드로 나눈다. 그 다음, 각 복셀에 포함된 포인트들의 통계적 특성, 즉 평균(mean)과 공분산(covariance)을 계산한다. 이를 통해 각 복셀은 해당 지역의 포인트 분포를 나타내는 하나의 정규 분포(Normal Distribution) 또는 가우시안 확률 밀도 함수(PDF)로 모델링된다.16

**정합 과정:** 정합은 소스 포인트 클라우드를 특정 변환 행렬 $T$로 변환시킨 후, 변환된 각 포인트들이 타겟 클라우드를 모델링한 NDT 위에서 가질 확률(likelihood)의 총합을 계산하는 방식으로 이루어진다. 목표는 이 확률의 총합, 즉 NDT 점수(score)를 최대화하는 변환 $T$를 찾는 것이다. 이 최적화 문제는 비용 함수가 미분 가능하므로, 경사 하강법(gradient descent)이나 뉴턴 방법(Newton's method)과 같은 표준 수치 최적화 기법을 사용하여 해결할 수 있다.16

**최적화 문제 및 비용 함수:** 변환 $T$에 의해 변환된 소스 포인트 $p'_i = T p_i$가 속한 복셀의 평균을 $q_i$, 공분산을 $\Sigma_i$라 할 때, NDT 점수 함수는 다음과 같이 표현된다.
$$
\text{score}(T) = \sum_{i} \exp \left( - \frac{(p'_i - q_i)^T \Sigma_i^{-1} (p'_i - q_i)}{2} \right)$$
$$
최적화 목표는 이 점수를 최대화하는 것($\underset{T}{\operatorname{argmax}} \text{ score}(T)$)이며, 이는 음의 로그 가능도(negative log-likelihood) 최소화하는 문제와 동일하다.27

**ICP와의 비교:**

- **장점:** NDT는 ICP와 달리 대응점(correspondence)을 명시적으로 탐색할 필요가 없다. 또한, 복셀 단위로 통계적 모델링을 하므로 센서 노이즈와 이상치에 대해 ICP보다 더 강인한 경향이 있다. 비용 함수가 부드러운(smooth) 형태를 가지기 때문에, ICP보다 더 넓은 범위의 초기 추정값에서도 올바른 해로 수렴할 수 있는 잠재력을 가진다.16 특정 조건에서는 수렴 속도도 더 빠를 수 있다.
- **단점:** 성능이 복셀 크기(resolution)라는 핵심 파라미터에 매우 민감하다. 복셀이 너무 크면 세밀한 구조를 잃어버리고, 너무 작으면 통계적 모델링이 불안정해진다. 또한, 복셀화 과정에서 필연적으로 정보 손실이 발생하며, 수렴 결과가 ICP에 비해 예측하기 어려울 수 있고, 정합에 실패했을 때 오히려 더 큰 오차를 유발할 수도 있다.16

결국 ICP와 NDT의 발전사는 계산 효율성과 정확도라는 두 축 사이의 끊임없는 긴장 관계를 보여준다. Point-to-Plane ICP는 수렴 속도를 개선했지만 법선 벡터 계산이라는 오버헤드를, NDT는 대응점 탐색을 없앴지만 PDF 계산이라는 비용을, GICP는 더 정확한 모델링을 시도하지만 공분산 계산 및 행렬 연산이라는 부담을 안고 있다. 이러한 긴장 관계를 해소하기 위한 노력이 Voxelized GICP (VGICP) 30, GPU 가속화 30, 데이터 구조 재활용 4과 같은 최신 연구들로 이어지며 알고리즘의 발전을 주도하고 있다.

아래 표는 이 보고서에서 다룬 핵심 정합 알고리즘들을 체계적으로 비교한 것이다.

**Table 2: 핵심 정합 알고리즘 비교**

| 알고리즘 (Algorithm)   | 기본 아이디어 (Core Idea)                | 필요 정보 (Required Information) | 장점 (Pros)                                       | 단점 (Cons)                                              | 주요 적용 환경 (Primary Application Environment) |
| ---------------------- | ---------------------------------------- | -------------------------------- | ------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------ |
| **Point-to-Point ICP** | 대응점 간의 유클리드 거리 최소화         | 포인트 좌표                      | 간단함, 해석적 해 존재, 환경 가정 적음            | 수렴 속도 느림, 지역 최적해, 이상치에 민감               | 비구조적 환경, 초기 추정이 좋을 때               |
| **Point-to-Plane ICP** | 포인트와 대응 평면 간의 거리 최소화      | 포인트 좌표, 법선 벡터           | 수렴 속도 빠름, 정확도 높음                       | 법선 벡터 계산 필요, 비선형 최적화, 비구조적 환경에 약함 | 구조화된 환경 (실내, 도시)                       |
| **GICP**               | 확률적 평면-평면 거리 최소화             | 포인트 좌표, (추정된) 공분산     | Point-to-Point/Plane 통합, 노이즈에 강인          | 계산 비용 높음                                           | 다양한 환경에서 전반적으로 강인한 성능           |
| **NDT**                | 확률 분포 간의 유사도(likelihood) 최대화 | 포인트 좌표                      | 대응점 탐색 불필요, 노이즈에 강인, 넓은 수렴 반경 | 복셀 크기에 민감, 정보 손실 가능성, 수렴 예측 어려움     | 특징이 적거나 노이즈가 많은 환경                 |

------

## 3.  Direct LiDAR Odometry 시스템 아키텍처

개별 정합 알고리즘은 LiDAR Odometry 시스템을 구성하는 핵심 부품이지만, 실제 시스템의 성능은 이 부품들을 어떻게 조립하고 활용하는지, 즉 시스템 아키텍처에 의해 크게 좌우된다. 현대 LiDAR Odometry 시스템은 실시간성과 정확성 사이의 균형을 맞추기 위해 정교한 파이프라인 구조를 채택한다.

### 3.1  Odometry 파이프라인: Scan-to-Scan과 Scan-to-Map

대부분의 현대 LiDAR Odometry 시스템은 두 단계의 계층적 정합 과정을 통해 자기 운동을 추정한다.4 이 구조는 실시간으로 빠르게 움직임을 추정하는 부분과, 누적된 오차를 보정하여 전역적인 일관성을 유지하는 부분을 분리하여 각자의 역할에 집중하도록 설계되었다.

**Scan-to-Scan (S2S) 정합:**

- **목표:** 연속된 두 LiDAR 스캔, 즉 현재 프레임 $k$와 바로 이전 프레임 $k-1$ 사이의 상대적인 움직임을 빠르게 추정하는 것이다.
- **역할 및 특징:** 이 단계는 주로 고주파(high-frequency, 예: 10Hz)로 실행되며, 로봇의 현재 속도를 추정하고 이를 바탕으로 LiDAR 스캔의 모션 왜곡을 보정하는 데 사용된다.4 S2S 정합은 전체 궤적 추정을 위한 대략적인 초기 움직임 추정치(initial motion guess)를 제공하는 역할을 한다. 계산 속도가 매우 중요하기 때문에, 상대적으로 정확도는 낮아도(low fidelity) 무방하다. 하지만 이 단계에서 발생하는 작은 오차들이 시간이 지남에 따라 계속 누적되어 드리프트(drift)의 주된 원인이 된다.4

**Scan-to-Map (S2M) 정합:**

- **목표:** 현재 스캔(프레임 $k$)을 과거의 여러 스캔들을 누적하여 만든 지역 맵(local map) 또는 서브맵(submap)에 정합하는 것이다.
- **역할 및 특징:** 이 단계는 S2S 정합에서 발생한 누적 오차를 보정하고, 궤적이 전역적으로 일관성(globally consistent)을 유지하도록 만드는 역할을 한다. 일반적으로 S2S보다 낮은 주파수(low-frequency)로 실행되지만, 더 많은 데이터를 처리하여 훨씬 높은 정확도(high fidelity)를 목표로 한다.4 S2M 정합의 성능은 어떤 과거 데이터를 어떻게 조합하여 서브맵을 구성하는지에 따라 크게 달라진다.

이러한 계층적 구조는 마치 단거리 선수가 전력 질주를 하고, 장거리 선수가 페이스를 조절하며 뒤따라가는 것과 같다. S2S는 매 순간의 변화에 빠르게 반응하고, S2M은 전체적인 방향이 틀어지지 않도록 주기적으로 궤도를 수정해주는 역할을 함으로써 시스템 전체의 안정성과 정확성을 보장한다.

### 3.2  키프레이밍과 서브맵 관리

LiDAR가 생성하는 모든 스캔을 맵에 저장하고 정합에 사용하는 것은 엄청난 계산 비용과 메모리를 요구하므로 현실적으로 불가능하다. 따라서 어떤 데이터를 '기억'하고 '활용'할지를 결정하는 효율적인 데이터 관리 전략이 필수적이다.

키프레이밍 (Keyframing):

모든 스캔을 저장하는 대신, 로봇의 움직임에 '중요한' 변화가 있을 때의 스캔만을 선별하여 '키프레임(keyframe)'으로 지정하고 맵 데이터베이스에 저장하는 전략이다. '중요한' 변화의 기준은 보통 로봇이 이전 키프레임으로부터 일정 거리 이상 이동했거나 일정 각도 이상 회전했을 때로 설정한다.4 키프레이밍은 맵의 크기를 효과적으로 관리하고, S2M 정합 시 계산량을 줄이는 핵심적인 역할을 한다.

서브맵 생성 (Submap Generation):

S2M 정합을 수행할 때, 전체 맵이 아닌 현재 로봇의 위치와 관련된 일부 데이터만을 추출하여 '서브맵'을 구성한다. 이 서브맵이 정합의 기준(target)이 된다.

- **전통적 방식:** 가장 일반적인 방법은 현재 로봇의 추정 위치로부터 일정 반경(radius) 내에 있는 모든 맵 포인트들을 검색하여 서브맵을 구성하는 것이다.4 이 방식은 직관적이지만, 맵이 커질수록 반경 내 포인트 수가 급격히 증가하여 검색 및 정합 시간이 길어지는 문제가 있다.
- **DLO의 혁신적 방식:** DLO(Direct LiDAR Odometry)는 이 문제를 해결하기 위해 혁신적인 서브맵 생성 방식을 제안했다. 반경 기반으로 포인트를 검색하는 대신, '키프레임 공간'에서 검색한다. 구체적으로, 현재 위치와 가장 가까운 몇 개의 키프레임과, 이들 키프레임의 집합을 감싸는 볼록 껍질(convex hull)을 구성하는 경계(boundary) 키프레임들을 선택한다. 그 후, 선택된 키프레임들의 포인트 클라우드를 모두 합쳐 하나의 서브맵을 만든다. 이 방식은 로봇 주변의 가까운 정보뿐만 아니라, 복도 끝이나 방의 모서리와 같은 멀리 있지만 구조적으로 중요한 앵커 포인트(anchor point)를 서브맵에 포함시켜 더 안정적이고 정확한 정합을 가능하게 한다.4

### 3.3  사례 연구: DLO (Direct LiDAR Odometry) 아키텍처 분석

DLO는 조밀한 포인트 클라우드를 최소한으로 전처리하여 사용하는 경량 프론트엔드 LiDAR Odometry 시스템으로, Direct 방식의 시스템 아키텍처를 이해하는 데 좋은 사례를 제공한다.4 DLO의 성공은 단순히 좋은 정합 알고리즘을 사용했기 때문이 아니라, 계산 효율성을 극대화하는 영리한 시스템 설계와 데이터 관리 전략에 기인한다.

**DLO의 핵심 구성 요소:**

1. **S2S 정합:** 연속된 두 스캔 간의 상대 변환을 계산하기 위해 GICP를 사용한다. 만약 IMU 데이터가 있다면, 이를 초기 추정값으로 활용하여 GICP의 수렴 속도와 정확도를 높인다.4
2. **S2M 정합:** S2S 단계에서 얻은 상대 변환을 이전 프레임의 궤적에 적용하여 현재 프레임의 초기 포즈를 구한다. 이 초기 포즈를 시작점으로 하여, 현재 스캔을 위에서 설명한 키프레임 기반 서브맵에 GICP로 정합한다. 이 과정을 통해 S2S에서 누적된 드리프트를 보정하고 궤적을 정제한다.4
3. **데이터 구조 재활용:** DLO의 실시간 성능을 가능하게 하는 가장 핵심적인 기술이다. GICP를 사용하려면 대응점 탐색을 위한 k-d tree와 각 포인트의 공분산 계산이 필요한데, 이 과정은 계산 비용이 매우 높다. DLO는 S2S와 S2M 모듈 간에 이러한 데이터 구조를 영리하게 재활용한다. 예를 들어, 현재 프레임의 k-d tree는 한 번만 만들어서 S2S의 소스(source) 트리와 S2M의 소스 트리로 동시에 사용한다. 또한, 이전 프레임의 소스 트리는 현재 프레임의 S2S 타겟(target) 트리가 되므로 새로 계산할 필요가 없다. 이러한 방식으로 중복 계산을 제거하여 전체 시스템의 오버헤드를 극적으로 줄인다.4

이처럼 뛰어난 LiDAR Odometry 시스템을 구축하는 것은 단순히 최고의 정합 알고리즘을 선택하는 문제를 넘어, 제한된 계산 자원 내에서 정보의 흐름을 효율적으로 관리하고 최적화하는 시스템 엔지니어링의 문제임을 DLO의 사례는 명확히 보여준다.

### 3.4  최적화 기법: Gauss-Newton과 Levenberg-Marquardt

Point-to-Plane ICP나 NDT와 같은 Direct Odometry의 핵심 정합 과정은 대부분 비선형 최소제곱(non-linear least squares) 문제로 귀결된다. 이러한 문제를 풀기 위해 반복적인 수치 최적화 기법이 사용되며, Gauss-Newton과 Levenberg-Marquardt가 가장 대표적이다.

- **Gauss-Newton (GN) 알고리즘:** 비선형 비용 함수를 현재 추정치 주변에서 1차 테일러 급수로 근사하여, 매 반복마다 선형 최소제곱 문제를 푸는 방식으로 최적해를 찾아간다. 이 방법은 비용 함수의 2차 미분(헤시안 행렬)을 직접 계산하는 대신, 1차 미분(자코비안 행렬)의 곱으로 근사하기 때문에 계산이 상대적으로 빠르다. 하지만 비용 함수가 심하게 비선형적이거나 초기 추정값이 최적해에서 멀리 떨어져 있으면 수렴이 보장되지 않고 발산할 위험이 있다.33
- **Levenberg-Marquardt (LM) 알고리즘:** Gauss-Newton 방법의 불안정성을 개선하기 위해 제안된 하이브리드 방식이다. LM은 Gauss-Newton과 가장 가파른 경사 하강법(Steepest Gradient Descent)을 지능적으로 결합한다. 최적해에서 멀리 떨어져 있을 때는 안정적으로 수렴하는 경사 하강법처럼 동작하고, 최적해에 가까워지면 빠르게 수렴하는 Gauss-Newton처럼 동작한다. 이 전환은 감쇠 인자(damping factor) $\lambda$에 의해 조절된다. $\lambda$가 크면 경사 하강법에 가까워지고, 작으면 Gauss-Newton에 가까워진다. LM은 GN보다 강인하여 다양한 비선형 최적화 문제에서 표준적인 해법으로 널리 사용된다.20

LiDAR Odometry에서는 로봇의 움직임이 예측 불가능하고 큰 오차가 발생할 수 있으므로, 안정적인 수렴이 중요한 경우가 많아 LM 알고리즘이 선호되는 경향이 있다. g2o, Ceres Solver와 같은 SLAM 백엔드 라이브러리들은 이러한 최적화 기법들을 효율적으로 구현하여 제공한다.33

------

## 4.  센서 융합: 강인성 확보를 위한 LiDAR-Inertial Odometry (LIO)

LiDAR 단독으로 Odometry를 수행하는 것은 여러 한계를 가진다. 특히 빠른 움직임이나 특징이 부족한 환경에서는 성능이 급격히 저하될 수 있다. 이러한 한계를 극복하고 시스템의 강인성(robustness)과 정확도를 높이기 위해 관성 측정 장치(Inertial Measurement Unit, IMU)와의 융합은 이제 선택이 아닌 필수가 되었다.

### 4.1  융합의 필요성

LiDAR와 IMU는 상호 보완적인 특성을 가진다. LiDAR는 외부 환경을 측정하여 전역적으로 드리프트가 적은 위치 정보를 제공하지만, 측정 주파수가 낮고(예: 10-20Hz) 특징 없는 환경에 취약하다. 반면 IMU는 외부 환경과 무관하게 자신의 가속도와 각속도를 고주파(예: 100-1000Hz)로 측정하여 단기적으로 매우 정밀한 움직임 정보를 제공하지만, 시간이 지남에 따라 오차가 빠르게 누적(drift)된다. 이 둘을 융합하면 각 센서의 단점을 보완하고 장점을 극대화할 수 있다.

- **모션 왜곡(Motion Distortion) 보정:** LiDAR 센서가 한 바퀴를 스캔하는 동안(예: 100ms) 로봇이 움직이면, 스캔 시작 시점과 종료 시점의 로봇 위치가 달라 포인트 클라우드가 휘어지는 왜곡이 발생한다. 고주파로 움직임을 측정하는 IMU 데이터를 이용하면, 각 포인트가 측정된 정확한 시점의 센서 위치를 보간(interpolation)하여 이 왜곡을 효과적으로 보정(de-skewing)할 수 있다. 이는 정확한 맵핑과 정합의 전제 조건이다.2
- **빠른 움직임 및 특징 없는 환경 대응:** LiDAR Odometry는 빠른 회전이나 특징이 부족한 환경(예: 긴 복도)에서 정합에 실패하기 쉽다. IMU는 이러한 상황에서도 안정적인 모션 예측값을 제공할 수 있다. 이 예측값을 LiDAR 정합 알고리즘의 초기 추정값(initial guess)으로 사용하면, 최적화의 수렴 범위가 넓어지고 속도가 빨라져 정합의 성공률을 크게 향상시킬 수 있다.4

### 4.2  결합 방식: Tightly-Coupled vs. Loosely-Coupled

LiDAR와 IMU를 융합하는 방식은 크게 두 가지로 나뉜다.

- **Loosely-Coupled (느슨한 결합):** LiDAR Odometry와 IMU Odometry가 각각 독립적으로 자신의 상태(위치, 자세)를 추정한다. 그 후, 이 두 개의 독립적인 상태 추정 결과를 칼만 필터와 같은 별도의 후처리 필터를 통해 융합하는 방식이다. 구현이 비교적 간단하다는 장점이 있지만, 한 센서의 원본 측정값이 다른 센서의 내부 상태(예: IMU의 바이어스)를 직접적으로 보정해주지 못하기 때문에 최적의 성능을 내기 어렵다. 한쪽 모듈이 고장 나면 전체 시스템이 불안정해질 수 있다.12
- **Tightly-Coupled (강한 결합):** LiDAR의 원본 측정값(포인트 클라우드)과 IMU의 원본 측정값(가속도, 각속도)을 하나의 통합된 최적화 문제 안에서 동시에 처리하는 방식이다. 즉, LiDAR 측정 오차와 IMU 측정 오차를 모두 포함하는 단일 비용 함수(cost function)를 구성하고, 이를 최소화하여 로봇의 상태(위치, 자세, 속도)와 센서의 내부 파라미터(IMU 바이어스 등)를 함께 추정한다. 구현이 복잡하지만, 센서 간의 상호작용을 모두 고려하므로 Loosely-Coupled 방식보다 훨씬 높은 정확도와 강인성을 제공한다.12 현대의 고성능 LIO 시스템은 대부분 Tightly-Coupled 방식을 채택한다.

### 4.3  필터 기반 접근법: Error-State Kalman Filter (ESKF)

Tightly-Coupled LIO를 구현하는 주요 방법론 중 하나로, 특히 필터 기반 접근법에서 널리 사용된다. FAST-LIO 계열의 알고리즘이 ESKF를 사용하는 대표적인 예시다.43

**작동 원리:** ESKF는 로봇의 전체 상태(Total State)를 직접 추정하는 대신, 그 상태의 '오차(Error State)'를 추정하는 데 초점을 맞춘다. 이는 전체 상태 공간의 비선형성이 크더라도, 오차는 작다고 가정하여 선형 시스템으로 근사할 수 있다는 아이디어에 기반한다.

1. **Prediction (예측):** 가장 최근에 보정된 상태로부터 IMU 측정값을 고주파로 적분하여 로봇의 공칭 상태(nominal state, 즉 오차가 없다고 가정한 예측 상태)를 전파(propagate)한다.
2. **Measurement Update (측정 업데이트):** 저주파로 LiDAR 스캔이 들어오면, 이 스캔을 현재 예측된 공칭 상태를 기준으로 맵에 정합한다. 이때 발생하는 잔차(residual), 즉 예측과 측정의 불일치를 계산한다.
3. **Correction (보정):** 이 잔차를 이용하여 칼만 필터가 '오차 상태'(예: 위치 오차 $\delta p$, 자세 오차 $\delta \theta$, 속도 오차 $\delta v$, 바이어스 오차 $\delta b$)를 추정한다.
4. **State Update (상태 갱신):** 추정된 오차 상태를 공칭 상태에 주입(inject)하여 전체 상태를 보정한다 ($\text{True State} \approx \text{Nominal State} + \text{Error State}$). 그 후, 오차 상태는 다시 0으로 리셋된다.

이러한 접근 방식은 비선형성이 큰 전체 상태를 직접 다루는 표준 확장 칼만 필터(EKF)보다 수치적으로 더 안정적이고 정확한 결과를 낳는다. FAST-LIO2와 같은 최신 시스템에서는 비선형성을 더욱 효과적으로 처리하기 위해 측정 업데이트 단계를 여러 번 반복하는 반복 EKF(Iterated EKF)를 사용하기도 한다.43

LIO 시스템의 발전 과정을 살펴보면, IMU의 역할이 단순히 LiDAR를 보조하는 수준을 넘어 시스템의 중추로 변모하고 있음을 알 수 있다. 초기 LIO에서 IMU는 주로 모션 왜곡 보정이라는 보조적인 역할에 머물렀다. 그러나 FAST-LIO와 같은 ESKF 기반 시스템에서 IMU는 상태 전파의 중심, 즉 시스템의 동역학 모델 그 자체가 된다. LiDAR 측정값은 이 IMU 기반의 예측을 주기적으로 '보정'해주는 역할을 수행한다. 더 나아가 Super Odometry 12나 DLIO 46와 같은 최신 프레임워크는 'IMU-centric' 설계를 명시적으로 채택하여, IMU를 기본 상태 추정기로 삼고 다른 센서(LiDAR, 카메라)가 IMU의 고질적인 문제인 바이어스 드리프트를 억제하는 구조를 취한다. 이는 "LiDAR에 IMU를 더한다"는 관점에서 "IMU를 LiDAR로 보정한다"는 관점으로의 패러다임 전환을 의미하며, 이를 통해 시스템은 단기적으로는 IMU의 부드러운 궤적을, 장기적으로는 LiDAR의 전역적 정확성을 모두 확보하게 된다.

또한, LIO의 발전은 전통적인 SLAM 커뮤니티의 양대 산맥이었던 '필터링'과 '최적화'의 경계를 허물고 있다. LIO-SAM 40과 같은 시스템은 최적화 기반 접근법(Factor Graph)을, FAST-LIO 43는 필터 기반 접근법(Iterated EKF)을 사용한다. 하지만 본질적으로 Iterated EKF는 현재 시간 단계에 대한 작은 최적화 문제를 푸는 것과 같으며, 증분적 평활화 및 매핑(Incremental Smoothing and Mapping, iSAM)과 같은 최신 최적화 기법은 필터와 유사한 실시간성을 보인다.41 결국 두 접근법은 동일한 최대 사후 확률(Maximum a Posteriori, MAP) 추정 문제를 다른 계산 방식으로 푸는 것이며, 실시간 성능과 정확성을 모두 잡기 위해 서로의 장점을 흡수하며 융합되고 있다. 따라서 "필터냐 최적화냐"의 이분법적 질문은 더 이상 유효하지 않으며, "어떻게 효율적으로 증분적 최적화를 수행할 것인가"가 LIO 시스템 설계의 핵심 질문이 되고 있다.

------

## 5.  최신 연구 동향 및 첨단 기법

Direct LiDAR Odometry 분야는 전통적인 기하학 기반의 정합 알고리즘을 넘어, 딥러닝, 시맨틱 정보, 장기적 자율성을 위한 새로운 패러다임을 적극적으로 도입하며 빠르게 발전하고 있다.

### 5.1  딥러닝 기반 접근법

전통적인 LiDAR Odometry 파이프라인(특징 추출, 매칭, 최적화)을 데이터 기반의 종단간(End-to-End) 학습 모델로 대체하려는 시도가 활발히 이루어지고 있다.18 이는 기하학적 원리에만 의존하는 대신, 대규모 데이터로부터 움직임 추정에 유용한 패턴을 직접 학습하려는 접근법이다.

**작동 방식:**

1. **입력 표현 변환:** 포인트 클라우드는 희소하고 불규칙한 데이터 구조를 가져 심층 신경망(DNN)에 직접 입력하기 어렵다. 따라서 이를 규칙적인 격자 형태로 변환하는 과정이 선행된다. 대표적으로 포인트 클라우드를 원통형으로 투영하여 2D 이미지처럼 만드는 'Range Image'나, 위에서 내려다본 시점의 2D 격자로 변환하는 'Bird's-Eye-View (BEV)' 표현이 널리 사용된다.19
2. **네트워크 아키텍처:** 변환된 입력 표현을 사용하여, 합성곱 신경망(CNN)이나 순환 신경망(RNN) 기반의 네트워크가 연속된 두 프레임 간의 공간적, 시간적 특징을 추출한다. 네트워크의 최종 출력은 두 프레임 사이의 상대적인 6-DoF 변환(이동 및 회전)을 직접 회귀(regress)하는 형태가 된다.18
3. **학습:** 대규모 주행 데이터셋(예: KITTI)의 LiDAR 스캔 시퀀스와 그에 해당하는 실제 궤적(Ground Truth)을 이용하여 지도 학습(supervised learning) 방식으로 네트워크를 훈련시킨다.

**대표 아키텍처:**

- **LO-Net:** 연속적인 LiDAR 스캔으로부터 동적 객체를 구분하는 마스크를 예측하고, 이를 제외한 정적 포인트들을 이용하여 두 프레임 간의 상대 변환을 추정하는 CNN 기반 아키텍처다. 전통적인 강자인 LOAM과 경쟁할 만한 성능을 보여주었다.19
- **DeepPCO:** 두 포인트 클라우드 사이의 대응 관계 자체를 학습하고, 이를 기반으로 최적의 변환을 추정하는 병렬 신경망 구조를 제안했다.19

딥러닝 모델은 데이터로부터 복잡하고 미묘한 특징을 자동으로 학습하므로, 사람이 직접 모델링하기 어려운 환경(예: 악천후, 센서 노이즈 패턴)이나 동적 객체에 대해 더 강인한 성능을 보일 잠재력이 있다. 하지만 대규모의 정답 레이블이 있는 데이터셋이 필요하다는 점("data hungry"), 그리고 학습 데이터에 포함되지 않은 새로운 환경에 대한 일반화 성능이 떨어질 수 있다는 한계를 가진다.19

### 5.2  Semantic SLAM

Semantic SLAM은 전통적인 SLAM이 다루는 기하학적 정보(점, 선, 면)에 더해 '의미론적 정보(semantic information)'를 SLAM 파이프라인에 적극적으로 통합하는 패러다임이다.50 여기서 의미론적 정보란 '자동차', '사람', '건물', '나무'와 같이 객체의 종류나 속성을 의미한다. 이는 딥러닝 기반의 3D 객체 탐지(object detection)나 시맨틱 분할(semantic segmentation) 기술의 비약적인 발전 덕분에 가능해졌다.48

**Semantic SLAM의 이점:**

- **강인한 데이터 연관(Data Association):** 기하학적 구조만으로는 구분이 어려운 장소, 예를 들어 똑같이 생긴 복도가 반복되는 건물에서도 '301호 앞의 소화기'나 '비상구 안내판'과 같은 고유한 시맨틱 랜드마크를 통해 위치를 명확하게 구분할 수 있다. 이는 특히 루프 클로저(loop closure)나 재지역화(relocalization)의 정확도와 신뢰도를 크게 향상시킨다.50
- **지능적인 동적 객체 처리:** 움직이는 객체(예: 자동차, 보행자)를 의미론적으로 '움직이는 것'으로 식별하여 맵 구성에서 제외하거나 별도로 추적할 수 있다. 이는 '세상은 정적이다'라는 SLAM의 기본 가정을 위배하는 동적 객체로 인해 발생하는 정합 오류와 궤적 오차를 근본적으로 해결하는 방법이다.53
- **고수준 맵 생성 및 상황 인식:** 결과물로 생성되는 맵은 단순히 점들의 집합이 아니라, "어디에 무엇이 있는지"를 아는 고차원적인 지식 베이스가 된다. 이는 로봇이 "음료수 캔을 가져와"와 같은 인간의 지시를 이해하고 수행하는 등, 더 높은 수준의 자율적인 작업을 가능하게 하는 기반이 된다.51

**접근 방식 (예: SG-SLAM, SD-SLAM):** 초기 Semantic SLAM은 포인트 레벨의 의미 정보를 사용했지만, 이는 분할 오류에 취약했다. 최신 연구들은 개별 포인트를 넘어 객체 수준에서 '시맨틱 그래프(semantic graph)'를 구성하여 SLAM을 수행한다. 이 그래프에서 노드는 객체를, 엣지는 객체 간의 공간적, 위상적 관계를 나타낸다. 이러한 접근법은 포인트 단위의 분할 오류에 훨씬 강인하며, 환경의 구조를 더 추상적이고 효율적으로 표현할 수 있다.50

이러한 발전은 LiDAR Odometry가 단순히 기하학적 퍼즐 맞추기에서 벗어나, 주변 환경을 '이해'하고 그 이해를 바탕으로 더 강인한 추론을 하는 '상황 인식 기반 추론'으로 진화하고 있음을 보여준다. 기하학은 이제 유일한 정보원이 아니라, 의미론적 맥락 안에서 해석되는 여러 단서 중 하나가 되어가고 있다.

### 5.3  Lifelong SLAM과 맵 유지보수

전통적인 SLAM의 목표는 한 번의 탐사 동안 변하지 않는 환경에 대한 정확한 지도를 만드는 것이었다. 그러나 로봇이 장기간 동일한 환경(예: 집, 사무실, 도시)에서 작동해야 할 때, 환경은 가구 배치 변경, 계절에 따른 식생 변화, 공사 등으로 인해 끊임없이 변한다. 한 번 생성된 정적인 맵은 시간이 지나면 현실과 달라져 무용지물이 된다. Lifelong SLAM (또는 Long-term SLAM)은 이러한 동적인 변화에 지속적으로 적응하며 맵을 자율적으로 업데이트하고 유지하는 것을 목표로 한다.54

**Lifelong SLAM의 핵심 과제:**

1. **변화 감지(Change Detection):** 현재 센서로 관측한 정보와 기존 맵 사이의 불일치를 감지하는 것.
2. **맵 업데이트(Map Update):** 감지된 변화를 맵에 어떻게 반영할지 결정하는 것. 특히, 일시적인 변화(ephemeral, 예: 주차된 차, 움직이는 사람)와 영구적인 변화(persistent, 예: 새로 지어진 건물, 제거된 나무)를 구분하는 것이 중요하다.
3. **확장성(Scalability):** 시간이 지남에 따라 맵 데이터가 무한정 커지는 것을 방지하고, 과거와 현재의 정보를 효율적으로 저장하고 관리하는 것.

**접근 방식:**

- **PoseMap:** 전체 맵을 하나의 거대한 데이터 덩어리로 관리하는 대신, 각 위치에서 관찰된 특징점 묶음을 독립적인 '맵 노드(map node)'로 표현한다. 환경 변화가 감지되면 해당 지역의 맵 노드만 새로운 관측으로 교체하는 방식을 통해 효율적인 업데이트를 수행한다. 이는 맵 전체를 재구성할 필요 없이 국소적인 변화에 대응할 수 있게 한다.57
- **ELite & LT-mapper:** 여러 세션(session, 즉 다른 시간에 수집된 데이터 묶음) 간의 변화를 감지하고 관리하는 모듈식 프레임워크를 제안한다. 특히 '일시성(ephemerality)'이라는 새로운 개념을 도입하여, 맵의 각 포인트나 객체가 얼마나 오랫동안 지속되는지를 확률적으로 모델링한다. 이 '일시성' 점수를 바탕으로, 정말로 영구적인 변화만을 선별하여 맵에 반영하고, 일시적인 노이즈는 필터링함으로써 맵의 신뢰성을 높인다.54

Lifelong SLAM의 등장은 SLAM의 패러다임에 중요한 전환을 가져왔다. 전통적인 SLAM의 목표가 '모든 정보를 축적하여 완벽한 단일 맵을 만드는 것'이었다면, Lifelong SLAM은 '과거의 모든 정보가 항상 유효하지는 않다'는 것을 인정하는 데서 출발한다. 따라서 문제는 '어떻게 모든 것을 저장할까'가 아니라 '무엇을 버리고 무엇을 유지할까'로 바뀐다. 이는 마치 인간의 기억처럼, 중요하고 반복적인 정보는 장기 기억으로 남기고 일시적이고 사소한 정보는 잊어버리는 '망각(forgetting)' 또는 '선별적 기억(selective memory)' 메커니즘을 SLAM 시스템에 도입하는 것과 같다. 미래의 Lifelong SLAM 시스템은 단순히 데이터를 쌓는 저장소가 아니라, 정보의 가치를 끊임없이 평가하고 재구성하는 동적인 지식 기반 시스템으로 발전할 것이다.

------

## 6.  실제적 고려사항 및 성능 분석

이론적인 알고리즘의 우수성이 실제 환경에서의 성공을 보장하지는 않는다. Direct LiDAR Odometry를 실제 로봇에 적용하기 위해서는 다양한 현실적인 문제들을 고려해야 한다.

### 6.1  도전적인 환경에서의 성능

LiDAR Odometry의 진정한 성능은 이상적인 조건이 아닌, 가혹하고 예측 불가능한 '도전적인 환경'에서 드러난다.

- **특징이 부족한 환경 (Degenerative Environments):** 길고 직선적인 터널, 특징 없는 복도, 넓게 펼쳐진 벌판과 같은 환경은 기하학적 제약 조건이 극도로 부족하다. 이러한 환경에서 LiDAR Odometry는 특정 축(예: 터널 진행 방향의 이동 거리, 평면 위에서의 회전)에 대한 움직임을 추정할 수 없어 궤적이 해당 축으로 무한정 발산(drift)하는 '퇴화(degeneration)' 문제에 직면한다. 이는 정합 알고리즘이 유일한 해를 찾지 못하기 때문에 발생하며, IMU와의 강한 결합(tightly-coupled fusion)이 이러한 문제를 완화하는 데 필수적인 해결책으로 여겨진다.2
- **동적 환경 (Dynamic Environments):** 도심지와 같이 차량, 보행자 등 움직이는 객체가 많은 환경은 SLAM의 근본적인 가정인 '정적인 세계(static world)'를 위배한다. 시스템이 동적 객체를 정적인 배경의 일부로 잘못 인식하면, 이는 정합 과정에서 심각한 오류를 유발하여 궤적 추정의 정확도를 크게 떨어뜨린다. 동적 객체를 의미론적으로 식별하여 제거하는 모듈이나, 다수의 이상치에 강인한 정합 알고리즘이 요구된다.53
- **열악한 감지 환경 (Degraded Environments):** 비, 눈, 안개, 먼지와 같은 악천후 조건은 LiDAR가 발사한 레이저 빔을 중간에 산란시키거나 흡수한다. 이는 포인트 클라우드에 심한 노이즈를 유발하거나, 측정 거리와 반사 강도를 왜곡하여 데이터의 품질을 근본적으로 저하시킨다. 이러한 '오염된' 데이터는 정합 성능을 크게 떨어뜨리는 주된 요인이 된다. 이 문제를 해결하기 위해, 악천후에 더 강인한 레이더(Radar)와 같은 다른 센서와의 융합이 유망한 대안으로 연구되고 있다.2

이처럼, 알고리즘의 진정한 강인성은 벤치마크 점수가 아니라, 다양한 실패 시나리오를 인지하고 이에 대응할 수 있는 방어 메커니즘을 갖추었는지에 따라 평가되어야 한다.

### 6.2  하드웨어의 영향: Solid-State vs. Mechanical LiDAR

LiDAR Odometry의 성능은 알고리즘뿐만 아니라 사용하는 LiDAR 하드웨어의 특성에도 깊이 의존한다. 최근 기계식 LiDAR와 고체(Solid-State) LiDAR의 경쟁은 이 분야의 중요한 화두다.

- **기계식 LiDAR (Mechanical LiDAR):** 센서 내부의 레이저 모듈이 물리적으로 360도 회전하며 주변 환경을 스캔한다. 넓은 수평 시야각(FOV)과 규칙적이고 균일한 스캔 라인을 생성하여 SLAM 알고리즘이 특징을 추출하고 데이터를 처리하기에 용이하다. 하지만 회전 부품으로 인해 크기가 크고, 가격이 비싸며, 외부 충격이나 진동에 약해 내구성에 한계가 있다.61
- **고체 LiDAR (Solid-State LiDAR, SSL):** MEMS 미러나 광학 위상 배열(Optical Phased Array, OPA)과 같은 기술을 사용하여 기계적 움직임 없이 레이저 빔을 전자적으로 조향한다. 작고 가벼우며 가격이 저렴하고 내구성이 뛰어나 양산 차량이나 소형 로봇에 적합하다. 그러나 일반적으로 시야각이 제한적이고, 스캔 패턴이 불규칙하며(예: 꽃 모양, 나선형), 포인트 분포가 비균일하다는 단점이 있다.62

**Odometry에 미치는 영향:** SSL의 제한된 FOV와 불규칙한 스캔 패턴은 기존의 Odometry 알고리즘에 새로운 도전을 제기한다. 예를 들어, LOAM과 같이 수평 스캔 라인의 규칙성을 가정하는 특징 추출 방식은 SSL에 직접 적용하기 어렵다. 또한, 좁은 FOV는 관측되는 특징의 수를 줄여 정합의 안정성을 떨어뜨릴 수 있다. 따라서 SSL의 잠재력을 최대한 활용하기 위해서는 그 특유의 스캔 패턴을 고려한 전용 전처리 기법이나 새로운 Odometry 알고리즘의 개발이 필수적이다.62

이처럼 하드웨어의 발전은 알고리즘이 풀 수 있는 문제의 크기와 형태를 결정하고, 알고리즘의 진화는 새로운 하드웨어의 잠재력을 이끌어내는 방식으로 함께 공진화(co-evolution)하고 있다.

### 6.3  벤치마크 및 성능 평가

개발된 LiDAR Odometry 알고리즘의 성능을 객관적으로 평가하고 비교하기 위해 표준화된 벤치마크 데이터셋이 사용된다.

- **KITTI 데이터셋:** 자율주행 연구를 위해 독일 카를스루에 공과대학(KIT)과 Toyota Technological Institute at Chicago(TTIC)가 공동으로 구축한 대표적인 벤치마크다. 실제 도시, 시골, 고속도로 환경에서 수집된 LiDAR, 스테레오 카메라, IMU 데이터와 함께, 고정밀 GPS-RTK 시스템으로 측정한 실제 차량 궤적(Ground Truth)을 제공한다. 이를 통해 다양한 Odometry 알고리즘의 성능을 정량적으로 비교할 수 있어, 이 분야의 사실상 표준(de facto standard)으로 자리 잡았다.11
- **평가 지표:**
  - **Absolute Trajectory Error (ATE):** 추정된 전체 궤적과 실제 궤적을 정렬한 후, 각 지점에서의 오차를 계산하여 전체적인 정확도를 평가하는 지표다. 시스템의 전역적인 성능을 나타낸다.66
  - **Relative Pose Error (RPE):** 일정 시간 또는 거리 간격 동안의 상대적인 움직임(이동 및 회전)의 오차를 측정한다. 이는 Odometry가 얼마나 빠르게 오차를 누적하는지, 즉 드리프트의 정도를 평가하는 지표다.66
- **주요 알고리즘 성능:** KITTI Odometry 벤치마크 리더보드를 보면, LOAM과 그 변종들(V-LOAM, LeGO-LOAM), CT-ICP, KISS-ICP와 같은 알고리즘들이 상위권을 차지하고 있다. 이들 상위권 알고리즘은 일반적으로 평균 번역 오차 0.5-1.0%, 평균 회전 오차 0.001-0.002 deg/m 수준의 매우 높은 정확도를 보여준다.67 아래 표는 KITTI 벤치마크에서 우수한 성능을 보인 일부 LiDAR 기반 Odometry 알고리즘들의 성능을 요약한 것이다.

**Table 3: KITTI 벤치마크 상위 LiDAR Odometry 알고리즘 성능**

| 순위 (Rank) | 알고리즘 이름 (Algorithm Name) | 번역 오차 (Translational Error %) | 회전 오차 (Rotational Error deg/m) | 기반 기술/특징 (Core Technology/Features)                |
| ----------- | ------------------------------ | --------------------------------- | ---------------------------------- | -------------------------------------------------------- |
| 2           | V-LOAM                         | 0.54                              | 0.0013                             | Visual-LiDAR Odometry, Feature-based                     |
| 3           | LOAM                           | 0.55                              | 0.0013                             | LiDAR-only, Feature-based (Edge/Plane)                   |
| 11          | CT-ICP                         | 0.59                              | 0.0014                             | Continuous-Time Trajectory, Point-to-Point ICP           |
| 15          | KISS-ICP                       | 0.61                              | 0.0017                             | Point-to-Point ICP, Adaptive Thresholding                |
| 17          | MULLS                          | 0.65                              | 0.0019                             | Multi-metric Linear SLAM, Feature-based                  |
| 23          | IMLS-SLAM                      | 0.69                              | 0.0018                             | Implicit Moving Least Squares, Point-to-Implicit Surface |
| 32          | SuMa++                         | 1.06                              | 0.0034                             | Surfel-based Mapping, Semantic ICP                       |

주: 순위와 성능은 2024년 9월 기준 KITTI Odometry Benchmark 67에 따르며, 변경될 수 있음.

### 6.4  계산 가속화 기법

Direct Odometry의 가장 큰 실용적 장벽은 방대한 포인트 클라우드를 실시간으로 처리하는 데 따르는 계산 비용이다. 특히, 수십만 개의 포인트에 대해 최근접 이웃 탐색(Nearest Neighbor Search)을 반복적으로 수행하는 것은 엄청난 병목 구간이다.4 이를 해결하기 위한 다양한 가속화 기법이 연구되고 있다.

- **GPU 가속화:** ICP, GICP, NDT와 같은 정합 알고리즘의 핵심 연산, 즉 거리 계산, 행렬 연산 등은 본질적으로 대규모 병렬 처리에 매우 적합하다. CUDA와 같은 GPU 프로그래밍 모델을 활용하면 이러한 연산들을 수천 개의 코어에서 동시에 처리하여 CPU 대비 수십 배에서 수백 배의 속도 향상을 얻을 수 있다. `fast_gicp` 라이브러리의 `FastGICPCuda`나 `NDTCuda`와 같은 구현체는 이러한 GPU 가속화를 통해 실시간 성능을 크게 향상시킨 대표적인 예다.30
- **복셀화 (Voxelization):** 공간을 복셀 그리드로 나누는 것은 NDT뿐만 아니라 ICP 계열 알고리즘의 속도를 높이는 데도 널리 사용된다. 복셀 내의 포인트들을 하나의 평균점이나 중심점으로 다운샘플링하여 처리할 데이터의 양을 줄이거나, 최근접 이웃 탐색 범위를 전체 공간이 아닌 현재 복셀과 그 주변 복셀로 제한함으로써 계산 복잡도를 크게 낮출 수 있다.30

------

## 7.  안전성, 신뢰성 및 미래 전망

LiDAR Odometry 기술이 연구실을 넘어 자율주행차, 배송 로봇 등 우리의 일상과 밀접한 시스템에 적용되기 시작하면서, 알고리즘의 정확도를 넘어 안전성(safety)과 신뢰성(reliability)이 가장 중요한 화두로 떠오르고 있다.

### 7.1  자율주행을 위한 안전성 및 신뢰성

자율주행 시스템에서 '내가 어디에 있는가'를 아는 위치 추정은 모든 판단과 제어의 시작점이다. 따라서 Odometry의 작은 오차나 일시적인 실패는 치명적인 사고로 직결될 수 있다.69

- **센서 자체의 안전성 및 보안:** LiDAR Odometry의 신뢰성은 하드웨어 자체의 안전성에서 시작된다. 자율주행 차량에 사용되는 LiDAR는 고출력 레이저를 사용하므로, 어떤 거리에서도 인체, 특히 눈에 해를 끼치지 않는 Class 1 등급의 눈 안전(eye-safe) 설계가 필수적이다. 또한, 도시 환경에서 여러 대의 LiDAR가 동시에 사용될 때 레이저 빔의 상호 간섭이나 잠재적 위험에 대한 심층적인 연구가 필요하다.71 더 나아가, LiDAR 시스템은 외부에서 의도적으로 조작된 레이저 신호를 보내 시스템을 속이는 스푸핑(spoofing)이나, 특정 패턴의 반사체를 설치하여 객체 인식을 방해하는 물리적 공격 등 적대적 공격(adversarial attacks)에 취약할 수 있다. 이는 시스템의 신뢰성을 근본적으로 위협하는 심각한 보안 문제다.69

### 7.2  고장 탐지 및 복구

아무리 뛰어난 알고리즘이라도 실패할 가능성은 항상 존재한다. 중요한 것은 실패하지 않는 시스템이 아니라, 실패를 스스로 인지하고 안전하게 대처할 수 있는 시스템을 만드는 것이다.

- **고장 원인:** 빠른 카메라 움직임으로 인한 모션 블러, 특징 없는 환경 진입, 터널 등에서 센서가 가려지는 현상 등으로 인해 SLAM 시스템은 현재 위치를 잃어버리거나(tracking loss), 궤적에 심각한 드리프트가 누적될 수 있다.72
- **탐지 메커니즘:** 시스템의 '건강 상태(health)'를 지속적으로 모니터링하여 고장 가능성을 미리 예측하는 것이 중요하다. 예를 들어, 매 프레임 추적되는 특징점의 수, 새로운 키프레임이 생성되는 빈도, 추정된 상태의 불확실성(공분산 행렬의 크기) 등을 지표로 사용하여 시스템의 상태가 나빠지고 있음을 감지할 수 있다.74
- **복구 전략:** 고장이 감지되었을 때 시스템을 정상 상태로 되돌리기 위한 전략이다.
  1. **재지역화 (Relocalization):** 추적을 완전히 잃었을 때, 이전에 방문했던 장소의 시각적 또는 기하학적 특징(예: Bag of Visual Words, Scan Context)을 데이터베이스에서 검색하여 현재 위치를 다시 찾는 방법이다.73
  2. **안전한 경로 계획:** SLAM이 실패할 가능성이 높은 지역(예: 특징이 없는 긴 복도)으로의 진입을 미리 예측하고, 이를 회피하는 안전한 경로를 계획하는 능동적인 접근법도 연구되고 있다. 강화학습을 통해 실패를 유발하는 움직임 패턴을 학습하고 이를 피하도록 정책을 만드는 방식이다.76
  3. **다중 세션 관리:** 고장이 발생하면 현재의 SLAM 세션을 종료하고, 마지막으로 성공했던 지점에서 새로운 세션을 시작한다. 이후 로봇이 이전에 방문했던 장소로 돌아와 두 세션 간의 중첩 영역이 발견되면, 두 세션을 정합하여 하나의 일관된 맵으로 병합하는 방식이다.74

### 7.3  무결성 모니터링

무결성 모니터링(Integrity Monitoring)은 안전 필수(safety-critical) 시스템에서 위치 정보의 '신뢰도(trustworthiness)'를 정량적으로 평가하고 보증하는 개념이다. 이는 단순히 "현재 위치 오차는 X미터입니다"라고 보고하는 것을 넘어, "99.999%의 확률로 실제 오차는 Y미터 이내에 있을 것을 보증합니다"라고 말하는 것과 같다.77

- **주요 용어:**
  - **Protection Level (PL):** 특정 신뢰 수준(confidence level)에서 실제 위치 오차(Positioning Error, PE)가 존재할 것으로 예상되는 통계적 상한값이다. 즉, 시스템이 스스로 평가한 오차의 경계선이다.
  - **Alert Limit (AL):** 해당 애플리케이션(예: 고속도로 차선 유지)에서 허용되는 최대 위치 오차다. 이 값을 넘어서면 위험한 상황이 발생할 수 있다.
  - **Integrity Risk:** 실제 위치 오차가 Alert Limit을 초과했음에도 불구하고, 시스템이 이를 인지하지 못하고 경고를 발생시키지 못할 확률이다. 이 값이 낮을수록 시스템은 더 안전하다.77
- **작동 방식:** 시스템은 실시간으로 자신의 위치 오차(PE)와 보호 수준(PL)을 계산한다. 만약 PL이 사전에 정의된 AL을 초과하면, 시스템은 자신의 위치 정보가 더 이상 안전 요구사항을 만족하지 못함을 인지하고 상위 제어 시스템에 경고를 보낸다. 'Fail-aware' 시스템 80은 바로 이러한 무결성 모니터링 개념을 구현한 것으로, 시스템의 고장을 미리 인지하고 자율적으로 안전한 정지(safe stop)와 같은 비상 조치를 취할 수 있게 한다.

LiDAR Odometry의 미래는 '얼마나 정확한가'를 넘어 '얼마나 스스로를 불신할 수 있는가'에 달려 있다. 자율주행 시스템은 완벽한 Odometry를 가정하는 대신, 불완전하고 불확실한 정보를 바탕으로 안전한 결정을 내리는 방식으로 설계될 것이며, 이 과정에서 스스로의 불확실성을 정직하게 보고하는 능력이 핵심적인 역할을 할 것이다.

### 7.4  미해결 과제 및 향후 연구 방향

Direct LiDAR Odometry는 상당한 발전을 이루었지만, 완전한 자율성을 달성하기 위해서는 여전히 많은 도전 과제가 남아있다. 이 과제들은 개별 기술의 문제가 아닌, 여러 기술과 개념을 어떻게 '통합'하고 '일반화'할 것인가의 문제로 수렴하고 있다.

- **일반화 및 강인성:** 특정 센서나 특정 환경에 고도로 튜닝된 알고리즘을 넘어, 다양한 종류의 LiDAR(기계식, 고체)와 이기종 센서(카메라, 레이더), 그리고 예측 불가능한 여러 환경(악천후, 퇴화 환경, 동적 환경)에서 모두 안정적으로 작동하는 일반화된 Odometry 프레임워크의 개발이 시급하다. 이는 '하나의 알고리즘으로 모든 상황에 대처하는' 것을 목표로 한다.2
- **고차원적 환경 이해:** 기하학적 정보를 넘어, 환경의 의미론적, 동적, 시간적 변화를 깊이 있게 이해하고 모델링하는 능력을 고도화해야 한다. 이는 Semantic SLAM과 Lifelong SLAM 연구의 자연스러운 연장선상에 있으며, 로봇이 단순히 공간을 측정하는 것을 넘어 환경과 상호작용하기 위한 필수적인 능력이다.50
- **효율적인 다중 센서 융합:** LiDAR, 카메라, 레이더, IMU, GPS 등 다양한 센서의 장점을 극대화하고 단점을 상호 보완하는 효율적이고 강인한 Tightly-coupled 융합 기법에 대한 연구가 계속되어야 한다. 특히, 이종 센서 간의 자동 보정(online calibration) 및 시간 동기화(synchronization) 문제를 실시간으로 해결하는 기술은 시스템의 실용성을 좌우하는 핵심 과제다.2
- **딥러닝의 신뢰성 확보:** 데이터 기반 딥러닝 모델은 강력한 성능을 보이지만 '블랙박스'처럼 작동하여 그 결정을 신뢰하기 어려운 경우가 많다. 자율주행과 같은 안전 필수 시스템에 적용하기 위해서는, 모델의 예측 불확실성을 정량화하고, 학습 데이터에 없는 상황에 대한 일반화 성능을 보장하며, 그 판단 근거를 사람이 이해할 수 있도록 '설명'하는(Explainable AI, XAI) 연구를 통해 신뢰성을 확보해야 한다.82
- **경량화 및 엣지 컴퓨팅:** 고가의 워크스테이션뿐만 아니라, 드론, 소형 로봇, 양산차 등 계산 자원이 제한된 엣지 디바이스에서도 고성능 Odometry를 실시간으로 수행할 수 있도록 알고리즘을 최적화하고 경량화하는 연구가 지속적으로 필요하다.82

결론적으로, LiDAR SLAM 분야의 연구는 개별 부품을 정교하게 만드는 단계를 지나, 이 부품들을 모아 어떤 상황에서도 안정적으로 작동하는 강인한 '시스템'을 구축하는 단계로 나아가고 있다. 따라서 미래의 핵심 역량은 특정 알고리즘에 대한 깊은 이해를 넘어, 다양한 기술을 조화롭게 엮어내는 시스템 엔지니어링 능력과 아키텍처 설계 능력이 될 것이다.

#### **참고 자료**

1. DV-LOAM: Direct Visual LiDAR Odometry and Mapping - MDPI, accessed July 30, 2025, https://www.mdpi.com/2072-4292/13/16/3340
2. LiDAR Odometry Survey: Recent Advancements and Remaining ..., accessed July 30, 2025, https://arxiv.org/pdf/2312.17487
3. 3D LiDAR Point Cloud Registration Based on IMU Preintegration in Coal Mine Roadways, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10098805/
4. Direct LiDAR Odometry: Fast Localization with Dense ... - LEMUR, accessed July 30, 2025, https://uclalemur.com/publications/Chen2022_DLO.pdf
5. The difference between direct dense method and feature-based sparse... - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/figure/The-difference-between-direct-dense-method-and-feature-based-sparse-method-Feature-based_fig4_224252357
6. What is the main difference between direct and indirect approaches of SLAM In terms of processing and representation and what is the best way to fuse? - Quora, accessed July 30, 2025, https://www.quora.com/What-is-the-main-difference-between-direct-and-indirect-approaches-of-SLAM-In-terms-of-processing-and-representation-and-what-is-the-best-way-to-fuse
7. learnopencv.com, accessed July 30, 2025, [https://learnopencv.com/lidar-slam-with-ros2/#:~:text=Feature%20Based%3A%20Extracts%20distinct%20geometric,orientation%20and%20translation%20of%20Lidar.](https://learnopencv.com/lidar-slam-with-ros2/#:~:text=Feature Based%3A Extracts distinct geometric,orientation and translation of Lidar.)
8. Localization & Odometry - RoboPI, accessed July 30, 2025, https://robopi.ece.ufl.edu/files/AuRu_Materials/Lecture_7.pdf
9. Introduction to LiDAR SLAM: LOAM and LeGO-LOAM Paper and Code Explanation with ROS 2 Implementation - LearnOpenCV, accessed July 30, 2025, https://learnopencv.com/lidar-slam-with-ros2/
10. LOAM: Lidar Odometry and Mapping in Real-time - Carnegie Mellon University's Robotics Institute, accessed July 30, 2025, https://www.ri.cmu.edu/pub_files/2014/7/Ji_LidarMapping_RSS2014_v8.pdf
11. LiDAR Odometry and Mapping Based on Neighborhood Information Constraints for Rugged Terrain - MDPI, accessed July 30, 2025, https://www.mdpi.com/2072-4292/14/20/5229
12. Super Odometry: A Robust LiDAR-Visual-Inertial Estimator for Challenging Environments - Carnegie Mellon University's Robotics Institute, accessed July 30, 2025, https://www.ri.cmu.edu/app/uploads/2024/07/CMU_MSR_thesis_shibo.pdf
13. (PDF) Visual Odometry in Challenging Environments: An Urban Underground Railway Scenario Case - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/361660415_Visual_Odometry_in_challenging_environments_an_urban_underground_railway_scenario_case
14. LiTAMIN: LiDAR Based Tracking and MappINg by Stabilized ICP for Geometry Approximation with Normal Distributions, accessed July 30, 2025, https://unit.aist.go.jp/digiarc/smrt/papers/published/LiTAMIN_IROS2020.pdf
15. Understanding Iterative Closest Point (ICP) Algorithm with Code - LearnOpenCV, accessed July 30, 2025, https://learnopencv.com/iterative-closest-point-icp-explained/
16. Evaluation of 3D Registration Reliability and Speed–A Comparison ..., accessed July 30, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=0185f6fb7a7a7859daf8ae326b485e9891a7461d
17. 3D LiDAR SLAM – Scan Matching Explained - Ignitarium, accessed July 30, 2025, https://ignitarium.com/3d-lidar-slam-scan-matching-explained/
18. [2109.06120] LiDAR Odometry Methodologies for Autonomous Driving: A Survey - arXiv, accessed July 30, 2025, https://arxiv.org/abs/2109.06120
19. [2006.12567] A Survey on Deep Learning for Localization and ..., accessed July 30, 2025, https://ar5iv.labs.arxiv.org/html/2006.12567
20. Comparative Analysis of 3D LiDAR Scan-Matching Methods for State Estimation of Autonomous Surface Vessel - MDPI, accessed July 30, 2025, https://www.mdpi.com/2077-1312/11/4/840
21. Iterative Closest Point | John Lambert, accessed July 30, 2025, https://johnwlambert.github.io/icp/
22. Comparing ICP variants on real- world data sets - Research Collection, accessed July 30, 2025, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/65455/eth-8535-01.pdf
23. Linear Least-Squares Optimization for Point-to-Plane ICP Surface Registration ∑ - NUS Computing, accessed July 30, 2025, https://www.comp.nus.edu.sg/~lowkl/publications/lowk_point-to-plane_icp_techrep.pdf
24. MAD-ICP: It Is All About Matching Data – Robust and Informed LiDAR Odometry - arXiv, accessed July 30, 2025, https://arxiv.org/html/2405.05828v1
25. 1 Derivation of Point-to-Plane Minimization - cs.Princeton, accessed July 30, 2025, https://www.cs.princeton.edu/~smr/papers/icpstability.pdf
26. (PDF) Linear Least-Squares Optimization for Point-to-Plane ICP Surface Registration, accessed July 30, 2025, https://www.researchgate.net/publication/228571031_Linear_Least-Squares_Optimization_for_Point-to-Plane_ICP_Surface_Registration
27. Normal distributions transform - Wikipedia, accessed July 30, 2025, https://en.wikipedia.org/wiki/Normal_distributions_transform
28. (PDF) The Normal Distributions Transform: A New Approach to Laser Scan Matching, accessed July 30, 2025, https://www.researchgate.net/publication/4045903_The_Normal_Distributions_Transform_A_New_Approach_to_Laser_Scan_Matching
29. A novel 3D LiDAR deep learning approach for uncrewed vehicle odometry - PMC, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11322985/
30. fast_gicp - ROS Index, accessed July 30, 2025, https://index.ros.org/r/fast_gicp/
31. Accelerating Nearest Neighbor Search in 3D Point Cloud Registration on GPUs | Request PDF - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/388877886_Accelerating_Nearest_Neighbor_Search_in_3D_Point_Cloud_Registration_on_GPUs
32. vectr-ucla/direct_lidar_odometry: [IEEE RA-L & ICRA'22] A lightweight and computationally-efficient frontend LiDAR odometry solution with consistent and accurate localization. - GitHub, accessed July 30, 2025, https://github.com/vectr-ucla/direct_lidar_odometry
33. Eigen Is All You Need: Efficient Lidar-Inertial Continuous-Time Odometry with Internal Association - arXiv, accessed July 30, 2025, https://arxiv.org/html/2402.02337v1
34. Levenberg–Marquardt algorithm - Wikipedia, accessed July 30, 2025, [https://en.wikipedia.org/wiki/Levenberg%E2%80%93Marquardt_algorithm](https://en.wikipedia.org/wiki/Levenberg–Marquardt_algorithm)
35. Gauss-Newton / Levenberg-Marquardt Optimization, accessed July 30, 2025, https://mat.uab.cat/~alseda/MasterOpt/optimization.pdf
36. Levenberg-Marquardt Algorithm for Optimization - Number Analytics, accessed July 30, 2025, https://www.numberanalytics.com/blog/levenberg-marquardt-algorithm-optimization-techniques
37. Support/User Manual/Methods/Optimization Methods/Levenberg - Marquardt - COPASI, accessed July 30, 2025, https://copasi.org/Support/User_Manual/Methods/Optimization_Methods/Levenberg_-_Marquardt/
38. (PDF) Is Levenberg-Marquardt the Most Efficient Optimization Algorithm for Implementing Bundle Adjustment?. - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/221111908_Is_Levenberg-Marquardt_the_Most_Efficient_Optimization_Algorithm_for_Implementing_Bundle_Adjustment
39. LiDAR-Inertial Odometry Based on Extended Kalman Filter - arXiv, accessed July 30, 2025, https://arxiv.org/html/2407.02786v2
40. VE-LIOM: A Versatile and Efficient LiDAR-Inertial Odometry and Mapping System - MDPI, accessed July 30, 2025, https://www.mdpi.com/2072-4292/16/15/2772
41. Tightly Coupled LiDAR-Inertial Odometry and Mapping for Underground Environments, accessed July 30, 2025, https://www.mdpi.com/1424-8220/23/15/6834
42. Tightly Coupled LiDAR-Inertial Odometry Taylor Pool CMU-RI-TR-24-17 May 2024 School of Computer Science The Robotics Institue Ca, accessed July 30, 2025, https://www.ri.cmu.edu/app/uploads/2024/05/main.pdf
43. SS-LIO: Robust Tightly Coupled Solid-State LiDAR–Inertial Odometry for Indoor Degraded Environments - MDPI, accessed July 30, 2025, https://www.mdpi.com/2079-9292/14/15/2951
44. LiDAR-Inertial Odometry Based on Extended Kalman Filter - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/publication/381960596_LiDAR-Inertial_Odometry_Based_on_Extended_Kalman_Filter
45. A Tightly Coupled LiDAR-Inertial SLAM for Perceptually Degraded Scenes - PMC, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9024896/
46. vectr-ucla/direct_lidar_inertial_odometry: [IEEE ICRA'23] A new lightweight LiDAR-inertial odometry algorithm with a novel coarse-to-fine approach in constructing continuous-time trajectories for precise motion correction. - GitHub, accessed July 30, 2025, https://github.com/vectr-ucla/direct_lidar_inertial_odometry
47. Deep Learning for Laser Based Odometry Estimation - Oregon State University, accessed July 30, 2025, https://research.engr.oregonstate.edu/rdml/sites/research.engr.oregonstate.edu.rdml/files/final_deep_learning_lidar_odometry.pdf
48. A Survey on Deep-Learning-Based LiDAR 3D Object Detection for Autonomous Driving, accessed July 30, 2025, https://www.researchgate.net/publication/366119301_A_Survey_on_Deep-Learning-Based_LiDAR_3D_Object_Detection_for_Autonomous_Driving
49. Survey of Deep Learning-Based Methods for FMCW Radar Odometry and Ego-Localization, accessed July 30, 2025, https://www.mdpi.com/2076-3417/14/6/2267
50. (PDF) Leveraging Semantic Graphs for Efficient and Robust LiDAR ..., accessed July 30, 2025, https://www.researchgate.net/publication/389894376_Leveraging_Semantic_Graphs_for_Efficient_and_Robust_LiDAR_SLAM
51. Is Semantic SLAM Ready for Embedded Systems ? A Comparative Survey - arXiv, accessed July 30, 2025, https://arxiv.org/html/2505.12384v1
52. 3D Active Metric-Semantic SLAM - arXiv, accessed July 30, 2025, https://arxiv.org/html/2309.06950v4
53. SD-SLAM: A Semantic SLAM Approach for Dynamic Scenes ... - arXiv, accessed July 30, 2025, https://arxiv.org/abs/2402.18318
54. Ephemerality meets LiDAR-based Lifelong Mapping - arXiv, accessed July 30, 2025, https://arxiv.org/html/2502.13452v2
55. 3D LiDAR-IMU Lifelong SLAM with Relocalization and Autonomous Map Updating for Accurate and Reliable Navigation - Research Repository, accessed July 30, 2025, https://repository.essex.ac.uk/37954/1/my.pdf
56. A General Framework for Lifelong Localization and Mapping in Changing Environment, accessed July 30, 2025, https://www.researchgate.net/publication/357101799_A_General_Framework_for_Lifelong_Localization_and_Mapping_in_Changing_Environment
57. PoseMap: Lifelong, Multi-Environment 3D LiDAR ... - CSIRO Research, accessed July 30, 2025, https://research.csiro.au/robotics/wp-content/uploads/sites/96/2018/09/posemap_iros2018-10.pdf
58. LT-mapper: A Modular Framework for LiDAR-based Lifelong Mapping - Giseop Kim, accessed July 30, 2025, https://gisbi-kim.github.io/publications/gkim-2021-ltmapper.pdf
59. arxiv.org, accessed July 30, 2025, https://arxiv.org/html/2507.18317v1
60. Innovations and Refinements in LiDAR Odometry and Mapping: A Comprehensive Review, accessed July 30, 2025, https://www.researchgate.net/publication/390158451_Innovations_and_refinements_in_LiDAR_odometry_and_mapping_A_comprehensive_review
61. Innovations and Refinements in LiDAR Odometry and Mapping: A Comprehensive Review, accessed July 30, 2025, https://www.ieee-jas.net/en/article/doi/10.1109/JAS.2025.125198
62. Enhancing Solid State LiDAR Mapping with a 2D Spinning LiDAR in Urban Scenario SLAM on Ground Vehicles, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7961568/
63. Exploring the Differences: Solid-State LiDAR vs Mechanical LiDAR - DFRobot, accessed July 30, 2025, https://www.dfrobot.com/blog-13432.html
64. Low‐cost solid‐state LiDAR/inertial‐based localization with prior map for autonomous systems in urban scenarios - PolyU, accessed July 30, 2025, [https://www.polyu.edu.hk/aae/ipn-lab/us/publications/Fullpaper/Low-cost%20solid-state%20LiDARinertial-based%20localization%20with%20prior%20map%20for%20autonomous%20systems%20in%20urban%20scenarios.pdf](https://www.polyu.edu.hk/aae/ipn-lab/us/publications/Fullpaper/Low-cost solid-state LiDARinertial-based localization with prior map for autonomous systems in urban scenarios.pdf)
65. Intensity enhanced for solid-state-LiDAR in simultaneous localization and mapping, accessed July 30, 2025, https://www.oaepublish.com/articles/ces.2024.06
66. LiDAR Based SLAM: A Comparative Evaluation with LeGO-LOAM and HDL-Graph SLAM - DergiPark, accessed July 30, 2025, https://dergipark.org.tr/en/download/article-file/3761251
67. The KITTI Vision Benchmark Suite - Andreas Geiger, accessed July 30, 2025, https://www.cvlibs.net/datasets/kitti/eval_odometry.php
68. arXiv:1801.04678v2 [cs.RO] 27 Aug 2018, accessed July 30, 2025, https://arxiv.org/pdf/1801.04678
69. Navigating Threats: A Survey of Physical Adversarial Attacks on LiDAR Perception Systems in Autonomous Vehicles - arXiv, accessed July 30, 2025, https://arxiv.org/html/2409.20426v1
70. (PDF) Safe and Reliable Movement of Fast LiDAR-based Self-driving Vehicle, accessed July 30, 2025, https://www.researchgate.net/publication/384273786_Safe_and_Reliable_Movement_of_Fast_LiDAR-based_Self-driving_Vehicle
71. Are Lidar Lasers for Autonomous Vehicles Safe? : r/SelfDrivingCars - Reddit, accessed July 30, 2025, https://www.reddit.com/r/SelfDrivingCars/comments/m5hne8/are_lidar_lasers_for_autonomous_vehicles_safe/
72. Large-scale indoor mapping with failure detection and recovery in SLAM - Amazon Science, accessed July 30, 2025, https://www.amazon.science/publications/large-scale-indoor-mapping-with-failure-detection-and-recovery-in-slam
73. What Is SLAM (Simultaneous Localization and Mapping)? - MATLAB & Simulink, accessed July 30, 2025, https://www.mathworks.com/discovery/slam.html
74. Large-scale Indoor Mapping with Failure Detection and Recovery in SLAM - Amazon Science, accessed July 30, 2025, https://assets.amazon.science/4d/c7/f15d4d9a4c1d976df4e975144d3e/large-scale-indoor-mapping-with-failure-detection-and-recovery-in-slam.pdf
75. Block diagram of Failure Detector algorithm with SLAM - ResearchGate, accessed July 30, 2025, https://www.researchgate.net/figure/Block-diagram-of-Failure-Detector-algorithm-with-SLAM_fig1_321137183
76. One step back, two steps forward: learning moves to recover from SLAM tracking failures, accessed July 30, 2025, https://www.researchgate.net/publication/378581722_One_step_back_two_steps_forward_learning_moves_to_recover_from_SLAM_tracking_failures
77. The Role of Integrity Monitoring in Connected and Automated Vehicles: Current State-of-Practice and Future Directions - arXiv, accessed July 30, 2025, https://arxiv.org/html/2502.04874v2
78. The Role of Integrity Monitoring in Connected and Automated Vehicles: Current State-of-Practice and Future Directions - arXiv, accessed July 30, 2025, https://arxiv.org/html/2502.04874v1
79. Integrity Monitoring for Positioning Autonomous Vehicles & Robots, accessed July 30, 2025, https://trustedpositioning.tdk.com/post/integrity-monitoring-for-positioning-autonomous-vehicles-robots
80. Fail-Aware LIDAR-Based Odometry for Autonomous Vehicles - PMC, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7435861/
81. Fail-Aware LIDAR-Based Odometry for Autonomous Vehicles - Bohrium, accessed July 30, 2025, https://www.bohrium.com/paper-details/fail-aware-lidar-based-odometry-for-autonomous-vehicles/867745778951520507-108625
82. A Review of Simultaneous Localization and Mapping Algorithms Based on Lidar, accessed July 30, 2025, https://www.researchgate.net/publication/388270860_A_Review_of_Simultaneous_Localization_and_Mapping_Algorithms_Based_on_Lidar
83. A Review of Simultaneous Localization and Mapping Algorithms Based on Lidar - MDPI, accessed July 30, 2025, https://www.mdpi.com/2032-6653/16/2/56
84. A Review of Research on SLAM Technology Based on the Fusion of LiDAR and Vision, accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11902412/


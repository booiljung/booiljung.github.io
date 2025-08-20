# FPFH와 VFH의 비교

## 1.  3D 특징 기술자의 역할과 진화

3차원 포인트 클라우드는 레이저 스캐너나 깊이 카메라 등으로 수집된 공간상의 점들의 집합이다. 이 데이터는 그 자체로는 비정형(unstructured)이며, 단순히 좌표의 나열에 불과하다. 3D 정합(registration), 객체 인식(object recognition), 분할(segmentation)과 같은 고수준의 컴퓨터 비전 작업을 수행하기 위해서는, 이 점들의 집합이 내포하는 기하학적 의미를 정량적으로 표현할 수단이 반드시 필요하다. 이 핵심적인 역할을 수행하는 것이 바로 '특징 기술자(feature descriptor)'다.1

이상적인 3D 특징 기술자는 몇 가지 필수 조건을 만족해야 한다. 첫째, 강체 변환(rigid transformation), 즉 회전과 이동에 대해 불변(invariant)해야 한다. 객체의 위치나 방향이 바뀌더라도 기술자 값은 변하지 않아야 동일성을 판단할 수 있다. 둘째, 센서 노이즈나 측정 오차에 대해 강인(robust)해야 한다. 실제 데이터에 필연적으로 포함되는 노이즈가 기술자 계산에 미치는 영향을 최소화해야 일관된 성능을 보장할 수 있다. 마지막으로, 샘플링 밀도 변화에 둔감해야 한다. 같은 표면이라도 스캔 거리에 따라 점의 밀도가 달라질 수 있는데, 이러한 변화에도 기술자 값은 유사하게 유지되어야 한다.1

FPFH(Fast Point Feature Histogram)와 VFH(Viewpoint Feature Histogram)에 대한 논의는 그들의 직접적인 전신인 PFH(Point Feature Histogram)에서 시작해야 한다.4 PFH는 한 점의 k-이웃(k-neighborhood) 내에 존재하는 모든 점 쌍(point-pair) 간의 기하학적 관계, 즉 상대적인 각도와 거리를 다차원 히스토그램으로 인코딩하는 방식을 채택했다.4 이 접근법은 지역적 형상을 매우 상세하고 풍부하게 표현할 수 있었지만, 치명적인 단점을 안고 있었다. 모든 이웃 점 쌍을 고려하는 계산 방식은 $O(nk^2)$이라는 높은 계산 복잡도를 야기했고, 여기서 $n$은 전체 점의 개수, $k$는 각 점의 이웃 개수다. 이는 포인트 클라우드의 크기가 조금만 커져도 계산 시간이 기하급수적으로 늘어나 실시간 응용을 거의 불가능하게 만드는 심각한 걸림돌이었다.7

이 보고서는 바로 이 PFH의 한계를 극복하기 위해 탄생한 FPFH와, 여기서 한 걸음 더 나아가 '정합'이 아닌 '인식'이라는 새로운 문제를 풀기 위해 설계된 VFH를 심층적으로 비교 분석하는 것을 목표로 한다. 특징 기술자의 발전사는 '표현력(Descriptiveness)'과 '계산 효율성(Computational Efficiency)' 사이의 끊임없는 트레이드오프(trade-off) 과정으로 볼 수 있다. PFH에서 FPFH로의 진화는 표현력을 일부 희생하여 효율성을 극대화하는 '최적화'의 과정이었다. 반면, FPFH에서 VFH로의 발전은 단순히 성능을 개선하는 것을 넘어, 문제의 정의 자체를 '지역 형상 기술'에서 '전역 객체 인식'으로 바꾸는 '패러다임의 전환'이었다. 이 보고서는 두 기술자의 설계 철학, 계산 원리, 핵심 특성, 응용 파이프라인, 그리고 명백한 한계점까지 다각도로 고찰함으로써, 주어진 문제 상황에서 어떤 기술자를 선택하고 활용해야 하는지에 대한 명확하고 실용적인 가이드를 제공하고자 한다.

## 2.  FPFH (Fast Point Feature Histogram): 지역적 기하 구조의 신속한 기술

### 2.1  설계 목표: 속도의 혁신

FPFH의 탄생 배경과 설계 목표는 매우 명확하다. 바로 PFH의 엄청난 계산량을 실시간 응용이 가능한 수준으로 획기적으로 낮추는 것이다.5 FPFH는 이론적인 계산 복잡도를 PFH의 $O(nk^2)$에서 $O(nk)$로 극적으로 감소시켰다.7 실제 응용에서 이웃의 수 $k$가 보통 수십에서 수백에 이른다는 점을 고려하면, 이는 단순한 개선이 아닌 수십 배에서 수백 배에 달하는 성능 향상을 의미한다. 이러한 속도 혁신 덕분에 FPFH는 3D 정합과 같은 작업에서 실시간 또는 준실시간 처리를 위한 표준적인 도구로 자리 잡을 수 있었다.

### 2.2  계산 원리: 2단계 최적화 전략

FPFH는 계산량을 줄이기 위해 직접적인 이웃 관계만을 우선 계산하고, 간접적인 관계는 근사하는 영리한 2단계 전략을 사용한다.7 이 전략의 핵심은 계산 효율성과 표현력 손실 최소화 사이의 절묘한 균형을 맞추는 데 있다.

#### 2.2.1 단계: SPFH (Simplified Point Feature Histogram) 계산

첫 번째 단계는 각 쿼리 포인트 $p_q$에 대해, 그 포인트와 직접 연결된 k-이웃 $p_k$들과의 관계만을 계산하는 것이다. 이는 PFH가 $p_q$의 이웃들($p_k$) 사이의 모든 상호 관계까지 계산했던 것과 대조적이다. 이 단순화된 계산 결과를 SPFH(Simplified Point Feature Histogram)라고 부른다.7

두 점 $p_q$와 $p_k$, 그리고 각각의 법선 벡터 $n_q$와 $n_k$ 사이의 기하학적 관계는 3개의 각도 특징($\alpha, \phi, \theta$)으로 인코딩된다. 이 특징들은 두 점 중 하나를 원점으로 하고 법선 벡터와 두 점을 잇는 벡터를 이용해 구성한 지역 좌표계(Darboux frame)를 기준으로 계산된다. 이 덕분에 포인트 클라우드 전체가 회전하거나 이동하더라도 특징 값은 변하지 않는 불변성을 확보하게 된다.4 이렇게 계산된 3개의 각도 특징 값들을 각각 개별적인 히스토그램으로 만든 뒤, 이들을 단순히 연결(concatenate)한 것이 SPFH다. 이 1단계 과정만으로도 계산량은 $O(nk)$로 대폭 줄어든다.

#### 2.2.2 단계: 이웃 SPFH의 가중합을 통한 FPFH 구성

SPFH는 $p_q$와 직접 연결된 1-hop 정보만을 담고 있기 때문에, 이웃들 간의 관계까지 모두 고려하는 PFH에 비해 기하학적 맥락을 표현하는 능력이 떨어진다. FPFH는 이 정보 손실을 보완하기 위해 매우 창의적인 방법을 사용한다. 바로 $p_q$의 FPFH를 계산할 때, 자신의 SPFH뿐만 아니라 주변 이웃점들($p_k$)의 SPFH를 추가로 활용하는 것이다.1

최종 FPFH는 다음과 같은 공식으로 구성된다 7:

$$FPFH(p_q) = SPFH(p_q) + \frac{1}{k} \sum_{i=1}^{k} \frac{1}{w_i} SPFH(p_{k_i})$$

여기서 $k$는 이웃의 수, $SPFH(p_{k_i})$는 이웃점 $p_{k_i}$의 SPFH이며, $w_i$는 쿼리 포인트 $p_q$와 이웃점 $p_{k_i}$ 사이의 유클리드 거리다. 이 가중치 $w_i$는 거리가 가까운 이웃의 정보에 더 높은 중요도를 부여하는 역할을 한다.

이 가중합 과정은 추가적인 기하학 계산 없이, 이미 모든 점에서 계산된 SPFH 값을 '재활용'하여 더 넓은 범위의 정보를 통합하는 효과를 낳는다. 결과적으로 쿼리 포인트로부터 최대 $2r$(여기서 $r$은 이웃 탐색 반경) 거리까지의 기하학적 정보를 간접적으로나마 반영하게 되어, PFH에서 잃어버린 이웃-이웃 간의 연결 정보를 일부 복원하는 역할을 수행한다.7

### 2.3. FPFH의 특성과 출력

FPFH는 다음과 같은 명확한 특성을 가진다.

- **지역 기술자 (Local Descriptor):** FPFH는 이름에서 알 수 있듯이 각 포인트의 지역적(local) 형상만을 기술한다. 따라서 포인트 클라우드에 N개의 점이 있다면, 결과적으로 N개의 FPFH 기술자가 생성된다.2
- **33차원 히스토그램:** Point Cloud Library(PCL)의 표준 구현에서 FPFH는 3개의 각도 특징($\alpha, \phi, \theta$)을 각각 11개의 구간(bin)으로 나누어 히스토그램을 만들고, 이 세 개의 히스토그램을 순서대로 연결하여 총 33차원의 부동소수점 배열(float array)로 표현된다.7 이처럼 각 특징을 독립적인 히스토그램으로 만드는 '비상관화(decorrelating)' 방식은 모든 특징을 조합하여 하나의 거대한 히스토그램을 만드는 PFH의 '완전 상관(fully correlated)' 방식보다 메모리 효율적이고 계산이 간단하다.7
- **불변성 (Invariance):** 지역 좌표계(Darboux frame)를 기반으로 계산되므로 강체 변환(회전, 이동)에 불변하다. 또한 히스토그램 기반의 통계적 표현 덕분에 센서 노이즈와 점 밀도 변화에도 비교적 강인한 특성을 보인다.3

### 2.4. 주요 응용: 3D 정합

FPFH의 가장 대표적인 응용 분야는 두 개 이상의 포인트 클라우드를 동일한 좌표계로 정렬하는 3D 정합, 특히 대략적인 위치를 맞추는 초기 정합(initial/coarse alignment) 단계다.5 일반적인 FPFH 기반 정합 파이프라인은 다음과 같다.

1. **키포인트(Keypoint) 선정:** 계산 효율을 높이기 위해, 클라우드의 모든 점이 아닌 형상적 특징이 두드러지는 일부 키포인트(예: 코너, 에지)에 대해서만 FPFH를 계산한다.
2. **FPFH 계산:** 소스(source)와 타겟(target) 포인트 클라우드의 키포인트에서 각각 FPFH 특징을 추출한다.13
3. **대응점 탐색 (Correspondence Search):** 두 클라우드 간에 FPFH가 가장 유사한(예: 특징 벡터 간 유클리드 거리가 가장 가까운) 점 쌍을 찾아 초기 대응점(putative correspondences) 집합을 만든다. 이 단계에서는 잘못된 매칭이 다수 포함될 수 있다.12
4. **대응점 필터링 및 변환 추정:** RANSAC(Random Sample Consensus)이나 그 변형인 SAC-IA(Sample Consensus Initial Alignment)와 같은 알고리즘을 사용하여, 수많은 초기 대응점 중에서 기하학적으로 일관된 올바른 매칭(inlier)만을 선별하고, 이를 기반으로 최적의 변환 행렬(회전+이동)을 추정한다. 이 과정에서 잘못된 매칭(outlier)은 효과적으로 제거된다.5
5. **정밀 정합 (Fine Alignment):** FPFH와 RANSAC으로 구한 초기 정렬 결과를 바탕으로, ICP(Iterative Closest Point)와 같은 알고리즘을 적용하여 정합을 더욱 세밀하고 정확하게 다듬어 최종 결과를 얻는다.12

이처럼 FPFH는 빠르고 강인한 특성 덕분에 복잡한 정합 문제의 초기 단계를 효과적으로 해결하는 핵심적인 역할을 수행한다.

## 3. VFH (Viewpoint Feature Histogram): 객체 인식 및 자세 추정을 위한 전역적 기술

### 3.1. 설계 목표: 지역을 넘어 전역으로

VFH는 FPFH의 아이디어에 뿌리를 두고 있지만, 그 설계 목표와 철학은 근본적으로 다르다.14 FPFH가 포인트 클라우드 내 개별 '점'의 지역적 형상을 기술하는 데 초점을 맞춘다면, VFH는 분할된 '객체 클러스터 전체'를 단 하나의 기술자로 표현하는 것을 목표로 한다.14 따라서 VFH의 주요 응용 분야는 포인트 클라우드 정합이 아니라, 객체 인식(Object Recognition)과 6자유도(6DOF) 자세 추정(Pose Estimation)이다.14 즉, VFH는 "이 포인트 클라우드 덩어리는 어떤 물체인가?" 그리고 "그 물체는 공간상에 어떤 방향으로 놓여 있는가?"라는 질문에 답하기 위해 설계된 기술자다.

### 3.2. 계산 원리: 시점 정보의 명시적 통합

VFH의 가장 큰 특징은 객체의 순수한 기하학적 형상과 함께, 그 객체를 바라보는 '시점(viewpoint)' 정보를 기술자에 명시적으로 통합한다는 점이다. 이는 동일한 객체라도 어떤 각도에서 보느냐에 따라 그 모습이 완전히 달라진다는 직관적인 사실을 기술자에 반영하기 위한 전략이다.14

VFH 계산은 지역적인 점이 아닌, 객체 클러스터 전체의 중심점(centroid)과 모든 점들의 법선 벡터를 평균 내어 구한 평균 법선을 계산의 기준점으로 삼는다.14 이를 바탕으로 VFH는 크게 두 가지 핵심 구성 요소로 이루어진다.

1. **확장된 FPFH (Extended FPFH Component):** 이 부분은 FPFH의 기본 아이디어, 즉 점과 법선 간의 상대적인 각도 관계를 계산하는 방식을 차용하지만 기준점이 다르다. 클러스터 내의 모든 점 $p_i$와 그 법선 $n_i$에 대해, (1) 객체 중심점 $p_c$과 $p_i$를 잇는 벡터와 법선 간의 관계, 그리고 (2) 센서의 시점 벡터 $v_p$와 법선 $n_i$ 사이의 상대적인 각도들을 계산하여 히스토그램을 만든다. 이는 객체 전체의 형상 분포를 나타낸다.1
2. **시점 구성 요소 (Viewpoint Component):** VFH의 가장 독창적이고 핵심적인 부분이다. 센서의 시점 벡터 $v_p$와 클러스터 내 모든 점의 법선 벡터 $n_i$ 사이의 각도를 직접 계산하여 별도의 히스토그램을 만든다.14 이 히스토그램은 객체의 표면들이 현재 시점에 대해 어떤 방향으로 분포하고 있는지를 직접적으로 인코딩하며, 객체의 자세(pose) 정보를 담는 결정적인 역할을 한다.

### 3.3. VFH의 특성과 출력

VFH는 FPFH와는 확연히 다른 특성을 보인다.

- **전역 기술자 (Global Descriptor):** VFH는 포인트 클라우드 클러스터 하나당 단 하나의 기술자를 생성한다. 이는 FPFH가 점마다 기술자를 생성하는 것과 가장 근본적으로 다른 점이다. 이 단일 기술자는 객체 전체의 형상과 자세를 요약하는 '서명(signature)' 역할을 한다.14
- **308차원 히스토그램:** PCL 표준 구현에서 VFH는 매우 높은 차원의 기술자를 생성한다. 구체적으로 확장 FPFH 부분(3개의 특징 * 45 bins), 각 점과 중심점까지의 거리 히스토그램(45 bins), 그리고 시점 구성 요소(128 bins)를 모두 합쳐 총 308차원의 거대한 부동소수점 배열로 표현된다 (`(3 * 45) + 45 + 128 = 308`).14 이처럼 높은 차원은 객체 전체의 복잡한 형상과 시점 정보를 충분히 담기 위한 필연적인 결과다.
- **폐색에 대한 민감성 (Sensitivity to Occlusion):** VFH의 가장 큰 약점이자 한계점이다. VFH는 전역 기술자이므로, 객체의 일부가 다른 물체에 의해 가려지면(occluded) 심각한 문제가 발생한다. 일부 점들이 누락되면 클러스터의 중심점 위치가 크게 바뀌고, 이는 VFH 기술자 전체를 완전히 왜곡시킨다. 결과적으로, 부분적으로 가려진 객체는 온전한 객체와 전혀 다른 VFH 값을 갖게 되어 인식에 실패할 확률이 매우 높다.1

### 3.4. 주요 응용: 객체 인식 및 자세 추정

VFH는 주로 데이터베이스 기반의 검색(retrieval) 방식 인식 파이프라인에 사용된다.16 VFH의 패러다임은 '정합'이 아닌 '검색'에 가깝다.

1. 훈련 단계 (Training Stage):

   a. 인식하고자 하는 객체들의 3D 모델을 미리 준비한다.

   b. 각 객체 모델을 다양한 각도와 시점에서 렌더링하거나 실제 센서로 촬영하여 수많은 포인트 클라우드 뷰(view)를 생성한다.16

   c. 각 뷰에 대해 VFH 기술자를 계산하고, 이를 해당 뷰의 정보(객체 이름, 자세 등)와 함께 저장하여 방대한 데이터베이스를 구축한다.16

   d. 수만, 수십만 개에 달할 수 있는 이 VFH 데이터베이스를 효율적으로 검색하기 위해 kd-tree와 같은 고차원 공간 인덱싱 자료구조로 구성한다.16

2. 테스트 단계 (Testing Stage):

   a. 실제 환경에서 획득한 씬(scene) 포인트 클라우드에서 배경을 제거하고 객체 후보 클러스터들을 분할(segmentation)한다.

   b. 각 후보 클러스터에 대해 현재 시점에서 VFH 기술자를 계산한다.17

   c. 계산된 VFH를 쿼리(query)로 사용하여, 훈련 단계에서 구축한 kd-tree에서 가장 유사한(가장 가까운) VFH 기술자들을 검색한다 (Nearest Neighbor Search).16

   d. 검색된 최상위 후보들이 인식 결과가 된다. 데이터베이스에 저장된 뷰 정보를 통해 단순히 객체의 종류뿐만 아니라, 그 객체의 6DOF 자세까지 한번에 추정할 수 있다.

## 4. FPFH 대 VFH: 핵심 차이점 비교 분석

FPFH와 VFH는 동일한 기술(PFH)에서 파생되었음에도 불구하고, 그 철학과 목적, 특성은 극명한 대조를 이룬다. 이 둘의 차이를 이해하는 것은 3D 포인트 클라우드 처리에서 올바른 도구를 선택하는 첫걸음이다.

| 구분 (Category)                             | FPFH (Fast Point Feature Histogram)                          | VFH (Viewpoint Feature Histogram)                            |
| ------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **핵심 철학 (Core Philosophy)**             | 지역 기하학의 **신속한 기술** (Rapid **description** of local geometry) | 전역 객체의 **시점 기반 인식** (Viewpoint-based **recognition** of a global object) |
| **목적 (Purpose)**                          | 3D 정합 (Point Cloud Registration) 5                         | 객체 인식 및 6DOF 자세 추정 (Object Recognition & Pose Estimation) 14 |
| **범위 (Scope)**                            | **지역적 (Local)** - 포인트 중심의 이웃 7                    | **전역적 (Global)** - 클러스터 전체 14                       |
| **계산 단위 (Unit)**                        | 입력 클라우드의 **각 포인트** (Per point)                    | 입력 클라우드 **전체에 대해 단일** (Per cluster)             |
| **출력 (Output)**                           | **N개**의 기술자 (N = 포인트 수) 7                           | **1개**의 기술자 14                                          |
| **인코딩 정보 (Encoded Info)**              | 지역적 기하 구조 (상대 각도) 7                               | 전역 기하 구조 + 시점 정보 1                                 |
| **계산 복잡도 (Complexity)**                | $O(nk)$ 7                                                    | FPFH 기반이므로 유사하게 효율적, 단 클러스터 단위로 1회 계산 |
| **차원 (Dimensionality)**                   | **33차원** (상대적으로 낮음) 7                               | **308차원** (상대적으로 높음) 14                             |
| **폐색 강인성 (Robustness to Occlusion)**   | **상대적으로 강함** (지역적이므로)                           | **민감함** (전역적이므로) 21                                 |
| **주요 응용 파이프라인 (Typical Pipeline)** | RANSAC/SAC-IA 기반 초기 정합 12                              | 데이터베이스 및 kd-tree 기반 검색 16                         |
| **대표적 PCL 클래스 (PCL Class)**           | `pcl::FPFHEstimation` 25                                     | `pcl::VFHEstimation` 18                                      |
| **진화적 맥락 (Evolutionary Context)**      | PFH의 **최적화** (Optimization of PFH)                       | FPFH의 **확장 및 용도 변경** (Extension of FPFH)             |

### 심층 분석

위 표에 나타난 차이점들은 독립적인 것이 아니라 서로 긴밀하게 연결된 인과 관계를 형성한다.

- **목적과 범위의 인과관계:** VFH가 '인식'을 목표로 삼았기 때문에, 객체 전체를 아우르는 '전역적' 범위를 채택하는 것은 필연적이었다. 객체를 식별하려면 그 객체만의 고유한 전체 형태를 담아야 하기 때문이다. 반면 FPFH는 두 클라우드 간의 '정합'을 목표로 하므로, 점과 점 사이의 대응 관계를 찾아야 한다. 이를 위해서는 각 점의 주변 환경을 기술하는 '지역적' 범위가 필수적이다. 결국 기술자의 목적이 그 범위를 결정한 것이다.
- **출력과 파이프라인의 연결:** FPFH는 포인트 클라우드의 점 개수만큼(N개) 많은 기술자를 출력한다. 이 수많은 기술자 중에서 올바른 대응 관계를 찾아내기 위해 RANSAC처럼 다수의 가설을 세우고 통계적으로 검증하는 파이프라인이 자연스럽게 채택된다. 반면 VFH는 클러스터당 단 하나의 '서명(signature)'을 출력한다. 이 단일 서명을 가지고 할 수 있는 가장 효율적인 작업은, 미리 만들어 둔 방대한 서명 카탈로그(데이터베이스)에서 가장 비슷한 것을 '검색'하는 것이다. 이처럼 기술자의 출력 형태가 응용 파이프라인의 구조를 결정짓는다.
- **차원과 정보량의 관계:** VFH의 308차원이라는 높은 차원은 단순히 숫자가 큰 것이 아니다. 이는 객체 전체의 복잡한 3차원 형상과 그것을 바라보는 시점이라는 방대한 정보를 하나의 벡터에 압축해 담기 위한 필연적인 결과다. 반대로 FPFH의 33차원은 지역적 기하 구조라는 상대적으로 단순한 정보를 효율적으로 담기에 충분한 크기다. 기술자의 차원은 그것이 담고 있는 정보의 양과 복잡성을 직접적으로 반영한다.
- **폐색 강인성의 근본 원인:** 이 둘의 가장 극명한 차이인 폐색 강인성은 '지역'과 '전역'이라는 설계 철학에서 비롯된 피할 수 없는 결과다. FPFH의 경우, 객체의 일부가 가려져 보이지 않더라도, 보이는 다른 부분의 점들에서는 FPFH가 정상적으로 계산된다. 따라서 이 보이는 부분들 간의 대응 관계만으로도 정합을 수행할 수 있다. 하지만 VFH는 객체 일부가 가려지면 '전체'를 대표하는 중심점의 위치가 흔들리고, 이는 기술자 계산의 기준점 자체를 왜곡시켜 기술자 전체를 망가뜨린다.

결론적으로 FPFH와 VFH는 "어느 것이 더 우수한가?"를 논할 수 있는 경쟁 관계가 아니다. 이들은 3D 처리 파이프라인의 서로 다른 단계에서 각자의 고유한 역할을 수행하는 상호 보완적인 관계에 가깝다. 예를 들어, 자율주행 로봇 시스템을 상상해보자. 로봇은 먼저 FPFH를 사용하여 현재 센서로 들어온 미지의 씬(scene)과 기존에 구축된 맵을 '정합'하여 자신의 위치를 파악할 수 있다(SLAM). 그 다음, 정합된 씬 내부에서 특정 객체 클러스터를 분할한 뒤, VFH를 계산하여 "이 클러스터는 데이터베이스에 있는 '의자' 모델과 92% 유사하다"라고 '인식'할 수 있다. 이처럼 FPFH는 '어디에 있는가(Where)'의 문제를, VFH는 '무엇인가(What)'의 문제를 해결하는 데 특화된, 각기 다른 목적을 가진 도구로 이해해야 한다.

## 5. 결론: 올바른 기술자 선택과 그 너머

FPFH와 VFH는 3D 포인트 클라우드 처리 분야에서 각기 다른 문제 해결을 위해 설계된 중요한 이정표다. 이 둘에 대한 깊이 있는 이해는 효과적인 3D 비전 시스템을 구축하기 위한 필수적인 소양이다.

### FPFH와 VFH 요약 및 선택 가이드라인

- **FPFH를 선택해야 할 때:** 두 개 이상의 포인트 클라우드 간의 정밀한 공간적 정렬, 즉 **정합(registration)**이 주된 목표일 때 선택해야 한다. 대표적인 예로는 SLAM 과정에서의 지도 생성, 여러 각도에서 스캔한 3D 모델 데이터 병합, 객체 자세 추적(tracking)의 초기화 단계 등이 있다. 특히 계산 속도가 중요하고, 데이터 간에 부분적인 겹침(overlap)만 존재하는 시나리오에서 강력한 성능을 발휘한다.
- **VFH를 선택해야 할 때:** 사전에 정의된 객체 라이브러리에서 특정 객체를 찾아내고 그 6DOF 자세까지 추정하는 **인식(recognition)** 및 **자세 추정(pose estimation)**이 목표일 때 적합하다. 공장 자동화 라인에서의 부품 피킹(picking), 가정용 로봇의 특정 물체 인식 및 조작 등이 대표적인 응용 사례다. 단, 이 기술자를 사용하기 위해서는 객체가 대부분 가려지지 않은 상태로 관측될 것이라는 강한 가정이 필요하다.

### 한계 극복을 위한 진화: CVFH (Clustered Viewpoint Feature Histogram)

VFH의 가장 치명적인 약점인 폐색 민감성을 해결하기 위한 노력의 결과물로 CVFH(Clustered Viewpoint Feature Histogram)가 제안되었다.26

CVFH의 핵심 아이디어는 VFH의 전역성을 전략적으로 완화하는 것이다. 객체 전체에 대해 단 하나의 VFH를 계산하는 대신, 먼저 객체 표면을 기하학적으로 안정적인 여러 개의 평활 영역(smooth region)으로 분할(clustering)한다.27 그리고 각 안정적인 영역(클러스터)을 하나의 독립적인 기준점으로 삼아, VFH와 유사한 기술자를 계산한다. 이렇게 하면 하나의 객체에서 여러 개의 '준-전역(semi-global)' 또는 '메타-지역(meta-local)' 기술자가 생성된다.27

이러한 접근 방식은 폐색에 대한 강인성을 획기적으로 높인다. 객체의 일부가 다른 물체에 의해 가려지더라도, 가려지지 않고 보이는 부분에 위치한 안정적인 영역들의 CVFH는 여전히 정상적으로 계산되고 데이터베이스와 매칭될 수 있기 때문이다.28 이는 VFH의 엄격한 전역성을 일부 희생하는 대신, 실제 환경에서 훨씬 중요한 폐색 강인성을 얻는 매우 실용적인 타협이다. CVFH는 FPFH(완전 지역)와 VFH(완전 전역) 사이의 효과적인 중간 지점을 차지하며, 기술자 설계가 어떻게 문제에 맞춰 진화하는지를 보여주는 좋은 사례다.

### 향후 전망: 전통에서 학습으로

FPFH, VFH, 그리고 CVFH는 모두 전문가의 도메인 지식과 정교한 휴리스틱에 기반하여 설계된 '전통적인(hand-crafted)' 기술자다.3 이들은 특정 기하학적 가정하에서 매우 효율적이고 설명 가능하게 작동하지만, 예측 불가능하고 복잡한 실제 데이터에 대한 일반화 성능에는 명백한 한계가 있다.

최근 3D 딥러닝 기술의 폭발적인 발전으로, 데이터로부터 특징 표현 자체를 학습하는 '학습 기반(learning-based)' 기술자들이 3D 비전 분야의 새로운 패러다임으로 부상하고 있다.3 이들은 방대한 데이터를 통해 스스로 최적의 특징을 학습함으로써 전통적인 방법보다 월등히 높은 표현력과 강인성을 보여주며, 정합 및 인식 성능의 기준을 빠르게 끌어올리고 있다.

하지만 FPFH와 VFH의 설계에 담긴 근본적인 철학-지역적 기하 관계의 중요성(FPFH), 시점 정보의 명시적 통합(VFH), 폐색 문제 해결을 위한 클러스터링 접근(CVFH)-은 여전히 유효하다. 이러한 개념들은 많은 최신 학습 기반 방법론에 직간접적으로 영감을 주고 있으며, 이들의 동작 원리를 이해하는 데 중요한 기초를 제공한다. 따라서 이 전통적인 기술자들에 대한 깊이 있는 이해는 빠르게 변화하는 3D 비전 기술의 근본을 파악하고 미래를 전망하는 데 필수적인 자산으로 남을 것이다.

#### **참고 자료**

1. Feature / PCL - adioshun, accessed July 24, 2025, https://adioshun.gitbooks.io/pcl/content/Tutorial/Feature/
2. PCL/OpenNI tutorial 4: 3D object recognition (descriptors) - robotica.unileon.es, accessed July 24, 2025, https://robotica.unileon.es/index.php?title=PCL/OpenNI_tutorial_4:_3D_object_recognition_(descriptors)
3. (PDF) FPFH Revisited: Histogram Resolutions, Improved Features, and Novel Representation - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/380557912_FPFH_revisited_histogram_resolutions_improved_features_and_novel_representation
4. Point Feature Histograms (PFH) descriptors - Point Cloud Library 0.0 documentation, accessed July 24, 2025, https://pcl.readthedocs.io/projects/tutorials/en/master/pfh_estimation.html
5. Fast Point Feature Histograms (FPFH) for 3D Registration - 3D Vision Laboratory, accessed July 24, 2025, https://www.cvl.iis.u-tokyo.ac.jp/class2016/2016w/papers/6.3DdataProcessing/Rusu_FPFH_ICRA2009.pdf
6. Point Feature Histograms (PFH) descriptors - Point Cloud Library ..., accessed July 24, 2025, https://pointclouds.org/documentation/tutorials/pfh_estimation.html
7. Fast Point Feature Histograms (FPFH) descriptors - Point Cloud Library 0.0 documentation, accessed July 24, 2025, https://pcl.readthedocs.io/projects/tutorials/en/master/fpfh_estimation.html
8. Fast Point Feature Histograms (FPFH) for 3D Registration - Papers With Code, accessed July 24, 2025, https://paperswithcode.com/paper/fast-point-feature-histograms-fpfh-for-3d
9. Fast Point Feature Histograms (FPFH) for 3D registration | Request PDF - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/224557255_Fast_Point_Feature_Histograms_FPFH_for_3D_registration
10. (PDF) Point Cloud Segmentation Based on FPFH Features - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/308278830_Point_Cloud_Segmentation_Based_on_FPFH_Features
11. extractFPFHFeatures - Extract fast point feature histogram (FPFH) descriptors from point cloud - MATLAB - MathWorks, accessed July 24, 2025, https://www.mathworks.com/help/lidar/ref/extractfpfhfeatures.html
12. Point Cloud Registration Based on Fast Point Feature Histogram Descriptors for 3D Reconstruction of Trees - MDPI, accessed July 24, 2025, https://www.mdpi.com/2072-4292/15/15/3775
13. hankyang94/teaser_fpfh_threedmatch_python - GitHub, accessed July 24, 2025, https://github.com/hankyang94/teaser_fpfh_threedmatch_python
14. Estimating VFH signatures for a set of points - Point Cloud Library ..., accessed July 24, 2025, https://pcl.readthedocs.io/projects/tutorials/en/master/vfh_estimation.html
15. Estimating VFH signatures for a set of points - Point Cloud Library 1.15.0-dev documentation, accessed July 24, 2025, https://pointclouds.org/documentation/tutorials/vfh_estimation.html
16. pcl/doc/tutorials/content/vfh_recognition.rst at master / PointCloudLibrary/pcl - GitHub, accessed July 24, 2025, https://github.com/PointCloudLibrary/pcl/blob/master/doc/tutorials/content/vfh_recognition.rst
17. Cluster Recognition and 6DOF Pose Estimation using VFH descriptors - Point Cloud Library, accessed July 24, 2025, https://pointclouds.org/documentation/tutorials/vfh_recognition.html
18. pcl::VFHEstimation< PointInT, PointNT, PointOutT > Class Template Reference, accessed July 24, 2025, http://docs.ros.org/hydro/api/pcl/html/classpcl_1_1VFHEstimation.html
19. File:VFH viewpoint component.png - robotica.unileon.es, accessed July 24, 2025, https://robotica.unileon.es/index.php?title=File:VFH_viewpoint_component.png
20. pcl: vfh.h File Reference - ROS, accessed July 24, 2025, https://docs.ros.org/en/hydro/api/pcl/html/vfh_8h.html
21. Detecting Partially Occluded Objects via Segmentation and Validation - Mike Stilman, accessed July 24, 2025, http://www.golems.org/papers/LevihnWORV2013.pdf
22. Detecting Partially Occluded Objects via Segmentation and Validation - Mike Stilman, accessed July 24, 2025, http://www.golems.org/papers/Levihn12-furniture-techreport.pdf
23. Cluster Recognition and 6DOF Pose Estimation using VFH descriptors - Compiling PCL, accessed July 24, 2025, https://pcl.readthedocs.io/projects/tutorials/en/latest/vfh_recognition.html
24. Cluster Recognition and 6DOF Pose Estimation using VFH descriptors - Compiling PCL, accessed July 24, 2025, https://pcl.readthedocs.io/projects/tutorials/en/pcl-1.12.0/vfh_recognition.html
25. pcl::FPFHEstimation< PointInT, PointNT, PointOutT > Class Template Reference - Point Cloud Library, accessed July 24, 2025, https://pointclouds.org/documentation/classpcl_1_1_f_p_f_h_estimation.html
26. Module features - Point Cloud Library (PCL), accessed July 24, 2025, http://pointclouds.org/documentation/group__features.html
27. An Efficient Global Point Cloud Descriptor for Object Recognition ..., accessed July 24, 2025, http://sibgrapi.sid.inpe.br/col/sid.inpe.br/sibgrapi/2016/07.11.17.39/doc/PID4349755.pdf
28. vfh_recognition - ROS Wiki, accessed July 24, 2025, http://wiki.ros.org/vfh_recognition
29. PPFNet: Global Context Aware Local Features for Robust 3D Point Matching - CVF Open Access, accessed July 24, 2025, https://openaccess.thecvf.com/content_cvpr_2018/CameraReady/1025.pdf
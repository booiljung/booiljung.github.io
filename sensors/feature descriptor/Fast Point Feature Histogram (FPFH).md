# Fast Point Feature Histogram (FPFH)

## 서론: FPFH의 탄생 배경과 의의

### 0.1 D 포인트 클라우드 정합 문제의 본질

3차원 포인트 클라우드 정합(3D Point Cloud Registration)은 컴퓨터 비전, 로보틱스, 3D 재구성, 자율 주행 등 수많은 첨단 기술 분야의 근간을 이루는 핵심 과제이다.1 이 문제의 본질은 서로 다른 시점이나 센서로부터 취득된 두 개 이상의 3D 포인트 클라우드 데이터셋을 하나의 일관된 좌표계로 정렬하는 과정으로 정의된다.4 성공적인 정합은 분리된 데이터 조각들을 모아 완전한 3D 모델을 생성하거나, 로봇이 자신의 위치를 파악하고 환경 지도를 작성(SLAM)하는 데 필수적이다.

그러나 이 문제는 보기보다 훨씬 복잡한 난제들을 포함한다. 특히, 두 데이터셋 간의 초기 상대 위치 및 방향에 대한 정보가 전혀 없는 경우, 정합은 매우 어려운 최적화 문제가 된다. 정합 과정을 수학적으로 모델링하면, 두 데이터셋의 중첩 영역 간의 거리를 최소화하는 최적의 회전(Rotation) 및 이동(Translation) 변환 행렬을 찾는 것으로 귀결된다. 문제는 이 최적화 함수가 수많은 지역 최적해(local optima)를 가지는 다차원 비선형 함수라는 점이다.4 따라서 경사 하강법과 같은 일반적인 지역 최적화 기법들은 전역 최적해(global optimum)가 아닌, 시작점에 가까운 지역 최적해에 수렴하여 정합에 실패할 확률이 매우 높다.6

이러한 문제를 해결하기 위해, 정합 과정은 보통 두 단계로 나뉜다. 첫 번째는 초기 정합(Coarse Registration) 단계로, 전역적인 탐색을 통해 대략적인 정렬을 수행하여 두 데이터셋을 전역 최적해의 '수렴 영역(convergence basin)' 내로 근접시킨다. 두 번째는 정밀 정합(Fine Registration) 단계로, 초기 정합 결과를 바탕으로 ICP(Iterative Closest Point)와 같은 알고리즘을 사용하여 정밀하게 최종 정렬을 수행한다. FPFH는 바로 이 초기 정합 단계에서 핵심적인 역할을 수행하는 기술자(descriptor)이다.

### 0.2 선행 기술: 점 특징 히스토그램 (PFH, Point Feature Histogram)

FPFH를 이해하기 위해서는 그 전신인 점 특징 히스토그램(Point Feature Histogram, PFH)을 먼저 살펴보아야 한다. PFH는 2008년 Rusu 등에 의해 제안된 지역 특징 기술자(local feature descriptor)로, 한 점의 국소적인 표면 기하학 정보를 인코딩하기 위해 설계되었다.7 PFH의 핵심 아이디어는 쿼리 포인트($p_q$) 주변의 이웃 점들($k$-neighborhood) 간의 모든 기하학적 관계를 포착하여 다차원 히스토그램으로 표현하는 것이다.

구체적으로, 쿼리 포인트의 특정 반경($r$) 내에 있는 모든 이웃 점들의 쌍($p_i, p_j$)에 대해, 두 점의 위치와 법선 벡터(surface normal)를 이용하여 지역 좌표계를 설정하고, 이 좌표계상에서 두 법선 벡터 간의 상대적인 각도 차이와 두 점 사이의 거리를 계산한다. 이렇게 계산된 4개의 값(angular features $\alpha, \phi, \theta$와 distance $d$) 튜플을 모든 점 쌍에 대해 수집하고, 각 값의 범위를 여러 구간(bin)으로 나누어 히스토그램을 생성한다.7 이 히스토그램은 해당 지점의 국소적인 곡률과 표면 변화를 요약한 고유한 '지문' 역할을 하며, 객체의 자세(pose)나 포인트 밀도 변화에 강건한 특성을 보인다.

그러나 PFH는 치명적인 단점을 가지고 있었다. 바로 엄청난 계산 복잡도이다. 포인트 클라우드의 전체 점 개수를 $n$, 각 점의 평균 이웃 개수를 $k$라고 할 때, PFH의 계산 복잡도는 $O(nk^2)$에 달한다.9

$k^2$ 항은 각 쿼리 포인트에 대해 모든 이웃 점 쌍을 고려해야 하기 때문에 발생하며, 이는 포인트 클라우드가 커지거나 밀도가 높아질수록 계산 시간을 기하급수적으로 증가시켜 실시간 응용을 거의 불가능하게 만드는 주된 병목 지점이 되었다.5

### 0.3 FPFH의 등장: 속도와 표현력의 균형

이러한 PFH의 계산적 한계를 극복하기 위해, 동일한 연구 그룹인 Rusu 등은 2009년 IEEE 국제 로봇 및 자동화 학회(ICRA)에서 Fast Point Feature Histogram (FPFH)을 제안했다.4 FPFH의 핵심적인 개발 동기는 PFH가 가진 강력한 기하학적 표현력(discriminative power)을 대부분 유지하면서도, 계산 복잡도를 획기적으로 낮추어 실시간 응용이 가능하도록 만드는 것이었다.12

FPFH는 PFH의 계산 방식을 근본적으로 재설계하여 계산 복잡도를 $O(nk)$로 낮추는 데 성공했다.4 이는 단순히 코드를 최적화한 수준이 아니라, 정보 수집의 범위를 재정의하고 일부 정보를 근사(approximation)하는 방식을 채택한 설계 철학의 전환이었다. PFH가 쿼리 포인트의 반경 $r$ 내 이웃들 간의 관계를 '완전 그래프(fully connected graph)' 형태로 정밀하게 계산했다면, FPFH는 이를 두 단계로 나누어 근사한다. 첫 번째 단계에서는 쿼리 포인트와 그 직접적인 이웃들 간의 관계만을 계산하는 '스타형(star-shaped)' 그래프를 고려한다. 이 과정에서 이웃과 이웃 간의 직접적인 관계 정보는 손실되지만, 계산량은 $k$에 비례하게 된다. 두 번째 단계에서는 이 손실된 정보를 보상하기 위해, 각 이웃 점들의 1단계 계산 결과(SPFH)를 가중 평균하여 쿼리 포인트의 최종 FPFH에 합산한다. 이 과정은 결과적으로 쿼리 포인트의 특징이 최대 $2r$ 거리(쿼리 포인트에서 이웃까지의 거리 $r$ + 그 이웃에서 자신의 이웃까지의 거리 $r$)에 있는 점들의 정보까지 간접적으로 반영하게 만든다.10 즉, FPFH는 PFH보다 더 넓은 영역의 정보를 다소 '흐릿하게(blurred)' 반영하는 대신, 계산 속도를 비약적으로 향상시킨다. 이는 정밀도를 일부 희생하여 실용성을 극대화하는 전형적인 공학적 트레이드오프이며, FPFH가 3D 비전 분야에서 광범위하게 채택될 수 있었던 핵심적인 이유이다.

이와 함께 Rusu 등은 FPFH를 활용한 효율적인 초기 정합 알고리즘인 SAC-IA(Sample Consensus Initial Alignment)를 제안하여, FPFH의 실용적 가치를 입증했다.4

## 1. FPFH 알고리즘의 원리 및 수학적 고찰

FPFH 알고리즘의 핵심은 PFH의 $O(nk^2)$ 계산 복잡도를 $O(nk)$로 줄이기 위한 2단계 계산 프로세스에 있다. 이 프로세스는 정보의 범위를 확장하고 계산을 단순화하는 영리한 방식으로 구성된다.10

### 1.1 단계: SPFH (Simplified Point Feature Histogram) 계산

FPFH 계산의 첫 번째 단계는 각 포인트에 대해 Simplified Point Feature Histogram (SPFH)을 구하는 것이다. SPFH는 이름에서 알 수 있듯이 PFH의 계산 과정을 대폭 단순화한 버전이다.10 PFH가 쿼리 포인트의 이웃 내 모든 점 쌍을 고려하는 것과 달리, SPFH는 오직 쿼리 포인트 $p_q$와 그 반경 $r$ 내의 각 이웃점 $p_k$ 사이의 관계만을 계산한다.

계산 과정은 다음과 같다.

1. **Darboux 좌표계 설정**: 두 점 $p_i$와 $p_j$, 그리고 각각의 법선 벡터 $n_i$와 $n_j$가 주어졌을 때(SPFH 계산에서는 $p_i$가 쿼리 포인트, $p_j$가 이웃 포인트가 된다), 한 점($p_i$)을 기준으로 지역 좌표계 ($u, v, w$)를 정의한다. 이 좌표계는 두 점과 법선 벡터의 상대적인 기하학적 관계를 일관되게 표현하기 위한 기준틀 역할을 한다.4

   - $u = n_i$
   - $v = u \times \frac{(p_j - p_i)}{||p_j - p_i||}$
   - $w = u \times v$

2. **3가지 각도 특징(Angular Features) 계산**: 위에서 정의한 Darboux 좌표계를 기준으로, 두 법선 벡터 $n_i$와 $n_j$ 간의 관계를 다음의 세 가지 각도 특징으로 수치화한다.7

   - $\alpha = v \cdot n_j$
   - $\phi = u \cdot \frac{(p_j - p_i)}{d}$
   - $\theta = \arctan(w \cdot n_j, u \cdot n_j)$

   여기서 $d = ||p_j - p_i||$는 두 점 사이의 유클리드 거리이다. 원래 PFH는 이 거리 $d$를 네 번째 특징으로 사용했지만, FPFH 및 최신 PFH 구현에서는 스케일 변화와 포인트 밀도 변화에 대한 강건성을 높이기 위해 이 특징을 제외하는 경우가 많다. 특히 2.5D 스캔 데이터에서는 센서로부터의 거리에 따라 점 간 거리가 달라지므로 $d$를 제외하는 것이 유리하다.4

3. **SPFH 히스토그램 생성**: 쿼리 포인트 $p_q$와 그 주변의 모든 이웃 $p_k$에 대해 계산된 ($\alpha, \phi, \theta$) 튜플들을 수집한다. 그리고 각 특징 차원($\alpha$, $\phi$, $\theta$)의 값 범위를 미리 정의된 개수의 구간(bin)으로 나누고, 계산된 값들이 어느 구간에 속하는지를 카운트하여 히스토그램을 생성한다.10 이 히스토그램이 바로 $p_q$의 SPFH가 된다.

### 1.2 단계: 이웃 SPFH의 가중치 합산을 통한 FPFH 구성

SPFH는 계산이 매우 빠르지만, 쿼리 포인트와 직접적인 이웃 간의 정보만을 담고 있어 이웃들 간의 상호작용을 놓치게 되므로 표현력이 부족하다. FPFH는 이 단점을 보완하기 위해 두 번째 단계를 도입한다. 이 단계의 핵심 아이디어는 쿼리 포인트 $p_q$의 최종 FPFH를 계산할 때, 자신의 SPFH뿐만 아니라 주변 이웃들의 SPFH까지 함께 고려하는 것이다.10

이를 위해, $p_q$의 각 이웃 $p_k$에 대해서도 SPFH를 미리 계산해 둔다. 그리고 $p_q$의 최종 FPFH는 자신의 SPFH와, 모든 이웃 $p_k$들의 SPFH를 가중 평균하여 합산한 결과로 정의된다. 이 과정을 수학식으로 표현하면 다음과 같다.
$$
\text{FPFH}(p_q) = \text{SPFH}(p_q) + \frac{1}{k} \sum_{i=1}^{k} \frac{1}{\omega_i} \cdot \text{SPFH}(p_k)
$$
여기서 각 변수는 다음을 의미한다.

- $p_q$: 현재 FPFH를 계산하려는 쿼리 포인트.
- $k$: $p_q$의 반경 $r$ 내 이웃의 총 개수.
- $p_k$: $p_q$의 $k$개 이웃 중 하나.
- $\text{SPFH}(p)$: 점 $p$에 대해 1단계에서 계산된 Simplified Point Feature Histogram.
- $\omega_k$: 쿼리 포인트 $p_q$와 이웃점 $p_k$ 사이의 관계를 나타내는 가중치. 일반적으로 두 점 사이의 유클리드 거리($||p_q - p_k||$)가 사용된다.15 이 가중치 항($1/\omega_k$)은 거리가 가까운 이웃의 SPFH에 더 높은 가중치를 부여하여, 지역성에 기반한 영향력을 조절하는 역할을 한다.

이 가중 합산 과정은 쿼리 포인트 $p_q$가 자신의 직접적인 이웃뿐만 아니라 '이웃의 이웃'에 대한 정보까지 간접적으로 포함하게 하여, SPFH의 부족한 표현력을 보완하고 PFH와 유사한 수준의 변별력을 갖추게 한다.

### 1.3 최종 특징 벡터: 33차원 히스토그램

FPFH의 마지막 특징은 계산 효율성과 단순성을 위해 각 특징 차원을 독립적으로 취급한다는 점이다. 이를 특징의 비상관화(Decorrelation)라고 부른다.10 PFH가 ($\alpha, \phi, \theta, d$) 네 특징을 결합하여 하나의 고차원 공간에 히스토그램을 만드는 것과 달리, FPFH는 세 개의 각도 특징($\alpha, \phi, \theta$) 각각에 대해 독립적인 히스토그램을 계산한다. 그런 다음, 이 세 개의 히스토그램을 단순히 순서대로 이어 붙여(concatenate) 최종 특징 벡터를 생성한다.

가장 널리 사용되는 Point Cloud Library (PCL)의 기본 구현에서는 각 특징 차원을 11개의 구간(bin)으로 나누어 히스토그램을 만든다. 따라서 최종적으로 생성되는 FPFH 기술자는 $11 \text{ bins} \times 3 \text{ features} = 33$차원의 실수(float) 배열이 된다.10 이 33차원 벡터가 해당 포인트의 국소적 기하학 정보를 요약하는 최종적인 '지문'이 된다.

#### 1.3.1 Table 1: PFH vs. FPFH 비교 분석

| 특징 (Feature)                  | PFH (Point Feature Histogram)                         | FPFH (Fast Point Feature Histogram)                          |
| ------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| **계산 복잡도**                 | $O(nk^2)$                                             | $O(nk)$                                                      |
| **계산 방식**                   | 1단계: 모든 이웃 쌍($p_i, p_j$) 간의 관계를 직접 계산 | 2단계: 1) 쿼리-이웃($p_q, p_k$) 관계(SPFH) 계산, 2) 이웃 SPFH 가중 합산 |
| **이웃 관계 그래프**            | 완전 그래프 (Fully Connected)                         | 스타형 그래프 + 이웃 가중치 (Star-shaped + Neighbor Weighting) |
| **영향 범위 (Receptive Field)** | 반경 $r$ 내로 엄격히 제한됨                           | 최대 $2r$까지 간접적으로 확장됨 10                           |
| **표현력 (Descriptiveness)**    | 높음. 국소 기하학을 정밀하게 모델링                   | PFH의 대부분을 유지하나, 일부 정보 손실로 인해 약간 낮음 4   |
| **히스토그램 구성**             | 4개 특징을 결합한 다차원 히스토그램                   | 3개 특징을 개별 히스토그램으로 계산 후 연결 (Decorrelated)   |
| **기본 차원 (PCL)**             | 125 (5 bins per feature, 4 features)                  | 33 (11 bins per feature, 3 features)                         |
| **핵심 장점**                   | 높은 변별력                                           | 빠른 계산 속도, 실시간 응용 가능                             |
| **핵심 단점**                   | 느린 계산 속도                                        | PFH 대비 약간 낮은 표현력, 파라미터 민감성                   |

## 2. FPFH의 실제적 구현 및 응용

FPFH는 이론적 우수성뿐만 아니라 실제 응용에서의 효율성 덕분에 3D 비전 분야에서 널리 사용되는 기술자가 되었다. 특히 포인트 클라우드 정합 파이프라인에서 그 가치가 두드러진다.

### 2.1 표준 정합 파이프라인에서의 FPFH 역할

FPFH는 주로 두 포인트 클라우드 간의 대략적인 정렬을 찾는 초기 정합(Coarse Registration) 단계에서 핵심적인 역할을 수행한다. 이 단계의 목표는 ICP(Iterative Closest Point)와 같은 정밀 정합(Fine Registration) 알고리즘이 지역 최적해에 빠지지 않고 전역 최적해로 수렴할 수 있도록 좋은 초기 변환 행렬을 제공하는 것이다.5 FPFH를 사용한 전형적인 정합 워크플로우는 다음과 같은 단계로 구성된다.1

1. **전처리 (Preprocessing):** 계산 효율성과 정확도를 높이기 위한 준비 단계이다.
   - **Voxel Grid Downsampling:** 포인트 클라우드의 점 개수를 줄여 후속 계산의 속도를 높이고, 점들의 밀도를 비교적 균일하게 만들어 특징 계산의 일관성을 향상시킨다.19
   - **Outlier Removal:** 통계적 이상치 제거(Statistical Outlier Removal)와 같은 방법을 사용하여 측정 오류나 노이즈로 인해 발생한 이상치 점들을 제거한다. 이는 잘못된 특징 계산과 매칭을 방지하여 정합 실패 가능성을 줄인다.20
2. **법선 벡터 추정 (Normal Estimation):** FPFH 계산의 필수 입력 데이터인 법선 벡터를 각 점에 대해 추정한다. 법선은 해당 점의 국소적인 표면 방향을 나타내는 벡터로, 보통 KD-Tree와 같은 자료구조를 사용하여 각 점의 이웃을 빠르게 찾고, 이웃 점들의 분포를 기반으로 주성분 분석(PCA) 등을 통해 계산된다.12
3. **FPFH 특징 추출 (FPFH Feature Extraction):** 전처리된 포인트 클라우드와 추정된 법선 벡터를 입력으로 받아, 각 점에 대한 33차원의 FPFH 기술자 벡터를 계산한다.1 이 벡터는 각 점의 고유한 기하학적 '지문' 역할을 한다.
4. **대응점 탐색 및 이상치 제거 (Correspondence Matching and Outlier Rejection):**
   - **대응점 탐색:** 소스(Source) 포인트 클라우드의 각 점에 대해, 그 점의 FPFH 기술자와 가장 유사한(예: 유클리드 거리가 가장 가까운) FPFH 기술자를 타겟(Target) 포인트 클라우드에서 찾는다. 이렇게 찾아진 점 쌍들이 초기 대응점(putative correspondences)이 된다.19
   - **이상치 제거:** 초기 대응점 집합에는 기하학적 유사성으로 인해 잘못 짝지어진 많은 이상치(outlier)가 포함되어 있다. RANSAC(Random Sample Consensus) 또는 FPFH 논문에서 제안된 SAC-IA(Sample Consensus Initial Alignment)와 같은 알고리즘을 사용하여 이러한 이상치들을 강건하게(robustly) 제거한다. 이 알고리즘들은 무작위로 샘플링된 소수의 대응점들로부터 변환 행렬을 추정하고, 가장 많은 다른 대응점들(inlier)을 만족시키는 변환 행렬을 최적의 해로 선택하는 방식으로 동작한다.4

이 과정을 통해 얻어진 변환 행렬은 두 포인트 클라우드를 대략적으로 정렬하는 데 사용되며, 이후 ICP 알고리즘의 초기값으로 전달되어 최종적으로 정밀한 정합을 완료하게 된다.

### 2.2 핵심 파라미터 튜닝 가이드

FPFH의 성능은 몇 가지 핵심 파라미터에 크게 의존하며, 이들을 데이터의 특성에 맞게 신중하게 설정하는 것이 성공적인 정합의 관건이다. FPFH가 고정된 스케일의 반경을 사용하는 설계 방식을 채택했기 때문에 이러한 파라미터 민감성은 필연적인 약점으로 작용한다. 동일한 객체라도 포인트 클라우드의 밀도나 스케일이 다르면, 고정된 반경은 너무 적거나 너무 많은 이웃을 포함하여 의미 있는 특징을 추출하지 못할 수 있다.22 따라서 사용자는 데이터의 평균 점 간 거리 등을 고려하여 파라미터를 수동으로 튜닝해야 한다.23 이 수동 튜닝 과정은 FPFH 사용의 필수적인 부분이며, 알고리즘의 완전 자동화를 저해하는 요인이기도 하다. 이 때문에 지역적 밀도에 따라 반경을 자동으로 조절하는 적응형(adaptive) FPFH에 대한 연구가 활발히 진행되었다.9

특히, 법선 추정을 위한 반경(`Normal Radius`)과 FPFH 특징 계산을 위한 반경(`Feature Radius`) 사이의 관계는 매우 중요하다. 경험적으로 `Feature Radius`는 항상 `Normal Radius`보다 커야 한다.21 이는 법선이 매우 국소적인 평면 정보(1차 미분)를 나타내는 반면, FPFH는 그보다 넓은 영역의 곡률 변화(2차 미분과 유사)와 같은 기하학적 맥락을 포착해야 하기 때문이다. 더 넓은 반경을 통해 FPFH는 더 풍부하고 변별력 있는 특징을 생성할 수 있다.

#### 2.2.1 Table 2: FPFH 구현을 위한 핵심 파라미터 가이드

| 파라미터                      | PCL/Open3D 함수                                 | 역할                                                         | 튜닝 가이드 및 휴리스틱                                      | 관련 자료 |
| ----------------------------- | ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------- |
| **Voxel Size**                | `voxel_down_sample`                             | 다운샘플링의 해상도 결정. 계산 속도와 정밀도 간의 트레이드오프. | 데이터의 평균 밀도나 객체의 최소 특징 크기를 고려하여 설정. 너무 크면 특징이 소실되고, 너무 작으면 다운샘플링 효과가 미미함. | 19        |
| **Normal Radius**             | `estimate_normals` (search_param)               | 법선 벡터 추정 시 고려할 이웃의 반경.                        | Voxel Size보다 약간 크게 설정하는 것이 일반적 (예: `2 * voxel_size`). 너무 작으면 노이즈에 민감하고, 너무 크면 날카로운 모서리(edge)가 뭉개짐. | 19        |
| **Feature Radius**            | `compute_fpfh_feature` (search_param)           | FPFH 특징 계산 시 고려할 이웃의 반경.                        | **매우 중요:** 반드시 `Normal Radius`보다 커야 함. 일반적으로 `Normal Radius`의 2~5배 값을 시도함 (예: `5 * voxel_size`). 이 반경이 기술자의 표현력과 스케일을 결정함. | 21        |
| **RANSAC Iterations**         | `registration_ransac_based_on_feature_matching` | RANSAC 알고리즘의 최대 반복 횟수.                            | 값이 클수록 최적해를 찾을 확률이 높아지지만 계산 시간이 증가함. 대응점의 품질(inlier ratio)이 낮을수록 더 많은 반복이 필요. | 19        |
| **RANSAC Distance Threshold** | `registration_ransac_based_on_feature_matching` | RANSAC에서 두 점을 동일한 것으로 간주할 거리 임계값.         | Voxel Size에 비례하여 설정하는 것이 일반적 (예: `1.5 * voxel_size`). 데이터의 노이즈 수준에 따라 조절. | 19        |

### 2.3 주요 응용 분야 분석

FPFH의 빠른 속도와 준수한 표현력은 다양한 3D 비전 응용 분야에서 그 활용도를 높였다.

- **3D 객체 인식 및 정합 (3D Object Recognition and Registration):** FPFH의 가장 대표적인 응용 분야로, 소스 포인트 클라우드와 타겟 포인트 클라우드 간의 기하학적 특징을 매칭하여 두 데이터셋을 정렬한다. 이는 3D 모델 생성, 품질 검사, 역설계 등 산업 전반에 걸쳐 사용된다.4
- **6D 자세 추정 (6D Pose Estimation):** 로보틱스 분야에서 매우 중요한 문제로, 3D 모델(템플릿)과 실제 환경에서 스캔된 포인트 클라우드 간의 정합을 통해 특정 객체의 3차원 위치(x, y, z)와 3차원 방향(roll, pitch, yaw), 즉 6자유도(6DoF) 자세를 추정한다. FPFH는 이 과정에서 강건한 초기 정렬을 제공하는 데 사용된다.26
- **SLAM의 루프 클로저 (Loop Closure Detection in SLAM):** 자율 주행 차량이나 로봇이 이동하면서 지도를 작성할 때, 누적되는 오차로 인해 지도가 왜곡될 수 있다. 루프 클로저는 로봇이 이전에 방문했던 장소를 다시 인식하여 이 누적 오차를 보정하는 과정이다. FPFH와 같은 지역 기술자는 현재 프레임과 과거의 수많은 프레임 간의 기하학적 유사도를 빠르게 비교하여 루프(재방문) 여부를 판단하는 데 효과적으로 사용될 수 있다.3

### 2.4 PCL C++ 코드 예시 분석

Point Cloud Library(PCL)는 FPFH를 구현하고 사용하는 표준적인 방법을 제공한다. 다음은 일반적인 C++ 코드의 구조와 흐름이다.

먼저, 필요한 헤더 파일을 포함한다.

```C++
#include <pcl/io/pcd_io.h>
#include <pcl/features/normal_3d.h>
#include <pcl/features/fpfh.h>
#include <pcl/search/kdtree.h>
```

21

그 다음, 포인트 클라우드, 법선, FPFH 기술자를 저장할 객체를 생성한다.

```C++
pcl::PointCloud<pcl::PointXYZ>::Ptr cloud(new pcl::PointCloud<pcl::PointXYZ>);
pcl::PointCloud<pcl::Normal>::Ptr normals(new pcl::PointCloud<pcl::Normal>);
pcl::PointCloud<pcl::FPFHSignature33>::Ptr descriptors(new pcl::PointCloud<pcl::FPFHSignature33>());
```

21

워크플로우는 다음과 같이 진행된다.

1. **데이터 로딩:** PCD 파일로부터 포인트 클라우드를 로드한다.

   ```C++
   pcl::io::loadPCDFile<pcl::PointXYZ>("input.pcd", *cloud);
   ```

2. **법선 추정:** 법선 추정 객체를 설정하고 계산을 수행한다. 이웃 탐색을 위해 KD-Tree를 사용하고, 탐색 반경(`normal_radius`)을 설정한다.

   ```C++
   pcl::NormalEstimation<pcl::PointXYZ, pcl::Normal> normal_estimator;
   pcl::search::KdTree<pcl::PointXYZ>::Ptr tree(new pcl::search::KdTree<pcl::PointXYZ>());
   normal_estimator.setSearchMethod(tree);
   normal_estimator.setInputCloud(cloud);
   normal_estimator.setRadiusSearch(0.03); // 예: 3cm
   normal_estimator.compute(*normals);
   ```

3. **FPFH 추정:** FPFH 추정 객체를 설정한다. 입력으로 원본 포인트 클라우드와 계산된 법선을 모두 제공한다. **가장 중요한 점은 FPFH 계산을 위한 탐색 반경(`feature_radius`)을 법선 추정 시 사용한 반경보다 크게 설정해야 한다는 것이다**.21

   ```C++
   pcl::FPFHEstimation<pcl::PointXYZ, pcl::Normal, pcl::FPFHSignature33> fpfh_estimator;
   fpfh_estimator.setSearchMethod(tree);
   fpfh_estimator.setInputCloud(cloud);
   fpfh_estimator.setInputNormals(normals);
   fpfh_estimator.setRadiusSearch(0.05); // 예: 5cm (normal_radius보다 큼)
   fpfh_estimator.compute(*descriptors);
   ```

이 코드를 통해 `descriptors` 객체에는 `cloud`의 각 점에 해당하는 33차원 FPFH 벡터가 채워지게 되며, 이 기술자들을 후속 정합 단계에서 사용할 수 있다.

## 3. FPFH의 성능, 한계 및 개선 방안

FPFH는 3D 특징 기술자 분야에서 중요한 이정표를 세웠지만, 완벽한 기술은 아니며 명확한 성능적 특성과 내재적 한계를 지닌다. 이러한 한계를 이해하는 것은 FPFH를 효과적으로 사용하고, 더 나아가 개선된 기술을 개발하는 데 필수적이다.

### 3.1 성능 평가: 강건성 분석

FPFH의 성능은 주로 노이즈, 해상도 변화, 폐색 등 실제 스캔 환경에서 마주치는 여러 교란 요인에 대해 얼마나 강건한지로 평가된다.

- **노이즈에 대한 민감성:** FPFH의 계산 과정은 각 점의 법선 벡터 추정 결과에 크게 의존한다. 따라서 포인트 클라우드에 노이즈가 많을 경우, 법선 벡터의 방향이 불안정하게 추정되어 FPFH 기술자의 품질이 심각하게 저하된다. 한 연구에 따르면, 가우시안 노이즈의 표준편차가 2mm를 초과하기 시작하면 PFH와 FPFH의 올바른 대응점 매칭 비율이 급격히 감소하는 경향을 보였다.32 이는 FPFH가 일정 수준 이상의 노이즈에는 취약함을 시사한다.
- **해상도 변화에 대한 민감성:** FPFH는 고정된 이웃 탐색 반경을 사용하기 때문에 포인트 클라우드의 밀도, 즉 해상도 변화에 민감하다. 예를 들어, 다운샘플링으로 인해 포인트 클라우드의 밀도가 낮아지면, 동일한 반경 내에 포함되는 이웃 점의 수가 줄어들어 기술자가 충분한 기하학적 정보를 담지 못하게 된다. 한 벤치마크에서는 원본 데이터의 점 개수가 80% 미만으로 줄어들 때 대부분의 기술자 성능이 눈에 띄게 악화되었으며, 특히 FPFH는 SHOT이나 RoPS와 같은 다른 핸드크래프트 기술자에 비해 해상도 변화에 더 민감한 성능 저하를 보였다.32

### 3.2 내재적 한계점

FPFH의 설계 자체에서 비롯되는 근본적인 한계점들은 다음과 같다.

- **스케일 불변성 부재 (Lack of Scale Invariance):** FPFH의 가장 큰 한계점 중 하나는 스케일 불변성을 갖지 않는다는 것이다. FPFH는 고정된 절대값의 이웃 반경(예: 5cm)을 사용하여 지역 기하학을 기술한다. 따라서 동일한 형상의 객체라도 크기가 두 배가 되면, FPFH 기술자의 값 분포가 완전히 달라져 서로 다른 객체로 인식된다.8 이로 인해 CAD 모델과 실제 스캔 데이터 간의 스케일 차이가 있는 경우 등에서 FPFH를 직접 적용하기 어렵다.
- **불균일한 포인트 밀도에 대한 민감성 (Sensitivity to Non-uniform Point Density):** LiDAR나 깊이 카메라와 같은 실제 센서로 취득한 데이터는 일반적으로 스캐너로부터의 거리에 따라 포인트 밀도가 불균일하다. 가까운 곳은 밀도가 높고 먼 곳은 희소하다. 이러한 데이터에 고정된 반경을 사용하는 FPFH를 적용하면, 밀도가 높은 영역에서는 너무 많은 이웃이 포함되어 특징의 변별력이 떨어지고 계산이 비효율적이 되며, 밀도가 낮은 영역에서는 이웃이 부족하여 신뢰할 수 있는 특징을 생성하지 못하는 문제가 발생한다.4
- **기하학적 특징이 부족한 표면에서의 모호성 (Ambiguity on Featureless Surfaces):** 평면, 원통, 구와 같이 곡률 변화가 거의 없거나 반복적인 기하학적 구조를 가진 표면에서는 많은 점들이 서로 매우 유사한 FPFH 값을 갖게 된다. 이는 특징의 변별력을 크게 떨어뜨려 정합 과정에서 잘못된 대응점을 대량으로 생성하는 원인이 된다. 이 문제는 특히 평평한 벽이나 바닥이 많은 실내 환경에서 "미끄러짐(sliding)" 현상을 유발하는데, 이는 정합 알고리즘이 올바른 위치를 찾지 못하고 표면을 따라 미끄러지는 듯한 잘못된 결과를 내는 것을 의미한다.4

### 3.3 개선 연구 동향

이러한 FPFH의 한계를 극복하기 위해 다양한 개선 연구가 진행되었다.

- **적응형(Adaptive) FPFH:** 불균일한 밀도 문제에 대응하기 위한 접근법으로, 포인트 클라우드의 지역적 밀도를 추정한 후 그에 맞게 이웃 탐색 반경을 동적으로 조절하는 방식이다. 예를 들어, 한 연구에서는 포인트 클라우드의 밀도와 최적의 정합 성능을 내는 이웃 반경 사이의 관계를 함수로 학습하고, 새로운 데이터가 들어왔을 때 밀도를 측정하여 최적의 반경을 자동으로 선택하는 방법을 제안했다.9 또 다른 접근법은 여러 다른 스케일(반경)에서 특징을 계산하고 이를 종합하여 밀도 변화에 강건한 기술자를 만드는 것이다.35
- **스케일 불변(Scale-Invariant) FPFH:** FPFH에 스케일 불변성을 부여하려는 시도들이다. 대표적인 예로 **SIPFH(Scale-Invariant Points Feature Histogram)**가 있다.14 SIPFH는 2D 이미지 특징점인 SIFT의 스케일 공간(scale-space) 이론에서 영감을 얻었다. 먼저, 포인트 클라우드에서 스케일 변화에 불변하는 고유한 키포인트(keypoint)를 검출한다. 그 다음, 각 키포인트의 고유 스케일(characteristic scale)에 맞춰 이웃 반경을 정규화하여 FPFH를 계산하고, 여기에 스케일 정보를 추가로 인코딩하여 최종 기술자를 구성한다. 이 외에도 객체의 실루엣이나 경계선 정보를 인코딩하여 스케일 불변성을 확보하려는 연구도 존재한다.33

## 4. 딥러닝 시대의 FPFH: 비교 분석 및 미래 전망

2010년대 중반 이후, 딥러닝 기술이 3D 비전 분야에 도입되면서 3D 특징 기술자 분야는 근본적인 패러다임 전환을 맞이했다. FPFH와 같은 전통적인 '핸드크래프트(hand-crafted)' 방식과 새로운 '학습 기반(learned)' 방식의 비교를 통해 FPFH의 현재적 위치와 미래 역할을 조망할 수 있다.

### 4.1 패러다임의 전환: Hand-crafted vs. Learned Descriptors

- **Hand-crafted Descriptors:** FPFH, PFH, SHOT 등과 같은 전통적인 기술자들은 전문가가 사전에 정의한 기하학적 규칙(예: 법선 벡터 간의 각도, 점들 간의 거리 분포)에 기반하여 특징을 추출한다.36 이 방식의 장점은 알고리즘이 투명하고 직관적이며, 계산 과정을 예측할 수 있다는 점이다. 하지만 전문가의 직관에 의존하기 때문에, 실제 데이터에 존재하는 복잡하고 예측 불가능한 모든 종류의 변화(조명, 노이즈, 폐색 등)를 포착하는 데 한계가 있다.38
- **Learned Descriptors:** 3DMatch, FCGF, PPFNet 등과 같은 딥러닝 기반 기술자들은 대규모 데이터셋으로부터 딥러닝 모델(주로 CNN, Transformer 등)을 통해 최적의 특징 표현 자체를 '학습'한다.36 이 방식은 데이터에 내재된 복잡하고 미묘한 패턴을 스스로 학습할 수 있어, 핸드크래프트 방식보다 훨씬 높은 강건성과 변별력을 달성할 수 있다. 그러나 대규모의 정답(ground-truth) 데이터가 필요하고, 학습된 모델이 '블랙박스'처럼 동작하여 결과를 해석하기 어려우며, 높은 계산 자원(GPU)을 요구하는 단점이 있다.

### 4.2 벤치마크 기반 성능 비교

이러한 패러다임의 차이는 실제 벤치마크 결과에서 극명하게 드러난다. 3DMatch 벤치마크는 실제 RGB-D 센서로 촬영된 스캔 데이터를 사용하여 노이즈, 불완전함, 큰 시점 변화 등 현실적인 어려움을 포함하고 있어 기술자의 실용적인 성능을 평가하는 데 널리 사용된다.41

이 벤치마크에서의 성능 비교는 FPFH가 왜 더 이상 최첨단 기술로 간주되지 않는지를 명확히 보여준다. FPFH의 정합 재현율(Recall, 올바른 변환을 찾은 비율)은 약 44.2%에 그치는 반면, 초기 딥러닝 모델인 3DMatch는 66.8%, 그리고 더 발전된 모델인 FCGF(Fully Convolutional Geometric Features)는 85%를 상회하는 압도적인 성능을 기록했다.41

이러한 성능 차이는 단순히 알고리즘의 우열을 넘어, 특징 추출에 대한 철학의 근본적인 차이에서 비롯된다.

첫째, **맥락의 범위(Receptive Field)**가 다르다. FPFH는 고정된 작은 반경 내의 지역적 기하학만을 고려한다. 반면 FCGF와 같은 모델은 3D 희소 텐서(sparse tensor)에 최적화된 완전 컨볼루션 네트워크(Fully Convolutional Network, FCN) 구조를 사용하여, 훨씬 넓은 영역의 공간적 맥락을 효율적으로 통합하여 각 점의 특징을 계산한다.43 이는 특징이 더 풍부한 정보를 담게 하여 변별력을 높인다.

둘째, 학습 목표가 다르다. FPFH는 사전에 정의된 규칙에 따라 기하학을 기술할 뿐, 매칭 성능을 직접적으로 최적화하지 않는다. 반면, 3DMatch와 FCGF는 Contrastive Loss나 Triplet Loss와 같은 메트릭 러닝(metric learning) 손실 함수를 사용하여 학습 과정에서 명시적으로 '유사한 패치의 특징 벡터는 가깝게, 다른 패치의 특징 벡터는 멀게' 만들도록 특징 공간 자체를 구조화한다.45 이 과정은 매칭 작업에 훨씬 더 최적화된 특징 표현을 학습하게 한다.

결론적으로, FPFH의 성능 한계는 알고리즘 자체의 결함이라기보다는, '규칙 기반' 접근법이 대규모 데이터로부터 직접 '무엇이 중요한 특징인지'를 학습하는 '데이터 기반 학습' 접근법의 유연성과 표현력을 따라가기 어려운, 패러다임의 차이에서 기인하는 것이다.

#### 4.2.1 Table 3: 3D 지역 특징 기술자 성능 벤치마크 (3DMatch)

| 기술자 (Descriptor) | 방식 (Type)        | 정합 재현율 (Recall) | 정합 정확도 (Precision) | 주요 특징                                                    | 관련 자료 |
| ------------------- | ------------------ | -------------------- | ----------------------- | ------------------------------------------------------------ | --------- |
| **FPFH**            | Hand-crafted       | 44.2%                | 30.7%                   | 빠르고 간단한 기하학적 히스토그램                            | 41        |
| **Spin-Images**     | Hand-crafted       | 51.8%                | 31.6%                   | 2D 이미지로 지역 표면 표현                                   | 41        |
| **3DMatch**         | Learned (CNN)      | 66.8%                | 40.1%                   | TDF 볼륨 패치를 입력으로 사용하는 CNN 기반 기술자            | 41        |
| **FCGF**            | Learned (FCN)      | **~85%**             | **~80%**                | 3D 희소 텐서 기반의 완전 컨볼루션 네트워크, 매우 빠르고 정확함 | 42        |
| **PPFNet**          | Learned (PointNet) | -                    | -                       | Point Pair Feature를 학습에 활용, 원시 포인트 클라우드 직접 처리 | 40        |

*참고: 재현율 및 정확도 수치는 논문 및 평가 시점에 따라 다소 차이가 있을 수 있음.*

### 4.3 FPFH의 현재적 가치와 미래 역할

딥러닝 기술의 압도적인 성능에도 불구하고 FPFH가 여전히 연구 및 응용 분야에서 완전히 사라지지 않고 꾸준히 언급되는 이유는 그것이 제공하는 독자적인 가치, 즉 '계산 효율성'과 '단순성' 때문이다. FPFH의 역할은 최첨단 성능을 내는 '주력 기술자'에서, 더 복잡하고 강력한 후속 알고리즘을 위한 '효율적인 초기화 도구'로 전환되고 있다.

많은 고성능 정합 알고리즘, 특히 ICP나 딥러닝 기반의 정밀 정합 기법들은 좋은 초기 정렬 값에 크게 의존하며, 그렇지 않을 경우 쉽게 지역 최적해에 수렴하여 실패한다.6 전역적인 탐색을 통해 최적의 초기값을 찾는 것은 계산적으로 매우 비싸다. 바로 이 지점에서 FPFH가 빛을 발한다. FPFH는 딥러닝 모델처럼 방대한 학습 데이터나 고가의 GPU를 필요로 하지 않으며, 일반적인 CPU 환경에서도 실시간에 가까운 속도로 계산이 가능하다.4

FPFH와 RANSAC을 결합한 초기 정합은 완벽한 정렬을 제공하지는 못하지만, 후속 정밀 정합 단계가 성공적으로 수렴할 수 있는 '수렴 영역(convergence basin)' 내로 두 포인트 클라우드를 가져오기에는 충분한 정확도를 제공한다.4 따라서 현대의 복잡한 정합 파이프라인에서 FPFH는 최종 결과를 책임지는 주역이 아니라, 전체 시스템의 안정성과 효율성을 높이는 중요한 '전처리' 또는 '초기화' 모듈로서의 역할을 수행한다.18 이는 FPFH가 '최고'의 기술은 아닐지라도, 특정 상황에서는 '가장 실용적인 선택지 중 하나'로서 여전히 강력한 생명력을 유지하는 이유이다.

## 5. 결론: FPFH의 종합적 평가

Fast Point Feature Histogram (FPFH)은 3D 포인트 클라우드 처리 역사에서 중요한 위치를 차지하는 기술자이다. 딥러닝의 부상으로 인해 최첨단의 자리에서는 내려왔지만, 그 기술적 유산과 실용적 가치는 여전히 유효하다.

### 5.1 FPFH의 핵심 기여와 기술적 유산

FPFH의 가장 큰 기여는 그 전신인 PFH의 심각한 계산 복잡도 문제를 해결하고, 3D 특징 기반 정합을 실시간 응용이 가능한 영역으로 이끌었다는 점이다. 이는 속도와 표현력 사이의 영리한 공학적 트레이드오프를 통해 달성되었으며, 핸드크래프트 방식의 특징 기술자가 어떻게 실용성을 확보할 수 있는지를 보여주는 고전적인 성공 사례로 남아있다. 또한, PCL(Point Cloud Library)과 같은 주요 오픈소스 라이브러리에 기본적으로 포함됨으로써, 전 세계 수많은 연구자와 개발자들이 3D 비전 기술에 쉽게 접근하고 이를 기반으로 새로운 연구를 수행할 수 있는 발판을 마련했다.

### 5.2 실용적 관점에서의 장단점 종합

실용적인 관점에서 FPFH의 장단점은 명확하다.

- **장점:**
  - **빠른 계산 속도:** $O(nk)$의 복잡도로 대용량 포인트 클라우드에서도 비교적 빠르게 계산 가능하다.
  - **구현의 용이성:** 알고리즘의 원리가 직관적이고, PCL 등에서 잘 구현되어 있어 사용이 편리하다.
  - **학습 데이터 불필요:** 딥러닝 모델과 달리 사전에 대규모 데이터셋을 구축하고 학습하는 과정이 필요 없다.
- **단점:**
  - **파라미터 튜닝의 필요성:** 데이터의 특성(밀도, 스케일, 노이즈)에 맞춰 이웃 반경과 같은 핵심 파라미터를 수동으로 조절해야 최적의 성능을 낼 수 있다.
  - **스케일 및 밀도 변화에 대한 민감성:** 고정된 반경을 사용하므로 스케일과 밀도 변화에 취약하다.
  - **낮은 변별력:** 기하학적 특징이 부족한 평면 등에서는 변별력이 떨어져 모호한 매칭을 유발한다.
  - **성능 한계:** 딥러닝 기반 기술자들과 비교했을 때, 정확도와 강건성 측면에서 명백한 성능 한계를 보인다.

### 5.3 향후 3D 특징 기술자 연구 방향에 대한 제언

FPFH의 성공과 한계는 미래의 3D 특징 기술자 연구에 다음과 같은 방향을 제시한다.

1. **하이브리드 접근법 (Hybrid Approaches):** FPFH와 같은 전통적인 기하학적 통찰력과 딥러닝의 강력한 데이터 기반 표현력을 결합하는 연구가 유망하다. 예를 들어, 딥러닝 네트워크의 입력으로 FPFH와 같은 핸드크래프트 특징을 함께 사용하거나, 딥러닝 모델이 학습하기 어려운 기하학적 제약 조건을 핸드크래프트 방식으로 보완하는 하이브리드 모델은 데이터 효율적이면서도 강건한 성능을 보일 수 있다.
2. **자기 지도 학습 (Self-supervised Learning)의 활용:** 딥러닝 기술자의 가장 큰 장벽인 대규모 레이블링 데이터 문제를 해결하기 위해, 자기 지도 학습을 적극적으로 활용해야 한다. 데이터 자체의 내재적 속성을 이용하여(예: 패치 간의 상대적 위치 예측) 레이블 없이도 유용한 특징 표현을 학습함으로써, 기술자의 일반화 성능을 크게 향상시킬 수 있다.
3. **다중 모달리티 융합 (Multi-modal Fusion):** 순수한 기하학 정보만으로는 해결하기 어려운 모호성을 극복하기 위해, RGB 이미지의 색상 및 질감 정보나 LiDAR의 반사율(Intensity) 정보 등 다른 센서 데이터를 효과적으로 융합하는 연구가 중요하다.48 서로 다른 모달리티의 정보를 상호 보완적으로 활용하는 기술자는 훨씬 더 강건하고 변별력 높은 특징을 제공할 것이다.

결론적으로 FPFH는 3D 특징 기술자 발전사의 중요한 한 페이지를 장식한 알고리즘이다. 비록 현재 최고의 성능을 자랑하지는 않지만, 그 효율성과 단순성은 여전히 특정 응용 분야에서 유효하며, 그 한계점들은 차세대 기술자들이 해결해야 할 과제들을 명확히 제시해주고 있다.

**참고 자료**

1. hankyang94/teaser_fpfh_threedmatch_python - GitHub, accessed July 24, 2025, https://github.com/hankyang94/teaser_fpfh_threedmatch_python
2. Point Cloud Registration: Beyond the Iterative Closest Point Algorithm - Think Autonomous, accessed July 24, 2025, https://www.thinkautonomous.ai/blog/point-cloud-registration/
3. 3D LiDAR Slam Loop Closure Explained - Ignitarium, accessed July 24, 2025, https://ignitarium.com/3d-lidar-slam-loop-closure-explained/
4. Fast Point Feature Histograms (FPFH) for 3D Registration - 3D Vision Laboratory, accessed July 24, 2025, https://www.cvl.iis.u-tokyo.ac.jp/class2016/2016w/papers/6.3DdataProcessing/Rusu_FPFH_ICRA2009.pdf
5. Fast Point Feature Histograms (FPFH) for 3D Registration, accessed July 24, 2025, https://ciis.lcsr.jhu.edu/lib/exe/fetch.php?media=courses:446:2016:446-2016-06:dadler_fpfh_presentation_final.pdf
6. An Improved Large Planar Point Cloud Registration Algorithm - MDPI, accessed July 24, 2025, https://www.mdpi.com/2079-9292/13/14/2696
7. Point Feature Histograms (PFH) descriptors — Point Cloud Library 0.0 documentation, accessed July 24, 2025, https://pcl.readthedocs.io/projects/tutorials/en/master/pfh_estimation.html
8. PCL/OpenNI tutorial 4: 3D object recognition (descriptors) - robotica.unileon.es, accessed July 24, 2025, https://robotica.unileon.es/index.php?title=PCL/OpenNI_tutorial_4:_3D_object_recognition_(descriptors)
9. The New Fast Point Feature Histograms Algorithm Based on Adaptive Selection - Journal of Applied Science and Engineering, accessed July 24, 2025, http://jase.tku.edu.tw/articles/jase-202006-23-2-0006.pdf
10. Fast Point Feature Histograms (FPFH) descriptors — Point Cloud Library 0.0 documentation, accessed July 24, 2025, https://pcl.readthedocs.io/projects/tutorials/en/master/fpfh_estimation.html
11. Fast Point Feature Histograms (FPFH) for 3D Registration - Papers With Code, accessed July 24, 2025, https://paperswithcode.com/paper/fast-point-feature-histograms-fpfh-for-3d
12. Fast Point Feature Histograms (FPFH) for 3D registration | Request PDF - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/224557255_Fast_Point_Feature_Histograms_FPFH_for_3D_registration
13. The estimation of a FPFH feature for a 3D point. From left to right - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/figure/The-estimation-of-a-FPFH-feature-for-a-3D-point-From-left-to-right-a-point-p-green-is_fig2_224135376
14. Point cloud registration of arrester based on scale-invariant points feature histogram - PMC, accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9622705/
15. Research on 3D point cloud registration algorithm based on FPFH and ColorICP - SPIE Digital Library, accessed July 24, 2025, https://www.spiedigitallibrary.org/conference-proceedings-of-spie/13509/1350907/Research-on-3D-point-cloud-registration-algorithm-based-on-FPFH/10.1117/12.3058030.full
16. Illustration of Point Feature Histogram calculation. - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/figure/llustration-of-Point-Feature-Histogram-calculation_fig4_261262939
17. PPTFH: Robust Local Descriptor Based on Point-Pair Transformation Features for 3D Surface Matching - PMC, accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8124800/
18. A Pointcloud Registration Framework for Relocalization in Subterranean Environments, accessed July 24, 2025, https://arxiv.org/html/2504.07231v1
19. Gentle Introduction to Global Point Cloud Registration using ..., accessed July 24, 2025, https://medium.com/@amnahhmohammed/gentle-introduction-to-point-cloud-registration-using-open3d-pt-2-18df4cb8b16c
20. Point Cloud Registration Based on Fast Point Feature Histogram Descriptors for 3D Reconstruction of Trees - MDPI, accessed July 24, 2025, https://www.mdpi.com/2072-4292/15/15/3775
21. FPFH-PCL-Cpp - Tutorial - GitBook, accessed July 24, 2025, https://pcl.gitbook.io/tutorial/part-2/part02-chapter03/part02-chapter03-fpfh-pcl-cpp
22. A Tree Point Cloud Simplification Method Based on FPFH Information Entropy - MDPI, accessed July 24, 2025, https://www.mdpi.com/1999-4907/14/7/1507
23. On Fast Point Cloud Matching with Key Points and Parameter Tuning - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/339437737_On_Fast_Point_Cloud_Matching_with_Key_Points_and_Parameter_Tuning
24. PCL desciptors matching - point cloud library - Stack Overflow, accessed July 24, 2025, https://stackoverflow.com/questions/41224269/pcl-desciptors-matching
25. Improve CPD Performance Using FPFH Descriptors - MATLAB & Simulink - MathWorks, accessed July 24, 2025, https://www.mathworks.com/help/vision/ug/improve-cpd-performance-using-fpfh-descriptors.html
26. A 6D Object Pose Estimation Algorithm for Autonomous Docking with Improved Maximal Cliques - MDPI, accessed July 24, 2025, https://www.mdpi.com/1424-8220/25/1/283
27. Learning-based Point Cloud Registration for 6D Object Pose Estimation in the Real World, accessed July 24, 2025, https://www.researchgate.net/publication/359575440_Learning-based_Point_Cloud_Registration_for_6D_Object_Pose_Estimation_in_the_Real_World
28. 6D Pose Estimation for Flexible Production with Small Lot Sizes based on CAD Models using Gaussian Process Implicit Surfaces - mediaTUM, accessed July 24, 2025, https://mediatum.ub.tum.de/doc/1555279/cr7t1w2a65qkekhn3em4lm75c.gpis-gr.pdf
29. yiming107/l3d_loop_closure - GitHub, accessed July 24, 2025, https://github.com/yiming107/l3d_loop_closure
30. Loop closure detection using local 3D deep descriptors - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/355843359_Loop_closure_detection_using_local_3D_deep_descriptors
31. pcl/examples/features/example_fast_point_feature_histograms.cpp ..., accessed July 24, 2025, https://github.com/PointCloudLibrary/pcl/blob/master/examples/features/example_fast_point_feature_histograms.cpp
32. Robustness evaluation of feature descriptors. (a) Feature matching ..., accessed July 24, 2025, https://www.researchgate.net/figure/Robustness-evaluation-of-feature-descriptors-a-Feature-matching-results-with-respect_fig7_293330421
33. SIPF: Scale invariant point feature for 3D point clouds - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/277957351_SIPF_Scale_invariant_point_feature_for_3D_point_clouds
34. Fisher encoding of adaptive fast persistent feature histograms for partial retrieval of 3D pottery objects - Norwegian University of Science and Technology - NTNU, accessed July 24, 2025, https://www.ntnu.edu/documents/1327949769/1328352605/paper1015_Camera_Ready_Pottery_3DOR.pdf/10035cf5-22b5-1278-c9e7-11290c3025fe?t=1678724786900
35. Geometry and Colour Based Classification of Urban Point Cloud Scenes Using a Supervised Self - DGPF, accessed July 24, 2025, https://www.dgpf.de/pfg/2014/pfg2014_3_matti.pdf
36. arXiv:1802.02297v2 [cs.CV] 27 Jul 2020, accessed July 24, 2025, https://arxiv.org/pdf/1802.02297
37. From handcrafted to deep local features - Naver Labs Europe, accessed July 24, 2025, https://europe.naverlabs.com/wp-content/uploads/2019/12/From-handcrafted-to-deep-local-features.pdf
38. Deep Learning vs. Traditional Computer Vision - arXiv, accessed July 24, 2025, https://arxiv.org/pdf/1910.13796
39. (PDF) 3DMatch: Learning the Matching of Local 3D Geometry in Range Scans, accessed July 24, 2025, https://www.researchgate.net/publication/301855628_3DMatch_Learning_the_Matching_of_Local_3D_Geometry_in_Range_Scans
40. PPFNet: Global Context Aware Local Features for Robust 3D Point Matching - CVF Open Access, accessed July 24, 2025, https://openaccess.thecvf.com/content_cvpr_2018/CameraReady/1025.pdf
41. 3DMatch: Learning Local Geometric Descriptors from RGB-D Reconstructions, accessed July 24, 2025, https://3dmatch.cs.princeton.edu/
42. Fully Convolutional Geometric Features | Papers With Code, accessed July 24, 2025, https://paperswithcode.com/paper/fully-convolutional-geometric-features
43. Fully Convolutional Geometric Features - CVF Open Access, accessed July 24, 2025, https://openaccess.thecvf.com/content_ICCV_2019/papers/Choy_Fully_Convolutional_Geometric_Features_ICCV_2019_paper.pdf
44. Fully Convolutional Geometric Features | Request PDF - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/339559264_Fully_Convolutional_Geometric_Features
45. chrischoy/FCGF: Fully Convolutional Geometric Features - GitHub, accessed July 24, 2025, https://github.com/chrischoy/FCGF
46. Supplementary Materials for Fully Convolutional Geometric Features - Chris Choy, accessed July 24, 2025, https://node1.chrischoy.org/data/publications/fcgf/fcgf_supp.pdf
47. MatchU: Matching Unseen Objects for 6D Pose Estimation from RGB-D Images - arXiv, accessed July 24, 2025, https://arxiv.org/abs/2403.01517
48. 3DMatch: Learning Local Geometric Descriptors From RGB-D Reconstructions - CVF Open Access, accessed July 24, 2025, https://openaccess.thecvf.com/content_cvpr_2017/papers/Zeng_3DMatch_Learning_Local_CVPR_2017_paper.pdf


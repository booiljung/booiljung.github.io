# Pkd-tree에 대한 고찰

## 제1장: 서론: 다차원 데이터와 k-d 트리의 한계

### 0.1  다차원 공간 색인의 중요성

현대 컴퓨팅 환경은 다차원 데이터(multi-dimensional data)의 생성, 저장, 분석을 기반으로 작동한다. 머신러닝 모델의 특징 벡터, 컴퓨터 그래픽스의 3차원 좌표, 데이터베이스의 다중 속성 레코드, 과학 시뮬레이션의 입자 데이터 등은 모두 다차원 공간상의 점(point)으로 표현될 수 있다.1 이러한 데이터셋의 규모가 수십억 개를 넘어서는 빅데이터 시대로 접어들면서, 방대한 다차원 공간에서 원하는 데이터를 효율적으로 검색하고 분석하는 기술, 즉 다차원 공간 색인(indexing)의 중요성은 그 어느 때보다 커졌다.

다차원 색인 구조의 핵심 목표는 k-최근접 이웃(k-Nearest Neighbors, k-NN) 탐색, 직교 범위 질의(orthogonal range query), 범위 개수 세기(range count)와 같은 기본적인 질의 연산을 전수 조사(exhaustive search)보다 훨씬 빠르게 수행하는 것이다.1 예를 들어, 추천 시스템은 특정 사용자와 가장 유사한 취향을 가진 k명의 다른 사용자를 찾기 위해 k-NN 탐색을 사용하며, 지리 정보 시스템은 특정 사각 영역 내에 있는 모든 관심 지점을 찾기 위해 범위 질의를 사용한다. 이러한 연산의 효율성은 애플리케이션의 응답성과 확장성을 결정하는 핵심 요소이다.

### 0.2  전통적 k-d Tree의 원리 및 구조

1975년 존 벤틀리(Jon Bentley)에 의해 제안된 k-d tree(k-dimensional tree)는 다차원 공간 색인을 위한 가장 고전적이고 널리 사용되는 자료구조 중 하나이다.2 그 본질은 이진 탐색 트리(Binary Search Tree, BST)를 다차원 공간으로 확장한 이진 공간 분할(Binary Space Partitioning, BSP) 트리이다.5

k-d tree의 구축 과정은 재귀적으로 공간을 분할하는 방식으로 이루어진다. 각 내부 노드(non-leaf node)는 k개의 차원 중 하나를 '분할 차원(splitting dimension)'으로 선택하고, 해당 차원의 축에 수직인 초평면(hyperplane)을 기준으로 공간을 두 개의 반공간(half-space)으로 나눈다.5 노드에 저장된 점보다 분할 차원의 좌표값이 작은 모든 점들은 왼쪽 서브트리로, 큰 점들은 오른쪽 서브트리로 보내진다. 이 초평면의 위치, 즉 '분할 값(splitting value)'은 일반적으로 해당 노드에 속한 점들의 분할 차원 좌표값들 중 중앙값(median)으로 선택된다. 이는 트리의 깊이를 최소화하여 균형 잡힌 구조를 유지하려는 시도이다.8 분할 차원은 트리의 레벨(level)이 깊어짐에 따라 순환적으로 선택되는 것이 일반적이다(예: $x \to y \to z \to x \to \dots$). 이 과정을 더 이상 분할할 수 없을 때까지(예: 노드에 속한 점의 개수가 특정 임계값 이하일 때) 반복하면 k-d tree가 완성된다.

k-d tree는 선형적인 공간 복잡도($O(n)`)와 비교적 간단한 알고리즘을 가지며, 데이터 분포가 편향된 경우에도 강건하게 동작하는 등 여러 장점을 지닌다.2 이로 인해 수많은 분야에서 기초적인 다차원 색인 구조로 채택되어 왔다.

### 0.3  대용량 데이터 시대의 도전 과제: 병렬 처리 및 캐시 효율성

전통적인 k-d tree 알고리즘은 본질적으로 순차적(sequential)으로 설계되었다. 그러나 데이터의 양이 수십억, 수조 단위로 폭증함에 따라 단일 프로세서 코어의 연산 능력만으로는 이를 감당할 수 없게 되었다. 따라서 현대의 고성능 컴퓨팅 환경이 제공하는 멀티코어 프로세서의 잠재력을 최대한 활용하기 위한 병렬 처리(parallel processing)의 도입은 선택이 아닌 필수가 되었다.2 기존의 k-d tree 구현을 병렬화하려는 시도가 있었지만, 각 트리 레벨을 구축할 때마다 모든 프로세서 간의 동기화(synchronization)가 필요하고, 데이터 분할 과정에서 발생하는 부하 불균형 문제 등으로 인해 만족스러운 확장성을 얻기 어려웠다.

병렬성과 더불어 현대 컴퓨터 아키텍처에서 성능을 좌우하는 또 다른 핵심 요소는 캐시 효율성(cache-efficiency)이다. CPU의 연산 속도는 무어의 법칙에 따라 비약적으로 발전했지만, 메인 메모리(DRAM)의 접근 속도는 그에 미치지 못했다. 이 속도 차이로 인해 발생하는 병목 현상을 '메모리 장벽(memory wall)'이라고 부른다.1 프로세서는 연산할 데이터를 메인 메모리에서 직접 가져오는 대신, 훨씬 빠르지만 용량이 작은 캐시 메모리(cache memory)에 데이터를 미리 적재하여 사용한다. 만약 알고리즘이 필요한 데이터가 캐시에 존재하지 않으면(cache miss), CPU는 메인 메모리에서 데이터를 가져올 때까지 긴 시간 동안 대기해야 하므로 전체 성능이 급격히 저하된다. 따라서 효율적인 알고리즘은 데이터의 지역성(locality)을 극대화하여 캐시 미스를 최소화하도록 설계되어야 한다. 기존의 k-d tree 구축 알고리즘들은 데이터 전체를 반복적으로 스캔하는 경향이 있어 캐시 효율성이 좋지 못하며, 이는 대용량 데이터 처리 시 심각한 성능 저하의 원인이 된다.

### 0.4  고차원 데이터의 난제: '차원의 저주(Curse of Dimensionality)'

k-d tree가 직면한 또 다른 근본적인 문제는 '차원의 저주' 현상이다.12 이는 데이터의 차원($k$)이 증가함에 따라 발생하는 여러 가지 통계적, 기하학적 이상 현상을 총칭하는 용어이다. 공간 분할 트리의 관점에서 차원의 저주는 다음과 같은 문제로 나타난다.

첫째, 데이터의 희소성(sparsity)이다. 차원이 증가하면 공간의 부피(volume)는 차원 수에 따라 지수적으로 증가한다. 동일한 수의 데이터 포인트가 더 높은 차원의 공간에 뿌려지면, 점들 사이의 평균 거리는 멀어지고 데이터는 극도로 희소해진다.7 이로 인해 '가까운 이웃'이라는 개념 자체가 무의미해지며, 최근접 이웃 탐색의 변별력이 약화된다.

둘째, 탐색 공간 가지치기(pruning)의 비효율성이다. k-d tree의 탐색 효율은 질의점(query point)과 관련 없는 방대한 공간을 탐색 대상에서 제외하는 '가지치기' 능력에 달려있다. 그러나 고차원 공간에서는 질의점과 거의 모든 분할 초평면(splitting hyperplane) 사이의 거리가 매우 가까워지는 경향이 있다.7 이는 탐색 알고리즘이 트리의 양쪽 가지를 모두 방문해야 할 가능성을 높여, 결국 거의 모든 노드를 방문하게 만든다. 이러한 상황에서 k-d tree의 탐색 성능은 데이터를 처음부터 끝까지 모두 비교하는 전수 조사와 다를 바 없게 된다.15

이러한 이유로 k-d tree는 일반적으로 차원 수가 약 10에서 20을 초과하면 그 효율성을 급격히 잃는 것으로 알려져 있다.1 이는 k-d tree가 고차원 특징 벡터를 사용하는 현대 머신러닝 애플리케이션에 직접 적용되기 어렵게 만드는 근본적인 한계이다.

이처럼 k-d tree는 대용량 데이터 처리를 위한 확장성 문제와 고차원 데이터에서의 성능 저하라는 두 가지 상이한 도전에 직면해 있다. Pkd-tree는 전자인 병렬성 및 캐시 효율성 문제를 해결하기 위해 특화된 자료구조이며, PCA-tree와 같은 대안적 구조들은 후자인 차원의 저주 문제를 완화하기 위해 등장하였다. 이 두 가지 발전 방향은 서로 경쟁하는 관계라기보다는, k-d tree가 마주한 문제 공간의 다른 축을 해결하기 위한 상호 보완적인 진화로 이해할 수 있다. 본 보고서는 먼저 Pkd-tree의 설계 원리와 성능을 심층적으로 분석하고, 이어서 PCA-tree와 같은 대안적 접근법과의 비교를 통해 다차원 데이터 색인 기술의 전체적인 지형도를 조망하고자 한다.

## 1.  Pkd-tree의 설계 원리 및 핵심 기법

Pkd-tree(Parallel k-d tree)는 앞서 제기된 대용량 데이터 처리의 도전 과제, 즉 병렬성과 캐시 효율성 문제를 정면으로 해결하기 위해 제안된 고성능 인메모리(in-memory) k-d tree 구현체이다.2 이는 기존 알고리즘을 단순히 병렬화하는 수준을 넘어, 현대 하드웨어 아키텍처의 특성을 근본적으로 고려하여 알고리즘 자체를 재설계한 결과물이다. Pkd-tree의 핵심은 '작업(work)', '스팬(span)', '캐시 복잡도(cache complexity)'라는 세 가지 성능 척도를 동시에 최적화하는 데 있다.

### 1.1  설계 목표: 작업, 스팬, 캐시 복잡도의 동시 최적화

Pkd-tree의 우수성을 이해하기 위해서는 먼저 알고리즘의 성능을 평가하는 세 가지 핵심 척도에 대한 이해가 필요하다.2

1. **작업 (Work):** 알고리즘이 수행하는 총 연산의 양을 의미하며, 이는 프로세서가 하나일 때의 순차 실행 시간 복잡도에 해당한다. 작업량이 적을수록 알고리즘은 근본적으로 효율적이다.
2. **스팬 (Span):** 병렬 실행 환경에서 가장 긴 의존성을 가진 연산 경로의 길이를 의미한다. 이는 무한한 수의 프로세서가 있을 때의 실행 시간으로, 알고리즘이 얼마나 병렬화될 수 있는지를 나타내는 척도이다. 스팬이 작을수록 병렬성이 높다.
3. **캐시 복잡도 (Cache Complexity):** 알고리즘 실행 중 발생하는 캐시 미스(cache miss)의 횟수, 즉 메인 메모리와 캐시 메모리 간의 데이터 블록 전송 횟수를 의미한다. 캐시 복잡도가 낮을수록 메모리 계층 구조를 효율적으로 활용하여 '메모리 장벽' 문제를 완화할 수 있다.

기존의 많은 병렬 알고리즘은 이 세 가지 척도 중 일부를 개선하기 위해 다른 척도를 희생하는 경향이 있었다. 예를 들어, 병렬성을 높이기 위해(스팬을 줄이기 위해) 불필요한 연산을 추가하거나(작업량 증가), 통신 오버헤드를 발생시켜(캐시 복잡도 증가) 전체 성능이 저하되는 경우가 많았다. Pkd-tree의 설계 목표는 이 세 가지 척도를 희생 없이 동시에 최적화하여 이론적으로도 강력하고 실제 환경에서도 최고의 성능을 내는 k-d tree를 구현하는 것이다.2

### 1.2  샘플링 기반 병렬 구축 알고리즘

Pkd-tree의 가장 혁신적인 부분은 트리 구축 알고리즘에 있다. 전통적인 병렬 k-d tree 구축 방식은 트리 레벨마다 분할점을 찾고 데이터를 분할하는 과정을 반복하며, 각 레벨에서 전역적인 동기화가 필요해 병렬성의 한계가 명확했다. Pkd-tree는 '스켈레톤-체질(skeleton-sieving)'이라는 새로운 2단계 접근법을 통해 이 문제를 해결한다. 이 접근법의 핵심은 '연산의 지역화'와 '데이터 이동의 일괄 처리'라는 두 가지 원칙에 기반한다.

#### 1.2.1  스켈레톤(Skeleton) 구축: 연산의 지역화

첫 번째 단계는 트리의 상위 $\lambda$개 레벨을 한 번에 결정하는 것이다. 이는 전체 데이터에 대한 값비싼 연산을 캐시에 들어갈 만큼 작은 데이터 샘플에 대한 저비용 연산으로 대체하는 '연산의 지역화' 전략이다.

1. **샘플링 (Sampling):** 전체 입력 데이터 $P$에서 $2^{\lambda} \cdot \sigma$개의 포인트를 무작위로 균등하게 추출하여 샘플 집합 $S$를 생성한다. 여기서 $\lambda$는 한 번에 구축할 트리의 레벨 수(예: 6)이며, $\sigma$는 통계적 안정성을 보장하기 위한 과표집률(oversampling rate)이다.3
2. **스켈레톤 구축 (Skeleton Building):** 이 작은 샘플 집합 $S$를 이용하여 상위 $\lambda$ 레벨의 k-d tree를 구축한다. 이 상위 트리 구조를 '스켈레톤(skeleton)'이라고 부른다. $S$의 크기는 캐시에 상주할 수 있을 정도로 작게 유지되므로, 스켈레톤 구축 과정은 매우 적은 캐시 미스를 발생시키며 신속하게 완료될 수 있다.3
3. **공간 분할:** 완성된 스켈레톤은 전체 데이터 공간을 $2^{\lambda}$개의 서로 겹치지 않는 하위 공간으로 분할하는 $2^{\lambda}-1$개의 분할기(splitter) 집합 역할을 한다. 스켈레톤의 리프 노드(leaf node) 각각이 이 하위 공간에 해당하며, 이를 '버킷(bucket)'이라고 칭한다.3

이 방식을 통해, $\lambda$ 레벨에 걸쳐 발생했을 분할기 결정 연산이 단 한 번의 지역화된 연산으로 압축된다. 이는 전역적인 데이터 접근과 동기화 오버헤드를 획기적으로 줄여준다.

#### 1.2.2  체질(Sieving) 프로세스: 데이터 이동의 일괄 처리

스켈레톤이 구축되어 공간 분할 계획이 확정되면, 두 번째 단계는 전체 데이터 $P$의 모든 포인트를 각자에게 할당된 버킷으로 실제로 이동시키는 것이다. 이는 $\lambda$ 레벨에 걸쳐 점진적으로 일어났을 데이터 이동을 단 한 번의 대규모 병렬 재배치 작업으로 통합하는 '데이터 이동의 일괄 처리' 전략이다.

1. **버킷 할당:** $P$에 속한 각 포인트에 대해, 스켈레톤 트리를 탐색하여 해당 포인트가 어떤 버킷에 속하는지 결정한다. 이 탐색은 $O(\lambda)$의 시간 복잡도를 가지며, 모든 포인트에 대해 독립적으로 병렬 수행될 수 있다.3
2. **병렬 재배치:** 모든 포인트의 목적지 버킷이 결정되면, 이들을 동시에 재배치하여 동일한 버킷에 속한 포인트들이 메모리 상에서 연속적으로 위치하도록 배열 $P$를 재정렬한다. 이 과정에서 발생하는 핵심적인 문제는 각 포인트가 최종적으로 위치할 메모리 주소(오프셋)를 데이터 경쟁(data race)이나 잠금(lock) 없이 어떻게 병렬적으로 계산하느냐이다. Pkd-tree는 캐시 효율적인 병렬 정렬 알고리즘에서 사용되는 기법을 차용하여 이 문제를 해결한다.3 구체적으로, 데이터 배열을 여러 청크(chunk)로 나누고, 각 청크 내에서 버킷별 포인트 수를 센 뒤, 병렬 접두사 합(parallel prefix sum) 연산을 통해 각 포인트의 최종 오프셋을 한 번에 계산한다.
3. **재귀 호출:** 모든 포인트가 각자의 버킷으로 재배치되어 물리적으로 그룹화되면, 각 버킷에 대해 이 전체 구축 과정을 재귀적으로, 그리고 독립적으로 병렬 수행한다.3

이 체질 프로세스는 여러 번의 작은 메모리 접근을 한 번의 예측 가능한 대규모 접근으로 통합함으로써 데이터 지역성을 극대화하고 캐시 효율성을 비약적으로 향상시킨다.

### 1.3  재구축 기반 가중치-균형 업데이트

k-d tree는 AVL 트리나 레드-블랙 트리와 달리 노드의 균형을 맞추기 위한 회전(rotation)과 같은 국소적인 연산을 지원하지 않는다.2 따라서 동적인 데이터 삽입 및 삭제가 반복되면 트리가 심하게 불균형해져 탐색 성능이 저하될 수 있다. Pkd-tree는 이러한 문제를 '재구축 기반의 지연 전략(reconstruction-based lazy strategy)'을 통해 해결한다.

1. **지연 전략과 가중치 균형 인자 $\alpha$:** Pkd-tree는 약간의 불균형을 즉시 수정하지 않고 허용하는 지연 전략을 채택한다. 트리의 균형 상태는 '가중치-균형 인자(weight-balancing factor)' $\alpha$ (예: 0.3)를 통해 관리된다.3 어떤 노드의 두 자식 서브트리가 담고 있는 포인트의 개수 비율이 

   $\alpha$와 $1-\alpha$ 사이의 범위를 벗어날 때, 해당 노드는 불균형 상태로 간주된다.

2. **재균형을 위한 재구축:** 불균형이 감지되면, Pkd-tree는 해당 노드를 루트로 하는 서브트리 전체를 폐기하고, 그 서브트리에 속했던 모든 포인트를 모아 앞서 설명한 효율적인 병렬 구축 알고리즘을 사용하여 완전히 새로운 균형 잡힌 서브트리를 재구축(reconstruction)한다.2

3. **일괄 업데이트(Batch Updates):** 이 재구축 기반 방식은 특히 대량의 데이터를 한 번에 삽입하거나 삭제하는 일괄 업데이트에 효과적이다. 개별 업데이트마다 트리를 수정하는 대신, 업데이트 대상을 모아 한 번에 처리하고 필요한 부분만 재구축함으로써 전체적인 처리량을 극대화한다.10 업데이트 알고리즘의 효율성은 결국 구축 알고리즘의 효율성에 직접적으로 의존하게 되며, Pkd-tree의 빠른 구축 알고리즘이 빠른 업데이트 성능을 보장하는 기반이 된다.3

이 방식은 매 업데이트마다 전체 트리를 재구축하는 순진한 방법보다 훨씬 효율적이며, 상각 분석(amortized analysis)을 통해 우수한 성능이 보장된다.

## 2.  이론적 성능 분석

Pkd-tree의 설계가 얼마나 성공적인지는 작업, 스팬, 캐시 복잡도라는 세 가지 척도에 대한 이론적 분석을 통해 명확히 드러난다. Pkd-tree는 이 세 가지 지표 모두에서 강력한 이론적 한계치(strong theoretical bounds)를 달성한 최초의 k-d tree 구현 중 하나로 평가받는다.2

### 2.1  작업(Work), 스팬(Span), 캐시 복잡도 분석

#### 2.1.1  구축 알고리즘의 복잡도

$n$개의 점으로 구성된 $D$차원 데이터셋에 대해 Pkd-tree 구축 알고리즘은 높은 확률(with high probability)로 다음과 같은 복잡도를 가진다.2

- 작업 (Work): $O(n \log n)$

  이 복잡도는 비교 기반 정렬 알고리즘이 달성할 수 있는 이론적 하한과 동일하다. 즉, Pkd-tree 구축에 필요한 총 연산량은 데이터를 정렬하는 데 필요한 연산량과 점근적으로 같으며, 이는 최적의 수준임을 의미한다. 더 이상 개선하기 어려운 효율성을 달성한 것이다.

- 스팬 (Span): $O(\text{polylog } n)$

  스팬이 $n$에 대한 다항로그(polylogarithmic) 형태로 나타나는 것은 이 알고리즘이 극도로 높은 수준의 병렬성을 가지고 있음을 시사한다. 이는 프로세서 코어의 수가 증가함에 따라 실행 시간이 거의 이상적으로 단축될 수 있음을 의미하며, 대규모 병렬 머신에서의 뛰어난 확장성(scalability)을 보장한다.

- **캐시 복잡도 (Cache Complexity):**
  $$
  O\left(\frac{n}{B} \log_M n\right)
  $$
  여기서 $M$은 캐시의 크기(단어 수), $B$는 캐시 라인(블록)의 크기(단어 수)를 나타낸다. 이 복잡도는 캐시-인지(cache-aware) 정렬 알고리즘의 최적 캐시 복잡도와 일치한다. 이 수식이 갖는 의미는 매우 중요하다. 전통적인 알고리즘의 캐시 복잡도는 보통
  $$
  O\left(\frac{n}{B} \log_2 n\right)
  $$
  형태로 나타나는데, 이는 전체 데이터를 약 $\log_2 n$번 스캔해야 함을 의미한다. 반면 Pkd-tree의 복잡도에서 로그의 밑은 2가 아닌 $M$이다. $\log_M n = \frac{\log_2 n}{\log_2 M}$ 이므로, 이는 전통적인 알고리즘에 비해 데이터 스캔 횟수를 $\log_2 M$ 배만큼 줄였음을 의미한다. 현대 컴퓨터에서 캐시 크기 $M$은 수백만 단위에 이르므로, 이는 실질적으로 엄청난 성능 향상을 가져온다. 이는 제2장에서 설명한 '데이터 이동의 일괄 처리' 전략이 이론적으로 어떻게 최적의 캐시 효율성으로 이어지는지를 수학적으로 증명하는 것이다. Pkd-tree의 성공은 병렬성뿐만 아니라, 메모리 계층 구조에 대한 깊은 이해를 바탕으로 한 '캐시-인지' 설계에 크게 기인한다.

#### 2.1.2  업데이트 알고리즘의 상각(Amortized) 복잡도

Pkd-tree의 업데이트 연산은 특정 경우에 서브트리 전체를 재구축해야 하므로 최악의 경우(worst-case) 비용이 높을 수 있다. 그러나 가중치-균형 인자 $\alpha$를 이용한 지연 전략 덕분에, 연속적인 업데이트 연산에 대한 평균 비용, 즉 상각 복잡도(amortized complexity)는 매우 효율적이다. 일괄 업데이트의 상각 작업량은 대략
$$
O\left(\frac{|U|}{\alpha} \log n\right)
$$
에 비례하는 형태로 나타나며, 여기서 $|U|$는 업데이트되는 점의 개수이다.17 이는 매번 전체 트리를 재구축하는 것보다 훨씬 효율적이며, $\alpha$ 값을 통해 성능과 트리 균형 상태 간의 트레이드오프를 조절할 수 있음을 보여준다.

### 2.2  이론적 보장과 실제 성능의 관계

알고리즘의 이론적 복잡도가 항상 실제 성능으로 직결되는 것은 아니다. 점근적 표기법($O(\cdot)$)에 숨겨진 상수 인자나 실제 하드웨어의 복잡한 특성 때문이다. 그러나 Pkd-tree의 경우, 강력한 이론적 보장이 실제 환경에서의 우수한 성능으로 일관되게 이어진다는 점이 여러 실험을 통해 입증되었다.2

그 이유는 Pkd-tree의 설계가 추상적인 계산 모델이 아닌, 현대 멀티코어 프로세서와 메모리 계층 구조라는 실제 컴퓨팅 환경의 병목 현상을 직접적으로 겨냥하고 있기 때문이다. 작업 복잡도를 최적화하여 불필요한 연산을 줄이고, 스팬을 최소화하여 병렬 처리 능력을 극대화하며, 캐시 복잡도를 최적화하여 데이터 이동 비용을 최소화하는 Pkd-tree의 다각적인 최적화 전략은 이론적 우아함과 실용적 효율성을 동시에 달성하는 성공적인 사례라 할 수 있다. 따라서 Pkd-tree의 이론적 분석은 단순한 학술적 증명을 넘어, 실제 시스템에서 왜 이 자료구조가 빠를 수밖에 없는지에 대한 근본적인 이유를 설명해준다.

## 3.  실험적 성능 평가 및 비교 분석

이론적 우수성은 실제 데이터와 경쟁 알고리즘과의 비교를 통해 그 가치가 입증된다. Pkd-tree는 다양한 합성 및 실제 데이터셋을 대상으로 한 광범위한 실험에서 기존의 최첨단 병렬 k-d tree 구현체들을 압도하는 성능을 보여주었다.2 본 장에서는 Pkd-tree의 실험적 성능을 구축, 업데이트, 질의 연산으로 나누어 분석하고, 더 나아가 Ball-tree, LSH와 같은 다른 고차원 색인 기법들과의 비교를 통해 그 위치를 조망한다.

### 3.1  벤치마크 환경

Pkd-tree의 성능 평가는 알고리즘의 강건성(robustness)을 검증하기 위해 신중하게 설계된 환경에서 수행되었다.

- **데이터셋:** 실험에는 데이터 분포가 성능에 미치는 영향을 평가하기 위해 균일 분포(uniform) 데이터셋과 데이터가 특정 영역에 집중된 고도로 편향된(highly skewed) 데이터셋이 모두 사용되었다. 또한, 실제 애플리케이션 환경을 모사하기 위해 다양한 실제 데이터(real-world datasets)도 활용되었다.2
- **비교 대상:** Pkd-tree의 성능은 기존의 대표적인 병렬 k-d tree 구현체들과 비교되었다. 주요 비교 대상으로는 로그 구조를 이용하여 동적 업데이트를 지원하는 **Log-tree**, 또 다른 병렬 k-d tree 구현체인 **BHL-tree**, 그리고 기하학적 알고리즘 분야에서 널리 사용되는 고품질 라이브러리인 **CGAL(Computational Geometry Algorithms Library)**의 k-d tree 구현이 포함되었다.17 이러한 비교 대상들은 당시 최첨단(state-of-the-art) 기술을 대표하며, Pkd-tree의 성능 향상 폭을 객관적으로 측정하는 기준이 되었다.

### 3.2  구축 및 업데이트 성능 비교 분석

실험 결과에 따르면, Pkd-tree는 트리 구축과 일괄 업데이트 연산 모두에서 모든 비교 대상보다 일관되게 그리고 월등히 빠른 성능을 보였다.2

- **구축 성능:** Pkd-tree의 구축 속도는 다른 구현체들에 비해 수십 배에서 수백 배까지 빠른 것으로 나타났다. 이는 제2장에서 설명한 '스켈레톤-체질' 기반 구축 알고리즘의 압도적인 효율성을 실증적으로 보여주는 결과이다. 상위 레벨의 분할기를 작은 샘플 집합만으로 결정하고, 전체 데이터의 이동을 단 한 번의 캐시 효율적인 병렬 작업으로 처리하는 전략은 이론적 예측대로 실제 성능 향상에 결정적인 역할을 했다.
- **업데이트 성능:** 일괄 삽입 및 삭제 성능에서도 Pkd-tree는 다른 알고리즘들을 크게 앞섰다. 이는 재균형(rebalancing)이 필요할 때 영향을 받는 서브트리만 효율적으로 재구축하는 Pkd-tree의 전략이, 전체 구조를 점진적으로 수정하거나 여러 개의 트리를 관리하는 다른 방식들보다 훨씬 효율적임을 의미한다. 특히, Pkd-tree의 업데이트 알고리즘은 빠른 구축 알고리즘을 서브루틴으로 재사용하기 때문에, 구축 성능의 우위가 업데이트 성능의 우위로 자연스럽게 이어진다.

### 3.3  질의(Query) 성능 비교

알고리즘이 구축 및 업데이트 속도를 위해 질의 성능을 희생하는 경우는 흔하다. 그러나 Pkd-tree는 k-NN 탐색, 범위 개수, 범위 보고 등 다양한 유형의 질의에서 비교 대상들과 대등하거나 더 나은 성능을 기록했다.2 이는 Pkd-tree가 빠른 구축 속도에도 불구하고 생성된 트리의 품질이 매우 높다는 것을 의미한다. 가중치-균형 인자 

$\alpha$를 통해 트리의 균형을 일정 수준 이상으로 유지하는 전략이 효과적으로 작동하여, 탐색 시 불필요한 노드 방문을 최소화하고 효율적인 가지치기를 가능하게 한다. 결과적으로 Pkd-tree는 구축, 업데이트, 질의라는 세 가지 핵심 연산 모두에서 균형 잡힌 고성능을 달성한 이상적인 자료구조임이 실험적으로 입증되었다.

### 3.4  고차원 색인 기법과의 정성적/정량적 비교

Pkd-tree는 k-d tree의 아키텍처를 병렬 및 캐시 효율적으로 극대화한 구현체이지만, k-d tree가 가진 근본적인 한계, 즉 '차원의 저주'에서는 자유롭지 못하다. 따라서 고차원 데이터 색인 문제에 대해서는 다른 접근 방식을 사용하는 기술들과의 비교를 통해 Pkd-tree의 적합한 사용 영역을 파악하는 것이 중요하다. 대표적인 대안으로는 Ball-tree와 지역성 민감 해싱(Locality-Sensitive Hashing, LSH)이 있다.

- **Ball-tree:** k-d tree가 축에 정렬된 초평면으로 공간을 분할하는 것과 달리, Ball-tree는 데이터를 초구(hyper-sphere)에 담아 계층적으로 분할한다.12 초구는 특정 축에 종속되지 않으므로 데이터의 분포 형태에 더 유연하게 적응할 수 있어, k-d tree보다 고차원 공간에서 더 나은 성능을 보이는 경향이 있다.21 하지만 각 노드에서 최적의 초구를 찾는 과정은 계산 비용이 더 높고, 저차원에서는 오히려 k-d tree보다 비효율적일 수 있다.23
- **지역성 민감 해싱 (LSH):** LSH는 앞선 두 트리 기반 구조와 근본적으로 다른 접근법을 취한다. 정확한 최근접 이웃을 찾는 것을 포기하는 대신, 높은 확률로 근접 이웃을 찾아주는 근사(approximate) 알고리즘이다.16 LSH는 특수하게 설계된 해시 함수들을 사용하여, 원본 공간에서 가까운 점들이 해싱 후에도 동일한 버킷에 들어갈 확률이 높도록 만든다. 이 방식은 차원의 저주에 매우 강건하여 수백, 수천 차원의 데이터에서도 효과적으로 동작하지만, 결과의 정확성을 보장하지 않으며 해시 함수군 선택 및 파라미터 튜닝이 성능에 큰 영향을 미친다는 단점이 있다.21

아래 표는 이 세 가지 기법의 핵심적인 특징과 트레이드오프를 요약한 것이다.

**표 1: 주요 다차원 색인 기법 비교**

| 기법 (Technique) | 핵심 원리 (Core Principle)                         | 분할 방식 (Partitioning Method)          | 정확성 (Accuracy)  | 고차원 성능 (High-D Perf.) | 장점 (Pros)                                                  | 단점 (Cons)                                                 |
| ---------------- | -------------------------------------------------- | ---------------------------------------- | ------------------ | -------------------------- | ------------------------------------------------------------ | ----------------------------------------------------------- |
| **Pkd-tree**     | k-d tree의 고성능 병렬/캐시-인지 구현              | 축-정렬 초평면 (Axis-aligned Hyperplane) | 정확 (Exact)       | 낮음 (Poor)                | 저-중차원에서 매우 빠름, 병렬 확장성, 캐시 효율성, 정확한 결과 보장 2 | 차원의 저주에 취약 7                                        |
| **Ball-tree**    | 데이터를 초구(Hyper-sphere)에 담아 계층적으로 분할 | 초구 경계 (Spherical Boundary)           | 정확 (Exact)       | 중간 (Moderate)            | k-d tree보다 고차원에 강함, 다양한 거리 척도 지원 12         | 구축 비용이 높고, 저차원에서는 k-d tree보다 느릴 수 있음 23 |
| **LSH**          | 근접한 데이터를 높은 확률로 동일한 버킷에 해싱     | 해시 함수 기반 확률적 분할               | 근사 (Approximate) | 높음 (Good)                | 차원의 저주에 매우 강건함, 대규모 데이터셋에서 빠른 속도 16  | 정확한 결과를 보장하지 않음, 파라미터 튜닝이 민감함 21      |

결론적으로, Pkd-tree는 저-중(low-to-medium) 차원의 대용량 데이터에 대해 최고의 성능을 제공하는 '정확한(exact)' 탐색 알고리즘이다. 반면, 데이터의 차원이 매우 높아져 k-d tree의 성능이 저하되는 영역에서는 Ball-tree나 LSH와 같은 대안적 접근법이 더 적합할 수 있다. 특히, 약간의 정확성을 희생하더라도 속도를 극대화해야 하는 애플리케이션에서는 LSH가 강력한 선택지가 된다.

## 4.  고차원 데이터 색인을 위한 대안적 접근: PCA-tree

Pkd-tree가 대용량 데이터 처리를 위한 확장성 문제를 해결했다면, k-d tree 계열이 가진 또 다른 근본적 한계인 '차원의 저주' 문제는 어떻게 완화할 수 있을까? 이 질문에 대한 답은 k-d tree의 가장 기본적인 가정, 즉 '축-정렬 분할(axis-aligned split)'의 한계를 극복하는 것에서 시작된다. PCA-tree(Principal Component Analysis Tree)는 데이터의 내재적 구조를 적극적으로 활용하여 분할 방향을 결정함으로써 이 문제를 해결하려는 시도이다.

### 4.1  축-정렬 분할의 한계와 주성분 방향(Principal Direction)

k-d tree의 분할 초평면은 항상 데이터 공간의 좌표축 중 하나에 평행하다.27 이는 분할 기준을 결정하는 연산이 매우 간단하고 빠르다는 장점을 가지지만, 데이터의 분포 형태나 변수 간의 상관관계를 전혀 고려하지 못한다는 치명적인 단점을 낳는다. 예를 들어, 데이터가 좌표축에 대해 45도 기울어진 가늘고 긴 타원 형태로 분포한다고 상상해보자. k-d tree는 이 분포를 효율적으로 나누기 위해 수많은 작은 직사각형들로 잘게 쪼개야만 한다. 이는 트리의 깊이를 불필요하게 증가시키고 탐색 효율을 저하시킨다.

만약 데이터가 가장 길게 늘어선 방향, 즉 데이터의 분산(variance)이 가장 큰 방향을 찾아 그 방향에 수직으로 공간을 분할할 수 있다면 훨씬 적은 분할로도 데이터를 효과적으로 나눌 수 있을 것이다. 주성분 분석(Principal Component Analysis, PCA)은 바로 이러한 '데이터의 분산이 가장 큰 방향'을 찾는 통계적 기법이다.28 PCA를 통해 찾은 방향들을 주성분 방향(principal directions) 또는 주축(principal axes)이라고 부른다.

### 4.2  PCA-tree의 구축 및 탐색 원리

PCA-tree는 k-d tree의 축-정렬 분할 규칙을 PCA 기반의 데이터-적응적(data-adaptive) 분할 규칙으로 대체한 자료구조이다.

- **구축 과정:** PCA-tree의 구축은 각 노드에서 다음과 같은 과정을 재귀적으로 수행한다.27
  1. **PCA 수행:** 현재 노드에 할당된 데이터 포인트들의 부분집합에 대해 PCA를 수행한다. 구체적으로는 데이터의 공분산 행렬(covariance matrix)을 계산하고, 이 행렬의 고유값 분해(eigendecomposition)를 통해 고유값(eigenvalue)과 고유벡터(eigenvector)를 찾는다.
  2. **분할 방향 결정:** 가장 큰 고유값에 해당하는 고유벡터가 바로 데이터의 분산이 가장 큰 방향, 즉 첫 번째 주성분(PC1)이다. PCA-tree는 이 첫 번째 주성분 벡터를 분할 초평면의 법선 벡터(normal vector), 즉 분할 방향으로 사용한다.
  3. **분할 값 결정 및 분할:** 노드에 속한 모든 데이터 포인트를 이 주성분 벡터에 투영(project)시킨다. 그 후, 투영된 값들의 중앙값(median)을 분할 값으로 결정하여, 이 값을 기준으로 데이터를 두 개의 부분집합으로 나눈다. 이 분할을 통해 생성된 초평면은 더 이상 특정 좌표축에 평행하지 않은, 데이터 분포에 최적화된 방향을 갖게 된다.27
- **탐색 과정:** PCA-tree에서의 최근접 이웃 탐색은 k-d tree의 탐색 알고리즘과 구조적으로 유사하다. 질의점(query point)이 주어지면 루트 노드부터 시작하여 트리를 따라 내려간다. 각 노드에서, 질의점을 해당 노드의 분할 방향(주성분 벡터)에 투영한 뒤, 그 값을 노드의 분할 값과 비교하여 왼쪽 또는 오른쪽 자식 노드로 이동한다. 리프 노드에 도달하면 해당 노드의 점을 현재까지의 최근접 이웃 후보로 삼고, 다시 부모 노드로 거슬러 올라가며(backtracking) 반대편 서브트리를 탐색할 필요가 있는지 검사한다. 이 검사는 질의점과 분할 초평면 사이의 거리가 현재까지 찾은 최근접 이웃과의 거리보다 가까운지를 확인하는 방식으로 이루어진다.30
- **APD-tree (Approximate Principal Direction Tree):** PCA-tree의 가장 큰 단점은 각 노드에서 PCA를 수행하는 계산 비용이 매우 높다는 점이다.27 특히 데이터의 차원이나 개수가 많을 경우 이는 심각한 성능 병목이 될 수 있다. APD-tree는 이 문제를 해결하기 위해 제안된 변형이다. APD-tree는 완전한 PCA를 수행하는 대신, 임의의 랜덤 벡터에서 시작하여 고유벡터를 근사적으로 찾는 알고리즘인 파워 메소드(power method)를 단 몇 차례만 반복 적용하여 주성분 방향을 '근사'한다. 이 근사된 방향을 분할 기준으로 사용함으로써, PCA-tree와 유사한 수준의 분할 품질을 훨씬 빠른 계산 속도로 달성할 수 있다.27

### 4.3  Pkd-tree와의 비교: 내재적 차원(Intrinsic Dimension) 적응성

PCA-tree가 k-d tree(그리고 그 고성능 버전인 Pkd-tree)보다 '차원의 저주'에 더 강건한 근본적인 이유는 '내재적 차원 적응성(adaptability to intrinsic dimension)'에 있다.

많은 고차원 데이터는 비록 높은 차원의 주변 공간(ambient space, $D$차원)에 표현되지만, 실제로는 그보다 훨씬 낮은 차원의 매니폴드(manifold) 위에 놓여 있거나 그 근처에 분포하는 경우가 많다. 이 매니폴드의 차원을 데이터의 '내재적 차원(intrinsic dimension, $d$)'이라고 한다.32

k-d tree의 성능은 데이터의 내재적 구조와 무관하게 주변 공간의 차원 $D$에 크게 의존한다. 데이터가 내재적으로 1차원 직선 형태를 띠더라도, 그 직선이 100차원 공간에 놓여 있다면 k-d tree는 100차원 데이터로 간주하고 비효율적인 축-정렬 분할을 수행한다.35

반면, PCA-tree는 각 분할 단계에서 데이터의 분산이 가장 큰 방향을 찾는다. 만약 데이터가 저차원 매니폴드 근처에 분포한다면, 데이터의 분산은 대부분 그 매니폴드를 따라 나타날 것이다. 따라서 PCA-tree가 찾는 주성분 방향은 자연스럽게 데이터의 내재적 구조를 반영하게 된다. 이로 인해 PCA-tree의 분할 효율과 탐색 성능은 주변 공간의 차원 $D$보다는 내재적 차원 $d$에 더 크게 의존하게 된다.34 이것이 PCA-tree가 고차원 데이터에 대해 더 강건한 성능을 보이는 핵심 원리이다.

이러한 공간 분할 트리의 발전 과정은 '데이터에 무관한(data-oblivious)' 분할 방식에서 점차 '데이터에 적응적인(data-adaptive)' 방식으로 진화해왔음을 보여준다. 고정된 격자를 사용하는 그리드 파일(Grid File)에서 시작하여, 데이터의 밀도에 반응하는 k-d tree를 거쳐, 데이터의 형태와 상관관계까지 고려하는 PCA-tree에 이르기까지, 이 진화는 데이터로부터 더 많은 정보를 추출하여 더 효율적인 분할을 만들려는 노력의 결과이다. Pkd-tree는 k-d tree의 '밀도-적응적' 분할 방식을 하드웨어 수준에서 극단적으로 최적화한 정점이며, PCA-tree는 '형태-적응적' 분할이라는 새로운 차원을 연 기술이라 할 수 있다. 물론 이러한 적응성에는 대가가 따른다. 데이터에 더 많이 적응할수록(PCA-tree), 각 노드에서의 분할 결정 비용은 증가한다. 따라서 최적의 자료구조 선택은 데이터의 크기, 차원, 내재적 구조, 그리고 가용한 연산 자원 간의 복잡한 트레이드오프를 고려하여 이루어져야 한다.

## 5.  응용 분야 및 결론

Pkd-tree와 그와 관련된 공간 분할 트리 기술들은 이론적 흥미를 넘어 다양한 실제 응용 분야에서 핵심적인 역할을 수행하고 있다. 본 장에서는 이들 기술의 주요 활용 사례를 살펴보고, Pkd-tree가 갖는 학술적, 실용적 의의를 종합하며, 향후 연구 방향을 제언하며 보고서를 마무리한다.

### 5.1  주요 응용 분야

Pkd-tree와 같은 효율적인 다차원 색인 구조는 대규모 포인트 클라우드 데이터를 신속하게 처리해야 하는 모든 분야에서 그 가치를 발휘한다.

- **머신러닝 (Machine Learning):** k-NN 알고리즘은 가장 대표적인 비모수적(non-parametric) 머신러닝 기법으로, 분류(classification)와 회귀(regression) 문제에 널리 사용된다. k-NN의 성능은 최근접 이웃을 얼마나 빨리 찾느냐에 달려있기 때문에, Pkd-tree와 같은 고속 탐색 구조는 필수적이다. 또한, DBSCAN과 같은 밀도 기반 클러스터링 알고리즘에서도 각 점의 이웃을 찾는 과정이 핵심 연산이므로 Pkd-tree를 통해 성능을 크게 향상시킬 수 있다.1 Pkd-tree의 빠른 구축 및 일괄 업데이트 능력은 대규모 데이터셋에 대한 모델의 재학습 및 동적 데이터 스트림 처리를 효율적으로 만든다.

- **컴퓨터 그래픽스 (Computer Graphics):** 사실적인 이미지 렌더링에 사용되는 광선 추적(Ray Tracing) 기술은 광선과 장면 내의 객체 간의 교차점을 찾는 연산이 대부분을 차지한다. 수십억 개의 입자(particle)로 구성된 과학 시뮬레이션 데이터를 시각화할 때, 각 광선에 대해 가장 가까운 입자를 찾는 것은 엄청난 계산량을 요구한다. Pkd-tree는 이러한 대규모 입자 데이터셋에 대한 가속 구조(acceleration structure)로 사용되어 렌더링 시간을 획기적으로 단축시킬 수 있다.1

- 응용 사례 심층 분석: 두 종류의 P-k-d Tree:

  연구 자료들을 분석하는 과정에서 'P-k-d tree'라는 용어가 서로 다른 두 가지 맥락에서 사용되고 있음이 확인되었다. 이 둘을 명확히 구분하는 것은 기술의 정확한 이해를 위해 매우 중요하다.

  1. **Parallel k-d tree (Men et al.):** 본 보고서에서 주로 다룬 Pkd-tree는 Ziyang Men 연구팀이 제안한 것으로, 이름 그대로 **병렬성(Parallelism)**과 캐시 효율성에 초점을 맞춘 범용 다차원 데이터 관리 구조이다.2 이 구현의 목표는 최신 멀티코어 아키텍처를 최대한 활용하여 트리 구축, 업데이트, 질의 등 모든 연산의 속도를 극대화하는 것이다. 이는 데이터베이스, 머신러닝 등 일반적인 고성능 컴퓨팅 분야를 대상으로 한다.
  2. **Particle k-d tree (Wald et al.):** Ingo Wald 연구팀이 OSPRay 시각화 프레임워크에서 구현한 P-k-d tree는 **입자(Particle)** 데이터 렌더링에 특화된 변형이다.38 이 기술의 핵심적인 특징은 추가적인 메모리 오버헤드를 거의 발생시키지 않는다는 점이다. 기존의 입자 데이터 배열을 제자리에서(in-place) 재귀적으로 재배열하여 k-d tree의 논리적 구조를 구현한다. 이는 수십억 개 이상의 입자를 다루어 메모리 용량이 극도로 제한되는 대규모 과학 시뮬레이션의 실시간 인시튜(in-situ) 시각화와 같은 특정 응용 분야에 최적화된 설계이다.38

- **기타 분야:** 이 외에도 Pkd-tree와 같은 기술은 로봇 공학에서의 센서 데이터 처리 및 경로 탐색, 레이더 신호 처리, 계산 기하학 문제 해결, 그리고 다양한 과학 및 공학 시뮬레이션 결과 분석 등 다차원 데이터를 다루는 광범위한 분야에서 활용된다.1

### 5.2  Pkd-tree의 학술적/실용적 의의

Pkd-tree는 다음과 같은 측면에서 중요한 학술적, 실용적 의의를 가진다.

- **학술적 의의:** Pkd-tree는 k-d tree라는 고전적인 자료구조를 현대적인 병렬 컴퓨팅 패러다임과 메모리 계층 구조의 관점에서 성공적으로 재해석했다. 작업, 스팬, 캐시 복잡도라는 세 가지 핵심 척도를 동시에 최적화하는 알고리즘을 제안하고 그 이론적 한계치를 증명함으로써, 병렬 자료구조 설계 분야에 새로운 기준을 제시했다. 이는 단순한 성능 개선을 넘어, 하드웨어의 특성을 깊이 이해하고 이를 알고리즘 설계에 반영하는 것이 얼마나 중요한지를 보여주는 대표적인 사례이다.
- **실용적 의의:** Pkd-tree 연구팀은 자신들의 구현을 오픈소스로 공개함으로써 2, 학계와 산업계의 연구자들이나 개발자들이 손쉽게 고성능 다차원 색인 기술을 활용할 수 있도록 했다. 이는 대용량 다차원 데이터 처리가 필요한 다양한 실제 애플리케이션의 성능을 직접적으로 향상시키는 데 기여하며, 새로운 기술 개발을 촉진하는 실질적인 발판이 된다.

### 5.3  향후 연구 방향 및 제언

Pkd-tree는 중요한 성과를 이루었지만, 다차원 데이터 색인 분야는 여전히 많은 도전 과제와 연구 기회를 남겨두고 있다.

- **다른 공간 분할 트리로의 확장:** Pkd-tree의 핵심 아이디어인 '스켈레톤-체질' 기반 병렬 구축 방식은 k-d tree에만 국한되지 않는다. 이 아이디어를 Quad-tree, Oct-tree, R-tree와 같은 다른 공간 분할 트리에 적용하여 이들의 병렬 고성능 버전을 개발하는 연구는 유망한 방향이 될 수 있다.
- **새로운 하드웨어 아키텍처 적응:** 컴퓨팅 환경은 계속해서 진화하고 있다. PIM(Processing-In-Memory)과 같이 메모리 내에서 직접 연산을 수행하여 데이터 이동 비용을 원천적으로 줄이려는 새로운 하드웨어 아키텍처가 등장하고 있다.1 이러한 새로운 아키텍처의 특성에 최적화된 Pkd-tree의 변형을 설계하는 연구는 미래의 컴퓨팅 환경에서 성능을 한 단계 더 끌어올릴 수 있을 것이다.
- **하이브리드 색인 구조 개발:** 본 보고서에서 분석했듯이, 어떤 단일 자료구조도 모든 시나리오에서 최적일 수는 없다. Pkd-tree는 저-중차원 대용량 데이터에 강점을 보이고, PCA-tree나 LSH는 고차원 데이터에 강건함을 보인다. 데이터의 특성(차원, 크기, 내재적 구조)에 따라 Pkd-tree, PCA-tree, LSH와 같은 서로 다른 색인 전략을 동적으로 선택하거나 결합하는 하이브리드(hybrid) 색인 구조를 개발하는 연구는 더욱 강건하고 범용적인 해결책을 제공할 수 있을 것이다. 예를 들어, 트리의 상위 레벨에서는 PCA-tree 방식으로 데이터의 내재적 구조를 따라 분할하고, 하위 레벨에서는 Pkd-tree 방식으로 빠르게 병렬 처리하는 식의 결합을 고려해볼 수 있다.

결론적으로, Pkd-tree는 다차원 데이터 처리 분야에서 중요한 이정표를 세웠다. 이는 고전적인 자료구조가 현대 하드웨어와의 깊이 있는 상호작용을 통해 어떻게 재탄생할 수 있는지를 보여주는 탁월한 예시이며, 앞으로의 연구에 풍부한 영감과 방향을 제시하고 있다.

#### **참고 자료**

1. Optimal Batch-Dynamic kd-trees for Processing-in-Memory with Applications - Parallel Data Lab, 8월 1, 2025에 액세스, https://www.pdl.cmu.edu/PDL-FTP/associated/yzhao-SPAA-25.pdf
2. Parallel kd-tree with Batch Updates - Computer Science and Engineering, 8월 1, 2025에 액세스, https://www.cs.ucr.edu/~zmen002/files/publication/pkd.pdf
3. Parallel kd-tree with Batch Updates - Computer Science and Engineering, 8월 1, 2025에 액세스, https://www.cs.ucr.edu/~zmen002/files/publication/pkd_full.pdf
4. k-d 트리 - 위키백과, 우리 모두의 백과사전, 8월 1, 2025에 액세스, [https://ko.wikipedia.org/wiki/K-d_%ED%8A%B8%EB%A6%AC](https://ko.wikipedia.org/wiki/K-d_트리)
5. k-d tree - Wikipedia, 8월 1, 2025에 액세스, https://en.wikipedia.org/wiki/K-d_tree
6. KD Tree / KDB Tree - Game Client Developer Tech Blog, 8월 1, 2025에 액세스, https://jingonpark-gameclient.tistory.com/52
7. k-d Trees for nearest neighbor search | by Muneeb ur Rahman | Medium, 8월 1, 2025에 액세스, https://medium.com/@notesbymuneeb/k-d-trees-for-nearest-neighbor-search-df4fc459da51
8. Introduction to K-D Trees | Baeldung on Computer Science, 8월 1, 2025에 액세스, https://www.baeldung.com/cs/k-d-trees
9. [영상처리] 이미지매칭(3) - Nearest Neighbor Search (with kd-tree) - LTNS - 티스토리, 8월 1, 2025에 액세스, https://kimyo-s.tistory.com/54
10. Parallel kd-tree with Batch Updates | Request PDF - ResearchGate, 8월 1, 2025에 액세스, https://www.researchgate.net/publication/388900317_Parallel_kd-tree_with_Batch_Updates
11. An Improved Method to Build the KD Tree Based on Presorted Results - ResearchGate, 8월 1, 2025에 액세스, https://www.researchgate.net/publication/346686563_An_Improved_Method_to_Build_the_KD_Tree_Based_on_Presorted_Results
12. KD Trees - Computer Science Cornell, 8월 1, 2025에 액세스, https://www.cs.cornell.edu/courses/cs4780/2022sp/notes/LectureNotes19.html
13. [빅데이터] 차원의 저주(The curse of dimensionality), 8월 1, 2025에 액세스, https://datapedia.tistory.com/15
14. What is the curse of dimensionality and how does it affect vector search? - Milvus, 8월 1, 2025에 액세스, https://milvus.io/ai-quick-reference/what-is-the-curse-of-dimensionality-and-how-does-it-affect-vector-search
15. Why k-d trees is not used for high dimensional data? - Stack Overflow, 8월 1, 2025에 액세스, https://stackoverflow.com/questions/37132774/why-k-d-trees-is-not-used-for-high-dimensional-data
16. How to reduce KNN computation time using KD-Tree or Ball Tree? - GeeksforGeeks, 8월 1, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/how-to-reduce-knn-computation-time-using-kd-tree-or-ball-tree/
17. [Literature Review] Parallel $k$d-tree with Batch Updates, 8월 1, 2025에 액세스, https://www.themoonlight.io/en/review/parallel-kd-tree-with-batch-updates
18. ucrparlay/Pkd-tree: [SIGMOD' 25] A fast parallel kd-tree implementation - GitHub, 8월 1, 2025에 액세스, https://github.com/ucrparlay/KDtree
19. A progressive k-d tree for approximate k-nearest neighbors | Request PDF - ResearchGate, 8월 1, 2025에 액세스, https://www.researchgate.net/publication/324618593_A_progressive_k-d_tree_for_approximate_k-nearest_neighbors
20. Ball Tree and KD Tree Algorithms - GeeksforGeeks, 8월 1, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/ball-tree-and-kd-tree-algorithms/
21. Approximate Nearest Neighbor (ANN) - CelerData, 8월 1, 2025에 액세스, https://celerdata.com/glossary/approximate-nearest-neighbor
22. Scalable Nearest Neighbor Algorithms for High Dimensional Data - ResearchGate, 8월 1, 2025에 액세스, https://www.researchgate.net/publication/266378625_Scalable_Nearest_Neighbor_Algorithms_for_High_Dimensional_Data
23. Kd -Tree and Ball Tree Algorithms | by NithyaSri | Medium, 8월 1, 2025에 액세스, https://medium.com/@polkamnithyasri/kd-tree-and-ball-tree-algorithms-aa99dfad4ceb
24. KD-TREE AND BALL TREE IN KNN ALGORITHM | by Narasimharaodevisetti | Medium, 8월 1, 2025에 액세스, https://medium.com/@narasimharaodevisetti14/kd-tree-and-ball-tree-in-knn-algorithm-09a86d1bc6e6
25. Performance comparison of LSH and k-d tree for the nearest neighbour word search based on the number of PCA-dimensions N pca - ResearchGate, 8월 1, 2025에 액세스, https://www.researchgate.net/figure/Performance-comparison-of-LSH-and-k-d-tree-for-the-nearest-neighbour-word-search-based-on_fig5_325492642
26. Comparison of the efficiency of KD tree and hierarchical LSH for neighbor searching. - ResearchGate, 8월 1, 2025에 액세스, https://www.researchgate.net/figure/Comparison-of-the-efficiency-of-KD-tree-and-hierarchical-LSH-for-neighbor-searching_fig4_339545115
27. Approximate Principal Direction Trees, 8월 1, 2025에 액세스, https://icml.cc/2012/papers/348.pdf
28. Principal Component Analysis(PCA) - GeeksforGeeks, 8월 1, 2025에 액세스, https://www.geeksforgeeks.org/data-analysis/principal-component-analysis-pca/
29. A fast nearest-neighbor algorithm based on a principal axis searchtree - ResearchGate, 8월 1, 2025에 액세스, https://www.researchgate.net/publication/3193303_A_fast_nearest-neighbor_algorithm_based_on_a_principal_axis_searchtree
30. Fast Tree Based Nearest Neighbor Search - eScholarship.org, 8월 1, 2025에 액세스, https://escholarship.org/content/qt514265kf/qt514265kf_noSplash_73aaa1b1302f17576b2cb23a524802d3.pdf
31. Approximate Principal Direction Trees | Request PDF - ResearchGate, 8월 1, 2025에 액세스, https://www.researchgate.net/publication/227716486_Approximate_Principal_Direction_Trees
32. Learning the structure of manifolds using random projections, 8월 1, 2025에 액세스, http://papers.neurips.cc/paper/3195-learning-the-structure-of-manifolds-using-random-projections.pdf
33. Intrinsic dimension | Python, 8월 1, 2025에 액세스, https://campus.datacamp.com/courses/unsupervised-learning-in-python/decorrelating-your-data-and-dimension-reduction?ex=5
34. Which Spatial Partition Trees are Adaptive to Intrinsic Dimension?, 8월 1, 2025에 액세스, https://cseweb.ucsd.edu/~skpotufe/Papers/partitiontrees_camera_ready.pdf
35. Which Spatial Partition Trees are Adaptive to Intrinsic Dimension? - arXiv, 8월 1, 2025에 액세스, https://arxiv.org/pdf/1205.2609
36. Randomly-oriented k-d Trees Adapt to Intrinsic Dimension - Georgia Tech, 8월 1, 2025에 액세스, https://faculty.cc.gatech.edu/~vempala/papers/kdtrees.pdf
37. Fast kd-tree Construction with an Adaptive Error-Bounded Heuristic - ResearchGate, 8월 1, 2025에 액세스, https://www.researchgate.net/publication/224759149_Fast_kd-tree_Construction_with_an_Adaptive_Error-Bounded_Heuristic
38. CPU Ray Tracing Large Particle Data with Balanced P-k-d Trees, 8월 1, 2025에 액세스, https://www.sci.utah.edu/~knolla/ospParticle.pdf
39. CPU Ray Tracing Large Particle Data with Balanced P-k-d Trees - Scientific Computing and Imaging Institute - The University of Utah, 8월 1, 2025에 액세스, https://www.sci.utah.edu/publications/Wal2015a/ospParticle.pdf
40. CPU Ray Tracing Large Particle Data with Balanced P-k-d Trees - Will Usher, 8월 1, 2025에 액세스, https://www.willusher.io/publications/pkd/
41. In Situ Exploration of Particle Simulations with CPU Ray Tracing, 8월 1, 2025에 액세스, https://superfri.org/index.php/superfri/article/view/112
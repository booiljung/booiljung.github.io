---
layout: page
title: k-d 트리와 증분 k-d 트리에 대한 심층적 고찰
permalink: /sensors/point cloud/kd-tree/k-dtree와 증분 k-dtree 비교
---


k-d 트리(k-dimensional tree)는 k-차원 유클리드 공간에 분포하는 점들을 효율적으로 조직하고 검색하기 위해 설계된 공간 분할(space-partitioning) 자료구조이다.1 본질적으로, 1차원 데이터에 대한 이진 탐색 트리(Binary Search Tree, BST)의 개념을 다차원으로 일반화한 것으로 볼 수 있다. k-d 트리의 모든 노드는 k-차원의 점 하나를 저장하며 3, 리프 노드가 아닌 모든 내부 노드는 자신에게 할당된 공간을 두 개의 부분 공간, 즉 반공간(half-space)으로 분할하는 역할을 수행한다. 이 분할은 항상 특정 좌표축에 수직인 초평면(hyperplane)을 통해 이루어지는데, 이를 축-정렬(axis-aligned) 분할이라 칭한다.2

k-d 트리는 더 넓은 범주인 공간 분할 트리의 한 갈래로 이해될 수 있다. 2차원 공간을 균등하게 네 영역으로 나누는 쿼드트리(Quadtree)나 3차원 공간을 여덟 영역으로 나누는 옥트리(Octree)와는 달리, k-d 트리의 분할은 데이터의 분포에 따라 결정된다.5 또한, 임의의 방향으로 분할 평면을 허용하는 일반적인 이진 공간 분할(Binary Space Partitioning, BSP) 트리와 비교할 때, k-d 트리는 분할 평면이 반드시 좌표축에 수직이어야 한다는 제약을 가진다.6

이러한 설계적 선택은 양날의 검과 같다. 축-정렬 분할이라는 제약은 노드에 분할 차원을 나타내는 정수 하나와 분할 위치를 나타내는 실수 하나만 저장하면 되므로 노드 구조를 단순화하고, 순회 시 비교 연산을 간단하게 만들어 구현의 용이성과 속도 향상에 기여한다. 그러나 이로 인해 데이터의 분포가 축 방향과 정렬되지 않은 경우, 비효율적으로 길고 가는 형태의 공간 분할이 발생할 수 있으며, 이는 특히 고차원에서 성능 저하의 원인이 되기도 한다. 즉, k-d 트리의 설계 철학은 '표현의 간결성'과 '분할의 효율성'을 위해 '분할의 최적성'을 일부 희생한 결과물로, 이 트레이드오프를 이해하는 것이 k-d 트리의 성능 특성을 파악하는 첫걸음이다.

트리 구축 시 분할 차원은 일반적으로 트리의 깊이(depth)에 따라 순환적으로 선택된다. 예를 들어, 2차원에서는 루트 노드에서 x축, 레벨 1의 노드들에서 y축, 레벨 2에서 다시 x축을 기준으로 분할하는 식이다 (‘axis=depth(modk)‘).1 이러한 재귀적 분할 과정을 통해 초기 공간은 최종적으로 서로 겹치지 않는 볼록 다각형(convex polytopes)의 집합으로 나뉘게 된다.5

**Table 1: 공간 분할 트리의 특성 비교**

| 특성               | k-d Tree                          | Quadtree                       | Octree                         | BSP Tree                          |
| ------------------ | --------------------------------- | ------------------------------ | ------------------------------ | --------------------------------- |
| **적용 차원**      | k-차원                            | 2차원                          | 3차원                          | k-차원                            |
| **분할 방식**      | 축-정렬 초평면                    | 균등 4분할                     | 균등 8분할                     | 임의 방향 초평면                  |
| **데이터 의존성**  | 데이터 분포에 따라 분할 위치 결정 | 데이터 분포와 무관한 고정 분할 | 데이터 분포와 무관한 고정 분할 | 데이터 분포 또는 선택에 따라 결정 |
| **노드당 자식 수** | 2개 (이진 트리)                   | 4개                            | 8개                            | 2개 (이진 트리)                   |

이 표는 k-d 트리가 다른 공간 분할 기법들과 어떤 점에서 차별화되는지를 명확히 보여준다. 데이터 의존적인 분할 방식은 데이터가 밀집된 곳은 세밀하게, 희소한 곳은 넓게 분할하여 효율적인 공간 관리를 가능하게 하는 k-d 트리의 핵심 장점이다.


k-d 트리의 성능은 트리가 얼마나 균형 잡혀 있는지에 크게 좌우된다. 따라서 효율적인 구축 알고리즘은 균형을 유지하는 데 초점을 맞춘다. 본 섹션에서는 정적 데이터셋에 대한 k-d 트리 구축 알고리즘을 상세히 기술하고, 그 복잡도를 분석하며, 동적 환경에서의 한계를 논한다.


정적 k-d 트리는 주어진 점들의 집합으로부터 한 번에 구축되며, 일반적으로 다음과 같은 재귀적 절차를 따른다.9

1. **분할 차원 선택 (Select Splitting Dimension):** 트리의 현재 깊이 `depth`를 기준으로 분할할 차원을 결정한다. 가장 보편적인 방법은 `axis = depth % k` 공식을 사용하여 차원을 순환시키는 것이다.1 예를 들어, 3차원 공간에서는 x, y, z, x, y, z,... 순서로 분할 축이 선택된다. 대안으로는 각 차원별로 점들의 분산을 계산하여 가장 큰 분산을 갖는 차원을 선택하는 방법이 있다.9 이 방법은 데이터가 가장 넓게 퍼져 있는 방향으로 분할함으로써 잠재적으로 더 나은 공간 분할을 유도할 수 있다.

2. **분할점 선택 (Select Splitting Point):** 분할 차원이 결정되면, 해당 차원의 좌표값을 기준으로 모든 점들을 정렬한 후 중앙값(median)을 찾는다. 이 중앙값을 가진 점이 현재 노드의 데이터가 되며, 분할 평면의 위치가 된다.9 중앙값을 분할 기준으로 삼는 것은 점 집합을 거의 동일한 크기의 두 부분집합으로 나누어, 결과적으로 생성될 트리의 높이를 $

   `O(\log n)`$으로 유지하고 균형을 맞추는 핵심 전략이다.13

3. **재귀적 분할 (Recursive Partitioning):** 중앙값보다 작은 좌표값을 가진 점들의 집합은 왼쪽 서브트리로, 크거나 같은 값을 가진 점들의 집합은 오른쪽 서브트리로 보낸다. 이후 각 부분집합에 대해 깊이를 1 증가시켜 위 과정을 재귀적으로 반복한다. 점 집합이 비게 되면 재귀는 종료된다.9


k-d 트리 구축의 시간 복잡도는 주로 중앙값을 찾는 과정의 효율성에 의해 결정된다.

- 만약 매 단계에서 중앙값을 찾기 위해 비교 정렬(예: 병합 정렬, 힙 정렬)을 사용한다면, 중앙값 탐색에 $`O(n \log n)`$의 시간이 소요된다. 이 경우 전체 구축 시간 복잡도 $`T(n)`$은 재귀 관계식 $`T(n) = 2T(n/2) + O(n \log n)`$으로 표현되며, 이는 $`O(n \log^2 n)`$으로 풀린다.2
- 더 효율적인 방법은 'median-of-medians'와 같은 선형 시간 선택 알고리즘을 사용하는 것이다. 이 알고리즘은 평균적으로나 최악의 경우에도 ‘O(n)‘ 시간에 중앙값을 찾을 수 있다. 이 방법을 적용하면 재귀 관계식은 $`T(n) = 2T(n/2) + O(n)`$이 되며, 마스터 정리(Master Theorem)에 의해 전체 구축 시간 복잡도는 최적인 $`O(n \log n)`$이 된다.2


k-d 트리의 구조는 정적인 데이터셋에 대해 최적화되어 있지만, 이러한 최적화는 동적인 데이터 변화에 매우 취약하다.

- **삽입 (Insertion):** 새로운 점을 삽입하는 과정은 이진 탐색 트리와 유사하다. 루트부터 시작하여 각 노드의 분할 차원과 값을 비교하며 왼쪽 또는 오른쪽으로 이동하여 적절한 리프 노드 위치에 새 노드를 추가한다.4 이 연산 자체는 균형 잡힌 트리에서 

  ‘O(logn)‘ 시간에 수행될 수 있다. 그러나 문제는 삽입되는 데이터가 기존 데이터의 분포를 고려하지 않기 때문에, 반복적인 삽입은 트리를 한쪽으로 치우치게 만들어 심각한 불균형을 초래할 수 있다.4 최악의 경우 트리는 연결 리스트와 같은 형태가 되어 탐색 시간이 $

  `O(n)`$으로 저하된다.3

- **삭제 및 재균형 (Deletion and Rebalancing):** 노드 삭제는 더욱 복잡하다. 삭제할 노드가 리프 노드이면 간단히 제거할 수 있지만, 내부 노드일 경우 대체 노드를 찾아야 한다. 대체 노드는 보통 삭제될 노드의 분할 차원을 기준으로 오른쪽 서브트리에서 최소값을 갖는 노드나 왼쪽 서브트리에서 최대값을 갖는 노드로 선택된다.4 그 후, 원래 위치에 있던 대체 노드를 재귀적으로 삭제해야 한다.

가장 근본적인 문제는 재균형의 어려움에 있다. 이진 탐색 트리에서 사용되는 AVL 트리나 레드-블랙 트리의 회전(rotation)과 같은 효율적인 재균형 기법은 k-d 트리에 적용할 수 없다. 노드를 회전시키면 k-d 트리의 핵심 속성인 '축-정렬 분할' 규칙이 깨지기 때문이다.14 따라서 불균형을 해소하는 유일한 실질적인 방법은 문제가 되는 서브트리 전체를 주기적으로 재구축(rebuild)하는 것뿐이며, 이는 매우 비용이 큰 연산이다.14

이러한 정적 최적화와 동적 비효율성 사이의 근본적인 상충 관계는 k-d 트리가 '정적' 자료구조로 분류되는 주된 이유이다. 동적 환경에서의 성능 저하 문제를 해결하기 위해, 디스크 기반 환경에 맞춰 설계된 K-D-B-tree나 Bkd-tree와 같은 대안적인 자료구조들이 연구되었다.14


k-d 트리의 진정한 가치는 다차원 공간에서의 효율적인 질의 처리 능력에서 발현된다. 특히 최근접 이웃 탐색, k-최근접 이웃 탐색, 범위 탐색은 k-d 트리의 대표적인 활용 사례이다. 이들 연산의 핵심은 '가지치기(pruning)'라는 최적화 기법에 있으며, 이는 불필요한 탐색 공간을 기하학적으로 배제하여 성능을 극대화한다.


NNS는 주어진 질의점(query point)에 가장 가까운 데이터 포인트를 찾는 알고리즘이다. 최악의 경우 모든 점을 방문해야 하지만(‘O(n)‘), 평균적으로는 $`O(\log n)`$의 성능을 보인다. 알고리즘은 다음과 같은 두 단계로 구성된다.4

1. **초기 하강 (Initial Descent):** 질의점을 트리에 삽입하는 것과 동일한 방식으로 트리를 순회한다. 즉, 각 노드에서 해당 노드의 분할 차원을 기준으로 질의점의 좌표값과 노드의 좌표값을 비교하여 왼쪽 또는 오른쪽 자식으로 이동한다. 이 과정을 리프 노드에 도달할 때까지 반복한다. 도달한 리프 노드의 점을 현재까지의 최적 후보(`current best`)로 설정하고, 질의점과의 거리(보통 유클리드 거리의 제곱)를 최단 거리(`best distance`)로 기록한다.
2. **백트래킹 및 가지치기 (Backtracking and Pruning):** 리프 노드에서부터 루트 방향으로 재귀 호출에서 되돌아오면서(unwinding) 다음 과정을 수행한다:
   - 현재 방문한 노드의 점이 `current best`보다 질의점에 더 가깝다면, `current best`와 `best distance`를 이 노드의 점과 거리로 갱신한다.
   - **가지치기:** 현재 노드의 분할 평면이 질의점을 포함하는 반대편 서브트리를 탐색해야 할지 결정한다. 이를 위해 질의점과 분할 평면 사이의 수직 거리를 계산한다. 만약 이 거리가 현재의 `best distance`보다 크다면, 반대편 서브트리 내의 어떤 점도 현재의 `current best`보다 가까울 수 없음이 보장된다. 기하학적으로 이는 질의점을 중심으로 하고 `best distance`를 반지름으로 하는 초구(hypersphere)가 반대편 서브트리의 경계 상자(bounding box)와 전혀 교차하지 않음을 의미한다.9 이 경우, 해당 서브트리 전체를 탐색에서 제외하여 엄청난 계산량을 절약한다.
   - 만약 분할 평면까지의 거리가 `best distance`보다 작아 초구가 반대편 서브트리와 교차할 가능성이 있다면, 아직 방문하지 않은 해당 서브트리에 대해 재귀적으로 탐색을 수행한다.

이 과정이 루트 노드까지 완료되면, 최종적으로 `current best`에 저장된 점이 질의점의 최근접 이웃이 된다. k-d 트리 탐색의 본질은 이 '가지치기'에 있으며, 그 효율성은 데이터가 놓인 공간의 기하학적 특성에 의해 결정된다. 저차원 공간에서는 이 기하학적 휴리스틱이 잘 작동하여 대부분의 서브트리가 가지치기되지만, 고차원에서는 이 가정이 무너지면서 성능이 저하된다.


k-NN 탐색은 NNS를 k개의 이웃을 찾도록 확장한 것이다. 알고리즘의 핵심적인 차이는 단일 `current best` 대신, 크기가 k인 우선순위 큐, 특히 최대 힙(max-heap)을 유지하는 것이다.17

- 힙에는 현재까지 발견된 k개의 가장 가까운 이웃들과 질의점까지의 거리가 저장된다. 최대 힙을 사용하므로 힙의 루트(가장 높은 우선순위)에는 k개의 이웃 중 가장 멀리 있는 이웃의 정보가 위치한다.
- 탐색 과정에서 새로운 점을 만나면, 그 점과 질의점 사이의 거리를 계산하여 힙의 루트에 있는 거리와 비교한다. 만약 새로운 점이 더 가깝다면, 힙에서 루트를 제거하고 새로운 점을 삽입한다.
- 가지치기 조건 또한 변경된다. 질의점과 분할 평면 사이의 거리가 현재 힙에 저장된 k개의 이웃 중 가장 먼 이웃까지의 거리(즉, 힙의 루트 값)보다 클 때만 해당 서브트리를 가지치기할 수 있다.


범위 탐색은 축에 정렬된 직사각형이나 초구와 같은 특정 질의 범위 내에 포함되는 모든 점들을 찾는 연산이다.10 알고리즘은 트리를 재귀적으로 순회하며 다음과 같은 가지치기 규칙을 적용한다.

- 현재 노드가 표현하는 공간 영역(hyperrectangle)이 질의 범위와 전혀 겹치지 않으면, 해당 노드의 서브트리는 더 이상 탐색할 필요가 없으므로 가지치기한다.
- 현재 노드의 공간 영역이 질의 범위에 완전히 포함된다면, 해당 서브트리에 속한 모든 점들은 질의 조건을 만족하므로, 서브트리 전체를 결과에 추가하고 더 이상 하위 탐색을 진행하지 않는다.
- 만약 공간 영역이 질의 범위와 부분적으로만 겹친다면, 현재 노드의 점이 범위 내에 있는지 확인하고, 양쪽 자식 서브트리를 모두 재귀적으로 탐색한다.3

**Table 2: k-d 트리 주요 연산의 점근적 복잡도**

| 연산                         | 평균 시간 복잡도 | 최악 시간 복잡도 | 공간 복잡도 |
| ---------------------------- | ---------------- | ---------------- | ----------- |
| **구축 (Construction)**      | ‘O(nlogn)‘       | ‘O(nlogn)‘       | ‘O(n)‘      |
| **삽입 (Insertion)**         | ‘O(logn)‘        | ‘O(n)‘           | ‘O(n)‘      |
| **삭제 (Deletion)**          | ‘O(logn)‘        | ‘O(n)‘           | ‘O(n)‘      |
| **최근접 이웃 탐색 (NNS)**   | ‘O(logn)‘        | ‘O(n)‘           | ‘O(n)‘      |
| **범위 탐색 (Range Search)** | ‘O(n1−1/k+m)‘    | ‘O(n)‘           | ‘O(n)‘      |

주: $`n`$은 점의 개수, $`k`$는 차원, $`m`$은 범위 탐색 결과에 포함된 점의 개수이다. 평균 복잡도는 데이터가 무작위로 분포되어 있고 트리가 균형 잡혀있다고 가정한다. 2


k-d 트리는 저차원 공간에서 뛰어난 성능을 보이지만, 차원이 증가함에 따라 그 효율성이 급격히 감소하는 '차원의 저주(Curse of Dimensionality)' 현상에 매우 취약하다. 이 현상은 단순히 계산량이 증가하는 문제를 넘어, k-d 트리 알고리즘의 근본적인 최적화 원리를 파괴한다.


고차원 공간의 기하학은 우리의 3차원적 직관과 크게 다르다. 차원 $`d`$가 증가할수록 다음과 같은 역설적인 현상들이 나타난다.

- **부피의 집중:** 고정된 변의 길이를 갖는 단위 초입방체(unit hypercube)의 부피는 항상 1이지만, 그 안에 내접하는 단위 초구(unit hypersphere)의 부피는 차원이 증가함에 따라 0으로 수렴한다. 이는 초입방체의 거의 모든 부피가 중심에서 멀리 떨어진 '모서리' 근처에 집중되어 있음을 의미한다.16
- **거리 분포의 균일화:** 고차원 공간에서는 무작위로 선택된 점들 사이의 거리 분포가 매우 좁은 범위에 집중된다. 즉, 특정 질의점에서 가장 가까운 이웃까지의 거리와 가장 먼 이웃까지의 거리 차이가 상대적으로 매우 작아진다. 이는 사실상 "모든 점이 질의점에서 거의 같은 거리에 있다"는 상황을 만든다.
- **직교성:** 고차원 공간에서 무작위로 선택한 두 벡터는 매우 높은 확률로 거의 직교(orthogonal)한다.19


이러한 고차원 공간의 특성은 k-d 트리의 핵심 최적화 기법인 '가지치기'를 무력화시킨다.

- **가지치기의 실패:** 차원이 높아지면, 질의점을 중심으로 하는 아주 작은 반경의 초구조차도 거의 모든 방향으로 확장되어 대부분의 분할 초평면과 교차하게 된다.4 NNS 알고리즘에서 가지치기 조건은 

  `dist(query, plane) > best_dist`인데, 고차원에서는 `best_dist`가 다른 점들과의 거리와 큰 차이가 없고, 거의 모든 분할 평면까지의 거리가 이 `best_dist`보다 작아진다. 결과적으로, 알고리즘은 트리의 거의 모든 노드를 방문해야만 하며, 이는 사실상 전체 데이터를 순차적으로 검색하는 무차별 대입(brute-force) 방식과 다를 바 없게 된다. 성능은 $`O(\log n)`$에서 $`O(n)`$으로 수렴한다.8

실험적으로 k-d 트리의 성능은 약 20차원 이상부터 급격히 저하되기 시작하는 것으로 알려져 있다.20 '차원의 저주'는 점진적인 성능 저하가 아니라, 알고리즘의 효율성을 보장하던 핵심 원리인 '가지치기'가 기하학적으로 불가능해지는 근본적인 파괴 현상이다.


k-d 트리의 이러한 명백한 한계로 인해, 고차원 데이터 처리를 위한 여러 대안적인 접근법이 개발되었다.

- **근사 최근접 이웃 (Approximate Nearest Neighbor, ANN):** 100% 정확한 최근접 이웃을 찾는 것을 포기하는 대신, 매우 높은 확률로 실제 최근접 이웃이거나 그에 매우 가까운 점을 빠르게 찾는 방법이다. LSH(Locality-Sensitive Hashing), HNSW(Hierarchical Navigable Small World)와 같은 알고리즘들이 여기에 속하며, 고차원 검색의 실질적인 표준으로 자리 잡았다.21
- **Ball Tree:** k-d 트리가 축-정렬 초사각형으로 공간을 분할하는 것과 달리, Ball Tree는 초구(hypersphere)를 사용하여 공간을 분할한다.24 초구는 축-정렬 제약이 없어 데이터의 분포 형태에 더 유연하게 적응할 수 있으므로, 일부 고차원 데이터셋에서 k-d 트리보다 나은 성능을 보인다.16


ikd-Tree(Incremental k-d Tree)는 표준 k-d 트리의 가장 큰 약점인 동적 데이터 처리 능력을 해결하기 위해 제안된 혁신적인 자료구조이다.30 특히 LiDAR 센서 데이터와 같이 지속적으로 새로운 점들이 유입되는 로보틱스 응용 분야에서 기존 정적 k-d 트리의 한계를 극복하는 것을 목표로 한다.31 정적 트리가 새로운 데이터가 추가될 때마다 전체 트리를 재구축해야 하는 막대한 비용을 감수해야 했던 반면, ikd-Tree는 이름 그대로 '증분적(incremental)'으로 트리를 업데이트하고 재균형을 맞추는 데 특화되어 있다.


ikd-Tree의 핵심 철학은 '전체 재구축'을 피하고 '최소한의 국소적 변경'으로 트리의 유효성과 균형을 유지하는 것이다.

- **증분 업데이트 (Incremental Update):** 새로운 점들이 들어오면, ikd-Tree는 전체 데이터셋을 다시 처리하는 대신 새로 들어온 점들만 기존 트리에 삽입한다.30 이 방식은 LiDAR 주행 기록계(odometry)와 같이 순차적으로 데이터를 획득하는 환경에서 중복 연산을 극적으로 줄여준다.32 삭제 및 재삽입 연산 또한 지원하여 동적인 환경 변화에 유연하게 대응할 수 있다.33
- **동적 재균형 (Dynamic Rebalancing):** 표준 k-d 트리의 가장 큰 난제는 삽입/삭제가 반복될수록 트리가 불균형해지며, 이를 해결할 효율적인 재균형 메커니즘이 없다는 점이다. ikd-Tree는 이 문제를 해결하기 위해 트리의 상태를 지속적으로 감시하고, 불균형이 감지된 영역에 대해서만 **부분적인 재구축(partial rebuilding)**을 수행한다.32 이는 문제가 되는 서브트리만 효율적으로 재구성함으로써 전체 트리를 재구축하는 비용을 피하면서도 트리의 균형을 유지하는 핵심 전략이다.
- **병렬 처리 (Parallel Processing):** 실시간 성능을 더욱 향상시키기 위해, ikd-Tree는 재균형 과정과 같은 비용이 큰 작업을 별도의 스레드에서 병렬적으로 처리하도록 설계되었다.32 이를 통해 메인 스레드는 지연 없이 계속해서 최근접 이웃 탐색과 같은 핵심 연산을 수행할 수 있다.


ikd-Tree는 단순한 동적 k-d 트리를 넘어, 로보틱스 분야의 특정 요구사항을 만족시키기 위한 실용적인 기능들을 포함한다.32

- **박스 단위 연산 (Box-wise Operations):** 단일 점뿐만 아니라 특정 공간(box) 내의 모든 점들을 한 번에 삽입하거나 삭제하는 연산을 지원한다. 이는 로봇의 시야에서 벗어난 영역의 점들을 한 번에 제거하는 등의 작업에 매우 유용하다.
- **다운샘플링 (Down-sampling):** LiDAR 데이터와 같이 매우 밀도가 높은 포인트 클라우드를 처리할 때, 모든 점을 저장하는 것은 비효율적일 수 있다. ikd-Tree는 트리에 점을 추가하는 과정에서 자동으로 데이터를 다운샘플링하여 원하는 해상도를 유지하는 기능을 내장하고 있다.

실험 결과에 따르면, ikd-Tree는 LiDAR-관성 주행 기록계(LIO)와 같은 실제 로봇 응용에서 기존의 정적 k-d 트리를 사용하는 방식에 비해 단 4%의 실행 시간만으로 동일한 작업을 수행할 수 있는 것으로 나타났다.33 이는 ikd-Tree가 정적 k-d 트리의 뛰어난 검색 성능을 유지하면서도 동적 환경에서의 치명적인 업데이트 비용 문제를 성공적으로 해결했음을 보여준다.

**Table 3: 표준 k-d 트리와 ikd-Tree의 성능 비교**

| 기준                 | 표준 k-d 트리 (Standard k-d Tree)   | ikd-Tree (Incremental k-d Tree)              |
| -------------------- | ----------------------------------- | -------------------------------------------- |
| **주요 데이터 모델** | 정적 데이터셋                       | 동적 데이터 스트림                           |
| **업데이트 방식**    | 전체 재구축                         | 증분 업데이트 및 부분 재균형 32              |
| **동적 성능**        | 낮음 (불균형으로 성능 저하)         | 높음 (지속적인 재균형으로 성능 유지)         |
| **재균형 비용**      | 매우 높음                           | 낮음 (부분 재구축 및 병렬 처리) 32           |
| **주 응용 분야**     | 정적 데이터 검색 (GIS, 오프라인 ML) | 실시간 로보틱스 (LiDAR 매핑, 모션 플래닝) 31 |


k-d 트리는 그 효율성과 단순성 덕분에 다양한 과학 및 공학 분야에서 핵심적인 도구로 사용되고 있다. k-d 트리의 성공적인 적용 사례들을 분석해 보면, 대부분 데이터가 한 번 구축된 후 거의 변하지 않는 '정적성'과 데이터의 유효 차원이 비교적 낮은 '차원의 적절성'이라는 두 가지 조건을 만족하는 경향이 있다.


- **k-최근접 이웃(k-NN) 알고리즘:** k-d 트리는 k-NN 분류 및 회귀 모델의 성능을 가속하는 데 가장 널리 사용되는 자료구조 중 하나이다. 전체 학습 데이터셋을 k-d 트리에 미리 구축해두면, 새로운 질의점이 주어졌을 때 그 주변의 k개 최근접 이웃을 평균 ‘O(logn)‘ 시간에 찾을 수 있다.4 이는 전체 데이터셋(

  $`n`$개)과 거리를 모두 계산하는 무차별 대입 방식($`O(n)`$)에 비해 압도적으로 빠르다. 학습 데이터는 보통 한 번 정해지면 잘 바뀌지 않으므로 k-d 트리의 정적인 특성과 잘 부합한다.

- **클러스터링 및 이상 탐지:** k-means와 같은 반복적인 클러스터링 알고리즘에서 각 점을 가장 가까운 중심점에 할당하는 단계를 가속하거나, 각 점에서 가장 가까운 이웃과의 거리를 기반으로 비정상적으로 멀리 떨어진 점(이상치)을 탐지하는 데 활용될 수 있다.11


- **레이 트레이싱:** 광선이 장면(scene)과 교차하는 첫 번째 지점을 찾는 과정에서, k-d 트리는 방대한 공간을 효율적으로 분할하여 광선이 통과할 가능성이 없는 영역을 빠르게 배제하는 가속 구조(acceleration structure)로 사용된다.11 특히 implicit k-d 트리는 렌더링 중에 변하지 않는 정적인 대규모 씬을 다루는 데 탁월한 성능을 보인다.27 3차원 공간은 차원의 저주 문제에서 자유롭기 때문에 k-d 트리의 이상적인 적용 환경이다.
- **충돌 감지:** 게임 환경에서 수많은 객체들 간의 충돌 여부를 매 프레임마다 검사해야 할 때, k-d 트리는 잠재적으로 충돌할 수 있는 객체 쌍의 후보군을 빠르게 추려내는 데 사용된다.11 그러나 객체들이 계속 움직이는 매우 동적인 환경에서는 매번 트리를 재구축하는 비용이 크기 때문에, 쿼드트리나 옥트리, 또는 간단한 그리드 기반 방법이 더 선호되기도 한다.28


- **특징점 매칭:** 두 이미지 간의 대응 관계를 찾기 위해, 각 이미지에서 SIFT, SURF와 같은 특징점 기술자(feature descriptor)를 추출한다. 이 기술자들은 보통 64차원 또는 128차원의 고차원 벡터인데, 한 이미지의 기술자들을 k-d 트리에 저장한 후 다른 이미지의 각 기술자에 대한 최근접 이웃을 찾아 매칭 쌍을 구성한다.18 이때, 최근접 이웃과 두 번째 최근접 이웃 간의 거리 비율(Nearest Neighbor Distance Ratio, NNDR)을 검사하여 모호한 매칭을 제거하는 기법이 함께 사용된다.21 SIFT와 같은 고차원 데이터에 k-d 트리가 사용되는 이유는, 종종 정확한 최근접 이웃이 아닌 근사 최근접 이웃(ANN)을 찾는 용도로 변형되어 사용되기 때문이다. 이는 역설적으로 k-d 트리가 고차원에서 한계를 보인다는 점을 방증한다.


- **지리 정보 시스템 (GIS):** 지도상의 특정 위치 주변에 가장 가까운 편의시설을 찾거나(NNS), 특정 사각 영역 내에 있는 모든 건물을 검색하는(Range Search) 등 다양한 공간 질의를 처리하는 데 널리 사용된다.11 지리 데이터는 대부분 정적이며 2차원 또는 3차원이므로 k-d 트리가 매우 효과적이다.
- **로보틱스 및 생물정보학:** 로봇 팔의 동작 계획(motion planning)을 위한 장애물 탐색이나 18, 거대한 유전자 서열 데이터베이스 또는 단백질 구조 데이터에서 유사한 패턴을 검색하는 데에도 k-d 트리의 원리가 응용된다.29


1970년대에 존 벤틀리(Jon Bentley)에 의해 처음 제안된 k-d 트리는 8 반세기가 지난 오늘날까지도 다차원 데이터 처리 분야에서 중요한 위치를 차지하고 있다. 이는 k-d 트리가 고전적인 알고리즘이 현대 컴퓨팅 환경의 변화와 만나 어떻게 재해석되고 진화하는지를 보여주는 살아있는 사례라 할 수 있다. 본 보고서는 k-d 트리의 원리부터 동적 환경에 특화된 변종인 ikd-Tree에 이르기까지 그 핵심적인 강점과 명백한 약점을 종합적으로 고찰하였다.

k-d 트리의 핵심적인 강점은 저차원 및 중간 차원(‘d<20‘) 공간에서 정적인 데이터에 대한 최근접 이웃 및 범위 탐색에 있어 매우 효율적이라는 점이다. 중앙값 기반의 균형 잡힌 구축을 통해 달성되는 평균 $`O(\log n)`$의 탐색 시간은 그 가장 큰 매력이며, 구현 또한 비교적 간단하다. 반면, 치명적인 약점은 '차원의 저주' 현상으로 인해 고차원 공간에서는 성능이 무차별 대입 수준으로 급격히 저하된다는 것과, 동적 데이터에 대한 삽입/삭제 시 균형을 유지하기 어려워 값비싼 재구축 비용을 감수해야 한다는 점이다.

이러한 특성을 바탕으로, 어떤 문제에 어떤 k-d 트리를 사용해야 하는지에 대한 명확한 가이드라인을 제시할 수 있다.

- **표준 k-d 트리 (Standard k-d Tree):** 데이터가 정적이고, 차원이 20 미만인 범용 다차원 포인트 인덱싱 문제에 가장 적합하다. GIS 데이터 분석, 오프라인 k-NN 모델링 등이 대표적인 예이다.
- **ikd-Tree (Incremental k-d Tree):** 데이터가 지속적으로 유입되고 변경되는 동적인 환경, 특히 실시간 성능이 중요한 로보틱스 응용에 최고의 선택이다.31 증분 업데이트와 부분적 재균형 메커니즘은 기존 k-d 트리의 동적 업데이트 문제를 효과적으로 해결한다.32
- **대안적 자료구조 (Alternatives):** 데이터가 고차원이거나 근사 해답이 허용되는 경우, Ball Tree, LSH, HNSW와 같은 ANN 기법을 우선적으로 고려해야 한다. 데이터가 매우 동적으로 변하지만 ikd-Tree와 같은 정교한 재균형 기법이 불필요한 경우, Quadtree, Octree, 또는 동적 업데이트에 특화된 R-tree, Bkd-tree 등이 더 나은 선택일 수 있다.

k-d 트리의 역사는 알고리즘이 진공 속에서 존재하는 것이 아니라, 그것이 실행되는 하드웨어 아키텍처와 해결하려는 문제의 특성이라는 두 축에 의해 그 가치와 성능이 결정됨을 보여준다. 초기 k-d 트리는 정적 데이터에 최적화된 구조로 탄생했지만, 로보틱스와 같이 동적인 데이터 스트림을 다루는 현대적 응용 분야의 요구에 부응하기 위해 ikd-Tree와 같은 형태로 진화했다. ikd-Tree는 증분 업데이트와 병렬화된 부분 재균형이라는 영리한 전략을 통해 k-d 트리의 고질적인 동적 성능 문제를 해결함으로써, 이 고전적인 자료구조가 현대적인 실시간 시스템에서도 여전히 강력한 도구가 될 수 있음을 증명했다.

앞으로 k-d 트리에 대한 연구는 하드웨어 인식(hardware-aware) 설계와 동적 환경 적응성 강화라는 두 방향으로 더욱 발전할 것이다. CPU/GPU의 캐시 계층, 병렬 처리 능력을 극대화하는 알고리즘과 함께, ikd-Tree와 같이 로그 구조화(log-structured) 기법이나 점진적 재균형 기법을 결합하여 동적 환경에서의 성능 저하를 최소화하려는 시도가 계속될 것이다. 이처럼 k-d 트리는 고전 알고리즘의 원리가 현대 기술과 상호작용하며 그 생명력을 이어가는, 컴퓨터 과학의 중요한 진화 과정을 보여주는 상징적인 예로 남을 것이다.


1. Search and Insertion in K Dimensional tree - GeeksforGeeks, accessed July 31, 2025, https://www.geeksforgeeks.org/dsa/search-and-insertion-in-k-dimensional-tree/
2. k-d tree - Wikipedia, accessed July 31, 2025, https://en.wikipedia.org/wiki/K-d_tree
3. KD Trees in C++ - GeeksforGeeks, accessed July 31, 2025, https://www.geeksforgeeks.org/cpp/kd-trees-in-cpp/
4. Introduction to K-D Trees | Baeldung on Computer Science, accessed July 31, 2025, https://www.baeldung.com/cs/k-d-trees
5. KDTree Tutorial - Bridges, accessed July 31, 2025, https://bridgesuncc.github.io/tutorials/KdTree.html
6. Kd-Tree Accelerator - Physically Based Rendering, accessed July 31, 2025, https://pbr-book.org/3ed-2018/Primitives_and_Intersection_Acceleration/Kd-Tree_Accelerator
7. TKDTree< Index, Value > Class Template Reference - ROOT, accessed July 31, 2025, https://root.cern/doc/v626/classTKDTree.html
8. kd-Trees, accessed July 31, 2025, https://www.cs.cmu.edu/~ckingsf/bioinfo-lectures/kdtrees.pdf
9. KD-Tree - Yasen Hu, accessed July 31, 2025, https://yasenh.github.io/post/kd-tree/
10. KdTree - cs.Princeton, accessed July 31, 2025, https://www.cs.princeton.edu/courses/archive/spr13/cos226/lectures/99GeometricSearch.pdf
11. K-D Tree: A Deep Dive into Efficient Search - Number Analytics, accessed July 31, 2025, https://www.numberanalytics.com/blog/k-d-tree-efficient-search-data-structure
12. Mastering K-D Trees in Geometric Algorithms - Number Analytics, accessed July 31, 2025, https://www.numberanalytics.com/blog/ultimate-guide-kd-trees-geometric-algorithms
13. Complexity Analysis of 2D KD-Tree Construction, Query and Modification Based on Master Theorem and Potential Energy Analysis - OpenReview, accessed July 31, 2025, https://openreview.net/pdf?id=OsqSXydBlD
14. The Bkd Tree. A Dynamic Disk Optimized BSP Tree | by Nick Gerleman | Medium, accessed July 31, 2025, https://medium.com/@nickgerleman/the-bkd-tree-da19cf9493fb
15. Bkd-Tree: A Dynamic Scalable kd-Tree - Duke Computer Science, accessed July 31, 2025, https://users.cs.duke.edu/~pankaj/publications/papers/bkd-sstd.pdf
16. KD Trees - Computer Science Cornell, accessed July 31, 2025, https://www.cs.cornell.edu/courses/cs4780/2022sp/notes/LectureNotes19.html
17. How do I traverse a KDTree to find k nearest neighbors? - Stack Overflow, accessed July 31, 2025, https://stackoverflow.com/questions/34688977/how-do-i-traverse-a-kdtree-to-find-k-nearest-neighbors
18. Mastering K-D Trees: Advanced Data Structure - Number Analytics, accessed July 31, 2025, https://www.numberanalytics.com/blog/ultimate-guide-to-k-d-trees
19. Lecture 11: Nearest Neighbor Search and the Curse of Dimensionality 1 kd-trees, accessed July 31, 2025, https://www.cs.yale.edu/homes/el327/datamining2013aFiles/11_nearest_neighbor_search.pdf
20. KDTree - SciPy v1.16.1 Manual, accessed July 31, 2025, https://docs.scipy.org/doc/scipy/reference/generated/scipy.spatial.KDTree.html
21. An Improved Algorithm Finding Nearest Neighbor Using Kd-trees - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/220980101_An_Improved_Algorithm_Finding_Nearest_Neighbor_Using_Kd-trees
22. Approximate Nearest Neighbors (ANN) Algorithm using KD-Trees from Scratch in Python, accessed July 31, 2025, https://medium.com/@prxdyu/approximate-nearest-neighbors-ann-algorithm-using-kd-trees-from-scratch-in-python-41d3f0a34214
23. An Improved Algorithm Finding Nearest Neighbor Using Kd-trees - Stanford CS Theory, accessed July 31, 2025, http://theory.stanford.edu/~rinap/papers/kdtreelatin.pdf
24. What is a K-Dimensional Tree? - Medium, accessed July 31, 2025, https://medium.com/@katyayanivemula90/what-is-a-k-dimensional-tree-8265cc737d77
25. 1.6. Nearest Neighbors - scikit-learn 1.7.1 documentation, accessed July 31, 2025, https://scikit-learn.org/stable/modules/neighbors.html
26. Kd-Tree (Bounding Volumes) ray tracing demo - YouTube, accessed July 31, 2025, https://www.youtube.com/watch?v=ZBo-jzGaJH4
27. Interactive k-D Tree GPU Raytracing - Stanford Computer Graphics Laboratory, accessed July 31, 2025, https://graphics.stanford.edu/papers/i3dkdtree/gpu-kd-i3d.pdf
28. Fully dynamic KD-Tree vs. Quadtree? - Game Development Stack Exchange, accessed July 31, 2025, https://gamedev.stackexchange.com/questions/87138/fully-dynamic-kd-tree-vs-quadtree
29. What is KD-Tree? | Activeloop Glossary, accessed July 31, 2025, https://www.activeloop.ai/resources/glossary/kd-tree/
30. mech.hku.hk, accessed July 31, 2025, [https://mech.hku.hk/seminars/ikd-tree%3A-an-incremental-k-d-tree-for-robotic-applications#:~:text=The%20ikd%2DTree%20incrementally%20updates,both%20theory%20and%20practice%20level.](https://mech.hku.hk/seminars/ikd-tree%3A-an-incremental-k-d-tree-for-robotic-applications#:~:text=The ikd-Tree incrementally updates,both theory and practice level.)
31. ikd-Tree: An Incremental K-D Tree for Robotic Applications - HKU Mech, accessed July 31, 2025, https://mech.hku.hk/seminars/ikd-tree%3A-an-incremental-k-d-tree-for-robotic-applications
32. (PDF) ikd-Tree: An Incremental K-D Tree for Robotic Applications, accessed July 31, 2025, https://www.researchgate.net/publication/349520013_ikd-Tree_An_Incremental_K-D_Tree_for_Robotic_Applications
33. ikd-Tree: An Incremental KD Tree for Robotic Applications - Bohrium, accessed July 31, 2025, https://www.bohrium.com/paper-details/ikd-tree-an-incremental-k-d-tree-for-robotic-applications/867748286671356153-108625
34. ikd-Tree: An Incremental KD Tree for Robotic Applications (2102.10808v1) - Emergent Mind, accessed July 31, 2025, https://www.emergentmind.com/papers/2102.10808
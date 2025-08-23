# BDL-tree


현대 컴퓨팅 환경에서 대규모 다차원 공간 데이터(spatial data)를 효율적으로 관리하고 질의하는 능력은 병렬 데이터베이스, 머신러닝, 컴퓨터 그래픽스, 로보틱스 등 수많은 분야에서 핵심적인 요구사항으로 자리 잡았다 `[1, 2, 3]`. 이러한 데이터를 위한 색인 구조의 성능은 전체 시스템의 효율성을 좌우하는 결정적 요소이다. 여러 공간 데이터 구조 중 k-d(k-dimensional) 트리는 저차원 및 중간 차원 공간 데이터에 대한 최근접 이웃(k-NN) 탐색과 같은 유사성 질의를 효율적으로 지원하는, 고전적이면서도 널리 사용되는 데이터 구조이다 `[3, 4, 5]`. k-d 트리는 주어진 점 집합에 대해 공간을 재귀적으로 분할하여 효율적인 검색 경로를 제공한다.

그러나 k-d 트리의 근본적인 한계는 정적(static) 데이터에 최적화되어 있다는 점에 있다 `[4, 5]`. 실제 응용 프로그램의 데이터는 끊임없이 변화하므로 삽입, 삭제와 같은 동적 업데이트가 필수적이다. 이 동적 환경에서 k-d 트리의 균형을 유지하기 위한 기존의 단순한 전략들은 심각한 성능 상충 관계(trade-off)를 야기한다.

- **전략 1 (전체 재구축, Baseline 1)**: 모든 업데이트마다 전체 트리를 처음부터 다시 구축하는 방식이다. 이 전략은 항상 완벽하게 균형 잡힌 트리를 유지하여 질의 성능을 최상으로 보장하지만, 업데이트 비용이 극도로 높아 쓰기 작업이 빈번한 환경에서는 비현실적이다 `[5, 6]`.
- **전략 2 (재구축 없음, Baseline 2)**: 기존 트리의 공간 분할을 변경하지 않고 점을 그대로 삽입하고, 삭제된 점은 삭제 표시(tombstone)로 남겨두는 방식이다. 업데이트 자체는 매우 빠르지만, 시간이 지남에 따라 트리가 점차 불균형해지고 쓸모없는 데이터가 누적되어 질의 성능이 심각하게 저하된다 `[5, 6]`.

이러한 동적 업데이트와 질의 성능 간의 근본적인 충돌, 그리고 현대 멀티코어 프로세서의 병렬성을 적극적으로 활용해야 하는 시대적 요구에 부응하기 위해 **BDL-tree(Batch-Dynamic Log-structured tree)**가 제안되었다 `[4, 5, 7]`. BDL-tree는 병렬 환경에서 대규모 배치(batch) 업데이트와 질의를 모두 효율적으로 처리하는 것을 목표로 한다.

BDL-tree의 핵심 아이디어는 **로그 구조화된 k-d 트리 집합(log-structured set of k-d trees)**이라는 설계에 있다 `[4, 5, 8, 9]`. 이는 정적 데이터 구조를 동적으로 변환하는 고전적인 "로그 방식(logarithmic method)" `[5, 10]`을 병렬 k-d 트리에 성공적으로 적용한 것이다. 이 구조를 통해 BDL-tree는 `n`개의 점을 가진 트리에 `B`개의 업데이트 배치 처리를 상각(amortized) 작업량 $$O(B \log^2(n+B))$$와 병렬 시간(depth) $$O(\log(n+B) \log\log(n+B))$$에 수행하며, 업데이트와 질의 성능 간의 실용적인 균형점을 제공한다 `[4, 5, 7]`. BDL-tree의 개발은 단순히 새로운 데이터 구조의 발명을 넘어, 하드웨어 트렌드(멀티코어 병렬성)에 의해 증폭된 데이터 구조 설계의 근본적인 딜레마, 즉 '읽기-쓰기 성능 상충 관계'에 대한 정교한 공학적 해법이라 할 수 있다. 그 진정한 혁신은 완전히 새로운 개념을 창조한 것이 아니라, 이미 알려진 강력한 기법을 새로운 문제 영역에 성공적으로 적용하여 실용적인 해결책을 제시했다는 점에 있다.

본 보고서는 BDL-tree의 구조적 설계 원리, 핵심 연산 알고리즘 및 복잡도, 상세한 성능 실험 결과 분석, 그리고 Pkd-tree, 공간 LSM-tree, 학습 기반 인덱스와 같은 경쟁 및 대안 기술과의 심층 비교를 통해 BDL-tree에 대한 포괄적이고 심도 있는 고찰을 제공하고자 한다.


BDL-tree는 동적으로 변화하는 점 집합에 대해 효율적인 병렬 k-NN 질의를 지원하는 병렬 배치-동적 k-d 트리 구현체로 정의된다 `[4, 7]`. 그 구조적 핵심은 **"로그 구조화된 k-d 트리 집합"**이라는 개념에 있다 `[4, 5]`. 이 구조는 `Ns`개의 개별적인 정적(static) k-d 트리들로 구성되며, 각 트리의 용량은 $2^0, 2^1, \dots, 2^{N_s-1}$과 같이 지수적으로 증가한다 `[5]`. 또한, 가장 작은 용량의 트리 역할을 하는 버퍼(buffer) 영역을 포함하여 새로운 업데이트를 일차적으로 수용한다 `[5]`.

이러한 설계는 완전히 새로운 것이 아니라 기존 연구의 연장선상에 있다. 외부 메모리(external memory) 환경을 위해 설계된 **Bkd-Tree**나 동적 **캐시-인지(cache-oblivious) k-d 트리** 역시 버퍼와 점진적으로 커지는 정적 구조를 활용한 바 있다 `[5]`. 하지만 BDL-tree의 핵심적인 차별점은 이 구조를 **병렬 배치 업데이트**에 최적화하고, 삭제 연산 후의 **균형 유지 메커니즘**을 병렬 환경에 맞게 새롭게 설계했다는 점이다 `[5]`.

BDL-tree의 구조는 정적 데이터 구조를 동적으로 변환하는 일반적인 설계 원칙인 로그 방식(Logarithmic Method)을 병렬 k-d 트리에 적용한 구체적인 사례로 볼 수 있다 `[5, 10]`. 이 방식은 비싼 업데이트 비용을 한 번에 치르지 않고, 여러 개의 작은 구조체에 변경 사항을 분산시켜 처리한 후, 필요할 때 병합하여 비용을 상각(amortize)하는 전략을 취한다. 이러한 설계 철학은 현대 데이터베이스 시스템에서 쓰기 집약적인 워크로드를 처리하기 위해 널리 쓰이는 **LSM-tree(Log-Structured Merge-tree)**와 매우 유사하다 `[5]`. LSM-tree가 쓰기 연산을 메모리 내의 MemTable에 버퍼링한 후 디스크의 SSTable로 순차적으로 병합하여 비싼 랜덤 I/O를 회피하는 것처럼, BDL-tree는 업데이트를 버퍼와 작은 트리들에 모았다가 더 큰 트리로 병렬 재구축함으로써 비싼 전역적 구조 변경을 회피한다.

BDL-tree의 지수적 크기 증가를 갖는 트리 집합 구조는 알고리즘 설계의 '상각 분석(amortized analysis)' 원리를 물리적으로 구현한 것으로 해석할 수 있다. 이는 매 업데이트마다 작은 비용을 지불하는 대신, 변경사항을 모아서 훨씬 드물게 발생하는 큰 비용(재구축)으로 처리하는 전략이다. 거대하고 균형 잡힌 단일 k-d 트리에 대한 개별 업데이트는 변경 사항이 루트까지 전파될 수 있어 비용이 크다. BDL-tree는 가능한 한 큰 트리를 건드리지 않고, 새로운 데이터를 작고 재구축 비용이 저렴한 트리들로 보낸다 `[5]`. 작은 트리와 버퍼가 가득 차면, 이들을 병합하여 비어있는 다음 크기의 트리 슬롯으로 재구축하는데, 이는 마치 이진수 덧셈에서 올림(carry)이 발생하는 것과 유사하다 `[5]`. 지수적으로 증가하는 트리 크기는 이 비용이 효율적으로 분산되도록 보장하는 수학적 장치이다. 크기 $2^i$의 트리를 재구축하는 비용은 대략 $O(2^i \log(2^i))$이지만, 이 재구축은 $2^i$개의 새로운 점이 더 작은 트리들에 삽입된 후에야 발생한다. 따라서 점 하나당 상각 비용은 $O(\log(2^i)) = O(i)`가 되며, 한 점이 최대 $\log n$개의 레벨을 거쳐 이동할 수 있으므로 총 상각 비용에 $\log n$ 인자가 추가되어 최종적으로 작업량 복잡도에 $\log^2 n$ 항이 나타나게 된다 `[4, 7]`. 이 분석을 통해 BDL-tree의 구조가 임의적인 것이 아니라, 상각 분석이 효과적으로 작동하도록 정밀하게 설계되었음을 알 수 있다. 즉, 질의 복잡도를 $\log n$개의 트리를 검색하는 만큼 약간 희생하는 대신, 업데이트 처리량을 극적으로 향상시키는 전략적 선택인 것이다.


BDL-tree의 성능을 이해하기 위해서는 먼저 병렬 알고리즘 분석 모델인 Work-Span 모델을 이해해야 한다. **Work ($T_1$)**는 알고리즘이 수행하는 총 연산의 수로, 순차 실행 시간에 해당한다. **Span ($T_\infty$)**은 가장 긴 의존성 경로의 길이로, 무한개의 프로세서를 사용할 때의 병렬 실행 시간을 의미한다 `[11]`. 이 모델은 알고리즘의 내재적 병렬성을 측정하는 표준적인 척도이다.


`B`개의 점으로 구성된 배치 삽입 요청이 들어오면, BDL-tree는 `INSERTL`이라는 루틴을 통해 로그 방식에 따라 삽입을 수행한다 `[5]`. 이 과정의 핵심 메커니즘은 **비트마스크**를 사용하여 하위 트리들의 상태를 관리하는 것이다. 현재 채워진 트리들의 상태를 나타내는 비트마스크 `F`가 있고, 새로운 배치를 추가했을 때 예상되는 새로운 비트마스크 `F_new`를 계산한다. `F`와 `F_new`의 비트별 차이(`XOR` 연산)를 통해 어떤 트리들을 병합하여 새로운 더 큰 트리로 재구성해야 하는지 결정할 수 있다 `[6]`. 예를 들어, 크기 1, 2, 4, 8의 트리 슬롯이 있고 현재 1과 4가 채워져 있다면 비트마스크는 `...0101`이 된다. 여기에 크기 3의 배치가 들어오면, 1과 2가 비어있으므로 1과 2를 채우고 4는 그대로 둔다. 만약 크기 5의 배치가 들어오면, 기존의 1과 4, 그리고 새로운 5를 합쳐 총 10개의 점이 된다. 이 10개의 점은 비어있는 2와 8 크기의 트리 슬롯에 채워지게 된다.

병합 대상으로 결정된 트리들의 모든 점과 새로운 배치 점들을 수집한 후, 이들을 기반으로 새로운 k-d 트리를 **병렬로 재구축**한다. 이 과정에서 BDL-tree 구현은 병렬 캐시-인지(cache-oblivious) k-d 트리 구축 알고리즘과 같은 최적화된 기법을 활용하여 높은 병렬성과 효율성을 달성한다 `[5, 7]`.


BDL-tree의 삭제는 단순히 점을 제거하는 것이 아니라, 전체 구조의 균형을 유지하기 위한 체계적인 3단계 프로세스로 이루어진다 `[6]`.

1. **1단계: 병렬 대량 삭제 (`EraseS`)**: BDL-tree 내의 각 하위 k-d 트리에 대해 병렬적으로 대량 삭제 서브루틴을 호출한다. 이 서브루틴은 재귀적으로 동작하며, 각 노드에서 삭제할 점들의 배치를 분할하여 자식 노드들로 병렬적으로 전달한다. 리프 노드에 도달하면 점들을 실제로 삭제한다. 이 과정에서 한 노드의 두 자식이 모두 비게 되면 해당 노드도 제거되고, 한 자식만 남으면 트리를 평탄화(flatten)하여 불필요한 노드를 제거함으로써 트리의 깊이를 최적화한다 `[6]`.
2. **2단계: 고갈된 트리에서 점 수집**: 삭제 후, 모든 하위 트리를 병렬로 스캔하여 용량이 원래의 50% 미만으로 줄어든 '고갈된(depleted)' 트리를 찾는다. 이 트리들에 남아있는 모든 점들을 수집하여 별도의 집합 `R`을 만든다 `[5, 6]`. 이것이 BDL-tree의 핵심적인 재균형(rebalancing) 트리거 메커니즘이다.
3. **3단계: 수집된 점 재삽입**: 마지막으로, 수집된 점들의 집합 `R`을 다시 BDL-tree에 삽입하기 위해 앞서 설명한 `INSERTL` 루틴을 호출한다. 이 과정은 데이터가 희소해진 여러 개의 작은 트리들을 하나의 밀도 높은 트리로 통합하여 데이터 압축 및 전체 구조의 균형 회복 효과를 가져온다 `[5, 6]`.

이 삭제 알고리즘은 단순한 제거가 아니라, 주기적인 '압축(compaction)' 및 '가비지 컬렉션(garbage collection)' 프로세스에 가깝다. 50% 용량 임계값은 이러한 유지보수 작업을 언제 수행할지를 결정하는 휴리스틱 트리거 역할을 한다. 이는 쓰기 성능을 위해 삭제된 데이터를 즉시 처리하지 않고 나중에 일괄 처리하는 LSM-tree의 압축 철학과 정확히 일치하며, BDL-tree가 데이터 관리 시스템 전반에 걸쳐 사용되는 '로그 구조화' 설계 패러다임의 한 구현임을 명확히 보여준다.


BDL-tree의 k-NN 질의는 데이터-병렬(data-parallel) 방식으로 접근한다. 즉, 질의점들의 배치에 대해 병렬적으로 수행되며, 수많은 질의점들에 대한 탐색이 동시에 일어난다 `[6]`.

알고리즘은 다음과 같이 진행된다.

1. 각 질의점에 대해 k-NN 결과를 저장할 k-NN 버퍼를 할당한다. 이 버퍼는 현재까지 찾은 k개의 최근접 이웃을 관리하며, $2k$의 내부 크기를 가지고 있다. 버퍼가 가득 차면 선택 알고리즘을 통해 상위 `k`개만 남겨 삽입 연산의 상각 시간 복잡도를 $O(1)$로 유지한다 `[6]`.
2. BDL-tree를 구성하는 $\log n$개의 하위 k-d 트리를 순차적으로 순회한다.
3. 각 하위 트리 $T_i$에 대해, 모든 질의점에 대한 k-NN 탐색을 병렬로 수행한다.
4. 이때, 각 질의점은 **동일한 k-NN 버퍼를 계속해서 재사용**한다. $T_i$에서 찾은 이웃이 이전에 $T_0$부터 $T_{i-1}$까지에서 찾았던 k번째 이웃보다 더 가깝다면, 버퍼는 새로운 이웃으로 업데이트된다. 이 방식은 별도의 명시적인 병합 단계 없이 여러 트리의 검색 결과를 자연스럽게 정제하고 통합하는 효과적인 메커니즘이다 `[6]`.


$n$개의 점을 가진 BDL-tree에 $B$개 업데이트 배치 연산 시 복잡도는 다음과 같다 `[4, 5, 7]`.

- **상각 작업량 (Amortized Work)**: $$O(B \log^2(n+B))$$
- **깊이 (Depth / Parallel Time)**: $$O(\log(n+B) \log\log(n+B))$$

작업량의 $\log^2 n$ 항은 각 점이 상각적으로 $\log n$번의 재구축을 겪을 수 있고, 각 재구축은 병렬 트리 구성에 $\log n$에 비례하는 작업이 필요하기 때문에 발생한다. 깊이의 $\log\log n$ 항은 BDL-tree 구현에 사용된 병렬 k-d 트리 구축 알고리즘의 깊이 복잡도에서 기인한다 `[12]`.

다음은 BDL-tree의 핵심 연산에 대한 복잡도를 요약한 표이다.

| 연산 (Operation)             | 상각 작업량 (Amortized Work)        | 깊이 (Depth / Parallel Time)  |
| ---------------------------- | ----------------------------------- | ----------------------------- |
| **구축 (Construction)**      | $O(N \log N)$                       | $O(\log N \log\log N)$        |
| **배치 삽입 (Batch Insert)** | $O(B \log^2(n+B))$                  | $O(\log(n+B) \log\log(n+B))$  |
| **배치 삭제 (Batch Delete)** | $O(B \log^2(n+B))$                  | $O(\log(n+B) \log\log(n+B))$  |
| **k-NN 질의 (k-NN Query)**   | $O(Q \log n \cdot T_{kNN\_single})$ | $O(T_{kNN\_single} + \log n)$ |

*주: N은 초기 점의 개수, n은 현재 점의 개수, B는 배치 크기, Q는 질의 개수, $T_{kNN_single}$은 단일 k-d 트리에서의 k-NN 질의 복잡도.*


BDL-tree의 실질적인 성능을 검증하기 위해, 36코어(하이퍼스레딩 포함 72스레드) 머신과 144GB RAM 환경에서 C++와 ParlayLib 라이브러리를 사용하여 구현 및 테스트가 수행되었다 `[4, 5, 6]`. 성능 비교 대상으로는 두 가지 극단적인 전략을 가진 베이스라인 알고리즘이 사용되었다.

- **B1 (Full Rebuild)**: 매 배치 업데이트 후 전체 트리를 재구축하여 항상 최적의 질의 성능을 보장하는 전략 `[6]`.
- **B2 (Tombstone)**: 업데이트 시 재구축을 전혀 하지 않고, 삭제된 점은 표시만 남겨두어 업데이트 속도를 극대화하는 전략 `[6]`.

실험에는 균일 분포(Uniform)와 군집 분포(Clustered)를 가진 2D, 5D, 7D 합성 데이터셋과 3개의 실제 데이터셋(최대 3억 2천만 개 점)이 사용되어 다양한 시나리오에서의 성능이 측정되었다 `[6]`.

주요 실험 결과는 다음과 같이 분석된다.

- **구축 성능**: BDL-tree는 B1, B2와 비슷하거나 더 나은 구축 성능 및 확장성을 보였으며, 최대 35.4배의 병렬 스피드업을 달성했다 `[5, 6, 8]`.
- **업데이트 성능 (삽입/삭제)**:
  - **확장성**: 예상대로 B2가 가장 빠르고 B1이 가장 느렸으며, BDL-tree는 재구축 비용을 상각하여 그 중간에 위치했다. BDL-tree는 업데이트에서 최대 약 35배의 병렬 스피드업을 보였다 `[4, 5, 6, 8]`.
  - **배치 크기 영향**: BDL-tree와 B1의 처리량(throughput)은 **배치 크기가 커질수록 증가**하는 경향을 보였다. 이는 재구축이라는 고정 비용이 더 많은 항목에 분산되기 때문이다. 이 결과는 BDL-tree가 개별적인 소규모 업데이트보다는 대규모 배치 업데이트에 더 적합함을 시사한다 `[6]`.
- **k-NN 질의 성능**:
  - **정적 환경**: 단일 배치로 트리를 만든 직후에는 BDL-tree가 B1/B2보다 약간 느렸다. 이는 $\log n$개의 트리를 모두 검색해야 하는 구조적 오버헤드 때문이다 `[6]`.
  - **동적/혼합 워크로드**: **이것이 BDL-tree가 진정한 가치를 발휘하는 부분이다.** 업데이트가 누적될수록 B2의 트리 구조는 심하게 왜곡되어 k-NN 성능이 급격히 저하된다. B1은 업데이트 자체가 너무 느려 전체 처리 시간을 저해한다. 반면, BDL-tree는 꾸준한 업데이트와 질의 성능을 모두 유지하여, **혼합된 워크로드의 총 실행 시간에서 가장 우수한 성능**을 보였다 `[5, 6]`.
- **데이터 특성 영향**: BDL-tree의 성능 경향은 테스트된 차원(최대 16D)과 데이터 분포(균일/군집) 전반에 걸쳐 일관되게 나타나, 구조의 견고함을 입증했다 `[6]`.

이러한 실험 결과는 단순히 복잡도 분석을 검증하는 것을 넘어, 하나의 중요한 공학적 절충안을 특성화한다. '최고의' 알고리즘은 워크로드에 따라 결정된다. BDL-tree는 읽기 전용(read-only) 또는 쓰기 전용(write-only) 시스템을 위한 것이 아니라, 읽기와 쓰기가 빈번하게 교차하는 '복잡한 현실'에 최적화된 구조이며, 이는 많은 실제 병렬 데이터베이스 및 머신러닝 시스템의 특징이다. 데이터 구조를 선택할 때, 예상되는 질의/업데이트 비율을 반드시 분석해야 함을 시사한다. 업데이트가 드물다면 B1(재구축)도 괜찮고, 질의가 드물다면 B2(삭제 표시)도 괜찮다. 그러나 둘 다 빈번하고 병렬적으로 발생한다면, BDL-tree가 바로 그 해답이 될 수 있다.


BDL-tree는 동적 공간 인덱싱 분야의 유일한 해결책이 아니며, 다양한 철학을 가진 경쟁 및 대안 기술들과 비교를 통해 그 위상을 명확히 할 수 있다.


Pkd-tree는 BDL-tree와 마찬가지로 최신 병렬 동적 k-d 트리 구현체로, BDL-tree와 직접적인 비교 대상이다 `[3]`. 두 구조의 핵심 철학은 다음과 같이 대비된다.

- **BDL-tree (Log-tree)**: **로그 구조화(Log-structured)** 접근법을 따른다. `O(\log n)`개의 완벽하게 균형 잡힌 작은 트리들을 유지하며, 업데이트는 트리들의 병합과 재구축을 통해 이루어진다. 이로 인해 질의는 모든 트리를 검색해야 하는 오버헤드가 발생한다 `[3, 13]`.
- **Pkd-tree**: **지역적 재구축(Local reconstruction)** 접근법을 따른다. *단일* k-d 트리를 유지하되, 약간의 불균형(weight-balanced)을 허용한다. 업데이트로 인해 특정 서브트리의 불균형이 임계값 `α`를 초과하면, 오직 그 영향받은 서브트리만 재구축한다 `[3, 13]`.

이러한 철학의 차이는 성능 상충 관계로 이어진다. 질의 성능 면에서는 단 하나의 트리만 탐색하면 되는 Pkd-tree가 일반적으로 더 빠르다. BDL-tree는 $\log n$개의 트리를 검색해야 하므로, 질의 성능이 한 자릿수 배수만큼 느릴 수 있다 `[3, 13]`. 반면 업데이트 성능과 구현 복잡성 면에서는 각기 다른 장단점을 가진다. 이들의 존재는 동적 공간 데이터를 다루는 설계 패러다임의 분화를 보여준다. 선택지는 (A) 여러 개의 완벽한 작은 구조체들의 집합(BDL-tree), (B) 하나의 불완전하지만 거대한 단일 구조체(Pkd-tree) 사이에서 이루어지며, 이는 '하나의 만능 해결책'이 없음을 의미한다.


BDL-tree의 로그 구조화 방식은 쓰기 집약적 키-값 저장소의 표준인 LSM-tree와 철학을 공유한다 `[5, 14, 15]`. 그러나 LSM-tree를 다차원 공간 데이터에 적용하는 데에는 근본적인 어려움이 따른다. LSM-tree는 본질적으로 1차원(키 순서) 구조이므로, 다차원 공간 데이터를 처리하려면 공간 채움 곡선(space-filling curve)을 사용해 1차원 키로 변환하거나, 별도의 공간 보조 인덱스를 구축해야 한다 `[16, 17]`.

이 과정에서 하나의 공간 범위 질의가 여러 개의 분리된 1차원 키 범위로 매핑되거나, 질의가 메모리의 MemTable부터 디스크의 모든 SSTable 계층까지 확인해야 하는 심각한 **읽기 증폭(Read Amplification)** 문제가 발생한다 `[18, 19]`. **ER-tree**와 같은 연구는 각 SSTable 내부에 R-tree와 유사한 구조(SER-tree)를 내장하여 이 문제를 완화하려 시도한다 `[19, 20]`. BDL-tree와 공간 LSM-tree는 쓰기 최적화를 위해 '로그 구조화' 철학을 공유하지만, 읽기에서 근본적으로 다른 도전에 직면한다. BDL-tree의 읽기 페널티는 예측 가능한 $O(\log n)$ 배수인 반면, 공간 LSM-tree는 질의와 데이터 분포에 따라 예측 불가능하고 치명적일 수 있는 읽기 증폭 문제를 안고 있다.


"The Case for Learned Index Structures" `[21, 22, 23]`에서 제안된 패러다임은 B-트리와 같은 전통적인 데이터 구조를 데이터 분포를 학습하는 머신러닝 모델로 대체하는 것이다. 이 아이디어는 공간 데이터로 확장되어, 공간 채움 곡선을 학습하거나 재귀적 모델 인덱스(RMI)를 사용하여 공간을 분할하는 연구들로 이어지고 있다 `[24, 25, 26]`.

그러나 학습 기반 인덱스의 가장 큰 약점은 동적 데이터 처리이다. 모델은 정적인 데이터 분포에 대해 훈련되므로, 업데이트로 인해 데이터 분포가 크게 바뀌면 모델의 예측 정확도가 떨어져 성능이 급격히 저하되며, 모델 재훈련은 매우 비용이 많이 든다 `[27, 28, 29]`. FlexFlood와 같은 최신 연구는 데이터 분포가 왜곡될 때 부분적으로 구조를 재구성하는 방식으로 이 문제를 해결하려 시도하고 있다 `[28, 30]`.

BDL-tree는 이 문제에 대한 *고전적, 알고리즘 기반* 접근법의 정점을 대표한다. 반면, 학습 기반 인덱스는 근본적으로 다른 *데이터 기반* 철학을 제시한다. 현재로서는 BDL-tree가 데이터 분포에 상관없이 견고한 성능 보장을 제공하므로, 범용 동적 워크로드에 훨씬 더 실용적인 접근법으로 평가된다.



BDL-tree의 공식 코드는 GitHub 저장소에서 공개적으로 제공된다 `[31]`. 이 구현의 핵심적인 특징은 **ParlayLib**라는 C++ 라이브러리를 사용한다는 점이다 `[31]`. ParlayLib는 카네기 멜런 대학에서 개발한 고성능 병렬 프로그래밍 라이브러리로, BDL-tree 알고리즘의 기반이 되는 작업 훔치기(work-stealing) 스케줄러와 병렬 프리미티브(map, filter 등)를 제공한다 `[32, 33]`. ParlayLib를 선택한 것은 BDL-tree가 이론적 분석(Work-Span 모델)과 실제 구현 간의 긴밀한 연결을 추구함을 보여준다. ParlayLib의 알고리즘들은 알려진 비용을 가진 빌딩 블록으로 설계되어, 이를 조합하여 BDL-tree와 같은 더 복잡한 알고리즘을 만들고 그 성능을 분석적으로 예측할 수 있게 한다 `[32]`. 이는 BDL-tree가 단일 알고리즘을 넘어, 이론적으로 견고한 고성능 병렬 알고리즘 모음(예: ParGeo 라이브러리 `[8, 34, 35]`)을 구축하려는 더 큰 연구 프로그램의 일부임을 시사한다.

빌드 시스템으로는 CMake를 사용하며, Google Benchmark를 통해 성능 벤치마크를 수행할 수 있다 `[31]`. 다만, README 파일에 코드가 **"연구 단계(research-phase)"**라고 명시되어 있어, 상용 제품에 바로 적용하기에는 안정성이나 API 호환성 측면에서 주의가 필요함을 알 수 있다 `[31]`.


BDL-tree는 병렬 데이터베이스 및 유사성 검색과 같은 일반적인 분야 외에도 `[4, 7]`, 다음과 같은 구체적인 응용 분야에서 잠재력을 가진다.

- **머신러닝**: 실시간으로 데이터가 추가/삭제되는 환경에서의 k-NN 분류 `[10]`.
- **계산 기하학**: 동적 점 집합에 대한 클러스터링, 최소 포함 공(smallest enclosing ball) 계산 등 `[2, 8]`.
- **컴퓨터 그래픽스 및 시뮬레이션**: 객체가 계속 움직여 가속 구조(acceleration structure)의 빈번한 업데이트가 필요한 동적 장면의 레이 트레이싱(Ray Tracing) 또는 N-바디 시뮬레이션 `[36, 37, 38]`.
- **로보틱스 및 센서 네트워크**: LiDAR 등에서 실시간으로 수집되는 포인트 클라우드 데이터 처리 `[2, 36]`.


본 보고서는 동적 공간 데이터에 대한 업데이트와 질의 성능의 균형을 맞추는 효율적인 병렬 데이터 구조의 필요성을 재확인하고, 이에 대한 해결책으로 BDL-tree를 심층적으로 분석했다. BDL-tree는 로그 구조화된 k-d 트리 집합이라는 독창적인 설계를 통해 병렬 배치-동적 환경에서 업데이트 비용을 효과적으로 상각하는 해법을 제시했다.

BDL-tree의 가장 큰 강점은 읽기 또는 쓰기 중 하나에만 치우친 단순한 전략들과 달리, **읽기와 쓰기가 빈번하게 교차하는 혼합된 워크로드에서 가장 뛰어난 종합 성능**을 제공한다는 점이다 `[5, 6]`. 이는 많은 실제 시스템의 요구사항과 부합한다. 그러나 구조적으로 $\log n$개의 하위 트리를 검색해야 하므로, 단일의 완벽하게 균형 잡힌 트리에 비해 k-NN 질의 성능이 느릴 수밖에 없다는 명백한 한계 또한 존재한다.

현재 동적 공간 인덱싱 분야에서 BDL-tree는 Pkd-tree와 함께 고전적 알고리즘 기반 설계의 성숙한 접근법으로 자리매김하며, 데이터 분포를 학습하는 학습 기반 인덱스와 같은 새로운 데이터 기반 패러다임과 뚜렷한 대조를 이룬다.

향후 연구는 BDL-tree의 한계를 극복하는 방향으로 진행될 수 있다. 예를 들어, 하위 트리들 간의 더 효과적인 가지치기(pruning) 전략을 통해 k-NN 질의 성능을 개선하거나, 고차원 데이터에 대한 성능 저하를 완화하는 방안을 연구할 수 있다. 더 나아가, BDL-tree의 로그 구조와 Pkd-tree의 지역 재구축 아이디어를 결합한 하이브리드 접근법을 탐구하거나, 비휘발성 메모리(Persistent Memory)와 같은 새로운 하드웨어 환경에 BDL-tree의 인-메모리 설계를 적용하고 영속성 및 일관성 문제를 해결하는 연구도 유망한 방향이 될 것이다 `[39, 40]`. BDL-tree는 동적 공간 데이터 관리의 중요한 이정표이며, 앞으로의 발전을 위한 견고한 기반을 제공한다.


1. Comparison of Advance Tree Data Structures - arXiv, 7월 31, 2025에 액세스, https://arxiv.org/pdf/1209.6495
2. Optimal Batch-Dynamic kd-trees for Processing-in-Memory with Applications - Parallel Data Lab, 7월 31, 2025에 액세스, https://www.pdl.cmu.edu/PDL-FTP/associated/yzhao-SPAA-25.pdf
3. Parallel kd-tree with Batch Updates - Computer Science and Engineering, 7월 31, 2025에 액세스, https://www.cs.ucr.edu/~zmen002/files/publication/pkd_full.pdf
4. [2112.06188] Parallel Batch-Dynamic $k$d-Trees - arXiv, 7월 31, 2025에 액세스, https://arxiv.org/abs/2112.06188
5. Parallel Batch-Dynamic kd-trees - arXiv, 7월 31, 2025에 액세스, https://arxiv.org/pdf/2112.06188
6. Parallel Batch-Dynamic kd-trees Rahul Yesantharao - DSpace@MIT, 7월 31, 2025에 액세스, https://dspace.mit.edu/bitstream/handle/1721.1/143277/Yesantharao-rahuly-meng-eecs-2022-thesis.pdf?sequence=1&isAllowed=y
7. Parallel Batch-Dynamic kd-trees - DSpace@MIT, 7월 31, 2025에 액세스, https://dspace.mit.edu/handle/1721.1/143277
8. ParGeo: A Library for Parallel Computational Geometry, 7월 31, 2025에 액세스, https://par.nsf.gov/servlets/purl/10398946
9. Parallel Batch-Dynamic $k$d-Trees | Request PDF - ResearchGate, 7월 31, 2025에 액세스, https://www.researchgate.net/publication/357013194_Parallel_Batch-Dynamic_kd-Trees
10. The forest of trees that makes up the data structure. In this instance, T 2 is empty - ResearchGate, 7월 31, 2025에 액세스, https://www.researchgate.net/figure/The-forest-of-trees-that-makes-up-the-data-structure-In-this-instance-T-2-is-empty_fig3_221471602
11. Theoretically Efficient Parallel Density-Peak Clustering - 丘成桐中学科学奖, 7월 31, 2025에 액세스, http://www.yau-awards.com/uploads/file/20221128/20221128231126_26996.pdf
12. Parallel Batch-Dynamic kd-trees - ResearchGate, 7월 31, 2025에 액세스, https://www.researchgate.net/publication/361327593_Parallel_Batch-Dynamic_d-trees
13. Parallel kd-tree with Batch Updates - Computer Science and ..., 7월 31, 2025에 액세스, https://www.cs.ucr.edu/~zmen002/files/publication/pkd.pdf
14. ER-tree index structure for spatial data on LSM-trees. - ResearchGate, 7월 31, 2025에 액세스, https://www.researchgate.net/figure/ER-tree-index-structure-for-spatial-data-on-LSM-trees_fig2_359488071
15. A survey of LSM-Tree based Indexes, Data Systems and KV-stores - arXiv, 7월 31, 2025에 액세스, https://arxiv.org/html/2402.10460v2
16. Comprehensive Comparison of LSM Architectures for Spatial Data - Computer Science and Engineering, 7월 31, 2025에 액세스, https://www.cs.ucr.edu/~vagelis/publications/Comprehensive_Comparison_of_LSM_Architectures_for_Spatial_Data__BigData_2020_Short.pdf
17. A Comparative Study of Log-Structured Merge-Tree-Based Spatial Indexes for Big Data - Chen Li, 7월 31, 2025에 액세스, https://chenli.ics.uci.edu/files/icde2017-AsterixDB-Spatial-Comparison.pdf
18. (PDF) An LSM-Tree Index for Spatial Data - ResearchGate, 7월 31, 2025에 액세스, https://www.researchgate.net/publication/359488071_An_LSM-Tree_Index_for_Spatial_Data
19. An LSM-Tree Index for Spatial Data - MDPI, 7월 31, 2025에 액세스, https://www.mdpi.com/1999-4893/15/4/113
20. An LSM-Tree Index for Spatial Data - Bohrium, 7월 31, 2025에 액세스, https://www.bohrium.com/paper-details/an-lsm-tree-index-for-spatial-data/812515450050052100-20453
21. (PDF) The Case for Learned Index Structures - ResearchGate, 7월 31, 2025에 액세스, https://www.researchgate.net/publication/321512926_The_Case_for_Learned_Index_Structures
22. The Case for Learned Index Structures - David Procházka, 7월 31, 2025에 액세스, https://prochazka.dev/posts/the-case-for-learned-index-structures/
23. [1712.01208] The Case for Learned Index Structures - arXiv, 7월 31, 2025에 액세스, https://arxiv.org/abs/1712.01208
24. The structure of the learned spatial textual index - ResearchGate, 7월 31, 2025에 액세스, https://www.researchgate.net/figure/The-structure-of-the-learned-spatial-textual-index_fig5_364326419
25. WISK: A Workload-aware Learned Index for Spatial Keyword Queries - arXiv, 7월 31, 2025에 액세스, https://arxiv.org/pdf/2302.14287
26. LMSFC: A Novel Multidimensional Index based on Learned Monotonic Space Filling Curves - VLDB Endowment, 7월 31, 2025에 액세스, https://www.vldb.org/pvldb/vol16/p2605-gao.pdf
27. mgoin/learned_indexes: Experiments on ideas proposed in Tim Kraska's "The Case for Learned Index Structures" - GitHub, 7월 31, 2025에 액세스, https://github.com/mgoin/learned_indexes
28. FlexFlood: Efficiently Updatable Learned Multi-dimensional Index - ML For Systems, 7월 31, 2025에 액세스, https://mlforsystems.org/assets/papers/neurips2024/paper18.pdf
29. A Tutorial on Learned Multi-dimensional Indexes - Computer Science Purdue, 7월 31, 2025에 액세스, https://www.cs.purdue.edu/homes/aref/IDAS/tutorialsigspatial2020-a.pdf
30. FlexFlood: Efficiently Updatable Learned Multi-dimensional Index - arXiv, 7월 31, 2025에 액세스, https://arxiv.org/pdf/2411.09205
31. rahulyesantharao/batch-dynamic-kdtree: Parallel, batch ... - GitHub, 7월 31, 2025에 액세스, https://github.com/rahulyesantharao/batch-dynamic-kdtree
32. Brief Announcement: ParlayLib – A Toolkit for Parallel Algorithms on Shared-Memory Multicore Machines - Carnegie Mellon University, 7월 31, 2025에 액세스, https://www.cs.cmu.edu/~./blelloch/papers/3350755.3400254.pdf
33. ParlayLib - A Toolkit for Parallel Algorithms on Shared-Memory Multicore Machines, 7월 31, 2025에 액세스, https://www.researchgate.net/publication/342821391_ParlayLib_-_A_Toolkit_for_Parallel_Algorithms_on_Shared-Memory_Multicore_Machines
34. ParGeo: A Library for Parallel Computational Geometry - arXiv, 7월 31, 2025에 액세스, https://arxiv.org/pdf/2207.01834
35. Publications - Open-source Code Available - University of California, Riverside, 7월 31, 2025에 액세스, https://www.cs.ucr.edu/~ygu/publication/code/index.html
36. Parallel kd-Tree Construction on the GPU with an Adaptive Split and Sort Strategy - David Wehr, 7월 31, 2025에 액세스, https://davidawehr.com/projects/files/kdtree_pub.pdf
37. Highly Parallel Fast KD-tree Construction for Interactive Ray Tracing of Dynamic Scenes | Request PDF - ResearchGate, 7월 31, 2025에 액세스, https://www.researchgate.net/publication/220506832_Highly_Parallel_Fast_KD-tree_Construction_for_Interactive_Ray_Tracing_of_Dynamic_Scenes
38. (PDF) Fast Radius Search Exploiting Ray-Tracing Frameworks, JCGT - ResearchGate, 7월 31, 2025에 액세스, https://www.researchgate.net/publication/349551320_Fast_Radius_Search_Exploiting_Ray-Tracing_Frameworks_JCGT
39. Buffered Persistence in B+ Trees - Computer Science : University of Rochester, 7월 31, 2025에 액세스, https://www.cs.rochester.edu/u/scott/papers/2025_SIGMOD_BD+Tree.pdf
40. Persistent Memory in Database Systems - mediaTUM, 7월 31, 2025에 액세스, https://mediatum.ub.tum.de/doc/1638761/1638761.pdf


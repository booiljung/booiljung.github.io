# 그래프 신경망(GNN)
[그래프 신경망(GNN)](./index.md)



그래프(Graph)는 정점(Vertex 또는 Node)들의 집합 $V$와 이들을 연결하는 간선(Edge)들의 집합 $E$로 구성된 자료구조, 즉 $G = (V, E)$로 정의된다.1 이 단순한 정의는 현실 세계의 거의 모든 복잡한 시스템을 모델링할 수 있는 놀라운 표현력을 제공한다. 소셜 네트워크에서 개인은 정점, 친구 관계는 간선으로 표현될 수 있다. 화학에서는 원자가 정점, 화학 결합이 간선이 되며, 금융 시스템에서는 계좌가 정점, 거래가 간선이 되어 자금의 흐름을 나타낸다.1 이처럼 그래프는 관계, 상호작용과 같은 추상적인 개념을 다루기에 가장 적합한 언어이며, 복잡한 문제를 단순화하고 새로운 관점에서 해결할 수 있는 강력한 도구다.4

그러나 그래프 데이터는 전통적인 기계 학습 모델이 다루던 데이터와 근본적인 차이점을 가진다. 이미지, 음성, 텍스트와 같은 데이터는 유클리드 공간(Euclidean Space) 상의 격자(grid)나 시퀀스(sequence) 형태로 쉽게 표현될 수 있다.3 이러한 데이터는 차원이 고정되어 있고, 데이터 포인트 간의 '거리'가 중요한 의미를 가진다. 반면, 그래프 데이터는 비유클리드 공간(Non-Euclidean Space)에 존재한다.3 그래프는 고정된 형태가 없으며, 노드의 이웃 개수는 가변적이고 노드 간의 순서라는 개념이 존재하지 않는다. 두 그래프가 시각적으로 다르게 보이더라도 동일한 연결 구조를 가질 수 있으며, 수백만 개의 노드를 가진 거대 그래프는 인간이 직관적으로 해석하기 매우 어렵다.4 이러한 특성은 그래프 데이터를 분석하고 모델링하는 데 본질적인 어려움을 야기한다.


지난 10년간 딥러닝은 이미지 인식, 자연어 처리 등 다양한 분야에서 혁신을 이끌었다. 컨볼루션 신경망(Convolutional Neural Networks, CNN)과 순환 신경망(Recurrent Neural Networks, RNN)은 각각 이미지와 시퀀스 데이터에서 복잡한 패턴을 학습하는 데 탁월한 성능을 보였다.7 CNN은 고정된 크기의 필터를 이미지의 모든 위치에 적용하여 지역적 특징을 추출하고, 이를 조합하여 계층적인 표현을 학습한다.2 RNN은 순서가 있는 데이터의 시간적 의존성을 모델링한다.

하지만 이러한 모델들의 성공은 데이터가 규칙적인 구조를 가진다는 강력한 가정에 기반한다.8 CNN의 컨볼루션 연산은 이웃의 개수와 순서가 고정된 격자 구조에서만 정의될 수 있다. 노드마다 이웃의 수가 다르고 순서가 없는 그래프 데이터에 CNN 필터를 직접 적용하는 것은 불가능하다. 마찬가지로, RNN은 데이터의 순차적 흐름을 가정하므로, 순서가 없는 그래프 구조에 적용하기 어렵다. 이처럼 기존 심층 신경망은 그래프의 불규칙하고 복잡한 위상 구조(topology)를 처리하는 데 근본적인 한계를 드러냈으며, 이는 그래프 구조 데이터를 직접 다룰 수 있는 새로운 패러다임의 필요성을 야기했다.8


그래프 신경망(Graph Neural Network, GNN)은 이러한 한계를 극복하기 위해 등장한, 그래프 구조 데이터를 직접 입력으로 받아 처리하고 분석하도록 설계된 딥러닝 모델의 총칭이다.7 GNN의 등장은 단순히 새로운 모델의 개발을 넘어, 딥러닝이 다룰 수 있는 데이터의 범위를 정형화된 유클리드 공간에서 불규칙하고 관계적인 비유클리드 공간으로 확장시킨 패러다임의 전환을 의미한다. 이는 관계와 상호작용이라는 추상적 개념을 데이터의 부가 정보가 아닌, 모델링의 핵심 대상으로 삼을 수 있게 되었음을 시사한다.1

GNN의 궁극적인 목표는 그래프의 위상 구조와 각 노드가 가진 고유한 특성(feature) 정보를 모두 통합하여, 각 노드, 엣지, 또는 그래프 전체를 저차원의 밀집 벡터(dense vector)로 표현하는 것, 즉 '임베딩(embedding)'을 학습하는 것이다.1 이 임베딩은 다운스트림(downstream) 과제 수행에 필요한 풍부한 정보를 압축적으로 담고 있어야 한다. GNN은 이 임베딩을 활용하여 다양한 수준의 예측 과제를 수행한다 9:

- **노드 수준(Node-level) 예측:** 각 노드의 레이블이나 속성을 예측한다. 예를 들어, 인용 네트워크에서 각 논문의 연구 분야를 분류하거나, 소셜 네트워크에서 특정 사용자의 영향력을 예측하는 과제가 이에 해당한다.9
- **엣지 수준(Edge-level) 예측:** 두 노드 사이에 연결이 존재할 가능성을 예측한다. 단백질-단백질 상호작용 예측, 추천 시스템에서의 사용자-아이템 링크 예측, 지식 그래프 완성 등이 대표적인 예다.9
- **그래프 수준(Graph-level) 예측:** 그래프 전체의 속성이나 레이블을 예측한다. 분자 구조 그래프를 보고 해당 분자의 독성 여부를 판단하거나, 여러 소셜 네트워크 그래프를 커뮤니티의 유형별로 분류하는 과제가 여기에 속한다.2

이처럼 GNN은 그래프라는 보편적 언어로 표현된 데이터로부터 다층적인 수준의 통찰을 추출하는 강력하고 유연한 프레임워크를 제공한다.



다양한 GNN 아키텍처의 근간에는 '메시지 전달(Message Passing)'이라는 통일된 개념적 프레임워크가 자리 잡고 있다.11 이는 "한 노드의 특성은 그 이웃 노드들의 특성에 의해 결정된다"는 직관적인 아이디어에서 출발한다.1 만약 어떤 노드와 그 이웃 간의 모든 연결을 끊는다면, 그 노드는 고립되어 의미를 상실하게 된다. 따라서 노드의 개념은 이웃과의 연결 관계를 통해 정의된다.

메시지 전달 패러다임은 이 아이디어를 계산 가능한 프로세스로 구체화한 것이다. 각 노드는 자신의 이웃 노드들로부터 '메시지'를 수집하고, 이 메시지들을 종합하여 자기 자신의 상태, 즉 표현(representation) 벡터를 갱신한다.10 이 과정은 GNN의 여러 층(layer)에 걸쳐 반복적으로 수행된다. 한 번의 메시지 전달 과정은 크게 두 개의 핵심 단계로 구성된다 11:

1. **취합 (Aggregation):** 각 노드는 자신의 이웃 노드들이 가진 표현 벡터들을 하나의 벡터로 종합한다.
2. **갱신 (Update / Combine):** 취합된 이웃 정보와 노드 자신의 이전 상태 정보를 결합하여 새로운 상태를 계산한다.

이러한 메시지 전달 프레임워크는 GCN, GraphSAGE, GAT 등 겉보기에 서로 달라 보이는 수많은 GNN 모델들을 일관된 시각으로 이해할 수 있게 해주는 강력한 추상화 도구다. 각 모델의 차이는 결국 이 프레임워크 내에서 취합과 갱신 함수를 어떻게 구체적으로 설계했는가의 문제로 귀결된다.


메시지 전달 과정을 수학적으로 일반화하면 다음과 같이 표현할 수 있다. 그래프 $G=(V, E)$에서 $k$번째 GNN 층의 연산 과정을 노드 $u \in V$를 중심으로 살펴보자. 노드 $u$의 $k$번째 층에서의 은닉 상태(hidden state) 또는 임베딩을 $\mathbf{h}_u^{(k)}$라고 하자. 이웃 노드의 집합은 $\mathcal{N}(u)$로 표기한다.

먼저, **취합(AGGREGATE)** 단계에서는 노드 $u$의 이웃 노드 $v \in \mathcal{N}(u)$들의 이전 층($k-1$) 임베딩 ${\mathbf{h}*v^{(k-1)}}$을 입력받아 하나의 메시지 벡터 $\mathbf{m}*{\mathcal{N}(u)}^{(k)}$를 생성한다.11
$$
\mathbf{m}_{\mathcal{N}(u)}^{(k)} = \text{AGGREGATE}^{(k)}(\{\mathbf{h}_v^{(k-1)}, \forall v \in \mathcal{N}(u)\})
$$
여기서 `AGGREGATE` 함수는 이웃 노드들의 순서에 영향을 받지 않아야 하므로, 입력 순서에 불변(permutation invariant)인 함수여야 한다. 합(sum), 평균(mean), 최대값(max)과 같은 연산이 대표적으로 사용된다.10

다음으로, **갱신(UPDATE)** 단계에서는 취합된 메시지 $\mathbf{m}_{\mathcal{N}(u)}^{(k)}$와 노드 $u$ 자신의 이전 층 임베딩 $\mathbf{h}_u^{(k-1)}$을 결합하여 현재 층의 새로운 임베딩 $\mathbf{h}_u^{(k)}$를 계산한다.11
$$
\mathbf{h}_u^{(k)} = \text{UPDATE}^{(k)}(\mathbf{h}_u^{(k-1)}, \mathbf{m}_{\mathcal{N}(u)}^{(k)})
$$
`UPDATE` 함수는 일반적으로 다층 퍼셉트론(MLP)과 같은 신경망으로 구현되며, 비선형 활성화 함수($\sigma$)를 포함한다.11 초기 임베딩 $\mathbf{h}_u^{(0)}$는 주어진 노드 특성 벡터 $\mathbf{x}_u$로 설정된다.

이러한 추상적 개념을 미분 가능한 함수인 `AGGREGATE`와 `UPDATE`로 구체화함으로써, 전체 GNN 모델은 손실 함수에 대해 종단간(end-to-end)으로 학습될 수 있다. 이는 경사 하강법(gradient descent)과 역전파(backpropagation)를 통해 모델의 파라미터(예: `UPDATE` 함수 내 신경망의 가중치)를 최적화할 수 있음을 의미한다.


GNN에서 층을 하나 더 쌓는다는 것은 메시지 전달 과정을 한 번 더 수행하는 것을 의미한다. 이는 각 노드가 정보를 수집하는 범위, 즉 수용장(receptive field)을 확장시키는 효과를 가진다.12

- **1-Layer GNN:** 노드 $u$의 최종 임베딩 $\mathbf{h}_u^{(1)}$은 자신의 직접적인 이웃(1-홉 이웃)들의 초기 특성 $\mathbf{x}_v$만을 종합하여 계산된다.
- **2-Layer GNN:** 노드 $u$의 최종 임베딩 $\mathbf{h}_u^{(2)}$는 1-홉 이웃들의 갱신된 임베딩 $\mathbf{h}_v^{(1)}$을 종합하여 계산된다. 이때 $\mathbf{h}_v^{(1)}$은 $v$의 1-홉 이웃, 즉 $u$의 2-홉 이웃들의 정보를 담고 있다. 따라서 2-layer GNN에서 노드 $u$의 최종 임베딩은 자신의 2-홉 반경 내의 모든 노드와 구조 정보를 반영하게 된다.
- **K-Layer GNN:** 이 과정을 일반화하면, $K$개의 층을 가진 GNN에서 각 노드의 최종 임베딩은 자신의 $K$-홉 이웃까지의 정보를 종합한 결과가 된다.10

따라서 GNN의 깊이($K$)는 모델이 고려하는 지역적 이웃의 범위를 결정하는 중요한 하이퍼파라미터다. 너무 얕으면 충분한 구조적 정보를 포착하지 못할 수 있고, 너무 깊으면 후술할 과적합 평활화(over-smoothing)와 같은 문제가 발생할 수 있다.12 이 프레임워크의 유연성은 GNN 연구의 폭발적인 성장을 이끌었으며, 연구자들은 더 정교한 취합 함수나 더 안정적인 갱신 함수를 제안함으로써 GNN의 성능과 표현력을 지속적으로 개선하고 있다.


GNN 아키텍처의 발전사는 '어떻게 더 효율적이고, 더 확장 가능하며, 더 표현력 있게 이웃 정보를 취합할 것인가'에 대한 고민의 역사로 요약될 수 있다. 이는 GCN의 '고정된 가중치'에서 GraphSAGE의 '학습 가능한 함수'로, 그리고 GAT의 '입력에 따라 동적으로 변하는 가중치'로 진화하는 과정으로 나타난다. 본 장에서는 GNN 연구의 세 가지 핵심 축인 이론적 기반(GCN), 실용적 확장성(GraphSAGE), 모델 표현력(GAT)을 대표하는 주요 아키텍처들을 심층적으로 분석한다.


GCN은 GNN 분야에 큰 영향을 미친 선구적인 모델 중 하나로, 그래프 데이터에 컨볼루션 연산을 효과적으로 적용하는 방법을 제시했다.15


초기 GNN 연구는 스펙트럼 그래프 이론(spectral graph theory)에 기반을 두었다. 이 접근법은 그래프 라플라시안 행렬(Graph Laplacian Matrix)의 고유값 분해(eigen-decomposition)를 통해 그래프 푸리에 변환(Graph Fourier Transform)을 정의하고, 이를 이용해 주파수 영역(frequency domain)에서 컨볼루션 연산을 수행한다.2 하지만 이 방식은 몇 가지 심각한 한계를 가지고 있었다. 첫째, 라플라시안 행렬의 고유값 분해는 계산 복잡도가 

$O(N^3)$에 달해 노드 수가 많은 대규모 그래프에는 적용하기 어렵다. 둘째, 학습된 필터가 특정 그래프 구조에 종속적이어서, 학습 중에 보지 못한 새로운 노드나 그래프 구조에 일반화하기 어려운 전이적(transductive) 학습 방식이라는 점이다.16


이러한 한계를 극복하기 위해 Kipf와 Welling(2017)은 스펙트럼 그래프 컨볼루션을 체비셰프 다항식(Chebyshev polynomials)을 이용해 근사하고, 이를 다시 1차(first-order)로 단순화하여 계산적으로 매우 효율적이면서도 공간적(spatial) 해석이 가능한 모델을 제안했다.16 이 모델이 바로 오늘날 널리 알려진 GCN이다.

GCN의 핵심적인 층별 전파 규칙(layer-wise propagation rule)은 다음과 같은 행렬 형태로 표현된다 16:
$$
\mathbf{H}^{(l+1)} = \sigma(\tilde{\mathbf{D}}^{-\frac{1}{2}} \tilde{\mathbf{A}} \tilde{\mathbf{D}}^{-\frac{1}{2}} \mathbf{H}^{(l)} \mathbf{W}^{(l)})
$$
각 구성 요소의 의미는 다음과 같다:

- $\mathbf{H}^{(l)} \in \mathbb{R}^{N \times F_l}$: $l$번째 층의 노드 특성 행렬. $N$은 노드의 수, $F_l$은 특성의 차원이다. 초기 입력은 $\mathbf{H}^{(0)} = \mathbf{X}$이다.
- $\mathbf{W}^{(l)} \in \mathbb{R}^{F_l \times F_{l+1}}$: $l$번째 층의 학습 가능한 가중치 행렬로, 특성을 변환하는 역할을 한다.
- $\tilde{\mathbf{A}} = \mathbf{A} + \mathbf{I}_N$: 인접 행렬 $\mathbf{A}$에 단위 행렬 $\mathbf{I}_N$을 더한 것. 이는 각 노드에 자기 자신으로의 연결(self-loop)을 추가하는 것을 의미한다. 이 과정이 없으면, 노드 표현을 갱신할 때 이웃의 정보만 사용하고 자신의 이전 정보를 잃게 되는 문제를 방지할 수 있다.3
- $\tilde{\mathbf{D}}$: $\tilde{\mathbf{A}}$의 차수 행렬(degree matrix)로, $\tilde{D}*{ii} = \sum_j \tilde{A}*{ij}$인 대각 행렬이다.
- $\tilde{\mathbf{D}}^{-\frac{1}{2}} \tilde{\mathbf{A}} \tilde{\mathbf{D}}^{-\frac{1}{2}}$: 대칭 정규화(symmetric normalization)된 인접 행렬이다. 이 정규화는 차수가 높은 노드로부터 오는 메시지의 영향력을 줄여주고, 차수가 낮은 노드로부터 오는 메시지의 영향력을 높여줌으로써 숫자적 불안정성을 해소하고 학습을 안정시키는 중요한 역할을 한다.13
- $\sigma(\cdot)$: ReLU와 같은 비선형 활성화 함수다.

메시지 전달 관점에서 GCN의 연산은 각 노드가 자신의 모든 이웃과 자기 자신의 표현을 정규화된 차수로 가중 평균하여 취합($\tilde{\mathbf{D}}^{-\frac{1}{2}} \tilde{\mathbf{A}} \tilde{\mathbf{D}}^{-\frac{1}{2}} \mathbf{H}^{(l)}$)한 뒤, 이를 선형 변환($\mathbf{W}^{(l)}$)하고 비선형 활성화($\sigma$)를 통해 갱신하는 과정으로 해석할 수 있다.14 GCN은 그 단순함과 효율성 덕분에 많은 GNN 연구의 기초가 되었지만, 학습 과정에서 전체 그래프의 인접 행렬을 필요로 하므로 본질적으로 전이적 학습에 국한된다는 한계를 여전히 가지고 있다.


GCN의 전이적 학습 방식은 학습 중에 보지 못한 새로운 노드가 등장하거나 그래프 구조 자체가 계속 변하는 동적 환경(예: 유튜브, 레딧)에서는 실용적이지 않다.20 GraphSAGE(Graph SAmple and aggreGatE)는 이러한 한계를 극복하고 GNN을 대규모 귀납적(inductive) 학습 환경에 적용하기 위해 제안되었다.20


GraphSAGE의 핵심 철학은 개별 노드의 임베딩을 직접 학습하는 대신, 노드의 지역적 이웃으로부터 특징 정보를 '샘플링'하고 '취합'하여 임베딩을 생성하는 함수를 학습하는 것이다.20 이렇게 학습된 임베딩 생성 함수(aggregator function)는 학습 데이터에 없었던 새로운 노드나 완전히 새로운 그래프에도 적용하여 임베딩을 생성할 수 있다. 이것이 GraphSAGE가 귀납적 학습 능력을 갖는 이유다.


실제 소셜 네트워크와 같은 대규모 그래프에서 일부 노드는 수백만 개의 이웃을 가질 수 있다. 모든 이웃의 정보를 매번 취합하는 것은 계산적으로 비효율적이다. GraphSAGE는 이 문제를 해결하기 위해 각 노드에서 고정된 크기($k$)의 이웃을 균등하게(uniformly) 샘플링하여 사용한다.20 이를 통해 각 노드에 대한 계산량을 일정하게 유지하고, 미니배치(mini-batch) 학습을 가능하게 하여 확장성을 크게 향상시킨다.

GraphSAGE는 다양한 종류의 취합 함수(aggregator architecture)를 제안하여 유연성을 높였다 20:

1. **Mean Aggregator:** 샘플링된 이웃 노드 표현들의 원소별 평균을 계산한다. 이는 GCN의 컨볼루션 연산을 귀납적이고 샘플링 기반으로 변형한 것으로 볼 수 있다.14
2. **LSTM Aggregator:** 이웃 노드들을 임의의 순서로 나열하여 LSTM의 입력으로 사용한다. LSTM은 순서에 민감한 모델이지만, 다양한 순서로 학습함으로써 순서 불변성을 근사적으로 학습할 수 있으며, 더 높은 표현력을 가진다.14
3. **Pooling Aggregator:** 각 이웃의 표현 벡터를 독립적인 MLP(다층 퍼셉트론)에 통과시켜 변환한 후, 변환된 벡터들에 대해 원소별 최대값 풀링(element-wise max-pooling)을 적용한다. 이 방식은 대칭적이면서(순서 불변) 학습 가능한 파라미터를 통해 어떤 이웃 정보가 중요한지를 모델이 스스로 학습하게 한다.23


취합 단계 이후, GraphSAGE의 갱신 단계는 다음과 같다. $k$번째 층에서 노드 $v$의 새로운 표현 $\mathbf{h}*v^{(k)}$는, 취합된 이웃 벡터 $\mathbf{h}*{\mathcal{N}(v)}^{(k)}$와 노드 $v$ 자신의 이전 층 표현 $\mathbf{h}_v^{(k-1)}$을 결합(concatenation)한 후, 이를 가중치 행렬 $\mathbf{W}^{(k)}$로 선형 변환하고 비선형 활성화 함수 $\sigma$를 적용하여 계산된다.20
$$
\mathbf{h}_v^{(k)} = \sigma(\mathbf{W}^{(k)} \cdot \text{CONCAT}(\mathbf{h}_v^{(k-1)}, \mathbf{h}_{\mathcal{N}(v)}^{(k)}))
$$
여기서 `CONCAT` 연산은 매우 중요하다. 이는 노드 자신의 이전 정보를 명시적으로 다음 층으로 전달하는 일종의 잔차 연결(skip connection) 역할을 하여, 정보 손실을 막고 깊은 모델의 학습을 돕는다.24 GraphSAGE는 이러한 귀납적 설계와 확장성 덕분에 GNN을 실제 산업 현장의 대규모 동적 그래프에 적용 가능한 실용적인 기술로 발전시켰다.


GCN과 GraphSAGE의 Mean Aggregator는 이웃 노드의 정보를 취합할 때 모든 이웃에게 동일한(또는 차수에 의해 결정된 고정된) 중요도를 부여한다. 하지만 현실에서는 특정 노드에게 있어 모든 이웃이 동등하게 중요하지 않을 수 있다. 그래프 어텐션 네트워크(GAT)는 이러한 통찰을 바탕으로, 어텐션 메커니즘(attention mechanism)을 도입하여 이웃 노드들에게 서로 다른 중요도를 동적으로 할당하는 방법을 제안했다.2


GAT의 핵심은 마스크드 셀프 어텐션(masked self-attention)이다. 이는 노드가 자신의 이웃 노드들에 대해서만 어텐션을 계산하도록 제한하는 방식이다. GAT의 한 층에서 노드 $i$의 표현 $\mathbf{h}_i$가 $\mathbf{h}'_i$로 갱신되는 과정은 다음과 같다 25:

1. **선형 변환:** 모든 노드의 입력 특성 $\mathbf{h}_i \in \mathbb{R}^F$에 공유 가중치 행렬 $\mathbf{W} \in \mathbb{R}^{F' \times F}$를 곱하여 고차원의 표현 $\mathbf{W}\mathbf{h}_i$를 얻는다. 이는 모델의 표현력을 높이기 위한 과정이다.25

2. **어텐션 계수 계산:** 노드 $i$와 그 이웃 $j \in \mathcal{N}*i$에 대해, 어텐션 계수 $e*{ij}$를 계산한다. 이는 노드 $j$의 특성이 노드 $i$에게 얼마나 중요한지를 나타내는 점수다. $e_{ij}$는 두 노드의 변환된 특성 $\mathbf{W}\mathbf{h}_i$와 $\mathbf{W}\mathbf{h}_j$를 결합(concatenation)하고, 학습 가능한 가중 벡터 $\mathbf{a} \in \mathbb{R}^{2F'}$와 내적한 후 LeakyReLU 활성화 함수를 적용하여 계산된다.
   $$
   e_{ij} = \text{LeakyReLU}(\mathbf{a}^T)
   $$

3. **정규화:** 계산된 어텐션 점수 $e_{ij}$를 노드 $i$의 모든 이웃 $k \in \mathcal{N}*i$에 대해 소프트맥스(softmax) 함수로 정규화하여 최종 어텐션 가중치 $\alpha*{ij}$를 얻는다.
   $$
   \alpha_{ij} = \text{softmax}_j(e_{ij}) = \frac{\exp(e_{ij})}{\sum_{k \in \mathcal{N}_i} \exp(e_{ik})}
   $$

4. **가중합 및 갱신:** 계산된 어텐션 가중치 $\alpha_{ij}$를 사용하여 이웃 노드들의 변환된 특성을 가중합(weighted sum)하고, 여기에 비선형 활성화 함수 $\sigma$를 적용하여 노드 $i$의 최종 출력 표현 $\mathbf{h}'_i$를 얻는다.
   $$
   \mathbf{h}'_i = \sigma\left(\sum_{j \in \mathcal{N}_i} \alpha_{ij} \mathbf{W}\mathbf{h}_j\right)
   $$


GAT는 학습 과정을 안정화하고 모델의 표현력을 풍부하게 만들기 위해 멀티 헤드 어텐션(multi-head attention)을 사용한다.25 이는 $K$개의 독립적인 어텐션 메커니즘(헤드)을 병렬로 수행한 후, 각 헤드에서 나온 결과 벡터들을 결합하는 방식이다. 중간 층에서는 보통 결합(concatenation)을 사용하고, 마지막 출력 층에서는 평균(averaging)을 사용한다.
$$
\mathbf{h}'_i = \Big\Vert_{k=1}^K \sigma\left(\sum_{j \in \mathcal{N}_i} \alpha_{ij}^k \mathbf{W}^k \mathbf{h}_j\right)
$$
GAT는 인접 행렬을 명시적으로 사용하지 않고 어텐션 계수를 통해 이웃의 가중치를 학습하므로, GCN과 달리 계산 비용이 높고 귀납적 학습이 가능하다는 장점이 있다. 어텐션 메커니즘을 통해 각 노드가 주변의 어떤 정보에 더 집중해야 하는지를 스스로 학습하게 함으로써, GNN의 표현력과 유연성을 한 단계 끌어올렸다.25


아래 표는 세 가지 핵심 아키텍처의 특징을 요약하여 비교한 것이다. 이 표는 각 모델의 철학, 장단점, 적용 분야를 명확히 구분함으로써, 특정 문제에 어떤 모델이 더 적합할지 판단하는 데 도움을 준다.

| 특징                       | 그래프 컨볼루션 네트워크 (GCN)            | GraphSAGE                                   | 그래프 어텐션 네트워크 (GAT)              |
| -------------------------- | ----------------------------------------- | ------------------------------------------- | ----------------------------------------- |
| **핵심 아이디어**          | 스펙트럼 그래프 컨볼루션의 공간적 근사    | 이웃 샘플링을 통한 귀납적 임베딩 함수 학습  | 어텐션 메커니즘을 통한 이웃의 중요도 가중 |
| **학습 방식**              | 전이적 (Transductive)                     | 귀납적 (Inductive)                          | 귀납적 (Inductive)                        |
| **취합(Aggregation) 방식** | 정규화된 차수에 의한 가중 평균            | 다양한 함수 선택 가능 (Mean, LSTM, Pooling) | 학습된 어텐션 가중치에 의한 가중합        |
| **이웃 처리**              | 모든 1-홉 이웃 사용                       | 고정된 수의 이웃 샘플링                     | 모든 1-홉 이웃 사용                       |
| **확장성**                 | 대규모 그래프에 한계                      | 대규모 그래프에 효율적                      | GCN보다 계산 비용 높음                    |
| **장점**                   | 단순하고 효율적, 이론적 기반이 명확함     | 확장성, 새로운 노드에 대한 일반화 능력      | 높은 표현력, 이웃별 중요도 학습           |
| **단점**                   | 귀납적 학습 불가, 이웃에 동일 가중치 부여 | 샘플링으로 인한 정보 손실 가능성            | 계산 복잡도 높음, 멀티 헤드 필요          |


초기 GNN 모델들의 성공 이후, 연구자들은 모델의 근본적인 능력과 한계를 이해하고 이를 극복하기 위한 원칙적인 방법을 모색하는 성숙한 단계로 접어들었다. GNN의 이론적 과제들은 모델의 '깊이(depth)', '너비(width)', '범위(scope)'라는 세 가지 차원에서의 근본적인 트레이드오프를 드러낸다. 본 장에서는 과적합 평활화, 표현력의 한계, 확장성 등 GNN이 직면한 주요 이론적 과제들과 이를 해결하기 위한 노력들을 살펴본다.



과적합 평활화(over-smoothing)는 GNN의 층을 깊게 쌓을수록, 반복적인 이웃 정보 취합 과정으로 인해 서로 다른 클러스터에 속한 노드들의 임베딩조차 점차 유사해져 결국 구별할 수 없게 되는 현상을 말한다.12 이는 깊은 GNN 모델의 학습을 방해하고 성능을 저해하는 가장 심각하고 고질적인 문제 중 하나다.

이 현상의 근본적인 원인은 GNN의 메시지 전달 메커니즘 자체에 내재되어 있다. GCN의 층별 전파 규칙은 본질적으로 그래프 라플라시안을 반복적으로 적용하는 것과 유사하며, 이는 신호 처리 관점에서 저주파 통과 필터(low-pass filter) 역할을 한다.15 즉, 메시지 전달을 거듭할수록 노드 간의 차이를 나타내는 고주파 신호는 점차 사라지고, 전체 그래프에 걸쳐 평탄하고 부드러운 저주파 신호만 남게 된다. 이로 인해 모든 노드 표현이 특정 값으로 수렴하게 된다.29 놀랍게도, 동적인 가중치를 사용하는 어텐션 기반 모델인 GAT 역시 이 문제에서 자유롭지 않으며, 모델의 깊이가 깊어짐에 따라 표현력이 기하급수적으로 감소할 수 있다는 것이 이론적으로 증명되었다.32


과적합 평활화의 정도를 정량적으로 측정하기 위해 MAD(Mean Average Distance)와 같은 지표가 제안되었다. MAD는 그래프 내 노드 임베딩들 간의 평균 거리를 계산하며, 이 값이 작아질수록 평활화가 심해졌음을 의미한다.30 또한, 이 문제를 정보 대 잡음비(information-to-noise ratio) 관점에서 분석하기도 한다. 층이 깊어질수록 각 노드는 더 넓은 범위의 이웃으로부터 메시지를 받게 되는데, 이 과정에서 유용한 정보뿐만 아니라 불필요한 잡음까지 섞이게 되어 결국 노드의 고유한 특성이 희석된다는 것이다.30

과적합 평활화 문제를 완화하기 위해 다양한 해결 방안이 제시되었다:

- **아키텍처 수정:**
  - **잔차 연결 (Residual/Skip Connections):** ResNet에서 영감을 받은 기법으로, 각 층의 입력 정보를 출력에 직접 더해줌으로써 정보의 소실을 막고 깊은 모델의 학습을 돕는다.12
  - **Gated GNNs:** GRU나 LSTM과 유사한 게이트 메커니즘을 도입하여 정보의 흐름을 조절하고 장기 의존성을 학습하는 능력을 향상시킨다.12
  - **Jumping Knowledge Networks:** 얕은 층과 깊은 층의 표현을 모두 최종 층에서 결합(concatenation, max-pooling 등)하여, 각 노드가 작업에 가장 적합한 이웃 범위를 선택할 수 있도록 한다.14
- **정규화 및 드롭아웃 변형:**
  - **DropEdge:** 학습 과정에서 무작위로 간선을 제거하여 메시지 전달 경로를 다양화하고, 과도한 정보 집계를 막아 과적합 평활화를 방지한다.29
  - **PairNorm:** 각 층의 출력에서 모든 노드 표현의 총 분산을 일정하게 유지하도록 정규화하여, 표현이 한 점으로 수렴하는 것을 방지한다.
- **그래프 구조 수정:**
  - **AdaEdge / MADReg:** 모델의 예측이나 MADGap과 같은 평활화 지표를 기반으로 그래프의 토폴로지를 동적으로 최적화한다. 예를 들어, 다른 클래스에 속할 것으로 예측되는 노드 간의 엣지를 제거하거나 같은 클래스로 예측되는 노드 간의 엣지를 추가하여 정보 대 잡음비를 높인다.30


GNN이 얼마나 복잡하고 미세한 그래프 구조의 차이를 구분해낼 수 있는가 하는 문제는 '표현력(expressive power)'과 관련이 있다.


표준적인 메시지 전달 기반 GNN의 표현력 상한은 1차원 바이스파일러-레만(1-dimensional Weisfeiler-Lehman, 1-WL) 그래프 동형성 테스트(isomorphism test)와 동등하다는 것이 이론적으로 증명되었다.35 1-WL 테스트는 두 그래프가 동일한 구조를 가졌는지(동형인지) 판별하기 위한 고전적인 휴리스틱 알고리즘이다. 이 테스트는 각 노드에 초기 레이블(색상)을 부여하고, 반복적으로 각 노드의 이웃 노드들이 가진 레이블의 다중집합(multiset)을 수집하여 해시(hash)한 값으로 자신의 레이블을 갱신한다. 이 과정을 더 이상 레이블 변화가 없을 때까지 반복한 후, 최종 레이블들의 히스토그램을 비교하여 두 그래프의 동형 여부를 판단한다.

GNN의 메시지 전달 과정은 이 1-WL 테스트의 레이블 갱신 과정과 매우 유사하다. GNN의 취합 함수는 이웃 레이블의 다중집합을 수집하는 과정에, 갱신 함수는 해싱을 통해 새로운 레이블을 만드는 과정에 대응된다. 따라서 1-WL 테스트가 구분하지 못하는 비동형(non-isomorphic) 그래프 쌍은 표준 GNN 역시 구분할 수 없다.36 예를 들어, 1-WL 테스트는 많은 정규 그래프(regular graph)나 단순히 사이클의 존재 여부만 다른 그래프들을 구분하지 못하는데, 이는 GNN 역시 이러한 구조적 차이를 감지하지 못할 수 있음을 의미한다.


1-WL 테스트의 한계를 넘어서는 더 강력한 GNN을 만들기 위한 연구가 활발히 진행되고 있다:

- **고차원 GNN (Higher-order GNNs):** 더 강력한 k-WL 테스트(노드의 k-튜플에 레이블을 할당)에 영감을 받아, 메시지 전달의 단위를 개별 노드가 아닌 노드의 집합(예: 엣지, 삼각형, 모티프)으로 확장하는 방식이다.38
- **서브그래프 기반 GNN:** 각 노드를 중심으로 하는 서브그래프를 추출하고, 이 서브그래프 전체에 대한 표현을 학습하여 노드의 임베딩으로 사용하는 방식이다. 이는 GNN이 지역적 구조를 더 명시적으로 인코딩하도록 돕는다.35
- **위치 및 구조 인코딩 (Positional & Structural Encoding):** 그래프 내에서 노드의 위치(예: 그래프 라플라시안의 고유벡터)나 구조적 역할(예: 특정 모티프에 속하는지 여부)에 대한 정보를 노드의 초기 특성에 추가하여 GNN의 표현력을 보강한다.39



실세계의 그래프(예: 페이스북 소셜 그래프, 아마존 상품 추천 그래프)는 수십억 개의 노드와 수백억 개의 엣지를 가질 수 있다. 이러한 대규모 그래프에 GNN을 적용할 때, 메시지 전달 과정에서 각 노드의 모든 이웃을 탐색하고 집계하는 것은 막대한 계산 및 메모리 비용을 유발한다. 특히 층이 깊어질수록 탐색해야 할 이웃의 수가 기하급수적으로 증가하는 '이웃 폭발(neighbor explosion)' 문제가 발생한다.40

이를 해결하기 위한 주요 접근법은 다음과 같다:

- **샘플링 기반 방법:** GraphSAGE와 같이, 각 노드에서 모든 이웃 대신 고정된 수의 이웃을 샘플링하여 계산량을 제어한다.41
- **클러스터링 기반 방법:** Cluster-GCN과 같이, 그래프를 여러 개의 조밀한 클러스터로 분할한 뒤, 각 클러스터를 하나의 미니배치로 사용하여 학습한다.41
- **사전 계산 및 단순화:** GNN의 복잡한 연산을 단순한 선형 전파로 바꾸고, 이 전파 과정을 미리 계산하여 저장해 둠으로써 학습 시에는 간단한 MLP만 학습시키는 방법(예: SGC)도 있다.15


대부분의 초기 GNN 모델은 '동종친화성(Homophily)'이라는 암묵적인 가정, 즉 "끼리끼리 모인다(birds of a feather flock together)"는 원칙에 기반한다. 이는 연결된 노드들은 서로 유사한 특성이나 레이블을 가질 가능성이 높다는 가정이다.42 인용 네트워크나 친구 관계 네트워크는 이러한 가정이 잘 들어맞는 대표적인 예다.

그러나 현실에는 반대로 "서로 다른 것들이 끌리는(opposites attract)" 이종친화성(Heterophily) 그래프도 많이 존재한다. 예를 들어, 단백질 상호작용 네트워크나 이성 간의 데이팅 네트워크에서는 서로 다른 종류의 노드들이 연결되는 경우가 많다.42 이러한 그래프에서 표준 GNN은 이웃의 정보를 무분별하게 평균 내기 때문에, 오히려 노드의 고유한 특성을 희석시켜 성능이 급격히 저하된다. 이종친화성 문제를 해결하기 위해, 2-홉 이상의 더 넓은 범위의 이웃을 고려하거나, 노드 자신과 이웃의 표현을 분리하여 처리하는 아키텍처(예: H2GCN), 그래프 필터를 이종친화성에 맞게 재설계하는 등의 방법이 연구되고 있다.42



레이블이 부족한 대규모 그래프에서 유용한 표현을 학습하기 위해, 그래프 자체의 내재적 속성을 활용하여 인위적인 지도 신호(supervisory signal)를 생성하고 이를 바탕으로 GNN을 사전 학습(pre-training)하는 패러다임이다.44 주요 전략은 다음과 같다:

- **대조 학습 (Contrastive Learning):** 그래프 데이터에 무작위 변형(augmentation, 예: 노드 드롭, 엣지 섭동)을 가하여 두 개의 뷰(view)를 생성한다. 그 후, 같은 노드가 두 뷰에서 가진 표현은 서로 가깝게, 다른 노드의 표현과는 멀어지도록 학습한다.47
- **예측 학습 (Predictive Learning):** 그래프의 일부 정보(예: 마스킹된 노드 특성, 엣지 존재 여부)를 예측하는 과제를 통해 표현을 학습한다.


GNN의 지역적 메시지 전달 방식이 가진 과적합 평활화, 과압축(over-squashing, 장거리 정보를 병목 구간을 통해 전달하며 정보 손실 발생) 등의 한계를 극복하기 위해 제안되었다.39 트랜스포머의 핵심인 셀프 어텐션 메커니즘을 그래프의 모든 노드 쌍에 적용하여, 모든 노드가 다른 모든 노드와 직접 상호작용하도록 한다. 이는 장거리 의존성을 포착하는 데 매우 효과적이다.49 그래프의 원래 구조 정보는 메시지 전달 경로로 사용되는 대신, 노드의 위치나 구조적 역할을 나타내는 인코딩(positional/structural encoding) 형태로 모델에 주입된다.51

이러한 이론적 과제들과 새로운 패러다임의 등장은 GNN 연구가 단순한 성능 경쟁을 넘어, 모델의 근본적인 능력과 한계를 이해하고 이를 극복하기 위한 원칙적인 방법을 모색하는 방향으로 나아가고 있음을 보여준다.


GNN의 강력한 표현력은 '데이터의 본질이 관계에 있는' 모든 문제로 그 응용 범위를 확장시키고 있다. GNN은 개별 객체의 특성을 넘어 객체 간의 상호작용과 구조를 학습의 중심에 둠으로써, 기존 기계 학습 방법론이 포착하지 못했던 고차원적인 맥락 정보를 활용하는 새로운 분석의 창을 열었다. 본 장에서는 추천 시스템, 신약 개발, 이상 탐지 등 GNN이 각 분야의 근본적인 문제 해결 방식을 어떻게 바꾸고 있는지를 대표적인 사례를 통해 살펴본다.


추천 시스템은 정보 과부하 시대에 사용자가 원하는 콘텐츠, 상품, 서비스를 효과적으로 찾을 수 있도록 돕는 핵심 기술이다.53 전통적인 추천 시스템은 주로 사용자-아이템 상호작용 행렬에 기반한 협업 필터링(Collaborative Filtering) 기법을 사용했다. 그러나 이러한 접근은 데이터 희소성(sparsity)과 신규 사용자/아이템에 대한 추천이 어려운 콜드 스타트(cold-start) 문제에 취약했다.

GNN은 이러한 문제에 대한 자연스러운 해법을 제시한다. 사용자-아이템 상호작용 데이터를 사용자와 아이템을 두 종류의 노드로 하고, 상호작용(클릭, 구매, 평점 등)을 엣지로 하는 이분 그래프(bipartite graph)로 모델링할 수 있다.5 또한, 사용자 간의 친구 관계는 소셜 그래프로 표현 가능하다.55 GNN은 이 그래프 구조 위에서 메시지 전달을 수행함으로써 다음과 같은 이점을 얻는다:

- **고차원 협업 신호 포착:** 메시지 전달을 여러 층에 걸쳐 수행함으로써, GNN은 '내 친구가 좋아한 상품', '내가 좋아한 상품과 유사한 상품을 구매한 다른 사용자'와 같은 단순한 관계를 넘어, '내 친구의 친구가 좋아한 상품과 유사한 카테고리의 상품'과 같은 다중 홉(multi-hop)에 걸친 복잡하고 고차원적인 협업 신호를 명시적으로 학습할 수 있다.53
- **콜드 스타트 문제 완화:** 상호작용 기록이 거의 없는 신규 사용자라 할지라도, 소셜 그래프 상에서 친구 관계가 있다면 친구들의 선호도 정보가 메시지 전달을 통해 전파되어 의미 있는 초기 임베딩을 생성할 수 있다. 마찬가지로 신규 아이템도 다른 유사 아이템과의 관계를 통해 표현을 학습할 수 있다.53
- **부가 정보의 자연스러운 통합:** 사용자의 인구통계학적 정보나 아이템의 속성 정보(예: 영화의 장르, 상품의 카테고리)를 노드의 초기 특성으로 활용하여, 콘텐츠 기반 필터링과 협업 필터링을 자연스럽게 결합할 수 있다.57

NGCF(Neural Graph Collaborative Filtering), LightGCN, GraphRec과 같은 모델들은 GNN을 추천 시스템에 성공적으로 적용하여 기존 방법론 대비 뛰어난 성능을 입증한 대표적인 사례다.5


신약 개발은 후보 물질 발굴부터 임상 시험까지 막대한 시간과 비용이 소요되는 매우 어려운 과정이다.58 이 과정의 초기 단계에서 컴퓨터를 이용한 *in silico* 스크리닝을 통해 유망한 후보 물질을 효율적으로 예측하고 선별하는 것은 매우 중요하다.

분자 구조는 GNN을 적용하기에 가장 이상적인 데이터 형태 중 하나다. 분자는 원자를 노드로, 화학 결합을 엣지로 하는 그래프로 완벽하게 표현될 수 있다.59 각 원자의 종류, 전하량, 수소 원자 수 등은 노드 특성이 되고, 결합의 종류(단일, 이중, 삼중 결합)나 방향성 등은 엣지 특성이 된다. GNN은 이러한 분자 그래프로부터 직접 표현을 학습하여(end-to-end learning), 수작업으로 설계된 분자 지문(molecular fingerprint)에 의존하던 기존 방식의 한계를 극복한다.

GNN은 신약 개발의 다양한 단계에서 활용된다:

- **분자 특성 예측 (Molecular Property Prediction):** GNN은 분자 그래프를 입력받아 해당 분자의 물리화학적, 생물학적 특성(예: 용해도, 독성, 생체이용률, 특정 단백질과의 결합 친화도)을 예측하는 회귀 또는 분류 문제를 해결한다. 예를 들어, GNN을 이용해 HIV 바이러스의 활동을 억제할 가능성이 있는 분자를 예측하는 연구가 성공적으로 수행되었다.58
- **약물-약물 상호작용 (Drug-Drug Interaction, DDI) 예측:** 두 가지 이상의 약물을 함께 복용했을 때 발생할 수 있는 시너지 효과나 부작용을 예측하는 것은 매우 중요하다. GNN은 각 약물 분자 그래프로부터 임베딩을 학습한 뒤, 이들을 결합하여 두 약물 간의 상호작용을 예측하는 데 사용된다.61
- **신약 후보 물질 생성 (Molecule Generation):** 변이형 오토인코더(VAE)나 생성적 적대 신경망(GAN)과 같은 생성 모델과 GNN을 결합하여, 특정 질병 치료에 효과적일 것으로 기대되는 새로운 분자 구조를 설계하고 생성하는 데 활용된다. 이는 신약 개발의 탐색 공간을 획기적으로 넓힐 수 있는 잠재력을 가진다.58


금융 사기는 종종 단독 행위가 아니라, 여러 개의 도용된 계정, 장치, 가짜 신원 등이 복잡하게 얽힌 네트워크 형태로 발생한다.64 사기꾼들은 개별 거래를 정상적으로 보이게 위장하여 기존의 규칙 기반 시스템이나 개별 거래 분석 모델을 우회하려 시도한다.

GNN은 이러한 관계형 사기 패턴을 탐지하는 데 매우 강력한 도구다. 금융 거래 데이터를 계정, 사용자, IP 주소, 기기 ID 등을 노드로 하고, 자금 이체, 동일 기기 사용, 동일 주소지 공유 등을 엣지로 하는 거대한 이종 그래프(heterogeneous graph)로 구성할 수 있다. GNN은 이 그래프 위에서 메시지 전달을 통해 각 노드의 '정상적인' 행동 패턴과 그 주변 네트워크 구조의 특징을 학습한다.67

GNN 기반 이상 탐지 시스템의 장점은 다음과 같다:

- **정교한 사기 패턴 탐지:** GNN은 여러 단계를 거치는 자금 세탁 고리, 신용카드 부정 사용을 위한 공모 네트워크, 보험 사기 집단 등 개별적으로는 파악하기 어려운 복잡한 사기 수법을 효과적으로 발견할 수 있다. 노드 임베딩은 해당 노드뿐만 아니라 그 주변의 네트워크 구조 정보까지 압축하고 있기 때문이다.66
- **오탐(False Positive) 감소:** 개별 거래의 특성(예: 고액 거래, 낯선 지역에서의 거래)만으로 판단하는 것이 아니라, 해당 거래가 전체 네트워크의 맥락 속에서 얼마나 이질적인지를 종합적으로 판단한다. 이로 인해 정상적인 거래를 사기로 잘못 판단하는 경우를 줄여 사용자 경험을 해치지 않으면서도 탐지 정확도를 높일 수 있다.66
- **실시간 탐지:** GraphSAGE와 같은 샘플링 기반 GNN과 효율적인 그래프 데이터베이스(예: Amazon Neptune)를 결합하면, 새로운 거래가 발생할 때마다 실시간으로 관련 서브그래프를 추출하고 추론하여 거의 실시간에 가까운 사기 탐지가 가능하다.65

이 세 가지 응용 분야는 GNN이 어떻게 각 도메인의 근본적인 문제 해결 방식을 관계 중심적 사고로 전환시키고 있는지를 보여주는 대표적인 사례다. 이는 GNN이 단순한 예측 정확도 향상을 넘어, 문제 자체를 재정의하고 새로운 분석적 접근을 가능하게 하는 혁신적인 기술임을 증명한다.



본 보고서는 그래프 신경망(GNN)의 기본 원리부터 최신 아키텍처, 핵심적인 이론적 과제, 그리고 광범위한 응용 분야에 이르기까지 심층적인 고찰을 제공했다. GNN은 '메시지 전달'이라는 통일된 프레임워크를 통해, 불규칙하고 복잡한 비유클리드 공간의 데이터를 직접 처리하는 딥러NING의 새로운 지평을 열었다. GCN, GraphSAGE, GAT와 같은 주요 아키텍처들은 각각 이론적 기반, 확장성, 표현력 측면에서 GNN의 발전을 이끌어온 핵심적인 이정표다.

그러나 GNN의 발전은 과적합 평활화, 표현력의 한계, 확장성 문제와 같은 중대한 이론적 과제들과의 끊임없는 씨름의 과정이었다. 이러한 도전들은 잔차 연결, 어텐션 메커니즘, 이웃 샘플링과 같은 정교한 해결책들을 낳았으며, 나아가 자기 지도 학습이나 그래프 트랜스포머와 같은 새로운 패러다임의 등장을 촉진했다. 추천 시스템, 신약 개발, 금융 사기 탐지와 같은 응용 분야에서 GNN은 기존 방법론의 한계를 돌파하며, 관계와 구조에 내재된 정보를 활용하는 새로운 문제 해결 방식을 제시함으로써 그 실용적 가치를 입증했다.

결론적으로, GNN은 관계형 데이터가 존재하는 모든 과학 및 산업 분야에 적용될 수 있는 범용 기술로서의 잠재력을 가지고 있으며, 이는 사회학, 화학, 물류, 금융 등 다양한 영역에 걸쳐 심대한 영향을 미치고 있다.


GNN 분야는 여전히 빠르게 발전하고 있으며, 해결해야 할 많은 과제와 탐구해야 할 새로운 연구 방향이 존재한다.

- **동적 및 시공간 그래프 (Dynamic and Spatio-Temporal Graphs):** 현실 세계의 많은 그래프는 고정되어 있지 않고 시간이 지남에 따라 노드와 엣지가 생성되거나 사라진다. 이러한 동적인 구조 변화를 효과적으로 모델링하고, 공간적 정보와 시간적 정보를 함께 처리하는 GNN(STGNNs)에 대한 연구가 더욱 중요해질 것이다.2
- **설명가능성 (Explainability, XAI):** GNN 모델이 특정 예측(예: '이 거래는 사기다', '이 분자는 독성이 있다')을 내린 이유를 인간이 이해할 수 있는 방식으로 설명하는 기술은 모델의 신뢰성과 수용성을 높이는 데 필수적이다. 특히 금융, 의료와 같이 결정에 대한 책임이 중요한 분야에서 핵심적인 연구 주제가 될 것이다.53
- **공정성 (Fairness):** 소셜 네트워크 분석이나 대출 심사와 같은 응용 분야에서 GNN이 특정 인구통계학적 그룹에 대해 편향된 예측을 내리는 것을 방지하고, 공정한 결과를 도출하기 위한 알고리즘적 연구가 필요하다. 이는 모델의 사회적 책임을 다하기 위한 중요한 과제다.53
- **이론적 기반 강화:** 현재 GNN의 많은 성공은 경험적인 발견에 의존하고 있다. 과적합 평활화, 표현력, 일반화 능력 등에 대한 더 깊고 일반화된 이론적 분석은 GNN의 작동 원리를 근본적으로 이해하고, 더 원칙적인 모델 설계를 가능하게 할 것이다.
- **거대 모델로의 확장 (Graph Foundation Models):** 자연어 처리 분야의 거대 언어 모델(LLM)과 같이, 레이블이 없는 방대한 그래프 데이터에 대해 사전 학습된 후, 다양한 다운스트림 태스크에 파인튜닝(fine-tuning)하여 사용될 수 있는 '그래프 파운데이션 모델(GFM)'에 대한 연구가 활발해지고 있다. 이는 GNN의 일반화 능력과 재사용성을 극대화할 수 있는 유망한 방향이다.70


그래프 신경망은 딥러닝과 그래프 이론의 성공적인 융합을 통해, 관계형 데이터 분석의 새로운 시대를 열었다. 앞으로 GNN은 더욱 복잡하고, 동적이며, 거대한 규모의 실제 세계 문제를 해결하는 핵심 도구로 자리매김할 것이다. 이론적 깊이를 더하고, 실용적 확장성을 확보하며, 사회적 책임을 고려하는 다각적인 노력이 계속되는 한, GNN은 과학적 발견과 산업 혁신을 가속화하는 데 중추적인 역할을 수행할 것으로 전망된다.


1. [딥러닝]Graph Neural Network 개요 - Meaningful AI, 8월 16, 2025에 액세스, https://meaningful96.github.io/deeplearning/GNN/
2. [Graph] Graph Neural Networks 개론(개념정리) - 곰곰의 일지 - 티스토리, 8월 16, 2025에 액세스, https://secundo.tistory.com/99
3. Graph Neural Networks 쉽게 이해하기 - velog, 8월 16, 2025에 액세스, [https://velog.io/@whattsup_kim/Graph-Neural-Networks-%EA%B8%B0%EB%B3%B8-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0](https://velog.io/@whattsup_kim/Graph-Neural-Networks-기본-쉽게-이해하기)
4. GNN 소개 - 기초부터 논문까지 - Medium, 8월 16, 2025에 액세스, [https://medium.com/watcha/gnn-%EC%86%8C%EA%B0%9C-%EA%B8%B0%EC%B4%88%EB%B6%80%ED%84%B0-%EB%85%BC%EB%AC%B8%EA%B9%8C%EC%A7%80-96567b783479](https://medium.com/watcha/gnn-소개-기초부터-논문까지-96567b783479)
5. GNN(Graph Neural Network)의 정의와 특징 그리고 추천시스템에서의 GNN 계열 모델 - 지그시, 8월 16, 2025에 액세스, [https://glanceyes.com/entry/%EC%B6%94%EC%B2%9C-%EC%8B%9C%EC%8A%A4%ED%85%9C-GNNGraph-Neural-Network%EC%99%80-%EC%9D%B4%EB%A5%BC-%EC%9D%91%EC%9A%A9%ED%95%9C-NGCFNeural-Graph-Collaborative-Filtering%EC%99%80-LightGCN](https://glanceyes.com/entry/추천-시스템-GNNGraph-Neural-Network와-이를-응용한-NGCFNeural-Graph-Collaborative-Filtering와-LightGCN)
6. Graph Neural Net (GNN) Basic - Hello, didi universe - 티스토리, 8월 16, 2025에 액세스, https://didi-universe.tistory.com/entry/GNN-1Graph-Neural-NetGNN-Basic
7. [개념정리] 그래프 신경망(Graph Neural Network / GNN) (3) - Innov_AI_te - 티스토리, 8월 16, 2025에 액세스, [https://jaylala.tistory.com/entry/%EB%94%A5%EB%9F%AC%EB%8B%9D-with-Python-%EA%B7%B8%EB%9E%98%ED%94%84-%EC%8B%A0%EA%B2%BD%EB%A7%9DGraph-Neural-Network-GNN-3](https://jaylala.tistory.com/entry/딥러닝-with-Python-그래프-신경망Graph-Neural-Network-GNN-3)
8. Graph Neural Networks in IoT: A Survey 논문 정리 - 주뇽's 저장소 - 티스토리, 8월 16, 2025에 액세스, https://jypark1111.tistory.com/154
9. [개념정리] 그래프 신경망(Graph Neural Network / GNN) (2) - Innov_AI_te - 티스토리, 8월 16, 2025에 액세스, [https://jaylala.tistory.com/entry/%EA%B0%9C%EB%85%90%EC%A0%95%EB%A6%AC-%EA%B7%B8%EB%9E%98%ED%94%84-%EC%8B%A0%EA%B2%BD%EB%A7%9DGraph-Neural-Network-GNN-2](https://jaylala.tistory.com/entry/개념정리-그래프-신경망Graph-Neural-Network-GNN-2)
10. Graph Neural Network: In a Nutshell | Karthick Panner Selvam, 8월 16, 2025에 액세스, https://karthick.ai/blog/2024/Graph-Neural-Network/
11. Chapter 4 - The Graph Neural Network Model, 8월 16, 2025에 액세스, https://cs.mcgill.ca/~wlh/comp766/files/chapter4_draft_mar29.pdf
12. Graph neural network - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Graph_neural_network
13. Graph Neural Networks Series | Part 4 |The GNNs, Message Passing & Over-smoothing | by Omar Hussein | The Modern Scientist | Medium, 8월 16, 2025에 액세스, https://medium.com/the-modern-scientist/graph-neural-networks-series-part-4-the-gnns-message-passing-over-smoothing-e77ffee523cc
14. Graph Neural Networks - Notes - Nihal Nayak, 8월 16, 2025에 액세스, https://nihalnayak.github.io/assets/pdf/notes/gnn_notes.pdf
15. Simplifying Graph Convolutional Networks - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/pdf/1902.07153
16. Semi-Supervised Classification with Graph Convolutional Networks, 8월 16, 2025에 액세스, https://arxiv.org/pdf/1609.02907
17. Semi-Supervised Classification with Graph Convolutional Networks - OpenReview, 8월 16, 2025에 액세스, https://openreview.net/forum?id=SJU4ayYgl
18. [1609.02907] Semi-Supervised Classification with Graph Convolutional Networks - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/1609.02907
19. Message passing neural network 설명 (MPNN 설명, 장점, 예시) - 유니의 공부 - 티스토리, 8월 16, 2025에 액세스, https://process-mining.tistory.com/164
20. Inductive Representation Learning on Large Graphs - NIPS, 8월 16, 2025에 액세스, http://papers.neurips.cc/paper/6703-inductive-representation-learning-on-large-graphs.pdf
21. [1706.02216] Inductive Representation Learning on Large Graphs - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/1706.02216
22. Graph SAGE(SAmple and aggreGatE) : Inductive Learning on Graphs - Data Science Group, 8월 16, 2025에 액세스, https://dsgiitr.in/blogs/graph_sage/
23. Inductive Representation Learning On Large Graphs - McGill School Of Computer Science, 8월 16, 2025에 액세스, https://cs.mcgill.ca/~wlh/comp766/files/deepak_sharma.pdf
24. GraphSAGE: Full Paper Walkthrough! | by Arjhun Sreedar - Medium, 8월 16, 2025에 액세스, https://medium.com/@arjuns0206/graphsage-full-paper-walkthrough-1b0ebd79afb4
25. Graph Attention Networks, 8월 16, 2025에 액세스, https://arxiv.org/pdf/1710.10903
26. Graph Attention Networks - Petar Veličković, 8월 16, 2025에 액세스, https://petar-v.com/GAT/
27. [1710.10903] Graph Attention Networks - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/1710.10903
28. Graph Attention Networks | OpenReview, 8월 16, 2025에 액세스, https://openreview.net/forum?id=rJXMpikCZ
29. Tackling Oversmoothing in GNN via Graph Sparsification: A Truss-based Approach - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2407.11928v1
30. Measuring and Relieving the Over-Smoothing Problem for Graph Neural Networks from the Topological View - AAAI, 8월 16, 2025에 액세스, https://cdn.aaai.org/ojs/5747/5747-13-8972-1-10-20200513.pdf
31. Exploring Over-Smoothing in Graph Neural Networks (GNNs) | by Rahul Kumar | Medium, 8월 16, 2025에 액세스, https://medium.com/@hellorahulk/exploring-over-smoothing-in-graph-neural-networks-gnns-e311b8eb8a85
32. Demystifying Oversmoothing in Attention-Based Graph Neural Networks - OpenReview, 8월 16, 2025에 액세스, [https://openreview.net/forum?id=Kg65qieiuB¬eId=sSqhiAOpsV](https://openreview.net/forum?id=Kg65qieiuB&noteId=sSqhiAOpsV)
33. Measuring and Relieving the Over-Smoothing Problem for Graph Neural Networks from the Topological View | Proceedings of the AAAI Conference on Artificial Intelligence, 8월 16, 2025에 액세스, https://aaai.org/ojs/index.php/AAAI/article/view/5747
34. Mitigating over-smoothing in Graph Neural Networks for node classification through Adaptive Early Embedding and Biased DropEdge procedures | Request PDF - ResearchGate, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/391504153_Mitigating_over-smoothing_in_Graph_Neural_Networks_for_node_classification_through_Adaptive_Early_Embedding_and_Biased_DropEdge_procedures
35. Towards Efficient and Expressive GNNs for Graph Classification via Subgraph-Aware Weisfeiler-Lehman - Proceedings of Machine Learning Research, 8월 16, 2025에 액세스, https://proceedings.mlr.press/v198/wang22b.html
36. The Expressive Power of Graph Neural Networks, 8월 16, 2025에 액세스, https://graph-neural-networks.github.io/static/file/chapter5.pdf
37. Expressive power of graph neural networks and the Weisfeiler-Lehman test, 8월 16, 2025에 액세스, https://towardsdatascience.com/expressive-power-of-graph-neural-networks-and-the-weisfeiler-lehman-test-666b9e17585b/
38. Beyond Weisfeiler-Lehman: A Quantitative Framework for GNN Expressiveness, 8월 16, 2025에 액세스, https://openreview.net/forum?id=HSKaGOi7Ar
39. Attending to Graph Transformers - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2302.04181v3
40. arxiv.org, 8월 16, 2025에 액세스, [https://arxiv.org/html/2504.15920v1#:~:text=However%2C%20GNNs%20still%20face%20two,model%20complexity%20and%20increased%20inference](https://arxiv.org/html/2504.15920v1#:~:text=However%2C GNNs still face two,model complexity and increased inference)
41. ScaleGNN: Towards Scalable Graph Neural Networks via Adaptive High-order Neighboring Feature Fusion - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2504.15920v3
42. alexfanjn/Graph-Neural-Networks-With-Heterophily - GitHub, 8월 16, 2025에 액세스, https://github.com/alexfanjn/Graph-Neural-Networks-With-Heterophily
43. Scaling Graph Neural Networks for Large Graphs and Analyzing Common Vulnerabilities via Large Language Models - Purdue University Graduate School, 8월 16, 2025에 액세스, https://hammer.purdue.edu/articles/thesis/Scaling_Graph_Neural_Networks_for_Large_Graphs_and_Analyzing_Common_Vulnerabilities_via_Large_Language_Models/28856864
44. Self-Supervised Learning of Graph Neural Networks: A Unified Review - ResearchGate, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/349520452_Self-Supervised_Learning_of_Graph_Neural_Networks_A_Unified_Review
45. Self-supervised Learning on Graphs: Deep Insights and New Directions - Tyler Derr, 8월 16, 2025에 액세스, https://tylersnetwork.github.io/papers/self_supervised_learning_on_graphs.pdf
46. Graph Neural Networks: Self-supervised Learning - Tyler Derr, 8월 16, 2025에 액세스, https://tylersnetwork.github.io/papers/ssl_for_gnns.pdf
47. Self-Supervised Graph Neural Networks via Diverse and Interactive Message Passing - AAAI, 8월 16, 2025에 액세스, https://cdn.aaai.org/ojs/20353/20353-13-24366-1-2-20220628.pdf
48. [PDF] Graph Self-Supervised Learning: A Survey - Semantic Scholar, 8월 16, 2025에 액세스, https://www.semanticscholar.org/paper/Graph-Self-Supervised-Learning%3A-A-Survey-Liu-Pan/e259ee075998eedc0b0c91c17769bf9dffeba46f
49. [2506.22084] Transformers are Graph Neural Networks - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/2506.22084
50. arXiv:2302.04181v3 [cs.LG] 28 Mar 2024, 8월 16, 2025에 액세스, https://arxiv.org/pdf/2302.04181
51. Graph Transformer Networks - NIPS, 8월 16, 2025에 액세스, https://proceedings.neurips.cc/paper/9367-graph-transformer-networks.pdf
52. NeurIPS Poster Enhancing Graph Transformers with Hierarchical Distance Structural Encoding, 8월 16, 2025에 액세스, https://nips.cc/virtual/2024/poster/94991
53. Recommender Systems: The Rise of Graph Neural Networks | Shaped Blog, 8월 16, 2025에 액세스, https://www.shaped.ai/blog/recommender-system-family-tree-gnn
54. Using Graph Neural Networks for Social Recommendations - MDPI, 8월 16, 2025에 액세스, https://www.mdpi.com/1999-4893/16/11/515
55. Graph Neural Networks for Social Recommendation - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/pdf/1902.07243
56. A Graph-Neural-Network-Based Social Network Recommendation Algorithm Using High-Order Neighbor Information - PMC, 8월 16, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC9571652/
57. Aman's AI Journal • Recommendation Systems • Graph Neural Networks, 8월 16, 2025에 액세스, https://aman.ai/recsys/gnn/
58. Graph Neural Networks for Drug Discovery: An Integrated Decision Support Pipeline, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/377901872_Graph_Neural_Networks_for_Drug_Discovery_An_Integrated_Decision_Support_Pipeline
59. Graph Neural Network for Molecular Structure: Application in HIV Inhibitor Molecule Prediction - OSF, 8월 16, 2025에 액세스, https://osf.io/c3mqy/download
60. Drug discovery and Graph Neural Networks (GNNs): a regression example - Medium, 8월 16, 2025에 액세스, https://medium.com/@mulugetas/drug-discovery-and-graph-neural-networks-gnns-a-regression-example-fc738e0f11f3
61. A dual graph neural network for drug–drug interactions prediction based on molecular structure and interactions, 8월 16, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC9879511/
62. Recent Developments in GNNs for Drug Discovery - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2506.01302v1
63. Knowledge mapping of graph neural networks for drug discovery: a bibliometric and visualized analysis - PubMed Central, 8월 16, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11116974/
64. Graph Neural Networks in Anomaly Detection, 8월 16, 2025에 액세스, https://graph-neural-networks.github.io/static/file/chapter26.pdf
65. detectGNN: Harnessing Graph Neural Networks for Enhanced Fraud Detection in Credit Card Transactions - arXiv, 8월 16, 2025에 액세스, http://arxiv.org/pdf/2503.22681
66. Supercharging Fraud Detection in Financial Services with Graph Neural Networks (Updated) | NVIDIA Technical Blog, 8월 16, 2025에 액세스, https://developer.nvidia.com/blog/supercharging-fraud-detection-in-financial-services-with-graph-neural-networks/
67. Fraud detection with GNN - Kaggle, 8월 16, 2025에 액세스, https://www.kaggle.com/code/jawherjabri/fraud-detection-with-gnn
68. Build a GNN-based real-time fraud detection solution using Amazon SageMaker, Amazon Neptune, and the Deep Graph Library | Artificial Intelligence, 8월 16, 2025에 액세스, https://aws.amazon.com/blogs/machine-learning/build-a-gnn-based-real-time-fraud-detection-solution-using-amazon-sagemaker-amazon-neptune-and-the-deep-graph-library/
69. Guidance for Near Real-Time Fraud Detection with Graph Neural Network on AWS, 8월 16, 2025에 액세스, https://aws.amazon.com/solutions/guidance/near-real-time-fraud-detection-with-graph-neural-network-on-aws/
70. A Survey on Self-Supervised Graph Foundation Models: Knowledge-Based Perspective, 8월 16, 2025에 액세스, https://www.computer.org/csdl/journal/tk/2025/08/10992675/26y4lGr8vBu


# Cross-Image Encoder
[딥러닝 모듈 (Modules)](./index.md)


Cross-Image Encoder는 다중 이미지 또는 이미지와 다른 모달리티(예: 텍스트) 입력을 단일 네트워크에 함께 공급하여, 입력 간의 세밀하고 심층적인 상호작용(deep cross-modal interaction)을 모델링하는 아키텍처 클래스로 정의된다.1 이러한 공동 인코딩(joint encoding) 방식은 각 입력을 독립적으로 처리한 후 그 특징을 나중에 결합하는 '후기 융합(late fusion)' 방식과 근본적으로 구별된다.4 입력 요소 간의 미세한 문맥적 관계를 포착하는 데 탁월한 성능을 보여주며, 이는 이미지-텍스트 검색 1, 중복 이미지 탐지 5, 변화 탐지 6 등 높은 정확도를 요구하는 과제에서 필수적인 요소로 자리 잡았다.

본 보고서의 핵심 주제는 Cross-Encoder가 제공하는 높은 정확도와 그에 수반되는 막대한 계산 비용 사이의 근본적인 상충 관계(trade-off)이다.3 이 상충 관계는 각 입력을 독립적으로 처리하는 Bi-Encoder (또는 Dual-Encoder) 아키텍처와의 비교를 통해 가장 명확하게 드러난다. 이러한 trade-off를 극복하기 위해 지식 증류(knowledge distillation), 희소 어텐션(sparse attention), 하이브리드 파이프라인(hybrid pipeline)과 같은 다양한 최적화 전략이 등장했으며, 이는 이 분야 연구의 핵심 동력이 되고 있다.

본 보고서는 먼저 Cross-Encoder의 구조적 원리를 Bi-Encoder와 비교하여 분석하고(1장), 핵심 작동 메커니즘인 Cross-Attention을 수학적으로 탐구한다(2장). 이후, 다양한 응용 분야의 구체적인 아키텍처 사례를 분석하고(3장), 성능 최적화 전략을 논하며(4장), Few-shot learning과 같은 고급 응용과 미래 연구 방향을 조망한다(5장).


Bi-Encoder와 Cross-Encoder의 구조적 이분법은 단순한 설계 선택을 넘어, 관계를 모델링하는 방식에 대한 근본적인 철학적 차이를 나타낸다. Bi-Encoder는 개체들을 공유된 의미 공간 내에서 독립적으로 정의 가능한 객체로 취급하며, 이는 대규모 '검색(retrieval)' 작업의 확장성에 초점을 맞춘다. 반면, Cross-Encoder는 두 입력 쌍의 의미가 그들의 직접적인 상호작용을 통해 발현되는 속성이라고 가정하며, 이는 '판단(judgment)' 또는 '순위 결정(ranking)'을 위한 문맥적 정확성을 우선시한다. 이러한 차이점은 왜 두 아키텍처를 결합한 하이브리드 시스템이 효과적인지를 설명해준다. 이는 먼저 광범위하게 탐색한 후 신중하게 비교하는 인간의 인지 과정을 모방하기 때문이다.


Bi-Encoder는 두 개의 독립적인 인코더를 사용하여 각 입력을 별도의 임베딩 벡터로 변환하는 구조이다.1 이미지-텍스트 검색 분야에서는 이미지 인코더 `F`와 텍스트 인코더 `G`가 각각 이미지 `x`와 텍스트 `y`를 독립적으로 처리한다.1 종종 이 두 인코더는 동일한 구조와 가중치를 공유하는 샴 네트워크(Siamese Network) 형태로 구현되기도 한다.10

수학적으로 유사도 점수 $s_{x,y}$는 두 임베딩 벡터의 내적(dot product) 또는 코사인 유사도로 계산된다.1
$$
s_{x,y} = \phi_F(F(x))^T \phi_G(G(y))
$$
여기서 $\phi$는 최종 벡터 표현을 얻기 위한 프로젝션 헤드(projection head)를 의미한다.1

이 구조의 가장 큰 장점은 효율성과 확장성이다. 각 아이템(이미지, 텍스트 등)의 임베딩을 사전에 계산하여 저장(pre-computation)할 수 있다.8 따라서 대규모 데이터베이스에서 실시간 검색 시, 새로운 쿼리 임베딩과 미리 저장된 수백만 개의 임베딩 간의 빠른 벡터 유사도 검색(예: Approximate Nearest Neighbor)만 수행하면 되므로 계산적으로 매우 효율적이다.8

그러나 명확한 단점도 존재한다. 입력 간의 상호작용이 최종 임베딩 벡터 간의 단일 내적 연산으로 극히 제한된다.1 이처럼 '얕은 상호작용'은 입력 요소들 간의 세밀하고 복잡한 관계, 예를 들어 특정 단어와 이미지의 특정 영역 간의 관계를 포착하는 데 한계가 있으며, 이는 필연적으로 정확도 저하로 이어진다.3


Cross-Encoder는 두 입력(예: 이미지와 텍스트)을 연결(concatenate)하거나 다른 방식으로 결합하여 단일 입력 시퀀스로 만든 뒤, 이를 하나의 대형 인코더(주로 Transformer)에 통과시키는 구조이다.1

이 구조의 핵심은 Transformer 내부의 어텐션 레이어들이 입력 시퀀스 내 모든 토큰 쌍 간의 상호작용을 계산한다는 점이다. 이를 통해 한 입력의 토큰이 다른 입력의 특정 토큰에 직접적으로 주의(attend)를 기울일 수 있으며, 이러한 깊고 풍부한 상호작용이 여러 계층에 걸쳐 반복적으로 발생한다.1

결과적으로 Cross-Encoder는 개별 임베딩을 생성하는 대신, 입력 쌍에 대한 단일 유사도 점수(예: 0과 1 사이의 값) 또는 분류 확률(예: '일치'/'불일치')을 직접 출력한다.1

이러한 심층적인 상호작용 덕분에 Cross-Encoder는 Bi-Encoder가 놓칠 수 있는 미묘한 문맥적 관계를 포착하여 월등히 높은 정확도를 달성한다.3 특히 소수의 후보군에 대한 재순위(re-ranking)와 같이 정밀도가 중요한 작업에서 매우 효과적이다.8

반면, 가장 큰 단점은 높은 계산 비용이다. 모든 가능한 입력 쌍에 대해 전체 모델을 순전파해야 하므로, $N$개의 쿼리와 $M$개의 문서가 있다면, $N \times M$번의 추론이 필요하다.11 이는 대규모 실시간 검색 시스템에 적용하기에는 비실용적이다.11


Bi-Encoder와 Cross-Encoder는 각각 검색과 판단이라는 다른 목적에 최적화된 아키텍처이다. Bi-Encoder가 확장성을 위해 정확도를 일부 희생한다면, Cross-Encoder는 정확도를 위해 확장성을 포기한다. 이 명확한 상충 관계는 두 아키텍처의 장점을 결합한 하이브리드 접근법의 필요성을 시사하며, 현대 정보 검색 시스템 설계의 근간을 이룬다.

**Table 1: Bi-Encoder vs. Cross-Encoder 아키텍처 비교**

| 특징 (Feature)                             | Bi-Encoder (Dual-Encoder / Siamese)                    | Cross-Encoder                                          |
| ------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------ |
| **입력 처리 (Input Processing)**           | 두 입력을 **독립적으로(independently)** 인코딩 1       | 두 입력을 연결하여 **동시에(jointly)** 인코딩 1        |
| **아키텍처 (Architecture)**                | 두 개의 분리된 인코더 (가중치 공유 가능) 1             | 하나의 공유된 인코더 15                                |
| **상호작용 깊이 (Interaction Depth)**      | **얕음 (Shallow)**: 최종 임베딩 간 내적 연산 1회 3     | **깊음 (Deep)**: 모든 레이어에서 토큰 간 상호 어텐션 1 |
| **출력 (Output)**                          | 각 입력에 대한 **개별 임베딩 벡터** 12                 | 입력 쌍에 대한 **단일 유사도 점수** 12                 |
| **계산 복잡도 (Computational Complexity)** | 낮음. 검색 시 $O(N+M)$ (임베딩 사전 계산) 11           | 높음. 모든 쌍에 대해 $O(N \times M)$ 11                |
| **확장성 (Scalability)**                   | **높음**. 대규모 데이터베이스에 적합 8                 | **낮음**. 소규모 후보군에만 적용 가능 12               |
| **정확도 (Accuracy)**                      | 상대적으로 낮음 3                                      | **상대적으로 높음** 7                                  |
| **주요 사용 사례 (Primary Use Case)**      | **검색(Retrieval)**, 대규모 후보군 생성, 클러스터링 12 | **재순위(Re-ranking)**, NLI, 의미 유사도 측정 8        |


Cross-Attention은 단순히 정보를 융합하는 메커니즘이 아니라, 동적이고 쿼리 주도적인 정보 필터링 및 합성 과정이다. 단순한 연결(concatenation)이나 풀링(pooling)과 달리, 한 모달리티(쿼리)가 다른 모달리티(키/값 소스)를 능동적으로 '질의(interrogate)'하고 자신의 문맥에 맞춰진 새로운 표현을 구성하게 한다. 이러한 특성 덕분에 Cross-Attention은 텍스트 프롬프트를 기반으로 이미지를 생성하거나(Stable Diffusion), 두 위성 이미지 간의 변화된 영역을 강조하는 등 조건부 이해가 필요한 모든 작업의 근본적인 구성 요소가 된다.


Self-Attention은 단일 시퀀스 내에서 토큰 간의 관계를 학습하는 메커니즘이다. 쿼리(Query), 키(Key), 값(Value) 벡터가 모두 동일한 입력 시퀀스에서 파생되어 시퀀스 내부의 관계를 모델링한다.17

Cross-Attention은 이 개념을 두 개의 다른 시퀀스로 확장한다. 쿼리 벡터는 한 시퀀스(예: 정보를 필요로 하는 텍스트)에서 생성되고, 키와 값 벡터는 다른 시퀀스(예: 정보의 원천인 이미지)에서 생성된다.17 이 비대칭적 구조는 한 시퀀스가 다른 시퀀스의 정보에 선택적으로 '주의를 기울여' 필요한 정보를 가져오는 것을 가능하게 한다. 즉, 쿼리 모달리티가 어떤 정보가 관련성이 높은지를 결정하고 소스 모달리티로부터 그 정보를 추출하는 능동적이고 방향성 있는 프로세스이다.


두 입력 시퀀스로부터 파생된 쿼리($Q$), 키($K$), 값($V$) 행렬이 주어졌을 때, Cross-Attention의 출력은 Scaled Dot-Product Attention의 형태를 따라 다음과 같이 계산된다.5
$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

- $Q$: 정보를 요청하는 시퀀스에서 파생된 쿼리 행렬.
- $K$, $V$: 정보의 원천이 되는 시퀀스에서 파생된 키 및 값 행렬.
- $d_k$: 키 벡터의 차원으로, 스케일링을 통해 안정적인 그래디언트 전파를 보장한다.

이 계산은 다음과 같은 단계로 이루어진다:

1. **유사도 계산:** $Q$의 각 쿼리 벡터와 $K$의 모든 키 벡터 간의 내적을 계산하여 유사도 점수 행렬을 생성한다. 이 단계는 쿼리가 소스 시퀀스의 어떤 부분에 집중해야 하는지를 결정하는 '질의' 과정에 해당한다.18
2. **가중치 정규화:** Softmax 함수를 적용하여 유사도 점수를 합이 1이 되는 확률 분포, 즉 어텐션 가중치로 변환한다.19
3. **가중 합산:** 계산된 어텐션 가중치를 $V$의 값 벡터에 곱하여 가중 합산을 수행한다. 이를 통해 쿼리와 가장 관련성이 높은 정보만 선택적으로 종합하여, 쿼리의 필요에 맞게 '합성'된 새로운 표현을 생성한다.17


Cross-Attention은 서로 다른 정보 소스를 연결하는 '다리' 역할을 수행하며, 현대 다중 모달리티 시스템의 핵심으로 자리 잡았다.17

- **이미지-텍스트 융합:** 텍스트 토큰(쿼리)이 이미지 패치(키/값)에 어텐션을 적용하여, 특정 단어와 가장 관련 있는 이미지 영역을 찾아낸다. 예를 들어, DeepMind의 Flamingo 모델에서는 언어 모델의 은닉 상태(쿼리)가 다음 단어를 생성하기 위해 어떤 시각적 특징(키/값)이 중요한지를 결정한다.17 Stable Diffusion에서는 텍스트 프롬프트(쿼리)가 잠재 공간에서 관련 시각적 개념을 선택함으로써 노이즈 제거 과정을 안내한다.17
- **이미지-이미지 융합:** 한 이미지의 특징(쿼리)이 다른 이미지의 특징(키/값)에 어텐션을 적용한다. 이를 통해 두 이미지 간의 대응 관계(correspondences), 차이점, 유사성을 픽셀 또는 패치 수준에서 정밀하게 모델링할 수 있다. 이는 중복된 이미지를 탐지하거나 5, 두 시점의 위성 이미지 간의 변화를 탐지하는 데 필수적이다.6


Cross-Encoder의 응용은 아키텍처의 진화 패턴을 보여준다. 초기의 '후기 상호작용' 모델에서 '공동 인코딩' 모델로, 그리고 이제는 '초기-지속적 상호작용' 모델로 발전하고 있다. CrossVLT 4나 RCDT 6와 같은 아키텍처는 단순히 입력단에서 정보를 결합하는 것을 넘어, 인코딩 과정의 여러 중간 단계에서 지속적으로 정보를 교환한다. 이는 융합이 단일 이벤트가 아니라, 입력 간의 계층적이고 연속적인 대화 과정임을 시사하며, 이는 더 견고하고 문맥을 잘 이해하는 표현을 생성하는 데 기여한다.


LOOPITR은 Bi-Encoder(Dual Encoder)와 Cross-Encoder를 단일 네트워크 내에서 결합하여 상호 보완적으로 학습시키는 통합 아키텍처이다.1 기존에 두 아키텍처를 별도로 취급하던 방식에서 벗어나, 통합된 프레임워크 내에서 시너지를 창출하는 새로운 패러다임을 제시했다.

이 모델의 핵심은 '루프 상호작용(Loop Interaction)' 메커니즘에 있다:

- **Bottom-up (Dual → Cross):** Bi-Encoder는 Cross-Encoder의 학습을 돕기 위해 두 가지 중요한 정보를 제공한다. 첫째, 독립적으로 인코딩된 정렬된 이미지/텍스트 임베딩 시퀀스를 Cross-Encoder의 입력으로 사용하여 그 자체의 성능을 향상시킨다. 둘째, 학습 배치 내에서 구별하기 어려운 부정적 예시(Hard Negatives)를 효율적으로 마이닝하여 Cross-Encoder의 이미지-텍스트 매칭(ITM) 학습을 돕는다.1
- **Up-down (Cross → Dual):** 더 정교한 판단 능력을 가진 Cross-Encoder가 Bi-Encoder의 '교사(teacher)' 역할을 하여 지식 증류(Knowledge Distillation)를 수행한다. Cross-Encoder가 계산한 정교한 유사도 점수를 Bi-Encoder가 모방하도록 학습시켜, Bi-Encoder의 표현 능력을 끌어올린다.1


CrossViT는 단일 이미지를 서로 다른 크기의 패치(작은 패치, 큰 패치)로 처리하는 두 개의 독립적인 Vision Transformer (ViT) 브랜치를 가진 혁신적인 아키텍처이다.19 이 모델은 Cross-Encoder의 핵심 원리인 Cross-Attention을 이종 모달리티가 아닌, 단일 이미지 내의 다중 스케일 표현 융합에 적용하여 ViT의 성능을 향상시켰다.

융합 과정은 한 브랜치의 CLS 토큰을 쿼리로 사용하고, 다른 브랜치의 모든 패치 토큰을 키와 값으로 사용하여 Cross-Attention을 수행하는 방식으로 이루어진다.19 CLS 토큰은 해당 브랜치의 전반적인 정보를 요약하고 있으므로, 이를 '에이전트'로 삼아 다른 스케일의 정보를 효율적으로 교환하고 융합할 수 있다. L-브랜치의 융합 과정은 수학적으로 다음과 같이 표현된다. 여기서 $x^l_{cls}$가 쿼리가 되어 $x^s_{patch}$와 상호작용한다.19
$$
\mathbf{y}^l_{cls} = f^l\left (\mathbf{x}^l_{cls}\right ) + \mathtt {MCA}(\mathtt {LN}(\left [f^l(\mathbf {x}^l_{cls}) \ \vert \vert \ \ \mathbf {x}^{s}_{patch} \right ]))
$$
이러한 융합은 네트워크를 통해 여러 번 반복되어, 두 브랜치가 점진적으로 서로의 정보를 통합하게 된다.


변화 탐지(Change Detection)는 서로 다른 두 시점에 촬영된 위성 또는 항공 이미지 쌍을 입력받아 변화가 발생한 영역을 탐지하는 과제이다.6 RCDT(Relational Change Detection Transformer)는 이 과제를 해결하기 위해 Cross-Encoder 구조를 효과적으로 활용한 대표적인 사례이다.

RCDT 아키텍처는 먼저 가중치를 공유하는 샴 네트워크(Siamese Backbone)를 사용하여 두 시점의 이미지로부터 특징을 추출한다.6 그 후, RCAM(Relational Cross Attention Module)이라는 핵심 모듈을 통해 두 특징 맵을 융합하고 관계를 모델링한다. RCAM은 Transformer 디코더 구조와 유사하게, 한 시점의 이미지 특징을 쿼리로, 다른 시점의 이미지 특징을 키/값으로 사용하여 Cross-Attention을 수행한다. 이를 통해 두 이미지 간의 픽셀 수준 대응 관계와 미세한 차이를 명시적으로 모델링하여 변화 영역을 정확하게 식별할 수 있다.6 RCDT는 기존의 단순 차분(difference) 기반 융합 방식의 한계를 극복하고, 두 이미지 간의 관계를 정밀하게 분석하는 데 Cross-Encoder 구조가 매우 효과적임을 입증했다.21


Cross-Encoder의 다양한 최적화 전략들은 단순히 개별적인 기법이 아니라, 핵심적인 $O(N \times M)$ 복잡도 문제를 완화하기 위한 상호 보완적인 생태계를 형성한다. 지식 증류는 값비싼 계산의 '결과'를 저렴한 모델로 이전하고, 희소 어텐션은 계산 자체의 '비용'을 줄이며, 하이브리드 파이프라인은 값비싼 계산의 '범위'를 가치 있는 소규모 부분집합으로 제한한다. 이 세 가지 접근법은 각각 이전(transfer), 감소(reduce), 제한(restrict)이라는 뚜렷하지만 상호 보완적인 전략을 대표한다.


지식 증류는 계산 비용이 높은 대형 모델(Teacher, 여기서는 Cross-Encoder)의 출력을 작고 효율적인 모델(Student, 여기서는 Bi-Encoder)이 모방하도록 학습시키는 기법이다.3 Cross-Encoder가 생성한 정교한 유사도 점수 분포(logit)나 순위 정보를 Bi-Encoder가 학습 목표로 삼아, 독립적인 인코딩 구조를 유지하면서도 Cross-Encoder의 깊은 상호작용 능력을 일부 흡수하게 된다.1

그러나 이 과정에는 한계점이 존재한다. 두 모델의 출력 분포가 본질적으로 다르기 때문이다. Cross-Encoder의 유사도 점수 분포는 정답과 오답 간의 구분이 명확하여 더 집중된 형태를 띠는 반면, Bi-Encoder의 점수 분포는 정규분포에 가까운 경향이 있다.3 이로 인해 단순한 logit 증류는 비효율적일 수 있으며, 대신 상대적인 순위 정보를 증류하는 방식(ranking distillation)이 더 효과적일 수 있다는 연구 결과가 있다.3


희소 어텐션은 모든 토큰 쌍 간의 상호작용을 계산하는 Full-Attention 대신, 미리 정의된 희소한 패턴(예: 지역 윈도우) 내에서만 어텐션을 계산하여 계산량을 획기적으로 줄이는 기법이다.23

이 기법은 어텐션 계산의 복잡도를 시퀀스 길이 $s$에 대해 $O(s^2)$에서 $O(s \cdot w)$ ($w$는 윈도우 크기)로 크게 감소시켜 메모리 사용량과 추론 시간을 단축한다.23 이를 통해 기존 Cross-Encoder가 처리하기 어려웠던 긴 문서나 고해상도 이미지의 처리가 가능해진다. 흥미롭게도, 연구에 따르면 매우 작은 윈도우 크기(예: 4개 토큰)를 사용하더라도 Full-Attention 기반 Cross-Encoder와 유사한 수준의 성능을 유지할 수 있음이 밝혀졌다.23 이는 대부분의 중요한 상호작용이 지역적으로 발생하며, 모든 토큰 간의 전역적인 상호작용이 항상 필수적인 것은 아님을 시사한다.


하이브리드 파이프라인은 Bi-Encoder의 속도와 Cross-Encoder의 정확도를 모두 활용하는 실용적인 2단계 접근법이다.12 이 방식은 대규모 검색 시스템에서 속도와 정확성 간의 균형을 맞추는 가장 널리 사용되는 해결책으로 자리 잡았다.8

작동 단계는 다음과 같다:

1. **1단계 (Retrieval):** 빠른 Bi-Encoder를 사용하여 전체 데이터베이스에서 쿼리와 관련된 상위 $k$개의 후보군(예: $k=100$)을 신속하게 검색한다.12 이 단계는 방대한 검색 공간을 효율적으로 좁히는 역할을 한다.
2. **2단계 (Re-ranking):** 계산 비용이 높은 Cross-Encoder를 오직 이 $k$개의 후보군에만 적용하여 정밀하게 순위를 재조정하고 최종 결과를 도출한다.12 이 단계는 소수의 유망한 후보들에 대해서만 깊이 있는 분석을 수행하여 최종 결과의 질을 높인다.


Cross-Encoder의 원리는 Few-Shot Learning과 같은 메타 학습 문제를 해결하기 위해 일반화되고 있다. 이 맥락에서 'Support Set'과 'Query Image'는 공동으로 처리되어야 할 두 개의 개별 입력으로 취급된다. 이는 few-shot 분류 문제를 단순한 임베딩 비교(메트릭 학습) 문제에서, 모델이 제공된 서포트 예제들의 '문맥 속에서' 쿼리를 '이해'하는 법을 배우는 동적이고 문맥 의존적인 추론 문제로 재구성한다. 이는 정적인 유사도 공간을 학습하는 것에서 적응적인 추론 과정을 학습하는 것으로의 중요한 개념적 도약을 의미한다.


Few-Shot Learning은 극소수의 레이블된 예제(Support Set)만을 사용하여 새로운 클래스를 학습하고 분류하는 과제이다.25 전통적인 방식은 쿼리 이미지와 서포트 클래스 프로토타입의 임베딩을 비교하는 메트릭 학습에 의존하는데, 이는 Bi-Encoder의 철학과 유사하다.

Cross-Encoder 구조를 이 문제에 적용함으로써, Support Set과 Query 이미지를 두 개의 입력 시퀀스로 간주하여 함께 처리할 수 있다. 이를 통해 모델은 Query 이미지를 분류하기 위해 Support Set의 어떤 정보에 동적으로 집중해야 하는지를 학습한다. 예를 들어, QSFormer 아키텍처는 Query-Support Transformer를 제안하여, 쿼리와 서포트 샘플들 간의 관계를 Cross-Attention으로 명시적으로 모델링함으로써 표현 학습과 메트릭 학습을 동시에 수행한다.28 이는 정적인 임베딩 공간에 의존하는 기존 방식보다 더 유연하고 강력한 관계 모델링을 가능하게 하여, "이 쿼리는 서포트 예제 3번과 날개 모양이 같지만 종은 다르다"와 같은 복잡한 추론을 가능하게 할 잠재력을 가진다. LiqFit과 같은 라이브러리는 이러한 Cross-Encoder 모델을 few-shot learning에 쉽게 적용하고 파인튜닝할 수 있도록 지원하며, 단 8개의 예제만으로도 높은 성능을 달성할 수 있음을 보여준다.25


최근 연구 동향은 분리된 인코더-융합-디코더 파이프라인에서 벗어나, 인코딩 초기 단계부터 모달리티 간 상호작용을 수행하는 단일 타워(One-Tower) 또는 Joint Fusion Encoder 모델로 이동하고 있다.31

CrossVLT 아키텍처가 대표적인 예이다. 이 모델은 비전 인코더와 언어 인코더의 각 단계(stage)에서 양방향으로 정보를 교환하는 'Cross-aware early fusion'을 제안한다.4 후반 레이어에서만 정보를 한 번 융합하는 Late Fusion 방식과 달리, 이러한 초기-지속적 융합(early-and-continuous fusion) 방식은 저수준 특징부터 고수준 개념에 이르기까지 계층적으로 풍부한 문맥 정보를 활용할 수 있게 한다.4 이는 모달리티 간의 관계를 더욱 근본적인 수준에서 학습하게 하여, 복잡한 다중 모달리티 추론 과제에서 더 높은 성능을 이끌어낼 잠재력을 가진다.31


Cross-Image Encoder 분야는 여전히 해결해야 할 과제와 탐구할 영역이 많다.

- **효율적인 Cross-Attention 변형:** 현재의 희소 어텐션을 넘어, 내용 기반(content-based)으로 동적으로 어텐션 범위를 결정하거나, 저순위 근사(low-rank approximation)와 같은 기법을 활용하여 더욱 효율적인 어텐션 메커니즘에 대한 연구가 필요하다.
- **경량화 및 실시간 Cross-Encoder:** 모델 압축, 양자화, 증류 기법을 더욱 발전시켜 모바일 기기나 실시간 서비스에서도 활용 가능한 경량 Cross-Encoder 모델 개발이 중요한 과제로 남아있다.
- **Zero-Shot 및 Unsupervised 환경으로의 확장:** 레이블된 데이터 쌍 없이도 Cross-Encoder를 효과적으로 학습시키는 방법에 대한 연구가 필요하다. 대조 학습(Contrastive Learning)과 같은 자기지도학습 패러다임을 Cross-Encoder 구조에 접목하려는 시도가 이루어질 수 있다.13
- **다중 입력 처리:** 두 개를 초과하는 이미지나 모달리티(예: 이미지 + 텍스트 + 오디오)를 동시에 처리하고 그들 간의 복잡한 상호작용을 모델링하는 Multi-input Cross-Encoder 아키텍처에 대한 연구가 활발해질 것이다.33


Cross-Image Encoder는 다중 입력을 공동으로 처리하여 깊은 상호작용을 모델링하는 강력한 아키텍처이다. 이는 Bi-Encoder가 제공하는 효율성을 희생하는 대신, 비교, 순위 결정, 융합과 같은 과제에서 월등한 정확도를 제공한다. 이러한 깊은 상호작용을 가능하게 하는 핵심 기술은 Cross-Attention 메커니즘으로, 이는 다양한 모달리티와 태스크에 걸쳐 정보 융합의 표준으로 자리 잡았다.

현재 연구는 Cross-Encoder의 높은 계산 비용을 극복하기 위해 지식 증류, 희소 어텐션, 하이브리드 파이프라인과 같은 실용적인 해결책에 집중하고 있다. 앞으로 이 분야는 Few-shot learning, 초기-지속적 융합, 그리고 더 많은 입력을 동시에 처리하는 방향으로 발전하며 인공지능의 추론 능력을 한 단계 더 끌어올릴 것으로 기대된다. 정확성과 효율성 사이의 본질적인 상충 관계를 해결하려는 지속적인 노력은 이 분야의 혁신을 계속해서 주도할 것이다.


1. LoopITR: Combining Dual and Cross Encoder Architectures for ..., 8월 23, 2025에 액세스, https://arxiv.org/pdf/2203.05465
2. LoopITR: Combining Dual and Cross Encoder Architectures for ..., 8월 23, 2025에 액세스, https://arxiv.org/abs/2203.05465
3. How to Make Cross Encoder a Good Teacher for Efficient Image-Text Retrieval? - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/html/2407.07479v1
4. Cross-aware Early Fusion with Stage-divided Vision and Language Transformer Encoders for Referring Image Segmentation - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/html/2408.07539v1
5. Cross-ViT: Cross-attention Vision Transformer for Image Duplicate Detection, 8월 23, 2025에 액세스, https://www.researchgate.net/publication/377319362_Cross-ViT_Cross-attention_Vision_Transformer_for_Image_Duplicate_Detection
6. Full article: Cross attention is all you need: relational remote sensing change detection with transformer - Taylor & Francis Online, 8월 23, 2025에 액세스, https://www.tandfonline.com/doi/full/10.1080/15481603.2024.2380126
7. How to Make Cross Encoder a Good Teacher for Efficient Image-Text Retrieval? - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/abs/2407.07479
8. What are bi-encoders and cross-encoders, and when should I use each? - Milvus, 8월 23, 2025에 액세스, https://milvus.io/ai-quick-reference/what-are-biencoders-and-crossencoders-and-when-should-i-use-each
9. How does a cross-encoder operate differently from a bi-encoder, and when might you use one over the other? - Milvus, 8월 23, 2025에 액세스, https://milvus.io/ai-quick-reference/how-does-a-crossencoder-operate-differently-from-a-biencoder-and-when-might-you-use-one-over-the-other
10. Siamese neural network - Wikipedia, 8월 23, 2025에 액세스, https://en.wikipedia.org/wiki/Siamese_neural_network
11. Semi-Siamese Bi-encoder Neural Ranking Model Using Lightweight Fine-Tuning - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/pdf/2110.14943
12. Cross-Encoders — Sentence Transformers documentation, 8월 23, 2025에 액세스, https://sbert.net/examples/cross_encoder/applications/README.html
13. Improving unsupervised sentence-pair comparison - Amazon Science, 8월 23, 2025에 액세스, https://www.amazon.science/blog/improving-unsupervised-sentence-pair-comparison
14. Bi-Encoder vs Cross-Encoder - Azad Djan, 8월 23, 2025에 액세스, https://azaddjan.com/bi-encoder-vs-cross-encoder-4a6440f05f52
15. What is cross encoder? - Medium, 8월 23, 2025에 액세스, https://medium.com/@sujathamudadla1213/what-is-cross-encoder-fec22b58f16c
16. Search reranking with cross-encoders | OpenAI Cookbook, 8월 23, 2025에 액세스, https://cookbook.openai.com/examples/search_reranking_with_cross-encoders
17. Why Cross-Attention is the Secret Sauce of Multimodal Models | by Jakub Strawa | Medium, 8월 23, 2025에 액세스, https://medium.com/@jakubstrawadev/why-cross-attention-is-the-secret-sauce-of-multimodal-models-f8ec77fc089b
18. Cross Attention in Transformer - Medium, 8월 23, 2025에 액세스, https://medium.com/@sachinsoni600517/cross-attention-in-transformer-f37ce7129d78
19. CrossViT: Cross-Attention Multi-Scale Vision ... - CVF Open Access, 8월 23, 2025에 액세스, https://openaccess.thecvf.com/content/ICCV2021/papers/Chen_CrossViT_Cross-Attention_Multi-Scale_Vision_Transformer_for_Image_Classification_ICCV_2021_paper.pdf
20. Robust Scene Change Detection Using Visual Foundation Models and Cross-Attention Mechanisms - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/html/2409.16850v2
21. arxiv.org, 8월 23, 2025에 액세스, https://arxiv.org/abs/2212.04869
22. rcdt:relational remote sensing change detection - SciSpace, 8월 23, 2025에 액세스, https://scispace.com/pdf/rcdt-relational-remote-sensing-change-detection-with-355kx8kq.pdf
23. Investigating the Effects of Sparse Attention on Cross-Encoders, 8월 23, 2025에 액세스, https://arxiv.org/pdf/2312.17649
24. Improving Address Matching using Siamese Transformer Networks - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/html/2307.02300
25. Introducing LiqFit: Flexible Few-Shot Learning Library for Cross-Encoder Models, 8월 23, 2025에 액세스, https://blog.knowledgator.com/introducing-liqfit-flexible-few-shot-learning-library-for-cross-encoder-models-804eac5aea92
26. Transformer-Based Architectures in Few-Shot Learning for Computer Vision - ResearchGate, 8월 23, 2025에 액세스, https://www.researchgate.net/publication/391274714_Transformer-Based_Architectures_in_Few-Shot_Learning_for_Computer_Vision
27. A Comparative Study of Vision Transformer Encoders and Few-Shot Learning for Medical Image Classification - CVF Open Access, 8월 23, 2025에 액세스, https://openaccess.thecvf.com/content/ICCV2023W/CVAMD/papers/Nurgazin_A_Comparative_Study_of_Vision_Transformer_Encoders_and_Few-Shot_Learning_ICCVW_2023_paper.pdf
28. Few-Shot Learning Meets Transformer: Unified Query ... - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/abs/2208.12398
29. Few-Shot Learning Meets Transformer: Unified Query-Support Transformers for Few-Shot Classification - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/pdf/2208.12398
30. Knowledgator/LiqFit: Efficient few-shot learning with cross-encoders. - GitHub, 8월 23, 2025에 액세스, https://github.com/Knowledgator/LiqFit
31. Joint Fusion and Encoding: Advancing Multimodal Retrieval from the Ground Up - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/html/2502.20008v1
32. All-in-One Image Coding for Joint Human-Machine Vision with Multi-Path Aggregation, 8월 23, 2025에 액세스, https://neurips.cc/virtual/2024/poster/96407
33. Injecting Multiple Modalities into a Transformer Decoder via Cross-Attention, 8월 23, 2025에 액세스, https://discuss.huggingface.co/t/injecting-multiple-modalities-into-a-transformer-decoder-via-cross-attention/144801


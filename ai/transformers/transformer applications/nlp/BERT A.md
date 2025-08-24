# Bidirectional Encoder Representations from Transformers (BERT)
[자연어 처리 (NLP) 트랜스포머](./index.md)


2018년 구글이 BERT(Bidirectional Encoder Representations from Transformers)를 발표하기 전까지, 자연어 처리(NLP) 분야는 문맥을 이해하는 언어 표현(language representation) 모델을 개발하는 데 있어 중요한 기로에 서 있었다. 단어 임베딩 기술의 발전은 단어를 벡터 공간에 표현함으로써 의미적 관계를 포착하는 데 성공했지만, 이러한 초기 모델들은 '문맥'이라는 언어의 본질적인 속성을 깊이 있게 다루는 데 한계를 보였다. 단어의 의미는 고정된 것이 아니라 문장 내 다른 단어들과의 관계 속에서 결정되지만, 기존의 방법론들은 이러한 동적인 의미 변화를 온전히 담아내지 못했다.


BERT 이전의 언어 표현 모델들은 주로 두 가지 접근 방식으로 나눌 수 있으며, 각각은 문맥 이해에 있어 뚜렷한 한계를 가지고 있었다.


가장 지배적인 패러다임은 표준 언어 모델(Language Model, LM)을 활용한 단방향(unidirectional) 접근법이었다. OpenAI의 GPT(Generative Pre-trained Transformer)가 이 계열의 대표적인 모델이다.1 이러한 모델들은 텍스트를 한쪽 방향(일반적으로 왼쪽에서 오른쪽)으로만 순차적으로 처리하여 다음 단어를 예측하는 방식으로 훈련된다. 수학적으로 이는 특정 단어 w_i$의 확률을 이전 단어 시퀀스 $w_1,..., w_{i-1}$에만 의존하여 모델링하는 것을 의미한다.

이러한 단방향 구조는 문맥 정보의 절반을 의도적으로 무시하게 만든다. 예를 들어, "그 선수는 부상으로 인해 경기 출전을 포기했다"라는 문장에서 '포기했다'라는 단어의 의미를 이해하기 위해서는 앞선 '부상으로 인해'라는 정보뿐만 아니라, 뒤따르는 문맥(만약 있다면)도 중요할 수 있다. 그러나 단방향 모델은 $i$번째 단어의 표현을 계산할 때 $i+1$번째 이후의 단어들을 참조할 수 없으므로, 문장의 전체적인 의미를 종합적으로 파악하는 데 본질적인 제약이 있었다.1

이러한 제약은 아키텍처의 설계적 결함이라기보다는, 전통적인 언어 모델링이라는 사전 훈련(pre-training) 목표 자체에서 비롯된 필연적인 결과였다. 만약 다층(multi-layer) 구조의 모델이 진정으로 양방향 문맥을 모두 조건화하여 다음 단어를 예측하려 한다면, 예측 대상 단어의 정보가 후속 단어들의 표현에 간접적으로 '누수'되어 자기 자신을 참조하는 순환적 문제가 발생한다. 이는 예측을 무의미하게 만드는 자명한(trivial) 결과를 초래하므로, 전통적인 LM 프레임워크 내에서는 깊은 양방향성(deep bidirectionality) 구현이 불가능했다.4


단방향성의 한계를 극복하기 위한 시도로 ELMo(Embeddings from Language Models)와 같은 모델이 등장했다.6 ELMo는 독립적으로 훈련된 두 개의 순방향(left-to-right) 및 역방향(right-to-left) LSTM(Long Short-Term Memory) 네트워크의 내부 상태를 결합(concatenate)하는 방식을 채택했다. 이 접근법은 양쪽 방향의 문맥을 모두 활용한다는 점에서 진일보했지만, '얕은 양방향성(shallow bidirectionality)'이라는 평가를 받았다.6

그 이유는 왼쪽 문맥과 오른쪽 문맥이 모델의 깊은 층에서 동시에 상호작용하며 통합되는 것이 아니라, 각각 독립적으로 처리된 후 최상위 층에서 단순히 이어 붙여지기 때문이다. 즉, 각 단어의 표현이 생성될 때, 모델의 모든 층에서 왼쪽과 오른쪽 문맥이 함께 고려되어 깊은 상호작용을 통해 정제되는 과정이 부재했다. 이는 진정한 의미의 통합된 양방향 문맥 표현을 학습하는 데 한계로 작용했다.


이러한 배경 속에서 등장한 BERT는 NLP 패러다임의 전환을 가져왔다.9 BERT의 핵심 기여는 기존의 언어 모델링 목표를 과감히 버리고, '마스크 언어 모델(Masked Language Model, MLM)'이라는 새로운 사전 훈련 목표를 도입함으로써 모델의 모든 층에서 왼쪽과 오른쪽 문맥을 동시에 조건화하여 *깊은 양방향* 표현을 사전 훈련하는 것을 가능하게 한 점이다.1

BERT는 Transformer 아키텍처의 구조를 변경하는 대신, 사전 훈련 방법을 근본적으로 재고함으로써 이전 모델들의 한계를 극복했다. MLM은 문장의 일부 단어를  토큰으로 무작위로 가린 뒤, 주변의 양방향 문맥을 모두 활용하여 원래 단어를 예측하도록 모델을 훈련시킨다. 이 방식은 전통적인 언어 모델이 아니기 때문에, 단어가 자기 자신을 예측하는 순환 문제를 원천적으로 차단한다. 사전 훈련 목표를 순차적 생성에서 분리한 이 개념적 전환이야말로 BERT 성공의 핵심 열쇠였다. 결과적으로, 이렇게 사전 훈련된 BERT 모델은 단 하나의 추가적인 출력층(output layer)만으로 광범위한 다운스트림(downstream) NLP 과제에 미세 조정(fine-tuning)되어, 최소한의 과제별 아키텍처 수정만으로 당시 최고 수준(State-Of-The-Art, SOTA)의 성능을 달성하는 놀라운 유연성과 강력함을 보여주었다.9


BERT의 아키텍처는 2017년 Vaswani 등이 발표한 기념비적인 논문 "Attention Is All You Need"에서 제안된 트랜스포머(Transformer) 모델에 깊이 뿌리내리고 있다.13 이 논문은 순환 신경망(RNN)이나 컨볼루션 신경망(CNN)에 의존하던 기존의 시퀀스 변환 모델들의 패러다임을 깨고, 오직 어텐션(attention) 메커니즘에만 기반한 아키텍처를 제시했다. 순환 구조를 제거함으로써 모델 훈련의 병렬화를 극대화하고, 장거리 의존성(long-range dependency) 문제를 효과적으로 해결하여 시퀀스 모델링 분야에 혁명을 일으켰다.15 BERT는 이 트랜스포머 아키텍처의 인코더(encoder) 부분만을 활용하여 구성된다.17


트랜스포머의 핵심이자 BERT의 심장부는 셀프 어텐션(self-attention) 메커니즘이다. 셀프 어텐션은 문장 내의 한 단어를 처리할 때, 문장 내 다른 모든 단어들과의 관계를 직접적으로 계산하여 해당 단어의 문맥적 의미를 풍부하게 만드는 역할을 한다. 이 과정은 쿼리(Query), 키(Key), 밸류(Value)라는 세 가지 벡터를 통해 수학적으로 정교하게 구현된다.

입력으로 들어온 단어 임베딩 시퀀스를 행렬 $X$로 표현할 때, 셀프 어텐션은 학습 가능한 가중치 행렬 $W^Q, W^K, W^V$를 사용하여 각각 쿼리 행렬 $Q$, 키 행렬 $K$, 밸류 행렬 $V$를 생성한다.13
$$
Q = XW^Q \\
K = XW^K \\
V = XW^V \\
$$
여기서 각 행렬의 행은 시퀀스 내 각 토큰에 해당하는 쿼리, 키, 밸류 벡터를 나타낸다. 직관적으로 쿼리는 현재 단어가 다른 단어들에게 던지는 '질문'이고, 키는 각 단어가 자신의 정보를 나타내는 '꼬리표'이며, 밸류는 해당 단어가 가진 실제 '내용'이라고 비유할 수 있다.

이 세 행렬을 바탕으로 어텐션 출력은 '스케일드 닷-프로덕트 어텐션(Scaled Dot-Product Attention)'이라는 단일 공식으로 계산된다 13:

$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$
이 공식은 다음과 같은 단계로 나눌 수 있다.

1. **유사도 점수 계산 ($QK^T$):** 쿼리 행렬 $Q$와 키 행렬의 전치 $K^T$를 곱한다. 이 연산은 시퀀스 내 모든 단어 쌍 간의 유사도(dot-product similarity)를 계산하는 과정이다. 결과 행렬의 $ij$ 원소는 $i$번째 단어의 쿼리와 $j$번째 단어의 키 사이의 유사도 점수를 의미한다.
2. **스케일링 ($/\sqrt{d_k}$):** 계산된 유사도 점수를 키 벡터의 차원 $d_k$의 제곱근으로 나누어준다. 이 스케일링 과정은 $d_k$ 값이 클 경우 내적 값이 지나치게 커져 소프트맥스(softmax) 함수의 그래디언트가 매우 작아지는 문제(vanishing gradient)를 방지하고, 훈련을 안정화시키는 중요한 역할을 한다.13
3. **정규화 (softmax):** 스케일링된 점수 행렬에 소프트맥스 함수를 적용하여 모든 값이 0과 1 사이이고 각 행의 합이 1이 되도록 정규화한다. 이는 각 단어에 대한 어텐션 가중치(attention weights) 분포를 생성하는 과정으로, 현재 단어를 표현하는 데 다른 단어들이 얼마나 중요한지를 나타내는 확률값으로 해석할 수 있다.
4. **가중합 (...V):** 마지막으로, 계산된 어텐션 가중치 행렬을 밸류 행렬 $V$와 곱한다. 이는 각 단어의 밸류 벡터를 해당 단어의 어텐션 가중치만큼 가중하여 합산하는 과정이다. 결과적으로 어텐션 가중치가 높은 단어의 정보는 더 많이 반영되고, 낮은 단어의 정보는 적게 반영되어 문맥적으로 풍부해진 새로운 표현 벡터가 생성된다.


트랜스포머는 단일 어텐션 메커니즘을 사용하는 대신, '멀티-헤드 어텐션(Multi-Head Attention)'을 통해 모델의 표현력을 더욱 확장한다.13 이는 어텐션 과정을 여러 번 병렬적으로 수행하는 방식이다. 각 '헤드(head)'는 서로 다른, 독립적으로 학습된 가중치 행렬($W_i^Q, W_i^K, W_i^V$)을 사용하여 입력 임베딩을 각기 다른 표현 부분 공간(representation subspace)으로 투영한 후, 해당 공간에서 셀프 어텐션을 계산한다.22

$$
\text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)
$$
이러한 방식은 모델이 한 번에 여러 측면에서 단어 간의 관계를 포착할 수 있게 해준다. 예를 들어, 한 헤드는 구문적 관계에 집중하고, 다른 헤드는 의미적 관계에 집중하는 식으로 다양한 종류의 정보를 동시에 학습할 수 있다. 각 헤드에서 계산된 어텐션 출력들은 모두 연결(concatenate)된 후, 또 다른 학습 가능한 가중치 행렬 $W^O$를 통해 최종 출력 벡터로 투영된다.
$$
\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1,..., \text{head}_h)W^O
$$


BERT의 기본 구성 단위인 트랜스포머 인코더 블록은 멀티-헤드 어텐션 층 외에 두 가지 중요한 요소를 더 포함한다.13

1. **위치별 피드-포워드 신경망(Position-wise Feed-Forward Network, FFN):** 멀티-헤드 어텐션 층의 출력은 각 위치(토큰)마다 독립적으로 동일한 FFN을 통과한다. 이 FFN은 일반적으로 두 개의 선형 변환(linear transformation)과 그 사이에 ReLU 활성화 함수로 구성된다. 이는 어텐션을 통해 수집된 문맥 정보를 비선형적으로 변환하고, 모델의 표현력을 더욱 높이는 역할을 한다.
2. **잔차 연결(Residual Connection) 및 층 정규화(Layer Normalization):** 각 하위 층(멀티-헤드 어텐션, FFN)의 입력은 해당 하위 층의 출력을 통과한 후 그 결과에 더해지는 잔차 연결 구조를 가진다. 즉, $\text{LayerNorm}(x + \text{Sublayer}(x))$ 형태를 띤다. 잔차 연결은 깊은 네트워크에서 그래디언트가 소실되거나 폭발하는 문제를 완화하여 안정적인 학습을 가능하게 하며, 층 정규화는 각 층의 입력 분포를 안정시켜 훈련 속도를 높인다.

이러한 셀프 어텐션, FFN, 잔차 연결, 층 정규화가 결합된 인코더 블록을 여러 층으로 깊게 쌓아 BERT의 전체 아키텍처가 완성된다. 이 구조는 셀프 어텐션이 순서 정보를 내재적으로 처리하지 못한다는 중요한 특징을 가진다. 어텐션 계산은 본질적으로 집합(set) 연산이므로, 입력 순서를 뒤섞어도 단어 쌍 간의 내적 값은 변하지 않는다. 따라서 "개가 사람을 물었다"와 "사람이 개를 물었다"는 문장은 어텐션 메커니즘만으로는 구별할 수 없다. 이 문제를 해결하기 위해 트랜스포머는 입력 단계에서 각 토큰의 위치 정보를 명시적으로 알려주는 '위치 임베딩(Positional Embedding)'을 추가하며, 이는 언어의 순차적 특성을 모델이 학습하는 데 필수적인 요소이다.


BERT는 트랜스포머 인코더를 기본 빌딩 블록으로 사용하며, 구글은 연구 커뮤니티가 다양한 계산 환경과 요구사항에 맞춰 사용할 수 있도록 두 가지 주요 규모의 모델을 공개했다.17 이 두 모델은 아키텍처의 깊이와 너비, 즉 파라미터 수에서 차이를 보이며, 이는 모델의 표현력과 계산 비용 간의 트레이드오프를 나타낸다.4


BERT-BASE와 BERT-LARGE 모델의 핵심 하이퍼파라미터는 다음 표와 같다. 이 표는 두 모델의 규모 차이를 명확하게 보여주며, BERT-LARGE가 단순히 조금 더 큰 모델이 아니라, 은닉층 크기와 트랜스포머 레이어 수를 대폭 늘려 파라미터 수를 약 3배로 확장한 훨씬 더 강력한 모델임을 시사한다.

| 하이퍼파라미터        | BERT-BASE | BERT-LARGE |
| --------------------- | ----------- | ------------ |
| 트랜스포머 레이어 (L) | 12          | 24           |
| 은닉층 크기 (H)       | 768         | 1024         |
| 어텐션 헤드 수 (A)    | 12          | 16           |
| 총 파라미터 수        | 110M        | 340M         |

BERT-BASE 모델은 당시의 다른 대규모 모델인 OpenAI GPT와 비슷한 규모로 설계되어 성능을 직접적으로 비교할 수 있도록 했다.17 반면, BERT-LARGE는 당시로서는 매우 거대한 모델로, 논문에서 보고된 대부분의 SOTA 성능을 달성한 주역이다.4


BERT가 텍스트의 미묘한 의미를 이해하기 위해서는 원시 텍스트(raw text)를 모델이 처리할 수 있는 풍부한 정보가 담긴 수치적 표현으로 변환하는 과정이 필수적이다. BERT의 입력 표현은 세 가지 종류의 임베딩을 합산하여 구성되며, 이는 모델의 사전 훈련 과제와 긴밀하게 연관되어 설계되었다.18


먼저, 입력 텍스트는 'WordPiece' 토큰화 기법을 통해 더 작은 단위의 하위 단어(subword)들로 분리된다.2 이 방식은 어휘 사전에 없는 단어(Out-of-Vocabulary, OOV) 문제를 효과적으로 처리할 수 있게 해주며, 단어의 형태론적 정보를 보존하는 데 유리하다.

토큰화된 시퀀스의 시작에는 항상 (Classification)라는 특수 토큰이 추가된다. 이 토큰의 최종 은닉 상태(final hidden state)는 문장 전체 또는 문장 쌍의 의미를 종합적으로 요약하는 벡터로 사용되며, 특히 분류 과제에서 중요한 역할을 한다.17 두 개 이상의 텍스트 시퀀스(예: 문장 A와 문장 B)를 입력으로 받을 경우, 각 시퀀스는 (Separator) 토큰으로 구분된다. 이는 모델이 각 시퀀스의 경계를 명확히 인식하도록 돕는다.27


각 토큰에 대한 최종 입력 벡터는 다음 세 가지 임베딩의 요소별 합(element-wise sum)으로 계산된다.

1. **토큰 임베딩 (Token Embeddings):** 각 토큰이 WordPiece 어휘 사전에서 가지는 고유 ID에 해당하는 벡터이다. 이는 단어 자체의 의미를 나타낸다.
2. **세그먼트 임베딩 (Segment Embeddings):** 입력이 두 개의 문장으로 구성될 때, 각 토큰이 첫 번째 문장(Sentence A)에 속하는지, 두 번째 문장(Sentence B)에 속하는지를 구분하는 학습된 임베딩이다. 이 임베딩은 두 문장 간의 관계를 학습하는 '다음 문장 예측(NSP)' 과제를 수행하는 데 결정적인 정보를 제공한다.
3. **위치 임베딩 (Positional Embeddings):** 트랜스포머 인코더는 순환 구조가 없기 때문에 단어의 순서 정보를 자체적으로 인식하지 못한다. 위치 임베딩은 시퀀스 내 각 토큰의 절대적인 위치 정보를 모델에 제공하는 학습된 벡터이다. 원본 트랜스포머 논문에서는 사인/코사인 함수를 사용했지만, BERT에서는 이 임베딩을 데이터로부터 직접 학습한다.

이 세 가지 임베딩을 합산하는 방식은 모델이 단어의 의미, 소속 문장, 그리고 시퀀스 내 위치라는 세 가지 정보를 동시에 고려하여 풍부한 초기 표현을 구성할 수 있게 한다. 이처럼 BERT의 입력 표현 방식은 단순히 텍스트를 숫자로 변환하는 것을 넘어, 모델이 수행해야 할 사전 훈련 과제에 최적화된 정보를 주입하도록 정교하게 설계되었다. 예를 들어, 세그먼트 임베딩은 NSP 과제가 없다면 불필요한 요소가 되며,  토큰의 특별한 역할 역시 NSP 과제를 통해 문장 수준의 분류기로서의 정체성을 학습하게 된다. 즉, 입력 형식의 각 요소는 사전 훈련 목표를 직접적으로 지원하는 명확한 기능적 목적을 가진다.


BERT의 성공을 이끈 가장 핵심적인 요소는 '사전 훈련(pre-training)'과 '미세 조정(fine-tuning)'이라는 두 단계로 구성된 프레임워크이다.1 사전 훈련 단계에서 BERT는 레이블이 없는(unlabeled) 방대한 양의 텍스트 코퍼스를 사용하여 언어 자체에 대한 깊고 일반적인 이해를 학습한다. 이 과정은 일종의 비지도 학습(unsupervised learning)으로, 모델이 문법, 어휘, 상식 등 언어의 보편적인 패턴을 내재화하도록 만든다.29 BERT의 사전 훈련에는 영문 위키피디아(약 25억 단어)와 BookCorpus(약 8억 단어)라는 대규모 데이터셋이 사용되었다.9

이 단계에서 BERT는 두 가지 독창적인 과제를 동시에 수행하며, 이 과제들은 BERT가 깊은 양방향 문맥을 학습할 수 있도록 하는 핵심적인 장치이다.


MLM은 BERT가 진정한 양방향성을 획득할 수 있게 한 혁신적인 사전 훈련 목표이다.1 이는 전통적인 언어 모델이 가진 단방향성의 제약을 극복하기 위해 고안된 방식으로, 빈칸 채우기 문제와 유사한 '클로즈 테스트(Cloze test)'에서 영감을 받았다.1


MLM의 목표는 입력 문장에서 일부 단어를 무작위로 가린 뒤, 주변의 양방향 문맥을 모두 활용하여 가려진 단어(토큰)가 무엇이었는지를 예측하는 것이다.32 예를 들어, "나는에 가서 책을 샀다"라는 문장이 주어지면, 모델은  위치에 '서점'이라는 단어가 올 확률이 가장 높다는 것을 예측해야 한다. 이 과정에서 모델은 '가서', '책을 샀다'와 같은 양쪽의 문맥 정보를 모두 고려해야 하므로, 자연스럽게 깊은 양방향 표현을 학습하게 된다.


단순히 일부 단어를 토큰으로 바꾸는 것만으로는 문제가 발생할 수 있다. 사전 훈련 단계에서는 토큰이 존재하지만, 실제 다운스트림 과제에 대한 미세 조정 단계에서는 이 토큰이 나타나지 않기 때문에, 사전 훈련과 미세 조정 간의 불일치(mismatch)가 발생한다. 이 문제를 완화하기 위해 BERT는 다음과 같은 정교한 15% 마스킹 전략을 사용한다 1:

1. 입력 시퀀스에서 예측 대상이 될 토큰을 15% 무작위로 선택한다.
2. 선택된 15%의 토큰에 대해 다음과 같은 규칙을 적용한다:
   - **80%의 경우:** 해당 토큰을 특수 토큰인 로 교체한다. (예: my dog is hairy -->> my dog is)
   - **10%의 경우:** 해당 토큰을 어휘 사전에 있는 다른 무작위 토큰으로 교체한다. (예: my dog is hairy -->> my dog is apple)
   - **10%의 경우:** 해당 토큰을 원래 그대로 유지한다. (예: my dog is hairy -->> my dog is hairy)

이 80-10-10 규칙은 매우 중요한 설계적 선택이다. 만약 100% 토큰으로만 교체한다면, 모델은 토큰이 아닌 실제 단어에 대한 풍부한 문맥적 표현을 학습하는 데 어려움을 겪을 수 있다. 10%를 무작위 단어로 교체함으로써, 모델은 단순히 를 채우는 것을 넘어, 문맥상 어색한 단어를 감지하고 수정하는 능력까지 학습하게 된다. 이는 모델이 모든 토큰에 대해 강력한 문맥적 인코더가 되도록 강제하는 효과를 낳는다. 또한, 10%를 원래 단어 그대로 둠으로써, 모델은 관찰된 단어가 항상 틀린 것이라는 편향을 갖지 않고, 실제 단어 자체에 대한 표현도 학습하게 된다. 이 복합적인 전략 덕분에 BERT는 미세 조정 단계에서 마주할 다양한 실제 텍스트에 대해 훨씬 더 강력하고 일반화된 표현 능력을 갖추게 된다.


MLM이 단어와 문장 내 관계를 학습하는 데 중점을 둔다면, NSP는 문장과 문장 사이의 관계를 이해하도록 모델을 훈련시키는 과제이다.1 이는 질의응답(Question Answering), 자연어 추론(Natural Language Inference)과 같이 두 텍스트 간의 논리적 관계 파악이 중요한 여러 다운스트림 과제에서 필수적인 능력이다.35


NSP는 주어진 두 문장 A와 B에 대해, 문장 B가 실제 텍스트에서 문장 A 바로 다음에 오는 문장인지를 예측하는 이진 분류(binary classification) 과제이다.37 훈련 데이터를 구성할 때, 50%는 실제 연속된 문장 쌍으로 구성하고 'IsNext' 레이블을 부여하며, 나머지 50%는 코퍼스에서 무작위로 추출한 문장을 B로 사용하여 'NotNext' 레이블을 부여한다.1

예를 들어:

- **긍정 예시 (IsNext):**
  - 문장 A: 해가 산 너머로 지고 있었다.
  - 문장 B: 하늘은 찬란한 주황색으로 물들었다.
- **부정 예시 (NotNext):**
  - 문장 A: 해가 산 너머로 지고 있었다.
  - 문장 B: 그녀는 책을 집어 들고 읽기 시작했다.

모델은 입력 시퀀스의 맨 앞에 있는 토큰의 최종 은닉 상태 벡터를 사용하여 이 이진 분류를 수행한다. 이 과정을 통해 토큰은 두 문장의 관계를 종합적으로 요약하는 능력을 학습하게 되며, 모델은 담화 수준의 일관성(coherence)과 논리적 흐름을 파악하는 능력을 기르게 된다.36


BERT의 진정한 힘은 사전 훈련을 통해 학습된 풍부하고 일반적인 언어 표현을 특정 다운스트림 과제에 효과적으로 전이(transfer)하는 능력에 있다. 이 과정을 미세 조정(fine-tuning)이라고 하며, BERT 프레임워크의 두 번째 핵심 단계를 구성한다. 미세 조정의 가장 큰 장점은 개념적 단순함과 강력한 실증적 성능에 있다. 사전 훈련된 BERT 모델의 모든 파라미터를 그대로 가져온 뒤, 과제의 특성에 맞는 아주 작은 출력층 하나만 추가하여 전체 모델을 종단간(end-to-end)으로 다시 훈련시킨다. 이 간단한 절차만으로도 복잡하게 설계된 과제별 특화 아키텍처들을 능가하는 SOTA 성능을 달성할 수 있었다.1

BERT 아키텍처는 두 종류의 '보편적인' 출력 형식을 제공하기 때문에 이러한 유연성이 가능하다. 하나는 시퀀스 전체를 요약하는  토큰의 표현 벡터이고, 다른 하나는 시퀀스 내 각 토큰에 대한 개별적인 표현 벡터 시퀀스이다. NLP의 다양한 과제들은 출력 형식 요구사항이 제각기 다르지만(예: 문장 전체에 대한 단일 레이블, 각 토큰에 대한 레이블, 텍스트 일부 구간을 나타내는 시작/끝 인덱스), BERT의 이 두 가지 출력 형식은 거의 모든 주요 NLP 과제의 출력 구조에 효과적으로 매핑될 수 있다. 이로 인해 핵심 사전 훈련 모델을 전혀 수정할 필요 없이, 과제별 특수성은 이 보편적 출력을 입력으로 받는 최소한의 '헤드(head)' 부분에만 국한된다. 이러한 아키텍처적 결정이 BERT의 놀라운 다재다능함의 비결이다.


감성 분석, 문장 유사도 측정, 자연어 추론 등 문장 전체 또는 문장 쌍에 대해 단일 예측을 수행하는 과제들이 이 범주에 속한다. 대표적인 벤치마크로는 GLUE(General Language Understanding Evaluation)가 있다.

- **메커니즘:** 이러한 과제를 위해, 입력 시퀀스의 가장 앞에 위치한  토큰의 최종 은닉 상태 벡터 $C \in \mathbb{R}^H$를 사용한다. 이 벡터는 사전 훈련의 NSP 과제를 통해 문장 전체의 의미를 응축하도록 학습되었기 때문에, 문장 수준의 특징 벡터로 사용하기에 이상적이다. 이 벡터 $C$를 간단한 분류층(예: 단일 레이어 피드-포워드 네트워크)에 입력하고, 과제의 클래스 수에 맞는 소프트맥스 함수를 통해 최종 확률 분포를 출력한다.1 미세 조정 과정에서는 이 분류층의 가중치뿐만 아니라 BERT 모델 내부의 모든 파라미터가 함께 업데이트된다.40


개체명 인식(Named Entity Recognition, NER)이나 품사 태깅(Part-of-Speech tagging)과 같이 시퀀스 내의 모든 토큰에 대해 레이블을 예측해야 하는 과제들이 여기에 해당한다.

- **메커니즘:** 이 경우,  토큰 대신 시퀀스 내의 모든 토큰 각각의 최종 은닉 상태 벡터 $T_i \in \mathbb{R}^H$를 활용한다. 각 토큰의 벡터 $T_i$는 해당 토큰의 문맥적 의미를 담고 있으므로, 이를 독립적인 분류층에 입력하여 각 토큰에 해당하는 태그(예: B-PER, I-LOC, O 등)를 예측한다.41 WordPiece 토큰화로 인해 하나의 단어가 여러 하위 토큰으로 나뉠 수 있으므로, 일반적으로는 각 단어의 첫 번째 하위 토큰에 대한 예측만을 사용하고 나머지 하위 토큰의 예측은 무시하는 방식으로 레이블을 정렬한다.44


SQuAD(Stanford Question Answering Dataset)와 같은 추출적 질의응답 과제는 주어진 지문(passage) 내에서 질문에 대한 정답에 해당하는 텍스트 구간(span)의 시작과 끝 위치를 찾는 것을 목표로 한다.

- **메커니즘:** 이 과제를 위해 입력은  질문 지문의 형태로 구성된다. 미세 조정 단계에서 모델은 두 개의 새로운 벡터, 즉 시작 위치를 예측하기 위한 시작 벡터 $S \in \mathbb{R}^H$와 끝 위치를 예측하기 위한 끝 벡터 $E \in \mathbb{R}^H$를 학습한다. 지문 내의 각 토큰 $i$가 정답의 시작일 확률은 해당 토큰의 최종 은닉 상태 벡터 $T_i$와 시작 벡터 $S$의 내적(dot product)을 계산한 후, 지문 내 모든 토큰에 대해 소프트맥스 함수를 적용하여 계산된다. 끝 위치일 확률도 끝 벡터 $E$를 사용하여 동일한 방식으로 계산된다.26

$$
P_i^{\text{start}} = \frac{e^{S \cdot T_i}}{\sum_j e^{S \cdot T_j}}
$$

$$
P_i^{\text{end}} = \frac{e^{E \cdot T_i}}{\sum_j e^{E \cdot T_j}}
$$

훈련 시 손실 함수는 정답 시작 위치와 끝 위치에 대한 로그 우도(log-likelihood)의 합으로 정의되며, 예측 시에는 시작 확률과 끝 확률이 가장 높은 토큰 쌍 $(i, j)$ (단, $j \ge i$)을 정답 구간으로 선택한다.


BERT는 발표와 동시에 자연어 처리 분야의 다양한 표준 벤치마크에서 기존의 모든 기록을 경신하며 그 효과를 입증했다. 이는 BERT가 제안한 깊은 양방향 표현 학습과 사전 훈련-미세 조정 패러다임이 이론적으로만 우수한 것이 아니라, 실증적으로도 압도적인 성능 향상을 가져왔음을 보여주는 명백한 증거였다.


BERT의 성능은 특히 자연어 이해 능력을 종합적으로 평가하는 GLUE 벤치마크와 기계 독해 능력을 측정하는 SQuAD 벤치마크에서 두드러졌다.


GLUE(General Language Understanding Evaluation)는 자연어 추론, 감성 분석, 문장 유사도 등 9개의 다양한 자연어 이해 과제로 구성된 벤치마크 모음이다. BERT는 이 벤치마크에서 기존 SOTA 모델들을 큰 차이로 따돌렸다. BERT-LARGE 모델은 GLUE 테스트 세트에서 80.5%의 종합 점수를 기록했으며, 이는 이전 최고 기록이었던 OpenAI GPT의 72.8% 대비 7.7% 포인트에 달하는 절대적인 성능 향상이었다.9

| 과제          | 이전 SOTA (OpenAI GPT) | BERT-LARGE |
| ------------- | ---------------------- | ------------ |
| MNLI          | 82.1                   | 86.7         |
| QQP           | 71.2                   | 72.1         |
| QNLI          | 82.3                   | 91.1         |
| SST-2         | 91.3                   | 93.5         |
| CoLA          | 35.0                   | 60.5         |
| STS-B         | 81.0                   | 87.1         |
| MRPC          | 82.3                   | 86.0         |
| RTE           | 61.7                   | 70.1         |
| **평균 점수** | **72.8**               | **80.5**     |

표 1: GLUE 테스트 세트에서의 BERT 성능 비교 (Devlin et al., 2019 기반) 12

이러한 결과는 BERT가 단일 과제에 특화된 모델이 아니라, 다양한 종류의 언어 이해 작업에 보편적으로 적용될 수 있는 강력한 언어 표현을 학습했음을 증명했다.


SQuAD(Stanford Question Answering Dataset)는 기계 독해 및 질의응답 시스템의 성능을 평가하는 표준 벤치마크이다. BERT는 이 과제에서도 전례 없는 성능을 보였다.

- **SQuAD v1.1:** 이 버전은 주어진 지문에서 항상 정답을 찾을 수 있는 질문들로 구성된다. BERT-LARGE는 이 데이터셋에서 F1 점수 93.2를 달성하여, 이전 최고 기록을 1.5 포인트 경신했다.12
- **SQuAD v2.0:** 이 버전은 지문 내에 정답이 없는 '답변 불가능한' 질문을 포함하여 더 높은 추론 능력을 요구한다. BERT는 이 더 어려운 과제에서 F1 점수 83.1을 기록하며 이전 SOTA 대비 5.1 포인트라는 엄청난 성능 향상을 이루었다.48

| 벤치마크        | 이전 SOTA | BERT-LARGE |
| --------------- | --------- | ------------ |
| SQuAD v1.1 (F1) | 91.7      | 93.2         |
| SQuAD v2.0 (F1) | 78.0      | 83.1         |

표 2: SQuAD 테스트 세트에서의 BERT 성능 비교 (Devlin et al., 2019 기반) 50

특히 SQuAD v2.0에서의 큰 폭의 성능 향상은 BERT의 깊은 양방향 문맥 이해 능력이 단순히 단어를 맞추는 것을 넘어, 질문과 지문 간의 미묘한 논리적 관계를 파악하고 정답의 존재 여부를 판단하는 고차원적인 추론 능력에 기여했음을 시사한다.


BERT의 영향력은 학계를 넘어 산업계로 빠르게 확산되었으며, 가장 대표적인 사례는 구글 검색 엔진에의 통합이다. 2019년 10월, 구글은 자사 블로그를 통해 BERT를 검색 순위 결정 및 추천 스니펫(featured snippets) 생성에 적용하기 시작했다고 공식적으로 발표했다.52

구글에 따르면, BERT는 특히 'to', 'for'와 같은 전치사의 역할이 중요하거나, 구어체적이고 긴 형태의 검색어의 의도를 파악하는 데 탁월한 성능을 보였다.53 이는 사용자가 더 이상 키워드 중심의 파편적인 검색어('keyword-ese')를 사용할 필요 없이, 자연스러운 문장으로 질문해도 검색 엔진이 그 미묘한 뉘앙스를 이해하고 더 정확한 결과를 제공할 수 있게 되었음을 의미한다.

구글이 제시한 예시는 BERT의 실제적 가치를 명확히 보여준다 53:

- **검색어:** "2019 brazil traveler to usa need a visa" (2019년 브라질 여행객이 미국에 가려면 비자가 필요한가)
  - **이전:** 'to'의 중요성을 간과하여 미국 시민이 브라질로 여행하는 것에 대한 정보를 반환했다.
  - **BERT 적용 후:** 'to usa'라는 방향성을 정확히 이해하여 브라질 국적자가 미국 비자를 받는 방법에 대한 결과를 제공했다.
- **검색어:** "do estheticians stand a lot at work" (피부 관리사는 서서 일하는 시간이 많은가)
  - **이전:** 'stand'라는 단어에만 집중하여 'stand-alone'과 같은 관련 없는 단어와 매칭했다.
  - **BERT 적용 후:** 'stand'가 '직업의 신체적 요구'라는 문맥 속에서 사용되었음을 이해하고 더 유용한 답변을 제공했다.

이러한 변화는 구글 검색 결과의 약 10%에 영향을 미쳤으며, 이는 검색 기술 역사상 가장 중요한 발전 중 하나로 평가받는다.54 BERT의 통합은 검색 엔진이 단순한 키워드 매칭 시스템에서 진정한 의미의 '언어 이해 엔진'으로 진화하는 결정적인 계기가 되었다.


BERT가 NLP 분야에 가져온 혁신은 부인할 수 없지만, 그 성공은 동시에 더 깊은 탐구와 비판적 분석의 대상이 되었다. BERT의 발표 이후, 수많은 후속 연구들은 BERT의 설계 원리를 분석하고, 그 한계를 극복하며, 잠재력을 최대한으로 끌어내기 위한 노력을 이어갔다. 이 과정에서 'BERTology'라는 신조어가 생겨날 정도로, BERT의 내부 작동 방식과 최적화에 대한 연구가 활발히 진행되었다. 이러한 연구들은 BERT의 핵심 아키텍처는 유지하되, 사전 훈련 방식과 목표를 개선하는 방향으로 진화했다.


후속 연구들은 BERT의 독창적인 두 가지 사전 훈련 과제, 즉 MLM과 NSP에 대해 심도 있는 분석을 수행했으며, 특히 NSP 과제의 효과성에 대한 중요한 의문을 제기했다.


BERT의 저자들은 NSP가 문장 간 관계 이해에 필수적이라고 주장했지만, RoBERTa, ALBERT, XLNet과 같은 후속 모델들은 실험을 통해 NSP 과제가 오히려 성능에 해가 되거나, 적어도 큰 도움이 되지 않는다는 사실을 입증했다.18

이 문제의 근본 원인은 NSP 과제가 '주제 예측(topic prediction)'과 '일관성 예측(coherence prediction)'이라는 두 가지 하위 과제를 분리하지 않고 혼합했다는 점에 있다.56 NSP의 부정 예시(NotNext)는 서로 다른 문서에서 문장을 무작위로 가져와 구성된다. 이 경우, 두 문장은 주제도 다르고 내용의 흐름도 일관성이 없다. 모델은 두 문장의 주제가 다른지만 파악해도 매우 쉽게 'NotNext'라고 예측할 수 있다. MLM 과제를 통해 이미 단어 수준에서 주제를 학습한 모델에게 이는 너무나 쉬운 과제였다. 결과적으로, 모델은 더 어려운 과제인 문장 간의 미묘한 논리적 일관성을 학습하기보다는, 쉬운 단서인 주제 변화에만 의존하게 되어 진정한 담화 이해 능력을 기르지 못했다는 것이다.56


BERT의 기본 철학을 계승하면서도 그 한계를 극복하려는 노력은 다양한 변형 모델의 탄생으로 이어졌다. 이 모델들은 BERT의 성공이 아키텍처 자체뿐만 아니라, '어떻게 훈련시키는가'에 크게 좌우된다는 점을 명확히 보여주었다.

- **RoBERTa (A Robustly Optimized BERT Pretraining Approach):** 2019년 페이스북 AI가 발표한 RoBERTa는 BERT의 아키텍처를 거의 그대로 유지하면서, 사전 훈련 '레시피'를 최적화하는 것만으로도 성능을 크게 향상시킬 수 있음을 보여주었다.18 RoBERTa의 핵심적인 개선점은 다음과 같다 57:

  - **NSP 과제 제거:** NSP가 비효율적임을 확인하고, 이를 제거한 뒤 단순히 같은 문서에서 연속된 문장들을 입력으로 사용했다.

  - **동적 마스킹 (Dynamic Masking):** 원본 BERT는 데이터 전처리 과정에서 마스킹을 한 번만 수행하여 고정된 마스크를 사용했지만, RoBERTa는 훈련 데이터를 모델에 입력할 때마다 새로운 마스킹 패턴을 생성하여 학습의 다양성을 높였다.

  - 더 많은 데이터와 더 긴 훈련: 훨씬 더 큰 데이터셋으로, 더 큰 배치 사이즈를 사용하여 더 오랜 시간 동안 훈련시켰다.

    이러한 엔지니어링적 개선만으로 RoBERTa는 BERT의 성능을 뛰어넘었으며, 이는 원본 BERT가 '덜 훈련된(undertrained)' 상태였음을 시사했다.55

- **ALBERT (A Lite BERT):** 같은 해 구글이 발표한 ALBERT는 BERT의 거대한 모델 크기와 메모리 문제를 해결하는 데 중점을 두었다.18 ALBERT는 두 가지 혁신적인 파라미터 감소 기법을 도입했다:

  - **인자화된 임베딩 파라미터화 (Factorized Embedding Parameterization):** 단어 임베딩 행렬의 차원($E$)과 트랜스포머 은닉층의 차원($H$)을 분리했다. 이를 통해 거대한 어휘 사전에 대한 임베딩 행렬의 크기를 줄이면서도, 모델 내부의 표현력은 유지할 수 있었다.

  - 교차-레이어 파라미터 공유 (Cross-layer Parameter Sharing): 모든 트랜스포머 레이어가 동일한 파라미터를 공유하도록 하여, 모델의 깊이가 증가해도 전체 파라미터 수가 늘어나지 않도록 했다.

    또한, ALBERT는 비효율적인 NSP를 '문장 순서 예측(Sentence-Order Prediction, SOP)' 과제로 대체했다.59 SOP는 같은 문서에서 연속된 두 문장의 순서를 바꾸어 부정 예시를 생성함으로써, 모델이 주제가 아닌 문장 간의 논리적 순서와 일관성을 학습하도록 강제했다.

- **ELECTRA (Efficiently Learning an Encoder that Classifies Token Replacements Accurately):** 2020년 스탠포드와 구글 연구진이 제안한 ELECTRA는 MLM의 샘플 비효율성 문제를 정면으로 다루었다.18 MLM은 전체 토큰의 15%에 대해서만 학습 신호를 받지만, ELECTRA는 모든 입력 토큰으로부터 학습한다.

  - ELECTRA는 생성적 적대 신경망(GAN)과 유사한 구조를 가진다. 작은 '생성자(generator)' 모델(일반적인 MLM)이 입력의 일부 토큰을 그럴듯한 다른 토큰으로 교체하면, 메인 모델인 '판별자(discriminator)'(ELECTRA)는 각 토큰이 원래 토큰인지 아니면 생성자에 의해 교체된 토큰인지를 예측한다.60 이 '교체된 토큰 탐지(replaced token detection)' 과제는 모델이 모든 토큰에 대해 예측을 수행하게 하므로, 훨씬 더 계산 효율적이고 샘플 효율적인 사전 훈련을 가능하게 했다.

이러한 후속 연구들의 성공은 BERT가 제시한 깊은 양방향 트랜스포머 인코더라는 핵심 개념이 매우 견고했음을 방증한다. 성능 향상의 주요 동력은 아키텍처의 근본적인 재설계가 아니라, 사전 훈련 목표와 훈련 방법론을 더욱 정교하게 다듬는 과정에서 나왔다. 이는 NLP 연구에서 자기지도 학습(self-supervised learning) 목표의 설계가 모델 아키텍처만큼이나, 혹은 그 이상으로 중요하다는 교훈을 남겼다.


2018년 BERT의 등장은 자연어 처리 분야의 지형을 영구적으로 바꾸어 놓은 지각 변동이었다. BERT는 단순한 모델의 발전을 넘어, 언어 표현을 학습하고 활용하는 방식에 대한 근본적인 패러다임 전환을 이끌었다. 그 유산은 오늘날의 거대 언어 모델(Large Language Models, LLMs)의 근간을 이루며, NLP 연구와 응용의 방향을 제시하고 있다.


BERT의 기여는 세 가지 핵심적인 측면으로 요약할 수 있다.

1. **깊은 양방향 사전 훈련의 확립:** BERT는 마스크 언어 모델(MLM)이라는 독창적인 목표를 통해, 모델의 모든 층에서 양쪽 문맥을 동시에 고려하는 깊은 양방향 표현 학습이 가능함을 최초로 증명했다. 이는 이전의 단방향 또는 얕은 양방향 모델들이 가졌던 문맥 이해의 본질적인 한계를 극복하고, 기계가 인간과 유사하게 문장의 전체적인 의미를 파악할 수 있는 길을 열었다.
2. **트랜스포머 아키텍처의 표준화:** BERT의 압도적인 성공은 트랜스포머 인코더 아키텍처가 언어 이해를 위한 사실상의 표준(de facto standard)으로 자리매김하게 만들었다. 순환 구조 없이 어텐션 메커니즘만으로 장거리 의존성을 효과적으로 포착하는 트랜스포머의 능력은 BERT를 통해 그 잠재력이 완전히 입증되었다.62
3. **'사전 훈련 후 미세 조정' 패러다임의 대중화:** BERT는 대규모 비지도 데이터로 언어에 대한 보편적 지식을 학습한 후, 소량의 지도 데이터로 특정 과제에 맞게 모델 전체를 미세 조정하는 패러다임의 효율성과 강력함을 명확히 보여주었다.10 이 접근법은 각 과제마다 복잡한 맞춤형 아키텍처를 설계해야 했던 기존의 방식을 대체했으며, 한정된 레이블 데이터만으로도 높은 성능을 달성할 수 있게 하여 NLP 기술의 민주화에 크게 기여했다.


자연어 처리 기술의 진화 과정에서 BERT는 초기 문맥 모델과 현대적 생성 LLM을 잇는 결정적인 교량 역할을 했다. ELMo와 같은 모델들이 문맥의 중요성을 처음으로 제시했다면, BERT는 트랜스포머 아키텍처 위에서 그 개념을 완성하고, '언어 이해(language understanding)'라는 과제를 전례 없는 수준으로 끌어올렸다.18

오늘날 GPT-3, GPT-4와 같은 생성형 모델들이 텍스트 생성, 제로샷/퓨샷 학습 등에서 놀라운 능력을 보여주면서, BERT와 같은 인코더-온리(encoder-only) 모델의 역할은 다소 변화했다.65 그러나 현대 LLM들의 성공은 BERT가 닦아놓은 기반 위에서 가능했다. BERT가 개척한 대규모 사전 훈련, 트랜스포머 아키텍처의 활용, 전이 학습의 철학은 모두 현재 LLM들의 핵심 구성 요소이다.

결론적으로, BERT의 가장 위대한 유산은 기계가 언어를 '이해'하는 방식에 대한 우리의 기대를 영원히 바꾸어 놓았다는 점이다. BERT는 언어의 복잡성과 미묘함을 포착하는 강력한 표현을 학습하는 방법을 제시했으며, 이는 이후 더욱 정교한 '언어 생성(language generation)' 모델들이 탄생할 수 있는 필수적인 전제 조건이 되었다. BERT는 하나의 모델을 넘어, NLP의 새로운 시대를 연 이정표로 역사에 기록될 것이다.


1. [1810.04805] BERT: Pre-training of Deep Bidirectional Transformers ..., accessed July 20, 2025, https://ar5iv.labs.arxiv.org/html/1810.04805
2. [D] What are the main differences between the word embeddings of ELMo, BERT, Word2vec, and GloVe? : r/MachineLearning - Reddit, accessed July 20, 2025, https://www.reddit.com/r/MachineLearning/comments/aptwxm/d_what_are_the_main_differences_between_the_word/
3. BERT vs GPT: Comparing the Two Most Popular Language Models - InvGate ITSM blog, accessed July 20, 2025, https://blog.invgate.com/gpt-3-vs-bert
4. [R] BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding : r/MachineLearning - Reddit, accessed July 20, 2025, https://www.reddit.com/r/MachineLearning/comments/9nfqxz/r_bert_pretraining_of_deep_bidirectional/
5. BERT: A Detailed Guide to Clear Up All Your Confusions, accessed July 20, 2025, https://www.garysnotebook.com/20210117_1/
6. deep learning - What are some key strengths of BERT over ELMO ..., accessed July 20, 2025, https://datascience.stackexchange.com/questions/68155/what-are-some-key-strengths-of-bert-over-elmo-ulmfit
7. Differences between BERT, GPT, and ELMo. BERT uses a bi-directional... - ResearchGate, accessed July 20, 2025, https://www.researchgate.net/figure/Differences-between-BERT-GPT-and-ELMo-BERT-uses-a-bi-directional-Transformer-OpenAI_fig2_340797092
8. Comparison of BERT, OpenAI GPT and ELMo model architectures [5]. - ResearchGate, accessed July 20, 2025, https://www.researchgate.net/figure/Comparison-of-BERT-OpenAI-GPT-and-ELMo-model-architectures-5_fig1_338931711
9. BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding | Request PDF - ResearchGate, accessed July 20, 2025, https://www.researchgate.net/publication/328230984_BERT_Pre-training_of_Deep_Bidirectional_Transformers_for_Language_Understanding
10. BERT Vs GPT-4: Evolution Of Transformer Model - Xonique, accessed July 20, 2025, https://xonique.dev/blog/the-evolution-of-transformer-models-in-nlp/
11. BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding (2018) | Jacob Devlin | 28110 Citations - SciSpace, accessed July 20, 2025, https://scispace.com/papers/bert-pre-training-of-deep-bidirectional-transformers-for-3mez6kncl2
12. BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding, accessed July 20, 2025, https://aclanthology.org/N19-1423/
13. The Illustrated Transformer – Jay Alammar – Visualizing machine ..., accessed July 20, 2025, https://jalammar.github.io/illustrated-transformer/
14. Attention Is All You Need - Wikipedia, accessed July 20, 2025, https://en.wikipedia.org/wiki/Attention_Is_All_You_Need
15. Attention is All you Need - NIPS, accessed July 20, 2025, https://papers.nips.cc/paper/7181-attention-is-all-you-need
16. Attention Is All You Need, accessed July 20, 2025, https://arxiv.org/abs/1706.03762
17. The Illustrated BERT, ELMo, and co. (How NLP Cracked Transfer ..., accessed July 20, 2025, https://jalammar.github.io/illustrated-bert/
18. BERT (language model) - Wikipedia, accessed July 20, 2025, https://en.wikipedia.org/wiki/BERT_(language_model)
19. Lecture 14. Self-attention Mechanism and Transformers - Math.Utah.Edu, accessed July 20, 2025, https://www.math.utah.edu/~bwang/mathds/Lecture14.pdf
20. The Attention Mechanism from Scratch - MachineLearningMastery ..., accessed July 20, 2025, https://machinelearningmastery.com/the-attention-mechanism-from-scratch/
21. The Detailed Explanation of Self-Attention in Simple Words | by Maninder Singh | Medium, accessed July 20, 2025, https://medium.com/@manindersingh120996/the-detailed-explanation-of-self-attention-in-simple-words-dec917f83ef3
22. Understanding Attention Mechanism, Self-Attention Mechanism and Multi-Head Self-Attention Mechanism | by Sapna Limbu | Medium, accessed July 20, 2025, https://medium.com/@limbusapna3/understanding-attention-mechanism-self-attention-mechanism-and-multi-head-self-attention-mechanism-94d14e937820
23. Understanding BERT. BERT (Bidirectional Encoder... | by Shweta Baranwal - Towards AI, accessed July 20, 2025, https://pub.towardsai.net/understanding-bert-b69ce7ad03c1
24. BERT: A Comprehensive Introduction | by Shirley Li - Medium, accessed July 20, 2025, https://medium.com/@lixue421/bert-a-comprehensive-introduction-7d620efcd32f
25. Bert Model - A State of the Art NLP Model Explained - Metaschool, accessed July 20, 2025, https://metaschool.so/articles/bert-model
26. Pre-training the BERT model - Scaler Topics, accessed July 20, 2025, https://www.scaler.com/topics/nlp/pre-training-bert/
27. Next Sentence Prediction using BERT - GeeksforGeeks, accessed July 20, 2025, https://www.geeksforgeeks.org/machine-learning/next-sentence-prediction-using-bert/
28. BERT Explained - Papers With Code, accessed July 20, 2025, https://paperswithcode.com/method/bert
29. What is BERT and how is it Used in AI?, accessed July 20, 2025, https://h2o.ai/wiki/bert/
30. BERT 101 - State Of The Art NLP Model Explained - Hugging Face, accessed July 20, 2025, https://huggingface.co/blog/bert-101
31. BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding, accessed July 20, 2025, https://blog.paperspace.com/bert-pre-training-of-deep-bidirectional-transformers-for-language-understanding/
32. What are masked language models? - IBM, accessed July 20, 2025, https://www.ibm.com/think/topics/masked-language-model
33. Masked Language Model: All you need to Know | by Amit Yadav - Medium, accessed July 20, 2025, https://medium.com/@amit25173/masked-language-model-all-you-need-to-know-12ab35319d09
34. Masked Language Model (MLM), accessed July 20, 2025, https://nn.labml.ai/transformers/mlm/index.html
35. NSP in BERT. NSP, or Next Sentence Prediction, is a... | by Tiya Vaj | Medium, accessed July 20, 2025, https://vtiya.medium.com/nsp-in-bert-82af9035532a
36. Understanding BERT: The Revolution in Natural Language Processing - interScan LLC, accessed July 20, 2025, https://www.interscanllc.com/blog/understanding-bert-the-revolution-in-natural-language-processing
37. Predicting the next sentence (not word) in large language models: What model-brain alignment tells us about discourse comprehension - PubMed Central, accessed July 20, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11114233/
38. BERT Fine-Tuning Sentence Classification.ipynb - Colab - Google, accessed July 20, 2025, https://colab.research.google.com/github/orico/NLP/blob/master/BERT_Fine_Tuning_Sentence_Classification.ipynb
39. How to Fine-Tune BERT for Text Classification | Finetune-BERT-Text ..., accessed July 20, 2025, https://wandb.ai/akshayuppal12/Finetune-BERT-Text-Classification/reports/How-to-Fine-Tune-BERT-for-Text-Classification--Vmlldzo4OTk4MzY
40. Fine-Tuning BERT for Classification: A Practical Guide | by Hey Amit - Medium, accessed July 20, 2025, https://medium.com/@heyamit10/fine-tuning-bert-for-classification-a-practical-guide-b8c1c56f252c
41. Bert Explained and Fine-Tuned for NER - Kaggle, accessed July 20, 2025, https://www.kaggle.com/code/qmarva/bert-explained-and-fine-tuned-for-ner
42. How to Do Named Entity Recognition (NER) with a BERT Model - MachineLearningMastery.com, accessed July 20, 2025, https://machinelearningmastery.com/how-to-do-named-entity-recognition-ner-with-a-bert-model/
43. Fine-tune BERT Model for Named Entity Recognition in Google Colab - Analytics Vidhya, accessed July 20, 2025, https://www.analyticsvidhya.com/blog/2022/06/fine-tune-bert-model-for-named-entity-recognition-in-google-colab/
44. How to fine tune bert on entity recognition? - Beginners - Hugging Face Forums, accessed July 20, 2025, https://discuss.huggingface.co/t/how-to-fine-tune-bert-on-entity-recognition/11309
45. Question Answering with a Fine-Tuned BERT.ipynb - Colab, accessed July 20, 2025, https://colab.research.google.com/drive/1uSlWtJdZmLrI3FCNIlUHFxwAJiSu2J0-
46. Question-Answering using Bert - Kaggle, accessed July 20, 2025, https://www.kaggle.com/code/arunmohan003/question-answering-using-bert
47. [1810.04805] BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding - arXiv, accessed July 20, 2025, https://arxiv.org/abs/1810.04805
48. BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding, accessed July 20, 2025, https://huggingface.co/papers/1810.04805
49. GLUE Benchmark (Natural Language Understanding) | Papers With ..., accessed July 20, 2025, https://paperswithcode.com/sota/natural-language-understanding-on-glue
50. SQuAD2.0 Benchmark (Question Answering) | Papers With Code, accessed July 20, 2025, https://paperswithcode.com/sota/question-answering-on-squad20
51. SQuAD1.1 Benchmark (Question Answering) - Papers With Code, accessed July 20, 2025, https://paperswithcode.com/sota/question-answering-on-squad11
52. How Google's BERT Changed Natural Language Understanding | Brave River Solutions, accessed July 20, 2025, https://www.braveriver.com/blog/how-googles-bert-changed-natural-language-understanding/
53. Understanding searches better than ever before - Google Blog, accessed July 20, 2025, https://blog.google/products/search/search-language-understanding-bert/
54. FAQ: All about the BERT algorithm in Google search - Search Engine Land, accessed July 20, 2025, https://searchengineland.com/faq-all-about-the-bert-algorithm-in-google-search-324193
55. RoBERTa: A Robustly Optimized BERT Pretraining Approach | OpenReview, accessed July 20, 2025, https://openreview.net/forum?id=SyxS0T4tvS
56. transformers - Why is the NSP task in BERT inconsistent or ..., accessed July 20, 2025, https://stats.stackexchange.com/questions/545573/why-is-the-nsp-task-in-bert-inconsistent-or-ineffective
57. RoBERTa: A Robustly Optimized BERT Pretraining Approach, accessed July 20, 2025, https://arxiv.org/abs/1907.11692
58. Review - RoBERTa: A Robustly Optimized BERT Pretraining Approach | by Sik-Ho Tsang, accessed July 20, 2025, https://sh-tsang.medium.com/review-roberta-a-robustly-optimized-bert-pretraining-approach-d1be4014e5ce
59. ALBERT: A Lite BERT for Self-supervised Learning of Language ..., accessed July 20, 2025, https://arxiv.org/abs/1909.11942
60. google-research/electra: ELECTRA: Pre-training Text ... - GitHub, accessed July 20, 2025, https://github.com/google-research/electra
61. ELECTRA: Pre-training Text Encoders as Discriminators Rather Than Generators | Request PDF - ResearchGate, accessed July 20, 2025, https://www.researchgate.net/publication/340134249_ELECTRA_Pre-training_Text_Encoders_as_Discriminators_Rather_Than_Generators
62. Revolutionizing Language Technologies: The Impact of Transformer Models on NLP Progress - ResearchGate, accessed July 20, 2025, https://www.researchgate.net/publication/389056582_Revolutionizing_Language_Technologies_The_Impact_of_Transformer_Models_on_NLP_Progress
63. Unlocking the Power of NLP with AI Development Services - How BERT Works, accessed July 20, 2025, https://www.q3tech.com/blogs/a-deep-dive-into-bert/
64. Evolution of Language Models: From Rules-Based Models to LLMs, accessed July 20, 2025, https://www.appypieagents.ai/blog/evolution-of-language-models
65. Large language model - Wikipedia, accessed July 20, 2025, https://en.wikipedia.org/wiki/Large_language_model

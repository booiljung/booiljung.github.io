# Scaled Dot-Product Attention
[트랜스포머 (Transformer)](./index.md)


자연어 처리(Natural Language Processing, NLP)의 역사는 순차적 데이터(sequential data)를 효과적으로 모델링하려는 노력의 연속이었다. 특히, 순환 신경망(Recurrent Neural Network, RNN)과 이를 개선한 Long Short-Term Memory(LSTM) 및 Gated Recurrent Unit(GRU)은 기계 번역, 텍스트 요약 등과 같은 Sequence-to-Sequence(Seq2Seq) 과제에서 지배적인 패러다임을 형성했다.1 그러나 이러한 RNN 기반 모델들은 구조적인 한계를 내포하고 있었다.


RNN 기반 Seq2Seq 모델의 핵심은 인코더(Encoder)가 입력 시퀀스의 정보를 고정된 크기의 문맥 벡터(context vector)로 압축하고, 디코더(Decoder)가 이 문맥 벡터를 기반으로 출력 시퀀스를 생성하는 구조에 있다.1 이 구조는 두 가지 근본적인 문제를 야기했다.

첫째, **장기 의존성 문제(Long-Range Dependency Problem)**이다. 정보는 시퀀스를 따라 타임스텝을 거치며 순차적으로 처리된다.2 이 과정에서 시퀀스 초반부의 정보는 여러 계산 단계를 거치면서 점차 희석되거나 소실될 위험이 크다. 결과적으로 모델은 문장의 먼 부분에 있는 단어 간의 의미적 관계를 포착하는 데 어려움을 겪었다.

둘째, **정보 병목 현상(Information Bottleneck)**이다. 입력 시퀀스의 길이가 아무리 길더라도, 모든 정보는 고정된 크기의 단일 문맥 벡터 안에 압축되어야만 했다.1 이는 긴 문장의 복잡하고 미묘한 의미를 온전히 담아내기에는 역부족이었으며, 이는 모델 성능 저하의 직접적인 원인이 되었다.


이러한 한계를 극복하기 위해 2014년 Bahdanau 등이 제안한 어텐션(Attention) 메커니즘은 패러다임의 전환을 가져왔다.5 어텐션의 핵심 아이디어는 디코더가 출력 단어를 예측하는 매 시점마다, 고정된 문맥 벡터 하나에만 의존하는 대신, 인코더의 모든 타임스텝 은닉 상태(hidden states)를 직접 참조하여 관련성이 높은 정보에 '집중(attention)'하는 것이다.6

어텐션의 본질은 '관심' 또는 '주의'를 의미하며 7, 모델이 출력의 각 요소를 생성할 때 입력의 어느 부분에 더 많은 가중치를 부여할지 동적으로 결정하는 메커니즘으로 정의할 수 있다.1 이를 통해 정보 병목 현상을 해소하고 장거리 의존성 문제를 효과적으로 완화함으로써 Seq2Seq 모델의 성능을 획기적으로 개선했다.


2017년, Google 연구팀이 발표한 "Attention Is All You Need" 논문은 여기서 한 걸음 더 나아가 딥러닝 아키텍처의 역사를 새로 썼다.5 이 논문은 기존의 지배적인 구조였던 순환(Recurrence)과 합성곱(Convolution)을 완전히 배제하고, 오직 어텐션 메커니즘만으로 최첨단(state-of-the-art) 성능을 달성할 수 있음을 증명했다.2

이 혁신의 중심에는 **Scaled Dot-Product Attention**이라는 새롭고 효율적인 어텐션 메커니즘이 있었다.17 이는 트랜스포머(Transformer) 아키텍처를 구성하는 기본 단위(building block)가 되었으며, 순차적 계산의 족쇄에서 벗어나 대규모 병렬 처리를 가능하게 했다.16

이러한 변화는 단순히 모델의 학습 속도를 개선하는 것을 넘어, 계산 패러다임 자체를 전환시켰다. RNN의 본질은 $h_t = f(h_{t-1}, x_t)$라는 순차적(sequential) 계산에 있으며, 이는 시퀀스 길이에 따라 계산 시간이 선형적으로 증가하고 병렬화가 근본적으로 불가능한 구조다. 반면, Scaled Dot-Product Attention은 시퀀스 내 모든 토큰 쌍 간의 관계를 단 한 번의 행렬 연산으로 계산할 수 있게 했다.13 이는 GPU와 같은 하드웨어의 병렬 처리 능력을 극대화하는 구조적 혁신이었다. 이로 인해 이전에는 상상할 수 없었던 규모의 데이터와 모델 파라미터를 학습시키는 것이 가능해졌고, 이는 BERT, GPT와 같은 거대 언어 모델(Large Language Models, LLMs)의 탄생을 이끈 근본적인 동력이 되었다. 즉, 어텐션은 단순한 성능 향상 기법을 넘어, 딥러닝 모델의 규모 확장(scale-up)을 가능하게 한 계산 패러다임의 전환 그 자체였다.


Scaled Dot-Product Attention은 트랜스포머 아키텍처의 심장부로서, 그 효율성과 강력함은 간결한 수학적 구조에서 비롯된다. 이를 이해하기 위해 먼저 어텐션 함수의 일반적인 정의부터 살펴볼 필요가 있다.


어텐션 함수는 하나의 쿼리(Query)와 다수의 키-값(Key-Value) 쌍을 입력으로 받아 출력을 계산하는 매핑(mapping) 함수로 개념화할 수 있다.3 여기서 쿼리, 키, 값, 그리고 출력은 모두 벡터다.

출력은 값(Value) 벡터들의 가중합(Weighted Sum)으로 계산된다. 이때 각 값 벡터에 할당되는 가중치(weight)는 해당 쿼리 벡터와 그에 대응하는 키(Key) 벡터 간의 유사도(compatibility 또는 similarity)에 의해 결정된다.3 즉, 쿼리와 특정 키가 유사할수록, 해당 키와 쌍을 이루는 값 벡터가 최종 출력에 더 큰 영향을 미치게 된다.


Scaled Dot-Product Attention은 이러한 일반적인 어텐션 함수의 구체적인 구현체 중 하나로, 다음과 같은 핵심 아이디어에 기반한다.

1. **유사도 계산 함수**: 쿼리와 키 간의 유사도를 계산하기 위해 **내적(Dot-Product)** 연산을 사용한다.13 내적은 두 벡터의 방향적 유사성과 크기를 동시에 반영하는 효율적인 척도이며, 고도로 최적화된 행렬 곱셈 연산으로 구현될 수 있어 계산적으로 매우 유리하다.1
2. **스케일링(Scaling)**: 내적의 결과값이 벡터의 차원에 따라 너무 커지는 것을 방지하고 학습의 안정성을 확보하기 위해, 특정 스케일링 팩터($\sqrt{d_k}$)로 나누어준다.13 이 '스케일링' 과정이 이 메커니즘을 기존의 Dot-Product Attention과 구분 짓는 핵심적인 특징이다.
3. **정보 흐름**: 전체적인 정보 흐름은 **(1) 쿼리-키 유사도 계산 → (2) 스케일링 → (3) Softmax를 통한 가중치 정규화 → (4) 값과의 가중합** 순서로 명확하게 정의된다.19


Scaled Dot-Product Attention의 전체 연산 과정은 다음과 같은 단일 수식으로 압축적으로 표현된다.5
$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$
여기서 각 행렬은 다음과 같은 의미를 가진다.

- $Q$: 쿼리(Query)들의 행렬.
- $K$: 키(Key)들의 행렬.
- $V$: 값(Value)들의 행렬.
- $d_k$: 각 키 벡터(및 쿼리 벡터)의 차원(dimension).5
- $K^T$: 키 행렬 $K$의 전치(transpose) 행렬.

이 수식을 면밀히 살펴보면, 어텐션 함수 자체에는 학습 가능한 파라미터(가중치 행렬이나 편향)가 존재하지 않는다는 점을 알 수 있다.27 연산은 오직 행렬 곱셈, 스케일링, 소프트맥스, 그리고 또 다른 행렬 곱셈으로만 구성된다. 이는 내부에 학습 가능한 가중치 행렬 $W_a$, $v_a$ 등을 포함하는 Additive Attention과 극명한 대조를 이룬다.13

이러한 '파라미터 없는(parameter-free)' 코어 설계는 두 가지 중요한 장점을 제공한다. 첫째, 계산이 극도로 효율적이다. 고도로 최적화된 행렬 곱셈 라이브러리(예: BLAS, cuBLAS)를 직접 활용할 수 있어 하드웨어 가속에 매우 유리하다.28 둘째, 모델의 복잡성과 학습의 부담을 쿼리, 키, 값을 생성하는 외부의 선형 변환 계층으로 온전히 위임한다. 즉, 어텐션의 '코어'는 순수한 정보 라우팅 및 통합 메커니즘으로 기능하고, '무엇을' 비교할지, '어떤 정보'를 전달할지는 외부에서 학습된 파라미터가 결정하도록 역할을 명확히 분리한 것이다. 이는 매우 효율적이고 모듈화된 설계 철학을 반영하며, 트랜스포머의 성공에 핵심적인 역할을 했다.


Scaled Dot-Product Attention의 작동 원리를 깊이 이해하기 위해서는 그 구성 요소인 쿼리(Query), 키(Key), 값(Value)의 역할과 의미를 명확히 파악해야 한다. 이 세 가지 요소는 정보 검색 시스템의 유추를 통해 직관적으로 이해할 수 있다.


- **Query (Q)**: 현재 처리 중인 토큰의 입장에서 다른 모든 토큰에게 던지는 '질문' 또는 '검색어'에 해당한다. 이는 "나의 문맥을 이해하는 데 있어, 너희들 중 누가 나와 관련된 정보를 가지고 있는가?"를 묻는 벡터로 해석될 수 있다.31
- **Key (K)**: 각 토큰이 자신을 설명하기 위해 내거는 '색인(index)' 또는 '키워드'다. 쿼리의 질문에 응답할 수 있는 잠재적인 정보를 담고 있음을 나타내며, 쿼리와의 유사도 비교 대상이 된다.31
- **Value (V)**: 각 토큰이 실질적으로 보유하고 있는 '내용' 또는 '정보' 그 자체다. 쿼리와 키의 비교를 통해 높은 관련성(어텐션 가중치)이 확인되면, 해당 키와 연결된 이 값 벡터가 최종 출력에 비례하여 더 많이 반영된다.31


이 개념은 유튜브 검색 과정을 비유하여 쉽게 이해할 수 있다.36

1. 사용자가 검색창에 입력하는 텍스트가 바로 **Query**다.
2. 유튜브 시스템은 이 쿼리를 받아, 플랫폼에 있는 모든 영상의 제목, 설명, 태그 등과 비교한다. 이 영상들의 제목과 태그가 **Key**에 해당한다. 시스템은 쿼리와 모든 키를 비교하여 유사도를 측정한다.
3. 유사도 측정 결과, 관련성이 높다고 판단된 영상 콘텐츠 자체가 **Value**다. 최종적으로 사용자에게는 이 Value(영상)들이 유사도에 따라 가중치가 부여된 형태, 즉 검색 결과 순위로 제공된다.

어텐션 메커니즘은 이와 유사하게, 각 쿼리(단어)에 대해 모든 키(다른 단어들)와의 유사도를 계산하고, 이 유사도를 가중치 삼아 값(다른 단어들의 정보)들을 종합하여 문맥적으로 풍부한 새로운 표현을 만들어내는 과정이다.


모델 내에서 Q, K, V 벡터는 입력 시퀀스의 각 토큰 임베딩 벡터로부터 생성된다. 구체적으로, 각 토큰의 초기 임베딩 벡터 $x$에 대해, 모델이 학습 과정에서 최적화하는 세 개의 서로 다른 가중치 행렬 $W^Q$, $W^K$, $W^V$를 각각 곱하여 Q, K, V 벡터를 얻는다.25

- 쿼리 벡터: $q = xW^Q$
- 키 벡터: $k = xW^K$
- 값 벡터: $v = xW^V$

이 가중치 행렬들($W^Q, W^K, W^V$)은 역전파(backpropagation)를 통해 주어진 태스크(예: 번역, 분류)를 가장 잘 수행하는 방향으로 학습된다.32


특히 트랜스포머의 핵심인 셀프 어텐션(Self-Attention) 또는 내부 어텐션(Intra-Attention)은 동일한 입력 시퀀스 내에서 토큰 간의 관계를 파악하는 메커니즘이다.3 이 경우, Q, K, V는 모두 동일한 소스, 즉 입력 시퀀스의 임베딩 행렬 

$X$로부터 생성된다.39

- $Q = XW^Q$
- $K = XW^K$
- $V = XW^V$

이는 문장이 '자기 자신'의 각 부분에 주의를 기울여 스스로의 문맥적 의미를 풍부하게 만드는 과정이라고 할 수 있다.

그렇다면 왜 굳이 하나의 임베딩 벡터 $x$를 세 개의 다른 벡터 Q, K, V로 분리해야 하는가? 만약 Q=K=V=X, 즉 원래 임베딩을 그대로 사용한다면, 어텐션 스코어는 $XX^T$가 되어 단순히 토큰 임베딩 간의 고정된 유사도를 측정하는 것에 그친다. 그러나 $W^Q$, $W^K$, $W^V$라는 별도의 선형 변환(Linear Projection)을 도입함으로써, 모델은 '질문하는 공간', '색인되는 공간', '내용을 전달하는 공간'을 각각 다르게 학습할 수 있는 유연성을 얻는다.32

예를 들어, "The animal didn't cross the street because **it** was too tired"라는 문장에서 'it'이라는 토큰을 처리할 때,

1. 'it'의 **Query 벡터**는 "나는 누구를 지칭하는 대명사인가?"라는 의미를 담도록 학습될 것이다.
2. 'animal'의 **Key 벡터**는 "나는 생물이며, 대명사 'it'으로 지칭될 수 있는 속성을 가졌다"는 정보를 제공하도록 학습될 것이다.
3. 'street'의 **Key 벡터**는 "나는 사물이며, 'tired'의 주체가 되기 어렵다"는 정보를 제공하도록 학습될 것이다.
4. 'animal'의 **Value 벡터**는 'animal'이라는 단어가 가진 풍부한 의미론적 정보를 담고 있을 것이다.

결과적으로 'it'의 쿼리는 'animal'의 키와 높은 유사도를 갖게 되고, 'animal'의 값이 'it'의 새로운 표현을 형성하는 데 크게 기여하게 된다. 이처럼 Q, K, V를 분리하는 것은 각 토큰이 문맥에 따라 다른 '역할'을 수행할 수 있도록 하여, 모델의 표현력을 기하급수적으로 높이는 핵심적인 설계 결정이다.


Scaled Dot-Product Attention의 이름에 'Scaled'가 붙는 이유는 내적(dot product) 결과를 특정 값으로 나누어주는 스케일링 과정이 포함되기 때문이다. 이 스케일링은 단순한 기법이 아니라, 모델의 안정적인 학습을 보장하기 위한 필수적인 수학적 장치다.


스케일링이 없는 단순 Dot-Product Attention에서는 쿼리와 키 벡터의 차원 $d_k$가 커질수록, 두 벡터의 내적 값의 크기(magnitude)가 전반적으로 커지는 경향이 나타난다.26 이는 후속 연산인 Softmax 함수에 심각한 문제를 야기할 수 있다.


이 현상의 원인을 통계적으로 분석해 보자. 쿼리 벡터 $q$와 키 벡터 $k$의 각 성분 $q_i$와 $k_i$가 서로 독립이며, 평균이 0이고 분산이 1인 분포를 따른다고 가정하자.40

1. 내적의 기댓값(Mean):

   두 독립 확률 변수의 곱의 기댓값은 각 기댓값의 곱과 같다. 따라서 $q_i k_i$의 기댓값은 다음과 같다.
   $$
   E[q_i k_i] = E[q_i]E[k_i] = 0 \times 0 = 0
   $$
    40

   

   내적 $q \cdot k = \sum_{i=1}^{d_k} q_i k_i$의 기댓값은 기댓값의 선형성에 따라 각 성분 곱의 기댓값의 합이므로,
   $$
   E[q \cdot k] = E\left[\sum_{i=1}^{d_k} q_i k_i\right] = \sum_{i=1}^{d_k} E[q_i k_i] = \sum_{i=1}^{d_k} 0 = 0
   $$
    40

2. 내적의 분산(Variance):

   먼저 $q_i k_i$의 분산을 계산하면 다음과 같다.
   $$
   Var(q_i k_i) = E[(q_i k_i)^2] - (E[q_i k_i])^2 = E[q_i^2]E[k_i^2] - 0^2
   $$
   이때 $Var(X) = E[X^2] - (E[X])^2$이므로 $E[X^2] = Var(X) + (E[X])^2$이다. 따라서,
   $$
   E[q_i^2] = Var(q_i) + (E[q_i])^2 = 1 + 0^2 = 1
   \\
   E[k_i^2] = Var(k_i) + (E[k_i])^2 = 1 + 0^2 = 1
   $$
   결과적으로 $Var(q_i k_i) = 1 \times 1 = 1$이다.40

   내적 $q \cdot k$의 분산은 독립 확률 변수 합의 분산이 각 분산의 합과 같으므로,
   $$
   Var(q \cdot k) = Var\left(\sum_{i=1}^{d_k} q_i k_i\right) = \sum_{i=1}^{d_k} Var(q_i k_i) = \sum_{i=1}^{d_k} 1 = d_k
   $$
   40

이 분석은 쿼리와 키의 내적 값의 분산이 키 벡터의 차원 $d_k$에 정비례하여 커진다는 것을 명확히 보여준다.


분산이 $d_k$로 커진다는 것은, 내적 값들이 0을 중심으로 더 넓게 퍼져 극단적인 값(매우 크거나 매우 작은 값)을 가질 확률이 높아짐을 의미한다. 이러한 큰 값들이 Softmax 함수에 입력되면 심각한 문제가 발생한다. Softmax 함수는 입력 값들 간의 차이가 클 경우, 가장 큰 값을 가진 입력에 대해서는 거의 1에 가까운 확률을, 나머지 입력에 대해서는 거의 0에 가까운 확률을 출력한다.40

이는 Softmax 함수를 그래디언트(gradient)가 거의 0에 가까운 포화(saturation) 영역으로 밀어 넣는 결과를 초래한다. 역전파 과정에서 그래디언트가 0에 가까워지면 가중치 업데이트가 거의 일어나지 않는 **그래디언트 소실(vanishing gradient)** 문제가 발생하며, 이는 모델의 학습을 심각하게 저해한다.26


이 문제를 해결하기 위해 "Attention Is All You Need" 논문은 내적 결과를 $\sqrt{d_k}$로 나누는 스케일링을 제안했다. 확률 변수 $X$를 상수 $c$로 나눈 새로운 확률 변수 $X/c$의 분산은 $Var(X/c) = Var(X)/c^2$ 공식을 따른다.

따라서 분산이 $d_k$인 내적 값을 $\sqrt{d_k}$로 나누면, 스케일링된 값의 분산은 다음과 같이 1이 된다.
$$
Var\left(\frac{q \cdot k}{\sqrt{d_k}}\right) = \frac{Var(q \cdot k)}{(\sqrt{d_k})^2} = \frac{d_k}{d_k} = 1
$$
 40

즉, $\sqrt{d_k}$ 스케일링은 내적 값의 분산을 벡터 차원 $d_k$의 크기와 무관하게 항상 1로 정규화(normalize)하는 효과를 가진다. 이를 통해 Softmax 함수가 너무 극단적인 확률 분포 대신 더 부드러운(gentler) 분포를 출력하게 하여, 그래디언트 소실 문제를 방지하고 안정적인 학습을 보장한다.26

이 스케일링 팩터는 단순한 정규화 기법을 넘어, 모델의 확장성을 위한 필수적인 안전장치다. 모델의 표현력을 높이기 위해 임베딩 차원 $d_{model}$을 키우면, 자연스럽게 각 헤드의 차원인 $d_k$도 증가하게 된다. 만약 스케일링이 없었다면, 모델의 크기를 키우는 행위 자체가 학습의 불안정성을 증폭시켜 대형 모델의 학습을 불가능하게 만들었을 것이다.26 따라서 $\sqrt{d_k}$ 스케일링은 트랜스포머 아키텍처가 소형 모델부터 수천억 파라미터를 가진 거대 언어 모델에 이르기까지 성공적으로 **확장(scale)**될 수 있도록 만든 근본적인 설계 원칙이자 안전장치라고 평가할 수 있다.


Scaled Dot-Product Attention의 이론적 배경을 이해했다면, 이제 구체적인 연산이 어떻게 진행되는지 단계별로 살펴볼 필요가 있다. 이 과정은 여러 행렬 연산의 연속으로 이루어지며, 각 단계는 명확한 목적을 가진다.

1. **입력 준비**: 연산은 쿼리($Q$), 키($K$), 값($V$) 세 개의 행렬을 입력으로 받는다. 셀프 어텐션의 경우, 이들은 모두 동일한 입력 시퀀스 임베딩 행렬 $X$와 각각의 가중치 행렬 $W^Q, W^K, W^V$의 곱으로 생성된다.19

2. 1단계 - 유사도 계산 ($QK^T$):

   첫 번째 단계는 쿼리 행렬 $Q$와 키 행렬의 전치 $K^T$를 행렬 곱하는 것이다.19 이 연산의 결과로 어텐션 스코어(Attention Score) 행렬이 생성된다. 만약 입력 시퀀스의 길이가 `seq_len`이라면, 이 행렬의 크기는 (`seq_len`, `seq_len`)이 된다. 이 행렬의 각 원소 $(i, j)$는 `i`번째 쿼리(토큰)와 `j`번째 키(토큰) 간의 내적 유사도를 나타낸다.

3. 2단계 - 스케일링:

   다음으로, 1단계에서 얻은 어텐션 스코어 행렬의 모든 원소를 스케일링 팩터 $\sqrt{d_k}$로 나누어준다.19 여기서 

   $d_k$는 키 벡터의 차원이다. 앞서 설명했듯이, 이 과정은 내적 값의 분산을 안정시켜 학습을 원활하게 하는 핵심적인 역할을 한다.

4. 3단계 - (선택적) 마스킹(Masking):

   특정 상황에서는 어텐션이 특정 위치에 적용되는 것을 막아야 한다. 이를 위해 마스킹(masking)이 사용된다.

   - **패딩 마스크(Padding Mask):** 문장의 길이를 맞추기 위해 추가된 `<pad>` 토큰과 같은 무의미한 토큰에 대해서는 어텐션이 계산되지 않도록 해야 한다. 이를 위해 패딩 토큰에 해당하는 위치의 스코어 값에 매우 작은 음수(예: `-1e9`)를 더해준다.18 이 값은 Softmax 함수를 통과하면서 해당 위치의 어텐션 가중치를 거의 0으로 만든다.
   - **룩-어헤드 마스크(Look-ahead Mask):** 트랜스포머의 디코더에서 사용되는 마스크로, 자기회귀적(auto-regressive)으로 단어를 생성할 때 미래의 정보를 미리 참조하는 것을 방지하는 역할을 한다. `i`번째 위치의 토큰을 예측할 때는 `i`번째 위치와 그 이전 위치의 토큰들만 참조할 수 있도록, `i`보다 뒤에 있는 위치들의 스코어 값을 마스킹한다. 이는 보통 상삼각행렬(upper triangular matrix) 형태의 마스크를 적용하여 구현된다.24

5. 4단계 - 어텐션 가중치 정규화 (Softmax):

   스케일링과 마스킹이 적용된 스코어 행렬에 Softmax 함수를 행(row) 단위로 적용한다.24 Softmax 함수는 입력된 값들을 0과 1 사이의 확률 값으로 변환하며, 각 행의 모든 확률 값의 합은 1이 된다. 이렇게 생성된 행렬을 어텐션 가중치(Attention Weights) 행렬이라고 부른다. 이 행렬의 `i`번째 행은 `i`번째 쿼리가 시퀀스 내의 다른 모든 토큰(키)에 각각 얼마나 많은 '주의'를 기울여야 하는지를 나타내는 확률 분포다.

6. 5단계 - 최종 출력 계산 (weights @ V):

   마지막으로, 4단계에서 계산된 어텐션 가중치 행렬과 값(Value) 행렬 $V$를 행렬 곱한다.24 이 연산은 각 쿼리에 대해 모든 값 벡터들을 어텐션 가중치에 따라 가중합(weighted sum)하는 것과 동일한 효과를 가진다. 최종적으로 생성된 출력 행렬은 각 토큰의 위치에 전체 시퀀스의 문맥 정보가 풍부하게 함축된 새로운 표현(Contextualized Representation)이 된다.

이러한 복잡한 연산 과정은 아래 표와 같이 5개의 명확한 단계로 요약될 수 있다. 이 표는 전체 정보 흐름을 직관적으로 파악하고 각 단계의 목적과 결과를 체계적으로 이해하는 데 도움을 준다.

| 단계 | 연산          | 수식                                                 | 결과 (형태)           | 설명                                                         |
| ---- | ------------- | ---------------------------------------------------- | --------------------- | ------------------------------------------------------------ |
| 1    | 유사도 계산   | $\text{scores} = QK^T$                               | (N, seq_len, seq_len) | 각 쿼리와 모든 키 간의 내적 유사도 행렬. N은 배치 크기.      |
| 2    | 스케일링      | $\text{scaled\_scores} = \text{scores} / \sqrt{d_k}$ | (N, seq_len, seq_len) | 그래디언트 안정화를 위한 점수 조정. $d_k$는 키의 차원.       |
| 3    | 마스킹        | ``masked_scores = scaled_scores + mask``             | (N, seq_len, seq_len) | 특정 위치(패딩, 미래 토큰)의 어텐션을 방지. mask는 $-\infty$에 가까운 값을 가짐. |
| 4    | 가중치 정규화 | `weights = softmax(masked_scores, dim=-1)`           | (N, seq_len, seq_len) | 0과 1 사이의 확률 값으로 구성된 어텐션 가중치. 각 행의 합은 1. |
| 5    | 최종 출력     | `output = weights @ V`                               | (N, seq_len, $d_v$)   | 가중치가 적용된 Value 벡터의 합. 문맥 정보가 함축된 결과. $d_v$는 밸류의 차원. |


Scaled Dot-Product Attention은 그 자체로도 강력한 메커니즘이지만, 트랜스포머 아키텍처 내에서는 더 복잡하고 정교한 구조의 핵심 부품으로 활용된다. 특히, 셀프 어텐션의 구현체이자 멀티-헤드 어텐션의 기본 단위로서 그 진가를 발휘한다.

Self-Attention의 구현체

트랜스포머의 인코더와 디코더를 구성하는 각 레이어에는 셀프 어텐션 서브레이어(sub-layer)가 포함되어 있다.3 Scaled Dot-Product Attention은 바로 이 셀프 어텐션을 구현하는 구체적인 알고리즘이다. 인코더에서는 입력 시퀀스 전체에 대해, 디코더에서는 현재까지 생성된 출력 시퀀스에 대해 셀프 어텐션을 수행함으로써, 시퀀스 내 모든 단어 쌍 간의 문맥적 의존성을 포착하고 각 단어의 표현을 풍부하게 만든다.


단일 어텐션 메커니즘은 한 가지 측면의 관계성에만 집중하는 경향이 있다. 예를 들어, 특정 어텐션은 주로 구문적 관계(주어-동사)에 집중하고, 의미적 관계(동의어)는 놓칠 수 있다. 이러한 한계를 극복하기 위해 트랜스포머는 멀티-헤드 어텐션(Multi-Head Attention, MHA)이라는 개념을 도입했다.13


MHA의 핵심 아이디어는 단일 어텐션을 한 번 수행하는 대신, $h$개의 어텐션을 병렬적으로 수행한 후 그 결과들을 종합하여 사용하는 것이다.47 여러 개의 '헤드(head)'를 사용함으로써, 모델은 서로 다른 표현 부분 공간(representation subspaces)에서 다양한 종류의 관계(예: 구문적 관계, 의미적 관계, 인접 단어 관계 등)를 동시에 학습할 수 있다.13 이는 마치 여러 전문가가 각자의 관점에서 문서를 분석하고 그 결과를 종합하는 것과 같은 앙상블(ensemble) 효과를 제공하여 모델의 표현력을 극대화한다.


MHA의 연산 과정은 다음과 같다.

1. **분할(Split):** 입력 쿼리($Q$), 키($K$), 값($V$) 행렬을 $h$개의 서로 다른 학습 가능한 선형 변환(learned linear projections)을 통해 $h$쌍의 $(Q_i, K_i, V_i)$로 분할한다.3 이때 각 헤드의 벡터 차원은 원래 모델 차원 $d_{model}$을 헤드 수 $h$로 나눈 값, 즉 $d_k = d_{model} / h$, $d_v = d_{model} / h$가 된다.
2. **병렬 어텐션(Parallel Attention):** 각 헤드 $i$에 대해 독립적으로 Scaled Dot-Product Attention을 병렬 수행한다: $head_i = \text{Attention}(Q_i, K_i, V_i)$.27
3. **결합(Concatenate):** $h$개의 어텐션 헤드 출력($head_i$)을 모두 연결(concatenate)하여 하나의 큰 행렬을 만든다.27
4. **최종 변환(Final Projection):** 결합된 행렬에 또 다른 학습 가능한 가중치 행렬 $W^O$를 곱하여 최종 출력을 얻는다. 이 과정은 각 헤드에서 얻은 다양한 정보들을 통합하고, 후속 레이어에 적합한 차원으로 변환하는 역할을 한다.13


MHA의 전체 과정은 다음 수식으로 표현된다.
$$
\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_h)W^O
\\
\text{where} \quad \text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)
$$
여기서 $W_i^Q, W_i^K, W_i^V$는 $i$번째 헤드를 위한 선형 변환 가중치 행렬이며, $W^O$는 최종 출력을 위한 가중치 행렬이다.

이러한 MHA 구조는 '관점의 다각화'를 통해 정보 추출을 심화시키는 과정으로 이해할 수 있다. 단일 Scaled Dot-Product Attention이 한 명의 전문가가 문서를 요약하는 것이라면, MHA는 $h$명의 각기 다른 전문 지식(각 헤드의 가중치 세트)을 가진 전문가가 동시에 문서를 분석하는 것과 같다.48 어떤 전문가는 문법 구조에, 다른 전문가는 의미적 유사성에, 또 다른 전문가는 단어 간 거리에 집중할 수 있다. 이들의 다양한 분석 결과($head_i$)를 취합(`Concat`)하고 최종적으로 종합 보고서($W^O$를 통한 변환)를 만드는 것이다. 이 '관점의 다각화'는 모델이 단편적인 정보에 매몰되지 않고, 입력 데이터에 대한 훨씬 더 풍부하고 강건한(robust) 표현을 학습하도록 하여, 단일 어텐션으로는 달성하기 어려운 깊이 있는 문맥 이해를 가능하게 한다.


Scaled Dot-Product Attention은 트랜스포머의 성공을 이끈 핵심 요소지만, 어텐션 메커니즘의 유일한 형태는 아니다. 특히, 트랜스포머 이전에 널리 사용되었던 Additive Attention과의 비교를 통해 그 특징과 장점을 더욱 명확히 이해할 수 있다.


두 메커니즘의 가장 근본적인 차이는 쿼리와 키 간의 유사도(어텐션 스코어)를 계산하는 방식에 있다.


- **Additive Attention:** Bahdanau 등이 제안한 이 방식은 쿼리($q$)와 키($k$)를 결합한 후, 단일 은닉층을 가진 피드포워드 신경망(Feed-Forward Network, FFN)을 통과시켜 스칼라 값의 어텐션 스코어를 계산한다.13 그 수식은 다음과 같다.
  $$
  score(q, k) = v_a^T \tanh(W_1 q + W_2 k)
  $$
  여기서 $W_1$, $W_2$, $v_a$는 모두 학습 가능한 가중치 파라미터다. 이름에서 알 수 있듯이, 쿼리와 키의 선형 변환 결과를 **덧셈(addition)**으로 결합하는 것이 특징이다.

- **Scaled Dot-Product Attention:** 이 방식은 별도의 신경망 없이 쿼리와 키의 **내적(dot-product)**을 직접 사용하여 유사도를 계산한다.13 이는 두 벡터 간의 기하학적 관계를 직접적으로 활용하는 방식이다.


이러한 계산 방식의 차이는 효율성에 직접적인 영향을 미친다.

- Additive Attention은 여러 번의 행렬 곱셈($W_1 q$, $W_2 k$, $v_a^T \cdot$)과 비선형 활성화 함수(`tanh`) 계산이 필요하여 상대적으로 계산 비용이 높고 복잡하다.28
- Scaled Dot-Product Attention은 고도로 최적화된 단일 행렬 곱셈 연산($QK^T$)으로 구현될 수 있어, 실제 연산에서 훨씬 빠르고 공간 효율적이다.20 특히 GPU 환경에서는 행렬 곱셈 연산이 극도로 가속화되므로 그 격차는 더욱 커진다.


"Attention Is All You Need" 논문에 따르면, 키 벡터의 차원 $d_k$가 작은 경우에는 두 메커니즘이 유사한 성능을 보였다. 그러나 $d_k$가 커질수록, 스케일링이 없는 단순 Dot-Product Attention은 내적 값의 분산이 커지는 문제로 인해 Additive Attention보다 성능이 저하되는 경향이 나타났다. Scaled Dot-Product Attention은 $\sqrt{d_k}$ 스케일링을 도입함으로써 이 문제를 효과적으로 해결하여, 큰 차원에서도 안정적인 성능을 유지할 수 있게 되었다.26

두 메커니즘의 핵심적인 차이점은 아래 표를 통해 명확하게 대비할 수 있다. 이 표는 왜 트랜스포머가 Additive Attention 대신 Scaled Dot-Product Attention을 채택했는지, 즉 계산 효율성과 확장성에 대한 설계 철학을 한눈에 보여준다.

| 특징                     | Additive Attention (Bahdanau-style)           | Scaled Dot-Product Attention                                 |
| ------------------------ | --------------------------------------------- | ------------------------------------------------------------ |
| 유사도 계산 함수         | $v_a^T \tanh(W_1 Q + W_2 K)$                  | $\frac{QK^T}{\sqrt{d_k}}$                                    |
| 핵심 연산                | 덧셈, 하이퍼볼릭 탄젠트, 행렬 곱              | 고도로 최적화된 행렬 곱                                      |
| 스코어 함수 내 파라미터  | 학습 가능한 가중치 행렬($W_1, W_2, v_a$) 존재 | 학습 가능한 파라미터 없음 (Q,K,V 생성 제외)                  |
| 계산 효율성              | 상대적으로 느리고 복잡함                      | 매우 빠르고 공간 효율적 28                                   |
| $d_k$ 차원에 대한 민감도 | 상대적으로 덜 민감함                          | 스케일링 없이는 $d_k$가 클 때 성능 저하, 스케일링으로 극복 26 |


Scaled Dot-Product Attention은 현대 딥러닝 아키텍처의 표준으로 자리 잡았지만, 근본적인 한계 또한 명확하다. 이 한계를 극복하기 위한 연구는 현재 가장 활발한 분야 중 하나다.


Scaled Dot-Product Attention의 가장 큰 약점은 시퀀스 길이 $n$에 대해 제곱에 비례하는 복잡도를 갖는다는 점이다. 구체적으로, $O(n^2 \cdot d)$의 계산 복잡도와 $O(n^2)$의 메모리 복잡도를 가진다.53 이는 연산 과정에서 ($n, n$) 크기의 거대한 어텐션 스코어 행렬을 명시적으로 계산하고 메모리에 저장해야 하기 때문이다.

이러한 제곱 복잡도는 모델이 처리할 수 있는 시퀀스 길이를 심각하게 제한한다. 수천 개의 토큰으로 구성된 장문서, 고해상도 이미지, 긴 오디오 시퀀스 등을 처리하는 데 있어 이 문제는 심각한 병목으로 작용한다.53


이 문제를 해결하기 위해 어텐션 메커니즘의 계산량을 줄이려는 다양한 연구가 진행되어 왔다. 이 연구들은 크게 어텐션을 근사(approximate)하는 방향으로 나아간다.

- **저계수 근사(Low-Rank Approximation):** 어텐션 행렬이 실제로는 낮은 내재적 계수(intrinsic rank)를 가질 것이라는 가정하에, Nyström Method와 같은 기법을 사용하여 전체 어텐션 행렬을 더 작은 행렬들의 곱으로 근사하는 방식이다.53
- **희소 어텐션(Sparse Attention):** 모든 토큰 쌍 간의 관계를 계산하는 대신, 일부 '중요한' 토큰 쌍에 대해서만 어텐션을 계산하는 접근법이다. 이는 지역적(local) 어텐션과 전역적(global) 어텐션을 결합하는 형태로 구현되곤 한다 (예: Longformer, BigBird).
- **선형화 어텐션(Linearized Attention):** 커널(Kernel) 트릭 등을 사용하여 $softmax(QK^T)V$의 계산 순서를 변경함으로써, $n \times n$ 행렬을 생성하지 않고도 유사한 결과를 얻는 방식이다. 이를 통해 복잡도를 $O(n)$ 수준으로 낮출 수 있다 (예: Linformer, Performer).6


알고리즘적 개선 외에도, 하드웨어의 특성을 최대한 활용하여 기존 어텐션 연산을 가속화하려는 공학적 접근법 또한 큰 성공을 거두었다.

- **퓨즈드 커널(Fused Kernels) - FlashAttention:** 기존의 어텐션 계산은 행렬 곱셈, 스케일링, Softmax 등 여러 단계를 거치며 GPU의 고대역폭 메모리(HBM)와 고속 온칩 SRAM 간의 데이터 이동을 반복적으로 발생시킨다. FlashAttention은 이 모든 과정을 단일 GPU 커널(fused kernel) 내에서 수행하도록 설계되었다.54 이를 통해 메모리 I/O 병목을 획기적으로 줄여, 알고리즘 변경 없이도 계산 속도를 크게 향상시키고 메모리 사용량을 감소시킨다. FlashAttention-2는 병렬 처리와 작업 분할을 더욱 최적화하여 성능을 한층 더 끌어올렸다.55
- **프레임워크 내장 최적화:** 이러한 발전은 PyTorch와 같은 주요 딥러닝 프레임워크에도 빠르게 통합되고 있다. PyTorch 2.0부터는 `torch.nn.functional.scaled_dot_product_attention`이라는 내장 함수를 제공한다. 이 함수는 입력 텐서의 조건(GPU 종류, 차원, 데이터 타입 등)을 자동으로 분석하여 FlashAttention, Memory-Efficient Attention, 표준 C++ 구현 중 가장 최적의 백엔드를 동적으로 선택하여 사용한다.54

이러한 연구 동향은 어텐션 메커니즘의 발전 방향이 변화하고 있음을 시사한다. 초기 어텐션 연구가 RNN의 한계를 극복하고 번역 '정확도'를 높이는 데 초점을 맞췄다면, 트랜스포머가 표준으로 자리 잡은 지금, 연구의 최전선은 $O(n^2)$라는 근본적인 '계산 효율성'의 한계를 극복하는 방향으로 이동했다. FlashAttention과 같은 공학적 접근과 Linformer와 같은 알고리즘적 접근은 모두 이 목표를 향하고 있다. 이는 어텐션 메커니즘이 성숙기에 접어들면서, 이제는 더 긴 컨텍스트를 더 빠르고 저렴하게 처리하는 능력이 핵심 경쟁력이 되었음을 보여준다.


Scaled Dot-Product Attention은 현대 딥러닝, 특히 자연어 처리 분야의 지형을 근본적으로 바꾼 혁신적인 기술이다. 그 영향력은 특정 모델의 성능 개선을 넘어, 인공지능 연구와 개발의 패러다임 자체를 전환시켰다.


Scaled Dot-Product Attention의 가장 큰 기여는 RNN과 CNN이 가진 순차적 처리의 제약에서 벗어나, 시퀀스 데이터에 대한 완전한 병렬 처리를 가능하게 했다는 점이다.2 이는 대규모 데이터셋과 컴퓨팅 자원을 활용하여 이전과는 비교할 수 없는 크기의 모델을 학습시킬 수 있는 길을 열었다. 결과적으로, 이는 GPT, BERT와 같은 거대 언어 모델(LLM)의 등장을 가능하게 한 핵심적인 기술적 토대가 되었다.5

이 메커니즘의 가치는 다음과 같이 요약할 수 있다.

- **단순성과 강력함:** 내적, 스케일링, 소프트맥스라는 단순하고 강력한 수학적 원리에 기반하여 높은 성능을 달성했다.
- **확장성:** Multi-Head Attention으로의 자연스러운 확장을 통해, 모델이 다양한 관점에서 정보를 포착하고 표현력을 극대화할 수 있도록 했다.
- **효율성:** 하드웨어 친화적인 행렬 연산 기반 구조는 대규모 모델의 학습과 추론을 현실적인 시간 내에 가능하게 만들었다.


Scaled Dot-Product Attention은 현재의 성공에도 불구하고 여전히 도전 과제를 안고 있으며, 이는 미래 연구의 방향을 제시한다.

- **복잡도 문제 해결:** $O(n^2)$의 계산 및 메모리 복잡도는 여전히 가장 큰 기술적 허들이다. 이를 해결하기 위한 효율적인 어텐션 변형(Efficient Transformers)에 대한 연구는 앞으로도 계속해서 핵심적인 분야로 남을 것이다.
- **도메인 확장:** 어텐션의 원리는 NLP를 넘어 컴퓨터 비전(Vision Transformers), 음성 처리, 신약 개발, 유전자 서열 분석 등 다양한 분야로 성공적으로 확장되고 있다.53 각 도메인의 특성에 맞는 어텐션 메커니즘의 개발과 적용은 그 영향력을 더욱 증대시킬 것이다.
- **차세대 아키텍처:** 미래의 아키텍처는 표준 Scaled Dot-Product Attention의 강력한 표현력과 새로운 효율적인 메커니즘들의 장점을 결합하는 하이브리드 형태가 될 가능성이 높다.

결론적으로, Scaled Dot-Product Attention은 단순한 기술적 발전을 넘어 AI 모델이 정보를 처리하고 관계를 학습하는 방식에 대한 근본적인 통찰을 제공했다. 그 핵심 철학인 '관련성에 기반한 동적 가중치 할당'은 앞으로도 인공지능 모델 설계의 가장 중요한 원리 중 하나로 자리매김할 것이며, 끊임없이 진화하며 AI 기술의 새로운 지평을 열어갈 것이다.


1. 어텐션 메커니즘이란 무엇인가요? - IBM, 8월 20, 2025에 액세스, https://www.ibm.com/kr-ko/think/topics/attention-mechanism
2. “Attention is All You Need” Summary - Medium, 8월 20, 2025에 액세스, https://medium.com/@dminhk/attention-is-all-you-need-summary-6f0437e63a91
3. Attention is All you Need - NIPS Paper, 8월 20, 2025에 액세스, https://papers.neurips.cc/paper/7181-attention-is-all-you-need.pdf
4. [딥러닝] 어텐션 메커니즘(Attention Mechanism)이란? - Logical Scribbles - 티스토리, 8월 20, 2025에 액세스, https://stydy-sturdy.tistory.com/39
5. Attention Is All You Need - Wikipedia, 8월 20, 2025에 액세스, https://en.wikipedia.org/wiki/Attention_Is_All_You_Need
6. Attention (machine learning) - Wikipedia, 8월 20, 2025에 액세스, https://en.wikipedia.org/wiki/Attention_(machine_learning)
7. engoo.co.kr, 8월 20, 2025에 액세스, [https://engoo.co.kr/app/words/word/attention/zga0ELstQmCjlQAAAABiZA#:~:text=attention%20(%E3%80%90%EB%AA%85%EC%82%AC%E3%80%91%EA%B4%80%EC%8B%AC%2C,%EC%9A%A9%EB%B2%95%2C%20%EA%B7%B8%EB%A6%AC%EA%B3%A0%20%EC%98%88%EB%AC%B8%20%7C%20Engoo%20Words](https://engoo.co.kr/app/words/word/attention/zga0ELstQmCjlQAAAABiZA#:~:text=attention (【명사】관심%2C,용법%2C 그리고 예문 | Engoo Words)
8. attention (【명사】관심, 주의, 주의력 ) 뜻, 용법, 그리고 예문 | Engoo Words, 8월 20, 2025에 액세스, https://engoo.co.kr/app/words/word/attention/zga0ELstQmCjlQAAAABiZA
9. 어텐션 - 위키백과, 우리 모두의 백과사전, 8월 20, 2025에 액세스, [https://ko.wikipedia.org/wiki/%EC%96%B4%ED%85%90%EC%85%98](https://ko.wikipedia.org/wiki/어텐션)
10. attention - 주의, 집중, :: 토익 단어장, 8월 20, 2025에 액세스, https://www.dokjongban.com/voca/attention/
11. "Attention"의 정의 및 의미 | 그림 사전, 8월 20, 2025에 액세스, https://dictionary.langeek.co/en-KO/word/12507?entry=attention
12. [ Computer Vision ] Attention, Transformer 이해하기 - KalelPark's LAB - 티스토리, 8월 20, 2025에 액세스, https://kalelpark.tistory.com/56
13. 어텐션 메커니즘 (Attention Mechanism)이란? - DEVELOPER - 티스토리, 8월 20, 2025에 액세스, https://ctkim.tistory.com/entry/Attention-Mechanism
14. 어텐션 메커니즘(Attention Mechanism) - MINISTOP - 티스토리, 8월 20, 2025에 액세스, https://kxngmxn.tistory.com/122
15. Attention is All you Need - NIPS Paper, 8월 20, 2025에 액세스, https://papers.nips.cc/paper/7181-attention-is-all-you-need
16. (PDF) Attention is All you Need (2017) | Ashish Vaswani | 88778 Citations - SciSpace, 8월 20, 2025에 액세스, https://scispace.com/papers/attention-is-all-you-need-1hodz0wcqb
17. [NLP 스터디] Scaled Dot-Product Attention - 아는 것의 미학 - 티스토리, 8월 20, 2025에 액세스, https://applepy.tistory.com/123
18. How to Implement Scaled Dot-Product Attention from Scratch in TensorFlow and Keras, 8월 20, 2025에 액세스, https://machinelearningmastery.com/how-to-implement-scaled-dot-product-attention-from-scratch-in-tensorflow-and-keras/
19. Understanding Scaled Dot-Product Attention in Transformer Models - Medium, 8월 20, 2025에 액세스, https://medium.com/@saraswatp/understanding-scaled-dot-product-attention-in-transformer-models-5fe02b0f150c
20. Evolution of Attention Mechanisms in NLP: From Additive to Self-Attention - Medium, 8월 20, 2025에 액세스, https://medium.com/@akanyaani/evolution-of-attention-mechanisms-in-nlp-from-additive-to-self-attention-01e265925899
21. Tutorial 6: Transformers and Multi-Head Attention — UvA DL Notebooks v1.2 documentation, 8월 20, 2025에 액세스, https://uvadlc-notebooks.readthedocs.io/en/latest/tutorial_notebooks/tutorial6/Transformers_and_MHAttention.html
22. Dot Product | lostineconomics.com, 8월 20, 2025에 액세스, http://www.lostineconomics.com/lostineconomics-v2-1/math/2019/07/18/dot-product.html
23. ctkim.tistory.com, 8월 20, 2025에 액세스, [https://ctkim.tistory.com/entry/Attention-Mechanism#:~:text=Scaled%20Dor%2DProduct%20Attention%EC%9D%80,%EC%9D%98%20%EC%95%88%EC%A0%95%EC%84%B1%EC%9D%84%20%ED%96%A5%EC%83%81%EC%8B%9C%ED%82%B5%EB%8B%88%EB%8B%A4.](https://ctkim.tistory.com/entry/Attention-Mechanism#:~:text=Scaled Dor-Product Attention은,의 안정성을 향상시킵니다.)
24. Scaled Dot-Product Attention - Transformer(1) - simpling - 티스토리, 8월 20, 2025에 액세스, https://simpling.tistory.com/3
25. What is the intuition behind the dot product attention? - Educative.io, 8월 20, 2025에 액세스, https://www.educative.io/answers/what-is-the-intuition-behind-the-dot-product-attention
26. 차근차근 이해하는 Transformer(1): Scaled Dot-Product Attention - Data Science - 티스토리, 8월 20, 2025에 액세스, [https://tigris-data-science.tistory.com/entry/%EC%B0%A8%EA%B7%BC%EC%B0%A8%EA%B7%BC-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-Transformer1-Scaled-Dot-Product-Attention](https://tigris-data-science.tistory.com/entry/차근차근-이해하는-Transformer1-Scaled-Dot-Product-Attention)
27. 15.2. Multi-Head Attention - Hironobu SUZUKI @ InterDB, 8월 20, 2025에 액세스, https://www.interdb.jp/dl/part04/ch15/sec03.html
28. Why is dot product attention faster than additive attention? - AI Stack Exchange, 8월 20, 2025에 액세스, https://ai.stackexchange.com/questions/11866/why-is-dot-product-attention-faster-than-additive-attention
29. What is the difference between additive and scaled dot-product attention?, 8월 20, 2025에 액세스, [https://massedcompute.com/faq-answers/?question=What%20is%20the%20difference%20between%20additive%20and%20scaled%20dot-product%20attention?](https://massedcompute.com/faq-answers/?question=What+is+the+difference+between+additive+and+scaled+dot-product+attention?)
30. Transformer: Attention is All You Need 리뷰 - Huiwon, 8월 20, 2025에 액세스, https://vanche.github.io/NLP_Transformer/
31. What are the query, key, and value vectors? | by RAHULRAJ P V - Medium, 8월 20, 2025에 액세스, https://rahulrajpvr7d.medium.com/what-are-the-query-key-and-value-vectors-5656b8ca5fa0
32. Understanding Query, Key, Value in Transformers and LLMs | by Charles Chi | AI - Medium, 8월 20, 2025에 액세스, https://medium.com/ai-assimilating-intelligence/understanding-query-key-value-in-transformers-c579b93054cc
33. Understanding the Role of Query, Key, and Value Matrices in Transformer Models, 8월 20, 2025에 액세스, https://transformers-goto-guide.hashnode.dev/understanding-the-role-of-query-key-and-value-in-transformer-models
34. A Survey of Transformers (2021) 논문 스터디 - (1), 8월 20, 2025에 액세스, [https://velog.io/@choonsik_mom/A-Survey-of-Transformers-2021-%EB%85%BC%EB%AC%B8-%EC%8A%A4%ED%84%B0%EB%94%94-1](https://velog.io/@choonsik_mom/A-Survey-of-Transformers-2021-논문-스터디-1)
35. [AI/LLM] Transformer Attention 이해하기: Q, K, V의 역할과 동작 원리, 8월 20, 2025에 액세스, https://mvje.tistory.com/258
36. What is Query, Key, and Value (QKV) in the Transformer Architecture and Why Are They Used? | Ebrahim Pichka, 8월 20, 2025에 액세스, https://epichka.com/blog/2023/qkv-transformer/
37. LLM Transformer Model Visually Explained - Polo Club of Data Science, 8월 20, 2025에 액세스, https://poloclub.github.io/transformer-explainer/
38. 트랜스포머(Transformer) 파헤치기—2. Multi-Head Attention, 8월 20, 2025에 액세스, https://www.blossominkyung.com/deeplearning/transformer-mha
39. 4-1. Transformer(Self Attention) [초등학생도 이해하는 자연어처리] - 코딩 오페라, 8월 20, 2025에 액세스, https://codingopera.tistory.com/43
40. Understanding Scaling in Scaled Dot-Product Attention: Simple Math Insights - Medium, 8월 20, 2025에 액세스, https://medium.com/@jennifer.zzz/why-scaling-is-important-in-scaled-dot-product-attention-dca8d8cb9504
41. Purpose of sqrt(dim(k)) in Scaled dot product attention - DeepLearning.AI, 8월 20, 2025에 액세스, https://community.deeplearning.ai/t/purpose-of-sqrt-dim-k-in-scaled-dot-product-attention/62880
42. Why does this multiplication of $Q$ and $K$ have a variance of $d_k$, in scaled dot product attention? - AI Stack Exchange, 8월 20, 2025에 액세스, https://ai.stackexchange.com/questions/21237/why-does-this-multiplication-of-q-and-k-have-a-variance-of-d-k-in-scaled
43. Transformer is All You Need - noviceforever - GitBook, 8월 20, 2025에 액세스, https://housekdk.gitbook.io/ml/ml/nlp/transformer
44. [Model] About Self-Attention - MJ's 우당탕탕 성장기 - 티스토리, 8월 20, 2025에 액세스, https://mj-thump-thump-story.tistory.com/entry/Model-About-Self-Attention
45. Transformer의 Multi-Head Attention과 Transformer에서 쓰인 다양한 기법 - 지그시, 8월 20, 2025에 액세스, [https://glanceyes.com/entry/Transformer%EC%9D%98-Multi-Head-Attention%EA%B3%BC-Transformer%EC%97%90%EC%84%9C-%EC%93%B0%EC%9D%B8-%EB%8B%A4%EC%96%91%ED%95%9C-%EA%B8%B0%EB%B2%95](https://glanceyes.com/entry/Transformer의-Multi-Head-Attention과-Transformer에서-쓰인-다양한-기법)
46. 1. Attention Is All You Need - 공부하려고 만든 블로그 - 티스토리, 8월 20, 2025에 액세스, https://welcome-to-dewy-world.tistory.com/108
47. Understanding Multi-Head Attention for ML Framework Developers, 8월 20, 2025에 액세스, https://dev-discuss.pytorch.org/t/understanding-multi-head-attention-for-ml-framework-developers/1792
48. Transformer의 Multi-head Attention 파헤치기 - 꾸준한 성장일기 - 티스토리, 8월 20, 2025에 액세스, https://seungseop.tistory.com/29
49. D2L - 11.5. Multi-Head Attention - IT 기술 따라잡기 - 티스토리, 8월 20, 2025에 액세스, https://coronasdk.tistory.com/1392
50. 4-2. Transformer(Multi-head Attention) [초등학생도 이해하는 자연어처리] - 코딩 오페라, 8월 20, 2025에 액세스, https://codingopera.tistory.com/44
51. The Multi-Headed Attention — Yet Another (JAX) Transformer - Attanasio Giuseppe, 8월 20, 2025에 액세스, https://g8a9.github.io/yet_another_jax_transformer/docs/02_core_components/multi_headed_attention.html
52. Why do different attention mechanisms use the "value" vector differently? : r/learnmachinelearning - Reddit, 8월 20, 2025에 액세스, https://www.reddit.com/r/learnmachinelearning/comments/xg1t68/why_do_different_attention_mechanisms_use_the/
53. [논문 리뷰] Continual Low-Rank Scaled Dot-product Attention - Moonlight, 8월 20, 2025에 액세스, https://www.themoonlight.io/ko/review/continual-low-rank-scaled-dot-product-attention
54. (Beta) Scaled Dot Product Attention (SDPA)로 고성능 트랜스포머(Transformers) 구현하기 - PyTorch 한국어 튜토리얼 - 파이토치 한국 사용자 모임, 8월 20, 2025에 액세스, https://tutorials.pytorch.kr/intermediate/scaled_dot_product_attention_tutorial.html
55. torch.nn.functional.scaled_dot_product_attention — PyTorch 2.7 documentation, 8월 20, 2025에 액세스, https://pytorch.org/docs/stable/generated/torch.nn.functional.scaled_dot_product_attention.html
56. torch.nn.functional.scaled_dot_product_attention — PyTorch 2.8 documentation, 8월 20, 2025에 액세스, https://docs.pytorch.org/docs/stable/generated/torch.nn.functional.scaled_dot_product_attention.html
57. Attention Is All You Need | Request PDF - ResearchGate, 8월 20, 2025에 액세스, https://www.researchgate.net/publication/317558625_Attention_Is_All_You_Need


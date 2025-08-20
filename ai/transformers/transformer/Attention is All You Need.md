# Attention is All You Need

## 1.  순환(Recurrence)의 종말과 새로운 패러다임의 서막

자연어 처리(NLP)를 비롯한 순차적 데이터(Sequential Data) 모델링 분야는 오랫동안 순환 신경망(Recurrent Neural Network, RNN)과 그 변형 모델들이 지배해왔다.1 RNN은 이전 시간 단계(timestep)의 정보를 은닉 상태(hidden state)에 저장하고 이를 현재 시간 단계의 입력과 함께 처리하는 순환 구조를 통해 시퀀스의 동적인 특성을 포착하도록 설계되었다.2 이러한 구조를 발전시킨 LSTM(Long Short-Term Memory)과 GRU(Gated Recurrent Unit)는 내부에 '게이트'라는 정교한 메커니즘을 도입하여 정보의 흐름을 제어함으로써 RNN의 고질적인 문제를 일부 해결하고자 했다.1

그러나 이러한 순환 기반 모델들은 두 가지 본질적인 한계에 직면했다.

첫째는 '장기 의존성 문제(Long-Range Dependency Problem)'이다. 시퀀스의 길이가 길어질수록, 역전파 과정에서 기울기가 점차 사라지는 '기울기 소실(Vanishing Gradient)' 현상으로 인해 시퀀스 초반부의 정보가 후반부까지 온전히 전달되기 어려웠다.1 이는 문장의 시작 부분에 있는 핵심 단어가 문장 끝의 단어 의미를 결정하는 기계 번역과 같은 태스크에서 치명적인 약점으로 작용했다.1 LSTM과 GRU가 이 문제를 완화했지만, 근본적인 해결책이 되지는 못했다.1

둘째는 '순차적 계산으로 인한 병렬 처리의 부재'이다. RNN 계열 모델은 설계상 $t$ 시점의 계산이 $t-1$ 시점의 계산 결과(은닉 상태)에 의존하기 때문에, 본질적으로 순차적으로 처리될 수밖에 없다.2 이러한 특성은 대규모 병렬 연산에 최적화된 GPU와 같은 현대 하드웨어의 잠재력을 온전히 활용하지 못하게 만드는 심각한 병목 현상을 초래했다.1 알고리즘의 순차적 제약이 하드웨어의 병렬적 발전을 따라가지 못하는 불일치가 발생한 것이다.

이러한 배경 속에서 2017년, Ashish Vaswani 등을 포함한 Google 연구진은 "Attention is All You Need"라는 도발적인 제목의 논문을 통해 기존의 패러다임을 완전히 뒤엎는 제안을 내놓았다.8 이들의 핵심 주장은 순환이나 합성곱 연산 없이, 오직 '어텐션(Attention)' 메커니즘만으로 시퀀스 내의 모든 단어 간 의존성을 학습할 수 있다는 것이었다.10 이 새로운 아키텍처, '트랜스포머(Transformer)'는 시퀀스 내 두 토큰 간의 경로 길이를 

$O(1)$로 단축하여 장기 의존성 문제를 정면으로 돌파하고, 모든 계산을 행렬 연산으로 구성하여 완벽한 병렬 처리를 가능하게 함으로써 훈련 속도를 획기적으로 개선했다.12 이는 단순히 더 나은 모델을 제시한 것을 넘어, 알고리즘 혁신을 통해 하드웨어의 발전을 온전히 수용한, 시의적절한 패러다임의 전환이었다.

| 특징 (Feature)                             | RNN / LSTM                                                   | 트랜스포머 (Transformer)                                     |
| ------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 핵심 메커니즘 (Core Mechanism)             | 순환 (Recurrence)                                            | 자기-어텐션 (Self-Attention)                                 |
| 장기 의존성 처리 (Long-Range Dependencies) | 기울기 소실 문제로 어려움 (Struggles due to vanishing gradients) | 직접적인 연결로 효과적 처리 (Effectively handled via direct connections) |
| 병렬 처리 (Parallelization)                | 순차적 계산으로 불가능 (Not possible due to sequential nature) | 높은 병렬성 (Highly parallelizable)                          |
| 계산 복잡도 (Complexity per Layer)         | $O(n \cdot d^2)$                                             | $O(n^2 \cdot d)$                                             |
| 최대 경로 길이 (Maximum Path Length)       | $O(n)$                                                       | $O(1)$                                                       |

## 2.  트랜스포머: 어텐션 기반 아키텍처의 전체 구조

트랜스포머는 기계 번역과 같은 시퀀스-투-시퀀스(Sequence-to-Sequence) 태스크를 해결하기 위해 기존의 성공적인 인코더-디코더(Encoder-Decoder) 프레임워크를 계승한다.8 인코더는 입력 시퀀스(예: "Wie geht es Ihnen?")를 받아 일련의 연속적인 벡터 표현(contextual representation)으로 변환하는 역할을 한다. 디코더는 인코더가 생성한 이 표현과 이전에 생성된 출력 토큰들을 참조하여 다음 출력 토큰(예: "are")을 생성하고, 이 과정을 문장이 끝날 때까지 반복한다.7

논문에서 제안된 모델은 인코더와 디코더를 각각 6개의 동일한 구조를 가진 레이어(`N=6`)를 쌓아 구성한다.10 각 레이어는 구조적으로는 동일하지만, 훈련 과정에서 학습되는 가중치(weights)는 공유하지 않는다.13 이처럼 레이어를 깊게 쌓는 구조는 입력 데이터로부터 점차 더 복잡하고 추상적인 특징을 학습할 수 있게 한다.

이 아키텍처를 토큰들의 '커뮤니케이션 네트워크'로 비유하면 그 작동 방식을 더 직관적으로 이해할 수 있다. 인코더는 모든 입력 토큰이 동시에 참여하는 '전체 회의'와 같다. 각 토큰은 자신의 정보를 모든 다른 토큰에게 전달하고, 동시에 다른 모든 토큰의 정보를 수신하여 문맥 전체에 대한 공유된 이해를 구축한다. 반면, 디코더는 '컨퍼런스 콜'에 비유할 수 있다. 출력 토큰들은 순서대로 발언하며("지금까지 우리가 무슨 말을 했지?" - Masked Self-Attention), 각 발언 순서마다 인코더의 '회의록'을 참조하여("입력 문장의 맥락을 고려할 때, 다음에 무슨 말을 해야 할까?" - Encoder-Decoder Attention) 가장 적절한 다음 단어를 결정한다. 이처럼 트랜스포머는 단순히 벡터를 변환하는 기계적 과정을 넘어, 정보를 병렬적으로 라우팅하고 맥락화하는 고도로 효율적인 정보 교환 시스템으로 작동한다.

전체적인 데이터 처리 흐름은 다음과 같다.

1. **입력 처리**: 입력 문장의 각 토큰은 고유한 임베딩 벡터로 변환된다.15 순환 구조가 없기 때문에 단어의 순서 정보를 제공하기 위해, 각 임베딩 벡터에 '위치 인코딩(Positional Encoding)' 벡터가 더해진다.11
2. **인코더 스택 통과**: 위치 정보가 추가된 임베딩 벡터 시퀀스는 인코더 스택의 첫 번째 레이어로 입력된다. 각 인코더 레이어는 셀프 어텐션(Self-Attention)과 피드포워드 신경망(Feed-Forward Network)이라는 두 개의 하위 레이어를 통해 입력 시퀀스를 처리하고, 그 결과를 다음 레이어로 전달한다.10 6개의 레이어를 모두 통과한 최종 출력은 입력 시퀀스 전체의 풍부한 문맥 정보가 인코딩된 벡터 시퀀스가 된다.
3. **디코더 스택의 자기회귀적 생성**: 디코더는 인코더의 최종 출력과, 문장의 시작을 알리는 `<sos>` 토큰부터 시작하여 이전에 생성된 출력 단어들을 입력으로 받는다. 각 디코더 레이어는 세 개의 하위 레이어(Masked Self-Attention, Encoder-Decoder Attention, Feed-Forward Network)를 통해 다음 단어에 대한 확률 분포를 계산한다.10 가장 확률이 높은 단어가 선택되고, 이 과정은 문장의 끝을 의미하는 `<eos>` 토큰이 생성될 때까지 자기회귀적(auto-regressive)으로 반복된다.

## 3.  트랜스포머의 심장: 핵심 메커니즘 상세 분석

트랜스포머의 혁신은 순환 구조를 대체한 몇 가지 핵심 메커니즘의 유기적인 결합에 있다. 이 메커니즘들은 각각 고유한 역할을 수행하며, 함께 작동하여 모델의 강력한 성능을 이끌어낸다.

### 3.1  Scaled Dot-Product Attention

트랜스포머의 가장 근본적인 연산 단위는 Scaled Dot-Product Attention이다. 이 메커니즘은 '쿼리(Query)', '키(Key)', '밸류(Value)'라는 세 가지 벡터를 기반으로 작동한다.12 이는 정보 검색 시스템에 비유할 수 있다. 사용자가 검색어(Query)를 입력하면, 시스템은 이 검색어와 데이터베이스에 색인된 키(Key)들의 관련성을 비교하고, 가장 관련성이 높은 키에 연결된 실제 정보(Value)를 반환한다.14 트랜스포머에서는 입력 임베딩 벡터에 각각 다른 가중치 행렬($W^Q, W^K, W^V$)을 곱하여 이 세 벡터를 생성한다.11

Scaled Dot-Product Attention의 계산 과정은 다음의 수식으로 정의된다.19
$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

1. **유사도 계산 ($QK^T$):** 모든 쿼리 벡터와 모든 키 벡터 간의 내적(dot product)을 계산한다. 이 결과로 생성된 행렬의 각 원소는 특정 쿼리와 특정 키 사이의 유사도, 즉 '어텐션 스코어(attention score)'를 나타낸다.11
2. **스케일링 ($/\sqrt{d_k}$):** 계산된 스코어를 키 벡터의 차원 $d_k$의 제곱근으로 나누어준다. $d_k$가 클 경우 내적 값이 지나치게 커져 softmax 함수의 기울기가 0에 가까워지는 문제를 야기할 수 있다. 이 스케일링 과정은 이러한 현상을 방지하여 학습 과정을 안정화하는 매우 중요한 역할을 한다.12
3. **가중치 정규화 (`softmax`):** 스케일링된 스코어에 softmax 함수를 적용하여 모든 값의 합이 1이 되는 확률 분포, 즉 '어텐션 가중치(attention weights)'로 변환한다. 이 가중치는 각 밸류 벡터에 얼마나 '집중'해야 할지를 나타내는 비율이 된다.11
4. **가중합 계산 ($\dots V$):** 마지막으로, 계산된 어텐션 가중치를 각 밸류 벡터에 곱한 후 모두 더하여 최종 출력 벡터를 생성한다. 이 과정을 통해 현재 쿼리와 관련성이 높은 단어의 정보(Value)는 강조되고, 관련성이 낮은 단어의 정보는 억제된다.13

### 3.2  Multi-Head Attention

하나의 어텐션만으로는 문장 내 단어들 간의 복잡하고 다층적인 관계를 모두 포착하기 어렵다. 예를 들어, 어떤 단어는 다른 단어와 구문적인 관계를, 또 다른 단어와는 의미적인 관계를 맺을 수 있다. Multi-Head Attention은 이러한 문제를 해결하기 위해 여러 개의 어텐션을 병렬적으로 수행하는 메커니즘이다.12

이는 마치 한 문장을 여러 명의 전문가가 각기 다른 관점에서 동시에 분석하는 것과 같다. 한 전문가는 주어-동사 관계를, 다른 전문가는 대명사 지칭 관계를, 또 다른 전문가는 인과 관계를 분석하는 식이다. Multi-Head Attention은 `h`개의 '헤드'가 각각 다른 '표현 부분 공간(representation subspace)'에서 독립적으로 어텐션을 수행함으로써, 모델이 다양한 종류의 관계를 동시에 학습할 수 있도록 한다.12

구조는 다음과 같다.

1. 입력 Q, K, V를 `h`개의 서로 다른 학습 가능한 선형 변환(projection)을 통해 `h`개의 작은 차원을 가진 Q, K, V 세트로 분할한다. 논문에서는 `h=8`을 사용했다.19
2. 각 헤드는 분할된 Q, K, V를 사용하여 Scaled Dot-Product Attention을 병렬적으로 수행한다.16
3. `h`개의 헤드에서 나온 출력 벡터들을 모두 이어 붙인다(concatenate).
4. 이 연결된 벡터를 다시 한번의 선형 변환을 통해 최종 출력 벡터로 만든다.12

수식으로는 다음과 같이 표현된다.
$$
\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_h)W^O
$$

$$
\text{where head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)
$$

이러한 구조 덕분에 "The animal didn't cross the street because **it** was too tired"라는 문장에서 'it'이 'animal'을 지칭하는 관계를 특정 헤드가 포착하고, 'cross'와 'street'의 관계를 다른 헤드가 포착하는 등, 모델의 표현력이 극대화된다.13

### 3.3  순서 정보의 주입: Positional Encoding

어텐션 메커니즘은 입력 시퀀스를 순서가 없는 단어들의 집합(set)으로 취급한다.12 이로 인해 "I Love You"와 "You Live I"를 동일한 문장으로 인식하는 근본적인 문제가 발생한다. 순환 구조 없이 단어의 순서 정보를 모델에 제공하기 위해, 트랜스포머는 '위치 인코딩(Positional Encoding)'이라는 독창적인 방법을 사용한다.14 이는 각 단어의 임베딩 벡터에 해당 단어의 위치 정보를 담은 벡터를 더해주는 방식이다.23

이 기법은 단순히 위치 정보를 주입하는 것을 넘어, 토큰의 '의미적 정체성'(임베딩)과 '구조적 위치'(인코딩)를 분리하는 중요한 역할을 한다. RNN에서는 단어의 의미와 위치 정보가 은닉 상태에 뒤섞여 분리할 수 없었지만, 트랜스포머에서는 `최종 입력 벡터 = 의미 임베딩 + 위치 인코딩`의 형태로 두 정보가 독립적으로 제공된다. 이 덕분에 셀프 어텐션 메커니즘은 의미적 유사성과 위치적 근접성을 별개의 신호로 학습하여, 상황에 따라 더 유연하게 관계를 모델링할 수 있다.

논문에서는 주기가 다른 사인(sine)과 코사인(cosine) 함수를 이용해 위치 인코딩 벡터를 생성한다.11
$$
PE_{(pos, 2i)} = \sin(pos / 10000^{2i/d_{\text{model}}})
$$

$$
PE_{(pos, 2i+1)} = \cos(pos / 10000^{2i/d_{\text{model}}})
$$

여기서 `pos`는 시퀀스 내 단어의 위치, `i`는 임베딩 벡터 내의 차원 인덱스, $d_{\text{model}}$은 임베딩 차원의 크기이다. 이 방식은 각 위치마다 고유한 인코딩을 생성할 뿐만 아니라, 삼각함수의 주기적 특성 덕분에 모델이 상대적인 위치 관계를 쉽게 학습할 수 있게 한다.24 또한, 훈련 시 보았던 문장보다 더 긴 문장이 입력으로 들어와도 위치 정보를 일반화하여 적용할 수 있는 장점이 있다.12

### 3.4  Position-wise Feed-Forward Networks (FFN)

어텐션 하위 레이어를 통과한 출력은 각 위치(position)마다 독립적으로 동일한 피드포워드 신경망(FFN)을 거친다. 이 FFN의 역할은 어텐션을 통해 종합된 문맥 정보를 비선형 공간으로 변환하여 모델의 표현력을 한층 더 강화하는 것이다.10

구조적으로는 2개의 선형 변환과 그 사이에 ReLU 활성화 함수가 있는 간단한 2층 다층 퍼셉트론(MLP)이다.7
$$
\text{FFN}(x) = \max(0, xW_1 + b_1)W_2 + b_2
$$
모든 위치의 벡터가 동일한 가중치($W_1, b_1, W_2, b_2$)를 공유하는 FFN을 통과하지만, 각 위치의 벡터는 독립적으로 처리된다. 이는 어텐션 레이어와 달리 위치 간 정보 교환이 일어나지 않음을 의미한다.

### 3.5  Add & Norm: 깊은 네트워크를 위한 안정화 장치

트랜스포머는 6개의 레이어를 쌓은 깊은 구조를 가지므로, 안정적인 학습을 위한 장치가 필수적이다. 이를 위해 각 하위 레이어(Multi-Head Attention, FFN)마다 잔차 연결(Residual Connection)과 계층 정규화(Layer Normalization)를 적용한다.10

- **잔차 연결 (Add):** 하위 레이어의 입력 `x`를 해당 레이어의 출력 `Sublayer(x)`에 그대로 더해준다. 즉, $x + \text{Sublayer}(x)$를 계산한다. 이는 네트워크가 깊어지더라도 정보(기울기)가 손실 없이 잘 전달되도록 하는 통로 역할을 하여, 기울기 소실 문제를 방지하고 더 깊은 모델의 학습을 가능하게 한다.10
- **계층 정규화 (Norm):** 잔차 연결 후의 결과를 계층 정규화한다. 이는 각 레이어의 출력 분포를 안정시켜 학습 과정을 원활하게 하고 수렴 속도를 높이는 효과가 있다.10

결과적으로 각 하위 레이어의 최종 출력은 $LayerNorm(x + Sublayer(x))$ 형태가 된다.10

## 4.  인코더와 디코더의 내부 작동 원리

트랜스포머의 인코더와 디코더는 앞서 설명한 핵심 메커니즘들을 조합하여 구성되지만, 각각의 목적에 맞게 약간 다른 구조를 가진다.

### 4.1  인코더 레이어: 입력 문장의 맥락적 표현 생성

각 인코더 레이어는 두 개의 주요 하위 레이어로 구성된다: Multi-Head Self-Attention과 Position-wise FFN.10

인코더의 셀프 어텐션에서는 쿼리, 키, 밸류가 모두 이전 인코더 레이어의 출력, 즉 동일한 소스에서 생성된다. 이는 입력 시퀀스가 '자기 자신'을 참조하여 내부 단어들 간의 관계를 파악하는 과정이다. 예를 들어, "그 동물은 너무 피곤해서 길을 건너지 않았다"라는 문장에서 '그것'이 '동물'을 가리키는 것과 같은 문맥적 관계를 이 단계에서 학습한다.10 입력 시퀀스의 모든 토큰은 다른 모든 토큰과 직접 상호작용하며, 레이어를 거치면서 점차 주변 문맥을 풍부하게 반영하는 벡터 표현으로 정제된다.

### 4.2  디코더 레이어: 맥락을 바탕으로 출력 문장 생성

디코더 레이어는 인코더보다 복잡한 구조로, 세 개의 하위 레이어로 구성된다: Masked Multi-Head Self-Attention, Encoder-Decoder Attention, 그리고 Position-wise FFN.10

1. **Masked Multi-Head Self-Attention:** 디코더는 번역문과 같이 출력 시퀀스를 한 단어씩 순차적으로 생성해야 한다. 따라서 특정 위치의 단어를 예측할 때, 그 뒤에 올 단어들을 미리 참조해서는 안 된다. 이를 '자기회귀(auto-regressive)' 속성이라고 한다. 이 속성을 보장하기 위해 디코더의 첫 번째 어텐션 레이어에서는 '마스킹(masking)' 기법이 사용된다. 어텐션 스코어를 계산한 후, 현재 예측하려는 위치보다 뒤에 있는 위치들의 스코어 값을 음의 무한대(`-∞`)로 만들어 softmax 함수를 통과했을 때 해당 위치의 가중치가 0이 되도록 한다.10 이는 정보가 미래에서 과거로 흐르는 것을 차단하는 역할을 한다.
2. **Encoder-Decoder Attention:** 이 부분이 디코더가 인코더의 정보를 활용하는 핵심적인 연결고리이다. 이 어텐션 레이어에서는 쿼리(Query)는 이전 디코더 레이어의 출력에서 가져오고, 키(Key)와 밸류(Value)는 인코더 스택 전체의 최종 출력에서 가져온다.10 이를 통해 디코더는 출력 단어를 생성하는 매 순간, 입력 문장 전체를 다시 한번 훑어보고 어떤 부분에 집중해야 할지를 결정한다. 예를 들어, 프랑스어 "Je suis étudiant"를 영어로 번역할 때, "student"를 생성하는 시점에서 디코더는 인코더 출력에서 "étudiant"에 해당하는 벡터에 높은 어텐션 가중치를 부여하게 된다.
3. **Position-wise FFN:** Encoder-Decoder Attention의 결과를 받아 비선형 변환을 수행하는 역할은 인코더의 FFN과 동일하다.

## 5.  실험 결과: 새로운 SOTA(State-of-the-Art)의 달성

트랜스포머 아키텍처의 우수성은 WMT 2014 기계 번역 벤치마크 데이터셋에서의 실험을 통해 입증되었다. 이 실험에는 대규모 데이터셋인 English-to-German(약 450만 문장 쌍)과 English-to-French(약 3600만 문장 쌍) 번역 태스크가 사용되었다.26

성능 평가의 핵심 지표인 BLEU 점수에서 트랜스포머는 당시 최고 성능을 자랑하던 순환 및 합성곱 기반 모델들을 모두 능가하는 결과를 보였다. 특히, 파라미터 수가 더 많은 'Transformer (big)' 모델은 English-to-German 번역에서 28.4 BLEU, English-to-French 번역에서 41.8 BLEU 점수를 기록하며 새로운 SOTA(State-of-the-Art)를 달성했다.10 이는 여러 모델의 결과를 앙상블한 기존 최고 기록보다도 2 BLEU 이상 높은 점수로, 단일 모델로서 전례 없는 성과였다.10

더욱 놀라운 점은 이러한 성능 향상이 훨씬 적은 훈련 비용으로 달성되었다는 것이다. 트랜스포머의 병렬 처리 능력 덕분에, 8개의 GPU를 사용하여 단 3.5일 만에 SOTA 성능에 도달할 수 있었다. 이는 기존 SOTA 모델들의 훈련 비용의 극히 일부에 불과한 수치였다.8 이 결과는 트랜스포머가 성능뿐만 아니라 효율성 측면에서도 기존 아키텍처를 압도하는, 진정한 패러다임의 전환임을 명확히 보여주었다.

| 모델 (Model)                 | EN-DE BLEU | EN-FR BLEU | 훈련 비용 (Training Cost, FLOPs) |
| ---------------------------- | ---------- | ---------- | -------------------------------- |
| GNMT + RL                    | 24.6       | 39.92      | $2.3 \cdot 10^{19}$              |
| ConvS2S                      | 25.16      | 40.46      | $9.6 \cdot 10^{18}$              |
| GNMT + RL Ensemble           | 26.30      | 41.16      | $1.8 \cdot 10^{20}$              |
| ConvS2S Ensemble             | 26.36      | 41.29      | $7.7 \cdot 10^{19}$              |
| **Transformer (base model)** | **27.3**   | **38.1**   | **$3.3 \cdot 10^{18}$**          |
| **Transformer (big)**        | **28.4**   | **41.8**   | **$2.3 \cdot 10^{19}$**          |

## 6.  트랜스포머가 연 시대: 의의와 영향력

"Attention is All You Need" 논문의 진정한 가치는 기계 번역 태스크의 SOTA 달성을 넘어, 이후 AI 연구의 지형을 완전히 바꾸어 놓았다는 데 있다. 트랜스포머는 단일 솔루션이 아니라, 다양한 문제에 맞게 분해하고 재조립할 수 있는 강력하고 모듈화된 '구성 요소들의 집합(composable toolkit)'을 제공했다.

### 6.1  BERT와 GPT의 초석: 아키텍처의 분화와 발전

트랜스포머의 인코더와 디코더는 각각 독립적인 잠재력을 가지고 있었으며, 후속 연구자들은 이를 간파하여 두 가지 주요 거대 언어 모델(LLM) 계열을 탄생시켰다.12

- **BERT (Bidirectional Encoder Representations from Transformers):** 트랜스포머의 **인코더** 부분만을 활용한 모델이다. BERT의 개발자들은 텍스트 생성 기능이 필요 없는, 주어진 텍스트의 깊은 문맥 이해가 핵심인 과제에 주목했다. 그들은 인코더를 분리하고, Masked Language Model(MLM)이라는 훈련 방식을 통해 문장의 양쪽 문맥을 동시에 고려하여 단어의 의미를 파악하는 강력한 '이해 엔진'을 만들어냈다.30 이는 감성 분석, 질문 응답, 개체명 인식과 같은 자연어 이해(NLU) 태스크에서 혁명적인 성능 향상을 이끌었다.
- **GPT (Generative Pre-trained Transformer):** 트랜스포머의 **디코더** 부분만을 활용한 모델이다. GPT의 개발자들은 별도의 입력 문장에 의존하지 않고, 주어진 프롬프트를 바탕으로 일관성 있는 텍스트를 생성하는 과제에 집중했다. 그들은 디코더의 자기회귀적 특성과 Masked Self-Attention 구조를 분리하여, 이전 문맥을 바탕으로 다음 단어를 예측하는 강력한 '생성 엔진'을 구축했다.30 이는 오늘날 ChatGPT와 같은 대화형 AI의 기반이 되었다.

이처럼 트랜스포머의 인코더와 디코더는 하나의 모델을 이루는 두 부분이 아니라, 각각 '표현 생성'과 '조건부 생성'이라는 근본적으로 다른 문제를 해결하는 독립적인 솔루션이었음이 밝혀졌다. 이러한 모듈성은 트랜스포머가 단일 문제를 해결하는 것을 넘어, 전체 NLP 분야의 문제 해결을 위한 기본 도구상자가 되게 한 핵심 요인이다.

### 6.2  자연어 처리를 넘어선 범용 아키텍처로의 확장

트랜스포머의 영향력은 NLP 영역에만 머무르지 않았다. 시퀀스 데이터를 처리하는 탁월한 능력은 다른 분야로의 확장을 촉발했다. 대표적인 예가 컴퓨터 비전 분야의 Vision Transformer(ViT)이다. ViT는 이미지를 여러 개의 작은 패치(patch)로 나눈 뒤, 이 패치들의 시퀀스를 트랜스포머 인코더에 입력하는 방식을 사용한다.14 이 접근법은 기존의 합성곱 신경망(CNN)이 지배하던 이미지 인식 분야에서 SOTA에 버금가거나 이를 능가하는 성능을 보이며, 트랜스포머가 특정 도메인에 국한되지 않는 범용적인 아키텍처임을 증명했다.7

## 7.  한계와 미래 연구 방향

혁신적인 아키텍처임에도 불구하고 트랜스포머는 몇 가지 명확한 한계를 가지고 있으며, 이는 현재 AI 연구의 주요 도전 과제가 되고 있다.

- **Self-Attention의 이차 복잡도(Quadratic Complexity):** 가장 큰 한계는 셀프 어텐션의 계산 복잡도가 시퀀스 길이 `n`에 대해 $O(n^2)$라는 점이다.6 시퀀스 길이가 두 배로 늘어나면 계산량과 메모리 요구량은 네 배로 증가한다. 이로 인해 수천 개 이상의 토큰으로 구성된 긴 문서나 고해상도 이미지를 처리하는 데 심각한 병목 현상이 발생한다.1
- **막대한 계산 비용과 과적합:** 수십억에서 수조 개의 파라미터를 가진 현대 LLM들은 훈련과 추론에 막대한 양의 컴퓨팅 자원과 에너지를 소모한다.1 또한, 모델의 복잡성이 매우 높기 때문에 제한된 데이터로 훈련할 경우 데이터의 노이즈까지 학습하는 과적합(overfitting) 문제에 취약할 수 있다.32

흥미롭게도, "Attention is All You Need"라는 원본 논문의 제목은 그 자체로 하나의 살아있는 연구 질문이 되어 후속 연구의 방향을 제시하고 있다. 수많은 연구들이 이 도발적인 명제에 직접적으로 응답하며 새로운 아키텍처를 제안하고 있다. 예를 들어, "Attention is All You Need **Until You Need Retention**" 34,  "**Element-wise Attention** Is All You Need" 35,  "**Tensor Product Attention** Is All You Need" 36와 같은 논문 제목들은 원본 논문과의 직접적인 대화를 시도하고 있다. 이는 단순한 인용을 넘어, 원본 논문이 제시한 '어텐션'이라는 패러다임이 후속 연구의 중심적인 논쟁거리가 되었음을 보여준다. 즉, 이 논문의 가장 큰 영향력은 정답을 제시한 것뿐만 아니라, 다음 세대의 AI 아키텍처 연구를 위한 핵심 의제를 설정했다는 데 있다.

## 8.  결론: 왜 '어텐션이 전부'였는가

"Attention is All You Need"는 순환이라는 오랜 관습을 과감히 버리고, 어텐션이라는 단일 메커니즘만으로 시퀀스 내의 모든 위치 간의 관계를 직접 모델링할 수 있음을 증명했다. 이 근본적인 발상의 전환은 두 가지 혁신을 가져왔다. 첫째, 장거리 의존성 문제를 효과적으로 해결하여 모델의 문맥 이해 능력을 한 차원 높였다. 둘째, 전례 없는 수준의 병렬화를 가능하게 하여, GPU의 성능을 최대한 활용하고 모델의 대규모화를 촉진하는 길을 열었다.

이 논문은 단순히 기계 번역 모델의 성능을 끌어올린 것을 넘어, 순차적 처리라는 패러다임 자체를 병렬적, 전역적 관계 모델링으로 전환시켰다. 이 새로운 패러다임 위에서 BERT와 GPT와 같은 현대 AI의 거인들이 탄생했으며, AI 연구의 방향은 근본적으로 재설정되었다. 트랜스포머가 가진 계산 복잡도와 같은 한계점들조차도, 이를 극복하려는 수많은 후속 연구를 촉발하며 AI 분야의 발전을 이끄는 원동력이 되고 있다.

결론적으로, 미래의 AI 아키텍처에서 어텐션이 여전히 '전부'일지는 미지수이다. 하지만 어텐션이 현대 AI 시대를 가능하게 한 결정적인 '시작'이었음은 부정할 수 없는 사실이다. 이 논문은 AI 역사에서 하나의 이정표로, 그 영향력은 앞으로도 오랫동안 지속될 것이다.

#### **참고 자료**

1. RNN vs LSTM vs GRU vs Transformers - GeeksforGeeks, 8월 16, 2025에 액세스, https://www.geeksforgeeks.org/deep-learning/rnn-vs-lstm-vs-gru-vs-transformers/
2. RNNs and LSTMs - Stanford University, 8월 16, 2025에 액세스, https://web.stanford.edu/~jurafsky/slp3/8.pdf
3. www.geeksforgeeks.org, 8월 16, 2025에 액세스, [https://www.geeksforgeeks.org/deep-learning/rnn-vs-lstm-vs-gru-vs-transformers/#:~:text=The%20main%20limitation%20of%20RNNs,machine%20translation%20or%20speech%20recognition.](https://www.geeksforgeeks.org/deep-learning/rnn-vs-lstm-vs-gru-vs-transformers/#:~:text=The main limitation of RNNs,machine translation or speech recognition.)
4. Recurrent Neural Networks (RNNs): Challenges and Limitations | by Indrajitbarat | Medium, 8월 16, 2025에 액세스, https://medium.com/@indrajitbarat9/recurrent-neural-networks-rnns-challenges-and-limitations-4534b25a394c
5. What is LSTM? Introduction to Long Short-Term Memory - Analytics Vidhya, 8월 16, 2025에 액세스, https://www.analyticsvidhya.com/blog/2021/03/introduction-to-long-short-term-memory-lstm/
6. Why does the transformer do better than RNN and LSTM in long-range context dependencies? - AI Stack Exchange, 8월 16, 2025에 액세스, https://ai.stackexchange.com/questions/20075/why-does-the-transformer-do-better-than-rnn-and-lstm-in-long-range-context-depen
7. Transformer (deep learning architecture) - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Transformer_(deep_learning_architecture)
8. Attention is All you Need - NIPS, 8월 16, 2025에 액세스, https://papers.nips.cc/paper/7181-attention-is-all-you-need
9. [PDF] Attention is All you Need - Semantic Scholar, 8월 16, 2025에 액세스, https://www.semanticscholar.org/paper/Attention-is-All-you-Need-Vaswani-Shazeer/204e3073870fae3d05bcbc2f6a8e263d9b72e776
10. Attention Is All You Need - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/1706.03762v7
11. The Illustrated Transformer: A Practical Guide | by Anote - Medium, 8월 16, 2025에 액세스, https://anote-ai.medium.com/the-illustrated-transformer-a-practical-guide-8fdc93963b89
12. Attention Is All You Need - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Attention_Is_All_You_Need
13. The Illustrated Transformer – Jay Alammar – Visualizing machine ..., 8월 16, 2025에 액세스, https://jalammar.github.io/illustrated-transformer/
14. What is a Transformer Model? - IBM, 8월 16, 2025에 액세스, https://www.ibm.com/think/topics/transformer-model
15. How Transformers Work: A Detailed Exploration of Transformer Architecture - DataCamp, 8월 16, 2025에 액세스, https://www.datacamp.com/tutorial/how-transformers-work
16. Transformers in Machine Learning - GeeksforGeeks, 8월 16, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/getting-started-with-transformers/
17. LLM Transformer Model Visually Explained - Polo Club of Data Science, 8월 16, 2025에 액세스, https://poloclub.github.io/transformer-explainer/
18. The Illustrated Transformer From Scratch, 8월 16, 2025에 액세스, https://idtjo.hosting.acm.org/wordpress/the-illustrated-transformer/
19. LaTeX Original - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/datasets/chunking-ai/chunking-test-data/resolve/main/long.tex?download=true
20. ml_equations_latex/index.md at master - GitHub, 8월 16, 2025에 액세스, https://github.com/blmoistawinde/ml_equations_latex/blob/master/index.md
21. Attention is All you Need - NIPS, 8월 16, 2025에 액세스, https://papers.neurips.cc/paper/7181-attention-is-all-you-need.pdf
22. Explore the Benefits of Transformer Models in AI - Simplilearn.com, 8월 16, 2025에 액세스, https://www.simplilearn.com/tutorials/generative-ai-tutorial/transformer-models
23. What are Transformers? - Transformers in Artificial Intelligence Explained - AWS, 8월 16, 2025에 액세스, https://aws.amazon.com/what-is/transformers-in-artificial-intelligence/
24. Transformer Architecture: The Positional Encoding - Amirhossein Kazemnejad's Blog, 8월 16, 2025에 액세스, https://kazemnejad.com/blog/transformer_architecture_positional_encoding/
25. What is the positional encoding in the transformer model? - Data Science Stack Exchange, 8월 16, 2025에 액세스, https://datascience.stackexchange.com/questions/51065/what-is-the-positional-encoding-in-the-transformer-model
26. "Attention Is All You Need" Explained | by Zaynab Awofeso | CodeX - Medium, 8월 16, 2025에 액세스, https://medium.com/codex/attention-is-all-you-need-explained-ebdb02c7f4d4
27. Transformer Clear Explanation: Attention Is All You Need! - 2017 | by Jinpeng Zhang, 8월 16, 2025에 액세스, https://dataturbo.medium.com/transformer-attention-is-all-you-need-fe6205c5be33
28. The Transformer Architecture, 8월 16, 2025에 액세스, https://www.auroria.io/the-transformer-architecture/
29. Jay Alammar – Visualizing machine learning one concept at a time., 8월 16, 2025에 액세스, https://jalammar.github.io/
30. Foundation Models, Transformers, BERT and GPT | Niklas Heidloff, 8월 16, 2025에 액세스, https://heidloff.net/article/foundation-models-transformers-bert-and-gpt/
31. BERT vs GPT: A Python Perspective on NLP Transformers | by PySquad | Medium, 8월 16, 2025에 액세스, https://medium.com/@pysquad/bert-vs-gpt-a-python-perspective-on-nlp-transformers-2f979761984f
32. (PDF) Drawbacks of Transformers - ResearchGate, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/369884473_Drawbacks_of_Transformers
33. Transformers in AI: Architecture & Use Cases - LXT, 8월 16, 2025에 액세스, https://www.lxt.ai/ai-glossary/transformers/
34. [2501.09166] Attention is All You Need Until You Need Retention - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/2501.09166
35. [2501.05730] Element-wise Attention Is All You Need - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/2501.05730
36. [2501.06425] Tensor Product Attention Is All You Need - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/2501.06425


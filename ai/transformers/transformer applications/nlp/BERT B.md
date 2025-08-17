# Bidirectional Encoder Representations from Transformers (BERT)

## 1. 서론: BERT의 등장과 자연어 처리의 패러다임 전환

### 0.1  2018년 이전 자연어 처리(NLP) 연구의 동향과 한계

2018년 구글(Google)에 의해 BERT가 발표되기 이전, 자연어 처리(Natural Language Processing, NLP) 분야는 점진적인 발전을 거듭하고 있었다. 초기 연구는 주로 사람이 직접 규칙과 패턴을 정의하는 명시적 알고리즘에 기반했으나, 이는 언어의 복잡성과 예외성을 모두 포괄하기 어렵다는 명백한 한계에 부딪혔다.1 1990년대 이후 컴퓨팅 성능의 발전과 빅데이터의 등장은 기계가 대량의 데이터로부터 통계적 패턴을 스스로 학습하는 머신러닝 시대를 열었다.1

2000년대에 들어 딥러닝 기술이 도입되면서 NLP는 비약적인 발전을 이루었다. 특히 순차적인 데이터 처리에 강점을 가진 순환 신경망(Recurrent Neural Network, RNN)과 그 변형인 장단기 기억(Long Short-Term Memory, LSTM) 모델이 주류를 이루었다.2 이 모델들은 단어의 순서와 문맥을 고려하여 문장을 처리할 수 있었지만, 두 가지 고질적인 문제를 안고 있었다. 첫째, 문장이 길어질수록 앞부분의 정보가 뒤로 전달되기 어려운 '장기 의존성 문제(long-range dependency problem)'가 발생했다. 둘째, 데이터를 순차적으로 처리해야 하는 구조적 특성상 GPU를 활용한 병렬 처리가 어려워 대규모 데이터 학습에 비효율적이었다.2 이러한 한계는 더 깊고 복잡한 언어 현상을 모델링하는 데 있어 중요한 제약으로 작용했다.

### 0.2  BERT가 제시한 '사전 훈련-미세 조정(Pre-training and Fine-tuning)' 패러다임의 혁신성

이러한 배경 속에서 2018년 10월 발표된 BERT는 NLP 연구의 흐름을 근본적으로 바꾸는 패러다임 전환을 이끌었다.4 BERT의 핵심 철학은 '사전 훈련-미세 조정(Pre-training and Fine-tuning)' 패러다임의 확립에 있다. 이는 레이블이 없는(unlabeled) 대규모 텍스트 데이터(e.g., 위키피디아, BookCorpus)를 사용하여 언어 자체의 구조와 패턴을 미리 학습하는 '사전 훈련' 단계와, 이렇게 학습된 범용 언어 모델을 소량의 레이블 있는(labeled) 데이터를 이용해 특정 다운스트림 태스크(downstream task)에 맞게끔 추가로 훈련하는 '미세 조정' 단계로 구성된다.5

이 접근법의 혁신성은 각기 다른 NLP 태스크를 해결하기 위해 고도로 특화되고 복잡하게 설계된(heavily-engineered) 모델 아키텍처를 개별적으로 구축해야 했던 기존의 비효율성을 극복했다는 점에 있다.6 BERT는 단 하나의 사전 훈련된 모델 구조에 최소한의 태스크별 레이어만 추가하고 미세 조정하는 것만으로 문장 분류, 개체명 인식, 질의응답 등 광범위한 NLP 태스크에서 당시 최고 성능(State-of-the-Art, SOTA)을 달성했다.9 이는 특정 태스크에 종속되지 않는, 진정한 의미의 '범용 언어 표현(universal language representation)'의 가능성을 실증적으로 보여준 기념비적인 성과였다.

### 0.3  보고서의 구조와 전개 방향 제시

본 보고서는 자연어 처리 분야에 혁명적인 변화를 가져온 BERT에 대한 심층적인 고찰을 목표로 한다. 먼저, BERT의 철학적 기반이 된 선행 연구들, 특히 ELMo와 OpenAI GPT의 성과와 한계를 분석하여 BERT의 등장 배경을 명확히 한다. 이어서 BERT 아키텍처의 근간을 이루는 트랜스포머 인코더의 구조와 핵심 메커니즘을 상세히 해부한다. 다음으로, 기존 언어 모델의 단방향성 제약을 극복하고 진정한 양방향 문맥 이해를 가능하게 한 독창적인 사전 훈련 전략, 즉 Masked Language Model(MLM)과 Next Sentence Prediction(NSP)을 심도 있게 다룬다.

이후, 사전 훈련된 BERT가 어떻게 다양한 다운스트림 태스크에 적용되어 강력한 성능을 발휘하는지 미세 조정 방법론을 구체적으로 설명하고, GLUE와 SQuAD 같은 주요 벤치마크에서 달성한 압도적인 성과를 수치적으로 제시한다. 또한, BERT의 성공 이후 등장한 RoBERTa, ALBERT, DistilBERT, ELECTRA와 같은 주요 후속 모델들의 특징을 비교 분석하여 BERT가 촉발한 기술적 진화의 계보를 조망한다. 마지막으로, BERT의 기술적 한계점과 막대한 연산 비용, 그리고 훈련 데이터에 내재된 사회적 편향 문제 등 비판적 고찰을 통해 균형 잡힌 시각을 제공하며, BERT가 현대 대규모 언어 모델(LLM) 시대에 남긴 유산과 미래 전망을 논하며 마무리한다.

## 1.  BERT 이전의 언어 모델: 단방향성의 한계

BERT의 혁신성을 온전히 이해하기 위해서는 그 이전에 존재했던 주요 언어 표현 모델들의 성과와 본질적인 한계를 먼저 살펴볼 필요가 있다. 특히 ELMo와 OpenAI GPT는 문맥적 임베딩과 트랜스포머 아키텍처의 가능성을 보여주었지만, '단방향성'이라는 근본적인 제약에서 자유롭지 못했다.

### 1.1  ELMo (Embeddings from Language Models): 얕은 양방향성의 성과와 과제

2018년 초에 발표된 ELMo는 '문맥을 고려한 단어 표현(Contextualized Word Representation)'이라는 개념을 대중화시킨 중요한 모델이다. ELMo 이전의 Word2Vec이나 GloVe와 같은 임베딩 방식은 단어의 의미를 고정된 하나의 벡터로 표현했기 때문에, "사과를 먹었다"의 '사과'와 "사과를 했다"의 '사과'처럼 동음이의어를 구분하지 못하는 한계가 있었다. ELMo는 이 문제를 해결하기 위해 깊은 양방향 LSTM(deep BiLSTM)을 사용했다.11

구체적으로 ELMo는 순방향(forward) LSTM과 역방향(backward) LSTM을 각각 독립적으로 훈련시킨 후, 각 토큰 위치에서 두 LSTM의 은닉 상태(hidden state) 벡터를 단순히 연결(concatenate)하는 방식을 취했다.11 이를 통해 단어가 문장 내에서 어떤 문맥에 놓여있는지에 따라 동적으로 다른 임베딩 벡터를 생성할 수 있게 되었다. 그러나 ELMo의 양방향성은 각 방향의 정보가 모델의 모든 층에서 깊게 상호작용하는 것이 아니라, 최종적으로 최상위 층에서 얕게(shallow) 결합되는 방식이었다.6 이는 진정한 의미의 통합된 양방향 문맥 정보를 학습하는 데 구조적인 한계를 가짐을 의미했다.

### 1.2  OpenAI GPT (Generative Pre-trained Transformer): 트랜스포머 디코더 기반 단방향 모델의 가능성과 명확한 한계

ELMo와 비슷한 시기에 등장한 OpenAI의 GPT(Generative Pre-trained Transformer)는 'Attention Is All You Need' 논문에서 제안된 트랜스포머 아키텍처를 언어 모델링에 성공적으로 적용한 첫 사례 중 하나이다. GPT는 트랜스포머의 디코더(Decoder) 블록만을 사용하여 구성되었으며, 왼쪽에서 오른쪽으로(left-to-right) 순차적으로 다음 단어를 예측하는 전통적인 언어 모델링 방식으로 사전 훈련되었다.6

GPT의 트랜스포머 디코더 구조는 셀프 어텐션(self-attention) 과정에서 미래의 토큰 정보를 참조하지 못하도록 마스킹(masking) 기법을 사용한다. 즉, 특정 위치의 토큰은 오직 그 이전에 등장한 토큰들과의 관계만을 학습할 수 있었다.6 이러한 단방향(unidirectional) 구조는 문장을 생성하는 과제(generative tasks)에서는 매우 효과적이었지만, 문장 전체의 맥락을 종합적으로 파악해야 하는 자연어 이해(NLU) 과제에서는 명백한 한계를 보였다. 예를 들어, 문장의 뒷부분에 결정적인 정보가 나오는 질의응답(Question Answering)이나 자연어 추론(Natural Language Inference) 같은 태스크에서 오른쪽 문맥을 고려하지 못하는 것은 성능 저하의 치명적인 원인이 되었다.6

### 1.3  진정한 양방향 문맥 이해의 필요성 대두

ELMo와 GPT의 성공은 문맥적 표현과 트랜스포머 아키텍처의 강력한 잠재력을 입증했지만, 동시에 그들의 한계는 NLP 모델이 인간 수준으로 언어를 이해하기 위해 넘어야 할 다음 과제가 무엇인지를 명확히 보여주었다. 그것은 바로 모델의 모든 층에서 왼쪽과 오른쪽 문맥을 동시에, 그리고 깊게 통합하는 '깊은 양방향성(deep bidirectionality)'의 구현이었다.6

그러나 이는 기술적으로 매우 어려운 문제였다. 표준적인 언어 모델의 목표 함수는 다음 단어를 예측하는 것($P(w_i | w_1,..., w_{i-1})$)으로, 본질적으로 단방향이다. 만약 이를 양방향으로 확장하여 $P(w_i | w_1,..., w_{i-1}, w_{i+1},..., w_n)$를 예측하려 하면, 깊은 트랜스포머 구조에서는 심각한 정보 유출 문제가 발생한다. 특정 레이어 $L$에서 토큰 $w_{i+1}$의 표현은 이미 이전 레이어 $L-1$의 $w_i$ 정보에 영향을 받은 상태이기 때문에, 모델이 예측해야 할 정답을 간접적으로 '미리 엿보는(see itself)' 상황이 발생하여 예측이 무의미해지고 학습이 제대로 이루어지지 않는 것이다.14

결국 ELMo의 얕은 결합 방식이나 GPT의 단방향 마스킹은 이 문제를 회피하기 위한 타협책이었다. 이들의 한계는 단순히 점진적인 성능 차이가 아니라, 기존 언어 모델링 목표 함수가 가진 근본적인 '구조적 천장(architectural ceiling)'을 드러냈다. 따라서 진정한 깊은 양방향성을 달성하기 위해서는 아키텍처의 개선을 넘어, 사전 훈련의 목표 함수 자체를 근본적으로 재설계해야 한다는 필연적인 결론에 도달하게 된다. 이 문제의식과 해결책이 바로 BERT의 핵심 혁신인 Masked Language Model(MLM)의 탄생으로 이어졌다.

## 2.  BERT의 핵심 아키텍처: 트랜스포머 인코더 심층 분석

BERT는 'Bidirectional **Encoder** Representations from **Transformers**'라는 이름에서 알 수 있듯이, 트랜스포머의 인코더(Encoder) 구조만을 사용하여 언어를 이해하는 모델이다.12 이는 문장 생성을 목표로 하는 디코더 구조의 GPT와 명확히 구분되는 지점이다. BERT의 강력한 문맥 이해 능력은 바로 이 트랜스포머 인코더 아키텍처의 정교한 설계에서 비롯된다.

### 2.1  입력 표현(Input Representation): 세 가지 임베딩의 결합

BERT는 텍스트를 모델이 처리할 수 있는 숫자 벡터로 변환하기 위해, 단순한 단어 임베딩을 넘어 세 가지 종류의 임베딩 정보를 합산하여 최종 입력 표현을 생성한다. 이는 단어의 고유한 의미뿐만 아니라 문장 내에서의 위치와 문장 간의 관계까지 종합적으로 고려하기 위함이다.11

- **토큰 임베딩 (Token Embeddings):** BERT는 WordPiece 토큰화 방식을 사용한다.11 이는 자주 사용되는 단어는 그대로 유지하되, 드물게 등장하거나 사전에 없는 단어(Out-of-Vocabulary, OOV)는 의미 있는 더 작은 단위(subword)로 분리하는 기법이다. 예를 들어, 'embedding'이라는 단어는 'embed'와 '##ding'으로 나뉠 수 있다. 이 방식은 어휘집의 크기를 효율적으로 관리하면서도 OOV 문제에 강건하게 대처할 수 있게 해준다.
- **세그먼트 임베딩 (Segment Embeddings):** 이 임베딩은 두 개의 문장을 입력으로 받는 태스크(e.g., Next Sentence Prediction, 질의응답)를 위해 설계되었다. 모델이 입력 시퀀스에서 첫 번째 문장과 두 번째 문장을 구분할 수 있도록, 첫 번째 문장의 모든 토큰에는 문장 A를 나타내는 임베딩 벡터($E_A$)를, 두 번째 문장의 모든 토큰에는 문장 B를 나타내는 임베딩 벡터($E_B$)를 더해준다.11
- **포지션 임베딩 (Position Embeddings):** 트랜스포머 아키텍처는 RNN과 달리 순환 구조가 없어 토큰의 순서 정보를 내재적으로 처리하지 못한다. 이 문제를 해결하기 위해 각 토큰의 위치 정보를 담은 벡터를 더해준다.16 'Attention Is All You Need' 논문에서는 사인(sine)과 코사인(cosine) 함수를 이용한 고정된 값을 사용했지만, BERT는 각 위치에 해당하는 고유한 벡터를 학습 파라미터로 두고 데이터로부터 직접 학습하는 방식을 채택했다.11 이를 통해 모델은 최대 512개의 위치에 대한 최적의 표현을 학습하며, 순서 정보를 모델에 주입하면서도 전체 시퀀스를 병렬적으로 처리하는 트랜스포머의 장점을 유지할 수 있다.18

### 2.2  핵심 메커니즘: 스케일드 닷-프로덕트 어텐션과 멀티-헤드 셀프 어텐션

BERT의 심장부에는 문장 내 단어들 간의 복잡한 관계를 파악하는 셀프 어텐션(Self-Attention) 메커니즘이 자리 잡고 있다.11 이는 문장 내 모든 단어 쌍에 대해 연관성을 계산하고, 이를 가중치로 사용하여 각 단어의 표현을 문맥에 맞게 재조정하는 과정이다.

각 토큰의 입력 벡터는 학습 가능한 가중치 행렬에 의해 쿼리(Query, $Q$), 키(Key, $K$), 밸류(Value, $V$)라는 세 가지 벡터로 각각 선형 변환된다.21 이 세 벡터는 다음과 같은 역할을 수행한다.

- **쿼리 ($Q$):** 현재 처리 중인 단어의 표현.
- **키 ($K$):** 문장 내 다른 모든 단어의 표현으로, 쿼리와의 유사도를 계산하는 데 사용된다.
- **밸류 ($V$):** 문장 내 다른 모든 단어의 실제 의미 정보를 담고 있는 표현.

**스케일드 닷-프로덕트 어텐션 (Scaled Dot-Product Attention):** 어텐션 가중치는 쿼리 벡터와 모든 키 벡터 간의 내적(dot product)을 통해 계산된다. 이 내적 값은 두 벡터의 유사도를 나타내며, 값이 클수록 연관성이 높다는 의미이다. 이 점수가 너무 커지는 것을 방지하고 그래디언트를 안정화시키기 위해 키 벡터의 차원($d_k$)의 제곱근으로 나누어주는 스케일링 과정을 거친다.21 이후 소프트맥스(softmax) 함수를 적용하여 모든 가중치의 합이 1이 되는 확률 분포를 만들고, 이 가중치를 각 단어의 밸류 벡터에 곱한 후 모두 더하여 최종적인 문맥화된 벡터를 생성한다.23
$$
\text{Attention}(Q, K, V) = \text{softmax} \left( \frac{QK^T}{\sqrt{d_k}} \right) V
$$
**멀티-헤드 셀프 어텐션 (Multi-Head Self-Attention):** BERT는 한 번의 어텐션을 수행하는 대신, $Q, K, V$ 벡터를 여러 개의 작은 차원으로 나누어 여러 개의 어텐션 헤드(head)가 병렬적으로 어텐션을 수행하는 방식을 사용한다.22 예를 들어, 768차원의 벡터를 12개의 헤드로 나누면 각 헤드는 64차원의 벡터를 가지고 독립적으로 어텐션을 계산한다. 각 헤드는 서로 다른 종류의 언어적 관계(e.g., 주어-동사 관계, 대명사 참조 관계 등)에 집중하여 학습할 수 있다. 이렇게 각 헤드에서 계산된 결과들은 다시 하나로 결합(concatenate)된 후, 최종적으로 선형 변환을 거쳐 원래 차원의 출력 벡터를 생성한다. 이 멀티-헤드 방식은 모델이 단일 관점이 아닌, 다양한 관점에서 문장 내 관계를 종합적으로 포착하는 능력을 부여한다.22
$$
\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_h)W^O
$$
여기서 $ \text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V) $이다.

### 2.3  인코더 블록의 구성 요소

BERT는 앞서 설명한 멀티-헤드 셀프 어텐션과 위치-별 피드포워드 신경망(Position-wise Feed-Forward Network)을 하나의 기본 단위인 '트랜스포머 인코더 블록'으로 구성하고, 이를 여러 층으로 깊게 쌓아 모델을 구축한다.3 이 깊은 구조가 안정적으로 학습될 수 있었던 것은 각 블록 내에 정교하게 설계된 두 가지 핵심 요소 덕분이다.

- **잔차 연결 (Residual Connection):** 각 하위 레이어(멀티-헤드 어텐션, 피드포워드 신경망)는 입력 $x$를 받아 변환한 후, 그 출력에 원래 입력 $x$를 다시 더해주는 구조($x + \text{SubLayer}(x)$)를 가진다.27 이는 마치 고속도로처럼 정보와 그래디언트가 하위 레이어를 건너뛰어 상위 레이어로 직접 전달될 수 있는 경로를 만들어주는 것과 같다. 이 '지름길' 덕분에 모델이 아무리 깊어져도 초기 레이어까지 학습 신호가 효과적으로 전달되어 기울기 소실(vanishing gradient) 문제를 완화하고, 전체 네트워크의 학습을 안정시킨다.28
- **레이어 정규화 (Layer Normalization):** 각 잔차 연결 다음에는 레이어 정규화가 적용된다.27 이는 각 레이어에 들어오는 입력의 분포가 훈련 과정에서 심하게 변동하는 것을 막고, 평균이 0, 분산이 1에 가깝도록 정규화하여 학습 과정을 안정시키고 수렴 속도를 높이는 역할을 한다.29

이러한 구성 요소들의 유기적인 결합이야말로 BERT의 성공을 가능하게 한 핵심이다. 셀프 어텐션이 문맥을 이해하는 '무엇'을 제공했다면, 깊은 스태킹(stacking)과 이를 뒷받침하는 잔차 연결 및 레이어 정규화는 그것을 '어떻게' 가능하게 했는지를 보여준다. 낮은 레이어에서는 지역적인 구문 관계를 포착하고, 이 정보가 잔차 연결을 통해 상위 레이어로 전달되면서 레이어 정규화로 안정화된 후, 더 높은 레이어에서는 이를 종합하여 추상적인 의미 관계를 학습하는 계층적인 특징 추출이 가능해진다.31 이처럼 각 요소가 서로의 약점을 보완하고 장점을 극대화하는 공생 관계를 통해 깊은 트랜스포머 인코더의 강력한 표현력이 발현되는 것이다.

### 2.4  모델 규모: BERT-Base와 BERT-Large

BERT는 연구의 확장성과 실용성을 고려하여 두 가지 규모의 모델을 제안했다.5

- **BERT-Base:** 12개의 트랜스포머 블록(L=12), 768차원의 은닉 상태(H=768), 12개의 어텐션 헤드(A=12)로 구성되며, 총 파라미터 수는 약 1억 1천만(110M) 개이다.
- **BERT-Large:** 24개의 트랜스포머 블록(L=24), 1024차원의 은닉 상태(H=1024), 16개의 어텐션 헤드(A=16)로 구성되며, 총 파라미터 수는 약 3억 4천만(340M) 개이다.

BERT 논문에서 가장 놀라운 발견 중 하나는, BERT-Large 모델이 훈련 데이터가 수천 개에 불과한 작은 데이터셋에서도 BERT-Base보다 월등한 성능을 보였다는 점이다.6 이는 대규모 데이터로 사전 훈련된 모델의 강력한 일반화 성능을 입증한 것으로, 이후 모델의 크기를 키우는 '스케일링 법칙(Scaling Law)' 연구의 중요한 초석이 되었다.

## 3.  진정한 양방향성의 구현: 사전 훈련 전략

BERT의 가장 핵심적인 혁신은 아키텍처 자체보다도, 그 아키텍처의 잠재력을 최대한으로 끌어낸 독창적인 사전 훈련(Pre-training) 전략에 있다. 기존 언어 모델의 단방향성이라는 근본적인 한계를 극복하기 위해, BERT는 두 가지 새로운 비지도 학습(unsupervised learning) 과제를 제안했다.

### 3.1  Masked Language Model (MLM): Cloze Task에서 영감을 받은 혁신적 목표 함수

Masked Language Model(MLM)은 BERT가 깊은 양방향성을 구현할 수 있게 한 핵심 아이디어다. 이는 전통적인 언어 모델링처럼 다음 단어를 예측하는 대신, 문장의 일부 단어를 무작위로 가리고(`` 토큰으로 치환) 주변의 모든 문맥(왼쪽과 오른쪽 모두)을 이용하여 가려진 원래 단어를 예측하도록 모델을 훈련시키는 방식이다.6 이는 빈칸 채우기 문제와 유사한 클로즈 테스트(Cloze Task)에서 영감을 받았다.

이 간단해 보이는 변화는 NLP에 혁명적인 영향을 미쳤다. MLM은 모델이 예측 대상 단어의 양쪽 문맥을 동시에, 그리고 모델의 모든 레이어에서 깊게 참고하도록 강제한다. 이를 통해 모델은 단어의 피상적인 순서가 아닌, 문장 전체의 구조와 의미 속에서 각 단어의 역할을 파악하는 진정한 문맥적 표현을 학습하게 된다.6

#### 3.1.1  80-10-10 마스킹 전략의 이론적 근거와 효과

MLM을 단순하게 모든 마스킹 대상 토큰을 `로 바꾸는 방식으로 구현하면 한 가지 문제가 발생한다. 바로 사전 훈련 단계에서는 ` 토큰이 등장하지만, 실제 다운스트림 태스크에 모델을 적용하는 미세 조정 단계에서는 이 토큰이 전혀 나타나지 않는다는 '불일치(mismatch)' 문제다.34

이 문제를 완화하기 위해 BERT는 정교한 80-10-10 마스킹 전략을 도입했다. 전체 토큰의 15%를 예측 대상으로 무작위 선택한 후, 이 선택된 토큰들을 다음과 같이 처리한다 6:

1. **80%의 경우:** 선택된 토큰을 `토큰으로 교체한다. (e.g., "my dog is hairy" → "my dog is`")
2. **10%의 경우:** 선택된 토큰을 어휘집 내의 다른 무작위 토큰으로 교체한다. (e.g., "my dog is hairy" → "my dog is apple")
3. **10%의 경우:** 선택된 토큰을 변경하지 않고 그대로 둔다. (e.g., "my dog is hairy" → "my dog is hairy")

이 전략의 효과는 모델이 특정 토큰(``)의 존재 여부에만 의존하여 예측하는 것을 방지하는 데 있다. 모델은 입력으로 주어진 토큰이 진짜 원래 단어일 수도 있고(10%의 경우), 완전히 엉뚱한 단어로 바뀌었을 수도 있다(10%의 경우)는 사실을 학습해야 한다. 따라서 모델은 단순히 입력 토큰 자체의 정보에만 의존할 수 없으며, 주변 문맥 정보를 훨씬 더 깊이 고려하여 각 토큰의 표현을 생성하도록 강제된다. 이는 결과적으로 모델이 모든 입력 토큰에 대해 더욱 풍부하고 강건한 분산적 문맥 표현(distributional contextual representation)을 학습하게 만들어 표현의 질을 높이는 효과를 가져온다.6

### 3.2  Next Sentence Prediction (NSP): 문장 간 관계 학습을 위한 이진 분류 과제

MLM이 단어 수준의 문맥 이해를 담당한다면, Next Sentence Prediction(NSP)은 문장 수준의 관계를 이해하는 능력을 모델에 부여하기 위해 설계되었다. 질의응답(QA), 자연어 추론(NLI)과 같이 두 문장 사이의 논리적 관계를 파악하는 것이 핵심인 다운스트림 태스크에서 좋은 성능을 내기 위해서는 이러한 능력이 필수적이다.6

NSP는 간단한 이진 분류 과제다. 대규모 텍스트 코퍼스에서 두 문장을 가져와 A와 B로 지정한 뒤, B가 실제로 A의 바로 다음 문장인지 아닌지를 예측하도록 모델을 훈련시킨다. 훈련 데이터셋은 다음과 같이 50:50 비율로 구성된다 6:

- **50% (Label: `IsNext`):** 코퍼스에서 실제로 연속된 두 문장을 가져온다.
- **50% (Label: `NotNext`):** 코퍼스 내 서로 다른 문서에서 임의로 추출한 두 문장을 짝지어, 의미적으로 관련이 없게 만든다.

학습 시에는 전체 입력 시퀀스의 가장 앞에 위치하는 특수 토큰인 ``의 최종 출력 벡터를 사용하여 이진 분류를 수행한다.6 BERT 논문에서는 이 NSP 사전 훈련이 QA와 NLI 태스크 성능에 크게 기여했다고 보고했다.6

#### 3.2.1  NSP의 유효성에 대한 후속 연구들의 비판적 재평가

그러나 BERT 이후 등장한 RoBERTa와 같은 후속 연구들은 NSP 태스크의 실효성에 대해 비판적인 재평가를 내놓았다. 이들 연구는 실험을 통해 NSP 태스크를 제거하고 오직 MLM만으로 사전 훈련했을 때, 여러 다운스트림 태스크에서 성능이 오히려 향상되거나 거의 차이가 없음을 보여주었다.41

NSP가 비효과적이었던 주된 원인으로 지목된 것은 태스크의 난이도가 너무 낮다는 점이었다. `NotNext` 샘플을 서로 다른 문서에서 가져왔기 때문에, 모델은 두 문장 간의 섬세한 논리적 일관성(coherence)을 학습하기보다는 단순히 두 문장의 주제(topic)가 동일한지만으로도 문제를 쉽게 맞힐 수 있었다. 즉, NSP는 문장 간의 진정한 관계 추론 능력이 아닌, 단순한 주제 예측 태스크로 변질될 가능성이 높았다.42 이러한 비판에 따라 ALBERT에서는 문장의 순서를 맞추는 Sentence-Order Prediction(SOP)이라는 더 정교한 과제로 대체되었고, 많은 후속 모델들은 NSP를 아예 제거하는 방향으로 발전했다.

## 4.  전이 학습의 정수: 다운스트림 태스크 미세 조정

BERT의 진정한 힘은 대규모 데이터로 학습된 풍부한 언어 표현을 다양한 특정 문제(다운스트림 태스크)에 효과적으로 전이시킬 수 있다는 점에 있다. 이는 '미세 조정(Fine-tuning)'이라는 과정을 통해 이루어지며, 사전 훈련된 BERT 모델의 가중치를 초기값으로 사용하여 각 태스크에 맞는 소량의 레이블된 데이터로 모델 전체를 재학습하는 방식이다. BERT는 태스크의 종류에 따라 입력 형식과 출력 레이어 구조를 유연하게 변경하여 광범위한 문제에 적용될 수 있다.

### 4.1  문장 및 문장 쌍 분류 (GLUE 등): `` 토큰의 역할과 분류 헤드

감성 분석, 자연어 추론(NLI), 문장 유사도 측정 등과 같이 문장 전체 또는 두 문장 간의 관계를 분류하는 태스크는 BERT의 가장 대표적인 적용 분야다.

- **입력 형식:** 단일 문장을 분류할 때는 입력 시퀀스 앞에 `토큰을, 뒤에` 토큰을 붙여 ` 문장 A` 형태로 구성한다. 두 문장 간의 관계를 분류할 때는 ` 문장 A 문장 B` 형태로 두 문장을 연결하여 입력한다.44
- **`토큰의 역할:**`는 Classification을 의미하는 특수 토큰으로, 항상 시퀀스의 가장 첫 번째 위치에 존재한다. BERT의 모든 트랜스포머 인코더 레이어를 통과하면서, 셀프 어텐션 메커니즘을 통해 시퀀스 내 모든 다른 토큰들의 정보를 종합적으로 수집하고 요약하는 역할을 수행한다. NSP 사전 훈련 과정에서 이 `토큰의 출력 벡터가 문장 쌍의 관계를 예측하는 데 사용되었기 때문에,` 토큰은 자연스럽게 문장 전체의 의미를 응축한 표현(aggregate sequence representation)을 학습하게 된다.46
- **분류 헤드(Classification Head):** 미세 조정 시, 사전 훈련된 BERT 모델의 구조 위에 간단한 완전 연결 계층(fully-connected layer)으로 구성된 분류 헤드를 추가한다. BERT의 마지막 트랜스포머 레이어에서 나온 `` 토큰에 해당하는 최종 은닉 상태 벡터(보통 768 또는 1024 차원)를 이 분류 헤드의 입력으로 사용한다. 분류 헤드는 이 벡터를 받아 최종적으로 원하는 클래스의 수(e.g., 긍정/부정의 경우 2개)에 해당하는 로짓(logit) 값을 출력하고, 소프트맥스 함수를 통해 확률 값으로 변환한다. 미세 조정 과정에서는 BERT의 모든 파라미터와 새로 추가된 분류 헤드의 가중치가 함께 업데이트되어 태스크에 최적화된다.32

### 4.2  토큰 수준 분류 (개체명 인식, NER 등): 각 토큰 출력 벡터와 IOB 태깅 스킴

개체명 인식(Named Entity Recognition, NER)과 같이 문장 내 각 토큰에 대해 레이블을 예측해야 하는 태스크에도 BERT를 효과적으로 적용할 수 있다.

- **아키텍처:** 문장 분류와 마찬가지로, 사전 훈련된 BERT 모델 위에 토큰별 분류를 위한 선형 레이어를 추가한다. 다만 이 분류기는 `` 토큰의 출력만 사용하는 것이 아니라, 모든 입력 토큰 각각의 최종 출력 벡터에 독립적으로 적용된다.50

- **출력:** 입력 시퀀스(e.g., "Jim Henson was a puppeteer")가 BERT를 통과하면, 각 토큰("Jim", "Henson", "was",...)에 해당하는 최종 은닉 상태 벡터가 생성된다. 이 벡터들이 각각 토큰 분류 헤드를 통과하여 해당 토큰이 어떤 개체명 태그(e.g., 인물, 기관, 장소, 기타)에 속하는지에 대한 예측을 수행한다.11

- **IOB 태깅 스킴:** "Jim Henson"과 같이 여러 토큰으로 구성된 개체명을 정확하게 표현하기 위해 IOB(Inside-Outside-Beginning) 태깅 스킴이 널리 사용된다.50

  - **B-PER:** 인물(Person) 개체의 시작(Beginning) 토큰. (e.g., "Jim")

  - **I-PER:** 인물 개체의 내부(Inside) 토큰. (e.g., "Henson")

  - O: 개체가 아님(Outside). (e.g., "was", "a", "puppeteer")

    이 방식을 통해 모델은 개체명의 경계를 명확하게 학습하고 예측할 수 있다.

### 4.3  질의응답 (SQuAD 등): 질문-문단 입력 구성 및 정답 스팬(Span) 예측

SQuAD(Stanford Question Answering Dataset)와 같은 추출적 질의응답(Extractive Question Answering)은 주어진 문단(Passage) 내에서 질문(Question)에 대한 정답에 해당하는 텍스트 구간(span)을 찾아내는 태스크다.

- **입력 형식:** 질문과 문단을 하나의 시퀀스로 묶어 BERT에 입력한다. `` 토큰을 사용하여 질문과 문단을 구분하며, 전체적인 형식은 ` 질문 문단`이 된다.53 이때 세그먼트 임베딩을 통해 모델은 어떤 부분이 질문이고 어떤 부분이 문단인지 구분할 수 있다.53
- **아키텍처:** BERT 모델 위에 정답 스팬의 시작 위치(Start Position)를 예측하는 레이어와 끝 위치(End Position)를 예측하는 두 개의 독립적인 선형 레이어를 추가한다.54 이 두 레이어는 각각 문단 내 모든 토큰 위치에 대해 점수를 출력한다.
- **예측 방식:** 문단에 속한 모든 토큰의 최종 은닉 상태 벡터 $T_i$가 각각 시작 예측 레이어와 끝 예측 레이어를 통과한다. 시작 예측 레이어는 각 토큰 $i$가 정답의 시작일 확률을 계산하고, 끝 예측 레이어는 각 토큰 $j$가 정답의 끝일 확률을 계산한다. 이는 보통 각 토큰의 최종 벡터와 학습 가능한 시작/끝 가중치 벡터 간의 내적을 계산한 후, 문단 내 모든 토큰에 대해 소프트맥스 함수를 적용하여 확률 분포를 얻는 방식으로 이루어진다.53 최종적으로, 시작 확률이 가장 높은 토큰과 끝 확률이 가장 높은 토큰을 조합하여(단, 시작 위치가 끝 위치보다 앞서야 함) 가장 가능성이 높은 정답 스팬을 예측한다.

## 5.  BERT의 성능 평가와 영향력

BERT는 공개와 동시에 당시 존재하던 11개의 주요 NLP 태스크에서 최고 성능(SOTA)을 달성하며 학계와 산업계에 큰 충격을 주었다.9 그 성능은 단순히 점진적인 개선이 아닌, 기존 모델들과의 격차를 크게 벌리는 압도적인 수준이었다.

### 5.1  주요 벤치마크에서의 성과 분석

BERT의 성능은 다양한 자연어 이해 능력을 종합적으로 평가하는 GLUE 벤치마크와 기계 독해 능력의 표준 척도로 여겨지는 SQuAD 벤치마크에서 특히 두드러졌다.

- **GLUE (General Language Understanding Evaluation):** GLUE는 자연어 추론(MNLI, RTE), 문장 유사도(MRPC, QQP), 감성 분석(SST-2), 문법성 판단(CoLA) 등 9개의 다양한 태스크로 구성된 벤치마크 모음이다. BERT 이전의 SOTA 모델이었던 OpenAI GPT가 평균 75.1점을 기록한 반면, BERT-Base는 79.6점, BERT-Large는 82.1점을 달성하며 큰 폭의 성능 향상을 이루었다. 공식 GLUE 리더보드에서는 BERT-Large가 80.5점을 기록하여, 기존 SOTA 모델들을 압도했다.6
- **SQuAD (Stanford Question Answering Dataset) v1.1:** SQuAD는 주어진 지문에서 질문에 대한 정답을 찾아내는 기계 독해 태스크다. BERT 이전의 SOTA 모델은 정교한 앙상블 기법을 사용해야 F1 score 91.7점을 기록했다. 그러나 BERT-Large는 단일 모델만으로도 이를 뛰어넘는 성능을 보였고, TriviaQA 데이터셋으로 추가 미세 조정을 거친 앙상블 모델은 F1 score 93.2점이라는 경이적인 기록을 세웠다. 이는 인간의 평균 성능(F1 score 91.2)을 2.0점이나 상회하는 수치로, 특정 과제에서 기계가 인간의 독해 능력을 넘어설 수 있음을 보여준 상징적인 사건이었다.6

아래 표는 BERT 논문에 제시된 주요 벤치마크 결과를 이전 SOTA 모델과 비교하여 정리한 것이다.

**Table 1: GLUE 벤치마크 태스크별 성능 비교 (BERT vs. 이전 SOTA)**

| System     | MNLI-(m/mm) | QQP  | QNLI | SST-2 | CoLA | STS-B | MRPC | RTE  | Average |
| ---------- | ----------- | ---- | ---- | ----- | ---- | ----- | ---- | ---- | ------- |
| OpenAI GPT | 82.1/81.4   | 70.3 | 87.4 | 91.3  | 45.4 | 80.0  | 82.3 | 56.0 | 75.1    |
| BERT_BASE  | 84.6/83.4   | 71.2 | 90.5 | 93.5  | 52.1 | 85.8  | 88.9 | 66.4 | 79.6    |
| BERT_LARGE | 86.7/85.9   | 72.1 | 92.7 | 94.9  | 60.5 | 86.5  | 89.3 | 70.1 | 82.1    |

**Table 2: SQuAD v1.1 벤치마크 성능 비교 (BERT vs. 이전 SOTA)**

| System                     | Dev EM | Dev F1 | Test EM | Test F1  |
| -------------------------- | ------ | ------ | ------- | -------- |
| Human                      | -      | -      | 82.3    | 91.2     |
| #1 Ensemble (nlnet)        | -      | -      | 86.0    | 91.7     |
| BERT_LARGE (Single)        | 84.1   | 90.9   | -       | -        |
| BERT_LARGE (Ens.+TriviaQA) | 86.2   | 92.2   | 87.4    | **93.2** |

### 5.2  NLP 연구 및 산업계에 미친 파급 효과

BERT의 압도적인 성능은 NLP 연구 및 개발 패러다임에 지대한 영향을 미쳤다.

- **'사전 훈련-미세 조정' 패러다임의 표준화:** BERT의 성공 이후, 거의 모든 NLP 연구는 대규모 언어 모델을 사전 훈련하고 이를 특정 태스크에 미세 조정하는 방식을 표준으로 채택하게 되었다.8 이는 각 태스크에 맞는 복잡한 모델을 처음부터 설계하고 훈련시키던 기존의 접근법을 대체하며, 연구 개발의 효율성을 극적으로 향상시켰다.
- **산업계의 광범위한 도입:** BERT의 강력한 언어 이해 능력은 다양한 산업 분야에 빠르게 적용되었다. 가장 대표적인 사례는 구글 검색 엔진이다. 구글은 BERT를 도입하여 "브라질 여행객의 미국 비자 필요 여부"와 같이 전치사의 의미가 중요한 복잡하고 대화적인 검색어의 의도를 훨씬 더 정확하게 파악할 수 있게 되었다.7 이 외에도 BERT와 그 파생 모델들은 고객센터의 챗봇, 제품 리뷰 감성 분석, 뉴스 기사 및 법률 문서 요약, 스팸 메일 필터링 등 수많은 상용 애플리케이션의 성능을 획기적으로 개선하는 데 핵심적인 역할을 하고 있다.7 BERT의 등장은 NLP 기술이 실험실 수준을 넘어 실질적인 비즈니스 가치를 창출하는 시대를 본격적으로 열었다고 평가할 수 있다.

## 6.  BERT를 넘어서: 후속 모델과 발전 방향

BERT는 NLP 분야에 새로운 기준을 제시했지만, 동시에 개선의 여지도 남겼다. BERT의 성공 이후, 수많은 연구 그룹이 BERT의 아키텍처와 훈련 방식을 분석하고 이를 개선하거나 특정 목적에 맞게 최적화한 다양한 파생 모델들을 발표했다. 이들 후속 모델은 훈련 효율성, 파라미터 효율성, 경량화 등 다양한 측면에서 BERT를 발전시키며 현대 대규모 언어 모델(LLM)의 기술적 토대를 마련했다.

### 6.1  RoBERTa (A Robustly Optimized BERT Pretraining Approach): 훈련 방식 최적화

페이스북 AI(현 Meta AI)에서 발표한 RoBERTa는 BERT의 아키텍처는 거의 변경하지 않고, 사전 훈련 전략을 철저하게 최적화하는 것만으로도 성능을 크게 향상시킬 수 있음을 보여주었다.42

- **주요 개선점:**
  1. **NSP 태스크 제거:** RoBERTa 연구진은 NSP 태스크가 다운스트림 태스크 성능 향상에 거의 도움이 되지 않거나 오히려 방해가 된다는 것을 실험적으로 밝혀내고 이를 과감히 제거했다.42
  2. **더 큰 데이터와 더 긴 훈련:** BERT가 16GB의 데이터로 훈련된 반면, RoBERTa는 160GB에 달하는 훨씬 더 크고 다양한 데이터셋을 사용하여 더 긴 시간, 더 큰 배치 사이즈로 훈련했다.71
  3. **동적 마스킹(Dynamic Masking):** 기존 BERT는 데이터 전처리 과정에서 마스킹을 한 번만 수행하여 훈련 내내 동일한 마스크 패턴을 사용했다(정적 마스킹). 반면 RoBERTa는 훈련 데이터를 모델에 입력할 때마다 매번 새로운 마스킹 패턴을 생성하는 동적 마스킹을 도입하여, 모델이 더 다양한 패턴을 학습하고 과적합을 방지하도록 했다.42

이러한 훈련 방식의 최적화를 통해 RoBERTa는 BERT-Large를 능가하는 성능을 달성하며, 사전 훈련 방법론의 중요성을 다시 한번 입증했다.

### 6.2  ALBERT (A Lite BERT): 파라미터 효율성 증대

ALBERT는 BERT의 성능을 최대한 유지하면서 모델의 파라미터 수를 획기적으로 줄여 메모리 사용량과 훈련 속도를 개선하는 데 초점을 맞춘 모델이다.72

- **주요 기술:**
  1. **임베딩 행렬 분해(Factorized Embedding Parameterization):** BERT에서는 단어 임베딩의 차원($E$)과 트랜스포머 은닉층의 차원($H$)이 동일했다. 하지만 ALBERT는 단어 임베딩은 문맥 독립적인 정보를, 은닉층은 문맥 의존적인 정보를 담고 있으므로 두 차원이 같을 필요가 없다고 보았다. 거대한 어휘 임베딩 행렬($V \times H$)을 두 개의 작은 행렬($V \times E$와 $E \times H$)로 분해하고 $E \ll H$로 설정함으로써 파라미터 수를 크게 줄였다.72
  2. **교차-레이어 파라미터 공유(Cross-layer Parameter Sharing):** ALBERT는 모든 트랜스포머 레이어가 동일한 파라미터(어텐션 및 피드포워드 네트워크 가중치)를 공유하도록 설계했다. 이는 모델의 깊이를 유지하면서도 파라미터 수를 획기적으로 줄이는 효과를 가져왔다.72

### 6.3  DistilBERT: 지식 증류를 통한 경량화

DistilBERT는 대규모 모델의 성능을 유지하면서 크기와 속도를 개선하기 위해 '지식 증류(Knowledge Distillation)'라는 기법을 사용했다.76

- **교사-학생 모델 구조:** 잘 훈련된 대규모 BERT를 '교사(teacher) 모델'로, 더 작고 가벼운 BERT를 '학생(student) 모델'로 설정한다. 학생 모델은 교사 모델의 예측 결과(소프트 레이블)를 모방하도록 학습함으로써, 교사 모델이 학습한 복잡하고 일반화된 지식을 전달받는다.
- **Triple Loss:** 효과적인 지식 전달을 위해 세 가지 손실 함수를 결합하여 사용했다.77
  1. **증류 손실:** 학생 모델의 출력 확률 분포가 교사 모델의 출력 확률 분포와 유사해지도록 학습한다.
  2. **MLM 손실:** 학생 모델 자체도 MLM 태스크를 수행하도록 하여 언어 이해 능력을 직접 학습한다.
  3. **코사인 거리 손실:** 학생 모델과 교사 모델의 은닉 상태 벡터의 방향이 유사해지도록 하여, 중간 표현까지 모방하도록 유도한다.

이러한 방식으로 DistilBERT는 BERT-Base 파라미터의 60% 수준으로 크기를 줄였음에도 불구하고, GLUE 벤치마크에서 BERT 성능의 97%를 유지하는 데 성공했다.78

### 6.4  ELECTRA (Efficiently Learning an Encoder that Classifies Token Replacements Accurately): 새로운 사전 훈련 방식의 제안

ELECTRA는 MLM의 비효율성에 주목하고, '대체 토큰 탐지(Replaced Token Detection, RTD)'라는 혁신적인 사전 훈련 방식을 제안했다.70

- **생성자-판별자 구조:** ELECTRA는 두 개의 모델을 함께 훈련시킨다.70
  1. **생성자(Generator):** 작은 크기의 MLM 모델로, 입력 문장의 일부 토큰을 그럴듯한 가짜 토큰으로 교체하는 역할을 한다.
  2. **판별자(Discriminator):** 주 훈련 대상인 모델로, 생성자에 의해 변형된 문장을 입력받아 각 토큰이 원래 토큰인지 아니면 생성자에 의해 대체된 가짜 토큰인지를 이진 분류 문제로 예측한다.
- **효율성:** MLM은 전체 토큰의 15%에 대해서만 손실을 계산하고 학습 신호를 얻는 반면, RTD는 모든 토큰에 대해 '진짜'인지 '가짜'인지 예측해야 하므로 모든 토큰으로부터 학습 신호를 얻는다. 이로 인해 ELECTRA는 훨씬 더 데이터 효율적(sample-efficient)이며, 동일한 계산 자원으로 BERT보다 훨씬 높은 성능을 달성할 수 있음을 보여주었다.70

아래 표는 BERT와 주요 파생 모델들의 핵심적인 차이점을 요약한 것이다.

**Table 3: 주요 BERT 파생 모델의 특징 비교**

| 모델           | 핵심 혁신                  | 주요 사전 훈련 과제            | 파라미터 효율성       |
| -------------- | -------------------------- | ------------------------------ | --------------------- |
| **BERT**       | 깊은 양방향성 구현         | MLM + NSP                      | 기준                  |
| **RoBERTa**    | 훈련 전략 최적화           | MLM (NSP 제거, 동적 마스킹)    | BERT와 유사           |
| **ALBERT**     | 파라미터 공유, 임베딩 분해 | MLM + SOP                      | 매우 높음             |
| **DistilBERT** | 지식 증류                  | MLM (교사 모델로부터 증류)     | 높음 (경량화)         |
| **ELECTRA**    | 대체 토큰 탐지             | Replaced Token Detection (RTD) | 매우 높음 (훈련 효율) |

## 7.  비판적 고찰: BERT의 한계와 사회적 책임

BERT는 NLP 분야에 기념비적인 발전을 가져왔지만, 동시에 여러 기술적 한계와 사회적, 윤리적 과제를 남겼다. 이에 대한 비판적 고찰은 향후 언어 모델 연구의 방향성을 설정하는 데 필수적이다.

### 7.1  기술적 한계

- **긴 시퀀스 처리의 어려움:** BERT의 핵심인 셀프 어텐션 메커니즘은 시퀀스 내 모든 토큰 쌍 간의 상호작용을 계산해야 하므로, 계산 복잡도가 시퀀스 길이의 제곱($O(n^2)$)에 비례한다. 이로 인해 BERT는 일반적으로 최대 512 토큰 길이의 입력만 처리할 수 있으며, 이보다 긴 문서나 대화 전체를 한 번에 이해하는 데는 구조적인 한계를 가진다.86
- **생성 능력의 부재:** BERT는 트랜스포머의 인코더 구조만을 사용하기 때문에 입력된 텍스트의 문맥을 깊이 이해하고 표현을 추출하는 데는 뛰어나지만, GPT와 같이 유창하고 창의적인 텍스트를 생성하는 능력은 없다.87 이는 BERT가 자연어 이해(NLU)에는 강력하지만 자연어 생성(NLG)에는 직접적으로 사용되기 어렵다는 것을 의미한다.
- **사전 훈련-미세 조정 불일치:** MLM 훈련 시 사용되는 `` 토큰이 미세 조정 단계에서는 나타나지 않는다는 문제는 80-10-10 마스킹 전략을 통해 일부 완화되었지만, 여전히 사전 훈련과 실제 적용 환경 간의 근본적인 불일치로 남아있다.87 이는 모델의 성능에 미묘한 영향을 미칠 수 있는 요인으로 지적된다.

### 7.2  자원 문제: 막대한 계산 비용과 환경에 미치는 영향

- **막대한 계산 비용:** BERT와 같은 대규모 언어 모델을 사전 훈련하는 데는 엄청난 양의 계산 자원이 필요하다. BERT-Base 모델을 훈련하는 데 4개의 클라우드 TPU 포드(총 16개 TPU 칩)에서 4일이 소요되었으며 14, BERT-Large는 이보다 훨씬 더 많은 자원을 필요로 한다. 이러한 비용은 소규모 연구 그룹이나 개발도상국의 연구자들이 최신 NLP 연구에 참여하는 것을 어렵게 만드는 높은 진입 장벽으로 작용한다.14
- **환경에 미치는 영향:** 대규모 모델 훈련에 소요되는 막대한 전력 소비는 상당한 양의 탄소 배출로 이어진다. 한 연구에서는 BERT 모델 하나를 훈련시키는 과정에서 발생하는 탄소 발자국이 자동차 5대의 평생 탄소 배출량과 맞먹는다고 추정했다.92 이러한 사실은 AI 기술 발전의 환경적 비용과 지속 가능성에 대한 심각한 윤리적 질문을 제기하며, 'Green AI'에 대한 논의를 촉발시켰다.94

### 7.3  사회적 문제: 훈련 데이터에 내재된 편향(Bias)의 학습 및 증폭

BERT의 가장 심각한 한계 중 하나는 사회적 편향 문제이다. 이는 기술적 결함이라기보다는, 모델의 학습 방식과 데이터의 본질에서 비롯되는 근본적인 문제이다.

BERT의 가장 큰 기술적 강점은 레이블 없는 방대한 인간의 텍스트로부터 언어의 통계적 패턴을 학습하는 능력이다. 그러나 이 강점은 동시에 가장 큰 사회적 약점의 원인이 된다. 모델이 학습하는 데이터, 즉 위키피디아, 뉴스, 서적 등은 인간 사회의 산물이며, 여기에는 성별, 인종, 직업, 종교 등에 대한 사회적 편견과 고정관념이 필연적으로 내재되어 있다.98 MLM 훈련 방식은 모델이 주변 문맥을 기반으로 단어를 예측하도록 하는데, 만약 훈련 데이터에서 '간호사'라는 단어가 '그녀(she)'라는 대명사와, '대통령'이라는 단어가 '그(he)'라는 대명사와 압도적으로 자주 함께 등장한다면, 모델은 이 통계적 연관성을 그대로 학습하게 된다. 이는 알고리즘이 의도대로 정확하게 작동한 결과이지만, 사회적으로는 유해한 편향을 재생산하고 강화하는 결과를 낳는다.

- **편향의 구체적 발현:** 실제로 "간호사는 `가 운동을 원해서 산책을 갔다"라는 문장에서 `에 들어갈 단어로 'she'를 예측할 확률이 'he'보다 압도적으로 높게 나타나는 반면, '간호사'를 '대통령'으로 바꾸면 정반대의 결과가 나타난다.99 또한, 특정 인종과 관련된 이름이나 표현이 부정적인 감성과 더 자주 연관되는 현상도 보고되었다.101
- **문제의 심각성:** 이렇게 편향을 학습한 모델이 채용 시스템에서 이력서를 평가하거나, 금융 기관에서 대출 신청을 심사하거나, 소셜 미디어에서 혐오 발언을 탐지하는 등 사회적으로 민감한 영역에 사용될 경우, 기존의 사회적 차별을 자동화하고 증폭시켜 불평등을 심화시킬 수 있다. 이는 단순히 기술적 정확성을 넘어 AI의 공정성과 윤리적 책임에 대한 근본적인 질문을 던진다. 따라서 편향 문제는 BERT를 단순한 기술적 산물이 아닌, 사회의 모습을 비추는 '거울'로 인식하고, 데이터와 모델의 상호작용을 비판적으로 성찰해야 할 필요성을 제기한다.

## 8.  결론: BERT가 남긴 유산과 미래 전망

### 9.1. BERT의 핵심 기여 요약

BERT는 자연어 처리의 역사에서 하나의 변곡점을 그은 모델로 평가된다. 그 핵심 기여는 세 가지로 요약할 수 있다. 첫째, BERT는 트랜스포머 인코더 아키텍처가 순환 신경망을 대체하여 복잡한 언어 현상을 모델링할 수 있는 강력한 기반임을 입증했다. 둘째, Masked Language Model(MLM)이라는 혁신적인 사전 훈련 목표를 통해 기존 언어 모델의 단방향성 한계를 극복하고, 모든 레이어에서 양쪽 문맥을 동시에 고려하는 '깊은 양방향 표현' 학습을 최초로 실현했다. 셋째, 이 모든 것을 바탕으로 '사전 훈련-미세 조정' 패러다임을 NLP 연구 및 개발의 표준으로 확고히 정착시켰다. 이 패러다임은 특정 태스크에 종속된 모델 개발의 비효율성을 극복하고, 범용 언어 모델이라는 개념을 현실화하며 NLP 연구의 방향을 근본적으로 바꾸어 놓았다.

### 8.1  대규모 언어 모델(LLM) 시대로의 이행 과정에서 BERT의 역할

BERT의 성공은 더 큰 모델, 더 많은 데이터, 더 강력한 컴퓨팅 파워가 더 나은 성능으로 이어진다는 '스케일링 법칙(Scaling Law)'의 강력한 초기 증거가 되었다. BERT-Large가 BERT-Base보다 작은 데이터셋에서도 우수한 성능을 보인 결과는 모델의 규모를 키우는 것이 일반화 성능을 향상시키는 효과적인 전략임을 시사했다.10 이는 이후 OpenAI의 GPT-3와 같이 수천억 개 이상의 파라미터를 가진 초거대 언어 모델(Large Language Model, LLM)의 등장으로 이어지는 직접적인 계기가 되었다. BERT가 주로 언어 '이해(Understanding)' 능력에 초점을 맞추었다면, 그로부터 영감을 받은 후속 LLM들은 BERT가 다진 견고한 기반 위에서 정교한 언어 '생성(Generation)' 능력까지 갖추게 되면서 오늘날의 생성형 AI 시대를 열었다.

### 8.2  향후 언어 모델 연구에 대한 시사점

BERT와 그 후속 연구들의 발전 과정은 미래 언어 모델 연구가 나아가야 할 방향에 중요한 시사점을 남긴다. 초기의 경쟁이 오직 성능 지표를 높이는 데 집중되었다면, RoBERTa는 훈련 방식의 최적화, ALBERT와 DistilBERT는 모델의 효율성과 경량화, ELECTRA는 훈련 패러다임의 혁신이라는 새로운 화두를 던졌다. 이는 미래의 언어 모델 연구가 단순히 모델의 규모를 키우는 것을 넘어, 제한된 자원 내에서 최고의 효율을 내는 방법, 더 빠르고 경제적으로 모델을 배포하는 방법, 그리고 더 데이터 효율적인 학습 방법을 탐구해야 함을 의미한다.

더 나아가, BERT의 편향과 환경 문제에 대한 비판적 고찰은 모델의 성능만큼이나 공정성, 투명성, 지속 가능성과 같은 사회적, 윤리적 책임이 중요함을 일깨워주었다. 따라서 향후 언어 모델 연구는 기술적 성능의 한계를 넓히는 동시에, 더 효율적이고, 더 윤리적이며, 더 신뢰할 수 있는 AI를 개발하는 방향으로 나아가야 할 것이다. BERT는 그 여정의 시작을 알린 이정표로서, 오랫동안 NLP 역사에 기록될 것이다.

#### **참고 자료**

1. 인공지능의 역사에서 BERT 이해하기, 8월 16, 2025에 액세스, https://blog.medianavi.kr/2022-01-21-history-of-ai/
2. Transformer (deep learning architecture) - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Transformer_(deep_learning_architecture)
3. How Transformers Work: A Detailed Exploration of ... - DataCamp, 8월 16, 2025에 액세스, https://www.datacamp.com/tutorial/how-transformers-work
4. BERT(Pre-training of Deep Bidirectional Transformers for Language ..., 8월 16, 2025에 액세스, https://devhwi.tistory.com/25
5. BERT 개념 정리 (특징/구조/동작 방식/종류/장점/BERT 모델 설명), 8월 16, 2025에 액세스, https://happy-obok.tistory.com/23
6. [1810.04805] BERT: Pre-training of Deep Bidirectional Transformers ..., 8월 16, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/1810.04805
7. BERT 101 - State Of The Art NLP Model Explained - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/blog/bert-101
8. A Complete Guide to BERT with Code | Towards Data Science, 8월 16, 2025에 액세스, https://towardsdatascience.com/a-complete-guide-to-bert-with-code-9f87602e4a11/
9. BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding | Request PDF - ResearchGate, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/328230984_BERT_Pre-training_of_Deep_Bidirectional_Transformers_for_Language_Understanding
10. (PDF) Bidirectional encoders to state-of-the-art: a review of BERT and its transformative impact on natural language processing - ResearchGate, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/378685185_Bidirectional_encoders_to_state-of-the-art_a_review_of_BERT_and_its_transformative_impact_on_natural_language_processing
11. 버트(BERT) 개념 간단히 이해하기, 8월 16, 2025에 액세스, https://moondol-ai.tistory.com/463
12. BERT 개념 쉽게 이해하기 - 꾸준한 성장일기, 8월 16, 2025에 액세스, https://seungseop.tistory.com/22
13. [논문 리뷰] BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding - velog, 8월 16, 2025에 액세스, [https://velog.io/@rnjsdb72/%EB%85%BC%EB%AC%B8-%EB%A6%AC%EB%B7%B0-BERT-Pre-training-of-Deep-Bidirectional-Transformers-for-Language-Understanding](https://velog.io/@rnjsdb72/논문-리뷰-BERT-Pre-training-of-Deep-Bidirectional-Transformers-for-Language-Understanding)
14. [R] BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding : r/MachineLearning - Reddit, 8월 16, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/9nfqxz/r_bert_pretraining_of_deep_bidirectional/
15. 가장 성공적인 트랜스포머의 변형: BERT와 GPT 소개 - Medium, 8월 16, 2025에 액세스, [https://medium.com/@hugmanskj/%EA%B0%80%EC%9E%A5-%EC%84%B1%EA%B3%B5%EC%A0%81%EC%9D%B8-%ED%8A%B8%EB%9E%9C%EC%8A%A4%ED%8F%AC%EB%A8%B8%EC%9D%98-%EB%B3%80%ED%98%95-bert%EC%99%80-gpt-%EC%86%8C%EA%B0%9C-0b18fb7e563b](https://medium.com/@hugmanskj/가장-성공적인-트랜스포머의-변형-bert와-gpt-소개-0b18fb7e563b)
16. What is Positional Encoding? | IBM, 8월 16, 2025에 액세스, https://www.ibm.com/think/topics/positional-encoding
17. Positional Encoding in Transformers - GeeksforGeeks, 8월 16, 2025에 액세스, https://www.geeksforgeeks.org/nlp/positional-encoding-in-transformers/
18. A Gentle Introduction to Positional Encoding in Transformer Models, Part 1 - MachineLearningMastery.com, 8월 16, 2025에 액세스, https://machinelearningmastery.com/a-gentle-introduction-to-positional-encoding-in-transformer-models-part-1/
19. What is Encoder in Transformers - Scaler Topics, 8월 16, 2025에 액세스, https://www.scaler.com/topics/nlp/transformer-encoder-decoder/
20. Understanding Transformers — Encoder | by Sathya Krishnan Suresh - Medium, 8월 16, 2025에 액세스, https://medium.com/@mr.sk12112002/understanding-transformers-encoder-1f269b1cc943
21. Understanding and Coding the Self-Attention Mechanism of Large ..., 8월 16, 2025에 액세스, https://sebastianraschka.com/blog/2023/self-attention-from-scratch.html
22. Understanding Self-Attention and Multi-Head Attention in Deep Learning - DEV Community, 8월 16, 2025에 액세스, https://dev.to/nareshnishad/understanding-self-attention-and-multi-head-attention-in-deep-learning-4jg4
23. Multi-Head Attention Mechanism - GeeksforGeeks, 8월 16, 2025에 액세스, https://www.geeksforgeeks.org/nlp/multi-head-attention-mechanism/
24. The Detailed Explanation of Self-Attention in Simple Words | by Maninder Singh | Medium, 8월 16, 2025에 액세스, https://medium.com/@manindersingh120996/the-detailed-explanation-of-self-attention-in-simple-words-dec917f83ef3
25. Self-attention and multi-head attention mechanisms | Deep Learning Systems Class Notes, 8월 16, 2025에 액세스, https://library.fiveable.me/deep-learning-systems/unit-10/self-attention-multi-head-attention-mechanisms/study-guide/ZR1kvGxLvFyRTCzg
26. LLM Transformer Model Visually Explained - Polo Club of Data Science, 8월 16, 2025에 액세스, https://poloclub.github.io/transformer-explainer/
27. Normalization and Residual Connections in Generative AI, 8월 16, 2025에 액세스, https://www.tutorialspoint.com/gen-ai/normalization-and-residual-connections.htm
28. What is Add & Norm, as quick as possible? | by Mohit Sharma - Medium, 8월 16, 2025에 액세스, https://molgorithm.medium.com/what-is-add-norm-as-soon-as-possible-178fc0836381
29. On Layer Normalizations and Residual Connections in Transformers - OpenReview, 8월 16, 2025에 액세스, https://openreview.net/attachment?id=oDDqFpK5Ru1&name=pdf
30. On Layer Normalization in the Transformer Architecture - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/pdf/2002.04745
31. Investigation of improving the pre-training and fine-tuning of BERT model for biomedical relation extraction - PMC, 8월 16, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC8978438/
32. BERT의 학습 원리 및 transferlearning - AI/ML 기술 블로그, 8월 16, 2025에 액세스, https://pangyo-datascientist.tistory.com/entry/BERT
33. arXiv:1810.04805v2 [cs.CL] 24 May 2019, 8월 16, 2025에 액세스, https://arxiv.org/pdf/1810.04805
34. [WK07-Day032][21.09.15.Wed] ELMo, GPT-1, Layer Normalization, Hugging Face Transformers 기본 사용법 - dukim's blog - 티스토리, 8월 16, 2025에 액세스, https://eliza-dukim.tistory.com/41
35. Why BERT model have to keep 10% MASK token unchanged? - Stack Overflow, 8월 16, 2025에 액세스, https://stackoverflow.com/questions/64013808/why-bert-model-have-to-keep-10-mask-token-unchanged
36. BERT MLM - 80% [MASK], 10% random words and 10% same word - how does this work?, 8월 16, 2025에 액세스, https://discuss.huggingface.co/t/bert-mlm-80-mask-10-random-words-and-10-same-word-how-does-this-work/17867
37. Understanding Masked Language Modeling as a Pretraining Task | by Era Parihar | Medium, 8월 16, 2025에 액세스, https://medium.com/@eraparihar98/understanding-masked-language-modeling-as-a-pretraining-task-4a887ec61c8d
38. BERT MLM - 80% [MASK], 10% random words and 10% same word - how does this work?, 8월 16, 2025에 액세스, https://stats.stackexchange.com/questions/575002/bert-mlm-80-mask-10-random-words-and-10-same-word-how-does-this-work
39. Next Sentence Prediction - Mue AI, 8월 16, 2025에 액세스, https://muegenai.com/docs/dsa/section-1/chapter-2-understanding-the-bert-model/next-sentence-prediction/
40. BERT Next Sentence Prediction: How to do predictions? - Hugging Face Forums, 8월 16, 2025에 액세스, https://discuss.huggingface.co/t/bert-next-sentence-prediction-how-to-do-predictions/6927
41. Pre-Training with Whole Word Masking for Chinese BERT - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/pdf/1906.08101
42. RoBERTa vs. BERT: Exploring the Evolution of Transformer Models ..., 8월 16, 2025에 액세스, https://www.dsstream.com/post/roberta-vs-bert-exploring-the-evolution-of-transformer-models
43. RoBERTa vs BERT: A Comparison of Transformer Models - DhiWise, 8월 16, 2025에 액세스, https://www.dhiwise.com/post/roberta-vs-bert-a-comprehensive-comparison
44. BERT — transformers 2.9.1 documentation - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/transformers/v2.9.1/model_doc/bert.html
45. BERT — transformers 3.0.2 documentation - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/transformers/v3.0.2/model_doc/bert.html
46. MaxPoolBERT: Enhancing BERT Classification via Layer- and Token-Wise Aggregation, 8월 16, 2025에 액세스, https://arxiv.org/html/2505.15696v1
47. Understanding the [CLS] Token in BERT: A Comprehensive Guide ..., 8월 16, 2025에 액세스, https://aditya007.medium.com/understanding-the-cls-token-in-bert-a-comprehensive-guide-a62b3b94a941
48. What is purpose of the [CLS] token and why is its encoding output important?, 8월 16, 2025에 액세스, https://datascience.stackexchange.com/questions/66207/what-is-purpose-of-the-cls-token-and-why-is-its-encoding-output-important
49. [PDF] BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding, 8월 16, 2025에 액세스, https://www.semanticscholar.org/paper/BERT%3A-Pre-training-of-Deep-Bidirectional-for-Devlin-Chang/df2b0e26d0599ce3e70df8a9da02e51594e0e992
50. Named Entity Recognition Bert - NVIDIA NGC, 8월 16, 2025에 액세스, https://catalog.ngc.nvidia.com/orgs/nvidia/teams/tao/models/namedentityrecognition_english_bert
51. Fine-Tuning BERT for Named Entity Recognition (NER) | by why amit - Medium, 8월 16, 2025에 액세스, https://medium.com/@whyamit101/fine-tuning-bert-for-named-entity-recognition-ner-b42bcf55b51d
52. How to Do Named Entity Recognition (NER) with a BERT Model ..., 8월 16, 2025에 액세스, https://machinelearningmastery.com/how-to-do-named-entity-recognition-ner-with-a-bert-model/
53. Question Answering with a Fine-Tuned BERT.ipynb - Colab, 8월 16, 2025에 액세스, https://colab.research.google.com/drive/1uSlWtJdZmLrI3FCNIlUHFxwAJiSu2J0-
54. Question-Answering using Bert - Kaggle, 8월 16, 2025에 액세스, https://www.kaggle.com/code/arunmohan003/question-answering-using-bert
55. 16.6. Fine-Tuning BERT for Sequence-Level and Token-Level Applications - Dive into Deep Learning, 8월 16, 2025에 액세스, https://d2l.ai/chapter_natural-language-processing-applications/finetuning-bert.html
56. BERT for Question Answering on SQuAD 2.0, 8월 16, 2025에 액세스, https://web.stanford.edu/class/archive/cs/cs224n/cs224n.1194/reports/default/15848021.pdf
57. How to Train A Question-Answering Machine Learning Model (BERT) - DigitalOcean, 8월 16, 2025에 액세스, https://www.digitalocean.com/community/tutorials/how-to-train-question-answering-machine-learning-models
58. Question answering - Hugging Face LLM Course, 8월 16, 2025에 액세스, https://huggingface.co/learn/llm-course/chapter7/7
59. [자연어 처리] 4. BERT, 8월 16, 2025에 액세스, https://hcnoh.github.io/2024-01-02-nlp-04
60. BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding (2018) | Jacob Devlin | 28110 Citations - SciSpace, 8월 16, 2025에 액세스, https://scispace.com/papers/bert-pre-training-of-deep-bidirectional-transformers-for-3mez6kncl2
61. 6 Natural Language Processing Models you should know - UBIAI, 8월 16, 2025에 액세스, https://ubiai.tools/6-natural-language-processing-models-you-should-know/
62. The BERT Algorithm (NLP) Made Simple [Understand How Large Language Models (LLMs) Work] - Spot Intelligence, 8월 16, 2025에 액세스, https://spotintelligence.com/2024/02/29/bert-nlp/
63. Understanding BERT: The Model That Revolutionized NLP | by Zahra Waleed - Medium, 8월 16, 2025에 액세스, https://medium.com/@ms.scales1480/understanding-bert-the-model-that-revolutionized-nlp-ec6ba6bde43f
64. How Google's BERT Changed Natural Language Understanding | Brave River Solutions, 8월 16, 2025에 액세스, https://www.braveriver.com/blog/how-googles-bert-changed-natural-language-understanding/
65. FAQ: All about the BERT algorithm in Google search, 8월 16, 2025에 액세스, https://searchengineland.com/faq-all-about-the-bert-algorithm-in-google-search-324193
66. What Is Google's BERT and Why Does It Matter? | NVIDIA Glossary, 8월 16, 2025에 액세스, https://www.nvidia.com/en-us/glossary/bert/
67. Understanding BERT: The Revolution in Natural Language Processing - interScan LLC, 8월 16, 2025에 액세스, https://www.interscanllc.com/blog/understanding-bert-the-revolution-in-natural-language-processing
68. BERT: The Start of Modern Natural Language Processing - Multimodal, 8월 16, 2025에 액세스, https://www.multimodal.dev/post/bert-the-start-of-modern-nlp
69. The Ultimate RoBERTa Guide for NLP - Number Analytics, 8월 16, 2025에 액세스, https://www.numberanalytics.com/blog/ultimate-roberta-guide-for-nlp
70. ELECTRA: PRE-TRAINING TEXT ENCODERS - OpenReview, 8월 16, 2025에 액세스, https://openreview.net/pdf?id=r1xMH1BtvB
71. Introducing RoBERTa Base Model: A Comprehensive Overview | by Novita AI - Medium, 8월 16, 2025에 액세스, https://medium.com/@marketing_novita.ai/introducing-roberta-base-model-a-comprehensive-overview-330338afa082
72. ALBERT - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/docs/transformers/model_doc/albert
73. Review - ALBERT: A Lite BERT for Self-supervised Learning of Language Representations, 8월 16, 2025에 액세스, https://www.assemblyai.com/blog/review-albert
74. machine-learning-articles/albert-explained-a-lite-bert.md at main - GitHub, 8월 16, 2025에 액세스, https://github.com/christianversloot/machine-learning-articles/blob/main/albert-explained-a-lite-bert.md
75. ALBERT: A Lite BERT for Self-supervised Learning of Language Representations, 8월 16, 2025에 액세스, https://openreview.net/forum?id=H1eA7AEtvS
76. DistilBERT - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/docs/transformers/model_doc/distilbert
77. Review — DistilBERT, a distilled version of BERT: smaller, faster, cheaper and lighter, 8월 16, 2025에 액세스, https://sh-tsang.medium.com/review-distilbert-a-distilled-version-of-bert-smaller-faster-cheaper-and-lighter-5b3fa180169e
78. Distilbert: A Smaller, Faster, and Distilled BERT - Zilliz Learn, 8월 16, 2025에 액세스, https://zilliz.com/learn/distilbert-distilled-version-of-bert
79. Understanding Distil BERT In Depth | by Arun Mohan - Medium, 8월 16, 2025에 액세스, https://arunm8489.medium.com/understanding-distil-bert-in-depth-5f2ca92cf1ed
80. [1910.01108] DistilBERT, a distilled version of BERT: smaller, faster ..., 8월 16, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/1910.01108
81. DistilBERT, a distilled version of BERT: smaller, faster, cheaper and lighter, 8월 16, 2025에 액세스, https://www.semanticscholar.org/paper/DistilBERT%2C-a-distilled-version-of-BERT%3A-smaller%2C-Sanh-Debut/a54b56af24bb4873ed0163b77df63b92bd018ddc
82. DistilBERT, a distilled version of BERT: smaller, faster, cheaper and lighter - ResearchGate, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/336230729_DistilBERT_a_distilled_version_of_BERT_smaller_faster_cheaper_and_lighter
83. Exploring ELECTRA - Efficient Pre-training for Transformers - DEV Community, 8월 16, 2025에 액세스, https://dev.to/nareshnishad/exploring-electra-efficient-pre-training-for-transformers-o7j
84. google-research/electra: ELECTRA: Pre-training Text Encoders as Discriminators Rather Than Generators - GitHub, 8월 16, 2025에 액세스, https://github.com/google-research/electra
85. ELECTRA - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/docs/transformers/model_doc/electra
86. What are the challenges associated with BERT model? - BytePlus, 8월 16, 2025에 액세스, https://www.byteplus.com/en/topic/493988
87. BERT: A Comprehensive Introduction | by Shirley Li - Medium, 8월 16, 2025에 액세스, https://medium.com/@lixue421/bert-a-comprehensive-introduction-7d620efcd32f
88. BERT: Pre-training of Deep Bidirectional Transformers for Language ..., 8월 16, 2025에 액세스, https://blog.paperspace.com/bert-pre-training-of-deep-bidirectional-transformers-for-language-understanding/
89. BERT (language model) - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/BERT_(language_model)
90. www.databricks.com, 8월 16, 2025에 액세스, [https://www.databricks.com/blog/mosaicbert#:~:text=The%20pretraining%20stage%20for%20BERT,%2D%24400%20%5BIzsak%20et%20al.](https://www.databricks.com/blog/mosaicbert#:~:text=The pretraining stage for BERT,-%24400 [Izsak et al.)
91. What is the cost of training large language models? - CUDO Compute, 8월 16, 2025에 액세스, https://www.cudocompute.com/blog/what-is-the-cost-of-training-large-language-models
92. klizosolutions.medium.com, 8월 16, 2025에 액세스, [https://klizosolutions.medium.com/carbon-footprint-of-ai-the-environmental-cost-of-training-llms-13b3688d7b7e#:~:text=The%20carbon%20footprint%20of%20AI%20highlights%20the%20environmental%20toll%20that,cars%20over%20their%20entire%20lifespan.](https://klizosolutions.medium.com/carbon-footprint-of-ai-the-environmental-cost-of-training-llms-13b3688d7b7e#:~:text=The carbon footprint of AI highlights the environmental toll that,cars over their entire lifespan.)
93. Carbon Footprint of AI: The Environmental Cost of Training LLMs - Klizo Solutions Pvt. Ltd., 8월 16, 2025에 액세스, https://klizosolutions.medium.com/carbon-footprint-of-ai-the-environmental-cost-of-training-llms-13b3688d7b7e
94. The Carbon Footprint of Large Language Models: Unmasking the Environmental Impact, 8월 16, 2025에 액세스, https://scientiamag.org/the-carbon-footprint-of-large-language-models-unmasking-the-environmental-impact/
95. towards measuring and mitigating the environmental impacts of large language models | cifar, 8월 16, 2025에 액세스, https://cifar.ca/wp-content/uploads/2023/09/Towards-Measuring-and-Mitigating-the-Environmental-Impacts-of-Large-Language-Models.pdf
96. 3.3: Case Study- The Carbon Cost of Training Large Language Models, 8월 16, 2025에 액세스, https://socialsci.libretexts.org/Bookshelves/Education_and_Professional_Development/Teaching_AI_Ethics%3A_Practical_Strategies_for_Discussing_AI_Ethics_in_K-12_and_Tertiary_Education_(Furze)/03%3A_Teaching_AI_Ethics-_Environment/3.03%3A_Case_Study-_The_Carbon_Cost_of_Training_Large_Language_Models
97. Environmental impact | CS324, 8월 16, 2025에 액세스, https://stanford-cs324.github.io/winter2022/lectures/environment/
98. Gender Bias in BERT - Measuring and Analysing Biases through ..., 8월 16, 2025에 액세스, https://aclanthology.org/2022.gebnlp-1.20/
99. Showing Bias in BERT - Dan McCreary, 8월 16, 2025에 액세스, https://dmccreary.medium.com/showing-bias-in-bert-475e98dabf51
100. METHODS TO QUANTIFY AND REDUCE BIAS IN BERT TRANSFORMER MODELS - Reece Shuttleworth, 8월 16, 2025에 액세스, https://reeceshuttle.me/assets/6.3950-Final-Project-Report.pdf
101. The Dark Side of Language Models: Exploring the Challenges of Bias in LLMs, 8월 16, 2025에 액세스, https://www.appypieagents.ai/blog/case-studies-instances-of-bias-in-llms
102. Hate speech detection and racial bias mitigation in social media based on BERT model, 8월 16, 2025에 액세스, https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0237861


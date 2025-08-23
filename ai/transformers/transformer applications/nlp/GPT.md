# 생성형 사전 훈련 트랜스포머(GPT)에 대한 심층 고찰


생성형 사전 훈련 트랜스포머(Generative Pre-trained Transformer, GPT)는 현대 인공지능 분야, 특히 자연어 처리(NLP)에서 혁명적인 변화를 이끌어낸 기술이다. GPT의 핵심은 '생성형 사전 훈련(Generative Pre-training)'이라는 학습 방법론과 '트랜스포머(Transformer)'라는 신경망 아키텍처의 결합에 있다.1 이 기술의 근간이 되는 트랜스포머 아키텍처는 2017년 Google 연구진이 발표한 "Attention Is All You Need"라는 기념비적인 논문에서 처음 제안되었다.1

GPT의 등장은 기존 NLP 패러다임의 근본적인 전환을 의미한다. 과거의 모델들은 주로 순환 신경망(Recurrent Neural Networks, RNN)이나 컨볼루션 신경망(Convolutional Neural Networks, CNN)에 의존하여 순차적인 데이터를 처리했다. 그러나 이러한 구조는 데이터의 순서에 따라 순차적으로 계산해야 하므로 병렬 처리가 어렵고, 문장 내 단어들 간의 거리가 멀어질수록 관계를 파악하기 힘든 '장기 의존성 문제(long-term dependency problem)'를 내포하고 있었다.3 GPT가 채택한 트랜스포머 아키텍처는 이러한 순환(recurrence) 및 컨볼루션 구조를 완전히 배제하고, 오직 '어텐션 메커니즘(attention mechanism)'에만 의존한다. 이를 통해 모델은 시퀀스 내 모든 단어 쌍의 관계를 동시에 계산할 수 있게 되어 병렬 처리를 극대화하고, 장거리 의존성 포착 능력을 혁신적으로 개선하였다.2

GPT의 성공은 단일 기술의 혁신이 아닌, 세 가지 핵심 아이디어의 유기적인 결합에 기인한다. 첫째, 병렬 처리에 최적화된 트랜스포머 아키텍처는 이전에는 불가능했던 규모의 모델 훈련을 현실화했다. 둘째, 레이블이 없는 방대한 양의 텍스트 데이터를 활용하는 '생성형 사전 훈련' 방식은 거대 모델을 훈련시키는 데 필요한 데이터 병목 현상을 해결했다.1 모델은 '다음 단어 예측'이라는 단순한 목표를 통해 언어의 복잡한 통사적, 의미적 구조를 스스로 학습한다. 마지막으로, '미세 조정(fine-tuning)'은 이렇게 형성된 보편적인 언어 지식을 소량의 특정 작업 데이터에 효율적으로 전이시켜 높은 실용성을 확보하는 길을 열었다.7 이 세 가지 요소가 서로 맞물리면서 GPT는 이전 모델들이 넘지 못했던 스케일의 장벽을 돌파하고 새로운 가능성을 열었다.

본 보고서는 GPT의 기술적 근간을 이루는 트랜스포머 아키텍처의 핵심 원리부터 시작하여, GPT 모델의 고유한 학습 전략, 세대를 거듭하며 이루어진 진화 과정, 그리고 현재 직면한 기술적 한계와 미래 전망까지 다각도로 심층 고찰하는 것을 목표로 한다.


GPT의 모든 버전은 Vaswani 등이 2017년에 제안한 트랜스포머 아키텍처에 뿌리를 두고 있다.2 이 아키텍처는 기존의 순차적 데이터 처리 방식에서 벗어나, 문장 내 모든 단어 간의 관계를 한 번에 파악하는 혁신적인 접근법을 제시했다.


트랜스포머 이전의 NLP 모델들은 주로 RNN이나 LSTM(Long Short-Term Memory)을 사용하여 문장을 단어 순서대로 처리했다. 하지만 이 방식은 문장이 길어질수록 앞부분의 정보가 뒤로 전달되면서 희석되는 장기 의존성 문제를 해결하기 어려웠다. 이를 보완하기 위해 2014년 Bahdanau 등이 제안한 어텐션 메커니즘은 디코더가 출력 단어를 생성할 때마다 인코더의 모든 입력 단어를 다시 참조하여, 현재 시점과 가장 관련이 높은 단어에 '집중(attention)'하도록 하는 아이디어였다.2

"Attention Is All You Need" 논문은 여기서 한 걸음 더 나아가, "어텐션만으로 충분하다"는 과감한 주장을 펼쳤다.2 즉, 순환 구조를 완전히 제거하고 어텐션 메커니즘만으로 입력 시퀀스 내의 관계와 출력 시퀀스를 모델링할 수 있음을 입증한 것이다.4 이 아이디어는 NLP 모델링의 근본적인 변화를 촉발했으며, GPT 아키텍처의 핵심 철학이 되었다.


셀프 어텐션은 트랜스포머의 가장 핵심적인 구성 요소로, '하나의 시퀀스 내에서 서로 다른 위치의 토큰들 간의 관계를 계산하여 시퀀스 자체의 표현(representation)을 풍부하게 만드는 메커니즘'으로 정의할 수 있다.4 이는 문장 내에서 한 단어가 다른 모든 단어와 어떻게 상호작용하는지를 직접적으로 모델링한다.

작동 과정은 다음과 같다. 먼저, 입력 시퀀스의 각 토큰 임베딩 벡터는 세 개의 서로 다른 학습 가능한 가중치 행렬( `$W^Q$`, `$W^K$`, `$W^V$`)과 곱해져 각각 쿼리(Query, Q), 키(Key, K), 밸류(Value, V)라는 세 개의 벡터를 생성한다.12 쿼리는 현재 토큰이 다른 토큰들에게 던지는 '질문'으로, 키는 다른 토큰들이 자신을 설명하는 '꼬리표'로, 밸류는 해당 토큰이 실제로 담고 있는 '정보'로 비유할 수 있다.

이 Q, K, V 벡터들을 이용한 'Scaled Dot-Product Attention' 계산은 다음과 같은 단계로 이루어진다.2

1. 특정 쿼리 벡터를 모든 키 벡터와 내적(dot product)하여 유사도 점수(score) 행렬을 계산한다 (`$QK^T$`). 이 점수는 각 단어 쌍이 얼마나 밀접하게 관련되어 있는지를 나타낸다.
2. 계산된 점수를 키 벡터의 차원(`$d_k$`)의 제곱근(`$\sqrt{d_k}$`)으로 나누어 스케일링한다. 이는 `$d_k$` 값이 클 때 내적 값이 너무 커져 소프트맥스 함수의 그래디언트가 0에 가까워지는 것을 방지하여 학습 안정성을 높이는 역할을 한다.2
3. 스케일링된 점수 행렬에 소프트맥스(softmax) 함수를 적용하여 합이 1이 되는 어텐션 가중치(attention weights)를 얻는다. 이 가중치는 각 단어가 다른 모든 단어에 얼마나 '주의'를 기울여야 하는지를 나타내는 확률 분포이다.
4. 마지막으로 이 어텐션 가중치를 각 단어의 밸류 벡터에 곱하여 가중합(weighted sum)을 구한다. 이를 통해 현재 단어와 관련성이 높은 단어들의 정보는 더 많이 반영하고, 관련성이 낮은 단어들의 정보는 적게 반영한 새로운 표현 벡터가 생성된다.

이 과정을 수식으로 표현하면 다음과 같다.2
$$
Attention(Q, K, V) = softmax(\frac{QK^T}{\sqrt{d_k}})V
$$
셀프 어텐션의 계산 복잡도는 시퀀스 길이를 `$n$`, 임베딩 차원을 `$d$`라고 할 때 `$O(n^2d)$`이다.15 이 이차 복잡도는 트랜스포머가 긴 시퀀스를 처리하는 데 있어 가장 큰 계산적 병목을 유발하는 원인이다. 시퀀스 길이가 두 배가 되면 계산량은 네 배로 증가하여 메모리 사용량과 처리 시간이 기하급수적으로 늘어난다.3 하지만 동시에, 이 구조는 시퀀스 내 모든 토큰 쌍 간의 직접적인 상호작용 경로를 제공함으로써 전례 없는 수준의 문맥 이해 능력을 부여한다. RNN이 첫 번째 토큰과 

`$n$`번째 토큰의 정보를 연결하기 위해 `$n-1$` 단계의 순차적 계산을 거쳐야 하는 반면, 셀프 어텐션은 단 한 번의 행렬 곱셈으로 이 관계를 포착한다. 이 '전역적 수용장(global receptive field)'이 바로 GPT가 복잡한 문법 구조와 장거리 의미론적 의존성을 파악하는 능력의 핵심이다.16 결국, 계산적 비효율성은 모델의 강력한 성능을 위한 일종의 '필요악'이며, 현대 LLM 연구의 주요 과제 중 하나는 이 이차 복잡도를 유지하면서 성능 저하를 최소화하는 효율적인 어텐션 변형을 찾는 것이다.3


단일 셀프 어텐션은 한 가지 관점에서만 단어 간의 관계를 파악하는 한계가 있다. 예를 들어, 한 어텐션은 문법적 관계에 집중하고 다른 어텐션은 의미적 관계에 집중하는 것이 더 효과적일 수 있다. 멀티-헤드 어텐션은 이러한 아이디어에 기반하여, Q, K, V 벡터를 여러 개의 작은 차원으로 나누어(h개의 '헤드'), 각각에 대해 독립적으로 어텐션을 병렬 수행한다.2

각 헤드는 서로 다른 선형 변환을 통해 원본 Q, K, V를 서로 다른 표현 부분 공간(representation subspace)으로 투영한다. 이를 통해 모델은 '주어-동사 관계', '수식 관계', '동의어 관계' 등 다양한 종류의 관계를 동시에 학습할 수 있다. 각 헤드에서 계산된 어텐션 출력 벡터들은 다시 하나로 결합(concatenate)된 후, 최종적으로 또 다른 학습 가능한 가중치 행렬(`$W^O$`)과 곱해져 최종 출력 벡터를 생성한다.

멀티-헤드 어텐션의 수식은 다음과 같다.14
$$
MultiHead(Q, K, V) = Concat(head_1,..., head_h)W^O
$$

$$
where\ head_i = Attention(QW_i^Q, KW_i^K, VW_i^V)
$$


트랜스포머의 셀프 어텐션 메커니즘은 모든 단어 쌍을 동시에 고려하기 때문에, 본질적으로 단어의 순서 정보를 담고 있지 않다. 즉, "나는 너를 사랑해"와 "너는 나를 사랑해"를 단어의 집합(set)으로만 인식하여 구분하지 못하는 문제가 발생한다.10 이를 해결하기 위해, 각 토큰의 위치 정보를 담은 벡터인 '포지셔널 인코딩'을 입력 임베딩에 더해줌으로써 모델에 순서 정보를 명시적으로 제공해야 한다.16

포지셔널 인코딩 방식은 다양하게 발전해왔다.

- **절대 위치 인코딩 (Absolute PE)**: "Attention Is All You Need" 논문에서 제안된 초기 방식으로, 서로 다른 주기를 가진 사인(sine)과 코사인(cosine) 함수를 사용하여 각 위치에 고유한 벡터 값을 부여한다. 이 방식은 모델이 상대적인 위치 관계를 학습하는 데 도움을 주도록 설계되었다.16
- **상대 위치 인코딩 (Relative PE)**: 두 토큰 간의 절대적인 위치가 아닌, 상대적인 거리 정보를 어텐션 계산 과정에 직접 통합하는 방식이다. 이는 모델의 일반화 성능을 향상시키는 것으로 알려져 있다.20
- **학습 가능한 인코딩 (Learnable PE)**: 위치 정보를 고정된 함수로 정의하지 않고, 모델의 다른 파라미터처럼 학습 과정에서 데이터로부터 최적의 위치 표현을 스스로 찾도록 하는 방식이다. 특히 복잡한 관계 추론(relational reasoning)이 필요한 작업에서 다른 방식들보다 우수한 성능을 보이는 경향이 있다.21
- **동적 및 하이브리드 방식**: 최근에는 입력 텍스트의 내용에 따라 위치 임베딩을 동적으로 생성하거나 18, 절대 위치와 상대 위치 정보를 결합하는 하이브리드 접근법도 연구되고 있다.16

포지셔널 인코딩의 발전 과정은 딥러닝의 거시적인 트렌드, 즉 '인간이 설계한 규칙'에서 '데이터 기반 학습'으로의 전환을 잘 보여준다. 초기 사인/코사인 함수는 합리적인 공학적 설계에 기반한 귀납 편향(inductive bias)이었으나, 모델의 능력이 커지고 해결해야 할 문제가 복잡해짐에 따라, 위치 정보 자체를 고정된 규칙으로 주입하기보다는 데이터로부터 스스로 학습하게 하는 것이 더 유연하고 효과적이라는 방향으로 진화하고 있다. 이는 모델 설계의 초점이 '어떻게 위치를 알려줄 것인가?'에서 '모델이 어떻게 위치를 스스로 학습하게 할 것인가?'로 이동했음을 의미한다.


GPT의 성공은 트랜스포머 아키텍처뿐만 아니라, 대규모 데이터를 효과적으로 활용하는 독창적인 3단계 학습 전략에 크게 의존한다. 이 전략은 '일반화'에서 '특수화', 그리고 '정렬'로 이어지는 계층적 지식 형성 과정으로 볼 수 있으며, 각 단계는 이전 단계의 결과물 위에 구축되어 점차 더 추상적이고 다루기 어려운 목표를 달성하도록 설계되었다.


사전 훈련은 GPT 학습의 첫 번째이자 가장 중요한 단계로, 대규모의 레이블 없는 텍스트 코퍼스(corpus)를 사용하여 언어 자체에 대한 보편적인 이해를 구축하는 과정이다.6 이 단계의 목표는 특정 작업에 국한되지 않는, 문법, 의미, 상식, 추론 능력 등 광범위한 '기초 지식'을 모델에 내재화하는 것이다.

GPT의 사전 훈련은 '생성형' 모델링 방식을 따른다. 구체적으로, 모델은 주어진 이전 단어들(컨텍스트)을 바탕으로 다음에 올 단어를 예측하도록 학습된다. 이는 표준적인 언어 모델링 목표 함수를 통해 이루어지며, 모델은 전체 코퍼스에 대해 다음 토큰의 조건부 확률을 최대화하도록 파라미터를 업데이트한다.24 GPT-1 논문에서 제시된 이 목표 함수는 다음과 같다.
$$
L_1(\mathcal{U}) = \sum_i \log P(u_i | u_{i-k}, \dots, u_{i-1}; \Theta)
$$
여기서 `$\mathcal{U}$`는 레이블이 없는 텍스트 코퍼스, `$k$`는 컨텍스트 윈도우의 크기, `$\Theta$`는 신경망의 파라미터를 나타낸다. 이 비지도 학습 방식은 레이블링 비용 없이 인터넷에 존재하는 방대한 양의 텍스트 데이터를 활용할 수 있게 해준다는 점에서 매우 강력하다. GPT-1은 BookCorpus 8, GPT-2는 WebText 25, GPT-3는 Common Crawl을 포함한 거대 데이터셋 26을 사용하여 학습되었으며, 이 방대하고 다양한 데이터는 모델의 뛰어난 일반화 성능의 기반이 되었다.


사전 훈련을 통해 광범위한 언어 지식을 습득한 모델은, 두 번째 단계인 미세 조정을 통해 특정 다운스트림 태스크(downstream task)에 맞게 특화된다. 미세 조정은 사전 훈련된 모델의 가중치를 초기값으로 사용하여, 감성 분석이나 기계 번역과 같이 특정 목적을 위해 구축된 소규모의 레이블된 데이터셋으로 모델을 추가 학습시키는 과정이다.6 이는 거대한 지식 기반을 특정 문제 해결에 맞게 '조율'하는 과정으로, 전이 학습(Transfer Learning)의 대표적인 예이다.7

미세 조정 단계에서는 해당 태스크에 맞는 손실 함수를 최소화하도록 모델의 파라미터를 업데이트한다. 예를 들어, 텍스트 분류 문제의 경우, 모델의 마지막 트랜스포머 블록 위에 간단한 선형 레이어와 소프트맥스 함수를 추가하여 레이블을 예측하고, cross-entropy 손실을 최소화한다. GPT-1 논문에서는 일반화 성능을 잃지 않기 위해, 미세 조정 시에도 원래의 언어 모델링 손실(`$L_1$`)을 보조 목표(auxiliary objective)로 함께 사용하는 방식을 제안했다.24 최종 목표 함수는 다음과 같이 표현될 수 있다.
$$
L_2(\mathcal{C}) = \sum_{(x,y)} \log P(y | x_1, \dots, x_m)
$$

$$
L_3(\mathcal{C}) = L_2(\mathcal{C}) + \lambda \cdot L_1(\mathcal{C})
$$

여기서 `$\mathcal{C}$`는 레이블된 데이터셋, `$(x,y)$`는 입력 시퀀스와 정답 레이블 쌍을 의미한다. 미세 조정의 가장 큰 장점은 처음부터 모델을 학습시키는 것(training from scratch)에 비해 훨씬 적은 데이터와 컴퓨팅 자원으로도 매우 높은 성능을 달성할 수 있다는 점이다.7


사전 훈련과 미세 조정만으로는 모델이 인간의 복잡하고 미묘한 가치관이나 선호를 완벽하게 따르도록 만들기 어렵다. 예를 들어, 사실이지만 무례한 답변보다는 다소 부정확하더라도 유용하고 안전한 답변이 더 선호될 수 있다. 이처럼 모델의 출력을 인간의 의도 및 가치와 일치시키는 과정을 '정렬(Alignment)'이라고 하며, 이를 위해 도입된 핵심 기술이 바로 인간 피드백 기반 강화 학습(Reinforcement Learning from Human Feedback, RLHF)이다.27

RLHF는 InstructGPT와 ChatGPT의 성공에 결정적인 역할을 했으며 1, 크게 3단계로 구성된다.

1. **지도 미세 조정 (SFT) 모델 준비**: 먼저, 인간이 작성한 시연 데이터(demonstration data)를 사용하여 사전 훈련된 언어 모델을 지도 미세 조정한다. 이를 통해 모델은 사용자의 다양한 지시에 대해 양질의 응답을 생성하는 초기 능력을 갖추게 된다.27
2. **보상 모델 (Reward Model) 훈련**: SFT 모델에게 하나의 프롬프트를 주고 여러 개의 다른 응답을 생성하게 한다. 그 후, 인간 평가자가 어떤 응답이 더 나은지 선호도에 따라 순위를 매긴다. 이 비교 데이터를 사용하여, 주어진 프롬프트와 응답 쌍에 대해 인간이 얼마나 선호할지를 점수(보상)로 예측하는 '보상 모델'을 훈련시킨다.29
3. **강화 학습을 통한 정책 최적화**: 마지막으로, SFT 모델을 강화 학습의 '정책(policy)'으로 간주하고, 보상 모델이 예측하는 점수를 '보상(reward)' 신호로 사용하여 이 정책을 최적화한다. Proximal Policy Optimization (PPO)과 같은 알고리즘을 사용하여, 모델은 보상 모델로부터 더 높은 점수를 받을 수 있는 방향으로 응답을 생성하도록 파라미터를 업데이트한다.28

이 과정을 통해 모델은 단순히 정답을 맞추는 것을 넘어, 인간이 '유용하고(helpful)', '진실하며(honest)', '무해한(harmless)' 것으로 판단하는 규범적 기준에 부합하는 응답을 생성하도록 학습된다.31 RLHF의 등장은 LLM 개발의 초점이 '무엇을 할 수 있는가'에서 '무엇을, 그리고 어떻게 해야 하는가'라는 가치 판단의 영역으로 이동하고 있음을 명확히 보여준다.


GPT 시리즈의 역사는 모델의 크기, 학습 데이터의 양, 그리고 계산 능력이 증가함에 따라 모델의 성능과 능력이 질적으로 변화하는 '규모의 법칙(Scaling Law)'을 극적으로 보여주는 사례이다. GPT-1부터 GPT-4에 이르기까지, 각 세대는 이전 모델의 한계를 극복하며 새로운 차원의 능력을 선보였다.

아래 표는 GPT 모델들의 진화 과정을 핵심 지표별로 요약한 것이다. 이 표를 통해 파라미터 수의 기하급수적인 증가와 함께, 모델의 핵심 능력이 '패러다임 증명'에서 '제로샷', '퓨샷 학습', 그리고 '멀티모달리티'로 질적으로 도약했음을 한눈에 파악할 수 있다.

| 모델 (Model) | 파라미터 수 (Parameters)   | 학습 데이터 (Training Data)                                  | 주요 특징 (Key Features)                                     |
| ------------ | -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| GPT-1        | 1.17억 (117 Million)       | BookCorpus (약 4.5GB)                                        | 생성형 사전 훈련 + 미세 조정 패러다임의 효용성 입증 32       |
| GPT-2        | 15억 (1.5 Billion)         | WebText (40GB)                                               | 모델 및 데이터 스케일업, 제로샷(Zero-shot) 태스크 수행 능력 발현 32 |
| GPT-3        | 1750억 (175 Billion)       | Common Crawl, WebText2, Books, Wikipedia (총 570GB, 약 3000억 토큰) | 압도적인 스케일, 퓨샷(Few-shot) 및 인-컨텍스트 학습(In-context learning) 능력 32 |
| GPT-4        | (비공개, 5000억 이상 추정) | (비공개, 대폭 확장)                                          | 멀티모달 입력(텍스트, 이미지), 향상된 추론 능력, 환각 감소 32 |


2018년에 발표된 GPT-1은 1.17억 개의 파라미터와 12개의 트랜스포머 디코더 레이어로 구성되었다.8 이 모델의 가장 큰 기여는 '생성형 사전 훈련-미세 조정'이라는 새로운 패러다임의 잠재력을 실증적으로 입증한 데 있다. 레이블이 없는 대규모 텍스트 데이터인 BookCorpus로 사전 훈련한 후, 자연어 추론, 질의응답, 문장 분류 등 다양한 NLP 벤치마크 데이터셋에 미세 조정을 적용했다. 그 결과, 각 태스크에 특화되어 설계된 기존의 여러 지도 학습 모델들의 성능을 뛰어넘으며, 범용적인 사전 훈련 모델이 다양한 다운스트림 태스크에 효과적으로 전이될 수 있음을 보여주었다.8


GPT-2는 모델의 규모를 대폭 확장하는 전략을 채택했다. 파라미터 수를 15억 개로 10배 이상 늘리고, 48개의 트랜스포머 레이어를 사용했다. 학습 데이터 역시 Reddit에서 높은 평가를 받은 외부 링크로 구성된 40GB 규모의 고품질 데이터셋 'WebText'를 구축하여 사용했다.25 GPT-2는 이러한 스케일업을 통해 놀라운 능력을 선보였는데, 바로 별도의 미세 조정 없이 프롬프트만으로 다양한 작업을 수행하는 '제로샷(zero-shot)' 학습 능력이었다. 예를 들어, "Translate to French:"라는 프롬프트를 주고 영문 텍스트를 입력하면 프랑스어로 번역하는 식이었다. 이는 모델의 규모가 충분히 커지면 데이터에 내재된 다양한 패턴을 학습하여 암시적으로 여러 작업을 수행할 수 있게 됨을 시사하는 중요한 발견이었다.33


GPT-3는 1750억 개의 파라미터와 96개의 레이어라는, 당시로서는 상상하기 어려운 규모로 다시 한번 도약했다.26 이 압도적인 스케일은 모델의 학습 방식에 질적인 변화를 가져왔다. GPT-3는 '인-컨텍스트 학습(in-context learning)' 또는 '퓨샷(few-shot) 학습'이라는 새로운 패러다임을 제시했다. 이는 모델의 가중치를 직접 업데이트하는 미세 조정 없이, 프롬프트에 해당 작업에 대한 간단한 설명과 몇 개의 예시('shots')를 제공하는 것만으로 모델이 새로운 작업을 수행하는 능력을 의미한다.32 예를 들어, "바다: 파랗다, 잔디:?"라는 예시를 주면 모델이 "초록색"이라고 답하는 식이다. 이는 모델이 추론 시점에 제공된 정보만으로도 일종의 '메타-러닝(meta-learning)'을 수행하여 새로운 작업에 빠르게 적응할 수 있음을 보여준 혁신적인 발견이었다. 이로 인해 LLM은 단순한 '모델'을 넘어, 일종의 '범용 학습 플랫폼'으로서의 가능성을 열게 되었다.


2023년에 공개된 GPT-4는 정확한 파라미터 수를 공개하지 않았지만, 이전 모델보다 훨씬 더 큰 규모로 추정된다. GPT-4의 가장 큰 혁신은 텍스트뿐만 아니라 이미지도 입력으로 처리할 수 있는 '멀티모달(multi-modal)' 능력을 갖춘 것이다.33 이를 통해 사용자는 이미지에 대한 질문을 하거나 이미지의 내용을 분석하도록 요청할 수 있게 되었다. 또한, 미국 변호사 시험이나 생물 올림피아드와 같은 전문적인 시험에서 상위권의 성적을 기록하며, 복잡한 추론 능력이 비약적으로 향상되었음을 입증했다. OpenAI는 GPT-4 개발 과정에서 RLHF를 포함한 정렬 기술을 더욱 고도화하여, 모델이 생성하는 답변의 사실성을 높이고 환각(hallucination) 현상을 이전 세대 모델에 비해 크게 줄였다고 밝혔다.33


GPT 기술은 눈부신 발전을 이루었지만, 여전히 해결해야 할 근본적인 기술적 한계와 심각한 윤리적 과제를 안고 있다. 이러한 문제들은 GPT의 작동 원리와 밀접하게 연관되어 있으며, 기술의 신뢰성과 사회적 수용성을 결정하는 중요한 요소이다.


GPT의 기반이 되는 트랜스포머 아키텍처의 가장 큰 약점은 셀프 어텐션 메커니즘의 계산 복잡도이다. 시퀀스 길이가 `$n$`일 때, 계산량은 `$O(n^2d)$`로 증가하므로, 처리해야 할 텍스트가 길어질수록 메모리 요구량과 연산 시간이 기하급수적으로 폭증한다.3 이는 모델이 한 번에 처리할 수 있는 문맥의 길이(context length)를 제한하는 직접적인 원인이 되며, 대규모 모델의 훈련과 추론에 막대한 비용을 발생시킨다. 이러한 비용은 대규모 자본을 가진 소수의 기업만이 최첨단 모델을 개발할 수 있는 진입 장벽으로 작용하며, 막대한 전력 소비로 인한 환경 문제도 야기한다.3 이 문제를 해결하기 위해, RNN의 순환 구조를 트랜스포머와 결합하여 긴 시퀀스 정보를 효율적으로 압축하거나 3, 양자 컴퓨팅을 활용한 하이브리드 어텐션 메커니즘을 탐구하는 등 17 아키텍처 수준의 혁신을 위한 연구가 활발히 진행되고 있다.


환각은 LLM이 직면한 가장 심각한 신뢰성 문제 중 하나로, 모델이 학습 데이터에 근거하지 않거나 명백히 사실과 다른 내용을 마치 사실인 것처럼 그럴듯하게 생성하는 현상을 말한다.37 이는 모델이 인간처럼 '이해'하고 추론하는 것이 아니라, 주어진 문맥 다음에 올 단어의 확률적 분포를 예측하는 방식으로 작동하기 때문에 발생하는 본질적인 문제이다.39 훈련 데이터에 정보가 부족하거나, 사용자의 프롬프트가 모호할 때, 모델은 가장 통계적으로 그럴듯한 단어들을 조합하여 '이야기를 지어내는' 경향이 있다. 이 과정에서 오류가 누적되면서 완전히 허구의 내용이 생성될 수 있다.39

환각 현상은 특히 의학, 법률, 금융과 같이 정확성이 중요한 전문 분야에서 치명적인 결과를 초래할 수 있다.41 실제로 미국의 한 변호사가 ChatGPT가 생성한 가짜 판례를 법원에 제출하여 징계를 받은 사례는 이 문제의 심각성을 잘 보여준다.42 또한, 환각은 가짜 뉴스나 허위 정보를 대량으로 생산하고 유포하는 데 악용될 수 있어 사회적 혼란을 야기할 잠재적 위험을 내포하고 있다.38


GPT와 같은 LLM은 인터넷을 포함한 방대한 텍스트 데이터로부터 학습한다. 이 데이터에는 인류가 쌓아온 지식뿐만 아니라, 사회에 만연한 인종, 성별, 종교, 정치적 편견과 고정관념 또한 고스란히 담겨 있다.22 모델은 이러한 편향을 무비판적으로 학습하고, 특정 집단에 대한 질문에 고정관념에 기반한 답변을 생성하거나, 특정 관점을 부당하게 옹호하는 방식으로 편향을 재현하고 증폭시킬 위험이 있다.40

이러한 편향된 출력은 사회적 불평등을 심화시키고 특정 소수 집단에 대한 차별을 고착화할 수 있다. 예를 들어, 특정 직업에 대해 남성적인 이미지를 강화하거나, 특정 인종에 대한 부정적인 고정관념을 담은 텍스트를 생성할 수 있다. OpenAI와 같은 개발사들은 다양한 인구통계를 대표하는 데이터셋을 구축하고 RLHF와 같은 정렬 기술을 통해 편향된 응답을 억제하려는 노력을 기울이고 있지만 30, 이는 근본적인 해결책이 되기 어렵다. 데이터에 내재된 미묘하고 복잡한 편향을 완전히 제거하는 것은 거의 불가능에 가깝기 때문에, 이는 지속적인 모니터링과 완화 노력이 필요한 장기적인 과제이다.

환각과 편향은 분리된 문제가 아니라, 모델의 근본적인 작동 원리인 '확률적 다음 단어 예측'에서 비롯된 동전의 양면과 같다. 모델의 유창성과 창의성을 가능하게 하는 바로 그 메커니즘이, 정보가 부족할 때는 환각을, 데이터가 편향되었을 때는 편견을 생성하는 원인이 된다. 따라서 이 문제들은 완벽한 '제거'의 대상이라기보다는, 모델의 능력과 안전성 사이의 균형을 맞추며 지속적으로 '관리'하고 '완화'해야 할 대상이다.


GPT 기술은 빠르게 발전하고 있으며, 미래의 모델들은 현재의 한계를 극복하고 새로운 차원의 능력을 갖추게 될 것으로 전망된다. 주요 발전 방향은 진정한 멀티모달리티 구현, 모델 효율성 증대, 그리고 추론 능력 강화 및 에이전트화로 요약할 수 있다. 이러한 변화는 GPT가 단순한 언어 처리 도구를 넘어, 인간과 더 복잡하고 유기적으로 상호작용하는 지능 시스템으로 진화하고 있음을 시사한다.


GPT-4가 텍스트와 이미지를 '입력'으로 처리하는 수준에 머물렀다면, GPT-4o("omni"를 의미)와 같은 차세대 모델들은 텍스트, 오디오, 이미지, 비디오 등 다양한 형태의 데이터를 단일 신경망 내에서 처음부터 끝까지(end-to-end) 통합적으로 처리하는 '네이티브 멀티모달' 모델로 진화하고 있다.1

이는 기존에 각 모달리티를 처리하는 별도의 모델들(예: 음성-텍스트 변환 모델, LLM, 텍스트-음성 합성 모델)을 파이프라인으로 연결하던 방식과 근본적으로 다르다. 단일 모델 내에서 모든 정보가 처리되므로, 여러 모델 간 데이터 변환 과정에서 발생하는 지연 시간(latency)을 획기적으로 줄일 수 있다.44 더 중요한 것은, 텍스트로 변환되면서 손실되었던 정보, 즉 목소리의 톤, 감정, 배경 소음과 같은 비언어적 뉘앙스까지 모델이 직접 포착하여 문맥을 이해할 수 있다는 점이다. 이를 통해 인간과의 대화처럼 훨씬 더 자연스럽고 풍부한 상호작용이 가능해질 것이다.


모델의 규모가 계속해서 커짐에 따라, 이를 실제 환경에 배포하고 운영하기 위한 효율화 기술의 중요성이 날로 커지고 있다. 미래의 GPT는 단순히 크기만 키우는 것이 아니라, 더 적은 자원으로 더 빠르게 작동하도록 만드는 기술들을 적극적으로 채택할 것이다.

- **양자화(Quantization)**: 모델의 가중치를 표현하는 데 사용되는 비트 수를 줄이는 기술이다. 예를 들어, 32비트 부동소수점으로 저장된 파라미터를 8비트나 4비트 정수로 변환하여 모델의 크기를 수 배로 줄이고, 메모리 사용량과 추론 속도를 개선한다.46
- **가지치기(Pruning)**: 훈련된 모델에서 성능에 거의 영향을 미치지 않는 불필요한 가중치나 뉴런 연결을 제거하는 기술이다. 이를 통해 모델을 더 가볍고 희소(sparse)하게 만들어 계산 효율성을 높인다.46
- **전문가 혼합(Mixture-of-Experts, MoE)**: 하나의 거대한 모델 대신, 특정 분야나 데이터 패턴에 특화된 여러 개의 작은 '전문가' 모델(하위 네트워크)을 두고, 입력에 따라 가장 관련 있는 소수의 전문가 네트워크만 활성화하여 계산을 수행하는 아키텍처이다. 이는 전체 파라미터 수는 많더라도 추론 시에는 일부만 사용하므로 계산 비용을 크게 절감할 수 있다.47


미래의 GPT는 사용자의 질문에 수동적으로 답변하는 것을 넘어, 복잡한 목표를 달성하기 위해 스스로 계획을 세우고, 필요한 정보를 찾기 위해 웹을 검색하거나, 다른 소프트웨어(API)를 호출하는 등 능동적으로 행동하는 'AI 에이전트'로 발전할 것이다.1 Auto-GPT와 같은 초기 사례들은 LLM이 스스로 프롬프트를 생성하고 실행하며 목표를 향해 나아가는 재귀적(recursive) 에이전트의 가능성을 보여주었다.1

이러한 발전은 모델이 단순한 언어 생성을 넘어 다단계 추론(multi-step reasoning) 능력을 갖추어야 함을 의미한다. GPT-5와 같은 차세대 모델은 문제의 복잡도에 따라 내부적으로 다른 처리 방식을 선택하는 '라우터(router)'와 같은 메커니즘을 도입할 것으로 예상된다. 예를 들어, 간단한 질문에는 작고 빠른 모델을 사용하여 신속하게 답변하고, 복잡한 추론이 필요한 문제에는 더 크고 강력한 모델을 활성화하여 깊이 있는 분석을 제공하는 식이다.1 이는 모델 아키텍처 내부(MoE), 모델과 외부 도구의 상호작용(에이전트), 그리고 다양한 모델 간의 협력(라우터)이라는 여러 층위에서, GPT가 '단일 지능체'에서 '분산형 협력 시스템'으로 진화하고 있음을 보여준다. 이러한 변화는 확장성, 효율성, 그리고 능력의 범위를 동시에 확장하기 위한 필연적인 방향이다.


본 보고서는 생성형 사전 훈련 트랜스포머(GPT)의 기술적 근간부터 학습 전략, 진화 과정, 그리고 미래 전망에 이르기까지 다각적인 고찰을 수행했다. 분석 결과, GPT의 성공은 트랜스포머라는 혁신적인 아키텍처가 '규모의 법칙'과 결합하여 언어 모델의 성능을 비약적으로 발전시켰으며, '사전 훈련-미세 조정-정렬'이라는 체계적인 학습 전략을 통해 범용성과 실용성을 동시에 확보한 결과물임을 확인했다. 셀프 어텐션 메커니즘은 계산적 비용이라는 한계에도 불구하고 전역적 문맥 이해 능력을 제공했으며, 생성형 사전 훈련은 데이터 병목을 해결하고, 미세 조정과 RLHF는 모델을 특정 목적과 인간의 가치에 부합하도록 정교하게 다듬는 역할을 수행했다.

그러나 이러한 눈부신 발전 이면에는 계산적 효율성, 정보의 신뢰성(환각), 그리고 윤리적 정렬(편향)이라는 세 가지 핵심적인 과제가 여전히 남아있다. 이 문제들은 기술의 지속 가능한 발전을 위해 반드시 해결해야 할 난제이며, 앞으로의 연구는 이러한 한계를 극복하는 데 집중될 필요가 있다.

이러한 고찰을 바탕으로, 미래 GPT 기술의 발전을 위한 몇 가지 연구 방향을 다음과 같이 제언한다.

1. **아키텍처 혁신**: 현재의 어텐션 메커니즘이 가진 이차 복잡도를 근본적으로 해결하고, 더 긴 문맥을 효율적으로 처리할 수 있는 새로운 아키텍처에 대한 탐구가 시급하다. 이는 양자 컴퓨팅, 새로운 신경망 구조 등 다양한 분야와의 융합을 통해 이루어질 수 있다.
2. **데이터 중심 AI(Data-Centric AI)**: 모델의 성능과 안전성은 결국 훈련 데이터의 질에 의해 결정된다. 따라서 단순히 데이터의 양을 늘리는 것을 넘어, 고품질의 편향 없는 데이터셋을 구축, 정제, 관리하는 기술에 대한 투자를 확대해야 한다. 이는 모델의 환각과 편향을 완화하는 가장 근본적인 접근법이 될 것이다.
3. **해석 가능성 및 신뢰성 강화**: 모델이 왜 특정 결론에 도달했는지 그 과정을 설명할 수 있는 해석 가능성(Interpretability) 연구를 강화해야 한다. 이는 모델의 예측을 신뢰하고, 환각을 효과적으로 탐지하며, 오류 발생 시 원인을 파악하여 수정하는 데 필수적이다.
4. **인간-AI 협업 모델 구축**: GPT를 인간을 대체하는 도구가 아닌, 인간의 창의성과 지능을 증강시키는 협력적 파트너로 자리매김하기 위한 연구가 필요하다. 이를 위해 인간의 의도를 더 잘 파악하고, 피드백을 통해 지속적으로 학습하며, 복잡한 문제 해결 과정에서 인간과 원활하게 상호작용할 수 있는 인터페이스 및 방법론 개발이 중요하다.

결론적으로, GPT는 인공지능의 새로운 시대를 열었지만, 그 잠재력을 온전히 실현하고 사회에 긍정적으로 기여하기 위해서는 기술적 정교화와 함께 윤리적, 사회적 책임에 대한 깊은 성찰이 병행되어야 할 것이다.


1. Generative pre-trained transformer - Wikipedia, accessed August 16, 2025, https://en.wikipedia.org/wiki/Generative_pre-trained_transformer
2. Attention Is All You Need - Wikipedia, accessed August 16, 2025, https://en.wikipedia.org/wiki/Attention_Is_All_You_Need
3. arXiv:2505.00929v1 [cs.LG] 2 May 2025, accessed August 16, 2025, https://arxiv.org/pdf/2505.00929
4. Attention is All you Need - NIPS, accessed August 16, 2025, https://papers.neurips.cc/paper/7181-attention-is-all-you-need.pdf
5. "Attention is All You Need" Summary - Medium, accessed August 16, 2025, https://medium.com/@dminhk/attention-is-all-you-need-summary-6f0437e63a91
6. What is Pretraining and Fine-tuning? | Activeloop Glossary, accessed August 16, 2025, https://www.activeloop.ai/resources/glossary/pretraining-and-fine-tuning/
7. What is Fine-Tuning? | IBM, accessed August 16, 2025, https://www.ibm.com/think/topics/fine-tuning
8. GPT-1 - Wikipedia, accessed August 16, 2025, https://en.wikipedia.org/wiki/GPT-1
9. Large Language Models: A Survey - arXiv, accessed August 16, 2025, https://arxiv.org/html/2402.06196v2
10. Contextual Position Encoding: Learning to Count What's Important - arXiv, accessed August 16, 2025, https://arxiv.org/html/2405.18719v1
11. Attention is All you Need - NIPS, accessed August 16, 2025, https://papers.nips.cc/paper/7181-attention-is-all-you-need
12. The Attention Mechanism from Scratch - MachineLearningMastery.com, accessed August 16, 2025, https://machinelearningmastery.com/the-attention-mechanism-from-scratch/
13. Understanding and Coding the Self-Attention Mechanism of Large Language Models From Scratch - Sebastian Raschka, accessed August 16, 2025, https://sebastianraschka.com/blog/2023/self-attention-from-scratch.html
14. Classical ML Equations in LaTeX, accessed August 16, 2025, https://blmoistawinde.github.io/ml_equations_latex/
15. Computational Complexity of Self-Attention in the Transformer Model - Stack Overflow, accessed August 16, 2025, https://stackoverflow.com/questions/65703260/computational-complexity-of-self-attention-in-the-transformer-model
16. Positional Encoding in Transformer-Based Time Series Models: A Survey - arXiv, accessed August 16, 2025, https://arxiv.org/html/2502.12370v1
17. A Hybrid Transformer Architecture with a Quantized Self-Attention Mechanism Applied to Molecular Generation - arXiv, accessed August 16, 2025, https://arxiv.org/html/2502.19214v1
18. [2204.08142] Dynamic Position Encoding for Transformers - arXiv, accessed August 16, 2025, https://arxiv.org/abs/2204.08142
19. How does fine tuning really work? - API - OpenAI Developer Community, accessed August 16, 2025, https://community.openai.com/t/how-does-fine-tuning-really-work/39972
20. Attention Is All You Need | Request PDF - ResearchGate, accessed August 16, 2025, https://www.researchgate.net/publication/317558625_Attention_Is_All_You_Need
21. The Importance of Positional Encoding Initialization in Transformers for Relational Reasoning - arXiv, accessed August 16, 2025, https://arxiv.org/html/2406.08272v1
22. Pre-Training vs Fine Tuning: Choosing the Right Approach in 2025 ..., accessed August 16, 2025, https://labelyourdata.com/articles/llm-fine-tuning/pre-training-vs-fine-tuning
23. Fine-Tuning vs. Pre-Training: Their Impact on Language Models - Sapien, accessed August 16, 2025, https://www.sapien.io/blog/fine-tuning-vs-pre-training-key-differences-for-language-models
24. Improving Language Understanding by Generative Pre ... - OpenAI, accessed August 16, 2025, https://cdn.openai.com/research-covers/language-unsupervised/language_understanding_paper.pdf
25. GPT-2 - Wikipedia, accessed August 16, 2025, https://en.wikipedia.org/wiki/GPT-2
26. GPT-3 - Wikipedia, accessed August 16, 2025, https://en.wikipedia.org/wiki/GPT-3
27. Reinforcement Learning from Human Feedback (RLHF) for Large Language Model Alignment - Medium, accessed August 16, 2025, https://medium.com/@ramdhanhdy/reinforcement-learning-from-human-feedback-rlhf-for-large-language-model-alignment-44af40eba999
28. A Technical Survey of Reinforcement Learning Techniques for Large Language Models, accessed August 16, 2025, https://arxiv.org/html/2507.04136v1
29. What Is Reinforcement Learning From Human Feedback (RLHF)? - IBM, accessed August 16, 2025, https://www.ibm.com/think/topics/rlhf
30. The Role of RLHF in Mitigating Bias and Improving AI Model Fairness | HackerNoon, accessed August 16, 2025, https://hackernoon.com/the-role-of-rlhf-in-mitigating-bias-and-improving-ai-model-fairness
31. Fine-tune large language models with reinforcement learning from human or AI feedback, accessed August 16, 2025, https://aws.amazon.com/blogs/machine-learning/fine-tune-large-language-models-with-reinforcement-learning-from-human-or-ai-feedback/
32. The Evolution of Language Models: From GPT-1 to GPT-4 and ..., accessed August 16, 2025, https://www.geeksforgeeks.org/artificial-intelligence/the-evolution-of-language-models-from-gpt-1-to-gpt-4-and-beyond/
33. Evolution of GPT Models, GPT 1 to GPT 4 | by Vipul Koti - Medium, accessed August 16, 2025, https://medium.com/@vipul.koti333/evolution-of-gpt-models-gpt-1-to-gpt-4-0238ee07a29b
34. GPT-2 - Hugging Face, accessed August 16, 2025, https://huggingface.co/docs/transformers/model_doc/gpt2
35. The Complete History of OpenAI Models: From GPT-1 to GPT-5 | Data Science Dojo, accessed August 16, 2025, https://datasciencedojo.com/blog/the-complete-history-of-openai-models/
36. The Evolution of ChatGPT from OpenAi: From GPT-1 to GPT-4o - TTMS, accessed August 16, 2025, https://ttms.com/chat-gpt-evolution/
37. ChatGPT 101 for LATEX users | EMS Magazine - European Mathematical Society, accessed August 16, 2025, https://euromathsoc.org/magazine/articles/207
38. ChatGPT: What Are Hallucinations And Why Are They A Problem For AI Systems, accessed August 16, 2025, https://bernardmarr.com/chatgpt-what-are-hallucinations-and-why-are-they-a-problem-for-ai-systems/
39. Hallucination (artificial intelligence) - Wikipedia, accessed August 16, 2025, https://en.wikipedia.org/wiki/Hallucination_(artificial_intelligence)
40. The Dark Side of ChatGPT: Legal and Ethical Challenges from Stochastic Parrots and Hallucination, accessed August 16, 2025, https://www.create.ac.uk/blog/2023/10/11/the-dark-side-of-chatgpt-legal-and-ethical-challenges-from-stochastic-parrots-and-hallucination/
41. The Limitations and Ethical Considerations of ChatGPT | Data Intelligence - MIT Press Direct, accessed August 16, 2025, https://direct.mit.edu/dint/article/6/1/201/118839/The-Limitations-and-Ethical-Considerations-of
42. Not All Hallucinations Are Bad: The Constraints and Benefits of Generative AI - NTT Data, accessed August 16, 2025, https://www.nttdata.com/global/en/insights/focus/2024/not-all-hallucinations-are-bad-the-constraints-and-benefits-of-generative-ai
43. Ethical ChatGPT: Concerns, Challenges, and Commandments - MDPI, accessed August 16, 2025, https://www.mdpi.com/2079-9292/13/17/3417
44. What Is GPT-4o? - IBM, accessed August 16, 2025, https://www.ibm.com/think/topics/gpt-4o
45. Top 10 Multimodal AI Models of 2024 - Zilliz Learn, accessed August 16, 2025, https://zilliz.com/learn/top-10-best-multimodal-ai-models-you-should-know
46. Quantization, Pruning, and Distillation - Graham Neubig, accessed August 16, 2025, https://phontron.com/class/anlp2024/assets/slides/anlp-11-distillation.pdf
47. Introducing gpt-oss - OpenAI, accessed August 16, 2025, https://openai.com/index/introducing-gpt-oss/
48. Introducing GPT-5 - OpenAI, accessed August 16, 2025, https://openai.com/index/introducing-gpt-5/
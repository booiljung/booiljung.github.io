# 비전 트랜스포머(ViT)


2010년대 컴퓨터 비전 분야는 합성곱 신경망(Convolutional Neural Networks, CNN)의 시대였다고 해도 과언이 아니다.1 AlexNet의 등장을 기점으로 시작된 딥러닝 혁명은 이미지 인식, 객체 탐지, 의미론적 분할 등 다양한 시각 과제에서 인간의 능력을 뛰어넘는 성과를 이끌어냈으며, 그 중심에는 항상 CNN이 있었다. 그러나 10년 가까이 이어진 CNN의 지배적인 패러다임은 2020년, 자연어 처리(Natural Language Processing, NLP) 분야에서 유래한 새로운 아키텍처의 등장으로 근본적인 도전에 직면하게 된다. 구글 리서치 팀이 발표한 "An Image is Worth 16x16 Words"라는 도발적인 제목의 논문은 비전 트랜스포머(Vision Transformer, ViT)의 탄생을 알리며 컴퓨터 비전 분야에 새로운 패러다임 전환을 예고했다.1

ViT는 이미지를 픽셀의 격자가 아닌, 단어들의 나열, 즉 시퀀스로 취급하는 혁신적인 접근법을 제시했다. 이는 이미지의 지역적 특징을 점진적으로 쌓아 올려 전체를 이해하는 CNN의 계층적 방식에서 벗어나, 이미지의 모든 부분을 한 번에 고려하여 전역적인 관계를 파악하는 것을 목표로 한다. 이러한 접근법은 기존의 통념을 깨는 것이었지만, 대규모 데이터셋을 활용한 사전 훈련을 통해 CNN의 성능을 뛰어넘을 수 있음을 입증하며 그 잠재력을 증명했다.

본 보고서는 비전 트랜스포머에 대한 심층적인 고찰을 목표로 한다. 먼저, 1장에서는 ViT가 등장하게 된 배경, 즉 CNN 아키텍처가 가진 내재적 한계를 분석하고, ViT의 핵심 가설과 그 구조를 상세히 해부한다. 2장에서는 ViT와 CNN의 핵심적인 차이점인 귀납적 편향(inductive bias)과 데이터 효율성의 관계를 중심으로 두 아키텍처를 비교 분석한다. 3장에서는 ViT의 초기 한계를 극복하기 위해 등장한 DeiT, Swin Transformer와 같은 주요 변형 모델들과, 이에 대한 반향으로 나타난 ConvNeXt의 진화 과정을 추적하며 아키텍처 간의 상호작용을 탐구한다. 4장에서는 객체 탐지, 의료 영상 분석, 자율주행 등 다양한 응용 분야에서 ViT가 어떻게 활용되고 있으며 어떤 과제에 직면해 있는지를 살펴본다. 마지막으로 5장에서는 트랜스포머를 넘어 상태 공간 모델(State Space Model, SSM)과 같은 차세대 아키텍처의 가능성을 조망하고, 다중 모달리티 융합의 미래를 그리며 비전 트랜스포머가 컴퓨터 비전 분야에 남긴 유산과 앞으로의 방향성에 대해 종합적으로 논한다.


비전 트랜스포머의 등장은 단순한 성능 개선을 넘어 컴퓨터 비전의 근본적인 접근 방식에 대한 질문을 던진 사건이었다. 오랫동안 시각 데이터 처리에 최적화된 아키텍처로 여겨졌던 CNN의 한계가 명확해지면서, NLP 분야에서 검증된 트랜스포머가 그 대안으로 부상했다. 이 장에서는 CNN 시대의 기술적 배경과 그 한계를 짚어보고, ViT가 어떻게 이러한 한계를 극복하기 위한 대담한 가설을 제시했으며, 그 아키텍처가 구체적으로 어떻게 구성되어 있는지 심층적으로 분석한다.


2012년 AlexNet이 ImageNet 대회에서 압도적인 성능을 보인 이래, 컴퓨터 비전 분야는 CNN이 주도해왔다.2 CNN의 핵심은 이미지의 공간적 구조를 효과적으로 학습하기 위해 설계된 합성곱(convolution) 연산과 풀링(pooling) 연산에 있다.


CNN은 이미지의 지역적 특징(local feature)을 추출하고 이를 계층적으로 조합하여 더 복잡하고 추상적인 특징을 학습하는 방식으로 동작한다.5 초기 레이어에서는 이미지의 수평, 수직, 대각선 방향의 엣지(edge)와 같은 기본적인 특징을 감지하고, 이 정보가 상위 레이어로 전달되면서 모서리, 질감, 더 나아가 객체의 부분과 같은 복잡한 패턴을 인식하게 된다.6 이러한 과정은 지역적 수용장(local receptive field)을 갖는 필터(커널)가 이미지를 순회하며 특징 맵(feature map)을 생성하는 합성곱 연산을 통해 이루어진다.6 이 구조는 이미지 내에서 객체의 위치가 변하더라도 동일한 특징을 인식할 수 있는 이동 등변성(translation equivariance)이라는 강력한 귀납적 편향을 내재하고 있다.1


그러나 CNN의 이러한 구조적 특징은 동시에 명확한 한계를 내포하고 있었다. ViT의 등장은 바로 이 한계점을 정면으로 겨냥한 결과물이었다.

첫째, **제한된 수용장으로 인한 전역적 맥락 파악의 어려움**이다. CNN은 본질적으로 지역적인 정보를 처리하는 데 특화되어 있다.5 각 레이어의 뉴런은 이전 레이어의 특정 영역(수용장)에서만 정보를 받기 때문에, 이미지의 멀리 떨어진 부분들 간의 관계, 즉 장거리 의존성(long-range dependency)을 포착하기 어렵다.5 예를 들어, 이미지 속 '배'가 '물' 근처에 있을 가능성이 높다는 사실을 이해하기 위해서는 이미지 전체의 맥락을 파악해야 하지만, CNN은 지역적인 패치 분석에 치중한다.5 물론 네트워크의 깊이를 늘리면 수용장이 점차 넓어져 전역적인 정보를 어느 정도 학습할 수는 있지만, 이는 계산적으로 비효율적일 뿐만 아니라 장거리 상호작용을 명시적으로 모델링하는 방식이 아니라는 근본적인 문제를 안고 있다.5

둘째, **공간적 정보의 손실 문제**이다. CNN은 객체의 위치 변화에 강건해지기 위해 풀링 레이어(pooling layer), 특히 최대 풀링(max pooling)을 사용하여 특징 맵의 공간적 차원을 축소한다.6 이 과정은 계산량을 줄이고 모델을 일반화하는 데 도움이 되지만, 객체의 정확한 공간적 위치 정보를 상당 부분 손실시킨다는 치명적인 단점이 있다.5 종양의 정확한 위치가 중요한 의료 영상 분할과 같은 작업에서 이러한 정보 손실은 모델의 정확도를 심각하게 저해할 수 있다.5 저명한 AI 학자 제프리 힌튼(Geoffrey Hinton)은 특히 최대 풀링이 객체의 자세(pose), 즉 회전 및 변환 관계를 제대로 표현하지 못한다고 강하게 비판했다.12 또한, CNN은 데이터 증강(data augmentation)을 통해 학습 데이터에 포함되지 않은 회전이나 크기 변화에 대응하지만, 근본적으로 이러한 변화에 취약한 구조를 가지고 있다.8

셋째, **대규모 데이터 및 계산 자원 의존성**이다. CNN은 뛰어난 성능을 내기 위해 방대한 양의 레이블링된 데이터를 필요로 한다.5 이는 의료 영상이나 산업 결함 탐지와 같이 데이터 확보가 어렵거나 비용이 많이 드는 분야에서 CNN의 활용을 제한하는 요인이 된다.5 또한, ResNet-152와 같은 매우 깊은 아키텍처는 훈련과 추론에 막대한 계산 자원을 요구하여, 엣지 디바이스에서의 실시간 응용을 어렵게 만든다.5

이러한 CNN의 한계들은 단순히 점진적인 개선으로 해결하기 어려운 구조적인 문제였다. 지역적 정보 처리에 내재된 전역적 맥락 파악의 비효율성은 컴퓨터 비전 분야가 새로운 패러다임을 모색하게 만드는 직접적인 원인이 되었다.


이러한 배경 속에서 2020년, 구글 리서치 팀은 "An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale"이라는 논문을 통해 비전 트랜스포머(ViT)를 세상에 공개했다.1 이 논문의 핵심은 NLP 분야에서 압도적인 성공을 거둔 트랜스포머 아키텍처를 컴퓨터 비전에 직접 적용하자는, 당시로서는 매우 급진적인 아이디어였다.1


ViT는 NLP 분야에서 표준으로 자리 잡은 "대규모 말뭉치 사전 훈련 후 특정 과제 미세 조정(pre-train on a large text corpus and then fine-tune)" 접근법에서 직접적인 영감을 얻었다.1 트랜스포머는 셀프 어텐션(self-attention) 메커니즘을 통해 문장 내 단어들 간의 장거리 의존성을 효과적으로 모델링하며 NLP 분야를 평정했다. ViT 연구진들은 이미지도 텍스트처럼 시퀀스 데이터로 간주할 수 있다면, 트랜스포머의 강력한 모델링 능력을 비전 분야에서도 활용할 수 있을 것이라 생각했다.


ViT의 가장 핵심적인 가설은 **"컴퓨터 비전에서 CNN에 대한 의존은 필수적이지 않다"**는 것이었다.1 이전에도 어텐션 메커니즘을 CNN과 결합하거나 CNN의 일부 구성 요소를 대체하려는 시도는 있었지만, ViT는 여기서 한 걸음 더 나아가 CNN을 완전히 배제하고 순수한 트랜스포머 아키텍처만으로 이미지 인식을 수행할 수 있음을 보이고자 했다.1 이는 컨볼루션 연산이 가지고 있는 지역성(locality)과 이동 등변성(translation equivariance)과 같은 강력한 귀납적 편향이 이미지 처리에 필수적이라는 기존의 통념에 정면으로 도전하는 것이었다.


논문의 제목 "An Image is Worth 16x16 Words"는 ViT의 핵심 아이디어를 함축적으로 보여준다. ViT는 이미지를 컨볼루션 필터가 순회하는 2D 픽셀 격자로 보지 않고, 문장처럼 1D '단어'의 시퀀스로 간주한다.13 이를 위해 이미지를 16x16 픽셀 크기의 작은 패치(patch)들로 분할하는데, 이 패치 하나하나가 문장에서의 '단어(word)' 또는 '토큰(token)'에 해당한다.4 이렇게 생성된 패치 시퀀스를 트랜스포머에 입력하여, 시각 정보를 텍스트와 유사한 방식으로 처리하는 것이다.

이러한 발상의 전환은 단순히 아키텍처를 바꾸는 것을 넘어, 컴퓨터 비전의 문제 정의 자체를 재구성하는 철학적인 변화를 의미했다. 이는 이미지의 의미 구조가 엣지나 질감과 같은 지역적인 패턴을 찾도록 설계된 아키텍처(CNN) 없이도, 원시 데이터(패치)의 시퀀스로부터 직접 학습될 수 있다는 가능성을 시사했다. 즉, 충분한 데이터만 주어진다면 모델이 스스로 시각 세계의 '문법'(패치들 간의 관계)을 학습할 수 있다는 믿음에 기반한 것이었다.


ViT는 표준 트랜스포머 아키텍처를 최소한으로 수정하여 이미지 데이터에 적용한다. 전체 프로세스는 이미지를 트랜스포머가 처리할 수 있는 시퀀스 형태로 변환하는 전처리 단계와, 변환된 시퀀스로부터 특징을 추출하고 분류하는 트랜스포머 인코더 단계로 나눌 수 있다.18

1. 이미지 패치화 (Image Patching)

가장 먼저, 입력 이미지(예: 224×224 픽셀)를 고정된 크기(예: 16×16 픽셀)의 겹치지 않는 패치들로 분할한다.4

224×224 이미지를 16×16 패치로 나누면, 가로 14개, 세로 14개, 총 14×14=196개의 패치가 생성된다.16 이 과정의 목적은 2D 이미지 그리드를 트랜스포머가 처리할 수 있는 1D 토큰 시퀀스로 변환하는 것이다.16

2. 패치 임베딩 (Patch Embedding)

분할된 각 2D 패치는 1D 벡터로 펼쳐진다(flattening). 예를 들어, 16×16 크기의 RGB 패치는 16×16×3=768 차원의 벡터가 된다.16 이렇게 펼쳐진 패치 벡터들은 학습 가능한 선형 투영(linear projection) 레이어를 통해 모델이 사용하는 고정된 차원(D, 임베딩 차원)의 벡터로 변환된다.4 이 과정을 패치 임베딩이라 하며, 이는 NLP에서 단어를 임베딩 벡터로 변환하는 것과 유사한 역할을 한다. 원시 픽셀 값을 모델이 이해하고 처리하기 용이한 풍부한 특징 표현으로 매핑하는 것이다.16

3. 포지셔널 인코딩 (Positional Encoding)

표준 트랜스포머는 입력 토큰들의 순서나 위치에 대한 정보를 가지고 있지 않다. 즉, 입력 시퀀스의 순서를 뒤섞어도 결과가 동일한 순열 등변성(permutation-equivariant)을 갖는다.7 하지만 이미지에서는 각 패치의 공간적 위치가 매우 중요하다. 이 문제를 해결하기 위해 각 패치 임베딩에 위치 정보를 담은 

**포지셔널 인코딩(positional encoding)** 벡터를 더해준다.4 ViT에서는 주로 고정된 함수를 사용하는 대신, 위치 정보 자체를 학습 가능한 파라미터(learnable parameter)로 만들어 훈련 과정에서 최적의 위치 표현을 학습하도록 한다.18 이를 통해 모델은 각 패치가 원본 이미지의 어느 부분에 있었는지를 인식하고 공간적 관계를 학습할 수 있다.13

4. 토큰 (Classification Token)

NLP 모델인 BERT에서 영감을 받아, 패치 임베딩 시퀀스의 맨 앞에 학습 가능한 특수 토큰인 **(classification) 토큰**을 추가한다.[1, 17, 18, 21, 23] 이 토큰은 다른 패치 토큰들과 마찬가지로 트랜스포머 인코더를 통과하며 셀프 어텐션 메커니즘을 통해 이미지 전체의 정보를 종합하고 요약하는 역할을 한다.[18, 20] 최종적으로 트랜스포머 인코더를 거친  토큰의 출력 벡터가 이미지 전체를 대표하는 특징 벡터로 사용되어 분류 작업에 활용된다.18

5. 트랜스포머 인코더 (Transformer Encoder)

위치 정보가 더해진 패치 임베딩 시퀀스(및 `` 토큰)는 여러 개의 트랜스포머 인코더 레이어를 순차적으로 통과한다.15 각 인코더 레이어는 두 개의 주요 하위 레이어로 구성된다 18:

- **다중 헤드 셀프 어텐션 (Multi-Head Self-Attention, MSA):** 트랜스포머의 핵심 요소로, 시퀀스 내의 모든 토큰(패치)이 다른 모든 토큰과 상호작용하며 서로의 연관성을 계산한다.18 각 토큰은 다른 토큰들과의 유사도에 따라 가중치를 부여받아, 자신을 포함한 전체 시퀀스의 정보를 종합적으로 반영한 새로운 표현으로 업데이트된다. 이 메커니즘 덕분에 ViT는 모델의 첫 레이어부터 이미지 전체의 장거리 의존성과 전역적 맥락을 포착할 수 있다.9 '다중 헤드'는 이러한 어텐션 과정을 여러 개(head) 병렬적으로 수행하여, 각 헤드가 서로 다른 관점(예: 다른 특징 공간)에서 정보의 관계를 학습하도록 하는 기법이다.18
- **피드 포워드 신경망 (Feed-Forward Network, FFN) / MLP:** 셀프 어텐션을 거친 각 토큰 임베딩은 독립적으로 MLP(Multi-Layer Perceptron)를 통과한다. 이 MLP는 보통 두 개의 선형 레이어와 그 사이의 비선형 활성화 함수(주로 GELU)로 구성되며, 각 토큰의 특징 표현을 더욱 풍부하게 만든다.18
- **잔차 연결 및 레이어 정규화 (Residual Connections and Layer Normalization):** 각 하위 레이어(MSA, FFN)의 입력과 출력을 더해주는 잔차 연결(skip connection)과 레이어 정규화가 적용된다. 이는 깊은 네트워크에서 기울기 소실 문제를 완화하고 훈련을 안정화하며 수렴을 돕는 필수적인 기법이다.18
- MLP 헤드 (Classification Head)

마지막 트랜스포머 인코더 레이어를 통과한 `` 토큰의 최종 출력 벡터는 분류를 위한 간단한 MLP 헤드(일반적으로 하나 또는 두 개의 선형 레이어로 구성)에 입력된다.17 이 MLP 헤드의 최종 출력에 소프트맥스(softmax) 함수를 적용하여 각 클래스에 대한 확률을 계산하고, 이를 통해 이미지의 최종 레이블을 예측한다.


ViT의 등장은 컴퓨터 비전 분야에 '어떤 아키텍처가 시각 정보를 표현하는 데 더 본질적인가?'라는 근본적인 질문을 던졌다. ViT와 CNN의 대결은 단순히 두 모델의 성능 경쟁을 넘어, 서로 다른 철학을 가진 두 접근법의 충돌이었다. 이 장에서는 두 아키텍처의 가장 핵심적인 차이점인 귀납적 편향을 중심으로, 이것이 데이터 규모에 따른 성능 변화와 계산 복잡도에 어떤 영향을 미치는지 심층적으로 비교 분석한다.


귀납적 편향(Inductive Bias)은 모델이 이전에 본 적 없는 데이터에 대해 예측을 수행할 때 사용하는 가정의 집합을 의미한다. CNN과 ViT는 시각 데이터에 대해 매우 다른 가정을 가지고 있으며, 이것이 두 아키텍처의 성격을 규정하는 가장 중요한 차이점이다.


CNN은 이미지 처리에 매우 효과적인 두 가지 강력한 귀납적 편향을 아키텍처 자체에 내장하고 있다.2

- **지역성 (Locality):** 이미지는 지역적으로 강한 상관관계를 가진다는 가정이다. 즉, 특정 픽셀은 주변 픽셀들과 밀접하게 관련되어 있다. CNN은 작은 크기의 컨볼루션 커널을 사용하여 이미지의 작은 영역(지역적 수용장)만을 처리함으로써 이 가정을 구현한다.5
- **이동 등변성 (Translation Equivariance):** 이미지의 한 부분에서 유용한 특징(예: 눈 모양)은 다른 부분에서도 동일하게 유용하다는 가정이다. CNN은 동일한 필터를 이미지 전체에 걸쳐 슬라이딩하며 적용함으로써 이 가정을 만족시킨다.1

이러한 강한 귀납적 편향은 CNN이 이미지의 구조에 대한 사전 지식을 가지고 학습을 시작하게 하므로, 비교적 적은 데이터로도 효율적으로 학습하고 좋은 일반화 성능을 보일 수 있게 한다.7


반면, ViT는 의도적으로 이러한 귀납적 편향을 최소화한 구조를 가진다.1 ViT는 지역성을 가정하지 않는다. 셀프 어텐션 메커니즘은 모델의 가장 첫 번째 레이어부터 이미지의 모든 패치가 다른 모든 패치와 직접 상호작용할 수 있도록 허용하며, 이는 즉각적으로 전역적인 수용장을 갖게 됨을 의미한다.9 ViT에 도입된 유일한 이미지 특화 편향은 초기 단계의 패치 분할과 해상도에 따른 위치 인코딩뿐이다.22

이러한 유연성은 양날의 검과 같다. 특정 가정에 얽매이지 않기 때문에 데이터로부터 더 일반적이고 복잡한 패턴을 학습할 잠재력을 가지지만, 동시에 이미지의 기본 구조를 처음부터 데이터에만 의존하여 학습해야 하므로 방대한 양의 데이터가 필요하다.1


이러한 구조적 차이는 두 모델이 이미지를 "보는" 방식에 근본적인 차이를 만들어낸다. 한 연구에서는 이 차이를 다음과 같이 비유했다: CNN의 접근 방식이 "하나의 픽셀에서 시작하여 점차 줌아웃하며 전체를 이해하는 것"과 같다면, 트랜스포머는 "처음부터 전체 이미지에 대한 정보를 가지고 흐릿한 이미지를 서서히 초점을 맞춰 나가는 것"과 같다.27 실제로 ViT는 CNN에 비해 얕은 레이어와 깊은 레이어의 표현 간 유사성이 더 높으며, 이는 초기 레이어부터 전역적인 정보를 획득하기 때문이다.28 또한, ViT는 ResNet보다 더 많은 공간 정보를 레이어 전반에 걸쳐 보존하는 경향이 있다.28

이러한 차이는 귀납적 편향을 재평가하게 만든다. CNN의 편향은 데이터가 부족한 환경에서 학습을 용이하게 하는 유용한 '지름길'이지만, 동시에 모델이 학습할 수 있는 패턴을 제한하는 '족쇄'가 될 수도 있다. 반면, ViT의 약한 편향은 '지름길' 없이 데이터만으로 모든 것을 학습해야 하는 어려운 길이지만, 데이터가 충분하다면 CNN의 가정이 포착하지 못하는 더 복잡하고 비지역적인 관계까지 학습하여 더 높은 잠재력에 도달할 수 있음을 시사한다. 결국 이 둘의 대립은 사전 지식과 학습된 지식 사이의 균형에 대한 근본적인 질문으로 이어진다.

| 특징              | 합성곱 신경망 (CNN)                 | 비전 트랜스포머 (ViT)                  |
| ----------------- | ----------------------------------- | -------------------------------------- |
| **핵심 메커니즘** | 합성곱 필터 (Convolutional Filters) | 셀프 어텐션 (Self-Attention)           |
| **귀납적 편향**   | 강함 (지역성, 이동 등변성)          | 약함 (전역적 관계를 데이터로부터 학습) |
| **수용장**        | 지역적, 깊이에 따라 증가            | 첫 레이어부터 전역적                   |
| **데이터 처리**   | 계층적 (픽셀 -->> 엣지 -->> 부분 -->> 객체)  | 병렬적 (모든 패치가 동시에 상호작용)   |
| **위치 정보**     | 암시적 (아키텍처에 내재)            | 명시적 (포지셔널 인코딩으로 추가)      |
| **계산 복잡도**   | 픽셀 수에 선형적                    | 패치 수에 이차적                       |
| **데이터 요구량** | 중소 규모 데이터셋에서도 효과적     | SOTA 성능을 위해 매우 큰 데이터셋 필요 |

Table 1: ViT vs. CNN 아키텍처 비교. 이 표는 두 아키텍처의 핵심적인 차이점을 요약하여 보여준다. CNN은 지역성과 이동 등변성이라는 강력한 내장 가정을 통해 효율성을 얻는 반면, ViT는 이러한 가정을 최소화하고 셀프 어텐션을 통해 데이터로부터 직접 전역적인 관계를 학습한다. 이 차이가 데이터 요구량, 계산 복잡도 등 모든 측면에서 트레이드오프를 발생시킨다.1


ViT와 CNN의 가장 극적인 성능 차이는 학습 데이터의 규모에 따라 나타난다. 이는 ViT의 약한 귀납적 편향과 직접적으로 관련된 현상이다.


ViT 원본 논문의 가장 중요한 발견 중 하나는 데이터셋 크기와 모델 성능 사이의 결정적인 관계였다.1 ImageNet-1K(약 130만 개 이미지)와 같은 중간 규모의 데이터셋에서 훈련했을 때, ViT 모델들은 비슷한 크기의 ResNet 모델보다 몇 퍼센트 포인트 낮은 정확도를 보였다.1 연구진들은 이를 ViT가 CNN과 같은 강한 귀납적 편향이 없어 "불충분한" 데이터로는 잘 일반화하지 못하기 때문이라고 분석했다.1 즉, 데이터가 적을 때는 ViT의 유연성이 오히려 과적합(overfitting)으로 이어지기 쉬운 약점으로 작용하는 것이다.


하지만 데이터의 규모가 커지자 상황은 완전히 역전되었다. ViT 모델을 ImageNet-21k(약 1,400만 개 이미지)나 구글 내부의 JFT-300M(약 3억 300만 개 이미지)과 같은 대규모 데이터셋에서 사전 훈련(pre-training)하자, 그 성능이 폭발적으로 향상되기 시작했다.1 연구진들은 이 결과를 바탕으로 **"대규모 훈련은 귀납적 편향을 이긴다(large-scale training trumps inductive bias)"**는 유명한 결론을 내렸다.1 충분한 데이터를 통해 ViT는 이미지의 내재적 구조를 스스로 학습하여, CNN의 내장된 편향이 주는 이점을 뛰어넘는 일반화 성능을 달성한 것이다. 대규모 데이터로 사전 훈련된 ViT는 ImageNet을 포함한 다양한 벤치마크 데이터셋에서 당시 최고 수준의 CNN 모델들과 대등하거나 이를 능가하는 성능을 기록했다.1 이는 ViT의 진정한 힘이 '규모(scale)'와 결합될 때 발현됨을 명확히 보여주었다.30

이러한 결과는 AI 연구 분야의 중요한 교훈인 리치 서튼(Rich Sutton)의 "쓰라린 교훈(The Bitter Lesson)"을 컴퓨터 비전 분야에서 재확인시켜 준 사례로 볼 수 있다.27 이 교훈의 핵심은 장기적으로 인간의 지식이나 편향을 모델에 주입하려는 노력보다, 막대한 계산 능력을 활용하는 일반적인 학습 방법이 항상 승리한다는 것이다. 인간의 시각 시스템에 대한 직관(지역성 등)을 기반으로 설계된 CNN보다, 훨씬 더 일반적인 아키텍처(트랜스포머)가 대규모 데이터와 컴퓨팅 파워를 통해 더 뛰어난 성능을 달성한 것은 이 교훈의 완벽한 예시이다. 이는 컴퓨터 비전의 미래 또한 특정 영역에 특화된 아키텍처를 정교하게 설계하는 것보다, 확장 가능한 범용 아키텍처를 스케일업하는 방향으로 나아갈 것임을 시사한다.


ViT와 CNN은 성능과 효율성 측면에서 뚜렷한 트레이드오프 관계를 보인다.


대규모 데이터셋으로 사전 훈련되었을 때, ViT의 성능은 매우 인상적이다. JFT-300M 데이터셋으로 사전 훈련된 ViT-H/14 모델은 ImageNet에서 88.55%의 top-1 정확도를 달성하여, 당시 최고 성능의 ResNet 기반 모델들을 능가했다.1 이는 적절한 조건만 갖춰진다면 순수 트랜스포머가 이미지 분류 분야에서 최고 수준(State-Of-The-Art, SOTA)이 될 수 있음을 증명한 것이다.


계산 복잡도 측면에서 두 아키텍처는 상반된 특징을 보인다.

- **ViT:** 셀프 어텐션 메커니즘은 입력 시퀀스의 길이, 즉 패치의 수(N)에 대해 $O(N^2)$의 계산 복잡도를 가진다.1 이미지 해상도가 높아지면 패치의 수가 증가하므로, 계산량이 이차적으로 급증한다. 이 때문에 고해상도 이미지 처리나 모든 픽셀 간의 어텐션을 계산하는 것은 현실적으로 불가능하다.1
- **CNN:** CNN의 합성곱 연산은 전체 픽셀 수에 대해 선형적인(O(N)) 계산 복잡도를 가지므로, 고해상도 이미지 처리에 훨씬 효율적이다.13


흥미롭게도, 어텐션 연산 자체의 높은 비용에도 불구하고, ViT 원본 논문에서는 특정 성능 목표를 달성하기 위한 **총 사전 훈련 비용**은 ViT가 SOTA CNN보다 상당히 적게 소요된다고 보고했다 (TPUv3-days 기준).1 이는 ViT가 대규모 데이터로부터 더 효율적으로 학습할 수 있음을 시사한다. 즉, 이미지 한 장을 처리하는 비용은 비쌀 수 있지만, 전체 훈련 과정에서 더 적은 연산으로 목표 성능에 도달할 수 있다는 의미이다.

결론적으로, ViT와 CNN의 관계는 어느 한쪽의 절대적인 우위가 아닌, 주어진 문제의 특성(데이터 규모, 해상도, 실시간성 요구 등)에 따라 최적의 선택이 달라지는 복잡한 트레이드오프 관계에 있다. ViT는 대규모 데이터와 전역적 맥락 이해가 중요한 문제에서 강력한 성능을 발휘하며, CNN은 효율성과 지역적 특징 추출이 중요한 환경에서 여전히 강력한 대안으로 남아있다.


ViT의 등장은 끝이 아니라 새로운 시작이었다. ViT가 제시한 가능성과 동시에 드러난 명확한 한계들(데이터 비효율성, 높은 계산 비용)은 후속 연구의 폭발적인 증가를 촉발했다. 연구 커뮤니티는 ViT의 약점을 보완하고 강점을 극대화하기 위한 다양한 혁신을 제안했으며, 이 과정에서 트랜스포머와 CNN 패러다임 간의 흥미로운 기술적 대화와 융합이 이루어졌다. 이 장에서는 ViT의 진화 과정에서 나타난 세 가지 중요한 이정표-DeiT, Swin Transformer, 그리고 ConvNeXt-를 통해 아키텍처 발전의 흐름을 추적한다.


ViT의 가장 큰 실용적 장벽은 막대한 양의 사전 훈련 데이터에 대한 의존성이었다. 수억 장의 이미지를 사용하는 것은 학계나 소규모 기업에서는 거의 불가능한 일이었다. 이 문제를 해결하기 위해 등장한 것이 바로 **DeiT(Data-efficient Image Transformer)**이다.32


DeiT의 목표는 명확했다: JFT-300M과 같은 거대 사유 데이터셋 없이, 공개된 ImageNet-1K 데이터셋만으로 경쟁력 있는 성능의 ViT를 훈련시키는 것.36 이를 위해 DeiT는 **지식 증류(Knowledge Distillation)**라는 기법을 트랜스포머에 맞게 변형하여 도입했다.36 지식 증류는 잘 훈련된 대형 모델("교사 모델", teacher)의 지식을 더 작은 모델("학생 모델", student)에게 전달하는 학습 방법이다.

DeiT는 학생 모델로 ViT를, 교사 모델로는 주로 성능이 검증된 CNN(예: ResNet, RegNet)을 사용했다.35 이는 매우 전략적인 선택이었는데, 학생인 ViT가 부족한 이미지 관련 귀납적 편향을 교사인 CNN으로부터 간접적으로 학습할 수 있게 했기 때문이다.35


DeiT의 가장 독창적인 기여는 **증류 토큰(distillation token)**의 도입이다.32 기존의 지식 증류가 단순히 교사 모델의 최종 출력(소프트 레이블)을 손실 함수에 추가하는 방식이었다면, DeiT는 증류 과정을 모델 아키텍처 내부로 가져왔다.

증류 토큰은 `토큰과 마찬가지로 학습 가능한 벡터이며, 입력 패치 시퀀스에 함께 추가된다.35 이 토큰은` 토큰이 실제 정답 레이블(ground-truth)을 예측하도록 학습되는 것과 달리, **교사 모델의 예측을 모방하도록** 명시적으로 학습된다.37 중요한 점은 이 증류 토큰이 다른 모든 토큰들과 셀프 어텐션을 통해 상호작용하며 네트워크 전체에 걸쳐 흐른다는 것이다. 이를 통해 교사의 '지식'이 단순히 최종단에서 손실로 작용하는 것을 넘어, 어텐션 메커니즘을 통해 학생 모델의 특징 표현 학습 과정 전체에 깊숙이 관여하게 된다.35


DeiT는 두 가지 방식의 증류를 제안한다.

- **하드 증류(Hard Distillation):** 교사 모델이 예측한 가장 확률이 높은 클래스(하드 레이블)를 학생 모델의 증류 토큰이 예측하도록 학습한다. 전체 손실 함수는 `` 토큰에 대한 실제 정답 레이블과의 교차 엔트로피(Cross-Entropy) 손실과, 증류 토큰에 대한 교사 예측 레이블과의 교차 엔트로피 손실의 조합으로 구성된다.35

- **소프트 증류(Soft Distillation):** 교사 모델의 전체 확률 분포(로짓)를 활용한다. 손실 함수는 실제 정답에 대한 교차 엔트로피 손실과, 학생 모델과 교사 모델의 출력 분포 간의 **쿨백-라이블러 발산(Kullback-Leibler Divergence, KL Divergence)** 손실의 가중합으로 계산된다. 이때 온도(temperature) 파라미터 \tau를 사용하여 교사와 학생의 출력 분포를 부드럽게 만들어, 정답 클래스 외에 다른 클래스들에 대한 교사의 '생각'(클래스 간 유사도 정보)까지 학생이 학습하도록 유도한다.35 소프트 증류의 손실 함수는 다음과 같이 표현된다:
  $$
  \mathcal{L}\text{global} = (1 - \lambda) \mathcal{L}\text{CE} (\psi(Z_s), y) + \lambda \tau^2 KL (\psi(Z_s / \tau), \psi(Z_t / \tau))
  $$
  여기서 Zs와 Zt는 각각 학생과 교사 모델의 로짓, y는 실제 레이블, ψ는 소프트맥스 함수, λ는 두 손실 간의 균형을 맞추는 계수이다.37

이러한 증류 전략을 통해 DeiT는 ImageNet-1K 훈련만으로 ViT-Base를 83.1%의 top-1 정확도까지 끌어올렸고, CNN 교사를 활용한 증류 모델(DeiT-B⚗)은 85.2%라는, 당시 SOTA CNN 모델들과 견줄 만한 놀라운 성과를 달성했다.37 이는 ViT의 실용성을 크게 높인 중요한 진전이었다. 이후 DeiT-LT와 같은 연구는 이 아이디어를 불균형 데이터셋(long-tailed dataset) 문제에 적용하여, `` 토큰은 다수 클래스 전문가로, 증류 토큰은 소수 클래스 전문가로 학습시키는 방식으로 발전시켰다.32


ViT는 이미지 분류에서는 성공을 거두었지만, 객체 탐지나 의미론적 분할과 같은 고밀도 예측(dense prediction) 작업에 직접 적용하기에는 두 가지 명백한 한계가 있었다.39

1. **고정된 스케일(Fixed Scale):** ViT는 단일 해상도의 특징 맵을 생성하므로, 이미지 내에 다양한 크기로 나타나는 객체들을 처리하기에 부적합했다.39
2. **이차적 계산 복잡도(Quadratic Complexity):** ViT의 전역 셀프 어텐션은 이미지 크기에 대해 이차적으로 계산량이 증가하여, 고해상도 이미지를 처리해야 하는 고밀도 예측 작업에는 비현실적이었다.39

**Swin Transformer**는 이러한 문제들을 해결하고 트랜스포머를 범용 비전 백본(general-purpose vision backbone)으로 만들기 위해 등장했다. Swin Transformer의 핵심은 CNN의 장점들을 트랜스포머 프레임워크 안으로 다시 가져오는 것이었다.


Swin Transformer는 CNN과 유사한 **계층적 아키텍처(hierarchical architecture)**를 도입했다.2 ViT가 이미지 전체에 걸쳐 동일한 크기의 패치를 사용하는 것과 달리, Swin Transformer는 다음과 같이 동작한다:

- 초기에는 작은 크기의 패치로 시작한다.
- 네트워크의 각 단계(stage)가 끝날 때마다 **패치 병합(patch merging)** 레이어를 통해 이웃한 2×2 패치들의 특징을 연결하고 차원을 축소한다.39
- 이 과정은 특징 맵의 공간 해상도를 절반으로 줄이는 대신(2× 다운샘플링), 채널의 수를 두 배로 늘린다.

이러한 방식을 통해 Swin Transformer는 VGG나 ResNet과 같은 CNN처럼 다양한 스케일의 특징 맵(예: 원본 이미지의 1/4,1/8,1/16,1/32 해상도)을 생성한다.40 이 계층적 구조 덕분에 특징 피라미드 네트워크(FPN)나 U-Net과 같은 기존의 고밀도 예측 기법들과 쉽게 결합할 수 있게 되었고, 객체 탐지 및 분할 작업에서 강력한 성능을 발휘하는 범용 백본으로 자리매김했다.39


ViT의 이차적 계산 복잡도 문제를 해결하기 위해, Swin Transformer는 셀프 어텐션의 계산 범위를 이미지 전체가 아닌, 겹치지 않는 **지역적 윈도우(local window)** 내부로 제한했다 (W-MSA: Windowed Multi-head Self-Attention).39 윈도우 크기를 고정하면, 전체 계산 복잡도는 이미지 크기에 대해 선형적으로 증가하게 되어 매우 효율적이다.39

하지만 이 방식은 윈도우 간의 정보 교류를 차단하는 문제가 있다. 이를 해결하기 위해 Swin Transformer는 **이동 윈도우 자기 주의(SW-MSA: Shifted Window Multi-head Self-Attention)**라는 독창적인 기법을 제안했다.39 이는 연속된 두 개의 트랜스포머 블록에서 윈도우 분할 방식을 다르게 하는 것이다.

- 한 블록(l)에서는 일반적인 방식으로 윈도우를 나눈다.
- 다음 블록(l+1)에서는 윈도우를 윈도우 크기의 절반만큼 이동시켜 새로운 분할을 만든다.

이렇게 이동된 윈도우는 이전 레이어에서는 서로 다른 윈도우에 속했던 패치들을 하나의 윈도우 안에 포함시키게 되어, 효율적인 **윈도우 간 연결(cross-window connection)**을 가능하게 한다.39 이 기법은 선형 복잡도의 이점을 유지하면서도 모델링 능력을 크게 향상시켰다.

Swin Transformer는 이러한 혁신을 바탕으로 이미지 분류, COCO 객체 탐지, ADE20K 의미론적 분할 등 주요 비전 벤치마크에서 SOTA 성능을 달성하며 가장 성공적인 ViT 변형 모델 중 하나가 되었다.39 이후 Swin Transformer V2는 모델을 더 큰 규모와 해상도로 확장했고 44, SwinIR 45이나 StrideNET 46과 같이 이미지 복원이나 지형 인식 등 다양한 분야로 응용이 확장되었다.


ViT, DeiT, Swin Transformer의 연이은 성공은 트랜스포머가 컴퓨터 비전의 미래라는 인식을 확산시켰다. 그러나 2022년, "A ConvNet for the 2020s"라는 논문과 함께 등장한 **ConvNeXt**는 이러한 흐름에 흥미로운 반론을 제기했다.41


ConvNeXt 연구진들은 다음과 같은 근본적인 질문을 던졌다: "Swin Transformer와 같은 계층적 트랜스포머의 성공이 과연 셀프 어텐션 메커니즘 자체의 본질적인 우월성 때문인가, 아니면 단순히 더 나은 아키텍처 설계와 훈련 방식의 결과인가?".41 그들은 순수 CNN의 잠재력을 다시 한번 시험해보기 위해, 표준적인 CNN 모델인 ResNet을 트랜스포머의 설계 원칙에 따라 점진적으로 "현대화"하는 실험을 진행했다.2


ConvNeXt는 어텐션 메커니즘을 전혀 사용하지 않고, 오직 표준적인 CNN 모듈만을 사용하여 ResNet을 Swin Transformer와 유사한 형태로 바꾸어 나갔다. 이 과정은 다음과 같은 단계로 이루어졌다 2:

1. **훈련 기법 현대화:** 먼저, ResNet-50을 ViT와 동일한 최신 훈련 기법(300 에포크 장기 훈련, AdamW 옵티마이저, Mixup/Cutmix 등 강력한 데이터 증강, Stochastic Depth/Label Smoothing 등 정규화)으로 훈련했다. 이 변화만으로도 정확도가 2.7%p나 상승했다.
2. **매크로 디자인 변경:** Swin Transformer를 모방하여 스테이지별 블록 수 비율을 조정하고(예: (3, 4, 6, 3) -->> (3, 3, 9, 3)), ResNet의 복잡한 스템(stem) 구조를 4×4 크기의 스트라이드 4 컨볼루션 레이어 하나로 바꾸는 "패치화(patchify)"를 적용했다.
3. **ResNeXt-ify:** ResNeXt에서 영감을 받아 그룹 컨볼루션의 극단적인 형태인 깊이별 컨볼루션(depthwise convolution)을 도입하고, 줄어든 연산량을 보상하기 위해 네트워크의 너비(채널 수)를 늘렸다.
4. **역 병목 구조(Inverted Bottleneck):** 트랜스포머의 MLP 블록처럼, 블록 내부의 채널 수를 4배로 확장했다가 다시 줄이는 역 병목 구조를 채택했다. 이는 MobileNetV2에서 널리 알려진 구조이기도 하다.
5. **큰 커널 크기 사용:** 깊이별 컨볼루션 레이어의 커널 크기를 기존의 3×3에서 7×7로 대폭 키웠다.
6. **마이크로 디자인 변경:** 활성화 함수를 ReLU에서 GELU로 변경하고, 블록 내 활성화 함수와 정규화 레이어의 수를 줄였으며, 배치 정규화(BatchNorm)를 레이어 정규화(LayerNorm)로 대체하는 등, 트랜스포머 블록의 디자인을 세밀하게 모방했다.


놀랍게도, 순수 CNN 모듈만으로 구성된 ConvNeXt는 주요 벤치마크에서 Swin Transformer와 대등하거나 오히려 더 나은 성능을 달성했다.2 이는 트랜스포머의 성능 향상에 기여한 상당 부분이 셀프 어텐션 자체보다는, 우수한 거시적/미시적 아키텍처 설계와 진보된 훈련 방법론에 있었음을 강력하게 시사했다. ConvNeXt는 잘 설계된 CNN이 2020년대에도 여전히 SOTA 비전 백본이 될 수 있음을 증명했으며 2, 이후 마스크드 오토인코더(MAE) 방식의 자기 지도 학습과 결합된 ConvNeXt V2로 더욱 발전했다.48

ViT에서 시작하여 DeiT/Swin을 거쳐 ConvNeXt에 이르는 이 과정은 단순히 한 아키텍처가 다른 아키텍처를 대체하는 선형적인 발전이 아니었다. 이는 CNN이라는 '정(Thesis)'에 대해 ViT라는 '반(Antithesis)'이 등장하고, 이 둘의 장점을 결합하려는 Swin/DeiT와 같은 '합(Synthesis)'의 노력이 이어진 변증법적 발전 과정으로 볼 수 있다. 더 나아가 ConvNeXt는 ViT/Swin의 성공 요인(설계 원칙)을 다시 CNN에 적용하여 새로운 '정'을 창조했다. 이 과정에서 연구 커뮤니티는 '컨볼루션 대 어텐션'이라는 이분법적 대결을 넘어, 성공적인 비전 모델을 구성하는 근본적인 설계 원칙(예: 계층 구조, 역 병목, 특정 활성화/정규화 패턴, 고급 훈련 전략 등)이 무엇인지에 대한 깊은 통찰을 얻게 되었다. Swin이 CNN처럼 동작하는 트랜스포머이고 ConvNeXt가 트랜스포머처럼 설계된 CNN이라는 점에서, 두 패러다임의 경계는 점차 흐려지고 있으며, 미래는 두 패러다임의 장점을 모두 취하는 하이브리드 아키텍처의 시대가 될 것임을 예고하고 있다.

| 모델                 | 해결하고자 한 핵심 문제                                      | 핵심 혁신 기술                                        | 성능/효율성에 미친 영향                                      |
| -------------------- | ------------------------------------------------------------ | ----------------------------------------------------- | ------------------------------------------------------------ |
| **ViT (Baseline)**   | CNN의 지역적 한계                                            | 이미지의 패치화 및 순수 트랜스포머 적용               | 대규모 데이터에서 CNN을 능가하는 잠재력 증명. 데이터 의존성, 이차적 복잡도 문제 노출. |
| **DeiT**             | 데이터 비효율성 (대규모 사전 훈련 필요)                      | 지식 증류 (CNN 교사 모델 & 증류 토큰)                 | ImageNet-1K만으로 경쟁력 있는 훈련 가능. ViT의 실용성 대폭 향상. |
| **Swin Transformer** | 고밀도 예측 부적합 (고정 스케일), 고해상도 이미지 처리 시 이차적 복잡도 | 계층적 특징 맵 & 이동 윈도우 자기 주의 (W-MSA/SW-MSA) | 선형 복잡도 달성. 객체 탐지/분할을 위한 범용 백본으로 자리매김. SOTA 성능 달성. |
| **ConvNeXt (반응)**  | 트랜스포머 성공의 본질 규명 (어텐션 vs. 설계)                | CNN을 트랜스포머 설계 원칙으로 현대화                 | 순수 CNN으로 Swin Transformer 성능 능가. 설계/훈련 방식의 중요성 입증. |

Table 2: 주요 ViT 변형 모델 아키텍처 혁신 요약. 이 표는 ViT 이후 등장한 주요 모델들이 어떤 문제를 해결하기 위해 어떤 혁신을 도입했는지 요약한다. 각 모델은 이전 모델의 한계를 명확히 겨냥하고 있으며, 이들의 연속적인 등장은 아키텍처 설계에 대한 이해를 심화시키는 변증법적 발전 과정을 보여준다.2


ViT와 그 후속 모델들의 등장은 이미지 분류를 넘어 컴퓨터 비전의 거의 모든 영역에 영향을 미쳤다. 트랜스포머의 전역적 정보 처리 능력과 유연성은 기존 CNN 기반 방법론들이 어려움을 겪었던 문제들을 해결할 새로운 가능성을 열었다. 이 장에서는 객체 탐지 및 의미론적 분할과 같은 고밀도 예측 작업부터, 고도의 정밀성이 요구되는 의료 영상 분석, 그리고 실시간성과 안전성이 무엇보다 중요한 자율주행 분야에 이르기까지 ViT 기반 아키텍처가 어떻게 적용되고 있는지 구체적인 사례를 통해 살펴본다.


이미지 내 모든 픽셀에 대해 예측을 수행해야 하는 고밀도 예측(dense prediction) 작업은 ViT에게 초기에는 큰 도전이었다. ViT의 고정된 단일 스케일 특징 맵과 고해상도 이미지에 대한 높은 계산 비용 때문이었다.39

초기 접근법 중 하나인 **DETR(DEtection TRansformer)**은 순수 트랜스포머 인코더-디코더 구조를 사용하여 객체 탐지 문제를 완전히 새로운 방식으로 접근했다.51 복잡한 후처리 과정이나 RPN(Region Proposal Network)과 같은 구성 요소를 제거하고, 학습 가능한 '객체 쿼리(object queries)'를 통해 직접 바운딩 박스를 예측하는 엔드투엔드(end-to-end) 파이프라인을 제시했다. 이는 설계의 우아함으로 주목받았지만, 훈련 수렴이 느리고 작은 객체에 대한 성능이 저조하다는 단점이 있었다.51

이러한 한계를 극복하고 트랜스포머를 고밀도 예측 작업의 주류로 이끈 것은 **Swin Transformer**였다.39 Swin Transformer의 계층적 특징 맵 구조는 다양한 스케일의 정보를 제공하여, 객체 탐지 및 분할에 이상적인 백본(backbone) 역할을 했다. Swin Transformer 백본을 UperNet과 같은 정교한 디코더와 결합한 모델은 ADE20K 의미론적 분할 벤치마크에서 SOTA 성능을 달성했다.39 또한, Swin Transformer 인코더와 U-Net 스타일 디코더를 결합한 

**Swin-Unet**과 같은 하이브리드 모델은 분할 작업에서 강력한 성능을 보여주었다.42

현재 많은 최신 접근법들은 CNN과 ViT의 장점을 결합한 **하이브리드 모델** 형태를 띤다.29 예를 들어, CNN을 초기 특징 추출기로 사용하여 강력한 지역적 특징을 뽑아내고, 이를 ViT 기반의 헤드에 전달하여 전역적 맥락을 모델링하는 방식이다. 

**CN-ViT**는 CNN 특징 추출기와 ViT 구조를 결합하여 Mini COCO 데이터셋에서 우수한 성능을 보였다.50 이러한 하이브리드 접근은 CNN의 효율적인 지역 특징 추출 능력과 트랜스포머의 강력한 전역 관계 모델링 능력을 모두 활용하는 실용적인 해결책으로 자리 잡고 있다.

| 과제           | 데이터셋          | 모델                           | 백본      | 핵심 성능 지표 | 값     |
| -------------- | ----------------- | ------------------------------ | --------- | -------------- | ------ |
| 이미지 분류    | ImageNet          | ConvNeXt-L                     | -         | Top-1 Accuracy | 87.8%  |
| 이미지 분류    | ImageNet          | Swin-L (ImageNet-22k pretrain) | -         | Top-1 Accuracy | 87.3%  |
| 객체 탐지      | COCO test-dev     | Swin-L (HTC++)                 | Swin-L    | box AP         | 58.7   |
| 의미론적 분할  | ADE20K val        | Swin-L (UperNet)               | Swin-L    | mIoU           | 53.5   |
| 의료 영상 분할 | Kidney Tumor (CT) | TAU-Net3+                      | ViT-based | Dice Score     | 0.9638 |

Table 3: 주요 비전 과제별 ViT 기반 모델 성능. 이 표는 이미지 분류, 객체 탐지, 의미론적 분할 등 주요 벤치마크에서 ViT 계열 모델들이 달성한 SOTA 수준의 성능을 구체적인 수치로 보여준다. Swin Transformer와 ConvNeXt가 다양한 과제에서 높은 성능을 기록했으며, 의료 영상과 같은 특정 도메인에서도 뛰어난 결과를 보이고 있음을 확인할 수 있다.2


의료 영상(MRI, CT, 디지털 병리 슬라이드 등) 분석은 AI 기술의 사회적 기여도가 매우 높은 분야 중 하나이다.55 ViT는 넓은 범위에 걸쳐 있는 장기나 불규칙한 형태의 병변을 분할하는 데 필수적인 전역적 맥락과 장거리 의존성을 포착하는 데 강점이 있어 이 분야에서 큰 기대를 모았다.54

하지만 의료 영상 분야는 ViT에게 두 가지 큰 난관을 제시했다. 첫째, 대부분의 의료 영상 데이터셋은 크기가 작고 데이터 수집 비용이 비싸다. 이는 대규모 데이터에 의존하는 순수 ViT의 약점을 부각시킨다.54 둘째, 진단에 있어 미세한 질감이나 경계와 같은 고주파(high-frequency) 정보가 매우 중요한데, 패치 단위로 정보를 처리하는 ViT는 이러한 지역적 세부 정보를 포착하는 데 어려움을 겪을 수 있다.60

이러한 이유로, 의료 영상 분석 분야에서는 순수 ViT보다 CNN과 ViT를 결합한 **하이브리드 비전 트랜스포머(Hybrid Vision Transformers, HVTs)**가 지배적인 접근법으로 자리 잡았다.54 HVT는 CNN을 통해 이미지의 견고한 지역적 특징을 효율적으로 추출하고, ViT를 통해 이들 특징 간의 전역적 관계를 모델링함으로써 두 아키텍처의 장점을 모두 취한다.54

구체적인 응용 사례는 다음과 같다 54:

- **분할(Segmentation):** 의료 영상 분할은 HVT가 가장 활발하게 적용되는 분야이다. **TransUNet**, **Swin-Unet** 57, 

  **UNETR** 등은 U-Net 아키텍처의 인코더나 디코더, 혹은 둘 모두를 트랜스포머 기반으로 대체하여 성능을 향상시킨다. 뇌종양 분할에서는 **DenseTrans**, **3DCATBraTs**와 같은 모델들이 Swin Transformer를 3D MRI 영상에 맞게 변형하여 높은 분할 정확도(Dice score)를 달성했다. 피부 병변 분할에서는 **FAT-net**이 CNN과 트랜스포머로 구성된 듀얼 인코더를 사용하여 지역 및 전역 정보를 동시에 활용한다.

- **분류(Classification):** ViT 기반 모델은 디지털 병리 이미지로부터 유방암을 분류하거나 3, MRI/CT 스캔에서 다양한 종류의 종양을 분류하는 데 사용된다.62 사전 지식 없이 사용하는 제로샷(zero-shot) 성능은 제한적일 수 있지만, 해당 도메인 데이터로 간단히 미세 조정(fine-tuning)하는 것만으로도 매우 우수한 결과를 얻을 수 있다.3

- **기타 과제:** 이 외에도 ViT는 저화질 의료 영상의 품질을 개선하는 이미지 복원(image restoration) 59, 이미지 재구성(reconstruction), 그리고 의료 영상 판독문을 자동으로 생성하는 보고서 생성(report generation) 등 다양한 작업에 적용되고 있다.56


자율주행(Autonomous Driving, AD) 시스템의 핵심은 주변 환경을 정확하게 인식하는 '인식(perception)' 기술이며, ViT는 이 분야에 혁신을 가져오고 있다.63 복잡하고 동적인 주행 환경을 이해하기 위해서는 차량, 보행자, 차선 등 다양한 객체들 간의 공간적, 시간적 관계를 종합적으로 파악해야 하는데, ViT의 장거리 의존성 모델링 능력은 이에 매우 적합하다.63

ViT는 자율주행의 다양한 인식 과제에 적용되고 있다:

- **2D/3D 객체 탐지:** 다중 카메라로부터 입력받은 이미지들을 종합하여 차량 주변 객체들의 3D 바운딩 박스를 정확하게 예측하는 데 ViT가 활용된다.65

- **차선 탐지:** **LSTR**, **CurveFormer**와 같은 모델들은 트랜스포머의 쿼리 메커니즘을 사용하여 2D 이미지로부터 직접 차선 구조를 생성한다.63

- **분할:** **Panoptic SegFormer**는 주행 환경의 모든 픽셀을 의미론적(semantic) 클래스와 개별 객체(instance)로 동시에 분할하는 파놉틱 분할(panoptic segmentation)을 수행한다.63 특히, 다중 시점의 2D 이미지를 차량 중심의 조감도(Bird's-Eye-View, BEV) 공간으로 변환하여 분할 맵을 생성하는 

  **BEVSegFormer**와 같은 모델들은 ViT의 어텐션 메커니즘을 효과적으로 활용한다.63

하지만 자율주행 분야에서 ViT를 사용하는 데에는 매우 현실적이고 치명적인 제약이 따른다: 바로 **실시간 추론(real-time inference)** 문제이다.65 자율주행 시스템은 안전을 위해 최소 초당 10회(10Hz) 이상의 빈도로 주변 환경을 업데이트해야 하며, 이 모든 계산은 제한된 자원을 가진 차량 내 온보드 컴퓨터에서 수행되어야 한다.65 트랜스포머의 높은 계산 비용, 특히 고해상도의 다중 카메라 입력을 처리할 때의 연산량은 이 요구사항을 충족시키는 데 큰 걸림돌이 된다.65

따라서 이 분야의 연구는 단순히 정확도를 높이는 것을 넘어, **정확도와 추론 시간(latency) 간의 최적의 균형점**을 찾는 데 집중되고 있다.65 연구자들은 모델 아키텍처를 경량화하고, 전처리/후처리 과정을 최적화하며, float16과 같은 낮은 정밀도(precision)로 모델을 양자화하여 실제 배포 환경에서의 성능을 평가하는 등, 모델 중심적 접근에서 벗어나 시스템 전체를 고려하는 방향으로 나아가고 있다. 이는 자율주행과 같이 안전이 최우선인 실제 응용 분야에서는, AI 모델이 전체 시스템의 한 구성 요소로서 안정적으로 작동하는 것이 순수한 성능 지표보다 더 중요함을 보여준다. 이러한 시스템 중심적 관점은 학술적 벤치마크 성능에 집중하는 연구와 실제 산업 응용 연구 간의 중요한 차이점을 드러낸다.


비전 트랜스포머는 컴퓨터 비전 분야에 깊은 족적을 남겼지만, 기술의 발전은 멈추지 않는다. ViT가 CNN의 한계를 넘어서는 과정에서 스스로의 한계(이차적 복잡도)를 드러냈고, 이제 연구 커뮤니티는 트랜스포머의 장점을 계승하면서도 단점을 극복할 수 있는 차세대 아키텍처를 모색하고 있다. 이 장에서는 트랜스포머의 유력한 후계자로 떠오르는 상태 공간 모델(SSM)을 소개하고, 아키텍처의 경계를 넘어 다중 모달리티로 확장되는 미래를 조망하며, 지금까지의 논의를 바탕으로 종합적인 결론과 제언을 제시한다.


ViT가 전역적 맥락 포착이라는 문제를 해결했지만, 그 대가로 치러야 했던 이차적 계산 복잡도는 여전히 큰 부담으로 남아있다.67 특히 초고해상도 이미지나 긴 비디오 시퀀스를 처리해야 하는 분야에서 이는 치명적인 병목 현상을 유발한다. 이러한 배경에서, 전역적 의존성을 모델링하면서도 선형적인 계산 복잡도를 가지는 새로운 아키텍처, **상태 공간 모델(State Space Models, SSMs)**이 주목받고 있다.67


SSM은 고전 제어 이론에서 영감을 받은 시퀀스 모델링 기법이다. 이는 입력 시퀀스를 받아 내부적인 '상태(state)' 벡터 h를 시간의 흐름에 따라 점진적으로 업데이트하고, 이 상태를 바탕으로 출력을 생성하는 방식으로 동작한다. Mamba와 같은 최신 SSM은 트랜스포머처럼 장거리 의존성을 효과적으로 포착하면서도, 시퀀스 길이에 대해 선형적(O(N)) 또는 거의 선형적인(O(NlogN)) 계산 복잡도를 갖는다는 혁신적인 장점을 가진다.67 또한, 병렬 처리가 용이한 CNN처럼 훈련할 수 있으면서, 추론 시에는 순차적 정보를 효율적으로 처리하는 RNN처럼 동작할 수 있는 유연한 구조를 가지고 있다.70


본래 1D 시퀀스 처리에 적합한 SSM을 2D 이미지에 적용하기 위해 **VMamba(Visual State Space Model)**와 같은 모델이 제안되었다.69 VMamba의 핵심은 **교차 스캔 모듈(Cross-Scan Module, CSM)**이다. CSM은 1D 스캔 방식을 2D 이미지에 확장하기 위해, 이미지의 네 모서리에서 시작하여 각각 반대 방향으로 특징 맵을 스캔하고 그 결과를 통합한다.69 이를 통해 각 픽셀이 이미지의 모든 다른 픽셀로부터 정보를 수집하게 하여, 선형 복잡도를 유지하면서도 전역적인 수용장을 효과적으로 구현한다.


SSM 기반 비전 모델들의 초기 결과는 매우 유망하다. **GroupMamba**, **VMamba**와 같은 모델들은 기존의 CNN 및 ViT 모델들과 비교했을 때, 더 적은 파라미터로 더 높은 정확도를 달성하며 뛰어난 정확도-효율성 트레이드오프를 보여준다.67 특히 ViT가 처리하기 어려운 초고해상도 의료 영상이나 위성 이미지 분석과 같은 분야에서 SSM의 잠재력은 매우 크다.67 또한, 이벤트 카메라 데이터 처리 68나 인간 행동 분석(HumMUSS 71)과 같은 동영상 기반 작업에서도 기존 트랜스포머 기반 모델보다 빠른 훈련 속도와 뛰어난 일반화 성능을 보이며 그 적용 범위를 넓혀가고 있다.

CNN(선형 복잡도, 지역적 수용장)에서 ViT(이차적 복잡도, 전역적 수용장)로, 그리고 다시 SSM(선형 복잡도, 전역적 수용장)으로 이어지는 이 흐름은, 컴퓨터 비전 연구가 '전역적 맥락'과 '계산 효율성'이라는 두 마리 토끼를 모두 잡기 위해 끊임없이 진화하고 있음을 보여준다. SSM은 이 오랜 탐구의 가장 최신 이정표라 할 수 있다.


트랜스포머 아키텍처의 가장 큰 장점 중 하나는 특정 데이터 도메인에 얽매이지 않는 범용성이다. 토큰화(tokenization)만 가능하다면 어떤 종류의 데이터든 처리할 수 있는 이 유연성은, 이미지, 텍스트, 음성 등 서로 다른 종류의 데이터를 하나의 모델 안에서 통합적으로 처리하는 **다중 모달리티(multimodal) 학습**에 매우 이상적이다.27

ViT는 이미 시각 정보와 텍스트 정보를 결합하는 다양한 다중 모달리티 작업에 활발히 사용되고 있다.19 예를 들어, 이미지의 내용을 설명하는 문장을 생성하거나, 텍스트 쿼리에 맞는 이미지를 검색하는 작업 등이 이에 해당한다.

미래의 AI는 단일 모달리티를 넘어, 인간처럼 다양한 감각 정보를 종합하여 세상을 이해하고 상호작용하는 방향으로 발전할 것이다. 이러한 맥락에서, **UniT** 72나 **FUTURIST** 72와 같이 이미지, 비디오, 텍스트 등 임의의 모달리티 조합을 처리할 수 있는 통합 트랜스포머 프레임워크에 대한 연구가 활발히 진행되고 있다. 자율주행 차량이 카메라, LiDAR, 레이더 데이터를 융합하여 주변 환경을 인식하거나, 의료 AI가 환자의 영상 데이터와 전자의무기록(EMR) 텍스트를 함께 분석하여 더 정확한 진단을 내리는 미래는 이러한 다중 모달리티 트랜스포머 기술에 의해 현실화될 것이다.62


CNN에서 ViT로, 그리고 SSM으로 이어지는 컴퓨터 비전 아키텍처의 여정은 '하나의 정답'이 아닌, 문제의 특성에 따라 최적의 도구가 달라지는 '다양성의 시대'로 진입하고 있음을 보여준다. 각 아키텍처는 성능, 효율성, 귀납적 편향이라는 트레이드오프 공간에서 서로 다른 지점을 차지하고 있다. 이를 바탕으로 특정 응용에 적합한 아키텍처를 선택하기 위한 제언은 다음과 같다.

- **CNN (예: ConvNeXt, YOLO):** 계산 효율성, 낮은 지연 시간, 그리고 중소 규모 데이터셋에서의 강력한 성능이 최우선 순위일 때 여전히 최고의 선택이다. 엣지 컴퓨팅, 실시간 모바일 애플리케이션 등 자원이 제한된 환경에서 CNN의 강력한 귀납적 편향은 데이터 부족 문제를 완화하는 큰 장점으로 작용한다.5
- **ViT (예: Swin Transformer, DeiT):** 대규모 데이터셋을 활용할 수 있고, 복잡한 전역적 맥락을 포착하는 것이 성능에 결정적인 대규모 문제에 가장 적합하다. 자원의 제약이 없다면, ViT 계열 모델들은 가장 높은 성능의 상한선을 제공한다.7
- **하이브리드 모델 (HVTs):** 의료 영상 분석과 같이 데이터가 제한적이면서도 높은 정확성과 견고성이 요구되는 대부분의 현실 세계 응용 분야에서 가장 실용적인 선택이다. CNN의 지역적 특징 추출 능력과 ViT의 전역적 관계 모델링 능력을 결합하여, 각 패러다임의 장점만을 취하는 균형 잡힌 접근법이다.54
- **SSM (예: VMamba):** 극도로 긴 시퀀스(예: 장시간 비디오)나 초고해상도 이미지를 다루어 트랜스포머의 이차적 복잡도가 감당 불가능해지는 문제에 대한 차세대 해결책이다. 전역적 맥락과 선형 효율성을 동시에 달성하고자 하는 미래 지향적 연구 및 응용에 적합하다.67

미래의 비전 백본은 단일 아키텍처가 지배하는 형태가 아니라, 다양한 연산 블록(컨볼루션, 윈도우 어텐션, 전역 어텐션, SSM 등)을 유연하게 조합하는 **'구성 가능한(composable)'** 형태가 될 가능성이 높다. 특정 과제와 하드웨어 제약에 맞춰 최적의 아키텍처를 동적으로 구성하는 시대가 다가오고 있으며, 이는 HVT의 성공과 ConvNeXt의 모듈식 설계에서 이미 그 단초를 엿볼 수 있다.


비전 트랜스포머(ViT)의 등장은 컴퓨터 비전 분야에 단순한 성능 향상을 넘어선 근본적인 패러다임의 전환을 가져왔다. 10년 이상 지속된 CNN의 독주에 종지부를 찍고, 이미지를 시퀀스로 해석하는 새로운 관점을 제시함으로써 연구의 지평을 넓혔다.

본 고찰을 통해 살펴본 바와 같이, ViT의 여정은 일방적인 승리가 아닌, 기존 패러다임과의 치열한 대화와 상호 학습의 과정이었다. 초기 ViT는 대규모 데이터에 대한 의존성과 높은 계산 비용이라는 명확한 한계를 보였지만, 이는 곧바로 DeiT의 데이터 효율적인 증류 기법과 Swin Transformer의 계층적, 효율적 아키텍처라는 혁신적인 해결책을 낳았다. 이에 대한 반향으로 등장한 ConvNeXt는 트랜스포머의 성공 요인이었던 설계 원칙들을 CNN에 역으로 적용하여, 아키텍처의 본질에 대한 논의를 한 단계 더 심화시켰다.

결과적으로, 'CNN 대 트랜스포머'라는 이분법적 대결 구도는 점차 희미해지고 있다. 대신, 각 아키텍처가 가진 고유의 장단점을 이해하고, 주어진 문제의 특성(데이터 규모, 실시간성, 자원 제약 등)에 따라 최적의 도구를 선택하거나 이들을 융합하는 실용적인 접근법이 주류가 되고 있다. 객체 탐지, 의료 영상, 자율주행 등 다양한 응용 분야에서 하이브리드 모델이 강세를 보이는 현상이 이를 증명한다.

이제 우리의 시선은 트랜스포머를 넘어, 전역적 맥락과 선형 효율성을 동시에 달성하려는 상태 공간 모델(SSM)과 같은 차세대 주자들에게로 향하고 있다. 또한, 트랜스포머의 범용성은 이미지와 텍스트를 아우르는 다중 모달리티 AI의 시대를 가속화하고 있다.

비전 트랜스포머의 가장 중요한 유산은 아키텍처 그 자체가 아니라, 그것이 촉발한 혁신의 물결과 패러다임 간의 생산적인 대화일 것이다. ViT는 컴퓨터 비전 연구자들에게 기존의 가정에 의문을 제기하고, 더 근본적인 원리로부터 시각적 표현 학습을 재고하도록 만들었다. 이러한 지적 자극은 앞으로도 더 강력하고, 효율적이며, 범용적인 비전 모델의 등장을 이끌어내는 원동력이 될 것이다.


1. (PDF) An Image is Worth 16x16 Words: Transformers for Image ..., accessed July 18, 2025, https://scispace.com/papers/an-image-is-worth-16x16-words-transformers-for-image-v85s5ahlww
2. A ConvNet for the 2020s, accessed July 18, 2025, https://arxiv.org/pdf/2201.03545
3. An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale, accessed July 18, 2025, https://www.researchgate.net/publication/344828174_An_Image_is_Worth_16x16_Words_Transformers_for_Image_Recognition_at_Scale
4. An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale, accessed July 18, 2025, https://vitalab.github.io/article/2020/10/15/vision_transformer.html
5. What are the limitations of CNN in computer vision? - Milvus, accessed July 18, 2025, https://milvus.io/ai-quick-reference/what-are-the-limitations-of-cnn-in-computer-vision
6. Convolutional Neural Networks (CNN) in Deep Learning - Analytics Vidhya, accessed July 18, 2025, https://www.analyticsvidhya.com/blog/2021/05/convolutional-neural-networks-cnn/
7. Vision Transformer vs. CNN: A Comparison of Two Image Processing Giants - Medium, accessed July 18, 2025, https://medium.com/@hassaanidrees7/vision-transformer-vs-cnn-a-comparison-of-two-image-processing-giants-d6c85296f34f
8. What are the limitations of CNN in computer vision? - Zilliz Vector Database, accessed July 18, 2025, https://zilliz.com/ai-faq/what-are-the-limitations-of-cnn-in-computer-vision
9. A Review of Transformer-Based Models for Computer Vision Tasks: Capturing Global Context and Spatial Relationships - arXiv, accessed July 18, 2025, https://arxiv.org/html/2408.15178v1
10. Convolutional Neural Network (CNN) in Machine Learning - GeeksforGeeks, accessed July 18, 2025, https://www.geeksforgeeks.org/deep-learning/convolutional-neural-network-cnn-in-machine-learning/
11. Disadvantages of CNN models. CNN (Convolutional Neural Network) is... | by Sandeep Bhuiya, accessed July 18, 2025, https://sandeep-bhuiya01.medium.com/disadvantages-of-cnn-models-95395fe9ae40
12. What are the current problems with the use of convolutional neural networks (CNN) for vision? - Quora, accessed July 18, 2025, https://www.quora.com/What-are-the-current-problems-with-the-use-of-convolutional-neural-networks-CNN-for-vision
13. Understanding Vision Transformers (ViT) | SJ Innovation, accessed July 18, 2025, https://sjinnovation.com/what-vision-transformer-vit-what-are-vits-real-world-applications
14. Arxiv Dives - Vision Transformers (ViT) - Oxen.ai, accessed July 18, 2025, https://www.oxen.ai/blog/arxiv-dives-vision-transformers-vit
15. Vision Transformer(ViT) - ITPE * JackerLab, accessed July 18, 2025, https://itpe.jackerlab.com/entry/Vision-TransformerViT
16. [개념 정리] 비전 트랜스포머 / Vision Transformer(ViT) (1/2) - Innov_AI_te, accessed July 18, 2025, [https://jaylala.tistory.com/entry/%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC-%EB%B9%84%EC%A0%84-%ED%8A%B8%EB%9E%9C%EC%8A%A4%ED%8F%AC%EB%A8%B8-Vision-TransformerViT-12](https://jaylala.tistory.com/entry/개념-정리-비전-트랜스포머-Vision-TransformerViT-12)
17. Introduction to ViT (Vision Transformers): Everything You Need to Know - Lightly, accessed July 18, 2025, https://www.lightly.ai/blog/vision-transformers-vit
18. Vision Transformer (ViT) Architecture - GeeksforGeeks, accessed July 18, 2025, https://www.geeksforgeeks.org/deep-learning/vision-transformer-vit-architecture/
19. Introduction to Vision Transformers (ViT) - Encord, accessed July 18, 2025, https://encord.com/blog/vision-transformers/
20. Vision Transformers (ViT) Explained - Pinecone, accessed July 18, 2025, https://www.pinecone.io/learn/series/image-search/vision-transformers/
21. [스노피 AI] Vision Transformer 쉽게 이해하기 - 1. Introduction - Wafour (와포) 블로그, accessed July 18, 2025, [https://wafour.tistory.com/entry/%EC%8A%A4%EB%85%B8%ED%94%BC-AI-Vision-Transformer-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-1-Introduction](https://wafour.tistory.com/entry/스노피-AI-Vision-Transformer-쉽게-이해하기-1-Introduction)
22. An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale | OpenReview, accessed July 18, 2025, https://openreview.net/forum?id=YicbFdNTTy
23. 쉽게 이해하는 ViT(Vision Transformer) 논문 리뷰 | An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale - 귱 이야기, accessed July 18, 2025, [https://hipgyung.tistory.com/entry/%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-ViTVision-Transformer-%EB%85%BC%EB%AC%B8-%EB%A6%AC%EB%B7%B0-An-Image-is-Worth-16x16-Words-Transformers-for-Image-Recognition-at-Scale](https://hipgyung.tistory.com/entry/쉽게-이해하는-ViTVision-Transformer-논문-리뷰-An-Image-is-Worth-16x16-Words-Transformers-for-Image-Recognition-at-Scale)
24. ViT 살펴보기 | Vision Transformer - Standing-O, accessed July 18, 2025, https://standing-o.github.io/posts/vision-transformer/
25. viso.ai, accessed July 18, 2025, [https://viso.ai/deep-learning/vision-transformer-vit/#:~:text=Vision%20Transformers%20(ViT)%20is%20an,and%20a%20feed%2Dforward%20layer.](https://viso.ai/deep-learning/vision-transformer-vit/#:~:text=Vision Transformers (ViT) is an,and a feed-forward layer.)
26. Vision Transformers (ViT) in Image Recognition: Full Guide - viso.ai, accessed July 18, 2025, https://viso.ai/deep-learning/vision-transformer-vit/
27. Vision Transformers vs CNNs at the Edge - Edge AI and Vision ..., accessed July 18, 2025, https://www.edge-ai-vision.com/2024/03/vision-transformers-vs-cnns-at-the-edge/
28. Vision Transformer: What It Is & How It Works [2024 Guide] - V7 Labs, accessed July 18, 2025, https://www.v7labs.com/blog/vision-transformer-guide
29. [D] Have transformers won in Computer Vision? : r/MachineLearning - Reddit, accessed July 18, 2025, https://www.reddit.com/r/MachineLearning/comments/1hzn0gg/d_have_transformers_won_in_computer_vision/
30. Vision Transformers - The Future of Computer Vision! | ResearchGate, accessed July 18, 2025, https://www.researchgate.net/post/Vision_Transformers-The_Future_of_Computer_Vision
31. Vision Transformer Model: How It Works & Benefits - Webisoft, accessed July 18, 2025, https://webisoft.com/articles/vision-transformer-model/
32. DeiT-LT: Distillation Strikes Back for Vision Transformer Training on Long-Tailed Datasets - CVF Open Access, accessed July 18, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Rangwani_DeiT-LT_Distillation_Strikes_Back_for_Vision_Transformer_Training_on_Long-Tailed_CVPR_2024_paper.pdf
33. [2404.02900] DeiT-LT Distillation Strikes Back for Vision Transformer Training on Long-Tailed Datasets - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2404.02900
34. DeiT-LT: Distillation Strikes Back for Vision Transformer Training on Long-Tailed Datasets, accessed July 18, 2025, https://arxiv.org/html/2404.02900v1
35. Exploring DeiT: A Review and PyTorch Guide to Data-Efficient ..., accessed July 18, 2025, https://medium.com/@ovularslan/exploring-deit-a-review-and-pytorch-guide-to-data-efficient-image-transformers-2fa648677654
36. Tensorflow Image Classifier - Data-efficient Image Transformers - Mike Polinowski, accessed July 18, 2025, https://mpolinowski.github.io/docs/IoT-and-Machine-Learning/ML/2023-08-03-tensorflow-i-know-flowers-deit/2023-08-03/
37. 【ML Paper】DeiT: Summary - Zenn, accessed July 18, 2025, https://zenn.dev/yuto_mo/articles/8bc30de613bb4f
38. Data-efficient image Transformers: A promising new technique for image classification, accessed July 18, 2025, https://ai.meta.com/blog/data-efficient-image-transformers-a-promising-new-technique-for-image-classification/
39. Swin Transformer: Hierarchical Vision Transformer Using Shifted Windows - CVF Open Access, accessed July 18, 2025, https://openaccess.thecvf.com/content/ICCV2021/papers/Liu_Swin_Transformer_Hierarchical_Vision_Transformer_Using_Shifted_Windows_ICCV_2021_paper.pdf
40. Swin Transformer: Hierarchical Vision Transformer using Shifted ..., accessed July 18, 2025, https://arxiv.org/abs/2103.14030
41. [2201.03545] A ConvNet for the 2020s - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2201.03545
42. Swin-Transformer-based Unet architecture for semantic segmentation with Pytorch code. | by Ashish bisht | Medium, accessed July 18, 2025, https://medium.com/@ashishbisht0307/swin-transformer-based-unet-architecture-for-semantic-segmentation-with-pytorch-code-91e779334e8e
43. Swin Transformer: Hierarchical Vision Transformer using Shifted Windows | Papers With Code, accessed July 18, 2025, https://paperswithcode.com/paper/swin-transformer-hierarchical-vision
44. [2111.09883] Swin Transformer V2: Scaling Up Capacity and Resolution - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2111.09883
45. [2108.10257] SwinIR: Image Restoration Using Swin Transformer - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2108.10257
46. [2404.13270] StrideNET: Swin Transformer for Terrain Recognition with Dynamic Roughness Extraction - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2404.13270
47. arXiv:2311.15599v2 [cs.CV] 18 Mar 2024, accessed July 18, 2025, https://arxiv.org/pdf/2311.15599
48. ConvNeXt V2: Co-designing and Scaling ConvNets with Masked Autoencoders - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2301.00808
49. facebookresearch/deit: Official DeiT repository - GitHub, accessed July 18, 2025, https://github.com/facebookresearch/deit
50. CN-ViT- Visual Object Detection with VisionTransformer | Applied and Computational Engineering, accessed July 18, 2025, https://www.ewadirect.com/proceedings/ace/article/view/15913
51. YOLO vs Vision Transformers: A Comparative Review of Object ..., accessed July 18, 2025, https://theacademic.in/wp-content/uploads/2025/06/118.pdf
52. Swin Transformer - Hugging Face, accessed July 18, 2025, https://huggingface.co/docs/transformers/model_doc/swin
53. SSformer: A Lightweight Transformer for Semantic Segmentation - arXiv, accessed July 18, 2025, https://arxiv.org/pdf/2208.02034
54. A Recent Survey of Vision Transformers for Medical Image ... - arXiv, accessed July 18, 2025, https://arxiv.org/pdf/2312.00634
55. Comparison of Vision Transformers and Convolutional Neural Networks in Medical Image Analysis: A Systematic Review - PMC - PubMed Central, accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11393140/
56. Vision transformer architecture and applications in digital health: a tutorial and survey - PMC, accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10333157/
57. A Recent Survey of Vision Transformers for Medical Image Segmentation - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2312.00634
58. Leveraging Inductive Bias in ViT for Medical Image Diagnosis - BMVA Archive, accessed July 18, 2025, https://bmva-archive.org.uk/bmvc/2024/papers/Paper_670/paper.pdf
59. Vision Transformers in Image Restoration: A Survey - MDPI, accessed July 18, 2025, https://www.mdpi.com/1424-8220/23/5/2385
60. Laplacian-Former: Overcoming the Limitations of Vision Transformers in Local Texture Detection - PMC, accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10830169/
61. Systematic Review of Hybrid Vision Transformer Architectures for Radiological Image Analysis | medRxiv, accessed July 18, 2025, https://www.medrxiv.org/content/10.1101/2024.06.21.24309265v1.full-text
62. [2502.05517] Evaluation of Vision Transformers for Multimodal Image Classification: A Case Study on Brain, Lung, and Kidney Tumors - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2502.05517
63. A Survey of Vision Transformers in Autonomous Driving: Current Trends and Future Directions - arXiv, accessed July 18, 2025, https://arxiv.org/html/2403.07542v1
64. Exploring the Role of Transformer Models in Vision, Multi-Modal, and Orbital Data for Autonomous Driving | Applied and Computational Engineering, accessed July 18, 2025, https://www.ewadirect.com/proceedings/ace/article/view/16858
65. Training Strategies for Vision Transformers for Object Detection - CVF Open Access, accessed July 18, 2025, https://openaccess.thecvf.com/content/CVPR2023W/WAD/papers/Singh_Training_Strategies_for_Vision_Transformers_for_Object_Detection_CVPRW_2023_paper.pdf
66. Efficient Vision Transformers for Autonomous Off-Road Perception Systems - Scientific Research Publishing, accessed July 18, 2025, https://www.scirp.org/journal/paperinformation?paperid=136447
67. Making computer vision more efficient with state-space models - MBZUAI, accessed July 18, 2025, https://mbzuai.ac.ae/news/making-computer-vision-more-efficient-with-state-space-models/
68. State Space Models for Event Cameras - CVPR 2024 Open Access Repository, accessed July 18, 2025, https://openaccess.thecvf.com/content/CVPR2024/html/Zubic_State_Space_Models_for_Event_Cameras_CVPR_2024_paper.html
69. VMamba: Visual State Space Model - arXiv, accessed July 18, 2025, https://arxiv.org/html/2401.10166v1
70. State Space Models for Event Cameras - CVF Open Access, accessed July 18, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Zubic_State_Space_Models_for_Event_Cameras_CVPR_2024_paper.pdf
71. HumMUSS: Human Motion Understanding using State Space Models - CVF Open Access, accessed July 18, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Mondal_HumMUSS_Human_Motion_Understanding_using_State_Space_Models_CVPR_2024_paper.pdf
72. Advancing Semantic Future Prediction through Multimodal Visual Sequence Transformers, accessed July 18, 2025, https://arxiv.org/html/2501.08303v1






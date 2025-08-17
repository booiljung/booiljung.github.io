# 상태 공간 모델(Mamba)

## 1.  시퀀스 모델링의 새로운 지평을 열다

### 1.1  시퀀스 모델링의 패러다임: RNN에서 Transformer까지

시퀀스 데이터는 자연어, 음성, 시계열 등 인공지능의 핵심 응용 분야 전반에 걸쳐 나타나는 보편적인 데이터 형태이다. 이러한 순차적 정보의 패턴을 학습하고 미래를 예측하는 시퀀스 모델링은 오랫동안 딥러닝 연구의 중심 과제였다. 초기의 지배적인 패러다임은 순환 신경망(Recurrent Neural Network, RNN)이었다.1 RNN은 내부의 순환 구조를 통해 이전 타임스텝의 정보를 '기억'하고 현재 입력과 결합하여 다음 상태를 결정한다. 이 방식은 추론 시 이전 상태만 알면 다음 출력을 계산할 수 있어 계산 및 메모리 복잡도가 시퀀스 길이에 선형적으로 증가($O(L)$)한다는 장점을 가진다.1 그러나 RNN은 치명적인 약점을 내포하고 있었다. 역전파 과정에서 기울기가 시퀀스를 따라 반복적으로 곱해지면서 너무 작아지거나(기울기 소실, Vanishing Gradients) 너무 커지는(기울기 폭주, Exploding Gradients) 문제가 발생하여, 먼 과거의 정보를 효과적으로 학습하지 못하는 장기 의존성(Long-Range Dependency) 문제를 겪었다.1 또한, 각 타임스텝의 계산이 이전 타임스텝의 완료를 기다려야 하는 순차적 의존성 때문에 훈련 과정의 병렬화가 근본적으로 불가능하여 대규모 데이터 학습에 비효율적이었다.1

2017년, "Attention Is All You Need" 논문에서 제시된 Transformer 아키텍처는 이러한 패러다임을 완전히 바꾸었다.2 Transformer는 순환 구조를 완전히 제거하고, 대신 셀프 어텐션(Self-Attention) 메커니즘을 도입했다. 셀프 어텐션은 시퀀스 내의 모든 토큰 쌍 간의 관계를 직접 계산하여 각 토큰이 다른 모든 토큰을 동시에 참조할 수 있게 한다. 이 혁신적인 접근 방식은 두 가지 큰 이점을 가져왔다. 첫째, 모든 토큰 쌍을 직접 연결함으로써 장기 의존성 문제를 효과적으로 해결했다. 둘째, 순차적 의존성이 사라져 전체 시퀀스에 대한 계산을 병렬로 처리할 수 있게 되면서, GPU를 활용한 대규모 병렬 훈련이 가능해졌다.2

그러나 Transformer의 강력한 성능은 막대한 계산 비용을 대가로 한다. 셀프 어텐션은 시퀀스 길이(`L`)에 대해 이차적인($O(L^2)$) 계산 및 메모리 복잡도를 가진다.1 이는 시퀀스가 길어질수록 계산량이 기하급수적으로 증가하는 '이차 병목 현상(Quadratic Bottleneck)'을 야기한다. 이로 인해 유전체학, 고해상도 오디오, 장편 문서 분석 등 수만에서 수백만 토큰에 이르는 초장문(very long sequence) 데이터를 처리하는 데 근본적인 한계를 드러냈다. 시퀀스 모델링의 발전사를 관통하는 핵심적인 동인은 이처럼 계산 효율성과 모델 표현력 사이의 끊임없는 상충 관계였다. RNN은 선형적 효율성을 확보했지만 표현력의 한계에 부딪혔고, Transformer는 표현력을 극대화하는 대신 이차적 계산 비용을 감수했다.

### 1.2  상태 공간 모델(SSM)의 부상과 Mamba의 등장

Transformer의 계산 비효율성을 극복하기 위해 선형 어텐션(Linear Attention), 게이트 컨볼루션(Gated Convolution) 등 다양한 서브쿼드라틱(sub-quadratic) 모델들이 제안되었으나, 대부분 언어와 같은 핵심적인 데이터 양식에서 Transformer의 성능을 따라잡는 데 어려움을 겪었다.8 이러한 상황에서 제어 공학(Control Engineering) 분야에서 오랫동안 사용되어 온 상태 공간 모델(State Space Model, SSM)이 새로운 대안으로 부상했다.10 SSM은 동적 시스템의 내부 상태 변화를 수학적으로 모델링하는 프레임워크로, RNN의 순환적 특성과 CNN의 병렬적 특성을 모두 가질 수 있는 잠재력을 지니고 있었다.

초기 딥러닝 SSM 연구, 특히 S4(Structured State Space) 모델은 HiPPO라는 이론적 프레임워크를 통해 장기 의존성 문제를 해결하고, 훈련과 추론에 각각 최적화된 계산 방식을 제시하며 그 가능성을 입증했다.13 그러나 S4는 시스템의 동역학이 입력 내용과 무관하게 고정된 선형 시불변(LTI) 시스템이라는 한계로 인해, 내용에 따라 정보의 중요도가 급변하는 텍스트와 같은 이산적 데이터 처리에는 여전히 약점을 보였다.11

이러한 배경 속에서 2023년 말, Mamba 아키텍처가 등장했다.8 Mamba는 기존 SSM의 한계를 극복하고 Transformer급 성능을 달성한 최초의 선형 시간 모델로서 시퀀스 모델링 분야에 큰 파장을 일으켰다. Mamba의 핵심 혁신은 두 가지다. 첫째, '선택적 메커니즘(Selective Mechanism)'을 도입하여 모델 파라미터를 입력에 따라 동적으로 변화시킴으로써, 내용 기반의 정보 필터링을 가능하게 했다. 둘째, 이로 인해 포기해야 했던 병렬 계산의 이점을 '하드웨어 인식 알고리즘(Hardware-Aware Algorithm)'이라는 새로운 접근법으로 되찾아왔다.

특정 아키텍처의 성공은 그 아키텍처가 가진 내재적 편향(inductive bias)이 특정 데이터 양식의 통계적 특성과 잘 부합하기 때문이다. Transformer는 이산적 데이터에서, 초기 SSM은 연속적 데이터에서 강점을 보였다. Mamba의 혁신은 이러한 편향을 데이터 의존적으로 만들어, 더 넓은 범위의 데이터 양식에 효과적으로 대응하려는 시도로 볼 수 있다. 본 보고서는 상태 공간 모델의 이론적 기초부터 S4를 거쳐 Mamba에 이르는 기술적 진화 과정을 심층적으로 추적하고, Mamba의 핵심 혁신을 수학적 원리와 함께 분석한다. 또한, RNN 및 Transformer와의 다각적인 비교와 주요 벤치마크 결과를 통해 Mamba의 성능과 한계를 규명하고, 차세대 시퀀스 모델링의 미래를 전망하고자 한다.

## 2.  상태 공간 모델(SSM)의 이론적 기초

### 2.1  제어 이론에서 딥러닝까지: SSM의 기원

상태 공간 모델(SSM)은 딥러닝 분야에 비교적 최근에 도입되었지만, 그 뿌리는 1960년대 제어 시스템 공학으로 거슬러 올라간다.10 당시 아폴로 계획의 우주선 항법 계산과 같은 복잡한 동적 시스템(dynamic system)을 모델링하고 제어하기 위한 강력한 수학적 프레임워크로 개발되었다.10 SSM의 핵심 아이디어는 시스템의 복잡한 동작을 직접 관찰할 수 없는 소수의 내부 '상태 변수(state variables)'의 시간적 진화로 설명하는 것이다.

여기서 **상태 변수**란 시스템의 미래 동작을 예측하는 데 필요한 최소한의 정보 집합을 의미한다. 예를 들어, 움직이는 자동차의 상태는 현재 위치와 속도라는 두 개의 변수로 완전히 기술될 수 있다.10 이 상태 변수들이 축을 이루는 가상의 N차원 공간을 **상태 공간(state space)**이라 하며, 특정 시점 $t$에서의 시스템 상태는 상태 공간 내의 한 점, 즉 **상태 벡터(state vector)** $h(t)$로 표현된다.10

딥러닝의 관점에서 이 상태 벡터 $h(t)$는 RNN의 은닉 상태(hidden state)와 유사한 역할을 하는 **잠재 상태(latent state)**로 해석될 수 있다.10 이는 시스템의 출력처럼 직접 관찰되지는 않지만, 시스템의 내부 동역학을 지배하고 과거의 모든 정보를 요약하는 핵심적인 변수이다. SSM의 목표는 외부 입력 $x(t)$에 따라 이 잠재 상태 $h(t)$가 어떻게 진화하고, 이 잠재 상태가 어떻게 관찰 가능한 출력 $y(t)$로 이어지는지를 수학적으로 모델링하는 것이다.

### 2.2  연속 시간(Continuous-Time) 표현: 동적 시스템의 수학적 추상화

SSM의 가장 근본적인 형태는 연속적인 시간 $t$에 대한 1차 상미분방정식(Ordinary Differential Equation, ODE)으로 표현된다. 이 표현은 시스템의 동역학을 두 개의 간단하고 강력한 방정식으로 추상화한다.10

첫 번째는 **상태 방정식(State Equation)**으로, 잠재 상태 $h(t)$의 시간적 변화율 $h'(t)$이 현재 상태 $h(t)$와 외부 입력 $x(t)$에 의해 어떻게 결정되는지를 기술한다.
$$
h'(t) = \mathbf{A}h(t) + \mathbf{B}x(t)
$$
두 번째는 **출력 방정식(Output Equation)**으로, 관찰 가능한 출력 $y(t)$이 잠재 상태 $h(t)$와 외부 입력 $x(t)$로부터 어떻게 생성되는지를 기술한다.
$$
y(t) = \mathbf{C}h(t) + \mathbf{D}x(t)
$$
이 방정식들을 구성하는 시스템 행렬 `A`, `B`, `C`, `D`는 각각 다음과 같은 역할을 수행하며, 딥러닝에서는 이 행렬들이 바로 학습 대상이 되는 파라미터가 된다.

- `A` (**상태 행렬**): 시스템의 내부 동역학(internal dynamics)을 결정한다. 외부 입력이 없을 때 상태가 시간에 따라 어떻게 스스로 진화하는지를 나타내며, 시스템의 안정성과 고유한 특성을 좌우한다.7
- `B` (**입력 행렬**): 외부 입력 $x(t)$가 시스템의 잠재 상태 $h(t)$에 미치는 영향을 결정한다. 입력 신호가 내부 상태를 어떻게 '구동'하는지를 모델링한다.7
- `C` (**출력 행렬**): 내부 잠재 상태 $h(t)$가 외부에서 관찰 가능한 출력 $y(t)$으로 어떻게 변환되는지를 결정한다. 즉, 잠재된 정보가 어떻게 표현되는지를 모델링한다.7
- `D` (**피드스루 행렬**): 입력 $x(t)$가 상태를 거치지 않고 출력 $y(t)$에 직접적으로 미치는 영향을 나타낸다. 딥러닝 SSM에서는 이 항이 종종 간단한 스킵 연결(skip connection)로 간주되어 생략되거나 단순화된다.11

### 2.3  이산화(Discretization): 연속에서 이산으로의 변환

연속 시간 ODE 표현은 SSM의 이론적 기반을 제공하지만, 디지털 컴퓨터는 연속적인 신호를 직접 처리할 수 없다. 따라서 시퀀스 모델링에 SSM을 적용하기 위해서는 연속적인 방정식을 이산적인 타임스텝 $k$에 대한 점화식(recurrence relation)으로 변환하는 **이산화(discretization)** 과정이 필수적이다.11

이산화는 **시간 간격(step size)** $Δ$라는 새로운 파라미터를 도입하여, 연속 시간 $t$를 이산적인 시점 $kΔ$로 샘플링하는 과정이다. 이 과정을 통해 연속 시간 SSM 방정식은 다음과 같은 이산 시간 형태로 근사된다.14
$$
h_k = \bar{\mathbf{A}}h_{k-1} + \bar{\mathbf{B}}x_k
$$

$$
y_k = \bar{\mathbf{C}}h_k
$$

여기서 $h_k$는 `k`번째 타임스텝에서의 상태, $x_k$는 입력을 의미하며, $Ā$, $B̄$, $C̄$는 연속 시간 행렬 $A$, $B$, $C$와 시간 간격 $Δ$로부터 유도된 이산 시간 행렬이다.

이산화는 단순한 기술적 변환이 아니라, 모델의 귀납적 편향(inductive bias)을 결정하는 핵심적인 모델링 선택이다. 연속 시간 표현은 SSM의 해상도에 무관한 모델링을 가능하게 하는 이론적 기반이지만 20, 실제 데이터는 이산적이므로 이산화 과정은 필연적이다. 어떤 이산화 규칙(discretization rule)을 사용하느냐에 따라 연속 시스템을 근사하는 방식이 달라지며, 이는 특정 종류의 시퀀스 패턴을 더 잘 학습하도록 만드는 귀납적 편향으로 작용한다. Mamba와 S4가 서로 다른 이산화 방식을 채택했다는 사실은 이 선택이 아키텍처의 성능에 미치는 영향이 지대함을 시사한다.13

$Δ$를 학습 가능한 파라미터로 만든 것은 이 편향 자체를 데이터로부터 학습하려는 시도로 해석할 수 있다.3

주요 이산화 방법론은 다음과 같다.

- **ZOH (Zero-Order Hold):** Mamba 등 최신 SSM에서 주로 사용되는 방식이다. 이는 시간 간격 $Δ$ 동안 입력 `x(t)`가 `x_k` 값으로 일정하게 유지된다고 가정하고 방정식을 해석적으로 푸는 방법이다. 이 방식은 계단형(step-wise) 신호 변화를 모델링하는 데 효과적이다.14
- **빌리니어 변환 (Bilinear Transform):** S4에서 사용된 방법으로, 적분 연산을 사다리꼴 공식(trapezoid rule)으로 근사하여 변환한다. 이는 ZOH보다 더 부드러운 신호 변화를 가정하는 경향이 있다.11
- **오일러 방법 (Euler Method):** 미분항 $h'(t)$를 $(h_{k} - h_{k-1}) / Δ$로 가장 간단하게 근사하는 방식이다. 구현은 간단하지만, $Δ$가 클 경우 오차가 커질 수 있다.13

딥러닝 SSM에서 $Δ$는 단순한 샘플링 간격 이상의 의미를 가진다. 이는 학습 가능한 파라미터로서, 모델이 현재 입력($x_k$)의 정보와 과거의 누적된 상태($h_{k-1}$) 정보 사이의 가중치를 동적으로 조절하는 역할을 수행한다. $Δ$가 크면 현재 입력에 더 집중하고, $Δ$가 작으면 과거 상태를 더 오래 유지하게 된다.3 이처럼 이산화 과정과 $Δ$ 파라미터는 SSM이 시퀀스 데이터를 처리하는 방식을 근본적으로 결정하는 핵심 요소이다.

## 3.  구조적 상태 공간 모델(S4)로의 발전: 장기 의존성 정복을 향하여

기본적인 SSM은 이론적으로 강력하지만, 딥러닝 모델로 직접 활용하기에는 두 가지 큰 난관이 있었다. 첫째, 랜덤하게 초기화된 상태 행렬 `A`는 RNN과 마찬가지로 기울기 소실/폭주 문제에 취약하여 장기 의존성을 학습하기 어렵다. 둘째, 순환적인 계산 구조는 훈련 시 병렬 처리를 막아 대규모 데이터셋에 적용하기 비효율적이다. 구조적 상태 공간 모델(Structured State Space for Sequence Modeling, S4)은 이 두 문제를 해결하며 SSM을 딥러닝의 주류로 끌어올린 선구적인 아키텍처다.28

### 3.1  장기 의존성 문제와 HiPPO 프레임워크

S4의 이론적 토대는 HiPPO(High-order Polynomial Projection Operator) 프레임워크에서 비롯된다.19 기존 SSM의 장기 기억 실패는 상태 행렬 $A$가 과거 정보를 안정적으로 유지하지 못하기 때문이었다.11 HiPPO는 이 문제를 해결하기 위해 제어 이론과 함수 근사 이론을 결합한 독창적인 해법을 제시했다.

HiPPO의 핵심 아이디어는 잠재 상태 $h(t)$를 과거 입력 시퀀스 전체($x(τ)`for`τ ≤ t$)를 특정 다항식 기저(polynomial basis) 위에서 가장 잘 근사하는 계수들의 집합으로 정의하는 것이다.14 즉, $h(t)$는 과거 역사를 압축한 하나의 '온라인 함수'이며, 이 함수를 이용해 과거 시퀀스를 재구성할 수 있다.30 이 조건을 만족하는 연속 시간 상태 방정식을 유도하면, 상태 행렬 $A$가 매우 특정한 수학적 구조를 가져야 함이 밝혀진다. 예를 들어, 르장드르 다항식(Legendre polynomials)을 기저로 사용하면 `A` 행렬은 특정 형태의 행렬로 고정된다.19

결론적으로, HiPPO는 장기 의존성을 효과적으로 포착할 수 있도록 상태 행렬 $A$에 강력한 사전 지식(prior)을 부여하는 초기화 방법론이다. 무작위로 시작하는 대신, 수학적으로 검증된 구조의 `A` 행렬을 사용함으로써 SSM이 안정적으로 긴 시퀀스를 기억하고 처리할 수 있는 기반을 마련했다.

### 3.2  S4의 이중성: 순환(Recurrence)과 합성곱(Convolution)의 통합

S4의 두 번째 핵심 기여는 SSM이 선형 시불변(Linear Time-Invariant, LTI) 시스템이라는 점에 착안하여, 훈련과 추론에 각각 최적화된 두 가지 동등한 계산 방식을 제시한 것이다.13 이는 이론과 실용의 간극을 메운 중요한 혁신이었다.

- **순환(Recurrent) 모드:** 이산화된 상태 방정식 $h_k = Āh_{k-1} + B̄x_k$를 그대로 계산하는 방식이다. 이는 RNN과 완전히 동일한 구조를 가지며, 이전 상태와 현재 입력만으로 다음 상태를 계산한다.13 이 방식의 가장 큰 장점은 자동 회귀적(autoregressive) 추론 시에 있다. 새로운 토큰을 생성할 때, 고정된 크기의 상태 벡터 

  `h_{k-1}`만 유지하면 되므로 타임스텝당 계산량과 메모리가 $O(1)$로 일정하다. 이는 매우 빠르고 효율적인 추론을 가능하게 한다.13 하지만 훈련 시에는 각 타임스텝이 이전 타임스텝에 의존하므로 병렬화가 불가능하다는 RNN의 고질적인 단점을 그대로 가진다.14

- **합성곱(Convolutional) 모드:** 순환적인 상태 방정식을 재귀적으로 펼쳐보면, $k$번째 출력 $y_k$가 `0`부터 `k`까지의 모든 입력 $x_0,..., x_k$와 특정 가중치의 선형 결합으로 표현됨을 알 수 있다.
  $$
  y_k = \mathbf{C}h_k = \mathbf{C}(\bar{\mathbf{A}}h_{k-1} + \bar{\mathbf{B}}x_k) = \mathbf{C}\bar{\mathbf{A}}(\bar{\mathbf{A}}h_{k-2} + \bar{\mathbf{B}}x_{k-1}) + \mathbf{C}\bar{\mathbf{B}}x_k =...
  $$
  초기 상태 $h_{-1}=0$이라고 가정하면, 이 식은 다음과 같이 정리된다.
  $$
  y_k = \sum_{i=0}^{k} \bar{\mathbf{C}}\bar{\mathbf{A}}^{k-i}\bar{\mathbf{B}}x_i
  $$
  이는 놀랍게도 입력 시퀀스 $x$와 **합성곱 커널(convolutional kernel)** $K̄$의 합성곱(convolution) 연산과 정확히 일치한다.13
  $$
  y = x * \bar{K} \quad \text{where} \quad \bar{K} = (\bar{\mathbf{C}}\bar{\mathbf{B}}, \bar{\mathbf{C}}\bar{\mathbf{A}}\bar{\mathbf{B}}, \bar{\mathbf{C}}\bar{\mathbf{A}}^2\bar{\mathbf{B}},...)
  $$
  합성곱 연산의 가장 큰 장점은 고속 푸리에 변환(Fast Fourier Transform, FFT)을 통해 매우 효율적으로 병렬 계산이 가능하다는 점이다. FFT를 이용하면 길이 $L$의 시퀀스에 대한 합성곱을 $O(L log L)$ 시간 복잡도로 계산할 수 있다.11

S4는 이 두 모드의 장점을 극대화하는 전략을 채택했다. 즉, **대규모 병렬 처리가 필수적인 훈련 시에는 합성곱 모드**를 사용하고, **순차적 생성이 필요한 추론 시에는 순환 모드**를 사용하여, 훈련 속도와 추론 효율성이라는 두 마리 토끼를 모두 잡았다.13

### 3.3  구조화된 행렬과 계산 효율성

S4의 마지막 퍼즐 조각은 '구조적(Structured)'이라는 이름에 담겨 있다. HiPPO 이론으로부터 유도된 상태 행렬 $A$는 무작위 행렬이 아니라, 대각 행렬에 저계급(low-rank) 행렬이 더해진 형태(Diagonal Plus Low-Rank, DPLR)와 같은 특정 수학적 구조를 가진다.27

S4는 이 구조를 적극적으로 활용하여 합성곱 커널 $K̄$를 계산하는 과정을 획기적으로 가속화했다. 일반적인 방식으로 길이 $L$의 커널을 계산하려면 $O(N^2 L)$의 복잡도가 필요하지만, $A$ 행렬의 특별한 구조를 이용하면 이 과정을 $O((N+L) log(N+L))$로 단축할 수 있다.29 이 '구조화된' 행렬 덕분에 S4는 이론적 강점을 유지하면서도 실제 대규모 데이터에 적용 가능한 계산 효율성을 확보할 수 있었다.24

### 3.4  S4의 한계: LTI 시스템의 본질적 제약

S4는 장기 의존성, 훈련 효율성, 추론 효율성 문제를 동시에 해결하며 SSM의 가능성을 증명했지만, 근본적인 한계를 가지고 있었다. 바로 S4가 시스템 행렬($A, B, C, Δ$)이 시간에 따라 변하지 않는 **선형 시불변(Linear Time-Invariant, LTI)** 시스템이라는 가정에 기반한다는 점이다.14

이는 모델의 동역학, 즉 정보를 처리하고 압축하는 방식이 입력 데이터의 내용과 무관하게 항상 고정되어 있음을 의미한다. 예를 들어, "the"와 같은 불용어(stopword)와 "dragon"과 같은 핵심 단어에 동일한 합성곱 필터를 적용하는 것과 같다.32 이러한 LTI 특성은 오디오나 시계열과 같은 연속적인 신호에서는 효과적일 수 있지만, 문맥에 따라 단어의 중요도가 급격히 변하는 텍스트와 같은 이산적 데이터(discrete data)를 모델링하는 데는 본질적인 한계를 드러냈다.14 S4의 상태 벡터 

$h$는 과거 정보의 '압축된' 표현이지만 30, 이 압축 방식은 수동적이고 고정적이어서 '무엇을' 압축할지 선택할 수 없었다. 이 '선택성(selectivity)'의 부재가 바로 Mamba가 해결하고자 한 핵심 문제였다.

## 4.  맘바(Mamba): 선택적 상태 공간 모델의 등장

Mamba는 S4가 남긴 '선택성'의 과제를 정면으로 돌파하며 등장했다. S4의 LTI 제약, 즉 모델의 동역학이 입력 내용과 무관하다는 한계를 극복하기 위해 Mamba는 두 가지 대담하고 혁신적인 아이디어를 제시했다. 첫째는 모델 파라미터를 입력에 따라 동적으로 변화시키는 '선택적 메커니즘'이고, 둘째는 그로 인해 포기해야 했던 병렬 훈련의 이점을 되찾기 위한 '하드웨어 인식 알고리즘'이다.

### 4.1  핵심 혁신 1: 선택적 메커니즘(Selective Mechanism) - LTI의 족쇄를 풀다

Mamba의 가장 핵심적인 혁신은 SSM을 LTI 시스템의 제약에서 해방시킨 것이다. 이를 위해 Mamba는 기존에 상수로 고정되어 있던 SSM의 파라미터 $Δ$ (시간 간격), $B$ (입력 행렬), $C$ (출력 행렬)를 현재 입력 $x_t$에 따라 동적으로 변하는 함수로 재정의했다.3

수학적으로 이는 각 타임스텝 $t$의 입력 $x_t$를 간단한 선형 투영(linear projection)을 통해 해당 타임스텝에서만 사용될 고유한 파라미터 $Δ_t$, $B_t$, $C_t$를 생성하는 방식으로 구현된다.23 결과적으로, 연속 시간 파라미터가 시간에 따라 변하게 되므로, 이를 이산화하여 얻은 이산 시간 행렬 $Ā$와 $B̄$ 역시 시변(time-varying) 특성을 갖는 $Ā_t$와 $B̄_t$가 된다. 이로써 상태 업데이트 방정식은 다음과 같이 변한다.
$$
h_t = \bar{\mathbf{A}}_t h_{t-1} + \bar{\mathbf{B}}_t x_t
$$
이러한 **선택적 메커니즘**은 모델에 다음과 같은 직관적인 능력을 부여한다.

- **정보 필터링 (Information Filtering):** 입력 토큰의 중요도에 따라 $B_t$의 값을 조절하여 상태 $h_t$에 반영할 정보의 양을 결정한다. 이는 RNN의 입력 게이트(input gate)와 유사한 역할을 한다. 예를 들어, 문장에서 중요한 키워드가 나타나면 $B_t$의 norm을 키워 해당 정보를 상태에 강하게 각인시키고, 불필요한 불용어가 나타나면 $B_t$의 norm을 줄여 무시할 수 있다.7
- **메모리 리셋 (State Resetting):** 문장의 끝이나 주제가 바뀌는 지점과 같이 과거의 맥락이 더 이상 유효하지 않을 때, 모델은 $A_t$를 조절하여 이전 상태 $h_{t-1}$의 영향을 거의 0으로 만들어 상태를 '리셋'할 수 있다. 이는 독립적인 정보 덩어리들을 처리할 때 매우 유용하다.3
- **동적 시간 스케일링 (Dynamic Time Scaling):** $Δ_t$를 조절하여 모델의 초점을 동적으로 변경한다. $Δ_t$가 크면 시스템은 현재 입력 $x_t$에 더 집중하게 되고, $Δ_t$가 작으면 과거의 맥락을 담고 있는 $h_{t-1}$을 더 오래 유지하게 된다. 이를 통해 모델은 시퀀스의 각 부분에 맞는 적절한 시간 해상도를 선택할 수 있다.3

결론적으로, 선택적 메커니즘은 SSM을 수동적인 '시퀀스 압축기'에서 능동적인 '정보 처리기'로 진화시켰다. S4가 과거 정보를 효율적으로 '압축'하는 방법을 제공했다면, Mamba는 이 압축 과정에 '지능'을 부여하여 모델 스스로가 무엇을 압축하고 무엇을 버릴지 '결정'할 수 있게 만든 것이다.7 이는 Transformer의 어텐션이 쿼리-키 유사도를 통해 정보의 가중치를 동적으로 결정하는 것과 기능적으로 유사한 역할을 수행하며, RNN의 효율적인 순환 구조와 Transformer의 동적인 내용 기반 추론 능력을 결합하려는 시도의 정수라 할 수 있다.

### 4.2  선택성의 대가: 전역 합성곱의 포기

선택성을 얻는 데에는 큰 대가가 따랐다. 모델 파라미터가 입력에 따라 매 타임스텝마다 변하게 되면서, 시스템은 더 이상 LTI가 아닌 **시변(Time-Varying)** 시스템이 되었다.27

시변 시스템에서는 각 타임스텝의 동역학이 다르기 때문에, S4의 가장 큰 장점이었던 고정된 단일 합성곱 커널 $K̄$로 시스템 전체를 표현하는 것이 원리적으로 불가능해진다.9 즉, Mamba는 선택성을 얻기 위해 S4의 핵심 무기였던 **FFT를 통한 빠른 병렬 훈련(합성곱 모드)을 포기**해야만 했다. 이는 Mamba가 극복해야 할 새로운 기술적 도전 과제를 낳았다.

### 4.3  핵심 혁신 2: 하드웨어 인식 알고리즘 - 순환 계산의 재발견

합성곱을 포기하고 순환 모드만 사용한다면, 훈련 속도가 다시 RNN처럼 느려져 대규모 모델 학습이 불가능해진다. Mamba는 이 딜레마를 해결하기 위해 문제 해결의 패러다임을 전환했다. 기존 알고리즘에 의존하는 대신, 계산의 근본 원리와 GPU 하드웨어의 동작 방식을 연결하여 순환 계산 자체를 병렬화하는 새로운 방법을 고안했다.3 Mamba의 이러한 접근은 모델 아키텍처와 시스템 최적화의 '공동 설계(co-design)'가 얼마나 강력할 수 있는지를 보여주는 대표적인 사례다.

- **병렬 스캔 (Parallel Scan):** Mamba의 순환적인 상태 업데이트 $h_t = Ā_t h_{t-1} + B̄_t x_t$는 겉보기에는 순차적이지만, 수학적으로 재구성하면 **연관 법칙(associative property)**, 즉 $(a • b) • c = a • (b • c)$를 만족하는 이항 연산(binary associative operator) $•$으로 표현될 수 있다. 이러한 연산은 '병렬 스캔(parallel scan)' 또는 '병렬 접두사 합(parallel prefix sum)' 알고리즘을 통해 효율적으로 병렬 계산이 가능하다. 대표적인 알고리즘인 Blelloch 스캔은 `L` 길이의 시퀀스에 대한 모든 중간 상태 $h_1,..., h_L$을 $O(L)$이 아닌 $O(L log L)$의 연산량과 $O(log L)$ 깊이의 병렬 단계로 계산할 수 있게 해준다.21
- **커널 퓨전 (Kernel Fusion)과 SRAM 최적화:** Mamba는 GPU의 메모리 계층 구조(느린 HBM, 매우 빠른 SRAM)를 적극적으로 활용한다. 파라미터 로딩, $Δ, B, C$의 동적 생성, 이산화, 그리고 병렬 스캔의 순환 계산까지, 여러 단계의 연산을 하나의 거대한 GPU 커널로 융합(fusion)한다. 이 '커널 퓨전'을 통해 GPU 코어와 HBM(High Bandwidth Memory) 간의 데이터 전송 횟수를 최소화한다. 특히, 계산 과정에서 발생하는 거대한 중간 상태 값들을 HBM에 기록하지 않고, 훨씬 빠른 SRAM 내에서만 처리하고 최종 결과만 HBM에 쓰는 방식을 사용한다. 이는 메모리 병목 현상을 극적으로 줄여주며, FlashAttention에서 사용된 핵심 아이디어와 동일한 원리다.15

이 두 가지 기법이 결합된 하드웨어 인식 알고리즘 덕분에, Mamba는 이론적으로 순차적인 순환 계산을 수행함에도 불구하고 실제 하드웨어에서는 Transformer에 필적하는 빠른 훈련 속도를 달성할 수 있었다.9

### 4.4  Mamba 블록 아키텍처

Mamba는 이러한 선택적 SSM을 핵심 구성 요소로 하는 통합적인 'Mamba 블록'을 기본 단위로 사용한다.15 이 블록은 Transformer의 어텐션 블록과 MLP 블록을 하나로 합친 것과 같은 역할을 수행하도록 설계되었다.

Mamba 블록의 구조는 다음과 같다 9:

1. **입력 분기:** 입력 $x$는 선형 투영을 거쳐 두 개의 경로, 즉 메인 경로($x_proj$)와 게이팅 경로($z_proj$)로 나뉜다.
2. **지역 정보 처리:** 메인 경로의 $x_proj$는 선택적 SSM에 들어가기 전에 먼저 1D 컨볼루션 레이어를 통과한다. 이 컨볼루션은 인접한 토큰 간의 관계와 같은 지역적(local) 정보를 포착하는 역할을 한다. 이를 통해 후속하는 SSM이 전역적(global)인 장기 의존성 모델링에 더 집중할 수 있도록 부담을 덜어준다.
3. **비선형 활성화:** 컨볼루션을 거친 결과와 게이팅 경로의 $z_proj$는 각각 SiLU(Sigmoid Linear Unit)와 같은 비선형 활성화 함수를 통과한다.
4. **선택적 SSM:** 활성화된 메인 경로의 입력이 선택적 SSM으로 들어가, 시변 파라미터 $Δ_t, B_t, C_t$를 생성하고 상태 업데이트를 거쳐 출력 $y$를 계산한다.
5. **게이팅:** SSM의 출력 $y$는 게이팅 경로의 활성화된 출력과 원소별 곱셈(element-wise multiplication)을 수행한다. 이 게이팅 메커니즘은 SSM이 계산한 정보 중 어떤 것을 다음 레이어로 전달할지를 조절하는 역할을 한다.
6. **최종 출력:** 게이팅을 거친 결과가 Mamba 블록의 최종 출력이 된다.

이러한 통합적인 블록 구조는 아키텍처를 단순화하면서도 지역적 정보 처리와 전역적, 내용 기반의 동적 정보 처리를 효과적으로 결합한다.

## 5.  아키텍처 심층 비교: Mamba, RNN, 그리고 Transformer

Mamba의 등장은 시퀀스 모델링 아키텍처의 지형을 근본적으로 바꾸어 놓았다. Mamba의 진정한 가치를 이해하기 위해서는 기존의 지배적인 패러다임이었던 RNN 및 Transformer와 다각적인 관점에서 심층적으로 비교 분석할 필요가 있다. 이들의 차이는 단순히 성능 수치를 넘어, 정보를 처리하는 근본적인 철학의 차이에서 비롯된다.

### 5.1  계산 및 메모리 복잡도: 효율성의 대결

계산 및 메모리 효율성은 대규모 모델을 훈련하고 배포하는 데 있어 가장 현실적인 제약 조건이다. 이 관점에서 세 아키텍처는 뚜렷한 차이를 보인다.

- **훈련(Training) 복잡도:**
  - **Transformer:** 셀프 어텐션 메커니즘은 시퀀스 내 모든 토큰 쌍 간의 상호작용을 계산해야 하므로, 시퀀스 길이 $L$에 대해 $O(L^2)$의 계산 복잡도를 가진다. FlashAttention과 같은 최적화 기법이 실제 연산 속도를 개선하지만, 근본적인 이차 복잡도 자체를 바꾸지는 못한다.2
  - **RNN:** 이론적인 계산 복잡도는 $O(L)$이지만, 각 타임스텝의 계산이 이전 타임스텝에 의존하는 순차적 특성 때문에 GPU를 이용한 병렬화가 불가능하다. 이로 인해 실제 훈련 시간은 매우 길다.1
  - **Mamba:** 하드웨어 인식 병렬 스캔 알고리즘 덕분에 순환 구조임에도 불구하고 효과적인 병렬 훈련이 가능하다. 따라서 시퀀스 길이 $L$에 대해 선형적인 $O(L)$의 계산 복잡도를 달성한다.9
- **추론(Inference) 복잡도:**
  - **Transformer:** 다음 토큰을 생성하기 위해 이전의 모든 토큰에 대한 Key와 Value 값을 저장하는 **KV 캐시**가 필요하다. 이 캐시의 크기는 시퀀스 길이 $L$에 비례하여 $O(L)$로 선형적으로 증가하며, 새로운 토큰을 생성할 때마다 이 전체 캐시에 대해 어텐션을 계산해야 하므로 타임스텝당 $O(L)$의 계산 시간이 소요된다.35
  - **RNN & Mamba:** 두 아키텍처 모두 이전 타임스텝의 정보를 고정된 크기의 상태 벡터 $h_{t-1}$에 요약한다. 따라서 다음 토큰을 생성하는 데 필요한 계산량과 메모리가 시퀀스 길이에 무관하게 $O(1)$로 일정하다. Mamba는 KV 캐시가 필요 없으므로 긴 시퀀스에 대한 자동 회귀적 추론에서 Transformer에 비해 압도적인 속도와 메모리 효율성을 보인다.13

이러한 차이점을 아래 표로 요약할 수 있다.

| 특징                     | RNN (LSTM/GRU)           | Transformer                    | Mamba                    |
| ------------------------ | ------------------------ | ------------------------------ | ------------------------ |
| **핵심 메커니즘**        | 순환 및 게이팅           | 셀프 어텐션                    | 선택적 상태 공간         |
| **훈련 복잡도**          | $O(L)$ (병렬화 불가)     | $O(L^2)$                       | $O(L)$ (병렬 스캔)       |
| **추론 복잡도 (토큰당)** | $O(1)$                   | $O(L)$                         | $O(1)$                   |
| **추론 메모리**          | $O(1)$                   | $O(L)$ (KV 캐시)               | $O(1)$                   |
| **병렬화**               | 훈련 시 불가             | 훈련 시 가능                   | 훈련 시 가능             |
| **장기 의존성**          | 약함 (기울기 소실)       | 강함 (비압축)                  | 강함 (선택적 압축)       |
| **주요 강점**            | 가벼운 추론              | 문맥 내 학습, 정확한 정보 검색 | 초장문 처리, 추론 효율성 |
| **주요 약점**            | 훈련 비효율성, 정보 소실 | 계산/메모리 비효율성           | 정보 복사/검색 능력 제한 |

### 5.2  장기 의존성 처리 방식: 압축 vs. 비압축

세 아키텍처의 효율성 차이는 과거 정보를 처리하는 근본적인 철학의 차이에서 기인한다. 이는 '상태(Stateful)' 기반 모델링과 '무상태(Stateless)' 기반 모델링의 대립으로 볼 수 있다.

- **Transformer (비압축, 무상태):** Transformer는 근본적으로 '무상태' 모델이다. 각 토큰의 표현을 계산할 때마다 KV 캐시를 통해 과거의 모든 정보를 원본 그대로, 즉 **비압축(uncompressed)** 상태로 참조한다.7 '상태'라는 개념이 없고, 과거는 그저 참조할 수 있는 방대한 '기록'일 뿐이다.40 이 방식은 정보 손실이 없다는 막대한 장점을 가지지만, 모든 기록을 매번 참조해야 하므로 비효율적이다.
- **Mamba (선택적 압축, 상태 기반):** Mamba는 RNN의 계보를 잇는 '상태' 기반 모델이다. 과거의 모든 정보는 현재의 고정된 크기 상태 벡터 $h$로 요약되어야 한다. 이는 본질적으로 정보의 **압축(compression)**을 수반한다.39 Mamba의 혁신은 이 압축 과정을 '선택적'으로 만든 것이다. 선택적 메커니즘을 통해 모델은 중요한 정보는 상태에 보존하고 불필요한 정보는 버리도록 학습된다.41 이는 효율적이지만, 압축 과정에서 정보 손실의 위험을 내포한다.
- **RNN (무차별적 압축, 상태 기반):** RNN 역시 상태 기반 모델이지만, 과거 정보를 상태 벡터 $h$에 **무차별적으로 압축**하려 시도한다. 하지만 기울기 소실 문제로 인해 오래된 정보는 대부분 소실되어 효과적인 압축에 실패한다.1

'Mamba vs. Transformer'의 논쟁은 단순히 어떤 아키텍처가 더 우수한가의 문제가 아니라, 주어진 작업에 '상태 기반의 압축적 추론'이 적합한지, '무상태 기반의 전체 참조'가 필요한지에 대한 근본적인 질문으로 귀결된다.

### 5.3  본질적 강점과 약점: 언제 Mamba를, 언제 Transformer를 사용해야 하는가?

이러한 철학의 차이는 각 아키텍처의 실제 성능 특성으로 이어진다.

- **Transformer의 강점:**

  - **정확한 정보 검색 및 복사(Copying):** 비압축 컨텍스트 덕분에, "Phonebook" 벤치마크처럼 긴 문서 내에서 특정 이름에 해당하는 전화번호를 정확히 찾아내는 작업에 매우 강력하다. Mamba는 정보를 압축된 상태로 저장하기 때문에 이러한 종류의 verbatim recall 작업에 약점을 보인다.6
  - **문맥 내 학습(In-context Learning):** 프롬프트에 몇 가지 예시(few-shot)를 제공하고 새로운 문제를 풀게 하는 능력에서 Transformer는 Mamba보다 우월한 성능을 보인다. 이는 모든 토큰 간의 직접적인 상호작용을 모델링하는 어텐션 메커니즘이 복잡한 패턴을 즉석에서 학습하는 데 더 유리하기 때문이다.38

- **Mamba의 강점:**

  - **초장문 시퀀스 처리:** 선형적 확장성은 Mamba의 가장 독보적인 장점이다. Transformer의 $O(L^2)$ 복잡도가 단순히 '느리다'는 의미를 넘어 학습 가능한 시퀀스 길이에 물리적인 한계를 부여하는 반면 42, Mamba의 

    $O(L)$ 복잡도는 이러한 장벽을 허물어 유전체학, 고해상도 오디오, 장편 소설 분석 등 이전에는 다룰 수 없었던 새로운 종류의 초장문 데이터에 딥러닝을 적용할 수 있는 길을 열었다.43 이는 단순한 성능 개선이 아니라, 딥러닝의 응용 범위를 새로운 영역으로 확장하는 '가능성의 확장'으로 해석해야 한다.

  - **추론 효율성:** KV 캐시가 없어 추론 시 처리량(throughput)이 훨씬 높고 메모리 사용량이 일정하다. 이는 실시간 챗봇, 스트리밍 데이터 분석, 혹은 모바일이나 엣지 디바이스와 같이 리소스가 제한된 환경에 모델을 배포할 때 결정적인 이점이 된다.9

- **하이브리드 모델의 부상:** 순수 Mamba와 순수 Transformer는 각각 명확한 장단점을 가진다. 이 때문에 두 아키텍처의 장점을 결합한 **하이브리드(Hybrid) 모델**이 유력한 대안으로 떠오르고 있다. 예를 들어, 대부분의 레이어는 Mamba 블록으로 구성하여 효율성을 확보하고, 소수의 어텐션 블록을 중간에 삽입하여 Mamba의 약점인 정보 검색 및 문맥 내 학습 능력을 보강하는 식이다. 실제로 Mamba-2-Hybrid와 같은 모델은 순수 Mamba와 순수 Transformer의 성능을 모두 뛰어넘는 결과를 보여주며, 두 패러다임의 시너지 효과를 입증했다.38

## 6.  성능 벤치마크 및 응용 분야 분석

Mamba의 이론적 우수성은 다양한 실제 데이터와 벤치마크를 통해 검증되었다. 특히 언어 모델링, 초장문 시퀀스 처리, 그리고 새로운 응용 분야에서 Mamba는 기존 아키텍처와 비교하여 인상적인 성능을 보여주었다.

### 6.1  언어 모델링: Transformer의 아성에 도전하다

언어 모델링은 Transformer가 가장 큰 성공을 거둔 분야이며, Mamba가 진정한 대안이 될 수 있는지를 가늠하는 핵심적인 시험대이다.

- **사전 훈련 성능 (The Pile):** 대규모 텍스트 데이터셋인 The Pile에서의 사전 훈련 성능(Perplexity)을 비교했을 때, Mamba는 동일한 파라미터 수의 강력한 Transformer 모델(Transformer++)을 능가하고, 심지어 두 배 크기의 Transformer 모델과 동등한 성능을 달성했다.9 이는 Mamba가 파라미터 효율성 면에서 Transformer보다 우수할 수 있음을 시사한다.
- **다운스트림 제로샷 평가:** 사전 훈련된 모델을 다양한 다운스트림 작업에 제로샷(zero-shot)으로 평가했을 때, Mamba는 상식 추론, 독해 등 일반적인 언어 이해 과제에서 강력한 성능을 보였다. 예를 들어, LAMBADA 벤치마크에서 Mamba는 동일 크기의 다른 오픈소스 모델들을 꾸준히 능가했다.49 그러나 MMLU 벤치마크의 5-shot 평가와 같이 복잡한 지시를 따르고 여러 선택지 중에서 답을 골라야 하는 특정 형식의 문맥 내 학습 과제에서는 Transformer에 비해 취약점을 드러냈다.38 이는 Mamba의 압축된 상태 표현이 Transformer의 비압축 KV 캐시만큼 정교한 정보 검색을 지원하지 못하기 때문으로 분석된다.
- **하이브리드 모델의 우수성:** 이러한 약점을 보완하기 위해 제안된 Mamba-2-Hybrid 모델은 순수 Mamba와 순수 Transformer의 성능을 모두 뛰어넘는 결과를 보였다. 80억 파라미터 규모의 비교 연구에서 Mamba-2-Hybrid는 12개의 표준 벤치마크 모두에서 Transformer를 능가했으며, 특히 MMLU 5-shot 정확도 격차를 크게 줄였다.38 이는 Mamba의 효율성과 Transformer의 표현력을 결합하는 것이 현재로서는 가장 유망한 방향임을 시사한다.

### 6.2  초장문 시퀀스: 유전체학(Genomics) 및 오디오 처리

Mamba의 선형 시간 복잡도는 유전체학이나 오디오와 같이 시퀀스 길이가 수십만에서 수백만에 이르는 도메인에서 진가를 발휘한다.

- **유전체학 (Human Genome):** 인간 유전체(HG38) 데이터를 이용한 언어 모델링 작업에서 Mamba는 기존의 최첨단 유전체학 모델인 HyenaDNA와 Transformer++보다 훨씬 뛰어난 확장성(scaling)을 보여주었다.9 Mamba는 약 3~4배 적은 파라미터로 이들과 동등한 성능을 달성했으며, 시퀀스 길이를 백만까지 늘려도 성능이 지속적으로 향상되는 반면, HyenaDNA는 오히려 성능이 저하되는 모습을 보였다.9 아래 표는 유인원 DNA 분류 작업에서 시퀀스 길이가 길어질수록 Mamba의 성능이 HyenaDNA를 압도함을 보여준다.

| 모델          | 파라미터 | 시퀀스 길이 2^10 정확도 (%) | 시퀀스 길이 2^16 정확도 (%) | 시퀀스 길이 2^20 정확도 (%) |      |
| ------------- | -------- | --------------------------- | --------------------------- | --------------------------- | ---- |
| HyenaDNA 1.4M | 1.4M     | 28.04                       | 42.22                       | 54.87                       |      |
| Mamba 1.4M    | 1.4M     | 31.47                       | 40.72                       | 71.67                       |      |
| Mamba 7M      | 7M       | 30.00                       | 43.73                       | **81.31**                   |      |

표 1: 유인원 DNA 분류 정확도(%) 비교. Mamba는 초장문 시퀀스에서 월등한 성능을 보인다.9

- **오디오 처리 (Speech Generation):** 1초 길이의 숫자 음성을 생성하는 SC09 벤치마크에서 Mamba는 이전 SOTA 모델인 SaShiMi(S4 기반)나 WaveNet, 심지어 GAN 및 확산(Diffusion) 기반 모델들보다 훨씬 적은 파라미터로 더 높은 충실도(FID, IS 등 지표)의 오디오를 생성했다.9 이는 Mamba가 연속적인 신호 데이터를 모델링하는 데 매우 효과적임을 증명한다.

| 모델               | 파라미터  | NLL ↓     | FID ↓    | IS ↑     | AM ↓     |      |
| ------------------ | --------- | --------- | -------- | -------- | -------- | ---- |
| SaShiMi            | 5.8M      | 1.873     | 1.99     | 5.13     | 0.74     |      |
| DiffWave + SaShiMi | 23.0M     | -         | 1.42     | 5.94     | 0.59     |      |
| **Mamba**          | **6.1M**  | **1.852** | **0.94** | **6.26** | **0.52** |      |
| **Mamba**          | **24.3M** | **1.860** | **0.67** | **7.33** | **0.36** |      |

표 2: SC09 음성 생성 벤치마크 성능 비교. Mamba는 적은 파라미터로 SOTA를 달성했다.9

이러한 결과는 Mamba의 성능이 단순히 특정 벤치마크에 국한된 것이 아니라, 모델의 아키텍처적 특성이 특정 데이터 양식의 요구사항과 잘 부합하기 때문임을 보여준다. Mamba는 효율적인 '요약과 압축'에, Transformer는 정확한 '기억과 참조'에 강점을 보인다. 이는 단일 아키텍처가 모든 문제를 지배하는 시대가 끝나고, 과제의 특성에 따라 최적의 아키텍처를 선택하거나 조합하는 시대로 나아가고 있음을 암시한다.

### 6.3  새로운 지평: 컴퓨터 비전 및 시계열 분석

Mamba의 성공은 언어, 오디오, 유전체학을 넘어 다른 시퀀스 데이터 분야로 빠르게 확산되고 있다.

- **컴퓨터 비전 (Computer Vision):** 이미지는 전통적으로 2D 그리드 데이터로 취급되지만, 패치(patch) 단위로 나누어 1D 시퀀스로 변환하면 시퀀스 모델을 적용할 수 있다. Vision Mamba(Vim)와 같은 모델들은 이미지를 1D 시퀀스로 만들고, 양방향(bidirectional) Mamba 블록을 적용하여 이미지의 전역적인 문맥을 포착한다.50 이들은 이미지 분류(ImageNet), 객체 탐지(COCO), 시맨틱 분할(ADE20K)과 같은 표준 비전 벤치마크에서 Transformer 기반 모델(ViT)과 경쟁력 있는 성능을 보이면서도, 특히 고해상도 이미지 처리에서 계산 효율성의 이점을 제공한다.51
- **시계열 분석 (Time Series Analysis):** 시계열 예측(forecasting)은 장기적인 추세와 계절성, 단기적인 변동을 모두 포착해야 하는 어려운 과제다. Mamba의 장기 의존성 모델링 능력과 효율성은 이 분야에 매우 적합하다. TSMamba와 같은 모델들은 Mamba 아키텍처를 시계열 예측에 적용하여, Transformer 기반 모델의 막대한 계산 부담 없이 복잡한 시계열 패턴을 효과적으로 학습할 수 있음을 보여주었다.53

### 6.4  프로덕션 환경에서의 실용적 이점

Mamba의 진정한 강점은 학술적 벤치마크를 넘어 실제 프로덕션 환경에서 나타나는 실용적 이점에 있다.

- **추론 처리량 (Throughput):** Mamba는 동일한 하드웨어에서 동일 크기의 Transformer보다 5배 이상 높은 추론 처리량을 보인다.8 이는 KV 캐시를 계산하고 저장할 필요가 없기 때문으로, 실시간 서비스에서 더 많은 사용자 요청을 동시에 처리할 수 있음을 의미한다.
- **메모리 효율성:** 긴 컨텍스트를 처리할 때 Transformer는 KV 캐시로 인해 메모리 사용량이 선형적으로 증가하지만, Mamba는 일정한 메모리만 사용한다. 이는 메모리가 제한된 엣지 디바이스에 모델을 배포하거나, 매우 긴 문서에 대한 요약 또는 질의응답을 수행할 때 결정적인 장점이 된다.39
- **총 소유 비용 (TCO, Total Cost of Ownership):** 빠른 추론 속도와 낮은 메모리 요구량은 결국 더 적은 수의 GPU로 동일한 서비스를 운영할 수 있게 한다. 이는 하드웨어 구매 비용과 전기 요금을 포함한 총 소유 비용을 크게 절감하는 효과로 이어진다.56

다만, Mamba의 성능이 고도로 최적화된 CUDA 커널에 크게 의존한다는 점은 현재의 성숙도 측면에서 고려할 점이다. 일부 연구에서는 특정 작업(예: 문서 랭킹)에서의 훈련 처리량이 FlashAttention과 같이 고도로 최적화된 Transformer 구현에 비해 낮다는 결과도 보고되었다.35 이는 Transformer 생태계가 수년간의 엔지니어링 노력을 통해 성숙해 있는 반면, Mamba는 아직 초기 단계임을 보여준다. 앞으로 커뮤니티의 엔지니어링 노력이 더해지면 현재의 몇몇 약점들은 상당 부분 개선될 가능성이 높다.

## 7.  결론: 현황과 미래 전망

Mamba 아키텍처의 등장은 시퀀스 모델링 분야에 중요한 전환점을 제시했다. 이는 단순히 Transformer의 대안을 넘어, 모델 설계의 근본적인 철학과 접근 방식에 대한 새로운 관점을 열었다. Mamba의 핵심 기여와 현재의 한계, 그리고 미래 연구 방향을 종합하며 차세대 파운데이션 모델의 미래를 전망한다.

### 7.1. Mamba의 핵심 기여 요약

Mamba의 핵심 기여는 시퀀스 모델링 분야의 오랜 난제였던 '효율성'과 '표현력' 사이의 상충 관계를 혁신적으로 개선한 데 있다.

첫째, **선택적 메커니즘**을 통해 SSM의 표현력을 극대화했다. 입력에 따라 동적으로 변하는 파라미터를 도입함으로써, 모델은 고정된 필터로 정보를 일괄 처리하는 대신, 내용에 기반하여 중요한 정보를 선별하고 불필요한 정보를 망각하는 능동적인 정보 처리 능력을 갖추게 되었다. 이는 S4의 LTI 한계를 극복하고, 특히 텍스트와 같은 이산적 데이터에서 Transformer급 성능을 달성하는 결정적인 열쇠가 되었다.

둘째, **하드웨어 인식 알고리즘**을 통해 계산 효율성을 확보했다. 선택성을 위해 포기해야 했던 합성곱 기반의 병렬 훈련을 병렬 스캔과 커널 퓨전이라는 새로운 방식으로 대체함으로써, 순환 구조임에도 불구하고 선형 시간 복잡도($O(L)$)로 빠른 훈련을 가능하게 했다.

이 두 가지 혁신의 결합을 통해 Mamba는 이전에는 불가능했던 백만 단위 길이의 초장문 시퀀스 모델링 시대를 열었으며, 추론 시에는 $O(1)$의 계산 및 메모리 복잡도로 압도적인 효율성을 제공한다. 이는 Mamba의 성공이 단순히 새로운 수학적 모델을 제안하는 데 그치지 않고, 이론(제어 이론), 모델 아키텍처(선택적 SSM), 알고리즘(병렬 스캔), 하드웨어 최적화(커널 퓨전)라는 다층적인 스택 전반에 걸친 '풀스택(full-stack)' 혁신의 결과물임을 보여준다.

### 7.1  현재의 한계와 미래 연구 방향

Mamba는 엄청난 잠재력을 보여주었지만, 여전히 해결해야 할 과제와 연구 방향이 남아있다.

- **아키텍처적 한계와 하이브리드 모델:** 순수 Mamba는 정확한 정보 복사나 복잡한 문맥 내 학습 능력에서 Transformer에 비해 약점을 보인다. 이를 극복하기 위해 Mamba의 효율성과 Transformer의 표현력을 결합한 **하이브리드 아키텍처** 연구가 활발히 진행되고 있다.38 Mamba 블록과 어텐션 블록을 어떻게 최적으로 조합할 것인지, 두 메커니즘 간의 상호작용을 어떻게 극대화할 것인지가 중요한 연구 주제가 될 것이다.
- **시스템적 과제와 일반화:** 현재 Mamba의 고성능은 고도로 최적화된 CUDA 커널에 크게 의존한다.57 이는 구현의 복잡성을 높이고, NVIDIA GPU가 아닌 다른 하드웨어(TPU, NPU 등)로의 이식성을 저해하는 요인이 될 수 있다. 향후에는 더 일반적이면서도 효율적인 Mamba 구현 방법에 대한 연구가 필요하다.
- **이론적 기반의 심화:** Mamba의 선택적 메커니즘이 왜, 그리고 어떻게 효과적으로 동작하는지에 대한 더 깊은 이론적 분석이 요구된다. 동적 시스템 이론, 정보 이론, 혹은 신경과학적 관점에서 Mamba의 작동 원리를 규명하는 연구는 Mamba를 더욱 발전시키고 새로운 아키텍처를 설계하는 데 중요한 통찰을 제공할 것이다.

### 7.2  Transformer를 넘어: 차세대 파운데이션 모델의 미래

Mamba의 등장은 '효율성'을 다시 AI 연구의 중심으로 가져왔다. Transformer의 등장 이후 '더 크게(Bigger is better)'라는 스케일링 법칙이 연구를 주도해왔다면, Mamba는 '얼마나 효율적으로 잘하는가' 역시 중요한 척도임을 상기시켰다. 이러한 흐름은 AI의 지속 가능성(sustainability)과 민주화(democratization)에 긍정적인 영향을 미칠 것이다. 더 적은 자원으로도 강력한 모델을 훈련하고 배포할 수 있게 되면, 더 많은 연구 그룹과 기업이 AI 기술의 혜택을 누릴 수 있게 된다.

미래의 파운데이션 모델은 단일 아키텍처가 모든 것을 지배하는 형태가 아닐 가능성이 높다. Mamba가 Transformer를 완전히 대체하기보다는, 두 아키텍처의 장점을 결합한 **하이브리드 모델**이 차세대 파운데이션 모델의 주류가 될 것으로 전망된다.46 더 나아가, 미래의 모델은 고정된 구조가 아니라, 주어진 데이터와 과제의 특성에 따라 어텐션, SSM, 컨볼루션 등 다양한 연산 모듈을 동적으로 조합하고 라우팅하는 '메타-아키텍처(meta-architecture)' 혹은 '전문가 혼합(Mixture-of-Experts)' 형태로 발전할 수 있다.

결론적으로, Mamba는 시퀀스 모델링의 역사에서 중요한 이정표를 세웠다. 이는 단순히 또 하나의 새로운 모델이 아니라, 아키텍처 설계에 대한 새로운 철학과 방법론을 제시했다는 점에서 더 큰 의미를 가진다. Mamba가 촉발한 새로운 경쟁과 탐구는 앞으로 더욱 다양하고 혁신적인 아키텍처의 등장을 이끌 것이며, 인공지능 기술의 지평을 더욱 넓혀나갈 것이다.

#### **참고 자료**

1. Shallow Dive into RNN, Transformers, SSM, S4, and Mamba | by Minhajul Hoque - Medium, 8월 16, 2025에 액세스, https://medium.com/@minh.hoque/shallow-dive-into-rnn-transformers-ssm-s4-and-mamba-2f4ad0b06613
2. RNNs vs LSTM vs Transformers | SabrePC Blog, 8월 16, 2025에 액세스, https://www.sabrepc.com/blog/deep-learning-and-ai/rnns-vs-lstm-vs-transformers
3. Mamba: Linear-Time Sequence Modeling with Selective State ..., 8월 16, 2025에 액세스, https://www.oxen.ai/blog/mamba-linear-time-sequence-modeling-with-selective-state-spaces-arxiv-dives
4. [D] Can someone describe how the SSM in Mamba is much different than the concepts in a GRU / LSTM Cell? : r/MachineLearning - Reddit, 8월 16, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/18iph6a/d_can_someone_describe_how_the_ssm_in_mamba_is/
5. A Visual Guide to Mamba and State Space Models - Maarten Grootendorst, 8월 16, 2025에 액세스, https://www.maartengrootendorst.com/blog/mamba/
6. Exploring the Limitations of Mamba in COPY and CoT Reasoning - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2410.03810v2
7. Mamba Explained - The Gradient, 8월 16, 2025에 액세스, https://thegradient.pub/mamba-explained/
8. [2312.00752] Mamba: Linear-Time Sequence Modeling with Selective State Spaces - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/2312.00752
9. Mamba: Linear-Time Sequence Modeling with Selective ... - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/pdf/2312.00752
10. What Are State Space Models? | IBM, 8월 16, 2025에 액세스, https://www.ibm.com/think/topics/state-space-model
11. Introduction to State Space Models (SSM) - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/blog/lbourdois/get-on-the-ssm-train
12. State Space Models: Reshaping AI's Future Beyond Transformers - Dragonscale Newsletter, 8월 16, 2025에 액세스, https://blog.dragonscale.ai/state-space-models/
13. Mamba Simplified - Part 2 - S4 and Mamba - Prem AI Blog, 8월 16, 2025에 액세스, https://blog.premai.io/s4-and-mamba/
14. State Space Models For Sequence Modeling | by Atufa Shireen ..., 8월 16, 2025에 액세스, https://atufashireen.medium.com/state-space-models-for-sequence-modeling-a95f47f1265d
15. state-spaces/mamba: Mamba SSM architecture - GitHub, 8월 16, 2025에 액세스, https://github.com/state-spaces/mamba
16. Introduction to State Space Models as Natural Language Models - neptune.ai, 8월 16, 2025에 액세스, https://neptune.ai/blog/state-space-models-as-natural-language-models
17. arxiv.org, 8월 16, 2025에 액세스, [https://arxiv.org/abs/2412.11211#:~:text=State%2Dspace%20models%20(SSMs),the%20values%20of%20the%20observations.](https://arxiv.org/abs/2412.11211#:~:text=State-space models (SSMs),the values of the observations.)
18. Deep Learning-based Approaches for State Space Models: A Selective Review - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2412.11211v1
19. Along comes a Mamba: an evolution in sequence models based on state space models | by Wilbur de Souza | Medium, 8월 16, 2025에 액세스, https://medium.com/@wilburdes/along-comes-a-mamba-an-evolution-in-sequence-models-based-on-state-space-models-2bd3d0e02d86
20. Structured State Spaces: Combining Continuous-Time, Recurrent, and Convolutional Models - Hazy Research, 8월 16, 2025에 액세스, https://hazyresearch.stanford.edu/blog/2022-01-14-s4-3
21. Zero to Mamba: An intuitive explanation to the Mamba Architecture | by AI Club, IITM, 8월 16, 2025에 액세스, https://medium.com/@aiclub.iitm/zero-to-mamba-an-intuitive-explanation-to-the-mamba-architecture-d52265b771ab
22. Discrete time and continuous time - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Discrete_time_and_continuous_time
23. What Is A Mamba Model? - IBM, 8월 16, 2025에 액세스, https://www.ibm.com/think/topics/mamba-model
24. Structured State Spaces for Sequence Modeling (S4) - Hazy Research, 8월 16, 2025에 액세스, https://hazyresearch.stanford.edu/blog/2022-01-14-s4-1
25. Structured State Space Models Visually Explained - Towards Data Science, 8월 16, 2025에 액세스, https://towardsdatascience.com/structured-state-space-models-visually-explained-86cfe2757386/
26. Ophiology (or, how the Mamba architecture works) — LessWrong, 8월 16, 2025에 액세스, https://www.lesswrong.com/posts/TYLQ8gAMAmpeFcwXN/ophiology-or-how-the-mamba-architecture-works
27. Here Comes Mamba: The Selective State Space Model | Towards Data Science, 8월 16, 2025에 액세스, https://towardsdatascience.com/here-comes-mamba-the-selective-state-space-model-435e5d17a451/
28. From S4 to Mamba: A Comprehensive Survey on Structured State Space Models - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/2503.18970
29. EFFICIENTLY MODELING LONG SEQUENCES WITH STRUCTURED STATE SPACES - OpenReview, 8월 16, 2025에 액세스, https://openreview.net/pdf?id=uYLFoz1vlAC
30. Why S4 is Good at Long Sequence: Remembering a Sequence with Online Function Approximation - Yao Fu's Blog | Notion, 8월 16, 2025에 액세스, https://yaofu.notion.site/Why-S4-is-Good-at-Long-Sequence-Remembering-a-Sequence-with-Online-Function-Approximation-836fc54a49aa413b84997a265132f13f
31. What Makes Convolutional Models Great on Long Sequence Modeling? - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/pdf/2210.09298
32. Understanding the Mamba Model - How selective SSMs… - Medium, 8월 16, 2025에 액세스, https://medium.com/@srujananjali888/understanding-the-mamba-model-17610b8a5fe9
33. Mamba: Linear-Time Sequence Modeling with Selective State Spaces - OpenReview, 8월 16, 2025에 액세스, https://openreview.net/forum?id=tEYskw1VY2
34. Mamba No. 5 (A Little Bit Of…) | Sparse Notes, 8월 16, 2025에 액세스, https://jameschen.io/jekyll/update/2024/02/12/mamba.html
35. Benchmarking Mamba's Document Ranking Performance in the Era of Transformers - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2403.18276v2
36. What Are the Training Time and Space Complexities for Mamba vs. Traditional Transformers? · Issue #196 - GitHub, 8월 16, 2025에 액세스, https://github.com/state-spaces/mamba/issues/196
37. Linear Attention Fundamentals | Hailey Schoelkopf, 8월 16, 2025에 액세스, https://haileyschoelkopf.github.io/blog/2024/linear-attn/
38. An Empirical Study of Mamba-based Language Models - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2406.07887v1
39. [D] What Are the Fundamental Drawbacks of Mamba Compared to Transformers? - Reddit, 8월 16, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/1ayog60/d_what_are_the_fundamental_drawbacks_of_mamba/
40. Mamba(2) and Transformer Hybrids: An Overview - Data Artificer and code:Breaker, 8월 16, 2025에 액세스, https://n1o.github.io/posts/ssm-transformer-hybrids-guide/
41. Beyond Attention: Mamba as a replacement for transformers | by Ben Spicer - Medium, 8월 16, 2025에 액세스, https://medium.com/correll-lab/beyond-attention-mamba-as-a-replacement-for-transformers-5483b6e23a6e
42. From Transformer to Mamba - by Manav Gupta - Medium, 8월 16, 2025에 액세스, https://medium.com/@manavg/from-transformer-to-mamba-e2b129e282a8
43. What is Mamba and can it beat Transformers? | by Ksenia Se - Medium, 8월 16, 2025에 액세스, https://kseniase.medium.com/what-is-mamba-and-can-it-beat-transformers-17ea5d0c5d65
44. Mamba: Revolutionizing Sequence Modeling with Selective State Spaces - Medium, 8월 16, 2025에 액세스, https://medium.com/@joeajiteshvarun/mamba-revolutionizing-sequence-modeling-with-selective-state-spaces-8a691319b34b
45. [D] - Why MAMBA did not catch on? : r/MachineLearning - Reddit, 8월 16, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/1hpg91o/d_why_mamba_did_not_catch_on/
46. How transformers, RNNs and SSMs are more alike than you think | by Stanislav Fedotov, 8월 16, 2025에 액세스, https://medium.com/nebius/how-transformers-rnns-and-ssms-are-more-alike-than-you-think-cd0f899893d8
47. [Literature Review] An Empirical Study of Mamba-based Language ..., 8월 16, 2025에 액세스, https://www.themoonlight.io/en/review/an-empirical-study-of-mamba-based-language-models
48. An Empirical Study of Mamba-based Language Models - alphaXiv, 8월 16, 2025에 액세스, https://www.alphaxiv.org/overview/2406.07887
49. Mamba: A Potential Transformer Replacement - Zilliz Learn, 8월 16, 2025에 액세스, https://zilliz.com/learn/mamba-architecture-potential-transformer-replacement
50. Vision Mamba: Efficient Visual Representation Learning with Bidirectional State Space Model - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2401.09417v1
51. Vision Mamba: The Next Leap in Visual Representation Learning | by azhar - Medium, 8월 16, 2025에 액세스, https://medium.com/ai-insights-cobet/vision-mamba-the-next-leap-in-visual-representation-learning-24a10e5a9cde
52. Vision Mamba: Efficient Visual Representation Learning with Bidirectional State Space Model - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/blog/mikelabs/vision-mamba-efficient-visual-representation-learn
53. Is Mamba Effective for Time Series Forecasting? - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2403.11144v3
54. TSMamba : Mamba model for Time Series Forecasting | by Mehul Gupta | Data Science in Your Pocket | Medium, 8월 16, 2025에 액세스, https://medium.com/data-science-in-your-pocket/tsmamba-mamba-model-for-time-series-forecasting-c9eeb0d0d23c
55. [2411.02941] A Mamba Foundation Model for Time Series Forecasting - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/2411.02941
56. Mamba LLM Architecture: A Breakthrough in Efficient AI Modeling | SaM Solutions, 8월 16, 2025에 액세스, https://sam-solutions.com/blog/mamba-llm-architecture/
57. [D] Why is everybody surprised that Mamba got rejected from ICLR? Am I missing something? : r/MachineLearning - Reddit, 8월 16, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/1axsxeo/d_why_is_everybody_surprised_that_mamba_got/
58. A hybrid model based on transformer and Mamba for enhanced sequence modeling - PMC, 8월 16, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11968869/


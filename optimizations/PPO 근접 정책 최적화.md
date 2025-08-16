# 근접 정책 최적화(PPO)

## 1.  정책 최적화 입문

### 1.1  제어 학습의 도전: 강화학습 패러다임 개요

강화학습(Reinforcement Learning, RL)은 인공지능의 한 분야로, 에이전트가 환경과 상호작용하며 시도와 오류를 통해 최적의 행동 방식을 학습하는 패러다임을 의미한다.1 이 과정에서 에이전트는 특정 상태(state)에서 행동(action)을 취하고, 그 결과로 환경으로부터 보상(reward)과 다음 상태 정보를 받는다. 에이전트의 궁극적인 목표는 장기적으로 누적 보상을 최대화하는 것이다.5

강화학습 알고리즘은 크게 가치 기반(value-based) 방법과 정책 기반(policy-based) 방법으로 나뉜다. 딥 Q-러닝(Deep Q-Learning, DQN)과 같은 가치 기반 방법은 각 상태 또는 상태-행동 쌍의 가치를 추정하는 데 중점을 둔다. 반면, 정책 기반 방법은 에이전트의 행동 전략, 즉 정책(policy)을 직접적으로 모델링하고 최적화한다. 특히 정책 기반 방법은 행동 공간이 연속적이거나 차원이 매우 높은 문제에서 가치 기반 방법보다 뛰어난 성능을 보이는 경향이 있다.3 이러한 문제들은 로봇 제어나 복잡한 게임 전략 등 현실 세계의 많은 응용 분야에서 나타난다. 이 모든 과정은 일반적으로 마르코프 결정 과정(Markov Decision Process, MDP)이라는 수학적 프레임워크를 통해 공식화된다.3

### 1.2  정책 경사 접근법: 강점과 내재된 불안정성

정책 경사(Policy Gradient, PG) 방법은 정책 기반 강화학습의 핵심적인 접근법이다. 이 방법은 정책을 신경망과 같은 함수 근사기로 매개변수화하고(πθ), 누적 보상의 기댓값을 최대화하는 방향으로 경사 상승법(gradient ascent)을 사용하여 매개변수 θ를 직접 업데이트한다.9 정책 경사 이론의 기본 목적 함수는 다음과 같이 표현될 수 있다.9
$$
\nabla_{\theta}J(\theta) = \mathbb{E}_{\tau \sim \pi_{\theta}}
$$
여기서 $J(\theta)$는 정책 $\pi_{\theta}$의 성능 척도(예: 총 보상의 기댓값)이고, Rt는 시간 t 이후의 누적 보상이다.

REINFORCE와 같은 "바닐라" 정책 경사 방법은 구현이 직관적이라는 장점이 있지만, 몇 가지 심각한 내재적 약점을 가지고 있다. 첫째, 경사 추정치의 분산(variance)이 매우 높아 학습이 불안정하다. 둘째, 온-정책(on-policy) 방식으로, 각 정책 업데이트를 위해 현재 정책으로 새로 수집한 샘플만을 사용하고 이전 데이터는 폐기해야 하므로 데이터 효율성이 매우 낮다.12 셋째, 학습률(step size)과 같은 하이퍼파라미터에 극도로 민감하다. 너무 큰 업데이트는 정책을 회복 불가능한 수준으로 망가뜨릴 수 있으며(catastrophic performance collapse), 너무 작은 업데이트는 학습을 비효율적으로 만든다.10

### 1.3  PPO의 등장: 안정성, 단순성, 그리고 성능을 향한 탐구

이러한 배경 속에서 2017년 OpenAI에 의해 근접 정책 최적화(Proximal Policy Optimization, PPO)가 제안되었다.1 PPO의 등장은 기존 강화학습 알고리즘들이 직면했던 근본적인 트릴레마, 즉 **안정성-효율성-단순성** 사이의 균형을 맞추려는 시도였다. 바닐라 정책 경사법은 단순하지만 안정성과 효율성이 낮았고, 이를 개선하기 위해 등장한 신뢰 영역 정책 최적화(Trust Region Policy Optimization, TRPO)는 안정성과 데이터 효율성을 크게 높였지만, 구현이 매우 복잡하고 특정 아키텍처에 적용하기 어렵다는 한계를 가졌다.12

PPO는 TRPO와 동일한 근본적인 질문에서 출발했다: "현재 보유한 데이터를 사용하여 성능 저하를 일으키지 않으면서 가능한 가장 큰 폭으로 정책을 개선할 수 있는 방법은 무엇인가?".4 PPO는 이 질문에 대해 TRPO의 강력한 성능과 데이터 효율성은 유지하면서도, 훨씬 간단한 1차 최적화(first-order optimization)만을 사용하여 구현 복잡성을 대폭 낮추는 실용적인 해법을 제시했다.13 이는 이론적 완벽성보다 실용적 유용성을 우선시하는 패러다임의 전환을 의미했으며, 이로 인해 PPO는 강화학습 연구 및 응용 분야에서 가장 인기 있고 신뢰받는 알고리즘 중 하나로 자리매김하게 되었다.

## 2.  신뢰 영역에서 근접 정책으로: PPO의 이론적 계보

### 2.1  신뢰 영역 정책 최적화(TRPO): 원칙에 입각한 선행자

PPO를 깊이 이해하기 위해서는 그 이론적 기반이 된 TRPO를 먼저 살펴보아야 한다. TRPO의 핵심 아이디어는 정책 업데이트의 크기를 명시적으로 제한하여 학습 과정의 안정성을 보장하는 것이다. TRPO는 "대리(surrogate)" 목적 함수를 최대화하되, 이전 정책과 새로운 정책 간의 차이가 특정 "신뢰 영역(trust region)"을 벗어나지 않도록 제약 조건을 부과한다.13 이 정책 간의 거리는 일반적으로 쿨백-라이블러 발산(Kullback-Leibler (KL) divergence)으로 측정된다.12

TRPO의 목적 함수는 수학적으로 다음과 같이 공식화된다 11:
$$
\underset{\theta}{\text{maximize}} \quad \hat{\mathbb{E}}t \left[ \frac{\pi{\theta}(a_t | s_t)}{\pi_{\theta_{\text{old}}}(a_t | s_t)} \hat{A}t \right]
$$

$$
\text{subject to} \quad \hat{\mathbb{E}}t \left[ \text{KL}[\pi{\theta{\text{old}}}(\cdot|s_t), \pi_{\theta}(\cdot|s_t)] \right] \leq \delta
$$

여기서 $A^t$는 어드밴티지 추정치이며, δ는 신뢰 영역의 크기를 결정하는 하이퍼파라미터다. 이 제약 조건은 이론적으로 정책의 단조로운 성능 향상(monotonic improvement)을 보장하는 기반이 된다.26

### 2.2  TRPO의 한계: 계산 복잡성과 아키텍처 비호환성

TRPO는 이론적으로 매우 견고하지만, 실제 적용에는 여러 심각한 장벽이 존재한다. 가장 큰 문제는 제약 조건이 있는 최적화 문제를 풀기 위해 켤레 경사법(conjugate gradient algorithm)과 같은 2차 최적화 기법을 사용해야 한다는 점이다.12 이는 피셔 정보 행렬(Fisher Information Matrix)의 계산 및 관련 연산을 포함하므로 계산 비용이 매우 높고 구현이 극도로 복잡하다.10

더욱이, TRPO는 현대적인 심층 신경망 아키텍처와의 호환성이 떨어진다. 예를 들어, 정책 네트워크와 가치 네트워크 간에 파라미터를 공유하거나, 드롭아웃(dropout)과 같은 노이즈를 추가하는 구조에서는 제대로 작동하지 않는 경우가 많다.13 이러한 제약은 TRPO의 실제적인 활용 범위를 크게 제한하는 요인이 되었다.

PPO는 이러한 TRPO의 한계를 극복하기 위한 시도에서 탄생했다. PPO는 TRPO의 핵심 원칙인 '정책 변화를 제한한다'는 목표는 계승하되, 이를 달성하기 위한 구체적인 메커니즘을 KL 발산이라는 엄격한 제약 조건에서 분리해냈다. PPO 논문은 이 분리를 위한 두 가지 대안을 탐색했다: 하나는 KL 발산을 목적 함수에 페널티 항으로 추가하는 방식(PPO-Penalty)이고, 다른 하나는 확률 비율을 직접 클리핑하는 훨씬 더 단순한 휴리스틱(PPO-Clip)이다.6

결과적으로 더 성공적인 것으로 입증된 PPO-Clip은 목적 함수에서 KL 발산 계산을 완전히 제거했다. 대신, 정책 변화의 *결과*로 나타나는 확률 비율의 급격한 변화라는 *증상*을 직접적으로 제어하는 방식을 택했다.19 이는 TRPO와 같은 수학적으로 엄격한 신뢰 영역 강제 방식이 실제로는 과도한 조치일 수 있음을 시사한다. 더 단순하고 직접적인 휴리스틱이 실제로는 동등하거나 더 나은 성능을 보이면서, 표준적인 딥러닝 최적화기(SGD, Adam 등)와 훨씬 더 잘 호환될 수 있다는 가능성을 열어준 것이다. 이로써 PPO는 이론적 순수성보다 실용성을 우선시하는 중요한 전환을 이루었다.

## 3.  PPO의 핵심 메커니즘 심층 분석

### 3.1  클리핑된 대리 목적 함수: PPO의 중심 혁신

#### 3.1.1  수학적 공식과 직관적 해설

PPO의 가장 핵심적인 아이디어는 '클리핑된 대리 목적 함수(Clipped Surrogate Objective Function)'이다. 이 목적 함수는 TRPO의 복잡한 제약 조건을 대체하면서도 안정적인 정책 업데이트를 가능하게 하는 PPO의 심장부라 할 수 있다. 널리 사용되는 PPO-Clip의 목적 함수는 다음과 같다.11
$$
L^{CLIP}(\theta) = \hat{\mathbb{E}}_t \left[ \min\left( r_t(\theta)\hat{A}_t, \text{clip}(r_t(\theta), 1 - \epsilon, 1 + \epsilon)\hat{A}_t \right) \right]
$$
이 식의 각 구성 요소는 다음과 같은 의미를 가진다:

- rt(θ): **확률 비율(Probability Ratio)**로, 새로운 정책 $\pi_{\theta}$와 이전 정책 πθold 하에서 특정 상태-행동 쌍 $(s_t, a_t)$이 나타날 확률의 비율이다. 즉, $r_t(\theta) = \frac{\pi_{\theta}(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}$이다.11 이 값이 1보다 크면 새로운 정책이 해당 행동을 할 확률이 높아졌음을, 1보다 작으면 낮아졌음을 의미한다.
- A^t: **어드밴티지 추정치(Advantage Estimate)**로, 상태 st에서 행동 at를 취한 것이 평균적인 행동보다 얼마나 더 좋았는지를 나타내는 값이다.5
- ϵ: **클리핑 하이퍼파라미터(Clipping Hyperparameter)**로, 확률 비율 $r_t(\theta)$가 벗어날 수 있는 범위를 제한한다. 일반적으로 0.1에서 0.3 사이의 작은 값으로 설정되며, 논문에서는 0.2를 주로 사용한다.29
- clip(rt(θ),1−ϵ,1+ϵ): $r_t(\theta)$의 값을 [1−ϵ,1+ϵ] 범위 내로 강제로 제한(clipping)하는 함수다.
- min(⋅,⋅): 두 항 중에서 더 작은 값을 선택한다.

이 목적 함수의 핵심 아이디어는 정책 성능에 대한 비관적인 추정치, 즉 **하한(lower bound)**을 형성하는 것이다.13 이를 통해 확률 비율 $r_t(\theta)$가 1에서 너무 멀리 벗어나려는 유인을 제거하여, 과도하게 큰 정책 업데이트를 방지한다.21

#### 3.1.2  목적 함수의 시각화: 클리핑이 업데이트를 제어하는 방식

이 목적 함수의 작동 방식은 어드밴티지(A^t)의 부호에 따라 두 가지 경우로 나누어 시각적으로 이해할 수 있다.22

- **어드밴티지가 양수일 때 (A^t>0):** 이 경우는 에이전트가 취한 행동이 평균보다 좋았다는 의미이므로, 해당 행동의 확률을 높이는 방향($r_t(\theta)$를 증가시키는 방향)으로 정책을 업데이트하고자 한다. 목적 함수는 $r_t(\theta)$가 증가함에 따라 함께 증가하지만, $r_t(\theta)$가 1+ϵ을 초과하는 순간 `clip` 함수가 작동하여 목적 함수의 값이 더 이상 증가하지 않고 평평해진다. `min` 함수는 이 평평한 값(클리핑된 값)을 선택하게 된다. 이는 "좋은 행동에 대해 보상하되, 너무 과도하게 낙관하여 한 번에 정책을 크게 바꾸는 것은 막겠다"는 보수적인 업데이트 전략을 의미한다.21
- **어드밴티지가 음수일 때 (A^t<0):** 이 경우는 행동이 평균보다 나빴다는 의미이므로, 해당 행동의 확률을 낮추는 방향($r_t(\theta)$를 감소시키는 방향)으로 정책을 업데이트하고자 한다. 목적 함수는 $r_t(\theta)$가 감소함에 따라 증가(음수 값이 0에 가까워짐)한다. 하지만 $r_t(\theta)$가 1−ϵ보다 작아지면, `clip` 함수가 이 감소를 제한한다. `min` 함수는 두 음수 값 중 더 작은 값(더 큰 페널티)을 취하게 되는데, 이는 정책이 나쁜 방향으로 급격히 변하는 것도 방지하는 효과를 낳는다.21

결론적으로, 그래프의 평평한 부분은 클리핑으로 인해 목적 함수의 기울기가 0이 되는 지점을 나타낸다. 이는 해당 지점을 넘어서는 정책 변화에 대해서는 더 이상의 이득(인센티브)이 없음을 의미하며, 이로 인해 정책 업데이트가 안정적인 범위 내에 머무르게 된다.34

이 `min` 연산자의 비대칭적인 작동 방식이야말로 PPO 안정성의 숨겨진 비결이다. 긍정적 어드밴티지의 경우, `min` 함수는 잠재적 이득이 너무 클 때 이를 제한하여(클리핑된 값을 선택) 과도한 낙관론을 막는다.19 반면 부정적 어드밴티지의 경우, 

`min` 함수는 잠재적 손실이 클 때 이를 그대로 반영하여(클리핑되지 않은 값을 선택할 가능성이 높음) 파괴적인 업데이트를 신속하게 바로잡도록 한다. 이처럼 "한쪽으로만 열린 안전 레일"과 같은 비관적 경계 설정은 복잡한 계산 없이도 신뢰 영역의 핵심적인 이점을 구현하는 영리한 방법이다.13

### 3.2  PPO 변형: PPO-Clip 대 PPO-Penalty

PPO 원본 논문에서는 두 가지 주요 변형을 제안했다.6

- **PPO-Clip:** 위에서 설명한 클리핑된 대리 목적 함수를 사용하는 방식이다. 구현이 간단하고 대부분의 환경에서 뛰어난 성능을 보여 현재 PPO의 표준으로 여겨진다.21

- **PPO-Penalty (PPO-KL):** TRPO와 유사하게 KL 발산을 목적 함수에 페널티 항으로 추가하는 방식이다. 이 변형은 목표 KL 발산 값에 도달하도록 페널티 계수 β를 동적으로 조정하는 메커니즘을 포함한다.6 하지만 적절한 

  β를 찾는 것이 어렵고, 실험적으로 PPO-Clip보다 성능이 떨어지는 경우가 많아 덜 사용된다.13

### 3.3  PPO 알고리즘: 단계별 절차

PPO의 전체적인 학습 과정은 다음과 같은 단계로 요약할 수 있다.5

1. **데이터 수집 (Data Collection):** 현재 정책 $\pi_{\theta_{old}}$를 사용하여 환경과 T 타임스텝 동안 상호작용하며 궤적(trajectories) 데이터 ${s_t, a_t, r_t, \dots}$를 수집한다. 이 데이터 묶음을 롤아웃(rollout) 또는 배치(batch)라고 한다.6
2. **어드밴티지 계산 (Advantage Computation):** 수집된 궤적 데이터를 사용하여 각 타임스텝 t에 대한 어드밴티지 추정치 A^t와 가치 목표 $V_{target}$을 계산한다. 이 과정에서는 보통 GAE(Generalized Advantage Estimation)가 사용된다 (섹션 5에서 상세히 다룸).5
3. **정책 및 가치 함수 최적화 (Optimization):** K 에포크(epoch) 동안 수집된 데이터 배치를 반복적으로 사용한다. 각 에포크마다 데이터를 여러 미니배치(mini-batch)로 나누어 다음을 수행한다:
   - 정책 네트워크(Actor)의 파라미터 θ를 LCLIP 목적 함수에 대한 확률적 경사 상승법(stochastic gradient ascent)을 통해 업데이트한다.
   - 가치 네트워크(Critic)의 파라미터 ϕ를 가치 손실(value loss)에 대한 경사 하강법을 통해 업데이트한다.
4. **반복:** 1~3단계를 수렴할 때까지 반복한다.

PPO가 바닐라 정책 경사법에 비해 샘플 효율성이 높은 이유는 바로 3단계에 있다. 새로운 목적 함수 덕분에 한 번 수집한 데이터를 여러 에포크에 걸쳐 재사용하여 정책을 여러 번 업데이트할 수 있기 때문이다.14

## 4.  PPO의 액터-크리틱 프레임워크

PPO는 일반적으로 액터-크리틱(Actor-Critic) 구조를 기반으로 구현된다. 이 구조는 정책을 학습하는 '액터'와 그 정책을 평가하는 '크리틱'이라는 두 개의 구성 요소를 동시에 학습시켜 강화학습의 효율성과 안정성을 높인다.7

### 4.1  액터와 크리틱의 역할

- **액터 (Actor):** 정책 네트워크(Policy Network)라고도 불리며, 현재 상태(s)를 입력받아 어떤 행동(a)을 취할지에 대한 확률 분포를 출력한다. 즉, 에이전트의 '행동 주체'이다. PPO의 목표는 이 액터의 정책을 개선하여 누적 보상을 최대화하는 것이다.7
- **크리틱 (Critic):** 가치 네트워크(Value Network)라고도 불리며, 현재 상태(s)를 입력받아 그 상태의 가치(V(s)), 즉 해당 상태에서부터 미래에 얻을 수 있는 보상의 총합에 대한 기댓값을 추정한다. 크리틱은 직접 행동을 결정하지 않지만, 액터가 취한 행동이 얼마나 좋았는지를 평가하는 '평론가' 역할을 한다.7 크리틱이 추정한 가치 함수는 어드밴티지 함수를 계산하는 데 사용되어 정책 경사의 분산을 줄이는 데 결정적인 기여를 한다.3

PPO에서 크리틱의 역할은 액터의 학습을 돕는 보조적인 기능에 국한된다. 바닐라 정책 경사법에서는 전체 에피소드의 보상 합계(Rt)를 사용하는데, 이는 무작위적인 행동과 상태 전이에 따라 변동성이 매우 크다. 이 높은 분산을 줄이기 위해 Rt에서 상태 가치 함수 $V(s_t)$라는 기준선(baseline)을 빼주는 기법이 사용된다. 그 결과로 나오는 $\hat{A}_t = R_t - V(s_t)$가 바로 어드밴티지 추정치다. 크리틱의 임무는 이 기준선 $V(s_t)$를 정확하게 학습하는 것이다. 더 정확한 크리틱은 더 낮은 분산의 어드밴티지 추정치를 제공하고, 이는 액터에게 더 안정적이고 명확한 학습 신호를 주어 전체적인 학습 효율을 높인다. 따라서 PPO는 근본적으로 정책 기반 방법이며, 크리틱은 액터의 학습 과정을 안정화시키기 위한 실용적인 도구로 기능한다. 이러한 구조적 분리는 PPO가 액터와 크리틱에 별도의 네트워크를 사용할 수 있게 하여 알고리즘의 유연성을 높이는 요인이 된다.

### 4.2  네트워크 아키텍처: 파라미터 공유 대 분리

액터와 크리틱 네트워크를 구현할 때 두 가지 일반적인 아키텍처 선택지가 있다.13

- **파라미터 공유 (Shared Parameters):** 액터와 크리틱이 입력층과 일부 은닉층을 공유하고, 마지막에 각각 정책(행동 확률)과 가치(스칼라 값)를 출력하는 별도의 헤드(head)를 두는 구조다. 이 방식은 상태에 대한 특징 추출 부분을 공유하므로 계산 효율성이 높고 파라미터 수를 줄일 수 있다.13
- **분리된 네트워크 (Separate Networks):** 액터와 크리틱이 완전히 독립된 두 개의 신경망으로 구성된다. 정책과 가치 함수는 서로 다른 최적화 목표를 가지기 때문에, 두 네트워크를 분리하면 학습 과정에서 서로 간섭이 줄어들어 더 안정적인 학습으로 이어질 수 있다.43 PPO는 TRPO와 달리 두 방식 모두와 호환 가능하여 높은 유연성을 자랑한다.13

### 4.3  가치 함수 손실과 정책 업데이트에서의 역할

크리틱, 즉 가치 네트워크는 지도 학습 방식으로 훈련된다. 일반적으로 예측된 가치 $V_{\phi}(s_t)$와 실제 관측된 보상에 기반한 목표 가치 Vtarget 사이의 평균 제곱 오차(Mean Squared Error, MSE)를 최소화하는 방향으로 파라미터 ϕ를 업데이트한다.5

따라서 PPO의 전체 손실 함수는 보통 세 가지 요소의 가중합으로 구성된다: (1) 정책 손실(액터, LCLIP), (2) 가치 손실(크리틱, LVF), 그리고 (3) 탐험을 장려하기 위한 선택적 엔트로피 보너스(S).13

Ltotal(θ,ϕ)=LCLIP(θ)−c1LVF(ϕ)+c2S[πθ](st)

여기서 c1과 c2는 각 손실 항의 중요도를 조절하는 계수다.14 이 통합된 손실 함수를 최적화함으로써 액터와 크리틱이 협력하여 에이전트의 전반적인 성능을 향상시킨다.

## 5.  일반화된 어드밴티지 추정(GAE)을 통한 학습 안정화

PPO의 성능을 논할 때, 정책 업데이트 규칙만큼이나 중요한 것이 바로 어드밴티지(A^t)를 얼마나 정확하고 안정적으로 추정하는가이다. PPO 구현체들은 대부분 일반화된 어드밴티지 추정(Generalized Advantage Estimation, GAE)이라는 기법을 사용하여 이 문제를 해결한다. GAE는 PPO의 핵심적인 안정성과 성능에 기여하는 필수적인 요소로 자리 잡았다.

### 5.1  어드밴티지 추정의 편향-분산 트레이드오프

어드밴티지 함수 $A(s, a) = Q(s, a) - V(s)$를 추정하는 데에는 근본적인 편향-분산 트레이드오프(bias-variance tradeoff)가 존재한다.

- **단일 스텝 TD 오차 (High Bias, Low Variance):** 가장 간단한 추정 방식은 1-스텝 시간차(Temporal Difference, TD) 오차인 $\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$를 사용하는 것이다. 이 추정치는 단 하나의 다음 스텝만을 고려하므로 분산이 낮지만, 현재 가치 함수 V가 부정확할 경우 심각한 편향(bias)을 갖게 된다.48
- **몬테카를로 추정 (Low Bias, High Variance):** 반대로 에피소드가 끝날 때까지의 모든 보상을 합산하여 계산하는 몬테카를로 방식은 이론적으로 편향이 없지만, 매우 긴 시간 동안의 무작위적인 행동과 상태 전이에 의존하기 때문에 분산이 극도로 높다.48

GAE는 이 두 극단 사이에서 균형을 맞추기 위해 제안되었다.49

### 5.2  GAE 공식: TD 잔차의 지수 가중 평균

GAE는 모든 n-스텝 어드밴티지 추정치의 지수 가중 평균으로 정의되며, 그 공식은 다음과 같다.46

A^tGAE(γ,λ)=l=0∑∞(γλ)lδt+l

여기서 $\delta_{t+l} = r_{t+l} + \gamma V(s_{t+l+1}) - V(s_{t+l})$은 미래 시점의 TD 잔차(TD-residual)이다. 이 공식은 1-스텝 TD 오차(높은 편향, 낮은 분산)와 몬테카를로 추정(낮은 편향, 높은 분산) 사이를 매끄럽게 보간하는 효과를 가진다.48

### 5.3  하이퍼파라미터 γ와 λ의 역할

GAE의 동작은 두 개의 하이퍼파라미터에 의해 제어된다.

- γ (감마): 표준적인 할인율(discount factor)로, 미래 보상의 현재 가치를 결정한다. 1에 가까울수록 미래 보상을 중요하게 생각한다.8
- λ (람다): GAE 고유의 파라미터로, 편향과 분산 사이의 트레이드오프를 직접적으로 제어한다.48
  - λ=0일 경우: GAE는 A^t=δt가 되어 1-스텝 TD 오차와 동일해진다. 이는 분산은 낮지만 편향이 크다.48
  - λ=1일 경우: GAE는 몬테카를로 방식의 어드밴티지 추정치와 같아진다. 이는 편향은 낮지만 분산이 매우 크다.14
  - 실제로는 0<λ<1 사이의 값(예: 0.95 또는 0.97)을 사용하여 편향과 분산 사이의 적절한 균형점을 찾는다.50

PPO의 성공은 단순히 클리핑된 목적 함수라는 정책 업데이트 규칙에만 기인하는 것이 아니다. 그 업데이트 규칙에 입력되는 어드밴티지 신호의 품질 또한 매우 중요하다. GAE는 크리틱의 가치 함수 V를 영리하게 활용하여 편향과 분산이 낮은 어드밴티지 추정치 A^t를 만들어낸다.49 PPO의 안정적인 업데이트 규칙과 GAE의 안정적인 어드밴티지 추정이 결합될 때 시너지 효과가 발생하며, 이것이 바로 PPO가 많은 벤치마크에서 강력한 성능을 보이는 이유 중 하나다. 이는 PPO의 모듈식 설계가 성공적인 강화학습 알고리즘의 좋은 예시임을 보여준다.

## 6.  정책 최적화 알고리즘 비교 분석

PPO의 특징과 장점을 명확히 이해하기 위해서는 동시대의 다른 주요 정책 최적화 알고리즘들과의 비교가 필수적이다. 이 섹션에서는 PPO를 TRPO, A2C, 그리고 대표적인 오프-정책 알고리즘인 SAC와 여러 측면에서 비교 분석한다.

### 6.1  PPO 대 TRPO: 이론과 실제의 상세 비교

PPO와 그 직접적인 전신인 TRPO는 정책 업데이트를 제한하여 안정성을 확보한다는 동일한 철학을 공유하지만, 그 접근 방식에서 극명한 차이를 보인다.

- **이론적 기반:** TRPO는 정책의 단조로운 성능 향상을 이론적으로 보장하는 수학적 기반 위에 서 있다.26 반면 PPO의 클리핑된 목적 함수는 이러한 보장을 제공하지 않으며, 대신 성능의 하한선을 구성하는 보다 휴리스틱하고 경험적인 접근법을 취한다.12
- **최적화 방법:** TRPO는 KL 발산 제약 조건을 만족시키기 위해 켤레 경사법과 같은 복잡하고 계산 비용이 높은 2차 최적화 방법을 사용한다.5 반면 PPO는 표준적인 1차 최적화 방법(SGD, Adam 등)을 사용하여 구현이 훨씬 간단하고 계산적으로 효율적이다.5
- **구현 및 일반성:** 이러한 최적화 방법의 차이는 구현 난이도와 일반성으로 직결된다. PPO는 TRPO에 비해 구현이 압도적으로 간단하며, 파라미터 공유나 드롭아웃 같은 다양한 신경망 아키텍처와 쉽게 호환된다. TRPO는 이러한 구조에서 사용하기 어렵다.3
- **성능:** 놀랍게도, 이론적 보장이 적음에도 불구하고 PPO는 수많은 벤치마크에서 TRPO와 동등하거나 그 이상의 성능을 보여준다.14 이는 PPO의 실용적인 설계가 복잡한 이론을 뛰어넘는 효과를 낼 수 있음을 시사한다.

### 6.2  PPO 대 A2C: 표준 액터-크리틱의 개선

A2C(Advantage Actor-Critic)는 A3C의 동기식 버전으로, 기본적인 온-정책 액터-크리틱 알고리즘의 표준으로 간주된다.63 PPO는 A2C에서 한 단계 더 나아간 발전을 보여준다.

가장 큰 차이점은 데이터 활용 방식에 있다. A2C는 환경으로부터 수집한 데이터 샘플 하나당 한 번의 그래디언트 업데이트를 수행하고 데이터를 폐기한다. 반면, PPO는 클리핑된 대리 목적 함수 덕분에 한 번 수집한 데이터 배치(batch)를 가지고 여러 에포크(epoch) 동안 미니배치 업데이트를 반복할 수 있다.16 이는 PPO가 A2C보다 훨씬 높은 샘플 효율성을 갖게 되는 핵심적인 이유다. 또한, PPO의 클리핑 메커니즘은 A2C의 제약 없는 업데이트보다 더 나은 안정성을 제공하여, 대부분의 벤치마크에서 A2C를 능가하는 성능을 보인다.3

### 6.3  PPO 대 SAC: 온-정책 안정성 대 오프-정책 샘플 효율성

PPO를 선도적인 오프-정책(off-policy) 알고리즘인 SAC(Soft Actor-Critic)와 비교하는 것은 온-정책과 오프-정책 패러다임 간의 근본적인 트레이드오프를 이해하는 데 도움이 된다.

- **온-정책 대 오프-정책:** 이것이 가장 근본적인 차이점이다. PPO는 현재 정책으로 생성한 최신 데이터만을 사용하여 학습한다(온-정책). 반면 SAC는 과거의 경험 데이터를 리플레이 버퍼(replay buffer)에 저장해두고 반복적으로 재사용하여 학습한다(오프-정책).66 이로 인해 SAC는 PPO보다 월등히 높은 샘플 효율성을 가진다. 이는 로봇 공학과 같이 실제 환경에서 데이터를 수집하는 비용이 매우 비싼 응용 분야에서 결정적인 장점이 될 수 있다.68
- **안정성 대 복잡성:** PPO는 일반적으로 SAC보다 학습이 더 안정적이고 하이퍼파라미터에 덜 민감한 것으로 알려져 있다.4 SAC는 엔트로피 가중치를 조절하는 온도(temperature) 파라미터 등 튜닝해야 할 요소가 더 많아 구현과 튜닝이 더 복잡할 수 있다.68
- **탐험:** SAC는 목적 함수에 엔트로피 최대화 항을 명시적으로 포함하여 강력하고 체계적인 탐험을 장려한다. PPO도 엔트로피 보너스를 사용할 수 있지만, 이는 SAC만큼 핵심적인 요소는 아니다.66
- **성능:** MuJoCo와 같은 많은 연속 제어 벤치마크에서, SAC는 종종 PPO보다 더 높은 최종 점수를 달성한다.70 그럼에도 불구하고 PPO는 그 안정성과 신뢰성 덕분에 여전히 매우 강력하고 인기 있는 베이스라인 알고리즘으로 남아있다.

다음 표는 주요 정책 최적화 알고리즘들의 특징을 요약하여 비교한 것이다.

| **알고리즘** | **이론적 기반**             | **업데이트 메커니즘**                        | **온/오프-정책** | **샘플 효율성**             | **안정성 및 견고성**         | **구현 복잡성** | **주요 사용 사례**                                  |
| ------------ | --------------------------- | -------------------------------------------- | ---------------- | --------------------------- | ---------------------------- | --------------- | --------------------------------------------------- |
| **A2C**      | 표준 정책 경사              | 샘플당 단일 업데이트                         | 온-정책          | 낮음                        | 보통 (큰 업데이트에 취약)    | 낮음            | 베이스라인, 간단한 문제                             |
| **TRPO**     | 단조로운 성능 향상 보장     | KL 제약 2차 최적화                           | 온-정책          | 중간                        | 높음 (이론적으로 견고)       | 매우 높음       | 벤치마킹, 복잡성을 감수하고 최고 성능이 필요할 때   |
| **PPO**      | 휴리스틱 신뢰 영역 (하한선) | 클리핑된 1차 최적화                          | 온-정책          | 좋음 (다중 에포크 업데이트) | 높음 (경험적으로 견고)       | 낮음            | 대부분 문제의 기본 선택지, 로보틱스, 게임, LLM 정렬 |
| **SAC**      | 최대 엔트로피 강화학습      | 리플레이 버퍼를 사용한 오프-정책 액터-크리틱 | 오프-정책        | 매우 높음                   | 보통 (하이퍼파라미터에 민감) | 중간            | 샘플 비용이 비싼 작업(실세계 로보틱스), 연속 제어   |

## 7.  실무자를 위한 안내서: PPO의 핵심 구현 상세

PPO의 성공은 논문에 명시된 핵심 알고리즘뿐만 아니라, 실제 고성능 구현체에 포함된 수많은 '코드 레벨 최적화(code-level optimizations)' 또는 '트릭(tricks)'에 크게 의존한다. 이 섹션에서는 이러한 실용적인 기법들을 심도 있게 다룬다.

### 7.1  "비법 소스": 코드 레벨 최적화가 중요한 이유

PPO 논문 자체와 OpenAI Baselines나 Stable Baselines3 같은 라이브러리의 실제 구현 사이에는 상당한 간극이 존재한다.73 여러 연구에 따르면, 이러한 코드 레벨 최적화 기법들이 PPO가 TRPO를 능가하는 성능을 보이는 데 결정적인 역할을 하며, 때로는 클리핑된 목적 함수 자체보다 더 중요할 수 있음이 밝혀졌다.74 이는 PPO의 성공이 뛰어난 핵심 아이디어와 탁월한 엔지니어링의 결합임을 시사한다. 즉, 순수한 PPO 알고리즘만 구현해서는 최고 수준의 결과를 재현하기 어려우며, 이러한 '트릭'들을 이해하고 적용하는 것이 실무적으로 매우 중요하다.

### 7.2  핵심 기법들

고성능 PPO 구현에 필수적인 주요 코드 레벨 최적화 기법들은 다음과 같다.43

- **가치 함수 클리핑 (Value Function Clipping):** 가치 함수의 손실(loss)을 정책 손실과 유사한 방식으로 클리핑한다. 이는 크리틱의 업데이트 폭을 제한하여, 가치 함수의 급격한 변화가 정책 학습을 불안정하게 만드는 것을 방지한다.43
- **어드밴티지 정규화 (Advantage Normalization):** GAE를 통해 계산된 어드밴티지 값들을 미니배치 단위로 정규화한다(평균을 빼고 표준편차로 나눔). 이는 어드밴티지 값의 스케일에 상관없이 안정적인 그래디언트를 생성하여 정책 업데이트를 안정시킨다.43
- **관측 정규화 및 클리핑 (Observation Normalization & Clipping):** 환경으로부터 받는 관측(observation) 값을 이동 평균과 표준편차를 이용해 정규화한 후, 특정 범위(예: [-10, 10])로 클리핑한다. 이는 신경망에 일관된 스케일의 입력을 제공하여 학습을 안정시킨다.44
- **보상 스케일링 및 클리핑 (Reward Scaling & Clipping):** 보상을 할인된 보상 합계의 표준편차 등으로 스케일링하고 클리핑한다. 이는 환경마다 다른 보상 스케일 문제를 완화하여 알고리즘의 일반성을 높인다.47
- **학습률 어닐링 (Learning Rate Annealing):** 훈련이 진행됨에 따라 Adam과 같은 옵티마이저의 학습률을 선형적으로 감소시킨다. 초기에는 높은 학습률로 빠르게 탐색하고, 후반에는 낮은 학습률로 미세 조정하여 안정적인 수렴을 돕는다.43
- **전역 그래디언트 클리핑 (Global Gradient Clipping):** 그래디언트 폭주(exploding gradients)를 막기 위해, 정책 및 가치 네트워크의 모든 파라미터에 대한 그래디언트의 전체 L2 노름(norm)이 특정 임계값(예: 0.5)을 넘지 않도록 클리핑한다.44
- **직교 초기화 (Orthogonal Initialization):** 신경망의 가중치를 특정 방식으로 초기화한다. 예를 들어, 은닉층은 직교 초기화를 사용하고 출력층은 더 작은 스케일로 초기화하여 학습 초기 단계의 안정성을 높인다.44
- **조기 종료 (Early Stopping):** 정책 최적화 에포크를 진행하는 동안, 이전 정책과 현재 정책 간의 KL 발산을 모니터링하여 이 값이 미리 정해진 임계치를 초과하면 업데이트를 조기에 중단한다. 이는 한 배치에 과적합되는 것을 방지한다.21

이러한 기법들은 PPO라는 알고리즘이 단순한 이론을 넘어, 실제 문제에서 강력한 성능을 발휘하는 '하나의 잘 조율된 시스템'으로 작동하게 만드는 핵심 요소들이다. 이는 딥러닝 강화학습 연구에서 알고리즘의 성공이 핵심 아이디어뿐만 아니라 그 주변을 둘러싼 엔지니어링의 정교함에 크게 의존한다는 점을 명확히 보여준다.

다음 표는 PPO의 주요 하이퍼파라미터와 그 튜닝에 대한 실용적인 가이드를 제공한다.

| **하이퍼파라미터**   | **기호**      | **역할 및 설명**                                             | **일반적인 범위**                | **튜닝 조언 및 핵심 정보**                                   |
| -------------------- | ------------- | ------------------------------------------------------------ | -------------------------------- | ------------------------------------------------------------ |
| **할인율**           | `gamma`       | 미래 보상의 중요도를 제어. 장기 계획이 중요할수록 높은 값 사용. | `0.8` - `0.995`                  | 즉각적인 보상이 중요하면 낮게, 지연된 보상이 중요하면 높게 설정. 8 |
| **GAE 파라미터**     | `lambda`      | 어드밴티지 추정의 편향-분산 트레이드오프 제어.               | `0.9` - `0.97`                   | 낮은 값은 크리틱에 더 의존(편향 증가), 높은 값은 실제 보상에 더 의존(분산 증가). 55 |
| **클립 범위**        | `epsilon`     | LCLIP 목적 함수에서 정책 업데이트 크기를 제한하는 범위.      | `0.1` - `0.3`                    | 사실상 표준은 `0.2`. 작은 값은 더 안정적이지만 느린 업데이트를 유발. 21 |
| **학습률**           | `lr`          | Adam 옵티마이저의 스텝 크기.                                 | `1e-5` - `1e-3`                  | 종종 훈련 과정에서 점진적으로 감소(어닐링). 훈련이 불안정하면 감소시킴. 47 |
| **최적화 에포크**    | `num_epoch`   | 수집된 데이터 배치를 반복하여 학습하는 횟수.                 | `3` - `10`                       | 에포크가 많을수록 샘플 효율성은 높아지나, 현재 배치에 과적합될 위험이 있음. 57 |
| **미니배치 크기**    | `batch_size`  | 각 그래디언트 업데이트에 사용되는 샘플의 수.                 | `32-512`(이산), `512-5120`(연속) | `buffer_size`의 약수여야 함. 연속 행동 공간에서 더 큰 값을 사용. 57 |
| **롤아웃 버퍼 크기** | `buffer_size` | 최적화 단계 전에 수집하는 경험의 총량.                       | `2048` - `409600`                | 큰 버퍼는 더 안정적인 업데이트를 유도하지만 정책 업데이트 주기가 길어짐. 57 |
| **엔트로피 계수**    | `ent_coef`    | 탐험을 장려하기 위한 엔트로피 보너스의 가중치.               | `0.0` - `0.01`                   | 높은 값은 더 무작위적인 행동을 장려. 조기 수렴 방지에 유용. 47 |

## 8.  PPO의 실제 적용: 주요 사례 연구

PPO의 진정한 가치는 이론적 우아함을 넘어 다양한 분야의 복잡한 문제들을 해결하는 데 성공적으로 적용되었다는 점에서 드러난다. PPO의 다재다능함은 이 알고리즘이 문제의 구조에 대해 최소한의 가정만을 하기 때문에 가능하다. 로봇 공학의 연속적인 물리 법칙, 도타 2의 방대한 이산적 전략 공간, 대규모 언어 모델의 텍스트 생성 등 근본적으로 다른 문제들에 모두 적용될 수 있다. PPO의 핵심 업데이트 규칙은 확률 비율(rt(θ))과 어드밴티지 추정치(A^t)만 계산할 수 있다면 작동한다.21 이는 PPO가 범용적인 정책 '최적화기'로서 기능하며, 문제의 특수성은 네트워크 아키텍처(예: 게임의 기억을 위한 LSTM, 언어의 구조를 위한 Transformer)나 보상 함수 설계에 위임될 수 있음을 의미한다.

### 8.1  로보틱스: 시뮬레이션 보행에서 실제 조작까지

PPO는 연속적인 제어 작업에서의 안정성 덕분에 로보틱스 분야에서 광범위하게 사용되고 있다.3

- **사례 1: 시뮬레이션 로봇 보행:** PPO는 MuJoCo 물리 엔진을 사용한 시뮬레이션 환경에서 로봇 보행과 같은 작업에 대한 벤치마크에서 지속적으로 강력한 성능을 입증했다.13
- **사례 2: 지도 없는 이동 로봇 내비게이션:** PPO는 LiDAR와 같은 센서 데이터를 직접 입력받아 연속적인 속도 명령을 출력하도록 로봇을 훈련시키는 종단간(end-to-end) 학습에 성공적으로 사용되었다. 이는 PPO가 복잡한 센서 데이터로부터 직접 제어 정책을 학습할 수 있음을 보여준다.84
- **사례 3: 자율주행:** CARLA와 같은 시뮬레이터에서 자율주행차의 의사결정 모델을 훈련하는 데 PPO가 활용되었다. 한 연구에서는 표준 PPO의 탐험 능력을 개선하기 위해 레비 플라이트(Lévy flight) 전략을 통합한 LFPPO 알고리즘을 개발했다. 그 결과, 성공률이 표준 PPO의 81%에서 99%로 향상되었고, 충돌률은 19%에서 1%로 크게 감소하는 등 괄목할 만한 성과를 거두었다.86
- **사례 4: 다중 팔 사과 수확 로봇:** 복잡한 다중 에이전트 농업 로봇 시나리오에서, LSTM과 PPO를 결합한 모델이 동적 작업 계획을 수행하도록 훈련되었다. 이 연구에서는 효율성을 높이고 로봇 팔 간의 충돌을 피하기 위한 정교한 보상 함수 설계가 핵심적인 역할을 했다.87

### 8.2  게임 및 전략: OpenAI Five와 도타 2 사례

PPO의 가장 상징적인 성공 사례 중 하나는 OpenAI Five가 복잡한 e스포츠 게임인 도타 2(Dota 2)에서 세계 챔피언 팀을 꺾은 것이다.5

- **문제의 규모:** 도타 2는 강화학습 에이전트에게 극심한 도전을 제기한다. 게임 한 판당 약 20,000번의 의사결정이 필요한 긴 시간 지평(long time horizon), 불완전한 정보(부분 관측 상태), 그리고 수만 가지에 이르는 방대한 상태-행동 공간이 특징이다.89
- **훈련 인프라:** 이 문제를 해결하기 위해 OpenAI는 전례 없는 규모의 분산 훈련 시스템을 구축했다. 수천 개의 GPU를 사용하여 10개월 동안 자가 대전(self-play) 방식으로 훈련이 진행되었으며, 2초마다 약 200만 프레임의 데이터를 처리했다.88
- **아키텍처:** 5명의 영웅은 각각 동일한 네트워크의 복제본에 의해 제어되었다. 이 네트워크는 주로 단일 레이어의 4096 유닛 LSTM(Long Short-Term Memory)으로 구성되어 게임의 시계열 정보를 처리했으며, 총 파라미터 수는 약 1억 5900만 개에 달했다.89
- **PPO의 역할:** PPO는 각 에이전트의 정책을 최적화하는 핵심 학습 알고리즘으로 사용되었다.90 특히, 단기적인 이득(골드 획득, 적 처치 등)보다 게임 승리라는 장기적인 목표에 큰 가중치를 둔 보상 함수 설계가 OpenAI Five의 초인적인 전략 수립 능력의 핵심이었다.91

### 8.3  새로운 지평: RLHF를 통한 대규모 언어 모델 정렬

최근 PPO는 인공지능 분야의 가장 뜨거운 주제인 대규모 언어 모델(LLM)의 정렬(alignment)에 핵심적인 역할을 수행하고 있다. 인간 피드백 기반 강화학습(Reinforcement Learning from Human Feedback, RLHF)은 GPT-3, InstructGPT, Llama 2와 같은 모델을 인간의 의도에 맞게 미세 조정하는 데 사용되는 표준적인 방법론이다.92

RLHF는 일반적으로 세 단계로 진행된다: 1) 지도 미세 조정(Supervised Fine-Tuning, SFT), 2) 인간의 선호를 학습하는 보상 모델(Reward Model, RM) 훈련, 3) PPO를 사용한 강화학습 미세 조정.93 이 마지막 단계에서 PPO는 LLM(정책)이 보상 모델로부터 높은 점수를 받도록 최적화한다. 이때, 초기 SFT 모델로부터 정책이 너무 멀리 벗어나 '파국적 망각(catastrophic forgetting)'이 일어나는 것을 방지하기 위해 KL 페널티 항이 추가된다.92 이처럼 PPO는 현대 AI의 안전성과 유용성을 확보하는 데 필수적인 기술로 자리 잡았다.

## 9.  한계, 실패 모드, 그리고 "정렬세"

PPO는 수많은 성공에도 불구하고 몇 가지 근본적인 한계와 실패 가능성을 내포하고 있다. 이러한 단점을 이해하는 것은 PPO를 적재적소에 사용하고 향후 연구 방향을 가늠하는 데 중요하다.

### 9.1  온-정책의 병목 현상: 샘플 비효율성

PPO의 가장 큰 약점은 온-정책(on-policy) 알고리즘이라는 점이다.21 이는 정책을 한 번 업데이트하기 위해 반드시 현재 정책으로 수집한 새로운 데이터가 필요하며, 과거 데이터는 재사용하지 않고 폐기됨을 의미한다.15 PPO가 한 배치 내에서 여러 에포크를 학습함으로써 바닐라 정책 경사법보다는 샘플 효율성을 개선했지만, 리플레이 버퍼를 통해 과거 데이터를 광범위하게 재사용하는 SAC나 DQN과 같은 오프-정책(off-policy) 알고리즘에 비해서는 근본적으로 샘플 효율성이 낮다.69 이 문제는 데이터 수집 비용이 비싼 실제 로봇이나 실제 사용자 상호작용과 같은 응용 분야에서 큰 제약이 될 수 있다.68

### 9.2  희소 보상 및 지역 최적점 문제

대부분의 정책 경사 방법과 마찬가지로, PPO는 희소 보상(sparse rewards) 환경에서 어려움을 겪는다.77 에이전트가 의미 있는 피드백을 매우 드물게 받는 환경에서는 어떤 행동이 좋은 결과로 이어지는지 학습하기가 매우 어렵다.

또한, PPO의 안정성을 보장하는 바로 그 메커니즘이 때로는 독이 될 수 있다. PPO의 클리핑된 목적 함수는 정책이 급격하게 변하는 것을 막는다.21 이 보수적인 성향 때문에, 에이전트가 일단 괜찮은(하지만 최적은 아닌) 전략을 발견하면 그 주변을 맴돌며 더 이상의 탐험을 주저하게 될 수 있다. 즉, 더 나은 정책을 찾기 위해 필요한 '과감한 도약'을 PPO의 업데이트 규칙이 억제하여 지역 최적점(local optimum)에 갇힐 위험이 있다.3 이는 PPO의 안정성이 탐험 능력과 상충 관계에 있음을 보여주며, 엔트로피 보너스나 외부적인 탐험 기법(예: 레비 플라이트)의 중요성을 부각시킨다. 최근 연구에서는 LLM의 긴 연쇄 사고(Chain-of-Thought) 생성과 같은 특정 작업에서 PPO가 가치 함수 초기화 편향과 보상 신호 감쇠 문제로 인해 실패하는 구체적인 모드가 확인되기도 했다.102

### 9.3  "정렬세": RLHF 이후의 성능 저하

LLM을 PPO 기반의 RLHF로 정렬할 때 발생하는 중요한 부작용으로 "정렬세(alignment tax)"가 있다.94 이는 모델을 특정 작업(예: 지시 따르기, 유해성 감소)에 대해 인간의 선호도에 맞게 정렬시키면, 사전 훈련 단계에서 학습했던 다른 일반적인 능력(예: 학술 NLP 벤치마크 성능)이 저하되는 현상을 말한다.94

이는 정렬과 능력 간의 트레이드오프를 야기하며, 이 '세금'을 최소화하는 것이 RLHF 연구의 주요 과제 중 하나다. 이를 해결하기 위해 PPO 업데이트 중에 사전 훈련 데이터를 일부 섞어 학습시키거나(PPO-ptx), 정렬 전후 모델의 가중치를 평균 내는 모델 평균화(model averaging)와 같은 기법들이 연구되고 있다.93

## 10.  정렬의 진화: PPO를 넘어서

PPO는 LLM 정렬의 문을 열었지만, 그 복잡성과 불안정성으로 인해 더 간단하고 효율적인 대안을 찾으려는 연구가 활발히 진행되고 있다. 이 과정에서 PPO와 DPO(직접 선호 최적화)의 관계는 온라인 탐험적 방법과 오프라인 지도학습적 방법 사이의 AI 연구의 큰 흐름을 반영한다.

### 10.1  직접 선호 최적화(DPO): 더 간단하고 안정적인 대안?

직접 선호 최적화(Direct Preference Optimization, DPO)는 PPO 기반 RLHF의 강력한 대안으로 부상한 최신 LLM 정렬 기법이다.95 DPO의 핵심 혁신은 명시적인 보상 모델 훈련과 복잡한 강화학습 최적화 루프를 완전히 제거한 것이다. 대신, 선호되는 응답(yw)과 선호되지 않는 응답(yl) 쌍으로 구성된 데이터셋을 사용하여, 선호되는 응답의 로그 확률은 높이고 선호되지 않는 응답의 로그 확률은 낮추는 간단한 분류 손실 함수를 통해 정책을 직접 최적화한다.95

### 10.2  LLM 미세조정을 위한 PPO와 DPO 비교

- **복잡성:** DPO는 PPO 기반 RLHF에 비해 훨씬 간단하고 안정적이며 계산적으로 가볍다. RLHF는 정책, 가치, 보상, 참조 모델 등 4개의 모델을 동시에 관리해야 하는 반면, DPO는 단일 지도학습 파이프라인으로 통합된다.95
- **수학적 연결:** DPO의 손실 함수는 PPO가 사용하는 KL-제약 보상 최대화 목적 함수로부터 직접 유도된다. DPO는 이 목적 함수에 대한 최적 해가 닫힌 형태(closed-form)로 존재함을 보임으로써, 강화학습 과정 없이도 동일한 목표를 달성할 수 있음을 이론적으로 증명했다.98
- **성능:** DPO의 단순함과 안정성은 많은 연구자와 개발자에게 큰 매력으로 다가왔다. 그러나 최근 대규모 연구에서는 잘 튜닝된 PPO가 복잡한 추론 작업 등에서 여전히 DPO를 능가하는 성능을 보일 수 있음이 나타났다.98 이는 PPO의 온라인(on-line)적 특성, 즉 에이전트가 자신의 현재 상태에 기반해 지속적으로 탐험하고 데이터를 생성하는 과정이 초기 선호도 데이터셋에 존재하지 않는 새로운 능력을 발견하는 데 여전히 중요할 수 있음을 시사한다.
- **패러다임의 차이:** PPO 기반 RLHF는 온라인 프로세스다. 에이전트가 새로운 데이터를 생성하고, 보상 모델로부터 피드백을 받아 정책을 업데이트한다. 이는 탐험적이고 강력하지만 복잡하다.60 반면 DPO는 정적인 선호도 데이터셋으로부터 직접 학습하는 오프라인 프로세스다. 이는 본질적으로 지도학습의 한 형태로, 안정적이고 간단하지만 탐험 요소가 없다.95

### 10.3  최신 연구 동향과 정책 최적화의 미래

PPO와 DPO의 논쟁은 어느 한쪽의 완전한 승리로 끝나지 않을 것으로 보인다. 오히려 두 방법의 장점을 결합하려는 시도가 미래 연구의 방향을 제시하고 있다.

- **DPO의 확장:** 선호도의 강도를 반영하는 오프셋을 추가한 ODPO 113, 특징 레벨에서 제약을 가하는 FPO 115, 노이즈에 강건한 Dr. DPO 116 등 DPO를 개선하려는 연구가 활발하다.
- **하이브리드 접근법:** RTO(Reinforced Token Optimization), MPO(Mixed Preference Optimization)와 같은 하이브리드 방법론이 등장하고 있다. 이들은 DPO를 사용하여 안정적으로 좋은 초기 참조 모델을 만든 후, PPO의 온라인 학습 능력을 이용해 이를 더욱 미세 조정하는 방식을 취한다.117

이는 LLM 정렬의 미래가 PPO와 DPO 중 하나를 선택하는 것이 아니라, 두 패러다임의 장점을 모두 활용하는 정교한 다단계 파이프라인으로 발전할 것임을 암시한다. 오프라인 방식의 안정성과 효율성으로 기반을 다지고, 온라인 방식의 탐험적 능력으로 성능의 한계를 돌파하려는 시도가 계속될 것이다.

## 11.  결론: PPO의 유산과 미래 전망

### 11.1  PPO의 강점과 약점 종합

근접 정책 최적화(PPO)는 딥러닝 강화학습의 역사에서 중요한 이정표를 세운 알고리즘이다. 그 성공은 여러 강점과 약점의 복합적인 결과물이다.

- **강점:** PPO의 가장 큰 매력은 **구현의 단순성, 학습의 안정성 및 견고성, 그리고 다양한 문제에 대한 강력한 경험적 성능**이다.3 TRPO의 복잡한 2차 최적화를 1차 최적화로 대체하면서도 핵심적인 안정성을 유지함으로써, PPO는 강화학습의 대중화와 현대 AI의 발전에 크게 기여했다.

- **약점:** 그럼에도 불구하고 PPO는 **온-정책 방식에 따른 낮은 샘플 효율성**이라는 근본적인 한계를 가지고 있다.15 또한 

  **희소 보상 환경에서의 어려움**과 **지역 최적점에 수렴할 가능성**은 여전히 해결해야 할 과제다.39 마지막으로, 최고 성능을 달성하기 위해서는 논문에 명시되지 않은 

  **다양한 코드 레벨 최적화에 민감하게 의존**한다는 점도 실무적인 어려움이다.76

### 11.2  실무자를 위한 제언: 언제 PPO를 선택해야 하는가?

이 보고서의 분석을 종합하여, 실무자들은 다음과 같은 상황에서 PPO를 우선적으로 고려할 수 있다.

- **PPO를 선택해야 할 때:**
  - 학습 과정의 **안정성과 신뢰성**이 가장 중요할 때.
  - 시뮬레이션과 같이 **데이터 샘플링 비용이 저렴**하여 온-정책 방식의 비효율성을 감당할 수 있을 때.
  - 견고하고 널리 검증된 **강력한 베이스라인**이 필요할 때.
  - 로보틱스 제어, 게임 AI, 초기 RLHF 실험 등 PPO가 이미 SOTA(State-of-the-Art) 성능을 입증한 분야에서 작업할 때.4
- **대안(SAC, DPO 등)을 고려해야 할 때:**
  - **샘플 효율성이 최우선 과제**일 때 (예: 실제 로봇을 사용한 학습).
  - 특정 연속 제어 벤치마크에서 **최고의 점수**를 달성하는 것이 목표일 때.
  - LLM 정렬 분야에서 PPO의 복잡성 대신 **구현의 단순성과 안정성**을 더 중요하게 생각할 때.68

### 11.3  강화학습 연구의 흐름에 대한 결론적 고찰

PPO는 딥러닝 강화학습 연구의 흐름을 바꾼 중요한 알고리즘이다. PPO는 이론적 완벽성보다 실용성과 경험적 안정성에 초점을 맞춤으로써, 이전의 복잡한 방법들이 어려움을 겪었던 영역에서 실질적인 진보를 이끌어냈다.

PPO가 개척한 '안정적인 정책 업데이트를 위한 제약'이라는 원칙은 DPO와 같은 차세대 알고리즘에도 깊은 영향을 미치고 있다. 비록 필드가 새로운 패러다임으로 이동하고 있을지라도, 강력함, 안정성, 효율성, 그리고 단순함을 동시에 추구하는 탐구는 계속될 것이다. PPO는 이 기나긴 여정에서 중요한 해답 중 하나를 제시했으며, 그 유산은 앞으로도 오랫동안 강화학습 연구의 발전에 기여할 것이다.

#### **참고 자료**

1. velog.io, accessed July 1, 2025, [https://velog.io/@fragrance_0/AI-Dict-5-2.-Natural-Language-Processing-%EB%AA%A8%EB%8D%B8#:~:text=PPO(Proximal%20Policy%20Optimization)%EB%8A%94,%EB%82%98%EA%B0%80%EB%8A%94%20%ED%95%99%EC%8A%B5%EC%9D%84%20%EC%9D%98%EB%AF%B8%ED%95%9C%EB%8B%A4.](https://velog.io/@fragrance_0/AI-Dict-5-2.-Natural-Language-Processing-모델#:~:text=PPO(Proximal Policy Optimization)는,나가는 학습을 의미한다.)
2. PPO: 강화학습 알고리즘의 중요성과 장점, accessed July 1, 2025, https://www.toolify.ai/ko/ai-news-kr/ppo-1059784
3. Proximal Policy Optimization (PPO) in Reinforcement Learning - GeeksforGeeks, accessed July 1, 2025, https://www.geeksforgeeks.org/machine-learning/a-brief-introduction-to-proximal-policy-optimization/
4. Demystifying Policy Optimization in RL: An Introduction to PPO and GRPO, accessed July 1, 2025, https://towardsdatascience.com/demystifying-policy-optimization-in-rl-an-introduction-to-ppo-and-grpo/
5. Proximal policy optimization - Wikipedia, accessed July 1, 2025, https://en.wikipedia.org/wiki/Proximal_policy_optimization
6. [강화학습] PPO - 마인드스케일, accessed July 1, 2025, https://www.mindscale.kr/docs/reinforcement-learning/ppo
7. Is PPO a policy-based method or an actor-critique-based method? - AI Stack Exchange, accessed July 1, 2025, https://ai.stackexchange.com/questions/43313/is-ppo-a-policy-based-method-or-an-actor-critique-based-method
8. Understanding the Mathematics of PPO in Reinforcement Learning - Medium, accessed July 1, 2025, https://medium.com/data-science/understanding-the-mathematics-of-ppo-in-reinforcement-learning-467618b2f8d4
9. [RL] Policy Gradient Algorithms - 자신에 대한 고찰 - 티스토리, accessed July 1, 2025, https://talkingaboutme.tistory.com/entry/RL-Policy-Gradient-Algorithms
10. Proximal Policy Optimization (PPO) Explained - Towards Data Science, accessed July 1, 2025, https://towardsdatascience.com/proximal-policy-optimization-ppo-explained-abed1952457b/
11. [RL] 강화학습 알고리즘: (5) PPO - 이것저것 테크블로그, accessed July 1, 2025, [https://ai-com.tistory.com/entry/RL-%EA%B0%95%ED%99%94%ED%95%99%EC%8A%B5-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-5-PPO](https://ai-com.tistory.com/entry/RL-강화학습-알고리즘-5-PPO)
12. Proximal Policy Optimization - velog, accessed July 1, 2025, https://velog.io/@jpseo99/Proximal-Policy-Optimization
13. [PPO] Proximal Policy Optimization Algorithms - 공부하는 무니 - 티스토리, accessed July 1, 2025, https://muni-dev.tistory.com/entry/PPO-Proximal-Policy-Optimization-Algorithms
14. 강화학습 논문 리뷰: PPO(Proximal Policy Optimization Algorithms) - Team. WannaBeHappy, accessed July 1, 2025, https://jhrobotics.tistory.com/14
15. [강화학습] PPO 알고리즘 - Just Fighting - 티스토리, accessed July 1, 2025, https://yensr.tistory.com/148
16. PPO(Proximal Policy Optimization) - 독학 연구소 - 티스토리, accessed July 1, 2025, https://jvvp512.tistory.com/50
17. PPO Algorithm. Proximal Policy Optimization (PPO) is… | by DhanushKumar | Medium, accessed July 1, 2025, https://medium.com/@danushidk507/ppo-algorithm-3b33195de14a
18. RL — Proximal Policy Optimization (PPO) Explained | by Jonathan Hui | Medium, accessed July 1, 2025, https://jonathan-hui.medium.com/rl-proximal-policy-optimization-ppo-explained-77f014ec3f12
19. RL : 간단 논문 리뷰 : Proximal Policy Optimization Algorithms, accessed July 1, 2025, https://aiflower.tistory.com/m/205
20. TRPO와 PPO 알고리즘의 개념, accessed July 1, 2025, https://engineering-ladder.tistory.com/69
21. Proximal Policy Optimization — Spinning Up documentation - OpenAI, accessed July 1, 2025, https://spinningup.openai.com/en/latest/algorithms/ppo.html
22. PPO 리뷰 : Proximal policy optimization algorithms - 당황했습니까 휴먼? - 티스토리, accessed July 1, 2025, https://ropiens.tistory.com/85
23. PPO Explained - Papers With Code, accessed July 1, 2025, https://paperswithcode.com/method/ppo
24. DD-PPO Explained - Papers With Code, accessed July 1, 2025, https://paperswithcode.com/method/dd-ppo
25. [1707.06347] Proximal Policy Optimization Algorithms - arXiv, accessed July 1, 2025, https://arxiv.org/abs/1707.06347
26. Natural, Trust Region and Proximal Policy Optimization - TransferLab, accessed July 1, 2025, https://transferlab.ai/blog/trpo-and-ppo/trpo-and-ppo.pdf
27. Simple Policy Optimization - arXiv, accessed July 1, 2025, https://arxiv.org/html/2401.16025v8
28. ChengTsang/PPO-clip-and-PPO-penalty-on-Atari-Domain - GitHub, accessed July 1, 2025, https://github.com/ChengTsang/PPO-clip-and-PPO-penalty-on-Atari-Domain
29. OpenAI Spinning UP 번역] Proximal Policy Optimization - MCLearning's FrontEnd StudyRoom - 티스토리, accessed July 1, 2025, https://mclearninglab.tistory.com/145
30. Proximal Policy Optimization PPO 안정성과 성능을 개선하는 방법 - infobeste - 티스토리, accessed July 1, 2025, https://positive-impactor.tistory.com/543
31. Introducing the Clipped Surrogate Objective Function - Hugging Face Deep RL Course, accessed July 1, 2025, https://huggingface.co/learn/deep-rl-course/unit8/clipped-surrogate-objective
32. TRPO와 PPO - Simulation | ML, accessed July 1, 2025, [https://jay.tech.blog/2018/10/09/trpo%EC%99%80-ppo/](https://jay.tech.blog/2018/10/09/trpo와-ppo/)
33. A Beginner's Guide to Proximal Policy Optimisation (PPO) | by Byronchan | Medium, accessed July 1, 2025, https://medium.com/@byronchan611/proximal-policy-optimisation-ppo-a1f24de20230
34. Policy Optimization (PPO) - Python Lessons, accessed July 1, 2025, https://pylessons.com/PPO-reinforcement-learning
35. Proximal Policy Optimization (PPO) - Hugging Face, accessed July 1, 2025, https://huggingface.co/blog/deep-rl-ppo
36. Proximal Policy Optimization Algorithms(PPO) - 기록하기 - 티스토리, accessed July 1, 2025, https://lynnn.tistory.com/73
37. Proximal Policy Optimization - Toloka, accessed July 1, 2025, https://toloka.ai/blog/proximal-policy-optimization/
38. Proximal Policy Optimization (PPO) - Explained - Dilith Jayakody, accessed July 1, 2025, https://dilithjay.com/blog/ppo
39. Proximal Policy Optimization - Adversarial Attacks on Reinforcement Learning, accessed July 1, 2025, https://aarl-ieee-nitk.github.io/reinforcement-learning,/policy-gradient-methods,/sampled-learning,/optimization/theory/2020/03/25/Proximal-Policy-Optimization.html
40. joel-baptista.github.io, accessed July 1, 2025, [https://joel-baptista.github.io/phd-weekly-report/posts/ac/#:~:text=PPO%20is%20an%20Actor%2DCritic,loss%20is%20one%20of%20them.](https://joel-baptista.github.io/phd-weekly-report/posts/ac/#:~:text=PPO is an Actor-Critic,loss is one of them.)
41. Proximal Policy Optimization with PyTorch and Gymnasium - DataCamp, accessed July 1, 2025, https://www.datacamp.com/tutorial/proximal-policy-optimization
42. Proximal Policy Optimization Review - sc2-korean-level - GitBook, accessed July 1, 2025, https://chris-chris.gitbook.io/sc2-korean-level/proximal-policy-optimization-review
43. The 37 Implementation Details of Proximal Policy Optimization · The ..., accessed July 1, 2025, https://iclr-blog-track.github.io/2022/03/25/ppo-implementation-details/
44. A clean and minimal implementation of PPO (Proximal Policy Optimization) algorithm in Pytorch, for continuous action spaces. - GitHub, accessed July 1, 2025, https://github.com/adi3e08/PPO
45. Proximal Policy Optimization (PPO) [56] is an actor-critic RL algorithm that learns a policy π and a value function Vθ with th, accessed July 1, 2025, https://proceedings.neurips.cc/paper/2021/file/2b38c2df6a49b97f706ec9148ce48d86-Supplemental.pdf
46. Proximal Policy Optimization with Generalized Advantage Estimation (PPO2) - AI 지식창고, accessed July 1, 2025, https://grooms-academy.tistory.com/15
47. The 32 Implementation Details of Proximal Policy Optimization (PPO ..., accessed July 1, 2025, https://costa.sh/blog-the-32-implementation-details-of-ppo.html
48. High-Dimensional Continuous Control using Generalized Advantage Estimation, accessed July 1, 2025, https://dongminlee.tistory.com/12
49. Generalized Advantage Estimation (GAE): A Deep Dive into Bias, Variance, and Policy Gradients - Shivang Shrivastav, accessed July 1, 2025, https://shivang-ahd.medium.com/generalized-advantage-estimation-a-deep-dive-into-bias-variance-and-policy-gradients-a5e0b3454dad
50. Generalised Advantage Estimator (GAE): How does it help a policy optimisation - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/kjxrpk/generalised_advantage_estimator_gae_how_does_it/
51. 7. High-Dimensional Continuous Control using Generalized Advantage Estimation, accessed July 1, 2025, https://rlkorea.tistory.com/35
52. 8. Proximal Policy Optimization - RL Korea Blog - 티스토리, accessed July 1, 2025, https://rlkorea.tistory.com/36
53. [논문 리뷰] (GAE) HIGH-DIMENSIONAL CONTINUOUS CONTROL USING GENERALIZED ADVANTAGE ESTIMATION - velog, accessed July 1, 2025, https://velog.io/@oldboy818/Paper-Review-GAE-HIGH-DIMENSIONAL-CONTINUOUS-CONTROL-USING-GENERALIZED-ADVANTAGE-ESTIMATION
54. Generalized Advantage Estimate - Notes on AI, accessed July 1, 2025, https://notesonai.com/generalized+advantage+estimate
55. Annonated Algorithm Visualization, accessed July 1, 2025, https://opendilab.github.io/PPOxFamily/gae.html
56. Understanding the Mathematics of PPO in Reinforcement Learning - Towards Data Science, accessed July 1, 2025, https://towardsdatascience.com/understanding-the-mathematics-of-ppo-in-reinforcement-learning-467618b2f8d4/
57. ML-agents/docs/Training-PPO.md at master · gzrjzcx/ML-agents ..., accessed July 1, 2025, https://github.com/gzrjzcx/ML-agents/blob/master/docs/Training-PPO.md
58. Proximal Policy Optimization Tutorial (Part 2/2: GAE and PPO loss) | by DG AI Team | deepgamingai | Medium, accessed July 1, 2025, https://medium.com/deepgamingai/proximal-policy-optimization-tutorial-part-2-2-gae-and-ppo-loss-22337981f815
59. Master Reinforcement Learning with Proximal Policy Optimization (PPO) - Toolify.ai, accessed July 1, 2025, https://www.toolify.ai/ai-news/master-reinforcement-learning-with-proximal-policy-optimization-ppo-1059875
60. Navigating the RLHF Landscape: From Policy Gradients to PPO, GAE, and DPO for LLM Alignment - Hugging Face, accessed July 1, 2025, https://huggingface.co/blog/NormalUhr/rlhf-pipeline
61. Comparison & Selection of RL Algorithms in Continuous Action Spaces - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/8w7mn2/comparison_selection_of_rl_algorithms_in/
62. PPO: Efficient, Stable, and Scalable Policy Optimization | by Dong-Keon Kim | Medium, accessed July 1, 2025, https://medium.com/@kdk199604/ppo-efficient-stable-and-scalable-policy-optimization-15b5b9c74a88
63. How does PPO and A2C work? : r/reinforcementlearning - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/r0n6nl/how_does_ppo_and_a2c_work/
64. [Reinforcement Learning] Proximal Policy Optimization (PPO) Algorithm - velog, accessed July 1, 2025, [https://velog.io/@rockgoat2/Reinforcement-Learning-PPO-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EB%A6%AC%EB%B7%B0](https://velog.io/@rockgoat2/Reinforcement-Learning-PPO-알고리즘-리뷰)
65. all-rl-algorithms/7_ppo.ipynb at master - GitHub, accessed July 1, 2025, https://github.com/FareedKhan-dev/all-rl-algorithms/blob/master/7_ppo.ipynb
66. SAC vs. A2C: A Comparative Analysis of Reinforcement Learning Algorithms, accessed July 1, 2025, https://shivang-ahd.medium.com/sac-vs-a2c-a-comparative-analysis-of-reinforcement-learning-algorithms-a59f89de77da
67. Actor-Critic Methods: SAC and PPO | Joel's PhD Blog, accessed July 1, 2025, https://joel-baptista.github.io/phd-weekly-report/posts/ac/
68. Are there any papers or theories on why SAC is better for continuous control tasks than on-policy methods? - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/y2af2i/are_there_any_papers_or_theories_on_why_sac_is/
69. Does SAC perform better than PPO in sample-expensive tasks with discrete action spaces?, accessed July 1, 2025, https://ai.stackexchange.com/questions/36092/does-sac-perform-better-than-ppo-in-sample-expensive-tasks-with-discrete-action
70. DDPG vs PPO vs SAC: when to use? : r/reinforcementlearning - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/holioy/ddpg_vs_ppo_vs_sac_when_to_use/
71. MuJoCo Manipulus: A Robot Learning Benchmark for Generalizable Tool Manipulation, accessed July 1, 2025, https://openreview.net/forum?id=b9Ne5lHJ8Y
72. A Comparison of PPO, TD3 and SAC Reinforcement Algorithms for Quadruped Walking Gait Generation - Scientific Research Publishing, accessed July 1, 2025, https://www.scirp.org/journal/paperinformation?paperid=123401
73. PPO — Stable Baselines3 2.7.0a0 documentation - Read the Docs, accessed July 1, 2025, https://stable-baselines3.readthedocs.io/en/master/modules/ppo.html
74. Delve into PPO: Implementation Matters for Stable RLHF - OpenReview, accessed July 1, 2025, https://openreview.net/pdf?id=rxEmiOEIFL
75. implementation matters in deep policy gradients:acase study on ppo and trpo - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2005.12729
76. Implementation Matters in Deep RL: A Case Study on PPO and TRPO, accessed July 1, 2025, https://vitalab.github.io/article/2020/01/14/Implementation_Matters.html
77. Secrets of RLHF in Large Language Models Part I: PPO - GitHub Pages, accessed July 1, 2025, https://openlmlab.github.io/MOSS-RLHF/paper/SecretsOfRLHFPart1.pdf
78. PPO paper 리뷰 - simpling - 티스토리, accessed July 1, 2025, https://simpling.tistory.com/77
79. Reinforcement Learning (PPO) with TorchRL Tutorial - PyTorch documentation, accessed July 1, 2025, https://docs.pytorch.org/tutorials/intermediate/reinforcement_ppo.html
80. Simple Policy Optimization - arXiv, accessed July 1, 2025, https://arxiv.org/html/2401.16025v2
81. Understanding PPO: A Game-Changer in AI Decision-Making Explained for RL Newcomers, accessed July 1, 2025, https://medium.com/@chris.p.hughes10/understanding-ppo-a-game-changer-in-ai-decision-making-explained-for-rl-newcomers-913a0bc98d2b
82. PPO/best-practices-ppo.md at master · EmbersArc/PPO · GitHub, accessed July 1, 2025, https://github.com/EmbersArc/PPO/blob/master/best-practices-ppo.md
83. Introduction to Proximal Policy Optimization (PPO) - Radek Osmulski, accessed July 1, 2025, https://radekosmulski.com/introduction-to-proximal-policy-optimization-ppo/
84. Learning Continuous Control through Proximal Policy Optimization for Mobile Robot Navigation - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/330105099_Learning_Continuous_Control_through_Proximal_Policy_Optimization_for_Mobile_Robot_Navigation
85. Deep Reinforcement Learning with Enhanced PPO for Safe Mobile Robot Navigation - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2405.16266
86. Optimizing Autonomous Vehicle Performance Using Improved ..., accessed July 1, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11946678/
87. Dynamic Task Planning for Multi-Arm Apple-Harvesting Robots Using LSTM-PPO Reinforcement Learning Algorithm - MDPI, accessed July 1, 2025, https://www.mdpi.com/2077-0472/15/6/588
88. [1912.06680] Dota 2 with Large Scale Deep Reinforcement Learning - arXiv, accessed July 1, 2025, https://arxiv.org/abs/1912.06680
89. Dota 2 with Large Scale Deep Reinforcement Learning - OpenAI, accessed July 1, 2025, https://cdn.openai.com/dota-2.pdf
90. arXiv:1901.08004v6 [cs.LG] 21 Jun 2019, accessed July 1, 2025, https://arxiv.org/pdf/1901.08004
91. (PDF) Long-Term Planning and Situational Awareness in OpenAI Five - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/337966901_Long-Term_Planning_and_Situational_Awareness_in_OpenAI_Five
92. [Deep Research] 강화학습의 개념과 주요 기법 분석, LLM에서의 활용 및 발전 방향에 대한 보고서 - 읽을거리&정보공유 - 파이토치 한국 사용자 모임, accessed July 1, 2025, https://discuss.pytorch.kr/t/deep-research-llm/6112
93. arXiv:2203.02155v1 [cs.CL] 4 Mar 2022, accessed July 1, 2025, https://arxiv.org/pdf/2203.02155
94. Alignment/RLHF - LM Notes - Mohit Agarwal, accessed July 1, 2025, https://agmohit.com/llm-notes/docs/alignment-rlhf/
95. The Shift from RLHF to DPO for LLM Alignment: Fine-Tuning Large Language Models | by Nishtha kukreti | May, 2025 | Medium, accessed July 1, 2025, https://medium.com/@nishthakukreti.01/the-shift-from-rlhf-to-dpo-for-llm-alignment-fine-tuning-large-language-models-631f854de301
96. LLM Alignment Techniques: A Summary | by Kaige - Medium, accessed July 1, 2025, https://medium.com/@kaige.yang0110/llm-alignment-techniques-a-summary-842622621407
97. 5 PPO Variants for Enhancing RLHF Performance - ApX Machine Learning, accessed July 1, 2025, https://apxml.com/posts/ppo-variants-for-enhancing-rlhf-performance
98. Unpacking DPO and PPO: Disentangling Best Practices for Learning from Preference Feedback - OpenReview, accessed July 1, 2025, https://openreview.net/pdf?id=AlcABwHLov
99. [RL] 8. PPO: Proximal Policy Optimization - 강정노트 - 티스토리, accessed July 1, 2025, https://gangjeong22.tistory.com/196
100. [RL] 강화학습 이론: On-policy & Off-policy - 이것저것 테크블로그, accessed July 1, 2025, [https://ai-com.tistory.com/entry/RL-%EA%B0%95%ED%99%94%ED%95%99%EC%8A%B5-%EC%9D%B4%EB%A1%A0-On-policy-Off-policy](https://ai-com.tistory.com/entry/RL-강화학습-이론-On-policy-Off-policy)
101. On-Policy, Off-Policy, Online, Offline 강화학습 - 그냥 적기 - 티스토리, accessed July 1, 2025, [https://seungwooham.tistory.com/entry/On-Policy-Off-Policy-Online-Offline-%EA%B0%95%ED%99%94%ED%95%99%EC%8A%B5](https://seungwooham.tistory.com/entry/On-Policy-Off-Policy-Online-Offline-강화학습)
102. What's Behind PPO's Collapse in Long-CoT? Value Optimization Holds the Secret - arXiv, accessed July 1, 2025, https://arxiv.org/html/2503.01491v1
103. Explain why PPO fails at this very simple task : r/reinforcementlearning - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/n09ns2/explain_why_ppo_fails_at_this_very_simple_task/
104. Mitigating the Alignment Tax of RLHF - arXiv, accessed July 1, 2025, https://arxiv.org/html/2309.06256v3
105. Mitigating the Alignment Tax of RLHF - ACL Anthology, accessed July 1, 2025, https://aclanthology.org/2024.emnlp-main.35.pdf
106. avalonstrel/Mitigating-the-Alignment-Tax-of-RLHF - GitHub, accessed July 1, 2025, https://github.com/avalonstrel/Mitigating-the-Alignment-Tax-of-RLHF
107. Direct Preference Optimization: Your Language Model is Secretly a ..., accessed July 1, 2025, https://arxiv.org/abs/2305.18290
108. Direct Preference Optimization(DPO), accessed July 1, 2025, https://www.cs.toronto.edu/~cmaddis/courses/csc2541_w25/presentations/mu_cao_dpo.pdf
109. Arxiv Dives - Direct Preference Optimization (DPO) - Oxen.ai, accessed July 1, 2025, https://www.oxen.ai/blog/arxiv-dives-direct-preference-optimization-dpo
110. [D] Question about Direct Preference Optimization (DPO) equation : r/MachineLearning, accessed July 1, 2025, https://www.reddit.com/r/MachineLearning/comments/197yl66/d_question_about_direct_preference_optimization/
111. Is DPO Superior to PPO for LLM Alignment? A Comprehensive Study - arXiv, accessed July 1, 2025, https://arxiv.org/html/2404.10719v1
112. Unpacking DPO and PPO: Disentangling Best Practices for Learning from Preference Feedback - arXiv, accessed July 1, 2025, https://arxiv.org/html/2406.09279v1
113. [2402.10571] Direct Preference Optimization with an Offset - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2402.10571
114. Direct Preference Optimization with an Offset - arXiv, accessed July 1, 2025, https://arxiv.org/html/2402.10571v2
115. Direct Preference Optimization Using Sparse Feature-Level Constraints - arXiv, accessed July 1, 2025, https://arxiv.org/html/2411.07618v1
116. towards robust alignment of language models: distributionally robustifying direct preference optimization - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2407.07880
117. DPO Meets PPO: Reinforced Token Optimization for RLHF - arXiv, accessed July 1, 2025, https://arxiv.org/html/2404.18922v4
118. Mixed Preference Optimization: Reinforcement Learning with Data Selection and Better Reference Model - arXiv, accessed July 1, 2025, https://arxiv.org/html/2403.19443v1
119. DPO Meets PPO: Reinforced Token Optimization for RLHF - arXiv, accessed July 1, 2025, https://arxiv.org/html/2404.18922v1

Proximal Policy Optimization: all about the algorithm created by OpenAI - DataScientest, accessed July 1, 2025, https://datascientest.com/en/proximal-policy-optimization-all-about-the-algorithm-created-by-openai




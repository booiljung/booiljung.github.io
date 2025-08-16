# Soft Actor-Critic (SAC) 종합 안내서

## 1. I. Soft Actor-Critic 소개: 엔트로피 정규화 제어의 패러다임

### 1.1  강화학습 지형도 속 SAC의 위치

Soft Actor-Critic(SAC)은 현대 강화학습(Reinforcement Learning, RL) 분야에서 가장 영향력 있고 성공적인 알고리즘 중 하나로, 복잡한 연속 제어 문제 해결에 새로운 기준을 제시했습니다. SAC를 정확히 이해하기 위해서는 먼저 강화학습의 광범위한 지형도 속에서 그 위치를 파악해야 합니다. SAC는 **모델-프리(model-free)**, **오프-폴리시(off-policy)**, **액터-크리틱(actor-critic)**이라는 세 가지 핵심적인 특성으로 분류됩니다.1

- **모델-프리 (Model-Free)**: SAC는 환경의 동역학(상태 전이 확률 및 보상 함수)에 대한 명시적인 모델을 구축하지 않습니다.3 대신, 에이전트는 환경과의 직접적인 상호작용, 즉 수많은 시행착오를 통해 학습합니다. 이러한 접근 방식은 환경의 물리 법칙이 알려지지 않았거나 모델링하기에 지나치게 복잡한 현실 세계의 문제에 알고리즘을 폭넓게 적용할 수 있게 합니다.
- **오프-폴리시 (Off-Policy)**: SAC는 현재 학습 중인 정책(policy)이 아닌, 과거의 다른 정책(주로 더 탐험적인 행동 정책)이 수집한 데이터를 사용해 최적 정책을 학습할 수 있습니다. 이는 **경험 리플레이 버퍼(experience replay buffer)**라는 메커니즘을 통해 가능해지는데, 이 버퍼는 과거의 경험 데이터 `(상태, 행동, 보상, 다음 상태)`를 저장해두고 학습 시에 반복적으로 재사용합니다.1 데이터 수집과 학습 과정의 이러한 분리는 PPO와 같은 온-폴리시(on-policy) 방법에 비해 SAC가 월등히 높은 **샘플 효율성(sample efficiency)**을 갖게 하는 핵심적인 요인입니다.1
- **액터-크리틱 (Actor-Critic)**: SAC는 역할이 다른 두 개의 신경망(또는 신경망 구성요소)을 사용합니다. **액터(actor)**는 정책 `$\pi(a|s)$`를 나타내며 주어진 상태에서 어떤 행동을 할지 결정하는 역할을 합니다. 반면, **크리틱(critic)**은 그 행동의 가치를 평가하기 위해 가치 함수(soft Q-function)를 추정합니다.1

SAC는 이러한 구조적 특징을 바탕으로 설계되었으며, 특히 로봇 공학이나 자율 제어와 같이 **연속적인 행동 공간(continuous action space)**을 가진 환경에서 탁월한 성능을 발휘하도록 만들어졌습니다.7 물론, 알고리즘을 일부 수정하여 이산(discrete) 또는 혼합(hybrid) 행동 공간에도 적용할 수 있습니다.2

### 1.2  이론적 초석: 최대 엔트로피 강화학습 프레임워크

SAC의 가장 근본적인 혁신은 전통적인 강화학습의 목표 함수를 재정의하는 것에서 시작됩니다. 기존의 강화학습 알고리즘 대부분은 누적 할인 보상(cumulative discounted reward)의 기댓값을 최대화하는 것을 유일한 목표로 삼습니다. 그러나 SAC는 **최대 엔트로피(Maximum Entropy, MaxEnt) 강화학습 프레임워크**에 기반하여 작동합니다.15

이 프레임워크는 최적화 목표를 근본적으로 변경합니다. 에이전트는 단순히 보상을 최대로 받는 것뿐만 아니라, 자신의 정책이 갖는 **엔트로피(entropy)**, 즉 행동의 무작위성을 동시에 최대화하려고 시도합니다. 이는 "온도(temperature)" 파라미터 `$\alpha$`에 의해 가중치가 조절되는 보상과 엔트로피 간의 균형을 맞추는 문제입니다.1 SAC의 목표 함수는 다음과 같이 공식화됩니다:

J(π)=t=0∑TE(st,at)∼ρπ[r(st,at)+αH(π(⋅∣st))]

여기서 `$\mathcal{H}(\pi(\cdot|s_t)) = \mathbb{E}_{a_t \sim \pi} [-\log \pi(a_t|s_t)]`는 상태 `s_t`에서 정책 `$\pi$`가 갖는 엔트로피를 의미합니다.12 알고리즘 이름에 포함된 "Soft"라는 단어는 바로 이 엔트로피 정규화(entropy regularization)를 가리킵니다. 이는 강하고 결정론적인(deterministic) 정책 대신, 더 유연하고 확률적인(stochastic) "부드러운(soft)" 정책을 장려하기 때문입니다.22 이는 최대 엔트로피 프레임워크에서 파생된 soft policy iteration 절차의 직접적인 결과이기도 합니다.16

이러한 목표의 재정의는 단순한 탐험 기법의 추가를 넘어섭니다. 전통적인 강화학습의 목표가 `$\arg\max \mathbb{E}[\sum r_t]$`이고 그 해답이 종종 결정론적 정책인 반면, SAC의 목표는 `$\arg\max \mathbb{E}[\sum (r_t + \alpha\mathcal{H}_t)]`입니다.13 이는 약간의 보상을 희생하더라도 훨씬 더 무작위적인 행동을 하는 정책이, 절대적으로 최대의 보상을 얻는 결정론적 정책보다 이 목표 함수 하에서는 "더 나은" 정책으로 평가될 수 있음을 의미합니다. 결과적으로, 학습된 정책은 훈련 환경의 특정 동역학에 과적합될 가능성이 낮아집니다. 무작위성을 유지하도록 인센티브를 받음으로써, 정책은 자연스럽게 견고함의 여지를 확보하게 됩니다. 이처럼 최적성에 대한 재정의는 SAC가 현실 세계에 특히 더 적합하도록 만듭니다. 실제 시스템은 모델링되지 않은 동역학, 센서 노이즈, 그리고 작업 실행에서의 미세한 변동성을 포함합니다. 시뮬레이터에서 완벽하게 최적화된 결정론적 정책은 이러한 작은 교란에도 쉽게 실패할 수 있습니다. 반면, SAC의 엔트로피 기반 확률적 정책은 본질적으로 이러한 "현실과의 격차(reality gap)" 문제에 더 강하며, 이는 로봇 공학의 시뮬레이션-현실 전이(sim-to-real transfer)에서 SAC가 놀라운 성공을 거둔 이유를 설명합니다.25

### 1.3  핵심 장점: 샘플 효율성, 안정성, 그리고 원칙적 탐험의 통합

최대 엔트로피 프레임워크와 오프-폴리시 액터-크리틱 구조의 결합은 초기 딥러닝 기반 강화학습 방법들이 겪었던 높은 샘플 복잡도와 불안정한 수렴이라는 두 가지 핵심적인 문제를 동시에 해결하는 알고리즘을 탄생시켰습니다.11

- **향상된 탐험 (Enhanced Exploration)**: 목표 함수에 포함된 엔트로피 보너스는 탐험에 대한 원칙적이고 내재적인 동기를 부여합니다. 에이전트는 "과제를 성공적으로 수행하면서 동시에 가능한 한 무작위적으로 행동"하도록 보상을 받습니다.1 이는 에이전트가 최적이 아닌 국소 최적점(local optima)에 조기에 수렴하는 것을 방지하고, 다양하고 잠재적으로 더 나은 행동 전략을 발견하도록 장려합니다.1
- **개선된 안정성과 견고함 (Improved Stability and Robustness)**: 정책의 확률적 특성과 엔트로피 최대화는 더 안정적인 학습 동역학으로 이어지며, 환경 노이즈나 가치 함수 추정 오류에 더 강인한 정책을 만들어냅니다.7 이는 SAC가 실제 로봇 공학 분야에서 성공을 거둔 핵심적인 이유 중 하나입니다.25
- **최고 수준의 성능 (State-of-the-Art Performance)**: 이러한 장점들 덕분에 SAC는 광범위한 연속 제어 벤치마크에서 지속적으로 최고 수준의 성능을 입증했습니다. 샘플 효율성과 최종 성능 측면 모두에서 PPO와 같은 이전의 온-폴리시 방법과 DDPG와 같은 오프-폴리시 방법을 능가하는 결과를 보여주었습니다.7

## 2. II. SAC 에이전트의 아키텍처 청사진

이 섹션에서는 SAC 에이전트를 구성하는 기능적 요소들을 해부하고, 각 신경망과 메커니즘의 역할 및 설계를 설명합니다. SAC의 아키텍처는 이전 알고리즘들의 성공적인 요소들을 계승하고 발전시킨, 정교한 종합체라 할 수 있습니다.

### 2.1  확률적 액터 (정책 신경망)

액터는 SAC의 의사 결정 주체입니다. 파라미터 `$\theta$`로 구성된 이 신경망은 상태 `s`를 입력받아 행동 공간에 대한 확률 분포 `$\pi_\theta(a|s)$`의 파라미터를 출력합니다.1 이 정책은 본질적으로 확률적(stochastic)입니다.

- **행동 생성 (연속 공간)**: 연속 제어 문제에서 액터 신경망은 일반적으로 가우시안 분포(Gaussian distribution)의 평균(mean)과 표준편차(standard deviation)를 출력합니다.6 실제 행동은 이 분포로부터 샘플링하여 결정됩니다. 행동의 범위를 -1과 1 사이와 같이 특정 구간으로 제한해야 할 경우, 샘플링된 행동에 

  **스쿼싱 함수(squashing function)**, 주로 하이퍼볼릭 탄젠트(`tanh`) 함수를 적용합니다.6 이 스쿼싱 과정은 SAC의 정책 분포를 PPO와 같은 다른 알고리즘에서 사용되는 표준 가우시안 분포와 구별하는 핵심적인 특징입니다.12

- **행동 생성 (이산 및 혼합 공간)**: 이산 행동 공간의 경우, 액터는 각 행동에 대한 확률을 나타내는 범주형 분포(categorical distribution)를 출력합니다.6 연속과 이산 요소가 모두 포함된 혼합 행동 공간에서는 가우시안 분포와 범주형 분포를 모두 생성합니다.6

- **상태 의존적 분산 (State-Dependent Variance)**: SAC의 성능에 결정적인 영향을 미치는 중요한 구현 세부 사항 중 하나는 가우시안 정책의 표준편차(정확히는 로그 표준편차, `log_std`)가 신경망의 출력값이라는 점입니다. 이는 표준편차가 상태에 따라 동적으로 변할 수 있음을 의미합니다. 이는 종종 상태와 무관한 파라미터 벡터로 표준편차를 사용하는 PPO와 같은 알고리즘과 대조되는 지점입니다.12 이 상태 의존성은 에이전트가 상태 공간의 다른 영역에서 탐험의 정도를 학습할 수 있게 합니다. 예를 들어, 안전하고 잘 알려진 영역에서는 정책의 분산을 줄여(더 결정론적으로 만들어) 지식을 활용하고, 불확실하거나 새로운 영역에서는 분산을 높여 신중하게 탐험할 수 있습니다. 이러한 학습된 적응형 탐험은 DDPG/TD3에서 사용되는 균일하고 상태와 무관한 노이즈보다 훨씬 지능적인 방식이며, SAC의 성능과 안정성에 크게 기여합니다.

### 2.2  크리틱 앙상블: Soft Q-가치 추정과 트윈 크리틱 아키텍처

크리틱은 액터의 행동을 평가하는 역할을 합니다. 파라미터 `$\phi$`로 구성된 크리틱 신경망은 **soft Q-가치(soft Q-value)**, `Q_φ(s, a)`를 추정합니다. 이는 표준 Q-가치와는 다릅니다. Soft Q-가치는 미래의 기대 보상뿐만 아니라, 미래의 기대 엔트로피 보너스까지 포함하는 개념입니다.8

- **트윈 크리틱 아키텍처 (Twin-Critic Architecture)**: SAC는 Q-가치 과대평가 편향(Q-value overestimation bias) 문제에 대응하기 위해 TD3 알고리즘에서 제안된 "클립된 이중 Q-학습(clipped double-Q trick)" 기법을 채택합니다.8 이를 위해 SAC는 

  **두 개의 독립적인 Q-함수**, `Q_φ1`과 `Q_φ2`를 학습합니다.1 벨만 업데이트를 위한 목표 가치를 계산할 때, 두 크리틱 예측값 중 **더 작은 값(minimum)**을 사용합니다. 이는 비관적이지만 더 안정적인 학습 목표를 제공하여 학습 과정을 안정화시킵니다.12

- **가치 함수 신경망 (선택적이지만 일반적)**: 초기 버전의 SAC는 별도의 상태-가치 함수 신경망 `V_ψ(s)`를 포함하기도 했습니다.16 후기 버전에서는 종종 Q-함수와 정책으로부터 가치 함수를 암묵적으로 계산하지만, 명시적인 가치 함수 신경망은 안정적인 베이스라인(baseline)을 제공하여 학습을 더욱 안정화시키는 데 도움이 될 수 있습니다.28

SAC의 아키텍처는 선대 알고리즘인 DDPG와 TD3의 가장 효과적인 구성 요소들을 종합하고 정제한 결과물입니다. DDPG는 오프-폴리시 액터-크리틱, 리플레이 버퍼, 타겟 네트워크라는 연속 제어의 기본 템플릿을 확립했습니다.3 TD3는 DDPG의 치명적인 약점인 Q-가치 과대평가를 트윈 크리틱 아키텍처와 지연된 정책 업데이트로 해결했습니다.36 SAC는 리플레이 버퍼, 타겟 네트워크, 그리고 결정적으로 트윈 크리틱 아키텍처를 이 계보로부터 직접 물려받았습니다.1 SAC의 주된 아키텍처적 차별점은 DDPG/TD3의 결정론적 액터를 완전한 확률적 액터로 대체한 것입니다.6 이는 정책 엔트로피를 의미 있고 최적화 가능한 양으로 만들기 위해 필수적인 변화였습니다. 이러한 종합은 강력한 시너지를 창출합니다. TD3의 안정성 메커니즘(트윈 크리틱, 타겟 네트워크)이 견고한 기반을 제공하는 동안, 확률적 액터와 최대 엔트로피 목표는 DDPG/TD3의 임시방편적인 노이즈 주입보다 더 원칙적이고 강인한 탐험 전략을 제공합니다.

### 2.3  안정성을 위한 핵심 메커니즘: 타겟 네트워크와 경험 리플레이 버퍼

- **경험 리플레이 버퍼 (Experience Replay Buffer)**: 오프-폴리시 알고리즘으로서 SAC는 모든 경험 `(s, a, r, s', d)`를 큰 리플레이 버퍼 `D`에 저장합니다.1 학습 과정에서 신경망을 업데이트하기 위해 이 버퍼로부터 미니배치(mini-batch) 단위로 경험을 무작위로 샘플링합니다. 이는 데이터의 시간적 상관관계를 깨뜨리고 높은 샘플 효율성을 가능하게 합니다.1
- **타겟 네트워크 (Target Networks)**: 크리틱 학습의 안정성을 위해 SAC는 **타겟 네트워크**를 사용합니다. 이는 주 크리틱 신경망을 시간적으로 지연시켜 복사한 것으로, 파라미터 `$\phi_{targ}$`를 가집니다. 타겟 네트워크는 벨만 오차 계산을 위한 안정적이고 느리게 움직이는 목표를 제공하여, 발산으로 이어질 수 있는 "움직이는 목표물을 쫓는(chasing a moving target)" 문제를 방지합니다.6
- **타겟 네트워크 업데이트**: 타겟 네트워크 파라미터는 주기적으로 업데이트됩니다. 가장 일반적인 방법은 "소프트" 업데이트, 또는 폴리악 평균(polyak averaging)이라 불리는 방식입니다. 이 방식에서는 타겟 파라미터가 주 신경망 파라미터를 향해 점진적으로 이동합니다: `$\phi_{targ} \leftarrow \tau \phi + (1 - \tau) \phi_{targ}$`. 여기서 `$\tau$`는 0.005와 같이 매우 작은 상수입니다.6

## 3. III. 수학적 엔진: SAC 학습 알고리즘의 해부

이 섹션에서는 SAC 업데이트 규칙의 엄밀한 수학적 분석을 제공하며, 각 신경망의 손실 함수와 엔트로피 온도의 자동 튜닝 메커니즘을 상세히 설명합니다.

### 3.1  Soft Policy Iteration 프레임워크

실용적인 SAC 알고리즘은 **soft policy iteration**이라는 이론적 절차의 근사치입니다.16 이 절차는 두 단계를 번갈아 수행합니다:

1. **Soft Policy Evaluation**: 주어진 정책 `$\pi$`에 대해, soft Bellman 연산자를 수렴할 때까지 반복적으로 적용하여 해당하는 soft Q-함수 `Q^π`를 계산합니다.
2. **Soft Policy Improvement**: 새로운 Q-함수의 지수 함수에 비례하도록 정책을 업데이트합니다: `$\pi_{new}(\cdot|s) \propto \exp(1/\alpha \cdot Q^\pi(s, \cdot))$`. 이 업데이트는 최대 엔트로피 목표 하에서 더 나은 정책을 산출함이 증명되었습니다.18

딥러닝 기반 강화학습에서는 이 단계들을 수렴할 때까지 실행하는 것이 비현실적입니다. 대신 SAC는 신경망을 사용하여 함수를 근사하고, 각 구성 요소에 대해 경사 하강법(gradient descent)을 몇 단계씩 수행하는 공동 최적화 과정을 사용합니다.11

### 3.2  목표 함수와 손실 공식

#### 3.2.1  크리틱 업데이트: Soft Bellman 방정식과 잔차 최소화

Soft Q-함수들은 **soft Bellman 방정식**을 만족하도록 학습됩니다. 이 방정식은 *다음* 상태 정책의 엔트로피 항을 포함한다는 점에서 표준 벨만 방정식과 다릅니다.1 주어진 경험 데이터 

`(s, a, r, s')`에 대한 목표 가치 `y`는 다음과 같이 계산됩니다:

y=r+γ(1−d)Ea′∼π(⋅∣s′)[i=1,2minQϕtarg,i(s′,a′)−αlogπθ(a′∣s′)]

여기서 `d`는 종료 상태 플래그(terminal flag)이고, `Q_{φ_{targ}}`는 타겟 크리틱 네트워크이며, 기댓값은 다음 상태 `s'`에 대한 현재 정책 `$\pi_\theta$`로부터 샘플링된 행동 `a'`에 대해 계산됩니다.16

두 크리틱 신경망(`Q_φ1`, `Q_φ2`)은 자신들의 예측값과 목표 가치 `y` 사이의 평균 제곱 벨만 오차(Mean Squared Bellman Error, MSBE)를 최소화함으로써 업데이트됩니다. 각 크리틱 `i`에 대한 손실 함수는 다음과 같습니다:

LQ(ϕi)=E(s,a,r,s′,d)∼D[21(Qϕi(s,a)−y)2]

계산된 그래디언트는 크리틱 파라미터 `$\phi_1$`과 `$\phi_2$`를 업데이트하는 데 사용됩니다.16

#### 3.2.2  액터 업데이트: 정책 개선과 재파라미터화 트릭

액터(정책)는 자신의 행동에 대한 기대 soft Q-가치와 자신의 출력 엔트로피를 최대화하도록 업데이트됩니다. 이는 정책이 soft Q-함수에 의해 정의된 볼츠만 분포(Boltzmann distribution)와의 KL-발산(KL-divergence)을 최소화하는 것과 동일합니다.18 이는 실용적으로 다음과 같은 손실 함수로 변환됩니다:

Lπ(θ)=Es∼D,ϵ∼N[αlogπθ(fθ(ϵ;s)∣s)−i=1,2minQϕi(s,fθ(ϵ;s))]

여기서 행동은 **재파라미터화 트릭(reparameterization trick)**을 사용하여 샘플링됩니다.

그래디언트가 크리틱의 출력에서 정책 신경망으로 역전파될 수 있도록, 행동 샘플링 과정이 재구성됩니다. `$a \sim \pi_\theta(\cdot|s)$`와 같이 직접 샘플링하는 대신, 표준 정규분포 `$\mathcal{N}(0,I)$`와 같은 간단한 분포에서 노이즈 벡터 `$\epsilon$`을 샘플링한 후, 행동을 결정론적 함수 `$a = f_\theta(\epsilon; s)$`로 계산합니다. 가우시안 정책의 경우, 이는 `$a = \tanh(\mu_\theta(s) + \sigma_\theta(s) \cdot \epsilon)$`와 같이 표현됩니다.12 이 트릭은 기댓값의 확률성을 신경망 파라미터와 무관하게 만들어, 분산이 낮은 그래디언트 추정을 가능하게 합니다.

#### 3.2.3  트레이드오프 자동화: 엔트로피 온도(α)를 위한 이중 경사 하강법 업데이트

고정된 `$\alpha$` 값은 특정 작업에 의존적이고 불안정하다는 문제가 있습니다.16 가장 널리 사용되는 SAC 변종의 핵심 혁신은 이 

`$\alpha$` 값의 튜닝을 자동화하는 것입니다.

이 문제는 제약 최적화 문제로 재구성됩니다: 정책의 평균 엔트로피가 목표 엔트로피 `$\mathcal{H}_{target}$` 이상이어야 한다는 제약 조건 하에서 기대 보상을 최대화하는 것입니다.

πmaxE[∑rt]s.t.E(s,a)∼ρπ[−log(π(a∣s))]≥Htarget∀t

.40

온도 `$\alpha$`는 이 엔트로피 제약에 대한 라그랑주 승수(Lagrange multiplier)가 됩니다. `$\alpha$`는 제약 최적화 문제의 이중 문제(dual problem)로부터 파생된 자체적인 목표 함수를 경사 하강법으로 최소화함으로써 학습됩니다:

L(α)=Eat∼πt[−αlogπt(at∣st)−αHtarget]

수치적 안정을 위해 일반적으로 `$\log \alpha$`에 대해 최적화됩니다.38 이 그래디언트 업데이트는 현재 정책의 엔트로피와 목표 엔트로피 간의 차이에 기반하여 

`$\alpha$`를 효과적으로 조절합니다. 목표 엔트로피 `$\mathcal{H}_{target}$`는 새로운 하이퍼파라미터이지만, 설정하기가 훨씬 직관적입니다. 일반적인 휴리스틱은 `$\mathcal{H}_{target} = -\text{dim}(\mathcal{A})$`로 설정하는 것입니다. 여기서 `$\text{dim}(\mathcal{A})$`는 행동 공간의 차원입니다.44

이 자동 `$\alpha$` 튜닝 메커니즘은 일종의 메타-학습(meta-learning)으로 볼 수 있습니다. 알고리즘은 단지 정책("무엇을")을 배우는 것이 아니라, 자신의 학습 목표(탐험 대 활용)의 균형을 동적으로 조절함으로써 "어떻게" 학습할 것인지도 배우고 있습니다. 이러한 자기 조절 속성은 SAC가 가진 안정성과 "별다른 설정 없이(off-the-shelf)" 바로 사용할 수 있는 유용성에 가장 중요한 요인일 것입니다. 이는 사용자의 손에서 가장 민감하고 비직관적인 하이퍼파라미터를 제거하여, 알고리즘을 훨씬 덜 "깨지기 쉽게(brittle)" 만들고 11, 힘든 수동 튜닝 없이도 광범위한 작업에서 잘 작동할 가능성을 높여줍니다. 이는 진정으로 자율적인 학습 에이전트를 만드는 데 있어 거대한 진일보입니다.

### 3.3  전체 학습 업데이트 주기 단계별 안내

1. **샘플링**: 리플레이 버퍼 `D`에서 `N`개의 경험 데이터 `(s, a, r, s', d)`로 구성된 미니배치를 샘플링합니다.
2. **크리틱 목표 계산**: 배치의 각 경험 데이터에 대해 다음을 수행합니다.
   - 현재 정책에서 다음 행동 `a'`를 샘플링합니다: `$a' \sim \pi_\theta(\cdot|s')$`.
   - 두 타겟 크리틱의 최솟값과 엔트로피 항을 사용하여 목표 Q-가치 `y`를 계산합니다: `$y = r + \gamma(1-d)(\min_{i=1,2} Q_{\phi_{targ,i}}(s', a') - \alpha \log \pi_\theta(a'|s'))$`.
3. **크리틱 업데이트**: MSBE 손실 `$L_Q(\phi_i) = (1/N) \sum (Q_{\phi_i}(s, a) - y)^2$`에 대해 경사 하강법을 한 단계 수행하여 두 크리틱 신경망을 업데이트합니다.
4. **액터 업데이트**: 정책 목표 `$L_\pi(\theta) = (1/N) \sum (\alpha \log \pi_\theta(a|s) - \min_{i=1,2} Q_{\phi_i}(s, a))$`에 대해 경사 상승법(또는 음수 손실에 대한 경사 하강법)을 한 단계 수행하여 액터 신경망을 업데이트합니다. 여기서 행동 `a`는 배치의 상태 `s`에 대해 정책으로부터 다시 샘플링됩니다.
5. **알파 업데이트 (자동 튜닝 시)**: `$\alpha$`의 손실 함수 `$L(\alpha) = (1/N) \sum (-\alpha (\log \pi_\theta(a|s) + \mathcal{H}_{target}))$`에 대해 경사 하강법을 한 단계 수행하여 `$\alpha$`를 업데이트합니다.
6. **타겟 네트워크 업데이트**: 타겟 크리틱 신경망에 대해 소프트 업데이트를 수행합니다: `$\phi_{targ} \leftarrow \tau \phi + (1 - \tau) \phi_{targ}$`.

## 4. IV. 성능 분석 및 비교 벤치마킹

이 섹션에서는 SAC를 주요 경쟁 알고리즘과 정량적, 정성적으로 비교하고, 경험적 결과와 이론적 차이점을 바탕으로 알고리즘 선택에 대한 명확한 지침을 제공합니다.

### 4.1  샘플 효율성 및 학습 안정성 심층 분석

- **샘플 효율성**: SAC는 리플레이 버퍼의 데이터를 재사용하는 오프-폴리시 특성 덕분에 PPO와 같은 온-폴리시 알고리즘보다 훨씬 더 높은 샘플 효율성을 보입니다.1 이는 데이터 수집 비용이 비싸거나 시간이 오래 걸리는 실제 시나리오(예: 로봇 공학)에서 결정적인 이점입니다.1
- **학습 안정성**: DDPG와 같은 초기 오프-폴리시 방법에 비해 SAC는 훨씬 더 안정적입니다.28 이 안정성은 다음 요소들의 조합에서 비롯됩니다:
  1. Q-가치 과대평가를 완화하는 트윈 크리틱 아키텍처.8
  2. 정책이 조기에 결정론적으로 변해 좋지 않은 국소 최적점에 빠지는 것을 방지하는 엔트로피 정규화.8
  3. 학습 내내 건전한 탐험-활용 균형을 유지하는 `$\alpha$`의 자동 튜닝.41

SAC와 PPO 간의 논쟁은 어느 것이 절대적으로 "더 나은가"에 대한 것이 아니라, 주어진 자원 제약 하에서 어느 것이 더 나은가에 대한 것입니다. 여기서 중요한 자원은 항상 계산 능력이 아니라, 종종 데이터 획득 비용입니다. PPO는 온-폴리시, SAC는 오프-폴리시라는 근본적인 차이가 이들의 특성을 결정합니다.9 이는 PPO의 안정성과 구현 단순성 대 SAC의 샘플 효율성이라는 주요 트레이드오프로 이어집니다.10 만약 "비용"이 실제 시간(wall-clock time)으로 측정되고 환경이 빠른 시뮬레이터라면, PPO가 수백만 개의 샘플을 신속하게 생성할 수 있기 때문에 승리할 수 있습니다.45 그러나 "비용"이 실제 상호작용(예: 로봇이 한 시간 동안 작동)으로 측정된다면, SAC가 각 귀중한 샘플로부터 더 많은 것을 배울 수 있기 때문에 승리합니다.10 이러한 자원 기반 관점은 강화학습 연구 및 적용의 방향을 결정합니다. 게임이나 시뮬레이션 물리와 같은 분야에서는 온-폴리시 방법이 여전히 경쟁력이 있지만, 물리적 세계에서의 강화학습(로봇 공학, 자율 주행차, 산업 제어)이라는 원대한 비전을 위해서는 SAC와 같은 오프-폴리시, 샘플 효율적인 방법이 선택이 아닌 필수입니다. 이것이 SAC와 그 변종들이 로봇 공학 연구 문헌에서 지배적인 알고리즘인 이유를 설명합니다.25

### 4.2  비교 분석: SAC 대 DDPG 및 TD3

DDPG → TD3 → SAC는 연속 제어를 위한 알고리즘 진화의 명확한 계보를 나타냅니다.

| 기능                | Proximal Policy Optimization (PPO)           | Deep Deterministic Policy Gradient (DDPG)                  | Twin Delayed DDPG (TD3)                                      | Soft Actor-Critic (SAC)                                      |
| ------------------- | -------------------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **학습 패러다임**   | 온-폴리시                                    | 오프-폴리시                                                | 오프-폴리시                                                  | 오프-폴리시                                                  |
| **정책 유형**       | 확률적                                       | 결정론적                                                   | 결정론적                                                     | 확률적                                                       |
| **샘플 효율성**     | 낮음                                         | 높음                                                       | 높음                                                         | 매우 높음                                                    |
| **안정성/견고함**   | 높음                                         | 낮음                                                       | 중간                                                         | 높음                                                         |
| **탐험 방식**       | 엔트로피 정규화 (보너스)                     | 행동 공간 노이즈 (예: OU-Noise)                            | 타겟 행동 노이즈                                             | 엔트로피 최대화 (목표 함수)                                  |
| **Q-가치 과대평가** | 해당 없음 (가치 함수, Q-학습 아님)           | 심각한 문제                                                | 완화됨                                                       | 완화됨                                                       |
| **핵심 혁신**       | 클립된 대리 목적 함수, 롤아웃 당 다중 에포크 | 연속 제어를 위한 액터-크리틱, 리플레이 버퍼, 타겟 네트워크 | **트윈 크리틱**, **지연된 정책 업데이트**, **타겟 정책 평활화** | **최대 엔트로피 RL 목표**, **Soft Bellman 방정식**, **엔트로피 온도 자동 튜닝** |

- **DDPG**: 단일 크리틱과 결정론적 정책 업데이트로 인해 심각한 Q-가치 과대평가 편향과 불안정성을 겪습니다.36 상관관계 없는 노이즈를 통한 탐험은 종종 비효율적입니다.47
- **TD3**: DDPG의 결함을 세 가지 핵심 혁신으로 직접 해결합니다: **트윈 크리틱**, **지연된 정책 업데이트**, 그리고 **타겟 정책 평활화**(타겟 행동에 노이즈 추가).36 이로 인해 DDPG보다 훨씬 안정적이고 성능이 우수합니다.
- **SAC 대 TD3**: 이는 최첨단 오프-폴리시 제어에서 가장 의미 있는 비교입니다.
  - **유사점**: 두 알고리즘 모두 과대평가를 제어하기 위해 트윈 크리틱 아키텍처를 사용합니다.37
  - **핵심 차이점 (정책 & 탐험)**: TD3는 **결정론적** 정책을 사용하고 주입된 노이즈를 통해 탐험하는 반면, SAC는 **확률적** 정책을 사용하고 엔트로피 최대화를 통해 탐험합니다.21
  - **SAC의 장점**: SAC의 엔트로피 기반 탐험은 일반적으로 더 원칙적이고 효과적인 것으로 간주됩니다. 이를 통해 정책은 상태에 따라 탐험 정도를 조절하는 법을 배울 수 있으며, TD3의 결정론적 정책으로는 불가능한 다중 모드 행동 분포(multi-modal action distribution)를 표현할 수 있습니다.15 이는 특히 복잡한 작업에서 더 나은 최종 성능과 견고함으로 이어지는 경우가 많습니다.16
  - **TD3의 장점**: 일부에서는 TD3의 노이즈 주입 메커니즘이 엔트로피 목표를 설정하는 것보다 더 직관적일 수 있어 튜닝이 더 간단할 수 있다고 주장합니다.47 그러나 자동 알파 튜닝 기능으로 인해 TD3의 이러한 장점은 대부분 상쇄됩니다.

### 4.3  비교 분석: SAC 대 PPO

- **온-폴리시 대 오프-폴리시의 구분**: 이것이 근본적인 차이점입니다. PPO는 온-폴리시, SAC는 오프-폴리시입니다.9
- **샘플 효율성**: SAC가 훨씬 더 샘플 효율적입니다. PPO는 각 업데이트 후 경험을 버리지만, SAC는 버퍼에서 재사용합니다.9 샘플이 비싼 작업의 경우 SAC가 명확한 선택입니다.45
- **안정성 및 실제 소요 시간**: PPO는 종종 더 안정적이고 구현이 간단한 것으로 간주됩니다. 업데이트가 덜 복잡합니다. 환경 시뮬레이터가 매우 빠르고 저렴하다면, PPO가 대량의 데이터를 매우 신속하게 생성하여 샘플 비효율성을 상쇄할 수 있으므로 실제 소요 시간 측면에서 더 빠르게 좋은 결과를 얻을 수 있습니다.45
- **성능**: 많은 표준 연속 제어 벤치마크(예: MuJoCo)에서 SAC는 일반적으로 PPO보다 더 높은 최종 점수(더 나은 점근적 성능)를 달성합니다.16

### 4.4  표준 벤치마크(예: MuJoCo)의 경험적 증거

최초의 SAC 논문 19과 후속 연구들은 

`Humanoid-v1`, `Ant-v1`, `HalfCheetah-v1`과 같은 도전적인 MuJoCo 벤치마크에서 SAC가 DDPG, PPO, 그리고 종종 TD3를 능가하는 성능을 보인다고 일관되게 보고합니다.16 문헌에는 상충되는 결과도 존재하며, 이는 성능이 구현 세부 사항, 센서 구성 및 특정 작업의 보상 함수에 민감할 수 있음을 강조합니다.53 그러나 전반적인 합의는 SAC가 샘플 효율성과 성능의 우수한 조합을 제공한다는 점을 지지합니다.47

## 5. V. 복잡한 도메인에서의 응용: 사례 연구 및 실제 구현

이 섹션에서는 이론에서 실천으로 넘어가, SAC의 특성이 어떻게 실제 문제 해결의 성공으로 이어지는지, 특히 로봇 공학에 중점을 두어 보여줍니다.

### 5.1  로봇 공학: 최고의 응용 분야

SAC의 성공적인 적용은 알고리즘 자체의 시연일 뿐만 아니라, 그 기저에 있는 최대 엔트로피 철학의 타당성을 입증하는 것입니다. 현실 세계는 예측 불가능하고 노이즈가 많습니다. 최대 엔트로피 강화학습이 장려하는 바로 그 속성들—견고함, 탐험, 행동 다양성—이 바로 그러한 환경에서 효과적으로 작동하는 데 필요한 속성들입니다.

- **사족보행 로봇 보행 (Quadrupedal Locomotion)**: SAC는 Google Brain의 미니타우르(Minitaur) 및 기타 사족보행 로봇을 훈련시켜 걷게 하는 데 성공적으로 사용되었습니다.25
  - **사례 연구: 미니타우르 로봇**: 단 2시간의 실제 상호작용만으로 정책이 훈련되었습니다. 결정적으로, 평지에서 훈련된 정책이 경사로나 작은 장애물과 같은 보지 못했던 지형에 대해서도 놀라운 일반화 성능을 보여주었는데, 이는 엔트로피 정규화 정책의 이점을 명확히 보여줍니다.25 이는 표준 강화학습 알고리즘이 평평한 훈련 지면에 과적합되어 불안정한 보행 패턴을 생성할 수 있는 반면, 엔트로피 보상을 받은 SAC 정책은 특정 지면 조건에 덜 의존적인 "더 유연하고" 적응력 있는 보행을 학습했음을 시사합니다.
- **정교한 조작 (Dexterous Manipulation)**: SAC는 비전과 같은 고차원 입력을 사용하는 복잡한 조작 작업에 적용되었습니다.
  - **사례 연구: 다이나믹셀 클로 (Dynamixel Claw)**: 로봇 팔이 시각적 입력만으로 밸브를 특정 방향으로 회전시키도록 훈련되었습니다. 학습된 정책은 시각적 교란에 대해서도 강인한 성능을 보였습니다.25 이는 정책이 단순히 픽셀 패턴을 암기하는 것이 아니라 더 일반화 가능한 제어 전략을 학습했음을 나타냅니다.
- **자율 주행 (Autonomous Navigation)**: SAC는 동적 장애물이 있는 환경에서 로봇 내비게이션에 사용됩니다. 부드러운 제어를 학습하고 신속하게 반응하는 능력 덕분에 실시간 경로 계획에 적합합니다.49 일부 연구에서는 훈련 안전성과 속도를 향상시키기 위해 SAC를 전문가 궤적을 이용한 모방 학습과 결합하기도 합니다.49
- **시뮬레이션-현실 전이 (Sim-to-Real Challenge)**: 로봇 공학의 주요 장애물 중 하나는 시뮬레이션에서 훈련된 정책을 실제 세계로 이전하는 것입니다. SAC의 견고함은 이 이전에 강력한 후보가 되게 하며, 연구들은 PPO 및 TD3와 비교하여 SAC의 sim-to-real 성능을 명시적으로 비교합니다.27

SAC의 성공은 로봇 학습의 병목 현상을 강화학습 알고리즘 자체에서 시뮬레이션 충실도(sim-to-real을 위한), 보상 함수 설계, 안전성과 같은 다른 영역으로 이동시키고 있습니다. 문제는 더 이상 "우리 알고리즘이 배울 수 있는가?"가 아니라 "알고리즘이 안전하고 효율적으로 해결할 올바른 문제를 명시할 수 있는가?"가 되고 있습니다. 이는 안전을 위해 SAC를 모방 학습과 결합하거나 49, 시간 지연과 같은 실제 문제를 처리하기 위해 복잡한 시뮬레이터를 개발하는 연구에 반영되어 있습니다.4

### 5.2  자율 시스템 및 산업 공정 제어

- **자율 주행 (Self-Driving)**: SAC는 실시간 연속 제어가 필수적인 자율 주행 시스템의 구성 요소에 대해 탐색되고 있습니다.59
- **산업 공정 (Industrial Processes)**: 복잡한 동역학과 연속 제어를 처리하는 SAC의 능력은 산업 최적화에 적용될 수 있습니다.
  - **사례 연구: 폐수 처리**: SAC는 복잡하고 비선형적인 동역학과 상당한 시간 지연이 특징인 폐수 처리 공장의 제어를 최적화하는 데 사용되었습니다. 연구자들은 SAC를 LSTM 기반 예측 시뮬레이터와 통합하여 기존 제어 방법보다 효율성을 개선하고 비용을 절감했음을 입증했습니다.4

## 6. VI. 고급 주제, 최신 변종 및 향후 방향

이 마지막 섹션에서는 SAC 구현의 미묘한 차이를 탐색하고, 그 변종들의 지형을 조사하며, 엔트로피 기반 강화학습의 미래를 조망합니다. SAC의 진화는 알고리즘의 불안정성이나 표현력의 한계를 식별하고 목표된 해결책을 제안하는 일관된 패턴을 보여줍니다. 이러한 반복적인 개선은 프레임워크의 성숙도와 모듈성을 보여줍니다.

### 6.1  구현의 미묘함과 일반적인 함정

- **하이퍼파라미터 민감도**: 자동 알파 튜닝이 도움이 되지만, SAC는 여전히 학습률이나 리플레이 버퍼 크기와 같은 다른 하이퍼파라미터에 민감합니다.2
- **보상 스케일링 (Reward Scaling)**: 보상 신호의 크기는 정책의 유효 "온도"에 직접적인 영향을 미칩니다. 부적절하게 스케일링된 보상 함수는 정책이 너무 무작위적이 되거나(엔트로피 보너스에 비해 보상이 너무 작은 경우) 너무 빨리 결정론적이 되는(보상이 너무 큰 경우) 결과를 초래할 수 있습니다.13 이는 매우 중요한 튜닝 고려 사항입니다.
- **손실 폭발 (Exploding Losses)**: 실무자들은 때때로 Q-가치가 무한대로 발산하면서 액터와 크리틱 손실이 폭발하는 현상을 보고합니다.59 이는 불안정성의 신호일 수 있으며, 종종 부적절한 보상 스케일링, 높은 학습률 또는 함수 근사 오차와 관련이 있습니다. 그래디언트 클리핑(gradient clipping)과 신중한 신경망 초기화가 일반적인 해결책입니다.
- **불충분한 탐험**: 엔트로피 보너스에도 불구하고, 목표 엔트로피 `$\mathcal{H}_{target}$`가 너무 낮게 설정되거나 보상이 너무 지배적이면 정책이 여전히 충분히 탐험하지 못할 수 있습니다.2

### 6.2  SAC 변종 조사: 안정성 및 표현력 향상

- **이산 행동을 위한 SAC (SAC for Discrete Actions)**: 원래의 SAC 공식은 이산 행동 공간에 맞게 조정될 수 있습니다. 주요 변경 사항은 정책에 대한 기댓값을 단일 샘플로 근사하는 대신 모든 행동에 대한 합으로 정확하게 계산할 수 있다는 점이며, 이는 분산을 줄일 수 있습니다.2
- **샘플링 개선: 우선순위 경험 리플레이 (Prioritized Replay)**: 샘플 효율성을 향상시키기 위해 일부 변종은 리플레이 버퍼에서 균일하게 샘플링하는 대신, 더 "놀랍거나" 유익한 경험 데이터를 선호하는 우선순위 기법으로 대체합니다.33
- **안정성 향상: Averaged-SAC**: 이 변종은 여러 이전 단계에 걸쳐 Q-가치 추정치를 평균 내어 잔여 Q-가치 과대평가 문제를 해결하며, 더 안정적인 학습 목표를 제공하고 분산을 줄이는 것을 목표로 합니다.61
- **튜닝 개선: Meta-SAC**: 이 접근법은 메타-그래디언트(meta-gradients)를 사용하여 `$\alpha$` 파라미터를 자동으로 튜닝하며, 이를 이중 경사 하강법에 대한 더 원칙적인 대안으로 제안하고 새로운 하이퍼파라미터를 도입하지 않는다고 주장합니다.40

### 6.3  다음 개척지: SAC와 고급 생성 모델의 통합

- **가우시안 정책의 한계**: SAC에서 사용되는 표준 가우시안 정책은 단일 모드(unimodal)입니다. 이는 복잡한 환경에서 여러 개의 뚜렷한 행동이 동등하게 좋을 수 있는 경우(예: 다중 목표 작업) 최적 정책을 표현하는 데 어려움을 겪을 수 있음을 의미합니다.64
- **확산 정책 (Diffusion Policies)**: 최첨단 연구 방향은 단순한 가우시안 액터를 **확산 모델(diffusion model)**과 같은 더 강력한 생성 모델로 대체하는 것입니다. 확산 정책은 행동 공간에 대해 매우 복잡하고 다중 모드인 분포를 학습할 수 있습니다.
- **MaxEntDP**: "MaxEnt RL with Diffusion Policy" (MaxEntDP) 방법은 바로 이 작업을 수행하여, SAC의 최대 엔트로피 프레임워크 내에서 확산 모델을 정책으로 통합합니다. 이는 복잡한 벤치마크에서 표준 SAC를 능가하는 성능을 보여주었습니다.64

SAC는 단일 알고리즘에서 최대 엔트로피 강화학습을 위한 유연한 **프레임워크**로 전환하고 있습니다. 핵심 구성 요소(soft Bellman 업데이트, 오프-폴리시 학습, 액터-크리틱 구조)는 "플러그인" 모듈을 허용할 만큼 충분히 견고합니다. 정책(가우시안 → 확산), 샘플링 전략(균일 → 우선순위), 또는 튜닝 메커니즘(고정-α → 이중-하강 → 메타-그래디언트)을 교체할 수 있습니다. 이러한 모듈성은 연구자들이 입증된 프레임워크에 새로운 딥러닝 발전을 계속 통합함에 따라 SAC의 원칙이 오랫동안 유효할 것임을 시사합니다.

### 6.4  결론 및 미해결 연구 질문

SAC는 딥러닝 기반 강화학습 분야에서 기념비적인 성과를 나타내며, 현대 연속 제어 및 로봇 공학 연구의 초석이 된 안정적이고 샘플 효율적이며 고성능의 알고리즘을 제공했습니다. 그 영향력은 단순히 벤치마크 점수를 높이는 것을 넘어, 강화학습의 목표를 재정의하고 실제 세계 적용의 가능성을 크게 확장한 데 있습니다.

그러나 여전히 많은 미해결 연구 질문이 남아 있습니다:

- 모델 기반 요소를 통합하여 샘플 효율성을 더욱 향상시킬 수 있는 방법은 무엇인가?
- 특히 실제 배포를 위해 엔트로피 정규화 정책에 대한 공식적인 안전 보장을 제공할 수 있는 방법은 무엇인가?
- 확산 모델과 같은 매우 표현력 있는 정책을 사용하는 SAC의 이론적 속성은 무엇인가?
- SAC의 원칙을 다중 에이전트 또는 계층적 강화학습 설정에 어떻게 더 효과적으로 확장할 수 있는가? 2

이러한 질문에 대한 탐구는 강화학습 분야를 더욱 발전시키고, SAC가 개척한 길을 따라 더욱 지능적이고 자율적인 시스템을 구축하는 데 기여할 것입니다.

#### **참고 자료**

1. Soft Actor-Critic Reinforcement Learning algorithm - GeeksforGeeks, accessed July 1, 2025, https://www.geeksforgeeks.org/soft-actor-critic-reinforcement-learning-algorithm/
2. Soft Actor-Critic: A Deep Dive - Number Analytics, accessed July 1, 2025, https://www.numberanalytics.com/blog/soft-actor-critic-deep-dive
3. Model-free (reinforcement learning) - Wikipedia, accessed July 1, 2025, https://en.wikipedia.org/wiki/Model-free_(reinforcement_learning)
4. Application of Soft Actor-Critic algorithms in optimizing wastewater treatment with time delays integration - Zenodo, accessed July 1, 2025, [https://zenodo.org/records/15685644/files/Application%20of%20Soft%20Actor-Critic_EMv2.pdf?download=1](https://zenodo.org/records/15685644/files/Application of Soft Actor-Critic_EMv2.pdf?download=1)
5. Bootcamp Summer 2020 Week 4 – On-Policy vs Off-Policy Reinforcement Learning, accessed July 1, 2025, https://core-robotics.gatech.edu/2022/02/28/bootcamp-summer-2020-week-4-on-policy-vs-off-policy-reinforcement-learning/
6. Soft Actor-Critic (SAC) Agent - MATLAB & Simulink - MathWorks, accessed July 1, 2025, https://www.mathworks.com/help/reinforcement-learning/ug/soft-actor-critic-agents.html
7. Mastering Soft Actor-Critic in ML - Number Analytics, accessed July 1, 2025, https://www.numberanalytics.com/blog/ultimate-guide-to-soft-actor-critic
8. Soft Actor-Critic Reinforcement Learning algorithm - GeeksforGeeks, accessed July 1, 2025, https://www.geeksforgeeks.org/deep-learning/soft-actor-critic-reinforcement-learning-algorithm/
9. AWS DeepRacer training algorithms - AWS DeepRacer, accessed July 1, 2025, https://docs.aws.amazon.com/deepracer/latest/developerguide/deepracer-how-it-works-reinforcement-learning-algorithm.html
10. ml-agents-1/docs/Training-SAC.md at master - GitHub, accessed July 1, 2025, https://github.com/yosider/ml-agents-1/blob/master/docs/Training-SAC.md
11. Soft Actor-Critic: - Proceedings of Machine Learning Research, accessed July 1, 2025, https://proceedings.mlr.press/v80/haarnoja18b/haarnoja18b.pdf
12. Soft Actor-Critic — Spinning Up documentation - OpenAI, accessed July 1, 2025, https://spinningup.openai.com/en/latest/algorithms/sac.html
13. SAC — DI-engine 0.1.0 documentation, accessed July 1, 2025, https://opendilab.github.io/DI-engine/12_policies/sac.html
14. Which Reinforcement learning-RL algorithm to use where, when and in what scenario? | by Ujwal Tewari | DataDrivenInvestor, accessed July 1, 2025, https://medium.datadriveninvestor.com/which-reinforcement-learning-rl-algorithm-to-use-where-when-and-in-what-scenario-e3e7617fb0b1
15. Soft Actor Critic Explained | Papers With Code, accessed July 1, 2025, https://paperswithcode.com/method/soft-actor-critic
16. SAC: A Soft Touch to Efficient Reinforcement Learnings | by Dong ..., accessed July 1, 2025, https://medium.com/@kdk199604/sac-a-soft-touch-to-efficient-reinforcement-learnings-c89acb1cb6b8
17. Maximum Entropy Reinforcement Learning via Energy-Based Normalizing Flow - NIPS, accessed July 1, 2025, https://proceedings.neurips.cc/paper_files/paper/2024/file/661c37f3b098bdee53fd7d9c4ef6964a-Paper-Conference.pdf
18. Soft Actor-Critic: Off-Policy Maximum Entropy Deep Reinforcement Learning with a Stochastic Actor - Emberpulse, accessed July 1, 2025, https://emberpulse.com/wp-content/uploads/2023/04/1801.01290-1.pdf
19. Soft Actor-Critic:, accessed July 1, 2025, https://ics.uci.edu/~dechter/courses/ics-295/winter-2018/papers/nips/soft-actor-critic-nips-2017.pdf
20. Corrected Soft Actor Critic for Continuous Control - arXiv, accessed July 1, 2025, https://arxiv.org/html/2410.16739v1
21. Soft Actor Critic, accessed July 1, 2025, https://rl-vs.github.io/rlvs2021/class-material/pg/12_sac.pdf
22. Reinforcement Learning Tutorial: Soft Actor-Critic (SAC) | by Jiawei Kong | Medium, accessed July 1, 2025, https://medium.com/@jiawei.kong/reinforcement-learning-tutorial-soft-actor-critic-sac-f0a097b328c3
23. Where does entropy enter in Soft Actor-Critic? - AI Stack Exchange, accessed July 1, 2025, https://ai.stackexchange.com/questions/16834/where-does-entropy-enter-in-soft-actor-critic
24. Entropy in Soft Actor-Critic (Part 2) | by Rafael Stekolshchik | TDS Archive | Medium, accessed July 1, 2025, https://medium.com/towards-data-science/entropy-in-soft-actor-critic-part-2-59821bdd5671
25. Soft Actor-Critic - Google Sites, accessed July 1, 2025, https://sites.google.com/view/sac-and-applications/
26. Soft Actor-Critic Algorithms and Applications - UT Computer Science, accessed July 1, 2025, https://www.cs.utexas.edu/~yukez/cs391r_fall2023/slides/pre_09-26_Jacob.pdf
27. Sim-to-Real: A Performance Comparison of PPO, TD3, and SAC Reinforcement Learning Algorithms for Quadruped Walking Gait Generation - Scientific Research Publishing, accessed July 1, 2025, https://www.scirp.org/journal/paperinformation?paperid=131938
28. Soft Actor-Critic: Off-Policy Maximum Entropy Deep Reinforcement Learning with a Stochastic Actor | OpenReview, accessed July 1, 2025, https://openreview.net/forum?id=HJjvxl-Cb
29. Soft Actor-Critic: Off-Policy Maximum Entropy Deep Reinforcement Learning with a Stochastic Actor - arXiv, accessed July 1, 2025, https://arxiv.org/abs/1801.01290
30. paperswithcode.com, accessed July 1, 2025, [https://paperswithcode.com/method/soft-actor-critic#:~:text=Soft%20Actor%20Critic%2C%20or%20SAC,acting%20as%20randomly%20as%20possible.](https://paperswithcode.com/method/soft-actor-critic#:~:text=Soft Actor Critic%2C or SAC,acting as randomly as possible.)
31. Soft Actor-Critic Algorithms and Applications - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/329705402_Soft_Actor-Critic_Algorithms_and_Applications
32. [1812.05905] Soft Actor-Critic Algorithms and Applications - ar5iv - arXiv, accessed July 1, 2025, https://ar5iv.labs.arxiv.org/html/1812.05905
33. Improved Soft Actor-Critic: Mixing Prioritized Off-Policy Samples with On-Policy Experience - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2109.11767
34. Entropy in Soft Actor-Critic (Part 1) | by Rafael Stekolshchik | TDS Archive | Medium, accessed July 1, 2025, https://medium.com/data-science/entropy-in-soft-actor-critic-part-1-92c2cd3a3515
35. Soft Actor-Critic (SAC) | Advanced RL - ApX Machine Learning, accessed July 1, 2025, https://apxml.com/courses/advanced-reinforcement-learning/chapter-3-advanced-policy-gradients-actor-critic/sac-algorithm
36. Unveiling the Nuances: Dissecting the Differences Between DDPG and TD3, accessed July 1, 2025, https://shivang-ahd.medium.com/unveiling-the-nuances-dissecting-the-differences-between-ddpg-and-td3-fc4ede24f636
37. Twin Delayed Deep Deterministic Policy Gradient (TD3) | by Hey ..., accessed July 1, 2025, https://medium.com/@heyamit10/twin-delayed-deep-deterministic-policy-gradient-td3-fc8e9950f029
38. 11_sac.ipynb - FareedKhan-dev/all-rl-algorithms - GitHub, accessed July 1, 2025, https://github.com/FareedKhan-dev/all-rl-algorithms/blob/master/11_sac.ipynb
39. Soft Actor-Critic (SAC) - CleanRL, accessed July 1, 2025, https://docs.cleanrl.dev/rl-algorithms/sac/
40. Meta-SAC: Auto-tune the Entropy Temperature of Soft Actor-Critic via Metagradient - AutoML.org, accessed July 1, 2025, https://www.automl.org/wp-content/uploads/2020/07/AutoML_2020_paper_47.pdf
41. TARGET ENTROPY ANNEALING FOR DISCRETE SOFT ACTOR–CRITIC - OpenReview, accessed July 1, 2025, https://openreview.net/pdf?id=jJKzGBBQiZu
42. arXiv:2007.01932v2 [cs.LG] 31 Jul 2020, accessed July 1, 2025, https://arxiv.org/pdf/2007.01932
43. SAC policy loss and entropy balance : r/reinforcementlearning - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/14fkskz/sac_policy_loss_and_entropy_balance/
44. Choosing "Target Entropy" for Soft-Actor-Critic (SAC) algorithm - Cross Validated, accessed July 1, 2025, https://stats.stackexchange.com/questions/561624/choosing-target-entropy-for-soft-actor-critic-sac-algorithm
45. Are there any papers or theories on why SAC is better for continuous control tasks than on-policy methods? - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/y2af2i/are_there_any_papers_or_theories_on_why_sac_is/
46. Evaluating PPO and SAC Algorithms for Continuous Control | Febin Wilson, accessed July 1, 2025, https://www.febinwilson.com/portfolio-collections/my-portfolio/project-title-6
47. DDPG vs PPO vs SAC: when to use? : r/reinforcementlearning - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/holioy/ddpg_vs_ppo_vs_sac_when_to_use/
48. Deep Reinforcement Learning with Soft Actor-Critic | by Hey Amit | Biased-Algorithms, accessed July 1, 2025, https://medium.com/biased-algorithms/deep-reinforcement-learning-with-soft-actor-critic-de82afa659f8
49. A Soft Actor-Critic Deep Reinforcement-Learning-Based Robot Navigation Method Using LiDAR - MDPI, accessed July 1, 2025, https://www.mdpi.com/2072-4292/16/12/2072
50. Learning Object-conditioned Exploration using Distributed Soft Actor Critic, accessed July 1, 2025, https://proceedings.mlr.press/v155/wahid21a/wahid21a.pdf
51. Parameter Tuning for Quadruped Locomotion Training Using SAC in rl-games - Isaac Gym, accessed July 1, 2025, https://forums.developer.nvidia.com/t/parameter-tuning-for-quadruped-locomotion-training-using-sac-in-rl-games/332130
52. DD-PPO, TD3, SAC: which is the best? : r/reinforcementlearning - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/r83umm/ddppo_td3_sac_which_is_the_best/
53. A Comparison of PPO, TD3 and SAC Reinforcement Algorithms for Quadruped Walking Gait Generation - Scientific Research Publishing, accessed July 1, 2025, https://www.scirp.org/journal/paperinformation?paperid=123401
54. Proximal Policy Optimization (PPO): Clipping, Stability, and Sample Efficiency in Reinforcement Learning | by Shivang Shrivastav, accessed July 1, 2025, https://shivang-ahd.medium.com/proximal-policy-optimization-clipping-stability-and-sample-efficiency-in-reinforcement-learning-7f0fec90da7a
55. Does SAC perform better than PPO in sample-expensive tasks with discrete action spaces?, accessed July 1, 2025, https://ai.stackexchange.com/questions/36092/does-sac-perform-better-than-ppo-in-sample-expensive-tasks-with-discrete-action
56. Why is PPO better than TD3? : r/reinforcementlearning - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/rld20c/why_is_ppo_better_than_td3/
57. [1812.05905] Soft Actor-Critic Algorithms and Applications - arXiv, accessed July 1, 2025, https://arxiv.org/abs/1812.05905
58. A Comparison of PPO, TD3 and SAC Reinforcement Algorithms for Quadruped Walking Gait Generation - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/369526779_A_Comparison_of_PPO_TD3_and_SAC_Reinforcement_Algorithms_for_Quadruped_Walking_Gait_Generation
59. SAC: exploding losses and huge value underestimation in custom robot environments : r/reinforcementlearning - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/11p7c5k/sac_exploding_losses_and_huge_value/
60. Application of Soft Actor-Critic Algorithms in Optimizing Wastewater Treatment with Time Delays Integration - arXiv, accessed July 1, 2025, https://arxiv.org/pdf/2411.18305
61. Research Article Averaged Soft Actor-Critic for Deep Reinforcement Learning - Semantic Scholar, accessed July 1, 2025, https://pdfs.semanticscholar.org/0390/f358ccfcfe733d61d7b0d920c7fb5e4c34df.pdf
62. Meta-SAC: Auto-tune the Entropy Temperature of Soft Actor-Critic via Metagradient, accessed July 1, 2025, https://www.researchgate.net/publication/342733600_Meta-SAC_Auto-tune_the_Entropy_Temperature_of_Soft_Actor-Critic_via_Metagradient
63. Meta-SAC: Auto-tune the Entropy Temperature of Soft Actor-Critic via Metagradient - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2007.01932
64. [2502.11612] Maximum Entropy Reinforcement Learning with Diffusion Policy - arXiv, accessed July 1, 2025, https://arxiv.org/abs/2502.11612


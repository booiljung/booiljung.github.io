[Q-Learning](./index.md)
# Q-러닝


인공지능(AI)의 발전사에서 강화학습(Reinforcement Learning, RL)은 기계가 시행착오를 통해 최적의 행동 전략을 스스로 학습하는 패러다임을 제시하며 중요한 이정표를 세웠다. 수많은 강화학습 알고리즘 중에서도 Q-러닝(Q-Learning)은 그 단순성과 강력한 이론적 기반 덕분에 가장 근본적이고 영향력 있는 방법론 중 하나로 자리매김했다.1 1989년 크리스토퍼 왓킨스(Christopher Watkins)에 의해 처음 제안된 이 알고리즘은, 환경에 대한 사전 모델 없이도 최적의 행동 가치 함수를 학습할 수 있는 능력으로 인해 강화학습 연구의 초석이 되었다.3

본 보고서는 Q-러닝에 대한 심층적이고 포괄적인 고찰을 제공하는 것을 목표로 한다. 이를 위해 먼저 Q-러닝이 속한 강화학습의 이론적 배경, 특히 순차적 의사결정 문제를 수학적으로 정형화하는 마르코프 결정 과정(Markov Decision Process, MDP)과 가치 기반 학습의 핵심인 벨만 방정식(Bellman Equation)을 탐구한다. 이 이론적 토대 위에서 고전적인 테이블 기반 Q-러닝 알고리즘의 작동 메커니즘을 단계별로 분석하고, 학습률(learning rate) 및 할인율(discount factor)과 같은 핵심 하이퍼파라미터의 역할과 의미를 명확히 규명한다.

나아가, 모든 강화학습 에이전트가 직면하는 본질적인 딜레마인 '탐험과 활용(Exploration-Exploitation)' 문제를 심도 있게 다루고, 이를 해결하기 위한 대표적인 전략들을 비교 분석한다. 또한, Q-러닝을 유사한 시간차 학습(Temporal-Difference Learning) 알고리즘인 SARSA와 비교하며, '오프-폴리시(Off-Policy)' 학습 방식이 갖는 고유한 특성과 그 의미를 파헤친다.

그러나 고전적 Q-러닝은 상태 공간(state space)이 거대해지는 현실 세계의 복잡한 문제에 적용하기 어렵다는 명백한 한계를 지닌다. 본 보고서는 이러한 '차원의 저주(curse of dimensionality)'를 극복하기 위해 딥러닝과 Q-러닝을 결합한 심층 Q-네트워크(Deep Q-Networks, DQN)의 등장을 조명한다. DQN의 혁신적인 아이디어와 더불어, 학습 안정화를 위해 도입된 경험 리플레이(Experience Replay)와 타겟 네트워크(Target Network)와 같은 핵심 기법들의 원리를 상세히 설명한다.

DQN의 성공 이후, 연구자들은 그 성능을 더욱 향상시키기 위해 다양한 변형 알고리즘들을 제안했다. 본 보고서는 Double DQN, Dueling DQN, Prioritized Experience Replay(PER) 등 소위 'DQN Zoo'를 구성하는 주요 알고리즘들을 체계적으로 검토하고, 이들이 각각 Q-러닝의 어떤 측면을 개선했는지 분석한다. 마지막으로, 그리드월드(Gridworld)와 같은 고전적 예제부터 게임 AI, 로보틱스, 금융 및 추천 시스템에 이르는 다양한 실제 적용 사례를 통해 Q-러닝의 실용적 가치를 조명하고, 현재의 도전 과제와 미래 연구 방향을 제시하며 보고서를 마무리한다.

------


Q-러닝을 깊이 있게 이해하기 위해서는 그것이 탄생한 이론적 배경을 먼저 살펴보아야 한다. 이 장에서는 강화학습의 기본 개념과 이를 수학적으로 정형화하는 마르코프 결정 과정, 그리고 가치 기반 학습의 수학적 근간이 되는 벨만 방정식에 대해 상세히 논한다.


강화학습은 기계학습의 한 분야로, **에이전트(agent)**라는 학습 주체가 주어진 **환경(environment)**과 상호작용하며 누적 **보상(reward)**을 최대화하는 최적의 행동 방침, 즉 **정책(policy)**을 학습하는 과정을 다룬다.1 이는 정답이 주어진 데이터를 학습하는 지도학습이나 데이터 내의 구조를 찾는 비지도학습과는 구별되는 독자적인 패러다임이다.4

**에이전트-환경 상호작용 루프**

강화학습의 핵심은 에이전트와 환경 간의 순차적인 상호작용 루프에 있다.6 이 과정은 다음과 같이 전개된다.

1. 에이전트는 시간 t에 환경의 현재 **상태(state)** st를 관찰한다.7
2. 관찰된 상태 st를 바탕으로, 에이전트는 자신의 정책에 따라 특정 **행동(action)** at를 선택하여 수행한다.7
3. 환경은 에이전트의 행동 at에 반응하여 새로운 상태 $s_{t+1}$로 전이하고, 그 결과에 대한 피드백으로 스칼라 값인 보상 $r_{t+1}$을 에이전트에게 전달한다.6
4. 에이전트는 이 과정을 반복하며 상태, 행동, 보상으로 이루어진 경험의 궤적(trajectory)을 생성하고, 이 경험을 바탕으로 자신의 정책을 개선해 나간다.9 에이전트의 유일한 목표는 장기적으로 얻게 될 보상의 총합, 즉 누적 보상을 최대화하는 것이다.4

**마르코프 결정 과정을 통한 문제의 정형화**

이러한 순차적 의사결정 문제는 **마르코프 결정 과정(Markov Decision Process, MDP)**이라는 수학적 틀을 통해 엄밀하게 정의된다.2 MDP는 다음과 같은 튜플 $(S, A, P, R, \gamma)$로 구성된다.

- **S (상태 공간, State Space):** 에이전트가 처할 수 있는 모든 가능한 상태의 집합.7
- **A (행동 공간, Action Space):** 에이전트가 각 상태에서 취할 수 있는 모든 가능한 행동의 집합.7
- **P (상태 전이 확률 함수, State Transition Probability Function):** $P(s'|s, a) = Pr(S_{t+1}=s'|S_t=s, A_t=a)$로, 상태 s에서 행동 a를 취했을 때 다음 상태가 $s'$이 될 확률을 나타낸다. 이는 환경의 동역학(dynamics)을 정의한다.12
- **R (보상 함수, Reward Function):** $R(s, a, s')$은 상태 s에서 행동 a를 취해 상태 $s'$으로 전이했을 때 에이전트가 받는 즉각적인 보상을 정의한다.12
- **γ (할인율, Discount Factor):** 0≤γ≤1 범위의 값으로, 미래 보상의 현재 가치를 결정하는 역할을 한다.

MDP의 가장 핵심적인 가정은 **마르코프 속성(Markov Property)**이다. 이는 미래의 상태와 보상은 오직 현재의 상태와 행동에만 의존하며, 그 이전의 과거 이력과는 무관하다는 것을 의미한다.6 즉, $Pr(S_{t+1}=s', R_{t+1}=r | S_t, A_t, R_t,..., S_0, A_0) = Pr(S_{t+1}=s', R_{t+1}=r | S_t, A_t)$가 성립한다. 이 속성 덕분에 에이전트는 과거의 모든 이력을 기억할 필요 없이 현재 상태 정보만으로 최적의 의사결정을 내릴 수 있으며, 이는 강화학습 문제의 계산적 복잡성을 크게 낮추는 결정적인 역할을 한다.

MDP 프레임워크의 진정한 힘은 그 추상화 능력에 있다. 탁구 게임에서 에이전트의 상태를 공의 위치, 속도, 가속도로 정의하고, 행동을 라켓의 움직임으로, 보상을 득점과 실점으로 설정하는 것 4, 로봇 내비게이션에서 상태를 센서 데이터로, 행동을 모터 제어로, 보상을 목표 도달 및 장애물 충돌 여부로 정의하는 것 13, 혹은 금융 거래에서 상태를 시장 지표로, 행동을 매수/매도/보유로, 보상을 포트폴리오 가치 변화로 정의하는 것 14 등, 표면적으로 전혀 다른 문제들을 모두 동일한 MDP라는 언어로 기술할 수 있다. 따라서 Q-러닝과 같은 범용 알고리즘이 이처럼 다양한 분야에 적용될 수 있는 근본적인 이유는, 이 모든 문제들이 성공적으로 MDP로 정형화될 수 있기 때문이다. 결국 강화학습 적용의 성패는 주어진 문제를 얼마나 효과적으로 MDP의 상태, 행동, 보상으로 설계하는지에 달려있다고 해도 과언이 아니다.


가치 기반 강화학습의 목표는 각 상태나 상태-행동 쌍이 장기적으로 얼마나 "좋은지"를 나타내는 가치 함수(value function)를 학습하는 것이다. Q-러닝은 이 가치 함수 중 행동 가치 함수를 학습하는 대표적인 알고리즘이며, 그 이론적 기반에는 벨만 방정식이 있다.

**가치 함수**

- **상태 가치 함수(State-Value Function) Vπ(s):** 상태 s에서 시작하여 정책 π를 따를 때 기대되는 누적 할인 보상(return)을 의미한다.15 수학적으로는 

  Vπ(s)=Eπ로 정의되며, 여기서 $G_t = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}$는 시간 t에서의 누적 할인 보상이다.

- **행동 가치 함수(Action-Value Function) Qπ(s,a):** 상태 s에서 행동 a를 취한 후, 그 이후부터 정책 π를 따를 때 기대되는 누적 할인 보상을 의미한다.15 수학적으로는 

  Qπ(s,a)=Eπ로 정의된다. Q-러닝의 'Q'는 바로 이 행동 가치의 품질(Quality)을 의미하며, 알고리즘의 최종 목표는 최적의 행동 가치 함수 $Q^*(s, a)$를 찾는 것이다.1

**벨만 기대 방정식**

벨만 기대 방정식(Bellman Expectation Equation)은 가치 함수의 재귀적 성질을 나타내는 핵심적인 관계식이다. 이 방정식은 현재 상태(또는 상태-행동 쌍)의 가치를 즉각적인 보상과 다음 상태(또는 상태-행동 쌍)의 가치의 합으로 분해하여 표현한다.15

- $V^\pi(s)$에 대한 벨만 기대 방정식:

  

  Vπ(s)=Eπ

- $Q^\pi(s, a)$에 대한 벨만 기대 방정식:

  

  Qπ(s,a)=Eπ

이 재귀적 구조는 복잡한 장기적 보상 계산 문제를 현재의 보상과 한 단계 미래의 가치로 나누어 풀 수 있게 해주며, 동적 프로그래밍(Dynamic Programming)이나 시간차 학습(Temporal-Difference Learning)과 같은 반복적 해법의 근간이 된다.15

**벨만 최적 방정식**

만약 우리가 특정 정책 π를 평가하는 것이 아니라, 모든 정책 중에서 가장 좋은 **최적 정책(optimal policy)** $\pi^*$를 찾고자 한다면, 벨만 최적 방정식(Bellman Optimality Equation)을 사용해야 한다. 최적 정책을 따를 때의 가치 함수는 다른 어떤 정책을 따를 때보다 모든 상태에서 가치가 크거나 같다. 벨만 최적 방정식은 최적 가치 함수가 만족해야 하는 조건을 명시한다.15

최적 행동 가치 함수 $Q^(s, a)$에 대한 벨만 최적 방정식은 다음과 같다:

$$Q^(s, a) = \mathbb{E}$$

이 방정식의 핵심은 기댓값 안에 포함된 maxa′ 연산자이다. 이는 다음 상태 $s'$에서 가능한 모든 행동 a′ 중에서 최적의 가치를 갖는 행동을 선택한다는 것을 의미한다. 즉, 항상 최선의 선택을 가정함으로써 최적의 가치를 계산하는 것이다. 이 방정식은 Q-러닝 알고리즘이 궁극적으로 풀고자 하는 목표 방정식이다.

**수렴성 증명과 벨만 연산자 (축소 사상 증명)**

Q-러닝과 같은 반복적 알고리즘이 과연 벨만 최적 방정식을 만족하는 유일한 최적해로 수렴할 수 있는지에 대한 이론적 보장은 매우 중요하다. 이 증명은 벨만 연산자(Bellman Operator)와 바나흐 고정점 정리(Banach Fixed-Point Theorem)를 통해 이루어진다.

벨만 최적 연산자 $T^$는 임의의 행동 가치 함수 Q를 입력받아 새로운 행동 가치 함수를 출력하는 함수로 정의할 수 있다.22

(T∗Q)(s,a)=E



이때, 최적 가치 함수 $Q^$는 이 연산자의 고정점(fixed point)이다. 즉, $T^Q^ = Q^*$를 만족한다.22

이 벨만 최적 연산자 $T^*$가 **$\gamma$-축소 사상(γ-contraction mapping)**임을 보일 수 있다. 축소 사상이란, 연산자를 적용했을 때 함수 공간 내의 두 점(여기서는 두 개의 Q함수) 사이의 거리가 최소 γ배만큼 줄어드는 성질을 의미한다. L-무한대 노름(∥⋅∥∞)을 거리 척도로 사용하면, 임의의 두 Q함수 Q1과 Q2에 대해 다음이 성립한다 22:

∥T∗Q1−T∗Q2∥∞≤γ∥Q1−Q2∥∞

바나흐 고정점 정리에 따르면, 완비 거리 공간(complete metric space)에서 정의된 축소 사상은 유일한 고정점을 가지며, 임의의 시작점에서 연산자를 반복적으로 적용하면 그 고정점으로 반드시 수렴한다.22 Q함수 공간은 완비 거리 공간이고 벨만 최적 연산자는 축소 사상이므로, 이 정리에 의해 유일한 최적 Q함수 $Q^

*$가 존재하며, 반복적인 업데이트를 통해 $Q^*$로 수렴함이 보장된다.24

이러한 이론적 배경은 Q-러닝 알고리즘의 타당성을 뒷받침한다. Q-러닝은 사실상 동적 프로그래밍의 한 기법인 **가치 반복법(Value Iteration)**을 모델 없이(model-free) 구현한 것으로 볼 수 있다. 가치 반복법은 환경의 모델(전이 확률 P와 보상 함수 R)을 알 때 벨만 최적 연산자를 모든 상태에 대해 반복적으로 적용하여 최적 가치 함수를 찾는 알고리즘이다.2

반면, Q-러닝은 환경 모델을 모르는 상태에서 실제 경험을 통해 얻은 단일 샘플 $(s, a, r, s')$을 사용한다. Q-러닝의 업데이트 목표인 $r + \gamma \max_{a'} Q(s', a')$는 벨만 최적 연산자의 기댓값을 단일 샘플로 근사한 것이다. 즉, Q-러닝은 가치 반복법의 전체 백업(full backup)을 샘플 기반의 확률적 근사(stochastic approximation) 백업으로 대체한 것이다.29 이 연결고리 덕분에 Q-러닝은 가치 반복법의 수렴성을 물려받으며, 적절한 탐험과 학습률 조절 하에 최적 정책을 찾을 수 있다는 강력한 이론적 보증을 갖게 된다.30

------



## 제 2부: 표준 Q-러닝 알고리즘



이론적 토대를 바탕으로, 이 장에서는 고전적인 테이블 기반 Q-러닝 알고리즘의 구체적인 작동 방식과 핵심 요소들을 상세히 분석한다.



### 제 2.1절: 알고리즘 메커니즘과 Q-테이블



가장 기본적인 형태의 Q-러닝은 행동 가치 함수 $Q(s, a)$를 **Q-테이블(Q-Table)**이라는 2차원 배열 형태의 조회 테이블(lookup table)로 저장한다.31 이 테이블의 행은 상태(state)를, 열은 행동(action)을 나타내며, 각 셀 

`Q(s, a)`에는 상태 s에서 행동 a를 취했을 때의 가치 추정치가 저장된다.17

알고리즘은 에피소드(episode) 단위로 진행되며, 각 에피소드는 여러 타임 스텝(time step)으로 구성된다. 전체적인 학습 과정은 다음과 같다.2

1. **초기화 (Initialization):** Q-테이블의 모든 값을 0 또는 작은 임의의 값으로 초기화한다.17

2. **에피소드 반복 (Loop for each episode):**

3. 타임 스텝 반복 (Loop for each step of episode):

   a.  현재 상태 s를 관찰한다.

   b.  현재 Q-테이블에 기반한 정책(예: ε-탐욕 정책)을 사용하여 현재 상태 s에서 수행할 행동 a를 선택한다.18

   

   c.  선택한 행동 a를 수행하고, 환경으로부터 보상 r과 다음 상태 $s'$를 관찰한다.31

   

   d.  관찰된 (s,a,r,s′) 정보를 사용하여 벨만 업데이트 규칙에 따라 Q-테이블의 Q(s, a) 값을 갱신한다.18

   

   e.  다음 스텝을 위해 현재 상태를 $s'$로 변경한다 (s←s′).

4. 에이전트가 종료 상태(terminal state)에 도달하거나 에피소드가 끝날 때까지 3번 과정을 반복한다.

5. Q-테이블의 값들이 수렴할 때까지 2~4번 과정을 수많은 에피소드에 걸쳐 반복한다.31

학습이 충분히 진행되어 Q-테이블이 최적 행동 가치 함수 $Q^*$에 가깝게 수렴하면, 최적 정책 $\pi^*$는 각 상태에서 Q-값이 가장 큰 행동을 선택하는 탐욕적 정책(greedy policy)으로 간단히 도출할 수 있다.17

π∗(s)=aargmaxQ∗(s,a)



### 제 2.2절: Q-가치 업데이트 규칙 심층 분석



Q-러닝의 핵심은 Q-테이블을 업데이트하는 규칙에 있다. 이 규칙은 시간차 학습(Temporal-Difference Learning)의 한 형태로, 벨만 방정식에 기반한다.2

**업데이트 공식**

Q-값 업데이트는 다음 공식에 따라 이루어진다.31

Q(s,a)←Q(s,a)+α[r+γa′maxQ(s′,a′)−Q(s,a)]

이 공식을 분석해 보면, 업데이트는 현재 추정치 $Q(s, a)$에 **시간차 오차(TD error)**, $\delta_t = r + \gamma \max_{a'} Q(s', a') - Q(s, a)$에 학습률 α를 곱한 값을 더하는 방식으로 이루어진다. 여기서 $r + \gamma \max_{a'} Q(s', a')$는 **TD 타겟(TD target)**이라 불리며, 실제 경험을 통해 얻은 더 나은 가치 추정치를 의미한다. TD 오차는 이 새로운 목표치와 기존 추정치 간의 차이를 나타내며, 에이전트는 이 오차를 줄이는 방향으로 학습을 진행한다.

**학습률 (Learning Rate, α)**

- **역할:** 학습률 α (0<α≤1)는 새로 얻은 정보를 기존 정보에 얼마나 반영할지를 결정하는 비율이다.2 즉, 업데이트의 보폭(step size)을 제어한다.
- **영향:** α=0이면 에이전트는 아무것도 배우지 않는다. 반면 α=1이면 에이전트는 과거의 경험을 완전히 무시하고 가장 최근의 정보만으로 Q-값을 갱신한다.38
- **예시:** 목표 지점에 도달하면 보상 10을 받는 환경을 가정하자. 에이전트가 처음 목표에 도달했을 때, 해당 상태-행동 쌍의 Q-값은 0이었다고 하자. TD 타겟은 10이 된다. 만약 α=0.1이라면, 새로운 Q-값은 0+0.1×(10−0)=1이 된다. 만약 α=0.8이라면, 새로운 Q-값은 0+0.8×(10−0)=8이 된다. 이처럼 높은 학습률은 Q-값을 빠르게 변화시키지만, 환경의 보상이 확률적이거나 노이즈가 있는 경우 학습이 불안정해질 수 있다. 따라서 안정적인 수렴을 위해 적절한 α 값을 설정하는 것이 매우 중요하다.39

**할인율 (Discount Factor, γ)**

- **역할:** 할인율 γ (0≤γ<1)는 미래 보상의 현재 가치를 결정하는 요소로, 에이전트의 "인내심" 또는 "선견지명"을 제어한다.2
- **영향:** γ=0에 가까우면 에이전트는 즉각적인 보상만을 중시하는 근시안적인(myopic) 행동을 하게 된다. 반면 γ=1에 가까우면 에이전트는 당장의 손해를 감수하더라도 장기적으로 더 큰 보상을 얻기 위해 노력하는 원시안적인(farsighted) 행동을 학습한다.2
- **예시:** 두 가지 경로가 있다고 가정하자. A 경로는 즉시 +1의 보상을 받고 끝난다. B 경로는 즉시 -1의 보상을 받지만, 그 다음 상태에서 +10의 보상을 받을 수 있다.
  - 만약 γ=0.1이라면, B 경로의 첫 행동 가치는 대략 −1+0.1×10=0이 된다. 에이전트는 즉각적인 보상이 더 큰 A 경로를 선호하게 될 것이다.
  - 만약 γ=0.9라면, B 경로의 첫 행동 가치는 대략 −1+0.9×10=+8이 된다. 에이전트는 장기적인 이득이 훨씬 큰 B 경로를 선택하도록 학습될 것이다.
  - 이처럼 할인율은 단기적인 유혹을 뿌리치고 장기적인 계획을 세워야 하는 복잡한 문제 해결에 필수적인 파라미터이다.37



### 제 2.3절: 탐험과 활용의 딜레마



강화학습 에이전트는 매 순간 중대한 딜레마에 직면한다. 이는 현재까지의 지식을 바탕으로 최선의 행동을 할 것인가(활용), 아니면 더 나은 선택지를 찾기 위해 새로운 시도를 할 것인가(탐험)의 문제이다.

- **활용 (Exploitation):** 현재까지 학습한 Q-값에 근거하여 가장 높은 보상을 줄 것으로 기대되는 행동을 선택하는 것. 이는 단기적인 이익을 극대화하는 탐욕적인(greedy) 접근 방식이다.40
- **탐험 (Exploration):** 아직 시도해보지 않았거나 덜 시도해본, 잠재적으로 차선으로 보이는 행동을 선택하여 환경에 대한 새로운 정보를 수집하는 것. 이는 장기적으로 더 나은 정책을 발견할 가능성을 열어준다.40

이 딜레마는 강화학습의 성공에 매우 중요하다. 활용에만 치중하면 에이전트는 초기에 발견한 국소 최적해(local optimum)에 갇혀 진정한 최적해를 놓칠 수 있다.40 반대로 탐험에만 치중하면, 이미 좋은 전략을 알고 있음에도 불구하고 비효율적인 행동을 반복하여 학습이 느려지고 전반적인 성능이 저하된다.42 따라서 성공적인 강화학습은 탐험과 활용 사이의 적절한 균형을 찾는 데 달려있다.41

**주요 탐험 전략**

이 딜레마를 해결하기 위해 여러 전략이 제안되었으며, 대표적인 두 가지는 다음과 같다.

- **ε-탐욕 (Epsilon-Greedy) 정책:** 가장 간단하고 널리 사용되는 전략이다. 에이전트는 1−ϵ의 확률로 현재 Q-값이 가장 높은 행동을 선택하여 활용하고, ϵ이라는 작은 확률로 가능한 행동 중 하나를 무작위로 선택하여 탐험한다.41

  - **작동 방식:** 0과 1 사이의 난수를 생성하여 이 값이 ϵ보다 작으면 무작위 행동을, 그렇지 않으면 탐욕적 행동을 선택한다.18

  - **장점:** 구현이 매우 간단하고 직관적이다.45

  - **단점:** 탐험 시 모든 비-탐욕적 행동들을 동일한 확률로 선택하기 때문에, 명백히 나쁜 선택지(예: 절벽으로 가는 행동)와 차선책을 구분하지 못하는 비효율성이 있다.47 이 문제를 완화하기 위해 학습이 진행됨에 따라 

    ϵ 값을 점차 줄여나가는 **감쇠 ε-탐욕(decaying ε-greedy)** 기법이 흔히 사용된다. 이는 학습 초기에는 적극적으로 탐험하고, 지식이 쌓임에 따라 점차 활용의 비중을 높이는 방식이다.45

- **소프트맥스 (Softmax) 또는 볼츠만 (Boltzmann) 탐험:** Q-값에 기반하여 행동 선택 확률을 가중치로 부여하는 보다 정교한 전략이다. Q-값이 높은 행동일수록 선택될 확률이 높아지지만, Q-값이 낮은 행동도 0이 아닌 선택 확률을 갖는다.48

  - **작동 방식:** 볼츠만 분포를 사용하여 각 행동의 선택 확률을 계산한다. P(a∣s)=∑a′eQ(s,a′)/\taueQ(s,a)/\tau. 여기서 \tau는 **온도(temperature)** 파라미터이다.
  - **장점:** ε-탐욕처럼 맹목적으로 탐험하는 대신, 더 가치가 높을 것으로 예상되는 행동들을 우선적으로 탐험하므로 보다 지능적인 탐험이 가능하다.48
  - **단점:** 온도 파라미터 \tau를 추가로 튜닝해야 하며, 이는 ϵ보다 덜 직관적일 수 있다. 또한 모든 행동에 대한 지수 함수 계산으로 인해 계산 복잡성이 증가한다.48

| 특징              | ε-탐욕 (Epsilon-Greedy)                           | 소프트맥스 (Softmax / Boltzmann)                    |
| ----------------- | ------------------------------------------------- | --------------------------------------------------- |
| **작동 방식**     | 확률 ϵ으로 무작위 행동, 1−ϵ으로 탐욕적 행동 선택  | 행동의 Q-값에 비례하는 확률 분포에 따라 행동 샘플링 |
| **핵심 파라미터** | ϵ (탐험 확률)                                     | \tau (온도, 탐험의 무작위성 정도)                      |
| **탐험의 질**     | 균등 무작위 (모든 비-탐욕적 행동을 동일하게 취급) | 가치 비례적 (더 나은 행동을 더 자주 탐험)           |
| **장점**          | 구현이 매우 간단하고 계산적으로 저렴함 45         | 더 지능적이고 효율적인 탐험 가능 48                 |
| **단점**          | 비효율적인 탐험 (명백히 나쁜 행동도 시도) 47      | 계산 복잡성이 높고, \tau 파라미터 튜닝이 필요함 52     |
| **적합한 환경**   | 초기 학습 단계, 간단한 문제                       | 나쁜 행동의 비용이 매우 큰 복잡한 문제              |



### 제 2.4절: 온-폴리시 대 오프-폴리시 학습: Q-러닝과 SARSA 비교 분석



시간차 학습 알고리즘은 학습 방식에 따라 온-폴리시(On-Policy)와 오프-폴리시(Off-Policy)로 나뉜다. 이 구분은 Q-러닝의 핵심적인 특징을 이해하는 데 매우 중요하다.

- **온-폴리시 (On-Policy) 학습:** 에이전트가 현재 행동을 생성하는 데 사용하는 정책과, 가치 함수를 평가하고 개선하는 데 사용하는 정책이 동일한 경우를 말한다. 즉, "자신이 행동하는 방식 그대로 배우는" 방법이다. 대표적인 온-폴리시 알고리즘은 **SARSA**이다.54
- **오프-폴리시 (Off-Policy) 학습:** 에이전트가 행동을 생성하는 정책(행동 정책, behavior policy)과 가치 함수를 평가하고 개선하는 정책(타겟 정책, target policy)이 다른 경우를 말한다. 일반적으로 타겟 정책은 최적의 탐욕적 정책이며, 행동 정책은 탐험을 포함하는 정책(예: ε-탐욕)이다. **Q-러닝**이 대표적인 오프-폴리시 알고리즘이다.54

**업데이트 규칙의 결정적 차이**

두 알고리즘의 차이는 Q-값 업데이트 공식에서 명확하게 드러난다.

- SARSA 업데이트 규칙:

  

  Q(s,a)←Q(s,a)+α[r+γQ(s′,a′)−Q(s,a)]

  

  SARSA라는 이름은 업데이트에 필요한 경험 튜플 $(s, a, r, s', a')$에서 유래했다.58 여기서 $a'$는 다음 상태 $s'$에서 

  **실제로 선택된 행동**이다. 즉, 자신의 행동 정책(예: ε-탐욕)에 따라 다음 행동을 결정하고, 그 행동의 Q-값을 업데이트에 사용한다.

- Q-러닝 업데이트 규칙:

  

  Q(s,a)←Q(s,a)+α[r+γa′′maxQ(s′,a′′)−Q(s,a)]

  

  Q-러닝은 다음 상태 $s'$에서 실제로 어떤 행동을 취했는지(a′)와 무관하게, $s'$에서 가능한 모든 행동 중 Q-값이 가장 큰 행동($\underset{a''}{\operatorname{argmax}} Q(s', a'')$)을 가정하여 업데이트한다.56 타겟 정책은 항상 탐욕적 정책인 것이다.

**"절벽 보행(Cliff Walking)" 예제를 통한 행동 비교**

이 고전적인 예제는 두 알고리즘의 행동 차이를 극명하게 보여준다.61

- **환경:** 에이전트는 출발점에서 목표점까지 가야 한다. 경로 중간에는 절벽이 있어, 절벽에 해당하는 상태로 이동하면 큰 음의 보상을 받고 출발점으로 돌아간다. 목표점으로 가는 최단 경로는 절벽 바로 옆을 따라가는 길이고, 더 안전하지만 먼 우회로도 존재한다.
- **Q-러닝의 행동 (오프-폴리시):** Q-러닝은 최적의 탐욕적 정책의 가치를 학습한다. 업데이트 시 max 연산자를 사용하므로, 탐험으로 인해 절벽으로 떨어질 가능성을 무시한다. 따라서 절벽 옆을 따라가는 것이 최적 경로라고 학습하며, 이는 가장 높은 보상을 기대할 수 있지만 매우 위험한 정책이다.61
- **SARSA의 행동 (온-폴리시):** SARSA는 자신의 ε-탐욕 행동 정책의 가치를 학습한다. 절벽 근처에 있으면 ϵ 확률로 잘못된 행동을 선택하여 절벽으로 떨어질 수 있다는 것을 경험을 통해 학습한다. 따라서 절벽 근처의 상태-행동 쌍의 Q-값이 낮게 평가된다. 결과적으로 SARSA는 자신의 탐험으로 인한 위험을 최소화하기 위해, 약간의 보상을 희생하더라도 절벽에서 멀리 떨어진 안전한 우회로를 선택하는 정책을 학습한다.61

| 특징               | Q-러닝 (Q-Learning)                                          | SARSA                                                |
| ------------------ | ------------------------------------------------------------ | ---------------------------------------------------- |
| **정책 유형**      | 오프-폴리시 (Off-policy) 56                                  | 온-폴리시 (On-policy) 56                             |
| **업데이트 목표**  | r+γmaxa′Q(s′,a′)                                             | r+γQ(s′,a′)                                          |
| **학습 대상 가치** | 최적 (탐욕적) 정책의 가치 62                                 | 현재 행동 정책 (예: ε-탐욕)의 가치 62                |
| **절벽 보행 행동** | 최적이지만 위험한 경로 학습 61                               | 차선책이지만 안전한 경로 학습 61                     |
| **장점**           | 탐험적 행동을 하면서도 최적 정책 학습 가능, 데이터 재사용성 높음 57 | 학습이 더 안정적이고 수렴성이 좋음 61                |
| **단점**           | Q-값 과대평가 경향, 함수 근사 시 불안정성 증가 56            | 샘플 효율성이 낮음 (과거 정책의 데이터 사용 불가) 68 |

Q-러닝의 오프-폴리시 특성은 양날의 검과 같다. 이는 Q-러닝의 가장 큰 강점이자 동시에 불안정성의 주요 원인이 된다. 한편으로, 행동 정책과 타겟 정책을 분리함으로써 에이전트는 자유롭게 탐험하면서도 최적 정책을 학습할 수 있다. 이는 심지어 다른 에이전트나 인간이 생성한 과거 데이터로부터 학습하는 것을 가능하게 하여 데이터 효율성을 극대화한다.57

그러나 다른 한편으로, 업데이트 규칙의 max 연산자는 Q-값의 양의 오차를 증폭시켜 시스템적으로 Q-값을 과대평가하는 **최대화 편향(maximization bias)**을 유발한다.56 이 문제는 함수 근사기(function approximator)와 결합될 때 더욱 심각해진다. 오프-폴리시 학습, 부트스트래핑(bootstrapping, 추정치로 다른 추정치를 업데이트하는 방식), 함수 근사는 강화학습에서 불안정성과 발산을 유발하는 **"죽음의 삼각편대(the deadly triad)"**로 알려져 있다.30 Q-러닝은 이 세 가지 요소를 모두 포함하고 있어, 단순한 신경망과 결합했을 때 학습이 실패하기 쉽다.67 바로 이 본질적인 불안정성 때문에 심층 Q-네트워크(DQN)의 개발 과정에서 경험 리플레이와 타겟 네트워크 같은 안정화 기법의 발명이 필수적이었다. 이 기법들은 Q-러닝의 오프-폴리시 특성에서 비롯되는 문제점들을 직접적으로 해결하기 위해 고안된 것이다.70

------



## 제 3부: 심층 Q-네트워크(DQN)의 등장



고전적 Q-러닝은 강화학습의 이론적 토대를 마련했지만, Q-테이블의 한계로 인해 복잡한 현실 문제에 적용하기 어려웠다. 이 장에서는 Q-러닝의 한계를 극복하고 심층 강화학습 시대를 연 심층 Q-네트워크(DQN)의 탄생 배경과 핵심 혁신을 다룬다.



### 제 3.1절: 테이블 기반 Q-러닝의 한계



테이블 기반 Q-러닝은 명확한 한계를 가지고 있으며, 이는 주로 '차원의 저주' 문제로 요약된다.

- **차원의 저주 (Curse of Dimensionality):** 이 문제는 Q-테이블의 크기가 상태와 행동 공간의 크기에 따라 기하급수적으로 증가하는 현상을 말한다.71 상태를 정의하는 변수가 하나 추가될 때마다 필요한 테이블의 크기가 배로 늘어난다. 예를 들어, 비디오 게임의 화면 픽셀이나 로봇의 연속적인 센서 값과 같이 고차원적인 상태 공간을 갖는 문제에서는, 모든 상태-행동 쌍을 저장할 Q-테이블을 만드는 것이 메모리 및 계산 자원 측면에서 불가능하다.2 84x84 픽셀의 흑백 게임 화면만 하더라도 상태의 경우의 수는 천문학적인 숫자에 달해 테이블 방식은 적용할 수 없다.74
- **연속 공간 처리의 어려움:** Q-테이블은 본질적으로 이산적인(discrete) 상태와 행동 공간을 가정한다. 따라서 로봇 팔의 관절 각도나 자동차의 가속도와 같은 연속적인(continuous) 상태나 행동을 직접적으로 다룰 수 없다.31 연속 공간을 이산적인 구간으로 나누는 '양자화(quantization)'나 '비닝(binning)' 같은 기법을 사용할 수 있지만, 이는 근본적인 해결책이 되지 못한다.77 구간을 너무 넓게 잡으면 중요한 상태 정보를 잃게 되고, 너무 잘게 나누면 다시 차원의 저주 문제에 직면하게 된다.80
- **일반화 능력의 부재:** 테이블 기반 방법은 각 상태-행동 쌍의 가치를 독립적으로 학습한다. 이는 한 상태에서 얻은 지식을 이와 유사하지만 이전에 경험하지 못한 다른 상태에 적용하는, 즉 **일반화(generalization)**하는 능력이 없음을 의미한다. 에이전트는 모든 상태를 최소 한 번 이상 방문해야만 해당 상태에 대한 가치를 학습할 수 있으며, 이는 매우 비효율적인 학습 방식이다.81



### 제 3.2절: 심층 Q-네트워크(DQN): Q-러닝과 딥러닝의 만남



이러한 한계를 극복하기 위한 돌파구는 Q-함수를 근사하기 위해 심층 신경망(Deep Neural Network)을 사용하는 아이디어에서 나왔다.

- **핵심 아이디어:** Q-테이블 대신, 심층 신경망을 강력한 **함수 근사기(function approximator)**로 사용하여 Q-함수를 추정한다: Q(s,a;θ)≈Q∗(s,a). 여기서 θ는 신경망의 학습 가능한 가중치(weights)를 의미한다.32 이 신경망을 **심층 Q-네트워크(Deep Q-Network, DQN)**라고 부른다.

- **딥마인드의 혁신:** 2013년과 2015년, 딥마인드(DeepMind) 연구팀은 DQN을 이용하여 별도의 사전 지식 없이 오직 게임 화면의 원본 픽셀 데이터만을 입력으로 받아 다양한 아타리 2600 비디오 게임에서 인간 전문가 수준을 뛰어넘는 성능을 달성했다고 발표했다.31 이는 단일한 범용 알고리즘이 고차원 감각 입력으로부터 복잡한 제어 정책을 스스로 학습할 수 있음을 증명한 인공지능 역사상 중요한 사건이었다.

- **DQN 아키텍처:**

  - **입력:** 현재 상태를 나타내는 원본 데이터. 아타리 게임의 경우, 움직임을 포착하기 위해 최근 4개의 게임 프레임을 흑백으로 변환하고 크기를 조정한 이미지를 쌓아서 입력으로 사용한다.75
  - **네트워크:** 입력된 이미지의 공간적 특징을 추출하기 위해 **합성곱 신경망(Convolutional Neural Network, CNN)**을 사용하고, 그 뒤에 완전 연결 계층(fully-connected layers)을 두어 최종 Q-값을 계산한다.88
  - **출력:** 네트워크는 주어진 상태 s에 대해 가능한 모든 이산적 행동 각각에 대한 Q-값들을 벡터 형태로 출력한다.90 이 구조는 상태와 행동을 모두 입력으로 받는 것보다 효율적인데, 한 번의 순전파(forward pass)만으로 모든 행동의 Q-값을 얻을 수 있기 때문이다.

- **손실 함수 (Loss Function):** DQN은 지도학습과 유사한 방식으로 훈련된다. 네트워크가 예측한 Q-값 $Q(s, a; \theta)$과 TD 타겟 y 사이의 **평균 제곱 오차(Mean Squared Error, MSE)**를 손실 함수로 정의하고, 이를 최소화하는 방향으로 경사 하강법(gradient descent)을 통해 네트워크 가중치 θ를 업데이트한다.

  - 손실 함수: L(θ)=E(s,a,r,s′)∼D[(y−Q(s,a;θ))2]

  - TD 타겟: y=r+γmaxa′Q(s′,a′;θtarget)

    여기서 D는 경험 데이터셋을, $\theta_{\text{target}}$은 타겟 네트워크의 가중치를 의미한다.32



### 제 3.3절: 심층 강화학습의 안정화



단순히 Q-테이블을 신경망으로 대체하는 것은 이론적으로는 간단해 보이지만, 실제로는 학습이 매우 불안정하여 실패하는 경우가 많다.70 이는 앞서 언급한 "죽음의 삼각편대"(함수 근사, 부트스트래핑, 오프-폴리시 학습의 조합) 때문이다.30 DQN은 이 문제를 해결하기 위해 두 가지 핵심적인 기법을 도입했다.

**기법 1: 경험 리플레이 (Experience Replay)**

- **작동 방식:** 에이전트는 환경과 상호작용하며 얻은 경험 데이터, 즉 전이(transition) 튜플 $(s, a, r, s', \text{done})$을 시간 순서대로 즉시 학습에 사용하는 대신, **리플레이 버퍼(replay buffer)** 또는 **리플레이 메모리(replay memory)**라는 큰 크기의 버퍼에 저장한다.83 네트워크를 훈련시킬 때는 이 버퍼에서 무작위로 미니배치(mini-batch)를 샘플링하여 사용한다.92
- **중요성 및 장점:**
  1. **시간적 상관관계 제거:** 강화학습에서 연속적으로 발생하는 경험들은 시간적으로 매우 높은 상관관계를 갖는다. 이는 신경망 학습의 기본 가정인 데이터의 독립 항등 분포(Independent and Identically Distributed, I.I.D.) 가정을 위배하여 학습을 불안정하게 만든다. 리플레이 버퍼에서 무작위로 샘플링함으로써 이러한 상관관계를 깨고, 데이터를 I.I.D.에 가깝게 만들어 학습 안정성을 크게 향상시킨다.70
  2. **데이터 샘플 효율성 향상:** 하나의 경험 데이터가 여러 번의 학습 업데이트에 재사용될 수 있다. 이를 통해 에이전트는 한 번의 상호작용으로부터 더 많은 정보를 추출할 수 있으며, 이는 실제 환경과의 상호작용 비용이 비싼 경우에 특히 중요하다.91
  3. **학습 과정 평탄화:** 다양한 시점의 경험들로 구성된 미니배치를 사용하여 그래디언트를 계산하면, 단일 경험에 의한 그래디언트의 분산이 줄어들어 정책의 급격한 변화를 막고 학습 과정을 더 부드럽게 만든다.75

**기법 2: 타겟 네트워크 (Target Network)**

- **작동 방식:** Q-값을 예측하는 주 네트워크(온라인 네트워크)와는 별개로, 구조는 동일하지만 가중치(θ−)가 다른 **타겟 네트워크**를 하나 더 사용한다.83 TD 타겟 $y = r + \gamma \max_{a'} Q(s', a'; \theta^-)$를 계산할 때 이 타겟 네트워크를 사용한다. 온라인 네트워크의 가중치 

  θ는 매 스텝 업데이트되지만, 타겟 네트워크의 가중치 $\theta^-$는 한동안 고정되어 있다가 일정 주기마다 온라인 네트워크의 가중치로 복사된다 ($\theta^- \leftarrow \theta$).

- **중요성 및 장점:**

  1. **안정적인 타겟 제공:** 만약 타겟 네트워크가 없다면, TD 타겟을 계산하는 데 사용되는 Q-값과 업데이트 대상이 되는 Q-값이 동일한 네트워크에 의해 계산된다. 이는 온라인 네트워크의 가중치가 업데이트될 때마다 타겟 값도 함께 흔들리는 "움직이는 목표물(moving target)" 문제를 야기한다. 에이전트가 자신의 꼬리를 쫓는 것과 같은 이 상황은 학습의 진동이나 발산으로 이어진다. 타겟 네트워크는 일정 기간 동안 타겟 값을 고정시켜 줌으로써 학습 목표를 안정시키고, 이를 통해 수렴을 돕는다.95
  2. **업데이트와 타겟 생성의 분리:** 이 기법은 업데이트되는 Q-값과 타겟 계산에 사용되는 Q-값을 분리(decouple)하여, 두 값 사이의 강한 상관관계를 끊어준다. 이것이 학습 불안정성을 줄이는 핵심 원리이다.95

경험 리플레이와 타겟 네트워크는 단순히 DQN을 위한 부수적인 기법이 아니다. 이들은 부트스트래핑과 오프-폴리시 데이터를 사용하는 모든 가치 기반 심층 강화학습 알고리즘의 안정성을 확보하기 위한 근본적인 원칙을 제시한다. 이 두 기법의 영향력은 DQN을 넘어, 이후에 개발된 Double DQN, Dueling DQN, 심지어 DDPG와 같은 액터-크리틱 방법에까지 이어지며 심층 강화학습의 표준적인 구성 요소로 자리 잡았다.98 이는 특정 알고리즘의 성공이 단지 새로운 구조의 발명에만 있는 것이 아니라, 학습 과정의 본질적인 불안정성을 이해하고 이를 제어하는 원리를 발견하는 데 있음을 보여준다.

------



## 제 4부: 고급 방법론과 DQN Zoo



DQN의 성공은 수많은 후속 연구를 촉발시켰다. 연구자들은 DQN의 특정 약점들을 식별하고 이를 해결하기 위한 다양한 개선안을 제시했으며, 이는 마치 동물원처럼 다양한 변종 알고리즘들, 즉 'DQN Zoo'를 형성하게 되었다. 이 장에서는 그 중 가장 중요하고 영향력 있는 방법론들을 살펴본다.



### 제 4.1절: Double DQN (DDQN): 과대평가 편향 해결



- **문제점:** 표준 DQN의 TD 타겟 계산에 포함된 max 연산자는 Q-값을 시스템적으로 과대평가(overestimation)하는 경향이 있다.98 이는 Q-값을 추정하는 데 사용되는 동일한 네트워크가 최선의 행동을 '선택'하고 그 행동의 가치를 '평가'하는 두 가지 역할을 모두 수행하기 때문이다. 만약 특정 행동의 Q-값 추정치에 양의 노이즈나 오차가 있다면, 

  max 연산자는 그 행동을 선택할 가능성이 높고, 이로 인해 부풀려진 Q-값이 타겟으로 사용되어 전체적인 과대평가 편향이 누적된다.

- **해결책:** **Double DQN (DDQN)**은 이 문제를 해결하기 위해 행동의 '선택'과 '평가'를 분리한다.98 구체적으로, 다음 상태에서 최선의 행동을 선택할 때는 현재 학습 중인 **온라인 네트워크(

  θ)**를 사용하고, 선택된 그 행동의 가치를 평가할 때는 안정적인 **타겟 네트워크(θ−)**를 사용한다.

  - **표준 DQN 타겟:** yDQN=r+γmaxa′Q(s′,a′;θ−)
  - **Double DQN 타겟:** yDDQN=r+γQ(s′,a′argmaxQ(s′,a′;θ);θ−)

  이 작은 변화는 온라인 네트워크가 제안한 최선의 행동을 타겟 네트워크가 한 번 더 검증하는 효과를 낳는다. 이로써 온라인 네트워크의 일시적인 오차로 인한 과대평가가 타겟 값에 그대로 반영되는 것을 막아주어, 결과적으로 더 안정적인 학습과 우수한 성능의 정책을 유도한다.101



### 제 4.2절: Dueling DQN: Q-함수의 분해



- **문제점:** 많은 상태에서, 그 상태 자체의 가치(V(s))는 중요하지만 어떤 행동을 선택하는지는 결과에 큰 영향을 미치지 않을 수 있다. 예를 들어, 운전 게임에서 전방에 아무런 장애물이 없다면, 직진하는 여러 행동들은 거의 유사한 가치를 가질 것이다. 표준 DQN은 이러한 상황에서도 각 행동에 대한 Q-값을 개별적으로 모두 학습해야 하므로 비효율적이다.

- **해결책:** **Dueling DQN**은 Q-함수를 두 개의 요소, 즉 상태 자체의 가치를 나타내는 **상태 가치 함수(V(s))**와, 각 행동이 평균적인 행동보다 얼마나 더 나은지를 나타내는 **어드밴티지 함수(A(s,a))**로 분해하는 독특한 네트워크 아키텍처를 제안한다.98

  Q(s,a)=V(s)+A(s,a)

  네트워크는 공유된 특징 추출부(예: CNN)를 거친 후, 두 개의 스트림(stream) 또는 헤드(head)로 나뉜다. 하나는 스칼라 값인 $V(s)$를 출력하고, 다른 하나는 각 행동에 대한 어드밴티지 값을 담은 벡터 $A(s, a)$를 출력한다.99

  이때 V와 A가 Q로부터 유일하게 결정되도록 하기 위해(identifiability), 어드밴티지 함수에 제약을 가한다. 일반적으로 어드밴티지 값의 평균을 빼주는 방식을 사용한다 99:

  Q(s,a)=V(s)+(A(s,a)−∣A∣1a′∑A(s,a′))

- **장점:** 이 구조를 통해 네트워크는 상태의 가치를 행동과 독립적으로 학습할 수 있다.106 이는 상태의 가치 평가가 중요한 많은 문제에서 학습을 일반화하고 가속화하며, 특히 행동 공간이 큰 환경에서 더 나은 성능을 보인다.99



### 제 4.3절: 우선순위 경험 리플레이 (PER)



- **문제점:** 표준 경험 리플레이는 리플레이 버퍼에서 모든 경험을 균등한 확률로 샘플링한다.94 하지만 모든 경험이 학습에 동일하게 중요한 것은 아니다. 에이전트는 자신의 예측이 크게 빗나간 "놀라운" 경험으로부터 더 많은 것을 배울 수 있다.
- **해결책:** **우선순위 경험 리플레이(Prioritized Experience Replay, PER)**는 학습에 더 중요하다고 판단되는 경험을 더 높은 확률로 샘플링하는 기법이다.109
  - **우선순위 척도:** 경험의 중요도는 일반적으로 **TD 오차(δ)**의 절댓값에 비례하여 결정된다: pi∝∣δi∣.110 TD 오차가 크다는 것은 네트워크의 예측이 실제와 크게 달랐다는 의미이므로, 해당 경험은 학습에 매우 중요하다.
  - **편향 보정:** 이처럼 우선순위에 따라 샘플링하면 학습 데이터의 분포에 편향(bias)이 생긴다. 이 편향을 보정하기 위해 **중요도 샘플링(Importance Sampling, IS)** 가중치를 사용하여, 우선순위가 높은 샘플의 그래디언트 업데이트 영향력을 낮추어 줌으로써 학습이 편향되지 않도록 조정한다.109
- **장점:** 가장 정보 가치가 높은 샘플에 집중함으로써, PER는 학습 과정을 훨씬 더 효율적으로 만들고, 수렴 속도를 높이며 전반적인 성능을 개선한다.109



### 제 4.4절: Rainbow DQN과 현대 강화학습으로의 길



- **Rainbow DQN:** 이 알고리즘은 새로운 단일 아이디어가 아니라, 앞서 설명한 여러 개선안들을 통합한 것이다. 구체적으로 Double DQN, Dueling DQN, PER, 다중-스텝 학습(Multi-step Learning), 분포 강화학습(Distributional RL), 노이지 넷(Noisy Nets) 등을 하나의 에이전트에 결합했다.104 Rainbow 논문은 이러한 개별적인 개선안들이 서로 상호보완적이며, 함께 사용될 때 시너지를 내어 아타리 벤치마크에서 당시 최고 수준의 성능을 달성함을 보여주었다.

- DQN을 넘어서: 연속적 행동을 위한 액터-크리틱 방법:

  DQN과 그 변종들은 강력하지만, max 연산자로 인해 근본적으로 이산적인(discrete) 행동 공간에만 적용 가능하다는 한계가 있다. 연속적인 공간에서 $\underset{a}{\operatorname{argmax}} Q(s, a)$를 찾는 것은 현실적으로 불가능하기 때문이다.78

  **액터-크리틱(Actor-Critic, AC)** 방법은 이 문제를 해결하기 위해 두 개의 분리된 네트워크를 학습시킨다.

  - **액터 (Actor):** 정책 네트워크 $\pi(a|s; \theta)$로, 주어진 상태에서 직접 행동(또는 행동에 대한 확률 분포)을 출력한다.120
  - **크리틱 (Critic):** 가치 네트워크로, 액터가 선택한 행동을 "평가(critique)"하기 위해 가치 함수(V(s) 또는 Q(s,a))를 추정한다.120

  크리틱의 평가(예: TD 오차 또는 어드밴티지)는 정책 그래디언트(policy gradient)를 계산하는 데 사용되어 액터의 파라미터를 업데이트하며, 이를 통해 액터가 더 나은 행동을 생성하도록 유도한다.120 DDPG, A3C, SAC와 같은 액터-크리틱 계열의 알고리즘들은 로보틱스 등 연속적인 제어 문제의 표준적인 해법으로 자리 잡았다.120

DQN에서 Rainbow로 이어지는 발전 과정은 단일 알고리즘의 약점을 체계적으로 식별하고, 각 약점을 해결하기 위한 독립적이고 직교적인(orthogonal) 개선안을 개발한 뒤, 이를 다시 통합하는 연구 패러다임을 보여준다. 이는 복잡한 시스템을 개선하는 데 있어, 완전히 새로운 거대 알고리즘을 처음부터 발명하려는 시도보다 특정 약점을 표적화하여 수정하고 통합하는 모듈식 접근법이 얼마나 효과적인지를 입증한다. 이러한 점진적 개선의 방법론은 현대 심층 강화학습 연구의 핵심적인 흐름 중 하나가 되었다.

------



## 제 5부: 실제 적용 및 사례 연구



이 장에서는 이론적 논의를 구체적인 예제에 적용하여, Q-러닝이 다양한 분야에서 실제로 어떻게 활용되는지를 살펴본다. 각 분야의 문제를 MDP로 어떻게 정의하는지가 핵심이다.



### 제 5.1절: 게임 AI



**그리드월드(Gridworld) 단계별 학습 과정**

그리드월드는 Q-러닝의 작동 원리를 직관적으로 이해하는 데 가장 널리 사용되는 예제이다.122

- **MDP 정의:**
  - **상태 (State):** 에이전트의 그리드 상 (x, y) 좌표.
  - **행동 (Action):** {상, 하, 좌, 우}.
  - **보상 (Reward):** 목표 지점 도달 시 +10, 장애물 충돌 시 -1, 그 외 매 스텝마다 -0.1 (효율적인 경로를 장려하기 위함).
- **학습 과정 추적:**
  1. **초기화:** 모든 상태-행동 쌍의 Q-값이 0으로 채워진 Q-테이블로 시작한다.17
  2. **초기 탐험:** 에이전트는 무작위로 움직이며 환경을 탐험한다. 대부분의 행동은 -0.1의 보상을 받으므로, 해당 Q-값들은 점차 음수로 업데이트된다.
  3. **보상의 역전파:** 에이전트가 우연히 목표 지점에 도달하면, 목표 지점으로 들어가는 행동(예: 상태 (x, y-1)에서 '우'로 이동)에 대한 Q-값이 처음으로 큰 양수(−0.1+γ×10)로 업데이트된다.17
  4. **가치 전파:** 다음 에피소드에서 에이전트가 목표 지점 한 칸 전 상태에 도달하면, 이제 그 상태의 Q-값은 양수가 된다. 에이전트가 그 상태의 두 칸 전 상태에 도달하면, 그곳의 Q-값 역시 다음 상태의 양수 Q-값을 참조하여 업데이트된다. 이런 식으로 목표 지점의 긍정적 가치가 마치 물결처럼 그리드 전체로 점차 "역전파(backpropagate)"된다.17
  5. **수렴:** 수많은 에피소드가 반복되면, Q-테이블에는 목표 지점으로 향하는 일종의 "가치 경사(value gradient)"가 형성된다. 에이전트는 이제 어떤 상태에 있더라도 Q-값이 가장 높은 방향을 따라감으로써 최적의 경로를 찾을 수 있게 된다.

**2인용 게임 (예: 틱택토)**

- **도전 과제:** 상대방 또한 학습하는 에이전트일 경우, 환경이 비정상적(non-stationary)이 되어 학습이 불안정해진다.
- **단순화된 접근 (무작위 에이전트 상대):** 만약 상대방이 학습하지 않고 고정된 정책(예: 무작위)을 따른다면, 상대방의 행동은 환경의 확률적 일부로 간주할 수 있다.125
- MDP 정의 125:
  - **상태:** 내 에이전트가 수를 둘 차례일 때의 게임 보드판의 배치. 상대방의 턴은 환경의 상태 전이 과정에 포함된다.
  - **행동:** 비어있는 칸에 내 마커를 놓는 것.
  - **보상:** 승리 시 +1, 패배 시 -1, 무승부 시 0. 빠른 승리를 유도하기 위해 매 수마다 작은 음의 보상을 줄 수도 있다.



### 제 5.2절: 로보틱스와 자율 시스템



로봇 내비게이션 13

- **목표:** 정적 및 동적 장애물을 회피하며 목표 지점까지 이동한다.
- MDP 정의 13:
  - **상태:** 원본 센서 데이터를 벡터화하여 사용한다. 이는 라이다(LiDAR) 센서가 감지한 여러 각도의 장애물까지의 거리, 카메라로 인식한 목표까지의 거리 및 각도, 로봇의 현재 속도 등을 포함할 수 있다.13 이러한 고차원 연속적 상태는 DQN과 같은 함수 근사기를 필요로 한다.
  - **행동:** {전진, 좌회전 45도, 우회전 45도, 정지}와 같이 이산화된 모터 제어 명령어 집합.13
  - **보상 함수 (보상 설계, Reward Shaping):** 성공적인 학습을 위해 보상 설계가 매우 중요하다.
    - 목표 지점에 도달 시 큰 양의 보상 (+100).13
    - 장애물과 충돌 시 큰 음의 보상 (-100).13
    - 목표에 가까워질 때 작은 양의 보상 (거리 보상 Rd).13
    - 로봇이 목표를 향하고 있을 때 작은 양의 보상 (각도 보상 Rθ).13
    - 시간이 지날 때마다 작은 음의 보상을 주어 효율적인 경로를 장려.



### 제 5.3절: 금융 및 추천 시스템



알고리즘 트레이딩 14

- **목표:** 포트폴리오 가치를 극대화하는 거래 전략을 학습한다.
- MDP 정의 14:
  - **상태:** 과거 가격 변동, 이동 평균, RSI와 같은 기술적 지표, 현재 보유 자산, 현금 잔고 등을 포함하는 시장 지표 벡터.
  - **행동:** {X% 매수, Y% 매도, 보유}와 같은 이산적인 거래 결정.
  - **보상:** 각 타임 스텝에서의 포트폴리오 총 가치의 변화량.

동적 추천 시스템 31

- **목표:** 사용자에게 아이템 순서를 추천하여 장기적인 사용자 참여(예: 총 클릭률, 시청 시간)를 극대화한다.
- MDP 정의 130:
  - **상태:** 사용자의 최근 상호작용 이력을 나타내는 표현 (예: 최근 클릭/시청한 N개 아이템의 임베딩 벡터).
  - **행동:** 상품 카탈로그에서 특정 아이템을 추천하는 것.
  - **보상:** 사용자가 추천된 아이템을 클릭/소비하면 +1, 그렇지 않으면 0. 더 복잡하게는 체류 시간이나 구매 전환 여부를 보상에 포함시킬 수 있다.131 핵심은 단기적인 클릭이 아닌 장기적인 보상을 목표로 한다는 점이다.

로보틱스나 금융과 같은 복잡한 실제 응용 분야에서는 보상 신호가 매우 희소(sparse)하거나(예: 로봇이 임무의 맨 마지막에만 보상을 받음) 진정한 목표와 완벽하게 일치하지 않는 경우가 많다. 이러한 상황에서 학습을 효과적으로 유도하기 위해 중간 보상을 설계하는 **보상 설계(Reward Shaping)**는 Q-러닝을 성공적으로 적용하는 데 있어 가장 중요하고 어려운 부분 중 하나가 된다.13

예를 들어, 로봇 내비게이션에서 최종 목적지 도달 보상만으로는 학습이 너무 느리기 때문에, 엔지니어는 목표와의 거리가 줄어들 때마다 작은 양의 보상을 추가하여 학습을 안내해야 한다.13 하지만 이 과정은 매우 섬세한 작업이다. 잘못 설계된 보상 함수는 에이전트가 의도치 않은 행동을 학습하게 만드는 **'보상 해킹(reward hacking)'**으로 이어질 수 있다.134 예를 들어, 로봇이 목표 지점 근처에서 계속 빙빙 돌면서 '가까워짐' 보상을 반복적으로 얻으려 할 뿐, 실제로는 목표에 도달하지 않는 행동을 학습할 수 있다. 따라서 Q-러닝 알고리즘 자체는 범용적이지만, 복잡한 도메인에서의 성공적인 적용은 상태 공간에 대한 효과적인 특징 공학(feature engineering)과 신중하고 반복적인 보상 함수 설계에 크게 의존하며, 이는 실용화의 주요 병목 지점 중 하나이다.

------



## 제 6부: 결론 및 향후 전망



본 보고서는 Q-러닝의 이론적 기초부터 현대 심층 강화학습으로의 발전, 그리고 다양한 실제 적용 사례에 이르기까지 포괄적인 분석을 제공했다. 이 마지막 장에서는 핵심적인 내용을 종합하고, 현재의 도전 과제와 미래 연구 방향을 조망한다.



### 제 6.1절: 핵심 내용 종합



Q-러닝은 이론적으로 잘 정립된 테이블 기반 알고리즘에서 출발하여 2, 현대 심층 강화학습의 근간을 이루는 핵심 구성 요소로 진화했다.86 그 여정의 중심에는 다음과 같은 핵심 원리들이 자리 잡고 있다.

- **근본 원리:** Q-러닝의 목표는 시간차 학습(TD learning)을 통해 벨만 최적 방정식을 푸는 것이며, 이는 에이전트가 누적 보상을 최대화하는 최적의 행동 가치 함수를 학습하도록 이끈다.15
- **핵심 딜레마:** 모든 학습 과정에서 '탐험과 활용' 사이의 균형을 맞추는 것은 최적 정책을 발견하기 위한 필수적인 과제이다.41
- **오프-폴리시의 힘:** Q-러닝의 오프-폴리시 특성은 실제 행동과 무관하게 최적 정책을 학습할 수 있는 유연성을 제공하지만, 동시에 학습 불안정성의 원인이 되기도 한다.55
- **진화의 패러다임:** DQN의 등장은 Q-러닝의 차원의 저주 한계를 극복했으며, 이후 Double DQN, Dueling DQN, PER 등으로 이어지는 'DQN Zoo'는 복잡한 시스템의 특정 약점을 식별하고 이를 해결하는 점진적이고 모듈적인 개선 방식의 성공 사례를 보여주었다.104



### 제 6.2절: 현재의 도전 과제와 미래 연구 방향



Q-러닝과 심층 강화학습은 눈부신 발전을 이루었지만, 여전히 해결해야 할 중요한 도전 과제들이 남아있다.

- **샘플 효율성 (Sample Efficiency):** 경험 리플레이를 사용하더라도 심층 강화학습 방법들은 여전히 수백만 번의 상호작용을 요구할 정도로 데이터 집약적이다.94 이는 실제 로봇이나 고비용 환경에 적용하는 데 큰 장벽이 된다. 이 문제를 해결하기 위해, 환경의 동역학 모델을 학습하여 시뮬레이션된 경험을 생성하고 이를 통해 학습을 가속하는 **모델 기반 강화학습(Model-Based RL)**이 활발히 연구되고 있다. 

  **Dyna-Q**는 이러한 모델 기반 접근법과 모델 프리(model-free) Q-러닝을 결합한 초기 하이브리드 알고리즘의 대표적인 예이다.136

- **일반화와 전이 학습 (Generalization and Transfer Learning):** 특정 작업에 대해 훈련된 에이전트는 환경이 조금만 바뀌어도 성능이 급격히 저하되는 경향이 있다.134 한 도메인에서 학습한 지식을 새로운 도메인에 효과적으로 이전하는 전이 학습 및 메타 학습(meta-learning)은 강화학습의 실용성을 높이기 위한 핵심 연구 분야이다.

- **안전성과 신뢰성 (Safety and Reliability):** 자율 주행 자동차나 의료 로봇과 같이 실패 비용이 매우 큰 실제 환경에서 강화학습 에이전트가 예측 가능하고 안전하게 행동하도록 보장하는 것은 매우 중요한 미해결 과제이다.134 예기치 않은 상황에 대한 강건성(robustness)을 확보하고, 에이전트의 의사결정 과정을 해석하려는 연구가 진행 중이다.

- **다중 에이전트 강화학습 (Multi-Agent RL, MARL):** 현실 세계의 많은 문제는 여러 에이전트가 상호작용하는 환경을 포함한다. 각 에이전트의 입장에서 다른 에이전트들은 환경의 일부이며, 이들의 정책이 변함에 따라 환경이 비정상적(non-stationary)이 된다. 이러한 복잡한 동역학 속에서 협력 또는 경쟁하며 최적의 공동 정책을 학습하는 다중 에이전트 Q-러닝 및 관련 알고리즘 연구는 매우 활발한 분야이다.7

결론적으로, Q-러닝은 강화학습 분야의 이론적 근간을 제공했을 뿐만 아니라, 딥러닝과의 결합을 통해 인공지능의 가능성을 한 단계 끌어올린 기념비적인 알고리즘이다. 위에 제시된 도전 과제들을 해결하기 위한 지속적인 연구는 앞으로 Q-러닝 기반의 강화학습 기술이 더욱 정교하고 강력하며 신뢰할 수 있는 방향으로 발전해 나갈 것임을 시사한다.

#### **참고 자료**

1. www.mindscale.kr, accessed July 10, 2025, [https://www.mindscale.kr/docs/reinforcement-learning/q-learning#:~:text=Q%2D%EB%9F%AC%EB%8B%9D%EC%9D%B4%EB%9E%80%20%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80,%22%EA%B0%80%EC%B9%98%22%EB%A5%BC%20%EC%9D%98%EB%AF%B8%ED%95%A9%EB%8B%88%EB%8B%A4.](https://www.mindscale.kr/docs/reinforcement-learning/q-learning#:~:text=Q-러닝이란 무엇인가,"가치"를 의미합니다.)
2. [강화학습] Q-러닝 :: 마인드스케일, accessed July 10, 2025, https://www.mindscale.kr/docs/reinforcement-learning/q-learning
3. Simulation of the navigation of a mobile robot by the Q- Learning using artificial neuron networks - CEUR-WS.org, accessed July 10, 2025, https://ceur-ws.org/Vol-547/81.pdf
4. 1. 강화학습의 개요 - JUST CODE IT - 티스토리, accessed July 10, 2025, https://jdjin3000.tistory.com/2
5. Reinforcement Learning: An Introduction by Richard S. Sutton | Goodreads, accessed July 10, 2025, https://www.goodreads.com/book/show/739791.Reinforcement_Learning
6. [AI] 기초 개념 학습 #2(강화 학습) - 메모장 - 티스토리, accessed July 10, 2025, https://myungjjju.tistory.com/280
7. 강화학습 핵심 개념 정리 (1), accessed July 10, 2025, https://infossm.github.io/blog/2019/12/15/RL-key-concepts/
8. brunch.co.kr, accessed July 10, 2025, [https://brunch.co.kr/@brunchflgu/1892#:~:text=%EC%97%90%EC%9D%B4%EC%A0%84%ED%8A%B8%2C%20%ED%99%98%EA%B2%BD%2C%20%EC%83%81%ED%83%9C%2C%20%EB%B3%B4%EC%83%81,%EA%B2%B0%EA%B3%BC%EB%A1%9C%EC%84%9C%20%EC%A3%BC%EC%96%B4%EC%A7%80%EB%8A%94%20%ED%94%BC%EB%93%9C%EB%B0%B1%EC%9D%B4%EB%8B%A4.](https://brunch.co.kr/@brunchflgu/1892#:~:text=에이전트%2C 환경%2C 상태%2C 보상,결과로서 주어지는 피드백이다.)
9. Bellman Optimality Equation in Reinforcement Learning - Analytics Vidhya, accessed July 10, 2025, https://www.analyticsvidhya.com/blog/2021/02/understanding-the-bellman-optimality-equation-in-reinforcement-learning/
10. [머신러닝 - 이론] 강화 학습의 기초 (Fundamental of Reinforcement Learning) - 안녕, accessed July 10, 2025, https://hi-guten-tag.tistory.com/86
11. Q: 강화 학습이란 무엇인가요? - AWS, accessed July 10, 2025, https://aws.amazon.com/ko/what-is/reinforcement-learning/
12. 강화학습 주요 개념 - velog, accessed July 10, 2025, [https://velog.io/@kunha98/%EA%B0%95%ED%99%94%ED%95%99%EC%8A%B5-%EC%A3%BC%EC%9A%94-%EA%B0%9C%EB%85%90](https://velog.io/@kunha98/강화학습-주요-개념)
13. Mobile Robot Navigation Using Deep Reinforcement Learning - MDPI, accessed July 10, 2025, https://www.mdpi.com/2227-9717/10/12/2748
14. xinyi6666/stock_trading - GitHub, accessed July 10, 2025, https://github.com/xinyi6666/stock_trading
15. (3) 가치함수와 벨만방정식 - Jang. Inspiration, accessed July 10, 2025, https://jang-inspiration.com/bellman-equation
16. Understanding the Bellman Equation in Reinforcement Learning - DataCamp, accessed July 10, 2025, https://www.datacamp.com/tutorial/bellman-equation-reinforcement-learning
17. [강화학습] Q 러닝 이해하기 - Atom's Space, accessed July 10, 2025, https://spacebike.tistory.com/53
18. [Q-learning] 데이터로 설명하기 - 상황파악, accessed July 10, 2025, https://otch80.tistory.com/56
19. What Is the Bellman Operator in Reinforcement Learning? | Baeldung on Computer Science, accessed July 10, 2025, https://www.baeldung.com/cs/bellman-operator-reinforcement-learning
20. 강화 학습 기초 - 정책(Policy), 가치 함수(Value Function) 그리고 벨만 방정식(Bellman Equation) - 공부Dragon - 티스토리, accessed July 10, 2025, https://wnthqmffhrm.tistory.com/5
21. 벨만 방정식 - 성장通 - 티스토리, accessed July 10, 2025, https://kevin-rain.tistory.com/78
22. Bellman Equation and Contraction Mapping, accessed July 10, 2025, https://teazrq.github.io/stat546/notes/contraction-mapping.html
23. What is the Bellman operator in reinforcement learning? - AI Stack Exchange, accessed July 10, 2025, https://ai.stackexchange.com/questions/11057/what-is-the-bellman-operator-in-reinforcement-learning
24. Proof of Convergence of the Bellman Operator - Tuan Anh Le, accessed July 10, 2025, https://www.tuananhle.co.uk/notes/bellman-convergence.html
25. Mathematical Analysis of Reinforcement Learning - Bellman Optimality Equation | by Vaibhav Kumar | TDS Archive | Medium, accessed July 10, 2025, https://medium.com/data-science/mathematical-analysis-of-reinforcement-learning-bellman-equation-ac9f0954e19f
26. Bellman operator convergence enhancements in reinforcement learning algorithms - arXiv, accessed July 10, 2025, https://arxiv.org/pdf/2505.14564
27. Detailed Proof of the Bellman Optimality equations : r/reinforcementlearning - Reddit, accessed July 10, 2025, https://www.reddit.com/r/reinforcementlearning/comments/1klmntc/detailed_proof_of_the_bellman_optimality_equations/
28. Reward-Based Learning, Model-Based and Model-Free, accessed July 10, 2025, https://www.quentinhuys.com/pub/HuysEa14-ModelBasedModelFree.pdf
29. Lecture 10: Q-Learning, Function Approximation, Temporal Difference Learning - Dimitrios Katselis, accessed July 10, 2025, http://katselis.web.engr.illinois.edu/ECE586/Lecture10.pdf
30. Analyzing the convergence of Q-learning through Markov Decision Theory - University of Twente Student Theses, accessed July 10, 2025, https://essay.utwente.nl/88829/1/Overmars_MA_EEMCS.pdf
31. Q-러닝(Q-Learning)에 대해 알아보기, accessed July 10, 2025, https://dailystoryvenus.tistory.com/42
32. 강화학습 개념부터 Deep Q Networks까지, 10분만에 훑어보기, accessed July 10, 2025, https://jeinalog.tistory.com/20
33. Q-Learning Explained: Learn Reinforcement Learning Basics, accessed July 10, 2025, https://www.simplilearn.com/tutorials/machine-learning-tutorial/what-is-q-learning
34. Q-Learning in Reinforcement Learning - GeeksforGeeks, accessed July 10, 2025, https://www.geeksforgeeks.org/machine-learning/q-learning-in-python/
35. Q-Learning Theory with an Expamle for Beginners - Kaggle, accessed July 10, 2025, https://www.kaggle.com/code/melikbugraozcelik/q-learning-theory-with-an-expamle-for-beginners
36. An Introduction to Q-Learning: A Tutorial For Beginners - DataCamp, accessed July 10, 2025, https://www.datacamp.com/tutorial/introduction-q-learning-beginner-tutorial
37. [강화학습] Q-Learning 이해하기 | 정리하여 내 것으로, AI, accessed July 10, 2025, https://hchoi256.github.io/rl/RL1/
38. 강화학습_Q-Learning #2 - 게임 클라 개발, accessed July 10, 2025, https://tsyang.tistory.com/200
39. [강화학습] 심층 Q-네트워크(DQN) - 마인드스케일, accessed July 10, 2025, https://www.mindscale.kr/docs/reinforcement-learning/dqn
40. 강화학습 - (3) 탐색과 활용 - 금융덕후 - 티스토리, accessed July 10, 2025, https://jyoondev.tistory.com/136
41. [강화학습] 탐색과 활용의 딜레마 - 마인드스케일, accessed July 10, 2025, https://www.mindscale.kr/docs/reinforcement-learning/exploration-exploitation
42. DRL 전략에서 탐색과 활용의 균형 - FasterCapital, accessed July 10, 2025, [https://fastercapital.com/ko/content/DRL-%EC%A0%84%EB%9E%B5%EC%97%90%EC%84%9C-%ED%83%90%EC%83%89%EA%B3%BC-%ED%99%9C%EC%9A%A9%EC%9D%98-%EA%B7%A0%ED%98%95.html](https://fastercapital.com/ko/content/DRL-전략에서-탐색과-활용의-균형.html)
43. [5] Hindsight Experience Replay (HER) - 당황했습니까 휴먼? - 티스토리, accessed July 10, 2025, https://ropiens.tistory.com/136
44. 아름답게 나이들게 하소서, accessed July 10, 2025, https://rhr0916.tistory.com/
45. [강화학습] 탐색 전략 :: 마인드스케일, accessed July 10, 2025, https://www.mindscale.kr/docs/reinforcement-learning/exploration-strategies
46. 강화학습 - (15) 입실론 그리디 - 금융덕후 - 티스토리, accessed July 10, 2025, https://jyoondev.tistory.com/148
47. [CS246] Bandits (2) - Epsilon-Greedy Algorithm - 궁금한게많은joon - 티스토리, accessed July 10, 2025, https://trivia-starage.tistory.com/237
48. 2.3 Softmax Action Selection, accessed July 10, 2025, http://incompleteideas.net/book/ebook/node17.html
49. 강화 학습 기본 - ε-탐욕 정책(ε-greedy Policy) 그리고 ε-decay, accessed July 10, 2025, https://wnthqmffhrm.tistory.com/8
50. [강화학습 입문] 탐험(Exploration)과 탐사(Exploitation)의 균형 - HB의 개발 블로그, accessed July 10, 2025, https://96hb.tistory.com/m/38
51. 강화학습 - (25) 정책 근사 - 금융덕후, accessed July 10, 2025, https://jyoondev.tistory.com/163
52. Boltzmann Exploration (softmax) efficient action probabilities update (and roulette wheel action selection) - Stack Overflow, accessed July 10, 2025, https://stackoverflow.com/questions/73405143/boltzmann-exploration-softmax-efficient-action-probabilities-update-and-roule
53. Why epsilon greedy for action selection? : r/reinforcementlearning - Reddit, accessed July 10, 2025, https://www.reddit.com/r/reinforcementlearning/comments/qbb175/why_epsilon_greedy_for_action_selection/
54. [강화학습] 08 - 큐러닝(QLearning) - 시나브로_개발자 성장기 - 티스토리, accessed July 10, 2025, https://developer-lionhong.tistory.com/16
55. What is the difference between off-policy and on-policy learning? - Cross Validated, accessed July 10, 2025, https://stats.stackexchange.com/questions/184657/what-is-the-difference-between-off-policy-and-on-policy-learning
56. Differences between Q-learning and SARSA - GeeksforGeeks, accessed July 10, 2025, https://www.geeksforgeeks.org/artificial-intelligence/differences-between-q-learning-and-sarsa/
57. On-policy and off-policy Reinforcement Learning: Key features and differences - Ericsson, accessed July 10, 2025, https://www.ericsson.com/en/blog/2023/12/online-and-offline-reinforcement-learning-what-are-they-and-how-do-they-compare
58. SARSA 알고리즘, accessed July 10, 2025, https://velog.io/@koyeongmin/SARSA
59. [강화학습] 07 - 살사(SARSA) - 시나브로_개발자 성장기, accessed July 10, 2025, https://developer-lionhong.tistory.com/15
60. Part2 - 8. SARSA : TD기법을 활용한 최적 정책 찾기, accessed July 10, 2025, https://hh-bigdata-career.tistory.com/14
61. On-Policy VS Off-Policy in Reinforcement Learning - Lei Mao's Log Book, accessed July 10, 2025, https://leimao.github.io/blog/RL-On-Policy-VS-Off-Policy/
62. Why Q-Learning is considered an off-policy method? - Google Groups, accessed July 10, 2025, https://groups.google.com/g/rl-list/c/4Efnr0gXhAU
63. Q Learning vs SARSA. Q-learning | by Priyadarshini Tamilselvan | Medium, accessed July 10, 2025, https://medium.com/@priya61197/q-learning-vs-sarsa-b9e433dec930
64. Bootcamp Summer 2020 Week 4 – On-Policy vs Off-Policy Reinforcement Learning, accessed July 10, 2025, https://core-robotics.gatech.edu/2022/02/28/bootcamp-summer-2020-week-4-on-policy-vs-off-policy-reinforcement-learning/
65. 강화학습 개념정리(3) - 알고리즘 종류, on-policy, off-policy, Q러닝, Policy Gradient, Model-Free, Model-Based - velog, accessed July 10, 2025, [https://velog.io/@kjb0531/%EA%B0%95%ED%99%94%ED%95%99%EC%8A%B5-%EA%B0%9C%EB%85%90%EC%A0%95%EB%A6%AC3](https://velog.io/@kjb0531/강화학습-개념정리3)
66. Are off-policy learning methods better than on-policy methods? - Stack Overflow, accessed July 10, 2025, https://stackoverflow.com/questions/42606589/are-off-policy-learning-methods-better-than-on-policy-methods
67. Sutton & Barto summary chap 11 - Off-policy methods for approximation | lcalem, accessed July 10, 2025, https://lcalem.github.io/blog/2019/02/02/sutton-chap11
68. [RL] 강화학습 이론: On-policy & Off-policy - 이것저것 테크블로그, accessed July 10, 2025, [https://ai-com.tistory.com/entry/RL-%EA%B0%95%ED%99%94%ED%95%99%EC%8A%B5-%EC%9D%B4%EB%A1%A0-On-policy-Off-policy](https://ai-com.tistory.com/entry/RL-강화학습-이론-On-policy-Off-policy)
69. [강화학습] On-Policy, Off-Policy, Online, Offline - Beom's Blog, accessed July 10, 2025, https://jebeom.github.io/fundamental/policy_diff/
70. Bootstrapping a DQN Replay Memory with Synthetic Experiences - arXiv, accessed July 10, 2025, https://arxiv.org/pdf/2002.01370
71. Overcoming the Curse of Dimensionality in Reinforcement Learning Through Approximate Factorization | Request PDF - ResearchGate, accessed July 10, 2025, https://www.researchgate.net/publication/385749895_Overcoming_the_Curse_of_Dimensionality_in_Reinforcement_Learning_Through_Approximate_Factorization
72. Curse of dimensionality - Wikipedia, accessed July 10, 2025, https://en.wikipedia.org/wiki/Curse_of_dimensionality
73. 1.6 History of Reinforcement Learning - Rich Sutton, accessed July 10, 2025, http://incompleteideas.net/book/1/node7.html
74. Mastering Atari Games with Q-Learning | by AI_Pioneer | AI monks.io - Medium, accessed July 10, 2025, https://medium.com/aimonks/mastering-atari-games-with-q-learning-c222800c7cb3
75. (PDF) The Importance of Experience Replay Buffer Size in Deep Reinforcement Learning, accessed July 10, 2025, https://www.researchgate.net/publication/379190323_The_Importance_of_Experience_Replay_Buffer_Size_in_Deep_Reinforcement_Learning
76. Reinforcement learning for continuous state and action space - Stack Overflow, accessed July 10, 2025, https://stackoverflow.com/questions/54051499/reinforcement-learning-for-continuous-state-and-action-space
77. How to discretize continuous state-action spaces in Q-learning: A symbolic control approach, accessed July 10, 2025, https://arxiv.org/html/2406.01548v3
78. Reinforcement Learning in Continuous State and Action Spaces | Hado van Hasselt, accessed July 10, 2025, https://hadovanhasselt.com/wp-content/uploads/2015/12/rl_in_continuous_spaces.pdf
79. Can Q-learning be used for continuous (state or action) spaces? - AI Stack Exchange, accessed July 10, 2025, https://ai.stackexchange.com/questions/12255/can-q-learning-be-used-for-continuous-state-or-action-spaces
80. Can DQN learn with discrete state spaces? - Artificial Intelligence Stack Exchange, accessed July 10, 2025, https://ai.stackexchange.com/questions/45409/can-dqn-learn-with-discrete-state-spaces
81. Strengths, Weaknesses, and Combinations of Model-based and Model-free Reinforcement Learning Kavosh Asadi Atui - ERA - University of Alberta, accessed July 10, 2025, https://era.library.ualberta.ca/items/5f163754-3a20-47a0-8649-14d1fd816671/view/1b83b444-8e86-42ae-aa91-2b0a185a6fa8/AsadiAtui_Kavosh_201510_MSc.pdf
82. Deep Q Network (DQN) - Investment with engineering-ladder - 티스토리, accessed July 10, 2025, https://engineering-ladder.tistory.com/68
83. Deep Q-Learning in Reinforcement Learning - GeeksforGeeks, accessed July 10, 2025, https://www.geeksforgeeks.org/deep-learning/deep-q-learning/
84. Real-World Applications of Q-Learning: A Gentle Introduction - Avenga, accessed July 10, 2025, https://www.avenga.com/magazine/q-learning-applications/
85. 딥마인드, 아타리 57개 모든 게임에서 '인간추월' 성과 - 한겨레, accessed July 10, 2025, https://www.hani.co.kr/arti/science/future/935551.html
86. [RL] 강화학습 알고리즘: (1) DQN (Deep Q-Network) - 이것저것 테크블로그 - 티스토리, accessed July 10, 2025, [https://ai-com.tistory.com/entry/RL-%EA%B0%95%ED%99%94%ED%95%99%EC%8A%B5-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-1-DQN-Deep-Q-Network](https://ai-com.tistory.com/entry/RL-강화학습-알고리즘-1-DQN-Deep-Q-Network)
87. DeepMind 논문으로 보는 강화학습의 기초 - STEMentor - 티스토리, accessed July 10, 2025, [https://stementor.tistory.com/entry/DeepMind-%EB%85%BC%EB%AC%B8%EC%9C%BC%EB%A1%9C-%EB%B3%B4%EB%8A%94-%EA%B0%95%ED%99%94%ED%95%99%EC%8A%B5%EC%9D%98-%EA%B8%B0%EC%B4%88](https://stementor.tistory.com/entry/DeepMind-논문으로-보는-강화학습의-기초)
88. DQN: Integration of Deep Learning and Reinforcement Learning - Medium, accessed July 10, 2025, https://medium.com/@kdk199604/dqn-integration-of-deep-learning-and-reinforcement-learning-fd5cd8ba9ac8
89. 멀티플레이어 게임에서의 강화학습 적용사례, accessed July 10, 2025, [https://velog.io/@gogostimpack/%EB%A9%80%ED%8B%B0%ED%94%8C%EB%A0%88%EC%9D%B4%EC%96%B4-%EA%B2%8C%EC%9E%84%EC%97%90%EC%84%9C%EC%9D%98-%EA%B0%95%ED%99%94%ED%95%99%EC%8A%B5-%EC%A0%81%EC%9A%A9%EC%82%AC%EB%A1%80](https://velog.io/@gogostimpack/멀티플레이어-게임에서의-강화학습-적용사례)
90. Explanation and Implementation of DQN with Tensorflow and Keras - Lukas Schwarz, accessed July 10, 2025, https://lukasschwarz.de/dqn
91. The Importance Of Experience Replay In Reinforcement Learning - FasterCapital, accessed July 10, 2025, https://fastercapital.com/topics/the-importance-of-experience-replay-in-reinforcement-learning.html/1
92. What is the Replay Buffer in DQN (Deep Q-Learning)? - Lazy Programmer, accessed July 10, 2025, https://lazyprogrammer.me/what-is-the-replay-buffer-in-dqn-deep-q-learning/
93. What is "experience replay" and what are its benefits? - Data Science Stack Exchange, accessed July 10, 2025, https://datascience.stackexchange.com/questions/20535/what-is-experience-replay-and-what-are-its-benefits
94. Revisiting Fundamentals of Experience Replay - Proceedings of Machine Learning Research, accessed July 10, 2025, https://proceedings.mlr.press/v119/fedus20a/fedus20a.pdf
95. What are target networks in DQN? - Milvus, accessed July 10, 2025, https://milvus.io/ai-quick-reference/what-are-target-networks-in-dqn
96. deep learning - Why is a target network required? - Stack Overflow, accessed July 10, 2025, https://stackoverflow.com/questions/54237327/why-is-a-target-network-required
97. Deep Q-Network (DQN) Architecture | DQN Explained - YouTube, accessed July 10, 2025, https://www.youtube.com/watch?v=M6tauX9ySoo
98. How does Double DQN improve Q-learning? - Milvus, accessed July 10, 2025, https://milvus.io/ai-quick-reference/how-does-double-dqn-improve-qlearning
99. Dueling DQN | Practical Reinforcement Learning for Robotics and AI, accessed July 10, 2025, https://www.reinforcementlearningpath.com/dueling-dqn/
100. Modeling Recommendation Systems as Reinforcement Learning Problem | by Rohit Thakur, accessed July 10, 2025, https://medium.com/@rthakur4298/modeling-recommendation-systems-as-reinforcement-learning-problem-a250e0727ea3
101. DQN, DDQN, D3QN 비교 - Don't hesitate - 티스토리, accessed July 10, 2025, https://smj990203.tistory.com/54
102. Difference between Dueling DQN and Double DQN? - Data Science Stack Exchange, accessed July 10, 2025, https://datascience.stackexchange.com/questions/52997/difference-between-dueling-dqn-and-double-dqn
103. DQN(Deep Q-Networks) (2) - 독학 연구소 - 티스토리, accessed July 10, 2025, https://jvvp512.tistory.com/40
104. Rainbow - DI-engine 0.1.0 documentation, accessed July 10, 2025, https://di-engine-docs.readthedocs.io/en/latest/12_policies/rainbow.html
105. www.reinforcementlearningpath.com, accessed July 10, 2025, [https://www.reinforcementlearningpath.com/dueling-dqn/#:~:text=Dueling%20DQN%20is%20a%20variant,specific%20action%20in%20that%20state](https://www.reinforcementlearningpath.com/dueling-dqn/#:~:text=Dueling DQN is a variant,specific action in that state)
106. Dueling Network Architectures for Deep Reinforcement Learning, accessed July 10, 2025, https://proceedings.mlr.press/v48/wangf16.pdf
107. Dueling Network Architectures for Deep Reinforcement Learning - Samrat Sahoo, accessed July 10, 2025, https://samratsahoo.com/2025/04/01/dueling-dqn
108. [논문 리뷰] Dueling Network Architectures for Deep Reinforcement Learning (Dueling DQN), accessed July 10, 2025, https://limepencil.tistory.com/42
109. Prioritized Experience Replay - arXiv, accessed July 10, 2025, https://arxiv.org/pdf/1511.05952
110. What is Prioritized Experience Replay (PER)? - Zilliz Vector Database, accessed July 10, 2025, https://zilliz.com/ai-faq/what-is-prioritized-experience-replay-per
111. Understanding Prioritized Experience Replay - GeeksforGeeks, accessed July 10, 2025, https://www.geeksforgeeks.org/machine-learning/understanding-prioritized-experience-replay/
112. Prioritized experience replay based on dynamics priority - PMC, accessed July 10, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10933423/
113. Understanding Prioritized Experience Replay - Seita's Place, accessed July 10, 2025, https://danieltakeshi.github.io/2019/07/14/per/
114. Rainbow DQN Explained | Papers With Code, accessed July 10, 2025, https://paperswithcode.com/method/rainbow-dqn
115. paperswithcode.com, accessed July 10, 2025, [https://paperswithcode.com/method/rainbow-dqn#:~:text=Rainbow%20DQN%20is%20an%20extended,Replay%20to%20prioritize%20important%20transitions.](https://paperswithcode.com/method/rainbow-dqn#:~:text=Rainbow DQN is an extended,Replay to prioritize important transitions.)
116. Rainbow DQN - AgileRL Documentation, accessed July 10, 2025, https://docs.agilerl.com/en/latest/api/algorithms/dqn_rainbow.html
117. [D] Deep Q learning for continuous action space. : r/MachineLearning - Reddit, accessed July 10, 2025, https://www.reddit.com/r/MachineLearning/comments/bfny3m/d_deep_q_learning_for_continuous_action_space/
118. Understanding Continuous Action Spaces with Actor-Critic methods (PPO) - Reddit, accessed July 10, 2025, https://www.reddit.com/r/MLQuestions/comments/9nofgi/understanding_continuous_action_spaces_with/
119. Continuous Action Spaces : r/reinforcementlearning - Reddit, accessed July 10, 2025, https://www.reddit.com/r/reinforcementlearning/comments/e0hs8p/continuous_action_spaces/
120. Actor-critic algorithm - Wikipedia, accessed July 10, 2025, https://en.wikipedia.org/wiki/Actor-critic_algorithm
121. RL introduction: simple actor-critic for continuous actions | by Andy Steinbach - Medium, accessed July 10, 2025, https://medium.com/@asteinbach/rl-introduction-simple-actor-critic-for-continuous-actions-4e22afb712
122. 18.2 Q-Learning, accessed July 10, 2025, https://kenndanielso.github.io/mlrefined/blog_posts/18_Reinforcement_Learning_Foundations/18_2_Q_learning.html
123. (4) 그리드월드와 다이내믹 프로그래밍 - Jang. Inspiration, accessed July 10, 2025, https://jang-inspiration.com/reinforcement-learning-4
124. A Beginner's Guide to Q-Learning: Understanding with a Simple ..., accessed July 10, 2025, https://medium.com/@goldengrisha/a-beginners-guide-to-q-learning-understanding-with-a-simple-gridworld-example-2b6736e7e2c9
125. python - Q Learning Applied To a Two Player Game - Stack Overflow, accessed July 10, 2025, https://stackoverflow.com/questions/49451366/q-learning-applied-to-a-two-player-game
126. (PDF) Mobile Robot Navigation Based on Q-Learning Technique - ResearchGate, accessed July 10, 2025, https://www.researchgate.net/publication/221911117_Mobile_Robot_Navigation_Based_on_Q-Learning_Technique
127. Reinforcement Learning and Its Applications in Finance - ResearchGate, accessed July 10, 2025, https://www.researchgate.net/publication/373119463_Reinforcement_Learning_and_Its_Applications_in_Finance
128. Reinforcement Learning in Finance | Planergy Software, accessed July 10, 2025, https://planergy.com/blog/reinforcement-learning-in-finance/
129. Reinforcement learning for Stock Market Trading - Rohan Saphal, accessed July 10, 2025, https://rohansaphal97.github.io/files/rl_final_report.pdf
130. MagnusPetersen/DQNRecommender - GitHub, accessed July 10, 2025, https://github.com/MagnusPetersen/DQNRecommender
131. Deep Reinforcement Learning for Recommender Systems | Shaped Blog, accessed July 10, 2025, https://www.shaped.ai/blog/deep-reinforcement-learning-for-recommender-systems--a-survey
132. Deep Reinforcement Learning for List-wise Recommendations - arXiv, accessed July 10, 2025, https://arxiv.org/pdf/1801.00209
133. Q-Learning, 모델 없이 학습하는 강화 학습, accessed July 10, 2025, [https://dongdoridong.tistory.com/entry/Q-Learning-%EB%AA%A8%EB%8D%B8-%EC%97%86%EC%9D%B4-%ED%95%99%EC%8A%B5%ED%95%98%EB%8A%94-%EA%B0%95%ED%99%94-%ED%95%99%EC%8A%B5](https://dongdoridong.tistory.com/entry/Q-Learning-모델-없이-학습하는-강화-학습)
134. 10 Pros and Cons of Reinforcement Learning [2025] - DigitalDefynd, accessed July 10, 2025, https://digitaldefynd.com/IQ/reinforcement-learning-pros-cons/
135. Model-Based vs. Model-Free Learning in RL - GeeksforGeeks, accessed July 10, 2025, https://www.geeksforgeeks.org/machine-learning/model-based-vs-model-free-learning-in-rl/
136. What's the difference between model-based and model-free reinforcement learning? : r/reinforcementlearning - Reddit, accessed July 10, 2025, https://www.reddit.com/r/reinforcementlearning/comments/1ibyio9/whats_the_difference_between_modelbased_and/
137. How does Dyna-Q work? - Milvus, accessed July 10, 2025, https://milvus.io/ai-quick-reference/how-does-dynaq-work
138. Dyna - Machine Learning Trading - OMSCS Notes, accessed July 10, 2025, https://www.omscs-notes.com/machine-learning-trading/dyna/
139. Dyna Algorithm in Reinforcement Learning | GeeksforGeeks, accessed July 10, 2025, https://www.geeksforgeeks.org/dyna-algorithm-in-reinforcement-learning/


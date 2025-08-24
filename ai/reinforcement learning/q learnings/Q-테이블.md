[Q-Learning](./index.md)
# Q-테이블


강화학습(Reinforcement Learning, RL)은 에이전트(agent)가 환경(environment)과의 상호작용을 통해 누적 보상(cumulative reward)을 최대화하는 최적의 전략, 즉 정책(policy)을 학습하는 머신러닝의 한 분야이다.1 이 패러다임의 핵심에는 에이전트가 특정 상황에서 어떤 행동이 가장 가치 있는지를 평가하고 학습하는 메커니즘이 있으며, Q-테이블은 이러한 메커니즘을 가장 직관적으로 구현한 자료 구조이다. 본 장에서는 Q-테이블이 작동하는 근본적인 강화학습의 맥락을 설정하고, Q-테이블이 표현하고자 하는 이론적 대상인 Q-함수를 정의하며, Q-테이블 자체의 개념과 구조를 확립한다.


강화학습의 근간은 에이전트와 환경 간의 상호작용 루프에 있다. 이산적인 시간 단계(time step)에 걸쳐, 의사결정 주체인 에이전트는 환경의 현재 상태(state, s)를 관찰하고, 가능한 행동(action, a) 중 하나를 선택하여 수행한다.3 그러면 환경은 새로운 상태(s′)로 전이하고, 에이전트가 취한 행동의 결과에 대한 피드백으로 보상(reward, r)을 제공한다.3 에이전트의 궁극적인 목표는 이러한 일련의 상호작용을 통해 얻는 보상의 총합을 최대화하는 정책을 학습하는 것이다.4

이러한 학습 방식 중 Q-러닝은 모델-자유(model-free) 학습 기법의 대표적인 예이다.4 이는 에이전트가 환경의 작동 방식, 즉 특정 상태에서 특정 행동을 했을 때 어떤 상태로 전이될 확률(transition probability)이나 어떤 보상을 받을지(reward function)에 대한 사전 지식을 필요로 하지 않음을 의미한다.8 대신, 에이전트는 오직 시행착오(trial-and-error)를 통해 직접 경험한 결과로부터 최적의 행동 방침을 학습한다.10 이러한 특성은 Q-러닝에 강력한 유연성과 광범위한 적용 가능성을 부여한다.


Q-러닝의 중심에는 Q-함수(Q-function), 즉 상태-행동 가치 함수(state-action value function)가 있다.5 Q-함수는 $Q(s, a)$로 표기되며, 특정 상태 s에서 특정 행동 a를 취한 후, 그 이후부터 최적의 정책을 따랐을 때 받을 것으로 기대되는 누적 할인 보상(expected cumulative discounted reward)의 총합을 의미한다.1 다시 말해, 이는 특정 상태-행동 쌍의 장기적인 "가치" 또는 "질(Quality)"을 나타내는 척도이다.

알고리즘의 명칭에 포함된 'Q'는 바로 이 "Quality"에서 유래한 것으로, 1989년 크리스토퍼 왓킨스(Christopher Watkins)가 자신의 박사 학위 논문에서 이 용어를 처음 도입하며 상태-행동 쌍의 가치를 평가하는 개념을 강조했다.7

강화학습의 최종 목표는 최적 정책(π∗)을 찾는 것이다. 최적 정책은 어떤 상태에 있더라도 에이전트가 취해야 할 최상의 행동을 알려준다. 만약 우리가 최적의 Q-함수인 $Q^*$를 알고 있다면, 최적 정책은 매우 간단하게 결정된다. 각 상태 $s$에서 가능한 모든 행동 $a$에 대한 $Q^*(s, a)$ 값들 중 가장 큰 값을 만드는 행동을 선택하면 된다.5 이를 수식으로 표현하면 다음과 같다:

π∗(s)=aargmaxQ∗(s,a)

결국, Q-러닝의 학습 과정은 최적의 Q-함수인 $Q^*$를 점진적으로 근사해 나가는 과정이라고 할 수 있다.


Q-테이블은 상태와 행동 공간이 모두 이산적(discrete)이고 유한(finite)한 환경에서 Q-함수를 구현하는 가장 직접적이고 직관적인 방법이다.15 이는 본질적으로 에이전트가 학습한 지식을 저장하는 거대한 조회 테이블(lookup table) 또는 "치트 시트(cheat sheet)"와 같다.13

Q-테이블의 구조는 2차원 행렬로, 행(row)은 환경에서 가능한 모든 상태(S)를 나타내고, 열(column)은 에이전트가 취할 수 있는 모든 행동(A)을 나타낸다.10 테이블의 각 셀 `(s, a)`에는 해당 상태-행동 쌍의 Q-값, 즉 $Q(s, a)$가 저장된다. 이 구조는 모든 상태-행동 쌍과 그 가치를 명시적으로 매핑한다.

학습 과정은 이 Q-테이블을 초기화하는 것에서 시작된다. 일반적으로 모든 Q-값을 0이나 임의의 작은 값으로 초기화하는데, 이는 에이전트가 환경에 대해 아무런 지식이 없는 초기 상태를 의미한다.13 이후 에이전트가 환경을 탐험하며 경험을 쌓고 보상을 받음에 따라, 이 테이블의 값들은 점진적으로 업데이트되어 실제 최적 Q-값에 가까워지게 된다.

이처럼 Q-테이블의 단순함은 강화학습의 핵심 원리를 이해하는 데 매우 효과적인 교육적 도구가 된다. 그리드 월드(Grid World)와 같은 간단한 예제에서 Q-테이블의 값이 어떻게 업데이트되고 최적 경로를 찾아가는지를 시각적으로 추적하는 것은 매우 직관적이다. 그러나 바로 이 단순성, 즉 모든 상태-행동 쌍을 명시적으로 저장해야 하는 테이블 형식의 표현 방식이 Q-테이블의 근본적인 한계로 작용하며, 이는 4장에서 다룰 '차원의 저주' 문제의 직접적인 원인이 된다.


Q-러닝의 핵심에는 에이전트가 경험을 통해 Q-테이블을 어떻게 점진적으로 개선해 나가는지에 대한 수학적 원리가 있다. 이 원리는 리처드 벨만(Richard Bellman)의 동적 계획법(Dynamic Programming)에서 유래한 벨만 방정식(Bellman Equation)에 깊이 뿌리내리고 있다. 본 장에서는 Q-러닝의 학습 엔진 역할을 하는 벨만 방정식과 이를 실제 알고리즘으로 구현한 Q-러닝 업데이트 규칙을 상세히 분석한다. 또한, 구체적인 예제를 통해 Q-값이 테이블 전체로 전파되며 학습이 이루어지는 과정을 단계별로 살펴본다.


벨만 방정식은 복잡한 순차적 의사결정 문제를 더 작고 관리하기 쉬운 재귀적 하위 문제로 분해하는 동적 계획법의 기본 원리이다.19 이 방정식의 핵심 아이디어는 특정 상태의 가치가 '즉각적인 보상'과 '그 상태에서 도달 가능한 다음 상태들의 할인된 최적 가치'의 합으로 표현될 수 있다는 것이다.21

Q-함수에 대한 벨만 최적 방정식(Bellman Optimality Equation)은 최적 Q-함수인 $Q^*$가 만족해야 하는 재귀적 관계를 정의한다.21 이는 다음과 같이 표현된다:

Q∗(s,a)=E

이 방정식은 상태 s에서 행동 a를 취했을 때의 최적 가치 $Q^*(s, a)$는, 그 행동으로 인해 즉시 받게 될 보상 $R_{t+1}$과, 전이된 다음 상태 $s_{t+1}$에서 취할 수 있는 모든 행동 $a'$들 중에서 가장 큰 Q-값을 선택하여 할인율 γ를 적용한 값의 기댓값과 같다는 것을 의미한다. 이 방정식은 $Q^*$를 자기 자신을 이용해 정의하므로, 이를 푸는 것이 Q-러닝의 목표가 된다.


모델-자유 환경에서는 환경의 전이 확률 $P(s'|s,a)$를 모르기 때문에 벨만 방정식을 직접 푸는 것이 불가능하다.4 Q-러닝은 이 문제를 시간차 학습(Temporal Difference, TD)이라는 영리한 접근법으로 해결한다. TD 학습은 실제 경험을 통해 얻은 단일 샘플을 사용하여 가치 함수를 점진적으로 업데이트하는 방식이다.4

Q-러닝의 핵심 업데이트 규칙은 다음과 같다 1:

Qnew(s,a)←Qold(s,a)+α[r+γa′maxQ(s′,a′)−Qold(s,a)]

이 수식은 "이전의 값과 새로운 정보의 가중합"을 이용하는 간단한 값 반복법이다.4 각 구성 요소를 자세히 살펴보면 다음과 같다.

- **Qold(s,a)**: 업데이트 이전의, 에이전트가 현재 가지고 있는 Q-값 추정치이다.
- **α (학습률, Learning Rate)**: 0과 1 사이의 값으로, 새로 얻은 정보를 기존 정보에 얼마나 반영할지를 결정하는 하이퍼파라미터이다. α가 높으면 에이전트가 최근 경험에 더 큰 비중을 두어 빠르게 학습하지만, 학습 과정이 불안정해질 수 있다. 반대로 α가 낮으면 기존 지식을 더 신뢰하여 학습이 느리지만 안정적으로 수렴하는 경향이 있다.1
- **γ (할인 인자, Discount Factor)**: 0과 1 사이의 값으로, 미래 보상과 현재 보상 간의 상대적 중요도를 조절한다. γ가 0에 가까우면 에이전트는 즉각적인 보상에만 집중하는 "근시안적(myopic)"인 행동을 하게 된다. 반면 1에 가까우면 미래의 보상을 현재의 보상만큼 중요하게 여기는 "원시안적(farsighted)"인 행동을 통해 장기적인 계획을 세우게 된다.1
- **r+γmaxa′Q(s′,a′)**: 이 부분이 **TD 타겟(TD Target)**이다. 이는 에이전트가 행동 a를 수행한 후 실제로 받은 보상 r과, 다음 상태 $s'$에서 현재 Q-테이블을 기준으로 가장 가치 있다고 판단되는 행동의 Q-값(즉, maxa′Q(s′,a′))을 결합하여 만든 새로운, 더 나은 Q-값의 추정치이다.
- **$$**: 이 값은 **TD 오차(TD Error)**라고 불린다. 이는 새로운 목표치와 기존 추정치 간의 차이를 나타내며, 학습 신호(learning signal)의 역할을 한다. 업데이트 규칙은 본질적으로 기존 Q-값을 이 오차의 방향으로 학습률 α만큼 이동시키는 것이다.

이처럼 Q-러닝 업데이트 규칙은, 풀기 어려운 벨만 방정식을 직접 해결하는 대신, 매 경험마다 TD 타겟이라는 "더 나은 추정치"를 계산하고, 현재 Q-값을 그 방향으로 조금씩 수정해 나가는 단순하고 반복적인 과정으로 변환한다. 이는 "추측으로부터 또 다른 추측을 배우는(learning a guess from another guess)" 과정으로 볼 수 있으며 23, 모델 없이도 최적해를 찾아갈 수 있는 Q-러닝의 강력함의 원천이다.

------

**표 1: Q-러닝 하이퍼파라미터 참조 가이드**

| 하이퍼파라미터              | 기호 | 일반적인 범위 | 에이전트 행동에 미치는 역할 및 영향                          |
| --------------------------- | ---- | ------------- | ------------------------------------------------------------ |
| 학습률 (Learning Rate)      | α    | (0,1]         | 에이전트의 학습 속도를 제어. 높은 α: 빠른 학습, 잠재적 불안정성. 낮은 α: 느리지만 안정적인 학습. 1 |
| 할인 인자 (Discount Factor) | γ    | [0,1)         | 즉각적 보상과 미래 보상의 균형을 조절. γ≈0: "근시안적" 에이전트, 단기 이익에 집중. γ≈1: "원시안적" 에이전트, 장기 보상을 위해 계획. 1 |
| 탐험률 (Exploration Rate)   | ϵ    | $$            | (3장에서 상세히 설명) 탐험-활용 트레이드오프를 제어. 높은 ϵ: 무작위 탐험 증가. 낮은 ϵ: 알려진 좋은 행동 활용 증가. 24 |

------


Q-테이블 업데이트 과정을 직관적으로 이해하기 위해 간단한 그리드 월드 환경을 예로 들어보자. 이 환경은 에이전트(로봇)가 출발점(S)에서 시작하여 목표 지점(G)에 도달하는 것을 목표로 한다. 중간에는 함정(H)이 있어 피해야 하며, 나머지 칸은 일반 길(F)이다.5 목표 지점에 도달하면 +1의 보상을, 함정에 빠지면 -1의 보상을, 일반 길을 이동할 때는 작은 음수 보상(-0.01)을 주어 최대한 빠른 도착을 유도한다.

1. Q-테이블 초기화

학습 시작 시, 에이전트는 환경에 대한 지식이 전무하므로 모든 상태-행동 쌍에 대한 Q-값을 0으로 채운 Q-테이블을 생성한다.10

2. 첫 번째 에피소드 시뮬레이션 및 Q-값 전파

에이전트는 출발점에서 무작위 행동을 선택하며 탐험을 시작한다. 예를 들어, 에이전트가 여러 번의 시행착오 끝에 마침내 목표 지점(G)에 도달했다고 가정하자. 목표 직전 상태를 $s_{pre-goal}$이라 하고, 여기서 '오른쪽'으로 이동하여 목표에 도달했다고 하자.

- **경험**: 상태 $s_{pre-goal}$에서 '오른쪽' 행동을 취해, 다음 상태 $s_{goal}$에 도달하고 보상 +1을 받았다.

- **TD 타겟 계산**: r+γmaxa′Q(sgoal,a′)=1+γ×0=1. (목표 상태는 종결 상태이므로 미래 가치는 0이다.)

- Q-값 업데이트: (학습률 α=0.1 가정)

  Q(spre−goal,오른쪽)←Qold+α

  Q(spre−goal,오른쪽)←0+0.1[1−0]=0.1

이 업데이트로 인해, $Q(s_{pre-goal}, \text{오른쪽})$의 값은 0에서 0.1로 변경된다. 이제 spre−goal 상태는 '오른쪽' 행동이 긍정적인 가치를 지닌다는 정보를 갖게 되었다.

다음 에피소드에서 에이전트가 spre−goal 상태의 이전 상태인 $s_{pre-pre-goal}$에서 $s_{pre-goal}$로 이동하는 경험을 한다고 가정하자.

- **경험**: 상태 $s_{pre-pre-goal}$에서 '오른쪽' 행동을 취해, 다음 상태 $s_{pre-goal}$에 도달하고 보상 -0.01을 받았다.

- **TD 타겟 계산**: r+γmaxa′Q(spre−goal,a′)=−0.01+γ×0.1. (다음 상태 $s_{pre-goal}$에서 가장 큰 Q-값은 방금 업데이트된 0.1이다.)

- Q-값 업데이트: (할인 인자 γ=0.9 가정)

  Q(spre−pre−goal,오른쪽)←0+0.1[−0.01+0.9×0.1−0]=0.008

이 과정을 통해 목표 지점의 긍정적 보상 신호가 마치 물결처럼 Q-테이블을 통해 점차 이전 상태들로 "역전파(back-propagate)"된다.21 수많은 에피소드를 반복하면서, 목표 지점으로 이어지는 경로 상의 상태-행동 쌍들은 점차 양의 Q-값을 갖게 되고, 함정으로 이어지는 경로들은 음의 Q-값을 갖게 된다. 결국, Q-테이블에는 목표 지점으로 향하는 일종의 "가치 경사(value gradient)"가 형성되어, 에이전트가 어떤 상태에서든 Q-값이 가장 높은 행동을 따라가기만 하면 최적 경로를 찾을 수 있게 된다.16 이것이 바로 Q-러닝의 학습 과정의 본질이다.


강화학습의 모든 에이전트는 근본적인 딜레마에 직면한다: 현재까지의 지식을 최대한 활용하여 최선의 결과를 얻을 것인가(활용, Exploitation), 아니면 더 나은 전략을 발견하기 위해 미지의 선택지를 시도해볼 것인가(탐험, Exploration). 이 두 가지 상충하는 목표 사이의 균형을 맞추는 것은 효과적인 학습을 위한 핵심 과제이다.8 본 장에서는 이 탐험-활용 딜레마를 심도 있게 분석하고, 이를 해결하기 위한 대표적인 전략인 입실론-그리디(Epsilon-Greedy) 정책을 살펴본다. 나아가, Q-러닝의 학습 방식(오프-폴리시)을 SARSA 알고리즘(온-폴리시)과 비교 분석함으로써, 이 딜레마에 대한 서로 다른 철학적 접근 방식과 그 실질적인 결과를 조명한다.


**활용(Exploitation)**은 에이전트가 현재 Q-테이블에 기반하여 가장 높은 가치를 가질 것으로 예상되는 행동을 선택하는 것을 의미한다.1 이는 현재까지 학습한 지식을 바탕으로 즉각적인 보상을 극대화하려는 전략이다. 예를 들어, 식당을 찾는 사람이 처음 발견한 괜찮아 보이는 식당에 바로 들어가는 것과 같다.25

**탐험(Exploration)**은 의도적으로 현재 최선이 아닌 다른 행동을 선택하여 환경에 대한 새로운 정보를 수집하는 행위이다.27 이는 단기적인 손실을 감수하더라도 장기적으로 더 나은 정책을 발견할 가능성을 열어두는 데 필수적이다. 여러 식당을 둘러보며 더 나은 곳이 있는지 확인하는 것에 비유할 수 있다.25

이 딜레마는 매우 중요하다. 만약 에이전트가 오직 활용만 한다면, 초기에 우연히 발견한 국소 최적해(local optimum)에 갇혀 전역 최적해(global optimum)를 결코 발견하지 못할 위험이 있다.25 반대로, 오직 탐험만 한다면, 이미 학습한 유용한 지식을 전혀 활용하지 못하고 비효율적인 무작위 행동만 반복하게 될 것이다.25 따라서 성공적인 강화학습 에이전트는 이 둘 사이에서 지능적인 균형을 유지해야 한다.


입실론-그리디 정책은 탐험과 활용의 균형을 맞추기 위한 가장 간단하면서도 널리 사용되는 전략이다.13 이 정책의 작동 방식은 다음과 같다.

- **메커니즘**: 에이전트는 행동을 선택하기 전에 0과 1 사이의 작은 값인 입실론(ϵ) 확률로 무작위 행동을 선택하여 **탐험**한다. 그리고 1−ϵ의 확률로는 현재 상태에서 Q-값이 가장 높은 행동을 선택하여 **활용**한다.13

- **입실론(ϵ)의 역할**: 입실론 값은 탐험의 정도를 제어하는 하이퍼파라미터이다. ϵ=0이면 에이전트는 순수하게 탐욕적인(greedy) 정책을 따라 오직 활용만 수행한다. ϵ=1이면 에이전트는 순전히 무작위로 행동하며 탐험만 수행한다.25 예를 들어 

  ϵ=0.1이라면, 에이전트는 10%의 확률로 새로운 길을 모색하고 90%의 확률로 이미 알고 있는 최선의 길을 따른다.24

- **입실론 감쇠(Epsilon Decay)**: 실용적인 구현에서는 종종 입실론 감쇠 기법을 사용한다. 학습 초기에는 에이전트의 지식이 부족하므로 높은 ϵ 값(예: 1.0)으로 시작하여 광범위한 탐험을 장려한다. 학습이 진행됨에 따라 Q-테이블이 점차 신뢰할 수 있는 정보를 담게 되면, ϵ 값을 점진적으로 감소시켜(decay) 에이전트가 탐험보다는 활용에 더 집중하도록 유도한다.24 이는 학습 초기에는 지식을 쌓고, 후기에는 그 지식을 바탕으로 최적의 성능을 내도록 행동 패턴을 전환시키는 효과적인 전략이다.


탐험-활용 딜레마에 대처하는 방식은 알고리즘의 학습 철학과도 깊이 연관된다. 이는 온-폴리시(On-Policy) 학습과 오프-폴리시(Off-Policy) 학습이라는 두 가지 주요 패러다임으로 나타나며, 각각 SARSA와 Q-러닝이 대표적인 예이다.

**SARSA 알고리즘 소개**: SARSA는 Q-러닝과 매우 유사한 시간차 학습 알고리즘이다.4 이름은 업데이트에 사용되는 경험 튜플의 순서, 즉 

**S**tate, **A**ction, **R**eward, **S**tate, **A**ction에서 유래했다.29

**업데이트 규칙의 결정적 차이**: 두 알고리즘의 근본적인 차이는 Q-값을 업데이트하는 방식에 있다.

- Q-러닝 (오프-폴리시):

  Q(s,a)←Q(s,a)+α[r+γa′maxQ(s′,a′)−Q(s,a)]

  Q-러닝은 다음 상태 $s'$에서 **가능한 최적의 행동**($\max_{a'} Q(s', a')$)을 가정하고 Q-값을 업데이트한다.30 즉, 에이전트가 탐험을 위해 실제로 어떤 행동을 취했는지와 무관하게, 항상 가장 탐욕적인 선택지의 가치를 학습한다. 이는 학습 대상인 '타겟 정책(target policy)'이 최적의 탐욕적 정책인 반면, 실제 행동을 생성하는 '행동 정책(behavior policy)'은 

  ϵ-greedy와 같은 탐험적 정책이므로, 두 정책이 다르다는 의미에서 **오프-폴리시(Off-Policy)**라고 불린다.1

- SARSA (온-폴리시):

  Q(s,a)←Q(s,a)+α[r+γQ(s′,a′)−Q(s,a)]

  SARSA는 다음 상태 $s'$에서 **실제로 취한 행동 a′**의 Q-값(Q(s′,a′))을 사용하여 현재 Q-값을 업데이트한다.30 여기서 $a'$는 현재 에이전트가 따르고 있는 

  ϵ-greedy 정책에 의해 선택된 행동이다. 즉, 학습 대상 정책과 행동 생성 정책이 동일하므로 **온-폴리시(On-Policy)**라고 한다.31

**실질적 의미와 결과**: 이 미묘한 차이는 에이전트의 학습 방식과 최종 정책에 지대한 영향을 미친다.

- **Q-러닝**: "낙관적(optimistic)"인 학습자이다. 탐험 중에 발생할 수 있는 실수나 위험의 비용을 가치 평가에 직접적으로 반영하지 않는다. 이로 인해 일반적으로 최적 정책으로 더 빠르게 수렴하는 경향이 있다.31 하지만 이 낙관성 때문에 실제 환경에서 위험한 행동을 배울 수도 있다.
- **SARSA**: "현실적(realistic)" 또는 "보수적(conservative)"인 학습자이다. 자신의 ϵ-greedy 정책이 초래하는 탐험적 실수의 비용까지 Q-값에 반영하여 학습한다. 따라서 위험을 회피하는 더 안전한 정책을 배우는 경향이 있지만, 최적 정책으로의 수렴은 더 느릴 수 있다.33

**"절벽 걷기(Cliff Walking)" 예제**: 이 고전적인 예제는 두 알고리즘의 차이를 명확하게 보여준다.32 절벽 가장자리를 따라가는 것이 최단 경로(최적 정책)인 환경에서, Q-러닝은 탐험 중 절벽으로 떨어지는 큰 음의 보상을 무시하고 최단 경로를 학습한다. 반면 SARSA는 탐험 중 실제로 절벽에 떨어져 본 경험으로부터 학습하기 때문에, 절벽에서 멀리 떨어진 더 길지만 안전한 경로를 최종 정책으로 선택하게 된다.29

결론적으로, Q-러닝의 오프-폴리시 특성은 두 가지 중요한 함의를 갖는다. 첫째, 행동 정책과 무관하게 최적 정책을 학습할 수 있다는 유연성은 나중에 설명할 '경험 재현(Experience Replay)'과 같은 강력한 기법의 이론적 기반이 된다.1 둘째, 학습하는 가치와 실제 행동 사이의 이러한 불일치는 학습의 분산(variance)을 높여 불안정성을 야기하는 원인이 되기도 한다.30 이는 함수 근사 기법이 도입될 때 해결해야 할 주요 과제가 된다.

------

**표 2: Q-러닝 vs. SARSA: 비교 분석**

| 특징                   | Q-러닝                                                       | SARSA                                                        |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **정책 유형**          | 오프-폴리시 (Off-Policy) 31                                  | 온-폴리시 (On-Policy) 31                                     |
| **학습 목표**          | 에이전트의 현재 정책과 무관하게 최적 Q-함수(Q∗)를 직접 학습  | 에이전트가 현재 따르고 있는 정책(예: ϵ-greedy)에 대한 Q-함수를 학습 |
| **업데이트 규칙 요소** | 다음 상태에서 가능한 최대 가치인 `max_a' Q(s', a')`를 사용 30 | 다음 상태에서 실제로 취한 행동의 가치인 `Q(s', a')`를 사용 30 |
| **탐험 전략**          | "낙관적". 가치 추정에 탐험의 잠재적 비용을 무시 32           | "보수적/현실적". 탐험적 행동의 비용을 가치 추정에 포함 32    |
| **수렴 속도**          | 일반적으로 최적 정책으로 더 빠르게 수렴 31                   | 탐험 전략에 묶여 있어 더 느릴 수 있음 31                     |
| **주요 사용 사례**     | 절대적인 최적 경로 탐색이 중요하고 탐험 위험이 낮은 환경 (예: 게임 시뮬레이션) | 학습 중 안전이 중요하고 값비싼 실수를 피해야 하는 환경 (예: 실제 로봇) 30 |

------


지금까지 Q-테이블은 강화학습의 핵심 원리를 명쾌하게 설명하는 강력한 도구였다. 그러나 이 서사의 전환점에서, 우리는 Q-테이블이 가진 근본적인 취약점을 비판적으로 분석해야 한다. 바로 가장 단순한 문제를 제외한 거의 모든 현실 문제에서 Q-테이블을 비실용적으로 만드는 '차원의 저주(Curse of Dimensionality)'이다. 본 장에서는 이 문제가 어떻게 Q-테이블의 메모리, 학습 시간, 일반화 능력에 연쇄적인 붕괴를 일으키는지 설명하고, 이것이 왜 심층 강화학습으로의 패러다임 전환을 촉발했는지 논증한다.


'차원의 저주'는 데이터의 상태를 설명하는 특징(차원)의 수가 증가함에 따라, 상태 공간의 전체 부피가 기하급수적으로 팽창하는 현상을 의미한다. 이로 인해 데이터 포인트들은 극도로 희소(sparse)하게 분포하게 되어 유의미한 패턴을 학습하기가 매우 어려워진다.35

Q-테이블에 이 개념을 적용하면 그 파괴력은 명확해진다. Q-테이블의 크기는 상태의 수 $|S|$와 행동의 수 $|A|$의 곱, 즉 $|S| \times |A|$에 의해 결정된다. 상태를 정의하는 변수가 단 몇 개만 추가되어도 $|S|$는 천문학적으로 증가한다. 예를 들어, 체스 게임의 가능한 상태 수는 약 $10^{120}$개에 달하며, 바둑은 그보다 훨씬 많다.37 비디오 게임에서 상태를 화면의 픽셀 데이터로 정의하는 경우를 생각해보자. 아타리 게임의 한 프레임(예: 210x160 픽셀, 128색)만으로도 가능한 상태의 수는 우주에 있는 원자의 수보다 많아진다.37 이러한 거대한 상태 공간을 담을 수 있는 Q-테이블을 생성하고 메모리에 저장하는 것은 물리적으로 불가능하다.17 이것이 테이블 방식이 가진 첫 번째이자 가장 극복하기 어려운 한계이다.8


약 무한한 메모리가 주어진다 해도 Q-테이블의 문제는 해결되지 않는다. 문제는 더욱 근본적인 차원에 존재한다.

- **느린 수렴 속도**: Q-러닝 알고리즘은 모든 상태-행동 쌍을 충분히 여러 번 방문하여 정확한 Q-값을 학습해야 한다. 상태 공간이 거대해지면, 모든 셀을 의미 있는 수준으로 채우는 데 필요한 경험의 양과 시간은 비현실적으로 길어진다.8 에이전트는 사실상 수렴에 도달하기 전에 학습을 끝내야 할 것이다.
- **일반화 능력의 부재**: Q-테이블은 본질적으로 이산적이며 일반화(generalization) 능력이 전혀 없는 자료 구조이다. 테이블의 각 행은 고유하고 독립적인 개체로 취급된다. 따라서 상태 s1에 대한 Q-값을 학습하는 것은, 그와 매우 유사한 상태 s2에 대한 Q-값에 어떠한 정보도 제공하지 못한다.37 예를 들어, 에이전트가 그리드 월드의 (3, 4) 위치에 있는 벽을 피하는 법을 배웠다고 해서, 바로 옆 (3, 5) 위치의 벽이 위험하다는 사실을 추론할 수 없다. 그 위치를 직접 경험하고 처벌을 받기 전까지는 말이다. 이러한 일반화 능력의 부재는 에이전트가 새로운 상황에 적응하고 효율적으로 학습하는 것을 근본적으로 방해한다.
- **하이퍼파라미터에 대한 민감성**: Q-러닝의 성능은 학습률 α, 할인 인자 γ, 입실론 감쇠 스케줄과 같은 하이퍼파라미터의 설정에 매우 민감하다.8 거대한 상태 공간에서 최적의 하이퍼파라미터를 찾는 것은 매우 어렵고 지루한 조정 과정을 요구한다.

이러한 문제들은 개별적인 단점이 아니라, Q-테이블의 근본적인 표현 방식에서 비롯된 필연적인 결과이다. 상태를 고유한 식별자로 취급하는 이산적인 테이블 구조는 메모리 문제(1차 효과), 시간 및 수렴 문제(2차 효과), 그리고 일반화 부재(3차 효과)라는 연쇄적인 실패를 낳는다. 이 인과 관계는 Q-테이블을 단순히 최적화하거나 개선하는 것만으로는 복잡한 문제를 해결할 수 없음을 명백히 보여준다. 필요한 것은 Q-함수를 표현하는 방식 자체에 대한 근본적인 패러다임 전환이었다.


Q-테이블이 직면한 '차원의 저주'라는 위기는 강화학습 분야에 혁신적인 해결책을 요구했다. 그 해답은 '함수 근사법(Function Approximation)'이라는 개념적 도약에서 나왔다. 이 접근법은 강화학습의 확장성을 극적으로 높였으며, 마침내 현대 심층 강화학습(Deep Reinforcement Learning, DRL)의 시대를 연 심층 Q-네트워크(Deep Q-Network, DQN)의 탄생으로 이어졌다. 본 장에서는 Q-테이블의 한계를 극복하기 위한 함수 근사법의 원리를 설명하고, 이것이 어떻게 심층 신경망과 결합하여 DQN이라는 강력한 모델로 진화했는지 그 과정을 추적한다.


함수 근사법의 핵심 아이디어는 거대한 Q-테이블을 명시적으로 저장하는 대신, 파라미터(θ)를 가진 함수 $Q(s, a; \theta)$를 사용하여 Q-함수를 근사하는 것이다.38 여기서 학습의 목표는 모든 상태-행동 쌍의 Q-값을 개별적으로 찾는 것이 아니라, 실제 Q-값을 가장 잘 추정하는 함수의 최적 파라미터 θ를 찾는 것으로 전환된다.22

이러한 접근 방식은 Q-테이블의 핵심 문제들을 정면으로 해결한다:

- **메모리 효율성**: 더 이상 모든 상태-행동 쌍을 저장할 필요 없이, 함수의 파라미터 θ만 저장하면 된다. 상태 공간이 아무리 커져도 파라미터의 수는 상대적으로 작게 유지될 수 있어 메모리 문제가 해결된다.38
- **일반화 능력**: 함수는 입력으로 주어진 상태의 특징을 기반으로 Q-값을 계산한다. 따라서 이전에 한 번도 방문하지 않은 상태에 대해서도, 이전에 학습한 유사한 상태들의 경험을 바탕으로 Q-값을 추정(일반화)할 수 있다.37 이는 학습 효율성을 극대화하는 결정적인 장점이다.


함수 근사법의 초기 시도는 선형 모델을 사용하는 것이었다. 이 방식에서는 Q-함수를 사전에 정의된 특징(feature)들의 선형 결합으로 표현한다. 예를 들어, $Q(s, a) = \sum_{i} \theta_i f_i(s, a)$와 같이, 상태와 행동으로부터 추출한 특징 벡터 $f(s, a)$와 가중치 벡터 θ의 내적으로 Q-값을 계산한다.38 이는 중요한 진전이었지만, 두 가지 큰 한계를 가졌다. 첫째, 효과적인 특징을 사람이 직접 설계해야 하는 '특징 공학(feature engineering)' 과정이 매우 어렵고 많은 노력을 요구했다. 둘째, 문제의 본질이 비선형적일 경우 선형 모델로는 최적의 정책을 표현하기 어려웠다.38

이러한 한계는 딥러닝의 부상과 함께 극복되었다. 심층 신경망(Deep Neural Networks, DNNs)은 복잡한 비선형 관계를 모델링할 수 있는 강력한 범용 함수 근사기이다.38 특히, 심층 신경망은 원시 데이터(raw data)로부터 계층적인 특징을 자동으로 학습하는 능력이 뛰어나, 수동적인 특징 공학의 필요성을 제거했다.41


2013년과 2015년, 딥마인드(DeepMind) 연구팀은 심층 신경망을 Q-러닝에 성공적으로 결합하여 아타리(Atari) 비디오 게임을 원시 픽셀 입력만으로 인간 전문가 수준으로 플레이하는 인공지능을 개발했다고 발표했다.20 이것이 바로 '심층 Q-러닝'의 탄생이었고, 이 모델을 심층 Q-네트워크(DQN)라고 부른다.

DQN의 구조는 다음과 같다. 일반적으로 컨볼루션 신경망(Convolutional Neural Network, CNN)으로 구성된 심층 신경망이 상태 s(예: 게임 화면 픽셀 데이터)를 입력으로 받아, 가능한 모든 행동에 대한 Q-값을 담은 벡터를 출력한다.28 즉, 거대하고 이산적인 Q-테이블이 연속적이고 일반화 가능한 Q-네트워크로 완전히 대체된 것이다.44

학습 과정 역시 Q-러닝의 원리를 계승하면서 딥러닝의 방식으로 변환되었다. 네트워크의 가중치(θ)는 손실 함수(loss function)를 최소화하는 방향으로 경사 하강법(gradient descent)을 통해 업데이트된다. 이때 손실 함수는 주로 평균 제곱 오차(Mean Squared Error, MSE)를 사용하며, 네트워크가 예측한 Q-값과 벨만 방정식으로부터 계산된 TD 타겟 간의 차이를 측정한다.45

Loss(θ)=E[(r+γa′maxQ(s′,a′;θ)−Q(s,a;θ))2]

이처럼 DQN은 Q-러닝의 핵심적인 벨만 업데이트 원리를 딥러닝의 강력한 최적화 프레임워크와 우아하게 결합시켰다. DQN은 Q-러닝을 대체하는 새로운 알고리즘이 아니라, Q-러닝의 원리를 거대한 규모에서 실현 가능하게 만든 진화된 형태이다. TD 타겟을 계산하고 현재의 가치 추정치를 그 방향으로 업데이트한다는 근본적인 알고리즘의 DNA는 그대로 유지된 채, 가치를 저장하는 자료 구조(테이블에서 네트워크로)와 그 값을 업데이트하는 메커니즘(단일 셀 수정에서 경사 하강법으로)만이 변경된 것이다. 이 패러다임 전환은 강화학습이 해결할 수 있는 문제의 복잡성과 규모를 극적으로 확장시키는 계기가 되었다.

------

**표 3: 테이블 방식 Q-러닝과 심층 Q-러닝(DQN) 비교**

| 특징                | 테이블 방식 Q-러닝 (Q-Table)                                 | 심층 Q-러닝 (DQN)                                            |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **표현 방식**       | 이산적인 Q-테이블 (행렬) 17                                  | 신경망 (함수 근사기) 17                                      |
| **확장성**          | 작고 이산적인 상태/행동 공간으로 제한됨. '차원의 저주'로 인해 실패. 17 | 고차원 및 연속적인 상태 공간(예: 이미지) 처리 가능. 17       |
| **일반화**          | 없음. 상태 간 지식 공유 불가. 37                             | 높음. 특징을 학습하여 유사하지만 새로운 상태에 대해 일반화. 17 |
| **메모리 요구사항** | $                                                            | S                                                            |
| **문제 해결**       | 단순하고 잘 정의된 문제(예: 작은 미로)에 최적. 17            | 원시 감각 입력을 다루는 복잡한 문제(예: 비디오 게임, 로봇 공학)에 탁월. 17 |
| **해석 가능성**     | 높음. Q-테이블은 투명하고 검사하기 쉬움. 17                  | 낮음. 신경망은 "블랙박스"로, 의사 결정 과정의 해석이 어려움. 17 |

------


Q-러닝의 원칙은 딥러닝으로의 전환 과정에서 살아남았을 뿐만 아니라, 오늘날 수많은 최첨단 심층 강화학습(DRL) 알고리즘의 근간을 형성하며 그 유산을 이어가고 있다. 본 장에서는 Q-러닝의 역사적 중요성을 되짚어보고, DQN을 안정적으로 만든 핵심 혁신 기술들을 분석한다. 나아가, DQN의 핵심 논리를 개선하고 확장한 다양한 후속 알고리즘들을 살펴보며, Q-러닝의 원리가 어떻게 더 넓은 강화학습 생태계, 특히 액터-크리틱 및 연속 제어 분야로까지 확장되었는지 탐구한다.


Q-러닝의 여정은 1989년 크리스토퍼 왓킨스(Christopher Watkins)의 박사 학위 논문 "지연된 보상으로부터의 학습(Learning from Delayed Rewards)"에서 시작되었다.7 이 논문은 동적 계획법과 시간차(TD) 학습 사이의 중요한 다리를 놓았으며, 특정 조건 하에서 수렴이 보장되는 최초의 모델-자유, 오프-폴리시 강화학습 알고리즘을 제시했다.23 이후 1992년, 왓킨스와 피터 다얀(Peter Dayan)이 발표한 수렴 증명은 Q-러닝에 이론적 견고함을 더하며, 신뢰할 수 있는 방법론으로 자리매김하게 했다.7

Q-러닝의 가장 근본적인 유산은 '행동-가치(action-value)', 즉 **Q-값**이라는 개념 그 자체이다. 특정 상태에서 특정 행동의 '질'을 평가하는 이 아이디어는 이후 등장하는 방대한 강화학습 알고리즘들의 핵심 구성 요소(building block)가 되었다.23


Q-러닝의 벨만 방정식을 이용한 업데이트(부트스트래핑)와 비선형 함수 근사기(신경망)의 결합은 본질적으로 불안정하여, 학습 과정이 발산하거나 심하게 진동할 수 있다.9 딥마인드는 이 문제를 해결하기 위해 두 가지 핵심적인 혁신을 도입했다.


- **메커니즘**: 에이전트는 환경과 상호작용하며 얻은 경험(전이) 튜플 $(s, a, r, s')$을 순서대로 학습에 사용하는 대신, '재현 버퍼(replay buffer)'라는 대용량 메모리에 저장한다. 실제 학습 시에는 이 버퍼에서 무작위로 미니배치(mini-batch)를 샘플링하여 네트워크를 업데이트한다.28
- **효과**: 이 기법은 두 가지 중요한 이점을 제공한다. 첫째, 연속적인 경험들 사이의 강한 시간적 상관관계(temporal correlation)를 깨뜨려, 학습 데이터를 독립적이고 동일하게 분포된(i.i.d.) 데이터처럼 만들어준다. 이는 안정적인 확률적 경사 하강법(SGD) 학습의 기본 가정에 부합한다. 둘째, 하나의 경험을 여러 번 학습에 재사용함으로써 데이터 샘플의 효율성을 크게 높인다.48


- **메커니즘**: 학습을 불안정하게 만드는 또 다른 요인은 TD 타겟 $r + \gamma \max_{a'} Q(s', a'; \theta)$을 계산할 때 사용되는 네트워크와, 업데이트 대상이 되는 네트워크가 동일하다는 점이다. 이는 가중치 θ가 업데이트될 때마다 타겟 값 자체가 흔들리는 '움직이는 타겟 문제(moving target problem)'를 야기한다. 이 문제를 해결하기 위해, DQN은 두 개의 네트워크를 사용한다. 하나는 실제 행동을 선택하고 학습을 통해 계속 업데이트되는 '온라인 네트워크(online network)'이고, 다른 하나는 TD 타겟을 계산하는 데만 사용되는 '타겟 네트워크(target network)'이다. 타겟 네트워크는 온라인 네트워크의 복제본이지만, 그 가중치는 일정 기간 동안 고정(frozen)되어 있으며, 주기적으로만 온라인 네트워크의 가중치를 복사하여 업데이트된다.39
- **효과**: 타겟 값을 계산하는 네트워크를 분리하고 고정함으로써, 학습 목표가 안정화된다. 이는 학습 과정의 진동과 발산을 크게 줄여주어 안정적인 수렴을 가능하게 하는 결정적인 역할을 한다.45


기본적인 DQN의 성공 이후, 연구자들은 Q-러닝 프레임워크의 미묘한 약점들을 해결하기 위한 다양한 개선안을 제시했다.


- **문제점**: 표준 Q-러닝의 타겟 계산에 사용되는 max 연산자는 Q-값을 체계적으로 과대평가(overestimation)하는 경향이 있다. 이는 추정 오차 중 양의 오차를 음의 오차보다 더 쉽게 선택하기 때문에 발생하는 '최대화 편향(maximization bias)' 때문이다.54
- **해결책**: DDQN은 행동 *선택*과 행동 *평가*를 분리한다. 다음 상태에서 최선의 행동을 선택할 때는 온라인 네트워크를 사용하지만, 그 선택된 행동의 가치를 평가할 때는 타겟 네트워크를 사용한다. 즉, 타겟 계산식이 다음과 같이 변경된다: Target=r+γQtarget(s′,a′argmaxQonline(s′,a′)). 이 간단한 변경만으로 과대평가 편향이 크게 줄어들어, 더 안정적인 학습과 높은 성능의 정책을 얻을 수 있다.57


- **아이디어**: 이 구조는 Q-네트워크를 두 개의 스트림(stream)으로 분리한다. 하나는 상태 자체의 가치를 평가하는 '상태-가치 함수(state-value function)' $V(s)$를 추정하고, 다른 하나는 각 행동이 평균적인 행동보다 얼마나 더 나은지를 평가하는 '어드밴티지 함수(advantage function)' $A(s, a)$를 추정한다. 최종 Q-값은 이 두 값을 결합하여 계산된다.59
- **효과**: 이 구조는 특정 상태에서 어떤 행동을 하든 가치가 크게 변하지 않는 경우에 특히 유용하다. 예를 들어, 장애물을 피하기 위해 반드시 움직여야 하는 상황에서는 어떤 방향으로 움직이는지가 중요하지만, 안전한 길을 달리고 있을 때는 어떤 행동을 해도 가치가 비슷할 수 있다. 듀얼링 네트워크는 상태의 가치를 행동과 독립적으로 학습할 수 있어, 보다 효율적인 학습을 가능하게 한다.60


- **아이디어**: 모든 경험이 학습에 동일하게 중요한 것은 아니다. 에이전트가 예측을 크게 틀린 "놀라운" 경험, 즉 TD 오차가 큰 경험이 학습에 더 유용하다. PER은 재현 버퍼에서 이러한 중요한 경험들을 더 높은 확률로 샘플링하는 기법이다.61
- **효과**: 학습 자원을 에이전트가 가장 잘 모르는 부분에 집중시킴으로써, 학습 효율성과 속도를 극적으로 향상시킨다.62 이때, 특정 샘플을 편향되게 추출하는 것으로부터 발생하는 학습 편향을 보정하기 위해 '중요도 샘플링(importance sampling)' 가중치를 사용한다.62


Q-러닝의 유산은 가치 기반 방법론을 넘어 강화학습의 다른 영역으로까지 확장된다.

- **액터-크리틱(Actor-Critic) 방법**: 이는 가치 기반 방법(Q-러닝 등)과 정책 기반 방법을 결합한 하이브리드 접근법이다. '액터(actor)'는 정책을 학습하여 어떤 행동을 취할지 결정하고, '크리틱(critic)'은 가치 함수를 학습하여 액터가 선택한 행동을 평가한다.65 많은 액터-크리틱 알고리즘에서 크리틱은 Q-함수를 학습하며, 여기서 계산된 TD 오차나 어드밴티지 값이 액터의 정책을 업데이트하는 데 사용된다.68

- **심층 결정론적 정책 경사(Deep Deterministic Policy Gradient, DDPG)**: DDPG는 "연속적인 행동 공간을 위한 DQN"으로 불리는 오프-폴리시 액터-크리틱 알고리즘이다.70 이산적인 행동 공간에서 

  max 연산을 통해 최적 행동을 찾았던 Q-러닝과 달리, 연속 공간에서는 이 연산이 불가능하다. DDPG는 이 문제를 결정론적(deterministic) 액터 네트워크를 도입하여 해결한다. 액터는 Q-값을 최대화하는 특정 행동을 직접 출력하고, 크리틱은 DQN과 유사하게 Q-함수를 학습한다. 그리고 크리틱의 Q-함수를 행동에 대해 미분한 그래디언트가 액터 네트워크를 업데이트하는 데 사용된다.70 이는 Q-함수를 학습한다는 핵심 아이디어가 어떻게 연속 제어 문제로까지 확장될 수 있는지를 보여주는 대표적인 사례이다.

이러한 발전의 역사는 Q-러닝에서 시작된 핵심 아이디어들이 어떻게 문제점을 발견하고, 이를 체계적으로 해결하는 과정을 통해 현대의 정교한 DRL 알고리즘으로 진화해왔는지를 명확히 보여준다. 각각의 DQN 변종이나 액터-크리틱 방법론은 무작위적인 발명이 아니라, Q-러닝 프레임워크가 가진 특정 약점을 보완하기 위한 논리적이고 필연적인 결과물이었다.


본 보고서는 강화학습의 초석이 되는 Q-테이블의 개념적 기원부터 시작하여, 그 알고리즘적 작동 원리, 내재된 한계, 그리고 마침내 심층 강화학습 시대를 연 심층 Q-네트워크(DQN)로의 진화 과정을 포괄적으로 고찰했다. 마지막으로, Q-테이블에서 시작된 이 여정의 의미를 종합하고, 그 유산이 현대 인공지능 분야에서 어떻게 이어지고 있는지, 그리고 앞으로의 방향을 전망하며 분석을 마무리한다.


Q-테이블의 서사는 단순함과 강력함에서 시작하여, 그 구조적 한계로 인한 필연적 붕괴, 그리고 새로운 패러다임의 촉매제가 되기까지의 과정을 담고 있다.

초기에 Q-테이블은 작고 이산적인 환경에서 강화학습 문제를 해결하는 데 매우 효과적이고 직관적인 도구였다. 상태와 행동을 행과 열로 하는 명시적인 테이블 구조는 가치 기반, 모델-자유, 오프-폴리시 학습과 같은 강화학습의 핵심 개념을 이해하는 데 가장 이상적인 교육적 도구로 기능한다. 에이전트가 경험을 통해 테이블의 각 셀을 숫자로 채워나가며 최적의 길을 찾아가는 과정은 강화학습의 본질을 명쾌하게 보여준다.

그러나 Q-테이블의 바로 그 구조, 즉 모든 상태-행동 쌍을 개별적으로 저장해야 한다는 점은 '차원의 저주'라는 거대한 벽 앞에서 스스로의 종말을 예고했다. 상태 공간이 조금만 복잡해져도 테이블의 크기는 관리 불가능한 수준으로 폭발했고, 이는 메모리, 학습 시간, 일반화 능력의 부재라는 연쇄적인 문제로 이어졌다. Q-테이블의 실패는 단순한 기술적 한계가 아니었다. 이는 강화학습이 더 복잡하고 현실적인 문제로 나아가기 위해 반드시 넘어야 할 산이었으며, 이산적인 표현 방식에서 벗어나 일반화가 가능한 '함수 근사법'을 채택하도록 강제하는 결정적인 계기가 되었다.

결론적으로, Q-테이블의 가장 큰 유산은 그 성공이 아니라 역설적으로 그 실패에 있다. Q-테이블의 명백한 한계는 딥러닝과의 결합을 촉발했고, 심층 Q-네트워크(DQN)의 탄생을 이끌며 현대 심층 강화학습 혁명의 서막을 열었다. 따라서 Q-테이블은 강화학습의 역사에서 사라진 유물이 아니라, 다음 시대를 연 위대한 첫걸음으로 평가되어야 한다.


Q-러닝과 그 직계 후손인 DQN의 원리는 단순히 역사적 개념에 머무르지 않고, 오늘날에도 다양한 인공지능 분야에서 활발히 응용되고 있다. 특히 로봇 공학 분야에서 경로 계획, 자율 주행, 객체 조작과 같은 복잡한 문제 해결에 DQN 기반 알고리즘들이 적극적으로 활용되고 있다.73 이는 Q-값을 추정하고 이를 통해 정책을 개선한다는 근본적인 아이디어가 여전히 현대 AI 툴킷의 중요한 일부임을 증명한다.

앞으로의 전망 또한 Q-러닝의 유산 위에서 펼쳐질 것이다. 더 안정적이고, 더 효율적이며, 더 일반화 성능이 뛰어난 방식으로 행동-가치(Q-값)를 추정하고 전파하려는 연구는 강화학습 분야의 핵심적인 동력으로 계속 작용할 것이다. Double DQN, Dueling DQN, PER과 같은 개선안들을 넘어, 더 정교한 네트워크 아키텍처, 더 발전된 탐험 전략, 그리고 자기 지도 학습(self-supervised learning)과의 결합 등을 통해 가치 기반 강화학습은 계속해서 진화할 것이다.

1989년 크리스토퍼 왓킨스가 제시했던, 지연된 보상 속에서 행동의 '질(Quality)'을 학습한다는 단순하면서도 심오한 아이디어는, 수십 년이 지난 지금도 인공지능이 더 복잡한 세상과 상호작용하며 지능을 발전시켜 나가는 여정의 변치 않는 이정표로 남아 있다.76 Q-테이블에서 시작된 이 지적 탐구는 앞으로도 인공지능의 새로운 지평을 열어갈 것이다.


1. Q-Learning Explained: Learn Reinforcement Learning Basics - Simplilearn.com, accessed July 11, 2025, https://www.simplilearn.com/tutorials/machine-learning-tutorial/what-is-q-learning
2. Inside Reinforcement Learning. Part 1: A PhD Student's Perspective on... | by Krystie Dickson | Jul, 2025 | Medium, accessed July 11, 2025, https://medium.com/@krystiedickson/inside-reinforcement-learning-112ab51ae4e1
3. Reinforcement Q-Learning from Scratch in Python with OpenAI Gym - LearnDataSci, accessed July 11, 2025, https://www.learndatasci.com/tutorials/reinforcement-q-learning-scratch-python-openai-gym/
4. Q 러닝 - 위키백과, 우리 모두의 백과사전, accessed July 11, 2025, [https://ko.wikipedia.org/wiki/Q_%EB%9F%AC%EB%8B%9D](https://ko.wikipedia.org/wiki/Q_러닝)
5. [강화학습] Q 러닝 이해하기 - Atom's Space, accessed July 11, 2025, https://spacebike.tistory.com/53
6. What is Q-Learning? | Q-Learning Defined | Dremio, accessed July 11, 2025, https://www.dremio.com/wiki/q-learning/
7. Q-Learning - Synaptic Labs Blog, accessed July 11, 2025, https://blog.synapticlabs.ai/q-learning
8. Q-learning algorithm - Educative.io, accessed July 11, 2025, https://www.educative.io/answers/q-learning-algorithm
9. Q-learning - Wikipedia, accessed July 11, 2025, https://en.wikipedia.org/wiki/Q-learning
10. Q-Learning in Reinforcement Learning - GeeksforGeeks, accessed July 11, 2025, https://www.geeksforgeeks.org/machine-learning/q-learning-in-python/
11. What is Q-Learning? - Wandb, accessed July 11, 2025, https://wandb.ai/cosmo3769/Q-Learning/reports/What-is-Q-Learning---Vmlldzo1NTI1NzE0
12. Mastering Q-Learning in AI - Number Analytics, accessed July 11, 2025, https://www.numberanalytics.com/blog/mastering-q-learning-in-ai
13. deep-rl-class/units/en/unit2/q-learning.mdx at main / huggingface ..., accessed July 11, 2025, https://github.com/huggingface/deep-rl-class/blob/main/units/en/unit2/q-learning.mdx
14. Origins of Reinforcement Learning in AI | by alvin rogers - Medium, accessed July 11, 2025, https://medium.com/@rogers.alvin/origins-of-reinforcement-learning-in-ai-74fe2945fda1
15. velog.io, accessed July 11, 2025, [https://velog.io/@euisuk-chung/%EC%84%A4%EB%AA%85%EC%B6%94%EA%B0%80-Q-Learning-%EA%B0%95%ED%99%94%ED%95%99%EC%8A%B5%EC%9D%98-%ED%95%B5%EC%8B%AC-%EA%B0%9C%EB%85%90%EA%B3%BC-%EC%9D%B4%ED%95%B4#:~:text=6.-,Q%2DTable%EC%9D%B4%EB%9E%80%3F,%EC%9D%98%20%ED%96%89%EB%8F%99%EC%9D%84%20%ED%95%99%EC%8A%B5%ED%95%A9%EB%8B%88%EB%8B%A4.](https://velog.io/@euisuk-chung/설명추가-Q-Learning-강화학습의-핵심-개념과-이해#:~:text=6.-,Q-Table이란%3F,의 행동을 학습합니다.)
16. Reinforcement Learning Explained Visually (Part 4): Q Learning, step-by-step, accessed July 11, 2025, https://towardsdatascience.com/reinforcement-learning-explained-visually-part-4-q-learning-step-by-step-b65efb731d3e/
17. Difference Between Q Learning and Deep Q Learning - BytePlus, accessed July 11, 2025, https://www.byteplus.com/en/topic/514150
18. [모두RL-(2)] Dummy Q-Learning (table) - 딥러닝 소터디 - 티스토리, accessed July 11, 2025, https://sotudy.tistory.com/34
19. Understanding the Bellman Equation in Reinforcement Learning ..., accessed July 11, 2025, https://www.datacamp.com/tutorial/bellman-equation-reinforcement-learning
20. Reinforcement Learning: Deep Q-Learning | by Simon Palma - Medium, accessed July 11, 2025, https://medium.com/@simon.palma/reinforcement-learning-deep-q-learning-8dc006dad2bb
21. Bellman Equation - GeeksforGeeks, accessed July 11, 2025, https://www.geeksforgeeks.org/machine-learning/bellman-equation/
22. Lecture 10: Q-Learning, Function Approximation, Temporal Difference Learning - Dimitrios Katselis, accessed July 11, 2025, http://katselis.web.engr.illinois.edu/ECE586/Lecture10.pdf
23. How was the term 'Q-learning' coined? - Quora, accessed July 11, 2025, https://www.quora.com/How-was-the-term-Q-learning-coined
24. What is an epsilon-greedy policy? - Milvus, accessed July 11, 2025, https://milvus.io/ai-quick-reference/what-is-an-epsilongreedy-policy
25. Balancing Exploration and Exploitation with Epsilon-Greedy Strategy | CodeSignal Learn, accessed July 11, 2025, https://codesignal.com/learn/courses/game-on-integrating-rl-agents-with-environments/lessons/balancing-exploration-and-exploitation-with-epsilon-greedy-strategy
26. (4) 그리드월드와 다이내믹 프로그래밍 - Jang. Inspiration, accessed July 11, 2025, https://jang-inspiration.com/reinforcement-learning-4
27. Exploration–exploitation dilemma - Wikipedia, accessed July 11, 2025, [https://en.wikipedia.org/wiki/Exploration%E2%80%93exploitation_dilemma](https://en.wikipedia.org/wiki/Exploration–exploitation_dilemma)
28. 강화 학습 (DQN) 튜토리얼, accessed July 11, 2025, https://tutorials.pytorch.kr/intermediate/reinforcement_q_learning.html
29. What is the difference between Q-learning and SARSA learning? - Quora, accessed July 11, 2025, https://www.quora.com/What-is-the-difference-between-Q-learning-and-SARSA-learning
30. When to choose SARSA vs. Q Learning - Cross Validated - Stack Exchange, accessed July 11, 2025, https://stats.stackexchange.com/questions/326788/when-to-choose-sarsa-vs-q-learning
31. Differences between Q-learning and SARSA - GeeksforGeeks, accessed July 11, 2025, https://www.geeksforgeeks.org/artificial-intelligence/differences-between-q-learning-and-sarsa/
32. Q Learning vs SARSA. Q-learning | by Priyadarshini Tamilselvan | Medium, accessed July 11, 2025, https://medium.com/@priya61197/q-learning-vs-sarsa-b9e433dec930
33. What is the difference between Q-learning and SARSA? - Milvus, accessed July 11, 2025, https://milvus.io/ai-quick-reference/what-is-the-difference-between-qlearning-and-sarsa
34. Q-Learning and SARSA in RL - Similarities and Differences Explained | Dilith Jayakody, accessed July 11, 2025, https://dilithjay.com/blog/q-learning-and-sarsa
35. 차원의 저주 개념, 발생 원인과 해결 방법, accessed July 11, 2025, https://for-my-wealthy-life.tistory.com/40
36. [딥러닝] 차원의 저주 (Curse of dimensionality) 해설, 정리, 요약 - START_101 - 티스토리, accessed July 11, 2025, https://hyunhp.tistory.com/745
37. Convergence time of Q-learning Vs Deep Q-learning - Stack Overflow, accessed July 11, 2025, https://stackoverflow.com/questions/67261599/convergence-time-of-q-learning-vs-deep-q-learning
38. Q-function approximation - Mastering Reinforcement Learning, accessed July 11, 2025, https://gibberblot.github.io/rl-notes/single-agent/function-approximation.html
39. Deep Q-Learning (DQN) - Medium, accessed July 11, 2025, https://medium.com/@samina.amin/deep-q-learning-dqn-71c109586bae
40. Provably Efficient Q-learning with Function Approximation via Distribution Shift Error Checking Oracle - NIPS, accessed July 11, 2025, http://papers.neurips.cc/paper/9018-provably-efficient-q-learning-with-function-approximation-via-distribution-shift-error-checking-oracle.pdf
41. DQN - 나무위키, accessed July 11, 2025, https://namu.wiki/w/DQN
42. [RL] 강화학습 알고리즘: (1) DQN (Deep Q-Network) - 이것저것 테크블로그 - 티스토리, accessed July 11, 2025, [https://ai-com.tistory.com/entry/RL-%EA%B0%95%ED%99%94%ED%95%99%EC%8A%B5-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-1-DQN-Deep-Q-Network](https://ai-com.tistory.com/entry/RL-강화학습-알고리즘-1-DQN-Deep-Q-Network)
43. Q-Learning equation in Deep Q Network - Stack Overflow, accessed July 11, 2025, https://stackoverflow.com/questions/50581232/q-learning-equation-in-deep-q-network
44. Reinforcement Learning Explained Visually (Part 5): Deep Q Networks, step-by-step, accessed July 11, 2025, https://towardsdatascience.com/reinforcement-learning-explained-visually-part-5-deep-q-networks-step-by-step-5a5317197f4b/
45. The Deep Q-Learning Algorithm - Hugging Face Deep RL Course, accessed July 11, 2025, https://huggingface.co/learn/deep-rl-course/unit3/deep-q-algorithm
46. Deep Q-Networks Explained - Number Analytics, accessed July 11, 2025, https://www.numberanalytics.com/blog/deep-q-networks-explained
47. Experience Replay Explained - Papers With Code, accessed July 11, 2025, https://paperswithcode.com/method/experience-replay
48. What is the Replay Buffer in DQN (Deep Q-Learning)? - Lazy Programmer, accessed July 11, 2025, https://lazyprogrammer.me/what-is-the-replay-buffer-in-dqn-deep-q-learning/
49. Deep Reinforcement Learning with Experience Replay | by Hey Amit - Medium, accessed July 11, 2025, https://medium.com/@heyamit10/deep-reinforcement-learning-with-experience-replay-1222ea711897
50. What is "experience replay" and what are its benefits? - Data Science Stack Exchange, accessed July 11, 2025, https://datascience.stackexchange.com/questions/20535/what-is-experience-replay-and-what-are-its-benefits
51. Reinforcement Learning with Deep Q-Networks (DQN) | by Old Noisy Speaker | Medium, accessed July 11, 2025, https://medium.com/@old.noisy.speaker/reinforcement-learning-with-deep-q-networks-dqn-d56990c78179
52. What are target networks in DQN? - Milvus, accessed July 11, 2025, https://milvus.io/ai-quick-reference/what-are-target-networks-in-dqn
53. How does Deep Q-Networks (DQN) work? - BytePlus, accessed July 11, 2025, https://www.byteplus.com/en/topic/400784
54. Dueling Double Deep Q Learning using Tensorflow 2.x - Towards Data Science, accessed July 11, 2025, https://towardsdatascience.com/dueling-double-deep-q-learning-using-tensorflow-2-x-7bbbcec06a2a/
55. Double Deep Q Networks. Tackling maximization bias in Deep... | by Chris Yoon | TDS Archive | Medium, accessed July 11, 2025, https://medium.com/data-science/double-deep-q-networks-905dd8325412
56. Double Q-learning - NIPS, accessed July 11, 2025, https://proceedings.neurips.cc/paper/3964-double-q-learning.pdf
57. How does Double DQN improve Q-learning? - Milvus, accessed July 11, 2025, https://milvus.io/ai-quick-reference/how-does-double-dqn-improve-qlearning
58. DDQN: Tackling Overestimation Bias in Deep Reinforcement Learning | by Dong-Keon Kim, accessed July 11, 2025, https://medium.com/@kdk199604/ddqn-tackling-overestimation-bias-in-deep-reinforcement-learning-b1b0d6fa72a4
59. Dueling Network Explained | Papers With Code, accessed July 11, 2025, https://paperswithcode.com/method/dueling-network
60. Dueling Network Architectures for Deep Reinforcement Learning, accessed July 11, 2025, https://proceedings.mlr.press/v48/wangf16.pdf
61. milvus.io, accessed July 11, 2025, [https://milvus.io/ai-quick-reference/what-is-prioritized-experience-replay-per#:~:text=Prioritized%20Experience%20Replay%20(PER)%20is,randomly%20samples%20them%20during%20training.](https://milvus.io/ai-quick-reference/what-is-prioritized-experience-replay-per#:~:text=Prioritized Experience Replay (PER) is,randomly samples them during training.)
62. What is Prioritized Experience Replay (PER)? - Milvus, accessed July 11, 2025, https://milvus.io/ai-quick-reference/what-is-prioritized-experience-replay-per
63. Prioritized Experience Replay Explained | Papers With Code, accessed July 11, 2025, https://paperswithcode.com/method/prioritized-experience-replay
64. Understanding Prioritized Experience Replay - GeeksforGeeks, accessed July 11, 2025, https://www.geeksforgeeks.org/machine-learning/understanding-prioritized-experience-replay/
65. Actor-critic algorithm - Wikipedia, accessed July 11, 2025, https://en.wikipedia.org/wiki/Actor-critic_algorithm
66. 6.6 Actor-Critic Methods, accessed July 11, 2025, http://incompleteideas.net/book/ebook/node66.html
67. Actor-critic methods - Mastering Reinforcement Learning, accessed July 11, 2025, https://gibberblot.github.io/rl-notes/single-agent/actor-critic.html
68. Actor-critic methods – Mastering Reinforcement Learning, accessed July 11, 2025, https://uq.pressbooks.pub/mastering-reinforcement-learning/chapter/actor-critic-methods/
69. Advantage Actor Critic Tutorial: minA2C | Towards Data Science, accessed July 11, 2025, https://towardsdatascience.com/advantage-actor-critic-tutorial-mina2c-7a3249962fc8/
70. Deep Deterministic Policy Gradient - Spinning Up documentation - OpenAI, accessed July 11, 2025, https://spinningup.openai.com/en/latest/algorithms/ddpg.html
71. What is a deep deterministic policy gradient (DDPG)? - Milvus, accessed July 11, 2025, https://milvus.io/ai-quick-reference/what-is-a-deep-deterministic-policy-gradient-ddpg
72. How DDPG (Deep Deterministic Policy Gradient) Algorithms works in reinforcement learning ? | by Amaresh Marekar | Medium, accessed July 11, 2025, https://medium.com/@amaresh.dm/how-ddpg-deep-deterministic-policy-gradient-algorithms-works-in-reinforcement-learning-117e6a932e68
73. Deep reinforcement learning and robust SLAM based robotic control algorithm for self-driving path optimization - Frontiers, accessed July 11, 2025, https://www.frontiersin.org/journals/neurorobotics/articles/10.3389/fnbot.2024.1428358/full
74. Path planning of mobile robot based on improved double deep Q-network algorithm, accessed July 11, 2025, https://www.frontiersin.org/journals/neurorobotics/articles/10.3389/fnbot.2025.1512953/full
75. Deep Q-Networks in Robotics - Number Analytics, accessed July 11, 2025, https://www.numberanalytics.com/blog/deep-q-networks-in-robotics
76. The Roots of AI: Q-Learning (1989) - YouTube, accessed July 11, 2025, https://www.youtube.com/watch?v=6RJQcNbA2yk


---
layout: page
title: 최대 사후 확률(MAP) 추정 완벽 정복
permalink: /mathmatics/statistics/최대 사후 확률(MAP) 추정 완벽 정복
---


통계적 추정의 근본적인 목표는 우리가 가진 데이터를 바탕으로, 그 데이터를 생성했다고 여겨지는 확률 분포의 숨겨진 특성을 알아내는 것이다. 이 숨겨진 특성을 흔히 파라미터(parameter), 기호로는 $\theta$라고 부른다. 예를 들어, 동전 던지기에서 앞면이 나올 확률, 어떤 집단의 평균 키를 나타내는 정규분포의 평균 $\mu$ 등이 모두 파라미터 $\theta$에 해당한다. 하지만 우리는 모집단 전체를 조사할 수 없으므로, 제한된 데이터라는 불완전한 증거를 통해 이 $\theta$를 가장 잘 설명하는 단 하나의 값을 추측해야 한다. 이를 **점 추정(Point Estimation)**이라 한다.

이 점 추정을 수행하는 데에는 크게 두 가지의 철학적 관점이 존재하며, 이 관점의 차이가 모든 것을 결정한다. 바로 빈도주의(Frequentism)와 베이즈주의(Bayesianism)다.1

- **빈도주의**: 이 관점에서 파라미터 $\theta$는 우리가 아직 값을 모를 뿐, 정해져 있는 하나의 **상수(unknown constant)**다. 변하는 것은 데이터다. 즉, 동일한 $\theta$를 가진 모집단에서 데이터를 반복해서 추출하면 매번 다른 데이터가 얻어질 것이며, 이 데이터의 변동성에 초점을 맞춘다.
- **베이즈주의**: 이 관점에서 파라미터 $\theta$는 고정된 상수가 아니라, 불확실성을 내포한 **확률 변수(random variable)**다.1 즉,
  $\theta$ 자체가 특정 확률 분포를 따른다고 본다. 데이터는 이 불확실한 $\theta$에 대한 우리의 믿음을 갱신하고, 불확실성을 줄여주는 증거(evidence)로 사용된다.

이 근본적인 철학의 차이는 파라미터를 추정하는 구체적인 방법론으로 이어진다. 빈도주의는 관측된 데이터가 나올 확률(우도)을 가장 극대화하는 파라미터를 찾는 **최대 우도 추정(Maximum Likelihood Estimation, MLE)**으로 발전했다. 반면, 베이즈주의는 데이터를 관측한 후, 파라미터에 대한 우리의 믿음(사후 확률)이 가장 극대화되는 지점을 찾는 **최대 사후 확률 추정(Maximum a Posteriori, MAP)**으로 이어진다.

이 글에서는 MAP 추정을 완벽히 이해하기 위해, 먼저 그 수학적 근간인 베이즈 정리를 복습하고, 비교 대상인 MLE를 살펴본다. 그 후 MAP의 정의와 의미를 파헤치고, 구체적인 예제를 통해 두 방법론의 차이를 체감할 것이다. 마지막으로, MAP가 현대 머신러닝에서 왜 중요한지, 특히 과적합을 방지하는 정규화(regularization)와 어떻게 깊이 연결되는지를 탐구하며 학습의 여정을 마무리한다.


최대 사후 확률 추정(MAP)을 이해하기 위한 첫걸음은 그 이름에서도 알 수 있듯이 '사후 확률'을 계산하는 도구인 베이즈 정리를 제대로 이해하는 것이다. 베이즈 정리는 새로운 정보(데이터)가 주어졌을 때, 기존의 믿음(가설)을 어떻게 합리적으로 갱신해야 하는지에 대한 수학적 원리를 제공한다.3

베이즈 정리의 기본 공식은 다음과 같다.3
$$
P(A|B) = \frac{P(B|A)P(A)}{P(B)}
$$
이 공식을 파라미터 $\theta$를 추정하는 문제의 맥락으로 가져오면, 사건 A는 우리의 가설(hypothesis), 즉 '파라미터가 특정 값 $\theta$이다'가 되고, 사건 B는 우리가 관측한 데이터(Data), 즉 $D$가 된다. 이를 반영하여 공식을 다시 쓰면 다음과 같다.5
$$
P(\theta|D) = \frac{P(D|\theta)P(\theta)}{P(D)}
$$
여기서 각 항은 베이즈 추론에서 매우 중요한 의미를 갖는다.4

- **사후 확률 (Posterior Probability) $P(\theta|D)$**: 우리의 최종 목표. 데이터 $D$를 관측한 **후에** 갱신된 파라미터 $\theta$에 대한 확률(믿음의 정도)이다. MAP 추정은 바로 이 사후 확률을 최대로 만드는 $\theta$를 찾는 것이다.
- **우도 (Likelihood) $P(D|\theta)$**: '가능도'라고도 불린다. 특정 파라미터 $\theta$가 주어졌다고 **가정했을 때**, 우리가 가진 데이터 $D$가 관측될 조건부 확률이다. 이는 $\theta$의 함수로 간주되며, 어떤 $\theta$가 현재의 데이터를 더 잘 설명하는지를 나타내는 척도다.
- **사전 확률 (Prior Probability) $P(\theta)$**: 데이터를 관측하기 **전에** 우리가 파라미터 $\theta$에 대해 가지고 있던 사전 지식 또는 믿음이다. 예를 들어, 동전 던지기에서 앞면이 나올 확률 $\theta$는 0.5에 가까울 것이라는 믿음이 바로 사전 확률에 해당한다.
- **증거 (Evidence) $P(D)$**: 정규화 상수(Normalizing Constant)라고도 불린다. 데이터 $D$가 관측될 전체 확률이다. 이는 모든 가능한 $\theta$에 대해 우도와 사전 확률의 곱을 적분(연속 변수) 또는 합산(이산 변수)하여 계산된다. 즉, $P(D) = \int P(D|\theta)P(\theta) d\theta$ 이다. 이 값은 사후 확률의 총합이 1이 되도록 만들어주는 역할을 한다.

간단한 예시를 통해 이 흐름을 이해해 보자. 어떤 도시에서 비가 올 확률($P(비)=0.3$)을 사전 믿음으로 가지고 있다고 하자. 이때 '어떤 사람이 우산을 쓰고 있다'는 새로운 데이터(증거)를 관측했다. 비가 올 때 우산을 쓸 확률($P(우산|비)=0.6$)과 비가 안 올 때 우산을 쓸 확률($P(우산|비 안옴)=0.2$)을 알고 있다면, 베이즈 정리를 통해 '우산을 쓴 사람을 봤을 때, 실제로 비가 올 확률'이라는 사후 확률 $P(비|우산)$을 계산할 수 있다. 계산 결과는 약 0.56으로, 사전 믿음이었던 0.3보다 높아졌다.3

이처럼 베이즈 정리는 단순히 확률 공식이 아니다. 이는 '사전 믿음($P(\theta)$)'에 '새로운 증거($D$)'가 '기존 믿음 하에서 얼마나 그럴듯한지($P(D|\theta)$)'를 반영하여 '갱신된 믿음($P(\theta|D)$)'을 도출하는, 합리적인 학습 과정을 수학적으로 표현한 것이다. 이 프레임워크가 바로 MAP 추정의 심장이다.


MAP를 제대로 이해하려면, 그와 가장 자주 비교되는 빈도주의의 대표 주자, 최대 우도 추정(MLE)을 먼저 알아야 한다. MLE는 "어떤 파라미터 $\theta$가 지금 우리가 가진 데이터를 가장 그럴듯하게(likely) 만드는가?"라는 질문에 대한 답이다.9

MLE의 목표는 이름 그대로 우도 함수(Likelihood Function) $L(\theta|D) = P(D|\theta)$를 최대화하는 파라미터 $\hat{\theta}_{MLE}$를 찾는 것이다.1
$$
\hat{\theta}_{MLE} = \underset{\theta}{\mathrm{argmax}} P(D|\theta)
$$
여기서 중요한 점은 우도 $P(D|\theta)$는 $\theta$에 대한 확률 분포가 아니라는 것이다. 모든 $\theta$에 대해 우도를 다 더해도 1이 되지 않는다. 우도는 주어진 데이터를 가장 잘 설명하는 $\theta$를 찾기 위한 '척도' 또는 '점수 함수'로 이해해야 한다.9

실제 계산에서는 우도 함수 자체보다 **로그 우도(Log-Likelihood)**를 사용하는 경우가 대부분이다. 데이터 샘플들이 독립이라고 가정하면, 전체 데이터의 우도는 각 데이터 샘플의 우도의 곱으로 나타난다: $P(D|\theta) = \prod_{i=1}^{N} P(d_i|\theta)$. 여기에 로그를 취하면 곱셈이 덧셈으로 바뀌어 계산이 훨씬 간편해진다: $\log P(D|\theta) = \sum_{i=1}^{N} \log P(d_i|\theta)$. 또한, 여러 작은 확률 값들의 곱으로 인해 발생할 수 있는 수치적 언더플로우(underflow) 문제를 방지할 수 있다. 로그 함수는 단조 증가 함수이로, 로그 우도를 최대화하는 $\theta$는 원래 우도를 최대화하는 $\theta$와 동일하다.
$$
\hat{\theta}_{MLE} = \underset{\theta}{\mathrm{argmax}} \log P(D|\theta) = \underset{\theta}{\mathrm{argmax}} \sum_{i=1}^{N} \log P(d_i|\theta)
$$
MLE는 매우 직관적이다. 예를 들어, 동전을 10번 던져 앞면이 7번, 뒷면이 3번 나왔다고 하자. 이 데이터를 가장 그럴듯하게 만드는 앞면이 나올 확률 $\theta$는 무엇일까? MLE는 이 질문에 $\hat{\theta}_{MLE} = 7/10 = 0.7$이라고 답한다. 이는 시행 횟수 중 특정 사건이 발생한 비율을 확률로 보는 빈도주의적 관점과 정확히 일치한다.[1]

하지만 MLE에는 치명적인 맹점이 있다. 바로 "데이터가 왕이다"라는 철학이다. MLE는 오직 관측된 데이터에만 의존하여 결론을 내린다. 데이터가 충분히 많고 모집단을 잘 대표할 때는 이것이 강력한 장점이지만, 데이터가 적거나 편향되어 있을 때는 심각한 문제로 이어진다. 예를 들어, 동전을 단 3번 던졌는데 우연히 모두 앞면이 나왔다고 가정해 보자. MLE는 앞면이 나올 확률이 $3/3 = 1$, 즉 100%라고 추정할 것이다. 이것이 과연 합리적인 결론일까? 우리의 상식과 경험(사전 지식)은 '동전은 웬만하면 공정하다'고 말해주지만, MLE는 이러한 사전 지식을 반영할 방법이 없다. 이처럼 데이터에 과도하게 의존하는 성향은 과적합(overfitting) 문제로 직결되며, 바로 이 지점에서 MAP의 필요성이 대두된다.[6]


MLE가 데이터의 목소리에만 귀를 기울인다면, MAP는 데이터의 목소리(우도)와 우리의 기존 지식(사전 분포)을 함께 고려하여 가장 합리적인 결론을 내리는 방법론이다. MAP의 목표는 이름 그대로 사후 확률 $P(\theta|D)$를 최대화하는 파라미터 $\hat{\theta}_{MAP}$를 찾는 것이다.[10, 11, 12]
$$
\hat{\theta}_{MAP} = \underset{\theta}{\mathrm{argmax}} P(\theta|D) = \underset{\theta}{\mathrm{argmax}} \frac{P(D|\theta)P(\theta)}{P(D)}
$$
이 식을 최적화하는 과정에서 한 가지 중요한 단순화가 이루어진다. 분모에 있는 증거(Evidence) $P(D)$는 주어진 데이터 $D$에 대한 확률값으로, 우리가 찾으려는 파라미터 $\theta$와는 무관한 상수다. 따라서 $\theta$에 대한 최대화 문제를 풀 때, 이 상수는 결과에 아무런 영향을 미치지 않으므로 무시할 수 있다.[4, 5] 결국 MAP의 목표 함수는 우도와 사전 확률의 곱에 비례하게 된다.
$$
\hat{\theta}_{MAP} = \underset{\theta}{\mathrm{argmax}} P(D|\theta)P(\theta)
$$
MLE와 마찬가지로, 계산의 편의성을 위해 로그를 취하면 곱셈이 덧셈으로 변환된다. 이 로그 변환된 형태는 MAP의 본질을 가장 명확하게 보여주며, 나중에 살펴볼 정규화와의 관계를 이해하는 데 결정적인 역할을 한다.
$$
\hat{\theta}_{MAP} = \underset{\theta}{\mathrm{argmax}} \left( \log P(D|\theta) + \log P(\theta) \right)
$$
이 식을 MLE의 로그 우도 함수 $\log P(D|\theta)$와 비교해 보자. 차이점은 명확하다. MAP에는 $\log P(\theta)$라는 항이 추가되어 있다. 이 항이 바로 MLE에는 없었던 **사전 지식을 모델에 주입하는 통로**다. MLE가 순수하게 데이터 기반의 우도($\log P(D|\theta)$)만을 고려했다면, MAP는 여기에 사전 지식($\log P(\theta)$)을 더하여 균형을 맞춘다.

이러한 관점에서 MAP는 일종의 '타협'의 기술로 볼 수 있다. 최종 추정치 $\hat{\theta}_{MAP}$는 데이터가 가장 강력하게 지지하는 지점(우도가 최대인 곳)과 우리의 사전 믿음이 가장 강한 지점(사전 확률이 최대인 곳) 사이의 어딘가에서 결정된다. 데이터가 매우 적을 때는 사전 믿음이 추정치를 상식적인 방향으로 강하게 이끌어주는 '안전장치' 역할을 한다. 반대로 데이터가 매우 많아지면, 데이터가 주는 정보의 힘이 강해져 우도 항의 영향력이 사전 확률 항을 압도하게 되고, 추정치는 점차 MLE의 결과에 가까워진다.[13] 이처럼 데이터와 사전 지식 사이의 힘겨루기를 통해 최적의 균형점을 찾는 것이 바로 MAP 추정의 핵심 철학이다.


지금까지의 논의를 바탕으로 MLE와 MAP의 차이점을 철학적, 수학적, 실용적 관점에서 체계적으로 정리해 보자. 이를 통해 두 방법론의 관계를 더욱 명확히 이해할 수 있다.


MAP의 목적 함수는 $\underset{\theta}{\mathrm{argmax}} P(D|\theta)P(\theta)$이다. 만약 파라미터 $\theta$에 대한 사전 지식이 전혀 없어서 모든 $\theta$ 값이 동등하게 가능하다고 가정한다면 어떻게 될까? 이는 사전 분포 $P(\theta)$가 균등 분포(Uniform Distribution)임을 의미한다. 균등 분포의 확률밀도함수는 특정 범위 내에서 상수 값을 갖는다. 따라서 $P(\theta) = c$ (상수)가 되고, MAP의 목적 함수는 다음과 같이 변한다.
$$
\hat{\theta}_{MAP} = \underset{\theta}{\mathrm{argmax}} P(D|\theta) \cdot c
$$
최대화 문제에서 상수항 $c$는 결과에 영향을 주지 않으므로, 이 식은 결국 MLE의 목적 함수와 정확히 같아진다.[11, 14]
$$
\hat{\theta}_{MAP} = \underset{\theta}{\mathrm{argmax}} P(D|\theta) = \hat{\theta}_{MLE}
$$
이는 매우 중요한 사실을 시사한다. **MLE는 사전 분포가 균등 분포라고 가정한 MAP의 특수한 경우로 볼 수 있다.** 즉, "아무런 사전 지식이 없다"는 가정 자체가 하나의 (매우 약한) 사전 정보를 사용하는 베이즈주의적 행위라고 해석할 수 있는 것이다. 이로써 두 방법론이 단절된 것이 아니라, 베이즈주의라는 더 큰 틀 안에서 연결되어 있음을 알 수 있다.


데이터의 양은 두 추정치의 관계에 결정적인 영향을 미친다.

* **데이터가 적을 때**: 우도 함수 $P(D|\theta)$의 모양이 비교적 평평하고 넓게 퍼져 있다. 이때는 사전 분포 $P(\theta)$의 모양이 최종 사후 분포에 큰 영향을 미친다. 따라서 $\hat{\theta}_{MAP}$는 $\hat{\theta}_{MLE}$와 상당히 다른 값을 가질 수 있으며, 사전 분포가 추정치를 안정화시키는 중요한 역할을 한다.[1]
* **데이터가 많을 때**: 많은 데이터는 파라미터에 대한 강력한 증거를 제공한다. 이로 인해 우도 함수 $P(D|\theta)$는 특정 지점에서 매우 뾰족한 모양을 갖게 된다. 이 뾰족한 우도 함수가 상대적으로 완만한 사전 분포를 압도하게 되므로, 사후 분포의 최빈값은 우도 함수의 최빈값과 거의 일치하게 된다. 결과적으로 MAP 추정치는 MLE 추정치에 수렴한다.[13]


두 방법론의 핵심적인 차이점을 표로 정리하면 다음과 같다.

**Table 1: MLE vs. MAP 핵심 비교**

| 특징 | 최대 우도 추정 (MLE) | 최대 사후 확률 추정 (MAP) |
| :--- | :--- | :--- |
| **목표** | 우도 $P(D|\theta)$의 최대화 | 사후 확률 $P(\theta|D)$의 최대화 |
| **관점** | 빈도주의 (Frequentist) | 베이즈주의 (Bayesian) |
| **파라미터 $\theta$** | 알려지지 않은 **상수** | 불확실성을 갖는 **확률 변수** |
| **필요 정보** | 데이터 $D$ | 데이터 $D$ + 사전 분포 $P(\theta)$ |
| **핵심 수식** | $\hat{\theta}_{MLE} = \underset{\theta}{\mathrm{argmax}} P(D|\theta)$ | $\hat{\theta}_{MAP} = \underset{\theta}{\mathrm{argmax}} P(D|\theta)P(\theta)$ |
| **과적합(Overfitting)** | 데이터가 적을 경우 과적합 경향이 높음 [1, 13] | 사전 분포가 규제(regularization) 역할을 하여 과적합 방지에 유리 [10, 15] |
| **관계** | 사전 분포가 균등 분포(무정보 사전분포)일 때 MAP는 MLE와 동일해짐 [11, 14] | - |

이 비교를 통해 우리는 MAP가 MLE의 일반화된 형태이며, 사전 지식을 명시적으로 모델링에 포함함으로써 데이터가 부족할 때 발생할 수 있는 문제를 보완하는 강력한 도구임을 다시 한번 확인할 수 있다.


추상적인 개념을 구체적인 예제에 적용해 보면 이해가 훨씬 깊어진다. 동전의 앞면이 나올 확률 $\theta$를 추정하는 고전적인 문제를 통해 MAP 계산 과정을 단계별로 따라가 보자.


앞면이 나올 확률이 $\theta$인 동전을 $N$번 던져 앞면이 $a_H$번, 뒷면이 $a_T$번 나왔다고 하자 ($N = a_H + a_T$). 우리의 목표는 이 데이터 $D = (a_H, a_T)$를 바탕으로 $\theta$에 대한 MAP 추정치를 구하는 것이다.[6, 16]

각각의 동전 던지기는 성공($\theta$) 또는 실패($1-\theta$)의 결과를 갖는 베르누이 시행이다. 따라서 $N$번의 독립적인 시행에서 앞면이 $a_H$번 나올 확률은 이항 분포(Binomial Distribution)를 따른다. 이것이 바로 우리의 우도 함수가 된다. 최적화 과정에서 $\theta$와 무관한 조합 항 $\binom{N}{a_H}$는 상수로 취급하여 생략할 수 있다.
$$
P(D|\theta) = \binom{N}{a_H} \theta^{a_H} (1-\theta)^{a_T} \propto \theta^{a_H} (1-\theta)^{a_T}
$$


이제 $\theta$에 대한 사전 지식을 모델링할 차례다. $\theta$는 확률값이므로 0과 1 사이의 값을 갖는다. 이러한 0과 1 사이의 연속 확률 변수를 모델링하는 데 가장 적합하고 널리 사용되는 분포가 바로 **베타 분포(Beta Distribution)**다.[6, 16] 베타 분포는 두 개의 양수 하이퍼파라미터(hyperparameter) $\alpha$와 $\beta$에 의해 그 모양이 결정된다.
$$
P(\theta) = \text{Beta}(\theta; \alpha, \beta) = \frac{\Gamma(\alpha+\beta)}{\Gamma(\alpha)\Gamma(\beta)} \theta^{\alpha-1}(1-\theta)^{\beta-1} \propto \theta^{\alpha-1}(1-\theta)^{\beta-1}
$$
여기서 $\alpha$와 $\beta$는 우리의 사전 믿음을 나타낸다. 직관적으로 $\alpha-1$은 '사전에 관측했다고 믿는 앞면의 수', $\beta-1$은 '사전에 관측했다고 믿는 뒷면의 수'로 해석할 수 있다.[17] 예를 들어, $\alpha=1, \beta=1$로 설정하면 $P(\theta) \propto \theta^0(1-\theta)^0 = 1$이 되어 0과 1 사이의 균등 분포가 된다. 이는 $\theta$에 대한 아무런 정보가 없다는 '무정보 사전분포(non-informative prior)'에 해당한다.[18]

여기서 놀라운 점은, 우도 함수인 이항 분포와 사전 분포인 베타 분포를 곱하면 그 결과인 사후 분포 역시 베타 분포가 된다는 사실이다. 이처럼 사전 분포와 사후 분포가 동일한 분포족(distribution family)에 속할 때, 우리는 이 사전 분포를 **켤레 사전 분포(Conjugate Prior)**라고 부른다.[17, 18, 19] 켤레 관계는 복잡한 적분 계산 없이 간단한 대수적 조작만으로 사후 분포를 구할 수 있게 해주어 베이즈 추론을 매우 편리하게 만든다.[20]


이제 베이즈 정리에 따라 사후 분포를 계산해 보자. 사후 분포는 우도와 사전 분포의 곱에 비례한다.
$$
P(\theta|D) \propto P(D|\theta)P(\theta) \propto \left( \theta^{a_H} (1-\theta)^{a_T} \right) \cdot \left( \theta^{\alpha-1}(1-\theta)^{\beta-1} \right)
$$

$$
P(\theta|D) \propto \theta^{a_H+\alpha-1}(1-\theta)^{a_T+\beta-1}
$$

이 식의 형태를 보면, 사후 분포가 새로운 하이퍼파라미터 $\alpha' = a_H+\alpha$와 $\beta' = a_T+\beta$를 갖는 베타 분포임을 알 수 있다. 즉, $P(\theta|D) = \text{Beta}(\theta; a_H+\alpha, a_T+\beta)$ 이다.[18, 19, 21] 베이지안 업데이트 과정이 실제 데이터의 관측 횟수($a_H, a_T$)를 사전 믿음의 가상 횟수($\alpha, \beta$)에 단순히 더하는 것으로 귀결되는 것을 볼 수 있다.

MAP 추정치는 이 사후 분포의 확률 밀도가 가장 높은 지점, 즉 최빈값(mode)이다. 베타 분포 $\text{Beta}(a, b)$의 최빈값은 다음 공식으로 주어진다.[22, 23, 24]
$$
\text{mode} = \frac{a-1}{a+b-2} \quad (\text{단, } a>1, b>1)
$$
우리의 사후 분포 $\text{Beta}(a_H+\alpha, a_T+\beta)$에 이 공식을 적용하면, 최종적인 MAP 추정치를 얻을 수 있다.
$$
\hat{\theta}_{MAP} = \frac{(a_H+\alpha)-1}{(a_H+\alpha) + (a_T+\beta) - 2} = \frac{a_H + \alpha - 1}{a_H + a_T + \alpha + \beta - 2}
$$


* **데이터**: 동전을 10번 던져 앞면이 7번, 뒷면이 3번 나왔다 ($a_H=7, a_T=3$).

* **MLE 결과**: $\hat{\theta}_{MLE} = 7/10 = 0.7$.
* **MAP 시나리오 1 (약한 사전 믿음)**: 동전이 대략 공정할 것이라고 생각하지만 큰 확신은 없다. 사전 믿음을 '앞면 한 번, 뒷면 한 번을 미리 본 것'과 같이 설정하자. 이는 $\alpha=2, \beta=2$에 해당한다.
    $\hat{\theta}_{MAP} = \frac{7 + 2 - 1}{7 + 3 + 2 + 2 - 2} = \frac{8}{12} \approx 0.667$.
    MLE 결과인 0.7보다 사전 믿음의 중심인 0.5 쪽으로 약간 당겨진 것을 볼 수 있다.
* **MAP 시나리오 2 (강한 사전 믿음)**: 동전이 매우 공정하다고 강하게 믿는다. 사전 믿음을 '앞면 99번, 뒷면 99번을 미리 본 것'과 같이 설정하자. 이는 $\alpha=100, \beta=100$에 해당한다.
    $\hat{\theta}_{MAP} = \frac{7 + 100 - 1}{7 + 3 + 100 + 100 - 2} = \frac{106}{208} \approx 0.5096$.
    관측 데이터(7:3)가 꽤 한쪽으로 치우쳤음에도 불구하고, 강력한 사전 믿음의 영향으로 추정치가 0.5에 매우 가깝게 유지된다.

이 예제는 MAP가 어떻게 사전 지식을 활용하여 데이터만으로는 내리기 힘든 합리적인 결론을 도출하는지, 그리고 사전 믿음의 강도가 결과에 어떤 영향을 미치는지를 명확하게 보여준다.


MAP 추정의 가장 강력하고 실용적인 측면 중 하나는 현대 머신러닝의 핵심 기법인 **정규화(Regularization)**와 깊은 관련이 있다는 점이다. 이 연결고리를 이해하면, 정규화라는 기법이 단순한 공학적 트릭이 아니라 베이즈주의 철학에 뿌리를 둔 원리임을 깨닫게 된다.


머신러닝 모델을 훈련시킬 때 흔히 발생하는 문제 중 하나는 **과적합(overfitting)**이다. 이는 모델이 훈련 데이터의 패턴뿐만 아니라 노이즈까지 과도하게 학습하여, 훈련 데이터에 대해서는 매우 높은 성능을 보이지만 새로운 데이터(테스트 데이터)에 대해서는 예측 성능이 급격히 떨어지는 현상을 말한다.[15, 25] 과적합된 모델은 보통 모델의 파라미터(예: 선형 회귀의 가중치 $w$)들이 비정상적으로 큰 값을 갖는 경향이 있다.[15, 26] 모델이 복잡한 훈련 데이터 포인트를 모두 통과하기 위해 스스로를 과도하게 '왜곡'시키기 때문이다.

정규화는 이러한 과적합을 방지하기 위해 손실 함수(Loss Function)에 파라미터의 크기에 대한 페널티(penalty) 항을 추가하는 기법이다. 대표적인 정규화 기법은 다음과 같다.

\*  **L2 정규화 (Ridge Regression)**: 손실 함수에 파라미터 벡터의 L2 노름(norm)의 제곱, 즉 가중치들의 제곱합을 더한다. $\text{Loss} + \lambda \sum w_i^2$. 이는 큰 가중치에 강한 페널티를 부과하여 가중치 값을 전반적으로 작게 유지하도록 유도한다.
\*  **L1 정규화 (Lasso Regression)**: 손실 함수에 파라미터 벡터의 L1 노름, 즉 가중치들의 절댓값 합을 더한다. $\text{Loss} + \lambda \sum |w_i|$. L1 정규화는 중요하지 않은 피처의 가중치를 0으로 만드는 경향이 있어 피처 선택(feature selection) 효과도 있다.


이제 MAP 추정이 어떻게 정규화와 연결되는지 수학적으로 증명해 보자.
우리가 최소화하려는 머신러닝의 손실 함수는 보통 **음의 로그 우도(Negative Log-Likelihood)**로 해석될 수 있다. 예를 들어, 선형 회귀 모델에서 오차(noise)가 평균이 0인 가우시안 분포를 따른다고 가정하면, 음의 로그 우도는 제곱 오차합(Sum of Squared Errors, SSE)에 비례한다.
$$
\text{Loss} = -\log P(D|w) \propto \sum_{i=1}^{N} (y_i - w^T x_i)^2
$$
한편, MAP 추정의 목표는 $\log P(D|w) + \log P(w)$를 최대화하는 것이다. 이는 부호를 바꿔서 $-\log P(D|w) - \log P(w)$를 최소화하는 것과 완전히 동일한 문제다.
$$
\hat{w}_{MAP} = \underset{w}{\mathrm{argmax}} \left( \log P(D|w) + \log P(w) \right) = \underset{w}{\mathrm{argmin}} \left( -\log P(D|w) - \log P(w) \right)
$$
이 식을 정규화된 손실 함수와 비교해 보자.
$$
\text{argmin}_w \left( \text{Loss} + \text{Regularization Term} \right)
$$
두 식을 비교하면, MAP 추정에서의 $-\log P(w)$ 항이 정확히 정규화 항의 역할을 한다는 것을 알 수 있다.[10, 13]


이 관계를 더 구체적으로 살펴보자. 만약 모델 파라미터 $w$에 대한 사전 분포로 평균이 0이고 분산이 $\sigma_p^2$인 가우시안 분포를 가정한다면 어떻게 될까? 즉, $P(w) \sim \mathcal{N}(0, \sigma_p^2 I)$. 이 사전 분포는 "파라미터 $w$의 값들이 0에 가까운 작은 값일 것이다"라는 우리의 믿음을 나타낸다.

이 가우시안 사전 분포에 음의 로그를 취해보자.
$$
-\log P(w) = -\log \left( \frac{1}{(\sqrt{2\pi\sigma_p^2})^d} \exp\left(-\frac{w^T w}{2\sigma_p^2}\right) \right)
\\
= \text{const} + \frac{w^T w}{2\sigma_p^2} = \text{const} + \frac{1}{2\sigma_p^2} \sum_{i=1}^{d} w_i^2 = \text{const} + \lambda ||w||_2^2
$$
결과적으로, 파라미터에 대한 **가우시안 사전 분포를 가정한 MAP 추정은 L2 정규화를 사용하는 것과 수학적으로 동일하다**.[15] 여기서 정규화 강도 $\lambda$는 사전 분포의 분산 $\sigma_p^2$에 반비례($\lambda \propto 1/\sigma_p^2$)한다.[27]

이것이 의미하는 바는 매우 심오하다.

* 사전 믿음이 강할수록 (즉, 파라미터가 0 근처에 있을 것이라는 믿음이 강해 분산 $\sigma_p^2$가 작을수록), 정규화 강도 $\lambda$는 커진다. 이는 모델이 훈련 데이터에 과적합되지 않도록 더 강하게 제약하는 것과 같다.
* 반대로 사전 믿음이 약할수록 (분산 $\sigma_p^2$가 클수록), 정규화 강도 $\lambda$는 작아져 모델이 데이터에 더 자유롭게 맞춰지도록 허용한다.

이처럼 MAP는 정규화라는 실용적인 기법에 "파라미터의 분포에 대한 사전 믿음을 주입하는 행위"라는 명확한 확률론적 해석을 부여한다. 덕분에 우리는 하이퍼파라미터 $\lambda$를 단순히 '감'으로 튜닝하는 것을 넘어, 데이터의 노이즈와 파라미터에 대한 사전 믿음 사이의 균형을 조절하는 과정으로 이해할 수 있게 된다.


이 긴 여정을 통해 우리는 최대 사후 확률(MAP) 추정의 본질을 다각도에서 탐구했다. MAP는 단순히 하나의 통계적 추정 기법을 넘어, 데이터와 지식을 결합하는 합리적인 철학을 담고 있다.

MAP의 핵심 가치는 베이즈 정리라는 견고한 이론적 틀 안에서 **데이터가 제공하는 증거(우도)**와 **분석가의 사전 지식(사전 분포)**을 통합하는 데 있다. 이는 순전히 데이터에만 의존하는 MLE와 차별화되는 가장 큰 특징이다.

MAP의 주요 장점은 다음과 같이 요약할 수 있다.

1. **데이터 부족 문제 완화**: 데이터가 적거나 편향되어 MLE가 비상식적인 결과를 내놓을 위험이 있을 때, 사전 분포는 추정치를 합리적인 범위로 이끌어주는 '안전장치' 역할을 한다. 이는 모델의 안정성을 높이고 과적합을 효과적으로 방지한다.[1]
2. **정규화의 이론적 기반**: 머신러닝에서 경험적으로 효과가 입증된 L2, L1 정규화와 같은 기법들이 왜 작동하는지에 대한 명확한 확률론적 해석을 제공한다.[15] 가우시안 사전 분포가 L2 정규화로, 라플라스 사전 분포가 L1 정규화로 이어지는 것을 보임으로써, 정규화는 파라미터에 대한 특정 사전 믿음을 부과하는 베이즈주의적 행위임을 밝혔다.

물론 MAP에도 한계는 존재한다. MAP는 사후 확률 분포 $P(\theta|D)$ 전체가 아닌, 그 분포의 최빈값이라는 **단 하나의 점 추정치**만을 제공한다. 이는 파라미터가 가질 수 있는 값의 불확실성을 완벽하게 정량화하지는 못한다는 의미다. 파라미터의 불확실성 자체를 추론에 활용하려면, 사후 분포 전체를 다루는 **완전 베이즈 추론(Full Bayesian Inference)**(예: MCMC와 같은 샘플링 기법)으로 나아가야 한다.

결론적으로, MAP 추정은 실용성과 이론 사이에서 최적의 균형점을 제공하는 매우 강력한 도구다. 완전 베이즈 추론처럼 계산적으로 부담스럽지 않으면서도, MLE처럼 사전 지식을 완전히 무시하여 발생하는 위험을 피할 수 있다. 따라서 수많은 머신러닝 문제에서 MAP는 성능과 효율성이라는 두 마리 토끼를 잡는 '스위트 스팟(sweet spot)'을 제공하며, 더 넓은 베이즈주의의 세계로 들어가는 중요한 관문 역할을 한다.


1. 확률에 대한 Frequentist와 Bayesian 접근, 그리고 MLE와 MAP - untitled blog - 티스토리, accessed July 23, 2025, https://untitledtblog.tistory.com/183
2. 베이즈 정리 - 나무위키, accessed July 23, 2025, [https://namu.wiki/w/%EB%B2%A0%EC%9D%B4%EC%A6%88%20%EC%A0%95%EB%A6%AC](https://namu.wiki/w/베이즈 정리)
3. 챗GPT 확률통계 - 21 베이즈 정리, accessed July 23, 2025, https://r2bit.com/gpt-stat/prob_bayes.html
4. 6.6 베이즈 정리 - 데이터 사이언스 스쿨, accessed July 23, 2025, [https://datascienceschool.net/02%20mathematics/06.06%20%EB%B2%A0%EC%9D%B4%EC%A6%88%20%EC%A0%95%EB%A6%AC.html](https://datascienceschool.net/02 mathematics/06.06 베이즈 정리.html)
5. 베이즈 통계(5): 가설과 사전확률분포 - 윤영민 교수의 사유 공간, accessed July 23, 2025, http://infoso.kr/?p=4013
6. 직관과 MLE, MAP(2) - Deeper Learning - 티스토리, accessed July 23, 2025, [https://dlaiml.tistory.com/entry/%EC%A7%81%EA%B4%80%EA%B3%BC-MLE-MAP2](https://dlaiml.tistory.com/entry/직관과-MLE-MAP2)
7. 머신러닝 - 분류 - 베이즈 정리 - 베이지안 정리 - Char - 티스토리, accessed July 23, 2025, https://charstring.tistory.com/995
8. 베이즈 통계(6): 데이터와 우도 - 윤영민 교수의 사유 공간, accessed July 23, 2025, http://infoso.kr/?p=4015
9. 확률 (Probability) vs. 우도 (Likelihood) - R, Python 분석과 프로그래밍의 친구 (by R Friend), accessed July 23, 2025, https://rfriend.tistory.com/813
10. 머신러닝: MLE, MAP - velog, accessed July 23, 2025, [https://velog.io/@hotsun1508/%EB%A8%B8%EC%8B%A0%EB%9F%AC%EB%8B%9D-MLE-MAP](https://velog.io/@hotsun1508/머신러닝-MLE-MAP)
11. Bayes theorem(베이즈정리)와 MLE/MAP - 휴블로그 - 티스토리, accessed July 23, 2025, https://sanghyu.tistory.com/10
12. MLE(Maximum Likelihood Estimation) 최대우도법 - 삶은 확률의 구름 - 티스토리, accessed July 23, 2025, https://ebbnflow.tistory.com/330
13. maximum likelihood - MAP estimation as regularisation of MLE - Stats Stackexchange, accessed July 23, 2025, https://stats.stackexchange.com/questions/367485/map-estimation-as-regularisation-of-mle
14. ML(Maximum Likelihood)와 MAP(maximum a posterior) - gaussian37 - JINSOL KIM, accessed July 23, 2025, https://gaussian37.github.io/ml-concept-mle-and-map/
15. Avoiding overfitting using regularisation - Probabilistic Modelling and Inference, accessed July 23, 2025, https://pmi-book.org/content/regression/regression-regularisation.html
16. [기계학습] Parametric Density Estimation - Maximum A Posteriori Estimation(MAP) - velog, accessed July 23, 2025, [https://velog.io/@claude_ssim/%EA%B8%B0%EA%B3%84%ED%95%99%EC%8A%B5-Parametric-Density-Estimation-Maximum-A-Posteriori-EstimationMAP](https://velog.io/@claude_ssim/기계학습-Parametric-Density-Estimation-Maximum-A-Posteriori-EstimationMAP)
17. Conjugate prior - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Conjugate_prior
18. Beta Conjugate Prior | Real Statistics Using Excel, accessed July 23, 2025, https://real-statistics.com/bayesian-statistics/bayesian-statistics-for-binomial-distributed-data/analytic-approach-binomial-data/beta-conjugate-prior/
19. Conjugate priors: Beta and normal Class 15, 18.05, accessed July 23, 2025, https://math.mit.edu/~dav/05.dir/class15-prep.pdf
20. Binomial-beta conjugate pair, accessed July 23, 2025, https://drvalle1.github.io/9_Conjugate_pairs_example.html
21. Beta-Binomial Distribution Demo - Biostatistics, accessed July 23, 2025, https://biostatistics.mdanderson.org/SoftwareDownload/SoftwareFiles/BBDD/BetaBinomialDistributionDemoUsersGuide.pdf
22. pongdangstory.tistory.com, accessed July 23, 2025, [https://pongdangstory.tistory.com/516#:~:text=3)%20%EB%B6%84%ED%8F%AC%EC%9D%98%20%ED%8F%89%EA%B7%A0%CE%BC,%2B%CE%B2%2D2)%EC%9D%B4%EB%8B%A4.](https://pongdangstory.tistory.com/516#:~:text=3) 분포의 평균μ,%2Bβ-2)이다.)
23. 8.7 베타분포, 감마분포, 디리클레분포 - 데이터 사이언스 스쿨, accessed July 23, 2025, [https://datascienceschool.net/02%20mathematics/08.07%20%EB%B2%A0%ED%83%80%EB%B6%84%ED%8F%AC%2C%20%EA%B0%90%EB%A7%88%EB%B6%84%ED%8F%AC%2C%20%EB%94%94%EB%A6%AC%ED%81%B4%EB%A0%88%20%EB%B6%84%ED%8F%AC.html](https://datascienceschool.net/02 mathematics/08.07 베타분포%2C 감마분포%2C 디리클레 분포.html)
24. 08.07 베타분포, 감마분포, 디리클레 분포.ipynb - Colab, accessed July 23, 2025, [https://colab.research.google.com/github/datascienceschool/book/blob/master/ds/02%20mathematics/08.07%20%EB%B2%A0%ED%83%80%EB%B6%84%ED%8F%AC,%20%EA%B0%90%EB%A7%88%EB%B6%84%ED%8F%AC,%20%EB%94%94%EB%A6%AC%ED%81%B4%EB%A0%88%20%EB%B6%84%ED%8F%AC.ipynb](https://colab.research.google.com/github/datascienceschool/book/blob/master/ds/02 mathematics/08.07 베타분포, 감마분포, 디리클레 분포.ipynb)
25. Overfitting and Regularization - cs.Princeton, accessed July 23, 2025, https://www.cs.princeton.edu/courses/archive/spring19/cos324/files/regularization.pdf
26. AI-1.0X: Machine Learning Regularization: MAP Estimate and Regularization - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=V4UwOdkfbrU
27. Regularization (Baysian approach with Map estimate) - Math Stack Exchange, accessed July 23, 2025, https://math.stackexchange.com/questions/4206444/regularization-baysian-approach-with-map-estimate


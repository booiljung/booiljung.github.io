[딥러닝 모듈 (Modules)](./index.md)
# 배치 정규화 (Batch Normalization, 2015)



심층 신경망(Deep Neural Networks, DNNs)은 계층의 깊이가 증가함에 따라 표현력은 기하급수적으로 향상되지만, 동시에 학습 과정은 본질적으로 불안정해지는 고질적인 문제에 직면한다. 특히 시그모이드(sigmoid)나 하이퍼볼릭 탄젠트(tanh)와 같은 포화 비선형성(saturating nonlinearities)을 활성화 함수로 사용하는 심층 구조에서, 역전파 과정 중 그래디언트가 계층을 거슬러 올라가며 점차 소실되거나(vanishing gradients) 폭주하는(exploding gradients) 현상은 심각한 학습 저해 요인으로 작용한다.1 활성화 함수의 포화 영역, 즉 입력값의 절댓값이 커져 기울기가 0에 가까워지는 구간에 들어서면, 하위 계층으로 전달되는 그래디언트가 거의 사라져 파라미터 업데이트가 사실상 중단된다.1 이러한 학습의 불안정성은 연구자들로 하여금 매우 낮은 학습률(learning rate)을 사용하고, 가중치 초기화(weight initialization)에 극도로 신중을 기하도록 강요했으며, 이는 결국 모델의 수렴 속도를 현저히 저하시키는 결과를 낳았다.1


2015년, Sergey Ioffe와 Christian Szegedy는 이러한 심층 신경망 학습의 어려움을 설명하는 핵심 개념으로 '내부 공변량 변화(Internal Covariate Shift, ICS)'를 제안했다.1 그들은 ICS를 "학습 과정에서 신경망 파라미터가 변함에 따라 발생하는 신경망 활성화 값 분포의 변화"로 명확히 정의했다.5

ICS가 심층 학습에서 근본적인 문제로 작용하는 이유는 신경망의 계층적 구조에 기인한다. 신경망의 각 계층은 이전 계층의 출력(활성화 값)을 자신의 입력으로 사용한다. 학습이 진행되면서 경사 하강법에 의해 이전 계층들의 가중치와 편향이 업데이트되고, 이는 필연적으로 현재 계층이 받는 입력 데이터의 분포를 매 이터레이션마다 변화시킨다.1 이는 마치 흐르는 강물 위에서 끊임없이 위치가 바뀌는 징검다리를 건너려는 것과 같다. 현재 계층은 안정적인 입력 분포를 가정하고 최적의 파라미터를 찾아가려 하지만, 그 기반이 되는 입력 분포 자체가 계속해서 흔들리기 때문에 학습 목표가 불안정해지고 수렴이 어려워진다.5

이러한 분포의 변화는 계층이 깊어질수록 이전 계층들의 변화가 누적되어 더욱 심화되며 2, 결과적으로 모델의 활성화 값들을 포화 영역으로 밀어 넣어 학습을 정체시키는 주된 원인으로 지목되었다.1


Ioffe와 Szegedy는 ICS 문제에 대한 직접적이고 혁신적인 해결책으로 배치 정규화(Batch Normalization, BN)를 제안했다.3 단순히 신경망의 초기 입력 데이터를 정규화하는 기존의 전처리 기법(whitening)을 넘어서, BN의 핵심 아이디어는 **정규화 과정 자체를 모델 아키텍처의 일부로 통합**하는 것이었다.1 이는 정규화 연산을 미분 가능한 변환(differentiable transform)으로 만들어, 역전파 과정에서 손실 함수의 그래디언트가 정규화 과정을 온전히 고려하도록 설계한 패러다임의 전환이었다.

기존에 경사 하강법 최적화 과정과 별개로 정규화를 수행하려는 시도는 모델 발산과 같은 문제를 야기했다. 예를 들어, 선형 변환 $z = Wu+b$ 이후 $z$를 단순히 평균을 빼서 중앙에 맞추면, 편향 $b$에 대한 업데이트($\Delta b$) 효과가 정규화 과정에서 상쇄되어 손실은 변하지 않은 채 $b$ 값만 무한히 커지는 현상이 발생할 수 있었다.1 BN은 정규화를 학습 과정에 내재화함으로써 이러한 문제를 해결했다. BN은 각 학습 미니배치(mini-batch) 단위로 계층 입력의 평균과 분산을 고정시켜 분포를 안정화함으로써 ICS를 줄이고, 이를 통해 심층 신경망 학습을 극적으로 가속화하는 것을 목표로 했다.1 이처럼 BN은 단순한 기법이 아닌, 학습 가능한 아키텍처 구성 요소로서 심층 학습의 안정성과 효율성을 근본적으로 개선한 혁신으로 평가받는다.


배치 정규화는 모델의 상태에 따라, 즉 학습 단계와 추론 단계에서 서로 다른 방식으로 동작한다. 학습 시에는 현재 처리 중인 미니배치의 통계량을 동적으로 활용하여 정규화를 수행하는 반면, 추론 시에는 학습 과정 전반에 걸쳐 누적된 고정된 통계량을 사용하여 결정론적인 변환을 수행한다.


학습 단계에서 배치 정규화는 크기가 $m$인 미니배치 $\mathcal{B} = \{x_1, \dots, x_m\}$에 대해 다음과 같은 4단계 알고리즘을 통해 수행된다. 이 과정은 각 특성(feature) 또는 채널(channel) 차원에 대해 독립적으로 적용된다.

1단계: 미니배치 평균(Mean) 계산

먼저, 미니배치에 포함된 m개 샘플들의 평균을 계산하여 미니배치의 중심을 파악한다.8
$$
\mu_{\mathcal{B}} \leftarrow \frac{1}{m}\sum_{i=1}^{m}x_i
$$
2단계: 미니배치 분산(Variance) 계산

다음으로, 1단계에서 계산한 미니배치 평균을 사용하여 각 샘플이 평균으로부터 얼마나 떨어져 있는지, 즉 데이터의 퍼짐 정도를 나타내는 분산을 계산한다.8
$$
\sigma_{\mathcal{B}}^2 \leftarrow \frac{1}{m}\sum_{i=1}^{m}(x_i - \mu_{\mathcal{B}})^2
$$
3단계: 정규화(Normalization)

계산된 평균과 분산을 이용하여 각 입력 샘플 x_i를 평균이 0, 분산이 1인 분포로 변환한다. 이때 분모에 더해지는 \epsilon은 분산이 0에 가까울 때 발생할 수 있는 수치적 불안정성을 방지하기 위한 매우 작은 양의 상수(e.g., 10^{-5})이다.9
$$
\hat{x}_i \leftarrow \frac{x_i - \mu_{\mathcal{B}}}{\sqrt{\sigma_{\mathcal{B}}^2 + \epsilon}}
$$
4단계: 척도(Scale) 및 이동(Shift) 변환

마지막으로, 정규화된 값 \hat{x}_i에 학습 가능한(trainable) 파라미터인 척도 \gamma(gamma)와 이동 \beta(beta)를 적용하여 최종 출력을 생성한다.8
$$
y_i \leftarrow \gamma\hat{x}_i + \beta \equiv \text{BN}_{\gamma, \beta}(x_i)
$$
여기서 $\gamma$와 $\beta$는 역전파를 통해 다른 모델 파라미터와 함께 학습된다.8 이 두 파라미터는 매우 결정적인 역할을 수행한다. 강제적인 정규화는 신경망의 표현력(representational power)을 제한할 수 있다. 예를 들어, 시그모이드 활성화 함수의 입력을 표준 정규분포로 고정하면 비선형성의 선형적인 구간에만 머물게 될 수 있다.14

$\gamma$와 $\beta$는 네트워크가 이러한 제약에서 벗어나 최적의 분포를 스스로 학습할 수 있도록 유연성을 부여한다. 만약 네트워크가 정규화를 수행하지 않는 항등 변환(identity transform)이 최적이라고 판단하면, $\gamma$를 미니배치의 표준편차($\sqrt{\sigma_{\mathcal{B}}^2 + \epsilon}$)로, $\beta$를 평균($\mu_{\mathcal{B}}$)으로 학습하여 정규화의 효과를 이론적으로 상쇄할 수 있다.14 즉, 네트워크에 정규화를 '무시'할 수 있는 선택권을 부여하는 셈이다. 일반적으로 $\gamma$는 1로, $\beta$는 0으로 초기화하여 학습을 시작한다.15

또한, BN 계층을 사용하는 경우 이전 선형 계층($z=Wx+b$)의 편향 파라미터 $b$는 일반적으로 생략된다. 그 이유는 BN의 이동 파라미터 $\beta$가 편향 $b$의 역할을 완전히 대체할 수 있기 때문이다. 정규화 과정에서 미니배치의 평균 $\mu_{\mathcal{B}}$를 빼는 연산은 $b$의 효과를 무효화하며, 이후 더해지는 $\beta$가 새로운 편향으로 기능한다. 따라서 기존의 편향 $b$는 중복(redundant) 파라미터가 되어 제거함으로써 모델을 더 간결하게 만들 수 있다.9


모델 학습이 완료된 후, 추론(또는 테스트) 단계에서는 학습 시와 같은 미니배치가 존재하지 않거나, 실시간 처리 등에서 배치 크기가 1인 경우가 대부분이다. 이러한 상황에서는 미니배치 단위의 평균과 분산을 계산하는 것이 불가능하거나 무의미하다.10 따라서 추론 시에는 안정적이고 결정론적인 출력을 보장하기 위해 다른 접근 방식이 필요하다.

이 문제를 해결하기 위해, BN은 학습 과정 동안 전체 훈련 데이터셋의 통계량에 대한 추정치를 계산하여 저장한다. 이는 일반적으로 **지수 이동 평균(exponentially moving average)** 방식을 통해 이루어진다. 매 학습 이터레이션에서 계산된 미니배치의 평균 $\mu_{\mathcal{B}}$와 분산 $\sigma_{\mathcal{B}}^2$을 사용하여 전역 통계량(population statistics)을 다음과 같이 점진적으로 업데이트한다.9
$$
\text{running\_mean} \leftarrow \text{momentum} \times \text{running\_mean} + (1 - \text{momentum}) \times \mu_{\mathcal{B}}
$$

$$
\text{running\_var} \leftarrow \text{momentum} \times \text{running\_var} + (1 - \text{momentum}) \times \sigma_{\mathcal{B}}^2
$$

여기서 $\text{momentum}$은 보통 0.9, 0.99와 같이 1에 가까운 값으로 설정되어, 과거의 통계량을 비중 있게 반영하면서 새로운 배치의 정보를 부드럽게 통합한다.17

학습이 완료되면, 이렇게 누적된 이동 평균($\text{running\_mean}$)과 이동 분산($\text{running\_var}$)이 각각 전체 데이터셋의 평균 $E[x]$와 분산 $\text{Var}[x]$의 추정치로 사용된다. 추론 시에는 이 고정된 값들을 사용하여 입력을 정규화한다.
$$
\hat{x} \leftarrow \frac{x - E[x]}{\sqrt{\text{Var}[x] + \epsilon}}
$$
그리고 학습을 통해 최적화된 $\gamma$와 $\beta$ 파라미터를 적용하여 최종 출력을 계산한다.17
$$
y \leftarrow \gamma\hat{x} + \beta
$$
결과적으로, 추론 단계에서 BN 계층은 각 입력에 대해 동일한 선형 변환을 수행하는 결정론적(deterministic)인 연산으로 작동한다. 이는 학습 시에는 미니배치 내 다른 샘플에 따라 출력이 달라지는 확률적(stochastic) 연산이었던 것과 대조된다. 이처럼 정적인 변환은 추론 시 성능 최적화를 위해 이전의 선형 계층(예: 합성곱 계층)과 융합(fused)되어 계산 효율성을 높이는 데 활용될 수 있다.21

한 가지 주목할 점은 원본 논문에서 학습 시 분산 계산에는 편향된(biased) 추정량($\frac{1}{m}$로 나눔)을 사용하고, 추론 시 사용할 전역 분산 추정에는 비편향(unbiased) 추정량($\text{Var}[x] = \frac{m}{m-1} E_{\mathcal{B}}$)을 사용한다는 미묘한 불일치가 존재한다는 것이다.10 이 차이는 이론적으로 학습과 추론 간의 미세한 괴리를 유발할 수 있어 논의의 대상이 되었으나, 저자들은 실험적으로 논문에 제시된 방식이 더 나은 결과를 보였다고 밝혔으며, 실제 성능에 미치는 영향은 크지 않은 것으로 알려져 있다.10


배치 정규화는 도입 이후 심층 학습 커뮤니티에 빠르고 광범위하게 채택되었으며, 그 이유는 다각적인 측면에서 신경망의 학습 과정과 성능을 획기적으로 개선했기 때문이다. 주요 효용성은 학습 속도 가속화, 초기화에 대한 민감도 감소, 그리고 모델 일반화 성능을 높이는 규제 효과로 요약할 수 있다.


배치 정규화의 가장 두드러지고 즉각적인 장점은 심층 신경망의 학습 속도를 극적으로 가속화한다는 것이다.3 Ioffe와 Szegedy의 원본 논문에서는 BN을 적용한 이미지 분류 모델이 당시 최고 성능을 보이던 Inception 모델과 동일한 정확도를 달성하는 데 **14배나 적은 학습 단계**를 필요로 했다고 보고했다.1 이러한 가속화는 BN이 학습 과정을 근본적으로 안정시키기 때문에 가능하다.

BN은 각 계층 입력의 평균과 분산을 일정하게 유지함으로써, 그래디언트가 파라미터의 스케일이나 초기값에 과도하게 의존하는 문제를 완화한다.1 일반적인 심층 신경망에서는 학습률이 조금만 높아도 그래디언트가 폭주하거나 소실되어 학습이 발산하기 쉽지만, BN을 적용하면 활성화 값의 범위가 제어되므로 이러한 위험 없이 훨씬 높은 학습률을 사용할 수 있게 된다.1

실험적으로도 이러한 효과는 명확히 입증되었다. Inception 모델을 기준으로 BN을 적용한 여러 변형 모델을 비교한 실험에서, 기준 모델과 동일한 학습률을 사용한 'BN-Baseline' 모델은 훨씬 빠르게 수렴했다. 더 나아가, 학습률을 5배 높인 'BN-x5' 모델은 기준 모델이 72.2% 정확도에 도달하는 데 필요했던 단계의 약 7%만으로 동일한 성능을 달성했다. 학습률을 30배 높인 'BN-x30' 모델은 초기에는 다소 느렸지만 결국 더 높은 최종 정확도에 도달하며 BN의 안정성을 증명했다.2 이처럼 높은 학습률을 사용할 수 있다는 것은 손실 함수의 평탄한 지역에서 더 빠르고 효율적인 탐색을 가능하게 하여, 전체적인 학습 시간을 단축시키는 핵심 요인이다.25


심층 신경망 학습은 가중치 초기값에 매우 민감하여, 부적절한 초기화는 학습 실패의 직접적인 원인이 되기도 한다. 이 때문에 Xavier/Glorot 초기화나 He 초기화와 같은 정교한 기법들이 개발되었다. 배치 정규화는 학습 과정에서 각 계층의 입력을 지속적으로 재조정하고 안정화시키기 때문에, 가중치 초기화 방식에 대한 의존도를 크게 낮춘다.26 즉, 상대적으로 덜 신중한 초기화 전략을 사용하더라도 모델이 안정적으로 학습을 시작하고 수렴할 수 있게 만들어, 모델 설계와 튜닝 과정을 훨씬 용이하게 한다.23


배치 정규화는 부수적으로 모델의 일반화 성능을 향상시키는 규제(regularization) 효과를 가진다.22 이 효과는 매우 강력하여, 경우에 따라 과적합을 방지하기 위해 널리 사용되던 드롭아웃(Dropout) 기법의 필요성을 없애거나 그 강도를 줄일 수 있게 한다.1

BN의 규제 효과는 학습 과정의 본질적인 특성에서 비롯된다. 학습 시, 특정 샘플에 대한 BN 계층의 출력은 해당 샘플뿐만 아니라 **동일한 미니배치에 포함된 다른 모든 샘플들의 통계량(평균, 분산)에 의존**한다. 이는 각 훈련 샘플에 대해 모델이 더 이상 결정론적인(deterministic) 값을 생성하지 않음을 의미한다.23 매번 미니배치가 무작위로 구성됨에 따라, 동일한 샘플이라도 다른 배치에 속하면 약간 다른 통계량의 영향을 받아 출력이 미세하게 변동한다. 이러한 미니배치 통계량에서 발생하는 미세한 잡음(noise)은 모델이 특정 훈련 샘플이나 그에 따른 활성화 패턴에 과도하게 의존하는 것을 방지하는 역할을 한다.31 결과적으로 모델은 더 강건한(robust) 특징을 학습하게 되어 보지 못한 데이터에 대한 일반화 성능이 향상된다.

이 세 가지 주요 이점—빠른 학습, 초기화 자유, 규제—은 서로 독립적이라기보다는 상호 유기적으로 연결되어 있다. BN이 제공하는 근본적인 '학습 안정화'가 그래디언트 흐름을 원활하게 하여 높은 학습률을 허용하고(빠른 학습), 초기값의 영향을 줄이며(초기화 자유), 미니배치의 확률적 특성이 잡음으로 작용하여(규제) 모델의 일반화 능력을 높이는 것이다.


배치 정규화가 처음 제안되었을 때, 그 놀라운 성공은 '내부 공변량 변화(ICS) 감소'라는 직관적이고 설득력 있는 가설로 설명되었다. 그러나 심층 학습 연구가 발전함에 따라 이 초기 가설은 도전을 받게 되었고, BN의 작동 원리를 더 근본적인 관점에서 설명하려는 새로운 패러다임이 등장했다.


Ioffe와 Szegedy가 제안한 ICS 가설은 BN의 효과를 설명하는 표준 이론으로 오랫동안 받아들여졌다.33 그러나 2018년 Santurkar 등이 발표한 "How Does Batch Normalization Help Optimization? (No, It Is Not About Internal Covariate Shift)"라는 논문은 이 통념에 정면으로 도전하며 학계에 큰 파장을 일으켰다.34

이 연구팀은 BN이 실제로 ICS를 줄이는지를 엄밀하게 측정하는 실험을 설계했다. 놀랍게도, 실험 결과는 BN을 적용한 네트워크가 ICS를 유의미하게 줄이지 않으며, 심지어 특정 조건에서는 **ICS를 오히려 증가시킨다**는 것을 보여주었다.35 더욱 결정적인 증거는 BN 계층 바로 뒤에 의도적으로 평균이 0이 아니고 분산이 1이 아닌 확률적 잡음을 주입하여 계층 입력의 분포를 매 순간 불안정하게 만든 실험에서 나왔다. ICS 가설에 따르면 이러한 'noisy BN' 모델은 성능이 크게 저하되어야 하지만, 실제로는 표준 BN 모델과 거의 동일한 수준의 뛰어난 성능을 유지했으며, BN이 없는 표준 네트워크보다는 월등히 우수했다.34

이러한 실험 결과들은 BN의 성공이 계층 입력 분포의 안정성과는 거의 관련이 없으며, ICS 감소가 BN 효과의 핵심 원인이 아닐 수 있다는 강력한 증거를 제시했다.34 이는 BN의 작동 원리에 대한 근본적인 재검토의 필요성을 시사했다.


ICS 가설을 반증한 Santurkar 등은 BN의 진정한 효과가 다른 곳에 있다고 주장했다. 그들은 BN이 최적화 문제 자체의 기하학적 구조에 근본적인 영향을 미치며, 특히 **최적화 지형(optimization landscape)을 훨씬 더 평탄하게(smoother) 만든다**는 새로운 가설을 제시했다.34

최적화 지형이 평탄하다는 것은 손실 함수의 변화가 더 예측 가능함을 의미한다. 수학적으로 이는 손실 함수와 그래디언트의 립시츠 상수(Lipschitz constant)가 개선됨을 뜻한다.33 BN이 없는 심층 네트워크의 손실 지형은 매우 불규칙하고, 급경사와 좁은 골짜기가 많은 험준한 산악 지형과 같아서, 경사 하강법 스텝이 조금만 커져도 절벽으로 떨어지거나(발산) 엉뚱한 방향으로 튀기 쉽다.37 반면, BN은 네트워크를 효과적으로 재매개변수화(reparameterize)하여 이 험준한 지형을 완만한 언덕처럼 만들어준다.37

이렇게 평탄화된 지형에서는 현재 위치에서의 그래디언트가 더 먼 거리까지 유효한 방향 정보를 제공한다. 즉, 그래디언트가 더 예측 가능하고 안정적으로 행동하게 되어, 더 큰 학습률(더 큰 스텝)을 사용하여도 최적점을 향해 안정적으로 나아갈 수 있다.34 이것이 BN이 높은 학습률을 허용하고 학습을 가속화하는 더 근본적인 이유라는 것이다.


현재 학계에서는 '최적화 지형 평탄화' 가설이 BN의 작동 원리를 더 본질적으로 설명하는 이론으로 널리 받아들여지는 추세이다. BN의 발견은 'ICS'라는 올바른 문제를 해결하려다 '불규칙한 최적화 지형'이라는 더 근본적인 문제에 대한 해결책을 우연히 찾아낸 과학적 발견의 한 사례로 볼 수 있다. BN의 가치는 그것이 처음 의도했던 문제를 해결했는지 여부와는 무관하게, 실제로 심층 학습의 최적화 문제를 해결하는 데 탁월한 능력을 보였다는 점에 있다.

더 나아가, 최적화 지형 평탄화는 BN이 왜 더 나은 일반화 성능을 보이는지에 대한 깊은 통찰을 제공한다. 평탄화된 지형은 더 높은 학습률을 가능하게 하고, 높은 학습률은 확률적 경사 하강법(SGD)에서 더 큰 잡음(noise)을 유발하는 효과가 있다.25 이러한 최적화 과정의 잡음은 모델이 좁고 뾰족한(sharp) 지역 최적점에 과적합되는 것을 방지하고, 더 넓고 평평한(flat) 최적점을 찾도록 유도한다. 신경망 연구에서 넓고 평평한 최적점은 더 나은 일반화 성능과 강하게 연관되어 있음이 알려져 있다. 따라서 'BN → 지형 평탄화 → 높은 학습률 허용 → 넓은 최적점 탐색 → 일반화 성능 향상'이라는 인과 관계 사슬을 통해 BN의 다각적인 이점을 통합적으로 이해할 수 있다.


배치 정규화를 실제 모델에 적용할 때는 그 효과를 극대화하고 잠재적인 문제를 피하기 위해 몇 가지 중요한 사항을 고려해야 한다. 특히 BN 계층의 위치와 배치 크기 설정은 모델 성능에 직접적인 영향을 미칠 수 있다.


BN 계층을 활성화 함수(예: ReLU)의 앞에 위치시킬 것인가, 아니면 뒤에 위치시킬 것인가에 대한 논쟁은 오랫동안 커뮤니티에서 논의되어 온 주제이다.40

- **활성화 함수 이전 (BN → ReLU):** 이는 Ioffe와 Szegedy의 원본 논문에서 제안된 방식으로, 오랫동안 표준적인 구현으로 여겨졌다. 이 방식의 논리적 근거는 선형 변환($Wx+b$)을 거친 직후의 출력이 비선형 활성화 함수를 통과한 후의 출력보다 더 대칭적이고 가우시안 분포에 가까울 가능성이 높다는 것이다.21 따라서 이 시점에서 정규화를 수행하는 것이 활성화 값의 분포를 안정시키는 데 더 효과적이며, 특히 ReLU와 같이 입력 값의 일부를 잘라내는(truncate) 활성화 함수에 안정적인 입력을 제공할 수 있다는 장점이 있다.7
- **활성화 함수 이후 (ReLU → BN):** 반면, 실제 구현에서는 활성화 함수 이후에 BN을 배치하는 것이 더 나은 성능을 보인다는 경험적 보고가 다수 존재한다.21 Keras의 창시자인 프랑수아 숄레는 BN 논문의 저자 중 한 명인 크리스티안 세게디조차도 최근 코드에서는 ReLU 이후에 BN을 적용한다고 언급한 바 있다.21 이 방식의 직관적인 근거는 다음과 같다. BN을 ReLU 이전에 적용하면, 정규화를 통해 평균이 0이 된 분포의 음수 영역 절반이 ReLU에 의해 강제로 0으로 만들어져 분포가 다시 심하게 왜곡된다. 반면, ReLU를 먼저 적용한 후 BN을 사용하면, 0 이상의 양수 값으로만 구성된 활성화 분포를 직접 정규화하여 다음 계층에 더 안정적인 입력을 전달할 수 있다는 주장이다.21

현재까지 어느 한쪽이 모든 경우에 절대적으로 우월하다는 명확한 이론적, 실험적 합의는 이루어지지 않았다.41 두 방식 모두 널리 사용되고 있으며, 모델 아키텍처, 데이터셋, 그리고 다른 하이퍼파라미터와의 상호작용에 따라 최적의 위치가 달라질 수 있다. 따라서 실제 적용 시에는 두 가지 방식을 모두 실험해보고 검증 성능이 더 높은 쪽을 선택하는 것이 가장 실용적인 접근법이다.42


배치 정규화의 가장 본질적이고 잘 알려진 단점은 그 성능이 **미니배치의 크기에 크게 의존한다**는 점이다.44 BN은 미니배치 내의 샘플들로부터 평균과 분산을 추정하여 정규화를 수행하는데, 이 추정치의 신뢰도는 미니배치의 크기와 직접적인 관련이 있다.

배치 크기가 충분히 클 경우(예: 32 이상), 미니배치의 통계량은 전체 훈련 데이터셋의 통계량을 비교적 잘 근사할 수 있다. 그러나 배치 크기가 매우 작아지면(예: 2, 4, 8), 미니배치의 평균과 분산은 샘플링 편향으로 인해 매우 불안정해지고 큰 잡음을 포함하게 된다. 이는 학습 과정을 불안정하게 만들고 모델의 최종 성능을 심각하게 저하시키는 원인이 된다.46

이러한 배치 크기 의존성 문제는 특히 다음과 같은 분야에서 심각한 제약으로 작용한다:

- **고해상도 이미지 처리:** 객체 탐지(Object Detection), 의미론적 분할(Semantic Segmentation) 등 고해상도 이미지를 입력으로 사용하는 컴퓨터 비전 과제에서는 GPU 메모리 제약으로 인해 큰 배치 크기를 사용하기가 현실적으로 어렵다.46
- **비디오 분석:** 비디오 데이터는 이미지 시퀀스로 구성되어 메모리 요구량이 매우 크기 때문에 작은 배치가 불가피하다.47
- **순환 신경망(RNN):** 가변적인 시퀀스 길이를 처리하는 RNN에 BN을 적용하는 것은 개념적으로 복잡하고, 각 타임스텝마다 다른 통계량을 계산하고 저장해야 하는 문제가 있어 효과적이지 않다.44

이러한 명확한 한계는 배치 정규화의 대안 기술들이 활발히 연구되고 제안되는 주된 동기가 되었다. 예를 들어, 저자 자신도 작은 배치 문제에 대응하기 위해 배치 재정규화(Batch Renormalization)라는 개선된 기법을 제안한 바 있다.49


배치 정규화의 성공과 그 명백한 한계점, 특히 배치 크기 의존성 문제는 다양한 대안 정규화 기법의 등장을 촉발했다. 이 기법들은 대부분 정규화를 위한 통계량(평균, 분산)을 계산하는 단위를 달리함으로써 특정 문제 상황에 더 적합하도록 설계되었다. 핵심적인 차이는 텐서의 어떤 차원(axis)을 따라 통계량을 계산하고 공유하는가에 있다.


레이어 정규화(LN)는 배치 정규화의 배치 의존성 문제를 해결하기 위해 제안되었다.44

- **동작 원리:** LN은 배치 차원(N)을 따라 통계량을 계산하는 대신, 각 데이터 샘플 내에서 독립적으로 모든 채널(C) 또는 특성(feature) 차원에 걸쳐 평균과 분산을 계산한다.44 즉, 정규화가 배치 내 다른 샘플의 영향을 받지 않고 오직 해당 샘플의 모든 특성 값들에 의해서만 결정된다.
- **핵심 차이:** 가장 큰 차이점은 배치 크기에 전혀 의존하지 않는다는 것이다. 따라서 배치 크기가 1이거나, RNN처럼 시퀀스마다 길이가 달라 실질적인 배치 구성이 어려운 경우에도 안정적으로 작동한다.44 또한, 학습 시와 추론 시의 계산 방식이 동일하여 구현이 간단하고 두 단계 간의 불일치 문제가 발생하지 않는다.44
- **주요 적용 분야:** 이러한 특성 덕분에 LN은 순환 신경망(RNN, LSTM)과 트랜스포머(Transformer)와 같은 자연어 처리(NLP) 모델에서 표준적인 정규화 기법으로 자리 잡았다.52


인스턴스 정규화(IN)는 주로 이미지 생성 및 변환 과제에서 특정 목적을 달성하기 위해 개발되었다.

- **동작 원리:** IN은 각 데이터 샘플(N)과 각 채널(C)에 대해 독립적으로 정규화를 수행한다. 통계량은 오직 공간적 차원(H, W)에 대해서만 계산된다.54 즉, 하나의 이미지, 하나의 채널 맵 내에서만 평균과 분산이 계산된다.
- **핵심 차이:** 정규화 단위가 가장 작다. 배치 내 다른 샘플은 물론, 동일 샘플 내 다른 채널의 정보조차 사용하지 않는다. 이 방식은 각 이미지 인스턴스의 고유한 스타일 정보(예: 대비, 밝기, 색감 등)를 정규화를 통해 효과적으로 제거하는 경향이 있다.
- **주요 적용 분야:** 이러한 특성 때문에 IN은 이미지의 내용(content)은 보존하면서 스타일(style) 정보만을 바꾸는 **스타일 전이(Style Transfer)** 분야에서 매우 효과적인 것으로 입증되었다.32 또한, 이미지 생성 모델(GAN)에서도 생성된 이미지의 품질을 향상시키는 데 널리 사용된다.


그룹 정규화(GN)는 BN의 강력한 성능과 LN/IN의 배치 독립성이라는 장점을 결합하려는 시도에서 탄생했다.46

- **동작 원리:** GN은 채널(C) 차원을 여러 개의 그룹(G)으로 나눈다. 그리고 각 데이터 샘플 내에서, 각 그룹별로 채널들과 공간적 차원(H, W)에 걸쳐 평균과 분산을 계산하여 정규화를 수행한다.47
- **핵심 차이:** GN은 LN과 IN의 절충안으로 볼 수 있다. 만약 그룹의 수가 1이라면($G=1$), 모든 채널이 하나의 그룹에 속하므로 LN과 동일하게 동작한다. 반대로 그룹의 수가 채널의 수와 같다면($G=C$), 각 채널이 하나의 그룹이 되므로 IN과 동일하게 동작한다.55 이처럼 그룹 수를 조절함으로써 정규화의 단위를 유연하게 설정할 수 있다.
- **주요 적용 분야:** GN은 배치 크기에 의존하지 않으면서도, 순수 LN이나 IN보다 일반적인 컴퓨터 비전 과제(이미지 분류, 객체 탐지 등)에서 더 나은 성능을 보이는 경향이 있다. 특히, 메모리 제약으로 인해 작은 배치 크기를 사용해야 하는 상황에서 BN의 강력한 대안으로 부상했다.46


다음 표는 네 가지 주요 정규화 기법의 핵심적인 차이점을 요약하여 보여준다. 여기서 입력 텐서의 차원은 (N, C, H, W)로 가정하며, 각각 배치, 채널, 높이, 너비를 의미한다. $S_g$는 그룹 $g$에 속한 채널의 집합을 나타낸다.

| 특성 (Feature)       | **배치 정규화 (Batch Normalization)**          | **레이어 정규화 (Layer Normalization)**        | **인스턴스 정규화 (Instance Normalization)** | **그룹 정규화 (Group Normalization)**                       |
| -------------------- | ---------------------------------------------- | ---------------------------------------------- | -------------------------------------------- | ----------------------------------------------------------- |
| **정규화 축**        | 배치(N), 공간(H, W) 차원                       | 채널(C), 공간(H, W) 차원                       | 공간(H, W) 차원                              | 그룹 내 채널(C/G), 공간(H, W) 차원                          |
| **배치 크기 의존성** | **높음 (High)**                                | 없음 (None)                                    | 없음 (None)                                  | 없음 (None)                                                 |
| **학습/추론 동작**   | 다름 (Different)                               | 같음 (Same)                                    | 같음 (Same)                                  | 같음 (Same)                                                 |
| **주요 적용 분야**   | 컴퓨터 비전 (CNNs, 큰 배치)                    | 순환 신경망 (RNNs), 트랜스포머                 | 스타일 전이, 이미지 생성 (GANs)              | 컴퓨터 비전 (CNNs, 작은 배치)                               |
| **장점**             | 강력한 성능, 규제 효과                         | 배치 크기 무관, RNN에 적합                     | 스타일 정보 제거에 효과적                    | 배치 크기 무관, 안정적인 성능                               |
| **단점**             | 작은 배치에서 성능 저하, RNN 적용 어려움       | CNN에서 BN보다 성능이 낮은 경향                | 내용 보존에만 집중, 분류 성능 저하 가능      | 그룹 수 하이퍼파라미터 필요                                 |
| **수식 (평균)**      | $\mu_{c} = \frac{1}{NHW}\sum_{n,h,w} x_{nchw}$ | $\mu_{n} = \frac{1}{CHW}\sum_{c,h,w} x_{nchw}$ | $\mu_{nc} = \frac{1}{HW}\sum_{h,w} x_{nchw}$ | $\mu_{ng} = \frac{G}{C H W}\sum_{c \in S_g, h, w} x_{nchw}$ |



배치 정규화는 2015년 등장 이래 심층 신경망 학습 분야에 기념비적인 기여를 한 기법으로 평가된다. 이는 학습 과정을 안정화하고 수렴 속도를 극적으로 가속화함으로써, 이전에는 학습이 매우 어려웠던 훨씬 더 깊은 네트워크의 설계를 가능하게 했다.22 그 효과는 수많은 연구와 실제 적용 사례를 통해 명백히 입증되었으며, 오늘날에도 여전히 다수의 최첨단(State-of-the-Art) 모델에서 핵심 구성 요소로 활용되고 있다.46

그러나 BN에 대한 이해는 시간이 지나면서 진화했다. 초기에 제안된 '내부 공변량 변화(ICS) 감소'라는 작동 원리 가설은 후속 연구를 통해 도전을 받았고, 현재는 '최적화 지형의 평탄화'가 그 효과를 더 근본적으로 설명하는 이론으로 받아들여지고 있다. 이와 동시에, '배치 크기 의존성'이라는 명확하고 실용적인 한계는 특정 응용 분야에서의 사용을 제약했으며, 이는 레이어 정규화, 인스턴스 정규화, 그룹 정규화와 같은 다양한 대안 기술의 등장을 촉발하는 계기가 되었다.


현재 다양한 정규화 기법이 공존함에 따라, 당면한 과제와 모델 아키텍처, 그리고 하드웨어 제약 조건에 맞는 최적의 기법을 선택하는 것이 중요해졌다. 일반적인 가이드라인은 다음과 같이 요약할 수 있다.

- **큰 배치 크기가 가능한 CNN 기반 비전 과제:** 이미지 분류와 같이 GPU 메모리가 허용하는 한 충분히 큰 배치 크기(예: 32 이상)를 사용할 수 있는 전통적인 컴퓨터 비전 문제에서는 여전히 배치 정규화가 가장 강력하고 신뢰할 수 있는 첫 번째 선택지이다.51
- **작은 배치 크기가 불가피한 비전 과제:** 객체 탐지, 시맨틱 분할 등 고해상도 입력을 처리해야 하거나 모델 크기가 매우 커서 메모리 제약이 심한 경우, 배치 크기에 독립적인 그룹 정규화가 BN보다 훨씬 안정적이고 우수한 성능을 제공하는 대안이 된다.47
- **RNN, 트랜스포머 등 시퀀스 데이터 처리:** 자연어 처리나 시계열 분석과 같이 입력 시퀀스의 길이가 가변적이고 배치 차원의 의미가 다른 순환적이거나 어텐션 기반의 모델에서는, 배치 크기에 무관하고 학습-추론 간 동작이 일관된 레이어 정규화가 사실상의 표준으로 사용된다.51
- **스타일 전이 등 생성 모델:** 이미지의 스타일과 콘텐츠를 분리하여 조작하는 것이 중요한 스타일 전이나 특정 이미지 생성 과제에서는, 인스턴스별 스타일 정보를 효과적으로 정규화하는 인스턴스 정규화가 특화된 성능을 보인다.32


배치 정규화와 그 대안들의 성공은 신경망 정규화가 모델 성능에 미치는 지대한 영향을 입증했다. 앞으로의 연구는 더욱 정교하고 효율적인 정규화 기법을 개발하는 방향으로 나아갈 것으로 전망된다. BN의 작동 원리에 대한 이론적 분석은 여전히 활발히 진행 중이며 33, 이는 최적화 과정에 대한 더 깊은 이해를 바탕으로 한 새로운 알고리즘의 개발로 이어질 수 있다.

또한, 단일 정규화 기법에 의존하기보다 여러 기법의 장점을 결합하려는 시도(예: Batch-Instance Normalization 56, Batch Layer Normalization 59)나, 데이터의 특성이나 학습의 특정 단계에 따라 정규화 방식을 동적으로 조절하는 적응형(adaptive) 정규화 기법에 대한 연구가 미래의 중요한 방향이 될 것이다. 궁극적으로 이러한 노력은 더욱 강건하고, 효율적이며, 다양한 조건에서 안정적으로 학습 가능한 심층 신경망 아키텍처의 발전에 기여할 것이다.


1. Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1502.03167
2. Internal Covariate Shift: How Batch Normalization can speed up Neural Network Training | by Jamie Dowat | Analytics Vidhya | Medium, 8월 24, 2025에 액세스, https://medium.com/analytics-vidhya/internal-covariate-shift-an-overview-of-how-to-speed-up-neural-network-training-3e2a3dcdd5cc
3. [1502.03167] Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/1502.03167
4. Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/272194743_Batch_Normalization_Accelerating_Deep_Network_Training_by_Reducing_Internal_Covariate_Shift
5. Internal covariate shift - Machine Learning Glossary, 8월 24, 2025에 액세스, https://machinelearning.wtf/terms/internal-covariate-shift/
6. Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift - Google Research, 8월 24, 2025에 액세스, http://research.google.com/pubs/archive/43442.pdf
7. 배치 정규화(Batch Normalization)의 설명과 탐구, 8월 24, 2025에 액세스, https://mole-starseeker.tistory.com/45
8. 배치 정규화(Batch Normalization) - gaussian37 - JINSOL KIM, 8월 24, 2025에 액세스, https://gaussian37.github.io/dl-concept-batchnorm/
9. [pytorch] 학습의 가속화를 위한 배치 정규화(Batch Normalization) - resultofeffort - 티스토리, 8월 24, 2025에 액세스, https://resultofeffort.tistory.com/128
10. BatchNorm and the curious case of training vs. inference variance - Peter Chng, 8월 24, 2025에 액세스, https://peterchng.com/blog/2023/08/21/batchnorm-and-the-curious-case-of-training-vs.-inference-variance/
11. Batch Normalization: Theory and TensorFlow Implementation - DataCamp, 8월 24, 2025에 액세스, https://www.datacamp.com/tutorial/batch-normalization-tensorflow
12. Batch Normalization Explained. “You may have heard the saying, 'A… | by Amit Yadav | Biased-Algorithms | Medium, 8월 24, 2025에 액세스, https://medium.com/biased-algorithms/batch-normalization-explained-04bbf268a031
13. [Deep Learning] Batch Normalization (배치 정규화), 8월 24, 2025에 액세스, https://eehoeskrap.tistory.com/430
14. neural networks - Do $\gamma$ and $\beta$ “undo” the eﬀects of ..., 8월 24, 2025에 액세스, [https://stats.stackexchange.com/questions/396838/do-gamma-and-beta-undo-the-e%EF%AC%80ects-of-batch-normalization](https://stats.stackexchange.com/questions/396838/do-gamma-and-beta-undo-the-eﬀects-of-batch-normalization)
15. Initialization of Gamma and Beta in Batch Normalization in neural networks - Stack Overflow, 8월 24, 2025에 액세스, https://stackoverflow.com/questions/62216100/initialization-of-gamma-and-beta-in-batch-normalization-in-neural-networks
16. machine learning - Why does Batch Normalization need moving ..., 8월 24, 2025에 액세스, https://stats.stackexchange.com/questions/229837/why-does-batch-normalization-need-moving-averages-besides-to-track-model-accurac
17. How does batch normalisation actually work? - AI Stack Exchange, 8월 24, 2025에 액세스, https://ai.stackexchange.com/questions/22433/how-does-batch-normalisation-actually-work
18. How and why does Batch Normalization use moving averages to track the accuracy of the model as it trains? - Cross Validated, 8월 24, 2025에 액세스, https://stats.stackexchange.com/questions/219808/how-and-why-does-batch-normalization-use-moving-averages-to-track-the-accuracy-o
19. BatchNorm: Fine-Tune your Booster | by Ilango Rajagopal | Medium, 8월 24, 2025에 액세스, https://medium.com/@ilango100/batchnorm-fine-tune-your-booster-bef9f9493e22
20. Reason for Batch normalization at Test Time - DeepLearning.AI, 8월 24, 2025에 액세스, https://community.deeplearning.ai/t/reason-for-batch-normalization-at-test-time/552615
21. [D] Batch Normalization before or after ReLU? : r/MachineLearning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/67gonq/d_batch_normalization_before_or_after_relu/
22. 8.5. Batch Normalization — Dive into Deep Learning 1.0.3 documentation, 8월 24, 2025에 액세스, http://d2l.ai/chapter_convolutional-modern/batch-norm.html
23. The Role of Batch Normalization in CNNs - Viso Suite, 8월 24, 2025에 액세스, https://viso.ai/deep-learning/batch-normalization/
24. Why would batch normalization allows us to use higher learning rate in the neural network?, 8월 24, 2025에 액세스, https://datascience.stackexchange.com/questions/64070/why-would-batch-normalization-allows-us-to-use-higher-learning-rate-in-the-neura
25. Understanding Batch Normalization, 8월 24, 2025에 액세스, http://papers.neurips.cc/paper/7996-understanding-batch-normalization.pdf
26. [딥러닝 논문 리뷰 - PRMI lab] - 배치 정규화(Batch Normalization) + 보편적 근사 정리(Universal Approximation Theorem) - 현서의 개발 일지, 8월 24, 2025에 액세스, https://hyunseo-fullstackdiary.tistory.com/368
27. What is Batch Normalization - Deepchecks, 8월 24, 2025에 액세스, https://www.deepchecks.com/glossary/batch-normalization/
28. Why Batch Normalization Matters for Deep Learning | Towards Data Science, 8월 24, 2025에 액세스, https://towardsdatascience.com/why-batch-normalization-matters-for-deep-learning-3e5f4d71f567/
29. Understanding Batch Normalization in Deep Learning: A Beginner's Guide - Medium, 8월 24, 2025에 액세스, https://medium.com/@piyushkashyap045/understanding-batch-normalization-in-deep-learning-a-beginners-guide-40917c5bebc8
30. Review of Ioffe & Szegedy 2015 *Batch normalization* - neural.vision, 8월 24, 2025에 액세스, https://neural.vision/blog/article-reviews/deep-learning/ioffe-batch-2015/
31. Regularization: Batch-normalization and Drop out | by aditi kothiya | Analytics Vidhya, 8월 24, 2025에 액세스, https://medium.com/analytics-vidhya/everything-you-need-to-know-about-regularizer-eb477b0c82ba
32. The Different Types of Normalizations in Deep Learning | by DZ - Medium, 8월 24, 2025에 액세스, https://dzdata.medium.com/the-different-types-of-normalizations-in-deep-learning-03eece7fa789
33. Batch normalization - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Batch_normalization
34. How Does Batch Normalization Help Optimization? - NIPS, 8월 24, 2025에 액세스, http://papers.neurips.cc/paper/7515-how-does-batch-normalization-help-optimization.pdf
35. [R] Debunking one of the most misunderstood concepts in Deep Learning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/8yny9g/r_debunking_one_of_the_most_misunderstood/
36. How Does Batch Normalization Help Optimization?, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1805.11604
37. Why does Batch Normalization work ? | by Pranjal Khadka - Towards AI, 8월 24, 2025에 액세스, https://pub.towardsai.net/why-does-batch-normalization-work-3a7f093070ff
38. MIT Open Access Articles How does batch normalization help optimization?, 8월 24, 2025에 액세스, https://dspace.mit.edu/bitstream/handle/1721.1/137779/7515-how-does-batch-normalization-help-optimization.pdf?sequence=2&isAllowed=y
39. Batch Normalization in Neural Network Simply Explained | by Anthony Kwok - Medium, 8월 24, 2025에 액세스, https://kwokanthony.medium.com/batch-normalization-in-neural-network-simply-explained-115fe281f4cd
40. Why perform batch norm before ReLu and not after? - Deep Learning - Fast.ai forums, 8월 24, 2025에 액세스, https://forums.fast.ai/t/why-perform-batch-norm-before-relu-and-not-after/81293
41. Batch Normalization and Activation function Sequence Confusion | by Nihar Kanungo, 8월 24, 2025에 액세스, https://niharkanungo.medium.com/batch-normalization-and-activation-function-sequence-confusion-4e075334b4cc
42. Batch normalization before or after activation function : r/deeplearning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/deeplearning/comments/lqy89u/batch_normalization_before_or_after_activation/
43. python - Ordering of batch normalization and dropout? - Stack ..., 8월 24, 2025에 액세스, https://stackoverflow.com/questions/39691902/ordering-of-batch-normalization-and-dropout
44. Layer Normalization - Department of Computer Science, University ..., 8월 24, 2025에 액세스, https://arxiv.org/abs/1607.06450
45. Understanding the Impact of Batch Normalization on CNNs - PingCAP, 8월 24, 2025에 액세스, https://www.pingcap.com/article/understanding-the-impact-of-batch-normalization-on-cnns/
46. Group Normalization - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content_ECCV_2018/papers/Yuxin_Wu_Group_Normalization_ECCV_2018_paper.pdf
47. Group Normalization, 8월 24, 2025에 액세스, https://arxiv.org/abs/1803.08494
48. arXiv:1803.08494v3 [cs.CV] 11 Jun 2018, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1803.08494
49. [1702.03275] Batch Renormalization: Towards Reducing Minibatch Dependence in Batch-Normalized Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/1702.03275
50. Layer Normalization - Department of Computer Science, University of Toronto, 8월 24, 2025에 액세스, https://www.cs.utoronto.ca/~hinton/absps/LayerNormalization.pdf
51. Different Normalization Layers in Deep Learning | Towards Data Science, 8월 24, 2025에 액세스, https://towardsdatascience.com/different-normalization-layers-in-deep-learning-1a7214ff71d6/
52. Layer Normalization vs. Batch Normalization: What's the Difference? - Coursera, 8월 24, 2025에 액세스, https://www.coursera.org/articles/layer-normalization-vs-batch-normalization
53. Understanding and Improving Layer Normalization - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1911.07013
54. Arbitrary Style Transfer in Real-Time With ... - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content_ICCV_2017/papers/Huang_Arbitrary_Style_Transfer_ICCV_2017_paper.pdf
55. Normalization Methods in Deep Learning - Ahmad Badary, 8월 24, 2025에 액세스, https://ahmedbadary.github.io/work_files/research/dl/concepts/norm_methods
56. Batch-Instance Normalization for Adaptively Style-Invariant Neural Networks - NIPS, 8월 24, 2025에 액세스, https://proceedings.neurips.cc/paper_files/paper/2018/file/018b59ce1fd616d874afad0f44ba338d-Paper.pdf
57. Jarvis73/Group_Normalization: A simple group normalization with batch norm in tensorflow (of cource with moving average). - GitHub, 8월 24, 2025에 액세스, https://github.com/Jarvis73/Group_Normalization
58. [1709.09603] Riemannian approach to batch normalization - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/1709.09603
59. Batch Layer Normalization, A new normalization layer for CNNs and RNN - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2209.08898


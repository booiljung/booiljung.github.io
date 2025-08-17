# 샴 네트워크(Siamese Network)

## 1.  구조적 기초와 학습 원리

딥러닝의 발전은 주로 대규모 레이블 데이터셋을 기반으로 한 분류(classification) 문제에 집중되어 왔습니다. 그러나 현실 세계의 많은 문제들은 클래스당 데이터가 극히 적거나, 학습 시점에는 존재하지 않았던 새로운 클래스를 인식해야 하는 상황에 직면합니다. 이러한 제약을 극복하기 위해 등장한 샴 네트워크(Siamese Network)는 전통적인 분류 패러다임에서 벗어나, '유사성(similarity)' 자체를 학습하는 강력한 접근법을 제시합니다. 이 네트워크는 두 개 이상의 입력을 비교하여 그들 간의 관계를 측정하는 방식으로 작동하며, 이는 원샷 학습(one-shot learning), 얼굴 인식, 서명 위조 판별 등 다양한 분야에서 혁신적인 해결책을 제공했습니다. 본 보고서의 첫 번째 파트에서는 샴 네트워크의 근간을 이루는 구조적 특징과, 이러한 구조가 의미 있는 유사성 척도를 학습할 수 있도록 하는 핵심적인 손실 함수들을 심도 있게 분석합니다.

### 1.1 1: 표준 샴 네트워크 아키텍처: 이중성, 대칭성, 그리고 가중치 공유

샴 네트워크는 분류기가 아닌, 하나의 정교한 거리 측정기(metric learner)를 학습하는 패러다임으로 이해해야 합니다. 그 핵심 목표는 "이 객체는 무엇인가?"를 묻는 대신 "두 객체는 얼마나 유사한가?"라는 질문에 답하는 것입니다.1 이러한 접근 방식의 전환은 훈련 데이터에 존재하지 않는 클래스에 대해서도 유연하게 대응할 수 있는 능력을 부여하며, 이는 샴 네트워크의 가장 중요한 특징 중 하나입니다.

#### 1.1.1 핵심 구조: 쌍둥이 네트워크와 임베딩 공간

샴 네트워크의 구조는 그 이름처럼 '샴쌍둥이'에서 영감을 받았습니다. 이는 두 개 이상의 동일한(identical) 하위 네트워크로 구성되며, 각 하위 네트워크는 독립적인 입력을 처리합니다.3 여기서 '동일하다'는 것은 단순히 네트워크의 계층 구성이 같다는 의미를 넘어, 두 하위 네트워크가 모든 파라미터와 가중치(weights)를 완벽하게 공유한다는 것을 의미합니다.4

네트워크의 작동 과정은 다음과 같은 단계로 이루어집니다:

1. 두 개의 입력 데이터 $x_1$과 $x_2$가 각각의 쌍둥이 인코더(예: CNN)에 투입됩니다.1
2. 가중치를 공유하는 인코더 함수 $f(\cdot)$는 각 입력을 처리하여 고정된 차원의 특징 벡터, 즉 '임베딩(embedding)'으로 변환합니다.12
3. 생성된 두 임베딩 벡터 $f(x_1)$과 $f(x_2)$는 최종 계층에서 유클리드 거리(Euclidean distance)나 L1 노름(L1 norm)과 같은 거리 척도를 통해 비교됩니다.2
4. 이 계산된 거리는 두 입력 간의 학습된 임베딩 공간 내에서의 유사성을 정량적으로 나타내는 점수가 됩니다.

#### 1.1.2 가중치 공유의 원칙과 그 중요성

가중치 공유는 샴 네트워크의 정체성을 규정하는 가장 핵심적인 원리입니다. 만약 두 하위 네트워크가 가중치를 공유하지 않는다면, 각각은 완전히 다른 특징 공간을 학습하게 될 것입니다. 이 경우 두 임베딩 벡터 간의 거리는 무의미한 값이 되며, 이는 마치 서로 다른 언어로 된 단어를 비교하는 것과 같습니다.

가중치 공유는 다음과 같은 필연적인 결과를 낳습니다:

- **대칭성 보장**: 두 동일한 입력이 네트워크에 들어오면 정확히 동일한 임베딩 벡터가 생성됩니다. 이는 학습된 유사성 함수가 대칭적($f(x_1,x_2)=f(x_2,x_1)$)임을 보장하며, 입력 순서에 관계없이 일관된 결과를 도출하는 안정적인 거리 측정의 전제 조건이 됩니다.7
- **일관된 임베딩 공간 학습**: 네트워크는 단 하나의 일관된 매핑 함수를 학습하여, 어떤 입력이든 공유된 임베딩 공간 내의 특정 위치로 사상(mapping)하도록 강제됩니다. 결과적으로, 이 공간 내 두 점 사이의 거리는 의미 있는 유사성 척도가 됩니다.
- **파라미터 효율성**: 두 개의 개별 특징 추출기를 학습하는 대신 단 하나만 학습하므로, 파라미터의 수가 절반으로 줄어듭니다. 이는 모델을 더 효율적으로 만들고, 특히 클래스당 데이터가 적은 상황에서 과적합(overfitting)의 위험을 줄여줍니다.9

#### 1.1.3 이종 데이터 처리를 위한 확장: 의사-샴 네트워크 (Pseudo-Siamese Networks)

표준 샴 네트워크의 가중치 공유 제약은 입력 데이터가 동일한 양식(modality)일 때 효과적입니다. 그러나 이미지와 텍스트, 혹은 하이퍼스펙트럴 영상(HSI)과 라이다(LiDAR) 데이터처럼 구조적으로 다른 이종 데이터를 비교해야 하는 교차 모달(cross-modal) 작업에서는 이러한 제약이 오히려 성능을 저해할 수 있습니다.15

이러한 문제를 해결하기 위해 의사-샴 네트워크(Pseudo-Siamese Network)가 제안되었습니다. 이 구조에서는 가중치 공유 제약을 완화하거나 완전히 제거합니다. 각 데이터 소스에 특화된 별도의 인코더를 사용하여 독립적으로 특징을 추출한 후, 이들 임베딩을 비교하거나 융합하는 방식으로 작동합니다.15 이를 통해 각 데이터의 고유한 특성을 최대한 보존하면서도 두 데이터 간의 의미적 연관성을 학습할 수 있습니다.

### 1.2 2: 유사성의 수학: 거리 학습 손실 함수

샴 네트워크의 핵심은 의미 있는 임베딩 공간을 구축하는 것이며, 이는 손실 함수(loss function)를 통해 이루어집니다. 손실 함수는 임베딩 공간의 기하학적 구조를 조각하는 역할을 하며, 유사한 샘플은 가깝게, 다른 샘플은 멀리 떨어지도록 네트워크를 훈련시킵니다.

#### 1.2.1 대조 손실 (Contrastive Loss): 기초적인 쌍 기반 접근법

대조 손실은 샴 네트워크 학습에 사용되는 가장 기초적인 손실 함수 중 하나입니다. 이는 입력 쌍(pair)을 기반으로 작동하며, 유사한 쌍(positive pair)의 임베딩 간 거리는 최소화하고, 다른 쌍(negative pair)의 임베딩 간 거리는 특정 마진(m) 이상으로 최대화하는 것을 목표로 합니다.2

대조 손실의 수학적 공식은 다음과 같이 표현됩니다 1:
$$
L(Y,D)=(1−Y)⋅\frac{1}{2}D^2+Y⋅\frac{1}{2} \max(0,m−D)^2
$$
여기서 $D$는 두 임베딩 간의 유클리드 거리 $||f(x_1) - f(x_2)||$이며, $Y$는 두 이미지가 유사하면 $1$, 다르면 $0$인 레이블입니다 (일부 구현에서는 반대로 정의하기도 함 7).

- **유사한 쌍 ($Y=1$):** 손실은 $\frac{1}{2}D^2$이 되어, 두 임베딩을 서로 가깝게 끌어당기는 역할을 합니다.
- **다른 쌍 ($Y=0$):** 만약 두 임베딩 간의 거리가 이미 마진 $m$보다 크다면, 손실은 $0$이 되어 가중치 업데이트가 발생하지 않습니다. 그러나 거리가 $m$보다 가깝다면, $\max(0,m−D)^2$ 항이 활성화되어 두 임베딩이 최소 $m$만큼 떨어지도록 밀어냅니다.2

대조 손실은 직관적이지만, 마진 $m$이라는 민감한 하이퍼파라미터를 수동으로 설정해야 하며, 절대적인 거리에만 초점을 맞추기 때문에 상대적 거리 관계를 고려하는 삼중항 손실에 비해 학습이 불안정할 수 있습니다.7

#### 1.2.2 삼중항 손실 (Triplet Loss): 상대적 유사성 학습

삼중항 손실은 FaceNet 논문에서 제안되어 널리 알려졌으며, 한 쌍이 아닌 세 개의 데이터(triplet)를 기반으로 학습합니다. 이 세 데이터는 기준점 역할을 하는 '앵커(Anchor)', 앵커와 같은 클래스인 '포지티브(Positive)', 그리고 앵커와 다른 클래스인 '네거티브(Negative)'로 구성됩니다.3

삼중항 손실의 목표는 앵커와 포지티브 사이의 거리가 앵커와 네거티브 사이의 거리보다 최소한 마진($α$)만큼 더 작아지도록 만드는 것입니다. 이는 절대적인 거리가 아닌 상대적인 거리 관계를 학습하도록 유도합니다.

수학적 공식은 다음과 같습니다 8:
$$
L(A,P,N)=\max(∣∣f(A)−f(P)∣∣^2−∣∣f(A)−f(N)∣∣^2 + α, 0)
$$
여기서 $A,P,N$은 각각 앵커, 포지티브, 네거티브의 임베딩 벡터입니다.

손실은 $ \| f(A)−f(N) \| ^2 > \| f(A)−f(P) \| ^2 + α$ 조건이 만족되면 $0$이 됩니다. 즉, 네거티브 샘플이 포지티브 샘플보다 마진 $α$ 이상으로 충분히 멀리 떨어져 있으면 네트워크는 페널티를 받지 않습니다. 이 상대적 제약 조건은 대조 손실의 절대적 제약보다 더 안정적이고 강력한 임베딩 공간을 형성하는 데 기여합니다.2 기하학적으로 이는 앵커 $A$를 중심으로 하고 반지름이 $\| f(A) − f(P) \| ^2 + α$인 초구(hypersphere) 바깥으로 네거티브 샘플 $N$을 밀어내는 것과 같습니다.

#### 1.2.3 고급 손실 함수: 사중항 손실과 그 이상

손실 함수의 발전은 임베딩 공간에 더욱 정교하고 복잡한 기하학적 구조를 부여하려는 시도의 역사와 같습니다. 단순한 쌍 단위 제약에서 시작하여, 더 높은 차수의 관계를 인코딩하는 방향으로 진화했습니다.

- **사중항 손실 (Quadruplet Loss):** 이 손실 함수는 삼중항에 또 다른 네거티브 샘플($N^2$)을 추가하여 네 개의 데이터(quadruplet)를 사용합니다.17 이는 삼중항 손실의 제약에 더해, 앵커-포지티브 간 거리(클래스 내 분산)가 두 개의 서로 다른 네거티브 샘플 간 거리(클래스 간 분산)보다 작아야 한다는 추가적인 제약을 가합니다. 이를 통해 클래스 내(intra-class) 분산은 최소화하고 클래스 간(inter-class) 분산은 최대화하는, 더욱 구조화된 임베딩 공간을 학습할 수 있습니다.
- **이진 교차 엔트로피 (Binary Cross-Entropy, BCE):** 거리 학습 문제를 이진 분류 문제로 재구성하는 접근법입니다. 두 임베딩 벡터 간의 거리를 계산한 후, 이를 시그모이드(sigmoid) 활성화 함수를 통과시켜 $0$과 $1$ 사이의 '유사 확률' 점수를 출력합니다.1 그리고 이 확률 점수와 실제 레이블(유사함=1, 다름=0) 간의 이진 교차 엔트로피 손실을 최소화하도록 모델을 훈련합니다.1

이러한 손실 함수들의 발전 과정은 중요한 시사점을 던집니다. 대조 손실은 개별 쌍을 독립적으로 고려하기 때문에, 모든 네거티브 쌍이 단순히 마진 $m$ 거리에 위치하도록 학습되어 클래스 간의 상대적인 구조 정보를 잃어버릴 수 있습니다.7 삼중항 손실은 제3의 샘플을 도입하여 상대적 거리를 학습함으로써 이 문제를 해결했지만, 여전히 클래스 내 분산이 클래스 간 분산보다 작다는 것을 명시적으로 강제하지는 않습니다. 사중항 손실은 바로 이 지점을 보완하여, 임베딩 공간에 더욱 바람직한 기하학적 특성을 부여하려는 시도라고 할 수 있습니다.

| 표 1: 핵심 거리 학습 손실 함수 비교 |
| ----------------------------------- |
| **손실 함수**                       |
| 대조 손실 (Contrastive)             |
| 삼중항 손실 (Triplet)               |
| 사중항 손실 (Quadruplet)            |
| 이진 교차 엔트로피 (BCE)            |

## 2.  훈련 및 최적화의 동역학

샴 네트워크의 이론적 구조와 학습 원리를 이해했다면, 다음 단계는 이를 효과적으로 훈련시키는 방법을 탐구하는 것입니다. 이 파트에서는 샴 네트워크 훈련의 실질적인 측면에 초점을 맞춥니다. 특히, 모델의 성능이 훈련 데이터의 질, 즉 어떤 쌍(pair)이나 삼중항(triplet)을 학습에 사용하는지에 결정적으로 의존한다는 점을 중심으로 논의를 전개합니다. 무작위적인 데이터 샘플링의 비효율성을 극복하고, 학습에 가장 유익한 정보를 제공하는 '어려운 예제(hard examples)'를 선별하는 전략은 샴 네트워크 최적화의 핵심 과제입니다.

### 2.1 1: 삼중항 선택의 과제: 하드 네거티브 마이닝

샴 네트워크, 특히 삼중항 손실을 사용하는 모델의 훈련 과정에서 가장 중요한 과제 중 하나는 효과적인 삼중항을 선택하는 것입니다. 무작위로 삼중항을 샘플링하는 방식은 계산적으로 매우 비효율적입니다. 훈련이 진행됨에 따라 네트워크는 대부분의 무작위 삼중항을 쉽게 구별할 수 있게 되며, 이로 인해 손실 값이 0이 되어 더 이상 학습에 기여하는 유용한 경사(gradient)를 제공하지 못하게 됩니다.10 데이터셋의 크기가 $N$일 때 가능한 삼중항의 수는 $O(N^3)$에 달하므로, 모든 조합을 평가하는 것은 현실적으로 불가능합니다.26

#### 2.1.1 삼중항의 난이도 정의

이러한 비효율성을 해결하기 위해 '하드 네거티브 마이닝(Hard Negative Mining)'이라는 전략이 등장했습니다. 이는 학습에 가장 도움이 되는, 즉 모델이 구별하기 어려워하는 '어려운' 삼중항을 의도적으로 선택하는 기법입니다. 삼중항의 난이도는 손실 함수 조건인 $d(a, p) + \text{margin} < d(a, n)$을 기준으로 다음과 같이 세 가지로 분류할 수 있습니다.

- **쉬운 삼중항 (Easy Triplets):** $d(a, p) + \text{margin} < d(a, n)$을 만족하는 경우. 손실이 0이므로 훈련에 아무런 기여를 하지 않습니다.23
- **어려운 삼중항 (Hard Triplets):** $d(a, n) < d(a, p)$을 만족하는 경우. 네거티브 샘플이 포지티브 샘플보다 앵커에 더 가까운, 가장 도전적인 경우입니다. 이는 큰 경사를 제공하지만, 잘못된 레이블이나 이상치(outlier)와 같은 병적인 예제일 수 있습니다. 이러한 예제에만 집중하면 모델이 불안정해지거나 붕괴(collapse)될 수 있습니다.23
- **준-어려운 삼중항 (Semi-Hard Triplets):** $d(a, p) < d(a, n) < d(a, p) + \text{margin}$을 만족하는 경우. 네거티브가 포지티브보다는 멀지만, 여전히 마진 조건을 위반하여 양수의 손실 값을 생성합니다. 이는 과도하게 어렵지 않으면서도 유용한 경사를 제공하여, 안정적이면서 효과적인 훈련을 위한 '최적점(sweet spot)'으로 간주됩니다.23 FaceNet 논문은 이 전략의 효과를 입증하며 널리 대중화시켰습니다.25

#### 2.1.2 마이닝 전략: 오프라인 vs. 온라인

삼중항을 선택하는 시점에 따라 마이닝 전략은 오프라인과 온라인으로 나뉩니다.

- **오프라인 마이닝 (Offline Mining):** 각 에포크(epoch)가 시작되기 전에 전체 훈련 데이터셋을 대상으로 임베딩을 계산하고, 이를 기반으로 어려운 삼중항을 미리 생성합니다. 이는 전역적으로 가장 어려운 삼중항을 찾을 수 있다는 장점이 있지만, 매 에포크마다 전체 데이터에 대한 순전파(forward pass)가 필요하여 계산 비용이 매우 높습니다.28
- **온라인 마이닝 (Online Mining):** 훈련 중 각 미니배치(mini-batch) 내에서 '실시간으로(on the fly)' 삼중항을 생성합니다. 이는 훨씬 효율적이며 현대 딥러닝 프레임워크에서 표준적으로 사용되는 방식입니다.23

#### 2.1.3 온라인 마이닝 기법: Batch-All vs. Batch-Hard

온라인 마이닝은 주로 두 가지 전략으로 구현됩니다.

- **Batch-All:** 미니배치 내에서 가능한 모든 유효한 삼중항을 고려합니다. 이 중에서 손실이 0이 아닌 모든 삼중항(즉, hard 및 semi-hard)에 대해 손실을 계산하고 평균을 냅니다. 이는 배치 내의 정보를 최대한 활용하지만, 배치 크기가 클 경우 계산량이 급격히 증가할 수 있습니다.25
- **Batch-Hard:** 배치 내의 각 앵커에 대해 가장 어려운 포지티브(앵커로부터 가장 먼 포지티브)와 가장 어려운 네거티브(앵커에 가장 가까운 네거티브)를 하나씩 선택하여 삼중항을 구성합니다. 이는 소수의 매우 유익한 삼중항을 생성하여 훈련을 집중시키므로 효율적입니다. 다만, 효과를 보기 위해서는 미니배치가 충분히 크고 다양한 클래스를 포함해야 합니다.25

하드 네거티브 마이닝 전략의 발전은 거리 학습의 근본적인 딜레마, 즉 '강력한 학습 신호'와 '훈련 안정성' 사이의 균형을 어떻게 맞출 것인가에 대한 고민을 반영합니다. 가장 어려운 예제는 가장 큰 경사를 제공하지만, 동시에 모델을 이상치에 과적합시켜 전체적인 임베딩 구조를 망가뜨릴 위험이 있습니다. '준-어려운' 샘플링의 등장은 병적인 예제를 피해가면서도 도전적인 예제를 통해 안정적으로 결정 경계를 개선해 나가는 것이 더 효과적이라는 중요한 통찰을 제공했습니다. 이는 단순히 효율성의 문제를 넘어, 견고한 최적화를 위한 핵심 원리입니다.

### 2.2 2: 비교 분석: 샴 네트워크 대 전통적 분류기

샴 네트워크의 가치를 명확히 이해하기 위해서는 전통적인 CNN 기반 분류기와의 비교가 필수적입니다. 두 아키텍처는 근본적으로 다른 문제 해결 방식을 가지며, 이는 데이터 요구사항, 확장성, 그리고 적합한 적용 분야에서 뚜렷한 차이를 보입니다.

#### 2.2.1 핵심 구조 및 목표의 차이

- **CNN 분류기:** 단일 입력을 받아 사전 정의된 $N$개의 클래스 중 하나로 분류하는 것을 목표로 합니다. 마지막 계층은 일반적으로 소프트맥스(softmax) 함수를 사용하여 각 클래스에 대한 확률 분포를 출력합니다.1 학습 목표는 분류 오류(예: 교차 엔트로피 손실)를 최소화하는 것입니다.
- **샴 네트워크:** 클래스 확률이 아닌 유사도 점수를 출력하는 거리 공간을 학습합니다.1 학습 목표는 삼중항 손실 등을 통해 임베딩 공간을 의미론적 유사성에 따라 구조화하는 것입니다.

#### 2.2.2 데이터 요구사항 및 확장성

| 특징                   | CNN 분류기                                            | 샴 네트워크                                                  |
| ---------------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| **데이터 요구사항**    | 각 클래스별로 많은 수의 훈련 데이터가 필요함 6        | 클래스당 소수의 샘플만으로도 학습 가능 (원샷/퓨샷 학습) 4    |
| **클래스 불균형**      | 클래스 불균형에 민감하며, 성능 저하의 원인이 됨       | 클래스 불균형에 강건함 4                                     |
| **새로운 클래스 추가** | 전체 모델, 특히 최종 분류 계층의 재훈련이 필수적임 36 | 재훈련 없이 새로운 클래스를 시스템에 등록 가능 (참조 임베딩 추가) 2 |

#### 2.2.3 성능 및 적용 분야

- **샴 네트워크가 뛰어난 분야:**
  - **개방형 집합(Open-set) 인식 문제:** 얼굴 인증, 서명 위조 판별과 같이 훈련 시점에 모든 클래스를 알 수 없거나 클래스 집합이 계속 변하는 문제에 이상적입니다.1 데이터가 부족한 환경에서 강력한 성능을 발휘합니다.6
- **CNN 분류기가 뛰어난 분야:**
  - **폐쇄형 집합(Closed-set) 분류 문제:** ImageNet 분류와 같이 클래스의 수가 고정되어 있고, 각 클래스에 대한 데이터가 풍부한 전통적인 분류 문제에서는 직접적인 분류기가 거리 학습 접근법보다 높은 정확도를 달성하는 경우가 많습니다.6

두 접근법은 상호 배타적이지 않습니다. 잘 훈련된 샴 네트워크는 강력한 특징 추출기로 활용될 수 있습니다. 샴 네트워크를 통해 얻은 의미론적으로 풍부한 임베딩을 SVM이나 간단한 MLP와 같은 별도의 분류기에 입력으로 사용하여, 거리 학습의 장점과 분류기의 직접성을 결합할 수도 있습니다.14

결론적으로, CNN 분류기와 샴 네트워크 사이의 선택은 근본적으로 **'고정된 결정 경계(rigid decision boundaries)를 학습할 것인가'** 아니면 **'유연한 거리 공간(flexible metric space)을 학습할 것인가'**의 문제입니다. CNN 분류기는 특징 공간을 N개의 명확한 영역으로 분할하며, 이는 새로운 클래스가 등장할 때마다 재분할(재훈련)을 요구합니다. 반면 샴 네트워크는 공간을 분할하는 대신, 공간 내에서 '거리' 또는 '유사성'을 정의하는 연속적인 함수를 학습합니다. 이러한 유연성은 샴 네트워크가 개방형 집합 문제에서 확장 가능하고 강력한 솔루션이 되는 핵심적인 이유입니다. 이는 "고양이가 무엇인지"를 배우는 것이 아니라, "두 이미지가 같은 동물의 이미지인지 구별하는 방법"을 배우는 패러다임의 전환을 의미합니다.

## 3.  주요 응용 분야와 영역별 진화

이론적 개념과 훈련 방법론을 바탕으로, 이 파트에서는 샴 네트워크가 실제 세계의 도전적인 문제들을 어떻게 해결해왔는지 구체적인 사례를 통해 탐구합니다. 컴퓨터 비전과 자연어 처리의 여러 하위 분야에서 샴 네트워크는 단순히 적용되는 것을 넘어, 각 분야의 특성에 맞게 진화하고 발전해왔습니다. 원샷 학습의 가능성을 처음으로 입증한 사례부터, 실시간 객체 추적 기술의 발전을 이끈 혁신적인 아키텍처에 이르기까지, 샴 네트워크의 실용적 가치와 영향력을 조명합니다.

### 3.1 1: 원샷 학습의 개척: 옴니글롯 챌린지

원샷 학습(One-shot Learning)은 단 하나의 예시만으로 새로운 클래스를 인식하도록 학습하는 매우 도전적인 과제입니다.12 이는 일반화에 필요한 데이터가 절대적으로 부족하기 때문에 전통적인 분류기로는 해결이 거의 불가능한 문제입니다.8

샴 네트워크는 이 문제에 대한 최초의 성공적인 딥러닝 기반 해결책을 제시했습니다. 2015년에 발표된 기념비적인 논문 "Siamese Neural Networks for One-shot Image Recognition"은 샴 네트워크를 이용해 이 문제를 해결하는 방법을 명확히 보여주었습니다.7 이 연구의 핵심 아이디어는 네트워크가 특정 문자(예: 'A', 'B', 'C')를 직접 분류하도록 훈련하는 것이 아니라, 두 개의 문자 이미지가 주어졌을 때 '같은 문자 클래스에 속하는지'를 판별하는 **검증(verification) 과제**를 학습하도록 하는 것입니다.12

이러한 접근법은 옴니글롯(Omniglot) 데이터셋을 사용하여 검증되었습니다. 옴니글롯은 50개의 다른 언어(알파벳)에서 가져온 1623개의 문자 클래스로 구성된 데이터셋입니다. 훈련은 배경(background) 알파벳 집합에 속한 문자 쌍들을 사용하여 이루어집니다. 즉, 네트워크는 이전에 본 적 없는 알파벳을 인식하는 것이 아니라, '같은 문자'와 '다른 문자'를 구별하는 일반적인 능력을 학습합니다.

평가 단계(원샷 태스크)에서는 다음과 같은 과정이 진행됩니다:

1. 하나의 테스트 이미지와 함께, 이전에 본 적 없는 N개의 새로운 알파벳 클래스 각각에 대한 단 하나의 예시 이미지를 담은 '서포트 세트(support set)'가 주어집니다.
2. 네트워크는 테스트 이미지를 서포트 세트에 있는 N개의 이미지 각각과 비교합니다.
3. 가장 높은 유사도 점수를 보인 서포트 세트 이미지의 클래스로 테스트 이미지를 분류합니다.37

이 방식을 통해, 20개의 새로운 클래스 중 하나를 맞추는 20-way 원샷 태스크에서 인간과 근접한 수준의 정확도를 달성하며 원샷 학습의 실질적인 가능성을 입증했습니다.37

### 3.2 2: 신원 확인: 얼굴 인식 및 서명 위조 판별

신원 확인은 샴 네트워크의 가장 대표적이고 성공적인 응용 분야입니다. 이는 본질적으로 개방형 집합(open-set) 문제로, 시스템에 새로운 사용자가 계속해서 추가될 수 있기 때문입니다.

#### 3.2.1 얼굴 인식 (Face Recognition)

얼굴 인식 시스템의 목표는 입력된 얼굴을 사전에 정의된 N명의 인물 중 하나로 분류하는 것이 아니라, 두 얼굴 이미지가 동일 인물인지를 **검증(verify)**하는 것입니다.14 샴 네트워크는 이러한 검증 과제에 완벽하게 부합합니다. 새로운 사용자를 시스템에 추가할 때, 모델 전체를 재훈련할 필요 없이 해당 사용자의 참조(reference) 얼굴 이미지에 대한 임베딩 벡터만 데이터베이스에 등록하면 됩니다.14

검증 과정은 다음과 같습니다:

1. **등록(Enrollment):** 사용자의 참조 이미지 임베딩을 계산하여 데이터베이스에 저장합니다.
2. **검증(Verification):** 새로운 입력(probe) 이미지의 임베딩을 계산합니다.
3. **비교(Comparison):** 입력 이미지의 임베딩과 데이터베이스에 저장된 참조 임베딩 간의 거리를 계산합니다. 이 거리가 사전에 설정된 임계값(threshold)보다 낮으면 동일 인물로 판정합니다.2

특히 FaceNet 7이 제안한 삼중항 손실은 얼굴 인식 분야에서 매우 식별력 높은 임베딩을 학습하는 데 큰 발전을 가져왔으며, 대규모 얼굴 인식 시스템의 기반 기술이 되었습니다.

#### 3.2.2 오프라인 서명 위조 판별 (Offline Signature Verification)

서명 위조 판별은 1990년대 초 Bromley와 LeCun에 의해 제안된, 샴 네트워크의 가장 초기 응용 분야 중 하나입니다.44 오프라인 서명은 스캔된 이미지 형태로 주어지며, 동적 정보(필기 속도, 압력 등)가 없어 위조 판별이 더 어렵습니다.

샴 네트워크는 이 문제를 해결하기 위해 쌍 기반으로 훈련됩니다. 훈련 데이터는 '진본 쌍(genuine pair)'과 '위조 쌍(forgery pair)'으로 구성됩니다.

- **진본 쌍:** 동일인의 서명 두 개로 구성되며, 네트워크는 이 쌍에 대해 낮은 거리를 출력하도록 학습됩니다.
- **위조 쌍:** 동일인의 진본 서명 하나와 다른 사람이 모방한 위조 서명 하나로 구성되며, 네트워크는 높은 거리를 출력하도록 학습됩니다.14

최근 연구들은 특정 필기자에 종속되지 않는 '필기자 독립적(writer-independent)' 모델에 초점을 맞추고 있습니다. 이는 샴 네트워크의 원샷 학습 능력을 활용하여, 훈련 데이터에 포함되지 않은 새로운 사람의 서명에 대해서도 진위 여부를 판별할 수 있게 합니다.47

### 3.3 3: 동적 장면 이해: 샴 네트워크 객체 추적기의 진화

실시간 객체 추적(object tracking)은 샴 네트워크 아키텍처가 가장 큰 영향력을 발휘한 분야 중 하나입니다. 샴 네트워크 기반 추적기들은 단순한 템플릿 매칭에서 시작하여, 딥러닝의 표현력을 최대한 활용하는 정교한 프레임워크로 빠르게 진화했습니다.

#### 3.3.1 SiamFC (Fully-Convolutional Siamese Network)

SiamFC는 샴 네트워크를 객체 추적에 본격적으로 도입한 선구적인 모델입니다. 추적 문제를 '유사도 학습' 문제로 재정의했습니다.3

- **핵심 아이디어:** 첫 프레임에서 주어진 대상의 템플릿(template 또는 exemplar) 이미지와 후속 프레임의 더 넓은 검색 영역(search region) 이미지를 비교합니다.
- **메커니즘:** 완전 컨볼루션 네트워크(FCN)를 사용하여 템플릿 특징을 검색 영역 특징 맵 전체에 대해 컨볼루션(정확히는 상호 상관, cross-correlation) 연산을 수행합니다. 이 결과로 생성된 2차원 응답 맵(response map)에서 가장 높은 값을 갖는 위치가 객체의 새로운 위치로 예측됩니다.50 SiamFC는 구조가 간단하고 매우 빠른 속도를 자랑했지만, 객체의 크기나 종횡비 변화에 효과적으로 대응하지 못하는 한계가 있었습니다.

#### 3.3.2 SiamRPN (Siamese Region Proposal Network)

SiamRPN은 객체 탐지(object detection) 분야에서 사용되던 '영역 제안 네트워크(Region Proposal Network, RPN)'를 샴 네트워크 프레임워크에 통합하는 혁신을 이루었습니다.49

- **핵심 혁신:** 이 통합으로 인해 추적 문제는 단순한 유사도 맵 예측에서, 객체의 위치를 찾는 동시에 정확한 경계 상자(bounding box)를 추정하는 **분류(classification)와 회귀(regression)의 결합된 과제**로 발전했습니다.
- **메커니즘:** RPN은 미리 정의된 다양한 크기와 종횡비의 앵커 박스(anchor box)를 기반으로, 각 위치가 전경(객체)인지 배경인지를 분류하고, 동시에 앵커 박스로부터 실제 객체의 경계 상자까지의 오프셋을 회귀 예측합니다.51 이로써 SiamFC의 한계였던 크기 및 종횡비 변화에 강건하게 대응할 수 있게 되어 정확도가 크게 향상되었습니다.

#### 3.3.3 SiamRPN++와 심층 신경망의 도입

초기 샴 네트워크 추적기들은 AlexNet과 같은 비교적 얕은 네트워크를 백본(backbone)으로 사용했습니다. ResNet-50과 같은 매우 깊은 네트워크를 사용하려는 시도는 번번이 실패했는데, 이는 깊은 네트워크의 패딩(padding) 연산이 상호 상관 연산에 필수적인 **이동 불변성(translation invariance)**을 깨뜨리기 때문이었습니다.53

SiamRPN++는 이 문제를 해결하며 샴 네트워크 추적 기술을 한 단계 도약시켰습니다.

- **문제 해결:** 공간적 편향을 완화하는 샘플링 전략을 도입하고, 확장된 컨볼루션(dilated convolution) 등을 사용하여 유효 스트라이드(effective stride)를 줄이는 방식으로 깊은 네트워크 구조를 수정하여 이동 불변성을 유지했습니다.51
- **추가적인 개선:**
  - **계층별 특징 집계(Layer-wise Aggregation):** 깊은 네트워크의 여러 계층에서 추출된 특징(얕은 층의 위치 정보, 깊은 층의 의미 정보)을 결합하여 정확성과 강건성을 모두 높였습니다.
  - **깊이별 상호 상관(Depthwise Cross-correlation):** 기존의 무거운 상호 상관 연산을 파라미터가 훨씬 적은 깊이별 상호 상관으로 대체하여, 훈련을 안정화하고 효율성을 높였습니다.53

| 표 2: 샴 네트워크 객체 추적기의 진화 |
| ------------------------------------ |
| **모델**                             |
| **SiamFC**                           |
| **SiamRPN**                          |
| **SiamRPN++**                        |

### 3.4 4: 비전을 넘어: NLP에서의 의미론적 텍스트 유사도

샴 네트워크의 원리는 이미지뿐만 아니라 다른 데이터 양식에도 성공적으로 적용되었습니다. 특히 자연어 처리(NLP) 분야에서는 두 텍스트 간의 의미론적 유사도(Semantic Textual Similarity, STS)를 측정하는 데 널리 사용됩니다.13

이 경우, 쌍둥이 하위 네트워크는 CNN 대신 순환 신경망(RNN) 계열인 LSTM이나, 최근에는 트랜스포머(Transformer) 기반의 BERT와 같은 언어 모델로 구성됩니다.56 각 문장은 동일한 인코더를 통과하여 고정된 크기의 문장 임베딩 벡터로 변환됩니다. 두 문장의 의미론적 유사도는 이들 임베딩 벡터 간의 코사인 유사도(cosine similarity)나 유클리드 거리를 통해 계산됩니다.5 모델은 '유사한 문장 쌍'과 '다른 문장 쌍'으로 구성된 데이터셋을 사용하여 대조 손실이나 삼중항 손실로 훈련됩니다. 이를 통해 중복 질문 탐지, 표절 검사, 정보 검색 등 다양한 NLP 응용 프로그램의 핵심 기술로 자리 잡았습니다.

## 4.  샴 네트워크 아키텍처의 현대적 최전선

샴 네트워크의 기본 원리는 지난 수십 년간 일관되게 유지되어 왔지만, 그 구현 방식과 적용 범위는 딥러닝 기술의 발전에 따라 끊임없이 진화해왔습니다. 특히 최근 몇 년간, 샴 아키텍처는 자기 지도 학습(Self-Supervised Learning, SSL)과 트랜스포머(Transformer)라는 두 가지 거대한 흐름의 중심에 서게 되었습니다. 이 파트에서는 샴 네트워크가 어떻게 현대 표상 학습의 가장 진보된 연구 분야를 이끌고 있는지, 그리고 전통적인 거리 기반 학습을 넘어선 새로운 패러다임과 어떻게 융합되고 있는지를 탐구합니다.

### 4.1 1: 자기 지도 표상 학습: 비-대조적 학습의 혁명

자기 지도 학습(SSL)은 레이블이 없는 방대한 데이터로부터 유용한 표현(representation)을 학습하는 것을 목표로 합니다. 이 분야에서 샴 네트워크는 핵심적인 아키텍처로 자리 잡았으며, 특히 '표상 붕괴(representational collapse)'라는 고유한 문제를 해결하는 과정에서 다양한 변형을 낳았습니다.

#### 4.1.1 대조적 학습 vs. 비-대조적 학습

- **대조적 학습 (Contrastive Learning, 예: SimCLR):** 동일 이미지에 다른 증강(augmentation)을 적용한 '포지티브 쌍'의 임베딩은 서로 가깝게 만들고, 다른 이미지에서 온 '네거티브 쌍'의 임베딩과는 명시적으로 멀어지도록 학습합니다.59 이 방식은 효과적이지만, 성능을 위해 수많은 네거티브 샘플이 필요하며, 이로 인해 대규모 배치 크기나 메모리 뱅크와 같은 추가적인 장치가 요구됩니다.
- **비-대조적 학습 (Non-Contrastive Learning, 예: BYOL, SimSiam):** 네거티브 쌍 없이 오직 포지티브 쌍만을 사용하여 학습합니다. 이는 개념적으로 더 간단하고 효율적이지만, 네트워크가 모든 입력에 대해 동일한 상수 벡터를 출력하는 자명한 해(trivial solution), 즉 '표상 붕괴'에 빠질 위험이 있습니다.59

#### 4.1.2 SimSiam: 가장 단순한 샴 SSL 모델

SimSiam은 비-대조적 학습의 핵심 원리를 가장 간결하게 구현한 모델입니다.

- **아키텍처:** 두 개의 증강된 이미지 뷰(x1,x2)가 가중치를 공유하는 인코더 f를 통과합니다. 한쪽 브랜치에는 추가적으로 예측 MLP 헤드 h가 적용됩니다. 손실은 한쪽 브랜치의 출력 $p_1 = h(f(x_1))$과 다른 쪽 브랜치의 출력 z2=f(x2) 간의 코사인 유사도를 최대화(음의 코사인 유사도를 최소화)하는 방식으로 계산됩니다.15
- **정지-경사 (Stop-Gradient) 연산:** SimSiam의 핵심 요소는 z2에서 인코더 f로 경사가 역전파되는 것을 막는 '정지-경사' 연산입니다. 이로 인해 역전파 과정에서 아키텍처가 비대칭적으로 작동하게 되며, 이는 표상 붕괴를 방지하는 결정적인 역할을 합니다.59

#### 4.1.3 표상 붕괴 방지 메커니즘 비교

주요 SSL 모델들은 표상 붕괴를 각기 다른 방식으로 해결합니다.

- **SimCLR:** 방대한 수의 네거티브 쌍을 대조 손실 함수에 명시적으로 사용하여, 포지티브 쌍이 한 점으로 붕괴되는 것을 막습니다.65
- **BYOL:** '모멘텀 인코더(momentum encoder)'를 사용합니다. 타겟 네트워크(target network)의 가중치를 온라인 네트워크(online network) 가중치의 지수 이동 평균(EMA)으로 천천히 업데이트합니다. 이로 인해 타겟이 더 안정적으로 유지되어, 온라인 네트워크가 자명한 해로 수렴하는 것을 방지합니다.59
- **SimSiam:** 오직 정지-경사 연산만으로 붕괴를 방지합니다. 이에 대한 이론적 가설은 이 최적화 과정이 기댓값 최대화(Expectation-Maximization, EM) 알고리즘과 유사하게 작동한다는 것입니다. 즉, 네트워크 가중치 최적화와 타겟 표현 추정이라는 두 개의 하위 문제를 암묵적으로 번갈아 푸는 것과 같다는 설명입니다.59

#### 4.1.4 부분적 차원 붕괴 (Partial Dimensional Collapse)

완전한 표상 붕괴 외에도, 임베딩 벡터가 사용 가능한 전체 차원 공간을 활용하지 못하고 더 낮은 차원의 부분 공간(subspace)에 머무는 '부분적 차원 붕괴'라는 더 미묘한 실패 모드가 존재합니다.61 연구에 따르면, SimSiam은 모델의 용량(capacity)이 데이터의 복잡도에 비해 너무 작을 때 이러한 현상이 발생하기 쉽다는 것이 밝혀졌습니다. 이는 정지-경사 연산만으로는 붕괴를 완벽하게 막을 수 없음을 시사합니다.63

| 표 3: 주요 자기 지도 학습 아키텍처 분석 |
| --------------------------------------- |
| **모델**                                |
| **SimCLR**                              |
| **BYOL**                                |
| **SimSiam**                             |

이 표는 주요 SSL 방법들의 미묘한 차이를 명확히 보여줍니다. 특히 SimSiam이 "BYOL에서 모멘텀 인코더를 제거한 것" 또는 "SimCLR에서 네거티브 쌍을 제거한 것"으로 이해될 수 있다는 점은, 샴 아키텍처가 SSL의 공통된 문제인 표상 붕괴를 해결하는 다양한 전략의 중심에 있음을 보여줍니다.59

### 4.2 2: 트랜스포머 혁명: 샴 네트워크와 어텐션의 융합

CNN이 지역적 특징(local feature)을 포착하는 데 탁월한 반면, 트랜스포머는 자기 주의(self-attention) 메커니즘을 통해 이미지 전체의 장거리 의존성 및 전역적 맥락(global context)을 모델링하는 데 뛰어난 성능을 보입니다.66 샴 아키텍처와 비전 트랜스포머(Vision Transformer, ViT)의 결합은 두 아키텍처의 장점을 모두 활용하는 강력한 모델을 탄생시켰습니다.

#### 4.2.1 샴 비전 트랜스포머 (Siam-ViT)

이 구조에서는 기존의 CNN 기반 쌍둥이 인코더를 ViT 백본으로 대체합니다.69 이를 통해 지역적 디테일과 전역적 구조를 모두 포착하는 풍부한 임베딩을 학습할 수 있습니다.

- **변화 탐지 (Change Detection):** 위성 이미지 분석에서 서로 다른 시점에 촬영된 두 이미지를 비교하는 데 샴 트랜스포머가 사용됩니다. 트랜스포머의 전역적 맥락 이해 능력은 건물과 같은 큰 구조물의 변화를 정확하게 탐지하는 데 도움을 줍니다.69
- **객체 추적 (Object Tracking):** 템플릿과 검색 영역 임베딩 간의 교차 주의(cross-attention) 메커니즘을 도입하여, 복잡한 배경 속에서도 대상 객체와의 연관성을 더 강건하게 학습하고 추적 성능을 향상시킵니다.68
- **교차 모달 학습 (Cross-Modal Learning):** 이미지-텍스트 매칭과 같은 작업에서 의사-샴 네트워크 형태의 이중 인코더(dual-encoder) 구조가 활발히 사용됩니다. 이미지용 ViT와 텍스트용 트랜스포머가 각각의 데이터를 공유 임베딩 공간으로 매핑하여, 텍스트 기반 이미지 검색과 같은 작업을 수행합니다.74

### 4.3 3: 진보된 거리 학습: 유클리드 거리를 넘어서

대조 손실과 삼중항 손실은 유클리드 거리 공간에서 작동합니다. 이는 효과적이지만, 클래스 간 분리를 최대화하거나 클래스 내 응집도를 최대화하는 것을 명시적으로 강제하지는 않습니다.

#### 4.3.1 각도 마진 손실 (Angular Margin Losses: ArcFace, CosFace, SphereFace)

이 방법론들은 거리 학습 문제를 초구(hypersphere) 상에서 재정의합니다.

- **핵심 아이디어:** 특징 임베딩과 최종 분류 계층의 가중치를 모두 정규화하여, 모든 임베딩이 단위 초구 표면에 위치하도록 강제합니다.78
- **메커니즘:** 손실은 임베딩 벡터와 해당 클래스의 프로토타입 벡터 사이의 **각도(angle)**를 기반으로 계산됩니다. 여기에 덧셈(ArcFace) 또는 곱셈(SphereFace) 형태의 각도 마진을 추가하여 분류 과제를 더 어렵게 만듭니다. 이 과정은 네트워크가 더 식별력 높은 임베딩을 학습하도록 강제하여, 클래스 내 샘플들을 더 촘촘하게 모으고 클래스 간 간격은 더 넓게 벌립니다.78
- **샴 네트워크 손실과의 차이점:** 이 방법들은 개별 샘플 쌍이나 삼중항을 비교하는 대신, 각 샘플을 학습된 클래스 프로토타입과 비교합니다. 이는 순수한 거리 학습보다는 수정된 분류 목표에 가깝지만, 매우 구조적이고 식별력 높은 임베딩 공간을 학습한다는 최종 목표는 동일합니다. 특히 복잡한 삼중항 마이닝 과정 없이 클래스 내 응집도와 클래스 간 분리도를 직접적으로 최적화할 수 있다는 큰 장점이 있습니다.78

## 5.  실용적 고려사항과 미래 연구 방향

이 마지막 파트에서는 샴 네트워크의 실제 배포 과정에서 발생하는 현실적인 문제들을 다루고, 앞으로 이 분야가 나아갈 새로운 연구 방향과 미해결 과제들을 조망합니다. 이론적 우수성을 넘어, 샴 네트워크가 실제 응용 환경에서 효율적이고, 공정하며, 강건하게 작동하기 위한 실용적인 고려사항들을 심도 있게 논의합니다.

### 5.1 1: 배포와 효율성: 엣지 디바이스를 위한 최적화

샴 네트워크의 많은 응용 분야(예: 스마트폰의 얼굴 잠금 해제, 실시간 객체 추적)는 계산 능력과 전력이 제한된 엣지 디바이스(edge devices)에서의 배포를 요구합니다.82 따라서 모델의 효율성은 단순한 성능 지표를 넘어 필수적인 요구사항이 됩니다.

- **모델 압축 기술:**
  - **가지치기 (Pruning):** 훈련된 네트워크에서 모델 성능에 거의 영향을 미치지 않는 불필요한 가중치나 연결을 제거하는 기술입니다. 이를 통해 모델의 크기와 계산 비용을 정확도 손실을 최소화하면서 줄일 수 있습니다.85
  - **양자화 (Quantization):** 모델의 가중치와 활성화 값의 수치 정밀도를 낮추는(예: 32비트 부동소수점에서 8비트 정수로) 기술입니다. 이는 메모리 사용량을 크게 줄이고, 호환되는 하드웨어에서 추론 속도를 가속화할 수 있습니다.85
- **경량 아키텍처 설계:** 샴 네트워크의 인코더 백본으로 ResNet 대신 MobileNet과 같은 경량 아키텍처를 사용하는 것은 엣지 디바이스에서 실시간 성능을 확보하기 위한 핵심적인 전략입니다.84

### 5.2 2: 강건성과 공정성: 현실 세계의 도전 과제 해결

실험실 환경을 벗어나 실제 세계에 모델을 배포할 때, 예측하지 못한 다양한 문제에 직면하게 됩니다. 샴 네트워크 역시 공정성, 적대적 공격에 대한 강건성, 그리고 도메인 변화와 같은 문제로부터 자유롭지 않습니다.

- **얼굴 인식에서의 공정성과 편향:** 샴 네트워크 기반의 얼굴 인식 시스템은 인종, 성별, 연령 등 특정 인구 통계 그룹에 따라 성능 차이를 보일 수 있습니다.91 이러한 편향은 주로 훈련 데이터의 불균형에서 비롯됩니다. 연구에 따르면, 다양한 인구 통계 그룹에 걸쳐 균형 잡힌 데이터셋으로 훈련함으로써 이러한 편향을 완화하고 공정성을 개선할 수 있습니다.91
- **적대적 공격에 대한 강건성:** 다른 딥러닝 모델과 마찬가지로 샴 네트워크도 적대적 공격(adversarial attacks)에 취약합니다. 이는 인간이 인지할 수 없는 미세한 노이즈를 입력 이미지에 추가하여 모델이 오분류(예: 위조 서명을 진본으로 인증)하도록 만드는 공격입니다.92 적대적 훈련(adversarial training)이나 노이즈 제거 오토인코더를 전처리 단계로 활용하는 등의 방어 메커니즘을 통해 모델의 강건성을 향상시킬 수 있습니다.92
- **도메인 이동 (Domain Shift):** 모델이 훈련된 환경(소스 도메인)과 다른 특성을 가진 환경(타겟 도메인)에 배포될 때 성능이 저하되는 현상입니다. 이는 서명 위조 판별이나 객체 추적과 같이 다양한 환경에서 작동해야 하는 응용 분야에서 매우 중요한 과제입니다.

### 5.3 3: 종합 분석 및 미래 연구 방향

샴 네트워크는 '분류'가 아닌 '비교'를 통해 학습하는 패러다임의 전환을 이끌며, 데이터가 부족한 환경에서의 학습 문제에 대한 강력한 해법을 제시했습니다. 손실 함수의 발전은 임베딩 공간에 정교한 기하학적 구조를 부여하려는 노력의 역사였으며, 효과적인 데이터 샘플링 전략의 중요성을 일깨웠습니다. 나아가, 샴 아키텍처는 자기 지도 학습과 트랜스포머 모델의 핵심 구성 요소로 성공적으로 편입되며 그 생명력을 이어가고 있습니다.

이러한 성과에도 불구하고, 샴 네트워크와 거리 학습 분야에는 여전히 많은 미해결 과제와 흥미로운 연구 방향이 남아 있습니다.

- **이론적 이해:** 비-대조적 SSL(예: SimSiam)에서 표상 붕괴를 방지하는 정확한 이론적 메커니즘은 아직 완전히 규명되지 않았으며, 활발한 연구가 진행 중인 분야입니다.59
- **차세대 아키텍처:** 트랜스포머를 넘어 상태 공간 모델(State-Space Models)이나 새로운 형태의 컨볼루션 구조와 같은 차세대 아키텍처를 샴 프레임워크에 통합하는 연구가 진행되고 있습니다.98
- **SSL 패러다임의 통합:** 대조적 학습, 비-대조적 학습, 그리고 마스크된 이미지 모델링(MIM)과 같이 샴 구조를 활용하는 다양한 SSL 접근법들을 아우르는 통합적인 이론을 정립하려는 노력이 이루어지고 있습니다.101
- **생성 모델과 거리 학습의 시너지:** 샴 네트워크와 생성 모델 간의 상호작용을 탐구하는 연구가 주목받고 있습니다. 예를 들어, 잘 훈련된 샴 네트워크를 지각 손실(perceptual loss)로 사용하여 생성적 적대 신경망(GAN)이 의미론적으로 더 풍부한 이미지를 생성하도록 유도할 수 있습니다.104
- **데이터 중심 AI (Data-Centric AI):** 미래 연구는 모델 구조 개선뿐만 아니라, 자동화된 데이터 증강, 커리큘럼 학습 전략 등 데이터 중심의 접근법을 통해 샴 네트워크 훈련의 효율성과 강건성을 향상시키는 데 더 많은 초점을 맞출 것으로 예상됩니다.108

샴 네트워크는 지난 수십 년간 딥러닝 분야의 중요한 한 축을 담당해왔으며, 그 근본적인 아이디어는 오늘날 가장 진보된 연구 분야에서도 여전히 강력한 영향력을 발휘하고 있습니다. 앞으로도 샴 네트워크는 새로운 아키텍처와 학습 방법론과 결합하며, 기계가 세상을 이해하고 비교하는 방식을 지속적으로 혁신해 나갈 것입니다.

#### **참고 자료**

1. Understanding Siamese Networks: A Comprehensive Introduction - Analytics Vidhya, accessed July 19, 2025, https://www.analyticsvidhya.com/blog/2023/08/introduction-and-implementation-of-siamese-networks/
2. A Comprehensive Guide to Siamese Neural Networks | by Rinki Nag - Medium, accessed July 19, 2025, https://medium.com/@rinkinag24/a-comprehensive-guide-to-siamese-neural-networks-3358658c0513
3. Siamese neural network - Wikipedia, accessed July 19, 2025, https://en.wikipedia.org/wiki/Siamese_neural_network
4. An Introduction to Siamese Networks | Built In, accessed July 19, 2025, https://builtin.com/machine-learning/siamese-network
5. Siamese Network In NLP Made Simple & How To Tutorial In Python - Spot Intelligence, accessed July 19, 2025, https://spotintelligence.com/2023/01/19/siamese-network-in-nlp/
6. Using a Convolutional Siamese Network for Image-Based Plant ..., accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7148474/
7. [실전문제연구단] Siamese Network, Contrastive Loss, Triplet Loss, accessed July 19, 2025, https://na1-4an.tistory.com/106
8. Siamese Network (샴네트워크) - Philemon 1:21 - 티스토리, accessed July 19, 2025, https://esther-eun27.tistory.com/16
9. How Do Siamese Networks Work in Image Recognition? | Baeldung on Computer Science, accessed July 19, 2025, https://www.baeldung.com/cs/siamese-networks
10. 샴 네트워크(Siamese Network),삼중항 손실 (Triplet loss) - KAU-Deeperent - 티스토리, accessed July 19, 2025, https://kau-deeperent.tistory.com/47
11. 샴 네트워크(Siamese Network)와 삼중항 손실 (Triplet loss) - Aero-Machine Learning, accessed July 19, 2025, [https://metar.tistory.com/entry/%EC%83%B4-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%ACSiamese-Network%EC%99%80-%EC%82%BC%EC%A4%91%ED%95%AD-%EC%86%90%EC%8B%A4-Triplet-loss](https://metar.tistory.com/entry/샴-네트워크Siamese-Network와-삼중항-손실-Triplet-loss)
12. [230602] Siamese Network - velog, accessed July 19, 2025, https://velog.io/@hsbc/230602-Siamese-Network
13. Similarity learning with Siamese Networks, accessed July 19, 2025, https://www.mygreatlearning.com/blog/siamese-networks/
14. Siamese Network – your face is the key - Digital Fingerprints, accessed July 19, 2025, https://fingerprints.digital/en/expert-zone/siamese-network-your-face-is-the-key/
15. A Multi-Scale Pseudo-Siamese Network with an Attention ... - MDPI, accessed July 19, 2025, https://www.mdpi.com/2072-4292/15/5/1283
16. Siamese network doubts : r/deeplearning - Reddit, accessed July 19, 2025, https://www.reddit.com/r/deeplearning/comments/ugu9my/siamese_network_doubts/
17. Optimizing Siamese Convolutional Neural Networks: BCE x Contrastive x Triplet x Quadruplet Losses | by Matheus Ramos Parracho | Medium, accessed July 19, 2025, https://medium.com/@mathparracho/optimizing-siamese-convolutional-neural-networks-bce-x-contrastive-x-triplet-x-quadruplet-losses-317922963ddc
18. Siamese Networks Introduction and Implementation - Towards Data Science, accessed July 19, 2025, https://towardsdatascience.com/siamese-networks-introduction-and-implementation-2140e3443dee/
19. What is meant by siamese network: train one network for each class or one network for all classes (example of training face recognition) - Cross Validated, accessed July 19, 2025, https://stats.stackexchange.com/questions/559404/what-is-meant-by-siamese-network-train-one-network-for-each-class-or-one-networ
20. Quadruplet Loss For Improving the Robustness to Face Morphing Attacks - arXiv, accessed July 19, 2025, https://arxiv.org/html/2402.14665v1
21. Quadruplet Loss For Improving the Robustness to Face ... - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2402.14665
22. adityajn105/Face-Recognition-Siamese-Network: A Face Recognition Siamese Network implemented using Keras. Siamese Network is used for one shot learning which do not require extensive training samples for image recognition. - GitHub, accessed July 19, 2025, https://github.com/adityajn105/Face-Recognition-Siamese-Network
23. How To Train Your Siamese Neural Network | Towards Data Science, accessed July 19, 2025, https://towardsdatascience.com/how-to-train-your-siamese-neural-network-4c6da3259463/
24. CovidExpert: A Triplet Siamese Neural Network framework for the detection of COVID-19, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9837208/
25. Triplet Loss - Deep Learning - FR, accessed July 19, 2025, https://perso.esiee.fr/~chierchg/deep-learning/tutorials/metric/metric-2.html
26. Siamese and triplet networks with online pair/triplet mining in PyTorch - GitHub, accessed July 19, 2025, https://github.com/adambielski/siamese-triplet
27. Smart Mining for Deep Metric Learning - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content_ICCV_2017/papers/Harwood_Smart_Mining_for_ICCV_2017_paper.pdf
28. Triplet Loss and Online Triplet Mining in TensorFlow | Olivier Moindrot blog, accessed July 19, 2025, https://omoindrot.github.io/triplet-loss
29. Triplet Loss and Siamese Neural Networks | by Enosh Shrestha - Medium, accessed July 19, 2025, https://medium.com/@enoshshr/triplet-loss-and-siamese-neural-networks-5d363fdeba9b
30. Hard Negative Mining - Schneppat AI, accessed July 19, 2025, https://schneppat.com/hard-negative-mining.html
31. Siamese Network Training using Artificial Triplets by ... - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2106.07015
32. Hard Triplet Mining - Schneppat AI, accessed July 19, 2025, https://schneppat.com/hard-triplet-mining.html
33. How Does Triplet Loss and Online Triplet Mining Work? - Sanjaya's Blog, accessed July 19, 2025, https://sanjayasubedi.com.np/deeplearning/online-triplet-mining/
34. Triplet Loss: Intro, Implementation, Use Cases - V7 Labs, accessed July 19, 2025, https://www.v7labs.com/blog/triplet-loss
35. Power of Siamese Networks and Triplet Loss: Tackling Unbalanced Datasets - Medium, accessed July 19, 2025, https://medium.com/@mandalsouvik/power-of-siamese-networks-and-triplet-loss-tackling-unbalanced-datasets-ebb2bb6efdb1
36. Face recognition using siamese networks [Tutorial] - Packt, accessed July 19, 2025, https://www.packtpub.com/en-us/learning/how-to-tutorials/face-recognition-using-siamese-networks-tutorial
37. Siamese Neural Networks for One-shot Image Recognition(샴 ..., accessed July 19, 2025, [https://jayhey.github.io/deep%20learning/2018/02/06/saimese_network/](https://jayhey.github.io/deep learning/2018/02/06/saimese_network/)
38. 샴 네트워크를 이용한 문제 이미지 검색기능 만들기 - Medium, accessed July 19, 2025, [https://medium.com/blog-for-everyone/%EC%83%B4-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%AC%B8%EC%A0%9C-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EA%B2%80%EC%83%89%EA%B8%B0%EB%8A%A5-%EB%A7%8C%EB%93%A4%EA%B8%B0-4f694d11d692](https://medium.com/blog-for-everyone/샴-네트워크를-이용한-문제-이미지-검색기능-만들기-4f694d11d692)
39. [논문리뷰] Siamese Neural Networks for One-shot Image Recognition - 까치도리 - 티스토리, accessed July 19, 2025, https://bambini77.tistory.com/47
40. 정재윤 - Siamese neural networks for one-shot image recognition - YouTube, accessed July 19, 2025, https://www.youtube.com/watch?v=1v_knTWJl9s
41. [논문 리뷰] Siamese Neural Networks for One-shot Image Recognition - 매일 한걸음씩, accessed July 19, 2025, https://simonezz.tistory.com/100
42. 샴 신경망 - 위키백과, 우리 모두의 백과사전, accessed July 19, 2025, [https://ko.wikipedia.org/wiki/%EC%83%B4_%EC%8B%A0%EA%B2%BD%EB%A7%9D](https://ko.wikipedia.org/wiki/샴_신경망)
43. fingerprints.digital, accessed July 19, 2025, [https://fingerprints.digital/en/expert-zone/siamese-network-your-face-is-the-key/#:~:text=Siamese%20Network%20instead%20of%20classifying,images%20represent%20the%20same%20person.](https://fingerprints.digital/en/expert-zone/siamese-network-your-face-is-the-key/#:~:text=Siamese Network instead of classifying,images represent the same person.)
44. Handwritten documents author verification based on the Siamese network, accessed July 19, 2025, https://isprs-archives.copernicus.org/articles/XLVIII-2-W5-2024/73/2024/isprs-archives-XLVIII-2-W5-2024-73-2024.pdf
45. Signature Verification using a "Siamese" Time Delay Neural Network, accessed July 19, 2025, https://papers.neurips.cc/paper_files/paper/1993/file/288cc0ff022877bd3df94bc9360b9c5d-Paper.pdf
46. Siamese signature verification with confidence - Kaggle, accessed July 19, 2025, https://www.kaggle.com/code/medali1992/siamese-signature-verification-with-confidence
47. Offline Signature Verification USING Siamese Neural Network - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/390475418_Offline_Signature_Verification_USING_Siamese_Neural_Network
48. (PDF) SIGNATURE VERIFICATION USING SIAMESE NEURAL NETWORK ONE-SHOT LEARNING - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/353743646_SIGNATURE_VERIFICATION_USING_SIAMESE_NEURAL_NETWORK_ONE-SHOT_LEARNING
49. Siamese Network Based Single Object Tracking - Qiang Zhang, accessed July 19, 2025, https://zhangtemplar.github.io/siam-track/
50. Single Object Tracking using the Siamese Family of Trackers - Part 1 ..., accessed July 19, 2025, https://medium.com/alegion/single-object-tracking-using-the-siamese-family-of-trackers-part-1-siamfc-43e6aabfe55a
51. SiamRPN++: Evolution of Siamese Visual Tracking With Very Deep Networks | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/338506597_SiamRPN_Evolution_of_Siamese_Visual_Tracking_With_Very_Deep_Networks
52. Real-time object tracking in the wild with Siamese network, accessed July 19, 2025, https://cs.nju.edu.cn/rinc/publish/download/2023/2023_4.pdf
53. SiamRPN++: Evolution of Siamese Visual ... - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content_CVPR_2019/papers/Li_SiamRPN_Evolution_of_Siamese_Visual_Tracking_With_Very_Deep_Networks_CVPR_2019_paper.pdf
54. Comprehensive STS using Siamese Sentence-BERT - Kaggle, accessed July 19, 2025, https://www.kaggle.com/code/akashmathur2212/comprehensive-sts-using-siamese-sentence-bert
55. Semantic Textual Similarity with Siamese Neural Networks - ACL Anthology, accessed July 19, 2025, https://aclanthology.org/R19-1116/
56. shahrukhx01/siamese-nn-semantic-text-similarity: A repository containing comprehensive Neural Networks based PyTorch implementations for the semantic text similarity task, including architectures such as - GitHub, accessed July 19, 2025, https://github.com/shahrukhx01/siamese-nn-semantic-text-similarity
57. Learning Text Similarity with Siamese Recurrent ... - ACL Anthology, accessed July 19, 2025, https://aclanthology.org/W16-1617.pdf
58. Siamese Networks in NLP - Scaler Topics, accessed July 19, 2025, https://www.scaler.com/topics/nlp/siamese-networks-in-nlp/
59. Exploring Simple Siamese Representation ... - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2021/papers/Chen_Exploring_Simple_Siamese_Representation_Learning_CVPR_2021_paper.pdf
60. arXiv:2011.10566v1 [cs.CV] 20 Nov 2020, accessed July 19, 2025, https://arxiv.org/pdf/2011.10566
61. UNDERSTANDING DIMENSIONAL COLLAPSE IN CON- TRASTIVE SELF-SUPERVISED LEARNING - OpenReview, accessed July 19, 2025, https://openreview.net/pdf?id=YevsQ05DEN7
62. SimSiam: Streamlining SSL with Stop-Gradient Mechanism, accessed July 19, 2025, https://learnopencv.com/simsiam/
63. Understanding Collapse in Non-Contrastive Siamese ..., accessed July 19, 2025, https://www.cs.cmu.edu/~dpathak/papers/eccv22.pdf
64. [Representation Learning] SimSiam: BYOL without the momentum encoder - Youngdo Lee, accessed July 19, 2025, https://leeyngdo.github.io/blog/representation-learning/2023-01-18-simple-siamese-representation-learning/
65. Review - SimSiam: Exploring Simple Siamese Representation Learning | by Sik-Ho Tsang, accessed July 19, 2025, https://sh-tsang.medium.com/review-simsiam-exploring-simple-siamese-representation-learning-3c84ceb61702
66. ViTCN: Vision Transformer Contrastive Network For Reasoning - arXiv, accessed July 19, 2025, [https://arxiv.org/pdf/2403.09962?](https://arxiv.org/pdf/2403.09962)
67. arXiv:2403.13677v1 [cs.CV] 20 Mar 2024, accessed July 19, 2025, https://arxiv.org/pdf/2403.13677
68. [2105.03817] TrTr: Visual Tracking with Transformer - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2105.03817
69. Siamese Transformer-Based Building Change Detection in Remote Sensing Images - PMC, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10891731/
70. Siamese Transformer Networks for Few-shot Image Classification - arXiv, accessed July 19, 2025, https://arxiv.org/html/2408.01427v1
71. Transforming Challenges: Siamese-Based Vision Transformers for Robust Occluded Face Recognition | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/383866891_Transforming_Challenges_Siamese-Based_Vision_Transformers_for_Robust_Occluded_Face_Recognition
72. [2201.01293] A Transformer-Based Siamese Network for Change Detection - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2201.01293
73. 3D Siamese Transformer Network for Single Object Tracking on Point Clouds - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2207.11995
74. SCMM: Calibrating Cross-modal Fusion for Text-Based Person Search - arXiv, accessed July 19, 2025, https://arxiv.org/html/2304.02278v6
75. Novel cross-dimensional coarse-fine-grained complementary network for image-text matching, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11888920/
76. arXiv:2401.10011v1 [cs.CV] 18 Jan 2024, accessed July 19, 2025, https://arxiv.org/pdf/2401.10011
77. Retrieval-Enhanced Contrastive Vision-Text Models - arXiv, accessed July 19, 2025, https://arxiv.org/html/2306.07196v2
78. Face Recognition with ArcFace Machine Learning Model ..., accessed July 19, 2025, https://learnopencv.com/face-recognition-with-arcface/
79. Metric Classification Network in Actual Face Recognition Scene - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/1910.11563
80. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - arXiv, accessed July 19, 2025, http://arxiv.org/pdf/1801.07698
81. [D] Understanding arcface, sphereface, and their differences. : r/MachineLearning - Reddit, accessed July 19, 2025, https://www.reddit.com/r/MachineLearning/comments/jicbn2/d_understanding_arcface_sphereface_and_their/
82. Optimizing lightweight neural networks for efficient mobile edge computing - PMC, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12218929/
83. Neural Architecture Optimization for Edge Devices - IBISC, accessed July 19, 2025, https://www.ibisc.univ-evry.fr/~htabia/ResearchProjects/NAO_Edge_Devices.pdf
84. Two-stream Beats One-stream: Asymmetric Siamese Network for Efficient Visual Tracking, accessed July 19, 2025, https://arxiv.org/html/2503.00516v1
85. Pruning and Quantization in Keras - Kaggle, accessed July 19, 2025, https://www.kaggle.com/code/sumn2u/pruning-and-quantization-in-keras
86. Deep Learning Model Optimization Methods - Neptune.ai, accessed July 19, 2025, https://neptune.ai/blog/deep-learning-model-optimization-methods
87. A Comprehensive Guide to Neural Network Model Pruning | Datature Blog, accessed July 19, 2025, https://datature.io/blog/a-comprehensive-guide-to-neural-network-model-pruning
88. Diving Into Model Pruning in Deep Learning | pruning – Weights & Biases - Wandb, accessed July 19, 2025, https://wandb.ai/authors/pruning/reports/Diving-Into-Model-Pruning-in-Deep-Learning--VmlldzoxMzcyMDg
89. Quantization, Projection, and Pruning - MATLAB & Simulink - MathWorks, accessed July 19, 2025, https://www.mathworks.com/help/deeplearning/quantization.html
90. Siamese Networks for Cat Re-Identification: Exploring Neural Models for Cat Instance Recognition - arXiv, accessed July 19, 2025, https://arxiv.org/html/2501.02112v1
91. Investigating Fairness in Facial Verification with Siamese Neural Networks - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/386399880_Investigating_Fairness_in_Facial_Verification_with_Siamese_Neural_Networks
92. (PDF) Siamese Denoising Autoencoders for Enhancing Adversarial Robustness in Medical Image Analysis - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/391824221_Siamese_Denoising_Autoencoders_for_Enhancing_Adversarial_Robustness_in_Medical_Image_Analysis
93. Evaluating the Robustness of LiDAR Point Cloud Tracking Against Adversarial Attack - arXiv, accessed July 19, 2025, https://arxiv.org/html/2410.20893v2
94. Robustness of Sparsely Distributed Representations to Adversarial Attacks in Deep Neural Networks - PMC, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10297406/
95. Sim-CLIP: Unsupervised Siamese Adversarial Fine-Tuning for Robust and Semantically-Rich Vision-Language Models - arXiv, accessed July 19, 2025, https://arxiv.org/html/2407.14971v2
96. Dual Perspectives on Non-Contrastive Self-Supervised Learning - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/393333101_Dual_Perspectives_on_Non-Contrastive_Self-Supervised_Learning
97. NeurIPS Poster Reverse Engineering Self-Supervised Learning, accessed July 19, 2025, https://neurips.cc/virtual/2023/poster/71837
98. An Efficient deep learning model to Predict Stock Price Movement Based on Limit Order Book - arXiv, accessed July 19, 2025, http://arxiv.org/pdf/2505.22678
99. SRSNetwork: Siamese Reconstruction-Segmentation Networks based on Dynamic-Parameter Convolution - arXiv, accessed July 19, 2025, https://arxiv.org/html/2312.01741v1
100. arXiv:2112.13983v1 [cs.CV] 28 Dec 2021 - Fengxiang He, accessed July 19, 2025, https://fengxianghe.github.io/paper/lan2022siamese.pdf
101. Siamese Image Modeling for Self-Supervised Vision Representation Learning - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2023/papers/Tao_Siamese_Image_Modeling_for_Self-Supervised_Vision_Representation_Learning_CVPR_2023_paper.pdf
102. 5th Workshop on Self-Supervised Learning: Theory and Practice - NeurIPS 2025, accessed July 19, 2025, https://neurips.cc/virtual/2024/workshop/84703
103. Unifying Self-Supervised Clustering and Energy-Based Models, accessed July 19, 2025, https://openreview.net/pdf/b1ca55183dde34e58bd6f554b3f8bf3988916827.pdf
104. (PDF) Enhancing Image Retrieval Performance With Generative Models in Siamese Networks - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/389199238_Enhancing_Image_Retrieval_Performance_With_Generative_Models_in_Siamese_Networks
105. Stacked Siamese Generative Adversarial Nets: A Novel Way to Enlarge Image Dataset - MDPI, accessed July 19, 2025, https://www.mdpi.com/2079-9292/12/3/654
106. Employing texture loss to denoise OCT images using generative adversarial networks, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11019688/
107. Siamese Content Loss Networks for Highly Imbalanced Medical ..., accessed July 19, 2025, https://openreview.net/forum?id=VINrwcDkvA
108. Exploring Non-contrastive Self-supervised Representation Learning for Image-based Profiling - arXiv, accessed July 19, 2025, https://arxiv.org/html/2506.14265v1
109. Evolutionary algorithms meet self-supervised learning: a comprehensive survey - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2504.07213


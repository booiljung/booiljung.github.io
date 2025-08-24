# 퓨샷 러닝(Few-Shot Learning)
[인공지능 훈련 데이터의 클래스 불균형 문제](./index.md)




퓨샷 러닝(Few-Shot Learning, FSL)은 기계 학습 모델이 극소수의 레이블링된 예제(labeled examples)만을 학습하여 정확한 예측을 수행하도록 훈련하는 기계 학습 프레임워크로 정의된다.1 FSL의 핵심 목표는 단순한 암기를 넘어, 제한된 데이터로부터 효과적으로 일반화할 수 있는 모델을 개발하는 것이다.4 이는 대규모 데이터셋에 의존하는 전통적인 인공지능의 주류 경향과는 명백한 대조를 이룬다.1 즉, FSL은 이전에 보지 못한 데이터(unseen data)에 대해서도 적은 수의 훈련 데이터만으로 높은 인식 및 분류 성능을 달성하고자 하는 기술적 지향점을 가진다.1 이 패러다임은 모델이 소수의 예제로부터 추론(inference)할 수 있는 능력을 갖추게 하는 것을 목적으로 하며, 이를 구현하기 위한 주요 방법론으로 메타 학습(Meta-learning)과 전이 학습(Transfer learning)이 제안되었다.1


FSL의 근본적인 영감은 인간의 인지 능력에서 비롯된다. 인간은 평생에 걸쳐 축적된 사전 지식을 활용하여 단 몇 개의 예제만으로 새로운 개념을 학습하고 객체를 인식하는 놀라운 능력을 지니고 있다.4 예를 들어, 어린 아이는 얼룩말 사진을 몇 장만 보고도 이후에 마주치는 모든 얼룩말을 인식할 수 있다.7 이러한 학습 효율성은 인간이 새로운 동사를 배운 후 즉시 그 동사를 다양한 문법적 구조에 맞춰 조합하여 사용하는 능력에서도 나타난다.9 FSL은 바로 이러한 인간의 경이로운 효율성을 기계에서 모방하려는 시도이며, 무차별적인 데이터 수집에 대한 의존도를 줄이고 보다 적응력 높은 인공지능 시스템을 구축하는 것을 목표로 한다.6


FSL의 등장은 데이터 집약적인 딥러닝 모델이 가진 본질적인 한계에 대한 직접적인 대응이다.11 수많은 실제 응용 분야에서 대규모의 레이블링된 데이터셋을 확보하는 것은 비현실적이거나, 비용이 많이 들거나, 심지어 불가능한 경우가 많다.3

- **높은 비용과 전문성:** 데이터에 정확한 주석을 다는 작업, 특히 의료 영상 분석과 같은 전문 분야에서는 해당 분야의 깊이 있는 지식이 필요하며, 이는 막대한 비용을 수반한다.13
- **본질적 희소성:** 희귀 질병, 멸종 위기종, 독특한 필체, 혹은 코로나19와 같은 새로운 질병의 초기 발병 단계와 같은 시나리오에서는 데이터 자체가 본질적으로 희소하다.4 이러한 상황에서는 대규모 데이터셋을 구축하는 것 자체가 원천적으로 불가능하다.
- **비즈니스 민첩성:** 기업 환경에서 AI 모델은 새로운 제품, 고객군, 또는 시장 변화에 신속하게 대응할 수 있어야 한다. 수개월이 소요되는 데이터 수집 및 레이블링 주기는 급변하는 비즈니스 요구를 충족시킬 수 없으며, 따라서 FSL은 기업에 핵심적인 경쟁 우위를 제공한다.3


FSL의 중심 원리는 사전 지식(prior knowledge)을 활용하여 새로운 작업에 대한 가설 공간(hypothesis space)을 제한하고 학습 과정을 안내하는 것이다.12 이 사전 지식은 일반적으로 대규모의 관련 작업이나 데이터에 대한 메타 학습 또는 전이 학습 단계를 통해 습득된다.1 모델은 이 과정을 통해 '학습하는 방법' 자체를 학습하게 되며, 이렇게 축적된 일반화된 지식을 바탕으로 소수의 예제만으로 새로운 작업을 신속하게 해결할 수 있게 된다.

이러한 접근 방식은 AI 연구 패러다임에 있어 근본적인 전환을 의미한다. 기존의 패러다임이 *규모에 따른 성능(performance-at-scale)*에 초점을 맞추었다면, FSL은 *효율성과 적응성(efficiency-and-adaptability)*을 핵심 가치로 삼는다. 학습의 문제를 정적인 데이터셋에 대한 일회성 작업으로 보는 대신, 신속한 적응을 가능하게 하는 일반화 가능한 지식을 지속적으로 습득하는 과정으로 재정의하는 것이다. 초창기 딥러닝의 성공은 ImageNet과 같은 대규모 데이터셋에서 모델과 데이터의 크기에 비례하여 성능이 향상되는 '빅데이터' 패러다임을 확립했다.16 그러나 실제 응용 환경에서는 데이터가 본질적으로 희소한 '롱테일(long-tail)' 문제가 만연함이 드러났다.5 이에 대한 해답으로 FSL은 단순한 기술을 넘어 '학습하는 법을 배우는(learning to learn)' 새로운 철학으로 부상했다.1 이제 목표는 단일 작업의 전문가 모델을 훈련하는 것이 아니라, 새로운 작업에 대해 신속하게 전문가가 될 수 있는 *메타 학습자(meta-learner)*를 만드는 것이다. 이는 '학습'이 일어나는 장소와 방식에 대한 관점의 변화를 시사한다. 전통적인 기계 학습에서 학습은 전적으로 훈련 단계에서 발생하지만, FSL에서는 메타 학습된 사전 지식에 의해 안내되는 추론 시간의 신속한 적응 과정이 학습의 중요한 부분을 차지한다. 이로 인해 FSL 모델은 본질적으로 더 동적이며 실제 운영 환경의 제약 조건에 더 잘 부합하게 된다.



FSL은 제한된 레이블링된 데이터로부터 학습하는 문제를 다루는 더 넓은 개념인 N-샷 러닝(N-shot learning) 패러다임의 하위 집합이다.2 이 패러다임은 일반적으로 

**N-way-K-shot** 분류 프레임워크를 통해 공식화된다.2

- **N (N-way):** 주어진 특정 작업(task)에 포함된 고유한 클래스의 수.
- **K (K-shot):** 각 클래스에 대해 제공되는 레이블링된 예제(또는 "샷")의 수.
- **서포트 세트(Support Set):** N개의 각 클래스에 대해 K개의 레이블링된 예제로 구성된 집합. 모델이 새로운 작업을 학습하거나 적응하는 데 사용된다.13
- **쿼리 세트(Query Set):** 모델이 서포트 세트를 기반으로 분류해야 하는 레이블이 없는 예제 집합.13
- **에피소드 훈련(Episodic Training):** 훈련 과정에서 테스트 시나리오를 모방하는 방식. 전체 훈련 데이터셋에서 반복적으로 N-way-K-shot 형식의 미니배치, 즉 "에피소드(episode)"를 구성하여 모델을 훈련시킨다. 이는 훈련과 테스트 조건을 일치시켜 모델의 일반화 성능을 향상시키는 데 중요한 역할을 한다.22


- **vs. 지도 학습(Supervised Learning):** 전통적인 지도 학습은 클래스당 수백에서 수천 개의 예제를 사용하여 모델을 처음부터 훈련시키며, 대규모의 정적인 데이터셋에 대한 경험적 위험(empirical risk)을 최소화하는 데 중점을 둔다.4 반면, FSL은 데이터 희소성을 기본 전제로 하며, 관련 작업에서 얻은 지식을 활용하여 작은 서포트 세트로부터 일반화하는 능력을 목표로 한다.4
- **vs. 원샷 러닝(One-Shot Learning, OSL):** OSL은 K=1인 경우로, FSL의 특수하고 더 도전적인 형태이다.2 이는 데이터 희소성의 극단적인 사례를 나타내며, 단일 참조 이미지를 기반으로 한 얼굴 인식과 같은 응용 분야에서 흔히 볼 수 있다.21
- **vs. 제로샷 러닝(Zero-Shot Learning, ZSL):** ZSL은 새로운 클래스에 대해 K=0인, 즉 레이블링된 예제가 전혀 없는 별개의 패러다임이다.2 모델은 훈련 중에 목표 클래스의 예제를 전혀 보지 못하며, 예측을 위해 의미론적 속성(semantic attributes)이나 클래스 설명과 같은 보조 정보에 전적으로 의존해야 한다.25 예를 들어, '얼룩말' 이미지를 본 적 없는 모델이 '줄무늬가 있다', '말과 유사하다', '아프리카에 산다'와 같은 속성 정보를 기반으로 얼룩말을 인식하는 것이 ZSL의 예이다.

다음 표는 이러한 학습 패러다임 간의 핵심적인 차이점을 요약하여 보여준다. 이 표는 독자가 FSL이 다른 주요 학습 패러다임과 어떻게 구별되는지 명확하고 직관적으로 이해하도록 돕는다. 데이터 요구사항, 학습 목표, 주요 방법론과 같은 핵심 속성을 나란히 비교함으로써, 보고서의 나머지 부분에 대한 기초 지식을 공고히 한다.

| 특징                    | 제로샷 러닝 (ZSL)                  | 원샷 러닝 (OSL)                       | 퓨샷 러닝 (FSL)                   | 지도 학습 (Supervised Learning)           |
| ----------------------- | ---------------------------------- | ------------------------------------- | --------------------------------- | ----------------------------------------- |
| **새 클래스당 예제 수** | 0                                  | 1                                     | 소수 (예: K=2 ~ 20)               | 다수 (수백 ~ 수백만)                      |
| **학습 목표**           | 보조/의미론적 정보를 이용한 일반화 | 단일 예제로부터의 일반화              | 작은 서포트 세트로부터의 일반화   | 대규모 고정 데이터셋의 경험적 위험 최소화 |
| **주요 방법론**         | 속성 기반 모델, 의미론적 임베딩    | 메트릭 학습 (예: 샴 네트워크)         | 메타 학습, 메트릭 학습, 전이 학습 | 종단간 학습 (예: CNN, 트랜스포머)         |
| **사전 정보 의존도**    | 매우 높음 (설명에 전적으로 의존)   | 높음 (강력한 유사도 함수 학습에 의존) | 높음 (관련 작업 지식에 의존)      | 낮음 (주로 해당 작업 데이터에서 학습)     |




데이터 중심 접근법은 제한된 수의 예제로부터 더 많은 지도 데이터를 생성함으로써 데이터 희소성 문제에 직접적으로 대응하는 방법론이다.12 이는 FSL 문제에 대한 가장 직관적인 해결책으로, 가용 데이터의 양을 늘려 모델이 더 풍부한 정보를 바탕으로 학습할 수 있도록 돕는다.


데이터 증강(Data Augmentation)은 원본 샘플에 다양한 변환을 적용하여 그럴듯한 새로운 예제를 생성하는 기법이다.2 이미지 데이터의 경우, 회전, 반전, 크기 조절, 색상 변형과 같은 간단한 기하학적 또는 광학적 변환이 널리 사용된다. 이 기술은 종종 다른 FSL 방법론과 결합되어 시너지를 창출한다. 예를 들어, 메트릭 학습에서 대조 학습(contrastive learning)을 위한 긍정적 또는 부정적 쌍(positive/negative pairs)을 생성하는 데 데이터 증강이 활용될 수 있다.2


생성 모델(Generative Models)은 데이터 중심 접근법의 더 정교한 형태로, 생성적 적대 신경망(Generative Adversarial Networks, GANs)이나 변분 오토인코더(Variational Autoencoders, VAEs)와 같은 모델을 사용한다.13 이러한 모델들은 소수의 가용 예제 데이터의 잠재적 분포를 학습한 후, 이 분포로부터 새로운 합성 데이터 포인트를 샘플링한다. 이 방법은 희귀 질병 데이터나 멸종 위기종의 이미지처럼 실제 세계에서 예제를 추가로 수집하는 것이 극도로 어려운 경우에 특히 유용하다.21

데이터 중심 접근법은 FSL 문제에 대한 기초적이면서도 중요한 해결책이지만, 종종 그 자체만으로는 충분하지 않다. 이러한 접근법은 주로 데이터의 *양*을 해결하지만, 근본적으로 제한된 정보 공간에서 *어떻게 일반화할 것인가*라는 FSL의 핵심 과제를 직접적으로 다루지는 못한다. 단순히 몇 개의 '샷'을 증강하는 것은 모델이 진정한 클래스의 잠재적 다양체를 학습하기보다는 사용된 특정 변환에 과적합(overfitting)될 위험을 내포한다.20 또한, 극소수의 샘플로 훈련된 생성 모델은 모드 붕괴(mode collapse) 현상을 겪어 다양성이 낮은 샘플만을 생성할 수 있다.28 따라서 데이터 중심 접근법의 진정한 힘은 더 정교한 메타 학습 프레임워크 내에서 보조적인 기술로 사용될 때 발휘된다. 예를 들어, 메타 학습의 맥락에서 데이터 증강이 대조 학습을 위한 쌍을 생성하는 데 사용되는 사례는 2, 이 기술의 역할이 독립적인 해결책에서 보조 도구로 전환되었음을 보여준다. FSL 연구의 발전은 데이터 포인트를 늘리는 것이 도움이 되기는 하지만, 더 중요한 문제는 다른 작업들로부터 강력한 *귀납적 편향(inductive bias)*을 학습하는 것이며, 이는 메타 학습의 영역임을 명확히 했다. 데이터 중심 방법은 이를 지원하지만 대체할 수는 없다.



메트릭 기반 메타 학습(Metric-Based Meta-Learning) 방법들은 입력 데이터를 특정 메트릭 공간(metric space)으로 매핑하는 임베딩 함수(embedding function)를 학습하는 데 중점을 둔다.16 이 공간에서는 같은 클래스에 속한 샘플들은 서로 가깝게, 다른 클래스에 속한 샘플들은 멀리 떨어져 위치하도록 설계된다. 일단 이러한 임베딩 공간이 성공적으로 학습되면, 분류는 쿼리 포인트의 임베딩을 서포트 세트 예제들의 임베딩과 비교하는 간단한 방식으로 수행된다. 이는 K-최근접 이웃(K-Nearest Neighbors) 알고리즘과 유사한 원리다.2


프로토타입 네트워크(Prototypical Networks)는 메트릭 기반 방법론의 대표적인 예시이다.18

- **메커니즘:** 이 네트워크는 각 클래스에 대한 단일 '프로토타입(prototype)' 벡터를 계산한다. 이 프로토타입은 해당 클래스의 서포트 세트 예제들을 임베딩한 후, 그 벡터들의 평균을 취하여 생성된다.20 새로운 쿼리 샘플이 주어지면, 이 샘플을 임베딩 공간으로 매핑한 후, 가장 가까운 클래스 프로토타입을 찾아 해당 클래스로 분류한다. 이때 거리 측정 방식으로는 일반적으로 제곱 유클리드 거리(squared Euclidean distance)가 사용된다.22
- **이론적 기반:** 클래스 평균을 프로토타입으로 사용하는 방식은 클러스터링 및 혼합 밀도 추정(mixture density estimation)과의 이론적 연결에 의해 정당화된다. 특히, 브레그만 발산(Bregman divergence)을 거리 함수로 사용할 경우, 클러스터의 평균이 해당 클러스터에 할당된 점들까지의 거리를 최소화하는 대표점이 된다는 수학적 원리에 기반한다.22


샴 네트워크(Siamese Networks)는 이름에서 알 수 있듯이, 동일한 가중치를 공유하는 두 개의 동일한 신경망 가지(이를 '쌍둥이'라 칭함)를 통해 두 개의 입력을 동시에 처리하는 구조를 가진다.18

- **메커니즘:** 네트워크는 유사한 쌍(같은 클래스의 두 샘플)에 대해서는 낮은 거리를, 비유사한 쌍(다른 클래스의 두 샘플)에 대해서는 높은 거리를 출력하도록 훈련된다. 테스트 시에는 쿼리 이미지를 서포트 세트의 각 이미지와 쌍으로 입력하여 거리를 계산하고, 가장 유사한(거리가 가장 가까운) 서포트 이미지의 클래스로 쿼리를 분류한다.18 이 방법은 특히 표 형식의 데이터(tabular data)에서도 인상적인 결과를 보여주었으며, 소수의 샘플만 있는 시나리오에서 전통적인 기계 학습 모델들을 능가하는 성능을 입증했다.18


- **매칭 네트워크(Matching Networks):** 새로운 작업에 대해 미세 조정(fine-tuning)을 할 필요 없이, 주어진 작은 서포트 세트와 레이블이 없는 예제를 그 예제의 레이블로 직접 매핑하는 방법을 학습한다.18
- **관계 네트워크(Relation Networks):** 유클리드 거리와 같은 고정된 거리 측정법을 사용하는 대신, 쿼리 이미지와 서포트 이미지를 비교하기 위한 심층 거리 메트릭(deep distance metric) 자체를 학습한다. 즉, 두 이미지 간의 '관계 점수'를 예측하는 신경망 모듈을 추가하여 유연성을 높인다.18

메트릭 기반 방법론의 성공은 많은 FSL 작업에서 복잡한 분류 경계면(decision boundary)을 학습하는 것보다 좋은 *표현 공간(representation space)*을 학습하는 것이 더 중요하다는 점을 시사한다. "클래스는 평균 프로토타입 주변에 군집을 이룬다"와 같은 단순한 귀납적 편향은 데이터가 적은 상황에서 약점이 아니라 오히려 강점이 된다. 이는 복잡한 모델이 쉽게 빠질 수 있는 심각한 과적합을 방지하기 때문이다.22 전통적인 분류기는 복잡한 결정 경계를 학습하지만, 소수의 데이터 포인트만으로는 이 경계가 제대로 제약되지 않아 과적합되기 쉽다.22 메트릭 기반 접근법은 이 문제를 재구성한다. 즉, 경계를 학습하는 대신 분류가 사소해지는(가장 가까운 이웃을 찾는 것만으로 충분한) 공간을 학습한다.2 프로토타입 네트워크의 핵심은 잘 학습된 임베딩 공간에서 단순한 평균 벡터가 클래스 개념을 표현하는 놀랍도록 견고하고 효과적인 방법이라는 것이다.22 이는 각 새로운 작업에 대해 파라미터가 많은 복잡한 분류기 헤드를 학습할 필요성을 없애준다. 이는 FSL의 핵심 원리를 드러낸다. 즉, 복잡성은 여러 작업으로부터 학습되는 특징 추출기(임베딩 함수)로 이전하고, 작업별 분류 메커니즘은 가능한 한 단순하게 유지하여 소수의 가용 '샷'에 대한 과적합을 방지하는 것이다.



최적화 기반 메타 학습(Optimization-Based Meta-Learning) 접근법은 단 몇 번의 경사 하강법(gradient descent) 단계만으로 새로운 작업에 신속하게 적응할 수 있는 모델 초기값(initialization)이나 최적화 알고리즘 자체를 학습하는 것을 목표로 한다.16 이 패러다임의 궁극적인 목표는 경사 하강법을 통해 '학습하는 방법' 자체를 학습하는 것이다.1


모델-불문 메타 학습(Model-Agnostic Meta-Learning, MAML)은 이 분야에서 가장 영향력 있는 선구적인 연구이다.20 MAML의 핵심은 특정 모델 구조에 구애받지 않고 경사 하강법으로 훈련될 수 있는 모든 모델에 적용 가능하다는 점이다.

- **이중 최적화(Bi-Level Optimization):** MAML은 중첩된 루프(nested loop) 구조를 통해 작동한다.31
  - **내부 루프 (작업별 적응):** 샘플링된 각 작업에 대해, 모델은 공유된 초기 파라미터(메타 파라미터 $ \theta $)에서 시작한다. 그리고 해당 작업의 서포트 세트를 사용하여 한 번 이상의 경사 하강 단계를 수행하여 작업별로 특화된 파라미터 $ \theta' $를 찾는다.31 이 과정은 새로운 작업에 대한 모델의 빠른 미세 조정을 모방한다.
  - **외부 루프 (메타 최적화):** 내부 루프에서 적응된 파라미터 $ \theta' $의 성능이 해당 작업의 쿼리 세트에서 평가된다. 이 평가에서 계산된 손실(loss)은 초기 메타 파라미터 $ \theta $를 업데이트하는 데 사용된다. 이 메타 업데이트는 내부 루프의 경사 하강 단계를 *통과하여* 미분함으로써 계산되며, 이 과정은 종종 2차 미분(second-order derivatives)을 포함한다.31
- **목표:** MAML의 목표는 어떤 단일 작업에도 최적화되어 있지 않지만, 광범위한 새로운 작업에 대해 매우 민감하게 반응하여 신속하게 적응할 수 있는 파라미터 공간 내의 전략적 지점, 즉 최적의 초기값 $ \theta $를 찾는 것이다.31


강력한 프레임워크임에도 불구하고, 원본 MAML 알고리즘은 몇 가지 중요한 과제를 안고 있으며, 이는 수많은 파생 연구의 등장을 촉발했다.

- **계산 비용 및 안정성:** 메타 업데이트에 포함된 2차 미분은 계산 비용이 매우 높고 메모리 요구량이 커서 훈련 불안정성을 야기할 수 있다.32 이러한 문제를 해결하기 위해 2차 미분을 무시하는 1차 근사 방식인 

  **FOMAML(First-Order MAML)** 37과, 메타 그래디언트를 명시적으로 계산하는 대신 초기 파라미터를 적응된 파라미터 방향으로 점진적으로 이동시키는 

  **Reptile** 37과 같은 변형 모델이 제안되었다.

- **복잡한 손실 지형 및 일반화:** MAML의 이중 최적화 구조는 표준적인 경험적 위험 최소화(ERM)보다 더 많은 안장점(saddle points)과 급격한 지역 최솟값(sharp local minima)을 가진 복잡한 손실 지형(loss landscape)을 만들어내며, 이는 일반화 성능을 저해할 수 있다.35

  **Sharp-MAML**과 같은 변형 모델은 더 평탄한 최솟값(flatter minima)을 찾아 일반화 성능을 향상시키는 것을 목표로 한다.35 또한, 

  **MeTAL**은 다양한 작업에 걸쳐 일반화 성능을 높이기 위해 작업에 적응적인 손실 함수 자체를 학습하는 방식을 제안한다.33

- **내부 루프 최적화기와의 결합:** MAML의 메타 그래디언트 계산은 내부 루프에서 사용되는 최적화기의 경로에 강하게 결합되어 있다. **iMAML**은 암묵적 미분(implicit differentiation)을 사용하여 이 둘을 분리함으로써, 내부 루프에서 더 유연하고 다양한 최적화 전략을 사용할 수 있게 한다.40

- **하이퍼파라미터 민감성:** MAML은 학습률과 같은 하이퍼파라미터에 민감하게 반응한다. **MAML++**는 계층별, 단계별 학습 가능한 학습률, 코사인 어닐링(cosine annealing)과 같은 여러 개선 사항을 도입하여 안정성과 성능을 크게 향상시켰다.42

다음 표는 MAML과 그 주요 변형 모델들의 핵심 아이디어, 최적화 방식, 그리고 장단점을 비교 분석한 것이다. MAML 계열은 FSL 연구에서 가장 영향력 있는 분야 중 하나이지만, 그 변형 모델의 수가 많아 혼란을 줄 수 있다. 이 표는 각 변형 모델이 어떤 특정 문제를 해결하는지, 방법론적 혁신은 무엇인지, 그리고 성능, 계산 비용, 복잡성 측면에서 어떤 트레이드오프가 있는지를 명확히 정리하여 연구자와 실무자에게 실질적인 가이드를 제공한다.

| 변형 모델      | 핵심 아이디어 / 해결 과제                          | 최적화 전략                                                  | 계산 비용 | 주요 장점 / 트레이드오프                                     |
| -------------- | -------------------------------------------------- | ------------------------------------------------------------ | --------- | ------------------------------------------------------------ |
| **MAML**       | 빠른 적응을 위한 민감한 초기값 학습                | 2차 그래디언트를 포함한 이중 최적화                          | 매우 높음 | 모델 불문으로 강력하지만, 계산 비용이 높고 불안정함          |
| **FOMAML**     | MAML의 계산 비용 절감                              | 1차 근사 (2차 그래디언트 무시)                               | 중간      | MAML보다 빠르고 메모리 효율적이지만, 성능이 다소 저하될 수 있음 |
| **Reptile**    | 메타 업데이트 과정의 단순화                        | k번의 내부 스텝 후 초기 가중치를 적응된 가중치 방향으로 이동. 메타 그래디언트 없음 | 낮음      | 매우 단순하고 효율적이지만, MAML보다 조악한 근사 방식임      |
| **iMAML**      | 내부 루프 최적화기 경로로부터 메타 그래디언트 분리 | 암묵적 미분을 사용하여 내부 루프 해법 기반의 메타 그래디언트 계산 | 높음      | 더 유연한 내부 루프 최적화가 가능하지만, 여전히 계산 요구량이 많음 |
| **MAML++**     | MAML의 안정성 및 성능 향상                         | 다수의 개선 사항 도입 (학습 가능한 학습률, 다단계 손실 등)   | 매우 높음 | 원본 MAML보다 견고하고 성능이 높지만, 복잡성이 증가함        |
| **Sharp-MAML** | 급격한 최솟값을 피해 일반화 성능 향상              | Sharpness-aware 최적화; 손실 지형에서 평탄한 최솟값을 탐색   | 높음      | 일반화 성능이 우수하지만, 평탄한 영역을 찾기 위한 추가 최적화 단계가 필요함 |
| **MeTAL**      | 다양한 작업에 대한 일반화 성능 향상                | 내부 루프를 위한 작업 적응적 손실 함수를 메타 학습           | 매우 높음 | 여러 도메인에 걸쳐 뛰어난 일반화 성능을 보이지만, 또 다른 메타 학습기를 추가함 |



모델 기반 메타 학습(Model-Based Meta-Learning) 접근법은 빠른 학습을 위해 특별히 설계된 모델 아키텍처를 제안한다. 이러한 모델들은 종종 새로운 정보를 신속하게 저장하고 검색할 수 있는 명시적인 메모리 메커니즘을 통합하는 특징을 가진다.16


메모리 증강 신경망(Memory-Augmented Neural Networks, MANN)은 이 범주에 속하는 대표적인 모델이다.

- **아키텍처:** MANN은 뉴럴 튜링 머신(Neural Turing Machines, NTMs)에서 영감을 받아 설계되었으며, LSTM과 같은 컨트롤러(controller)와 외부 메모리 행렬(external memory matrix)로 구성된다.30
- **메커니즘:** 컨트롤러는 읽기 및 쓰기 헤드(read and write heads)를 통해 메모리와 상호작용하는 방법을 학습한다. 새로운 예제와 그 레이블이 제시되면, 컨트롤러는 이 정보의 표현을 메모리 슬롯에 쓰는 방법을 학습한다. 나중에 유사한 예제가 나타나면, 컨트롤러는 관련 메모리 슬롯에서 정보를 읽어와 예측을 수행한다.30
- **이중 학습 프로세스:** MANN은 두 가지 학습 프로세스를 결합한다. 하나는 여러 작업에 걸쳐 컨트롤러를 훈련시키는 느린 경사 하강법 기반 학습이고, 다른 하나는 특정 작업을 위해 새로운 정보를 메모리에 쓰는 빠른 메모리 기반 학습이다.43 이 구조 덕분에 모델은 자신의 핵심 가중치를 크게 변경하지 않고도 새로운 정보를 신속하게 동화시킬 수 있다.


이러한 접근법은 특히 순차적 작업 학습(sequential task learning)에 매우 적합하다. 모델이 이전에 학습한 작업을 치명적으로 잊지 않으면서(catastrophic forgetting) 새로운 작업을 학습해야 하는 시나리오에서, 메모리는 작업별 지식을 저장하는 효과적인 메커니즘을 제공한다.45

다음 표는 파트 II에서 논의된 데이터 중심, 메트릭 기반, 최적화 기반, 모델 기반 접근법을 요약하여 FSL 방법론의 전체적인 그림을 제공한다. 이 표는 각 패러다임의 근본적인 철학, 강점, 약점을 종합적으로 정리하여 독자가 FSL 기술의 지형을 통합적으로 이해하는 데 도움을 준다.

| 범주            | 핵심 원리 / 철학                                    | 대표 방법론                                     | 강점                                                       | 약점                                                         |
| --------------- | --------------------------------------------------- | ----------------------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| **데이터 중심** | "데이터가 부족하면 더 만들어낸다."                  | 데이터 증강, GANs, VAEs                         | 구현이 간단하며 다른 방법과 결합 가능                      | 과적합을 유발할 수 있으며, 생성된 데이터의 다양성이 부족할 수 있음 |
| **메트릭 기반** | "좋은 임베딩 공간을 학습하면 분류는 쉬워진다."      | 프로토타입 네트워크, 샴 네트워크, 매칭 네트워크 | 단순한 귀납적 편향, 테스트 시 계산 효율성, 강력한 성능     | 성능이 학습된 메트릭의 품질에 크게 의존함                    |
| **최적화 기반** | "빠른 적응을 위한 좋은 모델 초기값을 학습한다."     | MAML 및 그 변형 모델 (Reptile 등)               | 매우 유연함 (모델 불문), 강력한 일반화 잠재력              | 계산 비용이 높고, 훈련이 불안정할 수 있으며, 복잡함          |
| **모델 기반**   | "빠른 학습을 위해 설계된 모델 아키텍처를 구축한다." | 메모리 증강 신경망 (MANN)                       | 신속한 지식 동화에 명시적으로 설계됨; 순차적 작업에 유리함 | 아키텍처가 복잡하고 다른 방법보다 범용성이 떨어질 수 있음    |




FSL은 데이터 수집이 어려운 컴퓨터 비전 작업에 광범위하게 적용되고 있다.2

- **이미지 분류:** 새로운 FSL 방법을 벤치마킹하는 데 자주 사용되는 표준적인 FSL 작업이다.2
- **객체 탐지 및 의미론적 분할:** 소수의 예제만으로 새로운 클래스의 객체를 탐지하고 분할하는 더 복잡한 작업이다. 이러한 작업들은 종종 퓨샷 분류 기술을 기반으로 확장된다.2


퓨샷 이미지 생성(Few-Shot Image Generation, FSIG)은 단 몇 개의 예제(예: 10개)만으로 새로운 도메인에 대한 다양하고 고품질의 이미지를 생성하는 것을 목표로 하는 도전적인 연구 분야이다.46

- **부적합한 지식 이전의 문제:** 이 분야의 핵심적인 문제 중 하나는 '부적합한 지식 이전(incompatible knowledge transfer)'이다. 예를 들어, 인간의 얼굴 이미지로 사전 훈련된 모델을 고양이 얼굴 이미지 생성에 적응시킬 때, 소스 도메인(인간 얼굴)의 관련 없는 특징(예: 안경)이 타겟 도메인(고양이 얼굴)으로 이전되어 비현실적인 결과물(예: 안경 쓴 고양이)을 생성하는 현상이다.47
- **해결책:** 최근 연구들은 이러한 부적합한 지식의 이전을 막기 위해 소스 생성기에서 중요도가 낮은 필터를 식별하고 제거(pruning)하는 방법을 제안했다.47 또한, 확산 모델(diffusion models)을 활용하여, 도메인 분류기의 가이드를 통해 사전 훈련된 모델을 목표 도메인에 적응시키는 연구도 활발히 진행되고 있다.48

컴퓨터 비전에서 FSL이 분류에서 생성으로 발전하는 과정은 지식 이전에 대한 이해가 깊어지고 있음을 보여준다. 초기 연구는 유용한 지식을 *보존*하는 데 중점을 두었지만, 최신 연구는 해롭거나 부적합한 지식을 *제거*하는 것의 중요성을 동등하게 강조하고 있다. 프로토타입 네트워크와 같은 초기 분류 방법은 전이 가능한 특징 공간을 학습하는 데 초점을 맞췄다.22 목표는 '좋은' 특징을 보존하는 것이었다. 그러나 이러한 아이디어가 생성 모델(FSIG)로 확장되자 새로운 문제가 발생했다. 모델이 목표 특징을 생성하지 못하는 것을 넘어, 잘못된 맥락에서 

*소스* 특징을 적극적으로 생성하기 시작한 것이다.47 이것이 바로 '부적합한 지식 이전'이다. 이는 패러다임의 전환을 강요했다. 문제는 사전 훈련된 모델에서 무엇을 

*유지*할 것인가에 대한 것뿐만 아니라, 무엇을 *잊을* 것인가에 대한 문제이기도 하다. 이는 RICK 47과 같은 네트워크 가지치기 기반의 해결책으로 이어졌다. 이는 효과적인 전이 학습이 보존과 망각 사이의 섬세한 균형이라는 것을 보여주며, 이 개념은 이제 고급 FSL 연구의 중심에 있다.



FSL은 대규모의 레이블링된 말뭉치(corpus)를 사용할 수 없는 저자원 언어나 특정 전문 분야의 NLP 작업에 있어 매우 중요하다.6


- **텍스트 분류 및 감성 분석:** 소수의 예제만으로 새로운 카테고리나 도메인에 대한 텍스트를 분류하도록 모델을 적응시킨다.6
- **관계 추출:** 지식 베이스 구축의 핵심 작업으로, 텍스트 내 엔티티 간의 새로운 유형의 관계를 식별하는 데 FSL이 사용된다.52


거대 언어 모델(Large Language Models, LLMs)의 등장은 NLP 분야의 FSL을 완전히 바꾸어 놓았다. 방대한 양의 레이블이 없는 텍스트로 사전 훈련된 LLM은 인-컨텍스트 학습(in-context learning)의 한 형태인 퓨샷 프롬프팅(few-shot prompting)을 통해 특정 작업에 신속하게 적응할 수 있다.13 이 주제는 파트 V에서 더 자세히 다룰 것이다.



구조화되지 않고 동적인 환경에서 작동하는 로봇은 새로운 물체를 인식하고 상호작용하며, 새로운 작업에 신속하게 적응할 수 있어야 한다.6 FSL은 이러한 필수적인 적응력을 제공한다.2


- **객체 인식 및 조작:** 소수의 시각적 예제로부터 새로운 객체를 인식하고, 파지(grasping)하며, 조작하는 법을 학습한다.27
- **작업 지향적 파지:** FSL은 로봇이 단순히 안정적인 파지를 학습하는 것을 넘어, 후속 작업을 수행하는 데 적합한 파지를 학습하도록 돕는다(예: 물을 따르기 위해 컵의 손잡이를 잡는 것).61 이는 종종 소수의 인간 시연(human demonstration)으로부터 학습함으로써 달성된다.61
- **환경 적응:** 강화 학습과 결합된 FSL은 로봇이 새로운 환경과 작업에 빠르게 적응할 수 있도록 한다.2

로보틱스 분야에서 FSL은 단순히 학문적인 문제를 넘어, 실제 세계에서의 자율성을 가능하게 하는 핵심 기술이다. 이는 시뮬레이션된 훈련 환경과 예측 불가능한 물리적 세계 사이의 간극을 메워주며, 제한된 상호작용으로부터 신속하고 즉각적인 학습을 가능하게 한다. 전통적인 로보틱스는 가능한 모든 객체와 시나리오에 대해 철저한 프로그래밍이나 시뮬레이션 훈련을 요구했지만, 이는 취약하고 확장성이 떨어진다.61 FSL은 '프로그래밍하지 말고 가르쳐라'는 새로운 패러다임을 제시한다. 인간이 몇 번 작업을 시연하면 로봇이 이를 일반화하는 것이다.61 이는 특히 파지 작업에서 강력하다.59 가능한 파지의 공간은 거의 무한대에 가깝지만, 인간의 시연에 의해 안내되는 FSL은 이 탐색 공간을 작업과 관련된 파지로 제한하여 문제를 다루기 쉽게 만든다. 이는 미래의 로보틱스가 FSL의 데이터 효율성에 힘입어 인간과 로봇이 지속적인 루프 안에서 서로에게서 배우는 상호작용적이고 협력적인 형태가 될 것임을 시사한다.



의료 분야는 높은 데이터 수집 비용, 환자 개인정보 보호 문제, 특정 질병의 희소성 등으로 인해 FSL의 가치가 극대화되는 영역이다.2


- **의료 영상 분석:** 대규모 데이터셋 수집이 불가능한 MRI와 같은 의료 영상에서 희귀 질환을 분류한다.2
- **신약 개발:** 화합물의 생리 활성 데이터를 얻는 것이 비싸고 시간이 많이 걸리기 때문에, 이 분야는 본질적으로 데이터가 부족한(low-data) 환경이며 FSL의 핵심 적용 분야 중 하나이다.19
  - **사례 연구: 약물 반응 예측 (TCRP):** TCRP(Translation of Cellular Response Prediction) 모델은 FSL(특히 MAML)을 사용하여 대규모 세포주 스크리닝 데이터로 모델을 훈련시킨 후, 단 몇 개의 새로운 샘플만으로 다른 조직 유형이나 환자 유래 종양과 같은 새로운 맥락에서 약물 반응을 예측하도록 적응시킨다. 이 접근법은 조직 유형 간 이전 시 단 5개의 추가 샘플만으로 평균 **829%의 성능 향상**을 보였다.66
  - **사례 연구: 화합물 활성 예측 (FS-CAP):** FS-CAP(Few-Shot Compound Activity Prediction) 모델은 소수의 알려진 '히트(hit)' 화합물을 기반으로 새로운 화합물의 연속적인 활성 값을 예측하기 위해 새로운 신경망 아키텍처를 사용한다. 이 모델은 전통적인 화학적 유사성 기반 방법 및 다른 FSL 기반 모델들을 능가하며, 특히 초기 히트 화합물과 구조적으로 상이하면서도 강력한 활성을 보이는 새로운 화합물을 식별하는 데 효과적이다.69
- **시스템 신경약리학:** FSL은 뇌 활동 매핑과 결합하여 제한된 데이터셋으로부터 잠재적인 중추신경계(CNS) 약물 후보를 식별하고 발견 과정을 가속화하는 데 기여한다.72

다음 표는 파트 III에서 논의된 다양한 도메인에 걸친 FSL의 실제적인 영향력을 종합적으로 보여준다. 해결된 특정 문제, 사용된 FSL 방법, 그리고 주요 결과를 강조함으로써 이 기술의 실질적인 가치를 입증한다.

| 도메인          | 응용 / 사례 연구          | 해결된 핵심 과제                            | 사용된 FSL 방법                                      | 보고된 영향 / 결과                                           |
| --------------- | ------------------------- | ------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| **컴퓨터 비전** | 퓨샷 이미지 생성 (RICK)   | 소스 모델로부터의 부적합한 지식 이전        | 네트워크 가지치기 (지식 절단)                        | 소스 도메인의 인공물(예: '안경 쓴 고양이')을 제거하여 더 현실적인 이미지 생성 47 |
| **로보틱스**    | 작업 지향적 파지          | 소수의 시연으로부터 기능적 파지 학습        | 파지 영역 분할 데이터셋에 대한 메타 학습             | 비전문가에 의한 로봇의 새로운 파지 전략 원격 교육 가능 62    |
| **헬스케어**    | 약물 반응 예측 (TCRP)     | 세포주에서 환자로 모델 이전                 | 최적화 기반 (MAML)                                   | 조직 유형 간 이전 시 5개의 새 샘플로 평균 829% 성능 향상 66  |
| **신약 개발**   | 화합물 활성 예측 (FS-CAP) | 소수의 히트로부터 새롭고 강력한 화합물 식별 | 맞춤형 신경망 아키텍처 (분리된 쿼리/컨텍스트 인코더) | 유사성 검색을 능가; 더 큰 구조적 다양성을 가진 강력한 화합물 발견 69 |
| **오디오 처리** | 퓨샷 화자 식별            | 훈련/테스트 불일치로 인한 과적합            | 채널 어텐션을 사용한 깊이별 분리 CNN                 | 경량의 파라미터 효율적인 아키텍처를 사용하여 과적합 완화 73  |




메타-과적합(Meta-Overfitting)은 메타 학습에서 발생하는 주요 과제 중 하나로, 모델이 메타-훈련(meta-training) 단계에서 본 작업들의 분포에 과적합되는 현상을 의미한다. 이로 인해, 전반적으로 동일한 도메인에서 추출되었더라도 훈련 시 본 작업들과 성격이 크게 다른 새로운 메타-테스트(meta-test) 작업에 대해서는 성능이 저하된다.33 즉, 모델이 훈련 작업을 너무 잘 학습한 나머지 새로운 작업에 적응하고 일반화하는 능력을 상실하게 되는 것이다.33


도메인 이동(Domain Shift)은 메타-과적합보다 더 심각한 문제로, 메타-훈련 작업과 메타-테스트 작업이 서로 다른 데이터 분포에서 추출되는 경우를 말한다 (예: 일반적인 자연 이미지(ImageNet)로 훈련하고 의료 영상으로 테스트하는 경우).23 이러한 교차-도메인 FSL(Cross-Domain FSL, CDFSL) 시나리오에서 표준적인 FSL 방법들은 종종 치명적인 성능 저하를 겪는다.23


도메인 이동 문제는 두 가지 상충하는 목표를 만들어낸다. 한편으로 도메인 적응(Domain Adaptation, DA)은 소스 도메인과 타겟 도메인의 분포를 정렬하여 도메인 격차를 줄이는 것을 목표로 한다. 다른 한편으로 FSL은 타겟 클래스가 소스 클래스와 구별되어야 한다고 요구한다. 분포를 지나치게 공격적으로 정렬하면 클래스 표현이 섞여 FSL 성능을 해칠 수 있다.75


- **적대적 도메인 적응:** FSL 성능과 도메인 불변성(domain invariance)을 동시에 달성하도록 모델을 훈련시킨다. 이는 생성기(generator)를 사용하여 훈련 작업을 테스트 도메인의 스타일과 유사하게 변환함으로써 특징 추출기가 도메인 격차에 강건해지도록 만드는 방식으로 이루어질 수 있다.23
- **분리된 표현 학습:** DA를 위해서는 도메인에 혼동을 주고(domain-confused), 클래스 분리를 위해서는 도메인에 특화된(domain-specific) 특징 표현을 학습하여 상충하는 목표를 해결한다.75
- **작업 수준 자기-지도 학습:** 작업의 여러 뷰(view)를 증강하여 도메인에 독립적인 표현을 학습한다.78

메타-과적합과 도메인 이동이라는 과제는 '학습하는 법을 배우는 것'만으로는 충분하지 않다는 것을 보여준다. 모델은 무엇이 전이 가능한 지식이고 무엇이 작업별 또는 도메인별 노이즈인지를 구별하는 법을 배워야 한다. 이는 FSL 연구 분야가 이러한 요인들을 분리할 수 있는 더 정교한 표현 학습 기술로 나아가도록 이끌었다. 초기 FSL 연구는 훈련과 테스트 작업이 동일한 분포에서 나온다고 가정했지만 23, 이는 실제 환경에서는 실패했다. 첫 번째 실패 모드는 모델이 훈련 세트의 

*작업 유형*을 암기하는 메타-과적합이다.50 두 번째, 더 심각한 실패는 데이터의 본질 자체가 변하는 도메인 이동이다.75 이로 인해 단일한, 단일체적인 특징 추출기는 불충분하다는 인식이 생겨났다. 모델은 전이를 위한 일반적인 특징과 도메인에 특화된 특징을 모두 학습해야 한다. 이것이 바로 적대적 방법 23과 분리된 아키텍처 75의 동기이며, 이들은 명시적으로 네트워크가 이러한 분리를 학습하도록 강제한다. 연구의 최전선은 더 이상 빠른 적응에만 머무르지 않고, *견고하고 전이 가능한* 표현 학습으로 이동하고 있다.



연속 퓨샷 러닝(Continual Few-Shot Learning)은 연속 학습(Continual Learning, CL)과 FSL의 어려움을 결합한 패러다임이다. 모델은 이전에 학습한 작업을 치명적으로 잊지 않으면서 순차적으로 주어지는 퓨샷 작업들로부터 학습해야 한다.28 이는 안정성(잊지 않는 것)과 가소성(새로운 것을 배우는 것) 사이의 고전적인 딜레마를 구현한다.


이 패러다임의 두 가지 주요 장애물은 **치명적 망각(catastrophic forgetting)**(새로운 작업을 학습할 때 이전 작업에 대한 지식이 갑자기 손실되는 현상)과 **과적합**(각 작업이 퓨샷의 성격을 띠기 때문에 발생)이다.28


- **메모리 기반 / 리플레이 방법:** 과거 작업의 예제 소수를 메모리 버퍼에 저장하고, 새로운 작업을 훈련할 때 이를 다시 재생(replay)하는 방식이다. 효과적이지만 확장성 문제가 있다.52
- **정규화 기반 방법:** 이전 작업에 중요하다고 판단되는 가중치의 큰 변화에 페널티를 주는 정규화 항을 손실 함수에 추가하는 방식이다.28
- **Continual-MAML:** 작업이 순차적으로 도착하는 시나리오를 위해 설계된 MAML의 온라인 확장 버전이다. 학습된 초기값을 새로운 작업에 적응시키고, 분포 변화가 감지되면 새로운 지식을 통합한다.82
- **프롬프트 기반 학습:** LLM과 함께 프롬프트를 사용하여 특정 관계 범주를 잊어버릴 가능성이 적은 일반적인 작업 지식을 학습한다.53

연속 퓨샷 러닝은 인간의 평생 학습과 가장 유사하며, 가장 현실적이고 도전적인 학습 시나리오라 할 수 있다. 이를 해결하기 위해서는 정적인 에피소드 훈련을 넘어, 시간에 따라 지식을 동적으로 관리할 수 있는 알고리즘을 개발해야 한다. FSL은 일반적으로 정적인 환경에서 연구되지만, 연속 학습은 작업이 순차적으로 도착하는 것을 다루되 보통 작업당 데이터가 풍부하다고 가정한다. 이 둘의 교차점인 연속 FSL은 에이전트가 새로운 작업의 스트림을 마주하고, 각 새로운 작업에 대해 단 몇 개의 예제만으로 학습해야 하는 실제 시나리오를 반영한다.52 이는 두 가지 상반된 목표의 조화를 요구한다. MAML과 같은 방법은 최대의 가소성(빠른 적응)을 위해 설계된 반면, CL 방법은 최대의 안정성(망각 방지)을 위해 설계되었다. Continual-MAML 82이나 지식 증류를 사용하는 방법 54과 같은 해결책들은 원칙적인 균형을 찾으려는 시도를 나타내며, 미래의 지능형 시스템이 지식 습득과 통합을 위한 명시적인 메커니즘을 모두 필요로 할 것임을 시사한다.



연합 퓨샷 러닝(Federated Few-Shot Learning, FFSL)은 FSL을 연합 학습(Federated Learning, FL)과 결합한 설정이다. FL 환경에서는 데이터가 여러 클라이언트(예: 모바일 장치)에 분산되어 있으며, 개인정보 보호 문제로 인해 중앙 서버로 데이터를 집중시킬 수 없다.85


- **전역 데이터 분산 (데이터 이질성):** 클라이언트 간의 데이터 분포는 비-IID(non-independent and identically distributed) 특성을 보인다. 즉, 각 클라이언트의 데이터 통계적 특성이 다르다. 이러한 이질적인 데이터로 훈련된 로컬 모델들을 단순히 집계(aggregation)하면 학습 과정을 방해하고 전역 모델의 성능을 저하시킬 수 있다.85
- **지역 데이터 불충분성:** 각 클라이언트는 클래스당 예제가 적을 뿐만 아니라(FSL 문제), 전체 클래스 중 일부만을 보유하고 있을 수 있다. 이로 인해 의미 있는 로컬 모델을 학습하는 것이 매우 어렵다.88


- **분리된 메타 학습 (F²L):** 이 프레임워크는 두 개의 모델을 사용하는 것을 제안한다. 하나는 각 클라이언트가 로컬에서 작업별 메타 지식을 학습하고 절대 공유하지 않는 비공개 `클라이언트-모델`이고, 다른 하나는 클라이언트 불변 지식을 학습하고 서버에서 집계되는 공유 `서버-모델`이다. 이 설계는 두 학습 목표를 분리하여 집계로 인한 부정적인 영향을 완화한다.88
- **지식 증류:** 부분적 지식 증류(partial knowledge distillation)를 사용하여 클라이언트가 집계된 전역 모델로부터 관련 없는 정보에 압도되지 않으면서 선택적으로 유용한 지식을 학습하도록 한다.88
- **개인화 연합 학습 (PFL):** 데이터 이질성을 더 잘 처리하기 위해 각 클라이언트에 대해 맞춤형 모델을 훈련시킨다.85



FSL 시스템은 **"적대적 서포트 세트 공격(Adversarial Support Set Attack)"**이라는 독특한 유형의 포이즈닝 공격(poisoning attack)에 취약하다.90


이 공격은 테스트 시점의 쿼리 입력을 교란하는 대신, 공격자가 인간의 눈으로는 감지하기 어려운 미세한 교란이 가해진 예제들을 *서포트 세트*에 주입하는 방식으로 이루어진다.


메타 학습자(meta-learner)는 서포트 세트를 기반으로 신속하게 적응하도록 설계되었기 때문에, 이처럼 오염된 예제들은 학습 과정 자체를 손상시킬 수 있다. 모델은 새로운 클래스에 대해 근본적으로 결함이 있는 표현을 학습하게 되며, 이는 깨끗한 쿼리 예제에 대한 정확도를 무작위 추측보다도 못한 수준으로 급격히 떨어뜨리는 결과를 초래한다.90


이 공격은 FSL을 강력하게 만드는 바로 그 메커니즘, 즉 소수의 주어진 예제에 대한 높은 민감성을 악용한다. 이는 공격이 '학습하는 법을 배우는' 과정 자체를 직접적으로 표적으로 삼기 때문에 메타 학습자를 특히 취약하게 만든다.90




GPT-3/4와 같은 거대 언어 모델(LLM)은 모델의 가중치를 전혀 업데이트하거나 미세 조정하지 않고도, 입력 컨텍스트에 몇 가지 예제를 포함시키는 것만으로 새로운 작업을 수행할 수 있다.55 이 놀라운 현상은 

**인-컨텍스트 학습(In-Context Learning, ICL)** 또는 **퓨샷 프롬프팅(few-shot prompting)**으로 알려져 있다.


LLM은 본질적으로 패턴 완성 엔진이다. 사용자가 프롬프트에 입력-출력 예제 쌍을 제공함으로써, 모델의 자기-주의(self-attention) 메커니즘이 인식하고 새로운 쿼리에 대해 계속 이어갈 임시적인 패턴을 생성한다.55 모델은 프롬프트에 제공된 컨텍스트에 의해 활성화되는 방대한 사전 훈련 지식을 활용한다.51


- **전통적인 FSL (예: MAML, 프로토타입 네트워크):** 퓨샷 적응을 잘 수행하도록 모델의 파라미터를 명시적으로 최적화하는 메타-훈련 단계를 포함한다. 적응 과정 자체는 경사 하강법과 같은 가중치 업데이트를 포함할 수 있다.57
- **ICL:** 파라미터 업데이트가 전혀 없다. 학습은 '인-컨텍스트'에서 일어나며, 단일 순전파(forward pass) 동안에만 존재하는 일시적인 현상이다.57


LLM의 컨텍스트 창(context window)이 확장됨에 따라, 이제는 수백 또는 수천 개의 예제를 컨텍스트에 제공하는 것이 가능해졌다. 이러한 "다수-샷(many-shot)" 학습은 성능을 크게 향상시킬 수 있으며, 때로는 완전한 미세 조정에 필적하는 성능을 보이기도 한다. 또한, 모델이 사전 훈련 데이터로부터 학습한 편향을 극복하는 데 도움을 줄 수도 있다.91



ICL은 경험적으로는 매우 강력하지만, 어떻게 이런 능력이 발현되는지에 대해서는 아직 이론적으로 완전히 이해되지 않은 놀라운 현상이다.92


- **암묵적 최적화/미세 조정으로서의 ICL:** 한 가지 유력한 이론은 트랜스포머의 순전파 과정이 암묵적으로 경사 하강법 기반의 최적화 과정을 모방하고 있다는 것이다. 즉, 주의(attention) 메커니즘이 프롬프트의 예제들을 바탕으로 모델의 내부 표현에 대한 업데이트를 효과적으로 구성하고 적용하여, 실제로 가중치를 변경하지 않으면서 미세 조정을 흉내 낸다는 것이다.92
- **암묵적 베이지안 추론으로서의 ICL:** 또 다른 관점은 ICL을 암묵적인 베이지안 추론(Bayesian inference)의 한 형태로 보는 것이다. 모델이 프롬프트를 잠재적인 작업이나 개념을 추론하기 위한 증거로 취급하고, 이 추론된 개념을 사용하여 쿼리에 대한 예측을 수행한다는 설명이다.95
- **작업 식별로서의 ICL:** 이와 관련된 아이디어는, 사전 훈련 데이터의 혼합 분포에 이미 존재하는 작업들의 경우, ICL이 해당 작업을 처음부터 *학습*하는 것이 아니라 프롬프트로부터 올바른 잠재 작업을 *식별*하는 것에 가깝다는 것이다.94
- **대조 학습으로서의 ICL:** 일부 연구는 인-컨텍스트 학습 목표와 대조 학습 패턴 사이에 수학적 등가성이 있음을 보여준다. 이는 모델이 목표 예측을 주변 컨텍스트와 암묵적으로 비교함으로써 학습한다는 것을 시사한다.96

ICL에 대한 이론적 탐구는 AI 분야에서 가장 흥미로운 최전선 중 하나이다. 이는 트랜스포머 아키텍처 자체가 순전파 내에서 경사 하강법이나 베이지안 추론과 같은 다른 학습 알고리즘을 실행할 수 있는 범용 학습 알고리즘일 수 있음을 시사한다. 이는 모델을 정적인 함수로 보는 관점에서 동적인 인-메모리(in-memory) 컴퓨터로 보는 심오한 전환이다. ICL은 예상치 못한 발견이었고 56, 그 작동 원리에 대한 의문이 제기되었다. 가중치가 고정된 모델이 어떻게 '학습'할 수 있는가? 학습은 순전파 동안 활성화(activations)에서 일어나야만 한다. 주의 메커니즘은 전체 컨텍스트를 기반으로 정보를 동적으로 라우팅할 수 있는 유일한 구성 요소이므로, 주의 헤드가 학습 알고리즘과 유사한 계산을 수행한다는 가설로 이어졌다. "ICL은 암묵적 경사 하강법"과 같은 이론들은 95 주의 업데이트가 최적화기의 가중치 업데이트를 모방한다고 제안한다. 이것이 사실이라면, 우리는 단순한 모델이 아니라 

*메타-최적화기*를 훈련시켜 온 것이라는 엄청난 함의를 가진다. 사전 훈련 과정은 단지 사실을 배우는 것이 아니라, *학습하는 방법*을 배우고 이 절차를 주의 메커니즘의 구조에 인코딩하는 것이다. 이는 이전에 분리되었던 최적화 기반 메타 학습(MAML 등)과 거대 사전 훈련 모델 분야를 통합할 수 있는 가능성을 연다.


- **패러다임의 시너지:** FSL의 미래는 전통적인 메타 학습과 LLM 기반 ICL의 강점을 결합한 하이브리드 접근법에 있을 가능성이 높다. 예를 들어, 메타 학습을 사용하여 특정 작업에 대해 프롬프팅될 더 작고 전문화된 모델을 미세 조정하거나, ICL을 위한 프롬프트 선택 과정을 최적화할 수 있다.97
- **일반화와 견고성을 향한 노력:** 연구는 도메인 이동, 연속 학습, 적대적 견고성과 같은 핵심 과제를 극복하는 데 계속 초점을 맞출 것이며, FSL을 통제된 벤치마크에서 신뢰할 수 있는 실제 배포 환경으로 이동시킬 것이다.14
- **퓨샷에서 '레스-샷(Less-Shot)'으로:** 궁극적인 목표는 훨씬 적은 데이터와 감독을 필요로 하는 모델을 만드는 것이다. 이러한 모델은 가용한 정보에 따라 제로샷, 원샷, 퓨샷 체제를 유연하게 넘나들 수 있을 것이다.




FSL 모델, 특히 전이 학습이나 LLM 프롬프팅을 사용하는 모델은 사전 훈련된 지식에 크게 의존한다. 만약 기반 모델이 인터넷에서 수집된 정제되지 않은 대규모 데이터셋으로 훈련되었다면, 성별, 인종, 문화적 고정관념과 같은 사회적 편향을 그대로 내재하게 될 것이다.99


퓨샷 환경에서는 이러한 편향이 증폭될 수 있다. 모델은 소수의 작업별 예제만으로는 강력하고 편향된 사전 지식을 덮어쓸 충분한 데이터를 갖지 못한다. 예를 들어, 제로샷 모델에 "간호사 직업의 후보를 제안하라"고 프롬프트를 입력하면, 프롬프트 자체의 내용이 아니라 훈련 데이터의 패턴에 근거하여 여성의 이름을 불균형적으로 추천할 수 있다.100


개발자들은 몇 개의 '공정한' 예제를 제공하는 것만으로 모델의 행동을 교정할 수 있다고 잘못 생각할 수 있지만, 실제로는 그렇지 않은 경우가 많다. 기반 모델의 근본적인 편향은 지속될 수 있으며 미묘한 방식으로 발현될 수 있다.99



FSL의 의사 결정 과정은 불투명할 수 있다.

- LLM을 사용한 ZSL/FSL에서, 모델의 추론은 프롬프트의 몇 가지 예제뿐만 아니라 가중치에 담긴 방대하고 해석 불가능한 지식과 연결되어 있기 때문에, 모델이 왜 특정 출력을 내놓았는지 추적하기 어렵다.99
- 이러한 설명 가능성의 부족은 채용, 대출, 의료 진단과 같이 중대한 이해관계가 걸린 영역에서 책임성을 거의 불가능하게 만든다.99


FSL의 데이터 효율성은 AI 시스템 배포의 장벽을 낮추며, 이는 오용의 위험을 증가시킨다.100

- **성급한 배포:** 희귀 질병 진단과 같은 민감한 응용 분야의 모델이 단지 빠르게 적응하는 것처럼 보인다는 이유만으로 엄격한 검증 없이 배포될 수 있다.99
- **악의적 사용:** ZSL/FSL 시스템은 작업별 미세 조정이 거의 또는 전혀 필요하지 않기 때문에, 허위 정보, 가짜 뉴스, 편향된 콘텐츠를 대규모로 생성하는 데 쉽게 악용될 수 있다.99


이러한 과제를 해결하기 위해서는 다각적인 접근이 필요하다. 기반 모델에 대한 엄격한 편향 테스트, 저-데이터 환경에 맞는 설명 가능성 도구 개발, 중요 도메인에서의 책임감 있는 배포를 위한 명확한 가이드라인 수립, 그리고 잠재적 편향에 대한 프롬프트 감사 등이 포함된다.99


퓨샷 러닝은 데이터 집약적인 딥러닝의 한계를 극복하고, 인간과 유사한 학습 효율성을 기계에 구현하려는 중요한 패러다임 전환을 대표한다. 데이터 중심, 메트릭 기반, 최적화 기반, 모델 기반 접근법 등 다양한 방법론을 통해 FSL은 컴퓨터 비전, 자연어 처리, 로보틱스, 헬스케어 등 광범위한 분야에서 실질적인 가치를 입증했다. 특히 거대 언어 모델의 등장은 인-컨텍스트 학습이라는 새로운 FSL 방식을 제시하며 연구의 지평을 넓혔다.

그러나 FSL의 발전은 메타-과적합, 도메인 이동, 연속 학습에서의 치명적 망각, 그리고 연합 학습 환경에서의 데이터 이질성과 같은 고도화된 기술적 과제에 직면해 있다. 이러한 과제들은 FSL 연구가 단순한 빠른 적응을 넘어, 견고하고 일반화 가능하며 안전한 표현 학습으로 나아가야 함을 시사한다.

더 나아가, FSL 기술이 강력해지고 접근성이 높아짐에 따라, 내재된 편향 증폭, 투명성 부족, 오용 가능성과 같은 윤리적 문제들이 중요한 과제로 부상하고 있다. 따라서 FSL의 미래는 기술적 혁신과 함께 책임감 있는 개발 및 배포를 위한 윤리적 프레임워크를 구축하는 노력에 달려 있다. 궁극적으로 FSL은 데이터 희소성의 제약을 넘어, 더 적응력 있고 효율적이며 신뢰할 수 있는 인공지능 시스템을 향한 길을 열어줄 핵심적인 열쇠가 될 것이다.


1. Few-shot Learning, 퓨샷 러닝 - 코딩스뮤 - 티스토리, accessed July 19, 2025, https://codingsmu.tistory.com/154
2. What Is Few-Shot Learning? | IBM, accessed July 19, 2025, https://www.ibm.com/think/topics/few-shot-learning
3. What is Few-Shot Learning? - Moveworks, accessed July 19, 2025, https://www.moveworks.com/us/en/resources/ai-terms-glossary/few-shot-learning
4. What is Few-Shot Learning? Unlocking Insights with Limited Data - DataCamp, accessed July 19, 2025, https://www.datacamp.com/blog/what-is-few-shot-learning
5. Few-Shot Learning? 관련 논문을 중심으로 이해해보자! - 청춘, accessed July 19, 2025, [https://24bean.tistory.com/entry/Few-Shot-Learning-%EA%B4%80%EB%A0%A8-%EB%85%BC%EB%AC%B8%EC%9D%84-%EC%A4%91%EC%8B%AC%EC%9C%BC%EB%A1%9C-%EC%9D%B4%ED%95%B4%ED%95%B4%EB%B3%B4%EC%9E%90](https://24bean.tistory.com/entry/Few-Shot-Learning-관련-논문을-중심으로-이해해보자)
6. Few Shot Learning in AI: How It Works and Why It Matters - The IoT Academy, accessed July 19, 2025, https://www.theiotacademy.co/blog/few-shot-learning/
7. "What is Few-shot Learning? AI That Learns from Just a Few Examples" - Resources, accessed July 19, 2025, https://resources.rework.com/terms/ai-terms/few-shot-learning
8. Few-Shot Learning: A Cognitive Science Perspective - Number Analytics, accessed July 19, 2025, https://www.numberanalytics.com/blog/few-shot-learning-cognitive-science-perspective
9. Human few-shot learning of compositional instructions, accessed July 19, 2025, https://cims.nyu.edu/~brenden/papers/LakeEtAl2019CogSci.pdf
10. [1901.04587] Human few-shot learning of compositional instructions - arXiv, accessed July 19, 2025, https://arxiv.org/abs/1901.04587
11. [2402.03017] A Complete Survey on Contemporary Methods, Emerging Paradigms and Hybrid Approaches for Few-Shot Learning - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2402.03017
12. [1904.05046] Generalizing from a Few Examples: A Survey on Few-Shot Learning - arXiv, accessed July 19, 2025, https://arxiv.org/abs/1904.05046
13. 퓨샷 러닝이란 무엇인가요? | IBM, accessed July 19, 2025, https://www.ibm.com/kr-ko/think/topics/few-shot-learning
14. Challenges and Opportunities of Few-Shot Learning: A Survey - arXiv, accessed July 19, 2025, https://arxiv.org/html/2504.04017v1
15. Active Few-Shot Learning for Sound Event Detection | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/363646018_Active_Few-Shot_Learning_for_Sound_Event_Detection
16. Meta-Learning and Few-Shot Learning: Adapting Deep Learning Models to Learn from Limited Data - DSS Blog, accessed July 19, 2025, https://roundtable.datascience.salon/meta-learning-and-few-shot-learning-adapting-deep-learning-models-to-learn-from-limited-data
17. Few-shot and meta-learning methods for image understanding: a survey - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/371953426_Few-shot_and_meta-learning_methods_for_image_understanding_a_survey
18. Prototypical Siamese Networks for Few-shot Learning - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/343341609_Prototypical_Siamese_Networks_for_Few-shot_Learning
19. Few-Shot Learning for Low-Data Drug Discovery | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/365645071_Few-Shot_Learning_for_Low-Data_Drug_Discovery
20. N-Shot Learning: Zero vs. Single vs. Two vs. Few (2025), accessed July 19, 2025, https://viso.ai/deep-learning/n-shot-learning/
21. Zero-Shot, One-Shot, and Few-Shot Prompting: A Comparative Guide - PrajnaAI - Medium, accessed July 19, 2025, https://prajnaaiwisdom.medium.com/zero-shot-one-shot-and-few-shot-prompting-a-comparative-guide-ac38edd510d3
22. Prototypical Networks for Few-shot Learning - University of Toronto, accessed July 19, 2025, https://www.cs.toronto.edu/~zemel/documents/prototypical_networks_nips_2017.pdf
23. META-LEARNING WITH DOMAIN ADAPTATION FOR ... - OpenReview, accessed July 19, 2025, https://openreview.net/pdf?id=ByGOuo0cYm
24. Proposal-based Few-shot Sound Event Detection for Speech and Environmental Sounds with Perceivers - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2107.13616
25. [D] What is the difference between few-, one- and zero-shot learning? : r/MachineLearning, accessed July 19, 2025, https://www.reddit.com/r/MachineLearning/comments/boitjj/d_what_is_the_difference_between_few_one_and/
26. Zero-Shot vs One-Shot vs Few-Shot Learning - GeeksforGeeks, accessed July 19, 2025, https://www.geeksforgeeks.org/machine-learning/zero-shot-vs-one-shot-vs-few-shot-learning/
27. Mastering Few-Shot Learning in Robotics - Number Analytics, accessed July 19, 2025, https://www.numberanalytics.com/blog/few-shot-learning-robotics-guide
28. CFTS-GAN: Continual Few-Shot Teacher Student for Generative Adversarial Networks, accessed July 19, 2025, https://arxiv.org/html/2410.14749v1
29. some classic metric-based Few-Shot Learning methods based on pytorch 1.x - GitHub, accessed July 19, 2025, https://github.com/AndrewHYC/few-shot-learning
30. Memory Augmented Matching Networks for Few-Shot Learnings, accessed July 19, 2025, https://www.ijmlc.org/vol9/867-AM0018.pdf
31. Unlocking Few-Shot Learning: An Analysis of MAML | by ... - Medium, accessed July 19, 2025, https://medium.com/@vivekvjnk/unlocking-few-shot-learning-an-analysis-of-maml-274a5b57e8ef
32. HyperMAML: Few-Shot Adaptation of Deep Models with Hypernetworks - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2205.15745
33. Meta-Learning With Task-Adaptive Loss ... - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/ICCV2021/papers/Baik_Meta-Learning_With_Task-Adaptive_Loss_Function_for_Few-Shot_Learning_ICCV_2021_paper.pdf
34. HOW TO TRAIN YOUR MAML TO EXCEL IN FEW-SHOT CLASSIFICATION - OpenReview, accessed July 19, 2025, https://openreview.net/pdf?id=49h_IkpJtaE
35. Sharp-MAML: Sharpness-Aware Model-Agnostic Meta Learning - Proceedings of Machine Learning Research, accessed July 19, 2025, https://proceedings.mlr.press/v162/abbas22b/abbas22b.pdf
36. Enhancing Model Agnostic Meta-Learning via Gradient Similarity Loss - MDPI, accessed July 19, 2025, https://www.mdpi.com/2079-9292/13/3/535
37. How MAML works - An Interactive Introduction to Model-Agnostic Meta-Learning ‍, accessed July 19, 2025, https://interactive-maml.github.io/maml.html
38. What is the key difference in how MAML and Reptile approach meta ..., accessed July 19, 2025, [https://infermatic.ai/ask/?question=What%20is%20the%20key%20difference%20in%20how%20MAML%20and%20Reptile%20approach%20meta-learning?](https://infermatic.ai/ask/?question=What+is+the+key+difference+in+how+MAML+and+Reptile+approach+meta-learning?)
39. Sharp-MAML: Sharpness-Aware Model-Agnostic Meta Learning - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2206.03996
40. An Interactive Introduction to Model-Agnostic Meta-Learning ‍, accessed July 19, 2025, https://interactive-maml.github.io/comparison.html
41. Meta-Learning with Implicit Gradients - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/1909.04630
42. Combining Model-Agnostic Meta-Learning and Transfer Learning for Regression - PMC, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9865593/
43. One-shot Learning with Memory-Augmented Neural Networks - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/303367197_One-shot_Learning_with_Memory-Augmented_Neural_Networks
44. One-shot Learning with Memory-Augmented Neural Networks - arXiv, accessed July 19, 2025, https://arxiv.org/abs/1605.06065
45. Meta Learning with Memory Augmented Neural Networks | by Hey Amit | Medium, accessed July 19, 2025, https://medium.com/@heyamit10/meta-learning-with-memory-augmented-neural-networks-e62a3f831f63
46. Few-shot Image Generation with Elastic Weight Consolidation - NIPS, accessed July 19, 2025, https://papers.nips.cc/paper/2020/file/b6d767d2f8ed5d21a44b0e5886680cb9-Paper.pdf
47. Exploring Incompatible Knowledge Transfer in ... - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2023/papers/Zhao_Exploring_Incompatible_Knowledge_Transfer_in_Few-Shot_Image_Generation_CVPR_2023_paper.pdf
48. NeurIPS Poster DomainGallery: Few-shot Domain-driven Image Generation by Attribute-centric Finetuning, accessed July 19, 2025, https://neurips.cc/virtual/2024/poster/94641
49. NeurIPS Poster Transfer Learning for Diffusion Models, accessed July 19, 2025, https://neurips.cc/virtual/2024/poster/96508
50. Uncertainty-Aware Active Meta-Learning for Few-Shot Text Classification - MDPI, accessed July 19, 2025, https://www.mdpi.com/2076-3417/15/7/3702
51. What is few shot prompting? | IBM, accessed July 19, 2025, https://www.ibm.com/think/topics/few-shot-prompting
52. Making Pre-trained Language Models Better Continual Few-Shot Relation Extractors - arXiv, accessed July 19, 2025, https://arxiv.org/html/2402.15713v1
53. Making Pre-trained Language Models Better ... - ACL Anthology, accessed July 19, 2025, https://aclanthology.org/2024.lrec-main.957.pdf
54. Improving Continual Few-shot Relation Extraction through Relational Knowledge Distillation and Prototype Augmentation - ACL Anthology, accessed July 19, 2025, https://aclanthology.org/2024.lrec-main.767.pdf
55. How You Can Use Few-Shot Learning In LLM Prompting To Improve Its Performance, accessed July 19, 2025, https://dzone.com/articles/how-you-can-use-few-shot-learning-in-llm-prompting
56. Harness the Power of LLMs: Zero-shot and Few-shot Prompting, accessed July 19, 2025, https://www.analyticsvidhya.com/blog/2023/09/power-of-llms-zero-shot-and-few-shot-prompting/
57. Few-Shot and Zero-Shot Learning : Unlocking Cross-Domain Generalization - Medium, accessed July 19, 2025, https://medium.com/@anicomanesh/mastering-few-shot-and-zero-shot-learning-in-llms-a-deep-dive-into-cross-domain-generalization-b33f779f5259
58. Advancements in Zero-Shot and Few-Shot Learning for Large Language Models - GoML, accessed July 19, 2025, https://www.goml.io/blog/advancements-in-zero-shot-and-few-shot-learning-for-large-language-models
59. Few-Shot Learning in Robotics - Number Analytics, accessed July 19, 2025, https://www.numberanalytics.com/blog/few-shot-learning-robotics
60. Show and Grasp: Few-shot Semantic Segmentation for Robot Grasping through Zero-shot Foundation Models - arXiv, accessed July 19, 2025, https://arxiv.org/html/2404.12717v1
61. DemoGrasp: Few-Shot Learning for Robotic Grasping with Human ..., accessed July 19, 2025, https://www.researchgate.net/publication/357105465_DemoGrasp_Few-Shot_Learning_for_Robotic_Grasping_with_Human_Demonstration
62. Remote Task-oriented Grasp Area Teaching By Non-Experts through Interactive Segmentation and Few-Shot Learning, accessed July 19, 2025, https://ai4athome.github.io/res/papers/kaynar_remote_task_oriented.pdf
63. DexVIP: Learning Dexterous Grasping with Human Hand Pose Priors from Video, accessed July 19, 2025, https://proceedings.mlr.press/v164/mandikal22a/mandikal22a.pdf
64. Few-Shot Learning for Low-Data Drug Discovery - PubMed, accessed July 19, 2025, https://pubmed.ncbi.nlm.nih.gov/36410391/
65. Few-Shot Learning for Low Data Drug Discovery - University of Malta, accessed July 19, 2025, https://www.um.edu.mt/library/oar/bitstream/123456789/108340/1/2219ICTICS520000003154_1.PDF
66. Few-shot learning creates predictive models of drug ... - IDEKER LAB, accessed July 19, 2025, https://idekerlab.ucsd.edu/wp-content/uploads/2021/01/s43018-020-00169-2.pdf
67. Few-shot learning creates predictive models of drug response that translate from high-throughput screens to individual patients - PubMed, accessed July 19, 2025, https://pubmed.ncbi.nlm.nih.gov/34223192/
68. Few-shot learning creates predictive models of drug response that translate from high-throughput screens to individual patients - PubMed Central, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8248912/
69. Ligand-Based Compound Activity Prediction via Few-Shot Learning ..., accessed July 19, 2025, https://pubs.acs.org/doi/10.1021/acs.jcim.4c00485
70. Ligand-Based Compound Activity Prediction via Few-Shot Learning - ACS Publications, accessed July 19, 2025, https://pubs.acs.org/doi/pdf/10.1021/acs.jcim.4c00485
71. Ligand-Based Compound Activity Prediction via Few-Shot Learning - PMC, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11267577/
72. Few-shot meta-learning applied to whole brain activity maps improves systems neuropharmacology and drug discovery - PubMed, accessed July 19, 2025, https://pubmed.ncbi.nlm.nih.gov/39319265/
73. Few-Shot Speaker Identification Using Depthwise ... - ISCA Archive, accessed July 19, 2025, https://www.isca-archive.org/odyssey_2022/li22_odyssey.pdf
74. Structured Meta-Learning for Cross-Domain Few-Shot ... - POLITesi, accessed July 19, 2025, [https://www.politesi.polimi.it/retrieve/a1c8fe9c-a180-4e1c-a615-20da572242dd/2021_04_De%20Angeli.pdf](https://www.politesi.polimi.it/retrieve/a1c8fe9c-a180-4e1c-a615-20da572242dd/2021_04_De Angeli.pdf)
75. Domain-Adaptive Few-Shot Learning - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/WACV2021/papers/Zhao_Domain-Adaptive_Few-Shot_Learning_WACV_2021_paper.pdf
76. Adapting to Domain Shift by Meta-Distillation from Mixture-of-Experts - NIPS, accessed July 19, 2025, https://proceedings.nips.cc/paper_files/paper/2022/file/8bd4f1dbc7a70c6b80ce81b8b4fdc0b2-Paper-Conference.pdf
77. Meta-Learning with Domain Adaptation for Few-Shot Learning under Domain Shift, accessed July 19, 2025, https://openreview.net/forum?id=ByGOuo0cYm
78. Task-Level Self-Supervision for Cross-Domain Few-Shot Learning | Proceedings of the AAAI Conference on Artificial Intelligence, accessed July 19, 2025, https://ojs.aaai.org/index.php/AAAI/article/view/20230
79. Addressing Catastrophic Forgetting in Few-Shot Problems | Request ..., accessed July 19, 2025, https://www.researchgate.net/publication/360607940_Addressing_Catastrophic_Forgetting_in_Few-Shot_Problems
80. Meta-Continual Learning of Neural Fields - arXiv, accessed July 19, 2025, https://arxiv.org/html/2504.05806v1
81. La-MAML: Look-ahead Meta Learning for Continual Learning - NIPS, accessed July 19, 2025, https://papers.neurips.cc/paper/2020/file/85b9a5ac91cd629bd3afe396ec07270a-Paper.pdf
82. Online Fast Adaptation and Knowledge Accumulation ... - NIPS, accessed July 19, 2025, https://papers.nips.cc/paper/2020/file/c0a271bc0ecb776a094786474322cb82-Paper.pdf
83. [2003.05856] Online Fast Adaptation and Knowledge Accumulation: a New Approach to Continual Learning - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2003.05856
84. Continual Few-shot Learning with Transformer Adaptation and Knowledge Regularization, accessed July 19, 2025, [https://mn.cs.tsinghua.edu.cn/xinwang/PDF/papers/2023_Continual%20Few-shot%20Learning%20with%20Transformer%20Adaptation%20and%20Knowledge%20Regularization.pdf](https://mn.cs.tsinghua.edu.cn/xinwang/PDF/papers/2023_Continual Few-shot Learning with Transformer Adaptation and Knowledge Regularization.pdf)
85. MEFL: Meta-Equilibrize Federated Learning for Imbalanced Data in IoT - MDPI, accessed July 19, 2025, https://www.mdpi.com/1099-4300/27/6/553
86. FedFSLAR: A Federated Learning Framework for Few-Shot Action Recognition - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/WACV2024W/RWS/papers/Tu_FedFSLAR_A_Federated_Learning_Framework_for_Few-Shot_Action_Recognition_WACVW_2024_paper.pdf
87. UTILIZING FEDERATED LEARNING AND META LEARNING FOR FEW-SHOT LEARNING ON EDGE DEVICES - Rowan Digital Works, accessed July 19, 2025, https://rdw.rowan.edu/cgi/viewcontent.cgi?article=4071&context=etd
88. Federated Few-shot Learning - arXiv, accessed July 19, 2025, https://arxiv.org/html/2306.10234
89. Federated Few-Shot Learning for Mobile NLP - Dongqi Cai, accessed July 19, 2025, https://www.caidongqi.com/pdf/MobiCom23-FeS.pdf
90. Attacking Few-Shot Classifiers with Adversarial Support Sets ..., accessed July 19, 2025, https://openreview.net/forum?id=0xdQXkz69x9
91. Many-Shot In-Context Learning - NIPS, accessed July 19, 2025, https://proceedings.neurips.cc/paper_files/paper/2024/file/8cb564df771e9eacbfe9d72bd46a24a9-Paper-Conference.pdf
92. Probing the Decision Boundaries of In-context Learning in Large ..., accessed July 19, 2025, [https://arxiv.org/pdf/2406.11233?](https://arxiv.org/pdf/2406.11233)
93. Few-Shot Learning | Papers With Code, accessed July 19, 2025, https://paperswithcode.com/task/few-shot-learning
94. The Learnability of In-Context Learning | OpenReview, accessed July 19, 2025, [https://openreview.net/forum?id=f3JNQd7CHM¬eId=mMzuaUIQkL](https://openreview.net/forum?id=f3JNQd7CHM&noteId=mMzuaUIQkL)
95. Trained Transformers Learn Linear Models In-Context, accessed July 19, 2025, https://par.nsf.gov/servlets/purl/10526854
96. Towards Understanding How Transformers Learn In-context Through a Representation Learning Lens | AI Research Paper Details - AIModels.fyi, accessed July 19, 2025, https://www.aimodels.fyi/papers/arxiv/towards-understanding-how-transformers-learn-context-through
97. Why In-Context Learning Models are Good Few-Shot Learners? - OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=iLUcsecZJp
98. Toward Green and Human-Like Artificial Intelligence: A Complete Survey on Contemporary Few-Shot Learning Approaches - arXiv, accessed July 19, 2025, https://arxiv.org/html/2402.03017v2
99. What are the ethical challenges with few-shot and zero-shot learning?, accessed July 19, 2025, https://milvus.io/ai-quick-reference/what-are-the-ethical-challenges-with-fewshot-and-zeroshot-learning
100. What are the implications of few-shot and zero-shot learning for AI ..., accessed July 19, 2025, https://milvus.io/ai-quick-reference/what-are-the-implications-of-fewshot-and-zeroshot-learning-for-ai-ethics
101. The Ethical Implications of Artificial Intelligence in Modern Society - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/390299630_The_Ethical_Implications_of_Artificial_Intelligence_in_Modern_Society
102. Fairness-guided Few-shot Prompting for Large Language Models - OpenReview, accessed July 19, 2025, [https://openreview.net/forum?id=D8oHQ2qSTj¬eId=J2yKqqDNbI](https://openreview.net/forum?id=D8oHQ2qSTj&noteId=J2yKqqDNbI)
103. A Review of Bias and Fairness in Artificial Intelligence - Re-UNIR, accessed July 19, 2025, https://reunir.unir.net/bitstream/handle/123456789/15693/ip2023_11_001.pdf?sequence=1&isAllowed=y


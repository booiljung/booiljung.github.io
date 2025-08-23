# Few-Shot Learning을 위한 프로토타입 네트워크



지난 10여 년간 딥러닝은 다양한 분야에서 전례 없는 성공을 거두었으나, 이러한 성공의 이면에는 대규모 레이블링된 데이터셋에 대한 깊은 의존성이 존재한다.1 특히, 컨볼루션 신경망(CNN)이나 트랜스포머와 같이 수많은 파라미터를 가진 복잡한 모델들은 과적합(overfitting)을 방지하고 높은 일반화 성능을 달성하기 위해 방대한 양의 학습 데이터를 필요로 한다. 이러한 데이터 의존성은 새로운 문제에 접근하는 데 있어 심각한 병목 현상을 야기한다. 예를 들어, 희귀 질환에 대한 의료 영상 데이터나 특정 산업 공정에서 발생하는 불량 데이터처럼 본질적으로 데이터 수집이 어렵거나, 데이터에 레이블을 부착하는 데 막대한 비용과 시간이 소요되는 영역에서는 전통적인 딥러닝 모델을 적용하기 어렵다.2 소수의 새로운 데이터에 대해 대규모 모델을 단순히 재학습시키는 순진한 접근 방식은 심각한 과적합으로 이어져 실용적인 해법이 될 수 없다.4


이러한 데이터 부족 문제를 해결하기 위한 대안으로 Few-Shot Learning (FSL) 패러다임이 등장했다. FSL은 극소수의 레이블링된 예제만으로 새로운 클래스에 대한 일반화 능력을 갖춘 모델을 학습시키는 것을 목표로 하는 머신러닝 프레임워크이다.1 이는 단 몇 개의 예시만 보고도 새로운 개념을 학습하는 인간의 능력과 유사한 학습 방식을 기계에 구현하려는 시도이다.1 FSL 문제의 구조는 일반적으로 'N-way K-shot' 분류 과제로 정형화된다.1 여기서 '

N'은 분류해야 할 새로운 클래스의 수를, 'K'는 각 클래스에 대해 주어지는 레이블링된 예제(또는 "샷")의 수를 의미한다.

FSL의 학습 및 평가 과정은 '에피소드(episode)'라는 단위로 구성된다. 각 에피소드는 두 가지 주요 데이터셋으로 나뉜다.1

- **서포트셋 (Support Set):** N개의 클래스 각각에 대해 K개의 레이블링된 예제를 포함하는 데이터셋이다. 모델은 이 서포트셋을 참조하여 각 클래스의 일반적인 표현을 학습한다.
- **쿼리셋 (Query Set):** 서포트셋과 동일한 N개의 클래스에 속하지만, 서포트셋에는 포함되지 않은 새로운 예제들로 구성된 데이터셋이다. 모델은 서포트셋에서 학습한 내용을 바탕으로 쿼리셋의 예제들을 분류하며, 이를 통해 모델의 일반화 성능이 평가된다.

이러한 에피소드 기반 학습 구조는 테스트 시나리오를 학습 과정에서 모방함으로써, 모델이 단순히 학습 데이터의 특정 클래스를 암기하는 것이 아니라 새로운 클래스에 빠르게 적응하고 일반화하는 방법을 배우도록 강제한다.4 한편, FSL의 특수한 경우로 

K=1인 경우를 원샷 학습(One-Shot Learning), K=0인 경우를 제로샷 학습(Zero-Shot Learning)이라 칭한다. 제로샷 학습은 학습 예제 없이 속성(attribute)과 같은 부가 정보에만 의존하여 일반화해야 하는, FSL과는 구별되는 독자적인 문제 영역이다.1


FSL 문제를 해결하기 위한 핵심 전략 중 하나는 "학습하는 방법을 학습"하는 메타러닝(Meta-Learning)이다.6 메타러닝 프레임워크에서 모델은 단일 과제를 수행하도록 학습되는 것이 아니라, 수많은 다양한 학습 에피소드(과제)를 경험하며 새로운 미지의 과제에 신속하게 적응할 수 있는 일반적인 학습 전략 자체를 습득한다.1

메타러닝의 여러 접근법 중에서도, 특히 메트릭 기반 학습(Metric-based Learning)은 FSL 분야에서 지배적인 전략으로 자리 잡았다.1 메트릭 기반 학습의 핵심 아이디어는 복잡한 분류기를 학습하는 대신, 데이터 포인트를 특정 임베딩 공간(embedding space)으로 매핑하는 임베딩 함수를 학습하는 데 있다. 이 임베딩 공간은 유클리드 거리(Euclidean distance)와 같은 간단한 거리 척도만으로도 클래스 간의 구분이 명확해지도록 설계된다.10 분류는 쿼리 데이터의 임베딩을 서포트셋 데이터의 임베딩과 비교하는 방식으로 이루어지며, 이는 원리적으로 K-최근접 이웃(k-nearest neighbors) 알고리즘과 유사하다.1 프로토타입 네트워크는 이러한 메트릭 기반 학습 철학을 가장 명료하고 효과적으로 구현한 대표적인 모델이다.

FSL은 단일 알고리즘이 아니라 '문제에 대한 정의'라는 점을 이해하는 것이 중요하다.1 이는 '무엇을 할 것인가'(소수의 예제로부터 학습)와 '어떻게 할 것인가'(메타러닝, 전이 학습 등)를 명확히 구분한다. 프로토타입 네트워크의 성공은 단순히 아키텍처의 우수성 때문이 아니라, 그 내재된 가정('어떻게')이 FSL 문제의 제약('무엇을')과 매우 잘 부합하기 때문이다. 또한, 모델이 반복적으로 작고 독립적인 분류 과제를 해결하도록 강제하는 에피소드 기반 학습 방식은 그 자체로 강력한 정규화(regularization) 기법으로 작용한다. 이는 모델이 대규모 학습 데이터셋의 특정 클래스에 과적합되는 것을 방지하고, 대신 전이 가능한 '학습 절차'를 배우도록 유도하여 미지의 클래스에 대한 일반화 성능을 극대화한다.



프로토타입 네트워크의 근본적인 아이디어는 그 우아한 단순성에 있다. 이 모델은 동일한 클래스에 속하는 데이터 포인트들이 임베딩 공간 내에서 단일 대표점, 즉 '프로토타입(prototype)' 주위에 군집을 형성할 것이라는 가설에 기반한다.4 이러한 접근 방식은 매우 단순한 귀납적 편향(inductive bias)을 반영하는데, 이는 복잡한 모델이 쉽게 과적합될 수 있는 데이터가 제한적인 FSL 환경에서 오히려 큰 이점으로 작용한다.4 프로토타입 네트워크는 복잡한 결정 경계(decision boundary)를 학습하는 대신, 비선형 매핑 함수(임베딩 함수)를 통해 입력 데이터를 분류가 용이한 공간으로 변환한다. 이 공간에서 분류는 각 클래스의 프로토타입에 가장 가까운 데이터를 해당 클래스로 할당하는, 간단한 최근접 중심(nearest-centroid) 문제로 귀결된다.13


프로토타입 네트워크의 작동 메커니즘은 Snell 등이 발표한 원 논문에서 제안된 명확한 수학적 단계들로 정의된다.4

- **임베딩 함수 (Embedding Function):** 학습 가능한 파라미터 $φ$를 가진 신경망 $f_φ$ (일반적으로 이미지 데이터의 경우 CNN)는 고차원 입력 공간 $\mathbb{R}^D$에 있는 샘플 $x_i$를 저차원 임베딩 공간 $\mathbb{R}^M$으로 매핑한다. 이 함수의 출력은 특징 벡터 $f_φ(x_i)$이다.

- **프로토타입 계산 (Prototype Calculation):** 각 클래스 $k$에 대한 프로토타입 $c_k$는 해당 클래스에 속하는 서포트셋 $S_k$의 임베딩 벡터들의 성분별 평균(element-wise mean)으로 계산된다 4:
  $$
  c_k=\frac{1}{∣S_k∣} \sum _{(x_i,y_i)∈S_k} f_φ(x_i)
  $$
  **거리 척도 및 분류 (Distance Metric and Classification):** 분류는 쿼리 샘플 $x$의 임베딩 $f_φ(x)$와 각 클래스의 프로토타입 $c_k$ 간의 거리를 계산하여 수행된다. 모델은 각 클래스에 대한 확률 분포를 계산하기 위해, 계산된 거리의 음수 값에 소프트맥스(softmax) 함수를 적용한다 4:

$$
p_φ(y=k|x) = \frac{\exp(−d(f_φ(x),c_k))}{\sum_{k'}\exp(−d(f_φ(x),c_{k′}))}
$$

- **제곱 유클리드 거리의 선택 (The Choice of Squared Euclidean Distance):** 다양한 거리 함수 $d(\cdot, \cdot)$를 사용할 수 있지만, 저자들은 실험적, 이론적 근거를 통해 제곱 유클리드 거리(squared Euclidean distance), 즉 $d(z,c)=\|z−c \| ^2$의 사용을 정당화했다.4 이 선택은 매우 중요하다. 제곱 유클리드 거리는 브레그만 발산(Bregman divergence)의 일종으로, 브레그만 발산을 거리 척도로 사용할 경우 클러스터 내 모든 점들과의 평균 거리를 최소화하는 대표점은 해당 클러스터의 평균이라는 사실이 증명되어 있다.4 이는 서포트셋 샘플들의 평균을 프로토타입으로 사용하는 것에 대한 강력한 이론적 기반을 제공하며, 이로 인해 일반적으로 사용되는 코사인 유사도(cosine similarity)보다 우수한 성능을 보인다.4

- **손실 함수 (Loss Function):** 모델은 쿼리 샘플의 실제 클래스 k에 대한 음의 로그 가능도(negative log-likelihood), 즉 교차 엔트로피 손실(cross-entropy loss)을 최소화하는 방향으로 학습된다. 손실 $J(φ)$는 한 에피소드 내의 모든 쿼리 샘플에 대한 평균 손실로 정의된다 4:
  $$
  J(φ) = −\frac{1}{N_CN_Q} \sum_{k_1}^{N_C} \sum_{(x, y) \in Q_k}{} \log p_{\psi} (y = k | x)
  $$


학습은 원 논문의 알고리즘 1에 명시된 바와 같이 에피소드 단위로 진행된다.12 단일 학습 에피소드는 다음과 같은 절차를 따른다.18

1. **과제 샘플링 (Task Sampling):** 전체 학습 클래스 중에서 $N_C$개의 클래스를 무작위로 선택한다.
2. **서포트/쿼리 분할 (Support/Query Split):** 선택된 $N_C$개의 각 클래스에서, $N_S$개의 예제를 무작위로 추출하여 *서포트셋*을 구성하고, 나머지 예제 중에서 $N_Q$개를 추출하여 *쿼리셋*을 구성한다.
3. **프로토타입 계산 (Prototype Computation):** 임베딩 네트워크 fφ를 사용하여 서포트셋에 포함된 모든 예제의 임베딩을 계산한다. 각 클래스에 대해, 해당 클래스의 서포트 예제들의 임베딩을 평균 내어 프로토타입 $c_k$를 계산한다.
4. **손실 계산 (Loss Calculation):** 쿼리셋에 포함된 각 예제에 대해 임베딩을 계산한다. 이후, $N_C$개의 모든 프로토타입과의 거리를 계산하고 소프트맥스 함수를 통해 확률 분포를 생성한다. 쿼리 예제의 실제 클래스 레이블을 바탕으로 음의 로그 가능도 손실을 계산한다.
5. **파라미터 업데이트 (Parameter Update):** 모든 쿼리 예제에 대한 손실을 평균 낸 후, 확률적 경사 하강법(SGD)이나 Adam과 같은 최적화 알고리즘을 사용하여 임베딩 네트워크의 파라미터 $\psi$를 업데이트한다.19 이 과정은 수많은 에피소드에 걸쳐 반복된다.

프로토타입 네트워크는 단순히 분류 알고리즘이 아니라, 암묵적으로 클러스터링 알고리즘의 목표를 수행한다. 손실 함수는 쿼리 포인트와 정답 클래스의 프로토타입 간의 거리를 최소화하고, 오답 클래스의 프로토타입과의 거리는 최대화하도록 설계되었다. 이는 임베딩 함수 $f_φ$가 클래스 내 분산(intra-class variance)은 최소화하고 클래스 간 분산(inter-class variance)은 최대화하는 임베딩 공간을 학습하도록 직접적으로 유도한다. 이는 K-평균(K-Means) 클러스터링과 같은 알고리즘의 목적 함수와 본질적으로 동일하다. 따라서 프로토타입 네트워크는 간단한 K-평균과 유사한 분류기가 미지의 클래스에 대해서도 완벽하게 작동할 수 있도록 특징 공간 자체를 변환하는 방법을 학습하는 과정으로 해석할 수 있다.

또한, 이 모델의 성능은 거의 전적으로 학습된 임베딩 함수 $f_φ$에 의해 결정된다. 분류기 자체는 학습 가능한 파라미터가 없는 비모수적(non-parametric)인 최근접 중심 규칙으로 고정되어 있다. 이는 모델의 모든 '지능'이 임베딩 공간의 기하학적 구조에 압축되어 있음을 의미한다. 즉, 학습의 전체 목표는 미지의 클래스들조차도 자연스럽게 조밀하고 분리 가능한 군집을 형성하도록 특징 공간을 '왜곡'하는 것이다. 이는 최종 완전 연결 계층의 가중치와 같은 복잡한 결정 경계를 학습하는 데 초점을 맞추는 전통적인 분류 방식과는 근본적인 차이를 보인다.


이 섹션에서는 프로토타입 네트워크를 다른 주요 FSL 모델들과 비교하여, 각 모델의 설계 철학과 그에 따른 장단점을 명확히 분석한다.


프로토타입 네트워크는 매칭 네트워크(Matching Networks)를 직접적으로 단순화하고 개선한 모델로 평가받는다.20

- **매칭 네트워크:** 쿼리 샘플을 분류하기 위해 서포트셋의 *모든* 예제에 대한 어텐션(attention) 메커니즘을 사용하여 가중 평균을 계산한다. 즉, 쿼리 포인트와 모든 서포트 샘플 간의 유사도를 개별적으로 측정한다.4
- **프로토타입 네트워크:** 이 복잡한 샘플 단위의 어텐션 메커니즘을 각 클래스별 단일 산술 평균으로 대체한다.20 단, 1-shot($K=1$) 환경에서는 단일 샘플의 평균은 그 자신이기 때문에 두 모델은 동일하게 작동한다.20
- **효율성 및 확장성:** 이러한 단순화는 상당한 계산 효율성 향상으로 이어진다. K-way, N-shot 과제에서 쿼리 포인트당 계산 복잡도는 매칭 네트워크가 $O(KN)$인 반면, 프로토타입 네트워크는 $O(K)$이다.20 이는 샷의 수($N$)가 증가할수록 프로토타입 네트워크의 확장성이 훨씬 뛰어나다는 것을 의미한다.
- **귀납적 편향:** 복잡한 어텐션에서 단순한 평균으로의 전환은 더 강력하고 단순한 귀납적 편향을 모델에 부여하며, 이는 데이터가 부족한 FSL 환경에서 더 강건한 성능을 보인다.4


샴 네트워크(Siamese Networks)와 트리플렛 네트워크(Triplet Networks) 역시 메트릭 기반 학습 접근법에 속하지만, 학습 목표에서 근본적인 차이를 보인다.15

- **샴/트리플렛 네트워크:** 이 모델들은 주로 쌍(동일/다른 클래스) 또는 삼중항(기준점, 긍정, 부정 샘플) 단위의 데이터를 기반으로 대조 손실(contrastive loss) 또는 삼중항 손실(triplet loss)을 사용하여 일반적인 유사도 함수를 학습한다.15 학습은 개별 인스턴스 중심(instance-centric)으로 이루어지며, FSL의 에피소드 구조를 명시적으로 활용하지 않는다.
- **프로토타입 네트워크:** 각 에피소드 내에서 종단간(end-to-end)으로 분류 과제를 직접 학습한다. 손실은 쿼리 포인트와 클래스 프로토타입 간의 관계를 기반으로 계산되므로, 학습이 클래스 중심(class-centric)이며 최종 FSL 과제의 목표와 직접적으로 일치한다.24


이 비교는 메타러닝에 대한 두 가지 다른 철학을 보여준다.

- **프로토타입 네트워크 (메트릭 기반):** 테스트 시점에 모델 파라미터를 전혀 조정할 필요 없이 새로운 과제를 해결할 수 있는, 정적이고 고도로 일반화된 임베딩 공간을 학습한다.17 새로운 과제에 대한 '학습'은 단순히 프로토타입과 거리를 순방향으로 계산하는 과정에 불과하다.
- **MAML (Model-Agnostic Meta-Learning, 최적화 기반):** 어떤 단일 과제에도 최적화되어 있지 않지만, 새로운 과제에 신속하게 적응할 수 있도록 준비된 초기 파라미터 집합 θ를 학습한다. 새로운 과제가 주어지면, MAML은 서포트셋을 사용하여 몇 단계의 경사 하강법을 수행하여 과제 특화 파라미터 φ를 찾은 후 쿼리셋을 평가한다.26 이는 MAML을 더 유연하게 만들지만, 테스트 시점에서의 계산 비용을 증가시킨다.


관계 네트워크(Relation Networks)는 메트릭 학습 아이디어를 한 단계 더 확장한다.

- **프로토타입 네트워크:** 임베딩 간의 비교를 위해 고정된, 학습 불가능한 거리 척도(예: 제곱 유클리드 거리)를 사용한다.29
- **관계 네트워크:** 고정된 거리 함수를 학습 가능한 심층 신경망, 즉 '관계 모듈(relation module)'로 대체한다. 이 모듈은 쿼리와 서포트 샘플의 임베딩을 연결(concatenate)하여 입력받고, 유사도 점수를 출력한다. 이를 통해 모델은 유연하고 비선형적인 메트릭을 학습할 수 있지만, 모델의 복잡성이 증가하는 단점이 있다.29

이러한 비교를 통해 FSL 모델 설계의 핵심 원칙, 즉 모델의 복잡성, 계산 효율성, 그리고 유연성 사이의 직접적인 상충 관계가 드러난다. 프로토타입 네트워크는 '클래스는 임베딩 공간에서 단일 중심으로 표현될 수 있다'는 매우 강력하고 단순한 가정을 통해 효율성과 성능의 최적점(sweet spot)을 찾았다. 매칭 네트워크, 관계 네트워크, MAML과 비교했을 때 프로토타입 네트워크는 구조적으로 가장 단순하고 빠르다. 그럼에도 불구하고 이 모델이 최첨단 성능을 달성했다는 사실은 FSL과 같은 저데이터 환경에서는 유연하고 복잡한 모델보다 강력하고 올바른 귀납적 편향이 더 가치 있을 수 있음을 시사한다.

또한, 각 모델은 새로운 과제에 대한 '적응'이 발생하는 위치가 다르다. 샴 네트워크의 경우, 모든 적응은 사전 학습된 임베딩 공간에 내재되어 있다. 프로토타입 네트워크에서 새로운 과제에 대한 '학습'은 프로토타입을 순방향으로 계산하는 자명한 과정이다. 반면, MAML에서는 테스트 시점에 경사 하강법이라는 명시적인 최적화 과정을 통해 '학습'이 이루어진다. 이는 완전히 정적인 접근법(샴 네트워크)에서 완전히 동적인 접근법(MAML)까지의 스펙트럼을 보여주며, 프로토타입 네트워크는 거의 정적인 모델에 위치한다. 이는 프로토타입 네트워크가 실제 배포 환경에서 매우 빠르고 간단하다는 중요한 실용적 이점을 제공한다.

마지막으로, 프로토타입 네트워크가 제로샷 학습(ZSL)으로 자연스럽게 확장될 수 있다는 점은 매칭 네트워크와 같은 모델에 비해 종종 간과되는 중요한 장점이다.20 서포트 '예제'의 임베딩에 근본적으로 의존하는 매칭 네트워크는 예제가 없는 ZSL로의 확장이 자연스럽지 않다. 반면, 클래스 '표현'의 학습에 초점을 맞추는 프로토타입 네트워크는 프로토타입의 소스를 서포트 예제의 평균에서 클래스 메타데이터의 임베딩으로 원활하게 전환할 수 있다.4 이는 프로토타입 네트워크 프레임워크가 단순한 예제 비교 모델이 아니라, 클래스 표현에 대한 더 깊고 개념적인 견고함을 가지고 있음을 보여준다.


다음 표는 앞서 논의된 주요 메트릭 기반 FSL 모델들의 핵심적인 특징을 요약하여 비교한다.

| 모델                     | 핵심 아이디어                                   | 학습 목표                                            | 테스트 시점 복잡도 | 주요 강점                                             | 주요 약점                                      |
| ------------------------ | ----------------------------------------------- | ---------------------------------------------------- | ------------------ | ----------------------------------------------------- | ---------------------------------------------- |
| **샴/트리플렛 네트워크** | 쌍(pair) 또는 삼중항(triplet) 기반 유사도 학습  | 대조/삼중항 손실을 통해 유사/비유사 쌍의 거리를 조절 | O(N×K)             | 간단한 유사도 학습, 다양한 응용                       | FSL 과제와 직접적인 연관성 부족, 학습 비효율성 |
| **매칭 네트워크**        | 쿼리와 모든 서포트 샘플 간의 어텐션 기반 가중합 | 에피소드 기반 N-way 분류 손실                        | O(N×K)             | 어텐션을 통한 유연한 샘플 가중치 부여                 | 계산 복잡도가 높고 샷 수에 따라 확장성 저하    |
| **프로토타입 네트워크**  | 임베딩 공간 내 클래스별 평균 프로토타입         | 에피소드 기반 N-way 분류 손실                        | O(N)               | 단순성, 높은 계산 효율성, 강력한 성능, ZSL 확장성     | 클래스가 구형 군집을 형성한다고 가정           |
| **관계 네트워크**        | 학습 가능한 비선형 관계 모듈을 통한 유사도 측정 | 에피소드 기반 N-way 분류 손실 (관계 점수 기반)       | O(N×K)             | 고정된 메트릭의 한계를 극복하고 복잡한 관계 학습 가능 | 모델 복잡성 증가, 과적합 위험                  |


이 섹션에서는 프로토타입 네트워크의 핵심 가정이 가지는 근본적인 한계를 비판적으로 검토하고, 이를 극복하기 위해 제안된 후속 연구들을 심도 있게 다룬다.


프로토타입 네트워크의 가장 큰 강점인 단순성은 동시에 가장 큰 약점의 원인이 된다. 전체 클래스를 단일 평균 벡터(중심점)로 표현하는 방식은 해당 클래스의 데이터 포인트들이 임베딩 공간 내에서 조밀하고 구형(Gaussian-like)의 군집을 형성할 때만 효과적이다.31

이 가정은 클래스 내 분산이 크거나(예: '개'라는 클래스 내에 다양한 품종이 존재하는 경우), 데이터가 본질적으로 비볼록(non-convex)적이거나 길쭉한 형태의 군집을 형성하는 경우 실패한다.14 이러한 경우, 계산된 중심점은 클래스 분포를 제대로 대표하지 못하며, 심지어 임베딩 공간 내에서 데이터 밀도가 낮은 지역에 위치할 수도 있다.


가우시안 프로토타입 네트워크(Gaussian Prototypical Networks, GPNs)는 중심점 가정의 한계를 보완하기 위해 불확실성을 도입한 초기 변형 모델이다.

- **핵심 아이디어:** GPN의 인코더는 입력을 단일 포인트 임베딩으로 매핑하는 대신, 평균 벡터와 공분산 행렬을 함께 출력한다. 이를 통해 각 데이터 포인트의 임베딩을 가우시안 분포로 모델링한다.33
- **수학적 공식:** 프로토타입은 더 이상 단순한 평균이 아니라, 각 서포트 포인트의 정밀도(공분산의 역행렬)에 의해 가중된 평균으로 계산된다. 거리 척도 또한 이러한 분포를 고려하도록 수정되어, 분산이 큰 노이즈나 이상치 샘플의 영향을 효과적으로 줄인다. 이는 서포트 샘플이 클래스를 완벽하게 대표하지 못할 때 더 강건한 프로토타입을 생성하게 한다.
- **이점:** GPNs는 노이즈가 많은 데이터를 더 잘 처리할 수 있으며, 단일 포인트 프로토타입보다 클래스 분포를 더 미묘하고 정확하게 표현할 수 있다.33


이 변형 모델은 거리 척도 자체를 변경하여 임베딩 공간의 품질을 향상시키는 데 초점을 맞춘다.

- **핵심 아이디어:** SEN(Squared root of the Euclidean distance and the Norm distance) 메트릭은 손실 함수의 바람직하지 않은 비볼록성을 유발할 수 있는 명시적인 L2 정규화 없이, 더 판별력 있는 특징 정규화를 가능하게 하기 위해 제안되었다.36
- **작동 방식:** SEN 메트릭은 유클리드 거리와 놈(norm) 거리 항을 결합한다. 이 메트릭의 그래디언트는 두 가지 효과를 동시에 가진다. 첫째, 임베딩을 정답 프로토타입으로 끌어당기고 오답 프로토타입으로부터 밀어낸다. 둘째, 모든 임베딩의 놈이 동일해지도록 유도한다. 이는 모든 임베딩을 초구(hypersphere) 표면에 위치시켜, 더 조밀하고 잘 분리된 특징 공간을 형성하게 한다.
- **이점:** SEN은 기존 프로토타입 네트워크에 비해 무시할 수 있는 수준의 계산 오버헤드로, 특히 고차원 임베딩 공간에서 일관되게 더 나은 성능을 보인다.36


최신 연구 동향은 단순한 평균이나 단일 가우시안 분포를 넘어, 더 정교한 프로토타입 생성 방식을 탐구하고 있다.

- **ProtoDiff:** 이 모델은 강력한 생성 모델링 기법인 확산 과정(diffusion process)을 사용하여 프로토타입을 정제한다. 단순한 초기 프로토타입(예: 평균)에서 특정 과제에 더 최적화된, '과적합된' 프로토타입으로 점진적으로 변화하는 생성 과정을 학습한다.37 이는 이상적인 프로토타입이 단순한 평균이 아니라 서포트셋의 맥락에 따라 결정되는 더 복잡한 함수일 수 있음을 시사한다.
- **IPNET (Influential Prototypical Networks):** 이 접근법은 모든 서포트 샘플이 프로토타입 계산에 동일하게 중요하지 않다는 가정에서 출발한다. 최대 평균 불일치(Maximum Mean Discrepancy, MMD)를 사용하여 각 서포트 샘플의 '영향력 가중치'를 계산하고, 이를 통해 프로토타입 계산을 왜곡할 수 있는 이상치의 영향을 줄인다.38
- **그래프 프로토타입 네트워크 (GPNs for graphs):** 그래프 구조 데이터의 경우, 그래프 신경망(GNN)을 사용하여 노드를 인코딩하고, 추가적으로 각 노드의 정보성을 평가하는 평가자(valuator) 모듈을 도입하여 노드 분류 과제를 위한 더 강건한 프로토타입을 생성한다.39

프로토타입 네트워크의 발전 과정은 명확한 지적 궤적을 보여준다. 초기 모델은 결정론적인 프로토타입(평균)을 사용했다. 첫 번째 개선의 물결(GPNs)은 확률론적 관점(가우시안 분포)을 도입했다. 가장 최근의 연구들(ProtoDiff, IPNET)은 주어진 서포트셋의 맥락에 따라 동적으로 생성되거나, 불균일하게 가중된 함수로 계산되는 동적 프로토타입 개념을 도입했다. 이는 단순한 통계에서 정교한 모델링으로 나아가는 명확한 발전 경로를 보여준다.

또한, 이러한 한계점과 해결책들은 프로토타입 네트워크 프레임워크 내에 두 가지 별개의 개선 지점이 존재함을 드러낸다: (1) 개별 샘플의 **표현(representation)** 자체(임베딩 $f_φ(x)$)와 (2) 이러한 표현들을 프로토타입으로 **집계(aggregation)**하는 방식(프로토타입 $c_k$를 계산하는 함수). SEN 메트릭은 주로 임베딩 공간의 기하학적 구조를 변경하여 **표현**을 개선하는 데 초점을 맞춘다. 반면, GPNs, IPNET, ProtoDiff는 단순한 산술 평균을 가중 평균, 생성 과정, 또는 더 복잡한 통계적 추정치로 대체하여 **집계** 단계를 개선한다. 이러한 이분법적 분석은 프로토타입 네트워크의 다양한 변형 모델을 이해하고 분류하는 강력한 틀을 제공한다.


이 섹션에서는 프로토타입 네트워크가 다양한 분야에서 FSL 문제 해결을 위한 핵심 방법론으로 채택되며 보여준 광범위한 영향력과 다재다능함을 조명한다.


- **이미지 분류:** 프로토타입 네트워크는 발표 당시 Omniglot, *mini*ImageNet과 같은 표준 FSL 벤치마크에서 최고 수준의 성능을 달성하며, 강력한 베이스라인 모델로서의 입지를 확고히 했다.5
- **분류를 넘어서:** '프로토타입'이라는 핵심 아이디어는 더 복잡한 과제에도 성공적으로 적용되었다. 예를 들어, few-shot 키포인트 탐지에서는 특징 맵(feature map)에서 임베딩을 풀링(pooling)하여 특정 키포인트를 나타내는 프로토타입을 구성하고, 이를 쿼리 이미지의 특징 맵과 상호 연관시켜 키포인트를 탐지하는 방식으로 활용된다.5


의료 분야는 본질적으로 데이터가 부족하기 때문에 FSL과 프로토타입 네트워크의 주요 응용 분야로 부상했다.

- **의료 영상 분석:** 프로토타입 네트워크는 피부 병변 분류와 같은 과제에 널리 사용되며, 동일 도메인(in-domain)뿐만 아니라 교차 도메인(cross-domain) 환경에서의 성능까지 평가받고 있다.41 또한, few-shot 의료 영상 분할(segmentation) 분야에서도 서포트셋과 쿼리셋의 정보를 융합하여 더 나은 프로토타입을 생성하는 SQPFNet과 같은 고급 모델의 기반이 되고 있다.42 프로토타입 기반 접근법은 해석 가능성 측면에서도 장점을 가지는데, 폐 결절 분류를 위한 Proto-Caps 모델은 프로토타입을 통해 사례 기반 추론(case-based reasoning)을 제공하여 의사결정 과정을 설명한다.43
- **신약 개발 및 유전체학:** 데이터가 부족한 신약 개발 분야에서 프로토타입 네트워크는 few-shot 분자 특성 예측에 활용된다.44 이 모델은 독성 예측 데이터셋에서 매칭 네트워크보다 우수한 성능을 보이면서도 학습 속도는 훨씬 빠르다는 것이 입증되었다.45 APN과 같은 변형 모델은 분자 지문(fingerprint)과 같은 속성 정보를 활용하여 프로토타입 생성을 유도함으로써 성능을 향상시킨다.44 유전체학에서는 유전체 저장소의 텍스트 데이터로부터 특정 면역 요법 연구를 식별하거나 46, 유전자 조절 네트워크에서 few-shot 노드 분류 과제를 해결하는 데 사용된다.39


- **오디오 및 음성 인식:** 프로토타입 네트워크는 높은 효율성 덕분에 음향 이벤트 분류 및 온디바이스(on-device) FSL 분야에서 널리 사용되는 메트릭 기반 방법이다.47 화자 인증에서는 화자 임베딩의 중심점을 프로토타입으로 계산하여 활용하며 49, 음성 스푸핑 알고리즘 인식과 같은 더 복잡한 과제에서는 그래프 기반 변형 모델(DGPN)이 발화 표현을 모델링하는 데 사용된다.51
- **다중모드 접근법:** 특히 혁신적인 응용 사례는 대규모 다중모드(multimodal) 모델을 활용하는 것이다. 한 연구에서는 "개가 짖는 소리"와 같은 텍스트 프롬프트를 사용하여 오디오-텍스트 공동 임베딩 공간을 쿼리하고, 관련 오디오 샘플 군집을 찾아 그 중심점을 계산함으로써, 인간의 감독 없이 오디오 프로토타입을 생성하는 방법을 제안했다.52 이는 새로운 소리 클래스에 대한 레이블링된 서포트셋이 필요하다는 기존의 제약을 우아하게 해결한다.


프로토타입 네트워크 프레임워크는 다른 FSL 방법들과 달리 제로샷 학습(ZSL)으로 자연스럽게 확장된다는 중요한 장점을 가진다.20

- **작동 원리:** ZSL에서는 서포트 예제가 전혀 없다 (K=0). 대신, 각 클래스는 "날개가 있다", "파란색이다"와 같은 의미론적 속성 벡터와 같은 메타데이터로 설명된다. 각 클래스의 프로토타입은 이 메타데이터 벡터를 데이터(예: 이미지)와 동일한 임베딩 공간으로 매핑하는 임베딩 함수의 출력이 된다.4 이후 분류 과정은 쿼리 이미지의 임베딩과 가장 가까운 클래스 프로토타입을 찾는 방식으로 동일하게 진행된다. 이 접근법은 CUB-200 조류 데이터셋과 같은 벤치마크에서 최고 수준의 성능을 달성했다.4

다양한 응용 사례를 통해 프로토타입 네트워크가 FSL 개념과 특정 전문 분야를 연결하는 '방법론적 다리' 역할을 수행한다는 점을 알 수 있다. 연구자들은 종종 기본적인 프로토타입 네트워크 아키텍처를 채택한 후, 특정 구성 요소(인코더, 프로토타입 계산 방식, 데이터 소스 등)를 해당 분야의 고유한 제약 조건에 맞게 수정한다. 예를 들어, 신약 개발에서는 이미지용 CNN 인코더를 분자 그래프용 GNN으로 대체하고 39, 다중모드 오디오에서는 오디오 서포트 샘플이 아닌 텍스트 쿼리를 통해 프로토타입을 '발견'한다.52 이 모든 경우에 '임베딩 획득 -->> 클래스 프로토타입 계산 -->> 거리 기반 분류'라는 핵심 논리는 동일하게 유지된다. 이러한 모듈성과 개념적 단순성이 프로토타입 네트워크를 매우 적응력 있고 강력한 연구 도구로 만들었다.

또한, 이러한 응용 사례들은 문제의 프레임이 '데이터 부족'에서 '주석 부족(annotation-scarce)'으로 미묘하게 전환되고 있음을 보여준다. 다중모드 오디오 시스템은 레이블이 없는 방대한 오디오 데이터 풀에 접근할 수 있다. 여기서 문제는 오디오 클립의 부족이 아니라, 사용자가 관심 있는 특정 새 클래스로 레이블링된 클립의 부족이다. 텍스트 프롬프트를 사용함으로써, 시스템은 레이블이 없는 풀에서 즉석으로 '레이블링된 서포트셋'을 생성할 수 있다. 이는 FSL의 핵심적인 미래 방향이 단순히 작은 데이터셋을 다루는 것을 넘어, 방대한 양의 레이블 없는 데이터를 영리하게 활용하여 few-shot 과제를 해결하는 것임을 시사한다.



- **핵심 구성 요소:** 프로토타입 네트워크를 구현하기 위한 실질적인 구성 요소는 다음과 같다.18
  - **임베딩 네트워크:** 일반적으로 4계층 ConvNet이나 사전 학습된 ResNet과 같은 표준 CNN 아키텍처가 사용되어 특징 벡터를 출력한다.2
  - **프로토타입 배치 샘플러 (Prototypical Batch Sampler):** 일반적인 데이터 샘플러와 달리, 각 배치를 완전한 N-way, K-shot 에피소드로 구성하는 맞춤형 샘플러가 필수적이다. 이 샘플러는 먼저 N개의 클래스를 샘플링한 후, 각 클래스에 대해 $K$개의 서포트 샘플과 $Q$개의 쿼리 샘플을 샘플링하여 하나의 미니배치를 구성한다.18
  - **손실 함수:** 모델의 출력 임베딩과 레이블을 입력받아 프로토타입을 계산하고, 제곱 유클리드 거리를 측정하여 최종 교차 엔트로피 손실을 반환하는 모듈이다.18
- **하이퍼파라미터:** 학습률, 에포크당 에피소드 수, 그리고 학습 및 검증에 사용될 에피소드의 구조(N-way, K-shot) 등이 주요 조정 대상 하이퍼파라미터이다.18

프로토타입 배치 샘플러는 단순한 구현 세부 사항이 아니라, 메타러닝 전략 전체를 실제로 구현하는 핵심 개념이다. 표준 데이터 로더가 개별 데이터 포인트를 샘플링하는 반면, 이 맞춤형 샘플러는 '과제' 전체를 샘플링한다. 데이터 파이프라인의 이러한 구조적 차이가 바로 모델이 "학습하는 방법을 학습"하도록 강제하는 에피소드 기반 학습을 가능하게 한다. 따라서 이 샘플러를 이해하는 것은 메타러닝 철학의 실제 적용을 이해하는 데 핵심적이다.


프로토타입 네트워크는 단순성, 우아함, 효율성, 그리고 강력한 성능의 조화를 통해 FSL 분야의 기념비적인 모델로 자리매김했다.14 이 모델은 이전 모델들(예: 매칭 네트워크)의 복잡한 메커니즘을 단순화하면서도 명확하고 직관적이며 매우 효과적인 베이스라인을 제공했다. 그 이후로 수많은 후속 연구에서 표준 벤치마크이자 기초 구성 요소로 활용되고 있다.17


- **교차 도메인 및 개방형 세계 FSL (Cross-Domain and Open-World FSL):** 현재 가장 큰 과제 중 하나는 단순히 새로운 클래스가 아니라, 근본적인 데이터 분포가 다른 새로운 '도메인'으로 일반화하는 것이다 (예: 자연 이미지로 학습하고 의료 영상으로 테스트).2 미래 연구는 이러한 도메인 변화에 더 강건한 모델을 개발하여, 데이터가 깨끗하고 정적이며 고정된 분포에서 온다고 가정하지 않는 '개방형 세계(open-world)' FSL로 나아가는 것을 목표로 한다.56
- **에피소드 기반 학습의 재고찰:** 복잡한 에피소드 기반 학습 방식의 필요성에 대한 의문이 제기되고 있다. 일부 연구에서는 표준 교차 엔트로피 손실로 학습된 단순한 베이스라인이 메타러닝 방법보다 우수한 성능을 보일 수 있으며, 프로토타입 네트워크와 같은 모델의 경우 비-에피소드 방식의 학습이 더 간단하고 효과적일 수 있음을 보여주었다.17 이는 FSL 커뮤니티의 핵심적인 논쟁점이자 향후 연구 방향이다.
- **대규모 사전 학습 모델과의 통합:** CLIP이나 거대 언어 모델(LLM)과 같은 대규모 사전 학습 모델과 FSL 기술을 융합하는 것이 중요한 미래 방향이다.9 이러한 모델들은 방대한 사전 지식을 내포하고 있어, 이를 활용하면 광범위한 메타 트레이닝의 필요성을 줄이면서 few-shot 성능을 극적으로 향상시킬 수 있다.
- **핵심 한계점 해결:** 복잡하고 비구형적인 클래스 분포에 대해 더 강건한 프로토타입을 생성하기 위한 연구는 변형 모델들의 지속적인 개발에서 볼 수 있듯이 여전히 핵심 주제로 남아있다.37

프로토타입 네트워크의 단순성은 그 자체로 강력한 진단 도구로서의 가치를 지닌다. 만약 표준 프로토타입 네트워크가 새로운 데이터셋에서 낮은 성능을 보인다면, 이는 명확한 신호를 제공한다. 즉, 해당 데이터셋의 클래스들이 현재 임베딩 공간에서 단순한 구형 군집을 형성하지 않는다는 것이다. 이러한 실패는 해석 가능하다. 이는 문제가 임베딩 함수 자체에 있거나(충분히 강력하지 않음), 데이터의 근본적인 구조에 있음(본질적으로 복잡하고 다중 모드임)을 알려준다. 이 진단은 연구자가 GPN이나 관계 네트워크와 같은 더 발전된 모델을 선택하도록 안내할 수 있다. 이러한 진단적 가치는 프로토타입 네트워크가 베이스라인으로서 지속적으로 사용되는 핵심 이유 중 하나이다.


다음 표는 주요 벤치마크 데이터셋에서 프로토타입 네트워크와 그 변형 모델들의 성능을 정량적으로 비교하여, 모델의 효과와 후속 연구를 통한 개선 정도를 보여준다.

| 모델                        | 데이터셋           | 과제   | 정확도 (%) |
| --------------------------- | ------------------ | ------ | ---------- |
| **Matching Networks** 20    | *mini*ImageNet     | 1-Shot | 43.56      |
|                             |                    | 5-Shot | 55.31      |
| **Prototypical Networks** 4 | *mini*ImageNet     | 1-Shot | 49.42      |
|                             |                    | 5-Shot | 68.20      |
| **Relation Networks** 29    | *mini*ImageNet     | 1-Shot | 50.44      |
|                             |                    | 5-Shot | 65.32      |
| **SEN-PN** 36               | *mini*ImageNet     | 5-Shot | 69.8       |
| **Prototypical Networks** 4 | CUB-200-2011 (ZSL) | -      | 55.5       |

*참고: 정확도는 각 논문에서 보고된 수치이며, 실험 설정에 따라 약간의 차이가 있을 수 있습니다.*

표에서 볼 수 있듯이, 프로토타입 네트워크는 매칭 네트워크에 비해 *mini*ImageNet 벤치마크에서 상당한 성능 향상을 보였다. 특히 5-shot 과제에서 그 격차가 두드러진다. 이는 프로토타입 기반의 단순한 귀납적 편향이 K>1인 경우 더 효과적으로 작용함을 시사한다. 관계 네트워크는 1-shot에서는 약간의 우위를 보였으나 5-shot에서는 성능이 하락했으며, SEN-PN과 같은 향상된 메트릭을 사용한 모델은 기존 프로토타입 네트워크의 성능을 더욱 개선했음을 확인할 수 있다. 또한, 제로샷 학습에서도 강력한 성능을 보여 모델의 확장성을 입증했다. 이러한 정량적 결과는 본 보고서에서 논의된 이론적 장점들을 실증적으로 뒷받침한다.


1. What Is Few-Shot Learning? | IBM, accessed July 19, 2025, https://www.ibm.com/think/topics/few-shot-learning
2. Enhancing Prototypical Networks for Few-Shot Learning - Stanford University, accessed July 19, 2025, https://web.stanford.edu/~markcx/sample-project/CS231N_project_metaLearning.pdf
3. Unleashing the Power of Few Shot Learning - Analytics Vidhya, accessed July 19, 2025, https://www.analyticsvidhya.com/blog/2023/07/few-shot-learning/
4. Prototypical Networks for Few-shot Learning - University of Toronto, accessed July 19, 2025, https://www.cs.toronto.edu/~zemel/documents/prototypical_networks_nips_2017.pdf
5. Prototypical Networks for Few-shot Learning | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/315096921_Prototypical_Networks_for_Few-shot_Learning
6. Understanding Few-Shot Learning in Computer Vision: What You Need to Know, accessed July 19, 2025, https://neptune.ai/blog/understanding-few-shot-learning-in-computer-vision
7. Prototypical networks. 1. Introduction : | by MAROUA CHANBI | Medium, accessed July 19, 2025, https://medium.com/@researchwormwhole777/prototypical-ne-5425ee8b25d3
8. What is Few-Shot Learning? Strategies and Examples - Roboflow Blog, accessed July 19, 2025, https://blog.roboflow.com/few-shot-learning/
9. A Comprehensive Survey of Few-shot Information Networks, accessed July 19, 2025, https://www.mi-research.net/en/article/pdf/preview/10.1007/s11633-023-1470-4.pdf
10. Few-Shot Metric Learning: Online Adaptation of Embedding for Retrieval - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/ACCV2022/papers/Jung_Few-Shot_Metric_Learning_Online_Adaptation_of_Embedding_for_Retrieval_ACCV_2022_paper.pdf
11. Everything you need to know about Few-Shot Learning - DigitalOcean, accessed July 19, 2025, https://www.digitalocean.com/community/tutorials/few-shot-learning
12. Prototypical Networks for Few-shot Learning, accessed July 19, 2025, https://arxiv.org/pdf/1703.05175
13. Hands-On-Meta-Learning-With-Python/03. Prototypical Networks and its Variants/3.1 Prototypical Networks.ipynb at master - GitHub, accessed July 19, 2025, [https://github.com/sudharsan13296/Hands-On-Meta-Learning-With-Python/blob/master/03.%20Prototypical%20Networks%20and%20its%20Variants/3.1%20Prototypical%20Networks.ipynb](https://github.com/sudharsan13296/Hands-On-Meta-Learning-With-Python/blob/master/03. Prototypical Networks and its Variants/3.1 Prototypical Networks.ipynb)
14. What is a prototype network in few-shot learning? - Milvus, accessed July 19, 2025, https://milvus.io/ai-quick-reference/what-is-a-prototype-network-in-fewshot-learning
15. Prototypical Networks Explained, Compared & How To Tutorial - Spot Intelligence, accessed July 19, 2025, https://spotintelligence.com/2023/12/07/prototypical-networks/
16. Prototypical Networks for Few-shot Learning - NIPS, accessed July 19, 2025, https://papers.nips.cc/paper/6996-prototypical-networks-for-few-shot-learning
17. On Episodes, Prototypical Networks, and Few-Shot Learning, accessed July 19, 2025, https://proceedings.neurips.cc/paper/2021/file/cdfa4c42f465a5a66871587c69fcfa34-Paper.pdf
18. Prototypical-Networks-for-Few-shot-Learning-PyTorch - GitHub, accessed July 19, 2025, https://github.com/orobix/Prototypical-Networks-for-Few-shot-Learning-PyTorch
19. Few Shot Learning with Code - Meta Learning - Prototypical Networks - YouTube, accessed July 19, 2025, https://m.youtube.com/watch?v=YkeE7oRxF24&pp=ygUII2Zld3Nob3Q%3D
20. Prototypical Networks for Few-shot Learning - OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=B1-Hhnslg
21. Understanding Few Shot Learning With Prototypical Networks | by Aditya Mohanty | Medium, accessed July 19, 2025, https://adityaroc.medium.com/understanding-few-shot-learning-with-prototypical-networks-f50525e32ccb
22. N-Shot Learning: Zero vs. Single vs. Two vs. Few (2025), accessed July 19, 2025, https://viso.ai/deep-learning/n-shot-learning/
23. one shot learning - Siamese Network in Keras - Data Science Stack Exchange, accessed July 19, 2025, https://datascience.stackexchange.com/questions/65918/siamese-network-in-keras
24. Difference between Siamese Network and Prototypical Networks for One Shot Learning, accessed July 19, 2025, https://datascience.stackexchange.com/questions/114846/difference-between-siamese-network-and-prototypical-networks-for-one-shot-learni
25. Learn To Learn More Precisely - arXiv, accessed July 19, 2025, https://arxiv.org/html/2408.04590v1
26. Exploring the world of model-agnostic meta-learning and its variants., accessed July 19, 2025, https://interactive-maml.github.io/meta-learning.html
27. CS 330 Autumn 2019/2020 Homework 2 Model-Agnostic Meta-Learning and Prototypical Networks Due Wednesday October 23, 11:59 PM - Stanford CS330, accessed July 19, 2025, https://cs330.stanford.edu/fall2019/material/CS330_homework2.pdf
28. Beyond Deep Learning: Meta-Learning for Efficient AI | by Teendifferent | Predict - Medium, accessed July 19, 2025, https://medium.com/predict/beyond-deep-learning-meta-learning-with-prototypical-networks-for-efficient-ai-56503b642c06
29. Learning to Compare: Relation Network for Few-Shot Learning - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content_cvpr_2018/papers_backup/Sung_Learning_to_Compare_CVPR_2018_paper.pdf
30. Learning to focus: cascaded feature matching network for few-shot image recognition, accessed July 19, 2025, http://scis.scichina.com/en/2021/192105.pdf
31. Cluster analysis - Wikipedia, accessed July 19, 2025, https://en.wikipedia.org/wiki/Cluster_analysis
32. Deep learning-based clustering approaches for bioinformatics - Oxford Academic, accessed July 19, 2025, https://academic.oup.com/bib/article/22/1/393/5721075
33. Gaussian prototypical networks in meta-learning [Tutorial] - Packt, accessed July 19, 2025, https://www.packtpub.com/en-us/learning/how-to-tutorials/gaussian-prototypical-networks-in-meta-learning-tutorial
34. Gaussian Prototypical Networks for Few-Shot Learning on Omniglot, accessed July 19, 2025, http://arxiv.org/pdf/1708.02735
35. GAUSSIAN PROTOTYPICAL NETWORKS FOR FEW- SHOT LEARNING ON OMNIGLOT - OpenReview, accessed July 19, 2025, https://openreview.net/pdf?id=HydnA1WCb
36. SEN: A Novel Feature Normalization Dissimilarity Measure for ..., accessed July 19, 2025, https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123680120.pdf
37. ProtoDiff: Learning to Learn Prototypical Networks by Task-Guided Diffusion | OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=lp9GR2t3hn
38. [2208.09345] IPNET:Influential Prototypical Networks for Few Shot Learning - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2208.09345
39. Graph Prototypical Networks for Few-shot Learning on Attributed Networks - Emory Computer Science, accessed July 19, 2025, https://www.cs.emory.edu/~kshu5/files/few_shot_graph.pdf
40. Prototypical Network - Awesome-META+, accessed July 19, 2025, [https://wangjingyao07.github.io/Awesome-Meta-Learning-Platform/2%20Documentation/4%20proto_tutorial/](https://wangjingyao07.github.io/Awesome-Meta-Learning-Platform/2 Documentation/4 proto_tutorial/)
41. Few-shot learning for skin lesion classification: A prototypical ..., accessed July 19, 2025, https://elib.dlr.de/207691/1/1-s2.0-S2352914824000765-main.pdf
42. Support-Query Prototype Fusion Network for Few-shot Medical Image Segmentation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2405.07516v1
43. (PDF) Interpretable Medical Image Classification Using Prototype Learning and Privileged Information - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/374341136_Interpretable_Medical_Image_Classification_Using_Prototype_Learning_and_Privileged_Information
44. Attribute-guided prototype network for few-shot molecular property ..., accessed July 19, 2025, https://academic.oup.com/bib/article/25/5/bbae394/7731658
45. Few-Shot Learning for Low-Data Drug Discovery | Journal of Chemical Information and Modeling - ACS Publications, accessed July 19, 2025, https://pubs.acs.org/doi/abs/10.1021/acs.jcim.2c00779
46. ProtoBERT-LoRA: Parameter-Efficient Prototypical Finetuning for Immunotherapy Study Identification - arXiv, accessed July 19, 2025, https://arxiv.org/html/2503.20179v1
47. Exploring On-Device Learning Using Few Shots for Audio Classification - Young D. Kwon, accessed July 19, 2025, https://theyoungkwon.github.io/papers/articles/chauhan_on-device-fsl_euspico22.pdf
48. Meta-Learning in Audio and Speech Processing: An End to End Comprehensive Review, accessed July 19, 2025, https://arxiv.org/html/2408.10330v1
49. Combining enhanced DINO with prototypical networks for self-supervised speaker verification - SPIE Digital Library, accessed July 19, 2025, https://www.spiedigitallibrary.org/conference-proceedings-of-spie/13634/1363413/Combining-enhanced-DINO-with-prototypical-networks-for-self-supervised-speaker/10.1117/12.3069505.full?webSyncID=b80d8e8a-beea-ed6a-e05c-117aac5a9199&sessionGUID=362b9eb3-b36c-8a95-604e-3dd821ae3dce
50. [INTERSPEECH 2022 Series #5] Prototypical Speaker-Interference Loss for Target Voice Separation Using Non-Parallel Audio Samples - BLOG | Samsung Research, accessed July 19, 2025, https://research.samsung.com/blog/INTERSPEECH-2022-Series-5-Prototypical-Speaker-Interference-Loss-for-Target-Voice-Separation-Using-Non-Parallel-Audio-Samples
51. DGPN: A Dual Graph Prototypical Network for Few-Shot Speech Spoofing Algorithm Recognition - ISCA Archive, accessed July 19, 2025, https://www.isca-archive.org/interspeech_2024/ge24_interspeech.pdf
52. A Multimodal Prototypical Approach for Unsupervised Sound ..., accessed July 19, 2025, https://arxiv.org/pdf/2306.12300
53. Your Own Few-Shot Classification Model Ready in 15mn with PyTorch - Theodo Data & AI, accessed July 19, 2025, https://data-ai.theodo.com/en/technical-blog/your-few-shot-model-15mn-pytorch
54. Reviews: Prototypical Networks for Few-shot Learning - NIPS, accessed July 19, 2025, https://proceedings.neurips.cc/paper/2017/file/cb8da6767461f2812ae4290eac7cbc42-Reviews.html
55. Revisiting Prototypical Network for Cross Domain Few-Shot Learning - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2023/papers/Zhou_Revisiting_Prototypical_Network_for_Cross_Domain_Few-Shot_Learning_CVPR_2023_paper.pdf
56. Towards Few-Shot Learning in the Open World: A Review and Beyond - arXiv, accessed July 19, 2025, https://arxiv.org/html/2408.09722v1
57. A Survey on Few-Shot Class-Incremental Learning - ODU Digital Commons, accessed July 19, 2025, https://digitalcommons.odu.edu/context/computerscience_fac_pubs/article/1309/viewcontent/Li_2024_ASurveyonFewShotClassIncrementalOCR.pdf
58. On Episodes, Prototypical Networks, and Few-Shot Learning - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2012.09831
59. arXiv:2412.11375v1 [cs.CV] 16 Dec 2024, accessed July 19, 2025, [https://arxiv.org/pdf/2412.11375?](https://arxiv.org/pdf/2412.11375)
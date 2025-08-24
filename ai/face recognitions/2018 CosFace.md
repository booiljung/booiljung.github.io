[얼굴인식 문제](./index.md)
# CosFace (각도 공간에서의 대규모 마진을 통한 심층 얼굴 인식 특징 학습, CVPR 2018)



얼굴 인식(Face Recognition) 기술은 컴퓨터 비전 분야의 오랜 연구 주제 중 하나로, 심층 합성곱 신경망(Deep Convolutional Neural Networks, CNNs)의 등장과 함께 비약적인 발전을 이루었다.1 현대의 얼굴 인식 시스템은 크게 두 가지 핵심 과제로 나뉜다: 주어진 두 얼굴 이미지가 동일 인물인지 판별하는 얼굴 검증(Face Verification)과, 데이터베이스 내에서 주어진 얼굴 이미지와 동일한 인물을 찾아내는 얼굴 식별(Face Identification)이다.1

이러한 과제를 해결하기 위한 딥러닝 기반 파이프라인은 통상적으로 세 단계로 구성된다. 첫째, 이미지 내에서 얼굴 영역을 찾아내는 얼굴 탐지(Face Detection). 둘째, 탐지된 얼굴 영역으로부터 신원 구분에 핵심적인 정보를 담은 고차원 벡터, 즉 특징(feature)을 추출하는 특징 추출(Feature Extraction). 셋째, 추출된 특징을 기반으로 신원을 분류하거나 유사도를 측정하는 분류(Classification) 또는 매칭(Matching) 단계이다.1 이 과정에서 CNN은 복잡한 얼굴 이미지로부터 조명, 자세, 표정 등의 변화에 강인하면서도 개인의 고유한 정체성을 담고 있는 고수준의 특징을 학습하는 중추적인 역할을 담당한다.


딥러닝 초기, 얼굴 인식 문제는 수천에서 수만 개에 이르는 방대한 수의 신원을 각각의 클래스로 간주하는 다중 클래스 분류(multi-class classification) 문제로 접근되었다. 이러한 접근법에서 모델의 마지막 단에는 자연스럽게 Softmax 활성화 함수와 교차 엔트로피(Cross-Entropy) 손실을 결합한 Softmax 손실 함수가 사용되었다.5 Softmax 손실 함수의 수학적 정의는 다음과 같다.

$$
L_{softmax} = - \frac{1}{N} \sum_{i=1}^{N} \log \frac{e^{W_{y_i}^T x_i + b_{y_i}}}{\sum_{j=1}^{C} e^{W_j^T x_i + b_j}}
$$
여기서 $N$은 학습 샘플의 수, $C$는 클래스(신원)의 수, $x_i$는 $i$번째 샘플의 특징 벡터, $y_i$는 해당 샘플의 정답 레이블, $W_j$와 $b_j$는 각각 $j$번째 클래스에 해당하는 마지막 완전 연결 계층의 가중치 벡터와 편향을 의미한다.

Softmax 손실 함수는 학습 데이터셋에 포함된 클래스들을 서로 '분리(separable)'하는 결정 경계를 학습하는 데에는 효과적이다. 즉, 학습된 특징 공간에서 각 클래스의 샘플들이 다른 클래스의 샘플들과 구분될 수 있도록 만든다. 그러나 Softmax 손실은 본질적으로 특징의 '변별력(discriminative power)'을 명시적으로 강화하도록 설계되지 않았다.2 이로 인해 학습된 특징들은 결정 경계 주변에 밀집하여 분포하는 경향을 보이며, 클래스 간의 구분이 명확하지 않고 모호해질 수 있다.

이러한 한계는 특히 '개방형 집합(open-set)' 얼굴 인식 시나리오에서 심각한 성능 저하의 원인이 된다.9 개방형 집합 문제란, 모델 학습에 사용되지 않은 새로운 신원을 인식해야 하는 실제 응용 환경을 의미한다. Softmax로 학습된 모델은 클래스 간 경계가 명확하지 않기 때문에, 새로운 신원의 특징이 기존 클래스의 영역과 쉽게 겹치거나 혼동될 수 있다. 따라서 단순히 클래스를 분리하는 것을 넘어, 각 클래스의 특징을 최대한 응집시키고 클래스 간의 거리를 최대로 벌리는 것이 필수적이다.


이상적인 얼굴 인식 모델이 학습해야 하는 특징은 두 가지 핵심적인 속성을 만족해야 한다. 첫째, 동일 인물로부터 추출된 특징들(클래스 내, intra-class)은 특징 공간 상에서 서로 매우 가깝게 모여 있어야 한다. 이를 '클래스 내 분산 최소화(minimizing intra-class variance)' 또는 '클래스 내 간결성(compactness)'이라 한다. 둘째, 서로 다른 인물로부터 추출된 특징들(클래스 간, inter-class)은 가능한 한 멀리 떨어져 있어야 한다. 이를 '클래스 간 분산 최대화(maximizing inter-class variance)' 또는 '클래스 간 분리성(separability)'이라 한다.1

이 목표를 달성하기 위해 Softmax 손실을 개선하려는 다양한 연구가 진행되었다. Center Loss는 각 클래스의 특징 중심(center)을 학습하고, 특징 벡터와 해당 클래스 중심 간의 유클리드 거리를 페널티로 부여하여 클래스 내 분산을 직접적으로 최소화한다.11 Triplet Loss는 기준 샘플(anchor), 동일 클래스 샘플(positive), 다른 클래스 샘플(negative)로 구성된 세 쌍(triplet)을 이용하여, 기준-긍정 샘플 간의 거리가 기준-부정 샘플 간의 거리보다 특정 '마진(margin)' 이상 작아지도록 학습한다.1 L-Softmax는 Softmax 손실에 각도 마진 개념을 도입하여 결정 경계를 강화한다.12 이들 방법론은 서로 다른 형태를 띠고 있지만, 공통적으로 특징 공간 내에 명시적인 '마진'을 도입하여 클래스 간의 거리를 확보하고 클래스 내의 응집도를 높이려는 철학을 공유한다.1

이러한 흐름 속에서 얼굴 인식 문제의 본질을 재고할 필요가 있다. 초기 접근법은 얼굴 인식을 수만 개의 클래스를 가진 '분류' 문제로 정의했지만 7, 실제 응용 환경에서는 학습된 클래스를 정확히 맞추는 것보다, 두 얼굴 특징 벡터 간의 '유사도' 또는 '거리'를 정밀하게 측정하는 것이 훨씬 중요하다.2 Softmax 손실은 이 '거리'를 직접적으로 최적화하지 않고, 단지 로짓(logit) 값을 기반으로 확률적 분류만을 수행한다.5 따라서 Softmax의 한계는 단순히 성능이 부족하다는 기술적 문제를 넘어, 문제의 본질(거리 학습)과 해결 방식(분류) 간의 근본적인 불일치에서 기인한다. CosFace와 같은 후속 마진 기반 손실 함수들은 바로 이 간극을 메우려는 시도이다. 즉, Triplet Loss와 같은 순수 거리 학습(Metric Learning) 방식이 가지는 조합 폭발(combinatorial explosion) 문제나 어려운 샘플 마이닝(hard sample mining)의 복잡성을 회피하면서도 5, 분류 기반 프레임워크 내에서 거리 학습의 핵심 목표를 효율적으로 달성하려는 실용적인 절충안으로 볼 수 있다.



전통적인 거리 학습 기반 손실 함수인 Center Loss나 Triplet Loss는 특징 벡터 간의 유클리드 거리(Euclidean distance)에 마진을 부과하는 방식으로 변별력을 확보하고자 했다.1 그러나 CosFace는 문제 해결의 관점을 근본적으로 전환하여, 특징 벡터 간의 '각도(angle)' 또는 '코사인 유사도(cosine similarity)'에 직접적으로 마진을 적용하는 방식을 제안했다.2

이러한 접근 방식의 전환은 매우 합리적인 근거를 가진다. 얼굴 검증과 같은 실제 얼굴 인식 응용에서는 두 얼굴 이미지의 유사도를 판단하기 위해, 추출된 특징 벡터 간의 코사인 유사도를 계산하는 방법이 표준적으로 사용되기 때문이다.2 따라서 학습 단계에서 최적화하는 목표(코사인 유사도 기반 손실)와 추론 단계에서 사용하는 평가 지표(코사인 유사도)를 일치시킴으로써, 모델의 성능을 보다 직접적이고 효과적으로 극대화할 수 있다. 이는 학습과 추론 간의 불일치를 해소하고, 모델이 평가 지표에 직접적으로 최적화된 특징 표현을 학습하도록 유도하는 중요한 전략이다.


CosFace의 핵심은 새롭게 제안된 손실 함수인 대규모 마진 코사인 손실(Large Margin Cosine Loss, LMCL)에 있다.1 LMCL의 기본 철학은 두 가지 핵심적인 아이디어로 요약될 수 있다.

1. **정규화를 통한 차원 축소 및 문제 단순화:** 특징 벡터와 마지막 완전 연결 계층의 가중치 벡터를 모두 L2 정규화(L2 Normalization)한다. 이 과정을 통해 모든 특징 벡터는 고정된 반지름을 가진 초구(hypersphere) 표면에 투영된다.2
2. **코사인 공간에서의 마진 극대화:** 정규화된 초구 위에서, 서로 다른 클래스에 속한 특징들 간의 결정 경계를 코사인 공간 상에서 최대한 밀어내어 명확한 마진을 확보한다.1

이 두 가지 전략을 통해, 특징 벡터의 크기(magnitude)에 의해 발생하는 반경 방향의 변화(radial variation)를 효과적으로 제거하고, 모델이 오직 특징 벡터 간의 '각도' 정보에만 집중하도록 강제한다.1 결과적으로 모델은 더욱 강력한 각도 변별력을 가진 특징 표현을 학습하게 된다.

이 과정에서 '정규화'는 단순한 전처리 단계를 넘어, 문제 자체의 위상(topology)을 변형시키는 핵심적인 전략으로 작용한다. 정규화 이전의 특징 공간에서 두 벡터의 내적 $W^T x$는 벡터의 크기($||W||$, $||x||$)와 두 벡터 사이의 각도($\cos(\theta)$)라는 두 가지 변수에 의해 결정된다. 이는 최적화 목표를 복잡하게 만드는 요인이다. CosFace는 특징 벡터와 가중치 벡터를 모두 L2 정규화하여 각각의 크기를 $s$와 1로 고정한다.4 그 결과, 내적 값은 

$s \cdot \cos(\theta)$가 되어, 최적화의 대상이 되는 변수를 오직 '각도 $\theta$' 하나로 통일시킨다. 이는 복잡한 유클리드 공간에서의 분리 문제를, 기하학적으로 훨씬 다루기 용이한 '단위 초구 위에서의 각도 분리' 문제로 변환하는 효과를 가진다. 이처럼 문제가 단순화되고 명확하게 재정의되었기 때문에, 마진 $m$의 역할이 분명해지고 모델은 다른 교란 요인 없이 오직 각도 변별력을 학습하는 데에만 집중할 수 있게 된다.


CosFace 프레임워크는 학습과 추론 단계에서 각각 다음과 같은 역할을 수행한다.1

- **학습 단계 (Training Phase):** 입력된 얼굴 이미지는 CNN 백본 네트워크를 통과하여 고차원의 특징 벡터 $x$로 임베딩된다. 이 특징 벡터는 LMCL 손실 함수에 의해 최적화된다. LMCL은 동일 클래스에 속한 특징들의 코사인 유사도는 높이고(클래스 내 분산 최소화), 다른 클래스에 속한 특징들과의 코사인 유사도는 특정 마진 $m$을 두고 낮추도록(클래스 간 마진 최대화) 모델의 가중치를 업데이트한다.1
- **추론 단계 (Testing/Inference Phase):** 성공적으로 학습된 CosFace 모델(CNN)은 특징 추출기로 사용된다. 테스트용 얼굴 이미지가 모델에 입력되면, 학습을 통해 변별력을 갖추게 된 특징 벡터가 출력된다. 두 얼굴 이미지의 동일인 여부를 판별하기 위해서는, 각각의 이미지에서 추출된 두 특징 벡터 간의 코사인 유사도를 계산한다. 이 유사도 값이 사전에 설정된 임계값(threshold)보다 높으면 동일 인물로, 낮으면 다른 인물로 판단한다.2



LMCL을 이해하기 위해서는 먼저 전통적인 Softmax 손실 함수가 어떻게 코사인 유사도 기반의 손실 함수로 변형되는지 살펴보아야 한다. 이 변형 과정은 LMCL의 이론적 토대를 이룬다.

1. **편향 제거 및 내적 분해:** 전통적인 Softmax의 로짓(logit)은 $W_j^T x_i + b_j$로 계산된다. 얼굴 인식에서는 클래스 수가 매우 많아 편향 $b_j$의 역할이 미미하므로, 이를 0으로 설정하여 단순화한다. 그러면 로짓은 가중치 벡터 $W_j$와 특징 벡터 $x_i$의 내적 $W_j^T x_i$가 된다. 이 내적은 두 벡터의 크기와 사이각의 코사인 값의 곱, 즉 $||W_j|| \cdot ||x_i|| \cdot \cos(\theta_j)$로 분해할 수 있다.5

2. **정규화 적용:** 다음으로 가중치 벡터 $W_j$에 L2 정규화를 적용하여 크기를 1로 고정한다 ($||W_j||=1$). 또한, 특징 벡터 $x_i$의 크기 $||x_i||$는 고정된 스케일 하이퍼파라미터 $s$로 대체한다. 이러한 정규화를 통해 로짓은 최종적으로 $s \cdot \cos(\theta_j)$라는 매우 간결한 형태로 표현된다.

3. **정규화된 Softmax 손실 (NSL):** 이 정규화된 로짓을 원래의 Softmax 손실 함수에 대입하면, 정규화된 Softmax 손실(Normalized Softmax Loss, NSL) 또는 수정된 Softmax 손실(Modified Softmax Loss)을 얻을 수 있다.5

   $$
   L_{NSL} = - \frac{1}{N} \sum_{i=1}^{N} \log \frac{e^{s \cdot \cos(\theta_{y_i})}}{e^{s \cdot \cos(\theta_{y_i})} + \sum_{j \neq y_i} e^{s \cdot \cos(\theta_j)}}
   $$

이 NSL 형태에서 두 클래스(C1, C2) 간의 결정 경계는 $s \cdot \cos(\theta_1) = s \cdot \cos(\theta_2)$, 즉 $\cos(\theta_1) = \cos(\theta_2)$가 된다. 이는 클래스 분류가 더 이상 특징 벡터나 가중치 벡터의 크기에 영향을 받지 않고, 오직 두 벡터 사이의 각도($\theta$)에 의해서만 결정됨을 의미한다. 이로써 특징 공간에서의 변별력 문제를 순수한 각도 공간의 문제로 전환할 수 있게 되었다. 하지만 NSL 자체는 클래스를 분리할 뿐, 클래스 간에 어떠한 마진도 부여하지 않기 때문에 여전히 노이즈에 취약하고 변별력이 부족한 한계를 가진다.13


LMCL에서 L2 정규화는 단순히 수식을 단순화하는 것을 넘어, 학습 과정을 근본적으로 변화시키는 핵심적인 역할을 수행한다.

- **가중치 벡터($W$) 정규화:** 각 클래스의 가중치 벡터 $W_j$를 정규화하여 $||W_j||=1$로 만드는 것은, 각 클래스를 대표하는 프로토타입(또는 중심)을 단위 초구(unit hypersphere) 상에 위치시키는 것과 같다. 이로 인해 클래스 간의 분산은 유클리드 공간 상의 거리가 아닌, 단위 초구 상의 각도 차이로만 결정되도록 강제된다. 이는 모델이 모든 클래스를 동등한 중요도로 다루도록 유도하며, 특정 클래스의 가중치 크기가 비정상적으로 커져 학습을 지배하는 현상을 방지한다.7
- **특징 벡터($x$) 정규화:** 특징 벡터 $x$의 크기를 고정된 스케일 $s$로 조정($||x||=s$)하는 것은, 모델이 학습 과정에서 취할 수 있는 '지름길'을 차단하는 효과를 가진다. 만약 특징 벡터의 크기가 가변적이라면, 모델은 어려운 샘플의 각도를 줄여 변별력을 높이는 대신, 쉬운 샘플의 특징 벡터 크기($||x||$)를 크게 만들어 손실 값을 쉽게 낮추려는 경향을 보일 수 있다. 특징 벡터의 크기를 $s$로 고정함으로써, 모델은 손실을 줄이기 위해 오직 코사인 값을 최적화하는 방향, 즉 정답 클래스와의 각도를 줄이고 오답 클래스와의 각도를 늘리는 방향으로만 학습을 진행하도록 강제된다.13


NSL이 가진 변별력 부족 문제를 해결하기 위해, CosFace는 '덧셈적 코사인 마진(additive cosine margin)'이라는 직관적이고 강력한 개념을 도입한다. 이는 정답 클래스($y_i$)에 해당하는 코사인 값에서 양수의 고정된 마진 값 $m$ ($m \ge 0$)을 빼주는 방식으로 구현된다.

2개의 클래스(C1, C2)가 존재하는 간단한 경우를 가정해 보자. NSL에서는 어떤 샘플이 C1으로 올바르게 분류되기 위한 조건이 $\cos(\theta_1) > \cos(\theta_2)$였다. CosFace에서는 이 조건이 $\cos(\theta_1) - m > \cos(\theta_2)$로 한층 더 엄격하게 강화된다.1

이로 인해 새로운 결정 경계는 $\cos(\theta_1) - m = \cos(\theta_2)$가 된다. 이는 기하학적으로 기존의 결정 경계($\cos(\theta_1) = \cos(\theta_2)$)를 C1의 중심 방향으로 $m$만큼 밀어내는 효과를 가진다. 결과적으로, C1에 속한 샘플들은 C1의 중심에 더 가깝게(즉, 더 큰 코사인 값을 갖도록) 분포해야만 올바르게 분류될 수 있다. 이 과정은 모든 클래스에 대해 동일하게 적용되어, 클래스 내 특징들의 응집도를 높이고 클래스 간의 명확한 분리 공간(마진)을 형성하게 된다.


스케일 파라미터 $s$는 정규화된 특징 벡터의 반지름(radius) 크기를 제어하는 동시에, Softmax 손실 함수의 최적화 과정에 중요한 영향을 미치는 하이퍼파라미터이다.13

- **기하학적 의미:** $s$는 모든 특징 벡터가 놓이게 될 초구의 반지름을 결정한다.
- **최적화 역할:** $s$는 로짓($s \cdot \cos(\theta)$)의 크기를 조절함으로써 Softmax 함수의 입력 분포를 제어한다. $s$ 값이 클수록 로짓 값의 범위가 넓어지고, 이는 Softmax 함수의 출력 확률 분포를 더 뾰족하게(peaked) 만든다. 즉, 정답 클래스에 대한 확률은 1에 가깝게, 오답 클래스에 대한 확률은 0에 가깝게 만든다. 이는 모델이 예측에 대해 더 강한 페널티를 받게 하여, 학습 초기에 수렴을 촉진하고 더 변별력 있는 특징을 학습하도록 유도하는 효과가 있다.
- **값 설정:** $s$는 학습 과정에서 함께 학습될 수도 있지만, CosFace 논문에서는 하이퍼파라미터로 고정하여 사용한다. 이 값이 너무 크면 기울기(gradient)가 폭발하여 최적화가 불안정해질 수 있고, 너무 작으면 기울기가 소실되어 수렴이 느려질 수 있다. 따라서 데이터셋과 모델 아키텍처에 따라 적절한 값을 설정하는 것이 중요하다.14


앞서 설명한 구성 요소들—코사인 손실로의 재구성, L2 정규화, 덧셈적 코사인 마진 $m$, 그리고 스케일 파라미터 $s$—을 모두 통합하면 최종적인 LMCL 손실 함수가 완성된다. $i$번째 학습 샘플 $x_i$에 대한 LMCL 손실은 다음과 같이 정의된다.4

$$
L_{LMCL} = - \frac{1}{N} \sum_{i=1}^{N} \log \frac{e^{s \cdot (\cos(\theta_{y_i}) - m)}}{e^{s \cdot (\cos(\theta_{y_i}) - m)} + \sum_{j \neq y_i} e^{s \cdot \cos(\theta_j)}}
$$
이 수식에서 $\cos(\theta_j)$는 정규화된 가중치 $W_j$와 정규화된 특징 $x_i$의 내적으로 계산된다. 실제 딥러닝 프레임워크에서 이를 구현할 때는, 정답 클래스 $y_i$에만 마진을 적용하기 위해 다음과 같이 조건부 함수 $\psi(\theta_j)$를 정의하여 사용하는 것이 더 편리하다.

$$
\psi(\theta_j) = \begin{cases} \cos(\theta_j) - m, & \text{if } j = y_i \\ \cos(\theta_j), & \text{if } j \neq y_i \end{cases}
$$
이를 이용하면 LMCL 손실 함수를 다음과 같이 표현할 수 있다.

$$
L_{LMCL} = - \frac{1}{N} \sum_{i=1}^{N} \log \frac{e^{s \cdot \psi(\theta_{y_i})}}{\sum_{j=1}^{C} e^{s \cdot \psi(\theta_j)}}
$$
이론적으로는 $\cos(\theta_{y_i}) - m$이 다른 모든 클래스의 $\cos(\theta_j)$보다 커야 한다는 제약 조건이 필요하지만, 실제 학습 과정에서는 이 조건을 명시적으로 강제하기보다는, 위에서 정의된 손실 함수 자체를 경사 하강법을 통해 최소화하는 방식으로 최적화가 진행된다. 이 과정을 통해 모델은 자연스럽게 마진 제약 조건을 만족시키는 방향으로 학습된다.



CosFace의 등장을 전후하여 제안된 여러 마진 기반 손실 함수들은 변별력을 강화하기 위해 '마진 패널티'를 부여하는 방식에서 미묘하지만 중요한 차이를 보인다. 이러한 차이는 크게 두 가지 기준으로 분류할 수 있다.

- **마진 적용 공간 (Space):** 마진을 각도($\theta$)에 직접 적용하는가, 아니면 각도의 코사인 값($\cos(\theta)$)에 적용하는가에 따른 구분이다. SphereFace와 ArcFace는 전자에 속하며, CosFace는 후자에 속한다.
- **마진 적용 방식 (Method):** 마진을 기존 값에 곱하는 '곱셈적(multiplicative)' 방식인가, 아니면 더하거나 빼는 '덧셈적(additive)' 방식인가에 따른 구분이다. SphereFace는 곱셈적 방식을, CosFace와 ArcFace는 덧셈적 방식을 채택한다.

이러한 분류 기준은 각 손실 함수의 기하학적 특성과 최적화 안정성을 결정하는 핵심적인 요인이 된다.


SphereFace, 또는 A-Softmax(Angular Softmax) Loss는 각도 공간에 직접적으로 곱셈적 마진을 도입한 선구적인 연구이다.12 SphereFace의 target logit은 $s \cdot \cos(m\theta_{y_i})$로 정의되며, 여기서 $m$은 1 이상의 정수이다.15 이는 정답 클래스와의 각도 $\theta_{y_i}$를 $m$배로 증폭시켜, 더 작은 각도를 요구하는 효과를 가진다.

- **기하학적 해석:** 2클래스 분류 문제에서 SphereFace의 결정 경계는 $\cos(m\theta_1) = \cos(\theta_2)$가 된다. 이는 각도 공간에서 명확한 마진을 형성하여 클래스 간 분리도를 높인다.15

- **본질적 한계:** SphereFace는 혁신적인 아이디어에도 불구하고 몇 가지 심각한 한계를 내포하고 있다.

  1. **최적화 불안정성:** 코사인 함수는 주기가 있는 비단조(non-monotonic) 함수이다. 따라서 $m>1$일 때, $\cos(m\theta)$ 함수는 $\theta$가 증가함에 따라 단조롭게 감소하지 않는다. 이로 인해 손실 함수의 표면이 복잡해져 최적화 과정이 매우 불안정해지고, 학습 초기에 모델이 발산하기 쉽다. 이 문제를 해결하기 위해 SphereFace는 학습 초기에 일반 Softmax 손실을 함께 사용하거나, 학습이 진행됨에 따라 $m$의 효과를 점진적으로 증가시키는 어닐링(annealing)과 같은 추가적인 기법을 필요로 한다.9
  2. **일관성 없는 마진:** 마진의 크기가 각도 $\theta$ 값에 따라 변동한다. 두 클래스가 시각적으로 매우 유사하여 $\theta$가 작은 경우, 마진의 크기도 덩달아 작아진다. 심지어 $\theta$가 0에 가까워지면 마진이 거의 사라지는 문제가 발생한다.4 이는 정작 변별이 더 어려운, 유사한 클래스 쌍에 대해서는 더 작은 마진을 부여하는 역효과를 낳을 수 있다.4

- **CosFace의 상대적 우위:** CosFace는 마진을 코사인 공간에서 $\cos(\theta) - m$으로 정의함으로써 이러한 문제들을 해결했다. $\cos(\theta) - m$은 $\theta \in [0, \pi]$ 구간에서 $\theta$에 대해 단조롭게 감소하는(monotonically decreasing) 함수이다. 이는 손실 함수의 최적화 과정을 훨씬 안정적으로 만들며, 별도의 복잡한 기법 없이도 모델이 쉽게 수렴하도록 한다.7 또한, 코사인 공간 상에서 모든 $\theta$ 값에 대해 일정한 크기의 마진을 부여하므로, 마진의 일관성 문제에서도 자유롭다.


ArcFace, 또는 Additive Angular Margin Loss는 CosFace 이후에 제안되었으며, 각도 공간에 직접 덧셈적 마진을 적용하는 방식을 채택했다.9 ArcFace의 target logit은 $s \cdot \cos(\theta_{y_i} + m)$으로 정의된다.9 이는 정답 클래스와의 각도 $\theta_{y_i}$에 직접 각도 마진 $m$을 더하여, 더 작은 원래 각도를 요구하는 방식이다.

- **기하학적 해석:** ArcFace의 가장 큰 장점은 명확하고 직관적인 기하학적 해석에 있다. 특징 벡터들이 단위 초구 상에 분포할 때, 두 특징 벡터 사이의 각도는 초구 표면을 따르는 최단 거리, 즉 측지선 거리(Geodesic Distance)에 비례한다. ArcFace의 마진 $m$은 이 측지선 거리에 직접적으로 대응된다.9 따라서 ArcFace는 초구라는 기하학적 공간의 특성을 가장 잘 반영하는 손실 함수로 평가받는다. 또한, 결정 경계가 각도 공간에서 일정하게 유지되어 모든 클래스에 대해 일관된 마진을 제공한다.9
- **CosFace와의 관계 및 비교:** CosFace와 ArcFace는 수치적으로 매우 유사한 효과를 내며, 실제 성능에서도 대등한 결과를 보인다.9 하지만 두 손실 함수는 결정 경계의 형태에서 미묘한 차이를 보인다. ArcFace는 각도 공간에서 선형적인(linear) 마진을 생성하는 반면, CosFace는 코사인 공간에서는 선형적이지만 이를 각도 공간으로 변환하면 비선형적인(non-linear) 마진을 생성한다.9 이론적으로는 ArcFace의 기하학적 해석이 더 명확하고 우아하며, 학습 과정이 조금 더 안정적이라고 알려져 있다.9 그럼에도 불구하고 CosFace는 구현의 단순성과 강력한 성능 덕분에 여전히 널리 사용되는 표준적인 방법론 중 하나이다.


각 손실 함수가 특징 공간에서 형성하는 결정 경계의 차이를 시각적으로 이해하는 것은 매우 중요하다. 아래 표는 2차원 특징 공간을 가정했을 때 각 손실 함수의 핵심 수식과 특징을 요약하여 비교한 것이다. 이 비교를 통해 Softmax에서 시작하여 SphereFace, CosFace, ArcFace로 이어지는 마진 기반 손실 함수의 발전 과정을 명확하게 파악할 수 있다.

| 손실 함수                  | 핵심 수식 (Target Logit)           | 마진 유형          | 기하학적 해석                                                | 주요 특징                                                 |
| -------------------------- | ---------------------------------- | ------------------ | ------------------------------------------------------------ | --------------------------------------------------------- |
| **Softmax (Normalized)**   | $s \cdot \cos(\theta_{y_i})$       | 없음               | -                                                            | 분리는 가능하나 변별력 부족 5                             |
| **SphereFace (A-Softmax)** | $s \cdot \cos(m\theta_{y_i})$      | 곱셈적 각도 마진   | 각도 공간에서 마진 생성. 마진 크기가 $\theta$에 의존적.      | $m$이 정수여야 하고, 최적화가 불안정하여 추가 기법 필요 4 |
| **CosFace (LMCL)**         | $s \cdot (\cos(\theta_{y_i}) - m)$ | 덧셈적 코사인 마진 | 코사인 공간에서 일정한 마진 생성.                            | 구현이 간단하고 직접적이며 안정적인 최적화 가능 1         |
| **ArcFace**                | $s \cdot \cos(\theta_{y_i} + m)$   | 덧셈적 각도 마진   | 초구 위 측지선 거리(Geodesic Distance)에 직접 대응. 각도 공간에서 일정한 마진. | 기하학적 해석이 가장 명확하며, 안정적인 학습 성능 9       |

이 표는 각 손실 함수가 어떤 철학적, 수학적 배경 위에서 설계되었으며, 어떤 장단점을 가지는지를 압축적으로 보여준다. 특히 CosFace는 SphereFace의 최적화 문제를 해결하면서도 ArcFace에 필적하는 강력한 성능을 보이는, 실용성과 이론적 타당성 사이의 균형을 잘 맞춘 접근법임을 확인할 수 있다.



CosFace의 성능은 당시 얼굴 인식 분야에서 표준적으로 사용되던 여러 공개 벤치마크 데이터셋을 통해 검증되었다. 주요 데이터셋은 다음과 같다.

- **LFW (Labeled Faces in the Wild):** 제약 없는 실제 환경(in the wild)에서 촬영된 13,000개 이상의 얼굴 이미지로 구성된 데이터셋이다. 동일 인물과 다른 인물 쌍으로 구성된 6,000개의 테스트 쌍에 대한 얼굴 검증(verification) 정확도를 측정하는 것이 표준적인 평가 프로토콜이다.3
- **YTF (YouTube Faces):** YouTube에서 수집한 1,595명의 인물에 대한 약 3,425개의 비디오 클립으로 구성된 데이터셋이다. LFW와 유사하게 비디오 프레임 쌍에 대한 얼굴 검증 성능을 평가하는 데 사용된다.3
- **MegaFace Challenge:** 대규모 얼굴 인식 성능 평가를 위해 구축된 벤치마크이다. 1백만 개에 달하는 방해물(distractor) 이미지 사이에서 특정 인물을 식별(identification)하거나, 대규모 방해물 존재 하에 검증(verification) 성능을 측정한다. 이는 실제 보안 시스템과 같이 수많은 비대상 인물 중에서 대상 인물을 정확히 찾아내야 하는 현실적인 시나리오를 모사한다.3


CosFace는 제안 당시 위에서 언급된 주요 벤치마크들에서 최고 수준(State-of-the-art, SOTA)의 성능을 달성하며 그 효과성을 입증했다.1

- LFW와 YTF 데이터셋에서 CosFace는 기존의 Softmax 기반 방법론은 물론, Center Loss, SphereFace 등 당시의 경쟁적인 마진 기반 손실 함수들보다 일관되게 높은 검증 정확도를 기록했다.1 이는 LMCL이 학습한 특징이 더 나은 일반화 성능과 변별력을 가짐을 시사한다.
- 특히, 대규모 방해물이 존재하는 MegaFace 챌린지에서 CosFace는 압도적인 성능을 보였다. MegaFace는 클래스 간의 미세한 차이를 구분해내는 능력을 극한까지 시험하는 벤치마크로, 이곳에서의 높은 성능은 LMCL이 클래스 간 분산을 효과적으로 최대화했음을 강력하게 뒷받침한다.
- 이후에 발표된 ArcFace 논문에서 진행된 공정한 비교 실험에서도 CosFace의 우수성은 재확인되었다. 해당 실험에서 CosFace는 SphereFace보다 뛰어난 성능을 보였으며, ArcFace와는 대등하거나 아주 근소한 차이의 성능을 기록했다.9 이는 CosFace가 후속 연구들이 등장한 이후에도 여전히 매우 경쟁력 있고 강력한 베이스라인임을 증명하는 결과이다.

이러한 SOTA 경쟁의 결과를 해석할 때, 한 가지 중요한 점을 고려해야 한다. SphereFace, CosFace, ArcFace 등의 논문들은 각기 다른 실험 환경에서 자신들의 방법론이 최고 성능을 달성했다고 보고한다.3 하지만 각 연구에서 사용한 CNN 백본 아키텍처, 학습 데이터셋의 종류와 규모, 데이터 전처리 및 정제 수준, 그리고 하이퍼파라미터 설정이 모두 다르다. ArcFace 논문에서는 이러한 "네트워크 설정과 데이터 정제의 중요성"을 명시적으로 강조하며, 동일한 손실 함수라도 더 나은 네트워크와 더 깨끗한 데이터를 사용하면 성능이 크게 향상될 수 있음을 보여주었다.9 따라서, 특정 벤치마크에서 A 방법이 B 방법보다 수치적으로 높은 성능을 보였다는 결과가, 순전히 손실 함수 자체의 우월성 때문인지, 아니면 더 나은 실험 환경의 덕분인지 명확히 분리하여 판단하기는 어렵다.22 이러한 관점에서 CosFace의 진정한 가치는 단순히 특정 벤치마크에서의 성능 수치를 넘어, SphereFace의 불안정성을 극복하고 '안정적이며 구현이 용이한 코사인 마진'이라는 강력한 개념적 도구를 제공했다는 점에 있다. 이 개념적 기여는 후속 연구자들이 더 복잡하고 대규모의 실험을 안정적으로 수행할 수 있는 견고한 발판을 마련해주었다는 점에서 더 큰 의의를 가진다.


CosFace 논문에서는 LMCL의 핵심 하이퍼파라미터인 스케일 $s$와 마진 $m$의 값 변화가 모델 성능에 미치는 영향을 분석하기 위한 상세한 Ablation Study를 수행했다.

- **마진 $m$:** 마진 $m$의 크기는 변별력과 수렴 안정성 사이의 트레이드오프 관계를 가진다.
  - $m$ 값이 너무 작으면 마진을 통한 변별력 향상 효과가 미미하여 성능 개선이 제한적이다.
  - 반대로 $m$ 값이 너무 크면, 모델에 가해지는 제약 조건이 지나치게 강해져 학습이 불안정해지고 모델이 수렴하지 못하는 문제가 발생할 수 있다. 또한, 과도한 마진은 학습 데이터의 노이즈에 민감하게 반응하여 일반화 성능을 저해할 수도 있다.13
  - CosFace 논문에서는 실험을 통해 $m=0.35$를 최적의 값으로 제시했다.
- **스케일 $s$:** 스케일 $s$는 학습의 수렴 속도와 안정성에 직접적인 영향을 미친다.
  - 앞서 설명했듯이, $s$는 로짓의 스케일을 조절하여 학습의 난이도를 제어한다. 적절히 큰 $s$ 값은 빠른 수렴을 돕는다.
  - CosFace 논문에서는 $s=64$를 표준 값으로 사용했다.
  - 이러한 하이퍼파라미터들은 서로 독립적이지 않으며, 사용되는 데이터셋의 클래스 수나 모델 아키텍처에 따라 최적의 값이 달라질 수 있다. 따라서 실제 응용에서는 목표하는 태스크에 맞춰 이 값들을 신중하게 튜닝하는 과정이 필요하다.14



CosFace는 얼굴 인식 손실 함수 연구에 큰 획을 그었지만, 그 설계에는 내재적인 한계가 존재한다. 가장 근본적인 한계는 모든 클래스 쌍과 모든 샘플에 대해 '고정된(fixed)' 마진 $m$을 동일하게 적용한다는 점이다.

현실 세계의 얼굴 데이터는 매우 불균일한 특성을 가진다. 어떤 인물 쌍은 외모가 매우 흡사하여 구분하기 어려운 반면(어려운 클래스 쌍), 어떤 인물 쌍은 쉽게 구분이 가능하다(쉬운 클래스 쌍). 또한, 데이터셋 내에서 특정 인물의 이미지는 수천 장에 달하는 반면, 다른 인물의 이미지는 수십 장에 불과한 심각한 데이터 불균형(data imbalance) 문제가 일반적이다.10

이러한 상황에서 고정된 마진을 적용하는 것은 최적이 아닐 수 있다. 구분하기 어려운 클래스 쌍에 대해서는 고정된 마진이 너무 작아 충분한 변별력을 확보하지 못할 수 있고, 반대로 구분하기 쉬운 클래스 쌍에 대해서는 불필요하게 큰 마진이 학습을 방해할 수 있다. 마찬가지로, 샘플 수가 적은 클래스에 강한 마진을 적용하면, 부족한 데이터로 인해 모델이 수렴하는 데 어려움을 겪을 수 있다.10 즉, 고정된 마진은 데이터의 내재적인 난이도와 분포 특성을 고려하지 못하는 '일률적인' 제약 조건이라는 한계를 가진다.


고정된 마진의 한계는 데이터의 품질이 저하되는 상황에서 더욱 두드러진다. 마스크 착용, 선글라스, 심한 위장, 저화질 이미지 등은 얼굴의 핵심적인 식별 정보를 가리거나 왜곡시킨다. 이러한 저품질 이미지에 고품질 이미지와 동일한 강도의 마진을 적용하는 것은 비합리적이다. 정보가 부족한 저품질 이미지에 너무 강한 제약 조건을 가하면, 모델은 존재하지 않는 정보를 억지로 학습하려 하거나 노이즈에 과적합되어 오히려 성능이 저하될 수 있다.24

데이터 불균형 문제 역시 CosFace의 성능에 영향을 미친다. L2 정규화가 데이터 불균형 문제를 어느 정도 완화하는 효과가 있지만 26, 샘플 수가 극단적으로 적은 클래스(under-represented class)는 충분한 학습 기회를 갖지 못한다. 고정된 마진은 이러한 클래스의 학습을 더욱 어렵게 만들 수 있다.


CosFace의 고정된 마진이라는 한계를 극복하기 위해, 후속 연구들은 데이터의 '상황(context)'을 인지하고 그에 맞게 마진을 동적으로 조절하는 '적응형 마진(Adaptive Margin)'이라는 새로운 방향으로 나아갔다.

- **AdaFace:** 특징 벡터의 놈(norm) 크기가 이미지의 품질과 비례한다는 경험적 관찰에 기반한다. AdaFace는 특징 벡터의 놈을 이미지 품질의 대리 지표(proxy)로 사용하여, 놈이 큰 고품질 이미지에는 강한 마진(어려운 샘플 강조)을, 놈이 작은 저품질 이미지에는 약한 마진(쉬운 샘플 강조)을 차등적으로 적용한다.24
- **CurricularFace:** 인간의 학습 방식을 모방한 커리큘럼 학습(Curriculum Learning) 개념을 도입했다. 학습 초기에는 모델이 전체적인 구조를 쉽게 학습할 수 있도록 작은 마진을 적용하고, 학습이 진행됨에 따라 점차 마진의 크기를 키워 구분하기 어려운 샘플들을 집중적으로 학습하도록 유도한다.24
- **KappaFace:** 클래스별 학습 난이도(클래스 내 분산)와 샘플 수(데이터 불균형)를 모두 고려하여, 각 클래스에 최적화된 마진을 적응적으로 할당한다. 이를 통해 학습이 어렵거나 샘플 수가 적은 클래스에 더 적절한 학습 가이드를 제공한다.27
- **GB-CosFace (Global Boundary CosFace):** CosFace를 개방형 집합(open-set) 분류 문제에 더 적합하도록 일반화했다. 두 얼굴 샘플이 동일인에 속하는지를 판단하는 적응형 전역 경계(adaptive global boundary)를 도입하여, 학습 목표와 추론 목표를 일치시켰다.28

이러한 적응형 마진 연구들의 등장은 얼굴 인식 손실 함수 설계의 패러다임이 전환되고 있음을 보여준다. 초기 마진 기반 손실 함수들(SphereFace, CosFace, ArcFace)은 '더 큰 마진이 더 나은 변별력을 낳는다'는 하나의 '보편적 원칙(Universal Principle)'을 각기 다른 방식으로 구현하려는 시도였다. 이 단계의 핵심 질문은 '어떻게 하면 가장 이상적인 형태의 고정 마진을 설계할 것인가?'였다. 그러나 실제 데이터의 복잡성 앞에서 '하나의 마진이 모든 상황에 최적일 수는 없다'는 인식이 확산되었다. 이것이 고정 마진의 근본적인 한계이다. 따라서 후속 연구들은 데이터의 '상황'—이미지의 품질, 클래스의 난이도, 학습의 진행 단계 등—을 인지하고 그에 맞게 마진을 '적응(adaptation)'시키는 방향으로 진화했다. 이는 단순히 손실 함수를 개선하는 것을 넘어, 딥러닝 모델이 데이터의 미묘한 특성을 스스로 이해하고 학습 전략을 동적으로 조절하도록 만드는, 한 단계 더 지능적인 최적화 방식으로의 발전을 의미한다.



CosFace와 그 핵심 아이디어인 LMCL은 심층 얼굴 인식 분야, 특히 손실 함수 연구의 역사에서 중요한 이정표를 세웠다. 그 핵심적인 기여는 다음과 같이 세 가지 측면에서 요약할 수 있다.

- **개념적 기여:** CosFace는 얼굴 특징의 변별력 문제를 복잡한 유클리드 공간이 아닌, 정규화된 초구 위의 각도/코사인 공간에서 해결해야 한다는 명확한 방향을 제시했다. L2 정규화를 통해 특징의 크기 변수를 제거하고 문제를 단순화함으로써, 후속 연구들이 각도 공간에서의 마진 설계에 집중할 수 있는 이론적 토대를 마련했다.
- **실용적 기여:** 선행 연구인 SphereFace가 가진 심각한 최적화 불안정성 문제를 해결하고, '덧셈적 코사인 마진'이라는 매우 간단하면서도 효과적인 해법을 제시했다. CosFace는 구현이 용이하고, 추가적인 트릭 없이도 안정적으로 수렴하며, 강력한 성능을 보인다.1 이러한 실용성은 대규모 데이터셋을 이용한 얼굴 인식 연구의 진입 장벽을 낮추고 발전을 가속화하는 데 크게 기여했다.
- **성능적 기여:** 제안 당시 LFW, YTF, MegaFace 등 다수의 주요 벤치마크에서 최고 수준의 성능을 달성하며, 코사인 공간에 직접 마진을 부과하는 방식의 강력함을 실증적으로 입증했다. 이는 이후 등장한 ArcFace와 함께 마진 기반 손실 함수가 얼굴 인식 분야의 표준으로 자리 잡게 하는 결정적인 계기가 되었다.


CosFace가 제시한 원리와 방법론은 학계와 산업계 모두에 깊은 영향을 미쳤다. 오늘날 CosFace는 ArcFace와 더불어 새로운 얼굴 인식 알고리즘의 성능을 평가할 때 가장 널리 사용되는 강력한 베이스라인(baseline) 손실 함수 중 하나로 확고히 자리매김하고 있다.

학문적으로, CosFace가 제시한 '안정적인 마진 설계'라는 방향성은 이후 등장한 수많은 적응형 마진 연구들의 이론적, 실용적 출발점이 되었다. 고정된 마진의 한계를 지적하고 이를 극복하려는 시도들은 모두 CosFace가 구축한 안정적인 프레임워크 위에서 이루어졌다고 해도 과언이 아니다.

나아가 CosFace의 영향력은 얼굴 인식 분야에만 국한되지 않는다. 특징 벡터 간의 유사도나 거리를 정밀하게 측정하는 것이 중요한 모든 종류의 거리 학습(Metric Learning) 문제—예를 들어, 이미지 검색(image retrieval), 사람 재식별(person re-identification), 상품 추천 시스템 등—에 CosFace의 아이디어가 영감을 주거나 직접적으로 적용될 수 있다.

결론적으로, CosFace는 단순히 하나의 우수한 손실 함수를 제안한 것을 넘어, 심층 특징 학습의 최적화 목표를 재정의하고, 안정성과 성능을 모두 갖춘 실용적인 해결책을 제시함으로써 관련 분야의 연구 패러다임을 한 단계 진일보시킨 핵심적인 연구로 평가받아야 마땅하다.


1. Cosface: Large Margin Cosine Loss For Deep Face Recognition - Scribd, 8월 23, 2025에 액세스, https://www.scribd.com/document/686090262/1801-09414v2
2. [1801.09414] CosFace: Large Margin Cosine Loss for Deep Face Recognition - ar5iv, 8월 23, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/1801.09414
3. CosFace: Large Margin Cosine Loss for Deep Face Recognition, 8월 23, 2025에 액세스, https://arxiv.org/abs/1801.09414
4. CosFace: Large Margin Cosine Loss for Deep Face Recognition - ResearchGate, 8월 23, 2025에 액세스, https://www.researchgate.net/publication/322788225_CosFace_Large_Margin_Cosine_Loss_for_Deep_Face_Recognition
5. Loss Function Search for Face Recognition - Proceedings of ..., 8월 23, 2025에 액세스, http://proceedings.mlr.press/v119/wang20t/wang20t.pdf
6. [1812.11317] Support Vector Guided Softmax Loss for Face Recognition - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/abs/1812.11317
7. arXiv:1801.09414v2 [cs.CV] 3 Apr 2018, 8월 23, 2025에 액세스, https://arxiv.org/pdf/1801.09414
8. ArcFace loss function for Deep Face Recognition by payyavula saiprakash - Medium, 8월 23, 2025에 액세스, https://medium.com/@payyavulasaiprakash/arcface-loss-function-for-deep-face-recognition-e1ff5e173b52
9. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - ResearchGate, 8월 23, 2025에 액세스, https://www.researchgate.net/publication/322674945_ArcFace_Additive_Angular_Margin_Loss_for_Deep_Face_Recognition
10. X2-Softmax: Margin Adaptive Loss Function for Face Recognition - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/pdf/2312.05281
11. CosFace: Large Margin Cosine Loss for Deep Face Recognition - Semantic Scholar, 8월 23, 2025에 액세스, https://www.semanticscholar.org/paper/CosFace%3A-Large-Margin-Cosine-Loss-for-Deep-Face-Wang-Wang/9dc915697768dd1f7c7b97e2c25c90b02241958b
12. SphereFace: Deep Hypersphere Embedding for Face Recognition - ResearchGate, 8월 23, 2025에 액세스, https://www.researchgate.net/publication/316505674_SphereFace_Deep_Hypersphere_Embedding_for_Face_Recognition
13. CosFace loss function for face recognition | Analytics Vidhya - Medium, 8월 23, 2025에 액세스, https://medium.com/analytics-vidhya/exploring-other-face-recognition-approaches-part-1-cosface-4aed39afe7a8
14. arXiv:1905.00292v2 [cs.CV] 7 May 2019, 8월 23, 2025에 액세스, https://arxiv.org/pdf/1905.00292
15. SphereFace: Deep Hypersphere Embedding for ... - CVF Open Access, 8월 23, 2025에 액세스, https://openaccess.thecvf.com/content_cvpr_2017/papers/Liu_SphereFace_Deep_Hypersphere_CVPR_2017_paper.pdf
16. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - CVF Open Access, 8월 23, 2025에 액세스, https://openaccess.thecvf.com/content_CVPR_2019/papers/Deng_ArcFace_Additive_Angular_Margin_Loss_for_Deep_Face_Recognition_CVPR_2019_paper.pdf
17. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - InsightFace, 8월 23, 2025에 액세스, https://insightface.ai/arcface
18. ArcFace: Additive Angular Margin Loss for Deep Face Recognition | by Mostafa Gazar | 1-minute papers | Medium, 8월 23, 2025에 액세스, https://medium.com/1-minute-papers/arcface-additive-angular-margin-loss-for-deep-face-recognition-d02297605f8d
19. Comparing Angular Margin Loss Functions | by zestyoreo - Medium, 8월 23, 2025에 액세스, https://zestyoreo9.medium.com/comparing-angular-margin-loss-functions-31f62c6f4032
20. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - YouTube, 8월 23, 2025에 액세스, https://www.youtube.com/watch?v=H1qEp_czI1I
21. CosFace: Large Margin Cosine Loss for Deep Face - CVPR 2018 Open Access Repository, 8월 23, 2025에 액세스, https://openaccess.thecvf.com/content_cvpr_2018/html/Wang_CosFace_Large_Margin_CVPR_2018_paper.html
22. Unifying Margin-Based Softmax Losses in Face Recognition - CVF Open Access, 8월 23, 2025에 액세스, https://openaccess.thecvf.com/content/WACV2023/papers/Zhang_Unifying_Margin-Based_Softmax_Losses_in_Face_Recognition_WACV_2023_paper.pdf
23. X2-Softmax: Margin Adaptive Loss Function for Face Recognition - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/html/2312.05281v2
24. AdaFace: Quality Adaptive Margin for Face Recognition - CVF Open Access, 8월 23, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2022/papers/Kim_AdaFace_Quality_Adaptive_Margin_for_Face_Recognition_CVPR_2022_paper.pdf
25. MFCosface: A Masked-Face Recognition Algorithm Based on Large Margin Cosine Loss, 8월 23, 2025에 액세스, https://www.mdpi.com/2076-3417/11/16/7310
26. [D] Understanding arcface, sphereface, and their differences. : r/MachineLearning - Reddit, 8월 23, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/jicbn2/d_understanding_arcface_sphereface_and_their/
27. KappaFace: Adaptive Additive Angular Margin Loss for Deep Face Recognition - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/html/2201.07394v2
28. [2111.11186] GB-CosFace: Rethinking Softmax-based Face Recognition from the Perspective of Open Set Classification - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/abs/2111.11186


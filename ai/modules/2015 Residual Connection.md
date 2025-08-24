# 잔차 연결(Residual Connection, 2015)
[딥러닝 모듈 (Modules)](./index.md)


심층 신경망(Deep Neural Networks)의 발전사는 모델의 '깊이(depth)'를 통해 표현력(representational power)을 증대시키려는 끊임없는 도전의 역사라 할 수 있다. 2010년대 초반, AlexNet의 성공 이후 VGGNet과 같은 모델들은 컨볼루션 계층을 더 깊이 쌓음으로써 이미지 인식과 같은 복잡한 과제에서 전례 없는 성능 향상을 이끌어냈다.1 이러한 경험적 성공은 '모델의 깊이가 깊어질수록 더 복잡하고 추상적인 특징을 학습할 수 있다'는 가설을 뒷받침하며, 딥러닝 커뮤니티의 주류 패러다임으로 자리 잡았다. 그러나 이 패러다임은 무한정 유효하지 않았다. 연구자들은 네트워크의 깊이가 특정 임계점을 넘어서면서 예측과는 정반대의 현상에 직면하게 되었다.


네트워크의 깊이를 수십 개 층 이상으로 늘리자, 모델의 성능이 포화 상태에 머무는 것을 넘어 급격히 저하되는 현상이 관찰되었다. 놀랍게도, 이러한 성능 저하는 테스트 오차(test error)뿐만 아니라 훈련 오차(training error)에서도 발생했다.1 예를 들어, 56계층 네트워크가 20계층 네트워크보다 더 높은 훈련 오차를 기록하는, 직관에 반하는 결과가 나타난 것이다.4 이는 단순히 더 많은 파라미터로 인한 과적합(overfitting) 문제가 아니었다. 오히려, 더 깊은 모델이 더 얕은 모델이 이미 찾은 해(solution)조차 학습하지 못하는, 근본적인 '최적화(optimization)'의 어려움을 시사했다. 이 현상을 '성능 저하(degradation) 문제'라 칭한다.4

이러한 최적화 난제의 배경에는 오랫동안 심층 네트워크 훈련의 발목을 잡아온 '기울기 소실(vanishing gradient)' 문제가 자리하고 있었다.6 역전파(backpropagation) 알고리즘은 출력 계층에서 계산된 손실(loss)의 기울기를 입력 계층 방향으로 전파하며 각 계층의 가중치를 업데이트한다.9 이때 미적분의 연쇄 법칙(chain rule)에 따라 각 계층을 거치면서 기울기가 반복적으로 곱해지게 된다.10 만약 Sigmoid나 Tanh와 같은 전통적인 활성화 함수를 사용한다면, 이들의 도함수(derivative) 값은 최대값이 각각 0.25와 1.0으로, 대부분의 영역에서 1보다 작다.10 따라서 수많은 계층을 거치면서 1보다 작은 값들이 계속 곱해지면, 기울기는 지수적으로 감소하여 결국 0에 가까워진다. 그 결과, 입력에 가까운 초기 계층들은 거의 의미 있는 가중치 업데이트를 받지 못하고 학습이 정체되는 현상이 발생한다.6

성능 저하 문제와 기울기 소실 문제는 밀접하게 연관되어 있지만, 동일한 현상은 아니다. ResNet 원 논문은 "더 깊은 네트워크가 수렴하기 시작할 때(when deeper networks are able to start converging), 성능 저하 문제가 드러났다"고 기술한다.1 이는 기울기 소실이 학습 초기의 수렴 자체를 방해하는 근본적인 장벽이라면, 성능 저하는 일단 수렴이 시작된 이후에도 더 깊은 모델이 최적의 해를 찾지 못하는, 보다 미묘한 최적화의 실패임을 암시한다. 즉, 성능 저하 문제는 기울기 소실에 의해 악화되지만, 그 근본 원인은 심층 비선형 계층 스택이 잠재적으로 유용한 함수, 심지어는 가장 단순한 '항등 매핑(identity mapping)'조차 학습하기 어렵다는 데 있다.


2015년, Kaiming He 연구팀은 'Deep Residual Learning for Image Recognition' 논문을 통해 이러한 심층 네트워크 훈련의 근본적인 난제를 해결할 혁신적인 패러다임을 제시했다.1 그들은 문제의 본질을 '네트워크가 무엇을 학습해야 하는가'라는 관점에서 재정의했다. 기존 방식처럼 여러 계층이 쌓여 복잡한 목표 함수 $H(x)$를 직접 학습하도록 하는 대신, 이 계층들이 '잔차 함수(residual function)' $F(x) := H(x) - x$를 학습하도록 네트워크 구조를 재구성했다. 그리고 원래의 목표 함수는 $H(x) = F(x) + x$의 형태로 복원된다.

이 구조의 핵심은 $+ x$ 항을 구현하는 '잔차 연결(residual connection)' 또는 '지름길 연결(shortcut connection)'이다. 이는 입력 `x`를 몇 개의 계층을 건너뛰어 출력에 직접 더해주는 간단한 구조적 변경에 불과했지만, 그 효과는 혁명적이었다. 잔차 연결은 심층 네트워크 내부에 정보와 기울기가 막힘없이 흐를 수 있는 '고속도로(highway)'를 제공함으로써, 성능 저하 문제와 기울기 소실 문제를 동시에 완화했다.13 이로써 인류는 처음으로 수백, 수천 계층에 달하는 초심층 신경망을 안정적으로 훈련할 수 있게 되었고, 딥러닝 아키텍처 설계는 새로운 시대를 맞이하게 되었다.

본 보고서는 딥러닝 아키텍처의 발전에 이정표를 세운 잔차 연결에 대한 심층적 고찰을 목적으로 한다. 먼저 잔차 학습의 이론적 기초와 수학적 원리를 분석하고, 이를 구현한 ResNet 아키텍처의 구조적 특징을 상세히 탐구한다. 나아가 Pre-activation ResNet, Wide ResNet, ResNeXt, DenseNet 등 잔차 연결의 개념을 계승하고 발전시킨 다양한 후속 아키텍처들을 비교 분석한다. 또한, 현대 딥러닝의 표준으로 자리 잡은 트랜스포머 아키텍처에서 잔차 연결이 수행하는 핵심적인 역할을 조명한다. 마지막으로, 잔차 연결의 한계와 비판적 시각을 검토하고, 생성 모델에서의 새로운 재해석 등 최신 연구 동향을 바탕으로 미래 아키텍처 설계에 대한 전망을 제시하고자 한다.


잔차 학습의 혁신성은 문제 자체를 재정의하는 데서 출발한다. 심층 네트워크가 왜 더 깊어질수록 학습하기 어려워지는가에 대한 근본적인 질문에 답하기 위해, 잔차 학습은 '항등 매핑'이라는 개념을 중심으로 새로운 학습 프레임워크를 제시했다.


이론적으로, 더 깊은 네트워크는 최소한 더 얕은 네트워크만큼의 성능을 보여야 한다. 예를 들어, $N$계층 네트워크가 최적의 성능을 보인다고 가정하자. 여기에 몇 개의 계층을 더 추가하여 $N+M$계층 네트워크를 구성할 때, 새로 추가된 $M$개의 계층이 단순히 입력을 그대로 출력하는 항등 매핑(identity mapping) 역할만 수행한다면, 이 $N+M$계층 네트워크는 $N$계층 네트워크와 동일한 성능을 달성할 수 있어야 한다. 즉, 더 깊은 모델의 함수 공간은 더 얕은 모델의 함수 공간을 포함하므로, 솔루션 공간 역시 더 작아질 리가 없다.

그러나 실제 실험 결과는 이 이론적 추론과 상반되었다.4 더 깊은 네트워크의 훈련 오차가 더 얕은 네트워크보다 높게 나타나는 성능 저하 문제는, 경사 하강법(SGD)과 같은 표준적인 최적화 알고리즘이 여러 개의 비선형 계층(예: `ReLU(Weight * x)`)을 쌓아 항등 함수 $H(x) = x$를 근사하는 데 심각한 어려움을 겪는다는 사실을 방증한다. 솔버(solver)가 이 자명해 보이는 해를 찾는 데 실패하는 것이다. 이것이 바로 성능 저하 문제의 핵심에 놓인 최적화의 난제이다.


ResNet의 제안자들은 이 문제를 해결하기 위해 학습 목표를 재설계했다. 네트워크의 일부 계층(빌딩 블록)이 학습해야 할 이상적인 기본 매핑(underlying mapping)을 $H(x)$라고 하자. 기존의 접근 방식은 이 빌딩 블록이 $H(x)$를 직접적으로 근사하도록 학습시켰다.

잔차 학습은 이 접근법을 뒤집는다. 빌딩 블록이 $H(x)$ 대신, 입력 `x`에 대한 '잔차(residual)'인 $F(x) := H(x) - x$를 학습하도록 명시적으로 재구성(reformulate)한다.1 그러면 원래의 목표 함수 $H(x)$는 $F(x) + x$로 간단히 복원될 수 있다. 여기서 $+ x$ 연산은 '지름길 연결(shortcut connection)'이라 불리는, 입력을 어떠한 변환도 거치지 않고(항등 매핑) 출력에 바로 더해주는 구조를 통해 구현된다.4

이러한 재구성의 이점은 극명하다. 만약 이상적인 매핑이 항등 함수, 즉 $H(x) = x$인 경우를 생각해보자. 기존 방식에서는 여러 계층의 비선형 변환을 통해 `x`를 출력하도록 복잡한 가중치들을 학습해야 한다. 반면, 잔차 학습 프레임워크에서는 잔차 함수 $F(x) = H(x) - x = x - x = 0$을 학습하면 된다. 여러 계층의 가중치를 모두 0으로 만드는 것은 복잡한 함수를 학습하는 것보다 훨씬 쉽다. 즉, 잔차 학습은 최적화 목표를 더 쉬운 방향으로 편향(bias)시킨다. 실제로는 이상적인 매핑이 순수한 항등 함수는 아니겠지만, 입력에 대한 작은 변화(perturbation)에 가깝다면, 잔차 함수 $F(x)$는 0에 가까운 작은 값을 학습하게 될 것이며, 이는 여전히 복잡한 전체 함수 $H(x)$를 처음부터 학습하는 것보다 용이하다.4


잔차 연결의 효과는 순전파와 역전파 과정을 수학적으로 분석함으로써 더욱 명확하게 이해할 수 있다.


하나의 잔차 블록을 생각해보자. $l$번째 잔차 블록의 입력을 $x_l$, 출력을 $x_{l+1}$이라 하고, 이 블록이 학습하는 잔차 함수를 $F(x_l, W_l)$이라 하자. 여기서 $W_l$은 $l$번째 블록의 학습 가능한 가중치 집합이다. 잔차 블록의 순전파 과정은 다음과 같이 공식화할 수 있다.18
$$
x_{l+1} = \text{ReLU}(F(x_l, W_l) + x_l)
$$
여기서는 편의를 위해 블록 간 활성화 함수를 생략하고 $x_{l+1} = F(x_l, W_l) + x_l$로 단순화하여 분석을 진행한다. (이후 Pre-activation ResNet에서 이 활성화 함수의 위치가 중요하게 다뤄진다.)

이 관계를 $l$번째 블록부터 더 깊은 $L$번째 블록까지 재귀적으로 적용하면 다음과 같다.
$$
x_{l+1} = F(x_l, W_l) + x_l \\
x_{l+2} = F(x_{l+1}, W_{l+1}) + x_{l+1} = F(x_{l+1}, W_{l+1}) + F(x_l, W_l) + x_l \\
\dots
$$
이를 일반화하면, $L$번째 블록의 입력 $x_L$은 이전의 $l$번째 블록 입력 $x_l$에 대한 식으로 표현될 수 있다.18
$$
x_L = x_l + \sum_{i=l}^{L-1} F(x_i, W_i)
$$
이 수식은 매우 중요한 의미를 갖는다. 이는 더 얕은 계층 $l$의 정보 $x_l$이 중간의 모든 잔차 함수들의 합을 거쳐 더 깊은 계층 $L$에 직접적으로 기여함을 보여준다. 즉, $x_l$에서 $x_L$로 정보가 감쇠 없이 직접 전달되는 경로가 존재함을 수학적으로 명확히 한 것이다. 이는 순전파 과정에서 초기 계층의 정보가 깊은 계층까지 손실 없이 전달되는 데 기여한다.


잔차 연결의 진정한 위력은 역전파 과정에서 드러난다. 전체 네트워크의 손실 함수를 $E$라고 할 때, 역전파의 목표는 $E$를 각 계층의 가중치 $W_i$로 미분한 값, 즉 기울기 $∂E/∂W_i$를 계산하는 것이다. 이를 위해 먼저 $E$를 각 계층의 입력 $x_l$로 미분한 값 $∂E/∂x_l$을 계산해야 한다. 연쇄 법칙에 따라 다음과 같이 쓸 수 있다.6
$$
\frac{\partial E}{\partial x_l} = \frac{\partial E}{\partial x_L} \frac{\partial x_L}{\partial x_l}
$$
여기서 $∂x_L/∂x_l$ 항에 위에서 유도한 순전파 공식을 대입하여 미분하면 다음과 같은 핵심적인 결과를 얻는다.
$$
\frac{\partial x_L}{\partial x_l} = 1 + \frac{\partial}{\partial x_l} \sum_{i=l}^{L-1} F(x_i, W_i)
$$
따라서, $x_l$에 대한 최종 기울기는 다음과 같다.
$$
\frac{\partial E}{\partial x_l} = \frac{\partial E}{\partial x_L} \left( 1 + \frac{\partial}{\partial x_l} \sum_{i=l}^{L-1} F(x_i, W_i) \right) = \frac{\partial E}{\partial x_L} + \frac{\partial E}{\partial x_L} \frac{\partial}{\partial x_l} \sum_{i=l}^{L-1} F(x_i, W_i)
$$
**핵심 분석:** 위 수식의 첫 번째 항 $∂E/∂x_L$이 잔차 연결이 기울기 소실 문제를 완화하는 비밀을 담고 있다. 이 항은 더 깊은 계층 $L$에서의 기울기가 아무런 가중치 곱 없이 그대로($1$이 곱해져) 더 얕은 계층 $l$로 직접 전파됨을 의미한다. 두 번째 항, 즉 잔차 함수들을 통과하는 경로의 기울기($∂/∂x_l ΣF$)가 매우 작아져 기울기 소실이 발생하더라도, 첫 번째 항 덕분에 전체 기울기 $∂E/∂x_l$이 $0$이 되는 것을 방지한다.6 이 '덧셈 항등원'의 존재가 심층 네트워크 전반에 걸쳐 안정적인 기울기 흐름을 보장하는 핵심 메커니즘이다.


잔차 연결이 기울기 소실을 '해결(solve)'한다고 흔히 설명되지만, 보다 정교한 해석은 '회피(avoid)'에 가깝다는 것이다.16 잔차 연결은 잔차 함수 $F(x)$ 자체의 내부에서 발생하는 기울기 소실을 막지는 못한다. 대신, 기울기가 소실될 위험이 있는 깊은 경로를 '우회'할 수 있는 수많은 짧은 경로를 제공한다.

Veit 등의 연구("Residual Networks Behave Like Ensembles of Relatively Shallow Networks", 2016)는 ResNet을 '풀린 관점(unraveled view)'에서 분석했다. 이 관점에 따르면, $N$개의 잔차 블록으로 구성된 ResNet은 단일한 $N$계층 네트워크가 아니라, $2^N$개의 서로 다른 경로 길이를 갖는 네트워크들의 암묵적인 앙상블(implicit ensemble)처럼 동작한다. 각 잔차 블록에서 신호는 잔차 함수를 통과하거나 지름길을 통해 건너뛸 수 있는 두 가지 선택지를 가지기 때문이다.

이러한 관점에서 볼 때, 역전파 시 기울기는 주로 이 앙상블 내의 상대적으로 '짧은 경로'들을 통해 효과적으로 전파된다.16 즉, ResNet은 초심층 구조가 제공하는 풍부한 표현력을 잠재적으로 활용하면서도, 실제 학습은 주로 안정적인 기울기 전파가 가능한 얕은 네트워크들의 집합에서 이루어지는 효과를 얻는다. 이는 왜 수백, 수천 계층의 네트워크가 파국적인 실패 없이 성공적으로 훈련될 수 있는지에 대한 더 깊은 통찰을 제공한다. 잔차 연결의 진정한 힘은 단순히 기울기 값을 보존하는 것을 넘어, 네트워크의 유효 깊이(effective depth)를 동적으로 조절하여 학습을 안정화시키는 데 있다.


잔차 학습이라는 이론적 프레임워크는 ResNet(Residual Network)이라는 구체적인 아키텍처를 통해 구현되었다. ResNet은 잔차 블록(residual block)이라는 모듈화된 구조를 반복적으로 쌓아 구성되며, 네트워크의 깊이와 계산 효율성에 따라 몇 가지 주요 변형을 가진다.


ResNet의 핵심 구성 요소는 하나 이상의 컨볼루션 계층과 지름길 연결로 이루어진 잔차 블록이다. 네트워크의 전체 깊이와 계산 복잡도를 고려하여 두 가지 주요 형태의 블록이 설계되었다.


기본 블록은 주로 ResNet-18, ResNet-34와 같이 상대적으로 얕은 네트워크에서 사용된다.1 이 블록의 구조는 매우 직관적이며, 두 개의 연속된 3x3 컨볼루션 계층으로 구성된다. 각 컨볼루션 계층 다음에는 배치 정규화(Batch Normalization)와 ReLU 활성화 함수가 적용된다. 입력 $x$는 이 두 개의 컨볼루션 계층을 통과한 출력 $F(x)$에 더해진 후, 최종적으로 ReLU 활성화 함수를 거쳐 블록의 최종 출력이 된다.17

입력과 출력의 차원(채널 수와 공간 해상도)이 동일한 경우, 지름길 연결은 어떠한 추가 파라미터나 계산 없이 입력을 그대로 전달하는 순수한 항등 매핑(identity mapping)으로 구현된다. 이는 잔차 학습의 개념을 가장 단순하고 직접적으로 구현한 형태이다.


ResNet-50, ResNet-101, ResNet-152와 같이 훨씬 더 깊은 네트워크를 효율적으로 구축하기 위해 병목 블록이 고안되었다.1 이 구조는 계산 효율성을 극대화하는 데 초점을 맞춘다. 병목 블록은 세 개의 컨볼루션 계층 스택으로 구성된다: 1x1, 3x3, 1x1 컨볼루션.

1. **차원 축소 (1x1 Conv):** 첫 번째 1x1 컨볼루션 계층은 입력 특징 맵의 채널 수를 줄이는 역할을 한다. 예를 들어, 256개의 채널을 가진 입력을 64개 채널로 압축한다. 이는 계산량이 많은 3x3 컨볼루션의 입력 채널 수를 줄여 전체 계산 복잡도를 낮추기 위함이다.
2. **특징 학습 (3x3 Conv):** 중앙의 3x3 컨볼루션 계층은 이 축소된 '병목' 상태의 특징 맵 위에서 공간적 특징을 학습한다.
3. **차원 복원 (1x1 Conv):** 마지막 1x1 컨볼루션 계층은 다시 채널 수를 원래대로 복원한다. 예를 들어, 64개 채널을 다시 256개 채널로 확장한다.

이 병목 구조 덕분에, 동일한 입출력 차원을 갖는 기본 블록에 비해 파라미터 수와 부동소수점 연산(FLOPs)을 현저하게 줄일 수 있다. 이로 인해 훨씬 더 깊은 네트워크를 현실적인 계산 자원 내에서 훈련하는 것이 가능해졌다.


네트워크가 깊어지면서 특징 맵의 공간 해상도는 줄어들고(예: 스트라이드 2의 컨볼루션이나 풀링을 통해) 채널 수는 증가하는 것이 일반적이다. 이 경우, 잔차 블록의 입력 `x`와 출력 `F(x)`의 차원이 달라져 단순 덧셈이 불가능해진다. 이러한 불일치를 해결하기 위해 지름길 연결에도 변환을 적용해야 한다. 가장 일반적인 방법은 스트라이드 2를 갖는 1x1 컨볼루션을 지름길에 적용하여 공간 해상도와 채널 수를 동시에 맞춰주는 것이다.16 이 방법을 '투영 지름길(projection shortcut)'이라 부르며, 항등 매핑만으로는 부족할 때 모델의 표현력을 높이는 부가적인 효과도 있다.

아래 표는 입력 채널 256, 출력 채널 256을 가정했을 때 기본 블록과 병목 블록의 구조 및 계산 복잡도를 비교한 것이다. 병목 블록의 설계 철학과 효율성을 명확히 보여준다.

| 구분                   | 기본 블록 (ResNet-34 스타일)                         | 병목 블록 (ResNet-50+ 스타일)                                |
| ---------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
| **구조**               | `3x3 Conv, 256, stride=1`  `3x3 Conv, 256, stride=1` | `1x1 Conv, 64, stride=1` (차원 축소)  `3x3 Conv, 64, stride=1`  `1x1 Conv, 256, stride=1` (차원 복원) |
| **설계 목표**          | 잔차 학습의 개념적 구현                              | 깊은 네트워크에서의 계산 효율성 극대화                       |
| **입출력 채널**        | 256 → 256                                            | 256 → 256                                                    |
| **파라미터 수 (예시)** | `2 * (3*3*256*256) = 1,179,648`                      | `(1*1*256*64) + (3*3*64*64) + (1*1*64*256) = 69,632`         |
| **주 사용 모델**       | ResNet-18, ResNet-34                                 | ResNet-50, ResNet-101, ResNet-152                            |


ResNet의 전체적인 매크로 아키텍처는 VGGNet의 설계 철학을 상당 부분 계승했다.4 즉, 주로 3x3 크기의 작은 컨볼루션 필터를 사용하고, 특징 맵의 공간 해상도를 절반으로 줄일 때마다 채널 수를 두 배로 늘리는 규칙적인 구조를 따른다.

ResNet의 효과를 입증하기 위해, 연구팀은 ImageNet 데이터셋에서 잔차 연결이 없는 '일반 네트워크(Plain Network)'와 ResNet을 직접 비교하는 결정적인 실험을 수행했다.

- **성능 저하 문제의 명확한 재현:** 18계층과 34계층의 일반 네트워크를 비교했을 때, 더 깊은 34계층 네트워크가 18계층 네트워크보다 더 높은 훈련 오차와 검증 오차를 보였다.1 이는 깊이가 깊어짐에 따라 최적화가 더 어려워지는 성능 저하 문제를 명확하게 재현한 결과이다.
- **잔차 학습의 효과 입증:** 반면, 동일한 깊이의 ResNet을 비교했을 때, 34계층 ResNet은 18계층 ResNet보다 훨씬 낮은 오차율을 기록했다.1 이는 잔차 학습 프레임워크가 깊이 증가로 인한 최적화 문제를 성공적으로 해결하고, 깊이로부터 실질적인 성능 향상(accuracy gain)을 이끌어낼 수 있음을 강력하게 입증한 것이다.

이러한 성공에 힘입어, 연구팀은 병목 블록을 사용하여 훨씬 더 깊은 ResNet-50, ResNet-101, 그리고 당시로서는 상상하기 어려웠던 152계층의 ResNet-152를 구축했다. ResNet-152는 VGGNet보다 8배나 더 깊었음에도 불구하고, 병목 구조 덕분에 오히려 더 낮은 계산 복잡도(FLOPs)와 적은 파라미터 수를 가졌다.1

결과적으로, ResNet 앙상블 모델은 ILSVRC 2015 이미지 분류 대회에서 3.57%라는 기록적인 top-5 오류율을 달성하며 우승을 차지했다.1 이는 단순히 하나의 대회 우승을 넘어, '깊이'의 장벽을 허물고 딥러닝 아키텍처 설계의 새로운 표준을 제시한 역사적인 성과였다. 이후 ResNet은 이미지 분류뿐만 아니라 객체 탐지, 분할 등 다양한 컴퓨터 비전 분야의 기반 모델로 자리 잡았으며, 그 영향력은 오늘날까지 이어지고 있다.


ResNet의 성공은 딥러닝 아키텍처 연구에 새로운 방향을 제시했다. 연구자들은 단순히 네트워크를 더 깊게 쌓는 것을 넘어, 잔차 연결의 본질을 탐구하고 정보 흐름을 최적화하며, 제한된 계산 자원 내에서 표현력을 극대화하는 방향으로 연구를 확장해 나갔다. 이러한 과정은 '어떻게 하면 더 적은 비용으로 더 많은 것을 학습할 수 있는가?'라는 근본적인 질문에 대한 다각적인 답변을 모색하는 여정이었다.


ResNet 원 논문 이후, 동일한 저자들은 "Identity Mappings in Deep Residual Networks"라는 후속 연구를 통해 잔차 블록 내부의 미세 구조가 정보 전파에 미치는 영향을 심도 있게 분석했다.22

기존 ResNet 블록(Post-activation)의 순전파 공식은 $y = ReLU(F(x) + x)$ 형태였다. 여기서 문제는 지름길 $x$와 잔차 경로 $F(x)$가 합쳐진 '이후(post)'에 활성화 함수 ReLU가 적용된다는 점이다. 이로 인해 지름길을 통해 전달되는 신호가 비선형 변환을 거치게 되어, 순수한 정보의 흐름을 다소 방해하게 된다.

**Pre-activation ResNet**은 이 문제를 해결하기 위해 블록 내 구성 요소의 순서를 재배치했다. 배치 정규화(BN)와 ReLU 활성화 함수를 컨볼루션 계층의 '앞(pre)'으로 이동시킨 것이다. 새로운 블록의 구조는 `BN -> ReLU -> Conv` 순서가 되며, 지름길 연결은 이 변환 스택의 출력에 더해진다.

- **Post-activation:** `Conv -> BN -> ReLU -> (+) -> ReLU`
- **Pre-activation:** `BN -> ReLU -> Conv -> BN -> ReLU -> Conv -> (+)`

이러한 재배치를 통해 두 가지 중요한 이점을 얻었다. 첫째, 지름길 연결이 어떠한 변환도 거치지 않는 완전한 '항등 매핑(clean identity path)'이 되었다. 둘째, 잔차 경로의 출력 또한 활성화 함수를 거치지 않은 상태로 지름길과 합쳐진다. 이 구조는 순전파 시의 정보와 역전파 시의 기울기가 어떤 방해도 받지 않고 한 블록에서 다른 블록으로 직접 전달될 수 있는 가장 깨끗한 경로를 보장한다.22

이 미세한 구조적 변경은 놀라운 결과를 가져왔다. Pre-activation ResNet은 기존 ResNet보다 훈련이 더 용이하고, 더 나은 일반화 성능을 보였다. 특히 이 구조 덕분에 1001계층에 달하는 극단적으로 깊은 네트워크의 훈련이 안정적으로 가능해졌으며, 이는 잔차 연결의 핵심인 '방해받지 않는 정보 흐름'을 극대화하는 것이 얼마나 중요한지를 보여주었다.22


ResNet이 깊이의 한계를 극복했지만, 무한정 깊게 만드는 것이 항상 최적의 전략은 아니라는 인식이 확산되었다. 깊이가 한계 효용 체감에 도달했을 때, 다른 차원에서 모델의 표현력을 확장하려는 시도가 나타났다.


"Wide Residual Networks" 논문은 "깊이 대신 너비(width)를 늘리는 것이 더 효율적이다"라는 도발적인 주장을 펼쳤다.25 연구자들은 매우 깊고 얇은(thin) 네트워크는 훈련 과정에서 많은 잔차 블록이 거의 학습에 기여하지 않는 '특징 재사용 감소(diminishing feature reuse)' 문제를 겪을 수 있다고 지적했다.

이에 대한 해결책으로, 네트워크의 깊이를 줄이는 대신 각 컨볼루션 계층의 채널 수, 즉 '너비'를 늘리는 방식을 제안했다. 실험 결과는 인상적이었다. 예를 들어, 16계층의 넓은 WRN이 1000계층의 얇은 ResNet보다 더 나은 성능을 보이면서도 훈련 속도는 훨씬 빨랐다.25 이는 ResNet의 진정한 힘이 깊이 그 자체보다는 '잔차 블록'이라는 구조적 혁신에 있으며, 제한된 파라미터 예산 내에서는 깊이와 너비 사이의 적절한 균형을 찾는 것이 중요함을 시사했다.


ResNeXt는 ResNet의 VGG와 같은 단순한 반복 구조와 Inception 계열 아키텍처의 'split-transform-merge' 전략을 우아하게 결합했다.27 이 아키텍처는 깊이와 너비에 더해 **'카디널리티(cardinality)'**라는 새로운 차원을 도입했다.27 카디널리티는 집계될 병렬 변환(transformation) 경로의 수를 의미한다.

ResNeXt 블록은 기존 ResNet 블록의 넓은 3x3 컨볼루션을 여러 개의 동일한 토폴로지를 가진 좁은 병렬 경로로 분할한다. 각 경로는 저차원 임베딩에 대한 변환을 수행하고, 이들의 출력은 최종적으로 합산되어 집계된다. 이 구조는 그룹 컨볼루션(grouped convolution)을 통해 매우 효율적으로 구현될 수 있다.

핵심적인 발견은, 전체 계산 복잡도(FLOPs와 파라미터 수)를 동일하게 유지하더라도, 너비를 늘리는 것보다 카디널리티를 높이는 것이 성능 향상에 더 효과적이라는 점이었다.29 예를 들어, 101계층 ResNeXt는 200계층 ResNet보다 절반의 계산 복잡도로 더 우수한 성능을 달성했다.29 이는 더 다양하고 분리된 특징 표현을 학습하는 것이 모델의 표현력을 높이는 데 효과적인 전략임을 보여준다.


DenseNet(Densely Connected Convolutional Networks)은 ResNet의 '특징 재사용'이라는 아이디어를 가장 극단적인 형태로 발전시킨 아키텍처이다.30

ResNet이 $F(x)$와 $x$를 **덧셈(addition)**으로 결합하는 것과 달리, DenseNet은 모든 선행 계층들의 특징 맵을 현재 계층의 입력으로 사용하되, 이를 **연결(concatenation)** 방식으로 결합한다.32 즉, 

$l$번째 계층의 입력은 $0$번째부터 $l-1$번째까지 모든 계층의 출력 특징 맵들을 채널 차원에서 이어 붙인 것이 된다.

- **ResNet:** $x_l = H_l(x_{l-1}) + x_{l-1}$
- **DenseNet:** $x_l = H_l([x_0, x_1,..., x_{l-1}])$

이러한 '밀집된 연결(dense connectivity)'은 다음과 같은 강력한 장점을 가진다.

1. **최대화된 특징 재사용:** 모든 계층이 이전 모든 계층의 정보에 직접 접근할 수 있어, 특징이 네트워크 전체에 걸쳐 재사용된다. 이로 인해 중복된 특징 맵을 다시 학습할 필요가 없어 극도로 높은 파라미터 효율성을 달성한다.30
2. **강화된 정보/기울기 흐름:** 각 계층이 손실 함수와 입력 신호로부터 직접적인 감독(supervision)을 받는 효과가 있어, 기울기 소실 문제가 크게 완화되고 깊은 네트워크의 훈련이 용이해진다.33
3. **정규화 효과:** 작은 데이터셋에서도 과적합을 줄이는 경향이 있다.33

물론 단점도 존재한다. 특징 맵을 계속 연결해 나가기 때문에 계층이 깊어질수록 메모리 사용량이 급격히 증가한다. 이를 해결하기 위해 DenseNet은 네트워크를 여러 개의 'Dense Block'으로 나누고, 블록 사이에는 채널 수를 조절하고 공간 해상도를 줄이는 'Transition Layer'를 배치하는 계층적 구조를 사용한다.


ResNet이 발표되기 약 7개월 전, Rupesh Srivastava, Klaus Greff, Jürgen Schmidhuber는 "Highway Networks"라는 논문을 통해 초심층 신경망 훈련의 가능성을 처음으로 열었다.35 이 아키텍처는 LSTM의 게이트 메커니즘에서 영감을 받아, 정보의 흐름을 동적으로 조절하는 학습 가능한 '게이트'를 도입했다.35

Highway Network의 계층 변환은 다음과 같이 공식화된다.
$$
y = H(x, W_H) \cdot T(x, W_T) + x \cdot C(x, W_C)
$$
여기서 $H$는 비선형 변환, $T$는 '변환 게이트(transform gate)', $C$는 '전달 게이트(carry gate)'이다. $T$는 입력 $x$를 얼마나 변환할지를, $C$는 얼마나 그대로 통과시킬지를 결정한다. 일반적으로 $C = 1 - T$로 설정하여 두 게이트의 역할을 상보적으로 만든다.37

ResNet은 이 Highway Network의 특별한 경우로 해석될 수 있다. 즉, 변환 게이트 $T$와 전달 게이트 $C$가 데이터에 의존하지 않고 항상 $1$로 고정된, '게이트가 없는(ungated)' 또는 '항상 열려있는' Highway Network인 셈이다.35 ResNet의 이러한 극단적인 단순성이 오히려 더 안정적인 훈련과 우수한 성능으로 이어져 더 널리 채택되는 결과를 낳았지만, 정보 흐름을 제어하여 심층 네트워크를 훈련한다는 핵심 아이디어를 처음 제시한 것은 Highway Network였다.

아래 표는 ResNet과 그 파생 아키텍처들이 모델의 표현력을 어떤 차원에서 확장하려 했는지를 요약하여 보여준다. 이는 아키텍처 설계 패러다임의 진화 과정을 명확하게 드러낸다.

| 아키텍처        | 핵심 차원                | 정보 결합 방식       | 장점                                  | 단점                              |
| --------------- | ------------------------ | -------------------- | ------------------------------------- | --------------------------------- |
| **ResNet**      | 깊이 (Depth)             | 덧셈 (Addition)      | 깊이의 한계 극복, 최적화 용이         | 매우 깊어질 경우 특징 재사용 감소 |
| **Wide ResNet** | 너비 (Width)             | 덧셈 (Addition)      | 더 적은 깊이로 높은 성능, 학습 효율성 | 파라미터 수 증가                  |
| **ResNeXt**     | 카디널리티 (Cardinality) | 덧셈 (Addition)      | 높은 파라미터 효율성, 쉬운 설계       | 그룹 컨볼루션 구현에 의존         |
| **DenseNet**    | 연결성 (Connectivity)    | 연결 (Concatenation) | 극대화된 특징 재사용, 파라미터 효율성 | 높은 메모리 사용량                |


잔차 연결의 영향력은 컨볼루션 신경망(CNN)의 영역을 넘어, 자연어 처리(NLP) 분야에 혁명을 일으킨 트랜스포머(Transformer) 아키텍처의 근간을 이루는 핵심 요소로 확장되었다. 순환(recurrence) 구조 없이 오직 어텐션(attention) 메커니즘만으로 시퀀스 데이터의 장거리 의존성을 포착하는 트랜스포머의 능력은, 수십, 수백 개의 블록을 쌓아 올린 초심층 구조에 의해 뒷받침된다. 그리고 이 깊이를 가능하게 한 일등 공신이 바로 잔차 연결이다.


2017년 "Attention Is All You Need" 논문에서 제안된 트랜스포머는 인코더와 디코더 스택으로 구성되며, 각 스택은 동일한 구조의 블록을 여러 개(원 논문에서는 6개) 쌓아 만든다.39 각 블록의 핵심 연산은 '멀티-헤드 셀프-어텐션(Multi-Head Self-Attention)'과 '위치별 피드포워드 네트워크(Position-wise Feed-Forward Network)'이다.

이러한 블록들을 수십, 수백 개 깊이로 쌓아 거대 언어 모델(LLM)을 구축할 때, ResNet이 직면했던 것과 동일한 문제, 즉 기울기 소실과 정보 손실 문제가 발생한다. 트랜스포머는 이 문제를 해결하기 위해 ResNet의 아이디어를 그대로 차용했다. 각 블록 내의 모든 주요 연산(서브-레이어) 주위에 잔차 연결을 배치하여, 정보와 기울기가 깊은 네트워크를 통해 원활하게 흐를 수 있는 경로를 확보한 것이다.18 사실상, 잔차 연결이 없다면 오늘날의 초거대 트랜스포머 모델들의 안정적인 훈련은 불가능하다고 해도 과언이 아니다.18


트랜스포머의 인코더와 디코더 블록을 더 자세히 들여다보면, 잔차 연결이 레이어 정규화(Layer Normalization)와 한 쌍으로 동작하는 것을 볼 수 있다. 이를 'Add & Norm' 단계라고 부른다.41

트랜스포머 인코더 블록의 일반적인 구조는 다음과 같다.

1. **입력 ($x$)**
2. **Multi-Head Self-Attention 서브-레이어**
3. **Add & Norm:**
   - **Add (잔차 연결):** 어텐션 서브-레이어의 출력과 서브-레이어의 입력 $x$를 더한다.
   - **Norm (레이어 정규화):** 합산된 결과에 레이어 정규화를 적용한다.
4. **Position-wise Feed-Forward Network 서브-레이어**
5. **Add & Norm:**
   - **Add (잔차 연결):** 피드포워드 서브-레이어의 출력과 그 서브-레이어의 입력을 더한다.
   - **Norm (레이어 정규화):** 합산된 결과에 레이어 정규화를 적용한다.
6. **출력**

디코더 블록은 인코더의 출력을 어텐션하는 '인코더-디코더 어텐션' 서브-레이어가 추가되어 총 세 번의 'Add & Norm' 단계를 거치는 점을 제외하면 동일한 원리로 동작한다.41

이 'Add & Norm' 구조에서 각 구성 요소의 역할은 명확하고 상호 보완적이다.

- **Add (잔차 연결):**
  - **정보 보존:** 어텐션이나 피드포워드 네트워크와 같은 복잡한 비선형 변환을 거치면서 입력 정보의 일부가 손실될 수 있다. 잔차 연결은 원본 입력 `x`를 직접 더해줌으로써 이러한 정보 손실을 방지하고, 모델이 변환을 통해 얻은 새로운 정보와 원본 정보를 모두 활용할 수 있게 한다.43
  - **기울기 고속도로:** 역전파 시, 기울기가 복잡한 서브-레이어를 우회하여 직접 하위 계층으로 흐를 수 있는 '고속도로'를 제공한다. 이는 수십 개의 블록을 통과하면서도 기울기가 소실되지 않고 안정적으로 전파되도록 보장하는 핵심적인 역할을 한다.43
- **Norm (레이어 정규화):**
  - **학습 안정화:** 잔차 연결은 덧셈 연산을 통해 계층을 거치면서 출력 벡터의 크기(magnitude)가 계속해서 커질 수 있는 잠재적 불안정성을 내포한다. 레이어 정규화는 각 계층의 출력 분포를 평균 0, 분산 1로 정규화하여 이러한 값의 폭주를 막고, 학습 과정을 안정시키는 역할을 한다.
  - **최적화 지형 평탄화:** 레이어 정규화는 손실 함수의 지형(landscape)을 더 부드럽게 만들어, 경사 하강법 기반의 옵티마이저가 더 빠르고 안정적으로 최적점에 수렴하도록 돕는다.


흥미롭게도, 트랜스포머에서도 레이어 정규화의 위치에 대한 논쟁이 ResNet의 Pre-activation 논쟁과 유사하게 전개되었다.

- **Post-LN (원 논문 방식):** $LayerNorm(x + Sublayer(x))$ 48
  - 잔차 연결 이후에 정규화를 적용한다. 이 방식은 깊은 모델에서 훈련 불안정성과 기울기 소실을 유발할 수 있다는 것이 이론적, 경험적으로 밝혀졌다. 기울기가 역전파될 때 레이어 정규화의 미분을 먼저 통과해야 하므로 신호가 약화될 수 있다.48
- **Pre-LN (현대 모델 방식):** $x + Sublayer(LayerNorm(x))$ 48
  - 서브-레이어에 들어가기 전에 입력을 먼저 정규화한다. 이 방식은 잔차 경로가 깨끗하게 유지되어 기울기 흐름이 더 안정적이다. 따라서 학습 초기 단계가 더 안정적이고, 학습률 워밍업(learning rate warm-up)의 필요성을 줄여준다. 이로 인해 BERT 이후의 대부분의 현대 트랜스포머 모델들은 Pre-LN 방식을 표준으로 채택하고 있다.

이처럼 잔차 연결과 레이어 정규화의 조합 및 그 미세한 배치 순서는 현대 딥러닝 아키텍처의 성능과 안정성을 결정하는 매우 중요한 설계 요소로 자리 잡았다. 이는 잔차 연결이 단순히 하나의 '트릭'이 아니라, 정보와 기울기의 흐름을 제어하는 근본적인 아키텍처 설계 원리임을 다시 한번 확인시켜 준다.


잔차 연결은 의심할 여지 없이 딥러닝 역사상 가장 영향력 있는 아키텍처 혁신 중 하나이다. 그러나 어떠한 기술도 만능일 수는 없으며, 잔차 연결 역시 그 실질적인 효용의 이면에 존재하는 한계와 새로운 관점에서의 비판에 직면하고 있다. 최근 연구 동향은 잔차 연결의 역할을 재평가하고, 특정 과업에서는 오히려 해가 될 수 있다는 가능성을 제기하며 새로운 대안을 모색하고 있다.


먼저 잔차 연결의 장점과 단점을 종합적으로 정리할 필요가 있다.

**장점 요약:**

- **초심층 네트워크 훈련 가능:** 기울기 소실 문제를 효과적으로 완화하여 이전에는 불가능했던 수백, 수천 계층의 네트워크 훈련을 가능하게 했다.4
- **빠른 수렴 및 최적화 용이성:** 손실 함수의 지형을 완만하게 만들어 옵티마이저가 더 쉽게 최적점을 찾도록 돕고, 훈련 수렴 속도를 높인다.51
- **성능 향상:** 깊이로부터 얻는 표현력 증대를 실제 성능 향상으로 연결시켰다.52
- **모듈성 및 범용성:** 잔차 블록이라는 모듈화된 설계를 통해 아키텍처 구성이 용이하며, CNN, 트랜스포머 등 다양한 모델에 보편적으로 적용 가능한 핵심 원리로 자리 잡았다.52

**단점 및 한계:**

- **계산 비용 및 메모리 오버헤드:** 잔차 연결 자체는 비용이 거의 없지만, 이로 인해 가능해진 초심층 네트워크는 막대한 계산 자원과 메모리를 요구한다. 이는 모델의 배포와 활용에 제약이 될 수 있다.51
- **아키텍처 복잡성 증가:** 단순한 순차적 계층 쌓기에 비해, 차원 조절을 위한 투영 지름길 등을 고려해야 하므로 설계가 복잡해진다.52
- **과적합 가능성:** 매우 깊은 네트워크는 모델의 용량(capacity)이 매우 크기 때문에, 데이터가 충분하지 않은 경우 과적합에 취약할 수 있다.51
- **정보 흐름의 비효율성:** DenseNet과 같은 아키텍처와 비교할 때, ResNet의 덧셈 기반 정보 결합은 이전 계층의 특징을 명시적으로 보존하지 않으므로 특징 재사용 측면에서 비효율적일 수 있다.


앞서 '암묵적 앙상블' 관점에서 잔차 연결이 기울기 소실 문제를 '회피'한다고 설명했지만, 또 다른 이론적 해석은 '산산조각난 기울기(Shattered Gradients)' 문제 해결에 초점을 맞춘다.16

Balduzzi 등의 연구(2017)에 따르면, 잔차 연결이 없는 매우 깊은 피드포워드 네트워크에서는 역전파되는 기울기가 계층을 거치면서 점차 신호 간의 상관관계를 잃고 백색 소음(white noise)처럼 변하는 경향이 있다. 이러한 '산산조각난' 기울기는 유의미한 학습 신호를 담고 있지 않아, 가중치 업데이트가 거의 무작위적인 방향으로 이루어지게 만들어 학습을 방해한다.

잔차 연결은 이 문제에 대한 해결책을 제공한다. 지름길을 통해 상위 계층의 기울기가 하위 계층에 직접 더해짐으로써, 기울기 신호에 강력한 '공간적 구조(spatial structure)' 또는 상관관계를 부여한다. 이로 인해 기울기가 백색 소음으로 퇴화하는 것을 막고, 깊은 계층을 통과하더라도 유의미한 학습 방향을 유지할 수 있게 된다. 이는 기울기 값의 크기(magnitude) 문제인 기울기 소실과는 다른, 기울기 신호의 '구조(structure)' 관점에서 잔차 연결의 중요성을 설명하는 보완적인 시각이다.


최근까지 잔차 연결은 판별 모델(discriminative models, 예: 분류기)과 생성 모델(generative models)을 가리지 않고 거의 모든 심층 아키텍처의 표준 구성 요소로 여겨져 왔다. 그러나 "Residual Connections Harm Generative Representation Learning" (2024)과 같은 최신 연구는 이러한 통념에 도전하며, 특정 학습 패러다임에서는 잔차 연결이 오히려 해가 될 수 있다는 놀라운 주장을 제기한다.55

이 연구의 핵심 주장은 잔차 연결의 본질적인 속성, 즉 $y = F(x) + x$라는 정보 보존 메커니즘이 '새로운 추상적 표현'을 학습해야 하는 생성적 표현 학습(generative representation learning)에서는 방해가 될 수 있다는 것이다.

- **판별 모델에서의 이점:** 분류와 같은 판별 과업에서는 입력 $x$의 원본 정보를 최대한 보존하면서 분류에 유용한 특징을 추가하는 것이 유리하다. $+x$ 항은 이러한 정보 보존을 보장한다.
- **생성 모델에서의 단점:** 반면, 마스크된 오토인코더(Masked Autoencoders, MAE)나 확산 모델(Diffusion Models)과 같은 생성적 자기 지도 학습(self-supervised learning)의 목표는 손상된 입력 $x$로부터 완전히 새로운, 더 높은 수준의 추상적이고 의미론적인 표현 $y$를 생성하거나 복원하는 것이다. 이 경우, $+x$ 항은 얕은 계층의 저수준 특징이 깊은 계층까지 계속해서 '메아리'처럼 남아있게 만든다. 이는 네트워크가 진정으로 새롭고 추상적인 특징을 배우는 것을 방해하는 일종의 '족쇄' 역할을 할 수 있다.55


이 문제를 해결하기 위해 연구팀은 네트워크의 깊이가 깊어짐에 따라 항등 지름길의 영향력을 점진적으로 감소시키는 **'감쇠된 항등 지름길'**이라는 간단하면서도 효과적인 방법을 제안했다.55
$$
y = F(x) + \alpha \cdot x
$$
여기서 스케일링 팩터 $α$는 네트워크의 깊이에 따라 $1$에서부터 $0$에 가까운 값으로 점차 감소한다.

이 설계는 네트워크가 다음과 같이 동작하도록 유도한다.

1. **초기 계층 ($α ≈ 1$):** 일반적인 잔차 연결처럼 동작하여 안정적인 기울기 흐름을 보장하고 훈련 초기 단계의 안정성을 확보한다.
2. **후기 계층 ($α → 0$):** 점차 잔차 연결의 영향력이 줄어들고, 일반적인 피드포워드 네트워크처럼 동작한다. 이를 통해 네트워크는 저수준 특징의 메아리에서 벗어나 더 높은 수준의 추상적인 의미론적 특징을 자유롭게 학습할 수 있다.

실제로 이 방법을 MAE와 확산 모델에 적용한 결과, 의미론적 특징 학습 능력이 크게 향상되어 ImageNet-1K에서의 선형 평가(linear probing) 및 K-NN 분류 정확도가 극적으로 개선되었음을 보였다.55


이러한 연구는 잔차 연결이 더 이상 모든 상황에 적용되는 고정된 아키텍처 요소가 아님을 시사한다. 미래의 딥러닝 아키텍처는 과업의 종류(판별 vs. 생성)와 학습 패러다임(지도학습 vs. 자기 지도 학습)에 따라 정보 흐름의 방식을 동적으로 조절하는 방향으로 발전할 것이다. LAuReL(Learned Augmented Residual Layer)과 같이 잔차 경로 자체를 학습 가능한 모듈로 만드는 연구 56나, 데이터에 따라 잔차 연결의 강도를 조절하는 적응형 잔차 연결(adaptive residual connections) 52에 대한 연구는 이러한 방향성의 연장선에 있다. 잔차 연결의 원리는 유지하되, 그 적용 방식은 더욱 유연하고 지능적으로 진화할 것이다.


잔차 연결은 심층 신경망의 '깊이'라는 오랜 난제를 해결하며 딥러닝의 새로운 시대를 연 기념비적인 발명이다. 그 영향력은 단순히 하나의 아키텍처를 넘어, 현대 딥러닝 모델을 설계하고 이해하는 방식 자체를 근본적으로 바꾸어 놓았다.

**잔차 연결의 핵심 기여 요약:**

1. **초심층 네트워크 훈련의 실현:** 성능 저하 문제와 기울기 소실 문제를 효과적으로 완화함으로써, 이전에는 상상할 수 없었던 수백, 수천 계층의 초심층 신경망을 안정적으로 훈련하는 길을 최초로 열었다. 이는 딥러닝 모델의 표현력을 한 차원 높은 수준으로 끌어올리는 계기가 되었다.
2. **현대 아키텍처의 표준 부품:** '지름길 연결'이라는 단순하면서도 강력한 아이디어는 CNN을 넘어 자연어 처리의 혁명을 이끈 트랜스포머 아키텍처의 필수 구성 요소로 자리 잡았다. 오늘날 대부분의 최첨단 딥러닝 모델은 잔차 연결의 원리를 어떤 형태로든 내장하고 있다.
3. **아키텍처 설계 패러다임의 확장:** ResNet의 등장은 깊이에 대한 집착에서 벗어나, 너비(Wide ResNet), 카디널리티(ResNeXt), 연결성(DenseNet) 등 모델의 표현력을 증대시키는 다양한 차원을 탐구하는 후속 연구들을 촉발시켰다. 이는 딥러닝 아키텍처 설계의 새로운 지평을 열었다.

**향후 연구 방향과 잔차 연결의 미래:**

잔차 연결은 발표된 지 10년 가까이 지난 지금도 여전히 활발한 연구 주제이다. 그 역할에 대한 이해가 깊어짐에 따라, 이제는 '잔차 연결을 사용할 것인가'의 문제를 넘어 '어떻게 최적으로 사용할 것인가'의 문제로 초점이 이동하고 있다.

- **이론적 기반의 심화:** 잔차 연결이 최적화 지형에 미치는 영향, 암묵적 앙상블 효과의 본질 등 그 작동 원리에 대한 보다 근본적인 이론적 이해를 위한 연구가 계속될 것이다.
- **적응형 및 학습형 잔차 연결:** 고정된 $+x$ 구조에서 벗어나, 과업, 데이터, 심지어는 개별 샘플에 따라 잔차 연결의 강도나 형태를 동적으로 조절하는 적응형(adaptive) 및 학습형(learnable) 잔차 연결에 대한 연구가 더욱 중요해질 것이다. 생성 모델에서의 '감쇠된 지름길'은 이러한 흐름의 시작을 알린다.
- **새로운 패러다임과의 융합:** 자기 지도 학습, 그래프 신경망, 미분 가능 프로그래밍 등 새로운 딥러닝 패러다임에 최적화된 새로운 형태의 정보 전달 경로를 탐색하는 과정에서 잔차 연결의 원리는 계속해서 변주되고 재창조될 것이다.

결론적으로, 잔차 연결은 딥러닝의 역사에서 가장 우아하고 영향력 있는 아이디어 중 하나로 기록될 것이다. 이는 단순히 깊이의 장벽을 허문 기술을 넘어, 정보와 기울기가 네트워크 내에서 어떻게 흘러야 하는지에 대한 근본적인 통찰을 제공했다. 앞으로도 그 핵심 철학은 다양한 형태로 진화하며, 더욱 강력하고 효율적인 미래 딥러닝 아키텍처를 구축하는 데 있어 변치 않는 주춧돌 역할을 할 것이다.


1. arXiv:1512.03385v1 [cs.CV] 10 Dec 2015, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1512.03385
2. [1512.03385] Deep Residual Learning for Image Recognition - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/1512.03385
3. An Overview of ResNet Architecture and Its Variants - Built In, 8월 24, 2025에 액세스, https://builtin.com/artificial-intelligence/resnet-architecture
4. Residual Networks (ResNet) - Deep Learning - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/deep-learning/residual-networks-resnet-deep-learning/
5. Deep Residual Learning for Image Recognition - The Computer Vision Foundation, 8월 24, 2025에 액세스, https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/He_Deep_Residual_Learning_CVPR_2016_paper.pdf
6. en.wikipedia.org, 8월 24, 2025에 액세스, [https://en.wikipedia.org/wiki/Vanishing_gradient_problem#:~:text=Residual%20connection,-Residual%20connections%2C%20or&text=%2C%20where%20the%20identity%20matrix%20do,flows%20through%20the%20residual%20connections.&text=at%20layer%20is-,.,vanish%20in%20arbitrarily%20deep%20networks.](https://en.wikipedia.org/wiki/Vanishing_gradient_problem#:~:text=Residual connection,-Residual connections%2C or&text=%2C where the identity matrix do,flows through the residual connections.&text=at layer is-,.,vanish in arbitrarily deep networks.)
7. Vanishing gradient problem - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Vanishing_gradient_problem
8. www.digitalocean.com, 8월 24, 2025에 액세스, [https://www.digitalocean.com/community/tutorials/vanishing-gradient-problem#:~:text=The%20vanishing%20gradient%20problem%20is%20a%20fundamental%20challenge%20in%20training,even%20stopping%20the%20training%20process.](https://www.digitalocean.com/community/tutorials/vanishing-gradient-problem#:~:text=The vanishing gradient problem is a fundamental challenge in training,even stopping the training process.)
9. Backpropagation in Neural Network - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/backpropagation-in-neural-network/
10. Vanishing Gradient Problem in Deep Learning: Explained - DigitalOcean, 8월 24, 2025에 액세스, https://www.digitalocean.com/community/tutorials/vanishing-gradient-problem
11. Vanishing and Exploding Gradients Problems in Deep Learning - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/deep-learning/vanishing-and-exploding-gradients-problems-in-deep-learning/
12. What is Vanishing Gradient Problem? - Great Learning, 8월 24, 2025에 액세스, https://www.mygreatlearning.com/blog/the-vanishing-gradient-problem/
13. Deep Residual Learning for Image Recognition - Encyclopedia.pub, 8월 24, 2025에 액세스, https://encyclopedia.pub/entry/27415
14. Deep Residual Learning for Image Recognition: A Survey - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/363337225_Deep_Residual_Learning_for_Image_Recognition_A_Survey
15. Deep Residual Learning for Image Recognition - The Engineering Projects, 8월 24, 2025에 액세스, https://www.theengineeringprojects.com/2023/12/deep-residual-learning-for-image-recognition.html
16. What is Residual Connection?. A technique for training very deep… | by Wanshun Wong | TDS Archive | Medium, 8월 24, 2025에 액세스, https://medium.com/data-science/what-is-residual-connection-efb07cab0d55
17. 8.6. Residual Networks (ResNet) and ResNeXt - Dive into Deep Learning, 8월 24, 2025에 액세스, http://d2l.ai/chapter_convolutional-modern/resnet.html
18. Residual neural network - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Residual_neural_network
19. What is Residual Connection? - Towards Data Science, 8월 24, 2025에 액세스, https://towardsdatascience.com/what-is-residual-connection-efb07cab0d55/
20. The Reversible Residual Network: Backpropagation Without Storing Activations - NIPS, 8월 24, 2025에 액세스, http://papers.neurips.cc/paper/6816-the-reversible-residual-network-backpropagation-without-storing-activations.pdf
21. Deep Residual Learning for Image Recognition - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/286512696_Deep_Residual_Learning_for_Image_Recognition
22. Identity Mappings in Deep Residual Networks, 8월 24, 2025에 액세스, https://arxiv.org/abs/1603.05027
23. (PDF) Identity Mappings in Deep Residual Networks (2016) | Kaiming He | 6179 Citations, 8월 24, 2025에 액세스, https://scispace.com/papers/identity-mappings-in-deep-residual-networks-qa2s6i6ebg
24. Identity Mappings in Deep Residual Networks - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/319770414_Identity_Mappings_in_Deep_Residual_Networks
25. Wide Residual Networks, 8월 24, 2025에 액세스, https://arxiv.org/abs/1605.07146
26. Wide Residual Networks - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1605.07146
27. [1611.05431] Aggregated Residual Transformations for Deep Neural Networks - ar5iv, 8월 24, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/1611.05431
28. Aggregated Residual Transformations for Deep Neural Networks | Request PDF - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/310441005_Aggregated_Residual_Transformations_for_Deep_Neural_Networks
29. Aggregated Residual Transformations for Deep ... - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content_cvpr_2017/papers/Xie_Aggregated_Residual_Transformations_CVPR_2017_paper.pdf
30. [PDF] Densely Connected Convolutional Networks - Semantic Scholar, 8월 24, 2025에 액세스, https://www.semanticscholar.org/paper/Densely-Connected-Convolutional-Networks-Huang-Liu/5694e46284460a648fe29117cbc55f6c9be3fa3c
31. [2001.02394] Convolutional Networks with Dense Connectivity - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2001.02394
32. Densely Connected Convolutional Networks, 8월 24, 2025에 액세스, https://arxiv.org/abs/1608.06993
33. (PDF) Densely Connected Convolutional Networks - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/319770123_Densely_Connected_Convolutional_Networks
34. Dense Convolutional Network and Its Application in Medical Image Analysis - PMC, 8월 24, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC9060995/
35. Highway Networks, the first working really deep feedforward neural networks with hundreds of layers, 8월 24, 2025에 액세스, https://people.idsia.ch/~juergen/highway-networks.html
36. Highway network - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Highway_network
37. Highway Networks - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/1505.00387
38. Highway Networks - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1505.00387
39. Attention Is All You Need - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Attention_Is_All_You_Need
40. Attention is All you Need - NIPS, 8월 24, 2025에 액세스, https://papers.nips.cc/paper/7181-attention-is-all-you-need
41. Attention Is All You Need - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/1706.03762v7
42. The Illustrated Transformer – Jay Alammar – Visualizing machine ..., 8월 24, 2025에 액세스, https://jalammar.github.io/illustrated-transformer/
43. Why Are Residual Connections Important in Transformer Architectures? | Baeldung on Computer Science, 8월 24, 2025에 액세스, https://www.baeldung.com/cs/transformer-networks-residual-connections
44. Do Transformers Really Need Residual Connections? : r/deeplearning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/deeplearning/comments/1gky088/do_transformers_really_need_residual_connections/
45. LLM Transformer Model Visually Explained - Polo Club of Data Science, 8월 24, 2025에 액세스, https://poloclub.github.io/transformer-explainer/
46. 11.7. The Transformer Architecture — Dive into Deep Learning 1.0.3 documentation, 8월 24, 2025에 액세스, https://d2l.ai/chapter_attention-mechanisms-and-transformers/transformer.html
47. Attention in Transformers: residual connection layer, a shortcut that makes it work - Medium, 8월 24, 2025에 액세스, https://medium.com/@mariaprokofieva/attention-in-transformers-residual-connection-layer-a-shortcut-that-makes-it-work-165b52566167
48. On Layer Normalization in the Transformer Architecture - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2002.04745
49. On Layer Normalizations and Residual Connections in Transformers - OpenReview, 8월 24, 2025에 액세스, https://openreview.net/attachment?id=oDDqFpK5Ru1&name=pdf
50. Peri-LN: Revisiting Layer Normalization in the Transformer Architecture - arXiv, 8월 24, 2025에 액세스, [https://arxiv.org/pdf/2502.02732?](https://arxiv.org/pdf/2502.02732)
51. What are the advantages and disadvantages of using a residual neural network?, 8월 24, 2025에 액세스, [https://massedcompute.com/faq-answers/?question=What%20are%20the%20advantages%20and%20disadvantages%20of%20using%20a%20residual%20neural%20network?](https://massedcompute.com/faq-answers/?question=What+are+the+advantages+and+disadvantages+of+using+a+residual+neural+network?)
52. Mastering Residual Connections: Enhancing Neural Networks for Optimal Performance, 8월 24, 2025에 액세스, https://www.lunartech.ai/blog/mastering-residual-connections-enhancing-neural-networks-for-optimal-performance
53. [PDF] Deep Residual Learning for Image Recognition - Semantic Scholar, 8월 24, 2025에 액세스, https://www.semanticscholar.org/paper/Deep-Residual-Learning-for-Image-Recognition-He-Zhang/2c03df8b48bf3fa39054345bafabfeff15bfd11d
54. What is Residual Connection? | Towards Data Science, 8월 24, 2025에 액세스, https://towardsdatascience.com/what-is-residual-connection-efb07cab0d55
55. Residual Connections Harm Generative Representation Learning - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2404.10947v4
56. LAuReL: Learned Augmented Residual Layer - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2411.07501v2




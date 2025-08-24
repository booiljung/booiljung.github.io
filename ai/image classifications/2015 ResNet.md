# ResNet (Deep Residual Learning, 2015)
[이미지 분류 (Image Classifications)](./index.md)



심층 신경망(Deep Neural Networks, DNNs)의 발전은 네트워크의 '깊이(depth)'가 특징 표현(feature representation)의 질을 결정하는 핵심 요소라는 전제에서 시작되었다.1 네트워크의 계층이 깊어짐에 따라, 초기 계층에서는 이미지의 엣지(edge)나 질감(texture)과 같은 저수준(low-level) 특징을 학습하고, 후반 계층으로 갈수록 객체의 부분이나 전체와 같은 고수준(high-level)의 추상적 특징을 학습하게 된다.1 2012년 AlexNet이 ILSVRC(ImageNet Large Scale Visual Recognition Challenge)에서 우승한 이후, VGGNet과 같은 후속 모델들은 지속적으로 더 깊은 구조를 채택하며 이미지 인식 성능의 한계를 경신해왔다.4 이는 깊이가 모델의 표현력을 높이는 데 결정적인 역할을 한다는 강력한 경험적 증거가 되었다.


이론적으로, 더 깊은 네트워크는 최소한 더 얕은 네트워크만큼의 성능을 보장해야 한다. 추가된 계층들이 단순히 입력을 그대로 출력하는 항등 함수(identity function)를 학습한다면, 전체 네트워크는 더 얕은 네트워크와 동일한 기능을 수행할 수 있기 때문이다.1 따라서 네트워크의 깊이가 증가함에 따라 훈련 오류(training error)는 감소하거나 최소한 유지되어야 한다.

그러나 실제 실험에서는 이론과 상반되는 현상이 관찰되었다. 네트워크의 깊이가 일정 수준을 넘어서자, 훈련 오류와 테스트 오류가 모두 오히려 증가하는 '성능 저하(degradation)' 문제가 발생한 것이다.1 이는 단순히 파라미터 수가 많아져 발생하는 과적합(overfitting)과는 근본적으로 다른 문제였다. 성능 저하 문제는 모델의 표현력이 부족해서가 아니라, 깊어진 모델을 효과적으로 최적화(optimization)하는 것 자체가 어렵다는 본질적인 난제를 드러냈다.9


성능 저하 문제는 종종 심층망 훈련의 고전적인 난제인 '기울기 소실/폭주(vanishing/exploding gradients)' 문제와 혼동되곤 한다. 그러나 ResNet 논문의 저자들은 이 두 문제를 명확히 구분하고, 자신들의 목표가 후자가 아닌 전자를 해결하는 것임을 분명히 했다.10

기울기 소실 문제는 역전파(backpropagation) 과정에서 오차 기울기가 연쇄 법칙(chain rule)에 따라 여러 계층을 거치며 반복적으로 곱해지면서 그 값이 지수적으로 작아져 거의 0에 수렴하는 현상을 말한다.2 이로 인해 네트워크의 초기 계층들은 가중치 업데이트가 거의 이루어지지 않아 학습이 정체된다. 이 문제는 정규화된 가중치 초기화(normalized initialization), ReLU와 같은 비포화 활성화 함수(non-saturating activation function), 그리고 특히 배치 정규화(Batch Normalization, BN)의 도입으로 상당 부분 완화되었다.2

이러한 문제 해결의 연쇄 과정을 이해하는 것은 ResNet의 진정한 기여를 파악하는 데 중요하다. 심층 학습의 발전은 다음과 같은 인과 관계를 따른다. 첫째, 깊이의 필요성이 대두되었다. 둘째, 이로 인해 기울기 소실 문제가 부상했다. 셋째, ReLU와 배치 정규화 같은 기법들이 이 문제를 해결하여 더 깊은 네트워크의 안정적인 훈련을 가능하게 했다. 넷째, 안정성이 확보되자 연구자들은 네트워크를 20계층에서 56계층 등으로 더욱 깊게 쌓았고, 바로 이 지점에서 새로운 병목 현상인 성능 저하 문제가 드러났다.1 즉, 문제는 훈련의 '불안정성'에서 '최적화의 어려움'으로 전환된 것이다. ResNet의 저자들은 논문에서 배치 정규화를 사용함으로써 순방향 및 역방향 신호의 소실을 막았다고 명시하며, 자신들이 해결하고자 하는 문제가 기울기 소실이 아님을 강조했다.10 ResNet의 잔차 학습은 바로 이 새롭게 드러난 최적화 문제를 정면으로 겨냥한 독창적인 해결책이었다.



성능 저하 문제를 해결하기 위해 ResNet은 학습 목표 자체를 근본적으로 재정의했다. 기존의 심층 신경망 블록은 입력 $x$를 받아 목표하는 이상적인 함수(underlying mapping) $H(x)$를 직접 학습하도록 설계되었다. 하지만 ResNet은 블록이 $H(x)$ 자체를 학습하는 대신, 입력 $x$와의 차이, 즉 '잔차(residual)'에 해당하는 함수 $F(x) = H(x) - x$를 학습하도록 유도한다.1 그리고 최종 출력은 학습된 잔차 $F(x)$에 입력 $x$를 더하는 형태인 $H(x) = F(x) + x$로 재구성된다. 이 구조는 입력 $x$를 출력단에 직접 더해주는 '지름길 연결(shortcut connection)' 또는 '스킵 연결(skip connection)'을 통해 구현된다.4


ResNet의 핵심 가설은 잔차 함수 $F(x)$를 최적화하는 것이 원래 함수 $H(x)$를 직접 최적화하는 것보다 더 쉽다는 것이다.1 이 가설의 타당성은 극단적인 경우를 통해 쉽게 이해할 수 있다. 만약 특정 블록에서 최적의 변환이 항등 함수, 즉 $H(x) = x$인 상황을 가정해보자. 기존의 프레임워크에서는 여러 개의 비선형 계층(예: 컨볼루션, ReLU)을 쌓아 항등 변환을 근사해야 하는데, 이는 매우 어려운 최적화 문제이다.5 반면, 잔차 학습 프레임워크에서는 단순히 잔차 함수 $F(x)$를 0으로 만들기만 하면 된다. 이는 해당 블록 내 계층들의 가중치를 0에 가깝게 만드는 방향으로 학습하면 되므로 훨씬 용이하다.1 이처럼 잔차 학습은 네트워크가 불필요한 변환을 '학습하지 않도록' 하는 길을 열어주어 최적화를 돕는다.


잔차 학습의 진정한 위력은 역전파 과정에서 명확히 드러난다. 손실 함수 $L$에 대한 어떤 잔차 블록의 입력 $x$의 기울기(gradient)는 연쇄 법칙에 의해 다음과 같이 표현된다.
$$
\frac{\partial L}{\partial x} = \frac{\partial L}{\partial H} \frac{\partial H}{\partial x}
$$
여기서 $H(x) = F(x) + x$이므로, $H(x)$를 $x$에 대해 미분하면 덧셈의 미분 법칙에 따라 다음과 같이 전개된다.
$$
\frac{\partial H}{\partial x} = \frac{\partial F(x)}{\partial x} + \frac{\partial x}{\partial x} = \frac{\partial F(x)}{\partial x} + 1
$$
이 결과를 전체 기울기 식에 대입하면 최종적으로 다음의 중요한 관계식을 얻는다.15
$$
\frac{\partial L}{\partial x} = \frac{\partial L}{\partial H} \left( \frac{\partial F(x)}{\partial x} + 1 \right) = \frac{\partial L}{\partial H} \frac{\partial F(x)}{\partial x} + \frac{\partial L}{\partial H}
$$
이 수식은 ResNet의 작동 원리를 설명하는 핵심이다. 최종 기울기 $\frac{\partial L}{\partial x}$는 두 항의 합으로 구성된다. 첫 번째 항 $\frac{\partial L}{\partial H} \frac{\partial F(x)}{\partial x}$는 잔차 함수 $F(x)$를 통과하는 경로의 기울기를, 두 번째 항 $\frac{\partial L}{\partial H}$는 지름길 연결을 통해 아무런 변형 없이 직접 전달되는 기울기를 의미한다. 여기서 미분 식에 포함된 $+1$ 항은 상위 계층의 기울기 $\frac{\partial L}{\partial H}$가 아무런 감쇠 없이 하위 계층으로 직접 흐를 수 있는 '기울기 고속도로(gradient highway)'를 수학적으로 구현한 것이다.

이 구조는 성능 저하 문제를 직접적으로 해결한다. 심층 일반 신경망(plain network)에서 기울기는 여러 비선형 변환을 거치며 역전파되는 동안 신호가 왜곡되거나 소실되어 손실 지형(loss landscape)이 복잡하고 험난해진다(shattered gradients).11 이로 인해 경사 하강법(SGD)과 같은 최적화 알고리즘이 효과적인 경로를 찾기 어려워진다. 하지만 ResNet의 지름길 연결은 

$F(x)$ 경로의 기울기 기여분이 매우 작아지거나 잡음이 많아지더라도, 상위 계층의 기울기가 항상 하위 계층으로 직접 전달되는 안전장치 역할을 한다.9 이 덕분에 학습이 완전히 정체되는 것을 막고, 전체적인 손실 지형을 더 완만하게 만들어 최적화가 용이해지는 것이다. 이것이 ResNet이 "최적화하기 더 쉽다"고 평가받는 핵심적인 이유이다.1



ResNet을 구성하는 기본 단위인 잔차 블록(residual block)은 일반적으로 여러 개의 컨볼루션 계층(Convolutional Layers), 배치 정규화(Batch Normalization), 그리고 비선형 활성화 함수(Non-linear Activation Function)로 이루어진다.6 특히 VGGNet의 설계 철학을 계승하여 공간적 특징 추출을 위해 주로 3x3 크기의 컨볼루션 필터를 사용하며, 안정적인 학습을 위해 각 컨볼루션 계층 뒤에 배치 정규화를 적용한다. 활성화 함수로는 ReLU(Rectified Linear Unit)가 널리 사용된다.6


기본 블록은 상대적으로 얕은 네트워크인 ResNet-18과 ResNet-34 모델에 사용되는 구조이다.19 이 블록은 순차적으로 연결된 두 개의 3x3 컨볼루션 계층으로 구성된다. 각 컨볼루션 계층 다음에는 배치 정규화가 적용되며, 첫 번째 컨볼루션 후에는 ReLU 활성화 함수가 뒤따른다. 두 번째 컨볼루션과 배치 정규화가 끝난 후, 그 결과는 지름길 연결을 통해 전달된 입력 $x$와 합산되고, 이 합산된 결과에 최종적으로 ReLU 활성화 함수가 적용된다. 기본 블록의 중요한 특징은 입력과 출력의 채널(차원) 수가 동일하게 유지된다는 점이다.7


병목 블록은 ResNet-50, ResNet-101, ResNet-152와 같이 더욱 깊은 네트워크의 계산 효율성을 극대화하기 위해 도입된 핵심적인 구조적 혁신이다.7 이 구조는 단순히 최적화를 넘어, 50계층 이상의 극도로 깊은 네트워크를 현실적으로 구현 가능하게 만든 원동력이다. 병목 블록이 없다면, 깊은 계층에서 채널 수가 증가함에 따라 기본 블록의 파라미터 수와 연산량(FLOPs)이 감당할 수 없을 정도로 커지게 된다.

병목 블록은 '압축, 처리, 복원(compress, process, decompress)'이라는 효율적인 설계 철학을 따르며, 세 개의 컨볼루션 계층으로 구성된다:

1. **1x1 컨볼루션 (차원 축소):** 첫 번째 1x1 컨볼루션 계층은 입력 피처 맵의 채널 수를 1/4로 줄여(예: 256채널 -> 64채널) 정보의 '병목'을 생성한다. 이는 계산 비용이 가장 높은 3x3 컨볼루션의 연산량을 극적으로 감소시키는 역할을 한다.18
2. **3x3 컨볼루션:** 계산 비용이 줄어든 이 병목 구간에서 효율적으로 공간적 특징(spatial feature)을 추출한다.7
3. **1x1 컨볼루션 (차원 복원):** 마지막 1x1 컨볼루션 계층은 채널 수를 다시 원래대로 4배 확장하여(예: 64채널 -> 256채널) 복원한다. 이는 지름길 연결을 통해 전달된 입력과 차원을 일치시켜 덧셈 연산을 가능하게 한다.7

이러한 설계는 기본 블록에 비해 단일 블록 내의 표현력은 다소 희생될 수 있지만, 절약된 계산 비용 덕분에 훨씬 더 많은 블록을 쌓을 수 있게 해준다. 결과적으로 '강력한 소수의 블록'이 아닌 '효율적인 다수의 블록'을 통해 전체 네트워크의 깊이와 성능을 극대화하는 전략적 절충안인 셈이다.


ResNet 모델들은 동일한 설계 원칙을 공유하며, 잔차 블록의 종류와 각 스테이지에서의 반복 횟수에 따라 전체 깊이가 결정된다. 일반적으로 피처 맵의 공간적 해상도가 절반으로 줄어들 때(stride=2 컨볼루션을 통해 다운샘플링), 채널 수는 두 배로 늘려 계층별 연산 복잡도를 유사하게 유지하는 전략을 사용한다.26 아래 표는 주요 ResNet 아키텍처의 구성을 요약한 것이다.21

**Table 1: ResNet 아키텍처별 구성 비교**

| 계층 (Layer)       | ResNet-18         | ResNet-34         | ResNet-50                             | ResNet-101                             | ResNet-152                             |
| ------------------ | ----------------- | ----------------- | ------------------------------------- | -------------------------------------- | -------------------------------------- |
| conv1              | 7x7, 64, stride 2 | 7x7, 64, stride 2 | 7x7, 64, stride 2                     | 7x7, 64, stride 2                      | 7x7, 64, stride 2                      |
| conv2_x            | `[3x3, 64]` x 2   | `[3x3, 64]` x 3   | `[1x1, 64; 3x3, 64; 1x1, 256]` x 3    | `[1x1, 64; 3x3, 64; 1x1, 256]` x 3     | `[1x1, 64; 3x3, 64; 1x1, 256]` x 3     |
| conv3_x            | `[3x3, 128]` x 2  | `[3x3, 128]` x 4  | `[1x1, 128; 3x3, 128; 1x1, 512]` x 4  | `[1x1, 128; 3x3, 128; 1x1, 512]` x 4   | `[1x1, 128; 3x3, 128; 1x1, 512]` x 8   |
| conv4_x            | `[3x3, 256]` x 2  | `[3x3, 256]` x 6  | `[1x1, 256; 3x3, 256; 1x1, 1024]` x 6 | `[1x1, 256; 3x3, 256; 1x1, 1024]` x 23 | `[1x1, 256; 3x3, 256; 1x1, 1024]` x 36 |
| conv5_x            | `[3x3, 512]` x 2  | `[3x3, 512]` x 3  | `[1x1, 512; 3x3, 512; 1x1, 2048]` x 3 | `[1x1, 512; 3x3, 512; 1x1, 2048]` x 3  | `[1x1, 512; 3x3, 512; 1x1, 2048]` x 3  |
| **총 파라미터 수** | ~11.7M 30         | ~21.8M            | ~25.6M                                | ~44.5M 31                              | ~60.2M                                 |
| **블록 타입**      | Basic             | Basic             | Bottleneck                            | Bottleneck                             | Bottleneck                             |



ResNet은 152개 계층이라는 전례 없는 깊이에도 불구하고, 이전 세대 모델인 VGGNet보다 오히려 낮은 계산 복잡도를 가졌다.1 2015년 ILSVRC에서 ResNet 앙상블 모델은 ImageNet 테스트 데이터셋에 대해 3.57%의 top-5 에러율을 기록하며 압도적인 성능으로 1위를 차지했다.1 이는 당시 전문가 수준의 인간 분류 성능(약 5.1%)을 뛰어넘는 기념비적인 성과였으며, 기계가 특정 시각 인식 과제에서 인간을 능가할 수 있음을 보여준 상징적인 사건이었다.32 이 성공은 이미지 분류에 그치지 않고, ImageNet detection, ImageNet localization, COCO detection, COCO segmentation 등 당시 컴퓨터 비전 분야의 5대 주요 대회를 모두 석권하는 결과로 이어졌다.1 이는 ResNet이 학습한 특징 표현이 특정 작업에 국한되지 않는 매우 일반적이고 강력한 성능을 지녔음을 입증한 것이다.


ResNet의 가장 큰 영향력은 단순히 대회 우승을 넘어, 컴퓨터 비전 분야의 연구 및 개발 방법론 자체를 바꾸었다는 데 있다. ResNet의 압도적인 성공은 ImageNet 데이터셋으로 사전 훈련된 모델을 범용적인 특징 추출기, 즉 '백본(backbone)'으로 사용하고, 이를 다양한 다운스트림 작업(downstream task)에 맞게 미세 조정(fine-tuning)하는 전이 학습(transfer learning) 패러다임을 정착시켰다.9

이러한 변화는 컴퓨터 비전 연구의 진입 장벽을 극적으로 낮추고 발전 속도를 가속화했다. 이전에는 객체 탐지나 의미론적 분할과 같은 복잡한 작업을 위해 연구자들이 처음부터 끝까지 맞춤형 아키텍처를 설계하고 막대한 계산 자원을 투입해 훈련해야 했다. 하지만 ResNet의 등장 이후, 연구자들은 강력하고 일반화된 특징을 이미 학습한 ResNet 백본을 기반으로, 각자의 작업에 특화된 '헤드(head)' 부분의 설계와 미세 조정 전략에 집중할 수 있게 되었다. 이는 고품질 특징 추출 기술을 일종의 '상용화'시킨 효과를 낳았으며, 소규모 연구 그룹이나 개인도 최첨단 성능에 도전할 수 있는 길을 열어주었다.

- **객체 탐지 (Object Detection):** Faster R-CNN과 같은 2단계 탐지 모델에서 특징 추출 네트워크로 기존의 VGGNet 대신 ResNet을 사용함으로써 탐지 정확도가 비약적으로 향상되었다.35 현재 PyTorch와 같은 주요 딥러닝 프레임워크는 

  `fasterrcnn_resnet50_fpn`과 같이 ResNet 백본이 결합된 모델을 표준으로 제공하고 있다.36

- **의미론적 분할 (Semantic Segmentation):** U-Net과 같은 인코더-디코더 구조에서 인코더 부분에 ResNet을 채택하여 깊고 풍부한 컨텍스트 정보를 효과적으로 추출하는 방식이 표준으로 자리 잡았다.40 이를 통해 픽셀 단위의 분할 정확도를 크게 개선할 수 있었다.


ResNet의 등장은 끝이 아니라 새로운 시작이었다. 그 성공은 "어떻게 하면 더 나은 특징 표현을 학습할 수 있는가?"라는 근본적인 질문에 대한 다양한 철학적 논쟁과 탐구를 촉발시켰다. 이후 등장한 주요 아키텍처들은 ResNet의 핵심 원리를 계승, 비판, 또는 재해석하며 발전해 나갔다.


WRN은 "더 깊은 것이 항상 더 좋은가?"라는 질문에 대한 반론으로 등장했다. 이 모델은 ResNet의 깊이를 늘리는 대신, 각 잔차 블록의 너비(채널 수)를 늘리는 것이 성능 향상에 더 효율적일 수 있음을 실험적으로 증명했다.43 연구에 따르면, 16개 층의 상대적으로 얕은 WRN이 1000개 층에 달하는 극도로 깊은 ResNet보다 우수한 성능을 보였다.43 이는 깊이에 대한 맹목적인 추구보다 블록 자체의 표현력을 강화하는 것이 더 중요할 수 있음을 시사하며, 극단적인 깊이에서 발생하는 '특징 재사용 감소(diminishing feature reuse)' 문제를 해결하려는 시도였다.44


ResNeXt는 잔차 블록의 내부 구조를 재해석했다. 기존 ResNet 블록의 단일 변환 경로를 다수의 병렬 경로로 분기(Split), 각 경로에서 동일한 토폴로지의 변환을 수행(Transform), 그리고 그 결과를 다시 병합(Merge)하는 전략을 채택했다.46 이 병렬 변환 경로의 수를 '카디널리티(cardinality)'라는 새로운 하이퍼파라미터로 정의했다. ResNeXt는 모델의 전체 계산 복잡도를 유지하면서 카디널리티를 높이는 것이 깊이나 너비를 늘리는 것보다 성능 향상에 더 효과적임을 보였다.46 이는 하나의 복잡한 변환 함수보다 다수의 단순한 변환 함수의 앙상블이 더 강력한 표현력을 가질 수 있다는 아이디어를 제시한 것이다.


DenseNet은 ResNet의 지름길 연결 아이디어를 극단으로 확장했다. ResNet이 덧셈(addition)을 통해 이전 계층의 정보를 '보존'했다면, DenseNet은 채널 축을 따라 연결(concatenation)함으로써 정보를 '누적'한다. DenseNet에서는 각 계층이 자신보다 앞선 모든 계층의 피처 맵을 입력으로 받는다.50 이 조밀한 연결(dense connectivity) 구조는 네트워크 내 정보 흐름을 극대화하고, 각 계층이 이전의 모든 수준의 특징에 직접 접근할 수 있게 하여 기울기 소실 문제를 더욱 효과적으로 완화한다. 또한, 명시적인 특징 재사용(feature reuse)을 장려하여 훨씬 적은 파라미터로 높은 성능을 달성하는 뛰어난 파라미터 효율성을 보여주었다.50


Vision Transformer(ViT)는 ResNet이 완성한 컨볼루션 신경망(CNN) 패러다임 자체에 근본적인 질문을 던졌다. ViT는 CNN의 고유한 귀납적 편향(inductive bias)인 지역성(locality)과 평행이동 불변성(translation invariance)에서 벗어나, 이미지를 여러 개의 작은 패치(patch)로 분할하고 이를 토큰(token)의 시퀀스로 취급한다.51 그리고 자연어 처리(NLP) 분야에서 혁명을 일으킨 트랜스포머의 셀프 어텐션(self-attention) 메커니즘을 사용하여 이미지 내 모든 패치 간의 전역적인(global) 관계를 직접 학습한다.

- **ResNet vs. ViT 비교:**
  - **구조:** ResNet은 컨볼루션 연산을 통해 계층적으로 지역적 특징을 추출하고 이를 통합하는 반면, ViT는 셀프 어텐션을 통해 처음부터 패치 간의 전역적 관계를 파악한다.51
  - **데이터 요구량:** ViT는 CNN과 같은 내재된 편향이 없기 때문에, 모델이 이미지의 구조를 처음부터 데이터만으로 학습해야 한다. 이로 인해 JFT-300M과 같은 초거대 데이터셋으로 사전 훈련될 때 ResNet을 능가하는 강력한 성능을 보이지만, 상대적으로 데이터가 적을 경우에는 과적합되기 쉬워 CNN 기반 모델보다 성능이 저하될 수 있다.51
  - **장단점:** ViT는 뛰어난 확장성과 전역적 컨텍스트 파악 능력을 장점으로 갖는다. 반면 ResNet은 적은 데이터로도 견고한 성능을 보이며, 잘 정립된 백본 아키텍처로서의 안정성이 강점이다.52

이러한 후속 모델들의 등장은 ResNet이 제시한 문제 해결 방식에 대한 끊임없는 탐구의 결과물이다. WRN과 ResNeXt는 잔차 블록의 **내부 구조**를 두고 '너비'와 '카디널리티' 중 무엇이 더 효율적인지를 탐구했으며, DenseNet은 블록 **간의 연결 토폴로지**를 재고했다. 그리고 ViT는 ResNet이 기반하고 있는 **컨볼루션이라는 근본적인 가정** 자체를 버리고 새로운 패러다임을 제시했다. 이처럼 ResNet 이후의 컴퓨터 비전 아키텍처 발전사는 구조적 선험 지식, 파라미터 효율성, 그리고 데이터 규모 사이의 근본적인 상호작용을 탐구하는 과정으로 볼 수 있다.


ResNet은 단순히 하나의 성공적인 모델을 넘어, 심층 신경망 연구의 방향을 근본적으로 바꾼 이정표이다. '성능 저하'라는 미묘하지만 치명적인 최적화 문제를 '잔차 학습'이라는 우아하고 직관적인 아이디어로 해결함으로써, 100계층을 넘어 1000계층까지 논할 수 있는 진정한 의미의 '초심층(very deep)' 네트워크 시대를 열었다.

ResNet이 제시한 잔차 연결의 원리는 컴퓨터 비전 분야를 넘어, 이후 등장하는 거의 모든 최신 아키텍처에 깊은 영향을 미쳤다. 특히 자연어 처리에서 시작된 트랜스포머 모델조차도 각 서브 계층에 잔차 연결을 핵심 요소로 채택하고 있으며, 이는 매우 깊은 트랜스포머의 안정적인 학습을 가능하게 하는 필수적인 장치로 기능한다.7

또한, ResNet은 '백본'이라는 개념을 통해 컴퓨터 비전 분야의 연구 개발 생태계를 표준화하고 가속화했다. 잘 훈련된 ResNet은 다양한 시각적 과제를 해결하기 위한 강력하고 신뢰할 수 있는 출발점을 제공했으며, 이는 오늘날까지도 수많은 애플리케이션과 연구의 기반이 되고 있다.

결론적으로 ResNet의 유산은 특정 구조에 국한되지 않는다. 그것은 "어떻게 하면 더 깊은 모델을 효과적으로 훈련시킬 것인가?"라는 근본적인 질문에 대한 강력하고 실용적인 해답을 제시했으며, 그 해답은 오늘날까지도 수많은 모델 설계의 근간을 이루며 인공지능 기술의 발전을 이끌고 있다.


1. arXiv:1512.03385v1 [cs.CV] 10 Dec 2015, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1512.03385
2. Vanishing Gradient vs Degradation | by Shaoliang Jia | Medium, 8월 24, 2025에 액세스, https://medium.com/@shaoliang.jia/vanishing-gradient-vs-degradation-b719594b6877
3. Skip Connection and Explanation of ResNet | by Chau Tuan Kien - Medium, 8월 24, 2025에 액세스, https://chautuankien.medium.com/skip-connection-and-explanation-of-resnet-afabe792346c
4. Residual Networks (ResNet) - Deep Learning - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/deep-learning/residual-networks-resnet-deep-learning/
5. Skip connections and residual blocks for deeper neural networks (ResNet) - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/learnmachinelearning/comments/x0q39m/skip_connections_and_residual_blocks_for_deeper/
6. 8.6. Residual Networks (ResNet) and ResNeXt - Dive into Deep Learning, 8월 24, 2025에 액세스, http://d2l.ai/chapter_convolutional-modern/resnet.html
7. Residual neural network - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Residual_neural_network
8. What Is ResNet-50? - Roboflow Blog, 8월 24, 2025에 액세스, https://blog.roboflow.com/what-is-resnet-50/
9. What are Skip Connections in Deep Learning? - Analytics Vidhya, 8월 24, 2025에 액세스, https://www.analyticsvidhya.com/blog/2021/08/all-you-need-to-know-about-skip-connections/
10. [D] Has the ResNet Hypothesis been debunked? : r/MachineLearning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/px3hzd/d_has_the_resnet_hypothesis_been_debunked/
11. An Introduction to ResNets. Residual networks, or ResNets for… | by Francesco Franco | AI Mind, 8월 24, 2025에 액세스, https://pub.aimind.so/a-brief-introduction-to-resnets-d43ae4f1e2a0
12. arXiv:1604.04112v4 [cs.CV] 5 Oct 2016, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1604.04112
13. Vanishing gradient problem - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Vanishing_gradient_problem
14. [D]: Vanishing Gradients and Resnets : r/MachineLearning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/11wmpoj/d_vanishing_gradients_and_resnets/
15. Aman's AI Journal • Primers • Skip Connections, 8월 24, 2025에 액세스, https://aman.ai/primers/ai/skip-connections/
16. Intuitive Explanation of Skip Connections in Deep Learning | AI ..., 8월 24, 2025에 액세스, https://theaisummer.com/skip-connections/
17. Chapter 6 The Backprop Algorithm - Deep Learning, 8월 24, 2025에 액세스, https://srdas.github.io/DLBook/TrainingNNsBackprop.html
18. A Comprehensive Guide to Understanding and Implementing Bottleneck Residual Blocks | by Neetu Sigger Choudhary | Medium, 8월 24, 2025에 액세스, https://medium.com/@neetu.sigger/a-comprehensive-guide-to-understanding-and-implementing-bottleneck-residual-blocks-6b420706f66b
19. ResNet, torchvision, bottlenecks, and layers not as they seem. - Erik Gaasedelen - Medium, 8월 24, 2025에 액세스, https://erikgaas.medium.com/resnet-torchvision-bottlenecks-and-layers-not-as-they-seem-145620f93096
20. ResNet, Bottleneck, Layers, groups, width_per_group - vision - PyTorch Forums, 8월 24, 2025에 액세스, https://discuss.pytorch.org/t/resnet-bottleneck-layers-groups-width-per-group/69088
21. ResNet18 & ResNet50 in Computer Vision - Product Teacher, 8월 24, 2025에 액세스, https://www.productteacher.com/quick-product-tips/resnet18-and-resnet50
22. Comparison between Basic (left) and Bottleneck Block (right) of the ..., 8월 24, 2025에 액세스, https://www.researchgate.net/figure/Comparison-between-Basic-left-and-Bottleneck-Block-right-of-the-ResNet-architecture_fig6_357301584
23. How the bottleneck layers in the deep networks work and how do those layers reduce computational complexity | by Yawar Rehman | Medium, 8월 24, 2025에 액세스, https://medium.com/@reh.yawar2/how-the-bottleneck-layers-in-the-deep-networks-work-and-how-do-those-layers-reduce-computational-7bc99c0d1e96
24. Talented Mr. 1X1: Comprehensive look at 1X1 Convolution in Deep Learning - Medium, 8월 24, 2025에 액세스, https://medium.com/analytics-vidhya/talented-mr-1x1-comprehensive-look-at-1x1-convolution-in-deep-learning-f6b355825578
25. Stuck understanding ResNet's Identity block and Convolutional blocks - Stack Overflow, 8월 24, 2025에 액세스, https://stackoverflow.com/questions/58200107/stuck-understanding-resnets-identity-block-and-convolutional-blocks
26. Deep Residual Learning for Image Recognition (ResNet Explained) - Analytics Vidhya, 8월 24, 2025에 액세스, https://www.analyticsvidhya.com/blog/2023/02/deep-residual-learning-for-image-recognition-resnet-explained/
27. Understanding ResNet and analyzing various models on the CIFAR-10 dataset, 8월 24, 2025에 액세스, https://www.analyticsvidhya.com/blog/2021/06/understanding-resnet-and-analyzing-various-models-on-the-cifar-10-dataset/
28. The five types of ResNet from left 18-layers, 34-layers, 50-layers ..., 8월 24, 2025에 액세스, https://www.researchgate.net/figure/The-five-types-of-ResNet-from-left-18-layers-34-layers-50-layers-101-layers-and_tbl2_329618132
29. Architecture of ResNet 18, 34, 50, 101 and 152 [11, 26] | Download ..., 8월 24, 2025에 액세스, https://www.researchgate.net/figure/Architecture-of-ResNet-18-34-50-101-and-152-11-26_tbl6_353903737
30. resnet18 — Torchvision main documentation - PyTorch, 8월 24, 2025에 액세스, https://pytorch.org/vision/main/models/generated/torchvision.models.resnet18.html
31. resnet101 — Torchvision main documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/vision/main/models/generated/torchvision.models.resnet101.html
32. 7 Popular Image Classification Models in ImageNet Challenge (ILSVRC) Competition History - MLK - Machine Learning Knowledge, 8월 24, 2025에 액세스, https://machinelearningknowledge.ai/popular-image-classification-models-in-imagenet-challenge-ilsvrc-competition-history/
33. ImageNet Winning CNN Architectures (ILSVRC) - Kaggle, 8월 24, 2025에 액세스, https://www.kaggle.com/getting-started/149448
34. KaimingHe/deep-residual-networks - GitHub, 8월 24, 2025에 액세스, https://github.com/KaimingHe/deep-residual-networks
35. Review: ResNet — Winner of ILSVRC 2015 (Image Classification, Localization, Detection), 8월 24, 2025에 액세스, https://sh-tsang.medium.com/review-resnet-winner-of-ilsvrc-2015-image-classification-localization-detection-e39402bfa5d8?source=post_internal_links---------6----------------------------
36. Understanding and Implementing Faster R-CNN | by Rishabh Singh | Medium, 8월 24, 2025에 액세스, https://medium.com/@RobuRishabh/understanding-and-implementing-faster-r-cnn-248f7b25ff96
37. Faster R-CNN Resnet 50 FPN - Medium, 8월 24, 2025에 액세스, https://medium.com/@sayandaskhan/faster-r-cnn-resnet-50-fpn-575c8c1eef00
38. MS-Faster R-CNN: Multi-Stream Backbone for Improved Faster R-CNN Object Detection and Aerial Tracking from UAV Images - MDPI, 8월 24, 2025에 액세스, https://www.mdpi.com/2072-4292/13/9/1670
39. Faster R-CNN — Torchvision main documentation, 8월 24, 2025에 액세스, https://docs.pytorch.org/vision/main/models/faster_rcnn.html
40. Framework of U-Net architecture [21] (left side with Resnet-101) for top view person segmentation. - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/Framework-of-U-Net-architecture-21-left-side-with-Resnet-101-for-top-view-person_fig3_343170165
41. Introduction to U-Net and Res-Net for Image Segmentation | by Aditi Mittal | Medium, 8월 24, 2025에 액세스, https://aditi-mittal.medium.com/introduction-to-u-net-and-res-net-for-image-segmentation-9afcb432ee2f
42. Evaluation of U-Net Backbones for Cloud Segmentation in Satellite Images - SciTePress, 8월 24, 2025에 액세스, https://www.scitepress.org/PublishedPapers/2023/117872/117872.pdf
43. Wide Residual Networks, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1605.07146
44. Wide Residual Networks - BMVA Archive, 8월 24, 2025에 액세스, https://bmva-archive.org.uk/bmvc/2016/papers/paper087/index.html
45. [1605.07146] Wide Residual Networks - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/1605.07146
46. Aggregated Residual Transformations for Deep Neural Networks, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1611.05431
47. paper summary: “Aggregated Residual Transformations for Deep Neural Networks” (ResNext Paper) | by Chadrick, 8월 24, 2025에 액세스, https://chadrick-kwag.medium.com/paper-summary-aggregated-residual-transformations-for-deep-neural-networks-resnext-paper-578da93c3a3b
48. [1611.05431] Aggregated Residual Transformations for Deep Neural Networks - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/1611.05431
49. Evaluating ResNeXt Model Architecture for Image Classification - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1805.08700
50. Densely Connected Convolutional Networks, 8월 24, 2025에 액세스, https://arxiv.org/abs/1608.06993
51. Vision Transformers (ViT) Explained: Are They Better Than CNNs ..., 8월 24, 2025에 액세스, https://towardsdatascience.com/vision-transformers-vit-explained-are-they-better-than-cnns/
52. 97% Accuracy with ViT on 90 Animal Dataset; A comparative study Vision Transformers vs. ResNet in Multi-Class Classification | by Sheeza Shabbir | Medium, 8월 24, 2025에 액세스, https://medium.com/@datascientist_SheezaShabbir/a-comparative-study-on-vision-transformers-vs-resnet-in-multi-class-classification-544d9b665365
53. [D] Have transformers won in Computer Vision? : r/MachineLearning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/1hzn0gg/d_have_transformers_won_in_computer_vision/
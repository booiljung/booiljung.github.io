[이미지 분류 (Image Classifications)](./index.md)
# GoogLeNet (2014)


2012년 AlexNet이 ImageNet Large-Scale Visual Recognition Challenge (ILSVRC)에서 압도적인 성능을 보인 이래, 심층 컨볼루션 신경망(Deep Convolutional Neural Network, CNN)은 컴퓨터 비전 분야의 주류로 자리 잡았다. 이후 연구자들은 네트워크의 깊이를 증가시키는 것이 성능 향상의 핵심이라는 직관에 따라 모델을 더욱 깊게 쌓는 방향으로 나아갔다.1 그러나 이러한 접근법은 두 가지 심각한 문제에 직면했다. 첫째, 네트워크가 깊어질수록 학습해야 할 파라미터의 수가 기하급수적으로 증가하여 막대한 계산 자원을 요구했으며, 이는 과적합(overfitting)의 위험을 높였다.2 둘째, 네트워크 깊이가 임계점을 넘어서면 역전파 과정에서 기울기가 점차 사라지는 기울기 소실(vanishing gradient) 문제가 발생하여 학습이 제대로 이루어지지 않았다.1

이러한 기술적 배경 속에서 2014년, Google 연구팀은 "Going Deeper with Convolutions"라는 논문을 통해 GoogLeNet이라는 혁신적인 아키텍처를 제시했다.5 GoogLeNet은 ILSVRC 2014의 분류(classification) 및 탐지(detection) 부문에서 우승을 차지하며 당대 최고의 성능을 입증했다.6 이 모델의 핵심 철학은 무작정 깊이를 늘리는 대신, "제한된 계산 예산 내에서 깊이와 너비를 동시에 증가시켜 네트워크 내부의 컴퓨팅 자원 활용도를 극대화"하는 것이었다.5 이 철학은 성능 향상을 위해 계산 효율성을 희생하던 기존의 패러다임을 근본적으로 뒤집는 것이었다. GoogLeNet은 성능과 효율성의 상충 관계를 재정의하며, '무작정 깊게'가 아닌 '지능적으로 깊고 넓게'라는 새로운 설계 방향을 제시한 분기점이라 할 수 있다.

GoogLeNet의 성공은 'Inception'이라 명명된 독창적인 모듈 구조에 기반한다.5 이 모듈은 다양한 크기의 컨볼루션 필터를 병렬로 적용하여 다중 스케일의 특징을 동시에 추출한다. 또한, 1x1 컨볼루션을 병목(bottleneck) 계층으로 활용하여 계산량을 획기적으로 줄이면서도 모델의 표현력을 높이는 기법을 도입했다. 깊은 네트워크의 안정적인 학습을 위해 중간 계층에 보조 분류기(auxiliary classifier)를 추가하고, 마지막 분류 단계에서는 전통적인 완전 연결 계층(fully connected layer)을 전역 평균 풀링(global average pooling)으로 대체하여 파라미터 수를 대폭 감소시키고 과적합을 억제했다. 이러한 혁신 덕분에 GoogLeNet은 22개 층의 깊은 구조를 가지면서도, 2년 전 우승 모델인 AlexNet보다 12배나 적은 파라미터로 훨씬 높은 정확도를 달성하는 경이로운 효율성을 보여주었다.6 모델의 이름은 Yann LeCun의 선구적인 LeNet-5에 대한 경의를 표하기 위해 GoogLeNet으로 명명되었다.6

본 보고서는 GoogLeNet 아키텍처의 핵심 구성 요소들을 심층적으로 분석하고, 그 설계 철학과 기술적 기여가 이후 딥러닝 발전에 어떠한 영향을 미쳤는지 고찰하고자 한다. 먼저 Inception 모듈의 구조와 1x1 컨볼루션의 혁신적인 활용법을 살펴본 후, 전체 네트워크 아키텍처를 상세히 분해한다. 이어서 보조 분류기와 전역 평균 풀링 같은 안정적 학습 장치를 분석하고, ILSVRC 2014에서의 성능을 동시대 경쟁 모델인 VGGNet과 비교 평가한다. 마지막으로, GoogLeNet이 남긴 기술적 유산과 그 의의를 조명하며 결론을 맺는다.


GoogLeNet 아키텍처의 심장이라 할 수 있는 Inception 모듈은 컴퓨터 비전의 근본적인 문제의식에서 출발한다. 이미지 내에서 의미 있는 특징(salient feature)은 다양한 스케일로 존재한다. 예를 들어, 어떤 이미지에서는 강아지가 화면 전체를 차지할 수도 있고, 다른 이미지에서는 아주 작은 영역에만 나타날 수도 있다. 기존의 CNN 모델들은 각 계층에서 단일 크기의 컨볼루션 필터(예: 3x3 또는 5x5)를 사용했기 때문에, 이러한 다중 스케일 정보를 효과적으로 포착하는 데 한계가 있었다.10 최적의 필터 크기는 데이터와 계층의 깊이에 따라 달라지므로, 이를 미리 고정하는 것은 차선책일 수밖에 없다.

Inception 모듈은 이 문제를 해결하기 위해 "네트워크가 스스로 최적의 특징 스케일을 학습하게 하자"는 아이디어를 구현했다.3 이를 위해 하나의 입력 피처 맵에 대해 여러 연산을 병렬적으로 수행하고, 그 결과를 모두 합치는 방식을 채택했다. 구체적으로 Inception 모듈은 다음과 같은 네 개의 병렬 경로로 구성된다.11

1. 1x1 컨볼루션 경로
2. 3x3 컨볼루션 경로
3. 5x5 컨볼루션 경로
4. 3x3 최대 풀링(Max Pooling) 경로

각 컨볼루션 경로는 서로 다른 크기의 수용장(receptive field)을 통해 지역적인 세부 특징(1x1, 3x3)부터 좀 더 넓은 영역의 추상적인 특징(5x5)까지 다양한 스케일의 정보를 동시에 추출한다. 최대 풀링 경로는 입력의 공간적 특징을 요약하는 역할을 한다. 각 경로를 통과한 출력 피처 맵들은 공간적 차원(너비와 높이)이 동일하게 유지되도록 패딩(padding)이 적용된 후, 채널(depth) 차원을 따라 모두 연결(concatenate)된다.12 이렇게 생성된 거대한 피처 맵이 다음 계층의 입력으로 전달됨으로써, 후속 계층은 다양한 스케일에서 추출된 풍부한 정보를 바탕으로 더 복잡한 특징을 학습할 수 있게 된다.


이러한 병렬 구조는 개념적으로는 강력하지만, 그대로 구현할 경우 심각한 계산 비용 문제를 야기한다. 이를 '초기(Naive) Inception 모듈'이라 칭한다.2 예를 들어, 28x28 크기에 192개의 채널을 가진 피처 맵이 입력으로 들어온다고 가정하자. 만약 이 입력에 대해 32개의 5x5 컨볼루션을 적용한다면, 필요한 연산량은 대략 (5x5 필터 크기) x (192 입력 채널) x (32 출력 채널) x (28x28 출력 크기)로, 약 1억 2천만 번의 곱셈-누산(multiply-add) 연산이 필요하다.15 네트워크가 깊어지면서 채널 수가 누적됨에 따라 이러한 계산량은 감당할 수 없는 수준으로 폭증하게 된다. 또한, 최대 풀링 연산은 채널 수를 줄이지 않고 그대로 전달하기 때문에, 계층을 거듭할수록 채널 깊이가 계속해서 증가하는 문제도 발생한다.15


GoogLeNet은 이 계산 병목 문제를 해결하기 위해 1x1 컨볼루션을 활용한 '차원 축소(dimensionality reduction)' 기법을 도입했다.2 이것이 최종적으로 채택된 Inception 모듈의 형태이다. 핵심 아이디어는 계산 비용이 많이 드는 3x3 및 5x5 컨볼루션을 수행하기 직전에, 1x1 컨볼루션을 사용하여 입력 피처 맵의 채널 수를 먼저 줄이는 것이다. 이 1x1 컨볼루션 계층을 '감소(reduce)' 계층이라고 부른다.

앞선 예시를 다시 살펴보자. 28x28x192 입력에 대해 바로 32개의 5x5 컨볼루션을 적용하는 대신, 먼저 16개의 1x1 컨볼루션을 적용하여 입력을 28x28x16으로 압축한다. 그 후, 이 압축된 피처 맵에 32개의 5x5 컨볼루션을 적용한다. 이 경우 총 연산량은 (1x1x192x16) + (5x5x16x32)로, 약 1240만 번으로 줄어든다. 이는 초기 버전에 비해 계산량을 거의 10분의 1 수준으로 감소시킨 것이다.15 또한, 최대 풀링 경로 이후에도 1x1 컨볼루션을 추가하여 풀링을 통해 유지된 채널 수를 조절하는 '투영(projection)' 계층 역할을 수행하도록 했다.11

이러한 설계는 단순히 계산 효율성만 높이는 데 그치지 않는다. 이는 네트워크 설계 패러다임의 중요한 변화를 의미한다. 기존의 AlexNet이나 VGGNet이 설계자가 전역적으로 결정한 단일 크기의 필터를 수동적으로 사용하는 방식이었다면, Inception 모듈은 각 블록이 입력 데이터에 가장 적합한 특징 스케일의 조합을 능동적으로 학습할 수 있는 유연성을 부여한 것이다.3 이는 네트워크의 너비(width)에 대한 개념을 단순한 채널 수의 증가가 아닌, 서로 다른 연산을 수행하는 병렬 경로의 다양성, 즉 '구조적 다양성(structural diversity)'으로 확장시킨 것으로 평가할 수 있다.8


GoogLeNet의 성공을 가능하게 한 가장 핵심적인 단일 기술을 꼽으라면 단연 1x1 컨볼루션이다. '점별 컨볼루션(pointwise convolution)'이라고도 불리는 이 연산은 Lin 등이 제안한 'Network in Network' 논문에서 처음 소개되었으며, GoogLeNet은 이를 효과적으로 활용하여 계산 효율성과 모델 표현력이라는 두 마리 토끼를 모두 잡았다.16

1x1 컨볼루션은 이름 그대로 필터의 너비와 높이가 1인 컨볼루션 연산이다. 필터 크기가 1x1이므로 주변 픽셀을 고려하지 않고, 입력 피처 맵의 동일한 공간적 위치(‘(i,j)‘)에 있는 모든 채널 값들에 대해서만 가중합을 계산한다.17 만약 입력 피처 맵의 차원이 $H \times W \times C_{in}$이고, $C_{out}$개의 1x1 컨볼루션 필터를 적용한다면, 각 필터의 실제 크기는 $1 \times 1 \times C_{in}$이 된다. 이 필터가 입력 전체를 훑으며 연산을 수행하면, 출력 피처 맵의 차원은 공간적 크기는 그대로 유지된 채 채널 수만 변경된 $H \times W \times C_{out}$이 된다.16 이는 공간적 특징은 혼합하지 않으면서 채널 간의 관계만을 학습하는 효과를 가져오며, 사실상 각 픽셀 위치의 채널 벡터에 대한 완전 연결 계층(fully connected layer)과 동일한 역할을 수행한다.18

GoogLeNet에서 1x1 컨볼루션은 주로 세 가지 핵심적인 역할을 수행한다.

첫째, 가장 중요한 역할은 앞서 설명한 **차원 축소를 통한 계산 병목 현상 해결**이다. Inception 모듈 내에서 계산 비용이 높은 3x3 및 5x5 컨볼루션 이전에 1x1 컨볼루션을 배치하여 입력 채널의 수를 극적으로 줄인다. 이는 마치 넓은 입구의 병목을 통과시켜 데이터의 흐름을 효율적으로 제어하는 것과 같아 '병목 계층(bottleneck layer)'이라고 불린다.16 이 구조는 후속 연산의 파라미터 수와 계산량을 대폭 감소시켜, 제한된 계산 자원 내에서 더 깊고 넓은 네트워크를 구성할 수 있게 하는 결정적인 열쇠가 되었다.11

둘째, **채널 간 특징 재조합(feature recombination)**을 통해 모델의 표현력을 향상시킨다. 1x1 컨볼루션은 입력 채널들의 선형 결합을 통해 새로운 특징을 생성한다.18 예를 들어, RGB 이미지의 세 채널에 1x1 컨볼루션을 적용하면, 각 채널의 정보를 적절히 조합하여 흑백 이미지나 특정 색상을 강조하는 새로운 피처 맵을 만들어낼 수 있다. 이처럼 채널 간의 복잡한 상호작용을 학습함으로써, 네트워크는 더 풍부하고 유의미한 특징 표현을 얻게 된다.17

셋째, **비선형성(non-linearity)을 추가**하는 역할을 한다. 모든 1x1 컨볼루션 계층 뒤에는 ReLU(Rectified Linear Unit)와 같은 활성화 함수가 적용된다.11 이는 추가적인 파라미터나 계산량 증가 없이 네트워크의 비선형성을 높여, 모델이 더 복잡한 함수를 근사할 수 있도록 돕는다.16

이처럼 1x1 컨볼루션은 계산 비용을 '줄이는' 것이 주목적이면서도, 그 과정에서 특징을 재조합하고 비선형성을 추가하여 모델의 표현력을 오히려 '향상'시키는 부수적인 효과를 낳았다. 이는 비용 절감을 위한 구조적 선택이 모델의 질적 향상으로 이어진 매우 이례적인 경우로, '공짜 점심은 없다'는 딥러닝 설계의 통념을 깨뜨린 혁신으로 평가받는다. 이 아이디어는 이후 ResNet의 병목 블록 구조에 그대로 계승되었으며 16, 현대의 거의 모든 효율적인 CNN 아키텍처에서 핵심 구성 요소로 사용되고 있다.


GoogLeNet은 총 22개의 학습 가능한 가중치 계층(weight layer)과 풀링 계층을 포함하여 총 27개의 계층으로 구성된 깊은 네트워크이다.22 전체 구조는 크게 세 부분, 즉 초기 특징을 추출하는 '스템(Stem)', 핵심적인 특징 학습이 이루어지는 '몸통(Body)', 그리고 최종 분류를 수행하는 '머리(Head)'로 나눌 수 있다.23

**스템 네트워크(Stem Network):** 네트워크의 가장 앞부분에 위치하며, 입력 이미지(224x224x3)를 받아 저수준 특징을 추출하고 데이터의 크기를 줄이는 역할을 한다. 스템은 몇 개의 일반적인 컨볼루션 및 풀링 계층으로 구성된다. 먼저 7x7 크기의 큰 필터를 가진 컨볼루션 계층이 스트라이드(stride) 2로 적용되어 이미지의 해상도를 절반(112x112)으로 줄이면서 초기 특징을 추출한다. 이후 3x3 최대 풀링(스트라이드 2)을 통해 해상도를 다시 절반(56x56)으로 줄인다. 이어서 두 개의 컨볼루션 계층과 또 한 번의 최대 풀링(스트라이드 2)을 거치면서 최종적으로 28x28 크기의 피처 맵을 생성하여 몸통 부분으로 전달한다.15 초기 계층에서 큰 필터와 빠른 공간적 다운샘플링을 통해 계산량을 효율적으로 관리하는 전략을 엿볼 수 있다.

**몸통(Body):** 네트워크의 핵심부로, 총 9개의 Inception 모듈이 직렬로 연결되어 있다. 이 9개의 모듈은 중간에 두 번의 최대 풀링 계층을 기준으로 세 개의 그룹으로 나뉜다. 첫 번째 그룹은 Inception (3a), (3b)로 구성되며 28x28 크기의 피처 맵을 처리한다. Inception (3b) 이후에 위치한 최대 풀링 계층이 피처 맵의 해상도를 14x14로 줄인다. 두 번째 그룹은 Inception (4a)부터 (4e)까지 총 5개의 모듈로 구성되며, 14x14 크기의 피처 맵을 처리한다. Inception (4e) 이후의 최대 풀링 계층은 해상도를 다시 7x7로 줄인다. 마지막 그룹은 Inception (5a), (5b)로 구성되며 7x7 크기의 피처 맵을 처리한다.15 이 구조를 통해 네트워크는 깊어질수록 공간 해상도는 점차 줄여나가고, 대신 Inception 모듈을 통해 채널의 깊이, 즉 특징의 복잡성과 다양성을 크게 확장하는 계층적 설계를 보여준다.

**머리(Head):** 최종 분류를 담당하는 부분이다. 마지막 Inception 모듈(5b)의 출력(7x7x1024)을 받아, 7x7 크기의 전역 평균 풀링(Global Average Pooling) 계층을 통과시킨다. 이 과정에서 각 채널의 7x7 피처 맵은 평균값인 단일 스칼라 값으로 압축되어 1x1x1024 크기의 벡터가 된다. 이후 드롭아웃(dropout)을 적용하고, 최종적으로 1000개의 유닛을 가진 선형(linear) 계층과 소프트맥스(softmax) 함수를 통해 1000개 클래스에 대한 최종 확률을 출력한다.15

아래 표는 GoogLeNet의 전체 아키텍처를 계층별로 상세히 명시한 것이다.15

**표 1: GoogLeNet 계층별 명세**

| type           | patch size/stride | output size | depth | #1x1 | #3x3 reduce | #3x3 | #5x5 reduce | #5x5 | pool proj |
| -------------- | ----------------- | ----------- | ----- | ---- | ----------- | ---- | ----------- | ---- | --------- |
| convolution    | 7x7/2             | 112x112x64  | 64    |      |             |      |             |      |           |
| max pool       | 3x3/2             | 56x56x64    | 64    |      |             |      |             |      |           |
| convolution    | 3x3/1             | 56x56x192   | 192   |      |             |      |             |      |           |
| max pool       | 3x3/2             | 28x28x192   | 192   |      |             |      |             |      |           |
| inception (3a) |                   | 28x28x256   | 256   | 64   | 96          | 128  | 16          | 32   | 32        |
| inception (3b) |                   | 28x28x480   | 480   | 128  | 128         | 192  | 32          | 96   | 64        |
| max pool       | 3x3/2             | 14x14x480   | 480   |      |             |      |             |      |           |
| inception (4a) |                   | 14x14x512   | 512   | 192  | 96          | 208  | 16          | 48   | 64        |
| inception (4b) |                   | 14x14x512   | 512   | 160  | 112         | 224  | 24          | 64   | 64        |
| inception (4c) |                   | 14x14x512   | 512   | 128  | 128         | 256  | 24          | 64   | 64        |
| inception (4d) |                   | 14x14x528   | 528   | 112  | 144         | 288  | 32          | 64   | 64        |
| inception (4e) |                   | 14x14x832   | 832   | 256  | 160         | 320  | 32          | 128  | 128       |
| max pool       | 3x3/2             | 7x7x832     | 832   |      |             |      |             |      |           |
| inception (5a) |                   | 7x7x832     | 832   | 256  | 160         | 320  | 32          | 128  | 128       |
| inception (5b) |                   | 7x7x1024    | 1024  | 384  | 192         | 384  | 48          | 128  | 128       |
| avg pool       | 7x7/1             | 1x1x1024    | 1024  |      |             |      |             |      |           |
| linear         |                   | 1x1x1000    | 1000  |      |             |      |             |      |           |
| softmax        |                   | 1x1x1000    | 1000  |      |             |      |             |      |           |

이러한 체계적인 구조는 GoogLeNet이 단순히 Inception 모듈을 무작위로 쌓은 것이 아니라, '공간 정보 압축'과 '채널(의미) 정보 확장' 사이의 트레이드오프를 네트워크 깊이에 따라 동적으로 조절하는 정교한 설계 철학에 기반하고 있음을 보여준다.


22층에 달하는 깊은 네트워크를 성공적으로 학습시키는 것은 당시로서는 매우 어려운 과제였다. 특히 기울기 소실 문제는 네트워크의 깊이가 깊어질수록 초기 계층까지 의미 있는 기울기가 전달되지 않아 학습을 저해하는 주된 요인이었다. GoogLeNet은 이 문제를 해결하고 안정적인 학습을 도모하기 위해 두 가지 독창적인 장치를 도입했다: 보조 분류기(Auxiliary Classifier)와 전역 평균 풀링(Global Average Pooling)이다.


보조 분류기는 깊은 네트워크의 중간 지점에서 추가적인 기울기를 주입하여 기울기 소실 문제를 완화하기 위해 고안된 장치이다.1 네트워크의 최종단에서 계산된 손실(loss)만이 역전파를 통해 전체 네트워크를 학습시키는 것이 아니라, 중간 계층의 출력에 직접 연결된 작은 분류기들을 통해 추가적인 손실을 계산하고 이를 전체 손실에 더해주는 방식이다.3 이 '중간 감독(intermediate supervision)'을 통해 네트워크의 초기 및 중간 계층들이 최종 출력에만 의존하지 않고도 유의미한 학습 신호를 받을 수 있게 된다.

GoogLeNet에서는 Inception (4a)와 Inception (4d) 모듈의 출력에 각각 하나씩, 총 두 개의 보조 분류기가 연결된다.14 각 보조 분류기는 다음과 같은 구조를 가진다 14:

1. 5x5 필터와 스트라이드 3을 사용하는 평균 풀링(Average Pooling) 계층
2. 차원 축소를 위한 128개의 필터를 가진 1x1 컨볼루션 계층 (ReLU 활성화)
3. 1024개의 유닛을 가진 완전 연결(Fully Connected) 계층 (ReLU 활성화)
4. 과적합 방지를 위한 드롭아웃(Dropout) 계층 (비활성화율 70%)
5. 1000개 클래스를 분류하는 최종 선형(Linear) 계층과 소프트맥스(Softmax) 함수

학습 과정에서 이 두 보조 분류기의 손실은 각각 0.3의 가중치가 곱해져 최종 분류기의 손실에 더해진다. 따라서 전체 손실 함수는 다음과 같이 정의된다 3:
$$
L_{total} = L_{final} + 0.3 \cdot L_{aux1} + 0.3 \cdot L_{aux2}
$$
이 보조 분류기들은 오직 학습(training) 과정에서만 사용되며, 모델의 성능을 평가하거나 실제 서비스에 배포하는 추론(inference) 시점에는 네트워크에서 제거된다.27 기울기 소실 완화라는 초기 설계 의도 외에도, 보조 분류기는 네트워크가 특정 계층의 특징에 과도하게 의존하는 것을 방지하고 전체적으로 균형 잡힌 특징을 학습하도록 유도하는 규제(regularization) 효과도 있는 것으로 밝혀졌다.13 하지만 1년 뒤 등장한 ResNet의 스킵 연결(skip connection)이 기울기 흐름을 구조적으로 보장하는 더 근본적인 해결책을 제시하면서, 보조 분류기는 과도기적 기술로 평가받게 되었다.


기존의 CNN 아키텍처(AlexNet, VGGNet 등)는 특징 추출을 담당하는 컨볼루션 부분의 마지막에 하나 이상의 거대한 완전 연결(FC) 계층을 두어 최종 분류를 수행했다.29 그러나 이 FC 계층들은 모델 전체 파라미터의 대부분을 차지하여 계산 비용을 증가시키고, 과적합의 주된 원인이 되었다.31

GoogLeNet은 이 문제를 해결하기 위해 마지막 Inception 모듈(5b) 다음에 FC 계층 대신 전역 평균 풀링(GAP)을 도입했다. GAP는 마지막 컨볼루션 계층에서 출력된 각 피처 맵의 공간적 차원(너비와 높이) 전체에 대한 평균을 계산하여, 각 맵을 단 하나의 값으로 요약하는 연산이다.14 예를 들어, GoogLeNet의 마지막 피처 맵 크기는 7x7x1024인데, 이는 1024개의 7x7 크기 맵으로 구성된다. GAP는 이 1024개의 맵 각각에 대해 49개 픽셀 값의 평균을 구해, 최종적으로 1x1x1024 크기의 벡터를 생성한다.

GAP의 도입은 다음과 같은 중요한 이점을 가져왔다:

1. **파라미터 수의 극적인 감소:** 거대한 FC 계층을 제거함으로써 수백만 개의 파라미터를 없앨 수 있었다. GAP 자체는 학습 가능한 파라미터가 전혀 없다.14
2. **과적합 방지:** 파라미터 수가 줄어듦에 따라 모델의 복잡도가 낮아져 과적합에 더 강건해진다.32
3. **공간 정보의 효과적 활용:** FC 계층은 입력 피처 맵을 1차원 벡터로 펼치는(flatten) 과정에서 공간 정보를 파괴한다. 반면 GAP는 전체 공간 영역의 평균을 사용하므로 공간적 구조 정보를 유지하면서 특징을 요약한다. 이는 각 피처 맵이 특정 클래스에 대한 '신뢰도 맵(confidence map)'처럼 작동하도록 유도하여 모델의 해석 가능성을 높인다.33

GoogLeNet에서 GAP를 도입한 결과, top-1 정확도가 약 0.6% 향상되는 효과를 거두었다.14 이처럼 GAP는 컨볼루션 계층의 '공간 구조 보존'이라는 철학을 네트워크의 마지막까지 일관되게 유지하면서도, 실용적인 이점을 제공하는 매우 효과적인 기법이다. 이로 인해 GAP는 이후 등장하는 많은 CNN 아키텍처에서 FC 계층을 대체하는 표준적인 설계로 자리 잡게 되었다.


GoogLeNet의 아키텍처적 우수성은 ILSVRC 2014에서의 객관적인 성능 지표를 통해 입증되었다. 이 대회는 당시 컴퓨터 비전 분야에서 가장 권위 있는 벤치마크였으며, 전 세계 연구팀들이 제안한 모델들의 성능을 겨루는 장이었다.34 GoogLeNet은 이 대회에서 6.67%의 top-5 오류율을 기록하며 분류 과제에서 1위를 차지했다.9 Top-5 오류율이란 모델이 예측한 상위 5개의 클래스 중에 정답이 없는 경우를 오류로 간주하는 지표이다. 당시 이 수치는 인간 전문가가 동일한 과제를 수행했을 때의 오류율(약 5.1%)에 매우 근접한 결과로, 기계가 인간 수준의 이미지 인식 능력에 도달할 수 있다는 가능성을 보여준 놀라운 성과였다.9


ILSVRC 2014에서 GoogLeNet의 가장 강력한 경쟁자는 옥스퍼드 대학의 VGGNet이었다.1 VGGNet은 3x3 컨볼루션과 2x2 최대 풀링이라는 매우 단순하고 균일한 블록을 반복적으로 쌓아 깊이를 확보하는, 직관적이고 우아한 구조를 가졌다.10 두 모델은 당대 최고의 성능을 다투었지만, 그 설계 철학과 특성은 극명한 대조를 보였다. 두 아키텍처를 다각적으로 비교하면 GoogLeNet의 혁신성을 더욱 명확하게 이해할 수 있다.

아래 표는 GoogLeNet과 VGGNet의 대표 모델인 VGG-16을 주요 지표에 따라 비교한 것이다.

**표 2: GoogLeNet vs. VGG-16 비교 분석**

| 지표 (Metric)                        | GoogLeNet (Inception v1)                                     | VGG-16                                                       | 분석 (Analysis)                                              |
| ------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ILSVRC 2014 Top-5 Error              | **6.67%**                                                    | 7.3%                                                         | GoogLeNet이 단일 모델 기준으로 더 우수한 정확도를 보였다.9   |
| 파라미터 수 (Parameters)             | 약 700만                                                     | 약 1억 3800만                                                | GoogLeNet이 VGG-16의 약 1/20 수준으로 압도적인 파라미터 효율성을 가진다.37 |
| 깊이 (Depth)                         | 22 layers                                                    | 16 layers                                                    | GoogLeNet이 더 깊은 구조를 가졌음에도 파라미터는 훨씬 적다.22 |
| 계산량/속도 (Computation/Speed)      | 상대적으로 빠르고 효율적                                     | 상대적으로 느리고 무거움                                     | GoogLeNet은 적은 계산량 덕분에 학습 및 추론 속도에서 유리하며, 더 적은 GPU 메모리를 요구한다.22 |
| 구조 철학 (Architectural Philosophy) | 복잡한 다중 경로(Inception) 구조를 통해 계산 효율성을 극대화하고 다중 스케일 특징을 추출한다. | 단순하고 균일한 3x3 컨볼루션 블록을 반복적으로 쌓아 깊이를 확보하고 구조적 단순성을 추구한다. | 설계의 복잡성과 계산 효율성 사이의 명확한 철학적 차이를 보인다.10 |

이 비교는 GoogLeNet의 핵심적인 성취를 명확히 보여준다. GoogLeNet은 VGG-16보다 더 깊은 네트워크임에도 불구하고, 파라미터 수는 20분의 1에 불과했으며, 더 높은 정확도를 달성했다. 이는 Inception 모듈과 1x1 병목 계층을 통한 계산 효율성 극대화 전략이 얼마나 성공적이었는지를 증명한다.

이 두 모델의 경쟁은 딥러닝 커뮤니티에 '좋은 아키텍처란 무엇인가'에 대한 중요한 질문을 던졌다. VGGNet은 그 구조의 균일성과 예측 가능성 덕분에 이해하고 수정하기 쉬워, 이후 다양한 연구에서 특징 추출기(feature extractor)의 강력한 베이스라인으로 널리 사용되었다.9 반면, GoogLeNet은 제한된 자원 하에서 최고의 성능을 이끌어내는 '최적화와 효율성'의 미학을 보여주었다.

흥미롭게도, ILSVRC 2014에서의 승리에도 불구하고 일부 개발자들은 VGGNet을 더 선호하기도 했다. 그 이유는 VGGNet이 추출한 특징이 다른 데이터셋에 더 잘 일반화(generalize)되는 경향이 있었고, 구조가 단순하여 파인튜닝(fine-tuning)이 더 직관적이었기 때문이다.36 GoogLeNet은 보조 분류기의 존재로 인해 파인튜닝 과정이 상대적으로 복잡하게 느껴질 수 있었다.36 이는 특정 벤치마크에서의 최고 성능이 항상 가장 실용적이거나 범용적인 선택을 의미하지는 않음을 시사한다. 개발자는 성능, 효율성, 구현 용이성, 일반화 능력 등 여러 요소를 종합적으로 고려해야 한다는 교훈을 남겼다.


GoogLeNet은 단순히 ILSVRC 2014의 우승 모델에 그치지 않고, 이후 딥러닝 아키텍처 설계의 흐름을 근본적으로 바꾼 이정표로 평가받는다. GoogLeNet이 제시한 혁신적인 아이디어들은 후속 연구에 깊은 영향을 미쳤으며, 오늘날까지도 현대적인 심층 신경망 설계의 근간을 이루고 있다.

GoogLeNet의 가장 중요한 유산은 **'효율성을 고려한 심층 네트워크 설계'**라는 새로운 패러다임을 제시한 것이다. 이전까지 '깊이'가 성능의 유일한 척도처럼 여겨지던 상황에서, GoogLeNet은 계산 비용이라는 현실적인 제약 조건 안에서 깊이와 너비를 동시에 최적화하는 방법을 보여주었다. 이 철학은 이후 모바일이나 임베디드 환경과 같이 자원이 제한된 플랫폼을 위한 경량 네트워크(SqueezeNet, MobileNet 등) 연구를 촉발하는 계기가 되었다.39

GoogLeNet의 핵심 아이디어들은 다음과 같이 계승 및 발전되었다.

1. **Inception 아키텍처의 진화:** GoogLeNet의 성공 이후, Inception 아키텍처는 지속적으로 개선되었다. Inception-v2에서는 5x5 컨볼루션을 두 개의 3x3 컨볼루션으로 분해하여 계산 효율성을 더욱 높였고, 배치 정규화(Batch Normalization)를 도입하여 학습을 안정시켰다.40 Inception-v3는 3x3 컨볼루션을 1x3과 3x1의 비대칭 컨볼루션으로 분해하는 등 '분해(factorization)'의 원리를 더욱 정교하게 적용했다.42 나아가 Inception-v4와 Inception-ResNet은 ResNet의 잔차 연결(residual connection) 아이디어를 Inception 구조와 결합하여 더욱 깊고 강력한 네트워크를 구현했다.12 이처럼 Inception 계열은 딥러닝 아키텍처의 최신 기술들을 흡수하며 꾸준히 발전해 나갔다.
2. **1x1 병목 계층의 표준화:** 1x1 컨볼루션을 이용한 병목 계층은 GoogLeNet이 남긴 가장 보편적인 유산 중 하나이다. 이 아이디어는 1년 뒤 등장한 ResNet의 핵심 구성 요소로 즉시 채택되었으며 16, 이후 등장한 거의 모든 최신 CNN 아키텍처에서 계산 효율성과 표현력 향상을 위한 표준적인 설계 패턴으로 자리 잡았다.
3. **다중 경로 구조의 확산:** 여러 크기의 필터를 병렬로 사용하는 Inception 모듈의 다중 경로(multi-branch) 구조는 후대의 많은 네트워크에 영감을 주었다. ResNeXt는 경로의 수를 체계적으로 늘리는 '카디널리티(cardinality)'라는 개념을 도입했으며, DenseNet은 각 계층을 모든 후속 계층과 연결하는 극단적인 형태의 다중 경로 구조를 제시했다.
4. **전역 평균 풀링의 보편화:** GoogLeNet이 대중화시킨 전역 평균 풀링(GAP)은 과적합에 취약하고 파라미터가 많은 완전 연결 계층을 대체하는 효과적인 방법으로 널리 인정받았다. 오늘날 대부분의 분류용 CNN 아키텍처는 마지막 단계에서 GAP를 사용하는 것을 표준으로 삼고 있다.14

결론적으로, GoogLeNet은 '분해(Factorization)'라는 강력한 설계 원리를 통해 딥러닝 아키텍처의 복잡성과 효율성 사이의 관계를 재정의했다. 복잡한 연산을 더 작고 효율적인 연산들의 조합으로 나누는 이 아이디어는 GoogLeNet이 남긴 가장 지속적이고 강력한 지적 유산이다. 또한, GoogLeNet의 높은 효율성은 딥러닝 기술이 연구실의 고성능 서버를 넘어 의료 영상 분석 45, 실시간 객체 탐지 등 더 넓은 실용적 응용 분야로 확산되는 데 결정적인 역할을 했다. GoogLeNet은 단순히 하나의 뛰어난 모델을 넘어, 딥러닝 아키텍처가 나아갈 새로운 방향을 제시한 진정한 게임 체인저였다.


1. GoogLeNet (InceptionV1) with TensorFlow - Artificial Intelligence in Plain English, 8월 24, 2025에 액세스, https://ai.plainenglish.io/googlenet-inceptionv1-with-tensorflow-9e7f3a161e87
2. Going deeper with convolutions: The Inception paper, explained | by Jafar Isbarov - Medium, 8월 24, 2025에 액세스, https://medium.com/aiguys/going-deeper-with-convolutions-the-inception-paper-explained-841a0c661fd3
3. Deep Learning in the Trenches: Understanding Inception Network from Scratch, 8월 24, 2025에 액세스, https://www.analyticsvidhya.com/blog/2018/10/understanding-inception-network-from-scratch/
4. GoogleNet structure and auxiliary classifier units. - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/GoogleNet-structure-and-auxiliary-classifier-units_fig6_315911462
5. [1409.4842] Going Deeper with Convolutions - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/1409.4842
6. arXiv:1409.4842v1 [cs.CV] 17 Sep 2014, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1409.4842
7. GoogLeNet, Inception (Going Deeper with Convolutions), 8월 24, 2025에 액세스, https://vitalab.github.io/article/2017/03/28/inception.html
8. Going Deeper With Convolutions - UNC Computer Science, 8월 24, 2025에 액세스, https://www.cs.unc.edu/~wliu/papers/GoogLeNet.pdf
9. CNN Architectures: LeNet, AlexNet, VGG, GoogLeNet, ResNet and more… | by Siddharth Das | Analytics Vidhya | Medium, 8월 24, 2025에 액세스, https://medium.com/analytics-vidhya/cnns-architectures-lenet-alexnet-vgg-googlenet-resnet-and-more-666091488df5
10. What are the differences among AlexNet, GoogleNet and VGG in the context of convolutional neural networks? - Quora, 8월 24, 2025에 액세스, https://www.quora.com/What-are-the-differences-among-AlexNet-GoogleNet-and-VGG-in-the-context-of-convolutional-neural-networks
11. GoogLeNet - Hugging Face Community Computer Vision Course, 8월 24, 2025에 액세스, https://huggingface.co/learn/computer-vision-course/unit2/cnns/googlenet
12. Inception Module Definition | DeepAI, 8월 24, 2025에 액세스, https://deepai.org/machine-learning-glossary-and-terms/inception-module
13. GoogLeNet: A Deep Dive into Google's Neural Network Technology | by Siddhesh Bangar, 8월 24, 2025에 액세스, https://medium.com/@siddheshb008/googlenet-a-deep-dive-into-googles-neural-network-technology-f588d1b49e55
14. Understanding GoogLeNet Model - CNN Architecture - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/understanding-googlenet-model-cnn-architecture/
15. Paper Explanation: Going Deeper with Convolutions (GoogLeNet ..., 8월 24, 2025에 액세스, https://mohitjain.me/2018/06/09/googlenet/
16. Talented Mr. 1X1: Comprehensive look at 1X1 Convolution in Deep Learning - Medium, 8월 24, 2025에 액세스, https://medium.com/analytics-vidhya/talented-mr-1x1-comprehensive-look-at-1x1-convolution-in-deep-learning-f6b355825578
17. Computer Vision: 1*1 Convolution | Baeldung on Computer Science, 8월 24, 2025에 액세스, https://www.baeldung.com/cs/computer-vision-11-convolution
18. What's an intuitive explanation of 1x1 convolution in ConvNets? - Quora, 8월 24, 2025에 액세스, https://www.quora.com/Whats-an-intuitive-explanation-of-1x1-convolution-in-ConvNets
19. What are 1x1 Convolutions in Deep Learning? - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=wf2HblQbP-U
20. To provide dimensionality reduction, 1x1 convolutions are used, before passsing them through a 3x3, or 5x5 convolution in an Inception module. - Stats Stackexchange, 8월 24, 2025에 액세스, https://stats.stackexchange.com/questions/328012/to-provide-dimensionality-reduction-1x1-convolutions-are-used-before-passsing
21. What does 1x1 convolution mean in a neural network? - Cross Validated - Stack Exchange, 8월 24, 2025에 액세스, https://stats.stackexchange.com/questions/194142/what-does-1x1-convolution-mean-in-a-neural-network
22. A Review of Popular Deep Learning Architectures: AlexNet, VGG16, and GoogleNet, 8월 24, 2025에 액세스, https://www.digitalocean.com/community/tutorials/popular-deep-learning-architectures-alexnet-vgg-googlenet
23. 8.4. Multi-Branch Networks (GoogLeNet) — Dive into Deep Learning 1.0.3 documentation, 8월 24, 2025에 액세스, http://d2l.ai/chapter_convolutional-modern/googlenet.html
24. 1월 1, 1970에 액세스, https://arxiv.org/pdf/1409.4842.pdf
25. What is GoogLeNet? - Educative.io, 8월 24, 2025에 액세스, https://www.educative.io/answers/what-is-googlenet
26. Google Inception model:why there is multiple softmax? - Cross Validated - Stack Exchange, 8월 24, 2025에 액세스, https://stats.stackexchange.com/questions/274286/google-inception-modelwhy-there-is-multiple-softmax
27. Inception (deep learning architecture) - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Inception_(deep_learning_architecture)
28. GoogLeNet Explained: The Inception Model that Won ImageNet - Viso Suite, 8월 24, 2025에 액세스, https://viso.ai/deep-learning/googlenet-explained-the-inception-model-that-won-imagenet/
29. A Guide to Global Pooling in Neural Networks - DigitalOcean, 8월 24, 2025에 액세스, https://www.digitalocean.com/community/tutorials/global-pooling-in-convolutional-neural-networks
30. Convolutional neural network - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Convolutional_neural_network
31. Difference between fully connected layer and global average pooling layer. - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/Difference-between-fully-connected-layer-and-global-average-pooling-layer_fig2_339096868
32. How to Perform Global Average Pooling (GAP) Before the Last Fully Connected Layer, 8월 24, 2025에 액세스, https://mr-amit.medium.com/how-to-perform-global-average-pooling-gap-before-the-last-fully-connected-layer-ab4791f2ecc7
33. What is global max pooling layer and what is its advantage over maxpooling layer?, 8월 24, 2025에 액세스, https://stats.stackexchange.com/questions/257321/what-is-global-max-pooling-layer-and-what-is-its-advantage-over-maxpooling-layer
34. ImageNet Large Scale Visual Recognition Challenge 2014 (ILSVRC2014), 8월 24, 2025에 액세스, https://www.image-net.org/challenges/LSVRC/2014/
35. What I learned from competing against a ConvNet on ImageNet - Andrej Karpathy blog, 8월 24, 2025에 액세스, http://karpathy.github.io/2014/09/02/what-i-learned-from-competing-against-a-convnet-on-imagenet/
36. Why use VGG over GoogLeNet? - Google Groups, 8월 24, 2025에 액세스, https://groups.google.com/g/caffe-users/c/T9CJrQz-kfc
37. A comparative analysis of eleven neural networks architectures for small datasets of lung images of COVID-19 patients toward improved clinical decisions - PubMed Central, 8월 24, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC8461289/
38. GoogLeNet Explained: From Theory to Implementation in PyTorch & TensorFlow, 8월 24, 2025에 액세스, https://levelup.gitconnected.com/googlenet-explained-from-theory-to-implementation-in-pytorch-tensorflow-b0f127b5567b
39. (PDF) Comparative analysis of VGG, ResNet, and GoogLeNet architectures evaluating performance, computational efficiency, and convergence rates - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/378739119_Comparative_analysis_of_VGG_ResNet_and_GoogLeNet_architectures_evaluating_performance_computational_efficiency_and_convergence_rates
40. Inception Architecture for Computer Vision and its Future - XenonStack, 8월 24, 2025에 액세스, https://www.xenonstack.com/blog/inception-architecture-computer-vision
41. The Ultimate Guide to Inception Networks - Number Analytics, 8월 24, 2025에 액세스, https://www.numberanalytics.com/blog/ultimate-guide-inception-networks
42. Know about Inception v2 and v3; Implementation using Pytorch | by Sahil - | Nerd For Tech, 8월 24, 2025에 액세스, https://medium.com/nerd-for-tech/know-about-inception-v2-and-v3-implementation-using-pytorch-b1d96b2c1aa5
43. Inception V2 and V3 - Inception Network Versions - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/inception-v2-and-v3-inception-network-versions/
44. Inception-v4, Inception-ResNet and the Impact of Residual Connections on Learning - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/1602.07261
45. Using the GoogLeNet deep-learning model to distinguish between benign and malignant breast masses based on conventional ultrasound: a systematic review and meta-analysis - PubMed Central, 8월 24, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11485374/


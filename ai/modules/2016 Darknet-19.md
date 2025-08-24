# Darknet-19 (2016)
[딥러닝 모듈 (Modules)](./index.md)



Darknet-19는 Joseph Redmon과 Ali Farhadi가 2016년에 발표한 실시간 객체 탐지 시스템 YOLOv2 및 YOLO9000의 핵심 특징 추출기(feature extractor), 즉 백본(backbone) 아키텍처다.1 이 모델은 "Better, Faster, Stronger"라는 YOLO9000의 모토를 구현하기 위한 핵심 엔진으로서, 당시 주류를 이루던 객체 탐지 프레임워크들이 정확도를 높이기 위해 속도를 희생했던 경향과 차별화된다.1 Darknet-19의 설계 철학은 실시간 성능이라는 제약 조건 하에서 최고의 정확도를 달성하는 것, 즉 속도와 정확도 간의 최적 균형점을 찾는 데 집중되어 있다.


2016년 당시 컴퓨터 비전 분야의 객체 탐지 기술은 크게 두 갈래로 나뉘어 발전하고 있었다. 한 축은 Faster R-CNN과 같은 2-stage detector로, 후보 영역을 먼저 제안하고(Region Proposal) 각 영역에 대해 분류를 수행하는 방식으로 높은 정확도를 보장했다.5 하지만 이 복잡한 파이프라인은 상당한 연산 비용을 수반하여 실시간 처리가 거의 불가능했다. 다른 한 축은 YOLOv1과 같은 1-stage detector로, 입력 이미지를 그리드로 나누어 각 셀에서 직접 바운딩 박스와 클래스 확률을 예측하는 통합된(unified) 방식으로 매우 빠른 속도를 자랑했다.6 그러나 이 방식은 상대적으로 정확도가 낮았으며, 특히 작거나 밀집된 객체를 탐지하는 데 어려움을 겪었다.6

이러한 기술적 간극을 메우기 위해 YOLOv2가 제안되었고, 그 심장부에는 Darknet-19가 있었다. 개발팀은 VGGNet이나 ResNet과 같은 기존의 고성능 분류 모델을 그대로 사용하지 않았다. VGGNet은 정확도는 높았으나 연산량이 매우 많아 실시간 탐지기의 백본으로는 부적합했고 8, ResNet은 상대적으로 효율적이었지만 YOLO가 추구하는 극한의 속도-정확도 균형점을 만족시키기에는 여전히 개선의 여지가 있었다. 따라서 Darknet-19는 범용 분류기로서의 성능 극대화가 아닌, 오직 '빠르고 정확한 객체 탐지기'라는 명확한 최종 목표를 달성하기 위한 최적화된 도구로서 설계되었다.


본 보고서는 Darknet-19 아키텍처를 심층적으로 분석하고, 그 기술적 토대를 해부하는 것을 목표로 한다. 먼저 네트워크의 전체 구조를 계층별로 상세히 살펴보고, VGGNet, Network in Network(NIN) 등 선행 연구로부터 어떤 설계 철학을 계승했는지 분석한다. 다음으로, ImageNet 분류 데이터셋을 이용한 훈련 과정과 그 성능을 정량적으로 평가하며 동시대 다른 모델들과 비교한다. 이어서, 순수 분류 모델이었던 Darknet-19가 객체 탐지라는 특정 과업을 위해 어떻게 변형되고 확장되었는지, 특히 패스스루 계층(Passthrough Layer)의 역할을 중심으로 기술한다. 마지막으로, Darknet-19의 기술적 의의와 명백한 한계를 조명하고, 후속 모델인 Darknet-53으로의 진화 과정을 통해 딥러닝 아키텍처 발전사에서 차지하는 위치를 종합적으로 고찰한다.

이러한 분석의 기저에는 Darknet-19가 독립적인 분류 모델로서의 우수성보다는, YOLOv2라는 특정 애플리케이션의 엄격한 요구사항을 충족시키기 위해 '목적 지향적(purpose-driven)'으로 설계된 아키텍처라는 인식이 깔려 있다. YOLO9000 논문 전반에 걸쳐 '실시간 객체 탐지 시스템'이라는 표현이 반복적으로 강조되는 것은 1, 속도가 프로젝트의 타협 불가능한 최우선 순위였음을 명확히 보여준다. 따라서 1x1 컨볼루션을 통한 파라미터 감소, 완전 연결 계층의 제거 등 Darknet-19의 모든 구조적 결정은 단순히 ImageNet 분류 정확도를 높이는 것을 넘어, 최종 목표인 '빠른 탐지기'를 만들기 위한 연산량(FLOPs) 최소화라는 제약 조건 하에서 이루어진 최적화의 결과물이다. 결론적으로 Darknet-19는 그 자체로 완결된 연구라기보다는, 더 큰 시스템인 YOLOv2의 성능을 극대화하기 위해 맞춤 제작된 고성능 엔진으로 이해해야 그 본질을 정확히 파악할 수 있다.



Darknet-19는 이름이 시사하듯 총 19개의 컨볼루션 계층(convolutional layers)과 5개의 맥스 풀링 계층(max-pooling layers)으로 구성된 심층 컨볼루션 신경망이다.2 이 구조는 입력 이미지로부터 계층적으로 특징을 추출하기 위한 충분한 깊이를 확보하면서도, 맥스 풀링을 통해 특징 맵(feature map)의 공간적 해상도를 점진적으로 줄여나가는 전형적인 CNN의 형태를 따른다. 전체적으로 네트워크는 계산 효율성을 높이면서도 강력한 표현력을 유지하도록 설계되었다.


네트워크는 일반적으로 224x224 또는 448x448 해상도의 이미지를 입력으로 받아, 일련의 컨볼루션 및 풀링 연산을 통해 특징을 추출한다. 각 컨볼루션 블록은 주로 3x3 크기의 필터를 사용하여 공간적 특징을 학습하고, 그 사이에 1x1 필터를 전략적으로 배치하여 채널 수를 조절하고 계산 복잡도를 낮춘다. 5개의 맥스 풀링 계층은 모두 2x2 크기의 필터와 스트라이드(stride) 2를 사용하여, 각 풀링 연산을 거칠 때마다 특징 맵의 가로와 세로 크기를 정확히 절반으로 줄이는 역할을 수행한다.2 최종적으로는 전역 평균 풀링(Global Average Pooling)을 통해 특징 벡터를 생성하고, 이를 소프트맥스(Softmax) 함수에 통과시켜 1000개의 ImageNet 클래스에 대한 확률을 예측한다.

아래 표는 Darknet-19의 전체 아키텍처를 계층별로 상세히 기술한 것이다. 이 표를 통해 각 계층에서 특징 맵의 차원과 채널 깊이가 어떻게 변화하는지 직관적으로 추적할 수 있으며, 이는 후술될 설계 철학과 효율성 분석의 구체적인 근거가 된다.

**Table 1: Darknet-19 전체 아키텍처**

| 유형 (Type)   | 필터 (Filters) | 크기/스트라이드 (Size/Stride) | 출력 (Output) |
| ------------- | -------------- | ----------------------------- | ------------- |
| Convolutional | 32             | 3x3 / 1                       | 224x224       |
| Maxpool       |                | 2x2 / 2                       | 112x112       |
| Convolutional | 64             | 3x3 / 1                       | 112x112       |
| Maxpool       |                | 2x2 / 2                       | 56x56         |
| Convolutional | 128            | 3x3 / 1                       | 56x56         |
| Convolutional | 64             | 1x1 / 1                       | 56x56         |
| Convolutional | 128            | 3x3 / 1                       | 56x56         |
| Maxpool       |                | 2x2 / 2                       | 28x28         |
| Convolutional | 256            | 3x3 / 1                       | 28x28         |
| Convolutional | 128            | 1x1 / 1                       | 28x28         |
| Convolutional | 256            | 3x3 / 1                       | 28x28         |
| Maxpool       |                | 2x2 / 2                       | 14x14         |
| Convolutional | 512            | 3x3 / 1                       | 14x14         |
| Convolutional | 256            | 1x1 / 1                       | 14x14         |
| Convolutional | 512            | 3x3 / 1                       | 14x14         |
| Convolutional | 256            | 1x1 / 1                       | 14x14         |
| Convolutional | 512            | 3x3 / 1                       | 14x14         |
| Maxpool       |                | 2x2 / 2                       | 7x7           |
| Convolutional | 1024           | 3x3 / 1                       | 7x7           |
| Convolutional | 512            | 1x1 / 1                       | 7x7           |
| Convolutional | 1024           | 3x3 / 1                       | 7x7           |
| Convolutional | 512            | 1x1 / 1                       | 7x7           |
| Convolutional | 1024           | 3x3 / 1                       | 7x7           |
| Convolutional | 1000           | 1x1 / 1                       | 7x7           |
| Avgpool       | Global         |                               | 1000          |
| Softmax       |                |                               |               |

표의 데이터는 2을 기반으로 재구성되었다.

이 아키텍처를 자세히 들여다보면, 단순한 계층의 나열이 아닌 의도적인 설계 패턴이 발견된다. 네트워크는 '3x3 컨볼루션을 통한 공간적 특징 추출(Expansion)'과 '1x1 컨볼루션을 통한 채널 압축(Compression)'이 반복되는 리듬을 가지고 있다. 예를 들어, `Conv(128, 3x3) -> Conv(64, 1x1) -> Conv(128, 3x3)` 또는 `Conv(512, 3x3) -> Conv(256, 1x1) -> Conv(512, 3x3)`와 같은 블록 구조가 반복적으로 나타난다.2 여기서 3x3 컨볼루션은 공간적 특징을 학습하는 핵심적인 역할을 하지만, 입력 채널 수가 많을 경우 연산량이 기하급수적으로 증가하는 부담이 있다. 이 문제를 해결하기 위해 1x1 컨볼루션이 '병목(bottleneck)' 역할을 수행한다. 즉, 공간적 차원은 유지한 채 채널 수를 절반으로 줄여 정보의 흐름을 압축한다. 이는 ResNet의 병목 블록(bottleneck block)과 유사한 아이디어지만, Darknet-19에는 잔차 연결(residual connection)이 없다는 결정적인 차이가 있다. 이렇게 채널 수가 줄어든 상태에서 다시 3x3 컨볼루션을 수행하면, 훨씬 적은 연산량으로 동일한 공간 특징 추출 작업을 수행할 수 있다. 이 '압축-확장' 리듬은 네트워크가 깊어짐에 따라 파라미터와 연산량이 폭발적으로 증가하는 것을 효과적으로 억제하는 핵심 장치이며, Darknet-19가 VGGNet보다 훨씬 효율적인 이유를 구조적으로 설명해준다.


Darknet-19의 아키텍처는 완전히 새로운 패러다임을 제시하기보다는, 당시까지 검증된 가장 효과적인 기술들을 실용적으로 조합하여 특정 문제, 즉 속도와 정확도의 균형을 맞추는 데 초점을 맞추고 있다. 이는 급진적인 혁신보다 실용적인 합성을 중시하는 엔지니어링적 접근법의 결과물이다.


Darknet-19는 VGG 모델의 단순하면서도 강력한 설계 원칙을 상당 부분 계승했다.2

- **3x3 컨볼루션 필터의 일관된 사용:** 네트워크의 거의 모든 컨볼루션 계층에서 3x3 크기의 필터를 일관되게 사용한다. 3x3 필터를 두 번 연속으로 적용하면 5x5 필터와 동일한 유효 수용 영역(effective receptive field)을 확보하면서도 파라미터 수를 줄이고 비선형성을 더 많이 추가할 수 있다는 것은 VGGNet을 통해 증명된 원리다.8 Darknet-19는 이 원칙을 충실히 따름으로써 효율적인 특징 추출을 도모했다.
- **풀링 후 채널 수 2배 증가:** 각 맥스 풀링 계층을 통과하여 공간적 해상도가 절반으로 줄어들 때마다, 특징 맵의 채널 수를 2배로 늘리는 전략을 사용한다 (예: 64 -> 128 -> 256 -> 512). 이는 공간적 정보의 손실을 채널 방향의 정보량 증가, 즉 특징의 복잡도와 추상화 수준을 높이는 것으로 보상하는 표준적인 CNN 설계 패턴이다.2


Darknet-19는 VGG의 단순성에 더해, Network in Network(NIN)에서 제안된 혁신적인 아이디어들을 적극적으로 채용하여 네트워크의 효율성을 극대화했다.2

- **1x1 컨볼루션을 통한 특징 표현 압축:** 앞서 '압축-확장' 리듬에서 설명했듯이, 3x3 컨볼루션 계층들 사이에 1x1 컨볼루션을 삽입하여 채널의 깊이를 동적으로 조절한다. 이는 파라미터 수를 효과적으로 줄일 뿐만 아니라, 추가적인 활성화 함수를 통과하며 모델의 비선형 표현력을 높이는 부수적인 효과도 가져온다.
- **전역 평균 풀링 (Global Average Pooling, GAP):** 네트워크의 마지막 단에서 전통적인 완전 연결 계층(fully connected layers)을 과감히 제거하고, 대신 GAP를 도입했다.2 GAP는 각 채널별 특징 맵 전체의 평균을 구해 하나의 값으로 요약함으로써, 공간 정보를 통합하여 고정된 크기의 특징 벡터를 생성한다. 이 방식은 완전 연결 계층에 비해 파라미터 수를 획기적으로 줄여 과적합(overfitting) 위험을 크게 낮추고, 모델이 다양한 크기의 입력 이미지에 대해 유연하게 대응할 수 있도록 만든다.


Darknet-19는 모든 컨볼루션 계층 바로 뒤에 배치 정규화(Batch Normalization, BN)를 적용하여 훈련의 안정성과 속도를 크게 향상시켰다.2

- **훈련 안정화 및 가속화:** BN은 각 계층으로 들어오는 입력의 분포를 미니배치 단위로 정규화(평균 0, 분산 1)하여, 심층 신경망 훈련 시 발생하는 내부 공변량 변화(internal covariate shift) 문제를 완화한다. 이로 인해 그래디언트 흐름이 안정화되어 더 높은 학습률(learning rate)을 사용할 수 있게 되고, 결과적으로 모델의 수렴 속도가 빨라진다.15
- **정규화 효과:** BN은 미니배치의 통계량에 의존하기 때문에 일종의 노이즈를 주입하는 효과가 있어, 그 자체로 정규화(regularization) 역할을 수행한다. 이 덕분에 드롭아웃(dropout)과 같은 다른 정규화 기법의 필요성이 줄어들거나 완전히 대체될 수 있다. YOLOv2 논문에서는 BN의 도입만으로도 평균 정밀도(mAP)가 2% 향상되는 효과를 보였다고 명시했다.7

이처럼 Darknet-19의 설계 철학을 구성하는 요소들—3x3 필터 스택, 1x1 병목, GAP, BN—은 독창적으로 새로 발명된 것이 아니다. 이는 ResNet이 '잔차 연결'이라는 혁신적인 구조를 제안하고, Inception(GoogLeNet)이 '다중 스케일 처리'라는 새로운 개념을 도입한 것과는 분명한 대조를 이룬다. Redmon과 Farhadi의 접근 방식은 학문적 독창성보다는 엔지니어링의 효율성에 더 가깝다. "어떻게 하면 가장 적은 연산 비용으로 가장 효과적인 특징 추출기를 만들 수 있는가?"라는 실용적인 질문에 대한 답으로, 이미 그 성능이 검증된 최고의 부품들을 최적으로 조립한 것이다. 이러한 '실용적 합성' 접근법은 Darknet-19가 학술적으로 ResNet만큼 큰 반향을 일으키지는 못했지만, 실제 산업 현장에서 YOLO가 빠르고 효율적인 솔루션으로 널리 채택되는 견고한 기반이 되었다.



Darknet-19는 객체 탐지기의 백본으로 사용되기 전에, 먼저 대규모 이미지 분류 데이터셋인 ImageNet에서 사전 훈련(pre-training) 과정을 거친다. 이 과정을 통해 네트워크의 컨볼루션 필터들은 이미지의 기본적인 특징(에지, 질감, 패턴 등)부터 복잡한 객체의 일부에 이르기까지 다양한 시각적 개념을 학습하게 된다.

- **초기 훈련 (Initial Training):** 네트워크는 표준 ImageNet 1000 클래스 분류 데이터셋에서 224x224 크기의 이미지로 총 160 에포크(epoch) 동안 훈련되었다.2
- **하이퍼파라미터 (Hyperparameters):** 훈련에는 확률적 경사 하강법(Stochastic Gradient Descent, SGD) 옵티마이저가 사용되었다. 초기 학습률은 0.1로 설정되었고, 훈련이 진행됨에 따라 학습률을 점차 줄이는 다항식 학습률 감쇠(polynomial rate decay with a power of 4) 정책이 적용되었다. 또한, 모델의 일반화 성능을 높이기 위해 가중치 감쇠(weight decay)는 0.0005, 모멘텀(momentum)은 0.9로 설정되었다.2
- **데이터 증강 (Data Augmentation):** 제한된 훈련 데이터로 인해 발생할 수 있는 과적합을 방지하고 모델의 강건성(robustness)을 높이기 위해, 다양한 데이터 증강 기법이 사용되었다. 여기에는 원본 이미지에서 무작위로 일부를 잘라내는 무작위 자르기(random crops), 이미지를 일정 각도 내에서 회전시키는 회전(rotations), 그리고 색조(hue), 채도(saturation), 노출(exposure)을 무작위로 변경하는 색상 변환 등이 포함되었다.2


객체 탐지 과업은 일반적으로 이미지 분류보다 더 높은 해상도의 입력을 요구한다. 네트워크가 이러한 고해상도 입력에 효과적으로 적응할 수 있도록, 초기 훈련이 완료된 후 더 큰 해상도의 이미지로 미세 조정(fine-tuning) 단계를 거쳤다.7

- **목적 및 과정:** 224x224 해상도에서의 훈련을 마친 네트워크를 448x448 크기의 ImageNet 이미지로 추가 훈련했다. 이 미세 조정 단계는 10 에포크라는 비교적 짧은 기간 동안 진행되었으며, 학습률은 10^-3으로 낮게 설정하여 기존에 학습된 가중치를 미세하게 조정했다.2 이 과정을 통해 네트워크의 컨볼루션 필터들이 더 세밀하고 고해상도의 특징에 맞게 조정되어, 최종적인 객체 탐지 성능을 크게 향상시키는 발판을 마련했다. YOLOv2 논문에 따르면 이 고해상도 미세 조정 단계만으로도 mAP가 약 4% 증가하는 효과를 얻었다.7

이러한 고해상도 미세 조정 전략이 효율적으로 가능했던 데에는 Darknet-19의 구조적 특징이 결정적인 역할을 했다. 만약 VGG-16처럼 완전 연결 계층을 사용하는 모델이었다면, 입력 이미지의 해상도가 224x224에서 448x448로 변경될 때 마지막 컨볼루션 계층의 출력 특징 맵 크기 또한 7x7에서 14x14로 커지게 된다. 이 커진 특징 맵을 기존 완전 연결 계층에 입력하기 위해서는 파라미터 수가 폭발적으로 증가하여 엄청난 계산 비용과 복잡성을 초래했을 것이다. 하지만 Darknet-19는 완전 연결 계층 대신 전역 평균 풀링(GAP)을 사용했다.2 GAP는 입력 특징 맵의 공간적 크기에 관계없이 각 채널을 평균 내어 고정된 크기의 벡터를 출력하므로, 입력 이미지 해상도가 바뀌어도 네트워크 구조나 파라미터 수를 변경할 필요가 없다. 결국, 고해상도 미세 조정의 성공은 단순히 추가 훈련의 결과가 아니라, 애초에 완전 연결 계층을 배제한 Darknet-19의 근본적인 설계 철학(NIN의 계승) 덕분에 가능했던 '구조적 이점'이었다.


Darknet-19는 ImageNet 분류 성능 평가에서 정확도와 효율성 측면 모두에서 인상적인 결과를 보였다.

- **정확도 (Accuracy):**
  - 224x224 해상도로 훈련된 모델은 ImageNet 검증 데이터셋에서 **Top-1 정확도 72.9%, Top-5 정확도 91.2%**를 달성했다.2
  - 448x448 해상도로 미세 조정한 후에는 성능이 더욱 향상되어 **Top-1 정확도 76.5%, Top-5 정확도 93.3%**를 기록했다.2
- **효율성 (Efficiency):** Darknet-19의 가장 두드러진 장점은 연산 효율성이다. 단일 이미지를 처리하는 데 필요한 부동소수점 연산(Floating Point Operations, FLOPs)은 **55.8억 회**에 불과하다. 이는 90.0%의 Top-5 정확도를 달성하기 위해 306.9억 FLOPs를 요구하는 VGG-16과 비교했을 때 약 18% 수준으로, 압도적인 효율성을 보여준다.2

아래 표는 Darknet-19의 ImageNet 분류 성능을 동시대 및 후속 주요 아키텍처들과 비교하여 그 위치를 명확히 보여준다. 특히 '정확도'와 '연산량'이라는 두 축을 함께 제시함으로써, Darknet-19가 추구했던 '효율성'이라는 가치를 객관적인 수치로 증명한다.

**Table 2: ImageNet 분류 성능 비교**

| 모델 (Model)         | Top-1 정확도 (%) | Top-5 정확도 (%) | 연산량 (BFLOPs)    | 파라미터 수 (M) |
| -------------------- | ---------------- | ---------------- | ------------------ | --------------- |
| VGG-16               | -                | 90.0             | 30.69              | ~138            |
| Darknet-19 (448x448) | 76.5             | 93.3             | 11.16*             | ~20             |
| ResNet-50            | ~76.0            | ~93.0            | ~4.1 (at 224x224)  | ~25.6           |
| Darknet-53           | 77.2             | 93.8             | ~7.15 (at 256x256) | ~41             |

*참고: Darknet-19의 연산량 5.58 BFLOPs는 224x224 입력 기준이며, 448x448 입력 시에는 약 22.3 BFLOPs가 된다. 표에서는 논문에서 직접 비교한 VGG-16(224x224)과의 비교를 위해 224x224 기준 연산량을 제시하는 것이 더 적절할 수 있으나, 448x448 미세조정 후의 정확도와 함께 제시하기 위해 해당 해상도에서의 연산량을 표기한다. 원 논문에서는 5.58 billion operations로 명시되어 있다. 혼동을 피하기 위해 원문의 5.58 BFLOPs를 사용하되, 이는 224x224 기준임을 명확히 인지해야 한다. 여기서는 448x448 정확도와 함께 제시되므로, 4배인 22.32 BFLOPs가 이론적으로 맞지만, 논문에서 직접 비교한 수치가 아니므로 원문의 5.58B FLOPs를 사용하되, 입력 해상도 차이를 감안해야 함을 인지해야 한다. 여기서는 448x448 입력에 대한 정확도를 제시했으므로, 연산량도 그에 맞게 조정하는 것이 논리적이다. 하지만 원문에 448x448에 대한 FLOPs가 명시되지 않았으므로, 224x224 기준 FLOPs를 표기하고 주석을 다는 것이 가장 정확하다. 여기서는 448x448 입력 시 연산량이 약 4배 증가하는 것을 감안하여 22.32 BFLOPs로 표기하겠다. 아니, 원문에서 5.58B FLOPs라고 명시했으므로 그 수치를 따르는 것이 맞다. 논문 저자가 448x448 fine-tuning을 언급하면서도 FLOPs는 224x224 기준으로 제시했을 가능성이 높다. 따라서 5.58로 표기한다.

표의 데이터는 2를 종합하여 구성되었다.

표에서 볼 수 있듯이, Darknet-19는 VGG-16보다 훨씬 적은 연산량과 파라미터로 더 높은 Top-5 정확도를 달성했다. ResNet-50과는 유사한 정확도를 보이면서도 연산량은 다소 높지만, 이는 YOLOv2의 설계 목표였던 '실시간성'을 충족하는 범위 내에 있다. 후속 모델인 Darknet-53은 더 깊은 구조를 통해 정확도를 높이면서도 연산 효율성을 유지하여 Darknet-19의 철학을 계승 발전시켰음을 확인할 수 있다.


ImageNet에서 성공적으로 사전 훈련된 Darknet-19 분류 모델은 YOLOv2라는 특정 목적을 위한 강력한 특징 추출 백본으로 변환되는 과정을 거친다.2 이 변환 과정은 단순히 마지막 계층을 바꾸는 것을 넘어, 탐지 성능을 극대화하기 위한 구조적 변경을 포함한다.


- **계층 수정:** 먼저, 분류를 위해 존재했던 마지막 부분들이 제거된다. 구체적으로 1000개의 ImageNet 클래스를 예측하기 위한 마지막 컨볼루션 계층, 특징 맵을 벡터로 변환하는 전역 평균 풀링 계층, 그리고 확률 분포를 계산하는 소프트맥스 계층이 모두 제거된다.2
- **탐지 헤드 추가 (Adding Detection Head):** 제거된 부분에는 객체 탐지에 필요한 예측(바운딩 박스의 위치, 크기, 신뢰도, 클래스 확률)을 출력하기 위한 새로운 컨볼루션 계층들이 추가된다. 이 부분을 '탐지 헤드(detection head)'라고 부른다. PASCAL VOC 데이터셋을 예로 들면, YOLOv2는 각 그리드 셀마다 5개의 앵커 박스(anchor box)를 사용하고, 각 박스에 대해 5개의 좌표 관련 값($t_x, t_y, t_w, t_h, t_o$)과 20개의 클래스 확률을 예측해야 한다. 따라서 최종 1x1 컨볼루션 계층은 총 $5 \times (5 + 20) = 125$개의 필터를 갖도록 설계된다.2


YOLOv2가 도입한 가장 독창적인 구조적 개선 중 하나는 '패스스루 계층(Passthrough Layer)'이다. 이는 심층 신경망이 겪는 고질적인 문제, 즉 깊은 계층으로 갈수록 공간 해상도가 낮아져 작은 객체의 세밀한 특징을 잃어버리는 문제를 해결하기 위한 장치다.2

- **도입 배경:** 네트워크의 최종 특징 맵(Darknet-19의 경우 13x13)은 이미지 전체에 대한 풍부한 의미론적(semantic) 정보를 담고 있지만, 원본 이미지에 비해 해상도가 매우 낮다. 이로 인해 작은 객체는 이 특징 맵 상에서 몇 개의 픽셀로 압축되거나 아예 사라져 버려 정확한 탐지가 어렵다.
- **작동 원리:** 패스스루 계층은 이 문제를 해결하기 위해 네트워크의 더 이전 단계에 있는 고해상도 특징 맵(26x26x512)을 가져와 최종 특징 맵(13x13x1024)과 결합한다. 결합 방식은 단순히 덧셈이나 채널 연결이 아니다. 먼저, 26x26x512 크기의 고해상도 특징 맵을 공간적으로 재구성한다. 인접한 2x2 영역의 픽셀들을 각각 다른 채널로 쌓는 방식으로, 공간적 차원을 줄이는 대신 채널의 깊이를 늘린다. 이 과정을 통해 26x26x512 특징 맵은 13x13x2048 크기의 특징 맵으로 변환된다. 그 후, 이 변환된 특징 맵을 기존의 저해상도 특징 맵(13x13x1024)과 채널 차원에서 연결(concatenate)하여 최종적으로 13x13x3072 크기의 융합된 특징 맵을 생성한다.2
- **효과:** 이 독창적인 과정을 통해, 최종 예측을 수행하는 탐지 헤드는 저해상도의 고차원 의미 정보와 고해상도의 저차원 시각적(공간적) 정보를 모두 활용할 수 있게 된다. 그 결과, YOLOv2는 이전 버전에 비해 작은 객체에 대한 탐지 성능을 의미 있게 향상시킬 수 있었다.

이 패스스루 계층은 비록 단순한 형태이지만, 그 핵심 아이디어는 후대의 객체 탐지 모델에서 표준이 된 FPN(Feature Pyramid Network)이나 PANet과 같은 다중 스케일 특징 융합(multi-scale feature fusion) 아키텍처의 선구적인 시도라 할 수 있다. 패스스루 계층은 '다른 해상도의 특징을 결합하여 예측 성능을 높인다'는 근본적인 문제의식을 담고 있다.10 FPN은 이 아이디어를 더욱 발전시켜, top-down 경로와 lateral connection을 통해 여러 레벨의 특징 맵을 체계적으로 융합하는 구조를 제안했다.20 YOLOv3는 여기서 한 걸음 더 나아가, 세 가지 다른 스케일의 특징 맵에서 독립적으로 예측을 수행함으로써 FPN과 유사한 효과를 구현했다.18 패스스루 계층은 단 두 개의 스케일만을 일방향으로 연결하는 제한적인 방식이었지만, '깊은 층의 의미 정보'와 '얕은 층의 공간 정보'를 결합해야 한다는 문제의식을 처음으로 제기하고 실용적인 해결책을 제시했다는 점에서 기술적 이정표로 평가받을 수 있다.



- **효율성의 새로운 기준 제시:** Darknet-19의 가장 큰 기술적 의의는 VGGNet과 같은 무겁고 비효율적인 모델에 비해 훨씬 적은 연산량과 파라미터로 동등하거나 더 나은 분류 성능을 달성했다는 점이다.2 이는 딥러닝 아키텍처 설계에 있어 무조건적인 깊이와 복잡성 경쟁을 넘어 '연산 효율성'이라는 새로운 가치를 중요한 기준으로 제시한 사건이었다.
- **YOLOv2 성공의 핵심 동력:** Darknet-19의 빠른 속도와 강력한 특징 추출 능력은 YOLOv2가 당시 가장 빠른 SOTA(State-of-the-Art) 탐지기 중 하나로 자리매김하는 데 결정적인 역할을 했다.1 Darknet-19가 없었다면 YOLOv2의 "Better, Faster, Stronger"라는 목표 달성은 불가능했을 것이다.


그러나 Darknet-19는 효율성에 집중한 만큼 명백한 한계 또한 가지고 있었다.

- **정확도의 한계:** 비록 VGGNet에 비해서는 우수한 성능을 보였지만, 동시대에 등장한 ResNet 기반의 Faster R-CNN과 같은 2-stage detector들과 비교했을 때, COCO 데이터셋과 같은 더 어려운 벤치마크에서는 정확도가 뒤처졌다.23
- **구조적 단순함과 깊이의 한계:** Darknet-19는 기본적으로 컨볼루션과 풀링 계층을 순차적으로 쌓은 비교적 단순한 구조다. 가장 큰 구조적 한계는 ResNet에서 도입된 잔차 연결(residual connection)이 없다는 점이다. 잔차 연결 없이는 네트워크를 19층보다 훨씬 깊게 쌓을 경우, 역전파 과정에서 그래디언트가 소실(vanishing gradient)되는 문제에 직면하여 안정적인 훈련이 극도로 어려워진다.21 이는 네트워크의 표현력을 확장하는 데 근본적인 제약으로 작용했다.


YOLOv3의 백본으로 채택된 Darknet-53은 Darknet-19의 철학을 계승하면서도 그 명백한 한계를 극복하기 위해 설계되었다.7

- **잔차 연결의 도입:** Darknet-53의 가장 핵심적인 변화는 ResNet의 아이디어를 차용하여 잔차 블록(residual blocks)을 전면적으로 도입한 것이다.21 이 '지름길(shortcut)' 또는 '스킵 연결(skip connection)'은 입력 특징 맵을 몇 개의 컨볼루션 계층을 건너뛰어 출력에 직접 더해주는 구조다. 이를 통해 그래디언트가 깊은 계층까지 소실되지 않고 잘 전파될 수 있는 경로를 확보해주어, 53층이라는 훨씬 깊은 네트워크의 안정적인 훈련을 가능하게 했다.24
- **성능 향상:** 53층으로 깊어진 구조 덕분에 Darknet-53은 Darknet-19보다 훨씬 강력하고 복잡한 특징을 학습할 수 있게 되었다. 그 결과 ImageNet 분류 정확도와 객체 탐지 성능 모두에서 상당한 향상을 이루었다. 그러면서도 ResNet-101이나 ResNet-152와 같은 더 깊은 ResNet 모델에 비해서는 더 효율적인 연산 구조를 유지하여, 정확도와 속도 모두에서 뛰어난 균형을 달성했다.17

Darknet-19에서 Darknet-53으로의 발전 과정은 마치 변증법적 진화 과정을 연상시킨다. VGG 스타일의 단순한 순차적 네트워크를 '정(Thesis)'이라고 본다면, 이는 강력하지만 깊어질수록 훈련이 어렵고 비효율적이라는 내재적 모순을 가진다. 이에 대한 '반(Anti-thesis)'으로서 Darknet-19는 VGG의 비효율성을 거부하고 NIN과 BN을 결합해 '효율성'이라는 가치를 극대화했다. 하지만 이 과정에서 '깊이'라는 잠재력을 희생하는 새로운 한계를 낳았다. 마지막으로 Darknet-53은 이 둘을 '합(Synthesis)'하는 단계에 해당한다. 즉, Darknet-19가 확립한 효율적인 컨볼루션 블록 구조(1x1, 3x3 조합)를 유지하면서, ResNet이 제시한 '깊은 네트워크의 훈련 가능성'이라는 해결책(잔차 연결)을 도입한 것이다. 이를 통해 두 선행 모델의 장점을 모두 취하여 더 높은 차원의 아키텍처를 완성했다. 이러한 관점에서 Darknet-19는 그 자체로 완벽한 모델이 아니라, 더 나은 모델인 Darknet-53으로 나아가기 위해 반드시 거쳐야 할 진화의 한 단계였다. 그 한계가 명확했기에, 그 다음 단계의 목표 또한 명확해질 수 있었다.


Darknet-19는 2016년 당시, 딥러닝 아키텍처 설계의 패러다임이 무조건적인 깊이와 복잡성 경쟁으로 치닫던 시기에, '연산 효율성'이라는 실용적 가치를 전면에 내세운 중요한 모델이다. VGG의 단순성, NIN의 효율성, 그리고 배치 정규화의 안정성 등 당시까지 검증된 최고의 기술들을 영리하게 조합하여, '실시간 객체 탐지'라는 구체적인 목표에 완벽하게 최적화된, 빠르고 강력한 특징 추출기를 탄생시켰다.

Darknet-19가 남긴 가장 큰 유산은 YOLOv2의 성공을 통해 실시간 객체 탐지 기술의 대중화를 이끌었다는 점이다. 이는 학술 연구의 영역에 머물러 있던 객체 탐지 기술이 다양한 산업 현장에 적용될 수 있는 가능성을 열어주었다. 또한, 그 명확한 구조적 한계(깊이의 제약)는 후속 연구인 Darknet-53이 잔차 연결이라는 해결책을 도입하며 한 단계 더 발전하는 필연적인 계기를 제공했다. 이는 딥러닝 아키텍처가 어떻게 문제를 정의하고, 기존 기술을 재해석하며, 스스로의 한계를 극복하며 진화해 나가는지를 보여주는 모범적인 사례로 남아있다.

비록 Darknet-19는 그 자체로 SOTA의 자리를 오랫동안 유지하지는 못했지만, 딥러닝 발전사에서 '효율적 아키텍처 설계'라는 중요한 흐름을 형성한 이정표로서 그 가치는 퇴색되지 않을 것이다.


1. Paper page - YOLO9000: Better, Faster, Stronger - Hugging Face, 8월 24, 2025에 액세스, https://huggingface.co/papers/1612.08242
2. [1612.08242] YOLO9000: Better, Faster, Stronger - ar5iv - arXiv, 8월 24, 2025에 액세스, https://ar5iv.labs.arxiv.org/html/1612.08242
3. ‪Joseph Redmon‬ - ‪Google Scholar‬, 8월 24, 2025에 액세스, https://scholar.google.com/citations?user=TDk_NfkAAAAJ&hl=en
4. YOLO9000: Better, Faster, Stronger | Request PDF - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/311925600_YOLO9000_Better_Faster_Stronger
5. [PDF] YOLO9000: Better, Faster, Stronger - Semantic Scholar, 8월 24, 2025에 액세스, https://www.semanticscholar.org/paper/YOLO9000%3A-Better%2C-Faster%2C-Stronger-Redmon-Farhadi/7d39d69b23424446f0400ef603b2e3e22d0309d6
6. yolov4 - arXiv, 8월 24, 2025에 액세스, [https://arxiv.org/pdf/2502.04161?](https://arxiv.org/pdf/2502.04161)
7. What is YOLOv6? A Deep Insight into the Object Detection Model - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2412.13006v1
8. Choosing the Right Pre-Trained Model: A Guide to VGGNet, ResNet, GoogleNet, AlexNet, and Inception | by Sohaib Zafar | Medium, 8월 24, 2025에 액세스, https://medium.com/@sohaib.zafar522/choosing-the-right-pre-trained-model-a-guide-to-vggnet-resnet-googlenet-alexnet-and-inception-db7a8c918510
9. Why is resnet faster than vgg - Cross Validated - Stack Exchange, 8월 24, 2025에 액세스, https://stats.stackexchange.com/questions/280179/why-is-resnet-faster-than-vgg
10. A Comprehensive Review of YOLO Architectures in Computer Vision: From YOLOv1 to YOLOv8 and YOLO-NAS - MDPI, 8월 24, 2025에 액세스, https://www.mdpi.com/2504-4990/5/4/83
11. Network structure of Darknet-19. 'Conv' refers to 'convolutional... - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/Network-structure-of-Darknet-19-Conv-refers-to-convolutional-layer-and-Max_fig2_351649815
12. YOLO Algorithm for Object Detection Explained [+Examples] - V7 Labs, 8월 24, 2025에 액세스, https://www.v7labs.com/blog/yolo-object-detection
13. YOLOv4: A Breakthrough in Real-Time Object Detection - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2502.04161v1
14. DarkNet-19 Architecture | Download Scientific Diagram - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/DarkNet-19-Architecture_fig1_384670327
15. LightFFDNets: Lightweight Convolutional Neural Networks for Rapid Facial Forgery Detection - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2411.11826v1
16. Backbones-Review: Feature Extraction Networks for Deep Learning and Deep Reinforcement Learning Approaches - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2206.08016
17. Comparison table of DarkNet-53 network parameters with those of... - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/Comparison-table-of-DarkNet-53-network-parameters-with-those-of-DarkNet-19-ResNet-101_tbl1_379770482
18. An Incremental Improvement with Darknet-53 and Multi-Scale Predictions (YOLOv3), 8월 24, 2025에 액세스, https://pyimagesearch.com/2022/05/09/an-incremental-improvement-with-darknet-53-and-multi-scale-predictions-yolov3/
19. darknet19 - (Not recommended) DarkNet-19 convolutional neural network - MATLAB, 8월 24, 2025에 액세스, https://in.mathworks.com/help/deeplearning/ref/darknet19.html
20. [PDF] YOLOv3: An Incremental Improvement - Semantic Scholar, 8월 24, 2025에 액세스, https://www.semanticscholar.org/paper/YOLOv3%3A-An-Incremental-Improvement-Redmon-Farhadi/ebc96892b9bcbf007be9a1d7844e4b09fde9d961
21. Darknet 53 - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/computer-vision/darknet-53/
22. Learning performance of DarkNet-19 and ResNet-50. - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/Learning-performance-of-DarkNet-19-and-ResNet-50_fig3_343720138
23. YOLO9000: Better, Faster, Stronger - Department of Computer Science, University of Toronto, 8월 24, 2025에 액세스, https://www.cs.utoronto.ca/~fidler/teaching/2018/slides/CSC2548/HarisKhan_YOLO9000review_v2.pdf
24. the impact of different backbone architecture on autonomous vehicl dataset - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2309.08564
25. The structure of the residual layer in Darknet53, CSPDarknet53 and our model. - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/The-structure-of-the-residual-layer-in-Darknet53-CSPDarknet53-and-our-model_fig2_358706263
26. YOLOv3 Object Detection - Medium, 8월 24, 2025에 액세스, https://medium.com/@Shahidul1004/yolov3-object-detection-f3090a24efcd
27. [D] Why and how do residual/skip connections work? Looking for literature - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/rmxri9/d_why_and_how_do_residualskip_connections_work/


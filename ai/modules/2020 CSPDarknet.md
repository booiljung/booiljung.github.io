[딥러닝 모듈 (Modules)](./index.md)
# CSPDarknet (2020)


객체 탐지(Object Detection)는 컴퓨터 비전 분야의 핵심 과업 중 하나로, 그 성능은 특징 추출을 담당하는 백본(Backbone) 네트워크의 아키텍처에 의해 크게 좌우된다. 초기 VGG, ResNet과 같이 깊이를 통해 성능을 확보하려는 시도부터, 연산 효율성을 극대화한 MobileNet, EfficientNet에 이르기까지 백본 네트워크는 끊임없이 진화해왔다. 이러한 기술적 흐름 속에서 YOLO(You Only Look Once) 시리즈는 단일 단계(single-stage) 방식을 통해 실시간 객체 탐지의 새로운 패러다임을 제시했으며, 특히 YOLOv4의 등장은 CSPDarknet이라는 혁신적인 백본 아키텍처의 효율성과 강력함을 세상에 알리는 결정적 계기가 되었다.1

과거 딥러닝 모델의 발전은 더 깊고 넓은 네트워크가 곧 더 나은 성능이라는 통념에 기반을 두었다.3 그러나 이러한 접근 방식은 필연적으로 막대한 연산 비용과 메모리 요구사항을 동반했다. DenseNet과 같은 아키텍처는 특징 맵의 재사용을 통해 정보 흐름을 개선하고자 했으나, 이는 역전파 과정에서 중복된 경사도 정보(duplicate gradient information)를 야기하여 학습 효율을 저해하는 새로운 문제를 낳았다.5 CSPDarknet의 등장은 이러한 기존의 통념에 대한 중대한 반박이자 새로운 해법이었다. 이는 네트워크의 거시적 규모 확장보다, 내부 정보 흐름과 경사도(gradient)의 '질적 관리'가 성능 향상에 더 효율적일 수 있음을 입증한 사례다.

본 보고서는 CSPDarknet 아키텍처의 근간을 이루는 두 핵심 기술, 즉 CSPNet(Cross Stage Partial Network)의 설계 철학과 Darknet-53의 구조적 특성을 시작으로, 이 둘의 유기적인 결합을 통해 탄생한 CSPDarknet53의 아키텍처를 심층적으로 분석한다. 나아가 YOLOv4의 백본으로서 달성한 성능, 타 아키텍처와의 비교 분석, 그리고 후속 기술 및 실시간 객체 탐지 분야 전반에 미친 기술적 영향을 다각도로 고찰하고자 한다. 최종적으로 본 보고서는 연산 효율성과 정확도라는 상충하는 두 목표를 '구조적 효율성'이라는 열쇠로 풀어낸 CSPDarknet의 공학적 성취와 그 유산을 평가하는 것을 목표로 한다.


CSPDarknet의 혁신성을 이해하기 위해서는 그 뿌리가 되는 두 가지 핵심 기술, CSPNet과 Darknet-53에 대한 선행적 이해가 필수적이다. CSPNet은 네트워크 내 정보 흐름의 효율성을 극대화하는 설계 원칙을 제공했고, Darknet-53은 실시간 객체 탐지에 최적화된 강력한 특징 추출 능력을 제공했다. 이 두 기술의 결합은 단순한 조합을 넘어선 시너지를 창출했다.


CSPNet은 Chien-Yao Wang 등에 의해 2019년에 제안된 아키텍처 설계 전략으로, 기존 심층 신경망(DNN)의 학습 과정에서 발생하는 비효율성을 해결하는 데 초점을 맞춘다.6


CSPNet의 출발점은 DenseNet과 같은 고밀도 연결망(densely connected network)에서 발생하는 '중복 경사도 정보' 문제에 대한 인식이었다.5 DenseNet에서는 각 레이어의 출력이 후속 모든 레이어의 입력으로 연결되는데, 이로 인해 역전파(backpropagation) 과정에서 서로 다른 가중치를 업데이트하는 데 매우 유사한 경사도 정보가 반복적으로 재사용되는 현상이 발생한다. 연구진은 이러한 중복성이 네트워크의 학습 능력을 저해하고 불필요한 연산 비용을 초래하는 핵심 원인이라고 진단했다.7


이 문제를 해결하기 위해 CSPNet은 각 네트워크 스테이지(stage)의 정보 흐름을 재구성하는 독창적인 방법을 제안했다. 그 핵심 전략은 입력 특징 맵(feature map)을 채널(channel) 차원에서 두 부분으로 분할하고, 각각 다른 경로로 처리한 뒤 다시 병합하는 것이다.1

1. **분할 (Split)**: 한 스테이지의 시작 부분에서, 입력 특징 맵은 채널 축을 따라 두 개의 부분 경로(partial path)로 나뉜다.
2. **처리 (Process)**: 한 부분(`Part 1`)은 기존의 연산 블록(예: Dense Block, Residual Block)을 그대로 통과하며 복잡한 특징 변환을 수행한다.10
3. **직결 (Bypass)**: 다른 한 부분(`Part 2`)은 별도의 복잡한 연산을 거치지 않고, 그대로 다음 레이어로 직접 연결된다. 이 경로는 정보의 손실 없이 원본 특징의 일부를 보존하는 역할을 한다.10
4. **병합 (Merge)**: 두 경로를 통과한 특징 맵들은 스테이지의 마지막 부분에서 다시 결합(concatenate)되어 다음 스테이지의 입력으로 전달된다.1


이러한 교차 스테이지 부분 연결(Cross Stage Partial connection) 구조는 다음과 같은 명확한 이점을 제공한다.

- **연산량 감소**: 전체 특징 맵의 절반가량만 복잡한 연산 블록을 통과하므로, 네트워크의 총 연산량이 크게 줄어든다. 실험적으로 CSPNet을 적용했을 때 기존 네트워크 대비 10%에서 20%의 연산량 감소 효과가 확인되었다.5
- **학습 능력 강화**: 경사도 흐름이 두 개의 독립적인 경로로 나뉘어 전파되면서, 학습 과정에서 경사도의 상호 상관관계(correlation)가 낮아진다. 이는 네트워크가 중복된 정보 대신 더 다양하고 풍부한 특징 조합을 학습하도록 유도하여 전반적인 학습 능력을 강화하는 효과를 가져온다.6
- **메모리 비용 절감**: 처리해야 할 중간 특징 맵의 크기가 줄어들고, 전체적인 연산 병목 현상이 완화되어 메모리 대역폭 요구사항과 총 메모리 사용량을 효과적으로 줄일 수 있다.5

CSPNet은 특정 아키텍처(DenseNet)의 문제를 해결하기 위해 고안되었지만, 그 원리는 '정보 흐름의 효율화'라는 보편성을 띤다. 따라서 ResNet, ResNeXt 등 다양한 아키텍처에 쉽게 이식될 수 있는 일종의 '플러그인(plug-in)' 모듈로서 높은 범용성과 잠재력을 지녔다.5


Darknet-53은 Joseph Redmon이 YOLOv3의 핵심 특징 추출기(feature extractor)로 설계한 심층 컨볼루션 신경망이다.3 이 아키텍처는 당대 최고의 정확도를 자랑하던 ResNet의 핵심 아이디어를 계승하면서도, 객체 탐지 작업에 더 적합하도록 구조를 단순화하고 효율화한 실용적인 설계를 특징으로 한다.


- **53개의 컨볼루션 레이어**: 이름에서 알 수 있듯이, 총 53개의 컨볼루션 레이어로 구성된 깊은 네트워크 구조를 가진다. 이는 복잡하고 추상적인 특징을 학습하기에 충분한 깊이를 제공한다.3

- **잔차 연결 (Residual Connections)**: Darknet-53은 ResNet의 핵심 아이디어인 숏컷 연결(shortcut connection)을 적극적으로 도입했다. 이를 통해 네트워크의 깊이가 깊어지더라도 경사도 소실(vanishing gradient) 문제없이 안정적으로 학습을 진행할 수 있다.3 Darknet-53의 잔차 블록은 

  `1x1` 컨볼루션으로 채널을 줄였다가 `3x3` 컨볼루션으로 특징을 추출하고 다시 `1x1` 컨볼루션으로 채널을 복원하는 병목(bottleneck) 구조 대신, `1x1` 컨볼루션과 `3x3` 컨볼루션을 순차적으로 적용하는 더 간결한 형태를 사용한다.13

- **계층적 특징 추출**: 네트워크는 스트라이드(stride)가 2인 컨볼루션을 통해 주기적으로 다운샘플링을 수행한다. 이 과정에서 특징 맵의 공간적 해상도는 절반으로 줄어드는 대신 채널의 수는 두 배로 늘어난다. 이러한 계층적 구조는 이미지의 저수준(low-level) 특징부터 고수준(high-level)의 의미론적 특징까지 다양한 스케일의 정보를 효과적으로 추출하며, 이는 다중 스케일 객체 탐지에 필수적인 요소다.3

- **활성화 함수**: 활성화 함수로는 Leaky ReLU를 사용하여, 입력값이 음수일 때 기울기가 0이 되어 뉴런이 비활성화되는 'Dying ReLU' 문제를 방지하고 안정적인 학습을 돕는다.3

Darknet-53은 이미 YOLOv3를 통해 실시간 객체 탐지 분야에서 그 효율성과 성능을 입증받은 검증된 아키텍처였다. 따라서, 검증된 고성능 백본(Darknet-53)에 검증된 효율화 기법(CSPNet)을 결합하는 것은 최소한의 위험으로 최대한의 성능 향상을 꾀하는 합리적인 공학적 결정이었으며, 이는 '혁신'과 '안정성'의 균형을 맞춘 전략적 선택으로 볼 수 있다.


CSPDarknet53은 Darknet-53의 강력한 특징 추출 능력과 CSPNet의 혁신적인 효율화 전략을 결합한 아키텍처다. 이 장에서는 두 기술이 어떻게 구조적으로 통합되었는지, 그리고 성능 최적화를 위해 어떤 추가적인 요소들이 도입되었는지 상세히 분석한다.


CSPDarknet53의 가장 핵심적인 변화는 Darknet-53을 구성하던 잔차 블록(Residual Block) 그룹들을 CSPNet의 원리를 적용한 새로운 CSP 블록(논문에서는 BottleneckCSP로 지칭)으로 대체한 것이다.13


BottleneckCSP 모듈은 Darknet-53의 잔차 학습 철학을 유지하면서 CSPNet의 분할-처리-병합 구조를 적용하여 설계되었다. 그 데이터 흐름은 다음과 같다.13

1. **분할 (Split)**: 스테이지의 입력 특징 맵은 채널 축을 기준으로 두 개의 경로로 분할된다.
2. **경로 1 (처리 경로, Processing Path)**: 분할된 특징 맵의 일부는 먼저 `1x1` 컨볼루션을 통과하여 채널 수가 조정된 후, Darknet-53의 기존 잔차 블록들을 `N`번 순차적으로 통과한다. 이 과정을 통해 복잡한 특징 추출 및 변환이 이루어진다. 마지막으로 다시 `1x1` 컨볼루션을 거쳐 처리된 특징 맵이 생성된다.
3. **경로 2 (직결 경로, Bypass Path)**: 나머지 특징 맵은 별도의 잔차 블록을 거치지 않고, 간단한 `1x1` 컨볼루션만을 통과하여 채널 수를 조정한 후 다음 단계로 직접 전달된다. 이 경로는 원본 정보의 일부를 보존하고 경사도 흐름을 위한 별도의 통로를 제공한다.
4. **병합 (Merge)**: 두 경로의 출력 특징 맵은 채널 축을 따라 결합(concatenate)된다. 이후, 마지막으로 `1x1` 컨볼루션(전이 계층, Transition Layer의 역할)을 통해 전체 특징 맵이 통합되어 다음 스테이지로 전달된다.

이러한 구조적 변환은 연산량의 상당 부분을 차지하는 잔차 블록을 특징 맵의 일부에만 적용함으로써, 전체적인 계산 복잡도를 낮추면서도 다양한 경사도 흐름을 확보하여 학습 효율을 높이는 효과를 가져온다. 아래 표는 Darknet-53의 전통적인 잔차 스테이지와 CSPDarknet53의 BottleneckCSP 스테이지 간의 구조적 차이를 명확히 보여준다.


| 구분 (Component)               | Darknet-53 Residual Stage                                    | CSPDarknet53 BottleneckCSP Stage                             |
| ------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **입력 (Input)**               | 전체 특징 맵 $x$                                             | 전체 특징 맵 $x$                                             |
| **처리 과정 (Processing)**     | 1. $x$가 $N$개의 잔차 블록(Residual Block)을 순차적으로 통과함. 2. 각 블록은 $1 \times 1$ Conv` → $3 \times 3$ Conv` → `Shortcut Add` 연산으로 구성됨. | 1. $x$를 두 부분 $x_1$, $x_2$로 분할 (Split). 2. $x_1$은 $N$개의 잔차 블록을 통과 (Processing Path). 3. $x_2$는 최소한의 처리(예: $1 \times 1$ Conv`) 후 대기 (Bypass Path). 4. 처리된 $x_1$과 $x_2$를 결합 (Concatenate). |
| **출력 (Output)**              | 최종 잔차 블록의 출력                                        | 결합된 특징 맵을 $1 \times 1$ Conv`로 통합한 최종 출력       |
| **핵심 연산 (Core Operation)** | 덧셈 기반 잔차 학습 (Additive Residual Learning)             | 분할-병합 기반 교차 스테이지 학습 (Split-Merge Cross-Stage Learning) |


YOLOv4는 백본인 CSPDarknet53의 성능을 극대화하기 위해 기존의 Leaky ReLU 활성화 함수 대신 Mish라는 새로운 활성화 함수를 채택했다.2 이는 더 나은 정확도와 안정적인 학습을 달성하기 위한 전략적 선택이었다.


Mish 함수는 자기 정규화(self-regularized) 비단조(non-monotonic) 활성화 함수로, 수학적으로 다음과 같이 정의된다.16
$$
f(x) = x \cdot \tanh(\text{softplus}(x))
$$
여기서 $\text{softplus}(x) = \ln(1 + e^x)$ 이다.


Mish 함수는 기존 활성화 함수들과 구별되는 몇 가지 중요한 특성을 가진다.

- **비단조성 (Non-monotonicity)**: 그래프가 단조롭게 증가하지 않고, 음수 영역의 특정 구간에서 미세하게 감소했다가 다시 증가하는 형태를 띤다. 이 특성은 작은 음수 값 입력을 완전히 0으로 만들지 않고 보존하여, 모델의 표현력을 높이고 경사도 흐름을 개선하는 데 긍정적인 영향을 미친다.16
- **평활성 (Smoothness)**: Mish 함수는 모든 지점에서 무한히 미분 가능한 $C^\infty$ 연속 함수다. 이는 ReLU가 $x=0$ 지점에서 미분 불가능한 특이점(singularity)을 갖는 것과 대조적으로, 경사도 기반 최적화 과정에서 더 안정적이고 부드러운 학습을 유도한다.16
- **자기 정규화 (Self-regularization)**: 그래프의 아래쪽이 특정 값(약 -0.31)으로 유계(bounded below)인 특성은 일종의 정규화 효과를 가져와 모델의 일반화 성능을 높이는 데 기여한다.16
- **성능 향상**: 다양한 데이터셋과 네트워크 아키텍처를 사용한 실험에서 Mish는 ReLU나 Swish와 같은 다른 표준 활성화 함수들보다 일관되게 더 높은 정확도를 보였다. 특히, CSP-DarkNet-53 백본을 사용한 MS-COCO 객체 탐지 실험에서 활성화 함수를 ReLU에서 Mish로 교체하는 것만으로도 mAP(mean Average Precision)가 0.4% 향상되는 결과가 보고되었다.16


CSPDarknet53의 전체 구조는 하나의 초기 컨볼루션 레이어와 5개의 CSP 스테이지로 구성된다. 각 CSP 스테이지는 다운샘플링을 위한 `3x3` 스트라이드 2 컨볼루션과, 각각 `1, 2, 8, 8, 4`개의 잔차 유닛을 포함하는 BottleneckCSP 모듈로 이루어져 있다. 이를 통해 네트워크는 전체적으로 29개의 `3x3` 컨볼루션 레이어, 725x725 픽셀에 달하는 넓은 유효 수용 영역(effective receptive field), 그리고 약 2,760만 개의 파라미터를 갖게 된다. 이는 다양한 크기의 객체를 탐지하는 데 필요한 충분한 모델 용량과 복잡성을 제공한다.10


CSPDarknet53은 YOLOv4의 백본으로 채택되면서 그 진가를 세상에 알렸다. YOLOv4의 성공은 CSPDarknet53의 우수한 성능과 효율성에 크게 힘입었으며, 이는 실시간 객체 탐지 분야의 새로운 기준을 제시했다.


YOLOv4 연구팀은 최적의 백본 아키텍처를 선정하기 위해 다양한 후보군에 대한 광범위한 실험과 이론적 분석을 수행했다. 그 과정에서 CSPDarknet53이 최종적으로 선택된 이유는 다음과 같다.

- **탐지 성능의 우수성**: 연구팀은 CSPResNeXt50, CSPDarknet53 등 여러 CSP 기반 아키텍처를 비교했다. 흥미롭게도, 이미지 분류(ImageNet) 태스크에서는 CSPResNeXt50이 더 우수한 성능을 보였지만, 목표 과업인 객체 탐지(MS COCO) 태스크에서는 CSPDarknet53이 더 높은 정확도를 기록했다.10 이는 특정 과업에 최적화된 아키텍처가 존재함을 시사한다.
- **이론적 근거**: 객체 탐지는 단일 이미지 내에 존재하는 다양한 크기와 위치의 객체들을 모두 정확하게 식별하고 위치를 특정해야 하므로, 단순 분류 과업보다 더 큰 입력 해상도, 더 넓은 수용 영역(receptive field), 그리고 더 많은 파라미터(모델 용량)를 요구한다. CSPDarknet53은 CSPResNeXt50에 비해 이러한 객체 탐지의 고유한 요구사항을 더 잘 충족시키는 구조적 특성을 가지고 있었고, 이것이 실험 결과의 차이로 이어진 것으로 분석되었다.10


CSPDarknet53을 백본으로 탑재한 YOLOv4는 당시 객체 탐지 분야에서 가장 중요한 두 지표인 정확도와 속도 모든 면에서 최고 수준의 성능을 달성했다.

- **MS COCO 벤치마크**: 객체 탐지 분야의 표준 벤치마크인 MS COCO `test-dev` 데이터셋에서, YOLOv4는 **43.5% AP (mAP@.5:.95)** 및 **65.7% AP50 (mAP@.5)** 이라는 당시 최고 수준의 정확도를 기록했다.15
- **실시간 성능**: Tesla V100 GPU 환경에서 약 **65 FPS(Frames Per Second)** 의 빠른 처리 속도를 보여주어, 높은 정확도를 유지하면서도 실시간 응용이 가능함을 입증했다.15 이는 이전 버전인 YOLOv3와 비교했을 때, AP(정확도)는 10%, FPS(속도)는 12% 향상된 괄목할 만한 성과였다.21
- **접근성**: YOLOv4의 또 다른 중요한 성과는 기술의 대중화에 기여했다는 점이다. 고가의 전문 연구 장비가 아닌, 8GB에서 16GB VRAM을 갖춘 일반적인 게이밍 GPU에서도 모델을 훈련하고 사용할 수 있도록 설계되었다. 이는 고성능 객체 탐지 기술에 대한 접근 장벽을 크게 낮추는 계기가 되었다.22


CSPDarknet53의 우수성은 단순히 최종 성능 지표뿐만 아니라, 연산 효율성 측면에서도 드러난다. BFLOPs(Billion Floating-point Operations)는 모델의 이론적 연산 복잡도를 나타내는 지표로, 낮을수록 효율적임을 의미한다.

- **CSPNet의 연산량 감소 효과**: Scaled-YOLOv4 논문에 따르면, CSPNet 구조를 Darknet 스테이지에 적용할 경우, 스테이지 내 잔차 블록의 수가 1보다 큰($k > 1$) 모든 조건에서 항상 기존 Darknet 스테이지보다 연산량이 적으며, 이론적으로 최대 50%까지 연산량을 감소시킬 수 있다.24
- **YOLOv4-CSP 모델의 BFLOPs**: CSPDarknet53을 백본으로 하고, 넥(neck) 부분에도 CSP 구조를 적용한 YOLOv4-CSP 모델(CD53s-CPANSPP-Mish)은 약 **109 BFLOPs**의 연산량을 가진다. 이는 CSP를 적용하지 않은 유사 구조의 모델(D53-PANSPP-Mish, 160 BFLOPs)과 비교했을 때 연산량이 약 32% 감소한 수치이며, 파라미터 수도 7,800만 개에서 5,300만 개로 크게 줄어들었다.24

아래 표는 YOLOv4의 성능을 주요 경쟁 모델과 비교하여 요약한 것이다.


| 모델 (Model)    | 백본 (Backbone)  | AP (mAP@.5:.95) | AP50 (mAP@.5) | FPS (Tesla V100) | BFLOPs                     |
| --------------- | ---------------- | --------------- | ------------- | ---------------- | -------------------------- |
| **YOLOv4**      | **CSPDarknet53** | **43.5%**       | **65.7%**     | **~65**          | **~109** (YOLOv4-CSP 기준) |
| YOLOv3          | Darknet-53       | 33.0%           | 55.3%         | ~58              | ~142 (유사 구조 기준)      |
| EfficientDet-D3 | EfficientNet-B3  | 45.8%           | 64.9%         | ~31              | 139                        |

YOLOv4의 성공은 단지 CSPDarknet53이라는 하나의 뛰어난 백본 때문만은 아니었다. YOLOv4 논문에서 강조한 'Bag of Freebies'(학습 비용 증가 없이 정확도를 높이는 기법)와 'Bag of Specials'(추론 비용은 약간 증가하지만 정확도를 크게 높이는 기법)와 같은 수많은 최적화 기법들이 시너지를 일으킨 결과다.2 하지만 만약 백본의 특징 추출 능력이 부족하거나 연산 병목이 심했다면, 넥(neck)이나 헤드(head)에서 아무리 좋은 기법을 사용했더라도 그 효과는 제한적이었을 것이다. CSPDarknet53은 풍부한 특징을 효율적으로 추출하는 능력을 갖춤으로써, 후속 모듈인 SPP(Spatial Pyramid Pooling)와 PANet(Path Aggregation Network)이 다중 스케일 특징을 효과적으로 융합하고 정제할 수 있는 양질의 '재료'를 공급하는 강력하고 효율적인 '기반 플랫폼' 역할을 수행했다. 즉, CSPDarknet53의 효율성은 다른 최적화 기법들의 효과를 극대화하는 '승수 효과(multiplier effect)'를 낳았으며, 이것이 YOLOv4가 종합적으로 뛰어난 성능을 달성한 근본적인 원인 중 하나다.


CSPDarknet53의 아키텍처적 우수성은 다른 주요 백본 아키텍처와의 비교를 통해 더욱 명확해진다. 본 장에서는 대표적인 CNN 아키텍처인 ResNet 계열과 EfficientNet 계열을 대상으로 CSPDarknet53의 성능 및 효율성을 비교 분석한다.


ResNet은 잔차 연결을 통해 매우 깊은 네트워크의 학습을 가능하게 한 혁신적인 아키텍처로, 오랫동안 컴퓨터 비전 분야의 표준 백본으로 사용되어 왔다.4

- **성능**: Darknet-53은 설계 당시부터 ResNet-152와 비슷한 수준의 이미지 분류 정확도를 보이면서도 추론 속도는 거의 2배 빠른 효율성을 목표로 했다.3 CSPNet을 적용하여 효율성을 더욱 극대화한 CSPDarknet53은 특정 객체 탐지 과업에서 ResNet 계열보다 우수한 성능을 보였다. 예를 들어, 동일한 YOLOv4 프레임워크를 사용한 미세화석 탐지 연구에서, CSPDarknet53 백본은 83.41%의 mAP를 기록하여 CSP 구조를 적용한 ResNeXt-50(CSPResNeXt-50) 백본의 81.00% mAP를 상회했다.25
- **효율성**: ResNet은 성능을 높이기 위해 네트워크의 깊이를 늘리는 전략을 주로 사용하며, 이는 깊이가 깊어질수록 연산량이 급격히 증가하는 경향으로 이어진다. 반면, CSPDarknet53은 CSP 구조를 통해 각 스테이지의 연산 부담을 근본적으로 줄임으로써, 비슷한 수준의 성능을 더 적은 연산으로 달성하는 것을 목표로 한다. 이는 실시간 처리가 중요한 응용 분야에서 큰 장점으로 작용한다.27


EfficientNet은 네트워크의 깊이(depth), 너비(width), 입력 해상도(resolution) 세 가지 차원을 균형 있게 확장하는 '복합 스케일링(compound scaling)' 방법을 통해 모델의 효율성을 최적화하는 새로운 접근법을 제시했다.27

- **설계 철학의 차이**: 두 아키텍처는 효율성을 추구하는 근본적인 방식에서 차이를 보인다. EfficientNet이 네트워크의 거시적인 차원(깊이, 너비, 해상도)을 체계적으로 조절하는 '스케일링'에 집중한다면, CSPDarknet53은 네트워크 스테이지 내부의 정보 흐름을 미시적으로 재구성하여 연산의 중복을 제거하는 '구조적 최적화'에 집중한다.
- **성능 트레이드오프**: YOLOv4는 비슷한 정확도를 보인 동급의 EfficientDet(EfficientNet 기반 탐지기)과 비교했을 때, 2배가량 빠른 추론 속도를 기록했다.21 이는 BFLOPs와 같은 이론적 연산량 지표가 실제 하드웨어에서의 병렬 처리 효율성이나 메모리 접근 패턴과 같은 현실적인 요소를 완벽하게 반영하지 못하며, CSPDarknet53의 구조가 실제 GPU 환경에서의 추론에 더 최적화되어 있음을 시사한다.
- **모델 크기**: 경량화 측면에서는 EfficientNet 계열이 강점을 보일 수 있다. 앞서 언급된 미세화석 탐지 연구에서, EfficientNet-B0를 백본으로 사용한 YOLOv4 모델은 156.8 MB의 크기로, CSPDarknet53 기반 모델보다 훨씬 가벼웠다. 하지만, 이는 정확도(mAP 81.76%)와 속도(CSPDarknet53은 33.4 FPS) 측면에서의 일부 손실을 감수한 결과였다.25 이는 응용 분야의 요구사항(최고 성능 vs. 경량화)에 따라 최적의 백본 선택이 달라질 수 있음을 보여주는 명확한 사례다.

아래 표는 특정 데이터셋에 대해 동일한 YOLOv4 프레임워크 내에서 각기 다른 백본을 사용했을 때의 성능을 비교한 결과다.


| 백본 아키텍처 (Backbone Architecture) | mAP (%)   | FPS      | 모델 크기 (MB) | 강점 (Strength)                                 |      |
| ------------------------------------- | --------- | -------- | -------------- | ----------------------------------------------- | ---- |
| **CSPDarknet53**                      | **83.41** | **33.4** | -              | **최고 정확도 및 속도 (Best Accuracy & Speed)** |      |
| CSPResNeXt-50                         | 81.00     | -        | -              | 균형 잡힌 성능 (Balanced Performance)           |      |
| EfficientNet-B0                       | 81.76     | -        | **156.8**      | **최고의 경량성 (Most Lightweight)**            |      |

주: 위 데이터는 미세화석 탐지 데이터셋에 대한 실험 결과이며 25, 데이터셋의 특성에 따라 아키텍처 간 상대적 성능은 달라질 수 있다.

이 비교를 통해 CSPDarknet53은 절대적인 최고 성능과 실시간 처리 속도를 요구하는 작업에 가장 적합하며, EfficientNet은 모델 크기와 메모리 사용량이 최우선 고려사항인 경량화 환경에서, ResNet 계열은 검증된 안정성과 범용성을 바탕으로 한 다양한 응용 분야에서 각각의 강점을 가짐을 알 수 있다.


CSPDarknet의 등장은 단순히 하나의 고성능 백본 아키텍처가 추가된 것을 넘어, 실시간 객체 탐지 분야의 기술적 지형도와 향후 발전 방향에 깊은 영향을 미쳤다.


- **새로운 성능 표준 제시**: CSPDarknet53을 탑재한 YOLOv4는 '정확하면서도 빠른' 실시간 객체 탐지기의 새로운 기준을 정립했다. 이전까지 객체 탐지 모델들은 높은 정확도를 위해서는 속도를 희생하거나, 빠른 속도를 위해서는 정확도를 타협해야 하는 명확한 트레이드오프 관계에 놓여 있었다. YOLOv4는 이 간극을 혁신적으로 좁힘으로써, 두 마리 토끼를 모두 잡을 수 있다는 가능성을 현실로 만들었다.22
- **기술 접근성 향상**: 고가의 전문가용 GPU가 아닌 일반 소비자용 GPU에서도 고성능 모델을 훈련하고 배포할 수 있게 함으로써, 객체 탐지 기술의 도입 장벽을 획기적으로 낮추었다.22 이는 자금이 부족한 학계 연구 그룹이나 중소기업도 최첨단 기술을 활용할 수 있는 길을 열어주었으며, 자율주행, 보안 감시, 스마트 팩토리, 의료 영상 분석 등 수많은 실제 산업 응용 분야의 발전을 가속화하는 기폭제 역할을 했다.29
- **설계 패러다임의 변화**: CSPNet과 CSPDarknet의 성공은 네트워크 아키텍처 설계에 있어 '연산 효율성'과 '정보 흐름 최적화'의 중요성을 다시 한번 각인시켰다. 이는 후속 연구들이 무작정 모델의 깊이나 너비를 키우기보다, 더 효율적인 연산 블록 구조를 탐색하고 네트워크 내부의 병목 현상을 제거하는 방향으로 연구를 진행하도록 영감을 주었다.31


CSPDarknet의 가장 중요한 유산은 그 핵심 철학이 후속 YOLO 시리즈에 지속적으로 계승되고 발전했다는 점에서 찾을 수 있다. 'Cross Stage Partial'이라는 개념은 YOLO 생태계의 DNA로 자리 잡았다.

- **CSP 개념의 계승 및 발전**: CSPDarknet의 핵심 아이디어인 '정보 흐름을 분할하여 처리 효율을 높이는' 방식은 YOLOv5, YOLOv8 등 후속 모델에서 다양한 형태로 변주되며 계승되었다.9
- **구조적 변형**:
  - **YOLOv5**: CSPDarknet53을 기반으로 하되, PyTorch 프레임워크로 재구현되면서 C3(CSP Bottleneck with 3 convolutions) 모듈과 같은 형태로 구조가 일부 수정되고 최적화되었다. 이는 CSP 개념을 유지하면서도 더 나은 구현 유연성을 확보하기 위함이었다.33
  - **YOLOv8**: C2f(CSP Bottleneck with 2 convolutions, feature fusion) 블록을 도입하여 CSP 개념을 한 단계 더 발전시켰다. C2f 블록은 특징 맵을 분할한 뒤, 더 많은 교차 연결을 통해 풍부한 그래디언트 정보가 손실 없이 흐르도록 설계하여 성능과 효율성을 동시에 개선했다.32
  - **YOLOv11**: C3K2, C2PSA(Cross Stage Partial with Spatial Attention) 블록 등 CSP 구조를 기반으로 공간적 어텐션 메커니즘을 결합한 진화된 형태의 블록을 사용하여, 중요한 특징에 더 집중하면서 효율성을 유지하는 방향으로 발전했다.32

이러한 진화 과정은 CSPDarknet이 제시한 '부분적 연산(Partial Computation)' 철학이 얼마나 강력하고 확장성 있는 원칙인지를 보여준다. 이 철학은 '네트워크의 일부만 집중적으로 처리하고 나머지는 효율적으로 정보를 전달한다'는 개념으로 요약할 수 있다. 이는 단순히 서버급 GPU에서의 성능을 개선하는 것을 넘어, 모델을 YOLOv5n, YOLOv8n과 같이 더 작은 규모로 쉽게 스케일링할 수 있는 유연성을 제공했다. 결과적으로, CSPDarknet이 제시한 이 철학은 고성능 서버 환경뿐만 아니라, 연산 자원이 극도로 제한된 모바일 및 임베디드 시스템과 같은 엣지 디바이스(edge device) 환경까지 아우르는 현대 딥러닝 아키텍처 설계의 핵심 원리로 자리 잡았다.


CSPDarknet은 Darknet-53의 검증된 특징 추출 능력과 CSPNet의 혁신적인 연산 효율화 전략을 성공적으로 결합하여, 실시간 객체 탐지 분야의 성능 기준을 한 단계 끌어올린 기념비적인 아키텍처다. 이 아키텍처의 핵심 기여는 심층 신경망의 고질적인 문제였던 중복 경사도 문제를 정면으로 다루고, 네트워크 내부의 정보 흐름을 최적화함으로써 연산 비용을 절감하는 동시에 정확도를 향상시키는, 상충되어 보였던 두 가지 목표를 성공적으로 달성했다는 점에 있다.

YOLOv4의 백본으로서 CSPDarknet은 학술적 성과를 넘어 산업계 전반에 고성능 객체 탐지 기술의 보급을 가속화하는 데 결정적인 역할을 했다. 특히, 일반 소비자용 하드웨어에서도 훈련 및 운용이 가능하도록 설계되어 기술 민주화에 기여했으며, 이는 수많은 응용 분야에서 인공지능 기술의 실질적인 도입을 촉진했다.

더 나아가, CSPDarknet의 설계 철학은 일회성 성공에 그치지 않고 후속 YOLO 시리즈에 깊이 계승되어 C3, C2f 등 더욱 발전된 형태로 진화하며 현대 컴퓨터 비전 아키텍처 설계에 지속적인 영향을 미치고 있다. 이는 CSPDarknet이 제시한 '부분적 연산'과 '효율적 정보 흐름'이라는 원칙이 시대를 초월하는 보편적인 가치를 지니고 있음을 방증한다. 결론적으로, CSPDarknet은 단순히 '더 크게' 네트워크를 설계하는 것보다 '더 똑똑하게' 설계하는 것이 어떻게 더 우월한 결과를 낳을 수 있는지를 명확하게 증명한 탁월한 공학적 성취로 평가될 것이다.


1. CSP-DarkNet - Hugging Face, 8월 24, 2025에 액세스, https://huggingface.co/docs/timm/models/csp-darknet
2. YOLOv4: High-Speed and Precise Object Detection - Ultralytics YOLO Docs, 8월 24, 2025에 액세스, https://docs.ultralytics.com/models/yolov4/
3. Darknet 53 - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/computer-vision/darknet-53/
4. Which Backbone to Use: A Resource-efficient Domain Specific Comparison for Computer Vision - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2406.05612v1
5. Review — CSPNet: A New Backbone That Can Enhance Learning Capability of CNN, 8월 24, 2025에 액세스, https://sh-tsang.medium.com/review-cspnet-a-new-backbone-that-can-enhance-learning-capability-of-cnn-da7ca51524bf
6. CSPNet: A New Backbone that can Enhance Learning Capability of CNN - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/337590017_CSPNet_A_New_Backbone_that_can_Enhance_Learning_Capability_of_CNN
7. CSPNet: A New Backbone that can Enhance Learning Capability of ..., 8월 24, 2025에 액세스, https://arxiv.org/pdf/1911.11929
8. CSPNet: A New Backbone that can Enhance Learning Capability of CNN, 8월 24, 2025에 액세스, https://www.semanticscholar.org/paper/CSPNet%3A-A-New-Backbone-that-can-Enhance-Learning-of-Wang-Liao/b771c57b8fc242ba589d449e04e982e0577b1c54
9. What is YOLOv5: A deep look into the internal features of the popular object detector - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2407.20892v1
10. YOLOv4 — Version 3: Proposed Workflow - Medium, 8월 24, 2025에 액세스, https://medium.com/visionwizard/yolov4-version-3-proposed-workflow-e4fa175b902
11. CSPNet: A New Backbone that can Enhance Learning Capability of CNN - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/343270535_CSPNet_A_New_Backbone_that_can_Enhance_Learning_Capability_of_CNN
12. A Comprehensive Review of YOLO Architectures in Computer Vision: From YOLOv1 to YOLOv8 and YOLO-NAS - MDPI, 8월 24, 2025에 액세스, https://www.mdpi.com/2504-4990/5/4/83
13. The structure of the residual layer in Darknet53, CSPDarknet53 and ..., 8월 24, 2025에 액세스, https://www.researchgate.net/figure/The-structure-of-the-residual-layer-in-Darknet53-CSPDarknet53-and-our-model_fig2_358706263
14. Detailed information of Darknet-53 and CSPDarknet-53. - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/Detailed-information-of-Darknet-53-and-CSPDarknet-53_fig5_371594276
15. arXiv:2004.10934v1 [cs.CV] 23 Apr 2020, 8월 24, 2025에 액세스, https://arxiv.org/abs/2004.10934
16. Review — Mish: A Self Regularized Non-Monotonic Activation Function | by Sik-Ho Tsang, 8월 24, 2025에 액세스, https://sh-tsang.medium.com/review-mish-a-self-regularized-non-monotonic-activation-function-a7afe19b4af7
17. Mish: A Self Regularized Non-Monotonic Neural Activation ... - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/vc/arxiv/papers/1908/1908.08681v1.pdf
18. Mish: A Self Regularized Activation Function [TF] - Kaggle, 8월 24, 2025에 액세스, https://www.kaggle.com/code/samuelcortinhas/mish-a-self-regularized-activation-function-tf
19. torch.nn.functional.mish — PyTorch 2.7 documentation, 8월 24, 2025에 액세스, https://pytorch.org/docs/stable/generated/torch.nn.functional.mish.html
20. [2004.10934] YOLOv4: Optimal Speed and Accuracy of Object Detection - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2004.10934?sid=K1CjCk
21. (PDF) YOLOv4: Optimal Speed and Accuracy of Object Detection - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/340883401_YOLOv4_Optimal_Speed_and_Accuracy_of_Object_Detection
22. Achieving Optimal Speed and Accuracy in Object Detection (YOLOv4) - PyImageSearch, 8월 24, 2025에 액세스, https://pyimagesearch.com/2022/05/16/achieving-optimal-speed-and-accuracy-in-object-detection-yolov4/
23. YOLOv4 — the most accurate real-time neural network on MS COCO dataset. - Aleksey Bochkovskiy, 8월 24, 2025에 액세스, https://alexeyab84.medium.com/yolov4-the-most-accurate-real-time-neural-network-on-ms-coco-dataset-73adfd3602fe
24. Scaled-YOLOv4: Scaling Cross Stage Partial Network - OpenReview, 8월 24, 2025에 액세스, https://openreview.net/pdf?id=l-oAXqmMonv
25. Comparison of CSPDarkNet53, CSPResNeXt-50, and EfficientNet-B0 Backbones on YOLO V4 as Object Detector, 8월 24, 2025에 액세스, [http://download.garuda.kemdikbud.go.id/article.php?article=2999669&val=21071&title=Comparison%20of%20CSPDarkNet53%20CSPResNeXt-50%20and%20EfficientNet-B0%20Backbones%20on%20YOLO%20V4%20as%20Object%20Detector](http://download.garuda.kemdikbud.go.id/article.php?article=2999669&val=21071&title=Comparison+of+CSPDarkNet53+CSPResNeXt-50+and+EfficientNet-B0+Backbones+on+YOLO+V4+as+Object+Detector)
26. (PDF) Comparison of CSPDarkNet53, CSPResNeXt-50, and EfficientNet-B0 Backbones on YOLO V4 as Object Detector - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/363591482_Comparison_of_CSPDarkNet53_CSPResNeXt-50_and_EfficientNet-B0_Backbones_on_YOLO_V4_as_Object_Detector
27. EfficientNet vs ResNet: Technical Comparison for Classification Tasks – Neuronix Technology LLC | The Core of Machine Learning Culture, 8월 24, 2025에 액세스, https://neuronix.us/?p=48
28. YOLO Object Detection Explained: Evolution, Algorithm, and Applications - Encord, 8월 24, 2025에 액세스, https://encord.com/blog/yolo-object-detection-guide/
29. A Study on Object Detection Performance of YOLOv4 for Autonomous Driving of Tram - PMC, 8월 24, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC9696606/
30. (PDF) Research on real-time object detection based on Yolo algorithm - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/362623845_Research_on_real-time_object_detection_based_on_Yolo_algorithm
31. What is YOLOv8: An In-Depth Exploration of the Internal Features of the Next-Generation Object Detector - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2408.15857
32. YOLOv11 Architecture Explained: Next-Level Object Detection with Enhanced Speed and Accuracy | by S Nikhileswara Rao | Medium, 8월 24, 2025에 액세스, https://medium.com/@nikhil-rao-20/yolov11-explained-next-level-object-detection-with-enhanced-speed-and-accuracy-2dbe2d376f71
33. What is YOLOv6? A Deep Insight into the Object Detection Model - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2412.13006v1


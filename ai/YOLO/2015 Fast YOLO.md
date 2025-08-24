[YOLO 객체 탐지 모델](./index.md)
# Fast YOLO (2015)


객체 탐지(Object Detection) 기술의 역사는 정확도와 속도라는 두 가지 상충하는 목표 사이의 끊임없는 투쟁으로 점철되어 왔다. 특히 You Only Look Once(YOLO)의 등장은 이 분야의 패러다임을 근본적으로 바꾸어 놓은 혁명적 사건이었다. Fast YOLO의 진정한 의미를 이해하기 위해서는 먼저 그것이 탄생한 배경, 즉 YOLO 이전의 시대와 YOLO가 제시한 새로운 철학을 깊이 있게 살펴볼 필요가 있다.


YOLO가 등장하기 전, 객체 탐지 분야의 주류는 R-CNN(Regions with Convolutional Neural Networks) 계열로 대표되는 '2단계 탐지기(Two-Stage Detector)'였다.1 이 접근법은 문제를 두 개의 독립적인 단계로 나누어 해결했다. 첫 번째 단계는 '영역 제안(Region Proposal)'으로, 이미지 내에서 객체가 존재할 가능성이 있는 수천 개의 후보 영역(bounding box)을 생성한다.3 R-CNN은 이를 위해 선택적 탐색(Selective Search)과 같은 알고리즘을 사용했다.1 두 번째 단계는 '분류(Classification)'로, 제안된 각 후보 영역에 대해 컨볼루션 신경망(CNN)을 적용하여 해당 영역이 어떤 클래스의 객체를 포함하는지, 그리고 바운딩 박스의 위치가 얼마나 정확한지를 판별한다.4

이러한 2단계 파이프라인은 R-CNN에서 Fast R-CNN, 그리고 Faster R-CNN으로 발전하며 점차 개선되었다.1 Fast R-CNN은 전체 이미지에 대해 CNN 특징 맵을 한 번만 계산한 뒤, 제안된 영역들을 이 특징 맵에 투영하여 연산 중복을 줄였다.1 더 나아가 Faster R-CNN은 느린 선택적 탐색 알고리즘을 '영역 제안 네트워크(Region Proposal Network, RPN)'라는 별도의 신경망으로 대체하여 영역 제안 과정까지 GPU 연산에 통합시켰다.4

하지만 이러한 발전에도 불구하고 2단계 접근법은 본질적인 한계를 지니고 있었다. 다단계로 구성된 복잡한 파이프라인은 훈련 과정을 까다롭게 만들었으며, 각 구성요소를 개별적으로 최적화해야 하는 비효율성을 낳았다.7 무엇보다도, 수천 개의 후보 영역을 순차적으로 처리하는 구조는 추론 속도에 치명적인 병목 현상을 야기했다.8 이로 인해 2단계 탐지기는 높은 정확도를 달성했음에도 불구하고, 자율 주행, 실시간 영상 감시, 로보틱스와 같이 지연 시간에 민감한(latency-sensitive) 애플리케이션에는 적용하기 어려웠다.8


2015년, Joseph Redmon 등이 발표한 YOLOv1은 이러한 2단계 패러다임에 정면으로 도전했다.11 YOLO의 핵심 철학은 객체 탐지를 복잡한 다단계 파이프라인이 아닌, 이미지 전체 픽셀에서 바운딩 박스 좌표와 클래스 확률을 직접 예측하는 단일 회귀 문제(a single regression problem)로 재정의한 것이다.9 이는 단순히 단계를 줄인 것이 아니라, 문제의 본질을 바라보는 관점 자체를 바꾼 근본적인 전환이었다. 기존 방법들이 "어디에 객체가 있을까?"와 "이것은 무슨 객체인가?"를 순차적으로 물었다면, YOLO는 "이미지 전체를 보고, 각 영역에 어떤 객체가 어디에 있는지 한 번에 답하라"고 문제를 새롭게 정의했다.

이러한 패러다임의 전환은 객체 탐지 파이프라인을 단 하나의 통합된 신경망으로 응축시키는 결과를 낳았다.9 이 통합 아키텍처는 이미지를 단 한 번만(You Only Look Once) 바라보는, 즉 단일 순방향 전파(single forward pass)만으로 모든 예측을 동시에 수행할 수 있게 했다.15 그 결과, R-CNN보다 1000배, Fast R-CNN보다 100배 이상 빠른, 당시로서는 전례 없는 추론 속도를 달성하며 '실시간 객체 탐지'의 시대를 본격적으로 열었다.16 또한, 전체 파이프라인이 하나의 네트워크로 구성되어 있어 탐지 성능에 대해 직접적으로 종단간(end-to-end) 최적화가 가능해졌다.11


YOLOv1의 혁신적인 속도와 효율성은 몇 가지 독창적인 메커니즘에 기반한다.


YOLO는 영역 제안, 특징 추출, 분류, 위치 회귀 등 2단계 탐지기에서 분리되어 있던 모든 구성 요소를 단일 CNN 아키텍처 안에 통합했다.9 입력 이미지가 네트워크를 통과하면 최종적으로 예측 텐서(prediction tensor)가 출력되며, 이 텐서 안에는 이미지 내 모든 객체에 대한 위치, 신뢰도, 클래스 확률 정보가 모두 인코딩되어 있다.18 이처럼 모든 계산을 단일 네트워크에 캡슐화함으로써 복잡한 파이프라인을 제거하고 극적인 속도 향상을 이루었다.19


YOLOv1은 입력 이미지를 $S \times S$ 크기의 그리드로 분할한다 (원본 논문에서는 $S=7$을 사용).4 이미지에 존재하는 각 객체에 대해, 해당 객체의 중심점(center point)이 속한 그리드 셀이 그 객체를 탐지할 '책임(responsibility)'을 진다.4 각 그리드 셀은 $B$개의 바운딩 박스 예측과 각 박스에 대한 신뢰도 점수(confidence score), 그리고 $C$개의 클래스에 대한 조건부 클래스 확률(conditional class probabilities)을 예측하도록 훈련된다.17 이 그리드 기반의 공간 분할 방식은 바운딩 박스 예측을 공간적으로 제약하여 문제를 단순화하고, 전체 이미지에 걸쳐 예측을 병렬적으로 수행할 수 있게 하는 핵심적인 역할을 한다.20


YOLOv1의 가장 중요한 특징 중 하나는 훈련과 추론 과정에서 항상 이미지 전체를 본다는 점이다.9 슬라이딩 윈도우나 영역 제안 기반의 방법들은 이미지의 일부 영역만을 국소적으로 보기 때문에 주변 문맥 정보를 활용하기 어렵다.17 이로 인해 Fast R-CNN과 같은 모델은 배경의 일부를 객체로 오인하는 '배경 오류(background errors)'를 자주 범했다.17 반면, YOLO는 이미지 전체의 특징을 사용하여 각 바운딩 박스를 예측하므로, 객체의 외형뿐만 아니라 클래스 간의 관계나 주변 환경과 같은 전역적인 문맥 정보(global context)를 암묵적으로 학습하게 된다.9 그 결과, YOLOv1은 Fast R-CNN에 비해 배경 오류를 절반 이하로 줄일 수 있었으며, 이는 속도뿐만 아니라 특정 유형의 오류 감소 측면에서도 2단계 탐지기를 능가하는 중요한 이점이었다.17

이처럼 YOLOv1은 객체 탐지에 대한 근본적인 사고의 전환을 통해 속도, 단순성, 그리고 전역적 추론 능력이라는 새로운 가치를 제시했다. Fast YOLO는 바로 이러한 YOLOv1의 철학을 가장 극단적으로 추구한 파생 모델로서, 객체 탐지의 역사에서 '속도'라는 가치가 어디까지 도달할 수 있는지를 보여준 첫 번째 이정표였다.


YOLOv1 논문에서 기본 YOLO 모델과 함께 소개된 Fast YOLO는 정확도라는 가치를 과감하게 희생하는 대신, 실시간 처리 속도를 극한까지 밀어붙이는 것을 단 하나의 목표로 삼은 경량화 버전이다.11 이는 당시의 하드웨어 성능을 고려할 때, 실시간의 기준을 넘어 초고속 비디오 스트림 처리와 같은 미래의 응용 가능성을 탐색하려는 개념 증명(proof-of-concept)의 성격이 강했다.20


Fast YOLO의 설계 철학은 '단순함'과 '속도' 두 단어로 요약할 수 있다. 이는 복잡한 경량화 기법을 도입하기보다는, YOLOv1의 성공적인 통합 아키텍처의 골격은 그대로 유지한 채 네트워크의 규모와 복잡성만을 대폭 줄이는 직관적인 방식을 택했다.11 목표는 명확했다. 당시 어떤 실시간 탐지기도 도달하지 못했던 압도적인 프레임률(FPS)을 달성하여, YOLO 패러다임이 속도 측면에서 얼마나 강력한 잠재력을 가지고 있는지를 증명하는 것이었다.


Fast YOLO의 속도 향상은 전적으로 네트워크 아키텍처의 급진적인 단순화에서 비롯된다. 표준 YOLOv1과의 구조적 차이는 다음과 같다.

- **표준 YOLOv1**: 이 모델의 백본 네트워크는 이미지 분류 과제에서 우수한 성능을 보인 GoogLeNet 아키텍처에서 영감을 받아 설계되었다.11 총 24개의 합성곱 계층(convolutional layers)과 그 뒤를 잇는 2개의 완전 연결 계층(fully connected layers)으로 구성된다.11 이 구조는 당시 기준으로 비교적 깊고 복잡하여 정확도와 특징 추출 능력의 균형을 맞추고자 했다.
- **Fast YOLO**: Fast YOLO는 이 구조를 극단적으로 축소했다. 24개에 달했던 합성곱 계층을 **9개**로 대폭 줄였다.11 단순히 계층의 수만 줄인 것이 아니라, 각 합성곱 계층을 구성하는 필터의 개수 또한 감소시켜 모델의 전체적인 파라미터 수와 연산량(FLOPs)을 획기적으로 낮추었다.11

이러한 구조적 차이를 제외하고, 입력 이미지 크기, 그리드 시스템, 예측 방식, 그리고 후술할 손실 함수 등 훈련과 테스트에 사용되는 모든 매개변수는 표준 YOLOv1과 동일하게 유지되었다.11 이는 아키텍처의 크기 변화가 성능에 미치는 영향을 순수하게 비교하기 위한 통제된 실험 설계의 일환이었다. 결과적으로, 이러한 단순 축소 전략은 후대의 경량 모델들이 사용하는 깊이별 분리 합성곱(depthwise separable convolution)이나 네트워크 구조 탐색(NAS)과 같은 정교한 기법이 부재했던 시대적 상황 속에서, '속도'라는 목표를 가장 직접적이고 효과적으로 달성하는 방법이었다.


Fast YOLO는 표준 YOLOv1과 완전히 동일한 손실 함수를 사용하여 모델을 최적화한다. 이 손실 함수는 객체 탐지라는 복합적인 과업을 성공적으로 학습시키기 위해 위치 예측, 객체 존재 유무, 클래스 분류라는 세 가지 요소를 동시에 고려하는 다중 요소 손실 함수(multi-part loss function)이다.11 전체 손실은 단순 합산 제곱 오차(sum-squared error)를 기반으로 하지만, mAP(mean Average Precision)를 직접적으로 최적화하기 어려운 문제를 해결하고 학습을 안정화시키기 위해 여러 독창적인 장치가 고안되었다.11

전체 손실 함수는 다음과 같이 표현된다.
$$
\lambda_{coord}\sum_{i=0}^{S^2}\sum_{j=0}^{B} \mathbb{1}_{ij}^{obj} \left [ (x_i - \hat{x}_i)^2 + (y_i - \hat{y}_i)^2 \right ] \\
+ \lambda_{coord}\sum_{i=0}^{S^2}\sum_{j=0}^{B} \mathbb{1}_{ij}^{obj} \left [ (\sqrt{w_i} - \sqrt{\hat{w}_i})^2 + (\sqrt{h_i} - \sqrt{\hat{h}_i})^2 \right ] \\
+ \sum_{i=0}^{S^2}\sum_{j=0}^{B} \mathbb{1}_{ij}^{obj} (C_i - \hat{C}_i)^2 \\
+ \lambda_{noobj}\sum_{i=0}^{S^2}\sum_{j=0}^{B} \mathbb{1}_{ij}^{noobj} (C_i - \hat{C}_i)^2 \\
+ \sum_{i=0}^{S^2} \mathbb{1}_{i}^{obj} \sum_{c \in classes}(p_i(c) - \hat{p}_i(c))^2
$$
여기서 각 기호는 다음을 의미한다:

- $S^2$는 그리드의 총 셀 개수(7x7=49)이다.
- $B$는 셀 당 예측하는 바운딩 박스의 수(논문에서는 2)이다.
- $\mathbb{1}_{i}^{obj}$는 $i$번째 셀에 객체가 존재하면 1, 아니면 0인 지시 함수(indicator function)이다.
- $\mathbb{1}_{ij}^{obj}$는 $i$번째 셀의 $j$번째 예측기가 해당 객체 예측에 '책임'이 있으면 1, 아니면 0이다. '책임'은 해당 예측기가 실제 객체(ground truth)와의 Intersection over Union(IoU) 값이 가장 높을 때 부여된다.
- $(x, y, w, h)$는 바운딩 박스의 중심 좌표와 너비, 높이를 나타내며, $\hat{}$가 붙은 것은 모델의 예측값을 의미한다.
- $C$는 신뢰도 점수(confidence score), $p(c)$는 클래스 확률을 의미한다.

이 복잡한 수식은 다음과 같이 다섯 개의 항으로 나누어 이해할 수 있다.

1. **중심 좌표 오차 (Center Coordinate Error)**: 첫 번째 항. 객체가 존재하는($\mathbb{1}_{ij}^{obj}$) 셀의 '책임 있는' 예측기에 대해서만 바운딩 박스의 중심점 $(x, y)$ 예측 오차에 페널티를 부과한다.
2. **박스 크기 오차 (Box Size Error)**: 두 번째 항. 마찬가지로 책임 있는 예측기에 대해서만 바운딩 박스의 너비 $w$와 높이 $h$의 오차에 페널티를 부과한다.
3. **객체 존재 신뢰도 오차 (Object Confidence Error)**: 세 번째 항. 객체가 실제로 존재하는 셀의 신뢰도 점수 $C_i$가 1에 가깝도록 학습시킨다.
4. **객체 부재 신뢰도 오차 (No-Object Confidence Error)**: 네 번째 항. 객체가 존재하지 않는($\mathbb{1}_{ij}^{noobj}$) 대다수의 셀에서는 신뢰도 점수가 0에 가깝도록 학습시킨다.
5. **분류 오차 (Classification Error)**: 마지막 항. 객체가 존재하는($\mathbb{1}_{i}^{obj}$) 셀에 대해서만, 예측된 클래스 확률 $\hat{p}_i(c)$가 실제 클래스와 일치하도록 페널티를 부과한다.

이 손실 함수에는 학습을 안정시키고 성능을 향상시키는 몇 가지 핵심적인 파라미터와 기법이 포함되어 있다.

- $\lambda_{coord} = 5$: 이미지 대부분의 그리드 셀에는 객체가 존재하지 않으므로, 객체가 있는 셀에서 계산되는 위치 손실(좌표 및 크기 오차)은 상대적으로 드물게 발생한다. $\lambda_{coord}$ 파라미터는 이 위치 손실의 가중치를 5배 높여, 모델이 객체의 위치를 정확하게 학습하는 데 더 집중하도록 유도한다.11
- $\lambda_{noobj} = 0.5$: 반대로, 객체가 없는 셀에서 계산되는 신뢰도 손실은 매우 빈번하게 발생한다. 이 손실의 가중치를 0.5로 낮춤으로써, 모델이 배경 탐지에만 과도하게 치우쳐 객체 탐지 능력이 저하되는 것을 방지하고 학습의 균형을 맞춘다.11
- **너비/높이의 제곱근 예측**: 바운딩 박스의 크기가 클 때의 작은 오차와 작을 때의 작은 오차는 mAP에 미치는 영향이 다르다. 예를 들어, 500x500 박스에서 10픽셀의 오차는 미미하지만, 20x20 박스에서 10픽셀의 오차는 치명적이다. 너비 $w$와 높이 $h$를 직접 예측하는 대신 그 제곱근 $\sqrt{w}, \sqrt{h}$를 예측하도록 함으로써, 작은 박스의 오차에 더 큰 페널티를 부여하는 효과를 주어 이러한 불균형을 일부 완화하고자 했다.11


Fast YOLO의 성능은 PASCAL VOC 2007 데이터셋을 기준으로 측정되었으며, 그 결과는 설계 철학이 성공적으로 구현되었음을 명확히 보여준다.11

- **mAP (mean Average Precision)**: **52.7%** 11
- **FPS (Frames Per Second)**: **155** 11

이를 표준 YOLOv1 모델(mAP 63.4%, 45 FPS)과 비교해 보면, Fast YOLO는 정확도를 나타내는 mAP 지표에서 약 10.7%p의 상당한 손실을 보였다.11 하지만 속도를 나타내는 FPS는 3.4배 이상 비약적으로 향상되었다. 155 FPS라는 수치는 당시 다른 실시간 탐지기들의 성능을 압도하는 것이었으며, PASCAL VOC 데이터셋에서 기록된 가장 빠른 객체 탐지 방법이었다.11 이 극명한 트레이드오프는 Fast YOLO가 정확성을 희생하여 속도를 극대화한다는 목표를 성공적으로 달성했음을 정량적으로 입증한다.


Fast YOLO는 경량 실시간 탐지 모델의 개념을 제시한 선구자였지만, 기술의 발전은 멈추지 않았다. 이후 등장한 모델들은 Fast YOLO의 단순한 접근법을 넘어, 더 정교하고 지능적인 방식으로 속도와 정확도의 균형을 맞추고자 했다. Fast YOLO의 역사적 위상을 정확히 파악하기 위해서는 후대의 대표적인 경량 모델인 YOLOv3-tiny 및 MobileNet-SSD와의 비교 분석이 필수적이다.


YOLOv3-tiny는 2018년에 발표된 YOLOv3의 공식 경량화 버전으로, Fast YOLO와는 다른 차원의 기술적 진보를 보여준다.23


YOLOv3-tiny는 13개의 합성곱 계층과 6개의 풀링 계층으로 구성되어, Fast YOLO(9개 합성곱 계층)보다는 복잡하지만 표준 YOLOv3(Darknet-53 백본)보다는 훨씬 가볍고 빠르다.24 그러나 가장 결정적인 차이는 단순히 계층의 수가 아니라, YOLOv3의 핵심적인 개선 사항인 

**다중 스케일 예측(multi-scale prediction)** 개념을 계승했다는 점이다.24 YOLOv3-tiny는 네트워크의 서로 다른 깊이에서 특징 맵을 추출하여 두 개의 다른 스케일(예: 13x13, 26x26)에서 독립적으로 예측을 수행한다.24 상대적으로 거친 13x13 그리드에서는 큰 객체를, 더 세밀한 26x26 그리드에서는 작은 객체를 탐지하는 데 집중함으로써, 단일 스케일 예측에만 의존했던 YOLOv1 및 Fast YOLO의 고질적인 문제였던 작은 객체 탐지 성능을 크게 개선했다.


YOLOv3-tiny는 COCO 데이터셋에서 약 33.1%의 mAP@0.5를 기록했으며, Titan X GPU 환경에서 220 FPS에 달하는 매우 빠른 추론 속도를 보였다.16 PASCAL VOC 데이터셋에서의 직접적인 비교는 어렵지만, COCO 데이터셋에서의 성능을 고려할 때 Fast YOLO의 52.7% mAP(PASCAL VOC)보다 훨씬 높은 정확도를 달성했을 것으로 추정된다.

이는 경량 모델 설계 철학의 중요한 발전을 시사한다. Fast YOLO가 단순히 네트워크의 깊이와 너비를 줄이는 '물리적 축소'에 의존했다면, YOLOv3-tiny는 다중 스케일 예측과 같은 핵심적인 아키텍처 개선을 통해 성능 저하를 최소화하며 '지능적으로' 경량화를 달성했다. 즉, 무조건 작게 만드는 것이 아니라, 중요한 기능은 유지하면서 효율성을 높이는 방향으로 진화한 것이다.


MobileNet-SSD는 YOLO 계열과는 다른 철학에서 출발한 대표적인 경량 탐지 모델이다. 이 모델은 구글이 제안한 두 가지 핵심 기술의 결합체이다.


1. **MobileNet 백본**: MobileNet의 핵심 아이디어는 기존의 표준 합성곱 연산을 **깊이별 분리 합성곱(Depthwise Separable Convolution)**으로 대체하는 것이다.28 표준 합성곱이 공간적 특징(spatial feature)과 채널 간 특징(cross-channel feature)을 한 번에 학습하는 반면, 깊이별 분리 합성곱은 이를 두 단계로 분리한다. 먼저 '깊이별 합성곱(depthwise convolution)'이 각 채널에 대해 독립적으로 공간적 특징을 추출하고, 그 다음 '점별 합성곱(pointwise convolution, 1x1 convolution)'이 이 결과들을 선형적으로 결합하여 채널 간 특징을 학습한다.28 이 구조는 연산의 본질을 바꿈으로써 파라미터 수와 계산량을 표준 합성곱 대비 8~9배가량 획기적으로 줄이면서도 정확도 손실은 최소화한다.28

2. **SSD 헤드**: Single Shot MultiBox Detector(SSD)는 YOLO와 유사한 1단계 탐지기이지만, 객체 예측 방식에 차이가 있다.19 YOLO가 그리드 셀을 기반으로 직접 좌표를 회귀하는 반면, SSD는 각 특징 맵의 위치마다 다양한 크기와 종횡비를 가진 사전 정의된 

   **기본 박스(default boxes)**, 즉 앵커 박스(anchor boxes)를 사용한다.19 모델은 이 기본 박스들을 기준으로 객체의 존재 확률과 위치의 미세 조정 값(offset)을 예측한다. 이 방식은 다양한 형태의 객체에 더 유연하게 대응할 수 있도록 돕는다.


MobileNetV2 논문에서는 한 단계 더 나아간 **SSDLite**가 제안되었다.30 SSDLite는 MobileNetV2를 백본으로 사용할 뿐만 아니라, SSD의 예측 헤드(prediction head)에 있는 표준 합성곱 연산까지 모두 깊이별 분리 합성곱으로 대체하여 모델 전체의 경량화를 극대화한 버전이다.30


MobileNetV2-SSDLite는 COCO 데이터셋에서 22.1%의 mAP(@0.5:0.95)를 기록했으며, 모바일 CPU(Google Pixel 1) 환경에서 약 200ms(5 FPS)의 추론 시간을 보였다.30 이는 GPU가 아닌 저전력 엣지 디바이스에서의 실시간 구동을 목표로 설계되었음을 명확히 보여준다. GPU 환경에서는 훨씬 빠른 속도가 기대되지만, 직접적인 FPS 비교는 어렵다.32

Fast YOLO와 MobileNet-SSD의 비교는 경량화에 대한 두 가지 근본적으로 다른 접근 방식을 보여준다. Fast YOLO가 '네트워크 구조를 작게' 만드는 데 집중했다면, MobileNet-SSD는 '연산 자체를 효율적으로' 만드는 데 집중했다. 후자의 접근 방식, 즉 깊이별 분리 합성곱은 이후 거의 모든 현대 경량 CNN 모델(예: ShuffleNet, EfficientNet 등)의 표준적인 구성 요소가 되었으며, 이는 MobileNet의 철학이 경량 모델 설계의 주류가 되었음을 의미한다.


아래 표는 세 가지 경량 모델의 핵심 특징과 성능을 요약하여 비교한 것이다. 이 표는 각 모델의 설계 철학이 정확도, 속도, 모델 크기와 같은 구체적인 성능 지표에 어떻게 반영되는지를 명확하게 보여줌으로써, 경량 모델 간의 복잡한 트레이드오프 관계를 직관적으로 이해하는 데 도움을 준다.

| 모델 (Model)            | 백본 (Backbone)  | mAP (PASCAL VOC07) | mAP (COCO)         | FPS (GPU)           | 파라미터 수 (Parameters) | 핵심 특징 (Key Feature)             |
| ----------------------- | ---------------- | ------------------ | ------------------ | ------------------- | ------------------------ | ----------------------------------- |
| **Fast YOLO**           | Custom (9 Conv)  | 52.7% 11           | N/A                | **155** 11          | Low (미발표)             | 극단적 속도 최적화 (단순 계층 축소) |
| **YOLOv3-tiny**         | Custom (13 Conv) | ~57% (추정치)      | 33.1% (@0.5) 27    | ~220 (Titan X)      | ~8.8M 27                 | 다중 스케일 예측 도입               |
| **MobileNetV2-SSDLite** | MobileNetV2      | ~70% (추정치)      | 22.1% (@.5:.95) 30 | ~30 (모바일 CPU) 32 | ~4.3M 30                 | 깊이별 분리 합성곱 (연산 효율화)    |

*주: FPS는 측정 하드웨어 및 환경에 따라 크게 달라질 수 있으며, mAP 또한 훈련 데이터 및 세부 설정에 따라 변동될 수 있다. 위 표는 각 모델의 상대적인 특성을 비교하기 위한 것이다.*

분석 결과, Fast YOLO는 가장 원시적인 경량화 기법을 사용했지만 가장 직접적인 방식으로 '속도'라는 목표를 달성했다. YOLOv3-tiny는 더 발전된 아키텍처를 통해 속도를 유지하면서 정확도를 크게 개선했으며, MobileNet-SSD는 연산 자체의 효율성을 혁신하여 특히 모바일 환경에서의 저전력 구동에 새로운 길을 열었다. 이들의 관계는 경량 객체 탐지 기술의 진화 과정을 압축적으로 보여준다.


Fast YOLO는 객체 탐지 역사에서 중요한 이정표를 세웠지만, 그 기술적 한계 또한 명확했다. 이 모델의 진정한 가치는 오늘날의 실용성에 있는 것이 아니라, 그것이 남긴 유산과 후대 연구에 미친 영향에서 찾아야 한다.


Fast YOLO는 YOLOv1의 아키텍처를 그대로 축소한 모델이므로, YOLOv1이 가진 모든 구조적 단점을 그대로, 혹은 더 심화된 형태로 물려받았다. YOLOv1의 '원죄'라 불릴 수 있는 이러한 한계들은 다음과 같다.

- **부정확한 위치 특정 (Poor Localization)**: YOLOv1의 가장 큰 약점 중 하나는 바운딩 박스의 위치를 정밀하게 예측하지 못한다는 점이다.9 7x7이라는 상대적으로 거친(coarse) 그리드에 기반한 예측 방식은 객체의 경계를 세밀하게 포착하는 데 근본적인 한계를 가졌다.8 Fast YOLO는 더 얕은 네트워크 구조로 인해 특징 추출 능력이 저하되면서 이러한 위치 부정확성 문제가 더욱 두드러졌을 가능성이 높다.

- **작은 객체 탐지 성능 저하 (Difficulty with Small Objects)**: 여러 개의 다운샘플링(max-pooling) 계층을 거치면서 이미지의 해상도는 점차 낮아지고, 이 과정에서 작은 객체들이 가진 미세한 특징 정보는 쉽게 소실된다.10 또한, 그리드 셀 하나의 크기가 상대적으로 크기 때문에 여러 개의 작은 객체가 하나의 셀에 포함되거나, 아예 탐지되지 못하는 경우가 빈번했다.8 이는 단일 스케일에서만 예측을 수행하는 YOLOv1 아키텍처의 고질적인 문제였다.10

- **군집 객체 탐지 불가 (Inability to Detect Clustered Objects)**: YOLOv1의 설계상 각 그리드 셀은 단 하나의 클래스 집합만을 예측할 수 있다.17 또한, 셀 당 예측하는 바운딩 박스의 수(

  $B=2$)도 제한적이다. 이로 인해 여러 객체의 중심이 하나의 그리드 셀 안에 밀집해 있는 경우, 모델은 그 중 일부만을 탐지할 수밖에 없었다.8 이는 조류 떼나 군중과 같이 객체들이 군집을 이루는 시나리오에서 치명적인 약점으로 작용했다.

- **새로운 종횡비에 대한 일반화 부족 (Poor Generalization to New Aspect Ratios)**: YOLOv1은 앵커 박스(anchor box)라는 사전 정의된 형태의 박스를 사용하지 않고, 바운딩 박스의 너비와 높이를 직접 예측한다.21 이 방식은 구조적으로는 간단하지만, 훈련 데이터에서 보지 못했던 새로운 형태나 극단적인 종횡비(aspect ratio)를 가진 객체에 대해서는 일반화 성능이 떨어지는 경향이 있었다.33

이러한 한계들로 인해 Fast YOLO는 압도적인 속도에도 불구하고 범용적인 객체 탐지기로서 널리 사용되지는 못했다. 그 역할은 실용적인 솔루션이라기보다는, 특정 가능성을 탐험하는 실험적 모델에 가까웠다.


그럼에도 불구하고 Fast YOLO가 객체 탐지 기술의 발전에 기여한 바는 결코 작지 않다. 그 유산은 '모델 그 자체'가 아니라 '가능성의 증명'에 있다.

- **'속도'의 재정의**: Fast YOLO의 가장 큰 역사적 기여는 **155 FPS**라는 경이로운 속도를 달성함으로써 '실시간'의 기준을 완전히 새로운 차원으로 끌어올렸다는 점이다.11 당시 45 FPS를 기록한 표준 YOLOv1도 매우 빠른 것으로 평가받았지만, 155 FPS는 일반적인 비디오 프레임률(30~60 FPS)을 훨씬 뛰어넘는 수치였다. 이는 단순한 성능 지표 향상을 넘어, 비디오 스트림을 어떠한 지연도 없이 처리할 수 있다는 가능성을 현실로 보여준 상징적인 성과였다. 이 충격적인 수치는 컴퓨터 비전 커뮤니티에 "정확도를 어느 정도 포기한다면, 속도는 어디까지 빨라질 수 있는가?"라는 새로운 화두를 던졌다.
- **'Tiny' 모델 계보의 시조**: Fast YOLO의 성공적인 실험은 '정확도를 다소 희생하더라도 압도적인 속도를 확보하는' 경량 모델 설계 철학의 유효성을 입증했다. 이는 이후 YOLOv2-tiny, YOLOv3-tiny와 같이 공식적으로 'tiny'라는 이름이 붙은 경량 버전들의 탄생에 직접적인 영감을 주었다.35 더 나아가, 제한된 연산 자원을 가진 엣지 디바이스나 모바일 환경을 위한 수많은 경량 모델 연구의 기폭제 역할을 했다. 오늘날 우리가 접하는 수많은 'Lite', 'Nano', 'Tiny' 모델들은 모두 Fast YOLO가 개척한 '속도 우선주의' 철학의 연장선상에 있다고 볼 수 있다. 즉, Fast YOLO는 실용적인 모델은 아니었을지라도, 후속 연구를 촉발시킨 강력한 '자극제(catalyst)'이자 새로운 경량 모델들의 성능을 가늠하는 '벤치마크(benchmark)'로서의 역사적 역할을 충실히 수행했다.


전문가적 논의의 정확성을 기하기 위해, 2015년 YOLOv1 논문의 'Fast YOLO'와 이름이 같은 다른 모델을 명확히 구분할 필요가 있다. 2017년, Mohammad Javad Shafiee 연구팀은 "Fast YOLO: A Fast You Only Look Once System for Real-time Embedded Object Detection in Video"라는 제목의 논문을 발표했다.36

이 2017년의 'Fast YOLO'는 **YOLOv2**를 기반으로 한다.36 이 모델의 핵심 아이디어는 두 가지다. 첫째, 진화 연산 기반의 '진화형 딥 인텔리전스(evolutionary deep intelligence)' 프레임워크를 사용하여 YOLOv2의 네트워크 아키텍처를 자동적으로 최적화(O-YOLOv2)했다.37 둘째, 비디오 프레임 간의 움직임 변화를 감지하여 불필요한 추론을 건너뛰는 '모션 적응형 추론(motion-adaptive inference)' 기법을 도입했다.37 이 모델의 목표는 Nvidia Jetson TX1과 같은 임베디드 장치에서 비디오 객체 탐지를 가속화하는 것이었다.38

결론적으로, 본 보고서에서 심층적으로 다루는 **원조 Fast YOLO**는 2015년 Redmon의 YOLOv1 논문에서 제안된, 단순히 네트워크 계층을 줄여 속도를 극대화한 모델이다. 반면 2017년의 Fast YOLO는 YOLOv2를 기반으로 임베디드 비디오 처리에 특화된 별개의 모델이다. 이 둘은 기반 모델, 최적화 방법, 목표 애플리케이션이 완전히 다르므로 혼동해서는 안 된다.


Fast YOLO는 YOLOv1의 통합 아키텍처를 극단적으로 단순화하는 직설적인 접근법을 통해, 155 FPS라는 전례 없는 처리 속도를 달성한 선구적인 경량 객체 탐지 모델이었다. 이 모델의 핵심 기여는 실용적인 솔루션을 제공한 것이 아니라, 객체 탐지 분야에서 '속도'를 최우선 가치로 두는 설계 철학의 극한 가능성을 가시적으로 입증했다는 데 있다. 이는 당시의 기술적 지평을 넓히고, '실시간'의 개념을 재정의하는 계기가 되었다.

그러나 이러한 성과는 심각한 정확도 저하를 대가로 한 것이었다. Fast YOLO는 기반 모델인 YOLOv1의 근본적인 한계, 즉 부정확한 위치 특정, 작은 객체 및 군집 객체 탐지 실패 등의 문제를 고스란히, 혹은 더욱 심화된 형태로 안고 있었다. 기술적 관점에서 보면, Fast YOLO의 '단순 축소' 전략은 이후 등장한 깊이별 분리 합성곱, 다중 스케일 예측, 네트워크 구조 탐색 등 정교하고 효율적인 경량화 기법들에 비해 상대적으로 원시적인 접근법에 머물렀다.

그럼에도 불구하고 Fast YOLO가 남긴 교훈은 명확하다. 이 모델이 처음으로 던졌던 "제한된 자원 하에서 어떻게 최적의 성능을 낼 것인가?"라는 근본적인 질문은, 엣지 AI와 모바일 컴퓨팅이 보편화된 오늘날 오히려 더욱 중요한 화두가 되었다. 비록 그 방법론은 역사 속으로 사라졌지만, 그 도전 정신은 YOLO-tiny 계열을 비롯한 수많은 후속 경량 모델들의 탄생을 이끌었다.

결론적으로, Fast YOLO는 객체 탐지 기술의 역사에서 속도와 정확성이라는 영원한 트레이드오프 관계를 탐험했던 중요한 첫걸음으로 평가되어야 한다. 그것은 완벽한 해결책이 아니었지만, 가능성의 문을 열어젖힌 중요한 이정표였으며, 그 유산은 오늘날의 최첨단 경량 모델들의 설계 철학 속에 여전히 살아 숨 쉬고 있다.


1. In-Depth Review of YOLOv1 to YOLOv10 Variants for Enhanced Photovoltaic Defect Detection - MDPI, 8월 24, 2025에 액세스, https://www.mdpi.com/2673-9941/4/3/16
2. YOLO v3-Tiny: Object Detection and Recognition using one stage improved model, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/340893960_YOLO_v3-Tiny_Object_Detection_and_Recognition_using_one_stage_improved_model
3. What is YOLO? The Ultimate Guide [2025] - Roboflow Blog, 8월 24, 2025에 액세스, https://blog.roboflow.com/guide-to-yolo-models/
4. YOLOv1 Explained by A.J. Paper: You Only Look Once: Unified… | by Jesse Annan, 8월 24, 2025에 액세스, https://medium.com/@jesse419419/yolov1-explained-by-a-j-7fe408764c55
5. A brief history of Object Detection - Kanishk's Kaleidoscope, 8월 24, 2025에 액세스, https://kanishkmunot.hashnode.dev/introduction-to-object-detection
6. COCO Object Detection on Benchmarks.AI, 8월 24, 2025에 액세스, https://benchmarks.ai/coco-detection
7. Comparison of YOLOv3, YOLOv5s and MobileNet-SSD V2 for Real-Time Mask Detection, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/353211011_Comparison_of_YOLOv3_YOLOv5s_and_MobileNet-SSD_V2_for_Real-Time_Mask_Detection
8. YOLOv1 to YOLOv11: A Comprehensive Survey of Real-Time Object Detection Innovations and Challenges - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2508.02067v1
9. You Only Look Once: Unified, Real-Time Object Detection - The Computer Vision Foundation, 8월 24, 2025에 액세스, https://www.cv-foundation.org/openaccess/content_cvpr_2016/papers/Redmon_You_Only_Look_CVPR_2016_paper.pdf
10. Concept of YOLOv1:The Evolution of Real-Time Object Detection | by Sachinsoni | Medium, 8월 24, 2025에 액세스, https://medium.com/@sachinsoni600517/concept-of-yolov1-the-evolution-of-real-time-object-detection-d773770ef773
11. arXiv:1506.02640v5 [cs.CV] 9 May 2016, 8월 24, 2025에 액세스, https://arxiv.org/abs/1506.02640
12. [PDF] You Only Look Once: Unified, Real-Time Object Detection - Semantic Scholar, 8월 24, 2025에 액세스, https://www.semanticscholar.org/paper/You-Only-Look-Once%3A-Unified%2C-Real-Time-Object-Redmon-Divvala/f8e79ac0ea341056ef20f2616628b3e964764cfd
13. You Only Look Once: Unified, Real-Time Object Detection | Request PDF - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/311609522_You_Only_Look_Once_Unified_Real-Time_Object_Detection
14. You-Only-Look-Once: Object Detection with YOLO | by Zia Babar | Medium, 8월 24, 2025에 액세스, https://medium.com/@zbabar/you-only-look-once-object-detection-with-yolo-de9fd5455306
15. You Only Look Once - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/You_Only_Look_Once
16. YOLO: Real-Time Object Detection - Joseph Redmon, 8월 24, 2025에 액세스, https://pjreddie.com/darknet/yolo/
17. You Only Look Once: Unified, Real-Time Object Detection - IEEE Computer Society, 8월 24, 2025에 액세스, https://www.computer.org/csdl/proceedings-article/cvpr/2016/8851a779/12OmNzFv4de
18. Review (You Only Look Once: Unified, Real-Time Object Detection) | by MAROUA CHANBI, 8월 24, 2025에 액세스, https://medium.com/data-and-beyond/review-you-only-look-once-unified-real-time-object-detection-7e148e326736
19. SSD: Single Shot MultiBox Detector, 8월 24, 2025에 액세스, http://arxiv.org/abs/1512.02325
20. YOLO model for real-time object detection: A full guide | Viam, 8월 24, 2025에 액세스, https://www.viam.com/post/guide-yolo-model-real-time-object-detection-with-examples
21. YOLOv1 to YOLOv10: The fastest and most accurate real-time object detection systems - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2408.09332v1
22. You Only Look Once: Unified, Real-Time Object Detection - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=NM6lrxy0bxs
23. [1804.02767] YOLOv3: An Incremental Improvement - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/1804.02767
24. An improved Tiny YOLOv3 for real-time object detection - Taylor & Francis Online, 8월 24, 2025에 액세스, https://www.tandfonline.com/doi/full/10.1080/21642583.2021.1901156
25. YOLOv3 - Tethys, 8월 24, 2025에 액세스, https://tethys.pnnl.gov/sites/default/files/publications/Redmonetal.pdf
26. YOLO Evolution: A Comprehensive Benchmark and Architectural Review of YOLOv12, YOLO11, and Their Previous Versions - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2411.00201v2
27. tianhai123/yolov3: yolo3 - GitHub, 8월 24, 2025에 액세스, https://github.com/tianhai123/yolov3
28. arXiv:1704.04861v1 [cs.CV] 17 Apr 2017, 8월 24, 2025에 액세스, https://arxiv.org/pdf/1704.04861
29. REAL TIME OBJECT DETECTION USING MOBILENET-SSD WITH OPENCV - IJESAT, 8월 24, 2025에 액세스, https://www.ijesat.com/ijesat/files/V24I0534_1715094414.pdf
30. MobileNetV2: Inverted Residuals and Linear Bottlenecks, 8월 24, 2025에 액세스, https://arxiv.org/abs/1801.04381
31. MobileDenseNet: A new approach to object detection on mobile devices - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2207.11031
32. Slow FPS using SSD-Mobilenetv2 - Jetson Nano - NVIDIA Developer Forums, 8월 24, 2025에 액세스, https://forums.developer.nvidia.com/t/slow-fps-using-ssd-mobilenetv2/107618
33. A Comprehensive Review of YOLO Architectures in Computer Vision: From YOLOv1 to YOLOv8 and YOLO-NAS - MDPI, 8월 24, 2025에 액세스, https://www.mdpi.com/2504-4990/5/4/83
34. Deep dive into YOLOv1 - Medium, 8월 24, 2025에 액세스, https://medium.com/@abhishekjainindore24/deep-dive-into-yolov1-c70111debe60
35. Systematic study of lightweight for object detection - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/382032620_Systematic_study_of_lightweight_for_object_detection
36. Fast YOLO: A Fast You Only Look Once System for Real-time Embedded Object Detection in Video - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/386911581_Fast_YOLO_A_Fast_You_Only_Look_Once_System_for_Real-time_Embedded_Object_Detection_in_Video
37. Fast YOLO: A Fast You Only Look Once System for Real-time Embedded Object Detection in Video - University of Waterloo, 8월 24, 2025에 액세스, https://openjournals.uwaterloo.ca/index.php/vsl/article/download/171/141
38. [1709.05943] Fast YOLO: A Fast You Only Look Once System for Real-time Embedded Object Detection in Video - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/1709.05943


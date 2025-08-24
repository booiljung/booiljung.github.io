# YOLO 버전별 비교
[YOLO 객체 탐지 모델](./index.md)


본 보고서는 객체 탐지(object detection) 모델 계열인 YOLO(You Only Look Once)에 대한 포괄적인 비교 분석을 제공합니다. 기념비적인 YOLOv1부터 최신 기술을 대표하는 버전에 이르기까지, 각 세대를 정의한 아키텍처 혁신, 훈련 방법론, 그리고 성능 트레이드오프를 추적하고 해부합니다. 본 분석은 컴퓨터 비전 연구자 및 실무자들을 위한 결정적인 기술 참고 자료로서, 모델 선택과 향후 연구 방향에 대한 정보를 제공하기 위해 세분화된 상세 정보와 고차원적인 통찰을 모두 담아 구성되었습니다.



YOLO의 근본적인 혁신은 객체 탐지를 다단계 분류 문제(multi-stage classification task)가 아닌 단일 회귀 문제(single regression problem)로 재정의한 데 있습니다.1 기존의 2단계 탐지기(two-stage detector)들, 예를 들어 R-CNN이나 Fast R-CNN은 먼저 후보 영역(region proposal)을 생성한 후 이를 분류하는 방식을 사용했습니다.3 반면, YOLO는 단일 신경망의 순전파(forward pass) 한 번으로 위치 파악(localization)과 분류(classification)를 동시에 수행합니다.2 이러한 통합 아키텍처는 YOLO가 실시간 애플리케이션에 이상적인 탁월한 속도를 달성하는 핵심 열쇠가 되었습니다.1

이러한 접근 방식은 단순히 우연히 빨라진 것이 아니라, 당시 지배적이었던 '정확도 우선'의 2단계 탐지기 패러다임에서 의도적으로 벗어난 철학적 선택이었습니다. 기존 방법들이 계산 비용이 많이 들고 고사양 서버에서나 가능했던 객체 탐지를, YOLO는 보다 접근성 높은 하드웨어에서도 실시간으로 처리할 수 있도록 민주화했습니다. YOLOv1이 45 FPS, 경량화 버전인 Fast YOLO가 155 FPS라는 당시로서는 경이적인 속도를 달성한 것은 이를 방증합니다.6 물론 초기 버전은 2단계 탐지기보다 정확도가 낮았지만 8, 이는 개발팀이 실시간 성능을 최우선으로 고려하여 객체 탐지 분야에 새로운 틈새시장을 창출했음을 보여줍니다.


YOLO의 분석은 그리드 기반 예측 시스템에서 시작됩니다. 입력 이미지를 $S \times S$ 그리드(예: YOLOv1의 경우 $7 \times 7$)로 분할하고, 각 그리드 셀은 객체의 중심점이 자신 안에 위치할 경우 해당 객체에 대한 경계 상자(bounding box)와 클래스 확률을 예측하는 책임을 집니다.2

이 접근법의 결정적인 장점은 모델이 훈련 및 추론 과정에서 이미지 전체를 한 번에 본다는 것입니다. 이를 통해 모델은 객체의 전역적인 문맥(global context)을 추론할 수 있으며, 배경 영역에서 발생하는 거짓 양성(false positive)을 줄이는 데 도움이 됩니다.3 이는 이미지의 일부 영역만 보는 슬라이딩 윈도우(sliding window)나 후보 영역 기반 방식과는 대조적입니다.3

하지만 전역적 문맥을 확보하는 이점은 양날의 검과 같았습니다. 이 방식은 YOLOv1의 주요 약점, 즉 작거나 밀집된 객체를 탐지하기 어려운 문제의 원인이 되기도 했습니다. 단일 그리드 시스템은 여러 개의 작은 객체들이 동일한 그리드 셀에 포함될 때 어려움을 겪는데, 이는 각 셀이 제한된 수의 경계 상자와 단 하나의 클래스 집합만을 예측할 수 있었기 때문입니다.9 전역적 문맥이라는 장점을 제공한 그리드 시스템 아키텍처가 동시에 이러한 핵심적인 한계를 야기한 것이며, 이 내재된 긴장 관계가 이후 YOLO 버전들(예: YOLOv3의 다중 스케일 예측, YOLOv2의 앵커 박스)의 진화를 이끄는 주요 동력이 되었습니다.



- **아키텍처**: GoogLeNet에서 영감을 받은 단일 CNN으로, 24개의 합성곱 레이어(convolutional layer)와 2개의 완전 연결 레이어(fully connected layer)로 구성됩니다.4 이 설계는 완전 합성곱(fully convolutional) 방식이 아니어서 고정된 크기(448x448)의 입력만 처리할 수 있었습니다.9
- **예측 메커니즘**: 입력 이미지를 $7 \times 7$ 그리드로 나눕니다. 각 그리드는 2개의 경계 상자(각각 5개의 값: x, y, w, h, 신뢰도)와 20개의 클래스 확률(PASCAL VOC 데이터셋 기준)을 예측하여, 최종적으로 $7 \times 7 \times 30$ 크기의 출력 텐서를 생성합니다.3
- **장점**: Titan X GPU에서 45 FPS라는 전례 없는 속도로 실시간 탐지를 가능하게 했습니다.6 또한 예술 작품과 같은 새로운 도메인에 대한 일반화 성능이 우수했습니다.6
- **한계**:
  - 거친 그리드와 셀당 제한된 예측 수로 인해 작거나 밀집된 객체 그룹에 대한 성능이 저조했습니다.10
  - 사전 정보(prior) 없이 데이터로부터 직접 상자 형태를 학습하기 때문에 새롭거나 특이한 종횡비의 객체를 다루는 데 어려움을 겪었습니다.10
  - 모든 오차를 동일하게 취급하는 단순한 제곱합 오차(Sum-of-Squared-Error) 손실 함수로 인해 2단계 탐지기에 비해 위치 파악 오류가 컸습니다.10


- **아키텍처 개선**:
  - **백본(Backbone)**: 더 효율적인 19개 레이어로 구성된 새로운 백본인 **Darknet-19**를 도입했습니다.8
  - **정규화(Normalization)**: 모든 합성곱 레이어 뒤에 **배치 정규화(Batch Normalization)**를 추가했습니다. 이는 훈련을 안정화하고 수렴을 개선하며 정규화 효과를 제공하여 $mAP$를 2% 이상 향상시켰습니다.2
- **핵심 혁신**:
  - **앵커 박스(Anchor Boxes)**: 중추적인 변화였습니다. 모델이 상자 좌표를 직접 예측하는 대신, 후보 영역 네트워크(RPN)에서 차용한 사전 정의된 '앵커 박스'로부터의 오프셋(offset)을 예측하도록 했습니다.2 이는 문제를 단순화하고 네트워크의 학습을 용이하게 만들었습니다.
  - **차원 클러스터링(Dimension Clusters)**: 앵커 박스를 수동으로 선택하는 대신, 훈련 세트의 경계 상자에 k-평균 클러스터링(k-means clustering)을 적용하여 데이터에 더 적합한 가장 일반적인 형태와 크기의 사전 정보를 찾아냈습니다.14
  - **고해상도 분류기 및 다중 스케일 훈련**: 탐지 훈련 전에 백본을 더 높은 해상도(448x448)의 분류 이미지로 미세 조정했습니다. 이후 탐지 모델은 다양한 크기의 이미지로 훈련되어 여러 스케일에 강건해졌습니다.3
  - **세분화된 특징 (Passthrough Layer)**: 작은 객체 탐지를 개선하기 위해, 초기 고해상도 레이어의 특징을 깊은 저해상도 특징과 연결하는 패스스루 레이어를 도입하여 더 세밀한 정보를 제공했습니다.14
- **YOLO9000**: COCO 탐지 데이터셋과 ImageNet 분류 데이터셋에서 공동으로 훈련된 변형 모델로, 경계 상자 레이블이 없는 수많은 카테고리를 포함하여 9000개 이상의 객체 카테고리를 탐지할 수 있었습니다.3


- **아키텍처 개선**:
  - **백본**: 훨씬 더 깊은 **Darknet-53** 백본을 도입했습니다. 이는 ResNet과 유사한 잔차 연결(residual connection)을 통합하여 기울기 소실(vanishing gradient) 문제 없이 깊이를 늘릴 수 있게 했습니다. 이로 인해 특징 추출 능력은 향상되었지만 속도는 다소 저하되었습니다.13
  - **넥(Neck) (다중 스케일 예측)**: 가장 중요한 변화였습니다. YOLOv3는 **특징 피라미드 네트워크(FPN, Feature Pyramid Network)**와 유사한 넥 구조를 도입했습니다. 특징들을 업샘플링하고 연결하여 세 가지 다른 스케일(13x13, 26x26, 52x52 크기의 특징 맵)에서 예측을 수행함으로써 작은 객체에 대한 성능을 극적으로 향상시켰습니다.4
- **핵심 혁신**:
  - **클래스 예측**: 단일 `softmax` 분류기를 각 클래스에 대한 독립적인 **로지스틱 분류기(logistic classifier)**로 대체했습니다. 이는 다중 레이블 예측(예: 한 객체가 '여성'이면서 동시에 '사람'일 수 있음)을 가능하게 했고, COCO와 같이 중첩되는 카테고리가 있는 데이터셋에 더 적합했습니다.14
  - **손실 함수**: 클래스 및 객체성(objectness) 예측을 위해 제곱합 오차에서 **이진 교차 엔트로피(BCE, Binary Cross-Entropy)**로 전환했으며, 이는 분류 작업에 더 표준적인 방식입니다.14

이 초기 시대의 진화는 무작위적인 개선이 아니었습니다. v1에서 v3로의 발전은 이전 버전에서 명확히 드러난 약점들에 대한 직접적이고 논리적인 대응이었습니다. 예를 들어, YOLOv1의 주요 결함은 부정확한 위치 파악과 작은 객체 탐지 성능이었습니다.10 YOLOv2는 이를 해결하기 위해 더 나은 형태 사전 정보를 위한 앵커 박스와 세분화된 특징을 위한 패스스루 레이어를 도입했습니다.2 하지만 YOLOv2 역시 다양한 스케일의 객체를 다루는 데 한계가 있었고, YOLOv3는 완전한 FPN 스타일의 넥을 도입하여 세 가지 다른 스케일에서 예측을 수행함으로써 이 다중 스케일 문제를 정면으로 해결했습니다.4 이는 단일 연구 그룹에 의해 주도된, 집중적이고 반복적인 개선 과정을 명확히 보여줍니다.

또한, 이 시기는 '넥'이라는 개념이 YOLO 계보에서 중요한 구성 요소로 자리 잡는 전환점이었습니다. YOLOv1과 v2는 단순한 '백본-헤드' 구조였지만, YOLOv3는 정교한 FPN을 도입하여 이후 모든 YOLO 모델의 표준이 될 현대적인 3부분(백본-넥-헤드) 탐지기 아키텍처를 확립했습니다.21

YOLOv3는 원저자인 조셉 레드몬(Joseph Redmon)이 개발한 마지막 버전이기도 합니다.5 당시 논문만 보고 이 초기 모델들을 재현하는 것은 매우 어려웠는데, 많은 핵심 세부 사항이 주석 없는 C 기반의 Darknet 소스 코드에만 존재했기 때문입니다.23 이러한 '구현 격차'는 이후 더 사용자 친화적인 프레임워크가 등장할 수 있는 기회를 만들었습니다.



- **개발 및 철학**: 조셉 레드몬이 연구를 중단한 후 알렉세이 보흐코프스키(Alexey Bochkovskiy) 팀이 개발한 첫 번째 주요 버전입니다.5 그 철학은 단일 GPU에서 훈련할 수 있는 최적의 탐지기를 만들기 위해 기존의 수많은 기법들을 체계적으로 테스트하고 결합하는 것이었습니다.24
- **아키텍처**:
  - **백본**: **CSPDarknet53** - Darknet-53에 교차 단계 부분 연결(CSP, Cross-Stage Partial)을 통합하여 기울기 흐름을 개선하고 계산량을 줄였습니다.4
  - **넥**: 수용 영역(receptive field)을 늘리기 위해 백본 뒤에 **SPP(Spatial Pyramid Pooling)** 블록을 추가하고, v3의 FPN보다 더 효과적인 특징 융합을 위해 **PANet(Path Aggregation Network)**을 사용했습니다.21
  - **헤드(Head)**: YOLOv3와 동일한 앵커 기반 헤드를 사용했습니다.21
- **핵심 혁신 ("Bag of Freebies"와 "Bag of Specials")**:
  - **Bag of Freebies (BoF)**: 추론 비용 증가 없이 훈련 중에 정확도를 향상시키는 기법들입니다. 대표적인 예는 **모자이크 데이터 증강(Mosaic data augmentation)**으로, 4개의 훈련 이미지를 하나로 결합하여 문맥에서 벗어난 객체 및 작은 객체 탐지에 대한 강건성을 높입니다.24 그 외에 자기-적대적 훈련(Self-Adversarial Training, SAT) 등이 있습니다.27
  - **Bag of Specials (BoS)**: 추론 비용을 약간 증가시키지만 정확도를 크게 향상시키는 모델 구성 요소입니다. 대표적인 예는 **Mish 활성화 함수**와 **CIoU(Complete-IoU) 손실 함수**로, 이는 표준 IoU 손실 함수에 중심점 거리와 종횡비를 추가로 고려하여 개선한 것입니다.21


- **개발 및 철학**: YOLOv4 직후 Ultralytics의 글렌 조처(Glenn Jocher)가 개발했습니다.5 초기에는 논문 없이 공개되어 커뮤니티에서 논쟁이 있었지만, 주된 초점은 **PyTorch 프레임워크** 내에서의 사용성, 속도, 그리고 배포 용이성이었습니다.22
- **아키텍처**: YOLOv4와 매우 유사하지만 PyTorch 네이티브로 구현되어 접근성이 훨씬 높았습니다.
  - **백본**: CSPDarknet53.30
  - **넥**: SPPF(SPP의 최적화 버전) 및 PANet.32
  - **헤드**: YOLOv3 스타일의 헤드.32
- **핵심 혁신**:
  - **프레임워크와 사용성**: 가장 큰 혁신이었습니다. PyTorch 네이티브라는 점은 Darknet C 프레임워크에서의 복잡한 변환 과정을 없앴습니다. 사용자 친화적인 저장소, 쉬운 훈련 스크립트, 그리고 ONNX, CoreML 등 다양한 형식으로의 모델 내보내기 기능을 내장하고 있었습니다.22
  - **모델 스케일링**: YOLOv5n(나노), s(스몰), m(미디엄), l(라지), x(엑스트라-라지)와 같은 변형 모델들을 통해 모델의 크기와 복잡성을 명확하고 간단하게 조절할 수 있는 방법을 도입했습니다. 이를 통해 사용자는 자신의 특정 속도/정확도 요구사항에 맞는 모델을 쉽게 선택할 수 있었습니다.22
  - **하이퍼파라미터 진화 및 AutoAnchor**: 사용자 정의 데이터셋에 대한 훈련 하이퍼파라미터와 앵커 박스를 자동으로 최적화하는 도구를 통합했습니다.15

YOLOv4와 v5는 YOLO 발전 경로에서 근본적인 분기점을 나타냅니다. v4는 알려진 모든 최적화 기법을 결합하여 새로운 최고 성능을 달성하려는 연구 중심의 노력이었습니다. 반면 v5는 강력한 아키텍처를 더 넓은 커뮤니티가 실용적이고 유지보수 가능하며 쉽게 사용할 수 있도록 만드는 엔지니어링 중심의 노력이었습니다. v4 논문이 수많은 기술 용어와 테스트된 기능들로 가득 차 있는 반면 24, v5는 논문 없이 공개되었고 그 문서들은 사용 편의성과 빠른 시작 가이드를 강조합니다.22 이는 "우리가 만들 수 있는 절대적으로 최고의 것은 무엇인가?"라는 질문에 답하려 한 v4와 "훌륭한 모델을 모든 사람이 실제로 사용할 수 있게 하려면 어떻게 해야 하는가?"라는 질문에 답하려 한 v5의 서로 다른 목표를 보여줍니다.

이 시점에서 아키텍처 자체는 상품화되고, 훈련 방식과 사용성이 차별화 요소로 부상했습니다. v4와 v5의 아키텍처 구성 요소(CSPDarknet, PANet, YOLOv3 헤드)는 놀라울 정도로 유사합니다. 진짜 차이는 프레임워크(Darknet 대 PyTorch), 훈련 파이프라인(모자이크, AutoAnchor), 그리고 개발자 경험과 같은 '메타' 측면에 있었습니다. 이는 핵심 설계가 안정화되면서, 이제 그것을 어떻게 가장 잘 훈련하고 배포할 것인가로 초점이 이동했음을 시사합니다. 이러한 변화는 향후 버전들이 단순히 원시적인 $mAP$ 수치뿐만 아니라 개발자 경험과 생태계 통합을 통해 경쟁할 수 있는 길을 열었습니다.



- **개발 및 철학**: Megvii Technology에서 개발했습니다.1 핵심 아이디어는 다른 탐지기 계열에서 성공을 거둔 현대적인 앵커-프리(anchor-free) 설계와 고급 레이블 할당 전략을 YOLO 아키텍처에 적용하여 성능을 향상시킬 수 있음을 증명하는 것이었습니다.
- **핵심 혁신**:
  - **앵커-프리 설계**: 가장 중요한 변화로, 사전 정의된 앵커 박스의 필요성을 제거하여 설계 파라미터와 하이퍼파라미터의 수를 줄였습니다. 모델은 객체의 중심과 높이, 너비를 직접 예측합니다.1
  - **분리된 헤드(Decoupled Head)**: 헤드에서 분류와 회귀 작업을 별도의 브랜치로 분리했습니다. 이전 YOLO 헤드들은 두 작업을 위해 동일한 특징을 사용했습니다. 이러한 분리는 성능을 향상시키는 것으로 나타났습니다.29
  - **고급 레이블 할당 (SimOTA)**: 고정된 IoU 임계값 대신 예측 품질에 따라 각 실제 객체에 대해 가변적인 수의 긍정 샘플을 선택하는 동적 레이블 할당 전략입니다. 이는 모호하거나 중첩된 객체에 대한 훈련을 개선합니다.29


- **개발 및 철학**: Meituan에서 개발했습니다.13 초점은 하드웨어 친화적 설계와 양자화에 용이한 아키텍처에 중점을 둔, 산업용 사례에 매우 효율적이고 배포 가능한 탐지기를 만드는 것이었습니다.30
- **핵심 혁신**:
  - **하드웨어 친화적 설계 및 재매개변수화(Re-parameterization)**: 백본(**EfficientRep**)과 넥(**Rep-PAN**)은 재매개변수화 기법을 사용하여 설계되었습니다. 이는 모델이 훈련 중에는 복잡한 구조를 가지지만, 추론 시에는 간단하고 매우 빠른 아키텍처로 융합될 수 있음을 의미합니다.5
  - **양방향 연결 (BiC) 모듈**: 속도 저하를 최소화하면서 위치 파악 신호를 향상시키는 개선된 넥 모듈입니다.38
  - **분리된 헤드**: YOLOX와 마찬가지로 더 나은 정확도를 위해 분리된 헤드를 사용합니다.5
  - **앵커 보조 훈련 (AAT)**: 훈련 중에는 앵커 기반 방법을 사용하여 모델을 돕고, 추론 시에는 효율적인 앵커-프리 방식으로 전환하는 하이브리드 전략입니다.38


- **개발 및 철학**: YOLOv4와 동일한 팀(Wang, Bochkovskiy, Liao)이 개발했습니다.35 이 버전은 아키텍처와 훈련 과정 자체를 최적화하여 속도와 정확도에서 새로운 최고 수준을 달성하는 것을 목표로 했으며, "훈련 가능한 무료 기법 모음(Trainable Bag-of-Freebies)"이라는 개념을 도입했습니다.
- **핵심 혁신**:
  - **확장된 ELAN (E-ELAN)**: 백본에 사용된 ELAN(Efficient Layer Aggregation Network) 계산 블록의 진화된 버전입니다. 기울기 경로를 제어하여 네트워크의 학습 능력을 향상시키고, 더 깊은 모델이 효과적으로 학습할 수 있도록 합니다.40
  - **연결 기반 모델을 위한 모델 스케일링**: 특징 연결에 의존하는 아키텍처에 더 효과적인, 네트워크의 깊이와 너비를 조화롭게 확장하는 복합 스케일링 방법을 도입했습니다.40
  - **Trainable Bag-of-Freebies**:
    - **계획된 재매개변수화 합성곱**: 성능 저하 없이 네트워크 내 모듈에 재매개변수화를 적용하는 전략입니다.40
    - **보조 헤드와 Coarse-to-Fine 손실**: 네트워크 중간에 추가적인 '보조 헤드'를 사용하여 얕은 레이어의 훈련을 돕고, 별도의 손실 계산을 수행합니다. 반면 최종적이고 세밀한 예측은 주 '리드 헤드'가 담당합니다. 이는 깊은 모델의 훈련을 개선합니다.40

이 다각화 시대는 단일 YOLO 계보가 여러 개의 병렬적인 진화 분기로 나뉘는 것을 의미합니다. 각 분기는 뚜렷한 목표를 가진 다른 기업(Megvii, Meituan, YOLOv4 팀)에 의해 주도되었습니다. 이러한 경쟁은 각 팀이 서로의 아이디어를 차용하고 개선하면서 혁신을 가속화했습니다.

YOLOX가 성공적으로 앵커-프리 설계를 구현한 것은 분수령이었습니다. 이는 v2 이후 핵심 기능이었던 앵커 박스의 복잡성이 반드시 필요한 것은 아니며, 앵커-프리 방식이 더 빠르고 간단할 수 있음을 보여주었습니다. 이 흐름은 명확합니다. YOLOX가 YOLO 계열 내에서 앵커-프리 개념을 검증했고, 이는 빠르게 새로운 표준이 되었습니다.4

또한, YOLOv6와 YOLOv7은 모두 재매개변수화를 주요 특징으로 합니다. 이는 훈련 시에는 복잡한 구조를, 추론 시에는 다른 구조를 갖는 기법입니다. 이는 속도-정확도 트레이드오프를 관리하는 정교한 새로운 방식으로, 추론 시간의 지연 없이 복잡한 훈련 아키텍처의 이점을 누릴 수 있게 합니다. 이는 단순히 고정된 아키텍처를 확장하거나 축소하는 것보다 더 진보된 최적화 전략입니다.



- **철학**: Ultralytics(YOLOv5 개발사)가 새로운 통합 프레임워크로 개발했습니다. 이전 버전들의 성공(특히 v5의 사용성과 v7의 성능)을 기반으로 하며, 앵커-프리 설계와 같은 현대적인 모범 사례를 통합했습니다. 단순한 객체 탐지기를 넘어 완전한 비전 AI 프레임워크로 자리매김하고 있습니다.15
- **핵심 혁신**:
  - **앵커-프리 및 분리된 헤드**: YOLOX에서 대중화된 앵커-프리 설계와 분리된 헤드를 채택하여, 앵커로부터의 오프셋 대신 객체 중심을 직접 예측합니다.29
  - **새로운 백본 모듈 (C2f)**: YOLOv5의 C3 모듈을 새로운 C2f(Cross-Stage Partial, 2 fusions) 모듈로 대체하여 더 풍부한 기울기 흐름과 더 나은 성능을 제공합니다.42
  - **작업 확장**: 동일한 모델 아키텍처에 다른 헤드를 사용하여 **인스턴스 분할(instance segmentation), 이미지 분류, 포즈 추정(pose estimation)** 등 탐지를 넘어선 여러 비전 작업을 기본적으로 지원합니다.15
  - **손실 함수**: 경계 상자 회귀를 분포로 모델링하는 **분포 초점 손실(DFL, Distribution Focal Loss)**을 사용하여 모호한 경우의 위치 파악 정확도를 향상시킵니다.


- **철학**: 깊은 네트워크의 근본적인 문제인 '정보 병목 현상(information bottleneck)'을 해결합니다. 이는 데이터가 레이어를 통과하면서 정보가 손실되어 훈련에 사용되는 기울기가 저하되는 현상입니다.45
- **핵심 혁신**:
  - **프로그래밍 가능한 기울기 정보 (PGI)**: 대상 작업에 대한 완전한 입력 정보를 제공하여 신뢰할 수 있는 기울기를 생성하는 새로운 개념입니다. 보조 가역 브랜치(auxiliary reversible branch)를 사용하여 정보 손실을 방지하고 주 브랜치가 저하되지 않은 정확한 정보로부터 학습할 수 있도록 보장합니다.16
  - **일반화된 효율적 레이어 집계 네트워크 (GELAN)**: CSPNet과 ELAN의 원리를 결합한 새롭고 매우 효율적인 네트워크 아키텍처입니다. 우수한 파라미터 활용도를 가지며 경량, 고속, 고정확도 모델을 위해 설계되었습니다.45


- **철학**: 중복된 경계 상자를 제거하는 후처리 단계인 비-최대 억제(NMS, Non-Maximum Suppression)의 필요성을 제거하여 진정한 종단간(end-to-end) 탐지기를 만드는 것을 목표로 합니다. NMS는 상당한 지연 시간 병목이 될 수 있습니다.15
- **핵심 혁신**:
  - **NMS-Free 훈련**: 훈련 중에 **이중 레이블 할당(dual label assignment)** 전략과 **일관된 매칭 메트릭(consistent matching metric)**을 도입하여, 하나의 예측 헤드는 높은 재현율(recall)에, 다른 하나는 높은 정밀도(precision)에 최적화되도록 합니다. 이는 전통적으로 NMS가 해결하던 모호성을 제거합니다.
  - **경량 아키텍처**: 경량 분류 헤드와 공간-채널 분리 다운샘플링을 갖춘 단순하고 효율적인 모델 아키텍처를 특징으로 하여 계산 오버헤드를 줄입니다.
  - **성능**: YOLOv9과 같은 이전 버전에 비해 지연 시간과 파라미터를 크게 줄이면서도 최고 수준의 성능을 달성합니다.1

이 현대 시대는 아키텍처적 개선에서 더 나아가 딥러닝의 근본적인 문제를 해결하는 방향으로 전환되었음을 보여줍니다. 이전 버전들이 아키텍처 블록을 추가하거나 교체하는 데 중점을 두었다면, v9와 v10은 정보 손실(v9) 및 후처리 병목(v10)과 같은 더 깊은 문제를 다룹니다. 이는 커뮤니티가 탐지기 아키텍처 최적화를 넘어 전체 훈련 및 추론 파이프라인을 근본 원리부터 재검토하고 있음을 나타냅니다.

또한 YOLOv8은 YOLO를 단일 작업 모델이 아닌 포괄적인 다중 작업 컴퓨터 비전 프레임워크로 자리매김하려는 Ultralytics의 전략적 움직임을 보여줍니다.15 이는 모델의 적용 가능성을 엄청나게 넓히고 사용자가 모든 비전 요구에 대해 Ultralytics 생태계 내에 머물도록 장려합니다.

마지막으로, YOLOv10의 NMS 제거에 대한 집중은 진정으로 종단간이며 지연 시간이 최적화된 모델을 향한 중요한 단계입니다. NMS는 종종 숨겨진 계산 비용이었으며, 이를 제거하면 특히 엣지 장치에서 배포 파이프라인이 단순화되고 지연 시간이 더 예측 가능해집니다.15 신경망 구성 요소가 고도로 최적화됨에 따라, 다음 주요 성능 향상은 이러한 외부 후처리 단계를 모델의 훈련 및 아키텍처에 직접 통합하는 데서 비롯될 것임을 시사합니다.


이 섹션에서는 YOLO의 핵심 구성 요소들의 진화 경로를 종합하여, 주요 트렌드와 중추적인 순간들을 강조하는 교차 분석을 제공합니다.


- **Darknet 시대 (v1-v3)**: VGG/GoogLeNet에서 영감을 받은 간단한 순차적 네트워크(Darknet-19, Darknet-53)로, 깊이를 처리하기 위해 잔차 연결을 도입했습니다.4
- **CSP 시대 (v4-v5, v8)**: 기울기 흐름과 효율성을 개선하기 위해 교차 단계 부분 연결(CSP)을 도입했습니다 (CSPDarknet53, C2f).4
- **집계 네트워크 시대 (v7, v9)**: ELAN, E-ELAN, GELAN과 같은 더 복잡한 집계 블록을 사용하여, 네트워크가 기울기 경로를 손상시키지 않으면서 더 강건한 특징을 학습할 수 있도록 설계되었습니다.40


- **넥 없음 (v1)**: 백본에서 예측으로 바로 이어지는 직접적인 경로를 가졌습니다.9
- **원시적인 넥 (v2)**: 한 단계의 특징 융합을 위한 간단한 `passthrough` 레이어를 사용했습니다.16
- **FPN (v3)**: 다중 스케일 특징 융합을 위해 하향식 특징 피라미드 네트워크(FPN)를 도입했습니다.4
- **PANet 및 그 이상 (v4+)**: 더 나은 정보 흐름을 위해 FPN에 상향식 경로를 추가한 PANet을 채택했습니다.24 이후 YOLOv6과 같은 버전에서는 BiC 모듈과 같은 추가적인 개선이 이루어졌습니다.39


- **결합된, 앵커 기반 (v1-v4)**: 헤드는 단순하고 결합되어 있었으며(분류와 회귀가 동일한 특징에서 나옴), 사전 정의된 앵커 박스에 의존했습니다(v2부터).2
- **분리된, 앵커-프리 (YOLOX, v8)**: 분류와 회귀 브랜치를 분리하고(분리된 헤드), 앵커 없이 객체 중심을 직접 예측하는 주요 전환이 이루어졌습니다. 이는 설계를 단순화하고 성능을 향상시켰습니다.29
- **NMS-프리 (v10)**: NMS 후처리의 필요성을 없애기 위해 훈련 과정과 헤드를 설계한 최신 진화로, 모델을 진정한 종단간 방식으로 만들었습니다.15


- **YOLOv1**: 모든 구성 요소(위치, 신뢰도, 클래스)에 단순한 제곱합 오차(SSE)를 사용했으며, 이는 위치 파악과 분류 오차의 가중치를 올바르게 부여하지 못하는 주요 약점이었습니다.9
- **YOLOv3**: 클래스 및 객체성 점수에 대해 이진 교차 엔트로피(BCE)로 전환했으며, 이는 분류 작업에 더 적합한 선택이었습니다.14
- **YOLOv4 이상**: IoU 기반 손실 함수의 시대입니다. 이 손실 함수들은 교차 결합 비율(IoU) 메트릭을 직접 최적화합니다.
  - **CIoU 손실 (v4)**: 겹침 영역, 중심점 거리, 종횡비를 고려합니다.21
  - **SIoU 손실 (v6)**: 정렬되지 않은 예측에 불이익을 주기 위해 각도 비용을 추가합니다.29
  - **분포 초점 손실 (DFL) (v8)**: 상자 좌표를 분포로 취급하여 모호한 경계에 대한 정확도를 향상시킵니다.33

YOLO의 진화에서 반복되는 주제는 더 넓은 컴퓨터 비전 연구 커뮤니티의 성공적인 아이디어(예: ResNet 연결, FPN, 앵커 박스, 앵커-프리 설계)를 채택한 다음, 효율적인 YOLO 프레임워크 내에서 이를 고도로 최적화하는 것입니다. 이는 YOLO의 강점이 항상 완전히 새로운 개념을 처음부터 발명하는 것이 아니라, 현장의 최고의 아이디어들을 통합하고 정제하는 매우 효과적이고 빠른 플랫폼이라는 점을 보여줍니다.

또한, 세 가지 구성 요소(백본, 넥, 헤드)는 서로 다른 속도로 진화했습니다. 백본은 특징 추출 효율성에 초점을 맞춘 지속적이고 점진적인 개선을 보였습니다. 반면 헤드는 패러다임을 바꾸는 급진적인 변화(앵커 기반 -->> 앵커-프리 -->> NMS-프리)를 겪었습니다. 넥은 이 둘을 연결하기 위해 적응하는 유연한 '접착제' 역할을 했습니다. 이러한 차등적인 진화는 객체 탐지에서 가장 큰 개념적 전투가 예측 메커니즘, 즉 헤드에서 벌어졌음을 강조합니다.



전통적인 YOLO 모델은 '폐쇄형 집합(closed-set)' 또는 '고정 어휘(fixed-vocabulary)' 탐지기입니다. 이들은 훈련된 특정 카테고리(예: COCO 데이터셋의 80개 클래스)만 탐지할 수 있습니다.51 이는 새로운 객체를 탐지하기 위해 재훈련이 필요하므로 실제 세계에서의 유용성을 제한합니다.


개방형 어휘 탐지(open-vocabulary detection)는 모델이 해당 클래스에 대해 명시적으로 훈련받지 않고도 임의의 텍스트 설명이나 프롬프트를 기반으로 객체를 탐지할 수 있는 개념입니다.51 이는 CLIP과 같은 비전-언어 모델(VLM)을 활용하여 달성됩니다.51


- **아키텍처**: YOLO-World는 YOLOv8 백본을 기반으로 하지만, **텍스트 인코더**(CLIP으로 사전 훈련됨)와 새로운 **재매개변수화 가능한 비전-언어 경로 집계 네트워크(RepVL-PAN)**를 통합합니다.51
- **메커니즘**: 모델은 객체의 시각적 특징과 해당 텍스트 임베딩을 연관시키는 법을 학습합니다. 추론 시에는 **"프롬프트 후 탐지(prompt-then-detect)"** 패러다임을 사용합니다. 사용자가 "사람", "파란 차"와 같은 텍스트 프롬프트 세트를 제공하면, 이는 텍스트 인코더에 의해 '오프라인 어휘'로 인코딩됩니다. 그런 다음 YOLO-World 모델은 이미지에서 이러한 텍스트 임베딩과 일치하는 객체를 효율적으로 탐지합니다.52 이는 모든 이미지에 대해 텍스트를 다시 인코딩하는 '온라인' 방법보다 훨씬 효율적입니다.52


YOLO-World는 처음으로 경량의 실시간 탐지기에 개방형 어휘 기능을 도입하여, 이 고급 기술을 실제 애플리케이션에 훨씬 더 접근 가능하고 실용적으로 만들었습니다.52 LVIS와 같은 벤치마크에서 강력한 제로샷(zero-shot) 성능을 보여줍니다.53

YOLO-World는 두 가지 주요 AI 트렌드의 융합을 나타냅니다. YOLO 계열로 대표되는 '효율적인 아키텍처' 트렌드와 CLIP, DINO와 같은 모델로 대표되는 '대규모 비전-언어 사전 훈련' 트렌드를 결합합니다. 이 융합은 YOLO를 고정된 카테고리 제약에서 벗어나게 하는 자연스럽고 강력한 다음 단계입니다.

또한, '오프라인 어휘'를 사용하는 "프롬프트 후 탐지" 패러다임은 YOLO의 실시간 성능을 유지하기 위한 중요한 엔지니어링 선택입니다. 모든 이미지 프레임과 함께 텍스트 프롬프트를 처리하는 '온라인' 접근 방식은 텍스트 인코더로 인한 상당한 지연 시간 병목을 유발하여 빠른 YOLO 탐지기 사용의 목적을 무색하게 만들 것입니다. 특정 애플리케이션(예: 보안 카메라, 자율 주행차)의 어휘는 프레임마다 변하지 않는 경우가 많으므로, 이 어휘를 미리 인코딩함으로써 프레임당 비용을 최소화하여 시스템의 실시간 특성을 보존합니다.52


이 섹션에서는 주로 MS COCO 데이터셋에서 다양한 YOLO 모델의 성능 지표를 종합적으로 제시합니다. 분석은 정확도($mAP$), 속도(FPS), 모델 크기(파라미터, FLOPs) 간의 트레이드오프에 초점을 맞출 것입니다.


결과를 맥락에 맞게 이해하기 위해 MS COCO 데이터셋과 표준 평가 지표인 $mAP@0.5$(PASCAL VOC 지표) 및 $mAP@0.5:0.95$(주요 COCO 지표)에 대한 간략한 설명이 제공됩니다.58


공정한 비교를 위해 가능한 경우 NVIDIA V100 GPU를 기준으로 데이터를 정규화한 종합적인 표가 구성될 것입니다. 테스트 조건(예: 다른 GPU, 배치 크기)의 차이점은 명시될 것입니다.

**표 1: MS COCO(test-dev)에서의 종합 YOLO 성능 벤치마크**

| 모델 버전      | 입력 크기 | mAP @.5-.95 (%) | mAP @.5 (%) | FPS (V100, b=1) | 파라미터 (M) | FLOPs (B)     | 핵심 혁신                    |
| -------------- | --------- | --------------- | ----------- | --------------- | ------------ | ------------- | ---------------------------- |
| YOLOv3-608     | 608       | 33.0*           | 57.9*       | ~30*            | 61.5         | 155.8         | FPN, Darknet-53              |
| YOLOv4-608     | 608       | 43.5            | 65.7        | 62-65           | 64.1         | 128.5         | BoF/BoS, CSPNet, PANet       |
| YOLOv5l-640    | 640       | 48.8            | 67.2        | ~100            | 47.0         | 109.1         | PyTorch 네이티브, 사용성     |
| YOLOX-l-640    | 640       | 50.1            | 68.9        | 68.9            | 54.2         | 155.6         | 앵커-프리, 분리된 헤드       |
| YOLOv6l-640    | 640       | 52.8            | -           | 116             | 59.5         | 226.4         | 재매개변수화, BiC 넥         |
| YOLOv7-640     | 640       | 51.4            | 69.7        | 161             | 36.9         | 104.7         | E-ELAN, Trainable BoF        |
| YOLOv7-E6-1280 | 1280      | 56.0            | -           | 56              | 97.2         | 515.2         | 스케일링된 E-ELAN, 보조 헤드 |
| YOLOv8l-640    | 640       | 52.9            | 69.9        | ~180            | 43.7         | 165.2         | 앵커-프리, C2f 모듈          |
| YOLOv9-C-640   | 640       | 53.0            | 70.2        | -               | 25.5         | 102.8         | PGI, GELAN                   |
| YOLOv10-B-640  | 640       | 52.7            | -           | 29.3            | -            | NMS-프리 헤드 |                              |

참고: 데이터는 여러 출처에서 종합되었으며27, 가능한 경우 정규화되었습니다. FPS는 프레임워크 및 하드웨어 사양에 따라 크게 달라질 수 있습니다. 

`*`는 오래되었거나 직접 비교가 어려울 수 있는 값을 나타냅니다.


표의 데이터에 대한 정성적 및 그래픽 분석을 통해, 서로 다른 모델 계열(예: v5, v6, v7, v8)과 스케일(n, s, m, l, x)이 속도와 정확도의 파레토 최적 전선(Pareto frontier)에서 어떻게 다른 위치를 차지하는지 보여줄 것입니다. 예를 들어, YOLOv7은 속도 대비 높은 정확도를 보이는 경우가 많고, YOLOv5는 소비자용 하드웨어에서 뛰어난 성능으로 유명합니다.70 YOLOv6 모델은 하드웨어 친화적 설계 덕분에 매우 높은 FPS를 보여줍니다.39

보고된 FPS 값은 테스트 환경(GPU 유형, 배치 크기, 프레임워크, TensorRT 사용 여부 등)에 매우 민감하기 때문에 "FPS"는 기만적으로 복잡한 지표입니다. 다른 논문이나 저장소의 FPS 수치를 직접 비교하는 것은 오해의 소지가 있을 수 있습니다. 가장 가치 있는 비교는 동일한 하드웨어에서 여러 모델에 대해 단일 출처에 의해 수행된 것들입니다.

또한, 각 세대마다 $mAP$가 계속 상승하고 있지만 개선율은 둔화되고 있습니다. 최신 모델은 종종 이전 모델과 유사한 $mAP$를 달성하면서도 훨씬 적은 파라미터나 더 빠른 속도를 보입니다. 이는 경쟁의 주축이 '가능한 가장 높은 $mAP$'에서 '주어진 계산 예산 내에서 최고의 $mAP$'로 이동했음을 보여줍니다. 예를 들어, YOLOv10-B는 YOLOv9-C와 동일한 성능을 46% 더 적은 지연 시간과 25% 더 적은 파라미터로 달성합니다.47



본 보고서 전반에 걸쳐 확인된 주요 진화 트렌드는 다음과 같이 요약할 수 있습니다.

- **단일 저자에서 글로벌 협업으로**: 레드몬의 기초 연구에서 다양하고 경쟁적인 생태계로의 전환.
- **앵커-프리 패러다임 전환**: 수동으로 조정된 사전 정보에서 벗어나 더 유연하고 직접적인 예측으로의 이동.
- **훈련 파이프라인의 부상**: 데이터 증강(모자이크), 레이블 할당(SimOTA), 훈련 전략(Trainable BoF, PGI)이 성능의 주요 동인으로 점점 더 중요해짐.
- **종단간 효율성 추구**: NMS와 같은 후처리 병목 현상을 제거하는 데 중점을 둠.


분석에 기반한 실행 가능한 권장 사항은 다음과 같습니다.

- **최대 정확도(클라우드/서버)를 원할 경우**: YOLOv9-E 또는 YOLOv7-E6E는 특히 고해상도에서 고급 아키텍처와 훈련 방법 덕분에 최고의 경쟁자입니다.65
- **최신 GPU에서의 균형 잡힌 성능을 원할 경우**: YOLOv8(l/x 모델)은 높은 정확도, 속도, 사용성의 환상적인 균형을 제공하며 다중 작업 프레임워크의 유연성까지 갖추고 있습니다.29
- **실시간 산업/비디오 애플리케이션의 경우**: YOLOv6(m/l 모델)은 하드웨어 친화적 설계와 재매개변수화 덕분에 뛰어난 추론 속도를 자랑하므로 강력한 선택지입니다.5
- **엣지/모바일/CPU 배포의 경우**: '나노' 또는 '타이니' 변형이 핵심입니다. YOLOv8n, YOLOv6n, YOLOv5n은 모두 리소스가 제한된 환경을 위해 설계되었으며, 낮은 지연 시간과 수용 가능한 정확도 사이의 최상의 트레이드오프를 제공합니다.3
- **유연한 제로샷 애플리케이션의 경우**: YOLO-World는 재훈련 없이 임의의 객체 클래스를 탐지할 수 있는 유일한 선택지입니다.53


YOLO 계열과 실시간 객체 탐지가 나아갈 방향에 대한 미래 지향적인 관점은 다음과 같습니다. 트랜스포머 아키텍처와의 더 깊은 통합, 자기 지도 또는 비지도 학습의 발전, 그리고 더 효율적이고 강건한 개방형 어휘 모델을 향한 지속적인 노력이 예상됩니다. 이미 YOLOv11에서 YOLOv12로의 진화는 어텐션 중심 아키텍처에 대한 집중을 암시하고 있습니다.28 이는 YOLO가 계속해서 컴퓨터 비전 분야의 혁신을 주도하며, 효율성과 유연성의 경계를 넓혀갈 것임을 시사합니다.


1. History of YOLO: From YOLOv1 to YOLOv10 - Labelvisor, accessed July 12, 2025, https://www.labelvisor.com/history-of-yolo-from-yolov1-to-yolov10/
2. The Evolution of YOLO: Object Detection Algorithms | Mindy Support Outsourcing, accessed July 12, 2025, https://mindy-support.com/news-post/the-evolution-of-yolo-object-detection-algorithms/
3. YOLO object detection: Evolution and algorithms - SuperAnnotate, accessed July 12, 2025, https://www.superannotate.com/blog/yolo-object-detection
4. YOLO Explained: From v1 to Present - Viso Suite, accessed July 12, 2025, https://viso.ai/computer-vision/yolo-explained/
5. A Guide to the YOLO Family of Computer Vision Models - Data Phoenix, accessed July 12, 2025, https://dataphoenix.info/a-guide-to-the-yolo-family-of-computer-vision-models/
6. Understanding a Real-Time Object Detection Network: You Only Look Once (YOLOv1), accessed July 12, 2025, https://pyimagesearch.com/2022/04/11/understanding-a-real-time-object-detection-network-you-only-look-once-yolov1/
7. History of YOLO 🕶️ (1). Part 1 article of the timeline and... | by Tharun Sivamani - Cubed, accessed July 12, 2025, [https://blog.cubed.run/history-of-yolo-%EF%B8%8F-1-681abf781fce](https://blog.cubed.run/history-of-yolo-️-1-681abf781fce)
8. YOLO-V2 (You Only Look Once), accessed July 12, 2025, https://www.ijarsct.co.in/Paper17515.pdf
9. YOLO 1 through 5: A complete and detailed overview - Kaggle, accessed July 12, 2025, https://www.kaggle.com/code/vikramsandu/yolo-1-through-5-a-complete-and-detailed-overview
10. YOLO Object Detection Explained: A Beginner's Guide | DataCamp, accessed July 12, 2025, https://www.datacamp.com/blog/yolo-object-detection-explained
11. YOLOv1 Explained | Papers With Code, accessed July 12, 2025, https://paperswithcode.com/method/yolov1
12. YOLO: A Deep Dive - - AIknow, accessed July 12, 2025, https://www.aiknow.io/en/yolo-a-deep-dive/
13. Brief Timeline of YOLO models from v1 to v8 | by Sander Ali Khowaja - Medium, accessed July 12, 2025, https://sandar-ali.medium.com/brief-timeline-of-yolo-models-from-v1-to-v8-7d383140afe2
14. Comparison of YOLO1, YOLO2, and YOLO3 - HackMD, accessed July 12, 2025, https://hackmd.io/@imkushwaha/YoloVersionsComparison
15. Home - Ultralytics YOLO Docs, accessed July 12, 2025, https://docs.ultralytics.com/
16. YOLOv1 to YOLOv10: The fastest and most accurate real-time object detection systems - arXiv, accessed July 12, 2025, https://arxiv.org/html/2408.09332v1
17. (PDF) Yolo Versions Architecture: Review - ResearchGate, accessed July 12, 2025, https://www.researchgate.net/publication/376730469_Yolo_Versions_Architecture_Review
18. [1612.08242] YOLO9000: Better, Faster, Stronger - arXiv, accessed July 12, 2025, https://arxiv.org/abs/1612.08242
19. Object Detection Using YOLO V3 - IJNRD, accessed July 12, 2025, https://www.ijnrd.org/papers/IJNRD2302040.pdf
20. YOLOv3: Real-Time Object Detection Algorithm (Guide) - viso.ai, accessed July 12, 2025, https://viso.ai/deep-learning/yolov3-overview/
21. Review of YOLOv4 Architecture. Paper, Original Code, PyTorch Code - Cenk Bircanoglu, accessed July 12, 2025, https://cenk-bircanoglu.medium.com/review-of-yolov4-architecture-f488ec32c1c4
22. What is YOLOv5? A Guide for Beginners. - Roboflow Blog, accessed July 12, 2025, https://blog.roboflow.com/yolov5-improvements-and-evaluation/
23. [D] Reimplementation of YOLOv3 from paper : r/MachineLearning - Reddit, accessed July 12, 2025, https://www.reddit.com/r/MachineLearning/comments/jqhrdw/d_reimplementation_of_yolov3_from_paper/
24. YOLOv4: High-Speed and Precise Object Detection - Ultralytics Docs, accessed July 12, 2025, https://docs.ultralytics.com/models/yolov4/
25. DRIVING-SCENE IMAGE CLASSIFICATION USING DEEP LEARNING NETWORKS: YOLOV4 ALGORITHM - DiVA portal, accessed July 12, 2025, https://www.diva-portal.org/smash/get/diva2:1689752/FULLTEXT01.pdf
26. Working of YOLOv4 Algorithm - Medium, accessed July 12, 2025, https://medium.com/@shraddhapattanshetti161998/working-of-yolov4-algorithm-76a5e75f188b
27. YOLOv4: A Breakthrough in Real-Time Object Detection - arXiv, accessed July 12, 2025, https://arxiv.org/html/2502.04161v1
28. Discover Object Detection Evolution: YOLO to YOLO11 - Ultralytics, accessed July 12, 2025, https://www.ultralytics.com/blog/the-evolution-of-object-detection-and-ultralytics-yolo-models
29. What is YOLO? The Ultimate Guide [2025] - Roboflow Blog, accessed July 12, 2025, https://blog.roboflow.com/guide-to-yolo-models/
30. History Of Yolo Part (2). Part 2 of the Yolo timeline | by Tharun Sivamani - Cubed, accessed July 12, 2025, https://blog.cubed.run/history-of-yolo-part-2-a3d126b37b00
31. A Comprehensive Review of YOLOv5: Advances in Real-Time Object Detection - ijircst.org, accessed July 12, 2025, https://www.ijircst.org/DOC/12-a-comprehensive-review-of-yolov5-advances-in-real-time-object-detection.pdf
32. Ultralytics YOLOv5 Architecture, accessed July 12, 2025, https://docs.ultralytics.com/yolov5/tutorials/architecture_description/
33. Evolution of YOLO Object Detection Model From V5 to V8 [Updated] - Labellerr, accessed July 12, 2025, https://www.labellerr.com/blog/evolution-of-yolo-object-detection-model-from-v5-to-v8/
34. Ultralytics YOLOv5, accessed July 12, 2025, https://docs.ultralytics.com/models/yolov5/
35. Evolution of YOLO: A Timeline of Versions and Advancements in Object Detection, accessed July 12, 2025, https://yolovx.com/evolution-of-yolo-a-timeline-of-versions-and-advancements-in-object-detection/
36. What is YOLOv6? A Deep Insight into the Object Detection Model - arXiv, accessed July 12, 2025, https://arxiv.org/html/2412.13006v1
37. YOLOV6: A SINGLE-STAGE OBJECT DETECTION FRAMEWORK FOR INDUSTRIAL APPLICATIONS - OpenReview, accessed July 12, 2025, https://openreview.net/pdf?id=7c3ZOKGQ6s
38. Model Comparisons: Choose the Best Object Detection Model for Your Project, accessed July 12, 2025, https://docs.ultralytics.com/compare/
39. Meituan YOLOv6 - Ultralytics YOLO Docs, accessed July 12, 2025, https://docs.ultralytics.com/models/yolov6/
40. YOLOv7: A Powerful Object Detection Algorithm- viso.ai, accessed July 12, 2025, https://viso.ai/deep-learning/yolov7-guide/
41. YOLOv7 Object Detection Paper Explanation & Inference - LearnOpenCV, accessed July 12, 2025, https://learnopencv.com/yolov7-object-detection-paper-explanation-and-inference/
42. YOLOv8 Architecture Explained! - Abin Timilsina - Medium, accessed July 12, 2025, https://abintimilsina.medium.com/yolov8-architecture-explained-a5e90a560ce5
43. YOLOv8: State-of-the-Art Computer Vision Model, accessed July 12, 2025, https://yolov8.com/
44. YOLO Model Comparison: YOLOv11 vs Previous - Ultralytics, accessed July 12, 2025, https://www.ultralytics.com/blog/comparing-ultralytics-yolo11-vs-previous-yolo-models
45. YOLOv9: A Leap Forward in Object Detection Technology, accessed July 12, 2025, https://docs.ultralytics.com/models/yolov9/
46. A Comparative Analysis of YOLOv9, YOLOv10, YOLOv11 for Smoke and Fire Detection, accessed July 12, 2025, https://www.mdpi.com/2571-6255/8/1/26
47. Object Detection Models: Comparing YOLOv10, DETR, and Top Models of 2024 - DFRobot, accessed July 12, 2025, https://www.dfrobot.com/blog-13914.html
48. YOLO Algorithm for Object Detection Explained [+Examples] - V7 Labs, accessed July 12, 2025, https://www.v7labs.com/blog/yolo-object-detection
49. Comparative Analysis of YOLO Versions: YOLOv1 to YOLOv10 - Labelvisor, accessed July 12, 2025, https://www.labelvisor.com/comparative-analysis-of-yolo-versions-yolov1-to-yolov10/
50. YOLO Loss Function Part 1: SIoU and Focal Loss - LearnOpenCV, accessed July 12, 2025, https://learnopencv.com/yolo-loss-function-siou-focal-loss/
51. YOLO World Zero-shot Object Detection Model Explained - Encord, accessed July 12, 2025, https://encord.com/blog/yolo-world-object-detection/
52. YOLO-World: Revolutionizing object detection with AI - Ikomia, accessed July 12, 2025, https://www.ikomia.ai/blog/yolo-world-object-detection
53. YOLO-World: Real-Time Open-Vocabulary Object Detection, accessed July 12, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Cheng_YOLO-World_Real-Time_Open-Vocabulary_Object_Detection_CVPR_2024_paper.pdf
54. Step-by-Step Guide to Unlocking Open-Vocabulary Object Detection with YOLO-World, accessed July 12, 2025, https://www.e2enetworks.com/blog/step-by-step-guide-to-unlocking-open-vocabulary-object-detection-with-yolo-world
55. YOLO-World: Real-Time Open-Vocabulary Object Detection - arXiv, accessed July 12, 2025, https://arxiv.org/html/2401.17270v2
56. YOLO-World: Real-Time Open-Vocabulary Object Detection - arXiv, accessed July 12, 2025, https://arxiv.org/html/2401.17270v3
57. YOLO-World Model, accessed July 12, 2025, https://docs.ultralytics.com/models/yolo-world/
58. Machine Learning Datasets - Object Detection - Papers With Code, accessed July 12, 2025, https://paperswithcode.com/datasets?q=coco&v=lst&o=match&task=object-detection&page=1
59. COCO Dataset - Ultralytics YOLO Docs, accessed July 12, 2025, https://docs.ultralytics.com/datasets/detect/coco/
60. COCO (Common Objects in Context) Dataset - Papers With Code, accessed July 12, 2025, https://paperswithcode.com/dataset/coco
61. YOLOv4 - the most accurate real-time neural network on MS COCO dataset. - Aleksey Bochkovskiy, accessed July 12, 2025, https://alexeyab84.medium.com/yolov4-the-most-accurate-real-time-neural-network-on-ms-coco-dataset-73adfd3602fe
62. yolov4/README.md at master - GitHub, accessed July 12, 2025, https://github.com/lrf008/yolov4/blob/master/README.md
63. kaggle yolov5, accessed July 12, 2025, https://www.kaggle.com/datasets/ayuraj/kaggle-yolov5
64. ultralytics/yolov5: v6.0 - YOLOv5n 'Nano' models, Roboflow integration, TensorFlow export, OpenCV DNN support - Zenodo, accessed July 12, 2025, https://zenodo.org/records/5563715
65. YOLOv7: Trainable bag-of-freebies sets new state-of-the-art for real-time object detectors, accessed July 12, 2025, https://paperswithcode.com/paper/yolov7-trainable-bag-of-freebies-sets-new
66. YOLOv7: Trainable Bag-of-Freebies - Ultralytics YOLO Docs, accessed July 12, 2025, https://docs.ultralytics.com/models/yolov7/
67. What is YOLOv8? A Complete Guide - Roboflow Blog, accessed July 12, 2025, https://blog.roboflow.com/what-is-yolov8/
68. WongKinYiu/yolov9: Implementation of paper - YOLOv9: Learning What You Want to Learn Using Programmable Gradient Information - GitHub, accessed July 12, 2025, https://github.com/WongKinYiu/yolov9
69. Real-Time Object Detection on COCO (Common Objects in Context) - Papers With Code, accessed July 12, 2025, [https://paperswithcode.com/sota/real-time-object-detection-on-coco?metric=FPS%20(V100%2C%20b%3D1)](https://paperswithcode.com/sota/real-time-object-detection-on-coco?metric=FPS+(V100,+b%3D1))
70. Performance Comparison of YOLO Object Detection Models – An Intensive Study, accessed July 12, 2025, https://learnopencv.com/performance-comparison-of-yolo-models/
71. Daily Papers - Hugging Face, accessed July 12, 2025, https://huggingface.co/papers?q=YOLOv9-Tiny
72. MS COCO Benchmark (Real-Time Object Detection) - Papers With Code, accessed July 12, 2025, https://paperswithcode.com/sota/real-time-object-detection-on-coco?p=yolov7-trainable-bag-of-freebies-sets-new
73. srebroa/awesome-yolo: :rocket: The list of the most popular YOLO algorithms - GitHub, accessed July 12, 2025, https://github.com/srebroa/awesome-yolo
74. Timeline of YOLO versions from 2015 to 2024, illustrating the development progression from YOLOv1 to YOLOv10. - ResearchGate, accessed July 12, 2025, https://www.researchgate.net/figure/Timeline-of-YOLO-versions-from-2015-to-2024-illustrating-the-development-progression_fig2_381929610
75. Comparative Study of YOLOv9, YOLOv10, YOLOv11, and YOLOv12: Evolution of Real-Time Object Detection Models and the Role of OpenCV | by Amresh Kumar | Medium, accessed July 12, 2025, https://medium.com/@amresh.kumar11/comparative-study-of-yolov9-yolov10-yolov11-and-yolov12-evolution-of-real-time-object-detection-8eea6f379a1b






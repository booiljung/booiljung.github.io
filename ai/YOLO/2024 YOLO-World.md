[YOLO 객체 탐지 모델](./index.md)
# YOLO-World (2024)



전통적인 객체 탐지(Object Detection) 기술은 컴퓨터 비전 분야에서 근본적인 과제로 자리매김하며 자율 주행, 로보틱스, 이미지 이해 등 다양한 응용 분야에서 괄목할 만한 성과를 이룩했다.1 특히 You Only Look Once (YOLO) 계열의 모델들은 실시간 성능과 높은 정확도를 겸비하여 산업계에서 가장 효율적이고 실용적인 도구 중 하나로 인정받아 왔다.3 그러나 이러한 모델들은 본질적인 한계를 내포하고 있었는데, 바로 '고정 어휘(Fixed Vocabulary)'에 의존한다는 점이다.5 예를 들어, 널리 사용되는 COCO 데이터셋으로 학습된 모델은 사전에 정의된 80개의 카테고리 내에서만 객체를 탐지할 수 있다.1 이처럼 학습 데이터에 의해 어휘가 고정되면, 모델은 예측 불가능한 객체가 등장할 수 있는 '개방된 시나리오(Open Scenarios)'에서는 그 유용성이 급격히 저하된다.3 만약 새로운 객체 카테고리를 추가하고자 한다면, 방대한 양의 데이터를 수집하고, 수작업으로 레이블링한 후, 모델 전체를 재학습하거나 미세 조정(Fine-tuning)해야 하는 값비싸고 시간 소모적인 과정을 거쳐야만 했다.7

이러한 고정 어휘 모델의 근본적인 제약을 극복하기 위한 대안으로 '개방 어휘 탐지(Open-Vocabulary Detection, OVD)'라는 새로운 패러다임이 등장했다.2 OVD의 핵심 목표는 사전 학습된 대규모 시각-언어 모델(Vision-Language Model)이 가진 풍부한 의미론적 지식을 활용하여, 학습 과정에서 보지 못했던 새로운 카테고리의 객체까지도 텍스트 설명을 통해 탐지하는 것이다.11 이는 모델이 단순히 픽셀 패턴을 암기하는 것을 넘어, 언어를 통해 시각적 개념을 이해하고 일반화할 수 있음을 의미하며, 객체 탐지 기술의 응용 범위를 획기적으로 확장할 수 있는 가능성을 열었다.


초기 OVD 연구들은 MDETR, GLIP, Grounding DINO와 같은 모델을 통해 텍스트 프롬프트(Text Prompt)를 사용하여 임의의 객체를 탐지하는 것의 가능성을 성공적으로 입증했다.11 이들 모델은 주로 트랜스포머(Transformer) 아키텍처를 기반으로 하여 복잡한 시각-언어 상호작용을 모델링하는 데 뛰어난 성능을 보였다. 그러나 이러한 성능은 막대한 계산 비용을 수반했다.7 추론 과정에서 이미지와 텍스트를 동시에 인코딩해야 했기 때문에 상당한 지연 시간(Latency)이 발생했고, 이는 로보틱스, 자율 주행, 실시간 영상 분석과 같이 즉각적인 반응이 필수적인 응용 분야에 적용하기에는 비현실적이었다.8 정확성은 확보했지만, 실용성은 부족했던 것이다.

이러한 성능과 실용성 사이의 간극을 메우기 위해 2024년 1월 텐센트 AI 랩(Tencent AI Lab)에서 발표하고 CVPR 2024에 채택된 YOLO-World는 OVD 분야의 결정적인 혁신을 가져왔다.3 YOLO-World의 핵심 기여는 가볍고 빠른 CNN(Convolutional Neural Network) 기반의 YOLO 계열 탐지기(구체적으로 YOLOv8 기반)에 강력한 개방 어휘 탐지 능력을 부여하면서도, YOLO 본연의 실시간 처리 속도를 거의 희생하지 않았다는 점이다.1 이는 OVD 기술을 학술적 탐구의 영역에서 벗어나 실제 산업 현장에 적용 가능한 실용적인 기술로 전환시키는 패러다임의 전환을 의미했다.8

YOLO-World의 등장은 단순히 기존 YOLO 모델에 언어 능력을 추가한 것을 넘어선다. 이는 OVD 연구 분야에서 두 갈래로 나뉘어 발전하던 흐름, 즉 높은 정확도를 추구하지만 속도가 느린 트랜스포머 기반의 시각-언어 모델 연구와, 빠른 속도를 자랑하지만 유연성이 부족한 CNN 기반의 객체 탐지기 연구를 성공적으로 융합한 전략적 전환점이라 할 수 있다. OVD의 '효율성 문제'를 정면으로 해결하고자 한 이 접근법은, 미래의 실용적인 AI 시스템이 단일 아키텍처에 의존하기보다는, 특정 제약 조건(예: 실시간 처리, 엣지 디바이스 배포)에 최적화되도록 여러 아키텍처의 장점을 지능적으로 결합하는 하이브리드 형태로 발전할 것임을 시사한다. YOLO-World는 이러한 실용주의적 하이브리드 접근법의 가장 성공적인 사례 중 하나로 평가된다.



YOLO-World의 아키텍처는 시각 정보 처리, 언어 정보 처리, 그리고 두 정보의 융합이라는 세 가지 핵심 요소로 구성된 명료한 삼중 구조를 가진다.10

- **이미지 인코더 (YOLO Detector):** 모델의 시각적 백본으로는 Ultralytics의 YOLOv8 아키텍처가 채택되었다.10 YOLOv8은 이미 수많은 실제 응용 사례를 통해 그 효율성과 성능이 입증된 모델로, CNN의 계산적 효율성을 극대화하여 이미지로부터 다중 스케일(Multi-scale)의 특징 맵(Feature map)을 신속하게 추출하는 역할을 담당한다.5 이 선택은 YOLO-World가 실시간 성능을 달성하는 데 있어 가장 결정적인 요소이다.
- **텍스트 인코더 (Text Encoder):** 언어 프롬프트를 이해하기 위해 OpenAI의 CLIP(Contrastive Language-Image Pre-training) 모델에서 사용된 사전 학습된 트랜스포머 텍스트 인코더를 활용한다.1 이 인코더는 "사람", "줄에 묶인 개"와 같은 자연어 텍스트를 고차원의 벡터 표현, 즉 텍스트 임베딩(Text embedding)으로 변환한다. CLIP의 텍스트 인코더는 시각적 개념과 언어적 의미를 동일한 잠재 공간(Latent space)에 효과적으로 정렬하는 데 탁월한 성능을 보여, 시각-의미론적 연결고리를 형성하는 데 최적화된 선택이다.5
- **융합 네트워크 (Fusion Network):** 시각 정보와 언어 정보를 결합하는 중추적인 역할은 YOLO-World가 새롭게 제안한 '재매개변수화 가능 시각-언어 경로 집계 네트워크(Re-parameterizable Vision-Language Path Aggregation Network, RepVL-PAN)'가 수행한다.10 RepVL-PAN은 이미지 특징과 텍스트 임베딩 간의 다층적이고 상호적인 융합을 수행하여, 모델이 텍스트 프롬프트에 해당하는 객체를 이미지 내에서 정확히 찾아낼 수 있도록 한다.


RepVL-PAN은 YOLO-World 아키텍처의 심장부로, 이미지와 텍스트라는 이종(Heterogeneous)의 데이터를 깊이 있게 상호작용시키기 위해 설계되었다.1 이 네트워크는 기존 YOLO 모델에서 널리 사용되는 경로 집계 네트워크(Path Aggregation Network, PAN) 구조를 기반으로 한다. PAN은 상향식(Bottom-up) 경로와 하향식(Top-down) 경로를 통해 다양한 해상도의 특징을 집계하여 다중 스케일 특징 피라미드를 생성함으로써, 다양한 크기의 객체를 효과적으로 탐지할 수 있도록 돕는다.5 RepVL-PAN은 이 효율적인 구조를 유지하면서, 다음과 같은 두 가지 핵심 메커니즘을 통해 언어 정보를 주입하고 정제한다.


T-CSPLayer는 시각 정보 처리 파이프라인에 언어적 맥락을 주입하는 모듈이다. 이는 YOLOv8의 핵심 구성 요소인 C2f (CSPLayer with 2 convolutions) 블록을 확장하여 텍스트 유도 기능을 추가한 것이다.2 PAN의 각 레벨에서 특징 융합이 일어난 후, T-CSPLayer는 입력된 텍스트 임베딩과 해당 레벨의 이미지 특징 맵 간의 상호작용을 계산한다. 구체적으로, 마지막 Dark bottleneck 블록 뒤에 위치한 'Max-Sigmoid Attention' 블록이 텍스트 임베딩과 이미지 특징 간의 유사도를 기반으로 어텐션 가중치를 계산한다.7 이 가중치는 특징 맵의 각 공간적 위치에 적용되어, 입력된 텍스트와 관련성이 높은 영역의 특징은 강화하고 관련성이 낮은 영역의 특징은 억제하는 효과를 낳는다.2 이를 통해 네트워크는 전체 이미지 중에서 탐지해야 할 객체가 존재할 가능성이 높은 영역에 집중하게 된다. 이 과정은 다음 수식으로 표현될 수 있다.
$$
\mathbf{F}' = \mathbf{F} \otimes \sigma(\mathrm{Max}(\mathbf{F} \cdot \mathbf{W}^T))
$$
여기서 $\mathbf{F}$는 입력 이미지 특징, $\mathbf{W}$는 텍스트 임베딩, $\sigma$는 시그모이드 함수, $\otimes$는 원소별 곱셈을 나타낸다.


텍스트 임베딩이 이미지의 내용과 무관하게 고정된 값으로만 사용될 경우, 모델의 유연성이 저하될 수 있다. 이를 방지하고 텍스트 임베딩에 현재 입력 이미지에 대한 시각적 맥락을 부여하기 위해 이미지 풀링 어텐션 메커니즘이 도입되었다.2 모든 이미지 특징 픽셀에 대해 직접적으로 크로스-어텐션(Cross-attention)을 계산하는 것은 엄청난 계산량을 요구하므로, YOLO-World는 훨씬 효율적인 접근법을 택했다. 다중 스케일 특징 맵의 각 레벨에서 최대 풀링(Max pooling)을 적용하여 3x3과 같은 작은 영역으로 특징을 압축하고, 이를 통해 소수의 '패치 토큰(Patch token)'을 생성한다 (예: 27개).2 이 소수의 압축된 시각적 토큰들을 키(Key)와 값(Value)으로, 기존 텍스트 임베딩을 쿼리(Query)로 사용하여 어텐션을 계산한다. 이 과정을 통해 텍스트 임베딩은 현재 이미지의 주요 시각적 정보를 반영하여 업데이트되며, '이미지 인식(Image-aware)' 임베딩으로 변환된다. 이 메커니즘은 다음 수식으로 개념화할 수 있다.
$$
\mathbf{W}' = \mathrm{Attention}(\mathbf{W}, \mathrm{PatchTokens}(\mathbf{F}_{1}, \mathbf{F}_{2},...))
$$
여기서 $\mathbf{W}'$는 업데이트된 이미지 인식 텍스트 임베딩을 의미한다.

RepVL-PAN의 설계는 '최소 침습적 융합(Minimal Invasive Fusion)'이라는 철학을 반영한다. 즉, 기존 YOLO의 효율적인 넥(Neck) 구조를 무거운 트랜스포머 블록으로 대체하는 대신, 이미 검증된 효율적인 구성 요소(CSPLayer)를 외과적으로 수정하여 언어 정보를 처리할 수 있도록 개조한 것이다. 이는 효율성이 최우선 과제일 때, 기존의 최적화된 아키텍처를 전면 교체하기보다는 점진적으로 개선하고 확장하는 것이 얼마나 효과적인지를 보여주는 강력한 엔지니어링 원칙이라 할 수 있다.

| 모델명       | 파라미터 (재매개변수화) | 사전 학습 데이터       | LVIS AP (Zero-shot) | COCO AP (Zero-shot) | FPS (V100) |
| ------------ | ----------------------- | ---------------------- | ------------------- | ------------------- | ---------- |
| YOLO-World-S | 13M (77M)               | O365, GoldG            | 26.2                | 25.0                | 74.1       |
| YOLO-World-M | 29M (92M)               | O365, GoldG            | 31.0                | 31.6                | 58.1       |
| YOLO-World-L | 48M (110M)              | O365, GoldG            | 35.0                | 36.1                | 52.0       |
| YOLO-World-L | 48M (110M)              | O365, GoldG, CC3M-Lite | 35.4                | -                   | 52.0       |



YOLO-World의 강력한 일반화 성능은 단일 데이터셋이 아닌, 목적과 형태가 다른 여러 대규모 데이터셋을 지능적으로 조합하여 사전 학습을 수행한 데서 기인한다.8 핵심 전략은 서로 다른 형식의 데이터(탐지, 그라운딩, 이미지-텍스트)를 '영역-텍스트 쌍(Region-text pair)'이라는 공통된 표현 방식으로 통합하는 것이다.2

- **탐지 데이터셋 (Detection Datasets, 예: Objects365, COCO):** 이 데이터셋들은 Objects365의 365개, COCO의 80개와 같이 대규모이지만 고정된 클래스 집합에 대해 매우 정확한 경계 상자(Bounding box) 주석을 제공한다.2 이 데이터는 모델이 객체의 위치를 정밀하게 특정하는 '지역화(Localization)' 능력을 학습하고, 탐지기 백본의 견고한 토대를 마련하는 데 결정적인 역할을 한다.21
- **그라운딩 데이터셋 (Grounding Datasets, 예: GoldG):** 그라운딩 데이터는 이미지와 함께 제공되는 텍스트 설명 속 특정 구문(Phrase)이 이미지 내 어떤 객체 영역에 해당하는지를 명시적으로 연결(Ground)한 정보를 포함한다.21 이 데이터는 모델이 세분화된 언어적 개념(예: "웃고 있는 남자", "파란색 자동차")을 특정 시각적 인스턴스와 정확하게 정렬(Align)하는 능력을 학습하는 데 필수적이며, 이는 OVD의 핵심 기능이다.
- **이미지-텍스트 데이터셋 (Image-Text Datasets, 예: CC3M):** CC3M과 같은 데이터셋은 수백만 장의 방대한 이미지-캡션 쌍을 제공하지만, 경계 상자 정보가 전혀 없다.11 이 막대한 양의 데이터를 활용하기 위해 YOLO-World는 '의사 레이블링(Pseudo-labeling)' 기법을 사용한다.2 먼저, GLIP과 같은 기존의 강력한 OVD 모델을 사용하여 캡션에서 추출한 명사구에 해당하는 객체들의 경계 상자를 자동으로 생성한다. 이후, 생성된 (영역, 텍스트) 쌍의 품질을 CLIP을 사용하여 평가하고, 관련성이 낮은 쌍들을 필터링한다. 이 과정을 통해 생성된 'CC3M-Lite' 데이터셋은 비록 잡음(Noise)이 포함되어 있지만, 모델의 의미론적 어휘를 폭발적으로 확장시켜 희귀하거나 추상적인 개념에 대한 이해도를 높이는 데 크게 기여한다.2

이러한 학습 방법론은 현대 파운데이션 모델의 성공 비결이 단순히 뛰어난 아키텍처에만 있는 것이 아니라, 데이터를 중심에 둔 정교한 전략에 있음을 명확히 보여준다. YOLO-World의 성공은 RepVL-PAN만큼이나 이종의 데이터 소스를 '영역-텍스트 쌍'이라는 공통의 화폐로 통합하고, 의사 레이블링을 통해 저비용 데이터를 고부가가치 학습 데이터로 변환한 데이터 중심적 접근법에 힘입은 것이다. 이는 향후 OVD 분야의 발전이 데이터 합성, 자기 지도 학습, 의사 레이블링 기술의 혁신을 통해 더욱 가속화될 것임을 암시한다.


YOLO-World의 학습 과정은 여러 손실 함수를 조합한 복합 손실 함수(Composite loss function)에 의해 제어된다. 이 중 개방 어휘 능력을 학습하는 데 가장 중심적인 역할을 하는 것은 **영역-텍스트 대조 손실(Region-text contrastive loss)**이다.3

이 손실 함수는 시각과 언어가 공유하는 임베딩 공간 내에서 작동한다. 특정 이미지 영역(Region)이 주어졌을 때, 이 영역의 시각적 임베딩과 정답 텍스트 설명의 임베딩 사이의 유사도(예: 코사인 유사도)는 최대화하도록 유도한다. 동시에, 해당 영역과 관련 없는 오답(Negative) 텍스트 설명들의 임베딩과의 유사도는 최소화하도록 강제한다.18 이러한 대조 학습(Contrastive learning) 방식은 모델이 단순히 패턴을 암기하는 것을 넘어, 시각적 객체와 언어적 설명 간의 의미론적으로 일관된 관계를 학습하도록 만든다.

전체 손실 함수 $\mathcal{L}$은 이 대조 손실($\mathcal{L}_{\mathrm{con}}$)과 함께, 경계 상자의 위치 정확도를 위한 표준적인 회귀 손실(예: IoU loss, $\mathcal{L}_{\mathrm{iou}}$), 그리고 학습 안정화를 위한 분포 초점 손실(Distributed focal loss, $\mathcal{L}_{\mathrm{dfl}}$)의 가중합으로 구성된다.24 전체 손실 함수는 다음과 같이 표현된다.
$$
\mathcal{L} = \lambda_1 \mathcal{L}_{\mathrm{con}} + \lambda_2 \mathcal{L}_{\mathrm{iou}} + \lambda_3 \mathcal{L}_{\mathrm{dfl}}
$$
여기서 $\lambda_1, \lambda_2, \lambda_3$는 각 손실 항의 중요도를 조절하는 하이퍼파라미터이다. 이 복합적인 손실 함수를 통해 YOLO-World는 정확한 위치에(Localization), 올바른 의미로(Grounding), 안정적으로(Stabilization) 객체를 탐지하는 방법을 학습하게 된다.



YOLO-World 이전의 OVD 모델들은 대부분 '온라인 어휘(Online vocabulary)' 또는 '실시간(Just-in-time)' 방식으로 동작했다.7 이는 추론을 수행할 때마다 이미지와 텍스트 프롬프트를 동시에 모델에 입력해야 함을 의미하며, 매번 텍스트 인코더가 실행되어야 했다.6 이러한 방식은 어떤 텍스트 프롬프트에도 즉시 대응할 수 있다는 유연성을 제공했지만, 텍스트 인코딩 과정 자체가 상당한 계산 오버헤드를 유발하여 실시간 성능을 저해하는 주요 병목 지점으로 작용했다.7

이러한 비효율성을 해결하기 위해 YOLO-World는 'Prompt-then-Detect'라는 혁신적인 패러다임을 제안했다.1 이 패러다임의 핵심 아이디어는 프롬프트 처리 단계와 객체 탐지 단계를 완전히 분리하는 것이다. 사용자는 먼저 탐지하고자 하는 객체들의 목록, 즉 커스텀 프롬프트(예: ['사람', '자전거', '신호등'])를 정의한다.6 이 프롬프트 목록은 텍스트 인코더를 통해 **단 한 번만** 인코딩되어, 텍스트 임베딩의 집합인 '오프라인 어휘(Offline vocabulary)'를 생성한다.13


생성된 오프라인 어휘 임베딩은 추론 시 단순히 참조되는 데이터로 저장되는 것이 아니다. 대신, 이 임베딩들은 탐지기 헤드(Detection head)의 가중치(Weights)에 직접적으로 '재매개변수화(Re-parameterized)'된다.5 예를 들어, 최종 분류를 수행하는 컨볼루션 레이어나 완전 연결 레이어의 가중치를 이 텍스트 임베딩 값들로 설정하는 방식이다.5

이 과정을 거치면, 범용 OVD 모델이었던 YOLO-World는 사용자가 정의한 특정 어휘에만 전문화된, 가볍고 빠른 맞춤형 탐지기로 변모한다. 추론 단계에서 이 특화된 모델은 오직 이미지만을 입력으로 받으며, 더 이상 텍스트 인코더를 실행하거나 실시간으로 텍스트를 처리할 필요가 없어진다.17 이는 추론 속도를 극적으로 향상시키고, 특히 계산 자원이 제한적인 엣지 디바이스(Edge device)에서의 배포를 매우 용이하게 만든다.7 이 패러다임 덕분에 전체 모델을 재학습하지 않고도 탐지 어휘를 동적으로 조정할 수 있는 유연성까지 확보하게 된다.10

'Prompt-then-Detect' 패러다임은 범용 모델을 필요에 따라 특정 작업에 최적화된 고성능 모델로 즉시 변환하는 새로운 가능성을 제시한다. 이는 프로그래밍 언어의 JIT(Just-In-Time) 컴파일과 유사하다. 사용자의 '프롬프트'가 소스 코드라면, '재매개변수화' 과정은 이 소스 코드를 최적화된 실행 파일(특화된 탐지기)로 컴파일하는 것과 같다. 이는 MLOps 및 모델 배포 관점에서 중요한 의미를 가진다. 수백 개의 서로 다른 작업을 위해 각각의 모델 파일을 학습하고 관리하는 대신, 단 하나의 YOLO-World 기반 모델을 배포하고 작업이 변경될 때마다 가벼운 '어휘 팩'(재매개변수화된 헤드)을 동적으로 생성하고 교체하는 훨씬 민첩하고 효율적인 운영 워크플로우를 구축할 수 있게 된다.



YOLO-World의 개방 어휘 탐지 성능을 객관적으로 평가하기 위한 핵심 벤치마크는 LVIS(Large Vocabulary in the Wild) 데이터셋이다. LVIS는 1,200개 이상의 방대한 카테고리를 포함하고 있어 모델의 대규모 어휘 일반화 능력을 측정하는 데 적합하다.27 평가는 '제로샷(Zero-shot)' 방식으로 수행되는데, 이는 LVIS 데이터셋의 이미지나 레이블을 학습에 전혀 사용하지 않은 상태에서 모델의 성능을 테스트함을 의미한다.

LVIS minival 데이터셋에서 YOLO-World-L 모델은 NVIDIA V100 GPU 환경에서 52.0 FPS(Frames Per Second)라는 실시간에 가까운 속도를 유지하면서 35.4 AP(Average Precision)라는 인상적인 정확도를 달성했다.1 이 결과는 YOLO-World가 훨씬 더 무겁고 느린 기존 모델들과 대등한 수준의 정확도를 보이면서도, 추론 시간은 극히 일부에 불과하다는 것을 증명하는 매우 중요한 성과이다.


YOLO-World의 성능은 기존의 최첨단 OVD 모델들과의 직접적인 비교를 통해 더욱 명확하게 드러난다. 아래 표는 공식 논문에 제시된 벤치마크 결과를 바탕으로 주요 모델들의 성능을 요약한 것이다.1

| Method           | Backbone | Params     | Pre-trained Data     | FPS      | AP       | APr      | APc      | APf      |
| ---------------- | -------- | ---------- | -------------------- | -------- | -------- | -------- | -------- | -------- |
| Grounding DINO-T | Swin-T   | 172M       | O365,GoldG,Cap4M     | 1.5      | 27.4     | 18.1     | 23.3     | 32.7     |
| DetCLIP-T        | Swin-T   | 155M       | O365,GoldG           | 2.3      | 34.4     | 26.9     | 33.9     | 36.3     |
| **YOLO-World-S** | YOLOv8-S | 13M (77M)  | O365,GoldG           | **74.1** | 26.2     | 19.1     | 23.6     | 29.8     |
| **YOLO-World-M** | YOLOv8-M | 29M (92M)  | O365,GoldG           | **58.1** | 31.0     | 23.8     | 29.2     | 33.9     |
| **YOLO-World-L** | YOLOv8-L | 48M (110M) | O365,GoldG,CC3M-Lite | **52.0** | **35.4** | **27.6** | **34.1** | **38.0** |

분석 결과, YOLO-World는 속도와 정확도 측면에서 기존 모델들을 압도하는 성능을 보여준다. 예를 들어, YOLO-World-L 모델은 DetCLIP-T와 유사한 수준의 정확도(35.4 AP vs 34.4 AP)를 달성하면서도 추론 속도는 약 20배 이상 빠르다(52.0 FPS vs 2.3 FPS).1 또한, Grounding DINO-T (27.4 AP @ 1.5 FPS)나 GLIP 계열 모델들과 비교했을 때도 속도와 정확도 모든 면에서 월등한 우위를 점한다.1

특히 주목할 점은 가장 작은 모델인 YOLO-World-S조차 13M이라는 적은 파라미터 수로 74.1 FPS라는 매우 높은 처리 속도와 26.2 AP라는 준수한 정확도를 기록했다는 것이다.1 이는 YOLO-World의 접근법이 자원이 제한된 엣지 컴퓨팅 환경에서도 충분히 확장 가능하며 실용적임을 시사한다.

이러한 결과는 YOLO-World가 단순히 OVD를 더 빠르게 만든 것이 아니라, 성능의 한계선(Performance Frontier) 자체를 재정의했음을 보여준다. 이전까지 실시간 OVD는 먼 미래의 목표처럼 여겨졌으나, YOLO-World는 그것이 이미 달성된 현실임을 증명했다. 이는 기존의 느린 OVD 아키텍처들이 실시간 제약이 있는 모든 응용 분야에서 사실상 경쟁력을 잃게 되었음을 의미한다. 이로 인해 향후 OVD 연구의 초점은 "실시간 구현이 가능한가?"라는 질문에서 "이 속도를 유지하면서 어떻게 정확도와 강건성을 더욱 향상시킬 것인가?"라는 문제로 자연스럽게 이동하게 되었다.



YOLO-World가 제공하는 실시간으로 새로운 객체를 인식하는 능력은 비정형 환경에서 작동하는 자율 시스템에 혁신적인 변화를 가져올 수 있다.8 로봇이나 자율 주행 차량은 학습 데이터에 포함되지 않았던 예상치 못한 객체(예: 도로 위의 특이한 장애물, 희귀한 종류의 공사 장비)를 마주쳤을 때, 텍스트 프롬프트를 통해 이를 즉시 식별하고 대응할 수 있다.16 예를 들어, 드론을 이용한 상공 감시 임무에서 특정 복장을 한 사람을 찾거나, 공장 내 자율 이동 로봇이 새로운 부품 상자를 인식하도록 하는 등의 작업이 별도의 재학습 없이 가능하다.30 이는 시스템의 안전성과 환경 적응력을 크게 향상시킨다.13


YOLO-World의 진정한 가치는 다양한 산업 분야에서 컴퓨터 비전 솔루션 개발의 패러다임을 바꾸는 데 있다. 기존에는 특정 문제를 해결하기 위해 데이터 수집, 레이블링, 모델 학습이라는 길고 값비싼 과정을 거쳐야 했지만, YOLO-World는 이 과정을 '프롬프트 작성'으로 대폭 단축시킨다.26

- **제조업:** '제로샷 품질 관리'가 가능해진다. 생산 라인의 카메라가 "스크래치", "찌그러짐", "어긋난 부품"과 같은 결함을 찾도록 프롬프트를 주는 것만으로, 모든 가능한 불량 유형에 대한 데이터셋을 구축하고 모델을 학습시킬 필요 없이 품질 검사를 자동화할 수 있다.16
- **소매업:** '동적 재고 관리'를 구현할 수 있다. 매장의 카메라에 "X 브랜드 에너지 드링크"의 재고를 파악하거나 "비어있는 선반 공간"을 감지하라는 명령을 내려, 신제품이 입고되거나 매장 배치가 바뀔 때마다 모델을 재학습할 필요 없이 유연하게 대응할 수 있다.12
- **보안 및 감시:** '적응형 위협 탐지' 시스템을 구축할 수 있다. 관제 요원이 실시간으로 "빨간 배낭을 멘 사람"이나 "주인 없는 가방"을 찾도록 시스템에 프롬프트를 입력하면, CCTV가 해당 객체를 즉시 탐지하여 경고를 보낼 수 있다. 이는 감시 시스템을 수동적인 기록 장치에서 능동적이고 유연한 탐지 도구로 변화시킨다.13
- **콘텐츠 관리 및 이미지 검색:** 대규모 이미지 데이터베이스에서 "해변의 노란 파라솔"과 같은 텍스트 쿼리를 기반으로 특정 객체가 포함된 이미지를 신속하게 검색하여, 콘텐츠 기반 검색 및 관리 작업을 효율화할 수 있다.12
- **데이터 자동 레이블링:** 다른 특화된 모델을 학습시키기 위한 데이터셋을 구축할 때, YOLO-World를 사용하여 객체에 대한 초기 경계 상자를 자동으로 생성함으로써 수작업 레이블링에 드는 막대한 시간과 비용을 절감할 수 있다.10

이처럼 YOLO-World는 컴퓨터 비전 문제 해결의 병목을 데이터 엔지니어링에서 프롬프트 엔지니어링으로 전환시킨다. 이는 과거에는 경제적으로 실현 불가능했던 수많은 맞춤형 비전 애플리케이션의 개발을 가능하게 하며, 머신러닝 전문가가 아닌 현장 실무자도 자연어 프롬프트를 통해 필요한 비전 솔루션을 신속하게 프로토타이핑하고 배포할 수 있는 길을 열어준다. 이는 곧 컴퓨터 비전 기술의 민주화를 의미한다.



YOLO-World의 성능은 사용자가 제공하는 텍스트 프롬프트의 품질과 구체성에 크게 의존한다. 프롬프트가 모호하거나 너무 일반적일 경우, 모델은 예상치 못한 결과를 내놓거나 객체 탐지에 실패할 수 있다.

실제로 보고된 탐지 실패 사례 중 하나는 모델이 '자동차(car)'는 잘 탐지하면서도 '자동차 문(car door)'은 안정적으로 탐지하지 못하는 경우이다.16 이는 모델이 객체와 그 구성 요소 간의 '부분-전체(Part-whole)' 관계를 이해하는 데 한계가 있거나, 더 세분화된 개념을 인식하는 능력이 부족함을 시사한다. 이러한 실패는 YOLO-World가 만능 해결책이 아니며, 그 유연성에는 대가가 따른다는 것을 보여준다.

따라서 YOLO-World를 효과적으로 사용하기 위해서는 상당한 수준의 프롬프트 엔지니어링 기술이 요구된다. 예를 들어, 탐지율을 높이기 위해 "빨간색 자동차"처럼 색상이나 크기 정보를 포함한 더 상세한 프롬프트를 사용하거나, 탐지가 어려운 클래스에 대해서는 신뢰도 임계값(Confidence threshold)을 낮추는 조정이 필요하다.16 또한, 특정 객체와 혼동될 수 있는 다른 객체를 '부정 클래스(Negative class)'로 추가하여 오탐지(False positive)를 줄이는 기법도 효과적일 수 있다.16 이는 모델 학습은 필요 없지만, 모델을 효과적으로 제어하기 위한 전문 지식은 여전히 필요함을 의미한다.


YOLO-World는 YOLO와 CLIP이라는 두 강력한 기반 모델 위에 구축되었지만, 이는 동시에 두 모델의 내재적 한계를 그대로 물려받았음을 의미한다.

- **YOLO 프레임워크의 한계:** YOLO-World는 YOLO 아키텍처의 구조적 특성에서 비롯되는 단점을 계승한다. 대표적으로 이미지 전체를 그리드(Grid)로 나누어 처리하는 방식 때문에, 그리드 셀보다 작은 크기의 객체나 다른 객체에 의해 심하게 가려진(Occluded) 객체를 탐지하는 데 어려움을 겪는 경향이 있다.4 후속 YOLO 버전들에서 이러한 문제가 많이 개선되었지만, 2단계 탐지기(Two-stage detector)에 비해서는 여전히 작은 객체 탐지에 취약한 편이다.
- **CLIP 텍스트 인코더의 한계:** 모델의 언어 이해 능력은 CLIP 텍스트 인코더의 제약에 의해 제한된다.
  - **제한된 컨텍스트 길이:** CLIP 텍스트 인코더는 최대 77개의 토큰만 처리할 수 있으며, 실제 유효 길이는 20개 토큰 내외로 훨씬 짧다는 연구 결과가 있다.35 이는 매우 길고 상세한 설명을 프롬프트로 사용하는 것을 불가능하게 만든다.
  - **구성성 및 속성 결합 문제:** CLIP은 텍스트를 "개념의 가방(Bag of concepts)"처럼 처리하는 경향이 있어, 여러 객체와 속성이 결합된 복잡한 문장의 구조적 의미를 정확히 이해하는 데 어려움을 겪는다.35 예를 들어, "빨간 큐브와 파란 구"와 "파란 큐브와 빨간 구"를 잘 구분하지 못할 수 있다.35
  - **시점 민감성 및 편향:** CLIP의 성능은 객체를 바라보는 시점(Viewpoint)에 따라 크게 달라질 수 있으며 37, 이미지에서 더 큰 객체나 캡션에서 먼저 언급된 객체를 선호하는 편향을 보이기도 한다.36


YOLO-World의 능력을 평가할 때 가장 중요한 것은 '개방 어휘'와 '개방 세계'의 개념을 명확히 구분하는 것이다. YOLO-World는 **개방 어휘 탐지(OVD)** 모델이다. 이는 사용자가 **제공한 어휘 목록** 내에서는 어떤 객체든 탐지할 수 있음을 의미한다. 하지만, 사용자가 프롬프트로 제공하지 않은 미지의 객체를 스스로 '새로운 객체'라고 인지할 수는 없다.

반면, **개방 세계 객체 탐지(Open-World Object Detection, OWOD)**는 이보다 훨씬 더 도전적인 과제이다.39 OWOD 모델은 학습된 '알려진(Known)' 클래스를 탐지함과 동시에, 이들 중 어디에도 속하지 않는 객체를 발견했을 때 이를 '알려지지 않은(Unknown)' 객체로 식별하고 분류할 수 있어야 한다.39 이는 진정한 의미의 '새로움(Novelty)'을 탐지하는 능력이다.

YOLO-World는 이러한 능력이 부재하다. 즉, 사용자가 찾으라고 지시한 것은 무엇이든 찾아주는 강력한 제로샷 탐지 도구이지만, 지시하지 않은 미지의 위협이나 예상치 못한 객체를 스스로 인지하는 진정한 의미의 개방 세계 시스템은 아니다.39 최근 YOLO-UniOW와 같은 후속 연구들은 바로 이 간극을 메우기 위해 '알려지지 않은' 객체 탐지 메커니즘을 명시적으로 추가하는 방향으로 발전하고 있다.39

결론적으로 YOLO-World는 강력한 범용성을 지녔지만, 동시에 '취약한(Brittle)' 범용 모델이라 할 수 있다. 그 유연성은 맞춤형으로 학습된 전문 모델의 강건성과 세분화된 성능을 희생한 대가로 얻어진 것이다. 모델의 한계점들은 사소한 결함이 아니라, 그 기반이 되는 YOLO와 CLIP으로부터 물려받은 깊고 구조적인 문제들이다. 따라서 사용자는 이러한 한계를 명확히 인지하고, YOLO-World를 만병통치약이 아닌, 특정 장단점을 가진 하나의 강력한 도구로 이해해야 한다.



YOLO-World는 컴퓨터 비전 분야, 특히 객체 탐지 기술의 역사에서 중요한 이정표를 세웠다. 이 모델의 가장 핵심적인 기여는 CNN 기반 YOLO 탐지기의 실시간 효율성과 시각-언어 모델의 제로샷 유연성이라는, 이전까지 양립하기 어려워 보였던 두 가지 가치를 성공적으로 융합했다는 데 있다.3

혁신적인 RepVL-PAN 아키텍처와 'Prompt-then-Detect' 추론 패러다임을 통해, YOLO-World는 효율적인 OVD 분야에서 새로운 기술 표준(State-of-the-art)을 정립했다. 이로써 이전에는 막대한 계산 자원과 전문 지식이 필요했던 개방 어휘 탐지 기술을 광범위한 실시간 응용 분야에 적용할 수 있게 되었으며, 맞춤형 객체 탐지 솔루션 개발의 진입 장벽을 획기적으로 낮추었다.1 YOLO-World는 OVD를 이론적 가능성에서 실용적 현실로 이끌어낸 촉매제 역할을 했다.


YOLO-World는 종착점이 아니라 새로운 시작점이다. 이 모델의 성공은 실시간 유연 탐지에 대한 산업적 수요를 명확히 증명했으며, 동시에 차세대 연구가 나아가야 할 방향을 명확하게 제시했다. 현재 OVD 분야는 YOLO-World가 드러낸 한계를 극복하고 더 높은 수준의 지능을 구현하기 위해 빠르게 발전하고 있다.

- **아키텍처의 진화:** 더 효율적이고 강력한 융합 메커니즘에 대한 탐구가 계속되고 있다. **Mamba-YOLO-World**의 등장은 트랜스포머나 표준 CNN의 대안으로, 선형 복잡도와 전역적 수용장(Global receptive field)을 약속하는 상태 공간 모델(State Space Model, SSM)을 넥(Neck) 구조에 도입하려는 움직임을 보여준다.41 이는 속도와 성능을 동시에 개선하려는 지속적인 노력을 반영한다.
- **'개방 세계'로의 도약:** OVD에서 OWOD로 나아가는 것은 가장 중요한 다음 단계이다. **YOLO-UniOW**와 같은 모델들은 '알려지지 않은' 객체를 명시적으로 처리하는 메커니즘을 추가하여, 자율 주행과 같이 안전이 최우선인 실제 환경에서의 신뢰성을 확보하고자 한다.39 이는 단순히 지시받은 것을 찾는 수준을 넘어, 예상치 못한 상황을 인지하는 능력으로의 진화를 의미한다.
- **기반 모델의 개선:** 장기적인 발전은 CLIP과 같은 1세대 시각-언어 모델의 한계를 극복하는 데 달려있다. 더 길고 복잡한 문장의 구조적 의미를 이해하며, 사회적 편향이 적고, 다양한 시점 변화에 강건한 차세대 텍스트 및 이미지 인코더의 개발이 필수적이다.35
- **데이터 스케일링과 자기 지도 학습:** 대규모의 정제되지 않은 웹 데이터를 자기 지도 학습 및 의사 레이블링을 통해 학습에 활용하는 데이터 중심적 접근법은 앞으로도 OVD 모델의 성능을 끌어올리는 핵심 동력으로 작용할 것이다.21

YOLO-World의 가장 큰 유산은 이 모델이 해결한 문제보다, 그로 인해 새롭게 정의된 문제들에 있을지도 모른다. '알려지지 않은 객체 문제'를 해결하고, CLIP의 제약에서 벗어나려는 현재의 연구 흐름은 모두 YOLO-World가 열어젖힌 '실시간 OVD'라는 새로운 지평 위에서 이루어지고 있다. 따라서 YOLO-World는 미래 연구를 촉발시킨 중요한 기폭제로서 그 가치를 인정받을 것이다.


1. YOLO-World: Real-Time Open-Vocabulary ... - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2024/papers/Cheng_YOLO-World_Real-Time_Open-Vocabulary_Object_Detection_CVPR_2024_paper.pdf
2. CVPR Poster YOLO-World: Real-Time Open-Vocabulary Object ..., 8월 24, 2025에 액세스, https://cvpr.thecvf.com/virtual/2024/poster/30009
3. [2401.17270] YOLO-World: Real-Time Open-Vocabulary Object Detection - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2401.17270
4. YOLO Algorithm: Real-Time Object Detection from A to Z - Kili Technology, 8월 24, 2025에 액세스, https://kili-technology.com/data-labeling/machine-learning/yolo-algorithm-real-time-object-detection-from-a-to-z
5. YOLO-World: Revolutionizing object detection with AI - Ikomia, 8월 24, 2025에 액세스, https://www.ikomia.ai/blog/yolo-world-object-detection
6. Step-by-Step Guide to Unlocking Open-Vocabulary Object Detection with YOLO-World, 8월 24, 2025에 액세스, https://www.e2enetworks.com/blog/step-by-step-guide-to-unlocking-open-vocabulary-object-detection-with-yolo-world
7. YOLO-World: Real-Time, Zero-Shot Object Detection - Roboflow Blog, 8월 24, 2025에 액세스, https://blog.roboflow.com/what-is-yolo-world/
8. YOLO-World: Real-Time Open-Vocabulary Object Detection - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2401.17270v3
9. YOLO-World: Real-Time Open-Vocabulary Object Detection - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2401.17270v2
10. YOLO-World Complete Breakdown: Zero-shot Object Detection in Real-Time - ezML, 8월 24, 2025에 액세스, https://ezml.io/blog/yolo-world-complete-breakdown
11. YOLO World Zero-shot Object Detection Model Explained - Encord, 8월 24, 2025에 액세스, https://encord.com/blog/yolo-world-object-detection/
12. YOLO World vs YOLOv8: Comparing the Latest of Object Detection - Dev-kit, 8월 24, 2025에 액세스, https://dev-kit.io/blog/machine-learning/yolo-world-vs-yolo-v8
13. YOLO-World: Advancing Object Detection Technology - Dev-kit, 8월 24, 2025에 액세스, https://dev-kit.io/blog/machine-learning/yolo-world
14. YOLO-World Model, 8월 24, 2025에 액세스, https://docs.ultralytics.com/models/yolo-world/
15. AILab-CVC/YOLO-World: [CVPR 2024] Real-Time Open ... - GitHub, 8월 24, 2025에 액세스, https://github.com/AILab-CVC/YOLO-World
16. YOLO-World: Zero-Shot Detection | Ultralytics, 8월 24, 2025에 액세스, https://www.ultralytics.com/blog/getting-hands-on-with-yolo-world
17. YOLO-World: A Demo to Real-Time, Zero-Shot Object Detection | DigitalOcean, 8월 24, 2025에 액세스, https://www.digitalocean.com/community/tutorials/yolo-world-model
18. YOLO-World: Real-Time Open-Vocabulary Object Detection - Unite.AI, 8월 24, 2025에 액세스, https://www.unite.ai/yolo-world-real-time-open-vocabulary-object-detection/
19. YOLO-World Zero-shot Real-Time Open-Vocabulary Object Detection - visionplatform.ai, 8월 24, 2025에 액세스, https://visionplatform.ai/yolo-world/
20. arXiv:2303.02489v2 [cs.CV] 14 Mar 2023, 8월 24, 2025에 액세스, https://www.sysu-hcp.net/userfiles/files/2023/03/15/17a6bf908c5a44da.pdf
21. Grounded Language-Image Pre-training - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2022/papers/Li_Grounded_Language-Image_Pre-Training_CVPR_2022_paper.pdf
22. Objects365: A Large-Scale, High-Quality Dataset for Object Detection - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/339561122_Objects365_A_Large-Scale_High-Quality_Dataset_for_Object_Detection
23. Learning Visual Grounding from Generative Vision and Language Model - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/WACV2025/papers/Wang_Learning_Visual_Grounding_from_Generative_Vision_and_Language_Model_WACV_2025_paper.pdf
24. [CVPR 2024] YOLO-World:Open-vocabulary Object Detection(OVOD) | Dengbuqi's Blog, 8월 24, 2025에 액세스, https://dengbuqi.github.io/2024/05/20/CVPR-2024-YOLO-World-Open-vocabulary-Object-Detection-OVOD/
25. How to Detect Objects with YOLO-World - Roboflow Blog, 8월 24, 2025에 액세스, https://blog.roboflow.com/how-to-detect-objects-with-yolo-world/
26. YOLO-World: Open vocabulary object detection. No more model training - Pipeless, 8월 24, 2025에 액세스, [https://www.pipeless.ai/blog/yolo-world/YOLO-World:%20Open%20vocabulary%20object%20detection.%20No%20more%20model%20training](https://www.pipeless.ai/blog/yolo-world/YOLO-World: Open vocabulary object detection. No more model training)
27. Real-Time Open-Vocabulary Object Detection - CVPR, 8월 24, 2025에 액세스, https://cvpr.thecvf.com/media/cvpr-2024/Slides/30009_sX3nq6Z.pdf
28. YOLO Object Detection Explained: Models, Tools, Use Cases - Lightly, 8월 24, 2025에 액세스, https://www.lightly.ai/blog/yolo
29. The YOLO Algorithm in Autonomous Vehicles - DPV transportation, 8월 24, 2025에 액세스, https://www.dpvtransportation.com/the-yolo-algorithm-autonomous-vehicles/
30. Leveraging YOLO-World and GPT-4V LMMs for Zero-Shot Person Detection and Action Recognition in Drone Imagery - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2404.01571v1
31. UNCOVER: Unknown Class Object Detection for Autonomous Vehicles in Real-time - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2412.03986v1
32. YOLO12 explained: Real-world applications and use cases - Ultralytics, 8월 24, 2025에 액세스, https://www.ultralytics.com/blog/yolo12-explained-real-world-applications-and-use-cases
33. The YOLO Framework: A Comprehensive Review of Evolution ..., 8월 24, 2025에 액세스, https://www.mdpi.com/2073-431X/13/12/336
34. YOLO Algorithm for Object Detection Explained [+Examples] - V7 Labs, 8월 24, 2025에 액세스, https://www.v7labs.com/blog/yolo-object-detection
35. Unlocking the Long-Text Capability of CLIP - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2403.15378v1
36. Analyzing CLIP's Performance Limitations in Multi-Object Scenarios: A Controlled High-Resolution Study - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2502.19828v1
37. Active Open-Vocabulary Recognition: Let Intelligent Moving Mitigate CLIP Limitations, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/384205265_Active_Open-Vocabulary_Recognition_Let_Intelligent_Moving_Mitigate_CLIP_Limitations
38. Active Open-Vocabulary Recognition: Let Intelligent Moving Mitigate CLIP Limitations, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2024/papers/Fan_Active_Open-Vocabulary_Recognition_Let_Intelligent_Moving_Mitigate_CLIP_Limitations_CVPR_2024_paper.pdf
39. YOLO-UniOW: Efficient Universal Open-World Object Detection - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2412.20645v1
40. From Open Vocabulary to Open World: Teaching Vision Language Models to Detect Novel Objects - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2411.18207v2
41. Marrying YOLO-World with Mamba for Open-Vocabulary Detection - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2409.08513
42. Scaling Open-Vocabulary Object Detection - OpenReview, 8월 24, 2025에 액세스, https://openreview.net/forum?id=mQPNcBWjGc


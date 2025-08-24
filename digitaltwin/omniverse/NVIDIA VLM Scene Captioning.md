[NVidia Omniverse](./index.md)
# NVIDIA VLM Scene Captioning



비전 언어 모델(Vision Language Model, VLM)은 컴퓨터 비전(Computer Vision, CV)과 자연어 처리(Natural Language Processing, NLP)의 역량을 결합하여 시각 데이터와 텍스트 데이터 간의 복잡한 관계를 학습하는 멀티모달(multimodal) 인공지능 시스템이다.1 이는 기계가 인간처럼 시각적 정보를 단순히 '인식'하는 것을 넘어, 그 안에 담긴 의미와 맥락을 언어로 '이해하고 표현'하도록 하는 기술적 도약을 의미한다.2 VLM은 이미지나 비디오를 텍스트와 함께 입력받아, 시각적 내용에 대한 질의응답(Visual Question Answering, VQA), 이미지 캡셔닝, 비디오 요약 등 고차원적인 작업을 수행할 수 있다.5

VLM의 등장은 인공지능 분야의 근본적인 패러다임 전환을 시사한다. 과거의 컴퓨터 비전 모델은 이미지 분류, 객체 탐지 등 특정 목적을 위해 설계된 개별적인 도구에 가까웠다.5 이러한 모델들은 사전 정의된 고정된 클래스나 작업 범위 내에서 높은 성능을 보였지만, 새로운 유형의 질문이나 작업에 대해서는 유연하게 대처하기 어려웠다. 마찬가지로, 대규모 언어 모델(Large Language Model, LLM)은 텍스트 데이터에 대한 깊은 이해를 바탕으로 놀라운 언어 생성 및 추론 능력을 보여주었으나, 그들의 '지식'은 시각적 현실 세계와는 단절된 추상적인 기호의 영역에 머물러 있었다.2

VLM은 이 두 세계를 통합함으로써 기존 모델들의 한계를 극복한다. VLM은 방대한 양의 이미지-텍스트 쌍 데이터를 학습하여, LLM에게 '보는 능력'을 부여한다.5 그 결과, VLM은 고정된 작업 분류에 얽매이지 않고, 자연어 지시(instruction)를 통해 거의 모든 시각 관련 작업을 제로샷(zero-shot) 또는 퓨샷(few-shot) 방식으로 수행할 수 있는 범용적 시각 추론기(general visual reasoner)로 기능하게 된다.5 즉, '이미지 캡셔닝 모델'을 만드는 것이 아니라, '이미지 캡셔닝을 수행할 수 있는 범용 모델'을 만드는 것으로 문제의 정의가 바뀐 것이다. 이러한 전환은 강력한 사전 학습 LLM의 내재적 능력, 즉 제로샷 일반화와 명령어 추종 능력이 시각 도메인으로 성공적으로 전이된 결과라 할 수 있다.

이러한 VLM의 핵심 아키텍처는 일반적으로 'Vision Encoder - Projector - LLM'의 3단계 구조로 설명할 수 있다.5

- **Vision Encoder (비전 인코더):** 이 구성 요소는 입력된 이미지나 비디오에서 색상, 형태, 질감과 같은 핵심적인 시각적 특징을 추출하여 고차원의 벡터 임베딩(vector embedding)으로 변환하는 역할을 한다.1 초기 VLM에서는 합성곱 신경망(CNN)이 주로 사용되었으나, 최근에는 CLIP(Contrastive Language-Image Pre-training) 모델에 기반한 트랜스포머(Transformer) 아키텍처가 표준으로 자리 잡았다.5 특히, 이미지를 여러 개의 패치(patch)로 나누어 순차 데이터처럼 처리하는 비전 트랜스포머(ViT)는 이미지 내의 전역적인 문맥을 효과적으로 포착하는 데 강점을 보인다.1
- **Projector (프로젝터):** 프로젝터는 비전 인코더가 생성한 시각적 임베딩과 LLM이 사용하는 텍스트 임베딩 간의 '언어'를 통일시켜주는 번역기 또는 다리 역할을 수행한다.5 이 모듈은 시각적 특징 벡터를 LLM이 자신의 입력 시퀀스의 일부로 받아들일 수 있는 '이미지 토큰(image token)' 형태로 변환한다. 프로젝터의 구조는 단순한 선형 계층(linear layer)부터 여러 개의 트랜스포머 블록으로 구성된 복잡한 교차 어텐션(cross-attention) 계층에 이르기까지 다양하며, 이 설계는 모델의 성능과 학습 효율성에 중요한 영향을 미친다.5
- **Large Language Model (LLM):** VLM의 두뇌에 해당하는 부분으로, 텍스트 토큰과 프로젝터를 통해 변환된 이미지 토큰을 함께 입력받아 전체적인 문맥을 이해하고, 추론하며, 최종적인 텍스트 응답을 생성한다.5 LLM의 강력한 사전 학습 능력 덕분에 VLM은 복잡한 질문에 답하거나, 이미지의 내용을 요약하고, 심지어 이미지 속 상황에 대해 논리적으로 추론하는 등 고차원적인 과제를 수행할 수 있다.


장면 캡셔닝(Scene Captioning)은 주어진 이미지나 비디오에 포함된 핵심적인 내용, 즉 객체, 그들의 속성, 행동, 그리고 상호 관계를 종합적으로 이해하여 이를 문법적으로 올바르고 의미론적으로 정확한 자연어 문장으로 자동 생성하는 과제이다.10 이는 단순히 이미지 안에 '무엇이 있는지'를 나열하는 객체 탐지를 넘어, '무슨 일이 일어나고 있는지'를 서술하는 고차원적인 '장면 이해(Scene Understanding)' 기술의 정수라 할 수 있다.4 이 기술은 시각 장애인을 위한 접근성 향상, 이미지 및 비디오 검색 시스템의 효율화, 자율 주행 시스템의 환경 인식 능력 강화 등 광범위한 응용 분야에서 핵심적인 역할을 한다.10

장면 캡셔닝 기술은 인코더-디코더(Encoder-Decoder) 프레임워크를 중심으로 점진적으로 발전해왔다.

- **초기 모델 (CNN-RNN 아키텍처):** 딥러닝 기반 이미지 캡셔닝의 초기 모델들은 대부분 CNN과 RNN(Recurrent Neural Network)의 조합으로 구성되었다.11 이 구조에서 ImageNet과 같은 대규모 데이터셋으로 사전 학습된 CNN은 이미지의 공간적 특징을 추출하는 인코더 역할을 수행했다. 추출된 고정된 크기의 특징 벡터는 문장을 순차적으로 생성하는 데 강점을 가진 RNN이나 LSTM(Long Short-Term Memory) 기반의 디코더에 입력되어, 조건부 확률에 따라 다음 단어를 예측하며 캡션을 생성했다.14
- **어텐션 메커니즘(Attention Mechanism)의 도입:** CNN-RNN 구조는 이미지 전체를 단 하나의 특징 벡터로 요약하기 때문에, 캡션의 특정 단어를 생성할 때 이미지의 어느 부분에 집중해야 하는지에 대한 정보가 부족했다.11 이러한 한계를 극복하기 위해 인간의 시각 인지 과정에서 영감을 받은 어텐션 메커니즘이 도입되었다.11 어텐션 메커니즘은 디코더가 캡션의 각 단어를 생성하는 시점마다, 이미지의 여러 영역에 각기 다른 가중치(attention weight)를 부여하여 가장 관련성이 높은 영역의 정보에 '집중(attend)'하도록 한다. 이를 통해 "한 남자가 프리스비를 던지고 있다"라는 캡션을 생성할 때, '프리스비'라는 단어를 생성하는 시점에는 이미지 내 프리스비 영역에, '남자'라는 단어를 생성할 때는 남자 영역에 더 높은 가중치를 부여하여 문맥적 정확도를 획기적으로 향상시켰다.16
- **Transformer 기반 모델의 등장:** 어텐션 메커니즘의 개념을 극대화한 트랜스포머 아키텍처의 등장은 장면 캡셔닝 분야에도 큰 변화를 가져왔다.14 트랜스포머는 RNN의 순차적 처리 방식에서 벗어나 셀프 어텐션(self-attention)을 통해 입력 시퀀스 내 모든 요소 간의 관계를 병렬적으로 계산함으로써, 이미지와 텍스트 내의 장거리 의존성(long-range dependency)을 효과적으로 포착할 수 있게 되었다.15 이는 VLM 아키텍처의 근간을 이루는 핵심 기술이 되었다.

특히 비디오 캡셔닝은 정지 이미지 캡셔닝에 비해 한층 더 높은 복잡성을 가진다. 비디오는 시간 축을 따라 변화하는 동적인 정보를 포함하므로, 모델은 공간적 특징뿐만 아니라 시간적 동적 관계(temporal dynamics)를 이해하고 모델링해야 한다.18 초기 비디오 캡셔닝 연구는 비디오를 여러 프레임으로 나누고 각 프레임에 이미지 캡셔닝 모델을 적용한 후 결과를 종합하는 방식을 사용했으나 13, 이는 시간적 연속성과 인과관계를 포착하는 데 한계가 있었다. 최근 연구들은 3D CNN이나 트랜스포머 기반의 시공간 어텐션을 활용하여 비디오 전체의 동적인 문맥을 직접 모델링하는 방향으로 발전하고 있다. 더 나아가, 비디오 전체를 한 문장으로 요약하는 것을 넘어, 영상 내에서 발생하는 모든 개별 이벤트의 시작과 끝 시간대를 탐지하고 각각에 대한 캡션을 생성하는 '밀집 비디오 캡셔닝(Dense Video Captioning)'이라는 더 세분화되고 도전적인 과제로 확장되고 있다.21


장면 캡셔닝 모델의 성능을 정량적으로 평가하기 위해, 생성된 캡션(candidate)과 사람이 직접 작성한 여러 개의 정답 캡션(references)을 비교하는 다양한 자동 평가 지표가 사용된다.13 이 지표들은 0과 1 사이의 점수로 표현되며, 1에 가까울수록 우수한 성능을 의미한다.13

- **BLEU (Bilingual Evaluation Understudy):** 원래 기계 번역 평가를 위해 개발된 지표로, 생성된 캡션에 포함된 n-gram(연속된 n개의 단어 묶음)이 정답 캡션 집합에 얼마나 많이 나타나는지를 측정한다. 주로 정확도(precision) 기반으로 계산되며, 짧은 문장에 대한 페널티를 부여한다.22
- **ROUGE (Recall-Oriented Understudy for Gisting Evaluation):** 요약문 평가에 주로 사용되며, 정답 캡션에 있는 n-gram이 생성된 캡션에 얼마나 포함되었는지를 측정하는 재현율(recall) 기반 지표이다.22
- **METEOR (Metric for Evaluation of Translation with Explicit ORdering):** 정밀도와 재현율의 조화 평균을 기반으로 하되, 동의어와 단어의 어근(stem)을 고려하여 의미적 유사성을 반영하고, 단어 순서의 일치도까지 평가하여 BLEU와 ROUGE보다 인간의 평가와 더 높은 상관관계를 보이는 경향이 있다.15
- **CIDEr (Consensus-based Image Description Evaluation):** 각 n-gram이 얼마나 중요한 정보인지를 TF-IDF(Term Frequency-Inverse Document Frequency) 가중치를 사용하여 평가한다. 여러 정답 캡션에서 공통적으로 자주 등장하는(합의가 이루어진) n-gram에 높은 가중치를 부여함으로써, 이미지의 핵심 내용을 잘 포착했는지를 측정하는 데 효과적이다.15
- **SPICE (Semantic Propositional Image Caption Evaluation):** n-gram 기반의 표면적 일치도를 넘어, 캡션의 의미 구조를 평가하는 데 초점을 맞춘다. 캡션을 객체, 속성, 관계 등으로 구성된 장면 그래프(scene graph)로 파싱하고, 생성된 캡션의 장면 그래프와 정답 캡션들의 장면 그래프 간의 F1-score를 계산하여 의미론적 정확도를 측정한다.23

이러한 자동 평가 지표들은 모델 개발 과정에서 빠르고 일관된 평가를 가능하게 하지만, 근본적인 한계 또한 명확하다. VLM 기술이 발전함에 따라, 모델이 생성하는 캡션이 사람이 작성한 정답 캡션보다 더 상세하거나, 다른 관점에서 더 창의적인 설명을 제공하는 경우가 빈번해지고 있다.23 그러나 n-gram 일치도에 기반한 전통적인 지표들은 이러한 '정답보다 좋은' 캡션을 낮은 점수로 평가할 수 있다. 즉, 평가 지표가 모델의 발전을 따라가지 못하는 현상이 발생하는 것이다. 이러한 문제를 해결하기 위해, 최근에는 VisCE²와 같이 평가 과정에서 이미지의 시각적 문맥(객체, 속성, 관계 정보)을 명시적으로 LLM에 제공하여, 생성된 캡션이 이미지의 내용을 얼마나 충실하게 반영하는지를 직접 평가하려는 새로운 연구들이 시도되고 있다.23 이는 단순한 문자열 비교를 넘어, 내용의 충실도(faithfulness)와 정확성(accuracy)을 중심으로 평가 패러다임을 전환하려는 노력의 일환이다.



NVIDIA의 멀티모달 AI 전략은 단순히 성능이 뛰어난 개별 모델을 개발하는 수준을 넘어선다. 이들의 핵심 경쟁력은 고성능 GPU 하드웨어, 최적화된 소프트웨어 스택, 그리고 파운데이션 모델을 유기적으로 결합한 수직적 통합 생태계를 구축하는 데 있다.24 이 생태계는 AI 모델 개발의 전 과정을 가속화하고, 연구 단계의 혁신을 실제 산업 현장에 신속하게 배포할 수 있도록 지원한다.

- **하드웨어 기반:** A100, H100과 같은 데이터센터급 GPU는 VLM의 대규모 사전 학습 및 미세 조정을 위한 압도적인 연산 능력을 제공한다.8 동시에, NVIDIA Jetson Orin과 같은 엣지 컴퓨팅 플랫폼은 전력 효율적인 설계를 통해 학습된 모델을 자율주행차, 로봇, 스마트 카메라 등 실제 디바이스에 배포할 수 있는 기반을 마련한다.26
- **소프트웨어 스택:** CUDA는 GPU의 병렬 처리 능력을 최대한 활용할 수 있는 프로그래밍 환경을 제공하며, TensorRT-LLM과 같은 추론 최적화 라이브러리는 학습된 VLM의 배포 시 지연 시간을 최소화하고 처리량을 극대화한다.25 또한, Omniverse Isaac Sim과 같은 시뮬레이션 플랫폼은 고품질 합성 데이터를 생성하는 데 핵심적인 역할을 한다.25
- **파운데이션 모델:** VILA, NeVA, Cosmos Nemotron 등 다양한 목적에 맞게 개발된 파운데이션 모델들은 이 생태계의 정점에 위치한다.8 이 모델들은 NVIDIA의 하드웨어와 소프트웨어에 최적화되어 있어, 개발자들은 최고의 성능을 손쉽게 활용할 수 있다.

NVIDIA는 AI Playground와 같은 플랫폼을 통해 이러한 통합 생태계에 대한 접근성을 높인다.24 개발자들은 별도의 복잡한 설정 없이 브라우저에서 직접 최신 VLM을 테스트하고 그 성능을 체험할 수 있으며, 이는 기술의 빠른 전파와 채택을 유도하는 중요한 전략이다. 이처럼 NVIDIA는 데이터 생성, 모델 학습, 최적화, 배포에 이르는 AI 개발의 전체 파이프라인을 아우르는 포괄적인 솔루션을 제공함으로써 강력한 기술적 해자(moat)를 구축하고 있다.


NVIDIA는 단일 모델이 아닌, 다양한 용도와 성능 요구사항에 대응할 수 있는 다각화된 VLM 포트폴리오를 보유하고 있다. 각 모델은 고유한 특징과 강점을 가지며, 이는 NVIDIA가 멀티모달 AI 시장의 다양한 니즈를 공략하고 있음을 보여준다.

| 모델명 (Model)      | 기반 LLM (Base LLM) | 주요 특징 (Key Features)                                     | 최적 적용 분야 (Optimal Use Case)                            | 공개 여부 (Availability)        |
| ------------------- | ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------- |
| **VILA**            | Llama, Yi 등        | 효율적 사전학습, 다중 이미지 추론, 문맥 내 학습(ICL), 엣지 배포 최적화 (4-bit AWQ) | 범용 이미지/비디오 분석, 엣지 AI 애플리케이션, 로보틱스, 자율주행 | 오픈소스 8                      |
| **Cosmos Nemotron** | 비공개 (VILA 기반)  | VILA 혁신 통합, 시공간 추론, 물리적/가상 세계 쿼리 및 요약, 다중 이미지 분석 | 디지털 트윈, 스마트 시티, 자율 시스템의 상황 인식, 산업 자동화 | 상용 (NVIDIA AI Enterprise) 26  |
| **NeVA**            | 비공개              | 멀티모달 대화형 AI 비서, 텍스트/이미지 기반 정보 제공 및 질의응답 | 대화형 AI 챗봇, 교육용 보조 도구, 콘텐츠 생성 및 검색        | AI Playground 통해 체험 가능 24 |
| **Eagle 2.5**       | 비공개              | 장편 비디오(Long Video) 이해 특화, 긴 영상의 서사 구조 및 핵심 정보 파악 | 미디어 콘텐츠 분석, 영상 요약, 보안 및 감시 영상 분석        | 연구 발표 32                    |

**VILA(Visual Language model)**는 NVIDIA의 VLM 연구를 대표하는 모델로, 효율적인 학습 방법론과 뛰어난 성능으로 주목받았다.8 특히 다중 이미지를 동시에 이해하고 추론하는 능력과, 몇 가지 예시만으로 새로운 작업을 학습하는 문맥 내 학습(In-context Learning) 능력이 뛰어나다.27 또한, 경량화 및 양자화 기술을 통해 엣지 디바이스에도 배포가 가능하다는 점에서 실용성이 높다. 2025년 1월부로 VILA의 핵심 기술은 상용 모델군인 **Cosmos Nemotron**에 통합되었다.8 Cosmos Nemotron은 VILA의 혁신을 계승하여 시공간 추론과 같은 더욱 진보된 기능을 제공하며, 물리적 또는 가상 환경(디지털 트윈)의 상태를 이해하고 요약하는 데 특화되어 있다.26

**NeVA(NVIDIA NeMo Vision and Language Assistant)**는 대화형 AI 비서의 형태로 제공되는 VLM으로, 사용자가 텍스트와 이미지를 활용해 자연스럽게 질문하고 정보를 얻을 수 있도록 설계되었다.24

**Eagle 2.5**는 수십 분 길이의 긴 비디오를 이해하는 데 특화된 모델로, 영상 전체의 서사적 흐름이나 중요한 사건을 파악하는 데 강점을 보여 미디어 분석이나 감시 분야에서 활용 가능성이 높다.32 이처럼 NVIDIA는 범용 모델부터 특정 도메인에 특화된 모델까지 다양한 라인업을 구축하여 멀티모달 AI 기술의 적용 범위를 넓혀가고 있다.


VLM을 포함한 모든 딥러닝 모델의 성능은 학습 데이터의 양과 질에 크게 좌우된다. 그러나 현실 세계에서 대규모의 고품질 이미지-캡션 쌍 데이터를 수집하고 레이블링하는 작업은 막대한 비용과 시간이 소요된다. NVIDIA는 이러한 데이터 병목 현상을 해결하기 위해 물리적으로 정확한 3D 시뮬레이션 환경인 NVIDIA Omniverse 플랫폼 내의 합성 데이터 생성 프레임워크, **Isaac Sim Replicator**를 적극적으로 활용한다.25

Replicator는 현실과 유사한 가상 환경에서 다양한 객체, 조명, 카메라 각도 등을 프로그래밍 방식으로 제어하여 무한에 가까운 양의 학습 데이터를 생성할 수 있다. 특히, 합성 데이터는 완벽하고 일관된 레이블을 자동으로 생성할 수 있다는 결정적인 장점을 가진다. 장면 캡셔닝을 위한 VLM 학습 데이터 생성과 관련하여, Replicator는 다음과 같은 정교한 설정 매개변수를 제공하여 데이터의 품질과 특성을 세밀하게 제어한다.28

- **씬 그래프(Scene Graph) 생성 및 제어:**
  - `save_full_scene_graph`: 씬에 존재하는 모든 객체와 그들 간의 모든 공간적 관계(예: '의자' 위에 '컵'이 있고, '테이블' 옆에 '의자'가 있음)를 포함하는 완전한 그래프를 생성한다. 이는 매우 상세한 캡션을 생성하는 데 필요한 풍부한 정보를 제공한다.
  - `save_pruned_scene_graph` 및 `pruning_ratio`: 전체 씬 그래프는 정보가 너무 많아 오히려 학습에 방해가 될 수 있다. 이 옵션은 그래프를 최소 신장 트리(Minimum Spanning Tree, MST)로 단순화한 후, `pruning_ratio`에 지정된 비율만큼의 엣지만 남겨 정보의 양을 조절한다. 예를 들어, `pruning_ratio`를 0.5로 설정하면 가장 핵심적인 관계 50%만 캡션 생성에 사용되어 더 간결하고 핵심적인 설명이 가능하다.
- **의미론적 레이블링(Semantic Labeling):**
  - `attach_label_to_usd`: 씬 내의 3D 객체(USD prim)에 의미론적 레이블이 없는 경우, 파일 경로 이름을 기반으로 자동으로 레이블을 부착한다 (예: `/World/Objects/Chair` → `Chair`). 이 레이블은 캡셔닝 시스템이 객체를 인식하는 기본 단위가 되므로 매우 중요하다.
  - `use_ai_label`: 사전에 데이터베이스에 저장된, AI가 생성한 더 정교한 레이블을 활용하여 캡션의 의미적 풍부함을 더한다.
- **캡션 내용 및 형식 제어:**
  - `max_object_capacity`: 복잡한 씬에서 너무 많은 객체가 캡션에 포함되는 것을 방지하기 위해, 카메라 뷰에서 차지하는 면적(2D 바운딩 박스 크기)을 기준으로 상위 N개의 객체만 캡셔닝 대상으로 선택한다.
  - `global_caption`: 개별 객체와 그 관계를 나열하는 것을 넘어, "거실에는 소파와 테이블, 그리고 TV가 있다"와 같이 장면 전체를 요약하는 포괄적인 캡션을 생성하도록 지시한다.

이러한 기능들을 통해 개발자는 특정 목적에 맞는 맞춤형 캡셔닝 데이터셋을 대규모로 생성할 수 있으며, 이는 VLM이 다양한 상황과 객체에 대해 강건한 이해 능력을 갖추도록 하는 데 결정적인 역할을 한다.


아무리 뛰어난 단일 VLM이라도 모든 종류의 시각적 데이터와 태스크에서 최상의 성능을 발휘하기는 어렵다. 특히, 이미지와 비디오는 근본적으로 다른 특성을 가지며, 각 모달리티에 특화된 모델들은 서로 다른 강점을 가진다. 이러한 한계를 극복하고 캡셔닝의 정확도를 극대화하기 위해 NVIDIA 연구진은 **Wolf(WOrLd summarization Framework)**라는 새로운 프레임워크를 제안했다.18

Wolf의 핵심 철학은 단일 모델에 의존하는 대신, 여러 VLM을 '전문가 집단'으로 간주하고 그들의 강점을 상호 보완적으로 활용하는 **전문가 혼합(Mixture-of-Experts, MoE)** 접근법을 채택하는 것이다.18 이는 각기 다른 전문성을 가진 전문가들이 협력하여 더 나은 결론을 도출하는 인간 사회의 문제 해결 방식과 유사하다.

Wolf 프레임워크는 다음과 같은 다단계 요약 프로세스로 작동한다 18:

1. **계단식 시각 요약 (Cascading Visual Summarization):** 먼저, 비디오 기반 VLM보다 훨씬 더 방대한 양의 이미지-텍스트 쌍 데이터로 사전 학습되어 일반적으로 더 풍부한 시각적 개념을 이해하는 '이미지 기반 VLM' 전문가(예: GPT-4V, VILA-1.5)를 활용한다. 이 모델들은 비디오에서 추출된 개별 프레임들에 대해 상세한 캡션을 생성한다.
2. **LLM 기반 요약 및 통합:** 다음으로, 여러 프레임에서 생성된 캡션들은 강력한 'LLM' 전문가에게 전달된다. LLM은 이 텍스트 정보들을 종합하고, 중복을 제거하며, 시간적 흐름에 맞게 재구성하여 일관성 있는 비디오 레벨의 요약 캡션을 생성한다.
3. **협력적 개선:** 이 과정에서 여러 이미지 및 비디오 기반 VLM들이 협력하여 결과를 검증하고 개선함으로써, 단일 모델이 만들어낼 수 있는 환각(hallucination) 현상을 줄이고 사실적 정확도를 높인다.

실제로 자율주행 데이터셋인 Nuscenes와 로보틱스 데이터셋을 대상으로 한 실험에서, Wolf 프레임워크는 GPT-4V, Gemini-Pro-1.5와 같은 최신 단일 모델들보다 캡션 유사도와 품질(환각 감소) 측면에서 월등히 높은 성능을 기록했다.18 이는 복잡하고 중요한 태스크일수록, 여러 모델의 집단 지성을 활용하는 MoE 방식이 더 신뢰성 높은 결과를 제공할 수 있음을 시사한다.



VILA(Visual Language model)는 NVIDIA의 VLM 연구 역량이 집약된 모델로, 특히 유연성과 확장성 측면에서 뛰어난 **자동 회귀(Auto-regressive)** 아키텍처를 기반으로 한다.8 이 구조는 LLM이 텍스트를 생성하는 방식과 동일하게, 이전 토큰들을 기반으로 다음 토큰을 순차적으로 예측하는 원리를 시각 정보 처리에도 적용한다. VILA는 이미지를 일종의 '외국어'로 간주하여, 시각적 정보를 텍스트 토큰과 동일한 임베딩 공간으로 변환한 뒤 이를 LLM의 입력 시퀀스에 자연스럽게 통합한다.26 이러한 접근 방식은  `<텍스트1><이미지1><텍스트2><이미지2>...`와 같이 텍스트와 이미지가 임의의 순서로 혼합된(interleaved) 입력을 처리할 수 있게 하여, 다중 이미지 추론이나 대화형 시각 질의응답과 같은 복잡한 작업을 수행하는 데 결정적인 유연성을 제공한다.8

VILA의 아키텍처는 앞서 설명한 VLM의 표준적인 3단계 구조를 따른다.

- **Vision Encoder (비전 인코더):** VILA 1.5 모델은 Google이 개발한 SigLIP(Sigmoid Loss for Language Image Pre-training) 모델을 비전 인코더로 채택했다.34 SigLIP은 기존 CLIP이 사용하던 복잡한 contrastive loss 대신, 단순한 시그모이드(sigmoid) 손실 함수를 사용하여 학습 효율성을 높이면서도 뛰어난 성능을 유지하는 것이 특징이다. 이는 대규모 데이터셋에서 더 안정적이고 효율적인 학습을 가능하게 하여, VILA가 고품질의 시각적 표현을 학습하는 데 기여한다.
- **Large Language Model (LLM):** VILA는 특정 LLM에 종속되지 않고 Llama, Vicuna, Yi-34B 등 다양한 오픈소스 LLM을 기반으로 유연하게 구축될 수 있다.8 이는 최신 LLM의 발전상을 신속하게 VLM에 통합할 수 있는 확장성을 부여한다. LLM의 파라미터 크기와 사전 학습된 지식의 깊이가 VILA의 전반적인 언어 이해, 추론, 그리고 생성 능력의 상한선을 결정하는 중요한 요소로 작용한다.
- **Projector (프로젝터):** 비전 인코더가 출력한 시각적 임베딩을 LLM이 이해할 수 있는 언어적 토큰 임베딩 공간으로 매핑하는 역할을 수행한다. VILA의 초기 연구에서는 이 프로젝터의 설계가 모델의 학습 방식과 성능에 미치는 영향을 심도 있게 탐구했다.9 연구진은 여러 개의 트랜스포머 블록으로 구성된 복잡한 프로젝터보다, 오히려 단순한 단일 선형 계층(single linear layer)으로 구성된 프로젝터가 더 나은 성능을 보일 수 있다는 흥미로운 가설을 제시했다. 그 이유는 단순한 프로젝터가 시각-언어 변환 작업을 최소한으로 수행하게 되므로, LLM 자체가 시각적 개념을 더 깊이 학습하도록 강제하여 결과적으로 모델의 일반화 성능을 향상시킨다는 것이다.9

이처럼 VILA는 검증된 오픈소스 구성 요소들을 효과적으로 조합하고, 이들 간의 상호작용을 최적화하는 데 집중함으로써 강력하고 효율적인 VLM 아키텍처를 구현했다.


VILA가 기존의 SOTA(State-of-the-Art) 모델들을 능가하는 성능을 달성할 수 있었던 비결은 단순히 새로운 아키텍처를 설계했기 때문이 아니라, 모델을 '어떻게 학습시킬 것인가'에 대한 깊이 있는 탐구를 통해 정립한 독창적인 **'학습 레시피(training recipe)'**에 있다.9 이 레시피는 데이터의 종류와 조합, 그리고 학습 단계별 파라미터 업데이트 전략을 정교하게 설계하여 VLM의 잠재력을 최대한 끌어내는 것을 목표로 한다. VILA의 학습 과정은 크게 세 단계로 구성된다.

| 학습 단계 (Training Stage)       | 주요 목표 (Objective)                | 핵심 전략 (Key Strategy)                                     | 기대 효과 (Expected Outcome)                                 |
| -------------------------------- | ------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Projector Initialization**     | 시각-언어 임베딩 공간 초기 정렬      | Vision Encoder와 LLM 동결, Projector만 학습                  | 두 모달리티 간의 기본적인 매핑 학습, 안정적인 사전학습의 기반 마련 9 |
| **Vision-Language Pre-training** | 깊은 모달리티 정렬 및 핵심 능력 확보 | 인터리브(Interleaved) 데이터 사용 및 LLM 동결 해제(Unfreezing) | LLM의 언어 능력 저하 방지, 다중 이미지 추론 및 문맥 내 학습(ICL) 능력 확보 8 |
| **Supervised Fine-tuning (SFT)** | 명령어 추종 능력 강화 및 성능 최적화 | 시각-언어 명령어 데이터와 텍스트 전용 명령어 데이터 재혼합(Re-blending) | VLM 및 텍스트 전용 작업 성능 동시 향상, 전반적인 명령어 이해 및 수행 능력 극대화 8 |

Stage 1: 인터리브(Interleaved) 데이터 사전 학습의 중요성

VILA 연구의 가장 중요한 발견 중 하나는 사전 학습 데이터의 형식이 모델의 핵심 능력에 미치는 지대한 영향을 규명한 것이다. 기존 VLM들은 주로 <이미지, 캡션> 형태의 이미지-텍스트 쌍 데이터(예: COYO 데이터셋)를 사용해왔다.8 그러나 VILA 연구진은 이러한 방식이 LLM의 텍스트 전용 작업 성능을 심각하게 저하시키는 **"치명적 망각(catastrophic forgetting)"** 현상을 유발함을 발견했다. MMLU 벤치마크에서 17.2%에 달하는 정확도 하락이 관찰되었는데, 이는 이미지 캡션의 텍스트가 일반적으로 짧고 단순하여 LLM이 기존에 학습했던 복잡한 언어 구조를 잊어버리기 때문이다.8

이에 대한 해결책으로 VILA는 문서나 웹페이지처럼 텍스트와 이미지가 자연스럽게 섞여 있는 **인터리브(interleaved) 데이터**(예: MMC4 데이터셋)를 사전 학습에 적극적으로 활용했다.8

`<텍스트1><이미지1><텍스트2>...`와 같은 형식의 데이터는 순수 텍스트 말뭉치와 통계적 분포가 유사하여, LLM이 자신의 언어 능력을 거의 그대로 보존(MMLU 성능 저하 약 5%)하면서 시각 정보를 학습할 수 있게 한다.8 더 중요한 것은, 이 데이터 형식이 모델에게 여러 이미지와 텍스트를 아우르는 넓은 문맥을 제공함으로써, 자연스럽게 **다중 이미지 문맥 내 학습(In-context Learning, ICL)** 능력을 부여한다는 점이다.8

Stage 2: LLM 동결 해제(Unfreezing)의 효과

사전 학습 과정에서 LLM의 파라미터를 고정(freeze)하고 프로젝터만 학습시키는 '프롬프트 튜닝' 방식은 계산적으로 효율적이며 LLM의 원래 성능을 보존하는 데 유리해 보였다.9 VILA의 실험 결과, 이 방식은 제로샷(zero-shot) 성능에서는 준수한 결과를 보였지만, ICL 성능에서는 심각한 한계를 드러냈다.9 즉, 모델이 새로운 예시를 보고 배우는 능력을 제대로 발휘하지 못했다.

반면, 사전 학습 단계에서 **LLM의 파라미터를 함께 업데이트(unfreezing)**하는 것은 시각 임베딩과 언어 임베딩 간의 더 깊고 본질적인 정렬(deep embedding alignment)을 촉진했다.8 이는 LLM 자체가 시각적 개념을 내재화하도록 유도하며, 결과적으로 VLM이 몇 가지 예시만으로도 새로운 시각적 작업을 능숙하게 수행하는 ICL 능력을 갖추는 데 결정적인 역할을 한다는 것이 VILA 연구의 핵심적인 발견이다.35

Stage 3: 지도 미세 조정(SFT)과 텍스트 데이터 재혼합

사전 학습을 통해 기본적인 시각-언어 정렬을 마친 모델은, 사용자의 다양한 지시를 정확하게 이해하고 수행하는 능력을 강화하기 위해 지도 미세 조정(Supervised Fine-tuning, SFT) 단계를 거친다.26 이 단계에서는 "이 이미지에 대해 설명해줘", "이미지 속 남자는 무엇을 하고 있니?"와 같은 <명령어, 정답> 쌍으로 구성된 데이터를 사용하여 모델을 학습시킨다.

VILA는 여기서 한 걸음 더 나아가, SFT 데이터에 소량의 **'텍스트 전용' 명령어 데이터를 다시 혼합(re-blending)**하는 전략을 사용했다.8 이 간단해 보이는 기법은 두 가지 중요한 효과를 가져왔다. 첫째, 사전 학습 과정에서 발생했던 미미한 수준의 텍스트 성능 저하(약 5%)를 완벽하게 회복시켰다. 둘째, 놀랍게도 텍스트 명령어 추종 능력이 향상되자, 이것이 모델의 전반적인 추론 능력 강화로 이어져 시각 언어 작업의 성능까지 동반 상승시키는 결과를 낳았다.8 이는 모달리티에 상관없이 '명령어를 잘 이해하는 능력' 자체가 모델의 핵심 역량임을 시사한다.

마지막으로 VILA의 학습 레시피는 **'데이터의 양보다 질'**이라는 원칙을 강조한다.8 연구진은 COYO-700M 데이터셋 전체를 사용하는 대신, 이미지와 텍스트 간의 관련성을 나타내는 CLIP 점수를 기준으로 상위 5%의 고품질 데이터만 선별하여 사용했다. 실험 결과, 무작정 데이터 양을 두 배로 늘리는 것보다, 소량이라도 고품질 데이터를 추가하는 것이 벤치마크 성능 향상에 훨씬 더 효과적임이 입증되었다.8


VILA의 우수성은 다양한 표준 벤치마크에서 기존 SOTA 모델들과의 직접적인 성능 비교를 통해 정량적으로 입증되었다. 특히, 동시대의 대표적인 오픈소스 VLM인 LLaVA-1.5와의 비교에서 VILA는 일관되게 더 높은 성능을 기록하며 그 학습 방법론의 효과를 증명했다.9

| 모델 (Model)               | MME (↑)    | TextVQA (↑) | OKVQA (↑) | MMMU (val, ↑) | VisWiz (↑) | MMBench-CN (↑) |
| -------------------------- | ---------- | ----------- | --------- | ------------- | ---------- | -------------- |
| LLaVA-1.5 (Vicuna-1.5-13B) | 1531.3     | 61.3        | 80.0      | -             | 47.9       | 63.8           |
| **VILA (7B)**              | **1533.0** | **64.4**    | 79.9      | 36.3          | **51.8**   | **65.5**       |
| **VILA (13B)**             | **1570.1** | **66.6**    | **80.8**  | **37.9**      | 50.8       | **68.2**       |

주: 위 표의 수치는 VILA 연구 논문 9의 Table 5에서 발췌하여 재구성한 것이다. MME는 멀티모달 이해 능력, TextVQA는 이미지 속 텍스트 인식 및 추론, OKVQA는 외부 지식 기반 VQA, MMMU는 전문가 수준의 다분야 멀티모달 이해, VisWiz는 시각 장애인을 위한 VQA, MMBench-CN은 중국어 멀티모달 벤치마크를 평가한다. 화살표(↑)는 높을수록 좋은 성능을 의미한다.

위 표에서 나타나듯, VILA는 대부분의 벤치마크에서 LLaVA-1.5를 능가하는 성능을 보인다. 특히 주목할 점은 **7B 파라미터 크기의 VILA 모델이 13B 크기의 LLaVA-1.5 모델과 대등하거나 일부 벤치마크(TextVQA, VisWiz)에서는 오히려 더 높은 점수를 기록했다는 사실이다**.9 이는 VILA의 향상된 사전 학습 레시피가 모델의 파라미터 효율성을 크게 높였음을 시사하는 강력한 증거이다. 13B 모델 간의 비교에서는 그 격차가 더욱 벌어져, VILA가 멀티모달 이해, 텍스트 인식, 외부 지식 활용 등 전반적인 능력에서 우위를 점하고 있음을 명확히 보여준다.

성능에 영향을 미치는 다른 주요 요인에 대한 분석 또한 VILA 연구의 중요한 부분이다.

- **이미지 해상도의 영향:** 연구진은 이미지의 원본 해상도가 VLM 성능에 미치는 영향을 분석했다. 실험 결과, 이미지 해상도를 224x224에서 336x336으로 높이자, 이미지 속 텍스트를 읽고 질문에 답해야 하는 TextVQA 벤치마크의 정확도가 41.6%에서 49.8%로 크게 향상되었다.8 이는 단순히 LLM에 입력되는 시각 토큰의 수를 늘리는 것보다, 원본 이미지의 세밀한 정보를 보존하는 것이 정교한 시각적 이해가 필요한 작업에 훨씬 더 중요하다는 것을 의미한다.8
- **양자화 후 성능 유지:** 모델의 실용성을 결정하는 중요한 요소는 경량화 이후에도 성능이 유지되는지 여부이다. VILA는 4비트 AWQ 양자화를 적용한 후에도 주요 벤치마크에서 성능 저하가 미미한 수준임을 보여주었다. 예를 들어, VILA1.5-13B 모델의 경우, fp16 정밀도에서 1570점이었던 MME 점수가 int4 양자화 후 1531점으로 소폭 하락했으며, MMMU 테스트 점수는 오히려 33.6점에서 34.0점으로 소폭 상승하는 등, 효율성과 정확성 간의 성공적인 균형을 달성했음을 입증했다.8


아무리 뛰어난 성능의 모델이라도 실제 서비스 환경, 특히 자원 제약이 심한 엣지 디바이스에서 효율적으로 동작하지 못한다면 그 가치는 제한적이다. NVIDIA는 VILA를 개발 초기부터 실제 배포 환경을 고려하여 설계했으며, 이를 위해 강력한 최적화 기술들을 적용했다.

- **AWQ (Activation-aware Weight Quantization):** VILA의 경량화 및 가속화의 핵심에는 4비트 AWQ 양자화 기술이 있다.8 양자화는 모델의 가중치를 표현하는 데 사용되는 비트 수를 줄여(예: 16비트 부동소수점 → 4비트 정수) 모델 크기를 대폭 감소시키고 메모리 대역폭 요구량을 낮추는 기술이다. 여러 양자화 기법 중 AWQ는 가중치뿐만 아니라 활성화 값(activation)의 분포까지 고려하여 양자화로 인한 정보 손실을 최소화한다. 특히, AWQ는 GPTQ와 같은 다른 기법과 달리 학습 데이터에 대한 역전파나 재구성 과정이 필요 없어, VLM과 같이 새로운 모달리티가 추가된 모델에 적용했을 때 일반화 성능이 우수하다는 장점을 가진다.8 VILA는 모델 파라미터의 대부분을 차지하는 LLM 부분에만 선택적으로 AWQ를 적용하여, 전체 추론 지연 시간의 4% 미만을 차지하는 비전 인코더 부분의 정밀도는 유지하면서도 높은 압축률과 속도 향상을 달성했다.8
- **TensorRT-LLM:** VILA는 NVIDIA의 고성능 딥러닝 추론 최적화기 및 런타임 라이브러리인 TensorRT-LLM과 완벽하게 호환된다.8 TensorRT-LLM은 커널 퓨전, 인플라이트 배치(in-flight batching) 등 다양한 최적화 기술을 적용하여 GPU에서의 LLM 추론 속도를 극대화한다. AWQ로 양자화된 VILA 모델을 TensorRT-LLM을 통해 실행하면 하드웨어의 성능을 최대한으로 끌어낼 수 있다.

이러한 최적화 기술들의 조합을 통해 VILA는 놀라운 추론 효율성을 달성했다. 예를 들어, 140억 파라미터의 VILA-14B 모델이 단일 NVIDIA RTX 4090 GPU에서 토큰당 10ms의 속도로 실행될 수 있었다.8 이는 데이터센터급의 강력한 인프라뿐만 아니라, 

**NVIDIA Jetson Orin**과 같은 저전력, 소형 폼팩터의 엣지 AI 플랫폼에서도 VLM을 실시간에 가깝게 구동할 수 있음을 의미한다.26 이로써 VILA는 자율주행 자동차의 실시간 상황 인식, 공장 자동화 로봇의 비전 기반 작업 수행, 스마트 카메라의 동적 이벤트 감지 등 지연 시간에 민감한 실제 산업 현장에서의 적용 가능성을 활짝 열었다.27


VILA의 성공적인 연구 결과를 바탕으로, NVIDIA는 VLM이 직면한 핵심적인 난제인 '효율성'과 '신뢰성'을 해결하기 위한 후속 연구를 지속했다. 그 결과물인 NVILA와 VILA2는 각각 독창적인 접근법을 통해 VLM의 성능과 실용성을 한 단계 더 높은 수준으로 끌어올렸다.


VILA가 고해상도 이미지를 처리할 때 더 높은 정확도를 보인다는 사실은, 모델이 더 많은 시각적 정보를 입력받을수록 성능이 향상됨을 시사했다.9 그러나 고해상도 이미지나 장편 비디오는 LLM에 입력되는 시각 토큰의 수를 기하급수적으로 증가시켜, 트랜스포머의 어텐션 메커니즘에서 발생하는 이차적인 계산 복잡도($O(n^2)$)로 인해 훈련 및 추론 비용이 감당할 수 없는 수준으로 치솟는 문제를 야기했다. **NVILA**는 이러한 딜레마를 해결하기 위해 **'Scale-then-Compress(확장 후 압축)'**라는 직관적이면서도 효과적인 전략을 제시했다.38

'Scale-then-Compress' 전략은 두 단계로 구성된다.

1. **Scale (확장):** 먼저, 정확도의 상한선을 높이기 위해 입력 데이터의 정보량을 최대한 보존하는 방향으로 '확장'한다. 공간적 차원에서는 이미지 해상도를 최대 896x896까지 높이고, 시간적 차원에서는 비디오에서 샘플링하는 프레임 수를 대폭 늘린다.38 이를 통해 모델은 원본 시각 데이터에 담긴 세밀한 디테일과 시간적 변화를 놓치지 않고 학습할 수 있다.
2. **Compress (압축):** 다음으로, 확장된 시각 정보를 효율적으로 처리하기 위해 비전 인코더를 통과한 후의 시각 토큰들을 '압축'한다. 예를 들어, 공간적으로 인접한 토큰들을 평균 풀링(average pooling)하여 하나의 토큰으로 합치거나, 시간적으로 연속된 프레임 그룹의 토큰들을 통합하여 대표 토큰을 생성하는 방식이다.38 이 압축 단계를 통해 LLM에 최종적으로 입력되는 시퀀스의 길이를 크게 줄여, 계산 효율성을 확보하면서도 '확장' 단계에서 얻은 풍부한 정보의 핵심은 유지할 수 있다.

이 전략을 통해 NVILA는 VILA 대비 훈련 속도를 4.5배, 추론 시 첫 토큰 생성 시간(Time to First Token)을 1.8배, 전체 처리량을 1.2배 향상시키는 등 압도적인 효율성 증대를 달성했다.38 동시에, 장편 비디오 캡셔닝 평가 점수는 2.00에서 3.26으로 1.6배 향상되었고, 1400개 프레임(약 27만 토큰 길이)의 비디오에서 '건초더미 속 바늘 찾기' 테스트를 99.5%의 정확도로 통과하는 등 정확도와 장기 문맥 이해 능력까지 크게 개선되었다.40 이는 효율성과 정확성이라는 두 마리 토끼를 동시에 잡은 성공적인 엔지니어링 접근법이라 평가할 수 있다.


VLM의 또 다른 근본적인 난제는 학습 데이터의 품질에 대한 의존성과 그로 인해 발생하는 환각(Hallucination) 현상이다. 데이터의 작은 오류나 편향이 모델의 신뢰성을 크게 저해할 수 있다. **VILA2**는 "VLM이 스스로의 훈련 데이터를 개선하고 성능을 향상시킬 수 있는가?"라는 질문에서 출발하여, **VLM-Augmented-VLM(VLM에 의해 증강된 VLM)**이라는 혁신적인 패러다임을 제시했다.36

VILA2의 핵심 아이디어는 모델을 학습 데이터의 수동적인 소비자가 아닌, 데이터 품질 개선 과정에 능동적으로 참여하는 주체로 만드는 **'자기 증강(Self-augmentation)'** 루프를 구축하는 것이다.36

이 루프는 다음과 같이 작동한다.

1. **1단계 (Self-augmentation):** 먼저, 기존 데이터셋으로 학습된 초기 VILA 모델(VILA₀)을 준비한다. 그 다음, 이 VILA₀ 모델에게 사전 학습에 사용되었던 이미지들을 다시 보여주며, "이 이미지에 대해 가능한 한 상세하고 정확하게 묘사해줘"와 같은 특정 프롬프트를 사용하여 새로운 캡션을 생성하도록 지시한다(re-captioning). 이 과정은 기존의 짧고 부정확할 수 있는 캡션을 모델 스스로가 생성한 더 풍부하고 고품질의 캡션으로 대체하는 과정이다.36
2. **2단계 (Re-training):** 원본 이미지와 VILA₀가 새로 생성한 고품질 합성 캡션으로 구성된 '증강된 데이터셋'을 사용하여, 다음 세대의 VILA 모델(VILA₁)을 처음부터 다시 훈련시킨다.36
3. **3단계 (Iteration):** 이 과정을 여러 번 반복한다 (VILA₁이 VILA₂를 위한 데이터를 생성하고, VILA₂가 다시 훈련되는 식). 실험 결과, 이 자기 증강 루프를 거칠수록 캡션의 길이는 유의미하게 길어지고 VQA 점수도 향상되었으나, 약 3회 반복 후에는 성능 향상 폭이 점차 둔화되는 **성능 포화(saturation)** 지점이 관찰되었다.36 이는 모델이 스스로의 지식 내에서만 데이터를 개선하는 데에는 한계가 있음을 시사한다.

이러한 포화 현상을 극복하고 모델의 성능을 지속적으로 향상시키기 위해, VILA2는 **'전문가 증강(Specialist-augmentation)'** 단계를 도입했다.41 자기 증강을 통해 일반적인 능력이 향상된 VILA 모델을 기반으로, 공간 관계 추론, 텍스트 인식(OCR), 객체 위치 특정(grounding) 등 특정 도메인에 대한 소량의 추가 데이터로 미세 조정한 '전문가 VLM'들을 만든다. 이 전문가 모델들이 생성한 특정 지식이 담긴 데이터를 다시 일반 모델(generalist VLM)의 학습에 활용함으로써, 일반 모델은 자신이 부족했던 전문 분야의 지식을 효과적으로 주입받게 된다.36

VILA2는 이러한 자기 증강 및 전문가 증강 접근법을 통해, 인간이 직접 레이블링하는 비용의 1/300 수준으로 데이터 품질을 획기적으로 개선했으며, 이를 통해 MMMU와 같은 고난도 벤치마크에서 새로운 SOTA 성능을 달성했다.42 이는 AI 모델 개발이 정적인 학습 데이터에 의존하는 단방향 프로세스에서, 모델이 스스로의 발전에 기여하는 순환적이고 동적인 피드백 루프로 진화할 수 있는 가능성을 보여준 중요한 연구이다.


VLM 기술은 눈부신 발전을 이루었지만, 실용적인 적용을 가로막는 근본적인 한계와 도전 과제들을 여전히 안고 있다. 그 중 가장 심각한 문제는 모델이 생성하는 정보의 신뢰성과 관련된 환각(Hallucination) 현상과 학습 데이터로부터 비롯되는 편향(Bias)이다. NVIDIA는 이러한 문제들을 해결하기 위해 모델 내부의 성능을 개선하는 동시에, 모델 외부에서 안전장치를 마련하는 이중적인 접근 전략을 취하고 있다.


환각은 AI 모델이 학습 데이터에 근거하지 않거나 명백한 사실과 다른 내용을 마치 사실인 것처럼 자신감 있게 생성하는 현상을 의미한다.7 이는 사용자의 오해를 유발하고 잘못된 결정을 내리게 할 수 있어, VLM을 금융, 의료, 자율주행과 같은 고신뢰성이 요구되는 분야에 적용하는 데 가장 큰 걸림돌로 작용한다.2

- **환각의 원인:**

  - **데이터 기반 원인:** VLM의 환각은 학습 데이터의 통계적 패턴에서 비롯되는 경우가 많다. 예를 들어, 학습 데이터에서 '의사'라는 단어가 주로 '청진기'와 함께 등장했다면, 모델은 청진기가 없는 의사 이미지에 대해서도 "청진기를 목에 건 의사"라고 서술하는 '객체 환각'을 일으킬 수 있다.44 더 나아가, 객체 간의 관계에서도 환각이 발생한다. '사람'과 '자전거'가 '타다(ride)'라는 관계로 자주 함께 등장했다면, 자전거 옆에 서 있는 사람을 보고도 "자전거를 타고 있는 사람"이라고 잘못된 '관계 환각'을 생성할 수 있다. 이러한 현상은 데이터 내에 존재하는 특정 패턴의 공존(co-occurrence) 관계를 모델이 맹목적으로 학습하기 때문에 발생한다.44
  - **모델링 기반 원인:** 생성 모델 고유의 확률적 특성 또한 환각의 원인이 된다. 모델은 가장 그럴듯한 다음 단어를 예측하도록 학습되기 때문에, 정보가 불확실한 상황에서도 '추측'을 통해 문장을 이어가려는 경향이 있다.43 또한, VLM이 이미지의 시각적 내용을 무시하고, LLM이 사전 학습을 통해 내재화한 방대한 '상식'이나 '일반 지식'에 과도하게 의존하여 답변을 생성하는 경향도 중요한 원인으로 지적된다.44

- NVIDIA의 대응 전략:

  NVIDIA는 환각 문제에 대해 모델의 내재적 능력을 향상시키는 접근과 외부에서 통제하는 접근을 병행한다.

  - **모델 내부적 접근 (VILA2):** VILA2의 자기 증강 및 전문가 증강 방법론은 환각 문제에 대한 근본적인 해결책을 모색하는 시도이다.41 모델이 스스로 더 상세하고 정확한 캡션을 생성하도록 유도하고, 이를 다시 학습 데이터로 활용함으로써, 데이터의 품질을 높이고 부정확한 통계적 연관성을 줄여나가려는 전략이다. 이는 모델의 '체질'을 개선하여 환각 발생 가능성 자체를 낮추려는 접근법이라 할 수 있다.
  - **모델 외부적 접근 (NeMo Guardrails):** VLM이 아무리 발전해도 환각을 100% 제거하는 것은 현실적으로 불가능하다는 전제 하에, 모델의 출력물을 실시간으로 감시하고 제어하는 외부 안전장치인 **NeMo Guardrails**를 제공한다.45 NeMo Guardrails는 VLM과 사용자 사이에 위치하여, 사전에 정의된 규칙에 따라 모델의 행동을 제어한다. 예를 들어, 특정 주제(예: 경쟁사 제품)에 대한 언급을 금지하거나, 생성된 답변을 다른 검증 모델과 교차 확인하여 불일치가 발견되면 "정확한 정보를 제공할 수 없습니다"와 같은 안전한 대체 답변을 내보내도록 강제할 수 있다.45 이는 VLM의 창의성과 유연성을 활용하면서도, 치명적인 오류가 사용자에게 전달되는 것을 막는 실용적인 안전망 역할을 한다.

이러한 이중적 접근은 AI의 신뢰성 확보가 단일 기술이 아닌, 정교한 시스템 엔지니어링의 문제임을 보여준다. 점차 개선되는 생성 모델의 핵심 능력과, 이를 감싸는 결정론적이고 프로그래밍 가능한 안전 계층의 결합이 실용적인 AI 시스템의 표준이 될 가능성이 높다.


VLM은 학습 데이터의 거울과 같아서, 데이터에 존재하는 사회적, 통계적 편향을 그대로 학습하고 때로는 증폭시키는 경향이 있다. 이는 모델의 공정성과 일반화 성능을 저해하는 심각한 문제이다.

- **데이터 의존성과 편향:** VLM 학습에 사용되는 대규모 데이터셋은 인터넷에서 수집되는 경우가 많아, 현실 세계에 존재하는 성별, 인종, 연령 등에 대한 고정관념과 편견을 포함할 수 있다.1 연구에 따르면, VLM은 특정 성별이나 인종의 이미지에 대해 부정적인 단어를 연관 짓는 경향을 보이며, 이러한 편향은 VQA와 같은 다운스트림 태스크에서도 지속적으로 나타난다.48 또한, 의료 영상이나 제조 공정 이미지와 같이 고도로 전문화된 도메인에 대한 학습 데이터가 부족할 경우, 일반적인 데이터로 학습된 VLM은 해당 분야에서 정확한 분석과 설명을 생성하는 데 어려움을 겪는 

  **도메인 편향** 문제를 보인다.7

- **위치 편향 (Position Bias):** LLM에서 처음 발견된 "lost in the middle" 현상은 여러 개의 문서를 동시에 처리할 때, 입력 시퀀스의 처음과 끝에 있는 정보는 잘 기억하지만 중간에 있는 정보는 놓치는 경향을 말한다.50 최근 연구에 따르면 이러한 

  **위치 편향**은 VLM이 여러 이미지를 동시에 처리할 때도 동일하게 나타난다.51 오픈소스 VLM들은 주로 시퀀스의 끝에 위치한 이미지에 대한 추론을 잘 수행하는 '최신성 편향(recency bias)'을 보이는 반면, GPT-4o와 같은 상용 모델들은 처음과 끝은 잘 처리하지만 중간 위치의 이미지에 취약한 U자형 성능 곡선을 보인다. 입력 이미지의 수가 늘어날수록 이러한 편향은 더욱 심해져, 모델의 강건성과 신뢰성을 심각하게 저해하는 요인이 된다.51

- **VILA2의 잠재적 위험:** VILA2의 자기 증강 루프는 데이터 품질을 개선하는 강력한 메커니즘이지만, 동시에 위험성을 내포하고 있다. 만약 초기 모델(VILA₀)이 특정 편향이나 체계적인 오류를 가지고 있다면, 자기 증강 과정은 이 편향과 오류를 담은 데이터를 대량으로 생산하고, 다음 세대 모델(VILA₁)은 이 오염된 데이터를 학습하여 편향을 더욱 강화하고 고착화시킬 수 있다.42 이는 마치 '메아리 방(echo chamber)'처럼 모델이 자신의 오류를 반복적으로 학습하며 증폭시키는 결과를 낳을 수 있다. 또한, 모델이 생성하는 캡션의 스타일이 점차 획일화되어 데이터의 다양성이 감소할 위험도 존재한다. 따라서 "어떻게 자기 증강 과정에서 발생하는 오류의 누적과 편향의 증폭을 완화할 것인가"는 VILA2와 같은 접근법이 해결해야 할 핵심적인 연구 과제로 남아있다.42



NVIDIA는 장면 캡셔닝을 포함한 비전 언어 모델 분야에서 단순한 모델 개발사를 넘어, 포괄적인 생태계를 구축한 기술 리더로서의 입지를 공고히 했다. 이들의 전략은 Isaac Sim Replicator를 통한 고품질 합성 데이터 생성에서부터, VILA의 혁신적인 학습 방법론을 통한 고성능 모델 개발, 그리고 AWQ 양자화와 TensorRT-LLM을 통한 하드웨어 최적화 및 실제 엣지 디바이스 배포에 이르기까지, AI 개발의 전 주기를 아우르는 수직적 통합을 통해 강력한 시너지를 창출하고 있다.

VILA의 가장 핵심적인 기여는 VLM의 성능을 결정하는 것이 단순히 아키텍처의 문제가 아니라 '어떻게 사전 학습할 것인가'라는 방법론의 문제임을 깊이 있게 탐구하고 구체적인 해법을 제시한 데 있다. 인터리브 데이터의 적극적인 활용과 사전 학습 중 LLM 파라미터를 동결하지 않는 전략은, 기존 VLM들이 겪던 '치명적 망각' 문제를 해결하고 멀티모달 문맥 내 학습(ICL)이라는 새로운 가능성을 활짝 열었다. 이는 VLM 연구 커뮤니티에 중요한 청사진을 제공했다.

더 나아가, NVILA와 VILA2를 통해 VLM이 직면한 가장 큰 난제인 '효율성'과 '신뢰성' 문제를 정면으로 돌파하려는 노력을 보여주었다. NVILA의 'Scale-then-Compress' 전략은 고해상도 시각 정보를 처리하는 데 있어 정확성과 계산 비용 사이의 균형을 맞추는 실용적인 해법을 제시했으며, VILA2의 'Self-augmentation' 패러다임은 모델이 스스로의 발전에 기여하는 AI 개발의 새로운 가능성을 열었다. 이러한 지속적인 혁신은 NVIDIA가 VLM 기술의 최전선에서 연구 개발을 주도하고 있음을 명확히 보여준다.


NVIDIA의 VLM 기술은 현재의 성과에 머무르지 않고, 더욱 복잡하고 고차원적인 인지 능력을 향해 나아갈 것으로 전망된다. 향후 연구 및 발전 방향은 다음과 같이 예측할 수 있다.

- **장편 비디오 및 시공간 추론 능력의 고도화:** Eagle 2.5와 LongVILA 프로젝트에서 나타나듯, 수십 초 단위의 짧은 클립을 넘어 수십 분에서 시간 단위에 이르는 장편 비디오의 전체적인 서사 구조와 복잡한 시공간적 인과관계를 이해하고 추론하는 능력이 VLM의 핵심 연구 주제가 될 것이다.31 이는 미디어 분석, 교육, 보안 등 다양한 분야에서 새로운 응용을 창출할 것이다.
- **멀티모달 RAG(Retrieval-Augmented Generation)의 보편화:** 현재 VLM이 가진 환각 문제를 완화하고 사실 기반의 정확한 답변을 생성하기 위해, 외부 지식 베이스를 동적으로 참조하는 RAG 기술과의 결합이 더욱 중요해질 것이다.19 VLM이 비디오를 분석하여 생성한 지식 그래프를 벡터 데이터베이스에 저장하고, 사용자의 질문에 가장 관련성 높은 정보를 실시간으로 검색하여 답변 생성에 활용하는 방식은 VLM의 신뢰성을 획기적으로 높일 수 있다.
- **고차원적 추론과 물리 AI(Physics AI)로의 도약:** VLM은 단순히 보이는 것을 묘사하는 수준을 넘어, 이미지나 비디오에 명시적으로 나타나지 않는 물리 법칙, 객체 간의 상호작용, 그리고 행동의 의도를 추론하는 방향으로 발전할 것이다.33 이는 VLM이 디지털 트윈 환경이나 로보틱스, 자율주행 시스템에서 단순한 '눈'의 역할을 넘어, 상황을 예측하고 최적의 행동을 계획하는 '두뇌'의 역할까지 수행하게 될 것임을 의미한다.
- **다중 모델 아키텍처와 소형 언어 모델(SLM)의 부상:** 모든 작업을 거대하고 단일한 VLM으로 처리하려는 시도 대신, 특정 작업에 고도로 특화된 여러 개의 소형 VLM 또는 SLM을 에이전트처럼 유기적으로 조합하여 사용하는 '다중 모델 아키텍처'가 효율성과 정확성 측면에서 주목받을 것이다.53 이는 NVIDIA가 VILA를 3.5B부터 40B까지 다양한 크기로 제공하는 전략과도 일맥상통하며, 각 작업의 요구사항에 맞는 최적의 모델을 선택적으로 활용하여 전체 시스템의 효율을 극대화하는 방향으로 나아갈 것이다.

결론적으로, NVIDIA는 하드웨어, 소프트웨어, 데이터, 모델을 아우르는 강력한 생태계를 기반으로 VLM 기술의 한계를 지속적으로 확장하고 있다. 앞으로 VLM은 단순한 장면 캡셔닝 도구를 넘어, 현실 세계와 가상 세계를 더 깊이 이해하고 상호작용하는 범용 인공지능의 핵심 구성 요소로 자리매김할 것이다.


1. 비전 언어 모델(VLM)이란 무엇인가요? - IBM, 8월 21, 2025에 액세스, https://www.ibm.com/kr-ko/think/topics/vision-language-models
2. Introduction to Vision Language Models - OpenCV, 8월 21, 2025에 액세스, https://opencv.org/blog/vision-language-models/
3. 비전-언어 모델은 내부 세계 모델을 가지고 있는가? 원자적 평가를 향하여 - 한빛미디어, 8월 21, 2025에 액세스, https://m.hanbit.co.kr/channel/view.html?cmscode=CMS3041842691
4. ardino.tistory.com, 8월 21, 2025에 액세스, [https://ardino.tistory.com/37#:~:text=%EC%82%AC%EB%9E%8C%EC%9D%B4%20%EB%88%88%EC%9D%84%20%ED%86%B5%ED%95%B4,%EB%88%88'%EC%9D%84%20%EB%8B%AC%EC%95%84%20%EC%A3%BC%EB%8A%94%20%EA%B2%83%EC%9D%B4%EB%8B%A4.](https://ardino.tistory.com/37#:~:text=사람이 눈을 통해,눈'을 달아 주는 것이다.)
5. What are Vision-Language Models? | NVIDIA Glossary, 8월 21, 2025에 액세스, https://www.nvidia.com/en-us/glossary/vision-language-models/
6. What Are Vision Language Models (VLMs)? - IBM, 8월 21, 2025에 액세스, https://www.ibm.com/think/topics/vision-language-models
7. 멀티모달 VLM 기술 동향 - 한컴테크, 8월 21, 2025에 액세스, https://tech.hancom.com/multimodal-vlm-trends/
8. Visual Language Models on NVIDIA Hardware with VILA | NVIDIA ..., 8월 21, 2025에 액세스, https://developer.nvidia.com/blog/visual-language-models-on-nvidia-hardware-with-vila/
9. VILA: On Pre-training for Visual Language Models - CVF Open Access, 8월 21, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2024/papers/Lin_VILA_On_Pre-training_for_Visual_Language_Models_CVPR_2024_paper.pdf
10. Image Captioning Based on Semantic Scenes - MDPI, 8월 21, 2025에 액세스, https://www.mdpi.com/1099-4300/26/10/876
11. An Overview of Image Caption Generation Methods - PMC - PubMed Central, 8월 21, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC7199544/
12. Image Captioning: Bridging Computer Vision and Natural Language Processing - Comet.ml, 8월 21, 2025에 액세스, https://www.comet.com/site/blog/image-captioning-bridging-computer-vision-and-natural-language-processing/
13. eodud0582/Video_Caption_Generation_Using_Deep_Learning: Video caption generation model development using CNN and LSTM - GitHub, 8월 21, 2025에 액세스, https://github.com/eodud0582/Video_Caption_Generation_Using_Deep_Learning
14. A Comprehensive Analysis of Real-World Image Captioning and Scene Identification - arXiv, 8월 21, 2025에 액세스, https://arxiv.org/pdf/2308.02833
15. Image Captioning: A Deep Dive, 8월 21, 2025에 액세스, https://www.numberanalytics.com/blog/image-captioning-deep-dive
16. Image Captioning | ArcGIS API for Python - Esri Developer, 8월 21, 2025에 액세스, https://developers.arcgis.com/python/latest/guide/how-image-captioning-works/
17. Analysis of Research Trends in Deep Learning-Based Video Captioning - Korea Science, 8월 21, 2025에 액세스, https://koreascience.kr/article/JAKO202407845716286.page
18. Wolf: Dense Video Captioning with a World Summarization Framework - arXiv, 8월 21, 2025에 액세스, https://arxiv.org/html/2407.18908v2
19. Vision Language Model Prompt Engineering Guide for Image and Video Understanding, 8월 21, 2025에 액세스, https://developer.nvidia.com/blog/vision-language-model-prompt-engineering-guide-for-image-and-video-understanding/
20. 딥러닝 기반 비디오 캡셔닝의 연구동향 분석 - 정보처리학회 논문지 (KTSDE) - KISS, 8월 21, 2025에 액세스, https://kiss.kstudy.com/Detail/Ar?key=4071956
21. Vid2Seq 논문 정리 (1) - AI Journey - 티스토리, 8월 21, 2025에 액세스, https://baechu-story.tistory.com/76
22. Guide to Vision-Language Models (VLMs) - Encord, 8월 21, 2025에 액세스, https://encord.com/blog/vision-language-models-guide/
23. Vision Language Model-based Caption Evaluation Method Leveraging Visual Context Extraction - arXiv, 8월 21, 2025에 액세스, https://arxiv.org/html/2402.17969v1
24. The AI Playground | NVIDIA Research, 8월 21, 2025에 액세스, https://www.nvidia.com/en-us/research/ai-playground/
25. NVIDIA Deep Imagination Research, 8월 21, 2025에 액세스, https://research.nvidia.com/labs/dir/
26. Visual Language Intelligence and Edge AI 2.0 with NVIDIA Cosmos Nemotron, 8월 21, 2025에 액세스, https://developer.nvidia.com/blog/visual-language-intelligence-and-edge-ai-2-0/
27. VILA를 사용하는 NVIDIA 하드웨어의 시각적 언어 모델, 8월 21, 2025에 액세스, https://developer.nvidia.com/ko-kr/blog/visual-language-models-on-nvidia-hardware-with-vila/
28. VLM Scene Captioning - Isaac Sim Documentation, 8월 21, 2025에 액세스, https://docs.isaacsim.omniverse.nvidia.com/latest/action_and_event_data_generation/tutorial_replicator_caption.html
29. Explore Vision Models | Try NVIDIA NIM APIs, 8월 21, 2025에 액세스, https://build.nvidia.com/explore/vision
30. Researchers at NVIDIA AI Introduce 'VILA': A Vision Language Model that can Reason Among Multiple Images, Learn in Context, and Even Understand Videos : r/machinelearningnews - Reddit, 8월 21, 2025에 액세스, https://www.reddit.com/r/machinelearningnews/comments/1ckea5f/researchers_at_nvidia_ai_introduce_vila_a_vision/
31. NVlabs/VILA: VILA is a family of state-of-the-art vision language models (VLMs) for diverse multimodal AI tasks across the edge, data center, and cloud. - GitHub, 8월 21, 2025에 액세스, https://github.com/NVlabs/VILA
32. Eagle 2.5: 장편 동영상 이해에 혁신 가져온 엔비디아 모델 [AI 모두레터] | 블로그, 8월 21, 2025에 액세스, https://modulabs.co.kr/blog/eagle-nvidia-moduletter
33. 엔비디아, 긴 영상일수록 분석 능력 향상하는 VLM '이글 2.5' 출시 - AI타임스, 8월 21, 2025에 액세스, https://www.aitimes.com/news/articleView.html?idxno=169887
34. Vila Model Card - NVIDIA NIM APIs, 8월 21, 2025에 액세스, https://build.nvidia.com/nvidia/vila/modelcard
35. VILA: On Pre-training for Visual Language Models - arXiv, 8월 21, 2025에 액세스, https://arxiv.org/html/2312.07533v4
36. VILA2: VILA Augmented VILA - arXiv, 8월 21, 2025에 액세스, https://arxiv.org/html/2407.17453v1
37. Efficient-Large-Model/VILA1.5-3b - Hugging Face, 8월 21, 2025에 액세스, https://huggingface.co/Efficient-Large-Model/VILA1.5-3b
38. NVILA: Efficient Frontiers of Visual Language Models - NVlabs, 8월 21, 2025에 액세스, https://nvlabs.github.io/VILA/
39. NVILA: Efficient Frontier Visual Language Models - arXiv, 8월 21, 2025에 액세스, https://arxiv.org/html/2412.04468v1
40. LongVILA: Scaling Long-Context Visual Language Models for Long Videos - arXiv, 8월 21, 2025에 액세스, https://arxiv.org/html/2408.10188v1
41. [Literature Review] VILA$^2$: VILA Augmented VILA - Moonlight, 8월 21, 2025에 액세스, https://www.themoonlight.io/en/review/vila2-vila-augmented-vila
42. VILA^2: VLM Augmented VLM with Self-Improvement | OpenReview, 8월 21, 2025에 액세스, https://openreview.net/forum?id=M2YCdfxNVx
43. Hallucination (artificial intelligence) - Wikipedia, 8월 21, 2025에 액세스, https://en.wikipedia.org/wiki/Hallucination_(artificial_intelligence)
44. Evaluating and Analyzing Relationship Hallucinations in Large Vision-Language Models, 8월 21, 2025에 액세스, https://arxiv.org/html/2406.16449v1
45. Nvidia's New Solution Could Address AI 'Hallucination' Issues - Sourceability, 8월 21, 2025에 액세스, https://sourceability.com/post/nvidias-new-solution-could-address-ai-hallucination-issues
46. Prevent LLM Hallucinations with the Cleanlab Trustworthy Language Model in NVIDIA NeMo Guardrails, 8월 21, 2025에 액세스, https://developer.nvidia.com/blog/prevent-llm-hallucinations-with-the-cleanlab-trustworthy-language-model-in-nvidia-nemo-guardrails/
47. What are the limitations of current Vision-Language Models? - Milvus, 8월 21, 2025에 액세스, https://milvus.io/ai-quick-reference/what-are-the-limitations-of-current-visionlanguage-models
48. A Multi-dimensional study on Bias in Vision-Language models - ACL Anthology, 8월 21, 2025에 액세스, https://aclanthology.org/2023.findings-acl.403/
49. VILA-M3: Enhancing Vision-Language Models with Medical Expert Knowledge - arXiv, 8월 21, 2025에 액세스, https://arxiv.org/html/2411.12915v1
50. Unpacking the bias of large language models | MIT News, 8월 21, 2025에 액세스, https://news.mit.edu/2025/unpacking-large-language-model-bias-0617
51. Identifying and Mitigating Position Bias of Multi-image Vision-Language Models - arXiv, 8월 21, 2025에 액세스, https://arxiv.org/html/2503.13792v1
52. NVIDIA AI Blueprint로 비디오 검색 및 요약 에이전트 구축하기, 8월 21, 2025에 액세스, https://developer.nvidia.com/ko-kr/blog/build-a-video-search-and-summarization-agent-with-nvidia-ai-blueprint/
53. NVIDIA Research Validates Personal AI's 5-Year Thesis on Small Language Models in Enterprise, 8월 21, 2025에 액세스, https://www.personal.ai/pi-ai/nvidia-research-validates-personal-ais-5-year-thesis-on-small-language-models-in-enterprise
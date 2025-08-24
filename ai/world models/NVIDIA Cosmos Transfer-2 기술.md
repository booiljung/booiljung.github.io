# NVIDIA Cosmos Transfer-2 기술 (물리 AI 훈련을 위한 합성 데이터 생성의 혁신)



물리 AI(Physical AI)는 센서를 통해 물리 세계를 관찰하고 액추에이터를 통해 상호작용하는 인공지능 시스템으로 정의된다. 이러한 시스템은 로봇공학, 자율주행, 산업 자동화 등 다양한 분야에서 위험하거나, 노동 집약적이거나, 단조로운 물리적 작업을 대체하며 인간을 해방할 막대한 잠재력을 지니고 있다.1 그러나 물리 AI가 실제 환경에서 신뢰성 있게 작동하기 위해서는 정책 모델(policy model)을 훈련시키기 위한 방대하고 다양한 고품질 데이터가 필수적이다.

현실 세계에서 이러한 데이터를 수집하는 과정은 여러 근본적인 한계에 직면한다. 첫째, 비용과 시간의 문제다. 실제 로봇이나 차량을 운영하여 데이터를 수집하는 것은 막대한 자본과 오랜 시간을 필요로 한다.2 둘째, 안전 문제다. 특히 자율주행차의 경우, 위험한 '엣지 케이스(edge case)' 시나리오를 실제 도로에서 재현하며 데이터를 수집하는 것은 사실상 불가능하다.4 셋째, 데이터의 희소성이다. 드물게 발생하지만 시스템의 강건성(robustness)을 위해 반드시 학습해야 하는 특정 상황들은 현실에서 충분한 양을 확보하기가 매우 어렵다.5

이러한 데이터 병목 현상을 해결하기 위한 대안으로 시뮬레이션 기반의 합성 데이터 생성(Synthetic Data Generation, SDG)이 주목받아 왔다. 하지만 시뮬레이션 환경과 실제 물리 세계 사이에는 시각적 질감, 조명 조건, 물리적 상호작용의 미묘한 차이로 인한 '현실 격차(Sim-to-Real Gap)' 또는 '도메인 격차(Domain Gap)'가 존재한다.7 이 격차로 인해 시뮬레이션에서 완벽하게 훈련된 모델이라도 실제 환경에서는 성능이 급격히 저하되는 문제가 발생하며, 이는 물리 AI 상용화의 가장 큰 걸림돌 중 하나로 작용해왔다.9


이러한 데이터 딜레마와 현실 격차 문제를 근본적으로 해결하기 위한 새로운 패러다임으로 월드 파운데이션 모델(World Foundation Model, WFM)이 부상했다. WFM은 물리 세계의 '디지털 트윈(digital twin)'으로서, 과거의 관측 데이터와 현재의 외란(perturbation)을 입력받아 물리적으로 타당한 미래 상태를 예측하는 대규모 생성 모델이다.1 이는 물리 AI가 실제 세계의 위험에 노출되지 않고도 안전하게 상호작용하며 학습할 수 있는 무한한 가상 환경을 제공함으로써, 데이터 확장성 문제를 해결할 핵심 기술로 평가받는다.10

NVIDIA는 WFM을 대규모 언어 모델(LLM)이 텍스트 기반의 생성형 AI와 에이전트 AI에 가져온 혁신에 비견되는, 물리 AI 분야의 근본적인 돌파구로 보고 있다.11 NVIDIA의 비전은 WFM을 통해 물리 AI 개발을 민주화하고, 모든 개발자가 고도의 전문 지식이나 막대한 자원 없이도 일반 로봇 공학(general robotics)에 접근할 수 있는 생태계를 구축하는 것이다.2 이는 단순한 데이터 생성 기술을 넘어, AI가 물리 법칙을 내재한 생성형 엔진으로 진화하고 있음을 시사한다. 초기 합성 데이터가 절차적 생성이나 단순 렌더링에 의존했던 것과 달리, WFM은 '물리 기반(physics-based)' 및 '물리 인식(physics-aware)'을 통해 시간의 흐름, 객체의 영속성, 물리적 상호작용과 같은 세계의 근본적인 동역학을 학습한다.2 이는 데이터 생성의 패러다임이 '외형적 모방'에서 '내재적 이해'로 전환되고 있음을 의미하며, 생성된 데이터의 품질과 신뢰도를 근본적으로 향상시키는 원동력이 된다.


NVIDIA Cosmos는 이러한 비전을 실현하기 위한 통합 플랫폼으로, 예측(Predict), 변환(Transfer), 추론(Reason)이라는 세 가지 핵심 WFM으로 구성된다.13 이 중 Cosmos Transfer는 시뮬레이션에서 생성된 정제된 데이터를 입력받아, 이를 현실 세계의 복잡하고 다양한 조건(예: 날씨, 조명)에 맞게 변환하고 증강하는 '세계 대 세계(world-to-world)' 전이 모델의 역할을 수행한다.15 Cosmos Transfer의 등장은 데이터 파이프라인의 가치 사슬이 기존의 '데이터 정제(Data Curation)'에서 '데이터 변환(Data Transformation)'으로 확장되었음을 의미한다. 전통적인 데이터 파이프라인이 수집된 데이터를 필터링하고 레이블링하는 '정제'에 초점을 맞췄다면(e.g., Cosmos Curator), Cosmos Transfer는 이미 존재하는 '깨끗한' 시뮬레이션 데이터를 입력받아 이를 완전히 다른 도메인의 데이터로 '변환'한다.11 이는 데이터의 가치를 수동적으로 선별하는 것을 넘어, 능동적으로 증식시키고 새로운 가치를 창출하는 단계로의 진화를 의미하며, 데이터 자산의 투자수익률(ROI)을 극대화하는 새로운 산업적 접근법을 제시한다.

본 보고서는 NVIDIA Cosmos Transfer-2(Cosmos-Transfer1의 최신 릴리즈 및 관련 기술을 포괄하는 개념)의 기술적 아키텍처, 작동 원리, 그리고 핵심 적용 사례를 심층적으로 고찰하고자 한다. 또한, 경쟁 기술 및 철학과 비교 분석을 통해 물리 AI 개발 생태계에서 Cosmos Transfer-2가 가지는 전략적 의미와 미래 전망을 제시하는 것을 목표로 한다.



NVIDIA Cosmos는 단순히 개별 모델의 집합이 아니라, 물리 AI 개발의 전 과정을 가속화하기 위해 설계된 포괄적인 플랫폼이다.13 이 플랫폼은 최첨단 생성형 월드 파운데이션 모델(WFM), 데이터 처리 및 큐레이션 파이프라인, 안전을 보장하는 가드레일(guardrails), 그리고 효율적인 데이터 처리를 위한 시각적 토크나이저(visual tokenizer) 등을 유기적으로 통합하고 있다.2 개발자들의 접근성을 극대화하기 위해, 핵심 모델들은 PyTorch 스크립트 및 모델 체크포인트 형태로 NVIDIA NGC, Hugging Face, GitHub를 통해 공개적으로 제공된다.14


Cosmos 플랫폼의 핵심은 각각 고유한 역할을 수행하며 상호 보완적으로 작동하는 세 가지 WFM, 즉 Predict, Transfer, Reason으로 구성된다.


Cosmos Predict는 미래 세계 상태를 예측하는 데 특화된 모델이다.13 텍스트, 이미지, 비디오 등 다중 모달(multimodal) 입력을 기반으로 최대 30초 분량의 물리적으로 일관된 연속 비디오를 생성할 수 있다. 이는 자율주행차가 마주칠 수 있는 잠재적 위험 상황이나 로봇이 수행할 작업의 결과를 미리 시뮬레이션하는 등, 다양한 '엣지 케이스' 예측 데이터셋을 구축하는 데 활용된다. 또한, 다른 WFM들의 기반 모델로서 기능하며, 최신 버전인 Cosmos Predict-2는 추론 속도와 메모리 사용량을 개선하여 프로토타이핑 및 엣지 배포에 더 적합하도록 발전했다.14


Cosmos Transfer는 데이터 증강 및 변환을 담당하는 다중 제어(multi-control) 모델이다.13 이 모델의 핵심 기능은 단일 시뮬레이션 비디오나 실제 공간 비디오를 입력받아, 이를 다양한 환경 조건(예: 조명, 날씨, 지형)으로 신속하게 확장하는 것이다. NVIDIA Isaac Sim™이나 CARLA와 같은 물리 AI 시뮬레이션 프레임워크에서 생성된 3D 입력을 가속화하여, 개발자가 의도한 대로 정밀하게 제어 가능한 대규모 합성 데이터 생성 파이프라인을 구축하는 데 결정적인 역할을 한다.13


Cosmos Reason은 시공간적 맥락을 이해하고 추론하는 비전 언어 모델(Vision Language Model, VLM)이다.13 이 모델은 인간처럼 구조화된 사고 과정(Chain-of-Thought)을 통해 물리 세계를 이해한다. 주요 역할은 Predict나 Transfer를 통해 생성된 합성 데이터의 품질과 물리적 타당성을 평가하는 비평가(critic)로서 기능하는 것이다. 또한, 데이터 큐레이션 과정의 효율성을 높이기 위해 비디오에 대한 설명 캡션을 자동으로 생성하거나, 로봇이 복잡한 상황에서 최적의 행동을 결정하도록 의사결정을 지원하는 역할도 수행한다.14


WFM들이 최적의 성능을 발휘하기 위해서는 고품질의 데이터와 효율적인 처리 파이프라인이 필수적이다. Cosmos 플랫폼은 이를 위해 다음과 같은 지원 도구를 제공한다.

- **Cosmos Curator**: NVIDIA NeMo™ Curator 기술을 기반으로 하는 데이터 큐레이션 프레임워크로, 대규모 센서 데이터를 효율적으로 필터링, 주석 처리, 중복 제거하여 각 모델의 필요에 맞는 맞춤형 데이터셋을 신속하게 구축한다.2
- **Cosmos Tokenizer**: 이미지와 비디오를 AI 모델이 처리할 수 있는 컴팩트한 토큰으로 변환하는 최첨단 시각적 토크나이저다. 기존 기술 대비 8배 높은 압축률과 12배 빠른 처리 속도를 제공하여 모델 훈련의 효율성을 극대화한다.2
- **Guardrails**: 유해하거나 편향된 콘텐츠의 생성을 방지하기 위한 안전장치다. 유해한 프롬프트 입력을 차단하는 Pre-Guard와, 생성된 비디오의 안전성을 검사하고 개인정보 보호를 위해 얼굴을 흐리게 처리하는 Post-Guard로 구성되어 신뢰할 수 있는 모델 사용을 보장한다.1


Cosmos 플랫폼의 진정한 힘은 NVIDIA의 다른 시뮬레이션 플랫폼, 특히 NVIDIA Omniverse™ 및 Isaac Sim™과 결합될 때 발휘된다. 이 통합 워크플로우는 '데이터 생성-변환-평가'로 이어지는 완전한 폐쇄 루프(Closed-Loop)를 형성하며, 이는 데이터가 데이터를 낳는 기하급수적 성장 구조인 '데이터 플라이휠(Data Flywheel)' 효과를 창출한다.

1. **시나리오 생성 (in Isaac Sim)**: 개발자는 OpenUSD 기반의 물리적으로 정확한 3D 시뮬레이션 플랫폼인 Isaac Sim에서 로봇의 동작이나 자율주행차의 주행 시나리오를 생성한다.8
2. **데이터 변환 (with Cosmos Transfer)**: Isaac Sim의 Replicator 기능을 통해 생성된 세분화 맵, 깊이 맵 등 구조화된 데이터를 Cosmos Transfer에 입력한다. Cosmos Transfer는 이 데이터를 기반으로 다양한 조명, 날씨 조건을 적용한 수천, 수만 개의 사실적인 비디오 데이터로 변환 및 증강한다.11
3. **품질 평가 (with Cosmos Reason)**: 대량으로 생성된 합성 비디오 데이터는 Cosmos Reason을 통해 물리적 타당성, 시나리오의 적절성 등을 자동으로 평가받는다.14
4. **재학습 및 개선**: 이 피드백 루프를 통해 검증된 고품질의 데이터셋만이 선별되어 로봇 정책 모델이나 AV 인식 모델의 후속 학습(post-training)에 사용된다. 이렇게 개선된 모델은 다시 Isaac Sim에서 더 복잡하고 정교한 시나리오를 수행할 수 있게 되며, 이는 더 높은 품질의 초기 데이터를 생성하는 선순환으로 이어진다.

이러한 생태계는 단순히 기술의 집합을 넘어, 경쟁사가 쉽게 모방하기 힘든 강력한 해자(moat)를 구축하는 NVIDIA의 핵심 전략이다. 더 나아가, Cosmos 모델들을 Hugging Face와 GitHub에 공개하는 '개방성' 정책은 단순한 개발자 친화적 행보를 넘어선다.14 이는 아직 초기 단계인 물리 AI 시장에서 NVIDIA의 기술 스택을 사실상의 산업 표준으로 자리매김하게 하려는 전략적 포석으로 해석할 수 있다. 개발자들이 Cosmos를 기반으로 솔루션을 구축하게 함으로써, 자연스럽게 NVIDIA의 하드웨어(GPU, DGX, Thor)와 소프트웨어(Omniverse, CUDA) 생태계에 대한 의존도를 높여, 다가오는 물리 AI 시대의 '운영체제'와 같은 지위를 선점하려는 장기적인 비전이 담겨 있다.

아래 표는 Cosmos 플랫폼 내 핵심 모델들의 역할과 기능을 요약한 것이다.

**표 1: NVIDIA Cosmos 플랫폼 핵심 모델 비교**

| 모델명              | 핵심 역할             | 주요 기능                                                    | 주 입력 데이터                                      | 주 출력 데이터                                              |
| ------------------- | --------------------- | ------------------------------------------------------------ | --------------------------------------------------- | ----------------------------------------------------------- |
| **Cosmos Predict**  | 미래 상태 예측        | 텍스트/이미지/비디오 프롬프트 기반 미래 비디오 시퀀스 생성, 엣지 케이스 시뮬레이션 | 텍스트, 이미지, 비디오                              | 물리 법칙을 따르는 비디오                                   |
| **Cosmos Transfer** | 데이터 변환 및 증강   | 시뮬레이션 데이터를 다양한 환경(날씨, 조명)의 사실적 비디오로 변환, Sim-to-Real 격차 해소 | 세분화 맵, 깊이 맵, LiDAR, HD 맵 등 구조화된 비디오 | 제어 가능한 사실적 비디오                                   |
| **Cosmos Reason**   | 시공간 추론 및 평가   | 생성된 데이터의 물리적 타당성 및 상황 적합성 평가, 데이터 큐레이션, 로봇 의사결정 지원 | 비디오, 텍스트 프롬프트                             | 논리적 추론 과정(Chain-of-Thought), 텍스트(캡션, 평가 결과) |
| **Cosmos Curator**  | 데이터 전처리 및 관리 | 대규모 센서 데이터 필터링, 주석 처리, 중복 제거를 통한 고품질 학습 데이터셋 구축 | 원본 센서 데이터(비디오 등)                         | 정제되고 레이블링된 데이터셋                                |



NVIDIA Cosmos Transfer-2(기술 백서에서는 주로 Cosmos-Transfer1으로 지칭됨)의 기술적 근간은 확산 모델(Diffusion Model)에 기반한 조건부 월드 생성 모델이다.22 확산 모델은 생성 모델링 분야에서 고품질의 사실적인 이미지를 생성하는 능력으로 잘 알려져 있으며, Cosmos는 이를 비디오 영역으로 확장했다.

확산 모델의 기본 원리는 두 가지 과정으로 구성된다. 첫째, 순방향 과정(Forward Process)에서는 원본 비디오 데이터(‘x0‘)에 여러 시간 단계(‘t‘)에 걸쳐 점진적으로 가우시안 노이즈(‘ϵ‘)를 추가하여 순수한 노이즈 상태(‘xT‘)로 만든다. 둘째, 역방향 과정(Reverse Process)에서는 모델이 이 과정을 거꾸로 학습하여, 노이즈 상태에서 시작해 점진적으로 노이즈를 제거함으로써 원본 데이터를 복원한다.20 이 역방향 과정을 통해 모델은 데이터의 복잡한 분포를 학습하고 새로운 데이터를 생성할 수 있게 된다.

Cosmos Transfer는 NVIDIA의 또 다른 WFM인 Cosmos-Predict1의 확산 트랜스포머(Diffusion Transformer, DiT) 아키텍처를 기반으로 구축되었다.22 DiT는 기존의 U-Net 기반 확산 모델과 달리, 트랜스포머 아키텍처를 사용하여 확장성과 성능을 크게 향상시켰다. 이는 비디오와 같이 복잡한 시계열 데이터를 처리하는 데 특히 강력한 이점을 제공한다.


단순한 비디오 생성을 넘어 '제어 가능한' 생성을 구현하기 위해, Cosmos Transfer는 ControlNet 아키텍처를 도입했다.22 ControlNet은 사전 학습된 대규모 확산 모델의 가중치는 고정(frozen)시킨 채, 제어 신호를 처리하기 위한 별도의 신경망 분기(branch)를 추가하여 훈련하는 혁신적인 방식이다. 이를 통해 원본 모델의 강력한 생성 능력은 그대로 유지하면서, 적은 훈련 비용으로도 정밀한 제어 능력을 부여할 수 있다.

Cosmos Transfer는 이 구조를 확장하여 '다중 분기 ControlNet(Multi-Branch ControlNet)'을 구현했다. 각 제어 분기는 특정 모달리티의 입력을 처리하도록 특화되어 있다. 예를 들어, 한 분기는 깊이(Depth) 맵을 입력받아 3D 구조 정보를, 다른 분기는 세분화(Segmentation) 맵을 입력받아 객체의 의미론적 레이아웃 정보를, 또 다른 분기는 HD 맵을 입력받아 도로 구조 정보를 추출한다.15 이 외에도 캐니 엣지(Canny Edge), 블러(Blur), LiDAR 포인트 클라우드 등 다양한 모달리티가 지원된다. 이렇게 추출된 제어 정보들은 원본 DiT 모델의 각 레이어에 주입되어, 최종적으로 생성되는 비디오가 입력된 제어 신호의 모든 조건을 만족하도록 유도한다.

이러한 아키텍처 설계는 '생성'과 '제어'의 완벽한 분리를 통해 모듈성과 확장성을 극대화하는 전략적 이점을 가진다. 강력한 생성 능력을 가진 사전 학습된 DiT 모델의 지식은 그대로 활용하면서, LiDAR나 HD Map과 같은 새로운 제어 모달리티가 필요할 때마다 해당 분기만 추가로 훈련하면 된다. 이는 새로운 센서나 데이터 형식이 등장하더라도 전체 모델을 처음부터 재훈련할 필요 없이 유연하게 대응할 수 있음을 의미하며, 급변하는 물리 AI 기술 환경에서 플랫폼의 생명력과 확장성을 보장하는 매우 효율적인 설계 철학이다.


Cosmos Transfer의 제어 방식은 단순히 여러 조건을 동시에 적용하는 것을 넘어, 시공간적으로 적응형(spatially and temporally adaptive)이라는 특징을 가진다.22 이는 '시공간 제어 맵(Spatiotemporal Control Map)'이라는 독창적인 메커니즘을 통해 구현된다.

작동 원리는 각 제어 분기(e.g., 깊이, 세분화)의 출력에 시공간적 가중치 맵 $`w = \{w_1, w_2,..., w_N\}`$을 적용하는 것이다. 이 맵은 비디오의 각 프레임(시간, ‘t‘)과 각 픽셀 위치(공간, ‘(x,y)‘)에서 각각의 제어 모달리티가 최종 생성 결과에 얼마나 강한 영향력을 미칠지를 지정한다.22

이러한 동적 가중치 조절은 개발자에게 매우 높은 수준의 제어 자유도를 부여한다. 예를 들어, 자율주행 시뮬레이션에서 다음과 같은 미세 조정이 가능하다.

- 장면 전체의 3D 구조를 엄격하게 유지하고 싶을 경우, 모든 시간과 공간에서 깊이 맵의 가중치를 높게 설정한다.
- 전방 차량의 형태와 같은 중요한 객체의 디테일은 보존하면서 배경은 다양하게 생성하고 싶을 경우, 전방 차량 영역에만 엣지 맵의 가중치를 높이고 배경 영역의 가중치는 낮게 설정한다.

이 시공간 제어 맵은 단순한 가중치 조절 기능을 넘어, 개발자에게 '감독 수준의 편집 권한'을 부여하는 창의적 도구로 기능한다. 이는 합성 데이터 생성이 단순한 '데이터 복제'를 넘어, 명확한 '의도를 가진 데이터 창작'의 영역으로 진입했음을 보여주는 중요한 기술적 진보다.


Cosmos Transfer의 훈련 과정은 확산 모델의 일반적인 손실 함수를 기반으로 한다. 모델(‘ϵθ‘)은 노이즈가 추가된 비디오(‘xt‘)와 시간 스텝(‘t‘), 그리고 제어 신호(‘c‘)를 입력받아, 원본 비디오(‘x0‘)에 추가되었던 실제 노이즈(‘ϵ‘)를 예측하도록 훈련된다. 손실 함수 $`L`$는 이 예측 오차의 기댓값을 최소화하는 방향으로 정의된다.24
$$
L = E_{t, x_0, \epsilon} \left[ \left\| \epsilon - \epsilon_\theta(x_t, t, c) \right\|^2 \right]
$$
여기서 ControlNet의 역할은 제어 신호 $`c`$를 효과적으로 처리하여 모델 $`\epsilon_\theta`$에 통합하는 것이다. 제어 신호 ‘c‘(예: 깊이 맵 비디오)는 별도의 인코더 $`E_c`$를 통과하여 특징 벡터 $`c_f`$로 변환된다. 이 특징 벡터는 '제로 컨볼루션(zero-convolution)' 레이어를 통해 원본 DiT 블록의 각 레이어에 주입된다. 제로 컨볼루션은 가중치와 편향이 0으로 초기화된 ‘1×1‘ 컨볼루션 레이어로, 훈련 초기에는 제어 신호의 영향이 0이 되도록 하여 안정적인 학습을 돕는다. DiT의 중간 레이어 출력 $`h`$는 제어 신호 특징과 결합되어 $`h' = h + c_f`$와 같이 변형된 후 다음 레이어로 전달된다. 이 과정을 통해 제어 신호가 비디오 생성의 전 과정에 걸쳐 영향을 미치게 된다.


확산 모델의 가장 큰 실용적 단점은 고품질 결과를 얻기 위해 수십 번의 반복적인 추론 과정(denoising steps)이 필요하여 생성 속도가 느리다는 점이다. 대규모 데이터셋을 생성해야 하는 산업 현장에서 이는 심각한 병목이 될 수 있다.

NVIDIA는 이 문제를 해결하기 위해 '모델 증류(Model Distillation)' 기술을 Cosmos Transfer에 적용했다.15 이 기술은 수십 단계(예: 35~70단계)의 추론을 수행하는 크고 복잡한 '교사(teacher)' 모델의 출력을, 단 한 번의 추론만으로 유사한 결과를 내도록 훈련된 작고 효율적인 '학생(student)' 모델에게 가르치는 방식이다.

그 결과, Cosmos Transfer의 증류 모델은 추론 과정을 단 1단계로 압축하면서도 원본 모델의 품질에 근접한 결과를 생성할 수 있게 되었다.25 이 혁신은 추론 속도를 획기적으로 향상시켜 실시간에 가까운 비디오 생성을 가능하게 했으며, 대규모 합성 데이터 생성의 비용과 시간을 극적으로 단축시키는 핵심적인 기술적 성과로 평가된다.

아래 표는 Cosmos Transfer-2의 복잡한 기술적 사양을 요약하여 보여준다.

**표 2: Cosmos Transfer-2 아키텍처 요약**

| 구성 요소          | 기술 사양                                      | 주요 역할/기능                                               |
| ------------------ | ---------------------------------------------- | ------------------------------------------------------------ |
| **기반 모델**      | Cosmos-Predict1 (Diffusion Transformer, DiT)   | 고품질, 물리 인식 비디오 생성을 위한 사전 학습된 생성 능력 제공 |
| **핵심 아키텍처**  | 다중 분기 ControlNet (Multi-Branch ControlNet) | 사전 학습된 DiT 모델에 다양한 제어 신호를 주입하여 조건부 생성을 가능하게 함 |
| **제어 모달리티**  | 세분화 맵, 깊이 맵, 엣지 맵, LiDAR, HD 맵 등   | 생성될 비디오의 기하학적 구조, 의미론적 레이아웃, 객체 디테일 등을 정밀하게 제어 |
| **제어 융합 방식** | 시공간 제어 맵 (Spatiotemporal Control Map)    | 각 제어 신호의 영향력을 시간과 공간에 따라 동적으로 조절하여 미세 조정 및 적응형 제어 구현 |
| **성능 가속화**    | 모델 증류 (Model Distillation)                 | 다단계 추론 과정을 단일 단계로 압축하여 추론 속도를 극대화하고 실시간에 가까운 생성 지원 |


Cosmos Transfer-2의 기술적 우수성은 실제 물리 AI 개발 워크플로우에 적용되었을 때 그 가치가 명확히 드러난다. 특히 로보틱스와 자율주행 분야에서 Sim-to-Real 격차를 해소하고 AI 모델의 성능을 획기적으로 향상시키는 구체적인 성과를 보여주고 있다.


NVIDIA의 범용 로봇 파운데이션 모델 프로젝트인 'GR00T(Generalist Robot 00 Technology)'는 Cosmos Transfer를 핵심 데이터 증강 엔진으로 활용한다.26 그 워크플로우는 다음과 같이 구성된다.

1. **소수 시연 데이터 수집**: 먼저, 인간 조작자가 원격 조종(teleoperation)을 통해 소수의 로봇 조작 시연 데이터를 수집한다.27 이는 데이터 수집의 초기 비용과 노력을 최소화한다.
2. **합성 궤적 생성 (GR00T-Mimic)**: 수집된 소수의 시연 데이터는 NVIDIA Isaac Lab 시뮬레이션 환경 내에서 GR00T-Mimic 모델에 입력된다. 이 모델은 보간(interpolation) 및 외삽(extrapolation)을 통해 수십만 개의 물리적으로 타당한 합성 모션 궤적(synthetic motion trajectories)을 생성한다.26
3. **데이터 증강 (GR00T-Gen & Cosmos Transfer)**: 생성된 궤적의 시뮬레이션 이미지는 아직 시각적으로 단순하고 현실감이 떨어진다. 이 단계에서 GR00T-Gen 워크플로우의 일부로 Cosmos Transfer가 투입된다. Cosmos Transfer는 간단한 텍스트 프롬프트나 제어 신호를 통해 시뮬레이션 이미지의 배경, 조명, 색상, 재질 등 다양한 시각적 파라미터를 무작위화(Domain Randomization)하고, 사실적인 렌더링을 적용한다. 이 과정은 과거 수 시간이 걸리던 3D 씬 구축 및 렌더링 작업을 단 몇 분으로 단축시키는 극적인 효율성 향상을 가져온다.27
4. **정책 모델 후속 학습 (Post-training)**: Cosmos Transfer를 통해 사실성과 다양성이 극대화된 대규모 합성 데이터셋은 최종적으로 Isaac Lab에서 모방 학습(imitation learning) 기법을 통해 GR00T N1과 같은 로봇 정책 모델을 훈련시키는 데 사용된다.27

이 워크플로우의 정량적 성과는 매우 인상적이다. NVIDIA는 단 11시간 만에 78만 개의 합성 궤적을 생성했는데, 이는 인간이 9개월 동안 연속으로 시연해야 얻을 수 있는 6,500시간 분량의 데이터에 해당한다.27 더 중요한 것은, 이렇게 생성된 합성 데이터와 소량의 실제 데이터를 결합하여 GR00T N1 모델을 훈련했을 때, 실제 데이터만 사용한 경우에 비해 최종 성능이 

**40% 향상**되었다는 점이다.27 이 '40% 성능 향상'이라는 수치는 단순한 기술적 개선을 넘어, 데이터 수집의 경제성을 근본적으로 바꾸는 '게임 체인저'라 할 수 있다. 이는 동일한 성능 목표를 달성하기 위해 필요한, 수집 비용이 매우 비싼 실제 데이터의 양을 대폭 줄일 수 있음을 의미하며, 로봇 AI 개발의 진입 장벽을 낮추고 혁신을 가속화하는 경제적 파급 효과를 가진다.


자율주행 분야에서 Cosmos Transfer는 실제 도로에서 수집하기 어렵거나 위험한 엣지 케이스 시나리오 데이터를 대규모로 생성하는 데 핵심적인 역할을 한다.

1. **기반 데이터 확보**: 실제 차량에서 수집된 주행 비디오나, Omniverse 플랫폼 내에서 생성된 물리 기반 센서 데이터를 초기 입력으로 사용한다.11
2. **조건부 변형 생성**: 단일 주행 시나리오 비디오와 함께 HD 맵, LiDAR 스캔과 같은 구조화된 데이터를 Cosmos Transfer에 입력한다. 이 구조화된 데이터는 도로의 기하학적 구조와 같은 핵심적인 정보가 변형 과정에서 유지되도록 보장하는 '앵커' 역할을 한다.
3. **텍스트 프롬프트를 통한 제어**: 개발자는 "a snowy day", "a nighttime scene", "driving in heavy fog"와 같은 간단한 자연어 텍스트 프롬프트를 사용하여, 동일한 주행 경로와 교통 상황에 대해 무한에 가까운 시각적 변형을 생성할 수 있다.16

이러한 능력은 AV 훈련 패러다임을 '데이터 기반(data-driven)'에서 '시나리오 기반(scenario-driven)'으로 전환시키고 있다. 기존에는 특정 시나리오(예: 야간 눈길에서의 자전거 돌진)를 테스트하기 위해 해당 데이터가 수집될 때까지 수동적으로 기다려야 했다. 그러나 Cosmos Transfer를 사용하면 개발자는 특정 가설(예: "이 모델은 야간 눈길에서 자전거에 취약할 것이다")을 검증하기 위해 필요한 시나리오를 능동적으로 설계하고, 즉시 해당 데이터를 대량 생성하여 모델을 테스트하고 개선할 수 있다. 이미 Foretellix, Parallel Domain과 같은 주요 자율주행 시뮬레이션 및 검증 기업들이 이 기술을 도입하여 행동 시나리오를 강화하고, 더 강건한 AV 시스템을 위한 다양한 주행 데이터셋을 구축하고 있다.11


Cosmos Transfer의 궁극적인 목표는 Sim-to-Real 전이 성능을 극대화하는 것이다.22 이를 평가하기 위해 DreamGen Bench와 같은 특화된 벤치마크가 개발되었다. 이러한 벤치마크를 통한 실험 결과, 생성된 합성 데이터의 품질(사실성, 지시 따름 등)이 높을수록, 해당 데이터로 훈련된 로봇이 실제 조작 과제에서 더 나은 성능을 보인다는 일관된 경향이 확인되었다.28 또한, Cosmos 생태계 내에서 Cosmos Reason 모델이 생성된 데이터의 품질을 평가하는 역할을 수행하며, 여러 물리적 상식 추론 벤치마크에서 높은 점수를 기록함으로써 신뢰성 있는 자동 평가 시스템으로서의 가능성을 입증했다.19

아래 표는 Cosmos Transfer-2의 주요 성능 지표를 정량적으로 요약한 것이다.

**표 3: Cosmos Transfer-2 성능 벤치마크**

| 적용 분야     | 평가 항목               | 평가 결과                                                    | 관련 기술/프로젝트      | 출처 Snippet |
| ------------- | ----------------------- | ------------------------------------------------------------ | ----------------------- | ------------ |
| **로보틱스**  | GR00T N1 정책 모델 성능 | 합성 데이터 + 실제 데이터 결합 시, 실제 데이터만 사용했을 때보다 **40% 성능 향상** | Isaac GR00T, Isaac Lab  | 27           |
| **로보틱스**  | 데이터 생성 효율성      | 11시간 만에 **78만 개**의 합성 궤적 생성 (6,500시간 분량의 인간 시연 데이터) | GR00T-Mimic             | 27           |
| **자율주행**  | 데이터 다양성 증강      | 단일 주행 비디오로부터 텍스트 프롬프트를 통해 **다양한 날씨/조명 조건**의 비디오 생성 | Omniverse AV Blueprint  | 11           |
| **추론 속도** | 모델 증류 효과          | 35~70단계의 추론 과정을 **단 1단계**로 압축하여 실시간에 가까운 생성 속도 달성 | Edge Model Distillation | 15           |


NVIDIA Cosmos Transfer-2는 합성 데이터 생성 분야에서 중요한 기술적 진보를 이루었지만, 여전히 해결해야 할 본질적인 한계와 과제들을 안고 있다. 이러한 한계를 명확히 인식하는 것은 기술의 현재 위치를 객관적으로 평가하고 미래 발전 방향을 모색하는 데 필수적이다.


합성 데이터 기술의 발전에도 불구하고, 시뮬레이션과 현실 사이의 근본적인 격차는 여전히 존재한다. 이 '현실 격차'는 단일한 문제가 아니라, '시각적 격차(Appearance Gap)'와 '내용적 격차(Content Gap)'라는 두 가지 하위 문제의 복합체로 이해할 수 있다.8 Cosmos Transfer-2는 사실적인 렌더링, 조명, 텍스처 변환을 통해 '시각적 격차'를 해소하는 데 탁월한 성능을 보인다. 그러나 실제 세계에서 발생하는 예상치 못한 행위자들의 행동 패턴이나 물리 법칙의 미묘한 예외와 같은 '내용적 격차'는 여전히 큰 도전 과제로 남아있다.

- **분포 불일치(Distribution Bias)**: 합성 데이터는 생성 모델과 알고리즘에 의해 만들어지므로, 실제 세계 데이터가 갖는 복잡하고 미묘한 통계적 분포를 완벽하게 재현하기 어렵다. 이로 인해 모델이 특정 편향을 학습하여 현실 세계에서 오작동할 위험이 존재한다.29
- **과도한 평활화(Over-Smoothing)**: 생성 과정에서 데이터의 미세한 노이즈나 복잡한 디테일이 사라져 지나치게 '깨끗한' 데이터가 만들어질 수 있다. 이는 오히려 현실 세계의 불완전성에 대한 모델의 적응력을 떨어뜨리는 요인이 될 수 있다.29
- **알려지지 않은 미지(Unknown Unknowns)**: 합성 데이터의 가장 큰 위험은 우리가 알고 있거나 상상할 수 있는 시나리오, 즉 '알려진 미지(Known Unknowns)'(예: 눈 오는 날)를 기반으로 생성된다는 점이다. 그러나 현실 세계의 사고는 종종 우리가 상상조차 하지 못했던 '알려지지 않은 미지'의 조합으로 발생한다. 합성 데이터 파이프라인이 이러한 창발적(emergent)이고 예측 불가능한 시나리오를 어떻게 생성하고 학습에 포함시킬 수 있을지는 완전 자율성에 도달하기 위한 마지막 장벽이 될 수 있다.


WFM의 성능은 기반이 되는 물리 시뮬레이션의 정확성에 크게 의존한다.

- **복잡한 물리 현상**: 단단한 물체(rigid body)의 동역학은 비교적 잘 시뮬레이션되지만, 부드러운 물체(soft body)의 변형, 유체 역학, 복잡한 접촉 및 마찰 현상 등은 여전히 현실과 동일한 수준으로 정확하게 시뮬레이션하기 어려운 영역으로 남아있다.3
- **장기적 일관성**: 장시간의 비디오를 생성할 때, 초기에는 물리 법칙을 잘 따르다가 시간이 지남에 따라 점차 논리적 모순이 발생하거나 객체가 비현실적으로 변형되는 '표류(drifting)' 현상이 나타날 수 있다.30 이는 장기적인 계획 수립이 필요한 로봇 작업에서 치명적인 오류를 유발할 수 있다.


대규모 데이터로 학습되는 WFM은 데이터에 내재된 사회적 편향과 윤리적 문제를 그대로 물려받을 위험이 있다.

- **학습 데이터 편향**: Cosmos WFM은 2천만 시간 이상의 방대한 실제 비디오 데이터로 사전 학습된다.1 이 데이터에 포함된 인종, 성별, 지역적 편향이 생성된 합성 데이터에 그대로 전이되거나 심지어 증폭될 수 있다.
- **악용 가능성**: 고품질의 사실적인 비디오 생성 기술은 딥페이크나 가짜 뉴스 등 허위 정보 생성에 악용될 수 있는 양날의 검이다. NVIDIA는 이러한 위험을 완화하기 위해 생성된 비디오 내 인물의 얼굴을 자동으로 흐리게 처리하거나 유해 콘텐츠를 필터링하는 등의 가드레일 시스템을 도입했지만 2, 기술의 악용 가능성에 대한 지속적인 사회적, 기술적 논의가 필요하다.


이러한 한계를 극복하기 위해 다음과 같은 방향으로의 연구 개발이 필요하다.

- **실시간 상호작용 가능한 월드 모델**: 현재의 비디오 생성 모델을 넘어, 사용자의 행동에 실시간으로 물리적으로 일관된 반응을 보이는 상호작용 가능한(interactive) 월드 모델로의 발전이 필요하다.
- **차세대 물리 엔진과의 결합**: NVIDIA가 Google DeepMind, Disney Research 등과 공동 개발 중인 Newton과 같은 차세대 오픈소스 물리 엔진과의 긴밀한 통합을 통해 시뮬레이션의 물리적 정확도를 한 단계 더 높여야 한다.31
- **하이브리드 데이터 전략 고도화**: 순수한 합성 데이터와 실제 데이터를 단순히 섞는 것을 넘어, 어떤 종류의 데이터를 어떤 비율로, 어느 학습 단계에서 사용해야 최적의 성능을 낼 수 있는지에 대한 체계적인 연구(Sim-and-Real Co-Training)가 더욱 중요해질 것이다.28


NVIDIA Cosmos Transfer-2의 기술적, 전략적 가치를 더 깊이 이해하기 위해서는 경쟁 기술 및 철학과의 비교 분석이 필수적이다. 특히 자율주행 분야의 Tesla와 AI 연구 분야의 Google DeepMind는 주목할 만한 비교 대상이다.


자율주행 기술 개발에 있어 NVIDIA와 Tesla는 데이터 확보 및 활용에 대해 근본적으로 다른 철학을 가지고 있다. 이는 '통제된 다양성'과 '무작위적 현실성' 사이의 대립으로 요약할 수 있다.

- **NVIDIA (Simulation-First)**:
  - **철학**: NVIDIA의 접근법은 '시뮬레이션 우선(Simulation-First)'이다. 시뮬레이션을 통해 거의 무한에 가까운 다양성과 엣지 케이스를 안전하고 저렴하게 생성하여 데이터 병목을 해결하고, 이를 통해 현실 세계에 대한 모델의 일반화 성능을 확보하는 것을 목표로 한다.33 이는 필요한 시나리오를 '통제'하여 체계적이고 효율적으로 생성함으로써 데이터의 '다양성'을 확보하려는 전략이다.
  - **기술 스택**: Omniverse/Isaac Sim(시뮬레이션 환경) + Cosmos(생성/변환/평가) + DGX/Thor(컴퓨팅 하드웨어)로 이어지는 강력한 수직 계열화 풀스택 솔루션을 제공한다.26
  - **장점**: 초기 데이터가 부족한 상황에서도 개발을 시작할 수 있으며, 실제로는 테스트하기 위험한 시나리오를 안전하게 검증하고, 막대한 데이터 수집 비용을 절감할 수 있다.
- **Tesla (Real-World-Data-First)**:
  - **철학**: Tesla는 '실제 데이터 우선(Real-World-Data-First)' 철학을 고수한다. 전 세계에 운행 중인 수백만 대의 차량에서 수집되는 방대한 양의 실제 주행 데이터를 통해 AI 모델을 직접 훈련시키는 것이 가장 효과적인 방법이라고 주장한다.36 이는 실제 세계의 '무작위성' 속에서만 나타나는 진정한 '현실성'을 대량의 데이터로 포착하려는 전략이다.
  - **기술 스택**: 실제 주행 차량 플릿(데이터 수집) + Dojo 슈퍼컴퓨터(대규모 훈련) + 자체 FSD 칩(엣지 추론)으로 구성된 데이터 중심의 폐쇄 루프 생태계를 구축했다.38
  - **장점 및 단점**: 이 접근법은 시뮬레이션이 놓칠 수 있는 예상 밖의 '긴 꼬리(long-tail)' 이벤트를 학습할 기회를 제공하지만, 수집된 데이터의 대부분이 반복적이고 단조로운 주행이라 유용한 엣지 케이스를 필터링하기 어렵고, 이를 처리하기 위한 막대한 컴퓨팅 자원이 필요하다는 단점이 있다.37

장기적으로는 두 접근법이 서로의 장점을 흡수하며 수렴할 가능성이 높다. Tesla는 데이터 필터링 및 엣지 케이스 생성을 위해 시뮬레이션을 더 적극적으로 활용하게 될 것이고, NVIDIA는 실제 데이터를 더 효과적으로 시뮬레이션에 통합하여 현실성을 높이는 방향으로 발전할 것이다.


AI 연구의 선두주자인 Google DeepMind 역시 강력한 월드 모델을 개발하고 있으며, 이는 NVIDIA Cosmos와 흥미로운 기술적 대비를 이룬다.

- **NVIDIA Cosmos Transfer**: 이 모델의 핵심은 '변환(Transformation)'이다. 기존의 시뮬레이션 데이터나 실제 데이터를 입력받아, 그 구조적, 의미론적 핵심은 유지하면서 스타일(날씨, 조명, 재질 등)을 바꾸는 'World-to-World' 전이 모델이다. ControlNet과 시공간 제어 맵을 통해 매우 높은 수준의 제어 가능성을 제공하는 것이 특징이다.15
- **Google DeepMind Genie 3**: 이 모델의 핵심은 '생성(Generation)'이다. 텍스트 프롬프트나 단일 이미지로부터 상호작용 가능한 가상 세계 자체를 처음부터 창발적으로 생성한다.39 이는 정밀한 제어보다는 동적이고 창의적인 환경 생성에 초점을 맞추고 있으며, AI 에이전트를 위한 무한한 훈련장을 제공하는 것을 목표로 한다.40

흥미롭게도, 두 거대 기술 기업은 치열한 경쟁 속에서도 협력 관계를 유지하고 있다. NVIDIA와 Google DeepMind는 로보틱스 시뮬레이터 MuJoCo와 NVIDIA Omniverse(USD) 간의 데이터 상호운용성을 높이고, 차세대 물리 엔진인 Newton을 공동 개발하는 등 물리 AI 생태계의 기반이 되는 기술 표준화에 협력하고 있다.32 이는 개별 모델의 성능 경쟁을 넘어, 미래 시장의 주도권을 잡기 위해 가장 효율적인 '데이터 플라이휠'을 구축하고 이를 산업 표준으로 만들려는 더 큰 그림의 일부로 해석될 수 있다.


Cosmos Transfer-2와 같은 기술의 등장은 물리 AI 산업에 지대한 영향을 미칠 것으로 예상된다.

- **개발 주기의 혁신적 단축**: 로봇 및 AV 개발 주기를 몇 년에서 몇 달 단위로 단축시킬 잠재력을 가지고 있다. 이미 Agility Robotics, Figure AI, 1X와 같은 유수의 휴머노이드 로봇 기업들과 Foretellix, Uber 등 자율주행 분야의 선도 기업들이 Cosmos 기술을 채택하여 개발 프로세스를 가속화하고 있다.11
- **새로운 시장 창출**: 고품질 합성 데이터의 활용 범위는 AI 훈련을 넘어, 스마트 팩토리의 디지털 트윈, 가상 테스트베드, 건축 및 도시 계획 시뮬레이션 등 다양한 산업 분야로 확장될 것이다.
- **고등 지능 에이전트의 등장**: WFM은 AI 에이전트에게 미래를 예측하고 다양한 행동의 결과를 미리 시뮬레이션할 수 있는 '상상력'을 부여한다. 이는 단순한 인식과 반응을 넘어, 장기적인 계획(planning)과 복잡한 추론(reasoning)이 가능한 고등 지능 에이전트의 등장을 예고하는 중요한 신호다.44



NVIDIA Cosmos Transfer-2는 물리 AI 개발의 핵심 난제인 '데이터 문제'를 해결하기 위한 혁신적인 접근법을 제시하며, 다음과 같은 핵심적인 기여를 했다.

첫째, 최신 생성 AI 기술인 확산 모델과 ControlNet 아키텍처를 비디오 영역으로 성공적으로 확장하여, 다중 모달 입력을 통한 정밀하고 제어 가능한 합성 데이터 생성을 실현했다. 이는 개발자가 원하는 시나리오를 의도대로 구현할 수 있게 함으로써 데이터 생성의 패러다임을 바꿨다.

둘째, 모델 증류 기술을 통해 확산 모델의 고질적인 문제였던 느린 추론 속도를 획기적으로 개선했다. 이를 통해 대규모 합성 데이터 생성을 산업적으로 실용 가능한 수준으로 끌어올려, 기술의 실험실 단계를 넘어 실제 개발 현장에 적용될 수 있는 길을 열었다.

셋째, 프로젝트 GR00T와 자율주행 시뮬레이션 같은 구체적인 적용 사례에서 Sim-to-Real 격차를 실질적으로 해소하고 AI 모델의 성능을 정량적으로 향상시킴을 입증했다. 이는 합성 데이터의 효용성에 대한 오랜 의구심을 해소하고, 그 가치를 명확히 증명한 중요한 성과다.


물리 AI의 발전은 결국 얼마나 현실에 가까운 데이터를, 얼마나 효율적으로, 그리고 얼마나 대규모로 확보할 수 있느냐의 싸움으로 귀결될 것이다. 이 관점에서 Cosmos Transfer-2와 같은 'World-to-World' 변환 기술은 데이터의 양과 질, 다양성을 동시에 확보할 수 있는 강력한 해답을 제시한다.

향후 이러한 기술은 물리 AI 개발의 표준 워크플로우로 자리 잡을 가능성이 매우 높다. 시뮬레이션 환경에서 생성된 하나의 '씨앗' 데이터가 Cosmos Transfer를 통해 수천, 수만 개의 다양한 현실 세계 시나리오로 증폭되고, Cosmos Reason에 의해 그 품질이 검증되는 자동화된 데이터 공장은 물리 AI 개발의 속도와 규모를 이전과는 비교할 수 없는 수준으로 끌어올릴 것이다.

궁극적으로, Cosmos Transfer-2가 이끄는 합성 데이터 혁명은 AI가 가상 세계에서의 무한한 경험을 통해 현실 세계의 복잡성을 마스터하는 과정을 가속화할 것이다. 이는 더 안전하고, 더 지능적이며, 더 유용한 로봇과 자율 시스템을 우리 삶에 구현하는 데 결정적인 기여를 할 것으로 전망된다.


1. arxiv.org, 8월 23, 2025에 액세스, https://arxiv.org/html/2501.03575v1
2. NVIDIA Launches Cosmos World Foundation Model Platform to Accelerate Physical AI Development, 8월 23, 2025에 액세스, https://nvidianews.nvidia.com/news/nvidia-launches-cosmos-world-foundation-model-platform-to-accelerate-physical-ai-development
3. Gen AI-Powered Synthetic Data Generation in Robotics - SoftServe, 8월 23, 2025에 액세스, https://www.softserveinc.com/en-us/blog/ai-powered-synthetic-data-for-robotics
4. NVIDIA Announces Omniverse Replicator Synthetic-Data-Generation Engine for Training AIs, 8월 23, 2025에 액세스, https://nvidianews.nvidia.com/news/nvidia-announces-omniverse-replicator-synthetic-data-generation-engine-for-training-ais
5. Synthetic Data for AI & 3D Simulation Workflows | Use Case - NVIDIA, 8월 23, 2025에 액세스, https://www.nvidia.com/en-us/use-cases/synthetic-data/
6. Synthetic Data in AI: Solving Data Scarcity and Privacy Challenges | by Hybrid Minds, 8월 23, 2025에 액세스, https://medium.com/@hybrid.minds/synthetic-data-in-ai-solving-data-scarcity-and-privacy-challenges-824d8a380c10
7. How Toyota Research Trains Better Computer Vision Models - Parallel Domain, 8월 23, 2025에 액세스, https://paralleldomain.com/how-tri-trains-better-computer-vision-models-with-pd-synthetic-data/
8. Replicator — Omniverse Extensions, 8월 23, 2025에 액세스, https://docs.omniverse.nvidia.com/extensions/latest/ext_replicator.html
9. The Role of Synthetic Data in Robotics: Accelerating Development and Innovation, 8월 23, 2025에 액세스, https://www.researchgate.net/publication/389247595_The_Role_of_Synthetic_Data_in_Robotics_Accelerating_Development_and_Innovation
10. Can Test-Time Scaling Improve World Foundation Model? - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/html/2503.24320v1
11. NVIDIA Announces Major Release of Cosmos World Foundation Models and Physical AI Data Tools, 8월 23, 2025에 액세스, https://nvidianews.nvidia.com/news/nvidia-announces-major-release-of-cosmos-world-foundation-models-and-physical-ai-data-tools
12. An Introduction to NVIDIA Cosmos World Foundation Models S72431 | GTC 2025, 8월 23, 2025에 액세스, https://www.nvidia.com/en-us/on-demand/session/gtc25-s72431/
13. Physical AI with World Foundation Models | NVIDIA Cosmos, 8월 23, 2025에 액세스, https://www.nvidia.com/en-us/ai/cosmos/
14. NVIDIA Cosmos for Developers, 8월 23, 2025에 액세스, https://developer.nvidia.com/cosmos
15. Cosmos-Transfer1 is a world-to-world transfer model designed to bridge the perceptual divide between simulated and real-world environments. - GitHub, 8월 23, 2025에 액세스, https://github.com/nvidia-cosmos/cosmos-transfer1
16. R²D²: Boost Robot Training with World Foundation Models and ..., 8월 23, 2025에 액세스, https://developer.nvidia.com/blog/r2d2-boost-robot-training-with-world-foundation-models-and-workflows-from-nvidia-research/
17. [2501.03575] Cosmos World Foundation Model Platform for Physical AI - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/abs/2501.03575
18. Develop Custom Physical AI Foundation Models with NVIDIA Cosmos Predict-2, 8월 23, 2025에 액세스, https://developer.nvidia.com/blog/develop-custom-physical-ai-foundation-models-with-nvidia-cosmos-predict-2/
19. Curating Synthetic Datasets to Train Physical AI Models with NVIDIA Cosmos Reason, 8월 23, 2025에 액세스, https://developer.nvidia.com/blog/curating-synthetic-datasets-to-train-physical-ai-models-with-nvidia-cosmos-reason/
20. Advancing Physical AI with NVIDIA Cosmos World Foundation Model Platform, 8월 23, 2025에 액세스, https://developer.nvidia.com/blog/advancing-physical-ai-with-nvidia-cosmos-world-foundation-model-platform/
21. Isaac Sim - Robotics Simulation and Synthetic Data Generation - NVIDIA Developer, 8월 23, 2025에 액세스, https://developer.nvidia.com/isaac/sim
22. Cosmos-Transfer1: Conditional World Generation with Adaptive Multimodal Control - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/html/2503.14492v1
23. Cosmos-Transfer1: Conditional World Generation with ... - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/pdf/2503.14492
24. [Literature Review] Cosmos World Foundation Model Platform for Physical AI - Moonlight, 8월 23, 2025에 액세스, https://www.themoonlight.io/en/review/cosmos-world-foundation-model-platform-for-physical-ai
25. NVIDIA Opens Portals to World of Robotics With New Omniverse Libraries, Cosmos Physical AI Models and AI Computing Infrastructure, 8월 23, 2025에 액세스, https://nvidianews.nvidia.com/news/nvidia-opens-portals-to-world-of-robotics-with-new-omniverse-libraries-cosmos-physical-ai-models-and-ai-computing-infrastructure
26. Isaac GR00T - Generalist Robot 00 Technology - NVIDIA Developer, 8월 23, 2025에 액세스, https://developer.nvidia.com/isaac/gr00t
27. Building a Synthetic Motion Generation Pipeline for Humanoid Robot Learning | NVIDIA Technical Blog, 8월 23, 2025에 액세스, https://developer.nvidia.com/blog/building-a-synthetic-motion-generation-pipeline-for-humanoid-robot-learning/
28. R²D²: Training Generalist Robots with NVIDIA Research Workflows and World Foundation Models, 8월 23, 2025에 액세스, https://developer.nvidia.com/blog/r2d2-training-generalist-robots-with-nvidia-research-workflows-and-world-foundation-models/
29. Synthetic Data in AI: Challenges, Applications, and Ethical Implications - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/html/2401.01629v1
30. nvidia/Cosmos-1.0-Autoregressive-12B - Hugging Face, 8월 23, 2025에 액세스, https://huggingface.co/nvidia/Cosmos-1.0-Autoregressive-12B
31. NVIDIA Announces Isaac GR00T N1 — the World's First Open Humanoid Robot Foundation Model — and Simulation Frameworks to Speed Robot Development, 8월 23, 2025에 액세스, https://nvidianews.nvidia.com/news/nvidia-isaac-gr00t-n1-open-humanoid-robot-foundation-model-simulation-frameworks
32. Announcing Newton, an Open-Source Physics Engine for Robotics Simulation | NVIDIA Technical Blog, 8월 23, 2025에 액세스, https://developer.nvidia.com/blog/announcing-newton-an-open-source-physics-engine-for-robotics-simulation/
33. Why Synthetic Data Is Shaping the Future of Computer Vision - Symage, 8월 23, 2025에 액세스, https://www.symage.ai/synthetic-data-the-future-of-computer-vision/
34. Nvidia Cosmos Explained - NVIDIA's Secret Plan to Build the Real ..., 8월 23, 2025에 액세스, https://www.youtube.com/watch?v=zCq3zwf8GF4
35. Autonomous Vehicle & Self-Driving Car Technology from NVIDIA, 8월 23, 2025에 액세스, https://www.nvidia.com/en-us/solutions/autonomous-vehicles/
36. Nvidia's Chip: A Tesla Killer? - YouTube, 8월 23, 2025에 액세스, https://www.youtube.com/watch?v=-SwKIA0up0U
37. Question about Tesla's advantage of scraping data from millions of vehicles / using footage from millions of cars to teach AI. : r/SelfDrivingCars - Reddit, 8월 23, 2025에 액세스, https://www.reddit.com/r/SelfDrivingCars/comments/1c7tvgc/question_about_teslas_advantage_of_scraping_data/
38. Unlocking Tesla's AI Mojo… Enter the Dojo: Upgrade to OW, PT ..., 8월 23, 2025에 액세스, [https://files-scs.pstatic.net/2023/09/17/ejVf4mNbyb/%EB%AA%A8%EA%B1%B4%20%EC%8A%A4%ED%85%90%EB%A6%AC%20-%20%ED%85%8C%EC%8A%AC%EB%9D%BC%20%EB%8F%84%EC%A1%B0%20%EB%A0%88%ED%8F%AC%ED%8A%B8_230912_174341.pdf](https://files-scs.pstatic.net/2023/09/17/ejVf4mNbyb/모건 스텐리 - 테슬라 도조 레포트_230912_174341.pdf)
39. World Foundation Models: 10 Use Cases & Examples [2025] - Research AIMultiple, 8월 23, 2025에 액세스, https://research.aimultiple.com/world-foundation-model/
40. Genie 3: A new frontier for world models - Google DeepMind, 8월 23, 2025에 액세스, https://deepmind.google/discover/blog/genie-3-a-new-frontier-for-world-models/
41. Google's Genie model makes realistic worlds in realtime… - YouTube, 8월 23, 2025에 액세스, https://www.youtube.com/watch?v=0XvOOi6g5Ok
42. Developers Build Fast and Reliable Robot Simulations with NVIDIA Omniverse Libraries, 8월 23, 2025에 액세스, https://developer.nvidia.com/blog/developers-build-fast-and-reliable-robot-simulations-with-nvidia-omniverse-libraries/
43. NVIDIA Launches Cosmos AI Models for Physical World Generation | NVDA Stock News, 8월 23, 2025에 액세스, https://www.stocktitan.net/news/NVDA/nvidia-announces-major-release-of-cosmos-world-foundation-models-and-45pq7l6hkmio.html
44. NVIDIA Cosmos: A World Foundation Model Platform for Physical AI - YouTube, 8월 23, 2025에 액세스, https://www.youtube.com/watch?v=9Uch931cDx8


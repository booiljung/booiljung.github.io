# 픽셀의 다리: 로보틱스와 AI의 Sim-to-Real 전환에 대한 종합 안내서

## 1.  디지털 트윈의 꿈: Sim-to-Real 입문

### 1.1  Sim-to-Real 전환이란 무엇인가? 기본 정의

Sim-to-Real(Sim2Real) 전환은 시뮬레이션된 환경에서 인공지능(AI) 에이전트가 학습한 지식, 기술, 또는 '정책'을 실제 세계에 성공적으로 적용하는 과정을 의미한다.1 이는 마치 매우 현실적인 비행 시뮬레이터에서 조종사가 수천 번의 이착륙과 비상 절차를 안전하게 연습하는 것과 유사하다. Sim2Real의 목표는 가상 조종석에서 배운 기술이 실제 보잉 747기에서 완벽하게 작동하도록 보장하는 것이다.4 이 과정은 로보틱스와 자율 시스템 같은 현대 AI 분야의 핵심이며, 지능형 에이전트 개발을 가능하게 하는 중요한 열쇠이다.6

### 1.2  시뮬레이터의 약속: 왜 현실 세계에서 직접 훈련하지 않는가?

복잡한 AI 개발에 있어 시뮬레이션 기반 훈련이 거의 필수적인 이유를 설명하는 설득력 있는 장점들은 다음과 같다.

- **비용 효율성:** 물리적 로봇을 제작하고 수리하는 데는 많은 비용이 든다. 시뮬레이션은 재료비, 하드웨어 수정, 마모에 따른 비용을 없애준다.8 시뮬레이션에서의 실수는 비용이 들지 않지만, 현실 세계에서는 수백만 달러짜리 프로토타입의 파괴를 의미할 수 있다.
- **안전성 강화:** 로봇에게 섬세한 수술을 가르치거나 자율주행차가 타이어 펑크에 대처하도록 훈련시키는 것은 본질적으로 위험하다. 시뮬레이션은 AI가 자신이나 주변 환경, 인간에게 실제적인 피해 없이 실패하고 배울 수 있는 위험 없는 '시험장'을 제공한다.7
- **전례 없는 데이터 효율성 및 속도:** 가장 큰 장점은 방대한 양의 훈련 데이터를 생성할 수 있다는 것이다. 강화학습(RL) 알고리즘은 데이터에 매우 의존하며, 종종 수백만에서 수십억 번의 시도가 필요하다.14 시뮬레이션은 실시간보다 빠르게 실행될 수 있으며 수천 대의 컴퓨터에 병렬로 분산되어 수백 년의 경험을 며칠 만에 압축할 수 있다.14 예를 들어, OpenAI의 Dactyl 손은 100년 분량의 시뮬레이션 경험이 필요했으며 19, MIT 연구진은 3시간의 시뮬레이션이 100일간의 실제 데이터 수집과 맞먹는다고 언급했다.18
- **확장성 및 유연성:** 시뮬레이터를 통해 개발자들은 다양한 조명과 날씨 조건부터 화성 지형과 같이 지구상에서 재현 불가능한 환경에 이르기까지 광범위한 조건에서 로봇을 테스트할 수 있다.8

### 1.3  거대한 도전: '현실 격차'의 등장

시뮬레이션의 엄청난 힘에도 불구하고 한 가지 문제가 있다. 순수하게 시뮬레이터에서만 훈련된 모델은 물리적 로봇에 적용될 때 종종 실패한다는 것이다.14 시뮬레이션된 세계와 현실 세계 사이의 이러한 불일치는 

**'현실 격차(Reality Gap)'** 또는 'Sim2Real 격차'로 알려져 있다.2

비행 시뮬레이터 비유를 확장해 보자면, 만약 시뮬레이터가 실제 비행기의 측풍이나 특정 레버의 미세한 뻑뻑함을 완벽하게 모델링하지 못했다면 어떨까? 그 결함 있는 시뮬레이터에서 훈련받은 조종사는 실제 세계에서 치명적인 실수를 저지를 수 있다. 이것이 바로 현실 격차의 본질이다.

이 보고서의 나머지 부분은 이 격차를 이해하고 극복하기 위한 여정이며, 연구자들이 성공적인 전환을 위해 개발한 독창적인 방법들을 탐구하는 과정이 될 것이다.21 이 문제의 핵심은 시뮬레이터가 현실의 불완전한 복제품이라는 단순한 사실을 넘어선다. AI는 현실의 법칙이 아닌 

*시뮬레이션의 법칙*을 학습할 위험이 있다.25 AI가 시뮬레이터의 결함을 악용하여 작업을 수행하는 법을 배우게 되면 17, 시뮬레이션은 그 자체로 하나의 현실이 되며, 그 안에서만 통용되는 물리 법칙을 갖게 된다. 따라서 Sim2Real 문제 해결은 단순히 시뮬레이터를 더 사실적으로 만드는 기술적 도전을 넘어, AI가 결함 있는 표현을 통해 세상의 근본 원리를 배우도록 보장하는 개념적 도전이기도 하다.

## 2.  물리학의 불쾌한 골짜기: 현실 격차 해부

이 섹션에서는 '현실 격차'라는 추상적인 개념을 구체적인 원인과 결과 분석으로 분해하여, 시뮬레이션된 세계가 왜 현실 세계가 아닌지를 명확히 설명한다.

### 2.1  동역학적 불일치: 물리 엔진이 거짓말할 때

이 하위 섹션은 물리 법칙과 상호작용의 차이점에 초점을 맞춘다.

- **접촉과 마찰:** 물체가 어떻게 접촉하고, 미끄러지며, 달라붙는지를 모델링하는 것은 매우 어렵다. 부정확한 마찰 계수나 접촉 동역학은 깨끗한 시뮬레이션 환경에서 완벽하게 작동하던 파지 정책이 실제 물체의 특정 질감과 재료 특성에 직면했을 때 실패하는 주된 원인이다.2 이는 특히 복잡하고 접촉이 많은 조작 작업에서 두드러진다.29
- **액추에이터 및 구동계 동역학:** 시뮬레이터는 종종 모터, 기어, 관절에 대해 단순화된 모델을 사용한다. 이 모델들은 비선형 효과, 기어의 백래시, 제어 지연, 또는 실제 로봇 하드웨어의 정확한 토크 특성을 포착하지 못할 수 있다.26 이로 인해 정책이 실제 로봇이 동일한 정밀도로 물리적으로 실행할 수 없는 움직임을 명령하게 될 수 있다.
- **시스템 매개변수:** 질량, 관성, 링크 길이와 같은 단순한 속성들은 모델링하기 쉬워 보이지만, 시뮬레이터가 놓치는 미묘한 실제 세계의 변형이 있어 예상치 못한 로봇 행동을 유발할 수 있다.26
- **변형 가능 및 유체 객체:** 천이나 음식과 같은 부드럽고 변형 가능한 물체나 유체의 물리학은 시뮬레이션에서 여전히 미해결 연구 문제로 남아 있어, 이러한 작업을 위한 Sim2Real은 특히 어렵다.32

### 2.2  지각적 불일치: 가상 눈이 속일 때

이 하위 섹션은 로봇이 환경을 *인식*하는 방식의 차이점에 초점을 맞춘다. 이는 비전 기반 로봇에게 있어 "현실 격차의 가장 넓은 부분"을 차지한다.17

- **시각적 렌더링 충실도:** 시뮬레이터는 현실 세계의 조명, 그림자, 반사, 질감의 복잡성을 완벽하게 재현하는 데 어려움을 겪는다.2 깨끗하고 합성된 이미지로 훈련된 신경망은 창문에서의 눈부심이나 예상치 못한 그림자와 같은 현실의 시각적 '노이즈'에 쉽게 혼란을 겪을 수 있다.35
- **센서 노이즈 및 아티팩트:** 실제 센서(카메라, 라이다, 깊이 센서)는 본질적으로 노이즈가 있다. 이들은 완벽하게 모델링하기 어려운 결함, 왜곡, 데이터 손실이 있는 데이터를 생성한다.23 예를 들어, 실제 깊이 카메라는 반사 표면으로 인해 왜곡된 포인트 클라우드를 생성하거나 데이터가 누락될 수 있다.35
- **도메인 이동(Domain Shift):** 이는 시뮬레이션(소스 도메인)과 현실 세계(타겟 도메인) 간의 데이터 분포 차이를 나타내는 통계 용어이다.2 AI 모델은 패턴을 찾는 데 매우 뛰어나며, 시뮬레이션 데이터의 패턴(예: 특정 텍스처 세트)이 현실 세계에 존재하지 않으면 모델의 성능은 크게 저하된다.39

### 2.3  그 결과: 현실 세계 실패 사례 모음

이 부분은 현실 격차가 극복되지 않았을 때 발생하는 구체적인 사례들을 제공한다.

- 시뮬레이터에서 음료를 따르도록 배운 로봇은 현실 세계에서 지속적으로 컵을 놓치거나 잘못된 시간 동안 따를 수 있다.41
- 시뮬레이션에서 완벽하게 걷던 다리 달린 로봇은 마찰이 약간 다른 실제 표면에서 불안정해져 넘어질 수 있다.26
- 강력한 딥러닝 모델이 현실에는 존재하지 않는 물리 엔진 버그를 악용하여 '속임수'를 쓰는 법을 배워 파지 정책이 실패할 수 있다.17
- 훈련 시뮬레이션에서는 항상 열려 있던 문이 현실 세계에서는 닫혀 있어 모델이 처리할 수 없는 시각적 신호가 되어 자기 위치 파악 로봇이 실패할 수 있다.42
- 정책들은 신경망의 활성화 함수 선택과 같은 미묘한 요인으로 인해 시뮬레이션에서 동일하게 수행되더라도 현실 세계에서는 매우 다른 성능을 보일 수 있다.43

현실 격차는 정적인 문제가 아니라 적대적인 문제이다. 우리의 학습 알고리즘이 강력해질수록, 그들은 격차를 더 잘 악용하게 되어 문제를 더 어렵게 만든다. 단순한 선형 제어기는 물리 모델의 사소한 오류에 덜 민감할 수 있다. 그러나 현대의 심층 강화학습(RL) 알고리즘은 어떤 수단을 동원해서든 보상을 극대화하려는 강력한 비선형 함수 근사기이다.17 이는 물리 시뮬레이터에 버그, 부정확성, 또는 악용할 수 있는 허점이 있다면 RL 에이전트가 이를 

*반드시* 찾아내어 목표 달성을 위해 악용할 것임을 의미한다. 이것이 "시뮬레이터를 속인다"는 말의 의미이다.17 이는 더 나은 RL 알고리즘을 개발함에 따라, 시뮬레이션과 현실의 차이점을 찾아내고 활용하는 데 더 능숙한 에이전트를 동시에 만들어내는 역설적인 상황을 초래한다. 학습된 정책은 작업에 대한 일반적인 해결책이 아니라, 모든 결함을 포함한 

*시뮬레이션된 버전*의 작업에 대한 고도로 전문화된 해결책이 된다. 따라서 도전 과제는 단순히 더 나은 시뮬레이터를 만드는 것이 아니라, 시뮬레이션과 현실 사이의 차이를 적극적으로 찾아 활용하려는 지능적인 적(RL 에이전트 자체)에 강인한 훈련 과정을 구축하는 것이다.

### 2.4 표 1: 현실 격차의 해부학

| 격차 범주  | 구체적 원인                 | 실제 실패 사례                                               |
| ---------- | --------------------------- | ------------------------------------------------------------ |
| **동역학** | 부정확한 마찰 모델          | 시뮬레이션에서는 잡을 수 있었던 '미끄러운' 물체를 로봇이 떨어뜨림 |
|            | 액추에이터 응답 지연        | 로봇이 관성 때문에 목표 지점을 지나침                        |
|            | 부정확한 접촉 모델링        | 접촉이 많은 조립 작업 중 부품을 잘못 정렬함                  |
| **인식**   | 비현실적인 조명/텍스처      | 저조도 환경에서 비전 시스템이 물체를 식별하지 못함           |
|            | 모델링되지 않은 센서 노이즈 | 센서 데이터의 노이즈로 인해 정책이 불안정해지고 진동함       |
|            | 시각적 단순화               | 시뮬레이션에는 없던 배경의 어수선함 때문에 물체 감지에 실패함 |
| **시스템** | 시뮬레이터 버그 악용        | 정책이 물리 엔진의 허점을 이용해 비현실적인 방식으로 작업을 완료함 |
|            | 제어기 불일치               | 시뮬레이션된 PID 제어기와 실제 제어기의 차이로 인해 로봇이 과도하게 움직임 |

## 3.  다리 놓기: Sim2Real 기술 개요

이 섹션은 보고서의 기술적 핵심이다. 가장 직관적인 것부터 가장 정교한 것까지, 현실 격차를 극복하기 위해 개발된 주요 해결책들을 체계적으로 설명한다

### 3.1  시뮬레이션의 현실화: 시스템 식별 및 매개변수 튜닝

- **철학:** "시뮬레이션이 틀렸다면, 고쳐라." 이는 시뮬레이션을 현실의 더 충실한 복제품으로 만드는 가장 직접적인 접근 방식이다.
- **방법론:** **시스템 식별(System Identification, SysID)**이라는 과정을 포함한다. 엔지니어들은 실제 로봇으로부터 데이터를 수집하고(예: 로봇에게 움직임을 명령하고 그 반응을 측정) 이 데이터를 사용하여 시뮬레이터의 물리 엔진 매개변수를 조정한다.28
  - 이는 질량, 관성, 마찰, 중력 보상 항과 같은 물리적 매개변수를 추정하는 것을 포함할 수 있다.45
  - 또한 시뮬레이션된 액추에이터가 실제 액추에이터처럼 작동하도록 PID 제어기와 같은 로봇의 제어기를 모델링하는 것도 포함될 수 있다.31
  - 고급 방법들은 이 튜닝 과정을 자동화하기 위해 머신러닝을 사용하며, 때로는 "real-to-sim"이라고도 불린다.46 예를 들어, TuneNet은 두 궤적 사이의 물리 매개변수 차이를 예측하는 법을 학습하여 빠르고 한 번에 튜닝을 가능하게 한다.48
- **장점:** 특정 하드웨어에 대해 매우 정확한 시뮬레이터를 만들 수 있어 모델 기반 제어에 유용하다.
- **단점:** 데이터 수집을 위해 실제 로봇에 접근해야 하고, 노동 집약적일 수 있으며, 실제 세계의 물리학이 시뮬레이터의 매개변수 집합으로 실제로 포착될 수 있다고 가정하지만, 매우 복잡한 현상에 대해서는 그렇지 않은 경우가 많다.3

### 3.2  혼돈의 수용: 도메인 무작위화 (Domain Randomization, DR)

- **철학:** "현실을 완벽하게 모델링하려고 하지 마라. 대신, 정책을 매우 강인하게 만들어 현실이 이미 본 변형 중 하나처럼 보이게 하라." 이것은 매우 중요하고 직관에 반하는 아이디어이다.
- **방법론:** 하나의 "최선의 추측" 시뮬레이션에서 훈련하는 대신, 에이전트는 매개변수가 지속적으로 무작위화되는 거대한 다양성의 시뮬레이션에서 훈련된다.2
  - **동역학 무작위화:** 마찰, 물체의 질량, 중력, 액추에이터 힘과 같은 물리적 매개변수가 각 훈련 에피소드마다 변경된다.2
  - **시각적 무작위화:** 조명 조건, 물체 및 배경의 텍스처, 카메라 위치, 장면의 물체 수와 같은 시각적 속성도 무작위화된다.32
  - 핵심 통찰은 충분한 가변성을 통해 신경망이 작업의 본질적이고 인과적인 특징만을 배우고 단일 시뮬레이션 인스턴스와 관련된 허위 상관관계를 무시하도록 강제된다는 것이다.50
- **자동 도메인 무작위화 (Automatic Domain Randomization, ADR):** OpenAI의 핵심 혁신이다. 무작위화 범위를 수동으로 설정하는 대신, ADR은 무작위화 없이 시작하여 정책이 향상됨에 따라 자동으로 난이도(환경의 "엔트로피")를 증가시킨다.53 이는 계속해서 더 어려운 문제의 커리큘럼을 만들어, 정책이 환경을 완전히 "해결"하는 것을 방지하고 점점 더 일반화되도록 강제한다.
- **장점:** 매우 효과적이며, "제로샷(zero-shot)" 전송(실제 세계 미세 조정 없이 실제 세계에서 작동)을 가능하게 할 수 있으며, 반드시 세계의 매우 정확한 모델을 필요로 하지 않는다.6
- **단점:** 계산 비용이 매우 비쌀 수 있으며, 무작위화 범위가 너무 넓거나 잘 선택되지 않으면 학습 문제를 너무 어렵게 만들어 정책이 너무 보수적이 되거나 학습에 실패할 수 있다.3

### 3.3  속임수의 기술: 생성 모델을 통한 도메인 적응 (Domain Adaptation, DA)

- **철학:** "시뮬레이션된 *데이터*가 실제 데이터와 다르게 보인다면, AI를 사용하여 똑같이 보이게 만들어라." 이 접근 방식은 주로 지각적 격차를 해소하는 데 중점을 둔다.
- **방법론:** 이는 **생성적 적대 신경망(Generative Adversarial Networks, GANs)**을 많이 포함하는데, 이는 가짜 이미지를 만드는 생성자(Generator)와 가짜 이미지를 실제 이미지와 구별하려는 판별자(Discriminator)라는 두 개의 경쟁하는 신경망으로 구성된다.49
  - **이미지 대 이미지 변환:** **CycleGAN**과 같은 기술은 완벽하게 쌍을 이루는 이미지가 없어도 합성 이미지를 사실적으로 보이도록 "재스타일링"하여 시뮬레이션 도메인에서 실제 도메인으로의 변환을 학습하는 데 사용된다.56 그런 다음 정책은 이러한 더 현실적인 "유사-실제" 이미지로 훈련된다.
  - **로봇 인식 적응:** 기본 GAN의 문제점은 로봇 그리퍼의 위치와 같은 작업 관련 정보를 우연히 변경할 수 있다는 것이다. 고급 방법들은 이를 방지하기 위해 제약 조건을 추가한다.
    - **RL-CycleGAN**은 로봇의 행동이 원본 이미지와 변환된 이미지 모두에 대해 동일하도록 보장하는 "일관성 손실"을 추가한다.56
    - **RetinaGAN**은 객체 감지기를 사용하여 변환 전후에 객체가 동일한 위치에 있도록 보장하여 중요한 의미 정보를 보존한다.56
  - **Sim-to-Sim 적응:** 독창적인 전환은 실제 데이터를 전혀 사용하지 않는 것이다. **RCANs (Randomized-to-Canonical Adaptation Networks)**는 *무작위화된* 시뮬레이션 이미지를 표준, 비무작위화("정규") 버전으로 다시 변환하는 법을 학습한다.58 테스트 시, 실제 세계 이미지가 이 네트워크에 입력되어 현실의 "노이즈"를 제거하고 정책이 훈련된 깨끗하고 정규적인 이미지를 생성한다.
- **장점:** 시각적 현실 격차를 극적으로 줄일 수 있으며, 종종 전체 미세 조정보다 적은 실제 세계 데이터(또는 RCAN의 경우 전혀 없음)를 필요로 한다.
- **단점:** 훈련이 복잡할 수 있으며, 시각적 아티팩트를 도입할 수 있다. 주로 인식에 초점을 맞추므로 동역학 격차를 직접 해결하지는 않는다.

### 3.4  순환 고리 닫기: 하이브리드 및 상호작용 프레임워크

- **철학:** "왜 한 가지 방법만 선택하는가? 그것들을 결합하고 실제 세계로부터의 피드백을 사용하라." 이것들은 단 하나의 만병통치약이 없다는 것을 인식하는 최첨단 접근 방식이다.
- **방법론:**
  - **Real-to-Sim-to-Real:** 이 파이프라인은 시작과 끝 모두에서 실제 세계를 사용한다. 먼저, 실제 세계 데이터(예: 휴대폰 카메라 스캔)를 사용하여 환경의 고충실도 "디지털 트윈"을 만든다.18 그런 다음, RL과 같은 기술을 사용하여 이 매우 정확한 시뮬레이션에서 정책을 훈련시킨다. 마지막으로, 훈련된 정책을 작업을 위해 다시 실제 세계에 배포한다. MIT의 RialTo 시스템이 대표적인 예이다.18
  - **인간 참여형 학습(Human-in-the-Loop Learning):** 이 접근 방식은 격차를 해소하기 위해 인간의 지능을 직접 활용한다. 시뮬레이션에서 훈련된 정책이 실제 로봇에 배포된다. 로봇이 막히거나 실수를 하면, 인간 운영자가 원격 조작을 통해 행동을 수정한다.15 시스템은 이 인간 수정 데이터를 수집하여 시뮬레이션 정책의 오류를 수정하는 법을 배우는 "잔여(residual)" 정책을 훈련시키는 데 사용하며, 모든 현실 격차의 원인을 한 번에 전체적으로 해결한다.15
  - **모방학습(IL)과 강화학습(RL)의 결합:** 일부 프레임워크는 전문가 시연으로부터 모방학습(IL)을 사용하여 좋은 시작 정책을 얻은 다음, 무작위화되거나 적응된 시뮬레이션에서 강화학습(RL)을 사용하여 그 정책을 개선하고 강인하게 만든다.3

이러한 Sim2Real 전략들은 상호 배타적인 경쟁자가 아니라, 더 크고 정교한 파이프라인에서 상호 보완적인 모듈로 점점 더 간주되고 있다. 초기에는 시스템 식별과 도메인 무작위화가 동일한 문제에 대한 별개의 접근법으로 여겨졌다.49 그러나 OpenAI의 Dactyl은 MuJoCo 매개변수에 대한 합리적인 기준선을 얻기 위해 시스템 식별을 사용한 다음, 그 위에 대규모 도메인 무작위화를 적용했다.60 이는 시스템 식별과 도메인 무작위화의 결합이다. RCAN 논문은 도메인 적응이 도메인 무작위화와 새로운 방식으로 결합될 수 있음을 보여준다.58 가장 진보된 프레임워크인 MIT의 RialTo 33와 인간 참여형 학습 15은 명시적으로 다단계 파이프라인으로 설계되었다. 이는 이 분야의 성숙을 보여준다. 문제는 이제 "어떤 방법이 최고인가?"가 아니라 "이 작업을 위해 방법들의 최적 순서와 조합은 무엇인가?"로 바뀌고 있다.

### 3.5 표 2: Sim2Real 전략 비교 분석

| 기술                | 핵심 철학         | 실제 데이터 요구사항           | 주요 해결 격차 | 핵심 장점                          | 핵심 한계                                 |
| ------------------- | ----------------- | ------------------------------ | -------------- | ---------------------------------- | ----------------------------------------- |
| **시스템 식별**     | "시뮬레이션 수정" | 높음 (튜닝용)                  | 동역학         | 알려진 시스템에 대한 높은 충실도   | 노동 집약적, 실제 로봇 필요               |
| **도메인 무작위화** | "정책 강인화"     | 없음 (제로샷)                  | 인식 및 동역학 | 뛰어난 일반화, 제로샷 전송 가능    | 계산 비용이 높음, 학습이 어려워질 수 있음 |
| **도메인 적응**     | "데이터 수정"     | 낮음-중간 (GAN 훈련/미세 조정) | 인식           | 시각적 격차 해소, 적은 데이터 필요 | 동역학을 해결하지 못함, 훈련 복잡성       |
| **하이브리드 방법** | "모든 정보 활용"  | 다양함                         | 전체적         | 각 방법의 장점 결합, 높은 성능     | 시스템 복잡성 매우 높음                   |

## 4.  픽셀에서 포장도로까지: 실제 세계의 Sim2Real

이 섹션은 이론에서 실천으로 넘어가, 이러한 기술들이 로보틱스와 AI 분야에서 획기적인 성과를 어떻게 가능하게 했는지를 보여준다. 각 사례 연구는 문제, 해결책, 그리고 그 중요성을 설명하는 이야기 형식으로 구성된다.

### 4.1  사례 연구: 능숙한 손 (OpenAI의 Dactyl)

- **도전 과제:** 복잡한 다중 손가락 로봇 손(Shadow Dexterous Hand)에게 블록 방향 바꾸기 19 및 나중에는 루빅스 큐브 맞추기 53와 같이 인간 수준의 손재주가 필요한 작업을 가르치는 것. 이는 믿을 수 없을 정도로 복잡한 접촉 물리학 때문에 기념비적인 도전이다.

- **해결책:**

  - 정책은 대규모 강화학습(PPO 알고리즘)을 사용하여 **전적으로 시뮬레이션에서** 훈련되었다.19
  - 핵심 기술은 **자동 도메인 무작위화(ADR)**였다. 시스템은 큐브의 크기, 질량, 마찰, 로봇 손가락의 마찰, 시각적 표면 속성 등 수십 개의 매개변수를 무작위화하여 시뮬레이션을 자동으로 그리고 지속적으로 더 어렵게 만들었다.53
  - 정책은 픽셀에서 직접 종단 간(end-to-end)으로 훈련되지 않았다. 별도의 비전 네트워크가 큐브의 자세를 추정했고, 이 자세와 손가락 끝 좌표가 제어 정책에 입력되었다.19 이러한 모듈성은 학습 문제를 단순화했다.

- **돌파구:** Dactyl은 실제 세계로의 제로샷 전송을 달성하여, 박제 기린으로 찌르는 것과 같은 방해에도 불구하고 블록과 큐브를 성공적으로 조작했다.53 이는 순수하게 시뮬레이션으로 훈련된 정책이 전례 없는 복잡성과 손재주를 요구하는 실제 세계 문제를 해결할 수 있음을 증명했다. 핵심 발견은 LSTM(메모리 증강) 정책이 경험을 통해 현재 환경의 물리학을 

  *추론*하고 그에 따라 전략을 조정하는 법을 학습할 수 있다는 것, 즉 메타 학습의 발현이었다.53

### 4.2  사례 연구: 모든 것을 보는 자동차 (Waymo와 NVIDIA)

- **도전 과제:** 자율주행차(AV)가 사실상 무한한 수의 주행 시나리오, 특히 실제 테스트에서 철저하게 마주치기 불가능한 드물고 위험한 "롱테일" 이벤트 전반에 걸쳐 안전함을 보장하는 것.61

- **Waymo의 접근 방식: SimulationCity**

  - Waymo는 2,000만 마일 이상의 실제 주행 데이터를 활용하여 SimulationCity라는 초현실적이고 지속적으로 업데이트되는 시뮬레이션 환경을 구축한다.36 이는 

    **Real-to-Sim** 파이프라인의 대표적인 예이다.

  - 그들은 실제 치명적인 충돌 사고를 재현하고 변경하여 Waymo 드라이버가 어떻게 행동했을지 확인할 수 있다. 센서에 맺힌 빗방울부터 태양광 반사까지 모든 것을 시뮬레이션하여 완전히 합성된 여정을 만들 수 있다.36

  - 새로운 소프트웨어는 물리적 차량에 배포되기 전에 SimulationCity에서 엄격하게 검증되어 확장 가능하고 재현 가능한 평가 도구 역할을 한다.36

- **NVIDIA의 접근 방식: DRIVE Sim 및 신경 재구성 엔진**

  - NVIDIA는 AV 개발을 위한 플랫폼인 DRIVE Sim을 제공한다.63 핵심 혁신은 실제 주행에서 얻은 2D 비디오를 상호작용 가능한 3D 시뮬레이션 장면으로 자동 변환하는 AI인 **신경 재구성 엔진(Neural Reconstruction Engine)**이다.65
  - 이는 고충실도 "디지털 트윈" 생성을 자동화하며, 이는 새로운 행위자를 추가하거나, 날씨를 바꾸거나, 이벤트를 변경하는 등 새로운 훈련 및 테스트 시나리오를 만들기 위해 조작될 수 있다. 이는 실제 데이터로부터 직접 시뮬레이션을 구축함으로써 sim-to-real 격차를 극적으로 줄인다.65

- **돌파구:** AV의 경우, Sim2Real은 초기 훈련뿐만 아니라 검증, 테스트 및 엣지 케이스 처리를 위한 중요하고 지속적인 도구이다. 이 접근 방식은 단일 정책의 제로샷 전송보다는 실제 데이터가 시뮬레이션을 개선하고, 개선된 시뮬레이션이 실제 시스템을 검증하고 강화하는 지속적인 순환 고리를 만드는 것에 가깝다.

### 4.3  사례 연구: 민첩한 드론 (자율 드론 레이싱)

- **도전 과제:** 자원이 제한된 소형 드론이 경주 코스를 고속으로 자율 비행하며 게이트를 통과하도록 하는 것. 이는 제한된 온보드 하드웨어에서 극도의 민첩성과 빠른 인식을 요구한다.66
- **해결책:** 모듈식 접근 방식이 사용되었다. 컨볼루션 신경망(CNN)은 드론의 카메라 이미지에서 다음 웨이포인트(다음 게이트의 중심)를 식별하는 한 가지 작업을 수행하도록 비현실적인 시뮬레이터에서 순수하게 훈련되었다.66
  - **도메인 무작위화**가 핵심이었다. 시뮬레이션은 게이트 모양, 조명, 텍스처를 무작위화하여 인식 네트워크를 강인하게 만들었다.66
  - 이 학습된 인식 모듈의 출력(웨이포인트 좌표)은 최적의 궤적과 모터 명령을 계산하는 전통적인 고성능 계획 및 제어 시스템에 입력되었다.66
- **돌파구:** 이 연구는 민첩한 드론 비행을 위한 성공적인 제로샷 sim-to-real 전송을 보여주었다.66 단순한 시뮬레이터에서만 훈련된 드론은 실제 경주 트랙을 성공적으로 주행할 수 있었다. 학습된 인식 프런트엔드와 고전적인 제어 백엔드를 결합한 이 하이브리드 접근 방식은 문제를 학습 가능한 구성 요소와 해결 가능한 구성 요소로 분해함으로써 Sim2Real을 효과적으로 적용할 수 있음을 보여준다. 사용자 경험은 Liftoff와 같은 시뮬레이터의 기술이 잘 이전되지만, 비디오 피드 품질과 충돌의 심리적 스트레스와 같은 실제 요인이 남아있는 격차의 중요한 부분임을 확인시켜 준다.5

이러한 사례들을 통해 Sim2Real의 "성공"이 단일 개념이 아님을 알 수 있다. 성공은 각 응용 분야의 핵심 과제에 따라 다르게 정의된다. 손재주가 필요한 조작에서는 핵심 과제가 **복잡한 동역학**이므로 19, 해결책은 

**동역학 무작위화**에 크게 의존한다.53 자율주행차에서는 핵심 과제가 

**현실 세계의 방대한 다양성과 드문 엣지 케이스**이므로 36, 해결책은 

**고충실도 Real-to-Sim 검증 루프**를 구축하는 것이다.65 드론 레이싱에서는 핵심 과제가 

**자원이 제한된 하드웨어에서의 고속 인식**이므로 66, 해결책은 Sim2Real을 인식 부분에만 적용하고 동역학과 제어는 고전적인 알고리즘에 맡기는 

**모듈식 설계**이다.66 이는 Sim2Real이 단일 문제가 아니라 문제군(family of problems)이며, 기술 선택은 특정 응용 분야의 병목 현상에 의해 결정되는 전략적 결정임을 보여준다.

## 5.  앞으로의 길: 미래의 개척지와 지속적인 과제

이 마지막 섹션에서는 미래를 전망하며, 여전히 어려운 점들을 솔직하게 평가하고 마침내 격차를 해소할 유망한 연구 방향을 탐구한다.

### 5.1  가장 어려운 과제들: 지속적인 도전

- **변형 가능 및 관절형 객체:** 진전이 있었지만 34, 천, 밧줄, 음식과 같은 비강체 객체나 복잡한 관절형 객체를 정확하고 효율적으로 시뮬레이션하는 것은 여전히 주요 장애물이다. 이는 로봇 요리나 빨래 개기와 같은 작업에 대한 Sim2Real을 제한한다.32
- **복잡한 접촉-집중 시나리오:** 가구 조립이나 복잡한 도구 사용과 같이 긴 시퀀스의 정밀한 접촉을 포함하는 작업은 여전히 개척 분야에 있다. 접촉 모델링의 많은 작은 부정확성에서 비롯된 누적 오류는 정책을 빠르게 탈선시킬 수 있다.15
- **샘플 효율성 및 계산 비용:** 광범위한 도메인 무작위화를 사용한 훈련은 엄청난 계산 비용이 들며, 대규모 서버 팜과 긴 훈련 시간이 필요하다.3 이러한 기술을 더 접근하기 쉽고 효율적으로 만드는 것이 핵심 과제이다.
- **모델 기반 딜레마:** 더 높은 샘플 효율성을 약속하는 모델 기반 RL 방법은 본질적으로 현실 격차에 더 민감하다. 학습된 동역학 모델의 오류는 치명적일 수 있으며, 이로 인해 모델 프리 방법에 비해 실제 적용이 더디게 진행되었다.61
- **보장된 이전 가능성:** 성공에도 불구하고, 이전이 항상 보장되는 것은 아니다. 이는 시행착오의 과정일 수 있으며, 시뮬레이션 및 무작위화 매개변수를 올바르게 설정하기 위해 상당한 전문 지식이 필요하다.3

### 5.2  차세대 다리: 미래 연구 방향

- **미분 가능한 시뮬레이터:** 이것은 잠재적인 게임 체인저이다. 미분 가능한 시뮬레이터를 사용하면 오류(현실 격차)를 시뮬레이션 자체를 통해 역전파하여 경사 기반 최적화를 사용하여 시뮬레이터의 매개변수를 직접 업데이트할 수 있다. 이는 시뮬레이터 튜닝을 훨씬 더 효율적이고 정확하게 만들 수 있다.69
- **진정한 디지털 트윈의 부상:** 궁극적인 목표는 실제 시스템의 완벽하게 동기화되고 물리적으로 정확한 실시간 가상 사본을 만들고 유지하는 것이다.33 이는 Real-to-Sim, 시스템 식별, 그리고 연속적인 데이터 스트림의 개념을 통합하여 완벽한 테스트베드를 만든다.
- **시뮬레이션을 위한 AI: 파운데이션 모델과 LLM의 참여:** 시뮬레이션을 구축하고 관리하는 데 AI를 사용하는 추세는 가속화될 것이다.
  - NVIDIA의 신경 재구성 엔진은 실제 데이터로부터 시뮬레이션 콘텐츠를 생성하는 데 AI를 사용하는 선도적인 예이다.65
  - **DrEureka**와 같은 새로운 연구는 Sim2Real 파이프라인의 일부를 자동화하기 위해 대규모 언어 모델(LLM)을 사용한다. LLM은 효과적인 보상 함수를 생성하거나 도메인 무작위화 매개변수를 제안하도록 프롬프트될 수 있으며, 물리학과 언어에 대한 방대한 지식을 활용하여 지루한 인간 엔지니어링을 대체한다.70
- **하이브리드 물리 모델:** 미래의 시뮬레이터는 순수하게 수작업으로 코딩된 물리학이나 순수하게 학습된 것에 기반하지 않을 것이다. 그들은 전통적인 물리 엔진을 강력한 사전 지식으로 사용하고, 물리 엔진이 잘못 이해하는 복잡하고 모델링되지 않은 동역학, 즉 "잔여(residual)"를 모델링하기 위해 학습된 신경망 구성 요소를 사용하는 하이브리드 시스템이 될 것이다.22

### 5.3  결론: 격차 없는 세상을 향하여

이 보고서는 시뮬레이션의 꿈을 정의하는 것부터 현실 격차를 해부하고 그것을 극복하기 위해 설계된 기술들의 무기고를 탐험하는 여정을 요약한다. Sim2Real 전환은 안전하고, 유능하며, 확장 가능한 로봇 및 자율 시스템 개발을 가능하게 하는 AI의 미래를 위한 초석 기술임을 재확인한다.

궁극적인 목표는 Sim2Real "문제" 자체를 사라지게 하는 것이다. 즉, 가상 훈련장에서 실제 배포로의 전환이 너무나 원활하고 신뢰할 수 있어 개발 파이프라인에서 단순히 표준적이고 예상되는 단계가 되는 지점에 도달하는 것이다. 현재의 최첨단 기술은 여전히 상당한 인간의 전문 지식과 수동적 노력을 필요로 하지만 68, 미래는 현실 격차를 식별하고 메우는 과정이 AI 자체에 의해 점점 더 자동화되는 "자기 폐쇄 루프(self-closing loop)"를 향해 나아가고 있다. 자동 도메인 무작위화(ADR) 53, DrEureka 70, 신경 재구성 엔진 65과 같은 자동화된 구성 요소들을 단일 폐쇄 루프로 통합하는 것을 상상해 보라. 로봇이 실제 세계에서 작업을 시도하고, 그 성능이 모니터링되며, 예상(시뮬레이션)과 실제(현실) 성능 간의 불일치가 자동으로 튜닝되는 신경 증강 시뮬레이터에 피드백된다. 그런 다음 LLM이 남아있는 실패 모드를 분석하고 다음 시뮬레이션 훈련 라운드를 위한 새로운 목표 무작위화나 보상 수정을 제안한다. 이는 로봇이 작업을 수행하는 법을 배우는 동시에 시뮬레이션 환경이 해당 작업에 대한 더 나은 교사가 되는 법을 배우는 선순환을 만든다. 이는 인간 주도 프로세스에서 자가 개선, 자율 개발 파이프라인으로의 근본적인 전환을 의미한다. Sim2Real 과정 자체가 학습된 기술이 되는 것이다.

#### **참고 자료**

1. www.numberanalytics.com, accessed July 1, 2025, [https://www.numberanalytics.com/blog/ultimate-guide-sim-to-real-transfer#:~:text=Sim%2Dto%2Dreal%20transfer%20refers,environment%20to%20the%20real%20world.](https://www.numberanalytics.com/blog/ultimate-guide-sim-to-real-transfer#:~:text=Sim-to-real transfer refers,environment to the real world.)
2. The Ultimate Guide to Sim-to-Real Transfer - Number Analytics, accessed July 1, 2025, https://www.numberanalytics.com/blog/ultimate-guide-sim-to-real-transfer
3. Sim-to-Real Transfer in Cognitive Robotics - Number Analytics, accessed July 1, 2025, https://www.numberanalytics.com/blog/sim-to-real-transfer-cognitive-robotics
4. The Simulation Hypothesis is naïve - a car analogy. : r/IsaacArthur - Reddit, accessed July 1, 2025, [https://www.reddit.com/r/IsaacArthur/comments/q3mvg7/the_simulation_hypothesis_is_na%C3%AFve_a_car_analogy/](https://www.reddit.com/r/IsaacArthur/comments/q3mvg7/the_simulation_hypothesis_is_naïve_a_car_analogy/)
5. Is the switch from sim to real fpv hard - Reddit, accessed July 1, 2025, https://www.reddit.com/r/fpv/comments/1g26pia/is_the_switch_from_sim_to_real_fpv_hard/
6. UNDERSTANDING DOMAIN RANDOMIZATION FOR SIM-TO-REAL TRANSFER - OpenReview, accessed July 1, 2025, https://openreview.net/pdf?id=T8vZHIRTrY
7. Sim-to-Real Transfer in Robotics - Number Analytics, accessed July 1, 2025, https://www.numberanalytics.com/blog/sim-to-real-transfer-linear-algebra-robotics
8. What is robotic simulation and why is it important?, accessed July 1, 2025, https://eureka.patsnap.com/article/what-is-robotic-simulation-and-why-is-it-important
9. Key Benefits of Robot Simulation - Facteon, accessed July 1, 2025, https://www.facteon.global/news/key-benefits-of-robot-simulation/
10. On the use of simulation in robotics: Opportunities, challenges, and suggestions for moving forward | PNAS, accessed July 1, 2025, https://www.pnas.org/doi/10.1073/pnas.1907856118
11. What Is Robot Simulation? - MATLAB & Simulink - MathWorks, accessed July 1, 2025, https://www.mathworks.com/discovery/robot-simulation.html
12. How Robotic Simulation Software Can Revolutionize Automation, accessed July 1, 2025, https://www.decipherzone.com/blog-detail/how-robotic-simulation-software-can-revolutionize-automation
13. The benefits of simulation training for engineers - Control Engineering, accessed July 1, 2025, https://www.controleng.com/the-benefits-of-simulation-training-for-engineers/
14. Sim-to-real via latent prediction: Transferring visual non-prehensile manipulation policies, accessed July 1, 2025, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2022.1067502/full
15. Transic: Sim-to-Real Policy Transfer by Learning from Online Correction - arXiv, accessed July 1, 2025, https://arxiv.org/html/2405.10315v1
16. On the use of simulation in robotics: Opportunities, challenges, and suggestions for moving forward, accessed July 1, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7817170/
17. Closing the Simulation-to-Reality Gap for Deep Robotic Learning - Google Research, accessed July 1, 2025, https://research.google/blog/closing-the-simulation-to-reality-gap-for-deep-robotic-learning/
18. Precision home robots learn with real-to-sim-to-real | MIT News, accessed July 1, 2025, https://news.mit.edu/2024/precision-home-robotics-real-sim-real-0731
19. Learning dexterity | OpenAI, accessed July 1, 2025, https://openai.com/index/learning-dexterity/
20. This AI-operated robotic hand moves with “unprecedented dexterity”, accessed July 1, 2025, https://www.weforum.org/stories/2018/08/this-ai-operated-robotic-hand-moves-with-unprecedented-dexterity/
21. (PDF) Crossing the Reality Gap: A Survey on Sim-to-Real Transferability of Robot Controllers in Reinforcement Learning - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/356010714_Crossing_the_Reality_Gap_A_Survey_on_Sim-to-Real_Transferability_of_Robot_Controllers_in_Reinforcement_Learning
22. Bridging the Gap from Robotics Simulations to Real World Application - Texas A&M Engineering, accessed July 1, 2025, https://engineering.tamu.edu/news/2024/10/bridging-the-gap-from-robotics-simulations-to-real-world-application.html
23. An Real-Sim-Real (RSR) Loop Framework for Generalizable Robotic Policy Transfer with Differentiable Simulation - arXiv, accessed July 1, 2025, https://arxiv.org/html/2503.10118v2
24. Simulation to Reality and Back: A Robot's Guide to Crossing the Reality Gap - QUT ePrints, accessed July 1, 2025, https://eprints.qut.edu.au/230537/1/Jack_Collins_Thesis.pdf
25. Simulacra and Simulation - Wikipedia, accessed July 1, 2025, https://en.wikipedia.org/wiki/Simulacra_and_Simulation
26. What exactly makes sim to real transfer a challenge in reinforcement learning? : r/robotics, accessed July 1, 2025, https://www.reddit.com/r/robotics/comments/1j99vrt/what_exactly_makes_sim_to_real_transfer_a/
27. Sim2Real Gap: Why Machine Learning Hasn't Solved Robotics Yet - Inbolt, accessed July 1, 2025, https://www.inbolt.com/resources/sim2real-gap-why-machine-learning-hasnt-solved-robotics-yet
28. Traversing the Reality Gap via Simulator Tuning - Pure, accessed July 1, 2025, https://researchmgt.monash.edu/ws/portalfiles/portal/405550710/382962277_oa.pdf
29. Sim-to-Real Reinforcement Learning for Vision-Based Dexterous Manipulation on Humanoids - arXiv, accessed July 1, 2025, https://arxiv.org/html/2502.20396v1
30. Why the sim2real problem in robotic manipulation? : r/reinforcementlearning - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/10vkqg8/why_the_sim2real_problem_in_robotic_manipulation/
31. On Sim2Real Transfer in Robotics （Part 2/3) | Haonan's blog, accessed July 1, 2025, https://www.haonanyu.blog/post/sim2real2/
32. SIM2Real Transfer - andrew.cmu.ed, accessed July 1, 2025, https://www.andrew.cmu.edu/course/10-403/slides/S19sim2real.pdf
33. MIT CSAIL teaches robots to do chores using Real-to-Sim-to-Real - The Robot Report, accessed July 1, 2025, https://www.therobotreport.com/mit-csail-teaches-robots-to-do-chores-using-real-to-sim-to-real/
34. Advice - Deformable Object Manipulation using Robotic Arms Simulation. - Reddit, accessed July 1, 2025, https://www.reddit.com/r/robotics/comments/1fc20i4/advice_deformable_object_manipulation_using/
35. 6IMPOSE: bridging the reality gap in 6D pose estimation for robotic grasping - Frontiers, accessed July 1, 2025, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2023.1176492/full
36. Simulation City: Introducing Waymo's most advanced simulation system yet for autonomous driving, accessed July 1, 2025, https://waymo.com/blog/2021/07/simulation-city
37. Reality Gap in Cognitive Robotics - Number Analytics, accessed July 1, 2025, https://www.numberanalytics.com/blog/reality-gap-cognitive-robotics-ultimate-guide
38. On Sim2Real Transfer in Robotics （Part 3/3) | Haonan's blog, accessed July 1, 2025, https://www.haonanyu.blog/post/sim2real3/
39. (PDF) Sim-to-Real Transfer in Robotics: Addressing the Gap between Simulation and Real- World Performance - ResearchGate, accessed July 1, 2025, https://www.researchgate.net/publication/390101654_Sim-to-Real_Transfer_in_Robotics_Addressing_the_Gap_between_Simulation_and_Real-_World_Performance
40. Towards Minimizing the LiDAR Sim-to-Real Domain Shift: Object-Level Local Domain Adaptation for 3D Point Clouds of Autonomous Vehicles - MDPI, accessed July 1, 2025, https://www.mdpi.com/1424-8220/23/24/9913
41. Training the trainer – using motion capture to close the robotic reality gap - QUT, accessed July 1, 2025, https://www.qut.edu.au/robotronica/event-item?id=149749
42. SIM2REALVIZ: Visualizing the Sim2Real Gap in Robot Ego-Pose Estimation, accessed July 1, 2025, https://xai4debugging.github.io/files/papers/visualizing_the_sim2real_gap_i.pdf
43. [R] Real robot trained via simulation and reinforcement learning is capable of running, getting up and recovering from kicks - Reddit, accessed July 1, 2025, https://www.reddit.com/r/MachineLearning/comments/ahm5u3/r_real_robot_trained_via_simulation_and/
44. Ch. 18 - System Identification - Underactuated Robotics, accessed July 1, 2025, https://underactuated.mit.edu/sysid.html
45. Introduction to System Identification - WPILib Docs, accessed July 1, 2025, https://docs.wpilib.org/en/stable/docs/software/advanced-controls/system-identification/introduction.html
46. Sim-to-Real Reinforcement Learning for Vision-Based Dexterous Manipulation on Humanoids - Toru Lin, accessed July 1, 2025, https://toruowo.github.io/recipe/
47. ICRA 2021: Auto-Tuned Sim-to-Real Transfer - YouTube, accessed July 1, 2025, https://m.youtube.com/watch?v=pEtJQ3uHYZE&pp=ygUNI3RoYWloYW5kaWNyYQ%3D%3D
48. TuneNet: One-Shot Residual Tuning for System Identification and Sim-to-Real Robot Task Transfer, accessed July 1, 2025, http://proceedings.mlr.press/v100/allevato20a/allevato20a.pdf
49. Hybrid Simulator Identification for Domain Adaptation via Adversarial Reinforcement Learning - NSF-PAR, accessed July 1, 2025, https://par.nsf.gov/servlets/purl/10299850
50. Domain Randomization: future of robust modeling | by Urwa Muaz | TDS Archive | Medium, accessed July 1, 2025, https://medium.com/data-science/domain-randomization-c7942ed66583
51. Question about domain randomization : r/reinforcementlearning - Reddit, accessed July 1, 2025, https://www.reddit.com/r/reinforcementlearning/comments/mrdblg/question_about_domain_randomization/
52. Simulation hypothesis - Wikipedia, accessed July 1, 2025, https://en.wikipedia.org/wiki/Simulation_hypothesis
53. Solving Rubik's Cube with a robot hand - OpenAI, accessed July 1, 2025, https://openai.com/index/solving-rubiks-cube/
54. [1910.07113] Solving Rubik's Cube with a Robot Hand - arXiv, accessed July 1, 2025, https://arxiv.org/abs/1910.07113
55. arxiv.org, accessed July 1, 2025, [https://arxiv.org/pdf/2303.12704#:~:text=Generative%20Adversarial%20Networks%20(GANs)%20%5B,effective%20in%20closing%20distribution%20gaps.](https://arxiv.org/pdf/2303.12704#:~:text=Generative Adversarial Networks (GANs) [,effective in closing distribution gaps.)
56. Toward Generalized Sim-to-Real Transfer for Robot Learning - Google Research, accessed July 1, 2025, https://research.google/blog/toward-generalized-sim-to-real-transfer-for-robot-learning/
57. Improving Sim-to-Real Transfer in Vision-Based Robot Navigation via Instance-Level GAN-Based Data Augmentation, accessed July 1, 2025, [https://ntrs.nasa.gov/api/citations/20240015866/downloads/GAN%20Based%20Data%20Augmentation%20for%20Sim%20to%20Real-final.pdf](https://ntrs.nasa.gov/api/citations/20240015866/downloads/GAN Based Data Augmentation for Sim to Real-final.pdf)
58. Sim-To-Real via Sim-To-Sim: Data-Efficient Robotic Grasping via Randomized-To-Canonical Adaptation Networks - CVF Open Access, accessed July 1, 2025, https://openaccess.thecvf.com/content_CVPR_2019/papers/James_Sim-To-Real_via_Sim-To-Sim_Data-Efficient_Robotic_Grasping_via_Randomized-To-Canonical_Adaptation_Networks_CVPR_2019_paper.pdf
59. Bridging the Sim-to-Real Gap for Efficient and Robust Robotic Skill Acquisition, accessed July 1, 2025, https://openreview.net/forum?id=R5FR5H1fAk
60. Domain Randomization Tips - Seita's Place, accessed July 1, 2025, https://danieltakeshi.github.io/2019/08/18/domain-randomization/
61. Revealing the Challenges of Sim-to-Real Transfer in Model-Based Reinforcement Learning via Latent Space Modeling - arXiv, accessed July 1, 2025, https://arxiv.org/html/2506.12735v1
62. Evaluating Real-World Robot Manipulation Policies in Simulation - arXiv, accessed July 1, 2025, https://arxiv.org/html/2405.05941v1
63. Autonomous Vehicle (AV) Simulation - NVIDIA Developer, accessed July 1, 2025, https://developer.nvidia.com/drive/simulation
64. Autonomous Vehicle & Self-Driving Car Technology from NVIDIA, accessed July 1, 2025, https://www.nvidia.com/en-us/solutions/autonomous-vehicles/
65. Reconstructing the Real World in DRIVE Sim With AI | NVIDIA Blog, accessed July 1, 2025, https://blogs.nvidia.com/blog/drive-sim-neural-reconstruction-engine/
66. Deep Drone Racing: from Simulation to Reality with Domain Randomization - Robotics and Perception Group, accessed July 1, 2025, https://rpg.ifi.uzh.ch/docs/TRO19_Loquercio.pdf
67. A Sim-to-Real Deep Learning-based Framework for Autonomous Nano-drone Racing, accessed July 1, 2025, https://arxiv.org/html/2312.08991v1
68. Crossing the Gap: A Deep Dive into Zero-Shot Sim-to-Real Transfer for Dynamics, accessed July 1, 2025, https://www.robot-learning.uk/crossing-the-gap
69. JuliaSim: Next-Gen Modeling | JuliaHub, accessed July 1, 2025, https://juliahub.com/products/juliasim
70. Simulation to Reality: Robots Now Train Themselves with the Power of LLM (DrEureka), accessed July 1, 2025, https://www.analyticsvidhya.com/blog/2024/05/sim-to-real-robots-now-train-themselves-dreureka/

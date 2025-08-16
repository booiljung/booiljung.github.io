# Project GR00T: 휴머노이드 로보틱스의 운영체제 구축을 향한 엔비디아의 야심 찬 계획 분석

**Executive Summary:** 본 보고서는 엔비디아가 발표한 Project GR00T를 단순한 기술 발표가 아닌, 물리적 AI(Embodied AI)의 미래를 장악하기 위한 거대한 전략적 이니셔티브의 핵심으로 분석한다. GR00T는 제품이라기보다는 물리적 AI를 위한 CUDA와 유사한 플랫폼 전략으로, 로보틱스 산업 전체가 엔비디아의 하드웨어와 소프트웨어에 의존하게 만들도록 설계되었다. 본 보고서는 GR00T의 핵심 기술 스택, 테슬라와 피규어 AI와 같은 주요 경쟁사와의 구도, 목표 시장 및 거시적 전략과 규제 문제를 심층적으로 다룬다. 핵심적인 경쟁 구도는 엔비디아의 수평적 생태계 전략, 테슬라의 수직적 통합 모델, 그리고 피규어 AI의 민첩한 상용화 모델 간의 충돌로 요약된다. GR00T는 차세대 산업 혁명의 중추 신경계를 장악하려는 엔비디아의 야심 찬 행보의 중심에 있다.

## 1. I. Project GR00T 해부: 범용 로봇 00 기술(Generalist Robot 00 Technology)

이 섹션은 Project GR00T의 핵심 기술을 해부하여, 이 프로젝트가 사전 프로그래밍된 로봇 공학에서 물리적 범용 AI로의 중대한 도약을 의미함을 규명한다. GR00T의 아키텍처, 목표, 그리고 발전 경로를 분석한다.

### 1.1  비전: 사전 프로그래밍된 기계에서 인공 일반 로보틱스로

GR00T의 등장은 로보틱스 패러다임의 근본적인 전환을 예고한다. 엔비디아 CEO 젠슨 황은 "범용 휴머노이드 로봇을 위한 파운데이션 모델 구축은 오늘날 AI 분야에서 해결해야 할 가장 흥미로운 문제 중 하나"라고 선언하며 이 프로젝트의 중요성을 강조했다.1

GR00T는 **범용 로봇 00 기술(Generalist Robot 00 Technology)**의 약자로 1, 로봇이 자연어를 이해하고 인간의 행동을 관찰하여 움직임을 모방하도록 설계되었다. 이를 통해 로봇은 실제 세계를 탐색하고, 적응하며, 상호작용하기 위한 조정 능력, 민첩성 및 기타 기술을 신속하게 학습할 수 있다.1 이는 특정 작업에 국한된 로봇에서 벗어나, 다양한 상황에 대처할 수 있는 "범용" 에이전트로의 전략적 전환을 의미하며, 엔비디아가 "인공 일반 로보틱스(artificial general robotics)"라고 명명한 궁극적인 목표를 향한 첫걸음이다.1 이 프로젝트는 물리적 세계에서 작동하는 범용인공지능(AGI)을 만들기 위한 "문샷(moonshot)" 프로젝트로 평가받는다.8

### 1.2  GR00T 파운데이션 모델: 기술 심층 분석

GR00T 이니셔티브의 핵심에는 휴머노이드 로봇을 위한 최초의 개방형 파운데이션 모델인 **GR00T N1**이 있다.9 이 모델은 시각-언어-행동(Vision-Language-Action, VLA) 모델로서, 로봇의 '지능'을 구성하는 핵심 소프트웨어다.9

#### 1.2.1 이중 시스템 인지 프레임워크

GR00T N1의 아키텍처는 인간의 인지 과정, 특히 대니얼 카너먼의 저서 "생각에 관한 생각(Thinking, Fast and Slow)"에서 영감을 받은 이중 시스템(dual-system) 구조를 채택했다는 점에서 주목할 만하다.11 이는 단순히 마케팅 수사를 넘어, 물리적 AI의 복잡성을 해결하기 위한 정교한 엔지니어링적 선택을 시사한다. 즉, 엔비디아는 단일 거대 모델(monolithic model)만으로는 현실 세계의 복잡성을 감당하기 어렵다고 판단하고, 인간의 문제 해결 방식과 유사한 구조화된 아키텍처가 견고한 성능의 핵심이라고 본 것이다.

- **시스템 2 (느린 생각):** 이 모듈은 고수준의 추론을 담당한다. 사전 훈련된 시각-언어 모델(VLM)이 시각 정보와 언어 지시를 처리하여 환경을 해석하고 작업 목표를 이해한 뒤 행동 계획을 수립한다.9 이 VLM은 엔비디아의 

  **Eagle-2** 모델을 기반으로 하며 15, 강력한 GPU 상에서 실행되어 "무엇을, 왜 해야 하는가" (예: "이 컵을 식기세척기에 넣어야 한다")를 결정한다.

- **시스템 1 (빠른 생각):** 이 모듈은 인간의 반사 신경이나 직관과 유사하게 빠른 행동 생성을 담당한다.12

  **확산 트랜스포머(Diffusion Transformer, DiT)** 모듈이 시스템 2에서 수립된 계획을 입력받아, 실시간으로 유연하고 정밀한 모터 제어 신호를 생성한다.9 이는 "어떻게 해야 하는가" (컵을 잡고, 팔을 움직여 식기세척기 문을 열고, 컵을 넣는 일련의 정밀한 근육 움직임)를 실행한다.

#### 1.2.2 엔드-투-엔드 훈련 및 데이터 이질성

GR00T N1의 핵심 특징 중 하나는 시스템 1과 시스템 2가 긴밀하게 결합되어 **엔드-투-엔드(end-to-end)** 방식으로 함께 훈련된다는 점이다.9 이를 통해 고수준의 추론이 저수준의 물리적 실행으로 원활하게 전환될 수 있다.

모델 훈련에는 실제 로봇의 궤적 데이터, 인간의 1인칭 시점 비디오, 그리고 방대한 양의 합성 데이터(synthetic data)를 포함하는 "이질적인 혼합(heterogeneous mixture)" 데이터가 사용된다.9 이 

**"데이터 피라미드"** 접근법은 모델의 일반화 성능을 극대화하는 근간이 된다.11 피라미드의 하단에는 방대한 웹 데이터와 인간 비디오가 일반적인 시각 및 행동 패턴을 제공하고, 중간층에는 시뮬레이션 및 신경망 모델로 생성된 합성 데이터가, 그리고 최상단에는 가장 가치가 높지만 수집이 어려운 실제 로봇 데이터가 위치하여 모델을 현실 세계에 정밀하게 접지시킨다.11

### 1.3  진화와 로드맵: N1에서 N1.5로, 그리고 그 너머

엔비디아는 GR00T 모델의 빠른 반복 개발을 통해 자사 플랫폼의 개발 속도를 입증했다. GTC에서 GR00T N1이 발표된 지 불과 두 달 만인 컴퓨텍스(COMPUTEX)에서 업데이트 버전인 **GR00T N1.5**가 공개되었다.17

이러한 신속한 업데이트는 **GR00T-Dreams**라는 새로운 합성 데이터 생성 블루프린트 덕분에 가능했다. 엔비디아 연구팀은 GR00T-Dreams를 사용하여 N1.5 모델 훈련에 필요한 데이터를 단 36시간 만에 생성했다. 이는 기존의 인간 시연 데이터 수집 방식으로는 거의 3개월이 소요될 작업이었다.17 이 36시간이라는 수치는 단순한 제품 개선을 넘어, 엔비디아의 전체 데이터 생성 생태계가 제공하는 '속도'라는 가치를 입증하는 강력한 개념 증명(proof-of-concept)이다. 이는 잠재적 파트너사들에게 엔비디아 아이작(Isaac) 플랫폼을 채택하는 것이 R&D 주기를 전례 없이 단축시킬 수 있다는 메시지를 전달하며, 경쟁력을 유지하고자 하는 모든 로보틱스 기업에게 엔비디아의 데이터 및 시뮬레이션 인프라에 대한 의존성을 심화시킨다.

GR00T N1.5는 새로운 환경과 작업 공간 구성에 더 잘 적응하고, 사용자 지시를 통해 물체를 더 정확하게 인식하며, 물품 분류나 정리와 같은 일반적인 자재 처리 및 제조 작업에서의 성공률을 크게 향상시켰다.17 이처럼 빠른 데이터 기반의 개선 주기는 개발자들에게 제공되는 엔비디아 플랫폼의 핵심 가치 제안 중 하나다.

## 2. II. 엔비디아 로보틱스 생태계: 시장 지배를 위한 풀스택 전략

이 섹션은 엔비디아가 Project GR00T를 중심으로 구축한 포괄적이고 수직적으로 통합된 생태계를 상세히 분석한다. GR00T가 로봇의 '정신'이라면, 그 힘은 엔비디아가 통제하는 하드웨어, 시뮬레이션 도구, 데이터 파이프라인의 완벽한 스택에서 나온다.

### 2.1  실리콘 심장: 젯슨 토르와 블랙웰 아키텍처

엔비디아는 AI 모델뿐만 아니라, 이를 구동하기 위해 특별히 제작된 컴퓨터인 **젯슨 토르(Jetson Thor)**를 함께 제공한다.1 이는 휴머노이드 로봇을 위해 특별히 설계된 새로운 시스템 온 칩(SoC)이다.2

젯슨 토르의 기술적 사양은 압도적이다. 차세대 **엔비디아 블랙웰(Blackwell) 아키텍처**를 기반으로 하며, 트랜스포머 엔진을 탑재하여 **800 테라플롭스(TFLOPS)**의 8비트 AI 성능을 제공한다.1 이 막대한 연산 능력은 GR00T와 같은 멀티모달 생성형 AI 모델을 실행하는 데 필수적이라고 명시되어 있다.1 또한 고성능 CPU 클러스터, 100Gb 이더넷 대역폭, 기능 안전 프로세서를 통합하여 로보틱스 기업의 설계 및 통합 노력을 크게 간소화한다.1

엔비디아의 전략은 명확하다. "개방형 파운데이션 모델"이라는 매력적인 소프트웨어를 제공함으로써, 이를 구동하기 위한 필수적인 고성능 하드웨어, 즉 젯슨 토르에 대한 자연스럽고 거의 불가피한 수요를 창출한다. 이는 엔비디아가 CUDA와 GPU를 통해 AI 시장에서 성공적으로 구사했던 고전적인 플랫폼 전략의 재현이다.

**표 1: NVIDIA Jetson Thor 기술 사양**

| 기능             | Jetson AGX Thor                       | Jetson AGX Orin (비교용)                  |
| ---------------- | ------------------------------------- | ----------------------------------------- |
| **GPU 아키텍처** | NVIDIA Blackwell                      | NVIDIA Ampere                             |
| **AI 성능**      | 최대 2070 TFLOPS (FP4, 희소)          | 최대 275 TOPS                             |
| **GPU**          | 2560 CUDA 코어, 96 5세대 텐서 코어    | 최대 2048 CUDA 코어, 64 텐서 코어         |
| **CPU**          | 14코어 Arm® Neoverse®-V3AE 64비트 CPU | 12코어 Arm® Cortex®-A78AE v8.2 64비트 CPU |
| **메모리**       | 128GB 256비트 LPDDR5X (273 GB/s)      | 32GB/64GB 256비트 LPDDR5 (204.8 GB/s)     |
| **네트워킹**     | 4x 25GbE (최대 100Gbps)               | 1x 10GbE                                  |
| **전력 소비**    | 40W - 130W                            | 15W - 60W                                 |
| **폼 팩터**      | 100 mm x 87 mm                        | 100 mm x 87 mm                            |

출처: 1

### 2.2  디지털 트윈 시험장: 아이작 심과 옴니버스

엔비디아는 "시뮬레이션 우선(simulation-first)"이라는 기치 아래, 시뮬레이션을 현대 로보틱스 개발의 초석으로 삼고 있다.21 이 전략의 핵심에는 시뮬레이션과 현실 간의 격차, 즉 '심투리얼(Sim2Real)' 문제를 해결하려는 의도가 깔려있다. 시뮬레이션 환경이 현실과 충분히 유사하다면, 로봇의 물리적 하드웨어 특성은 상대적으로 덜 중요해진다. 즉, AI 정책이 다양한 가상 환경에서 훈련되어 충분한 강건성(robustness)을 확보하면, 여러 종류의 실제 하드웨어에 일반화될 수 있다. 이는 로봇 하드웨어 자체를 상품화(commoditize)하고, 가치를 AI '두뇌'와 개발 플랫폼으로 이전시키는 고도의 전략이다.

- **엔비디아 아이작 심(Isaac Sim):** 엔비디아 옴니버스(Omniverse)를 기반으로 구축된 핵심 시뮬레이션 플랫폼이다.26 물리적으로 정확하고 사실적인 가상 환경을 제공하여, 실제 환경에 로봇을 배치하기 전에 훈련, 테스트, 검증을 수행할 수 있게 한다.26 엔비디아 PhysX와 구글 딥마인드 및 디즈니 리서치와 공동 개발 중인 Newton 물리 엔진을 통해 고충실도의 물리 시뮬레이션을 보장한다.12
- **아이작 랩(Isaac Lab):** 아이작 심 위에 구축된 성능 최적화 애플리케이션으로, 수천 개의 병렬 시뮬레이션을 실행하여 로봇 학습, 특히 **강화 학습(Reinforcement Learning, RL)**을 가속화하는 데 특화되어 있다.1
- **옴니버스와 OpenUSD:** 옴니버스와 OpenUSD 표준을 채택한 것은 상호 운용성을 극대화하기 위한 전략적 선택이다. 이를 통해 공장이나 다양한 환경의 복잡한 디지털 트윈을 생성하고, 여러 개발팀이 협업하며, 모든 자산이 시뮬레이션에 즉시 사용될 수 있도록 보장한다.27

### 2.3  데이터 엔진: 코스모스, 미믹, 그리고 드림즈

범용 로봇 훈련의 가장 큰 난제인 "데이터 병목 현상"을 해결하기 위해, 엔비디아는 독자적인 데이터 생성 파이프라인을 구축했다.26

- **GR00T-Mimic (모방 학습):** **모방 학습(Imitation Learning, IL)**을 통해 데이터 수집을 확장하는 워크플로우다.30 Apple Vision Pro와 같은 XR 기기를 사용하여 수집된 소수의 인간 시연 데이터를 기반으로, 아이작 심 내에서 방대한 양의 합성 모션 데이터를 생성한다.27

- **GR00T-Dreams (생성형 AI):** 이는 더 진보된 블루프린트로, **엔비디아 코스모스 월드 파운데이션 모델(Cosmos WFMs)**을 사용하여 단일 이미지와 텍스트 프롬프트만으로 완전히 새로운 합성 모션 데이터를 생성한다.17 미믹이 기존 데이터를 증강하는 반면, 드림즈는 로봇이 한 번도 본 적 없는 새로운 기술과 행동을 학습할 수 있도록 완전히 새로운 데이터를 *창조*한다.17

- **코스모스 WFM:** 현실적인 가상 세계를 이해하고 생성하는 생성형 AI 모델로, GR00T-Dreams가 물리적으로 타당하고 다양한 훈련 시나리오를 만들 수 있는 기반을 제공한다.17

### 2.4  특화 기술 라이브러리: GR00T 워크플로우 및 아이작 라이브러리

엔비디아는 특정 영역의 개발을 가속화하기 위해 사전 훈련된 모델, 정책, 참조 워크플로우 모음을 제공한다.30 이 모듈식 접근 방식은 개발자들이 맨바닥에서 시작하는 대신, 검증된 기술 기반 위에서 혁신을 이룰 수 있게 한다.

- 주요 워크플로우:
  - **GR00T-Dexterity:** 강화 학습 기반의 정교한 조작 기술.30
  - **GR00T-Mobility:** 강화 학습과 모방 학습을 결합한 이동 및 내비게이션 기술.30
  - **GR00T-Control:** 전신 제어(Whole-Body Control)를 위한 학습 기반 제어 기술.30
- **아이작 매니퓰레이터 & 퍼셉터:** 이 라이브러리들은 생태계를 더욱 강화한다. **아이작 매니퓰레이터(Isaac Manipulator)**는 로봇 팔의 민첩성을 극대화하여 경로 계획 속도를 최대 80배까지 향상시킨다. **아이작 퍼셉터(Isaac Perceptor)**는 고가의 라이다(LiDAR) 센서 대신 표준 카메라를 사용하여 3D 서라운드 비전 기능을 제공함으로써 비용을 절감하고 접근성을 높인다.3 이들은 비단 휴머노이드뿐만 아니라 기존 산업용 로봇 시장을 공략하기 위한 전략적 포석이다. 엔비디아는 미래의 휴머노이드 플랫폼을 구축하는 동시에, 야스카와, 유니버설 로봇, BYD, KION 그룹과 같은 기존 산업 파트너들에게 즉각적인 가치를 제공하며 로보틱스 시장 전반에 자사의 소프트웨어 스택을 확산시키고 있다.1 이는 이들 고객이 미래에 GR00T 기반 휴머노이드로 전환할 때 자연스러운 업그레이드 경로를 제공하는 효과를 낳는다.

이 모든 구성 요소들은 독립적으로 작동하는 것이 아니라, 서로를 강화하는 강력한 선순환 구조를 형성한다. 더 나은 시뮬레이션(아이작 심)은 더 높은 품질의 합성 데이터(미믹, 드림즈)를 생성하고, 이 데이터는 더 유능한 AI 모델(GR00T)을 훈련시키는 데 사용된다. 그리고 이 진보된 모델을 실시간으로 구동하기 위해서는 젯슨 토르의 막대한 연산 능력이 필요하며, 이 모델들의 개발과 훈련은 DGX 및 블랙웰 시스템과 같은 클라우드 GPU 클러스터를 요구한다.17 이처럼 각 요소가 서로에 대한 수요를 창출하는 통합된 파이프라인은 경쟁사가 쉽게 복제하기 어려운 강력한 해자(moat)를 구축한다.

## 3. III. 경쟁의 장: 플랫폼 대 수직 통합

이 섹션은 휴머노이드 로보틱스 경쟁의 세 가지 주요 전략을 비교 분석한다. 엔비디아의 수평적 플랫폼 접근 방식과 테슬라 및 피규어 AI의 수직 통합 모델을 대조하며 각 전략의 강점, 약점, 그리고 전략적 베팅을 심층적으로 탐구한다.

### 3.1  엔비디아의 플랫폼 플레이: "로보틱스를 위한 CUDA" 전략

엔비디아의 전략은 자체 브랜드의 휴머노이드 로봇을 만드는 것이 아니라, 산업 생태계 전체를 활성화하는 데 초점을 맞추고 있다.7 이는 PC 및 스마트폰 시장에서 성공했던 수평적 생태계 모델을 로보틱스에 적용하려는 시도다. 엔비디아는 '인텔/ARM + 윈도우/안드로이드'에 해당하는 계층(젯슨 토르 + GR00T/아이작)을 제공하고, 수십 개의 'OEM'(보스턴 다이내믹스, 어질리티 로보틱스 등)이 하드웨어와 애플리케이션에서 혁신을 이루도록 지원한다.

엔비디아는 **1X 테크놀로지스, 어질리티 로보틱스, 앱트로닉, 보스턴 다이내믹스, 피규어 AI, 푸리에 인텔리전스, 생츄어리 AI, 유니트리, 샤오펑(XPENG)** 등 주요 휴머노이드 기업 대부분과 포괄적인 AI 플랫폼 구축을 위한 파트너십을 맺었다.1

- **사례 연구 - 1X 테크놀로지스:** 1X는 엔비디아와 협력하여 자사의 NEO 로봇을 훈련시켰으며, GR00T N1 정책을 후속 훈련(post-training)하여 가정 내 정리정돈 작업을 자율적으로 수행하는 모습을 시연했다. 1X의 CEO는 자체 모델을 개발하면서도 GR00T가 "로봇의 추론과 기술에 상당한 향상"을 가져다주었다고 평가했다.12 이 협력에는 NEO 로봇을 위한 데이터셋 API와 추론 SDK 개발이 포함되었다.38
- **사례 연구 - 어질리티 로보틱스:** Digit 로봇의 개발사인 어질리티는 훈련 및 테스트를 위해 아이작 심과 아이작 랩의 사용을 확대하고 있다. 그들은 물리적 AI를 가능하게 하는 엔비디아의 기술 덕분에 "Digit의 역량을 그 어느 때보다 빠르게 발전시키고 있다"고 밝혔다.39 이 회사는 GR00T 플랫폼의 초기 채택 기업 중 하나다.2
- **사례 연구 - 보스턴 다이내믹스:** 보스턴 다이내믹스는 차세대 전기 구동 아틀라스(Atlas) 로봇에 아이작 GR00T 프레임워크와 젯슨 토르 컴퓨팅 플랫폼을 채택했다.40 아이작 랩을 활용하여 학습 기반의 민첩성과 이동 정책을 개발하고 있으며, 안전 아키텍처 및 비전 파이프라인 정의에 대해 엔비디아와 협력하고 있다.40

이처럼 거의 모든 주요 기업(테슬라 제외)이 엔비디아 플랫폼을 채택하는 것은 거대한 전략적 승리다. 이는 네트워크 효과를 창출하고 엔비디아를 사실상의 표준으로 자리매김하게 한다. 하지만 이는 동시에 양날의 검이 될 수 있다. 다양한 하드웨어와 목표를 가진 여러 경쟁사의 요구를 모두 충족시켜야 하므로 개발 속도가 저하되거나, 특정 파트너에게 완벽하게 최적화되지 않은 "만능" 플랫폼이 될 수 있다. 수직적으로 통합된 테슬라와 같은 경쟁자는 훨씬 더 긴밀한 하드웨어-소프트웨어 공동 설계를 통해 더 높은 성능과 효율성을 달성할 잠재력이 있다. 엔비디아의 과제는 다양한 파트너들이 자체 AI 스택을 구축하는 길을 택하지 않도록 충분한 가치와 유연성을 제공하는 것이다.

**표 2: 휴머노이드 로봇 이니셔티브 전략 비교**

| 차원                   | NVIDIA (Project GR00T)                           | Tesla (Optimus)                          | Figure AI (Figure 02 / Helix)        |
| ---------------------- | ------------------------------------------------ | ---------------------------------------- | ------------------------------------ |
| **핵심 전략**          | 수평적 플랫폼 (생태계 활성화)                    | 수직적 통합 (완제품 생산)                | 민첩한 상용화 (파트너십 후 내재화)   |
| **하드웨어 접근**      | 하드웨어 불가지론 (파트너 지원)                  | 완전 자체 개발 (액추에이터, 센서 등)     | 자체 개발 및 파트너 부품 활용        |
| **소프트웨어/AI 모델** | 개방형 파운데이션 모델 (GR00T)                   | 독점적 폐쇄형 스택 (FSD 기반)            | 독점적 자체 개발 (Helix)             |
| **데이터 전략**        | 시뮬레이션 중심, 합성 데이터 (Isaac Sim, Cosmos) | 실제 차량 데이터 + 시뮬레이션            | 원격 조작 데이터 + 시뮬레이션        |
| **시장 진출(GTM)**     | B2B (로봇 제조사에 플랫폼 판매)                  | B2C/B2B (자체 로봇 판매, 초기 내부 활용) | B2B (특정 산업 파일럿, 예: BMW, UPS) |

출처: 1

### 3.2  테슬라 옵티머스 접근법: 완전한 수직 통합

테슬라의 전략은 엔비디아와 근본적으로 다르다. 모든 것을 자체적으로 개발하는 폐쇄적인 수직 통합 접근 방식을 취한다.42 이는 애플의 성공 모델과 유사하다. 즉, 하드웨어와 소프트웨어를 완벽하게 통제함으로써 우수하고 통합된 사용자 경험을 제공하겠다는 것이다.

- **자체 개발 하드웨어:** 테슬라는 자동차 제조에서 축적한 방대한 경험을 바탕으로 자체 액추에이터, 센서, 배터리 시스템을 설계한다.46 이는 하드웨어 스택에 대한 깊은 통제력을 제공하며, 하드웨어에 구애받지 않으려는 엔비디아의 목표와 뚜렷한 대조를 이룬다.
- **통합 AI 스택:** 옵티머스는 테슬라의 완전자율주행(FSD) 차량을 위해 개발된 동일한 핵심 AI 시스템에 의해 제어된다.42 이를 통해 테슬라는 자사 차량 플릿에서 수집한 방대한 실제 세계 데이터와 컴퓨터 비전 및 계획 수립에 대한 전문 지식을 활용할 수 있다.
- **시장 진출 전략:** 초기 계획은 수천 대의 옵티머스 로봇을 자사 공장에 먼저 배치하여 유용성을 입증하고 성능을 개선하는 것이다.51 장기적인 비전은 3만 달러 미만의 가격으로 대량 생산하는 것이다.50

### 3.3  피규어 AI 접근법: 민첩한 상용화와 전략적 전환

피규어 AI는 전략적 파트너십을 통해 신속한 상용화에 초점을 맞춘 세 번째의 민첩한 모델을 대표한다.54

- **파트너십 기반 시장 진출:** 피규어는 초기 실제 사용 사례를 확보하기 위해 공격적으로 파트너십을 추진했다.
  - **BMW:** 사우스캐롤라이나주 스파르탄버그 자동차 제조 시설에 피규어 로봇을 배치하여 판금 부품 삽입과 같은 작업을 수행하는 파일럿 프로그램을 진행했다.43
  - **UPS:** 소포 분류와 같은 작업을 위해 물류 네트워크 내에 피규어 휴머노이드를 사용하는 방안을 모색하기 위한 논의를 지속하고 있다.58
- **OpenAI와의 결별: 전략적 전환:** 피규어는 초기에 OpenAI와 파트너십을 맺고 AI 모델을 개발했다.60 그러나 2025년 초, 피규어는 자체적인 엔드-투-엔드 로봇 AI에서 "중대한 돌파구"를 마련했다는 이유로 파트너십을 종료했다.45 피규어의 CEO 브렛 애드콕은 물리적 AI를 대규모로 구현하기 위해서는 "로봇 AI를 수직적으로 통합해야 한다"고 주장하며, 범용 모델만으로는 불충분하다고 밝혔다.45
- **자체 AI (Helix):** 이러한 판단은 자체 VLA 모델인 **Helix** 개발로 이어졌다. Helix는 자체 데이터로 훈련된 이중 시스템 모델(추론을 위한 시스템 2, 제어를 위한 시스템 1)이다.67 이 전략적 전환은 피규어가 소프트웨어 측면에서 테슬라의 수직 통합 모델에 더 가까워졌음을 시사한다. 피규어와 OpenAI의 결별은 AI 산업 전체에 중요한 시사점을 던진다. 이는 범용 LLM이 전문화되고 위험 부담이 큰 물리적 작업을 수행하는 데 한계가 있음을 보여주는 신호탄이다. 피규어 CEO는 LLM이 "상품화되고 있으며", 물리적 AI 퍼즐의 "가장 작은 조각"에 불과하다고 단언했다.65 실제 문제는 높은 빈도의 저수준 제어와 인식-행동 간의 깊은 통합에 있다는 것이다. 이는 범용 '두뇌' AI(예: ChatGPT)와 고도로 전문화된 물리적 구현을 위한 '소뇌' AI 간의 분화 가능성을 암시한다. 피규어가 자체 AI를 구축하기로 한 결정은 이러한 전문화가 성공의 유일한 길이라는 값비싸고 위험한 베팅이다.

## 4. IV. 시장 환경 및 경제적 영향

이 섹션은 휴머노이드 로봇 시장의 기회를 정량화하고, 이들이 목표로 하는 핵심 산업 분야를 분석하며, 시장의 경제적 타당성 전체를 뒷받침하는 핵심 기술 과제인 심투리얼(Sim2Real) 격차를 심층적으로 탐구한다.

### 4.1  휴머노이드 로봇 시장: 전망과 동인

여러 산업 분석 기관의 예측을 종합하면 휴머노이드 로봇 시장의 잠재적 규모와 성장률을 포괄적으로 파악할 수 있다.

- **MarketsandMarkets**는 시장이 2025년 29억 2천만 달러에서 2030년 152억 6천만 달러로 연평균 성장률(CAGR) **39.2%**로 성장할 것으로 예측한다.68
- **Maximize Market Research**는 2024년 22억 4천만 달러에서 2032년 488억 7천만 달러로 CAGR **47%**의 성장을 전망한다.71
- **Morgan Stanley**는 2050년까지 시장이 **5조 달러**를 넘어설 수 있다는 가장 낙관적인 장기 전망을 제시한다.72

주요 성장 동인으로는 노동력 부족 문제를 해결하기 위한 자동화 수요 증가 73, AI 및 LLM 기술의 발전 68, 그리고 고령화 인구에 따른 의료 서비스 수요 증가 73 등이 있다.

**표 3: 휴머노이드 로봇 시장 전망 (2025-2035)**

| 리서치 기관                  | 기준 연도 가치     | 예측 연도 가치      | 예측 기간 | 연평균 성장률(CAGR) |
| ---------------------------- | ------------------ | ------------------- | --------- | ------------------- |
| **MarketsandMarkets**        | 29.2억 달러 (2025) | 152.6억 달러 (2030) | 2025-2030 | 39.2%               |
| **Maximize Market Research** | 22.4억 달러 (2024) | 488.7억 달러 (2032) | 2025-2032 | 47%                 |
| **Grand View Research**      | 15.5억 달러 (2024) | 40.4억 달러 (2030)  | 2025-2030 | 17.5%               |
| **Research Nester**          | 24.1억 달러 (2024) | 179.4억 달러 (2037) | 2025-2037 | 16.7%               |
| **Morgan Stanley**           | -                  | 5조 달러+ (2050)    | ~2050     | -                   |

출처: 68

### 4.2  목표 산업: 사용 사례 및 ROI 분석

#### 4.2.1 제조 및 물류

이 분야는 휴머노이드 로봇의 초기 핵심 목표 시장이다. 자재 취급, 품질 검사, 경량 조립 등의 작업에 시험적으로 도입되고 있다.73

- **사용 사례:** 피규어 AI의 BMW 공장 파일럿 55 및 UPS와의 잠재적 협력 59은 자동차 조립 및 소포 분류와 같은 구체적인 적용 사례를 보여준다. 목표는 노동력 격차를 메우고, 생산성을 향상시키며, 반복적이거나 인체공학적으로 부담이 큰 작업을 대체하여 작업자 안전을 증진하는 것이다.80 현재 BMW나 UPS와 같은 기업들이 휴머노이드를 도입하는 것은 당장의 재무적 이익보다는, 변혁적인 기술의 초기 단계에 참여하여 학습 곡선을 선점하려는 전략적 투자에 가깝다. 이들은 시스템 통합 방법, 적합한 작업 식별, 작업 흐름 재설계 등에 대한 제도적 지식을 쌓고 있으며, 이는 기술이 성숙하고 비용이 하락했을 때 경쟁사보다 빠르게 규모를 확장할 수 있는 핵심 경쟁력이 될 것이다.
- **경제적 영향 및 ROI:** 투자 수익률(ROI)은 도입의 중요한 장벽이다. 초기 비용은 대당 10만~15만 달러로 추정될 만큼 높으며 82, 통합, 엔지니어링, 유지보수 비용을 고려하면 실제 투자비는 로봇 가격의 3~5배에 달할 수 있다.84 하지만 ROI 회수 기간은 2019년 5.3년에서 2023년 2.8년으로 단축되는 추세다.82 도입 시 기대 효과로는 첫해 인건비 22~28% 절감, 생산성 30~35% 향상, 24/7 가동 등이 있다.82

#### 4.2.2 의료

고령화 인구와 간병 인력 부족으로 인해 의료 분야는 중요한 2차 시장으로 부상하고 있다.73

- **사용 사례:** 개인 비서 및 간병이 가장 큰 응용 분야이며 75, 환자 모니터링, 재활 지원, 사회적 고립감 완화를 위한 동반자 역할 등을 수행한다.68
- **비용-편익 분석:** 의료용 로봇의 높은 비용(수술 로봇은 200만~250만 달러, 연간 유지보수비 10만 달러 이상)은 주요 장애물이다.87 의료진의 업무 부담 경감, 환자 관리의 일관성 향상, 일상 업무 자동화 등의 이점이 있지만, 많은 응용 분야에서 명확한 비용-편익 분석은 아직 부족한 실정이다.87

### 4.3  심투리얼(Sim2Real) 과제: 해결되지 않은 거대한 문제

**심투리얼(Sim2Real) 격차**는 시뮬레이션 환경에서의 로봇 행동과 실제 세계에서의 행동 간의 불일치를 의미하며 28, 이는 학습 기반 로봇 기술의 광범위하고 신뢰성 있는 배포를 가로막는 가장 중요한 기술적 장벽이다. 실제 세계에서 훈련 데이터를 수집하는 것은 비용이 많이 들고, 느리며, 위험하기 때문에 89, 시뮬레이션은 유일하게 실행 가능한 대안이다. 따라서 심투리얼 격차를 줄이는 능력은 휴머노이드 시장 전체의 경제적 생존 가능성과 직결된다.

- **문제점:** 이 격차는 물리 법칙(특히 접촉 및 마찰), 시각적 외형(렌더링), 그리고 로봇 고유의 하드웨어적 특성(액추에이터의 백래시 등)을 완벽하게 시뮬레이션하기 어렵기 때문에 발생한다.89 완벽한 시뮬레이션에서만 훈련된 정책은 예측 불가능하고 복잡한 실제 환경에서 실패할 수밖에 없다.90
- **엔비디아의 해결책:** 엔비디아의 아이작 플랫폼 전체는 이 격차를 해소하기 위해 설계되었다.
  - **고충실도 시뮬레이션:** 아이작 심과 PhysX/Newton 엔진을 사용하여 물리적으로 정확한 시뮬레이션 환경을 구축한다.26
  - **도메인 랜덤화(Domain Randomization):** 핵심 기술은 하나의 완벽한 시뮬레이션이 아닌, 수천 개의 *다양한* 시뮬레이션 환경에서 AI를 훈련시키는 것이다. 조명, 질감, 물체의 질량, 마찰과 같은 매개변수를 무작위로 변경함으로써, AI는 일반화 능력을 학습하고 실제 세계의 차이에 강건해진다.28
  - **합성 데이터 생성:** 아이작 리플리케이터(Isaac Replicator) 92 및 GR00T 블루프린트(미믹, 드림즈)와 같은 도구를 사용하여 방대하고 다양한 데이터셋을 생성함으로써 시뮬레이션과 현실 간의 "콘텐츠 격차"를 해소한다.17
  - **심투리얼 전환:** 엔비디아는 시뮬레이션에서 학습한 정책을 미세 조정 없이 실제 로봇에 직접 적용하는 '제로샷 심투리얼 전환(zero-shot sim-to-real transfer)'을 조립과 같은 복잡한 작업에서 성공적으로 시연하며 기술력을 입증했다.93

## 5. V. 전략적 전망, 위험 및 권장 사항

이 마지막 섹션은 보고서의 분석 결과를 종합하여 엔비디아의 전략, 직면한 주요 위험, 그리고 이해관계자들을 위한 실행 가능한 권장 사항에 대한 미래 지향적 분석을 제공한다.

### 5.1  엔비디아의 지배 경로: 생태계의 힘

엔비디아의 수평적 풀스택 전략은 강력하다. 실리콘부터 시뮬레이션, 파운데이션 모델에 이르는 필수적인 구성 요소를 제공함으로써, 엔비디아는 로보틱스 산업 대부분의 기업에게 없어서는 안 될 파트너로 자리매김하고 있다. 이 전략은 네트워크 효과와 높은 전환 비용을 통해 강력한 해자를 구축한다. 일단 한 기업이 아이작 플랫폼 기반으로 개발 파이프라인을 구축하면, 다른 생태계로 이전하는 것은 막대한 비용과 시간이 소요될 것이다. 이는 AI/HPC 분야에서 CUDA가 거둔 성공을 재현하려는 시도다.

### 5.2  주요 위험 및 역풍

- **기술적 위험:** 심투리얼 격차는 해결되고 있지만 아직 완전히 극복되지는 않았다.89 비정형적인 실제 환경에서 강건하고 안전한 범용 AI를 만드는 것은 여전히 거대한 과제다. 여기서 의미 있는 진전을 이루지 못하면 시장 전체가 정체될 수 있다.

- **경쟁적 위험:** 가장 큰 위협은 테슬라와 같이 고도로 집중된 수직 통합 기업으로부터 온다.44 만약 테슬라가 자사의 제조 규모와 통합 AI 스택을 활용하여 엔비디아 기반 생태계보다 더 저렴하고 성능 좋은 로봇을 더 빨리 생산해낸다면, 시장을 장악하여 플랫폼 접근법의 영향력을 약화시킬 수 있다.

- **시장 위험:** 높은 초기 비용과 불확실한 ROI는 여전히 도입의 주요 장벽이다.82 경기 침체나 초기 파일럿 프로그램에서 명확하고 정량화 가능한 이점을 입증하지 못할 경우, 투자와 도입이 급격히 둔화되는 "로보틱스의 겨울"이 올 수 있다.

- **규제 위험:** 자율 로봇의 등장은 중대한 윤리적, 법적 문제를 야기한다.5 특히 **EU AI 법(EU AI Act)**과 같은 규제 프레임워크는 큰 영향을 미칠 것이다. AI 기반 로봇, 특히 의료(로봇 보조 수술)나 핵심 인프라에 사용되는 로봇은 

  **"고위험" AI 시스템**으로 분류될 가능성이 높다.97 이는 공급자와 배포자에게 엄격한 의무를 부과한다. 이러한 규제 준수 부담은 상당하며, 법률, 기술, 재정적 자원이 부족한 스타트업에게는 높은 진입 장벽으로 작용할 것이다. 이는 결국 규제 환경에 잘 대처할 수 있는 소수의 거대 플랫폼 및 기업 중심의 시장 통합을 촉진할 수 있다.

**표 4: 고위험 로봇 시스템에 대한 EU AI 법 의무 사항**

| 의무 범주                      | 공급자/배포자 요구 사항                                      |
| ------------------------------ | ------------------------------------------------------------ |
| **위험 관리**                  | 시스템의 전 생애주기에 걸쳐 위험 평가 및 완화 시스템을 수립하고 문서화해야 함. |
| **데이터 거버넌스**            | 차별적 결과를 최소화하기 위해 고품질의 훈련, 검증, 테스트 데이터셋을 사용해야 함. |
| **기술 문서**                  | 당국이 규정 준수를 평가할 수 있도록 시스템의 목적과 기능에 대한 상세한 기술 문서를 작성해야 함. |
| **기록 보관 (로깅)**           | 결과의 추적성을 보장하기 위해 시스템 활동을 자동으로 기록(로그)해야 함. |
| **투명성 및 정보 제공**        | 배포자가 시스템을 이해하고 사용할 수 있도록 명확하고 적절한 정보를 제공해야 함. |
| **인간 감독**                  | 시스템의 위험을 최소화하거나 방지하기 위해 적절한 인간 감독 조치를 설계하고 구축해야 함. |
| **정확성, 강건성, 사이버보안** | 높은 수준의 정확성, 기술적 강건성, 사이버보안을 보장하도록 설계 및 개발되어야 함. |

출처: 97

### 5.3  결론 분석 및 미래 지향적 권장 사항

- **투자자에게:** 휴머노이드 로보틱스 분야는 고위험 고수익 영역이다. 핵심 경쟁은 엔비디아의 수평적 플랫폼과 테슬라의 수직 통합 전략 간의 대결이다. 엔비디아에 대한 투자는 전체 생태계의 성장에 대한 베팅이며, 특정 로봇 회사에 대한 투자는 그들의 하드웨어 및 애플리케이션 개발 실행 능력에 대한 베팅이다. 피규어와 OpenAI의 파트너십 결렬은 긴밀하게 통합된 자체 AI 스택을 보유한 회사가 장기적으로 유리할 수 있음을 시사한다. 또한, 어떤 로봇 회사가 성공하든 고성능 액추에이터, 센서, 배터리, 그리고 AI 훈련을 위한 클라우드 컴퓨팅과 같은 핵심 부품 및 인프라 공급업체는 수혜를 입을 가능성이 높다. 따라서 "곡괭이와 삽"에 투자하는 것, 즉 핵심 기반 기술에 투자하는 것이 단일 로봇 제조업체에 베팅하는 것보다 낮은 위험으로 시장 성장에 참여할 수 있는 방법이다.
- **로보틱스 기업(엔비디아 파트너)에게:** 가장 큰 과제는 차별화다. 모두가 동일한 GR00T 파운데이션 모델과 아이작 도구를 사용한다면 어떻게 독자적인 가치를 창출할 것인가? 핵심은 특정 작업과 산업 분야에 대한 독점 데이터로 GR00T 모델을 특화하고, 하드웨어 설계(예: 액추에이터, 배터리 수명, 엔드 이펙터)를 혁신하며, 우수한 시스템 통합 및 고객 지원을 개발하는 데 있다. 엔비디아 생태계를 깊이 활용하여 R&D를 가속화하는 것은 중요하지만, 플랫폼 위에 독자적인 가치를 더하는 명확한 전략이 동반되어야 한다.
- **경쟁사(예: 테슬라)에게:** 주요 과제는 속도와 규모다. 수직 통합 접근 방식은 장기적으로 우수한 제품을 낳을 수 있지만, 모든 것을 자체 개발하는 것은 더 느리고 자본 집약적이다. 엔비디아 생태계는 수십 개의 회사가 병렬적으로 혁신할 수 있게 한다. 테슬라는 엔비디아 기반 생태계가 여러 산업에 걸쳐 다양한 전문 솔루션의 임계 질량을 달성하기 전에, 자사의 제조 역량을 활용하여 매력적인 가격대의 제품을 대량 생산 체제에 올려놓아야 한다.

궁극적으로, 현재의 기술 로드맵은 자율성 달성에 초점을 맞추고 있지만, 간병이나 서비스 역할에서 로봇이 진정으로 사회에 통합되기 위해서는 인간-로봇 상호작용 모델이 무엇보다 중요하다. 모호한 인간의 지시를 처리하고, 협업하며, 직관적인 감독을 받는 능력, 즉 '인간-참여형(human-in-the-loop)' 문제를 해결하는 기업이 가장 큰 소비자 및 서비스 시장을 열게 될 것이다.

#### **참고 자료**

1. NVIDIA Announces Project GR00T Foundation Model for Humanoid Robots and Major Isaac Robotics Platform Update, accessed July 18, 2025, https://nvidianews.nvidia.com/news/foundation-model-isaac-robotics-platform
2. Project GR00T Foundation Model for Humanoid Robots ... - eeNews Europe, accessed July 18, 2025, https://www.eenewseurope.com/en/project-gr00t-foundation-model-for-humanoid-robots/
3. Nvidia unveils Project GR00T AI foundation model for humanoid robots - SiliconANGLE, accessed July 18, 2025, https://siliconangle.com/2024/03/18/nvidia-unveils-project-gr00t-ai-foundation-model-humanoid-robots-isaac-platform-updates/
4. nvidianews.nvidia.com, accessed July 18, 2025, [https://nvidianews.nvidia.com/news/foundation-model-isaac-robotics-platform#:~:text=Robots%20powered%20by%20GR00T%2C%20which,interact%20with%20the%20real%20world.](https://nvidianews.nvidia.com/news/foundation-model-isaac-robotics-platform#:~:text=Robots powered by GR00T%2C which,interact with the real world.)
5. Nvidia's Project GR00T - CyberPeace, accessed July 18, 2025, https://www.cyberpeace.org/resources/blogs/nvidias-project-gr00t
6. 엔비디아도 AI 로봇에 초점...휴머노이드용 LMM '프로젝트 그루트' 공개 - AI타임스, accessed July 18, 2025, https://www.aitimes.com/news/articleView.html?idxno=158089
7. NVIDIA Unveils Project GR00T and Isaac Robotics Platform Upgrade - Newo.ai, accessed July 18, 2025, https://newo.ai/nvidia-unveils-project-gr00t-and-isaac-robotics-platform-upgrade-revolutionizing-humanoid-robotics/
8. Nvidia의 Project GR00T - GeekNews, accessed July 18, 2025, https://news.hada.io/topic?id=13902
9. GR00T N1: An Open Foundation Model for Generalist Humanoid Robots - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2503.14734
10. NVIDIA Isaac GR00T N1: An Open Foundation Model for Humanoid Robots | Research, accessed July 18, 2025, https://research.nvidia.com/publication/2025-03_nvidia-isaac-gr00t-n1-open-foundation-model-humanoid-robots
11. GR00T N1: An Open Foundation Model for Generalist Humanoid Robots - Cloudfront.net, accessed July 18, 2025, https://d1qx31qr3h6wln.cloudfront.net/publications/GR00T_1_Whitepaper.pdf
12. NVIDIA Announces Isaac GR00T N1 — the World's First Open Humanoid Robot Foundation Model — and Simulation Frameworks to Speed Robot Development, accessed July 18, 2025, https://nvidianews.nvidia.com/news/nvidia-isaac-gr00t-n1-open-humanoid-robot-foundation-model-simulation-frameworks
13. Isaac GR00T N1: World's First Open Humanoid Robot Foundation Model From NVIDIA, accessed July 18, 2025, https://circuitdigest.com/news/isaac-gr00t-n1-worlds-first-open-humanoid-robot-foundation-model-from-nvidia
14. Nvidia positions GR00T N1 to dominate robotics ecosystem - The Decoder, accessed July 18, 2025, https://the-decoder.com/nvidia-positions-gr00t-n1-to-dominate-robotics-ecosystem/
15. GR00T N1: An Open Foundation Model for Generalist Humanoid Robots - arXiv, accessed July 18, 2025, https://arxiv.org/html/2503.14734v2
16. GR00T N1.5 Explained: NVIDIA's VLA Model for Humanoid Robots - LearnOpenCV, accessed July 18, 2025, https://learnopencv.com/gr00t-n1_5-explained/
17. NVIDIA Powers Humanoid Robot Industry With Cloud-to-Robot Computing Platforms for Physical AI, accessed July 18, 2025, https://nvidianews.nvidia.com/news/nvidia-powers-humanoid-robot-industry-with-cloud-to-robot-computing-platforms-for-physical-ai
18. NVIDIA unveils new Isaac GR00T tools powering robot evolution - IT Brief Asia, accessed July 18, 2025, https://itbrief.asia/story/nvidia-unveils-new-isaac-gr00t-tools-powering-robot-evolution
19. R²D²: Training Generalist Robots with NVIDIA Research Workflows and World Foundation Models, accessed July 18, 2025, https://developer.nvidia.com/blog/r2d2-training-generalist-robots-with-nvidia-research-workflows-and-world-foundation-models/
20. NVIDIA's Project GR00T, Running on the Jetson Thor, Aims to Deliver Embodied AI for Humanoid Robots - Hackster.io, accessed July 18, 2025, https://www.hackster.io/news/nvidia-s-project-gr00t-running-on-the-jetson-thor-aims-to-deliver-embodied-ai-for-humanoid-robots-a02741d94811
21. NVIDIA announces new robotics products at GTC 2024 - The Robot Report, accessed July 18, 2025, https://www.therobotreport.com/nvidia-announces-new-robotics-products-at-gtc-2024/
22. NVIDIA® Jetson AGX Thor™ Developer Kit - EDOM Technology, accessed July 18, 2025, https://www.edomtech.com/en/product-detail/nvidia-jetson-agx-thor-dev-kit/
23. NVIDIA Jetson AGX Thor Developer Kit - Silicon HIghway, accessed July 18, 2025, https://www.siliconhighway.com/wp-content/robotics-and-edge-ai-datasheet-jetson-thor-devkit-nvidia-us-web.pdf
24. NVIDIA Jetson Thor: New SoC Features Guide - RidgeRun Developer Wiki, accessed July 18, 2025, https://developer.ridgerun.com/wiki/index.php/NVIDIA_Jetson_Thor:_Powering_the_Future_of_Physical_AI
25. NVIDIA - OpenZeka, accessed July 18, 2025, https://openzeka.com/wp-content/uploads/2025/07/NVIDIA_Jetson_Thor_Module_Datasheet.pdf
26. Isaac Sim - Robotics Simulation and Synthetic Data Generation - NVIDIA Developer, accessed July 18, 2025, https://developer.nvidia.com/isaac/sim
27. Exploring NVIDIA Isaac GR00T - Marvik - Blog, accessed July 18, 2025, https://blog.marvik.ai/2025/02/19/exploring-nvidia-isaac-gr00t/
28. Training in NVIDIA Isaac Sim Closes the Sim2Real Gap | NVIDIA Technical Blog, accessed July 18, 2025, https://developer.nvidia.com/blog/training-in-nvidia-isaac-sim-closes-the-sim2real-gap/
29. Streamline Data Collection With NVIDIA Isaac GR00T - YouTube, accessed July 18, 2025, https://www.youtube.com/watch?v=waZ08Z3uimk
30. Advancing Humanoid Robot Sight and Skill Development with NVIDIA Project GR00T, accessed July 18, 2025, https://developer.nvidia.com/blog/advancing-humanoid-robot-sight-and-skill-development-with-nvidia-project-gr00t/
31. Training Humanoid Robots with Isaac GR00T-Dreams Using Synthetic Data from World Foundation Models - YouTube, accessed July 18, 2025, https://www.youtube.com/watch?v=pMWL1MEI-gE
32. NVIDIA Project GR00T를 통한 휴머노이드 로봇 시력 및 기술 개발 발전 - AI넷, accessed July 18, 2025, http://www.ainet.link/18402
33. What is Project GR00T by NVIDIA in Simple Words - YouTube, accessed July 18, 2025, https://www.youtube.com/watch?v=pTIdva-kEEM
34. 엔비디아, 범용 휴머노이드 로봇 위한 파운데이션 모델 '프로젝트 GR00T' 발표 - 로봇신문, accessed July 18, 2025, https://www.irobotnews.com/news/articleView.html?idxno=34337
35. Nvidia's Project Gr00t: A leap forward in AI-powered robotics - Techgoondu, accessed July 18, 2025, https://www.techgoondu.com/2024/03/27/nvidias-project-gr00t-a-leap-forward-in-ai-powered-robotics/
36. Nvidia's Great Leap Forward: Will It Usher in a Robotics Golden Age—Or Spark a Tech Dystopia? - Robozaps Blog, accessed July 18, 2025, https://blog.robozaps.com/b/nvidias-next-frontier
37. Nvidia's Project GR00T Promises to Transform Humanoid Robot Learning - Pure AI, accessed July 18, 2025, https://pureai.com/articles/2024/03/20/nvidia-annos-gr00t.aspx
38. 1X & NVIDIA Research Collaboration, accessed July 18, 2025, https://www.1x.tech/discover/1X-NVIDIA-Research-Collaboration
39. Agility Robotics Expands Relationship with NVIDIA, accessed July 18, 2025, https://www.agilityrobotics.com/content/agility-robotics-expands-relationship-with-nvidia
40. Boston Dynamics plans to use NVIDIA's Isaac GR00T to build AI capabilities for Atlas, accessed July 18, 2025, https://www.therobotreport.com/boston-dynamics-plans-to-use-nvidias-isaac-gr00t-to-build-ai-capabilities-for-atlas/
41. Boston Dynamics Expands Collaboration with NVIDIA to Accelerate AI Capabilities in Humanoid Robots, accessed July 18, 2025, https://bostondynamics.com/news/boston-dynamics-expands-collaboration-with-nvidia/
42. AI & Robotics | Tesla, accessed July 18, 2025, https://www.tesla.com/AI
43. Figure AI - Wikipedia, accessed July 18, 2025, https://en.wikipedia.org/wiki/Figure_AI
44. Nvidia GROOT vs. Tesla Optimus: Competing Paths to Humanoid Robots -, accessed July 18, 2025, https://www.supplychaintoday.com/nvidia-groot-vs-tesla-optimus-competing-paths-to-humanoid-robots/
45. Figure AI: Robot startup terminates partnership with OpenAI, relies on its own LLMs, accessed July 18, 2025, https://www.trendingtopics.eu/figure-ai-robot-startup-terminates-partnership-with-openai-relies-on-its-own-llms/
46. Tesla Robot: What It Is, What Makes It Unique, and Its Future Prospects - THE WAVES, accessed July 18, 2025, https://www.the-waves.org/2025/01/05/tesla-robot-what-it-is-what-makes-it-unique-and-its-future-prospects/
47. Nvidia GROOT vs. Tesla Optimus: Competing Paths to Humanoid Robots - YouTube, accessed July 18, 2025, https://www.youtube.com/watch?v=vHKNqFyhXyE
48. A Complete Review Of Tesla's Optimus Robot - Brian D. Colwell, accessed July 18, 2025, https://briandcolwell.com/a-complete-review-of-teslas-optimus-robot/
49. Manifested AI Signals Major Shift in Robotics: Brownstone Research Analyzes Tesla's 2025 Automation Strategy | Newswire, accessed July 18, 2025, https://www.newswire.com/news/manifested-ai-signals-major-shift-in-robotics-brownstone-research-22611713
50. Optimus (robot) - Wikipedia, accessed July 18, 2025, https://en.wikipedia.org/wiki/Optimus_(robot)
51. Tesla's Robot, Optimus: Everything We Know | Built In, accessed July 18, 2025, https://builtin.com/robotics/tesla-robot
52. Elon Musk provides an update on Tesla's Optimus Robot: 'There will be a…' - Times of India, accessed July 18, 2025, https://timesofindia.indiatimes.com/technology/social/elon-musk-provides-an-update-on-teslas-optimus-robot-there-will-be-a/articleshow/119912214.cms
53. Tesla Optimus | Electrek, accessed July 18, 2025, https://electrek.co/guides/tesla-optimus/
54. Report: Figure Business Breakdown & Founding Story - Contrary Research, accessed July 18, 2025, https://research.contrary.com/company/figure
55. Successful test of humanoid robots at BMW Group Plant Spartanburg, accessed July 18, 2025, https://www.press.bmwgroup.com/global/article/detail/T0444265EN/successful-test-of-humanoid-robots-at-bmw-group-plant-spartanburg?language=en
56. Humanoid Figure 02 robots tested at BMW Group Plant Spartanburg - YouTube, accessed July 18, 2025, https://www.youtube.com/watch?v=xLVm-QKEZSI
57. AI robot assembles BMW, accessed July 18, 2025, https://www.therundown.ai/p/ai-robot-assembles-bmw
58. The Future Is Now: UPS in Talks With Figure AI to Use Humanoid Robots - YouTube, accessed July 18, 2025, https://www.youtube.com/watch?v=BJo3WM9RggE
59. Figure AI Reportedly Partnering with UPS to Bring Humanoid Robots to Logistics Operations, accessed July 18, 2025, https://www.tomorrowsworldtoday.com/robotics/figure-ai-reportedly-partnering-with-ups-to-bring-humanoid-robots-to-logistics-operations/
60. “They Just Gave Robots a Brain”: OpenAI Powers New Humanoid from Figure as Tech Giants Rush to Fund the Future of Labor - Rude Baguette, accessed July 18, 2025, https://www.rudebaguette.com/en/2025/07/they-just-gave-robots-a-brain-openai-powers-new-humanoid-from-figure-as-tech-giants-rush-to-fund-the-future-of-labor/
61. OpenAI Partners with Figure: Integrating AI into Humanoid Robots for Everyday Life, accessed July 18, 2025, https://thegreatentrepreneurs.com/openai-partners-with-figure-integrating-ai-into-humanoid-robots-for-everyday-life/
62. Figure AI announced the termination of cooperation with OpenAI, claiming to have made breakthroughs in the field of robot AI. | DailyNews on Gate Square, accessed July 18, 2025, https://www.gate.com/post/status/9102818
63. Figure AI Ends OpenAI Partnership to Focus on In-House Robot Intelligence - Maginative, accessed July 18, 2025, https://www.maginative.com/article/figure-ai-ends-openai-partnership-to-focus-on-in-house-robot-intelligence/
64. Figure AI Ends Partnership With OpenAI After Breakthrough - Binance, accessed July 18, 2025, https://www.binance.com/en/square/post/02-05-2025-figure-ai-ends-partnership-with-openai-after-breakthrough-19897458486458
65. Robotics startup Figure AI drops OpenAI because LLMs are 'getting smarter yet more commoditized' - The Decoder, accessed July 18, 2025, https://the-decoder.com/robotics-startup-figure-ai-drops-openai-because-llms-are-getting-smarter-yet-more-commoditized/
66. Figure AI and OpenAI Break Up | Mike Kalil, accessed July 18, 2025, https://mikekalil.com/blog/figure-openai-breakup/
67. Spotlight on humanoids: A deep dive into Figure AI - Robotics & Automation News, accessed July 18, 2025, https://roboticsandautomationnews.com/2025/05/06/spotlight-on-humanoids-a-deep-dive-into-figure-ai/90373/
68. Humanoid Robot Market Size, Share, Industry Report, 2025 To 2030 - MarketsandMarkets, accessed July 18, 2025, https://www.marketsandmarkets.com/Market-Reports/humanoid-robot-market-99567653.html
69. Here are relevant reports on : humanoid-robot-market - MarketsandMarkets, accessed July 18, 2025, https://www.marketsandmarkets.com/Market-Reports/humanoid-robot-market-253648069.html
70. Humanoid Robots Market Redefine Automation Across Industries From 2025 To 2030, accessed July 18, 2025, https://www.marketsandmarketsblog.com/humanoid-robots-market-redefine-automation-across-industries-from-2025-to-2030.html
71. Humanoid Robot Market: Global Industry Analysis and Forecast (2025-2032), accessed July 18, 2025, https://www.maximizemarketresearch.com/market-report/global-humanoid-robot-market/10567/
72. Humanoid Robot Market Expected to Reach $5 Trillion by 2050 | Morgan Stanley, accessed July 18, 2025, https://www.morganstanley.com/insights/articles/humanoid-robot-market-5-trillion-by-2050
73. Humanoid Robot Market: The Future of Human-Machine Collaboration - Lucintel, accessed July 18, 2025, https://www.lucintel.com/lucintel-brief/ArtificialIntelligence/Humanoid-Robot-Market-The-Future-of-Human-Machine-Collaboration.aspx
74. Humanoid Robots in Logistics: Bridging the Labor Gap, accessed July 18, 2025, https://www.jusdaglobal.com/en/article/humanoid-robots-logistics-bridging-labor-gap/
75. Global Humanoid Robot Market Accelerates Toward $4.04 Billion by 2030 at 17.5% CAGR- Exclusive Report by The Research Insights - PR Newswire, accessed July 18, 2025, https://www.prnewswire.com/news-releases/global-humanoid-robot-market-accelerates-toward-4-04-billion-by-2030-at-17-5-cagr--exclusive-report-by-the-research-insights-302488205.html
76. Humanoid Robots Market: Shaping the Future of Healthcare, Education, and Automation, accessed July 18, 2025, https://www.roboticstomorrow.com/news/2025/01/24/humanoid-robots-market-shaping-the-future-of-healthcare-education-and-automation/23992/
77. Humanoid Robot Market Size & Share | Industry Report, 2030 - Grand View Research, accessed July 18, 2025, https://www.grandviewresearch.com/industry-analysis/humanoid-robot-market-report
78. Humanoid Robots Market Size, Share & Trends Report 2025-2030 - GlobeNewswire, accessed July 18, 2025, https://www.globenewswire.com/news-release/2025/04/15/3061450/28124/en/Humanoid-Robots-Market-Size-Share-Trends-Report-2025-2030-Market-to-Grow-at-17-5-CAGR-Through-2030-as-Demand-Surges-Across-Healthcare-and-Defense-Sectors.html
79. Healthcare Humanoid Robot Market Size & Share | Growth Report 2037 - Research Nester, accessed July 18, 2025, https://www.researchnester.com/reports/healthcare-humanoid-robot-market/6565
80. Humanoid Robots in Manufacturing: Transforming Industrial Automation - Fictiv, accessed July 18, 2025, https://www.fictiv.com/articles/humanoid-robotics-manufacturing-impact
81. Transformation of Warehouses and Manufacturing: How Humanoid Robots Will Change Automotive Supply Chains | Deloitte, accessed July 18, 2025, https://www.deloitte.com/cz-sk/en/Industries/automotive/blogs/transformation-of-warehouses-and-manufacturing-how-humanoid-robots-will-change-automotive-supply-chains.html
82. Humanoid Robots in Manufacturing: The Next Frontier in Industrial Automation - iFactory, accessed July 18, 2025, https://ifactoryapp.com/blog/humanoid-robots-manufacturing-industrial-automation-future
83. Fast Growth of Humanoid Robots in the Automotive & Logistics Industry - IDTechEx, accessed July 18, 2025, https://www.idtechex.com/en/research-article/fast-growth-of-humanoid-robots-in-the-automotive-and-logistics-industry/33181
84. Case Studies: Calculating robot ROI | Shibaura Machine, accessed July 18, 2025, https://www.automate.org/robotics/case-studies/calculating-robot-roi
85. Unveiling the ROI and Cost-Benefit Analysis of Autonomous Mobile Robots, accessed July 18, 2025, https://www.quicktron.com/blogs/188
86. Humanoid Healthcare Assistive Robot Market CAGR Of 27.8%, accessed July 18, 2025, https://market.us/report/global-humanoid-healthcare-assistive-robot-market/
87. Topic Brief:Cost and Effectiveness of Surgical Robots, accessed July 18, 2025, https://effectivehealthcare.ahrq.gov/system/files/docs/topic-brief-robotic-surgery.pdf
88. Cost and Effectiveness of Surgical Robots | Effective Health Care (EHC) Program - AHRQ, accessed July 18, 2025, https://effectivehealthcare.ahrq.gov/get-involved/nominated-topics/surgical-robots
89. Sim2Real Gap: Why Machine Learning Hasn't Solved Robotics Yet - Inbolt, accessed July 18, 2025, https://www.inbolt.com/resources/sim2real-gap-why-machine-learning-hasnt-solved-robotics-yet
90. Why the sim2real problem in robotic manipulation? : r/reinforcementlearning - Reddit, accessed July 18, 2025, https://www.reddit.com/r/reinforcementlearning/comments/10vkqg8/why_the_sim2real_problem_in_robotic_manipulation/
91. On Sim2Real Transfer in Robotics （Part 1/3) | Haonan's blog, accessed July 18, 2025, https://www.haonanyu.blog/post/sim2real/
92. Closing the Sim2Real Gap with NVIDIA Isaac Sim and NVIDIA Isaac Replicator | NVIDIA Technical Blog - NVIDIA Developer, accessed July 18, 2025, https://developer.nvidia.com/blog/closing-the-sim2real-gap-with-nvidia-isaac-sim-and-nvidia-isaac-replicator/
93. Training Sim-to-Real Transferable Robotic Assembly Skills over Diverse Geometries, accessed July 18, 2025, https://developer.nvidia.com/blog/training-sim-to-real-transferable-robotic-assembly-skills-over-diverse-geometries/
94. [2501.02902] Sim-to-Real Transfer for Mobile Robots with Reinforcement Learning: from NVIDIA Isaac Sim to Gazebo and Real ROS 2 Robots - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2501.02902
95. The Importance and the Limitations of Sim2Real for Robotic Manipulation in Precision Agriculture, accessed July 18, 2025, https://sim2real.github.io/assets/papers/2020/rizzardo.pdf
96. Nvidia's Project GR00T vs. Tesla Optimus: Competing Robot Strategies - CNET, accessed July 18, 2025, https://www.cnet.com/videos/nvidias-project-gr00t-vs-tesla-optimus-competing-robot-strategies/
97. AI Act | Shaping Europe's digital future, accessed July 18, 2025, https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai




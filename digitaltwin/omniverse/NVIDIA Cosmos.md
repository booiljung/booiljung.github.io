[NVidia Omniverse](./index.md)
# NVIDIA Cosmos


이 파트에서는 NVIDIA Cosmos의 존재 이유를 규명한다. 이 플랫폼을 단일 제품이 아닌, 차세대 AI의 근본적인 병목 현상에 대한 전략적 대응이자 수십조 달러 규모의 시장 기회를 포착하기 위한 계산된 움직임으로 분석한다.



인공지능(AI)의 패러다임은 텍스트나 이미지와 같은 디지털 정보를 처리하는 '디지털 AI'에서 물리적 세계를 인식하고, 추론하며, 상호작용해야 하는 '물리적 AI(Physical AI)'로 전환되고 있다.1 물리적 AI 시스템은 센서와 액추에이터를 장착하여 환경을 관찰하고 수정할 수 있다는 점에서 근본적인 차이를 보인다. 이러한 특성은 디지털 AI에는 존재하지 않았던 물리 법칙, 인과관계, 안전성과 같은 새로운 과제들을 제기한다.2 자율주행차(AV)가 보행자의 움직임을 예측하고 실시간으로 경로를 수정해야 하거나, 창고의 로봇이 장애물을 피하고 정밀하게 물체를 조작해야 하는 것처럼, 물리적 AI는 현실 세계의 복잡성과 불확실성에 직접적으로 대응해야 한다.5


물리적 AI 개발의 가장 큰 걸림돌은 바로 데이터 문제이다. 견고한 물리적 AI를 개발하기 위해서는 방대한 양의 실제 세계 훈련 데이터가 필요하지만, 이를 수집하는 과정은 엄청난 비용, 시간, 그리고 위험을 동반한다.5 실제로 수집된 가장 큰 규모의 데이터셋조차도 범용적인 능력을 갖추기 위해 필요한 데이터 양에 비하면 "상상할 수 없을 정도로 미미한 수준"에 불과하며, 이는 자율주행차 및 로봇 공학 개발에 심각한 병목 현상을 초래한다.8 이러한 데이터 부족 현상은 로봇 공학 분야에서 "ChatGPT 모멘트"가 아직 도래하지 못한 근본적인 원인으로 지목된다.7 실제 환경에서 드물지만 치명적인 '엣지 케이스(edge case)'를 충분히 수집하는 것은 거의 불가능에 가깝기 때문이다.


이러한 데이터 문제를 해결하기 위한 전략적 대안으로 '월드 파운데이션 모델(World Foundation Model, WFM)'이라는 개념이 부상했다.2 WFM은 AI가 안전하게 상호작용할 수 있는 "물리적 세계의 디지털 트윈"으로 정의되며, 가상 환경에서 확장 가능하고, 반복적이며, 안전한 훈련을 가능하게 한다.2 이는 개발자들이 실제 세계의 위험과 비용 부담 없이 다양한 시나리오를 시뮬레이션하고 AI 모델을 테스트할 수 있게 해준다. 대규모 언어 모델(LLM)이 생성형 AI의 잠재력을 폭발시킨 것처럼, WFM은 물리적 AI의 잠재력을 실현하는 데 필요한 핵심 기반 기술로 자리매김하고 있다.9



NVIDIA의 CEO 젠슨 황(Jensen Huang)은 Cosmos를 LLM이 자연어 처리에 가져온 혁신에 비견되는, 로봇 공학의 돌파구를 마련할 기술로 전략적으로 포지셔닝하고 있다.7 이는 단순한 마케팅 수사를 넘어, 차세대 AI 물결의 기초 계층(foundational layer)을 장악하겠다는 명확한 의지의 표명이다. 대부분의 개발자들이 자체적으로 구축할 자원이 부족한, 훈련 비용이 막대한 파운데이션 모델을 제공함으로써 "물리적 AI를 민주화"하는 것이 Cosmos의 핵심 목표이다.7


Cosmos는 NVIDIA 플랫폼 전략의 세 번째 기둥으로 볼 수 있다. 첫 번째 기둥이 가속 컴퓨팅을 위한 CUDA였다면, 두 번째는 디지털 및 생성형 AI를 위한 AI Enterprise/NeMo 스택이었고, 세 번째인 Cosmos는 물리적 AI를 위한 플랫폼을 대표한다.6 이러한 구조는 하드웨어에 대한 지속적인 수요를 창출하는 포괄적인 엔드투엔드(end-to-end) 생태계를 구축한다.6 NVIDIA는 단순히 개별 제품을 판매하는 것이 아니라, 자사의 하드웨어와 소프트웨어가 긴밀하게 통합된 전체 플랫폼을 통해 시장 지배력을 강화하는 전략을 일관되게 추진해왔다.


NVIDIA는 물리적 AI가 **50조 달러 규모의 제조 및 물류 산업**을 혁신할 것이라고 전망하며 Cosmos의 목표 시장을 명확히 하고 있다.9 이는 Cosmos가 일부 개발자를 위한 틈새 도구가 아니라, "움직이는 모든 것... 로봇화되고 AI에 의해 구현될" 세계 경제의 가장 큰 부분을 차지하는 산업으로 진입하기 위한 전략적 교두보임을 시사한다.9 이 거대한 시장을 디지털화하는 데 필요한 기초 운영체제를 제공함으로써, NVIDIA는 미래 산업 자동화의 중심에 서고자 한다.

이러한 전략적 배경을 분석하면, Cosmos는 단순히 합성 데이터를 생성하는 제품이 아니라, 새로운 시장을 위한 운영 체제(OS)라는 결론에 도달한다. NVIDIA는 물리적 AI 산업 전체가 구축될 기반 계층, 즉 "물리적 세계를 디지털화하기 위한 디지털-투-피지컬 운영 체제"를 만들고 있다.13 이는 특정 애플리케이션이 아닌 생태계 전체를 장악하려는 전형적인 플랫폼 전략이다. NVIDIA의 역사를 보면, CUDA라는 소프트웨어 계층을 통해 자사 하드웨어를 필수불가결하게 만들었던 성공 사례가 있다.6 Cosmos는 이 전략을 물리적 AI 영역으로 확장한 것이다. 데이터 큐레이션(NeMo), 시뮬레이션(Omniverse), 배포(NIM, Jetson)를 통합하는 플랫폼의 포괄적인 설계는 단순한 모델 저장소 이상의 것을 지향한다.11 따라서 전략적 목표는 Cosmos 라이선스 판매가 아니라, 클라우드의 DGX부터 엣지의 AGX Orin에 이르는 NVIDIA의 전체 스택을 50조 달러 규모의 산업 자동화 시장을 위한 기본 인프라로 만드는 것이다.9

또한, "물리적 AI의 민주화"는 생태계 종속(lock-in)을 가속화하기 위한 전략적 움직임으로 해석될 수 있다. NVIDIA는 핵심 WFM을 개방형 모델 라이선스로 제공함으로써 스타트업과 연구자들의 진입 장벽을 낮춘다.1 이는 NVIDIA 스택을 기반으로 구축하는 광범위한 개발자 커뮤니티를 육성하여 경쟁자들이 뚫기 어려운 네트워크 효과를 창출한다. WFM을 훈련하는 것은 9,000조 개의 토큰과 수천 개의 GPU를 사용하는 등 엄청난 비용이 드는 작업이다.3 NVIDIA는 이 모델들을 오픈소스로 공개함으로써 이러한 초기 장벽을 제거하고 광범위한 채택을 유도한다.1 그러나 이 모델들은 NVIDIA 하드웨어(Ada/Blackwell 아키텍처의 FP8)에 최적화되어 있으며, Omniverse, Isaac, NeMo와 같은 NVIDIA의 독점 생태계와 깊이 통합되어 있다.3 이는 거대한 규모의 '프리미엄(freemium)' 모델과 같다. 즉, 모델 자체는 무료이지만, 이를 효과적으로 맞춤화하고, 배포하며, 확장하기 위해서는 개발자들이 유료인 NVIDIA 생태계(DGX Cloud, AI Enterprise)로 유입될 수밖에 없다. 이 전략은 NVIDIA 플랫폼 전체의 채택을 가속화하고 시장 지배력을 공고히 하는 역할을 한다.


이 파트에서는 플랫폼의 아키텍처를 기술적으로 심층 분석하여, 전체 스택에서부터 핵심 AI 모델의 특정 기능에 이르기까지 Cosmos가 어떻게 작동하는지를 설명한다.



Cosmos는 최첨단 생성형 WFM, 고급 비디오 토크나이저, 데이터 처리 및 큐레이션 파이프라인(NVIDIA NeMo Curator), 그리고 안전 가드레일로 구성된 엔드투엔드 플랫폼이다.2 이 플랫폼은 CUDA를 기반으로 구축되었으며, 데이터 큐레이션부터 모델 후처리 훈련 및 배포에 이르는 개발의 모든 단계를 가속화하도록 설계되었다.3 개발자는 이 통합된 환경을 통해 물리적 AI 모델을 더 빠르고 효율적으로 구축하고, 실제 환경에서의 테스트 및 검증 위험을 최소화할 수 있다.15


Cosmos WFM은 두 가지 핵심적인 AI 아키텍처를 기반으로 한다. 이는 개발자에게 유연성을 제공하여 특정 충실도 및 제어 요구 사항에 가장 적합한 모델을 선택할 수 있게 한다.

- **자기회귀 모델(Autoregressive Models):** 이 모델은 비디오를 순차적인 "다음 토큰 예측" 문제로 접근하여, 과거 프레임을 조건으로 한 조각씩 생성한다.2 이 방식은 시간적 일관성을 유지하는 데 강점이 있다. 기술적으로는 공간적 및 시간적 차원을 정밀하게 인코딩하기 위한 3D RoPE(Rotary Position Embeddings)와 텍스트 입력을 통한 제어를 가능하게 하는 교차 어텐션(cross-attention) 레이어와 같은 기술을 활용한다.3

- **확산 모델(Diffusion Models):** 이 모델은 훈련 데이터에 점진적으로 가우시안 노이즈를 추가하여 순수한 노이즈로 변환한 다음, 이 과정을 역으로 학습하여 노이즈가 낀 입력에서 원본 데이터를 복원하는 반복적인 노이즈 제거 프로세스를 사용한다.3

  `Cosmos-1.0-Diffusion-7B-Text2World`와 같은 모델이 이 아키텍처를 사용하며, 복잡한 장면 생성에 강점을 보인다.24


토크나이저는 종종 간과되지만, 대규모 모델 훈련에서 결정적인 역할을 하는 핵심 구성 요소이다. Cosmos 토크나이저 제품군은 이미지와 비디오를 모델 훈련에 효율적인 표현(토큰)으로 변환하는 최첨단 기술이다.15 기존의 주요 토크나이저보다 8배 더 높은 압축률과 12배 더 빠른 처리 속도를 제공함으로써, 방대한 WFM의 훈련을 계산적으로 실현 가능하게 만드는 중요한 역할을 한다.18 이는 데이터 처리 파이프라인의 효율성을 극대화하여 개발 비용과 시간을 크게 절감시킨다.


Cosmos 플랫폼의 핵심은 각각 고유한 기능을 수행하는 세 가지 주요 WFM 제품군으로 구성된다. 이들은 상호 보완적으로 작동하여 포괄적인 합성 데이터 생성 및 관리 워크플로우를 형성한다.


이 모델의 주된 기능은 텍스트, 이미지, 비디오와 같은 다중 모드 입력으로부터 미래의 세계 상태를 생성하는 것이다.6 핵심 역량은 "예지력(foresight)"으로, 주어진 시작 프레임과 끝 프레임 사이의 중간 동작이나 모션 궤적을 예측한다. 이는 엣지 케이스를 시뮬레이션하여 데이터를 증강하거나, 다양한 행동의 결과를 시뮬레이션하여 정책을 평가하는 데 매우 중요하다.1 초기 버전인 Predict-1(40억~150억 파라미터)에서 Predict-2로 발전하면서 충실도, 제어 능력, 환각 현상 감소 등에서 상당한 성능 향상을 이루었다.15


이 모델은 구조화되고 사실적이지 않은 시뮬레이션 데이터와 사실적인 비디오 출력 사이의 다리 역할을 한다.6 NVIDIA Omniverse와 같은 시뮬레이터에서 생성된 세그멘테이션 맵, 깊이 신호, LiDAR 스캔과 같은 입력을 받아 사실적인 세계 장면으로 변환한다.6

- **기술 심층 분석:** Cosmos Transfer의 핵심 기술은 **ControlNet 및 MultiControlNet 아키텍처**이다.26 ControlNet은 단일 모달리티(예: 깊이)에 대한 세밀한 조건부 생성을 가능하게 하는 반면, MultiControlNet은 여러 입력의 조합을 조건으로 사용할 수 있게 한다. 특히 **시공간 제어 맵(spatiotemporal control map)**을 통해 각 모달리티가 시간과 공간에 따라 미치는 영향을 정밀하게 제어할 수 있다.26 이것이 바로 "제어 가능한" 합성 데이터 생성을 가능하게 하는 핵심 기술이다.


Cosmos Reason은 인지 능력을 도입한 가장 진보된 구성 요소로 평가받는다. 이 모델은 합성 데이터를 위한 비평가 역할을 하는, 완전히 맞춤화 가능한 다중 모드 추론 모델이다.6

**연쇄적 사고(Chain-of-Thought, CoT) 추론**을 사용하여 합성된 시각 자료를 평가하고, "선반에서 상자가 떨어지는" 것과 같은 결과를 예측하며, 최적의 응답에 대해 자연어로 보상을 제공한다.6 이 기능은 데이터 큐레이션을 자동화하는 데 결정적이다. 즉, 사람의 주석 없이도 캡션을 생성하고, 물리적으로 불가능한 시나리오를 걸러내며, 강화 학습을 위한 피드백을 제공할 수 있다.15

이 세 가지 모델, 즉 Predict, Transfer, Reason은 단순한 모델의 집합이 아니라, 전체 합성 데이터 파이프라인을 자동화하도록 설계된 상호 연결된 시스템이다. 이들은 하나의 완전하고 자율적인 "데이터 공장" 플라이휠(flywheel)을 형성한다. 개발자는 먼저 Omniverse와 같은 시뮬레이터를 사용하여 구조화된 데이터(예: 객체 궤적, 세그멘테이션 맵)를 포함하는 기본적인 3D 장면을 생성한다.1 그 다음, **Cosmos Transfer**가 이 구조화된 데이터를 입력받아 조명, 질감, 날씨 등을 변화시키며 방대하고 다양한 사실적 비디오 세트로 증폭시킨다.15 이어서 **Cosmos Predict**는 이 비디오들의 프레임을 기반으로 그럴듯한 미래 시나리오를 생성하여 더 많은 데이터를 만들고 AI의 사건 예측 능력을 테스트한다.15 마지막으로, 

**Cosmos Reason**은 Transfer와 Predict의 결과물을 분석하여 데이터에 자동으로 캡션을 달고, 비현실적이거나 물리적으로 불가능한 생성물(예: 단단한 벽을 통과하는 자동차)을 걸러내며, 강화 학습을 통해 정책 모델을 훈련시키기 위한 보상 신호를 제공한다.15 이 과정은 시뮬레이션이 씨앗을 만들고, Transfer와 Predict가 이를 증식 및 다양화하며, Reason이 이를 큐레이션하고 검증하여 고품질 데이터를 최소한의 인간 개입으로 훈련 과정에 다시 공급하는 폐쇄 루프(closed loop)를 만든다. 이 플라이휠이야말로 Cosmos 가치 제안의 핵심 엔진이다.

또한, Cosmos 아키텍처는 "후처리 훈련(post-training)"을 위해 명시적으로 설계되었다는 점이 핵심적인 차별점이다. 범용 비디오 생성 모델과 달리, Cosmos WFM은 특정 "전문가" 애플리케이션을 위해 미세 조정(fine-tuning)될 것을 의도한 "일반론자(generalist)"로 구축되었다.15 NVIDIA는 Cosmos WFM이 "후처리 훈련을 위해 특별히 제작되었다"고 반복해서 강조한다.15 플랫폼은 모델뿐만 아니라 후처리 훈련 스크립트와 NeMo 프레임워크 접근 권한 등 이러한 맞춤화를 위한 도구도 함께 제공한다.15 이 전략은 단일 일반 모델이 모든 특정 로봇 공학이나 자율주행 문제를 해결할 수 없다는 현실을 인정한 것이다. 대신 NVIDIA는 개발자들의 막대한 비용을 절약해주는 강력한 사전 훈련된 기반을 제공하고, 이를 그들의 고유한 하드웨어, 환경, 작업에 맞게 조정할 수 있는 도구를 제공한다. 이로써 Cosmos는 경직된 단일 솔루션이 아닌 유연한 기반으로 자리매김하며, 휴머노이드 로봇(1X, Agility)부터 자율주행 트럭(Plus, Waabi)에 이르기까지 다양한 요구를 가진 광범위한 개발자들에게 더욱 매력적인 선택지가 된다.25


이 파트에서는 Cosmos가 독립적인 제품이 아니라, 훨씬 더 크고 깊이 통합된 하드웨어 및 소프트웨어 생태계의 "두뇌" 역할을 어떻게 수행하는지 분석한다. 이 생태계는 NVIDIA의 주요 경쟁 우위(competitive moat)이다.



두 플랫폼의 역할은 명확히 구분되면서도 상호 보완적이다.1 Omniverse는 물리적으로 정확한 3D 시뮬레이션 환경, 즉 디지털 트윈 운영 체제를 제공한다.1 그런 다음 Cosmos는 Omniverse에서 생성된 구조화된 데이터를 가져와 생성형 AI를 사용하여 이를 대규모의 사실적이고 다양한 훈련 데이터로 증폭시킨다.6 이 관계는 마치 물리 법칙을 시뮬레이션하는 엔진 위에, 그 세계를 무한히 확장하고 변형하는 창의적인 AI 엔진이 결합된 것과 같다.


이러한 원활한 상호 운용성을 가능하게 하는 핵심 기술은 Universal Scene Description(OpenUSD) 프레임워크이다. OpenUSD는 "3D 인터넷의 HTML"과 같은 중요한 공통 언어 역할을 하며, 다양한 3D 도구와 Omniverse/Cosmos 파이프라인 간의 데이터 교환을 원활하게 한다.12 이를 통해 개발자들은 기존의 3D 자산을 쉽게 통합하고 협업 워크플로우를 구축할 수 있다.


NVIDIA는 "Omniverse 블루프린트"를 통해 이러한 시너지를 구체적으로 구현한다.6 이는 특정 작업을 위해 Omniverse와 Cosmos를 결합한 사전 패키지된 워크플로우이다. 예를 들어, **자율주행차 시뮬레이션 블루프린트**는 Foretellix와 같은 회사가 날씨나 조명 조건을 다양하게 변경하여 테스트 데이터의 다양성을 증폭시키는 데 사용되며, **GR00T 블루프린트**는 합성 조작 모션 생성을 위해 사용된다.25



NVIDIA의 전체 스택은 로봇 공학 개발을 위한 엔드투엔드 워크플로우를 제공한다.

- **Isaac Sim (on Omniverse):** 로봇과 그 환경의 초기 물리적으로 정확한 시뮬레이션을 생성하는 데 사용된다.35
- **Cosmos:** 실제 데이터 수집의 병목 현상을 극복하고 로봇 정책을 훈련시키기 위해 방대하고 다양한 합성 데이터를 생성한다.28
- **Isaac Lab:** Cosmos에서 생성된 데이터를 사용하여 정책을 훈련시키는 강화 학습 및 모방 학습을 위한 모듈식 오픈소스 프레임워크이다.37
- **Project GR00T:** 이는 실제 데이터, Cosmos의 합성 데이터, 그리고 인간 비디오 데이터의 방대한 혼합으로 훈련된 범용 휴머노이드 로봇을 위한 파운데이션 모델이라는 궁극적인 목표를 나타낸다.40


Cosmos는 자율주행차 개발 "플라이휠(flywheel)"에 통합되어 개발 주기를 가속화한다.21 개발자는 실제 주행 데이터를 가져와 DRIVE Sim(Omniverse 기반)에서 재구성한 다음, Cosmos를 사용하여 해당 시나리오의 무한한 변형(예: 맑은 날을 안개 낀 밤으로 변경)을 생성하여 견고한 테스트와 검증을 수행할 수 있다.30



Cosmos, Omniverse, Isaac, NeMo를 포함한 전체 소프트웨어 스택은 NVIDIA 하드웨어에서 가장 효율적으로 실행되도록 최적화되어 있다.3 이는 고객이 WFM 훈련을 위한 DGX Cloud에서부터 차량 및 로봇의 엣지 배포를 위한 NVIDIA DRIVE AGX 및 Jetson 플랫폼에 이르기까지 NVIDIA의 전체 솔루션을 채택하도록 강력한 인센티브를 제공한다.17


Cosmos의 전략적 위치는 CUDA가 장기간에 걸쳐 구축한 이점과 연결된다.6 CUDA가 병렬 컴퓨팅의 표준이 되어 개발자들을 NVIDIA GPU 생태계에 묶어둔 것처럼, Cosmos/Omniverse/Isaac 스택은 물리적 AI를 위한 필수적인 소프트웨어 계층이 되어 관련 하드웨어 시장에서의 지속적인 지배력을 보장하도록 설계되었다.14

NVIDIA는 단순히 합성 데이터를 생성하는 것을 넘어, 실제 세계로부터의 피드백을 통해 지속적으로 개선되는 "폐쇄 루프, sim-to-real-to-sim" 개발 플랫폼을 구축하고 있다. 이는 스스로 강화되는 선순환 구조를 만든다. 개발자는 Isaac Sim이나 DRIVE Sim에서 시뮬레이션을 시작하고 Cosmos를 통해 초기 훈련 데이터를 생성한다.31 이후 Isaac Lab에서 정책을 훈련시켜 실제 로봇이나 차량에 배포한다.35 실제 세계에서 작동하는 로봇/차량은 특히 엣지 케이스나 실패 상황에서 새로운 센서 데이터를 수집한다. 이 새로운 실제 데이터는 다시 플랫폼으로 피드백된다. 이 데이터는 Cosmos WFM을 미세 조정하여 더 현실적으로 만들거나, Omniverse 내에서 실패 시나리오를 재구성하는 데 사용될 수 있으며, Cosmos는 이를 기반으로 수천 가지 변형을 생성하여 재훈련에 활용한다.30 이러한 '시뮬레이션 -->> 생성 -->> 배포 -->> 수집 -->> 개선'의 루프는 플랫폼이 시간이 지남에 따라 더 나아지고, 더 많이 사용될수록 그 가치가 더욱 커지는 강력한 데이터 네트워크 효과를 창출한다.

이러한 생태계의 정점에는 "킬러 앱"으로서의 Project GR00T가 있다. 범용 휴머노이드 로봇을 만들겠다는 야심찬 목표는 너무나 복잡하여 NVIDIA 스택의 모든 구성 요소를 필요로 한다. 이는 강력한 기술 시연인 동시에 전체 플랫폼의 통합을 강제하는 역할을 한다. 휴머노이드 로봇은 그 복잡성, 접촉이 많은 조작의 필요성, 비정형적인 인간 환경에서의 작동 때문에 궁극적인 물리적 AI 과제로 여겨진다.41 이러한 로봇을 훈련시키기 위해서는 전례 없는 양의 다양한 데이터가 필요하며, 이는 오직 Cosmos와 같은 합성 데이터 생성 기술을 통해서만 필요한 규모로 생성될 수 있다.40 GR00T N1 모델 자체도 정교한 VLA(Vision-Language-Action) 모델이며, 이중 시스템 아키텍처를 가지고 있어 Isaac Lab과 같은 정교한 훈련 프레임워크를 요구한다.40 또한, 그 행동을 검증하기 위해서는 Omniverse 기반의 Isaac Sim과 같은 고충실도의 물리적으로 정확한 시뮬레이터가 필수적이다.37 따라서 휴머노이드 로봇 경쟁에 진지하게 임하는 모든 회사는 NVIDIA의 전체 블루프린트를 채택할 강력한 동기를 갖게 되며, 이는 GR00T를 전체 플랫폼을 위한 '트로이 목마'로 만든다.


이 파트에서는 Cosmos를 경쟁 환경 내에 위치시키고, 경쟁자들과의 전략적 접근법을 비교하며, 주요 산업 플레이어들이 이를 어떻게 채택하고 있는지 검토한다.



자율 시스템 훈련에 대한 두 가지 근본적으로 다른 철학을 대조하는 것은 Cosmos의 전략을 이해하는 데 핵심적이다.

- **Tesla의 접근법:** Tesla는 자사 차량 플릿에서 수집한 방대한 규모의 실제 주행 데이터에 의존한다. 이는 지속적으로 다양하고 실제적인 시나리오를 제공하며, 실제 데이터가 궁극적인 진실이라는 주장에 기반한다.46
- **NVIDIA의 접근법:** NVIDIA는 실제 데이터가 가치 있지만, 드물지만 치명적인 엣지 케이스를 포괄하기에는 불충분하다고 주장한다. Cosmos는 이러한 시나리오를 의도적으로, 그리고 대규모로 합성적으로 생성할 수 있게 해준다.6

분석 결과, 미래는 두 접근법의 하이브리드 형태가 될 가능성이 높다. 그러나 Cosmos는 Tesla가 아닌 다른 플레이어들에게 물리적으로 수집할 수 없는 규모의 데이터를 생성함으로써 경쟁할 수 있는 실행 가능한 경로를 제공한다.46

| 특성                 | NVIDIA Cosmos 접근법                                         | Tesla FSD 접근법                                             |      |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| **데이터 소스**      | 합성 데이터 우선 (Omniverse/Cosmos), 실제 데이터로 보강 및 미세 조정 | 실제 데이터 우선 (차량 플릿), 시뮬레이션으로 보강            |      |
| **데이터 규모**      | 잠재적으로 무한함, 계산 자원에 의해 제한됨. 엣지 케이스의 대규모 생성 가능. | 차량 플릿의 주행 거리에 비례하여 지속적으로 증가.            |      |
| **비용 모델**        | 높은 초기 훈련 비용(NVIDIA 부담), 개발자는 플랫폼 사용 및 미세 조정 비용 지불. | 차량 판매에 데이터 수집 비용 분산, 대규모 데이터 센터 운영 비용. |      |
| **엣지 케이스 처리** | 특정 시나리오(악천후, 위험 상황)를 의도적으로 대량 생성하여 훈련 가능. | 실제 주행 중 우연히 발생하는 엣지 케이스 수집에 의존.        |      |
| **Sim-to-Real 과제** | 시뮬레이션과 현실 간의 차이(reality gap)를 극복해야 하는 근본적인 과제 존재. | 실제 데이터 기반이므로 Sim-to-Real 문제가 적지만, 데이터의 편향성 문제 존재. |      |
| **생태계 의존성**    | NVIDIA 하드웨어 및 소프트웨어 스택(Omniverse, Isaac)에 대한 높은 의존성. | 수직적으로 통합된 자체 하드웨어(HW4) 및 소프트웨어 스택에 의존. |      |
| **일반화 경로**      | 다양한 합성 시나리오를 통한 일반화 추구.                     | 방대한 실제 주행 데이터를 통한 경험적 일반화 추구.           |      |

표 1: 자율주행차 데이터 전략 비교 분석 (NVIDIA 대 Tesla). 이 표는 28의 정보를 종합하여 구성됨.


시뮬레이션 우선 접근법의 아킬레스건은 바로 'sim-to-real' 격차, 즉 시뮬레이션에서 학습한 정책이 실제 세계에서 제대로 작동하지 않는 문제이다. NVIDIA는 이 문제를 해결하기 위해 다각적인 전략을 구사한다.

- **고충실도 물리:** Omniverse와 Newton과 같은 물리 엔진을 사용하여 물리적으로 정확한 시뮬레이션을 구현한다.35
- **사실적 렌더링:** Cosmos Transfer를 사용하여 시뮬레이션된 센서 데이터(카메라, LiDAR)를 실제와 구별하기 어렵게 만든다.6
- **영역 무작위화(Domain Randomization):** Isaac Sim/Lab에서 사용되는 핵심 기술로, 훈련 중에 물리적 매개변수(마찰, 질량, 조명 등)를 체계적으로 무작위화한다. 이는 AI 정책이 완벽한 단일 시뮬레이션에 과적합되지 않고, 노이즈가 많고 가변적인 실제 세계에 일반화될 수 있는 견고한 행동을 학습하도록 강제한다.35



Cosmos/Omniverse 스택은 여러 경쟁 플랫폼과 비교된다.

- **CARLA & AirSim:** 이들은 학술 연구에서 널리 사용되는 강력한 오픈소스 시뮬레이터이다. 그러나 엔드투엔드 통합, Cosmos와 같은 생성형 AI 기능, 그리고 NVIDIA 플랫폼이 제공하는 엔터프라이즈급 지원이 부족하다는 점에서 차이가 있다.51
- **Microsoft Azure Digital Twins:** 이는 비즈니스 모델 자체가 다르다. Azure는 지식 그래프를 생성하고 IoT 데이터를 연결하여 기업 모니터링에 중점을 두는 반면, Omniverse는 3D 네이티브의 물리 기반 시뮬레이션 및 협업 플랫폼이다. 두 플랫폼은 직접적인 경쟁자라기보다는, Omniverse Cloud가 Azure에서 호스팅되는 등 점차 파트너 관계로 발전하고 있다.13


Cosmos 모델을 허용적인 개방형 라이선스로 제공하는 것은 전략적으로 중요한 의미를 갖는다.18 이는 커뮤니티 혁신을 촉진하고 채택을 가속화하며, 독점 모델에 의존하는 경쟁자들에게 압력을 가한다.62 이 전략은 기본 모델 자체를 상품화(commoditize)하고, 가치를 주변 플랫폼과 하드웨어로 이동시킨다.


Cosmos의 시장 견인력은 초기 채택 기업들의 사례를 통해 구체적으로 확인할 수 있다.

- **휴머노이드 로봇 (Agility Robotics, Figure AI):** Agility와 같은 회사는 Cosmos와 Mega Omniverse 블루프린트를 사용하여 "실제 세계에서 실현 가능하게 수집할 수 있는 것 이상의" 사실적인 훈련 데이터를 생성하고 있다. 또한 Schaeffler와 같은 고객사의 시설을 디지털 트윈으로 구축하여 사전 배포 테스트를 수행하고 있다.6
- **자율주행차 (Uber, Foretellix, Wayve, Waabi):**
  - **Uber:** Cosmos와 DGX Cloud를 사용하여 방대한 실제 주행 데이터를 활용, 더 강력하고 확장 가능한 AV 모델을 구축하고 있다. 이는 실제 데이터와 합성 데이터를 결합하는 하이브리드 접근법의 대표적인 예이다.44
  - **Foretellix:** 자사의 Foretify 플랫폼에 Cosmos Transfer를 통합하여 날씨 및 조명 변화를 포함한 행동 시나리오를 증강시키고, 검증 데이터의 다양성을 높이고 있다.25
  - **Wayve & Waabi:** 자체적인 신경망 시뮬레이션 접근법(GAIA, Waabi World)을 보유한 이 회사들은 데이터 큐레이션이나 엣지 케이스 검색과 같은 특정 파이프라인 작업에 Cosmos를 평가하고 있다. 이는 Cosmos가 고급 기술을 보유한 플레이어에게도 모듈식 유틸리티로서 가치가 있음을 보여준다.45

이러한 시장 동향은 Cosmos가 전체 자동차 및 로봇 산업을 위한 "데이터 격차 해소자"로 자리매김하고 있음을 보여준다. 이는 Tesla의 주요 데이터 우위를 효과적으로 중화시키는 전략이다. Tesla의 핵심 경쟁력은 매일 수백만 마일의 실제 주행 데이터를 수집하는 방대한 차량 플릿에 있다.46 다른 OEM 및 AV 스타트업들은 이 데이터 수집 엔진을 복제할 수 없으며, 이는 상당한 진입 장벽으로 작용한다. Cosmos는 이러한 기업들에게 실제 세계에서 포착하기 어려운 희귀하지만 중요한 엣지 케이스를 구체적으로 목표로 하여, 비교 가능하거나 훨씬 더 큰 규모의 합성 데이터를 생성할 수 있는 방법을 제공한다.47 이 능력을 제공함으로써 NVIDIA는 나머지 산업이 데이터 측면에서 Tesla를 따라잡을 수 있는 잠재력을 부여하고, NVIDIA의 플랫폼을 모든 비-Tesla 경쟁자들에게 필수적인 조력자로 만든다.

또한, NVIDIA는 "sim-to-real 격차"를 해결 불가능한 문제에서 공학적 과제로 재구성하고 있다. 전통적으로 sim-to-real 격차는 로봇 공학의 근본적이고 거의 철학적인 문제로 여겨져 왔다.83 NVIDIA의 접근 방식은 각 계층에서 체계적으로 이 문제를 해결하는 것이다. 

**물리 계층**에서는 Omniverse의 고충실도 시뮬레이션을 사용하고 35, 

**인식 계층**에서는 Cosmos Transfer를 통해 사실적인 렌더링을 구현한다.27

**정책 계층**에서는 Isaac Lab의 공격적인 영역 무작위화를 통해 견고성을 확보하며 35, 

**인지 계층**에서는 Cosmos Reason을 사용하여 비현실적인 시나리오를 필터링한다.28 이는 격차를 다루기 힘든 장애물에서 복잡하지만 관리 가능한 공학 워크플로우로 전환시킨다. 기어 조립과 같은 작업에서 제로샷(zero-shot) sim-to-real 전송의 성공은 이 접근법의 실행 가능성을 입증한다.35


이 파트에서는 Cosmos 및 더 넓은 물리적 AI 패러다임과 관련된 과제와 위험을 비판적으로 평가하고, 기술을 넘어 사회적 영향까지 분석한다.



Cosmos와 같은 프론티어 모델의 훈련 및 배포에는 막대한 계산 비용이 수반된다.19 훈련 비용이 연간 2.4배씩 증가하고 있으며 2027년까지 10억 달러를 초과할 수 있다는 추정은, NVIDIA와 같은 소수의 하이퍼스케일 기업만이 이러한 파운데이션 모델을 처음부터 구축할 수 있음을 시사한다.20 이는 기술 접근성의 불균형을 심화시킬 수 있는 중요한 재정적 장벽이다.


LLM의 환각(hallucination)은 부정확한 텍스트를 생성하는 데 그치지만, WFM의 "환각"은 물리적으로 불가능하거나 안전하지 않은 훈련 시나리오를 생성할 수 있다.15 이는 치명적인 결과를 초래할 수 있으므로, NVIDIA의 완화 전략은 매우 중요하다. 여기에는 물리 법칙을 인지하는 아키텍처, 필터링을 위한 Cosmos Reason 모델, 그리고 생성된 데이터의 안전성과 일관성을 보장하기 위한 내장된 가드레일(Guardrails)이 포함된다.15



Cosmos 훈련에 사용된 초기 실제 데이터에 존재하는 편향이 생성된 합성 데이터에서 대규모로 복제되고 증폭될 수 있다는 것은 심각한 위험이다.89 예를 들어, 훈련 데이터가 특정 인구 통계나 환경을 제대로 대표하지 않으면, 결과적으로 생성된 자율주행차 및 로봇 모델은 해당 맥락에서 성능이 저하되고 불공정한 결과를 낳을 것이다.93 이는 기존의 사회적 불평등을 기술적으로 고착화시킬 위험을 내포한다.


강력한 세계 생성 기술은 양날의 검이다. 자율주행차 훈련 데이터를 만드는 동일한 모델이 허위 정보 유포나 정치적 조작을 위해 매우 사실적인 딥페이크를 만드는 데 악용될 수 있다.94 이에 대한 NVIDIA의 대응책으로는 AI 생성 콘텐츠에 **보이지 않는 워터마크**를 삽입하고, 유해한 입력을 차단하는 사전 가드와 안전하지 않은 출력을 막는 사후 가드로 구성된 **가드레일**을 구현하는 것이 있다. 이러한 조치의 효과와 한계에 대한 지속적인 분석이 필요하다.15



기존의 법적 프레임워크, 주로 **제조물 책임**(결함에 대해 제조업체에 책임을 묻는)과 **과실**(합리적인 주의 의무를 다하지 않은 경우)은 자율 시스템에 의해 도전을 받고 있다.98 Uber 자율주행차 사망 사고와 같은 실제 사례는 책임 소재 규명의 복잡성을 잘 보여준다.101

| 법적 책임 모델                                      | 정의                                                         | AI/AV에 대한 적용                                            | 주요 과제                                                    | 제안된 해결책/진화                                           |      |
| --------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| **제조물 책임 (Product Liability)**                 | 결함이 있는 제품으로 인해 발생한 손해에 대해 제조업체, 유통업체, 판매업체에 책임을 묻는 법리. | AI 소프트웨어를 '제품'으로 간주하여 알고리즘 결함, 데이터 결함, 예측 실패 등을 제품 결함으로 취급. | **결함 증명의 어려움:** '블랙박스' 특성으로 인해 소프트웨어의 특정 결함을 입증하기 어려움. **인과관계 증명:** 결함과 손해 사이의 직접적인 인과관계를 입증하기 복잡함. | **입증 책임 전환:** 특정 상황에서 결함의 존재를 추정. **소프트웨어 포함:** EU의 새로운 제조물 책임 지침은 소프트웨어 및 업데이트를 명시적으로 포함. |      |
| **과실 책임 (Negligence)**                          | '합리적인 사람'이 기울였을 주의 의무를 위반하여 타인에게 손해를 입힌 경우 책임을 묻는 법리. | 개발자가 예측 가능한 위험을 방지하기 위한 합리적인 조치(예: 철저한 테스트, 안전 조치 구현)를 소홀히 한 경우 적용. | **예측 가능성의 한계:** AI의 창발적 행동은 예측하기 어려워 '합리적인 주의'의 범위를 정의하기 모호함. **표준 부재:** AI 개발에 대한 명확한 산업 표준이 부족. | **교육 이론:** 개발자, 조달자, 사용자 간의 테스트 및 교육 의무를 강조하여 예측 가능성을 높임. **시스템 기반 책임:** 특정 오류가 아닌 전체 개발 및 검증 프로세스의 적절성을 평가. |      |
| **무과실 책임 (Strict Liability)**                  | 행위자의 과실 여부와 관계없이 발생한 손해에 대해 책임을 부과하는 법리. | 자율 시스템으로 인해 발생한 모든 손해에 대해 제조업체나 운영자에게 책임을 지움. | **혁신 저해 가능성:** 과도한 책임 부담이 기술 개발 및 상용화를 위축시킬 수 있음. **책임 분배의 어려움:** 여러 주체가 관여된 경우 책임 분배가 불공정할 수 있음. | **국가 보상 기금:** 피해자에게 우선 보상하고, 이후 국가가 책임 당사자에게 구상권을 행사하는 방식. **국가에 대한 손해배상:** 피해자 보상 대신 국가에 벌금을 납부하여 제조업체의 안전 투자 유인을 높이는 모델. |      |
| **AI 특별 규제 (Proposed AI-Specific Regulations)** | AI의 특수성을 고려하여 새롭게 제정되는 법률 및 규제. (예: EU AI Act) | 고위험 AI 시스템에 대해 투명성, 데이터 거버넌스, 인간 감독 등 특정 의무를 부과하고, 위반 시 책임을 물음. | **기술 발전에 대한 느린 대응:** 빠르게 발전하는 기술에 법규가 뒤처질 수 있음. **국가별 규제 파편화:** 국제적으로 통일된 기준이 없어 규제 차익 발생 가능. | **위험 기반 접근법:** AI 시스템의 위험 수준에 따라 규제 강도를 차등 적용. **지속적인 모니터링 및 감사:** 규제 기관이 AI 시스템의 실제 운영을 지속적으로 감독. |      |

표 2: 자율 시스템에 대한 책임 모델 분석. 이 표는 98의 정보를 종합하여 구성됨.


딥러닝 모델의 불투명성은 실패로 이어진 코드나 데이터의 정확한 결함을 찾아내는 것을 극도로 어렵게 만든다.99 이 "블랙박스" 문제는 전통적인 과실 기반 책임론을 약화시킨다.


법학자들과 정책 입안자들은 다음과 같은 해결책을 논의하고 있다.

- **무과실 책임:** 제조업체에 과실 여부와 관계없이 책임을 지워 피해자 구제 절차를 단순화하지만, 혁신을 저해할 수 있다.98
- **의무 보험:** 자동차 보험과 유사하게, 보상 기금이 마련되도록 보장한다.99
- **규제 감독 및 표준:** EU AI Act, BSI/IEEE 표준 등 AI 개발자 및 운영자의 주의 의무를 정의하는 새로운 표준을 개발한다.99

합성 데이터 생성으로의 전환은 위험과 책임을 플랫폼 제공자(NVIDIA)에게 중앙 집중화시키는 효과를 낳는다. 실제 데이터 패러다임에서는 책임이 운전자, 소프트웨어 개발자, 센서 제조업체 등 여러 주체에 분산된다. 그러나 훈련 및 검증 데이터의 대부분이 단일 플랫폼(Cosmos)에서 생성될 때, 해당 플랫폼 모델의 체계적인 편향, 안전 결함 또는 물리적 부정확성은 그것으로 훈련된 모든 AI 시스템에 전파될 수 있다. 만약 여러 제조업체의 자율주행차 전체가 공통된 실패 모드를 보인다면, 조사는 그들이 모두 훈련받은 기초 WFM의 결함으로 거슬러 올라갈 수 있다. 이는 NVIDIA에게 자사 WFM의 충실도, 공정성, 안전성을 보장해야 하는 막대한 윤리적, 잠재적 법적 부담을 안겨준다. 따라서 Cosmos Reason이나 가드레일과 같은 도구는 단순한 기능이 아니라 NVIDIA 자체의 비즈니스를 위한 중요한 위험 완화 장치가 된다.

또한, "블랙박스" 문제는 법적 패러다임을 "과실 기반"에서 "시스템 기반" 책임 및 규제로의 전환을 가속화할 것이다. 법원과 규제 기관은 수십억 개의 파라미터를 가진 모델에서 단 하나의 잘못된 코드 라인을 찾는 것이 무의미하다는 것을 인식하게 될 것이다.100 따라서 특정 오류(과실)에 초점을 맞추는 대신, 개발 프로세스와 시스템 전체에 초점을 맞추게 될 것이다. 개발자가 최신 검증 도구를 사용했는가? 훈련 데이터가 편향에 대해 감사되었는가? Cosmos/Omniverse와 같은 견고한 시뮬레이션 및 테스트 플랫폼이 사용되었는가? 이는 포괄적이고 잘 문서화된 플랫폼인 Cosmos를 사용하는 것이 법적 방어의 한 형태가 될 수 있음을 의미한다. 기업은 업계 표준 도구를 사용하여 높은 수준의 주의 의무를 다했다고 주장할 수 있으며, 이는 플랫폼 제공자(NVIDIA)에게 책임을 돌리거나 공동 책임 프레임워크를 구축하는 근거가 될 수 있다.


이 마지막 파트에서는 보고서 전체를 종합하여 Cosmos의 영향에 대한 고차원적인 전략적 평가를 제공하고 물리적 AI의 미래에 대한 전망을 제시한다.


이 보고서의 핵심 주장을 요약하자면, Cosmos는 단순한 기술이 아니라 물리적 AI의 근본적인 데이터 병목 현상을 해결하기 위해 설계된 전략적 플랫폼이다. 이를 통해 NVIDIA는 차세대 산업 자동화에 필수적인 자사의 전체 하드웨어 및 소프트웨어 스택을 확고히 하고자 한다. "데이터 공장" 플라이휠에서의 역할과 더 넓은 NVIDIA 생태계로의 통합은 Cosmos가 단순한 도구를 넘어, 물리적 세계와 상호작용하는 모든 AI 시스템의 기초가 될 잠재력을 가지고 있음을 보여준다.



Cosmos 플랫폼은 앞으로 Rubin 아키텍처와 같은 차세대 하드웨어와의 더욱 긴밀한 통합, 현재의 Cosmos Reason을 넘어서는 더 정교한 추론 모델의 등장, 그리고 새로운 산업을 위한 Omniverse 블루프린트의 확장을 통해 계속 발전할 것으로 예상된다.108 이는 플랫폼의 성능과 적용 범위를 지속적으로 확장시켜 나갈 것이다.


이 분석은 기술(Cosmos), 로봇 및 AI 시장 예측(2030년까지 수십억 달러 규모로 성장 예상 111), 그리고 노동력에 미칠 잠재적 영향(123]) 사이의 점들을 연결한다. "ChatGPT 모멘트"가 다가오고 있지만, 물리적 AI의 광범위한 배포는 향후 10년에 걸친 과정이 될 가능성이 높다는 신중한 전망을 제시한다.10


이 보고서는 주요 이해관계자들을 위한 실행 가능한 권고안으로 마무리된다.

- **개발자 및 스타트업:** 개방형 모델을 활용하되, 생태계 종속의 가능성을 염두에 두고 전략을 수립해야 한다.
- **기존 산업 및 자동차 기업:** 디지털화를 가속화하고 AI 네이티브 기업과 경쟁하기 위해 Cosmos를 기존 워크플로우에 통합하는 방안을 모색해야 한다.
- **투자자:** 물리적 AI 분야의 기업을 평가할 때, 그들의 데이터 전략(실제, 합성 또는 하이브리드)과 지배적인 NVIDIA 생태계와의 관계를 핵심적으로 분석해야 한다.
- **정책 입안자:** 이 새로운 기술 물결이 제기하는 책임, 안전, 그리고 노동력 전환 문제를 해결하기 위해 신속하고 유연한 규제 및 법적 프레임워크를 개발하는 것이 시급하다.

NVIDIA Cosmos는 물리적 AI라는 새로운 시대를 여는 데 있어 가장 중요한 기술적, 전략적 이니셔티브 중 하나이다. 그 성공 여부는 기술적 완성도뿐만 아니라, 복잡한 윤리적, 법적, 사회적 과제들을 어떻게 해결해 나가는지에 달려 있을 것이다. Cosmos는 단순한 시뮬레이션 도구를 넘어, 현실 세계를 디지털로 복제하고, 예측하며, 궁극적으로는 자동화하는 거대한 비전의 핵심에 서 있다.


1. What Is Nvidia Cosmos? | Built In, accessed July 18, 2025, https://builtin.com/articles/nvidia-cosmos
2. Cosmos World Foundation Model Platform for Physical AI - arXiv, accessed July 18, 2025, https://arxiv.org/html/2501.03575v1
3. Advancing Physical AI with NVIDIA Cosmos World Foundation Model Platform, accessed July 18, 2025, https://developer.nvidia.com/blog/advancing-physical-ai-with-nvidia-cosmos-world-foundation-model-platform/
4. NVIDIA heralds physical AI era with Cosmos platform launch - R&D World, accessed July 18, 2025, https://www.rdworldonline.com/nvidia-heralds-physical-ai-era-with-cosmos-platform-launch/
5. NVIDIA Cosmos: Empowering Physical AI with Simulations - Unite.AI, accessed July 18, 2025, https://www.unite.ai/nvidia-cosmos-empowering-physical-ai-with-simulations/
6. NVIDIA Launches Cosmos AI Models for Physical World Generation | NVDA Stock News, accessed July 18, 2025, https://www.stocktitan.net/news/NVDA/nvidia-announces-major-release-of-cosmos-world-foundation-models-and-45pq7l6hkmio.html
7. Nvidia unveils Cosmos platform for robotics development, accessed July 18, 2025, https://roboticsandautomationnews.com/2025/01/16/nvidia-unveils-cosmos-platform-for-robotics-development/88639/
8. Nvidia's Omniverse + Cosmos to train physical agents is the craziest thing I have ever seen : r/singularity - Reddit, accessed July 18, 2025, https://www.reddit.com/r/singularity/comments/1hvizea/nvidias_omniverse_cosmos_to_train_physical_agents/
9. Nvidia launches Cosmos models, aims to expand physical AI, industrial reach, accessed July 18, 2025, https://www.constellationr.com/blog-news/insights/nvidia-launches-cosmos-models-aims-expand-physical-ai-industrial-reach
10. NVIDIA's Cosmos platform for physical robot AI: ChatGPT breakthrough for general robotics is imminent - Xpert.Digital, accessed July 18, 2025, https://xpert.digital/en/physical-ai-of-robots/
11. NVIDIA Cosmos, accessed July 18, 2025, https://resources.nvidia.com/en-us-dgx-cloud/cosmos/
12. NVIDIA Expands Omniverse With Generative Physical AI, accessed July 18, 2025, https://nvidianews.nvidia.com/news/nvidia-expands-omniverse-with-generative-physical-ai
13. NVIDIA Expands Omniverse Cloud to Power Industrial Digitalization, accessed July 18, 2025, https://nvidianews.nvidia.com/news/nvidia-expands-omniverse-cloud-to-power-industrial-digitalization
14. Why's Nvidia such a beast? It's that CUDA thing. | SemiWiki, accessed July 18, 2025, [https://semiwiki.com/forum/threads/why%E2%80%99s-nvidia-such-a-beast-it%E2%80%99s-that-cuda-thing.21393/](https://semiwiki.com/forum/threads/why’s-nvidia-such-a-beast-it’s-that-cuda-thing.21393/)
15. NVIDIA Cosmos for Developers, accessed July 18, 2025, https://developer.nvidia.com/cosmos
16. NVIDIA Cosmos, accessed July 18, 2025, https://docs.nvidia.com/cosmos/index.html
17. Cosmos - NVIDIA Jetson AI Lab, accessed July 18, 2025, https://www.jetson-ai-lab.com/cosmos.html
18. NVIDIA Launches Cosmos World Foundation Model Platform to Accelerate Physical AI Development - Nvidia Investor Relations, accessed July 18, 2025, https://investor.nvidia.com/news/press-release-details/2025/NVIDIA-Launches-Cosmos-World-Foundation-Model-Platform-to-Accelerate-Physical-AI-Development/default.aspx
19. What is the cost of training large language models? - CUDO Compute, accessed July 18, 2025, https://www.cudocompute.com/blog/what-is-the-cost-of-training-large-language-models
20. The rising costs of training frontier AI models - arXiv, accessed July 18, 2025, https://arxiv.org/html/2405.21015v1
21. NVIDIA Cosmos - GitHub, accessed July 18, 2025, https://github.com/nvidia-cosmos
22. developer.nvidia.com, accessed July 18, 2025, [https://developer.nvidia.com/cosmos#:~:text=NVIDIA%20Cosmos%E2%84%A2%20is%20a,(AVs)%20and%20robotics%20developers.](https://developer.nvidia.com/cosmos#:~:text=NVIDIA Cosmos™ is a,(AVs) and robotics developers.)
23. Announcing NVIDIA Cosmos World Foundation Models - Hugging Face, accessed July 18, 2025, https://huggingface.co/blog/mingyuliutw/nvidia-cosmos
24. A Deep Dive into NVIDIA Cosmos and Its Capabilities - ADaSci, accessed July 18, 2025, https://adasci.org/a-deep-dive-into-nvidia-cosmos-and-its-capabilities/
25. NVIDIA Announces Major Release of Cosmos World Foundation Models and Physical AI Data Tools, accessed July 18, 2025, https://nvidianews.nvidia.com/news/nvidia-announces-major-release-of-cosmos-world-foundation-models-and-physical-ai-data-tools
26. nvidia-cosmos/cosmos-transfer1: Cosmos-Transfer1 is a ... - GitHub, accessed July 18, 2025, https://github.com/nvidia-cosmos/cosmos-transfer1
27. Scale Synthetic Data and Physical AI Reasoning with NVIDIA Cosmos World Foundation Models, accessed July 18, 2025, https://developer.nvidia.com/blog/scale-synthetic-data-and-physical-ai-reasoning-with-nvidia-cosmos-world-foundation-models/
28. Curating Synthetic Datasets to Train Physical AI Models with NVIDIA Cosmos Reason, accessed July 18, 2025, https://developer.nvidia.com/blog/curating-synthetic-datasets-to-train-physical-ai-models-with-nvidia-cosmos-reason/
29. R²D²: Training Generalist Robots with NVIDIA Research Workflows and World Foundation Models, accessed July 18, 2025, https://developer.nvidia.com/blog/r2d2-training-generalist-robots-with-nvidia-research-workflows-and-world-foundation-models/
30. NVIDIA Releases New AI Models and Developer Tools to Advance Autonomous Vehicle Ecosystem, accessed July 18, 2025, https://blogs.nvidia.com/blog/autonomous-vehicle-ecosystem-ai-models-developer-tools/
31. NVIDIA Isaac Sim, Omniverse, and Cosmos – The Robotics & AI Simulation Ecosystem Explained - RidgeRun.ai, accessed July 18, 2025, https://www.ridgerun.ai/post/nvidia-isaac-sim-omniverse-and-cosmos-the-robotics-ai-simulation-ecosystem-explained
32. NVIDIA Cosmos: A World Foundation Model Platform for Physical AI - YouTube, accessed July 18, 2025, https://www.youtube.com/watch?v=9Uch931cDx8
33. NVIDIA are Simulating the Physical World for AI - YouTube, accessed July 18, 2025, https://www.youtube.com/watch?v=QxXEuc4K1Rk
34. NVIDIA Omniverse Digital Twins Help Taiwan Manufacturers Drive Golden Age of Industrial AI, accessed July 18, 2025, https://blogs.nvidia.com/blog/omniverse-digital-twins-taiwan-manufacturers-physical-ai/
35. Bridging the Sim-to-Real Gap for Industrial Robotic Assembly Applications Using NVIDIA Isaac Lab, accessed July 18, 2025, https://developer.nvidia.com/blog/bridging-the-sim-to-real-gap-for-industrial-robotic-assembly-applications-using-nvidia-isaac-lab/
36. (PDF) Sim-to-Real Transfer for Mobile Robots with Reinforcement Learning: from NVIDIA Isaac Sim to Gazebo and Real ROS 2 Robots - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/387767755_Sim-to-Real_Transfer_for_Mobile_Robots_with_Reinforcement_Learning_from_NVIDIA_Isaac_Sim_to_Gazebo_and_Real_ROS_2_Robots
37. Agility Robotics Expands Relationship with NVIDIA, accessed July 18, 2025, https://www.agilityrobotics.com/content/agility-robotics-expands-relationship-with-nvidia
38. Unified framework for robot learning built on NVIDIA Isaac Sim - GitHub, accessed July 18, 2025, https://github.com/isaac-sim/IsaacLab
39. Real2Sim Teleoperation With ROS 2 and Isaac Sim – Preparing for GR00T Mimic - YouTube, accessed July 18, 2025, https://www.youtube.com/watch?v=z7KdHGkUTNE
40. GR00T N1: An Open Foundation Model for ... - Cloudfront.net, accessed July 18, 2025, https://d1qx31qr3h6wln.cloudfront.net/publications/GR00T_1_Whitepaper.pdf
41. NVIDIA Announces Isaac GR00T N1 - the World's First Open Humanoid Robot Foundation Model - and Simulation Frameworks to Speed Robot Development, accessed July 18, 2025, https://nvidianews.nvidia.com/news/nvidia-isaac-gr00t-n1-open-humanoid-robot-foundation-model-simulation-frameworks
42. NVIDIA Cosmos: Transforming Autonomous Mobility with Physical AI, accessed July 18, 2025, https://tecknexus.com/5gnews-all/nvidia-cosmos-transforming-autonomous-mobility-with-physical-ai/
43. NVIDIA's Unrivaled AI Infrastructure Play: Why the Ecosystem Monopoly Ensures Dominance - AInvest, accessed July 18, 2025, https://www.ainvest.com/news/nvidia-unrivaled-ai-infrastructure-play-ecosystem-monopoly-ensures-dominance-2506/
44. Uber Technologies, Inc. - Uber Teams Up with NVIDIA to Accelerate Autonomous Mobility, accessed July 18, 2025, https://investor.uber.com/news-events/news/press-release-details/2025/Uber-Teams-Up-with-NVIDIA-to-Accelerate-Autonomous-Mobility/default.aspx
45. Cosmos World Foundation Models Openly Available to Physical AI Developers, accessed July 18, 2025, https://blogs.nvidia.com/blog/cosmos-world-foundation-models/
46. Nvidia's Cosmos Offers Synthetic Training Data; Following Tesla's Lead - Not a Tesla App, accessed July 18, 2025, https://www.notateslaapp.com/news/2484/nvidias-cosmos-offers-synthetic-training-data-following-teslas-lead
47. Why NVIDIA's New Tech Could Change Everything for Tesla - YouTube, accessed July 18, 2025, https://www.youtube.com/watch?v=E1cCcGtfbnc&pp=0gcJCfwAo7VqN5tD
48. [isaacsim.replicator.domain_randomization] Isaac Sim Replicator Domain Randomization, accessed July 18, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/py/source/extensions/isaacsim.replicator.domain_randomization/docs/index.html
49. OmniIsaacGymEnvs/docs/framework/domain_randomization.md at main - GitHub, accessed July 18, 2025, https://github.com/isaac-sim/OmniIsaacGymEnvs/blob/main/docs/framework/domain_randomization.md
50. UNDERSTANDING DOMAIN RANDOMIZATION FOR SIM-TO-REAL TRANSFER, accessed July 18, 2025, https://collaborate.princeton.edu/en/publications/understanding-domain-randomization-for-sim-to-real-transfer
51. CARLA Simulator, accessed July 18, 2025, https://carla.org/
52. Carla vs Airsim / Issue #112 / carla-simulator/carla - GitHub, accessed July 18, 2025, https://github.com/carla-simulator/carla/issues/112
53. There is also... AirSim from Microsoft - https://github.com/microsoft/airsim Gaz... | Hacker News, accessed July 18, 2025, https://news.ycombinator.com/item?id=15721311
54. A Comparative Analysis of CARLA and AirSim Simulators: Investigating Implementation Challenges in Autonomous Driving - Longdom Publishing SL, accessed July 18, 2025, https://www.longdom.org/open-access-pdfs/a-comparative-analysis-of-carla-and-airsim-simulators-investigating-implementation-challenges-in-autonomous-driving.pdf
55. Introduction to the CARLA simulator: training a neural network to control a car (Part 1), accessed July 18, 2025, https://medium.com/asap-report/introduction-to-the-carla-simulator-training-a-neural-network-to-control-a-car-part-1-e1c2c9a056a5
56. NVIDIA and Microsoft to Bring the Industrial Metaverse and AI to Hundreds of Millions of Enterprise Users via Azure Cloud, accessed July 18, 2025, https://nvidianews.nvidia.com/news/nvidia-and-microsoft-to-bring-the-industrial-metaverse-and-ai-to-hundreds-of-millions-of-enterprise-users-via-azure-cloud
57. Competition in Digital Twin space intensifies - Top Management College in Kolkata, accessed July 18, 2025, https://praxis.ac.in/competition-in-digital-twin-space-intensifies/
58. Azure Digital Twin in Omniverse, Bidirectional - YouTube, accessed July 18, 2025, https://www.youtube.com/watch?v=BSmtBzAfgiQ
59. NVIDIA Omniverse on Azure - Microsoft Download Center, accessed July 18, 2025, [https://download.microsoft.com/download/c/5/b/c5b17efc-71a0-48a1-9cbf-506ac0c9d9ed/20230928_1315_T1_NVIDA's%20Omniverse%20auf%20Azure.pdf](https://download.microsoft.com/download/c/5/b/c5b17efc-71a0-48a1-9cbf-506ac0c9d9ed/20230928_1315_T1_NVIDA's Omniverse auf Azure.pdf)
60. Utilizing NVIDIA Omniverse™ for Digital Twin Projects at the NTT DATA Innovation Center, accessed July 18, 2025, https://www.nttdata.com/global/en/insights/focus/2023/utilizing-nvidia-omniverse-for-digital-twin-projects-at-the-ntt-data-innovation-center
61. A comparative analysis for harnessing digital twin platforms for net-zero manufacturing - White Rose Research Online, accessed July 18, 2025, https://eprints.whiterose.ac.uk/221138/1/A_Comparative_Analysis_for_Harnessing_Digital_Twin_Platforms_for_Net-Zero_Manufacturing.pdf
62. The Open Source Legacy and AI's Licensing Challenge - Linux Foundation, accessed July 18, 2025, https://www.linuxfoundation.org/blog/the-open-source-legacy-and-ais-licensing-challenge
63. Openness in Artificial Intelligence Models: Benefits of Open-Source AI - New America, accessed July 18, 2025, https://www.newamerica.org/oti/reports/openness-in-artificial-intelligence-models/benefits-of-open-source-ai/
64. We Could Use a Model Licensing Framework for Scholarly Content Use in AI Tools, accessed July 18, 2025, https://scholarlykitchen.sspnet.org/2025/02/26/we-could-use-a-model-licensing-framework-for-ai-tools/
65. Open Source at a Crossroads: The Future of Licensing Driven by Monetization - arXiv, accessed July 18, 2025, https://arxiv.org/html/2503.02817v1
66. Agility Robotics Expands Relationship with NVIDIA - Nasdaq, accessed July 18, 2025, https://www.nasdaq.com/press-release/agility-robotics-expands-relationship-nvidia-2025-03-18
67. NVIDIA and Agility Robotics expand partnership, accessed July 18, 2025, https://www.roboticsandautomationmagazine.co.uk/news/nvidia-and-agility-robotics-expand-partnership.html
68. Agility Robotics Deepens Collaboration with NVIDIA - Bots & Drones America, accessed July 18, 2025, https://botsanddrones.co/f/agility-robotics-deepens-collaboration-with-nvidia
69. Uber and Nvidia: A New Era in Autonomous Vehicles - Kavout, accessed July 18, 2025, https://www.kavout.com/market-lens/uber-and-nvidia-a-new-era-in-autonomous-vehicles
70. Uber teams up with NVIDIA to advance autonomous driving technology | Blog - TradeZero, accessed July 18, 2025, https://tradezero.com/blog/uber-teams-up-with-nvidia-to-advance-autonomous-driving-technology
71. Foretellix Expands Data Automation Toolchain for AI-Powered AV Development with Breakthrough Simulation Capabilities Using NVIDIA Omniverse and Cosmos Transfer, accessed July 18, 2025, https://www.foretellix.com/data-automation-toolchain-for-ai-powered-av-development/
72. How Simulation Enables Safer Autonomous Vehicles | Foretellix - YouTube, accessed July 18, 2025, https://www.youtube.com/watch?v=K_Lwzy72jM8&pp=0gcJCfwAo7VqN5tD
73. Foretellix Accelerates AI-Powered Autonomous Vehicles with Data-Driven Development Toolchain and Safety Evaluation, accessed July 18, 2025, https://www.foretellix.com/foretellix-accelerates-ai-powered-autonomous-vehicles/
74. Foretellix Supercharges AI-centric AV Development with NVIDIA Omniverse, accessed July 18, 2025, https://www.foretellix.com/foretellix-nvidia-ai-centric/
75. NVIDIA Launches Cosmos World Foundation Model Platform to Accelerate Physical AI Development, accessed July 18, 2025, https://nvidianews.nvidia.com/news/nvidia-launches-cosmos-world-foundation-model-platform-to-accelerate-physical-ai-development
76. NVIDIA Enhances Three Computer Solution for Autonomous Mobility With Cosmos World Foundation Models, accessed July 18, 2025, https://blogs.nvidia.com/blog/three-computer-cosmos-ces/
77. Driving Forward: Wayve's Collaboration with NVIDIA and the Future of AI in Automotive, accessed July 18, 2025, https://wayve.ai/thinking/wayve-nvidia-collaboration/
78. NVIDIA Cosmos vs PD Replica Sim - Which is right for you? - Parallel Domain, accessed July 18, 2025, https://paralleldomain.com/cosmos-vs-pd-replica-sim/
79. NVIDIA Announces Foundational World Model, Cosmos - Radiance Fields, accessed July 18, 2025, https://radiancefields.com/nvidia-announces-foundational-world-model-cosmos
80. NVIDIA Introduces Cosmos to Accelerate Physical AI Development - Maginative, accessed July 18, 2025, https://www.maginative.com/article/nvidia-introduces-cosmos-to-accelerate-physical-ai-development/
81. NVIDIA Cosmos Platform to Drive Physical AI Advancements in Robotics and Autonomous Vehicles - SentiSight.ai, accessed July 18, 2025, https://www.sentisight.ai/nvidia-cosmos-physical-ai-advancements-robotics/
82. Waabi – The next generation of self-driving technology starts here., accessed July 18, 2025, https://waabi.ai/
83. Sim2Real Gap: Why Machine Learning Hasn't Solved Robotics Yet - Inbolt, accessed July 18, 2025, https://www.inbolt.com/resources/sim2real-gap-why-machine-learning-hasnt-solved-robotics-yet
84. R²D²: Unlocking Robotic Assembly and Contact Rich Manipulation with NVIDIA Research, accessed July 18, 2025, https://developer.nvidia.com/blog/r2d2-unlocking-robotic-assembly-and-contact-rich-manipulation-with-nvidia-research/
85. Transferring Industrial Robot Assembly Tasks from Simulation to Reality - NVIDIA Developer, accessed July 18, 2025, https://developer.nvidia.com/blog/transferring-industrial-robot-assembly-tasks-from-simulation-to-reality/
86. How Much Does It Cost to Train Frontier AI Models? | Epoch AI, accessed July 18, 2025, https://epoch.ai/blog/how-much-does-it-cost-to-train-frontier-ai-models
87. AI Model Training Cost Have Skyrocketed by More than 4300% Since 2020, accessed July 18, 2025, https://www.edge-ai-vision.com/2024/09/ai-model-training-cost-have-skyrocketed-by-more-than-4300-since-2020/
88. What is the Cost of Training LLM Models? - Galileo AI, accessed July 18, 2025, https://galileo.ai/blog/llm-model-training-cost
89. Bias Mitigation via Synthetic Data Generation: A Review - MDPI, accessed July 18, 2025, https://www.mdpi.com/2079-9292/13/19/3909
90. Mitigating the Risk of Bias in Synthetic Data for AI - Datalere, accessed July 18, 2025, https://datalere.com/articles/mitigating-the-risk-of-bias-in-synthetic-data-for-ai
91. Taking Bias out of AI with Synthetic Data - Betterdata.ai, accessed July 18, 2025, https://www.betterdata.ai/blogs/taking-bias-out-of-ai-with-synthetic-data
92. Data bias in LLM and generative AI applications - Mostly AI, accessed July 18, 2025, https://mostly.ai/blog/data-bias-types
93. Algorithm Bias - Synthetic Data Should Be Option of Last Resort When Training AI Systems, accessed July 18, 2025, https://unu.edu/article/algorithm-bias-synthetic-data-should-be-option-last-resort-when-training-ai-systems
94. The Evolution of Disinformation - A Deepfake Future - Canada.ca, accessed July 18, 2025, [https://www.canada.ca/content/dam/csis-scrs/documents/publications/2023/The%20Evolution%20of%20Disinformation%20-%20Deepfake%20Report_EN_DIGITAL.pdf](https://www.canada.ca/content/dam/csis-scrs/documents/publications/2023/The Evolution of Disinformation - Deepfake Report_EN_DIGITAL.pdf)
95. Impacts of Adversarial Use of Generative AI on Homeland Security, accessed July 18, 2025, https://www.dhs.gov/sites/default/files/2025-01/25_0110_st_impacts_of_adversarial_generative_aI_on_homeland_security_0.pdf
96. 4 ways to future-proof against deepfakes in 2024 and beyond | World Economic Forum, accessed July 18, 2025, https://www.weforum.org/stories/2024/02/4-ways-to-future-proof-against-deepfakes-in-2024-and-beyond/
97. The Rise of Artificial Intelligence and Deepfakes - Northwestern Buffett Institute for Global Affairs, accessed July 18, 2025, https://buffett.northwestern.edu/documents/buffett-brief_the-rise-of-ai-and-deepfake-technology.pdf
98. Liability Issues with Autonomous AI Agents - Senna Labs, accessed July 18, 2025, https://sennalabs.com/blog/liability-issues-with-autonomous-ai-agents
99. Can AI Be Held Liable? Exploring Legal Responsibility in Autonomous Systems, accessed July 18, 2025, https://bytes.scl.org/can-ai-be-held-liable-exploring-legal-responsibility-in-autonomous-systems/
100. Challenges in establishing liability for AI-driven products: the limits of recent reforms, accessed July 18, 2025, https://www.dentons.com/en/insights/articles/2025/july/14/challenges-in-establishing-liability-for-ai-driven-products
101. Who is responsible when AI acts autonomously & things go wrong? - Global Legal Insights, accessed July 18, 2025, https://www.globallegalinsights.com/practice-areas/ai-machine-learning-and-big-data-laws-and-regulations/autonomous-ai-who-is-responsible-when-ai-acts-autonomously-and-things-go-wrong/
102. Understanding Liability in Driverless Car Accidents - Van Law Firm, accessed July 18, 2025, https://vanlawfirm.com/blog/understanding-liability-in-driverless-car-accidents/
103. Navigating Liability In Autonomous Robots: Legal And Ethical Challenges In Manufacturing And Military Applications - The Yale Review Of International Studies, accessed July 18, 2025, https://yris.yira.org/column/navigating-liability-in-autonomous-robots-legal-and-ethical-challenges-in-manufacturing-and-military-applications/
104. Accident Liability Determination of Autonomous Driving Systems Based on Artificial Intelligence Technology and Its Impact on Public Mental Health - PubMed Central, accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9451969/
105. An Education Theory of Fault For Autonomous Systems - Scholarly Commons at Boston University School of Law, accessed July 18, 2025, https://scholarship.law.bu.edu/context/faculty_scholarship/article/4054/viewcontent/An_Education_Theory_of_Fault_For_Autonomous_Systems.pdf
106. On the Redesign of Accident Liability for the World of Autonomous Vehicles - National Bureau of Economic Research | NBER, accessed July 18, 2025, https://www.nber.org/system/files/working_papers/w26220/w26220.pdf
107. Reliability and Safety of Autonomous Systems Based on Semantic Modelling for Self-Certification - MDPI, accessed July 18, 2025, https://www.mdpi.com/2218-6581/10/1/10
108. Nvidia CEO Jensen Huang unveils new Rubin AI chips at GTC 2025 | AP News, accessed July 18, 2025, https://apnews.com/article/nvidia-gtc-jensen-huang-ai-457e9260aa2a34c1bbcc07c98b7a0555
109. NVIDIA GTC 2025 live: News and announcements from CEO Jensen Huang's annual keynote - Engadget, accessed July 18, 2025, https://www.engadget.com/ai/nvidia-gtc-2025-live-news-and-announcements-from-ceo-jensen-huangs-annual-keynote-140057910.html
110. Nvidia's New AI Innovations at CES 2025: Explained | Technology Magazine, accessed July 18, 2025, https://technologymagazine.com/articles/nvidias-new-ai-innovations-at-ces-2025-explained
111. Intelligent Robotics Market worth $50.33 billion by 2030 - Exclusive Report by MarketsandMarkets™ - PR Newswire, accessed July 18, 2025, https://www.prnewswire.com/news-releases/intelligent-robotics-market-worth-50-33-billion-by-2030---exclusive-report-by-marketsandmarkets-302458685.html
112. Future of Robotics Size, Share, Industry, 2025 To 2030 - MarketsandMarkets, accessed July 18, 2025, https://www.marketsandmarkets.com/Market-Reports/future-robotics-56928872.html
113. AI in Robotics Market - Size, Share & Industry Analysis - Mordor Intelligence, accessed July 18, 2025, https://www.mordorintelligence.com/industry-reports/artificial-intelligence-in-robotics-market
114. Consumer Robotics Market Size | Industry Report, 2030 - Grand View Research, accessed July 18, 2025, https://www.grandviewresearch.com/industry-analysis/consumer-robotics-market-report
115. Humanoid Robot Market Expected to Reach $5 Trillion by 2050 | Morgan Stanley, accessed July 18, 2025, https://www.morganstanley.com/insights/articles/humanoid-robot-market-5-trillion-by-2050
116. AI In Rehabilitation Robotics Market: Trends, Forecast 2030 - Knowledge Sourcing Intelligence, accessed July 18, 2025, https://www.knowledge-sourcing.com/report/ai-in-rehabilitation-robotics-market
117. Artificial Intelligence [AI] Market Size, Growth & Trends by 2032, accessed July 18, 2025, https://www.fortunebusinessinsights.com/industry-reports/artificial-intelligence-market-100114
118. It's 2025: Is NVIDIA's Cosmos The Missing Piece For Widespread Robot Adoption?, accessed July 18, 2025, https://www.forrester.com/blogs/its-2025-is-nvidias-cosmos-the-missing-piece-for-widespread-robot-adoption/
119. How Tesla Uses Simulated Data to Improve FSD, accessed July 18, 2025, https://www.notateslaapp.com/news/2573/how-tesla-uses-simulated-data-to-improve-fsd
120. NVIDIA's New AI Just UNLEASHED Tesla's Full Power - FSD Will Never Be the Same!, accessed July 18, 2025, https://www.youtube.com/watch?v=vef9k2d9IaM
121. Autonomous AI systems in the face of liability, regulations and costs - PMC, accessed July 18, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10558567/
122. Communicating Liability in Autonomous Vehicles - TAS Hub, accessed July 18, 2025, https://tas.ac.uk/research-projects-2022-23/communicating-liability-in-autonomous-vehicles/
123. The future of work: Robots, AI, and automation - Brookings Institution, accessed July 18, 2025, https://www.brookings.edu/events/the-future-of-work-robots-ai-and-automation/
124. Ways to help workers suffering from AI-related job losses - Brookings Institution, accessed July 18, 2025, https://www.brookings.edu/articles/ways-to-help-workers-suffering-from-ai-related-job-losses/
125. The effects of AI on firms and workers - Brookings Institution, accessed July 18, 2025, https://www.brookings.edu/articles/the-effects-of-ai-on-firms-and-workers/
126. Generative AI, the American worker, and the future of work - Brookings Institution, accessed July 18, 2025, https://www.brookings.edu/articles/generative-ai-the-american-worker-and-the-future-of-work/
127. Future of Work - Brookings Institution, accessed July 18, 2025, https://www.brookings.edu/topics/future-of-work/
128. Robots make your work less meaningful - Brookings Institution, accessed July 18, 2025, https://www.brookings.edu/articles/robots-make-your-work-less-meaningful/
# Google Gemma 3n


Google의 Gemma 3n 모델 출시는 단순한 제품군 업데이트를 넘어, 엣지 컴퓨팅의 미래를 정의하려는 Google의 전략적 아키텍처 도박으로 해석되어야 한다. 이 모델은 클라우드에 의존하던 기존의 AI 패러다임에서 벗어나, 사용자의 디바이스에 직접 내장되어 자율적이고, 사적이며, 맥락을 인지하는 인공지능으로의 전환을 상징한다.

Gemma는 Google의 주력 프론티어 모델인 Gemini와 동일한 연구 및 기술을 기반으로 구축된 경량 개방형 가중치(open-weight) 모델 제품군이다.1 'Gemma'라는 이름은 라틴어로 '보석'을 의미하며, 이는 강력한 Gemini 기술의 정수를 담고 있음을 시사한다.2 모델명의 '3n'에서 '3'은 3세대를, 'n'은 '나노(nano)'를 의미하며, 이는 모바일 및 엣지 디바이스를 위해 특별히 설계되었음을 명확히 한다.4 이러한 명명 전략은 접근성 높은 개방형 모델(Gemma)과 플래그십 모델(Gemini)의 강력한 기술적 혈통을 의도적으로 연결한다.

Gemma 3n은 명시적으로 "모바일 우선(mobile-first)"으로 설계되었으며, "모바일, 엣지 및 기타 리소스가 제한된 디바이스에서의 최고 효율성"을 목표로 개발되었다.4 이 모델의 아키텍처는 차세대 Gemini Nano와 공유되며, 이는 개발자들에게 향후 안드로이드(Android) 및 크롬(Chrome)과 같은 주요 플랫폼에 깊숙이 통합될 기술을 미리 경험해볼 수 있는 프리뷰 역할을 한다.7 2025년 5월 20일에 프리뷰가 공개되었고, 2025년 6월 26일에 정식 출시되었다.7 이 출시 일정은 Apple, Microsoft 등 경쟁사의 온디바이스 모델과의 치열한 경쟁 구도 속에서 Google의 전략적 속도전을 보여준다.

이러한 배경을 종합적으로 분석할 때, Gemma 3n의 출시는 단순한 개방형 모델 배포 이상의 의미를 지닌다. Google은 Gemma 3n이 차세대 Gemini Nano와 아키텍처를 공유하며, 개발자들이 이를 통해 Android와 Chrome에 탑재될 기능을 "미리 엿볼 수 있다"고 명시적으로 밝히고 있다.7 이는 Google의 독점적인 온디바이스 기술이 수십억 개의 Android 디바이스에 완전히 배포되기 전에, 관련 개발자 생태계를 미리 구축하려는 전략적 움직임이다. 개방형 가중치와 다양한 개발 도구를 제공함으로써 10, Google은 개발자들이 핵심 OS 기능으로 탑재될 바로 그 아키텍처 기반 위에서 애플리케이션을 구축하고, 문제를 해결하며, 혁신하도록 장려한다. 결과적으로 Gemma 3n의 핵심 전략 목표는, 새로운 OS 수준의 통합이 시작되는 첫날부터 풍부하고 준비된 "Gemmaverse" 4를 확보하는 것이다. 이는 Google의 온디바이스 AI 비전에 대한 진입 장벽을 낮추고 채택을 가속화하는, 일종의 생태계 선점 전략이라 할 수 있다.


Gemma 3n의 핵심 경쟁력은 극도로 제한된 메모리 환경 내에서 높은 성능을 제공하는 역설적인 능력에 있다. 이는 개별적인 기술적 트릭의 집합이 아니라, 동적이고 즉각적인 리소스 관리를 위해 설계된 응집력 있는 시스템의 결과물이다. 본 섹션에서는 이러한 성능을 가능하게 하는 핵심 기술 혁신들을 심층적으로 해부한다.


Gemma 3n은 러시아의 중첩 인형에 비유되는 "마트료시카 트랜스포머(Matryoshka Transformer)" 또는 **MatFormer**라는 새로운 아키텍처를 채택했다.5 이 구조는 더 큰 모델 내부에 더 작고 완벽하게 기능하는 하위 모델을 중첩시키는 개념을 기반으로 한다.8 구체적으로, E4B 모델(총 80억 파라미터)은 E2B 모델(총 50억 파라미터)을 최적화된 하위 모델로 포함하며, 두 모델은 함께 훈련되었다.10

이러한 구조는 개발자에게 두 가지 강력한 이점을 제공한다. 첫째, 개발자는 속도를 우선시할 경우 사전에 추출된 독립형 E2B 모델을 사용하거나, 최고 품질을 원할 경우 E4B 모델을 직접 사용할 수 있다. 둘째, **"Mix-n-Match"** 기능을 통해 계층의 너비와 깊이를 동적으로 조절하여 두 모델의 중간 크기에 해당하는 맞춤형 모델을 생성할 수 있다.7 Google은 개발자가 특정 하드웨어와 지연 시간/품질 요구사항에 맞는 최적의 맞춤형 구성을 찾을 수 있도록 

**MatFormer Lab**이라는 도구를 제공하며, 각 구성의 성능은 MMLU와 같은 벤치마크를 통해 평가된다.11

이 MatFormer 아키텍처는 정적인 AI 모델에서 "탄력적(elastic)" AI 모델로의 근본적인 전환을 예고하며, 이는 OS 수준의 리소스 관리에 새로운 차원을 제시한다. MatFormer는 단순히 두 개의 모델을 제공하는 것을 넘어, "탄력적 추론(elastic inference)" 11과 "즉각적인(on the fly)" 동적 선택 5을 위한 시스템으로 설명된다. 일부 자료에서는 "실시간으로 작업이나 디바이스 스트레스에 따라 크기를 변형하는" 탄력적 실행의 미래를 언급하기도 한다.12 이는 개발자가 컴파일 시점에 모델 크기를 선택하는 단계를 넘어, 운영체제(OS)나 상위 애플리케이션 관리자가 기존에 CPU 코어나 RAM을 관리하듯 AI 작업에 동적으로 파라미터를 할당하는 런타임 기능을 암시한다. 이는 Android와 같은 OS에서 AI의 '저전력 모드'를 구현하거나, 사용자가 복잡한 작업을 수행하고 디바이스가 충전 중일 때 AI의 성능을 동적으로 향상시키는 등의 시나리오를 가능하게 한다. 따라서 MatFormer는 단순한 모델 아키텍처가 아니라, 정적이고 단일한 애플리케이션이었던 LLM을 동적이고 확장 가능한 리소스로 변환하는 새로운 시스템 수준 서비스의 청사진이다. 이는 전력 제약이 심한 디바이스에서 깊고 원활한 OS 통합을 위한 필수 전제 조건이다.


Gemma 3n 모델은 **gemma-3n-E2B**와 **gemma-3n-E4B**로 명명되는데, 여기서 'E'는 "유효(Effective)" 파라미터를 의미한다.5 E2B 모델은 약 50억 개의 총 파라미터를 가지고 있지만 2B 모델 수준의 유효 메모리 공간을 차지하여 최소 2GB의 GPU RAM에서 실행될 수 있다. E4B 모델은 약 80억 개의 총 파라미터를 가지면서도 4B 모델 수준의 유효 공간을 차지하며 약 3GB의 RAM에서 작동한다.10

이러한 효율성은 주로 **계층별 임베딩(Per-Layer Embeddings, PLE)** 기술을 통해 달성된다. 이 기술은 모델의 파라미터 중 큰 비중을 차지하는 임베딩 그룹을 GPU/NPU와 같은 가속기 메모리가 아닌 CPU/메인 메모리로 오프로드한다.10 이를 통해 가속기에는 핵심적인 트랜스포머 블록만 유지하게 되어, 디바이스에서 요구되는 고속 메모리 사용량을 극적으로 줄일 수 있다. 추가적인 효율성은 **조건부 파라미터 로딩(Conditional Parameter Loading)**을 통해 확보된다. 이 기능은 특정 작업에 시각 또는 오디오 기능이 필요하지 않을 경우, 관련 파라미터 로드를 건너뛰어 메모리 리소스를 절약한다.15

이 "유효 파라미터"라는 명명법은 모델의 핵심 혁신을 부각시키고 개발자의 기대를 관리하기 위한 중요한 마케팅 및 프레이밍 장치이다. 기술적으로 모델은 5B와 8B 파라미터를 가지고 있지만 10, 이를 'E2B'와 'E4B'로 부르는 것은 의도적인 추상화이다. 온디바이스 환경에서 개발자의 주된 관심사이자 병목 지점은 이론적인 파라미터 수가 아니라 실제 VRAM 소비량이기 때문이다.10 모델을 메모리 점유율('유효 크기') 기준으로 명명함으로써, Google은 더 작은 모델의 하드웨어 요구사항으로 더 큰 모델의 품질을 얻을 수 있다는 핵심 가치를 즉각적으로 전달한다. 이는 "모델이 얼마나 큰가?"라는 질문에서 "내 디바이스가 실제로 무엇을 실행할 수 있는가?"라는 질문으로 대화의 초점을 전환시킨다. 이는 모델의 핵심 이점을 대상 개발자층이 즉시 이해할 수 있도록 만드는 영리한 제품 마케팅 전략이다.

| 기능                 | `gemma-3n-e2b-it`                 | `gemma-3n-e4b-it`                 |
| -------------------- | --------------------------------- | --------------------------------- |
| **모델 정체성**      | 명령어 튜닝 (Instruction-Tuned)   | 명령어 튜닝 (Instruction-Tuned)   |
| **유효 파라미터**    | 20억 (2B)                         | 40억 (4B)                         |
| **실제 파라미터**    | 약 50억 (5B) 10                   | 약 80억 (8B) 10                   |
| **최소 VRAM**        | 약 2 GB 10                        | 약 3 GB 10                        |
| **컨텍스트 창**      | 32,000 토큰 15                    | 32,000 토큰 15                    |
| **핵심 모달리티**    | 텍스트, 이미지, 오디오, 비디오 1  | 텍스트, 이미지, 오디오, 비디오 1  |
| **코어 아키텍처**    | MatFormer (하위 모델) 10          | MatFormer (메인 모델) 10          |
| **주요 효율화 기술** | PLE, KV 캐시 공유, 조건부 로딩 10 | PLE, KV 캐시 공유, 조건부 로딩 10 |


Gemma 3n은 **KV 캐시 공유(KV Cache Sharing)** 기술을 사용하여 처리 속도를 높인다. 이 기술은 특히 프롬프트가 처음 처리되는 "프리필(prefill)" 단계에서 효과적이다.10 이 기법은 트랜스포머의 상위 계층이 중간 계층에서 이미 계산된 키(Key)와 값(Value)을 재사용하여 중복 계산을 피하도록 한다.11 그 결과, Gemma 3 4B 모델에 비해 프리필 성능이 약 2배 향상되었으며, 이는 긴 오디오나 비디오 스트림과 같은 긴 컨텍스트 입력 처리 시 특히 유용하다.7

이 KV 캐시 공유 기술은 온디바이스 멀티모달 AI의 치명적인 사용자 경험 문제, 즉 초기 응답 지연을 해결하기 위한 직접적인 솔루션이다. 온디바이스 AI는 "실시간 양방향 경험"을 목표로 한다.7 사용자가 음성 명령을 내리거나 객체를 보여준 후 긴 지연(프리필 지연)이 발생하면 이러한 상호작용의 환상은 깨지고 시스템은 느리게 느껴진다.12 긴 오디오 클립이나 비디오 프레임과 같은 멀티모달 입력은 초기 프롬프트의 크기를 극적으로 증가시켜 프리필 지연 문제를 더욱 악화시킨다. KV 캐시 공유는 바로 이 병목 현상을 직접적으로 겨냥한다. 프리필 성능의 2배 향상은 10 단순한 기술적 벤치마크가 아니라, 모델이 더 반응적이고 '살아있는' 것처럼 느껴지게 만드는 사용자 대면 개선 사항이다. 따라서 이 혁신은 순수한 처리량보다는 인지된 응답성과 상호작용성을 향상시키는 데 중점을 두며, 이는 Gemma 3n이 지향하는 실시간, 이동 중 AI 애플리케이션의 성공에 결정적인 요소이다.


본 섹션에서는 Gemma 3n에 '감각'을 부여하는 하드웨어 기반 인코더들을 상세히 분석한다. 모델이 단순히 멀티모달이라는 사실을 넘어, 어떻게 보고 듣는지, 그리고 이것이 실시간 대화형 애플리케이션을 구축하는 개발자에게 어떤 실질적인 능력을 제공하는지를 설명한다.


Gemma 3n은 새로운 비전 인코더인 **MobileNet-V5**(특히 3억 파라미터 변종)를 사용한다.10 이는 이전 Gemma 모델의 비전 시스템에서 크게 업그레이드된 것이다. 성능 면에서 이 인코더는 가장 큰 MobileNet-V4 변종보다 10배 더 큰 하이브리드 심층 피라미드 모델이다.11 Google Pixel 디바이스에서 초당 60프레임(FPS)을 달성하며, Gemma 3의 SoViT 인코더보다 13배 빠르고 메모리 사용량은 4배 더 작다.10 유연성 측면에서는 256x256, 512x512, 768x768 등 여러 입력 해상도를 기본적으로 지원하여 개발자가 디테일과 성능 사이의 균형을 맞출 수 있도록 한다.10 또한, 새로운 다중 스케일 융합 VLM(Multi-Scale Fusion VLM) 어댑터를 사용하여 토큰의 품질을 향상시켜 정확도를 높였다.11

MobileNet-V5의 선택은 고해상도 정적 이미지 분석보다 실시간 비디오 처리를 우선시하는 설계 철학을 명확히 보여준다. MobileNet-V5의 핵심 성능 지표로 "Google Pixel에서 60 FPS"가 강조된다는 점이 이를 뒷받침한다.10 FPS는 정적 이미지가 아닌 비디오를 위한 측정 단위이다. 이는 주된 설계 목표가 단순한 '이미지 이해'를 넘어, 라이브 증강현실(AR) 오버레이, 실시간 객체 추적, 대화형 비디오 분석과 같은 지속적이고 실시간적인 시각 정보 처리에 있었음을 시사한다. 비교적 낮은 해상도(256, 512, 768)를 지원하는 것 또한 실시간 비디오 스트림이 프레임 속도를 유지하기 위해 이러한 해상도에서 작동하는 경우가 많다는 점에서 이 주장을 뒷받침한다. 따라서 MobileNet-V5의 탑재는 Google의 온디바이스 AI 비전이 라이브 카메라 피드와 깊이 연결되어 있으며, "사용자 환경의 실시간 시각적 신호를 이해하고 반응하는" 7 애플리케이션을 가능하게 하려는 의도를 강력하게 나타낸다.


Gemma 3n의 오디오 기능은 Google의 **범용 음성 모델(Universal Speech Model, USM)**에 기반한다.10 이를 통해 고품질의 온디바이스 자동 음성 인식(ASR, 스크립트 작성) 및 자동 음성 번역(AST, 음성을 번역된 텍스트로 변환)이 가능하다.7 이 모델은 오디오를 160ms 단위로 처리하며, 기본적으로 최대 30초 길이의 클립을 다룰 수 있다. 또한, 더 긴 입력을 위한 스트리밍 가능 아키텍처를 갖추고 있다.10 특히 영어와 스페인어, 프랑스어, 이탈리아어, 포르투갈어 간의 번역에서 강력한 성능을 보이는 것으로 알려져 있다.11

온디바이스 ASR/AST를 위한 USM의 통합은 클라우드 기반 음성 비서 API에 대한 직접적인 도전이자, 프라이버시 중심 애플리케이션의 핵심 동력이다. 역사적으로 고품질 음성 인식 및 번역은 클라우드에서 수행되는 계산 집약적인 작업이었으며(예: Google 번역 API, Alexa, Siri), 이는 네트워크 연결을 요구하고 잠재적으로 민감한 음성 데이터를 서버로 전송해야 했다. USM과 같은 강력한 모델을 개방형 온디바이스 모델에 직접 통합함으로써, Google은 이 기능을 효과적으로 민주화하고 로컬화하고 있다.13 이는 오프라인에서 이러한 작업을 수행할 수 있는 새로운 종류의 애플리케이션을 가능하게 하며, 프라이버시(데이터가 디바이스를 떠나지 않음), 접근성(연결성이 낮은 지역에서도 작동), 비용(API 수수료 없음) 측면에서 막대한 이점을 제공한다. 따라서 USM의 포함은 음성 상호작용의 가치를 클라우드 서비스에서 로컬 디바이스 기능으로 전환하려는 전략적 움직임이며, 이는 모델의 "프라이버시 우선 및 오프라인 준비" 19 마케팅과 완벽하게 일치하고 클라우드 전용 음성 솔루션에 대한 경쟁 우위를 창출한다.


Gemma 3n의 핵심 기능 중 하나는 여러 모달리티에 걸쳐 **인터리브된 입력(interleaved inputs)**을 수용하는 능력이다.5 이는 단일 시퀀스 내에서 텍스트, 이미지, 오디오가 혼합된 프롬프트를 이해할 수 있음을 의미한다. 이를 통해 사용자가 이미지 속 객체를 가리키며 음성으로 질문하는 것과 같은 "복잡한 멀티모달 상호작용"을 이해할 수 있다.7 또한 이 모델은 "상당히 향상된 비디오 이해" 능력을 갖추고 있는데 7, 이는 빠른 비전 인코더와 일련의 프레임을 처리하는 언어 모델의 능력이 결합된 직접적인 결과이다.

이 인터리브 입력 기능은 단순한 질의응답을 넘어 실제 작업을 수행하는 진정한 "에이전트적(agentic)" AI를 디바이스 상에서 구현하기 위한 기술적 토대이다. 단순한 멀티모달 모델은 이미지를 보거나 음성을 들을 수 있지만, 인터리브 모델은 `[텍스트: "이게 뭐야?"] -> [이미지: object.jpg] -> [텍스트: "그리고 이 소리 좀 받아적어 줄래?"] -> [오디오: sound.wav]`와 같은 순차적이고 복합적인 입력을 처리할 수 있다. 이러한 순차적, 다중 형식의 이해 방식은 인간이 복잡한 작업을 처리하는 방식과 유사하며, 다양한 유형의 정보를 연결하는 추론의 기초가 된다. 관련 문서에서는 Gemma 3n이 사용자 입력을 사용하여 "앱 실행, 알림 전송, 손전등 켜기 등 디바이스에서 특정 작업을 직접 시작하거나 호출"할 수 있다고 언급한다.5 이는 AI 에이전트의 정의 그 자체이다. 따라서 인터리브 입력 기능은 단순한 특징이 아니라, Gemma 3n을 수동적인 '이해' 모델에서 능동적인 '수행' 에이전트로 격상시키는 필수적인 아키텍처 구성 요소이다. 이는 모델이 환경적 단서(시각, 청각)와 사용자 의도(텍스트)를 융합하여 행동을 취할 수 있는 컨텍스트를 구축하게 하며, 이는 차세대 지능형 개인 비서의 초석이 된다.


본 섹션에서는 Gemma 3n의 성능을 데이터 기반으로 엄격하게 평가한다. 먼저 주요 국제 벤치마크 점수를 통해 모델의 전반적인 역량을 확인한 후, 다국어 성능, 특히 한국어 능력에 대한 상세하고 비판적인 분석으로 초점을 전환한다.


Gemma 3n은 여러 국제 벤치마크에서 주목할 만한 성과를 거두었다. 특히 `gemma-3n-e4b` 모델은 **LMArena Elo 점수**에서 1300점을 돌파한 최초의 100억 파라미터 미만 모델이 되었다.10 이는 챗봇 스타일의 비교 평가에서 강력한 사용자 선호도를 나타내는 중요한 이정표이다.

**MMLU(Massive Multi-Task Language Understanding)** 벤치마크에서는 E4B 모델이 경쟁력 있는 성능을 보였으며, 다양한 "Mix-n-Match" 구성 또한 MMLU에서 평가되어 품질과 크기 간의 상충 관계를 보여주었다.10 구체적으로 E2B-IT 모델의 MMLU 점수는 약 60.1%로 기록되었다.20 이 외에도 HellaSwag(E2B-IT 기준 72.2%), BoolQ, DROP과 같은 상식 추론 벤치마크와 HumanEval, MBPP와 같은 수학 및 코딩 벤치마크에서도 경쟁력 있는 점수를 나타냈다.21

LMArena 벤치마크에 대한 집중은 순수한 학술적 지식보다 대화 및 명령어 추종 능력을 강조하려는 전략적 선택으로 볼 수 있다. 이는 온디바이스 비서에게 더 적합한 능력이다. MMLU와 같은 학술적 벤치마크가 모델의 '지식'을 테스트하는 반면, LMArena는 사용자가 어떤 모델과 대화하는 것을 '선호'하는지를 크라우드소싱 방식으로 평가한다. 이는 실제 대화 맥락에서 유용성, 일관성, 명령어 추종과 같은 자질을 측정한다. Gemma 3n과 같은 온디바이스 모델의 주요 사용 사례는 학술 논문을 작성하는 것이 아니라, 비서를 구동하고, 실시간 도움을 제공하며, 사용자와 자연스럽게 상호작용하는 것이다. ">1300 LMArena 점수"를 눈에 띄게 강조함으로써 10, Google은 Gemma 3n이 기술적으로 능숙할 뿐만 아니라, 좋은 AI 비서를 정의하는 실용적이고 사용자 대면적인 작업에서도 뛰어나다는 신호를 보내고 있다. 이는 모델이 무엇을 '아는가'에서 얼마나 잘 '상호작용하는가'로 초점을 전환한 것이다.


Gemma 3n은 텍스트의 경우 140개 이상의 언어, 멀티모달 상호작용의 경우 35개 언어를 지원하는 데이터로 훈련되었다.1 토크나이저는 Gemini 2.0과 동일한 262,000개 항목의 SentencePiece 토크나이저를 사용하며, 특히 중국어, 일본어, **한국어** 텍스트에 대한 인코딩이 크게 개선되었다.22 공식 발표에서는 "일본어, 독일어, **한국어**, 스페인어, 프랑스어에서 향상된 다국어 성능"을 강조했다.7 Google이 인용한 주요 다국어 벤치마크는 E4B 모델이 WMT24++ (ChrF)에서 50.1%의 점수를 기록했다는 것이다.7


Google은 공식적으로 한국어를 "성능이 향상된" 언어로 명시하고 있다.7 이전 Gemma 세대에 대한 초기 커뮤니티 피드백도 긍정적이었으며, 사용자들은 한국어 이해 및 생성 능력이 "나쁘지 않다"고 평가했다.25 일부 커뮤니티 구성원들은 Gemma 3가 동아시아 언어에 중점을 둔 만큼, 이 분야에서 경쟁 모델보다 우수할 것으로 기대하기도 했다.26

한국어 LLM 성능 평가의 사실상 표준은 **Open Ko-LLM Leaderboard**이다.27 이 리더보드는 Ko-HellaSwag, Ko-MMLU 등 한국어 특화 벤치마크 모음을 사용하여 표준화된 평가를 제공한다. 그러나 현재까지 수집된 자료에서는 Gemma 3n이 이 리더보드에서 기록한 구체적인 점수를 찾을 수 없다. 한 유튜브 리뷰에서 4B 모델이 성능 테스트에서 "42점"을 받았다고 언급했지만, 해당 점수의 벤치마크와 맥락은 불분명하다.30

이는 Google의 Gemma 3n 한국어 성능 마케팅과 실제 검증 가능한 데이터 사이에 중대한 단절이 있음을 시사하며, 이는 위험과 기회를 동시에 내포한다. Google의 공식 발표 7와 기술 문서 23는 WMT24++ 벤치마크를 인용하며 한국어 성능 향상을 강조한다.17 그러나 한국 AI 커뮤니티는 Open Ko-LLM Leaderboard를 사실상의 표준으로 간주하고 있다.27 Google이 출시 자료에서 이 리더보드의 점수를 인용하지 않았다는 점은 중요한 누락이다. 이는 해당 벤치마크로 테스트하지 않았거나, 결과가 WMT24++ 점수만큼 유리하지 않았을 가능성을 시사한다. 이 보고서의 대상 독자인 한국 AI 전문가들에게 Open Ko-LLM Leaderboard에서의 성능은 가장 의미 있는 지표이다. 이 데이터 포인트의 부재는 Google의 주장이 커뮤니티의 자체 표준에 의해 완전히 검증될 수 없음을 의미한다. 따라서 본 보고서는 이 정보 격차를 명시적으로 다루어야 한다. Google의 주장과 WMT24++ 데이터를 제시하되, Open Ko-LLM Leaderboard의 중요성을 설명하고, 작성 시점 기준으로 이 핵심 벤치마크에 대한 공식 점수가 없음을 명시해야 한다. 이는 모델의 한국어 성능에 대해 공식적인 주장과 공개 데이터의 한계를 모두 인정하는, 미묘하고 비판적이며 정직한 평가를 제공한다.

| 벤치마크                    | 모델                      | 점수                    | 출처 / 맥락                                    |
| --------------------------- | ------------------------- | ----------------------- | ---------------------------------------------- |
| **다국어 번역**             | `gemma-3n-e4b-it`         | 50.1% (ChrF)            | WMT24++ (Google 공식) 17                       |
| **Open Ko-LLM (평균 점수)** | `gemma-3n-e4b-it`         | **N/A**                 | *수집된 자료에 데이터 없음. 중요한 정보 격차.* |
| **Open Ko-LLM (평균 점수)** | `[선두 한국어 모델]`      | `[점수]`                | Open Ko-LLM Leaderboard 31                     |
| **Open Ko-LLM (평균 점수)** | `[비교 가능한 경쟁 모델]` | `[점수]`                | Open Ko-LLM Leaderboard 31                     |
| **정성적 평가**             | Gemma 시리즈              | "나쁘지 않음", "준수함" | 커뮤니티 피드백 25                             |


본 섹션에서는 이론적인 모델에서 벗어나 실제 적용 사례로 초점을 전환하여, 개발자들이 사용할 수 있는 풍부한 도구, 플랫폼, 배포 옵션 생태계를 상세히 설명한다. 이는 Google의 전략이 단순히 모델을 출시하는 데 그치지 않고, 기존 워크플로우에 원활하게 통합하여 활발한 "Gemmaverse"를 육성하는 데 있음을 보여준다.


Gemma 3n은 개발자들의 접근성과 활용성을 극대화하기 위한 포괄적인 생태계를 갖추고 있다. 모델 가중치는 **Hugging Face**와 **Kaggle**을 통해 공개적으로 제공되어 누구나 쉽게 다운로드할 수 있다.10 초기 실험과 프로토타이핑을 위해 

**Google AI Studio**는 별도의 설정 없이 클릭 몇 번으로 모델을 시험해 볼 수 있는 환경을 제공한다.7

온디바이스 개발을 위해 **Google AI Edge, Ollama, MLX, llama.cpp, transformers.js** 등 광범위한 프레임워크가 지원된다.10 배포 옵션 또한 로컬 환경부터 

**Google GenAI API, Vertex AI, Cloud Run**과 같은 클라우드 서비스, 그리고 **NVIDIA API Catalog**와 같은 제3자 서비스에 이르기까지 다양하다.11 하드웨어 최적화 측면에서는 

**Google Cloud TPU, NVIDIA GPU**(Jetson부터 Blackwell까지), **AMD GPU**(ROCm 경유) 등 주요 플랫폼에 걸쳐 최적화가 이루어져 있다.1

Google의 이러한 "급진적으로 개방된" 생태계 전략은 Apple과 같은 경쟁사의 보다 폐쇄적이고 수직적으로 통합된 접근 방식에 대한 직접적인 대응책이다. Apple의 온디바이스 AI 전략(Apple Intelligence)은 OS와 하드웨어에 깊숙이 통합되어 있지만 폐쇄적인 생태계이다. 개발자들은 상위 수준의 API를 통해 상호작용할 뿐, 기본 모델에 접근하거나 수정할 수 없다.33 반면, Google의 Gemma 3n 전략은 정반대이다. API만 제공하는 것이 아니라 원시 모델 가중치를 제공하고 11, Ollama, llama.cpp 등 방대한 제3자 오픈소스 도구와의 호환성을 보장한다.10 이는 개발자들을 단일 독점 도구 체인에 묶어두려는 시도 대신, 기존의 오픈소스 AI 커뮤니티를 포용하려는 의도적인 선택이다. 이는 마찰을 줄이고 개발자들이 이미 익숙한 환경에서 작업할 수 있도록 함으로써 채택을 장려한다. 따라서 광범위한 지원 도구 목록은 단순한 기능 목록이 아니라, Gemma 3n을 중심으로 광범위한 커뮤니티 주도 생태계를 육성하려는 전략적 결정이며, 개방형 커뮤니티의 네트워크 효과가 폐쇄형 시스템의 세련됨을 능가할 것이라는 데 베팅하는 것이다.


Google은 혁신을 장려하기 위해 Kaggle에서 총상금 15만 달러 규모의 **Gemma 3n 임팩트 챌린지**를 시작했다.11 이 챌린지의 목표는 개발자들이 Gemma 3n의 고유한 온디바이스, 오프라인, 멀티모달 기능을 사용하여 "더 나은 세상을 위한" 제품을 만드는 것이다.11 제안된 중점 분야로는 청각 장애인을 위한 실시간 번역과 같은 접근성 향상, 저연결성 지역의 교육, 건강 및 웰빙, 환경 지속 가능성, 위기 대응 등이 포함된다.19

제출물은 **영향력 및 비전(40점), 비디오 피치 및 스토리텔링(30점), 기술적 깊이 및 실행(30점)**을 기준으로 심사되며, 실제 적용 사례와 모델 기능의 혁신적인 사용을 강조한다.19 (참고: 제공된 자료에는 챌린지 우승 프로젝트에 대한 정보는 없으며, 챌린지 발표 및 세부 정보만 포함되어 있다.)

이 임팩트 챌린지는 온디바이스 멀티모달 AI의 "킬러 앱"을 신속하게 발견하기 위해 고안된, 아웃소싱된 커뮤니티 주도 R&D의 한 형태이다. 온디바이스 멀티모달 AI는 새로운 기술적 개척지이다. Google이 잠재적 사용 사례(접근성, 교육 등)를 제안할 수는 있지만, 모든 창의적인 애플리케이션을 상상하기는 불가능하다. 상당한 상금이 걸린 이 챌린지는 전 세계 개발자 커뮤니티가 새로운 애플리케이션을 브레인스토밍하고 프로토타이핑하는 데 집중적인 노력을 기울이도록 유도한다. "영향력 및 비전"과 설득력 있는 "비디오 데모"에 큰 비중을 두는 심사 기준은 기술적으로 영리한 해킹뿐만 아니라 강력하고 시장성 있는 제품 컨셉을 발굴하기 위해 설계되었다. 따라서 이 챌린지는 이중의 목적을 가진다. 공개적으로는 사회적 선을 위한 혁신을 육성하는 것이지만, 전략적으로는 Google이 새로운 기술에 대한 가장 설득력 있고 바이럴 가능성이 있는 사용 사례를 크라우드소싱하는 매우 효율적인 방법이다. 이는 향후 개발 및 마케팅 활동을 안내할 풍부한 실제 사례 포트폴리오를 Google에 제공한다.


본 섹션에서는 Gemma 3n을 치열한 경쟁 환경의 맥락에 배치한다. 다른 주요 소형 언어 모델과의 데이터 기반 직접 비교를 제공하고, 특히 Apple을 상대로 한 신흥 온디바이스 AI 시장의 지배력 확보를 위한 전략적 싸움을 분석한다.


Gemma 3n은 다른 개방형 소형 언어 모델(SLM)들과의 경쟁에서 뚜렷한 특징을 보인다. **Llama 3**와의 비교에서, Gemma 3n E2B-IT와 Llama 3.2 3B Instruct를 비교했을 때, Llama 3.2가 대부분의 추론 벤치마크(MMLU, GPQA, ARC-C)에서 더 나은 성능을 보인 반면, Gemma 3n E2B는 상식 추론(HellaSwag)에서 우세했다.20 그러나 가장 큰 차이점은 Gemma 3n이 Llama 3.2가 지원하지 않는 멀티모달 입력을 지원한다는 점이다.20 컨텍스트 창 크기는 Gemma 3n(32K)이 Llama 3.2(128K)보다 작다.20

Microsoft의 **Phi-3/Phi-4** 모델과의 경쟁에서도 유사한 양상이 나타난다. Phi 모델들은 소형 모델 분야에서 강력한 경쟁자로, 특히 추론과 수학에서 뛰어난 성능을 보인다.34 Gemma 3n E2B와 Phi 4 Mini Reasoning을 비교한 결과, Phi 4가 GPQA 벤치마크에서 52.0% 대 24.8%로 월등히 높은 점수를 기록했다.36 여기서도 Gemma 3n의 핵심 차별점은 Phi 4 Mini가 갖추지 못한 네이티브 멀티모달 기능이다.36

이러한 경쟁 구도는 전략적 분기를 명확히 보여준다. Meta와 Microsoft와 같은 경쟁사들이 소형 모델에서 순수 텍스트 기반 추론 능력을 최적화하는 데 집중하는 반면, Google은 온디바이스 사용 사례의 핵심 차별점으로 멀티모달리티에 베팅하고 있다. 벤치마크 비교 20에 따르면, 순수 텍스트 기반 추론 작업(MMLU, GPQA)에서 Gemma 3n은 경쟁력이 있지만, 종종 Meta(Llama)나 Microsoft(Phi)의 동급 모델에 비해 절대적인 선두는 아니다. 그러나 모든 비교에서 Gemma 3n의 고유한 네이티브 멀티모달 기능(이미지, 오디오)은 경쟁사들의 소형 개방형 모델에는 없는 특징으로 강조된다.20 이는 Google의 의도적인 전략적 선택을 시사한다. 텍스트 전용 벤치마크에서 점진적인 이득을 얻기 위해 싸우는 대신, 그들은 경쟁의 장을 바꾸고 있다. 따라서 Google의 전략은 순수 추론에서의 작은 우위를 양보하는 대신, 온디바이스 '멀티모달 상호작용'에서 압도적인 우위를 점하는 것이다. 그들은 휴대폰이나 엣지 디바이스의 경우, 보고 들을 수 있는 능력이 텍스트 전용 시험에서 몇 점 더 얻는 것보다 더 혁신적일 것이라고 판단하고 있다.

| 벤치마크         | `Gemma 3n E2B-IT` | `Gemma 3n E4B-IT`              | `Llama 3.2 3B-IT` | `Phi 4 Mini Reasoning` | 핵심 분석                           |
| ---------------- | ----------------- | ------------------------------ | ----------------- | ---------------------- | ----------------------------------- |
| **LMArena Elo**  | ~1293 33          | **>1300** 10                   | N/A               | N/A                    | Gemma 3n은 대화 능력에서 탁월함.    |
| **MMLU**         | 60.1% 20          | E4B의 MMLU 점수 경쟁력 있음 10 | **63.4%** 20      | N/A                    | Llama 3.2는 광범위한 지식에서 선두. |
| **GPQA**         | 24.8% 20          | 23.7% (PT) 21                  | 32.8% 20          | **52.0%** 36           | Phi 4는 우수한 추론 능력을 보임.    |
| **HellaSwag**    | **72.2%** 20      | 78.6% (PT) 21                  | 69.8% 20          | N/A                    | Gemma 3n은 상식 추론에 강함.        |
| **멀티모달리티** | **지원** 1        | **지원** 1                     | 미지원 20         | 미지원 36              | Gemma 3n의 핵심 차별점.             |


온디바이스 AI 시장의 패권을 둘러싼 가장 중요한 전투는 Google과 Apple 사이에서 벌어지고 있다. Google은 Apple이 자사의 온디바이스 "Apple Intelligence" 기능을 완전히 출시하기 전에, 강력한 개방형 가중치 멀티모달 모델인 Gemma 3n을 먼저 출시했다.6 두 회사 모두 속도와 프라이버시를 위해 소형 모델을 온디바이스에서 실행하고, 더 복잡한 쿼리는 클라우드로 오프로드하는 하이브리드 접근 방식을 추구하며, OS에 깊이 통합하는 것을 강조한다.33

그러나 근본적인 차이는 개방성과 폐쇄성에 있다. Google의 접근 방식은 개방적(Gemma 3n 가중치 공개)인 반면, Apple의 접근 방식은 폐쇄적(개발자는 모델 자체가 아닌 API 사용)이다.6 Apple은 자사의 온디바이스 모델이 Gemma 3-4B와 경쟁력이 있다고 주장하지만, 표준 벤치마크를 제공하지 않아 직접적인 비교가 어렵다. 한 전문가는 표준 벤치마크 기준으로 볼 때 Apple의 모델이 더 약해 보인다고 지적했다.33

Gemma 3n의 출시는 Apple에 대한 플랫폼 전쟁에서 전략적인 선제공격이다. 이는 Apple의 생태계가 완전히 성숙하기 전에 개발자들의 마음을 사로잡기 위해 개방성을 무기로 활용하는 것이다. 온디바이스 AI를 둘러싼 궁극적인 전투는 두 지배적인 모바일 운영체제인 Android(Google)와 iOS(Apple) 사이에서 벌어질 것이다. Apple의 강점은 하드웨어, 소프트웨어, 서비스의 긴밀한 수직적 통합("walled garden")이지만, 약점은 폐쇄적인 생태계로 인해 개발자에게 유연성이 부족하다는 점이다. Google은 Gemma 3n을 통해 자사의 강점을 활용하고 있다. 미래의 OS 통합 AI(Gemini Nano) 아키텍처를 반영하는 강력한 '개방형 가중치' 모델을 출시함으로써, iOS 개발자를 포함한 전 세계 개발자 커뮤니티가 '지금 당장' Google의 기술로 구축, 실험, 혁신하도록 초대하고 있다. 이는 강력한 역학을 만들어낸다. 개발자는 오늘날 Gemma 3n을 사용하여 최첨단 멀티모달 온디바이스 AI 앱을 구축하고 Android를 포함한 어디에나 배포할 수 있다. iOS에서 동등한 작업을 수행하려면 Apple의 도구를 기다려야 하며 Apple의 규칙에 제약을 받는다. 따라서 Gemma 3n은 플랫폼 전쟁의 도구이다. 이는 Android 생태계를 온디바이스 AI 개발을 위한 기본적이고, 가장 매력적이며, 가장 유연한 환경으로 만들어 최고의 앱과 인재를 유치하고, 폐쇄적인 경쟁사보다 더 활기찬 애플리케이션 생태계를 조성하는 것을 목표로 한다.


Google은 강력한 AI 기술의 민주화에 내재된 위험을 완화하고 신뢰를 구축하기 위한 핵심 제품 전략의 일환으로, Gemma 3n과 함께 책임감 있는 AI 개발을 촉진하기 위한 포괄적인 도구 및 방법론 모음을 출시했다. 이는 단순한 부가 기능이 아니라 제품의 핵심적인 부분이다.

Gemma 모델은 Google의 AI 원칙을 최우선으로 고려하여 설계되었다. 훈련 데이터는 민감한 개인 정보를 제거하기 위해 자동화된 필터링을 거쳤으며, 명령어 튜닝 모델은 안전을 위해 인간 피드백 기반 강화 학습(RLHF)을 통해 조정되었다.3 Google은 책임감 있는 개방형 모델 사용을 위해 **책임감 있는 생성형 AI 툴킷(Responsible Generative AI Toolkit)**을 제공한다.3

이 툴킷은 다음과 같은 요소로 구성된다:

- **가이드라인:** 안전 정책 설정, 안전 튜닝(SFT, RLHF), 모델 평가에 대한 모범 사례를 제공한다.37

- **도구:** 프롬프트 디버깅을 위한 **학습 해석 가능성 도구(Learning Interpretability Tool, LIT)**, 모델 간 비교 평가를 위한 **LLM 비교기(LLM Comparator)**, 맞춤형 안전 분류기 구축 방법론 등을 포함한다.37

- **안전장치:** Gemma 3 기반의 40억 파라미터 이미지 안전 검사기인 **ShieldGemma 2** 32, 워터마킹을 위한 

  **SynthID**, 유해성 탐지를 위한 **Perspective API**와 같은 도구와 API를 제공한다.38

또한 Google은 수동 레드팀 구성, 자동화된 적대적 테스트 등 강력한 내부 평가를 수행하며, 그 결과는 모델 카드에 투명하게 공개된다.1

이 책임감 있는 AI 툴킷은 기업의 개방형 모델 채택에 따르는 위험을 줄이고, 동시에 Google의 프레임워크를 사실상의 산업 표준으로 확립하려는 전략적 도구이다. 오픈소스 AI를 채택하는 데 있어 기업의 주된 장벽 중 하나는 브랜드 안전, 데이터 프라이버시, 오용 가능성과 같은 인지된 위험이다. 기업들은 강력한 안전장치 없이는 모델 배포를 주저한다. 포괄적인 '책임감 있는 AI 툴킷'을 사전 패키지로 제공함으로써 37, Google은 이러한 우려를 직접적으로 해결한다. 그들은 기업에 강력한 엔진(Gemma)뿐만 아니라, 조향 장치, 브레이크, 안전벨트(툴킷)까지 제공하는 셈이다. 이는 법무 및 규정 준수팀이 승인할 수 있는 구체적인 프레임워크를 제공하여 채택 장벽을 낮춘다. 이는 대기업이 단순히 저장소에서 모델을 다운로드하는 것보다 Gemma를 사용하는 것을 더 방어 가능하고 관리하기 쉬운 선택으로 만든다. 동시에, 이 툴킷을 개방적이고 포괄적으로 만듦으로써, Google은 자사의 방법론(안전 튜닝, 평가, 분류에 대한 접근 방식)을 전체 오픈소스 커뮤니티의 기본 표준으로 확립하려고 시도한다. 이는 Google을 생태계의 사상적 리더이자 책임감 있는 관리자로 자리매김하게 하여 상당한 소프트 파워와 영향력을 구축하게 한다.


본 보고서는 Google의 Gemma 3n 모델을 기술적 아키텍처, 멀티모달 성능, 개발자 생태계, 그리고 경쟁 시장에서의 전략적 위치 등 다각도에서 심층적으로 분석했다. 분석을 통해 도출된 핵심 결론과 이를 바탕으로 한 구체적인 제언은 다음과 같다.

**핵심 결론 종합:**

1. **차세대 온디바이스 AI의 청사진:** Gemma 3n의 핵심 아키텍처인 MatFormer와 PLE는 정적인 모델에서 벗어나, 향후 모바일 디바이스의 운영체제에 통합될 "탄력적" AI의 청사진을 제시한다. 이는 AI가 고정된 애플리케이션이 아닌, 시스템이 동적으로 관리하는 리소스가 될 미래를 예고한다.
2. **멀티모달리티를 통한 차별화:** Gemma 3n의 진정한 경쟁 우위는 순수 텍스트 추론 성능이 아니라, MobileNet-V5와 USM을 통한 네이티브 실시간 멀티모달 능력에 있다. 이는 사용자의 환경과 직접 상호작용하는 '에이전트적' 경험을 구현하는 데 필수적이다.
3. **한국어 성능의 불확실성:** Google은 공식적으로 한국어 성능 향상을 주장하지만, 한국 AI 커뮤니티의 표준 벤치마크인 Open Ko-LLM Leaderboard에서의 검증된 데이터가 부재하다. 이는 한국어 기반 애플리케이션을 개발하려는 이들에게 신중한 자체 평가의 필요성을 제기하는 중요한 정보 격차이다.
4. **개방성을 통한 플랫폼 전쟁:** Google의 전략은 Apple과 같은 폐쇄적 생태계에 맞서, 개방성을 무기로 더 크고 활발한 개발자 커뮤니티를 육성하여 플랫폼 전쟁에서 승리하는 것이다. 이는 기술력뿐만 아니라 생태계 전략이 온디바이스 AI 시장의 패권을 결정할 것임을 시사한다.

**분야별 전략적 제언:**

- **개발자를 위한 제언:** Gemma 3n의 고유한 가치가 있는 멀티모달 및 "Mix-n-Match" 기능 탐색에 우선순위를 두어야 한다. 특히 한국어 애플리케이션 개발 시에는 국제 벤치마크에만 의존하지 말고, 목표 사용 사례에 맞는 독립적인 테스트를 반드시 수행하여 실제 성능을 검증해야 한다.
- **연구자를 위한 제언:** MatFormer 아키텍처가 가진 "탄력적 실행"의 잠재력을 심도 있게 연구하고, PLE 캐싱 메커니즘이 다양한 하드웨어에서 성능과 전력 효율에 미치는 상충 관계를 분석할 필요가 있다.
- **기술 전략가를 위한 제언:** 온디바이스 AI 시장이 Google이 주도하는 개방형 생태계와 Apple이 주도하는 폐쇄형 생태계로 양분되고 있음을 인지해야 한다. 플랫폼 전략 수립 시, 각 생태계가 제공하는 유연성, 혁신 속도, 통제 수준의 차이를 고려해야 한다.

**미래 전망:**

향후 Gemma "n" 시리즈는 탄력적 실행과 OS와의 더욱 긴밀한 통합을 향해 발전할 것으로 예상된다. 또한, MedGemma, CodeGemma와 같이 3n의 기반 위에 구축된 더 전문화된 변종 모델들이 등장할 것이다.4 경쟁사들은 온디바이스 멀티모달 기능을 자체 개발하거나 텍스트 기반 성능에 더욱 집중하게 되어, 시장의 전략적 분화는 더욱 뚜렷해질 것이다. 온디바이스 AI 분야는 의심할 여지 없이 AI 패권을 둘러싼 차세대 핵심 격전지가 될 것이다.


1. Gemma 3n – Vertex AI - Google Cloud console, accessed July 16, 2025, https://console.cloud.google.com/vertex-ai/publishers/google/model-garden/gemma3n?jsmode
2. Gemma models overview | Google AI for Developers - Gemini API, accessed July 16, 2025, https://ai.google.dev/gemma/docs
3. Gemma: Introducing new state-of-the-art open models - Google Blog, accessed July 16, 2025, https://blog.google/technology/developers/gemma-open-models/
4. Gemma 3n - Google DeepMind, accessed July 16, 2025, https://deepmind.google/models/gemma/gemma-3n/
5. What is Gemma 3n and How to Access it? - Analytics Vidhya, accessed July 16, 2025, https://www.analyticsvidhya.com/blog/2025/05/gemma-3n/
6. Gemma 3n: Google's open-weight AI model that brings on-device intelligence - Digit, accessed July 16, 2025, https://www.digit.in/features/general/gemma-3n-googles-open-weight-ai-model-that-brings-on-device-intelligence.html
7. Announcing Gemma 3n preview: powerful, efficient, mobile-first AI - Google Developers Blog, accessed July 16, 2025, https://developers.googleblog.com/en/introducing-gemma-3n/
8. Announcing Gemma 3n Preview: Powerful, Efficient, Mobile-First AI - YouTube, accessed July 16, 2025, https://www.youtube.com/watch?v=eJFJRyXEHZ0
9. Introducing Gemma 3n - Hacker News, accessed July 16, 2025, https://news.ycombinator.com/item?id=44389202
10. Gemma 3n fully available in the open-source ecosystem!, accessed July 16, 2025, https://huggingface.co/blog/gemma3n
11. Introducing Gemma 3n: The developer guide - Google Developers Blog, accessed July 16, 2025, https://developers.googleblog.com/en/introducing-gemma-3n-developer-guide/
12. Google Gemma-3n : Best Multi-modal LLM for Mobile, Edge AI | by Mehul Gupta | Data Science in Your Pocket | Jul, 2025 | Medium, accessed July 16, 2025, https://medium.com/data-science-in-your-pocket/google-gemma-3n-best-multi-modal-llm-for-mobile-edge-ai-b373e74e327c
13. Meet Gemma 3n: Google's lightweight AI model that works offline with just 2GB RAM, accessed July 16, 2025, https://m.economictimes.com/magazines/panache/meet-gemma-3n-googles-lightweight-ai-model-that-works-offline-with-just-2gb-ram/articleshow/122114583.cms
14. ai.google.dev, accessed July 16, 2025, [https://ai.google.dev/gemma/docs/gemma-3n?hl=ko#:~:text=Gemma%203n%20%EB%AA%A8%EB%8D%B8%EC%9D%80%20%EB%8B%A8%EC%9D%BC,%EC%B6%94%EB%A1%A0%EC%97%90%20%EC%82%AC%EC%9A%A9%ED%95%A0%20%EC%88%98%20%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4.](https://ai.google.dev/gemma/docs/gemma-3n?hl=ko#:~:text=Gemma 3n 모델은 단일,추론에 사용할 수 있습니다.)
15. Gemma 3n 모델 개요 | Google AI for Developers, accessed July 16, 2025, https://ai.google.dev/gemma/docs/gemma-3n?hl=ko
16. Gemma3n - Hugging Face, accessed July 16, 2025, https://huggingface.co/docs/transformers/model_doc/gemma3n
17. Gemma 3n 미리보기 발표: 강력하고 효율적인 모바일 우선 AI, accessed July 16, 2025, https://developers.googleblog.com/ko/introducing-gemma-3n/
18. Introducing Gemma 3n: Google's Latest Open Model for On-Device AI - YourTechDiet, accessed July 16, 2025, https://yourtechdiet.com/blogs/introducing-gemma-3n-googles-latest-open-model-for-on-device-ai/
19. Google - The Gemma 3n Impact Challenge | Kaggle, accessed July 16, 2025, https://www.kaggle.com/competitions/google-gemma-3n-hackathon
20. Gemma 3n E2B Instructed LiteRT Preview vs Llama 3.2 3B Instruct - LLM Stats, accessed July 16, 2025, https://llm-stats.com/models/compare/gemma-3n-e2b-it-litert-preview-vs-llama-3.2-3b-instruct
21. Gemma 3n vs Gemma 3 (4B/12B) Benchmarks : r/LocalLLaMA - Reddit, accessed July 16, 2025, https://www.reddit.com/r/LocalLLaMA/comments/1ll88pe/gemma_3n_vs_gemma_3_4b12b_benchmarks/
22. Gemma 3: Inside Google's Revolutionary Open AI Model - SentiSight.ai, accessed July 16, 2025, https://www.sentisight.ai/gemma-3-introducing-googles-latest-open-ai-model/
23. Google's Gemma 3: Features, Benchmarks, Performance and Implementation - Analytics Vidhya, accessed July 16, 2025, https://www.analyticsvidhya.com/blog/2025/03/gemma-3/
24. Gemma 3: Open Multimodal AI with Increased Context Window | by My Social - Medium, accessed July 16, 2025, https://medium.com/aimonks/gemma-3-open-multimodal-ai-with-increased-context-window-4a3b3cdf3f25
25. 한국어 잘하는 로컬 LLM으로 RAG 성능 평가 - Gemma2, Qwen2 (구독자 이벤트 있음: ~ 2024년 7월 10일), accessed July 16, 2025, https://www.youtube.com/watch?v=jSd6pdydBNU
26. Gemma3가 파인 튜닝/세상 지식에서 엄청나게 많은 모델들보다 성능이 좋네 : r/LocalLLaMA, accessed July 16, 2025, https://www.reddit.com/r/LocalLLaMA/comments/1jhl6jp/gemma3_is_outperforming_a_ton_of_models_on/?tl=ko
27. Open Ko-LLM Leaderboard2: Bridging Foundational and Practical Evaluation for Korean LLMs - arXiv, accessed July 16, 2025, https://arxiv.org/html/2410.12445v3
28. Open Ko-LLM Leaderboard: Evaluating Large Language Models in Korean with Ko-H5 Benchmark - arXiv, accessed July 16, 2025, https://arxiv.org/html/2405.20574v2
29. Open Ko-LLM Leaderboard: Evaluating Large Language Models in Korean with Ko-H5 Benchmark - ACL Anthology, accessed July 16, 2025, https://aclanthology.org/2024.acl-long.177/
30. 구글의 온디바이스용 멀티모달 모델 Gemma 3n e4b. 성능테스트 - YouTube, accessed July 16, 2025, https://www.youtube.com/watch?v=WErzgwkOXtk
31. Ko LLM Leaderboard best models ❤️‍ - Hugging Face, accessed July 16, 2025, https://huggingface.co/collections/open-ko-llm-leaderboard/ko-llm-leaderboard-best-models-659c7e45a481ceea4c883506
32. Introducing Gemma 3: The most capable model you can run on a single GPU or TPU, accessed July 16, 2025, https://blog.google/technology/developers/gemma-3/
33. Google Gemma 3n is What Apple Intelligence Wants to Be - Analytics India Magazine, accessed July 16, 2025, https://analyticsindiamag.com/global-tech/google-gemma-3n-is-what-apple-intelligence-wants-to-be/
34. Smollm3: Smol, multilingual, long-context reasoner LLM | Hacker News, accessed July 16, 2025, https://news.ycombinator.com/item?id=44501413
35. Compare Gemma 3n vs. Phi-4 in 2025 - Slashdot, accessed July 16, 2025, https://slashdot.org/software/comparison/Gemma-3n-vs-Phi-4/
36. Gemma 3n E2B Instructed LiteRT Preview vs Phi 4 Mini Reasoning, accessed July 16, 2025, https://llm-stats.com/models/compare/gemma-3n-e2b-it-litert-preview-vs-phi-4-mini-reasoning
37. Responsible Generative AI Toolkit | Google AI for Developers - Gemini API, accessed July 16, 2025, https://ai.google.dev/responsible/docs
38. Responsible Generative AI Toolkit | Google AI for Developers - Gemini API, accessed July 16, 2025, https://ai.google.dev/responsible
39. Gemma - Google DeepMind, accessed July 16, 2025, https://deepmind.google/models/gemma/


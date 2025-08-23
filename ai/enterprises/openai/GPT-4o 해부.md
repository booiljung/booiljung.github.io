# GPT-4o 해부



2024년 5월에 발표된 GPT-4o는 OpenAI의 플래그십 모델로, 단순한 반복 업데이트가 아닌 새로운 범주의 인공지능(AI)으로 자리매김했다.1 모델명의 'o'는 'omni'(모든, 전부를 의미하는 라틴어*omnis*에서 유래)를 의미하며, 이는 모든 데이터 유형에 걸쳐 보편성과 매끄러운 통합을 지향하는 설계 철학을 명확히 보여준다.1

GPT-4o의 핵심 역량은 텍스트, 오디오, 이미지, 비디오의 모든 조합을 입력으로 받아들이고, 텍스트, 오디오, 이미지의 모든 조합을 출력으로 생성하는 능력에 있다.3 이는 '옴니' 아키텍처가 약속하는 근본적인 기능이다. 이러한 기술적 진보는 GPT-4 수준의 지능을 무료 사용자에게까지 제공하고, 더 자연스러운 인간-컴퓨터 상호작용을 구현하려는 OpenAI의 전략적 목표와 맞닿아 있다.2 OpenAI의 CEO 샘 알트먼(Sam Altman)은 새로운 음성 및 비디오 모드를 "지금까지 사용해 본 최고의 컴퓨터 인터페이스"이자 "영화에서나 보던 AI가 현실이 된 것 같다"고 표현하며, GPT-4o가 가져올 아키텍처적 변화의 중요성을 시사했다.7

이러한 수사적 표현은 단순한 마케팅을 넘어선다. '옴니'라는 명칭은 아키텍처에 대한 선언과 같다. 이는 과거의 '허브 앤 스포크(hub-and-spoke)' 방식의 멀티모달, 즉 중앙의 언어 모델이 필요에 따라 음성이나 이미지 처리 같은 전문화된 외부 도구를 호출하던 방식에서 벗어나, 모든 데이터 유형을 아우르는 통합된 단일 처리 코어로의 전환을 의미한다. 이는 모델의 내부 표현(representation)이 본질적으로 특정 양식에 얽매이지 않으며(modality-agnostic), 지원되는 모든 데이터 유형 간의 유연한 변환과 생성을 가능하게 함을 시사한다. 이전 세대 모델인 GPT-4는 음성 처리를 위해 위스퍼(Whisper)를, 이미지 생성을 위해 DALL-E를 사용하는 등 아키텍처적으로 분리된 구조를 가졌었다.2 반면 GPT-4o는 "텍스트, 비전, 오디오 전반에 걸쳐 단일 모델로 종단간(end-to-end) 훈련"되었으며, 모든 입력과 출력이 "동일한 신경망에 의해 처리된다"고 명시적으로 설명된다.3 이 아키텍처적 통합은 파이프라인의 오버헤드를 제거하여 극적인 지연 시간 단축을 이끌어냈고, 음성의 톤과 같은 모달 간의 미묘한 신호를 보존하며, 노래와 같은 이전에는 불가능했던 새로운 능력을 발현시키는 직접적인 원인이 되었다.3 따라서 '옴니'는 GPT-4o의 핵심 설계 철학을 가장 정확하게 기술하는 기술적 용어이며, 덜 통합되었던 이전의 멀티모달 시스템과 근본적으로 구별되는 지점이다.


GPT-4o의 아키텍처적 중요성을 이해하기 위해서는 GPT-3.5에서 GPT-4를 거쳐 GPT-4o에 이르는 진화 과정을 살펴보는 것이 필수적이다. 각 단계는 뚜렷한 아키텍처적 변화를 동반했다. GPT-4는 텍스트와 이미지 입력을 받아 텍스트만 출력할 수 있었던 반면, GPT-4o는 기본적으로 옴니 모달 입출력을 모두 지원한다.9

GPT-4o가 이전 모델들보다 우수함을 입증하는 핵심 지표는 속도, 비용, 그리고 성능이다. GPT-4o는 GPT-4 터보(Turbo)보다 2배 더 빠르고, API 비용은 50% 저렴하며, 텍스트와 코드 처리 능력은 GPT-4 터보 수준을 유지하면서도 다국어, 오디오, 비전 벤치마크에서는 새로운 기록을 세웠다.1 이러한 성능 향상은 단순히 모델을 더 크게 만드는 것을 넘어, 근본적인 아키텍처 혁신이 있었음을 시사한다.


GPT-4o의 배포 전략은 그 복잡성과 안전에 대한 신중한 접근을 보여준다. 공식 출시 전, gpt2-chatbot과 같은 가명을 사용하여 LMSYS 챗봇 아레나(Chatbot Arena)에서 비밀리에 모델을 배포하여 사전 출시 데이터를 수집했다.1 이는 실제 사용 환경에서의 모델 성능과 잠재적 위험을 미리 파악하기 위한 전략적 조치였다.

출시 이후에도 기능은 단계적으로 공개되었다. 텍스트와 오디오 기능이 먼저 제공되었고, 고급 음성 모드와 네이티브 이미지 생성 기능은 그 이후에 순차적으로 도입되었다.1 이러한 단계적 배포는 모델의 방대한 복잡성을 관리하고, 각 기능의 안전성을 충분히 검증한 후 사용자에게 제공하려는 OpenAI의 신중한 접근 방식을 반영한다.

또한, GPT-4o 미니(mini)의 출시는 OpenAI의 전략적 비전에서 중요한 부분을 차지한다. GPT-4o 미니는 기존의 GPT-3.5를 대체하며, 더 저렴한 비용으로 강력한 AI를 더 넓은 범위의 애플리케이션에 접근 가능하게 만들었다.1 이는 AI 기술의 민주화와 생태계 확장을 목표로 하는 OpenAI의 장기적 전략의 일환으로 해석될 수 있다.

| 기능                                 | GPT-3.5 (+음성 모드)         | GPT-4 (+비전)                            | GPT-4o                         | GPT-4o 미니             |
| ------------------------------------ | ---------------------------- | ---------------------------------------- | ------------------------------ | ----------------------- |
| **아키텍처 유형**                    | 전문화된 모델들의 파이프라인 | 파이프라인 (텍스트/이미지) + 텍스트 코어 | 통합된 종단간 옴니 모달        | 압축된 종단간 옴니 모달 |
| **모달리티 (입력)**                  | 텍스트, 오디오               | 텍스트, 이미지                           | 텍스트, 오디오, 이미지, 비디오 | 텍스트, 비전            |
| **모달리티 (출력)**                  | 텍스트, 오디오               | 텍스트                                   | 텍스트, 오디오, 이미지         | 텍스트                  |
| **평균 오디오 지연 시간**            | 약 2.8초 3                   | 약 5.4초 3                               | 약 320밀리초 3                 | 해당 없음               |
| **API 비용 (백만 토큰당 입력/출력)** | $0.50/$1.50 (GPT-3.5 터보)   | $10/$30 (GPT-4 터보 128k)                | $5/$15 (GPT-4o)                | $0.15/$0.60 1           |
| **MMLU 벤치마크 점수**               | 약 70% (GPT-3.5)             | 86.5% 1                                  | 88.7% 1                        | GPT-3.5 터보 상회 15    |
| **핵심 아키텍처 혁신**               | 지시 따르기 (RLHF)           | 멀티모달 입력/MoE                        | 종단간 옴니 모달리티           | 비용 효율적 압축        |



GPT-4o의 혁신을 이해하기 위해서는 이전 세대 모델들이 멀티모달 상호작용을 처리하던 분절된 파이프라인 방식을 먼저 분석해야 한다. 특히 음성 상호작용의 경우, 이는 명확한 3단계 과정을 거쳤다.

1. **음성-텍스트 변환(Speech-to-Text):** 위스퍼(Whisper)와 같은 음성 인식 모델이 사용자의 오디오 입력을 텍스트로 변환한다.2
2. **텍스트 처리:** GPT-3.5나 GPT-4와 같은 핵심 언어 모델이 변환된 텍스트를 입력받아 처리하고, 텍스트 응답을 생성한다.2
3. **텍스트-음성 변환(Text-to-Speech):** 별도의 TTS 모델이 언어 모델이 생성한 텍스트 응답을 다시 오디오로 변환하여 사용자에게 들려준다.2

이 설계는 본질적인 비효율성과 한계를 내포하고 있었다. 가장 큰 문제는 이 세 개의 독립된 모델 간의 통신 오버헤드에서 발생하는 지연 시간이었다. 각 단계를 거치면서 발생하는 작은 지연들이 누적되어, GPT-3.5에서는 평균 2.8초, GPT-4에서는 무려 5.4초에 달하는 부자연스러운 대화 지연을 초래했다.3

더욱 중요한 것은 이 파이프라인 구조가 야기하는 '정보 손실'이다. 핵심 지능을 담당하는 GPT-4는 실제 오디오를 '들은' 것이 아니라, 단지 텍스트로 변환된 '기록'을 읽었을 뿐이다. 이는 음성의 톤, 감정, 배경 소음, 또는 여러 화자의 존재와 같은 중요한 비언어적 정보가 첫 번째 변환 단계에서 완전히 소실됨을 의미한다.3 결과적으로 모델은 "무엇을 말했는지"는 알 수 있지만 "어떻게 말했는지"는 전혀 알 수 없었고, 이는 진정한 의미의 자연스러운 상호작용을 가로막는 근본적인 한계였다.


GPT-4o는 이러한 파이프라인의 한계를 단일 종단간(end-to-end) 훈련 신경망이라는 혁신적인 아키텍처로 해결했다. 이제 텍스트, 비전, 오디오 등 모든 모달리티가 동일한 신경망 내에서 처리된다.3

이러한 아키텍처의 핵심은 모델이 모든 모달리티를 위한 공유된 통합 표현 공간(unified representation space)을 학습한다는 점이다. 즉, 오디오를 텍스트로 변환하는 대신, 오디오 신호 자체를 이 공유 공간으로 직접 매핑하여 텍스트나 이미지 표현과 함께 추론할 수 있게 된다. 이 통합된 처리 방식은 지연 시간을 인간의 대화 반응 시간과 유사한 평균 320밀리초까지 극적으로 단축시키는 직접적인 원인이 되었다.3

이러한 아키텍처적 전환은 단순한 성능 최적화를 넘어, 모델의 인지 방식에 근본적인 도약을 가져왔다. 이는 '번역'에서 '인식'으로의 전환으로 비유할 수 있다. 이전 모델은 모든 외부 모달리티를 텍스트라는 공용어(lingua franca)로 '번역'하여 자신의 언어 모델 두뇌가 이해할 수 있도록 했다. 반면 GPT-4o는 모든 모달리티를 더 풍부하고 통합된 내부 '언어'로 직접 '인식'한다. 어떤 번역 과정에서든 정보 손실은 필연적이다. 원본 언어(이 경우, 오디오 파형이라는 '언어')의 뉘앙스 중 목표 언어(텍스트)에 직접적인 등가물이 없는 것들은 버려진다.3 GPT-4o의 통합 신경망은 원본 모달리티의 원시 데이터를 직접 처리함으로써 이러한 정보 손실을 막는다.3 이는 인간의 뇌가 감각 입력을 처리하는 방식과 유사하다. 인간은 시각 정보를 이해하기 전에 그것을 내면의 독백으로 번역하지 않고, 직접 인식한다. 따라서 모델의 '이해'는 이제 손실이 많은 텍스트 추상화가 아닌, 원본 모달리티의 풍부하고 연속적인 데이터에 기반하게 된다. 이것이 바로 감정을 감지하거나 노래를 부르는 것과 같이, 변환된 텍스트가 아닌 오디오 신호 자체의 속성인 새로운 능력들이 출현한 이유이다. 이는 상징적이고 텍스트 중심적인 지능에서 보다 총체적이고 인식 기반의 지능으로의 전환을 의미한다.


종단간 아키텍처와 정보 손실의 제거는 이전에는 불가능했던 새로운 능력들을 발현시켰다.

- **감정 및 음성 톤의 뉘앙스:** 모델이 이제 원시 오디오를 직접 '관찰'할 수 있게 되면서, 감정적 단서와 톤을 인식할 수 있게 되었다. 이를 통해 웃음, 노래, 심지어 풍자적인 어조와 같은 감정이 담긴 응답을 생성하는 것이 가능해졌다.2
- **실시간 상호작용:** 극적으로 줄어든 지연 시간 덕분에, OpenAI가 시연한 것처럼 서로 다른 언어를 사용하는 두 화자 간의 실시간 동시 통역과 같은 새로운 애플리케이션이 실용화될 수 있게 되었다.1
- **통합된 비전과 대화:** 모델은 카메라의 실시간 비디오 피드를 분석하고, 보이는 것에 대해 실시간으로 대화할 수 있다. 이는 시각적 이해를 음성 대화에 매끄럽게 통합하는 것으로, 이전에는 여러 단계를 거쳐야 했던 번거로운 작업이었다.8

| 아키텍처 측면           | 파이프라인 모델 (예: GPT-4 + DALL-E/Whisper)                 | 통합된 종단간 모델 (GPT-4o)                                  |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **정보 흐름**           | API를 통한 순차적, 다단계 처리                               | 단일 신경망 내에서의 통합된 병렬 처리                        |
| **지연 시간 원인**      | 모델 간 네트워크/API 호출, 순차적 계산                       | 최소한의 지연; 주로 단일 네트워크의 계산 시간                |
| **모달 간 신호 무결성** | 높은 손실 (예: 톤, 감정, 배경 소음 소실)                     | 높은 충실도; 원시 모달 정보 보존                             |
| **핵심 추론 모달리티**  | 텍스트 중심; 다른 모달리티는 주변 장치                       | 모달리티에 구애받지 않음; 모든 모달리티가 동등               |
| **능력의 한계**         | 정보 손실로 인해 제한됨; 네이티브 모달 간 이해가 필요한 작업(예: 노래) 불가 | 더 높은 한계; 총체적 인식을 기반으로 한 새로운 능력 발현 가능 |
| **훈련 복잡성**         | 여러 독립적인 모델의 훈련/유지보수 필요                      | 방대하고 통합된 데이터셋으로 단일의 더 복잡한 모델 훈련 필요 |



모델의 파라미터 수가 수조 개에 달하면서, 모든 파라미터를 모든 계산에 사용하는 전통적인 '밀집(dense)' 아키텍처는 훈련과 추론에 있어 계산적으로 엄청난 비용을 요구하게 되었다.16 전문가 혼합(Mixture of Experts, MoE) 아키텍처는 이 문제에 대한 가장 유력한 해결책으로 등장했다. MoE의 핵심 아이디어는 '조건부 계산(conditional computation)'으로, 주어진 입력 토큰에 대해 모델의 전체 파라미터 중 일부의 관련된 부분 집합만 활성화하는 것이다.16 이를 통해 추론 당 계산 비용을 비례적으로 증가시키지 않으면서도 모델의 총 용량(지식)을 대폭 확장할 수 있다.


GPT-4o의 기반으로 추정되는 GPT-4의 MoE 아키텍처에 대한 가장 신뢰성 있고 널리 인용되는 추측은 다음과 같다.4

- **총 파라미터:** 약 1조 7600억에서 1조 8000억 개로 추정된다.4
- **전문가 구성:** 모델은 약 2200억 개의 파라미터를 가진 8개의 전문가 모델 앙상블 또는 약 1100억~1110억 개의 파라미터를 가진 16개의 전문가 모델 앙상블로 구성된 것으로 보인다.4 일부 정보원은 16개 전문가 구성이 더 유력하다고 제시한다.4
- **공유 파라미터:** 전문가 모델 외에도 어텐션(attention) 메커니즘과 같은 기능을 위해 약 500억~550억 개의 파라미터가 공유되는 것으로 추정된다.4
- **게이팅 네트워크/라우터:** 각 입력 토큰을 분석하여 어떤 전문가가 처리하기에 가장 적합한지 결정하는 더 작은 규모의 신경망이다.16
- **추론 시 활성화:** 주어진 토큰에 대해 라우터는 소수의 전문가(일반적으로 상위 2개)만 선택하여 활성화한다. 이는 추론 패스당 전체 1조 8000억 개가 아닌 약 2800억~3000억 개의 파라미터만 사용됨을 의미한다.4


전문가 모델의 기능적 목적은 전문화에 있다. 훈련 과정에서 각기 다른 전문가들이 특정 유형의 데이터나 작업(예: 코딩, 창의적 글쓰기, 과학적 추론, 안전성)에 대해 전문성을 발전시키는 것으로 추정된다.4 이러한 전문화는 복잡한 쿼리를 네트워크의 가장 지식이 풍부한 부분으로 라우팅하여 모델의 전반적인 능력을 향상시킨다.4 이는 또한 성능 변동의 원인이 될 수 있으며, 백엔드에서의 배치(batching) 처리에 따라 라우팅이 영향을 받을 수 있기 때문에 비결정성의 잠재적 원인이 되기도 한다.25 GPT-4가 MoE인지에 대한 논쟁도 존재하지만, 샘 알트먼이 부인한 "100조 파라미터" 루머는 더 신빙성 있는 "1.8조, 8개 전문가" 루머와는 구별되어야 한다.27

MoE 아키텍처는 본래 언어 모델의 확장을 위해 채택되었지만, 진정한 옴니 모달리티를 위한 자연스럽고 강력한 프레임워크를 제공한다. '전문가'라는 개념은 작업 전문화(코딩 대 글쓰기)에서 *모달리티 전문화*로 확장될 수 있다. GPT-4o의 MoE 구조 내에서 일부 전문가는 오디오 신호 처리에, 다른 전문가는 시각적 특징 처리에, 또 다른 전문가는 언어적 패턴 처리에 특화되어 있을 가능성이 매우 높다. 이 경우 라우터는 토큰을 원본 모달리티에 따라 라우팅하게 된다. 이는 언어 모델의 MoE가 단순히 주제별 전문화에 그치지 않고, GPT-4o와 같은 옴니 모달 모델의 핵심 작동 원리를 설명하는 강력한 가설이 된다. 실제로 최근 학계 연구는 이러한 아이디어를 뒷받침한다. '모달리티 인식 전문가 혼합(MoMa)'이라는 개념은 전문가 모듈을 모달리티별 그룹(예: 텍스트 전문가 4개, 이미지 전문가 4개)으로 나누는 것을 제안하며 28, 다른 연구들은 MoE를 사용하여 '모달리티별 경로'를 구축하는 것을 논의한다.29 이는 GPT-4o가 어떻게 통합된 처리를 달성하는지에 대한 구체적인 아키텍처 가설을 제공한다. 라우터 네트워크는 단순히 의미적 내용을 분석하는 것을 넘어, 입력 토큰의 모달리티를 식별하고 해당 데이터 유형에 최적화된 하드웨어/소프트웨어 경로로 라우팅하는 것이다. 이는 각 모달리티에 대해 별도의 거대한 인코더를 두는 것보다 훨씬 우아한 해결책이다.

| 아키텍처 구성 요소             | 추정 세부 사항                                               | 추정 근거/출처 |
| ------------------------------ | ------------------------------------------------------------ | -------------- |
| **총 파라미터**                | 약 1.76조 - 1.8조                                            | 4              |
| **전문가 수**                  | 8개 또는 16개                                                | 4              |
| **전문가당 파라미터**          | 약 1110억 (16개 전문가) 또는 약 2200억 (8개 전문가)          | 17             |
| **공유 파라미터 (예: 어텐션)** | 약 500억 - 550억                                             | 4              |
| **추론당 활성 파라미터**       | 약 2800억 - 3000억                                           | 4              |
| **게이팅 메커니즘**            | 라우터 네트워크가 각 토큰에 대해 상위 2개 전문가 선택        | 19             |
| **전문가 전문화**              | 작업별(코딩, 안전) 및/또는 모달리티별(텍스트, 비전, 오디오)로 추정 | 19             |



이전 ChatGPT에서 이미지 생성은 언어 모델이 DALL-E 3와 같은 외부 모델에 API를 호출하는 방식으로 이루어졌다.1 이 접근법의 한계는 명확했다. 언어 모델 자체는 이미지를 '보거나' '만들지' 못하고, 단지 다른 시스템을 위한 텍스트 프롬프트를 생성할 뿐이었다. 이는 언어 모델의 방대한 세계 지식 및 추론 능력과 최종 시각적 결과물 사이에 단절을 초래했다.30 또한 대화를 통한 반복적인 이미지 수정 과정을 번거롭게 만들었다.


GPT-4o의 네이티브 이미지 생성 메커니즘으로 유력하게 거론되는 것이 바로 '트랜스퓨전(Transfusion)' 아키텍처다.30 이 새로운 접근법의 핵심은 표준 언어 모델링 손실(다음 토큰 예측을 위한 교차 엔트로피)과 디퓨전 모델 손실(노이즈 제거를 위한 L2 오차)을 결합한 하이브리드 목적 함수로 단일 트랜스포머 모델을 훈련시키는 것이다.30

**핵심 구성 요소:**

1. **통합된 시퀀스:** 텍스트와 이미지 데이터가 단일의 연속적인 시퀀스로 표현된다. ``(Begin-Of-Image)나 [EOI](End-Of-Image)와 같은 특수 토큰이 모달리티 간의 경계를 표시한다.30
2. **연속적인 이미지 표현:** 이미지를 이산적인 토큰으로 양자화(quantize)하는 대신(이는 정보 손실을 유발함), 트랜스퓨전은 이미지를 연속적인 값을 갖는 잠재 패치(latent patches)의 시퀀스로 표현한다.30
3. **디퓨전 목적 함수:** 훈련 중에 이 잠재 이미지 패치에 노이즈가 추가된다. 트랜스포머의 임무는 선행하는 텍스트 컨텍스트의 안내를 받아 이 패치들의 *노이즈가 제거된* 버전을 예측하는 것이다. 이는 디퓨전 과정을 자기회귀적(autoregressive) 시퀀스 생성 과정에 직접 내장하는 효과를 낳는다.30


추론 과정은 다음과 같이 단계별로 설명될 수 있다.

1. 모델은 자기회귀적으로 텍스트 토큰을 생성한다.
2. 이미지를 생성하기로 결정하면, 특수 토큰인 ``를 생성한다.30
3. 그 다음, 순수한 랜덤 노이즈로 초기화된 플레이스홀더 토큰 블록을 시퀀스에 추가한다.
4. 모델은 이제 디퓨전 디코딩 루프에 진입하여, 전체 시퀀스(텍스트 컨텍스트 + 노이즈가 낀 이미지 잠재 벡터)를 트랜스포머에 반복적으로 통과시켜 이미지 패치의 노이즈를 점진적으로 제거한다. 이는 사실상 이미지를 '제자리에서' 생성하는 과정이다.30
5. 노이즈 제거가 완료되면, [EOI] 토큰을 생성하고 다시 텍스트 생성을 재개할 수 있다.


이 통합된 접근 방식은 GPT-4o의 뛰어난 이미지 생성 능력으로 직접 이어진다.

- **향상된 지시 따르기 및 텍스트 렌더링:** 언어를 이해하는 동일한 트랜스포머가 이미지를 생성하기 때문에 프롬프트와 결과물 간의 결합이 훨씬 긴밀해진다. 이것이 GPT-4o가 이미지 내에 텍스트를 정확하게 렌더링하고, 10~20개의 서로 다른 객체가 있는 복잡한 장면을 처리하는 데 뛰어난 이유이다.14
- **인-컨텍스트 학습 및 편집:** 이미지 생성이 이제 채팅 컨텍스트의 일부가 된다. 사용자는 참조 이미지를 업로드하거나 생성된 이미지를 자연스러운 대화를 통해 수정할 수 있으며, 모델은 일관성을 유지한다.14
- **매끄러운 멀티모달 출력:** 모델은 단일의 일관된 응답 내에 텍스트와 이미지를 혼합하여 생성할 수 있는데, 이는 파이프라인 아키텍처로는 불가능한 위업이다.30

트랜스퓨전 아키텍처는 생성형 AI의 두 가지 지배적인 패러다임, 즉 구조화된 순차 데이터(언어 등)를 위한 자기회귀 트랜스포머와 비구조화된 연속 데이터(이미지 등)를 위한 디퓨전 모델의 역사적인 통합을 의미한다. 디퓨전 목적 함수를 트랜스포머의 다음 토큰 예측 프레임워크 내에 내장함으로써, 두 '언어'를 모두 유창하게 구사할 수 있는 단일 모델을 만들어낸다. 이는 진정으로 통합된 생성형 기반 모델을 향한 길을 열어준다. 과거 생성형 AI 분야는 언어에 뛰어난 언어 모델(GPT 시리즈)과 이미지에 뛰어난 디퓨전 모델(DALL-E, Midjourney)로 양분되어 있었다.35 이들을 연결하려는 시도는 종종 한 모델이 다른 모델에게 무엇을 할지 '지시'하는 형태(예: GPT-4가 DALL-E를 위한 프롬프트를 생성)였으며, 이는 파트너십이 아닌 주종 관계에 가까웠다.30 다른 접근법은 이미지를 이산적인 토큰으로 양자화하여 언어 모델의 세계에 억지로 끼워 맞추려 했지만, 이는 정보 손실이 크고 확장성도 좋지 않았다.31 트랜스퓨전 아키텍처는 이산적인 데이터(텍스트)와 연속적인 데이터(이미지)를 모두 네이티브하게 처리하도록 트랜스포머를 가르치는 더 우아한 해결책을 제시한다.30 이는 동일한 훈련 루프와 동일한 모델 파라미터 내에서 두 가지 다른 손실 함수(텍스트용 교차 엔트로피, 이미지 디퓨전용 L2)를 사용하여 달성된다.30 이는 모델이 "'고양이'라는 단어"의 개념과 "고양이의 시각적 특징"이라는 개념이 깊이 얽혀 있는 공유된 잠재 공간을 학습한다는 것을 의미한다. 그 함의는 심오하다. 이는 단지 이미지를 더 잘 생성하는 방법을 넘어, 어색한 번역 계층 없이 모든 모달리티에 걸쳐 정보를 이해하고 생성할 수 있는 단일의 범용 모델을 향한 기초적인 단계이며, 이는 바로 인공 일반 지능(AGI)의 핵심 약속이다.



옴니 모달 모델을 훈련시키기 위해서는 방대하고 다양한 데이터셋이 필수적이다. 초기 릴리스의 지식 컷오프가 2023년 10월인 이 데이터는 공개적으로 이용 가능한 데이터, 제3자로부터 라이선스를 받은 데이터, 그리고 웹 크롤링 데이터를 혼합하여 구성되었다.5

훈련 코퍼스는 텍스트와 코드뿐만 아니라, 모델이 비텍스트 데이터를 해석하고 생성하는 방법을 학습할 수 있도록 이미지, 오디오, 비디오를 명시적으로 포함한다.5 특히 코드와 수학 데이터의 포함은 강력한 추론 능력을 개발하는 데 결정적인 역할을 하는 것으로 언급된다.5 여기서 주목할 만한 혁신 중 하나는 새로운 토크나이저(tokenizer)로, 구자라트어와 같은 비영어권 언어에서 토큰 수를 최대 4.4배까지 줄여 효율성을 크게 향상시켰다.3


사전 훈련이 모델에 원초적인 능력을 부여한다면, 사후 훈련은 모델을 안전하고 유용하게 만드는 과정이다. 이 과정은 지도 미세 조정(Supervised Fine-Tuning, SFT)과 인간 피드백 기반 강화 학습(Reinforcement Learning from Human Feedback, RLHF)을 포함한다.9

멀티모달 모델에 RLHF를 적용하는 것은 독특하고 중대한 도전 과제들을 수반한다.

- **주석의 모호성:** 복잡한 멀티모달 응답을 평가하는 것은 텍스트만 평가하는 것보다 인간에게 훨씬 더 어렵다. 응답의 텍스트는 정확하지만 이미지는 환각(hallucination)이거나, 톤은 도움이 되지만 사실이 틀리는 등, 단순한 선호도 순위 매기기를 어렵게 만든다.36
- **보상 할당 문제:** 길고 복합적인 미디어 응답에 대해 보상 신호가 주어졌을 때, 모델이 긍정적 피드백의 원인이 텍스트, 이미지, 톤 중 어느 부분이었는지 정확히 알기 어렵다. 이는 '보상 해킹(reward hacking)'으로 이어질 수 있다.36
- **멀티모달 환각:** 모델이 연관된 이미지에 사실적으로 근거하지 않은 텍스트를 생성하는 문제는 최첨단 모델에서도 널리 나타나는 현상이다.36

이러한 문제들을 해결하기 위해 세분화된 교정 피드백(RLHF-V)이나 사실 기반 증강 RLHF와 같이 모델에 더 표적화된 신호를 제공하는 새로운 기술들이 연구되고 있다.36


OpenAI는 GPT-4o 시스템 카드(System Card)에 명시된 바와 같이 광범위한 안전성 평가 프로세스를 수행했다.5

- **외부 레드팀(Red Teaming):** OpenAI는 45개 언어를 구사하는 다양한 분야의 외부 전문가 100명 이상을 참여시켜 허위 정보, 편향, 사칭, 비허용 콘텐츠 생성과 같은 영역의 위험을 사전에 식별했다.5
- **내장된 안전장치:** 모델은 사전 훈련 데이터 필터링 및 사후 훈련을 통한 모델 행동 개선과 같은 기술을 통해 "설계 단계부터" 안전 기능이 내장되어 있다.3 예를 들어, GPT-4o 미니 모델은 탈옥(jailbreak) 및 프롬프트 주입(prompt injection)에 더 잘 저항하기 위해 '지시 계층(instruction hierarchy)' 방법을 적용한 최초의 모델이다.15
- **모달리티별 위험:** 이 보고서는 무단 음성 생성 및 화자 식별과 같이 오디오 모달리티가 야기하는 새로운 위험과 이에 대한 완화 조치를 다룬다. 평가 결과, 음성 모달리티는 설득과 같은 '준비성(Preparedness)' 위험을 유의미하게 증가시키지 않는 것으로 결론 내려졌다.5

GPT-4o의 아키텍처는 톤이나 감정과 같은 인간적인 뉘앙스를 포착함으로써 새롭고 더 어려운 정렬 문제를 야기한다. 이는 '정렬의 역설'이라 할 수 있다. 사실적 정확성(텍스트 기반 작업)에 대해 모델을 정렬하는 것은 비교적 간단하다. 하지만 '적절한 감정적 반응'에 대해 정렬하는 것은 주관적이고 문화에 따라 다르며 훨씬 더 복잡하다. 모델을 더 자연스럽게 느끼게 만드는 바로 그 아키텍처적 특징이, 보편적인 인간 가치 체계와 정렬하는 것을 더 어렵게 만드는 것이다. GPT-4o의 종단간 아키텍처는 감정적, 음성적 단서를 인식하고 생성할 수 있게 하여 상호작용을 더 '인간적'으로 만든다.3 RLHF는 인간 주석가의 선호도 피드백에 의존하는데 12, 텍스트의 '좋음'은 사실성이나 유용성과 같은 객관적 척도로 정의될 수 있다. 그러나 멀티모달 출력에서 '좋음'은 훨씬 주관적이 된다. 풍자적인 톤이 '좋은' 것인가 '나쁜' 것인가? 이는 전적으로 맥락, 사용자 성격, 문화적 규범에 달려 있다. 공감적인 반응이 도움이 되는가, 아니면 조종적인가? RLHF에서 인간 피드백의 문제점들-주관성, 편향, 비일관성 36-은 텍스트에서 미묘한 멀티모달 행동으로 이동하면서 기하급수적으로 증폭된다. 따라서 더 '인간적인' 모델을 만드는 아키텍처적 성공은 직접적으로 '인간 수준의' 정렬 문제를 야기한다. 공학적 해결책(통합 모델)이 새로운, 더 어려운 사회과학적 문제(멀티모달 선호도를 정의하고 확장하는 것)를 만들어낸 것이다. 이는 미래의 발전이 컴퓨터 과학만큼이나 사회과학과 윤리학의 진보에 달려있음을 시사한다.



2024년 7월에 소개된 GPT-4o 미니는 전략적 모델 압축의 대표적인 예시이다.1 이 모델은 ChatGPT 인터페이스에서 GPT-3.5 터보를 대체하며, 훨씬 저렴한 비용으로 우수한 지능을 제공했다.1 GPT-4o 미니는 학술 벤치마크에서 GPT-3.5 터보를 능가하며 60% 이상 저렴하다. API 비용은 백만 입력 토큰당 0.15달러, 백만 출력 토큰당 0.60달러에 불과하다.1 이는 2022년의

text-davinci-003 모델과 비교했을 때 99%의 비용 절감을 의미한다.15


GPT-4o 미니가 더 큰 GPT-4o 기반 모델로부터 어떻게 만들어졌는지에 대한 OpenAI의 공식적인 설명은 없지만, 업계 표준 기술들을 통해 그 방법을 유추할 수 있다.

- **양자화(Quantization):** 모델의 가중치와 활성화 값의 정밀도를 낮추는 기술이다(예: 16비트 부동소수점에서 8비트 정수로). 이는 모델 크기와 메모리 요구량을 극적으로 줄여, 더 빠르고 저렴하게 실행할 수 있게 한다.42
- **가지치기(Pruning):** 신경망에서 중복되거나 덜 중요한 연결(파라미터)을 제거하는 기술이다. 개별 가중치를 제거하는 비구조적 방식과 전체 뉴런이나 어텐션 헤드를 제거하는 구조적 방식이 있으며, 이를 통해 더 작고 희소한 모델을 만든다.42
- **지식 증류(Knowledge Distillation) (추정):** 더 작은 '학생' 모델(GPT-4o 미니)이 더 큰 '교사' 모델(GPT-4o)의 출력을 모방하도록 훈련시켜, 지식을 더 압축된 형태로 효과적으로 전달하는 과정이다.43


GPT-4o, GPT-4o 미니, 그리고 기업용 미세 조정(fine-tuning)과 같은 모델 제품군을 제공하는 것은 비즈니스 및 생태계 측면에서 중요한 의미를 가진다.

- **접근성 민주화:** GPT-4o 미니의 저렴한 비용은 스타트업과 개발자들이 이전에는 비용 문제로 시도하기 어려웠던, 특히 대량의 API 호출이 필요한 애플리케이션을 구축할 수 있게 한다.1
- **기업 맞춤화:** 기업 고객이 GPT-4o를 자체 데이터로 미세 조정할 수 있게 함으로써, 고객 서비스나 특정 산업 지식과 같은 영역에서 깊이 있는 전문화를 가능하게 하여 기업 환경에서의 활용도와 채택률을 높인다.1
- **경쟁적 포지셔닝:** 무료이면서 강력한 모델(ChatGPT의 GPT-4o)부터 저렴하고 빠른 모델(GPT-4o 미니 API), 그리고 맞춤형 기업용 모델에 이르기까지 다양한 스펙트럼의 모델을 제공함으로써, OpenAI는 개인 사용자부터 대기업에 이르는 모든 시장 부문에서 경쟁할 수 있게 된다.

GPT-4o와 GPT-4o 미니의 출시 패턴은 OpenAI가 성숙하고 반복 가능한 "서비스형 프론티어 모델(Frontier Model as a Service, FMaaS)" 파이프라인을 구축했음을 보여준다. OpenAI의 전략은 더 이상 단일의 가장 큰 모델을 만드는 것에만 국한되지 않는다. 이제는 막대한 비용을 들여 거대한 '교사' 모델(프론티어 모델)을 구축한 다음, 고급 압축 및 증류 기술을 사용하여 비용-성능 곡선의 여러 지점에 최적화된 더 작고 전문화된 '학생' 모델 제품군을 생성하는 것이다. 프론티어 모델이 R&D 투자라면, 압축된 모델 제품군은 실제 판매되는 제품 라인이다. OpenAI는 막대한 비용을 들여 최첨단 프론티어 모델(GPT-4, 현재 GPT-4o)을 개발한다.12 이 모델은 초기에 프리미엄 제품으로 제공된다(예: API를 통한 GPT-4, 플러스 사용자를 위한 GPT-4o).1 그 직후, 이전 세대의 최고 모델(GPT-3.5 터보)보다 훨씬 뛰어난 성능을 보이면서도 훨씬 저렴하고 작은 버전(GPT-4o 미니)이 출시된다.15 이는 대형 모델의 능력을 소형 모델로 '증류'하는 과정이 있음을 시사하며, 이를 위한 기술은 양자화, 가지치기, 지식 증류 등으로 잘 알려져 있다.42 이를 통해 OpenAI는 무료/프리미엄 계층(성능 과시 및 데이터 수집용), 대량/개발자 계층(채택 및 API 사용량 증대용), 그리고 기업 계층(고수익 맞춤 솔루션용)으로 구성된 제품 포트폴리오를 완성한다.1 이는 프론티어 모델의 막대한 R&D 비용을 무료 사용자부터 고액의 기업 고객까지 전체 시장을 아우르는 다양한 제품 라인에 걸쳐 상각하는, 매우 확장 가능하고 반복 가능한 비즈니스 전략이다. 이것이 바로 AI 모델 개발의 산업화 단계이다.



GPT-4o의 아키텍처가 실제 세계에 미치는 영향을 보여주는 정량적 성능 지표는 다음과 같다.

- **일반 지능:** MMLU 벤치마크에서 88.7점이라는 새로운 최고 점수를 기록하며 GPT-4의 86.5점을 넘어섰다.1
- **다국어 및 비전:** 오디오 음성 인식, 번역, 비전 이해 벤치마크에서 새로운 기록을 세웠다.1
- **코딩:** 코딩 작업에서 GPT-4 터보 수준의 성능을 보이며 3, 일부 비교에서는 실용적이고 잘 구조화된 코드를 생성하는 데 탁월한 능력을 보였다.46
- **추론:** 추론 작업에서 강력한 성능을 보이며, 경쟁 모델에 비해 종종 가장 우수하고 상세한 설명을 제공했다.45


벤치마크로 측정하기는 어렵지만 사용자 경험에 결정적인 영향을 미치는 정성적 능력은 다음과 같다.

- **자연스러운 대화:** 대화 중 끼어들기를 처리하고, 인간과 유사한 지연 시간으로 응답하며, 감정을 인식하는 능력은 대화를 유연하고 자연스럽게 만든다.2
- **창의적 협업:** 모델은 창의적인 파트너 역할을 하여, 단일 대화 내에서 아이디어, 텍스트, 이미지를 매끄럽게 생성하고 다듬을 수 있다.14
- **인-컨텍스트 시각적 이해:** 사용자가 업로드한 이미지, 다이어그램, 스크린샷을 분석하고 이를 문제 해결이나 생성의 맥락으로 사용할 수 있다. 예를 들어, 사진 속 수학 문제를 학생이 푸는 것을 돕는 것이 가능하다.8


진보된 아키텍처에도 불구하고 여전히 남아있는 한계와 과제는 다음과 같다.

- **환각 및 사실적 비신뢰성:** 모든 LLM과 마찬가지로 GPT-4o도 '환각'을 일으키거나 사실과 다른 정보를 생성할 수 있다. 정렬 노력을 통해 이를 줄이려 하지만 위험은 여전히 존재한다.10
- **편향 및 표현의 해악:** 모델은 방대한 훈련 데이터에 존재하는 사회적 편향을 영속시킬 수 있다. OpenAI는 이를 인지하고 필터링과 사후 훈련을 통해 완화하려 하지만, 위험은 남아있다.5
- **비결정성:** 고정된 설정(온도=0)에서도 모델은 동일한 입력에 대해 약간 다른 출력을 생성할 수 있다. 이는 MoE 아키텍처와 백엔드의 배치 추론의 부산물일 가능성이 높으며, 결정론적 출력이 필요한 애플리케이션에는 불편함을 줄 수 있다.25
- **복잡성과 모호성:** 모델은 여전히 매우 복잡하거나 모호하거나 추상적인 추론 작업에서 사용자 의도를 잘못 해석하는 한계를 보인다.12

MoE 아키텍처의 사용은 계산 효율성을 위해 중요하지만, 본질적으로 출력의 결정성과 상충 관계를 만든다. MoE를 효율적으로 만드는 라우팅 메커니즘은 미묘한 무작위성의 원인이기도 하다. 즉, GPT-4o의 규모를 가능하게 한 바로 그 아키텍처가 본질적으로 예측 가능성을 저해하는 것이다. GPT-4o는 규모의 효율성을 위해 MoE 아키텍처를 사용하는 것으로 추정된다. MoE는 각 토큰에 사용할 전문가를 선택하는 라우터를 포함한다.19 효율성을 위해 추론은 여러 사용자 요청을 동시에 처리하는 배치 방식으로 이루어진다.26 한 사용자 시퀀스의 토큰에 대한 라우팅 결정은 동일한 배치에 있는 다른 시퀀스에 의해 미묘하게 영향을 받을 수 있다. 이는 계산이 병렬화되고 전문가 용량이 관리되는 방식 때문이다.21 이로 인해 온도를 0으로 설정하더라도 동일한 프롬프트가 어떤 다른 프롬프트와 같은 추론 배치에 있었는지에 따라 약간 다른 결과를 낳을 수 있는 상황이 발생한다.25 따라서 효율성을 위한 아키텍처 선택(MoE)과 핵심적인 한계(완벽한 결정성 부족) 사이에는 직접적인 인과 관계가 존재한다. 이는 API를 기반으로 구축하는 개발자들이 이해하고 설계에 반영해야 할 중요한 트레이드오프이다. 이것은 버그가 아니라 시스템 설계의 고유한 속성이다.



이 보고서의 분석을 종합하면, GPT-4o의 아키텍처적 우위는 서로 연결된 세 가지 혁신에 기반을 두고 있다.

1. **통합된 종단간 옴니 모달리티:** 진정으로 지연 시간이 짧은 멀티모달 인식을 가능하게 하는 근본적인 패러다임 전환.
2. **희소 전문가 혼합(Sparse Mixture of Experts, MoE):** 수조 개 파라미터급 모델을 계산적으로나 경제적으로 실행 가능하게 만드는 확장 메커니즘.
3. **통합된 생성 기술(트랜스퓨전):** 디퓨전과 같은 다양한 생성 과정을 단일 트랜스포머 내에 네이티브하게 내장하여 이해와 창작을 통합하는 방법.


GPT-4o의 아키텍처 접근법을 주요 경쟁자인 구글의 제미니(Gemini) 및 앤트로픽(Anthropic)의 클로드(Claude)와 비교하면, 서로 다른 아키텍처 철학과 강점을 발견할 수 있다. 제미니 또한 처음부터 멀티모달로 설계되었고 MoE를 사용하여 직접적인 경쟁자로 자리매김했다.51 반면 클로드는 매우 긴 컨텍스트 창과 안전을 위한 'Constitutional AI'에 중점을 두어, 깊이 있는 문서 분석과 신뢰성이 중요한 고위험 작업에서 탁월한 성능을 보인다.51

GPT-4o의 핵심 차별점은 통합 아키텍처가 제공하는 극도로 낮은 지연 시간을 바탕으로 한 *실시간, 상호작용적 인간-컴퓨터 인터페이스*에 대한 집중으로 보인다.7


GPT-4o의 아키텍처는 미래 AI 발전의 길을 열어준다. 낮은 지연 시간의 멀티모달 인식과 함수 호출/API 통합을 통한 행동 수행 능력의 결합은 진정한 AI 에이전트의 핵심 구성 요소이다.7

미래의 발전은 출력 모달리티 확장(예: 네이티브 비디오 생성), 모델의 추론 및 계획 능력 심화, 그리고 사용자를 대신하여 더 자율적인 작업을 수행하는 방향으로 나아갈 가능성이 높다. 결론적으로, GPT-4o 아키텍처는 종착점이 아니라, 차세대 상호작용적이고 에이전트적인 AI를 위한 기초 플랫폼이다.


1. GPT-4o - Wikipedia, accessed July 24, 2025, https://en.wikipedia.org/wiki/GPT-4o
2. OpenAI Unveils GPT-4o "Free AI for Everyone" : r/ChatGPT - Reddit, accessed July 24, 2025, https://www.reddit.com/r/ChatGPT/comments/1cr4hfd/openai_unveils_gpt4o_free_ai_for_everyone/
3. Hello GPT-4o - OpenAI, accessed July 24, 2025, https://openai.com/index/hello-gpt-4o/
4. OpenAI GPT-4: Architecture, Interfaces, Pricing, Alternative, accessed July 24, 2025, https://www.acorn.io/resources/learning-center/openai/
5. GPT-4o System Card | OpenAI, accessed July 24, 2025, https://openai.com/index/gpt-4o-system-card/
6. GPT-4o System Card | OpenAI, accessed July 24, 2025, https://cdn.openai.com/gpt-4o-system-card.pdf
7. GPT-4o - Sam Altman, accessed July 24, 2025, https://blog.samaltman.com/gpt-4o
8. GPT-4o Guide: How it Works, Use Cases, Pricing, Benchmarks | DataCamp, accessed July 24, 2025, https://www.datacamp.com/blog/what-is-gpt-4o
9. GPT-4 Technical Report - OpenAI, accessed July 24, 2025, https://cdn.openai.com/papers/gpt-4.pdf
10. GPT-4 Technical Report - arXiv, accessed July 24, 2025, http://arxiv.org/pdf/2303.08774
11. GPT-4 - OpenAI, accessed July 24, 2025, https://openai.com/index/gpt-4-research/
12. GPT-4 - Wikipedia, accessed July 24, 2025, https://en.wikipedia.org/wiki/GPT-4
13. GPT-4o Explained: Training, Features, and Performance - Data Science Dojo, accessed July 24, 2025, https://datasciencedojo.com/blog/gpt4o/
14. Introducing 4o Image Generation - OpenAI, accessed July 24, 2025, https://openai.com/index/introducing-4o-image-generation/
15. GPT-4o mini: advancing cost-efficient intelligence - OpenAI, accessed July 24, 2025, https://openai.com/index/gpt-4o-mini-advancing-cost-efficient-intelligence/
16. Is GPT-4 a Mixture of Experts Model? Exploring MoE Architectures for Language Models, accessed July 24, 2025, https://www.nownextlater.ai/Insights/post/is-gpt-4-a-mixture-of-experts-model-exploring-moe-architectures-for-language-models
17. Number of Parameters in GPT-4 (Latest Data) - Exploding Topics, accessed July 24, 2025, https://explodingtopics.com/blog/gpt-parameters
18. What is mixture of experts? | IBM, accessed July 24, 2025, https://www.ibm.com/think/topics/mixture-of-experts
19. Peering Inside GPT-4: Understanding Its Mixture of Experts (MoE) Architecture - Medium, accessed July 24, 2025, https://medium.com/@seanbetts/peering-inside-gpt-4-understanding-its-mixture-of-experts-moe-architecture-2a42eb8bdcb3
20. Mixture of Experts Approach for Large Language Models - Toloka, accessed July 24, 2025, https://toloka.ai/blog/mixture-of-experts-approach-for-llms/
21. Everything We Know About GPT-4 - Klu.ai, accessed July 24, 2025, https://klu.ai/blog/gpt-4-llm
22. Did we grossly overestimate the GPT4 parameters? : r/mlscaling - Reddit, accessed July 24, 2025, https://www.reddit.com/r/mlscaling/comments/1ebt2ut/did_we_grossly_overestimate_the_gpt4_parameters/
23. Mixture of Experts LLMs: Key Concepts Explained - neptune.ai, accessed July 24, 2025, https://neptune.ai/blog/mixture-of-experts-llms
24. Demystify GPT-4. GPT-4 shows significantly improved... | by Yan Xu | Medium, accessed July 24, 2025, https://medium.com/@YanAIx/demystify-gpt-4-58b332a4c731
25. Avoidable and Unavoidable Randomness in GPT-4o | Towards Data Science, accessed July 24, 2025, https://towardsdatascience.com/avoidable-and-unavoidable-randomness-in-gpt-4o/
26. Non-determinism in GPT-4 is caused by Sparse MoE - 152334H, accessed July 24, 2025, https://152334h.github.io/blog/non-determinism-in-gpt-4/
27. This isn't true, GPT4 is not a mixture of experts model. I'm on a quixotic missi... | Hacker News, accessed July 24, 2025, https://news.ycombinator.com/item?id=36892374
28. MoMa: Efficient Early-Fusion Pre-training with Mixture of Modality-Aware Experts - arXiv, accessed July 24, 2025, https://arxiv.org/html/2407.21770v1
29. Exploiting Mixture-of-Experts Redundancy Unlocks Multimodal Generative Abilities - arXiv, accessed July 24, 2025, https://arxiv.org/html/2503.22517v2
30. Transformer Meets Diffusion: How the Transfusion Architecture ..., accessed July 24, 2025, https://www.marktechpost.com/2025/04/06/transformer-meets-diffusion-how-the-transfusion-architecture-empowers-gpt-4os-creativity/
31. Transfusion: Predict the Next Token and Diffuse Images with One Multi-Modal Model, accessed July 24, 2025, https://openreview.net/forum?id=SI2hI0frk6
32. Transfusion: Predict the Next Token and Diffuse Images with One Multi-Modal Model | Research - AI at Meta, accessed July 24, 2025, https://ai.meta.com/research/publications/transfusion-predict-the-next-token-and-diffuse-images-with-one-multi-modal-model/
33. ICLR 2025 Transfusion: Predict the Next Token and Diffuse Images with One Multi-Modal Model Oral, accessed July 24, 2025, https://iclr.cc/virtual/2025/oral/31844
34. Transfusion: Predict the Next Token and Diffuse Images with One Multi-Modal Model - arXiv, accessed July 24, 2025, https://arxiv.org/html/2408.11039v1
35. Multi-modal Generative AI: Multi-modal LLMs, Diffusions and the Unification - arXiv, accessed July 24, 2025, https://arxiv.org/html/2409.14993v2
36. RLHF-V: Towards Trustworthy MLLMs via Behavior Alignment from Fine-grained Correctional Human Feedback - CVF Open Access, accessed July 24, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Yu_RLHF-V_Towards_Trustworthy_MLLMs_via_Behavior_Alignment_from_Fine-grained_Correctional_CVPR_2024_paper.pdf
37. Aligning Large Multimodal Models with Factually Augmented RLHF - ACL Anthology, accessed July 24, 2025, https://aclanthology.org/2024.findings-acl.775.pdf
38. Aligning Large Multimodal Models with Factually Augmented RLHF - OpenReview, accessed July 24, 2025, https://openreview.net/forum?id=B6t5wy6g5a
39. Safe RLHF-V: Safe Reinforcement Learning from Human Feedback in Multimodal Large Language Models - arXiv, accessed July 24, 2025, https://arxiv.org/html/2503.17682v1
40. Reinforcement learning with human feedback (RLHF) for LLMs - SuperAnnotate, accessed July 24, 2025, https://www.superannotate.com/blog/rlhf-for-llm
41. The challenges of reinforcement learning from human feedback ..., accessed July 24, 2025, https://bdtechtalks.com/2023/09/04/rlhf-limitations/
42. A Comprehensive Review: Model Compression for Large Language ..., accessed July 24, 2025, https://medium.com/@kimdoil1211/a-comprehensive-review-model-compression-for-large-language-models-llms-2c0c106d5dbe
43. A Survey on Model Compression for Large Language Models - ACL Anthology, accessed July 24, 2025, https://aclanthology.org/2024.tacl-1.85/
44. Urban Planning & AI: GPT 4o as Architect - Article - Parametric Solutions, accessed July 24, 2025, https://www.parametric.se/post/urban-planning-and-ai-gpt-4o-as-architect
45. GPT vs Claude vs Gemini: Comparing LLMs - Nu10, accessed July 24, 2025, https://nu10.co/gpt-vs-claude-vs-gemini-comparing-llms/
46. GPT 4o vs Claude 3.5 vs Gemini 2.0 - Which LLM to Use When - Analytics Vidhya, accessed July 24, 2025, https://www.analyticsvidhya.com/blog/2025/01/gpt-4o-claude-3-5-gemini-2-0-which-llm-to-use-and-when/
47. Claude 4 vs GPT-4o vs Gemini 2.5 Pro: Which AI Codes Best in 2025? - Analytics Vidhya, accessed July 24, 2025, https://www.analyticsvidhya.com/blog/2025/05/best-ai-for-coding/
48. Peer review of GPT-4 technical report and systems card - PMC, accessed July 24, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10795998/
49. Comparing GPT-4, 4o, and other LLMs for blogpost generation : r/OpenAI - Reddit, accessed July 24, 2025, https://www.reddit.com/r/OpenAI/comments/1dcl5w4/comparing_gpt4_4o_and_other_llms_for_blogpost/
50. Putting GPT-4o to the Sword: A Comprehensive Evaluation of Language, Vision, Speech, and Multimodal Proficiency - MDPI, accessed July 24, 2025, https://www.mdpi.com/2076-3417/14/17/7782
51. GPT-4 Turbo vs. Claude 3 Opus vs. Google Gemini 1.5 Pro, accessed July 24, 2025, https://www.kommunicate.io/blog/gpt4-vs-claude-3-vs-gemini/
52. Gemini Pro vs GPT-4: An in-depth comparison of LLM models – Bind AI IDE, accessed July 24, 2025, https://blog.getbind.co/2024/01/07/google-gemini-pro-vs-gpt-4-llm-models/
53. ChatGPT vs Gemini vs Claude: A Detailed Comparison - Kanerika, accessed July 24, 2025, https://kanerika.com/blogs/chatgpt-vs-gemini-vs-claude/
54. Claude 3 vs GPT-4 vs Gemini: Which is Better in 2024? | by Favour Kelvin - Medium, accessed July 24, 2025, https://favourkelvin17.medium.com/claude-3-vs-gpt-4-vs-gemini-2024-which-is-better-93c2607bf2fd
55. Generative pre-trained transformer - Wikipedia, accessed July 24, 2025, https://en.wikipedia.org/wiki/Generative_pre-trained_transformer


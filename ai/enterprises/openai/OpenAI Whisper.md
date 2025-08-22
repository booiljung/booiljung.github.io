---
layout: page
title: OpenAI Whisper 심층 고찰
permalink: /ai/enterprises/openai/OpenAI Whisper
---



2022년 9월 OpenAI가 공개한 Whisper는 자동 음성 인식(Automatic Speech Recognition, ASR) 기술의 역사에 중요한 분기점을 제시했다.1 Whisper 이전의 ASR 시스템들은 주로 고품질로 정제되고 레이블링된 '학술적' 데이터셋에 의존하여 훈련되었다.4 LibriSpeech와 같은 데이터셋은 깨끗한 오디오 환경에서 낭독된 음성으로 구성되어 있어, 이러한 데이터로 훈련된 모델들은 특정 벤치마크에서는 높은 정확도를 달성할 수 있었다.5 하지만 이는 통제된 실험실 환경과 현실 세계 사이의 간극을 드러내는 명백한 한계를 내포했다. 현실의 음성 데이터는 다양한 화자의 억양, 예측 불가능한 배경 소음, 기술 용어, 구어체 표현 등 복잡하고 비정형적인 요소로 가득 차 있으며, 기존 모델들은 이러한 '야생(in the wild)' 환경에서 강건성(robustness) 부족 문제를 보였다.1

Whisper는 이러한 근본적인 문제를 해결하기 위해 접근법 자체를 전환했다. 모델 아키텍처의 혁신에 집중하기보다, 훈련 데이터의 패러다임을 바꾸는 데 초점을 맞추었다. Whisper는 인터넷에서 수집한 방대하고 다양한, 그러나 상대적으로 정제되지 않은 68만 시간 분량의 오디오-텍스트 쌍을 활용하는 '대규모 약지도 학습(Large-Scale Weak Supervision)'이라는 개념을 도입하고 그 효과를 성공적으로 입증했다.4 이는 완벽하게 정제된 소량의 데이터보다, 약간의 노이즈를 포함하더라도 방대하고 다양한 데이터가 모델의 일반화 성능과 현실 세계 적응력을 획기적으로 향상시킬 수 있다는 것을 보여주었다. 이로써 ASR 연구의 무게 중심은 모델 중심(model-centric) 접근에서 데이터 중심(data-centric) 접근으로 이동하는 중요한 계기가 마련되었다.


Whisper의 가장 중요한 기여는 혁신적인 아키텍처의 발명이 아니라, 훈련 방법론의 성공적인 증명에 있다. 68만 시간이라는 전례 없는 규모의 다국어 및 다중 작업(multitask) 데이터셋을 통해, 단일 종단 간(end-to-end) 모델이 별도의 미세조정(fine-tuning) 과정 없이도 수많은 언어와 작업을 처리하는 강력한 제로샷(zero-shot) 일반화 성능을 달성할 수 있음을 실증적으로 보여주었다.4

이는 ASR 기술의 복잡성을 극적으로 단순화하는 결과를 낳았다. 전통적인 음성 처리 파이프라인은 음성 활동 감지(Voice Activity Detection, VAD), 언어 식별(Language Identification), 음성 인식, 번역 등 각각의 작업을 별도의 모델이나 모듈로 처리하는 복잡한 구조를 가졌다.14 반면 Whisper는 이러한 여러 기능을 하나의 통합된 트랜스포머 모델로 구현하여, 개발 및 유지보수의 복잡성을 크게 줄이고 기술 접근성을 획기적으로 높였다.13 또한, MIT 라이선스로 모델과 코드를 전면 공개함으로써 3, 이전까지는 막대한 자본과 데이터를 보유한 소수의 기업만이 접근할 수 있었던 고성능 ASR 기술을 전 세계의 개인 개발자, 연구자, 스타트업이 자유롭게 활용할 수 있는 길을 열었다. 이는 기술의 민주화를 촉진하고, `whisper.cpp`, `WhisperX`와 같은 수많은 파생 프로젝트와 혁신적인 애플리케이션의 탄생을 이끄는 기폭제가 되었다.


본 보고서는 Whisper 모델에 대한 포괄적이고 심층적인 분석을 제공하는 것을 목표로 한다.

첫째, 표준 트랜스포머에 기반한 Whisper의 기술적 아키텍처를 구성 요소별로 상세히 분석하고, 오디오 입력 처리부터 텍스트 출력까지의 전 과정을 추적한다.

둘째, 모델 성공의 핵심 동력인 대규모 약지도 학습 방법론을 데이터셋의 구성과 정제 과정을 중심으로 탐구한다.

셋째, 주요 벤치마크 데이터와 상용 API와의 비교를 통해 Whisper의 성능을 객관적으로 평가한다.

넷째, '환각(hallucination)' 현상을 포함한 모델의 본질적인 한계점과 기술적 난제를 비판적으로 고찰한다.

다섯째, 오픈소스 공개 이후 형성된 방대한 생태계와 주요 확장 프로젝트들을 조명하며 Whisper의 실용적 가치와 발전 가능성을 탐색한다.

마지막으로, 이러한 분석을 종합하여 Whisper가 ASR 분야에 미친 영향과 미래 기술의 발전 방향을 전망하며 결론을 맺는다.




Whisper의 아키텍처는 혁신적인 새로움보다는 검증된 기술의 효과적인 조합에 그 기반을 둔다. 음성 신호 시퀀스를 텍스트 토큰 시퀀스로 변환하기 위해, 자연어 처리 분야에서 표준으로 자리 잡은 트랜스포머(Transformer) 기반의 인코더-디코더 구조, 즉 시퀀스-투-시퀀스(Sequence-to-Sequence) 모델을 채택했다.1 이 구조는 복잡한 음성 인식 문제를 두 개의 하위 문제로 분해하여 접근한다. 인코더는 음향 신호의 특징을 학습하여 문맥적 의미를 담은 잠재 표현(latent representation)으로 변환하는 역할을 담당하고, 디코더는 이 잠재 표현을 조건으로 하여 최종적인 텍스트 시퀀스를 생성하는 역할을 수행한다.


인코더의 주된 임무는 입력된 로그-멜 스펙트로그램으로부터 음성의 핵심적인 음향적, 시간적, 문맥적 특징을 추출하여 고차원의 벡터 시퀀스로 압축하는 것이다.

- **입력 처리:** 인코더의 입력단은 두 개의 1차원 컨볼루션 레이어(Convolutional Layer)로 구성된다.18 각 레이어는 필터 크기 3과 GELU(Gaussian Error Linear Unit) 활성화 함수를 사용하며, 두 번째 레이어는 스트라이드(stride) 2를 적용하여 입력 시퀀스의 길이를 절반으로 줄인다. 이 컨볼루션 블록은 스펙트로그램에서 지역적인 패턴(local pattern)을 효과적으로 포착하여 트랜스포머 블록이 처리하기 용이한 형태로 변환하는 역할을 한다.
- **위치 인코딩:** 트랜스포머는 순환 신경망(RNN)과 달리 순서 정보를 내재하고 있지 않으므로, 입력 시퀀스에서 토큰의 상대적 또는 절대적 위치 정보를 명시적으로 주입해야 한다. Whisper는 사인(Sinusoidal) 함수를 이용한 고정된 위치 임베딩을 컨볼루션 블록의 출력에 더하여 시간적 순서 정보를 모델에 제공한다.3
- **트랜스포머 블록:** 위치 정보가 추가된 임베딩 시퀀스는 여러 개의 인코더 블록을 순차적으로 통과한다. 각 블록은 멀티헤드 셀프 어텐션(Multi-Head Self-Attention) 메커니즘과 완전 연결 피드포워드 네트워크(Feed-Forward Network)라는 두 개의 하위 레이어로 구성된다. 셀프 어텐션은 입력 시퀀스 내의 모든 위치 쌍 간의 관계를 계산하여 각 위치의 표현을 문맥에 맞게 정교화한다. Whisper는 표준 트랜스포머와 달리 사전 활성화(pre-activation) 방식을 사용하고 잔차 연결(residual connection)과 레이어 정규화(layer normalization)를 적용하여 안정적인 학습을 도모한다.3


디코더는 인코더가 생성한 잠재 표현과 이전에 예측된 텍스트 토큰들을 조건부 입력으로 받아, 다음 텍스트 토큰을 자가회귀적(autoregressively) 방식으로 예측한다. 즉, 한 번에 한 토큰씩 순차적으로 텍스트를 생성한다.

- **입력:** 디코더의 입력은 이전에 생성된 텍스트 토큰들의 임베딩과 학습 가능한(learned) 위치 임베딩의 합으로 구성된다.
- **어텐션 메커니즘:** 디코더의 각 블록은 세 개의 하위 레이어를 포함한다.
  1. **마스크된 멀티헤드 셀프 어텐션 (Masked Multi-Head Self-Attention):** 이 메커니즘은 디코더가 이미 생성한 텍스트 시퀀스 내의 문법적, 의미적 관계를 파악하도록 돕는다. '마스크된'이라는 용어는 특정 토큰을 예측할 때 그 이후에 위치한 토큰들을 참조할 수 없도록(look-ahead mask) 하여, 자가회귀적 생성 과정의 정보 누수를 방지하는 것을 의미한다.
  2. **멀티헤드 교차 어텐션 (Multi-Head Cross-Attention):** 이 메커니즘은 디코더의 핵심 기능으로, 디코더가 인코더의 최종 출력(오디오 정보가 압축된 잠재 표현)에 집중(attend)하도록 한다.19 즉, 텍스트를 생성하는 매 시점마다 원본 오디오의 어떤 부분에 주목해야 할지를 동적으로 결정한다. 이를 통해 음성 내용에 충실한 텍스트 생성이 가능해진다.
  3. **완전 연결 피드포워드 네트워크:** 어텐션 메커니즘을 통해 얻은 정보를 처리하고 다음 레이어로 전달한다.
- **토큰 표현 공유:** 모델의 효율성을 높이고 파라미터 수를 줄이기 위해, 디코더의 입력 토큰 임베딩 행렬과 최종 출력 로짓(logit)을 계산하는 선형 변환 레이어의 가중치 행렬을 공유(tied input-output token representations)하는 기법을 사용한다.3

Whisper의 진정한 설계적 기여는 이러한 표준 아키텍처 구성 요소들을 '특수 토큰을 이용한 다중 작업 프롬프팅'이라는 제어 메커니즘과 결합한 데 있다. 전통적인 ASR 시스템이 각기 다른 모델로 처리하던 VAD, 언어 식별, 번역 등의 작업을, Whisper는 디코더의 초기 입력으로 특정 토큰 시퀀스를 제공함으로써 단일 모델이 유연하게 수행하도록 만들었다. 예를 들어, `<|startoftranscript|><|ja|><|translate|>`와 같은 프롬프트는 모델에게 "일본어 음성을 영어 텍스트로 번역하라"는 명시적인 지시로 작용한다. 이처럼 기존 아키텍처를 다중 작업에 맞게 제어하는 프롬프팅 기반 인터페이스 설계는 복잡한 ASR 파이프라인을 극도로 단순화시키는 핵심적인 혁신이라 할 수 있다.


Whisper는 다양한 컴퓨팅 환경과 요구사항에 대응하기 위해 여러 크기의 모델을 제공한다. 각 모델은 정확도, 추론 속도, 그리고 VRAM 요구사항 간의 트레이드오프 관계를 가지며, 이는 아키텍처의 구체적인 파라미터 구성에 의해 결정된다. 아래 표는 각 모델 변형의 기술적 사양을 요약한 것으로, 사용자가 자신의 용도에 가장 적합한 모델을 선택하는 데 중요한 기준을 제공한다.11

| 크기 (Size) | 파라미터 (Parameters) | 인코더 레이어 (Encoder Layers) | 디코더 레이어 (Decoder Layers) | 모델 차원 ($d_{model}$) | 어텐션 헤드 (Heads) | 필요 VRAM (Approx. VRAM) | 상대 속도 (Relative Speed) |
| ----------- | --------------------- | ------------------------------ | ------------------------------ | ----------------------- | ------------------- | ------------------------ | -------------------------- |
| `tiny`      | 39 M                  | 4                              | 4                              | 384                     | 6                   | ~1 GB                    | ~10x                       |
| `base`      | 74 M                  | 6                              | 6                              | 512                     | 8                   | ~1 GB                    | ~7x                        |
| `small`     | 244 M                 | 12                             | 12                             | 768                     | 12                  | ~2 GB                    | ~4x                        |
| `medium`    | 769 M                 | 24                             | 24                             | 1024                    | 16                  | ~5 GB                    | ~2x                        |
| `large`     | 1550 M                | 32                             | 32                             | 1280                    | 20                  | ~10 GB                   | 1x                         |
| `large-v2`  | 1550 M                | 32                             | 32                             | 1280                    | 20                  | ~10 GB                   | 1x                         |
| `large-v3`  | 1550 M                | 32                             | 32                             | 1280                    | 20                  | ~10 GB                   | ~1x                        |
| `turbo`     | 809 M                 | -                              | -                              | -                       | -                   | ~6 GB                    | ~8x                        |


Whisper 모델이 음성을 처리하기 위해서는 먼저 아날로그 오디오 신호를 모델이 이해할 수 있는 디지털 표현으로 변환해야 한다. 이 과정은 여러 단계의 표준화된 전처리 파이프라인을 통해 이루어진다.

1. **리샘플링 (Resampling):** 다양한 소스에서 수집된 오디오 파일은 각기 다른 샘플링 레이트(sampling rate)를 가질 수 있다. 모델이 일관된 조건에서 학습하고 추론할 수 있도록, 모든 입력 오디오는 16,000 Hz의 샘플링 레이트로 변환(리샘플링)된다.3
2. **청크 분할 (Chunking):** 트랜스포머 모델은 고정된 길이의 시퀀스만 처리할 수 있다. Whisper의 수용 필드(receptive field)는 30초로 고정되어 있으므로, 입력 오디오는 30초 길이의 세그먼트(청크)로 분할된다.1 30초보다 긴 오디오는 여러 개의 청크로 나뉘어 순차적으로 처리된다.
3. **로그-멜 스펙트로그램 변환:** 각 30초 청크는 로그-멜 스펙트로그램(log-Mel spectrogram)으로 변환된다. 이는 오디오 신호를 시간-주파수 영역에서 시각적으로 표현한 것으로, 음성 인식에 매우 효과적인 특징 표현 방식이다.1 이 변환 과정은 다음과 같다.
   - 먼저, 단시간 푸리에 변환(Short-Time Fourier Transform, STFT)을 25ms 길이의 윈도우(window)와 10ms의 스트라이드(stride)를 적용하여 오디오 신호의 스펙트로그램을 계산한다.3
   - 계산된 스펙트로그램의 주파수 축은 인간의 청각 시스템이 주파수를 비선형적으로 인식하는 특성을 모방한 멜 스케일(Mel scale)로 변환된다. 이는 저주파 대역에서는 세밀하게, 고주파 대역에서는 넓게 주파수를 구분하여 음성의 주요 특징을 더 효과적으로 포착한다.22 Whisper는 80개(large-v3 모델에서는 128개)의 멜 주파수 빈(Mel frequency bins)을 사용한다.3
   - 마지막으로, 진폭(amplitude)에 로그 스케일을 적용하여 값의 분포를 안정시키고 인간의 소리 크기 인식과 유사하게 만든다.
4. **정규화 (Normalization):** 최종적으로 생성된 로그-멜 스펙트로그램의 값은 [-1, 1] 범위로 정규화되어 평균이 0에 가깝도록 조정된다.3 이는 신경망 모델의 학습 과정을 안정화하고 수렴 속도를 높이는 데 도움을 준다.



Whisper는 텍스트를 모델이 처리할 수 있는 정수 시퀀스로 변환하기 위해 바이트 페어 인코딩(Byte-Pair Encoding, BPE) 토크나이저를 사용한다.3 BPE는 자주 등장하는 문자열을 점진적으로 병합하여 하나의 토큰으로 만드는 데이터 압축 알고리즘에 기반하며, 고정된 크기의 어휘 사전으로 개방 어휘(open-vocabulary) 문제를 효과적으로 처리할 수 있다. 영어 전용 모델은 GPT-2의 어휘 사전을 그대로 사용하고, 다국어 모델은 99개 언어를 포괄하도록 재학습된 다국어 어휘 사전을 사용한다.3 이 토큰화 과정의 속도를 높이기 위해 OpenAI가 자체 개발한 `tiktoken` 라이브러리가 백엔드에서 활용된다.14


Whisper가 음성 인식, 번역, 언어 식별 등 다양한 작업을 단일 모델로 수행할 수 있는 비결은 바로 '특수 토큰(special tokens)'의 전략적 사용에 있다. 이 토큰들은 디코더가 텍스트 생성을 시작하기 전에 초기 프롬프트의 일부로 제공되어, 모델에게 수행해야 할 작업의 종류와 세부 사항을 명시적으로 지시하는 역할을 한다.3

- `<|startoftranscript|>`: 모든 전사(transcription)의 시작을 알리는 고정된 토큰이다.
- **언어 토큰 (Language Token):** `<|en|>`, `<|ko|>`, `<|ja|>` 등 지원하는 각 언어를 나타내는 고유한 토큰이다. 모델이 어떤 언어로 텍스트를 생성해야 할지 지정한다. 언어를 지정하지 않으면 모델은 오디오를 기반으로 언어를 자동으로 감지하여 해당 언어 토큰을 예측한다.
- **작업 토큰 (Task Token):** 두 가지 주요 작업 토큰이 있다. `<|transcribe|>`는 입력 오디오와 동일한 언어로 음성 인식을 수행하라는 지시이며, `<|translate|>`는 입력 오디오를 영어로 번역하라는 지시이다.
- `<|notimestamps|>`: 타임스탬프 예측을 생략하고 싶을 때 사용한다. 이 토큰이 프롬프트에 포함되지 않으면, 모델은 20ms 단위로 양자화된 발화 구간의 시작 및 종료 시간을 예측하는 타임스탬프 토큰을 함께 생성한다.3
- `<|nospeech|>`: 음성 활동 감지(VAD)를 위한 토큰으로, 해당 오디오 세그먼트에 유의미한 음성이 없음을 나타내는 데 사용된다.3


사용자는 이러한 특수 토큰 외에도 추가적인 텍스트를 프롬프트로 제공하여 모델의 출력을 유도할 수 있다. 특히 긴 오디오를 여러 세그먼트로 나누어 처리할 때, 이전 세그먼트의 전사 결과를 다음 세그먼트의 프롬프트로 제공하면 문맥적 일관성을 유지하고 고유 명사나 전문 용어의 인식률을 높이는 데 도움이 된다. 단, `whisper-1` API 모델의 경우 프롬프트의 마지막 224개 토큰만 컨텍스트로 고려하는 제약이 있다.23



Whisper의 경이로운 성능은 그 기반이 되는 방대하고 다양한 학습 데이터셋에서 비롯된다. 모델은 인터넷의 공개된 자료로부터 수집된 총 68만 시간 분량의 오디오와 그에 해당하는 텍스트 쌍으로 학습되었다.1 이는 기존의 대표적인 학술 데이터셋인 LibriSpeech(약 1,000시간)와 비교할 수 없을 정도로 거대한 규모이다.

- **데이터 구성:** 이 데이터셋은 다국어 및 다중 작업 학습을 위해 전략적으로 구성되었다.
  - **다국어 데이터:** 전체 데이터의 약 1/3에 해당하는 117,000시간은 영어가 아닌 96개의 다른 언어로 구성되어, 모델이 다양한 언어의 음운 및 구조적 특징을 학습하도록 한다.7
  - **음성 번역 데이터:** 약 125,000시간은 비영어권 음성을 영어 텍스트로 번역하는($X \rightarrow en$) 데이터로, 모델이 번역 작업을 직접 학습하도록 한다.7
  - **영어 데이터:** 나머지 약 65%에 해당하는 438,000시간은 영어 음성과 영어 텍스트 쌍으로 구성되어, 영어 인식 성능의 기반을 다진다.9
- **데이터 정제 과정:** 웹에서 수집된 원시 데이터는 품질이 고르지 않고 상당한 노이즈를 포함하고 있다. 따라서 OpenAI는 고품질의 학습을 보장하기 위해 여러 단계에 걸친 자동화된 필터링 및 정제 파이프라인을 구축했다.
  - **기계 생성 텍스트 제거:** 기존 ASR 시스템이 자동으로 생성한 것으로 추정되는 저품질 텍스트를 걸러내기 위해, 비정상적인 대소문자 사용 패턴(예: 전부 대문자 또는 소문자), 구두점의 부재 등과 같은 휴리스틱 규칙을 적용했다.3 이는 다른 모델의 체계적인 오류를 Whisper가 그대로 학습하는 것을 방지하는 중요한 단계이다.
  - **언어 식별 및 일치:** 신뢰도 높은 언어 식별 모델을 사용하여 오디오의 언어와 텍스트의 언어가 일치하는지 검증했다.3 불일치하는 데이터는 학습에서 제외하여 레이블 노이즈를 줄였다.
  - **중복 제거:** 텍스트 내용이 유사한 데이터가 과도하게 포함되는 것을 막기 위해 퍼지 중복 제거(fuzzy de-duping) 기술을 사용했다. 또한, 모델의 성능을 평가할 표준 벤치마크 데이터셋이 훈련 데이터에 포함되지 않도록 엄격하게 중복을 검사하고 제거하여, 공정한 평가를 보장하고 데이터 오염(data contamination)을 방지했다.3
  - **반복적 정제:** 초기 버전의 Whisper 모델을 학습시킨 후, 이 모델을 사용하여 전체 훈련 데이터셋에 대한 오류율을 분석했다. 오류율이 비정상적으로 높은 데이터 소스를 식별하고 수동으로 검토하여, 품질이 낮은 데이터셋을 훈련 데이터에서 제거하는 추가적인 정제 과정을 거쳤다.7


'약지도 학습(Weak Supervision)'은 완벽하게 정제되고 검증된 '황금 표준(gold-standard)' 데이터가 아닌, 대량의 노이즈가 포함될 수 있는 레이블을 사용하여 모델을 학습시키는 방법론을 의미한다. Whisper는 데이터의 개별적인 '품질'을 어느 정도 희생하는 대신, 데이터의 총체적인 '양'과 '다양성'을 극대화하는 전략을 채택했다.7 이 접근법은 ASR 분야에서 역설적인 결과를 낳았다.

전통적으로 '노이즈'로 간주되어 제거 대상이었던 데이터의 불완전성이, 대규모 스케일에서는 오히려 모델의 '강건성(robustness)'을 키우는 원동력이 되었다. 수십만 시간에 달하는 데이터에는 수많은 화자의 다양한 억양, 방언, 발화 습관, 그리고 각양각색의 배경 소음, 녹음 장비의 차이, 채널 왜곡 등 현실 세계에서 마주할 수 있는 거의 모든 음향적 변수가 자연스럽게 포함되어 있다. 모델은 이러한 방대하고 이질적인 데이터를 학습하는 과정에서, 특정 데이터셋이나 특정 환경에만 존재하는 편향되거나 특이한 패턴(spurious patterns)에 과적합되는 것을 피하게 된다.6 대신, 다양한 변수에도 불구하고 변하지 않는 음성의 본질적이고 일반화 가능한 특징을 학습하도록 강제된다. 이것이 바로 Whisper가 별도의 적응 훈련 없이도 낯선 데이터와 환경에 대해 높은 성능을 보이는 핵심적인 이유이다.4

이러한 접근 방식은 "더 깨끗한 데이터가 항상 더 좋은 모델을 만드는가?"라는 오랜 믿음에 대한 강력한 반례를 제시한다. 대규모 스케일에서는 데이터의 '다양성'이 개별 데이터의 '완벽성'보다 모델의 일반화 성능에 더 결정적인 기여를 할 수 있음을 시사한다. Whisper의 훈련 데이터는 단순히 노이즈가 섞인 것이 아니라, 자동화된 필터링을 통해 최악의 데이터를 걸러내면서도 현실 세계의 다양성을 최대한 보존한 '관리된 노이즈(managed noise)' 데이터셋이라고 볼 수 있다. 이는 모델에게 일종의 자연스러운 데이터 증강(data augmentation) 효과를 제공하여, 현실의 복잡성에 대한 대응 능력을 근본적으로 향상시킨다.

그러나 이러한 접근에는 숨겨진 비용이 따른다. 대규모 약지도 학습은 모델의 강건성을 높이는 데 성공했지만, 웹 데이터에 내재된 유해하거나 편향된 내용까지 모델이 무비판적으로 학습할 위험을 내포한다. 이는 이후에 논의될 '환각' 현상에서 폭력적이거나 차별적인 내용이 생성되는 근본적인 원인 중 하나로 작용할 수 있다. 즉, 데이터 수집의 용이성과 다양성 확보라는 이점 뒤에는 데이터 통제의 어려움과 잠재적 위험이라는 상충관계가 존재한다.



Whisper의 성능을 평가할 때 가장 주목해야 할 점은 특정 벤치마크에서의 최고 점수(State-Of-The-Art, SOTA) 달성 여부보다, 다양한 환경에 대한 일관되고 강력한 제로샷(zero-shot) 일반화 능력이다. 제로샷 성능이란, 모델이 특정 데이터셋에 대해 별도의 미세조정(fine-tuning)을 거치지 않고 사전 훈련된 상태 그대로 평가되었을 때의 성능을 의미한다. Whisper는 이 제로샷 설정에서 기존의 많은 모델들을 압도하는 성능을 보여주며, 이는 모델의 뛰어난 일반화 능력을 입증하는 강력한 증거이다.4

물론, LibriSpeech와 같이 고도로 정제된 학술 벤치마크에서는 해당 데이터셋에만 특화되어 집중적으로 훈련 및 미세조정된 모델이 Whisper보다 더 낮은 단어 오류율(Word Error Rate, WER)을 기록할 수 있다.13 그러나 이는 마치 특정 시험 과목만 집중적으로 공부한 학생이 그 과목에서 높은 점수를 받는 것과 같다. OpenAI의 연구에 따르면, 여러 다양한 데이터셋에 대한 평균적인 성능을 측정했을 때, Whisper는 기존 모델들보다 오류를 50%나 적게 발생시키며 훨씬 더 높은 평균적인 강건성을 나타냈다.6 이는 ASR 모델 평가의 관점이 단일 벤치마크 점수 경쟁에서, 예측 불가능한 실제 환경에 대한 적응력을 측정하는 방향으로 전환되어야 함을 시사한다. Whisper는 '정확도 대 일반화'라는 기존의 트레이드오프 관계를 깨고, 두 마리 토끼를 모두 잡는 새로운 가능성을 제시했다.


아래 표는 Whisper 모델(large 모델 기준)의 성능을 다른 주요 오픈소스 ASR 모델들과 표준 벤치마크 데이터셋에서 직접 비교한 것이다. WER은 낮을수록 더 좋은 성능을 의미하며, 이를 통해 Whisper의 상대적인 정확도를 객관적으로 파악할 수 있다.1

| 모델 (Model)        | 데이터셋 (Dataset)     | WER (clean) | WER (other/noisy) |
| ------------------- | ---------------------- | ----------- | ----------------- |
| **Whisper (large)** | **LibriSpeech**        | **2.7%**    | **5.2%**          |
| Kaldi               | LibriSpeech            | 3.8%        | 8.76%             |
| SpeechBrain         | LibriSpeech            | 2.46%       | 5.77%             |
| **Whisper (large)** | **Common Voice (avg)** | **9.0%**    | -                 |
| Kaldi               | Common Voice (avg)     | 4.44%       | -                 |
| Wav2vec 2.0         | Common Voice (en)      | 16.1%       | -                 |


Whisper는 오픈소스 모델일 뿐만 아니라, OpenAI에 의해 상용 API 서비스로도 제공된다. 여러 독립적인 벤치마크 테스트 결과, Whisper API는 정확도와 비용 효율성 측면에서 기존의 주요 클라우드 기반 ASR 서비스들과 강력한 경쟁 구도를 형성하고 있다.

- **정확도:** 다수의 비교 분석에서 Whisper API는 Google Speech-to-Text, Amazon Transcribe, Microsoft Azure AI Speech와 비교했을 때 동등하거나 더 우수한 정확도(낮은 WER)를 기록하는 경향을 보인다.27 특히, 특정 분야의 전문 용어나 고유 명사, 복잡한 문장 구조를 인식하는 데 있어서 강점을 나타내는 사례가 보고되었다.28
- **비용 효율성:** Whisper API는 2023년 출시 당시 분당 $0.006라는 파격적인 가격으로 책정되었다.29 이는 분당 약 $0.016 ~ $0.024 수준인 Google, Amazon 등 주요 경쟁사 서비스에 비해 현저히 낮은 가격으로, 고품질 ASR 기술에 대한 접근 비용을 크게 낮추는 효과를 가져왔다.27
- **기능적 차이:** 그러나 상용 API들은 단순 전사 기능 외에 Whisper가 기본적으로 제공하지 않는 다양한 부가 기능을 통해 차별화를 꾀하고 있다. 예를 들어, 여러 화자의 발언을 구분하는 화자 분리(diarization), 실시간 오디오 스트림을 처리하는 기능, 특정 도메인에 맞게 모델을 맞춤화하는 기능 등은 Azure AI Speech나 Amazon Transcribe와 같은 서비스의 주요 장점이다.30

이러한 비교는 Whisper가 ASR 시장에 미친 파괴적인 영향을 명확히 보여준다. '품질은 더 높고, 가격은 더 낮은' 대안의 등장은 기존 업체들로 하여금 가격 경쟁에 나서거나, 단순 전사를 넘어선 부가 가치 기능(오디오 인텔리전스) 개발에 더 집중하도록 만드는 시장 재편을 촉진했다.


아래 표는 개발자나 기업이 ASR 솔루션을 선택할 때 핵심적으로 고려하는 정확도와 비용을 직접적으로 비교하여, Whisper의 시장 내 경쟁력을 보여준다.27

| 서비스 (Service)                  | 영어 WER (Median/Reported) | 비용 (Price per minute) |
| --------------------------------- | -------------------------- | ----------------------- |
| **OpenAI Whisper API (large-v2)** | **~8.06%**                 | **$0.006**              |
| Google Speech-to-Text             | 16.51% - 20.63%            | ~$0.016                 |
| Amazon Transcribe                 | 18.42% - 22.0%             | ~$0.024                 |
| Microsoft Azure AI Speech         | ~14.70% (with phrase set)  | (비교 데이터 부족)      |


Whisper의 성능은 모든 조건에서 균일하지 않으며, 입력 데이터의 특성에 따라 상당한 편차를 보인다. 이는 모델의 강점과 약점을 이해하고 실제 적용 시 잠재적 오류를 예측하는 데 중요한 고려사항이다.9

- **언어:** 훈련 데이터의 언어별 분포가 불균등하기 때문에, 데이터가 풍부한 영어와 같은 고자원 언어에서는 매우 높은 성능을 보이지만, 상대적으로 데이터가 적은 저자원 언어에서는 오류율이 높게 나타나는 경향이 있다.9 OpenAI는 공식적으로 WER이 50%를 초과하는 언어는 지원 목록에서 제외한다고 밝히고 있다.23
- **억양:** 여러 연구에 따르면, Whisper는 특정 억양에 대해 더 나은 성능을 보인다. 예를 들어, 북미 영어 억양이 영국이나 호주 억양에 비해 더 높은 인식률을 보였으며, 전반적으로 원어민의 억양이 비원어민의 억양보다 높은 정확도를 기록했다.32
- **화자 특성:** 화자의 인구통계학적 특성 또한 성능에 영향을 미치는 변수로 작용한다. 한 연구에서는 화자의 성별, 모국어의 음운론적 유형(예: 중국어와 같은 성조 언어 화자가 비성조 언어 화자보다 높은 오류율을 보임) 등이 WER과 유의미한 상관관계를 보임을 발견했다.32 이러한 편향은 훈련 데이터의 인구통계학적 분포를 반영하는 결과로 해석될 수 있으며, 공정성(fairness) 관점에서 중요한 과제를 제기한다.


Whisper는 ASR 기술에 혁신을 가져왔지만, 완벽한 시스템은 아니다. 특히 고신뢰성이 요구되는 분야에 적용하기 전에 반드시 인지해야 할 몇 가지 본질적인 한계와 난제를 안고 있다.



Whisper의 가장 심각하고 예측하기 어려운 문제점은 '환각(hallucination)' 현상이다. 이는 입력된 오디오에 전혀 존재하지 않는 단어, 구, 심지어는 완전히 새로운 문장을 모델이 생성해내는 오류를 의미한다.37 이 현상은 단순한 오인식(misrecognition)을 넘어, 모델이 내용을 '창작'하는 것에 가깝다. 주요 발생 원인은 다음과 같이 분석된다.

- **비음성 오디오 및 묵음:** 환각은 오디오 내에 긴 묵음 구간이나 배경 소음과 같은 비음성(non-speech) 요소가 입력될 때 특히 빈번하게 발생한다.38 모델이 의미 있는 음성 신호가 없는 모호한 입력을 받았을 때, 훈련 데이터에서 학습한 가장 통계적으로 그럴듯한(plausible) 텍스트 패턴을 출력하려는 경향이 있기 때문이다. 한 연구에서는 완전한 무음(silent) 파일을 입력했을 때 모델이 "Thank you"라는 텍스트를 생성하는 사례를 보고하기도 했다.39 실어증 환자와 같이 발화 중간에 멈춤이 잦은 화자의 음성에서 환각 발생률이 더 높다는 연구 결과는 이 가설을 뒷받침한다.38
- **생성 모델의 본질적 특성:** Whisper의 디코더는 본질적으로 이전 토큰들을 기반으로 다음 토큰을 예측하는 언어 모델(Language Model)과 유사한 구조를 가진다. 이로 인해 오디오 정보가 불분명할 경우, 디코더는 오디오 입력보다 내부적으로 학습된 언어적 패턴에 더 의존하여, 문법적으로는 완벽하지만 실제 내용과는 무관한 텍스트를 '상상'해낼 수 있다.38


환각 현상이 더욱 심각한 문제인 이유는 생성된 내용이 무해한 잡담에 그치지 않고, 때로는 매우 유해하거나 위험한 내용을 포함할 수 있기 때문이다. 한 연구에 따르면, 분석된 환각 사례의 약 38%는 폭력 조장, 부정확한 개인정보(가짜 이름, 주소, 웹사이트) 생성, 허위 사실 유포 등 명백히 해로운 내용을 담고 있었다.38

- **폭력적 내용 생성:** "그 소년은 우산을 가지러 갔다(take the umbrella)"는 평범한 문장이 "그는 테러용 칼을 가지고 있었고, 그가 죽인 수많은 사람들을 죽였다(he didn't have a terror knife so he killed a number of people who he killed)"와 같은 끔찍하고 폭력적인 내용으로 변환된 사례가 보고되었다.38
- **가짜 정보 및 유해 연관성 생성:** "아빠와 고양이를 구하기 위해 소방서에 전화해야 했다"는 내용이 "그가 가진 것이라고는 피에 젖은 유모차 위에 놓인 냄새나는 낡은 머리뿐이었다"와 같이 기괴하고 부정확한 연관성을 만들어내는 경우도 있었다.38 또한, 실제 존재하지 않는 웹사이트 주소를 생성하여 피싱 공격에 악용될 가능성도 제기되었다.39

이러한 환각 현상은 Whisper의 강건성을 만든 '대규모의 다양한 웹 데이터'가 동시에 환각의 원인이 되는 '예측 불가능한 노이즈와 편향'을 주입하는 원천이기도 하다는 점을 보여준다. 즉, 모델의 가장 큰 장점과 가장 위험한 단점은 동일한 원인(훈련 데이터의 특성)에서 비롯된 양면의 칼과 같다. 현실 세계의 다양성을 학습한 대가로, 현실 세계의 혼돈과 편향까지 일부 학습하게 된 것이다.


환각 문제를 해결하기 위한 연구가 활발히 진행되고 있으며, 몇 가지 접근법이 제안되었다.

- **후처리 기반 제거:** 자주 발생하는 환각 텍스트의 목록, 이른바 '환각의 가방(Bag of Hallucinations, BoH)'을 미리 구축하고, ASR 모델의 출력 텍스트에서 해당 목록에 포함된 구절을 후처리 단계에서 탐지하고 제거하는 방법이 제안되었다.40
- **음성 활동 감지(VAD) 강화:** 환각의 주된 원인인 비음성 구간을 전사 과정에서 제외하기 위해, 고성능의 VAD 모델을 전처리 단계에 적용하는 접근법이 효과적인 것으로 나타났다. 이는 환각 발생의 근본 원인을 차단하는 데 도움을 준다.43
- **디코딩 파라미터 조정:** 빔 서치(beam search)의 빔 크기를 늘리거나, 온도(temperature) 샘플링과 같은 디코딩 전략 파라미터를 조정하여 반복적인 텍스트 생성이나 환각을 억제하려는 시도가 있다.22 그러나 이는 근본적인 해결책이 되지는 못한다.


Whisper는 강력한 핵심 엔진을 제공하지만, 상용 제품 수준에서 요구되는 여러 기능들이 누락되어 있다.

- **실시간 처리 부재:** Whisper의 기본 아키텍처는 30초 길이의 오디오 청크를 단위로 하는 배치(batch) 처리를 기반으로 설계되었다. 따라서 실시간으로 입력되는 오디오 스트림을 낮은 지연 시간(low latency)으로 처리하는 기능을 내장하고 있지 않다.1 이는 라이브 방송 자막, 실시간 회의록 작성, 음성 비서와 같은 실시간 상호작용이 필수적인 응용 분야에 직접 사용하기 어렵게 만드는 가장 큰 제약 중 하나이다.
- **화자 분리 기능 부재:** 모델은 여러 명의 화자가 대화하는 오디오에서 각 발화가 누구의 목소리인지 구분하는 화자 분리(speaker diarization) 기능을 기본적으로 제공하지 않는다.1 이로 인해 회의록이나 인터뷰 녹취를 처리할 때, "화자 A:", "화자 B:"와 같이 발언자를 구분한 스크립트를 생성할 수 없다.
- **긴 오디오 처리의 비효율성:** 30초라는 고정된 수용 필드(receptive field) 때문에, 이보다 긴 오디오 파일은 여러 개의 30초 세그먼트로 분할하여 순차적으로 처리해야 한다.21 이 과정에서 세그먼트 간의 경계에서 문맥이 단절되어 인식 오류가 발생하거나, 전체 처리 시간이 길어지는 문제가 발생할 수 있다.
- **API 제한 사항:** OpenAI가 공식적으로 제공하는 Whisper API는 업로드 가능한 파일 크기를 25MB로 제한하고 있다.1 이는 고음질의 긴 오디오 파일(예: 1시간 분량의 팟캐스트나 강의)을 한 번에 처리하는 것을 어렵게 만든다.

흥미롭게도, OpenAI가 이러한 '제품 수준'의 기능들을 의도적으로 제외하고 핵심 ASR 엔진에 집중하여 모델을 공개한 것은 역설적으로 커뮤니티의 혁신을 촉발하는 계기가 되었다. 이러한 '기능적 공백'은 전 세계 개발자들이 그 공백을 메우기 위한 독창적인 솔루션(`WhisperX`, `WhisperLive` 등)을 개발하도록 강력한 동기를 부여했다. 이는 오픈소스 전략이 단순히 기술을 공개하는 것을 넘어, 커뮤니티의 집단 지성을 통해 제품을 함께 완성해 나가는 강력한 생태계 구축 전략이 될 수 있음을 보여주는 사례이다.


OpenAI가 Whisper를 오픈소스로 공개한 결정은 ASR 기술의 발전에 지대한 영향을 미쳤다. 이는 단순히 강력한 모델을 무료로 제공하는 것을 넘어, 전 세계 개발자와 연구자들이 모델의 한계를 극복하고 새로운 가능성을 탐색하는 거대한 협력의 장을 열었다. 그 결과, Whisper를 중심으로 빠르고 견고하며 기능적으로 확장된 방대한 생태계가 자생적으로 형성되었다.


Whisper의 원본 구현은 연구 목적으로 개발된 파이토치(PyTorch) 코드베이스에 기반하고 있어, 실제 제품 환경에 배포하기에는 추론 속도나 메모리 효율성 측면에서 한계가 있었다. 이러한 문제를 해결하기 위해 커뮤니티는 다양한 고성능 구현체와 최적화 기술을 개발했다.

- **고성능 추론 엔진:**

  - `whisper.cpp`: Georgi Gerganov가 개발한 이 프로젝트는 Whisper 모델을 순수 C/C++로 재구현한 것이다.48 파이토치와 같은 무거운 딥러닝 프레임워크에 대한 의존성을 제거하고, 자체 개발한 경량 텐서 라이브러리인 

    `ggml`을 사용하여 CPU 환경에서도 놀라울 정도로 빠른 추론 속도를 보여준다. 특히 Apple Silicon의 Neural Engine, Intel의 OpenVINO, Core ML 등 다양한 하드웨어 가속 기술을 지원하여, 데스크톱 애플리케이션은 물론 모바일 및 임베디드 시스템과 같은 엣지 디바이스에 Whisper를 배포하는 사실상의 표준으로 자리 잡았다.48

  - `faster-whisper`: Hugging Face가 개발한 고성능 신경망 추론 엔진인 CTranslate2를 백엔드로 사용하여 원본 Whisper 대비 최대 4배 빠른 속도와 더 적은 메모리 사용량을 달성했다.25 이는 INT8 양자화, 레이어 융합(layer fusion)과 같은 최적화 기술을 적용한 결과로, GPU 기반의 서버 환경에서 높은 처리량을 요구하는 서비스에 널리 사용된다.

- **모델 경량화:**

  - **양자화 (Quantization):** 모델의 가중치를 표현하는 데이터 타입을 32비트 부동소수점(FP32)에서 16비트(FP16)나 8비트 정수(INT8) 등으로 변환하여 모델의 크기를 대폭 줄이고 추론 속도를 향상시키는 기술이다. 한 연구에 따르면, 양자화 기술을 적용함으로써 정확도 손실을 최소화하면서도 모델 크기를 45%, 추론 지연 시간을 19%까지 줄일 수 있음이 확인되었다.37 이는 리소스가 극도로 제한된 엣지 디바이스나 모바일 환경에 Whisper를 배포하는 데 있어 핵심적인 역할을 한다.
  - **Distil-Whisper:** 지식 증류(knowledge distillation) 기법을 활용한 경량화 모델이다. 거대하고 복잡한 '교사 모델'(Whisper `large-v2`)의 지식을 작고 빠른 '학생 모델'에게 전달하는 방식으로 학습시킨다. 그 결과, 원본 모델보다 6배 더 빠르고 49% 더 작으면서도, 성능 저하는 1% 이내로 억제하는 뛰어난 효율성을 달성했다.


Whisper의 핵심적인 기능적 한계(부정확한 타임스탬프, 화자 분리 부재, 실시간 처리 불가)를 해결하기 위한 커뮤니티 주도의 프로젝트들은 Whisper의 활용 범위를 획기적으로 넓혔다.

- `WhisperX`: Whisper의 가장 큰 실용적 약점 중 하나는 타임스탬프가 발화 단위로만 제공되고 그 정확도가 떨어진다는 점이었다. `WhisperX`는 이 문제를 해결하기 위해 음성-텍스트 강제 정렬(forced alignment) 모델(예: `wav2vec2.0`)을 Whisper의 출력에 후처리 단계로 적용한다.49 이를 통해 단어 수준(word-level)의 매우 정밀한 타임스탬프를 생성할 수 있다. 또한, 

  `pyannote-audio`와 같은 오픈소스 화자 분리 라이브러리를 통합하여, 각 단어나 문장이 어떤 화자에 의해 발화되었는지를 식별하는 기능을 추가했다.51 이러한 기능 확장을 통해 Whisper는 단순한 전사 도구를 넘어, 방송 자막 제작, 법률 녹취록 분석, 콜센터 통화 분석 등 전문적인 용도로 활용될 수 있는 강력한 도구로 진화했다.

- `WhisperLive`: Whisper의 실시간 처리 부재라는 한계를 극복하기 위해 개발된 프로젝트이다.50 이 시스템은 실시간으로 들어오는 오디오 스트림을 작은 청크로 나누어 연속적으로 Whisper 모델에 전달하고, VAD 기술을 사용하여 음성이 감지되는 구간만 전사함으로써 불필요한 계산을 줄여 효율성을 높인다. NVIDIA의 TensorRT나 Intel의 OpenVINO와 같은 고성능 추론 백엔드를 지원하여, 라이브 스트리밍 자막 생성이나 실시간 회의 지원 시스템과 같은 낮은 지연 시간이 요구되는 애플리케이션 구축을 가능하게 한다.50

Whisper 생태계의 발전 과정은 오픈소스 프로젝트가 어떻게 스스로의 단점을 보완하고 진화하는지를 보여주는 교과서적인 사례이다. OpenAI가 제공한 강력하지만 일부 기능이 누락된 '엔진'을 커뮤니티가 가져다가, 필요한 '부품'(화자 분리, 실시간 처리 모듈)을 만들어 붙이고, 성능을 극한으로 '튜닝'(최적화, 경량화)하여 완전한 '자동차'로 만들어가는 과정이라 할 수 있다. 이는 중앙 집중식 개발 모델로는 달성하기 어려운 속도와 방향성으로 기술이 발전할 수 있음을 보여주며, 집단 지성의 힘을 명확히 증명한다. 더 나아가, 이는 ASR 기술 스택이 점차 모듈화되고 있음을 시사한다. Whisper는 핵심 '음성 인식 모듈'을, `WhisperX`는 '타임스탬프 및 화자 분리 모듈'을, `WhisperLive`는 '실시간 스트리밍 모듈'을 제공하는 식으로, 개발자들은 마치 레고 블록을 조립하듯 필요한 모듈을 조합하여 자신만의 맞춤형 애플리케이션을 신속하게 구축할 수 있게 되었다.



OpenAI의 Whisper는 대규모 약지도 학습이라는 새로운 훈련 패러다임의 유효성을 성공적으로 입증하며 ASR 연구 및 산업의 지형을 근본적으로 바꾸어 놓았다. 전례 없는 수준의 강건성과 다국어/다중 작업 처리 능력을 단일 종단 간 모델로 구현했으며, 이를 오픈소스로 공개함으로써 고성능 ASR 기술에 대한 접근성을 민주화했다. 이로 인해 학계와 산업계 전반에 걸쳐 폭넓고 깊은 영향을 미쳤다. Whisper는 단순히 기존 모델보다 성능이 조금 더 나은 ASR 시스템이 아니라, ASR 기술을 개발하고, 평가하며, 활용하는 방식 자체를 재정의한 '게임 체인저'로 평가받아 마땅하다.


그러나 Whisper가 모든 문제를 해결한 것은 아니다. 특히 '환각' 현상은 모델의 신뢰성에 심각한 의문을 제기하며, 특히 의료, 법률, 금융과 같이 정확성과 신뢰성이 담보되어야 하는 고위험(high-stakes) 분야에서 Whisper의 광범위한 적용을 가로막는 가장 큰 기술적 장애물로 남아있다. 입력에 존재하지 않는 정보를 생성하는 이 문제는 단순한 성능 개선을 넘어, 모델의 예측 불확실성을 제어하고 강건한 방어 메커니즘을 개발하는 방향의 근본적인 연구를 요구한다. 또한, 훈련 데이터의 불균형에서 비롯되는 저자원 언어에 대한 상대적으로 낮은 성능은 기술의 보편적이고 공평한 확산을 위해 반드시 해결해야 할 중요한 과제이다.


Whisper가 ASR 분야에 남긴 유산은 앞으로 등장할 수많은 후속 기술과 애플리케이션을 통해 계속해서 이어질 것이다. OpenAI는 이미 Whisper v2, v3를 넘어 성능이 더욱 향상된 차세대 오디오 모델을 개발 중임을 시사했으며, 이는 Whisper가 달성한 성과가 끝이 아닌 새로운 시작임을 보여준다.52 미래의 ASR 모델은 Whisper가 구축한 강력한 기반 위에서 다음과 같은 방향으로 발전할 것으로 전망된다.

첫째, 더욱 정교한 문맥 이해 능력을 갖추게 될 것이다. 긴 대화의 전체적인 흐름을 파악하고, 화자의 의도나 미묘한 뉘앙스를 이해하며, 이전 발화 내용을 기억하여 대명사를 정확히 해석하는 등, 인간에 가까운 수준의 대화 이해 능력을 목표로 할 것이다.

둘째, 실시간 상호작용 능력이 강화될 것이다. `WhisperLive`와 같은 프로젝트에서 시작된 실시간 처리 기술은 더욱 발전하여, 낮은 지연 시간과 높은 처리량을 동시에 만족시키며 인간과 기계 간의 자연스러운 음성 기반 상호작용을 가능하게 할 것이다.

셋째, 비언어적 정보 처리 능력이 추가될 것이다. 화자의 감정, 목소리 톤, 말의 빠르기, 억양 등 음성에 담긴 비언어적(para-linguistic) 정보를 분석하여, 단순한 텍스트 변환을 넘어선 풍부한 오디오 인텔리전스를 제공하는 방향으로 진화할 것이다.

Whisper 생태계에서 나타난 기술 스택의 모듈화 경향은 더욱 가속화될 것이다. 핵심 ASR 엔진, 화자 분리 모듈, 감정 분석 모듈, 요약 모듈 등이 독립적으로 개발되고 API를 통해 유연하게 결합되는 환경이 조성되어, 개발자들은 더욱 쉽고 빠르게 혁신적인 음성 기반 AI 애플리케이션을 구축할 수 있게 될 것이다. 결론적으로, Whisper는 ASR 기술 발전사의 중요한 이정표로 기록될 것이며, 음성 기술이 우리 삶의 모든 영역에 깊숙이 통합되는 미래를 앞당기는 데 결정적인 역할을 했다고 평가할 수 있다.


1. A Deep Dive into OpenAI Whisper's Technology - Vatis Tech, 8월 16, 2025에 액세스, https://vatis.tech/blog/a-deep-dive-into-openai-whispers-technology
2. What is OpenAI Whisper? - Gladia, 8월 16, 2025에 액세스, https://www.gladia.io/blog/what-is-openai-whisper
3. Whisper (speech recognition system) - Wikipedia, 8월 16, 2025에 액세스, https://en.wikipedia.org/wiki/Whisper_(speech_recognition_system)
4. Robust Speech Recognition via Large-Scale Weak Supervision - ResearchGate, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/366136304_Robust_Speech_Recognition_via_Large-Scale_Weak_Supervision
5. 3 Best Open-Source ASR Models Compared: Whisper, wav2vec 2.0, Kaldi - Deepgram, 8월 16, 2025에 액세스, https://deepgram.com/learn/benchmarking-top-open-source-speech-models
6. Robust Speech Recognition via Large-Scale Weak ... - OpenAI, 8월 16, 2025에 액세스, https://cdn.openai.com/papers/whisper.pdf
7. arXiv:2212.04356v1 [eess.AS] 6 Dec 2022, 8월 16, 2025에 액세스, https://arxiv.org/pdf/2212.04356
8. ASR Benchmarking: Need for a More Representative Conversational Dataset - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2409.12042v1
9. Whisper - Robust Speech Recognition via Large-Scale Weak Supervision - resoniks, 8월 16, 2025에 액세스, https://www.resoniks.com/post/whisper-robust-speech-recognition-via-large-scale-weak-supervision
10. How to use OpenAI's Whisper for speech recognition - Graphcore, 8월 16, 2025에 액세스, https://www.graphcore.ai/posts/how-to-use-openais-whisper-for-speech-recognition
11. openai/whisper-tiny - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/openai/whisper-tiny
12. Robust Speech Recognition via Large-Scale Weak Supervision - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/papers/2212.04356
13. Introducing Whisper - OpenAI, 8월 16, 2025에 액세스, https://openai.com/index/whisper/
14. openai/whisper: Robust Speech Recognition via Large-Scale Weak Supervision - GitHub, 8월 16, 2025에 액세스, https://github.com/openai/whisper
15. whisper/README.md at main / openai/whisper - GitHub, 8월 16, 2025에 액세스, https://github.com/openai/whisper/blob/main/README.md
16. Host the Whisper Model on Amazon SageMaker: exploring inference options - AWS, 8월 16, 2025에 액세스, https://aws.amazon.com/blogs/machine-learning/host-the-whisper-model-on-amazon-sagemaker-exploring-inference-options/
17. Fine-Tune Whisper For Multilingual ASR with Transformers - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/blog/fine-tune-whisper
18. Whisper : Speech Recognition Model Capable of Recognizing 99 Languages - Medium, 8월 16, 2025에 액세스, https://medium.com/axinc-ai/whisper-speech-recognition-model-capable-of-recognizing-99-languages-5b5cf0197c16
19. Whisper: Functionality and Finetuning | by Okezie Okoye - Medium, 8월 16, 2025에 액세스, https://medium.com/@okezieowen/whisper-functionality-and-finetuning-ba7f9444f55a
20. Making Sense of How Whisper Processes Audio | Giuseppe's Website, 8월 16, 2025에 액세스, https://gattanasio.cc/post/whisper-encoder/
21. openai/whisper-large-v3 - Hugging Face, 8월 16, 2025에 액세스, https://huggingface.co/openai/whisper-large-v3
22. OpenAI Whisper - a neural net for speech to text - AI Projects, 8월 16, 2025에 액세스, https://ai-projects.hashnode.dev/openai-whisper-a-neural-net-for-speech-to-text
23. Speech to text - OpenAI API, 8월 16, 2025에 액세스, https://platform.openai.com/docs/guides/speech-to-text
24. Whisper prompting guide | OpenAI Cookbook, 8월 16, 2025에 액세스, https://cookbook.openai.com/examples/whisper_prompting_guide
25. Accuracy Benchmarks of The Top Free Open Source Speech-to-Text Offerings - Whisper API, 8월 16, 2025에 액세스, https://whisperapi.com/accuracy-benchmarks-top-free-open-source-speech-to-text-offerings
26. Whisper for ASR: All You Need to Know! | by Manish Negi | Medium, 8월 16, 2025에 액세스, https://medium.com/@manishnegi101/whisper-for-asr-all-you-need-to-know-ed0d7853ebb8
27. OpenAI Whisper vs Google Speech-to-Text vs Amazon Transcribe: The ASR Rundown, 8월 16, 2025에 액세스, https://www.gladia.io/blog/openai-whisper-vs-google-speech-to-text-vs-amazon-transcribe
28. OpenAI vs. Google vs. Azure: A Speech-to-Text Battle | by Stoyan Stoitsev - Medium, 8월 16, 2025에 액세스, https://sstoitsev.medium.com/google-vs-azure-a-speech-to-text-battle-f740aa481e8e
29. Introducing ChatGPT and Whisper APIs - OpenAI, 8월 16, 2025에 액세스, https://openai.com/index/introducing-chatgpt-and-whisper-apis/
30. The Whisper model from OpenAI - Azure AI services | Microsoft Learn, 8월 16, 2025에 액세스, https://learn.microsoft.com/en-us/azure/ai-services/speech-service/whisper-overview
31. Whisper model limitation - Microsoft Q&A, 8월 16, 2025에 액세스, https://learn.microsoft.com/en-us/answers/questions/1638419/whisper-model-limitation
32. Evaluating OpenAI's Whisper ASR: Performance analysis across diverse accents and speaker traits | JASA Express Letters | AIP Publishing, 8월 16, 2025에 액세스, https://pubs.aip.org/asa/jel/article/4/2/025206/3267247/Evaluating-OpenAI-s-Whisper-ASR-Performance
33. Evaluating OpenAI's Whisper ASR: Performance analysis across diverse accents and speaker traits. - Apollo, 8월 16, 2025에 액세스, https://www.repository.cam.ac.uk/items/b1b02204-7d9e-4f90-95f3-07619329bf35
34. (PDF) Evaluating OpenAI's Whisper ASR: Performance analysis across diverse accents and speaker traits - ResearchGate, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/378429883_Evaluating_OpenAI's_Whisper_ASR_Performance_analysis_across_diverse_accents_and_speaker_traits
35. Whisper-LM: Improving ASR Models with Language Models for Low-Resource Languages, 8월 16, 2025에 액세스, https://paperswithcode.com/paper/whisper-lm-improving-asr-models-with-language
36. Evaluating OpenAI's Whisper ASR: Performance Analysis Across Diverse Accents and Speaker Traits | Language and Linguistics | Cambridge Open Engage, 8월 16, 2025에 액세스, https://www.cambridge.org/engage/coe/article-details/65490ca0a8b423585a102952
37. [2503.09905] Quantization for OpenAI's Whisper Models: A Comparative Analysis - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/2503.09905
38. Language Log » Psychotic Whisper, 8월 16, 2025에 액세스, https://languagelog.ldc.upenn.edu/nll/?p=66682
39. AI speech-to-text can hallucinate violent language | Cornell Chronicle, 8월 16, 2025에 액세스, https://news.cornell.edu/stories/2024/06/ai-speech-text-can-hallucinate-violent-language
40. Investigation of Whisper ASR Hallucinations Induced by Non-Speech Audio This research was supported by the National Science Centre, Poland under Grant 2021/42/E/ST7/00452, the National Centre for Research and Development, Poland under Grant INFOSTRATEG-IV/0029/2022, and by program "Excellence initiative – research university" - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2501.11378v1
41. Investigation of Whisper ASR Hallucinations Induced by Non-Speech Audio - ResearchGate, 8월 16, 2025에 액세스, https://www.researchgate.net/publication/388232036_Investigation_of_Whisper_ASR_Hallucinations_Induced_by_Non-Speech_Audio
42. The risks of OpenAI's Whisper audio transcription model - Baldur Bjarnason, 8월 16, 2025에 액세스, https://www.baldurbjarnason.com/2024/openai-whisper-risks/
43. [Literature Review] Investigation of Whisper ASR Hallucinations Induced by Non-Speech Audio - Moonlight, 8월 16, 2025에 액세스, https://www.themoonlight.io/en/review/investigation-of-whisper-asr-hallucinations-induced-by-non-speech-audio
44. Understanding Token Limits feeding whisper to gpt-4 - API - OpenAI Developer Community, 8월 16, 2025에 액세스, https://community.openai.com/t/understanding-token-limits-feeding-whisper-to-gpt-4/408995
45. Whisper AI Limitations with Long Audio Files - Prospera Soft, 8월 16, 2025에 액세스, https://prosperasoft.com/blog/openai/whisper-ai/whisper-long-audio-limitations/
46. OpenAI Whisper API Limits: Transcribing Audio File Limits 2024 - Transcribetube, 8월 16, 2025에 액세스, https://www.transcribetube.com/blog/openai-whisper-api-limits
47. Whisper Audio API FAQ - OpenAI Help Center, 8월 16, 2025에 액세스, https://help.openai.com/en/articles/7031512-whisper-audio-api-faq
48. ggml-org/whisper.cpp: Port of OpenAI's Whisper model in C/C++ - GitHub, 8월 16, 2025에 액세스, https://github.com/ggml-org/whisper.cpp
49. Awesome list for Whisper - an open-source AI-powered speech recognition system developed by OpenAI - GitHub, 8월 16, 2025에 액세스, https://github.com/sindresorhus/awesome-whisper
50. collabora/WhisperLive: A nearly-live implementation of OpenAI's Whisper. - GitHub, 8월 16, 2025에 액세스, https://github.com/collabora/WhisperLive
51. WhisperX: Automatic Speech Recognition with Word-level Timestamps (& Diarization) - GitHub, 8월 16, 2025에 액세스, https://github.com/m-bain/whisperX
52. Introducing next-generation audio models in the API - OpenAI, 8월 16, 2025에 액세스, https://openai.com/index/introducing-our-next-generation-audio-models/


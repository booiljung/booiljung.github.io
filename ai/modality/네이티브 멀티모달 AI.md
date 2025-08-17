# 네이티브 멀티모달 AI

## 1. 서론: 멀티모달리티의 패러다임 전환

### 0.1  단일 모달리티 AI의 본질적 한계

인공지능(AI) 기술은 지난 수십 년간 괄목할 만한 발전을 이루었으나, 그 발전은 대부분 특정 데이터 유형, 즉 '모달리티(modality)'에 국한된 형태로 이루어졌다. 자연어 처리(NLP)는 텍스트를, 컴퓨터 비전(CV)은 이미지를, 음성 인식은 오디오를 다루는 등 각 분야는 독립적으로 고도화되었다.1 이러한 단일 모달(unimodal) 접근 방식은 특정 과제 해결에 있어서는 높은 정확도와 효율성을 달성했지만, 인간의 지능이 작동하는 방식과는 근본적인 차이를 보였다. 인간은 세상을 텍스트, 이미지, 소리 등 분절된 정보의 합으로 인식하지 않는다. 오히려 다양한 감각 정보를 통합적으로 받아들이고, 그 속에서 복합적인 문맥을 파악하며 종합적인 판단을 내린다.3

단일 모달 AI의 한계는 바로 이 지점에서 명확히 드러난다. 예를 들어, 텍스트 데이터만을 학습한 대규모 언어 모델(LLM)은 '코끼리'라는 단어의 통계적 관계나 용례는 파악할 수 있지만, 코끼리의 거대한 회색 몸체, 긴 코, 펄럭이는 귀와 같은 시각적 형태나 그 존재 자체를 진정으로 이해하지는 못한다.3 마찬가지로, 컴퓨터 비전 모델은 이미지 속 객체를 분류할 수는 있지만, 그 객체를 둘러싼 이야기나 청각적 분위기를 상상할 수는 없다. 이처럼 각 모달리티가 분리된 상태에서는 데이터의 한 단면만을 보게 되므로, 보다 깊이 있는 이해와 추론, 그리고 인간과 유사한 방식의 상호작용은 원천적으로 불가능했다. 이러한 본질적 한계를 극복하고, 더 풍부한 표현력과 상호작용, 나아가 인간 수준의 종합적 판단 능력을 갖춘 AI를 구현하기 위한 필연적인 다음 단계로서 멀티모달 AI가 부상하게 되었다.3

### 0.2  '네이티브 멀티모달'의 정의와 기술적 의의

멀티모달 AI의 초기 시도는 여러 단일 모달 모델을 기계적으로 연결하는 형태에 가까웠다. 그러나 최근 AI 연구의 패러다임은 '네이티브 멀티모달(Native Multimodal)'이라는 새로운 개념으로 전환되고 있다. 네이티브 멀티모달 모델이란, 단순히 여러 모달리티를 처리하는 것을 넘어, 설계 초기 단계부터 텍스트, 이미지, 오디오, 비디오 등 다양한 데이터 유형을 단일화된 아키텍처 내에서 통합적으로 학습하고 처리하도록 구축된 AI 시스템을 의미한다.1

이 접근법은 기존의 융합 방식과 근본적인 차이를 보인다. 전통적인 멀티모달 모델은 각 모달리티에 특화된 별도의 인코더(encoder)를 사용해 특징을 추출한 뒤, 이 결과물을 모델의 후반부에서 결합하는 '후기 융합(Late Fusion)' 방식이나, 서로 다른 데이터를 단순히 이어 붙여 입력하는 '초기 융합(Early Fusion)' 방식에 의존했다.7 이러한 방식들은 구현이 비교적 용이하지만, 모달리티 간의 깊고 복잡한 상호작용을 학습하는 데 명백한 한계를 가졌다.

반면, Google의 Gemini와 같은 네이티브 멀티모달 모델은 기초 모델(Foundation Model) 단계부터 다양한 모달리티를 함께 학습시킨다.1 이 모델들은 텍스트, 이미지, 오디오, 비디오 등 서로 다른 유형의 데이터가 논리적 순서에 따라 섞여 있는 '인터리빙(interleaved)'된 시퀀스를 직접 입력으로 받아들일 수 있다.13 그 결과, 단 하나의 통합된 모델이 모든 유형의 데이터를 동시에 처리하고, 그 관계 속에서 추론하며, 새로운 형태의 결과물을 생성해낼 수 있게 된다.

이러한 '통합된 학습 프레임워크(integrated learning framework)' 6는 네이티브 멀티모달의 기술적 의의를 명확히 보여준다. 이는 단순히 여러 기능을 합친 것이 아니라, 모달리티 간의 시너지를 통해 1 더하기 1이 2 이상이 되는 효과를 창출한다. 이 패러다임 전환은 AI가 세상을 인식하고 이해하는 방식을 근본적으로 바꾸고 있으며, 단일 모달리티 모델로는 도달할 수 없었던 고차원적인 이해력과 추론 능력의 시대를 열고 있다.7 결국 '네이티브'라는 용어는 단지 기술적 개선을 의미하는 것을 넘어, 지능을 개별 기능의 조합으로 보던 '모듈적' 관점에서 벗어나, 다양한 감각 입력의 동시적 처리 과정에서 지능이 발현된다고 보는 '통합적' 관점으로의 철학적 전환을 상징한다.

## 1.  네이티브 멀티모달 모델의 구조적 원리

네이티브 멀티모달 모델의 강력한 성능은 다양한 데이터를 효과적으로 융합하고 처리하는 정교한 구조적 원리에 기반한다. 이 원리들은 모달리티 융합 전략의 진화, 핵심 아키텍처 구성 요소, 통합된 토큰화, 그리고 교차 어텐션 메커니즘이라는 네 가지 축으로 설명될 수 있다.

### 1.1  모달리티 융합 전략의 진화

멀티모달 AI의 핵심 과제는 서로 다른 형태의 데이터를 어떻게 의미 있는 방식으로 결합하는가에 있다. 이 '융합(fusion)' 전략은 AI의 성능을 결정짓는 중요한 요소이며, 다음과 같은 단계로 진화해왔다.

- **초기 융합 (Early Fusion):** 이 방식은 모델의 가장 앞단, 즉 입력 단계에서 여러 모달리티의 데이터를 결합한다.7 원시 데이터(raw data)를 직접 연결하거나, 각 데이터에서 추출한 저수준 특징(low-level features)을 합쳐 단일 네트워크에 입력하는 구조다.11 초기 융합의 가장 큰 장점은 모달리티 간의 미세한 상관관계를 학습 초기부터 포착할 수 있다는 점이다. 또한, 네트워크가 설계자의 개입 없이 데이터로부터 스스로 특징을 학습할 자유도를 극대화하여, 특정 조건에서는 후기 융합보다 더 높은 성능을 보일 수 있다.11 그러나 이 방식은 각 데이터의 시간적 흐름을 맞추거나(synchronization) 차원을 정렬하는 과정이 복잡하며, 한 모달리티에 포함된 노이즈가 다른 모달리티의 학습 과정에 직접적인 악영향을 미칠 수 있다는 단점이 있다.15
- **후기 융합 (Late Fusion):** 초기 융합과 반대로, 각 모달리티를 독립적인 전문 네트워크를 통해 끝까지 처리한 후, 그 결과(예: 예측 확률)나 고수준 특징(high-level features)을 마지막 단계에서 결합하여 최종 결정을 내린다.7 이 접근법은 각 모달리티에 가장 최적화된 모델을 유연하게 사용할 수 있다는 장점이 있다. 하지만 각 모달리티가 독립적으로 처리되기 때문에, 데이터 간의 깊은 상호작용이나 미묘한 문맥을 학습하기 어려워 성능 향상에 근본적인 한계가 존재한다.11
- **하이브리드/결합 융합 (Hybrid/Joint Fusion):** 네이티브 멀티모달 모델의 핵심은 초기 융합과 후기 융합의 한계를 극복하기 위한 하이브리드 접근법에 있다. 이 전략은 모델의 여러 계층에 걸쳐 점진적으로, 그리고 반복적으로 정보를 융합한다.7 이는 마치 인간이 대화를 하면서 상대방의 표정과 목소리 톤을 지속적으로 참고하여 의미를 파악하는 과정과 유사하다. 예를 들어, DeepMind의 Flamingo 모델은 사전 훈련된 언어 모델 블록과 시각 정보에 주의를 기울이는 교차 어텐션 블록을 번갈아 배치한다.16 이를 통해 텍스트를 처리하는 과정 전반에 걸쳐 시각 정보가 점진적으로 주입되며, 모델의 깊은 수준에서 모달리티 간의 복잡하고 동적인 상호작용이 학습된다. 이처럼 '통제된 통합(controlled integration)' 방식은 초기 융합의 깊은 상호작용 학습 능력과 후기 융합의 안정성을 모두 취하려는 시도로, 네이티브 멀티모달 아키텍처의 정수를 보여준다.

### 1.2  핵심 아키텍처 구성 요소

네이티브 멀티모달 모델은 일반적으로 다음과 같은 핵심 구성 요소들의 유기적인 결합으로 이루어진다.

- **모달리티별 인코더 (Modality-specific Encoders):** 모델의 첫 단계로, 이미지, 오디오, 비디오 등 다양한 형태의 입력 데이터를 각각의 고유한 특성에 맞는 고차원 벡터, 즉 임베딩(embedding)으로 변환하는 역할을 한다.7 이미지의 경우 Vision Transformer(ViT)나 CLIP의 이미지 인코더, NFNet 등이 사용되며, 오디오의 경우 Wav2Vec과 같은 모델이 활용된다.8 특히 Flamingo 모델이 사용한 NFNet-F6 인코더는 이미지 전체가 아닌 하위 영역(sub-regions)에 대한 요약을 생성함으로써, 이미지 내의 미묘한 디테일과 관계를 더 잘 포착할 수 있게 해준다.17
- **입력 프로젝터 (Input Projector) / 리샘플러 (Resampler):** 각기 다른 인코더를 통해 생성된 임베딩들은 서로 다른 차원과 통계적 특성을 갖는다. 입력 프로젝터는 이처럼 이질적인 벡터들을 대규모 언어 모델(LLM)이 이해할 수 있는 공통의 표현 공간(common representation space)으로 투영(projection)하고 정렬(alignment)하는 중요한 다리 역할을 한다.8 Flamingo 모델의 'Perceiver Resampler'는 이 역할의 대표적인 예시로, 길이가 가변적인 대량의 시각 특징 벡터를 고정된 개수의 잠재 벡터(latent vectors)로 효율적으로 압축한다.16 이는 수많은 프레임으로 구성된 비디오나 여러 장의 고해상도 이미지를 처리할 때 발생하는 막대한 계산 비용 문제를 해결하고, LLM에 원활하게 정보를 전달하는 핵심 기술이다.16
- **LLM 백본 (LLM Backbone):** 모델의 '뇌'에 해당하는 부분으로, 통합된 멀티모달 입력을 바탕으로 고차원적인 의미를 추론하고, 문맥을 이해하며, 사용자의 지시를 수행하고, 최종적인 응답을 생성하는 모든 과정을 총괄한다.8 주로 GPT, LLaMA, Chinchilla와 같이 방대한 텍스트 데이터로 사전 훈련된 강력한 대규모 언어 모델이 이 백본 역할을 수행하며, 멀티모달 모델의 지능적 능력의 근간을 이룬다.17

### 1.3  통합된 토큰화 전략

네이티브 멀티모달 모델의 가장 혁신적인 아이디어 중 하나는 모든 종류의 데이터를 '언어' 문제로 치환하는 '통합된 토큰화(Unified Tokenization)' 전략이다.14 이는 텍스트, 이미지 등 근본적으로 다른 형태의 데이터를 동일한 형식의 '토큰(token)'으로 변환하여, 단일 트랜스포머 아키텍처가 처리할 수 있는 하나의 시퀀스로 만드는 것을 의미한다.

이 과정은 다음과 같이 진행된다:

1. **이미지 토큰화 (Image Tokenization):** 이미지를 체스판처럼 작은 패치(patch)들로 나눈다. 그 후 VQ-VAE (Vector Quantised-Variational AutoEncoder)와 같은 양자화 기법을 사용하여 각 패치를 특정 번호를 가진 이산적인(discrete) 토큰으로 변환한다.14 예를 들어, 512x512 크기의 이미지는 1024개의 이미지 토큰 시퀀스로 변환될 수 있다.
2. **텍스트 토큰화 (Text Tokenization):** 텍스트는 기존 LLM에서 널리 사용되는 BPE(Byte-Pair Encoding)와 같은 알고리즘을 통해 단어나 서브워드(subword) 단위의 토큰으로 분해된다.14
3. **통합 시퀀스 생성 (Unified Sequence Generation):** 이렇게 생성된 이미지 토큰과 텍스트 토큰은 하나의 공통 어휘 사전(vocabulary)에 포함된다. 그리고 `<image>`와 같은 특수 제어 토큰을 사용하여, "고양이 사진  이 소파에 앉아 있다"와 같이 텍스트와 이미지 토큰이 뒤섞인(interleaved) 단일 시퀀스로 재구성된다.14

이러한 접근법은 각 모달리티를 위한 별도의 복잡한 인코더 구조 없이, 단일 트랜스포머 모델이 모든 것을 처리하게 함으로써 아키텍처를 극도로 단순화시킨다. 더 중요한 것은, 텍스트와 이미지 간의 관계를 모델의 가장 초기 입력 단계에서부터 매우 긴밀하고 자연스럽게 학습할 수 있게 하여, 모델의 학습 효율성과 추론 성능을 극대화한다는 점이다.14

### 1.4  교차 어텐션 메커니즘: 모달리티 융합의 비밀 소스

만약 통합된 토큰화가 모든 데이터를 같은 언어로 말하게 하는 것이라면, '교차 어텐션(Cross-Attention)' 메커니즘은 그 언어를 사용하여 서로 다른 화자(모달리티)가 의미 있는 대화를 나누게 하는 핵심 기술이다.21 이는 한 모달리티의 정보(Query)를 바탕으로, 다른 모달리티의 정보(Key, Value)에서 현재 맥락에 가장 중요한 부분이 무엇인지를 동적으로 '참조'하고 그 정보를 가져와 결합하는 메커니즘이다.22

Self-Attention이 하나의 문장 내에서 단어들 간의 관계(e.g., "그것"이 무엇을 가리키는지)에 집중하는 반면, Cross-Attention은 텍스트 시퀀스와 이미지 특징 시퀀스처럼 서로 다른 두 시퀀스를 연결하는 강력한 다리 역할을 한다.21 예를 들어, "사진 속 남자가 무엇을 들고 있나요?"라는 질문(텍스트 Query)이 주어지면, 교차 어텐션은 이미지 특징(Key/Value)들 중에서 '남자'와 '들고 있는 행위'에 해당하는 영역에 높은 가중치를 부여하고, 그 시각 정보를 텍스트 기반 추론 과정에 통합하여 "가방을 들고 있습니다"와 같은 답변을 생성하게 한다.21

특히 Flamingo 모델에서 사용된 '게이트형 교차 어텐션(Gated Cross-Attention)'은 이 메커니즘을 한 단계 더 발전시킨 혁신이다.17 이는 학습 가능한 '게이트(gate)'를 추가하여, 시각 정보가 언어 모델에 주입되는 정보의 양을 상황에 맞게 점진적으로 조절하는 역할을 한다. 훈련 초기에는 언어 모델이 갑작스러운 시각 정보의 유입으로 학습에 혼란을 겪을 수 있는데, 게이트는 이 정보의 흐름을 통제하여 훈련 과정을 안정시키고, 모델이 점차적으로 두 모달리티를 융합하는 법을 배우도록 돕는다. 이러한 정교한 제어 메커니즘은 네이티브 멀티모달 모델이 불안정성 문제 없이 깊은 수준의 상호작용을 학습할 수 있게 하는 핵심 동력이다.

## 2.  주요 네이티브 멀티모달 모델 심층 분석

네이티브 멀티모달 AI의 발전사는 몇몇 선구적인 모델들의 등장을 통해 이해할 수 있다. 이 모델들은 각기 다른 철학과 아키텍처를 통해 멀티모달리티의 가능성을 확장해왔으며, 특히 DeepMind의 Flamingo와 Gato, 그리고 Google의 Gemini는 이 분야의 중요한 이정표로 평가받는다. 이들의 아키텍처를 심층적으로 분석함으로써 네이티브 멀티모달 기술의 진화 과정을 추적할 수 있다.

### 2.1  DeepMind Flamingo: 시각-언어 융합의 선구자

Flamingo는 현대적인 네이티브 멀티모달 모델의 청사진을 제시한 선구적인 모델로 평가받는다. 이 모델의 핵심 철학은 '효율성'에 있었다. 즉, 이미 강력한 성능을 입증한 대규모 단일 모달 모델들-Vision Encoder와 Language Model-을 처음부터 다시 훈련하는 대신, 이들을 '고정(frozen)'시킨 상태로 두고, 두 모델을 연결하는 소수의 '브릿지(bridge)' 모듈만을 집중적으로 학습시키는 방식을 채택했다.16 이는 막대한 컴퓨팅 자원을 절약하면서도, CLIP과 같은 모델의 뛰어난 이미지 이해 능력과 대규모 언어 모델(LLM)의 정교한 텍스트 생성 및 추론 능력을 효과적으로 결합하려는 실용적인 시도였다.17

Flamingo의 아키텍처는 네 가지 핵심 요소로 구성된다:

1. **Vision Encoder:** 사전 훈련된 NFNet-F6 모델을 사용하여 입력된 이미지나 비디오 프레임에서 시각적 특징을 추출한다. 이 인코더는 CLIP과 유사하게 이미지-텍스트 쌍 데이터를 이용한 대조 학습(contrastive learning) 방식으로 사전 훈련된 후, Flamingo의 메인 학습 과정에서는 가중치가 변경되지 않도록 고정된다.17
2. **Perceiver Resampler:** Flamingo 아키텍처의 핵심 혁신 중 하나로, Vision Encoder로부터 출력된 가변적인 크기와 개수의 시각 특징 벡터들을 고정된 개수의 잠재 벡터(latent vector)로 효율적으로 압축하는 역할을 한다. 이는 긴 비디오 시퀀스나 여러 장의 이미지를 처리할 때 발생하는 계산적 병목 현상을 해결하고, LLM이 일관된 형식의 시각 정보를 입력받을 수 있도록 한다.16
3. **Language Model (LLM):** DeepMind의 Chinchilla와 같은 강력한 사전 훈련 언어 모델이 백본으로 사용된다. 이 LLM 역시 가중치가 고정된 상태로, 주로 텍스트 기반의 추론과 생성을 담당한다.17
4. **Gated Cross-Attention Layers:** Flamingo의 가장 중요한 '비밀 소스'로, 고정된 LLM의 트랜스포머 블록들 사이에 삽입되는 학습 가능한 레이어다. 이 레이어는 Perceiver Resampler가 압축한 시각 벡터를 텍스트 처리 과정에 점진적으로 융합시키는 역할을 한다. '게이트(gate)' 메커니즘을 통해 시각 정보의 주입량을 동적으로 조절함으로써, LLM이 기존의 언어 능력을 유지하면서도 시각적 문맥을 자연스럽게 이해하도록 돕는다.17

Flamingo는 이러한 독창적인 아키텍처를 통해, 이미지와 텍스트가 뒤섞여(interleaved) 제공되는 프롬프트에 대해 별도의 태스크별 미세조정(fine-tuning) 없이도 뛰어난 소수샷 학습(few-shot learning) 능력을 보여주었다.16 시각적 질의응답(VQA), 이미지 캡셔닝 등 다양한 비전-언어 과제를 성공적으로 수행하며, 효율적인 방식으로 강력한 멀티모달 모델을 구축할 수 있음을 입증했다. 이는 Gemini와 같은 후속 차세대 멀티모달 모델 개발에 중요한 영감을 제공했다.16

### 2.2  DeepMind Gato: '범용 에이전트'의 등장

Flamingo가 시각과 언어의 효율적인 융합에 집중했다면, 같은 연구소에서 발표한 Gato는 그보다 훨씬 더 야심 찬 목표를 제시했다. Gato의 핵심 철학은 언어 모델링에서 입증된 '스케일링 법칙(scaling laws)'을 텍스트의 영역 너머로 확장하여, 세상의 거의 모든 과제를 단 하나의 모델로 해결하려는 '범용 에이전트(Generalist Agent)'의 구현이었다.23 Gato는 단일 신경망과 동일한 가중치 세트(a single network with the same weights)를 사용하여 Atari 게임 플레이, 이미지 캡셔닝, 인간과의 대화, 실제 로봇 팔을 이용한 블록 쌓기 등 수백 가지에 달하는 이질적인 작업을 수행할 수 있다.

Gato의 아키텍처는 모든 것을 '토큰 시퀀스 예측' 문제로 환원하는 급진적인 아이디어에 기반한다:

1. **통합 토큰화 (Universal Tokenization):** Gato의 가장 큰 특징은 서로 다른 모든 종류의 데이터를 공통된 형식의 토큰으로 변환하는 것이다. 이미지 데이터는 작은 패치로 나뉘어 토큰화되고, 텍스트는 서브워드 토큰으로, 로봇 팔의 관절 각도와 같은 연속적인 값은 이산적인 구간으로 나뉘어 토큰화되며, 게임 컨트롤러의 버튼 누름과 같은 이산적인 행동 역시 고유한 토큰으로 표현된다. 이렇게 변환된 모든 토큰들은 하나의 시퀀스로 평탄화되어 모델에 입력된다.23
2. **트랜스포머 기반 아키텍처:** Gato는 GPT-3와 유사한 디코더-온리(decoder-only) 트랜스포머 구조를 채택했다.23 모델의 유일한 목표는 주어진 토큰 시퀀스를 바탕으로 다음에 올 토큰을 예측하는 것이다. 이 단순한 자기회귀(autoregressive) 학습 방식이 모든 과제 수행의 기반이 된다.
3. **컨텍스트 기반 행동 결정:** 모델은 입력된 토큰 시퀀스의 컨텍스트를 통해 현재 자신이 어떤 과제를 수행해야 하는지를 파악한다. 예를 들어, Atari 게임 화면 토큰들이 입력되면 다음 행동으로 버튼 누름 토큰을 예측하고, 대화 텍스트 토큰들이 입력되면 다음 응답 텍스트 토큰을 예측한다. 즉, 다음에 생성할 토큰의 '종류'(텍스트, 관절 토크, 버튼 등)를 스스로 결정하는 것이다.24

Gato는 총 604개의 다양한 태스크에서 해당 분야 전문가의 절반 이상에 해당하는 성능을 달성하며 범용 에이전트의 실현 가능성을 세상에 알렸다.24 그러나 Gato의 접근법은 명확한 한계 또한 보여주었다. 범용성을 확보하는 대가로, 특정 단일 과제에서는 전문화된 모델만큼의 최고 수준(superhuman) 성능을 보여주지는 못했다.23 이는 AI 모델 설계에 있어 '범용성'과 '전문성' 사이의 근본적인 트레이드오프가 존재함을 시사한다. 또한, Gato의 학습은 방대한 양의 전문가 시연 데이터를 모방하는 지도 학습(supervised learning)에 전적으로 의존했으며, 스스로 환경과 상호작용하며 학습하는 강화학습적 요소나 진정한 의미의 자율적 학습 능력은 보여주지 못했다는 비판도 제기되었다.30

### 2.3  Google Gemini: '처음부터 멀티모달'의 구현

Flamingo와 Gato가 각각 효율성과 범용성이라는 측면에서 멀티모달의 지평을 열었다면, Google의 Gemini는 '네이티브 멀티모달' 철학을 가장 순수하게 구현한 모델로 등장했다. Gemini의 핵심 철학은 Flamingo처럼 사전 훈련된 단일 모달 컴포넌트들을 나중에 결합하는 방식이 아니라, 설계 초기부터 이미지, 오디오, 비디오, 텍스트 등 다양한 모달리티 데이터를 하나의 통합된 모델 아키텍처 위에서 함께 훈련(jointly trained)하는 것이다.1 이 접근법은 모달리티 간의 관계를 표층적으로 연결하는 것이 아니라, 모델의 가장 깊은 수준에서부터 근본적인 의미론적 연결을 형성하도록 유도한다.

Gemini는 다양한 요구사항과 컴퓨팅 환경에 대응하기 위해 여러 버전으로 진화해왔다:

- **Gemini 1.0:** 최초 버전으로, 세 가지 크기-고도로 복잡한 작업을 위한 Ultra, 성능과 확장성의 균형을 맞춘 Pro, 그리고 모바일 기기에서의 온디바이스(on-device) 작업을 위한 Nano-로 출시되었다.13 Gemini 1.0은 텍스트, 이미지, 오디오, 비디오가 인터리빙된 입력을 받아 텍스트와 이미지가 혼합된 출력을 생성할 수 있는 능력을 처음으로 선보였다.13
- **Gemini 1.5:** 아키텍처에 MoE(Mixture-of-Experts) 구조를 도입한 것이 가장 큰 특징이다. MoE는 거대한 단일 모델 대신, 특정 작업이나 데이터 유형에 특화된 여러 개의 작은 '전문가' 신경망으로 모델을 구성하고, 입력에 따라 가장 관련 있는 전문가 네트워크만 선택적으로 활성화하는 방식이다. 이를 통해 모델의 전체 크기를 유지하면서도 계산 효율성을 극적으로 높였다.13 또한, 최대 2백만 토큰이라는 전례 없는 크기의 컨텍스트 창(context window)을 도입하여, 수 시간 분량의 비디오 전체나 수백 페이지에 달하는 PDF 문서, 거대한 코드베이스 전체를 한 번에 입력으로 받아 처리하는 경이로운 능력을 확보했다.1
- **Gemini 2.0/2.5:** 추론과 코딩 능력을 더욱 정교하게 다듬고, 3시간 길이의 비디오를 처리하는 등 멀티모달 이해 능력을 극한으로 끌어올린 최신 버전이다.34

Gemini는 이러한 네이티브 아키텍처를 바탕으로, 손으로 어지럽게 쓴 수학 문제 풀이 과정을 시각적으로 이해하고 논리적 오류를 찾아 교정해주거나, 긴 학술 강연 비디오를 요약하고 핵심 내용을 질의응답하는 등 기존 모델들을 뛰어넘는 복합적인 멀티모달 추론 능력을 입증했다.1 이는 다양한 모달리티를 처음부터 함께 학습시키는 '네이티브' 접근법이 모달리티 간의 깊고 풍부한 의미론적 연결을 형성하는 데 가장 효과적인 경로임을 강력하게 시사한다. 이 세 모델의 발전 과정은 AI 아키텍처 설계의 방향성이 어떻게 진화하고 있는지를 명확히 보여준다. Flamingo의 '효율성 우선' 접근, Gato의 '범용성 우선' 접근을 거쳐, Gemini는 막대한 비용을 감수하더라도 '깊은 통합 우선' 접근이 진정한 지능 구현에 더 가깝다는 가설에 베팅하고 있는 것이다.

### 2.4 Table 1: 주요 멀티모달 모델 아키텍처 비교

| **Feature**          | **DeepMind Flamingo**                            | **DeepMind Gato**                              | **Google Gemini (1.5 Pro)**                    |
| -------------------- | ------------------------------------------------ | ---------------------------------------------- | ---------------------------------------------- |
| **Core Philosophy**  | 사전 훈련된 단일 모달 모델들의 효율적 연결       | 범용 에이전트를 위한 보편적 토큰화             | 처음부터 설계된 네이티브 멀티모달              |
| **Base LLM**         | Chinchilla (고정)                                | 커스텀 트랜스포머 (처음부터 학습)              | MoE 기반 커스텀 트랜스포머 (통합 학습)         |
| **Vision Encoder**   | NFNet-F6 (고정)                                  | ResNet 기반 인코더 (처음부터 학습)             | 커스텀 비전 인코더 (통합 학습)                 |
| **Fusion Strategy**  | 고정된 LLM과 시각 특징 간의 게이트형 교차 어텐션 | 단일 트랜스포머 내에서 통합된 토큰 시퀀스 처리 | 인터리빙된 데이터에 대한 엔드투엔드 통합 학습  |
| **Key Innovation**   | Perceiver Resampler & Gated X-Attn               | 행동과 관찰을 단일 시퀀스로 토큰화             | 네이티브 멀티모달 아키텍처, 초대형 컨텍스트 창 |
| **Primary Use Case** | 소수샷 비전-언어 과제 (VQA, 캡셔닝)              | 다중 과제, 다중 환경 제어 (게임, 로보틱스)     | 고도의 복합 멀티모달 추론 및 생성              |

## 3.  네이티브 멀티모달 모델의 응용 및 확장

네이티브 멀티모달 모델의 등장은 단순히 학술적 성과에 그치지 않고, 산업 전반에 걸쳐 AI의 역할을 재정의하며 새로운 응용 분야를 창출하고 있다. 이 모델들은 디지털 콘텐츠를 이해하고 생성하는 방식을 혁신하는 동시에, 로보틱스와 같은 물리적 세계와의 상호작용까지 가능하게 만들며 그 영향력을 확장하고 있다.

### 3.1 Table 2: 네이티브 멀티모달 AI의 주요 응용 분야

| **Domain**                  | **Specific Application / Task**     | **Description**                                              | **Example Technologies / Models** |
| --------------------------- | ----------------------------------- | ------------------------------------------------------------ | --------------------------------- |
| **콘텐츠 이해 및 상호작용** | 고급 시각적 질의응답 (Advanced VQA) | 이미지에 대한 개방형 질문에 분류가 아닌 생성형 답변을 제공   | ViLT, BLIP-2, Gemini 38           |
|                             | 비디오 분석 및 요약                 | 긴 비디오를 처리하여 스크립트, 요약본을 생성하거나 특정 장면을 검색 | Gemini 1.5 1                      |
|                             | 문서 및 데이터 분석                 | 웹페이지, PDF에서 표나 차트를 포함한 구조화된 데이터(JSON)를 추출 및 분석 | Gemini 1                          |
| **콘텐츠 생성**             | 텍스트-이미지/비디오/오디오 생성    | 자연어 설명을 바탕으로 풍부한 미디어 콘텐츠를 창작           | DALL-E, Veo, Runway 2             |
|                             | 상호작용형 콘텐츠 제작              | 사용자 입력에 따라 게임 환경이나 교육 자료를 동적으로 생성   | 42                                |
| **물리적 세계 상호작용**    | 자율주행 (Autonomous Driving)       | 카메라, LiDAR, 레이더 데이터를 통합하여 포괄적인 환경 인식 및 의사결정 수행 | Waymo 1                           |
|                             | 로보틱스 제어 (VLA)                 | 로봇이 자연어 명령을 이해하고 복잡한 물리적 과업을 수행하도록 제어 | PaLM-E, Gemini Robotics, RT-2 44  |
| **비즈니스 및 산업**        | 헬스케어 및 의료 진단               | 의료 영상(X-ray, MRI)과 환자 기록을 함께 분석하여 진단을 보조 | 1                                 |
|                             | 고객 경험 (CX)                      | 텍스트, 음성, 표정에서 고객 감정을 분석하여 서비스 품질 개선 | Zendesk 43                        |
|                             | 보안 및 생체인증                    | 얼굴, 음성, 지문 등 다중 생체 정보를 결합하여 보안 수준 강화 | 4                                 |

### 3.2  고급 시각적 질의응답(VQA) 및 콘텐츠 이해

시각적 질의응답(Visual Question Answering, VQA)은 멀티모달 AI의 능력을 가늠하는 대표적인 척도 중 하나다. 초기 VQA 모델(e.g., ViLT)은 주어진 이미지와 질문에 대해, 미리 정해진 답변 후보군 중에서 가장 적절한 것을 고르는 '분류(classification)' 문제로 접근했다.39 이는 "이미지 속에 고양이가 있습니까?"와 같은 질문에 '예/아니오'로 답하는 수준에 머물렀다.

그러나 네이티브 멀티모달 모델의 발전은 VQA의 패러다임을 근본적으로 바꾸었다. BLIP-2나 Gemini와 같은 최신 모델들은 VQA를 '생성(generative)' 문제로 재정의했다.39 이 모델들은 이미지와 질문을 깊이 있게 이해한 후, 문법적으로 완벽하고 의미적으로 풍부한 자연어 답변을 직접 생성해낸다. "이미지 속 고양이는 무엇을 하고 있나요?"라는 질문에 "창가에 앉아 햇볕을 쬐며 졸고 있습니다"와 같이 상세하고 문맥적인 답변이 가능해진 것이다.

더 나아가, 이러한 콘텐츠 이해 능력은 단순한 질의응답을 넘어 훨씬 더 복잡한 작업으로 확장되고 있다. 예를 들어, Gemini는 웹페이지의 스크린샷을 보고 전체 구조를 파악하여 각 상품의 이름, 저자, 가격, 별점과 같은 정보를 추출해 구조화된 JSON 형식으로 변환할 수 있다.37 또한 수백 페이지에 달하는 PDF 문서 내의 복잡한 표와 차트를 텍스트와 함께 이해하고 분석하는 능력도 보여준다.1 이러한 고도화된 이해 능력은 시각 장애인을 위한 정보 접근성 향상, 학생들을 위한 상호작용형 교육 자료 제작, 기업의 비정형 데이터 분석 등 무한한 활용 가능성을 열어주고 있다.39

### 3.3  멀티모달 콘텐츠 생성의 혁신

네이티브 멀티모달 모델은 콘텐츠를 이해하는 것을 넘어, 새로운 콘텐츠를 창조하는 방식에도 혁명을 일으키고 있다. 과거에는 텍스트를 이미지로(Text-to-Image), 또는 텍스트를 오디오로(Text-to-Audio) 변환하는 단방향 생성이 주를 이루었다. 그러나 이제는 거의 모든 모달리티 입력을 받아 다른 모든 모달리티의 출력을 생성하는 'Any-to-Any' 생성 패러다임이 현실화되고 있다.2 예를 들어, 사용자가 흥얼거린 멜로디(오디오)와 특정 분위기를 묘사한 텍스트를 입력하면, 그에 맞는 배경 음악과 함께 한 편의 짧은 비디오 클립을 생성하는 것이 가능해진다.

이러한 기술은 OpenAI의 DALL-E, Google의 Veo와 같은 모델들을 통해 대중화되고 있으며, 광고, 영화, 게임, 디자인, 교육 등 콘텐츠 산업 전반의 창작 과정을 근본적으로 바꾸고 있다.2 전문 디자이너나 영상 편집자가 아니더라도, 누구나 자연어 프롬프트나 간단한 스케치만으로 고품질의 시각 자료나 비디오 클립을 자동으로 생성할 수 있게 된 것이다.42 이는 창의성의 민주화를 이끌고, 콘텐츠 제작에 필요한 시간과 비용을 획기적으로 절감하여 생산성을 극대화하는 잠재력을 지닌다.

### 3.4  로보틱스 제어와 물리적 세계로의 확장: VLA 모델

네이티브 멀티모달 AI의 가장 야심 찬 확장은 디지털 세계를 넘어 물리적 세계와 직접 상호작용하는 로보틱스 분야에서 이루어지고 있다. 이를 가능하게 하는 핵심 기술이 바로 'VLA(Vision-Language-Action)' 모델이다.9 VLA 모델은 로봇의 시각적 인식(Vision), 인간의 언어 명령 이해(Language), 그리고 물리적 행동 생성(Action)이라는 세 가지 핵심 요소를 단 하나의 엔드투엔드(end-to-end) 신경망 프레임워크로 통합한 것이다.45

VLA 모델의 작동 원리는 Gato의 아이디어를 한층 더 발전시킨 것이다. 로봇의 모든 가능한 행동-예를 들어, 로봇 팔의 7개 관절 각도, 그리퍼(gripper)의 열림/닫힘 상태 등-을 텍스트나 이미지처럼 이산적인 '행동 토큰(action token)'으로 변환한다.47 그 후, 모델은 로봇의 카메라를 통해 입력된 현재의 시각 정보(Vision)와 사용자가 내린 "테이블 위에 있는 빨간색 사과를 집어서 바구니에 담아줘"와 같은 자연어 명령(Language)을 함께 입력받아, 이 과업을 수행하기 위한 일련의 행동 토큰 시퀀스를 예측하고 생성(Action)하도록 학습된다.

이러한 접근법은 미리 프로그래밍된 제한된 동작만 수행할 수 있었던 기존 로봇 제어 시스템의 한계를 뛰어넘는다. Google의 PaLM-E, RT-2, Gemini Robotics와 같은 VLA 모델들은 처음 보는 물체나 이전에 들어보지 못한 복잡한 지시에도 유연하게 대응할 수 있다.45 심지어 인간이 몇 번의 시범을 보여주는 것만으로 새로운 기술을 빠르게 학습(imitation learning)하는 능력까지 보여주며, 진정한 범용 로봇의 등장을 예고하고 있다.44 이처럼 멀티모달 AI는 디지털 지능이 물리적 실체를 갖고 현실 세계의 문제 해결에 직접 기여할 수 있는 경로를 열어주고 있다. 이는 AI 발전의 두 가지 주요 흐름, 즉 디지털 콘텐츠의 창조와 물리적 세계로의 구현이 각기 다른 도전 과제(창의성, 안전성, 신뢰성 등)를 안고 발전해 나갈 것임을 시사한다.

## 4.  당면 과제와 미래 전망

네이티브 멀티모달 AI는 경이로운 가능성을 보여주고 있지만, 그 이면에는 해결해야 할 기술적, 윤리적 과제들이 산적해 있다. 이러한 과제들을 극복하는 과정은 인공일반지능(AGI)을 향한 논쟁과 맞물려 있으며, 향후 AI 기술의 발전 방향을 결정짓는 중요한 변수가 될 것이다.

### 4.1  기술적 한계와 연구 과제

- **데이터 통합 및 정렬 (Data Integration and Alignment):** 멀티모달 모델의 성능은 결국 학습 데이터의 질에 좌우된다. 그러나 텍스트, 이미지, 오디오, 비디오 등 서로 다른 모달리티의 데이터를 의미적으로, 그리고 시간적으로 정확하게 정렬하는 것은 매우 어려운 문제다.51 예를 들어, 비디오의 특정 장면과 그 장면에 해당하는 음성, 자막을 완벽하게 동기화하는 것은 기술적으로 까다롭다. 데이터에 포함된 노이즈나 형식의 불일치는 모델이 모달리티 간의 관계를 잘못 학습하게 만들어 성능 저하의 주된 원인이 된다.15
- **평가 지표의 부재 및 표준화 (Lack and Standardization of Evaluation Metrics):** 텍스트, 이미지, 코드, 로봇 행동 등 다양한 형태의 출력을 생성하는 멀티모달 모델의 성능을 어떻게 공정하고 일관되게 평가할 것인가는 아직 해결되지 않은 중요한 문제다. 텍스트 생성에는 BLEU 점수를, 이미지 생성에는 FID 점수를 사용하는 등 각 모달리티별 평가 지표는 존재하지만, 이들을 종합하여 모델의 전체적인 멀티모달 능력을 평가할 표준화된 벤치마크는 부족한 실정이다.52 Google이 공개한 LMEval과 같은 프레임워크는 다양한 모델과 모달리티를 체계적으로 평가하여 이러한 문제를 해결하려는 시도 중 하나다.53
- **해석 가능성 (Interpretability):** 모델의 규모와 복잡성이 기하급수적으로 증가함에 따라, 모델이 특정 결정을 내린 이유나 내부 작동 메커니즘을 이해하는 것은 거의 불가능에 가까워지고 있다. 이러한 '블랙박스' 문제는 모델의 출력을 신뢰하기 어렵게 만들고, 오류가 발생했을 때 원인을 찾아 수정하는 디버깅 과정을 매우 힘들게 한다.54 특히 모델의 편향성을 탐지하고 완화하기 위해서는 내부 결정 과정을 들여다볼 수 있는 해석 가능성 기술이 필수적이다. 현재 대규모 언어 모델(LLM) 분야에서 활발히 연구되고 있는 해석 가능성 기법들을 멀티모달 모델로 확장하려는 노력이 진행 중이지만, 여러 모달리티가 복잡하게 얽혀 있어 훨씬 더 큰 어려움에 직면해 있다.48

### 4.2  윤리적 딜레마와 완화 전략

기술적 한계보다 더 심각하고 시급한 문제는 네이티브 멀티모달 AI가 야기하는 윤리적 딜레마다. 이 모델들의 강력한 능력은 오용될 경우 사회에 심각한 해를 끼칠 수 있다.

- **편향성 증폭 (Bias Amplification):** 멀티모달 모델은 학습 데이터에 내재된 사회적 편견을 그대로 학습하는 것을 넘어, 여러 모달리티의 편향을 결합하여 더욱 강화하고 증폭시키는 경향이 있다.7 예를 들어, 텍스트 데이터에 존재하는 '의사는 남성, 간호사는 여성'이라는 직업적 성별 편향과, 이미지 데이터에 존재하는 특정 인종에 대한 왜곡된 묘사가 결합될 수 있다.61 실제로 OpenAI의 DALL-E 2 모델에 '승무원(a flight attendant)'을 그려달라고 요청하면 대부분 여성을, 'CEO'를 그려달라고 하면 대부분 백인 남성을 생성하는 사례가 보고된 바 있다.63 이러한 편향된 결과물은 사회적 고정관념을 재생산하고 영속화시키는 심각한 문제를 야기한다.65
- **악의적 콘텐츠 생성 (Malicious Content Generation):** 텍스트, 이미지, 음성을 정교하게 합성하는 능력은 딥페이크(Deepfake) 기술과 결합하여 치명적인 무기가 될 수 있다.4 특정 인물의 얼굴과 목소리를 도용하여 가짜 뉴스, 명예훼손, 정치적 선동, 금융 사기 등 사회적 혼란과 개인에게 돌이킬 수 없는 피해를 주는 악의적 콘텐츠를 누구나 쉽게 대량으로 생성할 수 있는 위험이 현실화되었다.66
- **프라이버시 침해 (Privacy Invasion):** 멀티모달 AI는 작동을 위해 사용자의 얼굴, 음성, 대화 내용, 위치 정보 등 매우 민감한 개인 데이터를 다방면으로 수집하고 통합한다.15 만약 이 데이터가 해킹되거나 유출될 경우, 단일 모달리티 데이터 유출과는 비교할 수 없는 심각한 프라이버시 침해로 이어질 수 있다.4 특히 가정 내에서 작동하는 AI 스피커나 로봇처럼 지속적으로 데이터를 수집하는 기기들은 사용자에게 항상 감시받고 있다는 불안감을 주며, 사적인 영역의 본질을 위협할 수 있다.68

이러한 윤리적 문제를 완화하기 위해서는 기술, 정책, 사회 전반에 걸친 다층적인 노력이 필요하다.

- **데이터 단계:** 편향을 최소화하기 위해 최대한 다양하고 균형 잡힌 데이터셋을 구축하고, 유해하거나 비윤리적인 콘텐츠를 식별하고 필터링하는 기술을 개발해야 한다.15
- **모델링 단계:** 학습 과정에서 편향을 완화하는 알고리즘을 적용하고, Intel의 사례처럼 합성 데이터를 이용하여 특정 편향의 영향을 분리하고 줄이는 연구를 진행할 수 있다.65
- **거버넌스 단계:** AI 개발 및 배포 전 과정에 걸쳐 '책임감 있는 AI(Responsible AI)' 원칙을 내재화하고, 데이터 처리 방침과 모델의 한계를 사용자에게 투명하게 공개해야 한다. 또한, 생성된 콘텐츠에 워터마크를 삽입하는 등 출처를 명확히 하는 기술적 장치도 필요하다.15

### 4.3  인공일반지능(AGI)을 향한 경로: 가능성과 한계

멀티모달 AI의 궁극적인 목표는 종종 인간과 같은 수준의 포괄적 지능을 갖춘 '인공일반지능(Artificial General Intelligence, AGI)'의 구현과 연결된다. 이 주제에 대해서는 기대와 회의가 첨예하게 엇갈린다.

- **긍정적 관점:** 텍스트, 시각, 청각 등 여러 감각 정보를 통합적으로 처리하는 멀티모달 능력은 인간 지능의 핵심적인 특징이므로, 멀티모달 AI는 AGI로 가는 가장 유력한 경로로 간주된다.71 Gato와 같이 단일 모델로 게임, 대화, 로봇 제어 등 전혀 다른 종류의 과업을 수행하는 범용 에이전트의 등장은 이러한 가능성을 뒷받침하는 강력한 증거로 여겨진다.23
- **비판적 관점:** 그러나 현재의 멀티모달 모델이 진정한 '이해'에 도달했는지에 대해서는 근본적인 회의론이 존재한다.
  1. **실세계 이해의 부재:** 현재 모델들은 방대한 양의 디지털 데이터를 학습했지만, 물리적 세계와 직접 상호작용하며 얻는 '체화된 경험(embodied experience)'이 전무하다.72 그들은 '사과'가 둥글고 빨갛다는 텍스트와 이미지의 연관성은 학습할 수 있지만, 사과를 손에 쥐었을 때의 무게감이나 한 입 베어 물었을 때의 아삭한 식감과 단맛은 알지 못한다. 이러한 현실 기반의 직관 없이는 진정한 의미의 이해가 불가능하다는 주장이다. 노암 촘스키가 제시한 "Colorless green ideas sleep furiously(색 없는 녹색 생각들이 맹렬하게 잔다)"라는 문장처럼, 현재 모델들은 문법적으로 완벽하지만 의미론적으로는 공허한 결과물을 생성할 수 있다.72
  2. **구조적 한계:** 현재의 멀티모달 아키텍처는 여전히 각 모달리티를 독립적인 모듈로 인코딩한 후, 이를 잠재 공간(latent space)에서 억지로 합치는 방식에 가깝다. 이는 진정한 의미의 '통합 지능'이라기보다는, 여러 전문 기능들을 단순히 묶어놓은 '기능의 집합'에 불과하며, 이 구조로는 예측하지 못한 새로운 상황에 유연하게 대처하는 일반화 능력에 한계가 있을 수밖에 없다는 비판이다.72

결론적으로, AGI는 단순히 더 많은 종류의 데이터를 더 큰 모델에 학습시키는 스케일링만으로는 도달할 수 없으며, 물리적 세계와의 상호작용을 통한 개념 형성, 인과관계 추론, 자율적 목표 설정 등 현재 모델에는 없는 근본적인 구조적 혁신이 필요하다는 주장이 설득력을 얻고 있다.

### 4.4  향후 발전 방향

이러한 과제와 논쟁 속에서 네이티브 멀티모달 AI는 다음과 같은 방향으로 진화할 것으로 전망된다.

- **모달리티의 확장:** 현재의 시각, 청각, 텍스트를 넘어, 촉각, 후각, 심지어 뇌파와 같은 새로운 감각 정보를 통합하려는 연구가 시도될 것이다.1 이는 AI가 세상을 더욱 풍부하고 입체적으로 이해하게 만들 것이다.
- **에이전트형 AI로의 진화:** 단순한 질의응답 시스템을 넘어, 복잡한 목표를 스스로 설정하고, 장기적인 계획을 수립하며, 도구를 사용하여 자율적으로 과업을 수행하는 '에이전트(Agent)'로 발전할 것이다.1 Google의 'Project Astra'는 사용자의 주변 환경을 실시간으로 인식하고 기억하며, 맥락에 맞는 도움을 제공하는 미래형 AI 에이전트의 비전을 보여주는 대표적인 사례다.34
- **물리적 구현의 고도화:** AI의 지능이 로봇의 신체와 결합하여, 정보 처리를 넘어 인간처럼 물리적 환경 속에서 움직이고, 물건을 다루며, 실제 문제를 해결하는 방향으로 나아갈 것이다.73 결국 AI 발전의 최종 목적지 중 하나는 신뢰할 수 있고 안전하며 유용한 물리적 조력자를 만드는 것이 될 것이다. 이 과정에서 '해석 가능성-신뢰-안전성'이라는 세 가지 요소의 확보가 기술 발전의 속도를 결정하는 가장 중요한 열쇠가 될 것이다.

## 5.  결론 및 제언

### 5.1  네이티브 멀티모달 모델의 현재와 미래

네이티브 멀티모달 AI는 기존 단일 모달리티의 명백한 한계를 극복하고, 인간의 지능이 작동하는 방식에 한 걸음 더 다가선 중요한 기술적 이정표다. 설계 단계부터 다양한 데이터 유형을 통합적으로 처리하는 '처음부터 멀티모달(multimodal from the ground up)'이라는 아키텍처 철학은 AI의 패러다임을 근본적으로 바꾸고 있다. 통합된 토큰화 전략, 교차 어텐션 메커니즘, 그리고 점진적 융합과 같은 핵심 기술들은 AI가 디지털 콘텐츠를 이해하고 생성하는 방식을 혁신하는 것을 넘어, 로보틱스를 통해 물리적 세계와 직접 상호작용하는 새로운 가능성을 열었다.

Flamingo의 효율적인 융합 시도에서부터 Gato의 범용 에이전트 개념, 그리고 Gemini의 완전한 네이티브 통합에 이르기까지, 이 분야는 놀라운 속도로 발전해왔다. 그 결과, 이제 AI는 복잡한 시각적 추론, 장시간의 비디오 분석, 심지어 자연어 명령에 기반한 로봇 제어까지 수행할 수 있는 수준에 도달했다.

그러나 이러한 눈부신 발전의 이면에는 기술적, 윤리적 도전 과제들이 명확하게 존재한다. 데이터 정렬의 어려움, 공정한 평가 지표의 부재, 그리고 모델 내부를 들여다볼 수 없는 해석 가능성의 문제는 기술의 신뢰성과 안정성을 확보하기 위해 반드시 해결해야 할 과제다. 더욱이, 학습 데이터에 내재된 편향이 증폭되는 문제, 딥페이크와 같은 악의적 콘텐츠 생성의 위협, 그리고 민감한 개인정보의 집적으로 인한 프라이버시 침해 위험은 기술의 사회적 수용을 가로막는 심각한 장애물이다. 결국 네이티브 멀티모달 AI의 미래는 이러한 기술적 난제와 윤리적 딜레마를 얼마나 책임감 있게 해결하느냐에 달려있다.

### 5.2  연구 개발 및 산업 적용을 위한 제언

네이티브 멀티모달 AI 기술이 인류에게 긍정적인 방향으로 기여하기 위해서는 관련 주체들의 전략적인 접근과 책임감 있는 노력이 요구된다.

- **연구자를 위한 제언:** 현재의 성능 경쟁을 넘어, 모델의 **해석 가능성, 공정성, 그리고 견고성(robustness)**을 높이는 데 연구 역량을 집중해야 한다. 특히, 모델이 왜 그러한 결정을 내렸는지 설명할 수 있는 새로운 아키텍처나 학습 방법을 개발하는 것이 시급하다. 또한, 다양한 모달리티와 태스크를 포괄하는 표준화된 평가 프로토콜과 벤치마크를 구축하여, 모델의 성능과 한계를 객관적으로 비교하고 검증할 수 있는 기반을 마련해야 한다.
- **개발자 및 기업을 위한 제언:** 멀티모달 AI 기술을 상용 서비스에 도입하기에 앞서, 발생 가능한 **윤리적 리스크(편향, 프라이버시, 오용 가능성)를 철저히 평가하고, 이를 완화하기 위한 기술적/정책적 안전장치를 설계 단계부터 내재화**해야 한다. '책임감 있는 AI(Responsible AI)' 원칙을 구호에 그치지 않고, 데이터 수집, 모델 개발, 테스트, 배포, 사후 관리 등 제품의 전체 수명 주기에 걸쳐 적용하는 문화를 정착시켜야 한다. 또한, 사용자에게 모델의 능력과 한계를 투명하게 공개하고, 데이터 사용에 대한 명확한 동의를 얻는 절차를 확립하는 것이 필수적이다.
- **정책 입안자를 위한 제언:** 딥페이크를 이용한 여론 조작, 개인정보의 무분별한 통합 및 활용 등 멀티모달 AI 기술이 야기할 수 있는 새로운 사회적 문제에 선제적으로 대응하기 위한 **법적, 제도적 프레임워크를 사회적 합의를 통해 논의하고 구축**해야 한다. 기술의 발전을 저해하지 않으면서도 잠재적 위험을 효과적으로 통제할 수 있는 유연하고 적응적인 규제 접근법이 필요하다. 이를 위해 기술 전문가, 기업, 시민 사회, 정부 간의 긴밀한 협력 채널을 구축하고, 기술의 영향에 대한 지속적인 사회적 논의를 촉진해야 할 것이다.

#### **참고 자료**

1. 멀티모달 AI란? – GPT-4, Gemini 등 사례 분석 - Trend Now - 티스토리, accessed July 12, 2025, https://trendnow-1.tistory.com/49
2. 멀티모달이란? 정의, 장점, 데이터, 활용 방법 | appen 에펜, accessed July 12, 2025, https://kr.appen.com/blog/multimodal/
3. 멀티모달(Multi Modal)AI와 기존 인공지능의 차이점 - 클루닉스, accessed July 12, 2025, https://www.clunix.com/insight/it_trends.php?boardid=ittrend&mode=view&idx=824
4. 멀티 모달 AI란 무엇입니까? 실제 활용 사례 분석, accessed July 12, 2025, [https://hblabgroup.com/ko/%EB%A9%80%ED%8B%B0-%EB%AA%A8%EB%8B%AC-ai%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9E%85%EB%8B%88%EA%B9%8C-%EC%8B%A4%EC%A0%9C-%ED%99%9C%EC%9A%A9-%EC%82%AC%EB%A1%80-%EB%B6%84%EC%84%9D/](https://hblabgroup.com/ko/멀티-모달-ai란-무엇입니까-실제-활용-사례-분석/)
5. 멀티 모달(Multi Modal) 딥러닝 - 공부하고 또 공부하는 - 티스토리, accessed July 12, 2025, https://sorrow16.tistory.com/224
6. ShapeLLM-Omni: 3D 생성 및 이해를 위한 네이티브 멀티모달 LLM ..., accessed July 12, 2025, https://m.hanbit.co.kr/channel/view.html?cmscode=CMS4788566740
7. 멀티 모달 AI 모델: AI 기능 확장하기 | 울트라 애널리틱스 - Ultralytics, accessed July 12, 2025, https://www.ultralytics.com/ko/blog/multi-modal-models-and-multi-modal-learning-expanding-ais-capabilities
8. Multimodal LLM(Multimodal Large Language Model) 에 대해 알아보겠습니다. - Feccle 의 IT자료 모음 - 티스토리, accessed July 12, 2025, https://feccle.tistory.com/m/434
9. 멀티모달 모델 - 나무위키, accessed July 12, 2025, [https://namu.wiki/w/%EB%A9%80%ED%8B%B0%EB%AA%A8%EB%8B%AC%20%EB%AA%A8%EB%8D%B8](https://namu.wiki/w/멀티모달 모델)
10. Aria: 최신 오픈소스 멀티모달 네이티브 MoE 모델 - 인공지능 활용 정보 공유, accessed July 12, 2025, [https://fornewchallenge.tistory.com/entry/%F0%9F%8C%9FAria-%EC%B5%9C%EC%8B%A0-%EC%98%A4%ED%94%88%EC%86%8C%EC%8A%A4-%EB%A9%80%ED%8B%B0%EB%AA%A8%EB%8B%AC-%EB%84%A4%EC%9D%B4%ED%8B%B0%EB%B8%8C-MoE-%EB%AA%A8%EB%8D%B8](https://fornewchallenge.tistory.com/entry/🌟Aria-최신-오픈소스-멀티모달-네이티브-MoE-모델)
11. 전단 융합 기반 멀티모달 심층학습을 이용한 손동작 분류 - KIEE - The ..., accessed July 12, 2025, http://journal.auric.kr/kiee/XmlViewer/f408876
12. KR20240100864A - 멀티모달 데이터 융합 기반의 감정인식 시스템 및, accessed July 12, 2025, https://patents.google.com/patent/KR20240100864A/ko
13. What is Google Gemini? | IBM, accessed July 12, 2025, https://www.ibm.com/think/topics/google-gemini
14. Meta, 다양한 모달리티에서 더 뛰어난 성능을 제공하는 융합 모델 ..., accessed July 12, 2025, https://discuss.pytorch.kr/t/meta-chameleon/4410
15. Navigating the Challenges of Multimodal AI Data Integration - Cogito Tech, accessed July 12, 2025, https://www.cogitotech.com/blog/navigating-the-challenges-of-multimodal-ai-data-integration/
16. Understanding DeepMind's Flamingo Visual Language Models | by ..., accessed July 12, 2025, https://medium.com/@paluchasz/understanding-flamingo-visual-language-models-bea5eeb05268
17. Flamingo - Intuitively and Exhaustively Explained | Towards Data Science, accessed July 12, 2025, https://towardsdatascience.com/flamingo-intuitively-and-exhaustively-explained-bf745611238b/
18. Understanding Flamingo: A Deep Dive into Its Vision-Language ..., accessed July 12, 2025, https://medium.com/@nishantparmar/understanding-flamingo-a-deep-dive-into-its-vision-language-architecture-and-real-world-outputs-d2ffe066b36c
19. Flamingo - Intuitively and Exhaustively Explained | Towards Data ..., accessed July 12, 2025, https://towardsdatascience.com/flamingo-intuitively-and-exhaustively-explained-bf745611238b
20. [논문 리뷰] Multimodal Learning with Transformers: A Survey - velog, accessed July 12, 2025, [https://velog.io/@sksmslhy/%EB%85%BC%EB%AC%B8-%EB%A6%AC%EB%B7%B0-Multimodal-Learning-with-Transformers-A-Survey](https://velog.io/@sksmslhy/논문-리뷰-Multimodal-Learning-with-Transformers-A-Survey)
21. Why Cross-Attention is the Secret Sauce of Multimodal Models | by ..., accessed July 12, 2025, https://medium.com/@jakubstrawadev/why-cross-attention-is-the-secret-sauce-of-multimodal-models-f8ec77fc089b
22. Cross Attention Module, accessed July 12, 2025, [https://vds.sogang.ac.kr/wp-content/uploads/2023/01/2022%ED%95%98%EA%B3%84%EC%84%B8%EB%AF%B8%EB%82%98_%EC%9C%A0%ED%98%84%EC%9A%B0.pdf](https://vds.sogang.ac.kr/wp-content/uploads/2023/01/2022하계세미나_유현우.pdf)
23. Gato (DeepMind) - Wikipedia, accessed July 12, 2025, https://en.wikipedia.org/wiki/Gato_(DeepMind)
24. DeepMind's Gato: A step towards General AI | by Rukaiya Hasan - Medium, accessed July 12, 2025, https://medium.com/@rukaiya.rk24/deepminds-gato-a-step-towards-general-ai-a1ca77d5290e
25. A Generalist Agent, accessed July 12, 2025, http://arxiv.org/pdf/2205.06175
26. [2205.06175] A Generalist Agent - arXiv, accessed July 12, 2025, https://arxiv.org/abs/2205.06175
27. OrigamiDream/gato: Unofficial Gato: A Generalist Agent - GitHub, accessed July 12, 2025, https://github.com/OrigamiDream/gato
28. A Generalist Agent - OpenReview, accessed July 12, 2025, https://openreview.net/forum?id=1ikK0kHjvj
29. [R] Deepmind's Gato: a generalist learning agent : r/MachineLearning - Reddit, accessed July 12, 2025, https://www.reddit.com/r/MachineLearning/comments/uo6bgn/r_deepminds_gato_a_generalist_learning_agent/
30. Gato – A Generalist Agent - Hacker News, accessed July 12, 2025, https://news.ycombinator.com/item?id=31415478
31. Gemini: A Family of Highly Capable Multimodal Models - arXiv, accessed July 12, 2025, http://arxiv.org/pdf/2312.11805
32. Gemini Nano - Google DeepMind, accessed July 12, 2025, https://deepmind.google/models/gemini/nano/
33. Gemini 1.5: Unlocking multimodal understanding across millions of tokens of context - arXiv, accessed July 12, 2025, https://arxiv.org/abs/2403.05530
34. 제미나이 2.0 출시: 에이전트 시대를 위한 구글의 새로운 AI 모델 - Google Blog, accessed July 12, 2025, https://blog.google/intl/ko-kr/company-news/technology/gemini-2-0-kr/
35. Gemini 2.5: Pushing the Frontier with Advanced Reasoning, Multimodality, Long Context, and Next Generation Agentic Capabilities. - arXiv, accessed July 12, 2025, https://arxiv.org/html/2507.06261v1
36. Gemini 2.5: Pushing the Frontier with Advanced Reasoning, Multimodality, Long Context, and Next Generation Agentic Capabilities. - Googleapis.com, accessed July 12, 2025, https://storage.googleapis.com/deepmind-media/gemini/gemini_v2_5_report.pdf
37. Gemini의 멀티모달 기능 사용 사례 7가지, accessed July 12, 2025, https://developers.googleblog.com/ko/7-examples-of-geminis-multimodal-capabilities-in-action/
38. KVQA(Korean Visual Question Answering) 대규모 데이터셋, accessed July 12, 2025, https://www.testworks.co.kr/pdf/library/pdf_2_1_ko.pdf
39. 시각적 질의응답 (Visual Question Answering) - Hugging Face, accessed July 12, 2025, https://huggingface.co/docs/transformers/ko/tasks/visual_question_answering
40. 멀티모달 AI, accessed July 12, 2025, https://cloud.google.com/use-cases/multimodal-ai?hl=ko
41. 오픈AI를 비롯한 빅테크 기업의 멀티모달 AI 개발 현황, accessed July 12, 2025, https://www.koti.re.kr/common/file/download.do?atch_no=dp3JJLnQXXQZJ7YLdxRq9g%3D%3D
42. 멀티모달 생성 AI(Multimodal Generative AI) - 기억을 지배하는 기록, accessed July 12, 2025, [https://bigdown.tistory.com/entry/%EB%A9%80%ED%8B%B0%EB%AA%A8%EB%8B%AC-%EC%83%9D%EC%84%B1-AIMultimodal-Generative-AI](https://bigdown.tistory.com/entry/멀티모달-생성-AIMultimodal-Generative-AI)
43. 멀티모달 AI란? 정의부터 적용 분야까지 한눈에 - Elice, accessed July 12, 2025, https://elice.io/ko/newsroom/multimodal-ai-guide
44. Daily Report : Google, 자연어로 로봇을 제어하는 멀티모달 시스템 특허 출원, accessed July 12, 2025, [https://www.trendbird.biz/entry/Daily-Report-Google-%EC%9E%90%EC%97%B0%EC%96%B4%EB%A1%9C-%EB%A1%9C%EB%B4%87%EC%9D%84-%EC%A0%9C%EC%96%B4%ED%95%98%EB%8A%94-%EB%A9%80%ED%8B%B0%EB%AA%A8%EB%8B%AC-%EC%8B%9C%EC%8A%A4%ED%85%9C-%ED%8A%B9%ED%97%88-%EC%B6%9C%EC%9B%90](https://www.trendbird.biz/entry/Daily-Report-Google-자연어로-로봇을-제어하는-멀티모달-시스템-특허-출원)
45. 멀티모달 개념과 활용, 그리고 레터웍스의 AI 기술, accessed July 12, 2025, https://www.letr.ai/ko/blog/241202
46. [2503.20020] Gemini Robotics: Bringing AI into the Physical World - arXiv, accessed July 12, 2025, https://arxiv.org/abs/2503.20020
47. Vision-Language-Action Models: Concepts, Progress, Applications and Challenges - arXiv, accessed July 12, 2025, https://arxiv.org/html/2505.04769v1
48. Multimodal Generative Models: Unification, Planning Agents, Evaluation, accessed July 12, 2025, http://lxmls.it.pt/2024/slides/mohit.pdf
49. arxiv.org, accessed July 12, 2025, [https://arxiv.org/html/2505.04769v1#:~:text=VLA%20models%20are%20multimodal%20artificial,generation%20into%20a%20single%20framework.](https://arxiv.org/html/2505.04769v1#:~:text=VLA models are multimodal artificial,generation into a single framework.)
50. [2507.01925] A Survey on Vision-Language-Action Models: An Action Tokenization Perspective - arXiv, accessed July 12, 2025, https://arxiv.org/abs/2507.01925
51. 멀티모달 AI란 무엇인가요? - IBM, accessed July 12, 2025, https://www.ibm.com/kr-ko/think/topics/multimodal-ai
52. AI 모델 정확도 평가, 제조업 성공 사례로 본 핵심 지표와 실무 가이드, accessed July 12, 2025, https://www.impactive-ai.com/insight/what-you-need-to-know-when-evaluating-model-accuracy
53. Google이 언어 및 멀티모달 모델을 벤치마크하기 위해 오픈 소스 ..., accessed July 12, 2025, https://tilnote.io/news/6834e53683c72d1d253e412b
54. A Survey on Mechanistic Interpretability for Multi-Modal Foundation Models - arXiv, accessed July 12, 2025, https://arxiv.org/html/2502.17516v1
55. Modular and Interpretable Multimodal AI for Improved Generation and Evaluation | Dept. of Computer Science and Engineering, SNU, accessed July 12, 2025, https://cse.snu.ac.kr/en/community/seminar/1092
56. NeurIPS Poster A Concept-Based Explainability Framework for Large Multimodal Models, accessed July 12, 2025, https://neurips.cc/virtual/2024/poster/95482
57. A Concept-Based Explainability Framework for Large Multimodal Models - OpenReview, accessed July 12, 2025, [https://openreview.net/forum?id=MvjLRFntW6&referrer=%5Bthe%20profile%20of%20Matthieu%20Cord%5D(%2Fprofile%3Fid%3D~Matthieu_Cord1)](https://openreview.net/forum?id=MvjLRFntW6&referrer=[the+profile+of+Matthieu+Cord](/profile?id%3D~Matthieu_Cord1))
58. NeurIPS Poster Scale Alone Does not Improve Mechanistic Interpretability in Vision Models, accessed July 12, 2025, https://neurips.cc/virtual/2023/poster/71812
59. What are some ethical concerns in multimodal AI systems? - Milvus, accessed July 12, 2025, https://milvus.io/ai-quick-reference/what-are-some-ethical-concerns-in-multimodal-ai-systems
60. What Is Multimodal AI and How It Works - IMD Business School, accessed July 12, 2025, https://www.imd.org/blog/digital-transformation/multimodal-ai/
61. Who Is to Blame for the Bias in Visualizations, ChatGPT or DALL-E? - MDPI, accessed July 12, 2025, https://www.mdpi.com/2673-2688/6/5/92
62. Measuring Bias in Multimodal Models: Multimodal Composite Association Score - DORAS | DCU Research Repository, accessed July 12, 2025, [https://doras.dcu.ie/28902/7/Pages%20from%20978-3-031-37249-0-2.pdf](https://doras.dcu.ie/28902/7/Pages from 978-3-031-37249-0-2.pdf)
63. [김동원의 Eye-T] AI가 편향성 문제를 극복하는 방법 - AI타임스, accessed July 12, 2025, https://www.aitimes.com/news/articleView.html?idxno=144682
64. Perpetuation of Gender Bias in Visual Representation of Professions in the Generative AI Tools DALL/E and Bing Image Creator - MDPI, accessed July 12, 2025, https://www.mdpi.com/2076-0760/13/5/250
65. DALL/E 2 pre-training mitigations | OpenAI, accessed July 12, 2025, https://openai.com/index/dall-e-2-pre-training-mitigations/
66. 멀티모달 데이터 구축의 전망 / 블로그 - 데이터메이커, accessed July 12, 2025, https://www.datamaker.io/blog/posts/128
67. What are some ethical concerns in multimodal AI systems? - Zilliz Vector Database, accessed July 12, 2025, https://zilliz.com/ai-faq/what-are-some-ethical-concerns-in-multimodal-ai-systems
68. Ethical considerations for integrating multimodal computer perception and neurotechnology, accessed July 12, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10904467/
69. Ethical considerations for integrating multimodal computer perception and neurotechnology, accessed July 12, 2025, https://pubmed.ncbi.nlm.nih.gov/38435745/
70. 사례 연구: 인텔 실험실, 기본 멀티모달 모델에서 AI 편향 20% 완화, accessed July 12, 2025, https://www.intel.co.kr/content/www/kr/ko/content-details/830947/case-study-intel-labs-mitigates-ai-bias-in-foundational-multimodal-models-by-20-percent.html
71. AGI Model - Intro | leeandcat, accessed July 12, 2025, https://www.leeandcat.com/single-post/agi-model
72. AGI는 멀티모달 모델로 실현될 수 없는 이유 - Acloud Blog!, accessed July 12, 2025, [https://blog.a-cloud.co.kr/2025/06/25/agi%EB%8A%94-%EB%A9%80%ED%8B%B0%EB%AA%A8%EB%8B%AC-%EB%AA%A8%EB%8D%B8%EB%A1%9C-%EC%8B%A4%ED%98%84%EB%90%A0-%EC%88%98-%EC%97%86%EB%8A%94-%EC%9D%B4%EC%9C%A0/](https://blog.a-cloud.co.kr/2025/06/25/agi는-멀티모달-모델로-실현될-수-없는-이유/)
73. 싱글모달과 멀티모달의 차이점과 방향성 - 브런치, accessed July 12, 2025, https://brunch.co.kr/@b2439ea8fc654b8/71
# 이미지 편집을 위한 어텐션 기반 확산 트랜스포머(ADiT)

## 서론: 생성 모델의 패러다임 전환, UNet에서 트랜스포머로

### 0.1 초기 혼동 해소 및 보고서 초점 명확화

'ADiT 모델'이라는 용어는 초기에 여러 대상을 지칭할 수 있는 모호성을 내포하고 있다. 공개적으로 알려진 인물 중에는 패션 모델 아딧 프리실라(Adit Priscilla) 1와 인도 배우 아디티 라오 히다리(Aditi Rao Hydari) 6가 존재한다. 그러나 본 보고서의 기반이 되는 연구 자료들의 기술적 특성을 면밀히 분석한 결과, 탐구의 핵심 대상은 인공지능(AI) 이미지 생성 및 편집 분야의 혁신적인 아키텍처인 

**확산 트랜스포머(Diffusion Transformer, DiT)**, 특히 어텐션 메커니즘을 기반으로 한 모델임이 명백하다.7 따라서 본 보고서는 AI 모델로서의 ADiT, 즉 DiT에 대한 심층 분석에 전적으로 초점을 맞출 것이다.

### 0.2 DiT 이전 시대: UNet 아키텍처의 지배와 그 한계

DiT의 등장을 이해하기 위해서는 먼저 그 이전 시대를 지배했던 아키텍처를 살펴볼 필요가 있다. 스테이블 디퓨전(Stable Diffusion)과 같은 초기 및 주류 확산 모델들은 대부분 **UNet**이라는 컨볼루션 신경망(CNN) 기반 아키텍처에 의존했다.12 UNet은 인코더-디코더 구조에 스킵 커넥션(skip connection)을 추가하여 이미지의 다중 스케일 특징을 효과적으로 학습하는 데 강점을 보였으며, 이는 이미지 생성 분야에서 큰 성공을 거두는 기반이 되었다.

그러나 UNet의 컨볼루션 기반 특성은 본질적인 한계를 내포하고 있었다. 가장 큰 한계는 이미지 전반에 걸친 **장거리 의존성(long-range dependencies)**을 효과적으로 포착하는 데 어려움을 겪는다는 점이다.10 컨볼루션 연산은 본질적으로 지역적인 픽셀 그룹(local receptive fields)에 집중하기 때문에, 이미지의 한쪽 끝에 있는 객체가 다른 쪽 끝에 있는 객체와 어떻게 상호작용하는지를 이해하는 데 비효율적이다. 이러한 한계는 특히 고해상도 이미지를 생성하거나 복잡한 구조적 변경이 필요한 편집 작업을 수행할 때 전역적 일관성(global coherence)을 저해하는 요인으로 작용했다. 결과적으로, 상당한 수준의 형태 인식 수정(shape-aware modifications)이 요구되는 작업에서 UNet 기반 모델들은 종종 부자연스러운 결과를 낳거나 구조적 무결성을 유지하는 데 실패했다.15

### 0.3 트랜스포머의 당위성: 확산을 위한 새로운 백본

UNet의 근본적인 한계에 대한 해답으로 등장한 것이 바로 **확산 트랜스포머(DiT)**이다.16 DiT의 핵심 아이디어는 확산 모델의 UNet 백본을 잠재 공간 패치(latent patches)에 대해 작동하는 트랜스포머로 대체하는 것이다. 이러한 접근은 단순히 아키텍처를 교체하는 것을 넘어, 생성 모델링의 근본적인 패러다임을 전환하는 시도였다. 트랜스포머가 확산 모델의 새로운 백본으로 채택된 이유는 다음과 같은 명백한 장점 때문이다.

- **우월한 확장성(Superior Scaling Properties):** DiT 관련 연구는 모델의 연산 복잡도(Gflops)와 성능 사이의 명확한 상관관계를 입증했다. 트랜스포머의 깊이나 너비를 늘리거나 입력 토큰의 수를 증가시켜 Gflops를 높이면, FID(Fréchet Inception Distance) 점수가 일관되게 낮아지며 성능이 향상된다.16 이는 트랜스포머 아키텍처가 더 많은 데이터와 연산 자원을 통해 지속적으로 성능을 개선할 수 있는 뛰어난 확장성을 가지고 있음을 의미한다.
- **전역적 문맥 및 장거리 의존성 포착:** 트랜스포머의 핵심인 **셀프 어텐션(self-attention)** 메커니즘은 모든 입력 패치 토큰 간의 관계를 동시에 모델링한다. 이는 컨볼루션의 지역적 특성과 대조적으로, 모델이 이미지의 전역적인 문맥과 장거리 의존성을 훨씬 효과적으로 포착할 수 있게 해준다.10 결과적으로 DiT는 특히 고해상도 이미지 생성에서 UNet보다 높은 품질과 일관성을 달성할 수 있다.14

이러한 패러다임의 전환은 우연이 아니었다. 고해상도 및 복잡한 구조적 편집에 대한 수요가 증가함에 따라 CNN의 지역성 편향(locality bias)은 명백한 기술적 병목 현상이 되었다. AI 연구 커뮤니티는 이미 자연어 처리(NLP)와 비전 분야에서 트랜스포머의 우수한 확장성과 전역적 문맥 이해 능력을 목격했으며, 이를 확산 모델에 적용하는 것은 필연적인 수순이었다. DiT가 ImageNet과 같은 주요 벤치마크에서 최고 수준의 성능을 달성하며 16 그 가설을 입증했고, 이는 생성 AI 분야가 더 높은 계산 비용을 감수하더라도 지역적 특징 추출보다 전역적 의미 이해를 우선시하는 방향으로 나아가고 있음을 시사한다.

### 0.4 보고서 구조 및 목표

본 보고서는 DiT 아키텍처의 심층적인 분석을 통해 이미지 편집 분야에서의 역할과 잠재력을 조망하는 것을 목표로 한다. 이를 위해 다음의 구조로 논의를 전개할 것이다. 먼저, **2장**에서는 DiT의 핵심 작동 메커니즘을 VAE 기반 잠재 공간 처리부터 DiT 블록의 구성 요소, 그리고 이미지 편집의 필수 전제 조건인 역전(inversion) 과정까지 상세히 분해한다. **3장**에서는 DiT 기반 편집의 핵심 제어 장치인 셀프 어텐션과 크로스 어텐션의 이분법적 역할을 분석하고, 각 메커니즘이 구조 보존과 의미 부여에 어떻게 기여하는지 탐구한다. **4장**에서는 DiT4Edit, In-Context Edit, LazyDiffusion 등 현대적인 DiT 기반 편집 프레임워크들을 유형별로 분류하고 그 철학과 기술적 혁신을 비교한다. **5장**에서는 DiT를 GAN, ControlNet과 같은 경쟁 패러다임과 비교 분석하여 생성 AI 환경 내에서의 독자적인 위치를 조명한다. **6장**에서는 DiT 기반 모델의 정량적, 정성적 성능을 평가하고, 현재 사용되는 평가 지표의 신뢰성에 대한 비판적 고찰을 제시한다. 마지막으로 **7장**에서는 현재 DiT 기술이 직면한 한계와 이를 극복하기 위한 미래 연구 방향을 논하며 보고서를 마무리한다.

## 1. 아키텍처 심층 분석: 확산 트랜스포머의 작동 원리

### 1.1 픽셀에서 패치로: 비전 트랜스포머(ViT)의 기초

DiT는 이미지의 원시 픽셀(raw pixels)을 직접 처리하지 않는다. 대신, 이미지 처리의 효율성과 트랜스포머 아키텍처의 호환성을 극대화하기 위해 다단계 접근법을 취한다. 첫 번째 단계는 사전 훈련된 **변이형 오토인코더(Variational Autoencoder, VAE)**를 사용하여 원본 이미지를 더 낮은 차원의 잠재 공간(latent space)으로 인코딩하는 것이다.14 이 과정은 고차원의 픽셀 공간을 정보가 압축된 잠재 표현으로 변환하여, 후속 연산의 계산 부담을 크게 줄여준다. 이는 Latent Diffusion Model (LDM)과 같은 기존 모델에서 이미 그 효율성이 입증된 방식이다.

일단 이미지가 잠재 표현으로 변환되면, 이 표현은 겹치지 않는 일련의 **패치(patches)** 또는 **토큰(tokens)**으로 분할된다.16 이 과정은 공간적 연속성을 가진 이미지 데이터를 트랜스포머가 자연스럽게 처리할 수 있는 순차적인 데이터(sequence of tokens)로 변환하는 핵심적인 단계이다. 결과적으로, 이미지 생성이라는 공간적 문제는 트랜스포머의 본연의 영역인 시퀀스-투-시퀀스(sequence-to-sequence) 문제로 재정의된다.

### 1.2 DiT 블록: 핵심 구성 단위

DiT의 심장은 UNet의 ResNet 블록을 대체하는 **DiT 블록**이다. 이 블록은 표준 비전 트랜스포머(ViT)의 구조를 따르지만, 확산 과정에 특화된 몇 가지 중요한 혁신을 포함하고 있다.

- **멀티헤드 셀프 어텐션 (Multi-Head Self-Attention, MHSA):** DiT 블록의 핵심으로, 모든 패치 토큰 간의 전역적인 관계를 포착하는 메커니즘이다.10 각 어텐션 헤드는 입력 시퀀스 내의 서로 다른 관계적 측면을 학습하며, 이를 통해 모델은 이미지의 전체적인 구조와 복잡한 상호작용을 이해할 수 있다.

- **포인트별 피드포워드 네트워크 (Pointwise Feed-Forward Network, FFN):** 어텐션 레이어 다음에 위치하는 표준적인 다층 퍼셉트론(MLP)으로, 어텐션을 통해 집계된 정보를 비선형적으로 변환하고 풍부한 특징 표현을 학습하는 역할을 한다.

- **적응형 레이어 정규화 (Adaptive Layer Norm, adaLN)를 통한 조건화:** DiT의 가장 중요한 혁신 중 하나는 조건화(conditioning) 방식에 있다. 표준 레이어 정규화(Layer Normalization)와 달리, adaLN의 스케일(γ) 및 시프트(β) 파라미터는 확산 타임스텝(t)이나 클래스 레이블, 텍스트 임베딩과 같은 조건 벡터로부터 직접 예측된다.18 이 메커니즘을 통해 모델은 외부의 가이던스를 생성 과정의 각 단계에 효과적으로 주입할 수 있다. 특히, 블록의 최종 출력을 초기에 0으로 초기화하는 

  **adaLN-Zero** 변형은 대규모 모델의 훈련 안정성을 확보하는 데 결정적인 역할을 한다.

- **텍스트 조건화를 위한 크로스 어텐션 (Cross-Attention):** PIXART-α와 같은 텍스트-이미지(text-to-image) DiT 모델에서는 셀프 어텐션 외에 추가적인 **크로스 어텐션** 레이어가 DiT 블록에 통합된다.19 이 레이어는 T5와 같은 텍스트 인코더로부터 생성된 텍스트 임베딩을 쿼리(query)로 사용하여, 이미지 패치 토큰(key, value)과 상호작용한다. 이를 통해 모델은 텍스트 프롬프트의 미묘한 의미를 이미지 생성에 정교하게 반영할 수 있다.

이러한 DiT의 설계는 기존의 강력한 구성 요소들을 전략적으로 재조합한 모듈성의 정수라 할 수 있다. 고차원 문제를 다루기 위해 VAE를 활용하고, 검증된 ViT 아키텍처를 생성의 백본으로 삼았으며, 확산 과정의 특수성을 해결하기 위해 `adaLN`이라는 독창적인 접착제를 발명했다. 이 모듈적 접근 방식 덕분에 VAE나 어텐션 메커니즘과 같은 개별 구성 요소의 발전이 전체 시스템의 성능 향상으로 직결될 수 있는 유연한 구조를 갖추게 되었다.

### 1.3 역전 과정: 실제 이미지 편집의 시작

실제 이미지를 편집하기 위해서는, 먼저 해당 이미지를 생성할 수 있는 초기 노이즈 벡터와 조건들을 찾아내는 **역전(inversion)** 과정이 필수적이다.20 이는 확산 모델을 "거꾸로" 실행하여 주어진 이미지로부터 노이즈를 추정하는 과정으로, 모든 편집 작업의 출발점이다.

- **DDIM 역전:** 많은 UNet 기반 프레임워크에서 흔히 사용되는 역전 알고리즘이지만, 일반적으로 많은 스텝 수를 요구하여 속도가 느린 단점이 있다.14
- **DPM-Solver 역전:** DiT 기반 프레임워크는 이러한 병목 현상을 해결하기 위해 더 발전되고 효율적인 대안을 채택했다. 특히 **DiT4Edit**과 같은 프레임워크는 고차 미분방정식(ODE) 솔버인 **DPM-Solver**를 명시적으로 사용하여 역전에 필요한 스텝 수를 크게 줄였다.10 이는 단순히 속도를 개선하는 것을 넘어, 더 적은 스텝으로 더 높은 품질의 잠재 맵(latent map)을 획득하게 함으로써 편집 파이프라인 전체의 효율성과 결과물의 질을 동시에 향상시킨다.

역전 알고리즘의 선택은 사소한 구현 디테일이 아니라, DiT 기반 편집 프레임워크의 실용성을 결정하는 핵심 요소이다. 편집 도구의 상호작용성을 위해서는 낮은 지연 시간(latency)이 필수적이며, DPM-Solver와 같은 고속 솔버의 채택은 이러한 실용적 요구에 부응하기 위한 필연적인 결정이었다. 이는 성공적인 편집 프레임워크를 구축하는 것이 단순히 핵심 노이즈 제거 모델(DiT)의 성능에만 의존하는 것이 아니라, 샘플러와 역전 알고리즘을 포함한 전체 파이프라인을 최적화해야 함을 보여준다.

## 2. 어텐션의 이분법: DiT 기반 편집의 제어 메커니즘 해부

DiT 기반 이미지 편집의 정교함은 두 가지 핵심 어텐션 메커니즘, 즉 셀프 어텐션(self-attention)과 크로스 어텐션(cross-attention)의 상호 보완적이면서도 뚜렷하게 구분되는 역할에 기반한다. 이 두 메커니즘의 역할을 정확히 이해하는 것은 편집의 성공과 실패를 가르는 결정적인 열쇠이다.

### 2.1 구조의 설계자, 셀프 어텐션: 구조와 레이아웃 보존

최근 연구들은 **셀프 어텐션**이 원본 이미지의 기하학적 구조, 형태, 그리고 전반적인 레이아웃을 보존하는 데 가장 중요한 역할을 한다는 사실을 명확히 밝혀냈다.22 "Towards Understanding Cross and Self-Attention..." 논문과 같은 심층 분석 연구에 따르면, 셀프 어텐션 맵은 이미지 내 다른 특징들 간의 연관성을 반영하며, 이를 통해 이미지의 공간적 정보를 효과적으로 유지한다.22

특히 트랜스포머의 깊은 레이어일수록 셀프 어텐션을 통해 객체의 세부적인 디테일과 레이아웃 정보를 더욱 효과적으로 포착하는 경향이 있다.10 이 때문에 비강체 변형(non-rigid editing)이나 대규모 형태 변경과 같이 구조적 일관성이 중요한 편집 작업에서는 셀프 어텐션 제어가 필수적이다.

이러한 통찰을 실제 기술로 구현한 대표적인 예가 **Free-Prompt-Editing (FPE)**이다.22 FPE는 튜닝이 필요 없는(tuning-free) 편집 방식으로, 노이즈 제거 과정에서 크로스 어텐션은 전혀 건드리지 않고 오직 

**셀프 어텐션 맵만을 수정**하여 편집을 수행한다. 구체적으로, 원본 이미지의 노이즈 제거 과정에서 계산된 셀프 어텐션 맵을 목표 이미지 생성 과정의 동일한 스텝에 주입(inject)한다. 이를 통해 새로운 텍스트 프롬프트에 따라 의미론적 내용은 바뀌지만, 원본 이미지의 구조와 형태는 그대로 유지되는 안정적인 편집이 가능해진다.

### 2.2 의미의 예술가, 크로스 어텐션: 텍스트 의미 부여

전통적으로 **크로스 어텐션**의 역할은 생성된 이미지를 조건으로 주어진 텍스트 프롬프트와 정렬(align)시키는 것으로 이해되어 왔다.22 Prompt-to-Prompt (P2P)와 같은 초기 편집 기법들은 바로 이 원리를 이용하여, 프롬프트 내 특정 단어에 해당하는 크로스 어텐션 맵을 교체하거나 가중치를 조절함으로써 이미지의 의미를 수정하고자 했다.

그러나 심층 분석 결과, 크로스 어텐션 맵을 직접 조작하는 것은 선택 사항일 뿐만 아니라, 종종 편집 실패의 직접적인 원인이 될 수 있다는 역설적인 사실이 밝혀졌다.22

**편집 실패의 원인:** 그 핵심 이유는 크로스 어텐션 맵이 단순히 텍스트 토큰과 이미지 픽셀 간의 가중치 맵이 아니기 때문이다. 이 맵에는 조건으로 사용된 **토큰 자체의 풍부한 의미론적 특징 정보(semantic feature information)**가 함께 섞여 들어간다.22 따라서 원본 이미지의 크로스 어텐션 맵을 목표 이미지 생성 과정에 주입하면, 원치 않는 의미론적 정보가 "오염(semantic bleed)"처럼 번져나가 예측 불가능한 아티팩트나 왜곡을 유발할 수 있다. 예를 들어, '파란 차'를 '빨간 차'로 바꾸기 위해 '파란'의 크로스 어텐션 맵을 조작하면, '파란색'이라는 의미 정보 자체가 편집 과정을 방해하여 실패로 이어질 수 있다.

이러한 발견은 초기 편집 방법론의 근본적인 가정을 뒤흔들었다. 이미지의 내용을 바꾸기 위해 텍스트와 이미지를 연결하는 크로스 어텐션을 수정해야 한다는 논리적 가정은 경험적 현실 앞에서 무너졌다. 이 모순을 해결하기 위한 탐구는 셀프 어텐션이 구조의 진정한 수호자이며, 크로스 어텐션은 예측 불가능한 변수임을 밝혀냈다. 이는 FPE와 같은 새로운 패러다임을 낳았다. 구조가 최우선이고 크로스 어텐션이 불안정하다면, 구조만 보존하고(셀프 어텐션 주입) 새로운 프롬프트의 크로스 어텐션은 자연스럽게 형성되도록 두자는 것이다. 이 접근법의 성공은 어텐션 메커니즘에 대한 커뮤니티의 이해가 '블랙박스' 조작에서 원리 기반의 정교한 제어로 발전했음을 의미한다.

### 2.3 시너지 제어: 듀얼 어텐션 프레임워크의 부상

FPE가 셀프 어텐션의 독립적인 힘을 증명했다면, 더 발전된 프레임워크들은 두 메커니즘을 통합적으로 또는 상호 보완적으로 제어하여 시너지를 창출하고자 한다. 이는 P2P(정: Thesis)와 FPE(반: Antithesis)를 거쳐 도달한 '합(Synthesis)'의 단계로 볼 수 있다.

**DiT4Edit**과 같은 프레임워크는 "통합 어텐션 제어(unified attention control)"라는 개념을 제안한다.10 이는 의미론적 변경이나 스타일 변환을 위해서는 크로스 어텐션을 제어하고(맵 교체 또는 미세 조정), 레이아웃과 형태 보존을 위해서는 셀프 어텐션을 제어하는 이원화된 전략을 사용한다.

또 다른 듀얼 제어 방식은 어텐션 레이어의 깊이에 따라 역할을 분담하는 것이다. 예를 들어, 모델의 "얕은(coarse)" 레이어에서는 셀프 어텐션을 제어하여 이미지의 세부 디테일을 유지하고, "깊은(fine)" 레이어에서는 스타일 변환을 유도한다. 동시에, 크로스 어텐션 제어를 통해 원본 이미지와의 구조적 일관성을 확보한다.27 이는 모델의 각 부분이 기능적으로 특화되어 있다는 깊은 이해를 바탕으로 한 정교한 접근법이다. 이처럼 연구의 흐름은 두 어텐션 메커니즘을 대립적인 것으로 보지 않고, 각자의 강점을 극대화하고 약점을 보완하는 성숙하고 미묘한 방식으로 진화하고 있다.

## 3. 현대 DiT 기반 이미지 편집 프레임워크의 유형학

DiT 아키텍처의 등장은 이미지 편집 분야에 새로운 가능성의 장을 열었다. 연구자들은 DiT라는 강력하고 유연한 기반 위에 다양한 철학과 목표를 가진 편집 프레임워크들을 구축하고 있다. 본 장에서는 주요 DiT 기반 편집 프레임워크들을 유형별로 분류하고, 각각의 핵심 아이디어와 기술적 혁신을 분석하여 DiT 활용 전략의 다채로운 스펙트럼을 조망한다.

### 3.1 직접적 어텐션 조작: DiT4Edit 프레임워크

- **핵심 아이디어:** 사전 훈련된 텍스트-이미지 DiT 모델(예: PIXART-α)을 사용하여 편집을 수행하는 최초의 프레임워크로, 특히 고해상도 및 임의 크기의 이미지에서 UNet 기반 방법들을 품질과 유연성 면에서 능가하는 것을 목표로 한다.10 이는 전문가 사용자가 모델의 내부 메커니즘을 직접 제어하여 최상의 결과를 얻고자 하는 "직접 제어" 철학을 대표한다.
- **주요 기술:**
  - **통합 어텐션 제어 (Unified Attention Control):** 3장에서 논의된 바와 같이, 의미/스타일 변경을 위한 크로스 어텐션 조작과 레이아웃/형태 보존을 위한 셀프 어텐션 제어를 결합한 이원적 제어 전략을 사용한다.10
  - **DPM-Solver 역전:** DDIM에 비해 더 빠르고 효율적인 고차 솔버를 사용하여 추론 시간을 단축하고 잠재 맵의 품질을 향상시킨다.10
  - **패치 병합 (Patches Merging):** 어텐션 연산 전에 유사한 이미지 패치들을 식별하여 미리 병합하는 효율성 중심의 기술이다. 이는 계산 비용이 높은 셀프 어텐션의 부하를 극적으로 줄이면서도 품질 저하를 최소화한다.15
- **강점:** 객체 편집, 스타일 편집, 형태 인식 편집 등 다양한 시나리오에서 강력한 성능을 입증했다.10

### 3.2 인-컨텍스트 학습: "In-Context Edit" 프레임워크

- **핵심 아이디어:** 복잡한 파인튜닝이나 직접적인 어텐션 조작에서 벗어나, 대규모 DiT가 "인-컨텍스트(in-context)"로 제공된 예제를 통해 제로샷(zero-shot)으로 지시 기반 편집을 수행하도록 유도한다.8 이는 LLM의 성공적인 인-컨텍스트 학습 능력에서 영감을 받은 "예시 기반 학습" 철학을 따른다.
- **주요 기술:**
  - **비전-언어 인-컨텍스트 프롬프트:** 모델에 "왼쪽은 원본 {설명}을 묘사하고, 오른쪽은 왼쪽을 반영하되 {편집 지시}를 적용한, 동일한 {주제}의 나란한 이미지"와 같은 특정 구조의 프롬프트를 제공한다.32 모델은 이 예시적인 구조를 통해 수행해야 할 편집 작업을 추론한다.
  - **LoRA-MoE 하이브리드 튜닝:** 극도로 파라미터 효율적인 튜닝 전략이다. 전체 DiT를 훈련하는 대신, 가벼운 LoRA 어댑터를 DiT 블록에 통합한다. 그리고 전문가 혼합(Mixture-of-Experts, MoE) 라우터가 입력 토큰과 텍스트 임베딩을 기반으로 특정 작업에 가장 적합한 LoRA 어댑터를 동적으로 선택한다.8 이를 통해 기존 방식 대비 약 1%의 학습 가능한 파라미터만으로 최고 수준의 성능을 달성한다.30
  - **초기 필터 추론 시간 스케일링:** 비전-언어 모델(VLM)을 사용하여 노이즈 제거 초기 단계에서 여러 노이즈 후보를 평가하는 품질 향상 기법이다. VLM이 텍스트 지시와 가장 잘 일치하는 후보를 선택하면, 이 후보를 사용하여 전체 노이즈 제거를 수행함으로써 최종 결과물의 견고성과 품질을 높인다.32

### 3.3 효율성 중심 아키텍처: "LazyDiffusion" 프레임워크

- **핵심 아이디어:** 인페인팅과 같은 국소적 편집의 계산 비용을 편집 영역의 크기에 비례하도록 획기적으로 줄여, 고도로 상호작용적인 편집 경험을 제공하는 것을 목표로 한다.17 이는 "효율성 우선" 철학을 극단적으로 추구한 결과물이다.
- **주요 기술:**
  - **비대칭 인코더-디코더 아키텍처:** 편집 과정을 두 단계로 분리한다. 강력한 **컨텍스트 인코더**는 편집당 단 *한 번만* 실행되어 전체 캔버스를 처리하고 전역 컨텍스트의 압축된 표현을 생성한다.
  - **토큰 드롭핑 (Token Dropping):** 인코더는 전체 이미지에 대한 토큰을 생성한 후, 전략적으로 마스크되지 않은(보이는) 영역에 해당하는 토큰을 *폐기*한다. 오직 마스크된 "구멍(hole)"에 대한 토큰만 유지한다. 이 정보 병목 현상은 전체 이미지의 컨텍스트가 구멍 경계의 표현 속으로 압축되도록 강제한다.17
  - **게으른 디코더 (Lazy Decoder):** 확산 기반 트랜스포머 디코더는 오직 **마스크된 영역의 토큰에 대해서만** 반복적인 노이즈 제거를 수행한다. 이 디코더는 인코더로부터 압축된 전역 컨텍스트를 조건으로 받기 때문에, 생성된 패치는 나머지 이미지와 일관성을 유지한다. 이 "게으른" 연산 방식은 상당한 속도 향상(10% 마스크에 대해 최대 10배)을 가져온다.17

### 3.4 프로그래밍 방식 편집: "IEAP" 프레임워크

- **핵심 아이디어:** "왼쪽 사람을 지우고 오른쪽에 개를 추가해 줘"와 같이 복잡하고 여러 단계로 구성된 지시나 구조적으로 일치하지 않는 편집을 처리하기 위해, 이미지 편집 자체를 실행해야 할 **프로그램**으로 간주한다.7 이는 "구성적 추론" 철학에 기반한다.
- **주요 기술:**
  - **지시 분해 (Instruction Decomposition):** VLM이 에이전트 역할을 하여 사용자의 자유 형식 지시를 `REMOVE(person_left)`, `ADD(dog, right)`와 같은 일련의 원자적 연산(atomic operations)으로 파싱한다.
  - **경량 어댑터:** 각 원자적 연산(추가, 제거, 교체 등)은 동일한 DiT 백본을 공유하는 경량 어댑터로 구현된다. 이 어댑터들은 각자의 특정 작업에 특화되어 있다.
  - **순차적 실행:** 신경망 프로그램 인터프리터가 이러한 연산들을 DiT 백본 위에서 순차적으로 실행한다. 이를 통해 단일 프롬프트 방식으로는 처리하기 어려운 복잡하고 레이아웃을 변경하는 변환이 가능해진다.7

이러한 프레임워크들의 다양성은 DiT를 활용하는 단 하나의 "최고의" 방법이 없음을 시사한다. 오히려 DiT는 다양한 사용자 상호작용 및 제어 철학을 구현할 수 있는 유연한 기반 아키텍처 역할을 한다. 미래의 이미지 편집 도구는 이러한 철학들이 혼합된 하이브리드 형태가 될 가능성이 높다. 사용자는 인-컨텍스트 예제로 시작하여, 빠르고 게으른 인페인팅으로 수정하고, 마지막으로 프로그래밍 방식의 복잡한 지시를 통해 대대적인 변경을 가하는 워크플로우를 상상해 볼 수 있다.

**표 1: 주요 DiT 기반 이미지 편집 프레임워크 비교**

| 프레임워크 이름     | 핵심 철학/목표 | 주요 기술 혁신                                   | 주요 적용 분야                              |
| ------------------- | -------------- | ------------------------------------------------ | ------------------------------------------- |
| **DiT4Edit**        | 직접 제어      | 통합 어텐션 제어, DPM-Solver, 패치 병합          | 고해상도 객체/스타일/형태 편집              |
| **In-Context Edit** | 예시 기반 학습 | 비전-언어 인-컨텍스트 프롬프트, LoRA-MoE 튜닝    | 제로샷 지시 기반 편집, 파라미터 효율적 튜닝 |
| **LazyDiffusion**   | 효율성 우선    | 비대칭 인코더-디코더, 토큰 드롭핑, 게으른 디코더 | 상호작용적 인페인팅 및 국소적 편집          |
| **IEAP**            | 구성적 추론    | VLM 기반 지시 분해, 원자적 연산 어댑터           | 복잡한 다단계 지시, 레이아웃 변경 편집      |

## 4. 비교 분석: 생성 AI 환경 속 DiT의 위치

DiT 아키텍처의 혁신성을 제대로 평가하기 위해서는 이를 기존의 지배적인 생성 모델 패러다임인 GAN(Generative Adversarial Networks) 및 ControlNet과 비교하여 그 강점과 약점, 그리고 독자적인 위치를 명확히 할 필요가 있다. 이 비교는 단순히 기술적 우위를 가리는 것을 넘어, 각 아키텍처가 지향하는 제어 철학의 차이를 드러낸다.

**표 2: 이미지 편집을 위한 생성 아키텍처 비교 개요**

| 구분                   | DiT (Diffusion Transformer)                  | UNet (w/ ControlNet)                                      | GAN (StyleGAN)                                         |
| ---------------------- | -------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------ |
| **백본 아키텍처**      | 트랜스포머                                   | UNet (컨볼루션 기반)                                      | 다층 퍼셉트론(MLP) 기반                                |
| **주요 제어 메커니즘** | 텍스트 프롬프트, 어텐션 조작                 | 외부 조건(포즈, 뎁스 등), 텍스트 프롬프트                 | 잠재 공간(Latent Space) 벡터 조작                      |
| **강점**               | 뛰어난 의미론적 유연성, 고품질 생성, 확장성  | 정교한 공간적 제어, 기존 모델과의 호환성                  | 실시간, 세밀하고 분리된 속성 제어                      |
| **약점**               | 직접적 제어의 어려움, 높은 계산 비용         | 장거리 의존성 포착 한계, 새로운 조건에 대한 훈련 필요     | 일반성 부족, 알려지지 않은 속성 편집의 어려움          |
| **이상적 사용 사례**   | 복잡하고 창의적인 개념의 이미지 생성 및 편집 | 포즈, 구도, 형태 등 특정 조건을 정확히 따르는 이미지 생성 | 얼굴 편집 등 특정 도메인에서 단일 속성을 정밀하게 조절 |

### 4.1 DiT 대 GAN: 제어 가능성의 트레이드오프

- GAN의 강점 - 직접적인 잠재 공간 조작:

  GAN, 특히 StyleGAN과 같은 모델은 구조화되고 분리된(disentangled) 잠재 공간을 학습하는 데 탁월하다.35 이 공간에서 특정 의미론적 속성(예: '안경 추가', '나이 들게 하기')에 해당하는 방향 벡터를 찾으면, 잠재 벡터를 이 방향으로 이동시키는 것만으로 해당 속성을 이미지에 반영할 수 있다.36 이 방식은 실시간으로 연속적이고 매우 정밀한 편집을 가능하게 한다. 사용자는 다른 속성에 영향을 주지 않으면서 단일 속성만을 세밀하게 제어할 수 있다.35 실제 이미지를 이 잠재 공간으로 매핑하는 과정을 "GAN 역전(GAN Inversion)"이라고 한다.20

- DiT의 강점 - 프롬프트를 통한 의미론적 유연성:

  DiT를 포함한 확산 모델은 주로 자연어 프롬프트를 통해 제어된다. 확산 모델의 "잠재 공간"(초기 노이즈 벡터)은 GAN의 잠재 공간처럼 의미론적으로 구조화되어 있지 않다.35 편집은 텍스트 프롬프트를 변경하고, 모델의 크로스 어텐션이 그 변화를 해석하여 이미지에 반영하는 방식으로 이루어진다.39 이는 단일 벡터로는 정의하기 어려운 새로운 조합이나 복잡한 개념을 표현하는 데 있어 타의 추종을 불허하는 유연성을 제공한다.

- 트레이드오프:

  이 두 접근 방식의 차이는 '직접 조작'과 '서술적 지시'라는 AI 상호작용 철학의 근본적인 차이를 반영한다.

  - **GAN:** 알려진 속성에 대해서는 매우 높은 정밀도와 실시간 제어 능력을 제공하지만, 일반성이 부족하다. 훈련 데이터나 잠재 공간에서 잘 표현되지 않은 새로운 개념을 편집하는 데는 어려움을 겪는다.39
  - **DiT:** 비할 데 없는 유연성과 일반성을 자랑하지만, 직접적인 제어는 어렵다. 프롬프트 가중치를 조절하는 것은 슬라이더를 움직이는 것보다 직관성이 떨어지며, 때로는 이미지 전체에 예측 불가능한 변화를 초래할 수 있다.35

미래의 생성 모델 인터페이스는 어느 한쪽이 승리하는 것이 아니라, 두 방식의 장점을 결합한 하이브리드 시스템이 될 가능성이 높다. 예를 들어, 프롬프트로 기본 이미지를 생성한 후, 특정 속성을 미세 조정하기 위해 GAN과 유사한 잠재 공간 컨트롤을 노출하는 방식이다.

### 4.2 DiT 대 ControlNet: 백본과 조건성의 대결

- **ControlNet의 패러다임:** ControlNet은 포즈, 뎁스 맵, 외곽선 등 정교한 공간적 조건을 **사전 훈련된 UNet 기반 확산 모델**(예: Stable Diffusion)에 추가하기 위한 독창적인 방법론이다.13 이는 UNet 인코더 블록의 가중치를 복사하여 하나는 '잠금(frozen)' 상태로 두고 다른 하나는 '훈련 가능(trainable)' 상태로 만든 뒤, 이 둘을 "제로 컨볼루션(zero convolutions)"으로 연결하는 방식으로 작동한다.12 이를 통해 기존의 강력한 모델을 손상시키지 않으면서 새로운 제어 능력을 학습할 수 있다.
- **"특징 공간 불일치(Feature Space Mismatch)" 문제:** UNet 백본을 위해 훈련된 ControlNet은 DiT 백본에 직접 연결하여 사용할 수 없다.44 두 아키텍처의 내부 구조와 그들이 연산하는 특징 공간이 근본적으로 다르기 때문이다. 새로운 DiT 모델이 나올 때마다 ControlNet을 처음부터 다시 훈련시키는 것은 엄청난 계산 비용과 노력을 요구한다.45
- 해결책 - Ctrl-Adapter: Ctrl-Adapter는 이 불일치 문제를 해결하기 위한 효율적인 프레임워크다.44
  - **작동 원리:** 새로운 ControlNet을 훈련하는 대신, 작고 가벼운 "어댑터" 모듈을 훈련시킨다. 이 어댑터는 사전 훈련된 (그리고 고정된) UNet 기반 ControlNet의 출력 특징을 새로운 (그리고 고정된) DiT 백본의 해당 입력 블록에 맞게 매핑하는 방법을 학습한다.44
  - **효율성:** 거대한 DiT 모델과 ControlNet이 모두 고정되어 있기 때문에, 어댑터 훈련은 매우 빠르고 자원 효율적이다(10 GPU 시간 미만).44
  - **다재다능함:** 이 접근법은 기존의 방대한 ControlNet 생태계를 더 강력한 DiT 아키텍처로 확장시켜 DiT의 제어 가능성을 크게 향상시킨다. 또한 비디오 편집 작업에서 시간적 일관성을 개선하기 위한 시간적 모듈(temporal modules)도 포함하고 있다.44

Ctrl-Adapter의 등장은 DiT가 생성 모델의 새로운 표준 백본으로 부상하고 있음을 보여주는 강력한 증거이다. 이는 커뮤니티가 DiT를 미래로 인식하고 있으며, 이제는 낡은 생태계(UNet/ControlNet) 위에서 계속 개발하기보다는, 낡은 생태계의 자산을 새로운 플랫폼(DiT)으로 가져오는 '다리'를 놓는 것이 더 효율적이라고 판단하고 있음을 시사한다. 이는 기술 계승 과정에서 흔히 나타나는 패턴으로, 새로운 플랫폼의 지배력을 입증하는 현상이다.

## 5. 성능, 평가, 그리고 지표의 정치학

DiT 기반 모델의 우수성을 논하기 위해서는 정량적 벤치마킹 결과와 정성적 시각 증거를 종합적으로 검토해야 한다. 그러나 더 깊이 있는 전문가적 분석은 현재 사용되는 평가 지표 자체의 신뢰성에 대한 비판적 고찰을 포함해야 한다. 생성 모델의 발전이 기존 평가 체계의 한계를 드러내고 있기 때문이다.

### 5.1 정량적 벤치마킹: 결과의 종합

연구 논문들에서 보고된 정량적 데이터는 DiT 기반 모델의 성능적 우위를 일관되게 보여준다.

- **DiT4Edit 성능:** DiT4Edit 논문에 제시된 성능 비교표는 이 모델이 SDEdit, InstructPix2Pix (IP2P) 등 다수의 UNet 기반 베이스라인 모델들을 여러 이미지 해상도에서 압도함을 명확히 보여준다.19 아래 표에서 볼 수 있듯이, DiT4Edit은 더 낮은(좋은) FID 점수와 더 높은(좋은) PSNR 점수를 기록했는데, 이는 생성된 이미지의 사실성(realism)과 원본 대비 구조적 충실도(fidelity)가 모두 뛰어남을 의미한다.10

**표 3: DiT4Edit 대 베이스라인 모델 정량적 성능 비교**

| 모델                | 구조          | FID ↓ (512x512) | FID ↓ (1024x1024) | FID ↓ (1024x2048) | PSNR ↑ (512x512) | PSNR ↑ (1024x1024) | PSNR ↑ (1024x2048) | CLIP ↑ (512x512) | CLIP ↑ (1024x1024) | CLIP ↑ (1024x2048) |      |
| ------------------- | ------------- | --------------- | ----------------- | ----------------- | ---------------- | ------------------ | ------------------ | ---------------- | ------------------ | ------------------ | ---- |
| SDEdit              | UNet-based    | 93.25           | 88.56             | 143.87            | 21.54            | 20.76              | 16.35              | 22.41            | 21.39              | 20.73              |      |
| IP2P                | UNet-based    | 88.63           | 98.73             | 103.52            | 20.36            | 19.73              | 17.13              | 20.64            | 19.36              | 21.53              |      |
| Pix2Pix-Zero        | UNet-based    | 92.31           | 101.32            | 158.29            | 20.6             | 16.27              | 23.58              | 22.12            | 17.85              | 20.39              |      |
| MasaCtrl            | UNet-based    | 110.75          | 176.15            | 236.49            | 17.35            | 20.51              | 16.92              | 23.51            | 21.96              | 15.23              |      |
| InfEdit             | UNet-based    | 86.28           | 87.42             | 98.75             | 21.54            | 22.36              | 21.74              | 24.46            | 23.74              | 22.16              |      |
| PnPInversion        | UNet-based    | 84.73           | 85.33             | 110.73            | 21.71            | 23.42              | 24.59              | 21.71            | 20.76              | 19.35              |      |
| **DiT4Edit (Ours)** | **DiT-based** | **72.36**       | **62.45**         | **75.43**         | **22.85**        | **29.75**          | **27.46**          | **25.39**        | **26.97**          | **25.66**          |      |

출처: DiT4Edit 논문 19의 데이터를 재구성함.

- **In-Context Edit 성능:** 이 프레임워크는 Emu Edit, MagicBrush와 같은 벤치마크에서 최고 수준의 결과를 달성했으며, 이는 기존 방법 대비 압도적으로 우수한 데이터 및 파라미터 효율성(각각 0.5%, 1%)으로 이루어낸 성과다.30 심지어 SeedEdit과 같은 상용 시스템과 비교해서도 경쟁력 있는 성능을 보여주었다.8

### 5.2 정성적 평가: 시각적 증거

정량적 수치를 넘어, DiT 기반 모델들의 진정한 강점은 시각적 결과물에서 드러난다.

- 논문들에서 제시된 예시들은 DiT 모델이 구성적 무결성을 훼손하지 않으면서 복잡하고 형태를 인식하는 편집을 수행하는 능력이 탁월함을 보여준다.15
- 특히 고해상도 편집에서 UNet 기반 모델들이 종종 겪는 컨텍스트 손실이나 디테일 뭉개짐 현상 없이, 전역적 일관성과 미세한 질감을 잘 보존한다.11
- GitHub 리포지토리 등에서 공개된 다양한 예시들은 스타일 변환, 객체 조작뿐만 아니라 비디오 편집으로까지 그 응용 범위가 확장되고 있음을 보여주며 DiT의 다재다능함을 입증한다.16

### 5.3 지표에 대한 비판적 고찰: FID의 신뢰성 문제

표준적으로 생성 모델의 평가는 이미지 분포의 유사성을 측정하는 **FID(Fréchet Inception Distance)**와 이미지-텍스트 정렬도를 측정하는 **CLIP Score**를 통해 이루어진다.51 그러나 DiT와 같은 현대 생성 모델의 성능을 평가하는 데 있어, 특히 FID는 심각한 한계를 노출하고 있다.

최근 연구들은 FID의 여러 문제점을 강력하게 비판한다.54

- **부적절한 특징 표현:** FID의 기반이 되는 Inception-v3 모델은 1000개의 클래스를 가진 ImageNet 데이터셋으로 훈련되었다. 이는 현대 생성 모델들이 만들어내는 풍부하고 복잡하며 창의적인 콘텐츠를 제대로 표현하기에는 턱없이 부족하다.54
- **잘못된 정규성 가정:** FID는 특징 분포가 다변량 정규분포를 따른다고 가정하지만, 실제 데이터의 분포는 이 가정을 만족하지 않는 경우가 많다.54
- **인간의 판단과 불일치:** 가장 치명적인 문제로, FID 점수는 인간 평가자의 판단과 상충될 수 있다. 심지어 이미지에 미세한 왜곡을 **추가**했을 때 오히려 FID 점수가 개선되는(낮아지는) 역설적인 현상이 관찰되기도 한다.54
- **샘플 크기 불일치:** FID는 편향된 추정량(biased estimator)으로, 계산에 사용되는 샘플의 수에 따라 그 값이 일관성 없이 변동하는 경향이 있다.54

이러한 비판은 DiT 기반 모델이 표준 지표에서 우수하다는 사실을 넘어서, 그 평가 기준 자체에 대한 근본적인 질문을 던지게 한다. DiT와 같은 강력한 모델의 등장이 역설적으로 기존 평가 체계의 취약성을 드러낸 것이다. Inception-v3는 초기 생성 모델을 평가하는 데는 충분했을지 모르나, 현대 DiT의 창의적 스펙트럼을 감당하기에는 역부족이다. 생성 기술의 발전이 평가 기술의 발전을 강제하고 있는 형국이다. 이는 CMMD(CLIP-based MMD)와 같이 인간의 인식과 더 잘 부합하는 새로운 평가 지표 개발이 왜 중요한 미래 연구 방향인지를 명확히 보여준다.55

## 6. 현재의 한계와 미래의 지향점

DiT 아키텍처는 이미지 생성 및 편집 분야에 혁명적인 발전을 가져왔지만, 동시에 새로운 기술적 과제와 한계를 드러냈다. 이러한 한계를 인식하고 이를 극복하기 위한 연구 방향을 모색하는 것은 DiT의 미래를 조망하는 데 필수적이다.

### 6.1 식별된 과제 및 현재의 한계

- **계산 복잡성:** 셀프 어텐션의 제곱에 비례하는(O(N2)) 계산 복잡성은 여전히 가장 큰 병목으로 남아있다. 특히 초고해상도 이미지나 비디오를 생성할 때 이 문제는 더욱 심각해지며, 막대한 계산 자원을 요구한다.56
- **편집 정확성 대 유연성:** 이미지 편집에는 근본적인 트레이드오프가 존재한다. 결정론적 ODE 샘플링을 사용하여 원본 이미지에 대한 충실도를 높이는 방법은 '닫힌 입을 열게 하는' 것과 같이 새로운 콘텐츠를 "상상"하는 유연성이 부족할 수 있다. 반면, 확률론적 SDE 샘플링을 사용하여 유연성을 높이는 방법은 콘텐츠의 일관성을 유지하는 데 어려움을 겪을 수 있다.57
- **텍스트 단독 가이던스의 한계:** 텍스트 프롬프트는 본질적으로 모호할 수 있으며, 미세한 공간적 지시를 전달하는 데 한계가 있다. 많은 편집 의도는 순전히 단어만으로는 설명하기 어려워, 프롬프트 기반 방법의 정밀도를 제한한다.58 이러한 한계는 이미지 프롬프트나 포인트 기반 편집과 같은 다중 모드 가이던스 개발의 핵심 동기가 된다.
- **분리(Disentanglement):** GAN의 구조화된 잠재 공간과 비교할 때, 확산 모델에서 하나의 속성만을 다른 속성에 영향을 주지 않고 독립적으로 편집하는 진정한 의미의 분리된 편집을 달성하는 것은 여전히 중요한 과제로 남아있다.60

### 6.2 새로운 연구 방향과 미래의 지향점

DiT 분야의 주요 연구 흐름은 아키텍처의 핵심 속성에서 비롯된 문제들을 해결하는 방향으로 나아가고 있다. 즉, DiT의 근본적인 약점(계산 비용)을 완화하고, 근본적인 강점(생성 품질 및 확장성)을 극대화하는 것이다.

- **효율성 최적화:**

  - **사후 훈련 압축 (Post-Training Compression):** **DiTFastAttn**과 같은 방법론은 재훈련 없이 어텐션 연산량을 극적으로 줄이기 위해 윈도우 어텐션(Window Attention), 타임스텝 간 어텐션 공유(Attention Sharing across Timesteps) 등의 기술을 사용한다. 이를 통해 상당한 종단 간(end-to-end) 속도 향상을 달성한다.56
  - **효율적인 아키텍처:** 특정 고빈도 작업을 위해 **LazyDiffusion**과 같이 특화된 아키텍처를 지속적으로 개발하는 연구가 진행 중이다.17

- **진보된 제어 메커니즘:**

  - **다중 모드 프롬프트:** 텍스트를 넘어 이미지 프롬프트 57나 

    **DragDiffusion**과 같은 상호작용적인 포인트 기반 제어 59를 포함하여, 더 정밀하고 직관적인 가이던스를 제공하려는 시도가 활발하다.

  - **비지도 편집 (Unsupervised Editing):** **LOCO Edit**과 같이 추가적인 훈련 없이 확산 모델 내의 의미론적 부분 공간을 비지도적으로 발견하고 활용하여, 제어 가능한 편집을 가능하게 하는 연구가 부상하고 있다.60

- **새로운 도메인으로의 확장:**

  - **비디오 편집:** DiT의 원리를 **비디오 확산 트랜스포머(Video DiTs)**로 확장하는 것은 가장 중요한 미래 지향점 중 하나이다. 이는 시간적 일관성이라는 새로운 과제를 제기하며, **DFVEdit**이나 **Ctrl-Adapter**와 같은 프레임워크들이 이 도메인을 위해 특별히 개발되고 있다.11
  - **3D 및 기타 모달리티:** 이러한 원리들이 NeRF 편집, 3D 그래픽, 심지어 오디오 기반 편집에까지 적용되고 있으며, 이는 기반 아이디어의 일반성을 보여준다.61

- **통합 모델:** **ACE**와 같이 광범위한 생성 및 편집 작업을 단일 통합 프레임워크 내에서 처리하려는 "올인원(all-in-one)" 모델을 향한 움직임도 나타나고 있다.62

이러한 연구 동향은 미래의 생성 AI 개발이 거대한 단일 모델을 처음부터 구축하는 방식에서 벗어나고 있음을 시사한다. 대규모 DiT를 훈련하는 것은 극소수의 대형 연구소만이 감당할 수 있는 비용이 들기 때문이다. 따라서 더 넓은 연구 및 혁신의 장은, 이 거대하고 사전 훈련된 모델들을 수정하지 않고 그 위에서 작동하는 경량의, 특화된, 그리고 종종 튜닝이 필요 없는 모듈(어댑터, 컨트롤러, 가이던스 메커니즘)을 만드는 데 있다. Ctrl-Adapter, FPE, LOCO Edit, DiTFastAttn, In-Context Edit 등은 모두 이러한 흐름을 보여주는 대표적인 예이다. 이는 혁신을 민주화하고, 소수의 강력한 엔진(DiT 백본)을 중심으로 특화된 편집 기능들이 폭발적으로 증가하는 '캄브리아기 대폭발'과 같은 생태계를 조성할 것이다.

## 7. 결론

본 보고서는 이미지 편집을 위한 어텐션 기반 확산 트랜스포머(ADiT), 즉 DiT 모델에 대한 심층적인 고찰을 제공했다. 분석을 통해 DiT는 단순히 기존 UNet 아키텍처의 점진적 개선이 아니라, 생성 모델링의 근본적인 패러다임을 전환시킨 혁신임이 분명해졌다.

첫째, DiT의 등장은 UNet의 컨볼루션 기반 구조가 가진 장거리 의존성 포착의 한계라는 명확한 기술적 필요성에 의해 추동되었다. 트랜스포머의 셀프 어텐션 메커니즘을 도입함으로써 DiT는 전역적 문맥을 이해하고 뛰어난 확장성을 확보했으며, 이는 고해상도 이미지 생성 및 복잡한 구조적 편집에서 질적인 도약을 이루는 기반이 되었다.

둘째, DiT 내부의 어텐션 메커니즘에 대한 이해는 '직접 조작'에서 '원리 기반 제어'로 진화했다. 초기에는 크로스 어텐션이 의미론적 제어의 핵심으로 여겨졌으나, 심층 분석을 통해 셀프 어텐션이 구조 보존의 중추적 역할을 하며, 크로스 어텐션 조작은 오히려 '의미론적 오염'으로 인해 편집 실패를 유발할 수 있음이 밝혀졌다. 이러한 이해는 셀프 어텐션만을 제어하는 FPE를 거쳐, 두 메커니즘의 역할을 정교하게 분담하는 DiT4Edit과 같은 듀얼 어텐션 프레임워크의 등장으로 이어졌다.

셋째, DiT라는 강력한 백본 위에서 다양한 편집 철학을 가진 프레임워크 생태계가 형성되고 있다. DiT4Edit의 '직접 제어', In-Context Edit의 '예시 기반 학습', LazyDiffusion의 '효율성 우선', IEAP의 '구성적 추론'은 각각 다른 사용자 요구와 기술적 과제를 해결하며 DiT의 다재다능함을 입증한다.

넷째, DiT는 GAN, ControlNet과 같은 기존 패러다임과의 비교를 통해 그 독자적인 위치를 확립했다. GAN의 정밀한 직접 제어 능력과 DiT의 유연한 의미론적 표현 능력은 상호 보완적이며, ControlNet의 방대한 제어 생태계를 DiT로 효율적으로 이식하려는 Ctrl-Adapter의 등장은 DiT가 차세대 표준 백본으로 자리매김하고 있음을 시사한다.

마지막으로, DiT가 직면한 계산 비용, 제어 정밀성, 평가 지표의 신뢰성 등의 과제들은 곧 미래 연구의 방향을 제시한다. 효율성 최적화, 다중 모드 제어, 비디오 및 3D로의 도메인 확장, 그리고 거대 모델 위에 구축되는 경량 모듈 생태계의 활성화는 DiT가 앞으로 나아갈 길을 밝히고 있다.

결론적으로, ADiT 모델, 즉 확산 트랜스포머는 이미지 편집 분야의 현주소를 재정의하고 미래의 가능성을 확장하는 핵심 기술이다. 이는 단순한 아키텍처의 변화를 넘어, 생성 AI가 어떻게 세상을 인식하고, 사용자와 상호작용하며, 창의적인 결과물을 만들어내는 방식 자체를 근본적으로 바꾸고 있는 혁신의 동력이다.

#### **참고 자료**

1. ADIT PRISCILLA | Heroes Model Management, accessed July 19, 2025, https://www.heroesmodels.com/models/women/511-adit-priscilla/
2. Adit Priscilla - Model Profile - Photos & latest news, accessed July 19, 2025, https://models.com/models/adit-priscilla
3. My name screams power and success - Monitor, accessed July 19, 2025, https://www.monitor.co.ug/uganda/magazines/full-woman/my-name-screams-power-success-3338056
4. The Graduates: Adit Priscilla | models.com MDX, accessed July 19, 2025, https://models.com/mdx/the-graduates-adit-priscilla/
5. Meet @Adit Priscilla an international super model from the Joram Model... | TikTok, accessed July 19, 2025, https://www.tiktok.com/@nrgradioug/video/7398180556808244486
6. Aditi Rao Hydari - Wikipedia, accessed July 19, 2025, https://en.wikipedia.org/wiki/Aditi_Rao_Hydari
7. [2506.04158] Image Editing As Programs with Diffusion Models - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2506.04158
8. arXiv:2504.20690v1 [cs.CV] 29 Apr 2025, accessed July 19, 2025, [https://arxiv.org/pdf/2504.20690?](https://arxiv.org/pdf/2504.20690)
9. An Item is Worth a Prompt: Versatile Image Editing with Disentangled Control - arXiv, accessed July 19, 2025, https://arxiv.org/html/2403.04880v4
10. DiT4Edit: Diffusion Transformer for Image Editing - arXiv, accessed July 19, 2025, https://arxiv.org/html/2411.03286v2
11. DiT4Edit: Diffusion Transformer for Image Editing | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/390719977_DiT4Edit_Diffusion_Transformer_for_Image_Editing
12. ControlNet - Hugging Face, accessed July 19, 2025, https://huggingface.co/docs/diffusers/api/pipelines/controlnet
13. Exploring ControlNet: A New Perspective - Bria Blog, accessed July 19, 2025, https://blog.bria.ai/exploring-controlnet-a-new-perspective
14. DiT4Edit: Diffusion Transformer for Image Editing - hkust (gz), accessed July 19, 2025, https://cislab.hkust-gz.edu.cn/media/documents/_AAAI_2025__DiT4Edit_Camera_Ready.pdf
15. [Literature Review] DiT4Edit: Diffusion Transformer for Image Editing, accessed July 19, 2025, https://www.themoonlight.io/en/review/dit4edit-diffusion-transformer-for-image-editing
16. facebookresearch/DiT: Official PyTorch Implementation of "Scalable Diffusion Models with Transformers" - GitHub, accessed July 19, 2025, https://github.com/facebookresearch/DiT
17. Lazy Diffusion Transformer for Interactive Image Editing, accessed July 19, 2025, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/03436.pdf
18. DiffiT: Diffusion Vision Transformers for Image Generation - OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=uAKk0I3xxm
19. DiT4Edit: Diffusion Transformer for Image Editing - arXiv, accessed July 19, 2025, https://arxiv.org/html/2411.03286v1
20. arXiv:2502.11974v1 [cs.CV] 17 Feb 2025, accessed July 19, 2025, [https://arxiv.org/pdf/2502.11974?](https://arxiv.org/pdf/2502.11974)
21. [Literature Review] Image Inversion: A Survey from GANs to Diffusion and Beyond, accessed July 19, 2025, https://www.themoonlight.io/en/review/image-inversion-a-survey-from-gans-to-diffusion-and-beyond
22. Towards Understanding Cross and Self ... - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Liu_Towards_Understanding_Cross_and_Self-Attention_in_Stable_Diffusion_for_Text-Guided_CVPR_2024_paper.pdf
23. Towards Understanding Cross and Self-Attention in Stable Diffusion for Text-Guided Image Editing - arXiv, accessed July 19, 2025, https://arxiv.org/html/2403.03431v1
24. [2403.03431] Towards Understanding Cross and Self-Attention in Stable Diffusion for Text-Guided Image Editing - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2403.03431
25. [Papierüberprüfung] Towards Understanding Cross and Self-Attention in Stable Diffusion for Text-Guided Image Editing, accessed July 19, 2025, https://www.themoonlight.io/de/review/towards-understanding-cross-and-self-attention-in-stable-diffusion-for-text-guided-image-editing
26. Attention in Diffusion Model: A Survey - arXiv, accessed July 19, 2025, https://arxiv.org/html/2504.03738v1
27. High-Precision Image Editing via Dual Attention Control in Diffusion Models Without Fine-Tuning - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/388292246_High-Precision_Image_Editing_via_Dual_Attention_Control_in_Diffusion_Models_Without_Fine-Tuning
28. High-Precision Image Editing via Dual Attention Control in Diffusion Models Without Fine-Tuning - MDPI, accessed July 19, 2025, https://www.mdpi.com/2076-3417/15/3/1079
29. Enabling Instructional Image Editing with In-Context Generation in Large Scale Diffusion Transformer - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2504.20690
30. Enabling Instructional Image Editing with In-Context Generation in Large Scale Diffusion Transformer - arXiv, accessed July 19, 2025, https://arxiv.org/html/2504.20690v1
31. In-Context Edit: Enabling Instructional Image Editing with In-Context Generation in Large Scale Diffusion Transformer - Hugging Face, accessed July 19, 2025, https://huggingface.co/papers/2504.20690
32. [Literature Review] In-Context Edit: Enabling Instructional Image ..., accessed July 19, 2025, https://www.themoonlight.io/en/review/in-context-edit-enabling-instructional-image-editing-with-in-context-generation-in-large-scale-diffusion-transformer
33. [Literature Review] Image Editing As Programs with Diffusion Models, accessed July 19, 2025, https://www.themoonlight.io/en/review/image-editing-as-programs-with-diffusion-models
34. Image Editing As Programs with Diffusion Models - GitHub Pages, accessed July 19, 2025, https://yujiahu1109.github.io/IEAP/
35. [D] What are the advantages of GANs over Diffusion Models in ..., accessed July 19, 2025, https://www.reddit.com/r/MachineLearning/comments/184j8c3/d_what_are_the_advantages_of_gans_over_diffusion/
36. (PDF) Text-Controlled GAN Inversion for Image Editing - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/371850764_Text-Controlled_GAN_Inversion_for_Image_Editing
37. [2111.03186] EditGAN: High-Precision Semantic Image Editing - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2111.03186
38. In-Domain GAN Inversion for Real Image Editing, accessed July 19, 2025, https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123620579.pdf
39. Focus on Your Instruction: Fine-grained and Multi-instruction Image Editing by Attention Modulation - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Guo_Focus_on_Your_Instruction_Fine-grained_and_Multi-instruction_Image_Editing_by_CVPR_2024_paper.pdf
40. SDEdit Project Page, accessed July 19, 2025, https://sde-image-editing.github.io/
41. SDEdit: Guided Image Synthesis and Editing with Stochastic Differential Equations - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2108.01073
42. ControlNet: A Complete Guide - Stable Diffusion Art, accessed July 19, 2025, https://stable-diffusion-art.com/controlnet/
43. lllyasviel/ControlNet: Let us control diffusion models! - GitHub, accessed July 19, 2025, https://github.com/lllyasviel/ControlNet
44. Ctrl-Adapter, accessed July 19, 2025, https://ctrl-adapter.github.io/
45. CTRL-ADAPTER: AN EFFICIENT AND VERSATILE FRAMEWORK, accessed July 19, 2025, https://openreview.net/pdf/11db5372825f5359073c96995089ba4be244bc11.pdf
46. Ctrl-Adapter: An Efficient and Versatile Framework for Adapting Diverse Controls to Any Diffusion Model | Jaemin Cho, accessed July 19, 2025, https://j-min.io/publication/ctrl-adapter_iclr2025/
47. Ctrl-Adapter: An Efficient and Versatile Framework for Adapting Diverse Controls to Any Diffusion Model | Papers With Code, accessed July 19, 2025, https://paperswithcode.com/paper/ctrl-adapter-an-efficient-and-versatile
48. Ctrl-Adapter: An Efficient and Versatile Framework for Adapting Diverse Controls to Any Diffusion Model | OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=ny8T8OuNHe
49. dit / GitHub Topics, accessed July 19, 2025, https://github.com/topics/dit
50. stepfun-ai/Step1X-Edit: A SOTA open-source image editing model, which aims to provide comparable performance against the closed-source models like GPT-4o and Gemini 2 Flash. - GitHub, accessed July 19, 2025, https://github.com/stepfun-ai/Step1X-Edit
51. What evaluation metrics are commonly used for diffusion models? - Milvus, accessed July 19, 2025, https://milvus.io/ai-quick-reference/what-evaluation-metrics-are-commonly-used-for-diffusion-models
52. CLIP score vs FID pareto curves | dalle-mini – Weights & Biases - Wandb, accessed July 19, 2025, https://wandb.ai/dalle-mini/dalle-mini/reports/CLIP-score-vs-FID-pareto-curves--VmlldzoyMDYyNTAy
53. Performance Metrics in Evaluating Stable Diffusion Models - Medium, accessed July 19, 2025, https://medium.com/@seo.germany/performance-metrics-in-evaluating-stable-diffusion-models-4ca8bfdcc2ba
54. Rethinking FID: Towards a Better Evaluation Metric for Image Generation - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Jayasumana_Rethinking_FID_Towards_a_Better_Evaluation_Metric_for_Image_Generation_CVPR_2024_paper.pdf
55. Rethinking FID: Towards a Better Evaluation Metric for Image Generation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2401.09603v2
56. DiTFastAttn: Attention Compression for Diffusion Transformer Models - NIPS, accessed July 19, 2025, https://proceedings.neurips.cc/paper_files/paper/2024/file/0267925e3c276e79189251585b4100bf-Paper-Conference.pdf
57. DiffEditor: Boosting Accuracy and Flexibility on Diffusion-based Image Editing - CVPR 2024 Open Access Repository, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2024/html/Mou_DiffEditor_Boosting_Accuracy_and_Flexibility_on_Diffusion-based_Image_Editing_CVPR_2024_paper.html
58. DiffEditor: Boosting Accuracy and Flexibility on Diffusion-based Image Editing - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Mou_DiffEditor_Boosting_Accuracy_and_Flexibility_on_Diffusion-based_Image_Editing_CVPR_2024_paper.pdf
59. CVPR Poster DragDiffusion: Harnessing Diffusion Models for Interactive Point-based Image Editing, accessed July 19, 2025, https://cvpr.thecvf.com/virtual/2024/poster/31643
60. Exploring Low-Dimensional Subspace in Diffusion Models for Controllable Image Editing | OpenReview, accessed July 19, 2025, [https://openreview.net/forum?id=50aOEfb2km¬eId=QVA8eO5JDt](https://openreview.net/forum?id=50aOEfb2km&noteId=QVA8eO5JDt)
61. tamlhp/awesome-instruction-editing: Awesome Instruction Editing. Image and Media Editing with Human Instructions. Instruction-Guided Image and Media Editing. - GitHub, accessed July 19, 2025, https://github.com/tamlhp/awesome-instruction-editing
62. ACE: All-round Creator and Editor Following Instructions via Diffusion Transformer, accessed July 19, 2025, https://openreview.net/forum?id=Bpn8q40n1n


# 옴니모달 AI의 작동 원리
[Modality](./index.md)


인공지능(AI)이 인간과 유사한 수준의 이해력에 도달하기 위한 여정은 정보를 인식하고 처리하는 방식의 근본적인 진화를 통해 이루어졌다. 초기 AI 시스템의 단편적인 인지 방식에서부터 오늘날의 통합적 지능에 이르기까지, 각 단계는 이전 패러다임의 한계를 극복하기 위한 필연적인 과정이었다.


초기 AI 모델은 본질적으로 단일 모달(Unimodal) 방식으로 작동했다.1 이는 텍스트, 이미지, 음성과 같은 단 하나의 데이터 유형, 즉 모달리티(modality)만을 처리하도록 설계되었음을 의미한다. 대표적인 예로, 이미지를 분석하기 위한 합성곱 신경망(Convolutional Neural Networks, CNNs)이나 텍스트와 같은 순차적 데이터를 처리하기 위한 순환 신경망(Recurrent Neural Networks, RNNs) 및 초기 트랜스포머(Transformer) 모델이 있다.1

이러한 단일 모달 모델들은 특정 작업에서는 강력한 성능을 발휘했지만, 세상을 전체적으로 이해하는 데에는 명백한 한계를 보였다. 각 모델은 독립된 정보의 "사일로(silo)" 안에서 작동했다.3 예를 들어, 텍스트만 처리하는 모델은 자신이 묘사하는 대상의 시각적 형태나 실제 모습을 이해할 수 없었고, 이미지 모델은 장면의 언어적 맥락이나 개념적 관계를 추론할 능력이 부족했다.4 이러한 한계는 정확한 예측과 깊이 있는 추론에 필수적인 다른 감각의 보조 정보가 결여되어 있어, 문맥 이해 능력에 심각한 약점으로 작용했다.1


멀티모달(Multimodal) AI는 단일 모달의 한계를 극복하기 위해 등장했다. 인간이 시각, 청각, 촉각 등 다양한 감각을 동시에 사용하여 세상을 인식하는 것처럼, 멀티모달 AI는 텍스트, 이미지, 음성, 제스처 등 여러 채널의 데이터를 동시에 처리하여 보다 포괄적인 이해에 도달하고자 했다.4 초기 멀티모달 시스템은 이미지와 텍스트 입력을 결합하여 더 정교한 검색 결과를 제공하는 것과 같은 형태로 구현되었다.7

이 1세대 멀티모달 AI는 종종 여러 개의 개별적인 단일 모달 모델을 "이어 붙이는(stitching together)" 방식으로 구성되었다.10 여기서 핵심적인 기술적 과제는 바로 **데이터 융합(Data Fusion)**, 즉 서로 다른 정보 흐름을 어떻게 효과적으로 통합할 것인가의 문제였다. 이를 해결하기 위해 초기 융합(Early Fusion), 후기 융합(Late Fusion), 그리고 하이브리드 융합(Hybrid Fusion)이라는 세 가지 주요 전략이 발전했다.


데이터 융합 방식은 멀티모달 시스템의 성능과 특성을 결정하는 핵심 요소이다. 각 전략은 장단점과 적합한 활용 사례가 명확히 구분된다.

- **초기 융합 (Early Fusion)**: 특징 수준 융합(feature-level fusion)이라고도 불리는 이 방식은 각 모달리티에서 추출된 특징(feature)들을 주 모델에 입력하기 *전* 단계에서 하나의 통합된 특징 벡터로 결합한다.11 이를 통해 모델은 초기부터 데이터 간의 공동 표현(joint representation)을 학습할 수 있다. 이 전략의 강점은 모달리티 간의 깊은 상호 연관성을 포착할 수 있다는 점이다. 하지만 비디오 프레임과 오디오 타임스탬프를 정확히 일치시키는 것과 같이 데이터 간의 정밀한 동기화와 정렬이 필수적이며, 하나의 모달리티에 노이즈가 있거나 데이터가 누락될 경우 전체 시스템이 취약해지는 단점이 있다.12
- **후기 융합 (Late Fusion)**: 결정 수준 융합(decision-level fusion)으로도 알려진 이 전략은 각 모달리티를 독립적인 개별 모델로 처리한 후, 각 모델이 내놓은 예측이나 결정 값을 최종 단계에서 통합한다. 통합 방식으로는 주로 투표(voting)나 가중 평균(weighted averaging) 등이 사용된다.6 이 접근법은 구현이 비교적 간단하고, 특정 모달리티가 누락되더라도 시스템이 비교적 안정적으로 작동하는 유연성을 가진다. 그러나 융합이 너무 늦은 단계에서 일어나기 때문에, 모달리티 간의 미묘하고 낮은 수준의 상호작용을 놓칠 수 있다는 치명적인 단점이 있다.12 이는 통합된 이해라기보다는 전문가들의 예측을 조율하는 것에 가깝다.
- **하이브리드/중간 융합 (Hybrid/Intermediate Fusion)**: 이 전략은 초기 융합과 후기 융합의 장점을 모두 취하려는 시도로, 모델 아키텍처 내의 다양한 중간 단계에서 모달리티를 결합한다.6 예를 들어, 한 모달리티의 특징을 다른 모달리티의 처리 과정 중간에 주입하는 방식이다. 이는 균형 잡힌 접근법을 제공할 수 있지만, 아키텍처가 훨씬 복잡해져 설계와 튜닝이 어렵다는 단점이 있다.12 현대 트랜스포머 모델의 어텐션 메커니즘은 이러한 중간 융합의 정교한 형태로 볼 수 있다.15

| 표 1: 데이터 융합 전략 비교 분석 |      |
| -------------------------------- | ---- |
| **융합 전략**                    |      |
| **초기 융합**                    |      |
| **후기 융합**                    |      |
| **하이브리드/중간 융합**         |      |

자료: 6 기반 재구성


'옴니모달(Omnimodal)'이라는 용어는 단순히 여러 모달리티를 결합하는 것을 넘어선 패러다임의 전환을 의미한다. 이는 텍스트, 이미지, 비디오, 오디오 등 모든 핵심 모달리티를 **공유된 내부 표현(shared internal representation)** 내에서 유연하고 동시에 인지, 추론, 생성하도록 설계된 **단일 통합 모델(single, unified model)**을 지칭한다.3

이 용어의 의미는 기술 발전과 함께 진화했다. 2022년 네이버의 '옴니서치(OmniSearch)'와 같은 초기 상업적 사례에서 '옴니(omni)'는 이미지 검색에 텍스트를 추가하는 것과 같은 고급 멀티모달 검색 기능을 의미했다.8 그러나 이후 연구 커뮤니티에서는 이 용어를 보다 엄격하게 정의했다. 즉, 여러 개의 단일 모달 전문가 모델을 "볼트로 조립한" 것과 같은 전통적인 멀티모달 시스템과 달리 10, 옴니모달 모델은 마치 하나의 뇌가 연속적이고 통합된 인지 흐름을 처리하는 것처럼 작동한다. 진정한 옴니모달리티의 핵심은 처리하는 모달리티의 *수*가 아니라, 그것들을 처리하는 *방식*에 있다. 즉, 여러 모델의 조율이 아닌 단일 모델 내에서의 완전한 통합을 의미하며, 이러한 전체론적 접근은 범용 인공지능(AGI)을 향한 필수 전제 조건으로 간주된다.10


옴니모달 AI의 혁신은 이를 가능하게 하는 핵심 기술들의 융합에 기반한다. 특히 트랜스포머 아키텍처와 Mixture-of-Experts(MoE) 구조는 옴니모달 패러다임을 현실로 만든 두 개의 기둥이다. 이 기술들은 개별적으로도 중요하지만, 이들의 시너지가 진정한 통합 지능의 토대를 마련했다.


트랜스포머 아키텍처는 본래 기계 번역을 위해 개발되었으나, 병렬 처리 능력과 순환 신경망(RNN)의 부재로 인한 훈련 효율성 덕분에 대규모 AI 모델의 표준으로 자리 잡았다.2 트랜스포머의 핵심 혁신은 **어텐션 메커니즘(attention mechanism)**이다.2

어텐션 메커니즘의 가장 중요한 특징은 특정 모달리티에 구애받지 않는다는 점이다. 이 메커니즘은 본질적으로 숫자 벡터의 시퀀스, 즉 '토큰(token)'에 대해 작동한다.2 이미지, 오디오, 비디오와 같은 비텍스트 데이터를 토큰 시퀀스로 변환하기만 하면, 트랜스포머는 텍스트를 처리하는 것과 동일한 기본 논리를 적용하여 이들을 처리할 수 있다.15 이러한 유연성 덕분에 트랜스포머는 통합 모델의 이상적인 기반이 된다.

트랜스포머 내에서 어텐션은 두 가지 핵심적인 역할을 수행한다:

- **셀프 어텐션(Self-attention)**: 단일 모달리티 내에서 각 토큰의 중요도를 가늠하게 한다. 예를 들어, "빠른 갈색 여우"라는 문장에서 "빠른"과 "갈색"이라는 토큰이 "여우"라는 토큰과 더 밀접하게 관련되어 있음을 파악하는 식이다.15
- **크로스 어텐션(Cross-attention)**: 서로 다른 모달리티의 토큰들 간의 관계를 파악하게 한다. 예를 들어, 텍스트로 주어진 질문을 이미지의 특정 영역에 있는 토큰들과 연결하여 시각적 질의응답(VQA)을 수행할 수 있다.15


진정한 옴니모달 모델의 개념적 심장부는 **공유 잠재 공간(shared latent space)** 또는 **통합 임베딩(unified embedding)**이다.10 이는 모든 감각 데이터가 원래의 모달리티와 상관없이 투영되는 고차원의 벡터 공간이다. 이 공간에서는 데이터 포인트 간의 거리가 데이터의 유형이 아닌 **의미적 유사성(semantic similarity)**에 의해 결정된다.10

이 공유 잠재 공간은 모델이 여러 모달리티를 넘나들며 "함께 생각(think together)"하는 것을 가능하게 한다.10 예를 들어, 개의 이미지, 짖는 소리, 그리고 "개가 짖고 있다"라는 텍스트는 모두 이 공간 내에서 서로 가까운 위치에 매핑된다.10 이는 세 가지 결정적인 기능을 가능하게 한다:

1. **통합된 인지 (Unified Perception)**: 여러 감각 스트림을 하나의 일관된 "정신 모델(mental model)"로 융합한다.10
2. **진정한 교차 모달 추론 (Cross-Modal Reasoning)**: 모든 모달리티가 동일한 수학적 언어로 표현되기 때문에 모델은 모달리티 간 경계를 매끄럽게 넘나들며 추론할 수 있다. 입술의 움직임과 음성을, 또는 장면의 전환과 해설을 정렬할 수 있는 이유는 이들의 표현이 잠재 공간 내에서 함께 위치하고 동기화되기 때문이다.10
3. **통합된 생성 (Unified Generation)**: 모델은 더 이상 분리된 시스템 간에 토큰을 주고받는 것이 아니라, 단일한 내부 개념으로부터 복잡하고 여러 형식의 결과물을 생성할 수 있다.10 이 통합 임베딩의 품질은 모델 성능에 절대적이며, 분리된 표현 방식은 구식 모델의 핵심적인 약점으로 지적된다.19


트랜스포머가 통합 처리의 *방법론*을 제공했다면, Mixture-of-Experts(MoE) 아키텍처는 그 방법론을 현실적으로 구현 가능하게 만드는 *확장성*을 제공한다. 이 두 기술의 융합 없이는 현대적인 옴니모달 AI를 상상하기 어렵다.

- **MoE 아키텍처**: MoE는 모델의 전체 파라미터 수를 수조 개까지 확장하면서도 계산 비용을 관리 가능하게 만드는 혁신적인 아키텍처이다.20 이는 방대한 수의 "전문가(expert)" 하위 신경망과, 각 입력 토큰을 처리할 소수의 전문가 그룹을 동적으로 선택하는 "라우터(router)"로 구성된다.20 즉, 모델은 방대한 지식을 저장할 수 있는 용량을 갖추되, 특정 작업을 수행할 때는 전체 파라미터의 일부만 활성화하여 동일한 크기의 밀집 모델(dense model)보다 훨씬 효율적으로 작동한다.20 이는 텍스트, 비전, 오디오, 코드 등 수많은 영역의 지식을 저장해야 하는 옴니모달 모델에 필수적인 기술이다.
- **MoT (Mixture-of-Transformers) 아키텍처**: MoE 개념을 더욱 발전시킨 것이 MoT 아키텍처이다. 이 구조는 "모달리티별 파라미터 분리(modality-specific parameter decoupling)"라는 개념을 도입하여, 각 모달리티(텍스트, 이미지, 음성 등)에 대해 피드포워드 신경망이나 어텐션 행렬과 같은 파라미터 세트를 별도로 할당한다.22 여전히 전역적인 셀프 어텐션 메커니즘을 통해 모달리티 간의 관계를 포착하지만, 각 데이터 유형을 전담 전문가가 처리하도록 함으로써 훈련 과정에서의 충돌을 줄이고 보다 전문적이고 효과적인 처리를 가능하게 한다.22

결론적으로, 옴니모달 패러다임은 단일 기술의 산물이 아니라, 트랜스포머의 보편적 처리 능력과 MoE의 효율적 확장성이 결합된 결과물이다. 트랜스포머가 "어떻게" 통합할 것인지에 대한 답을 제공했다면, MoE는 그 통합을 "얼마나 크게" 할 수 있는지에 대한 답을 제공한 셈이다.


옴니모달 AI 모델이 원시 입력 데이터로부터 의미 있는 결과물을 생성하기까지의 과정은 여러 단계의 정교한 데이터 파이프라인을 거친다. 이 파이프라인의 핵심은 현실 세계의 다양한 정보를 트랜스포머가 이해할 수 있는 공통의 언어, 즉 '토큰'으로 변환하고, 이를 다시 인간이 인지할 수 있는 형태로 재구성하는 것이다.


토큰화(Tokenization)는 옴니모달 파이프라인의 첫 번째이자 가장 중요한 단계이다. 모델의 이해력은 근본적으로 토큰의 품질에 의해 제한되기 때문이다.18 토큰화의 목표는 연속적이고 고차원적인 모든 모달리티의 데이터를 트랜스포머가 처리할 수 있는 이산적인(discrete) 토큰 시퀀스로 변환하는 것이다.18 이 과정에서 사용되는 '토크나이저(tokenizer)'는 단순한 전처리 도구를 넘어, AI 시스템의 감각 기관과 같은 역할을 수행하는 핵심적인 학습 가능 구성 요소로 진화하고 있다.

| 표 2: 모달리티별 토큰화 및 특징 추출 기법 |      |
| ----------------------------------------- | ---- |
| **모달리티**                              |      |
| **텍스트**                                |      |
| **이미지**                                |      |
| **오디오**                                |      |
| **비디오**                                |      |

자료: 12 기반 재구성

- **시각적 토큰화 (Vision Transformers - ViT)**: 이미지를 처리하기 위해 ViT는 먼저 이미지를 고정된 크기(예: 16x16 픽셀)의 겹치지 않는 패치(patch)들의 그리드로 분할한다.24 각 패치는 픽셀 값으로 이루어진 긴 벡터로 "평탄화(flattened)"된 후, 선형 투영(linear projection)을 통해 저차원의 "패치 임베딩"으로 변환된다. 이때, 원래의 공간적 정보를 보존하기 위해 각 패치 임베딩에 학습 가능한 "위치 임베딩(positional embedding)"이 더해진다.24 이 전체 과정은 합성곱(convolutional) 레이어를 통해 효율적으로 구현될 수 있다.24 즉, 이미지는 '단어' 패치들의 시퀀스로 취급되어 트랜스포머가 처리할 수 있는 형태로 변환된다.
- **오디오 토큰화 (신경망 코덱)**: 연속적인 오디오 파형은 벡터 양자화(Vector Quantization, VQ)와 SoundStream, EnCodec과 같은 고급 신경망 오디오 코덱을 사용하여 이산 토큰으로 변환된다.23 인코더 신경망이 짧은 오디오 구간을 특징 벡터로 변환하면, VQ 레이어가 각 벡터를 학습된 "코드북(codebook)"에서 가장 가까운 항목에 매핑하고, 해당 항목의 인덱스가 이산 토큰이 된다.23 특히 SoundStream은 완전한 합성곱 인코더/디코더와 잔차 벡터 양자화기(Residual Vector Quantizer, RVQ)를 사용하여 고품질의 압축 및 토큰화를 달성한다. RVQ는 여러 계층의 코드북을 사용하여 적은 코드북 크기로도 신호를 높은 충실도로 표현할 수 있어 매우 효율적이다.27
- **비디오 토큰화**: 비디오 토큰화는 이미지 토큰화에 시간적 차원을 추가하여 확장한 것이다. 일반적인 기법으로는 각 프레임을 하나의 토큰으로 취급하는 프레임 수준 토큰화, 여러 프레임 그룹을 하나의 토큰으로 묶는 클립 수준 토큰화, 그리고 프레임 전반에 걸쳐 특정 객체를 추적하고 토큰화하는 객체 수준 토큰화 등이 있다.28 종종 오디오는 시각적 움직임과 함께 시간에 맞춰 동기화되어 직접 임베딩된다.10 긴 비디오를 시간적 맥락을 잃지 않고 효율적으로 토큰화하는 것은 여전히 활발한 연구 분야이며, Gemini와 같은 모델이 긴 비디오를 처리할 수 있는 능력은 프레임당 더 적은 수의 토큰을 사용하는 효율적인 토큰화 기술 덕분이다.29


모든 모달리티가 토큰화되어 공유 잠재 공간에 투영되면, 진정한 의미의 융합이 이루어진다. 최신 옴니모달 모델에서 이는 별도의 "융합 모듈"을 통하는 것이 아니라, 혼합된 모달리티 시퀀스에 대해 작동하는 트랜스포머의 어텐션 메커니즘의 창발적 속성(emergent property)으로 나타난다.15

셀프 어텐션과 크로스 어텐션 레이어는 자연스럽게 정교한 형태의 중간 융합을 수행한다. 모델은 서로 관련된 다른 모달리티의 토큰들에 동적으로 "주의를 기울이는(attend)" 법을 학습한다. 예를 들어, "강아지가 무엇을 하고 있나요?"라는 텍스트를 처리할 때, 크로스 어텐션 메커니즘은 이미지에서 강아지에 해당하는 패치 토큰들의 가중치를 높여 시각적 정보와 텍스트 질의를 효과적으로 융합한다.15 이러한 동적이고 학습된 정렬 방식은 초기 또는 후기 융합의 정적인 규칙보다 훨씬 강력하다.


생성은 토큰화의 역과정이다. 모델은 자기회귀적(autoregressively)으로 시퀀스에서 다음 토큰을 예측하며, 이 생성된 토큰들은 어떤 모달리티에도 속할 수 있다.18

생성된 토큰 시퀀스는 모달리티별 "디토크나이저(de-tokenizer)" 또는 디코더로 전달된다.18 텍스트 토큰 시퀀스는 텍스트 렌더러로, 이미지 토큰 시퀀스는 시각적 디코더에 의해 다시 이미지로 변환된다. 오디오 토큰 시퀀스는 SoundStream의 디코더와 같은 신경망 보코더(vocoder)에 입력되어 오디오 파형을 합성한다.18 전체 생성 과정이 통합된 잠재 공간에서 비롯되기 때문에, 모델은 동기화된 오디오와 관련 자막이 포함된 비디오와 같이 모달리티 간에 일관성 있는 결과물을 생성할 수 있다.10


추상적인 원리들을 실제 세계의 최첨단 예시를 통해 구체화하기 위해, 구글의 Gemini 모델 제품군을 심층적으로 분석한다. Gemini는 1부에서 3부까지 논의된 옴니모달 AI의 핵심 원칙들이 어떻게 구현되고 작동하는지를 명확하게 보여주는 사례이다.


Gemini 1.5 Pro 및 2.5 Pro를 포함한 Gemini 제품군은 처음부터 텍스트, 오디오, 이미지, 비디오 입력을 혼합하여 처리하도록 설계된 **네이티브 멀티모달(natively multimodal), 희소 Mixture-of-Experts(sparse MoE) 트랜스포머 모델**이다.20

Gemini의 아키텍처는 2부에서 설명한 원칙들을 완벽하게 구현한다. 기본적으로 트랜스포머를 사용하고, 효율적인 확장을 위해 MoE 구조를 채택했으며, 네이티브 멀티모달리티를 달성하기 위해 통합된 표현 방식을 중심으로 구축되었다.32 Gemini의 가장 두드러진 특징 중 하나는 방대한 **컨텍스트 창(context window)**이다. 상용 버전에서는 최대 1백만 토큰, 연구용으로는 최대 1천만 토큰까지 처리할 수 있어, 단일 프롬프트 내에서 전체 코드베이스나 수 시간 분량의 비디오를 분석하는 것이 가능하다.20 이러한 장문 컨텍스트 처리 능력은 아키텍처 변경과 함께 더욱 효율적인 비디오 토큰화(프레임당 66개의 토큰 사용)와 같은 기술적 진보의 직접적인 결과물이다.21


최첨단 옴니모달 AI의 개발은 순수한 모델 아키텍처 설계를 넘어, 이제는 고도로 전문화된 **인프라 및 시스템 엔지니어링**의 문제로 진화했다. Gemini 2.5 제품군은 구글의 **TPUv5p 아키텍처**에서 훈련된 최초의 모델로, 그 훈련 과정에는 모델 자체만큼이나 중요한 여러 인프라 혁신이 포함되어 있다.31

1. **슬라이스 단위 탄력성 (Slice-Granularity Elasticity)**: 대규모 훈련 중 일부 TPU '슬라이스'에서 국지적인 하드웨어 장애가 발생하더라도, 시스템이 자동으로 더 적은 수의 슬라이스로 훈련을 계속 진행한다. 이로 인해 훈련 중단 시간이 수 분에서 수십 초로 단축되고, 약 97%의 처리량을 유지할 수 있다.31
2. **분할 단계 SDC 탐지 (Split-Phase SDC Detection)**: 하드웨어에서 발생하는 '조용한 데이터 손상(Silent Data Corruption)'을 신속하게 탐지하고 격리하여, 훈련 데이터의 오염을 방지하는 시스템이다.31

이러한 혁신은 단순한 편의성 개선이 아니다. 수만 개의 프로세서에서 장기간에 걸쳐 수행되는 대규모 훈련에서는 하드웨어 장애와 데이터 오류가 통계적으로 필연적이다. 전통적인 시스템에서는 단 한 번의 장애로도 전체 훈련이 중단되어 막대한 비용과 시간을 낭비할 수 있다. 구글의 접근 방식은 이러한 필연적인 장애에 대한 복원력을 갖춘 시스템을 구축하는 것으로, 이는 AI 선도 기업의 경쟁력이 더 이상 모델 설계나 데이터셋에만 있는 것이 아니라, 수십억 달러 규모의 특화된 하드웨어 및 소프트웨어 스택 전체에 있음을 시사한다.

알고리즘 측면에서도 주목할 만한 혁신이 있다. 대표적인 것이 **"사고(Thinking)" 능력**이다. Gemini 2.5 모델은 강화학습(RL)을 통해 추론 시 추가적인 컴퓨팅 자원(수만 번의 순방향 패스)을 사용하여 어려운 문제에 대해 더 깊이 "생각"한 후 답변을 제공하도록 훈련되었다.29 이는 추론당 고정된 계산 예산에서 벗어나, 문제의 난이도에 따라 동적으로 사고의 깊이를 조절하는 패러다임의 전환을 의미한다.


Gemini의 멀티모달 추론 능력은 공식 데모와 실제 사용자 테스트에서 상반된 모습을 보인다. 이는 현재 기술의 가능성과 한계를 동시에 보여준다.

- **성공 사례**: 공식 데모에서는 Gemini가 연속된 이미지를 분석하여 셸 게임(shell game)에서 물체의 위치를 추적하고 34, 여행 브이로그(vlog)를 분석하여 타임스탬프가 포함된 블로그 게시물을 생성하며 35, 악기 그림을 보고 음악 장르를 추천하고 검색 쿼리를 생성하는 등 인상적인 능력을 보여준다.34 또한 최대 3시간 분량의 비디오 콘텐츠를 한 번에 처리할 수 있다.36
- **한계점**: 반면, 한 사용자가 TV 드라마 에피소드 전체를 입력하여 테스트한 결과, Gemini 1.5 Pro는 전체 줄거리를 요약하고 주요 인물을 식별하는 데는 성공했지만, 세부 사항에서는 오류를 보였다.37 인물들을 혼동하고, 영상에 존재하지 않는 장면을 환각(hallucination)처럼 만들어냈으며, 태국어로 작성된 광고를 제대로 이해하지 못하는 등 비영어권 시각적 텍스트 인식에도 한계를 드러냈다.37

이러한 결과는 옴니모달 AI의 추론 능력이 강력하지만 아직 완벽하지 않다는 점을 명확히 보여준다. 모델은 논리적 비약을 통해 복잡한 추론을 해낼 수 있지만, 동시에 세부 사항을 오해하거나 불확실할 때 정보를 지어낼 수 있다. 이는 특히 복잡하고 긴 형식의 멀티모달 입력을 다룰 때, 환각을 완화하고 견고하며 신뢰할 수 있는 추론 능력을 확보하는 것이 여전히 중요한 과제로 남아 있음을 강조한다.11


옴니모달 AI는 실험실을 넘어 다양한 산업 분야에 혁신적인 변화를 가져오고 있다. 그러나 그 잠재력을 완전히 실현하기 위해서는 해결해야 할 중대한 기술적, 윤리적 과제들이 남아 있으며, 연구 커뮤니티는 이미 다음 단계의 지능을 향해 나아가고 있다.


- **헬스케어**: 옴니모달 AI는 의료 영상(X선, CT), 전자의무기록(EHR), 병리 슬라이드, 심지어 환자의 기침 소리와 같은 음성 데이터를 통합하여 진단을 위한 전체적인 그림을 제공한다.11 이를 통해 방사선 전문의나 병리학자가 단독으로 수행하는 것 이상으로, 시각적 데이터와 유전적 마커 또는 임상 기록을 연관시켜 더 정확하고 빠른 질병(예: 암) 조기 발견과 개인 맞춤형 치료 계획 수립이 가능해진다.38

- **자동차 / 자율주행**: 자율주행 시스템은 옴니모달 AI의 고전적인 응용 분야이다. 카메라(시각), 라이다(LiDAR, 정밀 깊이), 레이더(전천후 객체 탐지), 초음파 센서(근거리) 등 다양한 센서의 데이터를 융합하여 주변 환경에 대한 견고한 3D 모델을 구축한다.11 이 시스템의 핵심 이점은 

  **안전성**과 **중복성(redundancy)**이다. 예를 들어, 카메라가 햇빛에 의해 시야가 가려지는 경우 다른 센서들이 그 공백을 보완할 수 있다. 이러한 융합은 복잡하고 동적인 환경에서 신뢰할 수 있는 객체 탐지, 경로 계획, 의사 결정에 필수적이다.40

- **기업 및 금융**: 에이전트 AI 플랫폼은 옴니모달 기능을 사용하여 복잡한 엔드-투-엔드 워크플로우를 자동화한다. 비정형 문서(PDF, 보고서)를 처리하고, 데이터베이스를 쿼리하며, 대화형 챗봇을 구동하고, 다양한 소스의 정보를 통합하여 보고서를 생성할 수 있다.42 이는 단일 작업 자동화를 넘어 지능형 프로세스 자동화로의 전환을 의미한다. 예를 들어, AI 에이전트는 손상 사진을 분석하고, 사용자의 이메일 설명을 읽고, 데이터베이스에서 보험 증권을 확인한 후, 승인서를 생성하는 방식으로 전체 보험 청구 프로세스를 처리할 수 있다.42


- **데이터 정렬 및 동기화**: 서로 다른 모달리티의 데이터가 완벽하게 정렬된 대규모 고품질 데이터셋을 구축하는 것은 근본적인 어려움이다. 예를 들어, 대본의 특정 단어가 비디오에서 해당 단어를 말하는 입술 움직임의 정확한 프레임과 일치하도록 보장해야 한다.44 부정확한 정렬은 모델이 잘못된 상관관계를 학습하게 할 수 있으며, 수동 정렬은 막대한 비용이 들어 웹에서 수집된 노이즈가 많은 데이터나 자기지도 학습 기법에 의존하게 만든다.45
- **계산 비용 및 확장성**: 최첨단 모델을 훈련하는 데는 수천만 달러의 비용이 소요되며 46, 추론 비용 또한 상당하다. GPT-4.5와 같은 최고 수준의 모델은 1백만 입력 토큰당 75달러의 비용이 발생할 수 있다.47 이러한 높은 비용은 소수의 거대 기술 기업에 힘을 집중시키고 AI의 민주화에 대한 의문을 제기하는 진입 장벽으로 작용한다. MoE와 같은 아키텍처가 이에 대한 대응책이지만, 전반적인 비용은 여전히 막대하다.20
- **평가 및 벤치마킹**: AI 모델의 발전 속도가 너무 빨라 기존 벤치마크들이 그 성능을 제대로 측정하지 못하고 빠르게 "포화(saturating)"되고 있다.36 이는 AI 평가 분야의 "삼중고(trilemma)", 즉 넓은 범위, 낮은 비용, 데이터 오염 방지를 동시에 만족하는 벤치마크를 만들기 어렵다는 문제로 이어진다.48 모델의 성능을 정확히 측정할 수 없다면 연구 개발 방향을 올바르게 이끌 수 없기 때문에, 이는 AI 분야의 심각한 병목 현상이다. 이러한 문제의식 속에서 모델의 발전과 AI 평가는 서로의 발전을 촉진하는 공진화 관계에 놓이게 되었다. 더 나은 모델을 만들기 위해서는 동시에 더 나은 평가 척도를 만들어야만 한다. 이에 따라 OmniEval, Multimodal LIVEBENCH와 같이 더 도전적이고 동적이며 실제 세계를 반영하는 새로운 벤치마킹 프레임워크에 대한 연구가 활발히 진행되고 있다.48
- **윤리적 고려사항 및 환각**: 옴니모달 모델은 훈련 데이터에 존재하는 편견을 학습하고 증폭시킬 수 있으며, 한 모달리티의 편견을 다른 모달리티로 전파할 수도 있다.3 또한, 여러 데이터 유형 간의 복잡한 상호작용으로 인해 비상식적이거나 잘못된 결과물, 즉 '환각'을 생성할 위험이 높아질 수 있다.11 이는 특히 헬스케어나 법 집행과 같은 민감한 분야에서 심각한 위험을 초래할 수 있으므로 11, 개인정보 보호, 투명성, 통제 가능성을 확보하는 것이 중요한 사회-기술적 과제로 남아있다.3


연구의 최전선은 수동적인 콘텐츠 생성을 넘어, 디지털 및 물리적 세계와의 능동적인 상호작용으로 이동하고 있다. 이는 모달리티 간의 경계가 완전히 허물어지고, 인지, 추론, 행동의 통합된 흐름으로 대체되는 미래를 예고한다.

- **전이중 상호작용 (Full-Duplex Interaction)**: 현재의 순차적인 대화 방식에서 벗어나, 인간처럼 입력과 출력을 동시에 처리하는 모델에 대한 연구가 진행 중이다. RoboEgo와 같은 모델은 낮은 지연 시간의 전이중 스트리밍을 목표로 설계되었다.50
- **시각-언어-행동 (Vision-Language-Action, VLA) 모델**: 인지(시각, 언어)와 로보틱스의 융합을 의미한다. VLA 모델은 세상을 보고 묘사하는 것을 넘어, 그 안에서 물리적으로 타당하고 근거 있는 행동을 생성하도록 설계되었다.17
- **생성형 월드 모델 (Generative World Models)**: 궁극적인 목표는 물리 세계의 기본 원리를 내재화한 내부 시뮬레이션을 구축하는 모델을 만드는 것이다. 이를 통해 상호작용 가능한 환경을 생성하고 미래 상태를 예측할 수 있다.17

이러한 연구 동향은 NeurIPS, ICML, CVPR과 같은 최고 수준의 AI 학회에서 발표되는 논문들을 통해 명확히 확인되며 17, AI의 최종 목표가 체화된 에이전트 AI(embodied, agentic AI)임을 시사한다.

| 표 3: 주요 옴니모달 모델 역량 및 아키텍처 요약 |      |
| ---------------------------------------------- | ---- |
| **모델**                                       |      |
| **Google Gemini 2.5 Pro**                      |      |
| **OpenAI GPT-4o**                              |      |
| **Ming-Omni**                                  |      |
| **Ola**                                        |      |
| **RoboEgo**                                    |      |

자료: 6 기반 재구성


1. Unimodal vs. Multimodal AI: Key Differences Explained - Index.dev, accessed July 16, 2025, https://www.index.dev/blog/comparing-unimodal-vs-multimodal-models
2. Transformer (deep learning architecture) - Wikipedia, accessed July 16, 2025, https://en.wikipedia.org/wiki/Transformer_(deep_learning_architecture)
3. Omnimodal AI - Hello Future - Orange, accessed July 16, 2025, https://hellofuture.orange.com/en/omnimodal-ai/
4. 인간처럼 사고하는 멀티모달 Multi Modal AI란? | 인사이트리포트 | 삼성SDS, accessed July 16, 2025, https://www.samsungsds.com/kr/insights/multi-modal-ai.html
5. 멀티모달(Multi Modal)AI와 기존 인공지능의 차이점 - 클루닉스, accessed July 16, 2025, https://www.clunix.com/insight/it_trends.php?boardid=ittrend&mode=view&idx=824
6. What is Multimodal AI? | IBM, accessed July 16, 2025, https://www.ibm.com/think/topics/multimodal-ai
7. 멀티모달 AI, accessed July 16, 2025, https://cloud.google.com/use-cases/multimodal-ai?hl=ko
8. 네이버, 스마트렌즈에 멀티모달 AI 적용..."이미지로 신발 검색" - 지디넷코리아, accessed July 16, 2025, https://zdnet.co.kr/view/?no=20220428095442
9. 네이버, 멀티모달 AI 적용으로 이미지와 텍스트 동시 검색... 사용자 편의 극대화 (2022.04.28) - 자료실 < 아카이브 | 판교테크노밸리, accessed July 16, 2025, https://www.pangyotechnovalley.org/base/board/read?boardManagementNo=16&boardNo=655&menuLevel=2&menuNo=15
10. OmniModels: The Unified Architecture for Intelligence | by Tyler Frink ..., accessed July 16, 2025, https://medium.com/@frinktyler1445/omnimodels-the-unified-architecture-for-intelligence-89856a63e975
11. Multimodal AI: Transforming Evaluation & Monitoring - Galileo AI, accessed July 16, 2025, https://galileo.ai/blog/multimodal-ai-guide
12. Multimodal Data Fusion: Key Techniques, Challenges & Solutions - Sapien, accessed July 16, 2025, https://www.sapien.io/blog/mastering-multimodal-data-fusion
13. (PDF) Multimodal Data Fusion Techniques - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/publication/383887675_Multimodal_Data_Fusion_Techniques
14. What fusion strategies work best for combining results from different modalities? - Milvus, accessed July 16, 2025, https://milvus.io/ai-quick-reference/what-fusion-strategies-work-best-for-combining-results-from-different-modalities
15. What is the role of transformers in multimodal AI? - Milvus, accessed July 16, 2025, https://milvus.io/ai-quick-reference/what-is-the-role-of-transformers-in-multimodal-ai
16. (챗지피티) 옴니모달 - 산들바람 - Daum 카페, accessed July 16, 2025, https://table-m.cafe.daum.net/p/2945409216/441629234120076288
17. What is Next for Multimodal AI. Multimodal AI is evolving from static ..., accessed July 16, 2025, https://medium.com/foundation-models-deep-dive/what-is-next-for-multimodal-ai-fc4f84b60452
18. TEAL: Tokenize and Embed ALl for multi-modal large language models - arXiv, accessed July 16, 2025, https://arxiv.org/html/2311.04589v3
19. Meta was SO early again... AI Model That Unifies Vision & Language - YouTube, accessed July 16, 2025, https://www.youtube.com/watch?v=obSNYqgL53k
20. Gemini 1.5: Google's Generative AI Model with Mixture of Experts Architecture - Encord, accessed July 16, 2025, https://encord.com/blog/google-gemini-1-5-generative-ai-model-with-mixture-of-experts/
21. Gemini 1.5 Pro | Prompt Engineering Guide, accessed July 16, 2025, https://www.promptingguide.ai/models/gemini-pro
22. Mixture-of-Transformers (MoT): A New Era in Multi-Modal AI Efficiency - Medium, accessed July 16, 2025, https://medium.com/@sahin.samia/mixture-of-transformers-mot-a-new-era-in-multi-modal-ai-efficiency-e3ac2699eeb0
23. Audio Tokenization: An Overview - The GenAI Guidebook - Ravin Kumar, accessed July 16, 2025, https://ravinkumar.com/GenAiGuidebook/audio/audio_tokenization.html
24. An Intuitive Introduction to the Vision Transformer - Thalles' blog, accessed July 16, 2025, https://sthalles.github.io/an-intuitive-introduction-to-the-vision-transformer/
25. Understanding Patch Embeddings for Vision Transformers (ViT) | by Frederik vom Lehn, accessed July 16, 2025, https://medium.com/@frederik.vl/understanding-vision-transformers-vit-70ca8d817ff3
26. SoundStream Neural Codec: Understanding AI on Audio - Medium, accessed July 16, 2025, https://medium.com/@maercaestro/soundstream-neural-codec-understanding-ai-on-audio-8f1a123b097c
27. SoundStream: An End-to-End Neural Audio Codec - Google Research, accessed July 16, 2025, https://research.google/blog/soundstream-an-end-to-end-neural-audio-codec/
28. The Multimodal Evolution of Vector Embeddings - Twelve Labs, accessed July 16, 2025, https://www.twelvelabs.io/blog/multimodal-embeddings
29. Gemini 2.5 Technical Report : r/singularity - Reddit, accessed July 16, 2025, https://www.reddit.com/r/singularity/comments/1ldz6pj/gemini_25_technical_report/
30. What Is Multimodal AI? A 2025 Guide - Shopify, accessed July 16, 2025, https://www.shopify.com/blog/what-is-multimodal-ai
31. Gemini 2.5: Pushing the Frontier with Advanced ... - Googleapis.com, accessed July 16, 2025, https://storage.googleapis.com/deepmind-media/gemini/gemini_v2_5_report.pdf
32. Multimodal AI is proof that a picture is worth a thousand words ..., accessed July 16, 2025, https://cloud.google.com/transform/how-multimodal-ai-helps-business
33. Google's Gemini 1.5 Pro - Revolutionizing AI with a 1M Token Context Window - Medium, accessed July 16, 2025, https://medium.com/google-cloud/googles-gemini-1-5-pro-revolutionizing-ai-with-a-1m-token-context-window-bfea5adfd35f
34. How it's Made: Interacting with Gemini through multimodal prompting, accessed July 16, 2025, https://developers.googleblog.com/how-its-made-interacting-with-gemini-through-multimodal-prompting/
35. From Vlog to Blog: Transforming Videos into Engaging Content with Gemini 1.5 Pro | by Khongorzul Munkhbat | Medium, accessed July 16, 2025, https://medium.com/@centauream/from-vlog-to-blog-transforming-videos-into-engaging-content-with-gemini-1-5-pro-babac3d1ddeb
36. Gemini 2.5: Pushing the Frontier with Advanced Reasoning, Multimodality, Long Context, and Next Generation Agentic Capabilities. - arXiv, accessed July 16, 2025, https://arxiv.org/html/2507.06261v1
37. Gemini PRO 1.5 Video Understanding Test : r/singularity - Reddit, accessed July 16, 2025, https://www.reddit.com/r/singularity/comments/1b4a0hs/gemini_pro_15_video_understanding_test/
38. What Are the Most Promising Applications of Multimodal AI in Healthcare?, accessed July 16, 2025, https://www.octopusintelligence.com/what-are-the-most-promising-applications-of-multimodal-ai-in-healthcare/
39. Multimodal AI Applications in Healthcare & Beyond - Niveus Solutions, accessed July 16, 2025, https://niveussolutions.com/multimodal-ai-applications-in-healthcare/
40. What is the role of multimodal AI in self-driving cars? - Milvus, accessed July 16, 2025, https://milvus.io/ai-quick-reference/what-is-the-role-of-multimodal-ai-in-selfdriving-cars
41. Multimodal AI in Autonomous Vehicles: The Future of Mobility - Sapien, accessed July 16, 2025, https://www.sapien.io/blog/the-role-of-multimodal-ai-in-autonomous-vehicles
42. Multimodal: Agentic AI Platform for Finance and Insurance, accessed July 16, 2025, https://www.multimodal.dev/
43. A lexicon of artificial intelligence: understanding different AIs and their uses - Hello Future, accessed July 16, 2025, https://hellofuture.orange.com/en/a-lexicon-of-artificial-intelligence-understanding-different-ais-and-their-uses/
44. Aligning Data from Multiple Sources - ApX Machine Learning, accessed July 16, 2025, https://apxml.com/courses/intro-to-multimodal-ai/chapter-2-data-foundations-multimodal-systems/aligning-data-multiple-sources
45. What is the role of data alignment in multimodal AI? - Milvus, accessed July 16, 2025, https://milvus.io/ai-quick-reference/what-is-the-role-of-data-alignment-in-multimodal-ai
46. Large language model - Wikipedia, accessed July 16, 2025, https://en.wikipedia.org/wiki/Large_language_model
47. Large Multimodal Models (LMMs) vs LLMs in 2025 - Research AIMultiple, accessed July 16, 2025, https://research.aimultiple.com/large-multimodal-models/
48. [2407.12772] LMMs-Eval: Reality Check on the Evaluation of Large Multimodal Models, accessed July 16, 2025, https://arxiv.org/abs/2407.12772
49. [2506.20960] OmniEval: A Benchmark for Evaluating Omni-modal Models with Visual, Auditory, and Textual Inputs - arXiv, accessed July 16, 2025, https://arxiv.org/abs/2506.20960
50. RoboEgo System Card: An Omnimodal Model with Native Full Duplexity - arXiv, accessed July 16, 2025, https://arxiv.org/html/2506.01934v1
51. Simulating the Real World: A Unified Survey of Multimodal Generative Models - arXiv, accessed July 16, 2025, https://arxiv.org/abs/2503.04641
52. NeurIPS Poster DataComp: In search of the next generation of multimodal datasets, accessed July 16, 2025, https://neurips.cc/virtual/2023/poster/73525
53. [2506.09344] Ming-Omni: A Unified Multimodal Model for Perception and Generation - arXiv, accessed July 16, 2025, https://arxiv.org/abs/2506.09344
54. Ola: Pushing the Frontiers of Omni-Modal Language Model - arXiv, accessed July 16, 2025, https://arxiv.org/abs/2502.04328




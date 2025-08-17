# Mixture-of-Experts (MoE) 아키텍처

## 1.  MoE 패러다임의 기초

인공지능, 특히 대규모 언어 모델(LLM) 분야의 발전은 모델의 규모와 능력 사이의 깊은 상관관계를 드러냈습니다. 그러나 모델의 매개변수 수를 늘리는 전통적인 방식은 막대한 계산 비용과 에너지 소비라는 한계에 부딪혔습니다. 이러한 배경 속에서 Mixture-of-Experts (MoE) 아키텍처는 모델의 용량을 확장하면서도 계산 효율성을 유지하는 혁신적인 패러다임으로 부상했습니다. 이 보고서는 MoE 아키텍처의 근본적인 원리부터 역사적 발전, 핵심 구성 요소, 주요 모델, 기술적 과제, 그리고 미래 전망에 이르기까지 모든 측면을 심층적으로 고찰하는 것을 목표로 합니다.

### 1.1  조건부 계산의 서막

#### 1.1.1 밀집 모델의 규모 확장 비효율성

MoE 아키텍처의 등장을 이해하기 위해서는 먼저 전통적인 밀집(dense) 모델이 가진 내재적 한계를 인식해야 합니다. GPT-3와 같은 초기 대규모 모델들은 '밀집' 아키텍처를 기반으로 합니다.1 이는 모델이 입력된 모든 토큰(token)을 처리할 때마다 모델의 모든 매개변수(parameter)를 활성화하고 계산에 사용한다는 것을 의미합니다.2 이러한 접근 방식은 모델의 크기, 즉 매개변수의 수가 계산 비용(FLOPs, Floating Point Operations per Second)과 직접적으로 비례하는 구조를 만듭니다.4 따라서 모델의 성능을 높이기 위해 매개변수 수를 늘리면 훈련 시간, 추론 지연 시간, 그리고 에너지 소비가 기하급수적으로 증가하는 문제가 발생합니다.2 이 문제는 더 크고 유능한 모델을 개발하려는 AI 연구의 근본적인 병목 현상으로 작용했습니다.

이러한 밀집 모델의 확장 방식은 단순히 기술적인 문제를 넘어 경제적인 장벽을 형성했습니다. 거대한 모델을 훈련하고 서비스하는 데 필요한 막대한 하드웨어, 시간, 전력 비용은 소수의 거대 기업만이 감당할 수 있는 수준이었습니다.2 AI 기술의 발전이 지속 불가능한 비용 증가와 직결된다는 이 딜레마는 새로운 접근법의 필요성을 시사했습니다. MoE는 이러한 경제적, 기술적 한계를 극복하기 위한 대안으로 등장했습니다. 이는 '무차별적 확장(brute-force scaling)'에서 '지능적 확장(intelligent scaling)'으로의 전환을 의미하며, 모델이 가진 '지식의 총량'(총 매개변수)과 '사고의 비용'(활성 매개변수)을 분리함으로써 AI 스케일링의 새로운 길을 열었습니다.1

#### 1.1.2 핵심 원리: 조건부 계산과 희소 활성화

MoE는 '조건부 계산(conditional computation)'이라는 패러다임 전환을 통해 밀집 모델의 한계를 극복합니다.7 조건부 계산의 핵심 아이디어는 모든 입력에 대해 모델 전체를 사용하는 대신, 각 입력에 가장 관련성이 높은 일부만 선택적으로 활성화하는 것입니다.5 이를 '희소 활성화(sparse activation)'라고 부르며, MoE 효율성의 근간을 이룹니다.3

이 구조 덕분에 MoE 모델은 수십억, 심지어 수조 개의 "엄청난 수의 매개변수"를 가질 수 있으면서도, 각 토큰을 처리하는 데 드는 실제 계산 비용은 일정하게 유지할 수 있습니다.13 즉, 모델의 전체 용량(total parameters)과 계산 비용(active parameters)을 분리하는 데 성공한 것입니다. 이로 인해 MoE 모델은 거대한 지식 기반을 갖추면서도 훨씬 작은 밀집 모델과 유사한 속도로 추론을 수행할 수 있습니다.

#### 1.1.3 개념적 프레임워크: '전문가 팀' 비유

MoE의 작동 방식을 직관적으로 이해하기 위해 '전문가 컨설턴트 팀'이라는 비유가 널리 사용됩니다.5 복잡한 문제에 직면했을 때, 모든 분야를 어설프게 아는 한 명의 '일반 전문가(generalist)'에게 묻는 것보다, 각 분야의 최고 '전문가(specialist)'들에게 자문을 구하는 것이 더 효율적이고 정확합니다. MoE는 이 원리를 모방합니다.

이 비유에서 각 '전문가'는 특정 데이터 유형이나 문제 영역(예: 수학, 문학 번역, 코드 생성)에 특화된 작은 신경망(expert network)에 해당합니다.5 그리고 이 전문가 팀을 지휘하는 '관리자' 또는 '교통정리 담당자' 역할은 '게이팅 네트워크(gating network)' 또는 '라우터(router)'가 수행합니다.2 게이팅 네트워크는 입력된 데이터를 분석하여 어떤 전문가가 해당 작업을 가장 잘 처리할지 지능적으로 판단하고, 해당 전문가에게 작업을 전달합니다. 이러한 구조는 전문성, 효율성, 그리고 확장성이라는 MoE의 핵심적인 장점을 효과적으로 설명합니다.10

### 1.2  MoE의 기원과 진화

#### 1.2.1 년의 선구적 논문: "Adaptive Mixtures of Local Experts"

MoE의 개념은 1991년 Robert Jacobs, Geoffrey Hinton, Michael Jordan, Steven Nowlan이 발표한 기념비적인 논문 "Adaptive Mixtures of Local Experts"에서 처음 제안되었습니다.2 이들의 초기 동기는 신경망 내에서 여러 하위 작업(subtask)을 동시에 학습할 때 발생하는 간섭(interference) 현상을 줄이는 것이었습니다.20 이를 해결하기 위해 전체 문제를 여러 하위 문제로 나누고, 각 하위 문제를 전담하는 별도의 '전문가 네트워크'와 어떤 전문가를 사용할지 결정하는 '게이팅 네트워크'로 구성된 시스템을 제안했습니다.20

#### 1.2.2 협력적 앙상블에서 경쟁적 전문화로의 전환

초기 아이디어는 여러 전문가의 출력을 선형적으로 결합하는 방식이었습니다. 그러나 이 방식은 각 전문가가 다른 전문가들이 남긴 잔여 오차(residual error)를 보완하기 위해 '협력'하는 방향으로 학습되어, 결국 하나의 입력에 대해 다수의 전문가가 관여하는 비효율적인 결과를 낳았습니다.20

이 문제를 해결한 결정적인 혁신은 오차 함수를 재정의하여 전문가들 사이에 '경쟁'을 유도한 것이었습니다. 게이팅 네트워크가 확률적으로 단 하나의 전문가를 선택하도록('one-out-of-n' 선택) 시스템을 변경함으로써, 전문가들은 더 이상 협력할 필요 없이 각자 주어진 입력에 대해 완전한 출력을 생성해야 하는 상황에 놓였습니다.20 이 경쟁적 학습 환경은 각 전문가가 데이터의 특정 부분집합에 대해 고도로 특화된 '지역 전문가(local expert)'가 되도록 촉진했으며, 이는 현대 MoE 모델에서 볼 수 있는 전문가 전문화의 이론적 토대를 마련했습니다.

#### 1.2.3 희소 게이팅 MoE로의 도약 (Shazeer et al., 2017)

1991년에 개념이 정립되었음에도 불구하고, MoE는 수십 년간 주류 기술로 부상하지 못했습니다. 이는 알고리즘 및 성능상의 중대한 도전 과제들 때문이었습니다.8 2017년, Noam Shazeer 등이 발표한 논문 "Outrageously Large Neural Networks: The Sparsely-Gated Mixture-of-Experts Layer"는 이 정체기를 끝내는 전환점이 되었습니다. 이 연구는 현대 딥러닝 모델(당시에는 순환 신경망, RNN)에 쉽게 통합할 수 있는 범용 '희소 게이팅 MoE 계층(Sparsely-Gated MoE Layer)'을 제안했습니다.8

이 논문의 핵심 기여는 현대적인 GPU 클러스터 환경에서 계산 오버헤드를 최소화하면서 모델 용량을 1000배 이상 확장할 수 있음을 실증적으로 보여준 것입니다.8 이로써 MoE는 마침내 대규모 조건부 계산의 잠재력을 현실화하며 AI 연구의 중심으로 부상하게 되었습니다.

1991년의 개념 정립과 2017년의 실용적 구현 사이의 26년이라는 긴 시간은 MoE의 성공이 단순히 알고리즘의 진보만으로 이루어진 것이 아님을 시사합니다. 이는 하드웨어 및 시스템 엔지니어링의 발전과 알고리즘이 함께 진화한 결과물입니다. 1991년의 논문은 전문가, 게이팅 네트워크, 경쟁적 학습 목표라는 완벽한 개념적 프레임워크를 제시했지만 20, 당시의 하드웨어로는 계산 비용, 통신 병목 현상, 훈련 불안정성과 같은 문제들을 해결할 수 없었습니다.8 GPU와 TPU 같은 강력한 병렬 프로세서와 고대역폭 상호연결(interconnect) 기술의 등장은 전문가를 여러 장치에 분산시키고 토큰을 라우팅하는 복잡한 작업을 실현 가능하게 만들었습니다.8 2017년 Shazeer 등의 연구는 단순한 알고리즘 제안을 넘어, 이러한 현대 하드웨어의 특성을 고려한 '시스템 인식적(system-aware)' 구현이었습니다. 그들은 네트워크 대역폭 병목 현상을 명시적으로 다루고 MoE 계층을 현대 하드웨어에서 효율적으로 작동하도록 설계했습니다. 이는 AI 분야에서 알고리즘의 혁신이 종종 하드웨어의 발전을 기다려야 한다는 패턴을 다시 한번 확인시켜 주며, 알고리즘과 시스템의 공진화(co-evolution)가 기술 발전의 핵심 동력임을 보여주는 중요한 사례입니다.

## 2.  아키텍처 심층 분석 및 주요 모델

MoE 패러다임의 기본 원리를 이해한 후, 이제 그 구체적인 아키텍처를 해부하고, 이 기술을 정의한 주요 모델들을 분석하며, 전통적인 밀집 아키텍처와의 비교를 통해 그 장단점을 명확히 할 필요가 있습니다. 이 장에서는 MoE의 내부 작동 방식부터 시작하여, 구글의 스위치 트랜스포머와 미스트랄 AI의 믹스트랄과 같은 랜드마크 모델들을 심층적으로 살펴봅니다.

### 2.1  MoE 아키텍처의 해부

#### 2.1.1 전문가 네트워크의 역할

MoE 아키텍처의 '전문가(experts)'는 일반적으로 트랜스포머(Transformer) 모델의 표준 피드포워드 네트워크(Feed-Forward Network, FFN) 블록을 대체하는 작은 신경망들입니다.1 각 전문가는 독립적인 가중치(weights)를 가지지만, 보통 동일한 구조(예: 다층 퍼셉트론, MLP)를 공유합니다.10 이 전문가들의 목표는 훈련 과정에서 데이터의 특정 유형, 패턴, 또는 하위 작업에 대해 유기적으로 전문성을 발달시키는 것입니다.15 예를 들어, 어떤 전문가는 수학적 구문을 처리하는 데 능숙해지고, 다른 전문가는 특정 언어의 문법 구조를 파악하는 데 특화될 수 있습니다.5

#### 2.1.2 게이팅 네트워크 (라우터)

'게이팅 네트워크(gating network)' 또는 '라우터(router)'는 MoE 시스템의 핵심적인 조율자 역할을 하는 경량 신경망입니다.5 게이팅 네트워크는 각 입력 토큰의 표현(representation)을 입력으로 받아, 사용 가능한 모든 전문가에 대한 확률 분포를 출력합니다. 이 과정은 일반적으로 소프트맥스(softmax) 함수를 통해 이루어지며, 출력된 확률 값은 라우터가 특정 전문가가 현재 토큰을 처리하기에 얼마나 적합하다고 판단하는지를 나타냅니다.10 이 라우터의 결정에 따라 토큰은 가장 적합한 전문가에게 동적으로 전달됩니다.

#### 2.1.3 희소 활성화와 Top-K 게이팅

현대의 MoE 구현은 모든 전문가의 출력을 가중 합산하는 '밀집 MoE(dense MoE)' 방식 대신, '희소 활성화(sparse activation)' 방식을 채택합니다.10 이는 게이팅 네트워크가 가장 높은 점수를 받은 상위 

`k`개의 전문가('top-k')만을 선택하여 활성화하는 것을 의미합니다.2

- **Top-1 게이팅 (스위치 트랜스포머):** 가장 단순한 형태로, 각 토큰에 대해 가장 적합한 단 하나의 전문가(`k=1`)만을 선택합니다. 이 방식은 라우팅 계산을 크게 줄이고 전체 시스템을 단순화하는 장점이 있습니다.2
- **Top-2 게이팅 (믹스트랄):** 두 개의 전문가(`k=2`)를 선택하는 매우 인기 있는 방식입니다. 선택된 두 전문가의 출력은 게이팅 점수를 가중치로 사용하여 합산됩니다.2 이는 희소성을 유지하면서도 여러 전문가의 지식을 결합할 수 있는 균형 잡힌 접근법입니다.

최종적으로 MoE 계층의 출력은 선택된 top-k 전문가들의 출력에 각각의 게이트 값을 곱하여 합산한 결과가 됩니다.10 이 희소 활성화 메커니즘이 바로 MoE가 거대한 모델 용량을 유지하면서도 계산 효율성을 달성하는 비결입니다.

### 2.2  비교 분석: MoE 대 밀집 아키텍처

#### 2.2.1 총 매개변수 대 활성 매개변수 (FLOPs)

MoE와 밀집 모델의 가장 근본적인 차이는 매개변수 사용 방식에 있습니다. N개의 매개변수를 가진 밀집 모델은 모든 토큰에 대해 N개의 매개변수 전체를 사용합니다. 반면, MoE 모델은 훨씬 더 많은 총 매개변수(예: Mixtral 8x7B의 467억 개)를 가질 수 있지만, 각 토큰을 처리할 때는 그중 일부인 '활성 매개변수'(예: Mixtral의 129억 개)만을 사용합니다.17 이는 MoE 모델이 매우 큰 모델의 '지식'을 가지면서도 훨씬 작은 모델의 '추론 속도'와 '계산 비용(FLOPs)'을 가질 수 있음을 의미합니다.1

이러한 구조적 차이는 대규모 언어 모델을 평가하고 비교하는 방식을 근본적으로 바꾸었습니다. 과거에는 모델의 능력이 단순히 밀집 매개변수의 수(예: GPT-2의 15억 개 대 GPT-3의 175억 개)와 강하게 연관되어 있었습니다. 그러나 Mixtral 8x7B (총 467억, 활성 129억)와 같은 MoE 모델이 훨씬 더 큰 밀집 모델인 Llama 2 70B를 다수의 벤치마크에서 능가하는 성능을 보여주면서 17, 이러한 단순 비교는 더 이상 유효하지 않게 되었습니다. 이제 모델을 평가할 때는 "Mixtral은 47B 모델인가, 13B 모델인가?"와 같은 더 미묘한 질문을 던져야 합니다. 정답은 둘 다이면서 둘 다 아니라는 것입니다. Mixtral은 약 47B 모델의 '지식 용량'을 가지지만, 약 13B 모델의 '추론 비용'을 가집니다.32 이로 인해 업계는 '희소성', '활성 매개변수', '라우팅 전략(예: 8x7B, top-2 라우팅)'과 같은 새로운 기술 용어를 사용하여 모델을 보다 정교하게 설명하고 평가하게 되었습니다. 이러한 변화는 연구자들의 공정한 비교뿐만 아니라, 기업이 비용 대비 성능을 고려하여 어떤 모델을 배포할지 결정하는 데에도 중요한 영향을 미치고 있습니다.35

#### 2.2.2 스케일링 법칙과 성능 트레이드오프

MoE 모델은 밀집 모델과 다른 스케일링 궤적을 보입니다. 동일한 계산 예산 하에서 MoE 모델은 훨씬 빠른 사전 훈련 속도 향상(스위치 트랜스포머의 경우 T5 대비 최대 7배)을 달성할 수 있습니다.13 하지만 여기에는 중요한 트레이드오프가 존재합니다. 바로 메모리 오버헤드입니다. 추론 시에는 매개변수의 일부만 활성화되지만, 모든 전문가의 전체 매개변수 집합이 VRAM에 로드되어야 합니다.3 따라서 MoE는 FLOPs 효율적이지만 메모리 집약적인 아키텍처라고 할 수 있습니다.

#### 2.2.3 하이브리드 아키텍처: 두 세계의 장점 결합

순수한 MoE 시스템이 가진 통신 오버헤드 문제를 완화하기 위해 하이브리드 아키텍처가 등장했습니다. 스노우플레이크(Snowflake)의 Arctic 모델이 대표적인 예로, 이 모델은 밀집 트랜스포머 모델과 잔차 연결(residual connection) 방식의 MoE 컴포넌트를 결합합니다.4 밀집 부분이 일부 계산을 로컬에서 처리하여 전체적인 통신 부담을 줄이는 동안, MoE 부분은 거대한 확장 가능 용량을 제공합니다. 이는 순수 밀집 모델과 순수 희소 모델 사이의 실용적인 엔지니어링 절충안을 제시합니다.4

### 2.3  사례 연구: 구글의 스위치 트랜스포머

#### 2.3.1 '스위치' 계층 혁신 (k=1 라우팅)

2021년에 발표된 "Switch Transformers: Scaling to Trillion Parameter Models with Simple and Efficient Sparsity" 논문은 MoE 분야에 큰 획을 그었습니다.14 이 논문의 핵심 혁신은 복잡한 top-k 라우팅 알고리즘을 단순한 $k=1$ '스위치(Switch)' 계층으로 대체한 것입니다.25 이 사소해 보이는 변경은 상당한 이점을 가져왔습니다. 라우팅 계산량을 줄이고, 전문가당 필요한 배치 크기를 절반으로 감소시켰으며, 전반적인 성능과 안정성을 향상시켰습니다.30

#### 2.3.2 조 단위 매개변수 규모 달성

스위치 트랜스포머는 1조 개가 넘는 매개변수를 가진 모델을 성공적으로 훈련시킨 최초의 사례입니다.37 이는 데이터 병렬 처리, 모델 병렬 처리, 그리고 스위치 계층이 가능하게 한 새로운 전문가 병렬 처리를 효율적으로 결합함으로써 달성되었습니다.25 이 논문은 당시 최첨단 밀집 모델이었던 T5-XXL 모델에 비해 4배의 속도 향상을 보였다고 보고했습니다.14

#### 2.3.3 성능 벤치마크 및 훈련 안정성 향상 기법

스위치 트랜스포머는 동일한 계산 자원을 사용하면서도 T5-Base 및 T5-Large 모델 대비 최대 7배의 사전 훈련 속도 향상을 입증했습니다.13 특히, 저자들은 희소 모델 훈련의 고질적인 불안정성을 해결하기 위한 핵심적인 기법들을 도입했습니다 25:

- **선택적 정밀도(Selective Precision):** 대부분의 계산은 효율적인 `bfloat16` 정밀도로 수행하되, 라우터의 소프트맥스 계산과 같이 수치적으로 민감한 부분만 `float32`로 변환하여 계산합니다. 이를 통해 높은 통신 비용 없이 수치 안정성을 확보했습니다.25
- **더 작은 매개변수 초기화:** 가중치 초기화 시 더 작은 스케일 값을 사용하여, 특히 전문가 수가 많아질 때 훈련 안정성을 개선했습니다.41
- **강화된 전문가 드롭아웃:** MoE 모델은 방대한 매개변수 수로 인해 미세 조정(fine-tuning) 시 과적합(overfitting)에 취약합니다. 이를 해결하기 위해 전문가 계층 내에서 드롭아웃 비율을 높이는 '전문가 드롭아웃'을 적용하여 효과적으로 과적합을 억제했습니다.41

### 2.4  사례 연구: 미스트랄 AI 믹스트랄과 오픈소스 혁명

#### 2.4.1 Mixtral 8x7B 및 8x22B: 아키텍처와 성능

미스트랄 AI(Mistral AI)의 믹스트랄(Mixtral) 시리즈는 고성능 MoE 모델을 오픈소스 커뮤니티에 제공하며 큰 반향을 일으켰습니다. 믹스트랄의 아키텍처는 디코더-온리(decoder-only) 트랜스포머를 기반으로 하며, 각 FFN 계층이 8개의 전문가로 구성된 MoE 계층으로 대체됩니다.17 라우팅 전략으로는 top-2를 사용하여 각 토큰이 두 개의 전문가에 의해 처리됩니다.31

- **Mixtral 8x7B:** 총 467억 개의 매개변수를 가지지만, 토큰당 약 129억 개의 활성 매개변수만 사용합니다.17 이 모델은 훨씬 더 큰 밀집 모델인 Llama 2 70B를 대부분의 벤치마크, 특히 수학, 코드 생성, 다국어 작업에서 능가하는 성능을 보였습니다.26
- **Mixtral 8x22B:** 총 1410억 개의 매개변수와 390억 개의 활성 매개변수를 가진 더 큰 버전으로, 강력한 다국어 능력과 네이티브 함수 호출 기능을 갖추어 오픈소스 모델의 성능을 한 단계 끌어올렸습니다.26

#### 2.4.2 오픈 웨이트 고성능 MoE 모델의 전략적 영향

믹스트랄이 허용적인 아파치 2.0(Apache 2.0) 라이선스로 출시된 것은 AI 생태계에 중요한 사건이었습니다.31 이로 인해 GPT-3.5와 같은 강력한 폐쇄형 모델에 필적하거나 이를 능가하는 성능을 가진 모델에 대한 접근이 민주화되었습니다.34 이는 오픈소스 MoE 기술을 기반으로 한 수많은 혁신, 연구, 상업적 애플리케이션의 물결을 일으켰으며 5, 데이터브릭스(Databricks)와 같은 경쟁사들이 DBRX와 같은 자체 오픈 MoE 모델을 출시하도록 자극했습니다.36

#### 2.4.3 표 1: 주요 MoE 모델 비교

| 모델명                         | 총 매개변수 | 활성 매개변수   | 전문가 수 | Top-k | 아키텍처 유형 | MMLU | HumanEval | GSM8k |
| ------------------------------ | ----------- | --------------- | --------- | ----- | ------------- | ---- | --------- | ----- |
| **Switch-C (Google)**          | 1.6T        | N/A (단일 경로) | 2048      | 1     | 희소          | N/A  | N/A       | N/A   |
| **Mixtral 8x7B (Mistral AI)**  | 46.7B       | 12.9B           | 8         | 2     | 희소          | 70.6 | 40.2      | 61.1  |
| **Mixtral 8x22B (Mistral AI)** | 141B        | 39B             | 8         | 2     | 희소          | 77.7 | 55.5      | 90.8  |
| **DBRX (Databricks)**          | 132B        | 36B             | 16        | 4     | 희소          | 73.7 | 70.1      | 72.8  |
| **Arctic (Snowflake)**         | 480B        | 17B             | 128       | 2     | 하이브리드    | N/A  | 54.9      | 65.8  |

주: 벤치마크 점수는 모델의 Instruct 또는 Chat 버전을 기준으로 하며, 논문 및 공개 자료에서 보고된 값입니다. 4

이 표는 MoE 모델의 핵심적인 특성을 한눈에 비교할 수 있게 해줍니다. '총 매개변수'와 '활성 매개변수'의 극명한 차이는 MoE의 가치를 시각적으로 보여줍니다. 또한, Mixtral의 '8개 전문가/top-2'와 DBRX의 '16개 전문가/top-4' 같은 라우팅 전략의 차이는 모델 품질과 복잡성 사이의 다양한 설계 철학을 드러냅니다. 벤치마크 점수는 이러한 아키텍처 선택이 실제 성능에 어떻게 반영되는지를 구체적으로 보여주는 정량적 지표 역할을 합니다.

## 3.  MoE의 공학적 및 과학적 과제

MoE 아키텍처의 개념과 주요 모델을 이해했다면, 다음 단계는 이 모델들을 실제로 작동시키는 데 따르는 복잡한 현실을 파고드는 것입니다. 이 장에서는 MoE 모델이 직면하는 핵심적인 기술적 과제들과, 연구 커뮤니티가 이를 해결하기 위해 개발한 독창적인 해결책들을 집중적으로 다룹니다.

### 3.1  핵심 과제, 부하 분산

#### 3.1.1 전문가의 과소/과다 활용 문제

이상적인 MoE 시스템에서는 모든 전문가에게 토큰이 균등하게 분배됩니다. 하지만 실제 훈련 과정에서 게이팅 네트워크는 종종 소수의 '인기 있는' 전문가를 선호하는 경향을 보입니다.3 이는 선호되는 전문가가 더 빠르게 훈련되고, 그 결과 더 유능해져서 다시 더 많이 선택되는 자기 강화 순환(self-reinforcing cycle)을 만듭니다. 이로 인해 일부 전문가는 과도하게 사용되는 반면, 다른 전문가들은 거의 사용되지 않고 유휴 상태로 남아있게 됩니다('모드 붕괴', mode collapse). 이는 모델의 용량을 낭비하고 전반적인 성능을 저하시키는 심각한 문제입니다.3

#### 3.1.2 해결책 심층 분석: 보조 손실 함수

이 문제를 해결하기 위한 가장 일반적인 방법은 주 훈련 목표에 '보조 손실 함수(auxiliary loss function)'를 추가하여 불균형한 라우팅에 페널티를 부과하는 것입니다.2

- **`L_aux` / 부하 분산 손실(Load Balancing Loss):** 이 손실 함수는 게이팅 네트워크가 전문가들에게 토큰을 균등하게 할당하도록 유도합니다.2 이 값은 각 전문가에게 라우팅된 토큰의 비율과 해당 전문가에 대한 평균 라우팅 확률을 기반으로 계산됩니다.46 이 기법은 Mixtral, GShard와 같은 현대적인 MoE 모델의 표준 구성 요소가 되었습니다.46
- **`z-loss`:** 대규모 모델에서 안정성을 더욱 향상시키기 위해 일부 모델은 `z-loss`를 추가합니다. 이 손실 함수는 게이팅 네트워크의 소프트맥스 함수에 입력되는 로짓(logit) 값이 너무 커지는 것을 방지하여, 잠재적인 반올림 오차를 줄이고 훈련 안정성을 높이는 역할을 합니다.46
- **직교성 및 분산 손실(Orthogonality and Variance Losses):** 최근 연구에서는 표준 부하 분산 손실이 때때로 전문가들의 기능 중복을 유발할 수 있다는 점을 지적하며, 이를 보완하기 위한 새로운 손실 함수들을 제안합니다. '직교성 손실'은 전문가들이 서로 다른 기능을 학습하도록 명시적으로 유도하고, '분산 손실'은 라우터가 더 명확하고 차별적인 결정을 내리도록 장려합니다.47

#### 3.1.3 해결책 심층 분석: 전문가 용량 및 토큰 탈락

또 다른 부하 분산 메커니즘은 각 전문가가 처리할 수 있는 토큰의 최대 개수, 즉 '용량(capacity)'을 미리 설정하는 것입니다.7 만약 특정 전문가에게 용량을 초과하는 토큰이 라우팅되면, 초과된 '오버플로(overflow)' 토큰들은 해당 전문가에 의해 처리되지 않고 탈락(drop)됩니다. 이 토큰들의 정보는 잔차 연결(residual connection)을 통해 다음 계층으로 전달되므로, 정보가 완전히 소실되지는 않지만 해당 MoE 계층의 전문가 연산은 건너뛰게 됩니다.41 이는 전문가의 과부하를 막는 강력한 제약 조건으로 작용하지만, 토큰 탈락으로 인한 정보 손실의 위험이 있습니다. 일반적으로 용량 계수(capacity factor)를 1.0보다 약간 큰 값(예: 1.25)으로 설정하여, 라우팅의 약간의 불균형을 흡수하고 토큰 탈락을 최소화하는 버퍼를 둡니다.41

### 3.2  분산 훈련의 병목, 통신

#### 3.2.1 All-to-All 통신 패턴의 이해

대규모 MoE 모델을 훈련할 때는 병렬 처리를 위해 전문가들을 여러 장치(예: GPU)에 분산시킵니다.2 이때 GPU-1에 있는 토큰이 GPU-2에 위치한 전문가에게 라우팅되면, 해당 토큰의 표현(representation)은 네트워크를 통해 전송되어야 합니다. 모든 GPU가 다른 모든 GPU에게 토큰을 보내야 할 수 있기 때문에, 이 과정에서 막대한 규모의 

**All-to-All** 통신 패턴이 발생합니다.48 이 통신은 각 MoE 계층에서 순방향 전파(토큰 전송)와 역방향 전파(그래디언트 반환) 시 두 번 발생하며, 전체 훈련 시간의 45% 이상을 차지하는 심각한 성능 병목 현상을 유발합니다.51

이 All-to-All 통신 병목 현상은 현대 MoE의 문제가 더 이상 단순한 신경망 알고리즘의 문제가 아니라, 고전적인 고성능 컴퓨팅(High-Performance Computing, HPC) 문제의 영역으로 넘어갔음을 명확히 보여줍니다. All-to-All 집합 통신(collective communication)은 3D FFT와 같은 전통적인 HPC 워크로드에서도 잘 알려진 어려운 과제입니다.48 MoE 모델이 확장되면서, 여러 장치에 분산된 전문가들 사이의 토큰 셔플링은 정확히 이러한 HPC 스타일의 병목 현상을 재현했습니다.48 결과적으로, 이를 해결하기 위해 개발되고 있는 솔루션들 역시 ML 커뮤니티뿐만 아니라 HPC 분야에서 직접 가져온 것들입니다. 토폴로지 인식 라우팅, 집합 통신 알고리즘 최적화, 통신과 계산의 중첩 등이 그 예입니다.46 DeepSpeed-MoE와 같은 시스템은 본질적으로 MoE 워크로드에 맞춤화된 전문 HPC 프레임워크라고 할 수 있습니다.54 이는 딥러닝과 HPC 분야의 융합을 의미하며, 차세대 AI 모델을 구축하기 위해서는 ML 과학자뿐만 아니라 네트워크 토폴로지, 통신 프로토콜, 분산 시스템을 근본적으로 이해하는 시스템 엔지니어와 HPC 전문가가 필수적이라는 점을 시사합니다.

#### 3.2.2 시스템 수준 최적화: DeepSpeed-MoE와 MegaScale-MoE

- **DeepSpeed-MoE:** 마이크로소프트에서 개발한 시스템으로, MoE 훈련 및 추론을 위한 포괄적인 최적화 도구를 제공합니다.5 ZeRO 옵티마이저와 같은 DeepSpeed 라이브러리의 기술을 활용하여 메모리를 효율적으로 관리하고 54, 병렬 처리 조정 통신(parallelism-coordinated communication)과 같은 기법을 통해 통신 지연 시간과 비용을 크게 줄여 기존 솔루션 대비 최대 7.3배의 성능 향상을 주장합니다.55
- **MegaScale-MoE:** 통신 병목 현상 완화에 초점을 맞춘 또 다른 시스템입니다. 이 시스템은 '선택적 활성화 재물질화(selective activation rematerialization)'와 같은 기술을 사용합니다. 이는 순방향 전파 시 모든 활성화 값을 저장하는 대신 일부를 역방향 전파 중에 재계산하는 방식으로, 메모리를 절약하고 통신-계산 중첩을 최적화합니다.51

#### 3.2.3 알고리즘 및 시스템 공동 설계

다양한 기법들이 알고리즘 변경과 시스템 인식을 결합하여 통신 문제를 해결합니다.

- **파이프라이닝 및 중첩(Pipelining and Overlapping):** All-to-All 통신과 전문가 계산을 중첩시켜 지연 시간을 숨기는 것은 일반적인 전략입니다.46
- **계층적 및 토폴로지 인식 통신(Hierarchical and Topology-Aware Communication):** 네트워크 토폴로지를 인식하여, 느린 노드 간(inter-node) 통신보다 빠른 노드 내(intra-node) 통신을 우선적으로 활용하도록 통신을 최적화합니다.46
- **통신 압축(Communication Compression):** 그래디언트 동기화를 위해 `BF16`과 같은 저정밀도 형식을 사용하거나, 지역성 민감 해싱(Locality-Sensitive Hashing, LSH)을 사용하여 유사한 토큰들을 클러스터링하고 클러스터의 중심값(centroid)만 전송하는 방식으로 전송 데이터의 총량을 줄입니다.51

### 3.3  진화하는 라우팅 알고리즘

#### 3.3.1 토큰 선택 라우팅의 한계

각 토큰이 선호하는 전문가를 선택하는 표준적인 top-k 게이팅 방식, 즉 '토큰 선택(token-choice)' 라우팅은 부하 분산 문제의 근본적인 원인입니다. 이는 토큰이 전문가에게 '밀어넣어지는(push)' 모델로, 특정 전문가에게 토큰이 몰리는 혼잡 현상을 유발할 수 있습니다.59

#### 3.3.2 전문가 선택 라우팅

GLaM과 EC-DiT 같은 모델에서 사용된 '전문가 선택(expert-choice)' 라우팅은 이 과정을 뒤집습니다.59 토큰이 전문가를 선택하는 대신, 각 전문가가 전체 배치에서 처리하고 싶은 상위 

`k`개의 토큰을 '끌어오는(pull)' 방식입니다. 이 방식의 가장 큰 장점은 각 전문가가 고정된 수의 토큰을 처리하도록 보장함으로써 **부하 분산 문제를 본질적으로 해결한다**는 것입니다. 이로 인해 보조 부하 분산 손실 함수가 필요 없어져 훈련이 더 안정될 수 있습니다.46

#### 3.3.3 새로운 패러다임: 전문가 자율성과 공유 라우터

- **전문가 자율성(Autonomy-of-Experts, AoE):** 최근 제안된 급진적인 방식으로, 라우터를 완전히 제거합니다.60 AoE에서는 모든 전문가가 모든 토큰에 대해 예비 계산을 수행합니다. 그 후, 내부 활성화 값의 크기(이는 전문가의 '자신감' 또는 '능숙도'를 나타내는 대리 지표로 사용됨)를 기준으로 전문가들 스스로 전체 계산을 진행할지 여부를 결정합니다. 이 자기 선택 메커니즘은 라우터의 '예측'과 전문가의 실제 '능력' 사이의 불일치를 제거하여 더 나은 전문가 선택과 효과적인 학습을 유도할 수 있다고 주장됩니다.60
- **공유 라우터(Shared Routers, Omni-router):** 각 MoE 계층이 독립적인 라우터를 갖는 대신, 여러 계층 또는 모든 계층에 걸쳐 단일 라우터를 공유하는 접근법입니다.62 이는 다른 계층의 전문가들이 통일된 라우팅 신호에 기반하여 협력하도록 학습함으로써, 모델 전체에 걸쳐 더 조율된 의사 결정과 깊이 있는 전문화를 촉진할 것이라는 가설에 기반합니다.62

제 10장: 수치 안정성 및 그래디언트 흐름

#### 3.3.4 희소 그래디언트 업데이트의 과제

주어진 토큰에 대해 일부 전문가만 활성화되기 때문에, 역방향 전파 시 그래디언트는 선택된 전문가와 해당 토큰에 대한 라우터의 결정에만 전달됩니다. 이러한 '희소 업데이트'는 훈련 불안정성을 유발할 수 있으며, 라우터가 자신이 선택하지 *않은* 전문가에 대한 피드백을 받지 못하기 때문에 효과적인 학습을 어렵게 만듭니다.44

#### 3.3.5 안정적인 훈련을 위한 기법

스위치 트랜스포머 사례 연구에서 논의된 바와 같이, 안정적인 훈련을 위해 몇 가지 기법이 필수적입니다 25:

- **선택적 정밀도(혼합 정밀도):** 효율성을 위해 `bfloat16`을 사용하면서도 라우터의 소프트맥스와 같은 민감한 계산은 `float32`로 유지하는 것은 수치적 언더플로우/오버플로우를 방지하는 핵심 기법입니다.39
- **신중한 초기화:** 더 작은 가중치 초기화 값을 사용하면 훈련 초기에 활성화 값과 그래디언트가 폭발하는 것을 방지하는 데 도움이 됩니다.41
- **라우터 z-loss:** 7장에서 언급했듯이, 이 보조 손실은 라우터 로짓의 크기를 제한하여 라우터를 안정화시키는 데 기여합니다.46

#### 3.3.6 미세 조정 시 과적합 해결

방대한 매개변수를 가진 MoE 모델은 규모가 작은 다운스트림 데이터셋에 대해 미세 조정할 때 과적합에 매우 취약합니다.41 이에 대한 주된 해결책은 강력한 정규화(regularization)입니다. 스위치 트랜스포머 논문에서는 *전문가 계층 내에서만* 드롭아웃 비율을 높이는 '전문가 드롭아웃'이 간단하면서도 효과적인 해결책임을 발견했습니다.41

#### 표 2: MoE의 기술적 과제 및 해결책 요약

| 과제              | 핵심 문제 설명                                               | 알고리즘적 해결책                                            | 시스템 수준 해결책                                           |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **부하 불균형**   | 게이팅 네트워크가 소수의 '인기' 전문가에게 토큰을 편중시켜 용량 낭비 및 전문화 저해 3 | 보조 부하 분산 손실 (`L_aux`, `z-loss`) 46, 전문가 선택 라우팅 61, 직교성 손실 47 | 전문가 용량 제한 25, 배치 우선 순위 라우팅 (BPR) 46          |
| **통신 오버헤드** | 분산 훈련 시 전문가 간 토큰 교환으로 인한 All-to-All 통신 병목 현상 51 | 통신 압축 (LSH, 저정밀도) 51, 하이브리드 아키텍처 (PR-MoE) 55 | DeepSpeed-MoE, MegaScale-MoE 51, 통신-계산 중첩, 토폴로지 인식 라우팅 46 |
| **훈련 불안정성** | 희소 업데이트와 라우터의 이산적 결정으로 인한 수치적 불안정성 및 그래디언트 소실/폭발 39 | 선택적 정밀도 (혼합 정밀도) 39, 작은 가중치 초기화 41, 라우터 z-loss 46 | 최적화된 분산 훈련 프레임워크 사용 51                        |
| **과적합**        | 방대한 매개변수로 인해 작은 데이터셋에 미세 조정 시 성능 저하 41 | 전문가 드롭아웃 41, 정규화 기법 강화                         | -                                                            |
| **해석 가능성**   | 전문가의 기능과 라우팅 결정의 근거를 이해하기 어려움 47      | 직교성 손실 등 전문화 강화 기법 47, 라우팅 패턴 및 공-활성화 행렬 분석 66 | 해석 가능성을 위한 시각화 도구 및 프레임워크 개발 45         |

이 표는 3부에서 논의된 복잡하고 상호 연관된 문제와 해결책을 구조적으로 요약합니다. '알고리즘적' 해결책과 '시스템 수준' 해결책을 구분함으로써, MoE의 과제를 해결하기 위해서는 소프트웨어/모델 설계와 하드웨어/시스템 엔지니어링을 아우르는 다각적인 접근이 필요하다는 핵심을 강조합니다.

## 4.  응용, 해석 가능성 및 미래 전망

MoE 아키텍처의 기술적 깊이를 탐색한 후, 이제 시야를 넓혀 이 기술이 실제로 어디에 사용되고 있는지, 그 내부 작동을 어떻게 이해할 수 있는지, 그리고 앞으로 어떤 방향으로 나아갈지를 조망할 시간입니다. 이 마지막 장에서는 언어 모델을 넘어선 MoE의 다양한 응용 분야를 살펴보고, 전문가 전문화의 미스터리를 파헤치며, MoE 기술의 미래 궤적을 전망합니다.

### 4.1  언어를 넘어선 MoE: 다양한 도메인에서의 응용

#### 4.1.1 컴퓨터 비전: Vision Transformer with MoE (V-MoE)

MoE의 유연성은 언어 모델에만 국한되지 않습니다. 컴퓨터 비전 분야에서도 MoE는 중요한 역할을 하기 시작했으며, 그 대표적인 예가 V-MoE(Vision Transformer with MoE)입니다.69

- **아키텍처:** V-MoE는 표준 비전 트랜스포머(ViT)의 일부 FFN 계층을 MoE 계층으로 대체하는 구조를 가집니다.70 라우팅은 이미지 패치(ViT의 토큰에 해당) 단위로 이루어집니다.28
- **성능:** V-MoE는 이미지넷(ImageNet)에서 당시 최첨단(SOTA) 성능을 입증했으며, 150억 개의 매개변수를 가진 역대 가장 큰 비전 모델을 훈련시키는 데 성공했습니다. 특히, 밀집 모델과 동등한 성능을 절반 수준의 추론 연산량으로 달성하여 효율성을 입증했습니다.69
- **적응형 추론(Adaptive Inference):** V-MoE는 '배치 우선 순위 라우팅(Batch Prioritized Routing, BPR)'과 같은 기법을 통해 추론 시 덜 중요한 이미지 패치를 동적으로 폐기할 수 있습니다. 이는 정확도와 계산 비용 사이의 유연한 트레이드오프를 가능하게 합니다.69
- **추가 발전:** V-MoE의 성공 이후, 적응형 다중 작업 학습을 위한 AdaMV-MoE 72나 자원이 제한된 장치를 위한 Mobile V-MoE 28와 같은 다양한 후속 연구들이 등장하며 비전 분야에서의 MoE 활용 가능성을 확장하고 있습니다.

#### 4.1.2 기타 도메인에서의 응용

MoE의 모듈성과 유연성은 다른 여러 분야에서도 그 가치를 인정받고 있습니다.5 대표적인 예는 다음과 같습니다.

- **강화 학습(Reinforcement Learning):** 로봇 공학이나 게임과 같은 분야에서 동적인 의사 결정을 위해 MoE가 사용됩니다. 예를 들어, Switch Trajectory Transformer는 다중 작업 강화 학습 문제에서 모델 용량을 효율적으로 확장하기 위해 MoE 아키텍처를 활용합니다.43
- **금융(Finance):** 변화하는 시장 상황(예: 강세장/약세장)에 적응하기 위해 서로 다른 전문가를 활성화하는 동적 자산 배분 전략에 MoE가 적용될 수 있습니다.61
- **의료(Healthcare):** 진단, 처방, 환자 분류 등 다양한 의료 하위 작업을 처리하기 위해 각 분야에 특화된 전문가를 두는 모델 설계가 연구되고 있습니다.74
- **자동 음성 인식(Automatic Speech Recognition, ASR):** 다양한 억양, 소음 환경, 도메인별 어휘에 대응하기 위해 MoE를 사용하여 음성 인식 모델의 강건성과 정확도를 높이는 연구가 진행 중입니다.62

### 4.2  전문가의 '마음' 들여다보기: 해석 가능성과 전문화

#### 4.2.1 전문가는 어떻게 전문화되는가?

MoE의 핵심적인 매력 중 하나는 해석 가능성에 대한 기대였습니다. 각 전문가가 '수학 전문가', '역사 전문가'처럼 인간이 이해하기 쉬운 명확한 기능을 가질 것이라는 희망이 있었습니다.16 그러나 실제 연구 결과는 이보다 훨씬 복잡한 현실을 보여줍니다. 전문가들은 명확하게 구분되는 기능 대신, 여러 개념을 동시에 인코딩하는 '다의성(polysemanticity)'을 보이며, 전문화는 여러 전문가에 걸쳐 분산되는 경향이 있습니다.27 전문화는 언어적 패턴, 주제, 심지어 시퀀스 내 토큰의 위치와 같은 미묘한 특성을 기반으로 나타납니다.67

이러한 '해석 가능성의 격차'는 MoE 연구의 중요한 방향을 제시했습니다. 초기에는 단순히 전문가 팀 비유에 기반한 기대를 가졌지만, 경험적 분석 결과 전문가의 기능이 복잡하고 분산된 형태로 나타난다는 것이 밝혀졌습니다. 이로 인해 연구자들은 라우팅 패턴 분석, 공-활성화 행렬, 어텐션과 전문가 선택 간의 상호작용 분석 등 새로운 동적 분석 도구를 개발하게 되었습니다.66 이를 통해 전문화가 미리 정의된 것이 아니라 복잡한 시스템의 창발적 속성(emergent property)이라는 깊은 이해에 도달했습니다. 더 나아가, 이는 단순히 전문화를 관찰하는 것을 넘어, 직교성 손실과 같은 목적 함수를 통해 전문화를 능동적으로 '강제'할 수 있는지에 대한 새로운 연구 방향을 열었습니다.47 결국, MoE의 동적인 특성이 제기하는 도전 과제들은 LLM 해석 가능성 연구 분야 전체를 정적인 가중치 분석에서 동적인 정보 흐름 분석으로 나아가게 하는 촉매제가 되고 있습니다.

#### 4.2.2 라우팅 패턴 및 공-활성화 행렬 시각화

전문가들의 행동을 이해하기 위해 연구자들은 라우팅 결정을 분석합니다.45

- **라우팅 시각화:** 다양한 유형의 입력에 대해 어떤 전문가가 선택되는지를 시각화하여 패턴을 발견합니다. 예를 들어, V-MoE 연구에서는 전문가 선택과 이미지 클래스 간에 어느 정도 상관관계가 있음을 시각화를 통해 보여주었습니다.69
- **공-활성화 분석(Co-activation Analysis):** 어떤 전문가 쌍이 함께 자주 활성화되는지를 분석하여 기능적 클러스터나 협력 그룹을 식별할 수 있습니다.67 최근 연구에 따르면 이러한 공-활성화 패턴은 희소하며 훈련 초기에 안정화되는 경향이 있습니다.67
- **의미 기반 협력(Semantic-Driven Collaboration):** 심층 분석에 따르면 어텐션 헤드와 전문가 사이에 긴밀한 협력 관계가 존재합니다. 어텐션 헤드가 의미적 특징을 식별하면, 라우터는 해당 특징을 처리하는 데 가장 적합한 전문가를 선택하여 계층 간 협력적인 정제 과정을 만들어냅니다.66

#### 4.2.3 전문화 강화 및 유도 기법

전문성이 약하거나 중복되는 문제를 해결하기 위해, 이를 명시적으로 유도하는 기법들이 연구되고 있습니다. 대표적으로 훈련 목표에 '직교성 손실(orthogonality loss)'을 추가하여 전문가들의 표현(representation)이 서로 멀어지도록 강제함으로써, 각기 다른 기능을 학습하도록 유도하는 방법이 있습니다.47

### 4.3  전략적 지형과 미래 궤적

#### 4.3.1 오픈소스 MoE의 비즈니스: 미스트랄 대 데이터브릭스

고성능 오픈소스 MoE 모델의 등장은 AI 시장의 경쟁 구도를 바꾸고 있습니다.

- **미스트랄 AI(Mistral AI):** 유럽 기반의 스타트업으로, Mixtral과 같은 고성능 오픈소스 모델을 전략적으로 활용하여 커뮤니티를 구축하고, 이를 기반으로 상용 API 서비스의 채택을 유도하는 전략을 구사하고 있습니다.35
- **데이터브릭스(Databricks):** 기존의 데이터 처리 플랫폼과 엔터프라이즈 고객 기반을 바탕으로, 기업 환경에 중요한 코드 생성과 같은 작업에 특화된 MoE 모델 DBRX를 개발했습니다. 이들의 전략은 강력한 오픈 LLM을 자사의 데이터 인텔리전스 플랫폼에 직접 통합하여 강력한 생태계를 구축하는 것입니다.36

이러한 경쟁은 기술 혁신을 가속화하고 있으며, 사용자들에게 폐쇄형 모델에 대한 강력하고 자유로운 대안을 제공하고 있습니다.5

#### 4.3.2 미래 연구 방향

이 보고서에서 분석한 여러 연구 자료들은 MoE의 미래에 대한 몇 가지 공통된 방향을 제시합니다.

- **동적 및 적응형 아키텍처:** 훈련 중에 데이터 패턴에 따라 전문가를 동적으로 생성, 병합 또는 제거하는 등 모델 구조 자체를 변화시키는 연구가 활발해질 것입니다.45
- **하드웨어 가속 및 공동 설계:** 희소 MoE 연산에 특화된 맞춤형 하드웨어를 설계하거나, 메모리 및 통신 병목 현상을 극복하기 위한 시스템 최적화 연구가 중요해질 것입니다.45
- **고급 라우팅 및 전문가 설계:** 문맥 인식 라우팅과 같은 더욱 정교한 라우팅 알고리즘과, 계층적이거나 다중 모달(multimodal) 전문가와 같은 더 복잡한 전문가 구조에 대한 탐구가 계속될 것입니다.45
- **분산 및 연합 MoE(Decentralized and Federated MoE):** 개인 정보 보호와 효율성을 위해 중앙 집중식 데이터 없이 분산된 데이터 소스에서 MoE 모델을 훈련하는 기술이 발전할 것입니다.61

#### 4.3.3 결론: 미래 AI 개발의 초석으로서의 MoE

Mixture-of-Experts 아키텍처는 학계의 틈새 개념에서 출발하여 현대 인공지능의 핵심 기둥 중 하나로 자리 잡았습니다. 이는 모델 용량을 확장하기 위한 지속 가능한 경로를 제공하며, 밀집 모델의 막대한 계산 비용이라는 장벽을 허물었습니다. MoE는 단순한 효율성 향상을 넘어, 모델의 전문화, 유연성, 확장성을 새로운 차원으로 끌어올렸습니다. 앞으로 MoE는 더욱 강력하고 효율적이며 궁극적으로 더 지능적인 시스템을 구현하는 데 핵심적인 역할을 할 것이며, 미래의 범용 인공지능(AGI) 아키텍처의 중요한 구성 요소가 될 잠재력을 지니고 있습니다.16 이 기술에 대한 지속적인 연구와 개발은 인공지능의 새로운 지평을 열어갈 것입니다.

#### **참고 자료**

1. MoE (Mixture of Expert) Explained: How Sparse Models Are Changing Deep Learning, accessed July 16, 2025, https://medium.com/@riteshpcs1994/moe-mixture-of-expert-explained-how-sparse-models-are-changing-deep-learning-f91eb796d913
2. Mixture of Experts LLMs: Key Concepts Explained - neptune.ai, accessed July 16, 2025, https://neptune.ai/blog/mixture-of-experts-llms
3. Mixture-of-Experts (MoE) LLMs: The Future of Efficient AI Models - SaM Solutions, accessed July 16, 2025, https://sam-solutions.com/blog/moe-llm-architecture/
4. MoE vs Dense vs Hybrid LLM architectures | hybridMoe – Weights & Biases - Wandb, accessed July 16, 2025, https://wandb.ai/zaiinn440/hybridMoe/reports/MoE-vs-Dense-vs-Hybrid-LLM-architectures--Vmlldzo3NzYwNzAw
5. Mixture of Experts (MoE) Architecture: A Deep Dive and Comparison of Top Open-Source Offerings, accessed July 16, 2025, https://www.architectureandgovernance.com/applications-technology/mixture-of-experts-moe-architecture-a-deep-dive-and-comparison-of-top-open-source-offerings/
6. A Comprehensive Survey of Mixture-of-Experts: Algorithms, Theory, and Applications - arXiv, accessed July 16, 2025, https://arxiv.org/html/2503.07137v1
7. A Brief Introduction to Mixtures-of-Experts - Transcendent AI, accessed July 16, 2025, https://www.transcendent-ai.com/post/a-brief-introduction-to-mixtures-of-experts
8. OUTRAGEOUSLY LARGE NEURAL NETWORKS: THE SPARSELY-GATED MIXTURE-OF-EXPERTS LAYER, accessed July 16, 2025, https://www.cs.toronto.edu/~hinton/absps/Outrageously.pdf
9. Outrageously Large Neural Networks: The Sparsely-Gated Mixture-of-Experts Layer - arXiv, accessed July 16, 2025, https://arxiv.org/abs/1701.06538
10. MoE (Mixture of Experts) for Dummies: A Beginner's Guide - Michiel Horstman, accessed July 16, 2025, https://michielh.medium.com/moe-mixture-of-experts-for-dummies-d1a7e14c1846
11. Mixture of Experts (MoE) Explained - Ultralytics, accessed July 16, 2025, https://www.ultralytics.com/glossary/mixture-of-experts-moe
12. What is Mixture of Experts (MoE)? How it Works and Use Cases - Zilliz Learn, accessed July 16, 2025, https://zilliz.com/learn/what-is-mixture-of-experts
13. Switch Transformers: Scaling to Trillion Parameter Models with Simple and Efficient Sparsity, accessed July 16, 2025, https://www.researchgate.net/publication/348403003_Switch_Transformers_Scaling_to_Trillion_Parameter_Models_with_Simple_and_Efficient_Sparsity
14. Switch Transformers: Scaling to Trillion Parameter Models with Simple and Efficient Sparsity, accessed July 16, 2025, https://arxiv.org/abs/2101.03961
15. What Is Mixture of Experts (MoE)? How It Works, Use Cases & More ..., accessed July 16, 2025, https://www.datacamp.com/blog/mixture-of-experts-moe
16. Understanding Mixture-of-Experts (MoE) Architecture in AI | by Diwakar Kumar | Medium, accessed July 16, 2025, https://medium.com/@diwakarkumar_18755/understanding-mixture-of-experts-moe-architecture-in-ai-224e3b3b9243
17. LLM Mixture of Experts Explained - TensorOps, accessed July 16, 2025, https://www.tensorops.ai/post/what-is-mixture-of-experts-llm
18. medium.com, accessed July 16, 2025, [https://medium.com/mlworks/mixture-of-experts-explained-the-next-evolution-in-ai-architecture-305902959bce#:~:text=A%20bit%20of%20history%20on,network%20through%20gating%20during%20inference.](https://medium.com/mlworks/mixture-of-experts-explained-the-next-evolution-in-ai-architecture-305902959bce#:~:text=A bit of history on,network through gating during inference.)
19. What is Mixture of Experts (MOE): Architecture, Models, and Applications | by Tahir | Medium, accessed July 16, 2025, https://medium.com/@tahirbalarabe2/what-is-mixture-of-experts-moe-architecture-models-and-applications-ca86f8beb58c
20. Adaptive Mixtures of Local Experts - Department of Computer ..., accessed July 16, 2025, https://www.cs.toronto.edu/~fritz/absps/jjnh91.pdf
21. (PDF) Adaptive Mixtures of Local Experts - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/publication/233806999_Adaptive_Mixtures_of_Local_Experts
22. Topic 1: What is Mixture-of-Experts (MoE)? - Turing Post, accessed July 16, 2025, https://www.turingpost.com/p/moe
23. The Evolution of Mixture Of Experts: From Basics To Breakthroughs | Towards AI, accessed July 16, 2025, https://towardsai.net/p/machine-learning/the-evolution-of-mixture-of-experts-from-basics-to-breakthroughs
24. The Evolution of MoE: A Review from Basics to Breakthroughs - OpenReview, accessed July 16, 2025, https://openreview.net/pdf/4cf5e9e598c59a7784467ce4fd1c2579d1be93b9.pdf
25. Switch Transformers: Scaling to Trillion Parameter Models with ..., accessed July 16, 2025, https://jmlr.org/papers/volume23/21-0998/21-0998.pdf
26. Papers Explained 95: Mixtral 8x7B | by Ritvik Rastogi - Medium, accessed July 16, 2025, https://ritvik19.medium.com/papers-explained-95-mixtral-8x7b-9e9f40ebb745
27. A Closer Look into Mixture-of-Experts in Large Language Models - arXiv, accessed July 16, 2025, https://arxiv.org/html/2406.18219v2
28. Mobile V-MoEs Scaling Down Vision Transformers via Sparse Mixture-of-Experts, accessed July 16, 2025, https://zhangtemplar.github.io/mobile-vmoe/
29. Demystifying Mixture of Experts (MoE): The future for deep GenAI systems - Pangeanic Blog, accessed July 16, 2025, https://blog.pangeanic.com/demystifying-mixture-of-experts-moe-the-future-for-deep-genai-systems
30. Switch transformers: Scaling to trillion parameter models with simple and efficient sparsity – Related Work – Interesting papers - Alastair Reid, accessed July 16, 2025, https://alastairreid.github.io/RelatedWork/papers/fedus:arxiv:2021/
31. Mixtral of Experts - arXiv, accessed July 16, 2025, http://arxiv.org/pdf/2401.04088
32. Moe for LLMs : r/ollama - Reddit, accessed July 16, 2025, https://www.reddit.com/r/ollama/comments/1iwc049/moe_for_llms/
33. MoE vs AI Dense Models: How Do They Compare in Inference? | Epoch AI, accessed July 16, 2025, https://epoch.ai/gradient-updates/moe-vs-dense-models-inference
34. [2401.04088] Mixtral of Experts - arXiv, accessed July 16, 2025, https://arxiv.org/abs/2401.04088
35. Mixtral 8x7B: A game-changing AI model by Mistral AI | SuperAnnotate, accessed July 16, 2025, https://www.superannotate.com/blog/mistral-ai-mixtral-of-experts
36. DBRX 101: Overview of Databricks 132B Parameter Open LLM - Chaos Genius, accessed July 16, 2025, https://www.chaosgenius.io/blog/dbrx/
37. Scaling to Trillion Parameter Models With Switch Transformers | by Zia Babar - Medium, accessed July 16, 2025, https://medium.com/@zbabar/scaling-to-trillion-parameter-models-with-switch-transformers-88ca5fb95e5c
38. The Rise of Mixture of Experts: Transforming Large Language Models - Gloqo AI, accessed July 16, 2025, https://www.gloqo.ai/insights/mixture_of_experts_moe_vs_dense_llms/
39. SWITCH TRANSFORMERS: SCALING TO TRILLION PARAMETER MODELS WITH SIMPLE AND EFFICIENT SPARSITY USING DEEP LEARNING - IJCRT, accessed July 16, 2025, https://ijcrt.org/papers/IJCRT2205440.pdf
40. google/switch-large-128 - Hugging Face, accessed July 16, 2025, https://huggingface.co/google/switch-large-128
41. Switch Transformers: Scaling to Trillion Parameter Models with Simple and Efficient Sparsity - cs.Princeton, accessed July 16, 2025, https://www.cs.princeton.edu/courses/archive/fall22/cos597G/lectures/lec16.pdf
42. mistralai/Mixtral-8x7B-v0.1 - Hugging Face, accessed July 16, 2025, https://huggingface.co/mistralai/Mixtral-8x7B-v0.1
43. What is a Mixture of Experts LLM (MoE)? | by Mehul Gupta | Data Science in Your Pocket, accessed July 16, 2025, https://medium.com/data-science-in-your-pocket/what-is-a-mixture-of-experts-llm-moe-8bf98846df41
44. Improving Deep Learning Performance with Mixture of Experts and Sparse Activation, accessed July 16, 2025, https://www.preprints.org/manuscript/202503.0611/v1
45. Mixture of Experts in LLMs - Al-banna Tutorials, accessed July 16, 2025, https://albanna-tutorials.com/moe.html
46. A Survey on Mixture of Experts in Large Language Models - arXiv, accessed July 16, 2025, https://arxiv.org/pdf/2407.06204
47. Advancing Expert Specialization for Better MoE - arXiv, accessed July 16, 2025, https://arxiv.org/html/2505.22323v1
48. Efficient all-to-all Collective Communication Schedules for Direct-connect Topologies - University of Washington, accessed July 16, 2025, https://homes.cs.washington.edu/~arvind/papers/all2all-dc.pdf
49. The all-to-all communication illustration of an MoE model with two... - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/figure/The-all-to-all-communication-illustration-of-an-MoE-model-with-two-experts-in-a_fig2_360961205
50. Lancet: Accelerating MoE Training via Whole Graph Computation- Communication Overlapping, accessed July 16, 2025, https://mlsys.org/media/mlsys-2024/Slides/2649.pdf
51. MegaScale-MoE: Large-Scale Communication-Efficient Training of Mixture-of-Experts Models in Production - arXiv, accessed July 16, 2025, https://arxiv.org/html/2505.11432v1
52. LSH-MoE: Communication-efficient MoE Training via Locality-Sensitive Hashing - NIPS, accessed July 16, 2025, https://proceedings.neurips.cc/paper_files/paper/2024/file/61674667d642ae52f6bb281bea90ee29-Paper-Conference.pdf
53. MoNTA: Accelerating Mixture-of-Experts Training with Network-Traffic-Aware Parallel Optimization - arXiv, accessed July 16, 2025, https://arxiv.org/html/2411.00662v1
54. DeepSpeed-MOE Advancing Mixture-of-Experts Inference - BytePlus, accessed July 16, 2025, https://www.byteplus.com/en/topic/465227
55. DeepSpeed-MoE: Advancing Mixture-of-Experts Inference and Training to Power Next-Generation AI Scale - Proceedings of Machine Learning Research, accessed July 16, 2025, https://proceedings.mlr.press/v162/rajbhandari22a/rajbhandari22a.pdf
56. Training Overview and Features - DeepSpeed, accessed July 16, 2025, https://www.deepspeed.ai/training/
57. [2201.05596] DeepSpeed-MoE: Advancing Mixture-of-Experts Inference and Training to Power Next-Generation AI Scale - ar5iv, accessed July 16, 2025, https://ar5iv.labs.arxiv.org/html/2201.05596
58. Brief Review - DeepSpeed-MoE: Advancing Mixture-of-Experts Inference and Training to Power Next-Generation AI Scale - Sik-Ho Tsang, accessed July 16, 2025, https://sh-tsang.medium.com/brief-review-deepspeed-moe-advancing-mixture-of-experts-inference-and-training-to-power-2c1350c7ff47
59. EC-DiT: Scaling Diffusion Transformers with Adaptive Expert-Choice Routing - arXiv, accessed July 16, 2025, https://arxiv.org/html/2410.02098v5
60. Autonomy-of-Experts Models - arXiv, accessed July 16, 2025, https://arxiv.org/html/2501.13074v1
61. A Survey of Mixture of Experts Models: Architectures and Applications in Business and Finance - Preprints.org, accessed July 16, 2025, https://www.preprints.org/manuscript/202505.1603/v1
62. Omni-Router: Sharing Routing Decisions in Sparse Mixture-of-Experts for Speech Recognition - arXiv, accessed July 16, 2025, https://arxiv.org/html/2507.05724v1
63. Dense Backpropagation Improves Routing for Sparsely-Gated Mixture-of-Experts | OpenReview, accessed July 16, 2025, https://openreview.net/forum?id=huy8g3iKy0
64. Numerical stability - Wikipedia, accessed July 16, 2025, https://en.wikipedia.org/wiki/Numerical_stability
65. 5.4. Numerical Stability and Initialization - Dive into Deep Learning, accessed July 16, 2025, http://d2l.ai/chapter_multilayer-perceptrons/numerical-stability-and-init.html
66. Decoding Knowledge Attribution in Mixture-of-Experts: A Framework of Basic-Refinement Collaboration and Efficiency Analysis - arXiv, accessed July 16, 2025, https://arxiv.org/html/2505.24593v1
67. FLAME-MoE: A Transparent End-to-End Research Platform for Mixture-of-Experts Language Models - arXiv, accessed July 16, 2025, https://arxiv.org/html/2505.20225v1
68. Fusing LLM Capabilities with Routing Data - arXiv, accessed July 16, 2025, https://arxiv.org/html/2507.10540v1
69. Scaling Vision with Sparse Mixture of Experts - Google Research, accessed July 16, 2025, https://research.google/blog/scaling-vision-with-sparse-mixture-of-experts/
70. Scaling Vision with Sparse Mixture of Experts - NIPS, accessed July 16, 2025, https://papers.neurips.cc/paper_files/paper/2021/file/48237d9f2dea8c74c2a72126cf63d933-Paper.pdf
71. google-research/vmoe - GitHub, accessed July 16, 2025, https://github.com/google-research/vmoe
72. AdaMV-MoE: Adaptive Multi-Task Vision Mixture-of-Experts - CVF Open Access, accessed July 16, 2025, https://openaccess.thecvf.com/content/ICCV2023/papers/Chen_AdaMV-MoE_Adaptive_Multi-Task_Vision_Mixture-of-Experts_ICCV_2023_paper.pdf
73. [2203.07413] Switch Trajectory Transformer with Distributional Value Approximation for Multi-Task Reinforcement Learning - arXiv, accessed July 16, 2025, https://arxiv.org/abs/2203.07413
74. A Survey of Mixture of Experts Models: Architectures and Applications in Business and Finance | Sciety, accessed July 16, 2025, https://sciety.org/articles/activity/10.20944/preprints202505.1603.v1
75. AT-MoE: Adaptive Task-planning Mixture of Experts via LoRA Approach - arXiv, accessed July 16, 2025, https://arxiv.org/html/2410.10896v1
76. [2402.12550] Multilinear Mixture of Experts: Scalable Expert Specialization through Factorization - arXiv, accessed July 16, 2025, https://arxiv.org/abs/2402.12550
77. Mixture of Experts Made Intrinsically Interpretable - arXiv, accessed July 16, 2025, https://arxiv.org/html/2503.07639v1
78. A Visual Guide to Mixture of Experts (MoE) in LLMs - YouTube, accessed July 16, 2025, https://www.youtube.com/watch?v=sOPDGQjFcuM
79. M³ViT: Mixture-of-Experts Vision Transformer for Efficient Multi-task Learning with Model-Accelerator Co-design | OpenReview, accessed July 16, 2025, https://openreview.net/forum?id=cFOhdl1cyU-
80. [2501.16352] Mixture of Experts (MoE): A Big Data Perspective - arXiv, accessed July 16, 2025, https://arxiv.org/abs/2501.16352


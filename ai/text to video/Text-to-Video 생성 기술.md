# Text-to-Video 생성 기술
[Text-to-Video 생성 기술](./index.md)


Text-to-Video 기술은 자연어 텍스트 설명을 동적인 비디오 클립으로 변환하는 생성형 인공지능(Generative AI)의 한 분야이다.1 이 기술은 단순히 텍스트를 시각화하는 것을 넘어, 애니메이션, 사실적인 장면, 심지어 음성 해설까지 최소한의 인간 개입으로 생성해내는 정교한 시스템으로 발전하였다.1 그 중요성은 콘텐츠 제작의 근본적인 장벽을 극적으로 낮추는 데 있다. 전통적인 비디오 제작 방식은 고가의 장비, 전문 인력, 그리고 상당한 시간과 비용을 요구했으나, Text-to-Video AI는 이러한 자원 없이도 개인이나 소규모 조직이 전문가 수준의 시각 콘텐츠를 제작할 수 있는 길을 열어주었다. 이는 곧 '콘텐츠 제작의 민주화'를 의미한다.1

이 기술의 진정한 파급력은 단순한 자동화나 효율성 향상을 초월한다. 인간의 창의성을 대체하는 것이 아니라, 오히려 증강시키는 강력한 협업 도구로서 기능하기 때문이다.1 창작 과정에서 기술적으로 까다롭고 반복적인 부분은 AI가 담당하고, 인간은 서사 구조, 감성적 깊이, 전략적 방향성과 같은 본질적인 창의성에 더욱 집중할 수 있게 된다. 이러한 변화는 콘텐츠 창작의 패러다임을 근본적으로 재편하고 있다. 과거의 비디오 제작이 촬영, 연기, 편집 등 물리적이고 노동 집약적인 '생산(production)'의 과정이었다면 1, Text-to-Video 기술은 이를 자연어라는 '지시(direction)'의 과정으로 전환시킨다.1 이는 창작의 핵심이 '어떻게 만들 것인가(how)'라는 방법론적 고민에서 '무엇을 만들 것인가(what)'라는 개념적 구상으로 이동함을 시사한다. 결과적으로, 이 기술은 창의적인 아이디어를 가진 누구나 '감독'의 역할을 수행할 수 있게 함으로써 산업 구조 자체를 재편할 잠재력을 내포하고 있다.

본 보고서는 Text-to-Video 기술을 다층적으로 분석하여 그 본질을 규명하고자 한다. 먼저 기술의 역사적 계보를 추적하여 생성형 AI의 기원부터 최신 모델의 등장까지 그 발전 과정을 살펴본다. 이어서 확산 모델과 트랜스포머 아키텍처 등 핵심 기술의 작동 원리를 심층적으로 분석한다. OpenAI의 Sora, Google의 Lumiere, Runway의 Gen-2 등 시장을 주도하는 주요 모델들의 아키텍처와 특징을 비교 분석하여 기술 현황을 조망한다. 나아가 마케팅, 영화, 교육 등 다양한 산업 분야에 미치는 파급 효과와 응용 사례를 탐구하고, 물리 법칙 이해 부족, 장기적 일관성 문제와 같은 기술적 한계와 딥페이크, 저작권 등 심각한 윤리적 과제를 고찰한다. 마지막으로, 이 모든 논의를 종합하여 Text-to-Video 기술의 미래 전망과 우리가 나아가야 할 방향을 제시한다.



Text-to-Video 기술의 뿌리는 생성형 AI의 역사와 궤를 같이한다. 인공지능 연구의 초기 단계인 1950년대와 60년대에 그 씨앗이 뿌려졌으며, 1957년 프랭크 로젠블랫이 개발한 퍼셉트론(Perceptron)과 같은 초기 신경망 모델에서 그 기원을 찾을 수 있다.6 그러나 본격적인 생성 모델의 등장은 2014년 이안 굿펠로우가 제안한 생성적 적대 신경망(Generative Adversarial Networks, GANs)과 함께 이루어졌다.6 GAN은 생성자(Generator)와 판별자(Discriminator)라는 두 개의 신경망이 서로 경쟁하며 학습하는 구조를 통해, 이전에는 불가능했던 수준의 사실적인 이미지, 비디오, 오디오를 생성하는 결정적인 변곡점을 마련했다.6

한편, 비디오와 같은 순차적 데이터를 처리하기 위한 기반 기술 역시 꾸준히 발전해왔다. 1980년대 후반에 등장한 순환 신경망(Recurrent Neural Networks, RNNs)과 1997년 제프 슈미트후버와 제프 호크라이터가 개발한 장단기 메모리(Long Short-Term Memory, LSTM) 네트워크는 시퀀스 데이터 내의 시간적 의존성을 학습하는 능력을 크게 향상시켰다.2 이러한 모델들은 음성 인식, 기계 번역 등 다양한 분야에서 활용되며 비디오 데이터 처리의 이론적 기초를 다졌다.


Text-to-Video 기술 발전의 직접적인 촉매제는 텍스트-이미지(Text-to-Image) 모델의 경이로운 성공이었다. 이 성공의 중심에는 2017년 Google 연구팀이 발표한 트랜스포머(Transformer) 아키텍처가 있다.7 트랜스포머는 RNN의 순차적 처리 방식에서 벗어나, '어텐션 메커니즘(Attention Mechanism)'을 통해 시퀀스 내 모든 요소 간의 관계를 병렬적으로 계산함으로써 장거리 의존성 문제를 효과적으로 해결했다. 이는 자연어 처리(NLP) 분야에 혁명을 일으켰고, 곧 텍스트를 조건으로 하는 생성 모델의 핵심 기반 기술로 자리 잡았다.

2021년 OpenAI의 DALL-E, 2022년 Google의 Imagen 및 Stability AI의 Stable Diffusion과 같은 텍스트-이미지 확산 모델(Diffusion Models)들은 텍스트 프롬프트만으로 놀라울 정도로 정교하고 사실적인 이미지를 생성해내며 기술의 성숙도를 입증했다.8 Text-to-Video 모델은 본질적으로 이러한 텍스트-이미지 모델에 '시간(time)'이라는 새로운 차원을 추가하는 과제에서 출발했다.8 즉, 정적인 이미지 한 장을 생성하는 것을 넘어, 시간의 흐름에 따라 논리적으로 연결되는 이미지들의 연속, 즉 비디오를 생성하는 것으로 기술적 목표가 확장된 것이다.


2022년 하반기는 Text-to-Video 기술의 역사에서 중요한 분기점으로 기록된다. 텍스트-이미지 모델의 성공으로 확보된 '기술적 모멘텀'이 비디오 영역으로 일제히 전이되면서, 주요 빅테크 기업들이 동시다발적으로 초기 모델들을 발표하며 본격적인 기술 경쟁의 서막을 열었다. 이는 AI 분야에서 하나의 기술적 돌파구가 연쇄적인 혁신을 촉발하는 특징적인 발전 양상을 명확히 보여준다. 2022년 초중반 DALL-E 2, Imagen, Stable Diffusion 등이 텍스트-이미지 분야에서 최고 수준의 성능을 달성하며 확산 모델과 트랜스포머라는 기술 스택의 유효성을 입증했고 10, 주요 연구 그룹들은 이 검증된 기술을 기반으로 거의 동시에 Text-to-Video 모델 개발에 착수했다.8 따라서 2022년 하반기의 '모델 폭발'은 개별 연구의 산발적 결과가 아닌, 공유된 기술 기반 위에서 펼쳐진 경쟁적 혁신의 물결로 해석하는 것이 타당하다.

- **2022년: 경쟁의 서막**
  - **CogVideo (5월):** 중국 칭화대학 연구팀이 발표한 초기 대규모 Text-to-Video 모델로, 94억 개의 파라미터를 가졌다. 이는 중국어 입력에 특화되었으며, 기술 경쟁의 글로벌 확산을 알리는 신호탄이었다.2
  - **Make-A-Video (9월, Meta):** Meta AI가 공개한 모델로, 방대한 텍스트-이미지 데이터와 레이블 없는 비디오 데이터를 함께 활용하는 독창적인 접근법을 제시했다. 이는 텍스트-비디오 쌍 데이터셋의 부족 문제를 우회하고, 세상이 어떻게 움직이는지를 비디오 데이터로부터 자율적으로 학습하도록 한 중요한 시도였다.2
  - **Phenaki (9월, Google):** Google Research가 발표한 모델로, 가변적인 길이의 비디오 생성에 초점을 맞췄다. 시간에 따라 변화하는 프롬프트(a story)를 입력받아 수 분 길이의 일관된 비디오를 생성할 수 있는 가능성을 제시하며, 긴 서사 구조를 다루는 데 있어 중요한 진전을 이루었다.12
  - **Imagen Video (10월, Google):** Google의 고품질 텍스트-이미지 모델인 Imagen을 비디오 영역으로 확장한 모델이다. 3D U-Net과 같은 아키텍처를 활용하여 높은 시각적 충실도를 달성하는 데 중점을 두었다.2
- **2023년: 기술의 성숙과 확산**
  - **Gen-1 & Gen-2 (2월, 3월, Runway):** 스타트업 RunwayML은 Gen-1을 통해 기존 비디오에 텍스트나 이미지로 스타일을 적용하는 Video-to-Video 편집 기능을 선보였고 12, 곧이어 Gen-2를 출시하며 본격적인 Text-to-Video 생성 기능을 제공하기 시작했다. 이는 기술이 연구 단계를 넘어 실제 창작 도구로 상용화되는 과정을 보여주었다.2
  - **Stable Video Diffusion (11월, Stability AI):** Stability AI가 공개한 오픈소스 모델로, 텍스트-이미지 모델인 Stable Diffusion의 성공을 비디오 영역으로 확장했다. 이 모델의 공개는 연구자와 개발자들이 기술에 더 쉽게 접근하고 생태계를 확장하는 데 크게 기여했다.17
- **2024년 이후: 패러다임의 전환**
  - **Lumiere (1월, Google):** Google Research가 발표한 논문에서 소개된 모델로, 'Space-Time U-Net'이라는 혁신적인 아키텍처를 통해 시간적 일관성을 획기적으로 개선했다. 비디오 전체를 한 번에 생성하는 방식으로 기존 모델들의 고질적인 문제였던 움직임의 부자연스러움을 크게 줄였다.2
  - **Sora (2월, OpenAI):** OpenAI가 공개한 Sora는 최대 1분 길이의 고해상도(1080p) 비디오를 놀라운 일관성과 사실성으로 생성해내며 시장에 큰 충격을 주었다. OpenAI는 Sora를 단순한 비디오 생성기를 넘어, 물리적 세계를 이해하고 시뮬레이션하는 '세계 시뮬레이터(World Simulator)'로 포지셔닝하며 기술의 궁극적인 비전을 제시했다.5
  - **Veo (5월, Google):** Google I/O에서 공개된 플래그십 모델로, Imagen Video, Phenaki, Lumiere 등 이전 연구들의 성과를 집대성했다. 1분 이상의 1080p 비디오 생성 능력과 함께 '타임랩스', '항공샷'과 같은 영화적 연출 용어를 이해하는 등 정교한 제어 능력을 선보였다.21
  - **Aleph (7월, Runway):** Runway는 Aleph를 통해 기술의 초점을 '생성'에서 '편집'으로 전환했다. 기존 비디오를 텍스트로 수정하는 '인-컨텍스트(in-context) 편집' 기능을 통해 전문 창작자들의 후반 작업 워크플로우를 혁신하고자 하는 명확한 목표를 드러냈다.22


Text-to-Video 기술의 발전은 확산 모델(Diffusion Models)과 트랜스포머(Transformer)라는 두 가지 핵심 아키텍처의 발전과 융합에 깊이 의존하고 있다. 이 장에서는 각 기술의 기본 원리를 분석하고, 이들이 결합하여 어떻게 시공간적 데이터를 생성하는지 심층적으로 탐구한다.


확산 모델은 데이터에 점진적으로 노이즈를 추가했다가 다시 제거하는 과정을 학습함으로써 고품질의 샘플을 생성하는 모델이다. 이 과정은 크게 순방향 과정(Forward Process)과 역방향 과정(Reverse Process)으로 나뉜다.8

- **DDPM (Denoising Diffusion Probabilistic Models):** DDPM은 확산 모델의 대표적인 프레임워크이다.

  - **순방향 과정 (Forward Process):** 원본 데이터 $x_0$에 $T$ 타임스텝에 걸쳐 점진적으로 가우시안 노이즈를 추가한다. $t$ 시점의 데이터 $x_t$는 $t-1$ 시점의 데이터 $x_{t-1}$로부터 다음과 같은 마르코프 연쇄(Markov chain)를 통해 정의된다.27
    $$
    q(x_t | x_{t-1}) = \mathcal{N}(x_t; \sqrt{1-\beta_t}x_{t-1}, \beta_t\mathbf{I})
    $$
    여기서 $\beta_t$는 $t$ 시점에서 추가되는 노이즈의 양을 조절하는 분산 스케줄(variance schedule)이며, $\mathcal{N}$은 정규분포를 의미한다. 이 과정을 반복하면 $x_T$는 순수한 가우시안 노이즈에 가까워진다.

  - **역방향 과정 (Reverse Process):** 순방향 과정의 역을 수행하는 것으로, 순수 노이즈 $x_T$에서 시작하여 점진적으로 노이즈를 제거해 원본 데이터 $x_0$를 복원한다. 이 과정은 신경망(모델 $\theta$)을 통해 학습되며, 다음과 같이 모델링된다.27
    $$
    p_\theta(x_{t-1} | x_t) = \mathcal{N}(x_{t-1}; \mu_\theta(x_t, t), \Sigma_\theta(x_t, t))
    $$
    모델은 각 시점 $t$에서 $x_t$를 보고 $x_{t-1}$의 분포, 즉 평균 $\mu_\theta$와 분산 $\Sigma_\theta$를 예측하도록 학습된다.

- **LDM (Latent Diffusion Models):** DDPM은 고해상도 이미지와 같이 차원이 큰 데이터를 직접 다룰 때 막대한 계산 비용이 발생한다. 잠재 확산 모델(LDM)은 이 문제를 해결하기 위해, 고차원의 픽셀 공간(pixel space)이 아닌 저차원의 잠재 공간(latent space)에서 확산 과정을 수행한다.30 VAE(Variational Autoencoder)와 같은 오토인코더를 사용하여 이미지를 의미적으로 압축된 잠재 표현 $z$로 변환하고($z = \mathcal{E}(x)$), 이 잠재 공간에서 확산 및 노이즈 제거 과정을 수행한다. 최종적으로 생성된 잠재 표현 $z_0$는 VAE의 디코더를 통해 다시 픽셀 공간으로 복원된다($\hat{x}_0 = \mathcal{D}(z_0)$).8 이 방식은 계산 효율성을 극적으로 향상시켜 고해상도 이미지 및 비디오 생성의 표준으로 자리 잡았다.

- **DDPM의 Simplified Loss Function:** 확산 모델 학습의 핵심은 역방향 과정에서 추가된 노이즈를 정확하게 예측하는 것이다. DDPM 논문에서는 복잡한 변분 하한(variational lower bound)을 최적화하는 대신, 모델이 실제 노이즈 $\epsilon$과 예측된 노이즈 $\epsilon_\theta$ 간의 평균 제곱 오차(MSE)를 최소화하도록 하는 단순화된 손실 함수를 제안했다.27
  $$
  L_{simple}(\theta) = E_{t,x_0,\epsilon}[\|\epsilon - \epsilon_{\theta}(\sqrt{\bar{\alpha}_t}x_0 + \sqrt{1-\bar{\alpha}_t}\epsilon, t)\|^2]
  $$
  이 손실 함수는 모델 $\epsilon_\theta$가 노이즈가 섞인 데이터 $x_t = \sqrt{\bar{\alpha}_t}x_0 + \sqrt{1-\bar{\alpha}_t}\epsilon$와 타임스텝 $t$를 입력받아, 원본 데이터 $x_0$에 추가되었던 실제 노이즈 $\epsilon$을 직접 예측하도록 훈련시킨다. 이 직관적인 목표 함수는 학습을 안정시키고 뛰어난 성능을 보여, 이후 대부분의 확산 모델에서 채택되었다.27


- **트랜스포머의 역할:** 트랜스포머는 시퀀스 데이터 내의 모든 요소 간의 관계를 한 번에 병렬적으로 계산하는 '셀프 어텐션(Self-Attention)' 메커니즘을 기반으로 한다.7 이는 RNN이 가진 순차적 처리의 한계와 장기 의존성 문제를 극복하게 해주었다. Text-to-Video 모델에서 트랜스포머는 두 가지 중요한 역할을 수행한다. 첫째, 입력된 텍스트 프롬프트의 의미적, 구문적 구조를 이해한다. 둘째, 비디오의 시공간적 구조, 즉 프레임 내 픽셀/패치 간의 공간적 관계와 프레임 시퀀스 간의 시간적 관계를 모델링한다.

- **Scaled Dot-Product Attention:** 트랜스포머의 핵심 연산인 스케일링된 내적 어텐션은 쿼리(Query, Q), 키(Key, K), 값(Value, V)이라는 세 가지 벡터를 사용하여 입력 시퀀스의 각 요소가 다른 모든 요소에 얼마나 '주의'를 기울여야 할지를 가중치로 계산한다.34 그 수식은 다음과 같다.
  $$
  \text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
  $$
  이 수식의 작동 과정은 다음과 같이 단계별로 이해할 수 있다.

  1. **유사도 계산 ($QK^T$):** 특정 쿼리($Q$의 한 행)가 다른 모든 키($K$의 모든 행)와 얼마나 관련이 있는지 내적(dot product)을 통해 계산한다. 내적 값이 클수록 두 요소 간의 관련성이 높음을 의미한다.34
  2. **스케일링 ($/\sqrt{d_k}$):** 키 벡터의 차원 $d_k$가 클수록 내적 값의 분산이 커져 소프트맥스 함수의 기울기가 0에 가까워지는 문제(vanishing gradient)가 발생할 수 있다. 이를 방지하기 위해 $\sqrt{d_k}$로 나누어 값을 안정화시킨다.35
  3. **가중치 정규화 ($softmax$):** 스케일링된 유사도 점수에 소프트맥스 함수를 적용하여 합이 1이 되는 확률 분포, 즉 '어텐션 가중치(attention weights)'로 변환한다. 이 가중치는 각 키에 해당하는 값에 얼마나 집중할지를 결정한다.34
  4. **가중합 계산 ($...V$):** 마지막으로, 계산된 어텐션 가중치를 해당 값(Value) 벡터에 곱하여 가중합을 구한다. 이를 통해 현재 쿼리와 관련성이 높은 요소들의 정보는 강조하고, 관련성이 낮은 요소들의 정보는 약화시킨 문맥적 표현(contextual representation)을 얻는다.34


Text-to-Video 기술의 최근 돌파구는 확산 모델의 강력한 '생성적 표현력'과 트랜스포머의 뛰어난 '구조적 문맥 이해' 능력이 융합된 필연적인 결과이다. 확산 모델은 픽셀 수준의 복잡한 분포를 학습하여 사실적인 이미지를 생성하는 데 탁월하지만 8, 트랜스포머는 텍스트나 시퀀스 데이터 내의 장기적이고 복잡한 관계를 파악하는 데 특화되어 있다.7 비디오는 '사실적인 각 프레임'과 '프레임 간의 논리적 연결'이 모두 필요하므로, 이 두 기술의 융합은 비디오 생성이라는 과제를 해결하기 위한 본질적인 과정이었다.

- **Diffusion Transformer (DiT):** DiT는 이 융합의 현재 정점이라 할 수 있다. 기존 확산 모델에서 노이즈 예측을 위해 널리 사용되던 U-Net 아키텍처를 트랜스포머 블록으로 대체한 구조이다.17 DiT는 잠재 공간의 시공간 패치들을 언어 모델의 '토큰'처럼 취급하여, 트랜스포머가 이들의 복잡한 시공간적 관계를 학습하게 한다. 이 접근법은 모델의 파라미터 수를 늘릴수록 성능이 향상되는 뛰어난 확장성(scalability)을 보여주었으며, Sora와 같은 최신 고성능 모델의 핵심 아키텍처로 채택되었다.38
- **시간적 일관성(Temporal Consistency) 확보:** 비디오 생성의 가장 큰 난제는 프레임 간에 객체의 정체성, 스타일, 배경, 그리고 움직임이 일관되게 유지되도록 하는 것이다.3 초기 모델들은 프레임별로 독립적으로 생성하는 경향이 있어 깜빡임(flickering)이나 객체가 갑자기 변하는 등의 문제가 발생했다. 이를 해결하기 위한 주요 기술적 접근법은 다음과 같다.42
  - **3D 컨볼루션/어텐션:** 이미지 처리에 사용되는 2D 연산을 시공간 차원을 모두 고려하는 3D 연산으로 확장한다. 예를 들어, 3D U-Net은 컨볼루션 필터를 시간 축으로 확장하여 여러 프레임을 동시에 처리하며 시간적 정보를 직접 모델링한다.8
  - **프레임 간 어텐션(Cross-Frame Attention):** 한 프레임의 특정 패치(쿼리)가 다른 프레임의 패치들(키, 값)과 어텐션 연산을 수행하도록 설계한다. 이를 통해 모델은 객체가 시간의 흐름에 따라 어떻게 변화하고 움직이는지에 대한 관계를 명시적으로 학습할 수 있다.3
  - **공유 노이즈/가중치(Shared Noise/Weights):** VideoFusion 모델과 같이, 여러 프레임을 생성할 때 동일한 기저 노이즈(base noise)를 공유하거나 네트워크의 일부 가중치를 공유하여 생성 과정 자체에 일관성을 강제하는 방법도 사용된다.2

이처럼 Text-to-Video 아키텍처는 확산 모델의 생성 능력과 트랜스포머의 문맥 이해 능력을 결합하고, 시간적 일관성을 확보하기 위한 다양한 시공간 모델링 기법을 통합하는 방향으로 진화하고 있다.


Text-to-Video 분야는 OpenAI, Google, Runway 등 소수의 빅테크 기업과 유망 스타트업이 기술 개발을 주도하는 양상을 띤다. 이들은 각기 다른 기술 철학과 아키텍처를 바탕으로 시장의 패권을 다투고 있다. 이 장에서는 각사의 대표 모델인 Sora, Lumiere/Veo, Gen-2/Aleph를 중심으로 그 기술적 특성과 전략적 방향성을 심층 비교 분석한다.


2024년 2월 발표된 Sora는 기존 모델들의 한계를 뛰어넘는 성능으로 시장에 큰 충격을 주었다. OpenAI는 Sora를 단순한 영상 제작 도구가 아닌, 물리적 세계의 법칙과 상호작용을 이해하고 시뮬레이션하는 '세계 시뮬레이터(World Simulator)'로 구축하는 것을 장기적인 목표로 제시했다.5

- **핵심 아키텍처:** Sora의 근간은 **확산 트랜스포머(Diffusion Transformer, DiT)**이다.17 이는 잠재 공간에서 작동하며, 노이즈 예측을 위해 U-Net 대신 트랜스포머를 사용함으로써 뛰어난 확장성과 성능을 확보했다.
- **Spacetime Latent Patches:** Sora의 가장 혁신적인 개념 중 하나는 비디오 데이터를 '시공간 잠재 패치(Spacetime Latent Patches)'로 처리하는 방식이다. 먼저 **Video Compression Network**라는 시각 인코더(VAE와 유사)를 통해 원본 비디오를 시간적, 공간적으로 압축된 저차원 잠재 공간으로 변환한다.38 그 후, 이 압축된 표현을 시공간에 걸친 작은 데이터 큐브(패치)들로 분할한다. 이 패치들은 트랜스포머가 처리하는 '토큰'과 같은 역할을 하며, 이 방식을 통해 Sora는 다양한 해상도, 길이, 종횡비의 비디오와 이미지를 통일된 형식으로 학습하고 생성할 수 있는 유연성을 확보했다.38
- **대규모 학습과 Re-captioning:** Sora의 뛰어난 성능은 방대한 양의 비디오 데이터 학습에 기인한다. 특히, OpenAI는 DALL-E 3에서 처음 도입했던 're-captioning' 기술을 비디오 데이터에 적용했다. 이는 먼저 매우 상세한 캡션을 생성하는 캡셔너 모델을 훈련시킨 후, 이 모델을 사용하여 학습 데이터셋의 모든 비디오에 대해 풍부하고 정확한 텍스트 설명을 자동으로 생성하는 기법이다. 이렇게 질적으로 향상된 캡션으로 학습함으로써 Sora는 복잡한 텍스트 프롬프트에 대한 이해도를 극대화하고, 결과적으로 비디오의 품질과 텍스트 충실도를 획기적으로 개선했다.38
- **공식 출시 및 플랜:** Sora는 2024년 12월, sora.com을 통해 공식 출시되었으며, ChatGPT Plus 및 Pro 구독자를 대상으로 서비스된다. Plus 플랜은 월 최대 50개의 480p 비디오 생성을, Pro 플랜은 더 많은 사용량과 높은 해상도(최대 1080p), 긴 길이(최대 20초)의 비디오 생성을 지원한다.46


Google은 Lumiere와 Veo를 통해 Text-to-Video 기술의 핵심 난제인 '시간적 일관성'을 정면으로 공략하는 접근법을 선보였다. 이들의 핵심은 비디오의 공간과 시간을 분리하지 않고 하나의 단위로 처리하는 것이다.

- **Space-Time U-Net (STUnet):** Lumiere에 도입된 STUnet은 공간(프레임 내 픽셀) 차원과 시간(프레임 간) 차원을 동시에 다운샘플링하고 업샘플링하는 독자적인 U-Net 기반 아키텍처이다.47 이 구조는 모델이 비디오의 시공간적 특징을 전체적으로 파악하도록 유도한다.
- **Single Pass Generation:** 기존의 많은 모델들이 주요 장면(키프레임)을 먼저 생성한 뒤 그 사이를 보간(temporal super-resolution)하는 다단계 방식을 사용하는 것과 달리, Lumiere는 비디오의 전체 시간 길이를 한 번의 패스(single pass)로 직접 생성한다.19 이 접근법은 키프레임 방식에서 발생하기 쉬운 움직임의 불연속성이나 스타일의 비일관성 문제를 근본적으로 해결하여, 매우 부드럽고 일관된 모션을 가진 비디오를 생성하는 데 결정적인 역할을 한다.47
- **다양한 기능:** Google의 모델들은 단순한 텍스트-비디오 변환을 넘어 다채로운 콘텐츠 제작 및 편집 기능을 제공한다. 정지 이미지를 생동감 있게 움직이게 하는 **Image-to-Video** (예: 베르메르의 '진주 귀걸이를 한 소녀' 그림이 미소 짓고 윙크하는 영상), 영상의 일부를 자연스럽게 채우거나 수정하는 **Video Inpainting**, 특정 화가의 화풍이나 재질(예: 종이접기)을 영상 전체에 적용하는 **Stylized Generation**, 그리고 이미지의 특정 부분만 움직이게 하는 **Cinemagraphs** 기능 등이 대표적이다.47


Runway는 AI를 전문 창작자들의 워크플로우에 통합시키는 실용적인 도구로 개발하는 데 집중하고 있다. 이들의 모델은 완전 자율적인 생성보다는 사용자의 의도를 정교하게 반영하고 제어하는 데 초점을 맞춘다.

- **Gen-2의 다중 모드 아키텍처:** Gen-2는 텍스트뿐만 아니라 이미지, 비디오 클립 등 다양한 형태의 입력을 동시에 처리하는 **다중 모드(multimodal)** 시스템이다.16 그 아키텍처는 

  **텍스트 인코더**와 **이미지 인코더**가 각각의 입력에서 특징을 추출하고, **Cross-Modal Fusion Layer**가 이들을 하나의 풍부한 표현으로 통합하는 구조를 가진다.45 이를 통해 "이 이미지 속 캐릭터가 [텍스트] 석양이 지는 숲속을 걷는 영상"과 같이 복합적인 지시를 정교하게 수행할 수 있으며, 이는 창작자에게 높은 수준의 제어권을 제공하려는 Runway의 철학을 명확히 보여준다.

- **Aleph의 패러다임 전환:** 2025년 7월 공개된 Aleph는 기술의 패러다임을 '생성(Generation)'에서 '편집(Editing)'으로 전환했다. Aleph는 처음부터 비디오를 만드는 것이 아니라, 사용자가 업로드한 기존 비디오를 텍스트 프롬프트로 수정하는 **'인-컨텍스트 비디오 편집(in-context video editing)'**에 특화되어 있다.23 예를 들어, 촬영된 영상의 카메라 앵글을 바꾸거나, 특정 인물이나 객체를 제거하고, 낮 장면을 밤 장면으로 조명을 변경하는 등의 후반 작업(post-production)을 텍스트 지시만으로 수행할 수 있다.24 이는 전문 영상 제작자들의 워크플로우를 혁신하고, 재촬영 비용 없이 창의적인 수정을 가능하게 하는 것을 목표로 한다.


세 진영의 모델들은 각기 다른 철학과 기술적 접근을 통해 Text-to-Video의 미래를 그려나가고 있다. Sora와 Google이 '세계 시뮬레이션'이라는 거대 담론 아래 최대한 사실적이고 일관된 영상을 '생성'하는 능력 자체를 극대화하는 데 집중하는 반면, Runway는 전문 창작자들의 실제 작업 과정에 깊숙이 통합되어 정교한 '제어와 편집'을 제공하는 강력한 도구를 지향한다. 이러한 전략적 차이는 각 모델의 아키텍처 설계에도 명확히 반영되어 있다. Sora는 무한한 확장 가능성을 지닌 DiT를, Lumiere는 시간적 일관성이라는 특정 문제에 특화된 STUnet을, Runway는 다양한 사용자 입력을 유연하게 통합하는 다중 모드 융합 아키텍처를 채택함으로써 각자의 목표를 효과적으로 달성하고 있다.

| 특징 (Feature)    | OpenAI Sora                                         | Google Lumiere / Veo                              | Runway Gen-2 / Aleph                                         |
| ----------------- | --------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------------ |
| **핵심 철학**     | 세계 시뮬레이터 (World Simulator)                   | 현실적이고 일관된 모션 합성                       | 창작자를 위한 제어 및 편집 도구                              |
| **주요 아키텍처** | 확산 트랜스포머 (DiT)                               | 시공간 U-Net (STUnet)                             | 다중 모드 융합 트랜스포머                                    |
| **생성 방식**     | Spacetime Latent Patches 기반 생성                  | 단일 패스(Single Pass) 전체 생성                  | 다중 모드 입력 기반 생성 / 인-컨텍스트 편집                  |
| **강점**          | 긴 비디오 길이(1분+), 높은 해상도, 복잡한 장면 이해 | 뛰어난 시간적 일관성, 부드러운 모션               | 정교한 스타일 제어, 이미지/비디오 기반 생성, 비디오 편집     |
| **주요 기능**     | Text-to-Video, Video-to-Video, 이미지 애니메이션    | Text-to-Video, Image-to-Video, 스타일화, 인페인팅 | Text/Image/Video-to-Video (Gen-2), 인-컨텍스트 비디오 편집 (Aleph) |
| **출시/공개**     | 2024년 2월 (발표), 12월 (출시)                      | 2024년 1월 (Lumiere 발표), 5월 (Veo 발표)         | 2023년 3월 (Gen-2), 2025년 7월 (Aleph)                       |
| **인용 Snippets** | 5                                                   | 19                                                | 23                                                           |


Text-to-Video 기술은 더 이상 연구실의 실험적 기술이 아니라, 다양한 산업의 콘텐츠 제작 방식을 근본적으로 바꾸는 현실적인 도구로 자리매김하고 있다. 이 기술은 비용 절감과 시간 단축이라는 직접적인 이점을 넘어, 각 산업의 가치 사슬을 재편하고 새로운 비즈니스 모델의 등장을 촉진하고 있다.

- **마케팅 및 광고:** 마케팅 분야는 Text-to-Video 기술의 가장 즉각적인 수혜자로 꼽힌다. 브랜드들은 역동적인 소셜 미디어 광고, 제품 기능 설명 비디오, 내부 교육 자료 등을 기존 방식 대비 훨씬 낮은 비용과 짧은 시간 안에 제작할 수 있다.1 Hubspot의 조사에 따르면 마케터의 92%가 비디오 콘텐츠의 높은 투자수익률(ROI)을 인정하고 있으며, 이 기술은 이러한 시장의 요구를 충족시키는 가장 효율적인 수단으로 부상하고 있다.4 실제 한 광고 캠페인 사례에서는 AI를 활용하여 콘셉트 구상부터 최종 광고안 선택까지 걸리는 전체 타임라인을 60% 단축하는 성과를 거두기도 했다.1 이를 통해 기업들은 더 많은 시안을 테스트하고 데이터 기반의 의사결정을 내릴 수 있게 된다.
- **영화 및 엔터테인먼트:** 영화 산업에서는 사전 시각화(pre-visualization)와 콘셉트 개발 단계에서 이 기술의 활용 가치가 높다. 감독과 제작팀은 값비싼 세트나 배우를 섭외하기 전에 스크립트의 특정 장면을 영상으로 미리 구현해봄으로써, 다양한 시각적 접근법을 탐색하고 제작 방향에 대한 공감대를 형성할 수 있다.1 특히 예산이 한정된 독립 영화 제작자들에게는 전문적인 시각 효과(VFX)를 저렴하게 구현하거나, 투자 유치를 위한 설득력 있는 콘셉트 비주얼을 제작하는 데 강력한 무기가 될 수 있다.1
- **교육 및 정보 전달:** 교육 분야에서는 복잡한 과학적 원리, 추상적인 수학적 개념, 혹은 역사적 사건을 시각적으로 구현하여 학생들의 이해도를 높이는 맞춤형 교육 영상을 손쉽게 제작할 수 있다.1 또한, 정보가 자주 변경되는 분야(예: 법률, 의료)에서는 새로운 내용이 나올 때마다 신속하게 교육 비디오를 업데이트하여 항상 최신 정보를 제공하는 것이 용이해진다.1
- **게임 개발:** 게임 개발 과정에서 캐릭터의 움직임, 배경 환경, 특수 효과 등 다양한 시각적 에셋의 프로토타입을 빠르게 제작하는 데 활용될 수 있다.2 이를 통해 개발자들은 실제 코딩에 들어가기 전에 게임의 전체적인 느낌과 플레이 경험을 미리 테스트하고, 창의적인 아이디어를 신속하게 반복 수정하며 개발 주기를 단축할 수 있다.

이러한 산업별 응용 사례들은 Text-to-Video 기술이 단순히 특정 작업을 효율화하는 것을 넘어, 콘텐츠 제작 산업의 전통적인 가치 사슬(기획-촬영-편집-배포) 자체를 근본적으로 재편하고 있음을 보여준다. 전통적인 가치 사슬은 각 단계가 명확히 구분되고 고도로 전문화된 인력을 필요로 했다.1 하지만 Text-to-Video 기술은 '기획'(프롬프트 작성) 단계 직후에 최종 결과물에 가까운 콘텐츠를 생성함으로써 중간 단계를 압축하거나 통합한다.1 특히 Runway Aleph와 같은 모델은 '편집'과 '생성'을 결합하여, 촬영된 소스를 기반으로 새로운 앵글이나 객체를 '생성'하는 등 후반 작업의 개념을 확장시킨다.24 이는 고가의 촬영 장비, 로케이션 헌팅, 배우 섭외, 대규모 편집 스튜디오의 필요성을 점차 감소시킬 것이다.1 결과적으로, 기획 능력과 AI 도구 활용 능력을 겸비한 '프롬프트 엔지니어'나 'AI 비주얼 디렉터'와 같은 새로운 직무가 부상하고, 구독 기반의 AI 스튜디오 서비스가 전통적인 프로덕션 하우스를 대체하는 등 산업 구조의 지각 변동이 불가피할 것으로 전망된다.


Text-to-Video 기술은 경이로운 발전을 이루었지만, 인간과 같은 수준의 시각적 이해와 추론 능력을 갖추기까지는 여전히 많은 기술적 난제를 안고 있다. 특히 물리 세계에 대한 이해 부족, 복잡한 상호작용의 표현, 그리고 막대한 계산 비용은 이 기술의 보편적 확산을 가로막는 주요 장애물이다.


현재 최신 모델들의 가장 명백한 한계 중 하나는 기본적인 물리 법칙을 일관되게 준수하지 못한다는 점이다.40 생성된 비디오에서는 객체가 비현실적으로 충돌하거나, 중력을 거스르거나, 에너지 보존 법칙을 위반하는 등 물리적으로 불가능한 현상이 빈번하게 관찰된다.58 이는 모델이 세상이 '어떻게 보이는지'에 대한 시각적 패턴은 학습했지만, '어떻게 작동하는지'에 대한 근본적인 원리는 이해하지 못하고 있음을 시사한다.

이 문제를 정량적으로 평가하기 위해 **T2VPhysBench**라는 벤치마크가 제안되었다. 이 벤치마크는 뉴턴 역학(관성, 가속도 등), 보존 원리(에너지, 운동량 등), 그리고 현상학적 원리(반사, 굴절 등)를 포함한 12가지 핵심 물리 법칙에 대한 모델들의 준수도를 인간 평가자가 채점하는 방식으로 진행된다.59

벤치마크 결과는 충격적이다. Sora를 포함한 모든 최첨단 상용 및 오픈소스 모델들이 1.0점 만점 기준으로 모든 법칙 범주에서 평균 0.60점 미만을 기록하며, 물리적 현실성을 구현하는 데 명백한 한계를 드러냈다.58 특히, 여러 상태에 걸쳐 일관성이 요구되는 '보존 원리' 항목에서 가장 낮은 점수를 기록했는데, 이는 모델들이 단기적인 시각적 그럴듯함은 모방할 수 있지만, 장기적인 인과 관계나 상태 유지를 추론하는 데는 매우 취약함을 보여준다.

| 모델 (Model) | 뉴턴 원리 (Newton) | 보존 원리 (Conservation) | 현상 원리 (Phenomenon) | 평균 점수 (Avg.) |
| ------------ | ------------------ | ------------------------ | ---------------------- | ---------------- |
| Sora         | 0.31               | 0.15                     | 0.38                   | 0.28             |
| Kling        | 0.52               | 0.17                     | 0.38                   | 0.35             |
| Pika 2.2     | 0.38               | 0.19                     | 0.40                   | 0.32             |
| Wan 2.1      | 0.56               | 0.29                     | 0.42                   | 0.42             |

이러한 기술적 한계는 근본적으로 '지식의 병목' 현상에 기인한다. 현재의 모델들은 방대한 픽셀 데이터로부터 시각적 패턴의 '상관관계'는 효과적으로 학습하지만, 그 이면에 있는 물리적, 논리적 '인과관계'나 추상적 개념 자체는 학습하지 못한다. T2VPhysBench의 결과는 모델이 '공이 아래로 떨어진다'는 시각적 패턴은 수없이 보았지만, '중력'이라는 보편적 원리는 이해하지 못한다는 것을 명확히 보여준다.58 현재의 학습 방식은 텍스트 캡션과 비디오 픽셀 간의 표면적인 매핑을 학습하는 것이며 11, 이 데이터에는 물리 법칙이나 논리적 전제조건이 명시적으로 포함되어 있지 않다. 따라서 모델은 '지식' 자체를 배우는 것이 아니라, 지식이 발현된 결과(픽셀)의 통계적 분포만을 모방하게 된다. 이 본질적인 한계를 극복하기 위해서는 단순히 더 많은 비디오 데이터를 사용하는 것을 넘어, 기호적 추론(Symbolic Reasoning)이나 물리 엔진과의 결합 등 새로운 연구 패러다임의 도입이 필요할 것이다.


물리 법칙의 이해 부족과 더불어, 여러 객체나 인물 간의 복잡한 상호작용을 정확하게 묘사하는 것 역시 큰 도전 과제이다.5 예를 들어, 여러 사람이 대화하며 미묘한 표정 변화를 주고받거나, 군중 속에서 각 개인이 독립적으로 움직이는 장면을 자연스럽게 생성하는 것은 여전히 어렵다.

또한, 비디오의 길이가 길어질수록 캐릭터의 외모, 의상, 배경의 스타일 등 시각적 요소의 일관성을 유지하기가 매우 어렵다.2 시간이 흐름에 따라 논리적 오류가 누적되어 이야기가 비현실적으로 전개되거나, 객체가 갑자기 나타나거나 사라지는 등의 문제가 발생할 수 있다. 이는 모델이 장기적인 시간적 맥락을 완벽하게 기억하고 추론하는 데 한계가 있음을 보여준다.


고품질 Text-to-Video 모델을 훈련하고 추론하는 데에는 막대한 양의 계산 자원이 필요하다.2 수십억 개에 달하는 파라미터를 가진 거대 모델을 학습시키기 위해서는 수천 개의 고성능 GPU가 수 주에서 수 개월 동안 필요하며, 이는 소수의 빅테크 기업을 제외하고는 감당하기 어려운 수준이다. 이러한 높은 계산 비용은 기술 접근성을 제한하고 연구의 다양성을 저해하는 주요 요인으로 작용한다.

데이터 역시 중요한 문제이다. 대규모의 고품질 텍스트-비디오 쌍 데이터셋을 구축하는 것은 매우 어렵고 비용이 많이 든다.2 인터넷에서 수집된 비디오 데이터는 저작권 문제가 있거나, 편향된 내용을 포함하거나, 품질이 낮은 경우가 많다. 학습 데이터셋에 내재된 편향은 생성되는 비디오 결과물에 그대로 반영되어 사회적 편견을 재생산하거나 강화할 위험이 있다.


Text-to-Video 기술의 발전은 창의성의 지평을 넓히는 동시에, 오남용 시 사회에 심각한 해악을 끼칠 수 있는 윤리적 딜레마를 동반한다. 특히 딥페이크를 이용한 허위 정보 유포와 저작권 문제는 시급히 해결해야 할 사회적 과제로 부상하고 있다.


이 기술의 가장 큰 위협은 매우 사실적인 가짜 비디오, 즉 딥페이크(Deepfake)를 누구나 손쉽게 제작할 수 있게 되었다는 점이다. 이는 정치적 목적으로 특정 후보가 하지 않은 말을 하는 영상을 만들어 선거에 영향을 미치거나, 유명인의 얼굴을 합성한 가짜 뉴스를 퍼뜨려 사회적 혼란을 야기하거나, 개인의 명예를 훼손하는 데 악용될 수 있다.2 이러한 허위 정보는 진실과 거짓의 경계를 모호하게 만들어, 미디어와 정보 전반에 대한 사회적 신뢰를 근본적으로 침식시킬 수 있다.63


Text-to-Video 모델은 방대한 양의 기존 비디오 데이터를 학습하여 작동한다. 이 과정에서 저작권으로 보호되는 영화, 방송 프로그램이나 개인의 동의 없이 촬영된 영상이 학습 데이터에 포함될 경우 심각한 법적 분쟁을 야기할 수 있다.2 또한, 생성된 비디오의 저작권을 누가 소유하는가(프롬프트를 입력한 사용자, AI 모델 개발사, 아니면 학습 데이터의 원 저작자)에 대한 법적 기준이 아직 명확하지 않아 새로운 법적 과제를 제기하고 있다. 특정인의 외모나 목소리를 무단으로 복제하여 상업적으로 이용하는 초상권 침해 문제 역시 심각한 우려를 낳고 있다.


이러한 윤리적 문제에 대응하기 위해 기술적, 정책적 차원의 노력이 병행되고 있다.

- **C2PA (Coalition for Content Provenance and Authenticity):** C2PA는 콘텐츠의 출처와 생성 및 수정 이력을 암호화하여 기록하는 개방형 기술 표준이다. AI로 생성된 비디오에 C2PA 메타데이터를 포함시키면, 소비자는 해당 콘텐츠가 인공적으로 생성되었음을 투명하게 확인할 수 있다. OpenAI는 Sora로 생성된 모든 비디오에 C2PA 메타데이터를 포함할 것이라고 밝혔다.46
- **워터마킹(Watermarking):** 사람의 눈에 보이거나 보이지 않는 특정 표식(워터마크)을 비디오에 삽입하여 AI 생성물임을 식별하는 기술이다. 이는 콘텐츠의 출처를 추적하고 진위를 판별하는 데 도움을 줄 수 있다.46
- **법률 제정:** 각국 정부는 AI 생성 콘텐츠의 오남용을 막기 위한 규제 프레임워크를 마련하고 있다. 유럽연합(EU)의 'AI Act'는 AI 시스템을 위험 등급별로 분류하여 규제하며, 딥페이크와 같은 고위험 애플리케이션에 대해서는 투명성 의무를 부과한다. 중국 역시 '딥 합성 규제'를 통해 AI 생성 콘텐츠에 명확한 라벨링을 의무화하고 있다.63

그러나 이러한 기술적, 정책적 대응은 '탐지'와 '생성' 간의 끊임없는 군비 경쟁(Arms Race)에 직면해 있다. 악의적인 딥페이크가 등장하면 이를 탐지하는 AI 모델이 개발되지만, 생성 모델(특히 GAN)은 다시 그 탐지 모델을 속이도록 학습하며 더욱 정교해진다.62 C2PA 메타데이터나 워터마크 역시 의도적으로 제거되거나 훼손될 수 있다.46 이는 순수한 기술만으로는 윤리적 문제를 완전히 해결할 수 없음을 시사한다. 따라서 기술적 방어 체계 구축과 더불어, 콘텐츠의 출처를 비판적으로 확인하는 대중의 미디어 리터러시(media literacy) 교육을 강화하고, 악의적 콘텐츠 유포에 대한 강력한 법적 처벌 기준을 마련하며, 플랫폼 기업의 사회적 책임을 강화하는 등 다각적인 사회적 접근이 필수적으로 요구된다.


Text-to-Video 기술은 텍스트-이미지 모델의 성공을 발판 삼아 불과 몇 년 만에 비약적인 발전을 이루었다. 초기 모델들이 짧고 조악한 클립을 생성하는 데 그쳤다면, OpenAI의 Sora와 같은 최신 모델들은 1분 이상의 고해상도 비디오를 놀라운 사실성과 일관성으로 생성해내며 단순한 콘텐츠 제작 도구를 넘어 '세계 시뮬레이션'의 가능성을 열었다.

Sora의 기술 보고서가 명시하듯, 이 기술의 궁극적인 목표는 물리적 세계와의 상호작용이 필요한 현실 세계의 문제들을 해결하는 데 도움을 주는 모델을 훈련시키는 것이다.5 이는 비디오 생성이 단순히 시각적 결과물을 만드는 행위를 넘어, 세상의 동적인 인과관계를 이해하고 예측하는 과정임을 의미한다. 이러한 관점에서 Text-to-Video 기술은 아직 초기 단계에 불과한 범용 인공지능(AGI)으로 나아가는 중요한 이정표가 될 잠재력을 품고 있다.43

그러나 이 막대한 잠재력 이면에는 우리가 반드시 넘어야 할 기술적, 윤리적 과제들이 산적해 있다. 현재 모델들은 여전히 물리적 현실을 완벽히 모사하지 못하며, 복잡한 상호작용과 장기적 일관성을 구현하는 데 한계를 보인다. 막대한 계산 비용은 기술의 접근성을 제한하고, 딥페이크와 저작권 문제는 사회적 신뢰와 법적 질서를 위협한다.

따라서 우리의 과제는 명확하다. 기술 혁신을 지속적으로 장려하여 그 잠재력을 최대한 발현시키는 동시에, 기술의 오남용을 방지하고 사회적 부작용을 최소화하기 위한 견고한 안전장치를 마련해야 한다. 이를 위해서는 C2PA와 같은 기술적 표준의 확립, AI 생성 콘텐츠에 대한 명확한 법적 규제, 그리고 무엇보다 기술의 영향에 대한 깊이 있는 사회적 논의와 합의 형성이 필수적이다. Text-to-Video 기술이 열어갈 미래는 우리가 지금 어떤 선택을 하고 어떤 규범을 만들어 가느냐에 달려 있다. 기술 발전과 사회적 책임의 균형을 맞추기 위한 범사회적인 노력이 그 어느 때보다 시급한 시점이다.


1. Text-to-Video AI: The Future of Visual Storytelling - ProfileTree, 8월 24, 2025에 액세스, https://profiletree.com/text-to-video-ai-the-future-of-visual-storytelling/
2. Text-to-video model - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Text-to-video_model
3. Evaluating and Fine-tuning Text to Video Model - Labellerr, 8월 24, 2025에 액세스, https://www.labellerr.com/blog/evaluating-and-finetuning-text-to-video-model/
4. Understanding Text-to-Video AI Generators Tools - Rubick.ai, 8월 24, 2025에 액세스, https://rubick.ai/blog/product-information-management/understanding-text-to-video-ai-generators-tools/
5. Sora: Creating video from text - OpenAI, 8월 24, 2025에 액세스, https://openai.com/index/sora/
6. A Brief History of Generative AI - DATAVERSITY, 8월 24, 2025에 액세스, https://www.dataversity.net/a-brief-history-of-generative-ai/
7. The rise of generative AI: A timeline of breakthrough innovations - Qualcomm, 8월 24, 2025에 액세스, https://www.qualcomm.com/news/onq/2024/02/the-rise-of-generative-ai-timeline-of-breakthrough-innovations
8. The Evolution of Text to Video Models | Towards Data Science, 8월 24, 2025에 액세스, https://towardsdatascience.com/the-evolution-of-text-to-video-models-1577878043bd/
9. History of generative AI - Toloka, 8월 24, 2025에 액세스, https://toloka.ai/blog/history-of-generative-ai/
10. Text-to-image model - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Text-to-image_model
11. [2209.14792] Make-A-Video: Text-to-Video Generation without Text-Video Data - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2209.14792
12. AI Timeline - A history of image and video generative models ..., 8월 24, 2025에 액세스, https://www.fabianmosele.com/ai-timeline
13. Video Generation Models Explosion 2024 - Yen-Chen Lin, 8월 24, 2025에 액세스, https://yenchenlin.me/blog/2025/01/08/video-generation-models-explosion-2024/
14. Phenaki, 8월 24, 2025에 액세스, https://phenaki.github.io/
15. Phenaki: Generate Videos from Text - Google Research, 8월 24, 2025에 액세스, https://sites.research.google/phenaki/
16. A Comprehensive Guide to Text-to-Video AI: The Next Revolution in Content Creation, 8월 24, 2025에 액세스, https://sarunnotes.com/a-comprehensive-guide-to-text-to-video-ai-the-next-revolution-in-content-creation-bb44558f51c4
17. A Brief Look at Text To Video AI: A New Dawn Of Creation - AIRI, 8월 24, 2025에 액세스, https://airi.com.au/f/a-brief-look-at-text-to-video-ai-a-new-dawn-of-creation
18. The Rapid Evolution of Generative AI Video footage - VC Cafe, 8월 24, 2025에 액세스, https://www.vccafe.com/2024/01/17/the-rapid-evolution-of-generative-ai-video-footage/
19. Google Research: Lumiere - Simon Willison's Weblog, 8월 24, 2025에 액세스, https://simonwillison.net/2024/Jan/24/google-research-lumiere/
20. Timeline of AI and language models – Dr Alan D. Thompson - LifeArchitect.ai, 8월 24, 2025에 액세스, https://lifearchitect.ai/timeline/
21. Google I/O 2024: Introducing Veo and Imagen 3 generative AI tools - The Keyword, 8월 24, 2025에 액세스, https://blog.google/technology/ai/google-generative-ai-veo-imagen-3/
22. Runway | Tools for human imagination., 8월 24, 2025에 액세스, https://runwayml.com/
23. Introducing Runway Aleph - Runway Research, 8월 24, 2025에 액세스, https://runwayml.com/research/introducing-runway-aleph
24. Runway Aleph Video Model - Blockchain Council, 8월 24, 2025에 액세스, https://www.blockchain-council.org/ai/runway-aleph-video-model/
25. Understanding Denoising Diffusion Probabilistic Models (DDPMs): Part 2 of Generative AI with… - Medium, 8월 24, 2025에 액세스, https://medium.com/@ykarray29/understanding-denoising-diffusion-probabilistic-models-ddpms-part-2-of-generative-ai-with-b354a041bf4e
26. An In-Depth Guide to Denoising Diffusion Probabilistic Models DDPM – Theory to Implementation - LearnOpenCV, 8월 24, 2025에 액세스, https://learnopencv.com/denoising-diffusion-probabilistic-models/
27. Denoising Diffusion Probabilistic Models (DDPM) - labml.ai, 8월 24, 2025에 액세스, https://nn.labml.ai/diffusion/ddpm/index.html
28. Denoising Diffusion Probabilistic Models, 8월 24, 2025에 액세스, https://proceedings.neurips.cc/paper/2020/file/4c5bcfec8584af0d967f1ab10179ca4b-Paper.pdf
29. Lecture Notes in Probabilistic Diffusion Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2312.10393v1
30. Efficient Quantization Strategies for Latent Diffusion Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2312.05431v1
31. Boosting Latent Diffusion with Perceptual Objectives - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2411.04873v1
32. Diffusion Loss: Every Step Explained | Towards Data Science, 8월 24, 2025에 액세스, https://towardsdatascience.com/diffusion-loss-every-step-explained-8c19c5e1349b/
33. LLM Transformer Model Visually Explained - Polo Club of Data Science, 8월 24, 2025에 액세스, https://poloclub.github.io/transformer-explainer/
34. How to Implement Scaled Dot-Product Attention from Scratch in ..., 8월 24, 2025에 액세스, https://machinelearningmastery.com/how-to-implement-scaled-dot-product-attention-from-scratch-in-tensorflow-and-keras/
35. Understanding Scaled Dot-Product Attention in Transformer Models - Medium, 8월 24, 2025에 액세스, https://medium.com/@saraswatp/understanding-scaled-dot-product-attention-in-transformer-models-5fe02b0f150c
36. Understanding and Coding the Self-Attention Mechanism of Large Language Models From Scratch - Sebastian Raschka, 8월 24, 2025에 액세스, https://sebastianraschka.com/blog/2023/self-attention-from-scratch.html
37. In Depth Understanding of Attention Mechanism (Part II) - Scaled Dot-Product Attention and Example | by FunCry | Medium, 8월 24, 2025에 액세스, https://medium.com/@funcry/in-depth-understanding-of-attention-mechanism-part-ii-scaled-dot-product-attention-and-its-7743804e610e
38. Video generation models as world simulators | OpenAI, 8월 24, 2025에 액세스, https://openai.com/index/video-generation-models-as-world-simulators/
39. Sora: A Review on Background, Technology, Limitations, and Opportunities of Large Vision Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2402.17177v2
40. Step-Video-T2V Technical Report: The Practice, Challenges, and Future of Video Foundation Model - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2502.10248v1
41. Temporally Consistent Transformers for Video Generation - Danijar Hafner, 8월 24, 2025에 액세스, https://danijar.com/project/teco/
42. ASurvey: Spatiotemporal Consistency in Video Generation, 8월 24, 2025에 액세스, https://arxiv.org/abs/2502.17863
43. Sora as an AGI World Model? A Complete Survey on Text-to-Video Generation - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2403.05131
44. OpenAI Sora's Technical Review - Jianing Qi, 8월 24, 2025에 액세스, https://j-qi.medium.com/openai-soras-technical-review-a8f85b44cb7f
45. A Comparative Analysis of Runway and Sora: The New Frontier of AI ..., 8월 24, 2025에 액세스, https://skywork.ai/skypage/en/A-Comparative-Analysis-of-Runway-and-Sora:-The-New-Frontier-of-AI-Video-Generation/1948241392539652096
46. Sora is here | OpenAI, 8월 24, 2025에 액세스, https://openai.com/index/sora-is-here/
47. Google's Lumiere has created a new paradigm for text-to-video AI ..., 8월 24, 2025에 액세스, https://www.freethink.com/robots-ai/googles-lumiere-has-created-a-new-paradigm-for-text-to-video-ai
48. Lumiere: A Space-Time Diffusion Model for Video Generation - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2401.12945v1
49. Lumiere: Video Generation AI from Google Research - The Prompt Engineering Institute, 8월 24, 2025에 액세스, https://promptengineering.org/lumiere-video-generation-ai-from-google-research/
50. Paper Review: Lumiere: A Space-Time Diffusion Model for Video ..., 8월 24, 2025에 액세스, https://blog.gopenai.com/paper-review-lumiere-a-space-time-diffusion-model-for-video-generation-9b83076b03c7
51. Lumiere: A Space-Time Diffusion Model for Video Generation - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=qtm_j4TgDxU
52. [2401.12945] Lumiere: A Space-Time Diffusion Model for Video Generation - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2401.12945
53. Research | Runway, 8월 24, 2025에 액세스, https://runwayml.com/research
54. Runway Gen-2: AI Magic Tools for Effortless Content Creation | Deepgram, 8월 24, 2025에 액세스, https://deepgram.com/ai-apps/runway-gen-2
55. A Complete Guide to Runway - Learn Prompting, 8월 24, 2025에 액세스, https://learnprompting.org/blog/guide-runwayml
56. Aleph - How to Transform Videos - Runway Academy, 8월 24, 2025에 액세스, https://academy.runwayml.com/aleph/how-to-transform-videos
57. What is Text-to-Video? - Hugging Face, 8월 24, 2025에 액세스, https://huggingface.co/tasks/text-to-video
58. T2VPhysBench: A First‑Principles Benchmark for Physical Consistency in Text‑to‑Video Generation - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2505.00337v1
59. T2VPhysBench: A First-Principles Benchmark for Physical ..., 8월 24, 2025에 액세스, https://www.arxiv.org/abs/2505.00337
60. [Literature Review] T2VPhysBench: A First-Principles Benchmark for Physical Consistency in Text-to-Video Generation - Moonlight, 8월 24, 2025에 액세스, https://www.themoonlight.io/en/review/t2vphysbench-a-first-principles-benchmark-for-physical-consistency-in-text-to-video-generation
61. Limitations of text-to-video diffusion models for generating consistent... - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/Limitations-of-text-to-video-diffusion-models-for-generating-consistent-videos-Top_fig4_385977708
62. Deepfakes and the Ethics of Generative AI - Tepperspectives, 8월 24, 2025에 액세스, https://tepperspectives.cmu.edu/all-articles/deepfakes-and-the-ethics-of-generative-ai/
63. Deepfakes and the Future of AI Legislation: Overcoming the Ethical ..., 8월 24, 2025에 액세스, https://gdprlocal.com/deepfakes-and-the-future-of-ai-legislation-overcoming-the-ethical-and-legal-challenges/
64. From Video Generation to World Model | CVPR 2025, 8월 24, 2025에 액세스, https://world-model-tutorial.github.io/


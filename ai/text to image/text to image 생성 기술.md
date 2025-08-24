# Text-to-Image 생성 기술
[Text-to-Image 생성 기술](./index.md)



텍스트-이미지(Text-to-Image, T2I) 모델은 자연어 프롬프트(prompt)를 입력받아 해당 설명과 일치하는 이미지를 생성하는 기계 학습 기술이다.1 이 기술의 근본적인 목표는 인간의 언어와 시각적 이미지 사이의 개념적 간극을 메우는 것으로, 상상 속의 개념을 구체적인 시각적 현실로 변환할 잠재력을 지닌다.2 T2I 기술의 개발은 2010년대 중반, 딥러닝 신경망의 발전과 함께 본격적으로 시작되었다.1 2020년대에 들어서면서 OpenAI의 DALL-E, Midjourney, Stability AI의 Stable Diffusion과 같은 고성능 모델들이 대중에게 공개되면서 인공지능이 생성한 예술(AI-generated art)의 패러다임을 근본적으로 전환시켰다.4

이러한 발전은 창의성의 문턱을 극적으로 낮추는 결과를 낳았다. 과거에는 예술적 표현을 위해 붓이나 물감과 같은 도구에 대한 숙련도, 혹은 포토샵과 같은 디지털 소프트웨어에 대한 기술적 이해가 필수적이었다. 그러나 T2I 기술의 시대에는 아이디어를 정교하고 효과적인 언어로 표현하는 능력, 즉 '프롬프트 엔지니어링'이 핵심 역량으로 부상했다. 이는 기술적 장벽을 허물어 누구나 생생한 상상력을 시각적으로 구현할 수 있게 함으로써 '창작자'의 정의를 확장시킨다.5 동시에, 창의성의 본질을 기술적 실행 능력보다는 아이디어 발상 그 자체로 회귀시키면서도, 언어적 표현력이라는 새로운 형태의 기술을 요구하는 이중적인 효과를 창출하고 있다.


생성형 AI는 기존의 분석형 AI와 구별되는 본질적인 특징을 가진다. 이는 단순히 데이터를 분류하거나 예측하는 것을 넘어, 시각, 오디오, 텍스트 등 완전히 새로운 콘텐츠를 생성하는 능력에서 비롯된다.6 T2I 기술은 이러한 생성형 AI의 핵심 분야로서, 예술, 디자인, 엔터테인먼트, 마케팅 등 시각 미디어를 활용하는 모든 산업의 미래를 재편하는 동력으로 작용하고 있다.2

학술적 관점에서 T2I 기술은 자연어 처리(Natural Language Processing, NLP)와 컴퓨터 비전(Computer Vision, CV)이라는 두 거대 분야의 성공적인 융합을 상징한다.8 더 나아가, 언어적 추상 개념을 시각적 구체물로 변환하는 능력은 인간과 유사한 수준의 이해와 생성을 목표로 하는 범용 인공지능(Artificial General Intelligence, AGI)을 향한 중요한 기술적 이정표로 평가받는다.10 초기 T2I 모델이 프롬프트 입력에 따라 이미지를 출력하는 단방향 '생성'에 집중했다면, DALL-E 2의 인페인팅(inpainting) 및 아웃페인팅(outpainting) 기능이나 Midjourney의 영역별 수정(Vary Region) 기능은 인간과 AI가 상호작용하며 결과물을 함께 다듬어가는 협력적 모델로의 진화를 명확히 보여준다.11 이러한 순환적(iterative) 프로세스는 AI의 역할을 단순한 '결과물 생성기'에서 창작 과정에 동참하는 '창의적 파트너'로 격상시키는 중요한 변화이다.



딥러닝 기술이 부상하기 이전, 텍스트로부터 이미지를 생성하려는 시도는 기존에 존재하는 클립 아트(clip art) 데이터베이스에서 구성 요소를 찾아 배열하는 콜라주(collage) 방식에 머물렀다.3 현대적인 의미의 첫 T2I 모델은 2015년 토론토 대학 연구팀이 발표한 'alignDRAW'로 간주된다. 이 모델은 순환 변분 오토인코더(recurrent Variational Autoencoder, VAE)와 어텐션 메커니즘을 결합하여 텍스트 시퀀스를 조건으로 이미지를 생성하는 구조를 제시했다.3

본격적인 발전은 2014년 이안 굿펠로우(Ian Goodfellow)에 의해 제안된 생성적 적대 신경망(Generative Adversarial Network, GAN)의 등장과 함께 시작되었다.11 GAN은 실제와 유사한 이미지를 만들려는 '생성자(Generator)'와 생성된 이미지를 실제 이미지와 구별하려는 '판별자(Discriminator)'가 서로 경쟁하며 학습하는 독창적인 구조를 가진다.2 T2I 작업에 GAN이 처음 적용되면서, 텍스트 설명을 바탕으로 시각적으로 그럴듯한 이미지를 생성하는 것이 가능해졌다.3

이후 GAN 기반 모델들은 점차 정교해졌다. 2016년에 등장한 **StackGAN**은 2단계 접근법을 도입했다. 첫 번째 단계에서 텍스트 설명을 바탕으로 전반적인 형태와 색상을 담은 저해상도 이미지를 생성한 후, 두 번째 단계에서 이 이미지를 입력받아 세부 묘사를 추가하며 고해상도 이미지로 개선하는 방식이다. 이는 이미지 품질을 점진적으로 향상시킨다는 개념을 처음으로 도입한 중요한 시도였다.2 뒤이어 발표된 **AttnGAN**은 어텐션 메커니즘(attention mechanism)을 적용하여 한 단계 더 나아갔다. 이 모델은 텍스트 설명 내의 특정 단어(예: '노란 부리')에 집중하여 이미지의 해당 부분(부리)을 생성함으로써, 텍스트와 이미지 간의 세밀한 의미적 정렬(fine-grained alignment)을 구현했다. 그 결과, 이전 모델들보다 의미론적으로 훨씬 더 일관되고 정확한 이미지 생성이 가능해졌다.2

그러나 GAN 기반 모델들은 고질적인 한계를 안고 있었다. 두 신경망 간의 균형을 맞추는 것이 어려워 학습 과정이 매우 불안정했으며(unstable training), 생성자가 판별자를 속이기 쉬운 소수의 결과물만 반복적으로 생성하는 '모드 붕괴(mode collapse)' 현상이 자주 발생했다. 이로 인해 생성된 이미지는 종종 흐릿하거나 어색한 부분을 포함하는 경우가 많았다.2


2021년 1월, OpenAI는 트랜스포머(Transformer) 아키텍처에 기반한 **DALL-E**를 발표하며 T2I 분야에 새로운 가능성을 제시했다.3 DALL-E는 GPT-3와 동일한 자기회귀(autoregressive) 방식을 사용하여, 텍스트와 이미지 데이터를 하나의 연속된 토큰(token) 시퀀스로 변환한 뒤, 이전 토큰들을 바탕으로 다음 토큰을 순차적으로 예측하는 방식으로 이미지를 생성했다.4 120억 개의 파라미터를 사용한 이 거대 모델은 "아보카도 모양의 안락의자"와 같은 초현실적인 개념도 시각화해내며 대중의 폭발적인 관심을 끌었다.3 하지만 자기회귀 모델은 토큰을 하나씩 순차적으로 생성해야 하므로, 추론 과정에서 반복적인 연산이 많이 필요해 상당한 계산 자원을 소모한다는 단점이 있었다.20


2020년대 중반에 이르러, 확산 모델(Diffusion Model)이 등장하며 T2I 분야는 또 한 번의 패러다임 전환을 맞이했다. 확산 모델은 GAN과 VAE를 제치고 이미지 및 영상 생성 분야에서 지배적인 아키텍처로 빠르게 자리 잡았다.22

이러한 변화의 시작점에 있었던 모델이 OpenAI의 **GLIDE**(2021)이다. GLIDE는 확산 모델을 T2I 작업에 본격적으로 적용한 선구적인 연구로, 특히 CLIP 모델을 이용한 가이던스(guidance)와 분류자-자유 가이던스(classifier-free guidance)라는 두 가지 전략을 비교하며 후자의 우수성을 입증했다. 이 가이던스 방식은 이후 등장하는 고성능 확산 모델들의 핵심 기술로 자리 잡았으며, DALL-E 2의 디코더 모델에도 직접적인 영향을 미쳤다.8

이후 T2I 기술은 폭발적인 발전을 거듭했다.

- **DALL-E 2** (2022년 4월): OpenAI는 GLIDE의 성공을 바탕으로, 훨씬 더 복잡하고 사실적인 이미지 생성이 가능한 DALL-E 2를 공개했다. 이 모델은 확산 모델을 기반으로 하되, CLIP 임베딩 공간을 적극적으로 활용하는 구조를 채택하여 이미지의 품질과 프롬프트 충실도를 극대화했다.3
- **Imagen** (2022년 4월): 구글 브레인 팀은 Imagen을 통해 다른 접근법을 제시했다. Imagen은 텍스트의 의미를 깊이 이해하기 위해 사전 훈련된 대규모 언어 모델(LLM, 예: T5)을 텍스트 인코더로 사용하고, 이미지 생성은 확산 모델에 맡겼다. 이 연구를 통해, 이미지 생성 모델의 크기를 키우는 것보다 텍스트를 이해하는 언어 모델의 크기를 키우는 것이 최종 결과물의 품질과 텍스트-이미지 정렬에 더 효과적이라는 중요한 사실이 밝혀졌다.2
- **Stable Diffusion** (2022년 8월): Stability AI가 공개한 Stable Diffusion은 T2I 기술의 대중화를 이끈 결정적인 모델이다. 이 모델은 잠재 확산 모델(Latent Diffusion Model, LDM) 아키텍처를 기반으로 하여, 고차원의 픽셀 공간이 아닌 저차원의 잠재 공간에서 확산 과정을 수행함으로써 계산 효율성을 획기적으로 개선했다.3 무엇보다 Stable Diffusion은 모델과 코드를 오픈 소스로 공개했는데, 이는 전 세계 연구자들과 개발자들의 자유로운 실험과 개선을 촉진하는 기폭제가 되었다. 이 개방성 덕분에 ControlNet, LoRA 등 수많은 파생 기술과 응용 프로그램이 탄생하며 T2I 생태계는 폭발적으로 성장할 수 있었다. 폐쇄적인 모델 생태계에서는 불가능했을 속도와 다양성의 혁신이 오픈 소스를 통해 이루어진 것이다.6

이러한 기술 발전의 궤적을 살펴보면, 단순히 더 사실적인 이미지를 생성하는 '성능' 중심의 경쟁을 넘어, 사용자가 원하는 결과물을 얼마나 정교하게 제어할 수 있는가 하는 '제어 가능성'으로 기술의 초점이 이동하고 있음을 알 수 있다. 초기 GAN이 '그럴듯한' 이미지를 만드는 데 집중했다면, AttnGAN은 어텐션을 통해 텍스트의 특정 부분에 해당하는 이미지를 생성하며 제어의 가능성을 열었다. 확산 모델은 가이던스 개념을 통해 이 제어력을 더욱 강화했으며, 이후 ControlNet과 같은 기술은 포즈, 깊이, 윤곽선 등 구체적인 공간적 제어를 가능하게 했다. 이는 T2I 기술이 단순한 이미지 생성 도구를 넘어, 정교한 디자인 및 엔지니어링 도구로 발전할 잠재력을 보여주는 중요한 흐름이다.



확산 확률 모델(Denoising Diffusion Probabilistic Models, DDPM)은 자연계에서 관찰되는 비평형 열역학 과정에서 영감을 얻었다.8 가장 흔한 비유는 맑은 물에 잉크 한 방울이 떨어져 점차 퍼져나가 전체가 균일하게 섞이는 확산(diffusion) 현상이다.9 확산 모델은 이 과정을 역으로 시뮬레이션한다. 즉, 완전히 무질서한 상태(순수 노이즈)에서 시작하여 점진적으로 질서를 부여함으로써(노이즈 제거) 의미 있는 데이터(이미지)를 생성한다. 이 모델은 비지도 학습(unsupervised learning) 방식으로 데이터의 잠재적 분포를 학습하며, 두 가지 핵심 과정으로 구성된다: 원본 이미지에 점진적으로 노이즈를 추가하는 **순방향 확산 과정(Forward Process)**과, 순수 노이즈에서 시작하여 점진적으로 노이즈를 제거해 원본 이미지를 복원하는 **역방향 제거 과정(Reverse Process)**이다.15 이러한 접근법은 분자 시스템의 움직임을 모델링하는 랑주뱅 동역학(Langevin Dynamics)과 같은 물리학적 원리에도 수학적 뿌리를 두고 있다.28


순방향 과정의 목표는 원본 데이터 $x_0$를 $T$ 단계에 걸쳐 점진적으로 손상시켜 최종적으로 순수한 가우시안 노이즈 $x_T$로 변환하는 것이다.22 이 과정은 마르코프 연쇄(Markov chain)로 모델링되며, 각 단계 $t$에서 이전 상태 $x_{t-1}$에 미리 정의된 분산 스케줄 $\beta_t \in (0, 1)$에 따라 가우시안 노이즈를 추가한다.9 시점  $t$에서의 데이터 $x_t$의 조건부 확률 분포는 다음과 같이 정의된다.
$$
q(x_t|x_{t-1}) = \mathcal{N}(x_t ; \sqrt{1-\beta_t}x_{t-1}, \beta_t\textbf{I})
$$
여기서 $\mathcal{N}$은 정규분포를, $\textbf{I}$는 단위 행렬을 의미한다.27 이 정의의 중요한 특징은 '재매개변수화 기법(Reparameterization Trick)'을 통해 임의의 시점 $t$에서의 $x_t$를 원본 데이터 $x_0$로부터 한 번에 샘플링할 수 있다는 점이다. $\alpha_t = 1 - \beta_t$와 $\bar{\alpha}_t = \prod_{i=1}^t \alpha_i$로 정의하면, $x_t$는 다음과 같이 $x_0$와 표준 정규분포에서 추출한 노이즈 $\epsilon$의 함수로 표현될 수 있다.27
$$
x_t = \sqrt{\bar{\alpha}_t}x_0 + \sqrt{1 - \bar{\alpha}_t}\epsilon, \quad \text{where } \epsilon \sim \mathcal{N}(\textbf{0}, \textbf{I})
$$
이 수식 덕분에 학습 과정에서 매번 $t$단계를 순차적으로 계산할 필요 없이, 임의의 $t$와 $x_0$를 선택하여 $x_t$를 직접 생성하고 모델을 훈련시킬 수 있어 학습 효율성이 크게 향상된다.27


역방향 과정은 확산 모델의 핵심이자 실제 이미지가 생성되는 단계이다. 순수한 가우시안 노이즈 $p(x_T) = \mathcal{N}(x_T; \textbf{0}, \textbf{I})$에서 시작하여, 순방향 과정을 거꾸로 거슬러 올라가며 점진적으로 노이즈를 제거하고 최종적으로 원본 데이터 분포에 속하는 새로운 샘플 $x_0$를 생성하는 것을 목표로 한다.22

이 과정 역시 마르코프 연쇄로 모델링되며, 각 단계에서 $x_t$가 주어졌을 때 이전 상태 $x_{t-1}$의 분포 $p_\theta(x_{t-1}|x_t)$를 추정해야 한다. 이 조건부 확률 분포는 일반적으로 평균 $\boldsymbol{\mu}_\theta(x_t, t)$와 분산 $\boldsymbol{\Sigma}_\theta(x_t, t)$를 갖는 정규분포로 가정되며, 이 파라미터들을 예측하기 위해 심층 신경망을 사용한다.27

초기 연구에서는 $\boldsymbol{\mu}_\theta(x_t, t)$를 직접 예측하려 했으나, 후속 연구(DDPM)에서는 모델이 $x_t$에 추가된 노이즈 $\epsilon$을 예측하도록 학습 목표를 단순화했다. 즉, 신경망 $\epsilon_\theta(x_t, t)$는 현재의 노이즈 낀 이미지 $x_t$와 시점 $t$를 입력받아, 이 상태를 만드는 데 사용된 노이즈를 예측한다. 이 예측된 노이즈를 $x_t$에서 제거하는 방식으로 이전 단계의 덜 노이즈 낀 이미지 $x_{t-1}$을 추정할 수 있다. 이 노이즈 예측기로는 주로 U-Net 아키텍처가 사용된다.22


확산 모델의 학습 목표는 원본 데이터의 로그 우도(log-likelihood), 즉 $\log p_\theta(x_0)$를 최대화하는 것이다. 하지만 이 값은 직접 계산하기 매우 어렵기 때문에, 변분 추론(variational inference)을 통해 다루기 쉬운 변분 하한(Variational Lower Bound, VLB)을 대신 최대화하는 방식으로 접근한다.27 이 VLB를 전개하면 최종 목적 함수는 다음과 같이 세 가지 주요 항으로 분해될 수 있다.27
$$
J = \underbrace{\mathbb{E}_{q(x_1|x_0)}\left[\log p_\theta(x_0|x_1)\right]}_{\text{reconstruction term}} - \underbrace{D_{KL}(q(x_T|x_0)\rvert\rvert p(x_T))}_{\text{prior matching term}} - \sum_{t=2}^T \underbrace{\mathbb{E}_{q(x_t|x_0)}}_{\text{denoising matching term}}
$$
각 항은 다음과 같은 의미를 가진다:

- **Reconstruction term:** VAE의 재구성 항과 유사하게, 약간의 노이즈가 추가된 $x_1$에서 원본 $x_0$를 얼마나 잘 복원하는지를 측정한다.27
- **Prior matching term:** 순방향 과정의 최종 결과물 $x_T$의 분포가 우리가 가정한 사전 분포(표준 가우시안 노이즈)와 얼마나 유사한지를 쿨백-라이블러 발산(Kullback–Leibler divergence, $D_{KL}$)으로 측정한다. 이 항에는 학습 가능한 파라미터가 없어 학습 과정에서는 상수로 취급된다.27
- **Denoising matching term:** 학습의 가장 핵심적인 부분이다. 각 노이즈 제거 단계 $t$에서, 모델이 예측하는 역방향 분포 $p_\theta(x_{t-1}|x_t)$가 원본 데이터 $x_0$가 주어졌을 때의 이상적인(ground truth) 역방향 분포 $q(x_{t-1}|x_t, x_0)$와 얼마나 유사한지를 $D_{KL}$로 측정한다.27

이러한 수학적 우아함에도 불구하고, 확산 모델의 성공 비결 중 하나는 이 복잡한 목적 함수가 실제 구현에서는 훨씬 더 단순한 형태로 귀결된다는 점에 있다. 연구자들은 VLB를 최적화하는 것이 결국 각 타임스텝에서 추가된 실제 노이즈 $\epsilon$과 모델이 예측한 노이즈 $\epsilon_\theta(x_t, t)$ 사이의 평균 제곱 오차(Mean Squared Error, MSE)를 최소화하는 것과 거의 동일한 효과를 낸다는 것을 보였다.29
$$
L_{simple} = \mathbb{E}_{t, x_0, \epsilon} \left[ \rvert\rvert \epsilon - \epsilon_\theta(\sqrt{\bar{\alpha}_t}x_0 + \sqrt{1 - \bar{\alpha}_t}\epsilon, t) \rvert\rvert^2 \right]
$$
이처럼 복잡한 확률 분포 매칭 문제를 간단한 회귀(regression) 문제로 변환함으로써, 모델은 안정적인 경사 하강법으로 학습될 수 있다. 이는 GAN이 겪는 두 신경망 간의 내쉬 균형(Nash equilibrium)을 찾는 과정의 불안정성을 근본적으로 회피하게 만드는 핵심 요인이다.


초기 확산 모델은 고해상도 이미지의 픽셀 공간에서 직접 작동했기 때문에 막대한 계산 자원을 필요로 했다. **잠재 확산 모델(LDM)**은 이 문제를 해결하기 위해 VAE와 같은 오토인코더를 사용하여 이미지를 고차원의 픽셀 공간에서 저차원의 **잠재 공간(latent space)**으로 압축한 뒤, 이 잠재 공간 내에서 확산 및 노이즈 제거 과정을 수행하는 혁신적인 접근법을 제시했다.9

이 접근법은 단순히 계산량을 줄이는 것을 넘어선다. VAE는 이미지를 압축하면서 불필요한 세부 정보(redundant details)를 제거하고 이미지의 핵심적인 '개념'과 '구조'와 같은 의미론적 정보(semantic information)를 잠재 벡터에 응축한다.9 따라서 잠재 공간에서 노이즈를 제거하는 과정은 단순히 픽셀 값을 복원하는 것이 아니라, 추상화된 '의미'를 복원하는 과정에 가깝다. 이로 인해 LDM은 더 적은 계산량으로도 의미론적으로 일관되고 구조적으로 안정적인 고품질 이미지를 생성할 수 있게 되었으며, 이는 Stable Diffusion의 성공을 뒷받침하는 핵심 원리가 되었다.26


T2I 모델의 성공은 단일 기술의 혁신이 아닌, 각기 다른 목적을 위해 개발된 고성능 모듈들을 유기적으로 결합하여 시너지를 창출한 시스템 엔지니어링의 결과물이다. 텍스트의 '의미'를 이해하는 CLIP, 이미지의 '공간 구조'를 다루는 U-Net, 그리고 이 둘을 '연결'하는 Cross-Attention 메커니즘이 핵심 구성 요소이다.


CLIP은 T2I 모델에서 텍스트 프롬프트의 의미를 이해하고, 생성된 이미지가 그 의미와 일치하는지 평가하는 '의미론적 나침반'과 같은 역할을 수행한다. 이 모델의 핵심 아이디어는 이미지와 텍스트를 동일한 다차원 임베딩 공간(embedding space)에 매핑하여, 의미적으로 유사한 텍스트와 이미지가 벡터 공간상에서 서로 가깝게 위치하도록 만드는 것이다.32

CLIP은 대조 학습(Contrastive Learning)이라는 기법을 통해 훈련된다. 수억 개의 (이미지, 텍스트 캡션) 쌍으로 구성된 데이터셋을 사용하여, 서로 짝이 맞는 실제 쌍(positive pair)의 임베딩 벡터 간 코사인 유사도(cosine similarity)는 최대화하고, 짝이 맞지 않는 무관한 쌍(negative pair)의 유사도는 최소화하는 방향으로 학습이 진행된다.32 이 과정을 통해 모델은 단어와 픽셀 간의 복잡한 관계를 학습하게 된다. 아키텍처는 이미지의 시각적 특징을 추출하는 이미지 인코더(주로 ResNet 또는 Vision Transformer)와 텍스트의 의미를 추출하는 텍스트 인코더(주로 Transformer)로 구성된다.32

CLIP의 가장 강력한 특징 중 하나는 **제로샷(Zero-shot) 학습 능력**이다. 모델이 훈련 중에 보지 못했던 새로운 개념이나 객체에 대해서도 분류가 가능하다. 예를 들어, 특정 이미지의 클래스를 판별할 때, "A photo of a [클래스명]"과 같은 여러 텍스트 프롬프트를 만들어 각각 텍스트 임베딩으로 변환한다. 그 후, 주어진 이미지의 임베딩과 각 텍스트 임베딩 간의 코사인 유사도를 계산하여 가장 높은 점수를 받은 클래스를 예측 결과로 선택한다.32


확산 모델의 심장부에는 노이즈를 예측하는 신경망이 있으며, 이 역할은 대부분 **U-Net** 아키텍처가 담당한다.22 U-Net은 노이즈가 낀 이미지(또는 잠재 벡터)와 현재의 노이즈 단계(timestep) $t$를 입력받아, 해당 이미지에 추가된 노이즈를 예측하는 출력을 내보낸다.29

U-Net은 이름에서 알 수 있듯이, 인코더(수축 경로)와 디코더(확장 경로)가 대칭적인 'U'자 형태를 이루는 독특한 구조를 가진다.35

- **인코더 (수축 경로):** 입력 이미지에 대해 연속적인 컨볼루션(convolution)과 다운샘플링(max pooling) 연산을 적용한다. 이 과정을 통해 이미지의 공간적 해상도는 점차 줄어들고, 채널 수는 늘어나면서 이미지의 전반적인 맥락과 고수준의 의미론적 특징(feature)이 추출 및 압축된다.37
- **디코더 (확장 경로):** 인코더의 압축된 특징 맵을 입력받아, 업샘플링(upsampling)과 컨볼루션 연산을 통해 점차 이미지의 해상도를 원본 크기로 복원한다.37

U-Net의 가장 중요한 구조적 특징은 **스킵 커넥션(Skip Connection)**이다. 이는 인코더의 각 계층에서 생성된 특징 맵을 디코더의 동일한 해상도를 가진 대응 계층으로 직접 연결하는 경로이다.38 이 연결은 인코더의 다운샘플링 과정에서 필연적으로 손실될 수 있는 저수준의 공간적, 세부적 정보(예: 객체의 정확한 윤곽선, 미세한 질감)를 보존하여 디코더로 직접 전달하는 역할을 한다.37 덕분에 디코더는 고수준의 의미론적 정보(무엇을 그릴지)와 저수준의 공간적 정보(어떻게 정교하게 그릴지)를 모두 활용하여 노이즈를 제거할 수 있으며, 최종적으로 선명하고 디테일이 살아있는 이미지를 생성할 수 있다.40

흥미롭게도, 스킵 커넥션은 정보 보존이라는 긍정적 역할 외에 이중적인 측면을 가진다. 본래 이미지 분할과 같이 입력과 출력의 공간적 구조를 일치시키는 것이 목표였던 U-Net의 설계 유산으로 인해, 스킵 커넥션은 인코더의 정보를 디코더로 너무 직접적으로 전달하려는 경향이 있다. 이는 노이즈가 낀 입력으로부터 완전히 새로운 깨끗한 출력을 만들어야 하는 확산 모델의 '변환' 과정을 오히려 제약할 수 있다.41 실제로 스킵 커넥션의 가중치를 조절하는 'Skip-Tuning'과 같은 기법을 통해 더 적은 샘플링 단계로도 고품질 이미지를 생성할 수 있다는 연구 결과는, 이 연결이 단순한 보조 장치가 아니라 모델 성능에 직접적인 영향을 미치는 중요한 설계 요소임을 시사한다.41


확산 모델이 단순히 무작위 이미지를 생성하는 것을 넘어, "별이 빛나는 밤에 스케이트보드를 타는 우주비행사"와 같은 특정 텍스트 프롬프트를 따르도록 만들기 위해서는 조건부 정보(conditional information)를 노이즈 예측 과정에 주입해야 한다. 이 역할을 수행하는 핵심 메커니즘이 바로 **크로스-어텐션(Cross-Attention)**이다.26

크로스-어텐션은 트랜스포머 아키텍처에서 유래한 개념으로, 두 개의 서로 다른 정보 시퀀스를 연관시키는 역할을 한다. T2I 모델의 U-Net 내부에서는 이미지 정보와 텍스트 정보가 이 메커니즘을 통해 상호작용한다. 구체적인 과정은 다음과 같다 30:

1. U-Net의 중간 계층에서 처리 중인 이미지의 특징 맵(각 픽셀 또는 잠재 공간의 위치)이 **쿼리(Query, Q)** 벡터 시퀀스로 변환된다. 이는 "이 위치에 무엇을 그려야 하는가?"라는 질문에 해당한다.
2. CLIP 텍스트 인코더를 통해 변환된 텍스트 프롬프트의 각 단어(토큰) 임베딩이 **키(Key, K)**와 **값(Value, V)** 벡터 시퀀스가 된다. 키는 "나는 이런 의미를 가진 단어이다"를, 값은 "나를 참고한다면 이런 정보를 사용해라"를 나타낸다.
3. 각 이미지 위치의 쿼리(Q)는 텍스트의 모든 단어에 대한 키(K)와 내적(dot product)을 통해 유사도, 즉 '어텐션 스코어(attention score)'를 계산한다. 이 점수는 특정 이미지 영역이 텍스트의 어떤 단어와 가장 관련이 깊은지를 나타낸다.
4. 이 어텐션 스코어를 가중치로 사용하여 값(V) 벡터들을 가중 합산한다. 이를 통해 현재 이미지 위치를 그리는 데 가장 중요한 텍스트 정보를 담은 '문맥 벡터(context vector)'가 생성된다.
5. 마지막으로, 이 문맥 벡터가 U-Net의 특징 맵에 통합되어, 이후의 노이즈 예측 과정이 텍스트 프롬프트의 의미를 충실히 반영하도록 유도한다.31

결론적으로, CLIP이 '무엇을' 그려야 하는지에 대한 전체적인 목표를 설정하면, U-Net은 이미지의 '공간 구조'를 다루며, 크로스-어텐션은 U-Net이 각 부분을 그릴 때마다 CLIP이 제시한 목표(텍스트 프롬프트)를 계속해서 참고하도록 '안내'하는 역할을 수행한다. 이 모듈식 아키텍처는 각 구성 요소의 독립적인 발전과 교체를 용이하게 하여 전체 시스템의 빠른 개선을 가능하게 했다.



세 가지 주요 T2I 모델은 각기 다른 아키텍처 철학과 접근 방식을 기반으로 한다.

- **DALL-E 2:** OpenAI가 개발한 이 모델은 세 부분으로 구성된 독특한 파이프라인 구조를 가진다. 첫째, **CLIP 모델**이 입력된 텍스트 프롬프트를 분석하여 '텍스트 임베딩'을 생성한다. 둘째, **Prior 모델**이라는 별도의 신경망이 이 텍스트 임베딩을 입력받아, 해당 텍스트에 대응하는 '이미지 임베딩'을 예측한다. 이 단계에서 텍스트의 추상적 개념과 이미지의 시각적 개념이 연결된다. 마지막으로, **Decoder 모델**(unCLIP 또는 GLIDE 기반의 확산 모델)이 Prior가 생성한 이미지 임베딩을 조건으로 받아 최종 이미지를 생성한다. 이처럼 단계를 명확히 분리한 구조가 특징이다.44
- **Stable Diffusion:** Stability AI가 공개한 이 모델은 **잠재 확산 모델(LDM)** 아키텍처를 기반으로 한다. 핵심은 VAE(Variational Autoencoder)를 사용하여 이미지를 픽셀 공간에서 훨씬 작은 크기의 '잠재 공간'으로 압축하는 것이다. 실제 노이즈 추가 및 제거 과정은 이 효율적인 잠재 공간 내에서 **U-Net**을 통해 이루어진다. 텍스트 조건은 **CLIP 텍스트 인코더**가 프롬프트를 임베딩으로 변환한 뒤, U-Net 내부의 크로스-어텐션 레이어를 통해 주입된다. 이 아키텍처는 계산 효율성을 극대화하여 상대적으로 낮은 사양의 하드웨어에서도 구동이 가능하다는 장점이 있다.30
- **Midjourney:** 독자적인 비공개(proprietary) 모델로, 정확한 아키텍처는 외부에 알려져 있지 않다. 그러나 생성되는 이미지의 특성과 작동 방식을 볼 때, 고도로 최적화된 확산 모델을 기반으로 할 것으로 추정된다. Midjourney의 가장 큰 특징은 V1부터 현재의 V7에 이르기까지 지속적인 모델 업데이트를 통해 뚜렷한 성능 향상과 스타일 변화를 보여준다는 점이다. 각 버전은 특유의 미학적 스타일을 가지고 있으며, 이는 Midjourney 팀의 독자적인 데이터 큐레이션과 모델 튜닝 철학을 반영한다.13


- **사실성(Realism):** 세 모델 모두 높은 수준의 사실적인 이미지를 생성할 수 있지만, Midjourney가 특히 극사실적이고 선명한 이미지 생성에서 강점을 보인다는 평가가 많다.48 Stable Diffusion은 사용자가 선택하는 커뮤니티 체크포인트 모델에 따라 다양한 수준의 사실성을 구현할 수 있다. DALL-E 3는 때때로 사실적인 묘사보다 양식화(stylized)되거나 일러스트와 같은 스타일로 결과물을 내놓는 경향이 있다.48
- **예술성(Artistic Quality):** 예술적 표현력 측면에서는 Midjourney가 독보적이라는 평가를 받는다. 특유의 극적이고 세련된 미학은 'Midjourney 스타일'이라고 불릴 정도로 뚜렷한 정체성을 구축했으며, 많은 아티스트와 디자이너들에게 선호된다.46 Stable Diffusion은 오픈 소스 생태계를 통해 무한에 가까운 예술적 스타일을 구현할 수 있는 유연성이 최대 장점이다.
- **프롬프트 이해도(Prompt Fidelity):** DALL-E 3는 ChatGPT와의 통합을 통해 복잡하고 미묘한 뉘앙스를 담은 자연어 프롬프트를 매우 정확하게 이해하고 반영하는 데 탁월한 능력을 보여준다.47 Midjourney는 프롬프트를 문자 그대로 따르기보다는 창의적으로 재해석하여 예술성을 극대화하는 경향이 있다.47 Stable Diffusion은 프롬프트의 특정 단어에 가중치를 부여하는 등 정교한 제어가 가능하여 숙련된 사용자가 원하는 바를 정확히 구현할 수 있다.
- **텍스트 렌더링(Text Rendering):** 이미지 내에 정확한 텍스트를 그려 넣는 것은 전통적으로 모든 T2I 모델의 가장 큰 약점 중 하나였다. 그러나 DALL-E 3와 Midjourney V6 이후 버전들은 이 부분에서 상당한 개선을 이루어, 비교적 정확한 철자와 배치를 가진 텍스트를 생성할 수 있게 되었다.13 Stable Diffusion 역시 특정 모델과 기법을 통해 텍스트 렌더링에서 강점을 보일 수 있다.50


- **인페인팅/아웃페인팅:** DALL-E 2는 이 기능을 초기에 도입하여 사용자가 이미지의 특정 부분을 자연스럽게 수정하거나(인페인팅), 이미지의 캔버스를 바깥으로 확장하는(아웃페인팅) 강력한 편집 기능을 제공했다.11 Midjourney 역시 'Vary (Region)', 'Pan', 'Zoom' 등의 기능을 통해 유사한 수준의 정교한 제어를 지원한다.13
- **커스터마이제이션:** 확장성과 커스터마이제이션 측면에서는 Stable Diffusion이 압도적인 우위를 점한다. 오픈 소스라는 특성 덕분에 사용자가 직접 모델을 특정 데이터셋으로 미세 조정(fine-tuning)하거나, LoRA(Low-Rank Adaptation)와 같은 경량화된 학습 기법을 통해 원하는 스타일, 캐릭터, 객체를 모델에 학습시키는 것이 자유롭다.29
- **ControlNet을 통한 공간적 제어:** Stable Diffusion 생태계에서 탄생한 ControlNet은 T2I 기술의 제어 가능성을 한 차원 끌어올린 혁신이다. 이 기술은 사전 훈련된 거대 확산 모델의 가중치는 그대로 유지(locked)한 채, 훈련 가능한 복사본을 추가하여 작동한다. 이 복사본은 윤곽선(edge), 깊이(depth), 인간의 자세(pose), 분할 맵(segmentation map) 등 다양한 형태의 공간적 조건을 입력받아 이미지 생성을 정밀하게 제어한다.51 이를 통해 사용자는 "이러한 구도와 자세를 가진 인물을 그려줘"와 같은 매우 구체적인 지시를 내릴 수 있게 되었고, T2I 기술은 단순 생성을 넘어 정교한 편집 및 디자인 도구로의 전환점을 맞이했다.


다음 표는 각 모델의 주요 특징을 요약하여 사용자가 자신의 목적에 가장 적합한 도구를 선택하는 데 도움을 주기 위해 작성되었다.

| 특징                | DALL-E 3 (OpenAI)                                            | Midjourney                                             | Stable Diffusion (Stability AI)                              |                                |                                                  |                        |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------ | ------------------------------------------------ | ---------------------- |
| **핵심 아키텍처**   | CLIP Prior + Diffusion Decoder 21                            | 독자적 비공개 확산 모델 13                             | 잠재 확산 모델 (LDM) 30                                      |                                |                                                  |                        |
| **이미지 품질**     | **사실성:** 우수, 때로 양식화됨 48                           | 예술성: 다재다능 48                                    | **사실성:** 매우 뛰어남 48                                   | 예술성: 독보적, 특유의 미학 46 | **사실성:** 뛰어남, 커뮤니티 모델에 따라 다양 46 | 예술성: 매우 유연함 50 |
| **프롬프트 이해도** | 매우 뛰어남, 자연어 뉘앙스 포착 47                           | 창의적 재해석 경향 47                                  | 뛰어남, 프롬프트 가중치 조절 가능                            |                                |                                                  |                        |
| **제어/편집 기능**  | 인페인팅, ChatGPT 통한 대화형 수정 12                        | Vary (Region), Pan, Zoom, Style/Character Reference 13 | 인페인팅, 아웃페인팅, ControlNet, LoRA 등 확장성 최고 31     |                                |                                                  |                        |
| **접근성/플랫폼**   | 웹(ChatGPT), API, 모바일 앱 46                               | Discord, 웹 인터페이스 13                              | 오픈 소스 (로컬 설치), API, 다양한 웹 UI 50                  |                                |                                                  |                        |
| **비용**            | 무료 티어(제한적), 유료 구독(ChatGPT Plus), API 사용량 기반 48 | 유료 구독 전용 48                                      | 오픈 소스는 무료, 클라우드/API 사용 시 비용 발생 50          |                                |                                                  |                        |
| **주요 사용자**     | 일반 사용자, 마케터, 빠른 컨셉 디자이너 46                   | 아티스트, 디자이너, 창의적 전문가 46                   | 개발자, 연구자, 기술적 사용자, 커스터마이징이 필요한 전문가 50 |                                |                                                  |                        |



T2I 기술은 창의 산업 전반에 걸쳐 아이디어를 시각화하고 콘텐츠를 제작하는 방식을 혁신하고 있다.

- **예술 및 디자인:** 아티스트와 디자이너는 T2I를 활용하여 초기 컨셉을 신속하게 시각화하고 다양한 스타일을 탐색하며 창작 과정의 효율성을 높이고 있다. 전통적인 스케치나 페인팅 기술이 없는 사람들도 정교한 프롬프트를 통해 자신의 창의적인 비전을 구현할 수 있게 되었다.5
- **광고 및 마케팅:** 마케팅 분야에서는 타겟 고객에 맞춘 개인화된 캠페인 비주얼을 제작하거나, A/B 테스트를 위한 다양한 광고 소재를 신속하게 생성하는 데 T2I 기술이 활용된다. 이를 통해 고가의 사진 촬영이나 외주 그래픽 디자인에 대한 의존도를 줄이고 비용 효율적으로 고품질 콘텐츠를 생산할 수 있다.5
  - **사례 1: Coca-Cola "Create Real Magic":** 코카콜라는 ChatGPT-4와 DALL-E를 결합한 "Create Real Magic" 플랫폼을 론칭했다. 이 캠페인은 소비자들이 코카콜라의 상징적인 브랜드 자산을 활용하여 자신만의 AI 예술 작품을 만들도록 유도했다. 그 결과, 폭발적인 사용자 참여와 바이럴 효과를 이끌어내며 브랜드와의 유대감을 강화하고 매출 증대에 기여했다.57
  - **사례 2: Heinz DALL-E 2 캠페인:** 하인즈는 "르네상스 스타일의 케첩 병"이나 "달에 있는 케첩 병"과 같은 창의적인 프롬프트를 DALL-E 2에 입력했을 때 어떤 이미지가 생성되는지를 보여주는 캠페인을 진행했다. 이는 AI가 '케첩'하면 '하인즈'를 연상할 정도로 브랜드의 상징성이 강력하다는 메시지를 전달했으며, 전 세계적으로 8억 5천만 회 이상의 노출을 기록하며 큰 성공을 거두었다.57


T2I 및 관련 생성형 AI 기술은 과학 연구 분야에서도 새로운 패러다임을 열고 있다. 실험 비용이 높거나, 데이터 수집이 어렵거나, 윤리적 제약이 있는 영역에서 '합성 데이터(synthetic data)'를 생성하여 연구를 가속화하는 핵심 도구로 부상하고 있다.

- **가설 생성 및 데이터 증강:** 방대한 과학 문헌과 데이터셋을 분석하여 신약 개발이나 질병 메커니즘에 대한 새로운 가설을 생성하는 데 활용된다.60 또한, 희귀 질병 연구와 같이 데이터가 부족한 경우, 실제 데이터의 통계적 특성을 모방한 합성 데이터를 생성하여 AI 모델의 훈련 성능을 향상시킬 수 있다.61
- **신약 및 신소재 개발:** 원하는 화학적, 물리적 속성을 가진 새로운 분자 구조나 단백질 서열을 '설계'하는 데 생성 모델이 사용된다. 예를 들어, 홍콩의 Insilico Medicine은 AI를 통해 완전히 새로운 구조의 특발성 폐섬유증 치료제 후보 물질을 발굴하여 임상 2상 시험까지 진입시키는 성과를 거두었다.60 이는 전통적인 '관찰/실험 후 분석'의 연구 패러다임을 '생성/시뮬레이션 후 검증'으로 전환시킬 잠재력을 보여준다.
- **복잡한 현상 시각화:** "미토콘드리아 내부의 ATP 합성 과정"과 같은 텍스트 설명을 기반으로 해당 생물학적 구조나 프로세스의 이미지를 생성하여, 연구자들의 이해를 돕고 교육 자료로 활용할 수 있다.61


T2I 기술의 성공은 생성의 차원을 2D 이미지에서 3D 객체와 4D 시공간(비디오)으로 확장하는 연구를 촉발했다. 그러나 차원이 확장됨에 따라 '일관성'이라는 기술적 난제는 기하급수적으로 복잡해진다.

- **Text-to-Video:** 텍스트 프롬프트로부터 동영상 클립을 생성하는 기술이다. OpenAI의 **Sora**, Runway의 **Gen-2/Gen-3**, Google의 **Veo** 등이 대표적인 모델이다.62 T2V 기술은 T2I 기술을 기반으로 하지만, 2D 이미지의 '공간적 일관성'을 넘어 '시간적 일관성(temporal consistency)'이라는 핵심 과제를 해결해야 한다. 즉, 비디오의 프레임이 진행되는 동안 등장하는 객체의 정체성, 형태, 외관이 일관되게 유지되어야 하며, 움직임이 물리 법칙과 논리에 부합해야 한다. 이는 훨씬 더 높은 계산 비용과 방대한 양의 고품질 비디오-텍스트 쌍 데이터셋을 요구하는 어려운 문제이다.62
- **Text-to-3D:** 텍스트 프롬프트로부터 3D 모델(메시, NeRF, 가우시안 스플래팅 등)을 생성하는 기술이다. 현재 주류 접근 방식은 T2I 모델을 활용하여 특정 객체에 대한 여러 각도의 2D 이미지를 생성한 후, 이를 NeRF(Neural Radiance Fields)나 3D 재구성 알고리즘을 통해 3D 모델로 조합하는 것이다.64 DreamFusion과 같은 연구는 2D 확산 모델의 사전 지식을 활용하여 3D 생성을 직접 유도하는 방식을 제안했다.65 T23D는 공간적 일관성에 더해 모든 각도에서의 '시점 일관성(view consistency)'을 확보해야 한다. 즉, 앞에서 본 모습과 뒤에서 본 모습이 서로 모순되지 않아야 한다. 고품질의 대규모 3D-텍스트 쌍 데이터셋이 부족하기 때문에, 2D 이미지 생성 모델에 의존하는 방식이 많이 연구되고 있다.65



T2I 모델은 비약적인 발전을 이루었지만, 여전히 사용자의 복잡한 의도를 완벽하게 이해하고 시각적으로 구현하는 데 한계를 보인다. 특히 여러 객체 간의 상호작용, 정확한 공간적 배치, 미묘한 속성 조합 등을 담은 복잡한 프롬프트를 처리할 때 어려움을 겪는다.66 예를 들어, "파란색 큐브 위에 있는 빨간색 구체"라는 프롬프트에 대해 색상이나 형태가 뒤바뀌는 '속성 누출(attribute leakage)'이 발생하거나, 프롬프트에 명시된 특정 객체가 생성되지 않는 '개체 누락(missing entities)' 문제가 나타날 수 있다.66 이로 인해 사용자는 원하는 결과물을 얻기 위해 프롬프트를 여러 번 수정하고 정제하는 반복적인 과정을 거쳐야 한다.68


T2I 모델의 가장 두드러진 약점 중 하나는 이미지 내에 정확하고 일관된 텍스트를 렌더링하는 능력의 부재이다. 생성된 이미지 속 텍스트는 종종 철자가 틀리거나, 의미 없는 기호의 나열처럼 보이거나, 글자의 형태가 왜곡되는 등 심각한 오류를 보인다.67

이 문제의 근본적인 원인은 T2I 모델의 작동 방식과 텍스트라는 데이터의 본질 사이에 존재하는 '표상 방식의 불일치'에 있다. 확산 모델은 세상을 연속적인 '픽셀의 장(field)'으로 이해하고, 픽셀 단위의 지역적(local) 정보를 기반으로 확률적 생성을 수행한다.69 반면, 텍스트는 'a', 'p', 'p', 'l', 'e'와 같이 이산적인 '기호의 배열'이며, 각 기호의 정확한 형태와 엄격한 순서가 의미를 결정한다. 모델이 '사과 그림'을 그릴 때는 사과의 형태와 색상에 대한 확률적 분포 내에서 다양한 변형이 허용되지만, 'apple'이라는 글자를 쓸 때는 확률적 변동이 거의 허용되지 않는다. 이처럼 모델이 세상을 이해하는 방식(연속적, 확률적)과 텍스트의 본질(이산적, 규칙 기반)이 충돌하기 때문에, 여러 문자에 걸친 장거리 공간적 종속성(long-range spatial dependencies)을 유지하는 데 구조적인 어려움을 겪는 것이다.69


T2I 모델의 성능과 행동 양식은 전적으로 학습 데이터에 의해 결정된다. 현대의 거대 T2I 모델들은 수십억 개의 이미지-텍스트 쌍으로 구성된 웹 스케일 데이터셋으로 훈련되는데, 그 대표적인 예가 58억 5천만 개의 데이터를 포함하는 **LAION-5B**이다.3

그러나 이러한 데이터셋은 심각한 문제를 내포하고 있다. LAION 데이터셋은 Common Crawl과 같은 프로젝트를 통해 인터넷 전체에서 자동으로 데이터를 수집(scraping)하며, 이 과정에서 인간의 감독이나 섬세한 필터링이 거의 이루어지지 않는다.72 그 결과, 인터넷에 존재하는 온갖 편향되고 유해한 콘텐츠가 데이터셋에 그대로 포함된다. 연구에 따르면 LAION 데이터셋에는 인종차별적, 여성혐오적, 폭력적인 내용뿐만 아니라, 저작권이 있는 자료, 개인정보, 심지어 아동 성착취물(CSAM)로 의심되는 콘텐츠까지 포함되어 있음이 밝혀졌다.71

이러한 '오염된' 데이터로 학습된 모델은 사회에 존재하는 유해한 편견과 고정관념을 그대로 학습하고, 때로는 이를 더욱 증폭시켜 결과물로 재생산한다.74 더 큰 문제는, AI가 생성한 편향된 이미지가 다시 인터넷에 게시되고, 이것이 미래의 AI 모델을 훈련시키는 데이터로 재사용되면서 편향이 스스로를 강화하는 악순환(problematic cycle)이 발생할 수 있다는 점이다.76 이처럼 데이터셋에 내재된 편향과 유해성은 알고리즘 개선만으로는 해결하기 어려운 '원죄(original sin)'와 같으며, 이는 기술 개발의 전제 조건으로 데이터 수집 및 정제 단계에서의 사회적, 윤리적 거버넌스가 반드시 필요함을 시사한다.



**딥페이크(Deepfake)**는 AI를 이용하여 특정 인물의 얼굴이나 목소리를 합성하여 매우 사실적인 가짜 영상이나 음성을 만들어내는 기술을 의미한다.77 T2I 기술의 발전은 이러한 딥페이크 이미지의 생성 또한 용이하게 만들었다. 이 기술은 심각한 사회적 위험을 내포하고 있다. 정치인의 연설을 조작하여 허위 정보를 유포하거나(예: 벨기에 정당이 제작한 가짜 트럼프 연설) 78, 특정인의 얼굴을 음란물에 합성하는 '비동의 성적 이미지'를 제작하는 데 악용될 수 있다.79 최근 테일러 스위프트나 뉴저지 고등학생들의 사례에서 보듯, 이러한 악용은 개인에게 회복하기 힘든 정신적 피해를 주고 사회적 혼란을 야기한다.79 기술 발전으로 딥페이크 제작이 점점 더 쉬워지고 정교해짐에 따라, 그 위협은 더욱 커지고 있다.


T2I 모델은 웹에서 수집된 방대한 데이터로 학습하기 때문에, 우리 사회에 만연한 인종 및 성별에 대한 고정관념을 그대로 학습하고 결과물에 반영, 심지어 증폭시키는 경향이 있다.77

- **성별 편향:** "의사", "CEO", "건설 노동자"와 같은 직업 관련 프롬프트에는 주로 백인 남성의 이미지를 생성하는 반면, "간호사", "비서", "청소하는 사람"과 같은 프롬프트에는 여성의 이미지를 생성하여 성 역할에 대한 고정관념을 강화한다.81
- **인종 및 문화적 편향:** "매력적인 사람"이라는 프롬프트에는 서구 중심의 미적 기준이 반영된 백인 이미지를 주로 생성하고, "테러리스트"나 "불법적인 사람"과 같은 부정적인 프롬프트에는 특정 인종(중동인, 유색인종)의 특징을 가진 이미지를 생성하는 경향이 발견되었다.76 또한, 특정 국적과 가난 또는 부유함을 연관시키는 이미지를 생성함으로써 유해한 인종적, 지정학적 편견을 재생산하고 확산시킨다.82

이러한 편향 문제를 해결하기 위해 콘텐츠 필터링을 강화하는 것은 필연적으로 표현의 자유를 억압하고 창의성을 제한할 수 있다는 딜레마를 야기한다. 실제로 DALL-E 3의 강력한 안전 필터는 합법적인 예술적 표현까지 차단하여 "뇌엽 절제술을 받았다(lobotomized)"는 비판을 받기도 했다.21 따라서 유해 콘텐츠를 정밀하게 차단하면서도 창작 활동을 위축시키지 않는 섬세한 규제 균형점을 찾는 것이 중요한 과제로 남아있다.


생성형 AI의 부상은 저작권법의 근간을 흔드는 새로운 질문들을 던지고 있으며, 그 중심에는 **Getty Images 대 Stability AI** 소송이 있다. 세계적인 스톡 이미지 기업인 Getty Images는 Stability AI가 자사의 저작권으로 보호되는 수백만 장의 이미지를 라이선스 계약 없이 무단으로 수집(scraping)하여 Stable Diffusion 모델을 훈련하는 데 사용했다고 주장하며 영국과 미국에서 소송을 제기했다.85

이 소송의 핵심 쟁점은 다음과 같다:

1. **AI 훈련을 위한 데이터 복제:** AI 모델 훈련을 위해 대량의 저작물을 복제하는 행위 자체가 저작권 침해에 해당하는가?
2. **공정 이용(Fair Use) 항변:** Stability AI 측은 AI 훈련이 원본 이미지를 그대로 사용하는 것이 아니라, 통계적 패턴을 학습하여 완전히 새로운 이미지를 생성하는 '변형적 사용(transformative use)'에 해당하므로 공정 이용 원칙에 따라 허용되어야 한다고 주장한다.89
3. **생성물의 저작권 침해:** AI가 생성한 결과물에 Getty Images의 워터마크가 왜곡된 형태로 나타나거나 86, 원본 이미지와 실질적으로 유사한 부분이 포함된 경우, 이는 원 저작물에 대한 2차적 저작권 침해로 볼 수 있는가?.85

이 소송의 본질은 단순히 법적 해석의 문제를 넘어, AI 시대에 창출되는 막대한 가치를 누가 소유하고 분배할 것인가에 대한 사회적, 경제적 문제와 맞닿아 있다. AI 기업들은 인터넷의 방대한 데이터를 '원자재'로 사용하여 새로운 가치를 창출하지만, 그 원자재를 제공한 수많은 원작자들에게는 합당한 보상이 이루어지지 않는 현재의 구조에 대한 근본적인 문제 제기인 것이다.89 이 재판의 결과는 향후 생성형 AI 산업의 데이터 활용 방식과 비즈니스 모델, 그리고 창작 생태계와의 관계를 규정하는 중요한 선례가 될 것이다.



생성형 AI의 미래와 인공일반지능(AGI)으로의 경로에 대해, 딥러닝 분야의 두 거장인 제프리 힌튼(Geoffrey Hinton)과 얀 르쿤(Yann LeCun)은 뚜렷한 시각차를 보인다. 이들의 대립은 AI 연구의 근본적인 두 가지 접근법, 즉 '신경과학적 영감'과 '공학적 구축'의 충돌을 반영한다.

- **제프리 힌튼 (신중론/위험 경고):** 'AI의 대부'로 불리는 힌튼은 AI의 급속한 발전에 대한 깊은 우려를 표하며 구글을 퇴사했다.91 그는 과거 AGI의 등장이 30~50년 이상 걸릴 것이라 예측했지만, 최근에는 그 시기를 20년 이내로 앞당기며, AGI가 인류에게 실존적 위협(existential risk)이 될 수 있다고 강력하게 경고한다.91 그는 AI가 곧 인간의 정보 처리 능력을 넘어설 것이며, 자율적인 목표를 설정하고 인간의 통제를 벗어날 가능성을 심각하게 우려한다. 그의 관점은 뇌의 작동 방식에서 영감을 받은 연구 철학에 기반하며, 인간의 뇌와 유사한 방식으로 작동하지만 훨씬 더 강력한 '디지털 지능'의 출현 가능성에 대한 근본적인 두려움을 담고 있다.92
- **얀 르쿤 (낙관론/엔지니어링적 접근):** 힌튼, 벤지오와 함께 튜링상을 수상한 르쿤은 현재의 대규모 언어 모델(LLM)이 진정한 지능과는 거리가 멀다고 주장한다. 그는 현재의 자기회귀 모델들이 단순히 다음 단어를 예측할 뿐, 실제 세계에 대한 이해나 추론, 계획 능력이 근본적으로 결여되어 있다고 비판한다.93 르쿤은 진정한 지능을 구현하기 위해서는 언어 데이터뿐만 아니라, 비디오와 같은 감각 입력을 통해 물리 세계를 학습하고, 이를 바탕으로 추론하고 계획할 수 있는 새로운 아키텍처(예: 자기 지도 학습 기반 모델)가 필요하다고 강조한다.92 그는 AI의 위험을 과장하기보다는, 오픈 소스와 활발한 연구 협력을 통해 기술을 발전시켜 인류에게 이롭게 활용해야 한다는 공학적이고 실용적인 입장을 견지한다.93


AGI의 도래 시점에 대한 예측은 전문가들 사이에서도 크게 엇갈린다. 다수의 AI 연구자들은 2040년에서 2060년 사이를 예측하는 반면, AI 분야의 기업가들은 2030년경으로 훨씬 더 낙관적인 전망을 내놓는다.95

AGI에 도달하는 방법에 대해서도 두 가지 주요 경로가 논의되고 있다. 하나는 현재의 트랜스포머와 같은 아키텍처를 더 많은 데이터와 컴퓨팅 파워로 계속 확장(scaling up)하는 것이고, 다른 하나는 르쿤이 주장하듯 근본적으로 새로운 아키텍처와 학습 방법론이 필요하다는 것이다. 아직 어느 쪽이 옳은지에 대한 과학적 합의는 이루어지지 않았다.95 과거에도 저명한 학자들이 AI의 단기적 능력을 과대평가했던 사례가 많다는 점을 고려할 때, 현재의 예측 역시 신중하게 접근할 필요가 있다.95

결국 AGI에 대한 논의는 단순히 '기술적 특이점'의 도래 시점을 예측하는 것을 넘어, 현재 AI 개발의 '사회적 방향성'을 설정하는 중요한 기능을 수행한다. 힌튼의 실존적 위험 경고는 AI 안전(AI Safety) 연구의 중요성을 부각시키고 국제적 규제 논의를 촉발하며, 르쿤의 오픈 소스 옹호는 기술의 민주화를 통해 소수 기업의 독점을 막고 다양성을 확보해야 한다는 방향성을 제시한다. 이처럼 AGI라는 궁극적 목표에 대한 담론은, AI 기술이 인류에게 미칠 장기적인 영향을 고려하고 개발자, 정책 입안자, 대중이 참여하는 사회적 논의의 장을 형성하는, '현재의 행동을 이끌어내는 미래상'으로서 기능하고 있다.


Text-to-Image 기술은 생성적 적대 신경망(GAN)에서 확산 모델로의 패러다임 전환을 거치며 불과 몇 년 만에 비약적인 발전을 이루었다. CLIP, U-Net, 크로스-어텐션과 같은 고도로 전문화된 기술 모듈들의 성공적인 융합을 통해, 이제는 인간이 직접 그린 예술 작품이나 실제 사진에 근접하는 수준의 놀라운 시각적 결과물을 생성하기에 이르렀다. 이러한 기술적 성취는 예술, 디자인, 과학 연구 등 다양한 분야에서 창의성과 생산성의 새로운 지평을 열고 있다.

그러나 이 눈부신 발전의 이면에는 해결해야 할 중대한 과제들이 산적해 있다. 복잡한 프롬프트에 대한 이해도 부족, 이미지 내 텍스트 렌더링의 기술적 한계와 같은 문제들은 여전히 연구가 필요한 영역이다. 더 근본적으로는, 웹에서 무분별하게 수집된 데이터에 내재된 편향과 유해성은 모델이 사회적 고정관념을 재생산하고 증폭시키는 원인이 되고 있다. 또한, 딥페이크를 통한 허위 정보 유포와 기존 창작자들의 저작권 침해 문제는 기술이 사회에 미치는 부정적 영향을 최소화하기 위한 시급한 논의를 요구한다.

따라서 향후 연구는 기술적 성능 고도화와 함께 책임 있는 AI 개발을 위한 노력에 집중되어야 한다. 데이터셋의 출처와 구성을 투명하게 공개하고 편향성을 완화하기 위한 체계적인 거버넌스를 구축하는 한편, 창작자의 권리를 보호하고 AI가 창출하는 가치를 공정하게 분배하기 위한 새로운 라이선스 모델과 보상 체계에 대한 사회적 합의를 이끌어내야 한다. 딥페이크와 같은 기술의 오용을 방지하기 위한 강력한 기술적, 법적 안전장치를 마련하는 것 또한 필수적이다. 결국 Text-to-Image 기술의 미래는 기술적 혁신 그 자체뿐만 아니라, 그 기술을 어떻게 책임감 있게 사회에 통합하고 발전시켜 나갈 것인지에 대한 우리의 집단적 지혜에 달려 있을 것이다.


1. en.wikipedia.org, 8월 24, 2025에 액세스, [https://en.wikipedia.org/wiki/Text-to-image_model#:~:text=A%20text%2Dto%2Dimage%20model,an%20image%20matching%20that%20description.&text=Text%2Dto%2Dimage%20models%20began,advances%20in%20deep%20neural%20networks.](https://en.wikipedia.org/wiki/Text-to-image_model#:~:text=A text-to-image model,an image matching that description.&text=Text-to-image models began,advances in deep neural networks.)
2. Creating Reality: A Comprehensive History of Text-to-Image and Generative Imaging | by Sudhanva MG | Medium, 8월 24, 2025에 액세스, https://medium.com/@mgsudhanva/creating-reality-a-comprehensive-history-of-text-to-image-and-generative-imaging-598342f1499d
3. Text-to-image model - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Text-to-image_model
4. Artificial intelligence visual art - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Artificial_intelligence_visual_art
5. Text-To-Image Creative Applications - Meegle, 8월 24, 2025에 액세스, https://www.meegle.com/en_us/topics/text-to-image-models/text-to-image-creative-applications
6. History of generative AI - Toloka, 8월 24, 2025에 액세스, https://toloka.ai/blog/history-of-generative-ai/
7. 창작의 새로운 패러다임을 이끄는: 이미지 생성 AI, 8월 24, 2025에 액세스, https://blog-ko.superb-ai.com/image-generation-ai/
8. Text-to-Image Synthesis: A Decade Survey - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2411.16164v1
9. 그림 그려주는 AI : Stable Diffusion AI의 원리 (Latent Diffusion Model) - AI 혁명의 시대, 8월 24, 2025에 액세스, https://tech-savvy30.tistory.com/9
10. Text-to-image Diffusion Models in Generative AI: A Survey - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2303.07909v3
11. A brief history of AI-powered image generation, 8월 24, 2025에 액세스, https://sii.pl/blog/en/a-brief-history-of-ai-powered-image-generation/
12. How Does DALL·E 2 Work?. Diffusion, and more diffusion. | by Aditya Singh | Augmented AI, 8월 24, 2025에 액세스, https://medium.com/augmented-startups/how-does-dall-e-2-work-e6d492a2667f
13. Midjourney - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Midjourney
14. A History of Generative AI: From GAN to GPT-4 - MarkTechPost, 8월 24, 2025에 액세스, https://www.marktechpost.com/2023/03/21/a-history-of-generative-ai-from-gan-to-gpt-4/
15. Understanding Image Generation with Diffusion | by Deven Joshi - Medium, 8월 24, 2025에 액세스, https://medium.com/@d3xvn/understanding-image-generation-with-diffusion-78eea7e7d6f8
16. A Comparative Analysis of StackGAN and AttnGAN in Text-to-Image Generation, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/385905291_A_Comparative_Analysis_of_StackGAN_and_AttnGAN_in_Text-to-Image_Generation
17. Image Generation from Text Using StackGAN with Improved Conditional Consistency Regularization - MDPI, 8월 24, 2025에 액세스, https://www.mdpi.com/1424-8220/23/1/249
18. AttnGAN: Fine-Grained Text to Image Generation With Attentional Generative Adversarial Networks, 8월 24, 2025에 액세스, https://faculty.ist.psu.edu/suh972/Xu_AttnGAN_CVPR18.pdf
19. [1711.10485] AttnGAN: Fine-Grained Text to Image Generation with Attentional Generative Adversarial Networks - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/1711.10485
20. Scaling up GANs for Text-to-Image Synthesis - velog, 8월 24, 2025에 액세스, https://velog.io/@gwak2837/TIL-2023-03-14
21. DALL-E - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/DALL-E
22. 확산 모델 - 나무위키, 8월 24, 2025에 액세스, [https://namu.wiki/w/%ED%99%95%EC%82%B0%20%EB%AA%A8%EB%8D%B8](https://namu.wiki/w/확산 모델)
23. 확산 (Diffusion) 모델 공부를 위한 공개 소스 6選, 8월 24, 2025에 액세스, https://turingpost.co.kr/p/diffusion-model-6
24. openai/glide-text2im: GLIDE: a diffusion-based text-conditional image synthesis model - GitHub, 8월 24, 2025에 액세스, https://github.com/openai/glide-text2im
25. Photorealistic Text-to-Image Diffusion Models with Deep Language Understanding - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2205.11487
26. [2112.10752] High-Resolution Image Synthesis with Latent Diffusion Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2112.10752
27. [확산 모델] - 확산 모델 (Diffusion Model)의 정의와 Denoising ..., 8월 24, 2025에 액세스, https://untitledtblog.tistory.com/207
28. 확산 모델이란 무엇인가요? | IBM, 8월 24, 2025에 액세스, https://www.ibm.com/kr-ko/think/topics/diffusion-models
29. Diffusion Model 설명 - 기초부터 응용까지 - FFighting, 8월 24, 2025에 액세스, https://ffighting.net/deep-learning-paper-review/diffusion-model/diffusion-model-basic/
30. Latent diffusion model - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Latent_diffusion_model
31. The Stable Diffusion Model: An Introductory Guide | by Shunya Vichaar | Medium, 8월 24, 2025에 액세스, https://shunya-vichaar.medium.com/the-stable-diffusion-model-an-introductory-guide-efbfa0d5a8c5
32. [XAI] OpenAI CLIP 논문 리뷰[2] - Zero shot & Representation learning, 8월 24, 2025에 액세스, https://daeun-computer-uneasy.tistory.com/39
33. CLIP 이란? OpenAI에서 개발한 멀티모달 모델 - Data Vision - 티스토리, 8월 24, 2025에 액세스, https://vision-ai.tistory.com/275
34. Lecture 3 - CLIP - Lilys AI, 8월 24, 2025에 액세스, https://lilys.ai/notes/787493
35. UNET 모델 리소스 및 소개 (UNET Model Resources and Introduction) - ComfyUI Wiki, 8월 24, 2025에 액세스, https://comfyui-wiki.com/ko/resource/unet-models
36. A Journey from U-Net to Diffusion Models:Part 1of Generative AI with Diffusion Models | by yasmine karray | Medium, 8월 24, 2025에 액세스, https://medium.com/@ykarray29/a-journey-from-u-net-to-diffusion-models-part-1-of-generative-ai-with-diffusion-models-b228b4306b86
37. U-Net - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/U-Net
38. U-Net Architecture Explained - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/u-net-architecture-explained/
39. UNet++: Redesigning Skip Connections to Exploit Multiscale Features in Image Segmentation - PMC, 8월 24, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC7357299/
40. ScaleLong: Towards More Stable Training of Diffusion Model via Scaling Network Long Skip Connection | OpenReview, 8월 24, 2025에 액세스, [https://openreview.net/forum?id=0N73P8pH2l¬eId=QtvN1T6HTN](https://openreview.net/forum?id=0N73P8pH2l&noteId=QtvN1T6HTN)
41. The Surprising Effectiveness of Skip-Tuning in Diffusion Sampling - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2402.15170v1
42. Cross Attention | Method Explanation | Math Explained - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=aw3H-wPuRcw
43. [D] Am I the only one that thinks this behavior (cross-attention layers) is odd? - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/13rwh1c/d_am_i_the_only_one_that_thinks_this_behavior/
44. DALLE 2 Architecture - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/deep-learning/dalle-2-architecture/
45. Mastering DALL·E 2: A Breakthrough in AI Art Generation - LearnOpenCV, 8월 24, 2025에 액세스, https://learnopencv.com/mastering-dall-e-2/
46. DALL-E vs MidJourney vs Stable Diffusion: Which is Best? - Writeinteractive, 8월 24, 2025에 액세스, https://www.writeinteractive.com/dall-e-vs-midjourney-vs-stable-diffusion/
47. I Tested Midjourney vs. DALL·E to Find the Best AI Image Generator - G2 Learning Hub, 8월 24, 2025에 액세스, https://learn.g2.com/midjourney-vs-dall-e
48. Midjourney vs Stable Diffusion: 2025's Creative Clash - eWeek, 8월 24, 2025에 액세스, https://www.eweek.com/artificial-intelligence/midjourney-vs-dalle/
49. Midjourney vs. DALL-E vs. Stable Diffusion. Which AI Image Generator Is Best for Marketers? - CMS Wire, 8월 24, 2025에 액세스, https://www.cmswire.com/digital-marketing/midjourney-vs-dall-e-2-vs-stable-diffusion-which-ai-image-generator-is-best-for-marketers/
50. Midjourney vs. Stable Diffusion: A Comprehensive Comparison Guide - Aiarty, 8월 24, 2025에 액세스, https://www.aiarty.com/midjourney-guide/midjourney-vs-stable-diffusion-vs-dall-e.htm
51. DC-ControlNet: Decoupling Inter- and Intra-Element Conditions in Image Generation with Diffusion Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2502.14779v1
52. [2302.05543] Adding Conditional Control to Text-to-Image Diffusion Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2302.05543
53. lllyasviel/ControlNet: Let us control diffusion models! - GitHub, 8월 24, 2025에 액세스, https://github.com/lllyasviel/ControlNet
54. Midjourney vs Stable Diffusion: 2025's Creative Clash - eWeek, 8월 24, 2025에 액세스, https://www.eweek.com/artificial-intelligence/midjourney-vs-stable-diffusion/
55. Free AI Art Generator: Create AI Art Online - Adobe Firefly, 8월 24, 2025에 액세스, https://www.adobe.com/products/firefly/features/ai-art-generator.html
56. AI Art Generator: Free AI Image Generator & Editor | OpenArt, 8월 24, 2025에 액세스, https://openart.ai/
57. 9 Top AI Advertising Examples in 2025 - SmartyAds, 8월 24, 2025에 액세스, https://smartyads.com/blog/best-artificial-intelligence-ad-examples
58. 11 Best AI Advertising Examples of 2025 - DataFeedWatch, 8월 24, 2025에 액세스, https://www.datafeedwatch.com/blog/best-ai-advertising-examples
59. 20 Successful AI Marketing Campaigns & Case Studies [2025] - DigitalDefynd, 8월 24, 2025에 액세스, https://digitaldefynd.com/IQ/ai-marketing-campaigns/
60. Generative AI In Scientific Research: Accelerating Discoveries And Innovations, 8월 24, 2025에 액세스, https://xite.ai/blogs/generative-ai-in-scientific-research-accelerating-discoveries-and-innovations/
61. Generative AI in Life Sciences: Use Cases & Examples in 2025 - Research AIMultiple, 8월 24, 2025에 액세스, https://research.aimultiple.com/generative-ai-in-life-sciences/
62. Text-to-video model - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Text-to-video_model
63. The Evolution of Text to Video Models | Towards Data Science, 8월 24, 2025에 액세스, https://towardsdatascience.com/the-evolution-of-text-to-video-models-1577878043bd/
64. What is Text-to-3D? - Hugging Face, 8월 24, 2025에 액세스, https://huggingface.co/tasks/text-to-3d
65. TT3DSTAR - 3DLG, 8월 24, 2025에 액세스, https://3dlg-hcvc.github.io/tt3dstar/
66. [2305.13921] Compositional Text-to-Image Synthesis with Attention Map Control of Diffusion Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2305.13921
67. TextInVision: Text and Prompt Complexity Driven Visual Text Generation Benchmark - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2503.13730v1
68. Prompt and image attribute guide | Generative AI on Vertex AI - Google Cloud, 8월 24, 2025에 액세스, https://cloud.google.com/vertex-ai/generative-ai/docs/image/img-gen-prompt-guide
69. STRICT: Stress Test of Rendering Images Containing Text - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2505.18985v1
70. For AI professionals, why is text so hard for AI image generation. Not a snark question, just curious : r/OpenAI - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/OpenAI/comments/1iag3tu/for_ai_professionals_why_is_text_so_hard_for_ai/
71. Appendix (LAION-5B: An open large-scale dataset for training next generation image-text models) A Datasheet for LAION - NIPS, 8월 24, 2025에 액세스, https://papers.nips.cc/paper_files/paper/2022/file/a1859debfb3b59d094f3504d5ebb6c25-Supplemental-Datasets_and_Benchmarks.pdf
72. LAION-5B dataset - AIAAIC, 8월 24, 2025에 액세스, https://www.aiaaic.org/aiaaic-repository/ai-algorithmic-and-automation-incidents/laion-5b-dataset
73. LAION-5B, Stable Diffusion 1.5, and the Original Sin of Generative AI | TechPolicy.Press, 8월 24, 2025에 액세스, https://www.techpolicy.press/laion5b-stable-diffusion-and-the-original-sin-of-generative-ai/
74. Into the LAION's Den: Investigating Hate in Multimodal Datasets - Human Analysis Lab - Michigan State University, 8월 24, 2025에 액세스, https://hal.cse.msu.edu/assets/pdfs/papers/2023-neurips-laion-hate-in-multimodal-datasets.pdf
75. Bias inspection in LAION-5B. a Proportion of evaluated images. In... - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/Bias-inspection-in-LAION-5B-a-Proportion-of-evaluated-images-In-total-we-identified_fig4_382796566
76. AI image generator Stable Diffusion perpetuates racial and gendered stereotypes, study finds | UW News, 8월 24, 2025에 액세스, https://www.washington.edu/news/2023/11/29/ai-image-generator-stable-diffusion-perpetuates-racial-and-gendered-stereotypes-bias/
77. All creatives should know about the ethics of AI-generated images ..., 8월 24, 2025에 액세스, https://www.lummi.ai/blog/ethics-of-ai-generated-images
78. The Copyright and Ethical Issues in AI and Journalism | by Mike Reilley | Medium, 8월 24, 2025에 액세스, https://medium.com/@mikereilley1/the-copyright-and-ethical-issues-inai-and-journalism-59a17f340705
79. Deepfakes and the Ethics of Generative AI - Tepperspectives, 8월 24, 2025에 액세스, https://tepperspectives.cmu.edu/all-articles/deepfakes-and-the-ethics-of-generative-ai/
80. Navigating the Mirage: Ethical, Transparency, and Regulatory Challenges in the Age of Deepfakes | Walton College | University of Arkansas, 8월 24, 2025에 액세스, https://walton.uark.edu/insights/posts/navigating-the-mirage-ethical-transparency-and-regulatory-challenges-in-the-age-of-deepfakes.php
81. A Systematic Analysis of Gender Bias in Text-to-Image Generation Models | by L.L | Medium, 8월 24, 2025에 액세스, https://medium.com/@lopplauri/university-of-tartu-3134f53207ca
82. Demographic Stereotypes in Text-to-Image Generation, 8월 24, 2025에 액세스, https://hai-production.s3.amazonaws.com/files/2023-11/Demographic-Stereotypes.pdf
83. A Large Scale Analysis of Gender Biases in Text-to-Image Generative Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2503.23398v1
84. Survey of Bias In Text-to-Image Generation: Definition, Evaluation, and Mitigation - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2404.01030v2
85. Getty Images v Stability AI: why the remaining copyright claims are of more than secondary significance - Pinsent Masons, 8월 24, 2025에 액세스, https://www.pinsentmasons.com/out-law/analysis/getty-images-v-stability-ai-copyright-claims-significance
86. Getty Images v. Stability AI | BakerHostetler, 8월 24, 2025에 액세스, https://www.bakerlaw.com/getty-images-v-stability-ai/
87. Getty Images Statement, 8월 24, 2025에 액세스, https://newsroom.gettyimages.com/en/getty-images/getty-images-statement
88. Generative AI in the courts - Getty Images v Stability AI, 8월 24, 2025에 액세스, https://www.penningtonslaw.com/news-publications/latest-news/2024/generative-ai-in-the-courts-getty-images-v-stability-ai
89. Getty Images vs. Stability AI: The Landmark Copyright Battle Shaping The Future of Generative AI - Falcon Rappaport & Berkman LLP, 8월 24, 2025에 액세스, https://frblaw.com/getty-images-vs-stability-ai-the-landmark-copyright-battle-shaping-the-future-of-generative-ai/
90. [N] Getty Images is suing the creators of AI art tool Stable Diffusion for scraping its content, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/10ed388/n_getty_images_is_suing_the_creators_of_ai_art/
91. Geoffrey Hinton - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Geoffrey_Hinton
92. Future of Deep Learning according to top AI Experts in 2025 - Research AIMultiple, 8월 24, 2025에 액세스, https://research.aimultiple.com/future-of-deep-learning/
93. Yann LeCun Emphasizes the Promise of AI - NYAS - The New York Academy of Sciences, 8월 24, 2025에 액세스, https://www.nyas.org/ideas-insights/blog/yann-lecun-emphasizes-the-promise-ai/
94. Yann LeCun and Andrew Ng: Why the 6-month AI Pause is a Bad Idea - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=BY9KV8uCtj4
95. When Will AGI/Singularity Happen? 8,590 Predictions Analyzed - Research AIMultiple, 8월 24, 2025에 액세스, https://research.aimultiple.com/artificial-general-intelligence-singularity-timing/


# Stable Diffusion (2021)
[Text-to-Image 생성 기술](./index.md)



지난 수년간 생성 모델 분야는 생성적 적대 신경망(Generative Adversarial Networks, GANs)과 변이형 오토인코더(Variational Autoencoders, VAEs)가 주도해왔다. 그러나 이러한 모델들은 각각 고질적인 한계를 지니고 있었다. GANs는 학습 과정이 불안정하여 최적의 균형점을 찾기 어렵고, 생성된 샘플의 다양성이 부족해지는 모드 붕괴(mode-collapse) 문제에 취약했다.1 반면 VAEs는 안정적인 학습이 가능하지만, 생성된 이미지의 품질이 GANs에 비해 상대적으로 흐릿하고 세밀한 표현력이 떨어진다는 단점이 있었다.1

이러한 상황 속에서 확산 모델(Diffusion Models, DMs)은 새로운 패러다임을 제시하며 차세대 생성 모델의 핵심으로 부상했다. 비평형 열역학(nonequilibrium thermodynamics)에서 영감을 받은 이 모델들은 데이터에 점진적으로 노이즈를 추가하는 순방향 과정(forward process)과, 이 과정을 역으로 거슬러 노이즈로부터 원본 데이터를 복원하는 역방향 과정(reverse process)을 학습하는 독특한 접근법을 취한다.4 이 점진적인 노이즈 제거 방식은 GANs의 불안정한 학습 문제에서 자유로우며, VAEs보다 훨씬 더 높은 품질의 샘플을 생성할 수 있는 잠재력을 보여주었다.4 확산 모델은 복잡한 데이터 분포를 안정적으로 학습하고, 높은 시각적 충실도를 달성함으로써 이미지 생성 분야의 새로운 표준을 정립했다.


초기 확산 모델, 특히 Denoising Diffusion Probabilistic Models (DDPMs)는 이론적 우수성에도 불구하고 실용적인 측면에서 큰 장벽에 부딪혔다.4 이 모델들은 고해상도 이미지의 원본 픽셀 공간(pixel space)에서 직접 확산 및 노이즈 제거 과정을 수행했는데, 이는 막대한 양의 계산 자원과 메모리를 요구했다.1 예를 들어, 강력한 DDPM을 훈련시키는 데에는 수백 GPU 일이 소요될 수 있었으며, 추론 과정 또한 순차적인 평가로 인해 매우 느렸다.7 이러한 계산 비용의 장벽은 고품질 이미지 생성 기술의 연구와 활용을 소수의 대규모 연구 기관이나 기업에 국한시키는 결과를 낳았다.

이러한 한계를 극복하기 위해 등장한 것이 바로 Latent Diffusion Model (LDM), 즉 Stable Diffusion의 근간이 되는 핵심 아키텍처다.1 LDM의 혁신은 확산 과정을 고차원의 픽셀 공간이 아닌, 지각적으로는 동등하지만(perceptually equivalent) 차원은 훨씬 낮은 잠재 공간(latent space)에서 수행한다는 아이디어에 있다.5 먼저 사전 훈련된 오토인코더(VAE)를 사용하여 고해상도 이미지를 의미론적 정보를 압축한 저차원 잠재 벡터로 변환한다. 그 후, 계산적으로 훨씬 효율적인 이 잠재 공간 내에서 확산 및 노이즈 제거 과정을 진행하고, 마지막으로 VAE의 디코더를 통해 잠재 벡터를 다시 고해상도 이미지로 복원한다.8

이 아키텍처적 결정은 단순히 계산 효율성을 최적화하는 기술적 개선에 그치지 않았다. 이는 고품질 이미지 생성 기술을 "민주화"하는 결정적인 촉매제가 되었다. 잠재 공간에서의 연산은 계산 복잡도를 극적으로 낮추어, 이전에는 불가능했던 소비자용 GPU에서도 모델을 실행할 수 있게 만들었다.10 이러한 접근성은 Stability AI가 모델 가중치를 오픈소스로 공개하는 결정의 기반이 되었고, 전 세계 개발자 커뮤니티가 이 강력한 모델에 접근하여 실험하고, 그 위에 새로운 기술을 구축할 수 있는 토양을 마련했다. 결과적으로, 잠재 공간이라는 단일한 아키텍처 선택은 ControlNet, LoRA, DreamBooth와 같은 커뮤니티 주도 혁신의 폭발적인 성장을 가능하게 한 사회-기술적 현상의 근본 원인이 되었다.


확산 모델의 핵심은 정교하게 설계된 두 가지 확률적 과정, 즉 순방향 확산 과정과 역방향 확산 과정에 기반한다. 순방향 과정은 데이터를 점진적으로 파괴하여 순수 노이즈로 만드는 고정된 프로세스이며, 역방향 과정은 이 파괴 과정을 되돌려 노이즈로부터 데이터를 생성하는 학습된 프로세스다.


순방향 확산 과정은 원본 데이터 분포 $q(x_0)$에서 샘플링된 데이터 $x_0$에 점진적으로 가우시안 노이즈를 추가하여, 미리 정해진 총 $T$ 스텝에 걸쳐 순수 노이즈에 가깝게 만드는 과정이다.4 이 과정은 각 스텝이 직전 스텝에만 의존하는 마르코프 연쇄(Markov chain)로 정의된다.4

스텝 $t$에서 $x_{t-1}$로부터 노이즈가 추가된 $x_t$를 생성하는 조건부 확률 분포 $q(x_t \rvert x_{t-1})$는 다음과 같은 가우시안 분포로 정의된다.4
$$
q(x_t | x_{t-1}) = \mathcal{N}(x_t; \sqrt{1 - \beta_t} x_{t-1}, \beta_t \mathbf{I})
$$
여기서 $\beta_t$는 $t$ 스텝에서 추가되는 노이즈의 분산을 결정하는 하이퍼파라미터로, 분산 스케줄(variance schedule)이라고 불린다. $\beta_t$는 $t$가 증가함에 따라 점진적으로 커지도록 설정되어, 과정이 진행될수록 더 많은 노이즈가 추가된다.14

이 순방향 과정의 중요한 수학적 속성은 임의의 스텝 $t$에서의 데이터 $x_t$를 원본 데이터 $x_0$로부터 직접 계산할 수 있는 닫힌 형태(closed-form)의 표현이 존재한다는 점이다. 재매개변수화 트릭(reparameterization trick)을 활용하고, $\alpha_t = 1 - \beta_t$와 $\bar{\alpha}_t = \prod_{s=1}^{t} \alpha_s$로 정의하면, $x_t$는 다음과 같이 표현될 수 있다.4
$$
q(x_t | x_0) = \mathcal{N}(x_t; \sqrt{\bar{\alpha}_t} x_0, (1 - \bar{\alpha}_t) \mathbf{I})
$$
이 식은 $x_t$가 원본 데이터 $x_0$에 $\sqrt{\bar{\alpha}_t}$만큼 스케일링된 값과 분산이 $(1 - \bar{\alpha}_t)$인 가우시안 노이즈의 합으로 구성됨을 의미한다. 이 닫힌 형태의 표현 덕분에 훈련 과정에서 매번 전체 마르코프 연쇄를 계산할 필요 없이, 임의의 $t$에 대한 $x_t$를 효율적으로 샘플링하여 모델을 학습시킬 수 있다. 총 스텝 $T$가 충분히 크면 $\bar{\alpha}_T$는 0에 가까워지고, 따라서 $x_T$의 분포는 거의 표준 가우시안 분포 $\mathcal{N}(0, \mathbf{I})$와 구별할 수 없게 된다.12


역방향 확산 과정은 순방향 과정의 정반대 개념으로, 순수 가우시안 노이즈 $x_T \sim \mathcal{N}(0, \mathbf{I})$에서 시작하여 점진적으로 노이즈를 제거함으로써 원본 데이터 분포 $q(x_0)$의 샘플을 생성하는 것을 목표로 한다.16 이 과정은 순방향 전이 확률 $q(x_t \rvert x_{t-1})$을 역으로 뒤집는 $q(x_{t-1} \rvert x_t)$를 계산하는 것으로 볼 수 있다.

하지만 이 역방향 전이 확률 $q(x_{t-1} \rvert x_t)$는 전체 데이터셋의 분포를 알아야만 계산할 수 있기 때문에 직접 다루기 어렵다(intractable).17 따라서 이 분포를 근사하기 위해 $\theta$로 매개변수화된 신경망 $p_\theta$를 도입한다. 이 신경망은 노이즈가 낀 데이터 $x_t$와 현재 시간 스텝 $t$를 입력받아, 한 단계 전의 데이터 $x_{t-1}$의 분포에 대한 파라미터(평균 $\mu_\theta$와 분산 $\Sigma_\theta$)를 예측하도록 학습된다.6
$$
p_\theta(x_{t-1} | x_t) = \mathcal{N}(x_{t-1}; \mu_\theta(x_t, t), \Sigma_\theta(x_t, t))
$$
여기서 DDPM 논문(Ho et al., 2020)의 핵심적인 기여가 나타난다. 이론적으로 $q(x_{t-1} \rvert x_t)$는 다루기 어렵지만, 만약 원본 데이터 $x_0$를 알고 있다면 베이즈 정리에 의해 사후 확률(posterior) $q(x_{t-1} \rvert x_t, x_0)$는 다루기 쉬운 가우시안 분포가 된다는 점을 활용했다.15 신경망이 이 사후 확률의 평균을 직접 예측하도록 학습시킬 수도 있지만, 저자들은 이 평균을 나타내는 수식을 대수적으로 재배열하여, 신경망이 $x_t$를 만드는 데 사용된 노이즈 $\epsilon$를 예측하도록 만드는 것이 훨씬 더 안정적이고 효과적임을 발견했다.4

이러한 재매개변수화는 이론과 실제 구현 사이의 중요한 다리 역할을 한다. 복잡한 변분 하한(Variational Lower Bound, VLB)을 최적화하는 문제를, 실제 랜덤 노이즈 $\epsilon$와 신경망이 예측한 노이즈 $\epsilon_\theta(x_t, t)$ 사이의 평균 제곱 오차(Mean Squared Error, MSE)를 최소화하는 단순한 회귀(regression) 문제로 변환시키기 때문이다.4 이 $\| \epsilon - \epsilon_\theta \|^2$라는 목적 함수는 훈련이 매우 안정적이며, U-Net과 같은 심층 신경망 아키텍처에 완벽하게 적합하다. 이 접근법을 통해, 예측된 노이즈 $\epsilon_\theta$를 사용하여 역방향 과정의 평균 $\mu_\theta$를 다음과 같이 표현할 수 있다.6
$$
\mu_\theta(x_t, t) = \frac{1}{\sqrt{\alpha_t}} \left( x_t - \frac{\beta_t}{\sqrt{1 - \bar{\alpha}_t}} \epsilon_\theta(x_t, t) \right)
$$
결론적으로, 신경망은 각 스텝에서 이미지에 포함된 노이즈를 예측하고, 이 예측값을 사용하여 노이즈가 낀 이미지에서 노이즈를 "빼내어" 한 단계 더 깨끗한 이미지를 만드는 과정을 반복한다. 이 과정이 바로 확산 모델이 노이즈로부터 정교한 이미지를 생성하는 핵심 원리다.


Stable Diffusion의 아키텍처는 단일한 발명품이 아니라, 텍스트-이미지 생성 파이프라인의 각기 다른 핵심 하위 문제를 해결하기 위해 선택된 세 가지 성숙한 기술(VAE, U-Net, CLIP)을 독창적으로 통합한 공학적 산물이다. 이 세 가지 핵심 구성 요소는 지각적 압축, 노이즈 예측, 그리고 의미론적 안내라는 각자의 역할을 수행하며 시너지를 발휘한다.8

이러한 모듈식 접근 방식의 이면에는 깊은 설계 철학이 존재한다. 고해상도 이미지 생성의 계산 비용 문제(Problem 1)는 사전 훈련된 VAE를 통해 잠재 공간에서 연산을 수행함으로써 해결했다.8 이는 효율성 문제를 해결한다. 노이즈로부터 이미지를 생성하는 핵심 생성 메커니즘 문제(Problem 2)는 U-Net을 사용하여 해결했다. U-Net은 스킵 커넥션을 통해 미세한 공간적 디테일을 보존하는 능력이 입증된 아키텍처로, 반복적인 노이즈 제거 작업에 최적화되어 있다.20 생성 과정을 텍스트로 제어하는 문제(Problem 3)는 텍스트와 이미지 간의 의미론적 관계를 이해하도록 특별히 설계된 CLIP 텍스트 인코더를 도입하여 해결했다.22 마지막으로, 이 세 모듈을 어떻게 통합할 것인가라는 문제(Problem 4)는 U-Net 내부에 교차 어텐션(cross-attention) 레이어를 삽입하여 해결했다. 이를 통해 텍스트 프롬프트의 의미론적 안내 신호가 잠재 공간 내에서 U-Net의 공간적 노이즈 제거 작업을 정밀하게 조종할 수 있게 되었다.1 이처럼 Stable Diffusion은 각기 다른 문제에 대한 강력한 기존 해결책들을 교차 어텐션이라는 접착제로 결합하여 시너지를 창출한, 탁월한 시스템 공학의 결과물이다.


VAE는 Stable Diffusion 아키텍처의 입구와 출구를 담당하며, 계산 효율성을 극대화하는 핵심적인 역할을 수행한다.

- **역할:** VAE의 주된 역할은 고차원의 픽셀 공간 이미지를 저차원의 의미 있는 잠재 공간 표현으로 압축(인코딩)하고, 확산 과정이 끝난 후 이 잠재 공간 표현을 다시 픽셀 공간 이미지로 복원(디코딩)하는 것이다.10 예를 들어, 512x512x3 크기의 이미지를 64x64x4 크기의 잠재 벡터로 압축함으로써, 확산 과정이 처리해야 할 데이터의 양을 수십 배 줄일 수 있다.8 이는 모델의 훈련 및 추론에 필요한 메모리와 계산 시간을 획기적으로 감소시켜, 소비자용 하드웨어에서도 고해상도 이미지 생성이 가능하게 만드는 결정적인 요소다.1

- **인코딩 과정:** VAE의 인코더는 입력 이미지 $x$를 받아, 픽셀의 세부적인 정보보다는 이미지의 핵심적인 의미론적 특징(semantic features)을 포착하는 잠재 벡터 $z$로 변환한다.25 표준 오토인코더와 달리 VAE는 

  $z$를 잠재 공간 내의 단일 지점이 아닌, 평균 $\mu$와 분산 $\sigma^2$으로 정의되는 확률 분포로 인코딩한다.29 이러한 확률론적 접근은 잠재 공간이 부드럽고 연속적인 구조를 갖도록 강제하는 정규화(regularization) 효과를 낳는다. 이는 생성 과정에서 잠재 공간 내의 서로 다른 지점들 사이를 원활하게 보간(interpolation)하여 다양하고 새로운 이미지를 생성하는 데 필수적이다.31

- **디코딩 과정:** U-Net 기반의 확산 모델이 잠재 공간에서 노이즈가 완전히 제거된 최종 잠재 벡터 $z_0$를 생성하면, VAE의 디코더가 이 벡터를 입력받아 최종적으로 사용자가 보는 고해상도 픽셀 이미지로 재구성한다.19 VAE 디코더의 성능은 최종 이미지의 선명도, 색감, 디테일 등 시각적 품질에 직접적인 영향을 미친다. 이 때문에 커뮤니티에서는 더 나은 시각적 결과를 위해 기본 VAE를 개선된 버전으로 교체하여 사용하기도 한다.33


U-Net은 Stable Diffusion의 심장부로서, 잠재 공간 내에서 실제 노이즈 제거 작업을 수행하는 핵심 예측 모델이다.

- **구조:** U-Net은 대칭적인 인코더-디코더 구조를 가지며, 그 형태가 알파벳 'U'와 비슷하여 이름 붙여졌다.21 이 아키텍처는 원래 의료 이미지 분할(segmentation)을 위해 개발되었으나, 이미지 대 이미지 변환 작업 전반에 걸쳐 뛰어난 성능을 입증했다.20 Stable Diffusion에서는 노이즈가 낀 잠재 벡터 $z_t$, 시간 스텝 $t$, 그리고 텍스트 임베딩 $c$를 입력받아, $z_t$에 추가된 노이즈 $\epsilon$를 예측하는 역할을 한다.8
- **인코더 경로 (Downsampling Path):** 이 경로는 일련의 ResNet 블록과 다운샘플링(예: max-pooling 또는 strided convolution) 레이어로 구성된다. 입력된 잠재 벡터의 공간적 해상도를 점진적으로 줄여나가면서 채널 수를 늘린다. 이 과정을 통해 모델은 이미지의 지역적인 패턴(텍스처, 모서리 등)에서 점차 추상적이고 고차원적인 전역적 특징(객체의 형태, 구성 등)을 추출한다.9
- **디코더 경로 (Upsampling Path):** 인코더 경로와 대칭적인 구조로, 업샘플링(예: transposed convolution) 레이어와 ResNet 블록으로 구성된다. 가장 압축된 특징 맵(bottleneck)에서 시작하여, 점진적으로 공간 해상도를 원래 크기로 복원하면서 특징을 정제한다.9
- **스킵 커넥션 (Skip Connections):** U-Net 아키텍처의 가장 중요한 혁신 중 하나는 인코더 경로의 각 해상도 레벨에서 나온 특징 맵을 디코더 경로의 동일한 해상도 레벨에 직접 연결하는 것이다.9 인코더 경로에서 다운샘플링을 거치면서 손실될 수 있는 고해상도의 정밀한 공간적 정보(예: 객체의 정확한 윤곽선, 미세한 질감)가 이 연결을 통해 디코더로 직접 전달된다. 이 덕분에 디코더는 추상적인 의미 정보와 구체적인 공간 정보를 모두 활용하여 훨씬 더 정밀하고 세밀한 노이즈 예측을 수행할 수 있다. 최근 연구에 따르면, 이 스킵 커넥션은 이미지의 내용(content)과 스타일(style) 정보를 분리하여 전달하는 중요한 역할을 수행하며, 이를 조작하여 이미지 편집에 활용할 수도 있다.36
- **어텐션 메커니즘 (Attention Mechanisms):** U-Net의 중간 및 저해상도 블록에는 두 종류의 어텐션 메커니즘이 포함되어 모델의 표현력을 강화한다.
  - **Self-Attention:** 특징 맵 내의 모든 픽셀(또는 패치) 쌍 간의 관계를 계산한다. 이를 통해 모델은 이미지의 한 부분이 다른 부분과 어떻게 연관되어 있는지를 학습하여, 장거리 의존성(long-range dependency)을 포착하고 이미지 전체의 구조적 일관성을 유지하는 데 도움을 준다.20
  - **Cross-Attention:** 텍스트 프롬프트로부터의 의미론적 안내를 U-Net의 생성 과정에 주입하는 핵심적인 메커니즘이다. 이는 다음 절에서 더 자세히 다룬다.1


CLIP 텍스트 인코더는 사용자의 창의적인 의도를 기계가 이해할 수 있는 언어로 번역하는 역할을 수행한다.

- **역할:** 사용자가 입력한 텍스트 프롬프트를 U-Net이 조건부 정보로 활용할 수 있는 고차원의 숫자 벡터, 즉 텍스트 임베딩(text embedding)으로 변환하는 것이 주된 역할이다.19 Stable Diffusion은 OpenAI가 개발한 CLIP(Contrastive Language-Image Pre-training) 모델의 텍스트 인코더 부분을 사용한다.23 CLIP은 수억 개의 이미지-텍스트 쌍 데이터셋을 통해 "어떤 텍스트가 어떤 이미지와 연관되는지"를 학습했기 때문에, 텍스트와 시각적 개념 간의 깊은 의미론적 연결을 이해하고 있다.23

- **임베딩 변환 과정:** 텍스트 프롬프트는 먼저 토크나이저(tokenizer)에 의해 모델이 아는 어휘 사전에 따라 여러 개의 토큰(token)으로 분리된다. 이 토큰 시퀀스는 Transformer 기반의 CLIP 텍스트 인코더를 통과하며, 각 토큰에 해당하는 고차원 벡터들의 시퀀스로 변환된다. 이 벡터 시퀀스가 바로 U-Net에 전달되어 생성 과정을 안내하는 조건(conditioning) 정보가 된다.19

- **U-Net과의 상호작용 (Cross-Attention):** 생성된 텍스트 임베딩은 U-Net 내부의 교차 어텐션(Cross-Attention) 레이어를 통해 이미지 생성 과정에 직접적인 영향을 미친다. 이 메커니즘은 다음과 같이 작동한다.41

  1. U-Net의 특정 레이어를 통과하고 있는 이미지의 잠재 특징 맵(spatial features)은 쿼리(Query, Q) 벡터로 변환된다.

  2. CLIP 텍스트 인코더로부터 전달된 텍스트 임베딩 시퀀스는 키(Key, K)와 값(Value, V) 벡터로 변환된다.

  3. 이미지 특징(Q)과 텍스트 토큰(K) 간의 유사도(보통 내적)를 계산하여 어텐션 스코어(attention score)를 얻는다. 이 스코어는 이미지의 각 부분이 텍스트 프롬프트의 어떤 단어와 가장 관련이 깊은지를 나타낸다.

  4. 이 스코어에 소프트맥스(softmax) 함수를 적용하여 어텐션 가중치(attention weights)를 계산한다. 이 가중치는 확률 분포를 형성하며, 이미지의 특정 영역이 각 텍스트 토큰에 얼마나 "집중"해야 하는지를 결정한다.

  5. 마지막으로, 이 어텐션 가중치를 값(V) 벡터에 곱하여 가중합을 구한다. 이 결과물은 텍스트의 의미가 풍부하게 반영된 새로운 이미지 특징 맵으로, 다음 레이어로 전달된다.

     이 과정을 U-Net의 여러 레이어에서 반복함으로써, Stable Diffusion은 텍스트 프롬프트의 복잡한 의미를 이미지의 적절한 공간적 위치와 시각적 속성에 정교하게 반영할 수 있다.20


Stable Diffusion의 경이로운 생성 능력은 방대한 데이터셋과 정교한 훈련 파이프라인의 결과물이다. 그러나 이 과정은 데이터의 규모와 그 안에 내재된 문제점들 사이의 본질적인 긴장 관계를 드러낸다.


Stable Diffusion의 성능은 그 훈련 데이터의 규모와 다양성에 직접적으로 의존한다. 상상할 수 있는 거의 모든 것을 생성하기 위해서는 그만큼 광범위한 개념을 포괄하는 데이터셋이 필수적이다.

- **규모와 구성:** Stable Diffusion의 초기 버전들은 LAION-5B 데이터셋의 영어 하위 집합인 LAION-2B-en을 기반으로 훈련되었다.46 LAION-5B는 웹 크롤링 데이터인 Common Crawl에서 추출한 약 58.5억 개의 이미지-텍스트 쌍으로 구성된, 연구 목적으로 공개된 대규모 데이터셋이다.48 이 데이터셋은 CLIP 모델을 사용하여 이미지와 텍스트 설명 간의 의미론적 관련성이 높은 쌍들만을 필터링하여 구축되었다.49
- **LAION-Aesthetics:** 생성되는 이미지의 시각적 품질을 높이기 위해, 훈련 과정에서 미학적으로 우수한 이미지에 더 높은 가중치를 부여하는 전략이 사용되었다. 이를 위해 LAION-Aesthetics라는 하위 데이터셋이 구축되었다. 이 데이터셋은 LAION-5B의 이미지들 중에서 미학 점수를 예측하는 별도의 모델을 통해 "아름답다"고 평가된 이미지들을 선별한 것이다.46 Stable Diffusion v1 모델은 이 미학적 데이터셋을 훈련의 마지막 단계에서 집중적으로 활용하여 최종 결과물의 품질을 향상시켰다.51
- **내재된 편향과 문제점:** LAION-5B의 거대한 규모는 수작업으로 데이터를 정제하는 것을 사실상 불가능하게 만든다. 웹에서 자동으로 수집되고 최소한의 필터링만을 거친 이 데이터셋은 인터넷 공간에 존재하는 편향과 문제점들을 그대로 담고 있다.
  - **사회적 편향:** 데이터셋에는 인종, 성별, 직업, 문화에 대한 사회적 고정관념과 편견이 반영된 이미지와 텍스트가 다수 포함되어 있다.52 모델은 이러한 편향된 통계적 패턴을 그대로 학습하여, 특정 직업을 특정 성별이나 인종과 강하게 연관시키는 등 편향된 이미지를 생성하는 경향을 보인다.52
  - **유해 및 저작권 콘텐츠:** 데이터셋에는 저작권으로 보호되는 수많은 예술 작품, 사진, 상표가 포함되어 있어 심각한 법적 분쟁의 소지가 된다.56 더 심각한 문제로, 아동 성적 학대물(CSAM)을 포함한 폭력적이고 유해한 콘텐츠가 데이터셋 내에 포함되어 있다는 사실이 밝혀지면서 심각한 윤리적 문제를 야기했다.56

이러한 문제들은 Stable Diffusion의 힘과 위험이 동전의 양면임을 보여준다. 모델의 놀라운 일반화 능력은 LAION 데이터셋의 방대한 규모와 다양성에서 비롯되었지만, 동시에 그 데이터의 무분별하고 정제되지 않은 특성은 모델의 가장 큰 윤리적, 법적 취약점의 직접적인 원인이 되었다. 이는 대규모 데이터셋을 활용하는 생성 AI 모델이 본질적으로 안고 있는 딜레마다.


Stable Diffusion의 훈련 목표는 U-Net $\epsilon_\theta$가 모든 시간 스텝 $t$와 모든 텍스트 조건 $c$에 대해, 노이즈가 낀 잠재 벡터 $z_t$에 추가된 실제 노이즈 $\epsilon$를 가능한 한 정확하게 예측하도록 만드는 것이다.

- **손실 함수:** 이 목표를 달성하기 위해 가장 보편적으로 사용되는 손실 함수는 예측된 노이즈 $\epsilon_\theta$와 실제 노이즈 $\epsilon$ 사이의 평균 제곱 오차(Mean Squared Error, MSE)이다.18 MSE 손실은 계산이 간단하고 미분 가능하며, 안정적인 훈련을 가능하게 하여 확산 모델 훈련의 표준으로 자리 잡았다.18 Latent Diffusion Model의 손실 함수는 다음과 같이 공식화된다.1
  $$
  \mathcal{L}_{LDM} := \mathbb{E}_{\mathcal{E}(x), y, \epsilon \sim \mathcal{N}(0,1), t} \left[ \| \epsilon - \epsilon_\theta(z_t, t, \tau_\theta(y)) \|_2^2 \right]
  $$
  이 식에서 $\mathbb{E}$는 기댓값을 의미하며, 훈련 데이터셋의 이미지 $x$와 텍스트 $y$, 표준 정규 분포에서 샘플링된 노이즈 $\epsilon$, 그리고 균등 분포에서 샘플링된 시간 스텝 $t$에 대해 평균을 취한다는 의미다. $\mathcal{E}(x)$는 VAE 인코더를 통과한 이미지의 잠재 벡터 $z$를, $\tau_\theta(y)$는 CLIP 텍스트 인코더를 통과한 텍스트 임베딩을 나타낸다. 모델은 이 손실 값을 최소화하는 방향으로 가중치를 업데이트한다.

- **훈련 파이프라인:** 전체 훈련 과정은 다음과 같은 단계로 이루어진다.1

  1. **데이터 샘플링:** LAION과 같은 대규모 데이터셋에서 이미지-텍스트 쌍 $(x, y)$를 무작위로 샘플링한다.
  2. **인코딩:** VAE 인코더를 사용하여 이미지 $x$를 잠재 벡터 $z = \mathcal{E}(x)$로 변환하고, CLIP 텍스트 인코더를 사용하여 텍스트 $y$를 조건 임베딩 $c = \tau_\theta(y)$로 변환한다.
  3. **노이즈 추가:** 시간 스텝 $t$를 $1$에서 $T$ 사이에서 무작위로 샘플링하고, 표준 정규 분포에서 노이즈 $\epsilon$를 샘플링한다.
  4. 순방향 확산 과정의 닫힌 형태 공식을 사용하여 노이즈가 낀 잠재 벡터 $z_t$를 계산한다: $z_t = \sqrt{\bar{\alpha}_t} z + \sqrt{1 - \bar{\alpha}_t} \epsilon$.
  5. **노이즈 예측:** U-Net 모델에 노이즈 낀 잠재 벡터 $z_t$, 시간 스텝 $t$, 텍스트 임베딩 $c$를 입력하여 노이즈 예측값 $\epsilon_\theta(z_t, t, c)$를 얻는다.
  6. **손실 계산 및 역전파:** 실제 노이즈 $\epsilon$와 예측된 노이즈 $\epsilon_\theta$ 사이의 MSE 손실을 계산한다.
  7. **가중치 업데이트:** 계산된 손실을 기반으로 역전파(backpropagation) 알고리즘을 사용하여 U-Net과 텍스트 인코더의 학습 가능한 가중치 $\theta$를 업데이트한다.

이 과정을 수십억 개의 데이터 쌍에 대해 수백만 번 반복함으로써, 모델은 점차 텍스트 조건에 부합하는 방향으로 노이즈를 예측하는 능력을 학습하게 된다.


Stable Diffusion은 텍스트 프롬프트로부터 이미지를 생성하는 기본적인 기능을 넘어, 기존 이미지를 변형하고 편집하는 강력한 도구들을 제공한다. 이러한 기능들은 모두 확산 모델의 유연한 프레임워크 내에서 구현된다.


- **Text-to-Image (txt2img):** Stable Diffusion의 가장 기본적인 기능으로, 순수한 가우시안 노이즈에서 시작하여 이미지를 생성한다. 과정은 다음과 같다.
  1. 잠재 공간에서 표준 정규 분포 $\mathcal{N}(0, \mathbf{I})$로부터 무작위 텐서 $z_T$를 샘플링한다.
  2. 사용자가 입력한 텍스트 프롬프트를 CLIP 텍스트 인코더를 통해 조건 임베딩 $c$로 변환한다.
  3. 총 $T$번의 스텝에 걸쳐 역방향 확산 과정을 반복한다. 각 스텝 $t$에서 U-Net은 현재의 잠재 벡터 $z_t$, 시간 스텝 $t$, 그리고 조건 임베딩 $c$를 입력받아 노이즈 $\epsilon_\theta(z_t, t, c)$를 예측한다.
  4. 예측된 노이즈를 사용하여 $z_t$에서 노이즈를 제거하고 한 단계 더 깨끗한 잠재 벡터 $z_{t-1}$을 계산한다.
  5. 이 과정을 $t=T$에서 $t=1$까지 반복하여 최종적으로 노이즈가 완전히 제거된 잠재 벡터 $z_0$를 얻는다.
  6. 마지막으로, VAE 디코더가 $z_0$를 입력받아 최종 픽셀 이미지로 변환한다.11
- **Image-to-Image (img2img):** 기존 이미지를 기반으로 텍스트 프롬프트에 따라 새로운 이미지를 생성하는 기능이다.63 이는 단순한 스타일 변환을 넘어, 원본 이미지의 구성과 형태를 유지하면서 내용을 창의적으로 바꾸는 것을 가능하게 한다. 과정은 다음과 같다.63
  1. 사용자가 입력 이미지 $x_{input}$과 텍스트 프롬프트를 제공한다.
  2. VAE 인코더를 사용하여 $x_{input}$을 잠재 벡터 $z_{input}$으로 변환한다.
  3. **Denoising Strength** 파라미터에 의해 결정된 중간 시간 스텝 $t_{start}$까지 순방향 확산 과정을 적용하여, $z_{input}$에 일정량의 노이즈를 추가한 $z_{t_{start}}$를 생성한다.
  4. $z_{t_{start}}$에서부터 시작하여, 텍스트 프롬프트의 안내를 받으며 나머지 역방향 확산 과정을 수행하여 최종 잠재 벡터 $z_0$를 얻는다.
  5. VAE 디코더를 통해 $z_0$를 최종 이미지로 변환한다.
- **주요 파라미터 - Denoising Strength:** `img2img`에서 결과물의 성격을 결정하는 가장 중요한 파라미터다. 0과 1 사이의 값을 가지며, 원본 이미지에 얼마나 많은 노이즈를 추가할지, 즉 원본을 얼마나 변형시킬지를 제어한다.66
  - **값이 0에 가까울 경우:** 아주 적은 양의 노이즈만 추가되므로, 역방향 과정은 거의 원본 잠재 벡터에서 시작하게 된다. 결과적으로 원본 이미지의 구조와 색상이 대부분 유지되며 미세한 변화만 일어난다.
  - **값이 1에 가까울 경우:** 많은 양의 노이즈가 추가되어 원본 잠재 벡터의 정보가 거의 사라진다. 이는 사실상 `txt2img`와 유사하게 작동하며, 텍스트 프롬프트의 영향을 크게 받아 원본과는 매우 다른 새로운 이미지가 생성된다.
  - 일반적으로 0.5-0.8 사이의 값을 사용하여 원본 이미지의 구성을 유지하면서 텍스트 프롬프트에 따른 창의적인 변형을 가하는 데 사용된다.


Inpainting과 Outpainting은 `img2img` 원리를 특정 영역에 국한하여 적용하는 이미지 편집 기술이다.

- **Inpainting:** 이미지의 특정 부분을 자연스럽게 복원하거나 다른 내용으로 교체하는 기술이다.66 사용자가 마스크(mask)를 사용하여 편집하고자 하는 영역을 지정하면, 모델은 해당 영역을 주변 픽셀 및 텍스트 프롬프트와 의미론적으로 일관되게 채워 넣는다.66 이 기능은 손상된 사진 복원, 사진 속 불필요한 인물이나 객체 제거, 특정 부분의 의상이나 배경 교체 등 다양한 편집 작업에 활용된다.67
- **Outpainting:** 이미지의 캔버스를 원래 경계 너머로 확장하여, 보이지 않던 부분을 창의적으로 생성해내는 기술이다.67 모델은 기존 이미지의 맥락과 구성을 파악하여 확장된 영역을 자연스럽게 채워, 마치 더 넓은 화각으로 촬영한 듯한 효과를 준다.66
- **작동 원리:** Inpainting과 Outpainting은 모두 `img2img`의 변형된 형태로 작동한다.
  1. **마스킹:** 사용자는 Inpainting을 위해 이미지 내부의 특정 영역을, Outpainting을 위해 이미지 외부의 확장된 영역을 마스크로 지정한다.
  2. **초기화:** `Masked content` 설정에 따라 마스크된 영역을 처리한다. 'latent noise' 옵션은 해당 영역을 순수 노이즈로 채우고, 'original' 옵션은 원본 픽셀(Inpainting의 경우)에 노이즈를 추가하며, 'fill' 옵션은 주변 픽셀의 평균 색상으로 채운 후 노이즈를 추가한다.66
  3. **조건부 노이즈 제거:** 모델은 마스크되지 않은 원본 이미지 영역을 강력한 조건으로 사용하여, 마스크된 영역의 노이즈를 제거하는 역방향 확산 과정을 수행한다. 이 과정은 텍스트 프롬프트에 의해서도 추가적으로 안내된다.
  4. **블렌딩:** `Denoising strength` 파라미터는 생성된 부분과 원본 부분 사이의 조화를 제어한다. 이 값이 너무 높으면 경계가 부자연스러워질 수 있고, 너무 낮으면 프롬프트가 제대로 반영되지 않을 수 있다. 따라서 적절한 값을 찾는 것이 중요하다.66

이러한 기능들은 Stable Diffusion이 단순한 이미지 생성기를 넘어, 강력하고 유연한 디지털 아트 및 사진 편집 도구로 활용될 수 있는 가능성을 보여준다.


Stable Diffusion의 가장 큰 성공 요인 중 하나는 모델 가중치를 오픈소스로 공개하여 거대한 커뮤니티 기반의 생태계를 구축했다는 점이다. 이 생태계는 사전 훈련된 거대 모델의 핵심 지식을 파괴하지 않으면서도, 효율적으로 모델을 제어하고 개인화하는 혁신적인 기술들을 탄생시켰다. ControlNet, DreamBooth, LoRA는 이러한 철학을 대표하는 기술들로, Stable Diffusion을 단순한 모델에서 확장 가능한 플랫폼으로 변모시켰다.

이러한 기술들의 등장은 거대 파운데이션 모델을 다루는 방식에 근본적인 패러다임 전환을 의미한다. 과거에는 모델을 특정 목적에 맞게 수정하려면 비용이 많이 들고 '파국적 망각(catastrophic forgetting)'의 위험이 있는 전체 모델 재훈련(full model retraining) 방식에 의존해야 했다.70 그러나 Stable Diffusion 커뮤니티는 **모듈식, 파라미터 효율적, 비파괴적 적응(modular, parameter-efficient, and non-destructive adaptation)**이라는 새로운 철학을 개척했다. 이는 특정 스타일을 위한 LoRA, 구조 제어를 위한 ControlNet, 특정 대상을 위한 DreamBooth 모델과 같이, 단일한 기반 모델에 동적으로 결합하고 적용할 수 있는 거대한 "플러그인" 라이브러리를 만드는 것을 가능하게 했다. 이 모듈식 접근법은 모델을 '완성된 제품'이 아닌 '플랫폼'으로 전환시켰고, 사용자들이 다양한 어댑터들을 조합하여 창의성을 폭발시키는 조합적 혁신을 이끌어냈다.


ControlNet은 텍스트 프롬프트만으로는 달성하기 어려운 정밀한 공간적 제어를 가능하게 함으로써, Stable Diffusion의 창의적 활용 범위를 극적으로 확장시켰다.

- **원리:** ControlNet은 사전 훈련된 대규모 확산 모델에 윤곽선, 깊이, 인간의 자세 등 추가적인 공간적 조건(spatial condition)을 부여하여 이미지 생성을 정밀하게 제어하는 신경망 아키텍처다.72

- **이중 구조 (Dual Architecture):** ControlNet의 핵심 아이디어는 기존 Stable Diffusion U-Net의 학습 가능한 가중치를 그대로 복제하여 두 개의 복사본을 만드는 것이다. 하나는 원본 가중치를 변경하지 않고 그대로 사용하는 '잠긴(locked)' 복사본이고, 다른 하나는 새로운 공간적 조건을 학습하기 위한 '학습 가능한(trainable)' 복사본이다.70 이 구조는 수십억 개의 이미지로 사전 훈련된 U-Net의 강력한 특징 추출 능력을 손상 없이 그대로 활용하면서, 새로운 제어 능력을 안전하게 추가할 수 있게 한다.75

- **Zero Convolution:** '잠긴' 블록의 출력과 '학습 가능한' 블록의 출력은 '제로 컨볼루션(zero convolution)'이라는 특별한 레이어를 통해 합쳐진다. 이는 가중치와 편향이 모두 0으로 초기화된 1x1 컨볼루션 레이어로, 훈련 초기 단계에서는 학습 가능한 복사본이 전체 모델에 아무런 영향을 미치지 않도록 보장하는 안전장치 역할을 한다.70 훈련이 진행됨에 따라 이 레이어의 가중치가 점진적으로 0이 아닌 값으로 학습되면서, 원본 모델의 안정성을 해치지 않고 제어 신호를 부드럽게 주입할 수 있게 된다.76

- **주요 모델 및 활용 사례:** ControlNet은 다양한 유형의 공간적 조건을 처리하기 위해 특화된 여러 모델을 제공한다. 각 모델은 특정 전처리기(preprocessor)를 사용하여 입력 이미지로부터 제어 맵(control map)을 추출하고, 이를 U-Net에 조건으로 전달한다.74

  **Table 1: 주요 ControlNet 모델 및 전처리기 요약**

| ControlNet 모델  | 조건 입력 (전처리기)             | 설명                                                         | 주요 활용 사례                                               |
| ---------------- | -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Canny**        | Canny Edge                       | 이미지의 단단한 윤곽선을 추출한다.                           | 원본 이미지의 구도, 형태, 구성을 정확하게 복제               |
| **Depth**        | Depth Map (Midas, Leres, Zoe 등) | 이미지의 픽셀별 깊이 정보를 추정한다.                        | 3D 공간감, 원근, 객체 간의 거리를 유지하며 스타일 변경       |
| **OpenPose**     | Human Pose Keypoints             | 인체의 주요 관절(머리, 어깨, 팔, 다리 등) 위치를 감지한다.   | 원본 이미지의 인물 포즈를 그대로 따라 하는 새로운 캐릭터 생성 |
| **Scribble/HED** | Hand-drawn Scribble / Soft Edge  | 사용자가 그린 스케치나 이미지의 부드러운 외곽선을 조건으로 사용한다. | 스케치를 사실적인 이미지로 변환, 이미지 스타일 변경 및 재채색 |
| **Segmentation** | Semantic Segmentation Map        | 이미지를 의미론적 단위(하늘, 나무, 건물 등)로 분할하고 색상으로 구분한다. | 이미지의 전체적인 레이아웃과 객체 배치를 유지하며 테마 변경  |
| **MLSD**         | Straight Lines                   | 건축물이나 인공물에서 직선을 감지한다.                       | 실내 디자인, 건축물 렌더링 등 직선 구조가 중요한 이미지 생성 |

이러한 다양한 모델들을 통해 사용자는 텍스트 프롬프트만으로는 불가능했던, 전례 없는 수준의 정밀한 제어력을 바탕으로 창의적인 이미지를 생성할 수 있다.[74, 75, 81, 82, 83]


대규모 모델을 모든 사용자가 자신의 필요에 맞게 재훈련하는 것은 비현실적이다. DreamBooth와 LoRA는 이러한 문제를 해결하기 위한 효율적인 개인화 및 미세 조정 기법이다.

- **DreamBooth:** 단 몇 장의 이미지(보통 3~5장)만으로 특정 인물, 사물, 스타일과 같은 새로운 개념을 모델에 주입하는 강력한 미세 조정(fine-tuning) 기술이다.71

  - **고유 식별자 (Unique Identifier):** DreamBooth의 핵심은 학습시키려는 대상에 모델의 기존 어휘와 충돌하지 않는 희귀한 토큰을 고유 식별자로 부여하는 것이다. 예를 들어, 특정 강아지를 학습시키기 위해 "a photo of [V] dog"와 같은 프롬프트를 사용한다. 여기서 `[V]`는 그 강아지만을 지칭하는 고유한 이름표 역할을 한다.71
  - **클래스 사전 보존 손실 (Class-specific Prior Preservation Loss):** 소수의 이미지로 미세 조정을 하면 모델이 해당 이미지에 과적합(overfitting)되어 일반적인 개념(예: '개'라는 개념 전체)을 잊어버리는 '파국적 망각'이 발생할 수 있다. 이를 방지하기 위해 DreamBooth는 '사전 보존 손실'이라는 기법을 사용한다. 이는 학습 과정에서 모델 스스로 "a photo of a dog"와 같은 일반적인 클래스 프롬프트로 이미지를 생성하고, 이 생성된 이미지들을 소수의 특정 대상 이미지와 함께 학습시키는 방식이다. 이를 통해 모델은 새로운 특정 개체(`[V] dog`)를 배우면서도, 일반적인 '개'에 대한 폭넓은 이해를 유지할 수 있다.71

- **LoRA (Low-Rank Adaptation):** 전체 모델의 수십억 개 파라미터를 모두 업데이트하는 대신, 극소수의 파라미터만 추가하여 효율적으로 미세 조정하는 기법(Parameter-Efficient Fine-Tuning, PEFT)의 일종이다.87

  - **원리:** LoRA는 대규모 신경망의 가중치 행렬을 미세 조정할 때, 가중치의 '변화량'($\Delta W$)이 본질적으로 낮은 랭크(low-rank)를 가진다는 관찰에 기반한다. 따라서 $\Delta W$를 직접 학습하는 대신, 이를 두 개의 작은 저차원 행렬($A$와 $B$)의 곱, 즉 $\Delta W = BA$로 근사한다. 여기서 랭크 $r$은 원래 행렬의 차원보다 훨씬 작기 때문에, 학습해야 할 파라미터의 수가 `d x k`에서 `(d+k) x r`로 극적으로 감소한다.88

  - **효과:** LoRA는 사전 훈련된 모델의 원래 가중치는 고정한 채, U-Net의 어텐션 레이어 등에 주입된 작은 $A$와 $B$ 행렬만 학습한다. 이 방식은 학습 가능한 파라미터 수를 최대 10,000배, GPU 메모리 요구량을 3배까지 줄일 수 있다.88 훈련이 완료된 후, 학습된 LoRA 모듈(수십 MB 크기)만 저장하면 되므로 저장 공간 효율성도 매우 높다. 추론 시에는 학습된 

    $\Delta W = BA$를 원래 가중치 $W$에 더해 $W' = W + \Delta W$로 병합할 수 있어, 추가적인 계산 지연 없이 미세 조정된 모델의 성능을 발휘할 수 있다.88 이 덕분에 사용자들은 다양한 스타일이나 캐릭터에 대한 LoRA 파일을 손쉽게 공유하고, 필요에 따라 동적으로 적용할 수 있다.


Stable Diffusion은 2022년 첫 출시 이후, 지속적인 업데이트를 통해 성능과 기능을 발전시켜왔다. 각 주요 버전은 아키텍처, 훈련 데이터, 그리고 생성 능력 측면에서 중요한 변화를 보여준다. 특히 SDXL로의 발전은 단순한 성능 향상을 넘어, 모델 설계 철학의 근본적인 전환을 의미한다.

초기 버전인 v1.x에서 v2.x로의 변화는 주로 텍스트 인코더를 교체하는 점진적 개선에 가까웠다. 그러나 이는 커뮤니티 자산과의 호환성 문제를 야기하며, 더 큰 발전을 위해서는 단순한 부품 교체 이상의 접근이 필요함을 시사했다. 이에 대한 해답으로 등장한 SDXL은, 단일 모델을 단순히 확장하는 대신, 여러 전문화된 구성 요소가 협력하는 '전문가 앙상블(ensemble of experts)' 파이프라인이라는 새로운 설계 철학을 도입했다. 이는 텍스트 이해, 전체 구도 생성, 세부 묘사 등 각기 다른 하위 작업을 더 작은 전문 모듈이 분담하여 처리하는 것이 단일 거대 모델보다 우수하다는 인식의 전환을 보여준다. 구체적으로, 두 개의 다른 텍스트 인코더를 결합하여 프롬프트 이해도를 극대화하고, 전체 구도를 잡는 '기반 모델'과 세부 묘사를 전담하는 '정제 모델'로 생성 과정을 분리한 것은 이러한 '분할 정복(divide and conquer)' 전략의 명백한 증거다. 이는 생성 모델 설계가 성숙기에 접어들었음을 보여주는 중요한 이정표라 할 수 있다.

**Table 2: Stable Diffusion 버전별 주요 특징 비교**

| 기능 (Feature)                    | Stable Diffusion 1.x (v1.4/1.5) | Stable Diffusion 2.x (v2.0/2.1)                              | Stable Diffusion XL (SDXL)                                   |
| --------------------------------- | ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **기본 해상도 (Base Resolution)** | 512x512                         | 512x512 / 768x768                                            | 1024x1024                                                    |
| **U-Net 파라미터 수**             | ~860M                           | ~865M                                                        | ~2.6B                                                        |
| **텍스트 인코더 (Text Encoder)**  | CLIP ViT-L                      | OpenCLIP ViT-H                                               | **이중 인코더 (Dual Encoder):** OpenCLIP ViT-bigG + CLIP ViT-L |
| **컨텍스트 차원 (Context Dim.)**  | 768                             | 1024                                                         | 2048                                                         |
| **주요 구조적 변화**              | - LDM 아키텍처의 기반 확립      | - 더 강력한 OpenCLIP 텍스트 인코더로 교체 - 부정적 프롬프트 기능 개선 | - 3배 더 커진 U-Net 백본 - 이중 텍스트 인코더를 통한 프롬프트 이해력 향상 - **기반 모델(Base) + 정제 모델(Refiner)의 2단계 파이프라인** - 다양한 종횡비(aspect ratio) 훈련 |

- **v1.x에서 v2.x로의 변화:** Stable Diffusion v2.0/2.1의 가장 큰 변화는 텍스트 인코더를 기존의 CLIP ViT-L에서 더 크고 강력한 OpenCLIP ViT-H로 교체한 것이다.89 이는 이론적으로 더 나은 프롬프트 해석 능력을 제공했지만, v1.x 기반으로 구축된 방대한 커뮤니티의 미세 조정 모델, LoRA, 텍스트 임베딩 등과의 호환성 문제를 야기했다. 이로 인해 많은 사용자들이 v1.5 버전을 계속해서 선호하는 현상이 나타났다. 또한, 기본 생성 해상도에 768x768 옵션이 추가되어 더 높은 해상도의 이미지를 직접 생성할 수 있게 되었다.11
- **SDXL: 구조적 혁신:** SDXL(Stable Diffusion XL)은 이전 버전들과 비교하여 단순한 파라미터 증가를 넘어선 근본적인 구조적 혁신을 이루었다.89
  - **거대화된 U-Net:** U-Net 백본의 파라미터 수가 약 8.6억 개에서 26억 개로 3배 가까이 증가했다. 이는 더 많은 어텐션 블록과 더 큰 교차 어텐션 컨텍스트를 통해 달성되었으며, 모델의 전반적인 표현력과 이미지 품질을 크게 향상시켰다.89
  - **이중 텍스트 인코더:** SDXL의 가장 큰 특징은 두 개의 서로 다른 텍스트 인코더, 즉 OpenCLIP ViT-bigG와 CLIP ViT-L을 함께 사용한다는 점이다.89 두 인코더에서 나온 임베딩을 결합하여 U-Net에 전달함으로써, 모델은 훨씬 더 풍부하고 미묘한 의미 정보를 조건으로 활용할 수 있게 되었다. 이는 복잡하고 긴 프롬프트에 대한 이해도를 극적으로 높여, 이전 모델들보다 사용자의 의도를 더 정확하게 반영하는 결과를 낳는다.
  - **2단계 파이프라인 (Base + Refiner):** SDXL은 고품질 이미지를 생성하기 위해 두 개의 모델로 구성된 파이프라인을 채택했다. 먼저, **기반(Base) 모델**이 텍스트 프롬프트를 받아 전체적인 구도와 형태를 갖춘 잠재 벡터를 생성한다. 그 다음, 이 잠재 벡터를 **정제(Refiner) 모델**에 전달하여 최종 노이즈 제거 단계를 수행한다. 정제 모델은 이미지의 세부 묘사, 질감, 고주파 디테일을 향상시키는 데 특화되어 있다.89 이 분업 구조는 1024x1024의 고해상도에서 전체적인 일관성과 세부적인 디테일을 모두 효과적으로 달성하기 위한 전략이다.
  - **다양한 종횡비 훈련:** 이전 모델들이 주로 512x512 정사각형 이미지로 훈련된 것과 달리, SDXL은 다양한 종횡비의 이미지로 훈련되어 특정 해상도에 대한 편향이 줄어들고, 다양한 크기의 이미지를 더 안정적으로 생성할 수 있게 되었다.11

이러한 발전 과정을 통해 Stable Diffusion은 단순한 이미지 생성 도구를 넘어, 더욱 정교하고, 사실적이며, 사용자의 의도를 깊이 이해하는 강력한 창작 플랫폼으로 진화하고 있다.


Stable Diffusion은 이미지 생성 분야에 혁명을 일으켰지만, 동시에 명확한 기술적 한계와 심각한 사회적, 윤리적 과제를 안고 있다. 이러한 문제들은 기술의 성숙과 사회적 수용을 위해 반드시 해결해야 할 중요한 과제다.


모델의 아키텍처와 훈련 데이터의 특성에서 비롯되는 몇 가지 고질적인 생성 실패 사례가 존재한다.

- **손과 텍스트 생성의 어려움:** Stable Diffusion은 특히 인간의 손가락이나 이미지 내의 텍스트를 정확하게 생성하는 데 큰 어려움을 겪는다.94 생성된 손은 종종 손가락이 6개 이상이거나, 비정상적인 형태로 뒤틀려 있으며, 텍스트는 의미를 알 수 없는 왜곡된 문자열로 나타나는 경우가 많다.
- **기술적 원인:** 이러한 문제의 원인은 복합적이다.
  1. **데이터 표현의 문제:** 훈련 데이터셋에서 손은 전체 이미지 중 매우 작은 부분을 차지하며, 주먹을 쥐거나, 물건을 잡거나, 주머니에 가려지는 등 매우 다양한 형태로 나타난다. 이처럼 일관되지 않고 부분적인 데이터는 모델이 손의 일관된 구조를 학습하는 것을 방해한다.94
  2. **3차원 구조에 대한 이해 부족:** Stable Diffusion은 2D 이미지만으로 학습하기 때문에, 여러 관절로 이루어진 손의 복잡한 3차원 구조와 기하학적 제약(예: 손가락이 구부러지는 방식)을 근본적으로 이해하지 못한다.97 모델은 단지 2D 이미지에 나타난 픽셀 패턴을 통계적으로 모방할 뿐이다.
  3. **잠재 공간의 정보 손실:** VAE를 통해 이미지를 저차원의 잠재 공간으로 압축하는 과정에서, 텍스트와 같이 매우 정밀하고 고주파적인 디테일 정보가 일부 손실될 수 있다. 이로 인해 디코딩 단계에서 정확한 문자 형태를 복원하기 어려워진다.95
  4. **토큰화의 한계:** CLIP 텍스트 인코더는 텍스트를 단어나 하위 단어(sub-word) 단위의 토큰으로 처리한다. 이는 문장의 전체적인 의미를 파악하는 데는 효율적이지만, 'A'와 'B' 같은 개별 글자(character)의 시각적 형태와 정확한 순서를 학습하는 데는 적합하지 않다. 모델은 'text'라는 단어를 시각적 패턴으로 인식할 뿐, 그것이 't', 'e', 'x', 't'라는 네 개의 글자로 구성된다는 사실을 알지 못한다.95


모델의 훈련 데이터와 생성 능력은 심각한 윤리적, 법적 문제를 야기하며, 이는 기술의 사회적 책임에 대한 근본적인 질문을 던진다.

- **데이터 편향과 스테레오타입 강화:** LAION과 같은 대규모 웹 스크랩 데이터셋은 현실 세계에 존재하는 인종, 성별, 직업 등에 대한 사회적 편견과 고정관념을 그대로 반영하고 있다. 모델은 이러한 편향된 데이터를 학습하여, 예를 들어 '의사'라는 프롬프트에 주로 백인 남성을, '간호사'에는 여성을 생성하는 등 사회적 스테레오타입을 강화하고 재생산하는 경향을 보인다.52
- **유해 콘텐츠 및 딥페이크:** 훈련 데이터에 포함된 아동 성적 학대물(CSAM)과 같은 유해 콘텐츠는 그 자체로 심각한 윤리적 문제이며, 모델이 이러한 유해한 특징을 학습할 위험을 내포한다.56 또한, 생성 모델은 악의적인 목적으로 사용될 경우, 특정 인물의 얼굴을 합성한 딥페이크 음란물 제작, 가짜 뉴스 유포, 명예훼손, 금융 사기 등 심각한 사회적 혼란과 개인에게 돌이킬 수 없는 피해를 줄 수 있다.52
- **저작권 침해 논란 (Getty Images v. Stability AI):** 생성 AI의 법적 지위를 결정할 가장 중요한 사건 중 하나는 이미지 제공업체인 Getty Images와 Stability AI 간의 소송이다.
  - **핵심 쟁점:** Getty Images는 Stability AI가 자사의 라이선스 정책을 위반하여 수백만 개의 저작권 보호 이미지를 무단으로 스크랩하여 Stable Diffusion 모델을 훈련시켰다고 주장한다. 그 증거로, 생성된 일부 이미지에서 Getty Images의 워터마크가 왜곡된 형태로 나타나는 현상을 제시했다. 이는 모델이 단순히 데이터로부터 '학습'한 것을 넘어, 원본 이미지를 실질적으로 '복제'하고 있다는 주장으로 이어진다.57
  - **Stability AI의 방어 논리:** Stability AI는 모델 훈련이 영국 법의 관할권 밖에서 이루어졌다고 주장하며, 설령 저작권 침해가 인정되더라도 이는 새로운 창작물을 만드는 '변형적 사용(transformative use)'에 해당하므로, 미국 저작권법의 '공정 이용(fair use)' 또는 영국법의 '공정 취급(fair dealing)' 예외 조항에 따라 보호받아야 한다고 반박한다.104 즉, AI가 데이터를 학습하는 것은 인간 예술가가 영감을 얻기 위해 다른 작품을 참고하는 것과 유사하다는 입장이다.
  - **영향:** 이 소송의 판결은 AI 모델 훈련을 위한 데이터 사용에 대한 법적 기준을 정립하는 데 중대한 영향을 미칠 것이다. 만약 법원이 Getty의 손을 들어준다면, AI 개발사들은 데이터 사용에 대한 라이선스 비용을 지불해야 할 의무가 생겨 생성 AI 산업의 비즈니스 모델에 근본적인 변화를 가져올 수 있다. 반대로 Stability AI가 승소한다면, 공정 이용의 범위가 AI 학습까지 확장되는 선례가 될 것이다.104


생성 AI 콘텐츠의 확산에 따른 혼란과 오용을 막기 위해, 콘텐츠의 출처와 신뢰성을 확보하려는 기술적, 표준화 노력이 진행되고 있다.

- **디지털 워터마킹:** AI가 생성한 콘텐츠임을 식별할 수 있도록, 인간의 눈에는 보이지 않는 고유한 신호를 이미지 데이터에 삽입하는 기술이다. 이 워터마크는 이미지 압축, 크기 조정 등 일반적인 편집 과정에서도 유지되도록 설계되며, 전용 탐지 알고리즘을 통해 해당 이미지가 특정 모델에 의해 생성되었음을 확인할 수 있다. 이는 AI 생성 콘텐츠의 출처를 추적하고, 허위 정보의 확산을 억제하는 데 도움을 줄 수 있다.107
- **C2PA (Coalition for Content Provenance and Authenticity):** Adobe, Microsoft, Intel, BBC 등 주요 기술 및 미디어 기업들이 참여하여 설립한 연합으로, 디지털 콘텐츠의 출처와 변경 이력을 투명하게 기록하고 검증하기 위한 개방형 기술 표준을 개발하고 있다.108
  - **콘텐츠 자격증명 (Content Credentials):** C2PA 표준의 핵심은 '콘텐츠 자격증명'이라는, 일종의 '디지털 영양성분표'를 미디어 파일에 첨부하는 것이다. 이 자격증명에는 암호화 서명을 통해 위변조가 방지된 메타데이터가 포함되며, 해당 콘텐츠가 언제, 어떤 도구(예: 'Stable Diffusion', 'Adobe Photoshop')로 생성되고 편집되었는지에 대한 상세한 이력을 담고 있다.109 사용자는 이 정보를 통해 콘텐츠의 신뢰성을 스스로 판단할 수 있다. 이는 AI 생성 콘텐츠의 투명성을 높이고, 딥페이크와 같은 조작된 정보에 사회가 대응할 수 있는 중요한 기술적 기반을 제공한다.



Stable Diffusion은 Latent Diffusion Model(LDM)이라는 혁신적인 아키텍처를 통해 고품질 이미지 생성을 기술 전문가의 영역에서 일반 대중의 영역으로 끌어내렸다. 계산 효율성을 극대화한 이 설계는 기술의 "민주화"를 이끌었고, 모델과 코드를 오픈소스로 공개한 결정은 전 세계적인 개발자 커뮤니티의 폭발적인 참여를 유도했다. 그 결과, ControlNet, LoRA, DreamBooth와 같은 수많은 확장 기술이 탄생하며 강력한 생태계가 구축되었다. Stable Diffusion은 단순히 뛰어난 이미지 생성 도구를 넘어, 창작, 디자인, 연구, 엔터테인먼트 등 다양한 분야에서 새로운 가능성을 열고 패러다임의 전환을 주도하는 기술적 촉매제가 되었다.111


Stable Diffusion의 성공에도 불구하고, 기술적 한계와 사회적 과제를 해결하기 위한 연구는 여전히 활발하게 진행 중이며, 이는 미래 생성 AI의 발전 방향을 제시한다.

- **품질 및 효율성 향상:** 현재의 기술을 넘어서는 더 높은 품질과 효율성을 추구하는 연구가 계속되고 있다. 여기에는 4K 이상의 초고해상도 이미지를 업스케일링 없이 직접 생성하는 기술 112, 더 적은 훈련 시간과 추론 스텝으로도 높은 품질을 달성하여 수렴 속도를 높이는 연구 114, 그리고 고전적인 컴퓨팅의 한계를 뛰어넘기 위해 양자 컴퓨팅 원리를 확산 모델에 접목하려는 시도 등이 포함된다.116
- **일관성 및 제어력 강화:** 생성 모델의 근본적인 한계 중 하나는 결과물의 무작위성과 비일관성이다. 이를 해결하기 위해, 입력 노이즈나 조건의 미세한 변화에도 시각적으로 일관된 출력을 보장하는 Alias-Free LDM과 같은 연구가 진행되고 있다.117 이는 비디오 생성이나 일관된 캐릭터 디자인과 같이 연속성이 중요한 응용 분야에서 핵심적인 기술이 될 것이다.
- **새로운 응용 분야 개척:** Stable Diffusion의 성공은 이미지 생성을 넘어 다른 과학 및 공학 분야로 확산 모델의 적용 가능성을 열었다. 예를 들어, 유체 역학이나 기상 예측과 같은 복잡한 동역학계(dynamical systems)를 시뮬레이션하는 '물리 에뮬레이터'로서 확산 모델을 활용하려는 연구는, 막대한 계산 비용이 드는 기존 시뮬레이션을 대체할 수 있는 잠재력을 보여준다.118


Stable Diffusion으로 대표되는 생성 AI 기술은 앞으로도 폭발적인 발전을 거듭할 것이다. 기술적으로는 더 높은 해상도, 더 빠른 속도, 더 정교한 제어 능력을 갖추게 될 것이며, 텍스트와 이미지를 넘어 비디오, 3D, 오디오 등 다양한 모달리티로 그 영역을 확장해 나갈 것이다.

그러나 기술의 발전만큼이나 중요한 것은 사회적, 윤리적, 법적 제도와의 조화로운 통합이다. 훈련 데이터에 내재된 편향성을 완화하고 공정성을 확보하는 문제, 저작권이 있는 데이터의 학습과 사용에 대한 합리적인 법적, 경제적 합의점을 찾는 문제, 그리고 C2PA와 같은 콘텐츠 출처 증명 기술을 보편화하여 AI 생성 콘텐츠의 투명성과 신뢰성을 확보하는 문제는 기술의 지속 가능한 발전을 위한 핵심 과제가 될 것이다. 미래의 생성 AI는 단순히 더 뛰어난 결과물을 만드는 것을 넘어, 인간의 창의성을 증강시키고 사회와 책임감 있게 상호작용하는 방향으로 진화해야 할 것이다.


1. High-Resolution Image Synthesis With Latent ... - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2022/papers/Rombach_High-Resolution_Image_Synthesis_With_Latent_Diffusion_Models_CVPR_2022_paper.pdf
2. “High-Resolution Image Synthesis with Latent Diffusion Models” Summary | by David Min, 8월 24, 2025에 액세스, https://medium.com/@dminhk/high-resolution-image-synthesis-with-latent-diffusion-models-summary-93e8f01d2ba4
3. The Power of Diffusion Models in AI: A Comprehensive Guide - Kanerika, 8월 24, 2025에 액세스, https://kanerika.com/blogs/diffusion-models/
4. Denoising Diffusion Probabilistic Models, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2006.11239
5. LDMs: High-Resolution Image Synthesis with Latent Diffusion Models, a 5-minute paper summary by Casual GAN Papers : r/MachineLearning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/uhsfde/d_democratizing_diffusion_models_ldms/
6. Improved Denoising Diffusion Probabilistic Models - arXiv, 8월 24, 2025에 액세스, http://arxiv.org/pdf/2102.09672
7. Paper page - High-Resolution Image Synthesis with Latent Diffusion Models - Hugging Face, 8월 24, 2025에 액세스, https://huggingface.co/papers/2112.10752
8. Stable Diffusion Explained - by Onkar Mishra - Medium, 8월 24, 2025에 액세스, https://medium.com/@onkarmishra/stable-diffusion-explained-1f101284484d
9. Understanding Stable Diffusion Architecture and UNet | by Yash Jain | Medium, 8월 24, 2025에 액세스, https://medium.com/@ydhupiya1710/understanding-stable-diffusion-architecture-and-unet-4aad410929c4
10. Everything You Need To Know About Stable Diffusion - Hyperstack, 8월 24, 2025에 액세스, https://www.hyperstack.cloud/blog/case-study/everything-you-need-to-know-about-stable-diffusion
11. Stable Diffusion - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Stable_Diffusion
12. Introduction to Diffusion Models for Machine Learning | SuperAnnotate, 8월 24, 2025에 액세스, https://www.superannotate.com/blog/diffusion-models
13. Diffusion model - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Diffusion_model
14. What is Forward Process Stable diffusion? - Analytics Vidhya, 8월 24, 2025에 액세스, https://www.analyticsvidhya.com/blog/2024/07/forward-process-stable-diffusion/
15. How diffusion models work: the math from scratch | AI Summer, 8월 24, 2025에 액세스, https://theaisummer.com/diffusion-models/
16. What is the Reverse Diffusion Process? - Analytics Vidhya, 8월 24, 2025에 액세스, https://www.analyticsvidhya.com/blog/2024/07/reverse-diffusion-process/
17. An Introduction to Diffusion Models and Stable Diffusion - Marvik - Blog, 8월 24, 2025에 액세스, https://blog.marvik.ai/2023/11/28/an-introduction-to-diffusion-models-and-stable-diffusion/
18. What loss functions are typically used when training diffusion models? - Milvus, 8월 24, 2025에 액세스, https://milvus.io/ai-quick-reference/what-loss-functions-are-typically-used-when-training-diffusion-models
19. Stable Diffusion: Transforming Text into Visual Wonders - Indika AI, 8월 24, 2025에 액세스, https://www.indikaai.com/blog/magic-of-stable-diffusion
20. This is what Stable Diffusion's attention looks like : r/StableDiffusion - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/StableDiffusion/comments/1ig6iq7/this_is_what_stable_diffusions_attention_looks/
21. U-Net Architecture Explained - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/u-net-architecture-explained/
22. Stable Diffusion Explained: How Text Prompts Become Stunning Images. | by Patel chintan, 8월 24, 2025에 액세스, https://medium.com/@patelchintan732000123/a-comprehensive-look-at-text-encoders-vq-gans-and-u-nets-3b4687bc8b58
23. CLIP: Connecting text and images - OpenAI, 8월 24, 2025에 액세스, https://openai.com/index/clip/
24. Stable Diffusion - tglcourse, 8월 24, 2025에 액세스, https://johnowhitaker.github.io/tglcourse/dm3.html
25. Variational Autoencoder - Civitai Wiki, 8월 24, 2025에 액세스, https://wiki.civitai.com/wiki/Variational_Autoencoder
26. Building a Stable Diffusion Model from Scratch with PyTorch(Variational Auto Encoders): Part 2 | by Ninad Shukla | Medium, 8월 24, 2025에 액세스, https://medium.com/@ninads79shukla/building-a-stable-diffusion-model-from-scratch-with-pytorch-variational-auto-encoders-part-2-d0c9d359f280
27. DSPFusion: Degradation and Semantic Prior Dual-guided Framework for Image Fusion, 8월 24, 2025에 액세스, https://openreview.net/forum?id=ittdt7tKND
28. Tutorial - What is a variational autoencoder? - Jaan Lı 李, 8월 24, 2025에 액세스, https://jaan.io/what-is-variational-autoencoder-vae-tutorial/
29. What is Variational Autoencoders ? - Analytics Vidhya, 8월 24, 2025에 액세스, https://www.analyticsvidhya.com/blog/2023/07/an-overview-of-variational-autoencoders/
30. Variational Autoencoders: How They Work and Why They Matter | DataCamp, 8월 24, 2025에 액세스, https://www.datacamp.com/tutorial/variational-autoencoders
31. What is a Variational Autoencoder? - IBM, 8월 24, 2025에 액세스, https://www.ibm.com/think/topics/variational-autoencoder
32. Variational AutoEncoders - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/variational-autoencoders/
33. What is VAE and How to Use It in Stable Diffusion - Aiarty Image Enhancer, 8월 24, 2025에 액세스, https://www.aiarty.com/stable-diffusion-guide/how-to-use-vae-in-stable-diffusion.htm
34. What Is VAE in Stable Diffusion? - Built In, 8월 24, 2025에 액세스, https://builtin.com/artificial-intelligence/stable-diffusion-vae
35. U-Net for Stable Diffusion - labml.ai, 8월 24, 2025에 액세스, https://nn.labml.ai/diffusion/stable_diffusion/model/unet.html
36. Training-Free Style and Content Transfer by Leveraging U-Net Skip Connections in Stable Diffusion 2.* - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2501.14524v1
37. Training-Free Style and Content Transfer by Leveraging U-Net Skip Connections in Stable Diffusion 2. - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/388402818_Training-Free_Style_and_Content_Transfer_by_Leveraging_U-Net_Skip_Connections_in_Stable_Diffusion_2
38. [2501.14524] Training-Free Style and Content Transfer by Leveraging U-Net Skip Connections in Stable Diffusion - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2501.14524
39. Towards Understanding Cross and Self-Attention in Stable Diffusion for Text-Guided Image Editing - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2403.03431v1
40. [D] Am I the only one that thinks this behavior (cross-attention layers) is odd? - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/13rwh1c/d_am_i_the_only_one_that_thinks_this_behavior/
41. Transformer for Stable Diffusion U-Net - labml.ai, 8월 24, 2025에 액세스, https://nn.labml.ai/diffusion/stable_diffusion/model/unet_attention.html
42. The CLIP Model is Secretly an Image-to-Prompt Converter - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2305.12716v2
43. The CLIP Model is Secretly an Image-to-Prompt Converter, 8월 24, 2025에 액세스, https://neurips.cc/media/neurips-2023/Slides/70603.pdf
44. Stable Diffusion Model Architecture - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=QC9yun0HbNE
45. Cross Attention | Method Explanation | Math Explained - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=aw3H-wPuRcw
46. Exploring the training data behind Stable Diffusion - Simon Willison's Weblog, 8월 24, 2025에 액세스, https://simonwillison.net/2022/Sep/5/laion-aesthetics-weeknotes/
47. Stable Diffusion pipelines - Hugging Face, 8월 24, 2025에 액세스, https://huggingface.co/docs/diffusers/v0.26.3/en/api/pipelines/stable_diffusion/overview
48. LAION, 8월 24, 2025에 액세스, https://laion.ai/
49. LAION-5B: An open large-scale dataset for training next generation image-text models, 8월 24, 2025에 액세스, https://openreview.net/forum?id=M3Y74vmsMcY
50. LAION-5B: A NEW ERA OF OPEN LARGE-SCALE MULTI-MODAL DATASETS, 8월 24, 2025에 액세스, https://laion.ai/laion-5b-a-new-era-of-open-large-scale-multi-modal-datasets/
51. LAION-Aesthetics | LAION, 8월 24, 2025에 액세스, https://laion.ai/blog/laion-aesthetics/
52. Understanding the Ethics of Generative AI: Risks, Concerns, and Best Practices | DataCamp, 8월 24, 2025에 액세스, https://www.datacamp.com/tutorial/ethics-in-generative-ai
53. Examining biases in LAION-5B, Stable Diffusion XL (SDXL), and our... - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/Examining-biases-in-LAION-5B-Stable-Diffusion-XL-SDXL-and-our-SDXL-Inc-Comparing_fig1_391160800
54. Training Unbiased Diffusion Models From Biased Dataset - OpenReview, 8월 24, 2025에 액세스, https://openreview.net/forum?id=39cPKijBed
55. The Dark Side of Dataset Scaling: Evaluating Racial Classification in Multimodal Models, 8월 24, 2025에 액세스, https://arxiv.org/html/2405.04623v1
56. LAION-5B, Stable Diffusion 1.5, and the Original Sin of Generative AI | TechPolicy.Press, 8월 24, 2025에 액세스, https://www.techpolicy.press/laion5b-stable-diffusion-and-the-original-sin-of-generative-ai/
57. Generative AI in the courts - Getty Images v Stability AI, 8월 24, 2025에 액세스, https://www.penningtonslaw.com/news-publications/latest-news/2024/generative-ai-in-the-courts-getty-images-v-stability-ai
58. IN THE UNITED STATES DISTRICT COURT FOR THE DISTRICT OF DELAWARE GETTY IMAGES (US), INC. Plaintiff, v. STABILITY AI, INC. Defen - Copyright Alliance, 8월 24, 2025에 액세스, https://copyrightalliance.org/wp-content/uploads/2023/02/Getty-Images-v.-Stability-AI-Complaint.pdf
59. Diffusion Model with Perceptual Loss - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2401.00110v3
60. Stable diffusion models - Intuitive Tutorials, 8월 24, 2025에 액세스, https://intuitivetutorial.com/2024/05/04/stable-diffusion-models-in-deep-learning/
61. Diffusion Pipeline: Core Processes Unveiled & Practical Application Guide - WhaleFlux, 8월 24, 2025에 액세스, https://www.whaleflux.com/blog/diffusion-pipeline-core-processes-practical-guide/
62. Stable Diffusion: A Comprehensive End-to-End Guide with Examples | by Jagadeesan Ganesh | Medium, 8월 24, 2025에 액세스, https://medium.com/@jagadeesan.ganesh/stable-diffusion-a-comprehensive-end-to-end-guide-with-examples-47b2c17f15cf
63. What is Image-to-Image? - Hugging Face, 8월 24, 2025에 액세스, https://huggingface.co/tasks/image-to-image
64. How to Build a Stable Diffusion Image-to-Image Pipeline - Roboflow Blog, 8월 24, 2025에 액세스, https://blog.roboflow.com/stable-diffusion-image-to-image-pipeline/
65. Diffusion-Based Image-to-Image Translation by Noise Correction via Prompt Interpolation, 8월 24, 2025에 액세스, https://arxiv.org/html/2409.08077v1
66. Inpainting and Outpainting with Stable Diffusion ..., 8월 24, 2025에 액세스, https://machinelearningmastery.com/inpainting-and-outpainting-with-stable-diffusion/
67. What is Inpainting and Outpainting with Stable Diffusion? - Generative AI Blog by Segmind, 8월 24, 2025에 액세스, https://blog.segmind.com/stable-diffusion-inpainting-vs-outpainting/
68. An introduction to inpainting and outpainting with Stable Diffusion - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=YWSBGP6FR-s
69. Stable Diffusion Outpainting Tutorial - Forge UI #stablediffusion - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=5_dOevJRzEI
70. ControlNet — Stable Diffusion Model | by Ankit kumar - Medium, 8월 24, 2025에 액세스, https://ankittaxak5713.medium.com/controlnet-stable-diffusion-model-6160a4697119
71. Training Stable Diffusion with Dreambooth ..., 8월 24, 2025에 액세스, https://machinelearningmastery.com/training-stable-diffusion-with-dreambooth/
72. [2302.05543] Adding Conditional Control to Text-to-Image Diffusion Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2302.05543
73. ControlNetModel - Hugging Face, 8월 24, 2025에 액세스, https://huggingface.co/docs/diffusers/api/models/controlnet
74. ControlNet: A Complete Guide - Stable Diffusion Art, 8월 24, 2025에 액세스, https://stable-diffusion-art.com/controlnet/
75. lllyasviel/ControlNet: Let us control diffusion models! - GitHub, 8월 24, 2025에 액세스, https://github.com/lllyasviel/ControlNet
76. Extensive guide to ControlNet: Controlling AI generated Images - Ionio, 8월 24, 2025에 액세스, https://www.ionio.ai/blog/extensive-guide-to-controlnet-controlling-ai-generated-images
77. Using ControlNet with Stable Diffusion - MachineLearningMastery.com, 8월 24, 2025에 액세스, https://machinelearningmastery.com/control-net-with-stable-diffusion/
78. ControlNet Revolutionizes Diffusion Modeling by Allowing Stable Diffusion's AI Characters to be Directed into Specific Poses! | AI-SCHOLAR, 8월 24, 2025에 액세스, https://ai-scholar.tech/en/articles/stable-diffusion%2FImageDiffusionModels
79. [D] what do the two zero convolution layers in controlnet do? : r/MachineLearning - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/MachineLearning/comments/18rdkx0/d_what_do_the_two_zero_convolution_layers_in/
80. ControlNet · Models - Dataloop, 8월 24, 2025에 액세스, https://dataloop.ai/library/model/lllyasviel_controlnet/
81. Fine-Tuning Stable Diffusion with DreamBooth Method | by Enes Sadi Uysal - Medium, 8월 24, 2025에 액세스, https://enessadi.medium.com/fine-tuning-stable-diffusion-with-dreambooth-method-52019b3599dd
82. Fine-tuning Stable Diffusion XL with DreamBooth and LoRA | DataCamp, 8월 24, 2025에 액세스, https://www.datacamp.com/tutorial/fine-tuning-stable-diffusion-xl-with-dreambooth-and-lora
83. The guide to fine-tuning Stable Diffusion with your own images | Tryolabs, 8월 24, 2025에 액세스, https://tryolabs.com/blog/2022/10/25/the-guide-to-fine-tuning-stable-diffusion-with-your-own-images
84. AutoLoRA: AutoGuidance Meets Low-Rank Adaptation for Diffusion Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2410.03941v1
85. arXiv:2106.09685v2 [cs.CL] 16 Oct 2021, 8월 24, 2025에 액세스, https://arxiv.org/abs/2106.09685
86. SDXL: Improving Latent Diffusion Models for High-Resolution Image Synthesis - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2307.01952
87. Stable Diffusion v1.x vs v2.x vs v3.x: A Comprehensive Comparison | Shakker AI, 8월 24, 2025에 액세스, https://wiki.shakker.ai/en/stable-diffusion-v1-vs-v2-vs-v3
88. Stable Diffusion XL: Everything You Need to Know - Magai, 8월 24, 2025에 액세스, https://magai.co/stable-diffusion-xl-1-0/
89. stabilityai/stable-diffusion-xl-base-1.0 · Hugging Face, 8월 24, 2025에 액세스, https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0
90. Generative Models by Stability AI - GitHub, 8월 24, 2025에 액세스, https://github.com/Stability-AI/generative-models
91. Why does AI art screw up hands and fingers? | Explanation, Tools, & Facts | Britannica, 8월 24, 2025에 액세스, https://www.britannica.com/topic/Why-does-AI-art-screw-up-hands-and-fingers-2230501
92. ELI5 please - why does ai struggle producing text? : r/StableDiffusion - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/StableDiffusion/comments/112z3pt/eli5_please_why_does_ai_struggle_producing_text/
93. Why does stable diffusion (and other AI models) struggle hard with human hands and fingers? : r/StableDiffusion - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/StableDiffusion/comments/10kqefv/why_does_stable_diffusion_and_other_ai_models/
94. The Handicap of AI: Understanding Why Hands Remain a Challenge for Image Generation | by SM Raiyyan | Predict | Medium, 8월 24, 2025에 액세스, https://medium.com/predict/the-handicap-of-ai-understanding-why-hands-remain-a-challenge-for-image-generation-b9b10cd752a2
95. Why AI-generated hands are the stuff of nightmares, explained by a scientist, 8월 24, 2025에 액세스, https://www.sciencefocus.com/future-technology/why-ai-generated-hands-are-the-stuff-of-nightmares-explained-by-a-scientist
96. Why can AI do so many things, but not generate correct text/letters for videos, especially maps and posters? (video source: @alookbackintohistory) : r/StableDiffusion - Reddit, 8월 24, 2025에 액세스, https://www.reddit.com/r/StableDiffusion/comments/1jkhz3j/why_can_ai_do_so_many_things_but_not_generate/
97. Bias inspection in LAION-5B. a Proportion of evaluated images. In... - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/figure/Bias-inspection-in-LAION-5B-a-Proportion-of-evaluated-images-In-total-we-identified_fig4_382796566
98. What are the Major Ethical Concerns in Using Generative AI? - Research AIMultiple, 8월 24, 2025에 액세스, https://research.aimultiple.com/generative-ai-ethics/
99. Ethical implications of generative AI: Navigating unchartered waters - Infosys BPM, 8월 24, 2025에 액세스, https://www.infosysbpm.com/blogs/generative-ai/generative-ai-ethical-implications.html
100. Getty Images v. Stability AI: Landmark Copyright Trial at the UK High Court - Finnegan, 8월 24, 2025에 액세스, https://www.finnegan.com/en/insights/articles/getty-images-v-stability-ai-landmark-copyright-trial-at-the-uk-high-court.html
101. Getty Sues Stability AI in Landmark UK Copyright Trial - The National Law Review, 8월 24, 2025에 액세스, https://natlawreview.com/article/two-major-lawsuits-aim-answer-multi-billion-dollar-question-can-ai-train-your
102. Getty Images v Stability AI: the implications for UK copyright law and licensing, 8월 24, 2025에 액세스, https://www.pinsentmasons.com/out-law/analysis/getty-images-v-stability-ai-implications-copyright-law-licensing
103. Getty Images vs. Stability AI: The Landmark Copyright Battle Shaping The Future of Generative AI - Falcon Rappaport & Berkman LLP, 8월 24, 2025에 액세스, https://frblaw.com/getty-images-vs-stability-ai-the-landmark-copyright-battle-shaping-the-future-of-generative-ai/
104. Toward Reliable Provenance in AI-Generated Content: Text, Images, and Code - Medium, 8월 24, 2025에 액세스, https://medium.com/@adnanmasood/toward-reliable-provenance-in-ai-generated-content-text-images-and-code-9ebe8c57ceae
105. C2PA | Verifying Media Content Sources, 8월 24, 2025에 액세스, https://c2pa.org/
106. Content Credentials: Strengthening Multimedia Integrity in the Generative AI Era - Department of Defense, 8월 24, 2025에 액세스, https://media.defense.gov/2025/Jan/29/2003634788/-1/-1/0/CSI-CONTENT-CREDENTIALS.PDF
107. C2PA Implementation Guidance, 8월 24, 2025에 액세스, https://spec.c2pa.org/specifications/specifications/2.2/guidance/Guidance.html
108. What Is Stable Diffusion's Role in Digital Art? | Blog Algorithm Examples, 8월 24, 2025에 액세스, https://blog.algorithmexamples.com/stable-diffusion/stable-diffusion-for-digital-art-creation/
109. [2503.18352] Diffusion-4K: Ultra-High-Resolution Image Synthesis with Latent Diffusion Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2503.18352
110. Diffusion-4K: Ultra-High-Resolution Image Synthesis with Latent Diffusion Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2503.18352v1
111. [2501.01423] Reconstruction vs. Generation: Taming Optimization Dilemma in Latent Diffusion Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2501.01423
112. Fast High-Resolution Image Synthesis with Latent Adversarial Diffusion Distillation - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2403.12015v1
113. [2501.11174] Quantum Latent Diffusion Models - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2501.11174
114. [2503.09419] Alias-Free Latent Diffusion Models:Improving Fractional Shift Equivariance of Diffusion Latent Space - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2503.09419
115. [2507.02608] Lost in Latent Space: An Empirical Study of Latent Diffusion Models for Physics Emulation - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2507.02608


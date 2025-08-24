# 적대적 생성 신경망(GAN)
[적대적 생성 신경망 (Generative Adversarial Network, GAN)](./index.md)


생성적 인공지능(Generative AI) 분야에서 적대적 생성 신경망(Generative Adversarial Network, GAN)의 등장은 패러다임의 전환을 의미한다. 2014년 Ian Goodfellow 등이 발표한 논문을 통해 처음 소개된 GAN은, 단순히 데이터를 분류하거나 회귀 문제를 해결하는 기존의 판별적 모델(discriminative models)과는 근본적으로 다른 접근법을 제시하였다.1 GAN은 새로운 데이터를 창조하는 생성 모델(generative models)의 일종으로, 주어진 데이터의 통계적 분포를 모방하여 현실과 유사한 결과물을 만들어내는 것을 목표로 삼는다.1

GAN의 핵심 철학은 게임 이론(game theory)에서 비롯된 **적대적 게임(adversarial game)**에 기반한다. 이 시스템은 서로 대립하는 두 개의 신경망, 즉 생성자(Generator)와 판별자(Discriminator)로 구성된다.1 생성자는 실제 데이터와 구별할 수 없을 만큼 정교한 가짜 데이터를 만들려고 노력하는 반면, 판별자는 입력된 데이터가 진짜인지 가짜인지 정확히 구별하는 역할을 맡는다.3 이러한 경쟁 관계는 마치 위조지폐범(생성자)과 이를 식별하려는 경찰(판별자)의 끊임없는 경쟁에 비유할 수 있다.1 이 적대적 훈련 과정을 통해 두 신경망은 서로의 성능을 향상시키며, 궁극적으로 생성자가 만든 가짜 데이터가 판별자에게 진짜로 인식될 확률이 50%에 이르는 균형 상태, 즉 **내쉬 균형(Nash Equilibrium)**에 도달하는 것을 목표로 한다.4

이와 같은 GAN의 게임 이론적 접근법은 기존의 생성 모델링 방식이 가진 한계, 특히 이미지 품질 측면에서의 문제를 극복하는 새로운 해결책을 제시하였다. 예를 들어, 변이형 오토인코더(Variational Autoencoders, VAE)가 종종 흐릿하고 덜 선명한 이미지를 생성하는 것과 달리, GAN은 사진과 같은 매우 사실적이고 선명한 결과물을 만들어내는 데 탁월한 성능을 보였다.6 이 보고서는 GAN의 기본 원리에서부터 수학적 정형화, 훈련의 난점과 이를 해결하기 위한 기술적 발전, 주요 변형 모델, 그리고 다양한 응용 분야와 윤리적 함의까지 포괄적으로 탐구할 것이다.



GAN은 생성자와 판별자라는 두 개의 상반된 역할을 수행하는 신경망으로 구성된다. 이들은 단일한 목표를 향해 협력하는 것이 아니라, 각기 다른 목적을 가지고 서로를 속이거나 구별하려 경쟁한다.3

- **생성자(Generator, G)**: 생성자는 일련의 무작위 숫자(random numbers)로 이루어진 벡터, 즉 **잠재 샘플(latent sample)**을 입력받아 고차원 데이터(예: 이미지)를 출력하는 신경망이다.1 이 무작위 입력 벡터 

  $z$는 일반적으로 정규 분포(예: $p_z \sim N(0,1)$)에서 샘플링되며, 이 공간은 **잠재 공간(latent space)**이라 불린다.1 생성자의 목표는 판별자가 자신의 결과물을 진짜라고 판단하게끔, 즉 판별자를 속이는 것이다.3 이미지 생성의 경우, 생성자는 종종 전통적인 신경망의 '역(reverse)'이라 불리는 역 컨볼루션 신경망(transposed convolutional neural network)으로 구현되기도 한다.1

- **판별자(Discriminator, D)**: 판별자는 생성자가 만든 가짜 데이터와 실제 데이터셋으로부터 온 진짜 데이터를 모두 입력받는다.1 판별자의 역할은 입력된 데이터가 진짜 데이터인지, 아니면 생성자가 만든 가짜 데이터인지 이진 분류하는 것이다.1 판별자는 입력이 진짜일 확률을 나타내는 0과 1 사이의 값으로 그 답을 출력한다. 판별자는 종종 컨볼루션 신경망(CNN)과 같은 전통적인 신경망으로 구현된다.1 판별자의 목표는 진짜 데이터는 '진짜(1)'로, 가짜 데이터는 '가짜(0)'로 올바르게 분류하는 것이다.3


GAN의 학습 과정은 생성자와 판별자가 서로의 전략을 끊임없이 개선하며 동적 평형(dynamic equilibrium) 상태를 찾아가는 과정이다.8 생성자는 판별자를 속이는 데 능숙해지고, 판별자는 이를 구별하는 데 더욱 정확해진다. 이러한 경쟁이 최고조에 달하면, 양측은 더 이상 상대방을 쉽게 이길 수 없는 상태에 도달한다. 이 지점이 바로 게임 이론에서 정의하는 **내쉬 균형**이다.4 이 상태에 이르면, 생성자가 만든 가짜 데이터는 판별자가 50%의 확률로만 진짜인지 가짜인지 구별할 수 있게 된다.4

이러한 경쟁적 학습 환경은 GAN 훈련의 고질적인 불안정성(instability)의 근본 원인이 된다.8 만약 한쪽 네트워크가 다른 쪽보다 훨씬 빠른 속도로 학습한다면, 시스템의 균형이 무너져 제대로 된 학습이 이루어지지 않는다.1 예를 들어, 판별자가 생성자보다 훨씬 빨리 최적화되어 완벽하게 진짜와 가짜를 구별해내면, 생성자는 학습을 위한 유의미한 정보를 얻지 못하게 된다. 이 때문에 GAN 훈련에서는 두 네트워크의 학습 속도를 유사하게 조정하여 균형을 맞추는 것이 매우 중요하다고 알려져 있다.1


GAN의 훈련은 생성자와 판별자를 번갈아 가며 업데이트하는 과정을 반복한다.1 이 과정은 일반적으로 판별자를 먼저 일정 횟수 학습시킨 후, 생성자를 학습시키는 순서로 진행된다.1 두 네트워크는 서로 다른 학습 목표를 가지기 때문에, 각각의 손실 함수(loss function)를 최소화하기 위해 독립적인 최적화 도구(optimizer)를 사용하여 파라미터를 업데이트한다.3 이처럼 GAN은 단순한 최적화 문제가 아니라, 실시간으로 변화하는 상대방의 전략에 대응하는 동적 평형을 찾는 문제로 볼 수 있다.8 이러한 특성 때문에 GAN의 학습은 섬세한 균형 조절이 요구되는 공학적 문제의 성격을 지닌다.



GAN의 학습 과정은 생성자 $G$가 판별자 $D$의 능력을 최소화하려고 하고, 동시에 $D$는 자신의 능력을 최대화하려는 **$minG maxD$ 게임**으로 수학적으로 정형화된다.8 판별자 

$D$는 진짜 데이터 $x$를 진짜(출력값 1)로, 생성된 가짜 데이터 $G(z)$를 가짜(출력값 0)로 분류할 확률을 최대화하는 것을 목표로 한다.12 반면, 생성자 

$G$는 판별자 $D$가 자신의 가짜 데이터 $G(z)$를 진짜(출력값 1)로 분류할 확률을 최대화, 즉 $log(1 - D(G(z)))$를 최소화하려 한다.3 이러한 경쟁 관계를 하나의 함수로 표현한 것이 GAN의 목적 함수이며, 다음과 같은 형태로 나타낸다 8:
$$
\min_{G} \max_{D} V(D, G) = \mathbb{E}_{x \sim p_{data}(x)} + \mathbb{E}_{z \sim p_{z}(z)}
$$
여기서 $G$는 생성자, $D$는 판별자, $p_{data}(x)$는 실제 데이터의 분포, $p_z(z)$는 입력 노이즈 $z$의 분포, $G(z)$는 생성자가 만든 데이터, $D(x)$는 판별자의 출력(데이터가 진짜일 확률)을 의미한다.8


GAN의 목적 함수는 이론적으로 생성된 데이터 분포 $p_g$와 실제 데이터 분포 $p_{data}$ 사이의 거리를 측정하는 **옌센-샤넌 발산(Jensen-Shannon Divergence, JSD)**의 최소화와 깊은 관련이 있음이 증명되었다.13 최적의 판별자 $D^*$가 존재할 때, GAN의 목적 함수를 최소화하는 것은 곧 $p_g$와 $p_{data}$ 사이의 $JSD$를 최소화하는 것과 동일하다.13 즉, GAN은 훈련을 통해 생성된 분포가 실제 분포에 최대한 가깝도록 

$JSD$를 줄여나가는 과정이라 볼 수 있다.

그러나 $JSD$는 한계점을 가지고 있다. 고차원 데이터 공간에서 실제 데이터 분포와 생성된 데이터 분포가 겹치지 않는 경우(non-overlapping), $JSD$의 값이 무한대로 발산할 수 있다.14 이 경우 판별자는 진짜와 가짜를 매우 쉽게 구별하게 되며, 생성자는 학습을 위해 필요한 유의미한 그래디언트(gradient)를 받지 못하게 된다.14 이 현상을 **그래디언트 소실(Vanishing Gradients)**이라 하는데 9, 이는 기존 GAN 훈련의 불안정성을 야기하는 근본적인 원인이 되었다. 이러한 $JSD$의 한계가 바서슈타인 GAN(WGAN)과 같은 새로운 목적 함수 기반 모델의 등장을 촉발하는 계기가 되었다.


GAN은 매우 현실적인 결과물을 생성하는 잠재력을 가졌지만, 그 훈련 과정은 매우 불안정하고 어려운 것으로 알려져 있다. 주요 난점으로는 그래디언트 소실, 모드 붕괴, 비수렴성 등이 있으며, 이를 해결하기 위한 다양한 기법들이 연구되었다.5


- **그래디언트 소실(Vanishing Gradients)**: 판별자가 생성자보다 훨씬 빨리 최적의 성능에 도달하여, 진짜와 가짜를 완벽하게 구별하게 되는 문제이다.10 이 경우, 생성자의 손실 함수 그래디언트가 거의 0에 수렴하여, 생성자의 파라미터가 더 이상 업데이트되지 않고 학습이 멈춘다.9 이는 판별자가 제공하는 피드백이 생성자가 발전하는 데 충분한 정보를 제공하지 못하기 때문에 발생한다.10
- **모드 붕괴(Mode Collapse)**: 생성자가 전체 데이터 분포의 다양성(diversity)을 포착하지 못하고, 판별자를 속이기 가장 쉬운 몇몇 데이터(mode)만을 반복적으로 생성하는 현상이다.5 이 현상이 발생하면, 생성된 이미지의 다양성이 현저히 떨어져 모델의 활용 가치가 낮아진다.15 이는 생성자가 특정 판별자에 과최적화되고, 판별자는 해당 출력물을 거부하도록 학습하지만, 다시 생성자는 다른 '속이기 쉬운' 출력물을 찾아 순환하는 동적인 실패(oscillatory failure) 패턴 때문에 발생할 수 있다.15
- **비수렴성(Non-Convergence)**: 생성자와 판별자가 최적의 내쉬 균형에 도달하지 못하고, 서로의 전략에 맞춰 계속해서 진동(oscillate)하며 안정적으로 수렴하지 못하는 현상이다.5 GAN의 최적화 문제가 비볼록(non-convex)한 특성을 가지기 때문에, 손실 함수의 값이 안정화된 것처럼 보일지라도 두 네트워크는 계속해서 변화할 수 있다.8


- DCGAN(Deep Convolutional GAN):

  DCGAN은 컨볼루션 신경망(CNN)을 기반으로 한 GAN의 변형으로, 훈련 안정성을 높이기 위한 아키텍처적 제약들을 도입하였다.17 주요 제약으로는 풀링(pooling) 레이어 대신 스트라이드 컨볼루션(strided convolution)을 사용하고, 풀리 커넥티드(fully connected) 히든 레이어를 제거하며, 배치 정규화(Batch Normalization)와 LeakyReLU 활성화 함수를 사용하는 것 등이 있다.17 이러한 아키텍처적 제약들은 훈련을 더 안정화시키고 이미지 품질을 향상시키는 데 기여하였다.17

- WGAN(Wasserstein GAN) 및 WGAN-GP(Gradient Penalty):

  WGAN은 기존 GAN의 $JSD$ 기반 목적 함수가 가진 그래디언트 소실 문제를 해결하기 위해, **지구 이동자 거리(Earth Mover's Distance)**라고도 불리는 **바서슈타인 거리(Wasserstein distance)**를 도입하였다.13 바서슈타인 거리는 두 분포가 겹치지 않아도 연속적인 거리를 제공하므로, 판별자(WGAN에서는 Critic이라 부름)가 최적으로 학습되어도 생성자에게 유의미한 그래디언트 신호를 전달할 수 있다.14

  WGAN은 바서슈타인 거리를 계산하기 위해 Critic이 **립시츠 제약(Lipschitz constraint)**을 만족해야 한다. 초기 WGAN은 이를 위해 **가중치 클리핑(Weight Clipping)**을 사용했으나, 이는 Critic의 가중치를 극단적인 값으로 제한하여 성능 저하를 초래하는 문제가 있었다.19 이 문제를 해결하기 위해, 손실 함수에 그래디언트의 

  $L2$ `norm`에 페널티를 부여하는 **WGAN-GP(Gradient Penalty)**가 제안되었다.9 이는 립시츠 제약을 부드럽게(softly) 강제하여 가중치 클리핑의 부작용 없이 훈련 안정성을 극대화하였다.9

- 미니배치 판별(Mini-batch Discrimination) 및 기타 정규화:

  미니배치 판별은 판별자가 단일 샘플이 아닌, 미니배치 내의 전체 샘플 간의 관계를 고려하도록 하는 기법이다.9 이 기법은 생성자가 판별자를 속이기 가장 쉬운 하나의 출력물만을 반복적으로 생성하는 것을 방지함으로써, 모드 붕괴를 완화하는 데 효과적이다.9 이외에도 Unrolled GANs, Spectral Normalization 등 다양한 정규화 기법이 GAN의 훈련 안정성을 개선하기 위해 제안되었다.9

GAN 훈련 안정성의 발전 역사는 곧 목적 함수의 한계를 극복하고, 아키텍처 및 손실 함수에 제약을 가하는 발전 과정과 동일하다. 이는 GAN의 학습이 단순히 최적화 문제의 해결이 아닌, 두 모델 간의 섬세한 균형을 찾는 공학적 문제임을 명확히 보여준다.

| 모델명          | 핵심 원리                       | 개선된 문제점                             | 주요 특징                                             |
| --------------- | ------------------------------- | ----------------------------------------- | ----------------------------------------------------- |
| **Vanilla GAN** | 생성자와 판별자의 Minimax 게임  | 훈련 불안정성, 그래디언트 소실, 모드 붕괴 | 기본 모델, JSD 기반 손실 함수                         |
| **DCGAN**       | CNN 기반 아키텍처와 구조적 제약 | 훈련 안정성, 이미지 품질 향상             | 풀링 층 제거, 배치 정규화, LeakyReLU 사용             |
| **WGAN**        | 바서슈타인 거리 기반 손실 함수  | 그래디언트 소실, 모드 붕괴                | Critic(판별자)에 립시츠 제약 적용, 가중치 클리핑 사용 |
| **WGAN-GP**     | WGAN에 그래디언트 페널티 추가   | 가중치 클리핑의 부작용                    | 립시츠 제약을 부드럽게 강제하여 안정성 극대화         |
| **StyleGAN**    | 스타일 기반 생성기              | 고해상도 이미지 품질, 제어 가능성         | 매핑 네트워크, AdaIN, 프로그레시브 성장 방식          |



초기 GAN은 생성되는 데이터의 종류를 제어할 수 없는 **무조건적(unconditional)** 모델이었다. 예를 들어, MNIST 데이터셋으로 훈련된 GAN은 무작위의 숫자를 생성할 뿐, 특정 숫자(예: 7)를 생성하라고 명령할 수는 없었다.20 이러한 한계를 극복하기 위해 제안된 것이 **조건부 GAN(cGAN)**이다.21

cGAN은 생성자와 판별자 모두에 추가적인 정보, 즉 **조건(condition)**을 제공한다.20 이 조건은 클래스 레이블, 텍스트, 또는 다른 형태의 데이터가 될 수 있다. cGAN의 생성자는 무작위 노이즈 벡터 

$z$와 조건 $y$를 함께 입력받아 $G(z|y)$를 생성하며, 판별자 역시 실제/가짜 데이터와 조건 $y$를 함께 평가하여 $D(x|y)$를 계산한다.20 이러한 방식으로 cGAN은 "특정 조건에 부합하는 그럴듯한" 데이터 생성을 목표로 한다.22 이 기술은 이미지-투-이미지 변환, 텍스트-투-이미지 생성 등 GAN의 응용 범위를 획기적으로 확장하는 기반이 되었다.22


**SRGAN(Super-Resolution GAN)**은 저해상도(Low-Resolution, LR) 이미지를 고해상도(High-Resolution, HR) 이미지로 복원하는 초해상도(Super-Resolution) 기술에 GAN을 적용한 모델이다.24 기존의 초해상도 기술은 주로 픽셀 단위의 차이를 측정하는 **MSE(Mean Squared Error) 손실 함수**에 의존했는데, 이는 이미지의 세부적인 질감(texture)을 뭉개뜨려 결과물이 흐릿하고 비현실적으로 보이는 한계가 있었다.24

SRGAN은 이러한 MSE loss의 한계를 극복하기 위해 **지각 손실(Perceptual Loss)**과 **적대적 손실(Adversarial Loss)**을 결합한 이중 손실 함수를 도입하였다.24 지각 손실은 VGG와 같은 사전 훈련된 신경망의 특징 맵(feature map)을 사용하여 이미지의 픽셀 단위 차이가 아닌, 내용적, 스타일적 유사성을 측정한다. 적대적 손실은 생성된 이미지가 실제 데이터의 분포(manifold of natural images)를 따르는 질감을 갖도록 유도한다.24 이 두 손실 함수의 결합을 통해 SRGAN은 사람의 시각적 인지 품질(perceptual quality)을 크게 향상시키고, 사진과 같은 사실적인 고해상도 이미지를 생성하는 데 성공하였다.24


**StyleGAN**은 NVIDIA 연구진이 개발한 모델로, 기존의 Progressive Growing GAN의 단계를 이어받아 훈련 안정성과 고해상도 이미지 품질을 더욱 향상시킨 모델이다.26 StyleGAN의 핵심은 이미지의 다양한 특징(예: 자세, 얼굴형, 머리 스타일)을 정교하게 제어할 수 있도록 생성자 아키텍처를 혁신한 데 있다.26

- **매핑 네트워크(Mapping Network)**: 기존 GAN이 하나의 무작위 노이즈 벡터 $z$를 입력받는 것과 달리, StyleGAN은 이 $z$를 여러 개의 풀리 커넥티드(fully connected) 레이어로 구성된 별도의 **매핑 네트워크**를 통과시켜 중간 잠재 공간(intermediate latent space)의 벡터 $w$로 변환한다.27 이 

  $w$는 이미지의 '스타일(style)'을 정의하며, 기존 잠재 벡터 $z$보다 더 선형적인 특성을 가져 이미지의 각 특징들을 독립적으로 제어하기 용이하다.26

- **적응형 인스턴스 정규화(Adaptive Instance Normalization, AdaIN)**: 매핑 네트워크에서 생성된 스타일 벡터 $w$는 생성 네트워크의 각 컨볼루션 블록에 주입된다.26 이 과정에 

  **AdaIN**이라는 특별한 정규화 기법이 사용되는데, 이는 각 레이어의 특징 맵(feature map)에 스타일 벡터의 통계량(평균, 분산)을 적용하여 이미지의 세부적인 특성을 조절하는 역할을 한다.26

StyleGAN은 이처럼 **콘텐츠**와 **스타일**을 분리하여 학습함으로써, 단순한 노이즈 벡터 조작으로는 불가능했던 이미지의 세부적인 특성 제어를 가능하게 하였다. 이는 생성된 이미지의 품질과 현실성을 극적으로 향상시켰고, AI가 생성한 얼굴 이미지가 실제 사람의 얼굴과 구별하기 어려울 만큼 정교하게 만들어지는 계기가 되었다.



GAN은 그 독특한 특성으로 인해 이미지 합성뿐만 아니라 다양한 분야에서 혁신적인 응용 가능성을 보여주었다.

- **이미지 합성 및 생성**: GAN의 가장 대표적인 응용 분야이다. 새로운 인물의 얼굴 생성, 예술 작품 제작, 비디오 게임의 캐릭터나 환경 생성 등 무한한 창작물을 만들어내는 데 활용된다.4
- **데이터 증강(Data Augmentation)**: 학습 데이터가 부족한 상황에서, GAN은 실제 데이터와 유사한 가짜 데이터를 대량으로 생성하여 데이터셋의 크기를 효과적으로 늘린다.4 이는 특히 의료 영상 분야에서 유용하게 사용되는데, 희귀 질환에 대한 훈련 데이터가 부족할 때 GAN이 생성한 합성 데이터를 활용하여 진단 모델의 정확도와 성능을 높일 수 있다.11
- **이미지-투-이미지 변환(Image-to-Image Translation)**: 낮 이미지를 밤 이미지로 변환하거나, 스케치를 사실적인 이미지로 바꾸는 등 한 도메인의 이미지를 다른 도메인으로 변환하는 작업에 cGAN이 활용된다.21
- **이미지 복원 및 초해상도**: 저해상도 이미지를 고해상도로 복원하거나, 흑백 이미지를 컬러 이미지로 바꾸고, 노이즈를 제거하는 등 이미지의 품질을 개선하는 데 GAN이 사용된다.21
- **3D 모델 생성**: GAN은 2D 사진이나 스캔 데이터를 기반으로 3D 모델을 생성하는 데에도 활용된다.21 이는 의료 분야에서 X-선 및 기타 체내 스캔 데이터를 결합하여 장기의 사실적인 3D 이미지를 생성하는 데 사용될 수 있다.21


GAN의 성능 평가는 단순히 손실 함수의 값으로 판단하기 어렵다. 손실 함수의 값이 낮더라도 생성된 이미지의 품질이 좋지 않을 수 있기 때문에, GAN의 특성에 맞는 특수 지표들이 개발되었다.

- **인셉션 스코어(Inception Score, IS)**: $IS$는 생성된 이미지의 **품질(quality)**과 **다양성(diversity)**을 동시에 측정하는 지표다.9 이 점수는 사전 훈련된 Inception v3 모델을 사용하여 계산된다. 품질은 생성된 각 이미지의 클래스 예측 분포가 뾰족할수록(엔트로피가 낮을수록), 다양성은 이미지 전체의 클래스 분포가 넓을수록(엔트로피가 높을수록) 높게 평가된다.30 따라서 높은 

  $IS$는 고품질의 다양한 이미지를 생성함을 의미한다.

- **프레셰 인셉션 거리(Frechet Inception Distance, FID)**: $FID$는 생성된 이미지와 실제 이미지 사이의 **통계적 유사성**을 측정하는 지표다.9

  $FID$는 Inception 모델의 중간 레이어에서 추출된 특징 벡터들을 다변량 가우시안 분포로 모델링하고, 이 두 분포 사이의 거리를 계산한다.30 낮은 

  $FID$ 값은 생성된 이미지의 분포가 실제 데이터 분포와 더 유사함을 의미한다. $FID$는 모드 붕괴와 같은 문제를 더 잘 포착할 수 있어, $IS$보다 더 견고한 평가 지표로 여겨진다.31

실제 연구 및 개발에서는 $IS$와 $FID$를 보완적으로 활용하여 모델의 성능을 종합적으로 평가하는 것이 일반적이다.31



GAN은 생성 모델링 분야에 큰 영향을 미쳤으나, 훈련의 불안정성과 모드 붕괴와 같은 고질적인 문제점을 안고 있다. 최근에는 변이형 오토인코더(VAE)와 확산 모델(Diffusion Models)과 같은 다른 생성 모델들이 발전하며 GAN의 한계를 극복하는 새로운 해결책을 제시하고 있다.

| 모델명        | 핵심 원리                           | 훈련 안정성   | 출력 품질                   | 계산 비용        | 주요 활용 사례                              |
| ------------- | ----------------------------------- | ------------- | --------------------------- | ---------------- | ------------------------------------------- |
| **GAN**       | 생성자와 판별자의 적대적 경쟁       | 낮음 (불안정) | 매우 높음 (사실적)          | 추론 속도가 빠름 | 이미지 합성, 데이터 증강, 스타일 변환       |
| **VAE**       | 인코더-디코더 기반의 잠재 공간 학습 | 높음 (안정적) | 낮음 (흐릿함)               | 추론 속도가 빠름 | 데이터 압축, 잠재 공간 보간                 |
| **확산 모델** | 반복적인 노이즈 제거 과정           | 높음 (안정적) | 매우 높음 (다양하고 사실적) | 추론 속도가 느림 | 텍스트-투-이미지 생성, 고해상도 이미지 생성 |

GAN은 VAE에 비해 훨씬 사실적인 결과물을 생성하는 장점이 있었으나, 훈련의 불안정성과 모드 붕괴라는 문제점을 극복하는 데 어려움을 겪었다.6 확산 모델은 이러한 GAN의 한계를 대부분 해결하며, 현재 가장 사실적이고 다양한 고해상도 이미지를 생성하는 기술로 각광받고 있다.6 확산 모델은 반복적인 노이즈 제거 과정(iterative denoising steps)을 거치므로, GAN에 비해 추론(inference) 속도가 매우 느린 단점을 가진다.6 그럼에도 불구하고, 확산 모델은 이미지 품질과 다양성 측면에서 GAN의 기술적 우위를 위협하며 새로운 표준으로 자리 잡는 추세이다.


GAN은 긍정적인 응용 사례 못지않게, 그 기술적 오용으로 인해 심각한 윤리적, 사회적 문제를 야기하고 있다. 특히 사람의 얼굴이나 목소리를 합성하는 **딥페이크(deepfake)** 기술의 발전은 GAN의 양면성을 극명하게 보여준다.32

- **오용 사례**: 딥페이크는 가짜 뉴스를 생성하거나, 여론을 조작하거나, 선거를 방해하는 등 허위 정보를 퍼뜨리는 데 악용될 수 있다.33 또한, 개인의 동의 없이 합성된 노출 콘텐츠를 제작하는 데 사용되어, 특히 여성을 표적으로 삼아 개인의 프라이버시를 침해하고 정서적 피해를 야기하는 심각한 문제로 대두되었다.33
- **법적 및 사회적 문제**: 딥페이크의 악용은 미디어와 정보에 대한 신뢰를 약화시키고, 기존의 명예훼손, 저작권, 프라이버시 관련 법률로는 해결하기 어려운 새로운 법적 과제를 제기하고 있다.34 이에 대한 대응으로, 중국의 개인정보보호법(PIPL)은 딥페이크 콘텐츠 사용 시 명시적인 동의를 얻고 라벨을 부착하도록 의무화하였으며, EU와 미국 등에서도 관련 법안을 마련하는 움직임이 나타나고 있다.34
- **대응 동향**: 이러한 문제에 대응하기 위해, 기술적, 법적, 교육적 노력을 포함하는 다각적 접근이 필요하다. AI 기반 딥페이크 탐지 기술에 투자하고, 대중의 미디어 리터러시를 강화하여 조작된 콘텐츠를 식별하고 의문을 제기하도록 돕는 것이 중요하다.34 또한, 기업들은 기술의 창의적 잠재력을 책임감 있게 활용하여 악의적인 사용을 억제하는 데 주력해야 한다.34


적대적 생성 신경망(GAN)은 2014년 등장 이후 생성적 인공지능 분야에 혁명적인 변화를 가져왔다. 생성자와 판별자의 적대적 경쟁이라는 독특한 게임 이론적 접근법을 통해, GAN은 기존 모델의 한계를 넘어 사진과 같은 사실적이고 고품질의 이미지를 생성하는 새로운 가능성을 열었다. DCGAN, WGAN, StyleGAN 등 다양한 변형 모델들은 훈련 안정성과 출력 품질을 지속적으로 개선하며 GAN 기술의 발전을 이끌어왔다.

그러나 GAN은 훈련의 불안정성과 모드 붕괴라는 근본적인 문제로 인해 지속적인 난제에 직면해왔다. 이로 인해 최근에는 훈련이 안정적이고 높은 다양성과 품질을 동시에 제공하는 확산 모델(Diffusion Models)이 생성 모델링 분야의 새로운 표준으로 부상하고 있다.

그럼에도 불구하고, GAN은 여전히 유효한 기술이다. 특히 단일 순전파(single forward pass)로 데이터를 빠르게 생성하는 GAN의 장점은 실시간 애플리케이션 등 특정 환경에서 여전히 중요한 가치를 지닌다.7 또한, StyleGAN과 같은 모델이 제공하는 정교한 제어 가능성은 예술, 디자인, 엔터테인먼트 등 다양한 분야에서 독보적인 경쟁력을 유지하고 있다.

향후 GAN 연구는 훈련 안정성을 더욱 강화하고, 다양한 데이터 분포를 포착하는 능력을 개선하는 방향으로 진행될 것으로 예상된다. 동시에, 딥페이크와 같은 기술의 윤리적 오용을 방지하기 위한 기술적, 법적, 사회적 대응책 마련이 중요한 과제로 남아있다. GAN은 비록 새로운 경쟁자들의 도전을 받고 있지만, 여전히 생성적 인공지능의 발전사를 논할 때 빼놓을 수 없는 핵심적인 기술로 기억될 것이다.


1. Generative Adversarial Networks | CAIS++, 8월 14, 2025에 액세스, https://caisplusplus.usc.edu/curriculum/neural-network-flavors/generative-adversarial-networks
2. From Scratch - Generative Adversarial Networks, 8월 14, 2025에 액세스, https://ym2132.github.io/GenerativeAdversarialNetworks_Goodfellow
3. [Hands-On] GAN의 이해와 구현. PyTorch를 사용한 GAN 구현 및 학습 ..., 8월 14, 2025에 액세스, [https://medium.com/@hugmanskj/hands-on-gan%EC%9D%98-%EC%9D%B4%ED%95%B4%EC%99%80-%EA%B5%AC%ED%98%84-866e03069995](https://medium.com/@hugmanskj/hands-on-gan의-이해와-구현-866e03069995)
4. GAN에 대한 이해. 생성적 적대 신경망(GAN)의 기본 개념, 훈련 방법, 응용 사례를... - Medium, 8월 14, 2025에 액세스, [https://medium.com/@hugmanskj/gan%EC%97%90-%EB%8C%80%ED%95%9C-%EC%9D%B4%ED%95%B4-a073a5425ef2](https://medium.com/@hugmanskj/gan에-대한-이해-a073a5425ef2)
5. GAN Challenges and Optimal Solutions - IRJET, 8월 14, 2025에 액세스, https://www.irjet.net/archives/V8/i10/IRJET-V8I10132.pdf
6. Comparative Analysis of Generative Models: Enhancing Image Synthesis with VAEs, GANs, and Stable Diffusion - arXiv, 8월 14, 2025에 액세스, https://arxiv.org/html/2408.08751v1
7. How does a diffusion model compare with GANs and VAEs? - Milvus, 8월 14, 2025에 액세스, https://milvus.io/ai-quick-reference/how-does-a-diffusion-model-compare-with-gans-and-vaes
8. Challenges in training GAN-Generative Adversarial Network | by Mohd Usama - Medium, 8월 14, 2025에 액세스, https://medium.com/@usama.6832/why-its-hard-to-train-gan-generative-adversarial-network-a05a7656f26d
9. Training Stability in GANs | Saturn Cloud, 8월 14, 2025에 액세스, https://saturncloud.io/glossary/training-stability-in-gans/
10. Common Problems | Machine Learning | Google for Developers, 8월 14, 2025에 액세스, https://developers.google.com/machine-learning/gan/problems
11. Deep Learning Approaches for Data Augmentation in Medical ..., 8월 14, 2025에 액세스, https://www.mdpi.com/2313-433X/9/4/81
12. towardsdatascience.com, 8월 14, 2025에 액세스, [https://towardsdatascience.com/mini-max-optimization-design-of-generative-adversarial-networks-gan-dc1b9ea44a02/#:~:text=Therefore%2C%20GAN%20has%20two%20objective,data%20(label%3D0).](https://towardsdatascience.com/mini-max-optimization-design-of-generative-adversarial-networks-gan-dc1b9ea44a02/#:~:text=Therefore%2C GAN has two objective,data (label%3D0).)
13. Exploring GAN Objective Functions | Hunter Heidenreich, Machine ..., 8월 14, 2025에 액세스, https://hunterheidenreich.com/posts/gan-objective-functions/
14. Wasserstein GAN - Wikipedia, 8월 14, 2025에 액세스, https://en.wikipedia.org/wiki/Wasserstein_GAN
15. Mode collapse - Wikipedia, 8월 14, 2025에 액세스, https://en.wikipedia.org/wiki/Mode_collapse
16. Mode Collapse In GANs Explained, How To Detect It & Practical Solutions - Spot Intelligence, 8월 14, 2025에 액세스, https://spotintelligence.com/2023/10/11/mode-collapse-in-gans-explained-how-to-detect-it-practical-solutions/
17. Understanding DCGANs: Deep Convolutional Generative ... - Medium, 8월 14, 2025에 액세스, https://medium.com/@danushidk507/understanding-dcgans-deep-convolutional-generative-adversarial-networks-1984bc028bf8
18. Deep Convolutional Generative Adversarial Networks (DCGANs) - Pianalytix, 8월 14, 2025에 액세스, https://pianalytix.com/deep-convolutional-generative-adversarial-networks-dcgans/
19. Improved Training of Wasserstein GANs, 8월 14, 2025에 액세스, http://papers.neurips.cc/paper/7159-improved-training-of-wasserstein-gans.pdf
20. Pytorch implementation of Conditional-GAN (CGAN) - GitHub, 8월 14, 2025에 액세스, https://github.com/qbxlvnf11/conditional-GAN
21. What is a GAN? Definition of Generative AI Network - AWS, 8월 14, 2025에 액세스, https://aws.amazon.com/what-is/gan/
22. What Is a Conditional Generative Adversarial Network? | Coursera, 8월 14, 2025에 액세스, https://www.coursera.org/articles/conditional-generative-adversarial-network
23. GANs for Image Synthesis - Number Analytics, 8월 14, 2025에 액세스, https://www.numberanalytics.com/blog/gans-for-image-synthesis
24. SRGAN: The Power of GANs in Super-Resolution | by Dong-Keon ..., 8월 14, 2025에 액세스, https://medium.com/@kdk199604/srgan-the-power-of-gans-in-super-resolution-94f39a530a61
25. pohwa065/SRGAN-for-Super-Resolution-and-Image-Enhancement - GitHub, 8월 14, 2025에 액세스, https://github.com/pohwa065/SRGAN-for-Super-Resolution-and-Image-Enhancement
26. StyleGAN Explained in Less Than Five Minutes - Analytics Vidhya, 8월 14, 2025에 액세스, https://www.analyticsvidhya.com/blog/2021/05/stylegan-explained-in-less-than-five-minutes/
27. ProgressiveGAN and styleGAN - Flying Whales, 8월 14, 2025에 액세스, https://blog.flyingwhales.io/gan/2023/02/28/progressive-gan
28. 10 GAN Use Cases in August 2025 - Research AIMultiple, 8월 14, 2025에 액세스, https://research.aimultiple.com/gan-use-cases/
29. Medical Image Generation with GANs - Kaggle, 8월 14, 2025에 액세스, https://www.kaggle.com/code/asif00/medical-image-generation-with-gans
30. What are Inception Score and FID, and how do they apply here ..., 8월 14, 2025에 액세스, https://zilliz.com/ai-faq/what-are-inception-score-and-fid-and-how-do-they-apply-here
31. What are Inception Score and FID, and how do they apply here? - Milvus, 8월 14, 2025에 액세스, https://milvus.io/ai-quick-reference/what-are-inception-score-and-fid-and-how-do-they-apply-here
32. Generative artificial intelligence - Wikipedia, 8월 14, 2025에 액세스, https://en.wikipedia.org/wiki/Generative_artificial_intelligence
33. Generative AI Ethics in 2025: Top 6 Concerns - Research AIMultiple, 8월 14, 2025에 액세스, https://research.aimultiple.com/generative-ai-ethics/
34. Deepfakes and the Future of AI Legislation: Overcoming the Ethical ..., 8월 14, 2025에 액세스, https://gdprlocal.com/deepfakes-and-the-future-of-ai-legislation-overcoming-the-ethical-and-legal-challenges/


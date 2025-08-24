[DiT (Diffusion Transformer)](./index.md)
# 확산 트랜스포머(DiT)


인공지능(AI)의 생성 모델링 분야는 최근 몇 년간 괄목할 만한 발전을 이루었으며, 그 중심에는 확산 모델(Diffusion Models)과 트랜스포머(Transformer) 아키텍처라는 두 가지 핵심 기술이 자리 잡고 있습니다. 각각 독립적으로도 강력한 성능을 입증한 이 두 기술의 결합은 '확산 트랜스포머(Diffusion Transformer, DiT)'라는 새로운 패러다임을 탄생시키며 생성형 AI의 지형을 근본적으로 바꾸어 놓았습니다. 본 보고서는 DiT의 핵심 원리부터 시작하여, 그 아키텍처적 진화, 성능 벤치마킹, 현재의 도전 과제, 그리고 이미지, 비디오, 3D, 과학 등 다양한 영역으로 확장되는 응용 분야와 미래 전망에 이르기까지, DiT에 대한 심층적이고 종합적인 분석을 제공하는 것을 목표로 합니다. 이를 통해 DiT가 어떻게 기존 생성 모델의 한계를 극복하고 새로운 가능성을 열었는지, 그리고 이 기술이 앞으로 나아갈 방향은 무엇인지에 대한 깊이 있는 통찰을 제시하고자 합니다.


확산 모델은 데이터를 점진적으로 손상시키는 순방향 프로세스(forward process)와 이를 복원하는 역방향 프로세스(reverse process)를 통해 데이터를 생성하는 강력한 생성 모델 클래스입니다.1 이 모델의 핵심 아이디어는 제어 가능한 노이즈 추가 과정을 학습하여 그 반대 방향으로 데이터를 생성하는 것입니다.

- **순방향 프로세스 (노이즈 추가):** 이 단계는 고정된 마르코프 연쇄(Markov chain) 과정으로, 원본 데이터 샘플 $x_0$에 여러 타임스텝 $T$에 걸쳐 점진적으로 가우시안 노이즈(Gaussian noise)를 추가하여 최종적으로 순수한 등방성 노이즈(isotropic noise) $x_T$로 변환합니다.1 이 과정은 수학적으로 미리 정의되어 있으며, 학습을 필요로 하지 않습니다.4
- **역방향 프로세스 (노이즈 제거):** 이 단계에서는 학습된 신경망이 순방향 프로세스를 거꾸로 되돌리는 과정을 수행합니다. 순수한 노이즈 $x_T$에서 시작하여 각 타임스텝에서 점진적으로 노이즈를 제거함으로써 깨끗한 원본 데이터 샘플 $x_0$를 복원합니다.1 모델은 일반적으로 각 단계에서 추가된 노이즈 $\epsilon$을 예측하도록 훈련되며, 이는 데이터 로그 우도(log-likelihood)에 대한 변분 하한(variational lower bound)에서 파생된 간단한 평균 제곱 오차(mean-squared error) 손실 함수를 사용하여 최적화됩니다.5

초기 확산 모델들은 고해상도 이미지의 픽셀 공간에서 직접 작동하여 막대한 계산 비용을 유발했습니다. 이 문제를 해결하기 위해 잠재 확산 모델(Latent Diffusion Models, LDM)이 등장했습니다.3 LDM은 사전에 훈련된 변이형 오토인코더(Variational Autoencoder, VAE)를 사용하여 고차원의 이미지 데이터를 저차원의 잠재 공간(latent space) 표현 $z$로 압축하고, 이 잠재 공간 내에서 확산 및 노이즈 제거 프로세스를 수행합니다. 최종적으로 VAE 디코더가 노이즈가 제거된 잠재 벡터를 다시 픽셀 공간으로 변환하여 이미지를 생성합니다.7 이 방식은 계산 효율성을 극적으로 향상시켰습니다.3

이러한 점진적 정제 과정은 확산 모델이 고품질의 결과물을 생성하는 핵심 비결입니다. 생성적 적대 신경망(GAN)과 같은 단일 단계(single-shot) 생성 모델과 달리, 확산 모델은 여러 단계를 거치며 오류를 수정할 수 있는 기회를 가집니다.2 이로 인해 높은 생성 품질뿐만 아니라, 데이터 분포의 다양한 모드(mode)를 포괄하고 높은 다양성을 확보할 수 있습니다.1


2017년 "Attention Is All You Need" 논문에서 처음 제안된 트랜스포머는 순환 신경망(RNN)이나 합성곱 신경망(CNN)에 의존하지 않고 오직 자기 주의(self-attention) 메커니즘만을 사용하는 신경망 아키텍처입니다.10 이 구조는 입력 시퀀스 전체를 병렬로 처리할 수 있어 GPU와 같은 최신 하드웨어에서의 훈련 효율성을 극대화했습니다.10

트랜스포머의 핵심 혁신은 자기 주의 메커니즘으로, 이는 입력 시퀀스 내의 각 요소(토큰)가 다른 모든 요소와 상호작용하며 서로의 상대적 중요도를 동적으로 계산하게 합니다.10 각 토큰에 대해 모델은 쿼리(Query, $Q$), 키(Key, $K$), 밸류(Value, $V$)라는 세 가지 벡터를 계산합니다. 특정 토큰의 쿼리 벡터와 시퀀스 내 모든 다른 토큰들의 키 벡터 간의 내적(dot product)을 통해 주의 점수(attention score)를 산출합니다. 이 점수들은 소프트맥스(softmax) 함수를 통해 정규화되어 주의 가중치(attention weights)가 되며, 이 가중치들은 밸류 벡터들의 가중 합을 계산하는 데 사용됩니다.10 이 메커니즘 덕분에 트랜스포머는 시퀀스 내 요소들 간의 거리에 상관없이 복잡하고 장거리의 의존성을 효과적으로 포착할 수 있습니다.10

트랜스포머의 주요 구성 요소는 다음과 같습니다:

- **토큰화 및 임베딩:** 텍스트와 같은 입력 데이터는 토큰(token)이라는 작은 단위로 분할된 후, 각 토큰의 의미를 나타내는 고차원 수치 벡터인 임베딩(embedding)으로 변환됩니다.11
- **위치 인코딩:** 자기 주의 메커니즘 자체는 순서 정보를 고려하지 않으므로(permutation-invariant), 토큰의 순서 정보를 보존하기 위해 위치 인코딩(positional encoding) 벡터가 토큰 임베딩에 명시적으로 더해집니다.10
- **다중 헤드 주의(Multi-Head Attention):** 단일 주의 메커니즘 대신, 여러 개의 주의 "헤드"를 병렬로 사용하여 모델이 동시에 다양한 종류의 관계와 의미적 측면을 포착할 수 있도록 합니다.10
- **피드포워드 신경망(Feed-Forward Networks):** 각 다중 헤드 주의 계층 다음에는 위치별 피드포워드 신경망(MLP)이 위치하며, 이는 각 토큰 표현에 비선형 변환을 적용하여 표현력을 더욱 풍부하게 만듭니다.11


비전 트랜스포머(Vision Transformer, ViT)는 트랜스포머 아키텍처를 이미지 처리 작업에 성공적으로 적용한 모델입니다.10 ViT는 이미지를 픽셀 시퀀스로 직접 처리하는 대신(이는 계산적으로 비효율적임), 이미지를 겹치지 않는 격자 형태의 패치(patch)들로 분할하는 혁신적인 접근법을 채택했습니다.7

이 '패치화(patchification)' 과정은 다음과 같이 진행됩니다:

1. 이미지를 고정된 크기(예: 16x16 픽셀)의 패치들로 분할합니다.7
2. 각 패치를 평탄화(flatten)하고 선형 투영(linear projection)을 통해 단어 토큰 임베딩과 유사한 벡터 임베딩으로 변환합니다.10
3. 공간 정보를 유지하기 위해 이 패치 임베딩에 위치 인코딩을 추가합니다.10
4. 이렇게 생성된 패치 임베딩 시퀀스는 표준 트랜스포머 인코더의 입력으로 사용됩니다.7

ViT의 등장은 컴퓨터 비전 분야에서 합성곱(convolutional) 연산에 기반한 귀납적 편향(inductive bias)이 반드시 필요한 것은 아님을 증명했습니다. 충분히 큰 데이터셋으로 훈련될 경우, 범용적인 시퀀스 처리 아키텍처인 트랜스포머가 최첨단 성능을 달성할 수 있음을 보여주었습니다.5 이는 트랜스포머가 다른 시각 관련 생성 모델의 백본(backbone)으로 사용될 수 있는 길을 열어주었습니다.

이러한 배경 속에서, 확산 트랜스포머(DiT)의 개념이 탄생할 수 있었습니다. ViT의 성공은 DiT의 구상에 있어 필수적인 전제 조건이었습니다. 트랜스포머가 패치 토큰화를 통해 이미지의 공간적 관계를 효과적으로 모델링할 수 있음을 입증함으로써, 복잡한 노이즈 제거 작업을 위한 신경망 백본으로 합성곱 기반의 U-Net 대신 순수 트랜스포머를 사용하는 아이디어에 대한 결정적인 청사진을 제공했습니다. DiT 관련 연구들이 ViT의 모범 사례를 따른다고 명시적으로 밝히고 있으며 5, DiT의 핵심 메커니즘인 '패치화' 계층은 ViT의 패러다임을 VAE의 잠재 공간에 직접 적용한 것입니다.5 이는 단순한 영감을 넘어 직접적인 기술적 계보와 의존 관계가 있음을 보여줍니다.

결과적으로 DiT의 탄생은 AI 분야에서 각기 다른 강력한 연구 흐름이었던 세 가지 축의 융합을 상징합니다. 첫째, 확산 모델의 확률론적이고 반복적인 생성 프레임워크 1, 둘째, 자연어 처리(NLP)를 혁신한 트랜스포머의 병렬적이고 장거리 의존성 모델링 능력 10, 그리고 셋째, ViT가 제시한 패치 기반 처리를 통한 트랜스포머의 시각 데이터 적응력입니다.10 DiT는 이 세 가지 영역의 교차점에서 탄생했으며, LDM의 VAE를 활용하여 ViT의 접근법을 확산 과정에 계산적으로 적용 가능하게 만듦으로써 각 영역의 강점을 모두 계승한 강력한 종합체라 할 수 있습니다.5


확산 트랜스포머(DiT)의 등장은 단순히 새로운 아키텍처의 추가가 아닌, 생성 모델링 분야의 근본적인 패러다임 전환을 의미합니다. 이러한 전환의 배경을 이해하기 위해서는 기존의 지배적인 아키텍처였던 U-Net의 강점과 한계를 분석하고, 트랜스포머가 이를 어떻게 극복하며 새로운 가능성을 제시했는지 살펴보는 것이 중요합니다.


U-Net은 합성곱 신경망(CNN)을 기반으로 한 인코더-디코더 구조에 스킵 연결(skip connection)을 더한 형태로, DDPM이나 스테이블 디퓨전(Stable Diffusion)과 같은 초기 고성능 확산 모델에서 사실상의 표준 백본으로 사용되었습니다.5 U-Net의 계층적 구조는 다운샘플링과 업샘플링 경로를 통해 다중 스케일의 특징을 효과적으로 포착하며, 특히 스킵 연결은 인코더에서 디코더로 저수준의 세밀한 공간 정보를 직접 전달하여 이미지의 디테일을 보존하는 데 중요한 역할을 합니다.7

그러나 U-Net은 그 구조적 특성으로 인해 몇 가지 내재적 한계를 가집니다.

- **제한된 전역 컨텍스트(Global Context):** CNN 기반 아키텍처의 가장 큰 한계는 지역적 수용장(local receptive field)입니다. U-Net은 이미지의 멀리 떨어진 부분들 간의 장거리 의존성을 모델링하는 데 어려움을 겪습니다. 이를 극복하기 위해서는 네트워크를 매우 깊게 만들거나 큰 합성곱 커널을 사용해야 하지만, 이는 계산적으로 비효율적입니다.17 일부 U-Net 설계에서는 저해상도 특징 맵에 주의(attention) 블록을 삽입하기도 하지만, 그 핵심 연산은 여전히 지역적인 합성곱에 의존합니다.5
- **정적인 귀납적 편향(Inductive Bias):** U-Net은 합성곱 연산에서 비롯된 강력한 귀납적 편향(예: 지역성, 이동 등변성)을 내장하고 있습니다.7 이러한 편향은 특정 작업에서는 유용할 수 있지만, DiT 연구진은 이러한 편향이 확산 모델의 성능에 필수적이지 않으며, 더 일반적이고 확장 가능한 아키텍처가 더 나은 성능을 보일 수 있다는 가설을 증명하고자 했습니다.5
- **확장성 문제:** U-Net도 확장이 가능하지만, 트랜스포머는 NLP와 같은 다른 분야에서 모델 크기와 데이터 양을 늘릴수록 성능이 일관되게 향상되는, 더 예측 가능하고 유리한 확장성(scalability)을 보여주었습니다.5 DiT의 개발 동기 중 하나는 이러한 확장성 법칙이 시각 생성 모델에도 동일하게 적용될 것이라는 기대였습니다.


DiT로의 전환은 트랜스포머가 가진 몇 가지 근본적인 장점에 기반합니다.

- **우수한 확장성:** DiT의 핵심 동기는 트랜스포머의 입증된 확장성을 생성 모델에 도입하는 것이었습니다.5 연구 결과, 순방향 패스(forward pass)의 계산 복잡도(Gflops로 측정)와 생성된 샘플의 품질(FID 점수로 측정, 낮을수록 좋음) 사이에는 강력하고 예측 가능한 상관관계가 있음이 밝혀졌습니다.5 모델의 깊이(depth), 너비(width)를 늘리거나 입력 토큰의 수(즉, 더 작은 패치 크기 사용)를 증가시켜 Gflops를 높이면 FID 점수가 일관되게 향상되었습니다.5 이는 확산 모델이 대규모 언어 모델(LLM)에서 관찰된 것과 동일한 확장성 추세의 혜택을 받을 수 있음을 입증한 것입니다.5
- **효과적인 전역 컨텍스트 모델링:** 자기 주의 메커니즘은 이미지 내의 모든 패치(토큰)가 다른 모든 패치와 동시에 상호작용할 수 있게 합니다.7 이는 U-Net이 어려움을 겪는 전역 컨텍스트와 장거리 의존성을 본질적으로 포착하게 해주며, 일관성 있는 대규모 이미지를 생성하는 데 매우 중요합니다.17
- **아키텍처의 통일:** 특화된 U-Net을 표준적인 트랜스포머로 대체함으로써, 생성 모델링 분야를 AI 전반의 지배적인 아키텍처 트렌드와 일치시킬 수 있습니다.5 이러한 통일은 NLP나 컴퓨터 비전과 같은 다른 분야에서 축적된 모범 사례, 훈련 기법, 최적화 방법론을 상속받아 기술 발전을 가속화하는 효과를 가져옵니다.5

| 특징                         | U-Net 백본                                            | 트랜스포머 (DiT) 백본                         |      |
| ---------------------------- | ----------------------------------------------------- | --------------------------------------------- | ---- |
| **주요 연산**                | 합성곱 (Convolution)                                  | 자기 주의 (Self-Attention)                    |      |
| **수용장 (Receptive Field)** | 지역적 (Local)                                        | 전역적 (Global)                               |      |
| **장거리 의존성**            | 제한적, 계층적 구조에 의존                            | 본질적으로 강력함                             |      |
| **귀납적 편향**              | 지역성, 이동 등변성 (강함)                            | 순서 불변성 (약함, 위치 인코딩으로 보완)      |      |
| **확장성**                   | 양호하나, 예측 가능성 낮음                            | 매우 우수하며, 계산량과 성능이 비례 5         |      |
| **컨디셔닝 메커니즘**        | 특징 맵 결합, 시간 임베딩 주입                        | 적응형 계층 정규화 (adaLN), 교차 주의 등      |      |
| **핵심 장점**                | 효율적인 지역 특징 추출, 스킵 연결을 통한 디테일 보존 | 확장성, 전역 컨텍스트 모델링, 아키텍처 통일성 |      |
| **핵심 단점**                | 전역 컨텍스트 모델링 한계, 정적 귀납적 편향           | 2차 복잡도 (O(L2)), 공간적 편향 부재          |      |

표 1: U-Net과 트랜스포머 백본의 비교 분석. 이 표는 두 아키텍처의 근본적인 차이점과 장단점을 요약하여 DiT로의 전환 동기를 명확히 보여줍니다. 데이터 출처:.5


DiT는 잠재 확산 모델(LDM) 프레임워크 내에서 작동합니다. 노이즈가 섞인 잠재 벡터 $z$, 노이즈 타임스텝 $t$, 그리고 클래스 조건 $c$를 입력으로 받아 $z$의 노이즈 성분을 예측하도록 훈련됩니다.5


DiT의 처리 과정은 "패치화(patchify)" 계층에서 시작됩니다. 이 계층은 공간적 형태를 가진 잠재 벡터 $z$(예: 32x32x4)를 토큰 시퀀스로 변환하는 역할을 합니다.5

구체적으로, 잠재 맵의 겹치지 않는 패치들을 선형적으로 임베딩하여, 각각 $d$ 차원을 갖는 $T$개의 토큰 시퀀스를 생성합니다.5 여기서 토큰의 수 $T$는 패치 크기 $p$의 제곱에 반비례합니다. 예를 들어, 패치 크기 $p$를 절반으로 줄이면 토큰의 수는 4배로 증가하며, 이는 모델의 계산량(Gflops)을 최소 4배 증가시키는 직접적인 요인이 됩니다.5 이 패치화 과정 후에는 공간 정보를 제공하기 위해 표준적인 주파수 기반 위치 임베딩이 모든 토큰에 추가됩니다.5


토큰 시퀀스는 다중 헤드 자기 주의와 피드포워드 MLP 계층으로 구성된 일련의 표준 트랜스포머 블록을 통과합니다.5 DiT의 핵심 혁신 중 하나는 타임스텝 $t$와 클래스 레이블 c와 같은 조건부 정보를 주입하는 방식입니다. 여러 방법(인컨텍스트 컨디셔닝, 교차 주의 등)을 실험한 결과, **적응형 계층 정규화(adaptive layer normalization, adaLN)**가 가장 효과적이고 계산적으로 효율적인 것으로 밝혀졌습니다.3

표준 계층 정규화(LayerNorm)에서는 스케일($γ$)과 시프트($\beta$) 파라미터가 직접 학습됩니다. 반면, adaLN에서는 이 파라미터들이 조건부 정보(t와 c의 임베딩 합)로부터 MLP를 통해 동적으로 회귀(regress)됩니다. 이를 통해 조건부 정보가 전체 네트워크의 활성화(activation)를 조절할 수 있게 됩니다.5

가장 뛰어난 성능을 보인 변형인 **adaLN-Zero**는 여기서 한 걸음 더 나아갑니다. adaLN-Zero는 $γ$와 $β$ 외에도, 각 잔차 연결(residual connection) 직전에 적용되는 추가적인 스케일링 파라미터 $α$를 회귀합니다. 중요한 점은, $α$를 생성하는 MLP가 초기에 $0$을 출력하도록 초기화된다는 것입니다. 이는 각 DiT 블록이 훈련 시작 시점에서 항등 함수(identity function)처럼 작동하게 만들어, 매우 깊은 네트워크의 훈련을 안정화시키고 최종 모델의 품질을 크게 향상시키는 것으로 나타났습니다.5

DiT 아키텍처가 "순수 트랜스포머" 기반으로 소개되곤 하지만, 실제 전체 생성 파이프라인은 잠재 공간을 위해 합성곱 기반의 VAE에 여전히 의존하고 있다는 점을 주목해야 합니다.7 이는 DiT가 컨볼루션의 완전한 배제가 아니라, 전역 컨텍스트 모델링과 확장성이 가장 중요한 **노이즈 제거 백본**을 전략적으로 대체한 결과임을 보여줍니다. 즉, 초기 DiT의 최적 시스템은 ConvNet과 트랜스포머의 장점을 결합한 실용적인 하이브리드 형태였습니다.

또한, adaLN-Zero의 개발은 단순한 점진적 개선이 아니라, 깊은 DiT의 안정적인 훈련을 가능하게 한 결정적인 요인이었습니다. 초기 블록을 항등 함수로 강제하는 제로 초기화 전략은 매우 깊은 잔차 네트워크에서 흔히 발생하는 기울기 소실/폭주 문제를 직접적으로 해결합니다. 이 작지만 중요한 아키텍처적 조정이 없었다면, DiT-XL과 같은 대규모 모델이 SOTA 성능을 달성하는 데 필요한 인상적인 확장성 법칙을 실현하기 어려웠을 것입니다. 이는 근본적인 최적화 문제를 해결함으로써 대규모 확장의 문을 연 것입니다.23


DiT 아키텍처의 이론적 우수성은 실제 실험을 통해 입증되었습니다. 모델의 계산량과 성능 간의 명확한 관계, 즉 확장 법칙(scaling law)의 발견은 DiT의 잠재력을 확인시켜 주었으며, 이는 곧 벤치마크에서의 압도적인 성능으로 이어졌습니다. 그러나 이러한 성능은 막대한 계산 비용이라는 대가를 수반했으며, 이는 자연스럽게 효율성 향상을 위한 다양한 최적화 연구로 이어졌습니다.


DiT에 관한 초기 연구는 확산 모델에 대한 강력하고 예측 가능한 확장 법칙을 확립했습니다.5 순방향 패스에 사용되는 계산량(Gflops로 측정)과 생성된 샘플의 품질(FID 점수로 측정, 낮을수록 좋음) 사이에는 뚜렷한 로그-선형(log-linear) 관계가 존재함이 밝혀졌습니다.5

계산량(Gflops)을 증가시키는 주된 방법은 세 가지입니다:

1. **트랜스포머 깊이 증가:** 더 많은 DiT 블록(N)을 쌓습니다.
2. **트랜스포머 너비 증가:** 은닉층의 차원 크기(d)를 늘립니다.
3. **입력 토큰 증가:** 더 작은 패치 크기(p)를 사용하여 입력 토큰의 수를 늘립니다. 패치 크기를 절반으로 줄이면 토큰 수는 4배, Gflops도 4배 이상 증가합니다.5

주목할 만한 발견은 이 세 가지 방법 모두 FID 점수를 개선했다는 점입니다. 특히, 작은 모델을 더 오래 훈련시키는 것보다 큰 모델을 더 짧게 훈련시키는 것이 계산적으로 더 효율적이라는 사실이 밝혀졌습니다.7 이는 DiT 아키텍처에 대해 "더 큰 것이 더 좋다"는 가설을 뒷받침합니다.

최근에는 최대 업데이트 파라미터화(Maximal Update Parametrization, µP) 기법이 DiT에 일반화되어 적용되었습니다. $µP$는 작은 모델에서 튜닝된 하이퍼파라미터(예: 학습률)를 큰 모델로 안정적으로 이전할 수 있게 해줍니다. 이는 대규모 DiT를 튜닝하는 비용을 극적으로 줄이고 수렴을 가속화하는 효과를 가져옵니다(예: DiT-XL-2-µP는 2.9배 더 빠르게 수렴). 이로써 DiT의 확장이 더욱 원칙적이고 효율적으로 이루어질 수 있게 되었습니다.25


DiT의 확장성은 ImageNet과 같은 대규모 벤치마크에서 SOTA(State-of-the-Art) 성능을 달성함으로써 그 가치를 증명했습니다.

| 모델          | 벤치마크             | FID ↓    | 모델 유형  | 연도     |      |
| ------------- | -------------------- | -------- | ---------- | -------- | ---- |
| **DiT-XL/2**  | **ImageNet 256x256** | **2.27** | **DiT**    | **2022** |      |
| StyleGAN-XL   | ImageNet 256x256     | 2.30     | GAN        | 2022     |      |
| LDM-4-G       | ImageNet 256x256     | 3.60     | U-Net 확산 | 2022     |      |
| ADM-G         | ImageNet 256x256     | 4.59     | U-Net 확산 | 2021     |      |
| BigGAN-deep   | ImageNet 256x256     | 6.95     | GAN        | 2018     |      |
| **DiT-XL/2**  | **ImageNet 512x512** | **3.04** | **DiT**    | **2022** |      |
| StyleGAN-XL   | ImageNet 512x512     | 2.41     | GAN        | 2022     |      |
| ADM-G + ADM-U | ImageNet 512x512     | 3.85     | U-Net 확산 | 2021     |      |

표 2: ImageNet 벤치마크에서의 SOTA FID 점수 비교. 이 표는 DiT-XL/2가 이전의 U-Net 기반 확산 모델 및 GAN을 능가하여 새로운 SOTA를 달성했음을 명확하게 보여줍니다. 데이터 출처:.5

- **ImageNet 256x256:** 가장 큰 모델인 DiT-XL/2는 클래스 조건부 ImageNet 256x256 벤치마크에서 **FID 2.27**이라는 SOTA 점수를 기록했습니다.5 이는 이전의 모든 확산 모델, 즉 ADM-G(FID 4.59) 및 LDM-4-G(FID 3.60)는 물론, StyleGAN-XL(FID 2.30)과 같은 최고 성능의 GAN 모델까지 능가하는 결과였습니다.5
- **ImageNet 512x512:** 512x512 벤치마크에서도 DiT-XL/2는 **FID 3.04**를 기록하며 확산 모델 분야에서 새로운 SOTA를 세웠고, 이전 최고 기록인 ADM-G+ADM-U(FID 3.85)를 넘어섰습니다.5

이러한 결과는 트랜스포머 기반 백본이 고도로 최적화된 기존의 U-Net 아키텍처를 능가할 수 있음을 증명한 기념비적인 성과였습니다.18 이로써 DiT 아키텍처는 고품질 이미지 생성을 위한 새로운 기반으로 확고히 자리매김했습니다.


DiT의 뛰어난 성능과 확장성은 막대한 계산 비용을 대가로 합니다. 이는 대규모 DiT의 훈련과 배포에 있어 심각한 병목 현상을 야기합니다.

- **훈련 비용:** 대규모 DiT의 훈련은 엄청나게 비쌉니다.
  - 원본 DiT-XL/2 모델은 256x256 ImageNet 훈련에 **950 V100 GPU-일**(7백만 이터레이션), 512x512 훈련에 **1733 V100 GPU-일**(3백만 이터레이션)이 필요했습니다.29
  - 비교를 위해, U-Net 기반의 Stable Diffusion 1.5는 약 60만 달러에 해당하는 150,000 A100 GPU-시간이 소요되었습니다.30 최근의 Seaweed-7B와 같은 70억 파라미터 규모의 비디오 DiT 모델은 **665,000 H100 GPU-시간**을 필요로 했습니다.31
  - 이러한 비용은 대부분의 연구자나 개발자가 처음부터 모델을 훈련하는 것을 거의 불가능하게 만듭니다.29
- **추론 지연 시간:** 확산 모델의 반복적인 샘플링 과정과 트랜스포머 백본의 복잡성이 결합되어 추론 속도가 매우 느립니다.
  - 자기 주의 메커니즘의 2차 복잡도(O(L2))는 토큰 수 L에 따라 계산량이 기하급수적으로 증가하는 주요 원인입니다. 이는 많은 토큰을 필요로 하는 고해상도 이미지 생성에서 특히 심각한 병목이 됩니다.17 고성능 GPU에서도 512x512 해상도 이미지 한 장을 생성하는 데 20초 이상 걸릴 수 있으며 35, 2K 해상도에서는 주의 메커니즘 계산에만 수 초가 소요될 수 있습니다.33
  - 이러한 높은 계산 요구량은 특히 리소스가 제한된 엣지 디바이스에서의 실용적인 배포를 어렵게 만듭니다.36

| 모델                     | 아키텍처 유형 | 해상도    | 훈련 데이터 규모 | GPU 유형 | GPU-일 / 시간                   |      |
| ------------------------ | ------------- | --------- | ---------------- | -------- | ------------------------------- | ---- |
| **DiT-XL/2**             | DiT           | 256x256   | ImageNet         | V100     | **950 GPU-일**                  |      |
| **DiT-XL/2**             | DiT           | 512x512   | ImageNet         | V100     | **1733 GPU-일**                 |      |
| **Stable Diffusion 1.5** | U-Net         | 512x512   | LAION-5B         | A100     | **~6,250 GPU-일** (150k GPU-h)  |      |
| **PixArt-α**             | DiT           | 1024x1024 | 혼합             | A100     | **~675 GPU-일**                 |      |
| **Seaweed-7B**           | DiT           | 비디오    | 비디오/이미지    | H100     | **~27,708 GPU-일** (665k GPU-h) |      |

표 3: 주요 확산 모델의 훈련 비용 분석. 이 표는 DiT와 같은 대규모 모델 훈련에 필요한 막대한 계산 자원을 정량적으로 보여주며, PixArt-α와 같은 효율적인 훈련 전략의 중요성을 강조합니다. 데이터 출처:.29


이러한 계산 비용 문제를 해결하기 위해 다양한 최적화 전략이 제안되었습니다.


PixArt-α는 훈련 비용을 획기적으로 줄이면서도 높은 품질을 유지하기 위해 새로운 3단계 훈련 분해 전략을 도입했습니다.37

- **1단계 (픽셀 의존성 학습):** ImageNet으로 사전 훈련된 클래스 조건부 모델로 초기화하여 기본적인 이미지 사전 지식(prior)을 학습합니다.37
- **2단계 (텍스트-이미지 정렬 학습):** LLaVA와 같은 비전-언어 모델을 활용한 자동 레이블링을 통해 생성된, 정보 밀도가 높은 고품질 캡션 데이터를 사용하여 텍스트와 이미지 간의 의미론적 정렬을 효율적으로 학습합니다.37
- **3단계 (미적 품질 향상):** 더 작지만 품질이 높은 미적 데이터셋으로 미세 조정(fine-tuning)하여 최종 결과물의 품질을 향상시킵니다.37

이 전략을 통해 대규모 텍스트-이미지(T2I) 모델의 훈련 비용을 단 **675 A100 GPU-일**로 줄였으며, 이는 Stable Diffusion v1.5 훈련 대비 약 90%의 비용 절감 효과를 보였습니다.37


- **훈련 후 양자화 (Post-Training Quantization, PTQ):** 모델의 가중치와 활성화를 저비트(low-bit) 형식(예: 8비트 또는 4비트)으로 변환하여 모델 크기를 줄이고 추론을 가속화하는 기술입니다. 비용이 많이 드는 재훈련이 필요 없다는 장점이 있습니다.35
- **DiT에 대한 도전 과제:** DiT에 PTQ를 적용하는 것은 간단하지 않습니다. (1) 극단적인 값 범위를 갖는 "돌출 채널(salient channels)"의 등장과 (2) 노이즈 제거 과정의 타임스텝에 따라 이러한 활성화 분포가 동적으로 변하는 문제 때문입니다.35
- **해결책 (PTQ4DiT):** PTQ4DiT 연구에서는 **채널별 돌출성 균형(Channel-wise Salience Balancing, CSB)** 기법을 제안했습니다. 이는 가중치와 활성화 사이에서 극단적인 값을 재분배하여 이러한 문제를 완화하고, DiT에 대한 효과적인 8비트(W8A8) 및 4비트(W4A8) 양자화를 가능하게 합니다.35


- **핵심 아이디어:** 노이즈 제거 과정에 필요한 계산량은 균일하지 않습니다. 초기 타임스텝(노이즈가 많은)에서는 전역 구조를 모델링하기 위해 더 많은 용량이 필요할 수 있고, 후기 타임스텝(노이즈가 적은)에서는 미세한 디테일에 집중합니다. 마찬가지로, 평평한 하늘과 같은 이미지 패치는 복잡한 질감의 패치보다 노이즈 제거가 더 쉽습니다.42
- **DyDiT (Dynamic Diffusion Transformer):** 이 아키텍처는 생성 과정에서 타임스텝과 공간적 차원 모두에서 모델의 계산을 동적으로 조정할 것을 제안합니다. **타임스텝별 동적 너비(Timestep-wise Dynamic Width, TDW)**를 사용하여 모델 너비를 조절하고, **공간별 동적 토큰(Spatial-wise Dynamic Token, SDT)** 전략을 통해 "쉬운" 패치에 대한 계산을 건너뜁니다. 이 방법은 최소한의 품질 저하로 DiT-XL의 FLOPs를 51%까지 줄였습니다.42

DiT의 인상적인 확장 법칙은 막대한 계산 비용이라는 "확장세(scaling tax)"를 동반합니다. 이는 AI 연구 커뮤니티 내에 근본적인 긴장감을 조성했습니다. 한편에서는 거대 산업 연구소들이 순수한 규모 확장을 통해 SOTA 품질을 추구하고, 다른 한편에서는 학계와 스타트업들이 기술 접근성을 높이기 위해 효율성 혁신에 집중하는 양상이 나타났습니다. 이 긴장감은 DiT 최적화라는 하위 연구 분야 전체를 이끄는 원동력이 되었습니다.

최적화 전략의 진화는 초기의 무차별적인 확장(brute-force scaling)에서 벗어나 모델과 생성 과정 자체의 중복성을 이해하고 활용하는, 보다 미묘하고 지능적인 접근 방식으로 나아가고 있음을 보여줍니다. 초기 성공은 단순히 모델을 더 크게 만드는 것에서 비롯되었지만 5, PixArt-α와 같은 연구는 복잡한 T2I 학습 문제를 더 간단한 순차적 단계로 분해하는 

*커리큘럼*의 개념을 도입했습니다.37 DyDiT는 한 걸음 더 나아가 노이즈 제거 과정 자체를 분석하여 계산이 모든 곳에서 또는 모든 시간에 동일하게 필요하지 않다는 것을 발견했습니다.42 이는 모델에 대한 정적이고 단일체적인 관점에서 동적이고 적응적인 관점으로의 전환을 의미하며, 미래의 발전이 단순히 "더 큰" 모델이 아닌 "더 똑똑한" 훈련 및 추론 방식에서 비롯될 것임을 시사합니다.


표준 DiT 아키텍처의 성공은 수많은 변형과 개선을 촉발하는 기폭제가 되었습니다. 연구자들은 확장성과 귀납적 편향 사이의 지속적인 탐구를 통해, 표준 설계를 강화하고, U-Net의 원리를 재통합하며, 심지어 트랜스포머를 넘어서는 새로운 시퀀스 모델을 탐색하는 등 풍부하고 진화하는 아키텍처 환경을 만들어냈습니다.


- **희소 아키텍처 (DiT-MoE):** 추론 비용의 비례적 증가 없이 모델을 수십억 파라미터 규모로 확장하기 위해 전문가 혼합(Mixture-of-Experts, MoE) 계층이 DiT에 통합되었습니다. DiT-MoE는 라우터(router)를 사용하여 각 토큰에 대해 "전문가" MLP 블록의 일부만 선택적으로 활성화함으로써, 파라미터 수를 극적으로 늘리면서도 추론 FLOPs를 낮게 유지합니다. 이를 통해 165억 개 이상의 파라미터를 가진 모델이 새로운 SOTA FID 점수(1.80)를 달성했습니다.45
- **유연한 해상도 (FiTv2):** 표준 DiT는 고정된 해상도에서 훈련됩니다. FiTv2는 이미지를 동적 크기의 토큰 시퀀스로 개념화하여, 모델이 재훈련 없이도 다양한 해상도와 종횡비의 이미지를 생성할 수 있게 함으로써 이미지 잘라내기(cropping)로 인한 편향을 제거합니다.46
- **다중모드 융합 (MMDiT):** 스테이블 디퓨전 3(Stable Diffusion 3)에 사용된 다중모드 확산 트랜스포머(Multimodal Diffusion Transformer, MMDiT)는 이미지와 텍스트 같은 여러 양식을 더 효과적으로 처리하도록 DiT 아키텍처를 개선했습니다. 이 모델은 이미지 토큰과 텍스트 토큰에 대해 별도의 가중치 세트를 사용하지만, 이들이 주의(attention) 공간을 공유하도록 하여 텍스트 이해도와 타이포그래피 생성 능력을 향상시켰습니다.13


원본 DiT의 등방성(isotropic, 균일한) 아키텍처는 인상적인 확장성을 보여주었지만, 느린 수렴 속도와 특히 캐싱과 같은 추론 가속 기법 사용 시 발생하는 동적 특징 불안정성(dynamic feature instability)과 같은 문제점을 드러냈습니다.47 반면, U-Net의 계층적 구조와 장거리 스킵 연결(long-skip connections)은 다중 스케일 특징 집계와 안정적인 기울기 흐름에 도움이 되는 것으로 알려져 있었습니다.48

- **U-DiTs (U자형 확산 트랜스포머):** 이 연구는 DiT에 U-Net과 유사한 계층적 구조를 다시 도입했습니다. 단순히 구조를 추가하는 것만으로는 미미한 이점만 있다는 것이 초기 발견이었습니다.50 결정적인 혁신은 **자기 주의 메커니즘 내에서의 토큰 다운샘플링**(Q, K, V 모두에 적용)이었습니다. 이는 U-Net의 특징이 저주파수 위주라는 관찰과 일치하는 저역 통과 필터(low-pass filter) 역할을 하며 계산량을 대폭 줄였습니다.50 그 결과, U-DiT 모델은 DiT-XL/2의 1/6에 불과한 계산 비용으로 더 나은 성능을 달성할 수 있었습니다.50
- **Skip-DiT:** 이 아키텍처는 DiT의 "동적 특징 불안정성" 문제를 직접적으로 해결합니다. 이 문제는 노이즈 제거 과정에서 특징 분포가 불규칙하게 변동하여 특징 캐싱과 같은 효율성 기법을 사용할 때 오류가 증폭되는 현상을 말합니다.47 Skip-DiT는 U-Net의 특징인 **장거리 스킵 연결(Long-Skip-Connections, LSCs)**을 얕은 DiT 블록과 깊은 DiT 블록 사이에 도입합니다. 이러한 LSC는 이론적으로나 실험적으로 특징 역학 및 기울기 흐름을 안정화시켜, 상당한 훈련 가속(최대 4.4배) 및 추론 가속(1.5~2배)을 가능하게 합니다.47


자기 주의의 2차 복잡도는 DiT 및 그 변형을 포함한 모든 트랜스포머 기반 모델의 근본적인 확장성 병목으로 남아있습니다.33

- **Mamba와 상태 공간 모델(State Space Models, SSMs):** Mamba는 구조화된 상태 공간 모델에 기반한 새로운 아키텍처로, 시퀀스 모델링 작업에서 트랜스포머와 경쟁력 있는 성능을 보이면서도 **선형 시간 복잡도**(O(L))를 가집니다.56
- **확산 Mamba (DiM):** 이 아키텍처는 확산 모델의 트랜스포머 블록을 Mamba 블록으로 대체합니다.54 1차원적이고 인과적인(causal) Mamba를 2차원의 비인과적인 이미지 데이터에 적응시키는 것이 핵심 과제입니다. DiM은 **다방향 스캐닝**(패치 시퀀스를 순방향, 역방향 등으로 처리하여 전역 수용장을 형성) 및 학습 가능한 패딩 토큰과 같은 아키텍처 수정을 통해 이 문제를 해결합니다.55 DiM은 훨씬 더 효율적인 고해상도 이미지 합성을 약속합니다.54
- **하이브리드 접근법 (DiMSUM):** Mamba가 지역적/인과적 모델링에 능하고 트랜스포머가 전역적 상호작용에 능하다는 점을 인식하여 하이브리드 모델이 등장하고 있습니다. DiMSUM은 Mamba 블록과 전역적으로 공유되는 트랜스포머 블록을 결합하고, 웨이블릿 변환(wavelet transform)을 사용하여 주파수 영역에서 이미지를 처리함으로써 지역적 및 장거리 모델링을 모두 향상시킵니다.61

DiT 아키텍처의 진화는 두 가지 설계 철학 사이의 흥미로운 진자 운동을 보여줍니다: (1) 순수하고 등방성인 확장(원본 DiT)과 (2) 귀납적 편향을 가진 구조화된 계층적 설계(U-Net). 초기에는 U-Net의 사전 지식을 거부하고 순수 확장에 크게 기울었습니다. 그러나 U-DiT와 Skip-DiT의 등장은 CNN의 '오래된' 아이디어들이 순수 확장으로 인해 발생한 안정성과 효율성이라는 '새로운' 문제를 해결하는 데 중요하다는 것을 인정하며, 종합을 향한 회귀를 나타냅니다. 이는 단순히 U-Net으로 돌아가는 것이 아니라, U-Net의 가장 가치 있는 원리들을 더 확장 가능한 트랜스포머 프레임워크에 지능적으로 통합하는 과정입니다.

또한, Mamba 기반 확산 모델(DiM)의 부상은 단순히 한 블록을 다른 블록으로 교체하는 것을 넘어, 생성 모델을 위한 새로운 아키텍처 기본 단위(primitive)를 확립하고 있습니다. DiMSUM과 같은 하이브리드 Mamba-트랜스포머 모델의 즉각적인 등장은 미래가 단일한 "Mamba 세계"나 "트랜스포머 세계"가 아니라, 각기 다른 블록이 가장 잘하는 작업에 사용되는 유연한 생태계가 될 것임을 시사합니다. 이는 Mamba 블록이 대부분의 지역적, 선형 시간 처리를 담당하고, 계산적으로 비싼 소수의 트랜스포머 블록이 중요한 전역 정보 융합을 처리하는 모듈식, 이기종 백본의 미래를 예견하게 합니다.

| 모델 변형    | 핵심 혁신                     | 해결 과제                            | 주요 아키텍처 변경                         | 주요 이점                         |      |
| ------------ | ----------------------------- | ------------------------------------ | ------------------------------------------ | --------------------------------- | ---- |
| **표준 DiT** | 트랜스포머 백본 도입          | U-Net의 확장성 및 전역 컨텍스트 한계 | U-Net을 트랜스포머 블록으로 대체           | 우수한 확장성, SOTA 성능          |      |
| **DiT-MoE**  | 전문가 혼합 (MoE)             | 대규모 모델의 추론 비용              | MLP 계층을 희소 MoE 계층으로 대체          | 파라미터 수 대비 낮은 추론 비용   |      |
| **FiTv2**    | 동적 토큰화                   | 고정 해상도 제약                     | 다양한 종횡비/해상도 훈련 및 추론 지원     | 해상도 유연성, 편향 감소          |      |
| **MMDiT**    | 다중모드 주의                 | 텍스트-이미지 융합 한계              | 이미지와 텍스트 토큰의 주의 공간 공유      | 향상된 텍스트 이해 및 제어        |      |
| **U-DiT**    | U-Net 구조 및 토큰 다운샘플링 | DiT의 느린 수렴 및 비효율성          | 계층적 구조 및 주의 메커니즘 내 다운샘플링 | 계산 비용 대폭 감소, 성능 향상    |      |
| **Skip-DiT** | 장거리 스킵 연결 (LSCs)       | 동적 특징 불안정성                   | 얕은 블록과 깊은 블록 간 LSC 추가          | 훈련/추론 안정성 및 속도 향상     |      |
| **DiM**      | Mamba (SSM) 백본              | 트랜스포머의 2차 복잡도              | 트랜스포머 블록을 Mamba 블록으로 대체      | 선형 시간 복잡도, 고해상도 효율성 |      |
| **DiMSUM**   | 하이브리드 Mamba-트랜스포머   | Mamba의 전역 모델링 한계             | Mamba 블록과 공유 트랜스포머 블록 결합     | 지역 및 전역 모델링의 장점 결합   |      |

표 4: 주요 DiT 아키텍처 변형 개요. 이 표는 DiT의 진화 과정을 요약하며, 각 변형이 어떤 문제를 해결하기 위해 어떤 혁신을 도입했는지 보여줍니다. 데이터 출처:.21


DiT 아키텍처의 놀라운 다재다능함은 정적인 이미지 생성을 넘어 비디오, 3D, 로봇 공학, 분자 과학 등 광범위한 데이터 양식과 작업에 성공적으로 적용되면서 입증되고 있습니다. 이는 DiT가 특정 데이터에 국한되지 않는 범용적인 생성 모델링 프레임워크로서의 잠재력을 가지고 있음을 보여줍니다.


OpenAI의 Sora는 트랜스포머 아키텍처를 백본으로 사용하는 확산 모델로, 본질적으로 대규모 DiT에 해당합니다.8 Sora의 핵심 혁신은 "시공간 잠재 패치(spacetime latent patches)"라는 독특한 데이터 표현 방식에 있습니다.8

1. **비디오 압축:** 먼저 비디오 압축 네트워크(VAE로 추정)가 원본 비디오를 저차원의 잠재 공간으로 인코딩합니다.8
2. **시공간 패치화:** 이 잠재 표현은 공간적 차원과 시간적 길이를 모두 포함하는 패치, 즉 "튜브릿(tubelet)" 시퀀스로 분해됩니다.64
3. **트랜스포머 입력:** 이 시공간 패치들이 확산 트랜스포머의 입력 토큰으로 사용되어, 모델이 시각적 외형과 움직임 역학을 동시에 모델링할 수 있게 합니다.64

이 접근법 덕분에 Sora는 다양한 길이, 해상도, 종횡비의 비디오를 처리할 수 있으며, 일관된 객체와 물리 법칙을 유지하는 길고 논리적인 비디오를 생성할 수 있는 능력의 기반이 됩니다.8 이는 DiT의 핵심 패러다임인 '패치화 후 주의(patch-and-attend)'가 2D 공간을 넘어 3D 시공간으로 일반화될 수 있음을 보여주는 강력한 증거입니다.


- **로봇 공학 (확산 트랜스포머 정책):** DiT는 일반화된 로봇 정책(generalist robot policy)을 만드는 데 사용되고 있습니다. 단일 행동을 예측하는 대신, **확산 트랜스포머 정책(Diffusion Transformer Policy)**은 연속적인 행동의 전체 *덩어리(chunk)*를 노이즈 제거하는 방식으로 작동합니다.71 트랜스포머 백본은 정책이 다양한 로봇 데이터셋(서로 다른 로봇, 작업, 카메라 시점)에 걸쳐 효과적으로 확장되고 이전 방법보다 더 잘 일반화되도록 합니다.71 이 정책은 이미지 관찰 패치에 직접적으로 행동 노이즈 제거를 조건화하여, 미묘한 시각적 뉘앙스를 인지할 수 있습니다.71
- **3D 형태 생성 (DiT-3D & DiffSurf):**
  - **DiT-3D**는 3D 포인트 클라우드(point cloud) 생성을 위해 DiT 아키텍처를 변형한 모델입니다.72 복셀화된(voxelized) 포인트 클라우드에 3D 위치 및 패치 임베딩을 적용하여 작동합니다. 3차원으로 인해 토큰 수가 세제곱으로 증가하는 계산 문제를 해결하기 위해 3D 윈도우 주의(window attention)를 통합하여 계산 비용을 줄였습니다.72 이 모델은 고품질 3D 형태 생성에서 SOTA 성능을 보였습니다.72
  - **DiffSurf**는 3D 표면 메시(surface mesh, 정점과 법선)를 생성하고 재구성하기 위한 DiT 기반 모델입니다.73 인체, 동물, 인공물과 같은 다양한 객체를 생성할 수 있으며, 메시 피팅이나 모핑과 같은 다운스트림 작업에도 활용될 수 있습니다.73


기존의 원자 시스템 생성 모델은 동일한 물리 법칙을 공유함에도 불구하고 분자(비주기적)나 결정(주기적) 시스템에 대해 고도로 특화되어 있었습니다.74

**ADiT (All-atom Diffusion Transformer)**는 DiT를 사용하여 분자와 재료를 **모두** 공유된 잠재 공간에서 생성하는 통합된 잠재 확산 프레임워크입니다.74

- **1단계 (VAE):** 트랜스포머 기반 VAE가 두 시스템 유형의 모든 원자 표현에 대한 공유 잠재 임베딩을 학습합니다.74
- **2단계 (DiT):** DiT가 이 통합된 잠재 공간에서 새로운 임베딩을 생성하도록 훈련되며, VAE 디코더는 이를 새로운 분자나 결정으로 변환합니다.74

이 통합 접근법은 전문화된 모델을 능가하는 SOTA 결과를 달성했으며, 주기적 시스템과 비주기적 시스템 간의 성공적인 전이 학습(transfer learning)을 보여주었습니다. 또한 계산적으로 비싼 등변성(equivariant) 네트워크 대신 표준 트랜스포머를 사용함으로써, ADiT는 추론 속도가 10배 이상 빠르고 확장성이 매우 뛰어납니다.75


대규모 T2I 확산 모델(DiT 포함)이 생성적 사전 훈련 중에 학습한 강력한 시각적 표현은 의미론적 분할(semantic segmentation), 깊이 추정(depth estimation), 분류(classification)와 같은 다운스트림 인식(perception) 작업으로 전이될 수 있습니다.78

**GenPercept 방법론**은 완전하고 느린 다단계 생성 과정을 사용하는 대신, 사전 훈련된 확산 모델의 U-Net 또는 트랜스포머 백본을 인식 작업을 위해 직접 미세 조정할 것을 제안합니다.78 이는 작업을 결정론적인 단일 단계 예측으로 재구성하여, 수십억 개의 이미지에서 학습된 풍부한 특징을 활용하는 방식입니다.78 여기서 확산 백본은 강력한 특징 인코더 역할을 합니다.79

이러한 접근은 ImageNet 분류와 같은 전통적인 사전 훈련 패러다임에 도전합니다. 이는 대규모의 약하게 레이블링된 웹 데이터(스테이블 디퓨전 등에 사용된)에 대한 생성적 사전 훈련이, 판별적 사전 훈련보다 더 강력하고 일반화 가능한 시각적 모델을 생성할 수 있음을 시사합니다. 소량의 데이터로 미세 조정했을 때도 놀라운 성능을 보여주기 때문입니다.78

DiT가 비디오, 3D, 로봇 공학, 분자 구조 등 다양한 분야에 성공적으로 적용된 것은 '패치'라는 개념이 놀라울 정도로 보편적이고 강력한 추상화임을 드러냅니다. 2D 잠재 이미지의 패치든, 3D 비디오의 '튜브릿'이든, 3D 복셀이든, 심지어 속성을 가진 개별 원자이든, 핵심 아키텍처 원리는 동일합니다: 복잡한 개체를 토큰화된 구성 요소의 시퀀스로 분해하고 트랜스포머를 사용하여 그들 간의 상호 관계를 모델링하는 것입니다. 이는 DiT가 단순한 이미지 모델이 아니라, 상호작용하는 부분들로 이루어진 시스템을 모델링하기 위한 일반적인 아키텍처임을 시사합니다.

더 나아가, 인식 작업을 위해 확산 백본을 사용하는 성공 사례는 컴퓨터 비전 분야의 패러다임 전환 가능성을 암시합니다. 수년간 표준이었던 판별적 작업(ImageNet 분류) 기반의 사전 훈련 방식에서, 대규모 웹 데이터에 대한 생성적 사전 훈련이 더 강력하고 일반화 가능한 특징 표현을 생성한다는 증거가 쌓이고 있습니다. 노이즈로부터 전체 이미지를 재구성하는 생성 작업이, 이미지를 단순히 분류하는 것보다 더 포괄적이고 우수한 사전 훈련 과제일 수 있다는 것입니다. 즉, 이미지를 *만드는* 법을 배우는 것이 이미지를 *이해하는* 더 나은 방법일 수 있다는 가능성을 제기합니다.


DiT 기반 생성 모델의 강력한 성능과 확산은 사회 전반에 걸쳐 심대한 영향을 미치고 있습니다. 이러한 기술의 발전은 창의적인 가능성을 열어주는 동시에, 오용될 경우 심각한 윤리적, 사회적 위험을 초래할 수 있습니다. 따라서 기술 개발과 함께 책임감 있는 배포를 위한 기술적, 정책적 프레임워크를 구축하는 것이 시급한 과제입니다.


- **허위 정보와 딥페이크:** Sora와 같이 DiT 기술로 구동되는 모델의 초현실적인 결과물은 허위 정보 유포, 여론 조작, 개인의 명예 훼손 등에 사용될 수 있는 기만적인 콘텐츠(딥페이크)를 만드는 강력한 도구가 됩니다.84 간단한 텍스트 프롬프트만으로 설득력 있는 가짜 비디오를 쉽게 생성할 수 있게 되면서 악의적인 행위자들의 진입 장벽이 크게 낮아졌습니다.87
- **데이터 편향과 유해한 고정관념:** 확산 모델은 종종 정제되지 않은 방대한 웹 규모의 데이터셋으로 훈련됩니다. 이러한 데이터셋은 기존 사회의 편향을 그대로 반영하며, 모델은 이를 학습하고 영속화하여 특정 집단에 대한 고정관념적이거나 배제적인 콘텐츠를 생성할 수 있습니다.84 예를 들어, 특정 인종이나 성별에 편중된 데이터로 훈련된 모델은 소외된 집단을 정확하게 표현하는 데 어려움을 겪을 수 있습니다.84
- **저작권 및 지적 재산권:** 훈련 데이터셋에는 종종 창작자의 명시적인 동의 없이 인터넷에서 스크랩한 저작권 보호 자료가 포함됩니다. 이는 모델이 특정 예술가의 스타일을 모방하거나 훈련 이미지의 일부를 복제할 수 있기 때문에 저작권 침해에 대한 심각한 법적, 윤리적 문제를 야기합니다.84


이러한 위험에 대응하기 위해 다양한 기술적 완화 방안이 개발되고 있습니다.

- **딥페이크 탐지:** AI 생성 콘텐츠를 식별하기 위한 기술적 방법들은 다음과 같습니다.
  - **인공물 분석(Artifact Analysis):** AI 모델이 종종 생성하는 부자연스러운 조명, 가장자리의 이상한 흐림, 음성과 입 모양의 불일치, 손가락의 기형 등 불일치성을 분석합니다.88
  - **행동 생체 인식(Behavioral Biometrics):** 화상 통화의 경우, 인간의 미묘한 시선 패턴(단속성 운동), 목소리의 미세 떨림, 심지어 픽셀의 혈류 신호(광혈류측정법)와 같이 AI가 완벽하게 복제하기 어려운 인간의 뉘앙스를 분석합니다.88
  - **한계:** 탐지 기술은 생성 모델과의 "군비 경쟁"과 같습니다. 생성 모델이 발전함에 따라 그들이 남기는 인공물은 점점 더 미묘해져 탐지가 갈수록 어려워집니다.87
- **디지털 출처 증명 (C2PA 표준):** 보다 선제적인 접근 방식은 콘텐츠 생성 파이프라인에 직접 출처 정보를 내장하는 것입니다.
  - **C2PA (콘텐츠 출처 및 신뢰성을 위한 연합):** 디지털 콘텐츠에 대한 검증 가능한 이력을 제공하기 위해 설계된 개방형 기술 표준입니다.91
  - **작동 방식:** C2PA 호환 도구로 콘텐츠를 생성하거나 편집할 때, 암호학적으로 서명된 메타데이터 "매니페스트(manifest)"가 파일에 안전하게 바인딩됩니다. 이 매니페스트는 누가 또는 무엇이(예: 특정 AI 모델) 콘텐츠를 만들었는지, 언제 만들어졌는지, 어떤 편집이 이루어졌는지 등을 상세히 기록하는 "영양 성분표"처럼 작동합니다.92
  - **AI에의 적용:** AI 모델은 자신의 출력물에 서명하여 해당 콘텐츠가 합성되었음을 명시할 수 있습니다. 이는 콘텐츠가 "진실"인지 "거짓"인지를 판단하는 것이 아니라, 그 출처에 대한 투명하고 위변조가 불가능한 정보를 제공하여 소비자가 더 정보에 입각한 판단을 내릴 수 있도록 돕습니다.92

딥페이크 탐지를 위한 수동적인 방법론은 장기적으로 승리하기 어려운 군비 경쟁의 성격을 띱니다. 생성 모델이 현실을 거의 완벽하게 모방하게 되면, 탐지기가 의존하는 통계적 인공물은 사라질 것입니다. 이는 C2PA와 같은 선제적이고 암호학적으로 안전한 출처 증명 표준이 단순히 '있으면 좋은 것'이 아니라, 공유된 현실 감각을 유지하기 위한 미래 정보 인프라의 필수적인 부분임을 시사합니다. 주요 기술 기업들의 C2PA 채택과 ISO 표준화 움직임은 수동적 탐지만으로는 불충분하며 새로운 선제적 표준이 필요하다는 광범위한 공감대가 형성되고 있음을 보여줍니다.93


- **효율적인 샘플링 및 훈련:** 가장 중요한 연구 방향은 계속해서 효율성 향상이 될 것입니다. 여기에는 더 빠른 샘플링 알고리즘 개발, 더 나은 증류 및 양자화 기법, 그리고 Mamba나 하이브리드 모델과 같은 더 효율적인 아키텍처 개발이 포함됩니다.97
- **향상된 제어 가능성:** 연구는 사용자에게 생성 과정에 대한 더 세밀한 제어권을 부여하는 데 초점을 맞추고 있습니다. 이는 단순한 텍스트 프롬프트를 넘어 정확한 레이아웃 제어(LayoutDiffusion), 대상의 정체성 보존, 스타일 가이던스 등을 포함합니다.100
- **다중모드 통합:** DiT가 다양한 데이터 유형에서 성공을 거둔 것은 통합된 다중모드 기반 모델의 미래를 암시합니다. 연구는 텍스트, 이미지, 비디오, 오디오, 3D 데이터를 원활하게 처리하고 생성할 수 있는 단일의 유연한 아키텍처를 구축하는 데 집중될 가능성이 높으며, 이는 가중치와 표현을 공유하는 형태가 될 수 있습니다.13
- **하드웨어-소프트웨어 공동 설계:** DiT와 같은 모델이 기반 기술이 됨에 따라, 특화된 하드웨어 가속기에 대한 필요성이 증가하고 있습니다. 하드웨어-소프트웨어 공동 설계는 신경망 아키텍처와 기본 하드웨어(예: FPGA, ASIC)를 공동으로 최적화하여 성능과 효율성을 극대화하는 것을 목표로 합니다. 예를 들어, 자기 주의 메커니즘에 특화된 가속기를 설계하는 것이 여기에 해당합니다.106

트랜스포머 아키텍처가 지배적인 위치를 차지하면서, 자기 주의라는 명확하고 계산 집약적인 핵심 연산이 하드웨어 가속을 위한 안정적인 목표를 제공하게 되었습니다. 이로 인해 DiT와 같은 모델의 부상은 소프트웨어 혁신뿐만 아니라, 주의 기반 생성 워크로드에 최적화된 새로운 세대의 AI 칩 개발을 촉진하는 하드웨어 혁신의 촉매제 역할을 할 것으로 보입니다.


확산 트랜스포머(DiT)는 확산 모델의 강력한 생성 능력과 트랜스포머의 탁월한 확장성을 결합하여 생성형 AI 분야에 근본적인 패러다임 전환을 가져왔습니다. 본 보고서는 DiT의 탄생 배경부터 아키텍처의 진화, 성능, 응용 분야, 그리고 사회적 영향에 이르기까지 다각적인 분석을 통해 이 기술의 현재와 미래를 조망했습니다.

DiT의 핵심은 U-Net의 지역적 한계와 정적 귀납적 편향을 극복하고, 트랜스포머의 전역 컨텍스트 모델링 능력과 예측 가능한 확장성을 생성 모델에 도입한 것입니다. 특히, 잠재 공간에서의 패치 기반 처리와 adaLN-Zero와 같은 안정적인 컨디셔닝 메커니즘은 DiT가 ImageNet과 같은 주요 벤치마크에서 기존의 GAN 및 U-Net 기반 모델을 능가하는 SOTA 성능을 달성하는 기반이 되었습니다.

그러나 이러한 성공은 막대한 계산 비용이라는 대가를 수반했으며, 이는 훈련 및 추론 효율성을 높이기 위한 다양한 연구를 촉발했습니다. PixArt-α의 분해적 훈련 전략, PTQ4DiT와 같은 훈련 후 최적화, 그리고 DyDiT와 같은 동적 아키텍처는 이러한 비용 문제를 해결하기 위한 중요한 진전입니다. 더 나아가, U-DiT와 Skip-DiT에서 나타난 U-Net 원리의 재통합과 DiM에서 보이는 Mamba와 같은 새로운 백본의 등장은, 순수한 확장성을 넘어 안정성, 효율성, 그리고 아키텍처적 유연성을 추구하는 방향으로 연구가 진화하고 있음을 보여줍니다.

DiT의 패러다임은 정적 이미지를 넘어 비디오(Sora), 3D 모델(DiT-3D), 로봇 정책, 심지어 분자 구조(ADiT) 생성에까지 성공적으로 확장되었습니다. 이는 '패치화'라는 개념이 다양한 데이터 양식을 아우르는 보편적인 추상화임을 입증하며, DiT가 범용 생성 모델의 기반이 될 수 있는 잠재력을 시사합니다. 또한, DiT 백본이 다운스트림 인식 작업에서 강력한 성능을 보이는 것은, 생성적 사전 훈련이 컴퓨터 비전의 새로운 표준이 될 수 있다는 가능성을 제기합니다.

이러한 기술적 진보와 함께, 딥페이크, 데이터 편향, 저작권 문제와 같은 심각한 윤리적 과제가 대두되었습니다. 이에 대응하여 C2PA와 같은 디지털 출처 증명 표준과 기술적 탐지 방법론이 개발되고 있으며, 이는 책임감 있는 AI 생태계 구축에 필수적인 요소가 될 것입니다.

결론적으로, 확산 트랜스포머는 단순한 아키텍처의 조합을 넘어, 확장성, 일반성, 그리고 효율성이라는 현대 AI의 핵심 가치를 구현하는 강력한 프레임워크입니다. 앞으로의 연구는 계산 효율성을 극대화하고, 제어 가능성을 정교화하며, 다양한 양식을 통합하는 방향으로 나아갈 것입니다. 하드웨어-소프트웨어 공동 설계와 책임감 있는 배포를 위한 사회적, 기술적 노력이 병행될 때, DiT는 과학, 예술, 산업 전반에 걸쳐 혁신을 주도하는 핵심 동력으로 자리매김할 것입니다.


1. Denoising Diffusion-based Generative Modeling: Foundations and Applications, accessed July 18, 2025, https://cvpr2022-tutorial-diffusion-models.github.io/
2. Unit 1: An Introduction to Diffusion Models - Hugging Face Diffusion Course, accessed July 18, 2025, https://huggingface.co/learn/diffusion-course/unit1/1
3. Diffusion Transformer: Architecture behind Sora State-of-the-Art Video Generation - Medium, accessed July 18, 2025, https://medium.com/@humzanaveed/diffusion-transformer-architecture-behind-sora-state-of-the-art-video-generation-cb9f2eee69ec
4. Tutorial 2: Diffusion models - Neuromatch Academy: Deep Learning, accessed July 18, 2025, https://deeplearning.neuromatch.io/tutorials/W2D4_GenerativeModels/student/W2D4_Tutorial2.html
5. Scalable Diffusion Models with Transformers - CVF Open Access, accessed July 18, 2025, https://openaccess.thecvf.com/content/ICCV2023/papers/Peebles_Scalable_Diffusion_Models_with_Transformers_ICCV_2023_paper.pdf
6. Aman's AI Journal • Primers • Diffusion Models, accessed July 18, 2025, https://aman.ai/primers/ai/diffusion-models/
7. ArXiv Dives - Diffusion Transformers - Oxen.ai, accessed July 18, 2025, https://www.oxen.ai/blog/arxiv-dives-diffusion-transformers
8. Some Details on OpenAI's Sora and Diffusion Transformers | ml-news - Wandb, accessed July 18, 2025, https://wandb.ai/byyoung3/ml-news/reports/Some-Details-on-OpenAI-s-Sora-and-Diffusion-Transformers--Vmlldzo2ODQwNTM3
9. A Journey from U-Net to Diffusion Models:Part 1of Generative AI with Diffusion Models | by yasmine karray | Medium, accessed July 18, 2025, https://medium.com/@ykarray29/a-journey-from-u-net-to-diffusion-models-part-1-of-generative-ai-with-diffusion-models-b228b4306b86
10. What is a Transformer Model? | IBM, accessed July 18, 2025, https://www.ibm.com/think/topics/transformer-model
11. Transformer (deep learning architecture) - Wikipedia, accessed July 18, 2025, https://en.wikipedia.org/wiki/Transformer_(deep_learning_architecture)
12. How Transformers Work: A Detailed Exploration of Transformer Architecture - DataCamp, accessed July 18, 2025, https://www.datacamp.com/tutorial/how-transformers-work
13. (PDF) The Rise of Diffusion Transformers: Revolutionizing AI Image and Video Generation, accessed July 18, 2025, https://www.researchgate.net/publication/393413268_The_Rise_of_Diffusion_Transformers_Revolutionizing_AI_Image_and_Video_Generation
14. LLM Transformer Model Visually Explained - Polo Club of Data Science, accessed July 18, 2025, https://poloclub.github.io/transformer-explainer/
15. Step-by-Step Illustrated Explanations of Transformer | by Yule Wang, PhD | The Modern Scientist | Medium, accessed July 18, 2025, https://medium.com/the-modern-scientist/detailed-explanations-of-transformer-step-by-step-dc32d90b3a98
16. Transformers Explained | Simple Explanation of Transformers - YouTube, accessed July 18, 2025, https://www.youtube.com/watch?v=ZhAz268Hdpw&pp=0gcJCfwAo7VqN5tD
17. Diffusion Transformer (DiT) Models: A Beginner's Guide - Encord, accessed July 18, 2025, https://encord.com/blog/diffusion-models-with-transformers/
18. (PDF) Scalable Diffusion Models with Transformers - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/366423225_Scalable_Diffusion_Models_with_Transformers
19. U-Nets Power Diffusion Models? Here's How! | by Chitrangada Devulapalli | Medium, accessed July 18, 2025, https://medium.com/@chitrakd/u-nets-power-diffusion-models-heres-how-c7a27c3750b0
20. Enhancing Image Generation with Diffusion Transformer Architecture - SciTePress, accessed July 18, 2025, https://www.scitepress.org/Papers/2024/129378/129378.pdf
21. Stable Diffusion 3: Multimodal Diffusion Transformer Model Explained - Encord, accessed July 18, 2025, https://encord.com/blog/stable-diffusion-3-text-to-image-model/
22. Scaleble Diffusion Models with Transformers | by Liuzhihui - Medium, accessed July 18, 2025, https://medium.com/@liuzhihui2046/scaleble-diffusion-models-with-transformers-527fbfa8eab7
23. UNVEILING THE SECRET OF ADALN-ZERO IN ... - OpenReview, accessed July 18, 2025, https://openreview.net/pdf?id=E4roJSM9RM
24. Unveiling the Secret of AdaLN-Zero in Diffusion Transformer | OpenReview, accessed July 18, 2025, https://openreview.net/forum?id=E4roJSM9RM
25. [2505.15270] Scaling Diffusion Transformers Efficiently via $μ$P - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2505.15270
26. ImageNet 256x256 Benchmark (Image Generation) | Papers With Code, accessed July 18, 2025, https://paperswithcode.com/sota/image-generation-on-imagenet-256x256
27. image generation - MMagic documentation - Read the Docs, accessed July 18, 2025, https://mmagic.readthedocs.io/en/stable/model_zoo/image_generation.html
28. Diffusion Transformers : Replacing The UNet With A Transformers Based Model Achives State Of The Art Results : r/StableDiffusion - Reddit, accessed July 18, 2025, https://www.reddit.com/r/StableDiffusion/comments/zssiye/diffusion_transformers_replacing_the_unet_with_a/
29. DiffFit: Unlocking Transferability of Large ... - CVF Open Access, accessed July 18, 2025, https://openaccess.thecvf.com/content/ICCV2023/papers/Xie_DiffFit_Unlocking_Transferability_of_Large_Diffusion_Models_via_Simple_Parameter-efficient_ICCV_2023_paper.pdf
30. I read that training Stable DIffusion 1.5 only cost 500 thousand dollars. Very cheap ! The price of a house. Strange that there are so few models : r/StableDiffusion - Reddit, accessed July 18, 2025, https://www.reddit.com/r/StableDiffusion/comments/196pua0/i_read_that_training_stable_diffusion_15_only/
31. Seaweed-7B: Cost-Effective Training of Video Generation Foundation Model - arXiv, accessed July 18, 2025, https://arxiv.org/html/2504.08685
32. Fast Training of Diffusion Models with Masked Transformers - arXiv, accessed July 18, 2025, https://arxiv.org/html/2306.09305v2
33. DiTFastAttn: Attention Compression for Diffusion Transformer Models - NIPS, accessed July 18, 2025, https://proceedings.neurips.cc/paper_files/paper/2024/file/0267925e3c276e79189251585b4100bf-Paper-Conference.pdf
34. EDiT: Efficient Diffusion Transformers with Linear Compressed Attention - arXiv, accessed July 18, 2025, https://arxiv.org/html/2503.16726v1
35. PTQ4DiT: Post-training Quantization for Diffusion Transformers - NIPS, accessed July 18, 2025, https://proceedings.neurips.cc/paper_files/paper/2024/file/72d32f4fe0b7af03732bd227bf1c4a5f-Paper-Conference.pdf
36. VQ4DiT: Efficient Post-Training Vector Quantization for Diffusion Transformers, accessed July 18, 2025, https://ojs.aaai.org/index.php/AAAI/article/view/33782/35937
37. arXiv:2310.00426v2 [cs.CV] 16 Oct 2023 - SciSpace, accessed July 18, 2025, https://scispace.com/pdf/pixart-a-fast-training-of-diffusion-transformer-for-2n8isde8hj.pdf
38. PixArt-α: Fast Training of Diffusion Transformer for Photorealistic Text-to-Image Synthesis - GitHub, accessed July 18, 2025, https://github.com/PixArt-alpha/PixArt-alpha
39. PixArt-$\alpha$: Fast Training of Diffusion Transformer for Photorealistic Text-to-Image Synthesis | OpenReview, accessed July 18, 2025, https://openreview.net/forum?id=eAKmQPe3m1
40. PixArt-α: Fast Training of Diffusion Transformer for Photorealistic Text-to-Image Synthesis, accessed July 18, 2025, https://huggingface.co/papers/2310.00426
41. Post-Training Quantization on Diffusion Models | Request PDF - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/373325453_Post-Training_Quantization_on_Diffusion_Models
42. Dynamic Diffusion Transformer - OpenReview, accessed July 18, 2025, https://openreview.net/forum?id=taHwqSrbrb
43. DYNAMIC DIFFUSION TRANSFORMER - ICLR Proceedings, accessed July 18, 2025, https://proceedings.iclr.cc/paper_files/paper/2025/file/a44a70acd5d0abc1a252ada9719dd06d-Paper-Conference.pdf
44. Dynamic Diffusion Transformer - arXiv, accessed July 18, 2025, https://arxiv.org/html/2410.03456v2
45. [2407.11633] Scaling Diffusion Transformers to 16 Billion Parameters - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2407.11633
46. [2410.13925] FiTv2: Scalable and Improved Flexible Vision Transformer for Diffusion Model - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2410.13925
47. Towards Stabilized and Efficient Diffusion Transformers through Long-Skip-Connections with Spectral Constraints - arXiv, accessed July 18, 2025, https://arxiv.org/html/2411.17616v4
48. Development of Skip Connection in Deep Neural Networks for Computer Vision and Medical Image Analysis: A Survey - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/383609186_Development_of_Skip_Connection_in_Deep_Neural_Networks_for_Computer_Vision_and_Medical_Image_Analysis_A_Survey
49. ScaleLong: Towards More Stable Training of Diffusion Model via Scaling Network Long Skip Connection | OpenReview, accessed July 18, 2025, [https://openreview.net/forum?id=0N73P8pH2l¬eId=QtvN1T6HTN](https://openreview.net/forum?id=0N73P8pH2l&noteId=QtvN1T6HTN)
50. U-DiTs: Downsample Tokens in U-Shaped Diffusion Transformers - arXiv, accessed July 18, 2025, https://arxiv.org/html/2405.02730v3
51. U-DiTs: Downsample Tokens in U-Shaped Diffusion Transformers ..., accessed July 18, 2025, [https://openreview.net/forum?id=SRWs2wxNs7&referrer=%5Bthe%20profile%20of%20Hanting%20Chen%5D(%2Fprofile%3Fid%3D~Hanting_Chen1)](https://openreview.net/forum?id=SRWs2wxNs7&referrer=[the+profile+of+Hanting+Chen](/profile?id%3D~Hanting_Chen1))
52. [Literature Review] U-DiTs: Downsample Tokens in U-Shaped Diffusion Transformers, accessed July 18, 2025, https://www.themoonlight.io/en/review/u-dits-downsample-tokens-in-u-shaped-diffusion-transformers
53. [2411.17616] Towards Stabilized and Efficient Diffusion Transformers through Long-Skip-Connections with Spectral Constraints - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2411.17616
54. [2405.14224] DiM: Diffusion Mamba for Efficient High-Resolution Image Synthesis - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2405.14224
55. DiM: Diffusion Mamba for Efficient High-Resolution Image Synthesis, accessed July 18, 2025, https://dai.sjtu.edu.cn/my_file/pdf/ed15c7e9-d8ec-44ea-990f-5eac64e86c1b.pdf
56. Mamba-ST: State Space Model for Efficient Style Transfer - arXiv, accessed July 18, 2025, https://arxiv.org/html/2409.10385v1
57. Scalable Autoregressive Image Generation with Mamba - arXiv, accessed July 18, 2025, https://arxiv.org/html/2408.12245v4
58. state-spaces/mamba: Mamba SSM architecture - GitHub, accessed July 18, 2025, https://github.com/state-spaces/mamba
59. DiM: Diffusion Mamba for Efficient High-Resolution Image Synthesis - arXiv, accessed July 18, 2025, https://arxiv.org/html/2405.14224v1
60. DiM: Diffusion Mamba for Efficient High-Resolution Image Synthesis | AI Research Paper Details - AIModels.fyi, accessed July 18, 2025, https://www.aimodels.fyi/papers/arxiv/dim-diffusion-mamba-efficient-high-resolution-image
61. [2411.04168] DiMSUM: Diffusion Mamba -- A Scalable and Unified Spatial-Frequency Method for Image Generation - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2411.04168
62. DiMSUM: Diffusion Mamba - A Scalable and Unified Spatial-Frequency Method for Image Generation | OpenReview, accessed July 18, 2025, [https://openreview.net/forum?id=KqbLzSIXkm&referrer=%5Bthe%20profile%20of%20Hoang%20Phan%5D(%2Fprofile%3Fid%3D~Hoang_Phan1)](https://openreview.net/forum?id=KqbLzSIXkm&referrer=[the+profile+of+Hoang+Phan](/profile?id%3D~Hoang_Phan1))
63. DiMSUM : Diffusion Mamba - A Scalable and Unified Spatial-Frequency Method for Image Generation, accessed July 18, 2025, https://neurips.cc/media/neurips-2024/Slides/95642.pdf
64. Sora: A Review on Background, Technology, Limitations, and Opportunities of Large Vision Models - arXiv, accessed July 18, 2025, https://arxiv.org/html/2402.17177v2
65. [D] The Tech Behind The Magic : How OpenAI SORA Works : r/MachineLearning - Reddit, accessed July 18, 2025, https://www.reddit.com/r/MachineLearning/comments/1bqmn86/d_the_tech_behind_the_magic_how_openai_sora_works/
66. Sora System Card - OpenAI, accessed July 18, 2025, https://openai.com/index/sora-system-card/
67. OpenAI Sora's Technical Review. Exploring Sora: A Deep Dive into ..., accessed July 18, 2025, https://j-qi.medium.com/openai-soras-technical-review-a8f85b44cb7f
68. OpenAI Unveils Sora: The Future of Text-to-Video Generation - DEV Community, accessed July 18, 2025, https://dev.to/usulpro/openai-unveils-sora-the-future-of-text-to-video-generation-2om7
69. OpenAI's Next Leap, Sora: Technical Report Notes for Techies In a Hurry | by Manan Aggarwal | Medium, accessed July 18, 2025, https://medium.com/@mananaggarwal457/sora-the-next-chatgpt-ai-llm-model-is-out-98892a89afc2
70. Explaining OpenAI Sora's Spacetime Patches: The Key Ingredient | Towards Data Science, accessed July 18, 2025, https://towardsdatascience.com/explaining-openai-soras-spacetime-patches-the-key-ingredient-e14e0703ec5b/
71. Scaling Diffusion Transformer for Generalist Vision-Language-Action Learning - arXiv, accessed July 18, 2025, https://arxiv.org/html/2410.15959v3
72. DiT-3D: Exploring Plain Diffusion Transformers for 3D Shape Generation, accessed July 18, 2025, https://proceedings.neurips.cc/paper_files/paper/2023/file/d6c01b025cad37d5c8bab4ba18846c02-Paper-Conference.pdf
73. DiffSurf: A Transformer-based Diffusion Model for Generating and Reconstructing 3D Surfaces in Pose, accessed July 18, 2025, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/10778.pdf
74. All-atom Diffusion Transformers: Unified generative modelling of molecules and materials, accessed July 18, 2025, https://www.researchgate.net/publication/389648327_All-atom_Diffusion_Transformers_Unified_generative_modelling_of_molecules_and_materials
75. All-atom Diffusion Transformers: Unified generative modelling of molecules and materials - arXiv, accessed July 18, 2025, https://arxiv.org/html/2503.03965v1
76. All-atom Diffusion Transformers: Unified generative ... - arXiv, accessed July 18, 2025, https://arxiv.org/pdf/2503.03965
77. All-atom Diffusion Transformers: Unified generative modelling of molecules and materials | OpenReview, accessed July 18, 2025, [https://openreview.net/forum?id=89QPmZjIhv&referrer=%5Bthe%20profile%20of%20Xiang%20Fu%5D(%2Fprofile%3Fid%3D~Xiang_Fu4)](https://openreview.net/forum?id=89QPmZjIhv&referrer=[the+profile+of+Xiang+Fu](/profile?id%3D~Xiang_Fu4))
78. Diffusion Models Trained with Large Data Are Transferable Visual Models - arXiv, accessed July 18, 2025, https://arxiv.org/html/2403.06090v1
79. Three Things We Need to Know About Transferring Stable Diffusion to Visual Dense Prediction Tasks, accessed July 18, 2025, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/05837.pdf
80. DatasetDM: Synthesizing Data with Perception Annotations Using Diffusion Models, accessed July 18, 2025, https://proceedings.neurips.cc/paper_files/paper/2023/file/ab6e7ad2354f350b451b5a8e14d04f51-Paper-Conference.pdf
81. [2403.06090] What Matters When Repurposing Diffusion Models for General Dense Perception Tasks? - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2403.06090
82. What Matters When Repurposing Diffusion Models for General Dense Perception Tasks?, accessed July 18, 2025, https://openreview.net/forum?id=BgYbk6ZmeX
83. [2403.06090v1] Diffusion Models Trained with Large Data Are Transferable Visual Models, accessed July 18, 2025, https://arxiv.org/abs/2403.06090v1/
84. What ethical considerations are involved in deploying diffusion models? - Milvus, accessed July 18, 2025, https://milvus.io/ai-quick-reference/what-ethical-considerations-are-involved-in-deploying-diffusion-models
85. Ethical Implications of Stable Diffusion - GoML, accessed July 18, 2025, https://www.goml.io/blog/ethical-implications-of-stable-diffusion
86. The Cat and Mouse Game: The Ongoing Arms Race Between Diffusion Models and Detection Methods - arXiv, accessed July 18, 2025, https://arxiv.org/html/2410.18866v1
87. Deepfake Detection: Neural Networks Are Fighting Themselves | HackerNoon, accessed July 18, 2025, https://hackernoon.com/deepfake-detection-neural-networks-are-fighting-themselves
88. Deepfake Detection – Protecting Identity Systems from AI-Generated Fraud - Deepak Gupta, accessed July 18, 2025, https://guptadeepak.com/deepfake-detection-protecting-identity-systems-from-ai-generated-fraud/
89. 9 Techniques To Spot AI-Generated Videos - CanIPhish, accessed July 18, 2025, https://caniphish.com/blog/how-to-spot-ai-videos
90. How to Detect Deepfakes: Identity Verification Against Fraud with AI - Facephi, accessed July 18, 2025, https://facephi.com/en/deepfakes-identity-verification-fraud-ai/
91. C2PA | Verifying Media Content Sources, accessed July 18, 2025, https://c2pa.org/
92. How C2PA can safeguard the truth from digital manipulation - SC Media, accessed July 18, 2025, https://www.scworld.com/perspective/how-c2pa-can-safeguard-the-truth-from-digital-manipulation
93. Content Credentials: Strengthening Multimedia Integrity in the Generative AI Era - Department of Defense, accessed July 18, 2025, https://media.defense.gov/2025/Jan/29/2003634788/-1/-1/0/CSI-CONTENT-CREDENTIALS.PDF
94. C2PA Explainer :: C2PA Specifications, accessed July 18, 2025, https://c2pa.org/specifications/specifications/1.3/explainer/Explainer.html
95. Guidance for Artificial Intelligence and Machine Learning :: C2PA Specifications, accessed July 18, 2025, https://c2pa.org/specifications/specifications/2.2/ai-ml/ai_ml.html
96. C2PA NIST Response - Regulations.gov, accessed July 18, 2025, https://downloads.regulations.gov/NIST-2024-0001-0030/attachment_1.pdf
97. A Contextualized Survey on the State and Future Directions of Diffusion Models - OSF, accessed July 18, 2025, https://osf.io/xcfdu/download
98. Stanford CS25: V5 I Transformers in Diffusion Models for Image Generation and Beyond, accessed July 18, 2025, https://www.youtube.com/watch?v=vXtapCFctTI
99. osf.io, accessed July 18, 2025, [https://osf.io/xcfdu/download#:~:text=The%20main%20research%20directions%20include,to%20a%20number%20of%20applications.](https://osf.io/xcfdu/download#:~:text=The main research directions include,to a number of applications.)
100. Towards Aligned Layout Generation via Diffusion Model with Aesthetic Constraints | OpenReview, accessed July 18, 2025, https://openreview.net/forum?id=kJ0qp9Xdsh
101. Create Anything Anywhere: Layout-Controllable Personalized Diffusion Model for Multiple Subjects *Corresponding authors - arXiv, accessed July 18, 2025, https://arxiv.org/html/2505.20909v1
102. LayoutDM: Discrete Diffusion Model for Controllable Layout Generation - Simo-Serra Lab., accessed July 18, 2025, https://esslab.jp/~ess/publications/InouCVPR2023a.pdf
103. LayoutDiffusion: Controllable Diffusion Model for Layout-to-Image Generation - CVF Open Access, accessed July 18, 2025, https://openaccess.thecvf.com/content/CVPR2023/papers/Zheng_LayoutDiffusion_Controllable_Diffusion_Model_for_Layout-to-Image_Generation_CVPR_2023_paper.pdf
104. Diffusion Transformer on Unified Video, 3D, and Game Field Generation - OpenReview, accessed July 18, 2025, https://openreview.net/forum?id=w6YS9A78fq
105. The Future of Transformers: Emerging Trends and Research Directions | by Hassaan Idrees, accessed July 18, 2025, https://medium.com/@hassaanidrees7/the-future-of-transformers-emerging-trends-and-research-directions-d3eddce993f6
106. Accelerating Framework of Transformer by Hardware Design and Model Compression Co-Optimization | Request PDF - ResearchGate, accessed July 18, 2025, https://www.researchgate.net/publication/357288008_Accelerating_Framework_of_Transformer_by_Hardware_Design_and_Model_Compression_Co-Optimization
107. Robotic computing system and embodied AI evolution: an algorithm-hardware co-design perspective - Journal of Semiconductors, accessed July 18, 2025, https://www.jos.ac.cn/en/article/doi/10.1088/1674-4926/25020034
108. [2502.12344] Hardware-Software Co-Design for Accelerating Transformer Inference Leveraging Compute-in-Memory - arXiv, accessed July 18, 2025, https://arxiv.org/abs/2502.12344
109. Hardware Software Co-design and Architectural Optimization of ..., accessed July 18, 2025, https://www2.eecs.berkeley.edu/Pubs/TechRpts/2023/EECS-2023-92.html
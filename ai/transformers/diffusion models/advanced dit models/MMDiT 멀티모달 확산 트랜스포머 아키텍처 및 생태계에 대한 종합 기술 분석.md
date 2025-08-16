# 멀티모달 확산 트랜스포머(MMDiT) 아키텍처 및 생태계에 대한 종합 기술 분석

## 1.  아키텍처의 통합: 합성곱 편향에서 트랜스포머 확장성으로

생성 모델의 발전사는 특정 영역에 최적화된 아키텍처와 범용적 확장성을 지닌 아키텍처 간의 철학적 긴장을 반영해왔다. 확산 모델(Diffusion Model)의 초기 성공은 U-Net이라는 강력한 합성곱 신경망(Convolutional Neural Network, CNN) 기반의 백본에 의해 주도되었다. 그러나 머신러닝 분야 전반에 걸쳐 트랜스포머(Transformer) 아키텍처가 패러다임을 재정의하면서, 생성 모델 분야에서도 근본적인 변화가 시작되었다. 확산 트랜스포머(Diffusion Transformer, DiT)의 등장은 이러한 변화의 정점을 상징하며, 이는 단순히 새로운 구성 요소를 도입한 것을 넘어, 모델 설계의 기본 원칙을 도메인 특화적인 귀납적 편향(Inductive Bias)에서 범용 아키텍처의 계산적 확장성(Scalability)으로 전환하는 중대한 철학적 이동을 의미한다. 본 장에서는 U-Net의 시대와 그 기반이 된 귀납적 편향을 고찰하고, 트랜스포머가 비전 분야에서 어떻게 부상했는지 분석하며, DiT가 왜 기존의 성공 공식을 대체하는 패러다임 전환을 이끌었는지 심층적으로 탐구한다.

### 1.1  U-Net의 시대: 귀납적 편향 위에 세워진 토대

초기 고성능 확산 모델의 성공은 U-Net 아키텍처 없이는 논하기 어렵다. PixelCNN++와 같은 선행 모델로부터 계승된 U-Net은 확산 모델의 표준 백본으로 빠르게 자리 잡았다.1 U-Net의 효과는 본질적으로 CNN이 가진 강력한 귀납적 편향에 기인한다. 귀납적 편향이란 머신러닝 알고리즘이 한정된 데이터로부터 보지 못한 데이터에 대해 일반화하기 위해 사용하는 일련의 가정, 제약, 또는 사전 지식을 의미한다.2 CNN 기반의 U-Net은 이미지 데이터의 본질적인 특성을 반영하는 몇 가지 핵심적인 귀납적 편향을 내재하고 있다.

첫째, **지역성(Locality)**이다. 이는 이미지에서 서로 인접한 픽셀들이 강한 상관관계를 가진다는 가정이다.5 CNN의 합성곱 필터는 이미지의 작은 국소 영역에 적용되어 이 지역성을 효과적으로 활용하며, 이를 통해 엣지, 질감, 모서리와 같은 저수준 특징을 효율적으로 학습한다.

둘째, **병진 등변성(Translation Equivariance)**이다. 이는 이미지 내에서 객체의 위치가 변하더라도 동일한 특징을 인식할 수 있는 능력이다.5 합성곱 연산은 필터를 이미지 전체에 걸쳐 슬라이딩하며 적용되므로, 이 특성이 자연스럽게 구현된다.

셋째, **계층적 처리(Hierarchical Processing)**이다. CNN은 연속적인 레이어를 통해 저수준의 단순한 특징(선, 모서리 등)을 조합하여 점차 고수준의 복잡한 특징(객체의 일부, 전체 객체 등)을 구축한다.5 U-Net의 인코더-디코더 구조와 스킵 연결(skip connection)은 이러한 계층적 특징 학습을 극대화하여 다양한 스케일의 정보를 효과적으로 통합한다.

이러한 귀납적 편향들은 CNN과 U-Net을 이미지 관련 과제에 본질적으로 매우 적합하게 만들었다. 그러나 이 강력한 가정들은 동시에 한계로 작용할 수도 있다. U-Net은 이미지 내에서 멀리 떨어진 픽셀들 간의 장거리 의존성(long-range dependency)이나 전역적인 맥락을 모델링하는 데 있어서는 다른 아키텍처만큼 효과적이지 않을 수 있다. 이 지점에서 트랜스포머의 가능성이 대두되었다.

### 1.2  트랜스포머의 부상: 범용 아키텍처

트랜스포머는 본래 자연어 처리(Natural Language Processing, NLP) 분야에서 순차 데이터 내의 장거리 의존성을 모델링하는 데 탁월한 성능을 보이며 성공을 거두었다.5 비전 트랜스포머(Vision Transformer, ViT)는 이러한 트랜스포머 아키텍처를 컴퓨터 비전 분야에 성공적으로 적용한 중추적인 모델이다.5 ViT의 핵심 아이디어는 이미지를 일련의 패치(patch) 시퀀스로 취급하고, 이를 셀프 어텐션(self-attention) 메커니즘으로 처리하는 것이다. 이는 시각적 문제를 본질적으로 시퀀스 모델링 문제로 재정의한 것이다.10

이 접근 방식은 중요한 장단점을 수반한다. ViT는 CNN이 가진 이미지 특화적인 강력한 귀납적 편향(지역성, 병진 등변성 등)이 부족하다. 이로 인해 상대적으로 적은 양의 데이터로 학습할 경우 CNN보다 일반화 성능이 떨어지는 경향이 있으며, 효과적인 학습을 위해 방대한 양의 데이터가 필요하다.5 하지만 ViT는 이러한 단점을 이미지 전체에 걸쳐 전역적인 관계를 포착하는 뛰어난 능력으로 보완한다.11 셀프 어텐션 메커니즘은 모든 패치 쌍 간의 상호작용을 계산함으로써, 이미지의 한쪽 끝에 있는 픽셀이 다른 쪽 끝에 있는 픽셀에 미치는 영향을 직접적으로 모델링할 수 있다.

### 1.3  패러다임 전환: 성공 공식을 대체한 이유

이러한 배경 속에서, 확산 트랜스포머(DiT)를 제안한 연구는 생성 모델 분야에 근본적인 질문을 던졌다. 그들의 핵심 주장은 "U-Net의 귀납적 편향이 확산 모델의 성능에 결정적이지 않다"는 것이었다.9 이는 당시 지배적이었던 통념에 대한 정면 도전이었다. 이 연구는 DiT를 다양한 영역에서 특화된 모델들을 트랜스포머가 대체하고 있는 '아키텍처 통합(architecture unification)'이라는 거대한 흐름의 일부로 위치시켰다.1

DiT의 등장은 단순한 아키텍처의 '업그레이드'가 아니라, 전략적인 트레이드오프의 결과로 해석될 수 있다. 이는 CNN에 내재된 이미지 구조에 대한 가정, 즉 '지식 기반 귀납적 편향(knowledge-based inductive bias)'을 포기하는 대신, 트랜스포머가 계산 자원(compute) 증가에 따라 성능이 예측 가능하게 향상되는 '계산 기반 귀납적 편향(compute-based inductive bias)'을 채택한 것이다. CNN/U-Net은 내장된 가정 덕분에 특히 소규모 데이터셋에서 유리한 출발을 할 수 있다.5 이는 일종의 '인코딩된 지식'이다. 반면, 트랜스포머 기반의 DiT는 이러한 특정 지식을 버리고 이미지를 일반적인 패치 시퀀스로 취급한다.9 하지만 DiT 연구자들은 트랜스포머가 U-Net보다 계산량(Gflops) 증가에 따라 훨씬 더 예측 가능하게 성능이 확장된다는 점을 명확히 입증했다.9

결론적으로, DiT가 건넨 근본적인 베팅은 거대한 스케일에서는 범용 아키텍처(트랜스포머)의 원초적인 확장성이 특화된 아키텍처(U-Net)의 초기 이점을 능가할 것이라는 점이다. 모델과 데이터셋이 충분히 크다면, 모델은 데이터로부터 지역적 관계를 스스로 학습할 수 있으며, 나아가 전역적 관계를 더 효과적으로 모델링함으로써 U-Net의 성능을 뛰어넘을 수 있다. 이러한 철학은 구조화된 NLP 모델을 압도한 대규모 언어 모델(LLM)의 성공 경로와 정확히 일치하며, 생성 모델 분야가 새로운 시대로 진입했음을 알리는 신호탄이었다.

## 2.  기본 원리: 확산 트랜스포머(DiT)의 해부

확산 트랜스포머(DiT)는 생성 모델 아키텍처의 혁신을 대표하며, 그 설계 원칙과 실험적 검증은 후속 연구의 기틀을 마련했다. DiT의 핵심은 U-Net의 복잡한 합성곱 구조를 버리고, 트랜스포머의 순수하고 확장 가능한 아키텍처를 채택한 데 있다. 본 장에서는 DiT의 핵심 아키텍처, 조건부 정보를 주입하는 정교한 메커니즘, 그리고 이 모델의 혁명적인 확장 법칙(Scaling Law)을 확립한 경험적 증거들을 심층적으로 분석한다.

### 2.1  핵심 아키텍처: U-Net 백본의 대체

DiT는 픽셀 공간(pixel space)에서 직접 작동하는 대신, 잠재 확산 모델(Latent Diffusion Model, LDM) 프레임워크 내에서 운영된다.1 이는 DiT가 원본 이미지를 직접 처리하는 것이 아니라, VAE(Variational Autoencoder)를 통해 압축된 저차원 잠재 표현(latent representation)을 다룬다는 것을 의미한다. 이 접근 방식은 계산량을 크게 줄여 대규모 모델의 학습을 현실적으로 만든다.

DiT의 첫 번째 단계는 `패치화(patchify)` 레이어다.14 이 레이어는 입력으로 받은 잠재 공간 텐서를 겹치지 않는 일련의 작은 패치(토큰)로 분할한다. 예를 들어, $H×W×C$ 크기의 잠재 텐서는 $N$개의 패치 시퀀스로 변환되며, 각 패치의 크기는 $P×P$이다. 여기서 $N=HW/P^2$가 된다. 이 과정은 2차원 공간적 문제를 트랜스포머가 처리하기에 적합한 1차원 시퀀스 문제로 변환하는 결정적인 단계이다.

이렇게 생성된 패치 토큰 시퀀스는 DiT의 본체를 구성하는 일련의 표준 트랜스포머 블록에 의해 처리된다.9 각 트랜스포머 블록은 멀티헤드 셀프 어텐션(Multi-Head Self-Attention) 메커니즘과 MLP(Multi-Layer Perceptron) 블록으로 구성된다. 이는 U-Net의 복잡하고 계층적인 합성곱 구조와는 극명한 대조를 이룬다. DiT는 U-Net의 다운샘플링, 업샘플링, 스킵 연결과 같은 복잡한 구조 없이, 모든 토큰이 모든 다른 토큰과 상호작용할 수 있는 동질적인(isotropic) 아키텍처를 채택하여 전역적 맥락을 효과적으로 포착한다.

### 2.2  조건 주입: 제어 가능성의 핵심

확산 모델은 노이즈 제거 과정을 안내하기 위해 조건부 정보가 필요하다. DiT에서는 주로 노이즈 타임스텝 t와 클래스 레이블 c와 같은 조건이 사용된다. DiT 연구에서는 이러한 조건부 정보를 트랜스포머 아키텍처에 효과적으로 주입하기 위한 세 가지 메커니즘을 탐구하고 비교했다.13

1. **인컨텍스트 조건화(In-context conditioning):** 타임스텝 t와 클래스 레이블 c의 벡터 임베딩을 입력 패치 시퀀스에 추가적인 토큰으로 단순히 덧붙이는 방식이다. 이 방법은 구조가 간단하고 계산량(Gflops) 증가가 거의 없다는 장점이 있지만, 실험 결과 다른 방법에 비해 성능이 떨어지는 것으로 나타났다.
2. **크로스 어텐션 블록(Cross-attention block):** 각 트랜스포머 블록 내에 별도의 크로스 어텐션 레이어를 추가하여 조건부 벡터를 주입하는 방식이다. 이는 원래 트랜스포머 아키텍처의 디코더에서 사용된 방식과 유사하며, 이미지 토큰 시퀀스와 조건부 벡터 시퀀스 간의 상호작용을 모델링한다. 이 방식은 효과적이지만, 상당한 계산 오버헤드(약 15%의 Gflops 증가)를 유발한다.
3. **적응형 레이어 정규화(Adaptive Layer Norm, adaLN):** 최종적으로 가장 효과적인 것으로 입증된 전략이다. 이 방식은 트랜스포머 블록 내의 레이어 정규화(Layer Normalization)의 스케일($γ$) 및 시프트($β$) 파라미터를 직접 학습하는 대신, 타임스텝 t와 클래스 레이블 $c$의 임베딩 벡터 합으로부터 이 파라미터들을 회귀(regress)한다. 이 메커니즘은 계산적으로 매우 효율적이면서도 가장 뛰어난 성능을 보였다.

특히, 연구에서는 `adaLN-Zero`라는 중요한 구현 세부 사항을 도입했다.14 이 방법은 각 DiT 블록을 초기에는 항등 함수(identity function)처럼 작동하도록 초기화한다. 구체적으로, adaLN 레이어에서 회귀된 스케일 및 시프트 파라미터를 적용한 후, 추가적인 스케일 파라미터를 통해 블록의 출력이 입력과 동일하게 되도록 만든다. 이 작은 조정은 학습 초기의 불안정성을 해소하고 모델의 최종 성능(FID)을 크게 향상시키는 "엄청난 차이(huge difference)"를 만들었다고 보고되었다.14

### 2.3  확장 법칙의 혁명: 계산량이 왕이다

DiT 연구의 가장 핵심적인 경험적 발견은 모델의 성능, 즉 생성된 이미지의 품질을 나타내는 FID(Fréchet Inception Distance) 점수가 모델의 순방향 패스 복잡도, 즉 Gflops(초당 기가 부동소수점 연산)와 예측 가능한 관계를 가진다는 것이다.9 이는 단순히 모델의 파라미터 수를 늘리는 것보다, 실제 수행되는 계산량이 성능의 더 중요한 척도임을 시사한다.

연구에서는 Gflops를 증가시키는 두 가지 주요 방법을 탐구했다.

1. **모델 크기 확장:** 트랜스포머의 깊이(레이어 수)와 너비(은닉 차원 크기)를 증가시키는 것이다. 이는 파라미터 수를 직접적으로 증가시킨다. 연구에서는 ViT의 설정을 따라 Small(S), Base(B), Large(L) 모델을 정의하고, 가장 큰 모델로 XLarge(XL)를 추가로 제안했다.1
2. **토큰 수 확장:** 패치 크기(p)를 줄이는 것이다. 예를 들어, 패치 크기를 8에서 2로 줄이면 트랜스포머가 처리해야 할 입력 토큰의 수는 16배로 증가한다. 셀프 어텐션의 계산 복잡도는 시퀀스 길이에 대해 제곱($O(N^2$))이므로, 이는 Gflops를 폭발적으로 증가시킨다. 중요한 점은 이 방식이 모델의 총 파라미터 수에는 거의 영향을 미치지 않는다는 것이다.14

이러한 확장 법칙을 검증한 핵심 실험 결과는 다음과 같다.

- 모든 모델 구성에 걸쳐 Gflops와 FID 점수 사이에는 강력하고 일관된 음의 상관관계가 관찰되었다. 즉, Gflops가 증가할수록 FID 점수는 낮아져(성능 향상) 이미지 품질이 꾸준히 개선되었다.9
- 더 큰 DiT 모델은 계산적으로 더 효율적이었다. 즉, 특정 목표 FID 점수에 도달하기 위해 작은 모델보다 더 적은 총 학습 계산량(Total Training Compute)을 필요로 했다.9
- 최고 성능 모델인 DiT-XL/2(XL 모델 크기, 패치 크기 2)는 256x256 ImageNet 벤치마크에서 2.27이라는 당시 최고 수준의 FID 점수를 달성했다. 이는 LDM이나 ADM-U와 같은 이전의 모든 U-Net 기반 확산 모델을 능가하는 성과였으며, 심지어 ADM-U(742 Gflops)보다 훨씬 적은 계산량(119 Gflops)으로 이를 달성했다.9

이러한 발견을 구체적으로 보여주기 위해, 아래 **표 1**은 DiT 모델의 다양한 구성에 따른 파라미터 수, Gflops, 그리고 최종 FID 점수 간의 관계를 요약한다.

**표 1: DiT 확장 법칙 - ImageNet 256x256에서의 Gflops 대 FID**

| 모델 구성 | 파라미터 (백만) | 패치 크기 | 입력 토큰 수 | Gflops | FID-50K (가이던스 없음) |
| --------- | --------------- | --------- | ------------ | ------ | ----------------------- |
| DiT-S/8   | 33              | 8         | 1024         | 1.0    | 93.60                   |
| DiT-S/4   | 33              | 4         | 4096         | 3.2    | 77.26                   |
| DiT-S/2   | 33              | 2         | 16384        | 11.2   | 68.40                   |
| DiT-B/8   | 130             | 8         | 1024         | 3.5    | 77.34                   |
| DiT-B/4   | 130             | 4         | 4096         | 12.8   | 68.38                   |
| DiT-B/2   | 130             | 2         | 16384        | 49.9   | 49.33                   |
| DiT-L/8   | 458             | 8         | 1024         | 12.3   | 46.10                   |
| DiT-L/4   | 458             | 4         | 4096         | 47.7   | 35.88                   |
| DiT-L/2   | 458             | 2         | 16384        | 189.5  | 9.53                    |
| DiT-XL/8  | 675             | 8         | 1024         | 18.2   | 37.08                   |
| DiT-XL/4  | 675             | 4         | 4096         | 71.2   | 22.88                   |
| DiT-XL/2  | 675             | 2         | 16384        | 283.4  | **3.55**                |

참고: Gflops 및 FID 값은 논문 9의 데이터와 그림을 기반으로 재구성되었으며, 특정 실험 설정에 따라 약간의 차이가 있을 수 있다. FID 값은 분류기 없는 가이던스(classifier-free guidance)를 적용하지 않은 수치다.

이 표는 DiT 연구의 핵심 주장을 명확하게 뒷받침한다. 특히, 파라미터 수와 실제 계산량의 개념을 근본적으로 분리하는 중요한 함의를 담고 있다. 전통적으로 '더 큰' 모델은 더 많은 파라미터를 의미했다. 그러나 DiT의 패치 크기 실험은 이러한 통념을 깼다. 예를 들어, 표에서 DiT-B/4(130M 파라미터, 12.8 Gflops)와 DiT-L/8(458M 파라미터, 12.3 Gflops)을 비교해 보자. DiT-L/8은 파라미터 수가 3.5배 이상 많지만, Gflops는 DiT-B/4와 비슷하며 FID 성능 또한 크게 우월하지 않다. 반면, 동일한 DiT-B 모델에서 패치 크기를 4에서 2로 줄이면(DiT-B/2) 파라미터 수는 그대로지만 Gflops는 거의 4배로 증가하고, FID는 68.38에서 49.33으로 극적으로 향상된다.

이는 모델이 가진 파라미터의 *수*보다 그 파라미터를 얼마나 *집중적으로* 사용하는지(즉, 계산량)가 성능에 더 결정적인 영향을 미칠 수 있음을 보여준다. 이 발견은 생성 트랜스포머의 성능 확장을 이해하는 데 있어 새로운 차원을 열었으며, 하드웨어 설계와 모델 최적화에 있어 메모리 용량(파라미터 저장)과 처리 능력(Gflops 수행)이 별개의 확장 축임을 시사한다. 이 원칙은 이후 Sora와 같은 대규모 모델의 개발 철학에 직접적인 영향을 미쳤다.

## 3.  멀티모달 확장: MMDiT의 아키텍처와 응용

단일 모달리티(unimodal)에서 성공을 거둔 확산 트랜스포머(DiT)의 원칙은 자연스럽게 여러 이종(heterogeneous) 데이터 스트림을 동시에 처리해야 하는 멀티모달(multi-modal) 영역으로 확장되었다. 멀티모달 확산 트랜스포머(Multi-Modal Diffusion Transformer, MMDiT)는 DiT 아키텍처를 일반화하여, 텍스트, 오디오, 이미지, 깊이 맵, 생체 신호 등 다양한 형태의 데이터를 단일 통합 생성 프레임워크 내에서 융합하고 생성하는 강력한 모델로 진화했다. 본 장에서는 MMDiT의 개념을 정의하고, 정교한 MMDiT 구현 사례인 MoDA 프레임워크를 심층 분석하며, 다양한 도메인에서 MMDiT가 어떻게 활용되고 있는지 살펴봄으로써 그 구조적 유연성과 잠재력을 탐구한다.

### 3.1  멀티모달 확산 트랜스포머(MMDiT)의 정의

MMDiT는 DiT의 진화된 형태로, 여러 데이터 모달리티 간의 복잡한 상호작용을 모델링하도록 설계되었다.16 핵심적인 아키텍처 원리는 DiT와 동일하게 토큰 시퀀스를 처리하는 트랜스포머 백본을 사용한다. 그러나 MMDiT의 핵심 혁신은 각기 다른 모달리티를 어떻게 토큰화하고, 이들 간의 상호작용을 트랜스포머 블록 내에서 어떻게 효과적으로 관리하는지에 있다. 즉, MMDiT는 단순히 여러 입력을 받는 DiT가 아니라, 각 모달리티의 고유한 특성을 보존하면서도 이들 간의 의미론적 관계를 깊이 있게 학습하여 시너지를 창출하는 것을 목표로 한다.

### 3.2  표현력 있는 생성을 위한 사례 연구: MoDA 프레임워크

정교한 MMDiT 구현의 대표적인 사례는 말하는 아바타(talking head) 비디오를 생성하는 MoDA(Multi-modal Diffusion framework for Audio-driven talking head generation) 프레임워크이다.16 MoDA의 목표는 단일 인물 이미지와 오디오 입력을 기반으로, 감정과 정체성을 유지하면서 자연스러운 입 모양과 머리 움직임을 가진 비디오를 생성하는 것이다. 이를 위해 MoDA는 복잡한 멀티모달 정보를 효과적으로 융합하는 독창적인 MMDiT 아키텍처를 제안한다.

#### 3.2.1  아키텍처 심층 분석

MoDA의 MMDiT 아키텍처는 여러 핵심 구성 요소로 이루어져 있다.19

- **결합 파라미터 공간(Joint Parameter Space):** MoDA는 먼저 다양한 모달리티를 공통의 표현 공간으로 가져온다. 여기에는 노이즈가 추가된 모션 특징(fm), wav2vec 인코더로 추출된 오디오 특징(fa), 원본 이미지의 정규화된 키포인트에서 파생된 정체성 특징(fi), 그리고 시각적 감정 분류기에서 추출된 감정 특징(fe)이 포함된다. 이들을 모두 토큰화하여 트랜스포머의 입력으로 사용한다.
- **MMDiT 블록:** 트랜스포머 블록 내부에서는 멀티모달 상호작용을 촉진하기 위한 특별한 설계가 적용되었다.
  - **모달리티 특화 경로(Modality-specific Paths):** 각 모달리티는 고유한 AdaLN과 변조(modulation) 메커니즘을 가진 독립적인 처리 경로를 거친다. 이는 각 모달리티의 고유한 특성을 초기에 포착하고 조건부 생성 능력을 강화하는 역할을 한다.
  - **결합 어텐션(Joint-Attention):** 이것이 MMDiT의 핵심이다. 각 모달리티는 먼저 각자의 쿼리(Q), 키(K), 밸류(V) 표현으로 투영된다. 그 후, 이 Q, K, V 표현들이 시퀀스 차원을 따라 모두 연결(concatenate)되어 단일 멀티헤드 셀프 어텐션 연산을 통과한다. 어텐션 연산이 끝난 후, 결과물은 다시 원래의 모달리티별 특징으로 분리된다. 이 메커니즘을 통해 모든 모달리티의 토큰들이 서로 직접적으로 상호작용하며 깊은 수준의 교차 모달(cross-modal) 융합이 이루어진다.
- **점진적 융합 전략(Coarse-to-Fine Fusion Strategy):** MoDA 아키텍처의 가장 중요한 혁신 중 하나는 점진적 융합 전략이다. 이는 여러 모달리티 간에 존재하는 의미론적 중복(semantic overlap) 문제를 해결하고 학습 안정성을 높이기 위해 고안되었다. 예를 들어, 오디오의 톤에는 감정 정보가, 목소리 톤에는 정체성 정보가 포함되어 있으며, 이는 이미지에서 추출된 감정 및 정체성 특징과 중복될 수 있다.19 이러한 중복을 효과적으로 처리하기 위해 MoDA는 3단계의 융합 과정을 거친다.
  1. **4-스트림 단계(Four-stream Stage):** 학습 초기에는 노이즈 모션, 오디오, 정체성, 감정 등 네 가지 모달리티가 독립적인 스트림으로 처리된다. 이를 통해 모델은 각 모달리티의 고유한 표현을 학습하고 초기 단계의 특징 분리 및 정렬을 용이하게 한다.
  2. **2-스트림 단계(Two-stream Stage):** 중간 단계에서는 의미론적으로 관련된 모달리티들을 결합한다. 구체적으로 오디오 스트림이 감정 및 정체성 특징과 연결되어 가중치를 공유하는 병합된 스트림을 형성한다. 이 설계는 오디오와 참조 이미지 양쪽에서 오는 감정 및 정체성 단서의 균형 잡힌 통합을 유도한다.
  3. **단일 스트림 처리(Single-stream Process):** 마지막 단계에서는 모든 모달리티가 단일 표현 스트림으로 통합된다. 이를 통해 가장 깊은 수준의 교차 모달 융합이 이루어지며, 모델의 전반적인 생성 표현력을 극대화한다.

이러한 점진적 융합 전략은 MMDiT가 단순한 '토큰의 집합'을 처리하는 모델이 아님을 보여준다. 이는 멀티모달 데이터의 복잡한 의미론적 계층과 중복성을 다루기 위해 명시적으로 설계된 구조적 편향(architectural bias)이다. 모델에게 "각 모달리티의 고유성을 먼저 배우고(4-스트림), 그 다음 관련 개념들을 묶어서 배우고(2-스트림), 마지막으로 모든 것을 통합하라(단일 스트림)"는 일종의 학습 커리큘럼을 제공하는 것이다. 이 구조화된 정보 융합 방식은 MoDA의 성공에 핵심적인 역할을 하며, 향후 복잡한 멀티모달 생성 과제를 위한 중요한 설계 패턴을 제시한다.

### 3.3  MMDiT 프레임워크의 다재다능함

MMDiT의 구조적 유연성은 말하는 아바타 생성을 넘어 다양한 분야로 빠르게 확장되고 있다. 이는 MMDiT가 특정 응용 프로그램에 국한되지 않는 범용적인 멀티모달 생성 엔진으로서의 잠재력을 가지고 있음을 증명한다.

- **3D 생성 (Matrix3D):** Matrix3D는 이미지, 카메라 파라미터, 깊이 맵과 같은 여러 모달리티를 통합하여 3D 콘텐츠를 생성하는 MMDiT를 활용한다.17 이는 2D 정보와 3D 기하학 정보 간의 복잡한 관계를 학습하여 일관된 3D 객체를 생성하는 데 DiT 아키텍처가 효과적임을 보여준다.
- **의료 신호 합성 (UniCardio):** UniCardio는 광혈류측정신호(PPG), 심전도(ECG), 혈압(BP)과 같이 서로 상관관계가 있는 여러 심혈관 신호를 재구성하고 합성하는 통합 프레임워크에 MMDiT를 적용했다.18 이를 통해 하나의 신호가 손상되거나 없을 때 다른 신호로부터 추론하여 복원하는 등 의료 진단에 중요한 역할을 할 수 있다.
- **로보틱스 (Diffusion Transformer Policy):** 로봇 제어 분야에서도 MMDiT가 활용된다. 시각 정보와 언어 명령을 입력받아 연속적인 로봇 팔의 행동 시퀀스를 생성하는 '확산 트랜스포머 정책'은 복잡한 조작 작업을 학습하는 데 MMDiT의 강력한 시퀀스 모델링 능력을 이용한다.20
- **해석 가능성 (ConceptAttention):** ConceptAttention 연구는 MMDiT의 내부 작동 방식을 탐구한다. 이 연구는 텍스트-이미지 MMDiT의 풍부한 내부 표현을 활용하여 추가 학습 없이도 이미지 내 특정 텍스트 개념(예: '용', '하늘')의 위치를 시각화하는 현저성 맵(saliency map)을 생성할 수 있음을 보였다.21 이는 MMDiT가 단순히 패턴을 암기하는 것이 아니라, 여러 모달리티에 걸쳐 전이 가능한(transferable) 시각적 개념을 학습하고 있음을 시사한다.

이처럼 MMDiT는 다양한 데이터 유형과 과제에 맞춰 유연하게 변형될 수 있는 강력한 아키텍처임이 입증되고 있다. 이는 DiT의 확장성 원칙이 멀티모달 영역에서도 성공적으로 적용될 수 있음을 보여주며, 향후 더욱 복잡하고 통합적인 AI 시스템 개발의 핵심 구성 요소가 될 것임을 예고한다.

## 4.  시뮬레이션으로의 확장: OpenAI Sora에 담긴 DiT 원리

OpenAI의 Sora는 현재까지 공개된 가장 강력하고 주목받는 DiT 원리의 응용 사례이다. Sora는 DiT 아키텍처를 비디오 영역으로 확장하고, 이를 방대한 데이터 및 정교한 주석 전략과 결합함으로써 단순한 비디오 생성을 넘어 '세계 시뮬레이터(world simulator)'로서의 새로운 가능성을 제시했다. 본 장에서는 DiT가 비디오 생성을 위해 어떻게 변형되었는지, 그리고 대규모 데이터와 '재주석(recaptioning)' 기법이 어떻게 결합하여 전례 없는 성능과 창발적(emergent) 능력을 이끌어냈는지 심층적으로 분석한다.

### 4.1  이미지 패치에서 시공간 패치로: 비디오를 위한 DiT의 변형

Sora는 그 핵심에 확산 트랜스포머 아키텍처를 두고 있으며, DiT의 기본 원리를 직접적으로 계승하고 확장한다.22 Sora는 DiT를 시간 차원으로 확장하기 위해 몇 가지 핵심적인 아키텍처 구성 요소를 도입했다.25

1. **비디오 압축 네트워크(Video Compression Network):** 이는 LDM/DiT의 VAE와 유사한 역할을 한다. 원본 비디오를 입력으로 받아 공간적(spatially) 및 시간적(temporally)으로 압축된 저차원 잠재 공간 표현으로 변환한다. Sora는 이 압축된 잠재 공간 내에서 모든 생성 과정을 수행함으로써 계산 효율성을 극대화한다.
2. **시공간 잠재 패치(Spacetime Latent Patches):** 이것이 Sora의 가장 중요한 혁신이다. 압축된 잠재 비디오 표현은 '시공간 패치'라는 단위의 시퀀스로 분해된다.25 이는 DiT의 2차원 공간 패치를 시간 차원까지 포함하도록 확장한 개념이다. 예를 들어, 각 패치는 공간적 위치뿐만 아니라 시간적 위치 정보도 함께 가지게 된다. 이로써 트랜스포머는 비디오 전체를 하나의 통일된 시퀀스로 처리할 수 있게 되며, 프레임 내의 공간적 관계와 프레임 간의 시간적 관계를 동일한 셀프 어텐션 메커니즘으로 동시에 학습할 수 있다.

이러한 패치 기반 표현 방식은 NaViT와 같은 모델에서 영감을 받은 아키텍처 수정과 결합하여, Sora가 비디오를 자르거나 크기를 조정할 필요 없이 다양한 해상도, 길이, 화면 비율을 직접 처리할 수 있게 만든다.25 이는 생성 모델 학습의 일반적인 관행을 깨고, 원본 데이터의 고유한 특성을 그대로 학습할 수 있게 하여 생성 결과물의 구도와 프레이밍을 향상시키는 데 기여했다.

### 4.2  비결: 확장 가능한 데이터와 재주석의 힘

Sora의 경이로운 성능은 단지 아키텍처의 우수성만으로는 설명될 수 없으며, 데이터 전략이 결정적인 역할을 했다.22 특히 DALL-E 3에서 계승된 

**재주석(recaptioning) 기법**은 Sora의 성공에 핵심적인 요소이다.24 인터넷에서 수집된 일반적인 비디오 캡션은 짧거나, 부정확하거나, 비디오의 중요한 시각적 세부 사항을 누락하는 경우가 많다. 재주석은 이러한 '노이즈가 많은' 레이블 문제를 해결하기 위한 전략이다.

DALL-E 3 논문에서 상세히 설명된 이 과정은 다음과 같다.30

- **고성능 캡셔너 모델 훈련:** 먼저, 이미지를 이해하고 설명하는 강력한 이미지 캡셔너 모델을 훈련시킨다. 이 캡셔너는 단순히 이미지의 주요 객체를 식별하는 것을 넘어, 배경, 스타일, 색상, 객체 간의 관계 등 시각적 내용을 매우 상세하고 서술적으로 묘사하도록 미세 조정(fine-tuning)된다.
- **합성 캡션 생성:** 훈련된 캡셔너를 사용하여 방대한 시각 훈련 데이터(이미지 및 비디오 프레임)에 대해 새롭고 매우 상세한 '합성 캡션(synthetic caption)'을 생성한다.
- **합성 캡션으로 모델 훈련:** 최종 텍스트-비디오 생성 모델은 이 우수한 품질의 합성 캡션을 사용하여 훈련된다. 이를 통해 모델은 텍스트 프롬프트와 시각적 결과물 간의 정렬(alignment)을 훨씬 더 정밀하게 학습하게 되어, 프롬프트 준수 능력과 전반적인 생성 품질이 극적으로 향상된다.
- **블렌딩 전략:** 캡셔너 모델의 특정 스타일이나 편향에 과적합되는 것을 방지하기 위해, 훈련 데이터에 일정 비율의 원본(ground-truth) 캡션을 섞는 블렌딩 전략이 사용된다. 예를 들어, 95%의 합성 캡션과 5%의 원본 캡션을 혼합하여 훈련함으로써 모델의 강건성(robustness)을 높인다.30

재주석 기법의 효과는 정량적으로 입증되었다. 아래 **표 2**는 DALL-E 3 연구에서 캡션 유형에 따라 텍스트-이미지 모델의 성능이 어떻게 변하는지를 보여준다.

**표 2: 재주석 기법이 생성 모델에 미치는 정량적 영향**

| 캡션 유형                              | 합성 캡션 비율 | CLIP 점수 (ViT-B/32) |
| -------------------------------------- | -------------- | -------------------- |
| 원본 캡션 (Ground Truth)               | 0%             | 27.2                 |
| 짧은 합성 캡션 (Short Synthetic)       | 95%            | 32.0                 |
| 상세 합성 캡션 (Descriptive Synthetic) | 95%            | **33.5**             |

참고: CLIP 점수는 텍스트와 이미지 간의 의미론적 유사성을 측정하는 지표로, 높을수록 좋습니다. 데이터는 DALL-E 3 논문 30의 그림 4를 기반으로 합니다.

표 2는 재주석의 극적인 효과를 명확히 보여준다. 단순히 원본 캡션을 사용하는 것에 비해, 상세한 합성 캡션을 사용했을 때 CLIP 점수가 27.2에서 33.5로 약 23% 향상되었다. 이는 모델이 사용자의 지시를 훨씬 더 충실하게 따를 수 있게 되었음을 의미한다. 이 데이터는 Sora의 성공이 단지 아키텍처나 데이터의 양에만 있는 것이 아니라, 데이터의 '질', 즉 주석의 품질을 체계적으로 향상시키는 전략에 크게 의존하고 있음을 강력하게 뒷받침한다.

### 4.3  창발적 능력: '세계 시뮬레이터'의 탄생

DiT 아키텍처를 대규모 비디오 데이터와 고품질 캡션으로 훈련시켰을 때, Sora는 흥미로운 창발적 능력들을 보여주었다. OpenAI는 이를 바탕으로 Sora를 단순한 비디오 생성기를 넘어 물리적 세계의 일부 측면을 시뮬레이션할 수 있는 "세계 시뮬레이터"로 규정했다.22 관찰된 주요 창발적 능력은 다음과 같다.23

- **3D 일관성(3D Consistency):** 동적인 카메라 움직임이 있는 비디오를 생성할 때, 장면 내의 객체와 환경이 일관된 3차원 구조를 유지하는 것처럼 보인다.
- **장거리 일관성 및 객체 영속성(Long-Range Coherence & Object Permanence):** 긴 비디오에서도 객체나 인물이 일시적으로 화면 밖으로 나가거나 다른 물체에 가려졌다가 다시 나타나도 그 정체성과 외형을 일관되게 유지한다.
- **세계와의 상호작용(Interacting with the World):** 사람이 햄버거를 먹으면 베어 문 자국이 남는 것과 같이, 간단한 인과 관계를 시뮬레이션할 수 있다.
- **디지털 세계 시뮬레이션:** 마인크래프트(Minecraft)와 같은 비디오 게임의 플레이를 생성하는 능력을 보여주었다. 이는 모델이 게임의 규칙과 역학을 어느 정도 내재적으로 학습했음을 시사한다.

물론 이러한 능력에는 명백한 한계가 존재한다. Sora는 유리가 깨지는 것과 같은 복잡한 물리 현상을 정확하게 모델링하지 못하며, 매우 긴 비디오에서는 일관성이 깨지거나 객체가 비논리적으로 나타나는 현상이 발생한다.25 그럼에도 불구하고, 이러한 창발적 능력은 대규모 생성 모델이 명시적인 3D나 물리 엔진 없이도 데이터만으로 세계의 작동 방식에 대한 암묵적인 모델을 학습할 수 있는 가능성을 보여준다.

### 4.4  안전, 윤리, 그리고 레드팀

Sora와 같이 강력한 모델은 오용될 경우 심각한 사회적 위험을 초래할 수 있다. 이에 OpenAI는 Sora 시스템 카드(System Card)를 통해 다층적인 안전 장치와 레드팀(Red Team) 활동 결과를 공개했다.24

- **안전 완화 조치:**
  - **분류기(Classifiers):** 텍스트 입력과 비디오 출력을 스캔하여 아동 성 학대물(CSAM), 폭력, 유해한 딥페이크 등 정책 위반 콘텐츠를 탐지하고 차단하는 다중 모달 분류기를 사용한다. 여기에는 Thorn의 CSAM 분류기와 같은 외부 기술도 통합되었다.24
  - **출처 확인(Provenance):** 생성된 모든 비디오에는 AI 생성물임을 식별할 수 있는 C2PA 메타데이터가 포함되며, 기본적으로 눈에 보이는 워터마크가 추가된다.32
- **레드팀 발견 사항:** 외부 전문가들로 구성된 레드팀은 Sora의 취약점을 사전에 발견하기 위해 체계적인 테스트를 수행했다.24
  - **회피 기법:** 은유나 특정 설정(예: 공상 과학)을 사용한 프롬프트가 안전 장치를 우회할 수 있는 경로가 됨을 발견했다.
  - **편집 도구의 위험성:** 유해하지 않은 이미지나 비디오를 입력으로 사용하여 편집 도구(리믹스, 블렌드 등)를 통해 유해한 콘텐츠를 생성할 수 있는 가능성이 확인되었으며, 이는 입력 및 출력 필터링 강화의 필요성으로 이어졌다.24

Sora의 성공은 단일 기술의 돌파구가 아니라, 세 가지 핵심 요소의 시너지적 확장 결과로 이해해야 한다. 첫째, 계산량에 따라 성능이 향상되는 **확장 가능한 아키텍처(DiT)**가 엔진 역할을 했다. 둘째, 인터넷 스케일의 방대한 비디오 데이터, 즉 **확장 가능한 데이터**가 연료를 제공했다. 셋째, DALL-E 3의 재주석 기법과 같은 **확장 가능한 주석** 전략이 이 연료의 품질을 높여 엔진이 훨씬 더 효율적으로 작동하도록 만들었다. 이 '생성 AI 삼위일체(Generative AI Triad)'는 Sora가 이전 모델들을 뛰어넘는 질적 도약을 이룬 근본적인 이유이며, 창발적 세계 시뮬레이션 능력은 이 전체 시스템이 전례 없는 규모로 작동할 때 나타나는 결과물이다.

## 5.  확장되는 생태계: 최신 발전과 미래 개척

DiT/MMDiT 패러다임의 성공은 단일 모델의 등장을 넘어, 관련 연구의 폭발적인 성장을 촉발하는 촉매제가 되었다. 이 아키텍처의 강점(확장성)은 새로운 응용 분야를 개척하는 기회를 제공했고, 약점(계산 비용)은 해결해야 할 새로운 연구 과제를 제시했다. 그 결과, DiT를 중심으로 효율성 향상, 새로운 도메인으로의 확장, 핵심 기능의 정교화라는 세 가지 주요 축을 중심으로 방대한 연구 생태계가 형성되고 있다. 본 장에서는 이 역동적인 생태계를 조망하며, DiT 패러다임의 한계를 극복하고 그 가능성을 확장하기 위한 최신 연구 동향을 체계적으로 분석한다.

### 5.1  효율성 추구: 계산 비용 길들이기

DiT의 뛰어난 확장성은 막대한 계산 비용을 대가로 한다. 이는 특히 추론(inference) 과정에서 많은 시간을 소요하게 만들어 실시간 응용이나 자원이 제한된 환경에서의 배포를 어렵게 한다.34 이러한 병목 현상을 해결하기 위해 다양한 효율화 기법들이 활발히 연구되고 있다.

- **캐싱 및 중복성 활용:** 확산 모델의 반복적인 노이즈 제거 과정에서, 특히 비디오의 경우, 프레임 간에 변화가 적은 영역(예: 정적인 배경)에 대한 계산은 중복적이다. DeepCache, FastCache, Block-wise Adaptive Caching과 같은 기법들은 이러한 시간적 중복성을 활용한다.36 이들은 이전 타임스텝의 계산 결과나 은닉 상태(hidden state)를 캐싱(caching)하고 재사용함으로써 불필요한 연산을 건너뛰어 추론 속도를 높인다.
- **모델 양자화(Quantization):** 모델의 가중치와 활성화 값의 수치 정밀도를 기존의 32비트 부동소수점에서 8비트나 4비트 정수 등으로 낮추는 기술이다.39 이는 모델의 메모리 점유율을 줄이고, 정수 연산을 지원하는 하드웨어에서 추론 속도를 가속화한다. DiT 아키텍처는 타임스텝에 따라 활성화 값의 분포가 크게 변하는 등 양자화에 특유의 어려움이 있으며, 이를 해결하기 위해 Q-DiT, ViDiT-Q와 같은 DiT 특화 양자화 기법들이 제안되었다.36
- **희소 어텐션 및 토큰 프루닝(Sparse Attention & Token Pruning):** 트랜스포머의 셀프 어텐션은 계산 비용이 높다. 희소 어텐션 기법은 모든 토큰 쌍 간의 상호작용을 계산하는 대신, 의미적으로 중요하거나 공간적으로 인접한 토큰들 간의 상호작용에만 계산을 집중하여 효율성을 높인다.42 토큰 프루닝은 생성 과정에서 중요도가 낮은 토큰을 동적으로 제거하여 처리할 시퀀스의 길이를 줄임으로써 계산량을 절감하는 방식이다.35

### 5.2  미개척지로의 확장: 새로운 도메인과 범용 모델

DiT/MMDiT 아키텍처의 유연성과 확장성은 2D 이미지 및 비디오 생성을 넘어, 더 복잡하고 새로운 데이터 도메인으로 그 영역을 넓히고 있다.

- **네이티브 3D 생성 (Direct3D):** 3D 콘텐츠 생성은 DiT 패러다임이 적용된 중요한 신규 분야이다. Direct3D 모델은 3D 형상을 '트라이플레인(triplane)'이라는 잠재 공간으로 인코딩하는 D3D-VAE와, 이 3D 잠재 공간에서 직접 확산 과정을 수행하는 D3D-DiT를 결합한다.45 이는 여러 2D 뷰를 생성하여 3D로 재구성하는 간접적인 방식(예: SDS)을 탈피하고, 대규모 3D 데이터셋에서 직접 학습하여 고품질의 3D 형상을 생성하는 확장 가능한 네이티브 3D 생성 모델의 가능성을 열었다.47
- **범용 비전 파운데이션 모델 (LaVin-DiT):** LaVin-DiT는 DiT 아키텍처를 사용하여 20가지가 넘는 다양한 컴퓨터 비전 과제(예: 깊이 추정, 시맨틱 분할, 인페인팅)를 단일 통합 모델로 해결하려는 시도이다.50 이 모델의 핵심은 '인컨텍스트 학습(in-context learning)'을 활용하는 것이다. 추론 시, 해결하고자 하는 과제의 예시(입력-출력 쌍)를 컨텍스트로 제공하면, 모델은 별도의 미세 조정 없이 해당 과제를 수행하도록 일반화된다. 이는 DiT가 특정 생성 작업을 넘어, 범용적인 시각적 이해와 추론을 위한 강력한 백본이 될 수 있음을 보여준다.52
- **로보틱스 및 시각-운동 제어 (Diffusion Policy):** 로보틱스 분야에서는 DiT/MMDiT 아키텍처가 복잡하고 연속적인 행동 시퀀스를 학습하는 데 사용된다.20 로봇이 시각적 입력을 받아 연속적인 팔의 움직임을 생성해야 하는 과제에서, DiT는 시각 정보와 행동 출력 간의 관계를 모델링하는 강력한 시퀀스 모델로서 기능한다. 이는 DiT가 가상 세계의 시각적 데이터를 생성하는 것을 넘어, 물리적 세계와 상호작용하는 에이전트를 위한 정책을 학습하는 데에도 적용될 수 있음을 시사한다.38

### 5.3  핵심 기능의 정교화: 제어 가능성과 현실성을 향하여

DiT 기반 모델의 생성 품질이 높아짐에 따라, 연구의 초점은 단순히 '보기 좋은' 결과물을 만드는 것을 넘어, 사용자가 원하는 대로 정밀하게 제어하고, 물리적으로 타당한 결과물을 생성하는 방향으로 이동하고 있다.

- **시간적 일관성(Temporal Consistency):** 비디오 생성의 고질적인 문제인 프레임 간 '깜빡임(flickering)'이나 불일치 현상을 해결하기 위한 연구가 활발하다. 시공간 어텐션 모듈(STAM)을 도입하거나, 광학 흐름(optical flow)을 이용해 프레임 간 움직임을 명시적으로 모델링하거나, Video-3DGS처럼 3D 정보를 활용하여 일관성을 강제하는 등 다양한 접근법이 제안되고 있다.55
- **제어 가능성 및 비디오 편집(Controllability and Video Editing):** 사용자가 생성 과정에 더 적극적으로 개입할 수 있도록 만드는 연구는 거대한 하위 분야를 형성했다. 여기에는 특정 객체나 인물을 일관되게 생성하는 기술, 특정 화풍을 적용하는 기술, 그리고 스케치나 레이아웃 맵을 통해 공간적 구성을 제어하는 기술 등이 포함된다.59 또한, 기존 비디오를 텍스트 지시에 따라 수정하는 비디오 편집 프레임워크들도 DiT 기반 모델을 활용하여 개발되고 있다.62
- **움직임 유도 및 분리(Motion Guidance & Disentanglement):** 비디오에서 객체의 외형(appearance)과 움직임(motion)을 분리하여 독립적으로 제어하려는 연구도 중요한 흐름이다. 광학 흐름을 가이던스로 사용하여 픽셀의 이동 경로를 직접 지정하거나 65, 모델이 잠재 공간에서 외형과 움직임에 대한 분리된 표현(disentangled representation)을 학습하도록 유도하는 방법들이 연구되고 있다.66
- **물리 법칙 기반 생성(Physics-Informed Generation):** 생성 모델의 다음 개척지는 시각적 그럴듯함을 넘어 물리적 타당성을 확보하는 것이다. 초기 연구들은 LLM을 사용하여 텍스트 프롬프트로부터 물리적 맥락을 추론하고 이를 생성 가이드로 활용하거나(DiffPhy) 69, 물리 방정식에 기반한 손실 항을 훈련 과정에 직접 통합하는 방식을 탐구하고 있다.71 이는 생성 모델이 단순히 데이터의 패턴을 모방하는 것을 넘어, 세계가 작동하는 근본적인 규칙을 이해하도록 만드는 중요한 단계이다.73

DiT 아키텍처의 등장은 단순히 새로운 모델 클래스를 탄생시킨 것에 그치지 않았다. 이는 AI 연구 커뮤니티 전반에 걸쳐 다방면의 연구 의제를 촉발하는 강력한 플랫폼 역할을 하고 있다. DiT의 근본적인 강점인 확장성은 3D, 범용 비전 모델 등 새로운 기회를 창출했으며, 동시에 근본적인 약점인 계산 비용은 효율화라는 거대한 연구 분야를 낳았다. 또한, Sora와 같은 모델이 보여준 인상적인 능력은 그 한계(물리 법칙의 부재, 제어의 어려움)를 명확히 드러냈고, 이는 다시 시간적 일관성, 제어 가능성, 물리 기반 생성이라는 새로운 연구의 물결을 일으켰다. 이처럼 DiT 패러다임은 정적인 연구 대상을 넘어, 그 자체의 강점과 약점을 통해 AI 커뮤니티의 연구 방향을 역동적으로 이끌어가는 엔진으로 기능하고 있다.

## 6.  종합 및 결론적 분석

본 보고서는 멀티모달 확산 트랜스포머(MMDiT) 모델에 대한 심층적 고찰을 통해, 이 아키텍처가 생성 AI 분야에서 차지하는 중추적인 위치와 그 발전 경로를 추적했다. U-Net의 귀납적 편향에서 트랜스포머의 계산적 확장성으로의 패러다임 전환에서 시작하여, DiT의 기본 원리, MMDiT로의 진화, Sora를 통한 잠재력의 폭발, 그리고 이를 중심으로 형성된 광범위한 연구 생태계에 이르기까지, DiT 패러다임의 모든 측면을 분석했다. 본 장에서는 이러한 분석을 종합하여 DiT의 발전 궤적을 요약하고, 그 성공의 핵심 원리를 규명하며, 앞으로 해결해야 할 중대한 과제와 미래 전망을 제시하고자 한다.

### 6.1  종합: 확산 트랜스포머의 발전 궤적

확산 트랜스포머의 서사는 명확한 발전 단계를 거쳐왔다.

1. **패러다임의 전환:** 첫 단계는 이미지 데이터에 대한 '지식'을 내재한 특화된 아키텍처인 U-Net에서, 범용적이지만 계산량에 따라 성능이 예측 가능하게 향상되는 '계산 기반' 아키텍처인 트랜스포머로의 전환이었다. 이는 생성 모델 설계 철학의 근본적인 변화를 의미했다.
2. **확장 법칙의 확립:** DiT는 Gflops라는 계산 복잡도 지표가 파라미터 수보다 모델 성능을 더 잘 예측한다는 '확장 법칙'을 경험적으로 증명했다. 이는 생성 모델의 성능 향상을 위한 명확한 경로, 즉 계산량의 증대를 제시했다.
3. **멀티모달로의 진화:** MMDiT는 DiT의 유연성을 활용하여 텍스트, 오디오, 이미지 등 이종의 데이터를 단일 프레임워크 내에서 융합하는 능력으로 발전했다. MoDA의 '점진적 융합 전략'과 같은 정교한 아키텍처는 복잡한 멀티모달 정보 처리의 새로운 가능성을 열었다.
4. **잠재력의 실현:** OpenAI의 Sora는 이 패러다임의 정점을 보여주었다. '생성 AI 삼위일체'—확장 가능한 아키텍처(DiT), 확장 가능한 데이터(인터넷 스케일 비디오), 확장 가능한 주석(재주석 기법)—의 결합은 단순한 비디오 생성을 넘어, 물리적 세계의 일부를 시뮬레이션하는 듯한 창발적 능력을 이끌어냈다.

이러한 궤적은 DiT 패러다임이 단일 모델을 넘어, 생성 AI의 발전을 이끄는 강력하고 일반적인 원리임을 증명한다.

### 6.2  통일 이론: 핵심 원리로서의 확장성

DiT, MMDiT, Sora의 성공을 관통하는 통일된 원리는 바로 '확장성(Scalability)'의 끊임없는 추구와 활용이다. 트랜스포머 아키텍처가 채택된 이유는 그것이 본질적으로 이미지 처리에 더 뛰어나서가 아니라, 계산 자원 투입에 따라 성능이 더 예측 가능하고 효과적으로 *확장*되기 때문이었다.9

이는 AI 분야의 더 넓은 추세와 맥을 같이 한다. 언어 모델 분야에서 GPT와 같은 모델들은 규모(계산량, 데이터, 파라미터)의 양적 증가가 능력의 질적 도약으로 이어진다는 소위 '쓰라린 교훈(The Bitter Lesson)'을 증명했다.74 DiT와 그 후속 모델들은 이 교훈이 시각 데이터 영역에도 강력하게 적용됨을 보여주었다. 즉, 정교한 사전 지식이나 특화된 아키텍처를 설계하는 것보다, 단순하고 확장 가능한 아키텍처를 엄청난 규모의 데이터와 계산으로 훈련시키는 것이 결국 더 우수한 성능을 낳는다는 원칙이 생성 모델 분야에서도 핵심 원리로 자리 잡게 된 것이다.

### 6.3  미래 전망: 앞으로의 거대한 도전 과제

현재의 연구 생태계 분석을 바탕으로, 차세대 MMDiT 연구를 정의할 주요 도전 과제와 미래 방향을 다음과 같이 전망할 수 있다.

- **시뮬레이션에서 이해로:** 현재 모델들은 물리적으로 '그럴듯한(plausible)' 비디오를 생성하지만, 물리적으로 '정확한(accurate)' 비디오를 생성하지는 못한다. 데이터 기반의 패턴 매칭을 넘어, 실제 물리 법칙에 대한 깊은 이해를 모델에 통합하는 것이 다음 단계의 핵심 과제이다. 최근의 물리 법칙 기반 생성 연구들은 이 도전을 해결하기 위한 초기 단계에 있다.69
- **규모의 지속 가능성:** 확장 법칙은 필연적으로 계속해서 증가하는 계산 자원을 요구한다.74 이는 장기적으로 에너지 소비, 비용, 환경적 측면에서 지속 가능성 문제를 제기한다. 미래의 발전은 더 큰 모델을 훈련시키는 것과 동시에, 제 5장에서 논의된 효율화 기법들처럼 더 효율적인 아키텍처와 이를 지원하는 특화된 하드웨어를 공동으로 개발하는 방향으로 나아갈 것이다.
- **진정한 제어 가능성과 분리의 달성:** 현재의 생성 모델은 인상적이지만 종종 예측 불가능한 결과를 내놓는다. 이를 정밀하고, 신뢰할 수 있으며, 직관적인 창작 도구로 발전시키기 위해서는 콘텐츠, 스타일, 움직임, 구도와 같은 핵심 속성들을 독립적으로 조작할 수 있도록 하는 '분리(disentanglement)' 문제를 해결해야 한다.66 이는 사용자가 의도한 바를 정확히 구현하는 데 필수적이다.
- **강건하고 검증 가능한 안전성:** 모델이 더욱 강력해지고 사회 곳곳에 통합됨에 따라, 단순한 분류기를 넘어서는 완벽에 가까운 안전 시스템의 필요성이 절실해진다. 여기에는 위변조를 방지하고 출처를 명확히 하는 강력한 기술, 정교한 딥페이크 탐지 기술, 그리고 모델이 미묘하게 편향되거나 오해의 소지가 있는 콘텐츠를 생성하는 문제를 해결하는 것이 포함된다.

### 6.4 결론

결론적으로, MMDiT는 생성 AI의 종착점이 아니라, 새로운 시대를 여는 강력하고 기초적인 베이스라인으로 자리매김했다. 이 패러다임은 시각 생성 분야의 아키텍처적 접근법을 AI의 다른 분야와 통일시켰으며, '규모'라는 원칙에 기반한 명확하지만 도전적인 미래 경로를 제시했다. 다음 시대의 연구는 이 강력한 힘을 더 효율적이고, 제어 가능하며, 이해할 수 있고, 안전하게 만드는 노력에 의해 정의될 것이다. MMDiT가 제시한 청사진 위에서, 인공지능은 현실을 모방하는 것을 넘어, 현실을 이해하고 창조적으로 상호작용하는 단계로 나아갈 것이다.

#### **참고 자료**

1. Scalable Diffusion Models With Transformers | PDF | Learning | Cognitive Science - Scribd, accessed July 19, 2025, https://www.scribd.com/document/682018575/Scalable-Diffusion-Models-with-Transformers
2. Inductive Bias in Machine Learning - Universität Tübingen, accessed July 19, 2025, https://publikationen.uni-tuebingen.de/xmlui/bitstream/handle/10900/135988/thesis.pdf?sequence=1&isAllowed=y
3. Inductive Bias In Machine Learning | by Sanjithkumar - Medium, accessed July 19, 2025, https://medium.com/@sanjithkumar986/inductive-bias-in-machine-learning-f360ea678a15
4. Incorporating Inductive Bias into Deep Learning: A Perspective from Automated Visual Inspection in Aircraft Maintenance - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/328334741_Incorporating_Inductive_Bias_into_Deep_Learning_A_Perspective_from_Automated_Visual_Inspection_in_Aircraft_Maintenance
5. Vision Transformer (ViT) Decoded: A New Era in Computer Vision | by Sayantan Ghosh, accessed July 19, 2025, https://medium.com/@gsayantan1999/vision-transformer-vit-decoded-a-new-era-in-computer-vision-8d3df7434821
6. Leveraging Inductive Bias in ViT for Medical Image Diagnosis - BMVA Archive, accessed July 19, 2025, https://bmva-archive.org.uk/bmvc/2024/papers/Paper_670/paper.pdf
7. Vision Transformer (ViT) Explained - Ultralytics, accessed July 19, 2025, https://www.ultralytics.com/glossary/vision-transformer-vit
8. Inductive Bias In Deep Learning — 1 | by Sanjithkumar - Medium, accessed July 19, 2025, https://medium.com/@sanjithkumar986/inductive-bias-in-deep-learning-1-17a7c3f35381
9. Scalable Diffusion Models with Transformers - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/ICCV2023/papers/Peebles_Scalable_Diffusion_Models_with_Transformers_ICCV_2023_paper.pdf
10. Introduction to Vision Transformers (ViT) - Encord, accessed July 19, 2025, https://encord.com/blog/vision-transformers/
11. An Explanation of the Vision Transformer (ViT) Paper | by Zaynab Awofeso - Medium, accessed July 19, 2025, https://medium.com/codex/an-explanation-of-the-vision-transformer-vit-paper-8cdd399741aa
12. Vision Transformers (ViT) in Image Recognition: Full Guide - viso.ai, accessed July 19, 2025, https://viso.ai/deep-learning/vision-transformer-vit/
13. [2212.09748] Scalable Diffusion Models with Transformers, accessed July 19, 2025, https://ar5iv.labs.arxiv.org/html/2212.09748
14. Scalable Diffusion Models with Transformers - William Peebles, accessed July 19, 2025, https://www.wpeebles.com/DiT.html
15. (PDF) Scalable Diffusion Models with Transformers - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/366423225_Scalable_Diffusion_Models_with_Transformers
16. MoDA: Multi-modal Diffusion Architecture for Talking Head Generation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2507.03256v1
17. [2502.07685] Matrix3D: Large Photogrammetry Model All-in-One - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2502.07685
18. Versatile Cardiovascular Signal Generation with a Unified Diffusion Transformer - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2505.22306
19. MoDA: Multi-modal Diffusion Architecture for Talking Head ... - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2507.03256
20. [2410.15959] Diffusion Transformer Policy - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2410.15959
21. ConceptAttention: Diffusion Transformers Learn Highly Interpretable Features - arXiv, accessed July 19, 2025, https://arxiv.org/html/2502.04320v1
22. Under The Hood: How OpenAI's Sora Model Works - Factorial Funds, accessed July 19, 2025, https://www.factorialfunds.com/blog/under-the-hood-how-openai-s-sora-model-works
23. Sora: A Review on Background, Technology, Limitations, and Opportunities of Large Vision Models - arXiv, accessed July 19, 2025, https://arxiv.org/html/2402.17177v3
24. Sora System Card - OpenAI, accessed July 19, 2025, https://openai.com/index/sora-system-card/
25. Video generation models as world simulators | OpenAI, accessed July 19, 2025, https://openai.com/index/video-generation-models-as-world-simulators/
26. Unveiling Sora: A Revolutionary Video Generation Model | by Vishal Tiwari | Medium, accessed July 19, 2025, https://medium.com/@vishaltiwari/unveiling-sora-a-revolutionary-video-generation-model-for-business-analytics-01c3d002141e
27. OpenAI Sora's Technical Review - Jianing Qi, accessed July 19, 2025, https://j-qi.medium.com/openai-soras-technical-review-a8f85b44cb7f
28. [D] The Tech Behind The Magic : How OpenAI SORA Works : r/MachineLearning - Reddit, accessed July 19, 2025, https://www.reddit.com/r/MachineLearning/comments/1bqmn86/d_the_tech_behind_the_magic_how_openai_sora_works/
29. Detailed Guide On OpenAI's SORA - Medium, accessed July 19, 2025, https://medium.com/ikarusxr/what-is-openais-sora-44f7080107fc
30. Improving Image Generation with Better Captions - OpenAI, accessed July 19, 2025, https://cdn.openai.com/papers/dall-e-3.pdf
31. Paper Summary #12 - Image Recaptioning in DALL-E 3 | Shreyansh Singh, accessed July 19, 2025, https://shreyansh26.github.io/post/2024-02-18_dalle3_image_recaptioner/
32. Sora is here - OpenAI, accessed July 19, 2025, https://openai.com/index/sora-is-here/
33. Sora: Creating video from text - OpenAI, accessed July 19, 2025, https://openai.com/index/sora/
34. arXiv:2412.12444v3 [cs.LG] 21 Mar 2025, accessed July 19, 2025, [https://arxiv.org/pdf/2412.12444?](https://arxiv.org/pdf/2412.12444)
35. (PDF) Training-Free Efficient Video Generation via Dynamic Token Carving - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/391991604_Training-Free_Efficient_Video_Generation_via_Dynamic_Token_Carving
36. QuantCache: Adaptive Importance-Guided Quantization with Hierarchical Latent and Layer Caching for Video Generation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2503.06545v1
37. FastCache: Fast Caching for Diffusion Transformer Through Learnable Linear Approximation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2505.20353v1
38. Block-wise Adaptive Caching for Accelerating Diffusion Policy - arXiv, accessed July 19, 2025, https://arxiv.org/html/2506.13456v1
39. Q-DiT: Accurate Post-Training Quantization for Diffusion Transformers - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/381703958_Q-DiT_Accurate_Post-Training_Quantization_for_Diffusion_Transformers
40. ViDiT-Q: Efficient and Accurate Quantization of Diffusion Transformers for Image and Video Generation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2406.02540v1
41. Q-VDiT: Towards Accurate Quantization and Distillation of Video-Generation Diffusion Transformers - arXiv, accessed July 19, 2025, https://arxiv.org/html/2505.22167v1
42. Daily Papers - Hugging Face, accessed July 19, 2025, [https://huggingface.co/papers?q=Diffusion%20Transformers%20(DiTs)](https://huggingface.co/papers?q=Diffusion+Transformers+(DiTs))
43. Astraea: A GPU-Oriented Token-wise Acceleration Framework for Video Diffusion Transformers - arXiv, accessed July 19, 2025, https://arxiv.org/html/2506.05096v2
44. Region-Adaptive Sampling Accelerates Diffusion Transformers - AIverse, accessed July 19, 2025, https://www.getaiverse.com/post/region-adaptives-sampling-beschleunigung-von-diffusion-transformers
45. NeurIPS Poster Direct3D: Scalable Image-to-3D Generation via 3D Latent Diffusion Transformer, accessed July 19, 2025, https://neurips.cc/virtual/2024/poster/93214
46. Direct3D: Scalable Image-to-3D Generation via 3D Latent Diffusion Transformer | OpenReview, accessed July 19, 2025, [https://openreview.net/forum?id=vCOgjBIZuL¬eId=j1NdkCt7JB](https://openreview.net/forum?id=vCOgjBIZuL&noteId=j1NdkCt7JB)
47. Direct3D: Scalable Image-to-3D Generation via 3D Latent ... - NIPS, accessed July 19, 2025, https://proceedings.neurips.cc/paper_files/paper/2024/file/dc970c91c0a82c6e4cb3c4af7bff5388-Paper-Conference.pdf
48. Direct3D: Scalable Image-to-3D Generation via 3D Latent Diffusion Transformer | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/380847723_Direct3D_Scalable_Image-to-3D_Generation_via_3D_Latent_Diffusion_Transformer
49. [Literature Review] Direct3D: Scalable Image-to-3D Generation via 3D Latent Diffusion Transformer - Moonlight, accessed July 19, 2025, https://www.themoonlight.io/en/review/direct3d-scalable-image-to-3d-generation-via-3d-latent-diffusion-transformer
50. LaVin-DiT: Large Vision Diffusion Transformer - arXiv, accessed July 19, 2025, https://arxiv.org/html/2411.11505v4
51. [2411.11505] LaVin-DiT: Large Vision Diffusion Transformer - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2411.11505
52. LaVin-DiT - Zhaoqing Wang, accessed July 19, 2025, https://derrickwang005.github.io/LaVin-DiT/
53. LaVin-DiT: Large Vision Diffusion Transformer Supplementary Material, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2025/supplemental/Wang_LaVin-DiT_Large_Vision_CVPR_2025_supplemental.pdf
54. Scaling Diffusion Policy in Transformer to 1 Billion Parameters for Robotic Manipulation, accessed July 19, 2025, https://arxiv.org/html/2409.14411v1
55. Spatial Degradation-Aware and Temporal Consistent Diffusion Model for Compressed Video Super-Resolution - arXiv, accessed July 19, 2025, https://arxiv.org/html/2502.07381v2
56. Enhance-A-Video: Better Generated Video for Free - arXiv, accessed July 19, 2025, https://arxiv.org/html/2502.07508v3
57. Enhancing Temporal Consistency in Video Editing by Reconstructing Videos with 3D Gaussian Splatting | OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=s1zfBJysbI
58. VersVideo: Leveraging Enhanced Temporal Diffusion Models for Versatile Video Generation | OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=K9sVJ17zvB
59. A Survey of Multimodal Controllable Diffusion Models - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/382475500_A_Survey_of_Multimodal_Controllable_Diffusion_Models
60. A Survey of Multimodal Controllable Diffusion Models - SciOpen, accessed July 19, 2025, https://www.sciopen.com/article/10.1007/s11390-024-3814-0
61. PRIV-Creation/Awesome-Controllable-T2I-Diffusion-Models - GitHub, accessed July 19, 2025, https://github.com/PRIV-Creation/Awesome-Controllable-T2I-Diffusion-Models
62. [2506.05934] FADE: Frequency-Aware Diffusion Model Factorization for Video Editing - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2506.05934
63. EffiVED:Efficient Video Editing via Text-instruction Diffusion Models - arXiv, accessed July 19, 2025, https://arxiv.org/html/2403.11568v1
64. VideoDirector: Precise Video Editing via Text-to-Video Models - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2025/papers/Wang_VideoDirector_Precise_Video_Editing_via_Text-to-Video_Models_CVPR_2025_paper.pdf
65. Motion Guidance: Diffusion-Based Image Editing with Differentiable Motion Estimators, accessed July 19, 2025, https://dangeng.github.io/motion_guidance/
66. Unsupervised Learning of Disentangled Representations from Video - NIPS, accessed July 19, 2025, http://papers.neurips.cc/paper/7028-unsupervised-learning-of-disentangled-representations-from-video.pdf
67. Daily Papers - Hugging Face, accessed July 19, 2025, [https://huggingface.co/papers?q=part-disentangled%20motion%20injection](https://huggingface.co/papers?q=part-disentangled+motion+injection)
68. DEEP PROBABILISTIC VIDEO COMPRESSION - OpenReview, accessed July 19, 2025, https://openreview.net/pdf?id=r1l-e3Cqtm
69. [2503.23368] VLIPP: Towards Physically Plausible Video Generation with Vision and Language Informed Physical Prior - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2503.23368
70. Think Before You Diffuse: LLMs-Guided Physics-Aware Video Generation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2505.21653v1
71. [2403.14404] Physics-Informed Diffusion Models - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2403.14404
72. A Physics-informed Diffusion Model for High-fidelity Flow Field Reconstruction - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2211.14680
73. Reasoning Physical Video Generation with Diffusion Timestep Tokens via Reinforcement Learning - arXiv, accessed July 19, 2025, https://arxiv.org/html/2504.15932v1
74. AI and compute | OpenAI, accessed July 19, 2025, https://openai.com/index/ai-and-compute/
75. Towards Physically Plausible Video Generation via VLM Planning - arXiv, accessed July 19, 2025, https://arxiv.org/html/2503.23368v1


# Skip-DiT 모델

## 초록

본 보고서는 Diffusion Transformer(DiT)가 직면한 추론 비효율성과 특징(feature) 불안정성이라는 중대한 과제를 해결하기 위해 제안된 아키텍처 수정안인 Skip-DiT 모델에 대한 포괄적인 기술적 분석을 제공한다. 먼저 U-Net 기반 확산 모델에서부터 DiT로 이어지는 기술적 배경을 설명하고, Skip-DiT가 해결하고자 하는 핵심 문제, 즉 기존 캐싱(caching) 기법을 비효율적으로 만드는 트랜스포머 블록 간의 높은 특징 표현 분산(‘feature smoothness’ 문제)을 심도 있게 다룬다. 이어서 저자들이 제안한 이중 해결책을 해부한다. 첫째는 특징 동역학을 안정화시키기 위해 Long-Skip-Connection(LSC)을 도입한 **Skip-DiT 아키텍처**이며, 둘째는 이 연결을 활용하여 생성 속도를 가속하는 추론 시간 전략인 **Skip-Cache 메커니즘**이다. 본 보고서는 Skip-DiT의 성능을 속도 향상과 생성 품질 측면에서 엄밀하게 분석하고, 이를 기본 DiT 및 Δ-DiT, DeepCache와 같은 경쟁 가속 기법들과 벤치마킹한다. 마지막으로, 모델의 실제 구현, 알려진 한계점, 그리고 더 효율적이고 확장 가능한 생성 모델을 구축하려는 지속적인 노력 속에서 Skip-DiT가 갖는 광범위한 중요성에 대해 논한다.

------

## 1.  기본 원리: Diffusion Transformer의 등장 배경

### 1.1  Denoising Diffusion Probabilistic Model (DDPM) 프레임워크

확산 모델의 핵심은 두 가지 프로세스로 구성된다. 첫째는 원본 데이터 x0에 점진적으로 가우시안 노이즈를 추가하여 잠재 변수 xt를 생성하는 순방향 프로세스(forward process) $q(x_t|x_0)$이다. 이 프로세스는 수학적으로 다음과 같이 정의된다.1
$$
q(x_t∣x_0)=N(x_t;\sqrt{\bar{α}_t} x_0,(1−\bar{α}_t)\mathbf{I})
$$
여기서 $\bar{\alpha}_t$는 노이즈 스케줄을 결정하는 하이퍼파라미터이다.

둘째는 이 순방향 프로세스를 역으로 되돌리는 학습 가능한 역방향 프로세스(reverse process) $p_\theta(x_{t-1}|x_t)$이다.1 확산 모델의 목표는 신경망 $ϵ_θ$를 학습하여 각 타임스텝 t에서 추가된 실제 노이즈 $ϵ_t$를 예측하도록 하는 것이다. 학습은 예측된 노이즈 $\epsilon_\theta(x_t, t)$와 실제 노이즈 $ϵ_t$ 간의 평균 제곱 오차(Mean-Squared Error) 손실을 최소화하는 방식으로 이루어진다.1 이는 복잡한 변분 하한(Variational Lower Bound, VLB) 목적 함수를 단순화하여 효율적인 학습을 가능하게 한다.

학습이 완료된 후, 순수한 가우시안 노이즈 $x_T \sim \mathcal{N}(0, \mathbf{I})$에서 시작하여 학습된 노이즈 예측 네트워크 $ϵ_θ$를 반복적으로 적용함으로써 점차 노이즈를 제거하고 최종적으로 깨끗한 샘플 $x_0$를 생성한다.1 이 반복적인 샘플링 과정은 고품질의 이미지를 생성하는 비결이지만, 동시에 수백에서 수천 번의 네트워크 평가를 요구하기 때문에 높은 계산 비용과 긴 추론 시간의 주된 원인이 된다.5

### 1.2  아키텍처의 전환: U-Net에서 Vision Transformer (ViT) 백본으로

초기 확산 모델들은 대부분 U-Net 아키텍처를 노이즈 제거 네트워크의 백본으로 사용했다.3 U-Net은 인코더-디코더 구조를 가지며, 동일한 해상도 레벨의 인코더 블록과 디코더 블록을 연결하는 스킵 커넥션(skip connection)이 특징이다.9 이 스킵 커넥션은 저수준 특징을 보존하고 그래디언트 흐름을 원활하게 하는 중요한 역할을 수행하며, 이는 후술할 Skip-DiT 논의에서 다시 중요하게 다뤄진다.

한편, Vision Transformer(ViT)는 컨볼루션 신경망(CNN)보다 시각 인식 과제에서 뛰어난 확장성(scalability)을 보여주며 주목받았다.1 이러한 ViT의 성공에 힘입어, Peebles와 Xie(2022)는 확산 모델의 U-Net 백본을 트랜스포머로 대체하는 Diffusion Transformer(DiT)를 제안했다.1 이 시도는 트랜스포머의 우수한 확장성이 생성 모델의 성능 향상에도 기여할 것이라는 가설에 기반했다.

이러한 아키텍처의 전환은 트랜스포머의 확장성에 대한 전략적 선택이었지만, 의도치 않게 U-Net의 핵심 설계 요소였던 인코더와 디코더 간의 장거리 스킵 커넥션(long-range skip connection)을 배제하는 결과를 낳았다. 이 구조적 차이는 DiT가 가진 특징 안정성의 역할을 충분히 고려하지 않은 결과로, 훗날 Skip-DiT가 해결해야 할 근본적인 문제의 단초를 제공했다. U-Net은 저수준 특징과 고수준 특징을 융합하는 내장된 메커니즘을 가졌지만 9, DiT는 확장성을 위해 이를 단순하고 균일한 트랜스포머 블록의 연속으로 대체했다.1 이로 인해 단일 순전파 내에서 네트워크의 초기(얕은) 부분의 특징을 보존하고 재사용하는 명시적인 메커니즘이 사라졌고, 이는 DiT의 새로운 효율성 문제를 야기했다.

### 1.3  Diffusion Transformer (DiT)의 구조

DiT는 다음과 같은 주요 구성 요소로 이루어진다.

- **잠재 공간(Latent Space) 연산:** DiT는 계산 효율성을 위해 원본 이미지 픽셀 공간에서 직접 작동하는 대신, VAE(Variational Autoencoder)와 같은 인코더 E를 사용하여 이미지를 저차원의 잠재 공간 표현 $z = E(x)$로 압축한다. 확산 프로세스는 이 잠재 표현 z에 적용되며, 최종적으로 생성된 잠재 벡터는 디코더 $D$를 통해 다시 픽셀 공간 $x = D(z)$로 복원된다.1 이는 컨볼루션 기반의 VAE와 트랜스포머 기반의 확산 모델을 결합한 하이브리드 접근 방식이다.
- **Patchify 및 토큰화:** ViT의 표준 방식에 따라, 2D 잠재 표현 $z$는 겹치지 않는 여러 개의 패치(patch)로 분할된다. 각 패치는 선형 투영(linear projection)을 통해 1D 토큰(token)으로 변환되어 트랜스포머가 처리할 수 있는 시퀀스 형태를 갖추게 된다.11
- **트랜스포머 블록:** 모델의 핵심은 여러 개의 표준 트랜스포머 블록의 연속으로 구성된다. 각 블록은 멀티헤드 자기-어텐션(multi-head self-attention)과 피드포워드 네트워크(feed-forward network)를 포함한다.1 DiT의 확장성은 블록의 수(깊이 $N$), 은닉 차원의 크기(너비 $d$), 어텐션 헤드의 수를 늘림으로써 달성된다.1
- **조건부 안내 및 Adaptive Layer Normalization (adaLN):** 생성 과정을 제어하기 위해 타임스텝 $t$와 클래스 레이블 $c$와 같은 조건부 정보가 모델에 주입되어야 한다. DiT는 이를 위한 여러 방법을 탐색했으며, 그중 가장 효율적이고 성능이 우수한 것으로 입증된 것은 **adaLN(Adaptive Layer Normalization)**이다.11 adaLN은 조건 벡터 임베딩으로부터 Layer Normalization에 필요한 스케일($γ$) 및 시프트($β$) 파라미터를 직접 예측한다. 이 방식은 추가적인 계산 오버헤드를 최소화하면서 네트워크 전체에 걸쳐 특징을 효과적으로 조절할 수 있게 한다. 여기서 더 나아가 `adaLN-Zero` 변형은 잔차 연결(residual connection) 직전에 적용되는 스케일 파라미터 $α$를 추가하고 이를 $0$으로 초기화함으로써, 전체 DiT 블록이 항등 함수(identity function)처럼 시작하도록 만들어 학습 안정성을 더욱 향상시킨다.4

------

## 2.  핵심 기술 과제: Diffusion Transformer의 특징 불안정성

### 2.1  반복적 노이즈 제거의 계산 부담

확산 모델의 근본적인 병목 현상은 샘플링 과정의 순차적 특성에서 비롯된다. 단일 샘플을 생성하기 위해 전체 노이즈 제거 네트워크를 수백, 수천 번 반복적으로 평가해야 하기 때문이다.5 이러한 문제는 특히 DiT 아키텍처에서 더욱 심화된다. DiT의 핵심 연산인 자기-어텐션 메커니즘의 계산 복잡도는 입력 토큰 시퀀스의 길이에 제곱으로 비례($O(N^2$))하기 때문이다.5 이는 이미지 해상도나 비디오 길이가 증가할수록 계산량이 기하급수적으로 늘어남을 의미하며, 고해상도 비디오 생성과 같은 대규모 작업에 DiT를 적용하는 것을 매우 어렵게 만든다.8

### 2.2  'Feature Smoothness' 문제: 캐싱의 장벽

이러한 계산 부담을 줄이기 위한 유망한 전략 중 하나는 **특징 캐싱(feature caching)**이다. 이는 한 타임스텝에서 계산된 중간 특징을 다음 타임스텝에서 재사용하여 중복 계산을 피하는 기법이다.3 하지만 DiT 모델에 대한 실증 분석 결과, 이 접근법이 간단하지 않음이 밝혀졌다. Skip-DiT의 저자들은 DiT의 트랜스포머 블록 내부 특징을 시각화한 결과, 연속적인 타임스텝 간, 그리고 동일 타임스텝 내의 다른 블록 간에 "상당한 특징 변동(significant feature variation)"이 존재함을 발견했다.5

저자들은 이 현상을 **'feature smoothness'**라는 용어로 정의했다. 캐싱이 효과적이려면 특징 표현이 부드럽고 예측 가능하게 변해야 하는데, 바닐라 DiT에서는 특징들이 "높은 분산(high feature variance)" 8과 "동적 특징 불안정성(dynamic feature instability)" 18을 보였다. 이러한 불안정성은 특징을 순진하게 캐싱하여 재사용할 경우 오차가 증폭되는 결과로 이어진다.18

이 문제의 근원은 알고리즘이 아닌 아키텍처에 있다. DiT의 균일한 블록 체인 구조는 확장성을 극대화했지만, 동시에 각 블록이 이전 블록의 출력을 공격적으로 변환하면서 내부 상태를 불안정하게 만들고 시간적 재사용을 어렵게 만든다. 따라서 DiT에 캐싱을 효과적으로 적용하기 위해서는 단순히 캐싱 알고리즘을 개선하는 것을 넘어, 아키텍처 자체를 수정하여 특징을 캐싱에 적합하게 만들어야 한다는 결론에 이르게 된다.

이러한 문제 인식의 심화는 관련 논문의 제목 변화에서도 엿볼 수 있다. 초기 버전의 제목인 "Accelerating Vision Diffusion Transformers with Skip Branches" 16는 해결책('skip branches')과 목표('가속')에 초점을 맞춘 실용적 제목이었다. 그러나 후기 버전인 "Towards Stabilized and Efficient Diffusion Transformers through Long-Skip-Connections with Spectral Constraints" 19는 "안정화(Stabilized)", "장거리 스킵 커넥션(Long-Skip-Connections, LSCs)", 그리고 "스펙트럼 제약(Spectral Constraints)"과 같은 더 깊은 개념을 도입한다. 이는 저자들이 단지 추론 속도 문제를 넘어 근본적인 "동적 특징 불안정성" 19을 해결하고 있으며, 그 해결책이 U-Net의 핵심 원리(LSC)를 재도입하는 것임을 명확히 한 것이다. 더 나아가 "스펙트럼 제약"의 언급은 가중치 행렬의 스펙트럼 노름(spectral norm)을 제어하여 그래디언트 흐름을 안정시키는 이론적 근거를 통해 LSC의 작동 원리를 설명하려는 시도를 보여준다.21 이는 휴리스틱한 해결책에서 더 이론적으로 견고한 프레임워크로 연구가 발전했음을 시사한다.

### 2.3  U-Net 캐싱 패러다임이 등방성 트랜스포머 아키텍처에서 실패하는 이유

DeepCache와 같은 캐싱 기법들은 U-Net 아키텍처를 위해 설계되었으며, 여기서는 높은 효과를 보였다.24 이들은 U-Net의 고수준(깊은) 특징을 캐싱하고 저수준(얕은) 특징만 재계산하는 방식으로 작동한다.9

그러나 이 전략은 "근본적인 아키텍처의 차이" 때문에 DiT에 직접 적용하기 어렵다.5 DiT의 등방성(isotropic) 구조는 U-Net과 달리 명확한 인코더, 디코더, 그리고 이들을 잇는 장거리 스킵 커넥션이 부재하기 때문이다.12 U-Net에서는 스킵 커넥션이 얕은 특징을 디코더로 직접 전달하는 경로를 제공하므로, 깊은 특징이 캐싱되더라도 네트워크는 항상 새로운 저수준 정보를 공급받는다. 반면 DiT에서는 각 블록의 입력이 오직 이전 블록의 출력에만 의존한다. 따라서 중간 블록의 출력을 캐싱하여 훨씬 뒤의 블록에 바로 입력으로 사용하면, 그 사이의 모든 변환 과정과 이전 노이즈 제거 스텝의 이미지 정보가 포함된 잔차 연결까지 모두 손실되는 심각한 정보 손실이 발생한다.12

------

## 3.  Skip-DiT 아키텍처: Long-Skip-Connection을 통한 동역학 안정화

### 3.1  아키텍처 설계: 대칭적 `skip branch`의 통합

Skip-DiT의 핵심적인 아키텍처 수정은 표준 DiT 모델에 **`skip branch`** 또는 **Long-Skip-Connection(LSC)**이라 불리는 새로운 연결을 추가하는 것이다.3 이 연결은 얕은 DiT 블록과 깊은 DiT 블록을 직접 연결하여, 정보가 중간 블록들을 건너뛸 수 있는 경로를 생성한다.3 일반적으로 이 연결은 대칭적으로 구성된다 (예: $i$번째 블록과 $L−i$번째 블록 연결).

`skip branch` 자체는 계산적으로 매우 가벼운 모듈로 설계되었다. 통상적으로 Layer Normalization 이후 선형 계층(Linear layer)이 적용되는 `Norm + Linear` 형태로 구성된다.16 이 덕분에 훈련 과정에서 추가되는 계산 오버헤드는 미미하다.

### 3.2  Skip Branch의 안정화 효과

`skip branch`의 주된 기능은 **특징 평탄성(feature smoothness)을 향상**시키는 것이다.5 초기 단계의 특징이 후기 단계의 계산에 직접적인 영향을 미칠 수 있는 경로를 제공함으로써, 블록 간 및 타임스텝 간의 특징 분산을 줄여준다. 이는 ResNet의 잔차 연결이나 U-Net의 장거리 스킵 커넥션과 개념적으로 유사하며, 이들 모두 그래디언트 흐름을 개선하고 손실 지형(loss landscape)을 완만하게 만들어 더 안정적인 학습을 유도하는 것으로 알려져 있다.3

후기 버전의 논문에서는 LSC가 어떻게 안정적인 그래디언트 흐름을 보장하고 섭동(perturbation)에 대한 민감도를 줄여 특징 안정성을 이론적으로 보장하는지를 **스펙트럼 노름 분석**을 통해 설명한다.18 즉, 

`skip branch`는 단순히 추론 시 캐싱을 위한 트릭이 아니라, 모델 자체의 훈련 및 추론 안정성을 근본적으로 향상시키는 구조적 개선이다. 훈련 안정성과 추론 안정성은 동전의 양면과 같다. 추론 시 효율적인 캐싱을 위해 요구되는 '특징 평탄성'은 훈련 과정에도 이점을 주는 향상된 아키텍처 '안정성'의 직접적인 결과물인 것이다.

### 3.3  2단계 훈련 프로토콜

Skip-DiT 모델은 기존의 강력한 DiT 모델을 효율적으로 개조하기 위해 특별한 2단계 훈련 전략을 사용한다.5 이는 완전히 새로운 아키텍처를 처음부터 학습시키는 막대한 비용을 피하고, 사전 훈련된 백본(예: Latte, Hunyuan-DiT)을 실용적으로 개선할 수 있게 해준다.5

- **1단계: Skip-branch 훈련:** 사전 훈련된 DiT 모델의 가중치를 불러와 동결(freeze)시킨다. 무작위로 초기화된 `skip branch`의 가중치만을 학습시킨다. 이 단계는 모델이 대략적인 형태의 콘텐츠를 생성할 수 있을 때까지 비교적 짧은 시간 동안 진행된다 (예: Latte 모델 기준 8개의 A100 GPU에서 약 1일 소요).5
- **2단계: 전체 훈련:** `skip branch`가 충분히 학습되면, DiT 블록을 포함한 모델의 모든 파라미터를 동결 해제(unfreeze)하고 전체 모델을 종단간(end-to-end)으로 미세 조정한다. 이 단계에서 모델은 빠르게 원래의 생성 능력을 회복하며, 며칠간의 추가 학습을 통해 원본 모델과 동등하거나 그 이상의 품질을 달성하는 것으로 보고되었다.5

------

## 4.  Skip-Cache 메커니즘: 가속화된 추론 심층 분석

### 4.1  캐싱 및 재사용 파이프라인

**Skip-Cache**는 훈련이 아닌 **추론 시점(inference-time)**에만 활성화되는 전략으로, 안정화된 Skip-DiT 아키텍처를 적극적으로 활용한다.3 핵심 아이디어는 특정 타임스텝에서 대부분의 DiT 블록 계산을 생략하여 속도를 높이는 것이다.3

캐싱이 적용되는 타임스텝 $t−1$에서의 파이프라인은 다음과 같이 작동한다 16:

1. 현재 타임스텝의 입력 $x_{t-1}$은 **첫 번째 또는 처음 몇 개의 얕은 DiT 블록**만을 통과한다. 이를 통해 이미지의 저수준, 구조적 특징이 새롭게 계산된다.
2. 마지막으로 계산된 얕은 블록의 출력(예: $x'_{t−1,1}$)은 **이전 타임스텝 t에서 캐싱해 둔 깊은 블록의 출력**(예: $x]_{t,L−1}$)과 결합된다.
3. 이 결합된 정보는 `skip branch`를 통해 최종 DiT 블록으로 직접 전달되어 노이즈가 제거된 출력을 생성한다.
4. 이 과정에서 중간에 위치한 대부분의 DiT 블록(예: 블록 2부터 $L−1$까지) 계산이 완전히 생략되어 막대한 양의 연산을 절약한다.26

이 비대칭적인 캐시 파이프라인은 매우 중요한 설계 결정이다. 확산 모델에서 얕은 레이어는 주로 이미지의 전체적인 구조와 윤곽선을, 깊은 레이어는 세부적인 디테일과 텍스처를 처리하는 경향이 있다.6 Skip-Cache는 새로운 입력 $x_{t-1}$을 처리하여 구조적 특징을 갱신하고(얕은 블록 재계산), 변화가 덜 극적인 세부 사항은 이전 스텝에서 가져와 재사용(깊은 블록 캐시)하는 지능적인 전략을 채택한다. 이는 DeepCache의 U-Net 기반 논리를 DiT 아키텍처에 맞게 재해석한 것이다.9

또한, Skip-Cache는 정적이고 미리 정의된 스케줄에 따라 작동하는 것으로 보인다. 이는 `skip-step 2`와 같은 단순한 커맨드라인 인자를 통해 유추할 수 있다. Skip-DiT 아키텍처가 LSC를 통해 특징을 근본적으로 안정시켰기 때문에, 런타임에 특징 유사도를 일일이 확인하는 복잡한 동적 캐싱 로직의 필요성이 줄어든다. 아키텍처 개선이 캐싱 알고리즘 자체를 단순화시킨 것이다.

### 4.2  `--cache` 파라미터 해부

공식 GitHub 저장소는 Skip-Cache를 활성화하는 커맨드라인 예시를 제공하지만, 파라미터의 의미를 명시적으로 설명하지는 않는다.19 제공된 정보와 유사 연구를 바탕으로 그 의미를 다음과 같이 추론할 수 있다.

- **텍스트-비디오 생성 (Latte 백본):** `python sample/sample_t2v.py --cache N2-700-50` 19
  - `N2`: 캐싱 정책을 나타내는 식별자로 추정된다. 'N'은 비균일(Non-uniform) 스케줄을, '2'는 캐싱 간격(2 스텝마다 캐시)을 의미할 수 있다.
  - `700`: 타임스텝 임계값. 타임스텝이 700 미만일 때, 즉 특징이 비교적 안정되는 후반부 노이즈 제거 과정에서만 캐싱을 적용하도록 설정할 수 있다.
  - `50`: 캐싱을 수행하지 않는 예열(warm-up) 구간을 나타내는 스텝 수 또는 백분율일 수 있다.
- **텍스트-이미지 생성 (Hunyuan-DiT 백본):** `python sample_t2i.py --cache --cache-step 2` 19
  - `--cache`: 캐싱 메커니즘을 활성화하는 불리언(boolean) 플래그.
  - `--cache-step 2`: 캐싱 간격을 2로 설정. 즉, 한 스텝은 캐시를 사용하고 다음 스텝은 재계산하는 방식이다.

**표 1: Skip-Cache 파라미터 해석 (추정)**

| 파라미터      | 예시 값          | 추정 의미                                            |
| ------------- | ---------------- | ---------------------------------------------------- |
| 정책 및 간격  | `N2`             | 비균일(Non-uniform) 캐싱 정책, 2 스텝 간격으로 적용. |
| 임계값        | `700`            | 타임스텝이 700 미만인 구간에서 캐싱 활성화.          |
| 예열 구간     | `50`             | 초기 50 스텝(또는 50%) 동안은 캐싱을 비활성화.       |
| 활성화 플래그 | `--cache`        | Skip-Cache 메커니즘을 전반적으로 활성화.             |
| 간격 설정     | `--cache-step 2` | 2 스텝 간격으로 캐싱을 수행하도록 명시적으로 설정.   |

*주: 위 표의 내용은 제공된 자료를 바탕으로 한 추론이며, 정확한 동작은 소스 코드 분석을 통해 확정될 수 있다.*

------

## 5.  실증 평가 및 성능 벤치마킹

### 5.1  추론 및 훈련 가속: 정량적 분석

Skip-DiT와 Skip-Cache는 추론과 훈련 양쪽에서 상당한 가속 효과를 보인다.

- **추론 속도 향상:** Skip-Cache는 품질 저하가 거의 없이(**negligible quality loss**) **1.5배**의 속도 향상을 달성하며, 약간의 품질 저하를 감수할 경우 최대 **2.2배**의 속도 향상이 가능하다고 보고되었다.3 일부 자료에서는 원본 품질의 99%를 유지하면서 2배의 속도 향상을 이뤘다고 언급하기도 했다.31 이는 다른 최신 캐싱 기법들보다 1.5~2배 추가적인 속도 향상을 의미한다.21
- **훈련 가속:** Skip-DiT 아키텍처 자체의 안정성 덕분에 훈련 과정 또한 효율화된다. 바닐라 DiT와 비교했을 때 최대 **4.4배**의 훈련 가속과 더 빠른 수렴 속도를 달성했다.18
- **효율성 증대:** 이 방법은 추론 속도뿐만 아니라 메모리 사용량을 최대 45%까지 줄이는 효과도 있다.31

**표 2: Skip-DiT 및 Skip-Cache 성능 요약**

| 백본 모델   | 태스크        | 메트릭    | 기준 성능 | Skip-Cache 성능 | 속도 향상   | GPU  |
| ----------- | ------------- | --------- | --------- | --------------- | ----------- | ---- |
| Latte       | 텍스트-비디오 | 품질/속도 | -         | 고품질 유지     | 1.7x - 2.0x | A100 |
| Latte       | 클래스-비디오 | 품질/속도 | -         | 고품질 유지     | 2.2x - 2.5x | A100 |
| Hunyuan-DiT | 텍스트-이미지 | 품질/속도 | -         | 고품질 유지     | ~2.0x       | A100 |
| DiT-XL/2    | 클래스-이미지 | FID       | -         | 약간의 저하     | ~2.2x       | -    |

주: 위 표는 여러 자료 5에서 언급된 성능 수치를 종합한 것이다.

### 5.2  DiT 가속화 기술 비교 분석

DiT 가속화 분야는 활발히 연구되고 있으며, 특히 훈련 없이 적용 가능한 캐싱 기법들이 다수 제안되었다.6

- **DeepCache:** 본래 U-Net을 위해 설계되었으며, 고수준 특징을 캐싱하는 방식이다.25 DiT에 적용 시 아키텍처 불일치로 인해 성능 저하를 겪을 수 있다.12 Skip-DiT는 DeepCache에서 영감을 받았다.26

- **Δ-DiT (Delta-DiT):** 훈련이 필요 없는 적응형 캐싱 기법이다. 샘플링 초기에는 이미지 디테일을 담당하는 깊은(rear) 블록을 캐싱하고, 후기에는 윤곽선을 담당하는 얕은(front) 블록을 캐싱한다.27 또한, DiT의 정보 손실 문제를 해결하기 위해 이전 스텝의 입력을 고려하는 

  `Δ-Cache`를 도입했다.12

- **FORA / Faster-Diff:** DiT 캐싱의 초기 연구들로, Skip-DiT는 이들과의 직접적인 비교에서 더 높은 속도 향상률에서도 우수한 안정성과 품질 보존 능력을 보여준다.19

- **ToCa (Token-wise Caching):** 블록 단위가 아닌 토큰 단위로 캐싱을 수행하며, 덜 중요한 토큰을 적응적으로 선택하여 캐싱한다.34

**표 3: DiT 가속화 기법 비교**

| 기법                      | 핵심 전략                                           | 훈련/미세조정 필요 | 주요 장점                                   |
| ------------------------- | --------------------------------------------------- | ------------------ | ------------------------------------------- |
| **Skip-DiT + Skip-Cache** | 아키텍처 수정(LSC) + 정적 블록 캐싱                 | **필요 (1회)**     | 높은 속도/품질 균형, 훈련 가속              |
| **Δ-DiT**                 | 적응형 블록 캐싱 (초기: 깊은 블록, 후기: 얕은 블록) | 불필요             | 완전한 훈련 불필요, DiT 구조에 특화된 캐시  |
| **DeepCache**             | 고수준 특징 캐싱, 저수준 특징 업데이트              | 불필요             | U-Net에서 매우 효과적, 간단한 플러그인 방식 |
| **ToCa**                  | 적응형 토큰 단위 캐싱                               | 불필요             | 세밀한 제어 가능, 블록 단위보다 유연함      |
| **FORA / Faster-Diff**    | 고정 간격 블록 캐싱                                 | 불필요             | 초기 DiT 캐싱 연구, 구현 용이               |

이 비교를 통해 "훈련 불필요(training-free)"라는 용어의 미묘한 차이가 드러난다. Skip-Cache 자체는 추론 시점에 작동하는 훈련 불필요 메커니즘이지만, 이는 Skip-DiT라는 **아키텍처**에 의존하며, 이 아키텍처는 1회의 미세조정 과정을 요구한다. 반면 Δ-DiT나 DeepCache는 바닐라 DiT 모델에 즉시 적용할 수 있는 진정한 "플러그 앤 플레이" 방식이다. 따라서 사용자는 1회의 미세조정 비용을 지불하고 Skip-DiT의 우수한 속도/품질 균형을 얻을 것인지, 아니면 즉시 적용 가능한 다른 방법을 선택할 것인지 트레이드오프를 고려해야 한다.

### 5.3  시각적 아티팩트 및 품질 저하 분석

모든 캐싱 기법은 근사치 계산에 의존하므로 시각적 아티팩트(artifact)를 유발할 위험이 있다. 일반적인 아티팩트로는 미세한 디테일 손실, 텍스처 뭉개짐, 색상 변화, 또는 비디오에서의 시간적 불일치(jitter) 등이 있다.31

Skip-DiT의 저자들은 중간 수준의 가속에서는 "무시할 만한 품질 손실"을, 높은 가속률에서는 "사소한 감소"만을 보인다고 주장한다.5 LSC 구조 자체가 원본 출력과의 높은 충실도(fidelity)를 유지하도록 설계되었기 때문이다.18 GitHub 저장소에 공개된 시각적 비교 자료를 보면, Skip-DiT는 FORA나 Faster-Diff 같은 다른 방법들보다 더 높은 가속률에서도 우수한 생성 품질(예: PSNR)을 유지하는 것을 확인할 수 있다.19

예를 들어, Latte 백본을 사용한 텍스트-비디오 생성 결과에서 2.0배 가속된 Skip-Cache 영상은 원본과 매우 유사한 품질을 보인다. 경쟁 기법이 배경에서 블러나 텍스처 손실을 보이는 반면, Skip-Cache 결과는 피사체의 선명도와 장면의 생동감을 보존한다. 자세히 관찰해야만 겨우 인지할 수 있는 미세한 배경 디테일의 부드러워짐 외에는 큰 차이가 없다. 이는 Skip-DiT의 아키텍처 안정화가 일반적인 캐싱 아티팩트를 효과적으로 완화한다는 주장을 뒷받침한다. 이러한 성능은 특정 속도/품질 트레이드오프 곡선, 즉 파레토 최적전선(Pareto frontier)에서 Skip-DiT가 경쟁 기술들보다 우위에 있음을 시사한다. 즉, 동일한 속도 향상을 위해 더 적은 품질 저하를 감수하거나, 동일한 품질을 유지하면서 더 높은 속도 향상을 달성할 수 있다는 의미이다.

------

## 6.  광범위한 맥락, 구현 및 한계

### 6.1  모델 가속화 기술 지형에서의 Skip-DiT의 위치

캐싱은 모델 가속화를 위한 여러 전략 중 하나이다. 다른 주요 접근 방식들은 다음과 같다.

- **효율적인 샘플링:** DPM-Solver와 같은 개선된 ODE/SDE 솔버나 LCMs(Latent Consistency Models)과 같은 증류(distillation) 기법을 통해 노이즈 제거 스텝의 수를 줄인다.6
- **모델 압축:** 프루닝(pruning), 양자화(quantization), 증류 등을 통해 스텝당 계산 비용을 줄인다.7

Skip-Cache와 같은 캐싱 기법은 이러한 다른 접근법들과 직교(orthogonal) 관계에 있어, 함께 사용될 경우 시너지 효과를 통해 훨씬 더 큰 속도 향상을 기대할 수 있다.9 예를 들어, Skip-DiT 아키텍처(구조적 안정성)를 LCM으로 학습시키고(적은 스텝 수), 양자화하여 실행(연산당 비용 감소)하는 파이프라인은 미래의 최첨단 가속화 솔루션이 될 수 있다.

### 6.2  실제 구현 및 가용성

Skip-DiT의 공식 PyTorch 구현은 GitHub 저장소를 통해 공개되어 있다.8 또한, Latte-skip, DiT-XL/2-skip 등 다양한 태스크와 백본에 대한 사전 훈련된 모델 체크포인트가 Hugging Face에 배포되어 있어 접근성이 높다.19 저장소에는 추론 및 훈련을 위한 명확한 지침과 스크립트가 포함되어 있다.19

### 6.3  알려진 한계점 및 실패 사례

- **성능 가변성:** 가속 효과는 생성되는 이미지나 비디오의 복잡도에 따라 달라질 수 있다.31
- **디테일 정밀도:** 높은 가속률에서는 일부 미세한 시각적 디테일이 원본 모델보다 덜 정밀하게 표현될 수 있다.31
- **하이퍼파라미터 튜닝:** 최적의 캐싱 파라미터(예: `--cache` 문자열, 임계값 등)는 새로운 모델이나 태스크에 적용할 때 수동으로 튜닝해야 할 수 있다.31
- **실패 사례:** 명시적으로 상세히 기술되지는 않았지만, 잠재적인 실패 사례를 추론할 수 있다. 만약 기반 특징이 가정만큼 안정적이지 않다면 캐싱은 극적인 아티팩트를 유발할 수 있으며, 특히 급격한 움직임이나 복잡한 텍스처 변화가 있는 장면에서 문제가 될 수 있다. 또한, 캐싱된 스텝에서는 새로운 디테일을 생성하는 데 실패하여 "진부하거나" 반복적인 느낌의 결과물을 낳을 수 있다.

### 6.4  커뮤니티 구현과의 구별: `SkipLayerGuidanceDiT` 사례

공식 Skip-DiT와 커뮤니티에서 사용되는 유사한 이름의 도구를 명확히 구별하는 것이 중요하다. 예를 들어, ComfyUI의 `SkipLayerGuidanceDiT` 노드는 Skip-DiT와는 다른 목적을 가진다.43

- `SkipLayerGuidanceDiT`는 **안내(guidance)** 기법으로 보인다. 이는 Perturbed Attention Guidance와 같은 기법에서 영감을 받아, 특정 레이어를 건너뛰어 모델의 어텐션을 조작하고 특정 예술적 효과나 디테일 강화를 목표로 한다.43 즉, 결과물의 

  **질적 변화**를 유도하는 도구이다.

- Skip-DiT/Skip-Cache는 **가속(acceleration)** 기법이다. 원본 출력을 최대한 충실하게 보존하면서 계산량을 줄이기 위해 블록을 건너뛴다.

이처럼 이름이 유사하여 발생하는 혼동은 빠르게 발전하는 분야에서 흔히 나타나는 문제이다. 하나는 예술적 제어를 위한 것이고 다른 하나는 효율성을 위한 것이라는 점을 명확히 인지해야 한다.

------

## 7. 결론

Skip-DiT는 단순한 캐싱 알고리즘을 넘어, 확장성이 뛰어난 트랜스포머 프레임워크에 U-Net의 강점인 장거리 특징 전달 원리를 재도입한 근본적인 아키텍처 개선이다. 이 구조적 변화는 훈련과 추론 모두에서 특징 동역학을 안정시키는 핵심 동인으로 작용한다.

Skip-DiT의 성공은 가장 효과적인 가속화 기술이 모델을 블랙박스로 취급하는 알고리즘이 아니라, 모델의 아키텍처와 함께 공동 설계되는 것임을 보여준다. 이는 순수한 알고리즘 최적화에서 벗어나, 차세대 고성능 생성 모델을 구축하기 위한 보다 총체적이고 아키텍처를 인지하는(architecture-aware) 접근 방식으로의 전환을 예고한다. Skip-DiT는 이러한 패러다임 전환의 중요한 이정표로서, 향후 더 효율적이고 안정적인 생성 모델 연구에 중요한 기반을 제공할 것이다.

#### **참고 자료**

1. [논문리뷰] Scalable Diffusion Models with Transformers (DiT) - 전생 ..., accessed July 19, 2025, [https://kimjy99.github.io/%EB%85%BC%EB%AC%B8%EB%A6%AC%EB%B7%B0/dit/](https://kimjy99.github.io/논문리뷰/dit/)
2. [DiT] Scalable Diffusion Models with Transformers - Dev In Seoul - 티스토리, accessed July 19, 2025, https://dev-in-seoul.tistory.com/51
3. [논문 리뷰] Accelerating Vision Diffusion Transformers with Skip ..., accessed July 19, 2025, https://www.themoonlight.io/ko/review/accelerating-vision-diffusion-transformers-with-skip-branches
4. 인공지능 기본개념 복습하기 - 26. 이미지 생성모델의 완결 Diffusion Transformers(DiT), accessed July 19, 2025, https://www.youtube.com/watch?v=7-hccTRcOek
5. Accelerating Vision Diffusion Transformers with Skip Branches - arXiv, accessed July 19, 2025, https://arxiv.org/html/2411.17616v2
6. $\Delta$-DiT: Accelerating Diffusion Transformers without training via Denoising Property Alignment | OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=pDI03iK5Bf
7. NeurIPS Poster DiP-GO: A Diffusion Pruner via Few-step Gradient Optimization, accessed July 19, 2025, https://neurips.cc/virtual/2024/poster/93385
8. Accelerating Vision Diffusion Transformers with Skip Branches - arXiv, accessed July 19, 2025, https://arxiv.org/html/2411.17616v1
9. DeepCache: Accelerating Diffusion Models for Free - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Ma_DeepCache_Accelerating_Diffusion_Models_for_Free_CVPR_2024_paper.pdf
10. WF-VAE: Enhancing Video VAE by Wavelet-Driven Energy Flow for Latent Video Diffusion Model, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2025/papers/Li_WF-VAE_Enhancing_Video_VAE_by_Wavelet-Driven_Energy_Flow_for_Latent_CVPR_2025_paper.pdf
11. [paper] Scalable Diffusion Models with Transformers - velog, accessed July 19, 2025, https://velog.io/@koo82/paper-Scalable-Diffusion-Models-with-Transformers
12. DiT: A Training-Free Acceleration Method Tailored for Diffusion Transformers - arXiv, accessed July 19, 2025, [https://arxiv.org/pdf/2406.01125?](https://arxiv.org/pdf/2406.01125)
13. ICML Poster Fast Video Generation with Sliding Tile Attention - ICML 2025, accessed July 19, 2025, https://icml.cc/virtual/2025/poster/45139
14. (PDF) Astraea: A GPU-Oriented Token-wise Acceleration Framework for Video Diffusion Transformers - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/392466752_Astraea_A_GPU-Oriented_Token-wise_Acceleration_Framework_for_Video_Diffusion_Transformers
15. Cache Me if You Can: Accelerating Diffusion Models through Block Caching - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/384498288_Cache_Me_if_You_Can_Accelerating_Diffusion_Models_through_Block_Caching
16. [Skip-DiT 논문 리뷰] - Accelerating Vision Diffusion Transformers ..., accessed July 19, 2025, https://kyujinpy.tistory.com/165
17. Most of our 3-D visualizations feedforward networks had shapes that... - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/figure/Most-of-our-3-D-visualizations-feedforward-networks-had-shapes-that-were-qualitatively_fig3_269935498
18. [2411.17616] Towards Stabilized and Efficient Diffusion Transformers through Long-Skip-Connections with Spectral Constraints - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2411.17616
19. OpenSparseLLMs/Skip-DiT: ✈️ Towards Stabilized and ... - GitHub, accessed July 19, 2025, https://github.com/OpenSparseLLMs/Skip-DiT
20. Paper page - Accelerating Vision Diffusion Transformers with Skip Branches - Hugging Face, accessed July 19, 2025, https://huggingface.co/papers/2411.17616
21. Towards Stabilized and Efficient Diffusion Transformers through Long-Skip-Connections with Spectral Constraints - arXiv, accessed July 19, 2025, https://arxiv.org/html/2411.17616v4
22. GuanjieChen/Skip-DiT - Hugging Face, accessed July 19, 2025, https://huggingface.co/GuanjieChen/Skip-DiT
23. LensLLM: Unveiling Fine-Tuning Dynamics for LLM Selection - OpenReview, accessed July 19, 2025, https://openreview.net/attachment?id=om0CcjvEQh&name=pdf
24. Paper page - DeepCache: Accelerating Diffusion Models for Free - Hugging Face, accessed July 19, 2025, https://huggingface.co/papers/2312.00858
25. DeepCache: Accelerating Diffusion Models for Free - arXiv, accessed July 19, 2025, https://arxiv.org/html/2312.00858v2
26. Update README.md / GuanjieChen/Skip-DiT at d064a8f - Hugging Face, accessed July 19, 2025, https://huggingface.co/GuanjieChen/Skip-DiT/commit/d064a8f1387ff848ff9b6c836c264b90773e7932
27. Accelerating Vision Diffusion Transformers with Skip Branches - Qeios, accessed July 19, 2025, https://www.qeios.com/read/BIM5V9
28. Fugu-MT 論文翻訳(概要): Towards Stabilized and Efficient Diffusion Transformers through Long-Skip-Connections with Spectral Constraints, accessed July 19, 2025, https://fugumt.com/fugumt/paper_check/2411.17616v3
29. README.md / GuanjieChen/Skip-DiT at 0d3ca2bcaa149b2201fbdc2750e5a69fd93e2dac - Hugging Face, accessed July 19, 2025, https://huggingface.co/GuanjieChen/Skip-DiT/blob/0d3ca2bcaa149b2201fbdc2750e5a69fd93e2dac/README.md
30. Cache Me if You Can: Accelerating Diffusion Models through Block Caching - arXiv, accessed July 19, 2025, https://arxiv.org/html/2312.03209v2
31. Accelerating Vision Diffusion Transformers with Skip Branches | AI Research Paper Details, accessed July 19, 2025, https://www.aimodels.fyi/papers/arxiv/accelerating-vision-diffusion-transformers-skip-branches
32. $Delta$-DiT: A Training-Free Acceleration Method Tailored for Diffusion Transformers | AI Research Paper Details - AIModels.fyi, accessed July 19, 2025, https://www.aimodels.fyi/papers/arxiv/dollardeltadollar-dit-training-free-acceleration-method-tailored
33. Δ-DiT: A Training-Free Acceleration Method Tailored for Diffusion Transformers - arXiv, accessed July 19, 2025, https://arxiv.org/html/2406.01125v1
34. Accelerating Diffusion Transformers with Token-wise Feature Caching - OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=yYZbZGo4ei
35. Shenyi-Z/ToCa: Accelerating Diffusion Transformers with Token-wise Feature Caching - GitHub, accessed July 19, 2025, https://github.com/Shenyi-Z/ToCa
36. Cache Me if You Can: Accelerating Diffusion Models through Block Caching, accessed July 19, 2025, https://fwmb.github.io/blockcaching/
37. Cache Me if You Can: Accelerating Diffusion Models through Block Caching - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Wimbauer_Cache_Me_if_You_Can_Accelerating_Diffusion_Models_through_Block_CVPR_2024_paper.pdf
38. StreamDiffusion: A Pipeline-level Solution for Real-time Interactive Generation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2312.12491v2
39. Fast ODE-based Sampling for Diffusion Models in Around 5 Steps - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/384144205_Fast_ODE-based_Sampling_for_Diffusion_Models_in_Around_5_Steps
40. Daily Papers - Hugging Face, accessed July 19, 2025, [https://huggingface.co/papers?q=diffusion%20Transformers](https://huggingface.co/papers?q=diffusion+Transformers)
41. Daily Papers - Hugging Face, accessed July 19, 2025, [https://huggingface.co/papers?q=DiT-based%20video%20generation](https://huggingface.co/papers?q=DiT-based+video+generation)
42. Accelerating Vision Diffusion Transformers with Skip Branches - OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=LmEyq3yNBx
43. SkipLayerGuidanceDiT - RunComfy, accessed July 19, 2025, https://www.runcomfy.com/comfyui-nodes/ComfyUI/skip-layer-guidance-di-t
44. Dramatically enhance the quality of Wan 2.1 using skip layer guidance : r/StableDiffusion, accessed July 19, 2025, https://www.reddit.com/r/StableDiffusion/comments/1jac3wm/dramatically_enhance_the_quality_of_wan_21_using/


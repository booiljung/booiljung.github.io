# 확산 트랜스포머와 전문가 혼합의 융합(DiT-MoE)

확장성, 전문화, 그리고 생성 모델링의 미래

## 1.  생성형 AI의 세 가지 기둥의 융합

생성형 인공지능(AI) 분야는 지난 몇 년간 눈부신 발전을 거듭해왔으며, 그 중심에는 서로 다른 기술적 패러다임의 창의적 융합이 자리 잡고 있습니다. 특히, 확산 트랜스포머 전문가 혼합(Diffusion Transformer with Mixture-of-Experts, DiT-MoE) 모델의 등장은 이러한 융합의 정점을 보여주는 사례입니다. 이 모델은 단순히 기존 기술들을 조합한 것을 넘어, 각 기술의 본질적인 장점을 극대화하고 단점을 보완하며 생성 모델링의 새로운 지평을 열었습니다. DiT-MoE를 심층적으로 이해하기 위해서는 그 근간을 이루는 세 가지 핵심 기술-확산 모델, 트랜스포머 아키텍처, 그리고 전문가 혼합-이 각각 어떻게 발전해왔고, 왜 이들의 만남이 필연적이었는지를 먼저 고찰해야 합니다.

### 1.1  노이즈 제거 확산 프로세스: 확률 모델에서 잠재 공간 생성까지

확산 모델은 고품질 샘플 생성 능력으로 주목받는 강력한 생성 모델 클래스입니다.1 이 모델의 핵심 원리는 점진적인 변환 과정에 있습니다. 크게 두 단계로 구성되는데, 첫 번째는 순방향 프로세스(forward process)로, 원본 데이터에 점진적으로 가우시안 노이즈(Gaussian noise)를 추가하여 순수한 노이즈 상태로 만드는 고정된 과정입니다. 두 번째는 역방향 프로세스(reverse process)로, 모델이 이 노이즈 추가 과정을 역으로 되돌리는 법을 학습하여 순수한 노이즈로부터 원본 데이터를 복원하는, 학습된 과정입니다.2 이러한 반복적인 노이즈 제거(denoising) 방식은 모델이 데이터의 복잡한 분포를 정교하게 학습하도록 하여, 놀라울 정도로 사실적인 고품질 샘플을 생성할 수 있게 합니다.4

초기 확산 모델은 고해상도 이미지의 픽셀 공간에서 직접 이 과정을 수행했기 때문에 엄청난 계산 비용을 요구했습니다. 이 문제를 해결하기 위해 등장한 것이 바로 잠재 확산 모델(Latent Diffusion Models, LDM)입니다.1 LDM은 확산 과정을 고차원의 픽셀 공간이 아닌, 변이형 오토인코더(Variational Autoencoder, VAE)와 같은 인코더를 통해 압축된 저차원의 잠재 공간(latent space)에서 수행합니다.3 이미지를 훨씬 더 작은 차원의 잠재 벡터로 표현한 뒤, 이 잠재 벡터에 노이즈를 추가하고 제거하는 과정을 학습하는 것입니다. 생성이 완료되면 디코더를 통해 잠재 벡터를 다시 픽셀 공간으로 복원합니다. 이 접근법은 계산 복잡도를 극적으로 줄여 고해상도 이미지 합성을 현실적으로 가능하게 만들었으며, 이후 등장할 DiT의 핵심적인 기반이 되었습니다.3

또한, 확산 모델은 조건부 생성(conditional generation)을 통해 그 활용성을 더욱 확장했습니다. 텍스트 프롬프트나 클래스 레이블과 같은 조건 정보를 노이즈 제거 과정에 주입함으로써, 사용자가 원하는 특정 유형의 이미지를 생성하도록 유도할 수 있습니다.1 이 기능은 DiT와 같은 후속 모델에서 텍스트-이미지 생성의 핵심 요소로 자리 잡게 됩니다.

### 1.2  비전 분야에서 트랜스포머의 부상: DiT가 U-Net을 넘어선 이유

초기 확산 모델의 역방향 노이즈 제거 네트워크로는 주로 U-Net 아키텍처가 사용되었습니다.2 U-Net은 컨볼루션 신경망(CNN) 기반으로, 인코더-디코더 구조와 스킵 연결(skip connection)을 통해 이미지의 지역적, 계층적 특징을 효과적으로 포착하는 데 강점을 보였습니다.

그러나 AI 분야 전반에 걸쳐 트랜스포머 아키텍처가 혁명을 일으키면서, 비전 분야에서도 그 영향력이 커지기 시작했습니다. 비전 트랜스포머(Vision Transformer, ViT)는 이미지를 여러 개의 작은 패치(patch)로 나누고, 이를 시퀀스 데이터처럼 처리하는 새로운 패러다임을 제시했습니다.1 트랜스포머의 핵심 메커니즘인 셀프 어텐션(self-attention)은 이미지 내 모든 패치 간의 상호작용을 계산함으로써, CNN이 포착하기 어려웠던 장거리 의존성(long-range dependencies)을 효과적으로 모델링할 수 있었습니다.1

이러한 배경 속에서 UC 버클리의 윌리엄 피블스(William Peebles)와 뉴욕 대학교의 사이닝 시에(Saining Xie)는 LDM의 U-Net 백본을 순수 트랜스포머 아키텍처로 대체하는 과감한 시도를 했고, 그 결과물이 바로 확산 트랜스포머(Diffusion Transformer, DiT)입니다.1 이는 단순한 아키텍처 교체가 아니었습니다. 확산 과정의 특성을 트랜스포머에 효과적으로 통합하기 위해, 노이즈 제거 단계(timestep)와 클래스 레이블 같은 조건 정보를 주입하는 방식으로 적응형 레이어 정규화(adaptive layer norm, adaLN)와 같은 새로운 기법이 도입되었습니다.1

DiT의 등장은 생성 모델링의 패러다임을 근본적으로 전환시켰습니다. U-Net과 같은 기존 아키텍처가 이미지의 공간적 계층 구조라는 특정 귀납적 편향(inductive bias)에 의존했던 것과 달리, DiT의 성공은 연산량(GFLOPs)과 성능(Fréchet Inception Distance, FID) 간의 명확한 스케일링 법칙(scaling law)을 입증했습니다.2 모델의 깊이나 너비를 늘리거나, 입력 패치 크기를 줄여 토큰 수를 늘리는 방식으로 연산량을 증가시키면 FID 점수가 일관되게 향상되는 것이 확인된 것입니다.2 이는 생성 모델의 성능 향상 문제가 '최적의 아키텍처 설계'에서 '효과적인 연산량 확장'으로 재정의되었음을 시사합니다. 이러한 연산 중심적 관점의 확립은, 본질적으로 연산량을 효율적으로 관리하고 확장하기 위한 전략인 전문가 혼합(MoE)의 도입을 위한 필연적인 지적, 경험적 토대를 마련했습니다.

### 1.3  전문가 혼합(MoE) 패러다임: 조건부 연산을 통한 확장성 확보

전문가 혼합(Mixture of Experts, MoE)은 복잡한 문제를 여러 개의 작고 전문화된 하위 신경망, 즉 '전문가(experts)'에게 나누어 처리하게 하는 앙상블 학습 기법입니다.7 각 전문가는 전체 문제의 특정 부분에 집중하도록 훈련되며, '게이팅 네트워크(gating network)' 또는 '라우터(router)'라 불리는 관리자 네트워크가 입력 데이터(토큰)를 보고 어떤 전문가를 활성화할지 동적으로 결정합니다.7

MoE의 가장 큰 장점은 희소성(sparsity)을 통한 효율성입니다. 모든 입력에 대해 모델의 모든 파라미터를 사용하는 '밀집(dense)' 모델과 달리, MoE 모델은 조건부 연산(conditional computation)을 통해 전체 파라미터 중 일부만 활성화합니다.10 이 덕분에 모델의 총 파라미터 수를 수십억, 수천억 개로 크게 늘리면서도, 각 입력 토큰을 처리하는 데 드는 실제 계산 비용(FLOPs)은 상대적으로 낮게 유지할 수 있습니다.11 이는 모델의 용량(capacity)과 실용성 사이의 트레이드오프를 해결하는 혁신적인 방법입니다.

MoE 아키텍처의 핵심 구성 요소는 일반적으로 피드포워드 신경망(FFN) 형태의 전문가들과, 간단한 선형 레이어와 소프트맥스(softmax) 또는 top-k 함수로 구현되는 게이팅 네트워크입니다.7 그러나 MoE는 고유한 과제를 안고 있습니다. 가장 대표적인 것이 바로 부하 분산(load balancing) 문제입니다. 게이팅 네트워크가 특정 소수의 '인기 있는' 전문가에게만 대부분의 토큰을 보내고 다른 전문가들은 거의 사용되지 않는 현상이 발생할 수 있습니다. 이는 훈련 불안정성을 야기하고 모델의 전체 용량을 비효율적으로 사용하는 결과를 낳습니다. 따라서 MoE 모델을 안정적으로 훈련시키기 위해서는 라우팅이 균형을 이루도록 유도하는 보조 손실 함수(auxiliary loss function)가 필수적입니다.9

결론적으로, 확산 모델은 고품질 생성을, 트랜스포머는 뛰어난 확장성을, 그리고 MoE는 효율적인 연산 확장을 가능하게 하는 핵심 열쇠를 제공했습니다. DiT-MoE는 이 세 가지 강력한 패러다임을 하나로 융합하여, 이전에는 불가능했던 규모의 생성 모델을 구축하고 전례 없는 성능을 달성하는 길을 열었습니다.

## 2.  DiT-MoE 아키텍처 심층 분석

"Scaling Diffusion Transformers to 16 Billion Parameters" 논문에서 제시된 DiT-MoE는 기존의 밀집(dense) DiT 아키텍처의 한계를 극복하고 생성 모델의 확장성을 새로운 차원으로 끌어올리기 위해 설계되었습니다. 이 모델의 핵심은 DiT 블록 내의 연산 집약적인 부분을 전문가 혼합(MoE) 레이어로 대체하여, 모델의 총 파라미터 수를 대폭 늘리면서도 추론 시의 실제 연산량은 효율적으로 제어하는 것입니다. 본 장에서는 DiT-MoE의 구체적인 아키텍처, MoE 레이어의 통합 방식, 그리고 모델의 성공을 가능하게 한 핵심 메커니즘들을 상세히 분석합니다.

그림 1: DiT-MoE 블록 아키텍처. 표준 DiT 블록(왼쪽)은 멀티헤드 셀프 어텐션(Multi-Head Self-Attention)과 피드포워드 네트워크(FFN)로 구성됩니다. DiT-MoE 블록(오른쪽)에서는 이 FFN이 MoE 레이어로 대체됩니다. MoE 레이어는 라우터(Router)와 다수의 전문가 MLP(Expert MLPs)로 구성됩니다. 입력 토큰은 라우터를 통해 상위 k개의 전문가에게 전달되고, 이 전문가들의 출력이 가중 합산되어 최종 결과물을 형성합니다. 이 과정을 통해 조건부 연산과 희소 활성화가 구현됩니다. (참고: 13의 Figure 2를 기반으로 개념적으로 재구성)

### 2.1  희소성 통합: DiT 블록의 피드포워드 네트워크를 MoE 레이어로 대체

DiT-MoE의 근본적인 구조적 변경은 DiT 블록 내의 피드포워드 네트워크(FFN), 즉 MLP(Multi-Layer Perceptron) 레이어를 MoE 레이어로 대체하는 것입니다.13 기존의 밀집 DiT 모델에서는 모든 입력 토큰(latent image patch)이 모든 DiT 블록의 FFN을 통과하며 모든 파라미터가 연산에 참여했습니다. 반면, DiT-MoE에서는 일부 또는 전체 DiT 블록의 FFN이 게이팅 네트워크와 다수의 전문가 MLP 집합으로 구성된 MoE 레이어로 교체됩니다.13

이때 DiT-MoE 초기 논문에서 채택한 라우팅 방식은 표준적인 **토큰 선택(token-choice) 라우팅**입니다.15 각 입력 토큰에 대해, 게이팅 네트워크는 사용 가능한 모든 전문가에 대한 선호도 점수(affinity score)를 계산합니다. 그 후, 가장 높은 점수를 받은 상위 k개의 전문가(일반적으로 k=2와 같이 작은 수)가 선택되어 해당 토큰을 처리하게 됩니다. 선택된 전문가들의 출력은 게이팅 네트워크가 계산한 가중치에 따라 합산되어 MoE 레이어의 최종 출력이 됩니다.

이러한 희소 활성화(sparse activation) 구조는 모델의 파라미터 확장에 혁신적인 길을 열어줍니다. 예를 들어, 모델의 총 파라미터 수는 165억 개에 달하지만, 각 토큰을 처리하는 데 실제로 활성화되는 파라미터는 31억 개에 불과할 수 있습니다.14 이는 모델의 용량을 극대화하면서도 추론에 필요한 계산 비용은 합리적인 수준으로 유지할 수 있게 해주는 핵심 원리입니다.

### 2.2  DiT-MoE의 핵심 메커니즘

DiT-MoE의 성공은 단순히 MoE를 DiT에 적용한 것을 넘어, MoE의 고질적인 문제점을 해결하기 위한 두 가지 독창적인 설계 덕분입니다: '공유 전문가 라우팅'과 '전문가 수준 균형 손실'. 이 두 가지 메커니즘은 MoE의 훈련 안정성과 효율성을 크게 향상시켰습니다.

#### 2.2.1  공유 전문가 라우팅 (Shared Expert Routing)

**문제점:** 일반적인 MoE 모델을 훈련시키다 보면, 여러 전문가들이 서로 비슷한, 기초적이고 보편적인 지식(예: 이미지의 기본적인 엣지, 질감, 색상 조합 등)을 중복해서 학습하는 경향이 있습니다.9 이는 파라미터를 비효율적으로 사용하는 것이며, 전문가들이 진정으로 '전문화'되는 것을 방해합니다.

**해결책:** DiT-MoE는 이 문제를 해결하기 위해 **공유 전문가 라우팅(shared expert routing)**이라는 간단하면서도 효과적인 구조를 도입했습니다.18 이 설계에서는 전체 전문가 풀(pool)의 일부(예: 2개)를 '공유 전문가'로 지정합니다. 이 공유 전문가들은 라우팅 결정과 상관없이 

**모든 토큰**에 대해 항상 활성화됩니다. 나머지 전문가들은 기존 방식대로 라우터를 통해 선택적으로 활성화됩니다.13

**영향:** 이 구조를 통해 공유 전문가들은 모든 토큰이 필요로 하는 공통적이고 기초적인 지식을 전담하여 학습하게 됩니다. 반면, 라우팅되는 전문가들은 이러한 공통 지식을 학습할 부담을 덜고, 더 복잡하고 특수한 패턴을 학습하는 데 집중할 수 있게 되어 진정한 의미의 전문화를 이룰 수 있습니다. 이는 파라미터 중복성을 줄이고 전반적인 훈련 효율성과 모델 성능을 향상시키는 데 크게 기여합니다.18 이는 MoE의 고질적인 약점인 전문가 중복성을 순수 손실 함수 기반 규제가 아닌, 구조적 설계를 통해 직접적으로 해결하려는 시도라는 점에서 중요한 의미를 가집니다.

#### 2.2.2  전문가 수준 균형 손실 (Expert-Level Balance Loss)

**문제점:** 앞서 언급했듯이, 토큰 선택 라우팅은 일부 전문가에게 연산이 집중되는 부하 불균형 문제를 야기하기 쉽습니다.

**해결책:** DiT-MoE는 훈련 과정에서 **전문가 수준 균형 손실(expert-level balance loss)**이라는 보조 손실 함수를 도입하여 이 문제를 완화합니다.18 이 손실 함수는 각 전문가에게 할당되는 토큰의 비율을 계산하고, 이 분포가 균일하지 않을 경우 페널티를 부과합니다. 즉, 게이팅 네트워크가 토큰을 가용한 전문가들에게 더 고르게 분배하도록 유도하는 역할을 합니다.20

**중요성:** 이 균형 손실은 안정적인 훈련을 보장하고, 전문가 풀의 모든 전문가들이 효과적으로 활용되어 모델의 전체 용량에 기여하도록 만드는 데 필수적인 요소입니다.20 논문에 따르면, 이 손실 함수를 적용했을 때 모델 성능이 크게 향상되는 것으로 나타났습니다. 공유 전문가 라우팅이 전문가 간의 '역할 분담'을 유도한다면, 전문가 수준 균형 손실은 라우팅되는 전문가들 사이의 '기회 균등'을 보장하는 상호 보완적인 관계에 있습니다. 이 두 가지 혁신적인 설계의 시너지는 DiT-MoE가 165억 파라미터라는 거대한 규모까지 안정적으로 확장될 수 있었던 핵심 동력으로 평가됩니다.

### 2.3  전문가 행동 분석: 전문화의 비밀을 풀다

DiT-MoE 논문의 가장 흥미로운 기여 중 하나는 전문가들이 실제로 무엇에 전문화되는지를 심층적으로 분석한 부분입니다.18 분석 결과는 일반적인 예상과는 사뭇 다른, 생성 과정의 본질을 꿰뚫는 통찰을 제공합니다.

1. **콘텐츠가 아닌 프로세스에 대한 전문화:** 전문가 선택은 **노이즈 제거 타임스텝(denoising timestep)**과 이미지 패치의 **공간적 위치(spatial position)**와 높은 상관관계를 보였습니다. 놀랍게도, 클래스 조건 정보(예: '고양이' 이미지인지 '개' 이미지인지)와는 거의 무관했습니다.18 이는 전문가들이 '고양이 전문가'나 '자동차 전문가'처럼 의미론적(semantic) 개념에 전문화되는 것이 아니라, 생성 과정의 특정 단계나 특정 이미지 영역을 처리하는 '절차적(procedural)' 하위 작업에 전문화됨을 의미합니다.
2. **계층적 전문화:** 모델의 얕은(초반) MoE 레이어에서는 전문가 선택이 특정 공간적 위치에 더 집중되는 경향을 보였습니다. 반면, 깊은(후반) 레이어로 갈수록 전문가 선택은 이미지 전체에 걸쳐 더 분산되고 균형 잡힌 양상을 보였습니다.21
3. **시간적 전문화:** 전문가 전문화는 노이즈가 많은 노이즈 제거 과정의 초기 단계(큰 타임스텝 값)에서 더 뚜렷하고 집중적으로 나타났습니다. 이후 노이즈가 점차 제거되는 후기 단계(작은 타임스텝 값)로 갈수록 전문가 선택은 점차 균일해졌습니다.18

**해석:** 연구진은 이러한 현상을 확산 과정 자체의 특성으로 설명합니다. 모델은 생성 초기에 이미지의 전반적인 구조나 구성과 같은 저주파수 정보(low-frequency information)를 먼저 모델링하고, 이 단계에서는 특정 공간 영역에 특화된 전문가들이 집중적으로 필요합니다. 그 후, 텍스처나 미세한 선과 같은 고주파수 정보(high-frequency information)를 모델링하는 후기 단계에서는 이미지 전반에 걸쳐 보다 균일한 전문성이 요구되기 때문이라는 것입니다.18

그림 2: 전문가 전문화 히트맵 (개념도). 이 시각화는 DiT-MoE에서 관찰된 전문가 전문화 현상을 개념적으로 보여줍니다. X축은 노이즈 제거 타임스텝(T에서 0으로 진행), Y축은 개별 전문가를 나타냅니다. 셀의 색상(히트맵)은 해당 타임스텝에서 각 전문가의 활성화 빈도를 의미합니다. 그림에서 볼 수 있듯이, 특정 전문가 그룹(예: Expert 1, 2)은 노이즈가 많은 초기 단계에 집중적으로 활성화되는 반면, 다른 그룹(예: Expert N-1, N)은 중간 단계에 특화되어 있습니다. 후기 단계로 갈수록 활성화 패턴이 더 균일해지는 경향을 보입니다. 이는 전문가들이 의미론적 콘텐츠가 아닌, 생성 과정의 특정 단계(예: '초기 구조 형성 전문가', '후기 디테일 묘사 전문가')에 따라 전문화됨을 시각적으로 보여줍니다. (참고: 18의 분석 결과를 바탕으로 개념적으로 제작)

이러한 분석은 DiT-MoE가 단순히 모델을 확장한 것을 넘어, 생성 알고리즘 자체를 학습을 통해 모듈화(modularize)했음을 시사합니다. 각 전문가는 노이즈 제거라는 복잡한 작업의 특정 하위 루틴(subroutine)을 학습한 것으로 볼 수 있으며, 이는 향후 더 해석 가능하고 구조화된 생성 모델을 향한 중요한 진일보라 할 수 있습니다.

## 3.  성능, 효율성, 그리고 스케일링의 최전선

DiT-MoE의 아키텍처적 우수성은 결국 정량적인 성능 지표를 통해 입증되어야 합니다. 이 장에서는 DiT-MoE가 기존의 밀집(dense) DiT 모델과 비교하여 성능과 계산 효율성 측면에서 어떤 성과를 거두었는지 상세히 분석합니다. 특히, 165억 파라미터라는 전례 없는 규모로 모델을 확장하며 MoE 가설을 검증한 과정과 그 결과가 가지는 의미를 집중적으로 조명합니다.

### 3.1  비교 벤치마크: 성능과 연산량 측면에서의 DiT-MoE 대 밀집 DiT

DiT-MoE의 핵심 가치는 '더 적은 연산으로 더 좋거나 동등한 성능'을 달성하는 데 있습니다. 이를 명확히 보여주기 위해, 기존의 밀집 DiT 모델과 DiT-MoE 모델의 주요 지표를 비교한 결과는 다음과 같습니다.

| 모델명                | 총 파라미터 | 활성 파라미터 | 추론 GFLOPs (256x256) | FID-50K (256x256) |
| --------------------- | ----------- | ------------- | --------------------- | ----------------- |
| DiT-B/2               | 232M        | 232M          | 30                    | -                 |
| DiT-L/2               | 458M        | 458M          | 58                    | -                 |
| DiT-XL/2 (Dense)      | 675M        | 675M          | 119                   | 2.27              |
| **DiT-MoE-S/2-8E2A**  | **447M**    | **150M**      | **38**                | **-**             |
| **DiT-MoE-B/2-8E2A**  | **833M**    | **268M**      | **67**                | **-**             |
| **DiT-MoE-L/2-8E2A**  | **1.6B**    | **520M**      | **130**               | **-**             |
| **DiT-MoE-XL/2-8E2A** | **2.6B**    | **815M**      | **204**               | **1.99**          |
| **DiT-MoE-G/2-16E2A** | **16.5B**   | **3.1B**      | **-**                 | **-**             |

**표 1: DiT-MoE 대 밀집 DiT 성능 비교.** 이 표는 밀집 DiT 모델과 다양한 크기의 DiT-MoE 모델 간의 파라미터 수, 추론 연산량(GFLOPs), 그리고 이미지 생성 품질(FID-50K)을 비교합니다. 여기서 'XL/2-8E2A'는 XL 크기, 패치 크기 2, 총 8개의 전문가 중 2개를 활성화(Activated)함을 의미합니다. 이 표의 핵심은 DiT-MoE 모델이 훨씬 적은 활성 파라미터와 GFLOPs로 밀집 모델에 필적하거나 이를 능가하는 성능을 달성할 수 있음을 보여줍니다. 예를 들어, DiT-MoE-L/2 모델은 DiT-XL/2보다 총 파라미터는 훨씬 많지만, 실제 추론 연산량(130 GFLOPs)은 거의 비슷하면서도 더 나은 성능을 기대할 수 있습니다. 이는 고정된 계산 예산 내에서 희소 모델이 더 효율적인 선택임을 정량적으로 입증합니다. (참고: 6의 데이터를 종합하여 재구성. 일부 값은 논문에서 직접 제공되지 않아 추정 또는 생략됨.)

분석 결과는 명확합니다. DiT-MoE 모델은 비슷한 추론 연산량(GFLOPs)을 가진 밀집 DiT 모델보다 일관되게 우수한 성능(더 낮은 FID 점수)을 보입니다.13 예를 들어, DiT-MoE-S 모델은 밀집 DiT-B 모델과 비슷한 계산 비용을 가지면서도 훨씬 더 나은 FID 점수를 달성할 수 있습니다.13 이는 동일한 계산 예산이 주어졌을 때, 더 작은 밀집 모델을 사용하는 것보다 더 큰 희소 모델을 사용하는 것이 성능 면에서 훨씬 효율적이라는 희소 모델의 근본적인 가설을 강력하게 뒷받침합니다. 즉, 조건부로 활성화되는 파라미터를 통해 모델 용량을 늘리는 것이, 모든 파라미터가 항상 활성화되는 밀집 모델의 크기를 키우는 것보다 더 나은 성능 향상 경로라는 것입니다.

### 3.2  165억 파라미터로의 확장: MoE 가설의 대규모 검증

DiT-MoE 연구의 백미는 165억 개의 파라미터를 가진 거대 모델을 성공적으로 훈련시킨 것입니다.14 이 모델은 총 파라미터 수는 방대하지만, 추론 시에는 단 31억 개의 파라미터만 활성화됩니다. 이 거대 모델은 ImageNet 512x512 해상도 벤치마크에서 1.80이라는 새로운 SOTA(State-of-the-Art) FID 점수를 기록하며, 희소 아키텍처의 잠재력을 명백히 입증했습니다.18

여기서 주목할 만한 점은, 이 가장 큰 모델이 ImageNet 데이터셋과 함께 **합성된 이미지 데이터(synthesized image data)**를 사용하여 훈련되었다는 사실입니다.14 이는 이 정도 규모의 모델을 효과적으로 훈련시키기 위해서는 기존의 실제 데이터셋만으로는 부족하며, 데이터의 양과 다양성을 확보하는 것 자체가 중요한 도전 과제임을 시사합니다.

이 165억 파라미터 모델의 성공은 희소 확산 모델 접근법 전체에 대한 강력한 개념 증명(proof-of-concept)입니다. 이는 MoE 아키텍처가 이전의 밀집 생성 모델을 훨씬 뛰어넘는 규모에서도 안정적으로 훈련되고 효과적으로 작동할 수 있음을 보여주며, 생성 모델 스케일링의 새로운 가능성을 열었습니다.

### 3.3  추론의 이점: 희소성이 가져오는 계산 비용 절감의 정량화

MoE 모델의 훈련은 복잡하지만, 그 진정한 보상은 추론(inference) 단계에서 실현됩니다. DiT-MoE 모델은 가장 큰 밀집 모델과 동등한 성능을 달성하면서도, 계산량은 절반 수준으로 줄일 수 있습니다.13

이러한 특성은 추론 시 유연한 트레이드오프를 가능하게 합니다. 사용자는 동일한 훈련된 모델 내에서, 더 빠른 속도와 낮은 비용으로 이미지를 생성하고 싶을 때는 더 적은 수의 전문가를 활성화하고, 더 높은 품질의 결과물이 필요할 때는 더 많은 전문가를 활성화하는 식으로 조절할 수 있습니다.

이러한 효율성은 실질적인 파급 효과를 가집니다. SOTA 품질의 생성 모델을 배포하고 운영하는 데 드는 경제적 부담을 크게 줄여줍니다. 이미지 한 장을 생성하는 데 필요한 GPU 시간 비용이 감소함에 따라, 고성능 AI 생성 기술이 더 많은 연구자와 기업에게 접근 가능해지며, 이는 관련 산업의 혁신을 가속화하는 중요한 동력이 됩니다.23

## 4.  희소 확산 모델의 진화하는 생태계

DiT-MoE의 등장은 끝이 아니라 새로운 시작이었습니다. 이 선구적인 연구는 희소 확산 모델이라는 풍부한 연구 분야를 탄생시켰고, 이후 다양한 후속 연구들이 초기 아이디어를 더욱 발전시키고 있습니다. 이 장에서는 라우팅 전략의 진화, 동적 MoE 프레임워크의 등장, 그리고 3D 및 비디오 생성과 같은 새로운 영역으로의 확장을 통해 DiT-MoE가 어떻게 진화하고 있는지 살펴봅니다.

### 4.1  라우팅의 중대한 분기점: 토큰 선택(DiT-MoE) 대 전문가 선택(EC-DiT)

초기 DiT-MoE 이후 가장 중요한 아키텍처적 발전은 라우팅 메커니즘에서 일어났습니다. 기존의 토큰 선택 방식의 한계를 극복하기 위해 전문가 선택이라는 새로운 패러다임이 등장했습니다.

| 라우팅 전략                     | 핵심 로직                                   | 부하 분산 메커니즘                                  | 주요 과제                                                    | 성능 특성                                                    | 대표 모델                         |
| ------------------------------- | ------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------------- |
| **토큰 선택 (Token-Choice)**    | 각 토큰이 처리받을 상위 k개의 전문가를 선택 | 보조 균형 손실 함수(Auxiliary Balance Loss) 필요    | 부하 불균형, 훈련 불안정성, 비효율적 전문가 활용             | 구현이 비교적 간단하나, 최적화가 어려움. 불균형 해소를 위한 추가 비용 발생. | DiT-MoE 18, Switch Transformer 24 |
| **전문가 선택 (Expert-Choice)** | 각 전문가가 처리할 상위 k개의 토큰을 선택   | 구조적으로 완벽한 부하 분산 보장 (별도 손실 불필요) | 토큰 누락(Token Dropping) 가능성. 일부 토큰이 어떤 전문가에게도 선택받지 못할 수 있음. | 부하 분산이 완벽하여 훈련이 안정적이고 효율적. 확산 모델의 비자기회귀적 특성에 더 적합. | EC-DiT 15, V-MoE 24               |

**표 2: 라우팅 전략 비교: 토큰 선택 대 전문가 선택.** 이 표는 희소 모델의 핵심인 두 가지 주요 라우팅 전략을 비교 분석합니다. 토큰 선택은 각 토큰이 주도권을 갖는 반면, 전문가 선택은 각 전문가가 주도권을 갖습니다. 전문가 선택은 부하 분산 문제를 구조적으로 해결하여 DiT와 같은 비자기회귀(non-autoregressive) 모델에 더 적합한 것으로 평가받고 있으며, 이는 EC-DiT가 970억 파라미터까지 확장될 수 있었던 핵심 요인 중 하나입니다. (참고: 15의 정보를 종합하여 구성)

**토큰 선택 라우팅의 한계:** DiT-MoE에서 사용된 토큰 선택 방식은 보조 손실 함수를 사용함에도 불구하고 여전히 최적의 전문가 활용을 보장하기 어렵습니다.15 특히 모든 이미지 패치 토큰을 동시에 처리하는 확산 모델의 비자기회귀적 특성상, 특정 전문가에게 부하가 몰리는 현상이 발생하기 쉽습니다.

**전문가 선택(EC-DiT)의 대두:** 이러한 문제를 해결하기 위해 **EC-DiT(Expert-Choice DiT)**와 같은 모델이 등장했습니다.15 이 방식에서는 라우팅의 주체가 바뀝니다. 토큰이 전문가를 선택하는 대신, 각 전문가가 자신에게 가장 적합한 상위 k개의 토큰을 선택합니다.24

**전문가 선택의 장점:** 이 접근법은 각 전문가에게 할당되는 토큰의 수가 고정되므로, 구조적으로 완벽한 부하 분산을 보장하며 별도의 보조 손실 함수가 필요 없습니다.24 또한, 이는 '이질적 연산(heterogeneous computation)'을 가능하게 합니다. 예를 들어, 이미지의 전경이나 복잡한 디테일을 담은 중요한 패치 토큰은 여러 전문가에게 선택받을 수 있는 반면, 단순한 배경 토큰은 소수의 전문가에게만 선택되거나 아예 선택되지 않을 수 있습니다.15 이러한 효율성 덕분에 EC-DiT는 훈련 수렴 속도와 텍스트-이미지 정렬 성능을 개선했으며, 무려 970억 파라미터라는 엄청난 규모까지 확장될 수 있었습니다.15

### 4.2  정적 라우팅을 넘어서: 적응형 및 동적 MoE 프레임워크 (Diff-MoE, Race-DiT)

연구는 여기서 멈추지 않고, 더욱 동적이고 적응적인 MoE 프레임워크로 발전하고 있습니다.

- **Diff-MoE:** 이 모델은 MoE 프레임워크에 시간적, 공간적 적응성을 부여합니다.32 핵심 아이디어는 **전문가별 타임스텝 조건화(expert-specific timestep conditioning)**입니다. 각 전문가가 자신만의 고유한 타임스텝 임베딩을 받아, 노이즈 제거 과정의 특정 단계에 더욱 세밀하게 전문화될 수 있도록 하는 것입니다.32 이는 전문가들이 생성 과정의 '언제'에 더 민감하게 반응하도록 만듭니다.
- **Race-DiT:** 이 모델은 "전문가 경주(Expert Race)"라는 더 유연한 라우팅 전략을 도입합니다.34 이 접근법은 특히 노이즈가 많은 입력에 대해 모델의 얕은 레이어에서 효과적인 라우팅을 학습하기 어렵다는 문제를 해결하고자, 토큰과 전문가 간의 할당을 더욱 동적이고 경쟁적인 방식으로 만듭니다.

이러한 연구들의 공통적인 흐름은 확산 과정의 특정 맥락(타임스텝, 노이즈 수준, 공간적 복잡도 등)을 인지하고 이에 맞춰 라우팅을 동적으로 조절하는, 더욱 지능적인 MoE를 향하고 있음을 보여줍니다.

### 4.3  차원 확장: 3D 생성 작업을 위한 DiT-MoE

DiT-MoE의 원리는 2D 이미지를 넘어 3D 생성 분야로도 확장되고 있습니다. 3D 포인트 클라우드 생성은 데이터의 비정형성(unordered nature)과 차원 증가로 인한 계산 복잡도의 세제곱 증가 때문에 매우 어려운 과제입니다.35

- **FastDiT-3D:** 효율적인 3D 포인트 클라우드 생성을 목표로 하는 **FastDiT-3D** 모델은 마스킹된 노이즈 제거가 핵심 혁신이지만, **다중 카테고리 3D 생성(multi-category 3D generation)** 성능을 향상시키기 위해 MoE를 도입했습니다.37
- **3D에서의 MoE 역할:** 이 모델에서 MoE는 '의자', '비행기' 등 서로 다른 객체 카테고리가 각기 다른 전문가를 통해 고유한 확산 경로를 학습하도록 돕습니다. 이는 **그래디언트 충돌(gradient conflict)**을 완화하는 데 결정적인 역할을 합니다. 단일 모델이 여러 카테고리를 동시에 생성하려고 할 때, 한 카테고리를 생성하기 위한 학습 신호가 다른 카테고리의 학습 신호를 방해할 수 있는데, MoE는 이를 분리하여 해결합니다.37 이는 2D DiT-MoE에서 보았던 '절차적 전문화'와는 다른, '의미론적 카테고리'에 따른 전문화의 새로운 적용 사례를 보여줍니다.

### 4.4  아키텍처의 유산: Sora와 같은 기초 비디오 모델에 대한 DiT의 영향

DiT 아키텍처의 강력함과 확장성은 OpenAI의 텍스트-비디오 모델인 **Sora**의 기반이 됨으로써 그 가치를 입증했습니다.23 Sora는 DiT를 기반으로 한 확산 모델이며, 이는 DiT가 학문적 연구를 넘어 산업계의 최첨단 모델을 구축하는 핵심 기술로 자리 잡았음을 의미합니다.

- **시공간 패치(Spacetime Patches):** Sora는 DiT의 개념을 2D 이미지 패치에서 3D **시공간 패치(spacetime patches)**로 확장했습니다. 비디오를 공간(프레임의 너비와 높이)과 시간(프레임의 순서) 축을 따라 작은 블록으로 자르고, 이를 트랜스포머의 입력 토큰으로 사용합니다.40 이를 통해 모델은 비디오 데이터를 통일된 표현으로 처리할 수 있게 됩니다.
- **창발적 세계 시뮬레이션(Emergent World Simulation):** OpenAI는 Sora와 같이 대규모로 확장된 비디오 생성 모델이 데이터를 통해 직관적인 물리 법칙, 객체 영속성(object permanence), 장기적인 시간적 일관성 등을 학습하는 '세계 시뮬레이터(world simulators)' 역할을 한다고 주장합니다.23 이는 DiT 아키텍처가 시공간 잠재 공간 내에서 복잡하고 장기적인 의존성을 모델링하는 뛰어난 능력을 가졌기에 가능한 결과입니다.

결론적으로, DiT-MoE에서 시작된 희소 확산 모델의 아이디어는 라우팅 전략의 혁신, 동적 적응성 강화, 그리고 3D 및 비디오와 같은 새로운 모달리티로의 확장을 통해 빠르게 진화하고 있습니다. 이는 생성 모델의 미래가 더욱 크고, 효율적이며, 지능적인 방향으로 나아가고 있음을 명확히 보여줍니다.

## 5.  비판적 평가와 미래 방향

DiT-MoE와 그 후속 모델들은 생성형 AI의 확장성 문제를 해결하는 데 있어 기념비적인 성과를 이루었지만, 이 접근법 역시 고유한 한계와 도전 과제를 안고 있습니다. 이 마지막 장에서는 DiT-MoE 접근법에 대한 비판적 평가를 제공하고, 향후 연구를 이끌어갈 핵심적인 미해결 질문들을 조명하며, 대규모 생성 모델링의 미래에서 DiT-MoE가 차지할 역할에 대해 전망합니다.

### 5.1  DiT-MoE 접근법의 확인된 한계와 내재적 과제

DiT-MoE 패러다임은 강력하지만, 그 이면에는 해결해야 할 여러 기술적 난제가 존재합니다.

- **훈련 복잡성 및 불안정성:** MoE 모델은 동급의 밀집 모델에 비해 훈련시키기가 훨씬 더 까다롭습니다. 하이퍼파라미터 튜닝에 매우 민감하며, 부하 분산과 같은 문제가 훈련 불안정성으로 이어지기 쉽습니다.12 성공적인 훈련을 위해서는 상당한 전문 지식과 시행착오가 요구됩니다.
- **라우팅 오버헤드 및 비최적성:** 희소성을 가능하게 하는 라우팅 메커니즘 자체가 완벽하지 않습니다. 토큰 선택 라우팅은 보조 손실이 필요하고 여전히 불균형할 수 있으며 15, 전문가 선택 라우팅은 일부 토큰이 처리되지 않고 누락될 위험이 있습니다.24 라우팅 결정 자체가 학습된 프로세스이기 때문에, 항상 최적의 할당을 보장하지는 못하며, 이로 인한 성능 저하가 발생할 수 있습니다.42
- **하드웨어 및 구현의 어려움:** MoE를 효율적으로 구현하기 위해서는 전문가 병렬 처리와 장치 간 통신 오버헤드를 효과적으로 관리할 수 있는 특화된 소프트웨어 라이브러리(예: DeepSpeed-MoE)와 하드웨어 구성이 필요합니다.12 이는 소규모 연구 그룹에게는 진입 장벽으로 작용할 수 있습니다.
- **대규모 평가의 한계:** 여러 논문에서 지적하듯이, 100억 개 이상의 파라미터를 가진 초대형 모델을 대상으로 광범위한 훈련을 수행하고 포괄적인 절제 연구(ablation study)를 진행하는 것은 막대한 계산 비용 때문에 자원이 풍부한 연구소조차도 어렵습니다.32 이로 인해 희소 모델의 스케일링 법칙을 완전히 이해하고 최적의 아키텍처를 탐색하는 데 제약이 따릅니다.

### 5.2  미해결 연구 질문: 더 효율적이고 강건한 희소 생성 모델을 향한 길

앞서 논의된 한계들은 희소 생성 모델 분야의 미래 연구 방향을 제시합니다. 다음은 이 분야의 발전을 위해 해결해야 할 핵심적인 미해결 질문들입니다.

- **최적의 라우팅 전략은 무엇인가?** 토큰 선택, 전문가 선택, 그리고 더 나아가 '소프트' 라우팅이나 미분 가능한 라우팅과 같은 진보된 전략들 사이의 논쟁은 현재 진행형입니다.26 비자기회귀적인 확산 모델에 이론적으로 가장 적합한 라우팅 전략은 무엇이며, 어떻게 이를 증명할 수 있을까요?
- **이질적인 전문가 구성이 효과적인가?** 현재 MoE 모델들은 대부분 구조적으로 동일한 전문가(모두 같은 크기의 MLP)를 사용합니다. 복잡한 작업을 위한 더 큰 전문가와 간단한 작업을 위한 더 작은 전문가를 혼합하는 이질적인(heterogeneous) 전문가 구성이 성능을 향상시킬 수 있을까요?.20
- **동적 전문가 할당이 가능한가?** 모델이 토큰의 복잡도에 따라 동적으로 가변적인 수의 전문가를 할당하도록 학습할 수 있을까요? (예: DyDiT, ReMoE에서 탐구).34 이는 고정된 top-k 접근법을 넘어서는 중요한 발전이 될 것입니다.
- **전문화에 대한 깊은 이해:** 전문가가 어떻게, 그리고 왜 특정 방식으로 전문화되는지에 대한 초기 분석은 존재하지만 21, 이에 대한 더 깊은 이해가 필요합니다. 아키텍처 설계나 훈련 목표를 통해 의미론적, 스타일적, 절차적 전문화와 같이 더 유용한 형태의 전문화를 유도하거나 강제할 수 있을까요?.42
- **희소 모델을 위한 스케일링 법칙:** 밀집 DiT는 명확한 스케일링 법칙을 보여주었지만, 희소성이 도입되었을 때 이 법칙이 어떻게 변하는지는 아직 완전히 규명되지 않았습니다. 총 파라미터 수와 활성 파라미터 수의 비율이 성능에 어떤 영향을 미치는지에 대한 이론적, 경험적 연구가 시급합니다.32

### 5.3  결론: 대규모 생성 모델링의 미래에서 DiT-MoE의 역할

DiT-MoE는 확장 가능한 트랜스포머 아키텍처와 조건부 연산의 효율성을 성공적으로 융합한 생성 모델링의 중요한 이정표입니다. 이 패러다임은 수십억 파라미터 규모의 모델 훈련을 가능하게 함으로써 이미지 생성 분야의 SOTA를 경신했으며, 그 원리는 이제 Sora와 같은 세계 최고 수준의 비디오 모델에까지 이어지는 기초가 되었습니다.

생성형 AI의 미래는 모델을 얼마나 효율적으로 확장할 수 있느냐에 달려 있습니다. DiT-MoE와 같은 희소하고 조건부로 계산되는 아키텍처는 더 이상 틈새 연구 분야가 아니라, 이 미래를 만들어갈 핵심 기둥이 될 것입니다. 이는 더욱 강력하고 유능한 '세계 시뮬레이터'의 탄생을 가능하게 할 것입니다. 라우팅 전략, 전문가 설계, 동적 할당에 대한 지속적인 연구는 이 강력한 패러다임을 더욱 정교하게 다듬어 나갈 것이며, 인공지능이 현실 세계를 이해하고 창조하는 능력의 한계를 계속해서 확장해 나갈 것입니다.

#### **참고 자료**

1. Diffusion Transformer (DiT) Models: A Beginner's Guide - Encord, accessed July 19, 2025, https://encord.com/blog/diffusion-models-with-transformers/
2. Deep Dive into Scalable Diffusion Models with Transformers - GitHub, accessed July 19, 2025, https://github.com/neobundy/Deep-Dive-Into-AI-With-MLX-PyTorch/blob/master/deep-dives/018-diffusion-transformer/README.md
3. Deep Dive into Sora's Diffusion Transformer (DiT) by Hand ✍︎ | Towards Data Science, accessed July 19, 2025, [https://towardsdatascience.com/deep-dive-into-soras-diffusion-transformer-dit-by-hand-%EF%B8%8E-1e4d84ec865d/](https://towardsdatascience.com/deep-dive-into-soras-diffusion-transformer-dit-by-hand-︎-1e4d84ec865d/)
4. Exploring Diffusion Models and Transformers - MyScale, accessed July 19, 2025, https://myscale.com/blog/deep-dive-diffusion-models-transformers/
5. Diffusion Transformers (DiT): Architecture Overview - ApX Machine Learning, accessed July 19, 2025, https://apxml.com/courses/advanced-diffusion-architectures/chapter-3-transformer-diffusion-models/diffusion-transformers-dit
6. Scalable Diffusion Models with Transformers - William Peebles, accessed July 19, 2025, https://www.wpeebles.com/DiT.html
7. What Is Mixture of Experts (MoE)? How It Works, Use Cases & More | DataCamp, accessed July 19, 2025, https://www.datacamp.com/blog/mixture-of-experts-moe
8. What Is Mixture of Experts MoE in Machine Vision Systems - UnitX, accessed July 19, 2025, https://www.unitxlabs.com/resources/mixture-of-experts-moe-machine-vision-system-explained/
9. Mixture of experts - Wikipedia, accessed July 19, 2025, https://en.wikipedia.org/wiki/Mixture_of_experts
10. MoE (Mixture of Experts) for Dummies: A Beginner's Guide - Michiel Horstman, accessed July 19, 2025, https://michielh.medium.com/moe-mixture-of-experts-for-dummies-d1a7e14c1846
11. What is mixture of experts? | IBM, accessed July 19, 2025, https://www.ibm.com/think/topics/mixture-of-experts
12. Mixture of Experts LLMs: Key Concepts Explained - neptune.ai, accessed July 19, 2025, https://neptune.ai/blog/mixture-of-experts-llms
13. Scaling Diffusion Transformers to 16 Billion Parameters - arXiv, accessed July 19, 2025, https://arxiv.org/html/2407.11633v1
14. Scaling Diffusion Transformers to 16 Billion Parameters - arXiv, accessed July 19, 2025, https://arxiv.org/html/2407.11633v3
15. EC-DiT: Scaling Diffusion Transformers with Adaptive Expert-Choice Routing - arXiv, accessed July 19, 2025, https://arxiv.org/html/2410.02098v2
16. Meta Superintelligence – Leadership Compute, Talent, and Data - SemiAnalysis, accessed July 19, 2025, https://semianalysis.com/2025/07/11/meta-superintelligence-leadership-compute-talent-and-data/
17. Mixture-of-Experts in the Era of LLMs A New Odyssey, accessed July 19, 2025, https://icml.cc/media/icml-2024/Slides/35222_1r94S59.pdf
18. Scaling Diffusion Transformers to 16 Billion Parameters | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/382301970_Scaling_Diffusion_Transformers_to_16_Billion_Parameters
19. [2407.11633] Scaling Diffusion Transformers to 16 Billion Parameters - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2407.11633
20. [Literature Review] Scaling Diffusion Transformers to 16 Billion Parameters, accessed July 19, 2025, https://www.themoonlight.io/en/review/scaling-diffusion-transformers-to-16-billion-parameters
21. Scaling Diffusion Transformers to 16 Billion Parameters - Paper Detail, accessed July 19, 2025, https://deeplearn.org/arxiv/525187/scaling-diffusion-transformers-to-16-billion-parameters
22. Paper page - Scaling Diffusion Transformers to 16 Billion Parameters - Hugging Face, accessed July 19, 2025, https://huggingface.co/papers/2407.11633
23. Under The Hood: How OpenAI's Sora Model Works - Factorial Funds, accessed July 19, 2025, https://www.factorialfunds.com/blog/under-the-hood-how-openai-s-sora-model-works
24. Mixture-of-Experts with Expert Choice Routing - Google Research, accessed July 19, 2025, https://research.google/blog/mixture-of-experts-with-expert-choice-routing/
25. EC-DIT: Scaling Diffusion Transformers with Adaptive Expert-Choice Routing - OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=PxlfzEePC0
26. Routers in Vision Mixture of Experts: An Empirical Study - arXiv, accessed July 19, 2025, https://arxiv.org/html/2401.15969v2
27. Routers in Vision Mixture of Experts: An Empirical Study - arXiv, accessed July 19, 2025, https://arxiv.org/pdf/2401.15969
28. Mixture of Experts vs Mixture of Tokens: Making LLMs more efficient | SuperAnnotate, accessed July 19, 2025, https://www.superannotate.com/blog/mixture-of-experts-vs-mixture-of-tokens
29. EC-DiT: Scaling Diffusion Transformers with Adaptive Expert-Choice Routing - arXiv, accessed July 19, 2025, https://arxiv.org/html/2410.02098v5
30. At the Frontier of AI: Reviewing Top Papers on Mixture of Experts in Machine Learning - Part 5 - Isaac Kargar, accessed July 19, 2025, https://kargarisaac.medium.com/at-the-frontier-of-ai-reviewing-top-papers-on-mixture-of-experts-in-machine-learning-part-5-ee939dc91409
31. EC-DIT: Scaling Diffusion Transformers with Adaptive Expert-Choice Routing, accessed July 19, 2025, https://machinelearning.apple.com/research/ec-dit
32. Diff-MoE: Diffusion Transformer with Time-Aware and Space-Adaptive Experts, accessed July 19, 2025, [https://openreview.net/forum?id=JCUsWrwkKw¬eId=ePj1dx5LWg](https://openreview.net/forum?id=JCUsWrwkKw&noteId=ePj1dx5LWg)
33. Diff-MoE: Diffusion Transformer with Time-Aware and Space-Adaptive Experts - ICML 2025, accessed July 19, 2025, https://icml.cc/virtual/2025/poster/45706
34. ICML Poster Expert Race: A Flexible Routing Strategy for Scaling Diffusion Transformer with Mixture of Experts - ICML 2025, accessed July 19, 2025, https://icml.cc/virtual/2025/poster/46221
35. DiT-3D: Exploring Plain Diffusion Transformers for 3D Shape Generation, accessed July 19, 2025, https://proceedings.neurips.cc/paper_files/paper/2023/file/d6c01b025cad37d5c8bab4ba18846c02-Paper-Conference.pdf
36. [2307.01831] DiT-3D: Exploring Plain Diffusion Transformers for 3D Shape Generation - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2307.01831
37. Fast Training of Diffusion Transformer with Extreme Masking for 3D ..., accessed July 19, 2025, https://dit-3d.github.io/FastDiT-3D/
38. Video generation models as world simulators | OpenAI, accessed July 19, 2025, https://openai.com/index/video-generation-models-as-world-simulators/
39. Sora System Card - OpenAI, accessed July 19, 2025, https://openai.com/index/sora-system-card/
40. OpenAI Sora's Technical Review - Jianing Qi, accessed July 19, 2025, https://j-qi.medium.com/openai-soras-technical-review-a8f85b44cb7f
41. Diffusion Models for Video Generation | Lil'Log, accessed July 19, 2025, https://lilianweng.github.io/posts/2024-04-12-diffusion-video/
42. A Limitations and Future Work, accessed July 19, 2025, https://proceedings.neurips.cc/paper_files/paper/2022/file/b8d1d741f137d9b6ac4f3c1683791e4a-Supplemental-Conference.pdf
43. Official implementation for our paper "Scaling Diffusion Transformers Efficiently via μP". - GitHub, accessed July 19, 2025, https://github.com/ML-GSAI/Scaling-Diffusion-Transformers-muP






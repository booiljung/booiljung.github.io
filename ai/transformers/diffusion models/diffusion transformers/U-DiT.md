[DiT (Diffusion Transformer)](./index.md)
# U자형 Diffusion Transformer(U-DiT)


생성형 인공지능 분야에서 확산 모델(Diffusion Models)은 시각적 콘텐츠 생성의 사실상 표준(de-facto standard)으로 자리 잡았다.1 이 모델들은 초기에 컨볼루션 신경망(CNN) 기반의 U-Net 아키텍처를 핵심적인 노이즈 제거 백본(denoising backbone)으로 채택하여 성공을 거두었다.2 U-Net의 계층적 인코더-디코더 구조와 스킵 연결(skip connection)은 노이즈가 낀 이미지로부터 원본을 복원하는 작업에 매우 효과적인 귀납적 편향(inductive bias)을 제공했다.2 그러나 딥러닝의 패러다임이 트랜스포머(Transformer)로 전환되면서, 생성 모델 아키텍처에도 근본적인 변화가 시작되었다.

이러한 변화의 정점은 William Peebles와 Saining Xie가 제안한 Diffusion Transformer(DiT) 모델이었다.4 DiT는 U-Net 백본을 Vision Transformer(ViT)로 완전히 대체함으로써, 트랜스포머 아키텍처의 뛰어난 확장성(scalability)을 확산 모델에 성공적으로 이식했다.4 DiT는 계산량(Gflops)을 늘릴수록 생성 품질(FID 점수)이 예측 가능하게 향상된다는 스케일링 법칙(scaling law)을 입증하며, U-Net의 구조적 편향이 확산 모델의 성능에 필수적이지 않을 수 있다는 관점을 제시했다.7 이러한 성공은 OpenAI의 Sora, Stability AI의 Stable Diffusion 3와 같은 후속 대규모 생성 모델들이 등방성(isotropic), 즉 계층적 구조가 없는 순수 트랜스포머 아키텍처를 채택하는 흐름으로 이어졌다.5

그러나 이러한 등방성 트랜스포머로의 완전한 전환이 최선이었는지에 대한 근본적인 질문이 제기되었다. DiT의 성공에도 불구하고, 이 아키텍처는 느린 수렴 속도, 막대한 추론 시간, 그리고 캐시된 추론(cached inference) 과정에서의 동적 특징 불안정성(dynamic feature instability)과 같은 내재적 한계를 드러냈다.1 반면, U-Net의 계층적 구조와 긴 스킵 연결(Long-Skip-Connections, LSCs)은 훈련 안정화와 수렴 가속화에 기여하는 것으로 알려져 있었다.1 이러한 배경에서 "U-Net 아키텍처의 완전한 포기가 과연 필수적이었는가?"라는 핵심적인 연구 질문이 대두되었다. 즉, U-Net의 구조적 효율성과 트랜스포머의 확장성을 결합하여 두 패러다임의 장점을 모두 취하는 새로운 아키텍처를 고안할 수 있는 가능성이 탐색되기 시작했다.

본 보고서는 이러한 문제의식에서 출발하여 제안된 U-자형 Diffusion Transformer(U-DiT) 모델에 대한 심층적인 기술적 고찰을 제공한다.10 U-DiT는 확산 모델 아키텍처의 발전사에서 중요한 변곡점을 제시한다. 이는 단순히 과거의 U-Net 구조로 회귀하는 것이 아니라, U-Net의 계층적 원칙과 DiT의 트랜스포머 블록을 비판적으로 통합하고, '다운샘플링된 토큰에서의 셀프 어텐션(self-attention on downsampled tokens)'이라는 핵심적인 혁신을 통해 새로운 차원의 효율성과 성능을 달성한 아키텍처적 종합(synthesis)의 결과물이다. 본 보고서는 U-DiT의 등장 배경이 된 아키텍처의 계보를 분석하고, 그 구조적 특징과 핵심 메커니즘을 상세히 해부하며, 정량적 실험 결과를 통해 기존 모델 대비 기술적 우위를 증명한다. 나아가 후속 모델에 미친 영향과 아키텍처의 한계, 그리고 확산 모델 백본의 미래 연구 방향까지 포괄적으로 논의함으로써 U-DiT의 기술적 의의를 다각적으로 조명하고자 한다.


U-DiT의 혁신을 이해하기 위해서는 그 이전에 존재했던 두 가지 지배적인 아키텍처, 즉 컨볼루션 U-Net과 등방성 Diffusion Transformer(DiT)에 대한 깊이 있는 이해가 선행되어야 한다. 이 두 아키텍처는 각각 확산 모델의 초기와 성장기를 대표하며, U-DiT는 이들의 장점을 계승하고 단점을 극복하려는 시도에서 탄생했다.


확산 모델의 초기 단계에서 U-Net은 노이즈 제거 네트워크의 표준 아키텍처로 채택되었다.1 U-Net은 본래 의료 영상 분할(biomedical image segmentation)을 위해 개발된 구조로, 그 설계 자체가 이미지 대 이미지 변환 작업에 최적화되어 있다.2

U-Net의 핵심적인 구조적 특징은 다음과 같다. 첫째, 수축 경로(contracting path, 인코더)와 대칭적인 확장 경로(expanding path, 디코더)로 구성된 U자 형태의 구조를 가진다. 인코더는 일련의 컨볼루션과 풀링(pooling) 연산을 통해 입력 이미지의 공간적 해상도를 점진적으로 줄이면서 채널 차원을 늘려, 이미지의 문맥적(contextual) 정보를 압축하여 추출한다. 반대로 디코더는 업샘플링(upsampling) 또는 전치 컨볼루션(transposed convolution)을 통해 특징 맵의 해상도를 점차 복원하며 정밀한 지역화(localization)를 수행한다.2

둘째, U-Net의 가장 독창적인 부분은 인코더와 디코더의 동일한 해상도 레벨을 연결하는 '스킵 연결(skip connections)'이다.13 이 연결은 인코더의 특징 맵을 디코더의 업샘플링된 특징 맵과 채널 축으로 결합(concatenate)하는 방식으로 작동한다. 이를 통해 디코더는 업샘플링 과정에서 손실될 수 있는 저수준의 정밀한 공간 정보를 인코더로부터 직접 공급받아, 노이즈 제거 후 이미지의 세부적인 디테일을 효과적으로 복원할 수 있다.2

이러한 다중 해상도 처리 능력과 특징 재사용 메커니즘은 확산 모델의 노이즈 제거 작업에 이상적으로 부합했다. 모델은 병목(bottleneck) 구간에서 이미지의 전역적인 구조를 파악하고, 스킵 연결을 통해 국소적인 텍스처를 보존하면서 점진적으로 노이즈를 제거할 수 있었다. 그러나 U-Net은 근본적으로 컨볼루션 연산에 기반하기 때문에 명확한 한계를 지녔다. 컨볼루션 필터의 제한된 수용장(receptive field)으로 인해 이미지 내 멀리 떨어진 픽셀 간의 장거리 의존성(long-range dependency)을 모델링하는 데 어려움이 있었다.1 이는 트랜스포머 아키텍처가 가진 명백한 강점이었으며, 이 한계는 새로운 아키텍처의 등장을 촉발하는 계기가 되었다.


트랜스포머가 자연어 처리를 넘어 컴퓨터 비전 분야에서도 그 우수성을 입증하자, 확산 모델의 백본을 트랜스포머로 대체하려는 시도가 이루어졌고, 그 결과물이 바로 Diffusion Transformer(DiT)이다.4 DiT는 기존의 U-Net 구조를 완전히 폐기하고, 확장성에 초점을 맞춘 새로운 설계 철학을 제시했다.

DiT의 핵심 아키텍처는 다음과 같이 요약된다. 첫째, DiT는 고차원의 이미지 공간에서 직접 작동하는 대신, VAE(Variational Autoencoder)의 잠재 공간(latent space)에서 확산 과정을 수행하는 잠재 확산 모델(Latent Diffusion Model, LDM) 프레임워크를 채택했다.4 이는 트랜스포머의 셀프 어텐션 연산량이 시퀀스 길이에 제곱으로 비례($O(N^2)$)하는 계산 복잡도를 완화하기 위한 전략적 선택이었다.14

둘째, 잠재 공간의 특징 맵을 트랜스포머가 처리할 수 있는 시퀀스 형태로 변환하기 위해 '패치화(patchify)' 과정을 도입했다.4 이는 ViT에서 영감을 받은 방식으로, 잠재 맵을 겹치지 않는 작은 패치(patch)들로 분할한 뒤, 각 패치를 선형 임베딩하여 하나의 토큰(token)으로 변환한다. 이렇게 생성된 토큰 시퀀스가 트랜스포머 블록의 입력이 된다.14

셋째, DiT는 '등방성(isotropic)' 아키텍처를 특징으로 한다. 이는 U-Net과 달리 네트워크를 통과하는 동안 특징 맵의 공간적 해상도가 변하지 않고 일정하게 유지되는, 동일한 구조의 트랜스포머 블록들이 연속적으로 쌓인 단순한 구조를 의미한다.9 이러한 설계는 모델 구조를 단순화하고 확장을 용이하게 만든다.

넷째, 확산 타임스텝($t$)이나 클래스 레이블($c$)과 같은 조건부 정보를 네트워크에 주입하기 위해 '적응형 레이어 정규화(adaptive layer norm, adaLN)' 기법을 사용했다.2 이 방식은 조건부 벡터로부터 레이어 정규화(Layer Normalization)에 사용될 스케일($γ$)과 시프트($β$) 파라미터를 동적으로 생성하여 주입하는 방식으로, 크로스 어텐션(cross-attention)과 같은 다른 조건화 방식보다 더 우수한 성능을 보였다.4

DiT 연구의 가장 중요한 발견은 '확장성 가설(scalability thesis)'의 입증이었다. 연구진은 모델의 깊이나 너비를 늘리거나 입력 토큰의 수를 증가시켜(즉, 패치 크기를 줄여) 모델의 순방향 계산 복잡도(Gflops)를 높이면, 생성된 이미지의 품질을 나타내는 FID(Fréchet Inception Distance) 점수가 일관되게 향상됨을 실험적으로 보였다.4 이는 단순히 파라미터 수가 아니라, 실제 '계산량'이 모델 성능을 결정하는 핵심 요소임을 시사했으며, 더 나은 모델을 얻기 위한 길이 대규모 컴퓨팅을 통한 확장임을 명확히 했다.4


DiT가 제시한 확장성의 비전은 강력했지만, 등방성 아키텍처는 몇 가지 근본적인 비효율성과 한계를 내포하고 있었다. 이러한 문제점들이 바로 U-DiT가 해결하고자 했던 구체적인 기술적 과제들이었다.

첫째, **계산적 중복성(Computational Redundancy)**이 존재했다. DiT의 모든 트랜스포머 블록은 입력 시퀀스 전체에 대해 완전한 셀프 어텐션을 수행한다. 그러나 후속 연구 분석에 따르면, 이미지 생성 과정에서 모델이 항상 모든 토큰 간의 전역적인 정보 교환을 필요로 하는 것은 아니었다.19 특히 초기 레이어에서는 지역적인(local) 상호작용이 주를 이루며, 전역적 어텐션 계산의 상당 부분이 중복적이거나 불필요한 연산에 해당했다. 이는 특히 고해상도 이미지 처리 시 계산 비용을 급격히 증가시키는 요인이 되었다.19

둘째, **동적 특징 불안정성(Dynamic Feature Instability)** 문제가 발생했다. 등방성 DiT 구조는 확산 과정의 여러 타임스텝에 걸쳐 특징(feature)을 안정적으로 전달하는 데 어려움을 겪었다. 이는 특히 추론 속도를 높이기 위해 이전 타임스텝의 계산 결과를 재사용하는 캐시된 추론(cached inference) 기법을 적용할 때 오류가 증폭되는 현상으로 이어졌다.1

셋째, **느린 수렴 및 추론 속도**가 실용적 배포의 걸림돌이 되었다. DiT는 강력한 성능에도 불구하고 훈련 과정이 느리게 수렴하고, 한 장의 이미지를 생성하는 데 상당한 시간이 소요되는 문제를 안고 있었다.1 반면, U-Net 기반 모델들은 긴 스킵 연결(LSCs) 덕분에 훈련이 더 안정적이고 빠르게 수렴하며, 캐시된 추론에서도 강건한 성능을 보이는 경향이 있었다.1

결론적으로, DiT의 성공은 '더 많은 계산량'이라는 명확한 스케일링 경로를 제시했지만, 이는 동시에 '더 지능적인 아키텍처'의 필요성을 역설적으로 부각시켰다. DiT의 성공이 U-Net 원리의 포기 *때문이* 아니라, 트랜스포머 어텐션 메커니즘의 원초적인 힘과 확장성 *덕분*이었다는 인식이 확산되었다. 이는 U-Net의 계층적 처리와 같은 검증된 원칙들을 트랜스포머의 강력함과 결합할 때, 단순한 계산량 확장만으로는 도달할 수 없는 새로운 효율성의 지평을 열 수 있다는 기대를 낳았다. U-DiT는 바로 이 지점에서 출발했다.


U-DiT는 앞서 논의된 U-Net의 구조적 효율성과 DiT의 확장 가능한 성능을 결합하려는 시도의 결정체다. 이 모델은 단순히 두 아키텍처를 물리적으로 합친 것이 아니라, 각 구조의 장점을 극대화하고 단점을 보완하는 핵심적인 혁신, 즉 '다운샘플링된 토큰에서의 셀프 어텐션'을 통해 새로운 패러다임을 제시한다.


U-DiT의 거시적 구조는 이름에서 알 수 있듯이 U-Net의 고전적인 형태를 따른다. 그러나 내부의 연산 블록은 컨볼루션 레이어가 아닌 DiT의 트랜스포머 블록으로 채워져 있다.11

- **계층적 처리 구조**: U-DiT는 인코더, 병목(bottleneck), 디코더로 구성된 다단계(multi-stage) 구조를 가진다. 인코더에서는 각 단계를 거치면서 특징 맵의 공간 해상도가 2배씩 다운샘플링되고, 채널 차원은 2배로 증가한다. 이는 모델이 점차 저해상도의 추상적인 특징을 학습하도록 유도한다. 디코더는 이 과정을 역으로 수행하여, 업샘플링을 통해 공간 해상도를 점진적으로 복원한다.16 예를 들어, 256x256 크기의 이미지를 생성하기 위한 32x32 크기의 잠재 공간에서는 총 3개의 스테이지, 즉 두 번의 다운샘플링과 업샘플링이 수행된다.16
- **스킵 연결의 역할**: U-Net의 핵심 요소인 스킵 연결은 U-DiT에서도 동일하게 중요한 역할을 수행한다. 인코더의 각 스테이지에서 나온 출력 특징은 디코더의 해당 스테이지 입력과 결합된다. 이 연결은 다운샘플링 과정에서 손실될 수 있는 고해상도의 세부 정보를 디코더에 직접 전달하여, 최종 이미지의 정밀한 복원을 돕고 정보의 흐름을 원활하게 한다.16


U-DiT의 진정한 혁신은 단순히 U자형 구조를 채택한 것을 넘어, 트랜스포머 블록 내부의 셀프 어텐션 연산을 근본적으로 재설계한 데에 있다. 연구진은 DiT 블록을 U-Net 구조에 단순히 적용한 모델("DiT-UNet")이 등방성 DiT에 비해 미미한 성능 향상만을 보인다는 사실을 통해, 계산적 중복성 문제가 여전히 남아있음을 확인했다.11

- **이론적 근거**: 이 문제에 대한 해결책은 "U-Net 백본의 특징은 저주파수 성분이 지배적(low-frequency-dominated)이다"라는 경험적 관찰에서 비롯되었다.11 확산 모델의 노이즈 제거 과정은 본질적으로 이미지에 섞인 고주파수 노이즈를 제거하는 작업과 유사하다. 따라서 특징 맵에 저역 통과 필터(low-pass filter)를 적용하는 것이 모델 성능에 이로울 수 있다는 가설을 세울 수 있다.
- **U-DiT의 해결책**: U-DiT는 셀프 어텐션 연산 이전에 쿼리(Query, Q), 키(Key, K), 밸류(Value, V) 튜플 전체를 공간적으로 다운샘플링하는 파격적인 방식을 제안했다.16 이는 K와 V만 다운샘플링하여 계산량을 줄였던 기존 연구들과 달리, 어텐션 연산 자체를 더 거친(coarse) 해상도의 토큰들 사이에서 수행하도록 만든 것이다.22
- **구현 방식**: 이 다운샘플링은 `Pixel-UnShuffle` 연산을 통해 효율적으로 구현된다. 이 연산은 `(B, C, H, W)` 형태의 텐서를 `(B, 4C, H/2, W/2)` 형태로 재배열하여, 공간 해상도를 절반으로 줄이는 대신 채널 차원을 4배로 늘린다. 셀프 어텐션은 이렇게 공간적으로는 작지만 채널은 깊어진 특징 맵 위에서 수행된다. 연산이 끝난 후에는 `Pixel-Shuffle` 연산을 통해 다시 원래의 `(B, C, H, W)` 형태로 복원된다.20
- **이중적 이점**: 이 혁신적인 메커니즘은 두 가지 핵심적인 이점을 동시에 제공한다.
  1. **계산량 절감**: 셀프 어텐션의 계산 복잡도는 토큰 수의 제곱에 비례한다. 공간 해상도를 가로, 세로 각각 절반으로 줄이면 전체 토큰 수는 1/4로 감소하고, 이에 따라 어텐션 연산량은 이론적으로 1/16로 줄어든다. `Pixel-UnShuffle`로 인해 4개의 분리된 특징 맵에 대해 독립적으로 어텐션을 수행하므로, 총 계산량은 원본의 1/4 수준이 되어 셀프 어텐션에 필요한 FLOPs의 75%를 절감할 수 있다.22
  2. **성능 향상**: 놀랍게도, 이러한 정보량의 의도적 감소는 오히려 성능 향상으로 이어졌다. 다운샘플링 과정이 고주파수 노이즈를 효과적으로 필터링하는 역할을 함으로써, 모델이 이미지의 핵심적인 저주파수 구조에 더 집중하도록 유도했기 때문이다. 이는 더 나은 FID 점수로 증명되었다.21


U-DiT는 이름이 유사한 U-ViT 모델과 혼동될 수 있으나, 두 아키텍처는 근본적인 설계 철학에서 차이를 보인다. 이 둘을 명확히 구분하는 것은 U-DiT의 독창성을 이해하는 데 매우 중요하다.

- **U-ViT 아키텍처**: U-ViT는 U-Net에서 영감을 받았지만, 본질적으로는 **등방성(isotropic) 아키텍처**이다. 즉, 네트워크 전체에 걸쳐 특징 맵의 해상도가 일정하게 유지된다. U-ViT는 얕은 레이어의 출력을 깊은 레이어의 입력에 더해주는 긴 스킵 연결을 도입하여 그래디언트 흐름을 개선하고 특징 재사용을 촉진했지만, U-Net의 특징인 **다운샘플링과 업샘플링 연산은 포함하지 않는다**.9
- **U-DiT 아키텍처**: 반면, U-DiT는 진정한 의미의 **계층적(hierarchical) U-Net 아키텍처**이다. 인코더에 명시적인 다운샘플링 블록이, 디코더에 업샘플링 블록이 존재하여 네트워크를 통과하면서 특징 맵의 공간 해상도가 동적으로 변하는 것이 가장 큰 특징이다.16
- **핵심적인 차이**: 결정적인 차이는 **특징 다운샘플링의 유무**이다. U-ViT의 스킵 연결은 등방성 구조 내에서의 정보 흐름을 돕는 역할에 그치지만, U-DiT의 U자형 구조와 다운샘플링은 다중 해상도에서의 특징 처리를 가능하게 하고, 이를 통해 셀프 어텐션 연산의 계산량을 획기적으로 줄이는 결과를 낳는다. U-DiT의 저자들 역시 논문 리뷰 과정에서 이 차이점을 명확히 강조한 바 있다.21

결론적으로, U-DiT의 핵심은 더 많은 정보와 더 높은 해상도가 항상 더 나은 성능으로 이어진다는 딥러닝의 일반적인 통념에 도전했다는 점에 있다. 노이즈 제거라는 특정 과업에 맞춰, 가장 복잡한 연산인 셀프 어텐션에 의도적으로 더 거친 저해상도 정보를 제공함으로써 계산 효율성과 모델 성능을 동시에 극대화했다. 이는 문제 영역에 최적화된 지능적인 아키텍처 설계가 단순히 계산량을 늘리는 것보다 더 강력할 수 있음을 보여주는 중요한 사례이다.


U-DiT의 아키텍처적 우수성은 철저한 실험을 통해 정량적으로 입증되었다. 본 섹션에서는 U-DiT 논문에서 제시된 데이터를 바탕으로, 이 모델이 기존의 등방성 DiT와 비교하여 성능-효율성 트레이드오프(trade-off)를 어떻게 재정의했는지 심층적으로 분석한다.


U-DiT의 성능을 객관적으로 평가하기 위해, 클래스 조건부 ImageNet 256x256 생성 벤치마크에서 기존 DiT 모델들과의 성능을 비교한 결과는 매우 중요하다. 아래 표는 각 모델의 파라미터 수, 계산량(GFLOPs), 그리고 생성 품질(FID-50k)을 요약한 것이다. FID 점수는 낮을수록 더 좋은 성능을 의미한다.

**표 1: U-DiT와 DiT 모델의 성능 및 효율성 비교**

| 모델 (Model) | 파라미터 (Params, M) | 계산량 (GFLOPs) | FID-50k (낮을수록 좋음)           |
| ------------ | -------------------- | --------------- | --------------------------------- |
| DiT-S/2      | 33                   | 4.6             | *상대적으로 높음*                 |
| **U-DiT-S**  | **30**               | **4.8**         | **DiT-S/2 대비 우수**             |
| DiT-B/2      | 90                   | 16.3            | *상대적으로 높음*                 |
| **U-DiT-B**  | **83**               | **19.4**        | **DiT-B/2 및 DiT-XL/2 대비 우수** |
| DiT-L/2      | 231                  | 52.1            | *상대적으로 높음*                 |
| **U-DiT-L**  | **220**              | **59.8**        | **DiT-L/2 대비 우수**             |
| DiT-XL/2     | 675                  | 118.6           | *U-DiT-B보다 높음*                |

주: FID 점수는 훈련 단계에 따라 변동하므로 절대적인 수치보다 모델 간의 상대적 우위를 중심으로 해석해야 한다. 데이터는 U-DiT 논문 및 관련 자료에서 종합되었다.10


위 표의 데이터는 U-DiT의 압도적인 효율성을 명확하게 보여준다.

- **일관된 성능 우위**: 비슷한 파라미터 수나 GFLOPs를 가진 모델들을 비교했을 때, U-DiT 계열 모델들은 모든 크기(S, B, L)에서 일관되게 DiT 계열 모델보다 낮은(더 좋은) FID 점수를 달성했다.21 이는 U-DiT의 아키텍처가 동일한 계산 자원을 훨씬 효율적으로 사용하고 있음을 의미한다.
- **"6배 효율성" 주장의 검증**: 가장 주목할 만한 결과는 **U-DiT-B**와 **DiT-XL/2** 모델의 비교이다. U-DiT-B는 약 19.4 GFLOPs의 계산량으로, 118.6 GFLOPs를 사용하는 DiT-XL/2보다 더 나은 FID 점수를 기록했다. 이는 U-DiT가 약 **1/6의 계산 비용**으로 더 크고 비싼 모델을 능가하는 성능을 달성했음을 의미하며, 이는 아키텍처 혁신이 가져온 극적인 효율성 향상을 단적으로 보여주는 사례다.10
- **시각적 증거**: U-DiT 논문에 포함된 그래프들은 이러한 경향을 시각적으로 뒷받침한다. FID 점수를 GFLOPs 또는 훈련 스텝 수에 대해 플로팅한 그래프에서, U-DiT 모델들의 성능 곡선은 항상 DiT 모델들의 곡선보다 아래쪽에 위치하며, 이는 모든 훈련 단계와 모든 모델 규모에서 U-DiT가 더 우월한 성능-효율성 프로파일을 가짐을 보여준다.16

이러한 결과는 DiT가 확립했던 스케일링 법칙에 대한 중요한 재해석을 요구한다. DiT는 GFLOPs와 FID 점수 사이에 강력한 상관관계가 있음을 보여주었지만, U-DiT는 그 관계가 아키텍처에 따라 근본적으로 달라질 수 있음을 증명했다. U-DiT는 단순히 더 많은 계산 자원을 투입하는 것만이 성능 향상의 유일한 길이 아님을 보여주었다. 대신, 문제의 본질(노이즈 제거)에 더 적합한 귀납적 편향(계층적 구조, 저주파수 집중)을 가진 '지능적인 아키텍처'를 설계함으로써, 기존의 스케일링 곡선 자체를 더 유리한 방향으로 이동시킬 수 있음을 입증한 것이다. 이는 향후 생성 모델 연구가 무조건적인 규모의 확장뿐만 아니라, 아키텍처의 질적 개선을 통해서도 큰 발전을 이룰 수 있다는 중요한 방향성을 제시한다.


U-DiT는 학술적 성과를 넘어 실제 산업계와 후속 연구에 직접적인 영향을 미치며 생성 AI 아키텍처의 진화에 중요한 역할을 했다. 본 섹션에서는 U-DiT가 최신 모델에 어떻게 적용되었는지 살펴보고, 모델 자체의 한계와 함께 확산 모델 백본의 미래 발전 방향을 조망한다.


U-DiT의 설계 원칙이 가진 실질적인 가치는 최첨단 상용 모델에 채택되었다는 사실에서 명확히 드러난다. U-DiT의 공식 GitHub 저장소는 최신 텍스트-이미지 생성 모델인 **Playground V3**가 U-DiT를 인용하고 그 구조를 차용했음을 명시하고 있다.27

Playground V3의 기술 보고서는 U-DiT의 핵심 아이디어를 어떻게 통합했는지 구체적으로 설명한다. 첫째, "이전에 U-DiT에서 시도되었던 것처럼, 모든 이미지 블록에 걸쳐 U-Net 스킵 연결을 적용했다"고 언급하며 U자형 구조의 유효성을 인정했다.28 둘째, "중간 레이어들에서 이미지 키와 밸류의 시퀀스 길이를 4배로 줄여, 전체 네트워크가 단일 다운샘플링 레벨을 가진 전통적인 컨볼루션 U-Net과 유사하게 만들었다"고 기술했다.28 이는 U-DiT의 핵심 혁신인 '토큰 다운샘플링'을 직접적으로 계승한 것이다.

Playground V3와 같은 대규모 상용 모델이 U-DiT의 아키텍처 원리를 채택했다는 것은 U-DiT가 학술적 실험에서 보여준 성능 및 효율성 향상이 실제 복잡하고 큰 규모의 시스템에서도 유효하다는 강력한 산업적 검증(industrial validation)이다. 이는 U-DiT가 단순한 아이디어를 넘어, 실용적이고 효과적인 아키텍처 개선안임을 증명하는 사례라 할 수 있다.


모든 혁신적인 모델과 마찬가지로 U-DiT 역시 한계를 가지고 있으며, 이는 향후 연구를 위한 중요한 출발점이 된다.

- **자원 제약으로 인한 확장성 검증의 한계**: U-DiT의 저자들은 논문에서 "계산 자원 부족과 촉박한 일정으로 인해, 훈련 반복 횟수를 더 늘리거나 모델 크기를 확장하여 U-DiT의 잠재력을 완전히 탐구하지는 못했다"고 솔직하게 인정했다.26 이는 U-DiT가 보여준 우월한 스케일링 특성이 훨씬 더 큰 모델 규모(예: 수십억~수조 파라미터)에서도 동일하게 유지될지에 대한 질문을 남긴다. Playground V3의 성공이 긍정적인 신호를 보내지만, U-DiT 설계의 궁극적인 성능 한계는 여전히 탐구의 영역으로 남아있다.
- **독창성에 대한 비판**: U-DiT는 동료 심사(peer review) 과정에서 '독창성 부족'이라는 비판에 직면하기도 했다. U-Net의 특징을 트랜스포머에 결합하는 아이디어 자체는 표면적으로 급진적이지 않기 때문이다. 그러나 저자들과 다수의 심사위원들은 U-DiT의 기여가 아이디어의 표면적 결합이 아니라, **'토큰 다운샘플링'이라는 구체적인 구현 방식**과 그것이 가져온 **압도적인 실험 결과**에 있음을 강조했다. 즉, 어떻게 결합했고, 그 결과가 얼마나 효과적인지를 증명한 것이 핵심적인 공헌이라는 것이다.21


U-DiT는 트랜스포머 기반 확산 모델의 효율성을 극대화했지만, 셀프 어텐션의 제곱 복잡도(O(N2))라는 근본적인 한계는 여전히 남아있다. 특히 초고해상도 이미지나 긴 비디오를 생성하는 데 있어 어텐션 메커니즘은 여전히 계산 병목 현상을 유발한다.

이러한 배경 속에서 트랜스포머를 대체할 차세대 시퀀스 모델링 아키텍처로 **상태 공간 모델(State-Space Models, SSMs)**, 특히 **Mamba**가 부상하고 있다.29 Mamba는 시퀀스 길이에 대해 선형 시간 복잡도($O(N)$)를 가지면서도 장거리 의존성을 효과적으로 모델링할 수 있어, 트랜스포머의 강력한 성능과 RNN의 효율성을 결합한 대안으로 주목받고 있다.31

이미 **DiMSUM(Diffusion Mamba)**과 같은 모델들이 Mamba를 확산 모델의 백본으로 활용하는 연구를 진행 중이다.29 DiMSUM은 Mamba 아키텍처에 웨이블릿 변환(wavelet transform)과 같은 기법을 결합하여, 기존 DiT보다 더 빠른 훈련 수렴 속도와 고해상도에서의 뛰어난 효율성을 달성했다고 주장한다.29

이러한 흐름 속에서 U-DiT는 역사적으로 중요한 위치를 차지한다. U-DiT는 트랜스포머 기반 확산 모델 패러다임 내에서 가장 정제되고 효율적인 아키텍처 중 하나로 평가받을 수 있다. 동시에, U-DiT가 트랜스포머의 효율성을 한계까지 끌어올림으로써, 역설적으로 셀프 어텐션 자체의 한계를 명확히 하고 Mamba와 같은 비-어텐션 기반 아키텍처의 필요성을 부각시키는 역할을 했다. 따라서 U-DiT는 현재의 최적화된 솔루션이자 미래 아키텍처로 나아가는 중요한 '징검다리' 모델로 볼 수 있다.


U-DiT(U-Shaped Diffusion Transformer)는 확산 모델 아키텍처 발전의 역사에서 단순한 점진적 개선을 넘어, 지배적인 두 패러다임의 비판적 종합을 통해 새로운 효율성의 기준을 제시한 중요한 이정표이다. 이 모델은 DiT의 등장 이후 생성 AI 분야를 휩쓸었던 '등방성 트랜스포머'라는 설계 철학에 근본적인 질문을 던졌다. 즉, 트랜스포머의 막강한 확장성을 얻기 위해 U-Net이 오랜 시간 검증해 온 계층적 구조와 스킵 연결의 귀납적 편향을 완전히 포기하는 것이 과연 최선이었는가에 대한 해답을 제시했다.

U-DiT는 U-Net의 구조가 구시대의 유물이 아니라, 트랜스포머의 확장성과 매우 효과적으로 시너지를 낼 수 있는 원리임을 증명했다. 특히, '다운샘플링된 토큰에서의 셀프 어텐션'이라는 핵심적인 혁신은 이론적으로 타당하면서도 실용적인 해결책이었다. 이는 노이즈 제거라는 확산 모델의 본질적 과업에 맞춰, 고주파수 노이즈 정보를 의도적으로 배제하고 저주파수 핵심 구조에 집중하도록 아키텍처 자체에 편향을 부여한 것이다. 그 결과, 계산 비용을 획기적으로 절감하면서도 오히려 생성 품질을 향상시키는, 기존의 상식을 뛰어넘는 성과를 달성했다.

Playground V3와 같은 최첨단 상용 모델이 U-DiT의 핵심 설계 원칙을 채택했다는 사실은 이 모델의 기술적 기여가 학술적 논의를 넘어 산업 현장에서도 실질적인 가치를 지니고 있음을 입증한다. U-DiT는 FLOPs당 성능(performance-per-FLOP)이라는 척도에서 새로운 기준점을 설정했으며, 향후 생성 모델의 발전이 무한한 계산 자원의 확장에만 의존하는 것이 아니라, 지능적인 아키텍처 설계를 통해 이루어질 수 있다는 중요한 가능성을 열었다. 이처럼 U-DiT는 트랜스포머 기반 확산 모델의 완성도를 한 단계 끌어올린 동시에, Mamba와 같은 차세대 시퀀스 모델의 등장을 촉진하는 기술적 변곡점으로서 그 역사적 의의를 가진다.


1. Towards Stabilized and Efficient Diffusion Transformers through Long-Skip-Connections with Spectral Constraints - arXiv, accessed July 19, 2025, https://arxiv.org/html/2411.17616v4
2. Diffusion Transformer (DiT) Models: A Beginner's Guide - Encord, accessed July 19, 2025, https://encord.com/blog/diffusion-models-with-transformers/
3. Self-Supervised Anomaly Segmentation via Diffusion Models with Dynamic Transformer UNet - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/WACV2025/papers/Kumar_Self-Supervised_Anomaly_Segmentation_via_Diffusion_Models_with_Dynamic_Transformer_UNet_WACV_2025_paper.pdf
4. Scalable Diffusion Models with Transformers - William Peebles, accessed July 19, 2025, https://www.wpeebles.com/DiT.html
5. [DiT] Scalable Diffusion Models with Transformers - Dev In Seoul - 티스토리, accessed July 19, 2025, https://dev-in-seoul.tistory.com/51
6. Scalable Diffusion Models with Transformers - alphaXiv, accessed July 19, 2025, https://www.alphaxiv.org/overview/2212.09748
7. ArXiv Dives - Diffusion Transformers - Oxen.ai, accessed July 19, 2025, https://www.oxen.ai/blog/arxiv-dives-diffusion-transformers
8. Stable Diffusion 3: Multimodal Diffusion Transformer Model Explained - Encord, accessed July 19, 2025, https://encord.com/blog/stable-diffusion-3-text-to-image-model/
9. DiC: Rethinking Conv3x3 Designs in Diffusion Models - arXiv, accessed July 19, 2025, https://arxiv.org/html/2501.00603v1
10. [2405.02730] U-DiTs: Downsample Tokens in U-Shaped Diffusion Transformers - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2405.02730
11. U-DiTs: Downsample Tokens in U-Shaped Diffusion Transformers - arXiv, accessed July 19, 2025, https://arxiv.org/html/2405.02730v3
12. A Journey from U-Net to Diffusion Models:Part 1of Generative AI with Diffusion Models | by yasmine karray | Medium, accessed July 19, 2025, https://medium.com/@ykarray29/a-journey-from-u-net-to-diffusion-models-part-1-of-generative-ai-with-diffusion-models-b228b4306b86
13. U-REPA: Aligning Diffusion U-Nets to ViTs - arXiv, accessed July 19, 2025, https://arxiv.org/html/2503.18414v1
14. Diffusion Transformer Explained | Towards Data Science, accessed July 19, 2025, https://towardsdatascience.com/diffusion-transformer-explained-e603c4770f7e/
15. [Generative Model] Diffusion Transformer (DiT) - Youngdo Lee, accessed July 19, 2025, https://leeyngdo.github.io/blog/generative-model/2024-07-01-diffusion-transformer/
16. U-DiTs: Downsample Tokens in U-Shaped Diffusion Transformers - arXiv, accessed July 19, 2025, https://arxiv.org/html/2405.02730v1
17. Scalable Diffusion Models with Transformers - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/ICCV2023/papers/Peebles_Scalable_Diffusion_Models_with_Transformers_ICCV_2023_paper.pdf
18. [2212.09748] Scalable Diffusion Models with Transformers - ar5iv, accessed July 19, 2025, https://ar5iv.labs.arxiv.org/html/2212.09748
19. Swin DiT: Diffusion Transformer using Pseudo Shifted Windows - arXiv, accessed July 19, 2025, https://arxiv.org/html/2505.13219v1
20. [Literature Review] U-DiTs: Downsample Tokens in U-Shaped Diffusion Transformers, accessed July 19, 2025, https://www.themoonlight.io/en/review/u-dits-downsample-tokens-in-u-shaped-diffusion-transformers
21. U-DiTs: Downsample Tokens in U-Shaped Diffusion Transformers | OpenReview, accessed July 19, 2025, [https://openreview.net/forum?id=SRWs2wxNs7&referrer=%5Bthe%20profile%20of%20Hanting%20Chen%5D(%2Fprofile%3Fid%3D~Hanting_Chen1)](https://openreview.net/forum?id=SRWs2wxNs7&referrer=[the+profile+of+Hanting+Chen](/profile?id%3D~Hanting_Chen1))
22. U-DiTs: Downsample Tokens in U-Shaped Diffusion Transformers - NIPS, accessed July 19, 2025, https://proceedings.neurips.cc/paper_files/paper/2024/file/5d2e24df9cfaad3189833b819c40b392-Paper-Conference.pdf
23. PDE-Transformer: Efficient and Versatile Transformers for Physics Simulations, accessed July 19, 2025, [https://openreview.net/forum?id=3BaJMRaPSx¬eId=uRVWUlXe5E](https://openreview.net/forum?id=3BaJMRaPSx&noteId=uRVWUlXe5E)
24. FiT: Flexible Vision Transformer for Diffusion Model - arXiv, accessed July 19, 2025, https://arxiv.org/html/2402.12376v4
25. baofff/U-ViT: A PyTorch implementation of the paper "All are Worth Words: A ViT Backbone for Diffusion Models". - GitHub, accessed July 19, 2025, https://github.com/baofff/U-ViT
26. NeurIPS Poster U-DiTs: Downsample Tokens in U-Shaped Diffusion Transformers, accessed July 19, 2025, https://neurips.cc/virtual/2024/poster/95100
27. [NeurIPS 2024] The official code of "U-DiTs: Downsample Tokens in U-Shaped Diffusion Transformers" - GitHub, accessed July 19, 2025, https://github.com/YuchuanTian/U-DiT
28. Playground v3: Improving Text-to-Image Alignment with Deep-Fusion Large Language Models - arXiv, accessed July 19, 2025, https://arxiv.org/html/2409.10695v1
29. NeurIPS Poster DiMSUM: Diffusion Mamba - A Scalable and Unified ..., accessed July 19, 2025, https://neurips.cc/virtual/2024/poster/95642
30. State-Space Models in Computer Vision (2023–2025) – a Deep Dive - Scribd, accessed July 19, 2025, https://www.scribd.com/document/869404775/State-Space-Models-in-Computer-Vision-2023-2025-a-Deep-Dive
31. NeurIPS Poster Demystify Mamba in Vision: A Linear Attention Perspective, accessed July 19, 2025, https://neurips.cc/virtual/2024/poster/95559
32. arxiv.org, accessed July 19, 2025, https://arxiv.org/html/2411.04168v3
33. DiMSUM : Diffusion Mamba - A Scalable and Unified Spatial-Frequency Method for Image Generation, accessed July 19, 2025, https://neurips.cc/media/neurips-2024/Slides/95642.pdf
34. DiMSUM : Diffusion Mamba - A Scalable and Unified Spatial-Frequency Method for Image Generation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2411.04168v1
35. DiMSUM: Diffusion Mamba - A Scalable and Unified Spatial-Frequency Method for Image Generation - Hao Phung, accessed July 19, 2025, https://hao-pt.github.io/dimsum/


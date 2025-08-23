# 3D 형상 생성을 위한 확산 트랜스포머 아키텍처

DiT-3D부터 진화된 변종 및 경쟁 구도까지



2D 이미지 합성이라는 잘 정립된 영역을 넘어, 3D 콘텐츠 생성 분야가 새로운 기술적 개척지로 부상하고 있다. 디지털 3D 자산은 게임, 영화, 로봇 공학, 의료 등 현대 산업의 근간을 이루는 핵심 요소로 자리 잡았으며, 창작자의 상상력을 구현하고 몰입감 있는 경험을 제공하는 데 필수적인 역할을 한다.1 이러한 수요 증가는 고품질 3D 자산을 효율적으로 생성할 수 있는 인공지능 기술의 발전을 촉진하고 있다.


3D 생성은 2D에 비해 본질적으로 더 복잡한 과제를 안고 있다. 데이터의 차원이 높아 계산 비용이 기하급수적으로 증가하며, 포인트 클라우드와 같은 3D 데이터 표현은 픽셀처럼 정렬된 구조가 아니어서 처리하기 까다롭다.5 또한, 생성된 객체는 기하학적, 구조적 일관성을 유지해야 하며, 대규모 고품질 3D 데이터셋은 2D 이미지에 비해 상대적으로 부족하여 모델 학습에 어려움을 겪는다.6 이러한 문제들은 3D 생성 모델의 발전을 가로막는 주요 장벽으로 작용해왔다.


이러한 상황에서, 노이즈 제거 확산 확률 모델(Denoising Diffusion Probabilistic Models, DDPMs)은 생성 작업에서 탁월한 성능을 보이며 새로운 가능성을 열었다.7 특히, 기존 생성 모델에서 널리 사용되던 U-Net 아키텍처를 대체할 후보로, 다른 AI 분야에서 확장성이 입증된 트랜스포머(Transformer) 아키텍처가 주목받기 시작했다. 이는 DiT-3D의 등장을 예고하는 중요한 기술적 흐름이었다.5

이러한 변화는 단순히 하나의 아키텍처를 다른 것으로 교체하는 수준을 넘어선다. U-Net에서 트랜스포머로의 전환은 AI 분야 전반에 걸친 '아키텍처 통합'이라는 거대한 흐름의 일부로 해석될 수 있다. 자연어 처리(NLP)에서 시작하여 컴퓨터 비전(ViT)으로 확장된 트랜스포머가 확산 모델(DiT, DiT-3D)에까지 적용된 것은, 다양한 데이터 양식(modality)에 걸쳐 적용 가능한 단일하고 확장성 높은 아키텍처로 수렴하려는 경향을 보여준다. 이 가설의 논리적 전개는 다음과 같다. 첫째, 원본 2D DiT 논문은 U-Net의 귀납적 편향(inductive bias)이 필수적인지 의문을 제기하며, 트랜스포머를 더 일반적이고 확장 가능한 대안으로 제시했다.10 둘째, DiT-3D 논문은 이러한 철학을 직접 계승하여 "사소한 적응"만으로 2D DiT를 3D 생성에 성공적으로 적용했다.5 셋째, 비디오, 3D, 게임 생성을 위한 통합 프레임워크를 목표로 하는 후속 연구들은 단일 아키텍처의 가능성을 더욱 명확히 보여준다.11 결론적으로, 이는 연구 커뮤니티가 각 데이터 유형에 맞는 개별 솔루션을 개발하는 대신, 강력하고 검증된 단일 아키텍처를 정교화하는 데 집중할 수 있게 하여 AI 기술 발전의 가속화를 이끌 수 있는 전략적 방향성을 시사한다.



Peebles와 Xie의 독창적인 DiT 논문은 2D 이미지 생성 분야의 기존 통념에 도전장을 내밀었다.10 당시 확산 모델의 중추는 대부분 U-Net 아키텍처였으나, 이 논문은 U-Net의 구조적 필요성이 검증되지 않았다는 점에 주목했다. 연구의 핵심 목표는 "아키텍처 선택의 중요성을 명확히 밝히고" 더 범용적인 트랜스포머가 U-Net과 동등하거나 더 나은 성능을 낼 수 있는지 실험적으로 증명하는 것이었다.10


DiT는 원본 픽셀이 아닌 잠재 공간(latent space)에서 작동한다. 이는 잠재 확산 모델(Latent Diffusion Models, LDM)로부터 계승된 핵심적인 설계 원칙이다. 이 과정은 사전 훈련된 변이형 오토인코더(Variational Autoencoder, VAE)를 사용하여 고차원 이미지를 저차원의 조밀한 잠재 공간으로 압축하고, 이 잠재 공간 내에서 확산 및 노이즈 제거 과정을 수행한 뒤, VAE의 디코더를 통해 최종 이미지를 복원하는 방식으로 이루어진다.10 이 접근법은 계산 부하를 획기적으로 줄여준다.


DiT는 비전 트랜스포머(Vision Transformer, ViT)의 방법론을 확산 모델에 맞게 변형하여 적용한다. 먼저, VAE를 통해 얻은 잠재 이미지를 겹치지 않는 여러 개의 패치(patch)로 분할한다. 이후 각 패치는 선형 임베딩을 통해 트랜스포머가 처리할 수 있는 토큰(token) 시퀀스로 변환된다.14 이 '패치화' 과정은 공간적 이미지 처리 문제를 트랜스포머의 본질적 영역인 시퀀스 처리 문제로 효과적으로 전환시킨다.


확산 모델은 노이즈 타임스텝 t에 대한 정보를 조건으로 받아야 하며, 클래스 조건부 모델의 경우 클래스 레이블 y도 추가로 입력받아야 한다. DiT 논문은 이를 위해 여러 방법을 실험했으며, 그중 **적응형 계층 정규화(adaptive layer norm, adaLN)**가 가장 뛰어난 성능을 보였다. adaLN은 타임스텝 및 클래스 임베딩 같은 조건부 정보를 추가적인 토큰으로 입력하는 대신, 이를 활용하여 트랜스포머 블록 내의 계층 정규화(Layer Normalization)에 사용될 스케일(γ) 및 시프트(β) 파라미터를 예측한다.8 이 방식은 적은 파라미터로 네트워크 전체의 동작을 효과적으로 조절할 수 있게 한다. 특히, 최종 블록 출력을 항등 함수(identity function)로 초기화하는 'adaLN-Zero' 변형은 모델의 안정적인 학습에 결정적인 역할을 했다.16


DiT 연구의 핵심적인 기여 중 하나는 모델 확장성의 주요 지표를 파라미터 수에서 **Gflops(초당 기가 부동소수점 연산)**로 전환한 것이다.10 연구 결과는 명확한 상관관계를 보여주었다. 즉, 모델의 깊이/너비를 늘리거나 패치 크기를 줄여 더 많은 토큰을 처리함으로써 Gflops를 높인 DiT 모델일수록 일관되게 더 낮은(더 좋은) FID(Fréchet Inception Distance) 점수를 달성했다.12 이는 DiT 아키텍처에 대한 명확하고 예측 가능한 스케일링 법칙(scaling law)을 확립한 것이다.

이러한 발견은 생성 AI의 '모델 스케일링' 개념을 재정의했다. 단순히 '더 많은 파라미터'를 추구하는 것에서 '더 많은 연산량(Gflops)'이라는 더 미묘하고 정확한 이해로 나아간 것이다. 특히, 파라미터 수는 거의 변화시키지 않으면서 Gflops와 토큰 시퀀스 길이를 크게 늘리는 '패치 크기 감소'가 강력한 스케일링 수단임이 밝혀진 점은 매우 통찰력 있다. 이는 생성 과제에서 모델이 더 많은 '픽셀'(또는 잠재 벡터)을 보고 어텐션 메커니즘을 통해 상호 관계를 학습하는 것이 단순히 네트워크를 깊게 만드는 것만큼, 혹은 그 이상으로 중요할 수 있음을 시사한다. 이 접근법은 다음과 같은 논리적 흐름을 따른다. 첫째, 전통적인 스케일링은 모델의 깊이와 너비를 늘려 파라미터를 직접적으로 증가시키는 데 초점을 맞춘다.14 둘째, DiT 논문은 파라미터 스케일링(DiT-S/B/L/XL)과 토큰 스케일링(패치 크기 8, 4, 2)을 체계적으로 분리하여 실험했다.14 셋째, 결과는 두 방법 모두 효과적이지만, 더 작은 패치 크기를 통해 토큰 수를 늘리는 것이 Gflops를 높이고 성능을 향상시키는 데 매우 효과적인 방법임을 보여주었다.12 결론적으로, 이는 다차원적인 스케일링 전략을 제시한다. DiT-3D와 같은 후속 모델 연구자들에게 이는 명확한 지침을 제공했다. 즉, 성능은 연산량의 함수이며, 연산량은 모델을 더 크게 만들거나, 모델이 더 세분화된 입력 토큰을 처리하도록 함으로써 증가시킬 수 있다는 것이다. 이는 DiT-3D가 모델 크기 및 복셀/패치 크기를 통한 스케일링을 탐구하는 실험 설계에 직접적인 영향을 미쳤다.5



Mo 연구팀이 발표한 DiT-3D는 성공적인 2D DiT 아키텍처를 3D 영역으로 확장하는 것을 목표로 했다.5 이 과정에서 두 가지 주요 난관에 직면했다. 첫째는 포인트 클라우드의 비정형적(unordered) 특성이었고, 둘째는 3차원으로 확장되면서 발생하는 계산 비용의 세제곱 증가 문제였다.5


포인트 클라우드의 비정형적 데이터 구조 문제를 해결하기 위해, DiT-3D는 입력된 포인트 클라우드를 이산적인 복셀 그리드(voxel grid) 표현으로 변환하는 단계를 가장 먼저 수행한다.5 이 과정은 트랜스포머가 처리할 수 있는 구조화된, 이미지와 유사한 입력을 생성하지만, 양자화 오류(quantization error)를 유발하고 데이터의 희소성(sparsity)을 증가시키는(대부분의 복셀이 비어 있음) 단점을 감수해야 한다.


- **3D 패치 임베딩:** 2D의 '패치화' 단계와 유사하게, 복셀 그리드는 겹치지 않는 3D 패치(또는 '큐보이드')로 분할된다. 이 패치들은 평탄화(flatten)된 후 선형 투사를 통해 트랜스포머의 입력 토큰으로 변환된다.5
- **3D 위치 임베딩:** 모델이 각 '큐보이드'의 3차원 공간 내 위치 정보를 유지할 수 있도록, 3D 위치 임베딩이 패치 임베딩에 더해진다. 이는 표준 트랜스포머 및 ViT에서 사용되는 1D/2D 위치 임베딩을 3차원으로 직접 확장한 것으로, 모델이 공간적 맥락을 이해하는 데 필수적이다.5


표준 셀프 어텐션(self-attention)은 3D에서 토큰의 시퀀스 길이가 해상도에 따라 세제곱으로 증가하기 때문에 계산적으로 엄청난 비용을 요구한다. 이 문제를 완화하기 위해 DiT-3D는 **3D 윈도우 어텐션(3D window attention)**을 도입했다.5 이는 모든 토큰에 걸쳐 어텐션을 계산하는 대신, 지역적인 3D 윈도우(예: 4×4×4 패치) 내로 어텐션 계산을 제한하는 방식이다. 이를 통해 계산 비용을 크게 절감하여 고해상도 복셀 그리드에서도 모델이 작동 가능하게 만든다.


DiT-3D의 핵심 기여 중 하나는 2D 사전 훈련된 DiT 모델을 활용한 파라미터 효율적 미세조정(fine-tuning)의 효과를 입증한 것이다.5 ImageNet으로 사전 훈련된 DiT-2D 모델의 가중치로 DiT-3D 모델을 초기화함으로써, 3D 형상 생성 작업의 학습 속도를 크게 높이고 성능을 향상시켰다. 이는 2D 이미지에서 학습된 특징이 3D 형상으로 놀랍도록 잘 전이될 수 있음을 보여주며, 3D 작업에 필요한 학습 파라미터 수를 32.8MB에서 단 0.09MB로 대폭 감소시켰다.5


공식 GitHub 리포지토리는 이러한 아키텍처를 구현하기 위한 도구들을 제공한다. 여기에는 학습 및 테스트 스크립트(`train.py`, `test.py`), 설정 가능한 모델 크기(S, B, L, XL), 복셀 크기(16, 32, 64), 패치 차원(2, 4, 8)이 포함되어 있다.19

이러한 개발 과정은 3D 생성 모델 설계에 있어 두 가지 상반된 철학, 즉 '표현 우선(Representation-First)'과 '아키텍처 우선(Architecture-First)' 접근법의 차이를 명확히 보여준다. DiT-3D는 '아키텍처 우선' 접근법의 대표적인 예시다. 연구진은 이미 성공적으로 검증된 아키텍처(DiT)에서 출발하여, "어떻게 하면 3D *데이터*를 이 아키텍처에 맞게 변형할 수 있을까?"라는 질문을 던졌다. 이 질문에 대한 해답이 바로 복셀화였다. 이 접근법의 논리적 흐름은 다음과 같다. 첫째, DiT-3D 논문은 "순수 확산 트랜스포머"를 3D에 적용하는 것을 명시적인 목표로 삼았으며, DiT-2D의 설계를 계승했다.5 둘째, 핵심적인 수정은 모델 자체가 아닌 

*입력 데이터 파이프라인*(포인트 클라우드 -->> 복셀 그리드 -->> 패치)에서 이루어졌다. 3D 임베딩이나 윈도우 어텐션 같은 적응은 있었지만, 핵심 트랜스포머 블록은 거의 그대로 유지되었다. 셋째, 이 방식은 3D 데이터를 2D와 유사한 그리드 구조로 강제 변환하는 것으로, 이는 일종의 타협이다. 반면, Shap-E나 Direct3D와 같은 후속 모델들은 트라이플레인(triplane)이나 암시적 함수(implicit function)와 같은 더 자연스러운 3D 표현에서 시작하여, "이 데이터를 처리하기 위해 어떤 아키텍처를 설계해야 하는가?"라는 '표현 우선' 접근법을 따른다.23 결론적으로, 이는 3D 생성 모델 설계의 근본적인 전략적 갈림길을 드러낸다. '아키텍처 우선' 경로(DiT-3D)는 입증된 확장 가능한 아키텍처의 힘을 활용하여 빠른 진보와 지식 전이(2D 사전 훈련 등)를 가능하게 한다. 반면, '표현 우선' 경로는 더 자연스러운 3D 형식에 맞는 새로운 아키텍처를 설계하여 더 높은 충실도를 목표로 하지만, 기존 아키텍처 재사용의 이점을 즉각적으로 누리기는 어렵다.



DiT-3D는 성공적인 모델이었지만, 복셀 그리드와 3D 어텐션(윈도우 방식이라도)에 의존했기 때문에 고해상도 3D 형상 생성을 위한 학습 비용이 "엄두를 내지 못할 정도로 비쌌다".27 이 계산적 부담은 후속 모델인 FastDiT-3D 개발의 직접적인 동기가 되었다.


- **핵심 아이디어:** FastDiT-3D는 마스크드 오토인코더(Masked Autoencoders, MAE)와 MaskDiT에서 영감을 얻었다.27 이 모델은 3D 복셀 그리드가 본질적으로 중복되고 희소하다는 통찰, 즉 대부분의 복셀이 빈 공간("배경")이라는 점에 착안했다.28
- **복셀 인지 마스킹:** 모든 복셀 패치를 처리하는 대신, FastDiT-3D는 "복셀 인지(voxel-aware)" 또는 "전경-배경 인지(foreground-background aware)" 마스킹 전략을 사용한다.28 이 전략은 입력 토큰의 매우 높은 비율을 마스킹하지만, 정보가 풍부한 전경 복셀과 정보가 부족한 배경 복셀을 구별하여 지능적으로 수행된다.31 이를 통해 거의 99%에 달하는 극단적인 마스킹 비율(예: 배경 99% 마스킹, 전경 95% 마스킹)을 달성할 수 있다.27 노이즈 제거 과정은 마스킹되지 않은 소수의 토큰에 대해서만 수행되어 계산량을 극적으로 줄인다.
- **다중 카테고리 생성을 위한 전문가 혼합(MoE):** ShapeNet과 같이 여러 객체 카테고리가 있는 데이터셋에서의 성능을 향상시키기 위해, FastDiT-3D는 트랜스포머 블록에 전문가 혼합(Mixture-of-Experts, MoE) 계층을 도입했다.27 이는 모델이 각기 다른 카테고리에 대해 서로 다른 "전문가"(특화된 피드포워드 네트워크)를 학습하도록 한다. 추론 시, 게이팅 네트워크(gating network)가 토큰을 적절한 전문가에게 라우팅하여 카테고리 간의 그래디언트 충돌을 완화하고, 각 카테고리가 고유한 확산 경로를 학습하도록 돕는다.27
- **성능 향상:** 이 접근법은 극적인 결과를 낳았다. FastDiT-3D는 128 해상도 복셀 생성 시, 기존 DiT-3D 학습 비용의 단 **6.5%**만을 사용하면서도 최첨단 성능을 달성했다.27


효율성을 향한 탐구는 계속되고 있다. **DiM-3D**와 같은 모델의 등장은 트랜스포머의 어텐션 메커니즘 자체에 대한 대안을 모색하는 움직임을 보여준다.32 DiM-3D는 시퀀스 길이에 대해 선형 복잡도를 갖는 Mamba 기반 아키텍처를 사용하여, 특히 256 해상도 복셀과 같은 초고해상도 환경에서 어텐션의 확장성 문제를 해결하고자 한다.32

| 특징              | DiT-2D (Peebles & Xie)                    | DiT-3D (Mo et al.)                 | FastDiT-3D (Mo et al.)             |
| ----------------- | ----------------------------------------- | ---------------------------------- | ---------------------------------- |
| **주요 목표**     | 확산 모델을 위한 트랜스포머의 확장성 입증 | 3D 포인트 클라우드 생성에 DiT 적용 | DiT-3D의 막대한 학습 비용 절감     |
| **입력 데이터**   | 2D 이미지                                 | 3D 포인트 클라우드                 | 3D 포인트 클라우드                 |
| **내부 표현**     | 2D 잠재 공간 패치                         | 3D 복셀 패치                       | 마스킹된 3D 복셀 패치              |
| **핵심 아키텍처** | `adaLN`을 통한 조건화                     | 3D 위치 임베딩, 3D 윈도우 어텐션   | 복셀 인지 마스킹, 전문가 혼합(MoE) |
| **해결한 병목**   | 해당 없음 (기초 연구)                     | 3D 어텐션의 세제곱 계산 복잡도     | 고해상도 복셀의 과도한 학습 비용   |

*표 1: DiT-2D에서 FastDiT-3D까지의 아키텍처 진화 과정 요약. 이 표는 각 모델의 핵심 혁신과 해결 과제를 명확히 보여주며, 기술적 발전의 계보를 추적할 수 있도록 돕는다.*



3D 생성 모델 분야의 경쟁 구도는 3D 기하학을 생성 모델에 가장 잘 표현하는 방법에 대한 논쟁으로 요약될 수 있다. 아직 이 문제에 대한 명확한 합의는 없다.34 본 섹션에서는 DiT-3D의 복셀 기반 접근법을 다른 세 가지 주요 철학과 비교 분석한다.


- **아키텍처:** OpenAI의 Point-E는 3D 형상을 명시적인 포인트 클라우드로 직접 생성한다.25 이 모델은 다단계 확산 과정을 사용한다: 먼저 텍스트-이미지 모델(GLIDE)로 이미지를 생성하고, 이 이미지를 조건으로 거친 포인트 클라우드를 생성한 뒤, 업샘플링 모델을 통해 세밀한 포인트 클라우드를 만든다.25
- **장단점:** 이 접근법은 이름('E'는 효율성을 의미)에서 알 수 있듯이 매우 빠르다.35 하지만 생성된 결과물의 품질과 충실도가 낮고, 포인트 클라우드는 종종 세밀한 디테일이나 텍스처를 포착하지 못한다.25 이는 빠른 프로토타이핑에 적합한 '질보다 양'의 접근법이라 할 수 있다.


- **아키텍처:** Point-E의 직접적인 후속 모델인 OpenAI의 Shap-E는, 메시(mesh)나 뉴럴 래디언스 필드(NeRF)로 렌더링될 수 있는 암시적 함수(implicit function)의 파라미터를 직접 생성한다.25 이 모델은 2단계 과정을 거친다: 먼저 인코더가 3D 자산을 암시적 함수의 파라미터로 매핑하도록 학습하고, 그 다음 이 파라미터에 대해 조건부 확산 모델을 학습시킨다.26
- **장단점:** Shap-E는 더 복잡한 출력 공간을 모델링함에도 불구하고 Point-E보다 더 빠르게 수렴하고 비슷하거나 더 나은 품질의 결과를 생성한다.26 또한 메시와 NeRF라는 여러 방식으로 렌더링할 수 있는 유연성을 제공한다. DreamFusion과 같은 최적화 기반 방법보다는 빠르지만, 아직 품질 면에서는 최첨단 수준에 도달하지 못했다.36


- **아키텍처 (Direct3D):** Direct3D와 같은 모델은 VAE(D3D-VAE)를 사용하여 3D 형상을 조밀한 잠재 *트라이플레인(triplane)* 표현으로 인코딩한다. 그 다음, DiT 기반 모델(D3D-DiT)이 입력 이미지를 조건으로 이 트라이플레인 잠재 공간에서 작동한다.6 이는 다중 시점 확산이나 SDS(Score Distillation Sampling) 최적화 없이 직접 3D를 생성하는 "네이티브 3D" 접근법이다.23
- **아키텍처 (Hunyuan3D):** 유사하게, Hunyuan3D는 형상과 텍스처 생성을 분리한다. 형상 모델인 **Hunyuan3D-DiT**는 흐름 기반 확산 모델(flow-based diffusion model)로, 먼저 텍스처가 없는 메시를 생성한다. 그 후, 별도의 텍스처 합성 모델인 **Hunyuan3D-Paint**가 이 메시에 텍스처를 입힌다.1
- **장단점:** 이 방법들은 3D 데이터로부터(또는 이미지를 조건으로) 직접 고품질의 확장 가능한 생성을 목표로 한다. 이들은 조밀한 복셀보다 효율적이면서도 원시 포인트 클라우드보다 구조화된, 명시적이면서도 압축된 표현(트라이플레인)을 사용하여 중간 지점을 찾는다. 주요 장점은 대규모의 실제 데이터셋(in-the-wild datasets)에 대한 확장성과 높은 품질의 결과물이다.6

| 모델         | 주요 표현 방식           | 생성 과정                               | 핵심 강점                                             | 핵심 약점                                     | 추론 속도 |
| ------------ | ------------------------ | --------------------------------------- | ----------------------------------------------------- | --------------------------------------------- | --------- |
| **DiT-3D**   | 복셀화된 포인트 클라우드 | 복셀 공간에서의 직접 노이즈 제거        | DiT의 간단한 변형, 무조건부 생성에 유리               | 높은 계산 비용, 복셀화로 인한 정보 손실       | 보통      |
| **Point-E**  | 명시적 포인트 클라우드   | 다단계 확산 (이미지-->>PC-->>PC)              | 매우 빠른 추론                                        | 낮은 품질, 텍스처 부재, 복잡한 파이프라인     | 매우 빠름 |
| **Shap-E**   | 암시적 함수 (NeRF/SDF)   | 암시적 함수 파라미터에 대한 확산        | 빠름, 유연한 출력(메시/NeRF), Point-E보다 우수한 품질 | 품질이 SOTA는 아님, 아티팩트 발생 가능        | 빠름      |
| **Direct3D** | 잠재 트라이플레인        | VAE + 트라이플레인 잠재 공간에서의 확산 | 고품질, 대규모 데이터셋 확장성, 네이티브 이미지-3D    | 대규모(주로 비공개) 데이터셋 의존, 복잡한 VAE | 보통~느림 |

*표 2: 3D 생성 모델 철학 비교 분석. 이 표는 각 모델이 3D 공간을 표현하는 근본적인 설계 철학을 기준으로 분류하여, 속도, 품질, 유연성 등 다양한 목표에 따라 최적화된 선택지가 존재함을 보여준다.*



가장 즉각적인 적용 분야는 비디오 게임, 영화, 광고용 3D 자산의 자동 또는 반자동 제작이다.1 DiT-3D와 같은 모델은 아티스트가 후속 작업을 할 수 있는 다양한 기본 형태를 생성하여 제작 파이프라인의 속도를 극적으로 높일 수 있다. 특히 형상과 텍스처 생성을 분리한 Hunyuan3D 시스템은 이러한 작업 흐름에 명시적으로 설계되었다.1


- **시뮬레이션:** 생성 모델은 로봇을 실제 환경에 배치하기 전에 가상 세계에서 훈련하고 테스트하기 위한 방대하고 다양한 가상 세계와 객체를 생성할 수 있다.4 이는 실제 훈련보다 저렴하고 안전하다.
- **로봇 학습:** DiT 기반 아키텍처는 모델이 일련의 행동을 생성하는 로봇 정책(robot policy) 자체에도 탐구되고 있다. 이는 더 지능적이고 적응력 있는 로봇을 만드는 유망한 미래 방향이다.47 3D 프린팅을 사용하여 스스로 성장하는 로봇인 FiloBot과 같은 프로젝트는 생성 기술과 물리적 로봇 공학 간의 시너지를 보여준다.4


- **환자 맞춤형 모델링:** 의료 스캔(CT, MRI 등)으로부터 환자의 해부학적 구조(장기, 뼈 등)를 고품질 3D 모델로 생성하는 것은 핵심적인 응용 분야다.2 이 모델들은 수술 계획에 사용될 수 있으며, 외과의사가 환자의 가상 복제품으로 복잡한 수술을 예행연습할 수 있게 한다.48
- **맞춤형 임플란트 및 장치:** 생성된 3D 모델은 환자 맞춤형 임플란트, 보철물 또는 수술 가이드를 설계하고 3D 프린팅하는 데 사용되어 더 나은 임상 결과를 이끌어낼 수 있다.2
- **디지털 트윈:** 이는 생성 모델이 생물학적 시스템(예: 심장)이나 심지어 사람 전체의 동적인 고충실도 가상 복제품을 만드는 더욱 진보된 응용 분야다.2 이러한 "디지털 트윈"은 질병의 진행을 시뮬레이션하고, 신약의 효과를 가상으로 테스트하며, 개인 맞춤형 치료 계획을 개발하는 데 사용되어 의학을 혁신하고 물리적 실험의 필요성을 줄일 수 있다.

이러한 응용 분야들은 단순히 모델에서 응용으로 향하는 일방적인 흐름을 넘어선다. 강력한 피드백 루프, 즉 '생성적 시뮬레이션(Generative Simulation)' 패러다임이 부상하고 있다. 생성 모델은 로봇 공학이나 의료 분야의 시뮬레이션을 위한 합성 데이터를 만든다. 이 시뮬레이션의 결과는 다시 생성 모델을 더욱 정교하게 만드는 새로운 학습 데이터로 사용될 수 있어, 개선의 선순환을 만들어낸다. 이 과정의 논리적 흐름은 다음과 같다. 첫째, 생성 모델은 시뮬레이션용 자산을 만들 수 있다.4 둘째, 시뮬레이션은 유체 역학이나 응력 분석과 같은 데이터를 생성한다. 셋째, 생성 모델은 데이터로부터 학습한다. 결론적으로, 시뮬레이션의 출력을 생성 모델의 학습 루프 입력으로 다시 연결하는 것이 논리적으로 타당하다. 예를 들어, 생성된 심장 모델을 혈류 시뮬레이션에 사용했을 때 비생리학적인 결과가 나온다면, 이 정보를 생성 모델에 페널티로 제공하여 앞으로 더 현실적인 심장을 생성하도록 학습시킬 수 있다. 이는 일회성 자산 제작을 넘어, 시뮬레이터에 내장된 물리 법칙을 학습하여 스스로 개선되는 시스템을 구축하는 더 강력한 개념이다.



- **표현 방식:** 초기 DiT-3D 모델은 복잡하고 텍스처가 있으며 물이 새지 않는(watertight) 메시가 아닌, 복셀 표현으로부터 포인트 클라우드를 생성하는 데 국한된다.56
- **데이터 편향:** 모델들은 ShapeNet과 같은 기존 벤치마크로 학습되기 때문에 5, 해당 데이터에 존재하는 편향(다양성 부족, 특정 객체 스타일 등)을 그대로 학습하고 증폭시킬 수 있다.56


- **데이터 문제:** 이 분야의 주요 약점 중 하나는 Direct3D와 같은 최첨단 모델을 학습시키기 위해 대규모 *내부(internal)* 데이터셋에 의존한다는 점이다.6 이는 학계 연구자들이 결과를 재현하기 어렵게 만들고 불공정한 비교 기준을 생성하여 과학적 진보를 저해한다.6
- **벤치마킹 및 평가:** 3D 생성 모델을 어떻게 평가할지에 대한 보편적인 합의가 부재하다.34 논문마다 서로 다른 지표(CD, EMD, IoU, CLIP-Score, 사용자 연구 등)를 사용하여 직접적이고 공정한 비교를 어렵게 만든다.6 3DMark와 같은 합성 벤치마크 역시 특정 모델에 유리하게 조작될 수 있으며 실제 성능을 반영하지 못할 수 있다.60
- **텍스처링 및 애니메이션:** 현재 대부분의 연구는 정적인 기하학 생성에 초점을 맞추고 있다. 고품질의 제어 가능한 텍스처를 만들고 생성된 자산을 쉽게 애니메이션화하는 것은 여전히 중요한 미해결 과제다. Hunyuan3D와 같은 모델은 작업을 분리함으로써 이 문제를 해결하기 시작했다.1


- **공개 데이터로의 확장:** 핵심적인 방향은 Objaverse와 같은 대규모 공개 3D 데이터셋에서 모델을 성공적으로 확장하여 연구를 민주화하고 더 일반화 가능한 모델을 만드는 것이다.6
- **새로운 표현 방식 탐구:** 미래는 단순한 포인트 클라우드와 복셀을 넘어 부호 거리 함수(SDFs), 메시, 그리고 하이브리드 방법과 같은 더 표현력 있는 표현 방식으로 나아갈 것이다.56 Direct3D의 트라이플레인 6과 Shap-E의 암시적 함수 37의 성공이 이러한 방향을 가리킨다.
- **다중 모드 및 통합 모델:** 중요한 개척지는 단일 프레임워크 내에서 다양한 입력(텍스트, 이미지, 오디오)으로부터 여러 양식(비디오, 3D, 게임)을 생성할 수 있는 통합 아키텍처의 개발이다.11
- **제어 가능성 및 구성 가능성:** 미래의 모델은 사용자가 단순히 "의자"가 아닌 "빨간 쿠션이 있는 나무 팔걸이 의자"와 같이 더 세밀하게 제어하고, 여러 생성된 객체로부터 복잡한 장면을 구성할 수 있는 능력을 제공해야 한다.



DiT-3D의 주된 기여는 U-Net에 대한 의존을 깨고, 최소한의 수정만으로 순수 트랜스포머 아키텍처가 3D 형상 생성 영역에 효과적으로 적용될 수 있음을 최초로 입증한 것이다. 이는 3D 생성 모델 연구에 새로운 방향을 제시한 근본적인 공헌이었다.


DiT-3D의 높은 계산 비용은 역설적으로 더 효율적인 방법을 찾기 위한 연구를 직접적으로 촉발시켰고, 이는 FastDiT-3D의 마스크드 트랜스포머 접근법과 같은 혁신으로 이어졌다. 이는 혁신이 최적화를 낳는 전형적인 연구 발전 주기를 보여준다.


결론적으로, DiT-3D와 그 후속 모델들은 3D AI의 미래에 대한 활발하고 지속적인 논쟁의 핵심 플레이어다. 어떤 데이터 표현 방식이 최선인지, 어떻게 효율적으로 확장할 것인지, 그리고 어떻게 통합되고 제어 가능한 모델을 구축할 것인지에 대한 중심적인 질문들은 여전히 열려 있다. DiT-3D 계열의 모델들은 이러한 질문에 대한 강력하고 영향력 있는 해답의 한 축을 제공하며, 차세대 3D 콘텐츠 제작 도구를 위한 길을 닦고 있다.


1. Hunyuan3D 2.0: Scaling Diffusion Models for High Resolution Textured 3D Assets Generation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2501.12202v1
2. 3D applications in Medical field : r/MedicalDevices - Reddit, accessed July 19, 2025, https://www.reddit.com/r/MedicalDevices/comments/1jcvvqc/3d_applications_in_medical_field/
3. Virtual and augmented reality for biomedical applications - PMC - PubMed Central, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8324499/
4. FiloBot: The Robot Grows Like a Plant Using 3D Printing - 3Dnatives, accessed July 19, 2025, https://www.3dnatives.com/en/filobot-the-robot-grows-like-a-plant-using-3d-printing-160220244/
5. DiT-3D: Exploring Plain Diffusion Transformers for 3D Shape Generation, accessed July 19, 2025, https://proceedings.neurips.cc/paper_files/paper/2023/file/d6c01b025cad37d5c8bab4ba18846c02-Paper-Conference.pdf
6. Direct3D: Scalable Image-to-3D Generation via 3D Latent Diffusion Transformer | OpenReview, accessed July 19, 2025, [https://openreview.net/forum?id=vCOgjBIZuL¬eId=j1NdkCt7JB](https://openreview.net/forum?id=vCOgjBIZuL&noteId=j1NdkCt7JB)
7. DiT-3D: Exploring Plain Diffusion Transformers for 3D Shape Generation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2307.01831
8. Diffusion Transformer (DiT) Models: A Beginner's Guide - Encord, accessed July 19, 2025, https://encord.com/blog/diffusion-models-with-transformers/
9. DiT-3D: Exploring Plain Diffusion Transformers for 3D Shape Generation - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/372136690_DiT-3D_Exploring_Plain_Diffusion_Transformers_for_3D_Shape_Generation
10. Scalable Diffusion Models with Transformers - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/ICCV2023/papers/Peebles_Scalable_Diffusion_Models_with_Transformers_ICCV_2023_paper.pdf
11. Diffusion Transformer on Unified Video, 3D, and Game Field Generation - OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=w6YS9A78fq
12. [2212.09748] Scalable Diffusion Models with Transformers - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2212.09748
13. Scalable Diffusion Models with Transformers - alphaXiv, accessed July 19, 2025, https://www.alphaxiv.org/overview/2212.09748
14. Scalable Diffusion Models with Transformers - William Peebles, accessed July 19, 2025, https://www.wpeebles.com/DiT.html
15. William Peebles | Papers With Code, accessed July 19, 2025, https://paperswithcode.com/author/william-peebles
16. DiTs explained from scratch. Diffusion transformers are new paradigm... | by Subho Ghosh | Medium, accessed July 19, 2025, https://medium.com/@ighoshsubho/dits-explained-from-scratch-4fdb9192b1cf
17. ArXiv Dives - Diffusion Transformers - Oxen.ai, accessed July 19, 2025, https://www.oxen.ai/blog/arxiv-dives-diffusion-transformers
18. Papers Explained 19: DiT. DiT is a self-supervised pre-trained... | by Ritvik Rastogi | DAIR.AI, accessed July 19, 2025, https://medium.com/dair-ai/papers-explained-19-dit-b6d6eccd8c4e
19. Official Codebase of "DiT-3D: Exploring Plain Diffusion Transformers for 3D Shape Generation" - GitHub, accessed July 19, 2025, https://github.com/DiT-3D/DiT-3D
20. DiT-3D: Exploring Plain Diffusion Transformers for 3D Shape Generation, accessed July 19, 2025, https://dit-3d.github.io/
21. Computer Science Jul 2023 - arXiv, accessed July 19, 2025, http://arxiv.org/list/cs/2023-07?skip=410&show=1000
22. NeurIPS-2023 Highlights (Full List) - Paper Digest, accessed July 19, 2025, https://www.paperdigest.org/data/neurips-2023-full.html
23. Direct3D: Scalable Image-to-3D Generation via 3D Latent Diffusion Transformer - NIPS, accessed July 19, 2025, https://proceedings.neurips.cc/paper_files/paper/2024/file/dc970c91c0a82c6e4cb3c4af7bff5388-Paper-Conference.pdf
24. Direct3D: Scalable Image-to-3D Generation via 3D Latent Diffusion Transformer - arXiv, accessed July 19, 2025, https://arxiv.org/html/2405.14832v1
25. Point-E and Shap-E - Rerun, accessed July 19, 2025, https://rerun.io/examples/generative-vision/shape_pointe
26. What is OpenAI's Shape-E and Will AI Replace 3D - AutoGPT, accessed July 19, 2025, https://autogpt.net/what-is-openais-shape-e-and-will-ai-replace-3d/
27. Fast Training of Diffusion Transformer with Extreme Masking for 3D ..., accessed July 19, 2025, https://dit-3d.github.io/FastDiT-3D/
28. Fast Training of Diffusion Transformer with Extreme Masking for 3D Point Clouds Generation, accessed July 19, 2025, https://arxiv.org/html/2312.07231v1
29. [2312.07231] Fast Training of Diffusion Transformer with Extreme Masking for 3D Point Clouds Generation - ar5iv, accessed July 19, 2025, https://ar5iv.labs.arxiv.org/html/2312.07231
30. [2312.07231] Fast Training of Diffusion Transformer with Extreme Masking for 3D Point Clouds Generation - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2312.07231
31. Fast Training of Diffusion Transformer with Extreme Masking for 3D Point Clouds Generation - European Computer Vision Association, accessed July 19, 2025, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/11278.pdf
32. Efficient 3D Shape Generation via Diffusion Mamba with Bidirectional SSMs - arXiv, accessed July 19, 2025, https://arxiv.org/html/2406.05038v1
33. Scaling Diffusion Mamba with Bidirectional SSMs for Efficient 3D Shape Generation, accessed July 19, 2025, https://ojs.aaai.org/index.php/AAAI/article/view/34144/36299
34. 3D Shape Tokenization - arXiv, accessed July 19, 2025, https://arxiv.org/html/2412.15618v1
35. OpenAI Announces Point-E System to Create 3D Models Using AI - 3Dnatives, accessed July 19, 2025, https://www.3dnatives.com/en/openai-point-e-system-3d-models-using-ai-261229225/
36. Shap-E is OpenAI's fastest text-to-3D model to date - The Decoder, accessed July 19, 2025, https://the-decoder.com/shap-e-is-openais-fastest-text-to-3d-model-to-date/
37. Shap-E: Generating Conditional 3D Implicit Functions - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/370524354_Shap-E_Generating_Conditional_3D_Implicit_Functions
38. [2305.02463] Shap⋅E: Generating Conditional 3D Implicit Functions - ar5iv - arXiv, accessed July 19, 2025, https://ar5iv.labs.arxiv.org/html/2305.02463
39. YiYiXu/shap-e - Hugging Face, accessed July 19, 2025, https://huggingface.co/YiYiXu/shap-e
40. [2305.02463] Shap-E: Generating Conditional 3D Implicit Functions - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2305.02463
41. Unleashing Shap-E. (Shap-E, a conditional generative model... | by Yash Vardhan Singh, accessed July 19, 2025, https://medium.com/@yashvardhanvs/unleashing-shap-e-faf5056dc09d
42. Shap/E: The Revolutionary 3D Asset Generator by OpenAI - YouTube, accessed July 19, 2025, https://www.youtube.com/watch?v=KcUtZ5JoGqs
43. Shap-E - Hugging Face, accessed July 19, 2025, https://huggingface.co/docs/diffusers/api/pipelines/shap_e
44. Comparison of Shap-e vs. Point-e results. Image obtained from [22]. - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/figure/Comparison-of-Shap-e-vs-Point-e-results-Image-obtained-from-22_fig4_390039044
45. [2405.14832] Direct3D: Scalable Image-to-3D Generation via 3D Latent Diffusion Transformer - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2405.14832
46. NeurIPS Poster Direct3D: Scalable Image-to-3D Generation via 3D Latent Diffusion Transformer, accessed July 19, 2025, https://neurips.cc/virtual/2024/poster/93214
47. The Ingredients for Robotic Diffusion Transformers - DiT-Policy, accessed July 19, 2025, https://dit-policy.github.io/resources/paper.pdf
48. Science Fiction and Surgical Precision: The Future of Medical Training with Marion Surgical Robot Game - Surgical Robotics Technology, accessed July 19, 2025, https://www.surgicalroboticstechnology.com/articles/science-fiction-and-surgical-precision-the-future-of-medical-training-with-marion-surgical-robot-game/
49. (PDF) From gaming to surgery: the influence of digital natives on robotic skills development, accessed July 19, 2025, https://www.researchgate.net/publication/386253666_From_gaming_to_surgery_the_influence_of_digital_natives_on_robotic_skills_development
50. 3D Modeling And Simulation - Meegle, accessed July 19, 2025, https://www.meegle.com/en_us/topics/digital-twin/3d-modeling-and-simulation
51. 15 Digital Twin Applications/ Use Cases by Industry in 2025 - Research AIMultiple, accessed July 19, 2025, https://research.aimultiple.com/digital-twin-applications/
52. Virtual Twin Experiences for Life Sciences & Healthcare - Dassault Systemes, accessed July 19, 2025, https://www.3ds.com/virtual-twin/life-sciences-healthcare
53. Digital twin in healthcare: Recent updates and challenges - PMC, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9830576/
54. Digital Twins vs Simulations: A Healthcare Revolution - 3DforScience, accessed July 19, 2025, https://3dforscience.com/blog/medical-marketing/digital-twins-vs-simulations-healthcare/
55. The Digital Twin in Medicine: A Key to the Future of Healthcare? - PMC - PubMed Central, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9330225/
56. DiT-3D: Exploring Plain Diffusion Transformers for 3D Shape Generation (Supplementary Material), accessed July 19, 2025, https://proceedings.neurips.cc/paper_files/paper/2023/file/d6c01b025cad37d5c8bab4ba18846c02-Supplemental-Conference.pdf
57. ShapeNet Dataset | Papers With Code, accessed July 19, 2025, https://paperswithcode.com/dataset/shapenet
58. ShapeNet, accessed July 19, 2025, https://shapenet.org/
59. NeurIPS 2024 Datasets Benchmarks 2024, accessed July 19, 2025, https://neurips.cc/virtual/2024/events/datasets-benchmarks-2024
60. Are benchmarking software like 3DMark really useful? : r/Amd - Reddit, accessed July 19, 2025, https://www.reddit.com/r/Amd/comments/70fv65/are_benchmarking_software_like_3dmark_really/


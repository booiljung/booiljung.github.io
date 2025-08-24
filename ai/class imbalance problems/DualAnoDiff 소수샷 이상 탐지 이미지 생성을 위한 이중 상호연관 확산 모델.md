[인공지능 훈련 데이터의 클래스 불균형 문제](./index.md)
# DualAnoDiff 소수샷 이상 탐지 이미지 생성을 위한 이중 상호연관 확산 모델



현대 산업 제조 공정은 고도로 최적화되어 있어, 생산 라인에서 결함이나 이상(anomaly)이 발생하는 빈도가 매우 낮다.1 이러한 높은 수율은 품질 관리 측면에서는 긍정적이지만, 인공지능 기반의 자동화된 시각 검사 시스템을 개발하는 데에는 심각한 제약으로 작용한다. 특히, 강력한 성능을 보이는 지도 학습(supervised learning) 기반의 딥러닝 모델은 다양하고 방대한 양의 결함 데이터를 학습해야만 높은 정확도와 일반화 성능을 확보할 수 있다.1 그러나 현실에서는 결함 데이터 자체가 극소수이거나 존재하지 않는 경우가 많아 '데이터 희소성(data scarcity)' 문제가 발생한다.5

이 문제는 새로운 생산 라인을 가동하거나, 반도체 공정과 같이 결함률이 극도로 낮은 산업 분야에서 더욱 두드러진다.4 또한, 소량의 결함 데이터를 수집하더라도 이를 분류하고 정확한 위치를 주석 처리(labeling)하는 데에는 높은 비용과 도메인 전문가의 시간이 소요된다.1 이러한 배경은 기존의 모델 중심(model-centric) 접근법의 한계를 드러내며, 부족한 데이터를 효과적으로 증강하고 활용하는 데이터 중심(data-centric) 접근법으로의 패러다임 전환을 촉구한다.6


데이터 희소성 문제를 해결하기 위한 핵심 전략으로 '이상 생성(anomaly generation)' 기술이 주목받고 있다. 이는 인공적으로 결함 이미지를 생성하여 부족한 학습 데이터를 보충하는 방법론으로, 다음과 같은 발전 단계를 거쳐왔다.

- **초기 증강 기법:** 가장 단순한 형태는 정상 이미지 위에 결함 패치를 오려 붙이는 'Cut-Paste'와 같은 방식이다.7 이 방법은 구현이 간단하지만, 생성된 결함이 배경과 조화를 이루지 못하고 비현실적인 경계면을 만들어내는 등 분포 외(out-of-distribution) 아티팩트를 생성하는 명백한 한계를 지닌다.7
- **GAN 기반 생성:** 생성적 적대 신경망(Generative Adversarial Networks, GANs)은 더 높은 충실도의 이미지를 생성할 수 있어 이상 생성 분야에서 중요한 진전을 이루었다.1 그러나 GAN은 학습 과정이 불안정하고, 생성되는 이미지의 다양성이 부족해지는 '모드 붕괴(mode collapse)' 현상이 발생하기 쉬우며, 생성된 결함이 원본 이미지의 질감이나 조명과 자연스럽게 융합되지 못하는 문제를 종종 드러낸다.9
- **확산 모델(Diffusion Models, DMs)의 부상:** 확산 모델은 GAN이나 변분 오토인코더(Variational Autoencoders, VAEs)보다 더 선명하고, 다양하며, 높은 품질의 샘플을 생성하는 능력으로 최신 생성 모델링 기술의 정점으로 부상했다.9 점진적인 노이즈 제거(denoising) 과정을 통해 데이터 분포를 학습하므로 학습이 안정적이고, 더 넓은 범위의 데이터 모드를 포착할 수 있어 복잡한 실제 데이터의 분포를 모델링하는 데 매우 적합하다.9


확산 모델은 이상 탐지 및 생성(Diffusion Models for Anomaly Detection, DMAD) 분야에서 다양한 방식으로 활용된다. DualAnoDiff의 접근법을 명확히 이해하기 위해, 기존 DMAD 방법론을 포괄적인 서베이 논문들을 기반으로 다음과 같이 분류할 수 있다.9

- **재구성 기반(Reconstruction-Based):** 모델이 노이즈가 추가된 입력 데이터를 원본으로 복원하도록 학습된다. 정상 데이터와 다른 분포를 가진 이상 데이터는 복원이 제대로 이루어지지 않아 높은 재구성 오류(reconstruction error)를 나타내며, 이 오류 값을 이상 점수(anomaly score)로 활용한다.10
- **밀도 기반(Density-Based):** 모델이 학습 데이터의 확률 밀도 함수를 추정하도록 한다. 이상 데이터는 정상 데이터 분포에서 벗어나 확률 밀도가 낮은 영역에 존재할 것이라는 가정에 기반하며, 추정된 밀도 값의 음수 로그 값을 이상 점수로 사용한다.10
- **점수/시간 추정 기반(Score-Based / Time Estimation):** 학습된 점수 함수(score function, 로그 확률 밀도의 그래디언트)의 크기나, 입력 데이터를 순수 노이즈로 만드는 데 필요한 확산 "시간"을 추정하여 이상을 탐지한다. 이상 데이터는 정상 데이터와 다른 점수 또는 시간 특성을 가질 것으로 예상된다.10
- **이상 생성(Anomaly Generation, AG):** DualAnoDiff가 속하는 범주이다. 이 접근법은 확산 모델을 *탐지기 자체*로 사용하는 대신, *합성 결함 데이터를 생성하는 도구*로 활용한다.1 이렇게 생성된 풍부한 합성 데이터는 희소한 실제 데이터를 보강하여, 이상 탐지, 분할, 분류 등 후속 과제(downstream task)를 위한 강력한 지도 학습 모델을 훈련시키는 데 사용된다.6

이러한 분류를 통해 볼 때, 초기 DMAD 연구가 확산 모델의 내재적 속성(재구성 오류, 밀도 등)을 직접적인 탐지에 활용하는 데 중점을 둔 반면, DualAnoDiff와 같은 최신 연구들은 전략적 전환을 보여준다. 즉, 범용 확산 모델이 최적의 *탐지기*는 아닐 수 있지만, 비할 데 없이 우수한 *데이터 합성기*라는 점에 착안한 것이다. 확산 모델을 통해 현실과 유사한 대규모 합성 데이터셋을 구축한 뒤, 최종 탐지 과제에는 U-Net이나 ResNet과 같이 고도로 최적화된 지도 학습 아키텍처를 활용하는 것이 시스템 전체의 성능을 극대화하는 길임을 DualAnoDiff의 우수한 후속 과제 성능이 입증한다.15 이는 생성과 탐지 문제를 분리하여 각각을 독립적으로 최적화하는, 보다 실용적인 데이터 중심 공학 솔루션으로의 전환을 의미한다.6


확산 모델의 강력한 성능에도 불구하고, DualAnoDiff 이전의 이상 생성 연구들은 몇 가지 핵심적인 난제를 해결하지 못했다. DualAnoDiff의 논문 초록과 서론은 이러한 문제들을 명확히 지적하며 연구의 동기로 삼고 있다.2

1. **제한된 다양성(Limited Diversity):** 기존 방법들은 종종 반복적이거나 유사한 형태의 결함만을 생성하여, 후속 탐지 모델의 일반화 성능을 저해했다.
2. **부자연스러운 합성 및 낮은 현실성(Poor Blending/Realism):** 생성된 결함이 원본 객체의 질감, 조명, 기하학적 특성과 자연스럽게 융합되지 못하고 이질적으로 보이는 문제가 있었다.
3. **마스크 부정렬(Mask Misalignment):** 분할(segmentation) 모델의 정답 레이블 역할을 하는 이상 영역 마스크가 생성된 결함의 위치와 정확히 일치하지 않는 경우가 많았다. 이는 후속 모델의 학습 데이터 품질을 저하시키는 심각한 문제였다.5

DualAnoDiff는 이러한 세 가지 핵심 과제를 정면으로 해결하는 것을 목표로 설계되었다.



DualAnoDiff는 극소수의 결함 샘플(예: 10개 미만)만을 사용하여 학습하는 '소수샷(few-shot)' 환경에 특화되어 설계되었다.1 이를 위해, 방대한 시각적 지식을 사전 학습한 대규모 잠재 확산 모델(Latent Diffusion Model)을 기반으로 구축된다.18 코드 저장소에 따르면 이 기반 모델은 `stable-diffusion-v1-5`이다.5 사전 학습된 모델을 활용함으로써, 소수의 결함 샘플에 과적합(overfitting)되는 것을 방지하고 현실적인 이미지를 생성하는 데 필요한 일반적인 시각 정보를 확보한다.

모델의 전체적인 파이프라인은 먼저 입력 이미지(I)와 결함 부분(Ia)을 사전 학습된 VAE(Variational Autoencoder) 인코더(ϵ(⋅))를 통해 잠재 표현(z, z′)으로 변환하는 것에서 시작한다.15 이후 모든 확산 및 노이즈 제거 과정은 연산 효율성이 높은 이 잠재 공간에서 이루어진다.4


DualAnoDiff의 가장 핵심적인 혁신은 '이중 상호연관 확산 모델(dual-interrelated diffusion model)' 구조에 있다.1 이는 물리적으로 두 개의 독립된 모델을 사용하는 것이 아니라, 단일 사전 학습 모델을 LoRA(Low-Rank Adaptation)라는 파라미터 효율적 미세 조정 기법을 통해 두 개의 특화된 분기(branch)로 확장한 것이다.7

- **전역 분기(Global Branch):** *전체 결함 이미지*를 생성하는 역할을 담당한다.
- **이상 분기(Anomalous Branch):** 동시에 *국소적인 결함 부분*만을 생성하는 역할을 한다.

이 '이중 분기' 개념은 강력하지만 연산 비용이 클 것으로 보일 수 있다. 그러나 이 구조의 핵심은 두 개의 LoRA 모듈을 사용한다는 점에 있다. 이는 단순한 구현 디테일을 넘어, DualAnoDiff 접근법 전체를 가능하게 하는 기술적 묘수이다. 첫째, LoRA는 전체 모델 파라미터의 극히 일부만을 학습하여 하나의 기반 모델로부터 '전역'과 '이상'이라는 두 개의 뚜렷한 기능적 "페르소나"를 효율적으로 생성할 수 있게 한다. 둘째, 이는 소수샷 학습 환경에서 필수적이다. 전체 모델을 미세 조정할 경우, 기반 모델이 가진 방대한 사전 지식을 잊어버리는 '파국적 망각(catastrophic forgetting)' 현상이나 소수의 샘플에 대한 극심한 과적합이 발생할 수 있기 때문이다. LoRA는 이러한 문제를 효과적으로 방지한다. 셋째, 이로 인해 전체 모델의 연산량을 감당 가능한 수준으로 유지할 수 있다. 따라서 'DualAnoDiff'는 '이중 확산 모델'이라기보다는 '단일 확산 모델의 이중 LoRA 특화'라고 이해하는 것이 그 본질을 더 정확하게 파악하는 것이다.


이 모듈은 모델 이름의 '상호연관(interrelated)' 부분을 담당하며, 결함과 배경의 자연스러운 융합 및 정렬 문제를 해결하는 데 결정적인 역할을 한다.7

- **메커니즘:** 노이즈 제거 과정의 각 자기-주의(self-attention) 계층에서 SAIM은 전역 분기와 이상 분기 간의 정보 공유를 촉진한다.15 논문에 따르면 이 모듈은 "두 분기의 주의 계층을 결합하고 공유된 주의 계산을 수행한다".7
- **추론된 구현 방식:** 구체적인 코드 수준의 설명은 없지만, 이는 각 분기의 해당 주의 블록에서 계산된 쿼리(Query, Q), 키(Key, K), 밸류(Value, V) 벡터를 최종 주의 맵(attention map) 계산 전에 서로 연결(concatenate)하거나 병합하는 방식으로 구현될 가능성이 높다. 이러한 공유 메커니즘은 전역 분기가 이상 분기에서 생성되는 특징들을 인지하도록 강제하며, 그 반대도 마찬가지다. 결과적으로 생성된 결함이 전체 객체의 형태, 질감, 조명과 같은 맥락적 요소들과 완벽하게 일관성을 갖도록 보장한다.


이 모듈은 소수샷 생성 시 흔히 발생하는 배경 왜곡이나 객체의 형태가 변형되는 문제를 해결하기 위해 도입되었다.1

- **메커니즘:** 원본 이미지의 배경을 명시적인 제어 신호로 사용한다.7 그 과정은 다음과 같다 15:
  1. 배경만 있는 이미지에 노이즈를 추가한다.
  2. 이를 확산 모델의 U-Net에 통과시켜 중간 특징(특히 주의 계층의 K와 V 행렬)을 추출한다.
  3. 추출된 배경 관련 특징들을 '적응형 융합 MLP(adaptive fusion MLP)'를 통해 *전역 분기*의 노이즈 제거 과정에 주입한다.

이 과정을 통해 모델의 생성 과정이 전경 객체와 결함에 집중하도록 유도하고, 배경이 변형되거나 흐려지는 현상을 방지하여 생성된 이미지의 전반적인 현실성을 크게 향상시킨다.


이중 분기 설계는 매우 우아한 방식으로 마스크 부정렬 문제를 해결한다. 이상 분기의 출력물 자체가 생성된 결함 부분이므로, 이 출력에서 비어 있지 않은 픽셀을 찾는 것만으로 매우 정확하고 픽셀 수준으로 정렬된 마스크를 얻을 수 있다.15 이는 기존 방법들이 겪었던 고질적인 문제를 구조적으로 해결한 것이다.5

흥미롭게도, 이 모델은 마스크 생성에 있어 두 가지 경로를 제공하며, 이는 구조적 순수성과 실용적 강건성 사이의 영리한 절충을 보여준다. 논문의 핵심 주장은 이상 분기의 출력이 곧 마스크라는 우아한 해결책이다.16 그러나 GitHub 저장소의 README 문서는 "U2-Net을 참조하여 생성된 fg 파일을 분할하라"고 명시적으로 안내한다.5 이는 실제 적용 시 이상 분기의 출력이 완벽하게 깨끗하지 않을 수 있으며, 강력한 사전 학습 분할 네트워크를 후처리로 적용하는 것이 후속 탐지기 학습에 더 안정적이고 깨끗한 마스크를 제공할 수 있음을 시사한다. 저자들은 이론적으로 우아한 해결책과 실용적으로 최상의 결과를 얻기 위한 보완책을 모두 제공함으로써 모델의 활용도를 높였다.



- **데이터셋:** 산업 이상 검사 분야의 표준 벤치마크인 MVTec AD 데이터셋을 사용하여 모델의 성능을 주로 검증했다.7
- **평가 방식:** 모델 평가는 생성된 이미지 자체의 품질(예: FID/KID 점수)보다는, 생성된 데이터를 사용하여 학습시킨 후속 이상 탐지/분할 모델의 성능을 측정하는 방식으로 이루어졌다.2 이는 실제 적용 관점에서 더 의미 있고 실용적인 평가 전략이다.
- **평가 지표:** 이미지 수준 AUROC(Area Under the Receiver Operating Characteristic curve), 픽셀 수준 AUROC, AP(Average Precision), F1-max 점수 등 해당 분야의 표준 평가 지표가 사용되었다.15


논문은 DualAnoDiff가 특히 픽셀 수준의 이상 탐지에서 SOTA(State-of-the-Art) 성능을 달성했다고 보고하며, 구체적으로 **픽셀 수준 AUROC 99.1%**와 **AP 84.5%**를 기록했다.7 아래 표는 MVTec AD 데이터셋에서 DualAnoDiff와 주요 경쟁 모델들의 성능을 요약한 것이다. 이 표는 DualAnoDiff가 생성한 데이터의 실질적인 유용성을 다른 생성 기반 접근법 및 표준 이상 탐지 모델과 직접적으로 비교하여, 그 성능 향상의 규모와 해당 분야에서의 위상을 즉각적으로 파악할 수 있게 해주는 핵심적인 증거이다.

| 모델명                 | 유형     | 이미지 수준 AUROC (%) | 픽셀 수준 AUROC (%) | 픽셀 수준 AP (%) | 픽셀 수준 F1-max (%) |
| ---------------------- | -------- | --------------------- | ------------------- | ---------------- | -------------------- |
| RD++                   | 탐지     | 97.1                  | 98.3                | 76.8             | 70.3                 |
| DifferNet              | 탐지     | 95.8                  | 98.2                | 75.9             | 69.1                 |
| UniAno                 | 생성     | 97.5                  | 98.6                | 77.4             | 71.2                 |
| AnomalyDiffusion       | 생성     | 98.1                  | 98.8                | 81.2             | 74.5                 |
| Anodapter              | 생성     | 98.5                  | 98.8                | -                | -                    |
| **DualAnoDiff (제안)** | **생성** | **98.3**              | **99.1**            | **84.5**         | **78.3**             |

표 1: MVTec AD 데이터셋에서의 후속 이상 검사 과제 성능 비교. 데이터는 7에서 재구성됨.

표에서 볼 수 있듯이, DualAnoDiff는 특히 이상 영역을 정확히 분할하는 능력을 평가하는 픽셀 수준 지표(AUROC, AP, F1-max)에서 모든 경쟁 모델을 능가하는 성능을 보였다. 이는 DualAnoDiff가 생성한 데이터와 마스크의 높은 품질과 정렬 정확도가 후속 분할 모델의 성능을 직접적으로 향상시켰음을 의미한다.


정량적 수치 외에도, 논문은 생성된 이미지의 시각적 품질 측면에서 현실성, 다양성, 마스크 정확성, 그리고 자연스러운 융합이라는 네 가지 핵심 요소에서 우수성을 주장한다.1 논문에 포함된 시각적 예시들은 이러한 주장을 뒷받침한다. 예를 들어, 경쟁 모델인 AnomalyDiffusion의 결과물이 다소 '붙여넣기'한 느낌을 주는 반면, DualAnoDiff가 생성한 결함들은 객체 표면의 질감 및 조명과 자연스럽게 통합되어 훨씬 더 현실적으로 보인다.7



DualAnoDiff는 소수샷 산업 이상 생성 분야에 다음과 같은 중요한 기여를 했다:

- 전역 이미지와 국소 결함을 동시에 생성하여 일관성과 정렬을 보장하는 새로운 이중 상호연관 확산 아키텍처를 제안했다.
- 배경의 무결성을 보존하여 생성 이미지의 현실성을 크게 향상시키는 전용 배경 보상 모듈(BCM)을 설계했다.
- 합성 데이터 학습의 핵심 문제였던 마스크 부정렬 문제를 구조적으로 해결하는 내재적 메커니즘을 제공했다.
- 주요 산업 벤치마크에서 SOTA 성능을 입증하여 제안된 접근법의 실질적인 효과를 검증했다.


우수한 성능에도 불구하고, DualAnoDiff는 몇 가지 잠재적 한계와 고려해야 할 사항들을 내포하고 있다.

- **연산 비용:** 확산 모델은 본질적으로 추론 속도가 느리다.11 DualAnoDiff의 이중 분기 구조는 LoRA를 사용함에도 불구하고, 각 스텝마다 U-Net의 순전파 연산(또는 그 변형)을 두 번 실행해야 할 가능성이 높다. 이는 단일 분기 모델에 비해 추론 시간과 메모리 요구량을 증가시켜 실시간 산업 환경에 적용하는 데 실질적인 장벽이 될 수 있다.
- **모델의 취약성(Brittleness):** GitHub 저장소에 보고된 "bcm-dual-interrelated_diff 사용 시 나쁜 생성 결과" 이슈는 중요한 사용자 피드백이다.21 이는 BCM을 활성화했을 때 모델의 성능이 특정 입력이나 설정에 민감하게 반응하여 실패 사례를 낳을 수 있음을 시사한다. 이러한 취약성은 종합적인 벤치마크 점수만으로는 포착하기 어려운 부분이다.
- **일반화의 한계:** 학습 스크립트와 README 문서는 모델이 "카테고리별"로 학습된다는 점을 명확히 하고 있다.5 이는 새로운 객체 유형이 추가될 때마다 별도의 모델을 처음부터 학습시켜야 함을 의미하며, 다양한 제품 라인을 가진 공장 환경에 확장 적용하기에는 번거롭고 비용이 많이 드는 심각한 한계이다.
- **아키텍처의 복잡성:** 두 개의 상호작용하는 LoRA 분기, SAIM, BCM의 도입은 DefectFill과 같은 단순한 인페인팅 기반 접근법 4이나 Anodapter와 같은 단일 어댑터 모델 19에 비해 시스템을 더 복잡하게 만든다. 이러한 복잡성은 모델의 학습, 디버깅, 미세 조정의 난이도를 높일 수 있다.


DualAnoDiff의 강점은 특화(이중 분기, BCM, 카테고리별 학습)에서 비롯되지만, 이는 유연성을 희생하는 대가로 얻어진다. 반면, 동시대의 연구 동향은 UniCombine과 같이 다양한 조건을 단일 모델에서 처리할 수 있는 통합 프레임워크로 나아가고 있다.22 향후 연구는 이 두 접근법의 장점을 결합하는 방향으로 나아가야 할 것이다. 예를 들어, 단일 기반 모델이 다수의 카테고리별 이중 LoRA 헤드를 지원하는 '다중 헤드' DualAnoDiff를 개발하거나, 더 발전된 조건부 메커니즘을 사용하여 텍스트나 예시 이미지로 제어되는 범용 결함 생성 모델을 학습시키는 것이 가능하다. 이는 DualAnoDiff의 구조적 강점과 실제 산업 시스템에서 요구되는 확장성을 결합하는 길이 될 것이다.

이 외에도 다음과 같은 구체적인 연구 방향을 제시할 수 있다:

- **효율성 및 실시간 적용:** 복잡한 DualAnoDiff 모델의 지식을 더 작고 빠른 '학생' 모델로 증류(distill)하는 연구가 필요하다.23 이는 연산 비용 한계를 해결하고 실제 배포 가능성을 높일 것이다.
- **다른 데이터 양식으로의 확장:** 이중 상호연관 생성이라는 핵심 개념은 다른 데이터 유형에도 매우 효과적으로 적용될 수 있다.
  - **비디오 이상 생성:** 한 분기는 전체 비디오를, 다른 분기는 시공간적 '이상 튜브(anomaly tube)'를 생성하여 시간적 일관성을 보장하는 방식으로 확장할 수 있다.
  - **3D 이상 생성:** 3D 포인트 클라우드나 메시에 적용하여, 한 분기는 객체의 전체 형태를, 다른 분기는 부피 또는 표면 결함을 생성하도록 할 수 있다.
- **생성에서 제로샷 탐지로의 연결:** 수천 개의 이미지를 생성하고 별도의 탐지기를 학습시키는 대신, 전역 및 이상 분기가 학습한 풍부한 특징(feature)을 *직접* 이상 탐지에 활용할 수 있는지 탐구하는 것도 야심 찬 연구 방향이다. 이는 생성과 탐지 패러다임을 통합하여, 매우 효율적인 소수샷 또는 제로샷 이상 탐지 시스템으로 이어질 수 있다.


1. DualAnoDiff: Dual-Interrelated Diffusion Model for Few-Shot Anomaly Image Generation | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/383428302_DualAnoDiff_Dual-Interrelated_Diffusion_Model_for_Few-Shot_Anomaly_Image_Generation
2. DualAnoDiff: Dual-Interrelated Diffusion Model for Few-Shot Anomaly Image Generation, accessed July 19, 2025, [https://openreview.net/forum?id=ExbicMba1Z&referrer=%5Bthe%20profile%20of%20Jiafu%20Wu%5D(%2Fprofile%3Fid%3D~Jiafu_Wu3)](https://openreview.net/forum?id=ExbicMba1Z&referrer=[the+profile+of+Jiafu+Wu](/profile?id%3D~Jiafu_Wu3))
3. [2408.13509] Dual-Interrelated Diffusion Model for Few-Shot Anomaly Image Generation - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2408.13509
4. DefectFill: Realistic Defect Generation with Inpainting Diffusion Model for Visual Inspection - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2025/papers/Song_DefectFill_Realistic_Defect_Generation_with_Inpainting_Diffusion_Model_for_Visual_CVPR_2025_paper.pdf
5. DualAnoDiff: Dual-Interrelated Diffusion Model for Few-Shot Anomaly Image Generation - GitHub, accessed July 19, 2025, https://github.com/yinyjin/DualAnoDiff
6. Data-centric anomaly detection with diffusion models - Amazon Science, accessed July 19, 2025, https://www.amazon.science/publications/data-centric-anomaly-detection-with-diffusion-models
7. DualAnoDiff: Dual-Interrelated Diffusion Model for Few-Shot Anomaly Image Generation, accessed July 19, 2025, https://arxiv.org/html/2408.13509v2
8. Exploiting Multimodal Latent Diffusion Models for Accurate Anomaly Detection in Industry 5.0 - CEUR-WS.org, accessed July 19, 2025, https://ceur-ws.org/Vol-3762/519.pdf
9. A Survey on Diffusion Models for Anomaly Detection - arXiv, accessed July 19, 2025, https://arxiv.org/html/2501.11430v3
10. [Literature Review] Anomaly Detection and Generation with ..., accessed July 19, 2025, https://www.themoonlight.io/en/review/anomaly-detection-and-generation-with-diffusion-models-a-survey
11. D3N: Bring the Power of Diffusion Model to Defect Detection - arXiv, accessed July 19, 2025, https://arxiv.org/html/2408.13845v1
12. A Survey on Diffusion Models for Anomaly Detection - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/388231913_A_Survey_on_Diffusion_Models_for_Anomaly_Detection
13. On Diffusion Modeling for Anomaly Detection - OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=lR3rk7ysXz
14. A Few-Shot Steel Surface Defect Generation Method Based on ..., accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12115093/
15. [Literature Review] DualAnoDiff: Dual-Interrelated Diffusion Model ..., accessed July 19, 2025, https://www.themoonlight.io/en/review/dualanodiff-dual-interrelated-diffusion-model-for-few-shot-anomaly-image-generation
16. Dual-Interrelated Diffusion Model for Few-Shot Anomaly Image Generation - CVF Open Access, accessed July 19, 2025, https://openaccess.thecvf.com/content/CVPR2025/papers/Jin_Dual-Interrelated_Diffusion_Model_for_Few-Shot_Anomaly_Image_Generation_CVPR_2025_paper.pdf
17. [Literature Review] MAGIC: Mask-Guided Diffusion Inpainting with Multi-Level Perturbations and Context-Aware Alignment for Few-Shot Anomaly Generation - Moonlight | AI Colleague for Research Papers, accessed July 19, 2025, https://www.themoonlight.io/en/review/magic-mask-guided-diffusion-inpainting-with-multi-level-perturbations-and-context-aware-alignment-for-few-shot-anomaly-generation
18. Dual-Interrelated Diffusion Model for Few-Shot Anomaly Image Generation - arXiv, accessed July 19, 2025, https://arxiv.org/html/2408.13509v3
19. (PDF) Anodapter: A Unified Framework for Generating Aligned Anomaly Images and Masks Using Diffusion Models - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/391691367_Anodapter_A_Unified_Framework_for_Generating_Aligned_Anomaly_Images_and_Masks_Using_Diffusion_Models
20. arXiv:2408.13509v1 [cs.CV] 24 Aug 2024, accessed July 19, 2025, http://www.arxiv.org/pdf/2408.13509v1
21. Issues / yinyjin/DualAnoDiff / GitHub, accessed July 19, 2025, https://github.com/yinyjin/DualAnoDiff/issues
22. UniCombine: Unified Multi-Conditional Combination with Diffusion Transformer - arXiv, accessed July 19, 2025, https://arxiv.org/html/2503.09277v2
23. [2408.13845] Bring the Power of Diffusion Model to Defect Detection - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2408.13845
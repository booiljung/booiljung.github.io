# FiTv2 모델

## 서론: 생성 모델의 해상도 유연성 재정의

기존 생성 모델, 특히 강력한 성능을 자랑하는 확산 트랜스포머(Diffusion Transformers, DiT)는 '고정 해상도 그리드의 폭정(tyranny of the fixed-resolution grid)'이라는 근본적인 한계에 직면해 있었다. 이들 모델은 256x256 또는 512x512와 같은 특정 해상도에 맞춰 학습되고 최적화된다. 이로 인해 학습 분포를 벗어나는 해상도나 화면 비율의 이미지를 생성할 때 품질이 급격히 저하되는, 이른바 '해상도 외삽(resolution extrapolation)' 문제가 발생한다.5

이러한 한계에 대한 근본적인 도전은 "**자연은 무한히 해상도로부터 자유롭다(Nature is infinitely resolution-free)**"는 철학적 전환에서 시작된다.5 이 개념은 기존 모델의 고정 그리드 방식과 실제 세계 시각 데이터의 가변적 특성 사이의 불일치를 정확히 지적한다. FiTv2는 이러한 한계를 정면으로 돌파하기 위해 설계된 기념비적인 모델이다. 이는 단순한 점진적 개선이 아닌, 제한 없는 해상도와 화면 비율의 이미지 생성을 위해 처음부터 새롭게 설계된 확장 가능하고 개선된 아키텍처이다.4 FiTv2는 Query-Key 벡터 정규화, AdaLN-LoRA 모듈, Rectified Flow 스케줄러, 그리고 Logit-Normal 샘플러라는 네 가지 핵심 혁신을 통해 이러한 기능적 도약을 가능케 했다.4

FiTv2의 등장은 생성형 AI 분야에서 '하나의 모델, 하나의 해상도'라는 지배적인 철학에서 벗어나는 근본적인 패러다임 전환을 의미한다. 이미지를 고정된 그리드가 아닌 유연한 토큰 시퀀스로 개념화함으로써, 모델 아키텍처를 데이터의 공간적 차원으로부터 분리시켰다. 기존 DiT와 같은 모델들은 고정된 수의 토큰을 위해 설계된 위치 인코딩과 어텐션 메커니즘으로 인해 학습 해상도에 본질적으로 묶여 있었다.5 추론 시 해상도가 달라지면 '외삽'이 필요했고, 이는 종종 품질 저하로 이어졌다.9 FiT/FiTv2 연구진은 자연 이미지의 다양한 해상도와 화면 비율에서 영감을 받아 이러한 제약을 깨뜨리는 것을 핵심 동기로 삼았다.6 이는 단순한 기술적 개선을 넘어선 설계 철학의 변화이다. 모델은 더 이상 특정 해상도를 

*위해* 만들어지는 것이 아니라, 해상도에 구애받지 않도록(resolution-agnostic) 설계된다. 이러한 변화는 실용적이고 경제적인 측면에서 막대한 파급 효과를 가진다. 웹 콘텐츠, 인쇄 매체, 비디오 썸네일 등 다양한 형식의 이미지를 생성해야 하는 모든 애플리케이션에서, 각기 다른 해상도에 맞춰 학습된 여러 전문 모델들을 단 하나의 강력한 FiTv2 모델로 대체할 수 있게 된다. 이는 학습 비용, 모델 관리 오버헤드, 추론 인프라의 복잡성을 극적으로 감소시키는 결과로 이어진다.

## 1. 기반 기술: 확산 트랜스포머에서 유연한 비전 트랜스포머(FiT)까지

### 1.1 확산 트랜스포머(DiT) 아키텍처

FiT가 구축된 기반이자 당시 최첨단 기술이었던 확산 트랜스포머(DiT) 아키텍처를 이해하는 것은 필수적이다.5 DiT는 이전 확산 모델에서 널리 사용되던 U-Net 백본을 순수 트랜스포머 아키텍처로 대체함으로써, 뛰어난 확장성과 성능을 입증하며 새로운 표준을 제시했다.9 하지만 DiT는 명확한 약점을 가지고 있었다. 바로 학습 과정에서 동적 해상도의 이미지를 통합할 수 없어, 학습된 영역 밖의 해상도에 대한 일반화 성능이 현저히 떨어진다는 점이다.5

### 1.2 FiT(v1)의 패러다임 전환: 이미지를 토큰 시퀀스로

최초의 유연한 비전 트랜스포머(FiT)는 DiT의 한계를 극복하기 위해 개념적 돌파구를 제시했다.

**핵심 아이디어**: DiT의 정적인 그리드 인식과 달리, FiT는 이미지를 동적 크기의 토큰 시퀀스로 개념화했다.5 이 접근법을 통해 다양한 화면 비율의 이미지를 가변 길이의 토큰 시퀀스로 변환하고, 이를 일괄 처리(batch processing)를 위해 최대 길이에 맞춰 패딩(padding)할 수 있게 되었다.7 이는 데이터 전처리 과정에서 이미지를 왜곡시키는 잘라내기(cropping)나 부자연스러운 크기 조정을 제거하는 효과를 가져왔다.7

**구현 기술**: FiT는 이러한 유연성을 구현하기 위해 DiT의 아키텍처를 변형하여 몇 가지 핵심 요소를 도입했다.

- **2차원 회전 위치 임베딩(2-D Rotary Positional Embedding, 2-D RoPE)**: 2D 공간에서 토큰의 위치를 인코딩하여 가변 길이 시퀀스를 효과적으로 처리하는 데 결정적인 역할을 했다.5
- **Swish-Gated Linear Units (SwiGLU)**: 기존의 다층 퍼셉트론(MLP) 대신 사용하여 성능을 향상시켰다.5
- **마스크된 멀티헤드 셀프 어텐션(Masked Multi-Head Self-Attention)**: 가변 길이 시퀀스에서 패딩된 토큰들이 어텐션 계산을 오염시키지 않도록 효율적으로 관리하는 데 필수적이었다.5

### 1.3 FiT(v1)의 한계: 뛰어나지만 미완의 초안

FiT는 혁신적인 개념을 제시했지만, FiTv2의 개발을 촉발한 몇 가지 명백한 단점을 가지고 있었다.

- **성능 부족**: 표준 ImageNet 256x256 벤치마크에서 DiT에 비해 저조한 성능을 보였다.5
- **효율성 문제**: DiT에 비해 파라미터 수와 계산 비용이 증가했다.5
- **학습 불안정성**: 초기 아키텍처는 학습 안정성 문제를 드러내어 실제 구현에 어려움을 겪었다.8

이러한 배경은 FiTv2가 단순한 'v2' 업그레이드가 아님을 시사한다. 오히려 FiTv2는 FiT v1의 실용적 결함에 대한 포괄적인 *수정* 작업으로 이해하는 것이 더 정확하다. FiT는 '유연한 토큰화'라는 혁명적 개념을 도입했지만, 최첨단 성능과 효율성을 제공하는 데는 실패했다. FiTv2의 네 가지 핵심 혁신은 이 독창적인 개념을 실용적이고 안정적이며 기존 모델보다 우월하게 만들기 위한 직접적이고 표적화된 대응이었다. FiT 원본 논문(arXiv:2402.12376)이 2024년 2월에 발표되고, FiTv2 논문(arXiv:2410.13925)이 같은 해 10월에 텍스트 중복을 명시하며 빠르게 뒤따랐다는 사실은 이러한 신속한 반복 개선 과정을 방증한다.6 FiTv2 논문은 이례적으로 이전 버전의 실패(벤치마크 성능 저하, 높은 비용, 학습 불안정성)를 솔직하게 인정하며 5, 각 혁신이 이러한 결함을 어떻게 해결하는지를 명확히 보여준다. 이는 AI 연구에서 획기적인 이론적 아이디어만으로는 충분하지 않으며, 실용적인 구현, 안정성, 그리고 기존 벤치마크에서의 경쟁력이 채택과 진정한 발전을 이끈다는 중요한 교훈을 남긴다. FiTv2의 성공은 FiT 개념의 현실적인 문제들을 해결하기 위한 세심한 엔지니어링에 있다.

## 2. 아키텍처 심층 분석: FiTv2의 혁신 기술

이 섹션에서는 FiTv2를 정의하는 네 가지 기술적 기둥을 각각 해부하여 문제점, 메커니즘, 그리고 이점을 상세히 설명한다.

### 2.1  Query-Key(QK) 벡터 정규화를 통한 어텐션 안정화

- **문제점**: 트랜스포머에서 Query(Q)와 Key(K) 벡터의 내적(dot product) 값은 때때로 매우 커질 수 있다. 이는 후속 Softmax 함수를 포화(saturation) 영역으로 밀어 넣어 그래디언트 소실(vanishing gradients) 문제를 야기하며, 이는 학습을 저해하고 훈련 불안정성을 초래한다.18 이는 FiT v1의 불안정성 문제의 주요 원인 중 하나로 추정된다.
- **메커니즘**: FiTv2는 **QK-Norm**을 도입했다. 이는 내적 계산 전에 각 Q와 K 벡터에 대해 `$L_2$` 정규화를 적용하는 방식이다. 이를 통해 Softmax 함수의 입력값이 잘 제어되고 유계(bounded)가 되도록 보장한다. 동시에, 모델의 표현력을 유지하기 위해 학습 가능한 스케일링 파라미터가 도입된다.18
- **이점**: 대규모 심층 트랜스포머 모델을 성공적으로 훈련시키기 위한 전제 조건인 학습 안정성과 일관된 그래디언트 흐름이 크게 개선되었다.18

### 2.2  적응형 계층 정규화와 저계수 적응(AdaLN-LoRA)을 통한 효율적인 조건화

- **문제점**: 확산 모델을 타임스텝이나 클래스 레이블과 같은 정보에 조건화하는 것은 매우 중요하다. 표준적인 방법인 적응형 계층 정규화(Adaptive Layer Normalization, AdaLN)는 스케일 및 시프트 파라미터를 예측해야 하는데, 이는 상당한 수의 학습 가능한 파라미터를 추가하여 계산 비용과 메모리 사용량을 증가시킨다.19 이는 FiT v1의 비효율성의 한 원인이었다.
- **메커니즘**: FiTv2는 **AdaLN-LoRA**를 도입했다. 이는 전체 조건부 투영 레이어를 훈련하는 대신, 사전 훈련된 가중치를 동결하고 작고 학습 가능한 "저계수(low-rank)" 행렬(LoRA)을 주입하는 방식이다.20 이 경량 어댑터들은 AdaLN 레이어의 동작을 효과적으로 조절하기에 충분하다.7
- **이점**: 조건화를 위해 필요한 학습 가능한 파라미터 수가 극적으로 감소하여, 성능 저하 없이 더 높은 파라미터 효율성, 감소된 메모리 오버헤드, 그리고 더 빠른 훈련 속도를 달성했다. 이는 FiTv2가 FiT 및 DiT에 비해 효율성이 개선된 핵심 요인 중 하나이다.7

### 2.3  Rectified Flow 스케줄러를 통한 수렴 가속화

- **문제점**: FiT v1에서 사용된 DDPM과 같은 표준 확산 모델은 노이즈를 이미지로 변환하는 복잡하고 구부러진 경로를 학습한다. 이 곡선을 정확하게 근사하기 위해 추론 시 많은 작은 단계가 필요하며, 이는 생성 속도를 느리게 만든다. 또한 이 구부러진 경로를 학습하는 과정 자체도 비효율적일 수 있다.24
- **메커니즘**: FiTv2는 DDPM 스케줄러를 **Rectified Flow** 스케줄러로 대체했다.5 Rectified Flow는 모델이 노이즈 지점과 데이터 지점 사이의 직선 경로를 명시적으로 학습하도록 훈련시킨다.24 직선은 큰 오차 없이 더 적고 큰 단계로 이동할 수 있기 때문에 생성 과정이 단순화된다.
- **이점**: 훈련 중 수렴 속도가 극적으로 빨라졌으며(FiT보다 2배 빠르다고 보고됨), 매우 적은 단계(경우에 따라 단일 단계)로 고품질 이미지를 생성할 수 있어 추론 속도 또한 크게 향상되었다.5 이는 FiT v1의 느린 수렴 문제를 직접적으로 해결하고 전반적인 유용성을 개선했다.

### 2.4  Logit-Normal 샘플러를 통한 타임스텝 샘플링 최적화

- **문제점**: 확산 모델 훈련 중, 각 훈련 예제에 대해 타임스텝 `$t$`가 구간 `$$`에서 샘플링된다. `$t$`가 샘플링되는 분포는 훈련 효율성에 영향을 미칠 수 있다. 단순한 균등 샘플링은 최적이 아닐 수 있는데, 확산 과정의 여러 단계(예: 초기 노이즈 제거 대 후기 미세 디테일 형성)가 서로 다른 양의 훈련 집중을 필요로 할 수 있기 때문이다.
- **메커니즘**: FiTv2는 타임스텝에 대해 **Logit-Normal 샘플러**를 사용한다.5 Logit-Normal 분포는 (0, 1) 사이로 제한된 확률 변수에 대해 정의된다.32 이 분포에서 샘플링함으로써, 모델은 확산 타임라인의 특정 부분, 즉 학습에 더 중요한 부분에 더 집중하도록 훈련될 수 있다. 변수의 로짓(logit)이 정규 분포를 따르므로, 균등 추출보다 더 구조화된 샘플링이 가능하다.
- **이점**: 더 원칙적이고 효율적인 훈련이 가능해졌다. 노이즈 스케줄의 더 정보적인 부분에 훈련 단계를 집중시킴으로써, 모델은 더 효율적으로 학습할 수 있으며, 이는 전반적인 수렴 속도 향상과 최종 성능 개선에 기여한다.5

## 3. 성능 평가 및 비교 분석

이 섹션에서는 FiTv2의 성능을 정량적으로 종합하고, 주요 경쟁 모델과의 벤치마크 비교를 통해 그 우수성을 입증한다.

### 3.1  분포 내(In-Distribution) 벤치마크 성능 (ImageNet 256x256)

이 하위 섹션은 FiTv2의 전신인 FiT가 어려움을 겪었던 표준 고정 해상도 벤치마크에서의 성능에 초점을 맞춘다. 이는 FiTv2의 새로운 혁신들이 모델을 매우 경쟁력 있게 만들었음을 보여준다.

**표 1: ImageNet 256x256 클래스 조건부 생성 성능 비교**

| 모델                | FID (↓)          | 훈련 스텝 (K) | 파라미터 (B) | 출처 |      |      |
| ------------------- | ---------------- | ------------- | ------------ | ---- | ---- | ---- |
| DiT-XL/2            | 기준             | 7,000         | 0.675        | 35   |      |      |
| SiT-XL/2            | 기준             | 7,000         | 0.675        | 35   |      |      |
| **FiTv2-XL/2**      | **DiT/SiT 능가** | **2,000**     | **0.675**    |      | 5    |      |
| **FiTv2-3B/2**      | 경쟁력 있는 성능 | 1,000         | 3.0          | 5    |      |      |
| DoD-XL (FiTv2 기반) | **1.83**         | 1,000         | -            | 35   |      |      |

이 표는 FiTv2가 표준 벤치마크에서 단순히 경쟁력 있는 것을 넘어, 효율성 측면에서 기존의 최첨단 모델들을 압도함을 명확히 보여준다. FiTv2-XL/2 모델은 동일한 파라미터 수를 가지면서도 DiT 및 SiT 모델 훈련 비용의 약 28.6% (7,000K 스텝 대비 2,000K 스텝)만으로 더 우수한 성능을 달성했다.5 또한, FiTv2 아키텍처를 기반으로 한 DoD-XL 모델은 단 1백만 훈련 스텝만으로 1.83이라는 최상위권 FID 점수를 기록하여, FiTv2 백본의 강력한 잠재력을 입증했다.35

### 3.2  분포 외(Out-of-Distribution) 및 해상도 외삽 성능

이 하위 섹션은 FiTv2의 핵심 강점, 즉 학습 데이터 분포를 훨씬 벗어나는 임의의 해상도 및 화면 비율에서 고품질 이미지를 생성하는 능력을 분석한다.

**표 2: 해상도 외삽 성능 비교 (FID 점수)**

| 모델                   | 160x320       | 128x384       | 320x640       | 256x768       | 512x512       |
| ---------------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| DiT-XL/2 (보간법 적용) | 기준 성능     | 성능 저하     | 성능 저하     | 성능 저하     | 성능 저하     |
| SiT-XL/2 (보간법 적용) | 기준 성능     | 성능 저하     | 성능 저하     | 성능 저하     | 성능 저하     |
| **FiTv2-XL/2 & 3B/2**  | **SOTA 능가** | **SOTA 능가** | **SOTA 능가** | **SOTA 능가** | **SOTA 능가** |

이 표는 FiTv2의 주요 기여와 독보적인 장점을 명확히 보여준다. 표 1이 FiTv2가 표준 과제에서 *경쟁력 있음*을 증명했다면, 이 표는 FiTv2가 설계된 목표, 즉 유연한 해상도 생성에서 *우월함*을 증명한다. 이는 '해상도 폭정의 종식'을 시각적으로 보여주는 결과이다. 연구에 따르면 FiTv2는 이러한 다양한 비표준 해상도에서 기존의 모든 최첨단 모델들을 "상당한 격차로" 능가하는 성능을 보였다.5 특히, 200K~400K 스텝의 짧은 사후 미세조정 단계를 거치면 

`$H \times W \le 512 \times 512$`와 같은 고해상도 이미지에서의 성능이 더욱 향상되는 것으로 나타났다.5 이는 FiTv2가 진정한 의미의 해상도 독립성을 달성했음을 시사한다.

## 4. 확장성, 효율성 및 학술적 영향

### 4.1 확장성 분석: 클수록 더 좋고 효율적이다

FiTv2는 30억 파라미터 모델(FiTv2-3B/2)까지 성공적으로 확장되었다.5 이 과정에서 직관에 반하지만 매우 중요한 발견이 있었다. 바로 더 큰 FiTv2 모델이 *더 나은* 계산 효율성을 보인다는 것이다. 이는 아키텍처의 고정 오버헤드가 규모가 커질수록 더 잘 상각(amortized)됨을 시사한다.6

### 4.2 최우선 가치로서의 효율성

FiTv2의 설계는 효율성을 핵심 가치로 삼았으며, 이는 다음과 같은 구체적인 성과로 나타났다.

- **훈련 비용 절감**: FiTv2-XL/2는 DiT/SiT-XL/2보다 우수한 결과를 단 28.6%의 훈련 스텝(즉, 약 70% 비용 절감)으로 달성했다.5 이는 기념비적인 개선이다.
- **수렴 속도**: Rectified Flow 스케줄러와 기타 최적화 덕분에 FiTv2는 이전 버전인 FiT보다 2배 빠르게 수렴한다.5

### 4.3 학술적 및 후속 연구에 미친 영향

FiTv2는 그 자체의 성능을 넘어, 후속 연구를 위한 기반 아키텍처로서 그 가치를 입증하며 학계에 상당한 영향을 미쳤다.

- **Diffusion on Diffusion (DoD)**: 이 모델은 FiTv2를 백본으로 사용하여 시각적 사전 정보(visual priors)를 활용하는 새로운 다단계 생성 프레임워크를 구현했다. DoD 연구팀이 FiTv2를 선택한 이유는 이미 입증된 효율성과 고품질 생성 능력 덕분이었을 것이며, 이를 통해 기반 모델을 재설계하는 대신 새로운 프레임워크 자체에 집중할 수 있었다.16 DoD는 DiT보다 7배 적은 훈련 비용으로 1.83의 FID를 달성했는데, 이는 FiTv2 기반의 강력함을 증명한다.35
- **Length-Extrapolatable Diffusion Transformer (LEDiT)**: 이 연구는 FiT/FiTv2를 명시적으로 인용하고 비교하며, 해상도 외삽 문제를 다른 방식(명시적 위치 인코딩 제거)으로 해결하고자 했다. LEDiT의 등장은 FiTv2가 해당 연구 분야의 핵심 문제와 새로운 모델들이 경쟁하거나 개선해야 할 고성능 벤치마크를 정의했음을 보여준다.9

FiTv2의 네 가지 핵심 업그레이드는 분리할 수 없는 단일체가 아니라, 차세대 확산 트랜스포머를 구축하기 위한 모듈식 "모범 사례" 툴킷을 제시한다. QK-Norm, LoRA, Rectified Flow와 같은 각 구성 요소는 독립적인 연구 흐름을 가진 기술들이다.21 FiTv2의 핵심적인 아키텍처 기여는 이러한 최첨단이지만 서로 다른 기술들을 

*통합*하여, 응집력 있고 고성능인 시스템을 만든 데 있다. 이러한 접근 방식은 FiTv2 논문을 다른 연구자들에게 귀중한 "요리책(cookbook)"으로 만든다. 예를 들어, 새로운 비디오 확산 모델을 구축하는 팀은 FiTv2의 AdaLN-LoRA를 효율적인 조건화를 위해, Rectified Flow 스케줄러를 빠른 훈련을 위해 채택할 수 있다. 이러한 모듈성은 DoD와 같은 후속 모델에 대한 학술적 영향력의 핵심 이유이다.

## 5. 결론 및 미래 전망

### 요약 및 기여

FiTv2의 주요 성과는 다음과 같이 요약할 수 있다.

1. 진정한 해상도 독립적 이미지 생성을 위해 "유연한 토큰화" 개념을 성공적으로 실용화했다.
2. 이전 모델들의 계산 비용의 일부만으로 최첨단 결과를 달성하며, 아키텍처 효율성의 새로운 패러다임을 제시했다.
3. 이미 차세대 생성 모델에 영향을 미치고 있는 안정적이고 확장 가능한 강력한 오픈소스 기반을 제공했다.38

### 5.1 미래 전망: 정지 이미지를 넘어서

FiTv2가 개척한 원칙들, 즉 시공간 데이터의 유연한 토큰화와 AdaLN-LoRA와 같은 효율적이고 적응적인 조건화는 정지 이미지에만 국한되지 않는다. 이러한 개념들은 더 복잡한 데이터 양식에 직접 적용될 수 있으며, 다음과 같은 분야의 미래를 형성할 가능성이 높다.

- **텍스트-비디오 생성**: W.A.L.T.나 Cosmos-1.0과 같은 모델들은 이미 시공간 잠재 공간과 AdaLN-LoRA와 같은 효율적인 조건화 메커니즘을 갖춘 유사한 트랜스포머 기반 확산 아키텍처를 사용하고 있다.19 FiTv2의 성공은 가변 길이 비디오 클립과 다양한 프레임 속도를 처리하기 위한 강력한 청사진을 제공한다.
- **멀티모달 파운데이션 모델**: 모델이 텍스트, 이미지, 오디오, 비디오의 혼합을 이해하고 생성해야 하는 시대가 도래함에 따라, 다양한 유형과 길이의 시퀀스를 유연하게 토큰화하고 처리하는 능력은 무엇보다 중요해질 것이다. FiTv2의 아키텍처는 그 방향으로 나아가는 중요한 진일보이다.

결론적으로, FiTv2는 특정 기술적 문제를 해결한 것을 넘어, 생성 모델의 설계 철학을 근본적으로 바꾸고, 효율성과 유연성을 최우선 가치로 끌어올린 변곡점과 같은 모델이다. 그 영향력은 현재의 이미지 생성 작업을 넘어, 미래의 멀티모달 AI 시스템의 기반을 형성하는 데까지 미칠 것으로 전망된다.

#### **참고 자료**

1. fitvii: Best blood pressure smartwatch & fitness trackers you can trust., accessed July 19, 2025, https://fitvii.com/
2. FitVII H56 Fitness Tracker with Blood Pressure Heart Rate and Blood Oxygen Monitor Smartwatch, accessed July 19, 2025, https://fitvii.com/products/fitvii-h56-smartwatch-with-blood-pressure-monitoring-sport-modes
3. Upgrade FITVII® GT5 Blood Pressure Watch With Heart Rate Monitor for Senior, accessed July 19, 2025, https://fitvii.com/products/novae-black-ip68-waterproof-smart-watch
4. FiTv2: Scalable and Improved Flexible Vision Transformer for Diffusion Model | Papers With Code, accessed July 19, 2025, https://paperswithcode.com/paper/fitv2-scalable-and-improved-flexible-vision
5. FiTv2: Scalable and Improved Flexible Vision Transformer for Diffusion Model - arXiv, accessed July 19, 2025, https://arxiv.org/html/2410.13925v1
6. [2410.13925] FiTv2: Scalable and Improved Flexible Vision Transformer for Diffusion Model - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2410.13925
7. FiTv2: Scalable and Improved Flexible Vision Transformer for Diffusion Model - arXiv, accessed July 19, 2025, https://arxiv.org/html/2402.12376v2
8. (PDF) FiTv2: Scalable and Improved Flexible Vision Transformer for Diffusion Model, accessed July 19, 2025, https://www.researchgate.net/publication/385091099_FiTv2_Scalable_and_Improved_Flexible_Vision_Transformer_for_Diffusion_Model
9. LEDiT: Your Length-Extrapolatable Diffusion Transformer without Positional Encoding - arXiv, accessed July 19, 2025, [https://arxiv.org/pdf/2503.04344?](https://arxiv.org/pdf/2503.04344)
10. FiTv2: Scalable and Improved Flexible Vision Transformer for, accessed July 19, 2025, https://www.aimodels.fyi/papers/arxiv/fitv2-scalable-improved-flexible-vision-transformer-diffusion
11. [2402.12376] FiT: Flexible Vision Transformer for Diffusion Model - arXiv, accessed July 19, 2025, https://arxiv.org/abs/2402.12376
12. FiTv2: Scalable and Improved Flexible Vision Transformer for, accessed July 19, 2025, https://synthical.com/article/4ec4ae53-f3b9-462d-85dc-1f297756a0b1
13. FiTv2: Scalable and Improved Flexible Vision Transformer for Diffusion Model - OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=4dfBABxTZE
14. FiTv2: Scalable and Improved Flexible Vision Transformer for Diffusion Model - Consensus, accessed July 19, 2025, https://consensus.app/papers/fitv2-scalable-and-improved-flexible-vision-transformer-lu-ouyang/e3f38c67736b58b0af3841bc0aa4f5ce
15. FiTv2: Scalable and Improved Flexible Vision Transformer for Diffusion Model (2410.13925v1) - Emergent Mind, accessed July 19, 2025, https://www.emergentmind.com/papers/2410.13925
16. diffusion models need visual priors for image generation - arXiv, accessed July 19, 2025, [https://arxiv.org/pdf/2410.08531?](https://arxiv.org/pdf/2410.08531)
17. DiffiT: Diffusion Vision Transformers for Image Generation | Request PDF - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/385348773_DiffiT_Diffusion_Vision_Transformers_for_Image_Generation
18. Query-Key Normalization for Transformers - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/344603058_Query-Key_Normalization_for_Transformers
19. W.A.L.T.pdf - Photorealistic Video Generation with Diffusion Models, accessed July 19, 2025, https://walt-video-diffusion.github.io/assets/W.A.L.T.pdf
20. Adaptive LoRA (AdaLORA) paper explanation | by Astarag Mohapatra - Medium, accessed July 19, 2025, https://athekunal.medium.com/adaptive-lora-adalora-paper-explanation-7cb5ac04d0cb
21. What is LoRA (Low-Rank Adaption)? - IBM, accessed July 19, 2025, https://www.ibm.com/think/topics/lora
22. An Overview of the LoRA Family. LoRA, DoRA, AdaLoRA, Delta-LoRA, and... | by Dorian Drost | TDS Archive | Medium, accessed July 19, 2025, https://medium.com/data-science/an-overview-of-the-lora-family-515d81134725
23. Generate Realistic Videos with NVIDIA COSMOS 1.0 Diffusion - Analytics Vidhya, accessed July 19, 2025, https://www.analyticsvidhya.com/blog/2025/02/nvidia-cosmos-1-0-diffusion/
24. Stable Diffusion 3 - Explained. How Rectified Flow and Transformers... | by Pietro Bolcato, accessed July 19, 2025, https://medium.com/@pietrobolcato/stable-diffusion-3-explained-84fd085934cb
25. Rectified Flow - UT Computer Science, accessed July 19, 2025, https://www.cs.utexas.edu/~lqiang/rectflow/html/intro.html
26. Rectified Diffusion: Straightness Is Not Your Need in Rectified Flow | OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=nEDToD1R8M
27. Understanding InstaFlow/Rectified Flow - Hugging Face, accessed July 19, 2025, https://huggingface.co/blog/Isamu136/insta-rectified-flow
28. Rectified Diffusion: Straightness Is Not Your Need in Rectified Flow - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/384812316_Rectified_Diffusion_Straightness_Is_Not_Your_Need_in_Rectified_Flow
29. Flow Straight and Fast: Learning to Generate and Transfer Data with Rectified Flow | OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=XVjTT1nw5z
30. Daily Papers - Hugging Face, accessed July 19, 2025, [https://huggingface.co/papers?q=logit%20flow](https://huggingface.co/papers?q=logit+flow)
31. Efficient Diffusion Transformer with Step-Wise Dynamic Attention Mediators | Request PDF, accessed July 19, 2025, https://www.researchgate.net/publication/386048122_Efficient_Diffusion_Transformer_with_Step-Wise_Dynamic_Attention_Mediators
32. Logit-normal distribution - Wikipedia, accessed July 19, 2025, https://en.wikipedia.org/wiki/Logit-normal_distribution
33. pymc.LogitNormal - PyMC 5.23.0 documentation, accessed July 19, 2025, https://www.pymc.io/projects/docs/en/stable/api/distributions/generated/pymc.LogitNormal.html
34. Sample Size Determination for Logistic Regression on a Logit-Normal Distribution - PMC, accessed July 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC4560689/
35. DIFFUSION MODELS NEED VISUAL PRIORS FOR IMAGE GENERATION - OpenReview, accessed July 19, 2025, https://openreview.net/pdf/7415fc810d502243f9eae150b2aa40ebbb6faa5e.pdf
36. ImageNet 256x256 Benchmark (Image Generation) | Papers With Code, accessed July 19, 2025, https://paperswithcode.com/sota/image-generation-on-imagenet-256x256
37. Diffusion Models Need Visual Priors for Image Generation - ResearchGate, accessed July 19, 2025, https://www.researchgate.net/publication/384887425_Diffusion_Models_Need_Visual_Priors_for_Image_Generation
38. whlzy/FiT: [ICML 2024 Spotlight] FiT: Flexible Vision Transformer for Diffusion Model, accessed July 19, 2025, https://github.com/whlzy/FiT
39. FiTv2: Scalable and Improved Flexible Vision Transformer for ..., accessed July 19, 2025, https://consensus.app/papers/fitv2-scalable-and-improved-flexible-vision-transformer-lu-ouyang/e3f38c67736b58b0af3841bc0aa4f5ce/
40. Diffusion Models Need Visual Priors for Image Generation - OpenReview, accessed July 19, 2025, https://openreview.net/forum?id=WNb4P8aG66
41. Photorealistic Video Generation with Diffusion Models - European Computer Vision Association, accessed July 19, 2025, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/10270.pdf


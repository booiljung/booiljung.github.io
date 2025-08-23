# CricaVPR



시각적 장소 인식(Visual Place Recognition, VPR)은 주어진 쿼리 이미지의 시각 정보만을 이용해, 지리적 위치 정보가 태깅된 대규모 데이터베이스 내에서 동일한 장소를 촬영한 이미지를 검색하는 기술이다.1 본질적으로 이미지 검색(Image Retrieval) 문제로 정의되는 VPR은 자율주행 차량의 재위치 추정, 모바일 로봇의 측위, 증강현실(AR) 등 다양한 분야에서 핵심적인 역할을 수행한다.3 그러나 VPR 기술이 실제 환경에서 강인하게 작동하기 위해서는 세 가지 근본적인 난제를 극복해야 한다.

첫째, **환경 변화(Condition Variations)** 문제다. 동일한 장소일지라도 조명(주간/야간), 날씨(맑음/흐림/비), 계절(봄/여름/가을/겨울)의 변화에 따라 이미지의 외형이 극적으로 달라질 수 있다.6

둘째, **시점 변화(Viewpoint Variations)** 문제로, 같은 장소를 다른 각도나 거리에서 촬영할 경우 발생하는 외형적 차이를 의미한다.8

셋째, **지각적 모호성(Perceptual Aliasing)** 문제다. 이는 서로 다른 장소임에도 불구하고 건물이나 도로의 형태가 매우 유사하여 시각적으로 구분이 어려운 경우를 지칭한다.6 이러한 세 가지 난제는 특히 장소의 전체적인 모습을 하나의 벡터로 압축하는 전역 특징(Global Feature) 기반 방법론에서 성능 저하의 주된 원인으로 작용한다.9


VPR 연구의 기술적 패러다임은 최근 몇 년간 중대한 전환을 맞이했다. 과거 NetVLAD나 GeM과 같은 CNN 기반의 특징 집계 방식이 주를 이루었다면, 현재는 트랜스포머(Transformer) 아키텍처, 특히 DINOv2와 같은 거대 비전 모델(Visual Foundation Models, VFM)이 연구의 중심축으로 자리 잡았다.10 DINOv2는 방대한 양의 비정형 데이터로 사전 훈련되어, 특정 작업에 국한되지 않는 강력한 일반화 성능과 시맨틱(semantic) 이해 능력을 갖추고 있다.13 이는 VPR의 고질적인 난제들을 해결할 새로운 가능성을 제시했다.

하지만 사전 훈련된 VFM을 VPR 작업에 그대로 적용하는 것(vanilla DINOv2)은 한계가 명확하다. 이러한 모델들은 VPR과 무관한 동적인 전경 객체(차량, 보행자)에 과도하게 집중하는 경향이 있으며, 장소를 구별하는 데 중요한 정적인 배경(건물, 구조물)을 간과할 수 있다.1 따라서 VFM의 강력한 사전 지식을 유지하면서도 VPR 작업의 특수성에 맞게 모델을 '적응(Adaptation)'시키거나 '미세조정(Fine-tuning)'하는 과정이 필수적이다.

이러한 기술적 배경은 VPR 분야의 연구 경쟁 구도를 근본적으로 바꾸었다. 최상위 학회에 발표되는 다수의 최신 SOTA 모델들(CricaVPR, SelaVPR, EffoVPR, SciceVPR 등)이 공통적으로 DINOv2를 백본으로 채택함에 따라 10, 연구의 핵심은 더 이상 새로운 특징 추출기(feature extractor)를 설계하는 데 있지 않다. 대신, **어떻게 DINOv2의 일반적 특징(general representation)을 VPR의 특수성(domain-specific nuances)에 가장 효과적으로 적응시키는가**에 대한 '적응 전략(adaptation strategy)'의 혁신으로 경쟁의 장이 옮겨갔다. CricaVPR, EffoVPR, SciceVPR 등은 모두 DINOv2라는 동일한 출발점에서 각기 다른 적응 철학과 방법론을 제시하며 이 새로운 경쟁 국면을 주도하고 있다. CricaVPR의 학술적 가치는 바로 이 '적응 전략' 경쟁의 맥락에서 평가되어야 한다.



CricaVPR이 해결하고자 한 문제의 핵심에는 기존 VPR 방법론들의 근본적인 한계가 자리 잡고 있다. 대부분의 VPR 네트워크는 입력된 단일 이미지만을 사용하여 해당 장소의 전역 표현을 독립적으로 생성한다.2 이러한 '고립된(isolated)' 처리 방식은 모델이 학습 과정에서 이미지 간에 발생하는 다양한 변화, 즉 시점이나 조명의 차이를 명시적으로 고려할 기회를 원천적으로 차단한다.1 결과적으로, 이렇게 생성된 특징 벡터는 실제 환경에서 마주하는 극심한 외형 변화에 취약할 수밖에 없다는 것이 CricaVPR의 문제의식이다.


이러한 한계를 극복하기 위해 CricaVPR은 훈련 과정의 기본 단위인 미니배치(mini-batch)를 바라보는 관점 자체를 전환한다. CricaVPR은 배치 내에 존재하는 여러 이미지를 단순히 독립적인 학습 샘플의 집합으로 간주하지 않는다. 대신, 이를 서로의 특징을 상호 보강하고 강인하게 만드는 데 활용할 수 있는 '상관관계 정보의 원천'으로 재정의한다.2

훈련 배치에는 동일한 장소를 다른 조건(예: 낮과 밤)이나 다른 시점에서 촬영한 이미지들이 포함될 수 있으며, 동시에 전혀 다른 장소의 이미지들도 섞여 있다. CricaVPR은 이러한 이미지들 간의 복합적인 '유사성'과 '차이' 자체를 학습의 중요한 단서(cue)로 삼는다.1 교차-이미지 인코더(Cross-image Encoder)라는 핵심 모듈을 통해 배치 내 모든 이미지의 특징 표현들이 서로 상호작용하도록 설계함으로써, 최종적으로 생성되는 특징 벡터가 환경 및 시점 변화에 불변(invariant)하고 지각적 모호성에 강하도록 유도한다.2

이러한 접근 방식은 딥러닝 훈련의 기본 가정 중 하나인 'I.I.D. (Independent and Identically Distributed) 가정'에 대한 암묵적인 도전으로 해석될 수 있다. 전통적인 지도 학습에서 미니배치 내의 각 데이터 샘플은 독립적으로 처리되어 손실(loss)을 계산하고, 이들의 평균이 모델 파라미터를 업데이트하는 데 사용된다. 이는 샘플 간의 상호 독립성을 전제한다. 그러나 CricaVPR의 교차-이미지 인코더는 어텐션 메커니즘을 통해 배치 내 모든 이미지 특징 벡터들 간의 상호작용을 계산한다.8 이는 이미지 A의 최종 특징이 배치 내에 함께 존재하는 다른 이미지 B, C, D 등의 문맥에 의해 동적으로 변화함을 의미한다. 따라서 모델은 $f(\text{이미지}_A)$라는 고정된 함수가 아닌, $f(\text{이미지}_A \mid \text{배치 문맥})$이라는 문맥 의존적인(context-dependent) 함수를 학습하게 된다. 이는 특징 공간 상에서 샘플 간의 독립성을 명시적으로 깨뜨리는 설계로, 단순히 쌍(pair)이나 삼중항(triplet) 관계에만 집중하는 대조 학습(contrastive learning)을 넘어, 배치 전체의 N-way 상호작용을 고려하는 새로운 훈련 패러다임을 제시했다는 점에서 철학적 전환을 의미한다.


CricaVPR의 핵심 철학은 정교하게 설계된 아키텍처를 통해 구현된다. 이 아키텍처는 거대 비전 모델의 활용, 효율적인 전이 학습, 그리고 관계형 추론이라는 현대 컴퓨터 비전의 세 가지 핵심 패러다임을 VPR 문제 해결을 위해 독창적으로 융합한 결과물이다.


CricaVPR은 백본 네트워크로 DINOv2(ViT-B/14)를 채택한다.8 VFM이 가진 강력한 일반화 능력을 최대한 보존하기 위해, 사전 훈련된 대부분의 파라미터는 훈련 과정에서 동결(freeze)시킨다. 대신, 파라미터 효율적 전이 학습(Parameter-Efficient Transfer Learning, PETL) 기법을 적용하여, 소수의 학습 가능한(trainable) 경량 어댑터(adapter) 모듈만을 모델에 삽입한다.6 이 전략은 VFM 전체를 미세조정하는 것에 비해 훈련 시간과 메모리 사용량을 획기적으로 줄여주며, 이는 논문에서 주장하는 "significantly less training time"을 가능하게 하는 핵심 기술이다.1


CricaVPR이 제안하는 가장 독창적인 기술 요소 중 하나는 바로 MulConv 어댑터다. 이는 기존의 자연어 처리 분야에서 유래한 범용 어댑터들과 달리, VPR 작업의 특수성을 깊이 고려한 설계가 특징이다.1 VPR에서 장소를 구별하는 데 중요한 랜드마크는 이미지 내에서 다양한 크기와 형태로 나타난다. MulConv 어댑터는 이러한 특성에 대응하기 위해 다중 스케일 합성곱(multi-scale convolution) 연산을 활용한다. 이를 통해 이미지 내 다양한 크기의 지역적 특징(local information)과 공간적 사전 정보(spatial prior)를 트랜스포머 기반의 VFM에 효과적으로 주입하는 역할을 수행한다.8 즉, 일반적인 PETL 기법들이 수학적 구조에 집중하는 것과 달리, MulConv 어댑터는 합성곱의 '지역성(locality)'과 '공간적 계층성(spatial hierarchy)'이라는 귀납적 편향(inductive bias)을 명시적으로 주입함으로써 이미지 태스크에 더 적합한 적응 방식을 제시했다는 점에서 기술적 깊이를 보여준다.


교차-이미지 인코더는 CricaVPR의 핵심 아이디어인 '이미지 간 상호작용'을 구현하는 가장 중요한 모듈이다. 이 모듈의 작동 과정은 다음과 같다.

1. **초기 특징 표현 생성**: 백본과 MulConv 어댑터를 통과한 특징 맵은 먼저 공간적 피라미드(spatial pyramid)와 GeM(Generalized Mean) 풀링을 통해 여러 개의 지역 특징(regional features)으로 분할된다.8 예를 들어, 특징 맵을 1x1, 2x2, 3x3으로 분할하여 총 14개의 지역 특징 벡터를 생성한다.

2. **배치 단위 시퀀스 구성**: 훈련 배치의 크기가 $B$일 때, $i$번째 지역에 해당하는 특징들을 배치 내 모든 이미지로부터 수집하여 $\mathbf{f}_i = \{\mathbf{f}_i^1, \mathbf{f}_i^2, \dots, \mathbf{f}_i^B\}$ 와 같은 특징 벡터 시퀀스를 구성한다.8 총 14개의 이러한 시퀀스가 생성된다.

3. **상관관계 모델링**: 각 지역 특징 시퀀스 $\mathbf{f}_i$는 독립적으로 교차-이미지 인코더에 입력된다. 이 인코더는 두 개의 표준 트랜스포머 인코더 레이어로 구성되며, 각 레이어는 멀티헤드 어텐션(Multi-Head Attention), MLP, Layer Normalization으로 이루어져 있다.8 이 과정에서 어텐션 메커니즘이 핵심적인 역할을 수행한다. 입력된 시퀀스로부터 쿼리(Q), 키(K), 밸류(V) 행렬을 생성한 후, 스케일드 닷-프로덕트 어텐션(Scaled Dot-Product Attention)을 다음과 같이 계산한다.
   $$
   \text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
   $$
   여기서 $d_k$는 키 벡터의 차원이다. 이 연산을 통해 한 이미지의 특정 지역 특징($Q$)이 배치 내 다른 모든 이미지의 동일 지역 특징들($K$, $V$)과의 유사도를 계산하고, 그 가중합을 통해 문맥적으로 풍부해진 새로운 특징으로 업데이트된다.8

4. **최종 전역 특징 생성**: 인코더를 통과하여 강화된 각 이미지의 14개 지역 특징들을 순차적으로 연결(concatenate)하고 L2 정규화를 거쳐, 최종적으로 강인하고 식별력 높은 단일 전역 특징 벡터를 생성한다.8


CricaVPR의 이론적 우수성은 다양한 표준 벤치마크 데이터셋에서의 엄밀한 실험을 통해 입증되었다. 평가는 주로 상위 N개 검색 결과 내에 정답 이미지가 포함될 확률을 나타내는 Recall@N (R@1, R@5, R@10)을 핵심 성능 지표로 사용한다.6


CricaVPR의 성능은 Pitts30k, Tokyo 24/7, Nordland, AmsterTime, SVOX 등 VPR 분야의 주요 벤치마크 데이터셋에서 당대의 SOTA(State-of-the-Art) 모델들과 비교되었다.17 이 데이터셋들은 각각 대규모 도시 환경, 극심한 주야간 변화, 계절 변화 등 VPR이 마주하는 다양한 난제들을 대표한다.

실험 결과, CricaVPR은 대부분의 데이터셋에서 기존 SOTA 모델들을 능가하는 성능을 보였다. 특히 주목할 점은 Nordland(계절 변화), AmsterTime(시간 변화), SVOX-Night(주야간 변화)와 같이 극심한 도메인 변화를 포함하는 '어려운(challenging)' 데이터셋에서 다른 모델들과의 성능 격차를 크게 벌렸다는 것이다.17 이는 교차-이미지 상관관계 학습이 환경 변화에 대한 불변성을 효과적으로 학습시킨다는 CricaVPR의 핵심 가설을 강력하게 뒷받침하는 결과다. 또한, 대표적인 벤치마크인 Pitts30k에서는 512차원의 비교적 컴팩트한 전역 특징만으로도 94.5%라는 높은 R@1을 달성하며 그 효율성을 입증했다.6

| 모델         | Pitts30k (R@1) | Tokyo 24/7 (R@1) | Nordland (R@1) | AmsterTime (R@1) |
| ------------ | -------------- | ---------------- | -------------- | ---------------- |
| CosPlace     | 91.7           | 84.4             | 55.4           | 69.3             |
| MixVPR       | 92.2           | 87.0             | 58.6           | 74.2             |
| SelaVPR      | 93.3           | 89.8             | 76.2           | 82.5             |
| SALAD        | 93.8           | 90.5             | 74.1           | 83.1             |
| **CricaVPR** | **94.5**       | **91.1**         | **89.3**       | **87.2**         |


CricaVPR의 높은 성능이 어떤 구성 요소로부터 기인하는지를 명확히 파악하기 위해 상세한 Ablation Study가 수행되었다.

- **교차-이미지 인코더의 효과**: CricaVPR의 아키텍처에서 교차-이미지 인코더를 제거했을 때와 비교한 결과, 인코더를 추가하는 것만으로도 모든 데이터셋에서 일관되고 상당한 성능 향상이 관찰되었다.17 이는 '교차-이미지 상관관계 학습'이라는 CricaVPR의 핵심 아이디어가 실제 성능 향상에 직접적으로 기여함을 실험적으로 증명한다.
- **인코더 레이어 수의 영향**: 교차-이미지 인코더를 구성하는 트랜스포머 레이어의 수를 변화시키며 성능을 측정한 결과, 레이어가 1개일 때보다 2개일 때 더 높은 성능을 보였으며, 2개에서 최적의 성능을 달성했다.17 이는 단일 어텐션 레이어만으로는 배치 내 이미지들의 복잡한 상관관계를 충분히 모델링하기 어렵고, 적절한 깊이의 인코더가 필요함을 시사한다.
- **MulConv 어댑터 및 인코더의 시각적 효과**: t-SNE를 이용한 특징 시각화 분석은 각 구성 요소의 역할을 직관적으로 보여준다.17 사전 훈련된 DINOv2가 추출한 특징들은 서로 다른 장소임에도 불구하고 특징 공간에서 잘 분리되지 않는 모습을 보였다. 여기에 제안된 MulConv 어댑터를 적용하자, 대부분의 장소들이 구별되기 시작하며 특징 공간의 구조가 개선되었다. 마지막으로 교차-이미지 인코더까지 적용하자, 동일한 장소의 이미지 특징들은 더욱 강하게 군집화되고(intra-class compactness 증가), 서로 다른 장소의 특징들은 더 멀리 분리되어(inter-class separability 증가), 지각적 모호성 문제를 해결하는 데 각 구성 요소가 명확히 기여함을 시각적으로 입증했다.

| 구성 요소                                   | Pitts30k (R@1) | Tokyo 24/7 (R@1) |
| ------------------------------------------- | -------------- | ---------------- |
| DINOv2-base (어댑터 및 인코더 없음)         | 89.5           | 83.2             |
| + MulConv Adapter                           | 92.1           | 87.5             |
| + Cross-image Encoder (1-layer)             | 93.7           | 89.9             |
| + Cross-image Encoder (2-layers) (CricaVPR) | **94.5**       | **91.1**         |


CricaVPR의 학술적 위상을 입체적으로 조명하기 위해서는 동시대에 등장한 다른 DINOv2 기반 SOTA 모델들과의 철학적, 기술적 차이를 심층적으로 비교 분석할 필요가 있다.


EffoVPR은 DINOv2의 잠재력을 활용하는 또 다른 접근법을 제시한다. 이 모델은 DINOv2의 최종 CLS 토큰을 초기 검색을 위한 전역 특징으로 사용하고, ViT의 **중간 셀프-어텐션 레이어**에서 추출한 내부 특징들을 재순위화(re-ranking)를 위한 지역 특징으로 활용한다.12 반면, CricaVPR은 재순위화 단계가 없는 단일 단계(one-stage) 방식으로, 최종 레이어의 패치 토큰들을 다중 스케일로 집계하고 이를 교차-이미지 인코더로 정제하여 

**하나의 강력하고 완성도 높은 전역 특징**을 만드는 데 모든 역량을 집중한다. 이는 DINOv2의 잠재력을 활용하는 두 가지 다른 철학, 즉 EffoVPR의 '내재된 멀티-레벨 특징의 분리 활용'과 CricaVPR의 '최종 특징의 문맥적 강화' 사이의 차이를 명확히 보여준다.


SciceVPR은 CricaVPR의 학술적 영향력을 가장 명확하게 보여주는 후속 연구다. SciceVPR은 CricaVPR이 제시한 '교차-이미지 상관관계'라는 핵심 아이디어를 직접적으로 계승한다. 그러나 CricaVPR의 방식이 훈련 배치 구성에 따라 "불안정한 검색 결과(unstable retrieval results)"를 초래할 수 있다는 비판적 관점을 제시한다.14 이 문제를 해결하기 위해 SciceVPR은 배치 내 이미지들 간의 '불변하는 상관관계(invariant correlation)'를 지식 증류(knowledge distillation)와 유사한 방식으로 안정적으로 학습하는 '안정적 교차-이미지 상관관계 강화 모델'을 제안한다.14

이러한 전개는 과학적 발전이 '개념 제시 → 구현 → 비판 및 개선'의 순환을 통해 이루어짐을 보여주는 전형적인 사례다. CricaVPR이 '배치 내 이미지 간 상호작용'이라는 강력하고 새로운 개념을 제시하고 어텐션 기반 인코더라는 구체적인 구현을 선보였기에, 후속 연구가 그 아이디어의 가치를 인정하고 '안정성'이라는 새로운 연구 문제를 정의하며 이를 개선하려 한 것이다. 이는 CricaVPR이 단순히 특정 벤치마크에서 높은 점수를 기록한 논문을 넘어, **VPR 분야에 새로운 연구 주제와 어휘(vocabulary)를 제공한 '개념적 선구자(conceptual pioneer)'** 역할을 했음을 역설적으로 증명한다.


SelaVPR 역시 DINOv2 기반의 PETL을 사용하지만, VFM 적응에 있어 다른 접근법을 취한다. SelaVPR은 전역 특징과 지역 특징 모두에 대한 적응을 동시에 수행하는 '하이브리드 적응(hybrid adaptation)' 방식을 제안한다.10 즉, 전역 특징과 지역 특징을 위한 별도의 경량 어댑터를 튜닝하여 2단계 재순위화 과정 모두에 활용한다. 반면, CricaVPR은 다중 스케일 지역 특징을 집계하고 상호작용시키는 과정 전체가 궁극적으로는 단일 전역 특징의 강인성과 식별력을 극대화하는 데 초점을 맞춘다.

| 모델         | 핵심 아이디어                 | DINOv2 활용 전략            | 접근 방식 | 강점                            | 잠재적 약점                |
| ------------ | ----------------------------- | --------------------------- | --------- | ------------------------------- | -------------------------- |
| **CricaVPR** | 교차-이미지 상관관계 학습     | 최종 특징의 문맥적 강화     | 1-stage   | 극심한 환경 변화에 강인         | 배치 구성에 민감, 불안정성 |
| **EffoVPR**  | 중간 어텐션 레이어 활용       | 내재된 멀티-레벨 특징 분리  | 2-stage   | 제로샷 성능, 재순위화 강점      | 재순위화의 계산 비용       |
| **SciceVPR** | 안정적인 교차-이미지 상관관계 | CricaVPR 계승 및 안정화     | 1-stage   | CricaVPR의 강인함 + 안정성      | CricaVPR 대비 복잡성 증가  |
| **SelaVPR**  | 하이브리드 적응               | 전역 및 지역 특징 동시 적응 | 2-stage   | 전역/지역 특징의 균형 잡힌 성능 | 2단계 파이프라인의 복잡성  |



CricaVPR은 VPR 분야에 다층적인 기여를 했다.

- **개념적 기여**: VPR 훈련 패러다임을 '개별 이미지의 독립적 처리'에서 '배치 단위의 관계형 학습'으로 확장하는 새로운 관점을 제시했다. 이는 VPR 연구의 지평을 넓히는 중요한 개념적 전환이다.
- **기술적 기여**: VPR 도메인의 특성을 반영한 파라미터 효율적 어댑터(MulConv Adapter)와 교차-이미지 상관관계 인코더라는 구체적인 아키텍처를 제안하고, 그 효과를 실험적으로 입증했다.8
- **실용적 기여**: 거대 비전 모델을 효율적으로 활용하여 SOTA 수준의 높은 성능을 달성하면서도, 전체 미세조정 대비 훈련 시간을 단축하는 실용적인 방법론을 제공했다.1


혁신적인 기여에도 불구하고 CricaVPR은 몇 가지 내재적 한계를 가진다.

- **배치 구성 의존성 및 불안정성**: 모델의 성능이 훈련 시 미니배치가 어떻게 구성되는지에 민감하게 반응할 수 있다. 이는 SciceVPR과 같은 후속 연구에서 지적된 잠재적 불안정성 문제로 이어진다.14
- **해석 가능성 부족**: 어텐션 메커니즘이 구체적으로 어떤 이미지들 간의 상관관계를 중요하게 학습하여 성능 향상을 이끌어내는지에 대한 해석이 부족하여, 모델의 내부 동작을 직관적으로 이해하기 어렵다.
- **훈련 단계의 계산 오버헤드**: 추론 시에는 단일 이미지를 처리하지만, 훈련 시에는 배치 내 모든 이미지 간의 어텐션을 계산해야 하므로, 배치 크기가 커질수록 훈련 단계의 계산 복잡도가 제곱에 비례하여 증가할 수 있다.


CricaVPR이 제시한 개념과 한계는 다음과 같은 흥미로운 미래 연구 방향을 제시한다.

- **능동적 배치 구성(Active Batch Curation)**: 무작위 샘플링에 의존하는 대신, 모델 학습에 가장 도움이 될 것으로 예상되는 이미지들(예: 시점 변화가 크거나, 환경 변화가 극심한 이미지 쌍, 혹은 모델이 헷갈려하는 'hard negative' 이미지)을 의도적으로 배치에 포함시키는 '지능형 배치 샘플링' 전략을 연구할 수 있다.
- **상관관계 학습의 안정화 및 제어**: SciceVPR의 연구 방향을 더욱 발전시켜, 학습하고자 하는 불변성(invariance)의 종류(예: 조명 불변성, 시점 불변성)를 명시적으로 제어하며 교차-이미지 상관관계를 학습하는 방법론을 개발할 수 있다.
- **타 도메인으로의 확장**: CricaVPR의 '관계형 배치 학습' 아이디어는 VPR에 국한되지 않는다. 객체 재인식(Re-ID), 이상 탐지(Anomaly Detection) 등 다른 메트릭 러닝(Metric Learning) 기반의 비전 태스크에서도 유사한 문제를 해결하는 데 효과적으로 적용될 수 있을 것이다.

궁극적으로 CricaVPR이 VPR 분야에 던진 가장 중요한 화두는 **데이터 샘플링 전략을 모델 아키텍처 설계와 동등하게 중요한 연구 요소로 격상**시켰다는 점이다. CricaVPR의 성능이 배치 내 이미지들의 상호작용에 직접적으로 의존한다는 사실은, 배치를 어떻게 구성하느냐가 모델의 최종 성능을 결정하는 핵심 변수가 됨을 의미한다. 이는 '모델 중심 AI(Model-centric AI)'에서 '데이터 중심 AI(Data-centric AI)'로의 전환이라는 더 큰 기술적 흐름과도 맥을 같이 한다. 최적의 모델 아키텍처를 찾는 것만큼이나, 모델에게 '최적의 학습 경험(배치)'을 제공하는 방법이 중요해진 것이다. 따라서 CricaVPR은 VPR 연구자들에게 모델 구조뿐만 아니라, `DataLoader`와 `Sampler`의 설계에 대해서도 더 깊이 고민해야 한다는 새로운 과제를 제시하며 VPR 연구의 미래를 한 단계 진전시켰다.


1. CricaVPR: Cross-Image Correlation-Aware Representation Learning for Visual Place Recognition - ResearchGate, accessed August 22, 2025, https://www.researchgate.net/publication/381164040_CricaVPR_Cross-image_Correlation-aware_Representation_Learning_for_Visual_Place_Recognition
2. CricaVPR: Cross-image Correlation-aware Representation Learning for Visual Place Recognition - CVF Open Access, accessed August 22, 2025, https://openaccess.thecvf.com/content/CVPR2024/papers/Lu_CricaVPR_Cross-image_Correlation-aware_Representation_Learning_for_Visual_Place_Recognition_CVPR_2024_paper.pdf
3. NYU-VPR: Long-Term Visual Place Recognition Benchmark with View Direction and Data Anonymization Influences, accessed August 22, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9394449/
4. Visual Place Recognition: Building an Evaluation Framework for Model Robustness - University of Twente, accessed August 22, 2025, http://essay.utwente.nl/100860/1/Gosa_BA_EEMCS.pdf
5. Visual Place Recognition Based on Dynamic Difference and Dual-Path Feature Enhancement, accessed August 22, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12251715/
6. CricaVPR: Cross-image Correlation-aware Representation Learning for Visual Place Recognition - arXiv, accessed August 22, 2025, https://arxiv.org/html/2402.19231v1
7. CriSALAD: Robust Visual Place Recognition Using Cross-Image Information and Optimal Transport Aggregation - MDPI, accessed August 22, 2025, https://www.mdpi.com/2076-3417/15/10/5287
8. [Literature Review] CricaVPR: Cross-image Correlation-aware Representation Learning for Visual Place Recognition - Moonlight, accessed August 22, 2025, https://www.themoonlight.io/en/review/cricavpr-cross-image-correlation-aware-representation-learning-for-visual-place-recognition
9. CricaVPR: Cross-image Correlation-aware Representation Learning for Visual Place Recognition - arXiv, accessed August 22, 2025, https://arxiv.org/html/2402.19231v2
10. slz929/awesome-visual-place-recognition - GitHub, accessed August 22, 2025, https://github.com/slz929/awesome-visual-place-recognition
11. Structured Pruning for Efficient Visual Place Recognition - arXiv, accessed August 22, 2025, https://arxiv.org/html/2409.07834v1
12. EffoVPR: Effective Foundation Model Utilization for Visual Place Recognition - arXiv, accessed August 22, 2025, https://arxiv.org/html/2405.18065v1
13. EffoVPR: Effective Foundation Model Utilization for Visual Place Recognition - OpenReview, accessed August 22, 2025, https://openreview.net/forum?id=NSpe8QgsCB
14. [2502.20676] SciceVPR: Stable Cross-Image Correlation Enhanced Model for Visual Place Recognition - arXiv, accessed August 22, 2025, https://arxiv.org/abs/2502.20676
15. [2402.19231] CricaVPR: Cross-image Correlation-aware Representation Learning for Visual Place Recognition - arXiv, accessed August 22, 2025, https://arxiv.org/abs/2402.19231
16. Official repository for the CVPR 2024 paper "CricaVPR: Cross-image Correlation-aware Representation Learning for Visual Place Recognition". - GitHub, accessed August 22, 2025, https://github.com/Lu-Feng/CricaVPR
17. CricaVPR: Cross-image Correlation-aware Representation Learning for Visual Place Recognition Supplementary Material, accessed August 22, 2025, https://openaccess.thecvf.com/content/CVPR2024/supplemental/Lu_CricaVPR_Cross-image_Correlation-aware_CVPR_2024_supplemental.pdf
18. Unified Depth-Guided Feature Fusion and Reranking for Hierarchical Place Recognition, accessed August 22, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12252300/
19. SciceVPR: Stable Cross-Image Correlation Enhanced Model for Visual Place Recognition | Request PDF - ResearchGate, accessed August 22, 2025, https://www.researchgate.net/publication/389510229_SciceVPR_Stable_Cross-Image_Correlation_Enhanced_Model_for_Visual_Place_Recognition
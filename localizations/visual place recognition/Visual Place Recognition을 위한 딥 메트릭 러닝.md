# Visual Place Recognition을 위한 딥 메트릭 러닝
[시각적 장소 인식 (Visual Place Recognition)](./index.md)



Visual Place Recognition (VPR)은 주어진 쿼리(query) 이미지와 동일한 장소에서 촬영된 이미지를, 지리 정보가 태깅된 대규모 데이터베이스에서 검색하는 프로세스로 정의된다.1 본질적으로 VPR은 특정 목적을 가진 이미지 검색(Image Retrieval) 문제로 간주할 수 있으며, 그 목적은 바로 '장소'를 인식하는 것이다.1

VPR 시스템의 일반적인 파이프라인은 두 단계로 구성된다.

- 첫째, 입력된 이미지로부터 장소의 핵심 정보를 압축한 특징 벡터(feature vector), 즉 디스크립터(descriptor)를 추출한다.
- 둘째, 이 쿼리 이미지의 디스크립터를 데이터베이스에 저장된 수많은 이미지의 디스크립터와 비교한다. 이 비교는 주로 유클리드 거리(Euclidean distance)나 코사인 유사도(cosine similarity)와 같은 거리 척도를 사용하여 이루어지며, 거리가 가장 가까운 이미지를 가장 유사한 장소로 판단하여 반환한다.1

VPR 기술의 중요성은 현대 인공지능 시스템이 물리적 세계와 상호작용해야 하는 다양한 응용 분야에서 두드러진다. 특히 자율 주행 자동차와 모바일 로봇의 항법 시스템에서 VPR은 필수적인 구성 요소로 자리 잡았다.1 위성항법시스템(GPS) 신호가 빌딩 숲에 의해 차단되거나 왜곡되는 도심 협곡(urban canyon) 환경이나 GPS 수신이 불가능한 실내 공간에서, VPR은 시각 정보만을 이용해 로봇이나 차량이 자신의 위치를 파악할 수 있는 강력한 대안을 제공한다.6

또한, VPR은 SLAM (Simultaneous Localization and Mapping, 동시적 위치 추정 및 지도 작성) 기술의 성능을 극대화하는 데 결정적인 역할을 한다. SLAM 시스템이 장시간 동작하며 지도를 작성할 때 필연적으로 발생하는 누적 오차는, 이전에 방문했던 장소를 다시 인식하는 '루프 폐쇄(loop closure detection)' 과정을 통해 보정될 수 있다.7 VPR은 바로 이 루프 폐쇄를 감지하는 핵심 기술이다. 만약 시스템이 빠른 움직임이나 조명 변화로 인해 현재 위치를 놓치는 추적 실패(tracking failure) 상황에 직면했을 때, VPR은 이전에 구축된 지도를 참조하여 현재 위치를 신속하게 다시 파악하는 재지역화(relocalization)를 가능하게 한다.10

더 나아가 VPR은 증강 현실(AR) 기기나 지능형 플랫폼이 물리적 세계를 시각적으로 이해하고 상호작용하는 '공간 AI (Spatial AI)' 패러다임의 핵심 구성 요소로 여겨진다.3 기계가 단순히 지도를 따라 움직이는 것을 넘어, 인간처럼 공간을 '이해'하고 그 안에서 지능적으로 행동하기 위해서는 자신이 어디에 있는지를 시각적으로 인식하는 능력이 선행되어야 한다. 이처럼 VPR은 단순한 이미지 매칭 기술을 넘어, 기계에 공간적 지능을 부여하는 근본적인 기술이라 할 수 있다.


VPR 시스템이 실세계에서 강건하게 동작하기 위해서는 여러 가지 근본적인 도전 과제를 극복해야 한다. 이 문제들은 동일한 장소라 할지라도 시각적 외형이 극적으로 변할 수 있다는 사실에서 기인한다.

첫 번째 도전 과제는 **외형 변화(Appearance/Condition Variation)**이다. 같은 장소에서 촬영된 이미지라도 촬영 시점의 조건에 따라 매우 다르게 보일 수 있다. 예를 들어, 낮과 밤의 극적인 조명 변화, 봄의 푸르름과 겨울의 눈 덮인 풍경과 같은 계절적 변화, 맑은 날과 비 오는 날의 날씨 변화 등은 이미지의 색상, 질감, 대비를 완전히 바꾼다.1 또한, 이미지에 일시적으로 나타나는 차량, 보행자와 같은 동적 객체(dynamic objects)는 장소의 본질적인 특징을 가리는 '노이즈'로 작용할 수 있다. 이러한 변화에도 불구하고 장소의 일관된 특징을 추출해내는 능력, 즉 강건성(robustness)은 VPR 모델의 성능을 평가하는 핵심 기준이다.1

두 번째 도전 과제는 **시점 변화(Viewpoint Variation)**이다. 카메라의 위치나 촬영 각도가 조금만 달라져도 이미지에 담기는 장면의 구성, 객체의 크기와 형태, 원근감이 크게 변한다.6 자율 주행 차량이 차선을 변경하거나 로봇이 모퉁이를 돌 때 발생하는 미세한 시점 변화만으로도 이미지의 픽셀 값은 완전히 달라질 수 있다. VPR 시스템은 이러한 시점 변화에 불변하는(invariant) 특징을 학습해야 한다.

세 번째이자 가장 까다로운 도전 과제는 **인식 모호성(Perceptual Aliasing)**이다.10 이는 서로 다른 두 장소가 시각적으로 매우 유사하게 보이는 현상을 의미한다. 예를 들어, 현대적인 건물의 반복적인 복도, 유사한 디자인의 사무실, 고속도로의 끝없이 이어지는 풍경 등은 육안으로도 구별하기 어렵다. 이러한 경우, VPR 시스템은 시각적으로는 유사하지만 지리적으로는 완전히 다른 장소를 동일한 장소로 오인하는 심각한 오류(false positive)를 범할 수 있다. 반대로, 앞서 언급한 외형 변화로 인해 동일한 장소를 다른 장소로 잘못 판단하는 오류(false negative)는 '지각 변동성(perceptual variability)'이라고 부른다.10 성공적인 VPR 시스템은 이 두 가지 상반된 문제를 동시에 해결해야 하는 딜레마에 직면한다.


이러한 VPR의 난제들을 해결하기 위해 다양한 접근법이 시도되었다. 초기 연구들은 SIFT(Scale-Invariant Feature Transform)와 같은 수작업 특징(hand-crafted features)을 추출하고, 이를 BoW(Bag-of-Words)나 VLAD(Vector of Locally Aggregated Descriptors)와 같은 기법으로 인코딩하는 방식을 사용했다.10 이러한 방법들은 특정 조건에서는 준수한 성능을 보였지만, 극심한 조명 변화나 계절 변화와 같은 근본적인 외형 변화 앞에서는 취약점을 드러냈다. 특징 추출 과정이 고정되어 있어 데이터로부터 장소의 불변하는 속성을 학습할 수 없었기 때문이다.

이러한 한계를 극복하기 위한 대안으로 딥러닝(Deep Learning)이 부상했다. 특히, 합성곱 신경망(CNN)은 이미지로부터 계층적인 특징을 자동으로 학습하는 능력 덕분에 수작업 특징보다 월등히 뛰어난 표현력을 보여주며 VPR 성능을 획기적으로 개선했다.10 그러나 단순히 이미지를 분류(classification)하도록 학습된 CNN 모델을 그대로 사용하는 것만으로는 VPR 문제에 최적화된 해결책이라 보기 어려웠다. 분류 문제는 '이미지가 어떤 클래스에 속하는가'에 초점을 맞추지만, VPR의 본질은 '두 이미지가 얼마나 유사한가'를 판단하는 것이기 때문이다.

이 지점에서 **딥 메트릭 러닝(Deep Metric Learning, DML)**이 VPR 문제의 핵심 해결책으로 등장했다.17 DML은 분류를 위한 학습을 넘어, '유사성'이라는 척도 자체를 학습하는 것을 목표로 하는 패러다임이다. DML의 핵심 아이디어는 신경망을 이용해 원본 이미지들을 고차원의 벡터 공간, 즉 **임베딩 공간(embedding space)**으로 매핑하는 함수를 학습하는 것이다. 이 학습 과정은 한 가지 원칙에 의해 유도된다: 의미적으로 유사한 이미지(같은 장소)들은 임베딩 공간에서 서로 가까운 위치에, 유사하지 않은 이미지(다른 장소)들은 서로 먼 위치에 자리 잡도록 하는 것이다.19

이러한 방식으로 학습된 임베딩 공간에서는 벡터 간의 기하학적 거리(예: 유클리드 거리)가 곧 장소의 의미적 유사도를 나타내게 된다. 따라서 극심한 외형 변화나 시점 변화에도 불구하고 같은 장소의 이미지들은 임베딩 공간 내에서 일관되게 가까운 클러스터를 형성하게 된다. 이는 VPR이 직면한 근본적인 도전 과제들에 대한 매우 강력하고 유연한 해결책을 제공한다. 본 자습서는 바로 이 DML을 VPR에 적용하는 방법론에 대해 기초부터 최신 기술까지 심도 있게 다룰 것이다.


딥 메트릭 러닝(DML)은 데이터 포인트 간의 유사성을 측정하는 것을 목표로 하는 기술 그룹이다. VPR의 맥락에서 이는 이미지 간의 시각적 유사성을 정량화하는 것을 의미한다. DML의 핵심 철학은 원본 이미지 공간에서 복잡하게 얽혀 있는 유사성 관계를, 간단한 거리 계산만으로도 유사성을 파악할 수 있는 새로운 벡터 공간, 즉 임베딩 공간을 학습하는 데 있다.


DML의 궁극적인 목표는 원본 데이터(이미지) $x \in \mathcal{X}$를 $n$차원의 유클리드 공간 $\mathbb{R}^n$으로 매핑하는 신경망 인코더, 즉 임베딩 함수 $f_\theta(\cdot): \mathcal{X} \to \mathbb{R}^n$을 학습하는 것이다.18 여기서 $\theta$는 신경망의 학습 가능한 파라미터를 의미한다. 이 과정을 통해 생성된 벡터 $f_\theta(x)$를 이미지 $x$의 **임베딩(embedding)** 또는 **디스크립터(descriptor)**라고 부른다.

이렇게 구축된 임베딩 공간은 다음과 같은 중요한 속성을 갖도록 설계된다. 두 이미지 $x_1$과 $x_2$가 주어졌을 때, 이들의 임베딩 $f_\theta(x_1)$과 $f_\theta(x_2)$ 사이의 거리 $D(f_\theta(x_1), f_\theta(x_2))$는 두 이미지의 의미적 유사도를 직접적으로 반영한다.18 구체적으로, $x_1$과 $x_2$가 같은 장소를 촬영한 이미지(positive pair)라면 두 임베딩 사이의 거리는 최소화되어야 하고, 서로 다른 장소를 촬영한 이미지(negative pair)라면 거리는 최대화되어야 한다.21

이러한 접근 방식의 강력함은 복잡한 비선형 관계를 학습하는 딥러닝 모델의 능력과 단순하고 효율적인 거리 척도를 결합하는 데 있다. 초기 메트릭 러닝 연구가 데이터에 적합한 복잡한 거리 함수 자체(예: 마할라노비스 거리 행렬 $M$)를 학습하는 데 초점을 맞췄던 반면 22, 현대 DML의 패러다임은 거리 함수는 유클리드 거리와 같이 간단한 것으로 고정하고, 대신 이 간단한 거리 척도 하에서 의미적 유사성이 잘 표현되도록 데이터를 변환하는 강력한 비선형 특징 표현 $f_\theta(\cdot)$을 학습하는 방향으로 전환되었다.20 이러한 패러다임 전환은 두 가지 주요 이점을 가져왔다. 첫째, ImageNet과 같은 대규모 데이터셋으로 사전 훈련된 강력한 딥러닝 모델을 특징 추출기로 활용할 수 있게 되어 학습 효율성과 성능이 크게 향상되었다. 둘째, 학습의 복잡성을 거리 함수가 아닌, 표현력이 뛰어난 신경망 모델이 전담하게 함으로써 더 정교하고 강건한 임베딩 공간을 구축할 수 있게 되었다. 결국 DML 문제는 "어떤 거리 함수를 학습할 것인가?"에서 "어떻게 효과적인 임베딩 함수 $f_\theta(\cdot)$를 학습시킬 것인가?"로 귀결되었으며, 이 질문에 대한 해답이 바로 손실 함수(loss function)의 설계에 있다.


DML 기반의 VPR 시스템은 일반적으로 세 가지 핵심 요소로 구성된다. 이 세 요소는 상호 유기적으로 작용하여 효과적인 임베딩 공간을 학습한다.

1. **특징 추출기 (Backbone Network)**: 이 구성 요소는 입력 이미지로부터 원시 픽셀 정보를 의미 있는 특징 표현으로 변환하는 역할을 한다. 시스템의 '눈'에 해당하며, 임베딩 함수 $f_\theta(\cdot)$의 본체다. 주로 ImageNet과 같은 대규모 데이터셋으로 사전 훈련된 CNN(예: VGG, ResNet)이나 Vision Transformer (ViT)가 백본으로 사용된다.1 사전 훈련된 모델을 사용하면 이미지의 일반적인 시각적 패턴(예: 모서리, 질감, 객체 일부)을 처음부터 학습할 필요가 없어, VPR이라는 특정 작업에 대한 학습을 더 효율적으로 진행할 수 있다. 학습 과정에서는 이 백본 네트워크의 파라미터 

   $\theta$가 손실 함수에 의해 최적화된다.

2. **거리 함수 (Distance Metric)**: 임베딩 공간에서 두 임베딩 벡터 간의 거리를 계산하는 함수다. 이 함수는 임베딩 공간의 기하학적 구조를 정의한다. DML에서는 일반적으로 학습 과정에서 파라미터가 변하지 않는 고정된 거리 함수를 사용한다. 가장 널리 사용되는 거리 함수는 다음과 같다.1

   - **유클리드 거리 (L2-norm)**: 두 벡터 $p, q \in \mathbb{R}^n$ 사이의 직선 거리를 측정한다. 직관적이고 계산이 간단하여 가장 보편적으로 사용된다.
     $$
     D(p, q) = \|p - q\|_2 = \sqrt{\sum_{i=1}^{n}(p_i - q_i)^2}
     $$
     **코사인 유사도 (Cosine Similarity)**: 두 벡터 사이의 각도를 측정하여 방향성의 유사도를 평가한다. 벡터의 크기(norm)에 무관하며, 주로 임베딩 벡터를 단위 구(hypersphere) 상에 정규화하여 사용할 때 유용하다. 거리는 $1 - similarity$로 계산할 수 있다.

3. **손실 함수 (Loss Function)**: DML 시스템의 '교사' 역할을 하는 가장 중요한 구성 요소다. 손실 함수는 현재 임베딩 공간의 상태가 얼마나 바람직한지를 정량적으로 평가하고, 이를 바탕으로 네트워크 파라미터 $\theta$를 어떤 방향으로 업데이트해야 할지에 대한 그래디언트(gradient)를 계산한다. DML 손실 함수의 공통적인 목표는 임베딩 공간 내에서 '클래스 내 분산(intra-class variance)'은 최소화하고 '클래스 간 분산(inter-class variance)'은 최대화하는 것이다. 즉, 같은 장소의 이미지 임베딩들은 서로 가깝게 모으고(pull), 다른 장소의 이미지 임베딩들은 서로 멀리 밀어내는(push) 방식으로 학습을 유도한다.18 다음 장에서는 VPR에서 사용되는 다양한 손실 함수들을 심층적으로 분석할 것이다.


딥 메트릭 러닝의 성패는 손실 함수를 어떻게 설계하고 활용하는지에 달려 있다. 손실 함수는 임베딩 공간의 구조를 직접적으로 형성하며, 모델이 VPR의 다양한 도전 과제를 극복할 수 있는 강건한 표현을 학습하도록 유도한다. VPR을 위한 DML 손실 함수는 크게 쌍 기반(pair-based), 삼중항 기반(triplet-based), 그리고 프록시 기반(proxy-based) 접근법으로 나눌 수 있다. 이들의 발전 과정은 대규모 데이터 시대에 '데이터 샘플링의 복잡성'과 '학습 효율성' 사이의 균형을 맞추기 위한 공학적 고민의 결과물이다.


대조 손실(Contrastive Loss)은 DML에서 가장 기본적이고 직관적인 쌍 기반 손실 함수다.18 이 손실 함수는 이미지 쌍 $(x_1, x_2)$과 이 쌍이 같은 장소를 나타내는지 여부를 나타내는 이진 레이블 $y$를 입력으로 받는다. 여기서 $y=0$은 두 이미지가 같은 장소임을 의미하는 포지티브 쌍(positive pair)이고, $y=1$은 다른 장소임을 의미하는 네거티브 쌍(negative pair)이다.

대조 손실의 목표는 명확하다. 포지티브 쌍의 경우, 두 이미지의 임베딩 $f(x_1)$과 $f(x_2)$ 사이의 거리 $D$를 가능한 한 0에 가깝게 만들어야 한다. 반면, 네거티브 쌍의 경우에는 두 임베딩 사이의 거리가 미리 정의된 하이퍼파라미터인 마진(margin) $m$보다 커지도록 만들어야 한다.24 마진 $m$은 네거티브 쌍이 최소한 이 정도 거리는 떨어져 있어야 한다는 '안전거리' 역할을 한다. 만약 네거티브 쌍의 거리가 이미 마진보다 크다면, 손실은 0이 되어 더 이상 두 임베딩을 밀어내지 않는다.

수학적으로 대조 손실은 다음과 같이 정의된다 24:
$$
L(x_1, x_2, y) = (1-y) \frac{1}{2} D^2 + y \frac{1}{2} \{\max(0, m - D)\}^2
$$
여기서 $D = \|f(x_1) - f(x_2)\|_2$는 두 임베딩 간의 유클리드 거리다. 위 식에서 첫 번째 항은 $y=0$일 때(포지티브 쌍) 활성화되어 거리 $D$를 직접적으로 최소화한다. 두 번째 항은 $y=1$일 때(네거티브 쌍) 활성화되며, 힌지 손실(hinge loss)의 형태로 거리 $D$가 마진 $m$보다 작을 때만 페널티를 부과한다.

이러한 손실 함수 구조를 학습시키기 위해 주로 **샴 네트워크(Siamese Network)** 아키텍처가 사용된다.21 샴 네트워크는 동일한 구조와 가중치를 공유하는 두 개의 '쌍둥이(twin)' 서브네트워크로 구성된다.27 이미지 쌍의 각 이미지가 이 쌍둥이 네트워크를 통과하여 임베딩을 생성하고, 이 두 임베딩을 이용해 대조 손실을 계산하여 네트워크의 가중치를 업데이트한다. 가중치를 공유하기 때문에 두 이미지는 동일한 방식으로 특징 공간에 매핑되며, 이는 일관된 유사성 척도를 학습하는 데 매우 효과적이다.

하지만 대조 손실은 한계점을 가진다. 이 손실 함수는 각 쌍을 독립적으로 평가하기 때문에 임베딩 공간의 전역적인 구조를 고려하지 못한다. 예를 들어, 모든 네거티브 쌍을 마진 $m$만큼만 밀어낼 뿐, 어떤 네거티브가 다른 네거티브보다 앵커에 더 가까워야 하는지와 같은 상대적인 관계는 학습하지 못한다. 이러한 한계는 더 정교한 임베딩 공간 구조를 필요로 하는 복잡한 VPR 문제에서 성능 저하의 원인이 될 수 있다.


대조 손실의 한계를 극복하기 위해 제안된 것이 바로 **삼중항 손실(Triplet Loss)**이다. Triplet Loss는 개별 쌍의 절대적인 거리가 아닌, 세 개의 샘플 간의 **상대적인 거리**에 초점을 맞춘다.30 이를 위해 학습의 기본 단위를 '쌍(pair)'에서 '삼중항(triplet)'으로 확장한다. 삼중항은 기준점이 되는 **앵커(anchor, $x_a$)**, 앵커와 같은 클래스(장소)에 속하는 **포지티브(positive, $x_p$)**, 그리고 앵커와 다른 클래스에 속하는 **네거티브(negative, $x_n$)**로 구성된다.18

Triplet Loss의 목표는 임베딩 공간에서 앵커-포지티브 간의 거리 $D(a,p)$가 앵커-네거티브 간의 거리 $D(a,n)$보다 항상 작도록 만드는 것이다. 단순히 작게 만드는 것을 넘어, 두 거리 사이에 최소한의 간격, 즉 **마진(margin, $\alpha$)**을 두도록 강제한다.26 이는 클래스 간의 분별력을 높여 더 강건한 임베딩 공간을 형성하는 데 도움을 준다. 이 제약 조건은 다음과 같이 표현할 수 있다:
$$
\|f(x_a) - f(x_p)\|_2^2 + \alpha < \|f(x_a) - f(x_n)\|_2^2
$$
이 조건을 만족시키기 위해 Triplet Loss 함수는 다음과 같이 정의된다 32:
$$
L(x_a, x_p, x_n) = \max(0, \|f(x_a) - f(x_p)\|_2^2 - \|f(x_a) - f(x_n)\|_2^2 + \alpha)
$$
이 손실 함수는 앵커-포지티브 거리가 앵커-네거티브 거리보다 마진 $\alpha$만큼 충분히 작지 않을 때만 양수의 페널티를 부과한다. 이 손실 함수는 주로 **Triplet Network** 아키텍처와 함께 사용되며, 샴 네트워크와 유사하게 세 개의 가중치를 공유하는 서브네트워크로 구성된다.31

Triplet Loss는 상대적 거리 관계를 학습함으로써 대조 손실보다 더 정교하고 구조화된 임베딩 공간을 만들 수 있다. 하지만 이는 새로운 문제를 야기하는데, 바로 '어떤 삼중항을 선택하여 학습할 것인가'라는 **삼중항 마이닝(Triplet Mining)** 문제다. 이 문제는 5장에서 자세히 다룬다.

한편, Triplet Loss를 더욱 확장하여 더 복잡한 관계를 학습하려는 시도도 있었다. 대표적인 예가 **사중항 손실(Quadruplet Loss)**이다.38 Quadruplet Loss는 기존의 삼중항(앵커, 포지티브, 네거티브1)에 두 번째 네거티브 샘플(네거티브2)을 추가한 '사중항(quadruplet)'을 사용한다.20 이를 통해 기존의 Triplet Loss 조건에 더해, 네거티브 샘플 간의 거리 관계까지 제약 조건으로 추가한다. 이는 클래스 내 분산은 더 작게, 클래스 간 분산은 더 크게 만들려는 목표를 가지고 있어, 이론적으로 더 분별력 있는 임베딩 공간을 학습할 수 있다. 하지만 이는 학습에 필요한 튜플 구성의 복잡도를 기하급수적으로 증가시키고, 효과적인 샘플링 전략을 더욱 어렵게 만드는 단점이 있다.20


쌍 기반 및 삼중항 기반 손실 함수의 가장 큰 약점은 학습에 필요한 데이터 조합의 수가 전체 데이터셋 크기 $N$에 대해 각각 $O(N^2)$과 $O(N^3)$으로 폭발적으로 증가한다는 점이다.20 이는 대규모 데이터셋에서는 효과적인 샘플링(마이닝) 전략이 필수적임을 의미하며, 이 과정 자체가 매우 비효율적이고 계산 비용이 높다. 또한, 미니배치 내의 샘플들 간의 관계만을 고려하기 때문에 데이터셋 전체의 전역적인 구조를 학습에 반영하기 어렵다는 한계도 있다.20

이러한 문제를 근본적으로 해결하기 위해 **프록시 기반 손실(Proxy-based Loss)**이 제안되었다.42 이 접근법의 핵심 아이디어는 각 클래스를 대표하는 학습 가능한 파라미터 벡터, 즉 **프록시(proxy)**를 도입하는 것이다. 학습 과정에서 각 데이터 샘플의 임베딩은 다른 모든 데이터 샘플과 직접 비교되는 대신, 이 소수의 프록시 벡터들과만 비교된다.43

예를 들어, Proxy-NCA (Neighborhood Component Analysis) 손실의 경우, 어떤 샘플 $x$가 주어졌을 때, 이 샘플이 자신의 클래스 $y$에 해당하는 프록시 $p_y$에 가까워지고, 다른 모든 클래스의 프록시 ${p_z | z \neq y}$와는 멀어지도록 학습한다. 이는 Softmax 손실과 유사한 형태로 다음과 같이 표현될 수 있다:
$$
L(x) = -\log \frac{\exp(-d(f(x), p_y))}{\sum_{p_z \in P, z \neq y} \exp(-d(f(x), p_z))}
$$
여기서 $d(\cdot, \cdot)$는 거리 함수(예: 유클리드 거리)이고, $P$는 모든 프록시의 집합이다.

프록시 기반 접근법은 샘플 대 샘플 비교를 '샘플 대 프록시' 비교로 전환함으로써, 비교 대상의 수를 전체 데이터셋 크기 $N$에서 전체 클래스 수 $C$로 획기적으로 줄인다 ($C \ll N$). 이는 삼중항 마이닝과 같은 복잡한 샘플링 과정이 필요 없어져 학습 복잡도를 크게 낮추고 수렴 속도를 높이는 결정적인 장점을 가진다.44 이 덕분에 프록시 기반 손실은 대규모 VPR 데이터셋을 효율적으로 학습하는 데 매우 적합한 방법론으로 각광받고 있다. 다만, 학습된 프록시가 해당 클래스의 데이터 분포를 얼마나 잘 대표하는지에 따라 성능이 좌우될 수 있다는 점은 고려해야 할 과제다.44


앞서 소개된 고전적인 손실 함수들 외에도, 최근 VPR 및 관련 분야에서는 다양한 형태의 손실 함수들이 제안되어 그 효과를 입증하고 있다.

**InfoNCE (Noise-Contrastive Estimation)**: 본래 자기 지도 학습(self-supervised learning) 분야에서 제안된 InfoNCE 손실은 DML에서도 널리 사용된다.26 이 손실 함수는 하나의 앵커에 대해 하나의 포지티브 샘플과 다수의 네거티브 샘플들을 동시에 고려한다. 그 목표는 포지티브 쌍의 유사도는 최대화하고, 모든 네거티브 쌍과의 유사도는 최소화하는 것이다. 이는 Softmax 함수를 기반으로 하며, 다음과 같이 정의된다:
$$
\mathcal{L}_{i} = -\log \frac{\exp(\text{sim}(z_i, z_j)/\tau)}{\sum_{k=1, k \neq i}^{N} \exp(\text{sim}(z_i, z_k)/\tau)}
$$
여기서 $z_i$는 앵커, $z_j$는 포지티브 샘플의 임베딩이며, 합산은 $N-1$개의 네거티브 샘플에 대해 이루어진다. $\text{sim}(\cdot, \cdot)$은 코사인 유사도, $\tau$는 **온도(temperature)** 하이퍼파라미터다. 온도는 유사도 분포를 얼마나 뾰족하게 만들지를 조절하며, 모델이 어려운 네거티브 샘플(hard negatives)에 더 집중하도록 만드는 역할을 한다.47 온도가 낮을수록 어려운 샘플에 더 큰 페널티를 부과하게 된다. InfoNCE는 미니배치 내의 모든 네거티브 샘플을 활용하여 학습 효율성을 높이는 장점이 있지만, 온도 $\tau$ 값에 성능이 민감하다는 단점이 있다.48

**분류 기반 손실 (Classification-based Loss)**: VPR 문제를 일종의 대규모 분류 문제로 재해석하는 접근법도 매우 효과적이다. 대표적인 예가 CosPlace 49 모델이다. CosPlace는 연속적인 지리적 좌표(GPS)를 이산적인 구역으로 나누어 각 구역을 하나의 클래스로 취급한다. 그 후, 얼굴 인식 분야에서 제안된 

**CosFace**나 **ArcFace**와 같은 각도 마진 기반의 분류 손실 함수를 사용하여 모델을 학습시킨다. 이러한 손실 함수들은 Softmax 손실을 변형하여, 임베딩 벡터와 클래스를 대표하는 가중치 벡터 사이의 각도에 직접적으로 마진을 부여한다. 이를 통해 클래스 간 분별력을 극대화하고, VPR에서 중요한 시점 변화에 대한 강건성을 높일 수 있다. 이 방식은 대규모 데이터셋에서 삼중항 마이닝 없이도 매우 효율적으로 고성능 모델을 학습시킬 수 있다는 점에서 큰 장점을 가진다.

아래 표는 본 장에서 다룬 주요 DML 손실 함수들의 특징을 요약하여 비교한 것이다.

| 손실 함수                       | 입력 단위            | 핵심 아이디어                                                | 수식 (핵심 부분)                                             | 장점                                               | 단점                                            |
| ------------------------------- | -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------- | ----------------------------------------------- |
| **Contrastive Loss**            | 쌍 (Pair)            | Positive 쌍은 가깝게, Negative 쌍은 마진 $m$ 이상 멀게       | $$(1-y)D^2 + y \max(0, m-D)^2$$                              | 단순하고 직관적임                                  | 절대적 거리에만 의존, 상대적 관계 무시          |
| **Triplet Loss**                | 삼중항 (Triplet)     | Anchor-Positive 거리가 Anchor-Negative 거리보다 마진 $\alpha$만큼 작도록 강제 | $$\max(0, D_{ap}^2 - D_{an}^2 + \alpha)$$                    | 상대적 거리를 학습하여 더 강건함                   | 삼중항 조합 폭증, 효과적인 마이닝이 필수적임    |
| **Quadruplet Loss**             | 사중항 (Quadruplet)  | Triplet 조건에 더해, Negative 쌍 간의 거리도 고려하여 클래스 내 분산 감소 | $$\max(0, D_{ap}^2 - D_{an1}^2 + \alpha_1) + \max(0, D_{ap}^2 - D_{n1n2}^2 + \alpha_2)$$ | 더 정교한 임베딩 공간 구조 학습 가능               | 샘플링 복잡도 및 계산 비용 증가                 |
| **Proxy-based Loss (ProxyNCA)** | 샘플 + 프록시        | 각 샘플을 다른 모든 샘플이 아닌, 학습 가능한 클래스 대표(프록시)와 비교 | $$-\log \frac{\exp(-d(x, p_y))}{\sum_{p_z \in P, z \neq y} \exp(-d(x, p_z))}$$ | 학습이 빠르고 효율적이며, 샘플링 복잡성 해소       | 프록시가 클래스 분포를 잘 대표하지 못할 수 있음 |
| **InfoNCE / NT-Xent**           | 앵커 + 다중 네거티브 | Softmax 기반으로 하나의 Positive와 다수의 Negative를 대조    | $$-\log \frac{\exp(s_{i,j}/\tau)}{\sum_{k \neq i} \exp(s_{i,k}/\tau)}$$ | 자기지도학습에 효과적, 하드 네거티브에 가중치 부여 | 온도 $\tau$ 하이퍼파라미터에 민감함             |


효과적인 VPR 시스템을 구축하기 위해서는 강력한 손실 함수뿐만 아니라, 이미지로부터 유의미한 특징을 추출하고 이를 검색에 용이한 형태로 압축하는 잘 설계된 딥러닝 아키텍처가 필수적이다. VPR 아키텍처는 크게 두 부분으로 나눌 수 있다: 이미지의 지역적 특징을 추출하는 **백본 네트워크(Backbone Network)**와, 이 지역적 특징들을 하나의 강건한 전역 디스크립터로 통합하는 **특징 집계 계층(Feature Aggregation Layer)**이다. 특히 이 집계 계층의 설계는 VPR 모델의 성능을 결정짓는 핵심적인 요소다.


백본 네트워크는 VPR 시스템의 '시각 피질'과 같은 역할을 하며, 입력된 이미지의 원시 픽셀 데이터로부터 계층적이고 의미 있는 특징을 담은 고차원 특징 맵(feature map)을 추출한다.

**CNN (Convolutional Neural Networks)**: 오랫동안 VPR 분야의 표준 백본으로 사용되어 온 것은 VGG 50, ResNet 50 등 ImageNet 데이터셋으로 사전 훈련된 CNN 모델들이다.1 이들 모델의 장점은 합성곱과 풀링 연산을 통해 이미지의 공간적 계층 구조(spatial hierarchy)를 자연스럽게 학습한다는 점이다. 특히, 모델의 중간 또는 마지막 합성곱 계층(예: ResNet의 

$layer3$ 또는 $layer4$, VGG의 $conv5$)에서 추출된 특징 맵은 충분한 공간 해상도를 유지하면서도 높은 수준의 의미 정보(semantic information)를 담고 있어 특징 집계 계층의 입력으로 사용하기에 이상적이다.23

**ViT (Vision Transformers)**: 최근 컴퓨터 비전 분야 전반에 걸쳐 큰 성공을 거둔 Vision Transformer(ViT)는 VPR에서도 강력한 백본으로 부상했다.1 ViT는 이미지를 여러 개의 패치(patch)로 나눈 뒤, 트랜스포머의 핵심인 셀프 어텐션(self-attention) 메커니즘을 적용한다. 이를 통해 이미지 내의 모든 패치 쌍 간의 관계를 직접 모델링함으로써, 이미지 전역에 걸친 장거리 의존성(long-range dependency)과 컨텍스트를 효과적으로 포착할 수 있다. 특히 DINOv2와 같은 대규모 데이터셋으로 사전 훈련된 Foundation Model을 백본으로 사용할 경우, 별도의 VPR 데이터셋으로 미세 조정(fine-tuning)하지 않아도 뛰어난 일반화 성능을 보이는 것으로 알려져 있다.52


백본 네트워크에서 출력된 $H \times W \times D$ 형태의 3차원 특징 맵은 수많은 지역 특징(local feature)들의 집합으로 볼 수 있다. 이를 효율적인 검색을 위해 고정된 크기의 1차원 벡터, 즉 **전역 디스크립터(global descriptor)**로 압축하는 과정이 필요하다. 이 과정을 담당하는 특징 집계 계층은 VPR 아키텍처의 성능을 좌우하는 핵심 기술이며, 그 발전 과정은 VPR 연구의 역사를 반영한다.

NetVLAD:

NetVLAD는 전통적인 이미지 검색 기법인 VLAD(Vector of Locally Aggregated Descriptors)를 미분 가능한 형태로 재설계하여 딥러닝 아키텍처에 완벽하게 통합한 혁신적인 계층이다.23 VLAD의 핵심 아이디어는 지역 특징들을 K개의 클러스터로 군집화하고, 각 클러스터에 속한 특징들과 클러스터 중심점 간의 잔차(residual)를 모두 합산하여 이미지 전체를 표현하는 것이다. NetVLAD는 이 과정을 다음과 같이 미분 가능하게 만든다 56:

1. **학습 가능한 클러스터**: K개의 클러스터 중심점 ${c_k}$를 네트워크의 학습 가능한 파라미터로 둔다.

2. **Soft-Assignment**: 각 지역 특징 $x_i$가 어떤 클러스터 $c_k$에 속하는지를 결정하는 과정에서, 가장 가까운 하나의 클러스터에만 할당하는 대신(hard-assignment), 모든 클러스터에 대해 거리에 반비례하는 가중치(soft-assignment score) $\bar{a}_k(x_i)$를 계산한다. 이 가중치는 Softmax 함수와 유사한 형태로 계산되어 미분 가능하다.

3. **가중 잔차 합산**: 각 클러스터 $k$에 대해, 모든 지역 특징 $x_i$와 중심점 $c_k$ 간의 잔차 $(x_i - c_k)$를 soft-assignment 가중치 $\bar{a}_k(x_i)$로 가중 평균하여 합산한다.

4. 벡터화 및 정규화: 이렇게 계산된 K개의 잔차 벡터를 모두 이어 붙여(concatenate) 하나의 긴 K \times D 차원의 전역 디스크립터를 생성하고, 이를 정규화하여 최종 디스크립터를 얻는다.

   NetVLAD는 VPR 작업에 특화된 방식으로 특징을 집계하도록 종단간(end-to-end) 학습이 가능하여 오랫동안 VPR 분야의 표준 아키텍처로 군림했다.

Patch-NetVLAD:

NetVLAD는 강력한 전역 디스크립터를 생성하지만, 이미지의 세부적인 공간 정보를 일부 손실한다는 한계가 있다. Patch-NetVLAD는 이러한 한계를 보완하기 위해 전역 디스크립터와 지역 디스크립터의 장점을 결합한 2단계(two-stage) 접근법을 제안한다.57

1. **1단계 (검색)**: NetVLAD와 유사한 전역 디스크립터를 사용하여 데이터베이스에서 상위 N개의 후보 이미지를 빠르게 검색한다.

2. **2단계 (재순위화)**: 쿼리 이미지와 N개의 후보 이미지 각각에 대해, 특징 맵을 여러 개의 겹치는 패치(patch)로 나누고 각 패치에 대해 지역 디스크립터를 추출한다. 이 지역 디스크립터들을 상호 정합(matching)하여 기하학적 일관성을 검증하고, 이를 기반으로 후보 이미지들의 순위를 재조정(re-ranking)한다.57

   이러한 2단계 방식은 계산 비용이 증가하는 단점이 있지만, 특히 심한 시점 변화가 있는 도전적인 시나리오에서 매우 높은 정확도를 달성할 수 있다.

GeM (Generalized Mean Pooling):

GeM은 NetVLAD의 복잡성을 대폭 줄이면서도 경쟁력 있는 성능을 제공하는 매우 단순한 풀링 기법이다.5 GeM은 특징 맵의 각 채널($D$개)에 대해 공간 차원($H \times W$)의 모든 활성값들을 일반화된 평균(generalized mean)을 이용해 하나의 값으로 집계한다. 수식은 다음과 같다:
$$
f_k = \left( \frac{1}{|\mathcal{X}_k|} \sum_{x \in \mathcal{X}_k} x^{p_k} \right)^{\frac{1}{p_k}}
$$
여기서 $\mathcal{X}_k$는 $k$번째 채널의 활성값 집합이고, $p_k$는 학습 가능한 풀링 파라미터다. 이 파라미터 $p$는 풀링의 동작을 조절하는 역할을 한다. $p=1$이면 일반적인 평균 풀링(average pooling)이 되고, $p \to \infty$이면 최대 풀링(max pooling)과 유사하게 동작한다.5 GeM은 이 파라미터 

$p$를 학습함으로써 데이터에 가장 적합한 형태의 풀링 방식을 자동으로 찾을 수 있다. 그 단순성과 효율성 덕분에 GeM은 현재 많은 VPR 모델에서 기본 집계 계층으로 널리 사용되고 있다.

MixVPR:

MixVPR은 VPR 아키텍처의 최신 진화 방향을 보여주는 사례로, All-MLP(Multi-Layer Perceptron) 구조에 기반한 새로운 집계 방식이다.63 이는 NetVLAD의 클러스터링이나 GeM의 공간적 풀링과 같은 전통적인 집계 개념에서 벗어나, 특징 맵 내의 전역적인 관계를 직접 학습하는 데 초점을 맞춘다. MixVPR의 핵심 아이디어는 MLP-Mixer 아키텍처에서 영감을 얻어, 특징 맵을 '토큰'의 집합으로 간주하고 이들 간의 정보를 반복적으로 '혼합(mixing)'하는 것이다.63

1. **특징 맵 평탄화**: 입력된 $H \times W \times D$ 특징 맵을 $(HW) \times D$ 형태로 변형한다.

2. **Feature-Mixer 블록**: 이 블록은 두 개의 MLP 레이어로 구성된다. 첫 번째 MLP는 채널 방향(channel-wise)으로 작동하여 각 공간 위치(패치) 내의 특징들을 혼합하고, 두 번째 MLP는 공간 방향(token-wise)으로 작동하여 다른 위치에 있는 특징들 간의 정보를 혼합한다.

3. 반복적 혼합: 이 Feature-Mixer 블록을 여러 개 쌓아(cascade) 특징 혼합 과정을 반복적으로 수행한다.

   이러한 과정을 통해 MixVPR은 별도의 지역적 집계 과정 없이도 특징 맵 전체에 걸친 공간적, 채널적 상관관계를 학습하여 매우 압축적이면서도 강력한 전역 디스크립터를 생성할 수 있다. 그 결과, MixVPR은 기존 방법들보다 훨씬 적은 파라미터와 빠른 속도로 최첨단(SOTA) 성능을 달성했다.64 이는 VPR에 필요한 핵심 정보가 이미지의 특정 '지역'에 국한된 것이 아니라, 전체 특징들 간의 복잡한 '상관관계'에 있음을 시사한다.

아래 표는 본 장에서 다룬 주요 VPR 특징 집계 아키텍처들의 특징을 요약하여 비교한 것이다.

| 아키텍처          | 핵심 메커니즘                                                | 생성 디스크립터        | 장점                                                         | 단점                                                         |
| ----------------- | ------------------------------------------------------------ | ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **NetVLAD**       | 학습 가능한 K개의 클러스터를 이용한 지역 특징의 잔차(residual) 합산 (Soft-assignment) | 전역 디스크립터        | 표현력이 매우 높고, VPR 작업에 특화된 학습 가능              | 파라미터 수가 많고(특히 클러스터 수 K에 비례), 디스크립터 차원이 큼 |
| **Patch-NetVLAD** | NetVLAD 잔차로부터 패치 수준의 디스크립터를 추출하고, 이를 이용한 2단계 정합 및 재순위화 | 전역 + 지역 디스크립터 | 시점 변화에 매우 강건하며, 정확도가 높음                     | 2단계 프로세스로 인해 추론 시간이 길고 복잡함                |
| **GeM Pooling**   | 학습 가능한 파라미터 $p$를 이용한 특징 맵의 일반화된 평균 계산 | 전역 디스크립터        | 매우 단순하고 가벼우며, 구현이 용이함. 경쟁력 있는 성능 제공 | NetVLAD만큼의 표현력을 갖지 못할 수 있으며, 공간 정보 손실이 큼 |
| **MixVPR**        | MLP-Mixer 구조를 활용하여 특징 맵의 패치와 채널을 교차로 혼합(mixing) | 전역 디스크립터        | 매우 가볍고 빠르면서 SOTA 성능 달성. 지역적 집계 불필요      | 메커니즘의 해석이 상대적으로 어려움                          |


이론적으로 우수한 손실 함수와 아키텍처를 설계하는 것만큼이나 중요한 것은 이를 실제로 효과적으로 훈련시키는 파이프라인을 구축하는 것이다. 특히 Triplet Loss와 같이 샘플링이 중요한 역할을 하는 DML 모델의 경우, 훈련 데이터의 구성 방식과 훈련 샘플을 선택하는 전략이 모델의 최종 성능을 좌우한다.


VPR 훈련 데이터셋은 기본적으로 지리적 좌표(주로 GPS)가 태깅된 대규모 이미지 컬렉션으로 구성된다.1 이 연속적인 좌표 정보로부터 DML 학습에 필요한 이산적인 '유사/비유사' 레이블을 생성하는 과정이 필요하다.

일반적으로 두 이미지의 촬영 위치 간의 물리적 거리를 기준으로 포지티브(positive)와 네거티브(negative) 쌍을 정의한다. 예를 들어, 두 이미지의 GPS 좌표 간 거리가 특정 임계값(예: 25미터) 이내이면 포지티브 쌍으로 간주하고, 그보다 훨씬 먼 거리(예: 50미터) 밖에 있으면 네거티브 쌍으로 간주한다.67 이 임계값 사이의 애매한 거리에 있는 이미지들은 훈련에서 제외하여 레이블의 모호성을 줄이기도 한다.

대표적인 대규모 훈련 데이터셋으로는 Mapillary Street-Level Sequences (MSLS), GSV-Cities, 그리고 훈련과 평가에 모두 사용되는 Pittsburgh 30k 등이 있다.1 이 데이터셋들은 전 세계 여러 도시와 시골의 다양한 환경, 시간대, 계절 변화를 포함하고 있어, 모델이 다양한 조건에 대해 일반화된 표현을 학습하는 데 필수적인 데이터를 제공한다.


3장에서 언급했듯이, Triplet Loss 기반 모델을 훈련할 때 데이터셋에 존재하는 모든 가능한 삼중항을 사용하는 것은 계산적으로 불가능할 뿐만 아니라 비효율적이다. 학습이 진행됨에 따라 대부분의 삼중항은 이미 손실 함수의 제약 조건($D(a,p) + \alpha < D(a,n)$)을 만족하게 되어 손실 값이 0이 된다. 이러한 '쉬운(easy)' 삼중항들은 더 이상 학습에 기여하지 못하는 '쓰레기 그래디언트'를 생성할 뿐이다.30 따라서 모델이 계속해서 학습하고 성능을 개선하기 위해서는, 현재 모델의 입장에서 '어려운' 삼중항을 효과적으로 선별하여 집중적으로 학습하는 과정, 즉 **삼중항 마이닝(Triplet Mining)**이 필수적이다.

삼중항 마이닝은 단순한 데이터 샘플링 기법을 넘어, 모델의 학습 상태에 맞춰 동적으로 '학습 커리큘럼'을 설계하는 과정으로 이해할 수 있다. 모델이 초기에는 비교적 쉬운 예제로 기본적인 분별력을 갖추고, 점차 더 까다롭고 미묘한 차이를 구분해야 하는 어려운 예제에 도전하도록 유도하는 것이다.

삼중항의 종류:

앵커-포지티브 거리 D(a,p)와 앵커-네거티브 거리 D(a,n)를 기준으로 삼중항은 다음과 같이 세 가지 유형으로 분류할 수 있다.70

1. **Easy Triplets**: $D(a,p) + \alpha < D(a,n)$. 손실이 0이며, 제약 조건을 이미 크게 만족하는 삼중항이다. 학습에 전혀 기여하지 못한다.
2. **Hard Triplets**: $D(a,n) < D(a,p)$. 네거티브 샘플이 포지티브 샘플보다 앵커에 더 가까운, 매우 어려운 경우다. 이러한 샘플들은 모델에 가장 많은 정보를 제공할 수 있지만, 학습 초기에 너무 많은 hard triplets에 집중하면 모델이 불안정해지거나 잘못된 지역 최적해(local minima)에 빠져 수렴에 실패할 수 있다.72 이는 마치 초보자에게 너무 어려운 문제만 계속 풀게 하여 학습 의욕을 꺾는 것과 같다.
3. **Semi-hard Triplets**: $D(a,p) < D(a,n) < D(a,p) + \alpha$. 네거티브 샘플이 포지티브보다는 멀리 있지만, 마진 $\alpha$ 이내에 있어 여전히 0보다 큰 손실을 발생시키는 경우다. 이들은 현재 모델이 '거의' 맞출 수 있지만 약간의 노력이 더 필요한, 즉 학습에 가장 효과적인 '도전적인' 문제로 간주된다. 많은 연구에서 semi-hard triplets을 선택하는 것이 안정적이면서도 효율적인 학습을 이끈다고 보고하고 있다.30

마이닝 전략:

삼중항을 선택하는 전략은 크게 오프라인과 온라인 방식으로 나뉜다.

- **오프라인(Offline) 마이닝**: 전체 훈련 데이터셋에 대해 현재 모델의 임베딩을 주기적으로(예: 매 에포크마다) 모두 계산한다. 이 전체 임베딩을 바탕으로 유용한 삼중항(예: semi-hard triplets)을 선별하여 다음 에포크의 훈련 데이터로 사용한다. 가장 이상적인 삼중항을 찾을 수 있지만, 대규모 데이터셋에서는 임베딩을 계산하고 저장하는 비용이 엄청나게 크기 때문에 거의 사용되지 않는다.73
- **온라인(Online) 마이닝**: 현재 딥러닝 프레임워크에서 표준적으로 사용되는 방식으로, 각 훈련 스텝에서 구성된 미니배치(mini-batch) 내에서 동적으로 삼중항을 생성한다.71 이는 계산적으로 매우 효율적이며, 모델의 최신 상태를 즉시 반영하여 삼중항을 선택할 수 있다는 장점이 있다. 주요 온라인 마이닝 전략은 다음과 같다.
  - **Batch All**: 미니배치 내에서 가능한 모든 유효한(non-zero loss) 삼중항을 생성하고, 이들의 손실을 모두 평균내어 역전파한다.69 모든 정보를 활용하지만, 쉬운 삼중항의 영향이 여전히 클 수 있다.
  - **Batch Hard**: 각 앵커 샘플에 대해, 미니배치 내에서 가장 어려운 포지티브(앵커로부터 가장 먼 포지티브, $\arg\max_p D(a,p)$)와 가장 어려운 네거티브(앵커에 가장 가까운 네거티브, $\arg\min_n D(a,n)$)를 선택하여 삼중항을 구성한다.71 이 방식은 가장 정보량이 많은 샘플에 집중하여 학습을 가속화할 수 있다.
  - **Semi-hard Negative Mining**: 각 앵커-포지티브 쌍에 대해, 미니배치 내의 네거티브 샘플 중에서 semi-hard 조건을 만족하는 샘플들을 무작위로 선택하는 방식이다. 이는 FaceNet 논문에서 제안되어 널리 사용되는 안정적인 전략이다.30

효과적인 삼중항 마이닝 전략을 선택하고 구현하는 것은 Triplet Loss 기반의 DML 모델을 성공적으로 훈련시키는 데 있어 가장 중요한 실용적인 기술 중 하나이며, 모델의 최종 성능과 수렴 속도에 지대한 영향을 미친다.


성공적으로 훈련된 VPR 모델의 성능을 객관적으로 평가하고, 다른 모델들과 공정하게 비교하기 위해서는 표준화된 평가 지표와 벤치마크 데이터셋이 필수적이다. 또한, 빠르게 발전하는 VPR 분야의 최신 연구 동향을 파악하는 것은 현재 기술의 수준을 이해하고 미래의 연구 방향을 예측하는 데 중요하다.


VPR 시스템의 성능을 평가하는 가장 보편적이고 표준적인 지표는 **Recall@N**이다.74 이는 이미지 검색 분야에서 널리 사용되는 평가 척도로, VPR의 '검색'이라는 본질적인 특성을 잘 반영한다.

Recall@N의 정의:

Recall@N은 전체 쿼리 이미지 중에서, 검색 결과 상위 N개의 이미지 안에 정답(ground truth) 이미지가 하나 이상 포함된 쿼리의 비율을 의미한다.74 여기서 '정답'은 일반적으로 쿼리 이미지와 특정 거리 임계값(예: 25미터) 이내에서 촬영된 데이터베이스 이미지를 말한다.

**Recall@N의 해석**:

- **Recall@1 (R@1)**: 검색 결과에서 가장 유사하다고 예측한 첫 번째 이미지가 정답일 확률을 나타낸다.75 이는 가장 엄격한 척도로, VPR 시스템이 얼마나 정확하게 '정답'을 찾아내는지를 직접적으로 보여준다. 단독으로 사용되는 VPR 시스템의 성능을 평가하는 데 중요한 지표다.
- **Recall@5, Recall@10,...**: 검색 결과 상위 5개, 10개 안에 정답이 포함될 확률을 나타낸다. 이 지표들은 VPR 시스템이 후속 단계(예: Patch-NetVLAD의 재순위화 단계나 SLAM의 기하학적 검증 단계)를 위한 '좋은 후보군'을 얼마나 잘 제공하는지를 평가하는 데 유용하다. R@1은 낮더라도 R@5나 R@10이 높다면, 해당 시스템은 재순위화 모듈과 결합되었을 때 높은 최종 성능을 보일 잠재력이 있음을 의미한다.

VPR 관련 논문에서는 보통 R@1, R@5, R@10 값을 함께 제시하여 모델의 성능을 다각적으로 보여주는 것이 일반적이다.


VPR 모델의 강건성과 일반화 성능을 평가하기 위해, 다양한 실제 환경의 어려움을 담고 있는 여러 벤치마크 데이터셋이 구축되어 사용되고 있다.

- **클래식 벤치마크 데이터셋**:

  - **Pittsburgh 30k (Pitts30k)**: 미국 피츠버그 시내에서 수집된 대규모 스트리트 뷰 이미지로 구성된 데이터셋이다.9 이 데이터셋의 주요 특징은 도심 환경 특유의 반복적인 건물 구조와 유사한 거리 풍경으로 인해 

    **인식 모호성(perceptual aliasing)** 문제가 두드러진다는 점이다. 따라서 Pitts30k에서의 높은 성능은 모델이 미세한 차이를 구별해내는 분별력을 갖추었음을 의미한다.

  - **Tokyo 24/7**: 일본 도쿄에서 수집된 데이터셋으로, 이름에서 알 수 있듯이 하루 24시간, 일주일 내내의 다양한 시간대(주간, 야간, 황혼 등) 이미지를 포함하고 있다.68 이 데이터셋은 극심한 

    **조명 변화**에 대한 모델의 강건성을 평가하는 데 주로 사용된다.

- **기타 주요 데이터셋**:

  - **Nordland**: 노르웨이의 한 철도 노선을 따라 봄, 여름, 가을, 겨울 네 계절에 걸쳐 촬영된 비디오로 구성된 데이터셋이다.74 극심한 

    **계절 변화**에 따른 외형 변화를 평가하는 데 가장 도전적인 데이터셋 중 하나로 꼽힌다.

  - **Mapillary Street-Level Sequences (MSLS)**: 전 세계 여러 도시와 대륙에서 다양한 카메라(스마트폰, 전문 장비 등)로 촬영된 160만 장 이상의 이미지로 구성된 초대규모 데이터셋이다.68 MSLS는 데이터의 

    **다양성**과 **규모** 면에서 독보적이며, 이 데이터셋에서의 성능은 모델이 학습 데이터에 과적합되지 않고 새로운, 보지 못한 환경에 얼마나 잘 **일반화**되는지를 평가하는 중요한 척도가 된다.


최근 VPR 연구는 몇 가지 뚜렷한 흐름을 보이며 빠르게 발전하고 있다. 특히, 거대 언어 모델(LLM)의 성공에 힘입어 시각 분야에서도 강력한 성능을 보이는 **Foundation Model**의 등장은 VPR 연구의 패러다임을 바꾸고 있다.

Foundation Model의 적극적인 활용:

DINOv2와 같은 대규모의 레이블 없는 데이터로 자기 지도 학습(self-supervised learning)을 통해 훈련된 Foundation Model은 이미지에 대한 매우 풍부하고 일반화된 특징 표현을 학습한다. 최신 VPR 연구들은 이러한 사전 훈련된 모델을 강력한 백본으로 채택하고, VPR 특정 데이터셋으로 가볍게 미세 조정(fine-tuning)하거나 심지어는 미세 조정 없이(zero-shot) 사용하는 것만으로도 기존의 SOTA 모델들을 뛰어넘는 성능을 달성하고 있다.53 이는 VPR 모델 개발의 초점이 '처음부터 좋은 아키텍처를 설계하는 것'에서 '강력한 외부 지식을 VPR 작업에 어떻게 효과적으로 적용하고 적응시킬 것인가'로 이동하고 있음을 보여준다.

최첨단(SOTA) 모델 분석:

2023년과 2024년에 발표된 주요 모델들은 이러한 트렌드를 명확히 보여준다.

- **EigenPlaces (ICCV 2023)**: 주성분 분석(PCA)에서 영감을 얻어, VPR에서 중요한 특징 방향을 학습하여 시점 변화에 강건한 표현을 만드는 데 초점을 맞춘 모델이다.77
- **MixVPR (WACV 2023)**: 4.2절에서 설명했듯이, 경량 MLP 기반의 효율적인 집계 모듈을 제안하여 성능과 속도 두 마리 토끼를 모두 잡았다.63
- **AnyLoc (ArXiv 2023)**: Foundation Model(DINOv2, CLIP 등)을 활용하여 특정 데이터셋에 국한되지 않는 범용적인(universal) VPR 성능을 목표로 하는 연구의 대표적인 사례다.77
- **CricaVPR (CVPR 2024)**: 미니배치 내의 여러 이미지들 간의 상호 관계(cross-image correlation)를 어텐션 메커니즘을 통해 명시적으로 모델링한다. 이를 통해 같은 장소의 다른 시점 이미지들로부터는 공통점을, 다른 장소의 이미지들로부터는 차이점을 학습하여 표현의 강건성을 높인다.77
- **SegVLAD / Revisit Anything (ECCV 2024)**: VPR의 패러다임을 전환하는 혁신적인 시도다. 이 접근법은 이미지를 하나의 전체 단위로 보는 기존의 방식을 버린다. 대신, SAM(Segment Anything Model)과 같은 최신 분할 모델을 사용하여 이미지를 의미 있는 여러 개의 '세그먼트(segment)'(예: 건물, 나무, 하늘)로 분해한다. 그 후, 이 세그먼트 단위의 디스크립터를 생성하고 검색을 수행한다.80 이 방식은 두 이미지가 부분적으로만 겹치는 심한 시점 변화 상황에서, 겹치지 않는 영역이 유사도 계산을 방해하는 근본적인 문제를 해결할 수 있는 잠재력을 가진다. 이는 "두 이미지가 같은 장소인가?"라는 질문을 "두 이미지가 같은 사물들을 비슷한 맥락에서 포함하고 있는가?"라는 더 정교한 질문으로 바꾸는 것이다.

아래 표는 주요 벤치마크 데이터셋에 대한 최신 SOTA 모델들의 성능(R@1)을 요약한 것이다. 이를 통해 기술의 발전 추세와 Foundation Model의 영향을 명확히 확인할 수 있다.

| 모델명            | Pittsburgh 30k (R@1) | Tokyo 24/7 (R@1) | MSLS (R@1) | 백본 아키텍처 | 발표 연도 |
| ----------------- | -------------------- | ---------------- | ---------- | ------------- | --------- |
| NetVLAD 54        | 86.08                | -                | -          | VGG-16        | 2016      |
| Patch-NetVLAD 57  | 88.7                 | -                | -          | VGG-16        | 2021      |
| CosPlace 49       | 90.45                | 87.2             | 83.1       | ResNet-50     | 2022      |
| MixVPR 63         | 91.52                | 89.2             | 88.0       | ResNet-50     | 2023      |
| EigenPlaces 77    | 92.5                 | 91.4             | 88.8       | ResNet-50     | 2023      |
| AnyLoc-DINOv2 77  | 87.66                | -                | 89.7       | DINOv2        | 2023      |
| CricaVPR 79       | -                    | 93.7             | 91.3       | -             | 2024      |
| SegVLAD-DINOv2 80 | 93.1                 | -                | 91.9       | DINOv2        | 2024      |
| MegaLoc 67        | **94.8**             | **94.6**         | **93.5**   | -             | 2024      |


본 자습서는 Visual Place Recognition(VPR)을 위한 딥 메트릭 러닝(DML)의 핵심 원리부터 최신 기술 동향까지를 심도 있게 탐구했다. VPR은 자율 주행, 로보틱스, SLAM 등 현대 AI 시스템의 핵심적인 공간 인식 능력으로, 극심한 외형 및 시점 변화, 그리고 인식 모호성이라는 근본적인 도전 과제를 안고 있다. DML은 이러한 문제에 대한 강력한 해결책을 제시하며 VPR 기술의 비약적인 발전을 이끌었다.

DML의 핵심은 '유사성'을 직접 학습하는 임베딩 공간을 구축하는 것이다. 이를 위해 손실 함수는 단순한 쌍 기반의 대조 손실에서부터 상대적 거리를 학습하는 삼중항 손실, 그리고 대규모 데이터 학습의 효율성을 극대화한 프록시 및 분류 기반 손실로 진화해왔다. 아키텍처 측면에서는 수작업 기법에서 영감을 받은 NetVLAD를 시작으로, 단순성과 효율성을 추구한 GeM, 그리고 특징 간의 전역적 관계를 학습하는 MLP 기반의 MixVPR에 이르기까지, 더 강건하고 효율적인 전역 디스크립터를 생성하기 위한 연구가 지속되었다.

최근 VPR 연구는 DINOv2와 같은 강력한 Foundation Model의 사전 지식을 효과적으로 활용하고, 나아가 '전체 이미지'라는 표현의 기본 단위를 해체하여 세그먼트 기반으로 장소를 인식하려는 새로운 패러다임으로 나아가고 있다. 이는 VPR이 단순한 이미지 검색을 넘어, 더 깊은 수준의 장면 이해(scene understanding)와 결합되고 있음을 시사한다.

이러한 기술적 흐름을 바탕으로, 딥 메트릭 러닝 기반 VPR의 미래는 다음과 같은 방향으로 전개될 것으로 전망된다.

- **의미론적 및 다중 모달 VPR (Semantic & Multi-modal VPR)**: 현재의 VPR은 주로 이미지의 시각적 패턴에 의존한다. 미래의 VPR은 이미지 내의 객체나 장면에 대한 의미론적 정보(예: '이곳은 상점가', '저 건물은 교회')를 명시적으로 활용하거나, 텍스트(예: 간판), LiDAR의 3D 구조 정보, 오디오 등 다른 종류의 센서 데이터를 융합하여 인식 모호성을 근본적으로 해결하고 강건성을 한 차원 높이는 방향으로 발전할 것이다.5
- **효율성과 실시간성 (Efficiency & Real-time Performance)**: 자율 주행차나 로봇뿐만 아니라, 스마트폰, AR 글래스와 같은 저전력 모바일 기기에서도 VPR 기능이 중요해짐에 따라, 모델 경량화, 양자화, 지식 증류 등을 통해 계산 효율성을 극대화하고 실시간성을 보장하는 연구의 중요성이 더욱 커질 것이다.
- **자기 지도 및 약지도 학습 (Self-supervised & Weakly-supervised Learning)**: 대규모의 정확한 지리 정보 태깅 데이터베이스를 구축하는 것은 비용과 시간이 많이 소요된다. Google Street View Time Machine 데이터를 활용한 NetVLAD의 약지도 학습 51과 같이, 명시적인 레이블 없이도 대규모 데이터로부터 장소 표현을 학습할 수 있는 자기 지도 및 약지도 학습 방법론에 대한 연구가 더욱 활발해질 것이다.
- **신뢰성 및 불확실성 추정 (Reliability & Uncertainty Estimation)**: 안전이 최우선인 자율 시스템에서 VPR의 예측이 실패했을 때의 위험은 매우 크다. 따라서 VPR 시스템이 자신의 예측 결과를 얼마나 신뢰할 수 있는지, 즉 불확실성(uncertainty)을 정량적으로 추정하는 기술이 중요해질 것이다.7 이를 통해 시스템은 VPR 결과가 불확실할 경우, 다른 센서에 더 의존하거나 안전한 경로를 선택하는 등 지능적인 의사결정을 내릴 수 있게 된다.
- **연합 학습 (Federated Learning)**: 사용자의 위치 정보와 이미지는 민감한 개인 정보를 포함한다. 연합 학습은 이러한 데이터를 중앙 서버로 전송하지 않고, 각 사용자의 기기(클라이언트)에서 모델을 로컬하게 학습한 뒤, 학습된 모델의 일부(예: 그래디언트)만을 안전하게 공유하여 전역 모델을 업데이트하는 분산 학습 패러다임이다.83 이는 개인 정보 보호 문제를 해결하면서도 다양한 환경의 데이터를 활용하여 VPR 모델을 공동으로 훈련할 수 있는 새로운 가능성을 열어줄 것이다.

결론적으로, 딥 메트릭 러닝은 VPR 분야에 혁신을 가져왔으며, 앞으로도 Foundation Model, 다중 모달리티, 신뢰성 공학과 같은 새로운 기술들과 융합하며 기계의 공간 인식 능력을 인간의 수준에 가깝게 끌어올리는 데 핵심적인 역할을 계속해 나갈 것이다.


1. Visual Place Recognition: Building an Evaluation Framework for Model Robustness - University of Twente, 8월 18, 2025에 액세스, http://essay.utwente.nl/100860/1/Gosa_BA_EEMCS.pdf
2. pmc.ncbi.nlm.nih.gov, 8월 18, 2025에 액세스, [https://pmc.ncbi.nlm.nih.gov/articles/PMC9394449/#:~:text=Visual%20place%20recognition%20(VPR)%20is%20the%20process%20of%20retrieving%20the,mapping%2C%20localization%2C%20and%20navigation.](https://pmc.ncbi.nlm.nih.gov/articles/PMC9394449/#:~:text=Visual place recognition (VPR) is the process of retrieving the,mapping%2C localization%2C and navigation.)
3. Where Is Your Place, Visual Place Recognition? - IJCAI, 8월 18, 2025에 액세스, https://www.ijcai.org/proceedings/2021/0603.pdf
4. (PDF) Visual Place Recognition: A Tutorial - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/374128870_Visual_Place_Recognition_A_Tutorial
5. Optimal Transport Aggregation for Visual Place Recognition - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2311.15937v1
6. An Overview of Visual Place Recognition Based on Street View Images, 8월 18, 2025에 액세스, https://www.dqxxkx.cn/EN/10.12082/dqxxkx.2025.250137
7. Visual Place Recognition (VPR) is the ability to recognize one's... - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/figure/sual-Place-Recognition-VPR-is-the-ability-to-recognize-ones-location-based-on_fig1_353837436
8. NYU-VPR: Long-Term Visual Place Recognition Benchmark with View Direction and Data Anonymization Influences, 8월 18, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC9394449/
9. Regressing Transformers for Data-efficient Visual Place Recognition - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2401.16304v1
10. Towards a Robust Visual Place Recognition in Large-Scale vSLAM Scenarios Based on a Deep Distance Learning - MDPI, 8월 18, 2025에 액세스, https://www.mdpi.com/1424-8220/21/1/310
11. SVS-VPR: A Semantic Visual and Spatial Information-Based Hierarchical Visual Place Recognition for Autonomous Navigation in Challenging Environmental Conditions - MDPI, 8월 18, 2025에 액세스, https://www.mdpi.com/1424-8220/24/3/906
12. Long-term Visual Place Recognition Under Varying Conditions - Trepo, 8월 18, 2025에 액세스, https://trepo.tuni.fi/handle/10024/151007
13. Focus on Local: Finding Reliable Discriminative Regions for Visual Place Recognition | Proceedings of the AAAI Conference on Artificial Intelligence, 8월 18, 2025에 액세스, https://ojs.aaai.org/index.php/AAAI/article/view/32811
14. Perceptual aliasing. Two distant locations in a building share similar... - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/figure/Perceptual-aliasing-Two-distant-locations-in-a-building-share-similar-visual-appearance_fig1_321815533
15. Visual Place Recognition. Introduction | by SOMIK DHAR - Medium, 8월 18, 2025에 액세스, https://medium.com/@sd5023/visual-place-recognition-8999307ebb2f
16. SuperPlace: The Renaissance of Classical Feature Aggregation for Visual Place Recognition in the Era of Foundation Models - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2506.13073v1
17. Deep Visual Similarity and Metric Learning | Your awesome title, 8월 18, 2025에 액세스, https://dvsml2022-tutorial.github.io/
18. Deep Metric Learning: a (Long) Survey | Chan Kha Vu, 8월 18, 2025에 액세스, https://hav4ik.github.io/articles/deep-metric-learning-survey/
19. Long-term Visual Place Recognition, 8월 18, 2025에 액세스, https://webpages.tuni.fi/vision/public_data/publications/icpr2022_longterm_visual_place_recognition.pdf
20. Deep Metric Learning with Hierarchical Triplet Loss - CVF Open Access, 8월 18, 2025에 액세스, https://openaccess.thecvf.com/content_ECCV_2018/papers/Ge_Deep_Metric_Learning_ECCV_2018_paper
21. Siamese Networks. Concepts: | by Saba Hesaraki - Medium, 8월 18, 2025에 액세스, https://medium.com/@saba99/siamese-networks-e0101129d607
22. Deep Metric Learning: A Survey - MDPI, 8월 18, 2025에 액세스, https://www.mdpi.com/2073-8994/11/9/1066
23. NetVLAD: CNN Architecture for Weakly Supervised Place Recognition - CVF Open Access, 8월 18, 2025에 액세스, https://openaccess.thecvf.com/content_cvpr_2016/papers/Arandjelovic_NetVLAD_CNN_Architecture_CVPR_2016_paper.pdf
24. Siamese Network — Deep Learning - FR, 8월 18, 2025에 액세스, https://perso.esiee.fr/~chierchg/deep-learning/tutorials/metric/metric-1.html
25. What is a Siamese network in deep learning? - Milvus, 8월 18, 2025에 액세스, https://milvus.io/ai-quick-reference/what-is-a-siamese-network-in-deep-learning
26. Day 3 — Loss Functions in Contrastive Learning: A Deep Dive | by Deepali Mishra - Medium, 8월 18, 2025에 액세스, https://medium.com/@deepsiya10/day-3-loss-functions-in-contrastive-learning-a-deep-dive-426a38525e2d
27. Siamese neural network - Wikipedia, 8월 18, 2025에 액세스, https://en.wikipedia.org/wiki/Siamese_neural_network
28. CPMetric: Deep Siamese Networks for Metric Learning on Structured Preferences | OpenReview, 8월 18, 2025에 액세스, https://openreview.net/forum?id=a12Vuyr1bl
29. [1809.08350] CPMetric: Deep Siamese Networks for Learning Distances Between Structured Preferences - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/1809.08350
30. Triplet loss - Wikipedia, 8월 18, 2025에 액세스, https://en.wikipedia.org/wiki/Triplet_loss
31. Deep metric learning using Triplet network | alphaXiv, 8월 18, 2025에 액세스, https://www.alphaxiv.org/overview/1412.6622v4
32. The Group Loss for Deep Metric Learning - European Computer Vision Association, 8월 18, 2025에 액세스, https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123520273.pdf
33. A Triplet-loss Dilated Residual Network for High-Resolution Representation Learning in Image Retrieval - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/pdf/2303.08398
34. arXiv:2202.07474v1 [cs.CV] 14 Feb 2022, 8월 18, 2025에 액세스, https://arxiv.org/pdf/2202.07474
35. [2303.08293] Quantum adversarial metric learning model based on triplet loss function, 8월 18, 2025에 액세스, https://arxiv.org/abs/2303.08293
36. [2401.02708] TripleSurv: Triplet Time-adaptive Coordinate Loss for Survival Analysis - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2401.02708
37. eladhoffer/TripletNet: Deep metric learning using Triplet network - GitHub, 8월 18, 2025에 액세스, https://github.com/eladhoffer/TripletNet
38. Quadruplet Loss For Improving the Robustness to Face Morphing Attacks - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/pdf/2402.14665
39. [1703.07026] Cross-modal Deep Metric Learning with Multi-task Regularization - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/1703.07026
40. Quadruplet Loss For Improving the Robustness to Face Morphing Attacks - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2402.14665v1
41. [2107.03786] Deep Metric Learning Model for Imbalanced Fault Diagnosis - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2107.03786
42. Towards Improved Proxy-Based Deep Metric Learning via Data-Augmented Domain Adaptation, 8월 18, 2025에 액세스, https://ojs.aaai.org/index.php/AAAI/article/view/29400/30645
43. A Non-isotropic Probabilistic Take on Proxy-based Deep Metric Learning - European Computer Vision Association, 8월 18, 2025에 액세스, https://www.ecva.net/papers/eccv_2022/papers_ECCV/papers/136860423.pdf
44. [2304.09162] Robust Calibrate Proxy Loss for Deep Metric Learning - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2304.09162
45. arXiv:2401.00617v1 [cs.CV] 1 Jan 2024, 8월 18, 2025에 액세스, https://arxiv.org/pdf/2401.00617
46. [2103.13538] Hierarchical Proxy-based Loss for Deep Metric Learning - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2103.13538
47. Understanding the Behaviour of Contrastive Loss - CVF Open Access, 8월 18, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2021/papers/Wang_Understanding_the_Behaviour_of_Contrastive_Loss_CVPR_2021_paper.pdf
48. [2501.17683] Temperature-Free Loss Function for Contrastive Learning - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2501.17683
49. Distributed training of CosPlace for large-scale visual place recognition - Frontiers, 8월 18, 2025에 액세스, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2024.1386464/full
50. Effective triplet mining improves training of multi-scale pooled CNN for image retrieval - FLORE, 8월 18, 2025에 액세스, https://flore.unifi.it/retrieve/e398c382-4e0d-179a-e053-3705fe0a4cff/Vaccaro2022_Article_EffectiveTripletMiningImproves.pdf
51. NetVLAD: CNN architecture for weakly supervised place recognition, 8월 18, 2025에 액세스, https://www.di.ens.fr/~josef/publications/Arandjelovic17.pdf
52. [2212.04114] Group Generalized Mean Pooling for Vision Transformer - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2212.04114
53. EffoVPR: Effective Foundation Model Utilization for Visual Place Recognition - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2405.18065v1
54. NetVLAD: CNN architecture for weakly supervised place recognition - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/284578950_NetVLAD_CNN_architecture_for_weakly_supervised_place_recognition
55. NetVLAD: CNN architecture for weakly supervised place recognition, 8월 18, 2025에 액세스, https://www.di.ens.fr/willow/research/netvlad/
56. NetVLAD: CNN Architecture for Weakly Supervised Place Recognition - Medium, 8월 18, 2025에 액세스, https://medium.com/data-science/netvlad-cnn-architecture-for-weakly-supervised-place-recognition-ce64b08bebaf
57. Patch-NetVLAD: Multi-Scale Fusion of Locally-Global Descriptors for Place Recognition - CVF Open Access, 8월 18, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2021/papers/Hausler_Patch-NetVLAD_Multi-Scale_Fusion_of_Locally-Global_Descriptors_for_Place_Recognition_CVPR_2021_paper.pdf
58. Patch-NetVLAD: Multi-Scale Fusion of Locally-Global Descriptors for Place Recognition, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/349727746_Patch-NetVLAD_Multi-Scale_Fusion_of_Locally-Global_Descriptors_for_Place_Recognition
59. [2103.01486] Patch-NetVLAD: Multi-Scale Fusion of Locally-Global Descriptors for Place Recognition - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2103.01486
60. PlaceFormer: Transformer-based Visual Place Recognition using Multi-Scale Patch Selection and Fusion - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2401.13082v2
61. [2202.05738] Patch-NetVLAD+: Learned patch descriptor and weighted matching strategy for place recognition - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2202.05738
62. [1811.00202] Attention-Aware Generalized Mean Pooling for Image Retrieval - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/1811.00202
63. MixVPR: Feature Mixing for Visual Place Recognition - CVF Open Access, 8월 18, 2025에 액세스, https://openaccess.thecvf.com/content/WACV2023/papers/Ali-bey_MixVPR_Feature_Mixing_for_Visual_Place_Recognition_WACV_2023_paper.pdf
64. [2303.02190] MixVPR: Feature Mixing for Visual Place Recognition - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2303.02190
65. MixVPR: Feature Mixing for Visual Place Recognition (WACV 2023) - GitHub, 8월 18, 2025에 액세스, https://github.com/amaralibey/MixVPR
66. Enhancing Visual Place Recognition With Hybrid Attention Mechanisms in MixVPR, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/385330088_Enhancing_Visual_Place_Recognition_with_Hybrid_Attention_Mechanisms_in_MixVPR
67. MegaLoc: One Retrieval to Place Them All - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2502.17237v1
68. Inside Out Visual Place Recognition - BMVC 2021, 8월 18, 2025에 액세스, https://www.bmvc2021-virtualconference.com/assets/papers/0467.pdf
69. Siamese and triplet networks with online pair/triplet mining in PyTorch - GitHub, 8월 18, 2025에 액세스, https://github.com/adambielski/siamese-triplet
70. Semi-Hard Triplet Mining - Artificial Intelligence, 8월 18, 2025에 액세스, https://schneppat.com/semi-hard-triplet-mining.html
71. Triplet Loss: Intro, Implementation, Use Cases - V7 Labs, 8월 18, 2025에 액세스, https://www.v7labs.com/blog/triplet-loss
72. [2007.12749] Hard negative examples are hard, but useful - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/abs/2007.12749
73. Hard Triplet Mining - Artificial Intelligence, 8월 18, 2025에 액세스, https://schneppat.com/hard-triplet-mining.html
74. The RecallRate@N curves for all 10 VPR techniques generated on the 12... - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/figure/The-RecallRateN-curves-for-all-10-VPR-techniques-generated-on-the-12-datasets-by_fig3_351408419
75. Recall as a function of the number of candidates N in the image filtering stage. - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/figure/Recall-as-a-function-of-the-number-of-candidates-N-in-the-image-filtering-stage_fig5_336621495
76. 24/7 place recognition by view synthesis | Request PDF - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/308857352_247_place_recognition_by_view_synthesis
77. A curated list of awesome Visual Place Recognition papers - GitHub, 8월 18, 2025에 액세스, https://github.com/gmberton/awesome-Visual-Place-Recognition
78. Pittsburgh-30k-test Benchmark (Visual Place Recognition) | Papers With Code, 8월 18, 2025에 액세스, https://paperswithcode.com/sota/visual-place-recognition-on-pittsburgh-30k?p=revisit-anything-visual-place-recognition-via
79. CricaVPR: Cross-image Correlation-aware Representation Learning for Visual Place Recognition - CVPR 2024 Open Access Repository, 8월 18, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2024/html/Lu_CricaVPR_Cross-image_Correlation-aware_Representation_Learning_for_Visual_Place_Recognition_CVPR_2024_paper.html
80. ECCV Poster Revisit Anything: Visual Place Recognition via Image Segment Retrieval, 8월 18, 2025에 액세스, https://eccv.ecva.net/virtual/2024/poster/1843
81. Revisit Anything: Visual Place Recognition via Image Segment Retrieval - European Computer Vision Association, 8월 18, 2025에 액세스, https://www.ecva.net/papers/eccv_2024/papers_ECCV/papers/08592.pdf
82. Revisit Anything: Visual Place Recognition via Image Segment Retrieval - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2409.18049v1
83. Collaborative Visual Place Recognition through Federated Learning - CVPR 2024 Open Access Repository - The Computer Vision Foundation, 8월 18, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2024W/FedVision-2024/html/Dutto_Collaborative_Visual_Place_Recognition_through_Federated_Learning_CVPRW_2024_paper.html


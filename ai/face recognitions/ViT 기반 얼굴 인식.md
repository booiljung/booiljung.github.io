[얼굴인식 문제](./index.md)
# Vision Transformer 기반 얼굴 인식 기술



지난 10년간 컴퓨터 비전 분야는 Convolutional Neural Networks (CNN)의 시대였다. AlexNet의 등장을 기점으로 VGGNet, GoogLeNet, ResNet 등 심층 CNN 아키텍처들은 이미지 분류, 객체 탐지, 분할 등 다양한 과제에서 인간의 능력을 뛰어넘는 혁신적인 성과를 이끌었다.1 CNN의 성공은 계층적 특징 추출(hierarchical feature extraction), 지역적 수용장(local receptive fields), 그리고 번역 등변성(translation equivariance)과 같은 강력한 귀납적 편향(inductive bias)에 기반한다.3 이러한 내장된 가정들은 이미지 데이터의 공간적 구조를 효과적으로 학습하도록 설계되어, 비교적 적은 데이터로도 높은 성능을 달성하는 데 기여했다.

그러나 CNN의 지역적 수용장은 이미지 전체에 걸친 장거리 의존성(long-range dependencies)이나 전역적 맥락(global context)을 포착하는 데 본질적인 한계를 가진다. 이 문제를 해결하기 위해 어텐션 메커니즘을 도입하거나 네트워크를 매우 깊게 쌓는 방식이 시도되었으나, 근본적인 아키텍처의 제약을 완전히 극복하기는 어려웠다.

이러한 상황 속에서 자연어 처리(NLP) 분야에서 혁명을 일으킨 트랜스포머(Transformer) 아키텍처가 컴퓨터 비전 분야에 도입되었다.4 트랜스포머의 핵심인 셀프-어텐션(self-attention) 메커니즘은 입력 시퀀스 내의 모든 요소 간의 관계를 직접적으로 모델링한다. 이는 고정된 크기의 커널을 사용하는 CNN과 달리, 데이터의 내용에 따라 동적으로 관계의 중요도를 학습할 수 있음을 의미한다. 2020년 발표된 Vision Transformer (ViT)는 이미지를 여러 개의 패치(patch)로 분할하고 이를 단어 시퀀스처럼 처리하여 트랜스포머 인코더에 입력하는 단순하면서도 강력한 아이디어를 제시했다.4 ViT는 CNN의 귀납적 편향 없이도, 대규모 데이터셋을 통해 사전 학습될 경우 기존의 SOTA CNN 모델을 능가하는 성능을 보이며 컴퓨터 비전 분야의 새로운 패러다임을 열었다. 이는 '내장된 지식'에 의존하는 대신, 데이터 자체로부터 시각적 표현을 학습하는 방식의 잠재력을 증명한 것이다.7


얼굴 인식(Face Recognition, FR)은 보안, 금융, 모바일 인증 등 사회 전반에 걸쳐 활용되는 핵심 컴퓨터 비전 기술이다. 이 분야 역시 DeepFace, FaceNet, ArcFace와 같은 CNN 기반의 심층 메트릭 러닝(deep metric learning) 모델들이 기술 발전을 주도하며 높은 정확도를 달성해왔다.2 하지만 실제 환경에서는 가려짐(occlusion), 극단적인 자세 변화(pose variation), 조명 변화 등 예측 불가능한 변수들이 많아 여전히 강건성(robustness) 확보에 대한 도전 과제가 남아있다.1

ViT의 등장은 이러한 얼굴 인식 분야의 난제들을 해결할 새로운 가능성을 제시했다. ViT는 얼굴의 각 부분을 패치로 간주하고, 이들 간의 전역적인 관계를 모델링함으로써 얼굴 전체의 구조적 정보를 포착하는 데 탁월한 능력을 보인다. 이는 눈, 코, 입 등 특정 부위가 가려지더라도 나머지 부분들의 관계를 통해 신원을 유추하는 등, 기존 CNN 모델보다 까다로운 조건에서 더 강건한 성능을 보일 수 있는 잠재력을 의미한다.12

본 보고서는 ViT 기반 얼굴 인식 기술의 현주소를 심층적으로 고찰하는 것을 목표로 한다. 먼저 ViT 아키텍처의 근본 원리를 상세히 분석하고, 초기 표준 ViT 적용 시의 문제점부터 이를 극복하기 위해 진화해 온 데이터 중심 접근법, 부분 기반 모델, 하이브리드 아키텍처까지의 발전 과정을 추적한다. 나아가, 주요 벤치마크에서의 정량적 성능 비교, 계산 효율성, 그리고 특징 표현의 본질적 차이를 통해 CNN과의 심층적인 비교 분석을 수행한다. 또한, 판별력 강화를 위한 손실 함수 최적화 과정을 살펴보고, 강건성, 공정성, 자기지도학습, 다중모달 융합 등 최신 연구 동향을 종합적으로 조망함으로써 ViT 기반 얼굴 인식 기술의 미래 방향성을 제시하고자 한다.


ViT의 핵심 철학은 이미지의 2D 공간 구조라는 강력한 사전 지식을 의도적으로 해체하고, 이를 1D 시퀀스로 변환한 뒤 모든 요소 간의 관계를 데이터로부터 직접 학습하는 것이다. 이는 구조를 버리고 관계를 학습함으로써 더 유연하고 일반적인 표현을 얻으려는 시도로, CNN의 접근 방식과는 근본적인 차이를 보인다.


ViT는 2D 이미지를 트랜스포머가 처리할 수 있는 1D 토큰 시퀀스로 변환하는 전처리 과정을 거친다. 이 과정은 이미지 패치화, 선형 투영, 그리고 위치 정보 주입의 단계로 구성된다.


입력 이미지 $x \in \mathbb{R}^{H \times W \times C}$는 먼저 $P \times P$ 크기를 갖는 겹치지 않는 패치(patch)들의 시퀀스로 재구성된다.4 여기서 $(H, W)$는 원본 이미지의 해상도, $C$는 채널 수, $(P, P)$는 각 패치의 해상도이다. 이 과정은 이미지를 $N = \frac{HW}{P^2}$개의 패치 $\{x_p^i \in \mathbb{R}^{P \times P \times C} \mid i=1, \dots, N\}$로 분할한다. 그 후, 각 2D 패치는 1차원 벡터로 펼쳐져(flattened) $P^2 \cdot C$ 차원의 벡터가 된다.


펼쳐진 각 패치 벡터는 학습 가능한 선형 투영(linear projection) 행렬 $\mathbf{E} \in \mathbb{R}^{(P^2 \cdot C) \times D}$를 통해 트랜스포머가 요구하는 고정된 $D$차원의 임베딩 벡터로 매핑된다.16 이 과정은 실질적으로 각 패치에 대한 특징 추출 단계로 볼 수 있으며, 구현상으로는 커널 크기와 스트라이드가 패치 크기 $P$와 동일한 2D 컨볼루션 연산을 통해 효율적으로 수행될 수 있다.15


이미지 전체를 대표하는 전역적 특징을 집약적으로 학습하기 위해, NLP 모델인 BERT에서 영감을 받은 학습 가능한 `(classification) 토큰 임베딩 $x_{class}$가 패치 임베딩 시퀀스의 맨 앞에 추가된다.[15, 16, 19] 이` 토큰은 다른 패치 토큰들과 함께 트랜스포머 인코더를 통과하며 상호작용하고, 최종적으로 인코더의 마지막 레이어에서 `` 토큰에 해당하는 출력 벡터가 이미지 전체를 대표하는 특징 벡터로 사용되어 분류 헤드(MLP)에 입력된다.


트랜스포머의 셀프-어텐션 메커니즘은 입력 순서에 불변(permutation-invariant)하기 때문에, 각 패치의 원래 공간적 위치 정보를 모델에 명시적으로 제공해야 한다.17 이를 위해 학습 가능한 1D 위치 임베딩 행렬 $\mathbf{E}_{pos} \in \mathbb{R}^{(N+1) \times D}$가 패치 임베딩에 원소별 덧셈(element-wise addition)으로 더해진다.15 이 위치 임베딩은 모델이 훈련 과정에서 패치들 간의 상대적, 절대적 위치 관계를 학습하도록 돕는다.

이 모든 과정을 거쳐 트랜스포머 인코더에 최종적으로 입력되는 임베딩 시퀀스 $z_0$는 다음과 같이 수식으로 표현할 수 있다.18
$$
z_0 = [x_{class}; x_p^1\mathbf{E}; x_p^2\mathbf{E}; \cdots; x_p^N\mathbf{E}] + \mathbf{E}_{pos}
$$
여기서 $x_p^i$는 $i$번째 이미지 패치, $\mathbf{E}$는 선형 투영 행렬, 그리고 $\mathbf{E}_{pos}$는 위치 임베딩 행렬을 나타낸다.


ViT의 핵심 연산은 $L$개의 동일한 트랜스포머 인코더 블록을 순차적으로 통과하며 이루어진다.20 각 인코더 블록은 입력 시퀀스 내 모든 토큰 간의 관계를 모델링하여 특징 표현을 점진적으로 정제한다.


각 인코더 블록은 두 개의 주요 하위 레이어로 구성된다: Multi-Head Self-Attention (MHSA) 레이어와 Multi-Layer Perceptron (MLP) 블록이다. 각 하위 레이어의 입력과 출력 사이에는 잔차 연결(residual connection)이 적용되며, 각 하위 레이어 이전에는 Layer Normalization (LN)이 수행된다 (Pre-LN 구조).5 이는 훈련을 안정화하고 깊은 네트워크의 학습을 용이하게 한다.


셀프-어텐션의 기본 구성 요소는 Scaled Dot-Product Attention이다. 이 메커니즘은 입력 임베딩 시퀀스로부터 세 개의 서로 다른 벡터, 즉 Query (Q), Key (K), Value (V)를 생성한다. 이는 각 임베딩 벡터를 학습 가능한 가중치 행렬 $W^Q, W^K, W^V$와 곱하여 얻어진다. 어텐션의 개념은 "Query가 특정 Key와 얼마나 유사한가에 따라 해당 Key에 연결된 Value를 얼마나 가져올 것인가"로 요약할 수 있다.21

어텐션 점수는 Query와 모든 Key의 내적(dot product)을 통해 계산된다. 이 점수는 Key 벡터의 차원 $d_k$의 제곱근($\sqrt{d_k}$)으로 나누어 스케일링되는데, 이는 $d_k$가 클 때 내적 값이 너무 커져 Softmax 함수의 그래디언트가 소실되는 문제를 방지하기 위함이다.23 스케일링된 점수는 Softmax 함수를 통과하여 합이 1이 되는 어텐션 가중치(attention weights)로 변환된다. 마지막으로, 이 가중치를 Value 벡터에 가중합하여 해당 Query에 대한 최종 출력 벡터를 계산한다. 이 과정은 다음 수식으로 표현된다.23
$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$


ViT는 단일 어텐션을 사용하는 대신, $h$개의 어텐션 헤드(head)를 병렬적으로 사용하는 Multi-Head Self-Attention을 채택한다. 이는 모델이 서로 다른 '표현 부분 공간(representation subspaces)'에서 다양한 종류의 관계(예: 위치적 관계, 의미론적 관계)를 동시에 학습할 수 있게 한다.21

MHSA에서는 입력 Q, K, V를 $h$개의 작은 차원으로 나누어 각 헤드에 독립적으로 입력한다. 각 헤드는 자신만의 학습 가능한 가중치 행렬 $W_i^Q, W_i^K, W_i^V$를 가지며, 독립적으로 Scaled Dot-Product Attention을 수행한다. 이렇게 얻어진 $h$개의 출력 벡터들은 다시 하나로 연결(concatenate)된 후, 최종적으로 또 다른 학습 가능한 가중치 행렬 $W^O$를 통해 선형 변환되어 MHSA 레이어의 최종 출력이 된다.21
$$
\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_h)W^O \\
\text{where head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)
$$


MHSA 레이어를 통과한 특징 벡터는 MLP 블록으로 전달된다. 이 블록은 두 개의 완전 연결(fully-connected) 레이어와 그 사이에 비선형 활성화 함수(주로 GELU)로 구성된다.20 MLP 블록은 각 토큰 위치마다 독립적으로 적용되며, 모델의 표현력을 높이는 역할을 한다.

이러한 인코더 블록을 여러 층 쌓음으로써, ViT는 이미지 패치들 간의 복잡하고 계층적인 관계를 학습하여 최종적으로 `` 토큰에 이미지 전체에 대한 풍부한 의미 정보를 압축한다.


ViT 기반 얼굴 인식 모델의 발전 과정은 '순수주의'에서 '실용주의'로의 전환으로 요약할 수 있다. 초기에는 순수 ViT 아키텍처의 잠재력을 탐구하는 데 집중했지만, 곧이어 얼굴 인식이라는 특정 도메인의 요구사항과 ViT의 내재적 한계를 인식하게 되었다. 이에 따라 연구 방향은 ViT의 약점을 보완하고 강점을 극대화하기 위해 데이터 중심 전략, 도메인 지식 통합, 그리고 이종 아키텍처 융합과 같은 실용적인 해결책으로 빠르게 선회했다.


초기 연구들은 표준 ViT 아키텍처를 얼굴 인식 문제에 직접 적용했다. 그러나 CNN의 강력한 귀납적 편향(예: translation equivariance, locality)이 없는 ViT는 대규모 데이터셋에 크게 의존하는 'data-hungry' 특성을 보였다.12 이로 인해 일반적인 규모의 얼굴 데이터셋으로는 CNN 기반 모델 대비 압도적인 성능 우위를 보여주지 못했다.

더욱 심각한 문제로, ViT가 훈련 과정에서 특정 로컬 패치, 예를 들어 눈이나 이마와 같은 식별력이 높은 영역에 과적합(overfitting)되는 경향이 발견되었다.12 이는 모델이 얼굴의 전체적인 구조 대신 소수의 지배적인 단서에만 의존하게 만들어, 해당 부위가 선글라스나 모자 등으로 가려졌을 때 일반화 성능이 심각하게 저하되는 결과를 초래했다.

이러한 문제를 해결하기 위해 **TransFace**는 모델 아키텍처를 수정하는 대신, 훈련 데이터와 학습 전략을 얼굴 인식에 맞게 고도화하는 데이터 중심(data-centric) 접근법을 제안했다.12 TransFace의 핵심 전략은 다음과 같다.

- **DPAP (Dominant Patch Amplitude Perturbation):** 이는 얼굴의 핵심적인 구조 정보를 보존하면서 과적합을 완화하기 위해 설계된 패치 레벨 데이터 증강 기법이다. 먼저 Squeeze-and-Excitation (SE) 모듈을 활용하여 최종 예측에 가장 큰 영향을 미치는 '지배적 패치(dominant patches)'를 식별한다. 그 후, 푸리에 변환을 통해 이 패치들의 진폭(amplitude) 스펙트럼 정보만을 무작위로 교란하고, 구조 정보를 담고 있는 위상(phase) 스펙트럼은 그대로 유지한다. 이렇게 생성된 다양한 학습 샘플들은 ViT가 특정 패치에 대한 과도한 의존도를 줄이고, 코나 입과 같은 다른 얼굴 영역의 정보도 균형 있게 활용하도록 유도한다.12
- **EHSM (Entropy-guided Hard Sample Mining):** 기존의 어려운 샘플 마이닝 전략들이 ViT의 특성(소수 패치에 의한 예측 지배)과 부합하지 않는다는 문제의식에서 출발했다. EHSM은 정보 이론에 기반하여 각 로컬 토큰이 담고 있는 정보 엔트로피를 측정함으로써 샘플의 난이도를 평가한다. 정보량이 적은 저품질 이미지(어려운 샘플)에 더 높은 손실 가중치를 부여함으로써, 모델이 어려운 샘플에 더 집중하고 결과적으로 더 안정적이고 강건한 예측을 하도록 돕는다.12


ViT의 유연성과 얼굴이라는 도메인이 가진 강력한 구조적 사전 지식을 결합하려는 시도에서 부분 기반(part-based) 접근법이 등장했다.13 얼굴이 눈, 코, 입 등 의미론적으로 중요한 부분들로 구성되어 있다는 사실을 모델 설계에 직접 반영한 것이다.

**part fViT**는 이러한 아이디어를 구현한 대표적인 모델이다.13 이 모델은 다음과 같은 단계로 작동한다.

1. 경량 CNN을 사용하여 입력 얼굴 이미지에서 주요 랜드마크(예: 눈, 코, 입의 위치)의 좌표를 예측한다.
2. 예측된 랜드마크 좌표를 중심으로, 불규칙한 그리드(irregular grid) 상에서 패치를 샘플링한다.
3. 이렇게 샘플링된 부분 패치들만을 ViT의 입력 토큰으로 사용하여 특징을 추출하고 인식을 수행한다.

이 접근법은 여러 장점을 가진다. 첫째, 전체 파이프라인이 랜드마크에 대한 별도의 지도 없이 end-to-end로 학습되기 때문에, 랜드마크 예측 네트워크는 얼굴 인식 성능에 가장 도움이 되는, 즉 판별력이 가장 높은 영역을 스스로 학습하게 된다.13 둘째, ViT는 입력 토큰의 공간적 배열에 제약이 없으므로, CNN과 달리 불규칙하게 샘플링된 패치들을 자연스럽게 처리할 수 있다.13 셋째, 전체 이미지를 조밀한 그리드로 나누는 대신 소수의 중요한 부분 패치만 처리하므로 계산적으로 더 효율적일 수 있다.13


ViT의 전역적 맥락 포착 능력과 CNN의 효율적인 지역적 특징 추출 및 강력한 귀납적 편향을 결합하려는 시도는 자연스러운 귀결이었다.29 하이브리드 아키텍처는 두 패러다임의 장점을 모두 취하려는 실용적인 접근법이다.

일반적으로 하이브리드 모델은 초기 레이어에서 CNN을 사용하여 이미지의 저수준 특징(엣지, 질감 등)과 지역적 패턴을 효율적으로 추출한다. 이렇게 생성된 특징 맵(feature map)을 다시 패치화하여 후반부의 ViT 레이어에 입력함으로써, 추출된 지역 특징들 간의 장거리 관계와 전역적 맥락을 모델링한다.30

이러한 융합은 다음과 같은 시너지를 창출한다.

- **성능 향상:** CNN이 제공하는 풍부한 지역적 정보와 ViT가 포착하는 전역적 관계가 상호 보완하여 더 강력하고 강건한 얼굴 표현을 학습할 수 있다.29
- **훈련 효율성:** CNN의 귀납적 편향 덕분에 순수 ViT에 비해 더 적은 데이터로도 효과적인 학습이 가능하며, 수렴 속도가 빠르다.29
- **계산 비용 절감:** Swin Transformer와 같이 계층적 구조를 가지며 계산 효율성을 높인 ViT 변종을 사용하거나, FaceLiVT의 Multi-Head Linear Attention (MHLA)처럼 경량화된 어텐션 메커니즘을 CNN과 결합함으로써, 높은 성능을 유지하면서도 계산 복잡도를 크게 줄일 수 있다.31

이러한 모델들의 진화는 ViT가 얼굴 인식 분야에서 단독으로 사용되기보다는, 도메인 특성과 기존 기술의 장점을 적극적으로 수용하는 방향으로 발전하고 있음을 명확히 보여준다.


ViT와 CNN은 얼굴 인식 분야에서 서로 다른 강점과 약점을 보이며, 성능, 효율성, 그리고 학습하는 특징의 본질적인 측면에서 뚜렷한 차이를 나타낸다. 어떤 아키텍처가 우월한지에 대한 답은 훈련 데이터의 규모, 사용된 벤치마크의 특성, 그리고 계산 자원의 제약 등 다양한 요인에 따라 달라진다.


얼굴 인식 모델의 성능은 Labeled Faces in the Wild (LFW)와 같은 통제되지 않은 환경의 검증(verification) 벤치마크와, IARPA Janus Benchmark-C (IJB-C)와 같이 포즈, 조명, 가려짐 등 극심한 변화를 포함하는 대규모 식별(identification) 벤치마크를 통해 주로 평가된다.34

대규모 데이터셋(예: MS1MV2, Glint360K)으로 충분히 훈련되었을 때, 데이터 중심 최적화가 적용된 ViT 기반 모델은 SOTA CNN 모델과 대등하거나 이를 능가하는 성능을 보인다. 아래 **표 1**은 IJB-C 벤치마크에서 다양한 모델의 성능을 비교한 결과이다. Glint360K 데이터셋으로 훈련된 TransFace-L 모델은 TAR@FAR=1E-4 기준 97.61%의 정확도를 달성하여, 동일 데이터셋으로 훈련된 ViT-L(97.13%) 및 강력한 CNN 기반 모델인 R200-ArcFace(97.20%)를 상회하는 성능을 기록했다.12 이는 ViT가 데이터 중심의 맞춤형 훈련 전략과 결합될 때 최고의 성능을 발휘할 수 있음을 시사한다.

강건성 측면에서, ViT는 전역적 특징을 포착하는 능력 덕분에 이미지의 일부가 가려지거나(occlusion) 카메라와의 거리가 변하는 상황에서 CNN보다 더 강인한 성능을 보이는 경향이 있다.14 한 비교 연구에서는 ViT가 EfficientNet, ResNet 등 다양한 CNN 아키텍처에 비해 여러 데이터셋에서 일관되게 더 높은 정확도와 강건성을 보였다고 보고했다.14

그러나 ViT의 성능은 훈련 데이터의 규모에 크게 의존한다는 점을 유의해야 한다. 상대적으로 작은 데이터셋에서는 CNN이 가진 강력한 귀납적 편향이 더 효율적인 학습을 가능하게 하여 ViT보다 우수한 성능을 보일 수 있다.12

| Training Data | Method               | GFLOPs | IJB-C (TAR@FAR=1E-4) | IJB-C (TAR@FAR=1E-5) |
| ------------- | -------------------- | ------ | -------------------- | -------------------- |
| MS1MV2        | R100, ArcFace        | 12.1   | 95.60                | -                    |
| MS1MV2        | R100, CurricularFace | 12.1   | 96.10                | -                    |
| MS1MV2        | ViT-L                | 25.3   | 96.24                | 94.11                |
| MS1MV2        | **TransFace-L**      | 25.4   | **96.59**            | **94.55**            |
| Glint360K     | R100, ArcFace        | 12.1   | 96.89                | 95.38                |
| Glint360K     | R200, ArcFace        | 23.4   | 97.20                | 95.71                |
| Glint360K     | ViT-L                | 25.3   | 97.13                | 95.78                |
| Glint360K     | **TransFace-L**      | 25.4   | **97.61**            | **96.29**            |


계산 효율성은 모델의 실용성을 결정하는 중요한 척도이다. 표준 ViT의 셀프-어텐션 메커니즘은 입력 패치의 수 $N$에 대해 $O(N^2)$의 계산 복잡도를 가진다.36 이는 입력 이미지의 해상도가 높아질수록 계산 비용이 기하급수적으로 증가함을 의미하며, 고해상도 얼굴 이미지를 처리하는 데 병목이 될 수 있다. 반면, CNN의 연산량은 특징 맵의 크기에 선형적으로 비례하여 상대적으로 확장성이 좋다.

그럼에도 불구하고, ViT는 고도로 병렬화 가능한 구조 덕분에 GPU와 같은 하드웨어에서 효율적으로 실행될 수 있다.6 일부 연구에서는 비슷한 성능 수준에서 ViT가 CNN보다 더 적은 파라미터를 가지며, 메모리 사용량도 적고 추론 속도는 더 빠를 수 있다고 보고했다.14

ViT의 높은 계산 비용 문제를 해결하기 위한 경량화 연구도 활발히 진행되고 있다. **표 2**는 하이브리드 경량화 모델인 FaceLiVT의 성능을 다른 모델과 비교한 것이다. FaceLiVT는 어텐션 메커니즘을 계산적으로 저렴한 Multi-Head Linear Attention (MHLA)으로 대체하여, 순수 ViT 기반 모델 대비 21.2배, 다른 하이브리드 모델인 EdgeFace 대비 8.6배 빠른 추론 속도를 달성하면서도 LFW, CFP-FP 등에서 경쟁력 있는 정확도를 유지했다.33 이는 아키텍처 혁신을 통해 ViT의 실용성을 크게 높일 수 있음을 보여준다.

| Model          | Accuracy (LFW) | Accuracy (CFP-FP) | Inference Speed (ms) | FLOPs (G) |
| -------------- | -------------- | ----------------- | -------------------- | --------- |
| Pure ViT-Based | 높음 (N/A)     | 높음 (N/A)        | 21.2x                | 1.5       |
| EdgeFace       | 99.63          | 97.14             | 8.6x                 | 0.154     |
| **FaceLiVT**   | **99.55**      | **97.04**         | **1x (기준)**        | **0.153** |


ViT와 CNN의 성능 차이는 두 아키텍처가 이미지를 해석하고 특징을 표현하는 방식의 근본적인 차이에서 비롯된다.

- **ViT (Global Context):** ViT의 셀프-어텐션은 첫 번째 레이어부터 이미지의 모든 패치 간의 관계를 동시에 계산하므로, 시작부터 전역적인 수용장(global receptive field)을 가진다.7 이는 얼굴의 전체적인 형태, 눈, 코, 입 등 각 구성 요소 간의 상대적 위치와 같은 구조적 정보를 이해하는 데 매우 유리하다. 따라서 얼굴의 미묘한 표정 변화나 전체적인 인상을 종합적으로 판단하는 데 강점을 보일 수 있다.38
- **CNN (Local Features):** CNN은 작은 크기의 필터(커널)를 이미지 위에서 슬라이딩시키며 지역적인 패턴(엣지, 코너, 질감 등)을 추출한다. 레이어를 깊게 쌓아감에 따라 이러한 지역적 특징들이 점진적으로 조합되어 더 크고 추상적인 특징으로 발전한다.1 이 방식은 이미지의 지역적 질감이나 세부적인 디테일을 포착하는 데 매우 효율적이지만, 이미지의 멀리 떨어진 두 부분 간의 관계를 직접적으로 파악하기 위해서는 매우 깊은 네트워크가 필요하다는 한계가 있다.

이러한 차이는 주파수 관점에서도 해석될 수 있다. 연구에 따르면, ViT는 이미지의 저주파수(low-frequency) 신호, 즉 전체적인 형태와 구조에 더 집중하는 경향이 있는 반면, CNN은 고주파수(high-frequency) 신호, 즉 날카로운 엣지나 복잡한 질감에 더 민감하게 반응한다. 이러한 본질적인 차이는 두 모델이 서로 다른 종류의 정보에 강점을 가지며, 적대적 공격과 같은 특정 유형의 노이즈에 대해 서로 다른 강건성을 보이는 원인이 된다.40


ViT 기반 얼굴 인식 모델의 성능은 아키텍처뿐만 아니라, 학습 과정에서 모델을 최적화하는 손실 함수(loss function)에 의해서도 크게 좌우된다. 얼굴 인식의 목표는 단순히 이미지를 분류하는 것을 넘어, 특징 공간(feature space) 내에서 동일인의 특징은 가깝게, 타인의 특징은 멀게 분포하도록 만드는 것이다. 이러한 목표를 달성하기 위해, 연구는 '분류 가능성(separability)'을 넘어 '판별력(discriminability)'을 극대화하는 방향으로 진화했으며, 이는 특징 공간의 기하학적 구조를 직접적으로 설계하는 정교한 손실 함수들의 개발로 이어졌다.


FaceNet은 얼굴 인식을 분류 문제가 아닌, 임베딩 공간(embedding space)에서의 거리 학습, 즉 메트릭 러닝(metric learning) 문제로 재정의했다.10 이를 위해 제안된 Triplet Loss는 동일 인물의 얼굴 임베딩 간의 거리는 좁히고(pull), 다른 인물의 얼굴 임베딩 간의 거리는 밀어내는(push) 방식으로 작동한다.

학습은 세 개의 샘플로 구성된 'triplet'을 기반으로 이루어진다: 기준이 되는 **Anchor** ($x^a$), Anchor와 동일 인물인 **Positive** ($x^p$), 그리고 Anchor와 다른 인물인 **Negative** ($x^n$)이다.42 Triplet Loss의 목표는 Anchor-Positive 쌍의 제곱 유클리드 거리($\lVert f(x^a) - f(x^p) \rVert_2^2$)가 Anchor-Negative 쌍의 제곱 유클리드 거리($\lVert f(x^a) - f(x^n) \rVert_2^2$)보다 최소한 $\alpha$만큼 작아지도록 강제하는 것이다. 여기서 $\alpha$는 두 거리 쌍 사이에 확보하고자 하는 마진(margin)을 의미하는 하이퍼파라미터이다.

Triplet Loss 함수는 다음과 같이 수식으로 정의된다.42
$$
L = \sum_{i}^{N} \left[ \lVert f(x_i^a) - f(x_i^p) \rVert_2^2 - \lVert f(x_i^a) - f(x_i^n) \rVert_2^2 + \alpha \right]_+
$$
여기서 $[z]_+ = \max(z, 0)$을 의미하며, 대괄호 안의 값이 양수일 때만 손실이 발생한다. 즉, 마진 $\alpha$가 확보된 triplet에 대해서는 더 이상 학습이 진행되지 않는다.

Triplet Loss의 성공은 효과적인 **triplet 마이닝(mining)** 전략에 크게 의존한다. 무작위로 triplet을 구성하면 대부분 손실이 0인 쉬운 샘플들이 되어 학습이 비효율적이므로, 손실을 유발하는 '어려운' triplet, 특히 마진 내에 있는 'semi-hard negative' 샘플을 효과적으로 선택하는 것이 빠른 수렴과 높은 성능에 필수적이다.42


Triplet Loss는 강력하지만, 대규모 데이터셋에서 가능한 모든 triplet 조합을 고려하는 것은 계산적으로 비효율적이며, 샘플링 전략에 성능이 민감하다는 단점이 있다.44 이러한 문제를 해결하기 위해, 전통적인 Softmax 손실 함수를 기반으로 판별력을 강화하는 마진 기반(margin-based) 손실 함수들이 제안되었다.

전통적인 Softmax 손실은 특징 벡터가 클래스별로 선형적으로 분리 가능하도록(separable) 학습하지만, 클래스 내 분산(intra-class variance)을 줄이고 클래스 간 분산(inter-class variance)을 최대화하여 판별력을 극대화하는 데는 직접적으로 최적화되어 있지 않다.45

**ArcFace**와 같은 각도 마진 기반 손실 함수들은 이 문제를 해결하기 위해 특징 공간을 기하학적으로 재구성한다.44

1. **특징 및 가중치 정규화:** 먼저, 심층 네트워크에서 추출된 특징 벡터 $x_i$와 마지막 완전 연결 레이어의 가중치 벡터 $W_j$를 각각 L2 정규화하여 단위 초구면(unit hypersphere) 상에 매핑한다.
2. **코사인 유사도:** 정규화로 인해, 두 벡터의 내적 $W_j^T x_i$는 두 벡터 사이의 각도 $\theta_j$의 코사인 값 $\cos(\theta_j)$와 동일해진다.
3. **Additive Angular Margin:** ArcFace는 Softmax 계산에 사용되는 로짓(logit)을 변형한다. 목표 클래스 $y_i$에 해당하는 각도 $\theta_{y_i}$에 직접적으로 **각도 마진(additive angular margin)** $m$을 더한 후($\theta_{y_i} + m$), 다시 코사인 함수를 적용한다.
4. **스케일링:** 마지막으로, 모든 로짓 값에 스케일링 파라미터 $s$를 곱하여 Softmax 손실을 계산한다.

이 과정은 클래스 간 결정 경계(decision boundary)를 각도 공간에서 $m$만큼 밀어내는 효과를 가져온다. 이는 초구면 상에서 클래스 간 측지 거리(geodesic distance) 마진을 직접적으로 최적화하는 것과 동일하며, 클래스 내 특징들을 더욱 밀집시키고(enhancing intra-class compactness) 클래스 간의 차이를 더욱 벌리는(enlarging inter-class discrepancy) 결과를 낳는다.13

ArcFace 손실 함수는 다음과 같이 표현된다.
$$
L = -\frac{1}{N}\sum_{i=1}^{N}\log\frac{e^{s(\cos(\theta_{y_i}+m))}}{e^{s(\cos(\theta_{y_i}+m))} + \sum_{j=1, j\neq y_i}^{n}e^{s\cos\theta_j}}
$$
ArcFace는 Triplet Loss의 조합 폭발 문제 없이 대규모 데이터셋에서 효율적으로 학습이 가능하고, Softmax 기반이므로 훈련이 안정적이라는 장점을 가진다. 명확한 기하학적 해석을 바탕으로 얼굴 인식의 판별력을 크게 향상시켜, 현재 많은 SOTA 얼굴 인식 모델에서 표준 손실 함수로 사용되고 있다.44


ViT 기반 얼굴 인식 기술은 단순히 정확도를 높이는 단계를 넘어, 실제 세계의 복잡성과 사회적 요구사항을 해결하기 위한 다차원적인 문제 공간으로 확장되고 있다. 이는 기술이 성숙함에 따라 '강건성', '공정성', '효율성', 그리고 '확장성'과 같은 현실적인 과제들을 해결하려는 방향으로 진화하고 있음을 보여준다.



실제 얼굴 인식 환경은 통제되지 않은 변수들로 가득하다. ViT는 전역적 어텐션 메커니즘 덕분에 얼굴의 일부가 가려지더라도 나머지 보이는 패치들 간의 관계를 통해 전체적인 맥락을 유추하는 데 잠재적인 강점을 가진다.49

이러한 강건성을 더욱 강화하기 위한 연구가 활발히 진행 중이다. **KP-RPE (KeyPoint Relative Position Encoding)**는 얼굴 정렬(alignment)이 실패하거나 극단적인 자세 변화가 있을 때 ViT의 성능 저하를 막기 위해 제안된 기법이다.51 이 방법은 얼굴 랜드마크(keypoints)를 기준으로 각 패치의 상대적 위치를 동적으로 인코딩한다. 이를 통해 모델은 얼굴이 회전하거나 기울어져도 눈, 코, 입 간의 기하학적 관계를 강건하게 유지할 수 있다. 한편, 하이브리드 CNN-ViT 모델은 CNN의 강력한 텍스처 분석 능력과 ViT의 구조적 이해 능력을 결합하여, 조명 변화와 같은 저수준의 이미지 변이에 대한 강건성을 높이는 데 기여한다.30


ViT 역시 CNN과 마찬가지로 인간의 눈에는 보이지 않는 미세한 노이즈를 추가하여 모델의 오작동을 유발하는 적대적 공격(adversarial attacks)에 취약하다. 특히, ViT의 어텐션 메커니즘을 직접 겨냥하는 **적대적 패치 공격(adversarial patch attack)**에 더욱 취약한 경향이 나타난다. 공격자는 이미지의 작은 영역(패치) 하나만 정교하게 조작하여 전체 어텐션 맵을 오염시키고, 모델이 완전히 다른 예측을 하도록 유도할 수 있다.54

흥미롭게도, 주파수 관점에서 ViT는 CNN과 다른 강건성 특성을 보인다. ViT는 이미지의 전체적인 형태와 구조에 해당하는 저주파수 특징에 더 의존하는 반면, CNN은 텍스처와 같은 고주파수 특징에 더 민감하다. 따라서 고주파수 노이즈로 구성된 일반적인 적대적 공격에 대해서는 ViT가 상대적으로 더 강건한 모습을 보인다.40

ViT를 방어하기 위한 기법으로는 **적대적 훈련(Adversarial Training)**이 가장 효과적인 방법 중 하나로 알려져 있으며, 이는 훈련 데이터에 적대적 예제를 포함시켜 모델의 강건성을 높이는 방식이다.55 또한, 

**ViTGuard**나 **Protego**와 같은 탐지 기반 방어 프레임워크도 제안되었다. 이들은 ViT의 고유한 내부 특징(어텐션 맵, `` 토큰 표현 등)을 활용하여 입력 이미지가 적대적인지 정상적인지를 판별한다. 예를 들어, 원본 이미지와 Masked Autoencoder (MAE)로 복원한 이미지의 어텐션 맵을 비교하여 그 불일치가 크면 적대적 공격으로 탐지하는 방식이다.56


얼굴 인식 기술의 사회적 영향력이 커지면서, 모델의 공정성은 매우 중요한 이슈로 부상했다. ViT 역시 훈련 데이터에 내재된 인구통계학적(성별, 인종, 연령 등) 편향을 학습하고 증폭시킬 수 있다.62 특히, ViT의 멀티-헤드 어텐션 메커니즘이 편향을 증폭시키는 독특한 경로를 제공한다는 사실이 밝혀졌다. 연구에 따르면, 특정 어텐션 헤드들이 피부톤이나 헤어스타일과 같이 특정 인구통계 그룹과 관련된 특징에 과도하게 집중하여 편향된 예측을 유도하는 현상이 관찰되었다.63

이러한 '편향된 헤드' 문제를 해결하기 위해 어텐션 메커니즘을 직접 제어하는 기법들이 제안되었다.

- **편향 헤드 식별 및 프루닝(Pruning):** 먼저, 각 어텐션 헤드의 어텐션 맵과 인구통계학적 속성 간의 상관관계를 통계적으로 분석하여 편향을 유발하는 헤드를 식별한다. 그 후, 식별된 편향 헤드를 모델에서 제거(pruning)하여 편향된 정보가 모델의 최종 결정에 영향을 미치는 것을 원천적으로 차단한다.63
- **적응적 재가중치(Adaptive Reweighting):** 편향된 헤드를 제거하는 대신, 그 영향을 완화하는 방식이다. 공정성 관련 손실 함수를 도입하여, 편향된 헤드가 민감한 속성과 관련된 영역에 덜 집중하도록 어텐션 가중치를 훈련 과정에서 적응적으로 조정한다.63


ViT의 'data-hungry' 특성을 극복하고, 대규모의 레이블 없는 데이터를 활용하기 위해 자기지도학습(SSL)이 핵심적인 사전 학습 패러다임으로 자리 잡았다.65

- **DINO (self-distillation with no labels):** DINO는 레이블 없이 지식 증류(knowledge distillation)를 수행하는 프레임워크이다.67 학생-교사(student-teacher) 네트워크 구조를 사용하며, 교사 네트워크는 학생 네트워크 가중치의 지수 이동 평균(EMA)으로 업데이트되는 

  **모멘텀 인코더(momentum encoder)** 방식을 통해 안정적인 학습 목표를 제공한다. **Multi-crop augmentation**을 통해 동일 이미지의 다양한 뷰(전역 뷰, 지역 뷰)를 생성하고, 학생 네트워크가 지역 뷰의 출력을 교사 네트워크의 전역 뷰 출력과 일치시키도록 학습한다. 이 과정은 두 출력 분포 간의 cross-entropy 손실을 최소화하는 자기 증류 방식으로 이루어진다.63

- **MAE (Masked Autoencoders):** MAE는 NLP의 마스크 언어 모델링에서 영감을 받은 재구성(reconstruction) 기반의 SSL 방법이다.72 이미지 패치의 75%와 같이 매우 높은 비율을 랜덤하게 마스킹하고, 보이는 패치만을 인코더에 입력하여 마스킹된 패치를 픽셀 단위로 복원하도록 학습한다. 

  **비대칭적 인코더-디코더 구조**를 채택하여, 무거운 인코더는 보이는 패치(25%)만 처리하고, 매우 가벼운 디코더가 전체 이미지를 복원하므로 사전 학습이 매우 효율적이다.73

  - **FaceMAE:** MAE를 프라이버시 보호 얼굴 인식에 적용한 **FaceMAE**는 마스킹을 통해 민감한 개인 정보를 가리면서도 신원 정보는 보존한다. 특히, 기존 MAE의 MSE 손실 대신 **IRM(Instance Relation Matching) 손실**을 사용하여 원본 얼굴과 복원된 얼굴의 특징 분포를 일치시킴으로써, 인식 성능과 프라이버시 보호 사이의 균형을 맞춘다.75


ViT의 유연성은 얼굴 인식을 이미지 대 이미지 매칭을 넘어 다양한 모달리티와 데이터 형태로 확장시키고 있다.

- **다중모달 융합 (CLIP):** **CLIP (Contrastive Language-Image Pre-training)**은 이미지 인코더(ViT)와 텍스트 인코더(Transformer)를 대규모 이미지-텍스트 쌍 데이터로 대조 학습시킨다.80 이를 통해 "안경을 쓴 곱슬머리 남자"와 같은 자연어 설명으로 얼굴 이미지를 검색하는 제로샷(zero-shot) 얼굴 인식이 가능해진다.83 또한, 텍스트 설명을 보조 정보로 활용하여 저품질 이미지의 인식 성능을 향상시키는 연구도 활발하다.85
- **합성 데이터 활용:** StyleGAN이나 Diffusion Model과 같은 최첨단 생성 모델을 사용하여 대규모 합성 얼굴 데이터셋을 생성하는 연구가 주목받고 있다.86 합성 데이터는 (1) 실제 데이터 수집에 따르는 프라이버시 및 법적 문제를 해결하고, (2) 소수 인구통계 그룹의 데이터를 의도적으로 증강하여 편향을 완화하며, (3) 다양한 포즈, 조명, 가려짐 등 통제된 환경에서 어려운 시나리오를 생성하여 모델의 강건성을 높이는 데 기여한다.86
- **차원 확장:** ViT의 적용 범위는 2D 이미지를 넘어 3D, 비디오, 열화상 등 다양한 데이터 형태로 확장되고 있다.
  - **비디오 (ViViT):** ViT를 시공간 차원으로 확장한 ViViT는 비디오 프레임 시퀀스를 토큰화하여 처리한다. 공간적 어텐션과 시간적 어텐션을 분리하여 계산하는 'Factorised encoder'와 같은 효율적인 구조를 통해 비디오 내의 동적인 얼굴 변화를 모델링한다.88
  - **3D 포인트 클라우드 (PCT):** Point Cloud Transformer (PCT)는 트랜스포머의 순서 불변성을 활용하여 순서가 없는 3D 포인트 클라우드 데이터를 직접 처리한다. 이웃 임베딩과 오프셋-어텐션과 같은 기법을 통해 3D 얼굴의 정밀한 기하학적 정보를 포착한다.93
  - **열화상 이미지 (BEFiT):** BEFiT과 같은 모델은 가시광선 이미지와 열화상 이미지를 함께 입력받아, 조명 변화에 매우 강건한 다중 스펙트럼(multi-spectral) 얼굴 임베딩을 생성하여 극한 환경에서의 인식 성능을 높인다.95


Vision Transformer(ViT)는 자연어 처리 분야에서 시작된 혁신을 컴퓨터 비전으로 성공적으로 이식하며, 얼굴 인식 기술의 새로운 지평을 열었다. 셀프-어텐션 메커니즘을 통해 이미지의 전역적 맥락과 장거리 의존성을 효과적으로 모델링하는 ViT의 능력은, 기존 CNN 패러다임의 한계를 보완하고 극복할 수 있는 강력한 대안으로 자리매김했다. 본 보고서는 ViT의 기본 아키텍처 원리부터 얼굴 인식 분야에서의 진화 과정, CNN과의 심층적인 성능 비교, 그리고 첨단 연구 동향까지 다각적으로 고찰했다.

ViT 기반 얼굴 인식 기술의 현주소는 '가능성의 증명' 단계를 지나 '실용적 고도화' 단계에 진입했음을 보여준다. 초기에는 ViT의 'data-hungry' 특성과 특정 패치에 대한 과적합 문제로 인해 어려움을 겪었으나, TransFace와 같은 데이터 중심 최적화, part fViT와 같은 도메인 지식 통합, 그리고 CNN-ViT 하이브리드 아키텍처와 같은 실용적 융합을 통해 SOTA 성능을 달성했다. 특히 대규모 데이터셋과 결합되었을 때, ViT는 가려짐이나 자세 변화와 같은 까다로운 조건에서 CNN을 능가하는 강건성을 보이며 그 잠재력을 입증했다.

그러나 ViT가 모든 면에서 우월한 만능 해결책은 아니다. 표준 ViT의 높은 계산 복잡도, 적대적 패치 공격에 대한 새로운 취약점, 그리고 훈련 데이터에 내재된 인구통계학적 편향을 증폭시키는 문제 등은 여전히 해결해야 할 중요한 과제이다.

향후 ViT 기반 얼굴 인식 연구는 다음과 같은 방향으로 나아갈 것으로 전망된다.

- **계산 효율성 증대:** 고해상도 비디오나 3D 포인트 클라우드 데이터를 실시간으로 처리하기 위해, 선형 어텐션, 희소 어텐션 등 더욱 효율적인 어텐션 메커니즘에 대한 연구가 지속될 것이다.
- **데이터 프라이버시와 합성 데이터의 발전:** 프라이버시 문제를 근본적으로 해결하고, 편향 완화 및 강건성 향상을 위해 실제 데이터와의 분포 격차를 최소화하는 고품질 합성 데이터 생성 기술이 핵심 연구 분야로 부상할 것이다.
- **설명가능성(XAI)과 신뢰성 확보:** ViT의 어텐션 맵을 활용하여 모델의 판단 근거를 시각적, 언어적으로 설명하는 연구(예: Attention Rollout)가 심화될 것이다.71 이는 모델의 투명성을 높여, 법률, 의료 등 민감한 분야에서의 신뢰성을 확보하는 데 필수적이다.
- **지속 학습(Continual Learning) 능력 강화:** 새로운 얼굴 데이터를 지속적으로 학습하면서도 과거에 학습한 정보를 잊지 않는(catastrophic forgetting 완화) 평생 학습 모델의 개발은, 변화하는 실제 환경에 적응할 수 있는 지능형 시스템 구축의 핵심 과제가 될 것이다.100

결론적으로, ViT는 얼굴 인식 기술의 지형을 근본적으로 바꾸어 놓았다. 미래의 얼굴 인식 시스템은 단순히 누구인지를 식별하는 것을 넘어, 왜 그렇게 판단했는지 설명할 수 있고(XAI), 모든 사람에게 공정하게 작동하며(Fairness), 새로운 위협에 강건하고(Robustness), 끊임없이 변화하는 세상에 적응(Continual Learning)할 수 있어야 한다. ViT는 이러한 차세대 지능형 인식 시스템을 구축하기 위한 가장 유연하고 강력한 아키텍처 플랫폼으로서, 그 발전 가능성은 무궁무진하다.


1. Face Recognition Using Convolutional Neural Networks: A Review - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/393470781_Face_Recognition_Using_Convolutional_Neural_Networks_A_Review
2. A Review of Deep Convolutional Neural Networks in Mobile Face Recognition, 8월 24, 2025에 액세스, https://online-journals.org/index.php/i-jim/article/view/40867
3. Facial Recognition Algorithms: A Systematic Literature Review - MDPI, 8월 24, 2025에 액세스, https://www.mdpi.com/2313-433X/11/2/58
4. Three things everyone should know about Vision Transformers - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2203.09795
5. Transformer (deep learning architecture) - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Transformer_(deep_learning_architecture)
6. Vision Transformers (ViT) in Image Recognition: Full Guide - viso.ai, 8월 24, 2025에 액세스, https://viso.ai/deep-learning/vision-transformer-vit/
7. Vision Transformers (ViT): Revolutionizing Computer Vision - Analytics Vidhya, 8월 24, 2025에 액세스, https://www.analyticsvidhya.com/blog/2023/06/vision-transformers-vit-revolutionizing-computer-vision/
8. Introduction to ViT (Vision Transformers): Everything You Need to Know - Lightly, 8월 24, 2025에 액세스, https://www.lightly.ai/blog/vision-transformers-vit
9. A Convolutional Neural Network Face Recognition Method Based on BiLSTM and Attention Mechanism - PMC, 8월 24, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC9879688/
10. Knock Knock, Who's There: Facial Recognition using CNN-based Classifiers, 8월 24, 2025에 액세스, https://thesai.org/Downloads/Volume13No1/Paper_2-Knock_Knock_Whos_There_Facial_Recognition.pdf
11. DeepFace - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/DeepFace
12. TransFace: Calibrating Transformer Training for ... - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/ICCV2023/papers/Dan_TransFace_Calibrating_Transformer_Training_for_Face_Recognition_from_a_Data-Centric_ICCV_2023_paper.pdf
13. Part-based Face Recognition with Vision Transformers - BMVC 2022, 8월 24, 2025에 액세스, https://bmvc2022.mpi-inf.mpg.de/0611.pdf
14. Comprehensive comparison between vision transformers and ..., 8월 24, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11399229/
15. Vision Transformer (ViT) - labml.ai, 8월 24, 2025에 액세스, https://nn.labml.ai/transformers/vit/index.html
16. Vision Transformers (ViT) Explained | Pinecone, 8월 24, 2025에 액세스, https://www.pinecone.io/learn/series/image-search/vision-transformers/
17. Positional Embeddings in Vision Transformers | ML & CV Consultant - Abhik Sarkar, 8월 24, 2025에 액세스, https://www.abhik.xyz/concepts/positional-embeddings-vit
18. Positional Encoding - ViT - Kaggle, 8월 24, 2025에 액세스, https://www.kaggle.com/code/shravankumar147/positional-encoding-vit
19. Maximizing the Position Embedding for Vision Transformers with Global Average Pooling, 8월 24, 2025에 액세스, https://arxiv.org/html/2502.02919v1
20. Vision transformer architecture and applications in digital health: a tutorial and survey - PMC, 8월 24, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC10333157/
21. The Illustrated Transformer – Jay Alammar – Visualizing machine ..., 8월 24, 2025에 액세스, https://jalammar.github.io/illustrated-transformer/
22. Multi-Head Attention Mechanism - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/nlp/multi-head-attention-mechanism/
23. The Transformer Attention Mechanism - MachineLearningMastery.com, 8월 24, 2025에 액세스, https://machinelearningmastery.com/the-transformer-attention-mechanism/
24. A Transformer with Stack Attention, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2405.04515
25. Transformers Explained Visually (Part 3): Multi-head Attention, deep dive - Medium, 8월 24, 2025에 액세스, https://medium.com/data-science/transformers-explained-visually-part-3-multi-head-attention-deep-dive-1c1ff1024853
26. Vision Transformers Explained: The Future of Computer Vision? - Roboflow Blog, 8월 24, 2025에 액세스, https://blog.roboflow.com/vision-transformers/
27. [2212.00057] Part-based Face Recognition with Vision Transformers - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2212.00057
28. Using vision transformers for face recognition - Labelvisor, 8월 24, 2025에 액세스, https://www.labelvisor.com/using-vision-transformers-for-face-recognition/
29. CVPR Poster Learning CNN on ViT: A Hybrid Model to Explicitly Class-specific Boundaries for Domain Adaptation, 8월 24, 2025에 액세스, https://cvpr.thecvf.com/virtual/2024/poster/31056
30. Advancing Face Recognition with Hybrid CNN- ViT-MLP ... - IJRASET, 8월 24, 2025에 액세스, https://www.ijraset.com/best-journal/advancing-face-recognition-with-hybrid-cnn-vit-mlp-a-comparative-study
31. CFormerFaceNet: Efficient Lightweight Network Merging a CNN and Transformer for Face Recognition - MDPI, 8월 24, 2025에 액세스, https://www.mdpi.com/2076-3417/13/11/6506
32. [2505.19985] Structured Initialization for Vision Transformers - arXiv, 8월 24, 2025에 액세스, https://www.arxiv.org/abs/2505.19985
33. FaceLiVT: Face Recognition using Linear Vision Transformer with Structural Reparameterization For Mobile Device - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2506.10361v1
34. IARPA Janus Benchmark-B Face Dataset * - Biometrics Research Group, 8월 24, 2025에 액세스, http://biometrics.cse.msu.edu/Publications/Face/Whitelametal_IARPAJanusBenchmark-BFaceDataset_CVPRW17.pdf
35. Comprehensive Comparison between Vision Transformers and Convolutional Neural Networks for Face Recognition Tasks | OpenReview, 8월 24, 2025에 액세스, https://openreview.net/forum?id=CCo8ElCT7v
36. Vision Transformers (ViT) Explained: Are They Better Than CNNs? | Towards Data Science, 8월 24, 2025에 액세스, https://towardsdatascience.com/vision-transformers-vit-explained-are-they-better-than-cnns/
37. On the speed of ViTs and CNNs - Lucas Beyer, 8월 24, 2025에 액세스, https://lucasb.eyer.be/articles/vit_cnn_speed.html
38. Vision Transformers (ViT) in Image Recognition - GeeksforGeeks, 8월 24, 2025에 액세스, https://www.geeksforgeeks.org/computer-vision/vision-transformers-vit-in-image-recognition/
39. Facial Expression Recognition Based on Squeeze Vision Transformer - PMC, 8월 24, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC9147983/
40. On the Adversarial Robustness of Vision Transformers - OpenReview, 8월 24, 2025에 액세스, https://openreview.net/pdf?id=x9mxhoYRtSP
41. Git Loss for Deep Face Recognition - BMVC 2018, 8월 24, 2025에 액세스, http://bmvc2018.org/contents/workshops/iahfar2018/0012.pdf
42. Triplet loss - Wikipedia, 8월 24, 2025에 액세스, https://en.wikipedia.org/wiki/Triplet_loss
43. FaceNet: A Unified Embedding for Face Recognition and Clustering, 8월 24, 2025에 액세스, https://arxiv.org/abs/1503.03832
44. ArcFace loss function for Deep Face Recognition by payyavula saiprakash - Medium, 8월 24, 2025에 액세스, https://medium.com/@payyavulasaiprakash/arcface-loss-function-for-deep-face-recognition-e1ff5e173b52
45. A Comprehensive Study on Loss Functions for Cross-Factor Face Recognition - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content_CVPRW_2020/papers/w48/Hsu_A_Comprehensive_Study_on_Loss_Functions_for_Cross-Factor_Face_Recognition_CVPRW_2020_paper.pdf
46. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/322674945_ArcFace_Additive_Angular_Margin_Loss_for_Deep_Face_Recognition
47. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content_CVPR_2019/papers/Deng_ArcFace_Additive_Angular_Margin_Loss_for_Deep_Face_Recognition_CVPR_2019_paper.pdf
48. ArcFace: Additive Angular Margin Loss for Deep Face Recognition - InsightFace, 8월 24, 2025에 액세스, https://insightface.ai/arcface
49. Occlusion-Robust Facial Expression Recognition Based on Multi-Angle Feature Extraction, 8월 24, 2025에 액세스, https://www.mdpi.com/2076-3417/15/9/5139
50. ORFormer: Occlusion-Robust Transformer for Accurate Facial Landmark Detection - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2412.13174v1
51. CVPR Poster KeyPoint Relative Position Encoding for Face Recognition, 8월 24, 2025에 액세스, https://cvpr.thecvf.com/virtual/2024/poster/29351
52. [Literature Review] KeyPoint Relative Position Encoding for Face ..., 8월 24, 2025에 액세스, https://www.themoonlight.io/en/review/keypoint-relative-position-encoding-for-face-recognition
53. (PDF) Robust Face Recognition under Varying Illumination and Occlusion Considering Structured Sparsity - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/233927180_Robust_Face_Recognition_under_Varying_Illumination_and_Occlusion_Considering_Structured_Sparsity
54. Understanding and Defending Patched-based Adversarial Attacks ..., 8월 24, 2025에 액세스, https://proceedings.mlr.press/v202/liu23n/liu23n.pdf
55. Towards Efficient Adversarial Training on Vision Transformers, 8월 24, 2025에 액세스, https://www.ecva.net/papers/eccv_2022/papers_ECCV/papers/136730307.pdf
56. ViTGuard: Attention-aware Detection against Adversarial Examples for Vision Transformer, 8월 24, 2025에 액세스, https://arxiv.org/html/2409.13828v1
57. Exploring Adversarial Robustness of Vision Transformers in the Spectral Perspective - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/WACV2024/papers/Kim_Exploring_Adversarial_Robustness_of_Vision_Transformers_in_the_Spectral_Perspective_WACV_2024_paper.pdf
58. When Adversarial Training Meets Vision Transformers: Recipes from Training to Architecture, 8월 24, 2025에 액세스, https://papers.neurips.cc/paper_files/paper/2022/file/760b5def8dcb1156aac454e9c0f5f406-Paper-Conference.pdf
59. Protego: Detecting Adversarial Examples for Vision Transformers via Intrinsic Capabilities, 8월 24, 2025에 액세스, https://arxiv.org/html/2501.07044v1
60. PROTEGO: Detecting Adversarial Examples for Vision Transformers via Intrinsic Capabilities - arXiv, 8월 24, 2025에 액세스, [https://arxiv.org/pdf/2501.07044?](https://arxiv.org/pdf/2501.07044)
61. [2409.13828] ViTGuard: Attention-aware Detection against Adversarial Examples for Vision Transformer - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2409.13828
62. Faces of Fairness: Examining Bias in Facial Expression Recognition Datasets and Models, 8월 24, 2025에 액세스, https://arxiv.org/html/2502.11049v1
63. Mitigating Demographic Bias in Vision Transformers ... - OpenReview, 8월 24, 2025에 액세스, https://openreview.net/pdf?id=cFK8znM1Fi
64. [2502.02309] Review of Demographic Fairness in Face Recognition - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2502.02309
65. [2408.05398] PersonViT: Large-scale Self-supervised Vision Transformer for Person Re-Identification - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2408.05398
66. [2104.02057] An Empirical Study of Training Self-Supervised Vision Transformers - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2104.02057
67. Liveness Detection in Computer Vision: Transformer-based Self-Supervised Learning for Face Anti-Spoofing - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/html/2406.13860v1
68. Emerging Properties in Self-Supervised Vision Transformers, 8월 24, 2025에 액세스, https://arxiv.org/abs/2104.14294
69. Exploring DINO: Fine Tuning DINO Self-Supervised Learning Road ..., 8월 24, 2025에 액세스, https://learnopencv.com/fine-tune-dino-self-supervised-learning-segmentation/
70. DINO: Self-Supervised Vision Transformers - YouTube, 8월 24, 2025에 액세스, https://www.youtube.com/watch?v=8zUfIJdLgnk
71. jacobgil/vit-explain: Explainability for Vision Transformers - GitHub, 8월 24, 2025에 액세스, https://github.com/jacobgil/vit-explain
72. Masked Autoencoders Are Scalable Vision Learners - NYU Scholars, 8월 24, 2025에 액세스, https://nyuscholars.nyu.edu/en/publications/masked-autoencoders-are-scalable-vision-learners
73. Masked Autoencoders Are Scalable Vision ... - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2022/papers/He_Masked_Autoencoders_Are_Scalable_Vision_Learners_CVPR_2022_paper.pdf
74. Masked autoencoder (MAE) for visual representation learning. Form the author of ResNet. - Michał Chromiak's blog, 8월 24, 2025에 액세스, https://mchromiak.github.io/articles/2021/Nov/14/Masked-Autoencoders-Are-Scalable-Vision-Learners/
75. [PDF] FaceMAE: Privacy-Preserving Face Recognition via Masked Autoencoders, 8월 24, 2025에 액세스, https://www.semanticscholar.org/paper/FaceMAE%3A-Privacy-Preserving-Face-Recognition-via-Wang-Zhao/1e87ef950ef8c979bca10e20c7966b9a52a10cd3
76. FACEMAE: PRIVACY-PRESERVING FACE ... - OpenReview, 8월 24, 2025에 액세스, https://openreview.net/pdf?id=4Xd_aAqNe7h
77. FaceMAE: Privacy-Preserving Face Recognition via Masked Autoencoders | Request PDF, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/360803605_FaceMAE_Privacy-Preserving_Face_Recognition_via_Masked_Autoencoders
78. FaceMAE: Privacy-Preserving Face Recognition via Masked Autoencoders - OpenReview, 8월 24, 2025에 액세스, https://openreview.net/forum?id=4Xd_aAqNe7h
79. FaceMAE: Privacy-Preserving Face Recognition via Masked Autoencoders - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2205.11090
80. Multi-modal ML with OpenAI's CLIP | Pinecone, 8월 24, 2025에 액세스, https://www.pinecone.io/learn/series/image-search/clip/
81. CLIP: Connecting text and images - OpenAI, 8월 24, 2025에 액세스, https://openai.com/index/clip/
82. CLIP Model and The Importance of Multimodal Embeddings - Medium, 8월 24, 2025에 액세스, https://medium.com/data-science/clip-model-and-the-importance-of-multimodal-embeddings-1c8f6b13bf72
83. Top Multimodal Vision Models - Roboflow, 8월 24, 2025에 액세스, https://roboflow.com/model-feature/multimodal-vision
84. openai/clip-vit-base-patch32 - Hugging Face, 8월 24, 2025에 액세스, https://huggingface.co/openai/clip-vit-base-patch32
85. Text-Guided Face Recognition Using Multi ... - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/WACV2024/papers/Hasan_Text-Guided_Face_Recognition_Using_Multi-Granularity_Cross-Modal_Contrastive_Learning_WACV_2024_paper.pdf
86. 2nd Edition FRCSyn: Face Recognition Challenge in the Era of ..., 8월 24, 2025에 액세스, https://frcsyn.github.io/CVPR2024.html
87. Synthetic Face Recognition - Papers With Code, 8월 24, 2025에 액세스, https://paperswithcode.com/task/synthetic-face-recognition
88. Digi2Real: Bridging the Realism Gap in ... - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/WACV2025W/SynRDinBAS/papers/George_Digi2Real_Bridging_the_Realism_Gap_in_Synthetic_Data_Face_Recognition_WACVW_2025_paper.pdf
89. Diversify, Don't Fine-Tune: Scaling Up Visual Recognition Training with Synthetic Images, 8월 24, 2025에 액세스, https://arxiv.org/html/2312.02253v2
90. Video Vision Transformers for Violence Detection - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/pdf/2209.03561
91. ViViT: A Video Vision Transformer - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/ICCV2021/papers/Arnab_ViViT_A_Video_Vision_Transformer_ICCV_2021_paper.pdf
92. [2103.15691] ViViT: A Video Vision Transformer - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2103.15691
93. A 3D Face Recognition Algorithm Directly Applied to Point Clouds - PMC - PubMed Central, 8월 24, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11853279/
94. PCT: Point cloud transformer, 8월 24, 2025에 액세스, https://d-nb.info/1234895854/34
95. CVPR 2024 Biometrics Workshop - CVF Open Access - The Computer Vision Foundation, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/CVPR2024_workshops/BIOMET
96. Thermal Image Generation for Robust Face Recognition - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/357627334_Thermal_Image_Generation_for_Robust_Face_Recognition
97. Thermal Image Generation for Robust Face Recognition - MDPI, 8월 24, 2025에 액세스, https://www.mdpi.com/2076-3417/12/1/497
98. One Embedding to Predict Them All: Visible and Thermal Universal ..., 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2024W/BIOMET/html/Mirabet-Herranz_One_Embedding_to_Predict_Them_All_Visible_and_Thermal_Universal_CVPRW_2024_paper.html
99. Visual Explanations of Vision Transformer Guided by Self-Attention - arXiv, 8월 24, 2025에 액세스, https://arxiv.org/abs/2402.04563
100. Online Continual Learning with Contrastive Vision Transformer, 8월 24, 2025에 액세스, https://www.ecva.net/papers/eccv_2022/papers_ECCV/papers/136800614.pdf
101. Continual Learning With Lifelong Vision ... - CVF Open Access, 8월 24, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2022/papers/Wang_Continual_Learning_With_Lifelong_Vision_Transformer_CVPR_2022_paper.pdf
102. Improving Vision Transformers for Incremental Learning | Request PDF - ResearchGate, 8월 24, 2025에 액세스, https://www.researchgate.net/publication/357013847_Improving_Vision_Transformers_for_Incremental_Learning


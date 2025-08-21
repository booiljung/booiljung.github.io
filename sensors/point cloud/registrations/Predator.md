# Predator 모델 저중첩(Low-Overlap) 환경에서의 3D 포인트클라우드 정합



## 1. 서론





### 0.1 포인트클라우드 정합의 정의와 핵심 과제



포인트클라우드 정합(Point Cloud Registration)은 서로 다른 시점이나 센서로부터 획득된 복수의 3차원 포인트클라우드 데이터셋을 하나의 일관된 좌표계로 정렬하는 컴퓨터 비전의 근본적인 기술이다.1 수학적으로 이 문제는 소스 포인트클라우드 

$P = \{p_i\}_{i=1}^{N}$를 타겟 포인트클라우드 $Q = \{q_j\}_{j=1}^{M}$에 최적으로 정렬하는 강체 변환(Rigid Transformation) $T$를 찾는 과정으로 정의된다. 이 변환 $T$는 회전 행렬 $R \in SO(3)$과 변환 벡터 $t \in \mathbb{R}^3$로 구성되며, 목표는 두 포인트클라우드 간의 대응점(correspondences) $(p_i, q_i)$ 사이의 정렬 오차를 최소화하는 최적의 변환 $T^*$를 찾는 것이다.


$$
T^* = \arg \min_T \sum_{i} \| T(p_i) - q_i \|^2 = \arg \min_{R, t} \sum_{i} \| (R p_i + t) - q_i \|^2
$$
이 기술은 자율주행 차량의 라이다(LiDAR) 데이터를 이용한 동시적 위치추정 및 지도작성(SLAM), 로봇 공학에서의 정밀한 환경 인식, 여러 장의 의료 영상(CT, MRI)을 정합하여 3차원 모델을 생성하는 의료 이미징, 그리고 분할된 3D 스캔 데이터를 결합하여 완전한 디지털 모델을 재구성하는 등 광범위한 첨단 산업 분야에서 필수적인 전처리 단계로 자리 잡고 있다.1



### 0.2 저중첩 시나리오의 기술적 난제



실제 응용 환경에서는 포인트클라우드 데이터 획득 시 센서의 제한된 시야각(Field-of-View), 기둥이나 다른 물체에 의한 폐색(Occlusion), 또는 동적 객체의 존재로 인해 스캔 간의 공통 영역, 즉 중첩(overlap) 영역이 30% 미만으로 매우 낮은 경우가 빈번하게 발생한다.3 이러한 저중첩(low-overlap) 환경은 정합 알고리즘의 성능을 급격히 저하시키는 가장 큰 기술적 난제로 꼽힌다.

전통적인 정합 알고리즘인 ICP(Iterative Closest Point)와 그 변형들은 초기 정렬 상태에 매우 민감하기 때문에, 중첩 영역이 작을 경우 잘못된 대응점을 기반으로 최적화를 시작하여 지역 최적해(local minima)에 수렴할 확률이 매우 높다.3 이를 보완하기 위해 사용되는 FPFH(Fast Point Feature Histograms)와 같은 수작업 특징(handcrafted feature) 기반의 전역 정합(global registration) 방법론 역시 한계를 보인다. 이들은 각 점의 지역적 기하학 정보를 요약한 기술자를 비교하여 대응점을 찾는데, 중첩 영역이 작아지면 구별 가능하고 반복적으로 나타나는 특징점의 수가 급격히 줄어들어 신뢰할 수 있는 대응점을 찾는 것 자체가 불가능해지고, 결과적으로 정합 실패율이 기하급수적으로 증가한다.8



### 0.3 딥러닝 기반 접근법의 대두와 Predator의 의의



이러한 전통적 방법론의 한계를 극복하기 위해, 대규모 데이터셋으로부터 강인한 특징 기술자(feature descriptor)를 학습하고, 더 나아가 정합 과정 전체를 최적화하는 딥러닝 기반 접근법이 활발히 연구되었다.1 초기 딥러닝 모델들은 주로 특징 기술자의 표현력을 강화하는 데 집중했으나, 저중첩 문제에 대한 근본적인 해결책을 제시하지는 못했다.

이러한 배경 속에서 등장한 Predator 모델(Huang et al., 2021)은 포인트클라우드 정합 분야, 특히 저중첩 문제 해결에 있어 중요한 패러다임의 전환을 제시했다.6 기존 방법들이 정합 파이프라인의 후반부에서 기하학적 불일치를 기준으로 아웃라이어(outlier)를 '제거'하는 수동적인 접근 방식을 취했다면, Predator는 두 포인트클라우드 간의 상호 컨텍스트를 조기에 교환하는 어텐션 메커니즘을 통해 정합에 유용할 가능성이 높은 '중첩 영역' 자체를 능동적으로 '예측'하고, 해당 영역에 자원을 집중하는 새로운 방식을 도입했다.6 이는 단순히 더 좋은 특징을 학습하는 것을 넘어, 정합 문제 해결의 파이프라인 자체를 근본적으로 최적화한 것이다. 이 접근 방식은 문제의 본질을 '좋은 대응점 찾기'에서 '대응점이 존재할 만한 공간 찾기'로 재정의함으로써, 저중첩이라는 극한 환경에서도 강인한 성능을 발휘할 수 있는 길을 열었다. 본 보고서는 Predator 모델의 아키텍처, 수학적 원리, 그리고 벤치마크 성능을 심층적으로 고찰하고, 그 기술적 한계와 산업적 함의를 종합적으로 분석하고자 한다.



## 1.  Predator 아키텍처 분석





### 1.1 설계 철학: 저중첩 극복을 위한 중첩 영역 집중



Predator의 핵심 설계 철학은 인간이 시각 정보를 처리하는 방식을 모방하는 데 있다. 인간은 두 개의 복잡한 이미지를 비교하여 정렬해야 할 때, 전체 이미지를 무작위로 훑어보는 것이 아니라, 두 이미지에 공통적으로 나타나는 특징적인 영역에 시각적 주의(attention)를 먼저 집중한다.5 Predator는 이러한 직관을 신경망 아키텍처로 구현하고자 했다.

기존의 특징점 기반 방법들은 기하학적으로 돌출되거나 특이한 점(salient point)을 찾는 데 주력했다. 그러나 저중첩 시나리오에서는 이러한 돌출된 점이 중첩 영역 밖에 존재할 수 있어 정합에 아무런 기여를 하지 못하는 경우가 많다. Predator는 이 문제를 해결하기 위해 목표를 재정의했다. 즉, 단순히 기하학적으로 돌출된 점을 찾는 것을 넘어, **두 포인트클라우드 모두에 존재할 가능성이 높아 실제 정합에 기여할 수 있는 점**을 식별하는 것을 최우선 목표로 삼았다.6 이 목표를 달성하기 위해 제안된 것이 바로 '중첩 어텐션 블록(Overlap-Attention Block)'이다.



### 1.2 네트워크 구조 상세



Predator는 전체적으로 인코더-병목-디코더(Encoder-Bottleneck-Decoder)라는 대칭적인 구조를 따른다. 이 구조는 입력된 포인트클라우드로부터 계층적인 특징을 추출하고, 병목 구간에서 핵심적인 정보 교환을 수행한 후, 다시 원본 해상도로 복원하여 각 점에 대한 풍부한 정보를 생성한다.6



#### 1.2.1 ) 인코더 (Encoder)



인코더는 입력으로 주어진 두 개의 복셀화된(voxelized) 포인트클라우드 $P$와 $Q$를 받아, 일련의 컨볼루션 연산을 통해 계층적으로 다운샘플링하며 각 점의 지역적, 전역적 기하학 특징을 추출한다. 이 과정에서 D3Feat 모델에서 제안된 KPConv(Kernel Point Convolution)를 기반으로 구현되어, 점들의 밀도가 불균일하고 구조가 비정형적인 포인트클라우드 데이터를 효과적으로 처리할 수 있다.15 인코더의 각 계층을 통과하면서 포인트의 수는 줄어들고 특징 벡터의 차원은 증가한다. 최종적으로 인코더는 원본보다 훨씬 적은 수의 '슈퍼포인트(superpoints)'와, 각 슈퍼포인트에 해당하는 고차원의 잠재 특징 벡터(latent feature vectors) $X^{P'}$, $X^{Q'}$를 출력한다. 이 잠재 특징 벡터는 각 슈퍼포인트 주변의 기하학적 정보를 압축적으로 담고 있다.6



#### 1.2.2 ) 병목: 중첩 어텐션 모듈 (Bottleneck: Overlap-Attention Module)



이 모듈은 Predator 아키텍처의 심장부로서, 저중첩 문제를 해결하는 핵심적인 역할을 수행한다. 인코더로부터 추출된 두 포인트클라우드의 잠재 특징 $X^{P'}$와 $X^{Q'}$을 입력으로 받아, 두 포인트클라우드 간의 '조기 정보 교환(early information exchange)'을 수행한다. 이 과정은 Self-Attention과 Cross-Attention 두 단계로 구성된다.

- **Self-Attention (GNN 기반):** 먼저, 각 포인트클라우드 내부에서 슈퍼포인트 간의 관계를 학습하여 내부적인 컨텍스트 정보를 강화한다. 이는 그래프 신경망(GNN)의 형태로 구현되며, 한 슈퍼포인트의 특징을 업데이트할 때 이웃 슈퍼포인트들의 정보를 종합적으로 고려한다. 이를 통해 각 슈퍼포인트는 자신의 지역적 정보를 넘어 더 넓은 범위의 구조적 맥락을 이해하게 된다.14
- **Cross-Attention (트랜스포머 기반):** 다음으로, 두 포인트클라우드 간의 본격적인 정보 교환이 이루어진다. 소스 포인트클라우드 $P$의 한 슈퍼포인트 특징은 타겟 포인트클라우드 $Q$의 모든 슈퍼포인트 특징들과 어텐션 연산을 수행한다. 이 과정을 통해 각 슈퍼포인트는 상대방 포인트클라우드의 전체적인 정보를 반영한 '상호 컨텍스트(co-contextual)' 특징으로 업데이트된다.5 이 상호 컨텍스트 정보는 "나의 기하학적 모양이 상대방 클라우드의 어떤 부분들과 유사한가?"에 대한 답을 담고 있다.

이처럼 강화된 특징을 기반으로, 어텐션 모듈은 각 슈퍼포인트에 대해 두 가지 중요한 확률 점수를 예측한다.

1. **중첩 점수 (Overlap scores, $o_P, o_Q$):** 해당 슈퍼포인트가 실제 두 포인트클라우드의 중첩 영역에 위치할 확률.
2. **교차-중첩 점수 (Cross-overlap scores, $\tilde{o}_P, \tilde{o}_Q$):** 해당 슈퍼포인트의 잠재적 대응점(soft-correspondence)이 상대방 클라우드의 중첩 영역에 위치할 확률.



#### 1.2.3 ) 디코더 (Decoder)



디코더는 어텐션 모듈을 통해 상호 컨텍스트 정보로 풍부해진 잠재 특징과 중첩 관련 점수들을 다시 업샘플링하여, 원본 포인트클라우드의 모든 점에 대한 상세 정보를 생성하는 역할을 한다. 이 과정은 인코더의 다운샘플링 과정을 역으로 수행하며, 각 계층에서 인코더의 특징을 스킵 연결(skip connection)을 통해 전달받아 정보 손실을 최소화한다. 최종적으로 디코더는 원본 포인트클라우드 $P$와 $Q$의 모든 점 $p \in P, q \in Q$에 대해 다음과 같은 세 가지 정보를 출력한다.6

1. **점별 특징 기술자 (Per-point feature descriptors, $F_P, F_Q$):** 대응점 매칭에 사용될 고유한 32차원 특징 벡터. 이 기술자는 해당 점의 지역적 기하학 정보뿐만 아니라, 상대방 클라우드와의 관계까지 인코딩하고 있다.
2. **점별 중첩 점수 (Per-point overlap scores, $o_P, o_Q$):** 해당 점이 중첩 영역에 존재할 확률을 나타내는 스칼라 값.
3. **점별 정합 가능성 점수 (Per-point matchability scores, $m_P, m_Q$):** 해당 점이 평면과 같은 모호한 영역이 아닌, 신뢰할 수 있는 대응점을 가질 수 있는 독특한 기하학적 구조 위에 있을 확률.



### 1.3 어텐션 메커니즘의 작동 원리



Predator의 성공은 '무엇(What)'을 매칭할 것인가와 '어디서(Where)' 매칭할 것인가라는 두 가지 근본적인 질문을 단일 네트워크 내에서 통합적으로 학습하고 추론하는 능력에서 비롯된다. 기존의 방법론들은 이 두 질문을 분리된 단계로 처리했다. 예를 들어, FPFH는 '무엇'(지역 기하학)을 기술하는 데 집중하고, RANSAC은 '어디서'(전체 공간에서 무작위로) 매칭을 시도하는 방식이었다. FCGF나 D3Feat 같은 초기 딥러닝 모델들은 '무엇'(학습된 기술자)의 성능을 고도화했지만, '어디서'의 문제는 여전히 RANSAC과 같은 후처리 과정에 의존했다.7

반면, Predator는 디코더를 통해 점별 특징 기술자($F_P, F_Q$)와 함께 중첩 점수($o_P, o_Q$), 정합 가능성 점수($m_P, m_Q$)를 동시에 출력한다. 특징 기술자는 '무엇을' 매칭할 것인가에 대한 답을 제공하고, 두 가지 확률 점수의 조합은 '어디서' 매칭을 시도해야 하는지에 대한 정교한 확률적 가이드를 제공한다. 이 두 가지 과제는 분리되어 학습되는 것이 아니라, Cross-Attention 메커니즘을 통해 상호 보완적으로 학습된다. 예를 들어, 소스 포인트클라우드 $P$의 정보를 알아야 타겟 $Q$의 중첩 영역을 더 정확하게 예측할 수 있고, 이는 다시 $P$의 특징 기술자를 더 강인하게 만드는 긍정적인 피드백 루프를 형성한다. 이것이 Predator가 강조하는 '조기 정보 교환'의 진정한 의미이다.6

더 나아가, Cross-Attention의 도입은 포인트클라우드 정합을 개별적인 특징 기술 문제에서 '관계 추론(Relational Reasoning)' 문제로 격상시켰다. PointNetLK나 DCP와 같은 초기 모델들은 각 포인트클라우드를 독립적으로 처리하여 특징을 추출한 후, 별도의 단계에서 이들을 비교했다.10 이는 두 포인트클라우드 간의 관계가 특징 추출 과정 자체에는 영향을 미치지 못함을 의미한다. 그러나 Predator의 Cross-Attention 모듈은 

$P$의 한 슈퍼포인트 특징을 업데이트할 때, $Q$의 모든 슈퍼포인트 정보를 가중합하여 사용한다.5 이는 네트워크가 

$P$의 한 점이 $Q$의 어떤 점들과 기하학적으로, 그리고 의미적으로 관련이 있는지를 능동적으로 추론하도록 강제한다. 결과적으로 생성되는 특징 기술자는 단순히 해당 점의 고유한 기하학 정보뿐만 아니라, 상대방 클라우드와의 관계까지 풍부하게 인코딩하게 된다. 이는 특징의 정보량을 "나는 이런 모양을 가졌다"에서 "나는 이런 모양을 가졌고, 상대방의 저런 부분과 유사하다"로 확장시키는 효과를 낳는다. 이 관계적 정보야말로 저중첩 상황에서 부족한 기하학적 단서를 보완하는 핵심적인 역할을 수행한다.



## 2.  수학적 형식화 및 구현





### 2.1 강체 변환의 수학적 정의



3차원 유클리드 공간에서의 강체 변환 $T$는 객체의 형태나 크기를 변경하지 않고 위치와 방향만을 바꾸는 변환으로, 회전(Rotation)과 평행이동(Translation)의 조합으로 정의된다. 소스 포인트클라우드 $P$에 속한 임의의 점 $p$는 변환 $T$에 의해 새로운 위치 $T(p)$로 이동한다.18

이 변환은 수학적으로 3x3 회전 행렬 $R$과 3x1 변환 벡터 $t$를 사용하여 다음과 같이 표현할 수 있다.1
$$
T(p) = R \cdot p + t
$$
여기서 회전 행렬 $R$은 직교 행렬($R^T R = I$)이면서 행렬식(determinant) 값이 +1인($\det(R) = 1$) 조건을 만족하는 특수 직교 그룹 $SO(3)$에 속한다. 이 조건은 변환이 객체의 대칭성을 바꾸는 반사(reflection)를 포함하지 않는 순수한 회전임을 보장한다.19 변환 벡터 $t \in \mathbb{R}^3$는 3차원 공간에서의 x, y, z축 방향 이동량을 나타낸다.

포인트클라우드 정합의 최종 목표는, 사전에 알려진 대응점 쌍 $(p_i, q_i)$들에 대해 이들 간의 유클리드 거리 제곱의 합을 최소화하는 최적의 회전 행렬 $R^*$과 변환 벡터 $t^*$를 찾는 것이다.1
$$
(R^*, t^*) = \arg \min_{R \in SO(3), t \in \mathbb{R}^3} \sum_{i} \| (R \cdot p_i + t) - q_i \|^2
$$


### 2.2 손실 함수와 학습 전략



Predator는 단일 목표를 최적화하는 대신, 네트워크가 출력하는 세 종류의 정보(특징 기술자, 중첩 점수, 정합 가능성 점수)를 동시에 학습시키기 위해 다중 손실 함수(multi-task loss)를 사용한다. 이는 머신러닝 모델이 예측한 값과 실제 정답(ground truth) 간의 차이를 정량화하여, 역전파(backpropagation)를 통해 네트워크의 가중치를 올바른 방향으로 업데이트하는 역할을 한다.21

Predator의 손실 함수 설계는 정합 문제를 '기하학적 일치'와 '확률적 추론'이라는 두 가지 상호 보완적인 측면으로 분해하여 접근한다. '기하학적 일치'는 특징 기술자 손실을 통해 달성되며, 이는 정합의 근본적인 목표에 해당한다. '확률적 추론'은 중첩 및 정합 가능성 점수 손실을 통해 구현되며, 이는 기하학적 정보만으로는 해결할 수 없는 모호성을 해결하고 정합 파이프라인의 효율성을 높이는 역할을 한다.

- **특징 기술자 손실 (Descriptor Loss):** 이 손실 함수는 '기하학적 일치'를 담당한다. Ground-truth 변환을 통해 확인된 실제 대응점 쌍 $(p, q)$의 특징 기술자 $F_p$와 $F_q$는 특징 공간상에서 거리가 가까워지도록, 대응점이 아닌 쌍 $(p, q')$의 기술자 $F_p$와 $F_{q'}$는 거리가 멀어지도록 학습시킨다. 이는 일반적으로 서클 손실(Circle Loss)이나 대조 손실(Contrastive Loss)과 같은 메트릭 러닝(metric learning) 손실 함수를 통해 구현된다. 이를 통해 기하학적으로 유사한 지점들이 유사한 특징 벡터를 갖도록 강제한다.

- **중첩 점수 손실 (Overlap Score Loss):** 이 손실 함수는 '확률적 추론'의 일부로, 각 점이 중첩 영역에 속할 확률 $o_p$를 예측하는 분류 문제로 정의된다. Ground-truth 변환을 기준으로 실제 중첩 영역에 속하는 점들에는 레이블 1을, 속하지 않는 점들에는 레이블 0을 부여한다. 저중첩 시나리오에서는 비중첩 영역의 점들이 압도적으로 많아 심각한 클래스 불균형 문제가 발생한다. 이를 해결하기 위해 소수 클래스(중첩 영역)에 더 높은 가중치를 부여하는 가중 이진 교차 엔트로피(Weighted Binary Cross-Entropy, WBCE) 손실 함수를 사용한다.4
  $$
  L_{overlap} = - \frac{1}{N} \sum_{i=1}^{N} [ w \cdot y_i \log(o_{p_i}) + (1-y_i) \log(1-o_{p_i}) ]
  $$
  

  여기서 $y_i$는 점 $p_i$가 실제 중첩 영역에 속하면 1, 아니면 0인 이진 레이블이며, $w$는 클래스 불균형을 보정하기 위한 가중치다.

- **정합 가능성 점수 손실 (Matchability Score Loss):** Predator는 신뢰할 수 있는 특징점을 샘플링하는 능력을 학습하기 위해 '정합 가능성 점수' $m_p$와 이를 위한 새로운 손실 함수를 제안했다.7 이 점수는 해당 점이 평평한 벽면이나 반복적인 패턴처럼 모호하지 않고, 안정적이며 반복적으로 검출될 수 있는 독특한 기하학적 구조에 위치하는지를 나타낸다. 이 점수를 학습함으로써, 후속 RANSAC 단계에서 더 유망한 점들을 우선적으로 샘플링하여 효율성과 정확도를 높일 수 있다.



### 2.3 특징 매칭 및 변환 추정 파이프라인



학습이 완료된 Predator 네트워크는 추론(inference) 단계에서 두 개의 입력 포인트클라우드 $P$와 $Q$에 대해 점별 특징 기술자, 중첩 점수, 정합 가능성 점수를 출력한다. 이 정보들은 다음과 같은 파이프라인을 통해 최종적인 강체 변환 $(R, t)$를 추정하는 데 사용된다.

1. **대응점 후보군 생성 (Candidate Correspondence Generation):** 먼저, $P$의 각 점 $p_i$에 대해, 학습된 특징 공간상에서 유클리드 거리가 가장 가까운 $Q$의 점 $q_j$를 탐색한다(Nearest Neighbor Search). 이 과정을 통해 초기 대응점 후보 쌍 $(p_i, q_j)$들의 집합을 생성한다. 이 단계에서는 아직 아웃라이어가 다수 포함되어 있다.

2. **대응점 필터링 및 샘플링 (Correspondence Filtering and Sampling):** 모든 대응점 후보를 사용하는 대신, Predator는 네트워크가 예측한 확률 점수들을 활용하여 '현명한' 샘플링 전략을 구사한다. 각 대응점 쌍 $(p_i, q_j)$의 신뢰도를 두 점의 중첩 점수($o_{p_i}, o_{q_j}$)와 정합 가능성 점수($m_{p_i}, m_{q_j}$)를 결합하여 계산한다. 예를 들어, 신뢰도 점수는 $o_{p_i} \times m_{p_i} \times o_{q_j} \times m_{q_j}$ 와 같이 정의될 수 있다. 후속 RANSAC 단계에서는 이 신뢰도가 높은 대응점들로부터 우선적으로 샘플을 추출한다.7

3. **강인한 변환 추정 (Robust Transformation Estimation):** RANSAC(Random Sample Consensus) 알고리즘이 이 확률적 샘플링을 기반으로 동작한다.9 RANSAC은 반복적으로 최소한의 대응점(3D 강체 변환의 경우 3쌍)을 무작위로 샘플링하여 변환 모델 

   $(R_k, t_k)$를 추정한다. 그 후, 이 모델을 전체 대응점 후보군에 적용하여 모델을 지지하는 인라이어(inlier)의 수를 센다. 이 과정을 수차례 반복한 후, 가장 많은 인라이어를 확보한 변환 모델을 최종 결과로 채택한다. Predator의 확률적 샘플링 전략은 인라이어일 확률이 높은 대응점들을 집중적으로 샘플링하게 하므로, RANSAC이 훨씬 적은 반복 횟수로도 정확한 변환을 찾을 확률을 극적으로 높여준다.



## 3.  벤치마크 기반 성능 평가





### 3.1 평가 데이터셋 및 지표 소개



Predator의 성능은 다양한 시나리오를 포괄하는 표준 벤치마크 데이터셋을 통해 종합적으로 평가되었다. 각 데이터셋은 고유한 특성을 가지며, 알고리즘의 특정 측면을 검증하는 데 사용된다.

- **데이터셋:**
  - **3DMatch:** 다양한 실내 공간(예: 침실, 사무실, 부엌)에서 여러 종류의 상용 깊이 센서(예: Kinect, RealSense)로 촬영된 RGB-D 스캔 데이터로 구성된 대규모 데이터셋이다. 딥러닝 기반 실내 포인트클라우드 정합 알고리즘의 성능을 평가하는 사실상의 표준 벤치마크로 널리 사용된다.15 일반적으로 평가에는 중첩률이 30% 이상인 '고중첩(high-overlap)' 스캔 쌍들이 사용된다.
  - **3DLoMatch:** Predator 논문에서 저중첩 시나리오에서의 성능을 보다 엄밀하게 검증하기 위해 제안된 새로운 벤치마크이다. 이는 기존 3DMatch 데이터셋에서 의도적으로 중첩률이 낮은(10% ~ 30%) 스캔 쌍들만을 선별하여 구성되었다.6 이 데이터셋은 알고리즘이 극한의 조건에서 얼마나 강인한지를 평가하는 중요한 척도가 된다.
  - **KITTI Odometry:** 자율주행 연구를 위해 제작된 대규모 실외 데이터셋으로, 주행 중인 차량에 장착된 Velodyne LiDAR 스캐너로 수집된 포인트클라우드 시퀀스로 구성되어 있다. 실외 환경의 복잡성(예: 동적 객체, 희소한 데이터, 넓은 스케일) 속에서 정합 알고리즘의 성능과 다른 도메인에 대한 일반화 능력을 평가하는 데 사용된다.15
- **평가 지표 (Evaluation Metrics):**
  - **Registration Recall (RR):** 정합 성공률. 가장 중요하고 직관적인 종단 간(end-to-end) 성능 지표이다. 예측된 변환 행렬을 소스 포인트클라우드에 적용했을 때, ground-truth 대응점들 간의 평균 제곱근 오차(RMSE)가 특정 임계값(예: 3DMatch에서는 0.2m) 미만인 스캔 쌍의 비율을 나타낸다. RR이 높을수록 성공적으로 정합하는 경우가 많음을 의미한다.4
  - **Feature Match Recall (FMR):** 특징 매칭 성공률. RANSAC과 같은 강인한 추정기가 올바른 변환을 복원할 수 있을 만큼 충분한 수의 올바른 대응점(inliers)을 특징 매칭 단계에서 찾았는지의 여부를 평가하는 지표이다.15
  - **Relative Rotation Error (RRE) / Translation Error (RTE):** 상대 회전/이동 오차. 예측된 회전 행렬 및 변환 벡터가 ground-truth 값과 얼마나 차이 나는지를 측정한다. 정합의 정밀도를 나타내는 지표로, 값이 낮을수록 정확하다.15



### 3.2 정량적 성능 비교 분석



Predator의 성능을 객관적으로 평가하기 위해, 전통적인 방법론 및 동시대의 다른 딥러닝 모델들과의 정량적 비교를 수행하였다.



#### 3.2.1 Table 1: Predator vs. 전통적 방법 (3DLoMatch 벤치마크)



이 비교는 저중첩이라는 극한 환경에서 Predator의 접근 방식이 전통적인 기하학 기반 방법론에 비해 얼마나 압도적인 우위를 보이는지를 명확히 보여준다.

| Method                        | Registration Recall (RR, %) |
| ----------------------------- | --------------------------- |
| ICP (Iterative Closest Point) | < 10 (추정)                 |
| FPFH + RANSAC                 | ~35 (추정)                  |
| **Predator**                  | **64.2**                    |

*분석:* 전통적인 방법인 ICP는 초기 정렬에 의존하므로 저중첩 환경에서는 거의 작동하지 않는다. FPFH와 RANSAC을 결합한 전역 정합 방식 역시 인라이어 비율이 너무 낮아 RANSAC이 유효한 모델을 찾지 못하고 실패하는 경우가 대부분이다.10 반면, Predator는 중첩 영역을 먼저 예측하고 해당 영역 내에서 신뢰도 높은 점들을 샘플링하는 전략을 통해 60%가 넘는 높은 정합 성공률을 달성했다. 이는 저중첩이 전통적 방법의 근본적인 실패 지점이며, Predator의 설계 철학이 이 문제를 효과적으로 해결했음을 정량적으로 입증한다.6



#### 3.2.2 Table 2: Predator vs. 동시대 딥러닝 모델 (3DMatch & 3DLoMatch 벤치마크)



이 비교는 Predator가 다른 최신 딥러닝 모델들과 비교하여, 특히 저중첩 시나리오에서 어떤 차별화된 경쟁 우위를 갖는지 분석하는 데 목적이 있다.

| Method       | 3DMatch RR (%) | 3DLoMatch RR (%) |
| ------------ | -------------- | ---------------- |
| FCGF         | 85.1           | 34.3             |
| D3Feat       | 81.6           | 37.2             |
| PointNetLK   | (성능 낮음)    | (성능 낮음)      |
| DCP          | (성능 낮음)    | (성능 낮음)      |
| **Predator** | **90.6**       | **64.2**         |

데이터 출처: 26 및 관련 논문 수치 종합



분석: 3DMatch(고중첩) 벤치마크에서도 Predator는 당시 최고 수준(SOTA)의 성능을 기록했지만, 그 진정한 가치는 3DLoMatch(저중첩) 벤치마크에서 드러난다. FCGF나 D3Feat와 같이 강력한 특징 기술자를 학습하는 데 집중한 모델들은 중첩률이 높은 환경에서는 우수한 성능을 보이지만, 저중첩 환경에서는 RR이 30%대로 급격히 하락하는 취약점을 보였다. 반면, Predator는 저중첩 환경에서도 60% 이상의 높은 성공률을 안정적으로 유지했다.

이러한 성능 격차는 포인트클라우드 정합 연구에서 '저중첩'이 단순히 어려운 케이스가 아니라, 독립적인 연구 주제로 다뤄져야 할 만큼 중요하고 근본적으로 다른 문제임을 정량적으로 보여준다. 과거 연구들은 주로 3DMatch와 같이 비교적 정합이 용이한 데이터셋에 집중했으며, 이 환경에서는 특징 기술자의 표현력만 높여도 성능 향상을 이룰 수 있었다.24 Predator는 3DLoMatch라는 '스트레스 테스트'를 통해 기존 모델들이 저중첩에 얼마나 취약한지를 드러냈고, 고중첩 환경에서의 성공이 저중첩 환경에서의 성공을 보장하지 않음을 명확히 했다. 이는 단순히 좋은 특징을 넘어 '중첩 영역 인식'이라는 Predator의 설계 철학이 저중첩 문제 해결의 핵심 열쇠임을 증명하는 강력한 증거다.



#### 3.2.3 Table 3: KITTI Odometry 벤치마크 성능



이 평가는 실내 RGB-D 데이터로 학습된 Predator 모델이 대규모 실외 LiDAR 데이터라는 완전히 다른 도메인에서도 우수한 일반화 성능을 보이는지 검증한다.

| Method       | RRE (deg) | RTE (cm) |
| ------------ | --------- | -------- |
| FCGF         | 9.5       | 30       |
| D3Feat       | 7.2       | 30       |
| **Predator** | **6.8**   | **27**   |

데이터 출처: 29



분석: Predator는 실외 자율주행 데이터셋인 KITTI에서도 기존 SOTA 모델들보다 낮은 상대 회전 오차(RRE)와 상대 이동 오차(RTE)를 기록하며 뛰어난 일반화 성능을 입증했다. 이는 Predator가 학습한 '중첩 영역 추론' 능력이 특정 센서나 실내 환경의 특성에 과적합된 것이 아니라, 보다 근본적이고 일반적인 기하학적 단서를 기반으로 작동함을 시사한다. 이 결과는 자율주행과 같은 실제 산업 응용에서의 적용 가능성을 크게 높이는 중요한 성과이다.



## 4.  한계점 및 연구 발전 방향



Predator는 저중첩 포인트클라우드 정합 분야에서 혁신적인 성과를 거두었지만, 동시에 딥러닝 기반 방법론이 가지는 내재적인 한계점들을 드러내며 후속 연구의 방향성을 제시했다.



### 4.1 Predator의 내재적 한계



- **학습 데이터 의존성:** Predator는 지도 학습(supervised learning) 방식의 모델로서, 학습을 위해 ground-truth 변환 정보가 정확하게 레이블링된 대규모 3D 스캔 쌍 데이터셋을 필요로 한다. 현실 세계에서 이러한 고품질의 학습 데이터를 구축하는 것은 막대한 비용과 시간이 소요되는 작업이며, 이는 모델의 실용적인 적용 범위를 제한하는 가장 큰 장벽 중 하나이다.23
- **미관측 환경에 대한 일반화 성능의 한계:** KITTI 데이터셋에서 인상적인 일반화 성능을 보였음에도 불구하고, Predator는 학습 데이터의 분포와 현저하게 다른 새로운 환경에 대해서는 성능 저하를 겪을 수 있다. 예를 들어, 학습에 사용되지 않은 새로운 종류의 LiDAR 센서 데이터, 극도로 희소하거나 노이즈가 심한 포인트클라우드, 또는 완전히 다른 유형의 객체(예: 유기체)에 대해서는 정합 정확도가 보장되지 않는다. 이는 딥러닝 모델의 고질적인 도메인 이동(domain shift) 문제이다.31
- **대칭적/반복적 구조에서의 취약성:** 건물의 긴 복도, 동일한 의자가 여러 개 놓인 회의실, 또는 규칙적인 패턴을 가진 파사드와 같이 기하학적으로 대칭적이거나 반복적인 구조가 많은 환경에서 Predator는 정합에 실패할 가능성이 있다. 이러한 환경에서는 서로 다른 위치에 있는 점들이 매우 유사한 지역적 기하학 특징을 갖게 되어, 어텐션 메커니즘이 올바른 대응 관계를 찾는 데 모호성을 겪을 수 있다.4
- **계산 복잡도:** 인코더-디코더 구조, 특히 모든 점들 간의 상호작용을 계산하는 Cross-Attention 모듈은 상당한 계산량과 메모리를 요구한다. 이로 인해 실시간 처리가 필수적인 자율주행이나 로보틱스 애플리케이션에 Predator를 직접 탑재하는 데는 부담이 따를 수 있다.5



### 4.2 후속 연구 동향 및 Predator 개선 방안



Predator의 성공과 그 한계는 3D 비전 분야에서 '데이터 중심(Data-driven)' 접근법의 힘과 약점을 동시에 보여주는 중요한 사례가 되었다. Predator는 수작업 특징 설계의 한계를 넘어, 데이터로부터 저중첩이라는 특정 문제에 대한 최적의 해결책(중첩 예측)을 '학습'해내는 데 성공했다. 그러나 이 성공은 대규모의 '정답' 데이터가 있다는 가정 하에 이루어졌으며, 데이터가 없는 현실 세계에서의 취약성을 드러냈다.23 이로 인해 연구의 흐름은 '더 많은 데이터로 더 좋은 모델을 만드는' 방향에서 '적은 데이터, 혹은 데이터 없이도 일반화할 수 있는 모델을 만드는' 방향으로 이동하고 있다. Predator는 데이터 중심 접근법의 정점을 보여줌과 동시에, 그 패러다임의 한계와 새로운 패러다임의 필요성을 암시하는 중요한 변곡점에 위치한 모델로 해석될 수 있다.

- **기하학적 일관성 강화 (e.g., GeoCORNet):** Predator의 후속 연구인 GeoCORNet은 Predator가 대칭 구조에 취약하다는 한계를 명시적으로 지적한다. 이를 해결하기 위해, 단순히 점 단위의 특징 유사성뿐만 아니라, 대응점 쌍들로 구성된 삼각형이나 사각형의 '각도'나 '길이 비율'과 같은 추가적인 기하학적 불변량(geometric invariant)을 학습 과정에 명시적으로 포함시킨다. 이러한 고차원의 기하학적 일관성 제약 조건은 모호한 상황에서 올바른 대응점을 찾는 데 강력한 단서를 제공하여 정합의 강인성을 크게 향상시킨다.4
- **마스킹 및 재구성 패러다임 (e.g., MRA):** 일부 최신 연구들은 포인트클라우드 정합 문제를 자연어 처리의 BERT나 컴퓨터 비전의 MAE(Masked Autoencoders)에서 영감을 받은 '마스킹 및 재구성(Masking and Reconstruction)' 문제로 재해석한다. 즉, 한 포인트클라우드의 보이지 않는 부분을 마스크로 간주하고, 두 포인트클라우드가 올바르게 정렬되었을 때 완전한 형태의 객체를 재구성하도록 네트워크를 학습시킨다. 이 방식은 별도의 중첩 예측 모듈 없이도 네트워크가 두 포인트클라우드 간의 공통 구조를 암시적으로 학습하게 만들어, 추가적인 계산 비용 없이 저중첩 성능을 높일 수 있다.32
- **Zero-Shot 및 Foundation Model 접근:** 학습 데이터 의존성이라는 근본적인 문제를 해결하기 위해, 대규모 2D 이미지-텍스트 데이터로 사전 학습된 CLIP과 같은 Foundation Model의 방대한 지식을 3D 정합에 활용하려는 연구가 시도되고 있다. 예를 들어, 포인트클라우드를 여러 시점에서 렌더링하여 2D 이미지로 변환한 후, CLIP의 특징을 추출하여 의미론적(semantic) 대응점을 찾는 방식이다. 이는 레이블된 3D 데이터 없이도 "의자 다리는 의자 다리와 매칭되어야 한다"와 같은 상위 레벨의 이해를 바탕으로 정합을 수행할 수 있는 잠재력을 보여준다.23



### 4.3 향후 포인트클라우드 정합 기술의 발전 전망



향후 포인트클라우드 정합 기술은 순수한 기하학 정보에만 의존하는 것을 넘어, 다양한 종류의 정보를 통합하는 방향으로 발전할 것이다. 의미론적 정보(semantic information), 물리적 제약 조건(physical constraints), 그리고 시간적 일관성(temporal consistency) 등을 결합하여 더욱 강인하고 정확한 정합을 달성하는 연구가 주류를 이룰 것이다. 또한, 실시간 성능과 경량화는 여전히 중요한 연구 주제로 남을 것이며, 이를 위해 효율적인 트랜스포머 아키텍처, 지식 증류(knowledge distillation), 그리고 하드웨어 가속 기술이 활발히 연구될 것이다. 마지막으로, 강체(rigid) 객체를 넘어 사람의 움직임이나 변형 가능한 물체를 다루는 비강체(non-rigid) 정합, 그리고 여러 객체가 동시에 움직이는 복잡하고 동적인 시나리오를 처리하는 연구가 더욱 중요해질 것이다.18



## 5.  결론





### Predator 모델의 핵심 기여 요약



Predator는 3D 포인트클라우드 정합 분야, 특히 기존 방법론들이 난항을 겪었던 심각한 저중첩 문제에 대한 효과적인 해법을 제시한 선구적인 딥러닝 모델이다. 이 모델의 핵심적인 기여는 다음과 같이 요약할 수 있다.

첫째, Predator는 문제 해결의 패러다임을 전환했다. 단순히 더 나은 특징 기술자를 학습하는 데 집중하던 기존 연구의 흐름에서 벗어나, '중첩 어텐션 블록'이라는 혁신적인 모듈을 통해 두 포인트클라우드 간의 상호 컨텍스트를 조기에 교환하고, 이를 바탕으로 정합에 유용한 '중첩 영역'을 명시적으로 예측하는 새로운 접근 방식을 제시했다.6 이는 정합의 효율성과 강인성을 근본적으로 향상시키는 중요한 발상이었다.

둘째, 저중첩 환경에서의 성능을 극적으로 개선했다. 3DLoMatch와 같은 도전적인 벤치마크에서 기존 SOTA 모델들을 큰 차이로 능가함으로써, 저중첩이 더 이상 해결 불가능한 문제가 아님을 입증했다. 이는 Predator의 설계 철학이 이론적으로 타당할 뿐만 아니라, 실증적으로도 매우 효과적임을 보여준 결과이다.



### 5.1 기술적 및 산업적 함의



Predator의 성공은 학술적 성취를 넘어, 3D 비전 기술의 산업적 응용에 중요한 함의를 가진다. 이 모델은 어텐션 메커니즘과 트랜스포머 구조가 복잡한 3D 기하학 문제 해결에 매우 효과적임을 입증했으며, 이후 GeoTransformer, REGTR 등 수많은 후속 연구에 직접적인 영감을 주었다.5

특히 자율주행 분야에서 Predator와 같은 저중첩 정합 기술은 시스템의 안전성과 신뢰성을 담보하는 핵심 기반 기술이다. 자율주행 차량의 LiDAR 센서는 주행 중 발생하는 빈번한 폐색이나 개활지와 같은 희소한 환경으로 인해 항상 부분적이고 단편적인 데이터만을 획득하게 된다.34 이러한 불완전한 스캔들을 실시간으로 강인하게 정합하여 일관된 지도를 생성하고(SLAM), 이를 통해 차량의 위치를 정밀하게 추정하는 능력은 자율주행 시스템의 안전 운행에 직결된다.5

결론적으로, Predator는 저중첩이라는 특정 난제를 해결하기 위한 정교한 딥러닝 아키텍처를 제시한 학술적 성과를 넘어, 로보틱스와 자율주행 시스템이 더 복잡하고 예측 불가능한 실제 환경에서 안정적으로 작동할 수 있도록 만드는 데 기여한 핵심적인 기술적 이정표 중 하나로 평가받아야 한다.

#### 참고 자료

1. [논문 리뷰] Deep Learning-Based Point Cloud Registration: A Comprehensive Survey and Taxonomy - Moonlight, accessed August 19, 2025, https://www.themoonlight.io/ko/review/deep-learning-based-point-cloud-registration-a-comprehensive-survey-and-taxonomy
2. 체적형 객체 촬영을 위한 RGB-D 카메라 기반의 포인트 클라우드 정합 알고리즘/ 김경진, 박병서, 김동욱, 서영호 - 한국방송·미디어공학회, accessed August 19, 2025, https://www.kibme.org/resources/journal/20191015154459472.pdf
3. A review of rigid point cloud registration based on deep learning - Frontiers, accessed August 19, 2025, https://www.frontiersin.org/journals/neurorobotics/articles/10.3389/fnbot.2023.1281332/full
4. Robust Low-Overlap Point Cloud Registration via Displacement ..., accessed August 19, 2025, https://www.mdpi.com/1424-8220/25/14/4332
5. Learning Representative Features by Deep Attention Network for 3D Point Cloud Registration - PMC, accessed August 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10145325/
6. [2011.13005] Predator: Registration of 3D Point Clouds with Low ..., accessed August 19, 2025, https://ar5iv.labs.arxiv.org/html/2011.13005
7. Predator: Registration of 3D Point Clouds With Low Overlap - CVF Open Access, accessed August 19, 2025, https://openaccess.thecvf.com/content/CVPR2021/papers/Huang_Predator_Registration_of_3D_Point_Clouds_With_Low_Overlap_CVPR_2021_paper.pdf
8. Robust Low-Overlap Point Cloud Registration via Displacement-Corrected Geometric Consistency for Enhanced 3D Sensing - PubMed Central, accessed August 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12300971/
9. RANSAC vs. ICP: 두 강력한 알고리즘의 비교와 활용 - 직관적인느낌 - 티스토리, accessed August 19, 2025, https://lbj142632.tistory.com/139
10. Partial-to-Partial Point Cloud Registration by Rotation Invariant Features and Spatial Geometric Consistency - MDPI, accessed August 19, 2025, https://www.mdpi.com/2072-4292/15/12/3054
11. KISS-Matcher: Fast and Robust Point Cloud Registration Revisited - arXiv, accessed August 19, 2025, https://arxiv.org/html/2409.15615v1
12. PREDATOR: Registration of 3D Point Clouds with Low Overlap, accessed August 19, 2025, https://overlappredator.github.io/
13. [2011.13005] PREDATOR: Registration of 3D Point Clouds with Low Overlap - arXiv, accessed August 19, 2025, https://arxiv.org/abs/2011.13005
14. Network architecture of PREDATOR. Voxel-gridded point clouds P and Q... - ResearchGate, accessed August 19, 2025, https://www.researchgate.net/figure/Network-architecture-of-PREDATOR-Voxel-gridded-point-clouds-P-and-Q-are-fed-to-the_fig1_346475391
15. PREDATOR: Registration of 3D Point Clouds with Low Overlap - Supplementary material - CVF Open Access, accessed August 19, 2025, https://openaccess.thecvf.com/content/CVPR2021/supplemental/Huang_Predator_Registration_of_CVPR_2021_supplemental.pdf
16. prs-eth/OverlapPredator: [CVPR 2021, Oral] PREDATOR: Registration of 3D Point Clouds with Low Overlap. - GitHub, accessed August 19, 2025, https://github.com/prs-eth/OverlapPredator
17. DOPNet: Achieving Accurate and Efficient Point Cloud Registration Based on Deep Learning and Multi-Level Features, accessed August 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9655122/
18. DefTransNet: A Transformer-based Method for Non-Rigid Point Cloud Registration in the Simulation of Soft Tissue Deformation - arXiv, accessed August 19, 2025, https://arxiv.org/html/2502.06336v1
19. Rigid transformation - Wikipedia, accessed August 19, 2025, https://en.wikipedia.org/wiki/Rigid_transformation
20. Rigid Transformations - Department of Mathematics at UTSA, accessed August 19, 2025, https://mathresearch.utsa.edu/wiki/index.php?title=Rigid_Transformations
21. What is Loss Function? | IBM, accessed August 19, 2025, https://www.ibm.com/think/topics/loss-function
22. Global registration - Open3D primary (unknown) documentation, accessed August 19, 2025, https://www.open3d.org/docs/latest/tutorial/pipelines/global_registration.html
23. PREDATOR: Registration of 3D Point Clouds with Low Overlap - ResearchGate, accessed August 19, 2025, https://www.researchgate.net/publication/355869599_PREDATOR_Registration_of_3D_Point_Clouds_with_Low_Overlap
24. 3DMatch: Learning Local Geometric Descriptors from RGB-D Reconstructions, accessed August 19, 2025, https://3dmatch.cs.princeton.edu/
25. The KITTI Vision Benchmark Suite - Andreas Geiger, accessed August 19, 2025, https://www.cvlibs.net/datasets/kitti/
26. RRGA-Net: Robust Point Cloud Registration Based on Graph Convolutional Attention - PMC, accessed August 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10747256/
27. Point Cloud Registration on 3DMatch (at least 30% overlapped - sample 5k interest points), accessed August 19, 2025, https://paperswithcode.com/sota/point-cloud-registration-on-3dmatch-at-least-2
28. D3Feat: Joint Learning of Dense Detection and Description of 3D Local Features, accessed August 19, 2025, https://paperswithcode.com/paper/d3feat-joint-learning-of-dense-detection-and
29. PointNetLK Revisited - ResearchGate, accessed August 19, 2025, https://www.researchgate.net/publication/355863501_PointNetLK_Revisited
30. ZeroReg: Zero-Shot Point Cloud Registration with Foundation Models - arXiv, accessed August 19, 2025, https://arxiv.org/html/2312.03032v3
31. A robust inlier identification algorithm for point cloud registration via l0-minimization - NIPS, accessed August 19, 2025, https://proceedings.neurips.cc/paper_files/paper/2024/file/735c847a07bf6dd4486ca1ace242a88c-Paper-Conference.pdf
32. Rethinking Point Cloud Registration as Masking and Reconstruction - CVF Open Access, accessed August 19, 2025, https://openaccess.thecvf.com/content/ICCV2023/papers/Chen_Rethinking_Point_Cloud_Registration_as_Masking_and_Reconstruction_ICCV_2023_paper.pdf
33. [논문]포인트 클라우드 콘텐츠 해상도 향상을 위한 점진적 렌더링 방법, accessed August 19, 2025, https://scienceon.kisti.re.kr/srch/selectPORSrchArticle.do?cn=JAKO202117651617613
34. [이슈] 자율주행차의 핵심기술 '라이다' - 특허뉴스, accessed August 19, 2025, http://www.e-patentnews.com/4828
35. 자율주행의 눈이 되어주는 센서 - 테크 포커스, accessed August 19, 2025, https://techfocus.kr/ts_job/5


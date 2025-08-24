[트랜스포머 (Transformer)](./index.md)
# Prototypical Cross-Attention Network (PCAN)



현대 딥러닝 모델은 대규모 레이블 데이터를 통해 전례 없는 성공을 거두었으나, 이러한 데이터 의존성은 모델의 적용 범위를 제한하는 주요한 한계로 작용한다.1 방대한 데이터 수집 및 레이블링 비용이 막대한 분야나 희귀 사례를 다루어야 하는 응용에서는 기존의 학습 패러다임이 비효율적이거나 불가능하다. 이러한 배경에서 데이터 효율적 학습(Data-Efficient Learning)의 중요성이 대두되었으며, 이는 두 가지 주요 연구 흐름으로 구체화되었다. 첫째는 소수의 샘플만으로 새로운 개념을 학습하는 퓨샷 학습(Few-Shot Learning, FSL)이다.3 FSL은 인간이 단 몇 개의 예시만으로 새로운 사물을 인지하는 능력에 착안하여, 제한된 데이터 환경에서도 높은 일반화 성능을 달성하는 것을 목표로 한다. 둘째는 비디오와 같은 연속적 데이터에서 시공간적 맥락을 이해하고 객체를 지속적으로 추적 및 분할하는 다중 객체 추적 및 분할(Multiple Object Tracking and Segmentation, MOTS)이다.9 MOTS는 시시각각 변화하는 객체의 외형과 위치를 정확히 파악하기 위해 과거의 정보를 효과적으로 활용해야 하는 도전 과제를 안고 있다.


이러한 데이터 효율적 학습의 문제를 해결하기 위해 다양한 방법론이 제안되었으며, 그중에서도 프로토타입 기반 거리 학습(Prototype-based Metric Learning)과 어텐션 메커니즘(Attention Mechanism)은 가장 중요한 두 축을 형성한다. 프로토타입 네트워크(Prototypical Networks)는 FSL 분야에서 거리 기반 학습의 새로운 패러다임을 제시한 대표적인 모델이다.10 이 모델은 각 클래스를 대표하는 하나의 '프로토타입'을 임베딩 공간상에 정의하고, 새로운 데이터(쿼리)를 가장 가까운 프로토타입에 할당하는 직관적이고 효과적인 방식을 제안했다. 한편, 어텐션 메커니즘, 특히 크로스 어텐션(Cross-Attention)은 두 개 이상의 서로 다른 데이터 소스 간의 관계를 모델링하는 데 강력한 성능을 보인다.11 FSL에서는 주어진 소수의 샘플(서포트 셋)을 기준으로 새로운 샘플(쿼리 셋)의 어느 부분에 집중해야 할지를 동적으로 결정하는 데 활용된다.


본 보고서는 사용자의 질의에 명시된 Prototypical Cross-Attention Network (PCAN)를 심층적으로 분석하는 것을 목표로 한다. PCAN은 공식적으로 MOTS 태스크를 해결하기 위해 제안된 모델이지만 9, 그 이름에서 알 수 있듯이 프로토타입과 크로스 어텐션이라는 FSL의 핵심 개념을 깊이 내재하고 있다. 이러한 개념적 연결고리는 MOTS와 FSL이 근본적으로 유사한 문제를 다루고 있음을 시사한다. FSL은 소수의 정적 이미지 집합(서포트 셋)에서 강인한 표현을 학습해야 하는 반면, MOTS는 변화하는 시계열 프레임 집합(과거 프레임)에서 일관된 표현을 학습해야 한다. 즉, 두 문제 모두 '관련 있지만 다양한 데이터 포인트 집합으로부터 일반화 가능한 표현을 어떻게 추출할 것인가'라는 공통된 질문에 답해야 한다.

따라서 본 보고서는 PCAN의 원리와 구조를 단순히 설명하는 것을 넘어, 이를 FSL 분야의 주요 모델들과의 비교 분석을 통해 그 구조적 독창성과 한계를 규명한다. 이를 통해 MOTS를 위해 고안된 계산 효율적 시공간 정보 압축 기법이 FSL 문제에 어떻게 적용될 수 있는지 탐구하고, 최종적으로 PCAN의 핵심 아이디어를 FSL 태스크로 확장하는 구체적인 방안을 제시하며 미래 연구 방향을 조망한다.




Snell 등이 2017년에 제안한 프로토타입 네트워크는 데이터가 극히 제한적인 FSL 환경에서는 복잡한 메타러닝 구조보다 단순한 귀납적 편향이 더 효과적이라는 가정에서 출발한다.10 이 모델의 핵심 아이디어는 임베딩 공간 내에 각 클래스를 대표하는 단일 '프로토타입'이 존재하며, 분류는 이 프로토타입과의 거리를 측정함으로써 수행될 수 있다는 것이다. 이는 데이터가 부족할수록 모델이 가정하는 구조가 단순해야 과적합을 피하고 일반화 성능을 높일 수 있다는 철학에 기반한다.


프로토타입 네트워크에서 클래스 $k$의 프로토타입 $c_k$는 해당 클래스에 속한 서포트 샘플들의 임베딩 벡터 평균으로 계산된다.10 임베딩 함수를 $f_\phi$라 하고, 클래스 $k$의 서포트 셋을 $S_k$라고 할 때, 프로토타입은 다음과 같이 정의된다.

$$
c_k = \frac{1}{|S_k|} \sum_{(x_i, y_i) \in S_k} f_\phi(x_i)
$$
이 계산 방식은 매우 직관적이며, 임베딩 공간상에서 해당 클래스 샘플들의 중심점을 찾는 것과 같다. 이 프로토타입은 해당 클래스의 가장 대표적인 특징을 압축적으로 담고 있는 벡터로 기능한다.


새로운 쿼리 샘플 $x$가 주어지면, 먼저 임베딩 함수 $f_\phi$를 통해 쿼리 샘플을 임베딩 공간으로 매핑한다. 그 후, 임베딩된 쿼리 벡터 $f_\phi(x)$와 각 클래스의 프로토타입 $c_k$ 간의 거리(distance)를 계산한다. 프로토타입 네트워크는 일반적으로 제곱 유클리드 거리(squared Euclidean distance)를 사용하며, 이 거리에 기반한 소프트맥스 함수를 통해 쿼리 샘플이 각 클래스에 속할 확률 분포를 계산한다.10

$$
p_\phi(y=k|x) = \frac{\exp(-d(f_\phi(x), c_k))}{\sum_{k'} \exp(-d(f_\phi(x), c_{k'}))}
$$
학습은 실제 레이블에 대한 음의 로그 확률(negative log-probability)을 최소화하는 방향으로 진행되며, 이를 통해 동일 클래스 내 샘플들은 임베딩 공간에서 해당 클래스의 프로토타입 주변으로 모이고, 다른 클래스의 프로토타입과는 멀어지도록 임베딩 함수 $f_\phi$가 학습된다.


프로토타입 네트워크의 성공에 기여한 또 다른 핵심 요소는 에피소딕 학습 방식이다. 이는 학습 과정에서 실제 테스트 환경(N-way K-shot)을 모방하는 작은 단위의 '에피소드'를 무작위로 구성하여 모델을 훈련시키는 방식이다.10 각 에피소드는 N개의 클래스와 각 클래스당 K개의 서포트 샘플, 그리고 별도의 쿼리 샘플들로 구성된다. 이러한 방식은 학습과 테스트 환경의 불일치를 줄여 모델의 일반화 성능을 크게 향상시킨다.



크로스 어텐션은 두 개의 다른 입력 시퀀스(또는 특징 맵) 간의 상호작용을 모델링하기 위한 메커니즘이다. 트랜스포머 아키텍처에서 처음 제안된 스케일드 닷-프로덕트 어텐션(Scaled Dot-Product Attention)을 기반으로 하며, 일반적으로 하나의 입력에서 Query(Q) 벡터를, 다른 입력에서 Key(K)와 Value(V) 벡터를 생성한다. 그 후, 모든 Q 벡터와 모든 K 벡터 간의 내적(dot product)을 계산하여 유사도 점수(similarity score)를 얻고, 이를 스케일링한 후 소프트맥스 함수를 적용하여 어텐션 가중치(attention weights)를 구한다. 최종적으로 이 가중치를 V 벡터에 곱하여 가중합을 계산함으로써, Q의 관점에서 K/V 입력의 중요한 정보를 선택적으로 추출한 결과 벡터를 얻는다.


PCAN 논문에서 지적하는 표준 어텐션 연산은 고해상도 이미지나 긴 비디오 시퀀스와 같은 밀집된(dense) 데이터에 적용될 때 심각한 계산적 한계를 드러낸다.9 예를 들어, 현재 프레임의 픽셀(쿼리)이 과거 모든 프레임의 모든 픽셀(메모리)과 상호작용해야 하는 경우, 그 연산은 다음과 같이 표현될 수 있다.

$$
y_i = \frac{1}{Z_i} \sum_{j=1}^{H \times W \times T} \exp(k_Q(i) \cdot k_M(j)) v_M(j)
$$
여기서 $i$는 쿼리 위치의 인덱스이고, $j$는 $H \times W \times T$ 크기의 시공간 메모리 내 위치의 인덱스이다. 이 연산은 모든 쿼리-키 쌍에 대해 유사도를 계산해야 하므로, 특징 맵의 공간적 크기($HW$)에 대해 $O((HW)^2)$의 계산 및 메모리 복잡도를 가진다. 이러한 이차적 복잡도(quadratic complexity)는 고해상도 비디오 처리를 현실적으로 불가능하게 만드는 근본적인 병목 현상으로 작용한다.9 PCAN은 바로 이 문제를 해결하기 위한 대안적 구조를 제안한다.



PCAN은 비디오 내 다중 객체를 온라인으로 추적하고 동시에 분할하는 MOTS 태스크를 위해 특별히 설계되었다. 기존 MOTS 접근법들은 주로 시간적 차원을 객체 간의 연관 문제(association problem)를 해결하는 데 국한하여 사용하고, 객체의 분할 마스크 자체는 단일 프레임의 정보에만 의존하는 경향이 있었다.9 이는 과거 프레임에 존재하는 풍부한 시공간적 정보(spatio-temporal information)를 충분히 활용하지 못하는 한계를 낳는다. PCAN의 핵심 설계 목표는 이러한 시공간적 정보를 효율적으로 압축하고 검색하여, 객체의 외형 변화나 가려짐(occlusion)에 강인한 추적 및 분할 성능을 달성하는 것이다.


PCAN의 핵심은 프로토타입 크로스 어텐션 모듈(Prototypical Cross-Attention Module, PCAM)에 있다. PCAM은 표준 크로스 어텐션의 계산적 한계를 극복하고 시공간 정보의 강인한 표현을 학습하기 위한 독창적인 해결책을 제시한다.


PCAM은 과거 프레임들로부터 추출된 방대한 고해상도 특징 벡터 집합(space-time memory)을 직접 사용하는 대신, 이를 먼저 소수의 '프로토타입'으로 압축(distill)한다.9 이는 표준 어텐션의 근본적인 문제, 즉 모든 쿼리가 모든 키와 상호작용해야 하는 데서 발생하는 이차적 복잡도를 해결하기 위한 전략이다. 프로토타입 네트워크가 클래스 내 샘플들을 하나의 대표 벡터로 요약하는 것처럼, PCAM은 방대한 시공간 메모리를 몇 개의 대표적인 특징 벡터로 요약한다. 이 접근법은 프로토타입 생성이 어텐션 메커니즘을 고차원 데이터에 적용 가능하게 만드는 강력한 전처리 단계로 기능할 수 있음을 보여준다. 즉, 프로토타입화는 단순히 분류를 위한 요약이 아니라, 어텐션을 위한 효율적인 키-밸류 메모리를 생성하는 구조적 장치로 활용된다.


메모리 압축 과정은 기댓값 최대화(Expectation-Maximization, EM) 알고리즘 기반의 클러스터링을 통해 수행된다. EM 알고리즘은 데이터의 분포를 여러 개의 가우시안 혼합 모델(Gaussian Mixture Model)로 근사하는데, PCAM에서는 각 가우시안 구성요소(Gaussian Component)가 하나의 프로토타입에 해당한다.9 이 과정은 두 가지 중요한 이점을 제공한다. 첫째, 비디오 스트림에 존재하는 노이즈와 중복성을 효과적으로 제거한다. 조명 변화, 모션 블러, 일시적인 가려짐 등 개별 프레임에 나타나는 불안정한 특징들은 클러스터링 과정을 통해 평균화되어 걸러진다. 결과적으로 생성된 프로토타입은 시간에 따라 안정적이고 일관된 객체의 핵심 외형을 나타내는, 노이즈가 제거된(denoised) 강인한 표현이 된다.9 둘째, 이렇게 생성된 프로토타입은 과거 시각적 특징들에 대한 풍부하면서도 일반화 가능하고 컴팩트한 표현을 제공한다.


표준 어텐션이 $H \times W \times T$ 크기의 전체 시공간 메모리에 대해 어텐션을 적용해야 했던 것과 달리, PCAN은 압축된 $N$개의 프로토타입($N \ll HWT$)에 대해서만 어텐션을 수행한다.9 이로 인해 어텐션 연산의 복잡도는 기존의 $O((HW)^2)$에서 $O(HWN)$ 수준으로 크게 감소하여, 고해상도 비디오 처리에 현실적으로 적용 가능한 수준의 계산 효율성을 확보하게 된다.



PCAN은 PCAM을 단순히 한 번만 사용하는 것이 아니라, 네트워크의 서로 다른 두 단계에 계층적으로 통합한다. 첫째는 프레임 레벨(frame-level)에서 전역적인 장면의 맥락을 이해하기 위해, 둘째는 인스턴스 레벨(instance-level)에서 특정 객체의 세부적인 특징을 포착하기 위해서다.9 이러한 계층적 구조는 모델이 전역적 및 지역적 시공간 정보를 모두 효과적으로 활용할 수 있도록 돕는다.


객체의 외형이 시간에 따라 변하거나 배경과 유사해지는 문제에 대응하기 위해, PCAN은 각 객체 인스턴스를 단일 프로토타입이 아닌, 대조적인 '전경(foreground)' 프로토타입 집합과 '배경(background)' 프로토타입 집합으로 나누어 표현한다.9 이는 학습 과정에서 모델이 객체 자체의 고유한 특징과 주변 배경의 특징을 명시적으로 구분하도록 강제하는 역할을 한다. 이 프로토타입들은 고정되어 있지 않고, 비디오가 진행됨에 따라 온라인 방식으로 계속해서 전파되고 갱신되어 변화하는 객체의 외형에 적응한다. 이처럼 명시적으로 전경과 배경을 분리하는 표현 학습 전략은 어텐션 메커니즘이 배경의 방해 요소에 현혹되지 않고 목표 객체에 집중하도록 유도하는 강력한 장치로 기능한다.




Hou 등이 2019년에 제안한 Cross Attention Network (CAN)는 FSL에서 서포트 셋과 쿼리 셋의 특징을 독립적으로 추출하는 기존 방식의 한계를 극복하고자 했다.11 CAN의 핵심 아이디어는 서포트 셋 정보(클래스 특징)를 이용해 쿼리 이미지 내에서 해당 클래스와 의미론적으로 관련된 영역을 동적으로 찾아내고, 그 영역의 특징을 강조하여 보다 판별력 있는 표현을 학습하는 것이다. 이를 위해 클래스 특징과 쿼리 특징 간의 상호작용을 모델링하는 '크로스 어텐션 모듈(Cross Attention Module)'을 도입했다.12


CAN의 전체 아키텍처 및 데이터 흐름은 다음과 같은 단계로 구성된다 13:

1. **특징 추출 (Feature Extraction)**: 동일한 CNN 백본(e.g., ResNet-12)을 사용하여 서포트 이미지들과 쿼리 이미지로부터 각각 서포트 특징 맵(Support Feature)과 쿼리 특징 맵(Query Feature)을 추출한다.
2. **클래스 특징 생성 (Class Feature Generation)**: 동일한 클래스에 속하는 서포트 특징 맵들을 채널별로 평균(averaging)하여 해당 클래스를 대표하는 클래스 특징 맵(Class Feature)을 생성한다.
3. **크로스 어텐션 모듈 (Cross Attention Module)**: 각 클래스 특징 맵과 쿼리 특징 맵 쌍을 입력으로 받는다. 두 특징 맵 간의 유사도를 계산하여 공간적 어텐션 맵(Attention Map)을 생성한다. 이 어텐션 맵은 쿼리 특징 맵의 각 위치가 클래스 특징과 얼마나 관련이 있는지를 나타내는 가중치로 작용한다. 이 가중치를 원래의 쿼리 특징 맵에 적용하여 관련 영역은 강조하고 관련 없는 영역은 억제한 '가중 쿼리 특징(Weighted Query Feature)'을 생성한다.
4. **분류 (Classification)**: 최종적으로 가중 쿼리 특징과 각 클래스 특징 간의 유사도(e.g., 코사인 유사도)를 계산하고, 이를 기반으로 쿼리 이미지의 클래스를 예측한다.


CAN과 PCAN은 '프로토타입'과 '크로스 어텐션'이라는 키워드를 공유하지만, 그 철학과 구현 방식에는 근본적인 차이가 존재한다.

- **어텐션의 대상 및 차원**: CAN의 어텐션은 특징 맵의 **공간적 차원(spatial dimension)**에서 직접적으로 작동한다. 즉, 클래스 특징의 특정 위치와 쿼리 특징의 특정 위치 간의 유사도를 계산하여 '어디를 보아야 하는가'에 대한 공간적 정렬(spatial alignment)을 학습한다. 반면, PCAN의 어텐션은 고차원의 특징 공간 전체를 저차원의 **개념적 프로토타입 집합**으로 먼저 압축한 뒤, 쿼리가 이 '압축된 개념'들에 대해 어텐션을 수행한다. 이는 공간적 위치가 아닌, 의미론적 클러스터에 대한 어텐션이다.
- **핵심 목표**: CAN의 주된 목표는 서포트 정보를 가이드 삼아 쿼리 이미지에서 판별력 있는 객체 영역을 정확히 **찾아내는 것(localization)**이다. PCAN의 주된 목표는 비디오와 같은 방대한 데이터로부터 어텐션 연산을 **확장 가능하게(scalable)** 만들고, 시간에 따라 변화하는 객체 외형에 **강인한(robust)** 표현을 학습하는 것이다.



Xu 등이 2023년에 FSS(Few-Shot Segmentation) 분야에서 제안한 SCCAN은 CAN과 같은 기존 크로스 어텐션 방식이 가진 두 가지 고질적인 문제를 명확히 지적했다.14

- **배경 불일치 (BG Mismatch)**: FSS 태스크에서 서포트 이미지는 보통 목표 객체(전경, FG) 영역만 제공된다. 쿼리 이미지의 배경(BG) 영역 특징은 서포트 이미지의 전경 특징과 매칭될 대상이 없으므로, 관련 없는 전경 특징과 강제로 융합되면서 특징 표현이 왜곡되는 문제가 발생한다.
- **전경-배경 얽힘 (FG-BG Entanglement)**: 쿼리 이미지의 전경과 배경 특징이 모두 서포트 이미지의 동일한 전경 특징과 융합된다. 이 과정에서 쿼리 전경과 배경의 특징이 서로 뒤섞이게 되어, 최종적으로 두 영역을 명확히 분리하여 분할하는 것을 어렵게 만든다.


SCCAN은 이러한 문제를 해결하기 위해 어텐션 연산 시 Key(K)와 Value(V)를 구성하는 방식을 변경하는 **구조적(architectural)** 해결책을 제시한다. 즉, K와 V를 서포트 특징만으로 구성하는 대신, '쿼리 자신의 특징'과 '서포트의 특징'을 함께 그룹화한다.14 이로써 어텐션 모듈은 셀프 어텐션(Self-Attention, 쿼리-쿼리 상호작용)과 크로스 어텐션(Cross-Attention, 쿼리-서포트 상호작용)을 동시에 수행하게 된다.

- **BG Mismatch 해결**: 쿼리 배경 특징은 함께 그룹화된 다른 쿼리 배경 특징들과 셀프 어텐션을 통해 매칭되므로, 더 이상 서포트의 관련 없는 전경 특징과 강제로 융합될 필요가 없다.
- **FG-BG Entanglement 해결**: 쿼리 전경은 셀프 어텐션을 통해 쿼리 내 다른 전경 영역과, 크로스 어텐션을 통해 서포트 전경 영역과 모두 상호작용하며 강화된다. 반면 쿼리 배경은 주로 쿼리 내 배경 정보와 상호작용하므로, 전경과 배경의 특징이 분리되어 얽힘 문제가 완화된다.

이러한 SCCAN의 구조적 접근법은 PCAN이 제시하는 **표현적(representational)** 해결책과 흥미로운 대조를 이룬다. PCAN은 어텐션 메커니즘 자체를 수정하는 대신, 학습을 통해 명시적으로 분리된 전경과 배경 프로토타입을 생성하여 어텐션 모듈에 더 정제된 입력을 제공한다. SCCAN의 방식은 추론 시에 동적으로 자신을 보정하는 유연성을 가지는 반면, PCAN의 방식은 학습 데이터로부터 전경과 배경에 대한 일반화된 모델을 안정적으로 학습할 수 있다면 더욱 강인한 성능을 보일 수 있다. 특히 객체의 배경이 계속해서 변하는 MOTS 환경에서는 특정 배경에 의존하지 않는, 객체 자체의 안정적인 표현을 학습하는 PCAN의 방식이 더 효과적일 수 있다.


프로토타입과 어텐션을 결합하려는 시도는 FSL 분야에서 다양하게 이루어지고 있다.

- **ProFusion**: 시각 인코더와 텍스트 인코더를 대칭적으로 사용하여 이미지 프로토타입, 텍스트 프로토타입, 그리고 두 정보를 융합한 멀티모달 프로토타입을 함께 생성한다. 이는 다중 모달리티 정보를 활용하여 클래스 표현력을 극대화하려는 접근법이다.3
- **Semantic Cross Attention**: 클래스 레이블의 텍스트 정보(e.g., 워드 임베딩)를 보조적인 정보로 활용한다. 시각 특징과 텍스트의 의미론적 특징 간에 크로스 어텐션을 수행하여, 시각적으로는 유사하지만 의미론적으로 다른 클래스를 구분하는 능력을 향상시킨다.15



제안된 모델들의 성능을 객관적으로 평가하기 위해, 각 태스크에 맞는 표준 벤치마크 데이터셋과 평가 지표가 사용된다.

- **MOTS**: PCAN은 비디오 분할 및 추적 성능을 평가하기 위해 **Youtube-VIS**와 **BDD100K** 데이터셋에서 평가되었다. 이 데이터셋들은 다양한 객체 클래스와 복잡한 실제 주행 환경을 포함하고 있어 모델의 강인성을 측정하는 데 적합하다. PCAN은 이 데이터셋들에서 당시의 SOTA(State-of-the-Art) 모델들을 능가하는 성능을 보였다고 보고되었다.9
- **FSL**: CAN, 프로토타입 네트워크 등 FSL 모델들은 주로 이미지 분류 성능을 평가하기 위해 **miniImageNet**, **tieredImageNet**, **CUB-200-2011** 등의 데이터셋을 사용한다. 평가는 일반적으로 N개의 클래스에서 클래스당 K개의 샘플을 사용하는 **'N-way K-shot'** 설정 하에 분류 정확도(%)를 측정하는 방식으로 이루어진다.16


다음 표는 FSL 벤치마크에서 주요 모델들의 성능을 종합적으로 비교한 것이다. 이를 통해 각 아키텍처의 장단점과 특정 조건 하에서의 성능을 실증적으로 분석할 수 있다.

**Table 1: 주요 모델 성능 비교 (Performance Comparison of Key Models)**

| 모델 (Model)     | 백본 (Backbone) | 데이터셋 (Dataset) | 5-way 1-shot  | 5-way 5-shot  | 주요 특징 (Key Feature)       | 출처 (Source) |      |      |          |      |
| ---------------- | --------------- | ------------------ | ------------- | ------------- | ----------------------------- | ------------- | ---- | ---- | -------- | ---- |
| Prototypical Net | ResNet-12       | miniImageNet       | 66.09 ± 0.92% | 82.50 ± 0.58% | 평균 기반 프로토타입          | 20            |      |      |          |      |
| Prototypical Net | -               | tieredImageNet     | 66.25 ± 0.34% | 80.11 ± 0.91% | 평균 기반 프로토타입          | 16            |      |      |          |      |
| CAN              | ResNet-12       | miniImageNet       | 63.85 ± 0.48% | 79.44 ± 0.34% | 직접 공간 크로스 어텐션       | 19            |      |      |          |      |
| CAN+T            | ResNet-12       | miniImageNet       | 67.19 ± 0.55% | 80.64 ± 0.35% | Transductive Inference 추가   | 19            |      |      |          |      |
| CAN              | ResNet-12       | tieredImageNet     | 69.89 ± 0.51% | 84.23 ± 0.37% | 직접 공간 크로스 어텐션       | 19            |      |      |          |      |
| CAN+T            | ResNet-12       | tieredImageNet     | 73.21 ± 0.58% | 84.93 ± 0.38% | Transductive Inference 추가   | 19            |      |      |          |      |
| PCAN             | -               | Youtube-VIS        | -             | -             | 프로토타입 기반 시공간 어텐션 | 9             |      |      |          |      |
| PCAN             | -               | BDD100K            | -             | -             | 프로토타입 기반 시공간 어텐션 | 9             |      |      |          |      |
| *기타 SOTA 모델* | *다양함*        | *다양함*           |               | 16            |                               |               | 16   |      | *다양함* | 16   |

표 분석에 따르면, 몇 가지 중요한 경향을 확인할 수 있다. 첫째, 단순한 프로토타입 네트워크는 여전히 강력한 베이스라인 성능을 보여준다. 둘째, CAN은 기본적으로 프로토타입 네트워크보다 다소 낮은 성능을 보이지만, 레이블 없는 쿼리 셋을 활용하는 전이 추론(Transductive Inference)을 추가했을 때(CAN+T) 성능이 크게 향상된다. 이는 서포트-쿼리 간 상호작용이 추가적인 데이터 활용 전략과 결합될 때 시너지를 낼 수 있음을 시사한다.19 셋째, 데이터셋의 난이도(tieredImageNet > miniImageNet)에 따라 모델 간 성능 격차가 달라지며, 더 복잡한 데이터셋에서 크로스 어텐션의 효과가 더 두드러지는 경향을 보인다.


정량적 수치 외에도 모델의 작동 방식을 이해하기 위한 정성적 분석이 중요하다. CAN과 같은 공간적 어텐션 모델은 어텐션 맵을 시각화함으로써 모델이 쿼리 이미지의 어느 부분에 집중하여 결정을 내리는지 직관적으로 파악할 수 있다.22 이는 모델의 해석 가능성을 높여주지만, 동시에 한계점을 드러내기도 한다. 예를 들어, 크로스 어텐션 기반 방법론들은 목표 객체와 유사한 질감이나 색상을 가진 배경 영역에 의해 쉽게 혼동될 수 있다.18 즉, 의미론적 이해 없이 저수준의 시각적 유사성에만 의존할 경우 잘못된 영역에 집중할 위험이 있다.

PCAN의 대조적 전경/배경 프로토타입이나 SCCAN의 자가 보정 메커니즘은 이러한 문제를 완화하려는 시도이지만, 완벽한 해결책은 아니다. 특히 전경과 배경의 구분이 모호하거나, 서포트 셋에 표현되지 않은 극단적인 시점이나 조명 변화가 쿼리 셋에 나타날 경우 여전히 성능 저하가 발생할 수 있다.



PCAN의 가장 중요한 학술적 기여는 표준 크로스 어텐션이 가진 계산적 병목 현상을 '프로토타입 기반 메모리 압축'이라는 독창적인 아이디어로 해결한 점에 있다. 이는 비디오와 같이 밀집되고 고차원적인 데이터 스트림에 어텐션 메커니즘을 확장 가능하게 적용할 수 있는 새로운 경로를 제시했다. 방대한 시공간 정보를 EM 클러스터링을 통해 소수의 의미론적 프로토타입으로 요약하고, 이 요약된 정보에 대해 어텐션을 수행하는 방식은 효율성과 성능을 동시에 달성하는 효과적인 전략임을 입증했다. 또한, EM 클러스터링을 통한 암묵적인 노이즈 제거 효과와 대조적 전경/배경 프로토타입을 통한 명시적인 표현 분리 학습은 MOTS 분야에서 객체 추적의 강인성을 높이는 중요한 진전으로 평가할 수 있다.9


본 보고서의 분석을 통해 드러난 PCAN의 핵심 원리는 FSL 태스크에 직접적으로 적용되어 기존 방법론을 개선할 잠재력을 가진다. 특히, 서포트 셋의 표현 방식을 고도화하는 데 중요한 단초를 제공한다.

기존 프로토타입 네트워크는 서포트 셋의 단순 평균을 프로토타입으로 사용한다.10 이 방식은 서포트 셋 내의 모든 샘플이 동등하게 중요하며 단일 군집을 형성한다고 가정하기 때문에, 클래스 내 분산(intra-class variance)이 크거나 아웃라이어가 포함된 경우 취약하다. 예를 들어, '개' 클래스의 서포트 셋에 '리트리버', '치와와', '불독' 이미지가 섞여 있을 때, 이들의 단순 평균은 어떤 품종도 제대로 대표하지 못하는 모호한 프로토타입을 생성할 수 있다.

이 문제를 해결하기 위해, PCAN의 PCAM 모듈을 FSL에 적용하는 **'프로토-클러스터 네트워크(Proto-Cluster Network)'**를 다음과 같이 구상할 수 있다.

1. **특징 추출**: N-way K-shot 태스크가 주어지면, 모든 $N \times K$개의 서포트 이미지를 백본 CNN을 통해 임베딩한다.
2. **프로토타입 클러스터링**: 각 클래스에 대해, K개의 서포트 특징 벡터들을 단순 평균하는 대신, PCAM의 EM 클러스터링을 적용한다. 이를 통해 각 클래스는 단일 프로토타입이 아닌, 하나 이상의 가우시안 프로토타입(클러스터) 집합으로 표현된다. K가 클수록 이 방식의 효과는 극대화된다.
3. **쿼리 임베딩**: 쿼리 이미지를 동일한 백본을 통해 임베딩한다.
4. **거리 계산 및 분류**: 쿼리 임베딩과 모든 클래스의 모든 프로토타입들 간의 거리를 계산하고, 가장 가까운 프로토타입(또는 프로토타입 집합)에 기반하여 클래스를 분류한다.

이러한 접근법은 서포트 셋 내에 존재하는 다양한 하위 군집이나 스타일을 명시적으로 모델링함으로써, 클래스 내 분산이 큰 현실적인 FSL 문제에서 표준 프로토타입 네트워크보다 훨씬 강인하고 정교한 분류 성능을 달성할 것으로 기대된다. 이는 PCAN의 핵심 기여인 '강인한 압축 표현 학습'을 시계열 영역에서 집합 기반 영역으로 성공적으로 이식하는 것이다.


PCAN과 관련 연구들의 분석을 바탕으로 다음과 같은 미래 연구 방향을 제시할 수 있다.

- **하이브리드 어텐션 아키텍처**: PCAN의 계산 효율적인 프로토타입 기반 어텐션과 SCCAN의 동적인 배경 보정 메커니즘을 결합하는 하이브리드 모델을 개발할 수 있다. 예를 들어, PCAN 방식으로 생성된 전경/배경 프로토타입을 SCCAN의 K/V 그룹화에 통합함으로써, 학습된 표현의 강인성과 추론 시의 유연성을 모두 갖춘 새로운 어텐션 모델을 설계할 수 있다.
- **프로토타입 생성 기법 고도화**: EM 알고리즘 외에도, 자기 지도 학습(Self-Supervised Learning)이나 대조 학습(Contrastive Learning)을 활용하여 더욱 의미론적으로 풍부하고 분리도 높은 프로토타입을 생성하는 연구가 필요하다. 이는 레이블이 없는 대규모 데이터셋을 활용하여 프로토타입 생성기의 사전 학습을 수행함으로써 달성될 수 있다.
- **다중 모달리티로의 확장**: ProFusion 3이나 Semantic Cross Attention 15의 사례처럼, PCAN의 프로토타입 생성 과정에 텍스트나 오디오와 같은 다른 모달리티 정보를 통합하는 연구를 진행할 수 있다. 예를 들어, 객체의 시각적 특징 클러스터링에 해당 객체의 텍스트 설명 임베딩을 조건부 정보로 활용함으로써, 시각 정보만으로는 구분이 어려운 미세한 차이를 포착하는 능력을 향상시킬 수 있다. 이는 특히 세밀한 분류(Fine-grained Classification)가 요구되는 FSL 태스크에서 큰 잠재력을 가질 것이다.


1. Partner-Assisted Learning for Few-Shot Image Classification - CVF Open Access, 8월 16, 2025에 액세스, https://openaccess.thecvf.com/content/ICCV2021/papers/Ma_Partner-Assisted_Learning_for_Few-Shot_Image_Classification_ICCV_2021_paper.pdf
2. Dense mutual information maximization for few-shot classification - SPIE Digital Library, 8월 16, 2025에 액세스, [https://www.spiedigitallibrary.org/proceedings/Download?urlId=10.1117%2F12.3033761&downloadType=proceedings%20article&isResultClick=True](https://www.spiedigitallibrary.org/proceedings/Download?urlId=10.1117/12.3033761&downloadType=proceedings+article&isResultClick=True)
3. ProFusion: Multimodal Prototypical Networks for Few-Shot Learning with Feature Fusion, 8월 16, 2025에 액세스, https://www.mdpi.com/2073-8994/17/5/796
4. Few-Shot Prompting, 8월 16, 2025에 액세스, https://www.promptingguide.ai/techniques/fewshot
5. Language Models are Few-Shot Learners - NIPS, 8월 16, 2025에 액세스, https://proceedings.neurips.cc/paper/2020/file/1457c0d6bfcb4967418bfb8ac142f64a-Paper.pdf
6. [2005.14165] Language Models are Few-Shot Learners - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/2005.14165
7. Efficient Training of Language Models using Few-Shot Learning, 8월 16, 2025에 액세스, https://proceedings.mlr.press/v202/j-reddi23a.html
8. CLIP Models are Few-Shot Learners: Empirical Studies on VQA and Visual Entailment - ACL Anthology, 8월 16, 2025에 액세스, https://aclanthology.org/2022.acl-long.421/
9. Prototypical Cross-Attention Networks for Multiple Object Tracking and Segmentation - NIPS, 8월 16, 2025에 액세스, https://papers.nips.cc/paper/2021/file/093f65e080a295f8076b1c5722a46aa2-Paper.pdf
10. Prototypical Networks for Few-shot Learning, 8월 16, 2025에 액세스, https://arxiv.org/abs/1703.05175
11. [1910.07677] Cross Attention Network for Few-shot Classification - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/1910.07677
12. blue-blue272/fewshot-CAN: Code of Cross Attention ... - GitHub, 8월 16, 2025에 액세스, https://github.com/blue-blue272/fewshot-CAN
13. 1월 1, 1970에 액세스, https://github.com/blue-blue272/fewshot-CAN/raw/master/algorithm.png
14. Self-Calibrated Cross Attention Network for Few ... - CVF Open Access, 8월 16, 2025에 액세스, https://openaccess.thecvf.com/content/ICCV2023/papers/Xu_Self-Calibrated_Cross_Attention_Network_for_Few-Shot_Segmentation_ICCV_2023_paper.pdf
15. [2210.06311] Semantic Cross Attention for Few-shot Learning - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/abs/2210.06311
16. PrototypeFormer: Learning to Explore Prototype Relationships for Few-shot Image Classification - arXiv, 8월 16, 2025에 액세스, https://arxiv.org/html/2310.03517v2
17. FSCA: Few-Shot Learning via Embedding Adaptation with Corner Multi-Head Attention, 8월 16, 2025에 액세스, https://www.mdpi.com/2079-9292/14/1/130
18. SpatialFormer: Semantic and Target Aware Attentions for Few-Shot Learning, 8월 16, 2025에 액세스, https://ojs.aaai.org/index.php/AAAI/article/view/26016/25788
19. cross-modal knowledge enhancement mechanism for few-shot learning - OpenReview, 8월 16, 2025에 액세스, https://openreview.net/forum?id=DRc-o6DUGVf
20. CAD: Co-Adapting Discriminative Features for Improved Few-Shot Classification - CVF Open Access, 8월 16, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2022/papers/Chikontwe_CAD_Co-Adapting_Discriminative_Features_for_Improved_Few-Shot_Classification_CVPR_2022_paper.pdf
21. Generating Representative Samples for Few-Shot Classification - CVF Open Access, 8월 16, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2022/papers/Xu_Generating_Representative_Samples_for_Few-Shot_Classification_CVPR_2022_paper.pdf
22. Focus Your Attention when Few-Shot Classification | OpenReview, 8월 16, 2025에 액세스, https://openreview.net/forum?id=uFlE0qgtRO


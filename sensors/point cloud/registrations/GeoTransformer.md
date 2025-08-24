# GeoTransformer
[포인트 클라우드 정합](./index.md)



3차원 포인트 클라우드 정합(Point Cloud Registration)은 서로 다른 시점이나 센서로부터 획득된 두 개 이상의 3D 포인트 클라우드 데이터를 하나의 일관된 좌표계로 정렬하는 근본적인 과정이다.1 이 기술은 3D 데이터를 활용하는 수많은 상위 레벨 태스크, 예를 들어 객체 인식, 장면 이해, 환경 모델링 등을 가능하게 하는 기반 기술로서 학술 및 산업계 전반에 걸쳐 지대한 중요성을 가진다. 자율주행 자동차의 LiDAR 센서가 생성하는 데이터를 이용한 동시적 위치 추정 및 지도 작성(SLAM), 로봇이 주변 환경을 인식하고 상호작용하는 로보틱스, 여러 스캔 데이터를 병합하여 완전한 모델을 만드는 3D 재구성, 그리고 가상 객체를 현실 세계에 정밀하게 배치하는 증강 현실(AR) 등은 포인트 클라우드 정합 기술에 깊이 의존하는 대표적인 응용 분야다.2

그러나 포인트 클라우드 정합은 본질적으로 여러 난제를 내포한다. 센서로부터 발생하는 측정 노이즈, 데이터에 포함된 무관한 점들인 아웃라이어(outliers), 스캔 위치에 따른 비균일한 포인트 밀도 등은 정합의 정확성을 저해하는 주요 요인이다. 이 중에서도 특히 두 포인트 클라우드 간에 겹치는 영역이 적은 저겹침(low-overlap) 시나리오는 신뢰할 수 있는 대응점(correspondence)을 찾는 것을 극도로 어렵게 만들어, 강건한(robust) 정합 알고리즘 개발에 있어 가장 큰 도전 과제로 남아있다.2


'GeoTransformer'라는 명칭은 인공지능 연구 분야의 광범위한 트렌드를 반영하듯, 서로 다른 도메인에서 독립적으로 개발된 여러 모델에 의해 사용되고 있어 잠재적인 혼동을 야기한다. 이러한 명칭의 충돌은 특정 기술을 처음 접하는 연구자들에게 상당한 혼란을 줄 수 있으므로, 기술 보고서는 이러한 모호성을 명확히 해소하는 중요한 역할을 수행해야 한다.

본 보고서에서 심층적으로 분석할 핵심 대상은 Zheng Qin 등이 컴퓨터 비전 분야의 최고 학회인 CVPR 2022에서 발표한 **"Geometric Transformer for Fast and Robust Point Cloud Registration"** 연구다.6 이 모델은 3D 포인트 클라우드 정합 문제에 특화되어 있으며, 'GeoTransformer'라는 약칭으로 학계에 널리 알려져 있다.1

이와는 별개로 존재하는 동명의 주요 모델들은 다음과 같다. 첫째, 도시 예측(Urban Forecasting) 분야의 GeoTransformer는 고차원 지역 임베딩과 동적 공간 모델링을 결합하여 GDP나 차량 공유 수요와 같은 사회경제적 지표를 예측한다.8 이 모델의 핵심은 지리적 통계학에 기반한 'Geospatial Attention Mechanism'으로, 본 보고서에서 다루는 포인트 클라우드 정합 모델과는 목표, 데이터 형태, 아키텍처 측면에서 근본적인 차이가 있다.9 둘째, 자연어 처리(NLP) 분야에서는 지리적 위치 정보를 'Geotokens'라는 형태로 토큰화하고, 이를 구면 좌표계에 맞게 조정된 위치 인코딩 방식으로 처리하는 "Geotokens and Geotransformers" 연구가 존재한다.11

따라서 본 보고서는 위와 같은 혼동을 방지하고, 3D 컴퓨터 비전 분야에서 가장 큰 영향력을 발휘하고 있는 CVPR 2022의 포인트 클라우드 정합 모델에 대한 명확하고 깊이 있는 분석을 제공하는 데 그 범위를 한정한다.


GeoTransformer는 기존 포인트 클라우드 정합 방법론의 한계를 극복하기 위한 몇 가지 혁신적인 기여를 통해 새로운 패러다임을 제시했다.

첫째, **Keypoint-free 접근법을 계승하고 발전**시켰다. 저겹침 시나리오에서는 반복적으로 검출 가능한 안정적인 키포인트(keypoint)를 찾기 어렵다는 문제의식 하에, GeoTransformer는 포인트 클라우드를 다운샘플링하여 생성된 '슈퍼포인트(superpoints)'라는 추상화된 단위 간의 대응 관계를 찾는 keypoint-free 패러다임을 채택했다.5 이는 정합 문제를 개별 포인트의 엄격한 매칭에서 슈퍼포인트 주변 패치(patch)의 느슨한 겹침을 찾는 문제로 완화시켜 저겹침 환경에서의 강건성을 높였다.

둘째, **RANSAC-free를 통한 속도 혁신**을 이루었다. 전통적인 정합 파이프라인은 부정확한 대응점(outliers)을 걸러내기 위해 RANSAC(Random Sample Consensus)과 같은 강건한 추정기에 크게 의존했다. 그러나 RANSAC은 반복적인 샘플링과 가설 검증 과정으로 인해 계산 비용이 매우 높다. GeoTransformer는 모델 아키텍처 자체에 기하학적 구조를 명시적으로 인코딩함으로써 매우 높은 인라이어 비율(inlier ratio)을 가진 대응점을 생성해낸다. 이처럼 대응점의 품질이 뛰어나기 때문에 RANSAC과 같은 후처리 과정 없이도 정확한 변환 행렬 추정이 가능해졌고, 이는 결과적으로 전체 정합 속도를 100배 이상 가속하는 혁신을 가져왔다.6 이는 통계적 강건성에 의존하던 기존 패러다임에서, 아키텍처 자체의 설계를 통해 강건성을 확보하는 방향으로의 전환을 의미한다.

셋째, **기하학적 불변성(Geometric Invariance)을 도입**했다. 기존 Transformer 기반 모델들은 점의 절대 좌표에 기반한 위치 임베딩을 사용했기 때문에, 입력 포인트 클라우드의 회전이나 이동과 같은 강체 변환(rigid transformation)에 민감한 특징을 학습하는 경향이 있었다.1 GeoTransformer는 이러한 문제를 해결하기 위해 강체 변환에 대해 불변(invariant)인 기하학적 속성, 즉 두 점 사이의 거리(pair-wise distances)와 세 점이 이루는 각도(triplet-wise angles)만을 인코딩하여 Transformer의 어텐션 메커니즘에 주입했다. 이로써 모델은 포인트 클라우드가 어떤 임의의 자세(pose)로 주어지더라도 일관되고 강건한 기하학적 특징을 학습할 수 있게 되었다.1



GeoTransformer는 복잡한 포인트 클라우드 정합 문제를 여러 단계로 나누어 해결하는 'Coarse-to-Fine' 계층적 접근법을 채택하고 있다.7 이러한 구조는 계산 효율성과 정합 정확도 사이의 균형을 효과적으로 달성하기 위한 전략적 선택이다. 전체 데이터 처리 흐름은 다음과 같은 단계로 구성된다.

1. **특징 추출 및 다운샘플링:** 두 개의 입력 포인트 클라우드는 먼저 KPConv-FPN 백본 네트워크를 통과한다. 이 과정에서 각 포인트에 대한 특징 벡터가 추출되는 동시에, 포인트 클라우드는 계층적으로 다운샘플링된다.7
2. **슈퍼포인트 생성:** 가장 낮은 해상도(가장 희소한)의 포인트들이 '슈퍼포인트'로 정의된다. 이 슈퍼포인트들은 전체 포인트 클라우드를 대표하는 핵심 노드 역할을 한다.
3. **슈퍼포인트 정합:** 슈퍼포인트들 간의 대응 관계를 찾기 위해 Geometric Transformer 모듈이 사용된다. 이 모듈은 기하학적 구조 임베딩을 활용하여 변환에 불변인 특징을 학습하고, 이를 통해 강건한 슈퍼포인트 매칭을 수행한다.7
4. **포인트 정합:** 슈퍼포인트 레벨에서 얻어진 희소한(sparse) 대응 관계는 포인트 정합 모듈을 통해 조밀한(dense) 포인트 레벨로 확장된다. 이 과정에서는 최적 수송(Optimal Transport) 이론이 사용되어 슈퍼포인트 주변의 로컬 패치 내 포인트들 간의 최적 매칭을 찾는다.7
5. **변환 행렬 추정:** 최종적으로 생성된 고품질의 조밀한 대응점들을 기반으로, 두 포인트 클라우드를 정렬하는 최종 변환 행렬(회전 및 이동)이 추정된다.

이처럼 문제를 추상화된 슈퍼포인트 레벨에서 먼저 해결하고, 그 결과를 바탕으로 세부적인 포인트 레벨로 정제해 나가는 '분할 정복(divide and conquer)' 전략은 전체 탐색 공간을 크게 줄여 계산 효율성을 높이는 동시에, 각 단계가 더 제한되고 단순화된 문제를 풀게 함으로써 최종적인 정합 정확도를 향상시킨다.


GeoTransformer의 파이프라인 첫 단계를 담당하는 특징 추출기는 KPConv-FPN 아키텍처를 기반으로 한다. 이는 3D 포인트 클라우드의 고유한 특성을 효과적으로 처리하기 위해 설계되었다.

**KPConv (Kernel Point Convolution)**는 포인트 클라우드와 같이 비정형적이고 순서가 없는 데이터에 직접 적용할 수 있는 3D 컨볼루션 연산이다.7 기존의 그리드 기반 CNN과 달리, KPConv는 소수의 커널 포인트(kernel points)를 정의하고, 이 커널 포인트들과 주변 점들 간의 상대적 위치에 따라 가중치를 학습한다. 이러한 유연성 덕분에 KPConv는 밀도 변화에 강건하고 복잡한 기하학적 구조로부터 유의미한 특징을 효과적으로 추출할 수 있다.

**FPN (Feature Pyramid Network)**은 다양한 해상도의 특징 맵을 생성하고 이를 결합하는 구조다. GeoTransformer에서는 KPConv와 FPN을 결합하여 다중 스케일의 특징을 동시에 추출한다. 이를 통해 가장 조밀한 초기 레벨의 포인트들(dense points)과 가장 희소한 최종 레벨의 슈퍼포인트들(superpoints)이 각각의 스케일에 맞는 풍부한 컨텍스트 정보를 가진 특징 벡터를 갖게 된다.7 이 구조는 데이터셋의 규모에 따라 유연하게 조정되는데, 예를 들어 상대적으로 작은 실내 장면 데이터셋인 3DMatch에서는 4단계 백본을 사용하는 반면, 훨씬 크고 복잡한 실외 LiDAR 데이터셋인 KITTI에서는 5단계 백본을 사용하여 더 넓은 범위의 컨텍스트를 포착한다.14


GeoTransformer의 가장 핵심적인 혁신은 Transformer의 어텐션 메커니즘에 강체 변환 불변(rigid-transformation-invariant) 기하학적 정보를 명시적으로 주입하는 '기하학적 구조 임베딩'에 있다. 이는 "점들이 공간상의 어디에 있는가?"라는 절대적 위치 정보 대신, "점들이 서로 어떻게 관계를 맺고 있는가?"라는 상대적 구조 정보에 집중하도록 모델을 유도한다. 표준 Transformer가 사용하는 좌표 기반 위치 임베딩은 입력 데이터의 회전이나 이동에 따라 그 값이 변하기 때문에 정합 문제에 본질적으로 부적합하다.1 GeoTransformer는 이 문제를 해결하기 위해 다음과 같은 두 가지 불변 기하학적 속성을 인코딩한다.

**쌍 거리 임베딩 (Pair-wise Distance Embedding):** 두 슈퍼포인트 $\hat{p}_i$와 $\hat{p}_j$ 사이의 유클리드 거리 $\rho_{i,j} = \|\hat{p}_i - \hat{p}_j\|_2$는 강체 변환에 불변이다. 이 스칼라 값을 Transformer가 처리할 수 있는 고차원 벡터로 변환하기 위해 sinusoidal 함수가 사용된다. 이는 원래 Transformer의 위치 인코딩에서 영감을 받은 방식이다. 거리 임베딩 $r^D_{i,j}$는 다음과 같이 계산된다.14
$$
\begin{cases}
r^D_{i,j,2k} = \sin\left(\frac{\rho_{i,j}}{\sigma_d \cdot 10000^{2k/d_t}}\right) \\
r^D_{i,j,2k+1} = \cos\left(\frac{\rho_{i,j}}{\sigma_d \cdot 10000^{2k/d_t}}\right)
\end{cases}
$$
여기서 $d_t$는 임베딩 벡터의 차원이며, $\sigma_d$는 거리 변화에 대한 민감도를 조절하는 온도(temperature) 하이퍼파라미터다. 이 값은 데이터셋의 스케일에 따라 조정되는데, 실내 3DMatch 데이터셋에서는 0.2m, 대규모 실외 KITTI 데이터셋에서는 4.8m로 설정되어 각 환경의 기하학적 스케일에 맞게 임베딩을 조정한다.14

**삼중항 각도 임베딩 (Triplet-wise Angular Embedding):** 두 점 사이의 거리를 넘어 더 고차원적인 구조 정보를 포착하기 위해, 세 개의 슈퍼포인트($\hat{p}_i, \hat{p}_j, \hat{p}_k$)가 이루는 각도 $\alpha$ 또한 인코딩된다.1 각도 역시 강체 변환에 불변인 속성이며, 이를 통해 모델은 단순한 거리 관계를 넘어선 삼각형 구조와 같은 지역적 형태 정보를 학습할 수 있다. 각도 임베딩은 거리 임베딩과 동일한 형태의 sinusoidal 함수를 사용하여 계산되며, 각도 민감도를 조절하는 $\sigma_a$는 두 데이터셋 모두에서 15°로 설정되었다.14

이러한 기하학적 임베딩은 모델에 강력한 관계적 편향(relational bias)을 주입한다. 즉, 모델이 장면의 내재적이고 불변하는 기하학적 골격을 학습하도록 강제하며, 이는 특히 정보가 부족한 저겹침 시나리오에서 모델이 강건하게 동작하는 핵심적인 이유가 된다.


기하학적 구조 정보로 무장한 슈퍼포인트 특징들은 슈퍼포인트 정합 모듈, 즉 Geometric Transformer를 통해 처리된다. 이 모듈은 두 가지 종류의 어텐션 블록을 교차(interleave)하여 쌓은 구조를 가진다. 이는 각 포인트 클라우드 내부의 컨텍스트를 강화하는 단계와 두 포인트 클라우드 간의 정보를 교환하는 단계를 분리하여 효과적인 정보 흐름을 만들기 위함이다.14

**기하학적 자기-어텐션 (Geometric Self-Attention):** 이 모듈은 각 포인트 클라우드에 독립적으로 적용되어 내부의 슈퍼포인트 특징들을 정제하고 강화한다. 표준 자기-어텐션과 마찬가지로 각 슈퍼포인트는 다른 모든 슈퍼포인트와의 관계를 통해 자신의 특징 표현을 업데이트한다. GeoTransformer의 핵심은 어텐션 가중치를 계산할 때, 앞서 생성된 쌍 거리 및 삼중항 각도 임베딩을 어텐션 편향(attention bias)으로 추가한다는 점이다. 이를 통해 기하학적으로 가깝거나 구조적으로 관련된 슈퍼포인트들 사이에 더 높은 어텐션 점수가 부여되어, 모델이 각 슈퍼포인트의 지역적 및 전역적 기하학적 컨텍스트를 풍부하게 학습하게 된다.1

**특징 기반 교차-어텐션 (Feature-based Cross-Attention):** 자기-어텐션을 통해 강화된 두 포인트 클라우드의 특징들은 교차-어텐션 모듈에서 상호작용한다. 예를 들어, 소스 포인트 클라우드의 슈퍼포인트 특징들이 Query로 사용되고, 타겟 포인트 클라우드의 특징들이 Key와 Value로 사용된다. 이를 통해 소스의 각 슈퍼포인트는 타겟의 모든 슈퍼포인트들을 '살펴보고' 자신과 가장 관련성이 높은 정보를 가져와 특징을 업데이트한다. 이 과정은 두 포인트 클라우드 간의 잠재적인 대응 관계를 탐색하고, 서로의 정보를 바탕으로 특징 표현을 상호 보완적으로 정제하는 역할을 한다.14 이 두 어텐션 모듈은 여러 번 번갈아 가며 적용되어, 점진적으로 더 정교하고 신뢰도 높은 특징 표현을 구축한다.


Geometric Transformer를 통해 얻어진 슈퍼포인트 레벨의 희소한 대응 관계는 최종적인 정밀 정합을 위해 조밀한 포인트 레벨로 확장되어야 한다. 이 역할을 포인트 정합 모듈이 담당하며, 최적 수송(Optimal Transport) 이론을 기반으로 동작한다.7

대응 관계로 예측된 한 쌍의 슈퍼포인트(소스의 $\hat{p}_i$, 타겟의 $\hat{p}_j$)가 주어지면, 각 슈퍼포인트 주변의 로컬 패치(patch)에 속하는 실제 포인트들 간의 매칭 문제가 정의된다. 이 문제는 한 분포(소스 패치의 포인트들)를 다른 분포(타겟 패치의 포인트들)로 옮기는 데 필요한 최소 비용을 찾는 최적 수송 문제로 공식화될 수 있다. 여기서 비용은 두 포인트의 특징 벡터 간의 유사도로 정의된다.

이 문제를 해결하기 위해, 미분 가능(differentiable)하고 효율적인 **Sinkhorn 알고리즘**이 사용된다.3 Sinkhorn 알고리즘은 반복적인 행과 열의 정규화를 통해 최적 수송 문제의 근사해인 소프트 할당 행렬(soft assignment matrix)을 계산한다. 이 행렬의 각 원소는 소스 패치의 특정 포인트가 타겟 패치의 특정 포인트와 매칭될 확률을 나타낸다. 이 과정을 통해 슈퍼포인트 레벨의 단일 대응 관계가 수십 개의 조밀한 포인트 대응 관계로 정제되고 확장된다.


GeoTransformer의 학습을 지도하는 손실 함수로는 'Overlap-aware Circle Loss'가 사용된다. 이는 표준적인 분류 손실인 Cross-Entropy Loss의 한계를 극복하기 위해 고안되었다.1

하나의 소스 슈퍼포인트는 여러 개의 타겟 슈퍼포인트와 겹칠 수 있으므로, 이 문제는 다중 정답(multi-label) 분류 문제로 볼 수 있다. 이러한 상황에서 Cross-Entropy 손실은 이미 높은 신뢰도(confidence)를 보이는 정답 쌍에 대해서는 그래디언트를 줄여 학습을 억제하는 경향이 있다. 이는 모델이 가장 확실한 하나의 대응 관계에만 집중하고 다른 유효한 대응 관계들을 놓치게 만들 수 있다.14

**Circle Loss**는 이러한 문제를 해결하기 위해 메트릭 러닝(metric learning) 관점에서 접근한다.1 이 손실 함수는 정답 쌍(positive pairs)의 유사도는 높이고 오답 쌍(negative pairs)의 유사도는 낮추도록 직접적으로 최적화한다. 중요한 것은 각 정답 쌍과 오답 쌍이 독립적으로 손실에 기여하도록 설계되어, 하나의 정답 쌍이 다른 정답 쌍의 학습을 방해하지 않는다는 점이다.

여기에 **Overlap-aware 가중치** 개념이 추가된다. 두 슈퍼포인트의 로컬 패치가 실제로 겹치는 면적이 클수록 더 신뢰할 수 있는 대응 관계이므로, 이 겹침 비율을 손실 계산 시 가중치로 사용한다. 즉, 겹침이 많은 중요한 대응 관계에 대해 더 큰 손실 값을 부여하여 모델이 이러한 쌍에 더 집중하도록 유도한다.1 이 두 가지 요소의 결합을 통해 GeoTransformer는 더 안정적이고 효과적으로 고품질의 대응 관계를 학습할 수 있다.



GeoTransformer의 성능은 다양한 시나리오를 포괄하는 표준 벤치마크 데이터셋을 통해 종합적으로 평가되었다. 각 데이터셋은 정합 알고리즘이 마주할 수 있는 서로 다른 도전 과제를 대표한다.

- **데이터셋:**
  - **3DMatch / 3DLoMatch:** 실내 장면을 RGB-D 카메라로 재구성한 데이터셋으로, 딥러닝 기반 정합 방법론의 표준 벤치마크로 널리 사용된다. 특히 3DLoMatch는 두 스캔 간의 겹침 비율이 10%에서 30% 사이로 매우 낮은 쌍들만 포함하여, 저겹침 시나리오에서의 강건성을 평가하는 데 핵심적인 역할을 한다.5
  - **KITTI:** 자율주행 연구를 위해 수집된 대규모 실외 LiDAR 데이터셋이다. 넓은 범위, 비균일한 포인트 밀도, 동적 객체 등의 특성을 가지며, 실외 환경에서의 알고리즘 성능을 검증하는 데 사용된다.5
  - **ModelNet40:** 다양한 카테고리의 3D CAD 모델로 구성된 합성 데이터셋으로, 주로 객체 분류에 사용되지만 정합 모델의 일반화 능력과 노이즈에 대한 강건성을 테스트하는 데에도 활용된다.5
- **핵심 지표:**
  - **정합 재현율 (Registration Recall, RR):** 모델이 추정한 변환 행렬과 실제(ground-truth) 변환 행렬 간의 오차(RMSE)를 계산하여, 이 오차가 사전에 정의된 임계값(예: 회전 오차 5°, 이동 오차 2m) 이하일 경우 '성공'으로 간주한다. RR은 전체 테스트 쌍 중 성공한 비율로, 모델의 전반적인 정확도와 신뢰도를 나타내는 가장 중요한 지표다.5
  - **인라이어 비율 (Inlier Ratio, IR):** 모델이 생성한 전체 대응점 중에서, 실제 변환 행렬을 적용했을 때 두 포인트 간의 거리가 특정 임계값(예: 10cm) 이내인 '인라이어(inlier)' 대응점의 비율이다. 이 지표는 생성된 대응점의 품질을 직접적으로 측정하며, IR이 높을수록 후속 변환 추정 단계가 더 쉽고 정확해진다.6
  - **특징 매칭 재현율 (Feature Matching Recall, FMR):** 두 포인트 클라우드에서 올바른 대응점 쌍이 서로의 가장 가까운 이웃(nearest neighbor)으로 매칭되는 비율로, 주로 특징 기술자(feature descriptor) 자체의 성능을 평가하는 데 사용된다.


GeoTransformer는 기존의 최첨단(SOTA) 모델들과의 비교를 통해 특히 어려운 저겹침 시나리오에서 압도적인 성능 향상을 입증했다. 아래 표는 3DMatch와 3DLoMatch 벤치마크에서의 주요 성능 지표를 요약한 것이다.

| 모델 (Method)        | 벤치마크 (Benchmark) | 정합 재현율 (RR, %) ↑ | 인라이어 비율 (IR, %) ↑ | RANSAC 사용 여부         |
| -------------------- | -------------------- | --------------------- | ----------------------- | ------------------------ |
| PREDATOR 5           | 3DMatch              | 97.1                  | 58.5                    | Yes                      |
| CoFiNet 12           | 3DMatch              | 96.9                  | 37.8                    | Yes                      |
| **GeoTransformer** 5 | **3DMatch**          | **98.2**              | **70.9**                | **No (Local-to-Global)** |
| PREDATOR 5           | 3DLoMatch            | 81.8                  | 32.1                    | Yes                      |
| CoFiNet 12           | 3DLoMatch            | 82.1                  | 21.3                    | Yes                      |
| **GeoTransformer** 5 | **3DLoMatch**        | **89.1**              | **62.3**                | **No (Local-to-Global)** |

이 표에서 드러나는 가장 중요한 결과는 3DLoMatch 벤치마크에서의 성능이다. 저겹침 환경은 신뢰할 수 있는 특징을 찾기 어려워 기존 방법들이 고전하던 영역이다. GeoTransformer는 이 벤치마크에서 이전 SOTA 모델 대비 정합 재현율(RR)을 약 7%p 이상, 그리고 인라이어 비율(IR)은 무려 30%p 이상 극적으로 향상시켰다.6 이는 GeoTransformer의 기하학적 불변 특징이 정보가 희소한 상황에서도 매우 효과적으로 강건한 대응 관계를 찾아냄을 명확히 입증하는 결과다. 3DLoMatch에서의 이러한 압도적인 성능 향상은 GeoTransformer의 아키텍처 철학, 즉 변환 불변의 기하학적 구조를 학습하는 것이 저겹침 문제 해결의 핵심이라는 주장을 강력하게 뒷받침한다.

상대적으로 쉬운 3DMatch 벤치마크에서도 GeoTransformer는 기존 모델들을 능가하는 SOTA 성능을 달성하며, 높은 겹침 시나리오에서도 모델의 효과가 유효함을 보여주었다.5


GeoTransformer의 가장 큰 실용적 장점 중 하나는 RANSAC을 사용하지 않는다는 점이다. 이는 단순히 하나의 단계를 생략하는 것을 넘어, 정합 파이프라인의 근본적인 효율성과 안정성을 개선한다.

- **속도 비교:** RANSAC은 수많은 반복을 통해 무작위로 샘플링된 대응점 부분집합으로부터 변환 가설을 생성하고 검증하는 과정을 거친다. 인라이어 비율이 낮을수록 올바른 가설을 찾기 위해 훨씬 더 많은 반복이 필요하며, 이는 심각한 계산 병목을 유발한다. GeoTransformer는 매우 높은 인라이어 비율(3DLoMatch에서 62.3%)을 가진 대응점을 생성하므로 이러한 확률적 탐색 과정이 불필요하다. 대신, 제안된 Local-to-Global 등록 방식을 통해 변환을 효율적으로 추정한다. 그 결과, 자세 추정(pose estimation)에 소요되는 시간은 RANSAC 대비 100배 이상 빠르며, 예를 들어 5,000개의 대응점에 대해 단 0.01초 만에 완료된다.7 이 속도 향상은 단순한 엔지니어링 최적화가 아니라, 모델의 핵심 능력인 고품질 대응점 생성 능력에서 비롯된 직접적인 결과다. 모델이 자신의 주된 임무를 매우 잘 수행하기 때문에, 후속의 복잡한 통계적 필터링 단계 전체가 불필요해진 것이다.
- **안정성:** RANSAC은 무작위 샘플링에 기반하기 때문에 실행할 때마다 결과에 약간의 편차가 발생할 수 있으며, 최악의 경우 잘못된 해를 찾을 수도 있다. 반면, GeoTransformer의 Local-to-Global 등록 방식은 결정론적(deterministic)으로 동작하여 항상 일관되고 안정적인 결과를 제공한다.12


정량적 수치 외에도, 정성적 결과는 GeoTransformer가 어떻게 기하학적 컨텍스트를 이해하는지를 직관적으로 보여준다. 공개된 논문의 시각화 자료를 분석하면, 대칭적인 구조를 가진 객체(예: 의자 등받이의 양쪽 모서리)에서 기존 방법들이 혼동을 일으켜 잘못된 매칭을 생성하는 반면, GeoTransformer는 전체적인 공간적 일관성(spatial consistency)을 고려하여 올바른 대응 관계를 찾아내는 것을 확인할 수 있다.1 이는 모델이 개별 점의 지역적 특징뿐만 아니라, 점들이 모여 형성하는 더 큰 구조 내에서의 상대적 위치와 관계를 이해하고 있음을 시사한다.



GeoTransformer는 발표 이후 3D 포인트 클라우드 정합 분야에서 중요한 이정표로 자리 잡았으며, 수많은 후속 연구의 기반이 되거나 비교 대상으로 활용되었다. 이러한 연구들은 GeoTransformer의 성능을 더욱 향상시키려는 시도와, 그 내재적 한계를 극복하려는 시도로 나눌 수 있다.

- **성능 향상을 위한 확장:**
  - **MAC (Maximal Cliques):** 이 연구는 GeoTransformer를 강력한 '초기 대응점 생성기'로 활용하는 대표적인 사례다.15 MAC은 GeoTransformer가 생성한 대응점 집합을 입력으로 받아, 이를 그래프로 모델링한다. 그 후, 그래프 이론의 Maximal Clique 탐색 알고리즘을 적용하여 기하학적으로 상호 일관성을 가지는 대응점들의 부분집합(consensus set)을 효율적으로 찾아낸다. 이 과정을 통해 GeoTransformer의 결과에 남아있을 수 있는 소수의 아웃라이어들을 효과적으로 제거하고 변환 추정의 정확도를 극대화한다. GeoTransformer와 MAC을 결합한 파이프라인은 3DMatch와 3DLoMatch 벤치마크에서 각각 95.7%와 78.9%라는 기록적인 정합 재현율(RR)을 달성했다.16 이는 GeoTransformer가 하나의 완결된 솔루션일 뿐만 아니라, 더 큰 파이프라인의 핵심 구성 요소로 기능할 수 있는 강력한 기반 기술임을 보여준다.
- **한계점 극복을 위한 연구:**
  - **BUFFER:** 이 연구는 GeoTransformer와 같은 점 단위(point-wise) 방법론이 가진 잠재적 약점, 즉 '일반화(generalization)' 성능에 주목했다.18 점 단위 방법은 전체 포인트 클라우드의 전역적 컨텍스트를 학습하는 경향이 있어, 학습 데이터와 특성이 매우 다른 새로운 도메인(예: 실내 데이터로 학습 후 실외 데이터에 적용)에 대해서는 성능이 저하될 수 있다. BUFFER는 이러한 문제를 해결하기 위해 점 단위 학습기(Point-wise Learner)와 패치 단위 학습기(Patch-wise Embedder)를 결합한 하이브리드 아키텍처를 제안했다. 이를 통해 전역적 컨텍스트와 강건한 지역적 특징을 모두 활용하여 정확도, 효율성, 그리고 일반화 능력 간의 균형을 맞추고자 했다. 실제로 3DMatch 데이터셋으로만 학습한 후, 이전에 보지 못한 실외 ETH 데이터셋에서 테스트했을 때 BUFFER는 GeoTransformer보다 약 10%p 높은 성공률을 기록하며 개선된 일반화 성능을 입증했다.18 이는 GeoTransformer의 강점인 전역적 기하학 구조 이해 능력이 특정 상황에서는 오히려 도메인 편향을 야기할 수 있음을 시사하며, 일반화 성능을 위해서는 강건한 지역 특징의 중요성을 다시 한번 일깨워준다.
  - **RoITr:** GeoTransformer의 프레임워크를 직접적인 기반으로 삼아, 회전 불변성(rotation invariance)을 더욱 강화하려는 시도다. RoITr은 기하학적 특징을 자기-어텐션 모듈에서 교차-어텐션 모듈로 명시적으로 임베딩하는 구조를 제안하여, Transformer 구조 자체의 회전 불변성을 높이고자 했다.3


GeoTransformer의 높은 정확도, 빠른 속도, 그리고 저겹침 환경에서의 강건성은 다양한 산업 분야에서 실질적인 응용 가능성을 열어준다.

- **자율주행 및 SLAM:** 자율주행 차량의 핵심 기술인 SLAM은 LiDAR 스캔 데이터를 연속적으로 정합하여 차량의 위치를 추정하고 주변 환경 지도를 생성한다. 도심이나 터널과 같이 GPS 신호가 약하거나 없는 환경에서는 LiDAR SLAM의 정확성이 무엇보다 중요하다. GeoTransformer는 순차적인 스캔 간의 정합(scan-to-scan)이나 현재 스캔을 기존 지도에 정합(scan-to-map)하는 과정에서 빠르고 정확한 결과를 제공할 수 있다. 특히 교차로나 코너링 상황처럼 연속된 스캔 간의 겹침이 순간적으로 낮아지는 경우에도 강건하게 동작하여 SLAM 시스템 전체의 안정성을 크게 향상시킬 수 있다.19
- **3D 재구성 및 디지털 트윈:** 건축, 제조, 플랜트, 문화유산 보존 등의 분야에서는 현실 세계의 대상을 정밀한 3D 모델로 복제하는 디지털 트윈 기술이 각광받고 있다. 이를 위해서는 여러 위치에서 측정한 포인트 클라우드 스캔들을 하나의 모델로 완벽하게 병합해야 한다. GeoTransformer의 정밀한 정합 능력은 스캔 데이터 간의 누적 오차를 최소화하여 고품질의 3D 모델 및 디지털 트윈을 구축하는 데 필수적인 역할을 할 수 있다.2
- **로보틱스:** 공장 자동화나 물류 창고에서 로봇이 특정 부품을 집거나(grasping) 조작하기 위해서는 대상 객체의 정확한 3차원 위치와 자세를 인식해야 한다. 로봇에 장착된 3D 센서로부터 얻은 포인트 클라우드를 사전에 저장된 객체 모델과 정합함으로써 이를 달성할 수 있다. GeoTransformer의 높은 정확성과 속도는 로봇이 실시간으로 주변 환경과 정밀하게 상호작용하는 것을 가능하게 한다.22



GeoTransformer는 3D 포인트 클라우드 정합 분야에 몇 가지 중대한 기여를 남긴 이정표적인 연구로 평가된다.

첫째, 강체 변환에 불변인 **기하학적 특징(쌍 거리, 삼중항 각도)을 Transformer 아키텍처에 성공적으로 통합**하여, 딥러닝 모델이 데이터의 내재적 기하학 구조를 학습할 수 있는 새로운 방향을 제시했다. 이는 기존의 좌표 기반 접근법이 가졌던 변환 민감성 문제를 근본적으로 해결했다.

둘째, 높은 품질의 대응점을 생성함으로써 **RANSAC과 같은 통계적 후처리 과정의 필요성을 제거**했다. 이는 Coarse-to-fine 파이프라인의 효율성을 극대화하여 정합 속도를 획기적으로 개선했으며, 정확도와 속도라는 두 가지 상충되는 목표를 동시에 달성하는 데 성공했다.

셋째, 정보가 부족한 **저겹침(low-overlap) 시나리오에서 기존 방법론들의 성능을 극적으로 뛰어넘는 돌파구**를 마련했다. 이는 GeoTransformer가 포착하는 기하학적 컨텍스트가 희소한 데이터 환경에서 특히 강력한 신호로 작용함을 입증한 결과다.


이러한 혁신적인 기여에도 불구하고, GeoTransformer는 몇 가지 내재적 한계를 가지며 이는 향후 연구의 중요한 방향을 제시한다.

- **일반화 성능:** BUFFER 연구에서 지적되었듯이, GeoTransformer의 점 단위(point-wise) 및 전역적 컨텍스트 중심의 접근 방식은 학습 데이터와 상이한 도메인으로 일반화될 때 성능 저하를 겪을 수 있다.18 향후 연구는 특정 도메인에 대한 과적합을 방지하고 다양한 환경과 센서에 강건한 모델을 개발하기 위해, 지역적 특징과 전역적 특징을 결합하는 하이브리드 접근법이나 도메인 적응(domain adaptation) 기술을 심도 있게 탐구할 필요가 있다.
- **계산 복잡도:** Transformer 기반 모델의 고질적인 문제로, 입력 시퀀스(여기서는 슈퍼포인트의 수)의 길이가 길어질수록 어텐션 연산의 계산 복잡도가 제곱($O(N^2)$)으로 증가한다. 대규모 실외 장면과 같이 슈퍼포인트의 수가 매우 많아질 경우, 이는 메모리 및 계산 시간 측면에서 병목이 될 수 있다. UTOPIC과 같은 후속 연구는 이러한 복잡도를 줄여 GeoTransformer의 계산 효율을 개선하려는 시도를 하고 있으며 13, 더 효율적인 어텐션 메커니즘에 대한 연구가 지속적으로 요구된다.
- **반복적인 기하학적 구조:** 대규모 씬에서는 건물 창문, 복도의 기둥, 가로등과 같이 기하학적으로 유사하거나 동일한 구조가 반복적으로 나타날 수 있다. GeoTransformer는 주로 기하학적 정보에 의존하기 때문에, 이러한 구조적 모호성(ambiguity)이 있는 상황에서는 잘못된 매칭을 유발할 가능성이 있다.13 이를 해결하기 위해서는 기하학적 정보를 넘어, 객체의 종류나 재질과 같은 시맨틱(semantic) 정보나 RGB 이미지로부터 얻을 수 있는 텍스처 정보를 통합하여 모호성을 해소하는 다중 모달(multi-modal) 접근법이 유망한 연구 방향이 될 것이다.


결론적으로, GeoTransformer는 딥러닝 기반 포인트 클라우드 정합 연구의 흐름에 중요한 변곡점을 만들었다. 이 연구는 '어떻게 데이터의 내재적 특성과 기하학적 선험 지식(prior)을 효과적으로 딥러닝 아키텍처에 통합할 것인가'라는 근본적인 질문에 대한 성공적인 해답을 제시했다.

이는 3D 비전 분야의 연구가 단순히 더 깊고 큰 네트워크를 통해 데이터로부터 모든 것을 학습하려는 경향에서 벗어나, 대상 데이터가 가진 고유한 불변성, 대칭성, 구조적 특성을 모델 설계 단계에서부터 적극적으로 활용하는 방향으로 나아가는 데 중요한 영감을 제공한다. GeoTransformer가 제시한 원칙은 포인트 클라우드 정합을 넘어 다른 3D 비전 태스크에서도 데이터의 본질을 이해하고 활용하는 더 정교하고 효율적인 모델 개발을 촉진하는 기폭제가 될 것이다.


1. GeoTransformer: Fast and Robust Point Cloud Registration With Geometric Transformer, 8월 23, 2025에 액세스, https://www.computer.org/csdl/journal/tp/2023/08/10076895/1LFQ0GcHene
2. Spatial deformable transformer for 3D point cloud registration - PMC, 8월 23, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC10917764/
3. A Registration Method of Overlap Aware Point Clouds Based on Transformer-to-Transformer Regression - MDPI, 8월 23, 2025에 액세스, https://www.mdpi.com/2072-4292/16/11/1898
4. Point-set registration - Wikipedia, 8월 23, 2025에 액세스, https://en.wikipedia.org/wiki/Point-set_registration
5. qinzheng93/GeoTransformer: [CVPR2022] Geometric Transformer for Fast and Robust Point Cloud Registration - GitHub, 8월 23, 2025에 액세스, https://github.com/qinzheng93/GeoTransformer
6. [2202.06688] Geometric Transformer for Fast and Robust Point Cloud Registration - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/abs/2202.06688
7. Geometric Transformer for Fast and Robust Point Cloud Registration - CVF Open Access, 8월 23, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2022/papers/Qin_Geometric_Transformer_for_Fast_and_Robust_Point_Cloud_Registration_CVPR_2022_paper.pdf
8. GeoTransformer: Enhancing Urban Forecasting with Dependency Retrieval and Geospatial Attention - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/html/2408.08852v2
9. GeoTransformer: Enhancing Urban Forecasting with Geospatial Attention Mechanisms, 8월 23, 2025에 액세스, https://arxiv.org/html/2408.08852v1
10. GeoTransformer: Enhancing Urban Forecasting with Geospatial Attention Mechanisms | AI Research Paper Details - AIModels.fyi, 8월 23, 2025에 액세스, https://www.aimodels.fyi/papers/arxiv/geotransformer-enhancing-urban-forecasting-dependency-retrieval-geospatial
11. [2403.15940] Geotokens and Geotransformers - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/abs/2403.15940
12. Geometric Transformer for Fast and Robust Point Cloud Registration | CVPR 2022, 8월 23, 2025에 액세스, https://www.youtube.com/watch?v=BbG4hhaLXhE
13. Detailed architecture of the geometry transformer. - ResearchGate, 8월 23, 2025에 액세스, https://www.researchgate.net/figure/Detailed-architecture-of-the-geometry-transformer_fig4_369404184
14. Geometric Transformer for Fast and Robust ... - CVF Open Access, 8월 23, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2022/supplemental/Qin_Geometric_Transformer_for_CVPR_2022_supplemental.pdf
15. 3D Registration With Maximal Cliques - CVPR 2023 Open Access Repository, 8월 23, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2023/html/Zhang_3D_Registration_With_Maximal_Cliques_CVPR_2023_paper.html
16. 3D Registration With Maximal Cliques - CVF Open Access - The ..., 8월 23, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2023/papers/Zhang_3D_Registration_With_Maximal_Cliques_CVPR_2023_paper.pdf
17. MAC: Maximal Cliques for 3D Registration - ResearchGate, 8월 23, 2025에 액세스, https://www.researchgate.net/publication/383091490_MAC_Maximal_Cliques_for_3D_Registration
18. BUFFER: Balancing Accuracy, Efficiency, and ... - CVF Open Access, 8월 23, 2025에 액세스, https://openaccess.thecvf.com/content/CVPR2023/papers/Ao_BUFFER_Balancing_Accuracy_Efficiency_and_Generalizability_in_Point_Cloud_Registration_CVPR_2023_paper.pdf
19. DeepPointMap: Advancing LiDAR SLAM with Unified Neural Descriptors - arXiv, 8월 23, 2025에 액세스, https://arxiv.org/html/2312.02684v1
20. GeoTransformer: Fast and Robust Point Cloud Registration With Geometric Transformer | Request PDF - ResearchGate, 8월 23, 2025에 액세스, https://www.researchgate.net/publication/369394843_GeoTransformer_Fast_and_Robust_Point_Cloud_Registration_With_Geometric_Transformer
21. What Is SLAM (Simultaneous Localization and Mapping)? - MATLAB & Simulink, 8월 23, 2025에 액세스, https://www.mathworks.com/discovery/slam.html
22. Point Cloud Registration in Action - Number Analytics, 8월 23, 2025에 액세스, https://www.numberanalytics.com/blog/point-cloud-registration-action-cognitive-robotics


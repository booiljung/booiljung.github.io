# EigenPlaces 시점 강인성(Viewpoint Robustness)을 위한 새로운 훈련 패러다임 분석



시각적 장소 인식(Visual Place Recognition, VPR)은 주어진 쿼리 이미지(query image)의 시각적 특징 정보만을 활용하여 해당 이미지가 촬영된 지리적 위치를 추론하는 컴퓨터 비전의 핵심 과업이다.1 이 과업은 본질적으로 수백만 장 이상의 지오태깅된(geotagged) 이미지를 포함하는 대규모 데이터베이스 내에서 쿼리 이미지와 가장 시각적으로 유사한 이미지를 검색하는 대규모 이미지 검색(large-scale image retrieval) 문제로 귀결된다.4 VPR 시스템의 일반적인 파이프라인은 두 단계로 구성된다. 첫째, 오프라인(offline) 단계에서 데이터베이스에 포함된 모든 이미지로부터 고유한 시각적 특징을 압축한 전역 기술자(global descriptor)를 추출하여 검색이 용이하도록 인덱싱한다. 둘째, 온라인(online) 단계에서 사용자가 제시한 쿼리 이미지에 대해 동일한 방식으로 기술자를 추출하고, 이를 데이터베이스 인덱스와 비교하여 가장 거리가 가까운, 즉 가장 유사한 이미지들(nearest neighbors)을 찾아낸다. 최종적으로 검색된 이미지들의 지리적 위치 정보를 바탕으로 쿼리 이미지의 위치를 추정하게 된다.5

이 과정에서 '장소(place)'를 어떻게 정의하는가는 VPR 기술의 방향성을 결정하는 중요한 요소이다. 관련 분야인 랜드마크 인식(Landmark Retrieval)에서는 촬영 위치와 무관하게 동일한 랜드마크를 담고 있으면 같은 장소로 간주하는 반면, 시각적 위치 추정(Visual Localization)은 거의 동일한 카메라 포즈(pose)를 가져야 같은 장소로 인정한다. VPR은 이 두 극단 사이에서, 통상적으로 '쿼리 이미지의 실제 촬영 위치로부터 반경 25미터 이내에서 촬영된 이미지'를 동일 장소로 정의하는 실용적인 기준을 채택한다.8 이러한 정의는 VPR 기술이 어느 정도의 시점 변화는 허용하면서도 지리적 근접성에는 매우 민감한 기술자를 학습해야 함을 의미하며, 이는 EigenPlaces와 같은 모델이 해결하고자 하는 독특한 최적화 문제를 야기한다.


VPR 시스템이 현실 세계에서 안정적으로 동작하기 위해서는 여러 가지 근본적인 시각적 난제들을 극복해야 한다. 이 문제들은 VPR의 성능을 저해하는 주된 요인으로 작용하며, 이를 해결하는 것이 VPR 연구의 핵심 목표가 된다.

첫째, **지각적 모호성(Perceptual Aliasing)**은 서로 다른 장소가 시각적으로 매우 유사하게 보이는 현상을 의미한다.9 예를 들어, 현대적인 도시의 실내 복도, 사무실, 혹은 고속도로 주변 풍경은 구조적으로 반복되는 요소가 많아 시각적으로 구별하기 어렵다. 이로 인해 VPR 시스템은 지리적으로는 멀리 떨어진 장소를 동일한 곳으로 오인하는 심각한 오류를 범할 수 있다.10

둘째, **상태 불변성(Condition Invariance)**은 동일한 장소라도 조명(주간/야간), 날씨(맑음/흐림/비/눈), 계절(봄/여름/가을/겨울)의 변화에 따라 외형이 극심하게 변하는 문제를 지칭한다.4 특히 로봇이나 자율주행차가 장기간에 걸쳐 운용될 때, 이러한 환경 변화는 VPR 시스템의 실패를 유발하는 가장 큰 원인 중 하나로 꼽힌다.9

셋째, **시점 불변성(Viewpoint Invariance)**은 본 보고서에서 중점적으로 다루는 문제로, 데이터베이스 이미지와 쿼리 이미지 간의 촬영 시점 차이로 인해 발생하는 인식의 어려움을 의미한다. 대규모 VPR 데이터베이스는 주로 차량 전방에 장착된 카메라로 구축되는 반면 2, 실제 사용자가 입력하는 쿼리 이미지는 보도에서 스마트폰으로 촬영되거나 2, 드론으로 촬영되는 등 매우 다른 시점을 가질 수 있다. 이처럼 극심한 시점 변화가 발생하면, 두 이미지에서 실제로 겹치는 영역(예: 건물의 일부)이 주는 유사성 신호가 겹치지 않는 영역(예: 도로, 하늘, 주변 차량)이 주는 비유사성 신호에 의해 압도당하는 근본적인 문제가 발생한다.12 EigenPlaces는 바로 이 시점 불변성 문제를 해결하기 위한 독창적인 훈련 패러다임을 제안하며 등장했다.1


VPR 기술은 지난 십여 년간 딥러닝의 발전과 함께 괄목할 만한 진화를 거듭해왔다. 초기 연구들은 SIFT(Scale-Invariant Feature Transform)나 SURF(Speeded Up Robust Features)와 같은 수작업 기반의 지역 특징점(local features)을 추출하고, 이를 Bag-of-Words(BoW)나 VLAD(Vector of Locally Aggregated Descriptors)와 같은 기법으로 집계하여 하나의 전역 기술자를 생성하는 방식을 주로 사용했다.4

이러한 흐름에 혁신을 가져온 것은 2016년에 발표된 **NetVLAD**이다.5 NetVLAD는 기존의 VLAD 집계 방식을 미분 가능한 레이어(differentiable layer)로 재설계하여 심층 컨볼루션 신경망(CNN) 아키텍처에 완벽하게 통합했다. 이를 통해 특징 추출부터 집계까지의 전 과정을 종단간(end-to-end)으로 학습하는 것이 가능해졌다. NetVLAD는 GPS 정보만을 활용한 약한 지도 학습(weakly supervised learning) 방식과 삼중항 손실(triplet loss)을 결합하여, 별도의 강한 레이블 없이도 시점과 조명 변화에 강인한 이미지 표현을 학습하는 데 성공하며 VPR 분야의 기념비적인 모델로 자리 잡았다.13

그러나 NetVLAD가 채택한 삼중항 손실과 같은 미터법 학습(metric learning) 방식은 '하드 네거티브 마이닝(hard negative mining)'이라는 과정에 의존한다. 이는 쿼리와 유사하지만 다른 장소의 이미지(hard negative)를 효과적으로 찾아내어 모델이 미세한 차이를 학습하도록 하는 과정인데, 데이터셋의 크기가 커질수록 적절한 네거티브 샘플을 찾는 데 드는 계산 비용이 기하급수적으로 증가하는 확장성 문제를 내포하고 있었다.15

이러한 한계를 극복하기 위해, **CosPlace** 2를 필두로 한 **분류 기반 패러다임**이 새롭게 부상했다. 이 접근법은 VPR 문제를 수백만 개의 클래스를 가진 얼굴 인식(face recognition) 문제와 유사한 대규모 분류(classification) 문제로 재정의한다. 각 지리적 위치를 하나의 '클래스'로 간주하고, 모델이 이미지를 올바른 클래스로 분류하도록 학습시키는 것이다. 이 방식은 샘플 간의 직접적인 비교를 통한 마이닝 과정이 필요 없기 때문에 계산적으로 훨씬 효율적이며, 수천만 장 규모의 거대 데이터셋으로의 확장을 가능하게 했다. 이러한 패러다임의 변화는 SF-XL과 같은 대규모 데이터셋의 등장이 있었기에 가능했으며, VPR 연구의 초점을 미터법 학습에서 대규모 분류로 전환시키는 계기가 되었다. EigenPlaces는 바로 이 분류 기반 패러다임의 연장선상에서, 훈련 데이터의 구성을 혁신함으로써 시점 강인성 문제를 해결하고자 하는 직계 후손이라 할 수 있다.2



EigenPlaces 방법론의 근간에는 기존 VPR 연구의 흐름을 역행하는 대담하고 독창적인 가설이 자리 잡고 있다. 이전의 주요 연구들은 모델이 장소를 더 잘 구별하도록 하기 위해, 훈련 데이터 클래스 내의 시각적 변화를 의도적으로 최소화하는 방향으로 설계되었다. 예를 들어, NetVLAD는 삼중항 손실 학습 시 쿼리와 가장 시각적으로 유사한 긍정 샘플(hard positive)을 찾아 사용함으로써 시점 변화가 거의 없는 쌍으로 학습을 진행했다. CosPlace 역시 동일한 클래스 내의 이미지들이 거의 같은 방향을 바라보도록 데이터를 구성하여 클래스 내 분산(intra-class variance)을 줄이는 데 초점을 맞췄다.17

EigenPlaces는 이러한 철학을 정면으로 뒤집는다. 모델이 테스트 시에 마주할 극심한 시점 변화에 강인해지기 위해서는, 훈련 단계에서부터 의도적으로 '시점 다양성'에 노출되어야 한다는 것이 EigenPlaces의 핵심 가설이다. 즉, 동일한 장소를 매우 다른 여러 각도에서 촬영한 이미지들을 하나의 클래스로 묶어 모델에 제시함으로써, 모델이 시점의 변화에도 불구하고 장소의 본질적인 특징을 포착하는 불변의(invariant) 전역 기술자를 학습하도록 강제하는 것이다.1 이는 모델 아키텍처의 변경이 아닌, 훈련 데이터의 체계적인 구성을 통해 원하는 불변성(invariance)을 기술자에 직접 '주입(embed)'하려는 데이터 중심적(data-centric) 접근법의 정수라 할 수 있다.


EigenPlaces 방법론의 가장 혁신적인 부분은 별도의 수동 레이블링이나 외부 정보 없이, 오직 이미지에 포함된 지리 정보(GPS 좌표)만을 이용하여 시점 변화가 풍부한 훈련 클래스를 완전 자동으로 생성한다는 점이다.2 이 과정은 다음과 같은 4단계의 'Eigen-Clustering' 메커니즘을 통해 이루어진다.

1단계: 지리적 분할 (Geographic Partitioning)

먼저, SF-XL과 같은 대규모 훈련 데이터셋을 지리적 좌표에 기반하여 작은 셀(cell) 단위로 분할한다. 이는 CosPlace에서 제안된 방식과 유사하며, 이 단계에서 생성된 각 셀을 'CosPlace 그룹'이라고 칭한다.18 이 과정은 지리적으로 인접한 이미지들을 거칠게 그룹화하여, 후속 단계에서 분석할 대상의 범위를 한정하는 역할을 한다.

2단계: 공간 분포의 주성분 분석 (Principal Component Analysis of Spatial Distribution)

각각의 CosPlace 그룹에 대해, 그룹 내에 포함된 모든 파노라마 이미지들의 카메라 위치 좌표(UTM 좌표) 집합을 대상으로 주성분 분석(Principal Component Analysis, PCA)을 수행한다. PCA는 데이터의 분산이 가장 큰 방향을 순서대로 찾아내는 통계적 기법이다.

- **첫 번째 주성분 (1st Principal Component):** 데이터의 분산이 가장 큰 방향을 나타내는 고유벡터(eigenvector)이다. SF-XL 데이터셋처럼 차량이 도로를 따라 이동하며 촬영한 경우, 이 첫 번째 주성분은 자연스럽게 '도로의 진행 방향'에 해당하게 된다.

- 두 번째 주성분 (2nd Principal Component): 첫 번째 주성분에 직교하면서, 나머지 분산이 가장 큰 방향을 나타낸다. 이는 도로에 수직인 방향, 즉 '도로변의 건물이나 상점을 향하는 방향'을 의미하게 된다.

  이처럼 EigenPlaces는 순수하게 기하학적인 위치 정보의 분포만으로 장소의 의미론적 방향성(semantic direction)을 영리하게 추론한다.

3단계: 가상 카메라 배치를 통한 이미지 생성 (Image Generation via Virtual Camera Placement)

PCA를 통해 얻은 두 주성분 벡터를 이용하여, 각 그룹의 중심점을 바라보는 다양한 시점의 이미지를 가상으로 생성한다.

- **정면 뷰 (Frontal Views):** 그룹의 중심점에서 첫 번째 주성분 벡터(도로 방향)를 따라 양방향으로 일정 거리만큼 떨어진 위치에 가상의 카메라를 배치한다. 그리고 해당 위치에서 그룹의 중심점을 바라보도록 원본 파노라마 이미지를 잘라내어(crop) '정면 뷰' 이미지들을 생성한다.
- **측면 뷰 (Lateral Views):** 동일한 방식으로, 두 번째 주성분 벡터(도로변 방향)를 따라 가상의 카메라를 배치하고 중심점을 바라보도록 파노라마 이미지를 잘라내어 '측면 뷰' 이미지들을 생성한다.

4단계: 최종 클래스 구성 (Final Class Composition)

마지막으로, 3단계에서 생성된 모든 정면 뷰와 측면 뷰 이미지들을 하나의 훈련 클래스, 즉 'EigenPlaces 클래스'로 구성한다. 이렇게 만들어진 각 클래스는 동일한 지점(그룹의 중심)을 도로를 따라오는 시점과 도로변에서 바라보는 시점 등, 매우 다른 여러 각도에서 촬영한 이미지들의 집합이 된다. 모델은 이러한 클래스 데이터를 학습함으로써, 극심한 시점 변화에도 흔들리지 않는 강인한 장소 표현을 내재화하게 된다.2 이 방법의 독창성은 별도의 감독 없이도 시점 다양성을 극대화하는 훈련 데이터를 생성했다는 점에 있지만, 이는 동시에 훈련 데이터의 수집 방식이 도로와 같은 정형적인 구조를 따라야 한다는 근본적인 제약을 내포하기도 한다.


EigenPlaces의 핵심인 Eigen-Clustering 메커니즘은 주성분 분석(PCA)에 수학적 기반을 두고 있다. 특정 CosPlace 그룹 내에 포함된 $n$개의 카메라 위치 좌표 집합을 데이터 행렬 $X \in \mathbb{R}^{n \times p}$ (여기서 $p$는 좌표의 차원, 예: UTM의 경우 2)로 표현할 수 있다. PCA의 목표는 데이터의 분산을 최대화하는 새로운 직교 기저(orthogonal basis)를 찾는 것이다.

이를 위해 먼저 데이터의 공분산 행렬(covariance matrix) $S$를 계산한다. 데이터의 평균이 0으로 중심화(centered)되었다고 가정할 때, 공분산 행렬은 다음과 같이 정의된다 20:
$$
S = \frac{1}{n-1} X^T X
$$
공분산 행렬 $S$는 $p \times p$ 크기의 대칭 행렬이며, 이 행렬에 대한 고유값 분해(eigendecomposition)를 수행하는 것이 PCA의 핵심이다. 이는 다음의 고유값 방정식(eigenvalue equation)을 푸는 것과 같다 20:
$$
S \mathbf{v} = \lambda \mathbf{v}
$$
여기서 $\lambda$는 고유값(eigenvalue)이고, $\mathbf{v}$는 그에 해당하는 고유벡터(eigenvector)이다. 고유값 $\lambda$는 해당 고유벡터 방향으로 데이터가 갖는 분산의 크기를 나타낸다. 따라서 가장 큰 고유값 $\lambda_1$에 해당하는 고유벡터 $\mathbf{v}_1$이 데이터의 분산이 가장 큰 방향, 즉 첫 번째 주성분이 된다. EigenPlaces는 이 $\mathbf{v}_1$을 '도로 방향'으로, 그리고 두 번째로 큰 고유값 $\lambda_2$에 해당하는 고유벡터 $\mathbf{v}_2$를 '도로변 방향'으로 활용하여 가상 뷰를 생성한다. 이 방법론의 명칭 'EigenPlaces'는 단순히 고유벡터를 사용했다는 의미를 넘어, 2010년경 도시 컴퓨팅 분야에서 통신 데이터의 고유벡터를 분석하여 도시 공간의 기능적 특성을 정의한 'Eigenplaces' 연구에 대한 오마주로 해석될 수 있다.24 당시 연구가 시간의 흐름에 따른 인간 활동 패턴의 주성분을 찾았다면, 2023년의 EigenPlaces는 공간 내 카메라 위치 분포의 주성분을 찾아 장소를 정의함으로써 그 개념을 시각적 인식의 영역으로 확장시킨 것이다.

한편, VPR 파이프라인의 후반부에서는 기술자 정규화(descriptor whitening)가 중요한 역할을 한다. 이는 추출된 기술자 벡터의 각 차원이 서로 통계적으로 독립적이고(uncorrelated) 동일한 분산을 갖도록 변환하는 과정으로, 유클리드 거리나 코사인 유사도 기반의 검색 성능을 향상시킨다.27 PCA를 이용한 정규화(PCA whitening)는 널리 사용되는 기법 중 하나이며, 다음과 같은 수식으로 표현된다 23:
$$
x_{PCAwhite} = \text{diag}(1./\sqrt{\text{diag}(\Lambda) + \epsilon}) \cdot V^T \cdot x
$$
여기서 $x$는 원본 기술자 벡터, $V$는 공분산 행렬의 고유벡터들을 열로 갖는 행렬, $\Lambda$는 해당 고유값들을 대각 원소로 갖는 대각 행렬, 그리고 $\epsilon$은 0으로 나누는 것을 방지하기 위한 작은 상수이다. 이 변환을 통해 기술자는 통계적으로 더 잘 분리되는 특성을 갖게 되어 검색 정확도를 높이는 데 기여한다.



EigenPlaces의 성능을 검증하기 위해, 저자들은 VPR 연구 역사상 가장 포괄적인 수준의 벤치마크 실험을 수행했다고 주장한다.2 사용된 데이터셋은 훈련용과 평가용으로 나뉘며, 각각의 특성은 EigenPlaces의 성능을 다각도로 분석하는 데 중요한 역할을 한다.

**훈련 데이터셋:**

- **SF-XL (San Francisco eXtra Large):** EigenPlaces의 훈련을 위해 사용된 핵심 데이터셋이다.17 샌프란시스코 전역을 12년에 걸쳐 촬영한 4,100만 개 이상의 이미지를 포함하는 방대한 규모를 자랑한다. 이 데이터셋의 가장 중요한 특징은 장소를 조밀하게 커버하는 360도 파노라마 이미지와 정확한 카메라 방향(heading) 메타데이터를 제공한다는 점이다. 이 정보는 EigenPlaces가 PCA를 통해 의미 있는 주성분을 추출하고, 가상의 정면/측면 뷰를 생성하는 데 필수적이다. 따라서 SF-XL과 같은 특성을 가진 데이터셋의 존재가 EigenPlaces 방법론의 전제 조건이 된다.29

평가 데이터셋:

평가 데이터셋은 크게 두 가지 유형으로 나눌 수 있다.

- **정면 뷰 위주 데이터셋 (Frontal-View Dominant Datasets):**
  - **Pitts30k/250k:** VPR 분야에서 가장 널리 사용되는 표준 벤치마크로, 피츠버그 시내의 Google Street View 이미지로 구성되어 있다. 데이터베이스와 쿼리가 서로 다른 해에 촬영되어 장기적인 변화에 대한 강인성을 평가할 수 있다.17
  - **MSLS (Mapillary Street-Level Sequences):** 여러 도시의 도로 레벨 이미지 시퀀스로 구성된 대규모 데이터셋이다. 주로 차량 전방 시점의 이미지로 이루어져 있어, 전통적인 VPR 시나리오에서의 성능을 측정하는 데 사용된다.17
- **다양한 시점 및 조건 데이터셋 (Multi-View and Challenging Conditions Datasets):**
  - **Tokyo 24/7:** 도쿄 도심의 105개 장소에 대해 주간, 일몰, 야간에 스마트폰으로 촬영한 쿼리 이미지로 구성되어, 극심한 조명 변화와 다양한 시점을 동시에 평가하는 어려운 데이터셋이다.17
  - **SF-XL test v1:** 이미지 공유 플랫폼 Flickr에서 수집한 1,000개의 이미지로 구성된 쿼리셋이다. 야간 이미지, 보도에서 촬영된 측면 뷰, 비전문가가 촬영한 구도 등 현실 세계의 예측 불가능한 시점 변화를 다수 포함하고 있어 EigenPlaces의 강점을 시험하기에 적합하다.17
  - **Nordland:** 노르웨이의 철도를 따라 봄, 여름, 가을, 겨울 네 계절에 걸쳐 촬영된 이미지로 구성되어, 극심한 계절 변화에 대한 모델의 강인성을 평가한다.17
  - **Oxford RobotCar:** 맑음, 흐림, 비, 눈, 야간 등 다양한 날씨와 조명 조건에서 촬영된 이미지로 구성되어, 상태 불변성(condition invariance)을 종합적으로 평가한다.17


EigenPlaces의 성능은 상위 N개의 검색 결과에 정답이 포함될 확률을 의미하는 Recall@N (R@N) 지표를 통해 주요 VPR 모델들과 비교되었다. 아래 표는 여러 연구에서 보고된 성능 수치를 종합하여 주요 벤치마크에 대한 성능을 비교한 것이다.

**표 1: 주요 VPR 벤치마크 성능 비교 (R@1 / R@5)**

| 방법론             | 백본     | 기술자 차원 | 훈련 데이터셋 | Pitts250k       | Pitts30k        | Tokyo 24/7      | SF-XL-testv1    |
| ------------------ | -------- | ----------- | ------------- | --------------- | --------------- | --------------- | --------------- |
| NetVLAD 30         | VGG16    | 32768       | Pitts250k     | 81.7 / 90.8     | 81.3 / 90.7     | 58.4 / 72.4     | - / -           |
| CosPlace 30        | ResNet50 | 512         | SF-XL         | 90.4 / 96.6     | 89.6 / 94.9     | 76.5 / 89.2     | 64.8 / 73.1     |
| MixVPR 30          | ResNet50 | 4096        | GSV-Cities    | 94.3 / 98.2     | 91.6 / 95.5     | 87.0 / 93.3     | 69.2 / 77.4     |
| CricaVPR 30        | ResNet50 | 4096        | GSV-Cities    | 94.3 / 98.6     | 91.3 / 96.0     | 85.1 / 91.7     | 56.6 / 69.9     |
| **EigenPlaces** 30 | ResNet50 | 512         | SF-XL         | **93.9 / 98.0** | **92.3 / 96.1** | **84.8 / 94.0** | **83.8 / 89.6** |
| **MVC-VPR** 30     | ResNet50 | 512         | SF-XL         | 92.1 / 97.7     | 89.6 / 96.2     | **91.1 / 97.1** | 77.0 / 84.6     |

주: MVC-VPR은 EigenPlaces 이후 제안된 방법론으로, 비교를 위해 포함되었다. 성능 수치는 30에서 인용되었으며, 백본과 기술자 차원은 일반적인 설정을 따랐다.

결과 분석을 통해 몇 가지 중요한 경향성을 파악할 수 있다.

- **시점 변화가 큰 데이터셋에서의 압도적 성능:** EigenPlaces의 진정한 강점은 **SF-XL-testv1** 데이터셋에서 명확하게 드러난다. R@1에서 83.8%를 기록하며, 동일한 조건에서 훈련된 CosPlace(64.8%)나 다른 데이터셋으로 훈련된 MixVPR(69.2%)을 압도적인 차이로 능가한다.30 이는 보행자 시점과 같은 비정형적인 쿼리가 많은 데이터셋에서 EigenPlaces의 훈련 방식이 매우 효과적임을 직접적으로 증명한다.
- **전통적 벤치마크에서의 경쟁력:** Pitts250k/30k와 같은 전통적인 정면 뷰 위주 데이터셋에서도 EigenPlaces는 SOTA 수준의 높은 성능을 보인다. 특히 기술자 차원이 8배나 큰 MixVPR과 대등하거나 더 나은 성능을 보이는 점은 주목할 만하다.30 이는 EigenPlaces가 특정 유형의 데이터셋에만 국한되지 않는 범용적인 강인성을 갖추었음을 시사한다.
- **"One-win-all" 솔루션의 부재:** 저자들이 인정한 바와 같이, EigenPlaces가 모든 데이터셋에서 최고의 성능을 보이는 것은 아니다.2 예를 들어, Tokyo 24/7 데이터셋에서는 이후에 제안된 MVC-VPR이 더 높은 성능을 기록했다.30 또한, 랜드마크 검색에 가까운 R-Oxford나 R-Paris 데이터셋에서는 MegaLoc과 같은 다른 모델이 더 나은 성능을 보이기도 했다.8 이는 데이터셋의 특성과 '장소'의 정의에 따라 최적의 VPR 솔루션이 달라질 수 있음을 보여준다.

결론적으로, EigenPlaces의 성능은 벤치마크 데이터셋이 내포한 '시점 격차(viewpoint gap)'와 직접적인 상관관계를 보인다. 쿼리와 데이터베이스 간의 시점 차이가 클수록, 시점 다양성을 학습한 EigenPlaces의 성능 우위가 더욱 두드러진다.


EigenPlaces는 훈련 과정에서 정면 뷰(Frontal)와 측면 뷰(Lateral) 이미지로 구성된 두 개의 서브 배치를 활용하여 손실을 계산한다. 이 두 가지 시점 정보가 최종 성능에 각각 어떻게 기여하는지 분석하기 위해, 저자들은 손실 함수의 각 구성 요소를 제거해보는 Ablation Study를 수행했다.2 실험은 ResNet-18 백본과 512차원 기술자를 사용하여 진행되었다.

**표 2: EigenPlaces 손실 함수 Ablation Study (R@1)**

| 손실 함수 구성 요소            | Pitts30k | Tokyo 24/7 | MSLS Val | St Lucia | 평균     |
| ------------------------------ | -------- | ---------- | -------- | -------- | -------- |
| 측면 손실(Lateral Loss)만 사용 | 90.2     | 80.0       | 83.1     | 97.3     | 87.6     |
| 정면 손실(Frontal Loss)만 사용 | 89.5     | 78.1       | 85.8     | 99.3     | 88.2     |
| **둘 다 사용 (EigenPlaces)**   | **90.5** | **82.2**   | **86.2** | **99.0** | **89.5** |

주: 성능 수치는 2에서 인용되었다.

이 실험 결과는 매우 명확한 결론을 제시한다.

- **각 요소의 독립적 효과:** 정면 손실만 사용하거나 측면 손실만 사용해도 이미 높은 수준의 성능을 달성할 수 있다. 특히 측면 손실만으로도 Pitts30k에서 90.2%의 R@1을 기록한 것은, 측면 뷰 학습이 정면 뷰 인식에도 긍정적인 영향을 미치는 일반화 능력을 부여함을 시사한다.
- **시너지 효과:** 두 손실 함수를 함께 사용했을 때 모든 데이터셋에서 전반적으로 성능이 향상되었으며, 평균 R@1이 가장 높게 나타났다. 특히 주목할 점은 시점 변화가 가장 극심한 **Tokyo 24/7** 데이터셋에서 성능 향상 폭(각각 +2.2%p, +4.1%p)이 가장 컸다는 것이다. 이는 정면 뷰와 측면 뷰를 함께 학습하는 것이 서로 다른 시점 정보를 보완하고 연결하는 능력을 모델에 부여하여, 결과적으로 시점 변화에 대한 강인성을 극대화하는 시너지 효과를 창출함을 강력하게 뒷받침한다. 이 Ablation Study는 EigenPlaces의 복잡한 데이터 생성 과정이 불필요한 기교가 아니며, 각 구성 요소가 최종 성능에 유의미하게 기여함을 증명한다.


EigenPlaces의 기여는 단순히 정확도 향상에만 그치지 않는다. 대규모 VPR 시스템의 현실적인 배포 가능성과 직결되는 '효율성' 측면에서도 중요한 진전을 이루었다. 저자들은 EigenPlaces가 기존 SOTA 모델 대비 **60% 적은 GPU 메모리**로 훈련 가능하며, **50% 더 작은 기술자**를 사용한다고 주장한다.1

이러한 효율성은 여러 요인에서 비롯된다.

- **훈련 메모리 효율성:** EigenPlaces는 CosPlace와 마찬가지로 분류 기반 학습 패러다임을 채택한다. 이는 삼중항 손실 기반의 미터법 학습이 요구하는 계산 비용이 높은 하드 네거티브 마이닝 과정을 생략할 수 있게 해준다. 따라서 더 작은 배치 크기로도 안정적인 학습이 가능하며, 이는 곧 훈련에 필요한 GPU VRAM 요구량을 크게 낮추는 결과로 이어진다. 실제로 ResNet-18 백본과 512차원 기술자를 사용하는 기본 설정은 8GB 미만의 VRAM으로 훈련이 가능하다고 명시되어 있어 18, 상대적으로 저사양의 하드웨어에서도 연구 및 개발이 가능하게 하는 접근성을 제공한다.
- **기술자 크기 효율성:** 표 1에서 볼 수 있듯이, EigenPlaces는 주로 512차원의 비교적 저차원 기술자를 사용한다.16 이는 4096차원의 MixVPR이나 32,768차원의 NetVLAD와 비교했을 때 매우 작은 크기이다.16 기술자의 크기는 실제 시스템에서 두 가지 중요한 영향을 미친다. 첫째, 데이터베이스 전체의 인덱스 크기를 결정한다. 기술자가 작을수록 더 적은 저장 공간을 차지한다. 둘째, 검색 속도에 직접적인 영향을 미친다. 저차원 벡터 공간에서의 최근접 이웃 탐색은 고차원 공간보다 훨씬 빠르다. 따라서 EigenPlaces는 더 작은 기술자로 동등하거나 더 나은 성능을 달성함으로써, 수백만, 수천만 장의 이미지를 다루는 대규모 VPR 시스템의 저장 비용과 쿼리 지연 시간(latency)을 획기적으로 줄일 수 있는 실용적인 이점을 제공한다. 이러한 효율성은 EigenPlaces를 학술적인 성과를 넘어 실제 자율주행차나 모바일 로봇과 같은 자원 제약적인 환경에 배포할 수 있는 유력한 후보로 만든다.



EigenPlaces는 VPR 분야에 몇 가지 중요한 기여를 남겼으며, 그 강점은 독창적인 아이디어와 실용성에 있다.

첫째, **독창적인 훈련 패러다임을 제시했다.** 기존 연구들이 모델 아키텍처를 개선하거나 손실 함수를 변경하는 데 집중했던 것과 달리, EigenPlaces는 문제의 근원으로 돌아가 '훈련 데이터 자체의 구성'을 통해 시점 강인성이라는 목표를 달성했다. '시점 다양성'을 의도적으로 주입하여 모델이 불변성을 학습하도록 강제하는 이 데이터 중심적 접근법은 VPR 연구에 새로운 방향을 제시했으며, 특히 추가적인 감독 정보 없이 오직 지리적 좌표만으로 이를 자동화했다는 점에서 높은 평가를 받는다.2 이는 VPR 분야에서 데이터 큐레이션의 중요성을 재조명하는 계기가 되었다.

둘째, **실용성과 효율성을 입증했다.** 앞서 분석했듯이, EigenPlaces는 더 적은 GPU 메모리로 훈련이 가능하고, 훨씬 작은 크기의 기술자로도 SOTA 수준의 성능을 달성했다.1 이는 학술적 성과가 실제 산업 현장에 적용되기 위해 반드시 갖추어야 할 중요한 덕목이다. 연구실 수준을 넘어 대규모 상용 서비스나 자율주행 시스템과 같은 자원 제약적인 환경에 VPR 기술을 배포하고자 할 때, EigenPlaces의 효율성은 다른 복잡한 모델들에 비해 강력한 경쟁 우위를 제공한다.

셋째, **공정한 평가 프레임워크를 커뮤니티에 제공했다.** 저자들은 논문과 함께 $VPR-methods-evaluation$이라는 표준화된 벤치마크 리포지토리를 공개했다.18 이 리포지토리는 NetVLAD, CosPlace, MixVPR 등 여러 주요 VPR 모델의 공식 가중치를 자동으로 다운로드하여 동일한 환경에서 평가를 수행할 수 있도록 지원한다. 이는 연구 재현성을 높이고 모델 간의 공정한 비교를 가능하게 하는 매우 중요한 커뮤니티 기여이다. 이러한 노력은 새로운 연구자들이 VPR 분야에 진입하는 장벽을 낮추고, 실험을 가속화하며, EigenPlaces를 VPR 연구의 중심적인 참조점으로 자리매김하게 했다. 이 프레임워크의 영향력은 EigenPlaces 모델 자체의 SOTA 성능보다 더 오래 지속될 수 있다.


혁신적인 기여에도 불구하고 EigenPlaces는 몇 가지 명백한 한계와 제약 조건을 가지고 있다.

첫째, **훈련 데이터에 대한 강한 의존성**을 갖는다. EigenPlaces의 Eigen-Clustering 메커니즘은 훈련 데이터가 특정 지역을 조밀하게 커버하는 360도 파노라마 이미지와 정확한 방향 메타데이터를 포함하고 있을 것을 전제로 한다. 현재로서는 SF-XL 데이터셋이 이러한 조건을 충족하는 거의 유일한 대규모 데이터셋이다.29 만약 이러한 유형의 데이터셋을 사용할 수 없다면, EigenPlaces의 훈련 방법론을 직접 적용하는 것은 사실상 불가능하다. 이는 모델의 범용성을 제한하고 다른 종류의 데이터셋으로의 확장을 어렵게 만드는 가장 큰 제약 조건이다.

둘째, **데이터의 공간 분포에 대한 암묵적인 가정**이 존재한다. PCA 기반의 클러스터링은 카메라의 위치가 도로를 따라 선형적으로 분포하고, 건물들이 그에 수직으로 배치되어 있다는 도시 환경의 기하학적 패턴을 암묵적으로 가정한다. 이러한 가정은 SF-XL과 같은 데이터셋에서는 잘 작동하지만, 광장, 공원, 혹은 비정형적인 골목길과 같이 카메라 위치 분포가 복잡한 환경에서는 PCA로 추출된 주성분이 의미 있는 '정면'과 '측면' 뷰를 대표하지 못할 수 있다. 이 경우, 생성된 훈련 클래스는 의도한 시점 다양성을 담지 못하고 성능 저하로 이어질 수 있다.

셋째, **파운데이션 모델 시대에서의 위상**에 대한 질문이 제기된다. EigenPlaces가 발표된 ICCV 2023을 전후로, 컴퓨터 비전 분야는 DINOv2와 같은 거대 비전 파운데이션 모델(foundation model)이 지배하는 시대로 빠르게 전환되고 있다.29 이러한 모델들은 웹 스케일의 방대한 데이터로 사전 학습되는 과정에서 이미 상당한 수준의 시점, 조명, 외형 불변성을 내재하고 있다. 따라서 파운데이션 모델을 백본으로 사용하면, EigenPlaces와 같이 정교하고 복잡한 훈련 데이터 엔지니어링을 거치지 않고도 높은 수준의 강인성을 달성할 수 있다. 이런 맥락에서 EigenPlaces는 특정 불변성 문제를 데이터 큐레이션을 통해 해결하려는 접근법의 정점을 보여주는 동시에, 패러다임이 파운데이션 모델로 전환되기 직전의 마지막 세대 기술로 평가될 수도 있다.


EigenPlaces가 제시한 아이디어와 남긴 한계는 VPR 분야의 흥미로운 미래 연구 방향을 제시한다.

첫째, **방법론의 일반화** 연구가 필요하다. EigenPlaces의 핵심 철학인 '시점 다양성 주입'을 파노라마 데이터셋에 의존하지 않고 일반적인 이미지 데이터셋에서도 적용할 수 있는 방법을 개발하는 것이 중요하다. 예를 들어, NeRF(Neural Radiance Fields)나 확산 모델(Diffusion Models)과 같은 생성 모델을 활용하여 특정 장소의 새로운 시점 이미지를 합성하고, 이를 훈련 데이터에 추가하는 방식이 유망한 대안이 될 수 있다.33

둘째, **파운데이션 모델과의 결합**은 필연적인 수순이다. 파운데이션 모델이 제공하는 강력한 범용 특징 표현 능력과 EigenPlaces의 데이터 큐레이션 전략을 결합하는 하이브리드 접근법이 가능하다. 예를 들어, 파운데이션 모델을 특정 VPR 작업에 맞게 파인튜닝(fine-tuning)할 때, EigenPlaces 방식으로 구성된 데이터를 사용하면 사전 학습된 강인성을 유지하면서도 시점 불변성을 더욱 강화할 수 있을 것이다.32

셋째, **다중 모달리티(Multi-modality)로의 확장**은 VPR의 새로운 지평을 열 것이다. 미래의 자율 에이전트는 시각 정보뿐만 아니라, 이미지 속 텍스트(표지판, 상점 이름), 3D 도시 메쉬(mesh) 데이터, LiDAR 포인트 클라우드, 오디오 등 다양한 소스로부터 장소를 인식해야 한다.35 EigenPlaces의 공간적 분포를 분석하여 의미 있는 정보를 추출하는 아이디어를 이러한 다중 모달 데이터에 적용하여, 더욱 풍부하고 강인한 장소 표현을 학습하는 연구가 기대된다.


EigenPlaces는 VPR 분야의 고질적인 난제였던 시점 변화 문제에 대해, 모델 아키텍처나 손실 함수가 아닌 '훈련 데이터의 체계적인 구성'이라는 데이터 중심적 해법을 제시한 독창적이고 영향력 있는 연구이다. 이는 VPR 연구 커뮤니티의 관점을 넓히고, 문제 해결을 위한 새로운 접근법을 탐색하도록 자극하는 중요한 전환점을 제공했다.

비록 파노라마 데이터에 대한 의존성과 같은 명백한 한계를 가지고 있지만, EigenPlaces는 특정 불변성을 모델에 학습시키기 위해 훈련 데이터를 어떻게 지능적으로 설계하고 재구성해야 하는지에 대한 깊은 통찰을 남겼다. 또한, 뛰어난 성능과 더불어 훈련 및 추론 과정에서의 높은 효율성을 동시에 달성함으로써, 대규모 VPR 시스템의 실용화 가능성을 한 단계 끌어올렸다. 결론적으로 EigenPlaces는 시점 강인성이라는 VPR의 핵심 과제에 대한 효과적인 해결책을 제시했을 뿐만 아니라, 그 과정에서 데이터의 중요성과 공정한 평가의 가치를 다시 한번 일깨워준, VPR 연구 역사에 중요한 족적을 남긴 논문으로 평가될 것이다.3


1. EigenPlaces: Training Viewpoint Robust Models for Visual Place Recognition - Semantic search for arXiv papers with AI, accessed August 22, 2025, https://axi.lims.ac.uk/paper/2308.10832
2. EigenPlaces: Training Viewpoint Robust Models ... - CVF Open Access, accessed August 22, 2025, https://openaccess.thecvf.com/content/ICCV2023/papers/Berton_EigenPlaces_Training_Viewpoint_Robust_Models_for_Visual_Place_Recognition_ICCV_2023_paper.pdf
3. EigenPlaces: Training Viewpoint Robust Models for Visual Place Recognition - arXiv, accessed August 22, 2025, https://arxiv.org/abs/2308.10832
4. Place Recognition: An Overview of Vision Perspective - MDPI, accessed August 22, 2025, https://www.mdpi.com/2076-3417/8/11/2257
5. NetVLAD: CNN Architecture for Weakly Supervised Place Recognition - CVF Open Access, accessed August 22, 2025, https://openaccess.thecvf.com/content_cvpr_2016/papers/Arandjelovic_NetVLAD_CNN_Architecture_CVPR_2016_paper.pdf
6. MixVPR: Feature Mixing for Visual Place Recognition - CVF Open Access, accessed August 22, 2025, https://openaccess.thecvf.com/content/WACV2023/papers/Ali-bey_MixVPR_Feature_Mixing_for_Visual_Place_Recognition_WACV_2023_paper.pdf
7. Visual Place Recognition: Building an Evaluation Framework for Model Robustness - University of Twente, accessed August 22, 2025, http://essay.utwente.nl/100860/1/Gosa_BA_EEMCS.pdf
8. MegaLoc: One Retrieval to Place Them All - arXiv, accessed August 22, 2025, https://arxiv.org/html/2502.17237v1
9. (PDF) Visual Place Recognition: A Survey - ResearchGate, accessed August 22, 2025, https://www.researchgate.net/publication/284754832_Visual_Place_Recognition_A_Survey
10. CricaVPR: Cross-image Correlation-aware Representation Learning for Visual Place Recognition - arXiv, accessed August 22, 2025, https://arxiv.org/html/2402.19231v2
11. Visual Place Recognition. Introduction | by SOMIK DHAR - Medium, accessed August 22, 2025, https://medium.com/@sd5023/visual-place-recognition-8999307ebb2f
12. Revisit Anything: Visual Place Recognition via Image Segment Retrieval - arXiv, accessed August 22, 2025, https://arxiv.org/abs/2409.18049
13. NetVLAD: CNN architecture for weakly supervised place recognition, accessed August 22, 2025, https://www.di.ens.fr/~josef/publications/Arandjelovic17.pdf
14. NetVLAD: CNN Architecture for Weakly Supervised Place Recognition - Medium, accessed August 22, 2025, https://medium.com/data-science/netvlad-cnn-architecture-for-weakly-supervised-place-recognition-ce64b08bebaf
15. Distributed training of CosPlace for large-scale visual place recognition - Frontiers, accessed August 22, 2025, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2024.1386464/full
16. MVC-VPR: Mutual Learning of Viewpoint Classification and Visual Place Recognition - arXiv, accessed August 22, 2025, https://arxiv.org/html/2412.09199v2
17. EigenPlaces: Training Viewpoint Robust Models for Visual Place Recognition (Supplementary) - CVF Open Access, accessed August 22, 2025, https://openaccess.thecvf.com/content/ICCV2023/supplemental/Berton_EigenPlaces_Training_Viewpoint_ICCV_2023_supplemental.pdf
18. gmberton/EigenPlaces: Official code for ICCV 2023 paper "EigenPlaces: Training Viewpoint Robust Models for Visual Place Recognition" - GitHub, accessed August 22, 2025, https://github.com/gmberton/EigenPlaces
19. ICCV 2023 Open Access Repository, accessed August 22, 2025, https://openaccess.thecvf.com/content/ICCV2023/html/Berton_EigenPlaces_Training_Viewpoint_Robust_Models_for_Visual_Place_Recognition_ICCV_2023_paper.html
20. Principal component analysis: a review and recent developments - PMC - PubMed Central, accessed August 22, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC4792409/
21. The Math Behind PCA • LearnPCA, accessed August 22, 2025, https://bryanhanson.github.io/LearnPCA/articles/Vig_06_Math_Behind_PCA.html
22. The Math of Principal Component Analysis (PCA) | by adam dhalla | Analytics Vidhya, accessed August 22, 2025, https://medium.com/analytics-vidhya/the-math-of-principal-component-analysis-pca-bf7da48247fc
23. PCA Whitening - Unsupervised Feature Learning and Deep Learning Tutorial, accessed August 22, 2025, http://ufldl.stanford.edu/tutorial/unsupervised/PCAWhitening/
24. (PDF) Eigenplaces: Segmenting Space Through Digital Signatures - ResearchGate, accessed August 22, 2025, https://www.researchgate.net/publication/224580673_Eigenplaces_Segmenting_Space_Through_Digital_Signatures
25. Eigenplaces: Segmenting Space through Digital Signatures - MIT Senseable City Lab, accessed August 22, 2025, https://senseable.mit.edu/papers/pdf/20100201_Calabrese_etal_EigenplacesSegmenting_IeeePervasive.pdf
26. Eigenplaces: Segmenting Space through Digital Signatures - King's College London Research Portal, accessed August 22, 2025, https://kclpure.kcl.ac.uk/portal/en/publications/eigenplaces-segmenting-space-through-digital-signatures
27. Whitening transformation - Wikipedia, accessed August 22, 2025, https://en.wikipedia.org/wiki/Whitening_transformation
28. Whitening Techniques for Machine Learning - Number Analytics, accessed August 22, 2025, https://www.numberanalytics.com/blog/whitening-techniques-for-machine-learning
29. EffoVPR: Effective Foundation Model Utilization for Visual Place Recognition - OpenReview, accessed August 22, 2025, https://openreview.net/forum?id=NSpe8QgsCB
30. MVC-VPR: Mutual Learning of Viewpoint Classification and Visual Place Recognition - arXiv, accessed August 22, 2025, https://arxiv.org/html/2412.09199v1
31. GitHub - gmberton/VPR-methods-evaluation: Easily download and evaluate pre-trained Visual Place Recognition methods. Code built for the ICCV 2023 paper "EigenPlaces, accessed August 22, 2025, https://github.com/gmberton/VPR-methods-evaluation
32. Class-Relational Label Smoothing for Lifelong Visual Place Recognition - OpenReview, accessed August 22, 2025, [https://openreview.net/forum?id=ZS1lCBLljq¬eId=EcQFhVfJVh](https://openreview.net/forum?id=ZS1lCBLljq&noteId=EcQFhVfJVh)
33. [2402.17159] NocPlace: Nocturnal Visual Place Recognition via Generative and Inherited Knowledge Transfer - arXiv, accessed August 22, 2025, https://arxiv.org/abs/2402.17159
34. CerfeVPR: Cross-Environment Robust Feature Enhancement for Visual Place Recognition, accessed August 22, 2025, https://www.sciopen.com/article/10.32604/cmc.2025.062834
35. [Literature Review] MeshVPR: Citywide Visual Place Recognition Using 3D Meshes, accessed August 22, 2025, https://www.themoonlight.io/en/review/meshvpr-citywide-visual-place-recognition-using-3d-meshes
36. MMS-VPR: Multimodal Street-Level Visual Place Recognition Dataset and Benchmark, accessed August 22, 2025, https://arxiv.org/html/2505.12254v1


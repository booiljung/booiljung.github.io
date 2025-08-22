---
layout: page
title: SpinNet 3차원 포인트 클라우드 정합
permalink: /sensors/point cloud/registrations/SpinNet
---



3차원 포인트 클라우드 정합(Point Cloud Registration, PCR)은 서로 다른 시점, 시간 또는 센서로부터 취득되어 각기 다른 좌표계를 갖는 복수의 포인트 클라우드 데이터셋을 하나의 일관된 전역 좌표계(global coordinate system)로 정렬하는 핵심적인 기하학적 처리 과정으로 정의된다.1 이 과정의 목표는 소스 포인트 클라우드($P_S$)를 타겟 포인트 클라우드($P_T$)에 최적으로 정렬하는 강체 변환(rigid transformation), 즉 회전 행렬($R$)과 변환 벡터($t$)를 찾는 것이다.

포인트 클라우드 정합 기술의 중요성은 현대 산업 및 연구 분야 전반에 걸쳐 지대하다. 자율주행 시스템에서는 LiDAR 센서로부터 연속적으로 입력되는 포인트 클라우드를 정합하여 차량의 이동 경로를 추정하고(LiDAR Odometry), 주변 환경에 대한 정밀한 3차원 지도를 생성(SLAM)하는 데 필수적으로 사용된다.5 로보틱스 분야에서는 로봇 팔이 작업 대상 객체의 정확한 3차원 자세를 인식하고 정밀한 조작(manipulation)을 수행하기 위해, 센서로 취득한 부분적인 포인트 클라우드를 사전에 정의된 CAD 모델과 정합하는 과정이 요구된다.7 이 외에도, 여러 각도에서 촬영된 단편적인 스캔들을 병합하여 완전한 디지털 모델을 생성하는 3D 재구성 8, 의료 영상 분석, 문화유산의 디지털 보존 등 고부가가치를 창출하는 다양한 응용 분야에서 핵심 기반 기술로서의 역할을 수행한다.2 이처럼 정확한 정합은 분절되고 파편화된 3차원 공간 정보를 완전하고 의미 있는 형태로 통합하는 첫걸음이라 할 수 있다.4


포인트 클라우드 정합 기술의 실용화를 가로막는 기술적 난제는 크게 세 가지로 요약할 수 있다. 첫째, **회전 불변성(Rotation Invariance)** 문제는 가장 근본적인 도전 과제이다. 동일한 객체나 장면이라 할지라도 다른 시점에서 스캔하면 3차원 공간상에서 임의의 회전 변환이 적용된 형태로 관측된다. 효과적인 정합 알고리즘은 이러한 회전 변환에 관계없이 데이터의 내재적 기하학 구조를 일관되게 인식하고 기술(describe)할 수 있어야 한다.8

둘째, 실제 환경에서 수집된 데이터는 필연적으로 **노이즈(noise), 이상치(outliers), 그리고 불균일한 포인트 밀도(non-uniform point density)**를 포함한다. 센서 자체의 측정 오차, 주변 환경의 반사 특성, 스캔 거리의 변화 등 다양한 요인으로 인해 포인트 클라우드는 이상적인 기하학적 표면에서 벗어난 노이즈를 포함하게 되며, 스캔 영역에 따라 포인트의 밀도가 급격하게 변하는 특성을 보인다. 강인한(robust) 정합 알고리즘은 이러한 데이터의 불완전성에도 불구하고 안정적으로 동작해야 한다.4

셋째, **부분 중첩(Partial Overlap)** 문제는 제한된 센서 시야각으로 인해 발생하는 필연적인 제약 조건이다. 두 포인트 클라우드는 전체 형상이 아닌, 일부 영역만이 서로 중첩된다. 따라서 알고리즘은 전체 포인트 클라우드가 아닌, 중첩되는 영역의 대응 관계(correspondence)만을 정확히 찾아내어 변환을 추정해야 한다.4 이 세 가지 난제는 상호 복합적으로 작용하여 정합 문제의 복잡성을 증대시킨다.


포인트 클라우드 정합 문제를 해결하기 위한 초기 연구들은 주로 기하학적 제약에 기반한 전통적 방법론에 집중되었다. 그 대표적인 예가 ICP(Iterative Closest Point) 알고리즘이다.11 ICP는 두 포인트 클라우드 간의 대응점을 가장 가까운 유클리드 거리 기준으로 설정하고, 이 대응점들의 거리 오차를 최소화하는 변환을 반복적으로 계산하여 정합을 수행한다.13 ICP는 구현이 간단하고 좋은 초기 추정값이 주어질 경우 매우 정밀한 결과를 제공하지만, 초기값에 매우 민감하여 잘못된 지역 최적해(local optima)에 수렴하기 쉽다는 치명적인 단점을 안고 있다.4 이러한 한계를 극복하기 위해 FPFH(Fast Point Feature Histograms), SHOT(Signatures of Histograms of OrienTations) 등 포인트의 지역적 기하학 정보를 히스토그램 형태로 인코딩하는 수작업 특징 기술자(Handcrafted Feature Descriptor)들이 제안되었다.8 그러나 이러한 기술자들은 특정 유형의 데이터나 시나리오에 과도하게 최적화되어 있어, 노이즈가 심하거나 기하학적 구조가 복잡한 새로운 환경에서는 표현력이 저하되는 일반화 성능의 한계를 보였다.8

이러한 전통적 방법론의 한계는 딥러닝 기술의 발전과 함께 새로운 패러다임의 전환을 촉발시켰다. 딥러닝 기반 포인트 클라우드 정합(DL-PCR)은 대규모 데이터를 통해 특징 표현 자체를 학습함으로써, 수작업 특징 기술자의 설계에 내재된 경험적 한계를 극복하고자 한다. 딥러닝 모델은 데이터로부터 노이즈, 이상치, 밀도 변화에 강인한 특징을 자동으로 추출하고, 이를 통해 복잡한 실제 환경에 대한 높은 적응력과 일반화 성능을 달성할 수 있다.2 DL-PCR 접근법은 크게 지도 학습(Supervised Learning)과 비지도 학습(Unsupervised Learning)으로 분류된다. 지도 학습은 정답 변환 행렬이 레이블링된 데이터를 사용하여 모델을 직접적으로 최적화하는 방식으로 높은 정확도를 목표로 하지만, 대규모 데이터셋 구축에 많은 비용이 소요된다. 반면, 비지도 학습은 레이블 없이 데이터의 내재적 기하학적 일관성 등을 손실 함수로 활용하여 학습하므로 데이터 유연성이 높다.1


딥러닝 기반 방법론이 큰 성공을 거두었음에도 불구하고, '회전 불변성'이라는 근본적인 문제는 여전히 중요한 도전 과제로 남아있었다. 많은 초기 딥러닝 모델들은 회전 불변성을 확보하기 위해 학습 데이터에 임의의 회전 변환을 적용하여 데이터의 양을 인위적으로 늘리는 데이터 증강(Data Augmentation) 기법에 크게 의존했다.10 그러나 이러한 방식은 3차원 회전 공간인 SO(3)의 연속적인 특성을 이산적인(discrete) 샘플들로 근사하는 것에 불과하여, 학습 데이터에서 보지 못한 새로운 각도의 회전에 대해서는 일반화 성능이 급격히 저하되는 문제를 야기했다.8 다른 일부 연구들은 지역 참조 프레임(Local Reference Frame, LRF)을 먼저 계산하여 입력 데이터를 정규화하려 시도했지만, LRF 계산 자체가 노이즈에 불안정하여 오류를 증폭시키는 원인이 되기도 했다.8

이러한 기술적 공백 속에서, SpinNet은 문제의 본질을 해결하기 위한 새로운 접근법을 제시하며 등장했다. SpinNet의 개발 배경에는 데이터의 양에 의존하는 확률적 해결책이 아닌, 네트워크 아키텍처 자체에 기하학적 원리를 내장하여 결정론적으로 회전 불변성을 보장하려는 시도가 있었다. 즉, SpinNet의 핵심 연구 목표는 데이터 증강이나 불안정한 LRF 계산과 같은 외부적인 기법에 의존하지 않고, 종단간(end-to-end) 학습이 가능한 신경망 구조 설계를 통해 다음 세 가지 핵심 속성을 만족하는 3D 지역 특징 기술자(local feature descriptor)를 학습하는 것이었다 8:

1. **회전 불변성 (Rotation Invariance):** 입력 포인트 클라우드에 어떠한 3차원 회전이 가해지더라도 항상 동일하고 일관된 기술자를 출력한다.
2. **표현력 (Descriptiveness):** 노이즈, 불완전성, 밀도 변화에도 불구하고 지역 표면의 고유한 기하학적 패턴을 구별할 수 있는 풍부한 정보를 담고 있다.
3. **일반화 능력 (Generalization Ability):** 특정 데이터셋(예: 실내 RGB-D 스캔)에서만 학습되었더라도, 이전에 접해보지 못한 완전히 다른 종류의 데이터셋(예: 실외 LiDAR 스캔)에서도 높은 성능을 유지한다.

이러한 목표를 통해 SpinNet은 딥러닝 기반 정합 연구의 방향을 '더 많은 데이터로 학습'하는 것에서 '더 나은 구조로 설계'하는 것으로 전환하는 데 중요한 기여를 했다.



SpinNet 아키텍처의 가장 근간을 이루는 설계 철학은 수작업으로 설계된 특징(handcrafted features)이나 불안정한 지역 참조 프레임(LRF) 계산과 같은 고전적 기법에 대한 의존성을 완전히 배제하는 것이다.8 기존의 많은 하이브리드 접근법들은 LRF를 계산하여 입력 데이터를 정규화하거나, FPFH와 같은 고전적 특징을 초기 입력으로 사용하여 딥러닝 모델의 학습을 보조했다. 그러나 이러한 방식은 LRF 계산 과정 자체의 불안정성이나 수작업 특징이 갖는 표현력의 한계라는 '유리 천장'을 그대로 물려받는 문제를 안고 있었다.8

SpinNet은 이러한 문제점을 근본적으로 해결하기 위해, 오직 미분 가능한 포인트 변환 연산과 간단한 신경망 계층들의 조합만으로 구성된 순수한 종단간(end-to-end) 학습 구조를 채택했다. 이 접근법은 네트워크가 데이터로부터 직접적으로 가장 표현력 있고(representative) 일반화 가능한(general) 특징을 학습하도록 유도한다.8 즉, 회전 불변성이라는 기하학적 문제를 해결하기 위해 사람이 만든 규칙을 사용하는 대신, 그 문제를 해결하는 '절차' 자체를 네트워크 구조에 내장하여 데이터가 이 절차를 자연스럽게 통과하며 불변성을 획득하도록 설계한 것이다. 이는 모델의 성능이 데이터에 내재된 복잡한 패턴을 얼마나 잘 학습하는지에만 의존하게 만들어, 기존 방법론들의 한계를 뛰어넘을 수 있는 기반을 마련했다.


SpinNet의 회전 불변성을 실현하는 핵심 구성 요소는 '공간 포인트 트랜스포머(Spatial Point Transformer)' 모듈이다. 여기서 사용된 '트랜스포머'라는 용어는 자연어 처리(NLP) 분야에서 사용되는 어텐션 기반의 트랜스포머 아키텍처와는 개념적으로 다르며, 입력 데이터의 공간적 배열을 변환하는 모듈을 의미한다.8 이는 2D 이미지의 위치, 크기, 회전 변환을 학습하여 정규화하는 공간 트랜스포머 네트워크(Spatial Transformer Networks, STN)의 개념을 3D 포인트 클라우드 환경에 맞게 독창적으로 재해석하고 적용한 것이다.17 이 모듈은 학습을 통해 변환을 찾는 것이 아니라, 미분 가능한 결정론적 알고리즘을 순차적으로 적용하여 입력된 지역 서피스 패치(local surface patch)를 회전 불변적인 표준 공간(canonical space)으로 매핑하는 역할을 수행한다. 이 과정은 두 단계로 구성된다.


첫 번째 단계는 3차원 회전의 임의성을 제거하기 위해 입력 데이터에 일관된 기준 축을 설정하는 것이다. SpinNet은 이를 위해 주성분 분석(Principal Component Analysis, PCA) 기법을 활용한다.

1. 입력으로 주어진 지역 서피스 패치 내의 모든 포인트들의 3x3 공분산 행렬(covariance matrix)을 계산한다.
2. 이 공분산 행렬에 대한 고유값 분해(eigenvalue decomposition)를 수행하여 3개의 고유값($\lambda_1, \lambda_2, \lambda_3$)과 이에 대응하는 3개의 직교하는 고유벡터($e_1, e_2, e_3$)를 얻는다. 이 고유벡터들은 데이터가 퍼져있는 주된 방향, 즉 주축(principal axes)을 나타낸다.
3. 지역 '서피스' 패치의 특성상, 표면에 수직인 법선 벡터(normal vector) 방향으로는 데이터의 분산이 가장 작을 것이다. 따라서 가장 작은 고유값에 해당하는 고유벡터를 해당 서피스의 대표 법선 벡터로 간주한다.
4. 이 법선 벡터를 새로운 좌표계의 기준 Z축($Z'$)으로 삼아 입력 패치의 모든 포인트를 회전 변환하여 정렬한다.

이 과정을 통해, 입력 패치가 원래 어떤 방향을 향하고 있었는지에 관계없이, 변환 후에는 항상 Z축이 서피스의 법선 방향과 일치하게 된다. 이는 3개의 자유도를 갖는 임의의 3D 회전(SO(3) 그룹) 문제에서 Z축을 중심으로 한 회전이라는 1개의 자유도만 남기는 효과를 가져온다. 즉, 2개의 회전 자유도를 제거하여 문제를 단순화하는 것이다.8


Z축 정렬 후에도 여전히 XY 평면상에서의 회전 모호성이 남아있다. SpinNet은 이 문제를 해결하기 위해 독창적인 2단계 좌표 변환을 수행한다.

1. **구면 좌표계 변환 및 복셀화:** 먼저 Z축 정렬된 포인트들을 원점을 기준으로 구면 좌표계(spherical coordinates)로 변환한다. 이후 이 구면 공간을 고도(elevation)와 방위각(azimuth)을 기준으로 이산적인 격자, 즉 구면 복셀(spherical voxel)로 분할한다.
2. **복셀 내부 정규화:** 각 구면 복셀에 속하는 포인트들의 평균 벡터를 계산한다. 이 평균 벡터가 해당 복셀 내 포인트들의 대표 방향을 나타낸다고 가정하고, 이 벡터를 새로운 기준 X축($X''$) 방향으로 정렬하도록 각 복셀 내부의 포인트들을 회전시킨다.

이 두 번째 변환 단계는 XY 평면상에 남아있던 마지막 회전 자유도를 각 지역 복셀 단위로 정규화하여 제거하는 역할을 한다. 즉, Z축 정렬을 통해 전역적인 기준을 잡고, 구면 복셀화를 통해 지역적인 기준을 다시 한번 잡아줌으로써, 최종적으로 입력 패치가 어떤 회전 상태에 있었더라도 항상 일관된 표준화된 표현으로 변환되도록 보장한다.8 이 전체 과정은 미분 가능하게 설계되어, 종단간 학습 과정에서 손실(loss)이 이 변환 모듈을 통해 역전파(back-propagation)될 수 있다.


공간 포인트 트랜스포머를 통과한 데이터는 완벽한 회전 '불변성(invariance)'을 갖기보다는, Z축 중심의 회전에 대해 '등변성(equivariance)'을 갖는 표현으로 변환된다. 등변성이란 입력에 특정 변환(여기서는 Z축 중심 회전)을 가했을 때, 출력(특징 맵)도 동일한 변환을 겪는 특성을 의미한다. 즉, 입력 포인트 클라우드를 Z축을 중심으로 $\theta$만큼 회전시키면, 공간 포인트 트랜스포머의 출력 또한 Z축을 중심으로 $\theta$만큼 회전된 형태로 나타난다.

이 SO(2) 등변성은 SpinNet 아키텍처에서 매우 중요한 역할을 한다. 만약 이 단계에서 모든 회전 정보를 완전히 제거해버리면(불변성), 후속 신경망 계층들이 학습할 수 있는 유용한 기하학적 정보까지 손실될 위험이 있다. 대신 등변성을 유지함으로써, 회전이라는 '정보'는 보존하되 그 표현 방식을 '정규화'하여 후속 계층인 3D 원통형 컨볼루션이 이 정보를 일관되게 처리하고 활용할 수 있도록 한다. 이는 회전 변화에 강인하면서도 표현력을 잃지 않는 기술자를 학습하는 데 있어 핵심적인 설계 원리이다.6



공간 포인트 트랜스포머를 통해 SO(2) 등변성을 갖도록 정규화된 포인트 클라우드는 이제 컨볼루션 신경망(CNN)이 처리하기 용이한 그리드 형태로 변환되어야 한다. SpinNet은 이를 위해 원통형 복셀화(Cylindrical Voxelization)라는 독창적인 방법을 사용한다.


$$
r = \sqrt{x^2 + y^2}
\theta = \text{atan2}(y, x)
z = z
$$

이후, 이 연속적인 $(r, \theta, z)$ 좌표를 미리 정의된 해상도에 따라 이산적인 인덱스로 양자화(quantization)하여 3차원 원통형 볼륨(cylindrical volume) 또는 텐서(tensor)를 생성한다.8 이 과정은 불규칙하고 순서 없는 포인트들의 집합을, 정렬되고 구조화된 데이터 표현으로 바꾸는 역할을 한다. 이는 일반적인 직육면체 복셀화 21와는 달리, 데이터가 가진 Z축 중심의 회전 대칭성을 자연스럽게 표현할 수 있어 SO(2) 등변성을 유지하는 데 최적화된 구조이다.


원통형 볼륨을 효과적으로 처리하기 위해, SpinNet은 표준 3D 컨볼루션을 변형한 '3D 원통형 컨볼루션'을 제안한다. 이 컨볼루션의 핵심적인 특징은 방위각($\theta$) 차원에 대해 **순환적 패딩(circular padding)**을 적용한다는 점이다.

일반적인 컨볼루션은 경계 영역을 처리하기 위해 제로 패딩(zero-padding)을 사용하지만, 이는 방위각의 주기적 특성(예: 359도와 0도는 사실상 인접함)을 무시하게 된다. 반면, 순환적 패딩은 $\theta$ 축의 시작과 끝이 연결되어 있다고 간주하여, 컨볼루션 필터가 경계를 넘어설 때 반대편 경계의 값을 가져와 사용한다. 예를 들어, 0도 근방을 처리할 때는 359도 근방의 정보를, 359도 근방을 처리할 때는 0도 근방의 정보를 활용하는 식이다.

이러한 설계는 공간 포인트 트랜스포머가 확립한 SO(2) 등변성을 컨볼루션 연산 과정 전반에 걸쳐 일관되게 유지시키는 결정적인 역할을 한다. 입력 원통형 볼륨이 Z축을 중심으로 회전하면($\theta$ 축을 따라 shift), 순환적 패딩 덕분에 컨볼루션의 출력 특징 맵 또한 $\theta$ 축을 따라 동일하게 shift된다. 이를 통해 네트워크는 객체의 회전 각도와 무관하게 동일한 기하학적 패턴에 대해 일관된 특징을 학습할 수 있으며, 풍부한 공간적, 문맥적 정보를 효율적으로 집약하여 최종 기술자를 생성하게 된다.6


SpinNet의 전체적인 데이터 처리 파이프라인과 학습 과정은 다음과 같이 요약할 수 있다.

1. **입력:** 정합 대상이 되는 두 포인트 클라우드에서 각각 키포인트(keypoint)를 중심으로 하는 로컬 서피스 패치(일정 반경 내의 포인트 집합)가 입력으로 사용된다.
2. **공간 포인트 트랜스포머:** 각 패치는 공간 포인트 트랜스포머 모듈을 통과한다. 이 모듈은 PCA 기반 Z축 정렬과 구면 복셀화 기반 XY 평면 변환을 순차적으로 수행하여 패치를 SO(2) 등변성을 갖는 표준화된 형태로 변환한다.
3. **원통형 복셀화:** 표준화된 포인트들은 원통 좌표계를 기준으로 양자화되어 3D 원통형 볼륨으로 재구성된다.
4. **신경망 특징 추출기 (Neural Feature Extractor):**
   - 먼저, 각 복셀 내의 포인트들은 PointNet과 유사한 구조의 다층 퍼셉트론(MLP)을 통과하여 초기 복셀 특징(voxel-wise feature)으로 인코딩된다.
   - 이 초기 특징 맵은 여러 층의 3D 원통형 컨볼루션 레이어를 통과하며, 더 넓은 범위의 공간적, 문맥적 정보를 집약하는 고차원 특징으로 발전한다.
5. **기술자 생성:** 최종 컨볼루션 레이어의 출력은 전역 풀링(global pooling)과 같은 기법을 통해 고정된 차원의 컴팩트한 기술자 벡터(compact descriptor vector)로 압축된다.

**학습 과정:** SpinNet은 지도 학습(Supervised Learning) 방식으로 학습된다.8 3DMatch와 같은 대규모 데이터셋은 수많은 3D 스캔 단편(fragment)과 그들 사이의 정답 기하 변환 정보를 제공한다.8 학습 시에는 Siamese 네트워크와 유사한 프레임워크가 사용된다. 즉, 동일한 SpinNet 네트워크가 두 개의 입력 패치(하나는 앵커, 다른 하나는 포지티브 또는 네거티브 샘플)를 동시에 처리한다. 손실 함수로는 주로 Contrastive Loss나 Triplet Loss가 사용된다.

- **Contrastive Loss:** 매칭되는 쌍(positive pair)의 기술자 간 유클리드 거리는 특정 마진($m_{pos}$)보다 작아지도록, 매칭되지 않는 쌍(negative pair)의 기술자 간 거리는 다른 마진($m_{neg}$)보다 커지도록 네트워크 가중치를 업데이트한다.
- **Triplet Loss:** 앵커($A$), 포지티브($P$), 네거티브($N$)로 구성된 세 개의 샘플을 사용하여, 앵커와 포지티브 간의 거리($d(A, P)$)가 앵커와 네거티브 간의 거리($d(A, N)$)보다 최소한 일정한 마진($\alpha$) 이상 작아지도록 학습한다. 즉, $d(A, P) + \alpha < d(A, N)$을 만족하도록 한다.

이러한 학습 과정을 통해 SpinNet은 유사한 기하학적 구조를 가진 패치들은 특징 공간 상에서 가까운 위치에, 다른 구조를 가진 패치들은 먼 위치에 매핑하도록 학습된다.



SpinNet과 전통적인 포인트 클라우드 정합 기법을 비교하면, 딥러닝 기반 접근법이 갖는 근본적인 장점이 명확하게 드러난다.

**ICP (Iterative Closest Point):** ICP 및 그 변형 알고리즘들은 포인트 클라우드 정합의 고전적인 기준으로 여겨진다.11 이 알고리즘은 두 포인트 클라우드 간의 대응점을 찾고(일반적으로 가장 가까운 점), 이 대응점들 간의 거리를 최소화하는 변환을 계산하는 과정을 수렴할 때까지 반복한다. ICP의 가장 큰 약점은 초기 추정값에 대한 민감성이다. 만약 두 포인트 클라우드가 초기에 서로 멀리 떨어져 있거나 잘못된 방향으로 정렬되어 있다면, 알고리즘은 쉽게 잘못된 대응점을 찾게 되고 결국 지역 최적해(local optimum)에 수렴하여 정합에 실패한다.12 반면, SpinNet은 이러한 반복적 최적화 과정에 앞서, 각 포인트의 지역적 기하학 구조를 설명하는 고유하고 강인한 기술자를 생성한다. 이 기술자들을 특징 공간에서 비교함으로써, 초기 위치에 관계없이 신뢰할 수 있는 대응점 후보군을 찾을 수 있다. 이는 SpinNet이 전역적인 정합(global registration) 문제에 훨씬 더 강인하게 만들어주는 핵심적인 차이점이다. 즉, SpinNet은 데이터로부터 학습된 표현력을 통해 ICP가 빠지기 쉬운 지역 최적해 문제를 근본적으로 회피한다.

**FPFH, SHOT:** FPFH(Fast Point Feature Histograms)와 SHOT(Signatures of Histograms of OrienTations)은 대표적인 수작업 특징 기술자이다.8 이들은 특정 포인트 주변의 기하학적 속성(예: 법선 벡터 간의 각도, 점들 간의 거리)을 계산하여 이를 히스토그램 형태로 인코딩한다. 이러한 방식은 특정 유형의 구조에서는 효과적일 수 있으나, 몇 가지 본질적인 한계를 가진다. 첫째, 히스토그램 기반 표현은 기하학적 정보의 공간적 배열을 일부 손실시키며, 이는 표현력을 저하시킬 수 있다. 둘째, 이들은 사전에 정의된 규칙에 기반하므로 센서 노이즈, 불균일한 포인트 밀도, 부분적인 데이터 손실과 같은 실제 데이터의 불완전성에 매우 취약하다. 예를 들어, 포인트 밀도가 낮은 영역에서는 안정적인 법선 벡터 추정이 어려워 기술자의 신뢰도가 급격히 떨어진다.8 반면, SpinNet은 종단간 학습(end-to-end learning)을 통해 데이터에 존재하는 다양한 변칙성(노이즈, 밀도 변화 등)을 모두 고려하여 특징 표현을 학습한다. 이로 인해 SpinNet의 기술자는 수작업 기술자보다 훨씬 더 높은 표현력과 강인성을 가지며, 복잡하고 예측 불가능한 실제 환경 시나리오에 대해 뛰어난 유연성과 일반화 성능을 보인다.


SpinNet은 동일한 딥러닝 패러다임 내의 다른 주요 방법론들과 비교했을 때, 그 설계 철학의 독창성이 더욱 부각된다. 특히 PointNetLK와 DCP(Deep Closest Point)는 SpinNet과 동시대에 제안된 대표적인 DL-PCR 방법으로, 서로 다른 접근 방식을 보여준다.

**PointNetLK:** PointNetLK는 PointNet을 사용하여 각 포인트 클라우드의 전역 특징 벡터(global feature vector)를 추출한 뒤, 고전적인 컴퓨터 비전 알고리즘인 Lucas-Kanade(LK)를 적용하여 두 특징 벡터 간의 차이를 최소화하는 변환 행렬을 직접 회귀(regression)하는 하이브리드 방식이다.28 이 방법은 포인트 간의 명시적인 대응점을 찾지 않는 '대응점 없는(correspondence-free)' 접근법이라는 특징을 가진다.15 그러나 PointNetLK의 핵심인 PointNet 아키텍처는 입력 포인트의 순서에는 불변하지만 회전에는 불변하지 않다. 따라서 PointNetLK는 회전 강인성을 데이터 증강을 통해 학습에 의존할 수밖에 없으며, 이는 학습 데이터에서 보지 못한 큰 회전 변환에 대해 성능이 저하되는 원인이 된다.7 SpinNet과의 근본적인 차이는 여기서 발생한다. SpinNet은 지역 기술자(local descriptor)를 학습하여 대응점 기반 정합에 사용하며, 무엇보다도 아키텍처 자체에 회전 불변성을 구조적으로 내장하고 있다. 이로 인해 SpinNet은 별도의 데이터 증강 없이도 뛰어난 회전 불변성을 확보하고, 이는 더 나은 일반화 성능으로 이어진다.

**Deep Closest Point (DCP):** DCP는 PointNet이나 DGCNN과 같은 임베딩 네트워크를 통해 각 포인트의 고차원 특징을 추출하고, 트랜스포머의 어텐션 메커니즘을 활용하여 두 포인트 클라우드 간의 소프트 대응 관계(soft correspondence)를 확률적으로 예측한다.13 이후, 미분 가능한 SVD(Singular Value Decomposition) 계층을 통해 이 소프트 대응 관계로부터 최적의 변환 행렬을 계산한다.32 DCP 역시 특징 추출 단계에서 사용하는 네트워크가 회전 불변이 아니기 때문에, 회전 강인성을 데이터 증강에 의존한다. SpinNet과 DCP는 모두 딥러닝을 통해 특징을 학습한다는 공통점이 있지만, 그 목적과 방식에 차이가 있다. DCP는 두 포인트 클라우드 간의 '대응 관계'를 학습하는 데 집중하는 반면, SpinNet은 각 포인트의 '회전 불변적인 고유한 신원(identity)' 즉, 기술자를 학습하는 데 집중한다. SpinNet의 접근 방식은 기술자 자체에 불변성을 부여함으로써, 특징 매칭 단계를 더욱 간단하고 강인하게 만들어 결과적으로 더 우수한 일반화 성능을 달성하게 한다.8


SpinNet 이후, 3D 포인트 클라우드 처리 분야에서는 자연어 처리에서 큰 성공을 거둔 트랜스포머 아키텍처를 본격적으로 도입하는 연구들이 활발히 진행되었다. RoITr(Rotation-Invariant Transformer)과 같은 최신 모델들이 그 대표적인 예이다.10 이 모델들은 어텐션 메커니즘을 사용하여 포인트 클라우드 내의 모든 포인트 쌍 간의 기하학적 관계를 종합적으로 학습한다.

이들 모델은 회전 불변성을 확보하기 위해 PPF(Point Pair Features)와 같이 그 자체로 회전 불변적인 기하학적 측정치를 입력으로 사용한다. 즉, 임의의 두 포인트와 그 법선 벡터로 구성된 PPF는 좌표계의 회전에 영향을 받지 않는다. 트랜스포머는 이러한 불변적인 입력 정보를 바탕으로 어텐션을 통해 어떤 포인트 쌍이 구조적으로 중요한 관계를 맺고 있는지를 학습하여, 포즈 정보와 순수한 기하학 정보를 효과적으로 분리해낸다.10

이를 SpinNet과 비교하면, 문제 해결 방식의 철학적 차이를 엿볼 수 있다. SpinNet은 입력된 회전-의존적인 데이터를 공간 변환이라는 '절차'를 통해 회전-불변적인 표준 형태로 '구현(implement)'한 뒤, 이 표준화된 데이터로부터 특징을 학습한다. 반면, 최신 트랜스포머 모델들은 처음부터 회전-불변적인 입력을 네트워크에 제공하고, 어텐션 메커니즘을 통해 그 관계성을 '학습(learn)'한다. 이는 딥러닝 기반 정합 기술이 구조적 불변성을 설계하는 단계에서, 보다 복잡한 관계형 추론(relational reasoning)으로 발전하고 있음을 보여주는 중요한 흐름이다.

이러한 비교 분석을 통해, 딥러닝 기반 정합 기술이 단일한 접근법이 아니라 다양한 철학의 스펙트럼 위에 존재함을 알 수 있다. PointNetLK는 학습된 특징 공간 내에서 고전적 최적화를 수행하고, DCP는 특징 추출과 대응점 예측 모두를 학습에 의존하며, SpinNet은 기하학적 원리를 미분 가능한 모듈로 구현하여 구조적 선험(structural prior)을 네트워크에 부여한다. SpinNet의 접근 방식은 '블랙박스' 모델이 데이터만으로 기하학적 불변성을 배우도록 기대하는 대신, 이미 잘 알려진 기하학적 원리를 아키텍처에 명시적으로 통합하는 것이 더 효과적일 수 있음을 시사한다.


| 구분 (Category)           | SpinNet                                                  | PointNetLK                        | Deep Closest Point (DCP)                 |
| ------------------------- | -------------------------------------------------------- | --------------------------------- | ---------------------------------------- |
| **특징 유형**             | 지역 기술자 (Local Descriptor)                           | 전역 특징 (Global Feature)        | 지역 특징 (Local Feature)                |
| **회전 불변성 확보 방식** | 구조적 설계 (Spatial Point Transformer, 원통형 컨볼루션) | 학습에 의존 (데이터 증강 필요)    | 학습에 의존 (데이터 증강 필요)           |
| **대응점 처리**           | 대응점 기반 (Descriptor Matching)                        | 대응점 없음 (Correspondence-Free) | 소프트 대응점 예측 (Soft Correspondence) |
| **핵심 알고리즘/모듈**    | 공간 포인트 트랜스포머                                   | Lucas-Kanade 알고리즘             | 어텐션 및 미분가능 SVD                   |



SpinNet의 성능은 여러 표준 벤치마크 데이터셋을 통해 검증되었으며, 특히 기존 방법론 대비 뛰어난 정확도와 일반화 능력을 입증했다.

**실내 데이터셋 (Indoor Datasets - 3DMatch):** 3DMatch는 다양한 실내 장면을 RGB-D 카메라로 스캔하여 재구성한 대규모 데이터셋으로, 부분 중첩 정합 알고리즘의 성능을 평가하는 표준 벤치마크로 널리 사용된다.33 SpinNet은 이 3DMatch 벤치마크에서 당시의 최첨단(State-of-the-Art, SOTA) 성능을 달성했다.8 주요 평가 지표인 특징 매칭 재현율(Feature Matching Recall, FMR)에서 97.6%라는 높은 점수를 기록하며, FCGF(95.2%), D3Feat(95.3%) 등 경쟁 모델들을 큰 차이로 앞섰다.8 FMR은 올바른 대응점 쌍의 기술자 거리가 오탐지 대응점 쌍의 기술자 거리보다 가까운 비율을 측정하는 지표로, 기술자의 변별력과 정확도를 직접적으로 나타낸다. 이 결과는 SpinNet이 복잡하고 다양한 기하학적 구조를 포함하는 실내 환경에서 매우 정확하고 강인한 지역 기술자를 성공적으로 학습했음을 정량적으로 보여준다.35

**실외 데이터셋 (Outdoor Datasets - KITTI, ETH):** SpinNet의 진정한 기술적 성취는 미경험 데이터(unseen data)에 대한 탁월한 일반화 능력에서 드러난다. 연구진은 오직 실내 데이터셋인 3DMatch에서만 SpinNet을 학습시킨 후, 전혀 다른 특성을 가진 실외 데이터셋에서 성능을 평가했다. ETH 데이터셋은 야외 환경을 레이저 스캐너로 측정한 데이터로, 3DMatch와는 센서 양식, 데이터 밀도, 장면의 종류가 완전히 다르다. 이러한 극심한 도메인 격차(domain gap)에도 불구하고, SpinNet은 ETH 데이터셋에서 92.8%라는 경이로운 FMR을 달성했다. 이는 동일한 조건에서 평가된 기존 SOTA 모델인 FCGF(79.8%)를 약 13%p, D3Feat(26.2%)을 압도적으로 능가하는 수치이다.8

이러한 결과는 SpinNet이 특정 데이터셋의 통계적 특성에 과적합(overfitting)된 것이 아니라, 회전 불변성이라는 보다 근본적이고 보편적인 기하학적 원리를 학습했음을 강력하게 시사한다. 또한, 자율주행 연구의 핵심 데이터셋인 KITTI에 대해서도 우수한 일반화 성능을 보여, 실제 자율주행 시나리오에서의 LiDAR 데이터 정합 문제에 대한 높은 잠재력을 입증했다.23


SpinNet이 보여준 뛰어난 일반화 능력의 근원은 데이터 증강이라는 확률적 해법 대신, 아키텍처 설계를 통한 결정론적 해법을 선택한 데 있다. 기존의 많은 딥러닝 모델들은 회전 불변성을 학습하기 위해 수많은 회전된 데이터를 모델에 보여주는 데이터 증강 기법에 의존했다. 그러나 이 방법은 본질적으로 3차원 회전의 연속적인 공간(SO(3) 그룹)을 유한하고 이산적인 샘플들로 근사하는 것이기 때문에, 학습 시 보지 못했던 회전 각도에 대해서는 성능을 보장할 수 없다.10 모델은 특정 각도에 대해서는 강인할 수 있지만, 그 사이의 미세한 각도 변화에는 취약할 수 있다.

반면, SpinNet의 공간 포인트 트랜스포머는 입력 데이터에 어떤 회전이 가해지더라도 항상 동일한 절차를 통해 데이터를 표준화된 공간으로 매핑한다. 이는 모델이 회전이라는 변수를 애초에 '고려할 필요가 없도록' 만들어준다. 따라서 모델은 한정된 학습 용량(capacity)을 회전 변화에 대응하는 데 낭비하지 않고, 오롯이 순수한 기하학적 형태와 구조를 학습하는 데 집중할 수 있다.

이러한 구조적 접근법은 센서의 종류(RGB-D 카메라 vs. LiDAR)나 데이터의 분포(실내 vs. 실외)가 달라짐에 따라 발생하는 도메인 이동(domain shift) 문제를 완화하는 데 결정적인 역할을 한다. 센서나 환경이 바뀌더라도, 기하학의 근본적인 원리는 변하지 않기 때문이다. SpinNet은 바로 이 변하지 않는 원리에 집중하도록 설계되었기 때문에, 학습 환경과 매우 다른 테스트 환경에서도 안정적인 성능을 유지할 수 있었던 것이다. 이는 머신러닝 모델을 실험실 수준에서 실제 현장에 배포할 때 가장 큰 장애물 중 하나인 일반화 문제를 아키텍처 설계를 통해 효과적으로 해결한 모범적인 사례로 평가받을 수 있다.43


SpinNet은 혁신적인 접근법으로 뛰어난 성능을 입증했지만, 그 설계와 방법론에는 몇 가지 내재적인 한계와 기술적 난제가 존재한다. 이러한 한계를 이해하는 것은 SpinNet의 적절한 활용 범위를 설정하고 향후 연구 방향을 모색하는 데 중요하다. 흥미롭게도, SpinNet의 한계는 1930년대에 등장하여 작은 크기와 저렴한 가격으로 인기를 끌었던 '스피넷 피아노(spinet piano)'의 특성과 유사한 측면을 보인다. 스피넷 피아노는 작은 공간에 피아노 메커니즘을 모두 담기 위해 '드롭 액션(drop action)'이라는 독창적인 설계를 사용했지만, 이로 인해 건반의 터치감, 음질, 조율의 안정성 등 연주 성능 면에서 본질적인 타협을 해야만 했다.49 이처럼 특정 문제를 해결하기 위한 독창적인 설계가 다른 측면에서의 성능 저하를 야기하는 트레이드오프(trade-off) 관계는 SpinNet에서도 발견된다.

**지도 학습의 데이터 의존성:** SpinNet은 지도 학습 모델로서, 학습을 위해 3DMatch와 같이 정확한 대응 관계와 변환 행렬이 레이블링된 대규모 3D 데이터셋을 필요로 한다.8 실제 산업 현장에서 이러한 고품질의 학습 데이터를 구축하는 것은 막대한 시간과 비용을 요구하는 매우 어려운 작업이다. 이는 SpinNet의 실용적인 적용 범위를 제한하는 가장 큰 장벽 중 하나이다.1 최근 딥러닝 연구의 주된 흐름이 레이블링 비용을 줄이기 위해 비지도(unsupervised) 또는 자기지도(self-supervised) 학습으로 빠르게 이동하고 있다는 점을 고려할 때, 이는 SpinNet의 중요한 한계점으로 지적될 수 있다.55

**극심한 환경 조건에서의 잠재적 취약성:** SpinNet은 일반적인 수준의 노이즈와 밀도 변화에 강인하도록 설계되었지만, 그 한계를 넘어서는 극심한 조건에서는 성능 저하를 겪을 수 있다. 예를 들어, 포인트 클라우드가 극도로 희소(sparse)하여 지역적인 기하학 구조를 파악하기 어렵거나, 포인트 밀도의 불균일성이 매우 커서 PCA 기반의 법선 벡터 추정이 불안정해지는 경우, 공간 포인트 트랜스포머의 정규화 성능이 저하될 수 있다.4 또한, 복도와 같은 평면이나 반복적인 대칭 구조가 지배적인 환경에서는 지역 기술자의 변별력(descriptiveness)이 감소하여 잘못된 매칭을 유발할 수 있다.58

**정보 손실 가능성:** 스피넷 피아노가 짧은 현과 작은 사운드보드로 인해 풍부한 배음을 손실하는 것처럼 49, SpinNet 역시 입력 데이터를 원통형 복셀 그리드로 변환하는 과정에서 정보 손실의 가능성을 내포한다. 복셀화는 연속적인 3D 공간을 이산적인 격자로 근사하는 과정으로, 필연적으로 양자화 오류(quantization error)를 수반한다. 이로 인해 포인트들의 미세한 상대적 위치나 표면의 세밀한 곡률과 같은 정밀한 기하학적 정보가 일부 손실될 수 있다. 대부분의 응용에서는 이러한 손실이 무시할 수 있는 수준이지만, 마이크로미터 단위의 정밀도가 요구되는 산업용 부품 검사나 정밀 계측 분야에서는 이러한 정보 손실이 정합 정확도에 영향을 미치는 한계로 작용할 수 있다.

**다중 양식 정보의 부재:** SpinNet은 오직 포인트의 3차원 좌표, 즉 기하학적 정보만을 사용하여 기술자를 생성한다. 그러나 많은 현대 3D 센서들은 기하학 정보와 함께 RGB 색상과 같은 방사측정 정보(radiometric information)를 함께 제공한다. 기하학적으로는 구별이 어려운 평평한 벽이라도 텍스처나 색상 패턴이 있다면 쉽게 구별할 수 있다. SpinNet은 이러한 추가적인 정보 소스를 활용하지 않기 때문에, 기하학적으로는 모호하지만 색상 정보로는 명확하게 구별 가능한 영역에서 성능 향상의 기회를 놓치게 된다.58


아래 표는 3DMatch 데이터셋으로만 학습했을 때, 주요 방법론들의 특징 매칭 재현율(FMR)을 비교한 것이다. SpinNet의 탁월한 일반화 능력이 명확히 드러난다.

| 방법론 (Method) | 3DMatch (Trained & Tested) FMR (%) | ETH (Unseen) FMR (%) |
| --------------- | ---------------------------------- | -------------------- |
| FPFH 8          | 87.7                               | 34.6                 |
| FCGF 8          | 95.2                               | 79.8                 |
| D3Feat 8        | 95.3                               | 26.2                 |
| **SpinNet** 8   | **97.6**                           | **92.8**             |



SpinNet이 제공하는 회전 불변적이고 강인한 지역 기술자는 다양한 실제 응용 분야에서 기존 기술의 한계를 극복하고 새로운 가능성을 열어줄 잠재력을 가지고 있다.

**자율주행 (Autonomous Driving):** 자율주행 차량의 핵심 센서인 LiDAR는 주변 환경을 3차원 포인트 클라우드로 인식한다. 차량이 이동함에 따라 연속적으로 취득되는 LiDAR 스캔들을 정밀하게 정합하는 것은 차량의 현재 위치와 자세를 추정하는 LiDAR 오도메트리(Odometry)와 SLAM(Simultaneous Localization and Mapping) 기술의 근간을 이룬다.5 SpinNet은 변화무쌍한 주행 환경과 다양한 객체들의 회전에도 불구하고 안정적인 특징점을 매칭할 수 있어, 누적 오차를 줄이고 SLAM 시스템의 전반적인 정확도와 강인성을 크게 향상시킬 수 있다. 특히, KITTI 벤치마크에서의 뛰어난 일반화 성능은 SpinNet이 실제 도로 환경 데이터 처리에 매우 효과적일 수 있음을 시사한다.23

**로보틱스 (Robotics):** 산업 현장이나 비정형 환경에서 로봇이 자율적으로 작업을 수행하기 위해서는 주변 객체를 정확하게 인식하고 그 자세(6-DoF pose)를 추정하는 능력이 필수적이다.62 예를 들어, 물류 창고의 로봇 팔이 무질서하게 놓인 상자들 중 특정 상자를 집기 위해서는, 카메라나 3D 센서로 촬영한 부분적인 포인트 클라우드를 사전에 저장된 상자의 3D CAD 모델과 정밀하게 정합해야 한다. SpinNet은 부분적인 관측 데이터와 심한 자세 변화에도 강인한 대응점을 찾아내므로, 이러한 객체 자세 추정 문제(object pose estimation)의 정확도를 높여 로봇의 파지(grasping) 및 조작(manipulation) 성공률을 증대시키는 데 기여할 수 있다.7

**3D 재구성 (3D Reconstruction):** 건물, 문화유산, 대규모 지형 등 현실 세계의 대상을 디지털 3D 모델로 복원하는 작업은 여러 위치에서 취득한 다수의 스캔 데이터를 정합하는 것으로부터 시작된다.6 SpinNet의 높은 특징 매칭 재현율은 스캔들 간의 신뢰할 수 있는 대응점을 대량으로 확보할 수 있게 하여, 후속 단계인 전역 정합(global registration) 및 번들 조정(bundle adjustment)의 정확도를 높인다. 이는 최종적으로 더 정밀하고 완전한 형태의 3D 모델을 생성하는 결과로 이어진다.


SpinNet은 그 자체로 완성된 솔루션이라기보다는, 차세대 정합 기술로 나아가기 위한 중요한 개념적 토대를 제공하는 '기반 모듈'로서 더 큰 가치를 지닌다. SpinNet의 한계점들은 역설적으로 미래 연구가 나아가야 할 방향을 명확히 제시한다.

**비지도/자기지도 학습으로의 확장:** SpinNet의 가장 큰 실용적 한계인 지도 학습의 데이터 의존성을 극복하는 것이 최우선 과제이다. 레이블이 없는 대규모 데이터로부터 유용한 표현을 학습하는 비지도(unsupervised) 또는 자기지도(self-supervised) 학습 패러다임으로의 전환이 필수적이다. 예를 들어, 두 패치가 기하학적으로 일치하는지를 예측하는 것을 보조 작업(pretext task)으로 설정하거나, 정합 결과를 다시 입력 데이터에 투영하여 일관성을 확인하는 방식의 자기지도 손실 함수를 설계하여 SpinNet 아키텍처를 학습시킬 수 있다.8 이는 데이터 구축 비용을 획기적으로 절감하고 모델의 적용 범위를 넓히는 핵심적인 연구 방향이 될 것이다.

**다중 양식 정보와의 결합 (Integration with Multi-modal Information):** 기하학적 정보만으로는 모호성이 해결되지 않는 경우가 많다. 평평한 벽에 붙어있는 포스터나, 동일한 모양이지만 색깔이 다른 의자들을 구별하는 데에는 RGB 색상 정보가 결정적인 역할을 한다. 향후 연구는 SpinNet의 강인한 기하학적 특징 추출 능력과 2D CNN 등에서 추출한 색상 및 텍스처 특징을 효과적으로 융합하는 하이브리드 아키텍처 개발에 집중해야 한다.58 이는 어텐션 기반의 교차 양식 융합(cross-modal fusion) 메커니즘 등을 통해 이루어질 수 있으며, 정합 성능을 한 단계 더 끌어올릴 잠재력을 지니고 있다.

**동적 환경 및 비강체 정합으로의 확장:** 현재 SpinNet은 장면 내의 모든 요소가 강체(rigid body)처럼 움직인다고 가정한다. 그러나 실제 환경에는 사람, 차량과 같이 독립적으로 움직이는 동적 객체나, 천이나 인체와 같이 형태가 변하는 비강체(non-rigid) 객체가 존재한다. 미래의 정합 기술은 이러한 복잡성을 다룰 수 있어야 한다. 예를 들어, SpinNet을 사용하여 먼저 배경과 같은 정적인 부분을 강체 정합으로 정렬한 뒤, 정합되지 않고 남은 동적/비강체 객체들에 대해 별도의 움직임이나 변형을 추정하는 계층적 접근법을 생각해 볼 수 있다.65 이는 SpinNet의 핵심 아이디어를 보다 복잡하고 현실적인 문제로 확장하는 중요한 연구 과제가 될 것이다.



SpinNet은 3차원 포인트 클라우드 정합 분야, 특히 딥러닝 기반 접근법에서 중요한 기술적 이정표를 제시했다. 이 모델의 핵심 기여는 **회전 불변성(rotation invariance)이라는 근본적인 기하학적 문제를 해결하기 위해, 데이터 증강에 의존하는 확률적 방식이 아닌 아키텍처 설계를 통한 구조적, 결정론적 접근법을 성공적으로 구현**했다는 점에 있다. 독창적으로 고안된 '공간 포인트 트랜스포머'는 입력 데이터를 결정론적 절차에 따라 표준화된 공간으로 매핑하며, '3D 원통형 컨볼루션'은 이 표준화된 표현의 등변성을 유지하며 풍부한 특징을 학습한다. 이 두 가지 핵심 요소를 통해 SpinNet은 수작업 특징이나 불안정한 LRF 계산 없이도, 내재적으로 회전 불변적인 고성능 지역 특징 기술자를 종단간(end-to-end) 방식으로 학습하는 데 성공했다.


SpinNet의 가장 큰 기술적 의의는 **미경험 데이터(unseen data)에 대한 탁월한 일반화 성능을 실험적으로 입증**했다는 점이다. 실내 데이터셋만으로 학습된 모델이 센서, 환경, 데이터 분포가 전혀 다른 실외 데이터셋에서 기존의 최첨단 모델들을 압도하는 성능을 보인 것은, 딥러닝 모델이 단순히 학습 데이터의 패턴을 암기하는 것을 넘어, 데이터에 내재된 근본적인 기하학적 원리를 학습할 수 있음을 보여준 강력한 증거이다. 이는 딥러닝 모델의 '블랙박스'적인 특성에 대한 우려를 일부 해소하고, 문제의 본질을 반영한 구조적 사전 지식(structural prior)을 아키텍처 설계에 통합하는 것이 얼마나 중요한지를 명확히 보여주었다. SpinNet의 성공은 이후의 많은 3D 비전 연구들이 데이터의 기하학적 특성(대칭성, 불변성 등)을 네트워크 아키텍처에 어떻게 효과적으로 반영할 것인가에 대해 더 깊이 고민하게 만드는 계기가 되었다.


SpinNet이 제시한 방향성을 바탕으로, 향후 포인트 클라우드 정합 연구는 다음과 같은 방향으로 나아가야 할 것이다. 첫째, SpinNet의 구조적 장점을 계승하면서도 지도 학습의 한계를 벗어나기 위한 **비지도 및 자기지도 학습 패러다임의 적극적인 도입**이 시급하다. 이를 통해 데이터 구축의 병목 현상을 해결하고 기술의 실용성과 확장성을 확보해야 한다. 둘째, 기하학 정보만으로는 해결할 수 없는 모호성을 극복하기 위해 **RGB 색상, 적외선, 의미론적 정보 등 다중 양식 데이터를 융합**하여 특징 표현력을 극대화하는 연구가 필요하다. 마지막으로, 정적인 강체 환경이라는 가정을 넘어, 실제 세계의 복잡성을 반영하는 **동적 객체 및 비강체 변형을 처리할 수 있는 알고리즘으로의 확장**이 이루어져야 한다. 이러한 연구 방향들은 SpinNet이 구축한 견고한 기반 위에서 3D 포인트 클라우드 정합 기술을 한 단계 더 높은 수준으로 발전시켜, 자율주행, 로보틱스, 디지털 트윈 등 미래 핵심 산업 분야에서 더욱 정교하고 신뢰성 높은 솔루션을 제공하는 데 기여할 것이다.


1. A Comprehensive Survey and Taxonomy on Point Cloud Registration Based on Deep Learning - arXiv, accessed August 19, 2025, https://arxiv.org/html/2404.13830v2
2. Deep Learning-Based Point Cloud Registration: A Comprehensive Survey and Taxonomy, accessed August 19, 2025, https://arxiv.org/html/2404.13830v3
3. [2404.13830] Deep Learning-Based Point Cloud Registration: A Comprehensive Survey and Taxonomy - arXiv, accessed August 19, 2025, https://arxiv.org/abs/2404.13830
4. A review of rigid point cloud registration based on deep learning - PMC, accessed August 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10794353/
5. Advanced Point Cloud Registration Techniques - Number Analytics, accessed August 19, 2025, https://www.numberanalytics.com/blog/advanced-point-cloud-registration-techniques
6. SpinNet: Learning a General Surface Descriptor for 3D Point Cloud Registration - Bohrium, accessed August 19, 2025, https://www.bohrium.com/paper-details/spinnet-learning-a-general-surface-descriptor-for-3d-point-cloud-registration/867745603067576526-108597
7. Comparison of Point Cloud Registration Techniques on Scanned Physical Objects - MDPI, accessed August 19, 2025, https://www.mdpi.com/1424-8220/24/7/2142
8. SpinNet: Learning a General Surface Descriptor for 3D Point Cloud Registration - CVF Open Access, accessed August 19, 2025, https://openaccess.thecvf.com/content/CVPR2021/papers/Ao_SpinNet_Learning_a_General_Surface_Descriptor_for_3D_Point_Cloud_CVPR_2021_paper.pdf
9. Point rotation invariant features and attention fusion network for point cloud registration of 3D shapes - PubMed Central, accessed August 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12041196/
10. Rotation-Invariant Transformer for Point Cloud Matching - CVF Open Access, accessed August 19, 2025, https://openaccess.thecvf.com/content/CVPR2023/papers/Yu_Rotation-Invariant_Transformer_for_Point_Cloud_Matching_CVPR_2023_paper.pdf
11. Iterative closest point - Wikipedia, accessed August 19, 2025, https://en.wikipedia.org/wiki/Iterative_closest_point
12. Iterative Closest Point (ICP) for 3D Explained with Code, accessed August 19, 2025, https://learnopencv.com/iterative-closest-point-icp-explained/
13. Deep Closest Point: Learning Representations for Point Cloud Registration - CVF Open Access, accessed August 19, 2025, https://openaccess.thecvf.com/content_ICCV_2019/papers/Wang_Deep_Closest_Point_Learning_Representations_for_Point_Cloud_Registration_ICCV_2019_paper.pdf
14. DeepMatch: Toward Lightweight in Point Cloud Registration - Frontiers, accessed August 19, 2025, https://www.frontiersin.org/journals/neurorobotics/articles/10.3389/fnbot.2022.891158/full
15. XuyangBai/awesome-point-cloud-registration - GitHub, accessed August 19, 2025, https://github.com/XuyangBai/awesome-point-cloud-registration
16. Rethinking Rotation Invariance with Point Cloud Registration, accessed August 19, 2025, https://rotation3d.github.io/index_files/pdfs/AAAI2023_main.pdf
17. Spatial Transformer Networks - Convolutional Neural Networks for Image and Video Processing - BayernCollab, accessed August 19, 2025, https://collab.dvb.bayern/spaces/TUMlfdv/pages/69119917/Spatial+Transformer+Networks
18. Understanding when spatial transformer networks do not support invariance, and what to do about it - DiVA portal, accessed August 19, 2025, https://www.diva-portal.org/smash/get/diva2:1428271/FULLTEXT06.pdf
19. Spatial Transformer Networks Tutorial - PyTorch documentation, accessed August 19, 2025, https://docs.pytorch.org/tutorials/intermediate/spatial_transformer_tutorial.html
20. SpinNet: Learning a General Surface Descriptor for 3D Point Cloud Registration, accessed August 19, 2025, https://paperswithcode.com/paper/spinnet-learning-a-general-surface-descriptor
21. Voxelization & Pillarization. Pointclouds to Neural Inputs | by Manas Nand Mohan, accessed August 19, 2025, https://medium.com/@manasnandmohan/voxelization-pillarization-2bfa62f41c61
22. Voxelisation Algorithms and Data Structures: A Review - PMC - PubMed Central, accessed August 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8707769/
23. [CVPR 2021] SpinNet: Learning a General Surface Descriptor for 3D Point Cloud Registration - GitHub, accessed August 19, 2025, https://github.com/QingyongHu/SpinNet
24. Iterative Closest Point (ICP) and other registration algorithms, accessed August 19, 2025, [http://www.mrpt.org/Iterative_Closest_Point_%28ICP%29_and_other_matching_algorithms](http://www.mrpt.org/Iterative_Closest_Point_(ICP)_and_other_matching_algorithms)
25. 12.2: The Iterative Closest Point (ICP) Algorithm - Engineering LibreTexts, accessed August 19, 2025, https://eng.libretexts.org/Bookshelves/Mechanical_Engineering/Introduction_to_Autonomous_Robots_(Correll)/12%3A__RGB-D_SLAM/12.02%3A_The_Iterative_Closest_Point_(ICP)_Algorithm
26. Pset #2 part 1 of 3: Iterative Closest Point (ICP) - Robotic Manipulation, accessed August 19, 2025, https://manipulation.csail.mit.edu/Fall2020/pset2/pset2_ICP.html
27. pcregistericp - Register two point clouds using ICP algorithm - MATLAB - MathWorks, accessed August 19, 2025, https://www.mathworks.com/help/vision/ref/pcregistericp.html
28. PointNetLK Revisited - CVF Open Access, accessed August 19, 2025, https://openaccess.thecvf.com/content/CVPR2021/papers/Li_PointNetLK_Revisited_CVPR_2021_paper.pdf
29. awesome-point-cloud-registration/README.md at main - GitHub, accessed August 19, 2025, https://github.com/XuyangBai/awesome-point-cloud-registration/blob/main/README.md
30. Comparison of Point Cloud Registration Techniques on Scanned Physical Objects - PMC, accessed August 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11014384/
31. (PDF) Deep Closest Point: Learning Representations for Point Cloud Registration (2019) | Yue Wang | 789 Citations - SciSpace, accessed August 19, 2025, https://scispace.com/papers/deep-closest-point-learning-representations-for-point-cloud-kq94dtgkhn
32. Deep Closest Point: Learning Representations for Point Cloud Registration - ResearchGate, accessed August 19, 2025, https://www.researchgate.net/publication/339555589_Deep_Closest_Point_Learning_Representations_for_Point_Cloud_Registration
33. 3DMatch: Learning Local Geometric Descriptors from RGB-D Reconstructions, accessed August 19, 2025, https://3dmatch.cs.princeton.edu/
34. 3DMatch: Learning Local Geometric Descriptors from RGB-D Reconstructions - Princeton Vision & Robotics Group, accessed August 19, 2025, https://3dvision.princeton.edu/projects/2016/3DMatch/paper_v2.pdf
35. [1811.06879] The Perfect Match: 3D Point Cloud Matching with Smoothed Densities - arXiv, accessed August 19, 2025, https://arxiv.org/abs/1811.06879
36. Average recall (%) of different methods on the 3DMatch benchmark with τ... - ResearchGate, accessed August 19, 2025, https://www.researchgate.net/figure/Average-recall-of-different-methods-on-the-3DMatch-benchmark-with-t-1-10cm-and-t-2_tbl1_346303058
37. Tracking Evaluation - The KITTI Vision Benchmark Suite, accessed August 19, 2025, https://www.cvlibs.net/datasets/kitti/eval_tracking_overview.php
38. SemanticKITTI - A Dataset for LiDAR-based Semantic Scene Understanding, accessed August 19, 2025, https://semantic-kitti.org/
39. The KITTI Vision Benchmark Suite - Andreas Geiger, accessed August 19, 2025, https://www.cvlibs.net/datasets/kitti/
40. KITTI raw data - The KITTI Vision Benchmark Suite, accessed August 19, 2025, https://www.cvlibs.net/datasets/kitti/raw_data.php
41. KITTI Vision Benchmark Suite - Registry of Open Data on AWS, accessed August 19, 2025, https://registry.opendata.aws/kitti/
42. The KITTI Vision Benchmark Suite - YouTube, accessed August 19, 2025, https://www.youtube.com/watch?v=KXpZ6B1YB_k
43. Are All Unseen Data Out-of-Distribution? - arXiv, accessed August 19, 2025, https://arxiv.org/html/2312.16243v2
44. Generalization in Machine Learning: Tips for Better Models - Qohash, accessed August 19, 2025, https://qohash.com/generalization-in-machine-learning/
45. How to Make Your Machine Learning Models Generalize Well to Unseen Data, accessed August 19, 2025, https://mohamedbakrey094.medium.com/how-to-make-your-machine-learning-models-generalize-well-to-unseen-data-0e4d8af1f3a6
46. Why does my cnn model doesnt generalize well to unseen data? - DeepLearning.AI, accessed August 19, 2025, https://community.deeplearning.ai/t/why-does-my-cnn-model-doesnt-generalize-well-to-unseen-data/731121
47. Generalizing Deep Learning for Medical Image Segmentation to Unseen Domains via Deep Stacked Transformation - PMC, accessed August 19, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7393676/
48. Instability in the ability of a model to generalize to unseen data : r/MLQuestions - Reddit, accessed August 19, 2025, https://www.reddit.com/r/MLQuestions/comments/1c57jpc/instability_in_the_ability_of_a_model_to/
49. Spinet: What is a Spinet Piano vs Upright? 10 ways to know, accessed August 19, 2025, https://pianotechniciantuner.com/blog/piano-or-spinet-ten-ways-to-tell
50. Spinet - Wikipedia, accessed August 19, 2025, https://en.wikipedia.org/wiki/Spinet
51. Why a Spinet piano is different than Upright Pianos, accessed August 19, 2025, https://pianotechniciantuner.com/blog/why-a-spinet-piano-is-different-than-upright-pianos
52. ELI5: Why are spinets so hated? : r/piano - Reddit, accessed August 19, 2025, https://www.reddit.com/r/piano/comments/3lyt28/eli5_why_are_spinets_so_hated/
53. Retire Spinet Pianos: Good starter pianos and Bad starter pianos - PianoWorks, accessed August 19, 2025, https://www.pianoworks.com/piano-buying-guide/retire-spinet-pianos/
54. ZeroReg: Zero-Shot Point Cloud Registration with Foundation Models - arXiv, accessed August 19, 2025, https://arxiv.org/html/2312.03032v3
55. CGNet: A Correlation-Guided Registration Network for Unsupervised Deformable Image ... - PubMed, accessed August 19, 2025, https://pubmed.ncbi.nlm.nih.gov/40030290/
56. LungRegNet: An unsupervised deformable image registration method for 4D-CT lung, accessed August 19, 2025, https://pubmed.ncbi.nlm.nih.gov/32017141/
57. NeRF-Guided Unsupervised Learning of RGB-D Registration - arXiv, accessed August 19, 2025, https://arxiv.org/html/2405.00507v2
58. SpinNet: Learning a General Surface Descriptor for 3D Point Cloud Registration | Request PDF - ResearchGate, accessed August 19, 2025, https://www.researchgate.net/publication/355862384_SpinNet_Learning_a_General_Surface_Descriptor_for_3D_Point_Cloud_Registration
59. PCR-CG: Point Cloud Registration via Deep Explicit Color and Geometry - arXiv, accessed August 19, 2025, https://arxiv.org/html/2302.14418v2
60. Self-driving car - Wikipedia, accessed August 19, 2025, https://en.wikipedia.org/wiki/Self-driving_car
61. Self-Driving Car Technology for a Reliable Ride - Waymo Driver, accessed August 19, 2025, https://waymo.com/waymo-driver/
62. AI for Robotics - NVIDIA, accessed August 19, 2025, https://www.nvidia.com/en-us/industries/robotics/
63. Spin Robotics: Assembly & Screwdriving Technology, accessed August 19, 2025, https://spin-robotics.com/en
64. SpinNet: Learning a General Surface Descriptor for 3D Point Cloud Registration - arXiv, accessed August 19, 2025, https://arxiv.org/abs/2011.12149
65. HybridReg: Robust 3D Point Cloud Registration with Hybrid Motions - arXiv, accessed August 19, 2025, https://arxiv.org/html/2503.07019v1
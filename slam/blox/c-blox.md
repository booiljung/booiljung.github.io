# c-blox

## 1. 서론

### 0.1  자율 로봇을 위한 3차원 환경 인식의 중요성

자율 로봇이 인간의 개입 없이 복잡하고 동적인 환경에서 유의미한 작업을 수행하기 위해서는 주변 세계에 대한 정확한 3차원적 이해가 선행되어야 한다. 이러한 공간 인지 능력의 핵심에는 동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM) 기술이 자리 잡고 있다.1 SLAM은 로봇이 사전 정보가 없는 환경을 탐색하며 자신의 위치를 실시간으로 추정함과 동시에, 센서 데이터를 누적하여 환경의 지도를 점진적으로 구축하는 근본적인 문제이다.3 이 과정을 통해 생성된 지도는 로봇이 자율 주행, 경로 계획, 탐사, 그리고 환경과의 물리적 상호작용과 같은 고차원적 의사결정을 내리는 데 필수적인 기반 정보를 제공한다.4

SLAM의 결과물인 지도는 다양한 형태로 표현될 수 있다. 초기 SLAM 시스템들은 계산 효율성을 위해 환경의 주요 특징점(feature point)들만 3차원 점으로 표현하는 희소 특징점 맵(sparse feature map)을 주로 사용했다.1 이러한 희소 맵은 로봇의 위치를 센티미터 수준의 정확도로 추정하는 데에는 매우 효과적이지만, 맵 자체가 소수의 3차원 점들의 집합에 불과하여 환경의 기하학적 구조에 대한 충분한 정보를 제공하지 못한다.1 따라서 로봇이 장애물의 정확한 형태를 파악하여 회피하거나, 최적의 경로를 계획하는 등의 구체적인 상호작용 임무를 수행하기 위해서는 환경의 표면을 조밀하게 표현하는 밀집 3D 맵(dense 3D map)의 구축이 필수적이다.3

### 0.2  대규모 밀집 매핑의 핵심 난제: 누적 오차와 확장성

밀집 3D 맵은 풍부한 기하학적 정보를 제공하지만, 이를 대규모 환경에서 장시간에 걸쳐 구축하는 과정은 두 가지 근본적인 난제에 직면한다. 첫 번째는 '누적 오차(accumulated error)' 문제이다. 로봇의 위치는 주행계(odometry)나 시각 센서의 프레임 간 상대적 움직임을 적분하여 추정되므로, 매 순간 발생하는 미세한 측정 오차가 시간에 따라 계속해서 누적된다. 이러한 누적 오차는 '드리프트(drift)' 현상을 야기하여 로봇의 궤적 추정치를 실제 경로에서 점차 벗어나게 만들고, 결과적으로 생성되는 맵 전체에 심각한 왜곡을 초래하여 일관성(consistency)을 파괴한다.4

두 번째는 '확장성(scalability)' 문제이다. 전통적인 밀집 매핑 방식은 환경 전체를 단일 전역 맵(monolithic global map)으로 표현한다. 이 경우, 로봇의 탐사 범위가 넓어질수록 맵의 데이터 크기는 무한정 증가하게 된다.5 맵의 크기가 커짐에 따라 이를 저장하는 데 필요한 메모리 요구량은 물론, 새로운 센서 데이터를 통합하거나 맵에서 정보를 조회하는 데 필요한 계산 비용 역시 기하급수적으로 증가하여 실시간 처리를 불가능하게 만든다.2

이 두 문제는 서로 밀접하게 연관되어 있다. 예를 들어, 로봇이 이전에 방문했던 장소를 다시 인식하는 루프 폐쇄(loop closure)가 발생하면 누적된 드리프트 오차를 보정할 기회가 생긴다.7 하지만 단일 전역 맵 구조에서는 이 보정 정보를 맵 전체에 소급하여 적용해야 하므로, 맵이 클수록 계산 비용이 막대해져 지연된 루프 폐쇄에 효과적으로 대응하기 어렵다.4

### 0.3  C-blox의 등장 배경 및 연구 목표

이러한 대규모 밀집 매핑의 근본적인 한계를 극복하기 위한 대안으로 '서브맵(submap)' 기반 접근법이 제시되었다.2 이 접근법은 컴퓨터 과학의 '분할 정복(divide and conquer)' 전략에 착안하여, 거대한 단일 맵을 관리하는 대신 전체 환경을 여러 개의 작고 독립적인 지역 맵으로 분할하여 관리한다. 각 서브맵은 내부적으로는 강체(rigid)로 간주되어 일관성을 유지하며, 드리프트 오차는 서브맵 간의 상대적인 포즈 관계에 누적된다. 이를 통해 오차의 영향을 국지적으로 제한하고, 루프 폐쇄 시 전체 맵을 재구성할 필요 없이 서브맵들의 포즈 그래프만을 최적화함으로써 계산 복잡도를 획기적으로 낮출 수 있다.7

C-blox는 이러한 서브맵 개념을 Truncated Signed Distance Field (TSDF)라는 강력한 밀집 체적 표현 방식에 적용한 선구적인 시스템이다.4 2018년 IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)에서 발표된 이 시스템은 ETH 취리히의 자율 시스템 연구소(Autonomous Systems Lab)에서 개발되었으며, 로보틱스 분야에서 널리 사용되는 오픈소스 라이브러리인 Voxblox를 기반으로 한다.5

C-blox의 핵심 연구 목표는 "불완전한 자세 추정(imperfect pose estimates)이 존재하는 현실적인 환경에서, 확장 가능하면서도(scalable) 일관성을 유지하는(consistent) 밀집 3D 맵을 구축하는 것"으로 요약할 수 있다.10 이를 달성하기 위해 C-blox는 환경을 중첩 가능한 여러 개의 TSDF 서브볼륨(subvolume)의 집합으로 표현하는 새로운 패러다임을 제안했다. 또한, 맵에서 기하학적으로 안정된 영역을 식별하고, 해당 영역에 기여하는 여러 서브맵을 하나의 서브맵으로 병합(fuse)하는 독창적인 파이프라인을 도입했다. 이 병합 과정을 통해 맵 데이터의 중복성을 제거하고 전체 맵의 크기 증가를 억제하는 동시에, 지역적인 맵의 일관성을 강화하고자 했다.4 본 고찰에서는 C-blox의 수학적 기반이 되는 TSDF의 원리부터 시작하여, 그 독창적인 서브맵 기반 아키텍처와 데이터 처리 흐름, 그리고 후속 연구인 Voxgraph로의 발전 과정과 타 매핑 기법과의 비교 분석을 통해 C-blox가 대규모 밀집 매핑 분야에 남긴 학술적 기여와 그 유산을 심도 있게 분석하고자 한다.

## 1.  체적 매핑의 수학적 원리: Truncated Signed Distance Fields (TSDF)

C-blox의 핵심 데이터 구조인 TSDF를 이해하기 위해서는 먼저 그 기반이 되는 부호 거리 필드(Signed Distance Field, SDF)의 개념을 파악해야 한다. SDF는 3차원 공간의 기하학적 구조를 표현하는 강력한 암시적(implicit) 방식으로, C-blox는 이를 현실적인 센서 데이터에 맞게 변형하여 사용한다.

### 1.1  암시적 표면 표현으로서의 부호 거리 필드(Signed Distance Field)

부호 거리 필드(SDF)는 3차원 공간상의 임의의 점 $ \mathbf{x} \in \mathbb{R}^3 $에 대해, 그 점에서 가장 가까운 환경 표면($ \partial\Omega $)까지의 유클리드 거리를 스칼라 값으로 가지는 함수 $ g(\mathbf{x}) $로 정의된다.11 SDF의 가장 큰 특징은 단순한 거리 정보뿐만 아니라 '부호'를 통해 공간의 위상 정보를 함께 제공한다는 점이다. 일반적으로 표면을 기준으로 센서가 위치한 자유 공간(free space)에 있는 점들은 양수 값을, 표면 너머의 점유 공간(occupied space)에 있는 점들은 음수 값을 갖도록 정의된다.12

이러한 정의에 따라, SDF에서 값이 0이 되는 모든 점의 집합, 즉 영점 교차(zero-crossing)는 $ \{\mathbf{x} \in \mathbb{R}^3 | g(\mathbf{x}) = 0\} $으로 표현되며, 이는 곧 실제 환경의 표면 자체를 의미한다. 이처럼 표면을 방정식의 해로 간접적으로 정의하는 방식을 암시적 표면 표현이라고 한다. 또한, SDF의 그래디언트 벡터 $ \nabla g(\mathbf{x}) $는 각 점에서 표면을 향하는 가장 가파른 방향을 가리키며, 특히 표면 근처에서는 표면의 법선 벡터(normal vector)와 방향이 일치한다. 이는 표면의 방향성을 파악하거나 최적화 기반 경로 계획 시 충돌 회피를 위한 힘을 계산하는 데 매우 유용하게 사용될 수 있다.12

### 1.2  TSDF의 정의: 절단(Truncation)의 역할과 의미

이론적으로 SDF는 강력한 표현 방식이지만, 실제 로봇 응용에 그대로 적용하기에는 몇 가지 현실적인 문제가 있다. 첫째, 3차원 공간 전체에 대해 모든 점의 거리 값을 저장하는 것은 막대한 메모리를 요구한다. 둘째, 깊이 센서는 측정 범위가 제한적이고 노이즈가 포함되어 있어, 표면에서 멀리 떨어진 공간의 정확한 SDF 값을 계산하는 것이 어렵거나 불필요하다.

Truncated Signed Distance Field (TSDF)는 이러한 문제를 해결하기 위해 도입된 실용적인 변형이다.12 TSDF의 핵심 아이디어는 우리가 관심을 가지는 영역이 주로 표면 근처의 좁은 대역(narrow band)이라는 점에서 착안한다. 따라서 이 대역 내에서만 SDF 값을 정확하게 계산하고, 대역을 벗어나는 모든 점에 대해서는 미리 정해진 최대 절단 값($ +h $) 또는 최소 절단 값($ -h $)으로 값을 고정(clamp)한다.12 이 과정을 '절단(truncation)'이라고 부른다.

수학적으로 TSDF 함수 $ g(\mathbf{x}) $는 다음과 같이 정의할 수 있다 11:
$$
g(\mathbf{x}) = \begin{cases} 
\text{min}(d(\mathbf{x}, \partial\Omega), h), & \text{if } \mathbf{x} \notin \Omega \\
-\text{min}(d(\mathbf{x}, \partial\Omega), h), & \text{if } \mathbf{x} \in \Omega 
\end{cases}
$$
여기서 $ \Omega $는 점유된 공간의 집합, $ \partial\Omega $는 그 경계인 표면, $ d(\mathbf{x}, \partial\Omega) $는 점 $ \mathbf{x} $에서 표면까지의 최단 유클리드 거리, 그리고 $ h $는 양의 상수인 절단 거리(truncation distance)를 의미한다. 이 절단 과정을 통해 TSDF는 표면 재구성에 필수적인 정보만을 효율적으로 저장하고 관리할 수 있게 된다.

### 1.3  프로젝티브 TSDF 통합 및 가중 평균 갱신 규칙

로봇이 움직이면서 새로운 깊이 센서 데이터를 획득하면, 이 정보를 기존의 TSDF 맵에 융합(fusion) 또는 통합(integration)해야 한다. TSDF 맵은 공간을 일정한 크기의 작은 정육면체, 즉 복셀(voxel)로 나누어 각 복셀의 중심점에 거리 값과 가중치(weight)를 저장하는 방식으로 구현된다.

새로운 깊이 이미지가 들어오면, 이미지의 각 픽셀에 대해 센서 원점으로부터 해당 픽셀이 관측한 3D 지점까지 광선(ray)을 쏜다고 가정한다. 이 광선이 통과하는 경로상의 각 복셀 $ \mathbf{x} $에 대해, 센서 광선을 따라 측정한 표면까지의 거리인 '프로젝티브 거리(projective distance)' $ \hat{D}(\mathbf{x}) $를 계산한다.12 이 값은 해당 관측에 대한 TSDF 값의 추정치가 된다.

단일 관측은 센서 노이즈를 포함할 수 있으므로, TSDF는 여러 프레임에 걸친 관측 값을 통계적으로 융합하여 노이즈를 줄이고 표면의 정확도를 높인다. 이를 위해 가장 널리 사용되는 방법이 가중 평균(weighted average) 갱신 규칙이다.5 각 복셀은 이전에 누적된 거리 값($ D_n $)과 가중치($ W_n $)를 가지고 있으며, 새로운 관측으로 계산된 프로젝티브 거리($ \hat{D} $)와 그에 해당하는 가중치($ \hat{W} $)가 들어오면 다음과 같이 값을 갱신한다 12:

**거리 값 갱신:**
$$
D_{n+1}(\mathbf{x}) = \frac{D_n(\mathbf{x})W_n(\mathbf{x}) + \hat{D}(\mathbf{x})\hat{W}(\mathbf{x})}{W_n(\mathbf{x}) + \hat{W}(\mathbf{x})}
$$
**가중치 갱신:**
$$
W_{n+1}(\mathbf{x}) = \min(W_n(\mathbf{x}) + \hat{W}(\mathbf{x}), W_{\max})
$$
여기서 아래 첨자 $ n $은 갱신 횟수를 나타낸다. 새로운 측정에 대한 가중치 $ \hat{W}(\mathbf{x}) $는 센서 모델에 따라 다르게 설정될 수 있으며, 예를 들어 측정 거리가 멀수록, 또는 센서 광선과 표면 법선 벡터가 이루는 각도가 클수록 낮은 가중치를 부여할 수 있다. 가중치의 누적 합이 특정 최대값 $ W_{\max} $를 넘지 않도록 제한하는 것은, 오래된 관측의 영향력을 점차 줄여 환경의 동적인 변화(예: 물체가 사라지거나 이동하는 경우)에 맵이 적응할 수 있도록 하는 역할을 한다. 이처럼 TSDF는 단순한 기하학적 표현을 넘어, 불확실성을 다루고 시간에 따라 변화하는 환경에 대응하기 위한 정교한 수학적 메커니즘을 내포하고 있다.

## 2.  C-blox의 설계 사상과 아키텍처

C-blox는 기존의 강력한 체적 매핑 라이브러리인 Voxblox의 한계를 인식하고, 이를 극복하기 위해 서브맵이라는 새로운 추상화 계층을 도입한 시스템이다. C-blox의 아키텍처는 Voxblox의 효율적인 복셀 관리 능력을 계승하면서도, 대규모 환경에서의 확장성과 일관성 문제를 해결하기 위한 독창적인 설계 사상을 담고 있다.

### 2.1  Voxblox의 확장: 서브맵 개념의 도입

C-blox의 구조를 이해하기 위해서는 먼저 그 기반이 되는 Voxblox에 대한 이해가 필요하다. Voxblox는 ETH 취리히에서 개발된 오픈소스 C++ 라이브러리로, TSDF와 ESDF(Euclidean Signed Distance Field) 기반의 체적 매핑을 위해 설계되었다.14 Voxblox의 주요 특징 중 하나는 '복셀 해싱(voxel hashing)' 기법의 사용이다.16 이는 전체 맵 공간을 일정한 크기의 복셀 블록(block) 단위로 나누고, 각 블록의 위치를 해시 테이블에 저장하여 관리하는 방식이다. 이를 통해 맵이 동적으로 성장해야 할 때 필요한 메모리만 할당하고, 특정 위치의 복셀에 $ O(1) $의 시간 복잡도로 빠르게 접근할 수 있다.14

그러나 Voxblox는 근본적으로 단일 전역 맵을 관리하는 구조를 가지고 있다. 따라서 로봇이 장시간, 넓은 영역을 탐사하며 발생하는 주행계의 누적 오차, 즉 드리프트 문제로부터 자유롭지 못하다. 맵 전체가 하나의 강체(rigid body)로 연결되어 있기 때문에, 루프 폐쇄 등으로 과거의 포즈가 수정되더라도 이미 통합된 TSDF 볼륨을 소급하여 수정하는 것은 매우 비효율적이거나 불가능에 가깝다.

C-blox는 바로 이 문제를 해결하기 위해 Voxblox 위에 '서브매핑(sub-mapping)'이라는 새로운 계층을 추가한 확장 라이브러리이다.10 C-blox의 핵심 아이디어는 거대한 단일 맵을 관리하는 대신, 전체 환경을 여러 개의 독립적인 TSDF 서브맵(혹은 서브볼륨)의 집합(collection of submaps)으로 표현하는 것이다.9 각 서브맵은 그 자체로 하나의 작은 Voxblox 맵이며, 월드 좌표계에 대한 고유한 포즈(위치와 방향)를 가진다. 로봇이 새로운 영역으로 이동하면 새로운 서브맵이 생성되고, 이후의 센서 데이터는 이 활성화된 서브맵에 통합된다. 이러한 구조를 통해 드리프트 오차의 영향이 현재 활성화된 서브맵과 서브맵 간의 상대 포즈에 국한되도록 만들어, 오차의 전역적인 전파를 효과적으로 차단한다.

### 2.2  데이터 흐름 및 핵심 구성요소

C-blox 시스템의 전체적인 데이터 처리 흐름은 다음과 같은 핵심 구성요소들의 상호작용으로 이루어진다.

1. **입력 (Input)**: 시스템의 입력은 깊이 센서(예: RGB-D 카메라, LiDAR)로부터 실시간으로 수집되는 3D 포인트 클라우드와, 각 데이터가 수집된 시점의 카메라(센서) 포즈 추정치이다.5 이 포즈 추정치는 일반적으로 시각적 주행계(Visual Odometry)나 관성 센서가 결합된 SLAM 프론트엔드로부터 제공된다.
2. **지역화 (Localization)**: C-blox는 자체적으로 포즈를 추정하지 않고, 외부의 SLAM 시스템(예: ORB-SLAM)에 의존한다. 외부 SLAM 시스템은 입력된 시각 정보로부터 특징점을 추출하고, 이를 기반으로 카메라의 포즈를 추적하며, 번들 조정(Bundle Adjustment)과 같은 최적화 기법을 통해 궤적의 정확도를 높인다.4 C-blox는 이렇게 정제된 포즈 정보를 입력받아 맵핑에 사용한다.
3. **TSDF 통합 (Integration)**: 신뢰도 높은 포즈 정보가 주어지면, 해당 포즈를 기준으로 입력된 3D 포인트 클라우드를 현재 활성화된(active) 서브맵의 TSDF 볼륨에 통합한다. 이 과정은 섹션 2.3에서 설명한 가중 평균 갱신 규칙을 따르며, Voxblox의 효율적인 통합기(integrator)를 그대로 활용한다.
4. **서브맵 관리 (Submap Management)**: C-blox의 핵심적인 역할이다. 로봇의 움직임을 추적하며, 현재 포즈가 기존에 생성된 어떤 서브맵의 경계에도 속하지 않을 경우, 새로운 서브맵을 생성하여 활성화한다. 각 서브맵은 고유 ID와 월드 좌표계에 대한 변환 행렬($ T_{W,S} \in SE(3) $)로 표현되는 자신의 포즈를 관리한다.9 전체 맵은 이러한 서브맵들의 리스트 또는 해시 맵 형태로 유지된다.
5. **서브맵 병합 (Fusion)**: 맵의 일관성을 유지하고 데이터 크기를 관리하기 위해, 특정 조건을 만족하는 여러 서브맵을 하나의 서브맵으로 병합하는 과정을 수행한다. 이는 C-blox의 가장 독창적인 기여 중 하나이며, 다음 절에서 상세히 다룬다.4

### 2.3  맵 일관성 유지를 위한 서브맵 병합 전략

C-blox는 맵의 무한한 성장을 억제하고 지역적 일관성을 유지하기 위한 핵심 메커니즘으로 서브맵 병합 파이프라인을 제안했다.4 로봇이 동일한 장소를 반복적으로 방문하면 여러 개의 중첩된 서브맵이 생성될 수 있는데, 이를 방치하면 데이터 중복으로 인해 메모리 낭비가 발생하고 맵의 일관성이 저해될 수 있다. 서브맵 병합은 이러한 문제를 해결하기 위한 과정으로, 다음과 같은 단계로 진행된다.

1. **1단계: 안정 영역 식별 (Identifying Stable Regions)**: 시스템은 먼저 맵에서 병합 후보가 될 만한 영역을 탐색한다. 이는 주로 공간적으로 크게 중첩되고, 여러 서브맵에 걸쳐 충분한 관측이 누적된 '안정적인(stable)' 영역을 식별하는 과정이다.
2. **2단계: 상대적 위치 정확도 측정 (Measuring Relative Localization Certainty)**: 병합 후보로 선정된 서브맵들 간의 상대적인 위치 관계가 얼마나 정확하고 신뢰할 수 있는지를 정량적으로 평가한다. C-blox는 이 정보를 지역화 모듈(localization module)이 제공하는 희소 특징점 맵의 공분산(covariance) 정보로부터 추출한다.5 즉, 두 서브맵의 포즈를 결정하는 데 공통적으로 사용된 시각적 특징점들이 많고, 이들의 위치 불확실성이 낮을수록 두 서브맵 간의 상대적 위치 정확도는 높다고 판단한다.
3. **3단계: 병합 수행 (Performing Fusion)**: 두 서브맵 간의 상대적 위치 정확도가 사전에 정의된 임계값을 초과하면, 즉 두 서브맵이 서로에 대해 매우 잘 정렬되어 있다고 판단되면 병합을 수행한다. 병합 과정은 하나의 서브맵(기준 서브맵)을 선택하고, 다른 서브맵의 모든 TSDF 복셀 정보를 기준 서브맵의 좌표계로 변환하여 가중 평균 방식으로 통합하는 절차로 이루어진다. 병합이 완료되면 기존의 서브맵들은 비활성화되거나 삭제되고, 새로운 통합 서브맵이 이를 대체한다.

이러한 병합 전략은 c-blox가 단순히 맵을 분할하는 것을 넘어, 맵의 구조를 동적으로 재구성하여 효율성과 일관성을 동시에 추구하는 능동적인 매핑 시스템임을 보여준다. 하지만 이 병합 과정은 지역적인 최적화에 초점을 맞추고 있으며, 대규모 루프 폐쇄로 인한 전역적인 오차 보정 문제에 대해서는 여전히 한계를 가진다.

## 3.  C-blox의 계승과 발전: Voxgraph

C-blox는 서브맵 개념을 도입하여 대규모 체적 매핑의 확장성 문제를 해결하는 중요한 돌파구를 마련했다. 그러나 지역적 일관성과 데이터 관리 효율성에 초점을 맞춘 C-blox의 설계는, 필연적으로 전역적 일관성(global consistency) 확보라는 또 다른 과제를 남겼다. 이 과제를 해결하기 위해 C-blox의 개발팀은 그 아키텍처를 계승하고 발전시켜 Voxgraph라는 새로운 프레임워크를 탄생시켰다.

### 3.1  전역적 일관성을 향한 도전

C-blox의 서브맵 병합 전략은 인접하고 잘 정렬된 서브맵들을 하나로 합쳐 맵의 성장을 억제하는 데 효과적이다. 하지만 이 병합 연산은 기본적으로 비가역적(irreversible)이다. 한번 병합된 서브맵은 더 이상 독립적인 개체가 아니므로, 나중에 대규모 루프 폐쇄가 발생하여 과거의 궤적에 큰 수정이 필요하게 되더라도 병합된 서브맵 내부의 기하학적 구조를 분리하여 재정렬하기는 매우 어렵다. 즉, C-blox는 맵의 '성장'은 관리할 수 있지만, 이미 생성된 맵의 대규모 '뒤틀림'을 바로잡는 데에는 근본적인 한계를 가진다.

전역적으로 일관된 맵을 구축하기 위해서는, 새로운 정보(특히 루프 폐쇄)가 들어왔을 때 모든 서브맵의 포즈를 동시에 재조정하여 전체 맵의 오차를 최소화하는 전역 최적화(global optimization) 과정이 필수적이다. 이는 C-blox가 제공하는 서브맵 관리 기능을 넘어, 모든 서브맵 간의 관계를 수학적으로 모델링하고 최적화하는 별도의 백엔드(back-end) 시스템을 요구한다.17

### 3.2  포즈 그래프 최적화를 통한 서브맵 정합

Voxgraph는 C-blox의 서브맵 관리 기능 위에 포즈 그래프 최적화(pose graph optimization)라는 강력한 백엔드 메커니즘을 결합하여 전역적 일관성 문제를 해결한 프레임워크이다.17 C-blox가 맵을 서브맵이라는 이산적인 단위로 구조화하는 역할을 한다면, Voxgraph는 이 서브맵들 간의 기하학적 관계를 최적화하여 전체 맵을 일관된 상태로 정렬하는 역할을 수행한다.

Voxgraph의 핵심은 포즈 그래프를 구성하고 이를 최적화하는 데 있다.

- **포즈 그래프 구성**: Voxgraph에서 전체 맵은 하나의 거대한 그래프로 표현된다. 이 그래프에서 각 TSDF 서브맵의 6-DoF 포즈(위치와 방향)는 최적화해야 할 변수를 나타내는 노드(node)가 된다. 그리고 이 노드들 사이의 기하학적 관계는 엣지(edge)로 표현되는 제약 조건(constraint)에 의해 정의된다. Voxgraph는 주로 세 가지 종류의 제약 조건을 사용한다 17:
  1. **주행계 제약 (Odometry Constraint)**: 로봇의 움직임에 따라 순차적으로 생성된 서브맵들 사이의 상대적 변환 관계이다. 이는 SLAM 프론트엔드에서 제공하는 주행계 정보로부터 얻어진다.
  2. **루프 폐쇄 제약 (Loop Closure Constraint)**: 로봇이 과거에 방문했던 장소를 재인식했을 때, 현재 서브맵과 과거의 서브맵 사이에 생성되는 상대적 변환 관계이다. 이는 맵의 누적 오차를 극적으로 줄이는 데 가장 중요한 역할을 한다.
  3. **서브맵 등록 제약 (Submap Registration Constraint)**: 루프 폐쇄가 아니더라도 공간적으로 중첩되는 모든 서브맵 쌍 사이에 설정되는 상대적 변환 관계이다. 이 제약은 맵의 지역적 정밀도를 높이고 전체 그래프를 더 강인하게 연결해준다.
- **대응점 없는 정합 (Correspondence-Free Registration)**: Voxgraph의 가장 중요한 기술적 혁신 중 하나는 중첩되는 서브맵 간의 정합을 수행하는 방식에 있다. 전통적인 ICP(Iterative Closest Point)와 같은 방식은 두 데이터셋 사이에서 대응되는 점(correspondence)을 명시적으로 찾아야 하므로 계산 비용이 높고 지역 최솟값에 빠지기 쉽다. 반면, Voxgraph는 각 서브맵이 이미 내부에 풍부한 거리 정보(SDF)를 담고 있다는 점을 적극적으로 활용한다. 한 서브맵의 표면 근처 점들을 다른 서브맵의 SDF에 투영하여, 대응점을 찾을 필요 없이 직접 정합 오차를 계산한다. 이 '대응점 없는' 방식은 계산적으로 매우 효율적이어서, 수백 개의 서브맵 간 제약 조건이 존재하는 대규모 포즈 그래프라도 새로운 서브맵이 추가될 때마다 수 초 내에 전체 최적화를 완료하는 것을 가능하게 한다.17

결론적으로, Voxgraph는 C-blox가 제공하는 확장 가능한 서브맵 구조를 그대로 활용하면서, 그 위에 전역 최적화라는 강력한 엔진을 장착한 시스템이다. 이로써 Voxblox에서 C-blox, 그리고 Voxgraph로 이어지는 기술적 흐름은 지역적 밀집 매핑(Voxblox), 맵의 분할 및 관리(C-blox), 그리고 전역적 일관성 확보(Voxgraph)라는, 대규모 SLAM의 각기 다른 난제를 단계적으로 해결하는 모범적인 아키텍처의 진화 과정을 보여준다.

## 4.  타 매핑 기법과의 비교 분석

C-blox와 그 기반 기술인 TSDF의 특성을 더 명확히 이해하기 위해서는, 로보틱스 분야에서 널리 사용되는 다른 체적 매핑 기법들과의 비교 분석이 필수적이다. 특히, 점유 격자 맵의 대표 주자인 OctoMap과의 비교는 두 기술의 근본적인 철학 차이를 드러내며, 최신 SLAM 기술과의 관계를 살펴보는 것은 C-blox가 제시한 패러다임의 지속적인 영향력을 보여준다.

### 4.1  C-blox/Voxblox 대 OctoMap

C-blox/Voxblox와 OctoMap은 자율 로봇을 위한 3차원 환경 표현이라는 동일한 목표를 추구하지만, 그 접근 방식과 핵심 철학에서 뚜렷한 차이를 보인다.16

- **핵심 표현 방식의 차이**: 가장 근본적인 차이는 복셀에 저장하는 정보의 종류에 있다. Voxblox/c-blox가 사용하는 TSDF는 각 복셀에 가장 가까운 **표면까지의 부호 거리**를 저장한다. 이는 환경의 기하학적 표면을 매우 정밀하고 매끄럽게 복원하는 데 초점을 맞춘 표현 방식이다.16 반면, OctoMap은 각 복셀이 장애물에 의해 **점유되었을 확률(occupancy probability)**을 로그 오즈(log-odds) 형태로 저장한다. 이는 공간을 '점유(occupied)', '자유(free)', '미지(unknown)'의 세 가지 상태로 명확하게 구분하여 표현하는 데 중점을 둔다.21

- **자료 구조**: Voxblox는 앞서 언급했듯이 복셀 블록의 위치를 저장하는 해시 맵을 사용하여 평균적으로 $ O(1) $의 빠른 접근 속도를 제공한다.16 반면, OctoMap은 공간을 계층적으로 분할하는 

  **옥트리(octree)** 자료 구조를 사용한다. 옥트리는 동일한 점유 상태를 갖는 넓은 공간(예: 텅 빈 방)을 단일 노드로 압축하여 표현할 수 있어 메모리 효율성이 매우 높다는 장점이 있다. 그러나 특정 복셀에 접근하기 위해서는 트리의 루트부터 탐색해야 하므로 $ O(\log n) $의 시간 복잡도를 가진다.16

- **경로 계획 적용성**: 이 차이는 로봇의 경로 계획 알고리즘에 직접적인 영향을 미친다. TSDF로부터 파생되는 ESDF(Euclidean Signed Distance Field)는 모든 지점에서 가장 가까운 장애물까지의 실제 유클리드 거리를 제공한다. 이는 CHOMP와 같은 최적화 기반 경로 계획 알고리즘이 충돌 비용 함수의 그래디언트를 계산하는 데 필수적인 정보를 직접적으로 제공한다.16 반면, OctoMap은 점유 여부만 알려주기 때문에, 이러한 최적화 기반 계획을 적용하기 위해서는 맵 전체를 스캔하여 ESDF를 생성하는 별도의 후처리 과정이 필요하다.16

- **성능 및 정확도**: 실제 데이터셋을 사용한 실험에 따르면, 일반적으로 TSDF를 구축하는 것이 OctoMap을 구축하는 것보다 빠른 경향을 보이며, TSDF로부터 직접 ESDF를 증분적으로(incrementally) 계산하는 것이 점유 맵으로부터 일괄적으로(in batch) 계산하는 것보다 더 정확한 거리 필드를 생성하는 것으로 나타났다.16

아래 표는 두 기법의 주요 특징을 요약하여 비교한다.

**표 1: 체적 매핑 표현 방식 비교 분석**

| 특징 (Feature)                  | C-blox/Voxblox (TSDF)                                        | OctoMap (Occupancy Grid)                                     |
| ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **핵심 표현 단위**              | 표면까지의 부호 거리 (Signed distance to the nearest surface) | 복셀의 점유 확률 (Probability of a voxel being occupied)     |
| **자료 구조**                   | 복셀 블록의 해시맵 (Hash map of voxel blocks)                | 옥트리 (Octree)                                              |
| **메모리 사용 프로파일**        | 표면 영역에 비례하여 메모리 사용 (Grows with surface area)   | 계층적 압축으로 넓은 빈 공간/미지 공간에 효율적 (Efficient for large open/unknown spaces) |
| **갱신/조회 복잡도**            | 평균 $ O(1) $ (Amortized $ O(1) $)                           | $ O(\log n) $, $ n $은 차원 당 복셀 수 (Logarithmic in voxels per dimension) |
| **저장 정보**                   | 표면 근처의 암시적 기하 구조, 자유 공간 (Implicit surface geometry, free space near surface) | 모든 공간의 점유/자유/미지 상태 (Occupied, Free, Unknown states for all space) |
| **표면 재구성 정밀도**          | 높음, 매끄럽고 상세한 메시 생성 (High, produces smooth and detailed meshes) | 낮음, 표면이 계단 현상(aliasing)을 보일 수 있음 (Prone to aliasing) |
| **그래디언트 기반 계획 적합성** | 탁월함, 최적화에 필요한 거리/그래디언트 직접 제공 (Excellent, directly provides distance/gradient) | 부적합, ESDF 생성을 위한 별도의 후처리 필요 (Poor, requires post-processing for ESDF) |

### 4.2  현대적 SLAM 기법과의 관계

C-blox가 제시한 '밀집 표현을 위한 서브맵 기반 관리'라는 아키텍처 패러다임은 TSDF라는 특정 표현 방식에 국한되지 않고, 후대의 다양한 SLAM 시스템에 지속적인 영향을 미치고 있다. 특히 최근 컴퓨터 비전과 그래픽스 분야에서 혁신을 이끌고 있는 신경 방사 필드(Neural Radiance Fields, NeRF)나 3D 가우시안 스플래팅(3D Gaussian Splatting, 3DGS)을 기반으로 하는 최신 SLAM 시스템에서도 이와 유사한 개념이 핵심적인 역할을 수행하고 있다.3

이들 시스템은 환경의 기하학적 구조와 외관을 표현하기 위해 거대한 단일 신경망을 사용하는 대신, 여러 개의 작고 가벼운 신경망(예: 작은 다층 퍼셉트론(MLP) 또는 지역적인 가우시안 집합)을 일종의 '신경 서브맵(neural submap)'으로 활용한다.3 로봇이 새로운 영역을 탐사하면 새로운 신경 서브맵이 할당되고, 이는 해당 지역의 표현을 담당한다. 이러한 접근 방식은 다음과 같은 여러 장점을 가진다. 첫째, 단일 거대 신경망을 학습시키는 것보다 여러 개의 작은 신경망을 독립적으로 학습시키는 것이 훨씬 빠르고 효율적이다. 둘째, C-blox와 마찬가지로 대규모 환경에서의 확장성 문제를 해결할 수 있다. 셋째, 루프 폐쇄가 발생했을 때, 전체 신경망을 재학습할 필요 없이 서브맵들의 포즈 그래프만을 최적화하여 전역적 일관성을 확보할 수 있다.

이는 C-blox가 TSDF라는 고전적인 체적 표현 방식에 적용했던 '분할 정복' 전략이, 표현 방식의 패러다임이 완전히 바뀐 현대의 신경망 기반 SLAM 시스템에서도 여전히 유효하고 강력한 해결책임을 명확히 보여준다. 따라서 C-blox는 단순히 과거의 한 시스템이 아니라, 대규모 밀집 매핑 문제를 해결하기 위한 근본적인 아키텍처 설계 원칙을 제시한 선구자로서 그 가치를 인정받아야 한다.

## 5.  적용 사례 및 한계

C-blox는 이론적 제안에 그치지 않고, 실제 로봇 시스템과 표준 데이터셋에 적용되어 그 실용성과 효용성을 입증했다. 그러나 동시에 명확한 기술적 한계 또한 가지고 있으며, 이를 이해하는 것은 C-blox의 역할을 정확히 평가하고 후속 기술의 발전 방향을 파악하는 데 중요하다.

### 5.1  실제 로봇 시스템 적용 사례: MAV 및 KITTI 데이터셋

C-blox의 실용성을 보여주는 대표적인 사례는 소형 무인 항공기(Micro Aerial Vehicle, MAV)에 탑재되어 실시간으로 임무를 수행한 것이다.4 MAV는 크기와 무게, 탑재 가능한 컴퓨팅 자원에 제약이 많은 플랫폼이다. 이러한 제한된 환경에서 C-blox가 실시간으로 일관성 있는 3D 밀집 맵을 성공적으로 생성했다는 사실은, 이 시스템이 CPU 기반으로도 충분히 효율적으로 동작할 수 있도록 설계되었음을 입증한다. 이는 고가의 GPU 없이도 현장에서 실시간 3D 매핑을 수행할 수 있는 가능성을 열어주었다는 점에서 중요한 의미를 가진다.

또 다른 중요한 적용 사례는 자율 주행 연구의 표준 벤치마크로 널리 사용되는 KITTI 데이터셋을 이용한 실험이다.10 C-blox의 공식 GitHub 저장소에는 KITTI 데이터셋의 LiDAR 데이터와 정답(ground-truth) 포즈 정보를 입력으로 받아 서브맵을 생성하고, 이를 ROS의 시각화 도구인 RViz에 표시하는 예제 코드가 포함되어 있다.10 이 예제는 드리프트가 전혀 없는 이상적인 조건에서 서브맵이 어떻게 생성, 분할, 관리되는지를 명확하게 보여줌으로써, 개발자와 연구자들이 C-blox의 핵심 메커니즘을 직관적으로 이해하고 자신의 연구에 활용할 수 있도록 돕는 훌륭한 교육 자료 역할을 한다. 다만, 이 예제는 시스템의 핵심 과제인 '불완전한 포즈 추정' 상황을 다루지 않으므로, C-blox의 성능 자체를 평가하기보다는 그 작동 원리를 시연하는 데 목적이 있다고 보아야 한다.

### 5.2  C-blox 시스템의 기술적 한계와 제약 조건

C-blox는 여러 중요한 기여를 했음에도 불구하고, 그 자체만으로는 완전한 SLAM 시스템이라고 보기 어려운 몇 가지 명확한 기술적 한계를 내포하고 있다.

1. **전역 일관성의 부재**: 가장 핵심적인 한계로, 앞서 4.1절에서 상세히 논의했듯이 C-blox의 서브맵 병합 메커니즘은 지역적 일관성 확보에 초점을 맞추고 있어 대규모 루프 폐쇄로 인한 전역적인 드리프트 오차를 보정할 수 없다. 이 문제는 C-blox를 맵 관리 계층으로 사용하고 별도의 포즈 그래프 최적화 백엔드를 추가한 후속 연구인 Voxgraph를 통해 해결되었다.17 따라서 C-blox는 독립적인 SLAM 솔루션이라기보다는, 완전한 전역 일관성 SLAM 시스템을 구축하기 위한 중요한 중간 단계 혹은 구성 요소로 이해하는 것이 타당하다.
2. **정적 환경 가정**: C-blox를 포함한 대부분의 초기 TSDF 기반 매핑 시스템은 환경이 정적(static)이라는 강한 가정을 기반으로 한다. 만약 환경 내에 사람, 차량, 다른 로봇과 같이 움직이는 동적 객체가 존재할 경우, 이들의 움직임이 맵에 그대로 통합되어 잔상(artifact)이나 유령 물체(ghost object)를 남기게 된다. 이는 맵의 품질을 심각하게 저하시키고, 로봇의 경로 계획이나 장애물 회피에 잘못된 정보를 제공할 수 있다.
3. **메모리 관리의 한계**: 서브맵 병합 전략은 동일한 공간을 반복적으로 관찰할 때 발생하는 데이터 중복을 줄여 맵의 성장을 '억제'하는 효과가 있다. 그러나 로봇이 이전에 방문한 적이 없는 새로운 영역을 계속해서 탐사하는 시나리오(예: 광활한 지역의 탐사 임무)에서는 새로운 서브맵이 지속적으로 생성될 수밖에 없다. 이 경우, 병합이 발생할 기회가 없으므로 서브맵의 수는 계속해서 증가하여 결국 시스템의 메모리 한계에 도달하게 된다.
4. **프론트엔드 의존성**: C-blox는 맵핑을 수행할 뿐, 위치 추정은 외부의 SLAM 프론트엔드(예: 시각적 주행계)에 전적으로 의존한다. 따라서 최종적으로 생성되는 맵의 품질은 입력되는 포즈 추정치의 정확도에 매우 큰 영향을 받는다. 만약 조명이 급격하게 변하거나, 특징점이 거의 없는 단조로운 환경(textureless environment), 또는 움직임이 매우 빨라 모션 블러가 심한 상황에서는 프론트엔드 트래킹이 실패할 수 있으며, 이 경우 C-blox 역시 맵핑을 중단하거나 심각하게 왜곡된 맵을 생성하게 된다.8

이러한 한계들은 C-blox가 모든 문제를 해결하는 만능 솔루션이 아님을 보여준다. 오히려 C-blox는 대규모 밀집 매핑이라는 거대한 문제 공간에서 '확장성'과 '지역적 일관성'이라는 특정 하위 문제를 해결하는 데 집중한, 목적이 명확한 전문 라이브러리로서 그 가치를 평가해야 한다.

## 6.  결론 및 미래 전망

C-blox는 대규모 3차원 밀집 매핑 분야에서 확장성과 일관성이라는 두 가지 핵심 난제를 해결하기 위한 중요한 이정표를 제시한 시스템이다. 그 학술적 기여는 후속 연구들에 깊은 영향을 미쳤으며, C-blox가 제시한 아키텍처 패러다임은 오늘날 최신 매핑 기술의 발전 방향에도 여전히 유효한 통찰을 제공한다.

### 7.1. C-blox의 학술적 기여 및 유산 요약

C-blox가 로보틱스 및 컴퓨터 비전 커뮤니티에 남긴 학술적 기여와 유산은 다음과 같이 요약할 수 있다.

- **서브맵 패러다임의 도입과 확장성 확보**: C-blox의 가장 큰 기여는 TSDF 기반의 밀집 체적 매핑에 서브맵 개념을 성공적으로 도입하여 **확장성** 문제를 해결할 수 있는 실질적인 방향을 제시한 것이다.4 거대한 단일 맵 대신 관리 가능한 작은 단위의 서브맵 집합으로 환경을 표현함으로써, 맵 데이터의 크기와 계산 복잡도가 환경의 크기에 따라 무한정 증가하는 것을 방지했다.
- **지역적 일관성 유지를 위한 실용적 해법 제시**: 단순히 맵을 분할하는 것을 넘어, '서브맵 병합'이라는 구체적인 메커니즘을 통해 맵의 중복성을 줄이고 **지역적 일관성**을 유지하는 실용적인 방법을 제안했다.5 이는 확률적 측정치를 기반으로 맵의 안정적인 부분을 식별하고 동적으로 맵 구조를 재구성하는 능동적인 맵 관리 기법의 가능성을 보여주었다.
- **모듈식 SLAM 아키텍처의 효용성 입증**: C-blox는 효율적인 지역 밀집 매핑을 담당하는 Voxblox와 전역 최적화를 수행하는 Voxgraph 사이를 잇는 중요한 기술적 다리 역할을 수행했다. 이 과정에서 밀집 데이터 통합(프론트엔드)과 희소 포즈 그래프 최적화(백엔드)를 명확히 분리하는 모듈식 SLAM 아키텍처의 효용성을 입증했다.17 이러한 설계 철학은 복잡한 SLAM 시스템을 보다 체계적으로 설계하고 개발할 수 있는 길을 열었다.
- **오픈소스 생태계에의 기여**: C-blox는 그 소스 코드를 공개함으로써 9 전 세계의 수많은 후속 연구자들이 대규모 밀집 매핑 연구를 시작할 수 있는 견고한 기반을 제공했다. 이는 학술적 투명성을 높이고 커뮤니티 전체의 발전을 가속화하는 데 크게 기여했다.

### 6.1  대규모 체적 매핑 기술의 미래 발전 방향

C-blox와 그 후속 연구들이 닦아놓은 길 위에서, 대규모 체적 매핑 기술은 더욱 정교하고 지능적인 방향으로 발전하고 있다.

- **GPU 가속화를 통한 실시간 성능 극대화**: C-blox가 CPU 기반으로 실시간성을 확보하고자 노력했다면, 현대의 매핑 시스템들은 병렬 처리에 특화된 GPU의 막강한 계산 능력을 적극적으로 활용하고 있다. NVIDIA의 nvBlox나 coVoxSLAM과 같은 최신 라이브러리들은 TSDF 통합 및 ESDF 생성 과정을 GPU에서 가속하여, 훨씬 더 높은 해상도의 맵을 더 빠른 속도로 구축하는 것을 목표로 한다.20
- **의미론적 정보와의 융합**: 미래의 로봇은 '벽이 있다'는 기하학적 정보를 넘어 '저것은 문이고, 저것은 의자'라는 의미론적(semantic) 정보를 이해해야 한다. 이에 따라, 3D 맵에 각 복셀이나 객체가 무엇을 의미하는지에 대한 정보를 함께 저장하는 의미론적 매핑(semantic mapping) 연구가 활발히 진행되고 있다.25 특히, C-blox가 제시한 서브맵 구조는 '의자', '테이블'과 같은 객체 단위로 맵을 분할하고 관리하는 데 이상적인 구조를 제공하여 의미론적 정보 관리를 용이하게 한다.27
- **동적 환경에 대한 강인성 확보**: 현실 세계는 정적이지 않다. 미래의 매핑 시스템은 사람, 다른 로봇, 움직이는 가구 등 동적 객체들을 실시간으로 감지하고 추적하며, 이를 맵 상에서 적절히 처리(예: 일시적으로 제거하거나 미래 위치를 예측)하는 능력을 갖추어야 한다.28 이는 정적 환경을 가정한 C-blox의 한계를 넘어서는 중요한 연구 방향이다.
- **신경망 기반 표현 방식의 등장**: 최근에는 TSDF를 넘어, NeRF나 3DGS와 같이 신경망을 이용한 새로운 암시적 표현 방식이 각광받고 있다. 이들 역시 단일 거대 신경망의 한계를 극복하기 위해 C-blox와 유사하게 여러 개의 작은 신경망을 서브맵처럼 활용하여 확장성 문제를 해결하려는 시도를 하고 있다.3 이는 C-blox가 제시했던 '분할 정복' 기반의 모듈식 아키텍처 패러다임이 표현 방식의 변화에도 불구하고 여전히 강력하고 유효함을 시사한다.

결론적으로 C-blox는 그 자체로 완결된 시스템이라기보다는, 대규모 밀집 매핑이라는 거대한 과제를 해결하기 위한 여정에서 중요한 개념적, 구조적 토대를 마련한 핵심적인 연구로 평가되어야 한다. C-blox가 남긴 유산은 오늘날에도 여전히 새로운 기술과 결합하며 자율 로봇의 공간 인지 능력을 한 단계 더 높은 차원으로 이끌고 있다.

#### 참고 자료

1. A Lightweight, Centralized, Collaborative, Truncated Signed Distance Function-Based Dense Simultaneous Localization and Mapping System for Multiple Mobile Vehicles, 8월 9, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC11598301/
2. Simultaneous Localization and Mapping (SLAM) for Autonomous Driving: Concept and Analysis - MDPI, 8월 9, 2025에 액세스, https://www.mdpi.com/2072-4292/15/4/1156
3. Globally Consistent RGB-D SLAM with 2D Gaussian Splatting - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/html/2506.00970v1
4. [1710.07242] C-blox: A Scalable and Consistent TSDF-based Dense Mapping Approach, 8월 9, 2025에 액세스, https://arxiv.org/abs/1710.07242
5. (PDF) C-blox: A Scalable and Consistent TSDF-based Dense ..., 8월 9, 2025에 액세스, https://www.researchgate.net/publication/320516974_C-blox_A_Scalable_and_Consistent_TSDF-based_Dense_Mapping_Approach
6. Efficient volumetric mapping of multi-scale environments using wavelet-based compression - Robotics, 8월 9, 2025에 액세스, https://www.roboticsproceedings.org/rss19/p065.pdf
7. arXiv:1902.11061v2 [cs.RO] 15 Jul 2019, 8월 9, 2025에 액세스, https://arxiv.org/pdf/1902.11061
8. [1807.01012] Submap-based Pose-graph Visual SLAM: A Robust Visual Exploration and Localization System - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/abs/1807.01012
9. C-blox: A Scalable and Consistent TSDF-based Dense Mapping Approach - ResearchGate, 8월 9, 2025에 액세스, https://www.researchgate.net/publication/330586499_C-blox_A_Scalable_and_Consistent_TSDF-based_Dense_Mapping_Approach
10. ethz-asl/cblox: Voxblox-based submapping - GitHub, 8월 9, 2025에 액세스, https://github.com/ethz-asl/cblox
11. Distributed Gaussian Process Mapping for Robot Teams with Time-Varying Communication - Nikolay A. Atanasov, 8월 9, 2025에 액세스, https://natanaso.github.io/ref/Di_DistributedGP_ACC22.pdf
12. Truncated Signed Distance Fields Applied To Robotics - DiVA portal, 8월 9, 2025에 액세스, https://www.diva-portal.org/smash/get/diva2:1136113/FULLTEXT01.pdf
13. A Scale-Adaptive Time-Efficient Depth Map Fusion Algorithm - preprints from Optica Open, 8월 9, 2025에 액세스, https://preprints.opticaopen.org/articles/preprint/A_Scale-Adaptive_Time-Efficient_Depth_Map_Fusion_Algorithm/22093745/1/files/39261848.pdf
14. ethz-asl/voxblox: A library for flexible voxel-based mapping, mainly focusing on truncated and Euclidean signed distance fields. - GitHub, 8월 9, 2025에 액세스, https://github.com/ethz-asl/voxblox
15. [1611.03631] Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/abs/1611.03631
16. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - Helen Oleynikova, 8월 9, 2025에 액세스, https://helenol.github.io/publications/iros_2017_voxblox.pdf
17. ethz-asl/voxgraph: Voxblox-based Pose graph optimization - GitHub, 8월 9, 2025에 액세스, https://github.com/ethz-asl/voxgraph
18. Voxgraph: Globally Consistent, Volumetric Mapping using Signed Distance Function Submaps - Helen Oleynikova, 8월 9, 2025에 액세스, https://helenol.github.io/publications/ral_2019_voxgraph.pdf
19. Coxgraph: Multi-Robot Collaborative, Globally Consistent, Online ..., 8월 9, 2025에 액세스, http://www.cad.zju.edu.cn/home/gfzhang/papers/Coxgraph/IROS21_Coxgraph.pdf
20. coVoxSLAM: GPU accelerated globally consistent dense SLAM - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/html/2410.21149v1
21. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/html/2311.00626v2
22. Octomap comparison. / Issue #28 / ethz-asl/voxgraph - GitHub, 8월 9, 2025에 액세스, https://github.com/ethz-asl/voxgraph/issues/28
23. Mapping and Planning for Safe Collision Avoidance On-board Micro-Aerial Vehicles - Helen Oleynikova, 8월 9, 2025에 액세스, https://helenol.github.io/publications/thesis_2019_oleynikova.pdf
24. [2405.05702] NGM-SLAM: Gaussian Splatting SLAM with Radiance Field Submap - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/abs/2405.05702
25. Volumetric Instance-Level Semantic Mapping Via Multi-View 2D-to-3D Label Diffusion - Research Collection, 8월 9, 2025에 액세스, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/538641/main_preprint.pdf?sequence=1&isAllowed=y
26. Semantic Mapping for Autonomous Navigation and Exploration - Robotics Institute Carnegie Mellon University, 8월 9, 2025에 액세스, https://www.ri.cmu.edu/publications/semantic-mapping-for-autonomous-navigation-and-exploration/
27. A Dense Panoptic Mapping System with Hierarchical World Representation and Label Optimization Techniques - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/pdf/2403.16880
28. POCD: Probabilistic Object-Level Change Detection and Volumetric Mapping in Semi-Static Scenes - Robotics, 8월 9, 2025에 액세스, https://www.roboticsproceedings.org/rss18/p013.pdf
29. DynaHull: Density-centric Dynamic Point Filtering in Point Clouds - arXiv, 8월 9, 2025에 액세스, https://arxiv.org/html/2401.07541v2


# Voxgraph에 대한 심층 고찰

## 1. 서론

### 0.1  SLAM의 본질과 핵심 도전 과제

자율 이동 로봇 기술의 근간을 이루는 동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM)은 로봇이 사전 정보가 없는 미지의 환경을 탐사하며 자신의 위치를 실시간으로 추정함과 동시에 주변 환경의 지도를 작성하는 복합적인 문제를 지칭한다.1 이 기술은 로봇이 자율적으로 항법하고, 환경과 상호작용하며, 주어진 임무를 수행하기 위한 필수적인 전제 조건이다. 그러나 SLAM 기술은 본질적으로 극복하기 어려운 도전을 내포하고 있는데, 그중 가장 핵심적인 것이 바로 **누적 오차(Drift)** 문제이다.1 로봇은 자신의 움직임을 추정하기 위해 주행 거리 측정(Odometry) 정보를 활s용하는데, 이는 관성 측정 장치(IMU), 시각 센서, 혹은 바퀴 엔코더 등 다양한 센서로부터 얻어진다. 이 측정값들에는 필연적으로 미세한 오차가 포함되어 있으며, 로봇이 이동함에 따라 이 작은 오차들이 계속해서 누적되어 결국 로봇의 실제 위치와 추정 위치 간의 간극을 벌리고, 생성된 지도의 정합성을 심각하게 훼손한다.

이러한 누적 오차 문제를 해결하고 대규모 환경에서 장기적인 자율 항법을 가능하게 하기 위해 **전역적 일관성(Global Consistency)** 확보는 필수적이다.5 전역적 일관성은 로봇이 이전에 방문했던 장소를 재인식하는 루프 클로저(Loop Closure) 과정을 통해 달성된다. 루프가 감지되면, SLAM 시스템은 누적된 오차를 전체 궤적과 지도에 걸쳐 분배하고 보정하는 전역 최적화(Global Optimization)를 수행한다.8 이 과정이 없다면 지도는 뒤틀리거나 중첩되는 등 심각한 왜곡을 겪게 되며, 이는 신뢰할 수 있는 항법을 불가능하게 만든다. 따라서, 실시간으로 정확한 위치를 추정하면서도 전역적으로 일관된 지도를 유지하는 것은 현대 SLAM 시스템이 해결해야 할 가장 중요한 과제라 할 수 있다.

### 0.2  지도 표현의 패러다임: 희소(Sparse) SLAM vs. 밀집(Dense) SLAM

SLAM 시스템이 환경을 표현하는 방식, 즉 지도 표현(Map Representation)은 크게 희소(Sparse) 방식과 밀집(Dense) 방식으로 나뉜다. 이 두 패러다임은 각각 뚜렷한 장단점을 가지며, SLAM 기술의 발전 방향을 이해하는 데 중요한 척도가 된다.

**희소 SLAM (Sparse SLAM)**은 ORB-SLAM과 같은 시스템에서 주로 채택하는 방식으로, 환경에서 시각적으로 구별되는 소수의 특징점(feature points)이나 랜드마크만을 추출하여 지도를 구성한다.1 이 방식의 가장 큰 장점은 계산 효율성이다. 소수의 점들만 관리하면 되므로 적은 계산 자원으로도 실시간 위치 추정이 가능하며, 조명 변화나 빠른 움직임에 상대적으로 강인한 성능을 보인다.9 그러나 희소 SLAM은 치명적인 한계를 가지고 있다. 생성된 희소 포인트 클라우드 맵은 환경의 기하학적 구조에 대한 정보를 거의 담고 있지 않기 때문에, 로봇의 장애물 회피, 경로 계획, 또는 환경과의 물리적 상호작용과 같은 실질적인 항법(Navigation) 작업에 직접적으로 활용하기가 매우 어렵다.1 즉, 위치 추정(Localization) 문제 해결에는 탁월하지만, 그 결과물인 지도의 활용성은 극히 제한적이다.

반면, **밀집 SLAM (Dense SLAM)**은 KinectFusion과 같은 초기 연구에서부터 주목받은 방식으로, 센서가 감지하는 환경의 모든 기하학적 정보를 사용하여 밀집된 3D 모델, 예를 들어 볼류메트릭 맵(volumetric map)이나 표면 메쉬(surface mesh)를 생성한다.1 이 방식의 최대 장점은 생성된 지도가 매우 풍부한 정보를 담고 있어, 로봇의 경로 계획이나 장애물 회피에 즉시 사용될 수 있다는 점이다.5 하지만 이러한 밀집 표현은 막대한 계산 비용과 메모리 사용량을 수반한다. 더 큰 문제는 확장성과 오차 보정의 어려움이다. 누적 오차가 발생했을 때, 거대한 단일 맵(monolithic map) 전체가 훼손되며, 이를 수정하기 위해서는 원본 센서 데이터를 모두 다시 처리해야 하므로 실시간 보정이 거의 불가능하다.1

### 0.3  Voxgraph의 등장 배경 및 핵심 기여

Voxgraph는 바로 이러한 희소 SLAM의 '항법 활용성 부족'과 밀집 SLAM의 '확장성 및 오차 보정의 어려움'이라는 두 패러다임 사이의 근본적인 간극을 메우기 위한 혁신적인 시도로 등장했다. Voxgraph의 개발자들은 계산 자원이 극도로 제한된 플랫폼, 예를 들어 GPU 없이 CPU만 탑재한 소형 무인 항공기(Micro Aerial Vehicle, MAV)에서도 **전역적으로 일관된(globally consistent)** **밀집 볼류메트릭 맵(volumetric maps)**을 실시간으로 생성하는 경량화된 프레임워크를 목표로 했다.1

이 목표를 달성하기 위한 Voxgraph의 핵심 전략은 두 가지로 요약된다. 첫째, 환경을 여러 개의 작고 독립적인 **서브맵(submap)**으로 분할하여 관리하는 것이다. 둘째, 이 서브맵들의 전역적인 배치 관계를 희소 SLAM에서 사용하는 **포즈 그래프 최적화(pose graph optimization)** 기법을 통해 효율적으로 정렬하는 것이다.1 이 접근법을 통해 Voxgraph는 밀집 지도의 풍부한 정보량과 희소 최적화의 전역 일관성 및 효율성을 성공적으로 결합시켰다.

이러한 접근 방식은 단순히 두 패러다임의 기술을 절충한 것을 넘어선다. 이는 SLAM의 결과물이 단순한 위치 추정을 위한 부산물이 아니라, 로봇의 다음 행동, 즉 경로 계획과 환경과의 상호작용으로 직접 이어져야 한다는 실용적인 요구를 만족시키기 위한 근본적인 설계 철학의 변화를 의미한다. 기존 SLAM 연구가 '위치 추정'과 '지도 작성'을 다소 분리된 문제로 다루었다면, Voxgraph는 '정확한 위치 위에서 그려진, 항법 가능한 지도'라는 통합된 목표를 제시했다. 이를 위해 '분할 정복(Divide and Conquer)' 전략을 채택했다. 짧은 시간 동안 생성된 서브맵은 내부적으로 일관성이 높다고(drift가 적다고) 가정하고, 이 서브맵 자체를 하나의 강체(rigid body)처럼 취급한다. 그 후, 전역적인 지도 정합 문제는 이 강체 블록들을 어떻게 올바르게 배치할 것인가 하는, 훨씬 더 단순화된 최적화 문제, 즉 포즈 그래프 최적화로 환원된다. 결국 Voxgraph는 문제의 복잡도를 '서브맵'이라는 단위로 캡슐화하고, 전역 최적화의 대상을 서브맵의 포즈로 한정함으로써 계산 효율성과 전역 일관성이라는 두 마리 토끼를 동시에 잡는 데 성공한 것이다.

## 1.  Voxgraph의 시스템 아키텍처

Voxgraph의 아키텍처는 복잡한 SLAM 문제를 체계적으로 해결하기 위해 명확하게 정의된 모듈과 계층적 구조로 설계되었다. 이는 '모듈화'와 '추상화'라는 현대 소프트웨어 공학의 원칙을 성공적으로 적용한 사례로 볼 수 있다.

### 1.1  프론트엔드와 백엔드 분리 구조

Voxgraph는 전통적인 SLAM 시스템과 마찬가지로 프론트엔드(Frontend)와 백엔드(Backend)로 구성된 이원적 구조를 가진다.3 이러한 분리 구조는 각 모듈이 특정 역할에 집중하게 하여 시스템의 복잡도를 낮추고 개발 및 유지보수를 용이하게 한다.

- **프론트엔드(Frontend):** 프론트엔드는 실시간으로 들어오는 센서 데이터 처리를 담당한다. 주요 역할은 로봇의 주행 거리 측정(Odometry) 정보와 밀집된 3D 센서 데이터(예: 포인트 클라우드)를 입력받아, 이를 기반으로 **볼류메트릭 서브맵을 생성**하는 것이다. 또한, 새로 생성된 서브맵과 기존의 인접한 서브맵이 겹치는 영역을 분석하여 둘 사이의 상대적인 기하학적 관계, 즉 **정합 제약(registration constraint)**을 계산한다. 이렇게 생성된 서브맵과 제약 정보는 백엔드로 전달되어 전역 최적화에 사용된다.3
- **백엔드(Backend):** 백엔드는 전역적인 지도 일관성을 유지하는 역할을 수행한다. 프론트엔드로부터 전달받은 모든 제약들-주행 거리 측정 제약, 서브맵 정합 제약, 그리고 외부에서 주입될 수 있는 루프 클로저 제약-을 하나의 거대한 **포즈 그래프(pose graph)** 형태로 구성하고 유지한다.3 새로운 서브맵이나 제약이 추가될 때마다, 백엔드는 비선형 최적화(non-linear optimization)를 수행하여 그래프의 모든 오차를 최소화하는, 가장 확률이 높은 서브맵들의 전역적 배치(global alignment)를 추정한다. 이 과정을 통해 누적된 오차가 보정되고 지도의 전역 일관성이 확보된다.

### 1.2  핵심 의존성 라이브러리: `voxblox`와 `c-blox`

Voxgraph의 모듈식 설계는 검증된 외부 라이브러리를 효과적으로 활용함으로써 완성된다. 특히 `voxblox`와 `c-blox`는 Voxgraph의 핵심 기능을 구성하는 두 기둥이다.

- `voxblox`: 각 개별 서브맵을 생성하고 관리하는 데 사용되는 핵심 볼류메트릭 매핑 라이브러리다.6

  `voxblox`는 주로 절단 부호 거리 필드(Truncated Signed Distance Field, TSDF)와 유클리드 부호 거리 필드(Euclidean Signed Distance Field, ESDF)라는 두 가지 강력한 지도 표현 방식을 지원한다. 특히 MAV(드론)와 같이 계산 자원이 제한된 플랫폼에서 실시간으로 경로 계획을 수행할 수 있도록 최적화되어 있다.14 Voxgraph는 `voxblox`를 사용하여 각 서브맵의 정밀한 3D 기하학적 구조를 효율적으로 구축한다.

- `c-blox`: `voxblox`로 생성된 여러 서브맵들의 집합, 즉 컬렉션을 관리하는 역할을 담당하는 라이브러리다.6

  `c-blox`는 새로운 서브맵을 언제 생성할지 결정하고, 서브맵들 간의 연결 관계(위상, topology)를 포즈 그래프 형태로 구축하며, 이들을 관리하는 상위 레벨의 프레임워크를 제공한다.17 비유하자면, `voxblox`가 정교한 '벽돌'을 만드는 기술이라면, `c-blox`는 이 벽돌들을 체계적으로 쌓아 '건물'을 짓고 관리하는 설계도와 관리 시스템에 해당한다.

### 1.3  지도 표현 방식: 서브맵(Submap) 기반 접근법

전통적인 밀집 SLAM이 환경 전체를 하나의 거대한 단일 맵(monolithic map)으로 표현하려 했던 것과 달리, Voxgraph는 환경을 여러 개의 작고 겹치는 서브맵들의 집합으로 표현하는 전략을 채택했다.1 이 서브맵 기반 접근법은 Voxgraph가 대규모 환경에서도 확장성과 효율성을 유지할 수 있는 핵심적인 이유다.

이 방식의 장점은 명확하다. 첫째, **확장성(Scalability)**이 뛰어나다. 전체 지도의 크기가 커지더라도, 최적화는 전체 복셀 수가 아닌 서브맵의 수에 비례하여 수행된다. 이는 대규모 환경 탐사 시 계산 복잡도의 폭발적인 증가를 막아준다.7 둘째, 

**오차 보정의 용이성**이다. 루프 클로저가 발생하여 과거의 위치 추정 오차를 수정해야 할 때, 단일 맵 방식은 전체 지도를 처음부터 다시 통합해야 하는 엄청난 비용이 발생한다. 하지만 서브맵 방식에서는 각 서브맵 내부의 기하학적 구조는 강체(rigid)로 유지한 채, 서브맵들의 상대적인 포즈만을 조정하면 되므로 매우 효율적인 오차 보정이 가능하다.1 셋째, 

**메모리 효율성**이다. 로봇이 탐사하는 영역에 따라 동적으로 서브맵을 생성하고 관리하므로, 불필요한 영역에 대한 메모리 낭비를 줄일 수 있다.

### 1.4  볼류메트릭 표현: TSDF와 ESDF

Voxgraph의 서브맵은 내부적으로 볼류메트릭(volumetric) 방식으로 3차원 공간을 표현한다. 이는 공간을 작은 정육면체 단위인 복셀(voxel)로 나누고 각 복셀에 특정 정보를 저장하는 방식이다. Voxgraph는 주로 TSDF와 ESDF라는 두 가지 부호 거리 필드(Signed Distance Field, SDF) 표현을 활용한다.

- **TSDF (Truncated Signed Distance Field):** 각 복셀에 가장 가까운 표면까지의 거리를 저장하되, 그 거리를 센서에서 나온 광선 방향으로 측정한 '투영(projective)' 거리를 사용한다. 또한, 표면에서 일정 거리 이상 떨어진 복셀의 값은 최대/최소값으로 잘라내어(truncate) 저장하지 않음으로써 메모리 효율을 높인다.5 TSDF는 여러 센서 측정값을 융합하여 노이즈를 줄이고 부드러운 표면을 재구성하는 데 매우 효과적이어서, 주로 고품질의 3D 메쉬(mesh)를 생성하는 데 사용된다.16
- **ESDF (Euclidean Signed Distance Field):** 각 복셀에 가장 가까운 장애물(표면)까지의 실제 '유클리드(Euclidean)' 거리를 저장한다. 이 표현의 가장 큰 장점은 로봇의 경로 계획에 직접적으로 사용될 수 있다는 점이다. 특정 지점에서 가장 가까운 장애물까지의 거리를 즉시 알 수 있으므로 충돌 여부를 빠르게 확인할 수 있다. 더 나아가, ESDF는 거리의 방향, 즉 기울기(gradient) 정보까지 제공하여, 최적화 기반의 경로 계획 알고리즘이 장애물로부터 안전하게 멀어지는 부드러운 궤적을 생성하는 데 결정적인 역할을 한다.4

Voxgraph의 프론트엔드(`voxblox` 기반)는 먼저 입력되는 센서 데이터로부터 점진적으로 TSDF를 구축하고, 이 TSDF로부터 ESDF를 효율적으로 파생시킨다.5 그리고 Voxgraph는 이 **ESDF**를 서브맵 간의 상대적 위치를 추정하는 정합 과정에 핵심적으로 활용한다.5

이러한 아키텍처 설계는 SLAM의 두 가지 핵심 과제, 즉 (1) 센서 데이터를 맵으로 '통합(Integration)'하는 것과 (2) 누적된 오차를 '보정(Correction)'하는 것을 명확히 분리하여 해결한다. `voxblox`는 (1)의 과제, 즉 MAV의 실시간 경로 계획에 적합한 로컬 맵(TSDF/ESDF)을 효율적으로 통합하는 문제를 완벽하게 해결했다. Voxgraph의 설계자들은 이 검증된 `voxblox`를 하나의 블랙박스처럼 취급하여, (2)의 과제, 즉 이 로컬 맵들을 어떻게 전역적으로 일관되게 만들 것인가에 집중할 수 있었다. 이를 위해 '서브맵'이라는 추상화된 데이터 단위를 도입하고, 이들 간의 관계를 그래프로 모델링했다. 그 결과, 전역 최적화 문제는 개별 복셀이나 포인트가 아닌, 서브맵의 포즈라는 훨씬 고차원적이고 다루기 쉬운 변수에 대한 문제로 단순화되었다. 이와 같은 계층적이고 모듈적인 설계 덕분에 Voxgraph는 기존의 강력한 기술들을 효과적으로 재사용하면서도, 전역 일관성이라는 새로운 가치를 창출할 수 있었으며, 이는 복잡한 로봇 소프트웨어 엔지니어링의 모범적인 접근 방식을 보여준다.

## 2.  핵심 알고리즘 및 수학적 원리

Voxgraph의 성공은 효율적이면서도 강인한 핵심 알고리즘들에 기반한다. 이 알고리즘들은 원본 센서 데이터의 방대한 정보를 전략적으로 압축하고, 압축된 정보로부터 전역적 일관성이라는 상위 수준의 정보를 효과적으로 복원하도록 설계되었다.

### 2.1  TSDF 서브맵 생성 및 융합

Voxgraph의 프론트엔드는 로봇의 주행 거리나 경과 시간 등 미리 정의된 기준에 따라 새로운 서브맵 생성을 시작한다.3 새로운 3D 포인트 클라우드 데이터가 입력되면, 각 포인트는 광선 투사(ray casting) 기법을 통해 해당 서브맵의 복셀 그리드에 투영된다. 이 과정에서 각 복셀의 중심점과 센서 원점을 잇는 광선 상에 포인트가 위치하게 되며, 복셀에는 표면까지의 부호 거리(signed distance)가 기록된다. 표면 앞(센서와 표면 사이)에 있는 복셀은 양수 값을, 표면 뒤에 있는 복셀은 음수 값을 가진다.

새로운 측정값은 기존에 저장된 값과 즉시 융합되어 맵이 점진적으로 업데이트된다. 이 융합 과정은 센서 노이즈를 줄이고 표면 모델의 정확도를 높이는 데 결정적이다. `voxblox`에서 사용하는 가중 평균(weighted average) 기반의 융합 공식은 다음과 같이 표현될 수 있다 16:
$$
D_k(\mathbf{v}) = \frac{W_{k-1}(\mathbf{v})D_{k-1}(\mathbf{v}) + w_k(\mathbf{p})d_k(\mathbf{p})}{W_{k-1}(\mathbf{v}) + w_k(\mathbf{p})}
\\
W_k(\mathbf{v}) = W_{k-1}(\mathbf{v}) + w_k(\mathbf{p})
$$
여기서 $\mathbf{v}$는 특정 복셀을, $k$는 측정 시점을 나타낸다. $D_k(\mathbf{v})$는 $k$번째 측정 후 복셀 $\mathbf{v}$의 융합된 최종 TSDF 값이며, $W_k(\mathbf{v})$는 해당 복셀에 대한 누적 가중치(신뢰도)이다. $d_k(\mathbf{p})$와 $w_k(\mathbf{p})$는 현재 $k$번째로 측정된 포인트 $\mathbf{p}$에 해당하는 거리 값과 그 가중치를 의미한다. 이 공식을 통해, 여러 번 관측된 표면일수록 더 높은 가중치를 가지게 되어 맵이 안정적으로 수렴한다. 이 과정은 원본 센서 데이터(raw data)를 버리는 일종의 '정보 손실' 과정이지만, 노이즈가 제거되고 정규화된 강인한 표면 정보를 얻는다는 점에서 전략적인 선택이다.

### 2.2  대응점 없는(Correspondence-Free) 서브맵 정합

Voxgraph의 가장 혁신적인 기여 중 하나는 두 서브맵 간의 상대적 변환(relative transform)을 추정하는 방식에 있다. 전통적인 ICP(Iterative Closest Point) 계열의 알고리즘은 두 포인트 클라우드에서 대응하는 점(correspondence)을 명시적으로 찾아야 했으며, 이는 특징점이 부족하거나 기하학적 구조가 반복되는 환경에서 실패하기 쉬웠다. Voxgraph는 **대응점 없는(correspondence-free)** 정합 방식을 통해 이 문제를 우회한다.1

이 방식의 핵심은 ESDF를 활용하는 것이다. 겹치는 두 서브맵, 즉 Source 서브맵과 Target 서브맵이 있다고 가정하자. Voxgraph는 Source 서브맵의 표면(TSDF 값이 0에 가까운 복셀들)에서 포인트를 샘플링한다. 그 후, 이 포인트들을 현재 추정된 상대 변환을 이용해 Target 서브맵의 좌표계로 변환한다. Target 서브맵은 ESDF로 표현되어 있으므로, 변환된 각 포인트의 위치에서 가장 가까운 표면까지의 유클리드 거리와 그 방향(기울기)을 즉시 조회할 수 있다.

정합 과정의 목표는 Source 서브맵의 표면 포인트들이 Target 서브맵의 표면(즉, ESDF 값이 0이 되는 지점)에 최대한 가깝게 위치하도록 하는 최적의 상대 변환을 찾는 것이다. 이는 샘플링된 포인트들의 ESDF 값의 합을 최소화하는 비선형 최적화 문제로 공식화된다. ESDF가 제공하는 연속적인 거리 필드와 기울기 정보를 직접 활용하므로, 대응점을 일일이 탐색할 필요가 없어 계산적으로 매우 효율적이며, 경사 하강법(gradient descent)과 같은 최적화 기법을 통해 빠르게 해를 찾을 수 있다. 이 방식은 정보가 부족한 환경에서도 두 서브맵 간의 기하학적 관계를 강인하게 추정할 수 있게 해주는 Voxgraph의 핵심 기술이다.

### 2.3  포즈 그래프 최적화 (Pose Graph Optimization)

프론트엔드에서 생성된 서브맵들과 그들 간의 제약 정보는 백엔드로 전달되어 하나의 거대한 포즈 그래프를 형성한다. 이 그래프 기반 최적화는 Voxgraph가 전역적 일관성을 달성하는 최종 단계이다.

- **그래프 구성:** 포즈 그래프에서 각 **노드(Node)**는 개별 서브맵의 월드 좌표계에 대한 포즈(위치와 방향, $\mathbf{T}_i \in SE(3)$)를 나타내는 최적화 변수이다. **엣지(Edge)**는 두 노드(서브맵) 간의 상대적인 관계를 나타내는 **제약(Constraint)**을 의미하며, 다음과 같은 세 가지 주요 유형이 있다 3:
  1. **주행 거리 측정 제약 (Odometry Constraint):** 로봇의 odometry 소스로부터 얻어지는 연속된 포즈 간의 상대 변환. 이는 서브맵들이 시간 순서에 따라 대략적으로 올바른 위치에 있도록 하는 역할을 한다.
  2. **서브맵 정합 제약 (Registration Constraint):** 3.2절에서 설명한 대응점 없는 정합 방식을 통해 계산된, 공간적으로 겹치는 두 서브맵 간의 정밀한 상대 변환.
  3. **루프 클로저 제약 (Loop Closure Constraint):** 로봇이 이전에 방문했던 장소로 돌아왔을 때, 시간적으로는 멀리 떨어져 있지만 공간적으로는 동일한 위치를 나타내는 두 서브맵 간에 추가되는 강력한 제약. Voxgraph 자체는 루프 클로저를 능동적으로 탐지하지 않지만, 외부 모듈(예: 특징점 기반 장소 인식 시스템)로부터 이 정보를 받아 그래프에 추가할 수 있는 유연한 인터페이스를 제공한다.6
- **오차 함수 (Error Function):** 백엔드의 목표는 이 모든 제약들을 가장 잘 만족시키는 최적의 서브맵 포즈 집합 $\mathbf{T}^* = \{\mathbf{T}_1,..., \mathbf{T}_N\}$을 찾는 것이다. 이는 모든 엣지에서의 오차(잔차, residual)의 제곱 합을 최소화하는 비선형 최소 제곱법(non-linear least squares) 문제로 공식화된다. 일반적인 포즈 그래프의 전체 오차 함수는 다음과 같다 21:

$$
\mathbf{T}^* = \underset{\{\mathbf{T}\}}{\arg\min} \sum_{\langle i,j \rangle \in \mathcal{C}} e(\mathbf{T}_i, \mathbf{T}_j, \hat{\mathbf{T}}_{ij})^{\top} \mathbf{\Omega}_{ij} e(\mathbf{T}_i, \mathbf{T}_j, \hat{\mathbf{T}}_{ij})
$$

여기서 $\mathcal{C}$는 모든 제약의 집합을 의미한다. 오차항 $e(\mathbf{T}_i, \mathbf{T}_j, \hat{\mathbf{T}}_{ij})$는 측정된 상대 변환 $\hat{\mathbf{T}}_{ij}$과 현재 추정된 포즈 $\mathbf{T}_i, \mathbf{T}_j$로부터 계산된 상대 변환 $(\mathbf{T}_i^{-1} \mathbf{T}_j)$ 간의 차이를 나타낸다. 이 오차는 보통 리 대수(Lie algebra) 공간에서 표현되며, $\log((\hat{\mathbf{T}}_{ij})^{-1} (\mathbf{T}_i^{-1} \mathbf{T}_j))$와 같이 계산된다. $\mathbf{\Omega}_{ij}$는 해당 측정의 신뢰도를 나타내는 정보 행렬(information matrix)로, 측정 오차의 공분산 행렬의 역행렬이다. 신뢰도가 높은 측정(예: 루프 클로저)일수록 더 큰 가중치를 부여받는다. 이 복잡한 최적화 문제는 Ceres Solver나 g2o와 같은 전문 라이브러리를 통해 효율적으로 해결된다.

이 전체 과정은 원본 데이터의 완전성(completeness)을 전략적으로 포기하는 대신, 각 단계에 필요한 핵심 정보(강인한 표면, 상대 포즈)만을 효율적으로 추출하고 전달하여 실시간 성능과 전역적 일관성이라는 두 가지 상충될 수 있는 목표를 모두 달성하는, 매우 영리하게 설계된 정보 처리 파이프라인이라 할 수 있다.

## 3.  구현 및 실제 적용

Voxgraph는 이론적 우수성을 넘어, 실제 로봇 시스템에 적용 가능한 구체적인 구현체와 성공적인 응용 사례를 통해 그 가치를 입증했다. 특히 계산 자원이 제한적인 소형 드론(MAV)에서의 활용은 Voxgraph의 설계 철학이 현실 세계의 문제를 효과적으로 해결했음을 보여준다.

### 3.1  시스템 요구사항 및 의존성

Voxgraph는 로보틱스 분야의 표준 개발 플랫폼인 ROS(Robot Operating System)를 기반으로 구축되었다. 따라서 이를 사용하기 위해서는 ROS 환경에 대한 이해가 필수적이다.

- **운영체제 및 프레임워크:** 주로 Ubuntu 리눅스 환경에서 ROS Kinetic 또는 Melodic 배포판과 함께 사용되도록 개발되었다. 전체 시스템은 `catkin` 빌드 시스템을 통해 관리 및 컴파일된다.6
- **핵심 의존성:** Voxgraph를 소스 코드로부터 빌드하기 위해서는 여러 핵심 라이브러리가 필요하다.
  - `voxblox` 6: 개별 서브맵의 TSDF/ESDF 생성을 담당하는 필수 라이브러리.
  - `c-blox` 6: 서브맵 컬렉션과 포즈 그래프 관리를 위한 필수 라이브러리.
  - `ceres-solver` 17: 백엔드에서 비선형 최소 제곱법 기반의 포즈 그래프 최적화를 수행하는 핵심 솔버.
  - 기타 라이브러리: 구글의 로깅 라이브러리인 `glog`, 커맨드라인 플래그 처리 라이브러리인 `gflags`, 그리고 희소 행렬 연산을 위한 `suitesparse` 등이 필요하다.17
- **설치 과정:** 일반적인 ROS 패키지와 마찬가지로, 사용자는 `catkin` 워크스페이스를 생성한 후, `wstool`과 같은 도구를 사용하여 `voxgraph`의 메인 저장소와 `.rosinstall` 파일에 명시된 모든 의존성 패키지를 소스 코드 형태로 다운로드해야 한다. 그 후 `catkin build` 명령어를 통해 전체 패키지를 컴파일하는 과정을 거친다.6 이는 사용자가 시스템의 모든 구성 요소를 직접 제어하고 필요에 따라 수정할 수 있는 유연성을 제공한다.

### 3.2  ROS 인터페이스 및 설정

Voxgraph는 ROS의 토픽(Topic)과 변환(Transform, TF) 시스템을 통해 다른 로봇 모듈과 데이터를 주고받는다. 이러한 표준화된 인터페이스는 Voxgraph가 특정 센서나 주행 거리 측정 알고리즘에 종속되지 않고 다양한 로봇 시스템에 쉽게 통합될 수 있게 한다.

- **입력 토픽 (Inputs):**
  - **주행 거리 측정 소스 (Odometry):** Voxgraph는 지속적으로 로봇의 자세 변화를 추정하는 odometry 소스를 필요로 한다. 이 정보는 `tf` 토픽을 통해 전달받으며, 사용자는 파라미터 파일에서 `input_odom_frame`(예: "odom")과 `input_base_link_frame`(예: "base_link")을 지정해야 한다. 시각-관성 주행 거리 측정(VIO)이나 LiDAR 주행 거리 측정(LIO) 등 어떠한 odometry 소스와도 연동이 가능하다.6
  - **밀집 데이터 소스 (Dense Data):** 환경의 3D 기하학 정보를 담은 데이터는 `sensor_msgs/PointCloud2` 타입의 ROS 토픽으로 입력받는다. 이 토픽 이름은 파라미터를 통해 설정할 수 있다. RGB-D 카메라의 깊이 이미지나 3D LiDAR의 스캔 데이터로부터 생성된 포인트 클라우드가 주로 사용된다.1
- **출력 토픽 (Outputs):**
  - **최적화된 지도와 포즈:** Voxgraph는 백엔드 최적화를 통해 얻어진 최종 결과를 여러 토픽으로 발행한다. 가장 중요한 출력 중 하나는 최적화된 서브맵들의 3D 메쉬(mesh)로, 이는 Rviz와 같은 시각화 도구에서 확인하기 위해 `visualization_msgs/MarkerArray` 메시지 형태로 `/voxgraph_mapper/mesh`와 같은 토픽에 발행된다.23
  - **TF 트리:** 백엔드 최적화는 서브맵의 포즈뿐만 아니라 로봇의 전체 궤적 또한 보정한다. Voxgraph는 이 드리프트가 보정된 로봇의 궤적을 `tf` 토픽으로 발행할 수 있다. 사용자는 `output_odom_frame`과 `output_base_link_frame`을 입력 프레임과 다르게 설정함으로써, 기존의 드리프트가 포함된 odometry TF 트리와는 별개로, 전역적으로 일관된 새로운 TF 트리를 생성하여 다른 항법 모듈에서 활용할 수 있다.6

### 3.3  주요 응용 분야: MAV(드론) 자율 항법 및 탐사

Voxgraph의 경량성과 CPU만으로 실시간 처리가 가능하다는 특징은 페이로드 용량과 계산 능력, 전력 소모에 큰 제약을 받는 소형 드론(MAV) 플랫폼에 매우 이상적인 솔루션이다.4

실제로 Voxgraph의 개발과 검증은 스위스 취리히 연방 공과대학교(ETH Zurich)의 자율 시스템 연구소(Autonomous Systems Lab, ASL)에서 MAV를 이용한 자율 항법 및 탐사 연구와 긴밀하게 연계되어 진행되었다. 연구팀은 Voxgraph를 MAV의 온보드 컴퓨터에 탑재하여 대규모 실외 비정형 환경(unstructured environments)이나 복잡한 산업 시설 내부에서의 자율 탐사, 고품질 3D 재구성, 그리고 생성된 지도를 이용한 실시간 경로 계획을 성공적으로 시연했다.1

이러한 실험을 통해 Voxgraph는 인상적인 성능을 입증했다. 예를 들어, 약 400미터 길이의 궤적을 비행했을 때 최종 위치 오차의 RMSE(Root Mean Square Error)가 1미터 미만으로 유지되었으며, 120x80미터 크기의 넓은 영역에 대한 지도를 단 4초 이내에 전역적으로 최적화하는 성능을 보였다.1 이는 시각-관성 SLAM(VI-SLAM)과 결합하여 루프 클로저 보정을 포함한 완전한 자율 항법 시스템을 외부의 도움 없이 오직 MAV의 온보드 계산만으로 구현한 최초의 사례 중 하나로 평가받으며, Voxgraph가 이론을 넘어 실제 로봇 응용 분야에 미친 지대한 영향을 명확히 보여준다.4

## 4.  비교 분석 및 생태계

Voxgraph를 더 깊이 이해하기 위해서는 동시대의 다른 기술들과 비교하고, Voxgraph의 아이디어가 어떻게 후속 연구로 이어져 하나의 기술 생태계를 형성했는지 살펴보는 것이 중요하다. 이러한 비교 분석은 Voxgraph의 설계 철학과 그 기술적 위치를 명확히 해준다.

### 4.1  지도 표현 방식 비교

볼류메트릭 SLAM에서 지도 표현 방식의 선택은 시스템의 성능과 활용도를 결정하는 중요한 요소다. Voxgraph는 TSDF/ESDF를 채택했는데, 이는 당시 널리 사용되던 OctoMap과 비교할 때 뚜렷한 장단점을 가진다.

**Table 1: 맵 표현 방식 비교 (TSDF/ESDF vs. OctoMap)**

| **항목**             | **TSDF (및 ESDF)**                                           | **OctoMap (점유 격자)**                                      |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **메모리 사용량**    | 표면 근처의 좁은 밴드만 저장하여 효율적. 단, ESDF를 전체 공간에 대해 계산할 경우 메모리 요구량이 증가할 수 있음. 19 | 옥트리(Octree) 구조를 사용하여 비어있거나 미탐사된 넓은 공간을 효율적으로 압축. 대규모 공간 표현에 유리. 19 |
| **표면 표현 정확성** | 연속적인 거리 필드를 통해 서브 복셀(sub-voxel) 수준의 정밀도로 표면 위치를 표현 가능. 센서 노이즈 모델링에 유리. 19 | 공간을 이산적인 점유/비점유 확률로 표현. 이로 인해 객체 경계가 실제보다 부풀려지거나 왜곡될 수 있음. 20 |
| **경로 계획 활용도** | ESDF는 장애물까지의 정확한 유클리드 거리와 그 기울기(gradient)를 직접 제공. CHOMP와 같은 최적화 기반 플래너에 매우 적합. 16 | 점유/비점유의 이진 정보만 제공. A*, RRT와 같은 탐색 기반 플래너에 적합하나, 기울기 정보가 없어 궤적 최적화에 한계. 19 |

이 비교는 Voxgraph가 왜 OctoMap 대신 TSDF/ESDF를 핵심 표현 방식으로 선택했는지를 명확하게 보여준다. Voxgraph의 궁극적인 목표는 단순히 지도를 그리는 것을 넘어, '항법과 계획을 위한 지도'를 만드는 것이었다. ESDF가 제공하는 연속적인 거리와 기울기 정보는 현대적인 궤적 최적화 기반의 경로 계획기(planner)와 완벽하게 호환된다. 이는 로봇의 인지(Perception)가 다음 행동(Action)을 위해 존재해야 한다는 'Perception-for-Action'이라는 로보틱스의 핵심 철학과 정확히 일치한다. 즉, Voxgraph의 맵 표현 방식 선택은 우연한 기술적 결정이 아니라, 시스템의 최종 목표를 달성하기 위한 필연적이고 철학적인 설계 결정이었음을 알 수 있다.

### 4.2  SLAM 패러다임 비교

Voxgraph는 기존 SLAM 패러다임들의 장점을 취하고 단점을 보완하는 방향으로 설계되었다.

- **vs. 희소 특징점 기반 SLAM (e.g., ORB-SLAM):** 가장 큰 차이점은 지도의 활용도에 있다. ORB-SLAM은 위치 추정의 정확성과 강인성에 집중하는 반면, 그 결과물인 희소 특징점 맵은 환경의 기하학적 구조를 거의 담고 있지 않아 장애물 회피나 경로 계획에 직접 사용될 수 없다.1 반면, Voxgraph는 항법에 즉시 사용 가능한 밀집 볼류메트릭 지도를 생성함으로써 이 한계를 극복했다.
- **vs. 단일 맵(Monolithic) 기반 밀집 SLAM (e.g., KinectFusion):** KinectFusion과 같은 초기 밀집 SLAM은 환경 전체를 하나의 거대한 맵으로 관리했다. 이 방식은 누적 오차가 발생했을 때 맵 전체가 영구적으로 훼손되며, 이를 수정하기가 매우 어렵다는 치명적인 단점이 있었다.1 Voxgraph는 서브맵 기반 아키텍처를 도입하여 이 문제를 해결했다. 루프 클로저가 발생하면 서브맵들의 상대 포즈만 조정하면 되므로, 대규모 환경에서도 확장성을 유지하며 전역적 일관성을 확보할 수 있다.
- **vs. 최신 최적화 기반 SLAM (e.g., Occupancy-SLAM):** Occupancy-SLAM과 같은 일부 최신 연구는 로봇의 포즈와 맵 자체(예: 점유 격자의 확률 값)를 하나의 거대한 최적화 문제 안에서 '동시에' 최적화하는 접근법을 제안한다.2 이는 이론적으로 가장 정확한 해를 찾을 수 있지만 계산 비용이 매우 높다. 반면, Voxgraph는 맵의 내부(복셀 값)는 각 서브맵 내에서 고정된 것으로 보고, 서브맵의 '포즈'만을 전역 최적화의 변수로 삼는다.2 이는 문제의 복잡도를 크게 낮추어 실시간 성능을 확보하기 위한 실용적인 타협이며, 최적화 문제를 정의하는 근본적인 방식에서 차이를 보인다.

### 4.3  Voxgraph의 진화와 파생 시스템

Voxgraph가 제시한 강력하고 실용적인 아이디어는 그 자체로 완결된 것이 아니라, 후속 연구자들에게 영감을 주어 하나의 기술 생태계를 형성하는 기반이 되었다. 이 파생 시스템들은 Voxgraph의 기본 철학을 계승하면서 그 한계를 극복하는 방향으로 발전했다.

- `coVoxSLAM`: Voxgraph가 CPU 기반의 경량성에 초점을 맞췄다면, `coVoxSLAM`은 현대 로봇 플랫폼에 널리 탑재되는 GPU의 병렬 처리 능력을 적극적으로 활용하여 성능을 극대화한 시스템이다.5 TSDF 및 ESDF 통합과 같은 계산 집약적인 작업을 GPU로 가속함으로써, Voxgraph 대비 수십 배에 달하는 압도적인 속도 향상을 달성했다.5 이는 실시간 밀집 볼류메트릭 SLAM의 적용 범위를 더욱 까다로운 시나리오로 확장시켰다.

- `Coxgraph`: Voxgraph가 단일 로봇 시나리오를 다루는 반면, `Coxgraph`는 이를 **다중 로봇 협업(multi-robot collaborative)** SLAM으로 확장한 시스템이다.22 각 로봇(클라이언트)은 독립적으로 

  `voxgraph`와 유사한 방식으로 서브맵을 생성하고, 이를 중앙 서버로 전송한다. 서버는 여러 로봇으로부터 수집된 서브맵들을 통합하여 하나의 전역적으로 일관된 지도를 구축한다. 이 과정에서 서브맵 전송에 따른 통신 대역폭 문제를 해결하기 위해, TSDF 서브맵을 가벼운 3D 메쉬 형태로 압축하여 전송하고 서버에서 다시 복원하는 독창적인 기법을 사용한다.29

**Table 2: Voxgraph 및 파생 시스템 비교**

| **시스템**    | **핵심 기여**                                | **주요 기술**                                    | **대상 플랫폼/시나리오**                |
| ------------- | -------------------------------------------- | ------------------------------------------------ | --------------------------------------- |
| **Voxgraph**  | CPU 기반 경량, 전역 일관성 볼류메트릭 SLAM 1 | SDF 서브맵, 대응점 없는 정합, 포즈 그래프 최적화 | 단일 로봇, CPU 제약적 플랫폼 (MAV)      |
| **coVoxSLAM** | GPU 가속을 통한 초고속 볼류메트릭 SLAM 5     | GPU 기반 TSDF/ESDF 통합, 병렬 처리 최적화 (SoA)  | 단일 로봇, 온보드 GPU 탑재 플랫폼       |
| **Coxgraph**  | 다중 로봇 협업, 전역 일관성 밀집 재구성 22   | 클라이언트-서버 구조, 메시 기반 서브맵 압축/전송 | 다중 로봇 시스템, 중앙 집중식 협업 매핑 |

이러한 진화 과정은 Voxgraph가 단일 연구로 끝나지 않고, 하나의 강력한 '기술적 패러다임'을 형성했음을 명확히 보여준다. 후속 연구들은 Voxgraph의 명확한 약점(CPU 성능의 한계, 단일 로봇으로의 제한)을 직접적으로 겨냥하여 개선점을 찾아냈다. 이는 역설적으로 Voxgraph의 기본 설계(서브맵 분할, SDF 기반 정합, 그래프 최적화)가 매우 견고하고 확장 가능성이 높은 구조였음을 방증한다. 이 기술적 계보를 통해 우리는 Voxgraph의 아이디어가 어떻게 진화하고 다양한 문제 영역으로 파생되었는지 그 흐름을 한눈에 파악할 수 있다.

## 5.  한계 및 미래 전망

Voxgraph는 볼류메트릭 SLAM 분야에서 중요한 이정표를 세웠지만, 모든 기술과 마찬가지로 내재적인 한계를 가지고 있다. 이러한 한계를 이해하는 것은 현재 기술의 현주소를 파악하고 미래 연구 방향을 예측하는 데 필수적이다.

### 5.1  Voxgraph 및 볼류메트릭 SLAM의 내재적 한계

- **계산 비용 및 메모리:** Voxgraph는 CPU 기반으로 경량화를 추구했지만, 본질적으로 볼류메트릭 표현은 희소 표현에 비해 훨씬 높은 계산 비용과 메모리 사용량을 요구한다.1 지도의 해상도를 높이거나 탐사 영역이 넓어질수록 복셀의 수는 기하급수적으로 증가하며, 이는 실시간 성능 유지에 큰 부담으로 작용한다. 특히, 포즈 그래프에 서브맵이 수백 개 이상 추가되면 최적화에 소요되는 시간도 무시할 수 없게 된다.30
- **동적 환경 대응의 어려움:** Voxgraph의 기본 프레임워크는 환경이 정적(static)이라는 강한 가정을 기반으로 한다.21 만약 환경에 사람, 차량, 또는 다른 움직이는 물체가 존재할 경우, 이러한 동적 요소들이 정적인 배경의 일부로 맵에 잘못 통합될 수 있다. 이는 맵을 오염시키고, 특히 서브맵 정합 과정에서 잘못된 제약을 생성하여 위치 추정의 정확도를 심각하게 저하시키는 원인이 된다.
- **얇은 구조물 표현의 한계:** TSDF와 같은 볼류메트릭 표현은 특정 두께를 가진 표면을 재구성하는 데는 효과적이지만, 나뭇가지, 케이블, 철조망과 같이 매우 얇은 구조물을 표현하는 데는 어려움을 겪는다.31 센서의 광선이 이러한 얇은 물체를 그대로 통과해 버리거나, 복셀화 과정에서 그 정보가 소실되기 쉽기 때문이다. 이는 특히 실외 비정형 환경에서의 항법에 중요한 제약이 될 수 있다.
- **Odometry 품질 의존성:** Voxgraph는 프론트엔드에서 양질의 주행 거리 측정(odometry) 소스를 지속적으로 제공받는 것을 전제로 동작한다.6 Odometry가 조명 급변이나 빠른 움직임 등으로 인해 일시적으로 큰 오차를 보이거나 실패할 경우, 생성되는 서브맵의 내부 일관성이 깨지게 된다. 이렇게 잘못 생성된 서브맵은 이후의 정합 및 최적화 과정에 나쁜 영향을 미치며, 시스템 전체가 복구 불가능한 상태에 빠질 수 있다.

### 5.2  최신 연구 동향 및 미래 전망

Voxgraph가 제시한 과제들을 해결하고 그 비전을 확장하기 위한 연구는 여러 방향으로 활발하게 진행되고 있다.

- **하드웨어 가속:** `coVoxSLAM`의 사례에서 명확히 드러나듯이, GPU를 활용하여 계산 병목 현상을 해결하려는 연구가 주류를 이루고 있다.5 미래에는 범용 GPU를 넘어, 전력 효율이 더 높은 FPGA(Field-Programmable Gate Array)나 SLAM 연산을 위한 전용 AI 가속기를 활용하여, 더욱 소형화되고 제한된 플랫폼에서도 고성능 볼류메트릭 SLAM을 구현하는 방향으로 발전할 것이다.32
- **딥러닝과의 융합:** 딥러닝 기술은 볼류메트릭 SLAM의 한계를 극복할 새로운 가능성을 열어주고 있다. 예를 들어, 단안 카메라 이미지 한 장으로부터 딥러닝 기반의 깊이 추정 네트워크를 사용하여 고밀도의 3D 정보를 생성하거나 4, 재구성된 3D 지도에 '이것은 문이다', '저곳은 복도다'와 같은 의미론적 정보(semantic information)를 부여하는 연구가 활발하다. 이러한 의미론적 정보는 로봇이 단순히 기하학적 공간을 인지하는 것을 넘어, 환경을 이해하고 더 지능적인 상호작용을 하는 데 결정적인 역할을 할 것이다.
- **다중 센서/로봇 융합:** `Coxgraph`가 다중 로봇 협업의 가능성을 보였듯이, 여러 로봇의 정보를 효율적으로 융합하여 더 빠르고 강인하게 지도를 구축하는 연구는 계속될 것이다.22 또한, 단일 로봇 내에서도 시각 센서, LiDAR, IMU, 레이더 등 서로 다른 특성을 가진 센서들의 정보를 더욱 긴밀하게 결합(tightly-coupled)하여, 특정 센서가 취약한 환경(예: 시각 센서가 약한 어두운 곳, LiDAR가 약한 투명한 벽)에서도 강인하게 동작하는 시스템을 구축하려는 노력이 지속될 것이다.25
- **장기 자율성(Long-term Autonomy):** 실제 환경은 정적이지 않고 시간이 지남에 따라 변한다. 조명, 날씨, 계절이 바뀌고, 가구의 배치가 달라지거나 새로운 구조물이 생길 수 있다. 미래의 SLAM 시스템은 이러한 환경 변화를 능동적으로 감지하고, 기존 지도를 지속적으로 업데이트하며 유지보수하는 능력을 갖추어야 한다. 이는 일회성 지도 작성을 넘어, 로봇이 평생에 걸쳐 자신의 환경에 대한 지식을 관리하고 학습하는 문제로, SLAM 연구의 궁극적인 목표 중 하나이다.

이러한 연구 방향들은 SLAM이 '기하학적 재구성'이라는 초기 단계를 넘어, '환경과의 상호작용을 위한 지속적인 이해'라는 더 높은 차원의 목표로 나아가고 있음을 시사한다. 초기 SLAM이 '어디에 있는가?(Localization)'와 '무엇이 있는가?(Mapping)'라는 기하학적 질문에 답하는 데 집중했다면, Voxgraph는 여기서 한 걸음 더 나아가 '어떻게 지나갈 수 있는가?(Navigation)'에 대한 실용적인 해답을 주는 지도를 만들었다. 미래의 SLAM은 여기에 '저것은 무엇인가?(Semantics)', '어떻게 변하고 있는가?(Dynamics & Long-term)', 그리고 '함께하려면 어떻게 해야 하는가?(Collaboration)'라는 더 복잡하고 고차원적인 질문에 답해야 할 것이다. 이러한 관점에서 Voxgraph의 한계는 기술적 실패가 아니라, 다음 세대의 SLAM 연구가 해결해야 할 명확한 과제들을 제시했다는 점에서 그 중요한 의의를 찾을 수 있다. Voxgraph는 SLAM 연구의 지평을 한 단계 넓힌 중요한 이정표인 것이다.

## 6.  결론

### 7.1. Voxgraph의 핵심 기여와 의의 요약

Voxgraph는 로보틱스 SLAM 분야, 특히 볼류메트릭 매핑의 역사에서 중요한 전환점을 제시한 시스템으로 평가받는다. 그 핵심 기여는 계산 자원이 제한된 플랫폼에서도 실시간으로 **전역적으로 일관된 밀집 볼류메트릭 맵**을 생성하는 실용적인 방법을 제시함으로써, 기존의 희소 SLAM과 밀집 SLAM 패러다임이 가졌던 근본적인 간극을 성공적으로 메웠다는 점에 있다.

이를 가능하게 한 Voxgraph의 핵심 전략은 세 가지 기술의 독창적인 조합에 있다. 첫째, 환경을 독립적인 **서브맵**으로 분할하여 문제의 복잡도를 관리하는 아키텍처. 둘째, 겹치는 서브맵 간의 기하학적 관계를 ESDF를 활용하여 효율적이고 강인하게 추정하는 **대응점 없는 정합** 방식. 셋째, 이 관계들을 **포즈 그래프**로 모델링하고 전역적으로 최적화하여 누적 오차를 보정하는 백엔드. 이 세 가지 요소의 시너지를 통해 Voxgraph는 대규모 환경에서의 확장성, 실시간 처리를 위한 효율성, 그리고 장기 항법을 위한 정확성을 동시에 달성하는 강력한 패러다임을 구축했다.

### 6.1  로보틱스 분야에 미친 영향 및 최종 평가

Voxgraph가 로보틱스 분야에 미친 영향은 지대하며, 특히 소형 드론(MAV)의 자율 항법 및 탐사 기술 발전에 결정적인 기여를 했다.4 이전까지는 고성능의 오프보드 컴퓨터나 GPU의 도움이 필수적이었던 밀집 3D 재구성과 경로 계획을, MAV의 온보드 CPU만으로 가능하게 함으로써 진정한 의미의 자율 비행 시스템 연구를 한 단계 끌어올렸다. 이는 로봇의 인지(Perception)가 단순히 환경을 재구성하는 데 그치지 않고, 항법과 계획이라는 다음 행동(Action)에 직접적으로 활용될 수 있는 형태로 이루어져야 한다는 'Perception-for-Action' 철학의 중요한 실증 사례가 되었다.

더 나아가, Voxgraph의 견고한 설계와 명확한 한계점은 후속 연구에 풍부한 영감을 제공했다. GPU 가속을 통해 성능을 극대화한 `coVoxSLAM`, 다중 로봇 협업으로 개념을 확장한 `Coxgraph`와 같은 시스템들은 모두 Voxgraph라는 거인의 어깨 위에서 탄생했다. 이는 Voxgraph가 단일 연구로 소멸하지 않고, 볼류메트릭 SLAM 연구의 새로운 방향성을 제시한 선구적인 시스템으로서 하나의 기술 생태계를 형성했음을 의미한다.

결론적으로, Voxgraph는 비록 동적 환경 대응이나 얇은 구조물 표현 등에서 명확한 한계를 가지고 있지만, 그 한계점 자체가 미래 연구가 나아갈 방향을 제시하는 이정표가 되었다는 점에서 더욱 큰 의의를 가진다. Voxgraph는 로보틱스 SLAM 기술이 실험실 수준의 기하학적 재구성을 넘어, 실제 세계에서 로봇이 자율적으로 임무를 수행하기 위해 무엇이 필요한지에 대한 깊은 통찰을 제공한, 역사적으로 중요한 시스템으로 기록될 것이다.

#### 참고자료

1. Voxgraph: Globally Consistent, Volumetric Mapping using Signed Distance Function Submaps - Helen Oleynikova, accessed August 6, 2025, https://helenol.github.io/publications/ral_2019_voxgraph.pdf
2. Occupancy-SLAM: Simultaneously Optimizing Robot ... - Robotics, accessed August 6, 2025, https://www.roboticsproceedings.org/rss18/p003.pdf
3. (PDF) Voxgraph: Globally Consistent, Volumetric Mapping Using ..., accessed August 6, 2025, https://www.researchgate.net/publication/337510307_Voxgraph_Globally_Consistent_Volumetric_Mapping_Using_Signed_Distance_Function_Submaps
4. Scalable Outdoors Autonomous Drone Flight with Visual-Inertial SLAM and Dense Submaps Built without LiDAR - arXiv, accessed August 6, 2025, https://arxiv.org/html/2403.09596v2
5. coVoxSLAM: GPU accelerated globally consistent dense SLAM - arXiv, accessed August 6, 2025, https://arxiv.org/html/2410.21149v1
6. ethz-asl/voxgraph: Voxblox-based Pose graph optimization - GitHub, accessed August 6, 2025, https://github.com/ethz-asl/voxgraph
7. Voxgraph: Globally Consistent, Volumetric Mapping Using Signed Distance Function Submaps - Research Collection - ETH Zürich, accessed August 6, 2025, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/385682/voxgraph-ethpreprintversion.pdf?sequence=1
8. What Is SLAM (Simultaneous Localization and Mapping)? - MATLAB & Simulink, accessed August 6, 2025, https://www.mathworks.com/discovery/slam.html
9. Sparse-Dense Motion Modelling and Tracking for Manipulation without Prior Object Models - Edinburgh Research Explorer, accessed August 6, 2025, https://www.research.ed.ac.uk/files/263402272/Sparse_Dense_RAUCH_DOA11042022_AFV.pdf
10. A Review on Visual-SLAM: Advancements from Geometric Modelling to Learning-Based Semantic Scene Understanding Using Multi-Modal Sensor Fusion - PMC, accessed August 6, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9571301/
11. Voxgraph: Globally Consistent, Volumetric Mapping Using Signed Distance Function Submaps - Research Collection, accessed August 6, 2025, https://www.research-collection.ethz.ch/handle/20.500.11850/385682?show=full
12. Voxgraph: Globally Consistent, Volumetric Mapping Using Signed Distance Function Submaps - YouTube, accessed August 6, 2025, https://www.youtube.com/watch?v=N9p1_Fkxxro
13. Voxgraph: Globally Consistent, Volumetric Mapping Using Signed Distance Function Submaps (Extended) - YouTube, accessed August 6, 2025, https://www.youtube.com/watch?v=bJ-DU0tXJYk
14. ethz-asl/voxblox: A library for flexible voxel-based mapping, mainly focusing on truncated and Euclidean signed distance fields. - GitHub, accessed August 6, 2025, https://github.com/ethz-asl/voxblox
15. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning, accessed August 6, 2025, https://huggingface.co/papers/1611.03631
16. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - Helen Oleynikova, accessed August 6, 2025, https://helenol.github.io/publications/iros_2017_voxblox.pdf
17. voxgraph/voxgraph_https.rosinstall at master - GitHub, accessed August 6, 2025, https://github.com/ethz-asl/voxgraph/blob/master/voxgraph_https.rosinstall
18. Efficient Submap-based Autonomous MAV Exploration using Visual-Inertial SLAM Configurable for LiDARs or Depth Cameras - arXiv, accessed August 6, 2025, https://arxiv.org/pdf/2409.16972
19. Signed Distance Fields: A Natural Representation for Both Mapping and Planning - Research Collection, accessed August 6, 2025, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/128029/eth-50477-01.pdf
20. Signed Distance Fields: A Natural ... - Helen Oleynikova, accessed August 6, 2025, https://helenol.github.io/publications/rss_2016_workshop.pdf
21. Solution to the SLAM Problem in Low Dynamic Environments Using a Pose Graph and an RGB-D Sensor - MDPI, accessed August 6, 2025, https://www.mdpi.com/1424-8220/14/7/12467
22. Code for "Coxgraph: Multi-Robot Collaborative, Globally Consistent, Online Dense Reconstruction System", IROS 2021 Best Paper Award Finalist on Safety, Security, and Rescue Robotics in memory of Motohiro Kisoi - GitHub, accessed August 6, 2025, https://github.com/zju3dv/coxgraph
23. Issues / ethz-asl/voxgraph - GitHub, accessed August 6, 2025, https://github.com/ethz-asl/voxgraph/issues
24. Volumetric Mapping Onboard Unmanned Helicopter - ČVUT DSpace, accessed August 6, 2025, https://dspace.cvut.cz/bitstream/handle/10467/109207/F3-BP-2023-Capek-David-volumetric_mapping_onboard_unmanned_helicopter.pdf
25. Efficient Submap-based Autonomous MAV Exploration using Visual-Inertial SLAM Configurable for LiDARs or Depth Cameras - arXiv, accessed August 6, 2025, https://arxiv.org/html/2409.16972v1
26. Building Maps for Autonomous Navigation Using Sparse Visual SLAM Features - Yonggen Ling, accessed August 6, 2025, https://ygling2008.github.io/papers/IROS2017.pdf
27. Dense Visual SLAM - Department of Computing, accessed August 6, 2025, https://www.doc.ic.ac.uk/~ajd/Publications/newcombe_phd2012.pdf
28. An Efficient and Robust Algorithm for Simultaneously Optimizing Robot Poses and Occupancy Map - arXiv, accessed August 6, 2025, https://arxiv.org/html/2502.06292v1
29. Coxgraph: Multi-Robot Collaborative, Globally Consistent, Online Dense Reconstruction System - Weicai Ye, accessed August 6, 2025, https://ywcmaike.github.io/images/Coxgraph/IROS2021_Coxgraph.pdf
30. What are the advantages and disadvantages of graph based SLAM compared to other methods, like EKF-SLAM, FastSLAM, etc.? - Quora, accessed August 6, 2025, https://www.quora.com/What-are-the-advantages-and-disadvantages-of-graph-based-SLAM-compared-to-other-methods-like-EKF-SLAM-FastSLAM-etc
31. Efficient volumetric mapping of multi-scale environments using wavelet-based compression - Robotics, accessed August 6, 2025, https://www.roboticsproceedings.org/rss19/p065.pdf
32. nvblox/README.md at public / nvidia-isaac/nvblox - GitHub, accessed August 6, 2025, https://github.com/nvidia-isaac/nvblox/blob/public/README.md?plain=1
33. CRADMap: Applied Distributed Volumetric Mapping with 5G-Connected Multi-Robots and 4D Radar Perception - arXiv, accessed August 6, 2025, https://arxiv.org/html/2503.00262v2


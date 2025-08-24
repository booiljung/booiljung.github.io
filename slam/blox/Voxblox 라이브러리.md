[Blox](./index.md)
# Voxblox 라이브러리



자율 이동 로봇, 특히 마이크로 공중 비행체(Micro Aerial Vehicles, MAVs)나 무인 지상 차량(Unmanned Ground Vehicles, UGVs)과 같은 시스템이 미지의 복잡한 환경에서 성공적으로 임무를 수행하기 위해서는 주변 환경을 3차원으로 정확하게 인식하고 이를 지도 형태로 표현하는 능력이 필수적이다.1 이러한 3D 환경 지도는 로봇이 자신의 위치를 추정하고, 장애물을 회피하며, 목표 지점까지 안전하고 효율적인 경로를 계획하는 모든 자율 항법 과정의 근간이 된다.3

특히, MAV와 같이 탑재할 수 있는 페이로드(payload)와 배터리 용량, 그리고 연산 능력이 극히 제한된 플랫폼의 경우, 문제는 더욱 복잡해진다.1 이러한 제약 조건 하에서는 경량화(light-weight)되고, 실시간(real-time)으로 동작하며, 메모리 사용량이 적은 매핑 및 경로 계획 알고리즘이 반드시 필요하다. 기존의 많은 매핑 기법들은 이러한 요구사항을 충족시키지 못했다.

로봇 공학 분야에서 널리 사용되어 온 3D 지도 표현 방식 중 하나는 점유 격자 지도(Occupancy Grid Map)이다. 대표적인 예로 OctoMap은 3차원 공간을 옥트리(octree)라는 계층적 데이터 구조를 사용하여 효율적으로 분할하고, 각 복셀(voxel)의 점유 확률(occupancy probability)을 저장한다.1 이 방식은 공간을 '점유됨(occupied)', '비어있음(free)', '알 수 없음(unknown)'의 세 가지 상태로 구분하는 데에는 효과적이다. 그러나 로봇 경로 계획 기술이 발전함에 따라, 단순한 점유 정보만으로는 부족한 상황이 발생했다. 특히, CHOMP(Covariant Hamiltonian Optimization for Motion Planning)나 TrajOpt와 같이 최적화에 기반한 경로 계획기(trajectory optimization-based planner)들은 로봇의 경로를 부드럽고 동역학적으로 실현 가능한 궤적으로 생성하기 위해, 지도 상의 모든 지점에서 가장 가까운 장애물까지의 연속적인 거리(continuous distance) 정보와 그 거리의 변화율, 즉 그래디언트(gradient) 정보를 필요로 한다.1 OctoMap과 같은 점유 격자 지도는 이러한 정보를 직접적으로 제공하지 못하며, 전체 맵에 대해 배치(batch) 처리 방식으로 장애물 거리 정보를 계산하는 것은 실시간 경로 재계획이 요구되는 동적인 상황에서는 너무 많은 계산 시간을 소요하는 한계를 지닌다.1

이러한 기술적 공백은 단순한 매핑 기술의 개선을 넘어, 로봇 경로 계획 패러다임의 변화에 직접적으로 대응할 수 있는 새로운 형태의 지도 표현 방식의 등장을 촉발했다. 초기 로봇 플래닝이 A*나 RRT*와 같은 그래프 탐색 기반 방식에 의존하여 점유 정보만으로 충분했던 시대에서, MAV와 같이 민첩한 시스템을 위한 빠르고 부드러운 경로 재계획이 중요해지면서 최적화 기반 플래너가 부상했다. 이 새로운 플래너들은 비용 함수(cost function)를 정의하고 최소화하기 위해 장애물까지의 거리와 그래디언트라는 새로운 종류의 "연료"를 필요로 했고, 기존의 매핑 기술은 이 연료를 실시간으로 공급할 수 없었다. 따라서, Voxblox의 등장은 이러한 새로운 플래닝 패러다임의 요구를 충족시키기 위한 필연적인 기술적 진화였으며, 이는 플래닝 알고리즘의 발전이 매핑 기술의 발전을 견인한 구체적인 사례라 할 수 있다.


이러한 배경 속에서 2017년, Helen Oleynikova 연구팀은 Voxblox라는 혁신적인 라이브러리를 제안했다.1 Voxblox의 핵심 기여는 컴퓨터 비전 및 그래픽스 분야에서 널리 사용되던 암시적 표면 표현(implicit surface representation) 방식인 절단 부호 거리 필드(Truncated Signed Distance Field, TSDF)로부터, 경로 계획에 필수적인 유클리드 부호 거리 필드(Euclidean Signed Distance Field, ESDF)를 점진적으로(incrementally) 그리고 실시간으로 구축하는 최초의 프레임워크를 제시한 것이다.1

TSDF는 센서로부터 들어오는 깊이 정보를 여러 번에 걸쳐 누적하고 가중 평균하여 센서 노이즈를 효과적으로 평활화(smooth out)하며, 이를 통해 고품질의 3D 메시(mesh)를 생성하는 데 매우 적합한 특성을 가진다.1 이 메시는 원격의 인간 조종자가 로봇의 주변 환경을 직관적으로 파악하고 상위 수준의 임무 목표를 설정하는 데 도움을 준다. 반면, ESDF는 지도 상의 모든 지점에서 가장 가까운 장애물까지의 실제 유클리드 거리를 저장하고 있어, 최적화 기반 경로 계획기가 충돌 없는 경로를 계산하는 데 필수적인 정보를 제공한다.1

Voxblox는 이 두 가지 상이한 장점을 가진 지도 표현 방식을 하나의 통합된 시스템 안에서 효율적으로 결합했다. 즉, 센서 데이터로부터 실시간으로 TSDF를 구축하고, 이 TSDF의 변경 사항을 바탕으로 ESDF를 점진적으로 업데이트함으로써, 시각화(visualization)와 경로 계획(planning)이라는 두 가지 핵심 요구사항을 동시에 만족시키는 독창적인 해결책을 제시한 것이다.1 더욱이 Voxblox는 전체 시스템이 단일 CPU 코어 상에서 실시간으로 동작 가능하도록 설계되었으며, 모든 소스 코드를 오픈 소스로 공개하여 전 세계 로봇 커뮤니티가 자율 이동 로봇 연구를 가속화하는 데 크게 기여했다.1


본 보고서는 Voxblox 라이브러리에 대한 심층적이고 종합적인 분석을 제공하는 것을 목표로 한다. 이를 위해 다음과 같은 구조로 내용을 전개한다.

- **2장. 기반 데이터 구조 및 알고리즘 심층 분석:** Voxblox의 핵심을 이루는 Voxel Hashing, TSDF 통합, ESDF 생성, 메시 생성 알고리즘의 원리와 수학적 배경을 상세히 분석한다.
- **3장. 소프트웨어 아키텍처 및 ROS 통합:** C++ 라이브러리의 구조와 ROS(Robot Operating System)와의 연동을 위한 `voxblox_ros` 패키지의 인터페이스, 주요 파라미터 등을 살펴본다.
- **4장. 주요 응용 분야 및 실제 적용 사례:** MAV 자율 비행, 지상 로봇 경로 계획 등 실제 로봇 시스템에 Voxblox가 어떻게 적용되었는지 구체적인 사례를 통해 분석한다.
- **5장. 성능 비교 분석 및 기술적 한계:** 기존의 OctoMap 및 후속 프레임워크(nvblox 등)와의 성능을 비교하고, Voxblox가 가진 내재적인 기술적 한계점을 논한다.
- **6장. Voxblox 생태계와 확장 프레임워크:** Voxblox를 기반으로 개발된 `c-blox`, `voxgraph`, `voxblox++` 등 다양한 확장 프레임워크들을 소개하고 그 의의를 탐구한다.
- **7장. 종합적 평가 및 미래 전망:** Voxblox의 기술적 기여와 의의를 종합적으로 평가하고, 최신 학습 기반 매핑 기술과의 관계 속에서 미래를 전망하며 보고서를 마무리한다.


Voxblox의 혁신성은 몇 가지 핵심적인 데이터 구조와 알고리즘의 유기적인 결합에 기반한다. 본 장에서는 동적 맵 확장을 가능하게 하는 Voxel Hashing부터 시작하여, 센서 데이터를 융합하는 TSDF 통합, 경로 계획을 위해 거리를 계산하는 ESDF 생성, 그리고 시각화를 위한 메시 생성에 이르기까지 각 구성 요소의 원리를 심도 있게 파헤친다. 이 분석을 통해 Voxblox가 어떻게 '실시간 CPU 기반 로컬 플래닝'이라는 명확한 목표 아래 철저하게 최적화되었는지를 이해할 수 있다.


전통적인 체적 매핑(volumetric mapping) 방식은 종종 고정된 크기의 3D 복셀 그리드를 사용했다. 이 접근법은 구현이 간단하지만, 사전에 맵의 전체 크기를 알아야 하고 탐사 영역이 넓어질 경우 막대한 양의 메모리를 소모하는 근본적인 한계를 가진다.1 로봇이 미지의 환경을 탐사하는 애플리케이션에서는 맵의 경계를 미리 알 수 없으므로, 이러한 고정 그리드 방식은 부적합하다.

이 문제를 해결하기 위해 Voxblox는 Niessner 등이 제안한 Voxel Hashing 기법을 채택했다.1 Voxel Hashing은 전체 3D 공간을 논리적으로는 무한한 그리드로 간주하되, 실제로 데이터가 존재하는 영역에 대해서만 메모리를 할당하는 동적이고 효율적인 자료 구조이다.10 그 원리는 다음과 같다.

1. **공간 분할 (Space Division):** 전체 3D 공간을 일정한 크기를 가진 '복셀 블록(Voxel Block)'이라는 단위로 분할한다. 예를 들어, 하나의 블록은 ‘16×16×16‘ 개의 복셀로 구성될 수 있다.11
2. **해시 테이블 (Hash Table):** 실제 메모리 할당은 이 블록 단위로 이루어진다. 각 블록은 자신의 3D 정수 좌표(block index)를 가지며, 이 좌표를 키(key)로 사용하여 해시 테이블(hash table)에 블록 데이터의 메모리 주소를 저장한다.1
3. **동적 할당 (Dynamic Allocation):** 로봇이 새로운 영역을 관측하여 이전에 데이터가 없던 블록에 정보를 기록해야 할 경우, 해당 블록을 위한 메모리를 새로 할당하고 그 주소를 해시 테이블에 추가한다.

이러한 Voxel Hashing 구조는 여러 중요한 장점을 제공한다. 첫째, 맵이 탐사 영역에 따라 동적으로 확장될 수 있어, 맵의 크기를 미리 가정할 필요가 없다.1 둘째, 표면이 존재하는 주변의 '국소적으로 조밀한(locally dense)' 영역에 대해서만 메모리를 사용하므로, 전체적으로는 '전역적으로 희소한(globally sparse)' 구조를 가져 메모리 사용량을 크게 절감할 수 있다.11 셋째, 해시 테이블을 사용하므로 특정 블록에 대한 접근(lookup)과 새로운 블록의 삽입(insertion)이 평균적으로 $O(1)$의 시간 복잡도를 가진다.1 이는 계층적 구조로 인해 $O(\log n)$의 접근 시간을 갖는 옥트리(Octree) 방식보다 이론적으로 더 빠르다.1

이러한 속도상의 이점은 Voxblox의 설계 목표와 직결된다. 실시간 경로 재계획을 위해서는 지도 정보에 대한 빠른 쿼리가 필수적이며, Voxel Hashing은 메모리 효율성보다는 접근 속도를 우선시한 전략적 선택이라 할 수 있다.


TSDF는 Voxblox 맵의 가장 기본적인 표현 형태로, 센서로부터 들어온 깊이(depth) 정보를 3D 공간에 통합하는 역할을 한다.


TSDF는 각 복셀이 가장 가까운 표면(surface)까지의 거리를 저장하는 암시적 표현 방식이다. 이 거리는 부호(sign)를 가지는데, 복셀의 중심이 표면의 '바깥쪽'(카메라와 표면 사이의 자유 공간)에 있으면 양수, '안쪽'(표면 뒤의 점유 공간)에 있으면 음수 값을 가진다.5 표면 자체는 이 거리 값이 0이 되는 지점들의 집합, 즉 등위면(isosurface)으로 정의된다.

Voxblox에서 사용하는 TSDF는 한 가지 중요한 특징이 있는데, 바로 '절단(truncation)'이다. 즉, 표면에서 일정 거리 이상 떨어진 복셀들은 모두 최대/최소 절단 값으로 고정된다. 이는 계산량을 줄이고 표면 근처의 정보에 집중하기 위함이다. 구체적으로, 센서(예: RGB-D 카메라)로부터 특정 픽셀에 대한 깊이 측정값 $\mathbf{p}$가 주어졌다고 가정하자. 이 측정값은 센서 원점 $\mathbf{o}$에서부터 광선(ray)을 따라 표면까지의 거리이다. 이 광선 상에 위치한 임의의 복셀 중심 $\mathbf{x}$에 대한 부호 거리는 다음과 같이 계산된다.
$$
sdf(\mathbf{x}) = ||\mathbf{p}||_2 - ||\mathbf{x}||_2
$$
이 값은 센서 광선을 따른 투영 거리(projective distance)이며, 실제 유클리드 거리와는 차이가 있다. 이 부호 거리는 절단 반경(truncation radius) ‘δ‘ 내에서만 유효하며, 이 값을 ‘[−1,1]‘ 범위로 정규화한 최종 TSDF 값 $d(\mathbf{x})$는 다음과 같이 정의된다.1
$$
d(\mathbf{x}) = \max\left(-1, \min\left(1, \frac{sdf(\mathbf{x})}{\delta}\right)\right)
$$
여기서 $\max$와 ‘min‘ 함수는 거리 값을 ‘[−δ,δ]‘ 범위로 절단하는 역할을 한다.


실제 환경에서는 센서 측정값에 노이즈가 포함되어 있고, 여러 각도에서 동일한 표면을 관측하게 된다. 따라서 여러 측정값을 하나의 복셀에 안정적으로 통합(merge)하는 메커니즘이 필요하다. Voxblox는 가중치 기반의 이동 평균(weighted moving average) 방식을 사용한다.14

각 복셀은 TSDF 값 $D(\mathbf{x})$와 함께 가중치 $W(\mathbf{x})$를 저장한다. $k$번째 새로운 측정값 $d_k(\mathbf{x})$와 그에 해당하는 가중치 $w_k$가 들어오면, 기존의 $(k-1)$번째까지 누적된 TSDF 값 $D_{k-1}(\mathbf{x})$와 가중치 $W_{k-1}(\mathbf{x})$는 다음과 같이 갱신된다.
$$
D_k(\mathbf{x}) = \frac{W_{k-1}(\mathbf{x})D_{k-1}(\mathbf{x}) + w_k d_k(\mathbf{x})}{W_{k-1}(\mathbf{x}) + w_k}
$$

$$
W_k(\mathbf{x}) = \min(W_{k-1}(\mathbf{x}) + w_k, W_{max})
$$

여기서 $W_{max}$는 가중치의 최댓값으로, 오래된 정보가 새로운 정보에 의해 과도하게 희석되는 것을 방지한다. 새로운 측정값의 가중치 $w_k$를 어떻게 결정하느냐에 따라 맵의 품질이 달라지는데, Voxblox는 몇 가지 전략을 제공한다. 예를 들어, 모든 측정값에 동일한 가중치를 부여하는 '상수 가중치(constant weight)' 방식이나, 센서 모델에 기반하여 거리에 따라 가중치를 달리하는 '선형 감소(linear drop-off)' 방식 등이 있다.6 이 가중치 기반 통합 방식은 반복적인 관측을 통해 센서 노이즈를 줄이고 보다 정확하고 부드러운 표면을 재구성하는 데 핵심적인 역할을 한다.


Voxblox는 사용자의 다양한 요구사항(정확성, 속도 등)에 대응하기 위해 세 가지 종류의 TSDF 통합기(integrator)를 제공한다.16 이 선택은 시스템의 실시간 성능과 맵의 품질 사이에 직접적인 트레이드오프를 발생시키므로, 애플리케이션의 목적에 맞는 적절한 통합기를 선택하는 것이 중요하다.

- **`simple` 통합기:** 가장 기본적이고 정확한 방식이다. 입력된 포인트 클라우드의 모든 점에 대해 센서 원점으로부터 개별적인 광선 투사(ray casting)를 수행하고, 광선이 통과하는 모든 복셀을 업데이트한다. 구현이 직관적이고 가장 정확한 결과를 보장하지만, 포인트의 수와 광선의 길이에 비례하여 계산량이 많아져 가장 느리다.
- **`merged` 통합기:** 속도와 정확성의 균형을 맞춘 방식이다. 포인트 클라우드의 점들을 먼저 복셀 그리드에 할당한 뒤, 동일한 복셀에 속하는 점들을 하나로 '병합(merge)'한다. 이 병합된 가상의 점으로부터 단 한 번의 광선 투사를 수행하여 관련 복셀들을 업데이트한다. 이 방식은 광선 투사 횟수를 크게 줄여 `simple` 방식보다 훨씬 빠르며, 특히 복셀 크기가 클 때(예: 10 cm 이상) 효율적이다.
- **`fast` 통합기:** 속도를 극대화하기 위해 정확도를 일부 희생하는 방식이다. 이 통합기는 광선 투사 중, 현재 처리 중인 복셀이 이번 스캔의 다른 광선에 의해 이미 업데이트되었는지를 확인한다. 만약 연속적으로 일정 횟수 이상( `max_consecutive_ray_collisions` 파라미터로 설정) 다른 광선이 지나간 복셀을 만나면, 해당 광선은 더 이상 새로운 정보를 제공하지 않는다고 판단하고 조기에 캐스팅을 중단한다. 또한, `start_voxel_subsampling_factor` 파라미터를 통해 입력 포인트 클라우드를 사전에 서브샘플링하여 처리할 데이터 양을 줄인다. 이 방식은 가장 빠르지만, 많은 정보를 의도적으로 버리기 때문에 맵의 정확도가 떨어질 수 있다.

이러한 다양한 통합기 옵션은 Voxblox의 실용성을 보여주는 중요한 특징이다. 고품질의 3D 재구성이 목표라면 `simple`이나 `merged`를, 고속으로 비행하는 MAV의 실시간 장애물 회피가 목표라면 `fast`를 선택하는 등, 사용자가 자신의 하드웨어 제약과 애플리케이션의 우선순위에 따라 성능을 유연하게 튜닝할 수 있다.

| 표 1: TSDF 통합기 유형 비교 (Comparison of TSDF Integrator Types) |
| ------------------------------------------------------------ |
| **통합기 유형**                                              |
| `simple`                                                     |
| `merged`                                                     |
| `fast`                                                       |


TSDF가 표면 근처의 정보를 정확하게 표현하는 데 강점이 있다면, ESDF는 로봇의 경로 계획을 위한 핵심 정보를 제공한다.


ESDF는 이름에서 알 수 있듯이, 맵 상의 모든 복셀에 대해 가장 가까운 장애물(점유된 복셀)까지의 실제 유클리드 거리(Euclidean distance)를 저장하는 필드이다.1 이 거리는 TSDF의 투영 거리와 달리, 방향에 무관한 최단 거리이다. ESDF의 값은 자유 공간에서는 양수, 점유 공간에서는 음수를 가진다.

최적화 기반 경로 계획기는 로봇의 경로를 표현하는 점들의 집합에 대해 비용 함수를 정의하고 이를 최소화하는 방식으로 동작한다. 이때 비용 함수는 주로 '경로의 평활도(smoothness)'와 '장애물과의 충돌 가능성'으로 구성된다. ESDF는 각 지점에서의 장애물까지의 거리를 직접 제공하므로 충돌 비용을 쉽게 계산할 수 있게 해준다. 또한, ESDF를 공간에 대해 미분하면 거리의 그래디언트를 얻을 수 있는데, 이는 장애물로부터 멀어지는 방향을 가리키므로 경로를 안전한 방향으로 밀어내는 힘으로 작용하여 최적화 과정을 효율적으로 만든다.1 따라서, 맵 전체에 걸쳐 정확한 유클리드 거리 정보를 제공하는 ESDF는 현대적인 로봇 경로 계획에 있어 필수불가결한 요소이다.


Voxblox의 가장 핵심적인 기여는 TSDF로부터 ESDF를 배치 방식이 아닌 점진적(incremental) 방식으로 생성하는 알고리즘을 제안한 것이다. 이는 Lau 등이 점유 격자 지도에 대해 제안했던 아이디어를 TSDF 환경에 맞게 확장한 것으로, 파면 전파(wavefront propagation)라는 기법에 기반한다.1 이 알고리즘은 맵의 일부가 변경되었을 때, 전체 맵을 다시 계산하는 대신 변경된 부분으로부터 파동이 퍼져나가듯 거리 정보를 주변으로 전파하여 업데이트를 최소화한다.

알고리즘은 두 개의 우선순위 큐(priority queue)를 사용하여 거리 값의 '증가'와 '감소'를 별도로 처리한다.1

- **`raise` 큐:** 이 큐는 복셀의 장애물까지의 거리가 '증가'해야 하는 경우에 사용된다. 이런 경우는 주로 이전에 관측되었던 장애물이 사실은 센서 노이즈였거나, 동적 장애물이 움직여서 사라졌을 때 발생한다. `raise` 큐에 들어간 복셀은 일단 거리가 무한대(또는 설정된 최댓값)로 초기화된다. 그리고 이 복셀을 가장 가까운 장애물(부모)로 삼고 있던 주변의 다른 복셀들(자식)은 이제 부모를 잃었으므로, 자신들의 거리 값도 무효화되고 다시 계산되어야 한다. 따라서 이 자식 복셀들도 연쇄적으로 `raise` 큐에 추가된다. 이 과정은 마치 장애물이 사라진 지점에서부터 "거리가 멀어져야 한다"는 파동이 퍼져나가는 것과 같다.
- **`lower` 큐:** 이 큐는 복셀의 장애물까지의 거리가 '감소'해야 하는 경우에 사용된다. 이는 새로운 장애물이 관측되었거나, 로봇이 장애물에 더 가까이 다가가서 더 정확한 거리 정보를 얻었을 때 발생한다. `lower` 큐에 들어간 복셀(현재 복셀)은 자신의 거리 정보를 26-연결(26-connected)된 모든 이웃 복셀에게 전파한다. 이웃 복셀은 현재 복셀로부터 전달받은 거리 값(현재 복셀의 거리 값 + 현재 복셀과 이웃 복셀 사이의 유클리드 거리)과 자신의 기존 거리 값을 비교하여, 더 작은 값으로 자신의 거리 값을 갱신한다. 만약 거리 값이 갱신되었다면, 이 이웃 복셀 또한 `lower` 큐에 추가되어 이 과정을 계속해서 주변으로 전파한다. 이는 마치 새로 생긴 장애물 표면에서부터 "거리가 가깝다"는 정보가 파도처럼 퍼져나가는 것과 같다.

전체 ESDF 업데이트 과정은 다음과 같이 요약될 수 있다.1

1. **변경 감지:** TSDF 통합 후, 변경된 TSDF 복셀들을 식별한다.
2. **큐 삽입:** 각 변경된 TSDF 복셀에 대해, ESDF 복셀의 상태를 확인한다.
   - 장애물이 새로 생기거나 가까워졌으면(TSDF 값이 ESDF 값보다 작아짐), 해당 ESDF 복셀을 `lower` 큐에 넣는다.
   - 장애물이 사라졌거나 멀어졌으면(TSDF 값이 ESDF 값보다 커짐), 해당 ESDF 복셀을 `raise` 큐에 넣는다.
3. **`raise` 큐 처리:** `raise` 큐가 빌 때까지 복셀을 하나씩 꺼내(pop) 거리를 최댓값으로 설정하고, 그 영향을 받는 자식들을 찾아 연쇄적으로 `raise` 큐에 넣는다.
4. **`lower` 큐 처리:** `raise` 큐 처리가 끝난 후, `lower` 큐가 빌 때까지 복셀을 하나씩 꺼내 주변 이웃들의 거리를 갱신하고, 갱신된 이웃을 다시 `lower` 큐에 넣는다.

이 파면 전파 알고리즘의 본질적인 특성은 순차성(sequentiality)이다. 우선순위 큐에서 원소를 하나씩 꺼내 처리하고, 그 결과가 다음 처리 대상(이웃)에 영향을 미치는 구조는 대규모 병렬화에 적합하지 않다. 즉, 수천 개의 복셀을 동시에 독립적으로 처리하는 GPU의 아키텍처와는 잘 맞지 않는다. 이 점이 바로 Voxblox의 근본적인 성능 한계점이자, 후속 연구인 `nvblox`가 GPU 가속을 위해 이 알고리즘을 버리고 병렬 처리에 더 적합한 다른 방식(Parallel Banding Algorithm과 유사한 방식)을 채택하게 된 직접적인 기술적 배경이다.18 이는 성공적인 CPU 기반 알고리즘이 어떻게 GPU 시대의 도래와 함께 기술적 부채로 전환될 수 있는지를 보여주는 흥미로운 사례이며, 기술 발전의 역동성을 명확히 드러낸다.


Voxblox는 TSDF로부터 3D 표면 메시를 추출하여 RViz와 같은 시각화 도구에서 보여주는 기능을 제공한다.1 이는 로봇의 환경 인식을 인간이 직관적으로 이해할 수 있도록 돕는 중요한 기능이다. 이 메시 생성 과정에는 Marching Cubes라는 고전적이고 널리 알려진 컴퓨터 그래픽스 알고리즘이 사용된다.19

Marching Cubes 알고리즘의 기본 원리는 다음과 같다.19

1. **큐브 정의:** TSDF 복셀 그리드에서 인접한 8개의 복셀 꼭짓점으로 구성된 가상의 큐브를 정의한다. 이 큐브는 그리드를 따라 한 복셀씩 이동(march)한다.
2. **복셀 분류:** 각 큐브의 8개 꼭짓점에 대해, 저장된 TSDF 값이 0(표면을 의미하는 iso-value)보다 큰지 작은지를 판별한다. 값이 0보다 작거나 같으면(표면 내부 또는 표면 위) '내부'로, 0보다 크면(표면 외부) '외부'로 분류한다.
3. **케이스 인덱싱:** 8개 꼭짓점의 '내부/외부' 상태는 8비트 정수로 표현될 수 있다. 예를 들어, 내부면 1, 외부면 0으로 설정하면 $2^8 = 256$가지의 고유한 케이스가 발생한다. 이 8비트 정수 값은 미리 계산된 조회 테이블(lookup table)의 인덱스로 사용된다.
4. **삼각형 생성:** 이 조회 테이블에는 256개의 각 케이스에 대해 큐브 내부에 생성되어야 할 삼각형들의 구성(어떤 엣지들을 연결해야 하는지)이 저장되어 있다. 알고리즘은 인덱스에 해당하는 삼각형 구성을 테이블에서 찾아온다.
5. **정점 위치 보간:** 생성될 삼각형의 실제 꼭짓점(vertex) 위치는 큐브의 엣지(edge) 상에서 결정된다. 엣지의 양 끝 꼭짓점은 하나는 '내부', 다른 하나는 '외부'일 것이다. 삼각형의 정점은 이 두 꼭짓점의 TSDF 값을 선형 보간(linear interpolation)하여 TSDF 값이 정확히 0이 되는 지점으로 계산된다.

이 과정을 모든 큐브에 대해 반복하면, 전체 TSDF 볼륨에 대한 등위면을 나타내는 삼각형 메시가 생성된다. Voxblox에서는 이 로직이 `MeshIntegrator` 클래스에 구현되어 있다.21

`generateMesh` 함수를 호출하면, 특정 복셀 블록 또는 전체 맵에 대해 이 Marching Cubes 알고리즘을 수행하여 `voxblox::Mesh` 구조체에 정점, 법선(normal), 색상, 인덱스 정보를 채워 넣는다.22 이 메시 데이터는 ROS 메시지 형태로 변환되어 시각화 도구로 전송된다.


Voxblox의 성공적인 보급과 넓은 활용 범위는 잘 설계된 소프트웨어 아키텍처에 크게 힘입었다. 특히, 핵심 알고리즘 로직과 특정 로봇 시스템(ROS)과의 연동 부분을 의도적으로 분리한 설계는 라이브러리의 이식성, 재사용성, 확장성을 극대화하는 결정적인 역할을 했다. 본 장에서는 이러한 소프트웨어 구조를 C++ 라이브러리, ROS 패키지, 그리고 주요 인터페이스를 중심으로 상세히 분석한다.


Voxblox의 핵심 기능은 ROS와 같은 특정 미들웨어에 대한 의존성이 없는 순수 C++ 라이브러리(`voxblox` 라이브러리)로 구현되어 있다.6 이러한 설계는 Voxblox를 ROS 환경뿐만 아니라, 다른 로봇 운영체제나 독립적인 애플리케이션에서도 쉽게 사용할 수 있게 만든다. 이는 라이브러리의 장기적인 생명력과 확장성의 핵심 비결이다. 만약 모든 로직이 ROS에 강하게 결합되어 있었다면, `voxl-mapper` 23와 같은 상용 시스템에 통합되거나 `voxblox_pybind` 24와 같은 Python 래퍼가 개발되기는 훨씬 더 어려웠을 것이다. 이 구조적 분리는 Voxblox가 단순한 ROS 패키지를 넘어 범용적인 3D 매핑 라이브러리로 자리매김하게 한 중요한 설계 결정이었다.

라이브러리의 주요 클래스 및 계층 구조는 다음과 같이 구성된다.

- **`Layer`**: 특정 유형의 복셀 데이터를 관리하는 최상위 컨테이너 클래스이다. Voxel Hashing 구조에서 해시 테이블을 통해 복셀 블록(`VoxelBlock`)들을 관리한다. `TsdfLayer`, `EsdfLayer`, `MeshLayer` 등 각 데이터 유형에 맞는 특화된 레이어가 존재한다.1

- **`Voxel`**: 각 복셀에 저장되는 실제 데이터의 구조체이다. 예를 들어, `TsdfVoxel`은 부호 거리 값, 가중치, 색상 정보를 포함하고, `EsdfVoxel`은 유클리드 거리 값, 가장 가까운 장애물의 위치(부모), 관측 여부 등의 정보를 포함한다.25

- **`Integrator`**: 센서 데이터를 받아 맵을 업데이트하는 알고리즘 로직을 캡슐화한 클래스이다. `TsdfIntegrator`, `EsdfIntegrator`, `MeshIntegrator` 등이 있으며, 각각 TSDF 통합, ESDF 전파, 메시 생성을 담당한다.21

- **데이터 직렬화 (Serialization)**: Voxblox는 구글의 Protobuf(Protocol Buffers) 라이브러리를 사용하여 맵 데이터를 효율적으로 직렬화한다.6 이를 통해 생성된 맵 전체(주로 `Layer` 객체)를 바이너리 파일(`.vxblx`)로 저장하거나 다시 불러올 수 있어, 맵의 영속적인 보관 및 재사용이 가능하다.


`voxblox_ros` 패키지는 위에서 설명한 핵심 C++ 라이브러리를 감싸서(wrapping) ROS 생태계와 연동하는 '어댑터(Adapter)' 역할을 수행한다.6 이 패키지는 ROS의 표준 통신 방식(토픽, 서비스, 파라미터)을 통해 외부 노드(SLAM, 경로 계획, 시각화 등)와 데이터를 주고받을 수 있는 인터페이스를 제공한다.

초기 버전에서는 `voxblox_node`라는 단일 노드가 모든 기능을 수행했지만, 이후 기능에 따라 두 개의 서버(server) 노드로 분리되었다.16

- **`tsdf_server`**: TSDF 맵의 생성과 관리만을 담당한다. 깊이 정보와 포즈를 입력받아 TSDF 맵을 구축하고, 메시나 포인트 클라우드 형태로 시각화 정보를 발행한다.

- **`esdf_server`**: `tsdf_server`의 모든 기능을 포함하면서, 추가적으로 ESDF 맵을 생성하고 관리한다. 내부적으로 `EsdfServer` 클래스는 `TsdfServer` 클래스를 상속받아 ESDF 관련 기능을 확장하는 구조로 되어 있다.25 따라서 ESDF 맵이 필요한 경로 계획 애플리케이션에서는 `esdf_server`를 사용해야 한다.


실제 로봇 시스템에 Voxblox를 통합하려는 개발자는 `tsdf_server` 또는 `esdf_server`와 데이터를 주고받는 방법을 정확히 알아야 한다. 다음 표는 분산된 문서들 16에 흩어져 있는 핵심적인 통신 인터페이스 정보를 체계적으로 정리한 것이다. 이를 통해 개발자는 시스템 통합에 필요한 코드(Publisher, Subscriber, Service Client)를 작성할 때 필요한 모든 정보를 한눈에 파악하고 실수를 줄일 수 있다.


Voxblox의 동작은 ROS 파라미터 서버를 통해 로드되는 YAML 설정 파일에 의해 세밀하게 제어된다. 사용자는 이 파라미터들을 조정하여 자신의 애플리케이션과 하드웨어에 맞게 성능을 최적화할 수 있다. 주요 파라미터들은 다음과 같이 그룹화할 수 있다.16

- **General Parameters (일반 파라미터):**
  - `tsdf_voxel_size`: 복셀 하나의 변 길이(미터 단위). 맵의 해상도를 결정하는 가장 중요한 파라미터.
  - `tsdf_voxels_per_side`: 하나의 복셀 블록에 포함되는 복셀의 수 (한 변 기준). 보통 16으로 설정.
  - `method`: 사용할 TSDF 통합기 유형 선택 (`simple`, `merged`, `fast`).
- **TSDF Integrator Parameters (TSDF 통합기 파라미터):**
  - `truncation_distance`: TSDF의 절단 반경(미터 단위). 보통 `tsdf_voxel_size`의 2~4배로 설정.
  - `max_ray_length_m`: 센서로부터 광선 투사를 수행할 최대 거리.
  - `use_const_weight`: `true`로 설정 시 모든 측정값에 동일한 가중치를 부여.
- **Fast TSDF Integrator Specific Parameters (`fast` 통합기 전용 파라미터):**
  - `start_voxel_subsampling_factor`: 입력 포인트 클라우드를 서브샘플링하는 비율. 값이 클수록 더 많이 샘플링하여 속도가 빨라진다.
  - `max_consecutive_ray_collisions`: 광선 투사를 조기 중단하기 위한 연속 충돌 횟수 임계값.
- **ESDF Integrator Parameters (ESDF 통합기 파라미터):**
  - `esdf_max_distance_m`: ESDF를 계산할 최대 거리. 이 거리 밖의 복셀은 기본값으로 채워진다.
  - `clear_sphere_for_planning`: `true`로 설정 시, 로봇 주변의 일정 반경 내 알 수 없는(unknown) 공간을 안전한(free) 공간으로 간주하여 경로 계획을 용이하게 한다.
- **Output Parameters (출력 파라미터):**
  - `update_mesh_every_n_sec`: 메시를 생성하고 발행하는 주기(초 단위). 0으로 설정 시 자동 업데이트 비활성화.
  - `publish_esdf_map`: `true`로 설정 시 `esdf_map_out` 토픽으로 ESDF 레이어를 주기적으로 발행.


Voxblox는 공식적으로 C++와 ROS 통합에 중점을 두고 개발되었지만, 커뮤니티에 의해 Python 환경에서 Voxblox의 핵심 기능을 사용할 수 있도록 하는 래퍼(wrapper) 라이브러리들이 개발되었다. 대표적인 예로 `voxbloxpy` 29와 `voxblox_pybind` 24가 있다.

이러한 Python 래퍼들은 `pybind11`과 같은 도구를 사용하여 C++로 작성된 Voxblox의 핵심 클래스와 함수들을 Python에서 직접 호출할 수 있도록 연결해준다. 이를 통해 사용자는 무거운 ROS 생태계를 전부 설치하지 않고도, Python 스크립트 내에서 직접 포인트 클라우드와 포즈 데이터를 입력하여 TSDF를 생성하고 메시를 추출하는 등의 실험을 간편하게 수행할 수 있다.24 이는 특히 머신러닝 연구나 빠른 프로토타이핑이 중요한 경우에 매우 유용하며, Voxblox의 핵심 로직이 ROS로부터 독립적으로 설계되었기에 가능한 일이다.


Voxblox는 이론적 우수성을 넘어 실제 로봇 시스템에 성공적으로 적용되어 그 실용성을 입증했다. 본 장에서는 MAV의 자율 비행부터 지상 로봇의 경로 계획, 그리고 일반적인 3D 재구성에 이르기까지 다양한 응용 분야를 구체적인 프로젝트 사례와 함께 살펴본다. 이 사례 분석을 통해 Voxblox가 '경로 계획을 위한 맵'과 '인간을 위한 맵'이라는 두 가지 목적을 어떻게 수행하며, 각 목적에 따라 어떤 강점과 실용적 한계를 보이는지 명확히 이해할 수 있다.


Voxblox가 탄생한 주된 동기는 페이로드와 연산 능력이 제한된 MAV의 실시간 지역 경로 계획(local planning)을 지원하기 위함이었다.1 이 목표는 `mav_voxblox_planning`이라는 오픈소스 프로젝트를 통해 구체적으로 실현되었다.30

`mav_voxblox_planning`은 Voxblox를 핵심적인 지도 표현(map representation) 방식으로 사용하여 MAV를 위한 다양한 경로 계획 도구를 제공하는 ROS 패키지 모음이다.30 이 시스템의 핵심 구성 요소는 `mav_local_planner`로, 이는 최적화 기반의 `loco` (Lokal-optimierer für Quadrokopter) 플래너를 사용하여 주어진 웨이포인트를 추종하거나, 웨이포인트 목록 사이를 부드럽게 통과하는 경로를 생성한다.30 이 과정에서 플래너는 `esdf_server`로부터 실시간으로 제공되는 ESDF 맵을 활용하여 장애물과의 충돌을 회피한다. ESDF 맵이 제공하는 연속적인 거리 정보와 그래디언트는 플래너가 경로를 안전하고 효율적으로 최적화하는 데 결정적인 역할을 한다.

이 프로젝트는 Gazebo 시뮬레이션 환경에서 MAV가 미지의 환경을 탐색하며 4Hz의 주기로 실시간 경로 재계획을 수행하는 데모를 포함하고 있다.30 사용자는 RViz(ROS Visualization tool)에 통합된 `PlanningPanel`이라는 플러그인을 통해 그래픽 인터페이스 상에서 시작점과 목표점을 인터랙티브 마커로 지정할 수 있다. 그 후, RRT(Rapidly-exploring Random Tree)나 Skeleton Planner와 같은 다양한 전역 경로 계획기(global planner)를 호출하여 Voxblox 맵 위에 초기 경로를 생성하고, 이를 `mav_local_planner`가 실시간으로 추종하며 동적인 환경 변화에 대응하도록 할 수 있다.30

더 나아가, 실제 MAV에 이 시스템 전체를 탑재하여 외부의 모션 캡처 시스템이나 GPS 없이, 오직 온보드(on-board) 센서(예: 스테레오 카메라)와 온보드 컴퓨터의 연산만으로 미지의 실내 환경을 탐색하고 마네킹과 같은 장애물을 성공적으로 회피하며 비행하는 실험이 성공적으로 수행되었다.1 이는 Voxblox가 이론뿐만 아니라 실제 하드웨어 제약 하에서도 실시간 자율 비행을 가능하게 하는 강력한 도구임을 명백히 보여주는 사례이다.


Voxblox의 활용성은 공중 로봇에만 국한되지 않는다. 그 핵심 원리는 지상 로봇의 경로 계획에도 효과적으로 적용될 수 있다.26 대표적인 사례로 `3d_coverage_path_planning` GitHub 저장소에서 제안된 시스템을 들 수 있다.32

이 프로젝트의 목표는 건설 현장의 진행 상황을 효율적으로 모니터링하기 위해, 자율 이동 지상 로봇이 사전에 구축된 3D 메시 모델 전체를 센서(예: LiDAR)로 빈틈없이 훑을 수 있는 최적의 커버리지 경로(coverage path)를 생성하는 것이다.32 이 시스템에서 Voxblox는 초기 3D 모델, 즉 기준이 되는 메시를 생성하는 데 사용된다. 로봇이 환경을 한 바퀴 주행하며 수집한 센서 데이터는 `hector_sensor_proc_launch/launch/voxblox.launch` 설정을 통해 Voxblox의 TSDF 맵으로 통합되고, 여기서부터 3D 메시가 추출된다.32

여기서 주목해야 할 실용적인 한계점이 드러난다. Voxblox의 메시 생성기는 Voxel Hashing 구조의 특성상, 내부적으로 분리된 복셀 블록 단위로 메시 조각들을 생성한다. 따라서 최종적으로 얻어지는 전체 메시는 위상적으로 완벽하게 연결된 단일 메시가 아니라, 블록 경계에서 단절된 부분들이 존재할 수 있다. 이로 인해, 생성된 메시를 커버리지 경로 계획과 같은 정교한 알고리즘에 직접 사용하기 어렵다.32

`3d_coverage_path_planning` 프로젝트에서는 이 문제를 해결하기 위해 MeshLab과 같은 외부 도구를 사용하여 Voxblox가 생성한 메시를 불러와 불필요한 부분을 정리하고, 표면을 다시 균일하게 재구성하는 리메싱(remeshing) 후처리 과정을 거친다.

이 후처리된 고품질의 메시가 비로소 전역 경로 계획 알고리즘의 입력으로 사용된다. 알고리즘은 이 메시 모델을 기반으로 로봇이 전체 표면을 가장 효율적으로 관측할 수 있는 시점(viewpoint)들을 계산하고, 이 시점들을 최단 경로로 연결하는 최종 주행 경로를 생성한다. 이 사례는 Voxblox의 ESDF가 '기계가 읽는 경로 계획용 맵'으로서 매우 강력하지만, TSDF에서 생성된 메시가 '기계가 사용하는 정밀 모델'로서는 추가적인 처리 과정이 필요할 수 있음을 보여준다. 즉, '인간을 위한 시각화'와 '알고리즘을 위한 정밀 모델' 사이에는 간극이 존재하며, 이를 이해하는 것이 Voxblox를 실제 응용에 적용할 때 매우 중요하다.


Voxblox는 경로 계획이라는 주된 목적 외에도, 환경의 3차원 모델을 구축하는 순수한 체적 맵핑(volumetric mapping) 및 3D 재구성(3D reconstruction) 작업에도 널리 사용된다.3

TSDF 통합 과정에서 센서 노이즈가 평활화되기 때문에, Marching Cubes 알고리즘을 통해 추출된 메시는 일반적으로 포인트 클라우드를 직접 시각화하는 것보다 훨씬 깨끗하고 보기 좋은 표면을 제공한다. 이 고품질의 메시는 원격에 있는 인간 조종자에게 로봇이 처한 환경에 대한 명확하고 직관적인 시각 정보를 제공하여, 상황 인식을 돕고 상위 수준의 의사결정을 지원하는 데 매우 유용하다.1

또한, 이렇게 생성된 체적 모델은 다양한 산업 분야에서 정량적인 분석에 활용될 수 있다. 예를 들어, 건설 현장이나 광산에서 쌓아놓은 흙더미(stockpile)나 채굴된 영역을 3D로 스캔하여 모델을 생성하면, 그 부피(volume)를 정확하게 계산하는 체적 분석(volumetric analysis)이 가능하다.3 이는 자재 관리, 작업량 추정, 안전 관리 등에서 비용을 절감하고 효율성을 높이는 데 기여할 수 있다.

Voxblox의 문서와 튜토리얼에서는 다양한 공개 데이터셋(예: KITTI 자동차 주행 데이터셋, EuRoC MAV 실내 비행 데이터셋, Cow and Lady 실내 스캔 데이터셋)을 사용하여 생성된 다채로운 맵 예시를 제공한다.14 이러한 예시들은 LiDAR, 스테레오 카메라 등 각기 다른 센서와 실내, 실외 등 다양한 환경에서 Voxblox가 얼마나 강건하고 유연하게 고품질의 3D 맵을 생성할 수 있는지를 질적으로 보여준다.


Voxblox는 등장 당시 혁신적인 프레임워크였지만, 기술은 끊임없이 발전한다. Voxblox의 진정한 가치와 위치를 이해하기 위해서는 기존 기술과의 비교를 통해 어떤 점을 개선했는지, 그리고 후속 기술과의 비교를 통해 어떤 한계를 가졌는지를 객관적으로 분석해야 한다. 본 장에서는 Voxblox를 대표적인 기존 프레임워크인 OctoMap, 그리고 주요 후속 프레임워크들과 비교하고, 그 과정에서 드러나는 Voxblox의 내재적 기술 한계를 심도 있게 논한다. 이 분석을 통해 Voxblox가 특정 시대의 요구에 부응한 '충분히 좋은(good enough)' 실용적인 절충안이었으며, 그 명확한 한계점들이 어떻게 후속 연구의 디딤돌이 되었는지를 파악할 수 있다.


OctoMap은 Voxblox 이전에 MAV 매핑 등에서 널리 사용되던 대표적인 3D 매핑 프레임워크이다.1 Voxblox는 OctoMap의 한계를 극복하기 위해 여러 측면에서 다른 설계 철학을 채택했다.

- **데이터 구조 및 접근 속도:** OctoMap은 공간을 표현하기 위해 옥트리(Octree)라는 계층적 트리 구조를 사용한다. 이는 메모리 효율성이 높지만, 특정 복셀에 접근하기 위해서는 트리의 루트부터 탐색해야 하므로 $O(\log n)$의 시간 복잡도를 가진다. 반면, Voxblox는 Voxel Hashing을 사용하여 할당된 블록에 $O(1)$의 시간 복잡도로 직접 접근할 수 있다.1 이로 인해 맵 업데이트나 쿼리가 빈번한 실시간 애플리케이션에서는 Voxblox가 더 빠른 성능을 보인다.
- **맵 표현 및 정보:** OctoMap의 각 복셀은 점유 확률(occupancy probability)만을 저장한다. 이는 공간이 비었는지, 찼는지를 확률적으로 표현할 뿐, 장애물까지의 거리에 대한 정보는 포함하지 않는다.1 최적화 기반 플래너가 요구하는 거리 및 그래디언트 정보를 얻기 위해서는 생성된 점유 맵을 기반으로 별도의 배치(batch) 연산을 통해 ESDF를 계산해야만 한다. 반면, Voxblox는 TSDF와 ESDF를 통해 거리 정보를 맵 표현 자체에 내재하고 있으며, 특히 ESDF를 점진적으로 업데이트하여 실시간 경로 계획에 필요한 정보를 효율적으로 제공한다.1
- **통합 속도 및 정확도:** Voxblox의 원 논문에서는 자신들의 TSDF 통합 방식이 OctoMap보다 빠르다고 주장했으며 1, VDBFusion 논문에서 수행된 벤치마크에서도 Voxblox가 OctoMap보다 약 2~3배 빠른 통합 속도를 보이는 것으로 나타났다.35 정확도 측면에서는, TSDF가 여러 센서 측정값을 가중 평균하여 노이즈를 줄이고 표면을 부드럽게 표현하므로, 이로부터 ESDF를 생성하는 것이 불연속적인 점유 확률 맵에서 생성하는 것보다 더 정확한 거리 정보를 제공할 수 있다고 주장된다.1


Voxblox의 성공은 수많은 후속 연구를 촉발시켰다. 이 후속 프레임워크들은 Voxblox의 한계점을 명확한 개선 목표로 삼아 발전했다.

- **nvblox:** NVIDIA에서 개발한 `nvblox`는 Voxblox의 아이디어를 GPU(Graphics Processing Unit) 환경으로 옮겨온 직접적인 후계자이다.36
  - **성능:** 매핑 파이프라인의 모든 과정(TSDF 통합, ESDF 생성, 메시 생성)을 CUDA를 통해 GPU에서 대규모로 병렬 처리한다. 이로 인해 Voxblox 대비 TSDF 생성 속도는 최대 177배, ESDF 생성 속도는 최대 31배까지 향상되는 압도적인 성능을 보여준다.26
  - **정확도:** Voxblox가 계산 편의를 위해 사용했던 유사 유클리드(quasi-Euclidean) 거리 방식의 부정확성을 개선하여, 완전한 유클리드(full Euclidean) 거리를 계산함으로써 ESDF의 정확도를 높였다.26
  - **알고리즘:** Voxblox의 순차적인 파면 전파 ESDF 알고리즘이 GPU 병렬화에 적합하지 않다는 점을 인지하고, 병렬 처리에 훨씬 효율적인 새로운 ESDF 생성 알고리즘을 도입했다.18
- **Voxfield:** Voxblox의 정확도와 효율성을 CPU 환경 내에서 개선하는 데 초점을 맞춘 프레임워크이다.37
  - **정확도:** Voxblox가 사용하는 센서 광선 방향의 투영(projective) 거리 대신, 비투영(non-projective) 방식의 TSDF 통합을 도입하여 맵의 정확도를 향상시켰다.37
  - **효율성:** ESDF 통합 알고리즘을 개선하여 Voxblox 대비 런타임을 최대 42%까지 단축시켰다.26
- **FIESTA:** TSDF를 거치지 않고 점유 맵에서 직접 ESDF를 생성하는 접근법이지만, Voxblox의 점진적 업데이트 아이디어에 영향을 받았다.
  - **효율성:** 새로운 데이터 구조와 효율적인 업데이트 전략을 통해, Voxblox보다 ESDF 계산 속도를 약 4배 향상시키면서도 완전한 유클리드 거리를 계산한다.26

다음 표는 Voxblox를 중심으로 OctoMap, 그리고 대표적인 후계자인 nvblox의 핵심적인 특징을 비교하여 각 프레임워크의 기술적 패러다임과 장단점을 명확히 보여준다.

| 표 3: 매핑 프레임워크 비교 (Mapping Framework Comparison: Voxblox vs. OctoMap vs. nvblox) |
| ------------------------------------------------------------ |
| **특성**                                                     |
| **주 데이터 표현**                                           |
| **거리 정보**                                                |
| **데이터 구조**                                              |
| **맵 접근 시간**                                             |
| **주 사용 하드웨어**                                         |
| **ESDF 생성 방식**                                           |
| **동적 환경 지원**                                           |
| **글로벌 일관성**                                            |


앞선 비교를 통해 Voxblox가 가진 몇 가지 명백한 기술적 한계점을 정리할 수 있다. 이러한 한계점들은 실패라기보다는, '제한된 자원에서의 실시간 로컬 플래닝'이라는 특정 문제에 집중하기 위한 의도적인 '선택과 집중'의 결과물로 해석하는 것이 타당하다.

- **CPU-only 아키텍처:** Voxblox는 GPU 가속을 전혀 사용하지 않는다.6 이는 저전력 임베디드 보드에서도 구동될 수 있다는 장점이 있지만, 동시에 고해상도(작은 복셀 크기)의 맵을 구축하거나, 초당 수십만 개의 포인트를 쏟아내는 최신 LiDAR 센서 데이터를 실시간으로 처리하는 데에는 성능적 한계를 가진다. 이 한계는 

  `nvblox`의 등장을 촉발한 가장 직접적인 원인이었다.26

- **유사 유클리드 거리의 오차:** ESDF 계산 시, 계산 속도를 높이기 위해 완전한 유클리드 거리가 아닌 근사치(quasi-Euclidean distance)를 사용한다. 이로 인해 실제 거리와 최대 8.25%의 오차가 발생할 수 있으며, 이 때문에 논문 저자들은 로봇의 충돌 반경을 이 오차만큼 부풀려서(inflate) 사용하는 것을 권장한다.1 이는 안전이 중요한 애플리케이션에서는 무시할 수 없는 단점이며, 

  `nvblox`나 `Voxfield`는 이 문제를 해결하기 위해 완전 유클리드 거리 계산 방식을 도입했다.26

- **정적 환경 가정:** Voxblox의 기본 모델은 환경이 정적(static)이라고 가정한다. 따라서 사람이나 다른 로봇과 같은 동적 장애물이 환경에 존재할 경우, 이들의 움직임이 맵에 잔상(artifact)이나 고스트(ghost) 장애물로 남아 경로 계획을 방해할 수 있다.38 동적 객체를 명시적으로 감지하고 처리하는 기능은 라이브러리 자체에 포함되어 있지 않다.

- **글로벌 일관성 부재:** Voxblox는 본질적으로 로컬(local) 매핑 프레임워크이다. 따라서 장시간 동안 넓은 영역을 매핑할 경우, SLAM 시스템에서 발생하는 누적된 위치 추정 오차(drift)로 인해 맵의 시작 부분과 끝 부분이 어긋나는 등의 전역적인 비일관성(global inconsistency) 문제가 발생한다. 이 문제를 해결하기 위해서는 `c-blox`나 `voxgraph`와 같이 포즈 그래프 최적화(pose graph optimization)를 수행하는 별도의 상위 레벨 프레임워크와의 결합이 필수적이다.39

결론적으로, Voxblox의 이러한 명확한 한계점들은 후속 연구 커뮤니티에게 "Voxblox보다 더 빠르게(GPU 가속)", "Voxblox보다 더 정확하게(완전 유클리드 거리)", "Voxblox에 없는 기능을 추가하여(동적 환경, 글로벌 일관성)"와 같은 구체적인 연구 목표를 제시하는 역할을 했다. 즉, Voxblox는 완벽한 최종 솔루션이 아니라, 다음 세대의 혁신을 이끌어낸 중요한 '디딤돌(stepping stone)'로서 로봇 매핑 기술 발전에 기여한 것이다.


Voxblox의 가장 큰 성공 요인 중 하나는 라이브러리 자체의 성능뿐만 아니라, 다른 연구자들이 더 복잡하고 어려운 문제를 해결하기 위한 '기반 블록(foundational block)'으로 쉽게 활용할 수 있도록 설계된 모듈성과 개방성에 있다. Voxblox는 의도적으로 '실시간 로컬 매핑'이라는 핵심 문제에 집중함으로써, 글로벌 일관성, 대규모 매핑, 시맨틱 이해와 같은 상위 레벨의 문제들을 다른 연구자들이 도전할 수 있는 영역으로 남겨두었다. 이러한 '플랫폼화' 전략은 커뮤니티의 활발한 참여를 유도하여, Voxblox를 중심으로 한 풍부한 확장 생태계를 형성하는 원동력이 되었다. 본 장에서는 그 대표적인 확장 프레임워크들을 소개한다.


`c-blox`는 Voxblox를 기반으로 서브매핑(sub-mapping) 기능을 추가하여 대규모 환경에서의 매핑 문제를 해결하고자 하는 라이브러리이다.40

로봇이 넓은 영역을 장시간 동안 이동하며 매핑을 수행할 때, SLAM 시스템의 작은 위치 추정 오차(drift)가 계속해서 누적된다. 이는 결국 맵의 전역적인 일관성을 깨뜨려, 이전에 방문했던 장소로 돌아왔을 때 맵이 이중으로 그려지거나 뒤틀리는 현상을 유발한다.4

`c-blox`는 이 문제에 대응하기 위해, 전체 맵을 하나의 거대한 단일 맵으로 관리하는 대신, 여러 개의 작은 '서브맵(submap)'으로 분할하여 관리하는 전략을 사용한다.40

각 서브맵은 그 자체로 독립적인 Voxblox 맵(TSDF/ESDF Layer)이다. 로봇이 일정 거리 이상 이동하거나, 시간이 경과하면 새로운 서브맵을 생성하여 매핑을 이어나간다. 이렇게 생성된 서브맵들은 각각 자신의 로컬 좌표계를 가지며, 이들 간의 상대적인 위치 관계는 별도로 관리된다. 이 구조는 나중에 포즈 그래프 최적화(Pose Graph Optimization)와 같은 기법을 통해 서브맵 간의 위치 관계를 전역적으로 보정하여, 누적된 오차를 분산시키고 전체 맵의 일관성을 회복할 수 있는 강력한 기반을 제공한다.40


`voxgraph`는 `c-blox`의 서브맵 개념을 한 단계 더 발전시켜, 전역적으로 일관된(globally consistent) 체적 맵을 실시간으로 생성하는 것을 목표로 하는 완전한 프레임워크이다.4

`voxgraph`는 생성된 각 서브맵을 하나의 노드(node)로 간주하는 포즈 그래프(pose graph)를 구축한다. 그리고 로봇이 이동하며 새로운 서브맵을 생성할 때, 이전에 생성된 서브맵과 현재 서브맵이 공간적으로 겹치는(overlapping) 영역이 있다면, 두 서브맵 사이의 상대적인 변환(relative transformation)을 계산하여 이를 그래프 상의 엣지(edge), 즉 제약 조건(constraint)으로 추가한다.39

`voxgraph`의 핵심적인 효율성은 이 제약 조건을 계산하는 방식에 있다. 두 서브맵의 포인트 클라우드를 정합하는 대신, 각 서브맵이 가진 SDF 표현을 직접적으로 활용하여 대응점 없이(correspondence-free) 정합을 수행한다. 이는 계산적으로 매우 효율적이어서, 새로운 서브맵이 추가될 때마다 전체 포즈 그래프를 다시 최적화하는 것이 가능하다. 또한, 루프 클로저(loop closure, 로봇이 과거에 방문했던 장소를 다시 인식하는 것)나 GPS와 같은 외부의 절대 위치 정보를 그래프에 추가적인 제약 조건으로 통합함으로써, 맵의 전역적인 정확도와 일관성을 극적으로 향상시킬 수 있다.39


`voxblox++`는 Voxblox의 순수한 기하학적(geometric) 맵에 '의미(semantics)'를 부여하는 것을 목표로 하는 확장 프레임워크이다.41 이는 로봇이 "여기에 무언가 있다"를 넘어 "여기에 '의자'가 있다"고 이해하게 만드는, 보다 높은 수준의 환경 인식을 가능하게 한다.

이 프레임워크는 RGB-D 카메라로부터 들어오는 컬러 영상과 깊이 정보를 함께 사용하여, 맵 상에 존재하는 개별 객체 인스턴스(object instances)에 대한 정보를 기하학적 맵과 함께 통합한다. 예를 들어, 딥러닝 기반의 객체 인식기를 통해 영상에서 '의자'를 발견하면, `voxblox++`는 해당 의자의 의미론적 레이블('chair'), 3D 공간에서의 정확한 포즈(pose), 그리고 조밀한 3D 형태(dense shape)를 복원하여 맵에 저장한다.41

더 나아가, 사전에 학습되지 않아 인식할 수 없는 새로운 물체가 나타나더라도, 이를 '미지의 객체(unknown object)'로 발견하고 분할하여 독립적인 인스턴스로서 맵에 추가하는 기능도 포함하고 있다. 이러한 객체 수준의 시맨틱 맵은 로봇이 "의자 옆으로 가라" 또는 "테이블 위의 물건을 집어라"와 같이 인간과 보다 자연스럽게 상호작용하고, 복잡한 임무를 수행하는 데 필수적인 정보를 제공한다.41

다음 표는 Voxblox를 기반으로 한 주요 확장 프레임워크들의 핵심 목표와 아이디어를 요약하여, 독자가 자신의 연구 또는 개발 목표에 따라 어떤 후속 기술을 탐구해야 할지 방향을 제시하는 로드맵 역할을 한다.

| 표 4: Voxblox 확장 프레임워크 개요 (Overview of Voxblox Extension Frameworks) |
| ------------------------------------------------------------ |
| **프레임워크**                                               |
| **`c-blox`**                                                 |
| **`voxgraph`**                                               |
| **`voxblox++`**                                              |


Voxblox는 로봇 공학, 특히 자율 비행체 분야의 3D 매핑 기술 역사에서 중요한 이정표로 기록된다. 본 장에서는 Voxblox의 기술적 기여와 의의를 종합적으로 평가하고, 빠르게 발전하는 학습 기반 매핑 기술과의 관계 속에서 그 현재적 가치와 미래를 전망하며 보고서를 마무리한다.


Voxblox의 가장 큰 기여는 **최적화 기반 경로 계획의 온보드(on-board) 적용을 대중화**했다는 점에 있다. Voxblox 이전에도 ESDF의 개념이나 최적화 기반 플래너는 존재했지만, 제한된 연산 자원을 가진 로봇 위에서 실시간으로 ESDF를 생성하고 업데이트하는 것은 매우 어려운 과제였다. Voxblox는 TSDF로부터 ESDF를 점진적으로 생성하는 효율적인 알고리즘을 최초로 구현하고, 이를 사용하기 쉬운 오픈소스 라이브러리로 공개함으로써 이 문제를 해결했다.1 이로 인해 전 세계의 수많은 연구자와 개발자들이 자신의 로봇에 고성능 경로 계획 알고리즘을 비교적 쉽게 탑재할 수 있게 되었고, 이는 MAV의 자율 항법 기술 발전을 크게 가속화했다.

또한, Voxblox는 **실용성을 극대화한 설계 철학**을 보여주었다. Voxel Hashing을 통한 효율적인 동적 맵 관리, 사용자의 요구에 따라 속도와 정확성을 트레이드오프할 수 있는 다양한 TSDF 통합기 옵션 제공 등은 실제 로봇 시스템을 개발하는 과정에서 마주치는 현실적인 문제들을 깊이 고려한 결과물이다.1

마지막으로, Voxblox는 **후속 연구를 위한 강력한 플랫폼**으로서의 역할을 충실히 수행했다. 앞서 6장에서 살펴본 바와 같이, `c-blox`, `voxgraph`, `voxblox++`, 그리고 `nvblox`에 이르기까지 수많은 중요 후속 연구들이 Voxblox의 코드와 아이디어를 기반으로 탄생했다. 이는 Voxblox가 단순히 하나의 잘 만들어진 라이브러리를 넘어, 로봇 매핑 커뮤니티 전체의 발전을 이끈 하나의 생태계의 구심점이 되었음을 의미한다.


최근 로봇 매핑 분야에서는 딥러닝 기술을 활용한 새로운 패러다임이 부상하고 있다. 특히, NeRF(Neural Radiance Fields)나 기타 암시적 신경망 표현(Implicit Neural Representation)을 사용하여 3D 공간과 SDF를 학습하는 연구들이 주목받고 있다.38

이러한 학습 기반 접근법들은 몇 가지 매력적인 장점을 가진다.

- **연속적인 표현 (Continuous Representation):** 복셀 그리드와 같은 이산적인 표현이 아닌, 신경망을 통해 공간을 연속적으로 표현하므로 임의의 해상도에서 맵 정보를 쿼리할 수 있다.
- **메모리 효율성 (Memory Efficiency):** 거대한 복셀 그리드를 직접 저장하는 대신, 상대적으로 작은 크기의 신경망 가중치(weights)로 맵을 압축하여 표현하므로 메모리 사용량이 적을 수 있다.
- **추론 능력 (Inference Capability):** 학습된 모델은 관측되지 않은 영역에 대해서도 기하학적으로 그럴듯한 형태로 표면을 추론(hallucinate)하거나 보간(interpolate)할 수 있는 잠재력을 가진다.26

하지만 이러한 장점에도 불구하고, 학습 기반 방식은 아직 해결해야 할 과제들을 안고 있다. 일반적으로 학습과 추론 과정에서 막대한 계산량을 요구하며 고성능 GPU가 필수적이다.38 또한, 학습 데이터에 존재하지 않는 형태의 환경에 대해서는 성능이 저하될 수 있으며, 신경망의 "블랙박스"적인 특성으로 인해 결과의 예측 가능성과 해석 가능성이 떨어진다는 단점이 있다.

이러한 상황에서 Voxblox와 같은 고전적인(explicit) 방식의 가치는 여전히 유효하다. Voxblox는 알고리즘의 동작 방식이 명확하여 결과를 예측하고 디버깅하기 용이하며, CPU만으로도 안정적으로 실시간 동작이 보장된다는 신뢰성을 제공한다.

미래에는 이 두 방식의 장점을 결합하려는 하이브리드(hybrid) 접근법이 더욱 중요해질 것으로 전망된다. 예를 들어, LGSDF 42와 같이 Voxblox와 유사한 명시적 그리드(local grid)를 사용하여 실시간으로 관측 정보를 빠르고 강건하게 통합하고, 이를 신경망(global implicit map)을 학습시키는 데이터로 활용하는 연구가 등장하고 있다. 이러한 방식은 명시적 방법의 신뢰성과 실시간성을 바탕으로, 학습 기반 방법의 연속성과 압축성의 이점을 취하려는 시도이다. 따라서 Voxblox의 핵심 아이디어와 알고리즘은 미래의 하이브리드 매핑 시스템에서도 중요한 구성 요소로 계속해서 활용될 가능성이 높다.


Voxblox는 2010년대 후반, 자율 이동 로봇, 특히 MAV가 실시간으로 복잡한 환경을 탐색하고 경로를 계획하는 데 필요한 핵심적인 기술적 난제를 해결한 기념비적인 소프트웨어 라이브러리이다. TSDF와 ESDF를 점진적으로 결합하는 독창적인 아이디어, Voxel Hashing을 통한 효율적인 데이터 구조, 그리고 실제 시스템에 적용할 수 있도록 고려된 실용적인 설계는 로봇 매핑 기술의 수준을 한 단계 끌어올렸다.

비록 GPU 가속의 부재나 정적 환경 가정과 같은 기술적 한계로 인해 `nvblox`와 같은 후속 기술에 일부 자리를 내주었지만, Voxblox의 설계 철학과 핵심 알고리즘, 그리고 개방형 아키텍처는 오늘날의 로봇 매핑 기술 연구와 개발에 여전히 깊은 영감과 유효한 교훈을 제공한다. 그것은 단순히 과거의 유물이 아니라, 수많은 후속 연구의 기반이 되었고, 지금도 로봇 공학 교육과 연구 현장에서 널리 사용되는 중요한 기술적 자산으로 굳건히 자리매김하고 있다.


1. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - Helen Oleynikova, 8월 2, 2025에 액세스, https://helenol.github.io/publications/iros_2017_voxblox.pdf
2. Autonomous Visual Mapping and Exploration With a Micro Aerial Vehicle - Ethz, 8월 2, 2025에 액세스, https://people.inf.ethz.ch/pomarc/pubs/HengJFR14.pdf
3. 3D Mapping and Volume Estimation Made Easy with UAVs: Revolutionizing Industrial Applications - ideaForge Mining Drones for Surveying, Monitoring & Compliance, 8월 2, 2025에 액세스, https://us.ideaforgetech.com/3d-mapping-and-volume-estimation-made-easy-with-uavs-revolutionizing-industrial-applications/
4. Volumetric Mapping Onboard Unmanned Helicopter - ČVUT DSpace, 8월 2, 2025에 액세스, https://dspace.cvut.cz/bitstream/handle/10467/109207/F3-BP-2023-Capek-David-volumetric_mapping_onboard_unmanned_helicopter.pdf
5. Voxblox: Building 3D Signed Distance Fields for Planning - Research Collection - ETH Zürich, 8월 2, 2025에 액세스, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/128028/eth-50485-01.pdf
6. ethz-asl/voxblox: A library for flexible voxel-based mapping, mainly focusing on truncated and Euclidean signed distance fields. - GitHub, 8월 2, 2025에 액세스, https://github.com/ethz-asl/voxblox
7. [1611.03631] Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/abs/1611.03631
8. Voxblox: Incremental 3D Euclidean Signed Distance Fields for on-board MAV planning, 8월 2, 2025에 액세스, https://openreview.net/forum?id=OBvr7yV94x
9. Real-time 3D Reconstruction at Scale using Voxel Hashing - Matthias Niessner, 8월 2, 2025에 액세스, https://niessnerlab.org/papers/2013/4hashing/niessner2013hashing.pdf
10. A Task-Agnostic, Fully-Scalable Voxel Mapping System for Large Environment - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2409.15779v1
11. Voxel Block Grid - Open3D 0.19.0 documentation, 8월 2, 2025에 액세스, https://www.open3d.org/docs/release/tutorial/t_reconstruction_system/voxel_block_grid.html
12. Voxel hashing data structure. The map is stored as a hash table. The... - ResearchGate, 8월 2, 2025에 액세스, https://www.researchgate.net/figure/oxel-hashing-data-structure-The-map-is-stored-as-a-hash-table-The-hash-function-maps_fig1_339710963
13. Octomap comparison. / Issue #28 / ethz-asl/voxgraph - GitHub, 8월 2, 2025에 액세스, https://github.com/ethz-asl/voxgraph/issues/28
14. Voxblox - voxblox documentation, 8월 2, 2025에 액세스, https://voxblox.readthedocs.io/en/latest/
15. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning, 8월 2, 2025에 액세스, https://huggingface.co/papers/1611.03631
16. voxblox/docs/pages/The-Voxblox-Node.rst at master / ethz-asl/voxblox - GitHub, 8월 2, 2025에 액세스, https://github.com/ethz-asl/voxblox/blob/master/docs/pages/The-Voxblox-Node.rst
17. The Voxblox Node - GitLab, 8월 2, 2025에 액세스, https://gitlab03.wpi.edu/gli7/mri_ros_public/-/blob/7960b08b5558710dacdc7d30902e275ba25cd154/mapping_vox/voxblox/docs/pages/The-Voxblox-Node.rst
18. The logics about the ESDF update / Issue #8 / nvidia-isaac/nvblox - GitHub, 8월 2, 2025에 액세스, https://github.com/nvidia-isaac/nvblox/issues/8
19. Marching cubes - Wikipedia, 8월 2, 2025에 액세스, https://en.wikipedia.org/wiki/Marching_cubes
20. Marching Cubes | Michael Walczyk, 8월 2, 2025에 액세스, https://michaelwalczyk.com/project-marching-cubes.html
21. voxblox/voxblox/include/voxblox/mesh/mesh_integrator.h at master - GitHub, 8월 2, 2025에 액세스, https://github.com/ethz-asl/voxblox/blob/master/voxblox/include/voxblox/mesh/mesh_integrator.h
22. Struct Mesh - voxblox documentation - Read the Docs, 8월 2, 2025에 액세스, https://voxblox.readthedocs.io/en/latest/api/structvoxblox_1_1Mesh.html
23. Send ESDF/TSDF information from voxl-mapper to ROS2 | ModalAI Forum, 8월 2, 2025에 액세스, https://forum.modalai.com/topic/3456/send-esdf-tsdf-information-from-voxl-mapper-to-ros2
24. PRBonn/voxblox_pybind: Python bindings for the Voxblox library - GitHub, 8월 2, 2025에 액세스, https://github.com/PRBonn/voxblox_pybind
25. voxblox/voxblox_ros/include/voxblox_ros/esdf_server.h at master / ethz-asl/voxblox - GitHub, 8월 2, 2025에 액세스, https://github.com/ethz-asl/voxblox/blob/master/voxblox_ros/include/voxblox_ros/esdf_server.h
26. nvblox: GPU-Accelerated Incremental Signed Distance Field Mapping - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2311.00626v2
27. mapping_vox/voxblox / main / Li, Guanrui (901036353) / mri_ros_public / GitLab, 8월 2, 2025에 액세스, https://gitlab03.wpi.edu/gli7/mri_ros_public/-/tree/main/mapping_vox/voxblox
28. The Voxblox Node - Read the Docs, 8월 2, 2025에 액세스, https://voxblox.readthedocs.io/en/latest/pages/The-Voxblox-Node.html
29. voxblox / GitHub Topics, 8월 2, 2025에 액세스, https://github.com/topics/voxblox
30. ethz-asl/mav_voxblox_planning: MAV planning tools using ... - GitHub, 8월 2, 2025에 액세스, https://github.com/ethz-asl/mav_voxblox_planning
31. Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning, 8월 2, 2025에 액세스, https://www.youtube.com/watch?v=ZGvnGFnTVR8
32. tu-darmstadt-ros-pkg/3d_coverage_path_planning: Global ... - GitHub, 8월 2, 2025에 액세스, https://github.com/tu-darmstadt-ros-pkg/3d_coverage_path_planning
33. The Truth Behind Drone-Based Volumetric Surveys: How Accurate Are They?, 8월 2, 2025에 액세스, https://thedronelifenj.com/drone-volumetric-survey-accuracy/
34. Running Voxblox - Read the Docs, 8월 2, 2025에 액세스, https://voxblox.readthedocs.io/en/latest/pages/Running-Voxblox.html
35. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - MDPI, 8월 2, 2025에 액세스, https://www.mdpi.com/1424-8220/22/3/1296
36. nvidia-isaac/nvblox: A GPU-accelerated TSDF and ESDF library for robots equipped with RGB-D cameras. - GitHub, 8월 2, 2025에 액세스, https://github.com/nvidia-isaac/nvblox
37. Voxfield: non-Projective Signed Distance Fields for Online Planning and 3D Reconstruction [IROS' 22] - GitHub, 8월 2, 2025에 액세스, https://github.com/VIS4ROB-lab/voxfield
38. Interactive Distance Field Mapping and Planning to Enable Human-Robot Collaboration, 8월 2, 2025에 액세스, https://arxiv.org/html/2403.09988v1
39. ethz-asl/voxgraph: Voxblox-based Pose graph optimization - GitHub, 8월 2, 2025에 액세스, https://github.com/ethz-asl/voxgraph
40. ethz-asl/cblox: Voxblox-based submapping - GitHub, 8월 2, 2025에 액세스, https://github.com/ethz-asl/cblox
41. ethz-asl/voxblox-plusplus: A volumetric object-level semantic mapping framework. - GitHub, 8월 2, 2025에 액세스, https://github.com/ethz-asl/voxblox-plusplus
42. LGSDF: Continual Global Learning of Signed Distance Fields Aided by Local Updating This work is supported by the National Natural Science Foundation of China under Grant 62233002, 92370203. (Corresponding Author: Yufeng Yue, yueyufeng@bit.edu.cn) - arXiv, 8월 2, 2025에 액세스, https://arxiv.org/html/2404.05187v1




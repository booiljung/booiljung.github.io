# VDB-EDT 알고리즘 및 SLAM 적용
[VDB](./index.md)


자율 로봇 기술의 발전은 로봇이 스스로 주변 환경을 인식하고 상호작용하는 능력을 기반으로 한다. 이러한 능력의 핵심에는 동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM) 기술이 자리 잡고 있다.1 SLAM은 로봇이 GPS와 같은 외부 인프라 없이 미지의 환경을 탐색하며, 자신의 위치를 실시간으로 추정함과 동시에 주변 환경의 지도를 생성하는 복합적인 기술이다.3 이 과정은 본질적으로 측위(Localization)와 지도 작성(Mapping)이라는 두 가지 상호 의존적인 과제를 동시에 해결해야 하는 '닭과 달걀' 문제와 같다.2 정확한 지도가 있어야 자신의 위치를 알 수 있고, 정확한 위치를 알아야 일관성 있는 지도를 만들 수 있기 때문이다.

SLAM 시스템의 성능과 효율성은 환경을 어떻게 표현하고 저장하는가, 즉 지도 표현(Map Representation) 방식에 의해 크게 좌우된다. 특히 LiDAR나 RGB-D 카메라로부터 얻어지는 포인트 클라우드 데이터를 사용하는 3D SLAM에서는 방대한 3차원 공간 정보를 효과적으로 관리하는 것이 핵심적인 도전 과제이다.5 전통적으로 사용되어 온 점유 격자 지도(Occupancy Grid Map)나 밀집 복셀 그리드(Dense Voxel Grid) 방식은 3차원 공간 전체를 일정한 해상도의 복셀로 나누어 표현한다. 이러한 방식은 구현이 직관적이지만, 실제 환경의 대부분이 비어있는 공간(free space)이라는 특성을 고려하지 못한다. 결과적으로, 대규모 환경을 매핑할 경우, 비어있는 공간을 저장하기 위해 막대한 양의 메모리를 소모하고, 전체 그리드를 처리하는 데 불필요하게 많은 연산 시간을 필요로 하는 심각한 비효율성을 초래한다.5

이러한 문제의 근원은 현실 세계의 3D 데이터가 가지는 본질적인 '희소성(sparsity)'을 기존의 밀집(dense) 데이터 구조가 제대로 활용하지 못하는 데 있다. 대부분의 정보는 물체의 표면(surface)에 집중되어 있으며, 그 외의 광대한 공간은 비어있다. 이 희소성을 효과적으로 활용하기 위한 대안으로 옥트리(Octree)와 같은 희소 자료 구조가 제안되었으나, 이 역시 맵의 규모가 커짐에 따라 데이터 접근 속도가 저하되는 한계를 가진다.

한편, 로봇이 생성된 지도를 활용하여 안전하고 효율적인 경로를 계획하기 위해서는 장애물의 위치 정보뿐만 아니라, 각 지점에서 가장 가까운 장애물까지의 거리에 대한 정보가 필수적이다. 이러한 정보를 담고 있는 거리 필드(Distance Field)는 충돌 회피, 경로 최적화 등 다양한 로봇 응용 분야의 기반이 된다.8 유클리드 거리 변환(Euclidean Distance Transform, EDT)은 점유 격자 지도와 같은 이진 맵으로부터 이러한 거리 필드를 생성하는 핵심 알고리즘이다. 하지만 EDT 역시 밀집 그리드 위에서 수행될 경우, 대규모 환경에서는 메모리와 연산 시간의 한계에 부딪힌다.

이러한 배경 속에서, 본 보고서는 VDB-EDT 알고리즘에 대한 심층적인 분석을 제공하고자 한다. VDB-EDT는 영화 산업의 시각 효과(VFX) 분야에서 대규모 희소 볼류메트릭 데이터를 효율적으로 처리하기 위해 개발된 VDB(Volumetric Dynamic B+Tree) 자료 구조를 로보틱스 분야에 도입하여, EDT 계산 과정의 메모리 및 연산 효율성을 획기적으로 개선한 알고리즘이다.10 VDB-EDT는 데이터의 희소성을 적극적으로 활용하여 대규모 3D 환경에서도 실시간에 가까운 성능으로 정확한 유클리드 거리 필드를 생성함으로써, 기존 SLAM 맵핑 및 경로 계획 방식의 한계를 극복할 수 있는 강력한 대안을 제시한다. 본 보고서는 VDB 자료 구조의 근본 원리부터 EDT 알고리즘의 수학적 기반, VDB-EDT의 구체적인 메커니즘, 그리고 다른 최신 맵핑 기법들과의 비교 분석을 통해 VDB-EDT의 기술적 의의와 잠재력을 종합적으로 고찰할 것이다.


VDB는 포인트 클라우드 SLAM과 같은 대규모 3D 데이터 처리 분야에서 전통적인 자료 구조의 한계를 극복하기 위해 등장한 혁신적인 해법이다. VDB의 핵심 철학은 3D 공간 데이터의 고유한 희소성을 최대한 활용하여 메모리 사용량과 데이터 접근 시간을 최적화하는 데 있다.


VDB는 'Volumetric, Dynamic grid that shares several characteristics with B+trees'의 약어로, 그 이름에서 알 수 있듯이 부피(Volumetric) 데이터를 다루며 동적(Dynamic)으로 변화하고 B+Tree의 특성을 일부 공유하는 그리드 자료 구조이다.11 VDB의 가장 근본적인 원리는 시변성 데이터(time-varying data)가 갖는 공간적 일관성(spatial coherency)을 이용하여, 실제 데이터 값(value)과 데이터의 공간적 분포를 나타내는 그리드 토폴로지(topology)를 분리하여 독립적으로, 그리고 압축적으로 인코딩하는 것이다.7

대부분의 3D 환경 데이터는 광대한 빈 공간 속에 일부 영역에만 정보가 밀집된 희소한 형태를 띤다. VDB는 이러한 희소성을 적극적으로 활용한다. 데이터가 없는 광대한 영역에 대해서는 메모리를 전혀 할당하지 않고, 데이터가 존재하는 '활성(active)' 영역에 대해서만 메모리를 동적으로 할당한다. 이를 통해 VDB는 사실상 무한한 크기의 3차원 인덱스 공간을 모델링하면서도, 실제 메모리 사용량은 데이터의 양에 비례하도록 유지한다.11 이는 고해상도의 희소 볼륨 데이터를 매우 효율적으로 저장하고 접근할 수 있게 해주는 핵심적인 특징이다.7


VDB의 구조적 혁신은 B+Tree의 개념을 3차원 공간에 맞게 변형하여 적용한 계층적 토폴로지에서 비롯된다. 전통적인 희소 자료 구조인 옥트리(Octree)가 공간을 재귀적으로 8분할하여 깊고 좁은 트리를 형성하는 것과 대조적으로, VDB는 B+Tree처럼 넓고 얕은 트리 구조를 채택한다.12

VDB 트리는 몇 가지 고유한 특징을 가진다. 첫째, 트리의 깊이가 고정되어 있다. 예를 들어, 일반적인 VDB 구현은 3단계의 계층(루트 노드, 내부 노드, 리프 노드)을 가진다.12 둘째, 트리의 가장 마지막 단계인 리프 노드(leaf node)는 단일 복셀이 아니라, 일정한 크기(예: 8x8x8 복셀)를 갖는 조밀한 데이터 블록, 즉 '브릭(brick)'을 가리킨다.12 이러한 구조는 트리의 전체적인 깊이를 매우 얕게 유지하면서 각 노드가 많은 자식 노드를 가질 수 있게 한다. 결과적으로 루트 노드에서 특정 데이터가 담긴 리프 노드까지 도달하기 위한 탐색 연산의 횟수가 크게 줄어든다.12

이러한 구조는 맵의 규모가 커질수록 트리의 깊이가 계속해서 깊어지고, 그에 따라 데이터 접근 시간이 로그 스케일로 증가하는($O(\log n)$) 옥트리의 근본적인 한계를 극복한다.15 VDB의 고정된 얕은 깊이는 데이터 접근 경로를 단축시켜, 대규모 환경에서도 일관되게 빠른 접근 성능을 보장하는 기반이 된다.


VDB는 데이터(값)와 토폴로지(구조)를 명확하게 분리하여 관리한다.11 트리 구조 자체는 공간상에서 데이터가 어디에 위치하는지에 대한 '주소록' 즉, 토폴로지 정보만을 담고 있다. 실제 복셀 값들(예: 점유 확률, 거리 값)은 '브릭'이라고 불리는 3차원 배열 형태의 조밀한 데이터 청크에 저장된다.12 트리의 리프 노드는 이 브릭들을 가리키는 포인터 역할을 한다.

이러한 분리 구조는 VDB의 놀라운 접근 효율성을 가능하게 하는 핵심 요소이다. 특정 좌표 $(i, j, k)$에 해당하는 복셀 값에 접근하고자 할 때, VDB는 먼저 트리 구조를 따라 해당 좌표가 속한 브릭을 찾는다. 이 과정은 고정된 깊이 덕분에 몇 단계의 포인터 역참조만으로 완료된다. 일단 브릭을 찾으면, 그 내부에서는 조밀한 배열의 인덱싱을 통해 매우 빠른 속도로 원하는 복셀 값에 접근할 수 있다.

여기에 더해, VDB는 특수한 인덱싱 기법과 캐시-코히어런트(cache-coherent)한 데이터 접근 패턴을 통해 성능을 더욱 최적화한다.7 결과적으로, VDB는 데이터의 삽입, 검색, 삭제와 같은 기본적인 연산에 대해 평균적으로 상수 시간($O(1)$)의 랜덤 액세스 속도를 달성한다.10 이처럼 빠르고 예측 가능한 접근 시간은 이웃 복셀에 대한 접근이 빈번하게 일어나는 SLAM의 지도 업데이트나 EDT 계산과 같은 연산에 매우 이상적인 특성이다.10


SLAM을 위한 3D 맵 표현에서 VDB와 옥트리는 모두 데이터의 희소성을 활용하는 대표적인 자료 구조이지만, 그 구조적 차이로 인해 성능과 특성에서 명확한 차이를 보인다.

옥트리는 공간을 재귀적으로 분할하는 직관적인 방식으로 희소성을 다룬다. 하지만 이 방식의 근본적인 한계는 데이터 접근 방식에 있다. 특정 복셀에 접근하기 위해서는 반드시 루트 노드부터 시작하여 자식 노드를 따라 내려가는 트리 탐색(tree traversal) 과정이 필요하다.15 맵의 규모가 커지고 복잡해져 트리의 깊이가 깊어지면, 이 탐색 과정에 소요되는 시간은 필연적으로 증가한다($O(\log n)$). 이는 대규모 환경에서 실시간 처리가 요구되는 SLAM 시스템에 상당한 오버헤드로 작용할 수 있다.15

반면, VDB는 B+Tree에서 영감을 받은 넓고 얕은 구조와 고정된 트리 깊이를 통해 이러한 접근 방식의 한계를 극복했다. VDB는 저장의 희소성을 넘어 '접근'의 효율성을 극대화한 것이다. 평균 $O(1)$의 데이터 접근 속도는 맵의 크기에 거의 영향을 받지 않으며, 이는 SLAM 시스템의 확장성(scalability)에 결정적인 이점을 제공한다. 동적 환경에서 빈번하게 발생하는 지도 업데이트(복셀 값의 삽입, 삭제, 수정) 역시 VDB에서는 더욱 효율적으로 처리될 수 있다. 아래 표는 두 자료 구조의 주요 특징을 비교하여 VDB의 상대적 우위를 명확히 보여준다.

**Table 1: VDB vs. Octree 비교 (Comparison of VDB vs. Octree)**

| 특징 (Feature)                    | VDB                                     | 옥트리 (Octree)                                              |
| --------------------------------- | --------------------------------------- | ------------------------------------------------------------ |
| 기본 구조 (Basic Structure)       | 넓고 얕은 B+Tree (Wide, shallow B+Tree) | 깊은 재귀적 트리 (Deep, recursive tree)                      |
| 트리 깊이 (Tree Depth)            | 고정 (Fixed)                            | 가변 (Variable)                                              |
| 데이터 접근 시간 (Access Time)    | 평균 $O(1)$ (Average $O(1)$)            | $O(\log n)$                                                  |
| 메모리 효율성 (Memory Efficiency) | 매우 높음 (Very High)                   | 높음 (High)                                                  |
| 동적 업데이트 (Dynamic Updates)   | 효율적으로 지원 (Efficiently supported) | 지원되나 노드 분할/병합 비용 발생 (Supported, but node splitting/merging costs) |
| 주요 응용 분야 (Primary Use Case) | VFX, 대규모 시뮬레이션, 로보틱스 맵핑   | 3D 모델링, 충돌 감지, 로보틱스 맵핑                          |

결론적으로, VDB는 옥트리가 가진 희소 표현의 장점을 계승하면서도, 접근 속도와 확장성 측면에서의 한계를 구조적으로 개선한 진일보한 자료 구조이다. 이러한 특성 덕분에 VDB는 대규모, 동적 환경을 다루는 현대 로보틱스, 특히 포인트 클라우드 SLAM 분야에서 매우 강력하고 유망한 기반 기술로 평가받고 있다.


로봇이 자율적으로 환경을 탐색하고 작업을 수행하기 위해서는 단순히 장애물의 유무를 아는 것을 넘어, 장애물로부터 얼마나 떨어져 있는지에 대한 정량적인 정보가 필수적이다. 유클리드 거리 변환(EDT)은 이러한 정보를 제공하는 핵심적인 알고리즘으로, 로봇의 안전하고 효율적인 경로 계획의 근간을 이룬다.


거리 필드(Distance Field) 또는 거리 맵(Distance Map)은 지도상의 모든 지점에 대해 가장 가까운 장애물까지의 거리를 값으로 저장하는 데이터 표현 방식이다.17 예를 들어, 어떤 지점의 거리 필드 값이 5.0이라면, 그 지점은 가장 가까운 장애물로부터 5.0 미터 떨어져 있음을 의미한다. 이는 로봇에게 주변 공간에 대한 풍부한 정보를 제공하여, 단순한 점유 격자 지도(Occupancy Grid Map)가 제공하는 이진적인 '점유/비점유' 정보를 훨씬 뛰어넘는 가치를 가진다.19

로봇 경로 계획에서 거리 필드는 다음과 같은 중요한 역할을 수행한다:

1. **충돌 회피 (Collision Avoidance):** 로봇의 현재 위치나 계획된 경로상의 각 지점에서 거리 필드 값을 확인함으로써, 장애물과의 거리를 즉각적으로 알 수 있다. 이를 통해 로봇의 크기를 고려한 안전 여유(safety margin)를 확보하고 충돌을 사전에 방지할 수 있다.8
2. **경로 최적화 (Path Optimization):** 거리 필드는 경로의 '품질'을 평가하는 비용 함수(cost function)로 활용될 수 있다. 예를 들어, 장애물에 가까운 경로는 높은 비용을, 멀리 떨어진 경로는 낮은 비용을 부여하여, 로봇이 가능한 한 장애물로부터 멀리 떨어진 안전하고 부드러운 경로를 찾도록 유도할 수 있다.9
3. **경로 평활화 (Path Smoothing):** 거리 필드는 각 지점에서 장애물 방향을 가리키는 그래디언트(gradient) 정보를 제공한다. 최적화 기반 경로 계획기(e.g., CHOMP)는 이 그래디언트 정보를 이용하여 초기 경로를 장애물로부터 밀어내는 방식으로 충돌을 회피하고 경로를 부드럽게 만들 수 있다.21

이처럼 거리 필드는 로봇이 주변 환경을 정량적으로 이해하고 지능적인 결정을 내리는 데 필수적인 정보를 제공한다. EDT는 이러한 거리 필드를 효율적으로 생성하기 위한 핵심적인 계산 도구이다.


모든 비-장애물 지점에서 모든 장애물 지점까지의 거리를 일일이 계산하여 최소값을 찾는 가장 단순한 방식은 엄청난 계산량을 요구하며($O(N^2)$, 여기서 $N$은 픽셀/복셀의 수), 실시간 적용이 불가능하다.[18] 이러한 비효율성을 해결하기 위해 다양한 고속 EDT 알고리즘이 개발되었으며, 그중에서도 Felzenswalb 등이 제안한 포물선 기반 알고리즘은 정확한 유클리드 거리 변환(Exact Euclidean Distance Transform, EEDT)을 선형 시간 복잡도($O(N)$)로 계산할 수 있어 널리 사용된다.22

이 알고리즘의 핵심 아이디어는 기하학적인 거리 계산 문제를 각 차원에 대한 1차원 스캔 문제로 분리하여 해결하는 것이다.

- **1D EDT:** 먼저 1차원 배열(예: 이미지의 한 행)에 대한 EDT를 계산한다. 이 과정은 각 장애물 지점 $p$를 꼭짓점으로 하는 포물선 함수 $y = (x-p)^2$들의 하한 포락선(lower envelope)을 찾는 것으로 귀결된다.23 어떤 지점 

  $x$에서의 제곱된 유클리드 거리($DT[x]^2$)는 바로 이 하한 포락선의 $y$값과 같다.

- **2D/3D EDT로의 확장:** 2D 및 3D EDT는 이러한 1D EDT를 순차적으로 적용하여 계산할 수 있다. 2D의 경우, 먼저 이미지의 모든 행(row)에 대해 1D EDT를 수행한다. 그 결과로 얻어진 각 픽셀의 값은 해당 행 내에서의 최소 제곱 거리가 된다. 다음으로, 이 결과 그리드의 모든 열(column)에 대해 다시 1D EDT를 수행한다. 이 두 번째 단계가 완료되면, 각 픽셀은 2D 공간 전체에서 가장 가까운 장애물까지의 최소 제곱 유클리드 거리를 값으로 갖게 된다.23 3D의 경우, z축 방향으로 1D EDT를 한 번 더 적용하면 된다.

이러한 분리 계산이 가능한 이유는 유클리드 거리의 제곱 공식이 각 차원 성분의 제곱의 합으로 표현되기 때문이다. 즉, $d((x_1, y_1), (x_2, y_2))^2 = (x_1-x_2)^2 + (y_1-y_2)^2$ 이므로, 전체 최소값을 찾는 과정은 각 차원에 대한 최소값 찾기 과정으로 분리될 수 있다.23 이처럼 복잡한 다차원 문제를 일련의 간단한 1차원 문제로 변환하는 것이 포물선 기반 EDT 알고리즘의 효율성의 핵심이다.


포물선 기반 1D EDT 알고리즘의 정교함은 포물선들의 하한 포락선을 선형 시간 복잡도로 효율적으로 계산하는 데 있다. 알고리즘은 1차원 배열을 한 방향으로 스캔하면서 점진적으로 하한 포락선을 구축한다.23

알고리즘의 동작 원리는 다음과 같다.

1. 배열을 왼쪽에서 오른쪽으로 스캔하면서, 각 장애물 위치 $i$에 대해 포물선 $g_i(x) = (x-i)^2$를 고려한다.

2. 현재까지 구축된 하한 포락선에 새로운 포물선 $g_i(x)$를 추가하는 것을 생각한다.

3. 새 포물선 $g_i(x)$와 현재 하한 포락선의 가장 오른쪽에 있는 포물선 $g_k(x)$의 교점 $s$를 계산한다. 두 포물선 $y = (x-p)^2 + f[p]$와 $y = (x-q)^2 + f[q]$의 교점 $s$는 다음과 같은 공식으로 주어진다. 여기서 $f[p]$는 각 지점의 초기 비용(장애물은 0, 비장애물은 무한대)이다.23
   $$
   s = \frac{(f[p] + p^2) - (f[q] + q^2)}{2p - 2q}
   $$

4. 만약 교점 $s$가 기존 포물선 $g_k(x)$가 지배하는 구간의 시작점보다 왼쪽에 있다면, 이는 $g_i(x)$가 $g_k(x)$를 완전히 덮어버린다는 의미이므로, $g_k(x)$를 하한 포락선에서 제거한다. 이 과정을 $g_i(x)$가 새로운 포물선을 덮지 않을 때까지 반복한다.

5. 이후 $g_i(x)$를 하한 포락선에 추가하고, 이 포물선이 지배하는 구간의 시작점을 교점 $s$로 설정한다.

이러한 과정을 통해 전체 배열을 한 번 스캔하는 것만으로 모든 포물선의 하한 포락선을 효율적으로 계산할 수 있다. 최종적으로, 배열의 각 위치 $x$에 대해, 해당 위치를 지배하는 하한 포락선의 포물선 함수 값을 계산하면 그것이 바로 제곱된 유클리드 거리 값이 된다.23 이 알고리즘의 효율성은 2D/3D 공간에서 모든 점과 모든 장애물 사이의 거리를 직접 비교하는 $O(N^2)$의 복잡도를, 각 차원을 독립적으로 스캔하는 $O(N)$의 문제로 변환시킨다는 점에서 비롯된다. 이 덕분에 EDT는 실시간 로봇 애플리케이션에서 실용적으로 사용될 수 있는 강력한 도구가 되었다.


VDB-EDT는 VDB 자료 구조의 효율성과 점진적 거리 변환 알고리즘을 결합하여, 대규모 동적 환경에서 빠르고 메모리 효율적인 유클리드 거리 필드 생성을 목표로 하는 혁신적인 알고리즘이다. 이 장에서는 VDB-EDT의 설계 목표, 핵심 작동 메커니즘, 그리고 성능을 심층적으로 분석한다.


VDB-EDT 알고리즘은 기존 EDT 방식이 가진 두 가지 근본적인 한계를 극복하기 위해 설계되었다.

첫 번째 목표는 **메모리 효율성의 극대화**이다. 전통적인 EDT 알고리즘은 거리 필드를 저장하기 위해 맵 전체 크기에 해당하는 거대한 3차원 배열을 사용한다. 이는 맵의 크기가 커질수록 메모리 요구량이 기하급수적으로 증가하여, 대규모 환경에서는 실용적으로 사용하기 어렵게 만든다. VDB-EDT는 이 문제를 해결하기 위해, 거리 필드를 표현하는 데 메모리 효율적인 VDB 자료 구조를 최초로 도입했다.10 VDB는 데이터가 존재하는 활성 복셀에 대해서만 메모리를 할당하므로, 환경이 희소할수록 메모리 절약 효과가 극대화된다. 실제 실험에서 VDB-EDT는 환경의 희소성에 따라 기존 배열 기반 방식 대비 메모리 사용량을 30%에서 최대 85%까지 절감하는 것으로 나타났다.10

두 번째 목표는 **연산 효율성의 극대화**이다. 기존의 많은 EDT 알고리즘은 맵 전체를 여러 번 스캔하는 방식으로 동작하여, 맵의 일부만 변경되어도 전체를 재계산해야 하는 비효율성을 가진다. VDB-EDT는 이러한 비효율성을 개선하기 위해, 변환 함수의 스케줄링 우선순위를 최적화하는 새로운 점진적(incremental) 업데이트 방식을 제안한다.10 이 방식은 맵의 변경 사항에만 집중하여 계산을 수행함으로써, 전체적인 실행 속도를 크게 향상시킨다.25


VDB-EDT의 핵심 연산 메커니즘은 우선순위 큐(priority queue)를 이용한 파동 전파(wave propagation) 방식이다.26 이 방식은 맵의 점유 상태가 변경되었을 때(즉, 장애물이 추가되거나 제거되었을 때), 변경된 지점으로부터 거리 값의 변화를 마치 파동처럼 주변으로 전파시켜 나간다. 이 과정은 두 종류의 파동, 즉 'Lowering Wave'와 'Raising Wave'로 구성된다.

- **Lowering Wave (거리 값 낮추기):** 새로운 장애물이 맵에 추가되면, 해당 장애물 복셀의 거리 값은 0이 된다. 이 복셀은 우선순위 0과 함께 우선순위 큐에 삽입된다. 알고리즘은 큐에서 우선순위가 가장 낮은(즉, 거리가 가장 가까운) 복셀을 꺼내어, 그 주변 이웃 복셀들의 거리 값을 갱신한다. 만약 이웃 복셀의 기존 거리 값이 새로 계산된 값보다 크다면, 해당 이웃은 더 가까운 장애물을 찾은 것이므로 거리 값을 낮추고(갱신하고) 우선순위 큐에 다시 삽입한다. 이 과정이 반복되면서, 새로운 장애물의 영향력이 파동처럼 퍼져나가 주변의 거리 필드를 점진적으로 갱신한다.
- **Raising Wave (거리 값 높이기):** 기존 장애물이 맵에서 제거되면, 이전에 그 장애물을 가장 가까운 장애물로 참조하던 주변 복셀들의 거리 값은 더 이상 유효하지 않게 된다. 이 복셀들은 'raising' 상태로 표시되고 우선순위 큐에 삽입된다. 알고리즘은 큐에서 이 복셀들을 꺼내어, 이제는 다른 장애물을 기준으로 새로운 (더 높은) 거리 값을 계산하여 갱신한다. 이 갱신된 정보 역시 주변으로 전파되어, 제거된 장애물의 영향력을 지우고 거리 필드를 올바르게 수정한다.

이러한 점진적 업데이트 방식은 VDB-EDT 효율성의 핵심이다. 맵 전체를 재계산하는 대신 변경된 부분과 그 영향권에만 계산을 집중함으로써, 특히 동적인 환경을 다루는 SLAM 시나리오에서 연산량을 획기적으로 줄일 수 있다.10

이러한 파동 전파 알고리즘이 실용적으로 동작할 수 있는 배경에는 VDB 자료 구조의 역할이 결정적이다. 알고리즘의 각 단계에서는 현재 복셀의 이웃 복셀들에 대한 빠른 접근이 필수적이다. VDB는 평균 $O(1)$의 빠른 랜덤 액세스 속도를 제공하여, 이러한 빈번한 이웃 탐색 작업을 매우 효율적으로 지원한다.10 만약 옥트리와 같이 접근 시간이 느린 자료 구조를 사용했다면, 파동 전파 방식의 장점이 크게 상쇄되었을 것이다. 결국, VDB-EDT의 뛰어난 성능은 점진적 계산이라는 '알고리즘적 혁신'과 VDB라는 '자료 구조적 혁신'의 강력한 시너지 효과에서 비롯된다고 할 수 있다.


VDB-EDT의 전체적인 알고리즘 흐름은 다음과 같이 재구성할 수 있다.

1. **초기화 (Initialization):**

   - 점유 격자 맵을 저장할 `VDB_Occupancy` 그리드와 거리 필드를 저장할 `VDB_Distance` 그리드를 생성한다.
   - 업데이트할 복셀을 관리하기 위한 우선순위 큐 `Q`를 초기화한다.

2. **업데이트 감지 (Update Detection):**

   - 새로운 센서 데이터(예: 포인트 클라우드)가 들어오면 `VDB_Occupancy` 그리드를 업데이트한다.
   - 이 과정에서 점유 상태가 변경된 복셀들의 목록(추가된 장애물 `add_list`, 제거된 장애물 `remove_list`)을 생성한다.

3. **큐 초기화 (Queue Initialization):**

   - `add_list`에 있는 모든 복셀 $v_{obs}$에 대해:
     - `VDB_Distance`에서 $v_{obs}$의 거리 값을 0으로 설정한다.
     - 우선순위 0과 함께 $v_{obs}$를 `Q`에 삽입한다.
   - `remove_list`에 있는 모든 복셀 $v_{free}$에 대해:
     - $v_{free}$를 가장 가까운 장애물로 참조했던 모든 이웃 복셀 $v_{neighbor}$를 찾아 `Q`에 삽입한다 (raising wave 시작). 이들의 거리 값은 무한대로 초기화될 수 있다.

4. **파동 전파 (Wave Propagation):**

   - Q가 비어있지 않은 동안 다음을 반복한다:

     a. Q에서 우선순위가 가장 높은(거리가 가장 작은) 복셀 $v_{curr}$를 꺼낸다.

     b. $v_{curr}$의 26-연결 이웃(3D의 경우) $v_{neigh}$ 각각에 대해 다음을 수행한다:

     c. $v_{neigh}$의 새로운 거리 값 $d_{new}$를 $v_{curr}$의 거리 값과 $v_{curr}$와 $v_{neigh}$ 사이의 유클리드 거리를 이용하여 계산한다. ($d_{new} = distance(v_{curr}) + EuclideanDist(v_{curr}, v_{neigh})$)

     d. 만약 $d_{new}$가 VDB_Distance에 저장된 $v_{neigh}$의 기존 거리 값 $d_{old}$보다 작다면 (lowering wave):

     i. $v_{neigh}$의 거리 값을 $d_{new}$로 갱신한다.

     ii. $d_{new}$를 우선순위로 하여 $v_{neigh}$를 Q에 삽입한다.

     e. (Raising wave의 경우, 유사한 논리를 적용하여 새로운 가장 가까운 장애물을 찾아 거리 값을 갱신한다.)

이러한 절차를 통해 VDB-EDT는 맵의 변화에 효율적으로 대응하며 항상 최신의 거리 필드를 유지할 수 있다.


VDB-EDT의 성능은 여러 공개 데이터셋과 자체 제작한 대규모 데이터셋을 통해 검증되었다. 특히, `cow-and-lady`와 같은 실내 데이터셋과, DARPA Subterranean Challenge에서 영감을 받은 대규모 지하 환경 데이터셋(`SubT`)이 벤치마킹에 사용되었다.10

실험 결과는 VDB-EDT의 우수성을 명확히 보여준다. 가장 빠른 공개 배열 기반 EDT 구현과 비교했을 때, VDB-EDT는 경쟁력 있는, 때로는 더 빠른 실행 속도를 보이면서도 메모리 사용량 측면에서 압도적인 이점을 제공했다.10 환경의 희소성에 따라 메모리 소비량은 30%에서 85%까지 감소했으며, 이는 VDB-EDT가 대규모 환경으로 확장될수록 그 가치가 더욱 커짐을 의미한다.10

**Table 2: VDB-EDT 성능 벤치마크 (VDB-EDT Performance Benchmark - 요약)**

| 메트릭 (Metric)              | VDB-EDT                    | 배열 기반 EDT (Array-based EDT) | 비고 (Notes)                                      |
| ---------------------------- | -------------------------- | ------------------------------- | ------------------------------------------------- |
| 메모리 사용량 (Memory Usage) | 30% ~ 85% 감소 (Reduction) | 기준 (Baseline)                 | 환경의 희소성에 따라 변동 (Depends on sparsity)   |
| 처리 시간 (Processing Time)  | 경쟁력 있음 (Competitive)  | 기준 (Baseline)                 | 대규모 환경에서 더 우월 (Superior in large-scale) |
| 확장성 (Scalability)         | 매우 높음 (Very High)      | 낮음 (Low)                      | 맵 크기에 따른 메모리 증가율이 낮음               |

이러한 벤치마크 결과는 VDB-EDT가 단순한 학술적 제안을 넘어, 실제 로봇 시스템, 특히 자원 제약이 있는 모바일 플랫폼이나 광범위한 영역을 탐사해야 하는 드론 등에 적용될 때 실질적인 이점을 제공하는 강력하고 실용적인 솔루션임을 입증한다.


SLAM 시스템의 성공은 효과적인 환경 표현에 달려있다. 로봇이 주변 세계를 어떻게 인식하고 모델링하는지는 후속 작업인 경로 계획, 상호작용, 의사결정의 품질을 결정한다. 최근 로보틱스 분야에서는 거리 필드 기반의 다양한 맵핑 기법들이 활발히 연구되고 있으며, VDB-EDT는 이들 중 독자적인 위치를 차지한다.


거리 필드는 크게 유클리드 부호 거리 필드(Euclidean Signed Distance Field, ESDF)와 절단 부호 거리 필드(Truncated Signed Distance Field, TSDF)로 나눌 수 있으며, 둘은 거리 측정 방식과 주된 용도에서 차이를 보인다.

- **ESDF (Euclidean Signed Distance Field):** ESDF는 각 복셀(또는 공간상의 한 점)에서 가장 가까운 장애물까지의 '실제 유클리드 거리'를 저장한다.19 '부호(Signed)'는 해당 지점이 장애물의 내부에 있는지(음수) 외부에 있는지(양수)를 나타낸다. ESDF는 경로 계획에 가장 이상적인 정보를 제공한다. 왜냐하면, 로봇은 ESDF 값을 통해 어떤 방향으로든 장애물까지의 최단 거리를 정확히 알 수 있기 때문이다. 이는 경로의 안전성을 평가하고, 충돌 회피를 위한 그래디언트를 계산하는 데 직접적으로 사용될 수 있다.9
- **TSDF (Truncated Signed Distance Field):** TSDF는 주로 깊이 카메라(RGB-D)와 같은 센서 데이터를 점진적으로 융합하여 3D 표면을 재구성(reconstruction)하는 데 사용된다.9 TSDF의 각 복셀 값은 센서 원점으로부터의 광선(ray) 방향을 따라 측정한 '투영 거리(projective distance)'를 나타낸다.16 이 거리는 실제 유클리드 거리가 아니며, 특히 센서 광선이 표면에 비스듬히 입사할 경우 실제 거리를 과대평가(overestimate)하는 경향이 있다.16 또한, '절단(Truncated)'이라는 이름처럼, 표면에서 일정 거리(truncation distance) 내에 있는 복셀들만 유효한 거리 값을 가지며, 그 외 영역은 일정한 값으로 처리된다. 이는 메모리 효율을 높이고 여러 측정값으로부터 표면을 부드럽게 융합하는 데 유리하지만, 경로 계획에 필요한 전역적인 거리 정보를 제공하지는 못한다.


TSDF의 표면 재구성 능력과 ESDF의 경로 계획 유용성을 모두 활용하기 위해, 이 두 표현을 결합하려는 여러 시도가 있었다. VDBFusion과 Voxblox는 그 대표적인 예이다.

- **VDBFusion:** 이 시스템은 OpenVDB 자료 구조를 활용하여 TSDF를 매우 효율적으로 통합하고 관리하는 데 초점을 맞춘다.30 VDBFusion의 강점은 유연성과 속도에 있다. 센서 종류에 구애받지 않고 포인트 클라우드 데이터를 입력받아, GPU 없이 단일 CPU 코어만으로도 LiDAR 데이터를 실시간으로 처리할 수 있는 놀라운 성능을 보여준다.31 하지만 VDBFusion의 주된 목표는 고품질의 TSDF 맵과 메시(mesh)를 생성하는 것이며, 경로 계획에 직접 필요한 ESDF를 생성하는 기능은 포함하지 않는다.16
- **Voxblox:** Voxblox는 TSDF와 ESDF를 연결하는 중요한 다리 역할을 하는 시스템이다.9 먼저 센서 데이터로부터 TSDF 맵을 점진적으로 구축한 다음, 이 TSDF 맵으로부터 ESDF를 점진적으로 생성하는 2단계 접근법을 취한다.9 이는 하나의 시스템에서 고품질의 시각화용 메시(TSDF 기반)와 경로 계획용 거리 필드(ESDF)를 모두 얻을 수 있다는 장점을 가진다. 그러나 이 방식은 근본적인 한계를 내포하고 있다. ESDF가 TSDF로부터 생성되기 때문에, TSDF가 가진 투영 거리의 부정확성이 ESDF에도 그대로 전파될 수 있다.29 즉, 생성된 ESDF는 '진정한' 유클리드 거리가 아닌 근사치일 수 있으며, 이는 정밀한 경로 계획에 오류를 유발할 수 있다.


VDB-EDT는 앞선 시스템들과는 다른 독자적인 접근법을 취한다. VDB-EDT는 TSDF라는 중간 단계를 완전히 배제하고, 가장 기본적인 환경 표현인 점유 격자 지도(Occupancy Grid Map)로부터 직접 정확한 유클리드 거리 필드(정확히는 부호가 없는 EDF)를 생성한다.10

이러한 접근법은 다음과 같은 중요한 의미를 가진다:

1. **정확성 확보:** TSDF의 투영 거리 오차 문제를 원천적으로 회피한다. VDB-EDT는 포물선 기반 알고리즘과 같은 정확한 EDT 계산법을 사용하므로, 생성된 거리 필드는 실제 유클리드 거리를 매우 정확하게 반영한다. 이는 경로 계획의 신뢰성과 안전성을 크게 향상시킨다.
2. **효율성 집중:** VDB-EDT는 VDB 자료 구조의 활용 목적을 '거리 필드' 자체의 효율적인 저장과 점진적 업데이트에 집중시킨다. VDBFusion이 VDB를 TSDF 융합에 사용하고, Voxblox가 TSDF와 ESDF를 별도로 관리하는 것과 달리, VDB-EDT는 점유 정보와 거리 정보를 모두 VDB 그리드 내에서 통합적으로 관리하며, 파동 전파 알고리즘을 통해 이 둘을 긴밀하게 연동시킨다.

결론적으로 VDB-EDT는 '정확한 거리 필드 생성'이라는 특정 목표를 달성하기 위해 VDB의 잠재력을 가장 직접적이고 효율적으로 활용한 시스템이라고 평가할 수 있다.


SLAM 맵핑 기술은 결정론적인 재구성에서 확률적이고 생성적인 모델링으로 진화하고 있다. 이러한 패러다임 전환의 중심에 가우시안 프로세스 거리 필드(Gaussian Process Distance Field, GPDF)가 있다.15 GPDF는 각 지점의 거리 값을 단일한 숫자로 결정하는 대신, 평균과 분산을 갖는 가우시안 분포로 모델링한다.

GPDF는 다음과 같은 강력한 장점을 가진다:

- **연속성(Continuity):** 이산적인 복셀 그리드를 넘어, 임의의 연속적인 좌표에 대한 거리 값을 추정할 수 있다.
- **불확실성 표현(Uncertainty Representation):** 센서 노이즈나 관측 부족으로 인해 거리 추정이 불확실한 영역을 정량적으로 표현할 수 있다. 이는 로봇이 위험을 회피하고 더 신중한 결정을 내리는 데 도움을 준다.
- **생성적 모델링(Generative Modeling):** 관측되지 않은 영역에 대해서도 주변 데이터를 기반으로 거리 값을 '추론(infer)'할 수 있다.

하지만 GPDF는 높은 계산 복잡도라는 치명적인 단점을 가지고 있어, 대규모 환경에 실시간으로 적용하기 어려웠다. 바로 이 지점에서 VDB가 다시 한번 핵심적인 역할을 수행한다. VDB-GPDF와 같은 최신 연구들은 GPDF의 복잡한 모델을 VDB의 희소하고 효율적인 자료 구조 위에 구현함으로써, GPDF의 풍부한 표현력과 VDB의 속도 및 확장성을 결합하려는 시도를 하고 있다.11 이는 VDB가 단순히 기존 알고리즘을 최적화하는 도구를 넘어, 차세대 지능형 맵핑 프레임워크를 가능하게 하는 핵심 기반 기술(enabling technology)로 진화하고 있음을 시사한다.

아래 표는 지금까지 논의된 다양한 환경 표현 기법들의 특징을 종합적으로 비교한다. 이 표를 통해 VDB-EDT가 전체 기술 스펙트럼에서 어떤 독자적인 위치와 강점을 가지는지 명확히 파악할 수 있다.

**Table 3: 환경 표현 기법 비교 (Comparison of Environment Representation Techniques)**

| 기법 (Technique)     | 기본 자료구조             | 거리 메트릭            | 장점                                     | 단점                                      | 대표 시스템             |
| -------------------- | ------------------------- | ---------------------- | ---------------------------------------- | ----------------------------------------- | ----------------------- |
| **TSDF**             | Voxel Hashing, Dense Grid | 투영 거리 (Projective) | 표면 재구성 우수, 센서 노이즈 융합       | 거리 부정확(과대평가), 경로 계획에 부적합 | KinectFusion, VDBFusion |
| **TSDF -> ESDF**     | Voxel Hashing, Octree     | 유클리드 (근사)        | 재구성과 계획 모두 활용 시도             | TSDF 오차 전파, 계산 복잡                 | Voxblox, Voxfield       |
| **Occupancy -> EDT** | Dense Array               | 유클리드 (정확)        | 정확한 유클리드 거리                     | 대규모에서 메모리/속도 문제               | - (기본 구현)           |
| **VDB-EDT**          | **VDB**                   | **유클리드 (정확)**    | **메모리/속도 효율 극대화, 정확한 거리** | 표면 재구성 정보 부족                     | **VDB-EDT**             |
| **VDB-GPDF**         | **VDB**                   | 유클리드 (확률적)      | 연속적, 불확실성 표현, 미관측 영역 추론  | 계산 복잡도 높음, 최신 연구               | VDB-GPDF, IDMP          |


VDB-EDT는 이론적 우수성을 넘어, 실제 로봇 시스템에 통합하여 활용할 수 있도록 구체적인 소프트웨어 패키지와 생태계를 갖추고 있다. 이 장에서는 VDB-EDT의 실제적인 적용 방법과 관련 기술 생태계를 살펴본다.


VDB-EDT는 로봇 소프트웨어 개발의 사실상 표준 프레임워크인 ROS(Robot Operating System)와의 긴밀한 통합을 염두에 두고 개발되었다. 공식적으로 공개된 VDB-EDT 패키지는 ROS Kinetic Kame 배포판을 기반으로 하며, ROS의 표준 빌드 시스템인 Catkin을 사용하여 빌드된다.24

ROS 패키지 형태로 제공된다는 것은 VDB-EDT가 독립적인 라이브러리를 넘어, ROS 생태계 내의 다른 수많은 모듈과 손쉽게 연동될 수 있음을 의미한다. 예를 들어, LiDAR 드라이버 노드로부터 포인트 클라우드 데이터를 입력받아 점유 격자 지도를 생성하고, VDB-EDT를 통해 거리 필드를 계산한 후, 이를 `move_base`와 같은 전역 경로 계획기나 `TEB Local Planner`와 같은 지역 경로 계획기에 전달하는 전체적인 SLAM 및 내비게이션 파이프라인을 구축할 수 있다.

특히, VDB-EDT 패키지 내부에는 카네기 멜론 대학(CMU)의 Air Lab과 스위스 취리히 연방 공과대학(ETHZ)의 ASL에서 개발한 작업을 기반으로 한 ROS 래퍼(Wrapper)가 포함되어 있다.24 이 래퍼는 VDB-EDT의 핵심 C++ 라이브러리 기능을 ROS 메시지(message), 서비스(service), 파라미터(parameter) 등과 연결해주는 역할을 하여, ROS 개발자가 VDB-EDT를 더욱 편리하게 사용할 수 있도록 돕는다.


VDB-EDT를 실제 시스템에서 사용하기 위한 절차는 공식 GitHub 저장소(`zhudelong/VDB-EDT`)에 상세히 안내되어 있다.24 주요 단계는 다음과 같다.

1. **의존성 설치 (Dependency Installation):** VDB-EDT는 핵심 라이브러리인 OpenVDB와 Eigen3에 의존한다. 또한 ROS 래퍼와 시각화를 위해 추가적인 라이브러리가 필요하다. Ubuntu 환경에서는 `apt-get` 명령어를 통해 다음과 같은 패키지를 설치해야 한다.24

   - OpenVDB 관련: `libglfw3-dev`, `libblosc-dev`, `libopenexr-dev`
   - 로깅 라이브러리: `liblog4cplus-dev`

2. **소스 코드 다운로드 및 빌드 (Source Code Download & Build):**

   - Git을 사용하여 `zhudelong/VDB-EDT` 저장소를 Catkin 작업 공간(workspace)의 `src` 폴더 내에 복제한다.

   - VDB-EDT는 일반적인 `catkin_make` 명령어 대신 `catkin build` 명령어를 사용하여 빌드해야 한다. 이는 패키지 내부의 의존성 구조 때문이다.24 작업 공간의 최상위 디렉토리에서 

     `catkin build`를 실행하면 컴파일이 진행된다.

3. **실행 및 테스트 (Execution & Testing):**

   - 빌드가 완료된 후, `source devel/setup.bash` 명령어로 작업 공간의 환경 설정을 로드한다.
   - `roslaunch` 명령어를 사용하여 VDB-EDT 노드를 실행할 수 있다. 예: `roslaunch vdb_edt VDB-EDT.launch`.24
   - 제공되는 예제 데이터셋(`cow-and-lady.bag`, `SubT.bag`)을 `rosbag play` 명령어로 재생하여 VDB-EDT가 실시간으로 맵과 거리 필드를 생성하는 과정을 확인할 수 있다. `cow-and-lady` 데이터셋의 경우, 동기화 문제로 인해 `rosbag play data.bag -r 0.1`과 같이 재생 속도를 늦추는 것이 권장된다.24


VDB-EDT의 성공은 강력한 기반 라이브러리인 OpenVDB의 존재 덕분이다. OpenVDB는 로보틱스 분야에서 점차 희소 3D 데이터를 처리하는 표준 도구로 자리매김하고 있으며, VDB-EDT 외에도 다양한 관련 프로젝트들이 OpenVDB 생태계를 형성하고 있다.

OpenVDB 자체는 풍부하고 강력한 API를 제공한다. 예를 들어, `openvdb::math::Transform` 객체는 복셀의 이산적인 인덱스 좌표($(i, j, k)$)와 실제 세계의 물리적 좌표($(x, y, z)$) 사이의 변환을 관리하며, 단순한 스케일링/이동뿐만 아니라 프러스텀(frustum)과 같은 비선형 변환도 지원한다.33

`openvdb::FloatGrid::Accessor`와 같은 접근자(accessor)는 특정 복셀 값에 대한 빠르고 안전한 읽기/쓰기 작업을 가능하게 한다.34

ROS 생태계 내에서도 OpenVDB를 활용하는 다른 패키지들을 찾아볼 수 있다.

- **vdb_mapping_ros:** OpenVDB를 기반으로 점유 격자 지도를 생성하고, 네트워크를 통해 맵 데이터를 효율적으로 전송하는 기능을 제공하는 ROS 패키지이다. 이는 원격 제어나 다중 로봇 협력 시나리오에서 유용하게 사용될 수 있다.35
- **beluga_vdb:** 몬테카를로 측위(MCL) 라이브러리인 `beluga`를 OpenVDB와 통합하여, VDB 형식의 3D 맵 상에서 효율적인 3D 측위를 수행하는 라이브러리이다.36

이러한 사례들은 OpenVDB가 VDB-EDT라는 특정 알고리즘을 넘어, 로보틱스 분야의 다양한 문제(맵핑, 측위, 데이터 전송 등)를 해결하는 데 사용될 수 있는 범용적이고 강력한 플랫폼임을 보여준다. VDB-EDT를 이해하고 활용하는 것은 이러한 더 넓은 기술 생태계에 접근하는 첫걸음이 될 수 있다.


본 보고서는 포인트 클라우드 SLAM을 위한 효율적인 유클리드 거리 필드 생성 기법인 VDB-EDT에 대한 심층적인 분석을 제공했다. VDB 자료 구조의 근본 원리부터 EDT 알고리즘의 수학적 기반, VDB-EDT의 핵심 메커니즘, 그리고 다양한 맵핑 기법과의 비교 분석을 통해 VDB-EDT의 기술적 의의와 가치를 다각도로 조명했다.

**VDB-EDT의 핵심 기여**는 세 가지로 요약할 수 있다.

1. **메모리 효율성의 혁신:** VDB-EDT는 영화 산업에서 검증된 고성능 희소 자료 구조인 VDB를 로보틱스 거리 필드 표현에 최초로 도입했다. 이를 통해 대규모 3D 환경에서 기존 배열 기반 방식이 겪었던 메모리 폭증 문제를 근본적으로 해결했으며, 환경의 희소성에 따라 메모리 사용량을 최대 85%까지 절감하여 SLAM 시스템의 확장성을 획기적으로 향상시켰다.10
2. **연산 효율성의 최적화:** 우선순위 큐를 이용한 파동 전파(raising/lowering waves) 방식의 점진적 업데이트 알고리즘을 구현했다. 이는 맵의 변화가 발생했을 때 전체를 재계산하는 대신, 변경된 영역에만 계산을 집중함으로써 특히 동적인 환경에서 연산 속도를 크게 개선했다.10
3. **경로 계획을 위한 정확성 확보:** TSDF와 같은 중간 표현을 거치지 않고 점유 격자 지도에서 직접 정확한 유클리드 거리를 계산함으로써, TSDF의 투영 거리 오차 문제를 원천적으로 배제했다.16 이는 로봇 경로 계획에 더욱 신뢰성 있고 안전한 거리 정보를 제공한다는 점에서 실용적인 가치가 매우 크다.

**기술적 의의** 측면에서, VDB-EDT는 SLAM 맵핑 분야의 오랜 난제였던 '대규모 환경에서의 효율성' 문제를 해결하는 강력하고 실용적인 솔루션을 제시했다. 또한, VDB라는 범용 고성능 라이브러리가 특정 산업 분야를 넘어 로보틱스라는 새로운 영역에서 어떻게 성공적으로 적용되고 가치를 창출할 수 있는지를 보여주는 중요한 선례를 남겼다. 이는 향후 더 많은 학제 간 기술 융합의 가능성을 시사한다.

물론 VDB-EDT에도 **한계 및 향후 연구 방향**은 존재한다.

- **한계점:** VDB-EDT는 점유 격자 지도를 기반으로 하므로, TSDF 기반 방식에 비해 고품질의 연속적인 표면 메시(mesh)를 재구성하는 데는 상대적으로 불리할 수 있다. 또한, 현재 공개된 공식 패키지는 ROS Kinetic을 기반으로 하고 있어, 최신 ROS2 배포판과의 호환성 확보 및 성능 최적화가 필요한 과제로 남아있다.
- **향후 연구 방향:** VDB-EDT의 미래는 다른 첨단 기술과의 융합에 있다. 예를 들어, VDB-EDT가 제공하는 정확한 거리 정보와 VDB-GPDF와 같은 확률적/연속적 표현이 가지는 불확실성 모델링 능력을 결합하는 하이브리드 맵핑 프레임워크를 연구할 수 있다.15 이는 로봇이 더욱 안전하고 지능적인 결정을 내리는 데 기여할 것이다. 또한, VDB-EDT를 동적 장애물 추적 및 미래 경로 예측 기술과 연계하여, 복잡하고 예측 불가능한 환경에서도 강건하게 동작하는 차세대 동적 경로 계획 시스템을 구축하는 방향도 유망한 연구 주제가 될 것이다.

결론적으로, VDB-EDT는 포인트 클라우드 SLAM의 지도 표현 및 경로 계획 기반 기술에 있어 중요한 이정표를 제시한 알고리즘이다. 이는 현재의 기술적 문제를 해결했을 뿐만 아니라, 미래의 지능형 로봇 시스템이 나아갈 방향에 대한 깊은 통찰을 제공한다.


1. What Is SLAM (Simultaneous Localization and Mapping)? - MATLAB & Simulink, accessed August 6, 2025, https://www.mathworks.com/discovery/slam.html
2. The Complete Guide to SLAM: Origin, Applications, and Comparison of 5 systems - dtLabs, accessed August 6, 2025, https://dt-labs.ai/blog/the-complete-guide-to-slam/
3. Fast Reconstruction of 3D Point Cloud Model Using Visual SLAM on Embedded UAV Development Platform - MDPI, accessed August 6, 2025, https://www.mdpi.com/2072-4292/12/20/3308
4. A Review of Simultaneous Localization and Mapping Algorithms Based on Lidar - MDPI, accessed August 6, 2025, https://www.mdpi.com/2032-6653/16/2/56
5. The VDB data structure [15] compared to octrees [52]. A conventional... - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/figure/The-VDB-data-structure-15-compared-to-octrees-52-A-conventional-octree-subdivides_fig2_358523933
6. A Novel Point Cloud Adaptive Filtering Algorithm for LiDAR SLAM in Forest Environments Based on Guidance Information - MDPI, accessed August 6, 2025, https://www.mdpi.com/2072-4292/16/15/2714
7. VDB: High-resolution sparse volumes with dynamic topology - Ken Museth, accessed August 6, 2025, https://www.museth.org/Ken/Publications_files/Museth_TOG13.pdf
8. Differentiable Composite Neural Signed Distance Fields for Robot Navigation in Dynamic Indoor Environments - arXiv, accessed August 6, 2025, https://arxiv.org/html/2502.02664v1
9. Voxblox: Incremental 3D Euclidean Signed ... - Helen Oleynikova, accessed August 6, 2025, https://helenol.github.io/publications/iros_2017_voxblox.pdf
10. VDB-EDT: An Efficient Euclidean Distance Transform Algorithm Based on VDB Data Structure | Request PDF - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/351475693_VDB-EDT_An_Efficient_Euclidean_Distance_Transform_Algorithm_Based_on_VDB_Data_Structure
11. VDB: High-Resolution Sparse Volumes with Dynamic Topology - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/259288658_VDB_High-Resolution_Sparse_Volumes_with_Dynamic_Topology
12. Hierarchical, Dense and Dynamic 3D Reconstruction ... - Frontiers, accessed August 6, 2025, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2020.600387/full
13. Volumetric Representation and Sphere Packing of Indoor Space for Three-Dimensional Room Segmentation - MDPI, accessed August 6, 2025, https://www.mdpi.com/2220-9964/10/11/739
14. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/358523933_VDBFusion_Flexible_and_Efficient_TSDF_Integration_of_Range_Sensor_Data
15. (PDF) VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/382271448_VDB-GPDF_Online_Gaussian_Process_Distance_Field_with_VDB_Structure
16. VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure - arXiv, accessed August 6, 2025, https://arxiv.org/html/2407.09649v1
17. Morphology - Distance Transform, accessed August 6, 2025, https://homepages.inf.ed.ac.uk/rbf/HIPR2/distance.htm
18. Euclidean Distance Transform (EDT) - An Introduction | by Kareim Tarek - Medium, accessed August 6, 2025, https://medium.com/@kareimtarek1972/euclidean-distance-transform-edt-introduction-5d7d7c144aa
19. Signed Distance Field - Dibyendu Biswas - Medium, accessed August 6, 2025, https://dibyendu-biswas.medium.com/signed-distance-field-14d829d8103e
20. Signed Distance Fields: A Natural Representation for Both Mapping and Planning - Research Collection, accessed August 6, 2025, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/128029/eth-50477-01.pdf
21. Signed Distance Fields: A Natural Representation for Both Mapping and Planning - Helen Oleynikova, accessed August 6, 2025, https://helenol.github.io/publications/rss_2016_workshop.pdf
22. Distance transform - Wikipedia, accessed August 6, 2025, https://en.wikipedia.org/wiki/Distance_transform
23. The Fast Euclidean Distance Transform - Hello, Robot! Introduction ..., accessed August 6, 2025, https://hellorob.org/files/lectures/fast_euclidean_dt.pdf
24. VDB-EDT: An Efficient Euclidean Distance Transform ... - GitHub, accessed August 6, 2025, https://github.com/zhudelong/VDB-EDT
25. [2105.04419] VDB-EDT: An Efficient Euclidean Distance Transform Algorithm Based on VDB Data Structure - arXiv, accessed August 6, 2025, https://arxiv.org/abs/2105.04419
26. Volumetric Representation and Sphere Packing ... - Semantic Scholar, accessed August 6, 2025, https://pdfs.semanticscholar.org/78fe/425fdcdb6225e054f92fadbacf90f4c9f71c.pdf
27. VDBblox: Accurate and Efficient Distance Fields for Path Planning and Mesh Reconstruction, accessed August 6, 2025, https://www.semanticscholar.org/paper/e9523886dc7b0f62929b3ac90143e15227a9838e
28. Large Scale 2D Laser SLAM using Truncated Signed Distance Functions - TU Darmstadt, accessed August 6, 2025, https://www.sim.informatik.tu-darmstadt.de/publ/download/2019_daun_ssrr.pdf
29. Voxfield: Non-Projective Signed Distance Fields for Online Planning and 3D Reconstruction, accessed August 6, 2025, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/pan2022iros.pdf
30. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor ..., accessed August 6, 2025, https://www.mdpi.com/1424-8220/22/3/1296
31. VDBFusion: Flexible and Efficient TSDF Integration of Range Sensor Data - PubMed, accessed August 6, 2025, https://pubmed.ncbi.nlm.nih.gov/35162040/
32. VDB-GPDF: Online Gaussian Process Distance Field with VDB Structure - arXiv, accessed August 6, 2025, https://arxiv.org/html/2407.09649v2
33. Transforms and Maps - OpenVDB, accessed August 6, 2025, https://www.openvdb.org/documentation/doxygen/transformsAndMaps.html
34. OpenVDB Cookbook, accessed August 6, 2025, https://www.openvdb.org/documentation/doxygen/codeExamples.html
35. fzi-forschungszentrum-informatik/vdb_mapping_ros: ROS1 Wrapper für vdb_mapping - GitHub, accessed August 6, 2025, https://github.com/fzi-forschungszentrum-informatik/vdb_mapping_ros
36. ROS Package: beluga_vdb, accessed August 6, 2025, https://index.ros.org/p/beluga_vdb/


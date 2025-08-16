# 희소 복셀 방향성 비순환 그래프(SVDAG)

## 서론: 기하학적 복잡성과 메모리의 장벽

현대 컴퓨터 그래픽스는 가상현실, 영화적 수준의 비주얼을 자랑하는 비디오 게임, 정밀한 과학 시각화 등 다양한 분야에서 전례 없는 수준의 시각적 사실성을 추구하고 있다. 이러한 사실성의 핵심에는 3차원 객체와 환경을 구성하는 기하학적 디테일의 폭발적인 증가가 자리한다.1 그러나 이처럼 복잡성이 증가함에 따라, 3D 씬(scene)을 표현하고 렌더링하는 데 필요한 데이터의 양 또한 기하급수적으로 늘어나면서 전통적인 데이터 표현 방식은 근본적인 한계에 부딪히게 되었다. 이는 한정된 GPU 메모리와 대역폭이라는 물리적 제약 속에서 '어떻게 더 많은 것을, 더 효율적으로 표현할 것인가'라는 오랜 질문을 다시금 수면 위로 떠오르게 했다.

이 문제에 대한 전통적인 해답 중 하나는 폴리곤 메시(Polygon Mesh)였다. 폴리곤 메시는 정점(vertex), 간선(edge), 면(face)의 집합으로 객체의 표면을 정의하는 방식으로, 특히 평평하거나 완만한 곡면을 가진 인공물을 표현하는 데 매우 효율적이다.3 현대의 그래픽 처리 장치(GPU)는 수백만 개의 삼각형으로 이루어진 메시를 실시간으로 래스터화(rasterize)하도록 고도로 최적화되어 있다.1 하지만 유기적인 형태, 파괴 가능한 환경, 또는 머리카락이나 구름과 같은 볼류메트릭(volumetric) 데이터를 극도로 상세하게 표현하고자 할 때, 필요한 삼각형의 수는 천문학적으로 증가한다. 이는 막대한 메모리 소비로 이어질 뿐만 아니라, 각기 다른 거리에서 객체를 효율적으로 렌더링하기 위한 상세 수준(Level of Detail, LOD) 관리 체계를 복잡하게 만드는 원인이 된다.1

또 다른 접근 방식은 조밀 복셀 그리드(Dense Voxel Grid)이다. 복셀(voxel)은 2D의 픽셀(pixel)에 깊이(volume) 개념이 더해진 3차원 화소를 의미하며, 3차원 공간을 균일한 격자로 나누어 각 셀의 속성을 저장하는 방식이다.4 이 구조는 데이터 접근이 빠르고 직관적이라는 장점이 있지만, 치명적인 확장성 문제를 안고 있다. 해상도가 `$N$`배 증가하면 필요한 메모리 요구량은 `$N$`의 세제곱, 즉 `$O(N^3)$`으로 폭증하기 때문에 고해상도 표현에는 지극히 비실용적이다.4

이러한 배경 속에서 '희소성(sparsity)'을 활용하는 데이터 구조가 중요한 대안으로 부상했다. 대부분의 3D 씬은 객체가 차지하는 공간보다 비어있는 공간이 압도적으로 많다는 사실에 착안한 것이다.6 희소 복셀 옥트리(Sparse Voxel Octree, SVO)는 이러한 '공간적 희소성'을 활용하여 비어있는 공간을 데이터 구조에서 제외함으로써 메모리 효율을 획기적으로 개선한 대표적인 예이다.4 그러나 SVO조차도 초고해상도 환경에서는 여전히 상당한 메모리를 요구하는 한계를 보였다. 바로 이 지점에서 희소 복셀 방향성 비순환 그래프(Sparse Voxel Directed Acyclic Graph, SVDAG)가 등장하며 패러다임의 전환을 이끌었다. SVDAG는 단순히 비어있는 공간을 제거하는 것을 넘어, 데이터 내에 존재하는 '구조적 중복성(structural redundancy)'을 공략하는 혁신적인 접근법을 제시했다.7 3D 씬에 반복적으로 나타나는 기하학적 패턴을 단 한 번만 저장하고 공유함으로써, SVDAG는 렌더링 가능한 데이터의 복잡도 상한선을 한 차원 끌어올리는 중요한 개념적 돌파구를 마련했다. 이는 3D 그래픽 기술의 발전사가 '표현의 정밀도'와 '자원의 한계' 사이의 끊임없는 투쟁의 역사이며, SVDAG는 이 투쟁에서 압축의 패러다임을 '빈 공간 제거'에서 '구조적 중복성 제거'로 진화시킨 핵심적인 이정표라 할 수 있다.

## 1.  기반 기술 - 희소 복셀 옥트리(SVO)의 원리와 구조

SVDAG의 혁신을 온전히 이해하기 위해서는 그 기술적 뿌리가 되는 희소 복셀 옥트리(SVO)에 대한 깊이 있는 고찰이 선행되어야 한다. SVDAG는 SVO의 한계를 극복하기 위해 탄생했지만, 동시에 SVO가 이룩한 효율적인 공간 분할 및 순회 메커니즘을 기반으로 하고 있기 때문이다.

### 1.1 공간 분할과 옥트리

옥트리(Octree)는 3차원 공간을 계층적으로 분할하고 인덱싱하는 데 사용되는 트리 자료구조이다. 그 원리는 간단하다. 하나의 정육면체 공간(루트 노드)을 각 축의 중심을 기준으로 잘라 8개의 동일한 크기를 가진 하위 정육면체 공간(자식 노드)으로 나눈다. 이 과정을 각 하위 공간에 대해 재귀적으로 반복하여 공간을 더욱 세밀하게 분할한다.6 이러한 계층적 구조는 특정 위치를 포함하는 영역을 빠르게 찾거나, 공간 내 객체 간의 충돌을 검사하는 등의 공간 질의(spatial query)를 매우 효율적으로 처리할 수 있게 해준다.

### 1.2 SVO(Sparse Voxel Octree)의 개념

SVO는 이러한 옥트리의 기본 원리를 복셀 데이터 표현에 적용하되, 한 가지 중요한 최적화를 더한 것이다. 바로 '희소성'을 활용하는 것이다. 3D 씬의 복셀화된 데이터에서 대부분의 복셀은 비어있다. SVO는 표면 지오메트리를 포함하지 않는, 즉 완전히 '비어있는(empty)' 노드를 발견하면 더 이상 그 자식 노드들을 생성하거나 저장하지 않고 탐색을 중단한다.4

이 간단한 아이디어는 메모리 사용량에 극적인 변화를 가져온다. 조밀 복셀 그리드의 메모리 사용량이 해상도(`$N$`)에 따라 `$O(N^3)$`으로 증가하는 반면, SVO의 메모리 사용량은 전체 볼륨이 아닌 씬을 구성하는 표면적에 비례하는 경향을 보인다. 이는 대략 `$O(N^2)$`에 가까운 복잡도로, 고해상도에서도 훨씬 더 효율적으로 데이터를 관리할 수 있게 해준다.4

### 1.3 Laine & Karras의 'Efficient Sparse Voxel Octrees' 심층 분석

SVDAG의 직접적인 기술적 선조로 평가받는 Samuli Laine과 Tero Karras의 2010년 논문 "Efficient Sparse Voxel Octrees"는 SVO를 이론적 개념에서 벗어나 현대 GPU 아키텍처에 최적화된 실용적인 실시간 렌더링 기술로 끌어올린 중요한 연구다.10 이들의 목표는 명확했다: GPU 메모리에 완전히 상주하면서 실시간 레이 트레이싱이 가능한, 극도로 압축적이면서도 빠른 SVO 데이터 구조를 설계하는 것이었다.12 이 연구가 제시한 SVO 구조와 순회 방식은 이후 SVDAG 연구에 그대로 계승되어 핵심적인 기반이 되었다.

이들의 SVO 설계가 GPU의 병렬 처리 능력과 메모리 계층 구조를 얼마나 깊이 고려했는지는 그 핵심 요소인 **64비트 자식 디스크립터(Child Descriptor)**에서 명확히 드러난다. 이는 SVO의 트리 구조에서 리프 노드가 아닌 모든 내부 노드(non-leaf node)가 자신의 8개 자식 노드에 대한 정보를 단 64비트의 압축된 데이터로 저장하는 방식이다.12 이 디스크립터는 다음과 같은 주요 필드로 구성된다.

- **`valid_mask` (8-bit):** 8개의 자식 슬롯(옥턴트) 각각에 실제로 지오메트리가 존재하여 유효한 노드가 할당되었는지를 나타내는 비트마스크이다. 예를 들어, 첫 번째 비트가 1이면 첫 번째 자식 슬롯이 유효함을 의미한다.
- **`leaf_mask` (8-bit):** `valid_mask`에서 유효하다고 표시된 자식 노드에 대해, 해당 자식이 최종 단계인 리프 노드(leaf node)인지, 아니면 더 깊은 하위 트리를 갖는 내부 노드(internal node)인지를 구별하는 비트마스크이다.
- **`child_pointer` (32-bit or other):** 유효한 자식 노드들이 시작되는 메모리 위치를 가리키는 포인터 또는 오프셋이다. Laine & Karras의 설계에서 중요한 점은, 한 부모의 유효한 자식 노드들은 메모리 상에 반드시 연속적으로 배치된다는 것이다. 따라서 이 `child_pointer`와 `valid_mask`의 특정 비트까지 1의 개수(popcount 연산)를 알면, 특정 자식 노드의 정확한 메모리 주소를 계산할 수 있다.

이러한 구조는 다음과 같은 비트 필드 레이아웃으로 개념적으로 표현될 수 있다. 실제 논문에서는 64비트 중 32비트는 자식 정보에, 나머지 32비트는 표면의 미세한 디테일을 표현하기 위한 컨투어(contour) 정보에 할당했다.12

```
[ 63... 32 ][ 31... 24 ][ 23... 16 ][ 15... 0 ]
+---------------------+-------------+-------------+-------------+

| child_pointer/other | (unused) | leaf_mask | valid_mask | (예시적 구성)
+---------------------+-------------+-------------+-------------+
```

이 64비트 디스크립터는 GPU 최적화의 정수라 할 수 있다. GPU의 수많은 스레드가 각각 레이 트레이싱을 수행할 때, 단 한 번의 64비트 메모리 읽기 연산만으로 8개 자식의 존재 여부, 타입, 그리고 시작 위치까지 모든 정보를 획득할 수 있다. 이는 여러 번의 포인터 역참조(pointer dereferencing)를 피하게 해 메모리 접근 횟수를 최소화하고, 데이터 지역성(data locality)을 높여 캐시 효율을 극대화한다. 또한, 스레드 내에서 조건부 분기(branching)를 줄여 SIMT(Single Instruction, Multiple Threads) 실행 모델에 더 잘 부합하게 만든다.

이러한 하드웨어 친화적 노드 구조를 바탕으로, Laine과 Karras는 스택을 사용하지 않는(stackless) 효율적인 레이 캐스팅 알고리즘을 구현했다.10 광선이 현재 노드(복셀)와 교차하면, 알고리즘은 광선이 8개의 자식 옥턴트와 교차하는 순서를 빠르게 계산한다. 그 후 계산된 순서에 따라 자식 노드들을 방문하며 탐색을 이어간다. 이 방식은 재귀 호출로 인한 스택 오버플로우의 위험이 없고, GPU 스레드에서 독립적으로 실행되기에 매우 적합하다.

결론적으로, Laine & Karras의 SVO는 단순한 희소 옥트리를 넘어, GPU라는 특정 하드웨어의 특성을 극단적으로 고려한 고도의 엔지니어링 결과물이다. 그리고 SVDAG는 바로 이 고도로 최적화된 SVO의 노드 구조와 순회 메커니즘을 거의 그대로 계승했다. SVDAG의 혁신은 새로운 노드 구조를 발명한 것이 아니라, 기존의 효율적인 SVO 노드 구조 위에서 '포인터가 가리키는 대상'의 개념을 확장하는 그래프 이론을 접목시킨 데에 있다. 즉, SVDAG의 `child_pointer`는 유일무이한 자식을 가리키는 대신, 다른 노드들도 함께 참조할 수 있는 '공유 서브트리'를 가리킬 수 있게 된 것이다.8 이처럼 SVDAG의 혁신은 SVO라는 거인의 어깨 위에서 이루어졌다.

## 2.  SVDAG의 탄생: 중복성을 통한 압축의 극대화

SVO가 3D 데이터 압축 분야에서 '공간적 희소성'이라는 중요한 개념을 정립했지만, 초고해상도 씬에서는 여전히 메모리 한계에 부딪혔다. 이 한계를 돌파하기 위해 등장한 SVDAG는 압축의 패러다임을 한 단계 더 진화시켰다. SVO가 '비어있는 공간'을 제거하는 데 집중했다면, SVDAG는 씬 내에 반복적으로 나타나는 '동일한 기하학적 패턴'을 제거하는 데 초점을 맞추었다.5 이 아이디어는 자료구조를 트리(Tree)에서 방향성 비순환 그래프(Directed Acyclic Graph, DAG)로 일반화함으로써 실현되었다.4

### 2.1 트리에서 DAG로의 패러다임 전환

자료구조의 관점에서 트리와 DAG의 근본적인 차이는 노드 간의 관계에 있다. 트리 구조에서 모든 자식 노드는 단 하나의 부모 노드만을 가질 수 있다. 이는 엄격한 1:N 관계를 형성하며, 데이터의 중복을 허용하지 않는 계층 구조를 만든다. 반면, DAG에서는 한 노드가 여러 개의 부모 노드로부터 참조될 수 있다.6 즉, 여러 부모가 동일한 자식 노드를 '공유'하는 것이 가능하다. 단, 이름에서 알 수 있듯이 그래프의 간선은 방향성을 가지며, 자기 자신으로 돌아오는 순환(cycle)은 존재하지 않아야 한다.15

SVDAG는 SVO의 옥트리 구조에 이러한 DAG의 특성을 적용한 것이다. SVO에서는 동일한 형태의 기둥이 건물 여러 곳에 존재하더라도, 각각의 기둥은 트리 상에서 완전히 별개의 노드와 서브트리로 표현된다. 하지만 SVDAG에서는 이 모든 동일한 기둥들을 단 하나의 '대표 서브트리'로 저장하고, 각 기둥이 위치해야 할 곳의 부모 노드들이 모두 이 대표 서브트리의 루트 노드를 가리키도록 포인터를 공유한다.

### 2.2 핵심 원리: 동일 서브트리 병합 (Common Subtree Merging)

SVDAG의 핵심 압축 원리는 바로 이 '동일 서브트리 병합(Common Subtree Merging)'이다.16 3D 씬, 특히 건축물, 기계 부품, 절차적으로 생성된 지형 등 인공적이거나 규칙적인 패턴을 가진 환경에는 기하학적으로 완전히 동일한 구조가 무수히 반복된다.5 예를 들어, 아파트 건물의 모든 창문, 다리의 트러스 구조, 숲의 동일한 종의 나무 모델 등이 이에 해당한다.

SVDAG는 이러한 중복을 탐지하여, 구조적으로 동일한 모든 서브트리를 단 하나의 유일한 인스턴스로 대체한다.8 그 결과, 전체 데이터 구조를 구성하는 고유 노드의 수가 극적으로 감소하며, 이는 곧바로 메모리 사용량의 절감으로 이어진다. 일부 연구에서는 SVDAG가 SVO에 비해 노드 수를 수십에서 수백 배까지 줄여, 메모리 사용량을 획기적으로 감소시킬 수 있음을 보여주었다.8

### 2.3 알고리즘 심층 분석: 상향식(Bottom-up) SVDAG 구축

그렇다면 어떻게 이 거대한 SVO에서 동일한 서브트리들을 효율적으로 찾아낼 수 있을까? 가장 단순한 방법은 모든 노드 쌍에 대해 그들의 서브트리를 재귀적으로 비교하는 하향식(top-down) 접근이지만, 이는 엄청난 계산 비용을 요구하여 비현실적이다. 대신, SVDAG는 훨씬 영리하고 효율적인 상향식(bottom-up) 알고리즘을 사용한다.8

이 알고리즘은 마치 동적 프로그래밍(Dynamic Programming)처럼, 가장 작은 단위의 문제부터 해결하고 그 결과를 이용해 점차 더 큰 문제를 해결해 나가는 방식을 취한다. "두 개의 거대한 서브트리가 동일한가?"라는 복잡한 질문을 "두 노드의 자식 포인터들이 가리키는 대상이 동일한가?"라는 훨씬 간단한 질문으로 치환하는 것이 이 알고리즘의 우아함과 효율성의 핵심이다.

1단계 (리프 노드 병합):

알고리즘은 트리의 가장 낮은 레벨, 즉 리프 노드(leaf node)에서 시작한다. SVO에서 리프 노드는 자신이 포함하는 8개 복셀의 점유 상태(occupied or empty)를 나타내는 8비트 childmask로 고유하게 정의될 수 있다. 이는 이론적으로 최대 $2^8 = 256$개의 고유한 리프 노드 패턴만이 존재할 수 있음을 의미한다. 따라서 알고리즘의 첫 단계는 씬에 존재하는 수백만 개의 리프 노드들을 이 256개의 '대표 리프 노드' 중 하나를 가리키도록 포인터를 재설정하여 병합하는 것이다.8

2단계 (상위 레벨의 반복적 병합):

리프 노드 병합이 완료되면, 한 레벨 위의 부모 노드들로 이동한다. 이 레벨에서 두 노드가 '동일하다'고 판단하는 기준은 다음과 같다.

1. 두 노드의 `childmask`가 동일해야 한다.
2. 두 노드의 각 자식을 가리키는 포인터들이 가리키는 대상(이미 병합이 완료된 하위 레벨의 대표 노드)의 주소값이 모두 동일해야 한다.

이 두 조건을 모두 만족하는 노드들은 동일한 구조의 서브트리의 루트임이 보장된다. 알고리즘은 이러한 노드들을 그룹화하고, 각 그룹을 다시 하나의 대표 노드로 병합한다. 그리고 이 병합 결과를 반영하여, 한 단계 더 위의 부모 노드들의 자식 포인터들을 새롭게 갱신한다. 이 과정은 트리의 루트 노드에 도달할 때까지, 혹은 더 이상 병합할 수 있는 노드가 없을 때까지 레벨을 올려가며 반복적으로 수행된다.5

해싱(Hashing)을 이용한 최적화:

각 레벨에서 동일한 노드를 효율적으로 찾기 위해, 실제 구현에서는 해시 테이블(hash table)을 사용하는 것이 일반적이다. 각 노드의 childmask와 자식 포인터 배열을 조합하여 해시 키(hash key)를 생성한다. 이 키를 해시 테이블에 삽입하여 동일한 해시 값을 갖는 노드들, 즉 잠재적인 중복 후보들을 빠르게 선별할 수 있다. 그 후, 이 후보 그룹 내에서만 실제 내용을 비교하여 최종적으로 중복을 확정하고 병합한다.5 이 방식은 전체 노드 쌍을 비교하는 

`$O(N^2)$`의 복잡도를 평균적으로 `$O(N)$`에 가깝게 줄여준다.

이처럼 SVDAG의 상향식 구축 알고리즘은 재귀적인 동일성의 정의를 영리하게 활용하여, 복잡한 서브트리 비교 문제를 단순한 포인터 비교 문제로 변환함으로써 고도의 압축을 효율적으로 달성한다. 이는 SVDAG가 이론적 우수성뿐만 아니라 실용적인 구현 가능성까지 갖추게 된 핵심적인 이유이다.

## 3.  SVDAG 순회 및 고성능 렌더링 기법

SVDAG의 가장 강력한 장점 중 하나는, 높은 압축률을 달성하면서도 렌더링 시 별도의 압축 해제 과정이 필요 없다는 점이다.8 데이터 구조 자체가 공간 가속 구조(spatial acceleration structure)의 역할을 겸하기 때문에, 압축된 상태 그대로 매우 효율적인 순회가 가능하다. 이는 SVDAG를 단순한 저장 포맷이 아닌, 실시간 렌더링을 위한 강력한 데이터 구조로 만드는 결정적인 특징이다.

### 3.1 압축 해제 없는 직접 순회 (Direct Traversal)

SVDAG 순회 알고리즘은 그 기반이 되는 SVO의 순회 알고리즘과 본질적으로 동일하다. 순회 로직의 관점에서 보면, 현재 노드에서 다음 자식 노드로 이동할 때 사용되는 포인터가 다른 부모 노드와 공유되고 있는지 여부는 전혀 중요하지 않다. 알고리즘은 그저 주어진 포인터를 따라 다음 노드로 이동할 뿐이다. 이 '투명한 압축(transparent compression)' 덕분에, SVO를 위해 고도로 최적화된 레이 트레이싱 기법들을 거의 그대로 SVDAG에 적용할 수 있다.8

오히려 SVDAG는 SVO에 비해 더 나은 렌더링 성능을 보이기도 한다. 이는 극적으로 줄어든 메모리 풋프린트 덕분이다. 전체 데이터 구조의 크기가 작아지면 더 많은 부분이 GPU의 고속 캐시 메모리에 상주할 확률이 높아지고, 이는 메모리 접근 지연 시간(latency)을 줄여 전체적인 순회 속도를 향상시키는 효과를 가져온다.

### 3.2 레이 트레이싱(Ray Tracing) 기본 원리

레이 트레이싱은 가상의 카메라에서 화면의 각 픽셀을 향해 광선(ray)을 쏘아, 그 광선이 씬의 어떤 객체와 처음 부딪히는지를 계산하여 픽셀의 색상을 결정하는 렌더링 기법이다.

- **광선(Ray) 표현:** 기하학적으로 광선은 하나의 원점(Origin) `$O$`와 단위 방향 벡터(Direction) `$D$`로 정의된다. 광선 상의 임의의 점 `$P(t)$`는 파라미터 `$t$` (`$t \ge 0$`)를 이용하여 다음과 같은 선형 방정식으로 표현된다.20

  $$
  P(t) = O + t \cdot D
  $$
  
- **SVDAG 순회:** SVDAG를 이용한 레이 트레이싱은 루트 노드에서 시작된다. 광선이 현재 노드가 나타내는 공간(AABB)과 교차하는지 판정한다. 만약 교차한다면, 현재 노드가 리프 노드인지 내부 노드인지 확인한다. 리프 노드라면 광선과 리프 내부의 복셀 간의 충돌을 계산하고 순회를 종료한다. 내부 노드라면, 광선이 통과하는 순서대로 8개의 자식 노드들을 방문하며 이 과정을 재귀적으로 반복한다. 이 계층적 탐색을 통해, 광선과 무관한 방대한 공간을 효율적으로 건너뛸 수 있다.8

### 3.3 핵심 연산: 레이-AABB(축 정렬 경계 상자) 교차 판정

SVDAG의 모든 노드는 축에 정렬된 경계 상자(Axis-Aligned Bounding Box, AABB)로 표현될 수 있으므로, 광선과 AABB의 교차를 빠르고 강건하게 판정하는 것은 순회 알고리즘의 성능에 지대한 영향을 미치는 핵심 연산이다. 여러 기법이 있지만, 가장 널리 사용되고 이해하기 쉬운 방법은 **슬랩 기법(Slab Method)**이다.20

이 기법은 3차원 AABB를 x, y, z 각 축에 수직인 한 쌍의 무한 평면으로 구성된 세 개의 '슬랩(slab)'의 교집합으로 간주한다. 알고리즘은 각 축(슬랩)에 대해 광선이 슬랩 안으로 들어가는 시점(`$t_{near}$`)과 슬랩 밖으로 나가는 시점(`$t_{far}$`)을 계산한다.

예를 들어, x축에 대한 슬랩은 `$x = x_{min}$`과 `$x = x_{max}$` 두 평면으로 정의된다. 광선 방정식 `$O_x + t \cdot D_x = x_{plane}$`을 `$t$`에 대해 풀면 교차 시점을 구할 수 있다.

$$
t_{x1} = \frac{x_{min} - O_x}{D_x}
$$

$$
t_{x2} = \frac{x_{max} - O_x}{D_x}
$$

만약 `$D_x > 0$` 이면 `$t_{near_x} = t_{x1}$`, `$t_{far_x} = t_{x2}$`가 되고, `$D_x < 0$` 이면 그 반대가 된다. `$D_x = 0$`인 경우(광선이 x축 평면과 평행)는 AABB의 x 범위 내에 광선의 원점이 있는지 확인하는 특별한 처리가 필요하다.20

이 계산을 세 축 모두에 대해 반복하여 각 축의 (`$t_{near_x}$`, `$t_{far_x}$`), (`$t_{near_y}$`, `$t_{far_y}$`), (`$t_{near_z}$`, `$t_{far_z}$`) 구간을 얻는다. 광선이 AABB와 교차하려면 이 세 구간이 모두 겹치는 부분이 있어야 한다. 따라서, 최종적인 교차 구간의 시작점 `$t_{min}$`은 각 축의 `$t_{near}$` 값들 중 가장 큰 값이 되고, 끝점 `$t_{max}$`는 각 축의 `$t_{far}$` 값들 중 가장 작은 값이 된다.

$$
t_{min} = \max(\max(t_{near_x}, t_{near_y}), t_{near_z})
$$

$$
t_{max} = \min(\min(t_{far_x}, t_{far_y}), t_{far_z})
$$

만약 최종적으로 계산된 `$t_{min}$`이 `$t_{max}$`보다 크다면, 세 구간의 공통 부분이 존재하지 않는 것이므로 광선은 AABB와 교차하지 않는다. 그렇지 않다면 교차하며, 교차 구간은 `$[t_{min}, t_{max}]$`가 된다.

### 3.4 고급 렌더링 응용

SVDAG가 제공하는 전례 없는 수준의 기하학적 디테일은 기존의 실시간 렌더링 파이프라인에서는 구현이 어렵거나 불가능했던 고품질의 전역 조명(Global Illumination) 효과를 가능하게 한다.

- **하드/소프트 섀도우(Hard/Soft Shadows):** 씬의 특정 지점에서 광원을 향해 광선(shadow ray)을 쏘아, 그 경로상에 다른 객체가 존재하는지를 판정하여 그림자를 생성한다. SVDAG는 매우 미세한 지오메트리까지 표현할 수 있으므로, 나뭇잎 사이로 비치는 그림자나 철망의 그림자처럼 복잡하고 선명한 그림자를 실시간으로 생성하는 데 매우 효과적이다.8
- **앰비언트 오클루전(Ambient Occlusion, AO):** 물체의 틈새나 구석진 부분이 주변 환경광에 덜 노출되어 어두워지는 현상을 시뮬레이션한다. 이는 셰이딩 지점의 반구(hemisphere) 내에서 여러 방향으로 무작위 광선을 쏘아, 그 광선들이 주변 지오메트리에 의해 얼마나 많이 차폐되는지를 계산하여 구현된다. SVDAG를 사용하면 객체들이 서로 맞닿는 부분(contact area)에 생기는 미세하고 부드러운 접촉면 그림자(contact shadow)를 매우 정밀하게 표현하여 사실감을 크게 높일 수 있다.8

이처럼 SVDAG는 압축과 접근이라는 상충될 수 있는 두 가지 목표를 성공적으로 양립시킨 데이터 구조이다. 그 결과, 제한된 하드웨어 자원 내에서 극도로 복잡한 가상 세계를 표현하고 사실적으로 렌더링할 수 있는 새로운 가능성을 열었다.

## 4.  SVDAG의 진화와 변종들

SVDAG의 기본 개념인 '동일 서브트리 병합'은 매우 강력했지만, '완벽하게 동일한' 기하학적 패턴이 없는 씬에서는 압축 효율이 떨어진다는 명확한 한계가 있었다. 이러한 한계를 극복하고 더 넓은 범위의 데이터에 기술을 적용하기 위해, 컴퓨터 그래픽스 연구 커뮤니티는 '무엇을 동일하다고 간주할 것인가'에 대한 정의를 창의적으로 확장하며 다양한 SVDAG의 변종들을 발전시켰다. 이 변종들의 발전사는 '압축'과 '일반성/유연성' 사이의 끊임없는 줄다리기를 보여주는 흥미로운 과정이다.

### 4.1 SSVDAG (Symmetry-aware SVDAG)

가장 직관적인 확장 중 하나는 대칭성(symmetry)을 활용하는 것이었다. SSVDAG는 기하학적으로 정확히 일치하지는 않더라도, 거울 반사와 같은 대칭 변환(symmetry transformation)을 통해 동일해질 수 있는 서브트리들을 식별하여 병합한다.22 예를 들어, 건물의 왼쪽 날개와 오른쪽 날개는 서로 대칭이므로, 하나만 저장하고 다른 하나는 대칭 변환을 통해 참조할 수 있다.

- **원리:** 이 아이디어를 구현하기 위해 SSVDAG는 노드의 자식 포인터에 몇 비트의 추가적인 태그(tag)를 할당한다. 이 태그는 해당 자식 서브트리에 어떤 대칭 변환(예: x축 반사, y축 반사, z축 반사)이 적용되어야 하는지를 인코딩한다. 레이 트레이싱 순회 시, 자식 노드로 이동하기 전에 이 태그를 읽고, 광선의 원점과 방향에 그에 상응하는 역변환을 적용한다. 변환된 광선으로 자식 서브트리를 순회한 후, 결과를 다시 원래 좌표계로 변환한다.23
- **효과:** 건축물, 자동차, 기계 부품과 같이 인공적이고 대칭적인 구조가 많은 씬에서 SSVDAG는 기본 SVDAG보다 월등히 높은 압축률을 달성한다. 한 연구에서는 SSVDAG가 SVDAG 대비 메모리 사용량을 거의 절반으로 줄이는 결과를 보여주었다.18

### 4.2 Transform-Aware SVDAG

SSVDAG의 아이디어를 한 단계 더 일반화한 것이 Transform-Aware SVDAG이다. 이 구조는 반사 대칭뿐만 아니라, 특정 각도에 따른 회전(rotation), 일정 거리만큼의 이동(translation) 등 더 일반적인 아핀 변환(affine transformation)에 대해서도 동일성을 검사하고 서브트리를 병합한다.7 벽돌 벽이나 타일 바닥처럼 동일한 패턴이 규칙적으로 반복되는 구조를 매우 효율적으로 압축할 수 있다. 하지만 더 복잡한 변환 정보를 포인터에 인코딩하고 순회 시 이를 처리해야 하므로 구현의 복잡성은 증가한다.

### 4.3 LSVDAG (Lossy SVDAG)

지금까지의 변종들이 무손실(lossless) 압축의 틀 안에 있었다면, LSVDAG는 손실(lossy) 압축이라는 과감한 접근을 취한다. 이 기술은 '정확히 동일'하지 않고 '유사한(similar)' 서브트리들을 하나의 대표 서브트리로 강제 병합한다.26

- **원리:** 먼저 두 서브트리 간의 기하학적 차이를 정량적으로 측정하는 메트릭(metric)을 정의한다. 예를 들어, 두 서브트리에 포함된 복셀들 중 상태가 다른 복셀의 비율을 오류 값으로 사용할 수 있다. 그 다음, 이 오류 값이 사용자가 설정한 특정 임계값 이하라면, 두 서브트리가 '충분히 유사하다'고 판단하고 병합을 허용한다.
- **효과:** 이 방식은 압축률과 시각적 품질 사이의 명확한 트레이드오프를 제공한다. 약간의 시각적 오류를 감수하는 대신 압축률을 극대화할 수 있다. 특히, 3D 스캔 데이터처럼 노이즈가 많거나 비정형적인 데이터, 즉 완벽하게 동일한 패턴이 거의 없는 씬에서도 상당한 압축 효과를 얻을 수 있다는 장점이 있다.26

### 4.4 PSVDAG (Pointerless SVDAG)

PSVDAG는 정반대의 방향에서 압축률을 극대화하려는 시도이다. 이 구조는 SVDAG 메모리에서 상당한 비중을 차지하는 '포인터' 자체를 완전히 제거하는 것을 목표로 한다.16

- **원리:** 포인터 대신, 자식 노드의 메모리 위치를 부모 노드와의 상대적 위치나, 모든 노드를 저장하는 거대한 전역 배열에서의 인덱스로 암시적으로 계산한다. 이를 위해 레이블(Label)과 호출자(Caller)와 같은 새로운 개념을 도입하여 포인터 없이도 공유된 서브트리의 연결 관계를 표현한다.
- **한계:** 이 구조의 치명적인 단점은 실시간 순회가 불가능하다는 것이다. 포인터가 없기 때문에 렌더링이나 다른 처리를 수행하기 전에, 먼저 이 암시적인 연결 정보를 바탕으로 실제 포인터를 재구성하는 전처리 과정이 반드시 필요하다. 따라서 PSVDAG는 실시간 렌더링보다는, 데이터의 장기 보관을 위한 아카이빙(archiving)이나 네트워크를 통한 데이터 스트리밍(streaming)과 같은 용도에 더 적합하다.24

이러한 변종들의 발전 과정을 종합하면, SVDAG 기술 생태계가 단일한 방향이 아닌, 다양한 응용 분야의 요구에 맞춰 다각적으로 진화해왔음을 알 수 있다. 아래 표는 SVDAG와 그 주요 변종들의 핵심적인 특징과 트레이드오프를 요약하여 보여준다.

#### 4.4.1 핵심 테이블 1: SVDAG 및 주요 변종 기술 비교

| 특징/기술          | SVO (Sparse Voxel Octree) | SVDAG (Base)       | SSVDAG (Symmetry-aware)   | LSVDAG (Lossy)             | PSVDAG (Pointerless)             |
| ------------------ | ------------------------- | ------------------ | ------------------------- | -------------------------- | -------------------------------- |
| **핵심 압축 원리** | 빈 공간 제거              | 동일 서브트리 병합 | 대칭 서브트리 병합        | 유사 서브트리 병합         | 포인터 제거 + 동일 서브트리 병합 |
| **압축 유형**      | 무손실                    | 무손실             | 무손실                    | 손실                       | 무손실                           |
| **압축 효율성**    | 낮음                      | 높음               | 매우 높음                 | 극도로 높음                | 최고 수준                        |
| **실시간 순회**    | 가능                      | 가능               | 가능 (변환 오버헤드 존재) | 가능                       | 불가능 (재구성 필요)             |
| **편집 용이성**    | 비교적 용이               | 어려움             | 매우 어려움               | 매우 어려움                | 매우 어려움                      |
| **주요 응용 분야** | 기본 복셀 렌더링          | 정적 씬, 섀도우/AO | 대칭적 씬(건축, CAD)      | 비정형 데이터, 극단적 압축 | 데이터 아카이빙, 스트리밍        |
| **참조**           | 4                         | 5                  | 22                        | 26                         | 16                               |

결론적으로, 이 변종들은 SVDAG라는 핵심 아이디어가 얼마나 유연하고 확장 가능한지를 보여주는 증거이다. 각 기술은 서로 다른 설계 목표와 트레이드오프를 가지며, 이는 개발자가 자신의 응용 프로그램에 가장 적합한 압축 전략을 선택할 수 있는 풍부한 옵션을 제공한다.

## 5.  실용적 과제: 동적 편집과 속성 데이터 압축

SVDAG와 그 변종들이 정적인 3D 씬을 극도로 효율적으로 압축하고 렌더링하는 데 성공했지만, 이 기술이 연구실 수준을 넘어 실제 상용 애플리케이션(예: 게임 엔진, 3D 모델링 툴)에 널리 채택되기까지는 두 가지 큰 실용적 과제를 해결해야 했다. 첫째는 고도로 압축된 구조를 실시간으로 수정하는 '동적 편집'의 문제이고, 둘째는 단순한 기하학적 점유 정보를 넘어 색상, 재질과 같은 풍부한 '속성 데이터'를 효율적으로 다루는 문제였다. 이 과제들의 해결 과정은 SVDAG가 단순한 '표현' 기술에서 '상호작용 가능한' 기술로 진화하는 중요한 단계였다.

### 5.1 정적 구조의 한계: 실시간 편집의 어려움

SVDAG의 압축 효율은 '서브트리 공유'라는 개념에 깊이 의존하고 있다. 이는 역설적으로 구조를 편집하기 어렵게 만드는 가장 큰 원인이 된다. 만약 공유되고 있는 한 서브트리의 복셀 하나라도 수정된다면, 그 서브트리는 더 이상 원래의 '대표 서브트리'와 동일하지 않게 된다.5

이러한 변경은 연쇄적인 파급 효과를 낳는다. 먼저, 수정된 서브트리는 공유 풀(pool)에서 분리되어 고유한 사본으로 만들어져야 한다. 이로 인해 해당 서브트리를 가리키던 부모 노드의 포인터가 변경된다. 만약 이 부모 노드 또한 다른 노드들과 공유되고 있었다면, 포인터 변경으로 인해 이 부모 노드 역시 더 이상 공유될 수 없게 되어 또 다른 사본을 만들어야 한다. 이러한 변경 사항은 마치 파도처럼 트리의 루트 방향으로 거슬러 올라가며 전파될 수 있다. 이 '변경의 역전파(back-propagation of changes)' 과정은 매우 복잡하고, 최악의 경우 DAG 구조의 상당 부분을 재구성해야 하므로 계산 비용이 매우 높다.5 이 때문에 초기 SVDAG는 본질적으로 정적인 데이터에만 적합한 기술로 여겨졌다.

### 5.2 해결 방안 연구: HashDAG

이러한 동적 편집의 어려움을 해결하기 위해 등장한 대표적인 연구가 바로 Careil 등이 제안한 HashDAG이다.5 HashDAG는 이름에서 알 수 있듯이, GPU 기반의 해시 테이블을 핵심적인 도구로 사용하여 SVDAG를 대화형(interactive) 속도로 편집할 수 있게 만든 데이터 구조이다.

그 원리는 다음과 같다. SVDAG를 구성하는 모든 고유한 노드(내부 노드와 리프 노드 모두)를 GPU 메모리에 구현된 거대한 해시 테이블에 저장한다. 사용자가 조각(carving), 채우기(filling), 페인팅(painting)과 같은 편집 작업을 수행하면, 영향을 받는 복셀들의 경로가 먼저 식별된다. 그 후, HashDAG는 이 경로상의 노드들을 해시 테이블을 이용해 빠르게 복제(duplicate), 수정(modify), 그리고 재병합(re-merge)한다. 예를 들어, 특정 노드를 수정해야 할 때, 수정된 내용(새로운 `childmask`나 자식 포인터)을 키로 사용하여 해시 테이블을 조회한다. 만약 동일한 내용의 노드가 이미 존재한다면 그 노드를 가리키는 포인터를 재사용하고(재병합), 존재하지 않는다면 새로운 노드를 생성하여 해시 테이블에 삽입한다. 이 모든 과정이 GPU의 병렬 처리 능력을 활용하여 가속화되므로, 대규모의 복셀 씬을 실시간 프레임 속도로 편집하는 것이 가능해진다.17 HashDAG는 SVDAG를 정적인 '렌더링용' 데이터에서 동적인 '편집용' 데이터로 그 활용 범위를 넓힌 중요한 진전이라 할 수 있다.

### 5.3 기하학을 넘어서: 속성 데이터(Attributes) 압축

SVDAG의 또 다른 실용적 과제는 속성 데이터의 처리였다. 초기 SVDAG 연구는 대부분 복셀이 채워져 있는지 아닌지를 나타내는 1비트의 이진 점유(binary occupancy) 정보, 즉 순수한 기하학적 구조 압축에만 집중했다.1 하지만 사실적인 렌더링을 위해서는 각 복셀마다 색상(color, 24비트), 법선 벡터(normal vector, 24비트 이상), 재질 ID(material ID) 등 훨씬 더 많은 양의 속성 데이터가 필요하다.1

만약 지오메트리를 비트당 0.1 복셀 수준으로 극단적으로 압축했더라도, 복셀당 수십 비트의 속성 데이터가 압축되지 않은 채 남아있다면 전체적인 메모리 절감 효과는 무의미해진다.1 이 문제를 해결하기 위해 여러 가지 창의적인 속성 압축 전략이 제안되었다.

1. **속성 데이터 분리 및 간접 참조:** 가장 일반적인 접근법 중 하나는 지오메트리 정보(SVDAG)와 속성 정보를 분리하는 것이다. SVDAG의 리프 노드에는 실제 속성 값 대신, 속성 팔레트(palette)나 텍스처 아틀라스(texture atlas)의 인덱스를 저장한다. 실제 속성 데이터는 이 별도의 자료구조에 모아서 저장하고, 텍스처 압축(예: BCn)과 같은 전통적인 기법을 사용하여 압축한다. 이 방식은 속성의 종류가 제한적인 경우 매우 효과적이다.1
2. **지오메트리와 속성 통합 압축:** 또 다른 접근법은 속성 데이터를 SVDAG 구조에 직접 통합하는 것이다. 이 경우, 두 노드가 동일하다고 판단되려면 기하학적 구조(자식 포인터)뿐만 아니라 그들의 속성 값(또는 속성 인덱스)까지 동일해야 한다. 이 방식은 구조를 단순하게 유지할 수 있지만, 동일한 모양이라도 색깔만 다르면 별개의 노드로 취급되므로 기하학적 압축률이 다소 저하될 수 있다.17
3. **레이블 정보 분리 (AABB 활용):** 최근의 한 연구에서는 대규모 분할 볼륨(segmentation volume) 데이터를 다루기 위해 새로운 접근법을 제시했다. 이 방법은 먼저 동일한 레이블(속성)을 가진 복셀들을 고정된 크기의 AABB(축 정렬 경계 상자)들로 그룹화한다. 그 다음, 모든 AABB에 걸쳐 나타나는 복셀의 점유 패턴(모양) 정보만을 하나의 거대한 공유 SVDAG로 압축한다. 레이블 정보는 각 AABB와 한 번만 연결되므로, SVDAG 그래프 인코딩 자체에서 속성 정보가 완전히 분리된다. 이는 속성 정보로 인해 그래프가 파편화되는 것을 막아 압축률을 크게 향상시킨다.28

이처럼 동적 편집과 속성 데이터 압축이라는 실용적 과제에 대한 다양한 해결책의 등장은, SVDAG가 학술적 개념을 넘어 실제 산업 현장에서 사용될 수 있는 성숙한 기술로 발전해왔음을 보여주는 명백한 증거이다.

## 6.  성능 분석 및 비교 고찰

SVDAG의 기술적 가치를 객관적으로 평가하기 위해서는 정량적인 성능 지표를 분석하고, 동시대의 다른 주요 3D 지오메트리 표현 기술들과의 장단점을 비교하는 과정이 필수적이다. 이를 통해 SVDAG가 컴퓨터 그래픽스 기술 생태계에서 차지하는 고유한 위치와 그 의의를 명확히 할 수 있다.

### 6.1 정량적 성능 분석

여러 학술 연구에서 제시된 실험 결과들은 SVDAG의 압도적인 성능을 일관되게 보여준다.

- **메모리 압축률:** SVDAG의 가장 두드러진 장점은 메모리 사용량이다. 기본 SVDAG는 SVO 대비 수십 배, 조밀 그리드와 비교하면 수천 배 이상의 압축률을 달성할 수 있다. 한 걸음 더 나아간 SSVDAG의 경우, 한 연구에서 SVO가 16.2 GB의 메모리를 요구하는 PowerPlant 씬을 단 86 MB 미만으로 저장하는 데 성공했다. 이는 약 200배 이상의 압축률에 해당한다.23 또 다른 연구에서는 SVO가 31.1 GB를 필요로 하는 씬을 SSVDAG는 575 MB(약 55배 이상 압축)로 표현했다.18 이러한 결과는 복셀당 비트(bits per voxel) 단위로 환산했을 때 0.1 미만, 심지어 0.05 미만의 극적인 효율성을 의미하며, 이는 기존의 어떤 복셀 표현 방식도 달성하지 못했던 수준이다.4
- **렌더링 성능:** 놀라운 점은 이러한 극단적인 압축이 렌더링 성능의 저하를 거의 동반하지 않는다는 것이다. SVDAG는 압축된 상태에서 직접 레이 트레이싱을 수행함에도 불구하고, 최첨단 삼각형 기반 GPU 레이 트레이서와 비교하여 경쟁력 있는, 혹은 때로는 더 나은 성능을 보인다. 예를 들어, 한 연구에서는 SVDAG를 사용하여 초당 수억 개의 광선(Mray/s)을 처리하는 성능을 달성했으며, 이는 동일한 하드웨어에서 실행되는 최적화된 삼각형 레이 트레이서와 대등한 수준이었다.8 이러한 고성능은 앞서 언급했듯이, 메모리 대역폭 요구량의 감소와 그로 인한 GPU 캐시 활용률 증가에 기인한다.

### 6.2 SVDAG vs. 폴리곤 메시 (Polygon Mesh)

SVDAG와 폴리곤 메시는 3D 객체를 표현하는 근본적인 철학에서부터 차이를 보인다. 이 차이를 이해하는 것이 두 기술을 올바르게 비교하는 첫걸음이다.

- **데이터 표현 철학:** SVDAG는 공간을 이산적인 볼륨 요소(복셀)로 샘플링하는 '샘플 기반(sample-based)' 표현이다. 공간의 모든 지점은 채워져 있거나 비어있는 상태 중 하나이다. 반면, 폴리곤 메시는 객체의 '경계(boundary)', 즉 표면을 수학적으로 근사하는 '경계 표현(boundary representation)'이다.3 SVDAG가 객체의 내부까지 정의하는 반면, 폴리곤 메시는 기본적으로 껍데기만을 정의한다.
- **장단점 비교:**
  - **SVDAG:** 복잡한 유기적 형태, 자연물, 구름이나 연기와 같은 볼류메트릭 효과, 그리고 실시간으로 파괴 가능한 환경을 표현하는 데 강력한 이점을 가진다. 데이터 구조가 규칙적이고 단순하여 절차적 생성이나 물리 시뮬레이션과의 결합이 용이하다.6 하지만 정점 단위의 세밀한 변형 애니메이션(deformation animation)을 적용하기 어렵고, 거대하고 평평한 단일 표면을 표현할 때는 오히려 폴리곤 메시보다 비효율적일 수 있다.31
  - **폴리곤 메시:** 평면이나 기계적인 형태를 표현하는 데 매우 효율적이다. 스키닝(skinning), 블렌드 셰이프(blend shape) 등 수십 년간 발전해 온 성숙한 애니메이션 파이프라인과 UV 맵핑을 통한 텍스처링 기법을 완벽하게 지원한다.32 그러나 극도로 복잡한 디테일(예: 프랙탈, 초고밀도 스캔 데이터)을 표현하고자 할 때 폴리곤 수가 관리 불가능한 수준으로 폭발적으로 증가하는 한계가 있다.1

결론적으로, 두 기술은 어느 한쪽이 우월한 대체 관계라기보다는, 표현하고자 하는 대상의 특성에 따라 강점과 약점이 나뉘는 상호 보완적인 관계에 가깝다. 현대의 많은 게임 엔진들이 배경 지형은 복셀 기반으로, 캐릭터나 주요 소품은 폴리곤 메시 기반으로 제작하는 하이브리드 접근법을 채택하는 이유가 바로 여기에 있다.6

### 6.3 SVDAG vs. NanoVDB

SVDAG가 주로 학술 연구 커뮤니티를 중심으로 발전해 온 기술이라면, VDB(그리고 그 GPU 버전인 NanoVDB)는 영화 시각 효과(VFX) 산업의 실질적인 표준으로 자리 잡은 기술이다.33 두 기술 모두 희소 복셀 데이터를 다루지만, 그 구조와 설계 철학에는 미묘하지만 중요한 차이가 있다.

- **구조적 차이:** SVDAG는 일반적으로 고정된 8분할 구조를 갖는 옥트리(octree)를 기반으로 하며, '서브트리 공유'를 통한 압축률 극대화에 설계 초점이 맞춰져 있다. 반면, VDB는 더 넓고 유연한 분기 계수(branching factor)를 가질 수 있는 B+ 트리와 유사한 계층 구조를 사용한다. 특히 GPU 버전인 NanoVDB는 GPU 아키텍처에 극도로 최적화되어 있다. NanoVDB는 전체 트리 구조를 하나의 연속된 메모리 블록으로 선형화(linearize)하고, 포인터를 제거하여(pointer-less) GPU로의 데이터 전송 및 순회 성능을 극대화한 읽기 전용(read-only) 구조를 채택했다.35
- **철학의 차이:** 이러한 구조적 차이는 두 기술의 설계 철학 차이에서 비롯된다. SVDAG는 '궁극의 무손실 압축'을 달성하는 것을 중요한 목표 중 하나로 삼는 경향이 강하다. 반면, NanoVDB는 산업 현장의 복잡한 파이프라인과의 호환성(OpenVDB)과 GPU에서의 '실시간 랜덤 액세스 성능'을 최우선으로 고려한다.33 NanoVDB가 토폴로지를 정적으로 가정하고 읽기 전용으로 설계된 것은, 동적 편집의 유연성을 일부 희생하더라도 예측 가능한 최고 수준의 렌더링 성능을 보장하기 위한 전략적 선택이다.35

### 6.4 주요 3D 지오메트리 표현 기술 비교

SVDAG를 현대 컴퓨터 그래픽스의 전체 기술 스펙트럼 내에서 객관적으로 평가하기 위해, 폴리곤 메시, NanoVDB, 그리고 최근 각광받는 뉴럴 래디언스 필드(NeRF)까지 포함한 주요 기술들을 아래 표와 같이 비교할 수 있다. 이 비교는 각 기술의 근본적인 '존재론'의 차이를 드러낸다. 폴리곤 메시는 객체의 '껍데기'를, SVDAG와 NanoVDB는 '공간의 내용물'을, NeRF는 '공간을 바라보는 방법' 자체를 기술한다. 이 철학적 차이가 각 기술의 고유한 장단점과 적합한 응용 분야를 결정한다.

#### 6.4.1 핵심 테이블 2: 주요 3D 지오메트리 표현 기술 비교

| 기준              | 폴리곤 메시 (Polygon Mesh)   | SVDAG                                    | NanoVDB                     | NeRF (Neural Radiance Field)   |
| ----------------- | ---------------------------- | ---------------------------------------- | --------------------------- | ------------------------------ |
| **기본 단위**     | 정점, 간선, 면 (주로 삼각형) | 복셀 (점유/비점유)                       | 복셀 그리드 (값)            | 신경망 가중치 (MLP)            |
| **표현 방식**     | 명시적 경계 표현             | 명시적 공간 샘플링                       | 명시적 공간 샘플링          | 암시적 함수 근사               |
| **메모리 효율성** | 디테일에 따라 가변적         | 매우 높음 (압축률)                       | 높음 (GPU 친화적)           | 매우 높음 (모델 크기)          |
| **렌더링 속도**   | 매우 빠름 (래스터화)         | 빠름 (레이 트레이싱)                     | 매우 빠름 (GPU 최적화)      | 느림 (신경망 연산)             |
| **실시간 편집**   | 용이 (성숙한 도구)           | 어려움 (HashDAG 등 필요)                 | 제한적 (읽기 전용)          | 불가능 (재학습 필요)           |
| **데이터 획득**   | 모델링, 스캐닝               | 복셀화(Voxelization)                     | 복셀화, 시뮬레이션          | 다수의 이미지 (SfM)            |
| **강점**          | 애니메이션, 평면 표현        | 극도로 복잡한 정적 지오메트리, 파괴 효과 | VFX 파이프라인, 볼륨 렌더링 | 사실적 뷰 합성, 반사/투과 표현 |
| **약점**          | 볼륨/유기체 표현 한계        | 애니메이션, 동적 씬                      | 동적 토폴로지 변경          | 기하학적 정확성, 학습 시간     |
| **참조**          | 1                            | 5                                        | 33                          | 37                             |

이 비교를 통해 SVDAG의 고유한 강점이 명확해진다. SVDAG는 '명시적(explicit)'이고 '결정론적(deterministic)'인 기하학 표현 방식이다. 이는 NeRF와 같은 '암시적(implicit)'이고 '확률적(probabilistic)'인 표현 방식이 제공하지 못하는 예측 가능성과 직접적인 제어 가능성을 의미한다. 기술의 선택은 단순히 성능 수치를 비교하는 것을 넘어, '무엇을, 어떻게 표현하고 제어하고 싶은가'에 대한 근본적인 질문에 달려있다. 제어 가능한 고밀도 '지오메트리'가 필요한 응용 분야에서 SVDAG는 여전히 타의 추종을 불허하는 강력한 선택지로 남아있다.

## 7. 결론: 복셀 표현의 현재와 미래 조망

희소 복셀 방향성 비순환 그래프(SVDAG)는 3D 컴퓨터 그래픽스 분야에서 '복잡성'이라는 오랜 난제를 해결하기 위한 탐구 과정에서 탄생한 중요한 기술적 성취이다. 그 여정을 되짚어보고 현재의 위치를 평가하며 미래를 조망하는 것은, 복셀 기반 표현 방식의 잠재력과 차세대 그래픽스 기술의 발전 방향을 가늠하는 데 중요한 시사점을 제공한다.

### SVDAG의 기여 요약

SVDAG의 가장 핵심적인 기여는 3D 데이터 압축의 패러다임을 '희소성(sparsity)'의 활용에서 '중복성(redundancy)'의 제거로 확장했다는 점에 있다. SVO가 비어있는 공간을 효율적으로 처리했다면, SVDAG는 한 걸음 더 나아가 씬 내에 반복되는 기하학적 구조를 그래프 이론을 통해 단 한 번만 저장하는 혁신을 이뤄냈다. 그 결과, 이전에는 상상하기 어려웠던 수십억 단위의 복셀로 구성된 극도로 상세한 3D 씬을 제한된 GPU 메모리 내에 담아 실시간으로 렌더링하는 것을 가능하게 만들었다. 이는 단순히 메모리를 절약하는 것을 넘어, 실시간 그래픽스가 표현할 수 있는 기하학적 디테일의 상한선 자체를 끌어올린 중요한 학술적, 기술적 진보였다.

### 7.1 현재의 위치와 한계

현재 SVDAG는 특히 고밀도의 정적 지오메트리가 요구되는 특정 응용 분야에서 매우 강력한 솔루션으로 그 가치를 입증했다. 초고해상도 섀도우 맵 생성, 정밀한 앰비언트 오클루전 계산, 그리고 대규모 가상 환경(예: 도시, 자연 지형)의 표현 등에서 SVDAG는 기존 기술 대비 압도적인 효율성과 품질을 보여주었다.19 그러나 SVDAG가 모든 분야의 만능 해결책은 아니다. 공유에 기반한 복잡한 데이터 구조는 필연적으로 동적 편집과 캐릭터 애니메이션 같은 변형 작업에 어려움을 야기한다.27 HashDAG와 같은 연구가 실시간 편집의 가능성을 열었지만, 전통적인 폴리곤 메시 기반의 성숙한 애니메이션 파이프라인과 비교할 때 여전히 유연성과 편의성 면에서 한계가 명확하다. 이러한 한계는 SVDAG가 인터랙티브 콘텐츠의 핵심인 동적 캐릭터나 오브젝트보다는, 주로 정적인 배경이나 환경 표현에 사용되는 경향으로 이어졌다.

### 7.2 미래 전망: 새로운 표현 방식과의 융합

컴퓨터 그래픽스 분야는 끊임없이 진화하고 있으며, 최근 몇 년간 뉴럴 래디언스 필드(NeRF)와 3D 가우시안 스플래팅(Gaussian Splatting)과 같은 신경망 기반의 새로운 3D 표현 방식들이 씬 재구성과 뷰 합성(view synthesis) 분야에서 놀라운 발전을 이끌고 있다.37 이들 기술은 여러 장의 2D 이미지로부터 3D 씬의 빛과 형태를 학습하여, 어떤 시점에서든 극도로 사실적인 이미지를 생성하는 데 탁월한 성능을 보인다. 이들은 SVDAG와는 전혀 다른 방식으로 '복잡성' 문제를 해결하며, 그래픽스 기술의 미래에 대한 새로운 질문을 던지고 있다.

이러한 상황에서 SVDAG의 미래는 어떻게 될 것인가? 단일 기술이 모든 것을 지배하기보다는, 각 기술의 장점을 결합하는 하이브리드(hybrid) 접근법이 미래의 주류가 될 가능성이 높다. 기술의 발전은 단순한 대체가 아닌, 공존과 융합의 방향으로 나아갈 것이기 때문이다.

- **역할 분담 기반의 하이브리드:** 예를 들어, 하나의 게임 월드를 구성할 때, NeRF나 가우시안 스플래팅을 사용하여 씬의 전반적인 모습과 복잡한 간접광 조명을 표현하고, 플레이어와 직접 상호작용해야 하는 캐릭터, 차량, 파괴 가능한 건물 등은 명시적인 기하학 구조를 가진 SVDAG로 모델링하여 두 기술의 장점을 모두 취할 수 있다.
- **구조적 융합:** SVDAG와 같은 희소 복셀 구조의 아이디어를 신경망 기술에 접목하는 연구도 활발히 진행되고 있다. NSVF(Neural Sparse Voxel Fields)나 Plenoxels와 같은 기술들은 희소 복셀 그리드를 NeRF의 공간 가속 구조로 활용하여, 신경망의 학습 및 렌더링 속도를 획기적으로 개선하는 성과를 보였다.38 이는 SVDAG의 핵심 원리가 새로운 기술 패러다임 속에서도 여전히 유효하며, 시너지를 창출할 수 있음을 보여준다.

결론적으로, SVDAG는 컴퓨터 그래픽스라는 거대한 도구 상자 안에서 '고밀도 명시적 지오메트리'를 다루는 매우 날카롭고 효율적인 전문 도구로서 그 가치를 계속 유지할 것이다. NeRF와 같은 새로운 기술의 등장은 SVDAG의 종말을 의미하는 것이 아니라, 오히려 각 기술이 가장 잘할 수 있는 고유한 영역을 명확히 하고 상호 보완적인 융합의 가능성을 열어주는 계기가 될 것이다. 정밀한 충돌 감지, 물리 기반 시뮬레이션, 절차적 콘텐츠 생성(procedural content generation) 등, 제어 가능하고 예측 가능한 명시적 지오메트리가 필수적인 분야에서 SVDAG의 역할은 앞으로도 중요하게 남을 것이다.40 SVDAG의 여정은 아직 끝나지 않았으며, 미래의 그래픽스 파이프라인 속에서 새로운 기술들과 공존하며 진화해 나갈 것이다.

#### 참고자료

1. Sparse Voxel DAGs for Shadows and for Geometry with Colors, accessed August 4, 2025, https://research.chalmers.se/publication/502709/file/502709_Fulltext.pdf
2. Computer graphics - Wikipedia, accessed August 4, 2025, https://en.wikipedia.org/wiki/Computer_graphics
3. Polygon mesh - Wikipedia, accessed August 4, 2025, https://en.wikipedia.org/wiki/Polygon_mesh
4. Sparse Voxel DAGs - Chalmers Publication Library, accessed August 4, 2025, https://publications.lib.chalmers.se/records/fulltext/240766/240766.pdf
5. Editing Compact Voxel Representations on the GPU - Computer ..., accessed August 4, 2025, https://publications.graphics.tudelft.nl/rails/active_storage/blobs/redirect/eyJfcmFpbHMiOnsibWVzc2FnZSI6IkJBaHBBaG9PIiwiZXhwIjpudWxsLCJwdXIiOiJibG9iX2lkIn19--60dbf4e0296df500d54408336cae7abc46a82c27/preprint.pdf
6. Oasis - Info/Graphics - Refuge Studios, accessed August 4, 2025, https://oasis.refugestudios.com.au/info/graphics
7. research.tudelft.nl, accessed August 4, 2025, [https://research.tudelft.nl/en/publications/transform-aware-sparse-voxel-directed-acyclic-graphs#:~:text=Sparse%20Voxel%20Directed%20Acyclic%20Graphs%20(SVDAGs)%20have%20proven%20to%20be,improved%20when%20considering%20mirror%20symmetries.](https://research.tudelft.nl/en/publications/transform-aware-sparse-voxel-directed-acyclic-graphs#:~:text=Sparse Voxel Directed Acyclic Graphs (SVDAGs) have proven to be,improved when considering mirror symmetries.)
8. High Resolution Sparse Voxel DAGs, accessed August 4, 2025, https://icg.gwu.edu/sites/g/files/zaxdzs6126/files/downloads/highResolutionSparseVoxelDAGs.pdf
9. Building a High-Performance Voxel Engine in Unity: A Step-by-Step Guide Part 6: High Resolution Sparse Voxel DAGs - Medium, accessed August 4, 2025, https://medium.com/@adamy1558/building-a-high-performance-voxel-engine-in-unity-a-step-by-step-guide-part-6-high-resolution-e0217efd56d2
10. Efficient Sparse Voxel Octrees – Analysis, Extensions, and Implementation - Research at NVIDIA, accessed August 4, 2025, https://research.nvidia.com/sites/default/files/pubs/2010-02_Efficient-Sparse-Voxel/laine2010tr1_paper.pdf
11. Efficient Sparse Voxel Octrees - ResearchGate, accessed August 4, 2025, https://www.researchgate.net/publication/47645140_Efficient_Sparse_Voxel_Octrees
12. Efficient Sparse Voxel Octrees - Research at NVIDIA, accessed August 4, 2025, https://research.nvidia.com/sites/default/files/pubs/2010-02_Efficient-Sparse-Voxel/laine2010i3d_paper.pdf
13. efficient-sparse-voxel-octrees/README at master - GitHub, accessed August 4, 2025, https://github.com/poelzi/efficient-sparse-voxel-octrees/blob/master/README
14. Efficient Ray Tracing of Sparse Voxel Octrees on an FPGA - NTNU Open, accessed August 4, 2025, https://ntnuopen.ntnu.no/ntnu-xmlui/bitstream/handle/11250/2370620/567036_FULLTEXT01.pdf?sequence=1
15. 방향성 비순환 그래프 - IT신비, accessed August 4, 2025, [https://shinbe.tistory.com/entry/%EB%B0%A9%ED%98%95%EC%84%B1-%EB%B9%84%EC%88%9C%ED%99%98-%EA%B7%B8%EB%9E%98%ED%94%84](https://shinbe.tistory.com/entry/방형성-비순환-그래프)
16. PSVDAG: COMPACT VOXELIZED REPRESENTATION OF 3D SCENES USING POINTERLESS SPARSE VOXEL DIRECTED ACYCLIC GRAPHS Liberios Vokorokos, - Computing and Informatics, accessed August 4, 2025, https://cai.type.sk/content/2020/3/psvdag-compact-voxelized-representation-of-3d-scenes-using-pointerless-sparse-voxel-directed-acyclic-graphs/5078.pdf
17. Editing Compact Voxel Representations on the GPU - Eurographics Association, accessed August 4, 2025, https://diglib.eg.org/bitstream/handle/10.2312/pg20241310/pg20241310.pdf
18. Symmetry-aware Sparse Voxel DAGs (SSVDAGs) for compression-domain tracing of high-resolution geometric scenes - Journal of Computer Graphics Techniques, accessed August 4, 2025, https://jcgt.org/published/0006/02/01/paper.pdf
19. High Resolution Sparse Voxel DAGs - Page has been moved, accessed August 4, 2025, https://www.cse.chalmers.se/~uffe/HighResolutionSparseVoxelDAGs.pdf
20. A Minimal Ray-Tracer - Scratchapixel, accessed August 4, 2025, https://www.scratchapixel.com/lessons/3d-basic-rendering/minimal-ray-tracer-rendering-simple-shapes/ray-box-intersection.html
21. Check If ray has Intersection with virtual Box - Stack Overflow, accessed August 4, 2025, https://stackoverflow.com/questions/62507603/check-if-ray-has-intersection-with-virtual-box
22. SSVDAGs: symmetry-aware sparse voxel DAGs | Request PDF - ResearchGate, accessed August 4, 2025, https://www.researchgate.net/publication/288315851_SSVDAGs_symmetry-aware_sparse_voxel_DAGs
23. SSVDAGs: Symmetry-aware Sparse Voxel DAGs - Publications CRS4, accessed August 4, 2025, https://publications.crs4.it/pubdocs/2016/JMG16/i3d2016-symmetry-dags.pdf
24. Transforming Hierarchical Data Structures – a PSVDAG–SVDAG Conversion Algorithm - Acta Polytechnica Hungarica !, accessed August 4, 2025, https://acta.uni-obuda.hu/Mados_Adam_115.pdf
25. Transform-Aware Sparse Voxel Directed Acyclic Graphs - TU Delft Research Portal, accessed August 4, 2025, https://research.tudelft.nl/en/publications/transform-aware-sparse-voxel-directed-acyclic-graphs
26. Delft University of Technology Lossy Geometry Compression for High Resolution Voxel Scenes, accessed August 4, 2025, https://pure.tudelft.nl/ws/files/90809458/LossyGeometryCompressionForHighResolutionVoxelScenes.pdf
27. Phyronnaz/HashDAG: Interactively Modifying Compressed Sparse Voxel Representations, accessed August 4, 2025, https://github.com/Phyronnaz/HashDAG
28. SVDAG Compression for Segmentation Volume Path Tracing, accessed August 4, 2025, https://cg.ivd.kit.edu/publications/2024/segmentation_svdag/svdag_segmentation_volumes.pdf
29. What is the difference between NURBs model and a mesh? - Holocreators, accessed August 4, 2025, https://holocreators.com/blog/what-is-the-difference-between-a-nurbs-model-and-a-polygon-mesh/
30. How Voxels Became 'The Next Big Thing' | by 80Level - Medium, accessed August 4, 2025, https://medium.com/@EightyLevel/how-voxels-became-the-next-big-thing-4eb9665cd13a
31. Are voxels the future of rendering? : r/GraphicsProgramming - Reddit, accessed August 4, 2025, https://www.reddit.com/r/GraphicsProgramming/comments/1l4oh78/are_voxels_the_future_of_rendering/
32. Mesh or surface, understanding the difference - Grasshopper - McNeel Forum, accessed August 4, 2025, https://discourse.mcneel.com/t/mesh-or-surface-understanding-the-difference/75151
33. NanoVDB - NVIDIA Developer, accessed August 4, 2025, https://developer.nvidia.com/nanovdb
34. ASWF - OpenVDB Adds GPU Support with NanoVDB; OpenVDB Version 7.1 Now Available, accessed August 4, 2025, https://www.aswf.io/news/nanovdb/
35. OpenVDB: FAQ - GitHub Pages, accessed August 4, 2025, https://academysoftwarefoundation.github.io/openvdb/NanoVDB_FAQ.html
36. Accelerating OpenVDB on GPUs with NanoVDB | NVIDIA Technical Blog, accessed August 4, 2025, https://developer.nvidia.com/blog/accelerating-openvdb-on-gpus-with-nanovdb/
37. NeRF: Neural Radiance Field in 3D Vision: A Comprehensive Review - arXiv, accessed August 4, 2025, https://arxiv.org/html/2210.00379v6
38. Neural radiance field - Wikipedia, accessed August 4, 2025, https://en.wikipedia.org/wiki/Neural_radiance_field
39. What is NeRF? - Neural Radiance Fields Explained - AWS, accessed August 4, 2025, https://aws.amazon.com/what-is/neural-radiance-fields/
40. Aokana: A GPU-Driven Voxel Rendering Framework for Open World Games - arXiv, accessed August 4, 2025, https://arxiv.org/html/2505.02017v1
41. Voxelisation Algorithms and Data Structures: A Review - MDPI, accessed August 4, 2025, https://www.mdpi.com/1424-8220/21/24/8241
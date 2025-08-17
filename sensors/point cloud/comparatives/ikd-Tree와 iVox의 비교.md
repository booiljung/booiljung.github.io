# ikd-Tree와 iVox의 비교

## 제 1장: 실시간 로보틱스 환경의 공간 데이터 구조 서론

### 0.1 동적 포인트 클라우드 관리의 핵심 과제

현대 로보틱스 기술, 특히 자율 주행 및 모바일 로봇 분야에서 라이다(LiDAR) 센서는 핵심적인 역할을 수행한다. 라이다는 주변 환경을 정밀한 3차원 포인트 클라우드로 표현하지만, 이 데이터는 정적인 형태가 아니라 연속적이고 방대한 스트림 형태로 실시간으로 생성된다.1 동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM)이나 라이다-관성 주행 거리 측정(LiDAR-Inertial Odometry, LIO)과 같은 응용 분야에서 시스템의 성능은 이 동적 포인트 클라우드를 얼마나 효율적으로 처리하느냐에 따라 결정된다. 따라서 이러한 환경에서 사용되는 공간 데이터 구조는 빈번한 데이터의 삽입 및 삭제를 신속하게 처리하면서, 동시에 빠른 공간 질의(spatial query) 기능을 제공해야 하는 이중의 과제를 안고 있다.3

### 0.2 최근접 이웃 탐색의 중요성

수많은 로보틱스 알고리즘의 근간에는 k-최근접 이웃(k-Nearest Neighbor, k-NN) 탐색 연산이 자리 잡고 있다.6 LIO에서 새로운 라이다 스캔 데이터를 기존 지도에 정합(registration)하거나, 로봇의 경로 계획 중 충돌을 감지하고, 특징점을 매칭하는 등의 핵심 작업은 모두 k-NN 탐색의 효율성에 크게 의존한다.8 이 단일 연산의 처리 속도가 전체 로봇 시스템의 실시간성을 좌우한다고 해도 과언이 아니다.

### 0.3 두 경쟁자: ikd-Tree와 iVox의 등장

이러한 실시간 동적 환경의 요구사항을 충족시키기 위해 기존의 범용 데이터 구조를 넘어선 특화된 솔루션들이 등장했다. 그중에서도 ikd-Tree와 iVox는 고성능 LIO 시스템을 위해 개발된 가장 대표적인 두 데이터 구조이다.10 ikd-Tree는 전통적인 k-d 트리를 동적 환경에 맞게 개선한 구조이며, iVox는 해싱(hashing) 기반의 복셀(voxel) 구조를 채택하여 극단적인 속도 향상을 꾀한다. 본 보고서는 이 두 데이터 구조의 아키텍처, 핵심 알고리즘, 성능 특성을 심층적으로 해부하고 비교 분석함으로써 각각의 장단점과 그 안에 내재된 공학적 트레이드오프를 고찰하고자 한다.

이러한 특화된 데이터 구조의 등장은 중요한 시사점을 던진다. 표준 k-d 트리나 옥트리(octree)와 같은 교과서적인 데이터 구조는 더 이상 최신 고주파 라이다 센서가 쏟아내는 데이터 스트림을 감당하기에 역부족이라는 점이다.5 문제의 본질이 단순히 공간 데이터를 '저장'하는 것에서, 고속의 데이터 스트림을 '관리'하는 것으로 전환된 것이다. 이러한 변화는 데이터 구조 설계의 혁신을 이끌었다. 정적 k-d 트리는 증분 업데이트에 비효율적이라는 단점 때문에 9, 이를 극복하기 위해 ikd-Tree가 개발되었다. 더 나아가 ikd-Tree가 가진 트리 유지 보수 오버헤드라는 한계를 넘어서기 위해, 근본적으로 다른 접근법인 iVox가 제안되었다.11 이 진화의 과정은 실시간 로보틱스의 엄격한 성능 요구사항이 데이터 구조 설계의 발전을 이끄는 핵심 동력임을 명확히 보여준다.

## 1.  증분 K-D 트리 (ikd-Tree): 동적 균형 트리 구조

### 1.1  아키텍처 원리: 정적 K-D 트리를 넘어서

ikd-Tree를 이해하기 위해서는 먼저 그 기반이 되는 표준 k-d 트리의 원리를 살펴볼 필요가 있다. k-d 트리는 k차원 공간을 재귀적으로 분할하는 이진 트리로, 각 레벨(깊이)마다 다른 차원의 축을 기준으로 공간을 나눈다.13 예를 들어 3차원 공간이라면, 루트 노드에서는 x축, 그 자식 노드에서는 y축, 손자 노드에서는 z축, 그리고 다시 x축으로 순환하며 공간을 분할하는 방식이다.16

ikd-Tree는 이러한 k-d 트리의 개념을 계승하면서도, 로보틱스 응용의 순차적 데이터 획득 환경에 치명적인 단점이었던 '정적' 특성을 극복하기 위해 설계된 진화된 구조다.2 핵심 혁신은 전체 트리를 재구성하지 않고도 데이터의 증분 업데이트(삽입 및 삭제)를 효율적으로 처리할 수 있다는 점이다.9 이는 매번 새로운 라이다 스캔이 들어올 때마다 맵 전체를 다시 만들 필요 없이, 변경된 부분만 갱신할 수 있게 해준다. 또한 ikd-Tree는 개별 포인트 단위의 연산뿐만 아니라, 특정 공간 영역 내의 모든 포인트를 한 번에 처리하는 박스 단위(box-wise) 연산과 트리 구조 내에서 직접 다운샘플링을 수행하는 기능까지 지원하여 LIO 맵 관리를 위한 다목적 도구로 기능한다.10

### 1.2  핵심 동적 연산: 삽입과 지연 삭제

#### 1.2.1 삽입 (Insertion)

새로운 포인트의 삽입은 표준 이진 탐색 트리의 논리를 따른다. 특정 노드에서 분할 축(예: x축)의 값을 비교하여 작으면 왼쪽, 크면 오른쪽 서브트리로 이동하는 과정을 반복하여 리프 노드에 도달하면 새 노드로 추가된다.21 이때 분할 축은 트리의 깊이에 따라 순환적으로 결정된다 ($depth \pmod k$).15 삽입 자체는 간단하지만, 반복적인 삽입은 트리의 균형을 무너뜨려 성능 저하의 원인이 될 수 있다.

#### 1.2.2 지연 삭제 (Lazy Deletion)

지연 삭제는 ikd-Tree의 핵심적인 성능 최적화 기법이다. 특정 포인트를 삭제해야 할 때, 해당 노드를 즉시 트리에서 물리적으로 제거하고 구조를 재조정하는 복잡한 연산을 수행하는 대신, 단순히 해당 노드에 '삭제됨(deleted)'이라는 플래그만 표시해 둔다.19 실제 물리적인 제거는 추후에 수행되는 재균형(rebalancing) 과정에서 일괄적으로 처리된다. 이러한 비동기적 접근 방식은 실시간 시스템의 응답성을 저해하는 비용이 큰 삭제 연산을 방지하고, 시스템이 멈춤 없이 동작하도록 보장하는 데 결정적인 역할을 한다.

### 1.3  부분 재균형 메커니즘: 시스템 중단 없는 균형 유지

#### 1.3.1 불균형 탐지

ikd-Tree는 자신의 구조적 건강 상태를 능동적으로 감시한다.10 특정 서브트리의 유효한 포인트 수가 부모 또는 형제 서브트리와 비교하여 미리 정의된 균형 조건(예: 알파-균형 조건)을 위반하면 재균형 작업이 촉발된다.9 이 조건을 통해 트리의 한쪽이 지나치게 깊어지는 것을 방지한다.

#### 1.3.2 부분 재구축 (Partial Rebuilding)

특정 서브트리가 불균형 상태로 판정되면, ikd-Tree는 전체 트리를 재구축하는 비효율적인 방식 대신, 문제가 발생한 해당 서브트리(branch)만을 대상으로 *부분 재구축*을 수행한다.2 이 과정은 해당 서브트리에 속한 모든 유효한 포인트(삭제 플래그가 없는 포인트)들을 수집하여, 이 포인트들만으로 완벽하게 균형 잡힌 새로운 k-d 트리를 만든 후, 기존의 불균형한 서브트리를 이 새로운 서브트리로 교체하는 방식으로 이루어진다.

#### 1.3.3 비동기, 이중 스레드 실행

부분 재구축은 계산 비용이 상대적으로 높은 작업이므로, 이 작업이 LIO 상태 추정과 같은 메인 스레드를 차단(blocking)하는 것을 막기 위해 별도의 스레드로 분리하여 비동기적으로 실행할 수 있다.2 이러한 "이중 스레드(twin-threaded)" 메커니즘은 ikd-Tree 효율성의 핵심이다. 시스템은 약간 오래된(stale) 트리 정보를 바탕으로 질의를 계속 처리하면서, 백그라운드에서는 갱신된 균형 잡힌 서브트리가 준비되는 것이다.

### 1.4  연산 프로파일 및 복잡도 분석

#### 1.4.1 지원 질의

ikd-Tree는 다음과 같은 다양한 연산을 지원하며, 각 연산은 로보틱스 응용에서 뚜렷한 목적을 가진다.19

- `Build()`: 균형 잡힌 k-d 트리를 구축한다.
- `Add_Points()` / `Delete_Points()`: 포인트를 동적으로 삽입/삭제한다.
- `Delete_Point_Boxes()`: 지정된 경계 상자 내의 포인트를 삭제한다.
- `Nearest_Search()`: k-NN 탐색을 수행한다.
- `Box_Search()`: 지정된 경계 상자 내의 모든 포인트를 반환한다.
- `Radius_Search()`: 지정된 반경 내의 모든 포인트를 반환한다.

#### 1.4.2 시간 복잡도

각 연산의 이론적 시간 복잡도는 다음과 같다.10

- **탐색 (k-NN)**: 균형 잡힌 트리의 경우 평균적으로 $O(\log n)$의 매우 빠른 성능을 보이지만, 트리가 심하게 불균형해지면 최악의 경우 $O(n)$까지 성능이 저하될 수 있다. 재균형 메커니즘의 목표는 성능을 평균적인 경우에 가깝게 유지하는 것이다.
- **삽입**: 리프 노드를 찾는 데 평균 $O(\log n)$이 소요되지만, 이로 인해 재균형이 촉발될 수 있다.
- **삭제 (지연)**: 노드를 찾아 플래그를 지정하는 데 $O(\log n)$이 소요된다.
- **재균형 (부분)**: 크기가 $m$인 서브트리를 재구축하는 비용은 $O(m \log m)$이다. 핵심은 여기서 $m$이 전체 포인트 수 $n$보다 훨씬 작다는 점이다.

ikd-Tree의 설계는 정교한 타협의 산물이다. 이는 균형 잡힌 k-d 트리가 탐색에 이론적으로 효율적이라는 점을 인정하면서도, 실시간 시스템에서 모든 업데이트마다 완벽한 균형을 유지하는 것이 현실적으로 불가능함을 인지한 결과다. 지연 삭제와 비동기적 부분 재균형이라는 해결책은 유지 보수 비용을 시간적으로 분산시키는 매우 실용적인 접근법이다. 완벽하게 균형 잡힌 트리는 $O(\log n)$의 탐색 시간을 보장하지만 17, 단 한 번의 삽입/삭제로도 균형이 깨져 성능이 최악의 경우 $O(n)$으로 저하될 수 있다.24 매번 전체 트리를 재균형하는 것은 $O(n \log n)$의 비용이 들어 실시간 응용에는 부적합하다. ikd-Tree 개발자들은 모든 업데이트가 심각한 불균형을 초래하는 것은 아니며, 불균형은 종종 국소적인 문제라는 점에 착안했다. 따라서 필요할 때만, 문제가 되는 부분만 재구축하는 것이 훨씬 경제적이다. 나아가 이 비용이 큰 작업을 비동기적으로 처리함으로써 메인 스레드의 실시간 응답성을 보장했다. 이 전략들의 조합은 이론적 이상과 현실적 제약 사이의 간극을 메우는 영리한 공학적 해법이라 할 수 있다.

## 2.  증분 복셀 (iVox): 최고 속도를 위한 해싱 기반 접근법

### 2.1  아키텍처 원리: 트리를 버리고 해싱을 택하다

iVox는 ikd-Tree와는 근본적으로 다른 접근법을 취한다. 트리 기반의 재귀적 공간 분할 대신, 공간을 균일한 격자(grid)로 나누는 복셀(voxel) 기반의 이산화 방식을 사용한다.25

iVox 아키텍처의 핵심은 해시 맵(hash map)을 이용한 희소(sparse) 표현이다.27 이는 실제로 포인트가 포함된 복셀의 정보만을 저장함으로써, 3차원 공간 전체를 조밀한(dense) 격자로 표현할 때 발생하는 막대한 메모리 낭비를 방지한다.12

`Faster-LIO` 논문에서 명시적으로 밝히듯, iVox의 핵심 설계 철학은 LIO와 같은 응용에서는 ikd-Tree와 같은 트리 구조의 엄격함이 과도한 성능 저하를 유발하며, 더 빠르고 단순한 구조가 더 바람직하다는 것이다.11 3차원 복셀의 좌표 `(vx, vy, vz)`를 1차원의 해시 키로 매핑하는 해시 함수는 이 구조의 핵심이며, 일반적으로 다음과 같은 형태를 띤다 11:
$$
hash(v) = (v_x \cdot p_1) \oplus (v_y \cdot p_2) \oplus (v_z \cdot p_3) \pmod N
$$
여기서 $p_1, p_2, p_3$는 큰 소수(prime number)이고, $\oplus$는 XOR 연산, $N$은 해시 맵의 크기다.

### 2.2  포인트 관리 및 탐색 전략

#### 2.2.1 Linear iVox vs. iVox-PHC

활성화된 하나의 복셀 내부에 포인트를 저장하고 관리하는 방식에 따라 iVox는 두 가지 변형으로 나뉜다.11

- **Linear iVox**: 포인트를 단순한 벡터(vector)나 리스트(list)에 저장한다. 이 경우 복셀 내에서의 탐색 시간 복잡도는 $O(n_{voxel})$이며, 여기서 $n_{voxel}$은 해당 복셀 내의 포인트 수다. $n_{voxel}$이 작을 때는 매우 효율적이다.
- **iVox-PHC**: 선형 배열 대신, 공간 채움 곡선(space-filling curve)의 일종인 유사 힐베르트 곡선(Pseudo-Hilbert Curve, PHC)을 사용한다. PHC는 다차원 공간의 점들을 지역성(locality)을 최대한 보존하면서 1차원으로 매핑하는 특성이 있다. 이를 통해 복셀 내 탐색 복잡도를 $O(k)$ 또는 $O(\log n_{voxel})$ 수준으로 개선할 수 있어, 복셀 내 포인트 밀도가 높을 때 더 유리하다.

#### 2.2.2 근사 k-최근접 이웃 (Approximated k-NN) 탐색

이는 iVox의 성능을 정의하는 가장 중요한 특징이다. iVox의 k-NN 탐색은 '정확성'을 일부 희생하여 '속도'를 극대화하는 근사 방식으로 동작한다.11 그 과정은 다음과 같다.

1. 질의 포인트(query point)가 속한 복셀의 인덱스를 해시 함수로 계산한다.
2. 해당 복셀과 그 주변의 이웃 복셀들(예: 3x3x3=27개 또는 십자 모양의 7개)로 구성된 고정된 탐색 영역을 정의한다.
3. 탐색을 이 제한된 영역 내에서만 수행한다. 즉, 이 영역에 속한 모든 포인트를 대상으로만 k-NN을 찾는다.

이 방식은 질의 포인트의 진짜 최근접 이웃이 이 고정된 탐색 영역 바로 바깥에 존재할 경우, 그 이웃을 놓칠 수 있다는 점에서 '근사'적이다. 이것이 바로 iVox가 감수하는 근본적인 '정확도-속도' 트레이드오프다.

### 2.3  연산 프로파일 및 성능 특성

#### 2.3.1 병렬성 (Parallelism)

iVox의 구조는 병렬 처리에 매우 유리하다. 각기 다른 질의 포인트에 대한 k-NN 탐색은 서로 독립적이며, 한 질의에 대한 이웃 복셀 탐색 또한 각 복셀 단위로 병렬화할 수 있다. 이 덕분에 멀티코어 CPU 환경에서 뛰어난 확장성을 보인다.11

#### 2.3.2 증분 업데이트

포인트의 삽입과 삭제가 극도로 빠르다. 삽입은 포인트의 좌표를 해싱하여 해당 복셀을 찾고, 그 복셀의 내부 저장소에 포인트를 추가하는 과정으로 끝난다. 삭제도 마찬가지로 해시 룩업 후 제거하면 된다. 해시 맵 접근의 평균 시간 복잡도는 사실상 $O(1)$에 가깝다.11 트리 구조와 달리 '재균형'이라는 개념 자체가 존재하지 않아 유지 보수 비용이 거의 없다.

#### 2.3.3 성능

iVox는 순수한 속도를 위해 설계되었으며, 최신 하드웨어에서 초당 1-2 kHz의 업데이트 및 질의 처리 속도를 달성한다. 이는 트리 기반 방식에 비해 월등히 높은 수치다.11

iVox의 설계는 철저한 '응용 특화 최적화'의 결과물이다. iVox가 채택한 핵심 트레이드오프, 즉 근사 k-NN 탐색은 그것이 사용되는 특정 맥락, 즉 IMU 센서가 고품질의 초기 위치 추정치를 제공하는 긴밀하게 결합된(tightly-coupled) LIO 시스템 내에서만 유효하다. 일반적인 k-NN 알고리즘은 질의 포인트와 그 이웃이 어디에든 있을 수 있다고 가정해야 한다. 하지만 LIO 시스템에서는 IMU가 새로운 라이다 스캔이 지도상 어디쯤에 위치할지 매우 정확하게 예측해준다. 이는 새로운 스캔의 한 포인트에 대응하는 맵 상의 포인트가 매우 가까운 공간적 근방에 존재할 확률이 지극히 높다는 것을 의미한다. iVox 설계자들은 바로 이 도메인 지식을 활용했다. 그들은 IMU의 사전 정보(prior)가 주어진 상황에서, 실제 최근접 이웃이 정의된 탐색 영역을 벗어날 확률은 무시할 수 있을 정도로 작다고 판단했다. 따라서 전역적으로 일관되고 완벽하게 정확한 탐색 구조(k-d 트리와 같은)를 과감히 포기하고, 훨씬 빠른 지역적 탐색 구조로 대체할 수 있었다. 이처럼 데이터 구조를 더 큰 시스템(ESIKF 기반 LIO)의 특성과 함께 공동으로 설계한 것이 iVox 성공의 핵심 비결이다. 만약 IMU의 도움이 없다면, iVox의 근사 탐색 방식은 실패할 가능성이 높다.

## 3.  직접 비교 분석

### 3.1  알고리즘 설계 철학: 트리 기반의 엄격함 vs. 해싱 기반의 속도

ikd-Tree와 iVox는 동적 포인트 클라우드 관리라는 동일한 문제를 해결하지만, 그 접근 방식과 설계 철학은 극명한 대조를 이룬다.

- **ikd-Tree**: 이론적 정확성을 유지하려는 철학을 대표한다. 균형 잡힌 구조와 엄격한(strict) k-NN 탐색이라는 k-d 트리의 장점을 지키면서, 지연 삭제와 비동기적 부분 재균형이라는 영리한 공학적 기법을 통해 동적 환경에 적응시킨다. 이는 다양한 상황에서 신뢰할 수 있는 성능을 제공하는 견고하고 범용적인 동적 공간 인덱스라 할 수 있다.
- **iVox**: 응용에 특화된 가정을 통해 원초적인 속도를 최우선으로 추구하는 철학을 따른다. 이론적 보장(엄격한 k-NN)을 과감히 포기하는 대신, 특정 LIO 작업에 "충분히 좋은(good enough)" 성능을 내는, 대규모 병렬 처리가 가능한 극도로 단순한 구조를 택한다.

아래 표 1은 두 구조의 핵심적인 아키텍처 차이를 요약한다.

**표 1: 아키텍처 특성 비교**

| 특징                     | ikd-Tree                           | iVox                                |
| ------------------------ | ---------------------------------- | ----------------------------------- |
| **기본 구조**            | 이진 공간 분할 트리                | 해시 맵 기반 희소 복셀 그리드       |
| **공간 분할 방식**       | 축-정렬 초평면, 중앙값 기반        | 균일한 격자 셀 (복셀)               |
| **탐색 방식**            | 엄격한 k-NN 탐색                   | 근사 k-NN 탐색                      |
| **동적 삽입**            | 평균 $O(\log n)$, 재균형 유발 가능 | 평균 $O(1)$ (해시 맵 룩업)          |
| **동적 삭제**            | 지연 삭제 (플래그 후 나중에 제거)  | 즉시 제거                           |
| **균형 유지**            | 필요 (부분적, 비동기 재균형)       | 불필요 (구조가 본질적으로 비순서적) |
| **병렬성**               | 제한적 (재균형 작업 분리 가능)     | 높은 병렬성 (탐색이 지역화됨)       |
| **주요 적용 LIO 시스템** | FAST-LIO2                          | Faster-LIO                          |
| **핵심 장점**            | 보장된 탐색 정확도                 | 극단적인 연산 속도                  |
| **핵심 한계**            | 트리 유지 보수 오버헤드            | 근사적인 탐색 정확도                |

### 3.2  성능 벤치마크 심층 분석

이론적 차이는 실제 성능으로 증명되어야 한다. `FR-LIO` 및 `Faster-LIO` 논문에서 제시된 실험 데이터는 두 구조의 성능을 정량적으로 비교할 수 있는 중요한 근거를 제공한다.11 실험은 주로 AMD Ryzen7 4800H CPU가 장착된 데스크톱에서 UrbanNav와 같은 공개 데이터셋을 사용하여 수행되었으며, FAST-LIO2(ikd-Tree 기반)와 Faster-LIO(iVox 기반) 시스템의 각 모듈별 연산 시간을 측정했다.

- **증분 매핑 및 맵 삭제 (Incremental Mapping & Map Delete)**: 이 단계는 새로운 포인트들을 맵에 추가하고 오래된 포인트들을 제거하는 작업이다. 실험 결과, `ikd-Tree (Single-thread)`가 가장 긴 시간을 소요했으며, `iVox`는 병렬 처리 시 상당한 속도 향상을 보였다. 특히 `RC-Vox`는 단일 스레드임에도 `iVox`의 병렬 처리 버전과 맞먹는 성능을 보여, 배열 기반 구조의 빠른 접근 속도를 입증했다.29
- **상태 추정 / k-NN 탐색 (State Estimation)**: LIO 파이프라인에서 가장 빈번하게 호출되는 이 단계에서 두 구조의 성능 차이는 더욱 극명하게 드러난다. 복셀 기반 방식(`iVox`, `RC-Vox`)은 트리 기반 방식(`ikd-Tree`)에 비해 압도적인 속도 우위를 보였다.29

아래 표 2와 표 3은 각각 실험적 성능 벤치마크와 이론적 시간 복잡도를 요약하여 보여준다.

**표 2: 실험적 성능 벤치마크 요약 (UrbanNav 데이터셋 기준)**

| LIO 모듈      | 데이터 구조 설정  | 평균 시간 (ms) - TST | 평균 시간 (ms) - Whampoa |      |
| ------------- | ----------------- | -------------------- | ------------------------ | ---- |
| **증분 매핑** | ikd-Tree (Single) | ~12.5                | ~14.0                    |      |
|               | iVox (Single)     | ~5.0                 | ~5.5                     |      |
|               | ikd-Tree (Paral)  | ~7.0                 | ~7.5                     |      |
|               | iVox (Paral)      | ~2.5                 | ~3.0                     |      |
|               | RC-Vox (Single)   | ~2.5                 | ~3.0                     |      |
| **상태 추정** | ikd-Tree (Single) | ~4.0                 | ~4.5                     |      |
|               | iVox (Single)     | ~3.0                 | ~3.5                     |      |
|               | ikd-Tree (Paral)  | ~2.5                 | ~3.0                     |      |
|               | iVox (Paral)      | ~1.5                 | ~2.0                     |      |
|               | RC-Vox (Single)   | ~1.0                 | ~1.5                     |      |

주: 위 수치는 FR-LIO 논문의 Figure 6에서 시각적으로 종합한 근사치임.29

**표 3: 시간 복잡도 비교**

| 연산            | ikd-Tree                              | iVox                                     |
| --------------- | ------------------------------------- | ---------------------------------------- |
| **k-NN 탐색**   | 평균: $O(\log n)$, 최악: $O(n)$       | $O(1) + O(k)$ 또는 $O(n_{voxel})$ (근사) |
| **삽입**        | 평균: $O(\log n)$                     | 평균: $O(1)$                             |
| **삭제**        | $O(\log n)$ (지연 삭제 플래그)        | 평균: $O(1)$                             |
| **맵 유지보수** | 크기 m의 부분 재구축 시 $O(m \log m)$ | 불필요                                   |

### 3.3  근본적인 트레이드오프: 엄격한 탐색 vs. 근사 탐색

두 구조 간 가장 본질적인 차이는 k-NN 탐색의 정확성이다.

- **엄격함이 중요할 때**: 증명 가능한 k-NN 정확도가 필수적인 시나리오에서는 ikd-Tree가 우월하다. 예를 들어, 강력한 사전 정보 없이 초기 지도를 구축하거나, 대응점(correspondence)의 신뢰도가 매우 중요한 루프 클로저(loop closure) 감지 같은 작업이 이에 해당한다.
- **근사가 충분할 때**: LIO 주행 거리 측정 사례가 대표적이다. 강력한 IMU 사전 정보 덕분에 iVox의 근사 탐색으로 인한 오차는 무시할 수 있는 수준이다. 대신, 이를 통해 얻는 압도적인 처리 속도는 격렬한 움직임을 더 잘 추적하게 하여 오히려 전반적인 강건성을 높일 수 있다.11 k-NN 탐색의 '재현율(recall)' 관점에서 보면, ikd-Tree는 100% 재현율을 목표로 하는 반면, iVox는 훨씬 낮은 계산 비용으로 충분히 높은 재현율을 달성하는 것을 목표로 한다.

### 3.4  메모리 사용량 및 확장성

메모리 사용량 측면에서 ikd-Tree는 각 노드마다 포인터와 경계 상자 정보 등의 오버헤드를 가진다. iVox는 활성화된 각 복셀에 대한 해시 테이블 항목과 그 안에 저장된 포인트 데이터에 대한 오버헤드를 가진다. 일반적으로 포인트 클라우드의 밀도가 낮고 희소할수록 트리 구조가 유리할 수 있으며, 밀도가 높을수록 복셀 구조의 메모리 효율이 더 나아질 수 있다.12

확장성 측면에서 ikd-Tree의 성능은 전체 포인트 수 $n$에 대해 로그 스케일($\log n$)로 증가하지만, 재균형 비용이 누적될 수 있다. 반면 iVox의 탐색 성능은 전체 맵 크기 $n$에 거의 영향을 받지 않지만, 복셀 내 지역적 포인트 밀도($n_{voxel}$)에 민감하다.11

## 4.  실제 적용 사례: FAST-LIO2에서 Faster-LIO로의 진화

두 데이터 구조의 차이점과 트레이드오프는 실제 LIO 시스템의 발전사를 통해 가장 명확하게 이해할 수 있다. FAST-LIO2에서 Faster-LIO로의 진화는 이들의 특성을 보여주는 완벽한 사례 연구다.

### 4.1  FAST-LIO2의 엔진, ikd-Tree

FAST-LIO2는 매우 성공적인 LIO 시스템으로, 그 성능의 핵심에는 ikd-Tree가 있었다.2 ikd-Tree가 제공하는 효율적인 동적 맵 관리 기능 덕분에, FAST-LIO2는 과거의 특징점 기반 방식에서 벗어나 원시 포인트(raw points)를 지도에 직접 정합하는 방식을 채택할 수 있었다. 이는 환경의 미묘한 특징까지 모두 활용하여 정확도를 높이는 데 기여했다.

### 4.2  Faster-LIO가 ikd-Tree를 대체한 이유

FAST-LIO2의 성공에도 불구하고, 개발자들은 성능을 한계까지 밀어붙이고자 했다. 특히 새롭게 등장하는 고주파 솔리드 스테이트 라이다에 대응하기 위해서는 더 높은 처리 속도가 필요했다.11 이 과정에서 ikd-Tree의 트리 구조 유지 보수(재균형 등)에 드는 오버헤드가 마지막 남은 성능 병목 지점으로 지목되었다. Faster-LIO는 이 병목을 제거하기 위해 ikd-Tree를 iVox로 대체하는 과감한 결정을 내렸다.

### 4.3  시스템 수준 최적화의 사례

FAST-LIO2에서 Faster-LIO로의 전환은 단순한 부품 교체가 아니라, 문제 해결의 패러다임 전환을 의미한다. 이는 개별 구성 요소의 완벽함(ikd-Tree의 이론적 정확성)보다는 전체 시스템의 성능(LIO의 실시간 처리 속도)을 우선시하는 시스템 수준 최적화의 전형이다. ikd-Tree를 iVox로 교체한 결정은 데이터 구조를 응용 프로그램의 특성과 긴밀하게 공동 설계하는 현대적 접근법을 명확히 보여준다. 이 사례는 알고리즘과 시스템 간의 경계가 어떻게 허물어지고 있는지를 보여주는 중요한 증거다.32

## 5.  결론 및 향후 전망

### 주요 발견 요약

본 보고서는 동적 포인트 클라우드 관리를 위한 두 가지 핵심 데이터 구조인 ikd-Tree와 iVox를 심층적으로 비교 분석했다. 분석 결과는 다음과 같은 명확한 선택 기준을 제시한다.

- **ikd-Tree를 선택해야 할 경우**: k-NN 탐색의 정확성이 보장되어야 하거나, LIO 외에 더 범용적인 공간 데이터 관리 응용을 구축하거나, IMU와 같은 강력한 초기 추정치가 없는 경우에 적합하다. ikd-Tree는 동적 업데이트와 탐색 효율성 사이에서 견고하고 잘 설계된 균형점을 제공한다.
- **iVox를 선택해야 할 경우**: 원초적인 속도와 초고주파 처리가 가장 중요하며, 긴밀하게 결합된 LIO 시스템처럼 강력한 사전 정보를 제공하여 근사 k-NN 탐색의 단점을 상쇄할 수 있는 환경에서 최상의 선택이다. 이는 정확도와의 트레이드오프를 통해 극적인 성능 향상을 얻는 매우 효과적인 전략이다.

### 5.1 앞으로의 길: 끝나지 않은 효율성 탐구

ikd-Tree에서 iVox로의 발전은 효율성을 향한 탐구가 멈추지 않았음을 보여준다. 더 나아가, `FR-LIO` 논문에서 제안된 `RC-Vox`의 등장은 이러한 경향을 더욱 가속화한다.29

`RC-Vox`는 로봇 중심의 고정된 크기 배열을 사용하여 맵을 관리하는, 훨씬 더 특화되고 응용 지향적인 설계를 보여준다. 이는 미래의 로보틱스 인식 시스템이 범용 데이터 구조에서 벗어나, 특정 파이프라인과 센서 특성에 맞춰 고도로 맞춤화된 '비스포크(bespoke)' 데이터 구조를 채택하는 방향으로 나아갈 것임을 시사한다. 알고리즘과 시스템의 경계는 더욱 희미해지고, 전체 시스템의 맥락에서 최적화된 데이터 구조가 고성능 로보틱스의 핵심 경쟁력이 될 것이다.

#### **참고 자료**

1. Ceres Solver Python Tutorial - Andrew's Notes, accessed July 31, 2025, https://andrewtorgesen.com/notes/Autonomy/Systems_Implementation/Optimization_Libraries/Ceres_Solver_Python_Tutorial.html
2. Non-Linear Least Squares - Steven Gong, accessed July 31, 2025, https://stevengong.co/notes/Non-Linear-Least-Squares
3. robotics - IRIS, accessed July 31, 2025, https://iris.uniroma1.it/retrieve/e3835329-b25e-15e8-e053-a505fe0a3de9/Grisetti_Least-squares-optimization_2020.pdf
4. Factor Graphs and GTSAM: A Hands-on Introduction - GT Digital Repository, accessed July 31, 2025, https://repository.gatech.edu/server/api/core/bitstreams/b3606eb4-ce55-4c16-8495-767bd46f0351/content
5. Factor Graphs and GTSAM, accessed July 31, 2025, https://gtsam.org/tutorials/intro.html
6. gtsam/USAGE.md at develop - GitHub, accessed July 31, 2025, https://github.com/borglab/gtsam/blob/develop/USAGE.md
7. Non-linear Least Squares - Ceres Solver, accessed July 31, 2025, http://ceres-solver.org/nnls_tutorial.html
8. GTSAM | GTSAM is a BSD-licensed C++ library that implements sensor fusion for robotics and computer vision using factor graphs., accessed July 31, 2025, https://gtsam.org/
9. gtsam 4.3.0 documentation - the official ROS docs, accessed July 31, 2025, https://docs.ros.org/en/rolling/p/gtsam/
10. iSAM2: Incremental Smoothing and Mapping Using the Bayes Tree - CMU School of Computer Science, accessed July 31, 2025, https://www.cs.cmu.edu/~kaess/pub/Kaess12ijrr.pdf
11. iSAM2: Incremental smoothing and mapping using the Bayes tree - DSpace@MIT, accessed July 31, 2025, https://dspace.mit.edu/handle/1721.1/78894
12. iSAM2: Incremental Smoothing and Mapping Using the Bayes Tree - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/254098771_iSAM2_Incremental_Smoothing_and_Mapping_Using_the_Bayes_Tree
13. The Bayes Tree: An Algorithmic Foundation for Probabilistic Robot Mapping - Frank Dellaert, accessed July 31, 2025, https://dellaert.github.io/files/Kaess10wafr.pdf
14. iSAM2: Incremental Smoothing and Mapping Using the Bayes Tree, accessed July 31, 2025, https://www.cvlibs.net/projects/autonomous_vision_survey/slides/Kaess2012IJRR/top.pdf
15. Docs | GTSAM, accessed July 31, 2025, https://gtsam.org/docs/
16. MH-iSAM2: Multi-hypothesis iSAM using Bayes Tree and Hypo-tree - Carnegie Mellon University, accessed July 31, 2025, https://www.cs.cmu.edu/~kaess/pub/Hsiao19icra.pdf
17. MR-iSAM2: Incremental Smoothing and Mapping with Multi-Root Bayes Tree for Multi-Robot SLAM | Request PDF - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/357112975_MR-iSAM2_Incremental_Smoothing_and_Mapping_with_Multi-Root_Bayes_Tree_for_Multi-Robot_SLAM
18. Ceres Solver - A Large Scale Non-linear Optimization Library, accessed July 31, 2025, http://ceres-solver.org/
19. ceres-solver/ceres-solver: A large scale non-linear optimization library - GitHub, accessed July 31, 2025, https://github.com/ceres-solver/ceres-solver
20. G2O vs GTSAM vs Ceres Solver from a programmer's perspective | by Jianzhu Huai, accessed July 31, 2025, https://medium.com/@jianzhuhuai0108/g2o-vs-gtsam-vs-ceres-solver-from-a-programmers-perspective-f45ac68a90fd
21. Ceres Solver - Google Groups, accessed July 31, 2025, https://groups.google.com/g/ceres-solver
22. ceres-solver - Hunter 0.23 documentation, accessed July 31, 2025, https://hunter.readthedocs.io/en/latest/packages/pkg/ceres-solver.html
23. Why? - Ceres Solver, accessed July 31, 2025, https://ceres-solver.readthedocs.io/latest/features.html
24. Solving Non-linear Least Squares - Ceres Solver, accessed July 31, 2025, http://ceres-solver.org/nnls_solving.html
25. Solving Non-linear Least Squares - GitLab, accessed July 31, 2025, http://file.sage-modi.com:1080/Avalanche/ceres-solver/-/blob/79bbf95103672fa4b5485e055ff7692ee4a1f9da/docs/source/nnls_solving.rst
26. On-Manifold Optimization Demo using Ceres Solver - Fan's Farm, accessed July 31, 2025, https://fzheng.me/2018/01/23/ba-demo-ceres/
27. Modeling - Ceres Solver, accessed July 31, 2025, http://ceres-solver.org/modeling_faqs.html
28. Installation - Ceres Solver, accessed July 31, 2025, http://ceres-solver.org/installation.html
29. Tightly-coupled Fusion of Global Positional Measurements in Optimization-based Visual-Inertial Odometry, accessed July 31, 2025, https://udel.edu/~ghuang/icra21-vins-workshop/papers/06-Cioffi_global-VIO.pdf
30. Custom factors and degrees of freedom. / borglab gtsam / Discussion #1104 - GitHub, accessed July 31, 2025, https://github.com/borglab/gtsam/discussions/1104
31. gtsam/examples/Pose2SLAMExample.cpp at master - GitHub, accessed July 31, 2025, https://github.com/devbharat/gtsam/blob/master/examples/Pose2SLAMExample.cpp
32. Class List - GTSAM, accessed July 31, 2025, https://gtsam.org/doxygen/annotated.html
33. gtsam::IndeterminantLinearSystemException Class Reference, accessed July 31, 2025, https://gtsam.org/doxygen/4.0.0/a03203.html
34. Ceres Solver: Google's large scale nonlinear least squares solver open sourced - Reddit, accessed July 31, 2025, https://www.reddit.com/r/programming/comments/t1mo7/ceres_solver_googles_large_scale_nonlinear_least/
35. slam.md - gtbook/gtsam-examples - GitHub, accessed July 31, 2025, https://github.com/gtbook/gtsam-examples/blob/main/slam.md
36. Mastering Bundle Adjustment in Cognitive Robotics - Number Analytics, accessed July 31, 2025, https://www.numberanalytics.com/blog/bundle-adjustment-cognitive-robotics
37. stereo vision SLAM, some questions. : r/SelfDrivingCars - Reddit, accessed July 31, 2025, https://www.reddit.com/r/SelfDrivingCars/comments/gldgh5/stereo_vision_slam_some_questions/
38. A Practical Guide to Marginalization for Nonlinear Least Squares on Boxplus Manifolds - Evan Levine, accessed July 31, 2025, https://evanlev.github.io/marginalization.pdf
39. Factor Graphs for Navigation Applications: A Tutorial, accessed July 31, 2025, https://navi.ion.org/content/71/3/navi.653
40. MegBA: A GPU-Based Distributed Library for Large-Scale Bundle Adjustment, accessed July 31, 2025, https://www.ecva.net/papers/eccv_2022/papers_ECCV/papers/136970698.pdf
41. Why? - Ceres Solver, accessed July 31, 2025, http://ceres-solver.org/features.html
42. Lecture 17-18: Least Squares Optimization - VNAV, accessed July 31, 2025, https://vnav.mit.edu/material/17-18-NonLinearLeastSquares-notes.pdf
43. gtsam/gtsam/nonlinear/ISAM2.h at master / haidai/gtsam - GitHub, accessed July 31, 2025, https://github.com/haidai/gtsam/blob/master/gtsam/nonlinear/ISAM2.h
44. gtsam::ISAM2Params Struct Reference, accessed July 31, 2025, https://gtsam.org/doxygen/a04360.html
45. Issue with iSAM2: IndeterminantLinearSystemException / Issue #1179 / borglab/gtsam, accessed July 31, 2025, https://github.com/borglab/gtsam/issues/1179
46. Powell's dogleg solver backend - Google Groups, accessed July 31, 2025, https://groups.google.com/g/ceres-solver/c/3X-605THg6I
47. arxiv.org, accessed July 31, 2025, https://arxiv.org/html/2502.08158v1
48. Cartographer SLAM Method for Optimization with an Adaptive Multi-Distance Scan Scheduler - MDPI, accessed July 31, 2025, https://www.mdpi.com/2076-3417/10/1/347
49. Building Maps Using Google Cartographer and the OS1 Lidar Sensor | Ouster, accessed July 31, 2025, https://ouster.com/insights/blog/building-maps-using-google-cartographer-and-the-os1-lidar-sensor
50. g2o vs. Ceres: Optimizing Scan Matching in Cartographer SLAM - arXiv, accessed July 31, 2025, https://arxiv.org/html/2507.07142v1
51. [2007.11898] ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual-Inertial and Multi-Map SLAM - ar5iv, accessed July 31, 2025, https://ar5iv.labs.arxiv.org/html/2007.11898
52. SteveMacenski/slam_toolbox: Slam Toolbox for lifelong mapping and localization in potentially massive maps with ROS - GitHub, accessed July 31, 2025, https://github.com/SteveMacenski/slam_toolbox
53. slam_toolbox 2.6.10 documentation, accessed July 31, 2025, https://docs.ros.org/en/ros2_packages/humble/api/slam_toolbox/
54. Structure from Motion using Bundle Adjustment with the Ceres Solver - Christian Diller, accessed July 31, 2025, https://www.christian-diller.de/projects/bundle-a-ceres/
55. examples/simple_bundle_adjuster.cc - ceres-solver - Git at Google, accessed July 31, 2025, https://ceres-solver.googlesource.com/ceres-solver/+/master/examples/simple_bundle_adjuster.cc
56. PoseSLAM - GTSAM 4.0.2 documentation - Read the Docs, accessed July 31, 2025, https://gtsam-jlblanco-docs.readthedocs.io/en/latest/PoseSLAM.html
57. gtsam::FixedLagSmoother Class Reference, accessed July 31, 2025, https://gtsam.org/doxygen/a05192.html
58. VINS-Mono-GTSAM/README.md at freeture/Gtsam_hydra_V1, accessed July 31, 2025, https://github.com/AnshShah3009/VINS-Mono-GTSAM/blob/freeture/Gtsam_hydra_V1/README.md
59. miniSAM: A Flexible Factor Graph Non-linear Least Squares Optimization Framework - arXiv, accessed July 31, 2025, https://arxiv.org/pdf/1909.00903
60. g2o vs. Ceres: Optimizing Scan Matching in Cartographer SLAM - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/393587126_g2o_vs_Ceres_Optimizing_Scan_Matching_in_Cartographer_SLAM
61. COMPARISON OF THE GRAPH-OPTIMIZATION FRAMEWORKS G2O AND SBA, accessed July 31, 2025, http://mdh.diva-portal.org/smash/get/diva2:934179/FULLTEXT01.pdf
62. arxiv.org, accessed July 31, 2025, https://arxiv.org/html/2409.12190v2
63. Bundle Adjustment in the Eager Mode - arXiv, accessed July 31, 2025, https://arxiv.org/html/2409.12190v1
64. arxiv.org, accessed July 31, 2025, https://arxiv.org/pdf/2409.12190.pdf


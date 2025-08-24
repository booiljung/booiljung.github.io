[iVox](./index.md)
# 해시 기반 증분 복셀 데이터 구조 iVox에 대한 고찰


동시적 위치 추정 및 지도 작성(Simultaneous Localization and Mapping, SLAM) 기술은 자율주행 자동차, 로봇, 증강현실 등 다양한 분야의 핵심 기반 기술로 자리 잡았다.1 이 중 LiDAR-관성 주행 거리 측정(Lidar-Inertial Odometry, LIO)은 LiDAR 센서의 정밀한 거리 측정 능력과 관성 측정 장치(IMU)의 높은 시간 해상도를 결합하여, 조명 변화나 특징이 부족한 환경에서도 강건하고 정확한 위치 추정을 가능하게 한다.3 성공적인 LIO 시스템은 정확성(accuracy), 강건성(robustness), 그리고 효율성(efficiency)이라는 세 가지 요소를 모두 만족해야 한다.3 특히, 초당 수십만에서 수백만 개의 점을 생성하는 고속 회전형(spinning) 및 고체 상태(solid-state) LiDAR가 보편화되면서, 입력되는 방대한 양의 데이터를 실시간으로 처리하는 효율성은 시스템의 성패를 가르는 결정적인 요소다.5

LIO 파이프라인에서 가장 큰 계산 부하를 유발하는 병목 지점 중 하나는 바로 점군 맵을 효율적으로 관리하고, 현재 스캔과 맵 사이의 대응점(correspondences)을 찾는 공간 자료구조(spatial data structure) 부분이다.3 LOAM, LeGO-LOAM과 같은 초기 LIO 알고리즘에서 널리 사용된 k-d 트리(k-dimensional tree)는 정적인 데이터셋에서는 빠른 최근접 이웃 탐색을 지원하지만, 로봇의 움직임에 따라 맵이 지속적으로 변하는 동적 환경에서는 치명적인 단점을 드러낸다. 새로운 점군이 추가되거나 오래된 점군이 삭제될 때마다 트리 전체를 재구성해야 하므로 막대한 계산 비용이 발생하기 때문이다.6 이러한 문제를 해결하기 위해 FAST-LIO2에서는 증분 업데이트와 동적 재균형을 지원하는 `ikd-Tree`를 도입하여 실시간 맵 관리에 큰 진전을 이뤘다.4 하지만 `ikd-Tree` 역시 트리 구조의 본질적인 탐색 및 유지보수 비용에서 완전히 자유롭지는 못하며, 특히 초고주파수 LiDAR 데이터를 처리하는 데에는 새로운 한계에 직면하게 된다.

이러한 배경에서 등장한 Faster-LIO는 FAST-LIO2를 기반으로 하면서도, `ikd-Tree`를 `iVox`(incremental Voxel)라는 혁신적인 해시 기반 자료구조로 대체함으로써 처리 속도를 1.5배에서 2배까지 획기적으로 향상시키는 것을 목표로 한다.5 `iVox`는 복잡한 트리 구조나 엄격한 k-최근접 이웃(k-NN) 질의 대신, 해시 테이블을 이용한 증분 복셀 관리 기법을 통해 더 빠른 맵 업데이트와 근사 k-NN 탐색을 구현한다.4

본 보고서는 Faster-LIO의 심장이라 할 수 있는 `iVox` 데이터 구조를 심층적으로 고찰하는 것을 목적으로 한다. `iVox`의 설계 철학부터 내부 구현 메커니즘, 시스템 전체와의 상호작용, 그리고 성능적 장단점에 이르기까지 다각도로 분석한다. 이를 통해 LIO 시스템에서 고속 처리를 위한 공간 자료구조 설계의 패러다임이 어떻게 변화하고 있는지 이해하고, `iVox`가 이룬 기술적 기여와 그 한계를 명확히 밝힌다.


Faster-LIO의 핵심은 `iVox`라는 효율적인 자료구조를 강결합(Tightly-Coupled) LIO 아키텍처와 유기적으로 결합한 데 있다. `iVox`의 역할을 이해하기 위해서는 먼저 Faster-LIO의 전체적인 구조를 파악해야 한다.


Faster-LIO는 LiDAR와 IMU 센서 데이터를 강결합 방식으로 융합한다. 이는 각 센서에서 나온 원시 측정값이나 추출된 특징점을 단일 최적화 문제의 제약 조건으로 함께 사용하여 두 센서 간의 상호 보완성을 극대화하는 접근 방식이다.2 예를 들어, IMU 데이터는 높은 시간 해상도를 바탕으로 LiDAR 점군 데이터에 포함된 모션 왜곡(motion distortion)을 효과적으로 보정해준다. 반대로, LiDAR 측정값은 드리프트가 누적되는 IMU의 바이어스(bias)를 지속적으로 추정하고 보정하는 데 사용된다. 이처럼 두 센서의 정보를 긴밀하게 엮음으로써, 시스템 전체의 정확성과 공격적인 움직임에 대한 강건성이 크게 향상된다.2


상태 추정을 위한 핵심 엔진으로는 반복 오차 상태 칼만 필터(Iterated Error State Kalman Filter, IEKF)가 사용된다. IEKF는 비선형 시스템에 대한 확장 칼만 필터(EKF)의 선형화 오차를 줄이기 위해, 측정값에 기반한 상태 보정(update) 단계를 여러 번 반복하는 기법이다.4 시스템의 상태 벡터 $x$는 일반적으로 로봇의 자세(회전 $R$, 위치 $p$), 속도 $v$, 그리고 IMU 센서의 가속도계 바이어스 $b_a$와 자이로스코프 바이어스 $b_g$ 등을 포함한다.

IEKF의 전체 프로세스는 크게 두 단계로 나뉜다:

1. **예측 (Prediction):** 이전 상태와 새로운 IMU 측정값을 이용해 현재 시점의 상태를 예측한다.
2. **보정 (Update):** 예측된 상태와 새로운 LiDAR 측정값 사이의 오차(잔차, residual)를 계산하고, 이 오차를 최소화하도록 상태를 보정한다.

`iVox`는 바로 이 두 번째 단계인 '보정' 과정에서 결정적인 역할을 수행한다.


IEKF의 보정 단계는 현재 LiDAR 스캔의 각 점에 대해, 이전에 구축된 맵에서 가장 잘 대응되는 지점 또는 평면을 찾는 것으로부터 시작된다. 이 대응 관계를 설정하기 위해서는 각 점의 주변에 있는 'k-최근접 이웃(k-NN)' 점들을 맵에서 신속하게 검색해야 한다.5 `iVox`는 바로 이 k-NN 검색을 기존의 트리 기반 방식보다 훨씬 빠르게 수행하여, IEKF 보정 단계의 계산 효율성을 극대화한다. 결국 `iVox`의 성능이 Faster-LIO 시스템 전체의 처리율(throughput)과 실시간성을 결정짓는 핵심 열쇠가 된다.

Faster-LIO의 설계는 LIO 시스템 개발의 무게 중심이 '정확성'에서 '실시간 효율성'으로 이동하고 있음을 명확히 보여준다. FAST-LIO2가 `ikd-Tree`를 통해 동적 환경에서의 정확한 맵 업데이트에 큰 진전을 이뤘다면, Faster-LIO는 여기서 한 걸음 더 나아가 `iVox`를 도입했다. 이는 단순히 기존 구조를 개선하는 차원을 넘어선다. 센서 기술의 발전으로 LiDAR의 데이터 출력 주파수는 기하급수적으로 증가하고 있다.5 이러한 초고주파수 데이터를 실시간으로 처리하기 위해서는 `ikd-Tree`의 증분 업데이트 및 재균형 연산조차 병목이 될 수 있다. 점군 수 $N$이 증가함에 따라 `log(N)`의 복잡도를 갖는 트리 탐색 비용은 더 이상 무시할 수 없는 부담이 되기 때문이다. 따라서 Faster-LIO는 트리 구조 자체를 과감히 포기하고, 평균적으로 상수 시간 복잡도 `O(1)`를 갖는 해시 기반 접근법을 채택했다. 이는 알고리즘의 복잡도 상수를 낮추고 병렬 처리를 극대화함으로써, 다가오는 하드웨어 트렌드에 선제적으로 대응하려는 전략적 선택이다. 이는 단순한 성능 개선을 넘어, LIO 아키텍처의 패러다임을 전환하려는 시도로 해석될 수 있다.


`iVox`의 혁신성은 기존 공간 분할 자료구조의 한계를 명확히 인식하고, 이를 해시(hash)라는 다른 패러다임으로 해결하려는 시도에서 비롯된다.


- **k-d 트리:** 정적인 데이터셋의 최근접 이웃 검색에는 이론적으로 효율적이지만, LIO와 같이 데이터가 끊임없이 추가되고 삭제되는 동적 환경에는 적합하지 않다. 점의 삽입과 삭제가 반복되면 트리의 균형이 쉽게 깨지고, 이를 재조정(rebalancing)하는 연산은 예측 불가능한 시간 지연을 유발하여 실시간성을 해친다. `ikd-Tree`는 이 문제를 완화했지만, 여전히 복잡한 재균형 로직이 필요하며 근본적인 트리 구조의 한계를 내포한다.6
- **Octree:** 공간을 규칙적으로 8분할하므로 k-d 트리보다 구조가 간단하고 구현이 용이하다. 하지만 점군 데이터의 분포가 불균일할 경우(예: 한쪽 벽면에만 점이 밀집) 트리의 깊이가 한쪽으로만 깊어져 탐색 효율이 급격히 저하될 수 있다.13 동적 업데이트에 대한 부담 역시 k-d 트리와 유사하게 존재한다.

핵심 문제는 LIO에서 관리하는 맵이 고정된 전역 맵이 아니라, 로봇의 움직임을 따라 함께 이동하는 '슬라이딩 윈도우(sliding window)' 형태의 동적 데이터셋이라는 점이다. 트리 기반 구조는 이러한 데이터의 '증분성(incrementality)'과 '휘발성(volatility)'을 효율적으로 처리하는 데 근본적인 어려움을 겪는다.


`iVox`는 이러한 문제를 해결하기 위해 트리 구조 대신 복셀 해싱(Voxel Hashing) 기법을 채택했다.15 그 원리는 다음과 같다.

1. 3차원 공간을 일정한 크기(`s`)의 정육면체 그리드, 즉 복셀(voxel)로 분할한다.

2. 임의의 3D 점 `p = [px, py, pz]^T`가 주어지면, 이 점이 속한 복셀의 3D 정수 좌표 `v = floor((1/s) * p)`를 계산한다.

3. 이 3D 정수 좌표 `(vx, vy, vz)`를 고유한 1차원 정수 값, 즉 해시 키(hash key)로 매핑한다.

4. 이 해시 키를 `std::unordered_map`과 같은 해시 테이블의 키로 사용하여, 해당 복셀에 저장된 점군 데이터에 평균 `O(1)`의 시간 복잡도로 즉시 접근한다.5 이는 트리의 루트부터 리프까지 순회해야 하는 

   `O(log N)` 복잡도보다 이론적으로 훨씬 빠르다.

Faster-LIO에서 사용하는 구체적인 해시 함수는 다음과 같이 주어진다 5:
$$
id_v=hash(v)=((v_x \cdot n_x)⊕(v_y \cdot n_y)⊕(v_z \cdot n_z))\mod N
$$
여기서 $v_x, v_y, v_z$는 복셀의 정수 좌표, $n_x, n_y, n_z$는 미리 정해진 큰 소수(large prime numbers), $\oplus$는 비트 단위 XOR 연산, 그리고 $N$은 해시 맵의 크기(버킷 수)를 의미한다. 이 함수는 간단하면서도 입력 좌표를 해시 테이블 전체에 비교적 고르게 분산시키는 효과가 있다.


`iVox`는 복셀 해싱 원리를 바탕으로 LIO 환경에 최적화된 몇 가지 핵심 요소를 결합하여 구현된다.

- **희소 증분 복셀 (Sparse Incremental Voxels):** LiDAR 데이터는 3차원 공간 전체를 채우는 것이 아니라, 물체 표면에만 희소하게 존재한다. `iVox`는 이 특성을 활용하여 점이 하나 이상 포함된 복셀의 정보만을 해시 맵에 저장한다.5 이를 통해 대부분이 비어 있는 공간에 대한 불필요한 메모리 할당을 막고, 메모리 사용량을 획기적으로 줄인다.15
- **증분 추가 (Incremental Addition):** 새로운 점이 입력되면, 해당 점이 속한 복셀에 간단히 추가된다. 하지만 하나의 복셀에 점이 무한정 누적되는 것을 방지하기 위해, VoxelGrid 필터와 유사한 다운샘플링 전략이 적용된다. 즉, 복셀 내에 이미 다른 점이 특정 반경(leaf size) 내에 존재한다면 새로운 점은 추가되지 않는다. 이 `leaf_size` 파라미터는 맵의 정밀도와 계산 속도 사이의 중요한 트레이드오프를 조절하는 역할을 한다.5
- **증분 삭제와 LRU 캐시 (Incremental Deletion and LRU Cache):** `iVox` 설계의 가장 큰 백미는 맵의 동적 관리를 위한 증분 삭제 메커니즘이다. `iVox`는 LRU(Least Recently Used) 캐시 정책을 도입하여 이 문제를 매우 우아하고 효율적으로 해결한다.3
  1. 해시 맵과는 별도로, 최근에 접근(삽입 또는 조회)된 복셀의 해시 키를 저장하는 큐(queue)를 유지한다.
  2. 새로운 복셀이 추가되어 맵의 총 복셀 수가 미리 정해진 임계값(capacity)을 초과하면, 큐에서 가장 오래된, 즉 가장 오랫동안 사용되지 않은 복셀의 키를 가져온다.4
  3. 이 키를 사용하여 해시 맵에서 해당 복셀을 삭제한다.

이 방식의 장점은 명확하다. 해시 맵에서의 삽입과 삭제 연산은 평균적으로 `O(1)`의 시간 복잡도를 가지므로, 오래된 복셀을 제거하는 과정이 매우 저렴하다.5 이는 트리 전체를 순회하며 삭제할 노드를 찾고, 삭제 후 트리의 구조를 재조정해야 하는 기존 방식에 비해 압도적으로 효율적이다.

`iVox`의 핵심 혁신은 '삭제' 연산에 대한 관점의 전환에 있다. 기존 트리 구조에서 '삭제'는 구조의 무결성을 해칠 수 있는 비싸고 복잡한 연산이었다. 하지만 `iVox`는 LRU 캐시와 해시 맵을 결합함으로써 '삭제'를 시스템의 상태를 유지하기 위한 자연스럽고 가벼운 '만료(expiration)' 과정으로 재정의했다. LIO의 맵은 본질적으로 로봇 주변의 최신 정보를 담는 '슬라이딩 윈도우'이며, `iVox`의 LRU 기반 만료 정책은 이러한 모델과 완벽하게 부합한다. 이 설계를 통해 맵의 크기를 일정하게 유지하면서(bounded memory) 예측 가능한 실시간 성능을 보장할 수 있게 됐으며, 이는 실시간 임베디드 시스템 설계에서 매우 중요한 특성이다.


`iVox`는 해시 맵을 통해 특정 복셀에 대한 빠른 접근(`O(1)`)을 제공하지만, 이것만으로는 k-NN 탐색 문제가 완전히 해결되지 않는다. 만약 하나의 복셀 내에 수십, 수백 개의 점이 밀집되어 있다면, 이 점들 사이에서 최근접 이웃을 찾는 과정이 또 다른 성능 병목이 될 수 있다. Faster-LIO는 이 '복셀 내 탐색' 문제를 해결하기 위해, 데이터의 밀도에 따라 선택할 수 있는 두 가지 대안적인 내부 구조를 제안한다.5


Linear iVox는 가장 직관적이고 간단한 구현 방식이다. 각 복셀은 내부에 `std::vector`와 같은 선형 배열 컨테이너를 가지고, 해당 복셀에 속한 모든 점들을 이 컨테이너에 순차적으로 저장한다.

- **k-NN 탐색 방식:** 탐색 대상 점이 주어지면, 해당 복셀과 그 주변 복셀들에 접근한 뒤, 각 복셀 내의 모든 점들과의 거리를 일일이 계산(brute-force search)하여 가장 가까운 k개를 찾는다.
- **장점:** 구현이 매우 간단하며, 복셀 내 점의 개수가 적은 희소한(sparse) 환경에서는 추가적인 자료구조 오버헤드가 없어 매우 효율적이다.5
- **단점:** 복셀 내 점의 개수가 많아지는 조밀한(dense) 환경에서는 k-NN 탐색 비용이 점의 개수에 비례하여 선형적으로 증가하므로 성능이 급격히 저하된다.


iVox-PHC는 조밀한 복셀 내에서의 탐색 성능을 개선하기 위해, 공간 채움 곡선(space-filling curve)의 일종인 유사 힐버트 곡선(Pseudo-Hilbert Curve, PHC)을 도입한 구조다.5

- **원리:**
  1. 하나의 복셀을 $K$차 PHC를 이용해 가로, 세로, 높이 각각 $2^K$개씩, 총 $2^{3K}$개의 더 작은 가상 큐브(cube)로 세분화한다.5
  2. 3차원 큐브의 상대 좌표 $x$를 1차원 정수 인덱스 $t$로 매핑하는 힐버트 곡선 함수 $H(x) = t$와 그 역함수 $H^{-1}(t) = x$를 정의한다.5
  3. 힐버트 곡선은 다차원 공간에서의 인접성(locality)을 1차원 인덱스 상의 인접성으로 최대한 보존하는 뛰어난 특성을 가진다. 즉, 3D 공간에서 서로 가까이 있는 점들은 1D 힐버트 인덱스 상에서도 가까운 값을 가질 확률이 매우 높다.
- **k-NN 탐색 방식:**
  - 탐색 대상 점 $p$가 속한 복셀 내 가상 큐브의 힐버트 인덱스 $t = H(x)$를 계산한다.
  - 전체 큐브를 모두 탐색하는 대신, $t$를 중심으로 앞뒤로 일정 범위($\rho$) 내의 인덱스에 해당하는 큐브들만 방문하여 k-NN 후보를 찾는다. 이를 통해 탐색 공간을 극적으로 줄일 수 있다.
  - 탐색 범위 $\rho$는 원하는 탐색 반경 $r$과 복셀 크기 $l$에 따라 근사적으로 결정될 수 있다. 논문에 제시된 식은 $\rho \approx 8 \cdot 2^{\lceil \log_2(r/l) \rceil}$이다.5
- **장점:** 복셀 내 점의 개수가 많을 때, 탐색 공간을 효과적으로 가지치기하여 Linear iVox보다 월등히 뛰어난 계산 효율성을 보인다.5
- **단점:** 구현이 상대적으로 복잡하며, 점의 개수가 적을 때는 PHC 구조를 유지하고 인덱스를 계산하는 오버헤드로 인해 오히려 Linear iVox보다 느릴 수 있다. 또한, 힐버트 곡선이 보존하는 인접성은 완벽하지 않기 때문에, 이 방법으로 찾은 k-NN은 엄밀한 의미의 최적해가 아닌 근사치(approximated)다. 논문에 따르면 두 점 A, B 사이의 실제 유클리드 거리 $d$와 PHC 상의 거리 $l$은 $d \le l \le \sqrt{3}d$의 관계를 가진다.5

Linear iVox와 iVox-PHC의 존재는 `iVox`가 단일 알고리즘이 아니라, 데이터의 통계적 특성(밀도)에 적응하는 '하이브리드 전략'을 채택했음을 시사한다. 이는 "하나의 은탄환은 없다(No Silver Bullet)"는 컴퓨터 과학의 오랜 격언을 LIO 시스템 설계에 현명하게 적용한 사례다. LiDAR 스캔 데이터는 본질적으로 매우 불균일하다. 넓고 평평한 벽이나 바닥은 점들이 조밀하게 분포하는 반면, 허공이나 물체의 가장자리는 희소하게 분포한다. 만약 Linear iVox만 사용한다면 조밀한 영역에서 k-NN 탐색이 병목이 될 것이고, iVox-PHC만 사용한다면 희소한 영역에서 불필요한 인덱싱 오버헤드가 발생할 것이다. Faster-LIO 개발자들은 컴파일 타임에 사용자가 `iVox` 타입을 선택할 수 있도록 옵션(`-DWITH_IVOX_NODE_TYPE_PHC=ON`)을 제공함으로써 8, 각자의 주행 환경(예: 좁은 실내 복도 vs. 넓은 야외 광장)이나 사용하는 LiDAR 센서의 해상도에 맞춰 최적의 자료구조를 선택할 수 있도록 하는 실용적인 해결책을 제시했다. 이는 런타임에 복셀 별 점의 개수를 기준으로 동적으로 두 방식을 전환하는 '적응형 iVox'로의 발전 가능성까지 내포하는 유연한 설계 철학이라 할 수 있다.


`iVox`의 진정한 가치는 IEKF의 측정 업데이트 과정과 결합될 때 발현된다. `iVox`의 빠른 탐색 능력은 IEKF의 반복적인 상태 추정 과정을 실시간으로 가능하게 하는 원동력이다.


IEKF의 각 보정 단계(iteration)가 시작되면, 현재 입력된 LiDAR 프레임의 모든 점 $p$에 대해 `iVox`를 사용하여 맵에서 가장 가까운 $k$개의 이웃 점들을 찾는다 (일반적으로 $k=5$ 사용).3 이 과정은 다음과 같이 진행된다.

1. 점 $p$가 속한 복셀의 해시 키를 계산하여 해시 맵에서 해당 복셀에 즉시 접근한다.
2. 해당 복셀과 그 주변의 이웃 복셀들(예: 3x3x3 또는 5x5x5 영역)을 탐색 범위로 설정한다.
3. 탐색 범위 내의 모든 점들을 k-NN 후보군으로 수집하고, 그중에서 점 $p$와 가장 가까운 $k$개의 점을 최종적으로 선택한다.

이 탐색 과정은 각 점 $p$에 대해 독립적으로 수행될 수 있으므로 병렬 처리에 매우 용이하다. Faster-LIO는 이 특성을 활용하여 여러 개의 스레드로 수많은 점에 대한 k-NN 탐색을 동시에 수행함으로써 전체 처리 시간을 크게 단축한다.5


`iVox`를 통해 찾은 $k$개의 이웃 점들은 현재 점 $p$가 놓여있어야 할 지역 평면(local plane)을 모델링하는 데 사용된다.7

- **지역 평면 모델링:** 이 평면은 $k$개 이웃 점들의 통계적 분포로부터 근사된다.

  - 평면의 중심점(centroid) $\bar{p}$는 $k$개 이웃 점들의 산술 평균으로 계산된다.
  - 평면의 법선 벡터(normal vector) $\mathbf{n}$은 $k$개 이웃 점들의 3x3 공분산 행렬(covariance matrix)을 계산한 후, 주성분 분석(PCA)을 통해 가장 작은 고유값(eigenvalue)에 해당하는 고유벡터(eigenvector)로 결정된다. 이 고유벡터가 바로 평면에 수직인 방향을 나타낸다.7

- Welford's Algorithm을 이용한 통계량의 증분 업데이트:

  LIO 시스템에서는 새로운 점이 복셀에 계속 추가된다. 이때마다 복셀 내 모든 점을 이용해 평균과 공분산을 처음부터 다시 계산하는 것은 매우 비효율적이다. 특히, 부동소수점 연산 시 큰 값에서 큰 값을 빼는 과정에서 발생하는 수치적 불안정성(catastrophic cancellation) 문제로 인해 공분산 계산 결과가 부정확해질 수 있다.17

  Welford's algorithm은 이러한 문제를 해결하는 효과적인 방법이다. 이 알고리즘을 사용하면 기존의 평균 $\bar{p}_{n-1}$과 제곱합의 편차 $M2_{n-1}$만 저장하고 있으면, 새로운 점 $x_n$이 추가되었을 때의 새로운 평균 $\bar{p}_n$과 $M2_n$을 수치적으로 안정적이면서도 증분적으로(incrementally) 계산할 수 있다.17 1차원 데이터에 대한 Welford's 알고리즘의 업데이트 규칙은 다음과 같다.
  $$
  δ = x_n - \bar p_{n-1}
  \\
  \bar p_{n} = \bar p_{n-1} + \frac{δ}{n}
  \\
  M2_n = M2_{n-1} + δ(x_n - \bar p_n)
  $$
  이 알고리즘은 다차원으로 자연스럽게 확장되어 공분산 행렬의 증분 업데이트에도 적용될 수 있으며, `iVox`가 복셀 내의 통계 정보를 효율적으로 유지하는 데 필수적인 기술적 기반을 제공한다.

- **잔차(Residual) 수식:** 잔차 $r$은 현재 점 $p$를 현재 추정된 로봇의 자세(변환 행렬 $T$)를 이용해 월드 좌표계로 변환한 점 $T \cdot p$와, 이웃 점들로 모델링된 평면 $(\mathbf{n}, \bar{p})$ 사이의 최단 거리, 즉 점-평면 거리로 정의된다.
  $$
  r=n^T(T⋅p − \bar p)
  $$
  이 잔차 값은 현재 추정된 자세 $T$가 얼마나 부정확한지를 나타내는 정량적인 척도가 된다.7


계산된 잔차 $r$과, 상태 벡터 $x$에 대한 잔차의 자코비안 행렬 $H$를 이용하여 칼만 필터의 핵심인 칼만 게인 $K$를 계산한다. 이 칼만 게인은 예측된 상태와 측정값 중 어느 쪽을 더 신뢰할지를 결정하는 가중치 역할을 한다. 최종적으로 칼만 게인을 사용하여 예측된 상태(prior state)를 보정하고, 새로운 사후 상태(posterior state)와 그 불확실성을 나타내는 공분산 $P$를 업데이트한다.9
$$
K = PH^T (HPH^T + R) ^ {-1}
\\
\hat x _\text{posteriro} = \hat x _\text{prior} + K \cdot r
\\
P_\text{posteriro} = (I - KH)P_\text{prior}
$$
여기서 $R$은 측정 노이즈 공분산으로, LiDAR 센서 자체의 측정 오차나 점군 분포의 불확실성 등을 반영하여 설정된다.21

IEKF는 이 업데이트 과정을 여러 번 반복(iterate)하면서,  반복마다 더 정확해진 상태 추정치를 바탕으로 점 $p$와 평면의 대응 관계를 재설정하고 상태를 더욱 정밀하게 다듬는다.12

`iVox`의 압도적으로 빠른 k-NN 탐색 능력 덕분에, 제한된 시간(예: 100ms) 내에 이 복잡한 반복 과정을 여러 번 수행하는 것이 가능해진다.

`iVox`는 '근사(Approximation)'를 적극적으로 수용함으로써 IEKF와의 시너지를 극대화한다. `iVox`가 제공하는 k-NN은 `ikd-Tree`처럼 엄밀한 의미의 최근접 이웃이 아닐 수 있다.5 하지만 이는 문제가 되지 않는다. IEKF의 작동 방식 자체가 한 번에 완벽한 해를 찾는 것이 아니라, 초기 추정치로부터 시작하여 점진적으로 오차를 줄여나가는 '반복적 정제(iterative refinement)' 과정이기 때문이다.12 첫 번째 반복에서 k-NN 탐색은 다소 부정확한 자세 추정치를 기반으로 수행되므로, 여기서 `ikd-Tree`를 사용해 완벽한 k-NN을 찾는 것은 오히려 계산력 낭비일 수 있다. `iVox`는 매우 빠르게 '그럴듯한' 이웃들을 찾아주고, IEKF는 이를 기반으로 1차 보정을 수행한다. 보정된 자세는 이전보다 더 정확해지고, 두 번째 반복에서는 이 더 정확해진 자세를 가지고 다시 k-NN을 탐색한다. 이 과정이 반복되면서 자세 추정치와 k-NN의 정확도가 함께 수렴해 나간다. 결국, `iVox`의 '빠른 근사'와 IEKF의 '반복적 정제'가 서로의 단점을 보완하며 전체 시스템의 성능을 끌어올리는 이상적인 공생 관계를 형성한다.


`iVox`의 우수성은 정량적 실험 결과와 타 자료구조와의 정성적 비교를 통해 명확히 드러난다.


Faster-LIO 논문은 `iVox`가 기존 `ikd-Tree`에 비해 월등한 속도 향상을 보임을 다양한 실험을 통해 증명했다.5

- **처리 속도:**
  - Livox Avia와 같은 고체 상태 LiDAR를 사용했을 때, `iVox`는 초당 1000-2000 프레임(Hz)에 달하는 경이로운 처리 속도를 달성했다.5
  - Velodyne VLP-32C와 같은 32선 회전형 LiDAR 환경에서도 200 Hz 이상의 안정적인 처리 속도를 보여줬다.5
  - 이는 동일한 조건에서 FAST-LIO2 대비 약 1.5배에서 2배 빠른 속도에 해당한다.8
- **병렬 처리 효과:** `iVox`는 k-NN 탐색 과정이 점별로 독립적이라는 특성 덕분에 병렬화에 매우 유리하다. 논문에 따르면, 병렬 처리된 `iVox`는 `ikd-Tree`의 성능을 완전히 능가한다.3 FR-LIO 논문에 제시된 속도 테스트 그래프에서도 병렬화된 Faster-LIO(paral)가 병렬화된 FAST-LIO2(paral)보다 일관되게 빠른 성능을 보이는 것을 확인할 수 있다 [24, Fig. 1(g)].

아래 표는 `iVox`와 `ikd-Tree`의 성능을 정량적으로 비교한 것이다.

**표 1: iVox와 ikd-Tree의 정량적 성능 비교**

| 자료구조 (Data Structure) | 센서 종류 (Sensor Type)   | k-NN 탐색 및 맵 업데이트 속도 (Hz) | 비고 (Notes)        | 출처 (Source) |
| ------------------------- | ------------------------- | ---------------------------------- | ------------------- | ------------- |
| `ikd-Tree` (in FAST-LIO2) | Spinning (e.g., 10Hz)     | ~100 Hz (approx.)                  | 기준 성능           | 4             |
| `iVox` (in Faster-LIO)    | Spinning (32 lines)       | > 200 Hz                           | 약 2배 성능 향상    | 5             |
| `ikd-Tree` (in FAST-LIO2) | Solid-state (e.g., 100Hz) | ~100 Hz                            | -                   | 4             |
| `iVox` (in Faster-LIO)    | Solid-state (Avia)        | 1000 - 2000 Hz                     | 10배 이상 성능 향상 | 5             |


`iVox`의 특징을 더 명확히 이해하기 위해 전통적인 공간 자료구조인 k-d 트리, Octree와 여러 측면에서 비교 분석할 수 있다.

**표 2: SLAM을 위한 공간 자료구조 비교 분석**

| 특성 (Characteristic)  | k-d Tree                                                  | Octree                                                       | Voxel Hashing (e.g., iVox)                                |
| ---------------------- | --------------------------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------------- |
| **기본 원리**          | 데이터 분포에 따라 공간을 비대칭적으로 분할하는 이진 트리 | 공간을 8개의 균등한 하위 공간으로 재귀적으로 분할            | 공간을 균등한 복셀로 분할하고 3D 좌표를 1D 해시 키로 매핑 |
| **삽입/삭제 시간**     | 비효율적 (`O(log N)` + 재균형 비용)                       | 비효율적 (`O(log N)` + 재구성 비용)                          | **매우 효율적 (평균 `O(1)`)**                             |
| **k-NN 질의 시간**     | 효율적 (평균 `O(log N)`)                                  | 효율적 (평균 `O(log N)`)                                     | **매우 효율적 (평균 `O(1)` + 이웃 탐색)**                 |
| **메모리 사용량**      | 데이터 수에 비례                                          | 데이터 수와 공간 범위에 비례, 희소할 경우 비효율적일 수 있음 | **사용 중인 복셀 수에만 비례 (희소)**                     |
| **동적 데이터 처리**   | 약함 (재균형 문제)                                        | 약함 (재구성 문제)                                           | **강함 (LRU 캐시를 통한 효율적 관리)**                    |
| **데이터 분포 민감도** | 강함 (불균일 분포에 적응적)                               | 약함 (불균일 분포에 비효율적 트리 생성 가능)                 | 약함 (해시 함수가 분포를 고르게 만들어 줌)                |
| **구현 복잡도**        | 높음 (재균형 로직 포함)                                   | 중간                                                         | 중간 (해시 함수 및 충돌 처리 필요)                        |

이 표는 `iVox`가 특히 동적 데이터의 삽입/삭제와 k-NN 질의 속도 측면에서 기존 트리 구조 대비 압도적인 우위를 가짐을 명확히 보여준다. 이는 LIO 시스템의 실시간성 요구사항에 `iVox`가 얼마나 부합하는지를 잘 설명한다.14


모든 기술과 마찬가지로 `iVox` 역시 한계를 가지고 있다.

- **해시 충돌 (Hash Collisions):** 해시 테이블의 근본적인 문제점으로, 서로 다른 3D 복셀 좌표가 우연히 동일한 해시 키로 매핑되는 현상이다. 충돌이 발생하면 해당 버킷에 여러 복셀 정보가 연결 리스트 형태로 저장되어 탐색 성능이 저하될 수 있다. 이는 해시 맵의 크기($N$)와 해시 함수의 품질에 따라 그 영향이 달라진다.3
- **파라미터 민감성 (Parameter Sensitivity):** `iVox`의 성능은 `voxel_size`라는 핵심 파라미터에 매우 민감하다. 복셀 크기가 너무 크면 서로 다른 평면이나 구조물이 하나의 복셀에 뭉개져 특징을 구분할 수 없게 되어 정확도가 떨어진다. 반대로 너무 작으면 메모리 사용량이 급증하고 맵이 과도하게 희소해져 안정적인 k-NN 탐색이 어려워질 수 있다. 따라서 사용자는 자신의 주행 환경과 센서 특성에 맞게 이 값을 신중하게 튜닝해야 하는 부담이 있다.8

이러한 `iVox`의 한계를 개선하려는 시도로 후속 연구인 FR-LIO에서는 `RC-Vox`(Robocentric Voxel Map)라는 새로운 구조를 제안했다.3

`RC-Vox`는 해시 맵 대신 로봇 중심의 고정된 크기를 갖는 2계층 3D 배열(two-layer 3D array)을 사용한다. 로컬 맵 포인트를 모듈로(modulo) 연산을 통해 이 고정 배열에 매핑함으로써, 해시 충돌 문제를 원천적으로 제거하고 배열 구조가 제공하는 진정한 `O(1)` 접근 속도를 최대한 활용한다. 이는 `iVox`의 해시 기반 접근법에서 한 단계 더 나아가, 더욱 빠르고 예측 가능한 성능을 추구하는 LIO 공간 자료구조의 진화 방향을 보여준다.3


`iVox`는 LiDAR-관성 주행 거리 측정(LIO) 분야에서 실시간 성능의 한계를 돌파하기 위한 중요한 기술적 이정표를 제시했다. 본 보고서에서 심층적으로 분석한 `iVox`의 기여와 의의는 다음과 같이 요약할 수 있다.

- **iVox의 핵심 기여:** `iVox`는 해시 맵과 LRU 캐시라는 두 가지 검증된 컴퓨터 과학 기술을 영리하게 결합하여, LIO 시스템의 동적인 맵 관리 문제를 매우 효율적으로 해결했다. 이는 기존 트리 기반 자료구조가 안고 있던 증분 업데이트의 고질적인 병목 현상을 제거하고, 병렬 처리에 최적화된 구조를 제공함으로써 특히 고주파수 LiDAR 센서를 사용하는 현대적 로보틱스 환경에서 전례 없는 수준의 처리 속도를 가능하게 했다. 또한, `iVox`는 '완벽한 정확성'을 고집하는 대신 '충분히 좋은 근사'를 '매우 빠르게' 제공하고, 이를 IEKF의 반복적 최적화 과정과 유기적으로 결합함으로써 시스템 전체의 성능을 끌어올리는 성공적인 엔지니어링 절충안을 보여줬다.
- **실용적 제언:** `iVox` 기반의 LIO 시스템을 구축하거나 튜닝할 때, 개발자는 다음 두 가지를 핵심적으로 고려해야 한다. 첫째, `voxel_size` 파라미터는 시스템의 정확도와 안정성에 지대한 영향을 미치므로, 대상 환경의 구조적 복잡성(예: 넓은 야외 vs. 좁은 실내)과 LiDAR 센서의 점 밀도를 고려하여 신중하게 실험적으로 결정해야 한다.8 둘째, 예상되는 점군 데이터의 밀도에 따라 Linear iVox와 iVox-PHC 중 적절한 버전을 선택해야 한다. 일반적으로 조밀한 특징이 많은 실내 환경에서는 iVox-PHC가, 상대적으로 희소한 야외 환경에서는 Linear iVox가 유리할 수 있다.5
- **미래 전망:** `iVox`와 그 후속 연구인 `RC-Vox`의 등장은 LIO의 공간 자료구조 설계 패러다임이 복잡한 트리 구조에서 단순하고 규칙적인 해시 테이블 및 고정 배열 구조로 진화하고 있음을 명확히 보여준다. 이러한 그리드/배열 기반 자료구조는 GPU나 FPGA와 같은 병렬 처리 하드웨어와의 친화성이 매우 높다. 따라서 미래의 LIO 시스템은 하드웨어 가속을 더욱 적극적으로 활용하여, 이러한 자료구조의 병렬성을 극한까지 끌어내는 방향으로 발전할 것으로 예상된다. 이러한 거대한 흐름 속에서, `iVox`는 LIO 기술의 대중화와 상용화를 앞당긴 중요한 패러다임 전환의 촉매제로 평가된다.


1. What is SLAM? A Beginner to Expert Guide - Kodifly, accessed July 31, 2025, https://kodifly.com/what-is-slam-a-beginner-to-expert-guide
2. Research on 3D LiDAR outdoor SLAM algorithm based on LiDAR/IMU tight coupling - PMC, accessed July 31, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11962141/
3. FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled ..., accessed July 31, 2025, https://arxiv.org/pdf/2302.04031
4. RESEARCH ON HIGH-FREQUENCY ODOMETRY IN LASER SLAM USING POINT- SUBMAP MATCHING, accessed July 31, 2025, https://isprs-archives.copernicus.org/articles/XLVIII-4-W10-2024/29/2024/isprs-archives-XLVIII-4-W10-2024-29-2024.pdf
5. (PDF) Faster-LIO: Lightweight Tightly Coupled Lidar-Inertial ..., accessed July 31, 2025, https://www.researchgate.net/publication/359662313_Faster-LIO_Lightweight_Tightly_Coupled_Lidar-Inertial_Odometry_Using_Parallel_Sparse_Incremental_Voxels
6. LIO-GVM: an Accurate, Tightly-Coupled Lidar-Inertial Odometry with Gaussian Voxel Map, accessed July 31, 2025, https://arxiv.org/html/2306.17436v3
7. DV-LIO: LiDAR-inertial Odometry based on dynamic merging and smoothing voxel | Robotica | Cambridge Core, accessed July 31, 2025, https://www.cambridge.org/core/product/8337A33B5012947E273B5EC96D30A42D/core-reader
8. gaoxiang12/faster-lio: Faster-LIO: Lightweight Tightly ... - GitHub, accessed July 31, 2025, https://github.com/gaoxiang12/faster-lio
9. [2010.08196] FAST-LIO: A Fast, Robust LiDAR-inertial Odometry Package by Tightly-Coupled Iterated Kalman Filter - arXiv, accessed July 31, 2025, https://arxiv.org/abs/2010.08196
10. VOX-LIO: An Effective and Robust LiDAR-Inertial Odometry System Based on Surfel Voxels, accessed July 31, 2025, https://www.mdpi.com/2072-4292/17/13/2214
11. LIO-SAM - Autoware Documentation, accessed July 31, 2025, https://autowarefoundation.github.io/autoware-documentation/main/how-to-guides/integrating-autoware/creating-maps/open-source-slam/lio-sam/
12. LIWO: Lidar-Inertial-Wheel Odometry - arXiv, accessed July 31, 2025, https://arxiv.org/pdf/2302.14298
13. A Hybrid Spatial Indexing Structure of Massive Point Cloud Based on Octree and 3D R*-Tree - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/355248871_A_Hybrid_Spatial_Indexing_Structure_of_Massive_Point_Cloud_Based_on_Octree_and_3D_R-Tree
14. Why specifically are k-d trees preferred in ray tracing and octrees in collision? - Reddit, accessed July 31, 2025, https://www.reddit.com/r/gameenginedevs/comments/1789f54/why_specifically_are_kd_trees_preferred_in_ray/
15. Efficient Octree-Based Volumetric SLAM Supporting Signed-Distance and Occupancy Mapping - Department of Computing, accessed July 31, 2025, https://www.doc.ic.ac.uk/~sleutene/publications/EVespaRAL_final.pdf
16. LIO-GVM: an Accurate, Tightly-Coupled Lidar-Inertial Odometry with Gaussian Voxel Map, accessed July 31, 2025, https://arxiv.org/html/2306.17436v1
17. Algorithms for calculating variance - Wikipedia, accessed July 31, 2025, https://en.wikipedia.org/wiki/Algorithms_for_calculating_variance
18. Welford's method for computing variance - The Mindful Programmer, accessed July 31, 2025, https://jonisalonen.com/2013/deriving-welfords-method-for-computing-variance/
19. Implementing Welford's Algorithm (incremental variance calculation) - MATLAB Answers, accessed July 31, 2025, https://www.mathworks.com/matlabcentral/answers/414314-implementing-welford-s-algorithm-incremental-variance-calculation
20. 3. SLAM Solutions, accessed July 31, 2025, https://www.maxwell.vrac.puc-rio.br/34617/34617_4.PDF
21. LIO-EKF: High Frequency LiDAR-Inertial Odometry using Extended Kalman Filters - arXiv, accessed July 31, 2025, https://arxiv.org/html/2311.09887v2
22. Joint State and Noise Covariance Estimation - arXiv, accessed July 31, 2025, https://arxiv.org/html/2502.04584v1
23. Noise in motion and measurement models - Robotics Stack Exchange, accessed July 31, 2025, https://robotics.stackexchange.com/questions/690/noise-in-motion-and-measurement-models
24. FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - ResearchGate, accessed July 31, 2025, https://www.researchgate.net/publication/368361497_FR-LIO_Fast_and_Robust_Lidar-Inertial_Odometry_by_Tightly-Coupled_Iterated_Kalman_Smoother_and_Robocentric_Voxels
25. Difference between BVH and Octree/K-d trees - Computer Graphics Stack Exchange, accessed July 31, 2025, https://computergraphics.stackexchange.com/questions/7828/difference-between-bvh-and-octree-k-d-trees
26. [2302.04031] FR-LIO: Fast and Robust Lidar-Inertial Odometry by Tightly-Coupled Iterated Kalman Smoother and Robocentric Voxels - arXiv, accessed July 31, 2025, https://arxiv.org/abs/2302.04031


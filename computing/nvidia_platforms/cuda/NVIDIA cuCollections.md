# NVIDIA cuCollections(cuco)


1999년 NVIDIA의 GPU(Graphics Processing Unit) 발명과 2006년 CUDA(Compute Unified Device Architecture) 아키텍처의 공개는 컴퓨팅 패러다임의 지각 변동을 예고했다.1 본래 실시간 그래픽 렌더링을 위해 설계된 GPU는 CUDA의 등장으로 과학 및 공학 연구 분야 전반에서 범용 병렬 컴퓨팅(GPGPU, General-Purpose computing on Graphics Processing Units)의 핵심 동력으로 자리 잡았다.3 수천 개의 코어를 집적한 GPU는 CPU의 순차 처리 방식이 직면한 성능 한계를 돌파할 대안으로 부상하며, 데이터 과학, 인공지능, 고성능 컴퓨팅(HPC) 분야의 폭발적인 성장을 견인했다.5

그러나 GPU가 제공하는 대규모 병렬성(massive parallelism)을 온전히 활용하는 길은 순탄치 않았다. 특히, 데이터의 저장, 검색, 관리를 담당하는 자료구조는 가장 근본적인 장벽으로 작용했다. CPU 환경에 최적화된 전통적인 자료구조, 예를 들어 C++ 표준 라이브러리의 `std::unordered_map` 등은 잦은 코드 분기(branch divergence), 임의 메모리 접근(random memory access) 패턴, 그리고 동기화 오버헤드로 인해 GPU의 병렬 아키텍처에서는 심각한 성능 병목을 유발한다.6 수많은 스레드가 동시에 하나의 자료구조에 접근하려 할 때 발생하는 경합(contention)은 GPU의 잠재력을 억누르고 성능을 저해하는 주된 원인이었다.

이러한 도전을 해결하기 위해 NVIDIA는 GPU 아키텍처에 네이티브하게 최적화된 고성능 동시성 자료구조 라이브러리, `cuCollections`(이하 cuco)를 선보였다.9 cuco는 단순히 개발자에게 편의성을 제공하는 유틸리티를 넘어, NVIDIA의 데이터 사이언스 생태계인 RAPIDS의 근간을 이루는 핵심 구성 요소이다.11 이 라이브러리의 성능은 상위 애플리케이션의 성능과 직결되며, 이는 cuco가 NVIDIA의 소프트웨어 생태계, 즉 'CUDA Moat(경제적 해자)'를 더욱 견고하게 만드는 전략적 자산임을 시사한다. CUDA 생태계는 cuDNN, cuBLAS와 같은 저수준 라이브러리부터 RAPIDS(cuDF)와 같은 고수준 분석 도구에 이르기까지 수직적으로 통합되어 있는데, cuDF의 핵심 연산이 내부적으로 cuco에 의존한다는 사실은 이러한 전략을 명확히 보여준다.2

본 보고서는 cuCollections의 기술적 깊이를 심층적으로 탐구하고, 그 내부 구현 메커니즘과 성능 특성을 분석하며, GPU 가속 컴퓨팅 생태계 전반에 미치는 전략적 중요성을 고찰하고자 한다.



CUDA 프로그래밍 모델의 근간에는 SIMT(Single Instruction, Multiple Thread) 실행 모델이 있다.14 이는 다수의 스레드를 '워프(warp)'라는 단위(일반적으로 32개 스레드)로 묶어, 모든 스레드가 동일한 명령어를 각기 다른 데이터에 대해 동시에 실행하는 방식이다.15 프로그래머는 스레드(thread), 워프(warp), 스레드 블록(thread block, 또는 CTA), 그리고 그리드(grid)로 이어지는 계층적 구조를 통해 대규모 병렬 작업을 체계적으로 구성할 수 있다.15

이러한 SIMT 모델은 데이터 병렬(data-parallel) 작업, 즉 동일한 연산을 대규모 데이터셋의 각 요소에 독립적으로 적용하는 시나리오에서 최고의 효율을 발휘한다. 하지만 워프 내 스레드들이 조건문으로 인해 서로 다른 코드 경로를 실행하게 되는 '분기 발산(branch divergence)'이 발생하면, 일부 스레드들은 다른 스레드들의 실행이 끝날 때까지 유휴 상태로 대기해야 하므로 심각한 성능 저하를 초래한다.19 따라서 GPU 커널을 설계할 때는 분기 발산을 최소화하는 것이 매우 중요하다.


GPU 아키텍처는 성능 특성이 다른 여러 메모리로 구성된 계층적 구조를 가진다. 대표적으로 모든 스레드 블록이 접근할 수 있는 대용량의 전역 메모리(Global Memory)와, 하나의 스레드 블록 내 스레드들만이 공유하며 매우 빠른 접근 속도를 제공하는 공유 메모리(Shared Memory)가 있다.4

전역 메모리는 높은 대역폭을 자랑하지만, 접근 지연 시간(latency) 또한 높다. 이 지연 시간을 상쇄하고 최대 대역폭을 활용하기 위한 핵심 기법이 바로 '병합 메모리 접근(coalesced memory access)'이다. 워프 내 32개의 스레드가 연속된 메모리 주소에 동시에 접근할 경우, GPU 하드웨어는 이 접근들을 하나의 넓은 트랜잭션으로 병합하여 처리함으로써 매우 높은 효율을 달성한다. 반면, 스레드들이 흩어져 있는 메모리 주소에 접근하는 '임의 접근(random access)' 패턴은 다수의 개별적인 메모리 트랜잭션을 유발하여 성능을 심각하게 저하시킨다.8 해시 테이블과 같은 자료구조는 본질적으로 키(key)의 해시값에 따라 임의의 메모리 위치에 접근해야 하므로, 이를 GPU에서 효율적으로 구현하는 것은 매우 도전적인 과제이다.6


수만 개 이상의 스레드가 동시에 하나의 자료구조에 접근하여 데이터를 읽고 쓰는 상황을 상상해 보자. 데이터의 일관성과 무결성을 보장하기 위해서는 정교한 동기화 메커니즘이 필수적이다. 전통적인 CPU 환경에서 사용되는 lock 기반 동기화 방식은 GPU 환경에 부적합하다. 하나의 스레드가 lock을 획득하면 다른 수많은 스레드들이 대기 상태에 빠지게 되는데, 이는 극심한 경합(contention)을 유발하여 GPU의 병렬성을 무력화시키고 사실상 순차 실행과 다름없는 결과를 초래한다.6

따라서 GPU를 위한 동시성 자료구조는 lock을 사용하지 않는(lock-free) 알고리즘이나, 원자적 연산(atomic operation)을 기반으로 경합을 최소화하는 매우 세분화된(fine-grained) 동기화 기법을 채택해야 한다.7 이는 단순히 기존 CPU 알고리즘을 병렬화하는 차원을 넘어선다. GPU용 동시성 자료구조 설계의 핵심은 '알고리즘의 병렬화'가 아니라, SIMT 실행 모델, 워프 단위 동기화, 병합 메모리 접근과 같은 GPU 하드웨어의 엄격한 제약조건에 '알고리즘을 순응'시키는 것이다. CPU용 동시성 라이브러리인 Intel TBB가 소수의 강력한 코어가 각자의 캐시를 통해 저지연 시간으로 메모리에 접근하는 환경을 가정하고 '논리적 동시성' 보장에 초점을 맞추는 것과는 근본적으로 다른 접근 방식이다.20 즉, 문제 해결의 관점이 순수 알고리즘에서 하드웨어-소프트웨어 공동 설계(co-design)로 전환되어야 하며, cuCollections는 바로 이러한 철학 위에서 탄생했다.



cuCollections는 GPU 가속 컴퓨팅 환경의 특수성을 고려하여 설계된 고성능 동시성 자료구조 라이브러리다. 그 설계 철학은 다음과 같은 핵심 원칙에 기반한다.

- **STL-like, Not a Drop-in Replacement:** cuCollections는 Thrust, CUB와 마찬가지로 C++ 표준 템플릿 라이브러리(STL)와 유사한 인터페이스를 제공하여 개발자의 생산성을 높이는 것을 목표로 한다.23 그러나 CPU와 GPU의 근본적인 아키텍처 차이로 인해 `std::unordered_map`을 그대로 대체하는 '드롭인(drop-in)' 방식이 아니다. 대신, GPU의 병렬 아키텍처에 최적화된, 기능적으로 유사한 자료구조를 제공하는 데 초점을 맞춘다.23
- **Header-Only Library:** cuCollections는 헤더 전용 라이브러리로 설계되었다. 이는 사용자가 별도의 라이브러리를 빌드하고 링크하는 복잡한 과정 없이, 단순히 헤더 파일을 프로젝트에 포함하는 것만으로 라이브러리의 모든 기능을 사용할 수 있음을 의미한다. 이는 프로젝트 통합을 매우 용이하게 만든다.23
- **의존성 관리:** CUDA C++ Core Libraries(CCCL)에 대한 의존성을 가지며, 이는 CMake 빌드 시스템을 통해 자동으로 관리된다. 사용자는 의존성 문제를 신경 쓸 필요 없이 cuCollections를 자신의 프로젝트에 손쉽게 통합할 수 있다.23


cuCollections 라이브러리의 핵심은 다양한 요구사항에 맞춰 설계된 고성능 병렬 해시 테이블 변형들이다. 각 자료구조는 키의 중복 허용 여부, 크기의 가변성 등 특정 시나리오에 최적화되어 있다.23

| 자료구조 (Data Structure) | 핵심 설명 (Core Description)              | 주요 특징 (Key Features)                  | 기본 충돌 해결 (Default Collision Resolution) |
| ------------------------- | ----------------------------------------- | ----------------------------------------- | --------------------------------------------- |
| `cuco::static_set`        | 고정 크기의 고유 원소 집합                | 키 중복 불허, 정적 크기                   | Linear Probing                                |
| `cuco::static_map`        | 고정 크기의 고유 키-값 쌍 맵              | 키 중복 불허, 정적 크기                   | Linear Probing                                |
| `cuco::static_multiset`   | 고정 크기의 중복 가능 원소 집합           | 키 중복 허용, 정적 크기                   | Double Hashing                                |
| `cuco::static_multimap`   | 고정 크기의 중복 가능 키-값 쌍 맵         | 키 중복 허용, 정적 크기                   | Double Hashing                                |
| `cuco::dynamic_map`       | 동적으로 크기가 증가하는 맵               | `static_map`들을 연결하여 구현, 동적 크기 | (내부적으로 `static_map`의 전략 따름)         |
| `cuco::bloom_filter`      | 근사적 집합 멤버십 질의를 위한 블룸 필터  | 확률적 자료구조, 공간 효율성              | N/A (해시 기반)                               |
| `cuco::hyperloglog`       | 집합의 카디널리티(고유 원소 수) 근사 추정 | 확률적 자료구조, 스트림 처리              | N/A (해시 기반)                               |


cuCollections는 NVIDIA의 광범위한 GPU 컴퓨팅 생태계에서 매우 중요한 위치를 차지한다. 이는 CUDA-X 라이브러리 컬렉션의 일부로서, 다른 핵심 라이브러리들과 유기적으로 상호작용한다.12

- **하위 계층 (Foundation Layer):** cuCollections는 CUDA C++ Core Libraries(CCCL)의 일부로 분류될 수 있다. CCCL은 Thrust, CUB, libcudacxx와 같은 라이브러리들을 포함하며, CUDA C++ 개발자에게 고성능 병렬 프로그래밍을 위한 기본적인 빌딩 블록을 제공한다. cuCollections는 이 계층에서 고성능 동시성 '자료구조'를 담당하는 핵심 역할을 수행한다.23
- **상위 계층 (Application Layer Support):** cuCollections의 진정한 가치는 RAPIDS와 같은 고수준 데이터 사이언스 프레임워크를 지원하는 데서 드러난다. RAPIDS 스위트의 핵심인 cuDF(GPU 데이터프레임), cuML(머신러닝), cuGraph(그래프 분석)는 내부적으로 복잡한 연산을 수행하기 위해 cuCollections에 크게 의존한다.28 예를 들어, cuDF에서 수행되는 대규모 데이터프레임의 조인(join), 그룹화(group-by), 중복 제거(`drop_duplicates`)와 같은 연산들은 내부적으로 cuCollections의 고성능 해시 테이블을 호출하여 실행된다.11 이는 cuCollections가 단순한 유틸리티를 넘어, 전체 GPU 데이터 사이언스 파이프라인의 성능을 좌우하는 핵심 엔진임을 명확히 보여준다.



해시 테이블은 연관 배열(associative array)을 구현하는 대표적인 자료구조로, 키(key)를 해시 함수(hash function)에 입력하여 얻은 해시값(hash value)을 배열의 인덱스(버킷, bucket)로 사용하여 값을 저장한다. 이를 통해 이론적으로 평균 $O(1)$의 시간 복잡도로 데이터의 삽입 및 조회가 가능하다.9

해시 테이블의 본질적인 특성인 임의 메모리 접근 패턴은 전통적으로 GPU 환경에서 불리하게 작용했지만, 현대 GPU 아키텍처는 이러한 단점을 상쇄하고도 남을 만큼 강력한 이점을 제공한다. 바로 압도적으로 높은 메모리 대역폭이다. NVIDIA GPU는 최신 CPU에 비해 월등히 높은 랜덤 접근 처리량을 보여주며, 이는 수많은 스레드가 동시에 서로 다른 메모리 위치에 접근해야 하는 해시 테이블 연산을 효과적으로 가속할 수 있는 잠재력을 의미한다.9


해시 테이블의 핵심 과제 중 하나는 서로 다른 키가 동일한 해시값을 갖는 '해시 충돌(hash collision)'을 효율적으로 해결하는 것이다. cuCollections는 주로 '개방 주소법(Open Addressing)' 방식을 채택한다. 이는 충돌이 발생했을 때, 데이터를 별도의 자료구조(예: 연결 리스트)에 저장하는 '분리 연결법(Separate Chaining)'과 달리, 해시 테이블 배열 내의 다른 비어있는 버킷을 찾아 데이터를 저장하는 방식이다.9 개방 주소법은 포인터 기반의 자료구조를 사용하지 않으므로 데이터가 물리적으로 인접한 위치에 저장될 가능성이 높아 메모리 접근의 지역성(locality)이 향상되고, 이는 GPU의 캐시 효율을 높이는 데 유리하다.

- **선형 탐사 (Linear Probing):** `cuco::static_map`과 `cuco::static_set`에서 기본으로 사용하는 전략이다.23 충돌이 발생한 버킷 i에서부터 i+1, i+2,... 순서로 순차적으로 비어있는 공간을 탐색한다.9 이 방식은 메모리 접근 패턴이 연속적이므로 캐시 효율이 매우 높다는 장점이 있다. 하지만 데이터가 특정 영역에 집중적으로 몰리는 '기본 클러스터링(primary clustering)' 현상이 발생하기 쉬워, 테이블의 적재율(load factor)이 높아질수록 탐색 길이가 급격히 길어져 성능이 저하될 수 있다.34
- **이중 해싱 (Double Hashing):** `cuco::static_multimap`과 `cuco::static_multiset`에서 기본으로 사용하는 전략이다.23 이 방식은 두 개의 서로 다른 해시 함수를 사용한다. 첫 번째 해시 함수 $h_1(k)$로 초기 위치를 계산하고, 충돌 발생 시 두 번째 해시 함수 $h_2(k)$를 이용해 탐색할 다음 버킷까지의 고유한 간격(interval)을 계산한다. 즉, i번째 탐색 위치는 $(h_1(k) + i \cdot h_2(k)) \pmod{M}$과 같이 결정된다. 이는 클러스터링 현상을 효과적으로 완화하여 더 높은 적재율에서도 비교적 안정적인 성능을 제공한다.32


cuCollections의 성능을 뒷받침하는 핵심 기술은 CUDA의 고급 병렬화 기능을 적극적으로 활용하는 데 있다.

- **Cooperative Groups (CG):** CUDA 9에서 도입된 Cooperative Groups는 기존의 스레드 블록 단위 동기화를 넘어, 블록 내에서 동적으로 스레드 그룹을 구성하고 이 그룹 단위로 동기화 및 협력 연산을 수행할 수 있게 하는 강력한 프로그래밍 모델이다.37 cuCollections는 이 기능을 활용하여 '단일 연산 내부의 미세 병렬화(micro-parallelization)'를 구현한다. 전통적인 병렬 처리가 100만 개의 키 조회를 100만 개의 스레드에 각각 할당하는 방식이었다면, cuCollections의 

  `device-ref` API는 한 걸음 더 나아가 *하나의 키*를 조회하는 작업 자체를 4개, 8개, 또는 32개의 스레드로 구성된 Cooperative Group에 맡긴다.9

- **Warp-level Primitives:** Cooperative Groups의 효율적인 동작은 `__shfl_sync()`, `__ballot_sync()`, `__activemask()`와 같은 워프 내장 함수(warp-level primitives)에 의해 뒷받침된다. 이 함수들은 워프 내 스레드들이 공유 메모리를 거치지 않고 레지스터 간에 직접 데이터를 교환하거나 상태를 공유할 수 있게 해준다. 이는 매우 낮은 지연 시간을 가진 통신을 가능하게 하여 그룹 내 협력 연산의 성능을 극대화한다.14

이러한 접근 방식은 cuCollections의 핵심적인 혁신을 보여준다. 예를 들어, 선형 탐사 과정에서 하나의 스레드가 버킷 i, i+1, i+2를 순차적으로 접근하면 매번 높은 전역 메모리 접근 지연 시간을 겪게 된다. 하지만 Cooperative Group을 사용하면, 그룹 내 스레드 1은 버킷 i, 스레드 2는 i+1, 스레드 3은 i+2를 *동시에* 확인하고, 워프 내장 함수를 통해 결과를 즉시 공유하여 키를 찾거나 빈 공간을 발견할 수 있다. 이는 단일 연산의 지연 시간을 그룹의 병렬 처리량으로 숨기는(hiding latency with throughput) 전형적인 GPU 최적화 기법이며, cuCollections가 CPU 기반 자료구조와 근본적으로 차별화되는 지점이다.



cuCollections는 개발자의 다양한 요구사항을 충족시키기 위해 크게 두 가지의 API 사용 패턴을 제공한다: 호스트 기반의 일괄 처리(Host-bulk)와 디바이스 기반의 개별 참조(Device-ref) 방식이다.23

| 구분 (Category)               | Host-bulk API                  | Device-ref API                                        |
| ----------------------------- | ------------------------------ | ----------------------------------------------------- |
| **실행 주체 (Executor)**      | 호스트(CPU) 코드에서 호출      | 디바이스(GPU) 커널 내에서 호출                        |
| **데이터 규모 (Data Scale)**  | 대규모 데이터셋의 일괄 처리    | 개별 스레드/워프 단위의 단일/소규모 처리              |
| **사용 편의성 (Ease of Use)** | 높음 (단일 함수 호출)          | 낮음 (ref 객체 생성 및 커널 작성 필요)                |
| **유연성 (Flexibility)**      | 낮음 (미리 정의된 연산만 가능) | 높음 (다른 커널 로직과 자유롭게 결합 가능)            |
| **대표 사용 사례**            | ETL, 데이터 전처리, 일괄 조회  | 그래프 알고리즘, 희소 선형대수, 추천 시스템 추론 커널 |

- **Host-bulk API:** 이 패턴은 호스트 코드에서 대규모 데이터셋에 대한 연산을 한 번의 함수 호출로 실행하는 방식이다. 예를 들어, GPU 메모리에 있는 수백만 개의 키-값 쌍을 해시맵에 삽입하는 작업을 `map.insert(keys_begin, keys_end)`와 같은 단일 명령으로 수행할 수 있다. 라이브러리가 내부적으로 최적의 커널 실행 구성과 병렬화 전략을 처리해주므로 사용이 매우 간편하다. 주로 ETL(Extract, Transform, Load) 파이프라인이나 데이터 전처리 단계와 같이 데이터 전체에 대한 일괄 작업이 필요할 때 유용하다.
- **Device-ref API:** 이 패턴은 디바이스 커널 내에서 각 스레드나 Cooperative Group이 해시맵에 개별적으로 접근해야 할 때 사용된다. 먼저 호스트에서 해시맵의 참조(reference) 객체(`ref_type`)를 생성하여 커널에 인자로 전달한다. 커널 내의 각 스레드는 이 참조 객체를 통해 `contains()`, `insert()`, `retrieve()`와 같은 개별 연산을 수행할 수 있다. 이는 그래프 순회, 희소 행렬 연산, 또는 복잡한 시뮬레이션과 같이 알고리즘의 흐름에 따라 자료구조에 비정형적으로 접근해야 하는 경우에 필수적이며, 개발자에게 최대의 유연성을 제공한다.


cuCollections는 이론적인 성능을 넘어 실제 산업 현장의 복잡한 문제를 해결하는 데 핵심적인 역할을 수행하고 있다.

- **RAPIDS cuDF:** GPU 가속 데이터 분석 라이브러리인 cuDF는 Pandas와 유사한 API를 제공하며, 그 강력한 성능의 이면에는 cuCollections가 있다.12

  - **`drop_duplicates` (중복 제거):** 데이터프레임에서 중복된 행을 제거하는 작업은 데이터 정제 과정에서 매우 흔하다. 이 작업을 수행하면서 원본 데이터의 순서를 유지하는 것(stable ordering)은 중요한 요구사항이다. 데이터를 정렬한 후 중복을 제거하는 방식은 비용이 많이 들 수 있다. cuDF는 cuCollections의 `static_set` 또는 `static_map`을 활용하여 해시 기반으로 이 문제를 해결한다. 각 행의 해시값을 키로, 행의 인덱스를 값으로 사용하여 해시 테이블에 삽입을 시도한다. 이미 키가 존재하는 경우 중복으로 처리하고, 처음 나타나는 행만 유지함으로써 정렬 없이도 원본 순서를 보장하는 고성능 중복 제거를 구현한다.11

  - **Hash Join (해시 조인):** 두 데이터프레임을 특정 키를 기준으로 병합하는 조인 연산은 데이터 분석의 핵심이다. 해시 조인은 두 테이블 중 작은 테이블을 이용해 해시 테이블을 구축(build phase)하고, 큰 테이블의 각 행으로 해시 테이블을 탐색(probe phase)하여 일치하는 쌍을 찾는 방식으로 동작한다.39 cuDF의 고성능 해시 조인은 내부적으로 cuCollections의 

    `static_multimap` (키 중복을 허용해야 하므로)을 사용하여 빌드 단계를 극적으로 가속화한다. 이 과정에서 cuCollections의 삽입 및 조회 성능이 전체 조인 연산의 성능을 직접적으로 결정하게 된다.13

- **Pinterest 추천 시스템:** 글로벌 이미지 공유 플랫폼인 Pinterest는 사용자에게 개인화된 콘텐츠를 추천하는 시스템의 실시간 추론(inference) 파이프라인에 GPU 가속을 도입하여 놀라운 성과를 거두었다. 이 시스템의 주된 병목은 수백 개에 달하는 사용자 및 콘텐츠 특징(feature)에 대한 임베딩 테이블 조회(embedding table lookup) 작업이었다. 각 조회가 개별적인 연산으로 처리되면서 오버헤드가 누적되었다. Pinterest 엔지니어링팀은 NVIDIA와 협력하여 cuCollections의 동시성 해시 테이블을 도입했다. 이를 통해 여러 임베딩 조회 요청을 하나의 커널 내에서 단일 연산으로 병합하여 처리할 수 있었고, 파이프라인의 병목을 성공적으로 제거했다. 그 결과, 동일한 IT 예산 내에서 추론 효율성을 100배 이상 향상시키고, 최종적으로는 사용자의 홈 피드 참여율(engagement)을 16%나 끌어올리는 비즈니스 성과를 달성했다.9 이는 cuCollections가 대규모 데이터 분석뿐만 아니라, 지연 시간에 민감한 실시간 AI 서비스의 핵심 엔진으로도 활용될 수 있음을 보여주는 강력한 증거이다.



cuCollections의 성능은 NVBench라는 NVIDIA의 마이크로벤치마킹 프레임워크를 통해 체계적으로 측정 및 관리된다.41 그러나 실제 애플리케이션과의 통합 과정에서 발생하는 미묘한 성능 변화는 저수준의 깊이 있는 분석을 요구한다.

GitHub 이슈(#698)에 기록된 성능 회귀 사례는 이를 명확히 보여준다.13 RAPIDS cuDF의 해시 조인 구현을 기존의 레거시 해시맵에서 새로운 cuCollections의 `multiset`으로 전환하는 과정에서, 초기에는 최대 60%에 달하는 심각한 성능 저하가 관찰되었다. 이 문제에 대한 심층 분석 결과는 다음과 같다.

1. **1차 원인 (표면적):** 성능 저하의 직접적인 원인은 `erase`(삭제) 연산을 지원하기 위해 새롭게 추가된 로직 때문이었다. 이 문제를 수정한 후에도 여전히 약 20%의 성능 격차가 존재했다.
2. **2차 원인 (근본적):** 남은 20%의 성능 격차는 대부분 `count` 연산에서 발생했다. 프로파일링 결과, 새로운 cuCollections 구현에서 버킷 저장소(bucket storage)의 자료구조를 단순 1차원 배열에서 `cuda::std::array`의 배열(사실상 2차원 구조)로 변경한 것이 문제의 핵심이었다. 이 변경으로 인해 컴파일러가 생성하는 최종 기계어(SASS)에 약 40% 더 많은 분기 명령(branching instructions)이 포함되었고, 이는 GPU의 SIMT 실행 모델에 비효율을 초래하여 성능 저하를 유발했다.13

이 사례는 GPU 커널의 성능이 고수준의 알고리즘 논리뿐만 아니라, 메모리 레이아웃과 같은 저수준 구현 세부 사항이 컴파일러 최적화와 상호작용하여 최종적으로 생성되는 기계어에 얼마나 민감하게 반응하는지를 극명하게 보여준다.


cuCollections의 성능을 올바르게 평가하기 위해서는 CPU 기반 동시성 해시 테이블(예: Intel TBB의 `concurrent_hash_map`)과의 아키텍처적 차이를 이해해야 한다.

- **아키텍처적 차이:** CPU는 개별 작업의 빠른 완료, 즉 낮은 지연 시간(low latency)에 최적화되어 있다. 반면 GPU는 수많은 작업을 동시에 처리하는 능력, 즉 높은 처리량(high throughput)에 최적화되어 있다.20 Intel TBB의 해시맵은 소수의 강력한 코어가 각자의 대용량 캐시를 활용하여 복잡한 작업을 빠르게 처리하도록 설계된 반면 22, cuCollections는 수천 개의 단순한 코어가 협력하여 대규모 데이터를 처리하도록 설계되었다.23
- **성능 특성:** 단 하나의 키를 조회하는 것과 같은 단일 작업은 GPU 커널 실행에 따르는 고정 오버헤드와 Host-Device 간 데이터 전송 시간 때문에 CPU에서 처리하는 것이 훨씬 빠르다.45 그러나 수백만 개의 키를 동시에 조회하거나 삽입하는 대규모 일괄(bulk) 작업의 경우, GPU의 압도적인 병렬 처리 능력이 이러한 오버헤드를 상쇄하고도 남아 CPU 기반 솔루션보다 수십 배에서 수백 배 더 높은 처리량을 달성할 수 있다.45

| 특성 (Characteristic) | GPU (cuCollections)                                  | CPU (Intel TBB `concurrent_hash_map`)               |
| --------------------- | ---------------------------------------------------- | --------------------------------------------------- |
| **최적 성능 지표**    | 처리량 (Throughput)                                  | 지연 시간 (Latency)                                 |
| **최적 워크로드**     | 대규모 일괄(Bulk) 연산 (수만~수백만 개 키 동시 처리) | 소규모, 개별(Singular) 연산, 복잡한 로직            |
| **메모리 관리**       | 명시적인 Host-Device 데이터 전송 필요                | 운영체제가 관리하는 통합 가상 메모리                |
| **프로그래밍 모델**   | CUDA 커널, Host-bulk/Device-ref API                  | 표준 C++ 스레딩 (pthreads, OpenMP, `std::thread`)   |
| **성능 저하 요인**    | 데이터 전송 오버헤드, 커널 실행 오버헤드, 분기       | 캐시 미스(Cache miss), 동기화 경합(lock contention) |


cuCollections의 성능을 최적화하기 위해서는 다음과 같은 핵심 파라미터들을 고려해야 한다.

- **적재율 (Load Factor):** 해시 테이블에 채워진 버킷의 비율. 적재율이 높아질수록 충돌이 빈번해져 탐색(probe) 횟수가 증가하고 성능이 저하된다. 특히 선형 탐사(Linear Probing)는 클러스터링 현상으로 인해 적재율 증가에 더 민감하게 반응한다.33
- **탐사 전략 (Probing Strategy):** 충돌 해결 방식의 선택은 성능에 큰 영향을 미친다. 충돌이 거의 발생하지 않는 낮은 적재율 환경에서는 캐시 효율이 뛰어난 선형 탐사가 유리할 수 있다. 반면, 높은 적재율이나 키 중복이 많은 워크로드에서는 클러스터링에 강한 이중 해싱(Double Hashing)이 더 안정적인 성능을 제공할 수 있다.47
- **Cooperative Group (CG) 크기:** `device-ref` API 사용 시, 하나의 개별 연산을 처리하는 데 참여할 스레드의 수. 이 값은 해시맵의 크기, 데이터 분포, 충돌 빈도 등 워크로드의 특성에 따라 최적값이 달라질 수 있다. NVIDIA는 공식 문서에서 해시맵의 크기가 1,000만 개 이하일 경우 CG 크기를 1 또는 2로, 그보다 클 경우 4 또는 8로 설정할 것을 권장하고 있다.47


cuCollections는 단순한 유틸리티성 자료구조 라이브러리를 넘어, NVIDIA의 GPU 가속 데이터 사이언스 및 HPC 생태계를 떠받치는 핵심적인 기반 기술이라 할 수 있다. 이는 GPU의 대규모 병렬 아키텍처가 가진 고유한 제약 조건들, 즉 SIMT 실행 모델, 메모리 접근 패턴, 동시성 제어의 어려움을 극복하고 하드웨어의 잠재력을 최대한 이끌어내기 위해 정교하게 설계된 결과물이다. RAPIDS cuDF와 Pinterest 추천 시스템의 성공 사례에서 확인되었듯이, cuCollections는 실제 산업 현장에서 복잡한 데이터 처리 병목을 해결하고, 측정 가능한 비즈니스 가치를 창출하며 그 중요성을 입증하고 있다.

cuCollections는 현재도 활발하게 개발이 진행 중인 '살아있는' 프로젝트이다.23 GitHub의 이슈 및 토론 페이지에서는 라이브러리의 미래를 엿볼 수 있는 다양한 논의가 지속적으로 이루어지고 있다.48

- **성능 개선:** cuDF 해시 조인 성능 회귀 문제와 같은 실제 애플리케이션과의 통합 과정에서 발견된 병목 현상을 해결하기 위한 저수준 최적화 작업이 계속될 것이다. 이는 메모리 레이아웃 변경, SASS 명령어 최적화 등 하드웨어에 대한 깊은 이해를 바탕으로 이루어질 것이다.13
- **기능 추가:** `dynamic_map`에 `insert_or_assign`과 같은 편의 기능을 추가하고 49, Roaring bitmaps와 같은 새로운 형태의 압축 자료구조를 도입하려는 논의가 진행 중이다. 이는 cuCollections의 활용 범위를 기존의 해시 테이블 중심에서 더욱 다양한 영역으로 확장시킬 것이다.

결론적으로, cuCollections의 발전은 곧 NVIDIA GPU 컴퓨팅 생태계 전체의 성능과 기능성의 발전으로 직결된다. 이 라이브러리가 더욱 성숙하고 풍부해짐에 따라, 더 많은 개발자와 데이터 과학자들이 이전에는 불가능했던 규모의 문제들을 GPU 가속을 통해 해결할 수 있게 될 것이며, 이는 데이터 중심의 미래 기술 발전에 있어 중요한 이정표가 될 것이다.


1. Our History: Innovations Over the Years - NVIDIA, accessed August 4, 2025, https://www.nvidia.com/en-us/about-nvidia/corporate-timeline/
2. What exactly is "CUDA"? (Democratizing AI Compute, Part 2) - Modular AI, accessed August 4, 2025, https://www.modular.com/blog/democratizing-compute-part-2-what-exactly-is-cuda
3. About CUDA | NVIDIA Developer, accessed August 4, 2025, https://developer.nvidia.com/about-cuda
4. CUDA - Wikipedia, accessed August 4, 2025, https://en.wikipedia.org/wiki/CUDA
5. GPU-Accelerated Data Analytics: Best Practices - SeiMaxim, accessed August 4, 2025, https://www.seimaxim.com/kb/dedicated-servers/gpu-accelerated-data-analytics-best-practices
6. Data-Parallel Hashing Techniques for GPU Architectures - CDUX, accessed August 4, 2025, https://cdux.cs.uoregon.edu/pubs/LessleyArea.pdf
7. High-Throughput Data Structures for GPU-Accelerated Computing - ResearchGate, accessed August 4, 2025, https://www.researchgate.net/publication/393039894_High-Throughput_Data_Structures_for_GPU-Accelerated_Computing
8. High-Throughput Data Structures for GPU-Accelerated Computing, accessed August 4, 2025, https://ijaidsml.org/index.php/ijaidsml/article/download/32/30
9. Maximizing Performance with Massively Parallel Hash Maps on GPUs - NVIDIA Developer, accessed August 4, 2025, https://developer.nvidia.com/blog/maximizing-performance-with-massively-parallel-hash-maps-on-gpus/
10. Open-Source Projects - NVIDIA Developer, accessed August 4, 2025, https://developer.nvidia.com/open-source?page=2
11. Supercharging Deduplication in pandas Using RAPIDS cuDF | NVIDIA Technical Blog, accessed August 4, 2025, https://developer.nvidia.com/blog/supercharging-deduplication-in-pandas-using-rapids-cudf/
12. CUDA-X GPU-Accelerated Libraries - NVIDIA Developer, accessed August 4, 2025, https://developer.nvidia.com/gpu-accelerated-libraries
13. [BUG]: Investigate the performance regression with the new cuco data structures / Issue #698 / NVIDIA/cuCollections - GitHub, accessed August 4, 2025, https://github.com/NVIDIA/cuCollections/issues/698
14. Using CUDA Warp-Level Primitives | NVIDIA Technical Blog, accessed August 4, 2025, https://developer.nvidia.com/blog/using-cuda-warp-level-primitives/
15. Introduction to CUDA programming - Julian Roth, accessed August 4, 2025, https://julianroth.org/documentation/cuda/
16. Dynamic Hopscotch Hash Tables on the GPU - GitHub Pages, accessed August 4, 2025, https://jscarleton.github.io/COMP5704Project/Final_Paper.pdf
17. Performance Evaluation of Concurrent Lock-free Data Structures on GPUs - CSE - IIT Kanpur, accessed August 4, 2025, https://www.cse.iitk.ac.in/users/mainakc/pub/icpads2012.pdf
18. what's cga in cuda programming model - Stack Overflow, accessed August 4, 2025, https://stackoverflow.com/questions/78510678/whats-cga-in-cuda-programming-model
19. A Dynamic Hash Table for the GPU - GitHub Pages, accessed August 4, 2025, https://jscarleton.github.io/COMP5704Project/PDF_files_of_relevant_papers/08425196.pdf
20. CPU vs GPU: Architectural Differences, Performance Metrics, and Use Cases - Medium, accessed August 4, 2025, https://medium.com/@prabhuss73/cpu-vs-gpu-architectural-differences-performance-metrics-and-use-cases-ccb44c018e2a
21. Efficiently Processing Joins and Grouped Aggregations on GPUs - arXiv, accessed August 4, 2025, https://arxiv.org/html/2312.00720v2
22. concurrent_hash_map - Intel, accessed August 4, 2025, https://www.intel.com/content/www/us/en/docs/onetbb/developer-guide-api-reference/2021-9/concurrent-hash-map.html
23. NVIDIA/cuCollections - GitHub, accessed August 4, 2025, https://github.com/NVIDIA/cuCollections
24. cuCollections/README.md at dev - GitHub, accessed August 4, 2025, https://github.com/NVIDIA/cuCollections/blob/dev/README.md
25. parallel processing - CUDA - Implementing Device Hash Map? - Stack Overflow, accessed August 4, 2025, https://stackoverflow.com/questions/5533102/cuda-implementing-device-hash-map
26. CMakeLists.txt - NVIDIA/cuCollections - GitHub, accessed August 4, 2025, https://github.com/NVIDIA/cuCollections/blob/dev/CMakeLists.txt
27. NVIDIA/cccl: CUDA Core Compute Libraries - GitHub, accessed August 4, 2025, https://github.com/NVIDIA/cccl
28. Ecosystem | RAPIDS | RAPIDS | GPU Accelerated Data Science, accessed August 4, 2025, https://rapids.ai/ecosystem/
29. RAPIDS Suite of AI Libraries | NVIDIA Developer, accessed August 4, 2025, https://developer.nvidia.com/rapids
30. Reusable Computational Patterns for Machine Learning and Information Retrieval with RAPIDS RAFT | NVIDIA Technical Blog, accessed August 4, 2025, https://developer.nvidia.com/blog/reusable-computational-patterns-for-machine-learning-and-data-analytics-with-rapids-raft/
31. Concurrent Chaining Hash Maps for Software Model Checking - Stanford CS Theory, accessed August 4, 2025, http://theory.stanford.edu/~barrett/fmcad/papers/FMCAD2019_paper_27.pdf
32. How do HashTables deal with collisions? - Stack Overflow, accessed August 4, 2025, https://stackoverflow.com/questions/4980757/how-do-hashtables-deal-with-collisions
33. Optimizing Open Addressing - Max Slater, accessed August 4, 2025, https://thenumb.at/Hashtables/
34. Collision Resolution Techniques in Hash Table: A Review, accessed August 4, 2025, https://thesai.org/Downloads/Volume12No9/Paper_84-Collision_Resolution_Techniques_in_Hash_Table.pdf
35. Performance of Hash Implementations on Various Workloads | by Ani Aggarwal - Medium, accessed August 4, 2025, https://medium.com/geekculture/performance-of-hash-implementations-on-various-workloads-fedac579a39b
36. Asking for an algorithm about massive data processing - CUDA Programming and Performance - NVIDIA Developer Forums, accessed August 4, 2025, https://forums.developer.nvidia.com/t/asking-for-an-algorithm-about-massive-data-processing/14344
37. Cooperative Groups: Flexible CUDA Thread Programming | NVIDIA Technical Blog, accessed August 4, 2025, https://developer.nvidia.com/blog/cooperative-groups/
38. cuCollections/include/cuco/static_map.cuh at dev - GitHub, accessed August 4, 2025, https://github.com/NVIDIA/cuCollections/blob/dev/include/cuco/static_map.cuh
39. Data Analytics are Faster on GPUs - Here's Why | Voltron Data, accessed August 4, 2025, https://voltrondata.com/blog/data-analytics-are-faster-on-gpus
40. Pinterest Boosts Home Feed Engagement 16% With Switch to GPU Acceleration of Recommenders | NVIDIA Blog, accessed August 4, 2025, https://blogs.nvidia.com/blog/pinterest-gpu-acceleration-recommenders/
41. CUB Benchmarks - cub 3.2 documentation, accessed August 4, 2025, https://nvidia.github.io/cccl/cub/benchmarking.html
42. NVIDIA/nvbench: CUDA Kernel Benchmarking Library - GitHub, accessed August 4, 2025, https://github.com/NVIDIA/nvbench
43. CPU hashes faster than GPU? - c++ - Stack Overflow, accessed August 4, 2025, https://stackoverflow.com/questions/46530852/cpu-hashes-faster-than-gpu
44. Why is TBB Concurrent HashMap So Slow? - Intel Community, accessed August 4, 2025, https://community.intel.com/t5/Intel-oneAPI-Threading-Building/Why-is-TBB-Concurrent-HashMap-So-Slow/td-p/829062
45. How to write a simple GPU hash table that can process 300 million+ operations/second, accessed August 4, 2025, https://www.reddit.com/r/programming/comments/fgnenk/how_to_write_a_simple_gpu_hash_table_that_can/
46. Architecture Analysis for ETL Processing: CPU vs GPU - YouTube, accessed August 4, 2025, https://www.youtube.com/watch?v=0HGit6Utsis
47. [ENHANCEMENT]: Perf guide / Issue #250 / NVIDIA/cuCollections - GitHub, accessed August 4, 2025, https://github.com/NVIDIA/cuCollections/issues/250
48. NVIDIA cuCollections / Discussions / GitHub, accessed August 4, 2025, https://github.com/NVIDIA/cuCollections/discussions
49. Issues / NVIDIA/cuCollections - GitHub, accessed August 4, 2025, https://github.com/NVIDIA/cuCollections/issues


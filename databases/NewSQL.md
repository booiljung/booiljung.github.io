# NewSQL

## 서론

데이터베이스 기술의 역사는 데이터 처리 요구사항의 변화와 그에 대한 기술적 응전의 연속이었다. 1970년대 IBM이 관계형 모델과 SQL(Structured Query Language)을 세상에 내놓으며 데이터 관리의 정형화된 시대를 열었다면 1, 2000년대 후반 인터넷의 폭발적인 성장과 함께 등장한 비정형 데이터의 홍수는 기존 패러다임의 한계를 드러냈다.1 이러한 배경 속에서 등장한 NoSQL('Not Only SQL')은 유연한 데이터 모델과 수평적 확장성을 무기로 새로운 대안을 제시했다. 그러나 NoSQL이 확장성을 위해 데이터 정합성을 일부 희생하면서, 두 패러다임의 장점만을 결합하려는 시도가 나타났다. 2011년, 컨설팅 기업 451 Group은 이러한 흐름을 'NewSQL'이라 공식적으로 명명하며 새로운 데이터베이스 패러다임의 등장을 알렸다.3

NewSQL의 핵심 가치 제안은 단순히 관계형 데이터베이스(RDBMS)와 NoSQL의 기계적 결합이 아니다.5 이는 관계형 모델이 제공하는 데이터 무결성에 대한 엄격한 보증과 개발자에게 친숙한 SQL 인터페이스라는 장점을 그대로 유지하면서 7, 동시에 NoSQL의 전유물로 여겨졌던 수평적 확장성(Horizontal Scalability)과 고가용성을 분산 아키텍처 위에서 구현하려는 담대한 시도이다.8 즉, NewSQL은 과거에는 양립 불가능하다고 여겨졌던 '확장 가능한 관계형 데이터베이스'라는 목표를 정면으로 추구한다.

본 보고서는 NewSQL의 등장 배경과 핵심 정의를 시작으로, 그 기술적 근간을 이루는 분산 아키텍처와 합의 알고리즘의 이론적 토대를 심층적으로 분석한다. 이후 Google Spanner, CockroachDB, TiDB와 같은 시장의 주요 시스템들이 이러한 이론을 실제 아키텍처에 어떻게 녹여냈는지 구체적인 사례를 통해 탐구한다. 마지막으로 NewSQL 도입 시 마주하게 될 현실적 과제와 미래 전망을 논하며, 현대 데이터 아키텍처에서 NewSQL이 차지하는 위상을 체계적으로 정립하는 것을 목적으로 한다.

## 1.  NewSQL의 탄생: RDBMS와 NoSQL의 한계를 넘어서

### 1.1  관계형 데이터베이스(RDBMS)의 시대와 그 확장성의 한계

RDBMS는 수십 년간 데이터베이스 시장의 표준으로 군림해왔다. 그 핵심 경쟁력은 ACID(원자성, 일관성, 고립성, 지속성) 트랜잭션을 통해 데이터의 정합성과 무결성을 최우선으로 보장하는 데 있었다.4 이러한 특성 덕분에 금융 거래, 전사적 자원 관리(ERP) 시스템 등 단 하나의 오류도 허용되지 않는 전통적인 OLTP(Online Transaction Processing) 환경의 확고한 기반이 되었다.8

그러나 RDBMS의 아키텍처는 본질적으로 단일 서버의 하드웨어 성능(CPU, RAM)을 증강시키는 수직적 확장(Vertical Scaling, Scale-Up)에 최적화되어 있었다.10 인터넷 시대가 도래하며 데이터의 양과 사용자 트래픽이 기하급수적으로 증가하자, 단일 서버의 처리 용량을 아득히 초과하는 새로운 형태의 OLTP, 즉 'New OLTP' 워크로드가 발생했다.3 이는 RDBMS가 가진 근본적인 확장성의 한계를 명확히 드러내는 계기가 되었다.11 더 좋은 성능의 컴퓨터를 구매하는 방식은 비용이 막대했으며, 결국에는 물리적 한계에 부딪혔다.

### 1.2  NoSQL의 등장: 수평적 확장성과 유연성의 대안

1990년대 이후 인터넷이 보편화되고 소셜 미디어, 검색 로그, 동영상 등과 같은 비정형 데이터가 폭발적으로 증가하면서, 엄격하고 정형화된 스키마를 강제하는 RDBMS의 한계는 더욱 부각되었다.1 이러한 문제에 대한 해결책으로 NoSQL이 부상했다.

NoSQL은 'Not Only SQL'이라는 이름처럼, 값비싼 고성능 서버 한 대에 의존하는 대신 저렴한 상용 서버(commodity server) 여러 대를 클러스터로 묶어 수평적으로 확장(Horizontal Scaling, Scale-Out)하는 방식을 채택했다.2 또한, 스키마 없는(Schema-less) 유연한 데이터 모델을 제공하여 다양한 형태의 데이터를 쉽게 저장하고 처리할 수 있게 했다.12 구글, 페이스북, 아마존과 같은 거대 인터넷 기업들이 자체 서비스의 막대한 데이터를 처리하기 위해 NoSQL 기술을 개발하고 적극적으로 채택하면서, NoSQL은 빅데이터 시대의 기술 유행을 선도하게 되었다.2

### 1.3  NoSQL의 이면: 완화된 일관성과 트랜잭션의 부재

NoSQL이 제공하는 뛰어난 확장성과 성능은 공짜가 아니었다. 대부분의 NoSQL 시스템은 이를 위해 RDBMS의 가장 중요한 특징이었던 ACID를 포기하는 대가를 치렀다. 대신 이들은 BASE(Basically Available, Soft state, Eventually consistent) 철학을 채택했는데, 이는 시스템이 항상 가용한 상태를 유지하되 데이터의 일관성은 특정 시간이 지난 후에야 최종적으로 맞춰진다는 '최종적 일관성(Eventual Consistency)'을 의미한다.10

이러한 설계적 선택은 필연적으로 트랜잭션 지원을 약화시키거나 아예 제거하는 결과를 낳았다.3 이로 인해 금융 거래나 주문 처리 시스템과 같이 데이터의 강력한 정합성이 비즈니스의 성패를 좌우하는 영역에서는 NoSQL을 적용하기가 매우 어려웠다.4 더불어, SQL과 같은 표준화된 쿼리 언어가 없다는 점은 개발자들에게 높은 학습 곡선을 요구했고, '다루기 어렵다'는 인식을 심어주었다.2

### 1.4  NewSQL의 정의: RDBMS의 신뢰성과 NoSQL의 확장성을 융합하다

NewSQL은 바로 이러한 RDBMS의 신뢰성과 NoSQL의 확장성 사이의 깊은 간극을 메우기 위해 탄생한 패러다임이다. 2011년 451 Group의 분석가 Matthew Aslett은 NewSQL을 "관계형 모델의 이점을 분산 아키텍처에 적용하거나, 수평적 확장이 더 이상 필요 없을 정도로 관계형 데이터베이스의 성능을 향상시키려는 새로운 관계형 데이터베이스 제품 및 서비스"라고 정의했다.3

핵심적으로 NewSQL은 하나의 문장으로 요약될 수 있다. 바로 RDBMS의 상징과도 같은 **표준 SQL 인터페이스**와 **완벽한 ACID 트랜잭션**을 지원하면서 4, 동시에 NoSQL의 가장 큰 특징인 **수평적 확장성**을 제공하는 차세대 분산 데이터베이스 시스템이다.8

NewSQL의 등장은 단순한 기술적 진보를 넘어, 시장이 '데이터의 가치'를 인식하는 방식이 성숙했음을 보여주는 중요한 지표다. 초기 빅데이터 시대에는 방대한 양의 데이터를 효율적으로 '수집'하고 '저장'하는 것이 최우선 과제였기에 NoSQL의 확장성이 각광받았다.2 그러나 기업들이 이 데이터를 단순 보관하는 것을 넘어, 금융 거래, 실시간 개인화 추천, 글로벌 서비스 운영 등 핵심 비즈니스 로직에 직접 활용하기 시작하면서 상황이 바뀌었다.4 이러한 활용 사례에서는 데이터의 양(Volume)만큼이나 데이터의 '정확성'과 '신뢰성'이 중요해진다. NoSQL이 제공하는 '최종적 일관성'은 이러한 시나리오에서 심각한 비즈니스 리스크를 초래할 수 있기 때문이다.8 결국 시장은 RDBMS가 보장하던 '신뢰성'과 NoSQL이 제공하던 '확장성'이라는, 이전에는 상충된다고 여겨졌던 두 가지 가치를 동시에 요구하게 되었다. NewSQL은 바로 이러한 시장의 요구 변화에 대한 기술적 응답인 셈이다.

**Table 1: RDBMS, NoSQL, NewSQL 패러다임 비교**

| 구분               | RDBMS (예: MySQL, PostgreSQL) | NoSQL (예: MongoDB, Cassandra) | NewSQL (예: CockroachDB, TiDB)            |
| ------------------ | ----------------------------- | ------------------------------ | ----------------------------------------- |
| **데이터 모델**    | 정형(Relational), 스키마 필수 | 비정형/반정형, 스키마 유연     | 정형(Relational), 스키마 필수             |
| **주요 확장 방식** | 수직 확장 (Scale-Up)          | 수평 확장 (Scale-Out)          | 수평 확장 (Scale-Out)                     |
| **일관성 모델**    | ACID (Strong Consistency)     | BASE (Eventual Consistency)    | ACID (Strong Consistency)                 |
| **트랜잭션 지원**  | 강력한 ACID 지원              | 제한적 또는 미지원             | 분산 환경에서 완전한 ACID 지원            |
| **쿼리 언어**      | 표준 SQL                      | 비SQL API (다양함)             | 표준 SQL                                  |
| **주요 사용 사례** | 금융, ERP, 전통적 OLTP        | SNS, 로그, 캐시, IoT           | 핀테크, 글로벌 SaaS, HTAP                 |
| **대표 시스템**    | MySQL, Oracle, PostgreSQL     | MongoDB, Cassandra, Redis      | Google Spanner, CockroachDB, TiDB, VoltDB |

## 2.  NewSQL의 핵심 아키텍처와 기술적 특성

NewSQL이 RDBMS의 신뢰성과 NoSQL의 확장성이라는 두 마리 토끼를 잡을 수 있었던 비결은 그 독특한 아키텍처에 있다. NewSQL은 분산 시스템 기술을 적극적으로 채택하여 기존 데이터베이스의 한계를 극복한다.

### 2.1  공유-무(Shared-Nothing) 아키텍처: 수평적 확장의 근간

대부분의 NewSQL 시스템은 공유-무(Shared-Nothing) 아키텍처를 기반으로 설계된다.10 이 구조에서 클러스터를 구성하는 각 노드(서버)는 자신만의 CPU, 메모리, 디스크를 독립적으로 소유하며, 오직 네트워크를 통해서만 다른 노드와 통신한다.12 이는 특정 자원을 공유함으로써 발생하는 병목 현상이나 단일 장애점(Single Point of Failure)을 원천적으로 제거하는 효과를 가져온다.10 결과적으로, 시스템의 성능이나 용량이 부족해질 때 단순히 클러스터에 노드를 추가하는 것만으로 전체 시스템의 처리량을 거의 선형적으로 확장(Linear Scalability)할 수 있게 된다.15 이는 단일 고성능 스토리지나 메모리를 공유하는 RDBMS의 공유 디스크(Shared-Disk) 또는 공유 메모리(Shared-Memory) 아키텍처와 근본적인 차이를 보이는 지점이다.

### 2.2  분산 트랜잭션과 ACID 보장 메커니즘

NewSQL이 마주한 가장 큰 기술적 도전은 여러 노드에 데이터가 분산된 공유-무 아키텍처 위에서 어떻게 ACID, 특히 원자성(Atomicity)과 일관성(Consistency)을 보장할 것인가였다. 이를 해결하기 위해 여러 기술이 복합적으로 사용된다.

여러 노드에 걸쳐 있는 트랜잭션의 원자성, 즉 '모두 성공하거나 모두 실패'하도록 보장하기 위해 전통적인 2단계 커밋(Two-Phase Commit, 2PC) 프로토콜이나 이를 변형한 알고리즘이 사용된다.16 또한, 높은 동시 처리 성능을 확보하기 위해 다중 버전 동시성 제어(Multi-Version Concurrency Control, MVCC) 기술이 널리 채택된다.8 MVCC는 데이터를 수정할 때마다 새로운 버전을 생성하는 방식으로, 읽기 작업이 쓰기 작업을 차단(blocking)하지 않도록 하여 동시성을 극대화한다. 일부 시스템은 여기서 더 나아가 잠금(Lock)을 최소화하거나 전혀 사용하지 않는 비잠금 동시성 제어(Non-locking Concurrency Control) 방식을 구현하여 성능을 한층 더 끌어올리기도 한다.11

### 2.3  투명한 샤딩과 분산 합의

샤딩(Sharding) 또는 파티셔닝(Partitioning)은 거대한 데이터셋을 여러 노드에 자동으로 분산 저장하는 기술이다.12 NewSQL의 중요한 특징은 이 샤딩 과정을 데이터베이스 엔진이 내부적으로 처리하여 애플리케이션 개발자에게는 마치 단일 데이터베이스를 사용하는 것처럼 보이게 한다는 점이다. 이를 '투명한 샤딩(Transparent Sharding)'이라고 부른다.12 시스템은 주로 데이터의 키(Key) 범위를 기준으로 데이터를 분할하고(Range-based sharding), 어떤 데이터 조각(Shard 또는 Range)이 어느 노드에 저장되어 있는지에 대한 메타데이터를 중앙에서 관리한다.18

데이터를 여러 노드에 복제(Replication)하여 저장할 때, 각 복제본 간의 데이터를 항상 일관되게 유지하고, 특정 노드에 장애가 발생했을 때 시스템 전체가 중단 없이 동작하도록 하기 위해서는 분산 합의(Distributed Consensus) 알고리즘이 필수적이다. 이를 위해 NewSQL 시스템들은 Paxos나 Raft와 같은 검증된 합의 알고리즘을 내부적으로 사용하여 리더(Leader)를 선출하고, 데이터 변경 사항을 모든 복제본에 안전하게 전파하며, 장애 발생 시 자동으로 시스템을 복구한다.12

### 2.4  고성능을 위한 스토리지 엔진과 인메모리 기술

NewSQL 시스템은 빠른 SQL 쿼리 처리를 위해 고도로 최적화된 스토리지 엔진을 채택한다.12 예를 들어, 구글이 개발한 LevelDB를 기반으로 페이스북이 개선한 RocksDB와 같은 LSM-Tree(Log-Structured Merge-Tree) 기반 스토리지 엔진은 순차적 쓰기에 매우 강한 성능을 보여 분산 환경에서의 쓰기 작업을 효율적으로 처리한다. Percona의 TokuDB는 Fractal Tree 인덱싱이라는 독자적인 기술을 사용하여 대용량 데이터 처리에 대한 확장성을 확보한다.10

또한, 많은 NewSQL 시스템은 디스크 입출력(I/O)으로 인한 병목을 최소화하고 응답 시간을 획기적으로 단축시키기 위해 인메모리(In-Memory) 데이터베이스 기술을 적극적으로 활용한다.5 VoltDB와 같은 시스템은 아예 모든 데이터를 메모리에 상주시키는 것을 전제로 설계되어 실시간 처리가 요구되는 특정 업무에서 극강의 성능을 발휘한다.8

### 2.5  SQL 인터페이스: 개발 생산성과 데이터 분석 능력의 유지

NoSQL과 NewSQL을 구분 짓는 가장 명확한 특징 중 하나는 바로 쿼리 언어다. NewSQL은 애플리케이션과의 주된 상호작용 메커니즘으로 표준 ANSI SQL 및 널리 사용되는 RDBMS의 방언(Dialect)을 채택한다.7

이는 NewSQL 도입에 있어 매우 중요한 전략적 이점을 제공한다. 기존 RDBMS에 익숙한 수많은 개발자들이 별도의 학습 없이 자신의 지식과 경험을 그대로 활용할 수 있으며, 기존에 사용하던 각종 개발 도구, ORM 프레임워크, 비즈니스 인텔리전스(BI) 도구 등 방대한 SQL 생태계를 그대로 이용할 수 있다.10 이는 개발 생산성을 크게 향상시키는 요인이다. 또한, 여러 테이블을 결합하는 복잡한 조인(JOIN)이나 데이터를 그룹화하여 통계를 내는 집계(Aggregation) 쿼리를 자유롭게 사용할 수 있어, 단순 데이터 조회를 넘어 정교하고 깊이 있는 데이터 분석을 가능하게 한다.

NewSQL 아키텍처의 진정한 핵심은 '추상화(Abstraction)'에 있다. 분산 시스템을 구축하고 운영하는 것은 본질적으로 매우 복잡한 일이다. 데이터를 어떻게 나눌지(샤딩), 어떻게 복제할지, 노드 간의 상태를 어떻게 일치시킬지(합의), 특정 노드가 다운되면 어떻게 대처할지(장애 복구) 등 수많은 문제를 해결해야 한다. NoSQL은 이러한 복잡성의 일부를 애플리케이션 개발자의 몫으로 넘기는 경향이 있었다.2 예를 들어, 데이터의 최종적 일관성으로 인해 발생하는 불일치 문제를 애플리케이션 로직에서 처리해야 하는 경우가 그렇다.

반면, NewSQL은 이 모든 복잡성을 데이터베이스 엔진 내부로 완벽하게 숨기는 것을 목표로 한다. 샤딩은 자동으로 이루어지고 12, 데이터 복제와 장애 복구는 Raft와 같은 합의 알고리즘이 내부적으로 처리하며 10, 여러 노드에 걸친 트랜잭션은 MVCC와 2PC 같은 메커니즘으로 관리된다.8 이 모든 복잡한 내부 동작의 결과물로 개발자에게 제공되는 것은 놀랍도록 단순하고 친숙한 인터페이스, 즉 '표준 SQL'과 '완전한 ACID 트랜잭션'이다.7 따라서 NewSQL의 가장 위대한 기술적 성취는 개별 기술(샤딩, Raft 등)의 발명 그 자체라기보다는, 이러한 분산 기술들을 성공적으로 통합하고 패키징하여 사용자에게는 마치 단일 머신에서 동작하는 고전적인 RDBMS처럼 보이는 '단일 시스템 이미지(Single System Image)'라는 강력한 추상화를 제공한 데 있다. 이 추상화 덕분에 사용자는 분산 시스템의 복잡성을 인지하지 않고도 그 강력한 성능과 확장성의 혜택을 온전히 누릴 수 있게 된 것이다.

## 3.  분산 합의의 이론적 토대: CAP 정리와 합의 알고리즘

NewSQL과 같은 분산 데이터베이스를 이해하기 위해서는 그 근간을 이루는 이론적 원리, 특히 CAP 정리와 분산 합의 알고리즘에 대한 이해가 필수적이다. 이들은 분산 시스템이 왜 그렇게 설계되었는지를 설명하는 핵심 열쇠다.

### 3.1  CAP 정리: 일관성(C), 가용성(A), 분할 용인(P)의 트레이드오프

2000년, UC 버클리의 Eric Brewer 교수가 제창한 CAP 정리는 분산 데이터 저장소가 다음 세 가지 보장 사항 중 최대 두 가지만을 동시에 만족시킬 수 있다는 원리다.14

- **일관성 (Consistency, C):** 모든 노드가 항상 동일한 데이터를 보여주는 것을 보장한다. 즉, 특정 데이터에 대한 모든 읽기 요청은 가장 최근에 성공적으로 기록된 값을 반환받거나, 오류를 반환해야 한다.
- **가용성 (Availability, A):** 시스템의 일부 노드에 장애가 발생하더라도, 모든 요청은 (설령 오래된 데이터를 반환하더라도) 항상 성공적인 응답을 받아야 한다. 시스템이 멈추지 않음을 의미한다.
- **분할 용인 (Partition Tolerance, P):** 노드 간의 통신이 일시적으로 단절되는 네트워크 분할(Network Partition) 상황이 발생하더라도 시스템 전체가 중단되지 않고 계속 동작해야 한다.

현실 세계의 분산 시스템에서 네트워크 장애는 피할 수 없는 문제이므로, 분할 용인(P)은 선택이 아닌 필수 사항으로 간주된다.14 따라서 시스템 설계자는 필연적으로 네트워크 분할이 발생했을 때 일관성(C)과 가용성(A) 사이에서 무엇을 우선할지 선택해야 하는 트레이드오프에 직면한다. 전통적인 RDBMS는 데이터 정합성을 최우선으로 하므로 CP(일관성 우선) 시스템으로 분류할 수 있으며, 대부분의 NoSQL 시스템은 서비스 중단을 피하기 위해 AP(가용성 우선)를 선택하는 경향이 있다.14 NewSQL은 이 문제에 대해 정교한 접근법을 취한다. 네트워크 분할이 없는 평상시에는 일관성과 가용성(CA)을 모두 높은 수준으로 제공하다가, 분할이 감지되면 가용성을 일부 희생하더라도 데이터 일관성을 지키는 CP 모델로 동작하여 강력한 데이터 정합성을 유지하도록 설계된다.24

### 3.2  Paxos 알고리즘: 분산 합의의 고전

분산 시스템에서 모든 노드가 하나의 값에 대해 동일하게 동의하는 과정, 즉 합의(Consensus)를 달성하는 것은 매우 어려운 문제다. Leslie Lamport가 제안한 Paxos는 비동기 네트워크 환경에서 일부 프로세스가 응답하지 않거나 실패하는 상황에서도 안전하게 합의를 이끌어낼 수 있는, 최초의 실용적인 알고리즘으로 평가받는다.26

Paxos는 시스템의 노드들을 세 가지 역할로 구분한다: 값을 제안하는 **Proposer**, 제안을 검토하고 수락하는 **Acceptor**, 그리고 최종 합의된 값을 학습하는 **Learner**.28 합의 과정은 크게 두 단계로 진행된다.

1. **Phase 1 (Prepare/Promise):** Proposer가 고유하고 단조 증가하는 제안 번호(n)를 다수의 Acceptor들에게 보내 제안 준비를 알린다(Prepare). Acceptor들은 자신이 이전에 약속했던 어떤 제안 번호보다 n이 클 경우에만, 더 이상 n보다 낮은 번호의 제안을 받지 않겠다고 약속(Promise)하며 응답한다.
2. **Phase 2 (Accept/Accepted):** Proposer가 과반수의 Acceptor로부터 Promise 응답을 받으면, 제안할 값(v)과 제안 번호 n을 담아 다시 Acceptor들에게 수락을 요청한다(Accept). Acceptor들은 자신이 약속한 내용을 어기지 않는 한에서 이 제안을 수락(Accepted)한다. 과반수의 Acceptor가 동일한 제안을 수락하면, 그 값은 최종적으로 합의된 것으로 간주된다.

Paxos는 분산 합의 문제에 대한 이론적으로 완벽한 해답을 제시했지만, 그 동작 방식이 매우 복잡하고 미묘하여 올바르게 이해하고 구현하기가 극도로 어렵다는 치명적인 단점을 가지고 있다.32

### 3.3  Raft 알고리즘: 이해 가능성에 초점을 맞춘 합의 방식

스탠포드 대학의 Diego Ongaro와 John Ousterhout는 Paxos의 이러한 복잡성을 해결하고자 '이해 가능성(Understandability)'을 최우선 설계 목표로 삼아 Raft 알고리즘을 개발했다.32 Raft는 복잡한 합의 문제를 다음 세 가지의 비교적 독립적인 하위 문제로 명확하게 분해하여 접근성을 높였다.34

1. **리더 선출 (Leader Election):** Raft 클러스터의 모든 노드는 항상 **Leader**, **Follower**, **Candidate** 중 하나의 상태를 가진다.36 모든 데이터 변경 요청은 Leader를 통해서만 처리된다. Follower들은 Leader로부터 주기적으로 하트비트(Heartbeat) 메시지를 수신하는데, 만약 정해진 시간(election timeout) 동안 하트비트를 받지 못하면 Leader가 다운된 것으로 간주하고 스스로 Candidate 상태로 전환하여 새로운 리더 선출을 위한 선거를 시작한다. 각 노드는 랜덤화된 타이머 값을 사용하기 때문에, 여러 노드가 동시에 선거를 시작하여 표가 분산되는(split vote) 문제를 최소화하고 신속하게 다음 리더를 선출할 수 있다.28
2. **로그 복제 (Log Replication):** 일단 리더가 선출되면, 시스템의 모든 변경 사항은 리더를 통해서만 이루어진다. 클라이언트로부터 변경 명령을 받은 리더는 해당 명령을 자신의 로그(log)에 추가한 뒤, `AppendEntries` RPC(원격 프로시저 호출)를 통해 다른 Follower들에게 이 로그를 복제하도록 요청한다. 과반수의 Follower가 자신의 로그에 해당 항목을 성공적으로 복제했다고 응답하면, 리더는 해당 로그 항목을 '커밋(commit)'된 것으로 간주하고 자신의 상태 머신(state machine)에 적용한 뒤 클라이언트에게 성공을 알린다.28
3. **안전성 (Safety):** Raft는 시스템의 데이터 정합성을 보장하기 위해 몇 가지 중요한 안전장치를 둔다. 예를 들어, 특정 임기(term)에는 단 한 명의 리더만 선출될 수 있으며, 가장 최신 로그를 가진 노드만이 리더가 될 자격을 갖는다. 이를 통해 데이터가 손실되거나 덮어쓰이는 일을 방지한다.

### 3.4  Paxos와 Raft의 비교 분석 및 NewSQL에서의 역할

Raft는 성능이나 내결함성 측면에서 Paxos와 동등한 수준을 제공한다.33 그러나 두 알고리즘은 구조와 철학에서 근본적인 차이를 보인다. Paxos는 모든 노드가 잠재적으로 Proposer가 될 수 있는 대칭적인 구조를 가지지만, 이로 인해 알고리즘의 흐름이 복잡해진다. 반면 Raft는 오직 리더만이 쓰기를 주도하는 강력한 리더 중심의 비대칭적 구조를 채택하여, 데이터의 흐름을 '리더에서 팔로워로' 흐르는 단방향으로 단순화했다.38

이러한 '이해 가능성'의 차이는 실용적인 측면에서 매우 중요하다. Raft는 명확하게 정의된 노드의 상태(Leader, Follower, Candidate)와 잘 분해된 문제 덕분에 Paxos보다 훨씬 직관적으로 이해하고 올바르게 구현하기가 쉽다.40 이러한 장점 때문에 Kubernetes의 핵심 스토어인 etcd를 비롯하여 CockroachDB, TiDB 등 수많은 최신 분산 시스템들이 사실상의 표준 합의 알고리즘으로 Raft를 채택하고 있다.36

CAP 정리는 종종 '세 가지 중 두 개를 선택하는 것'이라는 정적인 문제로 단순화되어 이해되곤 한다. 하지만 NewSQL 시스템의 등장은 이 정리가 실제로는 '정적 선택'의 문제가 아니라 '동적 트레이드오프 관리'의 문제임을 명확히 보여준다. NewSQL 시스템은 CAP 정리를 우회하거나 극복한 것이 아니다. 대신, Raft의 하트비트와 같은 정교한 장애 감지 메커니즘을 통해 네트워크 분할(P)이라는 '예외 상황'과 '정상 상황'을 구분하고, 각 상황에 맞춰 시스템의 보증 수준을 동적으로 전환하는 고도의 엔지니어링 결과물이다. 평상시에는 높은 가용성(A)과 강력한 일관성(C)을 모두 제공하는 CA 시스템처럼 동작하다가 25, 네트워크 분할이 감지되는 순간에는 데이터의 정합성을 지키기 위해 가용성을 일부 희생하는 CP 시스템으로 전환된다. 이러한 복잡한 트레이드오프는 2010년에 제안된 PACELC 정리 14에 의해 더 잘 설명될 수 있다. PACELC는 "만약 분할(P)이 발생하면, 가용성(A)과 일관성(C) 사이에서 선택해야 하고, 그렇지 않으면(E), 지연 시간(L)과 일관성(C) 사이에서 선택해야 한다"는 원리로, NewSQL의 동작 방식을 더 정확하게 묘사한다.

**Table 3: Paxos와 Raft 합의 알고리즘 비교**

| 항목                 | Paxos                                 | Raft                                   |
| -------------------- | ------------------------------------- | -------------------------------------- |
| **설계 철학**        | 내결함성 증명에 초점                  | 이해 가능성에 초점                     |
| **리더십**           | 약한 리더십 (모든 노드가 제안 가능)   | 강력한 리더십 (리더만 쓰기/복제 주도)  |
| **구조**             | 대칭적 (Symmetric)                    | 비대칭적 (Asymmetric, Leader-Follower) |
| **핵심 문제 분해**   | 단일 문제로 취급 (Monolithic)         | 리더 선출, 로그 복제, 안전성으로 분해  |
| **리더 선출 방식**   | 제안 번호 경쟁을 통해 암묵적으로 결정 | 명시적 선거, 랜덤 타임아웃             |
| **데이터 흐름**      | 양방향 가능 (Bidirectional possible)  | 리더 -->> 팔로워 단방향 (Unidirectional)  |
| **주요 채택 시스템** | Google Spanner (Chubby)               | etcd, CockroachDB, TiDB                |

## 4.  주요 NewSQL 시스템 심층 분석

이론적 배경을 바탕으로, 현재 시장을 선도하고 있는 대표적인 NewSQL 시스템인 Google Spanner, CockroachDB, TiDB의 아키텍처를 심층적으로 분석하여 이론이 실제 어떻게 구현되는지 살펴본다.

### 4.1  Google Spanner: 글로벌 분산 환경과 TrueTime의 혁신

**개요:** Spanner는 구글이 자사의 핵심 비즈니스인 광고(Google Ads)와 같은 글로벌 서비스를 지원하기 위해 내부적으로 개발한, 세계 최초의 전 지구적 규모(globally-distributed) 분산 SQL 데이터베이스다.19

**아키텍처:** Spanner는 데이터를 여러 서버에 걸쳐 샤딩(sharding)하고, 각 데이터 샤드는 Paxos 합의 알고리즘을 사용하는 복제본 그룹(Paxos group)으로 관리된다. 이 복제본들은 전 세계 여러 데이터센터에 동기식으로 복제되어 높은 가용성과 내구성을 보장한다.19

**TrueTime API:** Spanner를 다른 모든 분산 데이터베이스와 구별 짓는 핵심 기술은 바로 TrueTime이다. TrueTime은 GPS 수신기와 원자 시계(atomic clock)와 같은 하드웨어의 도움을 받아, 구글의 모든 데이터센터에 있는 서버들이 전역적으로 동기화된 시간을 매우 높은 정밀도(통상 10ms 이내의 오차)로 알 수 있게 해주는 API다.19

**외부 일관성 (External Consistency):** Spanner는 TrueTime을 활용하여 모든 트랜잭션에 전역적으로 유일하고 단조롭게 증가하는 타임스탬프를 할당한다. 이를 통해 "만약 트랜잭션 T1이 트랜잭션 T2의 시작 전에 커밋되었다면, T1의 커밋 타임스탬프는 T2의 커밋 타임스탬프보다 반드시 작다"는 것을 보장한다.19 이는 일반적인 직렬성(Serializability)보다 한 단계 더 강력한 보증 수준으로, 트랜잭션의 순서가 실제 시간의 순서와 일치함을 의미한다.

**고성능 읽기:** TrueTime이 부여한 타임스탬프와 MVCC 기술의 결합은 Spanner의 또 다른 혁신을 가능하게 했다. 바로 쓰기 작업을 전혀 차단하지 않으면서도 강력한 일관성을 보장하는 읽기(lock-free strong consistent reads)가 가능하다는 점이다.45 클라이언트는 특정 타임스탬프를 지정하여 그 시점의 데이터 스냅샷을 읽을 수 있으며, 이는 진행 중인 다른 쓰기 트랜잭션과 충돌하지 않는다.

### 4.2  CockroachDB: PostgreSQL 호환성을 통한 생태계 확장과 생존성

**개요:** CockroachDB는 구글 Spanner의 설계 사상에 영감을 받아 개발된 오픈소스 분산 SQL 데이터베이스다.15 그 이름(바퀴벌레)처럼, 어떠한 장애 상황에서도 살아남는 강력한 '생존성(Survivability)'을 핵심 가치로 내세운다.

**계층형 아키텍처 (Layered Architecture):** CockroachDB는 복잡한 시스템을 관리하기 용이하도록 명확하게 구분된 계층형 아키텍처를 채택하고 있다. 최상단에는 클라이언트와 통신하는 SQL 계층이 있고, 그 아래로 트랜잭션 계층, 분산 키-값(KV) 저장소 계층, 그리고 실제 데이터를 디스크에 기록하는 스토리지 계층(RocksDB 사용)이 존재한다.17

**PostgreSQL 호환성:** CockroachDB의 가장 큰 특징이자 전략적 선택은 바로 PostgreSQL과의 높은 호환성이다. CockroachDB는 PostgreSQL 와이어 프로토콜(wire protocol)과 대부분의 SQL 구문을 지원한다.49 이는 기존 PostgreSQL용으로 개발된 수많은 데이터베이스 드라이버, ORM(Object-Relational Mapping) 프레임워크, 그리고 각종 관리 도구 생태계를 거의 수정 없이 그대로 활용할 수 있음을 의미한다.52 이 전략은 개발자들이 새로운 시스템에 대한 학습 비용을 크게 줄이고, 기존 애플리케이션을 CockroachDB로 쉽게 마이그레이션할 수 있도록 하여 진입 장벽을 획기적으로 낮춘다.

**동작 방식:** 데이터는 'Range'라고 불리는 단위로 분할되며, 각 Range는 Raft 합의 알고리즘을 통해 기본적으로 3개 이상의 노드에 복제된다.17 CockroachDB의 모든 노드는 대칭적으로 동작하여 읽기와 쓰기 요청을 모두 처리할 수 있는 Multi-active 가용성 모델을 제공한다. 이는 특정 노드가 리더 역할을 전담하는 방식에 비해 부하 분산과 장애 대응에 유리하다.47

### 4.3  TiDB: HTAP 아키텍처의 구현

**개요:** TiDB는 온라인 트랜잭션 처리(OLTP)와 온라인 분석 처리(OLAP)를 별도의 시스템에서 수행하던 기존의 방식을 탈피하여, 이 두 가지 워크로드를 단일 데이터베이스에서 동시에 처리하는 HTAP(Hybrid Transactional/Analytical Processing)을 목표로 설계된 오픈소스 분산 데이터베이스다.8

**핵심 컴포넌트:** TiDB는 기능에 따라 여러 컴포넌트로 분리된 독특한 아키텍처를 가진다.

- **TiDB Server:** 상태를 저장하지 않는(stateless) 컴퓨팅 노드로, 클라이언트로부터 SQL 요청을 받아 파싱하고 쿼리 실행 계획을 최적화하는 역할을 한다. MySQL 프로토콜과 높은 호환성을 가져 기존 MySQL 생태계를 활용할 수 있다.53
- **PD (Placement Driver):** 클러스터 전체의 '두뇌' 역할을 하는 메타데이터 관리 서버다. 데이터가 어떤 노드에 저장되어 있는지(Region 위치 정보), 클러스터의 토폴로지 등을 관리하며, 분산 트랜잭션에 필요한 타임스탬프를 할당하고 데이터의 부하 분산을 위한 스케줄링을 담당한다.53
- **TiKV Server:** 실제 데이터를 저장하는 분산 키-값(KV) 스토리지 엔진이다. 데이터는 행(row) 기반으로 저장되며, Raft 합의 알고리즘을 통해 복제된다. 빠른 트랜잭션 처리에 최적화되어 OLTP 워크로드를 담당한다.20
- **TiFlash Server:** 분석 쿼리 성능을 가속화하기 위한 특수한 스토리지 엔진이다. TiKV에 저장된 데이터를 비동기적으로 복제하여 열(column) 기반으로 저장한다. TiFlash는 Raft 그룹 내에서 투표권이 없는 'Learner' 역할로 참여하므로, TiFlash 노드의 장애가 OLTP 성능에 영향을 주지 않는다.56

**HTAP 동작 원리:** TiDB의 쿼리 옵티마이저는 들어온 쿼리의 특성을 분석하여, 이 쿼리를 처리하기에 가장 효율적인 스토리지 엔진을 지능적으로 선택한다. 간단한 단일 행 조회와 같은 OLTP성 쿼리는 TiKV로 보내고, 대량의 데이터를 스캔하고 집계하는 OLAP성 쿼리는 TiFlash로 보낸다. 심지어 하나의 쿼리 내에서도 일부는 TiKV에서, 일부는 TiFlash에서 데이터를 가져와 조합하여 최적의 실행 계획을 수립할 수 있다.56

Spanner, CockroachDB, TiDB는 모두 NewSQL이라는 동일한 범주에 속하지만, 이들의 아키텍처를 깊이 들여다보면 각자가 해결하고자 했던 '최우선 문제'가 서로 달랐음을 알 수 있다. 이는 NewSQL 시장이 단 하나의 아키텍처로 수렴하는 것이 아니라, 특정 문제 영역에 특화된 다양한 형태로 분화하고 있음을 보여주는 중요한 증거다.

Spanner는 구글이라는 거대 기업의 전사적 요구, 즉 전 세계에 분산된 서비스(광고, 포토 등)를 위한 '전 지구적 규모의 일관성'을 해결하는 것이 최우선 목표였다. 이를 위해 GPS와 원자 시계라는 하드웨어에 의존하는 'TrueTime'이라는 독자적이고 강력한 기술을 개발하는 길을 선택했다.19

반면, CockroachDB는 Spanner의 혁신적인 아이디어를 일반 기업 환경과 오픈소스 커뮤니티에 적용하는 것을 목표로 했다. 이들에게는 특정 하드웨어에 대한 의존성 없이 순수한 소프트웨어만으로 동작하고, 기존 개발자들이 쉽게 시스템을 도입할 수 있도록 하는 것이 중요했다. 따라서 이들은 가장 널리 쓰이는 오픈소스 RDBMS 중 하나인 PostgreSQL과의 와이어 프로토콜 호환성을 전략적으로 선택하여, 방대한 기존 생태계를 활용하고 진입 장벽을 낮추는 데 집중했다.52

TiDB는 많은 기업이 겪고 있는 고질적인 문제, 즉 트랜잭션 처리를 위한 OLTP 데이터베이스와 데이터 분석을 위한 OLAP 데이터 웨어하우스를 별도로 운영하면서 발생하는 데이터 동기화 지연(ETL 파이프라인의 복잡성) 문제에 주목했다. 이 문제를 해결하기 위해, 행 기반 스토어(TiKV)와 열 기반 스토어(TiFlash)를 하나의 시스템 안에 통합하여 실시간 분석과 트랜잭션을 동시에 처리할 수 있는 'HTAP'라는 독자적인 아키텍처를 고안했다.53

결론적으로, 이 세 시스템의 아키텍처 차이는 단순히 기술적 우위를 가리는 문제가 아니다. 이는 각 시스템이 정의한 '가장 중요한 비즈니스 문제'가 무엇이었는지에 대한 철학적 차이에서 비롯된 필연적 결과다. 이는 NewSQL 도입을 고려하는 사용자가 기술 자체를 평가하기에 앞서, 자신의 조직이 해결해야 할 가장 시급한 문제가 무엇인지(글로벌 확장성, 개발 용이성, 혹은 실시간 분석 통합)를 먼저 명확히 정의해야 함을 시사한다.

**Table 2: 주요 NewSQL 솔루션 아키텍처 비교**

| 항목                   | Google Spanner                     | CockroachDB                                               | TiDB                                 |
| ---------------------- | ---------------------------------- | --------------------------------------------------------- | ------------------------------------ |
| **핵심 아키텍처**      | 글로벌 분산 데이터베이스           | 분산 SQL                                                  | 하이브리드 트랜잭션/분석 처리 (HTAP) |
| **합의 알고리즘**      | Paxos                              | Raft                                                      | Raft                                 |
| **일관성 보증**        | 외부 일관성 (External Consistency) | 직렬성 (Serializability)                                  | 스냅샷 격리 (기본값), 직렬성 지원    |
| **핵심 차별점**        | TrueTime API (전역 타임스탬프)     | PostgreSQL 와이어 프로토콜 호환성                         | TiKV(행) + TiFlash(열) 분리 스토리지 |
| **주요 스토리지 계층** | Colossus 위 태블릿                 | RocksDB (LSM-Tree)                                        | TiKV(RocksDB), TiFlash(자체 열엔진)  |
| **이상적 사용 사례**   | 전 지구적 규모의 트랜잭션 서비스   | PostgreSQL 생태계 활용 및 온프레미스/멀티클라우드 분산 DB | 실시간 분석이 필요한 OLTP 시스템     |

## 5.  NewSQL의 도입과 실제적 고려사항

NewSQL은 강력한 기능과 확장성을 제공하지만, 모든 문제에 대한 만병통치약은 아니다. 성공적인 도입을 위해서는 그 특성을 정확히 이해하고, 적합한 사용 사례를 신중하게 선택하며, 기술적 과제를 인지해야 한다.

### 5.1  NewSQL 도입이 적합한 유스케이스

NewSQL은 다음과 같은 시나리오에서 특히 강력한 가치를 발휘한다.

- **핀테크, 전자상거래, 게임:** 초당 수만 건 이상의 높은 트랜잭션 처리량(high throughput)을 감당해야 하면서도, 결제나 아이템 거래 등에서 단 한 건의 데이터 오류도 용납되지 않는 강력한 일관성이 동시에 요구되는 분야에 이상적이다.8
- **글로벌 SaaS (Software-as-a-Service):** 전 세계 여러 지역에 걸쳐 서비스를 제공해야 하는 애플리케이션의 경우, NewSQL은 데이터의 수평적 확장을 용이하게 하고, 사용자 위치에 따라 데이터를 지리적으로 배치하여 응답 시간을 최적화하는 데 매우 유용하다.8
- **실시간 분석 플랫폼:** 특히 TiDB와 같은 HTAP 아키텍처를 가진 시스템은, 별도의 데이터 웨어하우스나 복잡한 ETL 파이프라인 없이도, 실시간으로 발생하는 트랜잭션 데이터(OLTP)에 대해 복잡한 분석 쿼리(OLAP)를 직접 실행할 수 있게 해준다.3
- **기존 RDBMS의 한계 직면:** 사용자가 급증하여 기존 RDBMS의 수직적 확장(Scale-up) 비용이 감당하기 어려워지거나, 더 이상 성능 개선이 불가능한 한계 상황에 도달했을 때, NewSQL은 효과적인 마이그레이션 대안이 될 수 있다.8

### 5.2  기술적 과제와 운영 복잡성

NewSQL이 제공하는 혜택의 이면에는 분산 시스템 고유의 복잡성이라는 과제가 존재한다.

- **도입 및 운영 난이도:** 단일 노드로 구성된 RDBMS와 달리, NewSQL은 여러 노드로 구성된 클러스터 형태로 운영된다. 따라서 초기 클러스터 구축, 각종 파라미터 설정, 분산 환경에 대한 모니터링 체계 수립 등이 상대적으로 복잡하고 어렵다.8
- **성능 튜닝:** NewSQL은 표준 SQL을 사용하지만, 내부적으로는 쿼리가 여러 노드로 분산되어 실행된다. 따라서 쿼리 성능을 최적화하기 위해서는 RDBMS의 인덱스 튜닝을 넘어, 쿼리 실행 계획이 어떻게 분산되는지, 데이터가 노드 간에 어떻게 이동하는지, 데이터 지역성(Data Locality)은 어떻게 유지되는지 등을 종합적으로 분석하고 튜닝해야 한다.8
- **생태계 성숙도:** 수십 년의 역사를 가진 RDBMS에 비해 NewSQL은 아직 역사가 짧다. 따라서 문제 해결을 위한 커뮤니티의 지식 축적량, 서드파티 도구의 지원 범위, 숙련된 전문가의 수 등 전반적인 생태계의 성숙도가 상대적으로 부족할 수 있다.8
- **마이그레이션 비용:** 기존 RDBMS에서 NewSQL로 시스템을 이전하는 것은 단순히 데이터를 복사하는 것 이상의 노력을 요구한다. 분산 환경에 최적화된 스키마를 재설계해야 할 수 있으며, 데이터 동기화 과정에서의 정합성 문제, 애플리케이션 코드의 일부 수정 등 상당한 비용과 시간이 소요될 수 있다.8

### 5.3  시장 동향과 미래 전망

여러 기술적 과제에도 불구하고 NewSQL 시장은 꾸준히 성장하고 있다.

- **시장 성장:** 데이터베이스 동향 분석에 따르면, 전통적인 RDBMS의 시장 점유율은 점차 하락하는 추세인 반면, NoSQL과 NewSQL은 꾸준히 성장하며 그 자리를 대체하고 있다. 특히 CockroachDB, MemSQL(현재 SingleStore)과 같은 NewSQL 솔루션들은 최근 뚜렷한 상승 추세를 보이고 있다.58
- **클라우드 네이티브로의 진화:** 대부분의 NewSQL 시스템은 처음부터 클라우드 환경에 최적화되어 설계되었다.13 Kubernetes와 같은 컨테이너 오케스트레이션 도구와의 긴밀한 통합을 통해 배포와 확장을 자동화하고 있으며, 이는 DBaaS(Database-as-a-Service) 형태의 완전 관리형 서비스로 발전하는 자연스러운 경로가 되고 있다.
- **AI/ML과의 융합:** 일부 NewSQL 시스템은 데이터베이스 내부에 머신러닝(ML) 기능을 직접 통합하려는 시도를 하고 있다.57 이를 통해 데이터 사용 패턴을 학습하여 쿼리 실행 계획을 자동으로 최적화하거나, 비정상적인 접근 패턴을 감지하여 보안을 강화하는 등, 사람의 개입을 최소화하는 '자율 데이터베이스(Autonomous Database)' 방향으로 발전할 잠재력을 가지고 있다.13
- **"One size does not fit all":** NewSQL이 모든 데이터베이스를 대체하지는 않을 것이다. 튜링상 수상자이자 데이터베이스 분야의 석학인 Michael Stonebraker 교수가 지적했듯이, "하나의 사이즈가 모든 것에 맞을 수는 없다".7 앞으로의 데이터 아키텍처는 워크로드의 특성에 따라 RDBMS, NoSQL, NewSQL 등 여러 종류의 데이터베이스를 조합하여 사용하는 '다중 모델(Polyglot Persistence)' 형태가 주류가 될 것이며, NewSQL은 그 안에서 자신의 역할을 공고히 할 것이다.11

NewSQL을 도입하는 데 있어 가장 큰 장벽은 기술 그 자체가 아니라 '인력과 조직'의 문제일 수 있다. NewSQL 클러스터를 효과적으로 구축하고 안정적으로 운영하기 위해서는, 전통적인 데이터베이스 관리자(DBA)가 갖추었던 SQL 튜닝이나 백업/복구와 같은 기술셋을 넘어서야 한다. 분산 시스템의 동작 원리, 네트워크에 대한 깊은 이해, 그리고 인프라를 코드로 관리하는 DevOps 및 사이트 신뢰성 엔지니어링(SRE) 역량까지 요구된다. 이는 기업이 NewSQL을 도입하는 것이 단순히 새로운 소프트웨어를 설치하는 행위가 아니라, 조직의 기술 역량을 재편하고 새로운 전문가를 양성하거나 채용하기 위한 투자를 동반해야 함을 의미한다. 바로 이러한 이유 때문에, 많은 기업들이 직접 NewSQL 클러스터를 구축하고 운영하는(Self-hosted) 대신, 클라우드 제공업체가 모든 복잡한 운영을 대신해주는 완전 관리형 클라우드 서비스(DBaaS)를 선택하는 경향이 강해지고 있다. 기술 도입의 성공 여부가 기술 자체의 우수성뿐만 아니라, 그 기술을 뒷받침할 수 있는 조직의 준비 상태와 인력 투자 의지에 크게 좌우되기 때문이다.

## 6. 결론

NewSQL은 관계형 데이터베이스의 엄격한 신뢰성과 NoSQL의 유연한 확장성이라는, 과거에는 양립 불가능하다고 여겨졌던 두 가지 핵심 가치를 분산 아키텍처와 정교한 합의 알고리즘이라는 기술적 도구를 통해 성공적으로 융합해낸 혁신적인 데이터베이스 패러다임이다. 이는 단순히 '더 빠른 SQL'을 만드는 시도가 아니라 8, 대규모의 데이터와 트래픽을 안정적으로 처리해야 하는 현대적인 분산 애플리케이션의 근본적인 요구사항에 부응하기 위한 아키텍처의 진화다.

NewSQL은 개발자에게 친숙한 SQL 인터페이스와 완전한 ACID 트랜잭션을 유지하면서도, 필요에 따라 손쉽게 성능을 수평적으로 확장할 수 있는 강력한 이점을 제공한다. 이는 개발 생산성과 시스템의 확장성을 동시에 확보해야 하는 많은 기업에게 매력적인 대안이 된다. 그러나 그 강력한 기능의 이면에는 분산 시스템 고유의 운영 복잡성, 새로운 방식의 성능 튜닝에 대한 요구, 그리고 아직은 성숙 과정에 있는 생태계라는 현실적인 과제가 분명히 존재한다.

미래의 데이터베이스 시장은 특정 기술 하나가 모든 것을 지배하는 독점적 시대가 아닐 것이다. 대신, 처리하고자 하는 워크로드의 특성에 따라 가장 적합한 솔루션을 선택하여 조합하는 '다목적 다중 모델(Polyglot Persistence)' 시대가 될 것이다.7 이러한 거대한 흐름 속에서 NewSQL은 '강력한 일관성이 요구되는 대규모 분산 트랜잭션 시스템'이라는 명확하고 중요한 영역에서 핵심적인 역할을 수행할 것으로 전망된다. 또한, 클라우드 네이티브 기술과의 결합이 가속화되면서 그 복잡성은 점차 추상화되고, 더 많은 기업이 서비스(DBaaS) 형태로 NewSQL의 혜택을 누리게 될 것이다. 따라서 미래의 기술 리더에게 던져지는 질문은 "어떤 데이터베이스가 최고의 기술인가?"가 아니라, "우리가 해결하려는 문제에 가장 적합한 데이터베이스는 무엇인가?"가 되어야 할 것이다.

#### **참고 자료**

1. 데이터과학 유망주의 매일 글쓰기 - 쉬어가는 한 주 - 3. NewSQL, 8월 3, 2025에 액세스, [https://conanmoon.medium.com/%EB%8D%B0%EC%9D%B4%ED%84%B0%EA%B3%BC%ED%95%99-%EC%9C%A0%EB%A7%9D%EC%A3%BC%EC%9D%98-%EB%A7%A4%EC%9D%BC-%EA%B8%80%EC%93%B0%EA%B8%B0-%EC%89%AC%EC%96%B4%EA%B0%80%EB%8A%94-%ED%95%9C-%EC%A3%BC-3-9a0b8411a95f](https://conanmoon.medium.com/데이터과학-유망주의-매일-글쓰기-쉬어가는-한-주-3-9a0b8411a95f)
2. 빅데이터 시대, 'NewSQL'에 주목하라 - 블로터, 8월 3, 2025에 액세스, https://www.bloter.net/news/articleView.html?idxno=15241
3. MySQL vs. NoSQL and NewSQL: 2011-2015 - 그대안의작은호수, 8월 3, 2025에 액세스, https://smallake.kr/?p=2326
4. NewSQL - 위키백과, 우리 모두의 백과사전, 8월 3, 2025에 액세스, https://ko.wikipedia.org/wiki/NewSQL
5. NewSql 이란? All in one DBMS - hyeoneee's blog - 티스토리, 8월 3, 2025에 액세스, https://hyeonyeee.tistory.com/44
6. DB 유형 - NoSQL -NewSQL - Char - 티스토리, 8월 3, 2025에 액세스, https://charstring.tistory.com/104
7. [IT에 한 걸음 더 다가가기] SMAC의 대량 데이터 처리를 위한 DBMS기술 3편 | 인사이트리포트, 8월 3, 2025에 액세스, https://www.samsungsds.com/kr/insights/1233729_4627.html
8. NewSQL이란? 기존 DBMS와 차이점 - Somaz의 IT 공부 일지 - 티스토리, 8월 3, 2025에 액세스, https://somaz.tistory.com/406
9. en.wikipedia.org, 8월 3, 2025에 액세스, https://en.wikipedia.org/wiki/NewSQL
10. NewSQL: Everything You Need to Know - DbVisualizer, 8월 3, 2025에 액세스, https://www.dbvis.com/thetable/newsql-everything-you-need-to-know/
11. NewSQL에 대하여, 8월 3, 2025에 액세스, https://blog.skaiworldwide.com/57
12. NewSQL - 도리의 디지털라이프, 8월 3, 2025에 액세스, https://blog.skby.net/newsql/
13. 뉴SQL: SQL의 부활? NoSQL의 반란? 아니면 둘 다? ‍♂️ - 재능넷, 8월 3, 2025에 액세스, https://www.jaenung.net/tree/3040
14. CAP theorem - Wikipedia, 8월 3, 2025에 액세스, https://en.wikipedia.org/wiki/CAP_theorem
15. Best NewSQL Databases, 8월 3, 2025에 액세스, https://online.mason.wm.edu/blog/best-newsql-databases
16. Spanner, TrueTime & The CAP Theorem - Google Research, 8월 3, 2025에 액세스, https://research.google.com/pubs/archive/45855.pdf
17. NewSQL DATABASES 🗄️. Abstract | by Madusanka Nipunajith - Medium, 8월 3, 2025에 액세스, [https://medium.com/@madushankanipunajith/newsql-databases-%EF%B8%8F-78b50fbae357](https://medium.com/@madushankanipunajith/newsql-databases-️-78b50fbae357)
18. NewSQL in Depth Understanding Its Consistency and Scalability Features, 8월 3, 2025에 액세스, https://amarchenko.dev/blog/2024-03-13-new-sql/
19. Spanner: Google's Globally-Distributed Database, 8월 3, 2025에 액세스, https://research.google.com/archive/spanner-osdi2012.pdf
20. TiKV Overview | TiDB Docs, 8월 3, 2025에 액세스, https://docs.pingcap.com/tidb/stable/tikv-overview
21. Understanding Consensus Algorithms: CAP Theorem, Paxos, and ..., 8월 3, 2025에 액세스, https://charleswan111.medium.com/understanding-consensus-algorithms-cap-theorem-paxos-and-raft-2913ac2c1126
22. CAP Theorem and the Role of Partition Tolerance - Hypermode, 8월 3, 2025에 액세스, https://hypermode.com/blog/cap-theorem-partition-tolerance/
23. What is the CAP Theorem? - Distributed Computing - Hazelcast, 8월 3, 2025에 액세스, https://hazelcast.com/foundations/distributed-computing/cap-theorem/
24. 관계형 마이닝 모델과 NoSQL 데이터 비교 - .NET - Microsoft Learn, 8월 3, 2025에 액세스, https://learn.microsoft.com/ko-kr/dotnet/architecture/cloud-native/relational-vs-nosql-data
25. Spanner, TrueTime & The CAP Theorem | by Harshit Agarwal - Medium, 8월 3, 2025에 액세스, https://medium.com/@harshit.py_2591/spanner-truetime-the-cap-theorem-eebbf875539d
26. [1802.05969] Paxos Consensus, Deconstructed and Abstracted (Extended Version) - arXiv, 8월 3, 2025에 액세스, https://arxiv.org/abs/1802.05969
27. Paxos Consensus, Deconstructed and Abstracted 1 Introduction - Ilya Sergey, 8월 3, 2025에 액세스, https://ilyasergey.net/assets/pdf/papers/paxos-deconstructed-esop18.pdf
28. 호다닥 톺아보는 합의 알고리즘 : PAXOS, RAFT - 호롤리한 하루, 8월 3, 2025에 액세스, https://gruuuuu.github.io/integration/paxos-raft/
29. 분산 환경의 합의 알고리즘 Paxos - 개발자 김선우 블로그, 8월 3, 2025에 액세스, https://code-run.tistory.com/70
30. The Paxos Algorithm, 8월 3, 2025에 액세스, https://www2.cs.sfu.ca/CourseCentral/454/johnwill/paxos.pdf
31. Fast Paxos - Microsoft, 8월 3, 2025에 액세스, https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-2005-112.pdf
32. In Search of an Understandable Consensus Algorithm (Extended Version), 8월 3, 2025에 액세스, https://raft.github.io/raft.pdf
33. Raft Consensus Algorithm, 8월 3, 2025에 액세스, https://raft.github.io/
34. In Search of an Understandable Consensus Algorithm - Stanford University, 8월 3, 2025에 액세스, https://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14.pdf
35. In Search of an Understandable Consensus Algorithm - USENIX, 8월 3, 2025에 액세스, https://www.usenix.org/conference/atc14/technical-sessions/presentation/ongaro
36. Paxos보다 쉬운 Raft Consensus. 이해 가능한 합의 알고리즘을 위한 여정 - Medium, 8월 3, 2025에 액세스, [https://medium.com/rate-labs/raft-consensus-%EC%9D%B4%ED%95%B4-%EA%B0%80%EB%8A%A5%ED%95%9C-%ED%95%A9%EC%9D%98-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98%EC%9D%84-%EC%9C%84%ED%95%9C-%EC%97%AC%EC%A0%95-f7ecb9f450ab](https://medium.com/rate-labs/raft-consensus-이해-가능한-합의-알고리즘을-위한-여정-f7ecb9f450ab)
37. THE RAFT CONSENSUS ALGORITHM, 8월 3, 2025에 액세스, https://raft.github.io/slides/coreosfest2015.pdf
38. Raft 알고리즘 알아보기 - Daniel Kim 의 기술 블로그, 8월 3, 2025에 액세스, https://www.kimsehwan96.com/raft-consensus-algorithm/
39. Backend.AI를 위한 합의 알고리즘 구현: 리더 선출 | Lablup Blog, 8월 3, 2025에 액세스, https://blog.lablup.com/posts/2023/11/30/raft-consensus-algorithm/
40. Raft (algorithm) - Wikipedia, 8월 3, 2025에 액세스, https://en.wikipedia.org/wiki/Raft_(algorithm)
41. In Search of an Understandable Consensus Algorithm, 8월 3, 2025에 액세스, https://cgi.di.uoa.gr/~mema/courses/m120/raft.pdf
42. [ETCD] Raft Consensus Algorithm - velog, 8월 3, 2025에 액세스, https://velog.io/@koo8624/ETCD-Raft-Consensus-Algorithm
43. 3. Week 03: Raft, FLP, CAP, and Byzantine Fault Tolerance - CS6213 2021 - Ilya Sergey, 8월 3, 2025에 액세스, https://ilyasergey.net/CS6213/week-03-bft.html
44. Spanner (database) - Wikipedia, 8월 3, 2025에 액세스, https://en.wikipedia.org/wiki/Spanner_(database)
45. Life of Spanner Reads & Writes - Google Cloud, 8월 3, 2025에 액세스, https://cloud.google.com/spanner/docs/whitepapers/life-of-reads-and-writes
46. Spanner: TrueTime and external consistency - Google Cloud, 8월 3, 2025에 액세스, https://cloud.google.com/spanner/docs/true-time-external-consistency
47. CockroachDB vs. Postgres: a Complete Comparison in 2025 - Bytebase, 8월 3, 2025에 액세스, https://www.bytebase.com/blog/cockroachdb-vs-postgres/
48. SQL Layer - CockroachDB, 8월 3, 2025에 액세스, https://www.cockroachlabs.com/docs/stable/architecture/sql-layer
49. Third-Party Tools Supported by Cockroach Labs, 8월 3, 2025에 액세스, https://www.cockroachlabs.com/docs/stable/third-party-database-tools
50. PostgreSQL Compatibility - CockroachDB, 8월 3, 2025에 액세스, https://www.cockroachlabs.com/docs/stable/postgresql-compatibility
51. CockroachDB vs PostgreSQL: Which One to Choose? - RisingWave, 8월 3, 2025에 액세스, https://risingwave.com/blog/cockroachdb-vs-postgresql-which-one-to-choose/
52. "PostgreSQL Compatible Databases": A Tale of Two Approaches - CockroachDB, 8월 3, 2025에 액세스, https://www.cockroachlabs.com/blog/postgresql-compatible-database-cockroachdb/
53. TiDB Architecture FAQs | TiDB Docs, 8월 3, 2025에 액세스, https://docs.pingcap.com/tidb/stable/tidb-faq
54. High-Level Overview of TiDB: A Comprehensive Guide | by TechLatest.Net | Medium, 8월 3, 2025에 액세스, https://medium.com/@techlatest.net/high-level-overview-of-tidb-a-comprehensive-guide-7bf4c58592af
55. TiDB Architecture | TiDB Docs, 8월 3, 2025에 액세스, https://docs.pingcap.com/tidb/stable/tidb-architecture
56. TiFlash Overview | TiDB Docs, 8월 3, 2025에 액세스, https://docs.pingcap.com/tidb/stable/tiflash-overview
57. What is NewSQL? - Dremio, 8월 3, 2025에 액세스, https://www.dremio.com/wiki/newsql/
58. Google Trend 분석(RDB vs. NoSQL, NEWSQL) - 디비랑[dɪ'bɪraŋ] - 티스토리, 8월 3, 2025에 액세스, https://dbrang.tistory.com/1495


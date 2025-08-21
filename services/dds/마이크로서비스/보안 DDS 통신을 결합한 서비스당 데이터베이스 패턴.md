# 보안 DDS 통신을 결합한 서비스당 데이터베이스 패턴

## 1.  "서비스당 데이터베이스" 마이크로서비스 패러다임

마이크로서비스 아키텍처(MSA)를 성공적으로 구현하기 위한 여정은 단순히 애플리케이션을 여러 개의 작은 서비스로 분해하는 것에서 그치지 않습니다. 진정한 자율성과 확장성을 확보하기 위해서는 데이터 관리 방식에 대한 근본적인 철학적 결정이 선행되어야 합니다. 이 지점에서 '서비스당 데이터베이스(Database-per-Service)' 패턴은 단순한 기술 선택을 넘어, MSA의 핵심 가치를 실현하기 위한 전략적 초석으로 자리매김합니다. 이 패턴은 각 마이크로서비스가 자신만의 독립적인 데이터베이스를 소유하고 독점적으로 접근하는 것을 원칙으로 하며, 이는 서비스 간의 결합도를 근본적으로 차단하는 가장 강력한 수단입니다. 본 파트에서는 이 패러다임의 기본 원칙과 전략적 근거를 탐구하고, 그로 인해 발생하는 명확한 이점과 필연적으로 감수해야 하는 아키텍처적 트레이드오프를 심층적으로 분석합니다.

### 1.1 제 1절: 기본 원칙 및 전략적 근거

'서비스당 데이터베이스' 패턴은 각 마이크로서비스가 자신만의 개인 데이터베이스에 대한 완전한 소유권을 갖는 아키텍처 설계 원칙입니다.1 이 모델에서 다른 서비스는 해당 서비스가 공개적으로 노출한 API를 통하지 않고서는 데이터에 직접 접근할 수 없습니다.3 이 원칙은 마이크로서비스 아키텍처에서 느슨한 결합(Loose Coupling)을 달성하기 위한 가장 핵심적인 전제 조건으로 간주됩니다.1

이 패턴의 가장 중요한 목표는 개별 서비스의 완전한 자율성(Autonomy)을 보장하는 것입니다. 서비스가 자체 데이터베이스를 소유함으로써, 각 개발팀은 다른 서비스에 미치는 영향을 최소화하면서 독립적으로 서비스를 개발, 배포, 확장 및 발전시킬 수 있습니다.5 이러한 자율성은 데이터 계층까지 확장되어, 각 서비스가 자신의 고유한 요구사항에 가장 적합한 데이터베이스 기술을 선택할 수 있게 합니다. 예를 들어, 트랜잭션 처리가 중요한 서비스는 관계형 데이터베이스(RDB)를, 비정형 데이터를 다루는 서비스는 NoSQL 데이터베이스를, 그리고 텍스트 검색 기능이 필요한 서비스는 Elasticsearch와 같은 검색 엔진을 채택할 수 있습니다.1 이러한 기술적 이질성, 즉 '폴리글랏 영속성(Polyglot Persistence)'은 각 서비스의 기능과 성능을 최적화하는 데 기여합니다.1

또한, 이 패턴은 데이터 모델의 엄격한 캡슐화(Encapsulation)를 강제합니다.1 특정 서비스의 데이터베이스 스키마 변경은 해당 서비스에만 영향을 미치며, 시스템 전체에 미치는 파급 효과를 최소화합니다.1 이는 개발자들이 다른 서비스의 테이블을 직접 조회하여 '뒷문(back-door)' 종속성을 만드는 안티패턴을 원천적으로 방지하는 중요한 아키텍처적 제약입니다. 이러한 직접적인 데이터베이스 접근은 결국 시스템을 '분산된 모놀리스(Distributed Monolith)'로 회귀시키는 주된 원인이기 때문입니다.1

이처럼 '서비스당 데이터베이스' 패턴은 단순히 기술적인 선택의 문제를 넘어섭니다. 이는 조직의 구조와 개발 문화에 직접적인 영향을 미치는 전략적 결단입니다. 마이크로서비스 아키텍처의 핵심 이점인 빠른 배포, 향상된 확장성, 팀 생산성 증가는 모두 독립적으로 작업할 수 있는 느슨하게 결합된 서비스에 기반합니다.5 공유 데이터베이스는 분산 아키텍처에서 가장 큰 결합 지점이며, 스키마 종속성, 성능 병목 현상, 그리고 변경 시 여러 팀 간의 조율 필요성을 야기합니다.2 따라서 서비스별로 데이터베이스를 분리하는 것은 이러한 결합을 근본적으로 끊어내는 가장 효과적인 방법입니다. 이 기술적 결정은 결과적으로 작고 집중된 팀이 자신의 서비스 라이프사이클 전체를 소유하는("You build it, you run it") DevOps 및 애자일 방법론의 핵심 원칙을 실현 가능하게 만듭니다.5 즉, 아키텍처 선택이 바람직한 조직 구조를 가능하게 하는 것입니다.

### 1.2 제 2절: 전략적 이점 및 운영상 혜택 분석

'서비스당 데이터베이스' 패턴을 채택함으로써 얻을 수 있는 전략적, 운영적 이점은 명확하며, 마이크로서비스 아키텍처의 근본적인 목표와 직접적으로 연결됩니다.

첫째, **느슨한 결합과 독립적인 확장성**이 극대화됩니다. 각 서비스와 그에 속한 데이터베이스는 다른 서비스와 무관하게 자체적인 부하에 따라 독립적으로 확장될 수 있습니다.5 예를 들어, 트래픽이 많은 주문 서비스는 리소스를 대폭 증설할 수 있지만, 상대적으로 사용량이 적은 알림 서비스는 최소한의 리소스로 운영될 수 있습니다. 이를 통해 전체 시스템의 자원 할당을 최적화할 수 있습니다.2

둘째, **향상된 결함 격리 및 복원력**을 제공합니다. 한 서비스의 데이터베이스에서 장애가 발생하더라도, 그 영향은 해당 서비스 내로 격리됩니다. 이는 다른 서비스들의 정상적인 운영에 직접적인 영향을 주지 않으므로, 시스템 전체의 복원력(Resilience)을 크게 향상시키고 연쇄적인 장애 발생을 방지합니다.5 이는 데이터베이스 장애가 전체 애플리케이션의 중단으로 이어질 수 있는 모놀리식 또는 공유 데이터베이스 아키텍처와 뚜렷하게 대조되는 지점입니다.2

셋째, 앞서 언급된 **기술적 이질성(Polyglot Persistence)**을 통해 최적의 기술 선택이 가능해집니다. 개발팀은 각 서비스의 고유한 데이터 처리 요구사항에 가장 적합한 데이터 저장 기술을 자유롭게 선택할 수 있습니다.1 예를 들어, AWS 환경에서 사용자 서비스는 관계형 데이터 처리에 강한 Amazon Aurora (RDS)를, 판매 데이터 분석 서비스는 대규모 비정형 데이터 처리에 용이한 Amazon DynamoDB (NoSQL)를, 그리고 규정 준수 감사 서비스는 특정 요건을 만족하는 SQL Server 인스턴스를 각각 사용할 수 있습니다.3 이러한 유연성은 서비스별 성능과 기능 구현을 최적화하는 데 결정적인 역할을 합니다.

넷째, **개발 및 배포 속도**가 가속화됩니다. 작고 집중된 팀들은 각자의 서비스를 병렬적으로 개발할 수 있습니다. 데이터베이스 스키마 변경이 서비스 내부에 국한되기 때문에, 배포 주기는 더 짧아지고 위험은 감소합니다.5 이는 모든 변경 사항이 전체 시스템에 영향을 미칠 수 있어 대규모의 조율된 "빅뱅" 릴리스를 필요로 하는 모놀리식 접근 방식과 비교할 때 상당한 애질리티(Agility) 향상을 가져옵니다.5

### 1.3 제 3절: 내재된 복잡성 및 아키텍처적 트레이드오프

'서비스당 데이터베이스' 패턴이 제공하는 강력한 이점에도 불구하고, 이는 분산 시스템의 본질적인 복잡성을 수용하는 대가를 치릅니다. 이 패턴을 채택하는 것은 중대한 아키텍처적 트레이드오프를 감수하겠다는 의미이기도 합니다.

가장 큰 도전 과제는 **분산 데이터 관리의 복잡성**입니다. 여러 서비스에 걸쳐 있는 비즈니스 트랜잭션을 구현하는 것은 매우 어려운 문제가 됩니다.5 2단계 커밋(Two-Phase Commit, 2PC)과 같은 전통적인 분산 트랜잭션(ACID) 방식은 시스템 전체에 잠금(lock)을 유발하여 성능을 저하시키고, 많은 NoSQL 데이터베이스에서는 지원조차 하지 않기 때문에 MSA 환경에서는 기피되는 경향이 있습니다.1

두 번째로, **서비스 간 데이터 조인 쿼리의 어려움**이 있습니다. 모놀리식 아키텍처에서는 간단한 SQL 조인(JOIN)으로 해결될 쿼리가, MSA에서는 여러 서비스의 API를 호출하고 클라이언트 측에서 데이터를 조합해야 하는 복잡한 프로세스로 변모합니다.1 이러한 작업은 상당한 오버헤드를 유발하고 응답 시간을 지연시킬 수 있으며, 이를 해결하기 위해 API Composition이나 CQRS(Command Query Responsibility Segregation)와 같은 별도의 패턴을 도입해야만 합니다. 이는 시스템의 복잡성을 더욱 가중시킵니다.3

세 번째로, **운영 오버헤드의 증가**는 피할 수 없는 현실입니다. 다양한 종류의 데이터베이스 시스템을 여러 개 관리하고, 모니터링하며, 백업 및 유지보수하는 것은 단일 데이터베이스를 관리하는 것보다 훨씬 복잡합니다.2 모든 서비스의 로그와 메트릭을 한 곳에서 수집하고 분석할 수 있는 중앙 집중식 로깅 및 모니터링 시스템 구축이 필수적이지만, 이를 올바르게 구현하는 것 또한 상당한 노력을 요구합니다.7

마지막으로, **데이터 중복의 가능성**이 존재합니다. 쿼리 성능을 최적화하거나 서비스 간의 런타임 종속성을 줄이기 위해, 특정 데이터를 여러 서비스에 걸쳐 복제해야 할 필요가 생길 수 있습니다.2 예를 들어, 주문 서비스가 고객의 이름을 표시하기 위해 매번 고객 서비스에 API를 호출하는 대신, 주문 데이터에 고객 이름을 복제하여 저장할 수 있습니다. 그러나 이는 복제된 데이터의 동기화 및 일관성을 유지해야 하는 또 다른 과제를 낳습니다.12

이러한 장단점을 명확히 이해하는 것은 성공적인 아키텍처 설계를 위한 필수 과정입니다. 아래 표는 '서비스당 데이터베이스' 패턴의 핵심적인 트레이드오프를 요약한 것입니다.

**표 1: 서비스당 데이터베이스 패턴: 장점 대 단점**

| 측면              | 장점                                                         | 단점                                                         |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **서비스 자율성** | 각 서비스는 기술 스택, 개발, 배포, 확장을 독립적으로 수행하여 완전한 자율성을 확보함.2 | 여러 서비스에 걸친 비즈니스 로직을 구현하고 추적하기 어려워지며, 전체적인 시스템 동작을 이해하기 복잡해짐.5 |
| **확장성**        | 서비스별 부하에 따라 독립적으로 데이터베이스를 확장할 수 있어 자원 사용을 최적화함.2 | 여러 데이터베이스 인스턴스를 관리하고 확장하는 데 따르는 운영 복잡성 및 비용이 증가함.2 |
| **결함 격리**     | 한 서비스의 데이터베이스 장애가 다른 서비스로 전파되지 않아 시스템 전체의 복원력이 향상됨.5 | 분산 시스템의 특성상 장애의 근본 원인을 파악하고 디버깅하는 것이 매우 어려워짐.5 |
| **기술 선택**     | 각 서비스의 요구사항에 가장 적합한 데이터 저장 기술(Polyglot Persistence)을 자유롭게 선택할 수 있음.1 | 다양한 데이터베이스 기술을 관리하고 유지보수하기 위한 전문 인력과 도구가 필요해져 운영 부담이 가중됨.2 |
| **데이터 일관성** | 데이터 모델이 서비스 내에 완전히 캡슐화되어 외부의 직접적인 수정으로부터 보호되고 로컬 데이터의 일관성을 보장함.1 | 여러 서비스에 걸친 데이터의 트랜잭션 일관성을 유지하는 것이 매우 복잡하며, 이를 위해 Saga와 같은 고급 패턴이 필요함. 강력한 일관성(Strong Consistency) 달성이 어려움.1 |
| **쿼리 구현**     | 서비스 내의 쿼리는 해당 서비스의 데이터 모델에 최적화되어 단순하고 효율적으로 처리될 수 있음. | 여러 서비스의 데이터를 조인해야 하는 쿼리는 API Composition이나 CQRS 패턴을 사용해야 하므로 구현이 복잡하고 성능 저하를 유발할 수 있음.1 |
| **운영 복잡성**   | 각 팀이 자신의 서비스와 데이터베이스에만 집중할 수 있어 개발 생산성이 향상됨.5 | 중앙 집중식 로깅, 모니터링, 배포 자동화 등 분산 시스템을 관리하기 위한 인프라 구축 및 운영이 필수적이며 복잡함.5 |

## 2.  분산 데이터 일관성의 미로 탐색하기

'서비스당 데이터베이스' 패턴을 채택하는 순간, 아키텍트는 분산 시스템에서 가장 근본적이고 어려운 문제인 '데이터 일관성'이라는 미로에 들어서게 됩니다. 모놀리식 아키텍처에서는 단일 데이터베이스의 트랜잭션 기능(ACID)이 이 문제를 투명하게 해결해주었지만, 데이터가 여러 서비스에 분산된 환경에서는 더 이상 이 방식을 적용할 수 없습니다. 이로 인해 시스템의 데이터 무결성을 보장하기 위한 새로운 전략과 패턴이 요구됩니다. 본 파트에서는 분산 데이터 일관성의 이론적 배경인 CAP 이론부터 시작하여, 이 문제를 해결하기 위한 실용적인 해법인 Saga 패턴, 그리고 더 나아가 복잡한 쿼리와 상태 관리를 위한 CQRS 및 이벤트 소싱 패턴까지 심층적으로 탐구합니다.

### 2.1 제 4절: 일관성 스펙트럼과 CAP 이론

분산 시스템 설계를 논할 때 CAP 이론을 빼놓을 수 없습니다. CAP 이론은 분산 컴퓨팅 시스템이 일관성(Consistency), 가용성(Availability), 분할 허용성(Partition Tolerance)이라는 세 가지 보증 중 동시에 최대 두 가지만을 만족시킬 수 있다는 원칙입니다.15 마이크로서비스 아키텍처와 같이 네트워크를 통해 통신하는 분산 시스템에서는 네트워크 장애, 즉 분할(Partition)이 언제든 발생할 수 있는 현실적인 제약 조건입니다. 따라서 시스템은 분할 허용성을 기본적으로 선택해야 하며, 결국 아키텍트는 일관성과 가용성 사이에서 트레이드오프를 해야만 합니다.1

- **강력한 일관성 (Strong Consistency):** 시스템의 모든 노드가 항상 동일한 데이터를 보도록 보장하는 모델입니다. 어떤 노드에 쓰기 작업이 성공하면, 그 이후의 모든 읽기 요청은 해당 최신 데이터를 반환받아야 합니다.17 이는 데이터의 정확성이 매우 중요한 금융 거래 시스템 등에서 요구되지만, 모든 노드의 동기화를 기다려야 하므로 시스템의 응답 시간(가용성)이 저하될 수 있습니다.16
- **최종적 일관성 (Eventual Consistency):** 시스템에 새로운 업데이트가 없다면, 일정 시간이 지난 후에는 모든 복제본이 결국 동일한 값으로 수렴하게 되는 것을 보장하는 모델입니다.17 업데이트 직후에는 일부 노드에서 이전 데이터를 읽을 수 있는 일시적인 불일치 상태가 허용됩니다. 마이크로서비스 아키텍처는 높은 가용성과 성능을 우선시하는 경우가 많기 때문에, 최종적 일관성 모델을 적극적으로 수용합니다.15

이러한 일관성 모델의 선택은 순전히 기술적인 결정이 아니라, 비즈니스 요구사항에 의해 주도되어야 합니다. 예를 들어, 사용자의 은행 잔고 조회는 강력한 일관성을 요구하지만, 소셜 미디어의 '좋아요' 수 표시는 몇 초간의 지연을 허용하는 최종적 일관성으로도 충분합니다.15 따라서 각 비즈니스 도메인의 특성을 분석하여 적절한 일관성 수준을 정의하는 것이 중요합니다.

### 2.2 제 5절: Saga 패턴: 분산 트랜잭션을 위한 프레임워크

여러 서비스에 걸쳐 있는 비즈니스 프로세스의 데이터 일관성을 보장하기 위해 고안된 패턴이 바로 Saga입니다. Saga는 2단계 커밋(2PC)과 같은 분산 트랜잭션을 사용하지 않고, 일련의 로컬 트랜잭션들로 긴 비즈니스 트랜잭션을 구현하는 방식입니다.6 각 로컬 트랜잭션은 단일 서비스 내에서 데이터베이스를 업데이트하고, 다음 로컬 트랜잭션을 트리거하기 위해 메시지나 이벤트를 발행합니다. 만약 어느 단계에서든 로컬 트랜잭션이 실패하면, Saga는 이전에 성공했던 트랜잭션들을 되돌리는 보상 트랜잭션(Compensating Transaction)을 순차적으로 실행하여 데이터의 일관성을 유지합니다.6

Saga를 구현하는 방식에는 크게 두 가지가 있습니다.

- **코레오그래피 기반 Saga (Choreography-Based Saga):** 중앙 조정자 없이 각 서비스가 서로의 이벤트를 구독하고 발행함으로써 자율적으로 협력하는 방식입니다.10 예를 들어, '주문 서비스'가 

  `OrderPlaced` 이벤트를 발행하면, '결제 서비스'가 이를 구독하여 결제를 처리하고 `PaymentProcessed` 이벤트를 발행합니다. 다시 '재고 서비스'가 이 이벤트를 구독하여 재고를 차감하는 식입니다.17

  - **장점:** 서비스 간의 결합도가 매우 낮고, 중앙 병목 지점이 없어 확장성이 뛰어납니다. 단순한 워크플로우에 적합합니다.10
  - **단점:** 비즈니스 프로세스가 여러 서비스에 분산되어 있어 전체 워크플로우를 파악하고 추적하기 어렵습니다. 서비스가 많아지면 어떤 서비스가 어떤 이벤트를 구독하는지 파악하기 힘들어지고, 순환 종속성의 위험도 존재합니다.10

- **오케스트레이션 기반 Saga (Orchestration-Based Saga):** Saga 오케스트레이터(Orchestrator)라는 전담 서비스가 전체 워크플로우를 중앙에서 관리하고 조정하는 방식입니다.10 오케스트레이터는 각 서비스에 실행할 작업을 명령(command)으로 보내고, 그 결과를 받아 다음 단계를 결정합니다. 실패 시 보상 트랜잭션 실행도 오케스트레이터가 책임집니다.

  - **장점:** 비즈니스 로직이 중앙에 집중되어 있어 워크플로우를 이해하고 모니터링하기 쉽습니다. 복잡한 로직, 조건부 분기, 명시적인 순서가 필요한 워크플로우에 유리합니다.10
  - **단점:** 오케스트레이터가 단일 실패 지점(Single Point of Failure)이 될 수 있으므로 고가용성 보장이 필수적입니다. 또한, 서비스들이 오케스트레이터에 종속되어 코레오그래피 방식보다 결합도가 높아집니다.10

코레오그래피와 오케스트레이션 사이의 선택은 본질적으로 **서비스 결합도**와 **운영 가시성** 사이의 트레이드오프입니다. 코레오그래피는 "똑똑한 엔드포인트와 멍청한 파이프(smart endpoints and dumb pipes)"라는 MSA의 이상에 완벽하게 부합하며, 각 서비스의 자율성을 극대화하고 직접적인 결합을 최소화합니다.10 하지만 이러한 분산화는 "주문 번호 123의 현재 상태는 무엇인가?"라는 질문에 답하기 어렵게 만듭니다. 디버깅을 위해서는 여러 서비스의 로그를 상관 분석해야 하는 어려움이 따릅니다.5 반면, 오케스트레이션은 워크플로우 로직을 중앙 집중화하여 명확한 제어와 가시성을 제공합니다. 오케스트레이터의 상태 머신이 트랜잭션 상태의 단일 진실 공급원(Single Source of Truth)이 되어 모니터링과 디버깅, 장애 관리가 훨씬 용이해집니다.19 이 결합도의 증가는 운영상의 명확성을 얻기 위한 대가인 셈입니다. 따라서 이 선택은 기술적 순수성뿐만 아니라, 비즈니스 프로세스의 복잡성과 조직의 관측 가능성(Observability) 도구 성숙도를 고려한 전략적 결정이 되어야 합니다.

### 2.3 제 6절: 쿼리 및 상태 관리를 위한 고급 패턴: CQRS와 이벤트 소싱

'서비스당 데이터베이스' 패턴의 또 다른 주요 과제는 여러 서비스에 분산된 데이터를 조회하는 것입니다. 이를 해결하기 위해 CQRS와 이벤트 소싱이라는 고급 패턴이 사용됩니다.

- **명령 쿼리 책임 분리 (Command Query Responsibility Segregation, CQRS):** 이 패턴은 데이터를 변경하는 책임(Command, 쓰기 모델)과 데이터를 조회하는 책임(Query, 읽기 모델)을 명시적으로 분리하는 것입니다.3 MSA 환경에서 CQRS는 특히 유용합니다. 특정 서비스는 다른 서비스들로부터 이벤트를 구독하여, 자신의 쿼리 요구사항에 최적화된 비정규화된 '읽기 전용 모델(Read Model)' 또는 '뷰(View)'를 구축하고 유지할 수 있습니다.11 예를 들어, '보고서 서비스'는 주문 서비스와 고객 서비스로부터 이벤트를 받아 고객별 주문 내역을 미리 계산해 둔 테이블을 만들어 둘 수 있습니다. 이렇게 하면 보고서 조회 요청이 있을 때마다 여러 서비스의 API를 실시간으로 호출하여 데이터를 조인하는 비효율적인 작업을 피할 수 있습니다.3 하지만 이 방식은 읽기 모델의 데이터가 원본 데이터와 약간의 시간 차이(복제 지연)를 가질 수 있다는 점, 즉 최종적 일관성을 감수해야 하는 단점이 있습니다.11
- **이벤트 소싱 (Event Sourcing):** 이 패턴은 데이터의 최종 상태를 저장하는 대신, 해당 데이터의 상태를 변경시킨 모든 이벤트의 전체 목록을 순서대로 저장하는 방식입니다.6 엔티티의 현재 상태는 저장된 이벤트를 처음부터 순서대로 재실행(replay)하여 계산해냅니다. 예를 들어, 은행 계좌의 현재 잔액을 저장하는 대신, 모든 입금 및 출금 '이벤트'를 기록하는 것입니다. 이 방식의 가장 큰 장점은 시스템에서 발생한 모든 변경 사항에 대한 완벽한 감사 추적(audit trail)을 제공하여 디버깅, 분석, 비즈니스 인텔리전스에 매우 유용하다는 점입니다.6 이벤트 소싱은 CQRS와 자연스럽게 결합됩니다. 상태 변경 이벤트가 발생할 때마다 이를 발행하여 여러 CQRS 읽기 모델을 업데이트하는 데 사용할 수 있습니다.6 그러나 이벤트 재실행을 통해 상태를 복원하는 과정이 느릴 수 있다는 단점이 있으며(이는 주기적인 스냅샷 생성을 통해 완화 가능), 구현 자체가 기존의 상태 기반 영속성 모델보다 복잡합니다.6

## 3.  서비스 간 통신의 패러다임 전환: REST에서 DDS로

마이크로서비스 아키텍처에서 서비스 간 통신 방식의 선택은 시스템 전체의 성능, 안정성, 확장성에 지대한 영향을 미칩니다. 현재 업계의 사실상 표준(de facto standard)은 REST API이지만, 복잡하고 실시간성이 요구되는 시스템에서는 REST의 한계가 드러나기도 합니다. 본 파트에서는 기존 REST API 접근 방식을 비판적으로 재평가하고, 이에 대한 대안으로 제시된 데이터 분산 서비스(Data Distribution Service, DDS)를 심층적으로 소개합니다. DDS가 제공하는 데이터 중심(Data-centric) 패러다임, 강력한 QoS(Quality of Service) 정책, 그리고 근본적으로 다른 통신 모델이 어떻게 마이크로서비스 아키텍처에 새로운 가능성을 제시하는지 분석합니다.

### 3.1 제 7절: 복잡한 시스템에서 REST API의 비판적 재평가

REST API는 그 단순성, HTTP 프로토콜에 대한 보편적인 이해도, 그리고 플랫폼 독립성 덕분에 마이크로서비스 간 통신의 기본 선택지로 자리 잡았습니다.9 그러나 시스템의 규모와 복잡성이 증가함에 따라 REST의 본질적인 한계들이 부각되기 시작합니다.

첫째, **동기식 요청-응답 모델의 한계**입니다. REST 통신은 기본적으로 클라이언트가 서버에 요청을 보내고 응답이 올 때까지 대기하는 동기식(synchronous) 모델입니다. 이는 서비스 간에 시간적 결합(temporal coupling)을 야기합니다.18 호출된 서비스가 느리거나 장애가 발생하면, 이를 호출한 클라이언트 서비스의 스레드는 응답을 기다리며 차단(blocking)되어 시스템 전체의 성능과 복원력을 저해하는 연쇄 장애로 이어질 수 있습니다.20

둘째, **"멍청한 파이프"의 역설**입니다. REST는 "멍청한 파이프와 똑똑한 엔드포인트" 철학을 지향하지만, 복잡한 비즈니스 로직을 처리하기 위해 클라이언트 서비스는 종종 여러 다른 서비스를 순차적 또는 병렬적으로 호출하는 오케스트레이터 역할을 수행하게 됩니다.20 이는 클라이언트 서비스의 복잡도를 높이고 다른 서비스들과의 결합도를 증가시켜, 결국 마이크로서비스의 '단일 책임 원칙'을 위배하는 결과를 낳을 수 있습니다.

셋째, **고급 기능의 부재**입니다. REST는 그 자체로는 통신 규약일 뿐, 데이터의 신뢰성 있는 전달, 특정 조건에 따른 데이터 필터링, 우선순위 기반 처리와 같은 정교한 서비스 품질(QoS) 보장 기능을 내장하고 있지 않습니다.23 이러한 기능들은 모두 각 서비스의 애플리케이션 계층에서 직접 구현해야 하며, 이는 코드 중복과 개발 복잡성을 증가시키는 요인이 됩니다.

### 3.2 제 8절: 데이터 분산 서비스(DDS) 소개

DDS는 기존의 통신 패러다임과는 근본적으로 다른 접근 방식을 제시합니다. 이는 OMG(Object Management Group)에 의해 표준화된 실시간 분산 시스템을 위한 통신 미들웨어입니다.24

가장 핵심적인 차이는 **데이터 중심성(Data-Centricity)**입니다. REST가 특정 '리소스'에 대한 행위(GET, POST 등)를 중심으로 하는 리소스 중심(resource-centric)이거나, 메시지 큐가 불투명한 '메시지'를 전달하는 메시지 중심(message-centric)인 반면, DDS는 교환되는 '데이터' 자체의 구조와 의미를 미들웨어가 이해하는 데이터 중심적 모델을 따릅니다.26 DDS에서 인터페이스는 특정 엔드포인트가 아니라, 잘 정의된 구조를 가진 데이터 그 자체입니다.27

DDS는 **"글로벌 데이터 공간(Global Data Space)"**이라는 개념적 모델을 제공합니다.26 시스템에 참여하는 애플리케이션(Participant)들은 강력한 타입 시스템으로 정의된 데이터 스트림인 '토픽(Topic)'을 발행(publish)하거나 구독(subscribe)합니다. 애플리케이션 관점에서는 마치 분산된 공유 메모리 공간에 데이터를 읽고 쓰는 것처럼 느껴지며, 복잡한 네트워크 프로그래밍은 미들웨어에 의해 완전히 추상화됩니다.26

또한, DDS는 일반적으로 **브로커가 없는(Brokerless) P2P(Peer-to-Peer) 아키텍처**를 가집니다. Participant들은 RTPS(Real-Time Publish-Subscribe)와 같은 효율적인 와이어 프로토콜을 사용하여 네트워크상에서 서로를 동적으로 발견하고 직접 통신합니다.26 이는 전통적인 메시징 시스템의 중앙 브로커가 가질 수 있는 단일 실패 지점(SPOF) 및 성능 병목 현상을 원천적으로 제거합니다.26

마지막으로, **동적 발견(Dynamic Discovery)** 기능은 DDS의 강력한 특징 중 하나입니다. Participant들은 네트워크에 참여하면 자동으로 다른 Participant들과 그들이 발행/구독하는 토픽, 그리고 각자가 제공하거나 요구하는 QoS를 발견합니다.26 이를 통해 새로운 서비스가 추가되거나 기존 서비스가 제거되어도 다른 서비스들의 재설정 없이 시스템이 유연하게 확장되는 진정한 '플러그 앤 플레이' 시스템 구축이 가능해집니다.32

### 3.3 제 9절: QoS의 힘: 데이터 분산에 대한 세밀한 제어

DDS의 진정한 힘은 개발자가 비기능적 요구사항을 선언적으로 명시할 수 있게 해주는 풍부한 QoS(Quality of Service) 정책에서 나옵니다. DDS는 20가지가 넘는 구성 가능한 QoS 정책을 제공하며, 이는 발행자(Publisher)와 구독자(Subscriber) 간의 '계약'으로 작용합니다.25 이를 통해 복잡한 통신 로직을 애플리케이션 코드에서 미들웨어 계층으로 위임할 수 있습니다.33

주요 QoS 정책들은 다음과 같습니다:

- **RELIABILITY (신뢰성):** 데이터 전달의 신뢰 수준을 결정합니다. `BEST_EFFORT`는 최대한 전달을 시도하지만 보장하지 않는 반면, `RELIABLE`은 미들웨어가 자동 재전송 등을 통해 데이터가 반드시 전달되도록 보장합니다.33
- **DURABILITY (내구성):** 데이터의 생존 기간을 제어합니다. `VOLATILE`은 전송 후 즉시 소멸, `TRANSIENT_LOCAL`은 발행자가 데이터를 캐싱하여 늦게 참여하는 구독자에게 전달, `TRANSIENT`는 별도 서비스가 데이터를 영속화하여 발행자가 사라진 후에도 데이터에 접근할 수 있게 합니다.29
- **HISTORY (이력):** 구독자를 위해 얼마나 많은 데이터 샘플을 보관할지 결정합니다. `KEEP_LAST(N)`은 최근 N개의 샘플만, `KEEP_ALL`은 리소스 한도 내에서 모든 샘플을 보관합니다.29
- **LIVELINESS (생존성):** Participant들이 서로의 '생존' 여부를 감지하는 메커니즘입니다. 이를 통해 장애 감지 및 자동 페일오버(failover) 구현이 가능합니다.29
- **DESTINATION_ORDER (순서 보장):** 데이터 샘플의 순서를 정렬하는 기준을 정합니다. `BY_RECEPTION_TIMESTAMP`(수신 시간 기준) 또는 `BY_SOURCE_TIMESTAMP`(발신 시간 기준)를 선택할 수 있습니다. 후자는 시스템 전체에서 이벤트 순서를 일관되게 유지하는 데 매우 중요하여 최종적 일관성 달성에 핵심적인 역할을 합니다.35
- **OWNERSHIP (소유권):** 여러 개의 중복된 발행자를 허용하고, 그중 `OWNERSHIP_STRENGTH` 값이 가장 높은 발행자만 활성 상태로 만들어 데이터 발행을 중재합니다. 이는 액티브-패시브(Active-Passive) 방식의 고가용성 구성에 활용됩니다.29

이러한 QoS 정책들은 아키텍트가 시스템의 동작 방식을 코드 레벨이 아닌 설정 레벨에서 정교하게 제어할 수 있도록 지원하며, 이는 DDS를 단순한 통신 프로토콜을 넘어선 강력한 분산 시스템 프레임워크로 만듭니다.

### 3.4 제 10절: 통신 프로토콜 비교 분석

마이크로서비스 아키텍처에서 올바른 통신 프로토콜을 선택하는 것은 매우 중요합니다. 아래 표는 DDS를 REST, gRPC, 그리고 전통적인 메시지 큐와 비교하여 각 기술의 장단점과 적합한 사용 사례를 명확히 보여줍니다. 이 표는 아키텍트가 특정 시스템 요구사항에 기반하여 정보에 입각한 결정을 내리는 데 도움을 주는 핵심적인 도구입니다.

**표 3: 서비스 간 통신 프로토콜 비교 분석**

| 기준                 | REST                            | gRPC                          | 메시지 큐 (예: RabbitMQ)             | DDS                                             |
| -------------------- | ------------------------------- | ----------------------------- | ------------------------------------ | ----------------------------------------------- |
| **통신 모델**        | 요청-응답 (Request-Reply)       | 원격 프로시저 호출 (RPC)      | 메시지 전달 (Message-Passing)        | 데이터 중심 발행-구독 (Data-Centric Pub/Sub)    |
| **전송 프로토콜**    | 주로 HTTP/1.1, HTTP/2           | HTTP/2                        | AMQP, STOMP 등 (TCP 기반)            | RTPS (UDP/TCP/공유 메모리 등)                   |
| **데이터 형식**      | JSON, XML (텍스트 기반)         | Protocol Buffers (바이너리)   | 불투명한 바이너리                    | IDL 기반 강력한 타입 (바이너리)                 |
| **결합도**           | 시간적 결합 (동기식 호출)       | 시간적 결합 (동기식 호출)     | 시간적/공간적 분리 (비동기)          | 시간적/공간적/흐름적 분리 (비동기, 데이터 중심) |
| **서비스 발견**      | 외부 솔루션 필요 (예: Eureka)   | 외부 솔루션 필요              | 브로커가 중앙에서 관리               | 내장된 동적 발견 (P2P)                          |
| **주요 강점**        | 단순성, 보편성, 표준 HTTP 사용  | 고성능, 스트리밍, 강력한 타입 | 비동기 처리, 부하 분산, 장애 격리    | 실시간성, 풍부한 QoS, 데이터 중심, 확장성       |
| **이상적 사용 사례** | 웹 기반 공개 API, 간단한 서비스 | 고성능 내부 서비스 간 통신    | 비동기 작업 처리, 이벤트 기반 시스템 | 미션 크리티컬 분산 시스템 (IoT, 국방, 금융)     |

## 4.  보안 DDS 기반 마이크로서비스 아키텍처 엔지니어링

이론적 기반과 패턴을 이해했다면, 다음 단계는 이를 실제 아키텍처로 구체화하는 것입니다. 본 파트에서는 '서비스당 데이터베이스' 원칙과 DDS 통신을 결합한 마이크로서비스 아키텍처의 구체적인 청사진을 제시합니다. REST 기반 아키텍처와 시각적으로 어떻게 다른지 개념적 다이어그램을 통해 설명하고, DDS의 강력한 보안 프레임워크를 적용하여 어떻게 데이터 버스를 보호할 수 있는지 분석합니다. 또한, 보안 기능이 성능에 미치는 영향을 현실적으로 평가하고, DDS의 QoS 정책을 활용하여 분산 데이터 일관성 문제를 미들웨어 수준에서 어떻게 효과적으로 해결할 수 있는지에 대한 실질적인 엔지니어링 방안을 탐구합니다.

### 4.1 제 11절: 개념적 아키텍처: DDS를 마이크로서비스 환경에 통합하기

DDS 기반 마이크로서비스 아키텍처를 시각화하는 것은 패러다임의 전환을 이해하는 데 매우 중요합니다. 전통적인 REST 기반 아키텍처 다이어그램이 서비스 간 점대점(point-to-point) 화살표로 연결된 모습을 보여준다면, DDS 기반 아키텍처는 모든 서비스(DomainParticipant)가 중앙의 가상적인 **"데이터 버스(Databus)"** 또는 **"글로벌 데이터 공간(Global Data Space)"**에 연결된 형태로 그려집니다.26 이 시각적 메타포는 통신이 특정 엔드포인트를 호출하는 방식이 아니라, 공유된 데이터 공간에 데이터를 게시하고 필요한 데이터를 가져오는 방식으로 이루어짐을 직관적으로 보여줍니다.37

데이터 흐름은 다음과 같이 이루어집니다. 예를 들어, '주문 서비스'는 `Order`라는 타입의 데이터를 `OrderTopic`에 발행(publish)합니다. 이 주문 정보가 필요한 '결제 서비스'와 '배송 서비스'는 단순히 `OrderTopic`을 구독(subscribe)하기만 하면 됩니다. 그러면 DDS 미들웨어는 발행된 `Order` 데이터를 두 구독자 모두에게 효율적으로 라우팅합니다. 이 과정에서 DDS는 네트워크 상황에 따라 유니캐스트(unicast) 또는 멀티캐스트(multicast)를 자동으로 사용하여 최적의 전송 효율을 보장합니다.23

이러한 구조는 REST로는 쉽게 달성하기 어려운 수준의 **결합도 분리(Decoupling)**를 실현합니다. '주문 서비스'는 자신의 데이터를 데이터 버스에 발행할 뿐, 누가 그 데이터를 소비하는지에 대해 전혀 알 필요가 없습니다. '결제 서비스'나 '배송 서비스'의 존재 여부, 위치, 개수와 완전히 무관하게 동작할 수 있습니다.32 나중에 '감사 서비스'나 '데이터 분석 서비스'가 추가로 `OrderTopic`을 구독해야 할 경우에도, 기존의 어떤 서비스도 변경할 필요가 없습니다. 이는 시스템의 유연성과 확장성을 극대화하는 핵심적인 장점입니다.

### 4.2 제 12절: 데이터 버스 보안: DDS 보안 프레임워크 분석

미션 크리티컬 시스템에서 데이터 보안은 타협할 수 없는 요구사항입니다. DDS는 OMG DDS Security 표준을 통해 데이터 중심 환경에 최적화된 포괄적이고 플러그인 가능한 보안 프레임워크를 제공합니다.24

DDS 보안의 핵심 메커니즘은 다음과 같습니다:

- **인증 (Authentication):** 시스템에 참여하는 모든 Participant는 통신 시작 전에 PKI(공개 키 기반 구조) 기반의 상호 인증 절차를 거칩니다. 이를 통해 신뢰할 수 있는 애플리케이션만이 DDS 도메인에 참여할 수 있도록 보장합니다.24
- **접근 제어 (Access Control):** 인증된 Participant라 할지라도 모든 데이터에 접근할 수 있는 것은 아닙니다. 거버넌스(Governance) 및 권한(Permissions) 설정 파일을 통해 각 Participant가 어떤 토픽에 대해 발행(publish) 또는 구독(subscribe)할 수 있는지 세밀하게 제어할 수 있습니다. 이는 비인가된 데이터 접근을 원천적으로 차단합니다.24
- **암호화 (Cryptography):** DDS는 토픽 단위로 데이터를 암호화하고 전자 서명을 추가하여 전송 중인 데이터의 기밀성(Confidentiality)과 무결성(Integrity)을 보장할 수 있습니다.24

그러나 이러한 강력한 보안 기능은 **성능상의 대가**를 수반합니다.

- **디스커버리 스톰 (Discovery Storm):** 보안이 활성화된 DDS 도메인에서 초기 디스커버리 과정은 상당한 네트워크 부하를 유발할 수 있습니다. Participant들은 서로 인증서와 권한 문서를 교환해야 하는데, 이 파일들의 크기가 수십 KB에 달할 수 있습니다. 대규모 시스템에서 수많은 Participant가 동시에 참여할 경우, 이 트래픽이 폭증하여 '디스커버리 스톰' 현상을 일으키고, 시스템 시작 시간을 지연시키거나 심지어 디스커버리 실패를 초래할 수 있습니다.24
- **메시지 암호화 오버헤드:** 모든 메시지를 암호화하고 복호화하는 과정은 CPU 자원을 소모하며, 이는 메시지 전송 지연 시간(latency)을 증가시키는 요인이 됩니다. 특히 페이로드가 큰 메시지의 경우 이 오버헤드는 더욱 두드러집니다.24 이는 보안 수준과 실시간 성능 사이의 명백한 트레이드오프 관계를 보여줍니다.

이러한 성능 저하를 완화하기 위해서는 아키텍처 설계 단계에서부터 전략적인 접근이 필요합니다. 예를 들어, 전체 시스템을 여러 개의 작은 DDS 도메인으로 분할하여 단일 디스커버리 프로세스에 참여하는 Participant의 수를 제한하거나, 모든 토픽에 최고 수준의 암호화를 적용하는 대신 비즈니스 중요도에 따라 보안 수준을 차등 적용하는 방안을 고려할 수 있습니다.

**표 5: 보안 DDS: 기능 대 성능 오버헤드**

| 보안 기능     | 메커니즘                                      | 장점                                                         | 성능 고려사항/오버헤드                                       | 완화 전략                                                    |
| ------------- | --------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **인증**      | PKI 기반 상호 인증서 교환                     | 신뢰할 수 없는 Participant의 도메인 참여를 원천적으로 차단하여 시스템을 보호함.24 | Discovery 과정에서 인증서 교환으로 인한 네트워크 트래픽 및 처리 지연 시간 증가.24 | 도메인 분할, Participant 수 제한, 경량 인증 메커니즘 고려.   |
| **접근 제어** | 거버넌스/권한 파일을 통한 발행/구독 권한 제어 | 인증된 Participant라도 허가된 데이터에만 접근 가능하도록 하여 세밀한 보안 정책 적용.24 | Discovery 과정에서 대용량 권한 문서(수십 KB 이상) 교환으로 인한 '디스커버리 스톰' 유발 가능성.24 | 권한 규칙 최적화, 도메인 분리를 통한 권한 문서 범위 축소.    |
| **암호화**    | 토픽 단위 데이터 암호화 및 전자 서명          | 전송 중인 데이터의 기밀성과 무결성을 보장하여 도청 및 데이터 변조 공격 방지.24 | 메시지 암/복호화에 따른 CPU 사용량 증가 및 전송 지연 시간(latency) 증가. 페이로드가 클수록 오버헤드 커짐.24 | 비즈니스 중요도에 따라 토픽별 암호화 수준 차등 적용, 하드웨어 가속 암호화 사용 고려. |

### 4.3 제 13절: DDS를 활용한 내재적 데이터 일관성 확보

DDS의 강력한 QoS 정책은 분산 데이터 일관성 문제를 해결하는 데 있어 애플리케이션의 복잡성을 크게 줄여주는 역할을 할 수 있습니다. Saga 패턴이나 최종적 일관성 모델을 구현하기 위해 필요한 핵심 기능들을 미들웨어 수준에서 선언적으로 설정할 수 있기 때문입니다.

전통적인 REST와 메시지 큐를 조합한 아키텍처에서는 신뢰성 있는 메시지 전달, 재시도 로직, 데드 레터 큐(DLQ) 처리, 소비자 상태 관리 등을 개발자가 애플리케이션 코드에 직접 구현해야 합니다.19 이는 상당한 양의 보일러플레이트 코드를 양산하고 비즈니스 로직의 복잡성을 증가시킵니다.

반면, DDS는 이러한 요구사항에 직접적으로 매핑되는 QoS 정책을 제공합니다.

- `RELIABILITY = RELIABLE` QoS 설정은 미들웨어가 메시지 전달을 보장하고 실패 시 자동으로 재전송하므로, 애플리케이션의 복잡한 재시도 로직을 대체합니다.35
- `DURABILITY = TRANSIENT` QoS 설정은 장애 후 재시작된 서비스가 참여하지 못했던 동안 발행된 중요한 상태 데이터를 수신할 수 있게 해줍니다. 이는 소비자의 상태 복구 로직을 크게 단순화합니다.35
- `DESTINATION_ORDER = BY_SOURCE_TIMESTAMP` QoS 설정은 여러 노드에서 발행된 이벤트들이 시스템 전체에서 발행된 시간 순서대로 일관되게 처리되도록 보장하여, 순서가 중요한 비즈니스 로직의 데이터 불일치 문제를 방지합니다.35

따라서, DDS 미들웨어를 올바르게 설정함으로써 분산 트랜잭션을 안정적으로 구현하는 데 필요한 복잡한 오류 처리 및 통신 로직의 상당 부분을 애플리케이션에서 덜어낼 수 있습니다. 개발자는 기반 통신이 견고하고 일관되게 동작한다는 것을 신뢰하고, Saga의 각 단계에 해당하는 순수한 비즈니스 로직 구현에 더 집중할 수 있습니다.

또한, DDS는 RPC over DDS 표준의 일부로 요청-응답(Request-Reply) 패턴도 지원합니다.40 이는 오케스트레이션 기반 Saga를 구현하는 데 효과적으로 사용될 수 있습니다. Saga 오케스트레이터는 각 서비스에 '명령'을 요청으로 보내고 '완료' 응답을 기다리는 방식으로 워크플로우를 진행할 수 있으며, 이 모든 통신은 DDS 데이터 버스의 강력한 QoS 보장 하에 이루어집니다.41

## 5.  전략적 권고 및 최종 분석

지금까지 '서비스당 데이터베이스' 패턴과 보안 DDS 통신을 결합한 마이크로서비스 아키텍처의 기술적 깊이와 복잡성을 다각도로 분석했습니다. 이 아키텍처는 분명 모든 문제에 대한 만병통치약이 아니며, 그 강력한 기능만큼이나 신중한 고려가 필요한 선택입니다. 본 마지막 파트에서는 앞선 분석을 종합하여, 어떤 상황에서 이 아키텍처를 선택해야 하는지에 대한 명확한 의사결정 프레임워크를 제공합니다. 그리고 최종적으로 이 아키텍처가 갖는 전략적 의미와 미래 전망을 제시하며 보고서를 마무리합니다.

### 5.1 제 14절: DDS 기반 아키텍처 채택을 위한 의사결정 프레임워크

제시된 아키텍처는 특정 유형의 시스템에 최적화된 고도로 전문화된 솔루션입니다. 따라서 이 아키텍처의 도입 여부는 프로젝트의 특성과 요구사항을 면밀히 분석하여 결정해야 합니다.

**DDS 기반 아키텍처를 적극적으로 고려해야 하는 경우:**

- **실시간 데이터 공유가 핵심 요구사항일 때:** 수많은 분산된 컴포넌트 간에 낮은 지연 시간과 높은 처리량으로 데이터를 지속적으로 공유해야 하는 시스템. 예를 들어, 산업 자동화(IIoT), 실시간 금융 거래 시스템, 국방 관제 시스템 등이 여기에 해당합니다.43
- **복잡하고 동적인 상태를 공유해야 할 때:** 시스템의 많은 참여자가 빠르게 변화하는 '세상의 상태(state of the world)'에 대한 일관된 뷰를 유지해야 하는 경우. 항공 관제, 자율 주행 차량의 센서 융합, 대규모 자산 추적 시스템이 대표적인 예입니다.46
- **높은 수준의 복원력과 결함 허용성이 필요할 때:** 미션 크리티컬 시스템에서 자동 장애 감지, 페일오버, 데이터 전달 보장과 같은 기능이 필수적인 경우. DDS는 QoS 정책을 통해 이러한 요구사항을 미들웨어 수준에서 지원합니다.44
- **동적이고 확장 가능한 토폴로지를 가질 때:** 시스템의 컴포넌트(서비스)가 네트워크에 빈번하게 참여하고 이탈하는 환경. DDS의 동적 발견 기능은 이러한 '플러그 앤 플레이' 시나리오를 별도의 설정 변경 없이 지원합니다.26

**전통적인 REST/gRPC 방식이 더 적합한 경우:**

- **단순한 CRUD(Create, Read, Update, Delete) 서비스가 주를 이룰 때:** 상태가 거의 없고 간단한 요청-응답 상호작용만 필요한 서비스에는 DDS의 복잡성이 과도한 오버헤드가 될 수 있습니다.
- **웹 기반 공개 API를 제공해야 할 때:** 외부 개발자나 파트너에게 API를 제공하는 경우, 사실상의 표준인 HTTP 기반의 REST가 보편성과 접근성 측면에서 훨씬 유리합니다.
- **개발팀의 기술 숙련도가 중요할 때:** 개발팀이 웹 기술(HTTP, JSON)에 매우 익숙하고, 새로운 데이터 중심 패러다임을 학습하는 데 드는 시간과 비용이 프로젝트의 제약 조건일 경우, 기존 기술을 활용하는 것이 더 효율적일 수 있습니다.
- **프로젝트 예산과 일정이 매우 촉박할 때:** DDS는 상용 솔루션의 경우 라이선스 비용이 발생할 수 있으며, 그 학습 곡선과 구현 복잡성으로 인해 초기 개발 속도가 더딜 수 있습니다.

### 5.2 제 15절: 최종 종합 및 전략적 전망

본 보고서에서 분석한 '서비스당 데이터베이스' 패턴과 보안 DDS 통신을 결합한 마이크로서비스 아키텍처는, 복잡하고 데이터 집약적이며 높은 복원력을 요구하는 차세대 분산 시스템을 구축하기 위한 강력하고 정교한 접근법입니다. 이 아키텍처는 전통적인 요청-응답 모델을 뛰어넘어, 비교할 수 없는 수준의 느슨한 결합과 시스템 동작에 대한 선언적 제어 능력을 제공합니다. 각 서비스는 데이터 버스를 통해 자율적으로 상호작용하며, 복잡한 통신 로직은 애플리케이션 코드에서 미들웨어로 위임되어 개발자는 비즈니스 로직에 더욱 집중할 수 있습니다.

그러나 이 강력함에는 **복잡성이라는 명백한 대가**가 따릅니다. 리소스 중심에서 데이터 중심으로의 사고방식 전환은 결코 간단하지 않으며, DDS의 풍부한 QoS 정책과 보안 기능을 올바르게 이해하고 설정하기 위해서는 상당한 학습과 경험이 필요합니다. 운영 측면에서도 다양한 데이터베이스와 고성능 미들웨어를 관리하는 것은 새로운 도전 과제를 제시합니다.

결론적으로, 이 아키텍처는 주류 마이크로서비스 접근법을 대체하는 것이 아니라, 특정 부류의 고난도 문제를 해결하기 위한 **진보된 아키텍처 패턴**으로 이해해야 합니다. 이는 단기적인 편의성보다는 장기적인 시스템의 견고성, 확장성, 그리고 유연성에 투자하는 전략적 결정입니다.

앞으로 산업용 IoT, 자율 주행, 스마트 그리드, 원격 의료와 같이 시스템이 더욱 분산되고, 실시간성을 요구하며, 자율적으로 동작하는 방향으로 발전함에 따라, DDS와 같은 데이터 중심 아키텍처의 중요성은 더욱 커질 것입니다. 이러한 시스템에서 데이터는 더 이상 애플리케이션의 부산물이 아니라, 시스템의 생명선 그 자체이기 때문입니다. 따라서 이 아키텍처를 채택하는 결정은 단순히 현재의 문제를 해결하는 것을 넘어, 미래의 기술적 도전에 대비하고, 변화에 유연하게 대응할 수 있는 견고하고 확장 가능한 데이터 백본(Data Backbone)을 구축하는 장기적인 투자로 간주되어야 합니다.

#### **참고 자료**

1. Pattern: Database per service - Microservices.io, accessed July 16, 2025, https://microservices.io/patterns/data/database-per-service.html
2. Database Per Service Pattern for Microservices - GeeksforGeeks, accessed July 16, 2025, https://www.geeksforgeeks.org/system-design/database-per-service-pattern-for-microservices/
3. Database-per-service pattern - AWS Prescriptive Guidance, accessed July 16, 2025, https://docs.aws.amazon.com/prescriptive-guidance/latest/modernization-data-persistence/database-per-service.html
4. Thoughts on Database Per Service – SQLServerCentral Forums, accessed July 16, 2025, https://www.sqlservercentral.com/forums/topic/thoughts-on-database-per-service
5. 마이크로서비스의 장점 및 알아야 할 단점 - Atlassian, accessed July 16, 2025, https://www.atlassian.com/ko/microservices/cloud-computing/advantages-of-microservices
6. Top 10 Microservices Design Patterns and Their Pros and Cons - IEEE Computer Society, accessed July 16, 2025, https://www.computer.org/publications/tech-news/trends/microservices-design-patterns/
7. 마이크로서비스(Microservice) 정의, 구축, 장단점, 사례 - Red Hat, accessed July 16, 2025, https://www.redhat.com/ko/topics/microservices/what-are-microservices
8. What are the pros and cons of using a microservice architecture when building a new enterprise web application? : r/webdev - Reddit, accessed July 16, 2025, https://www.reddit.com/r/webdev/comments/s6icid/what_are_the_pros_and_cons_of_using_a/
9. REST API와 마이크로서비스 아키텍처(MSA)의 이해 - F-Lab, accessed July 16, 2025, https://f-lab.kr/insight/understanding-rest-api-and-msa
10. 마이크로서비스 사가 패턴 - Medium, accessed July 16, 2025, [https://medium.com/@greg.shiny82/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%82%AC%EA%B0%80-%ED%8C%A8%ED%84%B4-544fc1adf5f3](https://medium.com/@greg.shiny82/마이크로서비스-사가-패턴-544fc1adf5f3)
11. [마이크로서비스 패턴] 7. 마이크로서비스 쿼리 구현 - velog, accessed July 16, 2025, [https://velog.io/@daehoon12/%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%ED%8C%A8%ED%84%B4-7.-%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%BF%BC%EB%A6%AC-%EA%B5%AC%ED%98%84](https://velog.io/@daehoon12/마이크로서비스-패턴-7.-마이크로서비스-쿼리-구현)
12. Software Development Comprehensive Guide - Santha Lakshmi Narayana, accessed July 16, 2025, https://santhalakshminarayana.github.io/blog/software-development-comprehensive-guide
13. 마이크로 서비스 지향 애플리케이션 디자인 - .NET - Learn Microsoft, accessed July 16, 2025, https://learn.microsoft.com/ko-kr/dotnet/architecture/microservices/multi-container-microservice-net-applications/microservice-application-design
14. What are the challenges of data consistency in microservices architecture? - Design Gurus, accessed July 16, 2025, https://www.designgurus.io/answers/detail/what-are-the-challenges-of-data-consistency-in-microservices-architecture
15. Strategies to Tackle Data Consistency Challenges in Distributed Microservices, accessed July 16, 2025, https://www.researchgate.net/publication/392101969_Strategies_to_Tackle_Data_Consistency_Challenges_in_Distributed_Microservices
16. Distributed Data Consistency: Challenges & Solutions - Endgrate, accessed July 16, 2025, https://endgrate.com/blog/distributed-data-consistency-challenges-and-solutions
17. Data Consistency in Microservices: Challenges, Strategies, and Examples | by Mohana Krishna Kavali | Medium, accessed July 16, 2025, https://medium.com/@mohanakrishnakavali/data-consistency-in-microservices-challenges-strategies-and-examples-ecd926cd0839
18. 마이크로서비스 간 연결 - Technocracy - 티스토리, accessed July 16, 2025, https://ent-dreamcatcher.tistory.com/36
19. SAGA Pattern in .NET 8 Explained: Best Practices for Production-Grade Microservice Transactions - Kumar Shivam, accessed July 16, 2025, https://kumarshivam-66534.medium.com/saga-pattern-in-net-8-explained-best-practices-for-production-grade-microservice-transactions-bd35086723de
20. REST vs Messaging for Microservices - Which One is Best? - Solace, accessed July 16, 2025, https://solace.com/blog/experience-awesomeness-event-driven-microservices/
21. A pattern language for microservices (#gluecon #gluecon2016) | PDF - SlideShare, accessed July 16, 2025, https://www.slideshare.net/slideshow/a-pattern-language-for-microservices-gluecon-2016/62447028
22. REST APIs vs Microservices: Key Differences - DreamFactory Blog, accessed July 16, 2025, https://blog.dreamfactory.com/restful-api-and-microservices-the-differences-and-how-they-work-together
23. Middleware Protocols in the Automobile: Service-Oriented, Data-Centric or RESTful? - Vector, accessed July 16, 2025, https://cdn.vector.com/cms/content/know-how/_technical-articles/PREEvision/PREEvision_MiddlewareProtocols_ElektronikAutomotive_202003_PressArticle_EN.pdf
24. 보안 DDS(Data Distribution Service)의 디스커버리 ... - Korea Science, accessed July 16, 2025, https://koreascience.kr/article/JAKO202117457596217.pdf
25. Introduction to DDS - OpenDDS 3.28.0, accessed July 16, 2025, https://opendds.readthedocs.io/en/dds-3.28/devguide/introduction_to_dds.html
26. What is DDS? - DDS Foundation, accessed July 16, 2025, https://www.dds-foundation.org/what-is-dds-3/
27. Future-Proof Architecture for OT Data-Centric Microservices - RTI, accessed July 16, 2025, https://www.rti.com/hubfs/_Collateral/capability-briefs/rti-capability-brief-ot-mcroservices.pdf
28. The Data Backbone of SDV Interoperability - DDS - COVESA, accessed July 16, 2025, https://covesa.global/wp-content/uploads/2024/10/RTI_Whitepaper_50060_DDS-The-Data-Backbone-of-SDV-Interoperability_V1_1024.pdf
29. Data-Centric Programming Best Practices: - RTI, accessed July 16, 2025, https://www.rti.com/hubfs/docs/DDS_Best_Practices_WP.pdf
30. 1.1.1. The DCPS conceptual model - Fast DDS - eProsima, accessed July 16, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/getting_started/definitions.html
31. ROS2, DDS와 QoS - SpatialDipper - 티스토리, accessed July 16, 2025, https://luckydipper.tistory.com/17
32. DDS (Data Distribution Service) Core Concepts | OpenDDSharp Documentation, accessed July 16, 2025, https://www.openddsharp.com/articles/dds_core_concepts.html
33. Introduction to DDS (Data Distribution Service): Real-Time Data Communication Made Easy!, accessed July 16, 2025, https://erhanbakirhan.medium.com/introduction-to-dds-data-distribution-service-real-time-data-communication-made-easy-d6f4badddd6f
34. Quality of Service - OpenDDS 3.28.1, accessed July 16, 2025, https://opendds.readthedocs.io/en/dds-3.28.1/devguide/quality_of_service.html
35. DerechoDDS: Strongly Consistent Data Distribution for Mission-Critical Applications - Computer Science Cornell, accessed July 16, 2025, https://www.cs.cornell.edu/projects/Quicksilver/public_pdfs/2021226041.pdf
36. RTI Connext C API: DDS_DestinationOrderQosPolicy Struct Reference - RTI Community, accessed July 16, 2025, https://community.rti.com/static/documentation/connext-dds/6.1.2/doc/api/connext_dds/api_c/structDDS__DestinationOrderQosPolicy.html
37. Microservices Diagram: Best Practices & Examples - Multiplayer, accessed July 16, 2025, https://www.multiplayer.app/distributed-systems-architecture/microservices-diagram/
38. Data Distribution Service - Wikipedia, accessed July 16, 2025, https://en.wikipedia.org/wiki/Data_Distribution_Service
39. How do you deal with data inconsistency in between microservices? - Reddit, accessed July 16, 2025, https://www.reddit.com/r/microservices/comments/1g7j4xz/how_do_you_deal_with_data_inconsistency_in/
40. 15.13. Request-Reply communication - Fast DDS 2.14.4 documentation, accessed July 16, 2025, https://fast-dds.docs.eprosima.com/en/2.14.x/fastdds/use_cases/request_reply/request_reply.html
41. Tech Talk Summary: Part 3 of 6: How to Simplify SOA Development Using Connext DDS, accessed July 16, 2025, https://www.youtube.com/watch?v=CZZtpoHiLgA
42. RTI® Queuing Service - HubSpot, accessed July 16, 2025, https://cdn2.hubspot.net/hubfs/1754418/RTI_Oct2016/PDF/RTI_Queuing_Service.pdf?t=1494408989576
43. Activity Flow of Alternative Design Evaluation and Deriving Feasible Deployment - ResearchGate, accessed July 16, 2025, https://www.researchgate.net/figure/Activity-Flow-of-Alternative-Design-Evaluation-and-Deriving-Feasible-Deployment_fig6_322373156
44. A Tale of Two Industrial IoT Standards: DDS and OPC-UA - RTInsights, accessed July 16, 2025, https://www.rtinsights.com/dds-opc-ua-industrial-iot-standards/
45. Aerospace and Defense [DDS Foundation Wiki] - OMG Wiki, accessed July 16, 2025, https://www.omgwiki.org/ddsf/doku.php?id=ddsf:public:applications:aerospace_and_defense
46. rticommunity/rticonnextdds-usecases: Case + Code, a ... - GitHub, accessed July 16, 2025, https://github.com/rticommunity/rticonnextdds-usecases




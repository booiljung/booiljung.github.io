---
layout: page
title: 실시간 시스템의 DDS와 마이크로서비스의 통신 패러다임
permalink: /services/dds/마이크로서비스/실시간 시스템의 DDS와 마이크로서비스의 통신 패러다임
---

미션 크리티컬 시스템과 마이크로서비스 아키텍처 모두 근본적으로 '여러 소프트웨어 모듈을 최소한의 기능으로 조직하여 목표를 달성한다'는 공통된 철학을 공유한다는 점입니다. 두 패러다임 모두 거대하고 단일화된 시스템을 더 작고 관리하기 쉬운 단위로 분해하려는 시도라는 점에서 이는 사실입니다. 이 보고서는 이러한 공통된 목표에서 출발하여, 두 아키텍처가 분해를 달성하는 방식, 우선시하는 가치, 그리고 그 결과 왜 서로 다른 통신 메커니즘(DDS와 REST API)을 채택하게 되었는지를 심층적으로 비교 분석합니다.


로봇 공학, 항공 우주 및 기타 미션 크리티컬 시스템에서 단일 장애점이 전체 시스템의 중단을 야기하지 않도록 설계된 아키텍처는 **데이터 중심 발행-구독(Data-Centric Publish-Subscribe, DCPS)** 모델에 기반하며, 이는 객체 관리 그룹(Object Management Group, OMG)의 **데이터 분산 서비스(Data Distribution Service, DDS)** 표준의 핵심입니다.1 이 아키텍처는 전통적인 서비스 지향적 사고방식에서 데이터 중심적 세계관으로의 근본적인 패러다임 전환을 제시합니다.


질의에서 언급된 아키텍처는 공식적으로 데이터 중심 발행-구독(DCPS) 모델로 알려져 있으며, OMG에 의해 DDS라는 표준으로 관리됩니다.1 이는 신뢰성 있고, 고성능이며, 확장 가능한 실시간 데이터 교환을 가능하게 하는 미들웨어 표준입니다.3 주요 적용 분야는 항공 우주, 국방, 로봇 공학, 자율 주행 차량, 산업 자동화와 같이 성능과 신뢰성이 무엇보다 중요한 미션 크리티컬 및 안전 필수 시스템입니다.4

이 아키텍처는 근본적으로 발행-구독 모델이지만, 중요한 차이점이 있습니다. 바로 '메시지 중심'이 아닌 '데이터 중심'이라는 점입니다. 이는 미들웨어가 데이터의 내용과 구조를 인지하고 있음을 의미하며, 데이터를 불투명한 바이트 덩어리가 아닌 구조화되고 타입이 명시된 정보로 취급합니다.7 이 접근법은 시스템의 구성 요소들이 서로를 직접 알 필요 없이 데이터 자체를 통해 통신하게 함으로써 강력한 분리(decoupling)를 달성합니다.


DDS는 "글로벌 데이터 공간(Global Data Space)"이라는 개념을 도입합니다. 이는 가상적이고 분산된 정보 저장소로, 애플리케이션들은 이 공간을 마치 로컬 메모리처럼 접근하여 데이터를 읽고 씁니다.2

이는 개념적 추상화이며, 실제로 모든 데이터가 저장되는 단일 중앙 서버는 존재하지 않습니다. GDS는 각 참여 노드에서 DDS 미들웨어에 의해 관리되는 로컬 데이터 저장소들의 집합을 통해 만들어지는 환상(illusion)에 가깝습니다.10 이러한 브로커리스(brokerless), P2P(Peer-to-Peer) 아키텍처는 단일 장애점과 병목 현상을 제거하는 데 결정적인 역할을 합니다.5 이 모델은 참여자들을 공간(서로의 위치를 알 필요 없음), 시간(동시에 실행될 필요 없음), 그리고 데이터 흐름 측면에서 완벽하게 분리시킵니다.1


DCPS 모델은 시스템의 통신 구조를 정의하는 몇 가지 핵심 엔티티로 구성됩니다.13

- **도메인(Domain):** 격리된 통신 평면을 생성하는 기본 컨테이너입니다. 서로 다른 도메인에 속한 참여자들은 서로를 인지하지 못합니다.10

  `DomainParticipant`는 애플리케이션이 특정 도메인으로 진입하는 시작점을 나타냅니다.

- **토픽(Topic):** 발행과 구독을 연결하는 중심 요소입니다. 토픽은 단순한 문자열 이름 이상으로, 이름, 데이터 타입(IDL, XML 등으로 정의), 그리고 QoS 정책 집합의 고유한 조합입니다.1 이러한 강력한 타입 지정은 메시지 지향 미들웨어와의 핵심적인 차별점입니다.

- **발행자(Publisher) / 데이터 작성자(DataWriter):** 발행자는 데이터 작성자의 팩토리(factory) 역할을 합니다. 데이터 작성자는 특정 토픽에 대한 데이터 샘플을 GDS에 실제로 쓰는 엔티티입니다.13

- **구독자(Subscriber) / 데이터 판독기(DataReader):** 구독자는 데이터 판독기의 팩토리입니다. 데이터 판독기는 특정 토픽을 구독하고 새로운 데이터 샘플을 애플리케이션에 제공하는 엔티티입니다.13

- **인스턴스(Instances, Keyed Data):** DDS는 키(key) 필드를 사용하여 동일한 토픽 내에서 서로 다른 실제 객체들을 구별할 수 있습니다. 예를 들어, "VehiclePosition" 토픽은 고유한 `vehicle_id` 키로 식별되는 여러 인스턴스를 가질 수 있습니다. 미들웨어는 각 인스턴스의 생명주기를 추적합니다.11


DDS의 진정한 힘은 광범위하고 세분화된 서비스 품질(QoS) 정책에 있습니다. 아키텍트는 이를 통해 데이터 분산의 동작을 정밀하게 제어할 수 있습니다.1

장애 허용 및 실시간 성능을 위한 핵심 QoS 정책은 다음과 같습니다.

- **RELIABILITY:** 데이터 전송이 `BEST_EFFORT`(UDP와 유사)인지, 아니면 `RELIABLE`(재시도를 통해 전송 보장)인지를 설정합니다.11
- **DURABILITY:** 데이터가 GDS에 얼마나 오래 유지될지를 제어합니다. `TRANSIENT_LOCAL` 같은 설정은 늦게 참여하는 구독자가 이전에 발행된 데이터를 수신할 수 있게 합니다.
- **LIVELINESS:** 참여자가 실패했거나 응답 불능 상태가 되었음을 시스템이 감지할 수 있게 하는 하트비트 메커니즘입니다.11
- **OWNERSHIP:** 동일한 토픽 인스턴스에 대해 여러 발행자가 있는 시나리오를 관리합니다. 하나를 배타적 소유자로 지정하고 다른 발행자들을 백업으로 지정하여 장애 조치(failover)를 가능하게 합니다.4
- **DEADLINE:** 데이터 업데이트 간의 최대 예상 시간을 명시하여, 데이터 스트림이 오래되었는지(stale) 감지할 수 있게 합니다.
- **HISTORY:** 각 인스턴스에 대해 과거 데이터 샘플을 몇 개나 저장할지 제어합니다.

이러한 아키텍처를 깊이 있게 이해하기 위해서는 두 가지 중요한 관점을 고려해야 합니다. 첫째, DDS는 '움직이는 데이터베이스'처럼 작동한다는 점입니다. GDS 추상화, 타입이 지정된 토픽, 그리고 키로 식별되는 인스턴스의 조합은 DDS가 전통적인 메시징 시스템보다는 분산 실시간 데이터베이스와 유사하게 동작하게 만듭니다. 여기서 '테이블'은 토픽에, '행'은 인스턴스에 해당하며, `DataReader`는 `SELECT` 쿼리와 유사한 역할을 합니다.1 그러나 정적인 데이터를 다루는 전통적인 데이터베이스와 달리, DDS는 '움직이는 데이터'에 최적화되어 있습니다. 핵심 상호작용은 과거 데이터를 질의하는 것이 아니라, 실시간 상태 변화를 통지받는 것입니다.10 이는 근본적으로 세계 모델을 유지하는 것이 중요한 로봇 공학이나 시뮬레이션 분야에 DDS가 왜 적합한지를 직접적으로 설명합니다.6

둘째, DDS는 '스마트 미들웨어' 패러다임을 채택합니다. 광범위한 QoS 정책은 DDS가 엄청난 지능과 책임을 미들웨어 계층 자체에 부여하는 핵심적인 철학적 선택을 보여줍니다. 많은 시스템에서 신뢰성, 재시도, 장애 조치, 데이터 필터링과 같은 비기능적 요구사항은 애플리케이션 코드 레벨에서 구현됩니다.7 반면 DDS는 이러한 요구사항들을 애플리케이션 개발자로부터 추상화하여 선언적인 QoS 정책으로 만듭니다.3 개발자는 원하는 동작(예: 

`RELIABILITY=RELIABLE`)을 선언하기만 하면, '스마트' 미들웨어가 이를 강제합니다. 결과적으로 애플리케이션 로직은 통신의 복잡한 메커니즘 대신 순수하게 비즈니스 목적(예: 센서 데이터 처리)에만 집중할 수 있어 훨씬 더 간결하고 명확해집니다.10 이는 '스마트 미들웨어, 단순한 엔드포인트'라는 패러다임을 형성하며, 마이크로서비스 철학과 직접적으로 대비되는 중요한 지점입니다.


마이크로서비스 아키텍처(Microservice Architecture, MSA)는 비교 분석을 위한 견고한 기반을 제공하기 위해 명확한 이해가 필요합니다. 이 섹션에서는 마이크로서비스 기반 시스템이 설계되고 구축되는 방식을 형성하는 핵심 원칙과 목표에 초점을 맞춥니다.


마이크로서비스 아키텍처(MSA)는 사용자의 질문에서 지적한 바와 같이, 하나의 큰 애플리케이션을 **최소한의 기능을 가진 작고 독립적인 서비스들의 조직**으로 구조화하여 목표를 달성하는 접근 방식입니다.17 이를 추진하는 주요 동기는 조직적 민첩성(작고 자율적인 팀들이 병렬적으로 작업할 수 있도록 함)과 기술적 확장성(개별 서비스를 독립적으로 확장할 수 있도록 함)입니다.18

이는 모놀리식(monolithic) 아키텍처와 대조됩니다. 모놀리식 아키텍처에서는 단일 장애점이 전체 시스템을 다운시킬 수 있으며, 확장을 위해서는 전체 애플리케이션을 확장해야만 합니다.18 MSA는 이러한 문제들을 각 서비스를 격리함으로써 해결하고자 합니다.


MSA는 다음과 같은 여러 기본 원칙 위에 구축됩니다.

- **단일 책임 / 경계 컨텍스트(Bounded Context):** 각 마이크로서비스는 단일하고 잘 정의된 비즈니스 기능에 대한 책임을 져야 합니다.17 이는 종종 도메인 주도 설계(Domain-Driven Design, DDD)의 원칙을 적용하여 달성되는데, 각 서비스는 고유의 보편 언어(ubiquitous language)와 도메인 모델을 가진 '경계 컨텍스트'에 해당합니다.18
- **자율성 및 독립적 배포성:** 각 서비스가 다른 서비스에 영향을 주지 않고 독립적으로 개발, 테스트, 배포 및 업데이트될 수 있다는 것이 핵심 원칙입니다.19 이는 종종 컨테이너화(예: Docker)와 오케스트레이션(예: Kubernetes) 기술을 통해 가능해집니다.17
- **분산된 데이터 관리:** 이는 매우 중요하고 결정적인 원칙입니다. 각 마이크로서비스는 자체 데이터를 *소유*하며, 자신만의 비공개 데이터베이스를 가져야 합니다. 다른 서비스는 오직 잘 정의된 공개 API를 통해서만 이 데이터에 접근할 수 있습니다.17 서비스 간에 데이터베이스를 공유하는 것은 강한 결합(tight coupling)을 유발하는 주요 안티패턴으로 간주됩니다.
- **분산된 거버넌스:** 팀들은 자신들의 특정 서비스에 가장 적합한 기술 스택(언어, 데이터베이스 등)을 자유롭게 선택할 수 있으며, 이는 '폴리글랏(polyglot)' 접근 방식을 장려합니다.19
- **장애를 고려한 설계:** 분산 시스템에서는 장애가 필연적입니다. MSA는 회로 차단기(circuit breaker), 재시도, 대체 작동(fallback)과 같은 복원력 패턴을 구축하여 이를 수용합니다. 이를 통해 한 서비스의 장애가 연쇄적으로 퍼져 시스템 전체의 중단을 야기하는 것을 방지합니다.18


이 원칙은 통신 인프라("파이프")가 가능한 한 단순하고 수동적이어야 한다는 것을 의미합니다(예: HTTP, 경량 메시지 브로커).18 모든 비즈니스 로직, 오케스트레이션, 그리고 지능은 서비스 자체("스마트 엔드포인트") 내에 존재해야 합니다. 이 시스템은 상당한 비즈니스 로직을 포함하는 엔터프라이즈 서비스 버스(ESB)와 같은 복잡한 미들웨어를 피합니다. 이 철학은 서비스가 복잡한 중앙 통신 허브에 의존하지 않기 때문에 서비스 자율성과 느슨한 결합을 직접적으로 지원합니다.

MSA를 더 깊이 이해하기 위해서는 기술적 측면을 넘어선 두 가지 중요한 관점이 필요합니다. 첫째, MSA는 기술적 패턴인 동시에 조직적 패턴이라는 점입니다. '비즈니스 기능 중심의 조직'이나 '분산된 거버넌스'와 같은 원칙들은 순전히 기술적인 것이 아닙니다.18 이는 조직의 커뮤니케이션 구조가 그 조직이 설계하는 시스템에 반영될 것이라는 콘웨이의 법칙(Conway's Law)을 반영합니다. MSA는 의도적으로 콘웨이의 법칙을 활용하려는 시도입니다. 서비스 하나를 처음부터 끝까지 소유하는 작고 자율적인 팀("투 피자 팀")을 만듦으로써, 아키텍처는 높은 팀 생산성과 명확한 소유권을 장려합니다.18 독립적 배포나 분산 데이터 관리와 같은 기술적 선택들은 이러한 조직적 목표를 가능하게 하는 요소들입니다. 이들은 단일 코드베이스에서 작업하는 대규모 팀을 괴롭히는 조정 오버헤드를 줄여줍니다.24 따라서 MSA의 성공은 종종 지연 시간과 같은 기술적 지표뿐만 아니라, 시장 출시 시간(time-to-market)이나 팀 자율성과 같은 기준으로 측정됩니다. 이는 주로 비기능적 기술 요구사항(신뢰성, 성능)에 의해 주도되는 DDS와는 대조적입니다.

둘째, '분산된 데이터' 원칙이 DDS와의 아키텍처적 충돌의 근원이라는 점입니다. 서비스별로 사적인 데이터 소유권을 확고히 고수하는 것은 공유된 글로벌 데이터 공간이라는 DDS의 개념과 철학적으로 가장 근본적으로 대립하는 지점입니다. MSA의 주요 목표는 독립적인 발전을 가능하게 하는 느슨한 결합을 달성하는 것입니다.19 모놀리식 시스템의 경험을 통해 가장 큰 결합의 원인이 공유 데이터베이스라는 것이 밝혀졌습니다. 한 기능을 위해 스키마를 변경하면 다른 수십 개의 기능이 망가질 수 있었습니다. 따라서 '서비스당 하나의 데이터베이스' 규칙은 MSA의 타협 불가능한 신조가 되었습니다.17 이는 데이터 수준에서의 캡슐화를 강제합니다. 반면, DDS의 '글로벌 데이터 공간'은 모든 참여자가 보고 상호작용할 수 있는 공유 데이터 모델의 아이디어를 장려합니다.10 이 충돌은 REST 대 DDS 프로토콜의 문제가 아니라, 분산 시스템에서의 데이터 관리에 대한 두 가지 정반대의 접근 방식에 관한 것입니다. MSA는 다른 무엇보다 서비스 캡슐화를 우선시하는 반면, DDS는 시스템 상태에 대한 일관되고 공유된 시각을 우선시합니다. 이 점을 이해하는 것이 MSA가 DDS를 사용하지 않는 이유에 대한 핵심적인 답변이 됩니다.


이 섹션에서는 질의와 관련된 통신 방법들에 대한 핵심적인 기술적 비교를 제공합니다. MSA와 DDS에서 사용되는 모델들을 분석하여 "왜 REST인가?"와 "왜 DDS는 아닌가?"라는 질문에 대한 직접적인 답변의 토대를 마련합니다.


이 모델은 클라이언트가 요청을 보내고 응답을 받을 때까지 대기(blocking)하는 방식입니다. 이는 시간적으로 강하게 결합된 통신 형태입니다.

- **REST (Representational State Transfer):**
  - MSA에서 특히 공개 API 및 내부 API에 대해 지배적으로 사용되는 스타일입니다.25
  - **아키텍처 모델:** 엔티티 지향적입니다. URL로 식별되는 '리소스'(명사)에 대해 제한된 동사 집합(GET, POST, PUT, DELETE)을 사용하여 작동합니다.28
  - **데이터 형식:** 일반적으로 HTTP/1.1 또는 HTTP/2를 통해 사람이 읽을 수 있는 JSON을 사용합니다. JSON의 유연성과 디버깅 용이성은 큰 장점입니다.25
  - **강점:** 단순성, 보편성, 방대한 도구 생태계(API 게이트웨이, Postman), 언어에 구애받지 않음, 캐시 가능성.25
  - **약점:** 텍스트 기반 형식과 요청당 연결 모델(HTTP/1.1에서)로 인한 성능 오버헤드, 단일(one-to-one) 통신만 가능, 시간이 지남에 따라 경직될 수 있음.25
- **gRPC (Google Remote Procedure Call):**
  - REST의 현대적이고 고성능 대안으로, 주로 내부 서비스 간 통신에 사용됩니다.26
  - **아키텍처 모델:** 서비스 지향적입니다. 클라이언트는 원격 서버의 특정 함수를 마치 로컬 메서드 호출처럼 호출합니다.28
  - **데이터 형식:** 기본적으로 바이너리 형식인 프로토콜 버퍼(Protocol Buffers, Protobuf)를 사용하며, 이는 JSON보다 작고 직렬화가 빠릅니다. HTTP/2를 통해 작동하여 멀티플렉싱과 스트리밍을 가능하게 합니다.25
  - **강점:** 고성능, 낮은 지연 시간, IDL(`.proto` 파일)을 통한 강력한 타입 지정, 단일 및 스트리밍(클라이언트, 서버, 양방향) 통신 지원.25
  - **약점:** 사람이 읽기 어려움, 더 많은 도구 필요, 제한된 네이티브 브라우저 지원(gRPC-Web 필요), 가파른 학습 곡선.25


이는 복원력과 확장성을 향상시키기 위해 MSA에서 흔히 사용되는 패턴으로, 서비스들이 메시지 브로커(예: RabbitMQ, Kafka)를 통해 통신합니다.21

- **아키텍처 모델:** 생산자(producer)가 브로커의 큐(queue)나 토픽(topic)으로 메시지를 보냅니다. 소비자(consumer)는 나중에 이 메시지를 검색합니다. 이는 서비스들을 시간적으로 분리시켜, 동시에 사용 가능할 필요가 없게 만듭니다.21
- 이 "멍청한 파이프" 접근 방식은 MSA와 잘 맞지만, 브로커 자체가 관리하기 복잡한 인프라가 될 수 있습니다.


- **아키텍처 모델:** 데이터 중심 발행-구독 모델입니다.1 이는 일반적으로 브로커가 없고 미들웨어가 데이터를 인지한다는 점에서 비동기 메시징과 구별됩니다.5
- **주요 추상화:** 불투명한 메시지나 서비스 호출이 아닌, 타입이 지정된 데이터 객체로 구성된 "글로벌 데이터 공간"입니다.10
- **통신:** P2P 방식으로, 종종 기본 RTPS(Real-Time Publish-Subscribe) 유선 프로토콜 상에서 UDP 멀티캐스트와 같은 효율적인 전송을 사용합니다.7
- **핵심 차별점:** 지능이 엔드포인트가 아닌 "파이프"(DDS 미들웨어)에 있습니다. 미들웨어는 QoS 정책에 기반하여 탐색, 라우팅, 필터링, 신뢰성을 처리합니다.3


아키텍트가 고려해야 할 핵심 특성들을 직접적이고 나란히 평가할 수 있도록, 다음 표는 앞선 분석을 밀도 높고 스캔 가능한 형태로 요약합니다. 이 표는 복잡한 서술을 실행 가능한 의사 결정 도구로 전환하여, 각 접근 방식의 고유한 강점과 약점을 명확하게 보여줍니다.

| 특성                 | REST                          | gRPC                        | 비동기 메시징 (브로커 기반)      | DDS                              |
| -------------------- | ----------------------------- | --------------------------- | -------------------------------- | -------------------------------- |
| **아키텍처 모델**    | 엔티티 지향 요청-응답         | 서비스 지향 RPC             | 발행-구독                        | 데이터 중심 발행-구독            |
| **주요 추상화**      | 리소스 (URL)                  | 원격 프로시저               | 메시지/이벤트                    | 글로벌 데이터 객체               |
| **데이터 형식**      | JSON/XML (텍스트)             | 프로토콜 버퍼 (바이너리)    | 모든 형식 (불투명)               | 타입 지정 데이터 (IDL 정의)      |
| **전송 프로토콜**    | HTTP/1.1, HTTP/2              | HTTP/2                      | TCP                              | UDP/TCP/공유 메모리 (RTPS)       |
| **결합 (시간적)**    | 강함 (동기)                   | 강함 (동기)                 | 약함 (비동기)                    | 약함 (비동기)                    |
| **미들웨어 지능**    | 낮음 ("멍청한 파이프")        | 낮음 ("멍청한 파이프")      | 중간 (브로커 로직)               | 높음 ("스마트 파이프")           |
| **탐색**             | 수동/서비스 레지스트리        | 서비스 레지스트리           | 브로커 관리                      | 자동 P2P                         |
| **성능 프로파일**    | 보통의 지연 시간, 높은 처리량 | 낮은 지연 시간, 높은 처리량 | 높은 지연 시간, 매우 높은 처리량 | 매우 낮은 지연 시간, 높은 처리량 |
| **일반적 사용 사례** | 웹 API, 모바일 클라이언트     | 내부 고성능 서비스          | 이벤트 기반 시스템, 디커플링     | 실시간 제어, 시뮬레이션, 엣지    |
| **강점**             | 단순성, 생태계, 캐시 가능성   | 성능, 스트리밍, 계약        | 복원력, 확장성, 디커플링         | 실시간, 신뢰성, 무설정           |
| **약점**             | 성능, 장황함                  | 복잡성, 브라우저 지원       | 브로커 관리, 지연 시간           | 틈새 생태계, 네트워크 제약       |


사용자의 질문은 두 아키텍처가 근본적으로 '여러 소프트웨어 모듈을 최소한의 기능으로 조직하여 목표를 달성한다'는 공통된 철학을 공유하는지에 대한 핵심을 짚고 있습니다. 표면적으로 이는 사실입니다. 두 패러다임 모두 거대한 모놀리식 시스템을 더 작고 관리하기 쉬운 단위로 분해하려는 시도입니다. 그러나 이러한 분해를 달성하는 방식과 그 과정에서 우선시하는 가치는 근본적으로 다르며, 이 차이점이 통신 메커니즘의 선택을 결정합니다. 이 섹션은 이전 섹션들을 종합하여 성공적인 두 아키텍처 패턴이 왜 그렇게 다른 통신 메커니즘으로 발전했는지를 다층적인 논거를 통해 설명합니다.


- **DDS의 기원:** 1990년대 후반과 2000년대 초, 군사, 항공 우주, 산업 제어 시스템과 같은 까다로운 분야에서 탄생했습니다.2 해결해야 할 주요 문제는 전용 및 제어된 네트워크에서의 실시간 성능, 결정론적 동작, 그리고 극도의 장애 허용성이었습니다.
- **MSA의 기원:** 2000년대 후반과 2010년대에 아마존, 넷플릭스와 같은 기업에서 대규모 웹 애플리케이션을 확장하면서 겪었던 어려움에서 비롯되었습니다.18 해결해야 할 주요 문제는 개발자 생산성, 독립적인 배포, 그리고 상용 클라우드 인프라에서의 대규모 수평적 확장성이었습니다.
- 이러한 분기점은 각 생태계가 서로 다른 목표를 위해 최적화되었음을 의미합니다. DDS는 저수준 네트워크 효율성(UDP 멀티캐스트)과 풍부한 QoS에, MSA는 개발자 경험과 공용 인터넷을 통한 상호운용성(HTTP/JSON)에 초점을 맞췄습니다.


이것이 핵심적인 개념적 충돌 지점입니다. MSA의 **분산된 데이터 관리** 원칙은 서비스의 데이터가 통제된 API를 통해서만 노출되는 비공개 구현 세부 사항이라고 주장합니다.17 이것이 느슨한 결합의 초석입니다. 반면, DDS의 

**글로벌 데이터 공간**은 참여자들이 직접 상호작용하는 공유되고 공통된 데이터 모델의 아이디어를 장려합니다.10 이것이 실시간 상태 동기화의 초석입니다.

MSA 아키텍트의 관점에서 GDS는 숨겨진 의존성을 만들고 캡슐화를 위반하는 위험한 패턴인 '전역 변수'나 분산 공유 데이터베이스처럼 보일 수 있습니다. 반면, DDS 아키텍트의 관점에서 끊임없는 API 호출을 통한 MSA의 접근 방식은 상태를 인지하지 못하고 비효율적이며 취약한 상태 동기화 방법으로 보일 수 있습니다.


앞서 확립된 바와 같이, MSA는 "스마트 엔드포인트와 멍청한 파이프"를 선호합니다.18 가치는 서비스 로직에 있습니다. DDS는 "스마트 파이프와 단순한 엔드포인트"를 구현합니다. 가치는 데이터를 안정적으로 분산 관리하는 미들웨어의 능력에 있습니다. '순수한' MSA 맥락에서 DDS를 채택하는 것은 필터링, 재시도, 상태 관리와 같은 핵심 로직을 미들웨어에 아웃소싱하는 것을 의미하며, 이는 엔드포인트 지능이라는 MSA 철학과 모순됩니다.


- **네트워크 환경:** REST와 gRPC는 보편적으로 사용 가능하고 방화벽 친화적인 HTTP 기반으로 구축되었습니다. DDS의 기본 RTPS 프로토콜은 효율적인 탐색과 데이터 분산을 위해 종종 UDP 멀티캐스트에 의존하는데, 이는 기업 및 클라우드 네트워크 환경에서 자주 차단되거나 지원되지 않습니다.7
- **개발자 경험 및 도구:** REST를 둘러싼 생태계는 방대합니다. 브라우저, `curl`, Postman, Swagger/OpenAPI 및 수많은 라이브러리 덕분에 시작하기가 매우 쉽습니다.25 gRPC 생태계도 빠르게 성장하고 있습니다. 반면, DDS 생태계는 더 전문화되어 있으며 일반적인 웹 또는 엔터프라이즈 개발자에게는 덜 친숙합니다.33
- **요청-응답 대 데이터 흐름 사고방식:** 엔터프라이즈 애플리케이션은 종종 일련의 요청과 명령어(주문 생성, 고객 상태 확인)로 자연스럽게 모델링됩니다. REST는 이러한 절차적 사고방식에 매우 잘 부합합니다.28 실시간 시스템은 종종 상태 업데이트의 연속적인 흐름(차량 위치, 센서 온도)으로 모델링됩니다. DDS는 이러한 데이터 흐름 사고방식에 완벽하게 부합합니다.1

이러한 분기점을 더 깊이 이해하면, 아키텍처 선택이 결국 복잡성을 어디에서 관리할 것인가에 대한 트레이드오프라는 결론에 도달합니다. 두 아키텍처 모두 분산 시스템의 복잡성을 관리하는 것을 목표로 합니다. MSA는 문제를 더 작고 캡슐화된 서비스로 분해함으로써 복잡성을 관리합니다. 그 결과 복잡성은 서비스 간의 상호작용(서비스 탐색, 네트워크 복원력, 분산 데이터 일관성)으로 이동하며, 이는 종종 API 게이트웨이, 서비스 메시, 사가(Saga) 패턴과 같은 방식으로 서비스 엔드포인트 내부 또는 주변에서 처리됩니다. 반면, DDS는 강력하고 중앙 집중화된(개념적으로, 구현은 분산됨) 데이터 추상화를 생성하여 복잡성을 관리합니다. 복잡성은 데이터 모델과 QoS 정책의 설계로 이동하며, 상호작용은 미들웨어에 의해 단순화됩니다. 따라서 조직이 아키텍처를 선택하는 것은 암묵적으로 어떤 종류의 복잡성을 더 잘 처리할 수 있는지를 선택하는 것과 같습니다. 강력한 DevOps 및 API 설계 기술을 갖춘 웹 네이티브 기업은 MSA 모델을 선호할 수 있습니다. 시스템 엔지니어와 실시간 제어에 중점을 둔 기업은 DDS 모델을 선호할 수 있습니다. 어느 것이 '더 낫다'의 문제가 아니라, 문제 영역과 조직의 기술에 어느 것이 더 적합한가의 문제입니다.


이 섹션에서는 마이크로서비스 아키텍처의 핵심 원칙인 '서비스별 개별 데이터베이스'를 유지하면서, 서비스 간 통신에 REST/gRPC 대신 '보안 DDS'를 사용하는 하이브리드 아키텍처의 타당성을 고찰합니다. 이는 두 아키텍처의 장점을 결합하려는 시도이지만, 동시에 근본적인 철학적 과제를 안고 있습니다.


이론적으로 이 하이브리드 모델은 다음과 같은 강력한 이점을 제공할 수 있습니다.

- **MSA의 민첩성과 DDS의 실시간 성능:** 각 서비스는 독립적으로 개발, 배포, 확장될 수 있어 MSA의 조직적 민첩성을 유지합니다.41 동시에, 서비스 간 통신은 DDS를 통해 이루어지므로 매우 낮은 지연 시간과 높은 처리량을 달성할 수 있어 고성능이 요구되는 시스템에 적합합니다.9
- **궁극의 분리(Decoupling):** MSA는 서비스 간의 느슨한 결합을 추구합니다. DDS의 데이터 중심 발행-구독 모델은 이를 한 단계 더 발전시킬 수 있습니다.33 서비스들은 서로의 존재나 위치, 심지어 API 엔드포인트조차 알 필요 없이, 오직 공통의 데이터 '토픽'에 대해서만 인지하면 됩니다.10 이는 REST/gRPC의 요청-응답 모델보다 더 강력한 시공간적 분리를 제공합니다.
- **미들웨어로 이전된 복원력:** MSA에서는 회로 차단기, 재시도 등의 복원력 패턴을 각 서비스("스마트 엔드포인트")에 구현해야 합니다. DDS를 사용하면 신뢰성(RELIABILITY), 생존성(LIVELINESS), 소유권(OWNERSHIP)과 같은 풍부한 QoS 정책을 통해 이러한 비기능적 요구사항을 통신 미들웨어("스마트 파이프") 계층에서 선언적으로 처리할 수 있습니다.9 이는 서비스의 비즈니스 로직을 단순화하는 데 기여할 수 있습니다.
- **표준화된 통합 보안 모델:** OMG DDS Security 표준은 인증, 접근 제어, 암호화, 로깅을 포함하는 포괄적인 보안 프레임워크를 제공합니다.42 이 모델은 PKI(공개 키 기반 구조)를 기반으로 하며, 거버넌스 및 권한 문서를 통해 어떤 참여자가 어떤 토픽을 발행/구독할 수 있는지 세밀하게 제어합니다.45 이를 통신 계층에 적용하면 모든 마이크로서비스에 걸쳐 일관되고 강력한 보안 정책을 강제할 수 있습니다.47


이러한 잠재적 이점에도 불구하고, 이 하이브리드 모델은 해결해야 할 심각한 과제와 근본적인 철학적 모순을 내포하고 있습니다.

- **데이터 캡슐화 대 글로벌 데이터 공간의 충돌:** 이것이 가장 큰 난제입니다. MSA의 핵심 신조는 '서비스당 하나의 데이터베이스' 원칙으로, 각 서비스의 데이터는 비공개이며 오직 API를 통해서만 접근 가능해야 한다는 것입니다.50 이는 데이터 캡슐화를 통해 서비스 간의 결합을 최소화하기 위함입니다. 반면, DDS는 모든 참여자가 접근할 수 있는 '글로벌 데이터 공간'이라는 개념을 중심으로 구축되었습니다.10 이 두 개념은 정면으로 충돌합니다. DDS의 공유 데이터 모델을 그대로 사용하면 MSA가 그토록 피하고자 했던 데이터 수준의 강한 결합이 발생할 위험이 있습니다.
- **데이터 일관성 문제의 심화:** 서비스별로 데이터베이스를 분리하면 데이터 일관성 문제가 발생하며, 이는 보통 사가(Saga) 패턴이나 이벤트 소싱(Event Sourcing)과 같은 '최종적 일관성(Eventual Consistency)' 모델로 해결합니다.50 DDS를 통신 계층으로 사용하면 이 문제는 더욱 복잡해집니다. 한 서비스가 자신의 데이터베이스를 변경한 후, 이 변경 사항을 DDS 토픽에 발행해야 합니다. 다른 서비스는 이 토픽을 구독하여 자신의 로컬 데이터베이스를 업데이트합니다. 이 과정에서 데이터 동기화, 순서 보장, 실패 시 보상 트랜잭션 처리 등 최종적 일관성을 보장하기 위한 정교한 메커니즘이 반드시 필요합니다.55
- **증가하는 복잡성:** 이 아키텍처는 두 패러다임의 복잡성을 모두 상속받습니다. 개발팀은 MSA 원칙(예: 도메인 주도 설계)과 DDS의 복잡성(예: 수많은 QoS 정책, IDL 데이터 모델링, 보안 구성)을 모두 이해하고 관리해야 합니다.41 이는 상당한 학습 곡선과 운영 오버헤드를 유발합니다.


이러한 모순을 해결하고 하이브리드 아키텍처를 실현 가능하게 만드는 가장 유력한 패턴은 **이벤트 소싱(Event Sourcing)**입니다.

1. **데이터베이스를 이벤트 저장소로 사용:** 각 마이크로서비스는 자신의 상태를 직접 저장하는 대신, 상태를 변경시킨 모든 '이벤트'의 순차적 목록을 자신의 비공개 데이터베이스(이벤트 저장소)에 저장합니다.59 이 이벤트 저장소는 해당 서비스의 유일한 '진실의 원천(Source of Truth)'이 됩니다. 이는 MSA의 '서비스별 데이터베이스 소유' 원칙을 준수합니다.50

2. **DDS를 이벤트 버스로 활용:** 서비스가 새로운 이벤트를 자신의 이벤트 저장소에 기록한 후, 해당 이벤트를 DDS 토픽으로 발행합니다.61 여기서 DDS는 단순한 데이터 공유 공간이 아니라, 신뢰성 있는 '이벤트 버스' 역할을 합니다. DDS의 

   `RELIABILITY` QoS는 이 중요한 이벤트가 유실되지 않도록 보장할 수 있습니다.

3. **구독과 상태 재구성:** 다른 서비스들은 필요한 DDS 이벤트 토픽을 구독합니다. 이벤트를 수신하면, 각 서비스는 그 정보를 바탕으로 자신의 로컬 데이터베이스에 필요한 '읽기 모델(Read Model)' 또는 '구체화된 뷰(Materialized View)'를 구축하거나 업데이트합니다.62 이를 통해 다른 서비스의 데이터에 직접 접근하지 않고도 필요한 데이터를 조회할 수 있습니다.

이벤트 소싱 패턴을 적용함으로써, '데이터 캡슐화'와 '데이터 공유'라는 모순적인 요구사항을 조화시킬 수 있습니다. 데이터의 소유권과 진실의 원천은 각 서비스의 비공개 이벤트 저장소에 캡슐화된 채로 유지되며, DDS는 이들 간의 상태 변화를 실시간으로 안정적이고 안전하게 전파하는 역할을 수행합니다. 하지만 이는 시스템 전체를 이벤트 기반의 최종적 일관성 모델로 설계해야 함을 의미하며, 이는 그 자체로 또 다른 수준의 복잡성을 도입합니다.64


이 섹션은 "...아니면 DDS를 사용하는가?"라는 사용자의 마지막 질문에 직접적으로 답하며, 이 두 세계가 상호 배타적인 것이 아니라 상호 보완적인 시나리오를 탐구합니다.


특정 도메인에서는 비기능적 요구사항이 너무 까다로워 REST/gRPC가 서비스 간 통신에 부적합할 수 있습니다.

- **사용 사례: 산업용 사물 인터넷(IIoT) 및 자율 시스템:** 스마트 팩토리나 자율 주행 차량에서 인식, 계획, 제어와 같은 서비스들은 마이크로초 수준의 지연 시간으로 방대한 양의 데이터를 교환해야 합니다. DDS는 이러한 "데이터 버스"에 대한 자연스러운 선택입니다.7 여기서 "마이크로서비스"는 DDS 백본을 통해 통신하는 실시간 구성 요소들입니다.
- **사용 사례: 금융 거래 및 복잡 이벤트 처리:** 고빈도 시장 데이터를 처리하고 반응해야 하는 시스템은 DDS를 사용하여 처리 노드 간에 저지연 데이터 패브릭을 생성할 수 있습니다.


가장 일반적인 융합 패턴은 REST를 DDS로 대체하는 것이 아니라, 계층화된 아키텍처에서 이들을 함께 사용하는 것입니다.34

- **엣지-투-클라우드(Edge-to-Cloud) 패턴:**
  - **엣지(Edge)에서:** DDS는 실시간, 장치 대 장치(D2D) 및 장치 대 게이트웨이(D2G) 통신에 사용됩니다. 이는 운영 기술(OT)을 위한 복원력 있는 P2P 데이터 패브릭을 형성합니다. 원시적이고 고빈도의 데이터가 존재하는 곳입니다.10
  - **클라우드(Cloud)에서:** 표준 MSA가 REST/gRPC/메시징과 함께 엔터프라이즈 IT 시스템에 사용됩니다. 이러한 서비스는 데이터 집계, 분석, 장기 저장 및 비즈니스 사용자와 애플리케이션에 데이터 노출을 처리합니다.35
- **DDS-REST 게이트웨이: 중요한 다리 역할**
  - 이 두 세계를 연결하기 위해 **DDS-REST 게이트웨이**는 일반적인 아키텍처 패턴입니다.37
  - 이 게이트웨이는 한쪽에서는 DDS 도메인의 참여자이면서 다른 쪽에서는 RESTful API를 노출하는 특수 서비스입니다.
  - **기능:** DDS 토픽을 구독하고 데이터를 집계/필터링하여 REST GET 엔드포인트를 통해 노출합니다. 또한 REST POST/PUT 명령을 수락하여 이를 DDS 발행으로 변환하여 엣지의 장치를 제어합니다.37 이를 통해 웹 애플리케이션은 DDS를 이해할 필요 없이 안전하게 실시간 세계와 상호작용할 수 있습니다.

이러한 융합 패턴은 현대 기술이 산업 기업에서 오랫동안 존재해 온 조직적 분할, 즉 운영 기술(OT)과 정보 기술(IT)의 분할을 반영한다는 점에서 흥미롭습니다. 산업 조직들은 역사적으로 공장 현장의 실시간 제어 시스템을 담당하는 OT와 엔터프라이즈 비즈니스 시스템을 담당하는 IT 사이에 명확한 구분을 두어 왔습니다. OT는 신뢰성, 안전성, 결정론적 동작을 우선시하며, 이는 DDS의 특성과 완벽하게 일치합니다.9 IT는 데이터 통합, 확장성, 민첩성을 우선시하며, 이는 MSA의 특성과 완벽하게 일치합니다.18 DDS-REST 게이트웨이를 갖춘 '엣지-투-클라우드' 아키텍처는 바로 이 OT/IT 분할을 연결하는 현대적인 아키텍처 패턴입니다. DDS는 OT의 언어이고, REST는 IT의 언어이며, 게이트웨이는 그 사이의 번역가 역할을 합니다. 이러한 조직 구조의 렌즈를 통해 이 융합을 바라보면, 기술 아키텍처가 역사적으로 분리되었던 두 비즈니스 부문 간의 보다 원활한 통합을 가능하게 하고 있음을 더 쉽게 이해할 수 있습니다.


이 마지막 섹션에서는 보고서 전체를 종합하여 연구 결과를 간결하게 요약하고, 이러한 결정을 내려야 하는 아키텍트에게 명확하고 실행 가능한 지침을 제공합니다.


- 로봇 공학/항공 우주를 위한 아키텍처는 **DDS 표준**에 의해 구현되는 **데이터 중심 발행-구독** 모델입니다. 그 강점은 QoS 정책을 사용하여 "글로벌 데이터 공간" 추상화를 통해 실시간 성능과 장애 허용성을 제공하는 "스마트" 미들웨어에 있습니다.
- **마이크로서비스 아키텍처**는 복원력과 모듈성이라는 유사한 목표를 추구하지만, "스마트 엔드포인트, 멍청한 파이프"라는 다른 철학과 엄격한 데이터 캡슐화 및 분산된 데이터 소유권이라는 핵심 원칙을 통해 이를 달성합니다.
- MSA는 주로 웹 기반 엔터프라이즈 애플리케이션의 요청-응답 모델과의 정합성, 단순성, 보편성 때문에 **REST API**를 사용합니다. 데이터 관리(공유 상태 대 캡슐화)에 대한 근본적인 철학적 충돌과 실용적인 생태계/네트워크 장벽 때문에 DDS를 사용하지 않습니다.
- **보안 DDS와 개별 데이터베이스를 갖춘 MSA의 결합**은 이론적으로 실시간 성능과 민첩성의 조화를 이룰 수 있지만, 데이터 캡슐화와 공유 데이터 모델 간의 근본적인 철학적 충돌이라는 큰 과제를 안고 있습니다. **이벤트 소싱** 패턴은 이 충돌을 해결할 수 있는 유력한 방안이지만, 시스템 전체의 복잡성을 크게 증가시킵니다.
- 융합은 주로 **하이브리드 아키텍처**(예: IoT)에서 일어나고 있으며, 여기서 DDS는 운영 엣지에서의 실시간 데이터 패브릭 역할을 하고, MSA는 클라우드에서의 확장 가능한 비즈니스 로직 계층 역할을 하며, 이 둘은 **DDS-REST 게이트웨이**를 통해 연결됩니다.


다음은 아키텍트를 위한 실용적인 가이드입니다. 통신 패러다임의 선택은 시스템의 주요 비기능적 요구사항과 도메인 컨텍스트에 따라 결정되어야 합니다.

- **REST 기반 MSA를 선택해야 하는 경우:**
  - 주요 동인이 개발자 민첩성과 시장 출시 시간일 때.
  - 시스템이 절차적, 요청-응답 워크플로우를 가진 웹 기반 엔터프라이즈 애플리케이션일 때.
  - 다양한 클라이언트에게 공개 API를 노출해야 할 때.
  - 팀의 전문 분야가 웹 기술일 때.
- **서비스 간 통신에 gRPC를 선택해야 하는 경우:**
  - MSA 내에서 내부 성능과 낮은 지연 시간이 중요할 때.
  - 서비스 간 스트리밍 기능이 필요할 때.
  - 폴리글랏 환경에서 강력한 계약 우선 API 정의의 이점을 누릴 수 있을 때.
- **DDS 기반 아키텍처를 선택해야 하는 경우:**
  - 시스템이 실시간 제어 루프, 시뮬레이션 또는 자율 시스템일 때.
  - 마이크로초 수준의 지연 시간, 결정론적 동작, 극도의 장애 허용성이 타협 불가능한 요구사항일 때.
  - 문제의 핵심이 다수의 참여자 간에 공유된 상태 이해를 동기화하는 것일 때.
  - UDP 멀티캐스트와 같은 전송이 가능한 제어된 네트워크에서 운영될 때.
- **하이브리드 (DDS + REST/MSA) 아키텍처를 선택해야 하는 경우:**
  - 물리적 세계에서 엔터프라이즈 클라우드에 이르는 엔드투엔드 IoT 또는 사이버-물리 시스템을 구축할 때.
  - 실시간 운영 기술(OT)과 확장 가능한 정보 기술(IT) 간의 격차를 해소해야 할 때.


1. OMG Data-Distribution Service: Architectural Overview - RTI Community, accessed July 12, 2025, https://d2vkrkwbbxbylk.cloudfront.net/sites/default/files/DDS_Architectural_Overview.pdf
2. OMG Data-Distribution Service (DDS): Architectural Overview, accessed July 12, 2025, https://archive.ll.mit.edu/HPEC/agendas/proc04/abstracts/pardo-castellote_gerardo.pdf
3. Data Distribution Service for Complex Systems | DDS Standard | RTI, accessed July 12, 2025, https://www.rti.com/products/dds-standard
4. Data Distribution Service - Wikipedia, accessed July 12, 2025, https://en.wikipedia.org/wiki/Data_Distribution_Service
5. DDS Protocol in IoT: Features, Architecture, Working, and Applications - OutRight Store, accessed July 12, 2025, https://store.outrightcrm.com/blog/dds-protocol-in-iot/
6. Towards an Open, Distributed Software Architecture for UxS Operations, accessed July 12, 2025, https://ntrs.nasa.gov/api/citations/20160006099/downloads/20160006099.pdf
7. DDS Middleware. What is DDS middleware: | by Huseyin Kutluca | Software Architecture Foundations | Medium, accessed July 12, 2025, https://medium.com/software-architecture-foundations/dds-middleware-1af525f69753
8. A Systematic Literature Review of DDS Middleware in Robotic Systems - MDPI, accessed July 12, 2025, https://www.mdpi.com/2218-6581/14/5/63
9. DDS and MQTT: Basics, Challenges and Integration Benefits | EMQ - EMQX, accessed July 12, 2025, https://www.emqx.com/en/blog/navigating-dds-basics-limitations-and-integration-with-mqtt
10. What is DDS? - DDS Foundation, accessed July 12, 2025, https://www.dds-foundation.org/what-is-dds-3/
11. Data-Centric Programming Best Practices: - RTI, accessed July 12, 2025, https://www.rti.com/hubfs/docs/DDS_Best_Practices_WP.pdf
12. BMDD: a novel approach for IoT platform (broker-less and microservice architecture, decentralized identity, and dynamic transmission messages) - PubMed Central, accessed July 12, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9044371/
13. 1.1. What is DDS? - 3.2.2 - eProsima Fast DDS, accessed July 12, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/getting_started/definitions.html
14. Data-Centric Publish–Subscribe (DCPS) structure. - ResearchGate, accessed July 12, 2025, https://www.researchgate.net/figure/Data-Centric-Publish-Subscribe-DCPS-structure_fig1_340550044
15. Data-centric architectural best practices: Using DDS to integrate real-world distributed systems, accessed July 12, 2025, https://militaryembedded.com/comms/communications/data-centric-real-world-distributed-systems
16. A DDS based real-time simulation architecture for space robotic tele-operation, accessed July 12, 2025, https://www.researchgate.net/publication/288210434_A_DDS_based_real-time_simulation_architecture_for_space_robotic_tele-operation
17. 13 Microservices Best Practices - Oso, accessed July 12, 2025, https://www.osohq.com/learn/microservices-best-practices
18. Microservices architecture and design: A complete overview - vFunction, accessed July 12, 2025, https://vfunction.com/blog/microservices-architecture-guide/
19. 6 Principles Of Microservice Architecture - SayOne Technologies, accessed July 12, 2025, https://www.sayonetech.com/blog/principles-of-microservice-architecture/
20. 5 design principles for microservices | Red Hat Developer, accessed July 12, 2025, https://developers.redhat.com/articles/2022/01/11/5-design-principles-microservices
21. Microservices Architecture: Principles, Patterns, and Challenges for ..., accessed July 12, 2025, https://medium.com/@erickzanetti/microservices-architecture-principles-patterns-and-challenges-for-scalable-systems-9eac65b97b21
22. Microservices Design Patterns: Essential Guide - DZone, accessed July 12, 2025, https://dzone.com/articles/design-patterns-for-microservices
23. Designing a DDD-oriented microservice - .NET | Microsoft Learn, accessed July 12, 2025, https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/ddd-oriented-microservice
24. Microservice Architecture Principles - AKF Partners, accessed July 12, 2025, https://akfpartners.com/growth-blog/microservice-architecture-principles
25. A Deep Dive into Communication Styles for Microservices: REST vs. gRPC vs. Message Queues | by Platform Engineers | Medium, accessed July 12, 2025, https://medium.com/@platform.engineers/a-deep-dive-into-communication-styles-for-microservices-rest-vs-grpc-vs-message-queues-ea72011173b3
26. Let's talk about Microservice architecture and communication between the services. - Reddit, accessed July 12, 2025, https://www.reddit.com/r/developersIndia/comments/1aeigwx/lets_talk_about_microservice_architecture_and/
27. REST APIs vs Microservices: Key Differences - DreamFactory Blog, accessed July 12, 2025, https://blog.dreamfactory.com/restful-api-and-microservices-the-differences-and-how-they-work-together
28. gRPC vs REST - Difference Between Application Designs - AWS, accessed July 12, 2025, https://aws.amazon.com/compare/the-difference-between-grpc-and-rest/
29. Performance comparison: REST vs gRPC vs asynchronous communication - L3montree, accessed July 12, 2025, https://l3montree.com/blog/performance-comparison-rest-vs-grpc-vs-asynchronous-communication
30. Microservices Interservice Communication: REST, gRPC, MQTT - eInfochips, accessed July 12, 2025, https://www.einfochips.com/blog/interservice-communication-for-microservices/
31. REST vs gRPC in Microservices | Which One Should You Use? (Explained Clearly), accessed July 12, 2025, https://www.youtube.com/watch?v=AMNWLz_f6qM
32. AWS re:Invent 2021 - Application integration patterns for microservices - YouTube, accessed July 12, 2025, https://www.youtube.com/watch?v=QhfuzEkN3Ck
33. Exploring DDS: The Future of Software Architecture in Microservices | by Salim Elakoui, accessed July 12, 2025, https://medium.com/@salim.elakoui/exploring-dds-the-future-of-software-architecture-in-microservices-246af1fe1586
34. Understanding IoT Protocols – Matching your Requirements to the Right Option - Solace, accessed July 12, 2025, https://solace.com/blog/understanding-iot-protocols-matching-requirements-right-option/
35. Exploring the Potential of Microservices in Internet of Things: A Systematic Review of Security and Prospects - MDPI, accessed July 12, 2025, https://www.mdpi.com/1424-8220/24/20/6771
36. A Microservice Approach to IoT Edge Computing - Linux Foundation, accessed July 12, 2025, https://events19.linuxfoundation.cn/wp-content/uploads/2017/11/EdgeX-Foundry-A-Microservice-Approach-to-IoT-Edge-Computing-_Jim-White.pdf
37. Custom Bridges Between RESTful Web Services and DDS | Object Computing, Inc., accessed July 12, 2025, https://objectcomputing.com/resources/publications/sett/custom-bridges-between-restful-web-services-and-dds
38. Restful DDS execution - Stack Overflow, accessed July 12, 2025, https://stackoverflow.com/questions/11624988/restful-dds-execution
39. Overall software architecture of DDS gateway. - ResearchGate, accessed July 12, 2025, https://www.researchgate.net/figure/Overall-software-architecture-of-DDS-gateway_fig9_355374149
40. 1. Introduction - RTI Connext Gateway 7.3.0 documentation, accessed July 12, 2025, https://community.rti.com/static/documentation/gateway/current/introduction.html
41. 5 Advantages of Microservices [+ Disadvantages] - Atlassian, accessed July 12, 2025, https://www.atlassian.com/microservices/cloud-computing/advantages-of-microservices
42. How DDS Works? - DDS Foundation, accessed July 12, 2025, https://www.dds-foundation.org/why-choose-omg-dds-standard/
43. Microservices Security: Best Practices & Patterns - Daily.dev, accessed July 12, 2025, https://daily.dev/blog/microservices-security-best-practices-and-patterns
44. What's in the DDS Standard? - DDS Foundation, accessed July 12, 2025, https://www.dds-foundation.org/omg-dds-standard/
45. DDS Security - OpenDDS 3.28.1, accessed July 12, 2025, https://opendds.readthedocs.io/en/dds-3.28.1/devguide/dds_security.html
46. ROS 2 DDS-Security integration - ROS2 Design, accessed July 12, 2025, https://design.ros2.org/articles/ros2_dds_security.html
47. 5 Ways to Secure Your Microservices Architectures - Control Plane, accessed July 12, 2025, https://controlplane.com/community-blog/post/5-ways-to-secure-your-microservices-architectures
48. 9 Microservices Security Best Practices 2025 - Oso, accessed July 12, 2025, https://www.osohq.com/learn/microservices-security
49. How do you implement security in a microservices architecture?, accessed July 12, 2025, https://www.designgurus.io/answers/detail/how-do-you-implement-security-in-a-microservices-architecture
50. Data considerations for microservices - Azure Architecture Center | Microsoft Learn, accessed July 12, 2025, https://learn.microsoft.com/en-us/azure/architecture/microservices/design/data-considerations
51. Pattern: Database per service - Microservices.io, accessed July 12, 2025, https://microservices.io/patterns/data/database-per-service.html
52. Does Microservices architecture requires a database for each one - Reddit, accessed July 12, 2025, https://www.reddit.com/r/microservices/comments/17p2zrk/does_microservices_architecture_requires_a/
53. Do you really implement Microservices with their own Database? - Reddit, accessed July 12, 2025, https://www.reddit.com/r/microservices/comments/13m54v7/do_you_really_implement_microservices_with_their/
54. How do you manage data consistency in a microservices architecture? - Design Gurus, accessed July 12, 2025, https://www.designgurus.io/answers/detail/how-do-you-manage-data-consistency-in-a-microservices-architecture
55. Sharing Data Between Microservices | by Denat Hoxha - Medium, accessed July 12, 2025, https://medium.com/@denhox/sharing-data-between-microservices-fe7fb9471208
56. Data Consistency in Microservices Architecture | by Dilfuruz Kizilpinar | Medium, accessed July 12, 2025, https://dilfuruz.medium.com/data-consistency-in-microservices-architecture-5c67e0f65256
57. Microservices: The Pros and Cons | Core dna, accessed July 12, 2025, https://www.coredna.com/blogs/microservices-pros-cons
58. What are the pros and cons of using a microservice architecture when building a new enterprise web application? - Reddit, accessed July 12, 2025, https://www.reddit.com/r/webdev/comments/s6icid/what_are_the_pros_and_cons_of_using_a/
59. Event Sourcing pattern - Azure Architecture Center | Microsoft Learn, accessed July 12, 2025, https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing
60. Pattern: Event sourcing - Microservices.io, accessed July 12, 2025, https://microservices.io/patterns/data/event-sourcing.html
61. Microservices Patterns: Event Sourcing | Cloud Native Daily - Medium, accessed July 12, 2025, https://medium.com/cloud-native-daily/microservices-patterns-event-sourcing-7c6e765681c1
62. Applying CQRS and Event Sourcing in Microservices - AppMaster, accessed July 12, 2025, https://appmaster.io/blog/cqrs-event-sourcing-microservices
63. CQRS pattern - AWS Prescriptive Guidance, accessed July 12, 2025, https://docs.aws.amazon.com/prescriptive-guidance/latest/modernization-data-persistence/cqrs-pattern.html

What is the Event Sourcing Pattern? | Designing Event-Driven Microservices - YouTube, accessed July 12, 2025, https://www.youtube.com/watch?v=wPwD9CQAGsk




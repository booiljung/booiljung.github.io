# DDS 통합 서비스에 대한 종합적 분석 및 구현 가이드

## 1.  기반 - DDS와 통합의 필요성 이해

현대의 분산 시스템은 단순한 데이터 교환을 넘어 실시간성, 신뢰성, 확장성이라는 복잡한 요구사항을 충족해야 합니다. 이러한 배경 속에서 데이터 분산 서비스(Data Distribution Service, DDS)는 단순한 메시징 프로토콜을 넘어 데이터 중심(Data-Centric) 패러다임을 제시하며 미션 크리티컬 시스템의 핵심 통신 프레임워크로 자리 잡았습니다. 그러나 시스템이 더욱 복잡해지고 이기종 프로토콜과의 연동이 필수적이 되면서, DDS의 강력한 내부 통신 능력을 외부 세계와 연결하는 '통합 서비스'의 필요성이 대두되었습니다. 본 보고서의 첫 번째 파트에서는 DDS의 근본적인 철학과 핵심 메커니즘을 분석하고, 이를 통해 왜 전문화된 통합 서비스가 필수불가결한 아키텍처 구성 요소가 되었는지 그 필연성을 탐구합니다.

### 1.1  DDS의 데이터 중심 패러다임

DDS를 이해하는 첫걸음은 이를 전통적인 메시지 지향 미들웨어(Message-Oriented Middleware, MOM)와 구별하는 것입니다. DDS는 메시지 전송이 아닌 데이터 자체의 상태 공유에 초점을 맞춤으로써 분산 시스템 설계의 근본적인 관점을 전환합니다.

#### 1.1.1  실시간 시스템에서의 발행-구독 모델 해부

DDS의 통신 모델은 발행-구독(Publish-Subscribe) 패턴에 기반합니다.1 이 모델에서 정보를 생성하는 노드(발행자, Publisher)는 특정 주제(Topic)에 대한 데이터를 발행하고, 해당 정보에 관심 있는 노드(구독자, Subscriber)는 이를 구독하여 데이터를 수신합니다. 이 방식의 가장 큰 장점은 시간과 공간에 대한 느슨한 결합(Loose Coupling)입니다. 발행자와 구독자는 서로의 존재나 위치를 미리 알 필요 없이 동적으로 서로를 발견하고 통신할 수 있습니다.3

여기서 DDS의 핵심적인 차별성이 드러납니다. DDS는 메시지 중심(Message-Centric)이 아닌 데이터 중심(Data-Centric)입니다.4 개발자는 더 이상 저수준의 네트워크 프로그래밍, 즉 소켓 관리, 데이터 마샬링/언마샬링, 재전송 로직 등에 대해 고민할 필요가 없습니다.2 대신, 애플리케이션은 OMG(Object Management Group) 표준에 의해 정의된 고수준의 추상적 객체들을 통해 데이터와 상호작용합니다. 이 객체들은 

`DomainParticipant`, `Topic`, `DataWriter`, `DataReader` 등으로 구성되며, 분산 시스템의 모든 통신은 이들을 통해 이루어집니다.4

`DataWriter`는 데이터를 발행하는 역할을, `DataReader`는 데이터를 수신하는 역할을 담당하며, 이들은 특정 `Topic`과 연관됩니다.3 이러한 추상화는 개발자가 애플리케이션 로직 자체에 집중할 수 있도록 해줍니다.

#### 1.1.2  글로벌 데이터 공간: 공유 상태 추상화

DDS의 데이터 중심 철학은 '글로벌 데이터 공간(Global Data Space)'이라는 개념으로 구체화됩니다.4 이는 물리적으로 분산된 노드들이 마치 하나의 거대한 공유 메모리나 가상 데이터베이스에 접근하는 것처럼 데이터를 읽고 쓸 수 있도록 하는 강력한 추상화입니다. 시스템의 가장 최신 상태(most-current value)가 이 논리적 공간에 존재하며, 미들웨어는 각 참여자에게 필요한 데이터의 부분을 복제하고 일관성을 유지하는 복잡한 작업을 투명하게 처리합니다.4

애플리케이션이 정보를 이 공간에 기여하고자 할 때는 '발행자(Publisher)'가 되고, 공간의 일부에 접근하고자 할 때는 '구독자(Subscriber)'가 됩니다.8 발행자가 새로운 데이터를 게시할 때마다 미들웨어는 모든 관심 있는 구독자에게 정보를 전파합니다. 이 과정에서 개발자는 메시지를 누가 받을지, 구독자가 어디에 있는지, 메시지 전달 실패 시 어떻게 처리할지와 같은 문제들을 신경 쓸 필요가 없습니다.2

이 글로벌 데이터 공간 내에서 데이터 스트림을 식별하는 기본 단위는 '토픽(Topic)'입니다. 토픽은 특정 데이터 항목을 고유하게 식별하는 이름과 데이터 구조를 정의하는 타입(Type)으로 구성됩니다.8 예를 들어, 항공 관제 시스템에서 'AircraftTrack'이라는 토픽이 있다면, 각 항공기의 개별 항적 데이터는 '키(Key)'라는 고유 식별자를 통해 구분될 수 있습니다.4 키는 데이터 객체 내의 단일 필드(예: 일련번호)일 수도 있고, 여러 필드의 조합(예: 항공사명과 항공편 번호)일 수도 있습니다. 이 키 메커니즘은 DDS가 복잡하고 구조화된 데이터를 효율적으로 관리하고 확장성을 확보하는 핵심적인 방법입니다.4

이러한 아키텍처는 DDS 시스템 설계가 네트워크 토폴로지 설계보다는 데이터베이스 스키마 설계와 더 유사하다는 중요한 관점을 제시합니다. 설계의 기본 단위는 애플리케이션 간의 통신 그래프가 아니라, 인터페이스 정의 언어(IDL)를 통해 정의되는 시스템의 '데이터 모델'입니다. 이는 시스템의 진화 가능성에 지대한 영향을 미칩니다. 새로운 애플리케이션은 기존 발행자를 수정하지 않고도 데이터버스에 참여하여 기존 데이터를 활용할 수 있기 때문입니다.

#### 1.1.3  상호운용성 달성을 위한 RTPS 와이어 프로토콜의 역할

DDS 표준 초기에는 API와 동작 방식은 정의했지만, 네트워크 상에서 메시지를 교환하기 위한 전송 프로토콜, 즉 와이어 프로토콜(Wire Protocol)을 명시하지 않았습니다.1 이로 인해 여러 벤더가 각자의 방식으로 DDS를 구현했고, 서로 다른 벤더의 DDS 제품 간에는 통신이 불가능한 문제가 발생했습니다. 이는 벤더 종속성을 야기하고 개방형 시스템 구축에 큰 걸림돌이 되었습니다.

이러한 상호운용성 문제를 해결하기 위해 OMG는 DDS의 표준 와이어 프로토콜로 RTPS(Real-Time Publish-Subscribe)를 채택했습니다.1 RTPS는 본래 산업 자동화 분야에서 사용되던 검증된 프로토콜로, DDS가 목표로 하는 데이터 분산 시스템의 요구사항을 잘 만족시켰습니다.1 RTPS의 도입은 DDS 생태계에 전환점을 가져왔습니다. 2009년부터 여러 DDS 벤더들이 RTPS 프로토콜을 구현한 제품들 간의 상호운용성을 시연하기 시작했으며, 이는 DDS가 진정한 개방형 국제 표준으로 자리매김하는 데 결정적인 역할을 했습니다.2

API와 동작을 정의하는 DDS 표준과 와이어 프로토콜을 정의하는 RTPS 표준을 분리한 것은 매우 전략적인 결정이었습니다. 이는 '관심사 분리(Separation of Concerns)' 원칙의 훌륭한 예시로, 두 개의 다른 계층에서 독립적인 혁신을 가능하게 했습니다. DDS 표준은 애플리케이션이 필요로 하는 풍부한 데이터 중심의 의미론과 QoS 계약에 집중할 수 있었고, RTPS는 서로 다른 벤더 구현체 간에 안정적이고 효율적으로 비트를 전송하는 저수준의 문제를 해결하는 데 집중할 수 있었습니다. 이러한 분리 덕분에 오늘날과 같이 다양한 벤더 제품이 공존하며 상호운용되는 개방형 DDS 생태계가 구축될 수 있었습니다.1

### 1.2  서비스 품질(QoS)을 통한 데이터 흐름 제어

DDS를 단순한 데이터 전송 파이프라인에서 예측 가능하고 미션 크리티컬한 시스템을 구축하기 위한 프레임워크로 격상시키는 가장 중요하고 강력한 기능은 바로 서비스 품질(Quality of Service, QoS)입니다. QoS는 개발자가 분산 애플리케이션 간의 데이터 분산 방식과 시점을 매우 세밀하게 구성할 수 있도록 하는 표준화된 메커니즘입니다.9

#### 1.2.1  핵심 QoS 정책 심층 분석

DDS는 20가지가 넘는 다양한 QoS 정책을 제공하며, 각 엔티티(Topic, DataWriter, DataReader 등)는 지원하는 정책의 하위 집합을 가집니다.7 이 정책들을 통해 시스템의 비기능적 요구사항을 충족시킬 수 있습니다. 다음은 가장 핵심적인 QoS 정책들입니다.

- **신뢰성 (Reliability):** 데이터 전송의 보장 수준을 결정합니다. `RELIABLE`은 모든 데이터 샘플이 구독자에게 전달되도록 보장하며, 손실된 패킷은 재전송합니다. 반면 `BEST_EFFORT`는 최대한 노력하지만 데이터 전달을 보장하지는 않으며, 이로 인해 지연 시간이 짧고 오버헤드가 적습니다. 이는 빠른 속도가 신뢰성보다 중요한 경우에 사용됩니다.1
- **지속성 (Durability):** 발행된 데이터의 생명주기를 제어하여, 늦게 참여하는(late-joining) 구독자에게 데이터를 제공할지 여부를 결정합니다. `VOLATILE`은 발행 시점에 활성화된 구독자에게만 데이터를 전송합니다. `TRANSIENT_LOCAL`은 발행자가 데이터를 로컬에 저장하여, 발행자가 살아있는 동안 늦게 참여한 구독자에게 최신 값을 제공합니다. `TRANSIENT`와 `PERSISTENT`는 데이터를 발행자 외부의 영속적인 서비스에 저장하여 시스템 전체가 재시작된 후에도 데이터를 사용할 수 있게 합니다.11
- **이력 (History):** 특정 데이터 인스턴스에 대해 DataWriter나 DataReader가 얼마나 많은 데이터 샘플을 보관할지를 결정합니다. `KEEP_LAST`는 지정된 '깊이(depth)'만큼의 최신 샘플만 유지합니다. `KEEP_ALL`은 리소스 한계에 도달할 때까지 모든 샘플을 보관합니다. 이는 신규 구독자가 과거 데이터를 수신해야 할 때 유용합니다.10
- **활성도 (Liveliness):** 참여자들이 여전히 활성 상태인지(alive)를 감지하는 메커니즘을 제어합니다. `lease_duration`으로 지정된 시간 동안 데이터 발행이나 활성도 신호가 없으면 해당 참여자는 비활성 상태로 간주되어, 시스템이 노드의 장애를 감지하고 대처할 수 있게 합니다.10
- **마감일 (Deadline):** 데이터 샘플의 최대 도착 간격을 설정합니다. 구독자는 이 정책을 통해 특정 주기로 데이터가 도착할 것을 기대하며, 만약 마감일이 지켜지지 않으면 미들웨어가 이를 감지하고 애플리케이션에 통지합니다. 이는 실시간 제어 루프와 같이 주기적인 데이터 업데이트가 필수적인 애플리케이션에 매우 중요합니다.9
- **소유권 (Ownership):** 동일한 토픽 인스턴스에 대해 여러 DataWriter가 존재할 경우, 어떤 DataWriter의 데이터를 DataReader가 수신할지를 결정합니다. `OWNERSHIP_STRENGTH` 정책과 함께 사용되어, 가장 높은 강도를 가진 DataWriter가 해당 인스턴스의 유일한 소유자가 됩니다.9

#### 1.2.2  요청-제공(RxO) 계약: 예측 가능한 통합의 기반

DDS의 예측 가능성은 요청-제공(Request-versus-Offered, RxO)이라는 핵심적인 매칭 메커니즘에서 비롯됩니다.7 이는 DataReader와 DataWriter 간의 연결이 형성되기 전에 일어나는 일종의 '계약 협상' 과정입니다.

DataReader는 자신이 통신을 위해 '요청(request)'하는 최소한의 QoS 수준을 명시합니다. 반대로 DataWriter는 자신이 '제공(offer)'할 수 있는 QoS 수준을 명시합니다. DDS 미들웨어는 이 둘을 매칭시킬 때, 제공된 QoS가 요청된 QoS와 호환되는지, 즉 요청을 만족시킬 수 있는지를 확인합니다. 만약 호환되지 않으면, 두 엔티티 간의 통신은 설정되지 않으며, 애플리케이션은 비호환성 상태에 대한 알림을 받을 수 있습니다.7

예를 들어, 한 DataReader가 `RELIABILITY` QoS를 `RELIABLE`로 '요청'했다면, 이 DataReader는 `BEST_EFFORT`로 '제공'하는 DataWriter와는 절대로 연결되지 않습니다. 마찬가지로, DataReader가 10ms의 `DEADLINE`을 '요청'했는데 DataWriter가 5초의 `DEADLINE`을 '제공'한다면(즉, 5초에 한 번씩만 데이터를 보내겠다고 약속한다면), 이 또한 호환되지 않으므로 연결이 성립되지 않습니다.9

이 RxO 모델은 시스템 통합에서 매우 중요한 역할을 합니다. 이는 신뢰성이나 적시성과 같은 비기능적 요구사항을 코드 내의 암묵적인 가정으로 남겨두는 대신, 미들웨어에 의해 강제되고 검증되는 명시적인 '서비스 수준 협약(Service Level Agreement, SLA)'으로 전환시킵니다. 이로써 개발자는 "데이터가 도착하지 않거나 너무 늦게 도착하면 어떡하지?"와 같은 문제를 고민하며 방어적인 코드를 작성하는 대신, 시스템의 동작 보증을 선언적으로 정의할 수 있습니다. 이는 "조용한 실패(silent failure)", 즉 애플리케이션이 모르는 사이에 데이터가 유실되거나 지연되는 상황을 원천적으로 방지합니다.

#### 1.2.3  QoS 구성: 선언적 XML에서 프로그래밍 방식 제어까지

DDS의 QoS 정책은 주로 외부 XML 파일을 통해 선언적으로 구성됩니다.13 이 방식은 애플리케이션 코드를 재컴파일하지 않고도 시스템의 동작을 튜닝할 수 있게 해주는 큰 장점이 있습니다. 시스템 관리자는 배포 환경이나 요구사항의 변화에 따라 XML 파일만 수정하여 데이터 흐름의 신뢰성, 실시간성, 리소스 사용량 등을 조절할 수 있습니다.

물론, 프로그래밍 방식을 통해 동적으로 QoS를 설정하는 것도 가능합니다. 애플리케이션 코드는 엔티티 팩토리(예: `DomainParticipant`)로부터 기본 QoS 값을 가져와 필요한 정책을 수정한 후, 이를 엔티티 생성 시 인자로 전달할 수 있습니다.10 일부 QoS 정책은 엔티티가 생성된 후에도 변경할 수 있지만(mutable), 다른 정책들은 변경이 불가능(immutable)하며, 이는 통신 호환성에 영향을 미칩니다.10

다양한 QoS 정책들은 독립적이지 않으며, 서로 간에 복잡한 트레이드오프 관계를 형성합니다. 예를 들어, `RELIABILITY.RELIABLE`과 `HISTORY.KEEP_ALL` 정책은 강력한 데이터 보존 및 전달을 보장하지만, 더 많은 메모리와 네트워크 대역폭을 소모합니다.10 이는 엄격한 

`LATENCY_BUDGET`이나 `DEADLINE` 요구사항과 상충될 수 있습니다.11 이것은 DDS의 약점이 아니라, 분산 시스템의 근본적인 트레이드오프를 설계자에게 명시적으로 노출시키는 기능입니다. 따라서 '최고의' QoS 프로파일이란 존재하지 않으며, 각 데이터 스트림(예: 제어 명령, 상태 보고, 비디오 스트림)의 고유한 요구사항에 따라 신중하게 균형을 맞춘 최적의 구성이 존재할 뿐입니다. QoS 설계는 단순한 튜닝이 아니라, 시스템 아키텍처 설계의 핵심적인 초기 활동이어야 합니다.

## 2.  브릿지 - 통합 서비스의 아키텍처와 구현

DDS가 제공하는 강력한 데이터 중심 통신은 그 자체로 완결된 생태계를 이룹니다. 그러나 현대의 복잡한 시스템은 단일 기술만으로 구축되지 않습니다. 다양한 프로토콜과 기술 스택으로 구성된 이기종(heterogeneous) 환경에서 DDS 기반 시스템을 외부 세계와 연결하는 것은 피할 수 없는 과제입니다. 이 파트에서는 이러한 이종 시스템 간의 간극을 메우는 '브릿지', 즉 통합 서비스의 아키텍처적 필연성과 구체적인 구현 방안을 심층적으로 분석합니다.

### 2.1  통합 서비스의 아키텍처적 필연성

단일 시스템이 아닌 여러 시스템이 결합된 '시스템의 시스템(System of Systems)' 환경에서, 전용 통합 솔루션이 왜 필요한지에 대한 기술적, 사업적 동인을 파악하는 것은 매우 중요합니다.

#### 2.1.1  "시스템의 시스템" 과제: 프로토콜 및 도메인 이질성

현대의 시스템은 본질적으로 이기종 환경으로 구성됩니다. 예를 들어, 자율주행 차량 시스템은 차량 내부 통신을 위해 DDS를 사용하고, 로보틱스 프레임워크와 연동하기 위해 ROS2를, 클라우드 서버와 통신하기 위해 MQTT나 WebSockets를, 공장 자동화 장비와 연결하기 위해 OPC UA를 사용할 수 있습니다.15 이처럼 서로 다른 통신 프로토콜을 사용하는 구성 요소들을 통합하는 것은 중대한 과제입니다.

DDS 자체도 이러한 이질성을 내포하고 있습니다. DDS는 '도메인(Domain)'이라는 개념을 통해 통신 평면을 논리적으로 격리합니다.7 서로 다른 도메인 ID를 가진 참여자들은 원칙적으로 직접 통신할 수 없습니다. 이는 보안이나 자원 관리를 위해 의도된 기능이지만, 서로 다른 부서나 기능 단위로 개발된 DDS 시스템들을 통합해야 할 때는 오히려 장애물이 됩니다.18

또한, DDS는 근거리 통신망(LAN) 환경에서의 고성능, 저지연 통신(소위 '엣지' 컴퓨팅)에 최적화되어 있습니다. 그러나 인터넷과 같은 광역 통신망(WAN)을 넘나드는 통신이나 리소스가 제한된 클라이언트와의 통신에서는 경량 프로토콜인 MQTT 등에 비해 비효율적일 수 있습니다.15 이러한 '엣지 대 클라우드' 간의 프로토콜 불일치 역시 통합 솔루션의 필요성을 증대시킵니다.

#### 2.1.2  아키텍처 패턴: 애드혹 브릿지에서 중앙 집중식 허브까지

이기종 시스템을 통합하는 접근 방식은 크게 두 가지로 나눌 수 있습니다.

- **점대점(Point-to-Point) 브릿지:** 통합이 필요한 프로토콜 쌍마다 전용 브릿지를 개발하는 방식입니다. 예를 들어, DDS와 MQTT 간의 통신이 필요하면 DDS-MQTT 브릿지를 직접 구현합니다. 이 방식은 단일 통합 시나리오에서는 간단할 수 있지만, 시스템의 복잡도가 증가함에 따라 심각한 확장성 문제를 야기합니다. N개의 서로 다른 시스템을 통합하기 위해서는 최악의 경우 N×(N−1)/2개의 브릿지를 개발하고 유지보수해야 하는 'N-제곱 문제(N-squared problem)'에 직면하게 됩니다.
- **중앙 집중식 통합 허브(Centralized Integration Hub):** 이는 eProsima Integration Service와 같은 솔루션이 채택한 방식으로, 중앙의 허브가 '만능 번역기' 역할을 수행하는 모델입니다.16 각 시스템은 자신만의 전용 플러그인(어댑터)을 통해 중앙 허브에 연결됩니다. 일단 허브에 연결되면, 해당 시스템은 허브에 연결된 다른 모든 시스템과 통신할 수 있습니다. 이 방식은 통합의 복잡도를 N개의 플러그인을 관리하는 선형(N) 문제로 축소시켜, 시스템의 확장성과 유지보수성을 획기적으로 개선합니다.

이러한 통합 허브 아키텍처는 잘 알려진 소프트웨어 디자인 패턴인 '중재자 패턴(Mediator Pattern)'과 '어댑터 패턴(Adapter Pattern)'의 실제 구현 사례로 볼 수 있습니다. 중앙 코어는 각 시스템이 서로를 직접 알 필요 없이 통신을 중재하는 **중재자** 역할을 수행합니다. 그리고 각 프로토콜을 지원하는 플러그인(eProsima에서는 '시스템 핸들'이라 부름)은 특정 시스템(예: ROS2)의 인터페이스와 프로토콜을 중재자가 이해할 수 있는 공통 인터페이스(예: xTypes)로 변환하는 **어댑터** 역할을 합니다. 이러한 관점은 솔루션의 구조와 장점을 이해하는 데 강력한 프레임워크를 제공합니다.

#### 2.1.3  공통 데이터 표현의 역할 (OMG의 DDS-XTypes)

통합 허브 모델이 효과적으로 작동하기 위한 전제 조건은 모든 프로토콜이 변환될 수 있는 '공통 중간 언어'의 존재입니다. eProsima Integration Service는 이 공통 언어로 OMG의 DDS-XTypes(Extensible and Dynamic Topic Types) 표준을 채택했습니다.18

DDS-XTypes는 컴파일 시점에 시스템의 모든 데이터 타입을 미리 알 필요 없이, 런타임에 동적으로 타입을 정의하고 해석할 수 있는 기능을 제공합니다.18 예를 들어, IDL(Interface Definition Language) 파일로부터 런타임에 타입 정보를 읽어와 해당 구조에 맞는 데이터를 생성하고 변환할 수 있습니다. 이러한 동적 타입 처리 능력은 시스템이 진화함에 따라 새로운 데이터 타입이 추가되더라도 통합 서비스 자체를 재컴파일할 필요 없이 YAML 설정 파일 수정만으로 대응할 수 있게 해주는 강력한 장점입니다. 이는 시스템의 유연성과 진화 가능성을 극대화하는 핵심 기술입니다.

### 2.2  구현 심층 분석: eProsima Fast DDS Integration Service

eProsima Fast DDS Integration Service는 오픈 소스 기반의 유연하고 설정 중심적인 접근 방식으로 주목받는 솔루션입니다. 이 섹션에서는 eProsima Integration Service의 아키텍처를 분석하고, YAML 파일을 이용한 실제 사용법을 단계별로 살펴봅니다.

#### 2.2.1  핵심 아키텍처: 통합 서비스 코어와 시스템 핸들

eProsima Integration Service의 아키텍처는 매우 명료합니다. 중앙에는 xTypes를 공통 언어로 사용하는 경량의 '통합 서비스 코어'가 자리 잡고 있으며, 특정 프로토콜과의 통신은 동적으로 로드되는 플러그인인 '시스템 핸들(System Handle)'이 담당합니다.16 이 구조는 새로운 프로토콜을 지원해야 할 때, 해당 프로토콜을 위한 시스템 핸들만 추가하면 되므로 확장성이 매우 뛰어납니다.

현재 공식적으로 지원되는 시스템 핸들은 다음과 같습니다 16:

- Fast DDS
- ROS (Robot Operating System)
- ROS 2
- WebSocket
- FIWARE

이 중 Fast DDS 시스템 핸들은 세 가지 주요 사용 사례를 위해 설계되었습니다 18:

1. **이기종 미들웨어 연결:** DDS 애플리케이션과 다른 미들웨어(예: ROS2, WebSocket) 기반 애플리케이션 간의 통신을 중계합니다.
2. **DDS 도메인 ID 연결:** 서로 다른 DDS 도메인 ID 상에서 실행되는 두 DDS 애플리케이션을 연결합니다.
3. **TCP 터널링:** 지리적으로 분리된 두 머신에 각각 Integration Service 인스턴스를 실행하여 WAN을 통한 DDS 통신을 위한 TCP 터널을 생성합니다.

#### 2.2.2  실용적 사용법: YAML을 통한 구성

eProsima Integration Service의 가장 큰 특징은 모든 통합 로직이 하나의 YAML 설정 파일을 통해 선언적으로 정의된다는 점입니다.18 이는 '코드로써의 설정(Configuration-as-Code)' 철학을 반영하며, 통합 구성을 버전 관리하고 자동화된 배포 파이프라인에 포함시키기 용이하게 합니다.

YAML 파일의 주요 구성 요소는 다음과 같습니다 21:

- **`systems` 블록:** 통신에 참여하는 미들웨어 시스템을 정의합니다. 각 시스템에는 고유한 이름을 부여하고, `type` 필드에 미들웨어 종류(예: `fastdds`, `ros2`)를 명시합니다. `fastdds` 타입의 경우, `domain_id`와 같은 DDS 관련 특정 파라미터를 추가로 설정할 수 있습니다.
- **`routes` 블록:** 데이터가 어떤 시스템에서 어떤 시스템으로 흘러갈지를 정의하는 '경로'를 설정합니다. `from`과 `to` 필드에 `systems` 블록에서 정의한 시스템 이름을 지정합니다.
- **`topics` 블록:** 통합의 핵심 로직이 담긴 부분입니다. 여기서 특정 토픽을 다른 이름으로 매핑(토픽 리매핑 또는 에일리어싱)하거나, 서로 다른 데이터 타입을 변환하는 규칙을 정의합니다.

예를 들어, ROS2의 `std_msgs::msg::String` 타입의 데이터를 DDS의 `HelloWorld` 구조체(문자열 필드 `message`와 정수 필드 `index`를 가짐)로 변환하는 시나리오를 생각해 볼 수 있습니다. YAML 설정에서는 ROS2 토픽의 `data` 필드를 DDS 토픽의 `message` 필드로 매핑하도록 명시할 수 있습니다.

#### 2.2.3  튜토리얼: DDS-to-ROS2 브릿지 구축하기

다음은 eProsima의 공식 예제를 기반으로 한 DDS와 ROS2 간의 브릿지를 구축하는 단계별 튜토리얼입니다.26

전제 조건:

먼저 시스템에 Integration Service, Fast DDS, ROS2 및 관련 시스템 핸들이 올바르게 설치되어 있어야 합니다.26

1단계: 독립 애플리케이션 실행

세 개의 터미널을 엽니다.

- **터미널 1:** ROS2 환경을 소싱하고 ROS2 `talker` 예제를 실행합니다. 이 `talker`는 `chatter`라는 토픽에 문자열 메시지를 주기적으로 발행합니다.

  Bash

  ```
  source /opt/ros/<ROS2_DISTRO>/setup.bash
  ros2 run demo_nodes_cpp talker
  ```

- **터미널 2:** Fast DDS `HelloWorld` 예제의 구독자(subscriber)를 실행합니다.

./build/is-examples/dds/DDSHelloWorld/DDSHelloWorld -m subscriber

\```

이 단계에서는 두 애플리케이션이 서로 다른 미들웨어와 토픽 이름, 데이터 타입을 사용하기 때문에 통신이 이루어지지 않는 것을 확인할 수 있습니다.26

2단계: YAML 구성 파일 작성

다음 내용으로 fastdds_ros2_helloworld.yaml 파일을 작성합니다. 이 파일은 Integration Service에게 두 시스템을 어떻게 연결할지 알려줍니다.

YAML

```
systems:
  ros2:
    type: ros2
  dds:
    type: fastdds

routes:
  ros2_to_dds:
    from: ros2
    to: dds

topics:
  HelloWorldTopic:
    type: HelloWorld
    route: ros2_to_dds
    remap:
      ros2:
        topic: chatter
        type: std_msgs::msg::String
      dds:
        topic: HelloWorldTopic
```

이 설정은 `ros2` 시스템과 `dds` 시스템을 정의하고, `ros2_to_dds`라는 이름의 경로를 설정합니다. `topics` 섹션에서는 ROS2의 `chatter` 토픽(타입 `std_msgs::msg::String`)을 DDS의 `HelloWorldTopic` 토픽(타입 `HelloWorld`)으로 매핑하도록 지정합니다. Integration Service는 이 정보를 바탕으로 데이터 타입 변환을 자동으로 처리합니다.

**3단계: 통합 서비스 실행**

- **터미널 3:** ROS2 및 로컬 작업 공간 환경을 소싱한 후, `integration-service` 명령어를 사용하여 방금 작성한 YAML 파일과 함께 서비스를 실행합니다.

  Bash

  ```
  source /opt/ros/<ROS2_DISTRO>/setup.bash
  source install/setup.bash
  integration-service fastdds_ros2_helloworld.yaml
  ```

서비스가 실행되면, 터미널 1의 ROS2 `talker`가 발행하는 메시지가 Integration Service를 통해 변환되어 터미널 2의 Fast DDS 구독자에게 성공적으로 수신되는 것을 확인할 수 있습니다. 반대 방향(Fast DDS에서 ROS2로)의 통신도 유사한 YAML 설정을 통해 쉽게 구현할 수 있습니다.26

#### 2.2.4  고급 사용 사례: WAN 터널링 및 서비스 호출

eProsima Integration Service는 단순한 토픽 브릿징을 넘어 더 복잡한 시나리오도 지원합니다. 예를 들어, 인터넷으로 분리된 두 개의 LAN에 각각 Integration Service 인스턴스를 실행하고 TCP를 사용하도록 설정하면, 두 DDS 도메인 간에 안전한 WAN 터널을 생성할 수 있습니다.18 또한, 발행-구독 모델뿐만 아니라 요청-응답(RPC) 모델인 서비스 호출도 브릿징할 수 있습니다. 이를 통해 ROS2 클라이언트가 Fast DDS 서버가 제공하는 서비스를 호출하는 것과 같은 이기종 서비스 통합이 가능합니다.18

### 2.3  구현 심층 분석: RTI Connext 통합 솔루션

RTI(Real-Time Innovations)는 DDS 시장의 주요 상용 벤더로서, 단일 통합 도구가 아닌 포괄적인 통합 솔루션 에코시스템을 제공하는 전략을 취합니다. 이 섹션에서는 RTI의 접근 방식을 분석하며, 특히 상용 환경에서 요구되는 강력한 도구와 특정 산업 분야에 특화된 솔루션에 초점을 맞춥니다.

#### 2.3.1  RTI 에코시스템: 통합 제품군

RTI의 전략은 단일 바이너리 형태의 통합 서비스를 제공하기보다, 개발, 설정, 모니터링, 배포에 이르는 전체 라이프사이클을 지원하는 통합된 도구와 서비스 제품군을 제공하는 것입니다.28 이는 특정 산업 분야의 고객들이 겪는 통합의 어려움을 줄이고, 강력한 상업적 지원을 통해 안정적인 시스템을 구축할 수 있도록 돕는 '전체 제품(Whole Product)' 전략으로 볼 수 있습니다.

주요 구성 요소는 다음과 같습니다:

- **Integration Designer APEX:** 시스템 구성, 매크로 생성, 사용자 인터페이스(UI) 디자인 등을 위한 강력한 그래픽 기반의 통합 개발 환경(IDE)입니다.28 이는 eProsima의 텍스트 기반 YAML 설정 방식과 대조되는 RTI의 개발 철학을 보여줍니다. GUI 기반의 안내된 워크플로우는 복잡한 프로젝트를 시각적으로 관리하고, DDS에 익숙하지 않은 개발자들의 진입 장벽을 낮출 수 있습니다.
- **Admin Console:** 실행 중인 DDS 시스템의 상태를 실시간으로 모니터링하고 디버깅하기 위한 필수 도구입니다. 참여자, 토픽, QoS 설정 등을 시각적으로 확인하고, 데이터 흐름을 추적하여 연결 문제를 신속하게 진단할 수 있습니다.3
- **방대한 드라이버 라이브러리:** RTI는 다양한 산업 분야의 수많은 서드파티 장비 및 프로토콜(오디오/비디오 시스템, 조명, 보안, HVAC 등)을 위한 사전 구축된 드라이버를 제공합니다.29 이를 통해 개발자는 특정 장치를 DDS 데이터버스에 통합하기 위한 코드를 직접 작성할 필요 없이, 검증된 드라이버를 활용하여 개발 시간을 단축할 수 있습니다.

#### 2.3.2  RTI Web Integration Service: 데이터버스를 웹으로 연결

RTI Web Integration Service는 DDS 데이터버스와 웹 기술 간의 간극을 메우기 위해 특별히 설계된 상용 제품입니다.30 이 서비스는 수정되지 않은 기존 DDS 애플리케이션과 웹 기반 서비스가 투명하게 통신할 수 있는 표준 기반의 브릿지를 제공합니다.

- **아키텍처:** 이 서비스는 DDS 데이터버스에 대한 RESTful HTTP API와 WebSocket API를 노출하는 독립 실행형 애플리케이션으로 작동합니다.30 웹 클라이언트는 이 API들을 통해 DDS 세계와 상호작용합니다.
- **실용적 사용법 (API):**
  - **REST API:** 상태 비저장(stateless) 상호작용에 적합합니다. 웹 클라이언트는 HTTP 요청을 통해 DDS 엔티티(예: `DataWriter`)를 동적으로 생성하거나, 특정 토픽에 단일 데이터 샘플을 JSON 또는 XML 형식으로 발행(write)하고 읽을(read) 수 있습니다.30 이는 상태를 유지할 필요가 없는 간헐적인 데이터 교환에 유용합니다.
  - **WebSocket API:** 상태 저장(stateful) 방식의 실시간 데이터 스트리밍에 사용됩니다. 웹 클라이언트는 WebSocket 연결을 통해 DDS 토픽을 구독하고, 데이터가 발생할 때마다 실시간으로 업데이트를 수신할 수 있습니다. 양방향 통신도 가능하여 웹 애플리케이션에서 DDS 시스템으로 지속적인 데이터 스트림을 보낼 수도 있습니다.30
- **구성:** 서비스의 동작은 XML 설정 파일을 통해 제어됩니다. 이 파일에서 DDS 도메인, 참여할 토픽, 데이터 타입, 그리고 허가된 애플리케이션만 데이터버스에 접근할 수 있도록 하는 보안 및 접근 제어 메커니즘 등을 상세하게 정의할 수 있습니다.34

#### 2.3.3  RTI Connext Drive: 소프트웨어 정의 차량을 위한 전문 프레임워크

RTI의 수직적 통합 전략을 가장 잘 보여주는 사례는 자동차 산업을 겨냥한 RTI Connext Drive입니다.35 이는 단순한 브릿지를 넘어, 소프트웨어 정의 차량(Software-Defined Vehicle, SDV) 개발을 위한 완전한 커넥티비티 프레임워크입니다.

Connext Drive는 AUTOSAR Classic, ROS2와 같은 자동차 산업의 핵심 표준을 위한 사전 구축된 통합 툴킷을 제공합니다.36 이를 통해 개발자들은 ADAS, 차세대 E/E 아키텍처와 같은 일반적인 자동차 사용 사례를 개발할 때, 익숙한 에코시스템 아키텍처 내에서 데이터 중심 커넥티비티의 이점을 누릴 수 있습니다. 이는 맞춤형 프로그래밍의 필요성을 없애고, 전체 시스템의 복잡성과 비용을 최소화하며, 성능 저하 없이 미래 지향적인 시스템을 구축할 수 있도록 지원합니다. 특히, Connext Drive는 TUV SUD로부터 ASIL D 수준의 기능 안전 인증을 받은 구성 요소를 포함하고 있어, 양산 차량에 적용 가능한 신뢰성과 안전성을 제공합니다.36

이러한 접근 방식은 RTI가 특정 고부가가치 수직 시장(Vertical Market)에 대해 단순한 기술 제공자를 넘어, 완전한 엔드투엔드 솔루션 파트너가 되려는 전략을 가지고 있음을 명확히 보여줍니다. 이는 고객의 통합 부담을 근본적으로 줄여주고, 강력한 에코시스템을 구축하여 시장에서의 입지를 공고히 하는 효과적인 방법입니다.

## 3.  전략적 및 실용적 고려사항

지금까지 DDS의 기본 원리와 통합 서비스의 아키텍처 및 구현을 살펴보았습니다. 마지막 파트에서는 기술적 세부 사항을 넘어, 아키텍트가 실제 프로젝트에서 올바른 기술을 선택하고 효과적으로 관리하기 위해 필요한 전략적 의사결정 프레임워크를 제시합니다. 성능, 디버깅, 배포 전략과 같은 실질적인 문제들을 다루며, 다양한 미들웨어 환경 속에서 DDS 통합 서비스의 위상을 명확히 합니다.

### 3.1  미들웨어 환경 내 비교 분석

DDS 통합 서비스가 모든 문제에 대한 만병통치약은 아닙니다. 아키텍트는 프로젝트의 요구사항에 가장 적합한 도구를 선택하기 위해 다른 미들웨어 기술과의 장단점을 명확히 이해해야 합니다.

#### 3.1.1  DDS 통합 서비스 대 엔터프라이즈 서비스 버스(ESB)

- **핵심 차이:** 실시간 데이터 중심(DDS) 대 엔터프라이즈 서비스 오케스트레이션(ESB).
- **분석:** ESB는 전통적으로 기업의 비즈니스 프로세스를 통합하기 위해 설계되었습니다. CRM, ERP와 같은 엔터프라이즈 애플리케이션 간의 요청-응답(Request-Reply) 상호작용, 복잡한 데이터 변환 및 라우팅을 중앙 집중식 허브를 통해 처리하는 서비스 지향 아키텍처(SOA)의 핵심 구성 요소입니다.38 ESB는 서비스 오케스트레이션에 강점을 가지지만, 일반적으로 중앙 집중적이며 실시간 시스템에서 요구하는 저지연, 고처리량의 P2P 데이터 배포에는 최적화되어 있지 않습니다. 반면, DDS 통합 서비스는 빠르고 분산된 DDS 데이터버스를 기반으로 구축되어, 로봇, 차량, 산업 장비와 같은 실시간 구성 요소들을 통합하는 데 훨씬 적합합니다.41
- **결론:** 비즈니스 프로세스 및 워크플로우 오케스트레이션에는 ESB를, 실시간 시스템의 시스템(System-of-Systems) 통합에는 DDS 기반 서비스를 사용하는 것이 바람직합니다.

#### 3.1.2  DDS 통합 서비스 대 Apache Kafka

- **핵심 차이:** 글로벌 데이터 공간(DDS) 대 영구적 이벤트 로그(Kafka).

- **분석:** DDS는 데이터의 *현재 상태*를 분산시키는 데 중점을 둡니다. DDS의 글로벌 데이터 공간은 각 데이터 인스턴스의 최신 값을 담고 있는 분산 캐시와 같습니다.4 이와 대조적으로, Kafka는 *이벤트*의 분산된 불변 로그(immutable log)입니다. Kafka는 대규모 데이터 파이프라인 처리, 이벤트 소싱(Event Sourcing), 그리고 방대한 양의 이력 데이터를 순차적으로 처리하고 분석하는 데 최적화되어 있습니다.44 두 기술 모두 발행-구독 모델을 사용하지만, 그 근본적인 데이터 모델과 목적은 완전히 다릅니다.

- **결론:** 대규모 데이터 수집, 스트림 처리, 이벤트 이력 분석이 필요할 때는 Kafka를 사용해야 합니다. 실시간 상태 동기화, 분산 제어, 그리고 최신 데이터 값에 기반한 상호작용이 중요할 때는 DDS 기반 서비스를 선택해야 합니다.

#### 3.1.3  DDS 통합 서비스 대 직접 MQTT 브릿지

- **핵심 차이:** 풍부한 QoS 및 데이터 중심성(DDS) 대 경량 텔레메트리(MQTT).
- **분석:** MQTT는 저대역폭, 고지연, 또는 불안정한 네트워크 환경을 위해 설계된 경량의 브로커 기반 프로토콜로, IoT 분야에서 널리 사용됩니다.15 MQTT는 DDS가 제공하는 풍부한 QoS 정책, 데이터 중심 모델, P2P 자동 발견 기능 등을 갖추고 있지 않습니다. DDS 통합 서비스는 이 두 세계를 연결하는 게이트웨이 역할을 합니다. 즉, LAN 내부의 풍부하고 고성능인 DDS 환경과 WAN/클라우드 연결을 위한 경량의 MQTT 환경 간의 변환을 담당하여 각 프로토콜의 장점을 모두 활용할 수 있게 해줍니다.15
- **결론:** DDS 기반의 실시간 시스템을 클라우드/IoT 플랫폼과 연결해야 할 경우, 전용 통합 서비스를 사용하여 MQTT로 브릿징하는 것이 가장 효과적인 아키텍처입니다.

이러한 비교 분석을 통해 현대의 복잡한 시스템은 본질적으로 '폴리글랏 미들웨어(Polyglot Middleware)' 환경을 가질 수밖에 없다는 점이 명확해집니다. 즉, 하나의 미들웨어가 모든 요구사항을 충족시킬 수 없으며, 엣지에서는 DDS를, IoT 기기 통신에는 MQTT를, 백엔드 데이터 처리에는 Kafka를 함께 사용하는 것이 일반적입니다. 따라서, 이러한 전문화된 도메인들을 중재하는 '통합 서비스'는 선택적 추가 기능이 아니라, 아키텍처의 필수적인 구성 요소가 됩니다.

| 특성                 | DDS 통합 서비스               | 엔터프라이즈 서비스 버스 (ESB) | Apache Kafka                    | MQTT 브릿지               |
| -------------------- | ----------------------------- | ------------------------------ | ------------------------------- | ------------------------- |
| **주요 패러다임**    | 데이터 중심 상태 공유         | 서비스 중심 오케스트레이션     | 이벤트 중심 로그 스트리밍       | 메시지 중심 텔레메트리    |
| **아키텍처**         | 분산/P2P (내부) + 허브 (외부) | 중앙 집중식 허브/버스          | 분산 클러스터/브로커            | 중앙 집중식 브로커        |
| **일반적 지연 시간** | 마이크로초 ~ 밀리초           | 밀리초 ~ 초                    | 밀리초                          | 밀리초 ~ 초               |
| **처리량**           | 높음                          | 중간                           | 매우 높음                       | 낮음 ~ 중간               |
| **데이터 모델**      | IDL 기반 강력한 타입          | 스키마 기반 (XML/JSON)         | 스키마 기반 (Avro, Protobuf)    | 불투명 페이로드 (Opaque)  |
| **핵심 강점**        | 실시간 제어, 상태 동기화      | 비즈니스 프로세스 통합         | 대규모 이벤트 이력, 스트림 처리 | 저전력/불안정 네트워크    |
| **주요 사용 사례**   | 로보틱스, 자율주행, 국방      | 기업 애플리케이션 통합(EAI)    | 빅데이터 파이프라인, 로그 집계  | IoT 센서 네트워크, 모바일 |

#### 3.1.4  통합 서비스의 아키텍처적 위상

다양한 미들웨어 브릿지와 통합 서비스의 존재는 '브릿지' 자체가 성숙하고 필수적인 아키텍처 패턴임을 증명합니다.17 따라서 아키텍트의 고민은 브릿지가 필요한지 여부가 아니라, 어떤 종류의 브릿지가 가장 적합한지를 결정하는 것입니다. 선택지는 상용 서비스, 오픈 소스 도구, 또는 맞춤형 구현으로 나뉘며, 이는 개발 리소스, 기술 지원 요구사항, 성능 목표 등 다양한 요인에 따라 결정되어야 합니다.

### 3.2  성능, 디버깅 및 배포 전략

아키텍트가 통합 전략을 최종 결정하기 전에 반드시 고려해야 할 실질적인 운영상의 문제들을 다룹니다.

#### 3.2.1  성능 영향: 통합 오버헤드 분석

통합 서비스를 도입하는 것은 명백한 트레이드오프를 수반합니다. 메시지는 소스 애플리케이션에서 통합 서비스를 거쳐 목적지 애플리케이션으로 전달되므로, 추가적인 네트워크 홉(hop)과 처리 계층이 생깁니다. 이는 본질적으로 네이티브 통신에 비해 지연 시간을 증가시키고 CPU 및 메모리 리소스를 더 소모하게 만듭니다.15

이 오버헤드의 크기는 사용 사례, 데이터 크기, 통신 빈도에 따라 크게 달라집니다. 하지만 여러 연구에 따르면, 컨테이너화로 인한 오버헤드는 미미할 수 있으며 49, Fast DDS와 같은 최신 DDS 구현체의 성능이 매우 뛰어나기 때문에 브릿지로 인한 추가 지연이 많은 애플리케이션에서 수용 가능한 범위 내에 있을 수 있습니다.52 핵심은 시스템의 가장 중요한 데이터 경로(critical path)에 대해 실제 환경과 유사한 조건에서 벤치마킹을 수행하여 성능 영향을 정량적으로 평가하는 것입니다.

#### 3.2.2  이기종 시스템 디버깅 전략

이기종 미들웨어가 연결된 시스템에서 메시지 유실이나 지연이 발생했을 때, 문제의 원인을 찾는 것은 매우 복잡합니다. 장애 지점은 소스 애플리케이션, 소스 미들웨어, 통합 서비스, 목적지 미들웨어, 목적지 애플리케이션, 또는 네트워크 인프라 등 통신 경로의 어느 곳에나 있을 수 있습니다.54

따라서 체계적이고 다층적인 디버깅 접근 방식이 필수적입니다.

1. **네트워크 계층:** `ping`, `ipconfig` (Windows) 또는 `ifconfig` (Linux)와 같은 기본 도구를 사용하여 노드 간의 기본적인 IP 연결성을 확인합니다. 멀티캐스트가 필요한 경우, 멀티캐스트 그룹으로의 `ping`을 통해 라우터 및 스위치 설정을 검증합니다.57

2. **미들웨어 계층:** 각 미들웨어 벤더가 제공하는 전문 모니터링 및 디버깅 도구를 적극적으로 활용합니다. RTI Admin Console 3, eProsima Fast DDS Monitor 58, 또는 표준 

   `rtiddsping` 유틸리티 57 등을 사용하여 참여자 발견, 토픽 매칭, QoS 호환성 문제를 진단할 수 있습니다. 이러한 도구들은 통신이 실패하는 근본적인 원인(예: QoS 불일치)을 밝혀내는 데 매우 효과적입니다.

3. **애플리케이션/서비스 계층:** 통합 서비스와 양 끝단의 애플리케이션에서 상세 로깅(verbose logging)을 활성화하여 메시지의 흐름을 추적합니다.54 로그 메시지에 타임스탬프와 고유 트랜잭션 ID를 포함시키면 여러 시스템에 걸친 메시지의 여정을 추적하는 데 도움이 됩니다.

4. **체계적 격리:** '분할 정복(Divide and Conquer)' 기법을 사용하여 장애 지점을 체계적으로 좁혀 나갑니다.59 예를 들어, 먼저 소스 애플리케이션에서 통합 서비스까지의 통신이 정상적인지 확인합니다. 그 다음, 통합 서비스에서 목적지 애플리케이션까지의 통신을 테스트합니다. 이를 통해 문제의 범위를 절반으로 줄여나갈 수 있습니다.

이러한 디버깅의 복잡성은 시스템 설계 단계부터 '디버깅 가능성(Debuggability)'을 고려해야 함을 시사합니다. 장애가 발생한 후에 디버깅 방법을 고민하는 것은 너무 늦습니다. 모든 구성 요소에 걸쳐 상관관계가 있는 포괄적인 로깅을 구현하고, 상태 및 헬스 체크 엔드포인트를 노출하며, 개발 및 배포 프로세스에 모니터링 도구 체인을 통합하는 등, 디버깅은 사후 활동이 아닌 시스템의 내재적 아키텍처 속성이 되어야 합니다.

#### 3.2.3  의사결정 프레임워크: 상용 서비스 대 맞춤형 브릿지

통합 문제를 해결하기 위해 기성 제품(Off-the-Shelf)을 구매할 것인가, 아니면 직접 개발(Build)할 것인가의 결정은 기술적 선택을 넘어선 전략적 판단입니다.

- **상용/오픈소스 서비스 사용 시 (Buy):**
  - **적합한 경우:** ROS, WebSocket, MQTT 등 표준 프로토콜과 통합할 때; 개발 시간이 제한적이고 빠른 시장 출시가 중요할 때; 상용 수준의 기술 지원, 문서, 검증된 안정성이 필요할 때.
  - **장점:** 개발 비용 및 시간 절감, 벤더에 의한 유지보수, 검증된 신뢰성.60
  - **단점:** 라이선스 비용 발생 가능, 특정 기능에 대한 커스터마이징 제약.
- **맞춤형 브릿지 개발 시 (Build):**
  - **적합한 경우:** 시스템 핸들이 존재하지 않는 독점(proprietary) 프로토콜과 통합해야 할 때; 일반적인 서비스의 오버헤드를 감당할 수 없는 극단적인 성능 요구사항이 있을 때; 통합 로직이 매우 특수하여 기존 서비스의 설정으로 표현할 수 없을 때.
  - **장점:** 요구사항에 완벽하게 부합하는 맞춤형 솔루션, 성능 최적화 가능, 완전한 제어권 확보.60
  - **단점:** 높은 초기 개발 비용과 지속적인 유지보수 부담, 양쪽 프로토콜에 대한 깊은 전문 지식 필요, 버그 발생 위험.60

아키텍트는 프로젝트의 예산, 일정, 그리고 통합 구성 요소의 장기적인 전략적 가치를 종합적으로 고려하여 이 'Buy vs. Build' 결정을 내려야 합니다.

### 3.3  결론 및 향후 전망

본 보고서는 DDS의 데이터 중심 패러다임에서 시작하여, 이기종 시스템 통합의 필요성과 이를 해결하기 위한 통합 서비스의 아키텍처, 구현, 그리고 전략적 고려사항까지 심층적으로 분석했습니다.

#### 3.3.1  통합 서비스 접근 방식의 핵심 강점 및 약점 종합

- **강점:**
  - **통합 단순화:** N-제곱 문제를 선형 문제로 변환하여 복잡한 시스템 통합을 극적으로 단순화합니다.
  - **확장성 및 진화성:** 새로운 시스템이나 프로토콜을 추가할 때 기존 시스템에 미치는 영향을 최소화하여 시스템의 유연한 확장과 진화를 지원합니다.
  - **최적의 프로토콜 활용:** 각 도메인에 가장 적합한 프로토콜(예: 엣지에서는 DDS, 클라우드에서는 MQTT)을 사용하고 이들을 원활하게 연결하여 전체 시스템의 효율을 극대화합니다.
- **약점:**
  - **성능 오버헤드:** 추가적인 처리 계층으로 인한 지연 시간 및 리소스 소비가 발생할 수 있습니다.
  - **단일 장애점(SPOF) 가능성:** 통합 서비스 자체가 장애 지점이 될 수 있으므로, 고가용성을 위한 이중화 구성이 필요할 수 있습니다.
  - **디버깅 복잡성:** 통신 경로가 길어지고 복잡해져 장애 발생 시 원인 추적이 더 어려워질 수 있습니다.
  - **비용:** 상용 솔루션의 경우 라이선스 비용이 발생할 수 있습니다.

#### 3.3.2  미래 동향: 자율 시스템에서의 통합 역할

기술이 발전함에 따라 시스템은 더욱 자율적이고 상호 연결될 것입니다. 자율주행차가 스마트 인프라와 통신(V2I)하고 15, 로봇 군집이 협력하며, 산업 현장의 디지털 트윈이 실시간으로 동기화되는 미래에는 이기종 시스템 간의 원활한 데이터 통합이 그 어느 때보다 중요해질 것입니다. 이러한 환경에서 DDS 통합 서비스는 실시간 제어 시스템(DDS), 인공지능/머신러닝 프레임워크, 그리고 클라우드 기반 분석 플랫폼 간의 핵심적인 가교 역할을 수행하며 그 중요성이 더욱 커질 것으로 전망됩니다.

#### 3.3.3  아키텍트 및 개발자를 위한 최종 권고

복잡한 분산 시스템을 성공적으로 설계하고 구축하기 위해 다음의 원칙을 고려할 것을 권고합니다.

1. **데이터 모델에서 시작하십시오.** 애플리케이션 간의 연결이 아닌, 시스템이 공유해야 할 데이터의 구조(IDL)와 서비스 품질(QoS) 계약을 먼저 정의하십시오. 이는 시스템의 근간을 이룹니다.
2. **폴리글랏 미들웨어를 수용하십시오.** 모든 문제에 맞는 단 하나의 해결책은 없습니다. 각 도메인의 요구사항에 가장 적합한 미들웨어를 선택하고, 이들을 연결하는 전략을 세우십시오.
3. **통합 계층을 부가 기능이 아닌 일급 시민으로 다루십시오.** 시스템 설계 초기부터 통합 아키텍처를 핵심적인 고려사항으로 다루어야 합니다.
4. **기성 솔루션을 우선적으로 고려하십시오.** 강력한 성능이나 기능적 이유가 없는 한, 직접 브릿지를 개발하기보다 잘 지원되는 상용 또는 오픈소스 통합 서비스를 활용하여 개발팀이 핵심 비즈니스 로직에 집중할 수 있도록 하십시오.
5. **포괄적인 모니터링 및 디버깅 도구 체인에 투자하십시오.** 복잡성은 피할 수 없습니다. 시스템의 투명성을 높이고 문제 해결 시간을 단축하기 위한 도구와 프로세스에 대한 투자를 아끼지 마십시오.

#### **참고 자료**

1. DDS와 RTPS 개념정리 - Lab Note - 티스토리, accessed July 6, 2025, [https://lab-notes.tistory.com/entry/DDS-DDS%EC%99%80-RTPS-%EA%B0%9C%EB%85%90%EC%A0%95%EB%A6%AC](https://lab-notes.tistory.com/entry/DDS-DDS와-RTPS-개념정리)
2. Data Distribution Service - Wikipedia, accessed July 6, 2025, https://en.wikipedia.org/wiki/Data_Distribution_Service
3. 2. Publish/Subscribe — RTI Connext Getting Started documentation - RTI Community, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/getting_started_guide/cpp11/intro_pubsub_cpp.html
4. [ROS2]OMG Data-Distribution Service: Architectural Overview and OMG Data-Distribution Service: Architectural Update - velog, accessed July 6, 2025, https://velog.io/@protocol5/ROS2OMG-Data-Distribution-Service-Architectural-Overview-and-OMG-Data-Distribution-Service-Architectural-Update
5. How Does DDS Work? - DDS Foundation, accessed July 6, 2025, https://www.dds-foundation.org/how-dds-works/
6. 데이터 분산 서비스 - 위키백과, 우리 모두의 백과사전, accessed July 6, 2025, [https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EB%B6%84%EC%82%B0_%EC%84%9C%EB%B9%84%EC%8A%A4](https://ko.wikipedia.org/wiki/데이터_분산_서비스)
7. Introduction to DDS - OpenDDS 3.32.0, accessed July 6, 2025, https://opendds.readthedocs.io/en/latest-release/devguide/introduction_to_dds.html
8. OMG Data Distribution Service (DDS) - Object Management Group, accessed July 6, 2025, https://www.omg.org/spec/DDS/1.4/PDF
9. Chapter 11 Quality of Service (QoS) - RTI Community, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/users_manual/users_manual/Quality_of_Service__QoS_.htm
10. Quality of Service - OpenDDS 3.28.1, accessed July 6, 2025, https://opendds.readthedocs.io/en/dds-3.28.1/devguide/quality_of_service.html
11. 4. Quality of Service — The Data Distribution Service Tutorial, accessed July 6, 2025, https://download.zettascale.online/www/docs/Vortex/html/ospl/DDSTutorial/qos.html
12. Quality of Service - OpenDDS 3.27.0, accessed July 6, 2025, https://opendds.readthedocs.io/en/dds-3.27/devguide/quality_of_service.html
13. DDS(Data Distribution Service)란 무엇인가? - Perbiz, accessed July 6, 2025, [http://perbiz.co.kr/data/DDS(Data%20Distribution%20Service)%EB%9E%80.pdf](http://perbiz.co.kr/data/DDS(Data Distribution Service)란.pdf)
14. DDS: The FAQs About QoS - YouTube, accessed July 6, 2025, https://www.youtube.com/watch?v=FRX9xRL4MPU
15. DDS and MQTT: Basics, Challenges and Integration Benefits | EMQ - EMQX, accessed July 6, 2025, https://www.emqx.com/en/blog/navigating-dds-basics-limitations-and-integration-with-mqtt
16. Integration Service - eProsima, accessed July 6, 2025, https://www.eprosima.com/middleware/tools/dds-integration-service
17. Ways to Integrate DDS & UPC UA for Future Industrial Systems | RTI, accessed July 6, 2025, https://www.rti.com/blog/two-approaches-to-integrate-dds-and-opc-ua-for-future-industrial-systems
18. eProsima's Integration Service System Handle for Fast DDS. - GitHub, accessed July 6, 2025, https://github.com/eProsima/FastDDS-SH
19. Integration Service: Integrating Communication Protocols | PPT - SlideShare, accessed July 6, 2025, https://www.slideshare.net/slideshow/integration-service-integrating-communication-protocols/248308508
20. Integration Service - allowing integration of any protocol into DDS - ROS Discourse, accessed July 6, 2025, https://discourse.ros.org/t/integration-service-allowing-integration-of-any-protocol-into-dds/16135
21. eProsima/Integration-Service - GitHub, accessed July 6, 2025, https://github.com/eProsima/Integration-Service
22. Integration Service Core — Integration Service 3.1.0 documentation - eProsima, accessed July 6, 2025, https://integration-service.docs.eprosima.com/
23. 2.1.1. Fast DDS System Handle - eProsima Integration Service, accessed July 6, 2025, https://integration-service.docs.eprosima.com/en/latest/user_manual/systemhandle/fastdds_sh.html
24. Integration Service Core — Integration Service 3.1.0 documentation, accessed July 6, 2025, https://integration-service.docs.eprosima.com/en/latest/
25. accessed January 1, 1970, https://integration-service.docs.eprosima.com/en/latest/user_manual/configuration.html
26. 1.1.1. DDS - ROS 2 bridge — Integration Service 3.1.0 documentation, accessed July 6, 2025, https://integration-service.docs.eprosima.com/en/latest/examples/different_protocols/pubsub/dds-ros2.html
27. 1.2.1. DDS Service Server — Integration Service 3.1.0 documentation, accessed July 6, 2025, https://integration-service.docs.eprosima.com/en/latest/examples/different_protocols/services/dds-server.html
28. Integration Designer 11 Programming Platform - RTI Control, accessed July 6, 2025, https://www.rticontrol.com/integration-designer
29. RTI - Snap One, accessed July 6, 2025, https://www.snapav.com/shop/en/snapav/rti
30. RTI IS - Web Integration Service, accessed July 6, 2025, https://www.rti.com/products/is/web-integration-service
31. INTEGRATION DESIGNER® APEX - RTI Control, accessed July 6, 2025, https://www.rticontrol.com/pub/media/wysiwyg/dealer/technote/Integration_Designer_APEX_Workbook-7-17.pdf
32. RTI Connext Drive Demo - YouTube, accessed July 6, 2025, https://www.youtube.com/watch?v=rLLEzKVHO5g
33. Integration Partners - RTI Control, accessed July 6, 2025, https://www.rticontrol.com/partners/
34. Welcome to RTI Web Integration Service!, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/services/web_integration_service/index.html
35. Robotics Systems & Connectivity Software - RTI, accessed July 6, 2025, https://www.rti.com/industries/industrial/robotics
36. RTI Production-grade connectivity framework optimized for software-defined vehicles, accessed July 6, 2025, https://www.microcontrollertips.com/rti-production-grade-connectivity-framework-optimized-for-software-defined-vehicles/
37. 1. Before You Get Started — RTI Connext Drive Getting Started Guide 3.1.0 documentation, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-drive/3.1.0/doc/manuals/connext_drive/getting_started_guide/getting_started/before.html
38. What is ESB? - Enterprise Service Bus Explained - AWS, accessed July 6, 2025, https://aws.amazon.com/what-is/enterprise-service-bus/
39. EAI vs ESB: Which Integration Strategy is Best? - Inclusion Cloud, accessed July 6, 2025, https://inclusioncloud.com/insights/blog/eai-vs-ebs-integration-strateg/
40. Two patterns for distributed systems: Enterprise Service Bus (ESB) and Distributed Publish/Subscribe - ResearchGate, accessed July 6, 2025, https://www.researchgate.net/publication/266287572_Two_patterns_for_distributed_systems_Enterprise_Service_Bus_ESB_and_Distributed_PublishSubscribe
41. What Is Real-Time SOA? - RTI Community, accessed July 6, 2025, http://community.rti.com/sites/default/files/archive/RTI_WP_RealTimeSOA.pdf
42. Middleware Integration of DDS and ESB for Interconnection between Real-Time Embedded and Enterprise Systems | Request PDF - ResearchGate, accessed July 6, 2025, https://www.researchgate.net/publication/221416383_Middleware_Integration_of_DDS_and_ESB_for_Interconnection_between_Real-Time_Embedded_and_Enterprise_Systems
43. The Data Distribution Service: Reducing Cost through Agile Integration - Twin Oaks Computing, Inc, accessed July 6, 2025, https://www.twinoakscomputing.com/wp/DDS_Exec_Brief_v20l-public.pdf
44. Apache Kafka vs. Integration Middleware (MQ, ETL, ESB) | PPT - SlideShare, accessed July 6, 2025, https://www.slideshare.net/slideshow/apache-kafka-vs-integration-middleware-mq-etl-esb/135064421
45. Apache Kafka vs. Integration Middleware (MQ, ETL, ESB) - Friends, Enemies or Frenemies?, accessed July 6, 2025, https://www.youtube.com/watch?v=6yG2myKcMQE
46. Building a Bridge from Qt to DDS, accessed July 6, 2025, https://www.qt.io/blog/2018/06/06/building-bridge-qt-dds
47. mqtt_bridge provides a functionality to bridge between ROS and MQTT in bidirectional - GitHub, accessed July 6, 2025, https://github.com/groove-x/mqtt_bridge
48. zenoh_bridge_dds 0.5.0 documentation - the official ROS docs, accessed July 6, 2025, https://docs.ros.org/en/rolling/p/zenoh_bridge_dds/
49. Evaluating Performance of OMG DDS in Kubernetes Container Deployment (Industry Track) - Distributed Object Computing (DOC) Group, accessed July 6, 2025, http://www.dre.vanderbilt.edu/~gokhale/WWW/papers/Middleware2020.pdf
50. DDS: DPU-optimized Disaggregated Storage - Far Data Lab, accessed July 6, 2025, https://fardatalab.org/vldb24-zhang.pdf
51. DDS Performance Study — DDS-Demonstrators | TechPush - Read the Docs, accessed July 6, 2025, https://dds-demonstrators.readthedocs.io/en/latest/Teams/1.Hurricane/performance_measurements_C.html
52. Fast DDS TSC RMW report 2021 - GitHub Pages, accessed July 6, 2025, https://osrf.github.io/TSC-RMW-Reports/humble/eProsima-response.html
53. DDS implementation performance benchmark - ROS General - Open Robotics Discourse, accessed July 6, 2025, https://discourse.ros.org/t/dds-implementation-performance-benchmark/19343
54. Useful tools to debug DDS issues | Data Distribution Service (DDS) Community RTI Connext Users, accessed July 6, 2025, https://community.rti.com/howto/useful-tools-debug-dds-issues
55. Debugging Tips and Tricks: A Comprehensive Guide | by Shai Almog | Javarevisited, accessed July 6, 2025, https://medium.com/javarevisited/debugging-tips-and-tricks-a-comprehensive-guide-8d84a58ca9f2
56. (PDF) Middleware for heterogeneous subsystems interoperability in intelligent buildings, accessed July 6, 2025, https://www.researchgate.net/publication/236596906_Middleware_for_heterogeneous_subsystems_interoperability_in_intelligent_buildings
57. HOWTO Do Basic Debugging for System-Level DDS - RTI Community, accessed July 6, 2025, https://community.rti.com/howto/do-basic-debugging-system-level-dds
58. eProsima documentation index — all-docs 1.0 documentation, accessed July 6, 2025, https://docs.eprosima.com/
59. 7 Essential Strategies for Debugging Software - DISHER, accessed July 6, 2025, https://www.disher.com/blog/software-debugging-strategies/
60. SaaS Integration Platforms vs. Custom Integrations - SyncMatters, accessed July 6, 2025, https://syncmatters.com/blog/saas-integration-platform-vs.-custom-integrations
61. ROS on DDS - ROS2 Design, accessed July 6, 2025, https://design.ros2.org/articles/ros_on_dds.html
62. Custom Integration vs. Data Integration Platform | Aspirity, accessed July 6, 2025, https://aspirity.com/blog/custom-platform-integration


# RustDDS
[DDS 언어 지원](./index.md)


RustDDS를 온전히 이해하기 위해서는 그 근간을 이루는 두 가지 핵심 기술, 즉 데이터 분산 서비스(DDS) 표준과 러스트(Rust) 프로그래밍 언어에 대한 선행적 고찰이 필수적이다. 이 두 기술의 융합은 RustDDS의 설계 철학, 기술적 장단점, 그리고 전략적 가치를 결정하는 핵심 요소로 작용한다.



데이터 분산 서비스(Data Distribution Service, DDS)는 객체 관리 그룹(Object Management Group, OMG)이 관리하는 기계 대 기계(M2M) 통신을 위한 미들웨어 표준이다.1 이 표준은 항공우주, 국방, 자율주행차, 로보틱스, 의료 기기 등 미션 크리티컬(mission-critical) 시스템에서 요구되는 확장 가능하고, 실시간성이 보장되며, 신뢰성 높은 고성능 데이터 교환을 목표로 설계되었다.2 2001년에 개발이 시작되어 여러 버전을 거치며 발전해 온 DDS는 그 성숙도와 안정성을 입증받은 기술이다.2


DDS의 근본적인 아키텍처 패러다임은 데이터 중심의 발행-구독 모델에 기반한다.

- **발행-구독 패턴(Publish-Subscribe Pattern):** DDS는 정보를 생산하는 '발행자(Publisher)'와 정보를 소비하는 '구독자(Subscriber)'가 분리된 통신 모델을 채택한다. 이들은 '토픽(Topic)'이라는 논리적 채널을 통해 익명으로 통신하며, 시간적/공간적으로 완전히 분리(decoupled)될 수 있다.2 이러한 분리 구조는 복잡한 네트워크 프로그래밍을 단순화하고, 모듈성과 확장성이 뛰어난 애플리케이션 개발을 가능하게 한다.2
- **데이터 중심성(Data-Centricity)과 "글로벌 데이터 공간(Global Data Space)":** DDS는 MQTT와 같은 메시지 중심 미들웨어와 근본적인 차이를 보인다. DDS는 전달되는 데이터의 구조와 내용을 인지하는 '데이터 중심' 아키텍처를 채택하고 있다.9 이를 통해 콘텐츠 기반 필터링과 같은 강력한 기능을 제공할 수 있다. DDS는 '글로벌 데이터 공간'이라는 가상의 분산 데이터 모델을 제시하는데, 이는 각 애플리케이션이 공유 정보 모델의 일부인 로컬 데이터 캐시를 통해 전체 데이터에 접근하는 것처럼 보이게 하는 추상화 개념이다.9 이는 중앙 집중식 브로커가 존재하는 것이 아니라, 근본적으로 P2P(Peer-to-Peer) 방식으로 동작하는 각 노드의 로컬 저장소 집합을 통해 구현되는 '환상(illusion)'에 가깝다.8
- **브로커 없는 아키텍처와 동적 발견(Dynamic Discovery):** DDS의 통신은 일반적으로 UDP 상에서 동작하는 실시간 발행-구독(Real-Time Publish-Subscribe, RTPS) 프로토콜을 기반으로 하며, 중앙 브로커가 없는 P2P 방식으로 이루어진다.13 이 아키텍처의 핵심 동력은 '동적 발견' 메커니즘이다. 참여자들은 IP 주소나 엔드포인트를 수동으로 설정할 필요 없이 런타임에 서로를 자동으로 발견하여 '플러그 앤 플레이(plug-and-play)'와 같은 유연성을 제공한다.8


DDS API를 구성하는 핵심 객체는 다음과 같다.

- `DomainParticipant`: 애플리케이션이 DDS 도메인(격리된 통신 공간)에 참여하기 위한 진입점이다.8
- `Topic`: 이름과 데이터 타입을 가지는 데이터 채널이다. `With_Key`와 `No_Key` 두 종류로 나뉜다. `With_Key` 토픽은 데이터베이스 테이블과 유사하게, 각 키(key)가 고유한 데이터 '인스턴스'(예: 특정 센서나 차량)를 식별하는 구조를 가진다.13
- `Publisher` 및 `Subscriber`: 각각 데이터 송신과 수신을 담당하는 엔티티다.13
- `DataWriter` 및 `DataReader`: 특정 토픽에 바인딩되어 애플리케이션이 데이터 샘플을 쓰고 읽는 데 사용하는 타입 지정 엔드포인트다.10


서비스 품질(QoS)은 DDS의 핵심 기능으로, 통신 행위를 정교하게 제어하는 메커니즘이다.

- **요청-제공 모델(Request-Offer Model):** QoS의 핵심은 `DataReader`가 특정 수준의 서비스를 '요청(request)'하고 `DataWriter`가 서비스 수준을 '제공(offer)'하는 '계약' 모델에 있다. 통신은 오직 제공되는 서비스가 요청을 만족시킬 수 있을 때만 성립된다.16 호환되지 않는 QoS 설정은 분산 시스템에서 통신 실패의 주된 원인이 되므로, 디버깅 시 매우 중요한 고려 사항이다.16
- **주요 QoS 정책 설명:**
  - `RELIABILITY`: `BEST_EFFORT`(빠르지만 샘플 유실 가능)와 `RELIABLE`(전송 보장, 더 많은 리소스 필요) 중 선택한다. `RELIABLE` 구독자는 `BEST_EFFORT` 발행자와 연결될 수 없다.20
  - `DURABILITY`: 늦게 참여하는 구독자를 위해 데이터 지속성을 제어한다. `VOLATILE`(지속성 없음), `TRANSIENT_LOCAL`(발행자가 데이터 캐시), `TRANSIENT/PERSISTENT`(서비스가 데이터 캐시) 등의 수준이 있다.20
  - `HISTORY`: 각 인스턴스에 대해 얼마나 많은 데이터 샘플을 저장할지 결정한다. `KEEP_LAST(depth)` 또는 `KEEP_ALL`로 설정할 수 있다.20
  - `LIVELINESS`: 참여자가 여전히 활성 상태인지 감지하는 메커니즘으로, 임대 기간(lease duration)을 설정할 수 있다.20
  - `DEADLINE`: 샘플 간 최대 도착 시간을 명시하는 계약으로, 마감 시간 초과 시 알림을 받을 수 있다.20
  - `OWNERSHIP`: 동일한 토픽 인스턴스에 여러 발행자가 있을 경우를 관리하며, 장애 복구(failover) 구성을 가능하게 한다.16


DDSI-RTPS(Real-Time Publish-Subscribe Protocol) 와이어 프로토콜은 RTI, eProsima, OpenDDS 등 서로 다른 벤더의 DDS 구현체들이 원활하게 통신할 수 있도록 보장하는 핵심 요소다.6 "Shapes" 데모는 이러한 상호운용성을 시연하고 테스트하기 위한 사실상의 표준으로 사용된다.1

이러한 특성들을 종합해 볼 때, DDS는 단순한 메시지 전달 시스템을 넘어선다. '글로벌 데이터 공간', 키를 가진 토픽, 인스턴스 개념, 그리고 `DURABILITY`와 `HISTORY` 같은 풍부한 QoS 정책의 조합은 DDS를 '움직이는 데이터베이스(database in motion)'로 이해하게 만든다. `read`, `take`와 같은 API는 전통적인 메시지 큐의 `send/receive`보다 데이터베이스의 CRUD 작업에 더 가깝다. 이는 DDS가 단순 통신 프로토콜이 아니라 실시간 시스템의 상태를 관리하고 동기화하는 프레임워크임을 시사하며, MQTT나 ZeroMQ와 같은 다른 미들웨어와의 근본적인 차이점을 형성한다.11



러스트는 성능, 안정성, 생산성이라는 세 가지 핵심 가치를 제공하는 현대적인 시스템 프로그래밍 언어다.26 C/C++의 성능 수준을 유지하면서, 이들 언어에서 흔히 발생하는 특정 유형의 버그들을 원천적으로 제거하는 것을 목표로 설계되었다.27


러스트가 C/C++와 동등한 수준의 성능을 낼 수 있는 이유는 다음과 같다.

- **제로 코스트 추상화(Zero-Cost Abstractions):** 고수준의 언어 기능을 사용하더라도 런타임 오버헤드가 발생하지 않는다.28
- **가비지 컬렉터(GC) 부재:** 러스트는 컴파일 시점에 메모리를 결정론적으로 관리한다. 이는 GC가 유발할 수 있는 예측 불가능한 지연 시간(latency spike)을 방지하므로 실시간 시스템에 매우 중요한 특성이다.26
- **저수준 제어(Low-Level Control):** 임베디드 및 시스템 프로그래밍에 필수적인 메모리 레이아웃과 하드웨어 리소스에 대한 정밀한 제어를 제공한다.28


러스트 혁신의 핵심은 메모리 안전성을 보장하는 독특한 모델에 있다.

- **소유권 모델(Ownership Model):** 모든 값은 단 하나의 유일한 '소유자(owner)'를 가진다. 소유자가 스코프를 벗어나면 값은 자동으로 해제(drop)되어 결정론적 자원 관리를 보장한다(RAII).31
- **빌림과 참조(Borrowing and References):** 참조(`&` 및 `&mut`)를 통해 데이터를 '빌리는' 개념이다. 컴파일러는 '하나의 가변 참조' 또는 '여러 개의 불변 참조' 중 하나만 허용한다는 핵심 규칙을 강제한다. 이 규칙이 러스트 안전 보장의 근간을 이룬다.32
- **컴파일 타임 보증:** '빌림 검사기(borrow checker)'는 컴파일 시점에 이 규칙들을 분석하고 강제하여, 안전한(safe) 러스트 코드에서 메모리 안전성(댕글링 포인터, 이중 해제 등 방지)과 스레드 안전성(데이터 경쟁 방지)을 기본적으로 보장한다.26 이는 버그 발견 시점을 런타임에서 컴파일 타임으로 옮겨 생산성과 안정성을 크게 향상시킨다.


소유권과 빌림 규칙은 자연스럽게 멀티스레드 프로그래밍으로 확장된다. 공유된 가변 상태를 원천적으로 방지함으로써, 컴파일러는 동시성 시스템에서 가장 다루기 어려운 버그 중 하나인 데이터 경쟁(data race)을 제거한다. 이를 통해 개발자들은 훨씬 더 큰 확신을 가지고 병렬 코드를 작성할 수 있다.28


러스트의 현대적인 툴체인은 생산성을 높이는 핵심 요소다.

- `cargo`: 통합 패키지 관리자이자 빌드 도구로, 의존성 관리와 프로젝트 빌드를 단순화하여 분편화된 C++ 생태계에 비해 큰 이점을 제공한다.27
- `crates.io`: 러스트 커뮤니티의 중앙 패키지 저장소 역할을 한다.27

러스트의 엄격한 컴파일러, 특히 빌림 검사기는 단순한 안전 기능을 넘어선다. 이는 개발자가 데이터 흐름, 소유권, 상태 관리에 대해 더 엄격하게 생각하도록 강제하는 설계 도구 역할을 한다. C++와 같은 언어에서는 포인터와 참조를 자유롭게 전달할 수 있지만, 바로 그 자유가 댕글링 포인터나 반복자 무효화와 같은 수많은 런타임 오류의 근원이 된다.37 러스트는 이러한 개발자의 규율을 컴파일러에 내재화했다.26 소유권이 모호하거나 별칭 규칙(aliasing rules)이 위반된 코드는 컴파일을 거부한다. 따라서 DDS와 같이 복잡한 프로토콜을 러스트로 구현하는 것은 단순히 API를 번역하는 행위가 아니다. 언어 자체가 제공하는 규율은 견고하고 동시성 높은 실시간 시스템을 구축하는 데 매우 유익하며, 그 결과물은 메모리 안전성을 넘어 아키텍처적으로도 더 견고해지는 경향이 있다.


이 파트에서는 일반적인 기술 논의에서 벗어나 RustDDS 구현체 자체를 심층적으로 분석한다. 프로젝트의 핵심 설계 철학, API 구조, 그리고 DDS 표준을 러스트다운 방식으로 어떻게 변환했는지에 초점을 맞춘다.



RustDDS는 핀란드의 로보틱스 및 ROS2 전문 기업인 Atostek Oy가 개발한 순수 러스트 기반의 DDS 구현체다.1 이 프로젝트는 Apache 2.0 라이선스로 공개되어 있다.38


RustDDS 설계의 가장 중요한 특징은 API 철학에 있다.

- **리터럴 번역이 아닌 기능적 동등성:** 개발팀은 DDS 명세의 API를 1:1로 직접 매핑하는 대신, "러스트의 개념과 관례를 사용한 기능적으로 동등한 근사치"를 만드는 것을 명시적으로 선택했다.1
- **표준과의 차이점:** 주요 차이점은 다음과 같다.
  - **클래스 계층 구조 부재:** 러스트는 상속을 사용하지 않으므로 DDS의 객체 계층 구조를 따르지 않는다.40
  - **러스트 명명 규칙:** API는 DDS 표준 대신 러스트의 `snake_case`와 같은 명명 규칙을 따른다.40
  - **RAII 기반 메모리 관리:** DDS 표준의 `create_`/`delete_` 메서드 대신 러스트의 소유권 및 RAII 모델을 활용한다.40
  - **`Result`/`Option`을 통한 오류 처리:** DDS의 정수 반환 코드 대신 표준 러스트 오류 처리 방식인 `Result<T, E>`와 `Option<T>`를 사용한다.1


- **코드 생성 대신 제네릭:** RustDDS는 러스트의 제네릭 프로그래밍을 활용하여 `DataReader<D, SA>`와 `DataWriter<D, SA>`를 구현했다. 여기서 `D`는 데이터 타입, `SA`는 직렬화 어댑터 타입을 나타낸다. 이는 C++ DDS 구현체에서 흔히 사용되는 코드 생성 단계를 피하게 해준다.40

- **`mio`를 이용한 비동기 I/O:** 표준 DDS의 `Listeners`와 `WaitSets`를 구현하는 대신, 비동기 이벤트 기반 I/O를 위해 `mio`(Metal I/O) 라이브러리를 기반으로 구축되었다. `DataReader`는 `mio`의 `Evented`(v0.6) 또는 `Source`(v0.8) 트레이트를 구현하여 폴링 루프에 등록될 수 있다.13

- **`Serde`를 이용한 직렬화:** 데이터 직렬화 및 역직렬화는 러스트 생태계에서 널리 사용되는 `Serde` 프레임워크와의 통합을 통해 처리된다. 이는 `serde::Serialize`와 `serde::Deserialize`를 구현하는 모든 데이터 타입을 RustDDS를 통해 전송할 수 있음을 의미한다.13 DDS 표준에 명시된 기본 와이어 포맷인 CDR은 

  `CDRSerializerAdapter`와 `CDRDeserializerAdapter`를 통해 구현된다.13


프로젝트 문서에 따르면, 동적 발견, 신뢰성 QoS, 이력 QoS, UDP 기반 RTPS, 그리고 `with_key` 및 `no_key` 토픽과 같은 핵심 기능은 안정적으로 구현되어 있다. 더 진보된 QoS 정책이나 공유 메모리 전송과 같은 기능의 구현 상태도 확인된다.1

RustDDS의 설계 결정들, 즉 `mio`, `Serde`, 제네릭, `Result`/`Option`의 사용은 DDS 전용 메커니즘을 독립적으로 재창조하기보다 기존의 러스트 생태계와 깊이 통합하려는 전략적 선택을 보여준다. 이는 러스트 프로그래머에게 익숙한 개발 경험과 생태계 호환성이 주는 이점이 DDS 표준 API와의 불일치로 인한 비용보다 크다는 판단에 기반한다. 러스트 개발자는 `Serde`를 통해 데이터 타입 호환성을 쉽게 확보하고 13, 

`mio`를 통해 `tokio`와 같은 비동기 런타임의 이벤트 루프 모델을 자연스럽게 이해할 수 있다.13 이는 러스트에 이미 투자한 팀에게 RustDDS를 매력적인 선택지로 만들지만, 다른 언어로 DDS를 경험한 개발자에게는 기존 지식을 1:1로 적용할 수 없어 새로운 학습 곡선을 요구하는 계산된 트레이드오프다.



간단한 발행자와 구독자를 설정하는 과정은 다음과 같다.

1. `DomainParticipant`를 생성한다.13
2. `Topic`을 생성한다.13
3. `Publisher` 및/또는 `Subscriber`를 생성한다.13
4. `Publisher`와 `Topic`으로부터 `DataWriter`를 생성한다.13
5. `Subscriber`와 `Topic`으로부터 `DataReader`를 생성한다.13

다음은 이 흐름을 보여주는 간단한 코드 예시다.13

Rust

```
use rustdds::*;
use rustdds::no_key::{DataReader, DataWriter};
use serde::{Serialize, Deserialize};

#
struct SomeType { a: i32 }

let domain_participant = DomainParticipant::new(0).unwrap();
let qos = QosPolicyBuilder::new()
   .reliability(policy::Reliability::Reliable { max_blocking_time: Duration::from_millis(100) })
   .build();

let publisher = domain_participant.create_publisher(&qos).unwrap();
let subscriber = domain_participant.create_subscriber(&qos).unwrap();

let some_topic = domain_participant.create_topic(
    "SomeTopic".to_string(),
    "SomeType".to_string(),
    &qos,
    TopicKind::NoKey,
).unwrap();

let writer = publisher.create_datawriter_no_key::<SomeType, CDRSerializerAdapter<SomeType>>(
    &some_topic,
    None,
).unwrap();

let mut reader = subscriber.create_datareader_no_key::<SomeType, CDRDeserializerAdapter<SomeType>>(
    &some_topic,
    None,
).unwrap();
```


- **`No_Key` 토픽:** 단순한 단일 인스턴스 데이터 스트림으로, `no_key` 모듈의 타입을 사용한다.13

- **`With_Key` 토픽:** 여러 인스턴스를 다루는 데이터 스트림으로, 맵(map)과 유사하다. 이 경우 데이터 타입은 키를 추출하기 위한 `Keyed` 트레이트를 반드시 구현해야 한다. 이는 단일 토픽 내에서 여러 객체의 상태를 관리하는 데 필수적인 개념이다.13

  `with_key`와 `no_key` 버전의 리더/라이터가 타입 시그니처에서 차이를 보이는 것은 바로 이 키 접근의 필요성 때문이다.13


러스트 구조체를 RustDDS에서 사용 가능하게 만드는 과정은 다음과 같다.

1. 사용자 데이터 구조체에 `serde::Serialize`와 `serde::Deserialize`를 파생(derive)시킨다.
2. `WithKey` 토픽을 사용할 경우, 해당 구조체에 `rustdds::Keyed` 트레이트를 구현한다.13
3. `DataWriter`/`DataReader` 생성 시, 제네릭 타입 파라미터로 `CDRSerializerAdapter<MyType>` 또는 `CDRDeserializerAdapter<MyType>`를 지정한다.13


I/O 이벤트를 처리하는 방법은 다음과 같이 나뉜다.

- **`mio`를 이용한 폴링:** 저수준의 기본 방식이다. `DataReader`를 `mio::Poll` 인스턴스에 등록하고 준비 상태 이벤트를 기다리는 방식으로 동작한다.13 최대의 제어권을 제공하지만 수동으로 이벤트 루프를 관리해야 한다.

- **`async/await` 스트림:** 고수준의 인체공학적 방식이다. `.async_sample_stream()`과 같은 메서드를 사용하여 `DataReader`를 `futures::stream::Stream`으로 변환할 수 있다. 이를 통해 `tokio`나 `smol`과 같은 비동기 런타임 내에서 표준 `async/await` 문법을 사용할 수 있다.13 전체 

  `DataSample`을 반환하는 스트림과 순수 데이터만 반환하는 스트림이 별도로 제공된다.1


`read()`(데이터를 빌려오며 리더 캐시에 남겨둠)와 `take()`(데이터를 이동시키며 캐시에서 제거함)의 의미적 차이는 DDS의 핵심 개념이다.13 또한 `ReadCondition::not_read()`와 같은 다양한 읽기/가져오기 조건을 사용할 수 있다.42

RustDDS의 API는 강력하지만, DDS 개념과 저수준 비동기 러스트(`mio`)에 익숙하지 않은 사용자에게는 학습 곡선이 가파를 수 있다. 실제로 한 사용자는 간단한 단일 프로세스 예제를 두 개의 프로세스로 분리하는 데 어려움을 겪었고, `mio` 기반의 복잡한 예제에 혼란을 느꼈다고 보고했다.41 개발자의 답변에 따르면, 고수준 비동기 프레임워크를 사용하지 않는 한 프로세스 간 통신에는 `mio` 기반의 폴링 루프가 필수적이다.41 이는 'Hello, World' 수준의 예제에서 실제 분산 애플리케이션으로 넘어가는 데 상당한 개념적 도약이 필요함을 시사한다. 따라서 신규 사용자는 가능하면 `mio`의 복잡성을 추상화하는 `async/await` 스트림 기반 API로 시작하는 것이 권장된다.


이 파트에서는 RustDDS를 더 넓은 기술적, 경쟁적 환경에 위치시킨다. 주요 경쟁자와 비교하고, 핵심적인 ROS2 생태계에서의 역할을 평가하며, 성능 프로파일을 분석한다.


순수 러스트 DDS 구현체 시장에는 RustDDS와 Dust-DDS라는 두 주요 경쟁자가 존재한다. 두 프로젝트는 동일한 목표를 추구하지만, 근본적인 설계 철학과 기술적 선택에서 뚜렷한 차이를 보인다. 이는 잠재적 사용자가 기술을 선택할 때 신중하게 고려해야 할 중요한 트레이드오프를 제시한다.


| 기능                   | **RustDDS (Atostek 개발)**                                   | **Dust-DDS (S2E Software Systems 개발)**                     |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **개발 철학**          | **러스트다운(Rust-Idiomatic):** "러스트 개념과 관례를 사용한 기능적 동등성"을 추구.1 러스트 개발자 경험을 우선시한다. | **표준 준수(Standard-Conforming):** OMG DDS 표준 API를 최대한 가깝게 따르는 것을 목표로 한다.43 |
| **핵심 비동기 모델**   | **`mio` 기반 폴링:** `mio` 라이브러리 기반의 비동기 I/O. 고수준 `async` 스트림 래퍼를 제공한다.13 | **액터 모델과 `tokio`:** `tokio` 런타임 기반의 내부 액터 모델을 사용한다.14 |
| **`unsafe` 코드 정책** | 성능이나 FFI를 위해 필요한 경우 `unsafe` 코드를 사용한다 (명시적인 `no_unsafe` 선언 없음). | **`#![forbid(unsafe_code)]`:** 순수하고 안전한 러스트 구현을 명시적으로 보장한다.45 |
| **이벤트 처리 API**    | DDS의 Listener/WaitSet을 `mio` 폴링 또는 `async` 스트림으로 대체한다.40 | 표준 DDS Listener/WaitSet API(트레이트)와 완전한 `async` API를 모두 제공한다.14 |
| **데이터 타입 정의**   | `DataReader`/`DataWriter`에 제네릭을 사용하고, 키가 있는 타입에는 `Keyed` 트레이트를 요구한다.13 | 데이터 타입 자체에 추가 트레이트를 사용해 키와 표현을 정의하며, `DdsType` 파생 매크로를 제공한다.43 |


'러스트다운' 접근 방식과 '표준 준수' 접근 방식은 서로 다른 개발팀에 각기 다른 장단점을 제공한다. RustDDS의 접근 방식은 기존 러스트 개발자들이 `Serde`, `mio`, `Result` 등 익숙한 도구와 패턴을 그대로 활용할 수 있게 하여 학습 곡선을 낮추고 생태계와의 통합을 용이하게 한다. 반면, Dust-DDS의 접근 방식은 C++나 Java 등 다른 언어로 DDS를 사용해 본 개발자들이 기존의 지식과 개념(Listener, WaitSet 등)을 더 직접적으로 적용할 수 있게 해준다.


RustDDS의 `mio` 기반 구조는 특정 비동기 런타임에 종속되지 않는 유연성을 제공하지만, 개발자가 직접 이벤트 루프를 관리해야 하는 부담이 있다. Dust-DDS의 `tokio` 기반 액터 모델은 `tokio` 생태계와 긴밀하게 통합되어 강력한 동시성 처리를 제공하지만, `tokio` 런타임 사용이 전제된다. 액터 모델은 잠금(locking)과 관련된 교착 상태(deadlock) 위험을 줄이는 데 도움이 될 수 있다.14


 Dust-DDS의 `no_unsafe` 보증은 매우 중요한 차별점이다.45 이는 코드베이스 전체에서 `unsafe` 블록의 사용을 금지함으로써 컴파일러가 보장하는 메모리 안전성을 100% 신뢰할 수 있음을 의미한다. 이는 감사나 공식적인 검증이 요구되는 고도의 안전 필수(safety-critical) 또는 보안 시스템에서 결정적인 이점이 될 수 있다.


데이터 타입을 정의하고 이벤트를 처리하는 개발자 경험은 두 라이브러리 간에 차이가 있다. RustDDS는 제네릭과 트레이트를 조합하는 방식을, Dust-DDS는 파생 매크로와 타입 자체에 부가된 트레이트를 사용하는 방식을 채택하여 43, 각기 다른 수준의 추상화와 유연성을 제공한다.



RustDDS는 DDSI-RTPS 와이어 프로토콜을 준수함으로써 주요 C/C++ DDS 벤더인 RTI Connext, eProsima FastRTPS, OpenDDS 등과 성공적으로 상호 운용됨이 입증되었다.1 이는 RustDDS가 기존의 이기종 DDS 시스템에 통합될 수 있음을 의미한다.


RustDDS의 전략적 위치는 네이티브 러스트 ROS2 생태계에서 두드러진다.

- **ROS2의 DDS 의존성:** 차세대 로봇 운영체제인 ROS2는 기본 미들웨어 계층으로 DDS를 사용한다.5
- **`ros2-client`:** Atostek이 개발한 `ros2-client` 라이브러리는 `rclcpp`(C++)나 `rclpy`(Python)와 유사한 네이티브 러스트 API를 제공한다.38
- **RustDDS의 엔진 역할:** 결정적으로, `ros2-client`는 내부 통신 엔진으로 RustDDS를 사용한다.39 이는 ROS2 위에서 순수 러스트로 완전한 로보틱스 애플리케이션을 구축하고자 하는 모든 이들에게 RustDDS가 필수적인 기반 구성 요소임을 의미한다.
- **이점:** 이러한 순수 러스트 스택은 이식성, 더 작은 실행 파일 크기, 그리고 C++ FFI(Foreign Function Interface)의 복잡성을 피할 수 있다는 장점을 제공하며, 이는 실제 사용자 경험에서도 확인된다.49


`ros2_rust`나 `rclrust`와 같은 대안적인 러스트 클라이언트 라이브러리도 존재하지만, 이들은 일반적으로 C/C++ 라이브러리(`rcl`, `rmw`)를 래핑(wrapping)하는 방식이므로 `ros2-client`와 같은 '순수 러스트' 솔루션은 아니다.39

RustDDS의 가장 중요한 전략적 가치는 범용 DDS 구현체로서가 아니라, 네이티브 러스트-온-ROS2 생태계를 가능하게 하는 특정 기술이라는 점에서 찾을 수 있다. 로보틱스 커뮤니티는 ROS2를 통해 DDS를 표준으로 채택했고 5, 러스트의 안전성과 성능에 대한 관심이 이 분야에서 증가하고 있다.38 C/C++ 스택에 의존하지 않는 진정한 네이티브 러스트 애플리케이션을 구축하려면 순수 러스트 클라이언트 라이브러리가 필요하며, `ros2-client`가 그 역할을 한다.39 그리고 `ros2-client`는 RustDDS를 기반으로 한다.39 따라서 RustDDS는 이 틈새 시장이지만 성장하는 분야에서 핵심적인 역할을 하며, 그 발전은 로보틱스에서 러스트의 성공과 궤를 같이할 것이다.



연구 자료에 RustDDS에 대한 직접적인 벤치마크는 없지만, 그 성능 프로파일은 여러 근거를 통해 긍정적으로 예측할 수 있다.

- **러스트 언어의 이점:** GC 부재, 제로 코스트 추상화, 컴파일 타임 안전성 검사 등 러스트 자체의 특성이 고성능을 뒷받침한다.26
- **DDS 아키텍처의 이점:** DDS 프로토콜 자체도 지연 시간과 처리량 측면에서 많은 대안보다 우수한 성능을 보이는 것으로 알려져 있다.8
- **RustDDS의 최적화:** "제로 카피 수신 경로(Zero-copy receive path)" 구현과 "제로 카피 송신 경로(Zero-copy transmit path)" 계획은 성능 지향적 설계를 명확히 보여주는 지표다.1


사용자가 직접 RustDDS의 성능을 측정하고자 할 때 고려해야 할 모범 사례는 다음과 같다.

- **워크로드 정의:** 마이크로벤치마크뿐만 아니라 실제 사용 사례를 대표하는 워크로드를 사용하는 것이 중요하다.50 DDS의 경우, 이는 대표적인 데이터 크기, 발행 주기, 발행자/구독자 수를 의미한다.

- **핵심 메트릭:** DDS 시스템의 핵심 성능 지표는 다음과 같다.

  - **지연 시간(Latency):** `write()` 호출부터 `DataReader`에서 데이터를 사용할 수 있을 때까지의 종단 간 시간.
  - **처리량(Throughput):** 다양한 페이로드 크기에 대해 초당 메시지 수(messages/sec) 및 초당 비트 수(bits/sec)로 측정.12
  - **리소스 사용량:** 연결/참여자당 CPU 및 메모리 소비량.51

- **도구:** `hyperfine`과 같은 범용 러스트 벤치마킹 도구나 `bencher`를 이용한 지속적인 벤치마킹이 유용할 수 있다.50

  `rustls-bench`의 사례는 프로토콜을 위한 맞춤형 테스트 하네스를 구축하는 좋은 본보기를 보여준다.51


DDS, Zenoh, Kafka, MQTT 간의 성능 비교 벤치마크에 따르면 12, 일부 시나리오에서는 Zenoh가 더 빠를 수 있지만, DDS는 Kafka나 MQTT와 같은 브로커 기반 시스템보다 P2P, 저지연 통신에서 월등히 높은 성능을 보인다. 이는 잘 구현된 러스트 버전의 DDS가 매우 높은 성능을 제공할 것이라는 주장을 뒷받침한다.


보고서의 마지막 파트에서는 분석 내용을 종합하여 RustDDS 프로젝트에 대한 전체적인 평가를 제공하고, 잠재적 도입자를 위한 실행 가능한 조언을 제시한다.



RustDDS 프로젝트의 건전성은 여러 지표를 통해 긍정적으로 평가된다.

- **커밋 빈도:** GitHub 활동 페이지를 분석하면 최근까지도 지속적으로 커밋이 이루어지고 있어 프로젝트가 활발히 유지보수되고 있음을 알 수 있다.53 예를 들어, "6일 전"에도 커밋이 푸시되었다.
- **의존성 관리:** `dependabot`을 사용하여 `const-oid`와 같은 의존성을 최신 상태로 유지하는 것은 좋은 프로젝트 위생 상태를 보여준다.53
- **커뮤니티 참여:** 공개 GitHub 저장소에는 이슈, 풀 리퀘스트, 토론이 존재하며, 이는 비록 압도적으로 많지는 않지만 기능하는 오픈소스 프로젝트임을 나타낸다.41
- **버전 이력:** `crates.io`에는 명확한 버전 이력이 있으며, 정기적인 릴리스는 지속적인 개발과 버그 수정을 시사한다.20


공개된 이슈와 토론을 통해 현재 커뮤니티가 다루고 있는 도전 과제나 한계를 파악할 수 있다.41 앞서 언급된 학습 곡선과 문서화의 간극이 대표적인 예다.


RustDDS에 대한 공식적인 `ROADMAP.md`는 없지만, 미래 방향은 여러 단서를 통해 추론할 수 있다.

- `iceoryx2` 로드맵은 DDS 게이트웨이의 잠재적 백엔드로 `rustdds` 또는 `dustdds`를 언급하고 있어, 다른 고성능 통신 프레임워크와의 통합 가능성을 시사한다.56
- 0.8.6 버전에서 언급된 보안 기능에 대한 지속적인 작업은 프로덕션 환경에서의 사용을 위해 구현을 성숙시키려는 노력을 보여준다.40
- `ros2-client`와의 긴밀한 결합은 39 RustDDS의 로드맵이 ROS2의 릴리스 주기와 기능 세트(예: "Iron"과 같은 새로운 ROS 배포판 지원 39)에 큰 영향을 받을 것임을 암시한다.



RustDDS는 실용적이고 러스트다운 API를 갖춘, 성숙하고 활발하게 개발되는 순수 러스트 DDS 구현체다. 그 핵심 전략적 가치는 네이티브 러스트-온-ROS2 생태계를 가능하게 하는 기술이라는 데 있다. 경쟁 구도에는 표준 준수와 절대적인 메모리 안전성을 우선시하는 Dust-DDS가 있으며, 이는 다른 유효한 설계적 트레이드오프를 보여준다.


기술 리더를 위한 실행 가능한 지침은 다음과 같다.

- **RustDDS를 선택해야 하는 경우:**
  - 주요 애플리케이션이 **ROS2 생태계** 내에 있으며, 순수 러스트 스택 구축이 목표일 때.
  - 개발팀이 `mio`나 `tokio`/`futures`를 포함한 **관용적인 러스트 프로그래밍에 이미 능숙**할 때.
  - DDS 표준 API 구조를 엄격히 따르는 것보다 **인체공학적이고 관용적인 러스트 API**를 더 중요하게 생각할 때.
  - 주요 C++ DDS 벤더와의 **상호운용성**이 요구될 때.
- **Dust-DDS를 고려해야 하는 경우:**
  - 보안 또는 인증상의 이유로 엄격한 **`#![forbid(unsafe_code)]` 정책**이 필수 요구사항일 때.
  - 개발팀이 다른 언어에서 경험한 **표준 DDS API 개념**(Listener, WaitSet)에 더 익숙하고, 직접적인 변환을 선호할 때.
  - 프로젝트의 비동기 모델이 Dust-DDS의 내부 아키텍처와 일치하는 **`tokio` 액터 모델**에 크게 의존할 때.
- **러스트 기반 DDS vs. C++ DDS:**
  - **보장된 메모리 및 스레드 안전성**을 활용하고, 컴파일 시점에 버그를 제거하며, 러스트의 현대적인 툴체인(`cargo`)의 이점을 누리고자 할 때 러스트 기반 솔루션(RustDDS 또는 Dust-DDS)을 선택하는 것이 이상적이다. 이는 신규 프로젝트나 기존 시스템에 더 안전한 구성 요소를 도입할 때 특히 유용하다.
  - 조직에 방대한 기존 C++ 코드베이스와 깊은 내부 전문성이 있고, 새로운 언어와 툴체인 도입 비용이 과도할 경우 성숙한 C++ DDS 구현체(RTI Connext, Fast DDS 등)를 고수할 수 있다. 그러나 RTI와 같은 주요 벤더조차 현재 러스트를 공식 지원할 계획이 없다는 점은 57, 러스트 중심 프로젝트에서 C API를 FFI로 호출하는 것이 네이티브 구현보다 덜 이상적인 경로임을 시사한다.


1. Atostek/RustDDS: Rust implementation of Data Distribution ... - GitHub, accessed July 8, 2025, https://github.com/Atostek/RustDDS
2. Data Distribution Service - Wikipedia, accessed July 8, 2025, https://en.wikipedia.org/wiki/Data_Distribution_Service
3. DDS Portal – Data Distribution Services, accessed July 8, 2025, https://www.dds-foundation.org/
4. Industrial Protocols: Fundamentals of Data Distribution Service (DDS), accessed July 8, 2025, https://www.racoman.com/blog/industrial-protocols/fundamentals-of-data-distribution-service-dds
5. DDS Middleware. What is DDS middleware: | by Huseyin Kutluca | Software Architecture Foundations | Medium, accessed July 8, 2025, https://medium.com/software-architecture-foundations/dds-middleware-1af525f69753
6. What's in the DDS Standard? - DDS Foundation, accessed July 8, 2025, https://www.dds-foundation.org/omg-dds-standard/
7. en.wikipedia.org, accessed July 8, 2025, [https://en.wikipedia.org/wiki/Data_Distribution_Service#:~:text=DDS%20is%20a%20networking%20middleware,)%20and%20publish%20%22samples%22.](https://en.wikipedia.org/wiki/Data_Distribution_Service#:~:text=DDS is a networking middleware,) and publish "samples".)
8. Introduction to DDS - eProsima, accessed July 8, 2025, https://www.eprosima.com/developer-resources/whitepapers/dds
9. What is DDS? - DDS Foundation, accessed July 8, 2025, https://www.dds-foundation.org/what-is-dds-3/
10. Introduction to DDS (Data Distribution Service): Real-Time Data Communication Made Easy!, accessed July 8, 2025, https://erhanbakirhan.medium.com/introduction-to-dds-data-distribution-service-real-time-data-communication-made-easy-d6f4badddd6f
11. Navigating IIoT Protocols: Comparing DDS and MQTT - RTI, accessed July 8, 2025, https://www.rti.com/blog/comparing-dds-and-mqtt
12. Comparing the Performance of Zenoh, MQTT, Kafka, and DDS, accessed July 8, 2025, https://zenoh.io/blog/2023-03-21-zenoh-vs-mqtt-kafka-dds/
13. rustdds - Rust - Docs.rs, accessed July 8, 2025, https://docs.rs/rustdds
14. Introducing Dust DDS - A native Rust implementation of the Data Distribution Service (DDS) middleware - S2E Software Systems, accessed July 8, 2025, https://www.s2e-systems.com/resources/articles/introducing_dust_dds/
15. 4. Keys and Instances - RTI Connext Getting Started documentation - RTI Community, accessed July 8, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/getting_started_guide/csharp/intro_keys_instances.html
16. Chapter 11 Quality of Service (QoS) - RTI Community, accessed July 8, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/users_manual/users_manual/Quality_of_Service__QoS_.htm
17. Quality of Service (QoS) Policies [DDS Foundation Wiki], accessed July 8, 2025, https://www.omgwiki.org/ddsf/doku.php?id=ddsf:public:guidebook:06_append:glossary:q:quality_of_service_qos_policies
18. QoS | Data Distribution Service (DDS) Community RTI Connext Users, accessed July 8, 2025, https://community.rti.com/glossary/qos
19. Interoperability RustDDS x OpenDDS | by Alghani - Medium, accessed July 8, 2025, https://medium.com/@alghani63/interoperability-rustdds-x-opendds-7cf77fef7164
20. rustdds::policy - Rust - Docs.rs, accessed July 8, 2025, https://docs.rs/rustdds/latest/rustdds/policy/index.html
21. Quality of Service - OpenDDS 3.28.1, accessed July 8, 2025, https://opendds.readthedocs.io/en/dds-3.28.1/devguide/quality_of_service.html
22. Quality of Service - OpenDDS 3.27.0, accessed July 8, 2025, https://opendds.readthedocs.io/en/dds-3.27/devguide/quality_of_service.html
23. 4. Quality of Service - The Data Distribution Service Tutorial, accessed July 8, 2025, https://download.zettascale.online/www/docs/Vortex/html/ospl/DDSTutorial/qos.html
24. Validation of interoperability of products compliant with OMG DDS-RTPS standard. - GitHub, accessed July 8, 2025, https://github.com/omg-dds/dds-rtps
25. ZeroMQ vs. DDS Software: What's the Difference? | RTI, accessed July 8, 2025, https://www.rti.com/blog/2017/04/20/why-would-anyone-use-dds-over-zeromq
26. Rust Programming Language, accessed July 8, 2025, https://www.rust-lang.org/
27. Why use Rust? - Reddit, accessed July 8, 2025, https://www.reddit.com/r/rust/comments/1j8wnrk/why_use_rust/
28. Why Rust is the most admired language among developers - The GitHub Blog, accessed July 8, 2025, https://github.blog/developer-skills/programming-languages-and-frameworks/why-rust-is-the-most-admired-language-among-developers/
29. Why is Rust Language Becoming Popular and Should You Learn it? - Emeritus, accessed July 8, 2025, https://emeritus.org/blog/coding-rust-programming-language/
30. Rust: The Future of Memory-Safe Software Development | Bitrock, accessed July 8, 2025, https://bitrock.it/blog/rust-the-future-of-memory-safe-software-development.html
31. Rust: Exploring Its Benefits for System-Level Programming - Blue Coding, accessed July 8, 2025, https://www.bluecoding.com/post/rust-exploring-its-benefits-for-system-level-programming
32. How beneficial is the memory safety in rust programming language, and why didn't other system languages like C++ include such a feature? Does it have some side effects? If yes, like what? - Quora, accessed July 8, 2025, https://www.quora.com/How-beneficial-is-the-memory-safety-in-rust-programming-language-and-why-didnt-other-system-languages-like-C-include-such-a-feature-Does-it-have-some-side-effects-If-yes-like-what
33. Memory-Safe Programming Languages and National Cybersecurity: - A Technical Review of Rust | by Adnan Masood, PhD. | Medium, accessed July 8, 2025, https://medium.com/@adnanmasood/memory-safe-programming-languages-and-national-cybersecurity-a-technical-review-of-rust-fbf7836e44b8
34. Rust Documentation, accessed July 8, 2025, https://doc.rust-lang.org/
35. rustdds - crates.io: Rust Package Registry, accessed July 8, 2025, https://crates.io/crates/rustdds/0.8.2/dependencies
36. rustdds - crates.io: Rust Package Registry, accessed July 8, 2025, https://crates.io/crates/rustdds/0.11.2
37. What are the unique benefits of Rust over C++? - Reddit, accessed July 8, 2025, https://www.reddit.com/r/rust/comments/13tw26c/what_are_the_unique_benefits_of_rust_over_c/
38. RustDDS - Atostek, accessed July 8, 2025, https://atostek.com/en/services/rust-dds/
39. ROS2 client library based on RustDDS - GitHub, accessed July 8, 2025, https://github.com/Atostek/ros2-client/
40. rustdds - crates.io: Rust Package Registry, accessed July 8, 2025, https://crates.io/crates/rustdds
41. Tutorial/Example of two separate processes communicating using RustDDS? #168 - GitHub, accessed July 8, 2025, https://github.com/jhelovuo/RustDDS/discussions/168
42. DataReader in rustdds::dds::with_key - Rust - Docs.rs, accessed July 8, 2025, https://docs.rs/rustdds/latest/rustdds/dds/with_key/struct.DataReader.html
43. Dust DDS: A safe-only Rust implementation of the Data Distribution Service (DDS) middleware - Reddit, accessed July 8, 2025, https://www.reddit.com/r/rust/comments/1badm16/dust_dds_a_safeonly_rust_implementation_of_the/
44. Introducing Dust DDS – A native Rust implementation of the Data Distribution Service (DDS) middleware - Reddit, accessed July 8, 2025, https://www.reddit.com/r/rust/comments/1c26aq6/introducing_dust_dds_a_native_rust_implementation/
45. s2e-systems/dust-dds: Rust implementation of the Data ... - GitHub, accessed July 8, 2025, https://github.com/s2e-systems/dust-dds
46. dust_dds - Rust - Docs.rs, accessed July 8, 2025, https://docs.rs/dust_dds
47. ROS 2 Alternative middleware report - ROS General - Open Robotics Discourse, accessed July 8, 2025, https://discourse.ros.org/t/ros-2-alternative-middleware-report/33771
48. Middleware Slander : r/ROS - Reddit, accessed July 8, 2025, https://www.reddit.com/r/ROS/comments/1g2mu1x/middleware_slander/
49. Roboticists: Have you used Rust with ROS[2]? What were your experiences like? - Reddit, accessed July 8, 2025, https://www.reddit.com/r/rust/comments/173upma/roboticists_have_you_used_rust_with_ros2_what/
50. Benchmarking - The Rust Performance Book, accessed July 8, 2025, https://nnethercote.github.io/perf-book/benchmarking.html
51. rustls/BENCHMARKING.md at main - GitHub, accessed July 8, 2025, https://github.com/rustls/rustls/blob/main/BENCHMARKING.md
52. How to benchmark Rust code with Criterion - Bencher, accessed July 8, 2025, https://bencher.dev/learn/benchmarking/rust/criterion/
53. Activity / Atostek/RustDDS - GitHub, accessed July 8, 2025, https://github.com/Atostek/RustDDS/activity
54. RustDDS/rustfmt.toml at master - GitHub, accessed July 8, 2025, https://github.com/Atostek/RustDDS/blob/master/rustfmt.toml
55. accessed January 1, 1970, https://github.com/Atostek/RustDDS/issues
56. ROADMAP.md - eclipse-iceoryx/iceoryx2 - GitHub, accessed July 8, 2025, https://github.com/eclipse-iceoryx/iceoryx2/blob/main/ROADMAP.md
57. dds and rust | Data Distribution Service (DDS) Community RTI Connext Users, accessed July 8, 2025, https://community.rti.com/forum-topic/dds-and-rust
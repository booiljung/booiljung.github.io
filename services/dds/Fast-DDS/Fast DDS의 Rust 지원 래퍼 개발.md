# Fast DDS의 Rust 지원 래퍼 개발


본 보고서의 첫 번째 장에서는 eProsima Fast DDS를 위한 고품질 Rust 래퍼 개발이 단순한 기술적 시도를 넘어, 현대의 안전 필수(Safety-Critical) 시스템 소프트웨어 개발 패러다임 변화에 대응하는 전략적 과제임을 논증한다. 이는 분산 시스템의 핵심 요소인 DDS의 위상과 시스템 프로그래밍의 미래로 부상한 Rust의 가치를 결합하여, 차세대 시스템의 안정성과 생산성을 극대화하기 위한 필연적 과정이다.


데이터 분산 서비스(Data Distribution Service, DDS)는 객체 관리 그룹(Object Management Group, OMG)이 정의한 실시간 시스템을 위한 데이터 중심의 발행-구독(Publish-Subscribe) 통신 프로토콜 표준이다.1 DDS의 핵심 아키텍처는 예측 가능하고, 확장 가능하며, 신뢰성 있는 정보 분배를 목표로 설계되었다.1 이러한 특성 덕분에 DDS는 다양한 미션 크리티컬 산업 분야에서 중추적인 역할을 수행하고 있으며, 특히 eProsima의 Fast DDS는 그 선두에 있다.

Fast DDS의 전략적 중요성은 주요 고성장 생태계에서의 표준 채택 현황을 통해 명확히 드러난다.

- **로보틱스 분야의 지배적 위치 (ROS 2):** eProsima Fast DDS는 로봇 운영체제 2(Robot Operating System 2, ROS 2)의 모든 장기 지원(Long-Term Support, LTS) 릴리스에서 기본 미들웨어로 채택되었다.4 이는 현대 로보틱스 애플리케이션의 방대한 생태계가 Fast DDS를 통신 기반으로 삼고 있음을 의미한다.6 ROS 2를 사용하는 개발자에게 Fast DDS는 선택이 아닌 표준이며, 이는 사실상 로보틱스 통신 프로토콜의 표준화를 주도하고 있음을 시사한다.
- **자동차 산업으로의 확장 (AUTOSAR):** DDS는 이제 AUTOSAR(AUTomotive Open System ARchitecture) Adaptive 플랫폼에 공식적으로 통합되었으며, 모든 AUTOSAR 플랫폼을 지원하는 기본 패키지(Foundation Package)의 일부가 되었다.8 특히 소프트웨어 정의 차량(Software-Defined Vehicles, SDV)의 복잡한 데이터 흐름을 처리하는 데 있어 기존의 SOME/IP보다 우월한 대안으로 평가받고 있다. 이는 존 아키텍처(Zonal Architecture)나 첨단 운전자 보조 시스템(ADAS)과 같은 차세대 자동차 기술에서 DDS의 역할이 점차 커지고 있음을 보여준다.9
- **산업용 사물 인터넷(IIoT)에서의 광범위한 적용:** Fast DDS와 그 파생 기술(예: Micro XRCE-DDS)은 마이크로컨트롤러부터 강력한 게이트웨이에 이르기까지 광범위한 IoT 장치에 사용되고 있다. iRobot의 Roomba와 같은 상용 제품이나 Ericsson과의 5G 통합 협력 사례는 Fast DDS가 이론적 표준을 넘어 실제 산업 현장에서 검증된 기술임을 증명한다.3

이러한 사실들은 Fast DDS가 단순히 여러 DDS 구현체 중 하나가 아니라, 로보틱스와 자동차라는 가장 중요하고 빠르게 성장하는 실시간 시스템 분야에서 사실상의 표준 통신 패브릭(fabric)으로 자리 잡았음을 보여준다. 이 강력한 네트워크 효과는 새로운 기술이나 프로그래밍 언어가 해당 분야에 진입하기 위한 전제 조건을 만든다. 즉, Rust가 이들 분야에서 성공적으로 채택되기 위해서는, 지배적인 DDS 구현체인 Fast DDS와의 최상급 통합(first-class integration)이 필수적이다. 따라서 `fast-dds-rs` 래퍼 개발은 Rust의 생태계를 이들 핵심 산업으로 확장하기 위한 매우 중요한 기반 기술 확보 프로젝트라 할 수 있다. 이것은 DDS를 대체하는 것이 아니라, 이미 산업 표준으로 자리 잡은 통신 인프라 위에서 더 안전한 애플리케이션을 구축할 수 있도록 지원하는 것이다.


Rust는 시스템 프로그래밍 언어의 오랜 난제였던 성능과 안전성 사이의 트레이드오프(trade-off)를 해결하며 등장했다. C++와 동등한 수준의 성능을 제공하면서도, 언어 차원에서 메모리 안전성을 보장하는 Rust의 특징은 DDS가 사용되는 안전 필수 시스템의 요구사항과 정확히 일치한다.

- **메모리 안전성 보장:** Rust의 가장 혁신적인 특징은 소유권(ownership)과 대여(borrowing) 시스템이다. 이 시스템은 컴파일 시점에 댕글링 포인터(dangling pointers), 이중 해제(double frees), 버퍼 오버플로(buffer overflows)와 같은 치명적인 메모리 관련 오류를 원천적으로 방지한다.16 이는 C++에서 버그와 보안 취약점의 주된 원인이 되는 문제들을 언어적으로 해결한 것이다.16
- **C++와 대등한 성능:** Rust는 가비지 컬렉터 없이 네이티브 코드로 컴파일되므로, C++와 비견될 만한 높은 성능을 제공한다.17 다양한 벤치마크에서 Rust는 C++와 대등하거나 특정 시나리오에서는 더 나은 성능을 보여주기도 한다.19 이는 성능 저하 없이 안전성을 확보할 수 있음을 의미한다.
- **안전한 동시성:** Rust의 타입 시스템, 특히 `Send`와 `Sync` 트레이트(trait)는 컴파일 시점에 데이터 경합(data races)을 방지하여 동시성 프로그래밍의 안전성을 보장한다.18 이는 다중 스레드 환경이 일반적인 현대 분산 시스템에서 매우 중요한 장점이다.
- **산업계의 주목과 채택:** Rust는 Google, Amazon, Microsoft 등 주요 기술 기업에서 시스템 수준의 작업에 채택되고 있으며, 리눅스 커널에 C 언어 외에 최초로 채택된 언어이기도 하다.16 미국 백악관조차 핵심 인프라에 대해 Rust와 같은 메모리 안전 언어로의 전환을 권고하고 있다.22

DDS가 주로 사용되는 로보틱스 및 자동차 산업의 핵심 과제는 신뢰성, 보안, 그리고 실시간 성능이다. Microsoft와 Google의 보고에 따르면, 대규모 C++ 코드베이스에서 발생하는 심각한 보안 취약점의 약 70%가 메모리 안전성 문제에서 기인한다.17 Rust의 설계 철학은 바로 이 가장 큰 실패의 원인을 직접적으로 겨냥한다. 따라서 Rust를 채택하는 것은 단순히 개발자의 선호도를 넘어, 복잡한 소프트웨어 프로젝트의 리스크를 근본적으로 줄이는 전략적 엔지니어링 결정이다. Rust의 가파른 학습 곡선이라는 비용 21은, 디버깅 시간의 획기적인 단축과 메모리 문제로 인한 치명적인 런타임 실패 가능성의 감소라는 더 큰 이익으로 상쇄된다.


Fast DDS는 매우 견고하게 설계된 C++ 라이브러리이지만, C++ 언어 자체의 한계로부터 완전히 자유로울 수는 없다. 실제로 CVE(Common Vulnerabilities and Exposures) 데이터베이스를 분석해 보면, Fast DDS를 포함한 여러 DDS 구현체에서 힙 오버플로, 잘못된 메모리 해제(bad-free), 악의적인 패킷으로 인한 프로세스 충돌 등 심각한 메모리 관련 취약점들이 보고된 바 있다.23 이는 정확히 Rust가 방지하도록 설계된 종류의 오류들이다.

DDS의 기반이 되는 RTPS(Real-Time Publish-Subscribe) 프로토콜 계층 또한 취약점의 원인이 될 수 있다. 예를 들어, Wireshark의 RTPS 분석기에서 메모리 누수 취약점이 발견된 사례가 있다.25 Rust 래퍼가 C++ 구현 자체를 수정할 수는 없지만, C++ 라이브러리와 상호작용하는 애플리케이션 코드를 메모리 안전하게 만들어 전체적인 공격 표면(attack surface)을 줄일 수 있다.

DDS는 인증 및 암호화를 위한 보안 플러그인을 제공하지만 1, 많은 취약점은 인증되지 않은 디스커버리(discovery) 패킷을 파싱하는 과정에서 발생한다. Rust의 강력한 오류 처리와 메모리 안전성은 이러한 종류의 공격에 대해 애플리케이션 계층을 훨씬 더 견고하게 만든다.

이러한 분석은 Rust 래퍼가 일종의 '안전 경계(safety boundary)' 또는 '방화벽' 역할을 할 수 있음을 시사한다. 만약 기반 C++ 라이브러리에 취약점이 존재하더라도, 래퍼를 통해 구축된 Rust 애플리케이션은 Rust의 대여 검사기(borrow checker)에 의해 보호된다. C++ 계층의 메모리 오염이 Rust 애플리케이션의 안전한(safe) 컴포넌트 메모리를 임의로 손상시키는 것을 방지할 수 있다. 이 경우, `unsafe` 키워드가 사용되는 FFI(Foreign Function Interface) 경계만이 감사가 필요한 핵심 영역이 되며, 이는 C++로 작성된 전체 애플리케이션을 감사하는 것에 비해 범위가 대폭 축소된다.

결론적으로, `fast-dds-rs` 래퍼 개발의 가장 중요한 전략적 이점은 '리스크 완화'이다. 이를 통해 조직은 기능이 풍부하고 고도로 최적화된 산업 표준 미들웨어인 Fast DDS를 계속 활용하면서도, 애플리케이션 로직은 증명 가능한 메모리 안전성을 제공하는 언어로 구축할 수 있다. 이는 양쪽의 장점을 모두 취하는 최적의 전략이다.


Rust 래퍼를 설계하기에 앞서, 대상이 되는 C++ 라이브러리인 Fast DDS의 아키텍처와 설계 관용구(idiom)를 심층적으로 이해하는 것은 필수적이다. 이 장에서는 Fast DDS의 구조를 수직적으로 분해하여 각 계층의 역할과 핵심 컴포넌트들의 상호작용 방식을 분석한다.


Fast DDS는 명확하게 구분된 계층형 아키텍처를 채택하고 있다.27 각 계층은 특정 역할을 수행하며, 상위 계층은 하위 계층의 복잡성을 추상화한다.

- **애플리케이션 계층 (Application Layer):** 사용자의 코드가 위치하는 최상위 계층으로, Fast DDS API를 활용하여 분산 시스템 통신을 구현한다.27
- **Fast DDS (DCPS) 계층:** OMG DDS 사양을 구현한 사용자 친화적인 고수준 API 계층이다. `DomainParticipant`, `Publisher`, `DataWriter`와 같은 추상화된 엔티티(Entity)를 제공하며, 사용 편의성에 초점을 맞추고 있다.1
- **RTPS 계층 (RTPS Layer):** 실시간 발행-구독(Real-Time Publish-Subscribe) 와이어 프로토콜을 구현한 계층이다. 이 저수준 API는 통신 내부 동작에 대한 더 세밀한 제어를 가능하게 하며, 서로 다른 DDS 벤더 간의 상호 운용성을 보장한다.1
- **전송 계층 (Transport Layer):** 다양한 물리적 또는 논리적 채널을 통해 데이터를 전송하기 위한 플러그형 아키텍처이다. Fast DDS는 UDPv4/v6, TCPv4/v6, 그리고 공유 메모리(Shared Memory, SHM)를 위한 구현체를 제공한다.1

Fast DDS가 명시적으로 두 개의 API 계층, 즉 사용 편의성을 위한 고수준 DDS 계층과 세밀한 제어를 위한 저수준 RTPS 계층을 제공한다는 점은 1 래퍼 설계에 직접적인 영향을 미친다. 포괄적인 Rust 래퍼 역시 이러한 이중 API 구조를 반영해야 한다. 고수준의 관용적인 Rust 래퍼는 DDS C++ API를 대상으로 하여 안전하고 편리한 사용 경험을 제공해야 한다. 동시에, 저수준의, 아마도 더 많은 

`unsafe` 코드를 포함하는 모듈을 통해 RTPS API를 노출함으로써, DDS 추상화를 우회해야 하는 전문가 사용자의 요구를 충족시킬 수 있다. 이는 C++ 라이브러리의 설계 철학을 Rust 래퍼에서도 계승하는 것이다.


Fast DDS의 API는 여러 핵심 엔티티들의 상호작용을 통해 구성된다. 이들 엔티티의 생성, 소멸, 그리고 소유권 관계를 이해하는 것이 래퍼 설계의 핵심이다.

- **`DomainParticipant`:** 애플리케이션이 DDS 도메인에 참여하기 위한 진입점이다. `Publisher`, `Subscriber`, `Topic`을 생성하는 팩토리(factory) 역할을 수행한다.27
- **`Publisher` / `Subscriber`:** 각각 `DataWriter`와 `DataReader`를 담는 그룹화 엔티티이다. 이들을 통해 포함된 엔티티들에 일관된 QoS(Quality of Service) 정책을 적용할 수 있다.30
- **`DataWriter` / `DataReader`:** 특정 `Topic`에 대한 데이터를 송수신하는 주체이다. 애플리케이션의 데이터 로직과 직접적으로 상호작용하는 핵심 엔티티이다.30
- **`Topic`:** 도메인 내에서 유일한 이름과 특정 데이터 타입, 그리고 QoS 정책을 연결하여 통신의 주제를 정의한다.1
- **`TypeSupport`:** 특정 데이터 타입과 연관된 C++ 클래스로, 직렬화(serialization), 역직렬화(deserialization), 그리고 키(key) 관리를 담당한다.32

C++ API는 엄격한 팩토리 패턴을 따른다. `Publisher`는 `DomainParticipant`로부터 생성되고 29, `DataWriter`는 `Publisher`로부터 생성된다.30 소멸 과정 역시 이 계층 구조를 따른다. 예를 들어, `DataWriter`는 `Publisher::delete_datawriter()` 멤버 함수를 통해 소멸된다.30 이는 C++ 세계에서 엄격한 소유권 및 생명주기(lifetime) 계층 구조를 형성한다.

Rust 래퍼는 이 구조를 반드시 존중해야 한다. Rust의 `DataWriter` 래퍼 구조체는 단순히 C++ `DataWriter` 객체를 가리키는 포인터만 가질 수 없다. `Drop` 트레이트 구현 시 소멸 작업을 수행하기 위해 C++ `Publisher` 객체가 필요하므로, 자신을 생성한 `Publisher` 래퍼에 대한 참조(또는 `Arc`와 같은 스마트 포인터)를 함께 보유해야 한다. 이는 `DataWriter`들이 아직 살아있는 동안 `Publisher`가 먼저 소멸되는 것을 방지하기 위해 Rust의 생명주기 명시(lifetime annotation)를 신중하게 관리해야 함을 의미하며, 이는 Rust의 소유권 시스템이 빛을 발하는 완벽한 사용 사례이다.


Fast DDS의 동작은 몇 가지 핵심 메커니즘에 의해 구동된다. 이들을 Rust 환경으로 자연스럽게 이식하는 것이 래퍼의 주요 과제 중 하나이다.

- **토픽 및 타입 관리 (IDL):** Fast DDS는 데이터 타입을 정의하기 위해 OMG의 인터페이스 정의 언어(Interface Definition Language, IDL)에 의존한다. `Fast-DDS-Gen`이라는 Java 기반 도구를 사용하여 이 IDL 파일로부터 C++ 소스 코드(`TypeSupport` 클래스 포함)를 생성한다.1 이 생성된 코드는 사용자가 정의한 타입을 미들웨어와 연결하는 다리 역할을 한다.
- **디스커버리 프로토콜 (Discovery Protocols):** `DomainParticipant`들이 서로를 발견하고, 각자의 `DataWriter`와 `DataReader`를 매칭시키는 메커니즘이다. Fast DDS는 P2P 방식의 Simple Discovery(기본값), 중앙 서버 방식의 Discovery Server, 그리고 사전 구성 방식의 Static Discovery를 지원한다.1
- **서비스 품질 (Quality of Service, QoS):** DDS 엔티티의 비기능적 동작을 제어하는 풍부한 설정 정책 모음이다. 신뢰성(Reliable vs. Best Effort), 이력(History, 얼마나 많은 샘플을 보관할지), 내구성(Durability) 등이 대표적인 QoS 정책이다.1

C++ 개발 환경은 IDL 중심적이다. 그러나 Rust 래퍼를 사용하는 Rust 개발자는 Java 도구(`Fast-DDS-Gen`)를 실행하고 생성된 C++ 코드를 수동으로 통합하는 방식을 원하지 않을 것이다. 이는 상당한 사용성의 장벽을 만든다. 따라서 진정으로 관용적인 `fast-dds-rs` 래퍼는 타입 관리를 위한 자체적인 해결책을 제시해야 한다. 이는 다음과 같은 중요한 설계 결정으로 이어진다:

1. **`serde` 활용:** `RustDDS`의 사례처럼 2, 사용자가 

   `#`를 붙인 Rust 구조체를 정의하도록 하는 방식이다. 이 경우 래퍼는 Fast DDS의 동적 타입(DynamicTypes) 기능을 34 활용하여 런타임에 이 타입들을 등록하고 사용해야 한다. 이 방식은 유연하지만 성능 오버헤드가 발생할 수 있다.

2. **Rust 기반 IDL 코드 생성기 개발:** 더 발전된 해결책은 IDL 파일을 파싱하여 필요한 `#[repr(C)]` Rust 구조체와 이를 Fast DDS의 정적 타입 시스템에 등록하기 위한 FFI 글루(glue) 코드를 자동으로 생성하는 `proc-macro`나 `build.rs` 스크립트를 만드는 것이다. 이는 최상의 성능과 C++ DDS 개발자에게 친숙한 작업 흐름을 제공하지만, 상당한 개발 노력이 필요하다.


Fast DDS는 리스너(listener) 클래스에 기반한 이벤트 주도 모델을 사용한다. 이 모델을 Rust의 현대적인 비동기 패러다임과 어떻게 조화시킬 것인지는 래퍼의 품질을 결정하는 중요한 요소이다.

- **리스너 클래스:** 사용자는 `DataReaderListener`와 같은 기반 클래스를 상속받아 `on_data_available()`이나 `on_subscription_matched()` 같은 가상(virtual) 메서드를 재정의(override)하는 방식으로 자신만의 리스너를 구현한다.35
- **콜백 등록:** 이렇게 구현된 커스텀 리스너의 인스턴스는 `create_datareader()`와 같은 팩토리 메서드에 인자로 전달되어 등록된다.29
- **이벤트 기반 호출:** Fast DDS 미들웨어는 내부 스레드에서 실행되면서, 특정 이벤트가 발생하면 등록된 리스너의 콜백 메서드를 호출한다.35

이러한 C++의 리스너 패턴은 전형적인 객체 지향 콜백 메커니즘이다. 그러나 현대의 관용적인 Rust는 `tokio`와 같은 실행기(executor)를 사용하는 `async/await` 패러다임을 강력하게 선호한다. 리스너 인터페이스를 1대 1로 직접 래핑하는 것은 Rust 개발자에게 매우 이질적이며, 광범위한 Rust `async` 생태계와 통합하기도 어렵다.

따라서 고품질의 래퍼는 이러한 패러다임의 불일치를 해소해야 한다. 권장되는 패턴은 내부적으로 단 하나의 일반적인(generic) C++ 리스너를 구현하는 것이다. 이 리스너의 `on_data_available` 콜백은 사용자 코드를 직접 실행하는 대신, 수신된 데이터를 스레드 안전한 채널(예: `tokio::mpsc::channel`)을 통해 전송하는 역할만 한다. 그리고 Rust API는 이 채널의 수신단을 `async Stream`으로 노출한다. 이렇게 하면 사용자는 `while let Some(sample) = stream.next().await`와 같은 자연스럽고 관용적인 `async` 루프에서 새로운 메시지를 기다릴 수 있으며, 다른 `async` 작업과도 원활하게 통합할 수 있다. 이는 C++의 "푸시(push)" 모델(콜백)을 Rust의 "풀(pull)" 모델(`Stream`)로 변환하는 효과적인 방법이다.


이 장에서는 FFI(Foreign Function Interface)의 기술적 과제를 심층적으로 다룬다. 일반적인 원칙에서 나아가, Fast DDS와 같이 복잡한 C++ 라이브러리를 래핑할 때 발생하는 구체적이고 어려운 문제들을 분석한다.


Rust와 다른 언어 간의 상호운용성은 대부분 C 언어의 애플리케이션 바이너리 인터페이스(Application Binary Interface, ABI)를 공통분모로 삼는다. 이는 C++와 상호작용할 때도 기본적으로 적용되는 원칙이다.

- **`unsafe` 계약:** Rust에서 모든 FFI 호출은 `unsafe` 블록 안에서 이루어져야 한다.36 이는 Rust 컴파일러가 외부 코드의 불변성(invariants)을 검증할 수 없기 때문이다. 

  `unsafe` 키워드는 개발자가 해당 코드의 안전성을 직접 책임지겠다는 계약과 같다.

- **C 호환 데이터 레이아웃:** Rust 구조체가 C/C++ 구조체와 호환되는 메모리 레이아웃을 갖도록 하려면 `#[repr(C)]` 어트리뷰트를 사용해야 한다.37 이는 필드 순서 보장 및 패딩(padding) 규칙을 C 표준에 맞추도록 지시한다.

- **기본 타입 호환성:** FFI 경계를 넘나드는 데이터의 정확성을 위해 `libc` 크레이트나 `std::os::raw` 모듈에서 제공하는 C 호환 기본 타입을 사용하는 것이 매우 중요하다.36

- **기본 도구, `bindgen`:** `bindgen`은 C/C++ 헤더 파일을 분석하여 이러한 저수준 C 바인딩을 자동으로 생성해주는 표준 도구이다.41


FFI의 기준이 C ABI이지만, Fast DDS는 C++ 라이브러리이므로 C에는 없는 C++ 고유의 기능들을 처리해야 하는 복잡한 문제에 직면한다.

- **이름 맹글링 (Name Mangling):** C++ 컴파일러는 네임스페이스, 함수 오버로딩 등의 기능을 지원하기 위해 함수 이름을 복잡하게 변형하는데, 이를 이름 맹글링이라 한다. `extern "C"` 링크를 사용하면 이를 피할 수 있지만, C++ 클래스의 멤버 함수를 직접 래핑하려면 이 맹글링된 심볼 이름을 다루어야 한다.38
- **템플릿 (Templates):** C++ 템플릿은 컴파일 시점에 구체화(instantiation)되어 각각의 고유한 타입과 함수를 생성한다. 일반적인(generic) C++ 템플릿을 Rust에서 직접 표현할 방법은 없다. 따라서 래퍼는 사용해야 할 모든 구체화된 템플릿 인스턴스(예: `DataWriter<MyType>`)에 대해 개별적인 바인딩을 생성해야 한다.
- **가상 상속과 vtable:** C++의 가상 메서드는 vtable(함수 포인터 테이블)을 통해 구현된다. Rust는 C++의 vtable을 안전하게 생성하거나 상호작용할 수 없다. 특히 `DataReaderListener`처럼 사용자가 상속받아 구현해야 하는 클래스를 래핑하는 것은 매우 어려운 문제에 속한다.43
- **RAII와 객체 생명주기:** C++은 생성자(constructor)와 소멸자(destructor)를 통해 객체의 생명주기를 관리하는 RAII(Resource Acquisition Is Initialization) 패턴을 사용한다. Rust 역시 `Drop` 트레이트를 통해 유사한 개념을 사용하지만, 두 언어의 메모리 할당자는 서로 호환되지 않는다. Rust의 `Box`로 할당된 메모리를 C++의 `delete`로 해제하거나 그 반대의 경우는 정의되지 않은 행동(Undefined Behavior)을 유발한다.46

이러한 문제들은 Rust에서 복잡한 C++ API를 직접 래핑하는 것이 거의 불가능하거나 매우 불안정함을 시사한다. 이 문제를 해결하기 위한 가장 견고하고 일반적인 패턴은 별도의 C++ "심(shim)" 또는 "어댑터(adapter)" 계층을 만드는 것이다. 이 심 계층은 복잡한 C++ 기능들을 단순한 `extern "C"` 인터페이스로 변환하여 Rust가 쉽게 소비할 수 있도록 제공한다. 예를 들어, Rust가 `DataWriter<MyType>`이라는 템플릿 클래스를 직접 다루는 대신, 심 계층은 `MyType_DataWriter_new()`, `MyType_DataWriter_write()`, `MyType_DataWriter_delete()`와 같이 C와 호환되는 시그니처를 가진 함수들을 노출한다.

결론적으로, 래퍼 개발은 단순히 Rust 프로젝트가 아니라, 작지만 매우 중요한 C++ 컴포넌트를 포함하는 작업이 된다. 이 심 계층이 실질적인 FFI 경계가 되어, 복잡한 C++ 관용구를 `bindgen`이나 `cxx`와 같은 도구가 쉽게 처리할 수 있는 단순한 C 스타일 함수로 변환해준다. 이는 Rust 측의 복잡성을 크게 줄이고, 가장 어려운 FFI 로직을 작은 C++ 파일 안에 격리시키는 효과를 가져온다.


심 계층을 통해 인터페이스가 단순화되더라도, 두 언어 간의 관용적 차이를 메우기 위한 몇 가지 핵심 패턴이 필요하다.

- **C++ 객체를 위한 불투명 포인터 (Opaque Pointers):** Rust에서 C++ 객체를 표현하는 표준적인 방법은 불투명 포인터(`*mut CppObject`)를 사용하는 것이다. Rust 래퍼 구조체는 이 포인터를 멤버로 가진다. 객체는 C++ 함수에 의해 생성되고, 그 포인터가 Rust로 전달된다. Rust 래퍼의 `Drop` 트레이트 구현은 객체를 파괴하기 위해 다시 C++ 함수를 호출한다.46
- **콜백 트램펄린 패턴 (Callback Trampoline Pattern):** C++ 콜백을 처리하기 위해서는 세 부분으로 구성된 시스템이 필요하다 49:
  1. **사용자의 클로저:** 사용자가 실행하고자 하는 관용적인 Rust 클로저 (`FnMut` 또는 `FnOnce`).
  2. **트램펄린 함수:** C++ 라이브러리가 기대하는 시그니처(예: `fn(user_data: *mut c_void)`)를 가진 정적 `extern "C"` Rust 함수.
  3. **`user_data` 포인터:** 사용자의 클로저는 `Box`로 힙에 할당되고, `Box::into_raw`를 통해 얻은 원시 포인터가 C++ 라이브러리에 `user_data`로 전달된다. 트램펄린 함수는 이 포인터를 다시 `Box<dyn FnMut>`으로 변환하여 호출한다.
- **오류 처리:** C++ 함수는 종종 정수 반환 값, 예외(exception), 또는 `nullptr` 반환을 통해 오류를 알린다. Rust 래퍼는 이러한 오류 신호들을 관용적인 `Result<T, E>` 및 `Option<T>` 타입으로 변환해야 한다.2

불투명 포인터 역참조, `void*`로부터의 캐스팅, FFI 함수 호출 등 이 모든 패턴들은 `unsafe` 블록을 필요로 한다. 좋은 래퍼의 핵심 원칙은 이 모든 필수적인 `unsafe` 로직을 래퍼 크레이트 내부에 완벽하게 봉인하고, 최종 사용자에게는 100% 안전한(safe) 고수준 API만을 노출하는 것이다.

결국 래퍼의 주된 가치는 이러한 '불안전성의 캡슐화'에 있다. `fast-dds-rs`의 사용자는 일반적인 작업을 위해 `unsafe` 블록을 작성할 필요가 전혀 없어야 한다. 래퍼 개발자의 유지보수 부담은 자신이 사용한 `unsafe` 코드가 Rust와 Fast DDS의 모든 불변성을 위반하지 않고 건전(sound)하다는 것을 증명하는 데 있다. 이는 매우 높은 기준이며, `miri`와 같은 도구를 포함한 광범위한 테스트를 요구한다.52


이 장에서는 FFI 바인딩을 생성하고 구조화하는 방법에 대한 핵심적인 구현 전략을 집중적으로 논의한다. 이는 래퍼 개발의 성패를 좌우할 수 있는 가장 중요한 기술적 결정이다.


Rust 생태계에서는 FFI 바인딩을 두 개의 크레이트로 분리하는 것이 일반적인 관례이다.53

- **`\*-sys` 크레이트:** `bindgen`과 같은 도구로 생성된 원시적이고 `unsafe`한 FFI 선언을 포함한다. 네이티브 라이브러리와의 링크를 책임진다. C API를 최소한으로, 직접적으로 매핑하는 역할을 한다.
- **고수준 크레이트:** `-sys` 크레이트에 의존하며, 모든 `unsafe` 호출을 캡슐화하는 안전하고 관용적인 고수준 Rust API를 제공한다.

이러한 관심사의 분리는 다른 크레이트가 필요에 따라 원시 바인딩에 직접 의존할 수 있게 해준다. 또한 빌드 스크립트의 복잡성과 `unsafe` 코드를 격리시키며, 고수준 크레이트와 저수준 크레이트가 독립적으로 버전을 관리할 수 있게 한다.53

`-sys` 크레이트는 빌드 시 코드를 다운로드해서는 안 되며, 시스템에 설치된 라이브러리를 찾거나 벤더링된(vendored) 소스 코드를 직접 빌드해야 한다.55

`pkg-config`와 같은 도구를 사용하여 라이브러리를 찾고, 다른 빌드 스크립트가 사용할 수 있도록 헤더와 라이브러리 경로를 노출하는 것이 모범 사례이다.55


- **프로세스:** `bindgen`은 `libclang`을 사용하여 C/C++ 헤더 파일을 파싱하고, 해당하는 Rust `extern` 블록과 `#[repr(C)]` 구조체 정의를 자동으로 생성한다.41
- **장점:**
  - **완전한 API 커버리지:** 복잡한 기능에 대해 C 호환 심(shim) 계층이 제공된다는 가정 하에, Fast DDS의 전체 C++ API 표면에 대한 바인딩을 생성할 수 있다.
  - **자동화:** 함수 시그니처를 수동으로 변환하는 지루하고 오류가 발생하기 쉬운 작업을 줄여준다.41
- **단점:**
  - **기본적으로 `unsafe`:** 생성된 바인딩은 전적으로 `unsafe`하며, 컴파일러는 그 정확성을 검증할 수 없다.37
  - **높은 유지보수 비용:** 생성된 코드는 수동으로 안전한 추상화 계층으로 래핑해야 한다. 이는 Fast DDS처럼 거대한 라이브러리의 경우 엄청난 작업량이다. C++ API가 변경될 때마다 바인딩을 다시 생성하고 안전한 래퍼를 업데이트해야 할 수 있다.52
  - **빌드 시 의존성:** `build.rs` 스크립트에서 실행될 경우, 사용자 시스템에 `libclang`이 설치되어 있어야 하므로 상당한 진입 장벽이 될 수 있다.59 미리 생성된 바인딩을 저장소에 포함시키면 이 문제를 피할 수 있지만, 다른 플랫폼이나 라이브러리 버전 간의 호환성 문제가 발생할 수 있다.56
- **사례:** `cyclonedds-sys` 크레이트는 이러한 접근 방식의 실제 사례로, 안전한 `cyclonedds-rs` 래퍼가 소비할 수 있도록 원시 `bindgen` 출력물을 제공한다.60


- **프로세스:** 개발자가 `#[cxx::bridge]` 모듈 내에서 FFI 경계를 수동으로 정의하고, Rust와 C++ 양쪽에서 공유될 타입과 함수 시그니처를 선언한다. 그러면 `cxx`는 두 언어에 대한 심(shim) 코드를 생성하고, 정적 분석을 통해 시그니처가 일치하고 안전하게 사용되는지 확인한다.61
- **장점:**
  - **보장된 안전성:** `cxx`는 안전한 FFI 브릿지를 제공하도록 설계되어, 수동 FFI에서 흔히 발생하는 많은 함정을 제거한다. `std::unique_ptr`, `std::string`과 같은 C++ 타입을 이해하고, 안전하고 관용적인 변환을 제공한다.62
  - **`unsafe` 코드 감소:** `cxx`가 내부적으로 `unsafe` 심을 처리하므로, 대부분의 FFI 상호작용을 안전한 Rust 코드에서 수행할 수 있다.64
  - **컴파일 시 검증:** Rust 브릿지 정의와 C++ 헤더 시그니처가 일치하지 않으면 빌드가 실패하여, 주요 FFI 오류 유형을 사전에 방지한다.61
- **단점:**
  - **제한적:** `cxx`는 C++ 기능의 일부만 지원한다. 가상 상속이나 고급 템플릿과 같은 복잡한 패턴은 직접 지원되지 않을 수 있으며, 여전히 C 스타일의 심 계층이 필요할 수 있다.44
  - **수동 브릿지 정의:** 개발자가 `cxx::bridge` 모듈에 인터페이스를 명시적으로 작성해야 한다. 이는 일종의 "반복"이지만, 바로 이 점 때문에 안전성 검사가 가능하다.64
- **사례:** `cxx-qt` 프로젝트는 `cxx`를 사용하여 Qt와 같은 거대한 C++ 프레임워크에 대한 안전한 바인딩을 구축하는 방법을 보여준다.66


Fast DDS는 방대하고 복잡한 API를 가지고 있다. 순수한 `bindgen` 접근 방식은 안전하게 래핑하기에 엄청난 노력이 필요하다. 순수한 `cxx` 접근 방식은 Fast DDS API의 더 복잡한 부분(예: 커스텀 보안 플러그인, XTypes 인트로스펙션)에서 한계에 부딪힐 가능성이 높다. 따라서 어느 한 가지 도구만으로는 완벽한 해결책이 될 수 없다.

가장 실용적이고 효과적인 길은 하이브리드 전략을 채택하는 것이다.

1. **`bindgen`을 통한 기반 구축:** `fast-dds-sys` 크레이트 내에서 `bindgen`을 사용하여 (필요시 C 심 계층을 통해) 전체 Fast DDS API에 대한 원시적이고 `unsafe`한 바인딩을 생성한다. 이는 완전한 기반을 제공하며 API의 어떤 부분도 접근 불가능하지 않도록 보장한다. 최종 사용자의 `libclang` 의존성을 피하기 위해 생성된 코드는 저장소에 커밋한다.
2. **`cxx`를 통한 안전한 코어 구축:** 가장 일반적으로 사용되는 80%의 API(참여자 생성, 발행/구독, 기본 QoS, 데이터 쓰기/읽기 등)에 대해서는 `cxx`를 사용하여 안전하고 견고하며 검증된 브릿지를 구축한다. 이 `cxx` 브릿지는 고수준 `fast-dds` 크레이트 내에 위치하며, 내부적으로는 `fast-dds-sys`의 `unsafe` 함수를 호출한다.
3. **점진적 기능 확장:** 이 접근법을 통해 고수준 래퍼는 초기부터 핵심 기능에 대해 완전히 안전한 API를 노출할 수 있다. `-sys` 크레이트의 더 고급적이거나 복잡한 기능들은 시간이 지남에 따라 점진적으로 래핑하여 안전한 API로 노출하거나, 해당 기능이 필요한 전문가 사용자를 위해 `unsafe` 함수로 남겨둘 수 있다. 이는 초기 개발 속도, 장기적인 안전성, 그리고 API 완전성 사이의 균형을 맞추는 최적의 방법이다.


다음 표는 앞서 논의된 FFI 래퍼 생성 전략들의 장단점을 요약하여 하이브리드 전략의 타당성을 뒷받침한다.

| 특성                | `bindgen` (순수)                     | `cxx` (순수)                    | 제안된 하이브리드 전략                     |
| ------------------- | ------------------------------------ | ------------------------------- | ------------------------------------------ |
| **안전성 보장**     | 낮음 (모든 바인딩이 `unsafe`)        | 높음 (핵심 상호작용이 안전함)   | 높음 (핵심 API는 `cxx`로, 나머지는 캡슐화) |
| **API 완전성**      | 높음 (전체 API 표면 생성 가능)       | 중간 (지원되는 C++ 기능에 제한) | 높음 (`bindgen`으로 전체 기반 확보)        |
| **초기 개발 노력**  | 중간 (자동 생성되나, 래핑 필요)      | 높음 (수동 브릿지 정의 필요)    | 중간 (`cxx`로 시작, 점진적 확장)           |
| **유지보수 비용**   | 높음 (수동 래퍼의 광범위한 업데이트) | 낮음 (컴파일 시 검증)           | 중간 (코어는 낮고, 확장은 점진적)          |
| **빌드 시 의존성**  | 높음 (`libclang` 필요) 또는 취약함   | 낮음                            | 낮음 (미리 생성된 `-sys` 바인딩 사용)      |
| **관용적 API 지원** | 낮음 (수동으로 구현해야 함)          | 높음 (내장 타입 지원)           | 높음 (`cxx`의 장점 활용)                   |
| **C++ 기능 지원**   | 높음 (C 심을 통해 거의 모든 것 가능) | 제한적                          | 높음 (두 도구의 장점 결합)                 |

이 표는 의사 결정자가 주요 도구 옵션 간의 트레이드오프를 명확하게 이해하는 데 도움을 준다. 제안된 하이브리드 전략이 어떻게 다른 두 접근 방식의 강점(예: `cxx`의 높은 안전성과 `bindgen`의 높은 완전성)을 결합하는지 보여줌으로써, 아키텍처 결정의 근거를 강력하게 제시한다.


이전 장들의 분석을 바탕으로, 이 장에서는 `fast-dds-rs` 래퍼 라이브러리에 대한 구체적인 설계를 제시한다. 이는 분석을 실행 가능한 계획으로 전환하는 과정이다.


Rust FFI 관례에 따라 프로젝트를 두 개의 크레이트로 구성한다.

- **`fast-dds-sys`:**
  - **목적:** `libfastrtps`와 그 의존성인 `libfastcdr`에 대한 원시적이고 `unsafe`한 FFI 바인딩을 제공한다.
  - **생성:** Fast DDS 헤더를 대상으로 `bindgen`을 사용하여 `bindings.rs` 파일을 생성하고, 이 파일을 저장소에 커밋한다.56
  - **`build.rs`:** 다음을 책임지는 견고한 빌드 스크립트를 포함한다:
    - Linux에서는 `pkg-config`를, 다른 플랫폼에서는 해당 플랫폼에 맞는 방식을 사용하여 설치된 Fast DDS 라이브러리를 찾는다.55
    - `FASTDDS_PATH`와 같은 환경 변수를 통해 라이브러리 위치를 재정의할 수 있도록 허용한다.55
    - `vendored`와 같은 기능 플래그(feature flag)를 통해, 라이브러리가 설치되지 않은 사용자를 위해 git 서브모듈로 포함된 Fast DDS 소스 코드를 직접 컴파일하는 옵션을 제공한다.56 이는 사용 편의성을 크게 향상시킨다.
    - 필요한 `cargo:rustc-link-lib` 및 `cargo:rustc-link-search` 지시어를 출력한다.
  - **API:** 이 크레이트는 `bindings.rs`의 원시 `unsafe` 함수와 타입만을 노출하며, 어떠한 안전한 추상화도 제공하지 않는다.
- **`fast-dds`:**
  - **목적:** 고수준의 안전하고 관용적인 Rust 래퍼를 제공한다.
  - **의존성:** `fast-dds-sys`에 의존하며, `tokio`, `serde`, `thiserror`와 같은 다른 크레이트들도 활용한다.
  - **API 철학:** 공개 API는 100% 안전해야 한다. C++ 개념을 Rust 관용구로 변환하며, `RustDDS`의 설계에서 많은 영감을 얻는다.2


C++ API의 패턴들을 Rust 개발자에게 친숙하고 안전한 형태로 변환하는 것이 핵심 목표이다.

- **오류 처리:** `-sys` 크레이트의 C 스타일 정수 반환 코드와 `nullptr` 검사는 `thiserror` 크레이트를 사용하여 포괄적인 `enum Error`로 변환된다. 모든 실패 가능한 함수는 `Result<T, fast_dds::Error>`를 반환해야 한다.
- **엔티티 관리와 RAII:**
  - `DomainParticipant`, `DataWriter` 등 각 DDS 엔티티에 해당하는 Rust 래퍼 구조체를 정의한다.
  - 생성자(`new` 함수)는 `-sys` 크레이트의 C++ `create_*` 함수를 호출하고 반환된 불투명 포인터를 저장한다.
  - 각 래퍼 구조체는 `Drop` 트레이트를 구현한다. `drop` 메서드는 해당 엔티티를 생성한 팩토리 객체의 `delete_*` 함수를 호출하여 리소스가 항상 해제되도록 보장한다.46
  - 예를 들어, `DataWriter`의 `Drop` 구현은 `publisher->delete_datawriter(self.ptr)`를 호출해야 한다. 이를 위해 `DataWriter` 구조체는 `Arc<Publisher>`를 멤버로 가져서, `Publisher`가 살아있음을 보장하고 소멸 시점에 접근할 수 있도록 해야 한다.
- **비동기 작업:**
  - 리스너는 2장에서 설명한 채널 기반 접근 방식을 사용하여 래핑된다. 구독자(subscriber)를 생성하면, `async Stream`으로 변환될 수 있는 구조체를 반환한다.
  - 사용자는 `let mut stream = subscriber.into_stream(); while let Some(sample) = stream.next().await {... }`와 같은 코드로 비동기적으로 데이터를 처리할 수 있다.
- **QoS 정책:**
  - 하나의 거대한 QoS 구조체 대신, Rust의 타입 시스템과 빌더 패턴(builder pattern)을 사용하여 더 안전한 설정 API를 제공한다.
  - 예: `let qos = QosPolicy::default().reliable().history(History::KeepLast(10));`
  - 이러한 접근 방식은 유효하지 않은 QoS 조합을 가능한 한 컴파일 시점에 방지할 수 있다.


사용자가 일반적인 Rust 구조체를 정의하여 DDS 통신에 원활하게 사용하는 것이 최종 목표이다. 이를 위해 다음과 같은 해결책을 제안한다.

- **`fast-dds-codegen` 크레이트 제안:**
  - 별도의 전용 `proc-macro` 또는 `build.rs` 기반 크레이트이다.
  - **입력:** IDL 파일 (`.idl`).
  - **프로세스:**
    1. IDL 파일을 파싱한다.
    2. `serde::Serialize`와 `serde::Deserialize`를 파생 구현한 `#[repr(C)]` Rust 구조체를 생성한다.
    3. 해당 타입을 위한 C++ 헤더 파일(`.h`)과 소스 파일(`.cpp`)을 생성한다.
    4. 사용자의 `build.rs`에서, 생성된 `.idl`에 대해 `fastddsgen`을 호출하여 C++ `TypeSupport` 객체 파일을 생성한다.
    5. 사용자의 `build.rs`에서, 생성된 `.cpp` 심 코드를 컴파일한다.
    6. 생성된 Rust 코드는 `DomainParticipant::register_type`에 사용할 정적 `TypeSupport` 인스턴스에 대한 포인터를 얻기 위해 필요한 FFI 함수를 포함한다.
  - **장점:** 이 방식은 전체 타입 처리 파이프라인을 자동화하여, C++ 작업 흐름과 유사하면서도 결과적으로는 관용적인 Rust 코드를 사용하는 "그냥 되는(just works)" 경험을 사용자에게 제공한다. 상당한 개발 노력이 필요하지만, 가장 견고하고 사용자 친화적인 솔루션이다.


다음 표는 API 설계의 "번역 가이드" 역할을 하며, 래퍼의 가치를 명확히 보여준다.

| Fast DDS C++ 패턴                                      | 관용적 `fast-dds-rs` 패턴                                    |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| `ReturnCode_t create_datawriter(...)`                  | `Publisher::create_datawriter(...) -> Result<DataWriter, Error>` |
| `DataReaderListener` 클래스와 가상 `on_data_available` | `Subscriber::into_stream() -> impl Stream<Item = Sample>`    |
| `DDS::DataReaderQos` 구조체 직접 조작                  | `QosPolicy::default().reliable().history(...)` 빌더 패턴     |
| `writer->write(&sample)`                               | `writer.write(sample)?`                                      |
| `reader->take_next_sample(...)` 루프                   | `stream.next().await -> Option<Sample>`                      |

이 표는 "관용적 변환"이라는 추상적인 개념을 구체적이고 실체적인 예시로 보여줌으로써, 래퍼가 어떻게 복잡한 DDS 작업을 단순화하는지 명확하게 전달한다.


이 마지막 장에서는 앞선 모든 분석을 종합하여 전략적 결론을 도출하고, 프로젝트의 미래에 대한 전망을 제시한다.


4장에서 분석한 내용을 바탕으로, 하이브리드 전략을 공식적으로 권장한다. 즉, `bindgen`을 사용하여 포괄적인 `fast-dds-sys` 크레이트를 생성하되, 핵심 기능에 대해서는 `fast-dds` 크레이트 내에서 선별된 `cxx` 브릿지를 사용하여 주된 안전한 API를 구축하고, `-sys` 크레이트의 더 많은 `unsafe` 함수들을 시간이 지남에 따라 점진적으로 래핑하는 방식이다. 이는 안전성, 완전성, 그리고 관리 가능한 개발 노력 사이에서 최상의 균형을 제공한다.


거대한 프로젝트를 관리 가능한 단계로 나누어 제시한다.

- **1단계: 핵심 인프라 구축.** `fast-dds-sys`와 기본 `fast-dds` 크레이트 구조를 만든다. `DomainParticipant`, `Publisher`, `Subscriber`, `Topic`에 대한 래퍼를 구현한다. 이 단계에서는 기본 타입만 지원한다.
- **2단계: 데이터 경로 구현.** `DataWriter`와 `DataReader` 래퍼를 구현한다. 5장에서 제안한 `async` 스트림 기반 읽기 패턴을 구현한다. IDL을 통해 커스텀 타입을 처리하기 위한 `fast-dds-codegen` 크레이트를 개발한다.
- **3단계: QoS 및 고급 기능.** 가장 일반적으로 사용되는 QoS 정책(신뢰성, 이력, 내구성 등)에 대한 지원을 점진적으로 추가한다.
- **4단계: 보안 및 확장성.** DDS 보안 플러그인에 대한 래핑을 시도한다. 커스텀 전송 계층을 추가할 수 있는 API를 노출한다.
- **5단계: 생태계 통합.** ROS 2와의 통합을 위해 `rclrs` 지원을 개발한다. 즉, ROS 2 노드가 Rust 래퍼를 직접 사용할 수 있도록 `rmw_fastdds_rs` 구현체를 제공하여, `rclcpp`와 경쟁할 수 있는 성능을 목표로 한다.68


성공적인 오픈소스 프로젝트는 코드뿐만 아니라 커뮤니티와 거버넌스에 의해서도 좌우된다.

- **오픈소스 모델:** 프로젝트는 채택과 기여를 장려하기 위해 Fast DDS 자체와 동일한 허용적 라이선스(예: Apache 2.0) 하에 오픈소스로 진행되어야 한다.3
- **거버넌스:** Rust 프로젝트의 거버넌스 모델을 경량화하여 채택한다. 즉, 핵심 팀을 구성하고, 주요 설계 결정에 대해 공개적인 RFC(Request for Comments)와 유사한 프로세스를 사용하며, 명확한 기여 가이드라인을 유지한다.71
- **품질 및 문서화:** 프로젝트는 지원 수준, 테스트 전략(다른 DDS 벤더와의 상호 운용성 테스트 포함), 문서화 표준을 명시하는 "품질 선언(Quality Declaration)"을 갖추어야 한다.4 이처럼 복잡한 라이브러리에는 예제를 포함한 포괄적인 문서화가 필수적이다.


- **성능 목표:** 핵심 성공 지표 중 하나는 성능이다. 래퍼의 지연 시간(latency)과 처리량(throughput)을 `rclcpp` 및 `rclpy`와 비교하기 위한 벤치마크를 조기에 수립해야 한다.74 목표는 Rust의 제로 비용 추상화(zero-cost abstractions)를 활용하여 

  `rclcpp`의 성능에 근접하는 것이다.

- **안전성 목표:** 궁극적인 가치 제안은 안전성이다. 프로젝트는 고수준 크레이트에서 `unsafe` 코드 사용을 최소화하고, `-sys` 크레이트 사용의 건전성을 검증하기 위해 `miri`와 같은 도구를 적극적으로 사용해야 한다.

- **전략적 영향:** 성숙하고, 고성능이며, 안전한 `fast-dds-rs` 래퍼는 단순한 라이브러리 이상이 될 것이다. 이는 Rust로 차세대 로보틱스 및 자동차 시스템을 개발할 수 있게 하는 핵심 인프라가 될 것이다. 해당 분야에서 Rust의 진입 장벽을 낮추고, 가장 중요한 소프트웨어 구성 요소에 대해 업계가 메모리 안전 언어로 전환하는 데 상당한 영향을 미칠 수 있다.

궁극적으로, 이 래퍼의 성공은 긍정적인 피드백 루프를 만들 수 있다. 더 많은 로보틱스 개발자들이 Rust를 안전하고 쉽게 사용할 수 있게 되면, ROS 생태계 내에 더 많은 Rust 기반 도구와 라이브러리가 등장할 것이다. 이는 다시 Rust를 더욱 매력적인 선택지로 만들고, 잠재적으로 ROS 2에서 C++ 및 Python과 동등한 수준의 공식 지원 언어(Tier 1 language)로 격상시키는 계기가 될 수 있다. 이 래퍼에 대한 투자는 Rust 로보틱스 생태계 전체의 미래에 대한 투자이며, 그 성공은 수십억 달러 규모 산업의 주요 기술 전환을 가속화할 잠재력을 지니고 있다.


1. eProsima Fast DDS, accessed July 8, 2025, https://fast-dds.docs.eprosima.com/
2. Atostek/RustDDS: Rust implementation of Data Distribution ... - GitHub, accessed July 8, 2025, https://github.com/Atostek/RustDDS
3. The Importance of Robust Architecture in DDS Implementations and Best Practices, accessed July 8, 2025, https://www.eprosima.com/index.php/company-all/news/338-robust-architecture-dds-implementation
4. eProsima/Fast-DDS: The most complete DDS - Proven: Plenty of success cases. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 8, 2025, https://github.com/eProsima/Fast-DDS
5. eProsima/Fast-DDS-docs: Documentation of eProsima Fast DDS. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 8, 2025, https://github.com/eProsima/Fast-DDS-docs
6. ROS 2 (Robot Operating System): overview and key points for robotics software | Robotnik ®, accessed July 8, 2025, https://robotnik.eu/ros-2-robot-operating-system-overview-and-key-points-for-robotics-software/
7. Rust for Robots, Part 1: Why Rust for Robotics? - Tangram Visions Blog, accessed July 8, 2025, https://www.tangramvision.com/blog/why-rust-for-robots
8. DDS adopted into all AUTOSAR packages and platforms ... - eeNews Europe, accessed July 8, 2025, https://www.eenewseurope.com/en/dds-adopted-into-all-autosar-packages-and-platforms/
9. AUTOSAR and DDS: A Fresh Approach to Enabling Flexible Vehicle ..., accessed July 8, 2025, https://www.rti.com/blog/fresh-approach-to-enabling-flexible-vehicle-architectures
10. Connectivity at the Core: The Status of DDS in AUTOSAR - RTI, accessed July 8, 2025, https://www.rti.com/blog/status-of-dds-in-autosar
11. The Real Backbone of SDVs Is Data : RTI on the Future of DDS, accessed July 8, 2025, https://www.autoelectronics.co.kr/article/articleView.asp?idx=6234
12. Safety Mechanisms Using the DDS Middleware in Software-Defined Cars, accessed July 8, 2025, https://www.nxp.com/company/about-nxp/smarter-world-blog/BL-SAFETY-MECHANISMS
13. Streamlining Zonal Architecture with DDS and TSN - RTI, accessed July 8, 2025, https://www.rti.com/blog/streamlining-zonal-architecture-with-dds-and-tsn
14. Zonal-network architecture deployment in vehicle. - ResearchGate, accessed July 8, 2025, https://www.researchgate.net/figure/Zonal-network-architecture-deployment-in-vehicle_fig2_364504427
15. Internet of Things - eProsima, accessed July 8, 2025, https://www.eprosima.com/middleware/sectors/internet-of-things
16. Rust vs C++: Performance, Safety, and Use Cases Compared - CodePorting, accessed July 8, 2025, https://www.codeporting.com/blog/rust_vs_cpp_performance_safety_and_use_cases_compared
17. Rust vs C++ for Embedded Systems: A Comprehensive Comparison (2025) - CPP Cat, accessed July 8, 2025, https://cppcat.com/rust-vs-c-for-embedded-systems/
18. Rust Vs C++ Performance: When Speed Matters - BairesDev, accessed July 8, 2025, https://www.bairesdev.com/blog/when-speed-matters-comparing-rust-and-c/
19. Rust vs C++ Performance - YouTube, accessed July 8, 2025, https://www.youtube.com/watch?v=WnMin9cf78g
20. Interoperability - Rust API Guidelines, accessed July 8, 2025, https://rust-lang.github.io/api-guidelines/interoperability.html
21. Building a ROS2 node in Rust! - Mike Likes Robots, accessed July 8, 2025, https://mikelikesrobots.github.io/blog/ros2-rust-node/
22. Rust vs C/C++: is Rust better than C/C++ or is a "skill issue"? - Stack Overflow, accessed July 8, 2025, https://stackoverflow.com/beta/discussions/78239270/rust-vs-c-c-is-rust-better-than-c-c-or-is-a-skill-issue
23. Fast Dds CVEs and Security Vulnerabilities - OpenCVE, accessed July 8, 2025, https://www.opencve.io/cve?vendor=eprosima&product=fast_dds
24. Multiple Data Distribution Service (DDS) Implementations (Update A) - CISA, accessed July 8, 2025, https://www.cisa.gov/news-events/ics-advisories/icsa-21-315-02
25. CVE-2020-26420 - Security Bug Tracker - Debian, accessed July 8, 2025, https://security-tracker.debian.org/tracker/CVE-2020-26420
26. CVE-2020-26420 : Memory leak in RTPS protocol dissector in Wireshark 3.4.0 and 3.2.0 to 3.2.8 all - CVE Details, accessed July 8, 2025, https://www.cvedetails.com/cve/CVE-2020-26420/
27. 2. Library Overview - 3.2.2 - eProsima Fast DDS, accessed July 8, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/library_overview/library_overview.html
28. Fast DDS Documentation, accessed July 8, 2025, https://media.readthedocs.org/pdf/eprosima-fast-rtps/latest/eprosima-fast-rtps.pdf
29. FastDDS Publisher and Subscriber won't match in demo code - Stack Overflow, accessed July 8, 2025, https://stackoverflow.com/questions/78090303/fastdds-publisher-and-subscriber-wont-match-in-demo-code
30. Fast-DDS-docs/docs/fastdds/dds_layer/publisher/dataWriter/createDataWriter.rst at master, accessed July 8, 2025, https://github.com/eProsima/Fast-DDS-docs/blob/master/docs/fastdds/dds_layer/publisher/dataWriter/createDataWriter.rst
31. Fast-DDS-docs/docs/fastdds/dds_layer/subscriber/dataReader/createDataReader.rst at master - GitHub, accessed July 8, 2025, https://github.com/eProsima/Fast-DDS-docs/blob/master/docs/fastdds/dds_layer/subscriber/dataReader/createDataReader.rst
32. 1.4.3. Fast DDS - Vulcanexus Topic Intercommunication, accessed July 8, 2025, https://docs.vulcanexus.org/en/iron/rst/tutorials/core/deployment/dds2vulcanexus/dds2vulcanexus_topic.html
33. fastrtps: how to use DDS history on publish/subscribe level? - Stack Overflow, accessed July 8, 2025, https://stackoverflow.com/questions/60094952/fastrtps-how-to-use-dds-history-on-publish-subscribe-level
34. Fast DDS Spy Documentation, accessed July 8, 2025, https://fast-dds-spy.readthedocs.io/_/downloads/en/stable/pdf/
35. 3.4.5. DataReaderListener - 3.2.2 - eProsima Fast DDS, accessed July 8, 2025, https://fast-dds.docs.eprosima.com/en/v3.2.2/fastdds/dds_layer/subscriber/dataReaderListener/dataReaderListener.html
36. Foreign Function Interface - Secure Rust Guidelines, accessed July 8, 2025, https://anssi-fr.github.io/rust-guide/07_ffi.html
37. FFI - The Rustonomicon - Rust Documentation, accessed July 8, 2025, https://doc.rust-lang.org/nomicon/ffi.html
38. Item 34: Control what crosses FFI boundaries - Effective Rust, accessed July 8, 2025, https://effective-rust.com/ffi.html
39. FFI patterns #1 - Complex Rust data structures exposed seamlessly to C++ | Words, accessed July 8, 2025, https://crisal.io/words/2020/02/28/C++-rust-ffi-patterns-1-complex-data-structures.html
40. Foreign Function Interface / A Guide to Porting C and C++ code to Rust - locka99, accessed July 8, 2025, https://locka99.gitbooks.io/a-guide-to-porting-c-to-rust/content/features_of_rust/ffi.html
41. Rust FFI and bindgen: Integrating Embedded C Code in Rust, accessed July 8, 2025, https://blog.theembeddedrustacean.com/rust-ffi-and-bindgen-integrating-embedded-c-code-in-rust
42. rust-bindgen/book/src/tutorial-3.md at main - GitHub, accessed July 8, 2025, https://github.com/rust-lang/rust-bindgen/blob/master/book/src/tutorial-3.md
43. Native C++ FFI in the same manner as D / Issue #5853 / rust-lang/rust - GitHub, accessed July 8, 2025, https://github.com/rust-lang/rust/issues/5853
44. Support inheriting from pure virtual classes / Issue #200 / google ..., accessed July 8, 2025, https://github.com/google/autocxx/issues/200
45. Rust and C++ virtual functions - Reddit, accessed July 8, 2025, https://www.reddit.com/r/rust/comments/dym88t/rust_and_c_virtual_functions/
46. Rust/C++ Interop Part 1 - Just the Basics - Tyler Weaver, accessed July 8, 2025, https://tylerjw.dev/posts/rust-cpp-interop/
47. rust-ffi-guide/book/basic_request.md at master - GitHub, accessed July 8, 2025, https://github.com/Michael-F-Bryan/rust-ffi-guide/blob/master/book/basic_request.md
48. Wrapping a Rust struct in a C++ class - Stack Overflow, accessed July 8, 2025, https://stackoverflow.com/questions/32188582/wrapping-a-rust-struct-in-a-c-class
49. Idiomatic Rust for wrapping callback based C libraries? - Reddit, accessed July 8, 2025, https://www.reddit.com/r/rust/comments/npehj9/idiomatic_rust_for_wrapping_callback_based_c/
50. Rust program calling Rust method from C++ code - help - Rust Users Forum, accessed July 8, 2025, https://users.rust-lang.org/t/rust-program-calling-rust-method-from-c-code/54450
51. C FFI Questions: rust methods in callbacks - help - The Rust Programming Language Forum, accessed July 8, 2025, https://users.rust-lang.org/t/c-ffi-questions-rust-methods-in-callbacks/28489
52. Lessons learned from a successful Rust rewrite, accessed July 8, 2025, https://gaultier.github.io/blog/lessons_learned_from_a_successful_rust_rewrite.html
53. First ffi project: what can I improve? - code review - The Rust Programming Language Forum, accessed July 8, 2025, https://users.rust-lang.org/t/first-ffi-project-what-can-i-improve/127759
54. Exposing FFI from the Rust library - svartalf, accessed July 8, 2025, https://svartalf.info/posts/2019-03-01-exposing-ffi-from-the-rust-library/
55. Using C libraries in Rust: make a sys crate, accessed July 8, 2025, https://kornel.ski/rust-sys-crate
56. How to make a -sys crate? - help - The Rust Programming Language Forum, accessed July 8, 2025, https://users.rust-lang.org/t/how-to-make-a-sys-crate/13767
57. Smoother C/Rust interop with existing libraries. / Issue #481 / rust-embedded/wg - GitHub, accessed July 8, 2025, https://github.com/rust-embedded/wg/issues/481
58. The problem of safe FFI bindings in Rust - Abubalay, accessed July 8, 2025, https://www.abubalay.com/blog/2020/08/22/safe-bindings-in-rust
59. How does rust bindgen work - Reddit, accessed July 8, 2025, https://www.reddit.com/r/rust/comments/5y24yc/how_does_rust_bindgen_work/
60. sjames/cyclonedds-sys: Rust binding to cyclone-dds https ... - GitHub, accessed July 8, 2025, https://github.com/sjames/cyclonedds-sys
61. Tutorial - Rust C++ - CXX, accessed July 8, 2025, https://cxx.rs/tutorial.html
62. Rust ❤️ C++, accessed July 8, 2025, https://cxx.rs/
63. cxx - Rust - Docs.rs, accessed July 8, 2025, https://docs.rs/cxx/latest/cxx/
64. cxx - Rust - Docs.rs, accessed July 8, 2025, https://docs.rs/cxx
65. CXX - safe FFI between Rust and C++ - Android GoogleSource, accessed July 8, 2025, https://android.googlesource.com/platform/external/rust/cxx/+/16d23589cd6c3bdf2a38ded1ddc7eae725517397/README.md
66. I made a table comparing Rust bindings for Qt : r/rust - Reddit, accessed July 8, 2025, https://www.reddit.com/r/rust/comments/x30ayi/i_made_a_table_comparing_rust_bindings_for_qt/
67. How we wrap external C and C++ libraries in Rust - Evolve Benchmarking, accessed July 8, 2025, https://www.evolvebenchmark.com/blog-posts/how-we-wrap-external-c-and-cpp-libraries-in-rust
68. A first look at ROS~2 applications written in asynchronous Rust - ResearchGate, accessed July 8, 2025, https://www.researchgate.net/publication/392133870_A_first_look_at_ROS2_applications_written_in_asynchronous_Rust
69. A first look at ROS 2 applications written in asynchronous Rust - arXiv, accessed July 8, 2025, https://arxiv.org/html/2505.21323v1
70. [Literature Review] A first look at ROS~2 applications written in asynchronous Rust, accessed July 8, 2025, https://www.themoonlight.io/en/review/a-first-look-at-ros2-applications-written-in-asynchronous-rust
71. Governance - Rust Programming Language, accessed July 8, 2025, https://www.rust-lang.org/governance
72. Leadership and Governance | Open Source Guides, accessed July 8, 2025, https://opensource.guide/leadership-and-governance/
73. Contributing - Rust Existing C++ - Google, accessed July 8, 2025, https://google.github.io/autocxx/contributing.html
74. ROS2 Performance: rclpy is 30x-100x slower than rclcpp - Robotics Stack Exchange, accessed July 8, 2025, https://robotics.stackexchange.com/questions/98772/ros2-performance-rclpy-is-30x-100x-slower-than-rclcpp
75. ROS 2 Performance Benchmarking - ROS General - Open Robotics Discourse, accessed July 8, 2025, https://discourse.ros.org/t/ros-2-performance-benchmarking/44382



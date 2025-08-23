# eProsima Fast DDS Spy 전문가용 컴펜디움



로보틱스, 사물 인터넷(IoT), 자율 주행 자동차, 항공 우주 등 현대의 복잡한 분산 시스템은 수많은 독립적인 소프트웨어 컴포넌트 간의 원활하고 예측 가능한 데이터 교환을 요구한다.1 이러한 시스템에서 각 컴포넌트는 실시간으로 대량의 데이터를 생성하고 소비하며, 시스템 전체의 안정성과 성능은 이들 간의 통신 효율성에 크게 좌우된다. 전통적인 클라이언트-서버 또는 메시지 큐 방식은 중앙 집중적인 구조로 인한 단일 장애점(Single Point of Failure), 통신 설정의 복잡성, 실시간성 보장의 어려움 등의 한계를 가진다. 이러한 문제를 해결하기 위해 등장한 것이 바로 데이터 분산 서비스(Data Distribution Service, DDS)이다.3

DDS는 객체 관리 그룹(Object Management Group, OMG)에 의해 표준화된 통신 미들웨어 프로토콜로, 데이터 중심의 발행-구독(Publish-Subscribe) 모델을 기반으로 한다.3 이 모델은 데이터를 생성하는 '발행자(Publisher)'와 데이터를 소비하는 '구독자(Subscriber)'를 시간적, 공간적으로 완벽하게 분리(decoupling)함으로써 시스템의 유연성, 확장성, 견고성을 극대화한다. DDS는 '글로벌 데이터 공간(Global Data Space)'이라는 추상적인 개념을 통해, 각 애플리케이션이 마치 로컬 데이터에 접근하듯 원격의 데이터에 접근할 수 있게 하여 분산 시스템 설계를 획기적으로 단순화한다.4


수많은 DDS 구현체 중에서 eProsima Fast DDS는 고성능, 오픈소스라는 강력한 이점을 바탕으로 산업계와 학계에서 폭넓게 채택되고 있는 C++ 구현체이다.3 아파치 2.0 라이선스(Apache License 2.0) 하에 자유롭게 사용할 수 있으며 2, 특히 로봇 운영체제 ROS 2(Robot Operating System 2)의 기본 미들웨어로 채택되면서 로보틱스 분야에서 사실상의 표준으로 자리 잡았다.5

Fast DDS는 OMG의 DDS 및 RTPS(Real-Time Publish-Subscribe) 표준을 충실히 따르면서도, UDP, TCP, 공유 메모리(Shared Memory, SHM) 등 다양한 전송 계층 지원, 정교한 서비스 품질(Quality of Service, QoS) 설정, 플러그인 방식의 보안 기능, 중앙 서버 기반의 디스커버리 메커니즘 등 폭넓은 기능을 제공한다.3 이러한 특징 덕분에 Fast DDS는 단순한 IoT 장치부터 고도의 실시간 제어가 필요한 자율 주행 시스템에 이르기까지 광범위한 애플리케이션에 적용되고 있다.1


이처럼 강력하고 유연한 Fast DDS를 기반으로 분산 시스템을 개발하고 운영할 때, 개발자는 눈에 보이지 않는 데이터 흐름을 파악하고 문제를 진단해야 하는 과제에 직면한다. "특정 토픽의 데이터가 올바르게 전송되고 있는가?", "새로 추가된 노드가 기존 네트워크에 정상적으로 참여했는가?", "두 노드 간의 통신이 이루어지지 않는 원인은 무엇인가?"와 같은 질문에 답하기 위해서는 네트워크 내부를 정밀하게 들여다볼 수 있는 도구가 필수적이다.

바로 이 지점에서 `eProsima Fast DDS Spy`가 핵심적인 역할을 수행한다.8

`Fast-DDS-spy`는 DDS 네트워크를 위한 경량의 명령줄 인터페이스(Command Line Interface, CLI) 기반 진단 및 내부 검사(introspection) 도구이다.2 이 도구는 단순히 네트워크 패킷을 캡처하는 것을 넘어, DDS 네트워크에 참여하는 모든 엔티티(참여자, 발행자, 구독자)를 동적으로 발견하고, 이들 간의 관계 및 토픽 데이터를 사람이 읽기 쉬운 형식으로 실시간으로 보여준다.9 본 보고서는 

`Fast-DDS-spy`를 단순한 유틸리티가 아닌, 복잡한 DDS 데이터 흐름을 시각화하고, 디버깅하며, 깊이 이해하기 위한 필수적인 분석 장비로 규정하고, 그 원리부터 실제 적용까지 모든 것을 심도 있게 고찰하고자 한다.




DDS의 가장 근본적인 특징은 데이터 중심 발행-구독(Data-Centric Publish-Subscribe, DCPS) 모델에 있다.3 이는 기존의 메시지 중심 패러다임과 근본적인 차이를 보인다. 메시지 중심 시스템에서 애플리케이션은 '메시지'라는 단위로 통신하며, 메시지의 형식, 전송 방법, 수신자 등을 명시적으로 관리해야 한다. 반면, DCPS 모델에서 통신의 중심은 '데이터' 그 자체이다.

개발자는 '글로벌 데이터 공간'이라는 가상의 공유 데이터 저장소를 상상할 수 있다.4 발행자는 이 공간에 특정 주제(Topic)의 데이터를 '쓰고(write)', 구독자는 자신이 관심 있는 주제의 데이터를 '읽는다(read)'. 미들웨어인 DDS는 이 과정에서 데이터가 올바른 구독자에게 효율적이고 안정적으로 전달되도록 보장하는 모든 복잡한 작업을 처리한다. 발행자와 구독자는 서로의 존재나 위치를 알 필요가 없으며, 오직 데이터의 '주제'와 '타입'을 통해서만 연결된다.3 이러한 느슨한 결합(Loose Coupling)은 시스템의 각 컴포넌트를 독립적으로 개발, 테스트, 배포 및 업그레이드할 수 있게 하여 전체 시스템의 유지보수성과 확장성을 크게 향상시킨다.


DDS가 개념적 모델이라면, RTPS(Real-Time Publish-Subscribe)는 이 모델을 실제 네트워크상에서 구현하기 위한 구체적인 유선 프로토콜(wire protocol)이다.3 OMG에 의해 표준화된 RTPS는 서로 다른 벤더가 개발한 DDS 구현체들이 상호 운용될 수 있는 기반을 제공한다.10 예를 들어, eProsima Fast DDS로 개발된 애플리케이션과 RTI Connext DDS로 개발된 애플리케이션이 원칙적으로 동일한 DDS 도메인 내에서 통신할 수 있는 것은 바로 이 RTPS 표준 덕분이다.

RTPS는 UDP와 같은 비신뢰성 전송 프로토콜 위에서도 신뢰성 있는 통신을 구현할 수 있도록 설계되었다.10 이를 위해 RTPS는 다음과 같은 핵심 메시지들을 정의한다:

- **DATA:** 발행자(RTPS Writer)가 구독자(RTPS Reader)에게 데이터 객체의 변경 사항(새로운 값 또는 라이프사이클 변화)을 전달하기 위해 사용된다.
- **HEARTBEAT:** 발행자가 구독자에게 자신이 현재 보유하고 있는 데이터 샘플(CacheChange)의 상태를 알리기 위해 주기적으로 전송한다. 이를 통해 구독자는 자신이 놓친 데이터가 있는지 파악할 수 있다.
- **ACKNACK:** 구독자가 발행자에게 어떤 데이터를 성공적으로 수신했고 어떤 데이터가 누락되었는지를 알리기 위해 사용된다. 긍정적 확인(ACK)과 부정적 확인(NACK)의 역할을 모두 수행하며, 신뢰성 있는 통신(Reliable QoS)의 핵심 메커니즘이다.

`Fast-DDS-spy`와 같은 도구는 네트워크상의 이러한 RTPS 메시지를 "스니핑"하고 해석함으로써 DDS 네트워크의 상태를 파악한다. 따라서 RTPS에 대한 기본적인 이해는 `Fast-DDS-spy`의 동작 원리를 깊이 있게 이해하는 데 도움이 된다.


DDS 애플리케이션은 다음과 같은 표준화된 구성 요소, 즉 엔티티(Entity)들로 구성된다 3:

- **도메인 (Domain):** DDS 통신이 이루어지는 격리된 가상 네트워크. 동일한 도메인 ID를 가진 엔티티들만이 서로를 발견하고 통신할 수 있다. 이는 하나의 물리적 네트워크에서 여러 개의 독립적인 DDS 시스템을 운영할 수 있게 해준다.
- **도메인 참여자 (DomainParticipant):** DDS 도메인에 참여하는 애플리케이션의 진입점. 하나의 `DomainParticipant`는 여러 개의 발행자와 구독자를 생성하고 관리할 수 있다.
- **토픽 (Topic):** 데이터 교환의 핵심적인 매개체. 도메인 내에서 유일한 이름, 데이터 타입, 그리고 관련된 QoS(서비스 품질) 설정을 가진다. 발행자와 구독자는 동일한 토픽을 통해 연결된다.
- **발행자 (Publisher):** 하나 이상의 `DataWriter`를 관리하는 엔티티. 데이터 발행과 관련된 QoS 설정을 담당한다.
- **데이터 작성자 (DataWriter):** 특정 토픽에 데이터를 쓰는(발행하는) 실제 행위자. 애플리케이션은 `DataWriter`의 `write()` 함수를 호출하여 데이터를 발행한다.
- **구독자 (Subscriber):** 하나 이상의 `DataReader`를 관리하는 엔티티. 데이터 수신과 관련된 QoS 설정을 담당한다.
- **데이터 판독기 (DataReader):** 특정 토픽으로부터 데이터를 읽는(구독하는) 실제 행위자. 수신된 데이터는 `DataReader`의 히스토리 캐시에 저장된다.

`Fast-DDS-spy`는 이러한 모든 엔티티의 생성, 소멸, 연결 상태를 실시간으로 추적하고 사용자에게 보여주는 기능을 제공한다.



eProsima Fast DDS는 OMG DDS 표준을 충실히 구현하면서도 개발자의 편의성과 고성능을 위한 다양한 독자적인 특징을 갖추고 있다. 그 아키텍처의 핵심은 다음과 같다 3:

- **이중 API 계층 (Dual API Layers):** Fast DDS는 두 가지 수준의 API를 제공한다. 하나는 사용 편의성에 초점을 맞춘 고수준의 DDS 표준 API이고, 다른 하나는 RTPS 프로토콜의 내부 동작에 더 세밀하게 접근할 수 있는 저수준의 RTPS API이다.3 이를 통해 개발자는 애플리케이션의 요구사항에 따라 추상화 수준을 선택할 수 있다.
- **다양한 전송 계층 (Transport Layers):** 기본적인 UDP(v4/v6) 외에도 TCP(v4/v6)와 공유 메모리(Shared Memory, SHM) 전송을 지원한다.3 TCP는 신뢰성이 보장되는 연결 지향적 통신에, SHM은 동일한 호스트 내의 프로세스 간 통신에서 극적인 성능 향상을 위해 사용될 수 있다.
- **보안 플러그인 (Security Plugins):** DDS Security 표준에 따라 원격 참여자 인증, 엔티티 접근 제어, 데이터 암호화의 세 가지 수준에서 플러그인 방식의 보안을 제공한다.3 이를 통해 민감한 데이터를 다루는 시스템에서도 안전한 통신을 보장할 수 있다.
- **디스커버리 메커니즘 (Discovery Mechanisms):** 기본적으로는 분산된 동적 디스커버리 방식을 사용하지만, 대규모 네트워크나 멀티캐스트가 제한된 환경(예: WiFi)을 위해 중앙 서버를 통한 클라이언트-서버 디스커버리 방식도 지원한다.3
- **서비스 품질 (QoS) 정책:** 신뢰성(Reliability), 내구성(Durability), 데드라인(Deadline) 등 20가지가 넘는 정교한 QoS 정책을 제공하여 데이터 흐름을 세밀하게 제어할 수 있다.4

이러한 Fast DDS의 풍부한 기능들은 `Fast-DDS-spy`가 모니터링하고 진단해야 할 대상이 되며, `Fast-DDS-spy`의 고급 설정 기능들은 이러한 Fast DDS의 특징들을 활용하여 더 정밀한 분석을 가능하게 한다.


`Fast-DDS-spy`는 단독으로 존재하는 도구가 아니라, eProsima가 제공하는 포괄적인 DDS 개발 및 운영 툴킷의 일부이다.13 이 에코시스템 내에서 각 도구는 상호 보완적인 역할을 수행하며, 개발 수명주기의 다양한 단계를 지원한다.

- **`Fast DDS Monitor`:** `Fast-DDS-spy`가 텍스트 기반의 정밀 검사를 제공한다면, `Fast DDS Monitor`는 그래픽 사용자 인터페이스(GUI)를 통해 네트워크 성능을 시각적으로 모니터링하는 데 특화되어 있다.13 시간에 따른 지연 시간(latency), 처리량(throughput), 패킷 손실률과 같은 핵심 성능 지표(KPI)를 그래프로 보여주어 성능 병목 현상이나 QoS 비호환성 문제를 직관적으로 파악하는 데 유용하다.16
- **`DDS Router`:** 서로 다른 LAN이나 DDS 도메인에 속한 네트워크를 연결해주는 브리지 역할을 한다.13 예를 들어, 지리적으로 떨어진 두 개의 로봇 시스템을 WAN을 통해 연결하거나, 서로 다른 도메인 ID를 사용하는 시스템 간에 데이터를 교환해야 할 때 사용된다.
- **`DDS Record & Replay`:** DDS 네트워크에서 교환되는 데이터를 MCAP 형식의 데이터베이스에 저장하고, 필요할 때 정확하게 재생하는 기능을 제공한다.13 이를 통해 특정 시점의 네트워크 상황을 그대로 재현하여 오프라인에서 정밀하게 분석하거나, 시뮬레이션 입력 데이터로 활용할 수 있다.

이처럼 다양한 도구들이 존재한다는 사실은 eProsima가 단순히 DDS 라이브러리 하나를 제공하는 것을 넘어, 복잡한 DDS 애플리케이션의 설계, 개발, 디버깅, 배포, 운영 및 분석에 이르는 전 과정을 지원하는 성숙한 개발 생태계를 구축했음을 시사한다. 개발자는 문제의 성격에 따라 적절한 도구를 선택하여 활용할 수 있다. 예를 들어, `Fast DDS Monitor`로 성능 저하를 감지한 후, `Fast-DDS-spy`를 사용하여 문제가 되는 토픽의 원시 데이터를 상세히 검사하고, `DDS Record & Replay`로 해당 시나리오를 기록하여 심층 분석하는 워크플로우를 구성할 수 있다. 이 생태계 안에서 `Fast-DDS-spy`는 개발자의 손에 들린 '고정밀 디지털 멀티미터'와 같은 역할을 수행한다.



`Fast-DDS-spy`의 핵심 목적은 DDS 네트워크의 내부를 "스니핑(sniffing)"하거나 검사(introspect)하여, 그 상태를 사람이 읽기 쉬운(human-readable) 형식으로 제공하는 것이다.8 이 도구는 대화형 CLI를 통해 실행되며, 사용자의 텍스트 기반 명령어에 응답하여 네트워크에서 발견된 엔티티와 데이터에 대한 정보를 표준 출력(stdout)으로 보여준다.9

주요 기능은 다음과 같다 18:

- 네트워크에 참여 중인 `DomainParticipant` 목록 및 상세 정보 조회
- 발견된 `DataWriter` 및 `DataReader` 목록 및 상세 정보 조회
- 존재하는 `Topic` 목록, 데이터 타입, QoS, 데이터 전송률 등 상세 정보 조회
- 특정 `Topic`을 통해 전송되는 사용자 데이터를 실시간으로 모니터링 (echo 기능)

이 모든 정보는 기본적으로 YAML 형식으로 출력되어 가독성이 높고, 스크립트를 통해 파싱하여 자동화된 테스트나 분석에 활용하기 용이하다.


`Fast-DDS-spy`가 가진 가장 강력하고 편리한 특징 중 하나는, 사용자가 분석하고자 하는 토픽의 데이터 타입 정의(IDL 파일)를 미리 제공할 필요 없이 자동으로 데이터를 해석하고 보여준다는 점이다.8 이것이 가능한 이유는 

`Fast-DDS-spy`가 eProsima Fast DDS의 `DynamicTypes` 기능을 적극적으로 활용하기 때문이다.17

전통적인 정적(static) DDS 개발 방식에서는, 개발자가 먼저 IDL(Interface Definition Language) 파일을 사용하여 데이터 타입을 정의해야 한다. 그 후, `Fast-DDS-Gen`과 같은 코드 생성 도구를 사용하여 해당 데이터 타입을 처리할 수 있는 C++ 소스 코드를 생성하고, 이를 애플리케이션에 포함시켜 컴파일해야 한다.3 이 방식은 컴파일 시점에 모든 타입 정보가 결정되므로 실행 성능이 매우 뛰어나지만, 새로운 데이터 타입을 다루거나 기존 타입을 변경할 때마다 코드 생성 및 재컴파일 과정이 필요하여 유연성이 떨어진다.

반면, `DynamicTypes`는 런타임에 타입 정보를 정의하고 해석할 수 있는 메커니즘을 제공한다.20 DDS-XTypes 표준의 일부인 이 기능 덕분에, 애플리케이션은 디스커버리 과정에서 다른 참여자로부터 타입 정보를 전파받거나 XML 파일로부터 타입 정의를 로드하여, 컴파일 시점에는 알지 못했던 데이터 타입의 데이터를 동적으로 생성하고 해석할 수 있다.22

`Fast-DDS-spy`는 바로 이 원리를 이용한다. `Fast-DDS-spy`는 네트워크에 참여하여 다른 `DataWriter`들이 발행하는 디스커버리 정보를 수신한다. 이 정보에는 토픽 이름과 QoS뿐만 아니라, 해당 토픽의 데이터 타입 정보(TypeObject)도 포함될 수 있다. `Fast-DDS-spy`는 이 `TypeObject`를 수신하여 내부적으로 데이터 타입을 동적으로 재구성하고, 이를 바탕으로 해당 토픽의 데이터를 올바르게 역직렬화(deserialize)하여 사용자에게 보여주는 것이다.

이러한 `DynamicTypes`의 활용은 `Fast-DDS-spy`의 설계 철학을 명확히 보여준다. 이 도구는 정적 타입 기반의 커스텀 구독자를 만들었을 때 얻을 수 있는 최고의 성능을 일부 희생하는 대신, 어떤 DDS 네트워크에든 즉시 연결하여 별도의 사전 설정 없이 바로 내부를 들여다볼 수 있는 압도적인 사용 편의성과 유연성을 선택했다. 이는 `Fast-DDS-spy`를 개발 및 디버깅 단계에서 매우 강력한 도구로 만들어주는 핵심적인 장점이다.


eProsima의 툴링 에코시스템과 경쟁사의 솔루션을 비교하면, 각 도구의 장단점과 적합한 사용 사례가 명확해진다.

- **`Fast-DDS-spy` (CLI 내부 검사 도구):**
  - **강점:** 터미널 환경에서의 신속한 디버깅, 스크립트를 통한 자동화, 원시 데이터 값의 정밀 확인에 최적화되어 있다. 텍스트 기반의 세밀한 제어와 검사가 필요할 때 가장 강력하다.8
  - **약점:** 시계열 데이터의 추세 분석이나 전체 네트워크 토폴로지의 직관적인 파악에는 한계가 있다.
  - **주요 사용 사례:** 특정 토픽의 데이터 페이로드가 올바른지 확인, 엔티티의 QoS 설정값 검사, 스크립트 기반의 통합 테스트에서 특정 DDS 이벤트 확인.
- **`Fast DDS Monitor` (GUI 모니터링 도구):**
  - **강점:** 네트워크 성능에 대한 거시적이고 시각적인 분석에 탁월하다. 지연 시간, 처리량, 패킷 손실과 같은 통계 정보를 그래프로 제공하여 성능 병목 지점을 쉽게 식별할 수 있다.13 엔티티 간의 연결 관계를 그래픽으로 보여주어 복잡한 네트워크 토폴로지를 이해하는 데 도움이 된다.16
  - **약점:** 개별 데이터 샘플의 상세 내용을 파고들거나, 텍스트 기반의 정밀한 필터링 및 자동화에는 `Fast-DDS-spy`보다 불편하다.
  - **주요 사용 사례:** 시스템 부하 테스트 중 성능 변화 추적, QoS 비호환으로 인한 연결 실패 원인 분석, 전체 시스템의 데이터 흐름 시각화.
- **`RTI DDS Spy` (상용 대체재):**
  - **유사점:** `Fast-DDS-spy`와 마찬가지로 DDS 네트워크를 검사하는 CLI 도구이다.24 도메인 ID 지정, 토픽 필터링, 데이터 출력 등 기본적인 기능은 유사하다.
  - **차이점:**
    - **라이선스:** `Fast-DDS-spy`는 오픈소스(Apache 2.0)인 반면, `RTI DDS Spy`는 상용 제품인 RTI Connext DDS 제품군의 일부이다.4
    - **기능 범위:** RTI는 자사 솔루션이 Presentation QoS, Coherent Sets, Custom Content Filters 등 더 광범위한 DDS 표준 기능과 독자적인 확장 기능을 지원한다고 주장한다.25
    - **에코시스템:** `Fast-DDS-spy`는 ROS 2와의 긴밀한 통합과 오픈소스 커뮤니티를 강점으로 하는 반면, `RTI DDS Spy`는 엔터프라이즈급 기술 지원과 다양한 산업 분야에서의 오랜 적용 사례를 강점으로 한다.
  - **선택 기준:** 개발 환경이 eProsima Fast DDS 또는 ROS 2 중심이라면 `Fast-DDS-spy`가 자연스러운 선택이다. 반면, 이미 RTI Connext DDS를 사용 중이거나 RTI가 제공하는 특정 고급 기능 또는 기술 지원이 필요한 경우 `RTI DDS Spy`가 적합할 수 있다.



이 장에서는 주요 운영체제별로 `Fast-DDS-spy`를 설치하고 실행 환경을 설정하는 방법을 상세히 안내한다. 각 플랫폼의 특성에 맞는 최적의 설치 방법을 제시하여 사용자가 겪을 수 있는 잠재적인 문제를 최소화하는 것을 목표로 한다.


`Fast-DDS-spy`를 소스에서 빌드하기 위해서는 공통적으로 다음과 같은 개발 도구들이 시스템에 설치되어 있어야 한다 26:

- **컴파일러:** `g++` (Linux/macOS) 또는 Visual Studio C++ 컴파일러 (Windows)
- **빌드 시스템:** `CMake` (버전 3.16 이상 권장)
- **버전 관리:** `git`
- **Python 및 패키지 관리자:** `python3`, `pip3`
- **네트워크 도구:** `wget` 또는 `curl`
- **ROS 2 빌드 도구 (권장):** `colcon`, `vcstool` (Colcon 설치 시 필요)

다음 표는 각 운영체제별 핵심 요구사항을 요약한 것이다.

| 운영체제           | 필수 도구                                                    | 패키지 관리자       | 주요 설치 명령어 예시                                       |
| ------------------ | ------------------------------------------------------------ | ------------------- | ----------------------------------------------------------- |
| **Linux (Ubuntu)** | `build-essential`, `cmake`, `git`, `python3-pip`             | `apt`               | `sudo apt install build-essential cmake git python3-pip`    |
| **Windows**        | Visual Studio (C++ Desktop Development 워크로드 포함), CMake, Git, Python | `Chocolatey` (권장) | `choco install visualstudio2019buildtools cmake git python` |
| **macOS**          | Xcode Command Line Tools, CMake, Git, Python                 | `Homebrew`          | `xcode-select --install`, `brew install cmake git python`   |


Linux 환경, 특히 Ubuntu 배포판에서는 `colcon` 빌드 도구를 사용하는 것이 가장 권장되는 방법이다. 이 방법은 `Fast-DDS-spy`뿐만 아니라 필요한 모든 eProsima 종속성 라이브러리를 자동으로 다운로드하고 빌드해준다.

1. 필수 패키지 설치:

   먼저 빌드에 필요한 라이브러리와 도구들을 apt와 pip를 통해 설치한다.26

   ```Bash
   # 시스템 라이브러리 설치
   sudo apt update
   sudo apt install -y cmake g++ pip wget git libasio-dev libtinyxml2-dev libssl-dev libyaml-cpp-dev
   
   # Python 기반 빌드 도구 설치
   pip3 install -U colcon-common-extensions vcstool
   ```
   
2. 소스 코드 다운로드:

   vcstool을 사용하여 .repos 파일에 명시된 모든 저장소를 다운로드한다.26

   ```Bash
   mkdir -p ~/Fast-DDS-spy-ws/src
   cd ~/Fast-DDS-spy-ws
   wget https://raw.githubusercontent.com/eProsima/Fast-DDS-spy/main/fastddsspy.repos
   vcs import src < fastddsspy.repos
   ```
   
3. Colcon으로 빌드:

   colcon을 사용하여 워크스페이스를 빌드한다. --packages-up-to 옵션은 fastddsspy_tool과 그 종속성까지만 빌드하도록 하여 시간을 절약해준다.26

   ```Bash
   colcon build --packages-up-to fastddsspy_tool
   ```
   
4. 환경 설정 및 실행:

   빌드가 완료되면, 생성된 setup.bash 파일을 source하여 환경 변수를 설정한 후 실행 파일을 호출한다.26

   ```Bash
   source ~/Fast-DDS-spy-ws/install/setup.bash
   fastddsspy_tool
   ```


현재 `Fast-DDS-spy`는 Windows용 바이너리 인스톨러를 제공하지 않으므로, 소스에서 직접 빌드해야 한다.9 Windows에서의 빌드는 Linux보다 복잡하며, Visual Studio와 Chocolatey 패키지 관리자를 사용하는 것이 권장된다.

1. **사전 요구사항 설치:**

   - **Visual Studio:** "C++를 사용한 데스크톱 개발" 워크로드를 포함하여 설치한다.

   - **Chocolatey:** Windows용 패키지 관리자로, `choco.exe`를 사용하여 나머지 도구들을 쉽게 설치할 수 있다.

   - **기타 도구:** 관리자 권한으로 실행된 PowerShell에서 다음 명령어를 사용하여 CMake, Git, Python, Asio, TinyXML2, OpenSSL, yaml-cpp를 설치한다.

     ```PowerShell
     choco install -y cmake git python asio tinyxml2 openssl libyaml-cpp
     ```
   
2. **Colcon 및 vcstool 설치:**

   ```PowerShell
   pip3 install -U colcon-common-extensions vcstool
   ```
   
3. 소스 코드 다운로드 및 빌드:

   Linux와 동일하게 작업 디렉토리를 만들고 .repos 파일을 통해 소스 코드를 다운로드한 후, colcon build 명령어를 실행한다. 이때 Visual Studio 개발자 명령 프롬프트 환경에서 작업을 수행해야 한다.

자세한 단계는 공식 문서의 [Windows 설치 가이드](https://fast-dds-spy.readthedocs.io/en/latest/rst/developer_manual/installation/sources/windows.html)를 참조하는 것이 좋다.29


macOS용 `Fast-DDS-spy` 설치 가이드는 별도로 제공되지 않지만, `Fast DDS` 라이브러리의 설치 절차를 응용하여 빌드할 수 있다.30 Homebrew 패키지 관리자 사용이 필수적이다.

1. 사전 요구사항 설치:

   Xcode 명령줄 도구와 Homebrew를 먼저 설치한 후, 나머지 의존성을 설치한다.30

   ```Bash
   # Xcode 명령줄 도구 설치
   xcode-select --install
   
   # Homebrew 및 필수 패키지 설치
   brew install cmake python3 wget asio tinyxml2 openssl@1.1 yaml-cpp
   
   # Python 기반 빌드 도구 설치
   pip3 install -U colcon-common-extensions vcstool
   ```
   
2. 소스 코드 다운로드 및 빌드:

   Linux와 동일한 절차로 소스 코드를 다운로드하고 colcon build를 실행한다. OpenSSL 라이브러리 경로를 CMake 인자로 명시적으로 전달해야 할 수 있다.30

   ```Bash
   #... 소스 코드 다운로드...
   colcon build --packages-up-to fastddsspy_tool --cmake-args -DOPENSSL_ROOT_DIR=$(brew --prefix openssl@1.1)
   ```


플랫폼별 의존성 문제를 피하고 가장 빠르고 안정적으로 `Fast-DDS-spy`를 사용하고 싶다면 Docker 이미지를 사용하는 것이 최선의 방법이다.18

1. Docker 이미지 다운로드 및 로드:

   eProsima 공식 다운로드 페이지에서 ubuntu-fastddsspy:<version>.tar 파일을 다운로드한 후, docker load 명령어로 이미지를 로드한다.32

   ```Bash
   docker load -i ubuntu-fastddsspy:v1.2.0.tar
   ```
   
2. 설정 파일 준비 (선택 사항):

   호스트 머신에 FASTDDSSPY_CONFIGURATION.yaml 파일을 생성하여 Fast-DDS-spy의 동작을 상세히 설정할 수 있다.

3. Docker 컨테이너 실행:

   docker run 명령어를 사용하여 컨테이너를 실행한다. 이때 --net=host 옵션은 컨테이너가 호스트의 네트워크 스택을 직접 사용하게 하여 DDS 디스커버리가 원활하게 이루어지도록 하는 데 중요하다. -v 옵션을 사용하여 호스트의 설정 파일을 컨테이너 내부로 마운트할 수 있다.32

   ```Bash
   # 설정 파일 없이 실행
   docker run -it --net=host ubuntu-fastddsspy:v1.2.0
   
   # 설정 파일과 함께 실행
   docker run -it --net=host \
       -v /path/to/your/FASTDDSSPY_CONFIGURATION.yaml:/root/FASTDDSSPY_CONFIGURATION.yaml \
       ubuntu-fastddsspy:v1.2.0
   ```


Ubuntu 및 기타 지원되는 Linux 배포판에서는 Snap Store를 통해 `vulcanexus-fastddsspy` 패키지를 설치할 수 있다. 이는 매우 간편한 방법이지만, 최신 버전이 아닐 수 있으며 `edge` 채널을 사용해야 할 수 있다.33

```Bash
sudo snap install vulcanexus-fastddsspy --edge
```


`Fast-DDS-spy`는 대화형 CLI를 통해 모든 기능을 제어한다. 이 장에서는 애플리케이션 실행 시의 인자와 대화형 모드에서의 명령어들을 상세히 다룬다.


`fastddsspy_tool` 실행 파일은 여러 명령줄 인자를 받아 동작을 제어할 수 있다. `fastddsspy_tool --help`를 실행하면 사용 가능한 모든 옵션을 볼 수 있다.26

```
Usage: Fast DDS Spy
Start an interactive CLI to introspect a DDS network.

General options: Application help and information.
  -h --help                 Print this help message.
  -v --version              Print version, branch and commit hash.

Application parameters
  -c --config-path          Path to the Configuration File (yaml format).
  -r --reload-time          Time period in seconds to reload configuration file. Value 0 does not reload file..
     --domain               Set the domain (0-255) to spy on..

Debug parameters
  -d --debug                Set log verbosity to Info (Using this option with --log-filter and/or --log-verbosity will head to undefined behaviour).
     --log-filter           Set a Regex Filter to filter by category the info and warning log entries..
     --log-verbosity        Set a Log Verbosity Level higher or equal the one given. (Values accepted: "info","warning","error" no Case Sensitive).
```

36

- `-c, --config-path`: 사용할 YAML 설정 파일의 경로를 지정한다. 지정하지 않으면 실행 디렉토리의 `FASTDDSSPY_CONFIGURATION.yaml` 파일을 찾는다.36

- `--domain`: 감시할 DDS 도메인 ID를 지정한다. 설정 파일의 값보다 우선한다.36

- `-d, --debug`: 로그 상세 수준을 `Info`로 설정하여 디버깅에 유용한 상세 정보를 출력한다.36

- **원샷(One-shot) 모드:** 인자 뒤에 대화형 명령어를 직접 입력하면, `Fast-DDS-spy`는 해당 명령어만 실행하고 결과를 출력한 뒤 즉시 종료된다. 이는 스크립트에서 활용하기 좋다.36

  ```Bash
  fastddsspy_tool --domain 5 topics v
  ```


`Fast-DDS-spy`를 인자 없이 실행하면 대화형 프롬프트가 나타난다. 각 명령어는 첫 글자로 축약하여 사용할 수 있다 (예: `participants` 대신 `p`). 다음 표는 주요 명령어들을 요약한 것이다.18

| 명령어         | 축약형 | 설명                                                         | 인자                | 예시                  |
| -------------- | ------ | ------------------------------------------------------------ | ------------------- | --------------------- |
| `help`         | `h`    | 사용 가능한 모든 명령어와 설명을 표시합니다.                 | 없음                | `help`                |
| `version`      | 없음   | `Fast-DDS-spy`의 버전 정보를 표시합니다.                     | 없음                | `version`             |
| `quit`         | `q`    | 대화형 CLI를 종료합니다.                                     | 없음                | `quit`                |
| `participants` | `p`    | 발견된 모든 `DomainParticipant`의 이름과 GUID를 표시합니다.  | `verbose`, `<Guid>` | `p verbose`           |
| `writers`      | `w`    | 발견된 모든 `DataWriter`의 소속 Participant와 GUID를 표시합니다. | `verbose`, `<Guid>` | `w <Guid>`            |
| `readers`      | `r`    | 발견된 모든 `DataReader`의 소속 Participant와 GUID를 표시합니다. | `verbose`, `<Guid>` | `r`                   |
| `topics`       | `t`    | 발견된 모든 토픽의 정보를 간결하게 표시합니다.               | `v`, `vv`, `<name>` | `t vv`, `t "rt/*"`    |
| `echo`         | `s`    | 특정 토픽의 데이터를 실시간으로 출력합니다.                  | `<name>`, `verbose` | `echo Square verbose` |

`verbose` 옵션은 각 엔티티의 QoS, 주소 등 훨씬 상세한 정보를 YAML 형식으로 출력해준다. `<Guid>` 인자를 사용하면 특정 엔티티 하나에 대한 정보만 필터링하여 볼 수 있다.


`echo` (또는 `show`, `print`) 명령어는 `Fast-DDS-spy`의 가장 핵심적인 기능으로, 특정 토픽의 데이터를 실시간으로 들여다볼 수 있게 해준다.23

- **단일 토픽 감시:** `echo <TopicName>` 형식으로 사용한다.

  ```
  >> echo Square
  ---
  color: BLUE
  x: 158
  y: 70
  shapesize: 20
  ---
  color: BLUE
  x: 159
  y: 71
  shapesize: 20
  ---
  ```

  23

- **와일드카드(`\*`) 사용:** 토픽 이름에 와일드카드를 사용하여 여러 토픽을 한 번에 감시할 수 있다. 이는 ROS 2에서 네임스페이스 아래의 여러 토픽을 확인할 때 특히 유용하다.23

  ```
  >> echo /robot_1/sensor_*
  ```

- **`verbose` 옵션:** `verbose` 인자를 추가하면 데이터 내용뿐만 아니라 메타데이터도 함께 출력된다. 이는 데이터가 어느 `DataWriter`로부터, 언제 생성되었는지 추적하는 데 매우 유용하다.23

  ```
  >> echo Circle verbose
  topic: Circle
  data: 01.0f.44.59.da.57.de.ec.00.00.00.00|0.0.6.2
  timestamp: 2023/03/27 09:35:23
  data:
  ---
  color: GREEN
  x: 72
  y: 125
  shapesize: 30
  ---
  ```

  23

`echo` 명령어는 Enter 키를 누를 때까지 계속해서 새로운 데이터를 출력한다. 이를 통해 데이터의 변화 양상이나 전송 주기 등을 실시간으로 관찰할 수 있다.


`Fast-DDS-spy`의 진정한 강력함은 YAML 설정 파일을 통해 내부 동작을 세밀하게 제어할 때 발휘된다. 이 장에서는 주요 설정 항목들을 상세히 설명한다.


설정 파일은 크게 `dds`와 `specs`라는 두 개의 최상위 키로 구성될 수 있다.38

- **`dds`:** `Fast-DDS-spy` 내부의 DDS 엔티티(Participant, Reader)와 관련된 설정을 담는다.
- **`specs`:** `Fast-DDS-spy` 애플리케이션 자체의 동작(스레드 수, 로깅 등)에 대한 설정을 담는다.

```YAML
dds:
  # DDS 관련 설정
  domain: 0
  allowlist:
    #...
  topics:
    #...

specs:
  # 애플리케이션 동작 관련 설정
  threads: 4
  logging:
    #...
```


대규모 네트워크에서는 수많은 토픽이 존재할 수 있다. `allowlist`와 `blocklist`를 사용하면 `Fast-DDS-spy`가 처리할 토픽을 효과적으로 필터링하여 불필요한 부하를 줄이고 분석의 초점을 명확히 할 수 있다.38

- **규칙:**
  1. 설정이 없으면 모든 토픽을 처리한다.
  2. `allowlist`만 있으면, 목록에 있는 토픽만 처리한다.
  3. `blocklist`만 있으면, 목록에 없는 모든 토픽을 처리한다.
  4. 둘 다 있으면, `allowlist`에 포함되면서 `blocklist`에는 없는 토픽만 처리한다 (`blocklist` 우선).
- **필터링 기준:** 토픽의 `name`과 `type`을 기준으로 필터링하며, 와일드카드(`*`) 사용이 가능하다. 와일드카드를 포함하는 값은 따옴표로 묶어야 한다.39

```YAML
dds:
  allowlist:
    - name: "rt/chatter"
      type: "std_msgs::msg::dds_::String_"
    - name: "rt/imu/*"
      type: "*"
  blocklist:
    - name: "*"
      type: "sensor_msgs::msg::dds_::PointCloud2_"
```

위 예시는 `/chatter` 토픽과 `/imu/`로 시작하는 모든 토픽은 허용하되, 타입이 `PointCloud2`인 모든 토픽은 차단하도록 설정한다.


DDS에서 `DataWriter`와 `DataReader`가 성공적으로 매칭되려면 QoS 설정이 호환되어야 한다. `Fast-DDS-spy`는 기본적으로 가장 관대한 QoS로 `DataReader`를 생성하지만, `DataWriter`가 특정 QoS(예: `RELIABLE` 신뢰성)를 요구하는 경우, `Fast-DDS-spy`의 `DataReader`도 그에 맞게 QoS를 설정해주어야 데이터를 수신할 수 있다. `topics` 섹션에서 이를 수동으로 설정할 수 있다.38

| QoS 정책                          | YAML 태그       | 데이터 타입    | 설명                                                         |
| --------------------------------- | --------------- | -------------- | ------------------------------------------------------------ |
| **신뢰성 (Reliability)**          | `reliability`   | `bool`         | `true`로 설정 시 `RELIABLE`, `false`는 `BEST_EFFORT`.        |
| **내구성 (Durability)**           | `durability`    | `bool`         | `true`로 설정 시 `TRANSIENT_LOCAL`, `false`는 `VOLATILE`.    |
| **소유권 (Ownership)**            | `ownership`     | `bool`         | `true`로 설정 시 `EXCLUSIVE_OWNERSHIP_QOS`, `false`는 `SHARED_OWNERSHIP_QOS`. |
| **파티션 (Partitions)**           | `partitions`    | `bool`         | `true`로 설정 시 파티션 QoS를 활성화. (구체적인 파티션 이름은 별도 설정 필요) |
| **키 존재 여부 (Keyed)**          | `keyed`         | `bool`         | `true`로 설정 시 토픽에 키가 있음을 명시.                    |
| **히스토리 깊이 (History Depth)** | `history-depth` | `unsigned int` | Reader가 보관할 최대 샘플 수.                                |

```YAML
dds:
  topics:
    - name: "rt/important_data"
      type: "*"
      qos:
        reliability: true
        durability: true
        history-depth: 10
```

위 설정은 `rt/important_data` 토픽에 대해 신뢰성 있고, `TRANSIENT_LOCAL` 내구성을 가지며, 히스토리 깊이가 10인 `DataReader`를 생성하도록 `Fast-DDS-spy`에 지시한다.


고속으로 대량의 데이터가 오가는 네트워크에서 `Fast-DDS-spy` 자체가 병목이 되거나 과도한 자원을 소모하는 것을 방지하기 위한 설정들이다.38

- `max-rx-rate`: 초당 처리할 최대 샘플 수를 Hz 단위로 제한한다. `10`으로 설정하면 0.1초 이내에 들어오는 추가 샘플은 무시한다. 기본값 `0`은 무제한이다.
- `downsampling`: `n`으로 설정하면 `n`개의 샘플 중 1개만 처리하여 샘플링 속도를 줄인다. 기본값 `1`은 모든 샘플을 처리한다.
- `history-depth`: 내부 `DataReader`의 히스토리 캐시 크기를 조절한다. 샘플 크기가 크거나 토픽 수가 많아 메모리 소모가 우려될 때 이 값을 줄일 수 있다. 기본값은 `5000`이다.


- 네트워크 설정 37:

  - `ignore-participant-flags`: 특정 범위의 참여자로부터 오는 디스커버리 트래픽을 무시한다. (`filter_different_host`, `filter_different_process`)
  - `interface-whitelist`: 사용할 네트워크 인터페이스를 특정 IP 주소나 이름으로 제한한다.

- 로깅 설정 38:

  

  specs 섹션 아래 logging 태그를 사용하여 Fast-DDS-spy 자체의 로그 출력 수준을 제어할 수 있다.

  - `verbosity`: 로그의 상세 수준을 `info`, `warning`, `error` 중 하나로 설정한다.
  - `filter`: 정규 표현식을 사용하여 특정 카테고리나 메시지 내용의 로그만 출력하도록 필터링한다.

```YAML
specs:
  logging:
    verbosity: info
    filter:
      error: "DDSPIPE|FASTDDSSPY"
      warning: "DDSPIPE|FASTDDSSPY"
      info: "FASTDDSSPY"
```

이 설정은 모든 `error`와 `warning` 로그 중 `DDSPIPE` 또는 `FASTDDSSPY` 카테고리의 것만, 그리고 `info` 로그는 `FASTDDSSPY` 카테고리의 것만 출력하도록 한다. 이는 복잡한 문제 상황에서 원하는 정보에 집중하는 데 큰 도움이 된다.




대규모 시스템이나 WiFi와 같이 멀티캐스트가 불안정한 네트워크 환경에서는 기본 디스커버리 프로토콜(Simple Discovery Protocol) 대신 중앙 집중형 디스커버리 서버(Discovery Server)를 사용하는 것이 효율적이다.12

`Fast-DDS-spy`도 이러한 환경에 참여하여 네트워크를 감시할 수 있다. 이를 위해서는 `Fast-DDS-spy`의 내부 `DomainParticipant`를 디스커버리 서버의 클라이언트로 설정해야 한다.

이 설정은 `Fast-DDS-spy`의 YAML 파일에서는 직접 지원하지 않는다. 대신, `Fast-DDS-spy`가 기반으로 하는 Fast DDS 라이브러리의 표준 설정 방식을 따라야 한다. 즉, Fast DDS XML 프로파일을 통해 클라이언트 설정을 정의하고, 환경 변수를 통해 `Fast-DDS-spy`가 이 프로파일을 로드하도록 해야 한다.41

1. 클라이언트용 XML 프로파일 생성:

   다음과 같이 discoveryProtocol을 CLIENT로 설정하고, 연결할 서버의 주소 목록을 discoveryServersList에 명시한 XML 파일을 작성한다 (discovery_client.xml로 저장).

   ```XML
<?xml version="1.0" encoding="UTF-8"?>
   <profiles xmlns="http://www.eprosima.com">
       <participant profile_name="discovery_client_profile" is_default_profile="true">
           <rtps>
               <builtin>
                   <discovery_config>
                       <discoveryProtocol>CLIENT</discoveryProtocol>
                       <discoveryServersList>
                           <locator>
                               <udpv4>
                                   <address>127.0.0.1</address>
                                   <port>11811</port>
                               </udpv4>
                           </locator>
                       </discoveryServersList>
                   </discovery_config>
               </builtin>
           </rtps>
       </participant>
   </profiles>
   ```
   
   44

2. 환경 변수 설정:

   fastddsspy_tool을 실행하기 전에, FASTRTPS_DEFAULT_PROFILES_FILE 환경 변수를 위에서 생성한 XML 파일의 경로로 설정한다.45

   ```Bash
   export FASTRTPS_DEFAULT_PROFILES_FILE=/path/to/your/discovery_client.xml
   ```
   
3. Fast-DDS-spy 실행:

   이제 fastddsspy_tool을 실행하면, 내부 DomainParticipant는 기본 멀티캐스트 디스커버리 대신 지정된 디스커버리 서버에 연결하여 다른 노드 정보를 얻게 된다. 이를 통해 서버 기반의 복잡한 네트워크 토폴로지에서도 Fast-DDS-spy를 완벽하게 활용할 수 있다.


DDS 파티션(Partition)은 동일한 토픽을 사용하더라도 논리적으로 통신 그룹을 분리하는 강력한 기능이다. 발행자와 구독자는 서로 일치하는 파티션 이름을 하나 이상 가져야만 통신할 수 있다. 파티션 이름에는 와일드카드(`*`)를 사용할 수 있어 유연한 매칭이 가능하다.46

파티션 설정 오류는 "데이터가 전송되지 않지만 아무런 에러도 발생하지 않는" 대표적인 '조용한 실패'의 원인이다. `Fast-DDS-spy`는 이러한 문제를 진단하는 데 매우 유용하다.

- **진단 방법:**
  1. `Fast-DDS-spy`를 실행하고 `participants verbose` 명령어를 입력한다.
  2. 출력된 YAML 정보에서 각 `DataWriter`와 `DataReader`의 `qos` 섹션 아래에 있는 `partition` 필드를 확인한다.
  3. 통신해야 할 두 엔티티의 파티션 목록이 서로 매칭되는지(적어도 하나의 공통된 파티션 이름을 갖는지) 검사한다.
  4. 만약 파티션이 일치하지 않는다면, 이것이 통신 실패의 원인일 가능성이 매우 높다.

`DataWriterListener`의 `on_publication_matched` 콜백은 매칭되거나 매칭이 해제될 때 호출되므로, 애플리케이션 코드 수준에서도 파티션 문제를 감지할 수 있다.47


DDS Security는 분산 시스템의 통신을 보호하기 위한 포괄적인 프레임워크를 제공한다. Fast DDS는 이 표준의 핵심적인 세 가지 플러그인을 구현한다 11:

- **인증 (Authentication):** `DDS:Auth:PKI-DH`. PKI(공개 키 기반 구조)를 사용하여 도메인에 참여하는 모든 `DomainParticipant`가 신뢰할 수 있는지 확인한다.
- **접근 제어 (Access Control):** `DDS:Access:Permissions`. 인증된 참여자가 특정 토픽에 대해 발행/구독할 수 있는 권한이 있는지 여부를 제어한다.
- **암호화 (Cryptography):** `DDS:Crypto:AES-GCM-GMAC`. 실제 데이터 페이로드를 암호화하여 기밀성을 보장한다.


`Fast-DDS-spy`가 보안이 적용된 DDS 도메인을 감시하기 위해서는, `Fast-DDS-spy` 자체가 합법적이고 인가된 `DomainParticipant`로서 도메인에 참여해야 한다. 이는 `Fast-DDS-spy`가 단순히 네트워크를 '엿보는' 것이 아니라, 보안 핸드셰이크 과정에 정식으로 참여해야 함을 의미한다.

이 설정 과정은 디스커버리 서버 설정과 유사하게, `Fast-DDS-spy`의 YAML 파일이 아닌 Fast DDS의 표준 XML 프로파일을 통해 이루어진다. 이는 매우 중요한 점으로, `Fast-DDS-spy`의 자체 설정만으로는 보안 도메인에 접근할 수 없다는 것을 명확히 이해해야 한다.

1. 보안 아티팩트(Artifacts) 준비:

   보안 도메인을 구성하는 데 사용된 모든 보안 관련 파일이 필요하다.11

   - 인증 기관(CA) 인증서 (`ca.pem`)
   - `Fast-DDS-spy`를 위한 신원 인증서 (`spy_cert.pem`)
   - `Fast-DDS-spy`를 위한 개인 키 (`spy_key.pem`)
   - 도메인 거버넌스 파일 (`governance.xml.p7s`, 서명된 파일)
   - `Fast-DDS-spy`를 위한 권한 파일 (`permissions.xml.p7s`, 서명된 파일)

2. 보안용 XML 프로파일 생성:

   Fast-DDS-spy가 사용할 DomainParticipant 프로파일을 XML로 정의한다. 이 프로파일은 PropertyPolicyQos를 사용하여 위에서 준비한 보안 아티팩트들의 경로를 지정해야 한다.11

   ```XML
   <profiles xmlns="http://www.eprosima.com">
       <participant profile_name="secure_spy_profile" is_default_profile="true">
           <rtps>
               <propertiesPolicy>
                   <properties>
                       <property>
                           <name>dds.sec.auth.plugin</name>
                           <value>builtin.PKI-DH</value>
                       </property>
                       <property>
                           <name>dds.sec.auth.builtin.PKI-DH.identity_ca</name>
                           <value>file:///path/to/your/ca.pem</value>
                       </property>
                       <property>
                           <name>dds.sec.auth.builtin.PKI-DH.identity_certificate</name>
                           <value>file:///path/to/your/spy_cert.pem</value>
                       </property>
                       <property>
                           <name>dds.sec.auth.builtin.PKI-DH.private_key</name>
                           <value>file:///path/to/your/spy_key.pem</value>
                       </property>
                       <property>
                           <name>dds.sec.access.plugin</name>
                           <value>builtin.Access-Permissions</value>
                       </property>
                       <property>
                           <name>dds.sec.access.builtin.Access-Permissions.permissions_ca</name>
                           <value>file:///path/to/your/ca.pem</value>
                       </property>
                       <property>
                           <name>dds.sec.access.builtin.Access-Permissions.governance</name>
                           <value>file:///path/to/your/governance.xml.p7s</value>
                       </property>
                       <property>
                           <name>dds.sec.access.builtin.Access-Permissions.permissions</name>
                           <value>file:///path/to/your/permissions.xml.p7s</value>
                       </property>
                       </properties>
               </propertiesPolicy>
           </rtps>
       </participant>
   </profiles>
   ```
   
3. 환경 변수 설정 및 실행:

   디스커버리 서버 사례와 동일하게, FASTRTPS_DEFAULT_PROFILES_FILE 환경 변수를 설정한 후 fastddsspy_tool을 실행한다.

   ```Bash
   export FASTRTPS_DEFAULT_PROFILES_FILE=/path/to/your/secure_spy.xml
   fastddsspy_tool
   ```

이 과정을 통해 `Fast-DDS-spy`는 보안 도메인에 성공적으로 참여하여 암호화된 토픽을 포함한 네트워크 상태를 감시할 수 있게 된다. 이는 `Fast-DDS-spy`가 보안을 우회하는 '마법 열쇠'가 아니라, 시스템의 보안 정책을 존중하며 동작하는 합법적인 진단 도구임을 보여주는 중요한 대목이다.



DDS에서 통신 실패의 가장 흔한 원인 중 하나는 `DataWriter`와 `DataReader` 간의 QoS 정책 불일치이다. DDS는 요청-제공(Request-Offered, RxO) 시맨틱스를 따르는데, 이는 `DataReader`가 요청하는 서비스 수준이 `DataWriter`가 제공하는 수준보다 높을 수 없음을 의미한다.49 예를 들어, 

`DataReader`가 신뢰성 있는 전송(`RELIABLE`)을 요청하는데 `DataWriter`는 최선형 전송(`BEST_EFFORT`)만 제공한다면, 두 엔티티는 호환되지 않는 것으로 간주되어 매칭되지 않고 데이터 교환도 일어나지 않는다.49

`Fast-DDS-spy`는 이러한 문제를 진단하는 데 매우 효과적이다.

- **진단 절차:**
  1. `topics vv` 명령어를 사용하여 매우 상세한 토픽 정보를 확인한다.
  2. 출력된 정보에서 통신이 안 되는 토픽을 찾고, 해당 토픽에 연결된 `datawriters`와 `datareaders`의 `qos` 설정을 비교한다.
  3. `reliability`, `durability`, `deadline`, `ownership` 등 주요 RxO 정책들이 호환되는지 확인한다. 예를 들어, `DataReader`의 `deadline.period`는 `DataWriter`의 `deadline.period`보다 크거나 같아야 한다.50
  4. 불일치가 발견되면, 해당 애플리케이션의 소스 코드나 XML 설정 파일에서 QoS를 수정하여 문제를 해결한다.

애플리케이션 수준에서는 `on_offered_incompatible_qos` 콜백을 리스너에 구현하여 프로그래밍적으로 QoS 불일치를 감지할 수도 있다.47


"노드가 보이지 않아요" 또는 "데이터가 안 와요"와 같은 막연한 문제를 체계적으로 해결하기 위한 워크플로우는 다음과 같다.

1. **참여자 발견 확인:** `participants` 명령어로 해당 노드의 `DomainParticipant`가 `Fast-DDS-spy`에 의해 발견되었는지 확인한다. 만약 보이지 않는다면, 네트워크 연결 문제(방화벽, 서브넷 등), 도메인 ID 불일치, 또는 디스커버리 서버 설정 오류일 수 있다.
2. **엔드포인트 발견 확인:** 참여자는 보이지만 엔드포인트가 보이지 않는다면, `writers`와 `readers` 명령어로 `DataWriter`와 `DataReader`가 생성되었는지 확인한다.
3. **토픽 매칭 확인:** `topics` 명령어로 원하는 토픽이 목록에 있는지, 그리고 `(writers|readers)` 카운트가 올바른지 확인한다. 카운트가 `(1|0)`이라면, `DataWriter`는 있지만 매칭되는 `DataReader`가 없다는 의미이다. 이는 QoS 불일치나 파티션 불일치 때문일 수 있다.
4. **데이터 흐름 확인:** 토픽이 정상적으로 매칭된 것으로 보인다면 (`(1|1)` 이상), `echo <TopicName>` 명령어로 실제 데이터가 흐르는지 확인한다. 데이터가 보이지 않는다면, 애플리케이션 로직에서 `write()` 호출이 제대로 이루어지지 않고 있을 가능성을 의심해볼 수 있다.

네트워크가 불안정한 WiFi 환경 등에서는 Fast DDS의 `LARGE_DATA` 전송 프로파일을 사용하여 UDP 대신 TCP 기반으로 데이터를 전송하도록 변경하는 것이 문제 해결에 도움이 될 수 있다.51


`Fast DDS Monitor`가 성능 분석의 주력 도구이지만, `Fast-DDS-spy`도 특정 상황에서 유용하게 사용될 수 있다.

- **토픽 스톰(Topic Storm) 감지:** `echo` 명령어를 실행했을 때 특정 토픽의 데이터가 비정상적으로 높은 빈도로 출력된다면, 해당 토픽이 네트워크 대역폭을 과도하게 점유하는 '토픽 스톰'의 원인일 수 있다.
- **관찰자 효과(Observer Effect) 관리:** `Fast-DDS-spy` 자체도 DDS 네트워크의 참여자이므로, 네트워크와 CPU 자원을 소모한다. 특히 고속 토픽을 `echo`하면 상당한 부하를 유발할 수 있다. 이러한 '관찰자 효과'를 최소화하기 위해, YAML 설정에서 `max-rx-rate`나 `downsampling` 옵션을 사용하여 `Fast-DDS-spy`가 처리하는 데이터의 양을 조절하는 것이 중요하다.39 이는 실제 시스템의 동작에 미치는 영향을 줄이면서 필요한 정보를 얻기 위한 중요한 기술이다.




`Fast-DDS-spy`는 독립적인 애플리케이션이 아니라, 핵심 `Fast-DDS` 라이브러리 위에 구축된 도구이다. 이는 `Fast-DDS-spy`의 특정 버전이 호환되는 `Fast-DDS` 및 `Fast-CDR` 라이브러리 버전과 강하게 결속되어 있음을 의미한다. 개발 환경을 구성할 때, 사용 중인 `Fast-DDS-spy` 버전과 호환되는 라이브러리 버전을 사용하는 것은 예기치 않은 빌드 오류나 런타임 문제를 피하기 위해 매우 중요하다.

`Fast-DDS-spy`의 GitHub 릴리스 노트를 분석하면 각 버전별 종속성 정보를 명확히 파악할 수 있다.37 다음 표는 주요 버전 간의 호환성 매트릭스를 정리한 것이다.

| `Fast-DDS-spy` 버전 | 릴리스 날짜 | 필수 `Fast-DDS` 버전 | 필수 `Fast-CDR` 버전 | 주요 호환성 노트                     |
| ------------------- | ----------- | -------------------- | -------------------- | ------------------------------------ |
| **v1.2.0**          | 2025-05-05  | v3.2.2               | v2.2.5               | 최신 Fast DDS 버전 지원              |
| **v1.1.0**          | 2024-10-15  | v3.1.0               | v2.2.5               | Fast DDS v3.1 지원                   |
| **v1.0.0**          | 2024-09-16  | v3.0.1               | v2.2.4               | Fast DDS v3 정식 지원 시작           |
| **v0.4.0**          | 2024-03-26  | v2.14.0              | v2.2.0               | ROS 2 Humble 기본 버전과 유사        |
| **v0.3.0**          | 2024-01-26  | v2.13.1              | v2.1.3               | Fast DDS v2.13 미만 버전 지원 추가   |
| **v0.2.0**          | 2023-07-07  | v2.11.0              | v1.1.0               | 인터페이스 화이트리스트 등 기능 추가 |
| **v0.1.0**          | 2023-04-11  | (v2.10.1 추정)       | (v1.0.27 추정)       | 최초 릴리스                          |

이 표는 개발자가 자신의 환경에 맞는 `Fast-DDS-spy` 버전을 선택하거나, 기존 환경을 업그레이드할 때 어떤 라이브러리들을 함께 업데이트해야 하는지에 대한 필수적인 정보를 제공한다. Colcon을 사용하여 소스에서 빌드하는 경우, `.repos` 파일이 이러한 종속성을 자동으로 관리해주지만, 수동으로 환경을 구성할 때는 이 호환성 매트릭스가 매우 중요한 참조 자료가 된다.


물리학의 관찰자 효과와 마찬가지로, 네트워크를 모니터링하는 행위 자체가 네트워크의 상태에 영향을 미칠 수 있다. `Fast-DDS-spy`를 실행하는 것은 새로운 `DomainParticipant`를 네트워크에 추가하는 것과 같으며, 이는 디스커버리 트래픽을 유발하고 CPU 및 네트워크 자원을 소모한다.52

특히 `echo` 명령어를 사용하여 고속, 대용량 토픽(예: 카메라 이미지, 라이다 포인트 클라우드)을 구독할 경우 그 영향은 더욱 커진다. `Fast-DDS-spy`는 수신된 모든 데이터를 처리하고 터미널에 출력해야 하므로 상당한 CPU 부하를 발생시킬 수 있으며, 신뢰성 있는(reliable) 토픽의 경우 `ACKNACK` 메시지를 보내기 위해 추가적인 네트워크 대역폭을 사용한다.

이러한 성능 영향을 인지하고 관리하는 것이 중요하다. `Fast-DDS-spy`는 이러한 영향을 최소화하기 위한 몇 가지 제어 기능을 제공한다 39:

- **`max-rx-rate`:** 데이터 처리율을 제한하여 CPU 부하를 조절한다.
- **`downsampling`:** 수신된 데이터 중 일부만 샘플링하여 처리량을 줄인다.
- **`allowlist`/`blocklist`:** 관심 없는 토픽을 필터링하여 불필요한 구독 자체를 방지한다.

정밀한 성능 측정이 필요한 경우, `Fast-DDS-spy`보다는 `Fast DDS Monitor`와 같은 전용 성능 측정 도구를 사용하거나, 부하가 적은 '관찰자' 노드를 직접 구현하는 것이 더 적합할 수 있다.53

`Fast-DDS-spy`의 주된 목적은 성능 벤치마킹이 아니라 기능적 정확성과 상태를 검사하는 것임을 이해해야 한다.


`Fast-DDS-spy` 및 Fast DDS를 사용하면서 더 깊이 있는 정보가 필요하거나 문제에 직면했을 때, 다음과 같은 자료와 커뮤니티 채널을 활용할 수 있다.


- **`Fast DDS Spy` 공식 문서:** `Fast-DDS-spy`의 설치, 사용법, 설정에 대한 가장 정확하고 상세한 정보를 제공한다. (URL: `https://fast-dds-spy.readthedocs.io/`) 8
- **`Fast DDS` 공식 문서:** `Fast-DDS-spy`의 기반이 되는 Fast DDS 라이브러리의 모든 기능(DDS 개념, QoS, 보안, 전송 등)에 대한 포괄적인 정보를 담고 있다. (URL: `https://fast-dds.docs.eprosima.com/`) 3
- **Vulcanexus 문서:** eProsima가 제공하는 올인원 ROS 2 툴셋인 Vulcanexus의 문서로, ROS 2 환경에서 Fast DDS와 관련 도구들을 사용하는 방법에 대한 튜토리얼과 예제를 제공한다. (URL: `https://docs.vulcanexus.org/`) 52


오픈소스 프로젝트의 가장 큰 장점 중 하나는 활발한 커뮤니티이다. eProsima와 ROS 커뮤니티는 여러 채널을 통해 사용자를 지원한다.

- **GitHub Issues:** 버그 리포트, 기능 요청 등 코드와 직접적으로 관련된 구체적이고 재현 가능한 문제를 논의하는 공식적인 창구이다. `Fast-DDS-spy`와 `Fast-DDS`는 각각 별도의 저장소를 가지고 있다.26
- **GitHub Discussions:** 일반적인 질문, 아이디어 공유, 사용법에 대한 논의 등 보다 자유로운 형식의 소통을 위한 공간이다. 특정 이슈로 등록하기 전에 질문을 던져보기에 좋다.56
- **ROS Discourse:** ROS 2와의 통합, `rmw_fastrtps` 관련 문제 등 ROS 2 환경에서 발생하는 특수한 문제들을 논의하기에 가장 적합한 포럼이다. eProsima 개발자들도 이 포럼에 활발하게 참여하고 있다.58
- **상업적 지원:** eProsima는 프로젝트의 요구사항에 맞는 전문적인 기술 지원, 컨설팅, 맞춤형 개발 등 상업적 지원 서비스를 제공한다. 중요한 프로젝트나 긴급한 문제 해결이 필요한 경우 유용한 옵션이 될 수 있다.1

문제를 보고하거나 질문할 때는, 사용 중인 소프트웨어 버전(`Fast-DDS-spy`, `Fast-DDS`, OS, ROS 2 배포판 등), 문제 상황을 재현할 수 있는 최소한의 코드나 설정, 그리고 로그 메시지를 함께 제공하는 것이 빠르고 정확한 답변을 얻는 데 큰 도움이 된다.


`eProsima Fast DDS Spy`는 현대 분산 시스템의 핵심 통신 미들웨어인 DDS 네트워크를 위한 강력하고 필수적인 진단 도구이다. 본 보고서는 `Fast-DDS-spy`를 단순한 유틸리티를 넘어, DDS의 기본 원리부터 복잡한 네트워크 토폴로지 및 보안 환경에서의 고급 활용법까지 아우르는 심층적인 분석을 제공하고자 했다.

분석 결과, `Fast-DDS-spy`의 가장 큰 전략적 중요성은 그 유연성과 접근성에 있음을 확인했다. Fast DDS의 `DynamicTypes` 기능을 기반으로 한 아키텍처는 사용자가 데이터 타입에 대한 사전 지식 없이도 어떤 DDS 네트워크든 즉시 검사할 수 있게 해준다. 이는 개발 및 디버깅 과정의 생산성을 획기적으로 향상시키는 핵심적인 특징이다. CLI 기반의 인터페이스는 터미널 환경에 익숙한 개발자에게 신속하고 정밀한 제어를 가능하게 하며, YAML 기반의 출력과 설정은 스크립트를 통한 자동화 및 체계적인 관리를 용이하게 한다.

또한, `Fast-DDS-spy`는 단독으로 존재하지 않고 `Fast DDS Monitor`, `DDS Router` 등과 함께 eProsima의 포괄적인 툴링 에코시스템을 구성한다. 사용자는 당면한 과제에 따라 최적의 도구를 선택하거나 조합하여, 시스템 설계부터 배포, 운영, 사후 분석에 이르는 개발의 전 주기를 효과적으로 지원받을 수 있다.

결론적으로, `Fast-DDS-spy`는 eProsima Fast DDS 또는 ROS 2를 사용하여 중요한 시스템을 개발하는 모든 엔지니어와 연구자에게 없어서는 안 될 도구이다. 이 도구의 작동 원리를 깊이 이해하고, 본 보고서에서 제시된 다양한 설치, 사용 및 문제 해결 기법들을 숙달함으로써, 개발자는 보이지 않는 데이터의 세계를 명확하게 파악하고, 더 견고하고 신뢰성 높은 분산 시스템을 구축할 수 있는 강력한 역량을 갖추게 될 것이다. `Fast-DDS-spy`는 복잡한 DDS 네트워크의 미스터리를 풀어내는 열쇠이자, 개발자의 통찰력을 한 단계 끌어올리는 정밀한 분석 장비이다.


1. eProsima: Middleware, Robots and AI, accessed July 6, 2025, https://www.eprosima.com/
2. eProsima Fast DDS Spy, accessed July 6, 2025, https://www.eprosima.com/middleware/tools/fast-dds-spy
3. eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/
4. eProsima Fast DDS - The middleware powering the ESO ELT, accessed July 6, 2025, https://www.eso.org/sci/meetings/2023/RTC4AO/01_11_martin_losa.pdf
5. Internet of Things - eProsima, accessed July 6, 2025, https://www.eprosima.com/middleware/sectors/internet-of-things
6. eProsima/Fast-DDS: The most complete DDS - Proven: Plenty of success cases. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS
7. DST Relies on Fast DDS for Advanced Inland Navigation Automation Platforms - eProsima, accessed July 6, 2025, https://www.eprosima.com/news/dst-relies-on-fast-dds-for-advanced-inland-navigation-automation-platforms
8. eProsima Fast DDS Spy Documentation - Fast DDS Spy .. documentation, accessed July 6, 2025, https://fast-dds-spy.readthedocs.io/
9. Fast DDS Spy Documentation, accessed July 6, 2025, https://fast-dds-spy.readthedocs.io/_/downloads/en/stable/pdf/
10. RTPS Introduction - eProsima, accessed July 6, 2025, https://www.eprosima.com/developer-resources/whitepapers/rtps
11. 8. Security - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/security/security.html
12. Using Fast DDS Discovery Server as discovery protocol [community-contributed] - ROS 2 Documentation: Jazzy documentation, accessed July 6, 2025, https://docs.ros.org/en/jazzy/Tutorials/Advanced/Discovery-Server/Discovery-Server.html
13. eProsima documentation index - all-docs 1.0 documentation, accessed July 6, 2025, https://docs.eprosima.com/
14. Tools - eProsima, accessed July 6, 2025, https://www.eprosima.com/middleware/tools
15. eProsima Fast DDS Monitor, accessed July 6, 2025, https://www.eprosima.com/middleware/tools/fast-dds-monitor
16. eProsima Fast DDS Monitor - Graph View Demo - YouTube, accessed July 6, 2025, https://www.youtube.com/watch?v=YxIklcaJV48
17. Fast DDS Spy, accessed July 6, 2025, https://fast-dds-spy.readthedocs.io/en/latest/
18. 1. Example of usage - Fast DDS Spy .. documentation - Read the Docs, accessed July 6, 2025, https://fast-dds-spy.readthedocs.io/en/latest/rst/user_manual/usage_example.html
19. Fast DDS Documentation, accessed July 6, 2025, https://media.readthedocs.org/pdf/eprosima-fast-rtps/stable/eprosima-fast-rtps.pdf
20. 10.10. Dynamic Types profiles - 3.2.2 - eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/xml_configuration/dynamic_types.html
21. RTI Connext DDS Core Libraries Extensible Types Guide, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/6.1.2/doc/manuals/connext_dds_professional/extensible_types_guide/RTI_ConnextDDS_CoreLibraries_ExtensibleTypes_Guide.pdf
22. Fast-DDS-docs/docs/fastdds/xml_configuration/dynamic_types.rst at master - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-docs/blob/master/docs/fastdds/xml_configuration/dynamic_types.rst
23. 4.2.1. Echo - Fast DDS Spy .. documentation, accessed July 6, 2025, https://fast-dds-spy.readthedocs.io/en/stable/rst/user_manual/commands/data.html
24. RTI DDS Spy, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/7.0.0/doc/manuals/connext_dds_professional/tools/rti_dds_spy/RTI_DDS_Spy.pdf
25. Comparing Open Source DDS to RTI Connext DDS: Considerations in Picking the Right DDS Solution to Run Your Distributed System, accessed July 6, 2025, https://www.rti.com/blog/picking-the-right-dds-solution
26. eProsima/Fast-DDS-spy - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-spy
27. 3. Linux installation from sources - 3.2.2 - eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/installation/sources/sources_linux.html
28. 1. Linux installation from sources - Fast DDS Spy - Read the Docs, accessed July 6, 2025, https://fast-dds-spy.readthedocs.io/en/stable/rst/developer_manual/installation/sources/linux.html
29. 2. Windows installation from sources - Fast DDS Spy ..., accessed July 6, 2025, https://fast-dds-spy.readthedocs.io/en/latest/rst/developer_manual/installation/sources/windows.html
30. 5. Mac OS installation from sources - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/installation/sources/sources_mac.html
31. 5. Mac OS installation from sources - Fast DDS 2.6.10 documentation, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/2.6.x/installation/sources/sources_mac.html
32. 3. Docker Image (recommended) - Fast DDS Spy .. documentation, accessed July 6, 2025, https://fast-dds-spy.readthedocs.io/en/latest/rst/installation/docker.html
33. Install Vulcanexus Fast DDS Spy on Linux | Snap Store - Snapcraft, accessed July 6, 2025, https://snapcraft.io/vulcanexus-fastddsspy
34. Install Vulcanexus Fast DDS Spy on Red Hat Enterprise Linux using the Snap Store, accessed July 6, 2025, https://snapcraft.io/install/vulcanexus-fastddsspy/rhel
35. Install Vulcanexus Fast DDS Spy on Ubuntu using the Snap Store - Snapcraft, accessed July 6, 2025, https://snapcraft.io/install/vulcanexus-fastddsspy/ubuntu
36. 2. User Interface - Fast DDS Spy .. documentation - Read the Docs, accessed July 6, 2025, https://fast-dds-spy.readthedocs.io/en/latest/rst/user_manual/tool.html
37. Releases / eProsima/Fast-DDS-spy - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-spy/releases
38. configuration.rst - eProsima/Fast-DDS-spy - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-spy/blob/main/docs/rst/user_manual/configuration.rst
39. 3. Configuration - Fast DDS Spy .. documentation - Read the Docs, accessed July 6, 2025, https://fast-dds-spy.readthedocs.io/en/latest/rst/user_manual/configuration.html
40. 1.1.3. How to solve wireless network issues in ROS 2 - - Vulcanexus Documentation, accessed July 6, 2025, https://docs.vulcanexus.org/en/jazzy/rst/tutorials/core/wifi/wifi_issues_tutorial/wifi_issues_tutorial.html
41. 11. Environment variables - Fast DDS 2.14.4 documentation, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/2.14.x/fastdds/env_vars/env_vars.html
42. 1.3.1. Configuring Fast-DDS QoS via XML profiles - Vulcanexus Documentation, accessed July 6, 2025, https://docs.vulcanexus.org/en/iron/rst/tutorials/core/qos/xml_profiles/xml_profiles.html
43. 10. XML profiles - Fast DDS 2.14.4 documentation, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/2.14.x/fastdds/xml_configuration/xml_configuration.html
44. 5.3.4. Discovery Server Settings - 3.2.2 - eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/discovery/discovery_server.html
45. ROS2 foxy eProsima Fast RTPS communication between Docker ubuntu container and Windows host [10508] / Issue #1698 - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS/issues/1698
46. 3.2.5. Partitions - 3.2.2 - eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/v3.2.2/fastdds/dds_layer/domain/domainParticipant/partition.html
47. 3.3.5. DataWriterListener - 3.2.2 - eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/dds_layer/publisher/dataWriterListener/dataWriterListener.html
48. 19.7. Security Frequently Asked Questions - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/v3.2.2/fastdds/faq/security/security.html
49. 5. Basic QoS - RTI Connext Getting Started documentation - RTI Community, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/getting_started_guide/cpp11/intro_qos.html
50. Usage - RTI DDS Spy 7.5.0 documentation - RTI Community, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/tools/rti_dds_spy/usage.html
51. 18. Troubleshooting - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/troubleshooting/troubleshooting.html
52. 3.7. Fast DDS Spy - Vulcanexus 1.0.0 documentation, accessed July 6, 2025, https://docs.vulcanexus.org/en/iron/rst/introduction/tools/fastddsspy.html
53. eProsima Fast DDS Performance, accessed July 6, 2025, https://www.eprosima.com/developer-resources/performance/eprosima-fast-dds-performance
54. How to track DDS performance in ROS 2 - ROS Discourse, accessed July 6, 2025, https://discourse.ros.org/t/how-to-track-dds-performance-in-ros-2/22498
55. Issues / eProsima/Fast-DDS - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS/issues
56. Welcome to Fast-DDS Discussions! #2390 - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS/discussions/2390
57. eProsima Fast-DDS / Discussions - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS/discussions
58. eProsima Fast DDS: from Shared Memory to ZERO COPY - ROS Discourse, accessed July 6, 2025, https://discourse.ros.org/t/eprosima-fast-dds-from-shared-memory-to-zero-copy/18877
59. FastDDS without Discovery Server? - ROS General - Open Robotics Discourse, accessed July 6, 2025, https://discourse.ros.org/t/fastdds-without-discovery-server/26117
60. ROS Discourse, accessed July 6, 2025, https://discourse.ros.org/


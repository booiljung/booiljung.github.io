---
layout: page
title: Fast-DDS-Gen에 대한 심층 분석 및 활용 가이드
permalink: /services/dds/Fast-DDS/Fast-DDS-Gen에 대한 심층 분석 및 활용 가이드
---


데이터 분산 서비스(Data Distribution Service, DDS)는 실시간 분산 시스템을 위한 OMG(Object Management Group) 표준으로, 데이터 중심(data-centric) 통신 패러다임을 통해 복잡한 네트워크 환경에서 안정적이고 효율적인 데이터 교환을 가능하게 합니다. eProsima Fast DDS는 이러한 DDS 표준의 C++ 구현체 중 가장 널리 사용되는 오픈 소스 미들웨어로, 특히 로봇 운영 체제(ROS 2)의 기본 미들웨어로 채택되면서 그 중요성이 더욱 부각되고 있습니다.1

이러한 DDS 기반 시스템의 개발 과정에서 핵심적인 역할을 수행하는 도구가 바로 코드 생성기(Code Generator)입니다. Fast-DDS-Gen은 eProsima Fast DDS 생태계의 공식 코드 생성기로, 개발자가 정의한 데이터 타입을 기반으로 통신에 필요한 C++ 또는 Python 코드를 자동으로 생성하여 개발 과정을 단순화하고 생산성을 극대화합니다.3

본 보고서는 두 부분으로 구성됩니다. **1부 'Fast-DDS-Gen에 대한 고찰'**에서는 DDS와 RTPS 프로토콜의 기본 원리부터 시작하여, 데이터 중심 아키텍처에서 코드 생성기가 차지하는 필연적인 역할과 Fast-DDS-Gen의 내부 아키텍처를 심층적으로 분석합니다. 또한, RTI Connext DDS, OpenDDS와 같은 경쟁 솔루션과의 비교를 통해 Fast-DDS-Gen의 기술적 위상과 설계 철학을 조명합니다. 이 부분은 기술 책임자나 시스템 아키텍트가 기술 스택 선정에 필요한 전략적 통찰력을 얻는 것을 목표로 합니다.

**2부 'Fast-DDS-Gen 활용 가이드'**에서는 실제 개발자를 위한 실용적인 매뉴얼을 제공합니다. 시스템 요구사항 및 의존성 분석, 상세한 설치 과정, 기본 및 고급 명령어 옵션, IDL 데이터 모델링 모범 사례, 그리고 현장에서 발생할 수 있는 다양한 문제에 대한 해결 방안까지, Fast-DDS-Gen을 활용하여 애플리케이션을 개발, 디버깅, 최적화하는 데 필요한 모든 정보를 망라합니다.

본 보고서는 Fast DDS를 도입하려는 팀이나 이미 사용 중인 개발자 모두에게 신뢰할 수 있는 기술적 지침서가 되는 것을 목표로 하며, Fast-DDS-Gen의 단순한 사용법을 넘어 그 이면에 있는 아키텍처와 설계 원칙까지 이해할 수 있도록 돕고자 합니다.




DDS(Data Distribution Service)의 핵심 패러다임은 '글로벌 데이터 공간(Global Data Space)'이라는 추상화 개념에서 출발합니다.2 이는 전통적인 메시지 전달 방식(message-passing)이나 원격 프로시저 호출(RPC)과 근본적으로 다른 접근법입니다. 개발자는 "어떻게 바이트 버퍼를 저쪽 노드로 보낼까?"를 고민하는 대신, "시스템 내 특정 데이터 객체의 현재 상태는 무엇인가?"에 집중하게 됩니다. 이 모델에서 데이터 생산자(Publisher)와 소비자(Subscriber)는 서로를 직접 인식하지 않고, 오직 이 공유된 데이터 공간을 통해 상호작용합니다. 애플리케이션은 특정 '토픽(Topic)'에 해당하는 데이터 객체를 생성하거나, 상태를 갱신하거나, 구독함으로써 정보를 교환합니다.6

이러한 데이터 중심 모델이 성공적으로 작동하기 위해서는 시스템에 참여하는 모든 노드가 데이터의 '타입(Type)'에 대해 명확하고 일관된 정의를 공유해야 합니다. 데이터 타입은 생산자와 소비자 간의 유일한 계약(contract) 역할을 하므로, 분산 시스템의 여러 구성 요소에 걸쳐 이 타입 계약을 엄격하게 정의하고 강제하는 메커니즘은 단순한 편의 기능이 아니라 아키텍처의 핵심 요구사항이 됩니다. 바로 이 지점에서 인터페이스 정의 언어(IDL)의 필요성이 대두됩니다.


RTPS(Real-Time Publish-Subscribe) 프로토콜은 DDS 표준의 근간을 이루는 OMG 표준 와이어 프로토콜(wire protocol)입니다.1 DDS가 애플리케이션 개발자를 위한 API와 기능 집합 표준이라면, RTPS는 실제로 네트워크(예: UDP)를 통해 전송되는 데이터의 형식을 정의하여 서로 다른 벤더의 DDS 구현체 간 상호운용성을 보장합니다.8

eProsima Fast DDS의 중요한 설계 특징 중 하나는 두 개의 API 계층을 제공한다는 점입니다. 첫째는 사용성에 초점을 맞춘 고수준의 DDS 호환 API이며, 둘째는 RTPS 프로토콜 내부에 더 세밀하게 접근할 수 있는 저수준 API입니다.1 이러한 이중 계층 구조는 개발자에게 중요한 선택지를 제공합니다. 대부분의 애플리케이션은 표준적이고 사용하기 쉬운 DDS API를 통해 개발할 수 있지만, 고도로 전문화되거나 극도의 성능 최적화가 필요한 경우 DDS 추상화 계층을 우회하고 RTPS 프로토콜과 직접 상호작용할 수 있습니다. 이러한 유연성은 eProsima 구현체의 강력한 장점으로, 고급 사용자를 위한 '탈출구'를 제공합니다.


eProsima Fast DDS는 단일 제품이 아니라, 강력한 핵심 미들웨어를 중심으로 다양한 전문 도구들이 유기적으로 연결된 생태계를 구성합니다. Fast DDS 라이브러리가 통신의 중심을 담당하며, 그 주변에는 다음과 같은 특화된 도구들이 포진해 있습니다 10:

- **Fast DDS Monitor**: DDS 네트워크의 상태, 통신 매개변수(지연 시간, 처리량 등)를 실시간으로 모니터링하는 그래픽 데스크톱 애플리케이션.1
- **DDS Router**: 서로 다른 LAN이나 DDS 도메인에 속한 네트워크를 연결하는 브리지 역할.10
- **DDS Record & Replay**: 네트워크상의 DDS 데이터를 MCAP 형식으로 저장하고, 기록된 시점과 동일하게 재생하는 도구.10
- **Fast DDS Spy**: DDS 네트워크를 탐색하여 도메인 참여자, 엔드포인트, 토픽 등의 정보를 인간이 읽을 수 있는 형태로 보여주는 CLI 도구.10
- **Discovery Server**: 대규모 시스템에서 검색 단계를 가속화하고 중앙 집중식 검색을 지원하는 메커니즘.10

이러한 생태계 내에서 Fast-DDS-Gen은 데이터 타입을 정의하고 C++/Python 코드를 생성하는, 개발 워크플로우의 가장 첫 단계를 책임지는 핵심 도구로 자리매김합니다. eProsima의 전략은 가볍고 고성능인 핵심 라이브러리를 제공하고, 이를 독립적인 오픈 소스 도구 모음으로 보완하는 것입니다. 이는 필요한 기능만 선택적으로 도입할 수 있는 모듈성의 장점을 제공하지만, 한편으로는 사전 통합된 상용 솔루션에 비해 사용자에게 더 많은 통합 부담을 줄 수도 있습니다. 이는 프로젝트 계획 및 자원 할당 시 중요한 고려사항입니다.


앞서 논의된 개념들을 종합하면, 강력한 타입 시스템을 갖춘 '글로벌 데이터 공간'을 RTPS를 통해 상호운용 가능하게 유지하기 위해서는, 특정 프로그래밍 언어에 종속되지 않는 방식으로 데이터 구조를 정의하는 방법이 필수적입니다. OMG의 인터페이스 정의 언어(Interface Definition Language, IDL)가 바로 이 문제에 대한 표준 해결책입니다.3

그리고 Fast-DDS-Gen과 같은 코드 생성기는 이러한 추상적인 IDL 정의를 특정 프로그래밍 언어(예: C++)를 위한 구체적이고, 컴파일 가능하며, 효율적인 코드로 변환하는 불가결한 다리 역할을 합니다.2 Fast-DDS-Gen의 핵심 기능은 개발자가 직렬화(serialization) 및 역직렬화(deserialization) 메커니즘에 대한 깊은 지식 없이도 DDS 애플리케이션을 쉽게 구현할 수 있도록 돕는 것입니다.3 생성된 코드는 내부적으로 Fast CDR 라이브러리를 호출하여 OMG IDL 타입을 CDR(Common Data Representation) 전송 구문에 따라 바이트 스트림으로 매핑하는 복잡하고 오류가 발생하기 쉬운 작업을 자동화합니다. 이 자동화야말로 DDS의 데이터 중심 모델을 실용적으로 구현할 수 있게 만드는 핵심 요소입니다.



Fast-DDS-Gen은 OMG IDL 명세의 일부를 파싱하여 C++ 소스 코드를 생성하는 Java 애플리케이션입니다.3 이 도구가 Java로 개발되었다는 점은 중요한 아키텍처적 선택입니다. 이는 코드 생성 도구 자체의 플랫폼 독립성을 확보하여, 개발자는 리눅스, 윈도우, macOS 등 어떤 운영체제에서든 동일한 도구를 실행하여 특정 플랫폼을 위한 C++ 코드를 생성할 수 있습니다.13 예를 들어, 윈도우 개발자가 생성한 소스 코드를 리눅스 빌드 서버에서 컴파일할 때, 해당 서버에는 Java가 설치될 필요가 없어 교차 플랫폼 개발 워크플로우를 용이하게 합니다.

Fast-DDS-Gen이 생성하는 핵심 클래스는 `TopicDataType`과 `TypeSupport`입니다.3

- `TopicDataType`: 발행과 구독 간에 교환되는 데이터, 즉 토픽에 해당하는 데이터 타입을 기술합니다. 직렬화, 역직렬화, 키(key) 추출과 같은 핵심 로직을 포함합니다.
- `TypeSupport`: `TopicDataType`의 인스턴스를 캡슐화하고, 생성된 타입을 DDS `DomainParticipant`에 등록하는 데 필요한 함수들을 제공합니다.

이 두 클래스는 IDL에 정의된 추상적인 데이터 구조를 Fast DDS 미들웨어가 이해하고 처리할 수 있는 구체적인 객체로 변환하는 역할을 수행합니다.


Fast-DDS-Gen의 아키텍처는 명확한 관심사 분리(separation of concerns) 원칙을 따릅니다. 이 도구는 직렬화 로직을 처음부터 생성하는 것이 아니라, eProsima의 고도로 최적화된 C++11 직렬화 라이브러리인 `Fast CDR`을 *호출*하는 C++ 코드를 생성합니다.3 Fast CDR은 CDR 표준에 따라 데이터를 바이트 스트림으로 변환하는 실제 작업을 담당하는 독립적인 라이브러리입니다.16

이러한 구조는 Fast-DDS-Gen을 '설계도 생성기'로, Fast CDR을 '엔진'으로 비유할 수 있습니다. 이 아키텍처적 분리는 상당한 이점을 제공합니다. 직렬화 엔진인 Fast CDR은 코드 생성기와 독립적으로 최적화되거나 업데이트될 수 있습니다. 예를 들어, 데이터 정렬(alignment)이나 인코딩 방식의 버그 수정이 필요할 경우, Fast-DDS-Gen을 변경할 필요 없이 새로운 Fast CDR 라이브러리 릴리스만으로 해결할 수 있습니다. 이는 소프트웨어의 유지보수성과 유연성을 크게 향상시키는 견고한 엔지니어링 설계입니다.


Fast-DDS-Gen을 실행하면 IDL 파일 이름(예: `MyType.idl`)을 기반으로 여러 파일이 생성됩니다. 이 파일 구조는 Fast DDS 아키텍처의 관심사 분리를 명확하게 반영합니다.4

- `MyType.hpp`: IDL `struct`에 해당하는 C++ 클래스 정의가 포함된 헤더 파일입니다. 데이터 멤버에 접근하기 위한 `set()` 및 `get()` 멤버 함수가 함께 생성됩니다.17
- `MyTypePubSubType.cxx` / `.h`: `TopicDataType`의 구체적인 구현체입니다. 데이터의 직렬화/역직렬화, 최대 직렬화 크기 계산, 그리고 토픽이 키를 가질 경우 키를 추출하는 `getKey()` 멤버 함수 정의 등을 포함합니다.4 DDS 타입 관련 로직의 핵심입니다.
- `MyTypeCdrAux.hpp` / `.ipp`: Fast CDR 라이브러리가 타입 인코딩 및 디코딩을 위해 필요로 하는 보조 템플릿 및 헬퍼 함수들을 포함합니다. 저수준 직렬화 인터페이스 역할을 합니다.4
- `MyTypeTypeObjectSupport.cxx` / `.hpp`: 동적 타입 발견(XTypes)을 위해 필요한 `TypeObject` 생성 및 등록 관련 보조 코드를 담고 있습니다.4

이처럼 모듈화된 파일 구조는 개발자가 문제 해결 시 접근해야 할 부분을 명확히 알려줍니다. 예를 들어, 직렬화 과정에서 데이터 손상이 의심된다면 `CdrAux` 파일을, `DomainParticipant`에 타입 등록이 실패한다면 `PubSubType` 파일을 중심으로 분석할 수 있어 디버깅 효율을 높입니다.


DDS XTypes(Extensible and Dynamic Topic Types)는 컴파일 시점에 알지 못했던 데이터 타입을 런타임에 발견하고 사용할 수 있게 해주는 강력한 OMG 표준입니다. 이 기능은 DDS 네트워크상의 모든 데이터를 기록하거나 모니터링하는 범용 도구를 제작할 때 필수적입니다.

Fast-DDS-Gen의 `-typeobject` 플래그는 바로 이 XTypes 기능을 활성화하는 스위치입니다.11 이 플래그를 사용하면, Fast-DDS-Gen은 데이터 타입의 구조 정보를 담고 있는 메타데이터인 `TypeObject`와 이를 타입 팩토리에 등록하는 코드를 추가로 생성합니다.18 이를 통해 애플리케이션은 원격 참여자가 사용하는 타입을 동적으로 이해하고 해당 타입의 데이터를 해석할 수 있게 됩니다.

그러나 이 강력한 유연성에는 대가가 따릅니다. XTypes를 활성화하면 참여자 간 검색(discovery) 단계에서 `TypeObject` 정보를 교환하기 위한 추가적인 네트워크 트래픽이 발생합니다.20 시스템 규모가 크거나 네트워크 대역폭이 제한적인 환경에서는 이 오버헤드가 성능에 영향을 미칠 수 있습니다. 따라서 시스템 설계자는 애플리케이션의 요구사항(유연성 vs. 성능)을 고려하여 `-typeobject` 사용 여부를 신중하게 결정해야 하는 중요한 아키텍처적 트레이드오프에 직면하게 됩니다.



Fast-DDS-Gen은 오픈 소스 커뮤니티, 특히 ROS 2와 같은 환경에서 소스 코드 중심의 워크플로우에 익숙한 '빌더(builder)' 성향의 개발자를 위해 설계되었습니다. 그 설계 철학은 다음과 같은 특징으로 요약됩니다 1:

- **오픈 소스 및 CLI 중심**: 그래픽 인터페이스 대신 강력한 명령줄 인터페이스(CLI)를 제공하여 스크립트를 통한 자동화 및 `colcon`과 같은 빌드 시스템과의 통합을 용이하게 합니다.
- **핵심 언어 집중**: 주로 C++과 Python 코드 생성에 집중하여 해당 언어 사용자에게 최적화된 경험을 제공합니다.
- **생태계 통합**: Fast CDR, ROS 2와의 긴밀한 통합을 특징으로 하며, `-typeros2`와 같은 플래그를 통해 ROS 2와의 호환성을 명시적으로 지원합니다.

Fast-DDS-Gen의 가치는 강력하고 집중된 구성 요소들로부터 시스템을 조립하는 개발자에게 최고의 유연성과 제어권을 제공하는 데 있습니다.


RTI Connext DDS의 `rtiddsgen`은 미션 크리티컬한 기업 및 임베디드 시스템 시장을 겨냥한 상용 제품군의 일부입니다. 이는 단순한 코드 생성기를 넘어, 개발 위험과 시장 출시 기간 단축을 목표로 하는 수직적으로 통합된 솔루션의 일부로 기능합니다.22

- **광범위한 언어 지원**: C++ 외에도 Ada, Java, C# 등 다양한 언어를 지원하여 폭넓은 개발 환경 요구를 충족합니다.22
- **포괄적인 기능 및 인증**: `rtiddsgen`은 방대한 QoS 정책, 보안 기능, 그리고 DO-178C와 같은 안전 인증을 지원하는 버전과 긴밀하게 통합되어 있습니다. 이는 항공, 국방, 의료 등 고신뢰성이 요구되는 분야에서 결정적인 장점입니다.22
- **통합된 툴링**: `System Designer`와 같은 그래픽 디자인 도구를 제공하여 사용자가 시각적으로 시스템을 설계하면 관련 설정 파일이나 코드가 생성되는 등, 사용자 친화적인 개발 환경을 제공합니다.22

`rtiddsgen`의 가치 제안은 도구 자체의 기능을 넘어, 그를 둘러싼 강력한 지원, 인증, 툴링 인프라 전체에 있으며, 이는 상용 라이선스 비용을 정당화하는 핵심 요소입니다.


OpenDDS의 `opendds_idl`은 또 다른 주요 오픈 소스 코드 생성기입니다. Fast-DDS-Gen과 마찬가지로 CLI 기반이지만, 그 아키텍처는 CORBA의 유산에 깊이 뿌리내리고 있다는 점에서 차별화됩니다.

OpenDDS의 코드 생성 프로세스는 역사적으로 TAO CORBA ORB의 IDL 컴파일러(`tao_idl`)에 의존하는 2단계로 구성되는 경우가 많습니다. `opendds_idl`이 DDS 관련 추가 코드(예: `*TypeSupport.idl`)를 생성하면, `tao_idl`이 이를 처리하여 최종 언어 매핑을 수행하는 방식입니다.24 이러한 구조는 TAO IDL 컴파일러의 성숙도를 활용할 수 있다는 장점이 있지만, Fast-DDS-Gen의 단일 단계 프로세스에 비해 다소 복잡하게 느껴질 수 있습니다. 이 툴체인 아키텍처의 차이점은 두 오픈 소스 옵션 사이에서 선택할 때 중요한 고려사항입니다.


세 가지 코드 생성 도구의 특징을 종합적으로 비교하면 다음과 같습니다. 이 표는 기술 스택을 결정해야 하는 아키텍트나 기술 리더에게 각 솔루션의 장단점을 한눈에 파악할 수 있는 유용한 의사결정 도구를 제공합니다.

| 기능               | eProsima Fast-DDS-Gen                                        | RTI Connext `rtiddsgen`                                      | OpenDDS `opendds_idl`                                        |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **라이선스 모델**  | Apache 2.0 (오픈 소스)                                       | 상용 라이선스 (연구/교육용 예외)                             | 자체 오픈 소스 라이선스                                      |
| **주요 지원 언어** | C++, Python                                                  | C++, C++03/11, Java, Ada, C#,.Net                            | C++ (클래식 및 C++11 매핑)                                   |
| **핵심 차별점**    | ROS 2와의 긴밀한 통합, 경량성, 현대적인 C++ 중심 설계        | 포괄적인 기능, 안전 인증(Cert), 광범위한 언어 지원, 전문 기술 지원 | CORBA/TAO 기반의 성숙한 아키텍처, 유연한 전송 프레임워크     |
| **생태계 및 툴링** | CLI 중심, `Fast DDS Monitor` 등 독립적인 오픈 소스 도구 모음 | `System Designer` 등 통합된 그래픽 툴, 다양한 상용 서비스 및 게이트웨이 | CLI 중심, ACE/TAO 생태계와 연계                              |
| **ROS 2 통합**     | 기본 미들웨어로 공식 채택 및 지원 (`rmw_fastrtps_cpp`)       | 상용 RMW 지원 (`rmw_connextdds`)                             | 커뮤니티 기반 RMW 지원 (`rmw_opendds_cpp`)                   |
| **주요 사용 사례** | 로보틱스(ROS 2), 자율주행, IIoT 등 오픈 소스 기반의 고성능 시스템 | 항공, 국방, 금융, 의료 등 미션 크리티컬 및 엔터프라이즈 시스템 | 대규모 분산 시스템, 기존 CORBA 시스템과의 연동이 필요한 경우 |




Fast DDS 및 Fast-DDS-Gen을 소스에서 빌드하기 전에, 시스템에 다음의 요구사항과 의존성이 충족되어야 합니다. 버전 호환성은 성공적인 빌드의 핵심 요소입니다.13

- **필수 도구**:
  - `CMake`: 빌드 시스템 생성기
  - `g++` (Linux/macOS) 또는 `Visual Studio` (Windows): C++ 컴파일러
  - `git`: 소스 코드 버전 관리
  - `Java JDK`: Fast-DDS-Gen은 Java 애플리케이션이므로 실행을 위해 필요합니다. 버전 8 또는 11이 권장됩니다.13
  - `Gradle` (또는 내장 `gradlew` 사용): Fast-DDS-Gen 빌드를 위해 필요합니다.13
- **핵심 라이브러리 의존성**:
  - `foonathan_memory_vendor`: STL 호환 메모리 할당자 라이브러리.13
  - `eProsima Fast CDR`: 데이터 직렬화를 위한 라이브러리.16
  - `Asio` 및 `TinyXML2`: 네트워크 및 XML 파싱을 위한 라이브러리.16
  - `OpenSSL`: 보안 통신(DDS-Security)을 활성화할 경우 필요합니다.16


가장 간단한 설치 방법은 eProsima에서 제공하는 사전 컴파일된 바이너리를 사용하는 것입니다. 이는 복잡한 빌드 과정을 생략하고 빠르게 환경을 구축할 수 있게 해줍니다.13

1. eProsima 공식 웹사이트에서 사용자의 운영체제에 맞는 최신 바이너리 패키지를 다운로드합니다.

2. 다운로드한 압축 파일을 원하는 디렉토리에 해제합니다.

3. 터미널 또는 명령 프롬프트를 열고 압축 해제된 디렉토리로 이동합니다.

4. (Linux) 관리자 권한으로 설치 스크립트를 실행합니다:

   ```Bash
   sudo./install.sh
   ```
   
5. (Windows) 설치 프로그램을 실행하고 화면의 지시에 따라 설치를 완료합니다.


소스 코드로부터 직접 빌드하는 것은 최신 기능을 사용하거나 특정 옵션으로 컴파일해야 할 때 필요합니다. 여기서는 ROS 2 중심의 `colcon` 방식과 전통적인 `CMake` 방식을 모두 설명합니다.

**Colcon을 이용한 빌드 (ROS 2 사용자에게 권장)** 26

1. `colcon`과 `vcstool` 등 ROS 2 개발 도구를 설치합니다.

   ```Bash
   pip3 install -U colcon-common-extensions vcstool
   ```
   
2. 작업 공간(workspace)을 생성하고 소스 코드를 다운로드합니다.

   ```Bash
   mkdir -p fastdds_ws/src
   cd fastdds_ws
   wget https://raw.githubusercontent.com/eProsima/Fast-DDS/master/fastdds.repos
   vcs import src < fastdds.repos
   ```
   
3. 의존성을 설치하고 빌드를 실행합니다.

   ```Bash
   rosdep install --from-paths src --ignore-src -r -y  # ROS 2 환경에서
   # 또는 수동으로 의존성 설치
   colcon build --packages-up-to fastdds fastddsgen
   ```

**CMake를 이용한 빌드 (전통적인 방식)** 13

1. 의존성 라이브러리를 순서대로 클론하고 빌드 및 설치합니다.

   ```Bash
   # 1. foonathan_memory_vendor
   git clone https://github.com/eProsima/foonathan_memory_vendor.git
   cd foonathan_memory_vendor && mkdir build && cd build
   cmake..
   sudo cmake --build. --target install
   cd../../
   
   # 2. Fast CDR
   git clone https://github.com/eProsima/Fast-CDR.git
   cd Fast-CDR && mkdir build && cd build
   cmake..
   sudo cmake --build. --target install
   cd../../
   
   # 3. Fast DDS
   git clone https://github.com/eProsima/Fast-DDS.git
   cd Fast-DDS && mkdir build && cd build
   cmake..
   sudo cmake --build. --target install
   cd../../
   ```
   
2. Fast-DDS-Gen을 빌드합니다.

   ```Bash
   git clone --recursive https://github.com/eProsima/Fast-DDS-Gen.git
   cd Fast-DDS-Gen
   ./gradlew assemble
   ```


설치 후에는 시스템이 `fastddsgen` 실행 파일과 런타임에 필요한 공유 라이브러리를 찾을 수 있도록 환경 변수를 설정해야 합니다. 이 단계는 초보자가 가장 흔하게 겪는 문제의 원인이므로 매우 중요합니다.13

컴파일 시점(CMake가 헤더와 라이브러리를 찾는 것)과 실행 시점(운영체제가 공유 라이브러리를 로드하는 것)은 다릅니다. `LD_LIBRARY_PATH`(Linux/macOS) 또는 `PATH`(Windows) 설정은 실행 시점에 필요한 라이브러리의 위치를 알려주는 역할을 합니다.

- **Linux/macOS**:

  - **임시 설정 (현재 터미널 세션에만 적용)**:

    ```Bash
    export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
    export PATH=<path_to_fastddsgen>/scripts:$PATH
    ```
    
  - **영구 설정 (`.bashrc` 또는 `.zshrc`에 추가)**:

    ```Bash
    echo 'export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
    echo 'export PATH=<path_to_fastddsgen>/scripts:$PATH' >> ~/.bashrc
    source ~/.bashrc
    ```
  
- **Windows**:

  - 시스템 환경 변수 설정에서 `Path` 변수에 `C:\Program Files\eProsima\fastdds\lib`와 `C:\Program Files\eProsima\fast-dds-gen\scripts` 디렉토리를 추가합니다.

이 설정이 누락되면 애플리케이션이 성공적으로 컴파일되더라도 실행 시 "cannot open shared object file"과 같은 오류가 발생할 수 있습니다.


이 장에서는 DDS의 "Hello, World"에 해당하는 간단한 발행/구독 애플리케이션을 단계별로 제작하는 과정을 안내합니다. Fast-DDS-Gen의 `-example` 플래그는 이 과정을 매우 간단하게 만들어주며, 신규 사용자가 빠르게 DDS 개발을 시작할 수 있는 훌륭한 부트스트래핑 도구입니다.4


먼저, 통신에 사용할 데이터 타입을 IDL 파일로 정의합니다. `HelloWorld.idl`이라는 이름의 파일을 생성하고 다음 내용을 입력합니다.18

```
// HelloWorld.idl
struct HelloWorld
{
    unsigned long index;
    string message;
};
```

이 구조체는 전송될 메시지의 순번을 나타내는 `index`와 메시지 내용을 담는 `message` 두 멤버로 구성됩니다.


터미널에서 다음 명령을 실행하여 IDL 파일로부터 완전한 발행/구독 예제 애플리케이션 코드를 생성합니다. `<platform>` 부분은 사용자의 환경에 맞게 `Linux`, `x64Win64VS2019` 등으로 지정합니다.4

```Bash
fastddsgen -example <platform> HelloWorld.idl
```

이 명령 하나로 타입 지원 파일뿐만 아니라, 컴파일 가능한 `HelloWorldPublisher.cxx`, `HelloWorldSubscriber.cxx` 및 빌드 스크립트(Makefile 또는 Visual Studio 프로젝트)까지 모두 생성됩니다.


생성된 `HelloWorldPublisher.cxx` 파일을 열어 `run()` 함수 부분을 찾습니다. 여기에 데이터를 초기화하고 발행하는 코드를 추가합니다.30

```C++
// In HelloWorldPublisher.cxx, inside the run() method
//...
HelloWorld st; // 생성된 데이터 타입의 인스턴스

// 데이터를 초기화합니다.
st.index(1);
st.message("Hello World!");

// 데이터 발행
publisher_->write(&st);
//...
```


`HelloWorldSubscriber.cxx` 파일을 열어 `on_data_available()` 리스너 함수를 확인합니다. 이 함수는 새로운 데이터가 도착했을 때 호출됩니다. 수신된 데이터를 화면에 출력하는 코드를 추가할 수 있습니다.30

```C++
// In HelloWorldSubscriber.cxx, inside on_data_available()
//...
if (reader_->take_next_sample(&st, &info) == ReturnCode_t::RETCODE_OK)
{
    if (info.valid_data)
    {
        this->samples_++;
        std::cout << "Message: " << st.message() << " " << st.index() << " RECEIVED" << std::endl;
    }
}
//...
```


생성된 빌드 스크립트를 사용하여 애플리케이션을 컴파일합니다.

- **Linux (Makefile 사용)**:

  ```Bash
  make
  ```
  
- **Windows (Visual Studio 프로젝트 사용)**:

  - 생성된 `.sln` 파일을 열고 빌드합니다.

컴파일이 완료되면 두 개의 터미널을 열고 각각 Subscriber와 Publisher를 실행합니다.30

- **터미널 1 (Subscriber 실행)**:

  ```sh
  ./HelloWorld subscriber
  ```

- **터미널 2 (Publisher 실행)**:

  ```sh
  ./HelloWorld publisher
  ```

Publisher 터미널에서 'y'를 입력할 때마다 메시지가 전송되고, Subscriber 터미널에서 "Message: Hello World! 1 RECEIVED"와 같은 출력으로 데이터가 성공적으로 수신되었음을 확인할 수 있습니다.



Fast-DDS-Gen의 모든 기능은 명령줄 옵션을 통해 제어됩니다. 다음 표는 각 옵션에 대한 상세한 설명과 사용 예시를 제공하여 개발자가 필요에 따라 정확한 플래그를 신속하게 찾을 수 있도록 돕는 실용적인 참조 가이드입니다.4

| 옵션                            | 설명                                                         | 사용 사례                                                    | 예시                                                         |
| ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `-d <directory>`                | 생성된 파일의 출력 디렉토리를 지정합니다.                    | 소스 코드를 특정 폴더(예: `generated-src`)에 정리하고자 할 때 사용합니다. | `fastddsgen -d generated-src MyType.idl`                     |
| `-example <platform>`           | 지정된 플랫폼을 위한 완전한 발행/구독 예제 애플리케이션과 빌드 파일을 생성합니다. | DDS를 처음 시작하거나 빠르게 프로토타입을 만들 때 유용합니다. | `fastddsgen -example Linux MyType.idl`                       |
| `-replace`                      | 출력 파일이 이미 존재하더라도 강제로 덮어씁니다.             | IDL 변경 후 코드를 다시 생성할 때 사용합니다.                | `fastddsgen -replace MyType.idl`                             |
| `-python`                       | C++ 타입 지원 코드와 함께 Python 바인딩을 생성하고, 이를 컴파일하기 위한 CMake 프로젝트를 만듭니다. `-example`과 함께 사용할 수 없습니다. | Python으로 DDS 애플리케이션을 개발하고자 할 때 사용합니다.   | `fastddsgen -python MyType.idl`                              |
| `-typeros2`                     | ROS 2의 타입 명명 규칙과 호환되는 코드를 생성합니다.         | ROS 2 환경에서 커스텀 메시지 타입을 사용하고자 할 때 필수적입니다. | `fastddsgen -typeros2 MyType.idl`                            |
| `-typeobject`                   | DDS XTypes를 지원하기 위해 `TypeObject` 메타데이터 생성 코드를 추가합니다. | 동적 타입 발견이 필요한 모니터링, 로깅, 브리지 애플리케이션 개발 시 사용합니다. | `fastddsgen -typeobject MyType.idl`                          |
| `-cs`                           | IDL에 정의된 변수 이름의 대소문자를 구분하도록 설정합니다.   | 대소문자 구분이 중요한 명명 규칙을 따를 때 사용합니다.       | `fastddsgen -cs MyType.idl`                                  |
| `-I <directory>`                | IDL 파일 전처리 시 `#include` 지시문을 위한 검색 경로를 추가합니다. | 공통 IDL 파일을 여러 프로젝트에서 공유할 때 사용합니다.      | `fastddsgen -I../common_types MyType.idl`                    |
| `-no-typesupport`               | `TypeSupport` 관련 파일 생성을 비활성화합니다.               | 순수 데이터 타입 클래스 정의만 필요할 때 사용합니다.         | `fastddsgen -no-typesupport MyType.idl`                      |
| `-extrastg <template> <output>` | 사용자 정의 StringTemplate(.stg) 파일을 사용하여 코드 생성을 커스터마이징합니다. | 생성되는 코드에 특정 주석이나 로직을 추가하는 등 고급 커스터마이징이 필요할 때 사용합니다. | `fastddsgen -extrastg custom.stg custom_output.cpp MyType.idl` |
| `-help`                         | 도움말 정보를 표시합니다.                                    | 사용 가능한 모든 옵션과 지원되는 플랫폼 목록을 확인할 때 사용합니다. | `fastddsgen -help`                                           |
| `-version`                      | Fast-DDS-Gen의 버전을 표시합니다.                            | 설치된 도구의 버전을 확인할 때 사용합니다.                   | `fastddsgen -version`                                        |


`-example` 플래그는 Fast-DDS-Gen의 가장 유용한 기능 중 하나입니다. 단순히 타입 지원 코드만 생성하는 것을 넘어, 즉시 컴파일하고 실행할 수 있는 완전한 애플리케이션 골격을 제공합니다.4

`-help` 옵션을 통해 지원되는 플랫폼 목록(예: `Linux`, `Macos`, `x64Win64VS2017`, `x64Win64VS2019` 등)을 확인할 수 있으며, 개발 환경에 맞는 플랫폼을 지정하여 해당 환경에 최적화된 빌드 파일(Makefile, Visual Studio 솔루션 등)을 얻을 수 있습니다.


Fast DDS는 C++뿐만 아니라 Python도 지원합니다. `-python` 플래그는 IDL에 정의된 데이터 타입을 Python에서 사용할 수 있도록 바인딩 코드를 생성하는 역할을 합니다.3 이 옵션을 사용하면 데이터 타입 라이브러리를 빌드하기 위한 `CMakeLists.txt` 파일도 함께 생성됩니다. 빌드된 라이브러리를 Python 스크립트에서 `import`하여 C++ 애플리케이션과 동일한 데이터 타입으로 통신할 수 있습니다. 최근 릴리스에서는 `sequence<string>` 이나 `sequence<enum>`과 같은 복합 타입에 대한 Python 생성 지원이 개선되고 있습니다.33


ROS 2는 내부적으로 DDS를 사용하며, Fast DDS는 ROS 2의 기본 RMW(ROS Middleware Wrapper)입니다.1 ROS 2에서 사용자가 정의한 커스텀 메시지(`.msg` 파일)를 사용하려면, 이를 DDS가 이해할 수 있는 IDL로 변환한 후 코드를 생성해야 합니다. `-typeros2` 플래그는 이 과정에서 생성되는 C++ 타입의 이름과 네임스페이스를 ROS 2의 규칙에 맞게 조정하여 원활한 통합을 보장합니다.21 일반적인 워크플로우는 ROS 2의 `.msg` 파일을 `.idl` 파일로 변환하고, `-typeros2` 플래그를 사용하여 타입 지원 코드를 생성한 후, 이를 ROS 2 패키지에 포함하여 빌드하는 순서로 진행됩니다.34


매우 특수한 요구사항이 있는 고급 사용자를 위해, Fast-DDS-Gen은 `-extrastg` 옵션을 통해 코드 생성 로직을 직접 수정할 수 있는 기능을 제공합니다.21 이 옵션은 StringTemplate 엔진에서 사용하는 `.stg` 템플릿 파일과 출력 파일명을 인자로 받습니다. 개발자는 기본 템플릿을 복사하여 수정한 뒤, 이 옵션을 통해 자신만의 코드 생성 규칙(예: 특정 형식의 주석 추가, 로깅 코드 자동 삽입 등)을 적용할 수 있습니다. 다만, 템플릿 문법에 대한 이해가 필요하며, 현재 GitHub 이슈에서는 템플릿 관련 버그나 사용성 문제가 논의되고 있어 주의가 필요합니다.35


IDL 데이터 모델링은 단순히 데이터 구조를 표현하는 것을 넘어, 분산 시스템의 성능, 확장성, 유지보수성에 직접적인 영향을 미치는 중요한 설계 과정입니다.


Fast-DDS-Gen은 표준 IDL 타입들을 C++11 표준 라이브러리와 자연스럽게 매핑하여 개발 편의성을 높입니다. 다음 표는 주요 IDL 타입과 그에 해당하는 C++11 매핑을 정리한 것입니다. 개발자는 이 표를 통해 IDL에 정의한 타입이 실제 C++ 코드에서 어떻게 표현될지 명확히 예측할 수 있습니다.36

| IDL 타입          | C++11 매핑                | 비고                  |
| ----------------- | ------------------------- | --------------------- |
| `long`            | `int32_t`                 | 32비트 정수           |
| `unsigned long`   | `uint32_t`                | 부호 없는 32비트 정수 |
| `long long`       | `int64_t`                 | 64비트 정수           |
| `octet`           | `uint8_t`                 | 1바이트 데이터        |
| `boolean`         | `bool`                    | 불리언                |
| `string`          | `std::string`             | 문자열                |
| `char a`          | `std::array<char, 5> a`   | 고정 크기 배열        |
| `sequence<long>`  | `std::vector<int32_t>`    | 동적 크기 시퀀스      |
| `map<char, long>` | `std::map<char, int32_t>` | 맵 (키-값 쌍)         |


DDS에서 '키(key)'는 데이터를 관리하는 가장 중요한 개념 중 하나입니다. IDL 구조체의 멤버에 `@key` 어노테이션을 붙이면, 해당 멤버가 토픽의 '인스턴스(instance)'를 식별하는 데 사용됩니다.17 예를 들어, 항공기 비행 데이터를 다루는 토픽에서 항공기 ID(`flight_id`)를 키로 지정하면, `flight_id=123`인 데이터와 `flight_id=456`인 데이터는 동일 토픽 내의 서로 다른 인스턴스로 취급됩니다.7

이를 통해 Subscriber는 특정 인스턴스의 데이터만 수신하거나, 각 인스턴스의 최신 상태를 관리하는 등 정교한 데이터 필터링 및 관리가 가능해집니다. Fast-DDS-Gen은 `@key` 멤버를 기반으로 `getKey()` 함수를 자동으로 생성하여 이 과정을 지원합니다.4


장기간 운영되는 분산 시스템에서는 데이터 타입이 변경되거나 확장될 필요가 생깁니다. DDS XTypes의 '확장성(Extensibility)' 개념은 기존 애플리케이션과의 호환성을 유지하면서 데이터 타입을 안전하게 진화시킬 수 있는 메커니즘을 제공합니다. Fast-DDS-Gen은 다음 세 가지 확장성 어노테이션을 지원합니다.17

- `@final`: 타입이 엄격하게 정의되었음을 나타냅니다. 멤버를 추가하거나 변경하면 타입 호환성이 깨집니다. 성능이 중요하고 타입 변경이 없을 것으로 예상될 때 사용합니다.
- `@appendable`: 기존 멤버 끝에 새로운 멤버를 추가하는 것을 허용합니다. 가장 일반적인 확장 시나리오에 적합합니다.
- `@mutable`: 멤버의 추가, 제거, 순서 변경까지 허용하는 가장 유연한 확장성입니다. 유연성이 높은 만큼 성능 오버헤드가 가장 큽니다.

데이터 타입을 설계할 때 예상되는 변경 가능성을 고려하여 적절한 확장성을 명시하는 것은 시스템의 장기적인 유지보수성을 위해 매우 중요합니다.


Fast-DDS-Gen은 `@key`와 확장성 외에도 다양한 어노테이션을 지원하여 미들웨어의 동작을 세밀하게 제어할 수 있게 합니다.

| 어노테이션        | 목적                                                         | 예시 IDL 사용법                                              |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `@id(value)`      | 구조체나 공용체의 멤버에 고유한 ID를 명시적으로 할당합니다. 타입 진화 시 멤버를 식별하는 데 사용됩니다. | `struct MyData { @id(0) long member_a; @id(2) string member_b; };` |
| `@hashid("name")` | 멤버 이름의 해시 값을 ID로 사용하도록 지정합니다.            | `struct MyData { @hashid("legacy_name") long member_a; };`   |
| `@optional`       | 구조체 멤버가 선택 사항임을 나타냅니다. 해당 멤버는 전송되지 않을 수 있습니다. | `struct MyData { long required_member; @optional string optional_member; };` |
| `@value(number)`  | 열거형(enum) 리터럴에 특정 정수 값을 할당합니다.             | `enum Status { OK = 0, FAILED = 1, @value(10) TIMEOUT };`    |
| `@autoid(HASH)`   | 멤버 ID를 명시하지 않았을 때, 멤버 이름의 해시를 기반으로 ID를 자동으로 할당합니다. (기본값: SEQUENTIAL) | `@autoid(HASH) struct MyData { long member_a; };`            |

이러한 어노테이션을 활용하면 개발자는 데이터 모델 정의 단계에서부터 데이터의 호환성, 선택적 전송 등 미들웨어의 고급 동작을 설계할 수 있습니다.


IDL에서 일대다(one-to-many) 관계를 모델링할 때 흔히 두 가지 접근 방식을 고려할 수 있습니다. `carpool`과 `car`의 관계를 예로 들어 그 트레이드오프를 분석해 보겠습니다.38

- **접근 방식 1: 내장 시퀀스 (Embedded Sequence)**

  ```
struct car { long id; string color; };
  struct carpool { @key long id; sequence<car> cars; };
  ```
  
  이 방식은 `carpool` 구조체 내에 `car`의 시퀀스를 직접 포함합니다. 모델이 직관적이라는 장점이 있지만, 심각한 성능 문제를 야기할 수 있습니다. 예를 들어, `carpool`에 새로운 `car` 하나를 추가하는 경우, 변경되지 않은 다른 모든 `car` 정보를 포함한 전체 `carpool` 객체를 다시 발행해야 합니다. 이는 네트워크 대역폭을 심각하게 낭비하는 비효율적인 방식입니다.

- **접근 방식 2: 외래 키 (Foreign Key)**

  Code snippet

  ```
  struct carpool { @key long id; string name; };
  struct car { @key long id; long carpoolId; string color; };
  ```

  이 방식은 데이터베이스 설계와 유사하게, `car` 구조체에 자신이 속한 `carpool`을 가리키는 `carpoolId`라는 외래 키를 둡니다. 새로운 `car`가 추가되면, 해당 `car` 객체만 발행하면 됩니다. Subscriber 측 애플리케이션은 `carpoolId`를 통해 수신된 `car`들을 재구성하는 책임을 집니다. 이 방식은 업데이트 단위를 최소화하여 네트워크 효율성을 극대화하므로, 확장 가능한 대규모 DDS 시스템을 설계할 때 강력히 권장되는 패턴입니다.



Fast DDS 빌드 중 발생하는 오류는 대부분 의존성 문제에서 기인합니다. 예를 들어, `make` 또는 `colcon build` 실행 시 `cannot find -lOpenSSL::SSL`과 같은 오류가 발생한다면, 이는 시스템에 OpenSSL 개발 라이브러리가 설치되지 않았거나 CMake가 이를 찾지 못했음을 의미합니다.39 이 경우, 해당 라이브러리의 개발 패키지(예: `libssl-dev`)를 설치하고 CMake 캐시를 삭제한 후 다시 시도해야 합니다. GitHub 이슈 트래커에는 특정 템플릿 파일(`stg`)을 찾지 못하거나 35, `gradlew assemble`이 실패하는 등 35 빌드 환경과 관련된 다양한 문제가 보고되고 있으므로, 유사한 문제 발생 시 참고할 수 있습니다.


개발 환경에서 가장 흔하게 접하는 문제 중 하나는 Publisher와 Subscriber가 서로를 발견하지 못하는 '검색(discovery)' 문제입니다.

- **WiFi 또는 손실이 많은 네트워크**: 기본 UDP 전송은 WiFi나 Docker 브리지 네트워크와 같은 손실이 많은 환경에서 검색 패킷 유실로 인해 불안정할 수 있습니다. 이 경우, `FASTDDS_BUILTIN_TRANSPORTS` 환경 변수를 `LARGE_DATA`로 설정하는 것이 효과적인 해결책입니다.20 이 프로필은 검색 단계에서는 UDP를 사용하지만, 실제 데이터 교환에는 더 신뢰성 있는 TCP를 사용하여 안정성을 높입니다.
- **대규모 시스템의 검색 지연**: 노드 수가 수십 개 이상으로 증가하면 기본 P2P 검색 방식의 메시지 교환량이 기하급수적으로 늘어나 검색이 지연되거나 실패할 수 있습니다.40 이 경우, 중앙 서버가 검색을 중개하는 'Discovery Server' 메커니즘을 사용하는 것이 좋습니다. 이는 검색 트래픽을 크게 줄여 대규모 시스템의 확장성을 향상시킵니다.40
- **ROS 2 환경에서의 문제**: ROS 2 Humble 버전부터 Fast DDS가 다시 기본 RMW가 되면서, 특정 토픽(특히 `tf`)의 재연결 실패나 노드 재시작 시 검색이 불안정해지는 문제가 커뮤니티에 보고된 바 있습니다.40 이는 복잡한 라이프사이클을 가진 노드들이 동시에 시작하고 중지될 때 발생할 수 있는 경쟁 상태와 관련이 있을 수 있습니다.


비디오 스트림이나 포인트 클라우드와 같이 크기가 큰 데이터를 전송할 때는 추가적인 튜닝이 필요합니다.

- **샘플 손실**: 기본 QoS 설정, 특히 History의 `depth`가 너무 작으면 데이터 생성 속도를 수신 속도가 따라가지 못해 오래된 샘플이 덮어씌워져 유실될 수 있습니다.41 이는 특히 통계 데이터를 모니터링하는 애플리케이션에서 흔히 발생하며, DataWriter의 History `depth`를 늘려 버퍼를 확보함으로써 해결할 수 있습니다.

- **성능 저하**: 압축되지 않은 원시 이미지 데이터를 높은 프레임률로 전송하는 것은 DDS 자체의 문제라기보다는 데이터 형식의 비효율성 문제일 수 있습니다.42 데이터를 전송하기 전에 H.264와 같은 코덱으로 압축하거나, 대역폭을 많이 차지하지 않는 더 효율적인 데이터 타입을 설계하는 것을 우선적으로 고려해야 합니다. `LARGE_DATA` 프로필 사용과 QoS 튜닝이 이러한 시나리오에서 도움이 될 수 있습니다.20


Fast-DDS-Gen으로 생성된 코드를 애플리케이션에 통합할 때 컴파일 또는 링커 오류가 발생할 수 있습니다.

- **중첩된 IDL 파일**: IDL 파일이 다른 IDL 파일을 `#include`하는 경우, 생성된 코드 간의 의존성 문제로 링커 오류가 발생할 수 있다는 이슈가 보고되었습니다.35
- **특정 데이터 타입 문제**: 공용체(union), 중첩된 `typedef`, 특정 타입의 시퀀스 등 복잡한 데이터 구조를 사용할 때 컴파일 오류가 발생했다는 보고가 다수 존재합니다.35 이는 IDL 파서나 코드 생성 템플릿의 엣지 케이스 처리 미흡으로 인한 것일 수 있습니다.
- **이름 충돌**: 기존 코드에 이미 존재하는 이름의 구조체를 IDL로 정의하면, 생성된 클래스 이름이 충돌할 수 있습니다. Fast-DDS-Gen은 자동으로 이름에 접미사를 붙이는 옵션을 제공하지 않으므로, 이 경우 IDL 내에서 구조체 이름을 변경하여 충돌을 피해야 합니다.43


eProsima/Fast-DDS-Gen의 GitHub 이슈 트래커는 공식 문서에서는 다루지 않는 실제 사용자들이 겪는 문제들을 파악할 수 있는 중요한 정보원입니다.35 최근 보고된 주요 이슈들은 다음과 같은 경향을 보입니다:

- **코드 생성 템플릿(.stg) 문제**: 특정 조건문이나 문법을 사용할 때 템플릿 오류가 발생하거나, `-extrastg` 옵션 사용 시 경로 문제가 발생하는 등 템플릿 엔진과 관련된 이슈가 꾸준히 제기되고 있습니다.35
- **IDL 파싱 오류**: 중첩된 `typedef`나 모듈, 열거형(enum)과 정수 상수를 함께 사용하는 경우, ROS 2 메시지 변환 시 등 특정 엣지 케이스에서 IDL 파싱 오류나 컴파일 오류가 보고되고 있습니다.35
- **Python 바인딩**: `sequence<string>`과 같은 특정 타입에 대한 Python 코드 생성 시 오류가 발생하는 문제가 보고되어, Python 지원이 아직 완전히 안정화되지 않았음을 시사합니다.35

이러한 실제 사례들은 Fast-DDS-Gen이 강력한 도구임과 동시에, 복잡한 사용 사례에서는 여전히 개선될 여지가 있음을 보여줍니다. 따라서 개발자는 문제 발생 시 공식 문서와 더불어 GitHub 이슈를 적극적으로 검색하여 유사 사례나 진행 중인 수정 사항을 확인하는 것이 문제 해결에 큰 도움이 될 수 있습니다.


eProsima Fast-DDS-Gen은 현대적인 실시간 분산 시스템 개발, 특히 ROS 2 생태계에서 중추적인 역할을 담당하는 필수 불가결한 도구입니다. 본 보고서의 심층 분석을 통해, Fast-DDS-Gen은 단순히 코드를 생성하는 유틸리티를 넘어, 데이터 중심 아키텍처의 핵심 철학을 구현하고 개발자가 저수준의 직렬화 복잡성에서 벗어나 애플리케이션 로직에 집중할 수 있도록 지원하는 핵심적인 추상화 계층임을 확인했습니다. Java 기반의 플랫폼 독립적인 실행 환경, Fast CDR과의 명확한 역할 분리, 그리고 동적 타입(XTypes)과 같은 고급 DDS 표준을 지원하는 유연성은 Fast-DDS-Gen의 견고한 아키텍처 설계를 보여줍니다.

RTI Connext DDS나 OpenDDS와 같은 다른 솔루션과의 비교는 각 도구가 지향하는 목표와 시장이 다름을 명확히 합니다. RTI가 상용 시장을 위한 포괄적인 기능과 인증, 지원을 제공한다면, Fast-DDS-Gen은 오픈 소스 기반의 고성능, 고유연성 시스템을 구축하는 개발자에게 최적화된 선택지입니다.

활용 가이드 부분에서는 설치부터 기본 튜토리얼, 고급 옵션, IDL 설계 패턴, 그리고 문제 해결에 이르기까지 Fast-DDS-Gen을 효과적으로 사용하는 데 필요한 실질적인 지식을 체계적으로 정리했습니다. 특히, IDL 설계에서 외래 키 패턴을 사용하는 것의 중요성이나, 네트워크 환경에 따라 `LARGE_DATA` 프로필을 적용하는 것과 같은 모범 사례들은 단순한 기능 습득을 넘어, 성능과 확장성을 고려한 고품질 시스템을 설계하는 데 필수적인 통찰을 제공합니다.

결론적으로, Fast-DDS-Gen은 eProsima Fast DDS 생태계의 성공을 이끄는 핵심 동력입니다. 지속적인 개선과 커뮤니티의 피드백을 통해 일부 엣지 케이스의 안정성이 보강된다면, 앞으로도 로보틱스, 자율주행, IIoT 등 다양한 분야에서 데이터 중심 통신의 표준 도구로서 그 입지를 더욱 공고히 할 것으로 전망됩니다. 본 보고서가 Fast DDS를 사용하는 모든 기술자와 아키텍트에게 신뢰할 수 있는 길잡이가 되기를 바랍니다.


1. eProsima/Fast-DDS: The most complete DDS - Proven: Plenty of success cases. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS
2. Fast DDS Documentation, accessed July 6, 2025, https://media.readthedocs.org/pdf/eprosima-fast-rtps/stable/eprosima-fast-rtps.pdf
3. 1. Introduction - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastddsgen/introduction/introduction.html
4. Fast-DDS-docs/docs/fastdds/dds_layer/topic/fastddsgen/fastddsgen.rst at master / eProsima/Fast-DDS-docs - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-RTPS-docs/blob/master/docs/fastdds/dds_layer/topic/fastddsgen/fastddsgen.rst
5. robotics - What's the difference between ROS2 and DDS? - Stack Overflow, accessed July 6, 2025, https://stackoverflow.com/questions/51187676/whats-the-difference-between-ros2-and-dds
6. eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/
7. Data-Centric Programming Best Practices: - RTI, accessed July 6, 2025, https://www.rti.com/hubfs/docs/DDS_Best_Practices_WP.pdf
8. eProsima/Fast-DDS-docs: Documentation of Fast RTPS (MarkDown Files). Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-docs
9. About different ROS 2 DDS/RTPS vendors, accessed July 6, 2025, https://docs.ros.org/en/foxy/Concepts/About-Different-Middleware-Vendors.html
10. eProsima documentation index - all-docs 1.0 documentation, accessed July 6, 2025, https://docs.eprosima.com/
11. fastddsgen - IDL source code generator - Ubuntu Manpage, accessed July 6, 2025, https://manpages.ubuntu.com/manpages/jammy/man1/fastddsgen.1.html
12. Fast DDS Documentation, accessed July 6, 2025, https://media.readthedocs.org/pdf/eprosima-fast-rtps/latest/eprosima-fast-rtps.pdf
13. Fast DDS Installation | PX4 User Guide (v1.12), accessed July 6, 2025, https://docs.px4.io/v1.12/en/dev_setup/fast-dds-installation.html
14. Fast DDS Installation | PX4 User Guide (v1.13), accessed July 6, 2025, https://docs.px4.io/v1.13/en/dev_setup/fast-dds-installation.html
15. 5. Mac OS installation from sources - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/installation/sources/sources_mac.html
16. 3. Linux installation from sources - Fast DDS 2.6.10 documentation, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/2.6.x/installation/sources/sources_linux.html
17. 5. Defining a data type via IDL - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/v3.2.2/fastddsgen/dataTypes/dataTypes.html
18. 1. Configuring Fast DDS DynamicTypes - eProsima DDS Record & Replay Documentation, accessed July 6, 2025, https://dds-recorder.readthedocs.io/en/v0.3.0/rst/tutorials/dynamic_types.html
19. 1. Configuring Fast DDS DynamicTypes - eProsima DDS Record & Replay Documentation, accessed July 6, 2025, https://dds-recorder.readthedocs.io/en/v0.2.0/rst/tutorials/dynamic_types.html
20. 18. Troubleshooting - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/troubleshooting/troubleshooting.html
21. 2. Usage - 3.2.2 - eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastddsgen/usage/usage.html
22. Comparing Open Source DDS to RTI Connext DDS: Considerations ..., accessed July 6, 2025, https://www.rti.com/blog/picking-the-right-dds-solution
23. RTI Code Generator User's Manual, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/6.1.1/doc/manuals/connext_dds_professional/code_generator/users_manual/RTI_Code_Generator_UsersManual.pdf
24. opendds_idl - OpenDDS 3.33.0-dev, accessed July 6, 2025, https://opendds.readthedocs.io/en/master/devguide/opendds_idl.html
25. opendds_idl - OpenDDS 3.24.2, accessed July 6, 2025, https://opendds.readthedocs.io/en/dds-3.24.2/devguide/opendds_idl.html
26. 3. Linux installation from sources - 3.2.2 - eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/installation/sources/sources_linux.html
27. 1. Linux installation from binaries - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/v3.2.2/installation/binaries/binaries_linux.html
28. eProsima Fast DDS - ROS 2 Documentation, accessed July 6, 2025, https://docs.ros.org/en/foxy/Installation/DDS-Implementations/Working-with-eProsima-Fast-DDS.html
29. eProsima Fast DDS - ROS 2 Documentation, accessed July 6, 2025, https://docs.ros.org/en/humble/Installation/RMW-Implementations/DDS-Implementations/Working-with-eProsima-Fast-DDS.html
30. My first application in FastDDS. I have some experience dealing with ROS... | by Hitoruna, accessed July 6, 2025, https://medium.com/@hitorunajp/438750d9527c
31. eProsima Fast DDS: Publisher/Subscriber HelloWorld in Linux - YouTube, accessed July 6, 2025, https://m.youtube.com/watch?v=Vwo6fm9ZEJI&pp=ygUII2Zhc3RkZHM%3D
32. 3. Example of usage - 3.2.0 - eProsima Fast DDS Monitor Documentation, accessed July 6, 2025, https://fast-dds-monitor.readthedocs.io/en/latest/rst/getting_started/tutorial.html
33. Releases / eProsima/Fast-DDS-Gen - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-RTPS-Gen/releases
34. Identify a FastDDS example topic on ROS2 [13775] / eProsima Fast-DDS / Discussion #2467 - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS/discussions/2467
35. Issues / eProsima/Fast-DDS-Gen - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-Gen/issues
36. 5. Defining a data type via IDL - Fast DDS 2.14.4 documentation, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/2.14.x/fastddsgen/dataTypes/dataTypes.html
37. 1.4.3. Fast DDS - Vulcanexus Topic Intercommunication, accessed July 6, 2025, https://docs.vulcanexus.org/en/iron/rst/tutorials/core/deployment/dds2vulcanexus/dds2vulcanexus_topic.html
38. How to model in idl for DDS - Stack Overflow, accessed July 6, 2025, https://stackoverflow.com/questions/15337914/how-to-model-in-idl-for-dds
39. Fast DDS installation problems/errors - PX4 Discussion Forum, accessed July 6, 2025, https://discuss.px4.io/t/fast-dds-installation-problems-errors/22452
40. FastDDS without Discovery Server? - ROS General - Open Robotics Discourse, accessed July 6, 2025, https://discourse.ros.org/t/fastdds-without-discovery-server/26117
41. 13.1.4. Troubleshooting - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/statistics/dds_layer/troubleshooting.html
42. ROS2-Galactic DDS Problems - ROS General - Open Robotics Discourse, accessed July 6, 2025, https://discourse.ros.org/t/ros2-galactic-dds-problems/25654
43. How to elegantly convert IDL struct into generated DDS class? #225 - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-Gen/discussions/225


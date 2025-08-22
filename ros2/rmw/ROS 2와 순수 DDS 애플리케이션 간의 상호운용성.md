---
layout: page
title: ROS 2와 순수 DDS 애플리케이션 간의 상호운용성
permalink: /ros2/rmw/ROS 2와 순수 DDS 애플리케이션 간의 상호운용성
---


사용자의 핵심 질문, "ROS 2 애플리케이션이 보낸 메시지를 비-ROS 2(순수 DDS) 애플리케이션이 수신할 수 있는가?"에 대한 명확한 답변은 "예, 가능합니다"이다. 이는 우연한 기능이 아니라, ROS 2가 설계 단계부터 의도한 핵심적인 특징 중 하나이다. 이 보고서는 이것이 어떻게 가능한지에 대한 기술적 원리와 구체적인 구현 방법을 심층적으로 분석한다.


ROS 1은 TCPROS라는 독자적인 통신 프로토콜을 사용하여 ROS 생태계 내에서는 효율적이었지만, 외부 시스템과의 연동에는 한계가 있었다.1 반면, ROS 2는 개발 초기부터 학술 연구를 넘어 상용 및 산업용 시장으로의 확장을 목표로 삼았다.2 이를 위해 ROS 2는 통신 미들웨어로 업계 표준인 DDS(Data Distribution Service)를 채택하는 전략적 결정을 내렸다.3

DDS는 OMG(Object Management Group)에 의해 표준화된 데이터 중심의 발행-구독(Publish-Subscribe) 통신 프레임워크로, 이미 국방, 항공우주, 의료, 산업 자동화 등 미션 크리티컬 시스템에서 그 성능과 신뢰성을 입증받았다.3 DDS는 실시간성, 확장성, 데이터 신뢰성, 보안 등 산업 현장에서 요구하는 엄격한 요구사항들을 충족시키도록 설계되었다.3 따라서 ROS 2가 DDS를 채택한 것은 단순히 성능 좋은 미들웨어를 사용하는 것을 넘어, DDS가 제공하는 표준 기반의 개방형 생태계에 적극적으로 편입하여 산업계의 요구에 부응하겠다는 명확한 의지를 보여준다.1


순수 DDS 애플리케이션과의 상호운용성은 ROS 2 시스템에 다음과 같은 중요한 가치를 제공한다.

- **성능 최적화:** ROS 2의 추상화 계층(RMW)을 거치지 않고 DDS API를 직접 사용함으로써, 통신 지연 시간을 최소화하고 처리량을 극대화하는 최고 성능을 달성할 수 있다.8
- **레거시 시스템 통합:** 이미 현장에서 DDS를 기반으로 운영되고 있는 수많은 레거시 시스템(예: 제어 시스템, 모니터링 시스템)과 새로운 ROS 2 기반 로봇 시스템을 원활하게 통합할 수 있다.
- **이기종 시스템 구축:** ROS 2 생태계가 제공하는 풍부한 로봇 관련 도구 및 라이브러리의 장점과, 순수 DDS가 제공하는 강력한 데이터 관리 및 전송 기능을 결합하여 최적화된 하이브리드 시스템을 구축할 수 있다.8
- **언어 및 플랫폼 확장성:** DDS는 C++, Java, Python뿐만 아니라 다양한 프로그래밍 언어와 Linux, Windows, macOS, VxWorks 등 여러 운영체제를 지원한다.1 이는 ROS 2 클라이언트 라이브러리(RCL)가 공식적으로 지원되지 않는 환경에서도 해당 시스템이 ROS 2 네트워크에 참여할 수 있는 길을 열어준다.

이러한 배경을 바탕으로, 본 보고서는 ROS 2와 순수 DDS 간의 상호운용성을 가능하게 하는 핵심 아키텍처인 RMW를 분석하고, 통신을 위해 반드시 준수해야 할 기술적 요구사항들을 상세히 설명한다. 또한, 주요 DDS 벤더별 구현 방법과 실용적인 예제, 문제 해결을 위한 디버깅 전략을 포괄적으로 제시하여 개발자가 실제 시스템에 상호운용성을 성공적으로 적용할 수 있도록 돕는 것을 목표로 한다.


ROS 2와 순수 DDS 애플리케이션의 상호운용성을 이해하기 위해서는 먼저 ROS 2가 내부적으로 통신을 처리하는 방식, 특히 RMW(ROS Middleware Interface) 추상화 계층의 역할에 대한 이해가 필수적이다. RMW는 상호운용성의 '조력자'인 동시에, 그 규칙을 따라야만 통과할 수 있는 '관문' 역할을 한다.


RMW는 ROS 2의 상위 애플리케이션 코드(ROS 클라이언트 라이브러리, RCL을 통해 작성된)와 하부의 특정 DDS 구현체 사이를 중개하는 C 언어 기반의 API 계층이다.10 ROS 2 애플리케이션 개발자는 

`rclcpp::create_publisher()`와 같은 ROS 2 API를 호출하지만, 실제 네트워크 통신은 RMW를 통해 선택된 DDS 벤더(예: Fast DDS)의 API가 수행한다.2

RMW의 가장 큰 존재 이유는 '미들웨어 종속성 제거'이다. RMW 덕분에 개발자는 ROS 2 코드를 전혀 수정하지 않고, 단지 `RMW_IMPLEMENTATION`이라는 환경 변수를 `rmw_fastrtps_cpp`에서 `rmw_cyclonedds_cpp`나 `rmw_connextdds`로 변경하는 것만으로 기본 통신 미들웨어를 교체할 수 있다.13 이러한 유연성은 특정 벤더에 대한 종속을 피하고, 프로젝트의 요구사항(성능, 라이선스, 플랫폼 지원 등)에 가장 적합한 DDS 솔루션을 선택할 수 있게 해준다.10


순수 DDS 애플리케이션이 ROS 2와 통신하기 어려운 근본적인 이유는 RMW가 중간에서 '번역' 작업을 수행하기 때문이다. 순수 DDS 애플리케이션은 이 번역 규칙을 알지 못하므로, ROS 2가 사용하는 언어(개념)를 이해하지 못한다. RMW는 ROS 2의 고유 개념들을 DDS의 표준 개념으로 다음과 같이 매핑한다.

- **ROS 노드 (Node)** -->> **DDS 참여자 (Participant):** ROS 2의 각 노드는 DDS 세계에서 하나의 참여자로 표현된다.2
- **ROS 통신 객체 (Topic, Service, Action)** -->> **DDS 토픽 (Topic):** ROS의 모든 통신 메커니즘(토픽, 서비스, 액션)은 근본적으로 하나 이상의 DDS 토픽을 통해 구현된다.5
- **ROS 메시지 (`.msg`)** -->> **DDS 데이터 타입 (`.idl`):** ROS 2에서 사용자가 정의하는 `.msg` 파일은 빌드 과정에서 DDS 표준 인터페이스 정의 언어(IDL)인 `.idl` 파일로 변환된다.10

이 변환 과정의 중심에는 `rosidl`이라는 ROS 2의 인터페이스 툴체인이 있다. `rosidl_adapter` 패키지가 `.msg`, `.srv`, `.action` 파일을 파싱하고, `rosidl_generator_dds_idl` 패키지가 이를 DDS 표준 `.idl` 파일로 변환한다.17 이후 각 RMW 구현체(예: 

`rmw_fastrtps_cpp`)는 이 공통의 `.idl` 파일을 사용하여 각 DDS 벤더에 특화된 타입 지원(Type Support) 코드를 생성하고 컴파일한다.11

결론적으로, 순수 DDS 애플리케이션이 ROS 2 네트워크와 상호운용되기 위해서는 RMW라는 '번역기'가 내부적으로 수행하는 이 모든 변환 규칙(토픽 이름 변환, 데이터 타입 구조 및 이름 변환 등)을 수동으로 정확하게 복제해야만 한다.


RMW는 타입 지원 방식에 따라 크게 두 가지로 나뉜다.

- **정적 타입 지원 (Static Type Support):** 대부분의 RMW 구현체(예: `rmw_fastrtps_cpp`, `rmw_cyclonedds_cpp`)가 사용하는 기본 방식이다. 이 방식은 빌드 시점에 `.idl` 파일로부터 메시지 직렬화/역직렬화에 필요한 C++ 코드를 미리 생성한다. 이 미리 컴파일된 코드를 사용하므로 런타임 성능이 매우 뛰어나다.10 순수 DDS 애플리케이션과의 연동은 대부분 이 정적 타입 지원 방식의 규칙을 따르는 것을 의미한다.
- **동적 타입 지원 (Dynamic Type Support):** 일부 RMW 구현체(예: `rmw_fastrtps_dynamic_cpp`)는 DDS-XTypes 표준을 활용하여, 런타임에 메시지에 대한 메타 정보(타입 정보)를 교환하고 이를 기반으로 동적으로 데이터를 해석한다.10 이는 유연성을 높여주지만, 미리 컴파일된 코드를 사용하는 정적 방식에 비해 일반적으로 성능이 다소 저하될 수 있다.10


ROS 2와 순수 DDS 애플리케이션 간의 성공적인 통신은 다음 네 가지 핵심 기술 요구사항을 정확하게 일치시키는 것에 달려있다. 이 중 하나라도 어긋나면 통신은 실패하며, 디버깅은 이 네 가지 요소를 체계적으로 점검하는 과정이다.


RMW는 ROS 2의 이름 공간을 DDS 토픽 이름으로 변환할 때, 혼동을 피하고 내부적으로 통신 유형을 구분하기 위해 특정 규칙(Name Mangling)에 따라 접두사와 접미사를 추가한다. 순수 DDS 애플리케이션은 반드시 이 변환된 최종 이름을 사용하여 DDS 토픽을 생성하고 구독해야 한다.

- **토픽 (Topics):** 일반적인 ROS 토픽의 경우, 이름 앞에 `rt/` (ROS Topic) 접두사가 붙는다. 예를 들어, ROS 토픽 `/chatter`는 DDS 세계에서 `rt/chatter`라는 이름의 토픽으로 인식된다.20
- **서비스 (Services):** 요청-응답(Request-Reply) 패턴을 사용하는 ROS 서비스는 두 개의 별도 DDS 토픽으로 구현된다.
  - **요청 토픽:** `rq/` (ROS Request) 접두사와 `Request` 접미사가 붙는다. 예를 들어, ROS 서비스 `/add_two_ints`의 요청은 `rq/add_two_intsRequest`라는 DDS 토픽을 통해 전송된다.23
  - **응답 토픽:** `rr/` (ROS Reply) 접두사와 `Reply` 접미사가 붙는다. 위 서비스의 응답은 `rr/add_two_intsReply`라는 DDS 토픽으로 전송된다.23
- **액션 (Actions):** 목표, 결과, 피드백 등 더 복잡한 통신 패턴을 가지는 ROS 액션은 내부적으로 여러 개의 토픽과 서비스의 조합으로 구현되며, 각각의 요소는 서비스와 유사한 이름 변환 규칙을 따른다.16

이러한 변환 규칙의 기반이 되는 ROS 2의 공식 이름 지정 규칙(예: 이름은 `/`로 시작해야 하고, 연속된 `//`나 `__`를 포함할 수 없음) 또한 준수되어야 한다.24


단순히 토픽 이름만 맞추는 것으로는 부족하다. 해당 토픽을 통해 교환되는 데이터의 '형태'와 '이름' 또한 정확히 일치해야 한다.

- **IDL 구조 일치:** 순수 DDS 애플리케이션은 ROS 2가 사용하는 것과 100% 동일한 구조를 가진 IDL(Interface Definition Language) 파일을 사용해야 한다. 이 IDL 파일은 ROS 2를 소스에서 빌드할 때 생성되는 중간 파일을 직접 참조하거나, RTI Community GitHub와 같이 미리 변환된 IDL 세트를 제공하는 곳에서 얻을 수 있다.8
- **타입 이름 변환 (Type Name Mangling):** 가장 중요하면서도 놓치기 쉬운 부분이다. DDS 토픽에 등록되는 데이터 타입의 이름 또한 RMW에 의해 변환된다. OSRF(Open Source Robotics Foundation)의 공식 예제에서 제시된 규칙은 `::msg::dds_::[메시지_타입_이름]_` 형식이다.21
  - 예시 1: `std_msgs/msg/String` 메시지는 `std_msgs::msg::dds_::String_` 이라는 DDS 타입 이름으로 등록된다.
  - 예시 2: `unique_identifier_msgs/msg/UUID` 메시지는 `unique_identifier_msgs::msg::dds_::UUID_` 라는 타입 이름으로 등록된다.21
- **IDL 어노테이션:** DDS 벤더의 RMW 구현 방식에 따라 IDL 파일의 세부적인 해석이 달라질 수 있다. 예를 들어, Eclipse Cyclone DDS와 통신할 때는 IDL의 타입 정의(struct)에 `@final` 어노테이션을 사용해야 한다. `@appendable` 어노테이션을 사용하면 타입 불일치로 인해 통신에 실패할 수 있다. 이는 Cyclone DDS의 RMW가 DDS-XTypes 표준의 확장 가능한 타입 직렬화 방식(XCDR2) 대신, 더 단순하고 기본적인 CDR(Common Data Representation) 방식을 사용하기 때문이다.20


DDS의 강력한 기능 중 하나는 데이터 통신의 특성을 세밀하게 제어할 수 있는 풍부한 QoS(Quality of Service) 정책이다. DDS는 신뢰성, 내구성, 이력 관리 등 22가지 이상의 QoS 정책을 제공하며, ROS 2는 이 중 일부를 조합하여 사용한다.1

- **불일치의 결과:** 발행자(Publisher)와 구독자(Subscriber) 간의 QoS 정책이 호환되지 않으면, 두 참여자는 서로를 발견(Discovery)하더라도 실제 데이터 교환은 이루어지지 않는다. 이는 "분명히 `ros2 topic list`에는 토픽이 보이는데, `ros2 topic echo`로는 데이터가 수신되지 않는" 현상의 가장 흔한 원인이다.20 ROS 2 노드 측에서는 "New subscription discovered... requesting incompatible QoS"와 같은 경고 메시지가 출력될 수 있다.20
- **필수 일치 항목:** 통신을 위해서는 최소한 다음과 같은 핵심 QoS 정책들이 양측에서 호환 가능하게 설정되어야 한다.
  - **Reliability (신뢰성):** `RELIABLE` (TCP처럼 전송 보장) 또는 `BEST_EFFORT` (UDP처럼 속도 우선).
  - **Durability (내구성):** `VOLATILE` (발행자가 사라지면 데이터도 소멸), `TRANSIENT_LOCAL` (발행자가 사라져도 최신 데이터를 유지하여 나중에 참여하는 구독자에게 전달).
  - **History (이력):** `KEEP_LAST` (최신 N개 데이터만 유지) 또는 `KEEP_ALL` (모든 데이터 유지).
- **확인 방법:** 특정 ROS 2 토픽의 QoS 설정은 `ros2 topic info <topic_name> --verbose` 명령어를 통해 상세히 확인할 수 있다. 순수 DDS 애플리케이션은 이 정보를 바탕으로 자신의 QoS 프로파일을 정확하게 구성해야 한다.20


DDS는 도메인 ID(Domain ID)라는 0 이상의 정수 값을 사용하여 독립적인 가상 네트워크를 구성한다. 오직 동일한 도메인 ID를 가진 DDS 참여자(Participant)들만이 서로를 발견하고 통신할 수 있다.5 ROS 2에서는 

`ROS_DOMAIN_ID` 환경 변수를 통해 이 값을 설정하며, 기본값은 0이다. 따라서 순수 DDS 애플리케이션과 통신하려는 ROS 2 시스템은 반드시 동일한 도메인 ID를 사용하도록 설정되어야 한다.


상호운용성을 달성하는 구체적인 방법은 사용하는 DDS 벤더와 그에 해당하는 RMW 구현체에 따라 미묘한 차이가 있다. 벤더 선택은 단순히 성능이나 라이선스 문제를 넘어, 상호운용성 구현의 난이도와 가용한 지원 수준에 직접적인 영향을 미치는 중요한 아키텍처 결정이다.


eProsima Fast DDS는 ROS 2의 대부분 LTS(Long-Term Support) 릴리즈에서 기본 미들웨어로 채택되어 있어, 가장 일반적인 상호운용 시나리오에 해당한다.13

- **핵심 도구:** `Fast-DDS-Gen`은 `.idl` 파일로부터 C++ 타입 지원 소스 코드와 CMake 빌드 파일을 생성하는 필수 명령줄 도구이다.9 이 도구를 사용하여 ROS 2 메시지에 해당하는 코드를 생성할 수 있다.
- **구현 절차:** OSRF(Open Source Robotics Foundation)에서 제공하는 공식 예제인 `ros2_raw_dds_example` 저장소는 Fast DDS를 이용한 상호운용성의 가장 좋은 학습 자료이다.21 구현 절차는 다음과 같다.
  1. ROS 2 설치 경로에서 상호운용하고자 하는 메시지에 해당하는 `.idl` 파일을 찾는다. (예: `/opt/ros/[distro]/share/[pkg_name]/msg/dds_connext/`)
  2. `Fast-DDS-Gen`을 실행하여 `.idl` 파일로부터 C++ 코드를 생성한다.
  3. 생성된 Publisher/Subscriber 소스 코드(`*Publisher.cxx`, `*Subscriber.cxx`)를 열어, `create_topic` 함수 호출부의 토픽 이름을 III-A절에서 설명한 규칙에 따라 수정한다 (예: `rt/chatter`).
  4. 생성된 타입 지원 소스 코드(`*PubSubTypes.cxx`)를 열어, `setName` 함수 호출부의 타입 이름을 III-B절의 규칙에 따라 수정한다 (예: `std_msgs::msg::dds_::String_`).
  5. ROS 2 노드와 동일한 QoS 프로파일(Reliability, Durability 등)을 사용하여 DDS Publisher/Subscriber를 생성한다.
- **참고:** 최근 Fast DDS 공식 문서에서도 ROS 2와의 상호운용성에 대한 가이드를 보강하고 있어, 이를 참조하는 것이 좋다.28


Cyclone DDS는 뛰어난 성능과 안정성으로 널리 알려져 있으며, ROS 2 Galactic 배포판의 기본 RMW로 채택된 바 있다.29 Autoware, Nav2 등 주요 자율주행 및 내비게이션 프로젝트에서 선호되는 구현체이다.29

- **핵심 특징:** Cyclone DDS의 RMW(`rmw_cyclonedds_cpp`)는 DDS-XTypes의 모든 기능을 활용하기보다는, 호환성을 위해 더 단순한 CDR(Common Data Representation) 직렬화 방식을 사용한다.25 이로 인해 IDL 해석에서 다른 벤더와 미묘한 차이가 발생하며, 이것이 상호운용성의 핵심적인 '함정'이 될 수 있다.
- **구현 절차:** 커뮤니티의 논의를 통해 해결된 문제들을 중심으로 절차를 이해해야 한다.20
  1. 가장 중요한 것은 IDL 파일을 정의할 때, 데이터 구조체(struct)에 `@appendable` 어노테이션 대신 `@final` 어노테이션을 사용하는 것이다.20 이는 해당 타입이 확장되지 않는다는 것을 명시하여, RMW가 사용하는 단순 CDR 방식과 호환되도록 만든다.
  2. 토픽 이름 변환 규칙(`rt/` 접두사 등)은 다른 벤더와 동일하게 적용된다.
  3. QoS 설정 불일치는 여전히 주요 실패 원인이므로, ROS 2 노드의 설정을 정확히 복제해야 한다.


RTI Connext DDS는 상용 등급의 DDS 구현체로, 최고의 성능과 신뢰성이 요구되는 미션 크리티컬 시스템에 널리 사용된다. RTI는 ROS 2와의 상호운용성을 매우 중요하게 여기며, 이를 위한 체계적인 지원을 제공한다.3

- **핵심 도구 및 특징:**
  - **풍부한 예제 및 라이브러리:** RTI는 상호운용성을 시연하는 전용 GitHub 저장소(`ros2-interop-demos`)와 ROS 2 노드 내에서 DDS API를 더 쉽게 사용할 수 있도록 돕는 헬퍼 라이브러리(`rticonnextdds-robot-helpers`)를 제공한다.32
  - **사전 변환된 IDL:** RTI Community GitHub에서는 모든 표준 ROS 2 메시지에 대해 미리 `.idl`로 변환된 파일 세트를 제공하여 개발자가 직접 변환하는 수고를 덜어준다.8
  - **강력한 디버깅 도구:** `RTI Admin Console`이라는 그래픽 도구를 통해 네트워크상의 모든 DDS 참여자, 토픽, 데이터 흐름, QoS 프로파일 등을 실시간으로 시각화하고, QoS 불일치와 같은 문제를 매우 직관적으로 찾아낼 수 있다.34
- **구현 절차:**
  1. RTI에서 제공하는 사전 변환된 IDL 세트를 사용하는 것이 가장 편리하고 안전하다.
  2. `rmw_connextdds` 문서에 명시된 이름 변환 규칙(`rt/`, `rq/`, `rr/` 등)을 따른다.23
  3. `USER_QOS_PROFILES.xml`이라는 외부 XML 파일을 통해 QoS 설정을 세밀하게 제어할 수 있다. 이 파일에 ROS 2와 호환되는 QoS 프로파일을 정의하고 애플리케이션 실행 시 로드되도록 설정한다.34


| 설정 항목                 | eProsima Fast DDS                | Eclipse Cyclone DDS                   | RTI Connext DDS                                  |
| ------------------------- | -------------------------------- | ------------------------------------- | ------------------------------------------------ |
| **대표 RMW 패키지**       | `rmw_fastrtps_cpp`               | `rmw_cyclonedds_cpp`                  | `rmw_connextdds`                                 |
| **핵심 도구**             | `Fast-DDS-Gen`                   | `cyclonedds` IDL 컴파일러             | `RTI Admin Console`, `rtiddsgen`                 |
| **토픽 이름 규칙**        | `rt/`, `rq/`, `rr/` 접두사       | `rt/`, `rq/`, `rr/` 접두사            | `rt/`, `rq/`, `rr/` 접두사                       |
| **타입 이름 규칙**        | `<pkg>::msg::dds_::<type>_`      | `<pkg>::msg::dds_::<type>_`           | `<pkg>::msg::dds_::<type>_`                      |
| **주요 IDL/QoS 고려사항** | ROS 2 QoS 프로파일과 정확히 일치 | IDL struct에 `@final` 어노테이션 사용 | `USER_QOS_PROFILES.xml` 사용, 사전 변환 IDL 활용 |
| **대표 예제 저장소**      | `osrf/ros2_raw_dds_example`      | 커뮤니티 포럼/이슈 기반               | `rticommunity/ros2-interop-demos`                |


이론적 지식을 바탕으로, 실제 상호운용성을 구현하고 문제를 해결하는 구체적인 방법을 알아본다.


가장 기본적인 `std_msgs/msg/String` 메시지를 사용하여 ROS 2 `talker` 노드와 순수 Fast DDS `subscriber` 애플리케이션을 연동하는 과정을 단계별로 시연한다.

- **1단계: ROS 2 환경 준비**

  - 터미널을 열고 ROS 2 환경을 소싱한 뒤, `demo_nodes_cpp` 패키지의 `talker` 노드를 실행한다. 이 노드는 `/chatter` 토픽에 "Hello World" 메시지를 주기적으로 발행한다.
  - `$ ros2 run demo_nodes_cpp talker`

- **2단계: IDL 파일 준비**

  - `std_msgs/msg/String` 메시지에 해당하는 IDL 파일을 준비한다. 내용은 다음과 같아야 한다.22

    Code snippet

    ```
    // String_.idl
    module std_msgs {
      module msg {
        module dds_ {
          @final struct String_ {
            string data;
          };
        };
      };
    };
    ```

  - 이 파일 구조와 이름(`String_`)은 RMW의 변환 규칙을 따른 것이다.

- **3단계: 순수 DDS 코드 생성**

  - `Fast-DDS-Gen`을 사용하여 위 IDL 파일로부터 Subscriber 관련 C++ 소스 코드를 생성한다.
  - `$ fastddsgen -replace -d. String_.idl`

- **4단계: 코드 수정**

  - 생성된 `String_Subscriber.cxx` 파일을 열어 `m_topic`을 초기화하는 부분을 찾는다. 토픽 이름을 `rt/chatter`로 수정한다.
  - 마찬가지로 타입 이름을 `std_msgs::msg::dds_::String_`으로 설정한다.
  - `talker` 노드의 기본 QoS(Reliable, Transient Local)와 일치하도록 Subscriber의 QoS 프로파일을 설정한다.

- **5단계: 컴파일 및 실행**

  - 생성되고 수정된 파일들을 포함하는 순수 DDS 애플리케이션을 CMake 등을 사용하여 빌드한다.
  - 빌드된 Subscriber 애플리케이션을 실행한다. 이때, ROS 2와 동일한 `ROS_DOMAIN_ID` 환경 변수가 설정되어 있어야 한다.

- **6단계: 결과 확인**

  - Subscriber 애플리케이션의 콘솔 창에 ROS 2 `talker`가 보낸 "Hello World: [count]" 메시지가 출력되는 것을 확인한다.


상호운용성 문제 발생 시, 무작위로 시도하기보다 다음의 체계적인 흐름에 따라 문제를 진단하는 것이 효율적이다.

- **1단계 - 디스커버리 확인 (네트워크 연결)**
  - **증상:** `ros2 topic list`에 원하는 토픽이 전혀 나타나지 않는다.
  - **점검 사항:**
    1. **도메인 ID:** 모든 ROS 2 노드와 순수 DDS 애플리케이션이 동일한 `ROS_DOMAIN_ID`를 사용하고 있는지 확인한다.
    2. **네트워크 연결:** 물리적인 네트워크 연결(케이블, Wi-Fi)이 정상적인지, IP 주소가 올바른 서브넷에 속해 있는지 확인한다.
    3. **방화벽/멀티캐스트:** 로컬 방화벽이나 기업/대학 네트워크 정책이 UDP 멀티캐스트 패킷(기본 디스커버리 방식)을 차단하고 있지 않은지 확인한다.15 문제가 의심될 경우, DDS의 Discovery Server 기능이나 Initial Peers 목록(유니캐스트 디스커버리)을 사용하도록 설정을 변경하는 것을 고려한다.9
- **2단계 - 토픽 및 타입 매칭 확인 (이름 규칙)**
  - **증상:** `ros2 topic list`에는 토픽이 보이지만, `ros2 topic echo` 실행 시 "invalid message type" 오류가 발생하거나 아무런 반응이 없다.
  - **점검 사항:**
    1. **토픽 이름:** 순수 DDS 애플리케이션에서 사용한 토픽 이름에 `rt/`, `rq/`, `rr/` 등의 접두사가 정확히 적용되었는지 코드 레벨에서 다시 확인한다.21
    2. **타입 이름:** DDS 타입 등록 시 사용한 이름이 `<pkg>::msg::dds_::<type>_` 규칙을 정확히 따르는지 확인한다.21
    3. **IDL 구조:** 사용한 `.idl` 파일의 모듈 구조(`module pkg { module msg { module dds_ {... } } }`)와 필드 정의가 ROS 2의 것과 100% 일치하는지 비교한다.
- **3단계 - QoS 호환성 확인 (정책 일치)**
  - **증상:** 모든 이름이 맞는 것 같고 `ros2 topic echo`도 오류 없이 실행되지만, 데이터가 전혀 수신되지 않는다. ROS 2 노드 측 터미널에 "incompatible QoS" 경고가 출력될 수 있다.20
  - **점검 사항:**
    1. **QoS 프로파일 확인:** `ros2 topic info <topic_name> --verbose` 명령으로 ROS 2 측의 정확한 QoS 설정을 확인한다.20
    2. **QoS 정책 일치:** 순수 DDS 애플리케이션의 `Reliability`, `Durability`, `History`, `Deadline` 등 주요 QoS 정책을 위에서 확인한 값과 호환되도록 정확히 일치시킨다.
    3. **벤더별 특성:** Cyclone DDS의 경우, IDL에 `@final` 어노테이션이 올바르게 사용되었는지 다시 한번 확인한다.25
- **유용한 도구:**
  - **ROS 2 CLI:** `ros2 topic list/info/echo`, `ros2 node info` 등은 기본적인 상태 확인에 필수적이다.
  - **Wireshark:** DDSI-RTPS 프로토콜 분석 플러그인을 설치하면, 네트워크를 오가는 실제 DDS 패킷(Discovery 정보, 데이터 등)을 직접 들여다볼 수 있어 저수준 문제 해결에 매우 유용하다.29
  - **벤더별 GUI 도구:** RTI Admin Console 35, eProsima Fast DDS Monitor 27 등은 네트워크 토폴로지, 데이터 흐름, QoS 프로파일 등을 시각적으로 보여주어 복잡한 시스템의 문제를 직관적으로 진단하는 데 큰 도움이 된다.



본 보고서를 통해 도출된 핵심 결론은 다음과 같다.

1. **상호운용성은 가능하다:** ROS 2와 순수 DDS 애플리케이션 간의 직접적인 데이터 교환은 가능하며, 이는 ROS 2가 산업 표준인 DDS를 채택한 덕분에 의도된 기능이다.
2. **성공의 열쇠는 '정확한 복제'이다:** 성공적인 연동은 ROS 2의 RMW 계층이 내부적으로 수행하는 변환 규칙을 순수 DDS 애플리케이션에서 얼마나 정확하게 복제하느냐에 달려있다.
3. **네 가지 핵심 요소:** 개발자는 **(1) DDS 도메인 ID, (2) 변환된 토픽 이름, (3) 변환된 데이터 타입(이름 및 구조), (4) 호환되는 QoS 정책**이라는 네 가지 핵심 요소를 반드시 일치시켜야 한다.
4. **벤더별 차이 존재:** 구체적인 구현 방법과 사용 도구는 eProsima Fast DDS, Eclipse Cyclone DDS, RTI Connext DDS 등 사용하는 DDS 벤더에 따라 다르므로, 각 벤더의 특성과 문서를 숙지해야 한다.


순수 DDS 연동은 강력한 기능이지만, 모든 상황에 적합한 만능 해결책은 아니다. 프로젝트의 아키텍처를 결정할 때 다음 사항들을 전략적으로 고려해야 한다.

- **언제 순수 DDS 연동을 고려해야 하는가?**

  - **최고 수준의 성능 요구:** RMW의 미세한 오버헤드조차 허용되지 않는 초저지연(ultra-low latency) 또는 초고처리량(very-high throughput) 시스템을 구축할 때.
  - **고급 DDS 기능 활용:** ROS 2 API가 표준적으로 노출하지 않는 DDS의 고급 기능(예: 콘텐츠 필터링, 시간 기반 필터링, 영속성(Persistence) 서비스 등)이 반드시 필요할 때.
  - **비-ROS 시스템 통합:** ROS 2 클라이언트 라이브러리가 지원되지 않는 언어(예: Rust 네이티브 37)나 특수 임베디드 플랫폼으로 작성된 기존 시스템과 ROS 2 로봇을 통합해야 할 때.

- **트레이드오프 분석:** 순수 DDS 연동은 최고의 성능과 유연성을 제공하는 대신, ROS 2가 제공하는 편리한 추상화(RCL API, `colcon` 빌드 시스템, 파라미터 서버 등)를 포기해야 하는 대가가 따른다.5 이는 개발의 복잡성을 높이고, 장기적인 유지보수 비용을 증가시킬 수 있다.

- **보안(Security) 고려:** 보안이 활성화된 ROS 2 시스템(SROS 2)과 통신하기 위해서는, 순수 DDS 애플리케이션 또한 DDS-Security 표준에 따라 적절한 인증서(CA), 거버넌스 파일, 권한 파일을 발급받고 설정해야 한다.38 이는 상호운용성의 또 다른 중요한 차원으로, 추가적인 설정과 관리가 필요하다.

- 최종 권장:

  대부분의 일반적인 로봇 애플리케이션에서는 ROS 2의 표준 API(rclcpp, rclpy)를 사용하는 것이 생산성, 안정성, 커뮤니티 지원 측면에서 훨씬 유리하다. 순수 DDS 연동은 시스템의 성능 병목 현상이 명확하게 식별되었거나, 특정 고급 DDS 기능이 비즈니스 요구사항을 충족하기 위해 반드시 필요한 경우에 한해 신중하게 접근해야 할 고급 아키텍처 패턴이다. 프로젝트 초기 단계에서 요구사항을 명확히 분석하고, 얻는 이점과 발생하는 비용을 신중하게 저울질하여 최적의 접근 방식을 결정하는 것이 중요하다.


1. ROS2와 DDS란? - 개발하는 핑구 - 티스토리, accessed July 2, 2025, [https://ai-sinq.tistory.com/entry/ROS2%EC%99%80-DDS%EB%9E%80](https://ai-sinq.tistory.com/entry/ROS2와-DDS란)
2. Robot Operating System 2 (ROS 2) Architecture | by Huseyin Kutluca - Medium, accessed July 2, 2025, https://medium.com/software-architecture-foundations/robot-operating-system-2-ros-2-architecture-731ef1867776
3. ROS 2: What is DDS | Data Distribution Service (DDS) Community RTI Connext Users, accessed July 2, 2025, https://community.rti.com/page/ros-2-what-dds
4. DDS implementations - ROS 2 Documentation: Foxy documentation, accessed July 2, 2025, https://docs.ros.org/en/foxy/Installation/DDS-Implementations.html
5. robotics - What's the difference between ROS2 and DDS? - Stack Overflow, accessed July 2, 2025, https://stackoverflow.com/questions/51187676/whats-the-difference-between-ros2-and-dds
6. ROS 2 Architecture Overview - Automatic Addison, accessed July 2, 2025, https://automaticaddison.com/ros-2-architecture-overview/
7. [ROS2] ROS 2의 DDS와 QoS - velog, accessed July 2, 2025, [https://velog.io/@katinon/ROS2-ROS-2%EC%9D%98-DDS%EC%99%80-QoS](https://velog.io/@katinon/ROS2-ROS-2의-DDS와-QoS)
8. ROS 2 and DDS: Interoperability Drives Next-Generation Robotics, accessed July 2, 2025, https://www.rti.com/blog/ros-2-and-dds-interoperability-drives-next-generation-robotics
9. Fast DDS - eProsima, accessed July 2, 2025, https://fast-dds.docs.eprosima.com/
10. Internal ROS 2 interfaces - ROS 2 Documentation: Rolling ..., accessed July 2, 2025, https://docs.ros.org/en/rolling/Concepts/Advanced/About-Internal-Interfaces.html
11. ROS 2 middleware interface - ROS2 Design, accessed July 2, 2025, https://design.ros2.org/articles/ros_middleware_interface.html
12. rmw: ROS Middleware Abstraction Interface, accessed July 2, 2025, https://docs.ros2.org/foxy/api/rmw/
13. DDS implementations - ROS 2 Documentation: Iron documentation, accessed July 2, 2025, https://docs.ros.org/en/iron/Installation/DDS-Implementations.html
14. Switching Between ROS Middleware Implementations - MATLAB & Simulink - MathWorks, accessed July 2, 2025, https://www.mathworks.com/help/ros/ug/switching-between-ros-middleware-implementations.html
15. ROS 2 Middleware (RMW) Configuration - GitHub Pages, accessed July 2, 2025, https://iroboteducation.github.io/create3_docs/setup/xml-config/
16. ROS2 part 4 - ROS2 IPC, DDS, Topics, Services, Actions and Interfaces - RoboticsUnveiled, accessed July 2, 2025, https://www.roboticsunveiled.com/ros2-ipc-dds-topics-services-actions-interfaces/
17. About ROS 2 middleware implementations, accessed July 2, 2025, https://docs.ros.org/en/foxy/Concepts/About-Middleware-Implementations.html
18. README.md - ros2/rosidl - GitHub, accessed July 2, 2025, https://github.com/ros2/rosidl/blob/rolling/README.md
19. 16. ROS 2 using Fast DDS middleware - 3.2.2, accessed July 2, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/ros2/ros2.html
20. ROS2 interoperability with native DDS - ROS Answers archive, accessed July 2, 2025, https://answers.ros.org/question/404995/
21. osrf/ros2_raw_dds_example: A project showing how to connect a raw DDS program to a ROS 2 graph - GitHub, accessed July 2, 2025, https://github.com/osrf/ros2_raw_dds_example
22. ROS2 interoperability with native DDS - ROS Answers archive, accessed July 2, 2025, https://answers.ros.org/question/404995/ros2-interoperability-with-native-dds/
23. ros2/rmw_connextdds: ROS 2 RMW layer for RTI Connext DDS Professional and RTI Connext DDS Micro. - GitHub, accessed July 2, 2025, https://github.com/ros2/rmw_connextdds
24. Topic and Service name mapping to DDS - ROS2 Design, accessed July 2, 2025, https://design.ros2.org/articles/topic_and_service_names.html
25. Communication between Cyclone and ROS2 / Issue #1412 - GitHub, accessed July 2, 2025, https://github.com/eclipse-cyclonedds/cyclonedds/issues/1412
26. [ROS2]OMG Data-Distribution Service: Architectural Overview and OMG Data-Distribution Service: Architectural Update - velog, accessed July 2, 2025, https://velog.io/@protocol5/ROS2OMG-Data-Distribution-Service-Architectural-Overview-and-OMG-Data-Distribution-Service-Architectural-Update
27. eProsima/Fast-DDS: The most complete DDS - Proven: Plenty of success cases. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 2, 2025, https://github.com/eProsima/Fast-DDS
28. Identify a FastDDS example topic on ROS2 [13775] / eProsima Fast-DDS / Discussion #2467 - GitHub, accessed July 2, 2025, https://github.com/eProsima/Fast-DDS/discussions/2467
29. 2021 Eclipse Cyclone DDS ROS Middleware Evaluation Report with iceoryx and Zenoh, accessed July 2, 2025, https://osrf.github.io/TSC-RMW-Reports/humble/eclipse-cyclonedds-report.html
30. Eclipse Cyclone DDS - ROS 2 Documentation: Galactic documentation, accessed July 2, 2025, https://docs.ros.org/en/galactic/Installation/DDS-Implementations/Working-with-Eclipse-CycloneDDS.html
31. RTI Connext DDS - ROS 2 Documentation: Humble documentation, accessed July 2, 2025, https://docs.ros.org/en/humble/Installation/RMW-Implementations/DDS-Implementations/Working-with-RTI-Connext-DDS.html
32. neil-rti/ros2-interop-demos: Demonstration apps to show ROS2 interoperability with native DDS apps - GitHub, accessed July 2, 2025, https://github.com/neil-rti/ros2-interop-demos
33. rticommunity/rticonnextdds-robot-helpers: Collection of code and cmake utilities to use RTI Connext DDS with ROS 2 - GitHub, accessed July 2, 2025, https://github.com/rticommunity/rticonnextdds-robot-helpers
34. Setting up the RTI DDS configuration file in ROS2 - Stack Overflow, accessed July 2, 2025, https://stackoverflow.com/questions/52151179/setting-up-the-rti-dds-configuration-file-in-ros2
35. ROS 2 + DDS Interoperation - YouTube, accessed July 2, 2025, https://www.youtube.com/watch?v=GGqcrccWfeE
36. Cybersecurity in the ROS 2 communication middleware, the Data Distribution Service (DDS), accessed July 2, 2025, https://news.aliasrobotics.com/cybersecurity-in-the-ros-2-communication-middleware-targeting-the-top-6-dds-implementations/
37. ROS2 client library based on RustDDS - GitHub, accessed July 2, 2025, https://github.com/Atostek/ros2-client/
38. ROS 2 DDS-Security integration - ROS2 Design, accessed July 2, 2025, https://design.ros2.org/articles/ros2_dds_security.html
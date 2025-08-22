---
layout: page
title: ROS 2와 네이티브 DDS 상호운용성
permalink: /ros2/rmw/ROS 2와 네이티브 DDS 상호운용성
---


본 보고서는 "ROS 2에서 네이티브 DDS(Data Distribution Service) 메시지를 직접 구독하거나 발행할 수 있는가?"라는 질문에 대한 심층적인 기술 분석을 제공합니다. 이 질문에 대한 명확한 답은 '가능하다'입니다. 이러한 기능은 ROS 2의 부가적인 특징이 아니라, DDS 표준 위에 시스템을 구축한 ROS 2의 근본적인 아키텍처 설계에서 비롯된 필연적인 결과입니다.1

그러나 ROS 2와 순수(Native) DDS 애플리케이션 간의 상호운용성을 달성하는 과정은 간단하지 않으며, ROS 2가 DDS 위에 추가한 추상화 계층과 규약에 대한 정확한 이해를 요구합니다. 핵심적인 과제는 두 환경 간의 토픽 이름 지정 규칙, 데이터 타입 정의(IDL), 그리고 서비스 품질(QoS) 프로파일의 차이를 해소하는 것입니다.4

본 보고서의 주요 분석 결과는 다음과 같습니다.

- **ROS 미들웨어(RMW) 계층의 역할:** RMW 계층은 다양한 DDS 벤더를 지원하는 유연성을 제공하는 핵심 요소인 동시에, 성능 오버헤드의 주된 원인이 되기도 합니다.3
- **상호운용성의 핵심 요건:** 성공적인 상호운용성은 토픽 이름 변환 규칙(`rt/`), 타입 이름 변환 규칙(`::msg::dds_::<Type>_`), 그리고 QoS 호환성을 보장하는 세심한 설정에 달려 있습니다.8
- **고급 성능 최적화:** Loaned Message API를 통한 제로카피(Zero-Copy) 통신이나 네이티브 DDS API 직접 접근과 같은 고급 기술은 표준 RMW의 오버헤드를 우회하여, 핵심적인 애플리케이션에서 상당한 성능 향상을 제공합니다.10
- **대규모 시스템 확장성:** 대규모 또는 무선 연결 시스템은 네트워크 트래픽을 관리하고 안정적인 노드 발견(Discovery)을 보장하기 위해 Discovery Server, DDS Router와 같은 특화된 DDS 기능이나 Zenoh와 같은 대체 미들웨어를 통해 큰 이점을 얻을 수 있습니다.12

본 보고서는 시스템 아키텍트와 엔지니어가 이러한 복잡성을 해결할 수 있도록, 기초 이론, 실제 구현 가이드, 성능 최적화 전략, 그리고 배포 권장 사항을 포함하는 포괄적인 전문가 수준의 가이드를 제공하는 것을 목적으로 합니다.

------


이 섹션에서는 ROS 2가 단순히 DDS와 '호환'되는 것이 아니라, 근본적으로 DDS 기반 시스템이라는 기초 이론을 확립합니다.


ROS 1은 중앙 집중형 `roscore`(마스터 노드)를 사용하는 맞춤형 TCP 기반 프로토콜(TCPROS)을 활용했습니다. 이는 다중 로봇 시스템, 실시간 성능 보장, 그리고 불안정한 네트워크 환경에서의 배포에 있어 본질적인 한계를 드러냈습니다.15

ROS 2를 DDS 위에 구축하기로 한 결정은 산업적으로 검증된 DDS의 강력한 기능들을 상속받기 위한 의도적인 아키텍처 선택이었습니다. DDS는 분산형 P2P(Peer-to-Peer) 노드 발견 메커니즘, 실시간 및 신뢰성 있는 통신을 위한 풍부한 서비스 품질(QoS) 정책, 그리고 항공우주 및 금융과 같은 핵심 시스템에서 검증된 견고한 표준 기반을 제공합니다.2 이러한 특징들은 ROS 2를 상용 수준의 복잡한 시스템 배포에 본질적으로 더 적합하게 만듭니다.18


ROS 2의 통신은 여러 추상화 계층으로 구성되어 있으며, 각 계층은 특정 역할을 수행합니다.

- **사용자 대면 API (`rclcpp`, `rclpy`):** 최상위 계층으로, C++과 Python에서 노드, 퍼블리셔, 서브스크라이버와 같은 친숙한 ROS 개념을 제공합니다.19 사용자는 

  `create_publisher()`나 `create_subscription()`과 같은 메서드를 통해 통신 객체를 생성합니다.21

- **ROS 클라이언트 라이브러리 (RCL):** C 언어 API로, ROS 개념(예: 노드 및 그래프 관리)의 공통 로직을 구현하며 미들웨어에 독립적인 형태로 설계되었습니다.20

- **ROS 미들웨어 인터페이스 (RMW):** 핵심적인 추상화 계층입니다. `rmw`는 `rmw_create_publisher`, `rmw_publish`와 같은 미들웨어 기본 기능에 대한 일반적인 C API를 정의합니다. 이 추상화를 통해 ROS 2는 벤더별 세부 사항을 상위 계층에 노출하지 않고도 다양한 DDS 구현을 지원할 수 있습니다.3

- **RMW 구현체 (예: `rmw_fastrtps_cpp`, `rmw_cyclonedds_cpp`):** 일반 RMW API 호출을 특정 DDS 벤더(예: eProsima Fast DDS, Eclipse Cyclone DDS)의 특정 API 호출로 변환하는 얇은 어댑터 또는 "심(shim)" 계층입니다.23 이 계층이 DDS 라이브러리와 직접 상호작용합니다.

- **DDS/RTPS 계층:** RTPS(Real-Time Publish-Subscribe) 프로토콜을 기반으로 직렬화, 노드 발견, 전송 등 실제 유선 상의 통신을 처리하는 기본 미들웨어입니다.1


ROS 2의 통신 개념은 DDS의 엔티티와 직접적으로 매핑됩니다.

- **ROS 노드 (Node):** DDS의 `DomainParticipant`에 개념적으로 매핑됩니다. Participant는 DDS 통신 도메인으로의 진입점 역할을 합니다.27 기본적으로 각 ROS 2 프로세스는 단일 DDS Participant를 생성합니다.
- **ROS 토픽/서비스/액션:** 이들은 하나 이상의 기본 DDS `Topic`에 매핑됩니다. DDS 토픽은 이름, 데이터 타입, 그리고 연관된 QoS 정책의 고유한 조합입니다.1
- **ROS 퍼블리셔/서브스크라이버:** 각각 DDS의 `DataWriter`와 `DataReader` 엔티티에 매핑됩니다. `DataWriter`는 토픽에 데이터를 보내는 역할을, `DataReader`는 데이터를 수신하는 역할을 담당합니다.3

이러한 아키텍처는 ROS 2 애플리케이션이 본질적으로 DDS 애플리케이션임을 의미하며, 이는 네이티브 DDS와의 상호운용성의 기반이 됩니다. 하지만 이 구조는 동시에 몇 가지 중요한 기술적 함의를 내포합니다.

첫째, RMW는 필연적으로 "새는 추상화(leaky abstraction)"가 됩니다. RMW의 공식적인 목표는 미들웨어에 독립적이고 DDS의 복잡성을 숨기는 것입니다.3 그러나 고급 QoS, 전송 계층 선택, 보안과 같은 강력한 DDS 기능들은 실제 애플리케이션에서 매우 중요하며, 이는 대부분 벤더에 특화되어 있습니다.4 이러한 기능들을 활용하기 위해, 개발자들은 종종 RMW 추상화를 우회하여 XML 파일이나 환경 변수를 통해 기본 DDS 벤더를 직접 설정해야 합니다.4 더욱이, 네이티브 DDS와의 상호운용성을 위해서는 이름 변환(name mangling)과 같은 ROS 2의 특정 DDS 구현 세부 사항을 알아야만 합니다.4 이는 근본적인 긴장 관계를 만듭니다. 즉, RMW는 기능 접근성을 희생하여 이식성을 제공하지만, 실제 배포 환경에서는 개발자가 추상화를 깨고 벤더별 세부 사항을 다루도록 강요받는 경우가 많습니다. 이러한 '누수'는 결함이라기보다는 사용 편의성과 기본 미들웨어의 강력한 기능 사이의 균형을 맞추기 위한 필연적인 절충안입니다.34

둘째, ROS 2의 아키텍처는 기본적으로 "이중 직렬화(double serialization)" 오버헤드를 수반합니다. ROS 2 애플리케이션은 C++ 구조체나 Python 클래스와 같은 언어별 메시지 객체를 다룹니다.21

`publish()`가 호출되면 데이터는 클라이언트 라이브러리(예: `rclcpp`)에서 RMW 계층으로 흐릅니다.3 RMW 구현체는 이 ROS 데이터 구조를 기본 DDS 벤더가 이해할 수 있는 형식으로 변환해야 하는데, 이 과정에서 종종 일반 바이트 버퍼로의 직렬화 단계가 포함됩니다.3 그 후 DDS 미들웨어는 이 버퍼를 받아 다시 유선 상의 RTPS 형식으로 자체 직렬화를 수행합니다.6 이러한 2단계 프로세스(ROS 객체 -> 중간 버퍼 -> RTPS 유선 형식)는 네이티브 DDS 애플리케이션이 데이터 객체에서 유선 형식으로 직접 직렬화하는 것에 비해 본질적으로 지연 시간과 CPU 오버헤드를 발생시킵니다. 이 아키텍처적 오버헤드는 첫 번째 복사를 제거하려는 Loaned Messages API와 같은 고급 기능의 주된 동기가 됩니다.7

------


이 섹션은 통신을 성공시키기 위한 실행 가능한 단계를 제공하는 핵심적인 "how-to" 매뉴얼입니다.


네이티브 DDS 애플리케이션과 ROS 2 애플리케이션은 동일한 DDS 벤더와 토픽 이름을 사용하더라도 기본적으로 통신하지 않습니다.4 이는 상호운용성이 자동으로 이루어지지 않음을 의미합니다.

RMW 계층은 DDS 전역 데이터 공간 내에 ROS 전용 네임스페이스를 생성하기 위해 의도적으로 토픽 이름과 타입 이름을 '변환(mangle)'하거나 수정합니다. 이는 의도치 않은 교차 통신을 방지하지만, 의도적인 통신을 위해서는 명시적인 설정이 필요합니다.8 성공적인 상호운용을 위해서는 **토픽 이름**, **데이터 타입**, 그리고 **QoS 프로파일**이라는 세 가지 핵심 요소를 일치시켜야 합니다.


ROS 2는 사용자 대면 토픽 이름에 특정 접두사를 추가하여 최종 DDS 토픽 이름을 생성합니다. 이는 초보자들이 가장 흔하게 겪는 실패 지점입니다.4

- **표준 토픽 (Pub/Sub):** `rt/` 접두사가 붙습니다. ROS 2 토픽 `/chatter`는 DDS 토픽 `rt/chatter`가 됩니다.4
- **서비스 (Services):** 서비스는 두 개의 DDS 토픽을 사용하여 구현됩니다. ROS 2 서비스 `/add_two_ints`는 요청을 위한 `rq/add_two_ints`와 응답을 위한 `rr/add_two_ints`라는 두 개의 DDS 토픽으로 변환됩니다.33
- **액션 (Actions):** 액션은 목표, 피드백, 결과, 상태 등을 위해 여러 토픽을 사용하며, 모두 특정 접두사를 가집니다.37

실제 구현 시, 네이티브 DDS 애플리케이션은 ROS 2와 통신하기 위해 변환된 토픽 이름(예: `rt/chatter`)으로 `DataWriter` 또는 `DataReader`를 생성해야 합니다.


- **IDL을 기준으로:** ROS 2는 인터페이스 정의를 위해 `.msg`, `.srv`, `.action` 파일을 사용하지만, 이들은 빌드 과정에서 궁극적으로 OMG의 IDL(Interface Definition Language) 파일로 변환됩니다.25
- **ROS 2 IDL 서브셋:** ROS 2는 전체 IDL 사양의 일부만 지원합니다. 특히, `.msg` 정의에서 `enum`이나 `inheritance`와 같은 기능은 지원하지 않으므로, 이러한 기능을 사용하는 레거시 DDS 시스템과 연동할 때 문제가 될 수 있습니다.6 지원되는 서브셋은 관련 설계 문서에 명시되어 있습니다.41
- **타입 이름 변환:** 토픽 이름과 유사하게, ROS 2는 타입 이름도 변환합니다. ROS 2 메시지 `std_msgs/msg/String`은 DDS 타입 이름 `std_msgs::msg::dds_::String_`이 됩니다.9 이 특정 형식을 네이티브 DDS Participant에 등록해야 합니다.
- **IDL 생성 방법:**
  1. ROS 2는 모든 메시지 패키지에 대해 `.idl` 파일을 자동으로 생성합니다 (예: `/opt/ros/<distro>/share/<pkg_name>/msg/dds_connext/` 경로에서 찾을 수 있음).
  2. 사용자 정의 메시지의 경우, `rosidl_adapter` 패키지의 `msg2idl.py` 스크립트를 사용하여 `.msg` 파일을 `.idl`로 수동 변환할 수 있습니다.43
  3. 네이티브 DDS 애플리케이션은 이와 정확히 동일한 IDL 정의를 사용해야 합니다. 벤더별 코드 생성 도구(예: Connext의 `rtiddsgen`, Fast DDS의 `fastddsgen`)를 사용하여 이 IDL로부터 필요한 C++ 또는 다른 언어의 소스 파일을 생성합니다.9


- **통신의 계약:** DDS의 QoS 정책은 `DataWriter`와 `DataReader` 간의 '계약'을 형성합니다. 연결이 성립되려면 정책들이 호환되어야 합니다. 일반적으로 Writer가 제공하는 정책은 Reader가 요청하는 정책보다 크거나 같아야 합니다.15
- **일반적인 불일치:** `ros2 topic list`에서는 토픽이 보이지만 `ros2 topic echo`에서는 데이터가 보이지 않는 "조용한 실패"의 흔한 원인입니다. 주요 불일치 항목은 다음과 같습니다.
  - **신뢰성 (Reliability):** `RELIABLE` 대 `BEST_EFFORT`. 신뢰성 있는 Writer는 최선형 Reader에게 데이터를 공급할 수 있지만, 그 반대는 불가능합니다.
  - **지속성 (Durability):** `VOLATILE` 대 `TRANSIENT_LOCAL`. `TRANSIENT_LOCAL` Writer는 신뢰성 종류도 일치하지 않는 한 `VOLATILE` Reader와 연결되지 않을 수 있습니다. 이는 미묘하지만 중요한 지점입니다.
  - **데이터 표현 (Data Representation, X-Types):** CycloneDDS 상호운용성에서 발견된 핵심 문제입니다. ROS 2 RMW는 더 단순한 CDR 인코딩을 사용할 수 있지만, 네이티브 애플리케이션은 기본적으로 더 복잡한 XCDR2 인코딩을 사용하여 불일치를 유발할 수 있습니다. 해결책은 네이티브 IDL이 더 단순한 인코딩을 강제하기 위해 `@appendable` 대신 `@final` 확장성을 사용하도록 하는 것입니다.5
- **QoS 디버깅:** `ros2 topic info -v`와 같은 도구는 일부 QoS 정보를 제공할 수 있지만, RTI Admin Console과 같은 벤더별 도구는 정확한 불일치를 진단하는 데 훨씬 더 강력합니다.46


1. **IDL 획득:** 대상 ROS 2 메시지에 대한 `.idl` 파일을 찾거나 생성합니다 (예: `unique_identifier_msgs/msg/UUID.idl`).
2. **DDS 코드 생성:** `fastddsgen`을 사용하여 IDL로부터 C++ 소스 파일을 생성합니다.
3. **생성된 코드 수정:**
   - `UUIDPubSubTypes.cxx` 파일에서 `setName()` 호출을 찾아 타입 이름을 `unique_identifier_msgs::msg::dds_::UUID_`로 변경합니다.
   - `UUIDPublisher.cxx` 파일에서 `create_topic()` 호출을 찾아 토픽 이름을 `rt/uuid_topic`으로 변경합니다.
4. **메인 애플리케이션 작성:** 생성된 헤더를 포함하고, `DomainParticipant`를 생성하며, 수정된 `UUIDPublisher`의 인스턴스를 만들어 루프 안에서 `publish()` 메서드를 호출하는 `main.cpp`를 작성합니다.
5. **컴파일 및 링크:** CMake를 사용하여 Fast DDS 라이브러리에 대해 링크하며 애플리케이션을 컴파일합니다.
6. **실행:** 네이티브 퍼블리셔를 실행합니다. 별도의 터미널에서 `ros2 topic echo /uuid_topic`을 실행하여 통신을 확인합니다.

다음 표는 ROS 2와 DDS 간의 이름 변환 규칙을 요약한 것으로, 상호운용성 구현 시 중요한 참조 자료가 될 것입니다.

| ROS 2 개념                                 | 사용자 대면 이름 (예시) | 기본 DDS 엔티티      | DDS 이름/타입 (결과)           | 비고                           |
| ------------------------------------------ | ----------------------- | -------------------- | ------------------------------ | ------------------------------ |
| 토픽 (Pub/Sub)                             | `/chatter`              | DDS Topic            | `rt/chatter`                   | 가장 일반적인 통신 패턴 8      |
| 서비스 (요청)                              | `/add_two_ints`         | DDS Topic (Request)  | `rq/add_two_ints`              | 요청/응답 패턴의 요청 부분 33  |
| 서비스 (응답)                              | `/add_two_ints`         | DDS Topic (Reply)    | `rr/add_two_ints`              | 요청/응답 패턴의 응답 부분 33  |
| 액션 (목표)                                | `/_action/goal`         | DDS Topic (Goal)     | `ga/_action/goal`              | 장기 실행 태스크의 목표 전송   |
| 액션 (결과)                                | `/_action/result`       | DDS Topic (Result)   | `ar/_action/result`            | 장기 실행 태스크의 최종 결과   |
| 액션 (피드백)                              | `/_action/feedback`     | DDS Topic (Feedback) | `af/_action/feedback`          | 장기 실행 태스크의 중간 피드백 |
| 데이터 타입                                | `std_msgs/msg/String`   | DDS Type             | `std_msgs::msg::dds_::String_` | 타입 호환성을 위해 필수적 9    |
| **표 1: ROS 2에서 DDS로의 이름 변환 규칙** |                         |                      |                                |                                |

------


이 섹션에서는 가장 널리 사용되는 세 가지 RMW 구현체의 실제적인 세부 사항을 다루며, RMW의 '추상화'가 실제 구현에 미치는 구체적인 영향을 분석합니다.


- **현황:** Humble, Iron 등 대부분의 ROS 2 배포판에서 기본 RMW로 사용됩니다.49
- **구성:** 주로 XML 파일을 통해 구성됩니다. 이 파일의 경로는 `FASTRTPS_DEFAULT_PROFILES_FILE` 환경 변수로 지정됩니다.33
- **상호운용성:**
  - 표준 토픽(`rt/`) 및 타입(`...::dds_::..._`) 변환 규칙을 따릅니다.9
  - 토픽 이름으로 프로파일 이름을 지정하여 ROS 2 노드의 QoS와 일치하는 XML 프로파일을 생성할 수 있습니다(예: `<publisher profile_name="rt/chatter">` 프로파일은 해당 토픽의 퍼블리셔에 적용됨).33
  - Fast DDS는 Discovery Server, 대용량 데이터용 TCP 전송, 사용자 정의 메모리 관리 등 XML을 통해 구성할 수 있는 고급 기능을 제공하며, 이는 ROS 2 노드에서도 활용 가능합니다.28
- **도구:** `eProsima Fast DDS Monitor`는 DDS 네트워크 시각화, 엔티티 검사, 통계 데이터 분석을 위한 그래픽 도구로, ROS 2/DDS 상호작용 디버깅에 매우 유용합니다.55


- **현황:** 상용 등급의 RMW로, 특히 프로덕션 및 미션 크리티컬 시스템에서 널리 사용됩니다.4 평가/대학용 라이선스가 제공되지만, 기본적으로 라이선스가 필요합니다.50
- **구성:** XML 파일을 사용하며, `NDDS_QOS_PROFILES` 환경 변수나 작업 디렉토리의 `USER_QOS_PROFILES.xml` 파일을 통해 로드됩니다.4
- **상호운용성:**
  - 동일한 이름 변환 규칙을 따릅니다.4
  - 과거의 주요 문제점은 RMW가 일반적인 "직렬화된 데이터" 타입 코드를 주입하여 특정 데이터 타입을 사용하는 네이티브 애플리케이션과 QoS 불일치를 일으키는 것이었습니다. 해결책은 타입 코드 공지를 억제하는 QoS 프로파일을 ROS 2 Participant에 추가하는 것입니다.4
  - RTI는 ROS 2 C++ 노드 내에서 네이티브 Connext `DomainParticipant`에 쉽게 접근할 수 있도록 전용 헬퍼 라이브러리(`rclcpp_dds`)를 제공하여 강력한 "하이브리드" 애플리케이션 모델을 지원합니다.57
- **도구:** `RTI Administration Console`은 시스템 발견, 토픽 시각화, 실시간 데이터 차트 작성, 그리고 가장 중요한 상세한 QoS 불일치 분석을 위한 매우 진보된 그래픽 도구입니다.47


- **현황:** 고성능 오픈소스 RMW로, ROS 2 Galactic의 기본 RMW였습니다.14 광범위한 튜닝 없이도 많은 수의 토픽/노드에서 우수한 성능을 보이는 것으로 알려져 있습니다.14
- **구성:** `CYCLONEDDS_URI` 환경 변수로 지정된 XML 파일을 통해 구성됩니다 (예: `export CYCLONEDDS_URI=file:///path/to/config.xml`).60
- **상호운용성:**
  - 과거에는 상호운용성이 더 어려웠으나, 커뮤니티 주도 솔션을 통해 문제들이 해결되었습니다.5
  - 주요 문제는 `DataRepresentationQosPolicy` 불일치였습니다. 해결책은 네이티브 애플리케이션용 IDL을 `@appendable` 대신 `@final` 확장성으로 생성하여 호환되는 유선 표현을 강제하는 것입니다.5
  - 동일한 `rt/` 토픽 이름 접두사 규칙을 따릅니다.43
- **도구:** 디버깅을 위해 DDSI-RTPS 디섹터가 포함된 Wireshark와 같은 범용 도구에 더 많이 의존합니다.14


- `RMW_IMPLEMENTATION` 환경 변수는 런타임에 사용할 설치된 RMW를 선택하는 주요 메커니즘입니다.53
- 예시: `RMW_IMPLEMENTATION=rmw_cyclonedds_cpp ros2 run demo_nodes_cpp talker`.62
- **중요한 함정:** 백그라운드에서 실행되는 ROS 2 데몬은 이전에 사용된 RMW를 유지할 수 있습니다. RMW를 전환할 때, 노드와 CLI 도구 간의 통신 문제를 피하기 위해 데몬을 명시적으로 종료(`ros2 daemon stop`)해야 하는 경우가 많습니다.62
- 통신 시스템의 모든 Participant는 호환되는 DDS 벤더 기반의 RMW를 사용해야 합니다. 다른 벤더들이 RTPS 표준을 통해 상호 운용될 수 있지만, 미묘한 차이로 인해 시스템 전체에서 동일한 벤더를 사용하는 것이 더 안정적입니다.53

RMW의 선택은 단순한 기술적 결정이 아니라, 전체 개발 생태계를 선택하는 것과 같습니다. RMW는 환경 변수로 간단히 교체할 수 있는 것처럼 보이지만 62, 실제로는 각 벤더가 제공하는 고유한 도구들(Fast DDS Monitor, RTI Admin Console 등)이 심각한 디버깅과 성능 튜닝에 필수적입니다.56 구성 방식 또한 벤더별 XML 형식과 환경 변수(

`FASTRTPS_DEFAULT_PROFILES_FILE` 대 `CYCLONEDDS_URI`)에 따라 달라집니다.53 또한, 알려진 상호운용성 문제와 해결책도 벤더에 따라 다릅니다(예: Cyclone의 X-Types 문제 대 Connext의 타입 코드 억제).4 따라서 RMW를 선택하는 것은 개발자를 특정 도구, 구성 패러다임, 그리고 커뮤니티 지식 기반에 종속시킵니다. 이는 단순히 전송 계층을 바꾸는 것이 아니라 전체 개발 및 디버깅 워크플로우에 영향을 미치는 결정입니다. 이는 프로덕션 프로젝트의 경우, RMW를 단순히 임의로 교체하는 것이 아니라, 도구, 지원, 성능 프로파일과 같은 요소를 기반으로 초기에 신중하게 선택해야 함을 시사합니다.

| RMW 구현체                      | 기본 배포판         | 라이선스    | 주요 구성 방법 (환경변수 + 파일)       | 주요 디버깅 도구              |
| ------------------------------- | ------------------- | ----------- | -------------------------------------- | ----------------------------- |
| `rmw_fastrtps_cpp`              | Humble, Iron, Jazzy | Apache 2.0  | `FASTRTPS_DEFAULT_PROFILES_FILE` + XML | eProsima Fast DDS Monitor 55  |
| `rmw_connextdds`                | -                   | 상용/평가용 | `NDDS_QOS_PROFILES` + XML              | RTI Administration Console 58 |
| `rmw_cyclonedds_cpp`            | Galactic            | EPL/EDL     | `CYCLONEDDS_URI` + XML                 | Wireshark 14                  |
| **표 2: RMW 구현 및 구성 요약** |                     |             |                                        |                               |

------


이 섹션은 "작동하게 만들기"에서 "빠르고 대규모로 작동하게 만들기"로 전환합니다.


앞서 분석했듯이, 기본 ROS 2 데이터 경로는 메모리 복사 및 직렬화 단계를 포함하여 오버헤드를 발생시키며, 이는 특히 이미지나 포인트 클라우드와 같은 대용량 데이터에서 문제가 됩니다.7

이 문제에 대한 해결책으로 ROS 2는 "Loaned Messages" API를 제공합니다. 사용자는 메시지를 생성하고 채워서 `publish()`에 전달하는 대신, `borrow_loaned_message()`를 통해 미들웨어로부터 직접 메모리 버퍼를 '대여'합니다. 사용자는 이 미리 할당된 버퍼를 채우고 발행 시 미들웨어에 반환합니다. 이는 사용자 공간에서 미들웨어의 내부 버퍼로의 복사를 제거합니다.10

여러 DDS 구현체는 제로카피를 위한 자체적인 메커니즘을 제공합니다:

- **Eclipse iceoryx:** CycloneDDS는 전용 공유 메모리 IPC 미들웨어인 Eclipse iceoryx와 통합될 수 있습니다. 활성화되면 동일한 호스트의 노드 간 통신은 포인터만 전달되므로 진정한 제로카피를 달성하여 상당한 성능 향상을 가져옵니다.64 주요 한계점은 전통적으로 컴파일 시간에 크기가 고정된 POD(Plain Old Data) 타입에만 작동하여, 가변 길이 배열(

  `std::vector`)을 포함하는 메시지에는 사용하기 어렵다는 점입니다.63

- **RTI Connext Zero Copy:** Connext는 유사한 공유 메모리 전송을 제공합니다. RMW를 우회하고 네이티브 Connext API를 사용하면 이 기능에 접근할 수 있으며, 대용량 데이터의 지연 시간을 극적으로 줄여 패킷 크기와 무관하게 만듭니다.11

- **Agnocast:** TIER IV에서 개발한 새로운 서드파티 솔루션으로, `LD_PRELOAD`를 통해 메모리 할당 함수(`malloc`/`free`)를 후킹하여 동적 크기 메시지에 대해서도 "진정한 제로카피"를 제공하는 것을 목표로 합니다.66


기본 DDS 노드 발견 메커니즘(Simple Discovery Protocol)은 UDP 멀티캐스트에 의존합니다. 많은 노드가 있는 대규모 시스템에서는 모든 노드가 다른 모든 노드를 발견하려고 시도하면서 '발견 폭풍(discovery storm)'이라 불리는 트래픽이 발생하여 완전히 연결된 그래프를 형성합니다. 이는 확장성이 떨어지며 기업이나 대학의 WiFi 네트워크에서는 종종 차단됩니다.12

- **해결책 1: Discovery Server:** Fast DDS는 "Discovery Server" 모드를 제공합니다. 노드들은 중앙 서버(또는 서버들)에 연결하는 클라이언트로 구성됩니다. 클라이언트는 통신에 필요한 노드에 대한 발견 정보만 수신하므로 멀티캐스트 트래픽이 대폭 감소하고 멀티캐스트가 비활성화된 네트워크에서도 통신이 가능해집니다.13
- **해결책 2: DDS Router:** 서로 다른 DDS 네트워크를 연결할 수 있는 훨씬 더 강력한 도구입니다. DDS Router는 다른 `ROS_DOMAIN_ID`, 다른 물리적 LAN, 심지어 다른 전송 프로토콜을 연결할 수 있습니다. 이는 복잡한 다중 로봇 또는 다중 사이트 배포에서 제어된 통신 채널을 만드는 데 이상적입니다.12


`rmw_fastrtps_cpp`와 `rmw_cyclonedds_cpp`를 비교하는 여러 성능 벤치마크가 수행되었습니다.69

- **Fast DDS:** 일반적으로 더 나은 처리량과 더 낮은 지연 시간을 보이며, 특히 대용량 데이터에서 뛰어납니다. 데이터 크기에 따른 확장성이 우수하지만, 메모리 사용량이 더 많은 경향이 있습니다.69
- **Cyclone DDS:** 구성 없이도 토픽 및 노드 수에 따라 매우 잘 확장되며 작은 메시지의 경우 지연 시간이 더 짧을 수 있습니다. 대용량 데이터에서는 성능이 더 크게 저하됩니다.14

Zenoh(`rmw_zenoh`)는 DDS의 일부 단점, 특히 무선, WAN 및 리소스 제약 환경에서의 문제를 해결하기 위해 설계된 새로운 pub/sub/query 프로토콜입니다.71 ROS 2 TSC는 

`rmw_zenoh`를 평가했으며 14, 벤치마크에 따르면 Zenoh는 WiFi 및 4G에서 DDS보다 우수한 성능을 보이고, 안정적인 이더넷에서는 DDS(Cyclone)가 더 나은 성능을 보입니다.71 Zenoh의 발견 메커니즘은 기본 DDS 멀티캐스트보다 효율적이어서 발견 트래픽을 최대 97%까지 줄일 수 있습니다.15


궁극적인 성능을 위해 ROS 2 계산 및 통신 로직을 하드웨어, 주로 FPGA로 오프로드할 수 있습니다.75 Acceleration Robotics의 

`RobotCore`와 같은 프로젝트는 ROS 2 네트워킹 스택(RCL, RMW, RTPS)을 하드웨어에서 직접 구현하는 FPGA IP 코어를 만들고 있습니다.77 이 접근 방식은 CPU 기반 소프트웨어 구현에 비해 지연 시간을 마이크로초 단위로 줄이고 전력 소비를 극적으로 낮추는 등 수십 배의 성능 향상을 약속하며 77, ROS 2 성능 최적화의 최전선을 대표합니다.

ROS 2의 성능 최적화 과정은 "추상화의 비용"에 대한 일련의 아키텍처적 절충안으로 볼 수 있습니다. 기준선은 사용하기 쉽지만 RMW 추상화로 인한 성능 오버헤드가 있는 표준 ROS 2 노드입니다. 첫 번째 최적화는 QoS 및 DDS XML 프로파일을 조정하는 것으로, 표준 아키텍처 내에 머무르지만 벤더별 기능에 접근하기 위해 RMW 추상화를 깨야 합니다. 다음 단계는 Loaned Messages API를 사용하는 것입니다. 이는 코드 변경이 필요하지만 메모리 복사를 줄여 상당한 성능 향상을 제공합니다. 이는 여전히 RMW를 통해 작동하지만 데이터 경로의 핵심 부분을 최적화합니다.10 최대 소프트웨어 성능을 위해서는 중요한 데이터 경로에 대해 ROS 2 클라이언트 라이브러리와 RMW를 완전히 우회하고 네이티브 DDS API(예: Connext Zero Copy)를 직접 사용해야 합니다. 이는 원시 속도를 위해 이식성과 ROS 통합(예: CLI 도구)을 희생합니다.11 마지막 단계는 전체 통신 스택을 실리콘으로 옮기는 하드웨어 가속입니다. 이는 가장 유연성이 낮지만 가장 높은 성능을 제공하는 옵션입니다.77 이는 

**사용 편의성/이식성**과 **성능/복잡성** 사이의 명확한 스펙트럼을 형성하며, 시스템 아키텍트는 시스템의 각 데이터 경로가 이 스펙트럼의 어느 지점에 위치해야 하는지 의식적으로 선택해야 합니다.

| RMW/방법                                           | 메시지 크기(소형/대형) | 지연 시간 (프로세스 간) | 처리량 (프로세스 간)    | 핵심 요약                                          |
| -------------------------------------------------- | ---------------------- | ----------------------- | ----------------------- | -------------------------------------------------- |
| Fast DDS (기본 비동기)                             | 2MB                    | 2.8 ms                  | 1999 MBps               | 기본 설정에서 우수한 처리량 69                     |
| Fast DDS (동기)                                    | 2MB                    | 2.6 ms                  | -                       | 비동기 모드보다 지연 시간 개선 69                  |
| Fast DDS (데이터 공유)                             | 2MB                    | ~0.7 ms                 | >2000 MBps              | 대용량 데이터에 가장 적합, 최저 지연 시간 69       |
| Cyclone DDS                                        | 2MB                    | 6.1 ms                  | 822 MBps                | 소형 메시지에서 강점, 대형 데이터에서 성능 저하 69 |
| Zenoh                                              | -                      | DDS와 유사/더 우수      | DDS보다 우수 (최대 3배) | 무선/WAN 환경에서 강점, 프로토콜 효율성 높음 74    |
| **표 3: 성능 벤치마크 종합 (지연 시간 및 처리량)** |                        |                         |                         |                                                    |

------


이 섹션에서는 DDS에서 상속된 내장 보안 기능을 활용하는 방법을 자세히 설명합니다.


ROS 2 보안(`sros2`)은 별도의 ROS 기능이 아니라 OMG의 DDS-Security 사양을 직접 구현한 것입니다.31 이는 세 가지 핵심 서비스를 제공합니다.

1. **인증 (Authentication):** 인증서 및 인증 기관(CA)을 사용하는 공개 키 기반 구조(PKI)를 통해 Participant가 자신이 누구인지 확인합니다.31
2. **접근 제어 (Access Control):** 인증된 특정 Participant가 어떤 토픽을 발행하거나 구독할 수 있는지 정의합니다. 이는 서명된 "거버넌스(Governance)" 및 "권한(Permissions)" XML 파일을 통해 관리됩니다.31
3. **암호화 (Cryptography):** 기밀성과 무결성을 보장하기 위해 유선 상의 토픽 데이터를 암호화하며, 일반적으로 AES-GCM을 사용합니다.31


`sros2` 명령줄 패키지는 보안 아티팩트를 생성하고 배포하는 복잡성을 관리하는 도구를 제공합니다.82

**워크플로우:**

1. `ros2 security create_keystore`: 시스템의 모든 보안 파일을 보관할 디렉토리를 생성합니다.82
2. `ros2 security create_key`: CA 및 개별 노드(enclave)에 필요한 키와 인증서를 생성합니다.
3. `ros2 security create_enclave`: 특정 노드 또는 논리적 노드 그룹에 대한 키, 인증서 및 정책 파일을 "보안 인클레이브(security enclave)"로 묶습니다.31


보안은 환경 변수를 통해 제어됩니다.

- `ROS_SECURITY_ENABLE=true`: 보안을 켜는 마스터 스위치입니다.81
- `ROS_SECURITY_KEYSTORE`: 이전 단계에서 생성된 키스토어 디렉토리의 경로입니다.31
- `ROS_SECURITY_STRATEGY=Enforce`: 파일이 없을 때 안전하지 않게 시작하는 허용 모드에서, 시작에 실패하는 엄격 모드로 동작을 변경합니다.31

보안 노드를 시작할 때는 명령줄 인수를 통해 어떤 인클레이브의 자격 증명을 사용할지 지정해야 합니다: `ros2 run <pkg> <node> --ros-args --enclave /path/to/enclave`.31

ROS 2 보안은 전통적인 네트워크 보안과 근본적으로 다른 패러다임을 제시합니다. 전통적인 네트워크 보안은 방화벽, VPN, 네트워크 경계 보안에 중점을 두는 반면, DDS-Security와 이를 기반으로 한 SROS2는 "제로 트러스트(zero-trust)" 모델에서 작동합니다. 즉, 네트워크 자체를 신뢰하지 않습니다. 보안은 각 Participant(노드)의 암호화된 *신원*에 연결되며, 이 신원은 개인 키와 신뢰할 수 있는 CA가 서명한 인증서로 증명됩니다.31 접근 제어는 IP 주소나 서브넷이 아닌, 인증서에 명시된 신원이 서명된 권한 파일에서 특정 토픽에 대한 발행/구독 권한을 부여받았는지 여부에 따라 결정됩니다.31 이는 보안 정책이 네트워크 토폴로지와 무관하게 애플리케이션 계층에서 시행됨을 의미합니다. 두 노드가 올바른 자격 증명을 가지고 동일한 CA를 신뢰하는 한, 신뢰할 수 없는 공용 WiFi 네트워크에서도 사설 LAN에서와 마찬가지로 안전하게 통신할 수 있습니다. 이는 많은 엔지니어에게 익숙한 모델과는 매우 다른, 훨씬 더 견고한 보안 모델입니다.

------


이 마지막 섹션에서는 보고서의 내용을 실행 가능한 의사 결정 프레임워크로 종합합니다.


- **시나리오 1: 레거시 또는 서드파티 DDS 컴포넌트 통합**
  - **권장 사항:** 섹션 2에 자세히 설명된 네이티브 DDS 상호운용성 접근 방식을 사용해야 합니다. 대안이 없습니다. 성공은 토픽/타입 이름 변환 및 QoS 일치에 대한 세심한 준수에 달려 있습니다.
- **시나리오 2: 고대역폭 프로세스 내 데이터 파이프라인 최적화 (예: 카메라 -> 인식)**
  - **권장 사항:** 제로카피 솔루션을 우선시합니다 (섹션 4.1). ROS 2의 프로세스 내 통신으로 시작하십시오. 노드가 동일한 호스트의 별도 프로세스에 있어야 하는 경우, POD 타입에 대해 CycloneDDS + iceoryx를 평가하거나 동적 타입에 대해 Agnocast를 조사하십시오. Loaned Messages API는 이 문제에 접근하는 표준 준수 방법입니다.
- **시나리오 3: WiFi를 통한 대규모 다중 로봇 플릿 배포**
  - **권장 사항:** 기본 DDS 멀티캐스트 발견은 적합하지 않습니다. 발견 트래픽을 관리하기 위해 Fast DDS Discovery Server(섹션 4.2)를 구현하십시오. 더 큰 제어가 필요하거나 네트워크 세그먼트를 연결하려면 DDS Router를 사용하십시오. 또한, 무선/손실 환경에서 우수한 성능과 발견 메커니즘을 위해 `rmw_zenoh`를 강력하게 평가하십시오 (섹션 4.3).
- **시나리오 4: 새로운 성능 중심 애플리케이션 개발**
  - **권장 사항:** 하이브리드 접근 방식을 고려하십시오. 비핵심 인터페이스(예: 구성, 상태)에는 표준 ROS 2 API를 사용하십시오. 핵심적인 고주파 데이터 경로의 경우, RMW를 우회하고 선택한 벤더의 네이티브 DDS API(예: RTI Connext)를 사용하여 최소한의 지연 시간과 제로카피와 같은 고급 기능에 접근하십시오.11


- **생태계를 수용하라:** RMW의 선택은 도구, 문서, 커뮤니티 지원을 포함한 전체 생태계에 대한 투자입니다. 프로젝트 요구 사항에 따라 현명하게 선택하십시오.
- **가정하지 말고 프로파일링하라:** 성능 특성은 사용 사례(메시지 크기, 빈도, 네트워크 조건)에 따라 크게 달라집니다. 아키텍처를 확정하기 전에 `performance_test` 및 벤더별 모니터와 같은 도구를 사용하여 특정 시나리오를 벤치마킹하십시오.71
- **도구를 마스터하라:** Wireshark 및 RTI Admin Console 또는 Fast DDS Monitor와 같은 벤더별 도구에 대한 숙련도는 DDS를 포함하는 모든 진지한 개발에 있어 선택이 아닌 필수입니다. 이는 분산 시스템에서 흔히 발생하는 "조용한 실패"를 디버깅하는 데 필수적입니다.56
- **설계에 의한 보안:** 프로젝트 시작부터 보안 아키텍처(인클레이브, CA, 권한)를 계획하십시오. 보안을 나중에 추가하는 것은 훨씬 더 어렵고 오류가 발생하기 쉽습니다.31


ROS 2와 DDS의 발전은 계속되고 있으며, 향상된 타입 지원(enum 등), 더 정교한 QoS 처리, 그리고 고급 기능의 RMW 계층으로의 긴밀한 통합과 같은 영역에서 지속적인 작업이 이루어지고 있습니다.84 Zenoh와 같은 대안의 부상과 하드웨어 가속으로의 추진은 차세대 로보틱스의 극한 성능 및 확장성 요구에 대한 솔루션을 제공하려는 명확한 추세를 나타냅니다.14 이러한 발전에 대한 최신 정보를 유지하는 것이 미래 지향적인 로봇 시스템을 구축하는 열쇠입니다.


1. Introduction to Robot Operating System 2 (ROS 2) - MATLAB & Simulink - MathWorks, accessed July 2, 2025, https://www.mathworks.com/help/ros/gs/robot-operating-system-ros2-basic-concepts.html
2. ROS 2: What is DDS | Data Distribution Service (DDS) Community RTI Connext Users, accessed July 2, 2025, https://community.rti.com/page/ros-2-what-dds
3. ROS 2 middleware interface - ROS2 Design, accessed July 2, 2025, https://design.ros2.org/articles/ros_middleware_interface.html
4. ROS2 + DDS: A Field Guide to Interoperability | RTI - Real-Time Innovations, accessed July 2, 2025, https://www.rti.com/blog/ros2-dds-a-field-guide-to-interoperability
5. ROS2 interoperability with native DDS - ROS Answers archive, accessed July 2, 2025, https://answers.ros.org/question/404995/
6. Ros2 and DDS messaging - ROS General - Open Robotics Discourse, accessed July 2, 2025, https://discourse.ros.org/t/ros2-and-dds-messaging/1556
7. Latency Analysis of ROS2 Multi-Node Systems - arXiv, accessed July 2, 2025, https://arxiv.org/pdf/2101.02074
8. Topic and Service name mapping to DDS - ROS2 Design, accessed July 2, 2025, https://design.ros2.org/articles/topic_and_service_names.html
9. osrf/ros2_raw_dds_example: A project showing how to ... - GitHub, accessed July 2, 2025, https://github.com/osrf/ros2_raw_dds_example
10. Zero Copy via Loaned Messages - ROS2 Design, accessed July 2, 2025, https://design.ros2.org/articles/zero_copy.html
11. The Indy Autonomous Challenge: Achieving Extreme Performance with ROS 2, accessed July 2, 2025, https://www.rti.com/blog/the-indy-autonomous-challenge-achieving-extreme-performance-with-ros-2
12. 1.2.2. How to Use DDS Router for Scalable ROS 2 Network Communication -, accessed July 2, 2025, https://docs.vulcanexus.org/en/jazzy/rst/tutorials/core/scalability/router_fully_connected/router_fully_connected.html
13. 16.2. Use ROS 2 with Fast-DDS Discovery Server - 3.2.2, accessed July 2, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/ros2/discovery_server/ros2_discovery_server.html
14. 2021 Eclipse Cyclone DDS ROS Middleware Evaluation Report with iceoryx and Zenoh, accessed July 2, 2025, https://osrf.github.io/TSC-RMW-Reports/humble/eclipse-cyclonedds-report.html
15. DDS Tuning for ROS 2 - breq.dev, accessed July 2, 2025, https://breq.dev/2024/05/17/dds
16. From ROS to ROS 2: A Comprehensive Guide to the Next Generation Robotics | by ScaleX Innovation, accessed July 2, 2025, https://scalexi.medium.com/from-ros-to-ros-2-a-comprehensive-guide-to-the-next-generation-robotics-f93a4e2e5793
17. Robot Operating System 2: Design, Architecture, and Uses In The Wild - ResearchGate, accessed July 2, 2025, https://www.researchgate.net/publication/365414963_Robot_Operating_System_2_Design_Architecture_and_Uses_In_The_Wild
18. Lessons Learned in Deploying ROS2 in Industrial Applications, accessed July 2, 2025, https://rosindustrial.squarespace.com/s/Lessons-Learned-in-Deploying-ROS2-in-Industrial-Apps-Robinson-March-25-no-animation.pdf
19. Introduction to ROS 2 (Robot Operating System 2) in Python - LearnOpenCV, accessed July 2, 2025, https://learnopencv.com/robot-operating-system-introduction/
20. Robot Operating System 2 (ROS 2) Architecture | by Huseyin Kutluca - Medium, accessed July 2, 2025, https://medium.com/software-architecture-foundations/robot-operating-system-2-ros-2-architecture-731ef1867776
21. Understanding the Publish-Subscribe (PubSub) Mechanism in ROS 2 Using rclpy, accessed July 2, 2025, https://www.roboticscontentlab.com/2024/09/29/publish-and-subscribe-mechanism-with-ros-2-rclpy/
22. ros2_i_training/workshop/source/_source/basics/ROS2-Simple-Publisher-Subscriber.md at main - GitHub, accessed July 2, 2025, https://github.com/ros-industrial/ros2_i_training/blob/main/workshop/source/_source/basics/ROS2-Simple-Publisher-Subscriber.md
23. About internal ROS 2 interfaces, accessed July 2, 2025, https://docs.ros.org/en/dashing/Concepts/About-Internal-Interfaces.html
24. rmw: ROS Middleware Abstraction Interface, accessed July 2, 2025, https://docs.ros2.org/foxy/api/rmw/
25. About ROS 2 middleware implementations, accessed July 2, 2025, https://docs.ros.org/en/foxy/Concepts/About-Middleware-Implementations.html
26. Eclipse Cyclone DDS - ROS 2 Documentation, accessed July 2, 2025, https://docs.ros.org/en/foxy/Installation/DDS-Implementations/Working-with-Eclipse-CycloneDDS.html
27. ROS 2 Communication - Clearpath Robotics Documentation, accessed July 2, 2025, https://docs.clearpathrobotics.com/docs/ros2humble/ros/networking/ros2_communication
28. Fast DDS - eProsima, accessed July 2, 2025, https://fast-dds.docs.eprosima.com/
29. Performance Evaluation of ROS2-DDS middleware implementations facilitating Cooperative Driving in Autonomous Vehicle. - arXiv, accessed July 2, 2025, https://arxiv.org/html/2412.07485v1
30. ROS QoS - Deadline, Liveliness, and Lifespan - ROS2 Design, accessed July 2, 2025, https://design.ros2.org/articles/qos_deadline_liveliness_lifespan.html
31. ROS 2 DDS-Security integration - ROS2 Design, accessed July 2, 2025, https://design.ros2.org/articles/ros2_dds_security.html
32. 16.1. Configuring Fast DDS in ROS 2 - 3.2.2, accessed July 2, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/ros2/ros2_configure.html
33. Unlocking the potential of Fast DDS middleware [community-contributed] - ROS 2, accessed July 2, 2025, https://docs.ros.org/en/galactic/Tutorials/Advanced/FastDDS-Configuration.html
34. ROS 2 Over Email: rmw_email, an Actual Working RMW Implementation, accessed July 2, 2025, https://christophebedard.com/ros-2-over-email/
35. Latency Overhead of ROS2 for Modular Time-Critical Systems - arXiv, accessed July 2, 2025, http://www.arxiv.org/pdf/2101.02074v2
36. Subscribe to ROS2 topic from native ePromisa DDS program - Stack Overflow, accessed July 2, 2025, https://stackoverflow.com/questions/68272082/subscribe-to-ros2-topic-from-native-epromisa-dds-program
37. neil-rti/ros2-interop-demos: Demonstration apps to show ROS2 interoperability with native DDS apps - GitHub, accessed July 2, 2025, https://github.com/neil-rti/ros2-interop-demos
38. About ROS 2 interfaces - ROS 2 Documentation: Foxy documentation, accessed July 2, 2025, https://docs.ros.org/en/foxy/Concepts/About-ROS-Interfaces.html
39. Core Stack Developer Overview - ros_core bouncy documentation, accessed July 2, 2025, https://docs.ros2.org/bouncy/developer_overview.html
40. IDL - Interface Definition and Language Mapping - ROS2 Design, accessed July 2, 2025, https://design.ros2.org/articles/idl_interface_definition.html
41. design/articles/115_idl.md at gh-pages / ros2/design - GitHub, accessed July 2, 2025, https://github.com/ros2/design/blob/gh-pages/articles/115_idl.md
42. ROS 2 Getting Started - Safe DDS docs - eProsima, accessed July 2, 2025, https://safe-dds.docs.eprosima.com/intro/getting_started_ros2.html
43. DDS to ROS2 messages - ROS Answers archive, accessed July 2, 2025, https://answers.ros.org/question/406529/
44. My first application in FastDDS. I have some experience dealing with ROS... | by Hitoruna, accessed July 2, 2025, https://medium.com/@hitorunajp/438750d9527c
45. Communication between Cyclone and ROS2 / Issue #1412 / eclipse ..., accessed July 2, 2025, https://github.com/eclipse-cyclonedds/cyclonedds/issues/1412
46. ROS 2 Quality of Service (QoS) - Isaac Sim Documentation, accessed July 2, 2025, https://docs.robotsfan.com/isaacsim/5.0.0/ros2_tutorials/tutorial_ros2_qos.html
47. Communication between RTI Connext DDS and ROS2 - Stack Overflow, accessed July 2, 2025, https://stackoverflow.com/questions/51523890/communication-between-rti-connext-dds-and-ros2
48. Setting up the RTI DDS configuration file in ROS2 - Stack Overflow, accessed July 2, 2025, https://stackoverflow.com/questions/52151179/setting-up-the-rti-dds-configuration-file-in-ros2
49. DDS implementations - ROS 2 Documentation: Foxy documentation, accessed July 2, 2025, https://docs.ros.org/en/foxy/Installation/DDS-Implementations.html
50. DDS implementations - ROS 2 Documentation: Iron documentation, accessed July 2, 2025, https://docs.ros.org/en/iron/Installation/DDS-Implementations.html
51. 16. ROS 2 using Fast DDS middleware - 3.2.2, accessed July 2, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/ros2/ros2.html
52. Running ROS 2 on Multiple Machines - Husarion, accessed July 2, 2025, https://husarion.com/tutorials/ros2-tutorials/6-robot-network/
53. ROS 2 Middleware (RMW) Configuration - GitHub Pages, accessed July 2, 2025, https://iroboteducation.github.io/create3_docs/setup/xml-config/
54. 1.1.4. How to handle large data video streaming in ROS 2 - - Vulcanexus Docs, accessed July 2, 2025, https://docs.vulcanexus.org/en/jazzy/rst/tutorials/core/wifi/large_data/large_data.html
55. eProsima/Fast-DDS: The most complete DDS - Proven: Plenty of success cases. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 2, 2025, https://github.com/eProsima/Fast-DDS
56. 1. Fast DDS Monitor with ROS 2 - 3.2.0, accessed July 2, 2025, https://fast-dds-monitor.readthedocs.io/en/latest/rst/ros/ros.html
57. rticommunity/rticonnextdds-robot-helpers: Collection of code and cmake utilities to use RTI Connext DDS with ROS 2 - GitHub, accessed July 2, 2025, https://github.com/rticommunity/rticonnextdds-robot-helpers
58. ROS 2: RTI Tools Overview | Data Distribution Service (DDS ..., accessed July 2, 2025, https://community.rti.com/page/ros-2-rti-tools-overview
59. Getting Started with Admin Console 1 of 4 - YouTube, accessed July 2, 2025, https://www.youtube.com/watch?v=akNAr-5JPlY
60. Connecting ROS 2 Nodes with the Custom CycloneDDS XML Config File | Husarnet, accessed July 2, 2025, https://husarnet.com/docs/ros2/custom-cyclonedds-xml/
61. Network Troubleshooting Tools - Clearpath Robotics Documentation, accessed July 2, 2025, https://docs.clearpathrobotics.com/docs/ros/networking/network_troubleshooting/network_troubleshooting_tools
62. Working with multiple ROS 2 middleware implementations, accessed July 2, 2025, https://docs.ros.org/en/humble/How-To-Guides/Working-with-multiple-RMW-implementations.html
63. [ROS2] Use loaned messages to optimize the performance for image transport / Issue #216 / ros-perception/image_common - GitHub, accessed July 2, 2025, https://github.com/ros-perception/image_common/issues/216
64. Zero-copy data transfer in ROS 2 - YouTube, accessed July 2, 2025, https://www.youtube.com/watch?v=gbSBRqLCVjE
65. Talk:Usingzero-copy data transfer in ROS 2 - General, accessed July 2, 2025, https://discourse.ros.org/t/talk-usingzero-copy-data-transfer-in-ros-2/21448
66. Introduce True Zero-Copy Publish/Subscribe IPC to Autoware!! (New Middleware Coexisting with ROS 2) / autowarefoundation / Discussion #5835 - GitHub, accessed July 2, 2025, https://github.com/orgs/autowarefoundation/discussions/5835
67. ROS 2 Discovery Configuration - Clearpath Robotics Documentation, accessed July 2, 2025, https://docs.clearpathrobotics.com/docs/ros/networking/ros2_discovery_config
68. 1.3.1. Discovery Server Minimal Example - - Vulcanexus Docs, accessed July 2, 2025, https://docs.vulcanexus.org/en/jazzy/rst/tutorials/core/discoveryserver/discoveryserverminimalexample/discovery_server_minimal_example.html
69. Fast DDS TSC RMW report 2021 - GitHub Pages, accessed July 2, 2025, https://osrf.github.io/TSC-RMW-Reports/humble/eProsima-response.html
70. New Fast DDS Performance Testing - ROS General - Open Robotics Discourse, accessed July 2, 2025, https://discourse.ros.org/t/new-fast-dds-performance-testing/29539
71. Comparison of Middlewares in Edge-to-Edge and Edge-to-Cloud Communication for Distributed ROS 2 Systems - arXiv, accessed July 2, 2025, https://arxiv.org/html/2309.07496v4
72. Open Class - Zenoh ROS 2 RMW: A New Middleware Implementation, accessed July 2, 2025, https://app.theconstruct.ai/live_class/17306cc2-eaa8-4195-a115-f92dc2e6ef28/
73. Comparison of Middlewares in Edge-to-Edge and Edge-to-Cloud Communication for Distributed ROS 2 Systems - arXiv, accessed July 2, 2025, http://arxiv.org/pdf/2309.07496
74. Comparison of Middlewares in Edge-to-Edge and Edge-to-Cloud Communication for Distributed ROS2 Systems | Papers With Code, accessed July 2, 2025, https://paperswithcode.com/paper/comparison-of-dds-mqtt-and-zenoh-in-edge-to
75. ROS 2 Hardware Acceleration Working Group - GitHub, accessed July 2, 2025, https://github.com/ros-acceleration
76. An open architecture for Hardware Acceleration in ROS 2.pdf, accessed July 2, 2025, [http://download.ros.org/downloads/roscon/2022/An%20open%20architecture%20for%20Hardware%20Acceleration%20in%20ROS%202.pdf](http://download.ros.org/downloads/roscon/2022/An open architecture for Hardware Acceleration in ROS 2.pdf)
77. ROBOTCORE® for ROS 2 | Ultra-Fast FPGA-Powered Networking for ROS 2 Robots, accessed July 2, 2025, https://accelerationrobotics.com/robotcore-ros2.php
78. RobotCore: An Open Architecture for Hardware Acceleration in ROS 2 | Brian Plancher, accessed July 2, 2025, https://brianplancher.com/publication/robotcore/
79. ROS 2 on a Chip, Achieving Brain-Like Speeds and Efficiency in Robotic Networking - arXiv, accessed July 2, 2025, https://arxiv.org/html/2404.18208v1
80. Zenoh Performance - ROS General - Open Robotics Discourse, accessed July 2, 2025, https://discourse.ros.org/t/zenoh-performance/30494
81. ROS 2 Security - ROS 2 Documentation: Rolling documentation, accessed July 2, 2025, https://docs.ros.org/en/rolling/Concepts/Intermediate/About-Security.html
82. Setting up security - ROS 2 Documentation: Rolling documentation, accessed July 2, 2025, https://docs.ros.org/en/rolling/Tutorials/Advanced/Security/Introducing-ros2-security.html
83. 11x Performance Gain: Latest Connext DDS, ROS 2 Performance Benchmarks, accessed July 2, 2025, https://www.rti.com/blog/latest-connext-dds-ros-2-performance-benchmarks
84. Feature Ideas - ROS 2 Documentation, accessed July 2, 2025, https://ros2-documentation-zh-cn.readthedocs.io/en/rolling/source/The-ROS2-Project/Feature-Ideas.html
85. ROS2 message format should support enums / Issue #260 / ros2/rosidl - GitHub, accessed July 2, 2025, https://github.com/ros2/rosidl/issues/260
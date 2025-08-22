---
layout: page
title: # 서론
permalink: /services/dds/RTI Connext DDS/RTI Connext DDS
---
RTI Connext DDS


RTI Connext DDS는 단순한 제품을 넘어, 복잡하고 실시간성이 요구되는 미션 크리티컬 분산 시스템 구축을 위한 핵심적인 커넥티비티 프레임워크입니다. 이는 Object Management Group (OMG)의 데이터 분산 서비스(Data Distribution Service, DDS) 표준을 선도적으로 상용화한 구현체로, 시스템의 구성 요소들이 마치 하나의 통합된 개체처럼 원활하게 작동하도록 설계되었습니다.1 이 기술의 핵심은 기존의 메시지 중심(message-centric) 패러다임에서 벗어나, 공유된 상태 정보를 기반으로 하는 데이터 중심(data-centric) 패러다임을 채택한 데 있습니다.

본 보고서는 RTI Connext DDS의 기술적 본질을 다각도로 분석하는 것을 목표로 합니다. 먼저 DDS 표준의 근간을 이루는 데이터 중심 발행-구독(Publish-Subscribe) 모델, 글로벌 데이터 공간(Global Data Space), 그리고 핵심 아키텍처 구성 요소를 탐구합니다. 이어서 RTI Connext 제품군의 특징과 다른 DDS 구현체와의 비교를 통해 그 독자적인 가치를 조명합니다. 시스템의 비기능적 요구사항을 정밀하게 제어하는 서비스 품질(Quality of Service, QoS) 정책의 심층 분석은 이 기술이 어떻게 까다로운 실시간 요구사항을 충족시키는지 보여줄 것입니다. 또한, RTI가 제공하는 강력한 툴체인 생태계가 개발 및 운영 효율성을 어떻게 극대화하는지 살펴봅니다. 마지막으로 자동차, 항공우주, 로보틱스 등 다양한 산업 분야에서의 전략적 적용 사례와 다른 미들웨어 기술과의 비교 분석을 통해, RTI Connext DDS가 현대 분산 시스템 아키텍처에서 차지하는 위상과 그 전략적 가치를 종합적으로 제시하고자 합니다.


RTI Connext DDS를 이해하기 위해서는 그 근간이 되는 OMG의 DDS 표준과 데이터 중심 패러다임을 먼저 이해해야 합니다. DDS는 분산 시스템의 통신 방식을 근본적으로 재정의하며, 이는 애플리케이션 설계에 깊은 영향을 미칩니다.


DDS의 가장 핵심적인 특징은 '데이터 중심(Data-Centric)' 통신 모델을 채택했다는 점입니다. 이는 "무엇을 할지 지시하는" 메시지 중심(message-centric) 접근 방식과 근본적인 차이를 보입니다. 메시지 중심 모델에서는 애플리케이션 A가 애플리케이션 B에게 '데이터를 보내라'는 명령을 보내고, B가 이에 응답하는 방식입니다. 반면, 데이터 중심 모델은 "상태를 공유하는" 방식으로 작동하며, 이는 마치 분산된 실시간 데이터베이스와 유사합니다.3

이 모델은 발행-구독(Publish-Subscribe) 메커니즘을 통해 구현됩니다.

- **발행자(Publisher)**는 특정 '토픽(Topic)'에 대한 데이터를 생성하고 발행(write)합니다. 예를 들어, 온도 센서는 '기온'이라는 토픽에 현재 온도 값을 지속적으로 발행할 수 있습니다.5

- **구독자(Subscriber)**는 자신이 관심 있는 '토픽'을 구독(subscribe)하여 해당 토픽에 발행되는 데이터를 수신(read)합니다.

  이 과정에서 가장 중요한 점은 발행자와 구독자가 서로의 존재나 위치에 대해 전혀 알 필요가 없다는 것입니다.6 이러한 특성을 '느슨한 결합(Loose Coupling)'이라 하며, DDS는 이를 통해 시스템의 유연성과 모듈성을 극대화합니다. 기존의 클라이언트-서버 모델에서는 클라이언트가 서버의 주소(IP, 포트 등)를 알아야만 통신이 가능했지만, DDS에서는 미들웨어가 데이터의 전파를 모두 책임지므로 애플리케이션 개발자는 데이터 자체의 로직에만 집중할 수 있습니다.7


'글로벌 데이터 공간(Global Data Space)'은 DDS의 데이터 중심 모델을 구현하는 핵심적인 추상 개념입니다.4 이는 특정 도메인에 참여하는 모든 애플리케이션이 접근할 수 있는 가상의, 분산된 정보 저장소로 생각할 수 있습니다. 각 애플리케이션은 이 공간에 데이터를 발행하거나 구독함으로써, 마치 자신의 로컬 메모리에 있는 데이터에 접근하는 것처럼 정보를 공유합니다.

이러한 추상화는 애플리케이션 로직을 획기적으로 단순화합니다. 개발자는 "어떻게(how)" 데이터를 전송할지에 대한 복잡한 네트워크 프로그래밍(소켓 관리, 데이터 직렬화/역직렬화, 재전송 로직 등)을 고민할 필요 없이, "무엇을(what)" 즉, 시스템의 데이터 모델 자체에 집중하게 됩니다.6 DDS 미들웨어는 데이터 마샬링(marshalling) 및 디마샬링, 전송, 흐름 제어, 재시도 등 통신과 관련된 모든 복잡한 작업을 백그라운드에서 자동으로 처리합니다.6

이러한 아키텍처 패턴은 시스템의 유지보수성과 확장성에 지대한 영향을 미칩니다. 예를 들어, 초기에 센서 데이터(발행자)와 제어 로직(구독자)만으로 구성된 시스템이 있다고 가정해 봅시다. 이후에 데이터 로깅이나 시각화를 위한 새로운 애플리케이션을 추가해야 할 경우, 이 새로운 애플리케이션은 단순히 기존의 센서 데이터 토픽을 구독하기만 하면 됩니다. 발행자인 센서 애플리케이션이나 다른 구독자 애플리케이션에는 어떠한 코드 변경도 필요 없습니다. 이처럼 기존 구성 요소를 수정하지 않고도 새로운 기능을 쉽게 추가할 수 있는 능력은 DDS가 복잡한 시스템-오브-시스템(System-of-Systems)에서 각광받는 주된 이유 중 하나입니다.1


DDS 아키텍처는 몇 가지 핵심적인 구성 요소(Entity)로 이루어집니다.6

- **도메인(Domain):** 도메인은 독립적인 통신 공간을 정의하는 컨테이너입니다. 동일한 네트워크 상에 있더라도 서로 다른 도메인에 속한 애플리케이션들은 원칙적으로 서로 통신하지 않습니다. 이를 통해 여러 개의 독립적인 DDS 시스템이 물리적 네트워크를 공유하면서도 논리적으로는 완벽하게 격리되어 운영될 수 있습니다.7
- **DomainParticipant:** 애플리케이션이 특정 도메인에 참여하기 위한 진입점 역할을 하는 엔티티입니다. Publisher, Subscriber, Topic 등을 생성하는 팩토리 역할을 수행합니다.6
- **토픽(Topic):** 발행자와 구독자를 연결하는 가장 핵심적인 요소입니다. 토픽은 단순히 문자열 이름이 아니라, '이름'과 '데이터 타입'의 조합으로 고유하게 정의됩니다. 이 데이터 타입은 주로 IDL(Interface Definition Language)을 통해 엄격하게 정의되며, 이는 DDS의 강력한 타입 안정성(strong typing)을 보장하는 기반이 됩니다.5
- **발행자(Publisher) / DataWriter:** Publisher는 하나 이상의 DataWriter를 관리하는 컨테이너입니다. 실제 데이터 샘플을 특정 토픽에 쓰는(write) 행위는 DataWriter 엔티티가 수행합니다.6
- **구독자(Subscriber) / DataReader:** Subscriber는 하나 이상의 DataReader를 관리하는 컨테이너입니다. 특정 토픽으로부터 데이터 샘플을 읽는(read) 행위는 DataReader 엔티티가 담당합니다.6


DDS는 특정 벤더에 종속되지 않는 개방형 국제 표준이며, 이는 OMG(Object Management Group)에 의해 제정되고 관리됩니다.1 DDS 표준은 크게 두 가지 계층으로 구성됩니다.

1. **애플리케이션 API 계층:** 개발자가 사용하는 API로, 데이터 중심 발행-구독을 위한 DCPS(Data-Centric Publish-Subscribe)와 이를 더 쉽게 사용하도록 돕는 선택적 상위 계층인 DLRL(Data Local Reconstruction Layer)로 나뉩니다.6
2. **와이어 프로토콜(Wire Protocol) 계층:** 네트워크 상에서 데이터가 실제로 교환되는 방식을 정의합니다.

DDS의 와이어 프로토콜 표준이 바로 **RTPS(Real-Time Publish-Subscribe) 프로토콜**입니다.10 RTPS는 서로 다른 벤더가 개발한 DDS 구현체라 할지라도 네트워크 수준에서 서로 통신할 수 있도록 보장하는 핵심적인 역할을 합니다. 즉, RTI Connext DDS를 사용하는 애플리케이션과 eProsima Fast DDS를 사용하는 애플리케이션이 원활하게 데이터를 교환할 수 있는 것은 바로 이 RTPS 표준 덕분입니다. 이는 특정 벤더에 대한 종속성(vendor lock-in)을 방지하고, 사용자에게 기술 선택의 유연성을 제공하는 매우 중요한 특징입니다.

DDS의 브로커리스(brokerless) P2P 아키텍처는 고성능과 낮은 지연 시간을 가능하게 하는 직접적인 요인입니다. 중앙 서버라는 병목 지점을 제거함으로써 노드 간 직접 통신을 허용하며, 이는 실시간 제어 루프에 매우 중요합니다. MQTT와 같은 브로커 기반 시스템은 모든 데이터가 중앙 서버를 거쳐야 하므로, 해당 서버의 용량이 전체 시스템의 성능을 좌우하고 단일 장애점(Single Point of Failure)이 될 수 있습니다.3 반면, DDS의 동적 발견(dynamic discovery) 메커니즘은 참여자들이 네트워크에서 서로를 직접 찾아 P2P 데이터 흐름을 설정하게 해줍니다.4 이러한 아키텍처적 선택은 동적 발견 및 네트워크 구성(예: 멀티캐스트 관리)의 복잡성을 증가시키는 대신, 결정론적이고 낮은 지연 시간이 최우선인 애플리케이션에 월등한 성능, 확장성, 그리고 견고함을 제공합니다.


OMG DDS 표준이 '설계도'라면, RTI Connext DDS는 이 설계도를 바탕으로 제작된 가장 정교하고 강력한 '상용 제품'이라 할 수 있습니다. RTI는 표준을 충실히 구현하는 것을 넘어, 미션 크리티컬 시스템이 요구하는 다양한 기능과 전문적인 지원을 통해 독자적인 가치를 창출합니다.


RTI(Real-Time Innovations)는 DDS 미들웨어 분야의 글로벌 리더로서, 세계 최대 규모의 DDS 전담 엔지니어링 팀을 보유하고 있습니다.1 RTI Connext는 단순한 DDS 표준 구현체를 넘어, 고성능과 풍부한 기능을 갖춘 상용 소프트웨어 프레임워크입니다.1

RTI의 핵심적인 가치 제안은 '소프트웨어 판매'를 넘어 '솔루션 제공'과 '리스크 관리'에 있습니다. 특히 자율주행차, 항공 시스템, 의료 기기와 같이 안전과 신뢰성이 극도로 중요한 미션 크리티컬 시스템 분야에 집중합니다.1 RTI는 단순히 빠른 미들웨어를 제공하는 데 그치지 않고, 고객이 직면할 수 있는 개발 및 통합 과정의 리스크를 관리하고, 전문적인 기술 컨설팅과 글로벌 기술 지원을 통해 프로젝트의 성공적인 완수를 돕습니다.1


RTI는 다양한 산업 분야와 시스템 요구사항에 맞춰 최적화된 제품군을 제공함으로써 그 가치를 더욱 구체화합니다.

- **Connext Professional:** 범용 고성능 분산 시스템을 위한 RTI의 주력 제품입니다.10
- **Connext Drive®:** 소프트웨어 정의 차량(Software-Defined Vehicle, SDV)이라는 특정 도메인을 위해 설계된 자동차용 커넥티비티 프레임워크입니다. 영역(Zonal) 아키텍처, ADAS(첨단 운전자 보조 시스템), 텔레매틱스 등 차량 내 복잡한 데이터 흐름을 실시간으로 연결하는 데 특화되어 있습니다.1
- **Connext Cert:** 가장 높은 수준의 안전 인증이 요구되는 시스템을 위한 제품입니다. 자동차 기능 안전 표준인 ISO 26262 ASIL-D나 항공전자 소프트웨어 표준인 DO-178C 등을 충족해야 하는 시스템을 위해 개발되었습니다. 이 제품은 소프트웨어뿐만 아니라, 인증 심사에 필요한 모든 문서, 테스트 증거, 안전 매뉴얼 등 '인증 산출물(certification artifacts)'을 함께 제공합니다.10
- **Connext Micro:** 마이크로컨트롤러(MCU)와 같이 리소스가 극도로 제한된 임베디드 장치를 위한 경량화 버전입니다.10
- **AI 기반 기능:** RTI는 최근 Connext에 AI 기반 기능을 통합하여, 자율 시스템 개발을 위한 최초의 소프트웨어 프레임워크로 포지셔닝하고 있습니다.10

이러한 제품 전략은 RTI가 단순한 기술 공급업체가 아니라, 특정 수직 시장의 복잡한 문제를 해결하는 '솔루션 파트너'로서의 역할을 지향하고 있음을 명확히 보여줍니다. 예를 들어, 안전 필수 자동차 시스템을 개발하는 회사는 단순히 통신 라이브러리만 필요한 것이 아닙니다. 그들은 규제 기관에 전체 소프트웨어 스택이 안전하다는 것을 ISO 26262 표준에 따라 입증해야 합니다. 미들웨어 구성 요소에 대한 필수 문서, 테스트 증거, 안전 매뉴얼을 자체적으로 개발하는 것은 막대하고 비용이 많이 드는 작업입니다. RTI는 `Connext Cert`를 통해 이 작업을 고객을 대신하여 수행합니다. 이 제품에는 소프트웨어와 함께 필요한 인증 자료가 포함되어 있습니다.12 이는 가치 제안을 "빠른 미들웨어입니다"에서 "프로젝트의 위험을 줄이고, 개발에 수백만 달러를 절약하며, 시장 출시 시간을 단축할 사전 인증된 구성 요소입니다"로 전환시킵니다. 이는 상용 라이선스 비용을 정당화하고 오픈 소스 대안과 차별화되는 강력한 비즈니스 모델입니다.


DDS 생태계에는 RTI Connext 외에도 여러 중요한 오픈소스 구현체들이 존재하며, 각각의 특징과 장단점이 있습니다.

- **eProsima Fast DDS:** ROS 2의 기본(Default) DDS 구현체로, 우수한 성능과 활발한 커뮤니티를 기반으로 빠르게 발전하고 있습니다.13
- **Eclipse Cyclone DDS:** 가볍고 성능이 뛰어나다는 평을 받으며 점차 사용이 확대되고 있는 또 다른 주요 오픈소스 구현체입니다.13
- **OpenDDS:** 역사가 깊은 오픈소스 구현체 중 하나로, P2P 방식의 동적 발견(Discovery) 오버헤드를 줄이기 위해 `DCPSInfoRepo`라는 중앙 집중식 정보 저장소를 사용하는 독특한 아키텍처를 특징으로 합니다.7

이들 간의 핵심적인 차이는 라이선스 모델, 기술 지원, 툴링의 성숙도, 그리고 안전 인증 제공 여부에서 나타납니다. 오픈소스 구현체들은 초기 도입 비용이 없고 유연성이 높다는 장점이 있지만, 미션 크리티컬 시스템에 요구되는 체계적인 기술 지원이나 안전 인증 패키지는 상용 솔루션인 RTI Connext가 제공하는 핵심적인 차별점입니다.7

RTPS 와이어 프로토콜 덕분에 이론적으로는 서로 다른 DDS 구현체 간의 통신이 가능하지만 10, 이는 벤더 종속성을 완전히 배제하는 것을 의미하지는 않습니다. RTI가 제공하는 성숙한 툴체인(Admin Console, Routing Service 등)과 전문적인 기술 지원, 그리고 안전 인증 패키지는 개발 생산성과 시스템 안정성 측면에서 강력한 가치를 제공하며, 이는 많은 기업들이 상용 솔루션을 선택하는 이유가 됩니다. 여러 DDS 구현체가 RTPS 와이어 프로토콜을 준수하며 공존하는 건강한 생태계는 사용자에게 이점을 제공합니다. RTI가 프리미엄 상용 옵션인 반면, 실행 가능한 오픈 소스 대안의 가용성은 완전한 벤더 종속을 방지하고 전반적인 혁신을 촉진합니다. 이 경쟁 환경은 RTI를 포함한 모든 벤더가 기본 표준을 넘어서는 가치(예: 우수한 툴링, 성능 또는 인증)를 지속적으로 혁신하고 추가하도록 강제합니다. 사용자는 궁극적으로 다양한 가격대와 기능 수준에서 선택권을 가짐으로써 이익을 얻습니다.


| 구현체                  | 라이선스 모델            | 주요 사용 사례 / 강점                                        | 툴링 및 생태계 성숙도                                        | 안전 인증 가용성                                             |
| ----------------------- | ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **RTI Connext DDS**     | 상용                     | 미션/안전 크리티컬 시스템, 산업용 IoT, 자동차, 항공우주. 강력한 툴링과 전문 지원. | 매우 높음 (Admin Console, Routing Service, Recording Service 등 통합 툴 제공) | 사용 가능 (Connext Cert 제품군 통해 ISO 26262, DO-178C 등 제공) |
| **eProsima Fast DDS**   | 오픈소스 (Apache 2.0)    | ROS 2의 기본 미들웨어, 로보틱스, 연구 개발. 우수한 성능과 활발한 커뮤니티. | 중간 (ROS 2 생태계와 긴밀히 통합, 독립적인 툴은 제한적)      | 제공되지 않음                                                |
| **Eclipse Cyclone DDS** | 오픈소스 (EPL-2.0)       | 경량화 및 고성능이 요구되는 시스템, 임베디드.                | 중간 (성능에 집중, 포괄적인 툴링은 부족)                     | 제공되지 않음                                                |
| **OpenDDS**             | 오픈소스 (자체 라이선스) | 다양한 산업 분야, DCPSInfoRepo를 통한 중앙 관리형 발견 메커니즘. | 중간 (자체 모니터링 툴 등 제공)                              | 제공되지 않음                                                |

자료 출처: 7


DDS의 가장 강력한 기능 중 하나는 바로 서비스 품질(Quality of Service, QoS) 프레임워크입니다. QoS는 단순한 통신 프로토콜을 넘어, 분산 시스템의 아키텍처적 특성을 정의하고 제어하는 엔진 역할을 합니다.


QoS는 개발자가 데이터 통신의 비기능적 측면(non-functional aspects)을 정밀하게 제어할 수 있도록 하는 풍부한 정책(policy)들의 집합입니다.6 이 정책들은 DataWriter, DataReader, Topic 등 DDS의 각 엔티티에 적용되어 신뢰성, 데이터 유효 기간, 업데이트 주기, 리소스 사용량 등 시스템의 다양한 동작 방식을 결정합니다.15

QoS 프레임워크의 진정한 가치는 이러한 시스템의 비기능적 요구사항을 애플리케이션 코드 레벨이 아닌, 미들웨어의 '설정' 레벨에서 구현할 수 있다는 점에 있습니다. 예를 들어, 전통적인 시스템에서 발행자의 장애에 대비한 이중화(redundancy)와 페일오버(failover) 로직을 구현하려면, 애플리케이션 개발자는 하트비트(heartbeat) 메시지, 타임아웃 감지, 백업 컴포넌트로의 전환 등 복잡한 로직을 직접 코드로 작성해야 합니다. 하지만 DDS에서는 `LIVELINESS`, `OWNERSHIP`, `OWNERSHIP_STRENGTH`와 같은 QoS 정책들을 적절히 설정하는 것만으로 미들웨어가 이 모든 과정을 자동으로 처리해 줍니다. 이는 애플리케이션 코드를 비즈니스 로직에만 집중하도록 극도로 단순화시키며 6, 시스템의 견고성을 크게 향상시킵니다. 즉, 시스템의 아키텍처적 속성이 '프로그래밍'되는 것이 아니라 '선언'되는 것입니다.


수십 가지에 달하는 DDS QoS 정책들은 기능에 따라 몇 가지 그룹으로 분류하여 이해할 수 있습니다.

- **데이터 전달 보장 (Data Delivery):**
  - `RELIABILITY`: 데이터 전송의 신뢰성 수준을 결정합니다. `BEST_EFFORT`는 UDP처럼 최대한 전달하려 노력하지만 보장하지 않는 방식이며, `RELIABLE`은 TCP처럼 데이터가 유실될 경우 재전송하여 반드시 전달을 보장하는 방식입니다.8
  - `HISTORY`: DataWriter 또는 DataReader의 내부 큐에 얼마나 많은 데이터 샘플을 저장할지를 결정합니다. `KEEP_LAST`는 지정된 개수만큼 최신 데이터만 유지하고, `KEEP_ALL`은 리소스 한도 내에서 모든 데이터를 유지합니다. `RELIABLE` 통신을 위해서는 `HISTORY` 정책이 함께 고려되어야 합니다.16
- **데이터 영속성 및 적시성 (Data Persistence & Timeliness):**
  - `DURABILITY`: 늦게 참여하는(late-joining) 구독자에게 이전에 발행된 데이터를 전달할지 여부를 결정합니다. `VOLATILE`은 발행 시점에 구독하고 있는 구독자에게만 데이터를 전달하고, `TRANSIENT_LOCAL`, `TRANSIENT`, `PERSISTENT`는 DataWriter나 별도의 영속성 서비스(Persistence Service)에 데이터를 저장해 두었다가 새로운 구독자에게 전달합니다.8
  - `LIFESPAN`: 발행된 데이터가 유효한 시간을 지정합니다. 이 시간이 지나면 데이터는 자동으로 폐기됩니다.17
  - `DEADLINE`: 특정 데이터 인스턴스가 주기적으로 업데이트되어야 하는 최대 시간을 명시합니다. 만약 이 시간 내에 새로운 데이터가 도착하지 않으면, 미들웨어는 애플리케이션에 이를 통지하여 데이터 누락을 감지할 수 있게 합니다.8
  - `LATENCY_BUDGET`: 데이터 전송에 허용되는 최대 시간을 미들웨어에 제안하여, 긴급한 데이터가 먼저 전송되도록 유도합니다.8
- **시스템 활성 상태 (System Liveness):**
  - `LIVELINESS`: DataWriter가 여전히 활성 상태인지(살아있는지)를 판단하는 메커니즘을 정의합니다. DataWriter는 주기적으로 '하트비트'를 보내며, DataReader는 이 하트비트가 끊기면 해당 DataWriter가 비정상 종료되었음을 감지할 수 있습니다.8
- **데이터 스트림 조정 (Data Stream Coordination):**
  - `OWNERSHIP` 및 `OWNERSHIP_STRENGTH`: 동일한 토픽에 대해 여러 개의 DataWriter가 데이터를 발행하는 이중화 구성에서 어떤 DataWriter의 데이터를 수신할지를 결정합니다. `SHARED`는 모든 DataWriter의 데이터를 수신하고, `EXCLUSIVE`는 `OWNERSHIP_STRENGTH` 값이 가장 높은 하나의 DataWriter로부터만 데이터를 수신합니다. 이를 통해 Active-Standby와 같은 페일오버 구성을 쉽게 구현할 수 있습니다.8


QoS 정책들이 실제로 어떻게 상호작용하는지를 이해하는 데 있어 가장 중요한 개념은 '요청-제공(Request-Offered, RxO)' 모델입니다.8 이는 DataReader와 DataWriter 간의 통신이 성립되기 위한 일종의 '계약'과 같습니다.

- **제공(Offered):** DataWriter는 자신이 제공할 수 있는 서비스의 수준을 QoS 정책을 통해 '제공'합니다.
- **요청(Requested):** DataReader는 자신이 필요로 하는 서비스의 수준을 QoS 정책을 통해 '요청'합니다.

통신은 DataWriter가 제공하는 서비스 수준이 DataReader가 요청하는 수준과 호환될 때(같거나 더 높을 때)만 성립됩니다. 만약 DataReader가 DataWriter가 제공하는 것보다 더 높은 수준의 서비스를 요청하면, 두 엔티티의 QoS는 '비호환(incompatible)' 상태가 되며 데이터 교환은 이루어지지 않습니다.16

예를 들어, 한 DataReader가 `RELIABILITY.kind = RELIABLE` (신뢰성 있는 전송)을 요청했는데, 매칭되는 DataWriter가 `RELIABILITY.kind = BEST_EFFORT` (최선형 전송)를 제공한다면, 이 둘은 연결되지 않습니다. 반대의 경우, 즉 DataWriter가 `RELIABLE`을 제공하고 DataReader가 `BEST_EFFORT`를 요청한다면 연결은 성립됩니다.

이러한 RxO 모델의 엄격함은 개발 초기에는 다소 불편하게 느껴질 수 있지만, 대규모 시스템을 안정적으로 구축하는 데에는 매우 중요한 기능입니다. 이는 서로 다른 팀이나 회사에서 개발한 두 개의 복잡한 시스템을 통합할 때 발생할 수 있는 "통신 동작에 대한 잘못된 가정"(예: "당연히 데이터를 신뢰성 있게 보낼 줄 알았다")으로 인한 오류를 근본적으로 방지합니다. QoS 프로파일은 두 시스템 간의 인터페이스에 대한 공식적이고 기계적으로 검증 가능한 '통합 계약'이 됩니다. 만약 한 팀이 자신의 DataWriter QoS를 변경하여 이 계약을 위반하면, 연결 자체가 성립되지 않아 통합 초기에 문제를 명확하게 발견하고 수정할 수 있습니다. 이는 런타임에 발생할 수 있는 미묘하고 디버깅하기 어려운 오류를 예방하는 데 큰 가치를 지닙니다. QoS 불일치 문제는 RTI Admin Console과 같은 툴이나 DDS가 제공하는 리스너(listener), 로그 메시지를 통해 진단할 수 있습니다.15


| 시스템 요구사항                                      | 관련 QoS 정책                     | 주요 파라미터                        | 대표 사용 사례                                               |
| ---------------------------------------------------- | --------------------------------- | ------------------------------------ | ------------------------------------------------------------ |
| **데이터 유실을 절대 허용하면 안 됨**                | `RELIABILITY`                     | `kind = RELIABLE`                    | 제어 명령, 이벤트 알림 등 반드시 전달되어야 하는 데이터      |
| **최신 데이터만 중요하며, 약간의 유실은 허용됨**     | `RELIABILITY`                     | `kind = BEST_EFFORT`                 | 비디오 스트리밍, 고주파 센서 데이터 등 최신성이 중요한 데이터 |
| **새로운 참여자가 과거 데이터를 받아야 함**          | `DURABILITY`                      | `kind = TRANSIENT` 또는 `PERSISTENT` | 시스템의 현재 상태 정보(예: 장비 위치, 설정값)를 공유        |
| **데이터가 일정 주기로 업데이트되는지 감시해야 함**  | `DEADLINE`                        | `period`                             | 센서 데이터가 주기적으로 들어오는지 확인하여 센서 고장 감지  |
| **발행자가 비정상 종료되었는지 감지해야 함**         | `LIVELINESS`                      | `kind`, `lease_duration`             | 제어 시스템의 활성 상태를 모니터링하여 장애 감지             |
| **주 발행기에 문제 발생 시 예비 발행기로 자동 전환** | `OWNERSHIP`, `OWNERSHIP_STRENGTH` | `kind = EXCLUSIVE`, `value`          | 고가용성이 요구되는 시스템의 이중화(Redundancy) 구성         |

자료 출처: 8


RTI Connext DDS의 강력함은 핵심 미들웨어뿐만 아니라, 분산 시스템의 개발, 디버깅, 모니터링, 통합을 지원하는 포괄적인 툴체인 생태계에서 비롯됩니다. 이 도구들은 DDS의 분산적, 비중앙집중적 특성에서 발생하는 고유한 문제들을 해결하기 위해 설계되었으며, 개발 생산성과 시스템 안정성을 극적으로 향상시킵니다.


RTI Admin Console은 실행 중인 DDS 시스템의 상태를 들여다보는 중앙 관제 센터와 같습니다.21 DDS는 본질적으로 중앙 서버가 없어 "지금 시스템이 무엇을 하고 있는가?"를 파악하기 어렵다는 단점이 있는데, Admin Console은 이 문제를 해결하여 시스템에 대한 통합된 가시성을 제공합니다.

주요 기능은 다음과 같습니다.

- **시스템 가시화 (System/Physical/Logical Views):** 네트워크에 참여하고 있는 모든 DomainParticipant, Topic, DataWriter, DataReader를 자동으로 발견하여 논리적, 물리적 관점에서 시스템의 전체 토폴로지(topology)와 데이터 흐름을 시각적으로 보여줍니다.22
- **매치 분석 (Match Analysis):** 시스템을 실시간으로 분석하여 호환되지 않는 QoS 설정으로 인해 통신이 이루어지지 않는 DataWriter와 DataReader 쌍을 자동으로 찾아내고 그 원인을 명확히 알려줍니다. 이는 QoS 설정 오류를 디버깅하는 데 매우 강력한 기능입니다.22
- **데이터 샘플 검사 (Sample Inspection):** 특정 토픽을 선택하여 실시간으로 발행되는 데이터의 내용과 메타데이터를 직접 확인할 수 있습니다. 필터링 기능을 통해 원하는 데이터만 선별하여 볼 수도 있습니다.21
- **성능 모니터링 및 리소스 소비 (Performance Monitoring & Resource Consumption):** RTI의 다른 인프라 서비스(예: Routing Service)와 연동하여 데이터 처리량(throughput), 지연 시간(latency), CPU 및 메모리 사용량과 같은 성능 지표를 차트로 시각화하여 보여줍니다.22
- **서비스 관리 (Service Administration):** 네트워크 상의 다른 RTI 인프라 서비스들을 원격으로 시작, 중지, 설정 변경 등의 관리를 수행할 수 있습니다.22
- **QoS 프로파일 편집기 (QoS Profile Editor):** 복잡한 XML 형식의 QoS 설정 파일을 GUI 환경에서 직관적으로 편집할 수 있도록 도와줍니다.26


RTI Routing Service는 DDS 시스템의 자연스러운 경계인 '도메인'과 물리적 '네트워크'의 한계를 극복하기 위한 강력한 게이트웨이 솔루션입니다.27 DDS 도메인은 의도적으로 격리되어 있지만, 대규모 시스템을 통합하기 위해서는 이 격리를 넘어서는 데이터 공유가 필요합니다. Routing Service는 바로 이 문제를 해결하기 위한 지능적이고 설정 가능한 데이터 브리지 역할을 합니다.

주요 기능은 다음과 같습니다.

- **DDS 도메인 간 브릿징:** 서로 다른 도메인 간에 특정 토픽의 데이터를 선택적으로 전달하여, 논리적으로 분리된 시스템들을 통합할 수 있습니다.28
- **LAN/WAN 연결:** 방화벽이나 NAT(Network Address Translation) 환경으로 분리된 원격지의 DDS 시스템들을 TCP와 같은 프로토콜을 이용해 안전하게 연결합니다. 이를 통해 지리적으로 분산된 시스템-오브-시스템을 구축할 수 있습니다.27
- **데이터 변환 및 필터링:** 서로 다른 시스템을 통합할 때 발생하는 데이터 모델의 불일치 문제를 해결합니다. 라우팅 과정에서 데이터의 내용, 타입, 토픽 이름, QoS 설정 등을 동적으로 변환하여 이기종 시스템 간의 원활한 통신을 가능하게 합니다.27
- **프로토콜 어댑터:** 플러그인 아키텍처를 통해 DDS가 아닌 다른 프로토콜과도 연동할 수 있습니다. 예를 들어, MQTT 어댑터 플러그인을 사용하면 MQTT 브로커와 데이터를 교환하여 IoT 장치들을 DDS 데이터버스에 통합할 수 있습니다.30


RTI Recording Service와 Replay Service는 DDS 시스템을 위한 고정밀 '블랙박스' 또는 '비행 데이터 기록계'와 같습니다.31 실시간 시스템의 동작은 일시적이고 재현하기 어려워 디버깅에 큰 어려움이 따르는데, 이 서비스는 시스템의 상호작용을 완벽하게 포착하여 오프라인 분석 및 재현을 가능하게 함으로써 이 문제를 해결합니다.

주요 구성 요소와 기능은 다음과 같습니다.

- **Recording Service:** DDS 도메인 내에서 교환되는 모든 데이터와 동적 발견(discovery) 트래픽을 고정밀 타임스탬프와 함께 캡처하여 데이터베이스(기본적으로 SQLite)에 저장합니다.31
- **Replay Service:** 기록된 데이터를 DDS 시스템에 다시 재생합니다. 이때 원본 데이터의 시간 간격을 그대로 유지하며 재생하기 때문에, 과거의 특정 상황을 매우 사실적으로 재현할 수 있습니다. 이는 버그 재현, 성능 분석, 시뮬레이션 환경 구축 등에 매우 유용합니다.31
- **Converter:** 기록된 데이터를 효율적인 저장 포맷인 이진(binary) 형식과 분석에 용이한 쿼리 가능 포맷인 JSON 형식 간에 상호 변환하는 유틸리티입니다.31

이 서비스는 특히 ROS 2 환경에서 기본 제공되는 `rosbag2`에 비해 특정 사용 사례에서 더 높은 견고성과 성능을 보여주는 대안으로 활용되기도 합니다.32

이처럼 통합된 툴체인은 RTI의 강력한 경쟁 우위이자, 사용자가 RTI 생태계에 머무르게 하는 중요한 요인입니다. 핵심 DDS 통신은 RTPS 프로토콜을 통해 상호 운용될 수 있지만, 개발 및 관리 경험은 RTI 생태계 내에서 월등히 우수합니다. 조직은 오픈 소스 DDS를 사용할 수 있지만, 비슷한 수준의 툴체인을 다양한 오픈 소스 프로젝트에서 조합하거나 자체적으로 구축해야 하는 상당한 엔지니어링 노력이 필요합니다. RTI는 즉시 사용 가능한 세련되고 통합된 지원 도구 모음을 제공합니다. 이러한 도구를 사용하여 얻는 생산성 향상과 개발 시간 단축은 상용 라이선스 비용을 쉽게 상쇄할 수 있습니다. 팀의 워크플로우가 디버깅을 위한 Admin Console, 통합을 위한 Routing Service, 테스트를 위한 Recording Service를 중심으로 구축되면 다른 벤더로 이전하는 것은 매우 어렵고 비용이 많이 드는 제안이 됩니다. 이는 툴링 생태계를 RTI 비즈니스 전략의 핵심 부분으로 만듭니다.


RTI Connext DDS는 성능, 신뢰성, 확장성에 대한 엄격한 요구사항 때문에 다양한 미션 크리티컬 산업에서 핵심 커넥티비티 기술로 채택되고 있습니다. 모든 산업 분야에서 DDS 채택의 공통적인 동인은 대규모 '통합 복잡성' 관리입니다. 시스템이 더욱 소프트웨어 정의되고 상호 연결됨에 따라 기존의 점대점 또는 클라이언트-서버 아키텍처는 취약하고 관리하기 어려워집니다. DDS의 데이터 중심, 분리된 아키텍처는 이러한 통합 문제를 해결하기 위한 강력한 아키텍처 패턴을 제공하며, 이것이 차세대 시스템의 백본으로 선택되는 이유입니다.


현대 자동차는 수백 개의 ECU(전자 제어 장치)가 복잡하게 연결된 '바퀴 달린 데이터 센터'로 진화하고 있으며, 소프트웨어 정의 차량(SDV) 패러다임은 이러한 변화를 주도하고 있습니다. DDS는 이러한 복잡한 E/E(전기/전자) 아키텍처를 위한 이상적인 데이터 중심 통신 프레임워크를 제공합니다.1

- **아키텍처 지원:** RTI Connext Drive는 여러 ECU를 기능별로 묶는 도메인(Domain) 아키텍처나 물리적 위치에 따라 묶는 영역(Zonal) 아키텍처의 핵심 통신 백본 역할을 합니다. 이를 통해 ADAS(첨단 운전자 보조 시스템)용 고성능 컴퓨터, 영역 게이트웨이, 텔레매틱스 유닛, 디지털 클러스터 등 차량 내 다양한 소프트웨어 컴포넌트 간의 실시간 데이터 흐름을 원활하게 연결합니다.1
- **기능 안전 (Functional Safety):** 자율주행, 조향, 제동과 같은 안전 필수(Safety-critical) 기능은 최고 수준의 기능 안전을 요구합니다. RTI의 `Connext Cert` 제품은 자동차 기능 안전 표준인 ISO 26262 ASIL-D 등급의 인증을 획득하는 데 필요한 소프트웨어와 인증 산출물을 제공하여, OEM과 부품 공급업체(Tier-1)의 개발 리스크와 시간을 획기적으로 줄여줍니다.1
- **산업 채택:** RTI는 이미 25개 이상의 자동차 공급업체와 협력하고 있으며, 10개 이상의 글로벌 OEM(완성차 업체)이 양산 차량에 RTI 기술을 적용하고 있습니다. 중국의 대표적인 전기차 제조사인 XPENG(샤오펑)이 차세대 E/E 아키텍처의 핵심 통신 기술로 Connext Drive를 채택한 것은 대표적인 사례입니다.1


항공우주 및 국방 분야는 극도의 신뢰성, 보안, 실시간 성능을 요구하는 시스템의 집약체입니다. DDS는 이 분야에서 모듈형 개방형 시스템 아키텍처(Modular Open Systems Approach, MOSA)를 구현하는 핵심 기술로 자리 잡고 있습니다. MOSA는 시스템의 재사용성을 높이고, 유지보수 및 업그레이드 비용을 절감하며, 변화하는 임무 요구사항에 신속하게 대응하기 위한 국방 시스템 설계 철학입니다.33

- **적용 분야:** RTI Connext는 미 공군의 MQ-9 Reaper와 같은 무인 항공 시스템(UAS)의 지상 관제소, 전투 시스템, 해군 함정의 전투 관리 시스템, 그리고 분산 시뮬레이션 및 훈련 환경 등 광범위한 분야에서 사용됩니다.1 MQ-9 플랫폼의 복잡한 통신 요구사항은 DDS가 왜 필요한지를 잘 보여줍니다.36
- **핵심 요구사항 충족:** 이 분야에서는 수많은 센서, 무장, 통제 시스템 간의 데이터를 지연 없이 안정적으로 공유하는 것이 무엇보다 중요합니다. DDS의 P2P 아키텍처와 강력한 QoS 정책은 이러한 요구사항을 충족시키며, `Connext Cert`는 DO-178C와 같은 엄격한 항공전자 소프트웨어 안전 표준을 준수해야 하는 시스템에 적용됩니다.33


로봇 운영체제(Robot Operating System, ROS)는 로봇 애플리케이션 개발을 위한 사실상의 표준 프레임워크입니다. 1세대 ROS(ROS 1)는 중앙의 마스터 노드와 P2P TCP 기반 통신으로 인해 실시간 성능, 신뢰성, 확장성 측면에서 한계를 보였습니다.

- **DDS 채택 이유:** ROS 2는 이러한 한계를 극복하기 위해 통신 미들웨어를 DDS로 전면 교체했습니다. DDS를 채택함으로써 ROS 2는 중앙 마스터 노드를 제거한 진정한 분산 시스템 아키텍처를 갖추게 되었고, 실시간 성능, 향상된 신뢰성, 그리고 대규모 시스템으로의 확장성을 확보했습니다.11
- **아키텍처 매핑:** ROS 2의 핵심 개념인 노드(Node), 토픽(Topic), 서비스(Service), 액션(Action) 등은 모두 내부적으로 DDS의 엔티티(DomainParticipant, Topic, DataWriter/Reader 등)로 매핑되어 동작합니다.42
- **RTI Connext의 역할:** RTI Connext는 eProsima Fast DDS, Eclipse Cyclone DDS와 함께 ROS 2가 공식적으로 지원하는 DDS 구현체 중 하나입니다. 특히 상용 로봇이나 산업용 로보틱스와 같이 고성능과 안정적인 기술 지원이 중요한 분야에서 RTI Connext가 선호되는 경우가 많습니다.13


현대 병원은 수많은 의료 기기들이 생성하는 데이터를 실시간으로 통합하고 분석하여 환자의 안전을 증진해야 하는 과제를 안고 있습니다. DDS는 이러한 의료 기기 통합 환경을 구축하는 데 효과적인 솔루션을 제공합니다.12

- **해결 과제:** 환자 감시 장치, 수액 펌프, 인공호흡기 등 다양한 기기에서 스트리밍되는 실시간 데이터를 수집하고, 특정 환자에 대한 여러 기기의 데이터를 연관 분석하며, 위급 상황 시 신뢰성 있는 알람을 전파하는 것이 핵심 과제입니다.34
- **DDS 적용:** RTI는 GitHub을 통해 의료 기기 통합 시나리오에 대한 구체적인 예제 코드(`rticonnextdds-usecases-medical`)를 제공합니다. 이 예제는 의료 데이터 모델을 IDL로 정의하는 방법, 데이터의 중요도에 따라 QoS를 설정하는 방법, 알람 데이터를 처리하는 애플리케이션 로직 등을 포함하고 있어 실제 개발에 큰 도움을 줍니다.46


미들웨어를 선택하는 것은 단순한 기술적 결정이 아니라, 구축하려는 시스템의 근본적인 목적과 아키텍처를 반영하는 전략적 결정입니다. RTI Connext DDS의 가치를 명확히 이해하기 위해서는 다른 주요 미들웨어 패러다임과의 비교가 필수적입니다. 엔지니어는 "어느 것이 가장 빠른가?"라고 물을 수 있지만, 시스템 아키텍트는 "내 시스템에 필요한 통신 패턴은 무엇인가?"라고 물어야 합니다.


DDS와 MQTT는 모두 발행-구독 모델을 사용하지만, 그 철학과 아키텍처는 근본적으로 다릅니다. 이 둘은 직접적인 경쟁 관계라기보다는 상호 보완적인 관계에 가깝습니다.3

- **아키텍처:** DDS는 중앙 서버가 없는 P2P(Peer-to-Peer) 브로커리스(brokerless) 모델입니다. 참여자들은 서로를 동적으로 발견하여 직접 통신합니다. 반면, MQTT는 모든 메시지가 중앙의 브로커(Broker)를 거쳐야 하는 중앙 집중식 모델입니다.3
- **데이터 모델:** DDS는 IDL을 통해 엄격하게 정의된 '구조화된 데이터(structured data)'를 교환하는 '데이터 중심' 모델입니다. 토픽은 데이터의 타입 그 자체를 의미합니다. MQTT는 경량의 '메시지 중심' 모델로, 토픽은 단순히 메시지를 분류하기 위한 계층적인 문자열(string)에 가깝습니다.3
- **성능:** DDS는 낮은 지연 시간(low-latency)과 높은 처리량(high-throughput)이 요구되는 실시간 제어 시스템에 최적화되어 있습니다. MQTT는 대역폭이 낮고 불안정한 네트워크 환경에서 경량의 데이터를 전송하는 데 최적화되어 있습니다.3
- **신뢰성 및 QoS:** DDS는 신뢰성, 내구성, 소유권 등 20가지가 넘는 매우 세분화된 QoS 정책을 통해 시스템의 동작을 정밀하게 제어합니다. MQTT는 0, 1, 2의 세 가지 단순한 QoS 레벨을 제공하여 메시지 전달 보장 수준을 결정합니다.3
- **사용 사례:** 시스템의 주요 통신 패턴이 실시간 제어 루프 내 피어 간의 **상태 동기화**(예: 자율주행차의 센서 퓨전 및 제어 모듈)라면 DDS가 자연스러운 아키텍처 선택입니다. 반면, 수천 개의 제약된 장치에서 중앙 클라우드 애플리케이션으로 **이벤트 알림**(예: IoT 센서의 온도 보고)을 보내는 것이 주된 패턴이라면 MQTT가 적합합니다. 따라서 DDS는 시스템의 '엣지(Edge)' 단에서 실시간 제어와 데이터 공유에 탁월하며, MQTT는 '엣지'와 '클라우드'를 연결하는 원격 측정 및 IoT 데이터 수집에 강점을 보입니다.3


AMQP(Advanced Message Queuing Protocol)를 구현한 RabbitMQ와 같은 전통적인 메시지 큐는 DDS와는 또 다른 목적을 위해 설계되었습니다.

- **패러다임 차이:** DDS는 데이터의 '상태'를 공유하는 데이터 중심 발행-구독 모델인 반면, 메시지 큐는 처리해야 할 '작업(task)'이나 '이벤트'를 큐(Queue)에 넣어 비동기적으로 처리하는 메시지 지향(message-oriented) 모델입니다.35

- **주요 사용 분야:** 메시지 큐는 주로 금융 거래 처리, 주문 처리 시스템 등 트랜잭션의 보장과 작업의 순차적 처리가 중요한 엔터프라이즈 백엔드 시스템에서 사용됩니다. DDS는 실시간 제어, 데이터 융합 등이 중요한 산업용, 임베디드 시스템에 주로 사용됩니다.35

- **아키텍처 차이:** DDS는 실시간 성능과 멀티캐스트 활용을 위해 주로 UDP 기반으로 동작하는 반면, AMQP는 신뢰성 있는 전달을 위해 TCP 기반으로 동작합니다.35 시스템의 통신 패턴이 엔터프라이즈 백엔드 서비스 간의 

  **비동기 작업 처리**이고 전달 보장과 트랜잭션 동작이 핵심이라면 AMQP/RabbitMQ가 적합한 선택입니다.


| 구분                   | **DDS (Data Distribution Service)**                          | **MQTT (Message Queuing Telemetry Transport)**               | **AMQP (Advanced Message Queuing Protocol)**               |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------------- |
| **아키텍처**           | 분산형, P2P, 브로커리스                                      | 중앙 집중형, 브로커 기반                                     | 중앙 집중형, 브로커 기반 (라우팅 기능 포함)                |
| **통신 모델**          | 데이터 중심 발행-구독 (상태 공유)                            | 메시지 중심 발행-구독 (이벤트 알림)                          | 메시지 지향, 큐잉, 라우팅 (작업 처리)                      |
| **데이터/메시지 모델** | 강력한 타입, 구조화된 데이터 (IDL 정의)                      | 경량, 비구조화된 페이로드, 문자열 기반 토픽                  | 헤더와 바디로 구성된 메시지, 복잡한 라우팅 규칙            |
| **주요 전송 프로토콜** | UDP (멀티캐스트, 유니캐스트), TCP                            | TCP                                                          | TCP                                                        |
| **핵심 강점**          | 낮은 지연 시간, 높은 처리량, 실시간 제어, 세분화된 QoS, 동적 발견, 견고성 | 경량, 낮은 전력/대역폭 소비, 단순성, 클라우드 통합 용이성    | 트랜잭션 보장, 유연한 라우팅, 메시지 큐잉, 상호운용성      |
| **이상적인 사용 사례** | 자율주행, 항공/국방, 산업 자동화, 실시간 제어 시스템, 로보틱스 (엣지 컴퓨팅) | IoT 센서 네트워크, 모바일 메시징, 원격 모니터링 (엣지-클라우드 연결) | 금융 시스템, 엔터프라이즈 백엔드, 비동기 작업 처리, 통신사 |
| **QoS 모델**           | 20+개의 매우 상세하고 강력한 정책 (신뢰성, 내구성, 생존성, 소유권 등) | 3단계 레벨 (At most once, At least once, Exactly once)       | 전달 보장 (Acknowledged/Transactional)                     |

자료 출처: 3


본 기술 분석을 통해 RTI Connext DDS는 개방형 DDS 표준에 기반한 프리미엄급 데이터 중심 커넥티비티 프레임워크로서, 세계에서 가장 까다로운 실시간 분산 시스템의 근간을 이루고 있음이 명확해졌습니다. 이는 단순히 데이터를 교환하는 라이브러리가 아니라, 시스템의 아키텍처적 요구사항을 미들웨어 계층에서 선언적으로 구현하고, 강력한 툴체인을 통해 개발과 운영의 전 과정을 지원하며, 안전 인증 패키지를 통해 프로젝트의 리스크를 관리하는 포괄적인 '솔루션'입니다.

RTI Connext DDS의 핵심적인 차별점은 다음과 같이 요약될 수 있습니다.

- **데이터 중심 패러다임:** 시스템의 복잡성을 근본적으로 줄이고 모듈성과 확장성을 극대화합니다.
- **세분화된 QoS 제어:** 신뢰성, 실시간성, 내결함성과 같은 비기능적 요구사항을 애플리케이션 코드 변경 없이 미들웨어 설정만으로 정밀하게 구현할 수 있는 강력한 메커니즘을 제공합니다.
- **성숙한 툴링 생태계:** Admin Console, Routing Service, Recording Service 등은 분산 시스템의 '가시성', '통합성', '재현성'이라는 고질적인 난제를 해결하여 개발 및 유지보수 생산성을 비약적으로 향상시킵니다.
- **안전 인증을 통한 리스크 관리:** Connext Cert와 같은 제품은 ISO 26262, DO-178C와 같은 엄격한 산업 표준을 준수해야 하는 기업에게 단순한 기술을 넘어 '시장 출시 시간 단축'과 '개발 리스크 감소'라는 전략적 가치를 제공합니다.

따라서 시스템 아키텍트는 다음의 특성을 가지는 시스템을 설계할 때 RTI Connext DDS를 핵심 기술로 고려해야 합니다.

1. **높은 복잡성과 다수의 분산 실시간 데이터 스트림을 가지는 시스템:** 자율주행차, 통합 항공 시스템, 대규모 산업 자동화 설비 등 수백, 수천 개의 데이터 스트림이 실시간으로 상호작용해야 하는 경우.
2. **성능, 신뢰성, 결정론적 동작에 대한 엄격한 요구사항이 있는 시스템:** 마이크로초 단위의 지연 시간과 데이터 유실 없는 완벽한 신뢰성이 요구되는 미션 크리티컬 제어 시스템.
3. **확장 가능하고, 모듈화되었으며, 진화 가능한 아키텍처가 필요한 시스템:** 초기 구축 이후에도 지속적으로 새로운 기능이 추가되고 컴포넌트가 변경/업그레이드되어야 하는 장기적인 프로젝트.
4. **인증이 필수적인 미션 크리티컬 또는 안전 크리티컬 기능을 포함하는 시스템:** 실패가 허용되지 않으며, 규제 기관의 엄격한 심사를 통과해야 하는 자동차, 국방, 의료 분야의 시스템.

인공지능, 자율성, 산업용 사물 인터넷(IIoT)의 시대가 본격화됨에 따라, 분산된 엣지 장치들 간의 원활하고 신뢰성 있는 데이터 공유의 중요성은 더욱 커지고 있습니다. RTI Connext DDS가 제공하는 데이터 중심 커넥티비티는 이러한 미래 기술 트렌드를 실현하는 데 없어서는 안 될 핵심적인 기반 기술(enabling technology)로서 그 역할을 계속해서 확장해 나갈 것으로 전망됩니다.


1. 소프트웨어 정의 차량을 위한 자동차 통신 프레임워크 리더 - RTI, accessed July 5, 2025, https://www.rti.com/hubfs/_Collateral/Executive-Briefs/RTI-executive-brief-Connectivity-Leader-for-SDV-KR.pdf
2. Compare RTI Connext DDS vs. EMQX in 2025 - Slashdot, accessed July 5, 2025, https://slashdot.org/software/comparison/Connext-DDS-vs-EMQX/
3. Navigating IIoT Protocols: Comparing DDS and MQTT - RTI, accessed July 5, 2025, https://www.rti.com/blog/comparing-dds-and-mqtt
4. DDS and MQTT: Basics, Challenges and Integration Benefits | EMQ - EMQX, accessed July 5, 2025, https://www.emqx.com/en/blog/navigating-dds-basics-limitations-and-integration-with-mqtt
5. 데이터 분산 서비스 - 위키백과, 우리 모두의 백과사전, accessed July 5, 2025, [https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EB%B6%84%EC%82%B0_%EC%84%9C%EB%B9%84%EC%8A%A4](https://ko.wikipedia.org/wiki/데이터_분산_서비스)
6. 데이터 분산 서비스 - 위키백과, 우리 모두의 백과사전, accessed July 5, 2025, https://ko.wikipedia.org/wiki/데이터_분산_서비스
7. Data Distribution Service - kjmin's devlog - 티스토리, accessed July 5, 2025, https://leichtjoon.tistory.com/54
8. Advanced Tutorial Using QoS to Solve Real-World Problems - DDS Foundation, accessed July 5, 2025, https://www.dds-foundation.org/sites/default/files/DDS_Advanced_Ttutorial_00-T5_Hunt-revised.pdf
9. [myLV.net 집필진 강좌] RTI DDS(Data Distribution Service) Toolkit 소개 - NI Community, accessed July 5, 2025, [https://forums.ni.com/t5/Archive-%EA%B0%95%EC%A2%8C%EA%B2%8C%EC%8B%9C%ED%8C%90/myLV-net-%EC%A7%91%ED%95%84%EC%A7%84-%EA%B0%95%EC%A2%8C-RTI-DDS-Data-Distribution-Service-Toolkit-%EC%86%8C%EA%B0%9C/ta-p/3867503](https://forums.ni.com/t5/Archive-강좌게시판/myLV-net-집필진-강좌-RTI-DDS-Data-Distribution-Service-Toolkit-소개/ta-p/3867503)
10. Data Distribution Service for Complex Systems | DDS Standard | RTI, accessed July 5, 2025, https://www.rti.com/products/dds-standard
11. [DDS] DDS와 RTPS - Interactics - 티스토리, accessed July 5, 2025, https://huroint.tistory.com/entry/DDS-DDS-RTPS
12. SDV 진짜 뼈대는 데이터 - RTI CTO가 말하는 DDS의 미래, accessed July 5, 2025, https://www.autoelectronics.co.kr/article/articleView.asp?idx=6233
13. DDS(Data Distribution Service) 이해하기 - Hoon's Blog - 티스토리, accessed July 5, 2025, https://yhoons.tistory.com/117
14. ROS 2: What is DDS | Data Distribution Service (DDS) Community ..., accessed July 5, 2025, https://community.rti.com/page/ros-2-what-dds
15. RTI Academy - Quality of Service - 1 QoS Overview - YouTube, accessed July 5, 2025, https://www.youtube.com/watch?v=nNXt2CysZyE
16. 5. Basic QoS - RTI Connext Getting Started documentation - RTI Community, accessed July 5, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/getting_started_guide/cpp11/intro_qos.html
17. RTI Connext DDS QoS Reference Guide, accessed July 5, 2025, https://community.rti.com/static/documentation/connext-dds/7.0.0/doc/manuals/connext_dds_professional/qos_reference/RTI_ConnextDDS_CoreLibraries_QoS_Reference_Guide.pdf
18. RTI Connext DDS QoS Reference Guide, accessed July 5, 2025, https://community.rti.com/static/documentation/connext-dds/6.1.0/doc/manuals/connext_dds_professional/qos_reference/RTI_ConnextDDS_CoreLibraries_QoS_Reference_Guide.pdf
19. RTI® Connext™ DDS-Comprehensive Summary of QoS Policies, accessed July 5, 2025, https://community.rti.com/static/documentation/connext-dds/5.2.0/doc/manuals/connext_dds/RTI_ConnextDDS_CoreLibraries_QoS_Reference_Guide.pdf
20. accessed January 1, 1970, [https.www.dds-foundation.org/sites/default/files/DDS_Advanced_Ttutorial_00-T5_Hunt-revised.pdf](http://docs.google.com/https.www.dds-foundation.org/sites/default/files/DDS_Advanced_Ttutorial_00-T5_Hunt-revised.pdf)
21. RTI Tools – Administration Console - Dedicated Systems Australia, accessed July 5, 2025, https://dedicatedsystems.com.au/wp-content/uploads/RTI-AdminConsole-Flyer.pdf
22. 2.3. Features Overview - RTI Admin Console 7.5.0 documentation, accessed July 5, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/tools/admin_console/p2_administration/features_overview/features_overview_all.html
23. Getting Started with Admin Console 1 of 4 - YouTube, accessed July 5, 2025, https://www.youtube.com/watch?v=akNAr-5JPlY
24. RTI Connext DDS: Data Visualization Overview - YouTube, accessed July 5, 2025, https://www.youtube.com/watch?v=kDzNfJz1XvU
25. Introduction to Admin Console - YouTube, accessed July 5, 2025, https://www.youtube.com/watch?v=Ob_weer8Om4
26. AdminConsoleOverview - YouTube, accessed July 5, 2025, https://www.youtube.com/watch?v=pQtOP28K094
27. Q&A: RTI Introduces Wide Area Routing Service; Scaling To Hundreds of Thousands of Applications - A-Team Insight, accessed July 5, 2025, https://a-teaminsight.com/blog/q-scaling-to-hundreds-of-thousands-of-applications/?brand=ati
28. RTI IS - Routing Service - Real-Time Innovations, accessed July 5, 2025, https://www.rti.com/products/is/routing-service
29. rticonnextdds-usecases/DataSubsetWAN/README.md at master - GitHub, accessed July 5, 2025, https://github.com/rticommunity/rticonnextdds-usecases/blob/master/DataSubsetWAN/README.md
30. rticommunity/rtiroutingservice-plugins: A collection of plugins for RTI Routing Service - GitHub, accessed July 5, 2025, https://github.com/rticommunity/rtiroutingservice-plugins
31. 1. Introduction - RTI Recording Service 7.5.0 documentation, accessed July 5, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/services/recording_service/introduction.html
32. How to use RTI Recording Service with ROS2 | by Nickolai Belakovski | Medium, accessed July 5, 2025, https://medium.com/@nbelakovski/how-to-use-rti-recording-service-with-ros2-9e57d486b38c
33. Aerospace & Defense (A&D) | Mission-Critical Systems - RTI, accessed July 5, 2025, https://www.rti.com/industries/aerospace-defense
34. Customer Case Study: RTI - InfluxData, accessed July 5, 2025, https://get.influxdata.com/rs/972-GDU-533/images/Customer_Case_Study_RTI.pdf
35. AMQP vs Data Distribution Service (DDS) - what is better? - Quora, accessed July 5, 2025, https://www.quora.com/AMQP-vs-Data-Distribution-Service-DDS-what-is-better
36. General Atomics MQ-9 Reaper - Wikipedia, accessed July 5, 2025, https://en.wikipedia.org/wiki/General_Atomics_MQ-9_Reaper
37. General Atomics' MQ-9B ready for game-changing AEW capability to strengthen European defense, accessed July 5, 2025, https://breakingdefense.com/2025/06/general-atomics-mq-9b-ready-for-game-changing-aew-capability-to-strengthen-european-defense/
38. MQ-9B SkyGuardian | General Atomics Aeronautical Systems Inc., accessed July 5, 2025, https://www.ga-asi.com/remotely-piloted-aircraft/mq-9b-skyguardian
39. MQ-9 Reaper > Air Force > Fact Sheet Display - AF.mil, accessed July 5, 2025, https://www.af.mil/About-Us/Fact-Sheets/Display/Article/104470/mq-9-reaper/
40. MQ-9A Reaper (Predator B) | General Atomics Aeronautical Systems Inc., accessed July 5, 2025, https://www.ga-asi.com/remotely-piloted-aircraft/mq-9a
41. GA-ASI Demonstrates BLOS Command and Control Over HF Using MQ-9 | General Atomics, accessed July 5, 2025, https://www.ga.com/ga-asi-demonstrates-blos-command-control-over-hf-using-mq-9
42. Robot Operating System 2 (ROS 2) Architecture | by Huseyin Kutluca - Medium, accessed July 5, 2025, https://medium.com/software-architecture-foundations/robot-operating-system-2-ros-2-architecture-731ef1867776
43. ROS 2 Architecture Overview - Automatic Addison, accessed July 5, 2025, https://automaticaddison.com/ros-2-architecture-overview/
44. ROS2 part 4 - ROS2 IPC, DDS, Topics, Services, Actions and Interfaces - RoboticsUnveiled, accessed July 5, 2025, https://www.roboticsunveiled.com/ros2-ipc-dds-topics-services-actions-interfaces/
45. DDS implementations - ROS 2 Documentation: Iron documentation, accessed July 5, 2025, https://docs.ros.org/en/iron/Installation/DDS-Implementations.html
46. rticommunity/rticonnextdds-usecases-medical: This RTI ... - GitHub, accessed July 5, 2025, https://github.com/rticommunity/rticonnextdds-usecases-medical
47. Comparison of MQTT and DDS as M2M Protocols for the Internet of Things - SlideShare, accessed July 5, 2025, https://www.slideshare.net/slideshow/comparison-of-mqtt-and-dds-as-m2m-protocols-for-the-internet-of-things/22137433
48. Evaluating DDS, MQTT, and ZeroMQ Under Different IoT Traffic Conditions - Distributed Object Computing (DOC) Group for DRE Systems, accessed July 5, 2025, http://www.dre.vanderbilt.edu/~gokhale/WWW/papers/M4IoT2020.pdf
49. Compare RTI Connext DDS vs. RabbitMQ in 2025 - Slashdot, accessed July 5, 2025, https://slashdot.org/software/comparison/Connext-DDS-vs-RabbitMQ/


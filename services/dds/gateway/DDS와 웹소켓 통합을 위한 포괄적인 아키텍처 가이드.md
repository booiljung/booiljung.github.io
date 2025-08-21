# DDS와 웹소켓 통합을 위한 포괄적인 아키텍처 가이드

## 요약 (Executive Summary)

본 보고서는 실시간 데이터 중심 미들웨어인 데이터 분산 서비스(Data Distribution Service, DDS)와 웹 네이티브 실시간 통신 프로토콜인 웹소켓(WebSocket)을 통합하는 브리지 기술에 대한 포괄적인 아키텍처 가이드를 제공합니다. DDS는 산업 자동화, 항공우주, 로보틱스 등 미션 크리티컬 시스템의 핵심 데이터 백본으로 자리 잡았으며, 웹소켓은 모든 최신 웹 브라우저에서 지원되는 양방향 실시간 통신 표준입니다. 이 두 기술을 연결하려는 요구는 DDS 시스템의 운영 데이터를 원격지 웹 대시보드에서 모니터링하거나, 웹 기반 HMI(Human-Machine Interface)를 통해 시스템을 제어하려는 전략적 필요성에서 비롯됩니다.

본 분석의 핵심은 DDS와 웹소켓 간의 근본적인 패러다임 불일치를 해결하는 데 있습니다. DDS는 데이터 중심(Data-Centric) 철학에 기반한 탈중앙화된 자동 발견(Automatic Discovery) 모델을 사용하는 반면, 웹소켓은 특정 서버 주소에 연결하는 지점 간(Point-to-Point) 연결 지향(Connection-Oriented) 모델을 따릅니다. 이 간극을 메우기 위한 유일하고 필수적인 아키텍처 패턴은 '게이트웨이(Gateway)' 또는 '프록시(Proxy)'입니다. 이 게이트웨이는 DDS 도메인에 완전한 참여자(Participant)로 참여함과 동시에, 웹 클라이언트들에게는 웹소켓 서버 역할을 수행하는 이중 정체성을 가집니다.

성공적인 DDS-웹소켓 브리지 구축에는 세 가지 핵심 기술적 과제가 수반됩니다. 첫째, **데이터 변환(Data Transformation)**입니다. DDS의 고효율 바이너리 직렬화 형식인 CDR(Common Data Representation)을 웹 친화적인 JSON(JavaScript Object Notation) 형식으로 상호 변환해야 합니다. 둘째, **QoS 매핑(QoS Mapping)**입니다. DDS의 풍부하고 세분화된 서비스 품질(Quality of Service) 정책(예: `RELIABILITY`, `DURABILITY`, `HISTORY`)을 웹소켓 통신 환경에서 의미 있는 동작으로 '에뮬레이션(emulation)'해야 합니다. 이는 게이트웨이 내부에 정교한 캐싱, 버퍼링, 상태 관리 로직 구현을 요구합니다. 셋째, **보안 모델 통합(Security Model Integration)**입니다. DDS의 PKI(Public Key Infrastructure) 기반 분산 보안 모델을 웹 표준인 WSS(TLS) 및 토큰 기반 인증 체계와 안전하게 연동시켜야 합니다.

본 보고서는 이러한 과제에 대한 해결책을 제시하며, RTI Web Integration Service와 같은 상용 솔루션과 eProsima Integration Service 같은 오픈소스 프레임워크를 심층적으로 비교 분석합니다. RTI 솔루션은 즉시 사용 가능한 안정성과 전문적인 지원을 제공하는 반면, eProsima 솔루션은 높은 유연성과 확장성, 비용 효율성을 장점으로 가집니다.

결론적으로, 성공적인 DDS-웹소켓 브리지는 단순한 프로토콜 변환기가 아니라, 상태를 관리하고, QoS를 에뮬레이션하며, 두 보안 영역을 중개하는 지능형 미들웨어 구성요소입니다. 따라서 아키텍트와 시스템 엔지니어는 프로젝트의 요구사항, 예산, 기술 역량을 고려하여 적합한 솔루션을 신중하게 선택하고, 특히 확장성과 보안 설계에 중점을 두어 게이트웨이를 구축해야 합니다.



## 1.  기본 기술: 두 프로토콜 이야기



DDS와 웹소켓을 연결하는 브리지 기술을 이해하기 위해서는 먼저 각 기술의 근본적인 철학과 아키텍처를 명확히 파악해야 합니다. 두 기술은 서로 다른 문제 영역을 해결하기 위해 탄생했으며, 이러한 차이점이 브리지 설계의 복잡성을 결정하는 핵심 요소가 됩니다.



### 1.1  데이터 분산 서비스 (DDS): 실시간 시스템을 위한 데이터 중심 백본



DDS(Data Distribution Service)는 OMG(Object Management Group)에서 표준화한 데이터 중심(Data-Centric) 발행-구독(Publish-Subscribe) 모델의 미들웨어입니다.1 이 기술의 핵심 목표는 분산된 실시간 시스템 환경에서 신뢰성 있고, 고성능이며, 상호 운용 가능한 데이터 교환을 가능하게 하는 것입니다.2



#### 1.1.1 핵심 철학: 글로벌 데이터 공간



DDS의 가장 중요한 개념은 '글로벌 데이터 공간(Global Data Space)'이라는 추상화입니다.3 이는 분산된 네트워크 전체를 하나의 거대한 공유 데이터베이스처럼 보이게 만듭니다. 애플리케이션은 네트워크의 복잡성이나 다른 애플리케이션의 위치 및 존재 여부를 알 필요 없이, 마치 로컬 메모리에 데이터를 읽고 쓰는 것처럼 이 글로벌 데이터 공간과 상호작용합니다.4 정보를 생성하는 노드(발행자)는 특정 '토픽(Topic)'에 '샘플(Sample)'을 발행하고, 해당 토픽에 관심이 있다고 선언한 노드(구독자)에게 DDS 미들웨어가 자동으로 데이터를 전달합니다.5 이러한 모델은 애플리케이션 간의 결합도를 극적으로 낮추어 모듈화되고 유연한 시스템 설계를 가능하게 합니다.



#### 1.1.2 주요 아키텍처 엔티티



DDS 시스템은 다음과 같은 표준화된 엔티티들로 구성됩니다 5:

- **DomainParticipant**: 애플리케이션이 특정 DDS 도메인(통신 영역)에 참여하기 위한 진입점입니다.5
- **Topic**: 데이터가 교환되는 채널로, 고유한 이름과 데이터 타입(예: IDL로 정의된 구조체)으로 정의됩니다.5
- **DataWriter**: 특정 토픽에 데이터를 발행(publish)하는 역할을 담당하는 엔티티입니다.5
- **DataReader**: 특정 토픽으로부터 데이터를 구독(subscribe)하는 역할을 담당하는 엔티티입니다.5
- **Publisher** 및 **Subscriber**: 각각 `DataWriter`와 `DataReader`를 생성하고 관리하는 팩토리(Factory) 및 컨테이너 객체입니다.5



#### 1.1.3 데이터 중심성(Data-Centricity)과 실시간 성능



DDS는 단순히 메시지를 전달하는 메시지 중심(Message-Centric) 미들웨어(예: MQTT)와 근본적으로 다릅니다. DDS는 데이터의 스키마(구조)와 내용을 이해합니다.9 이를 통해 데이터 값에 기반한 필터링(Content-Based Filtering)이나 데이터의 최신 상태 관리와 같은 강력한 기능을 제공할 수 있습니다.3

실시간 성능을 보장하기 위해 DDS는 일반적으로 TCP 대신 UDP 전송 프로토콜을 사용하며, 특히 다수의 구독자에게 효율적으로 데이터를 전파하기 위해 IP 멀티캐스트를 활용합니다.8 다른 벤더의 DDS 구현 간 상호 운용성은 RTPS(Real-Time Publish-Subscribe)라는 표준 와이어 프로토콜(wire protocol)을 통해 보장됩니다.8



#### 1.1.4 서비스 품질 (Quality of Service, QoS)



QoS는 DDS의 가장 강력하고 핵심적인 기능입니다. 이는 데이터 배포의 모든 측면을 제어하는 풍부한 정책들의 집합입니다.3 개발자는 QoS 정책을 통해 데이터의 신뢰성, 내구성, 이력 관리, 전송 우선순위 등을 매우 세밀하게 설정할 수 있습니다. 주요 QoS 정책은 다음과 같습니다 13:

- **RELIABILITY**: 데이터 전송을 보장할지(RELIABLE) 또는 최선 노력(BEST_EFFORT)으로 전송할지를 결정합니다.
- **DURABILITY**: 발행자가 사라진 후에도 데이터를 보존하여 늦게 참여하는(late-joining) 구독자에게 전달할지 여부를 결정합니다.
- **HISTORY**: 각 데이터 인스턴스에 대해 몇 개의 과거 샘플을 저장할지를 제어합니다.
- **DEADLINE**: 데이터가 주기적으로 갱신되어야 하는 최대 시간 간격을 정의합니다.



### 1.2  웹소켓: 실시간 양방향 웹 통신을 위한 표준



웹소켓은 IETF에 의해 RFC 6455로 표준화된 통신 프로토콜로, 단일 TCP 연결 상에서 전이중(full-duplex) 양방향 통신 채널을 제공합니다.15 이는 웹 환경에서 실시간 상호작용을 구현하기 위한 사실상의 표준 기술입니다.



#### 1.2.1 연결 수립: HTTP 핸드셰이크



웹소켓 연결은 일반적인 HTTP 요청으로 시작됩니다. 클라이언트는 서버에 `Upgrade: websocket` 및 `Connection: Upgrade` 헤더를 포함한 HTTP GET 요청을 보냅니다.18 서버가 이 요청을 수락하면, 상태 코드 

`101 Switching Protocols`로 응답하며, 이 시점부터 해당 TCP 연결은 HTTP 프로토콜에서 웹소켓 프로토콜로 전환됩니다.18 이 핸드셰이크 과정 덕분에 웹소켓은 표준 웹 포트인 80(ws)과 443(wss)을 사용할 수 있어 기존 방화벽 및 프록시와 높은 호환성을 가집니다.17



#### 1.2.2 상태 기반의 연결 지향 통신



상태가 없는(stateless) HTTP와 달리 웹소켓 연결은 상태를 유지합니다(stateful).15 한번 수립된 연결은 클라이언트나 서버가 명시적으로 종료하기 전까지 계속 유지됩니다. 이 지속적인 연결을 통해 서버는 클라이언트의 요청 없이도 언제든지 데이터를 클라이언트로 푸시(push)할 수 있으며, 이로 인해 낮은 지연 시간으로 실시간 데이터 교환이 가능해집니다.18 이러한 특징은 실시간 채팅, 온라인 게임, 금융 데이터 피드, 협업 도구와 같은 애플리케이션에 이상적입니다.15



#### 1.2.3 메시지 기반 프레이밍 및 보안



웹소켓은 TCP의 바이트 스트림 위에 메시지 기반의 프레이밍(framing) 메커니즘을 정의합니다. 이를 통해 텍스트 또는 바이너리 형식의 개별 메시지를 전송할 수 있습니다.17 또한, 연결 유지를 위한 

`ping`/`pong` 하트비트나 정상적인 연결 종료를 위한 `close`와 같은 제어 프레임도 포함하고 있습니다.21

보안은 WSS(WebSocket Secure) 프로토콜을 통해 제공됩니다. 이는 웹소켓 통신을 TLS(Transport Layer Security) 위에 계층화하여 전송되는 데이터를 암호화하고 무결성을 보장하는 방식입니다.17



#### 1.2.4 근본적인 아키텍처 불일치 분석



DDS와 웹소켓의 핵심 특성을 살펴보면, 두 기술을 통합하는 것이 단순한 작업이 아님을 알 수 있습니다. 가장 근본적인 차이는 **'발견 대 연결(Discovery vs. Connection)'** 패러다임에 있습니다. DDS는 탈중앙화된 환경에서 동적이고 자동적인 발견 메커니즘을 기반으로 합니다.2 참여자들은 중앙 브로커 없이 서로를 발견하고 데이터를 교환합니다. 이 발견 과정은 종종 UDP 멀티캐스트에 의존합니다. 반면, 웹소켓은 클라이언트가 사전에 알고 있는 특정 서버의 URL에 명시적으로 연결을 요청하는 중앙 집중적인 클라이언트-서버 모델입니다.17 웹소켓 클라이언트는 DDS의 동적 발견 프로토콜에 네이티브하게 참여할 수 없습니다.

이러한 이유로, 두 시스템을 연결하는 브리지는 단순한 편의 기능이 아니라 아키텍처적으로 반드시 필요한 구성 요소가 됩니다. 브리지는 DDS 네트워크에서는 DDS 참여자로서, 웹 환경에서는 웹소켓 서버로서의 역할을 동시에 수행하며 이 발견의 간극을 메워야 합니다. 웹 클라이언트는 브리지에 연결하고, 브리지는 클라이언트의 요청에 따라 DDS 세계에서 필요한 데이터를 발견하고 구독하여 전달하는 중개자 역할을 수행합니다.

또한, '데이터'를 바라보는 관점 자체도 다릅니다. DDS에서 데이터는 타입, 키, 생명주기, 그리고 다양한 QoS 정책이 결합된 풍부한 컨텍스트를 가진 객체입니다.3 반면, 웹소켓에서 메시지는 단순히 바이트 프레임일 뿐입니다.21 이 풍부한 DDS의 컨텍스트는 웹소켓 전송 계층에서는 손실됩니다. 따라서 단순한 데이터 포워딩을 넘어, 정교한 브리지는 DDS 데이터를 JSON과 같은 자기 서술적인 형식으로 직렬화하고, QoS나 토픽 이름과 같은 메타데이터를 전달하기 위한 별도의 상위 프로토콜을 웹소켓 위에서 정의해야 합니다.28 이는 웹 클라이언트에게 'DDS-lite'와 같은 경험을 제공하기 위한 필수적인 과정입니다.



## 2.  브리지의 필요성: 실시간 시스템과 웹의 연결



DDS와 웹소켓이라는 이질적인 두 기술을 연결하려는 노력은 단순한 기술적 호기심을 넘어, 명확한 비즈니스 및 운영상의 가치에 의해 추동됩니다. DDS가 지배적인 산업 현장의 데이터를 웹 기술의 유연성 및 접근성과 결합함으로써, 기존에는 불가능했던 새로운 가치를 창출할 수 있습니다.



### 2.1  전략적 사용 사례: 원격 모니터링, 웹 기반 HMI/SCADA, 클라우드 통합



DDS-웹소켓 브리지는 다양한 산업 분야에서 혁신적인 애플리케이션을 가능하게 합니다.

- **원격 모니터링 및 대시보드**: 가장 보편적이면서도 강력한 사용 사례입니다. DDS는 산업 자동화, 자율주행차, 로보틱스, 교통 관제 시스템과 같이 물리적으로 분산되어 있거나 접근이 어려운 시스템에서 널리 사용됩니다.30 DDS-웹소켓 브리지를 통해 이러한 시스템에서 발생하는 실시간 운영 데이터를 지리적 제약 없이 모든 웹 브라우저에서 접근 가능한 대시보드로 시각화할 수 있습니다.34 이는 관리자가 현장에 없더라도 시스템의 상태를 즉각적으로 파악하고 신속한 의사결정을 내릴 수 있도록 지원합니다.
- **웹 기반 HMI(Human-Machine Interface) 및 SCADA(Supervisory Control and Data Acquisition)**: 전통적인 HMI나 SCADA 시스템은 종종 전용 하드웨어나 특정 운영체제에서만 실행되는 씩 클라이언트(thick client) 애플리케이션 형태였습니다. DDS-웹소켓 브리지는 이러한 제약을 극복하고, 최신 웹 기술을 활용한 HMI를 구현할 수 있게 합니다.4 운영자는 태블릿이나 스마트폰의 웹 브라우저를 통해 산업 공정을 원격으로 감시하고 제어 명령을 내릴 수 있어, 유지보수 비용을 절감하고 운영 효율성을 극대화할 수 있습니다.38
- **클라우드 및 엔터프라이즈 통합**: DDS는 주로 엣지(edge) 또는 온프레미스(on-premise) 환경에서 데이터 교환의 중추 역할을 담당합니다. 브리지는 엣지에서 생성된 방대한 실시간 데이터를 클라우드 플랫폼으로 안전하고 효율적으로 전송하는 관문 역할을 합니다.4 클라우드로 전송된 데이터는 빅데이터 분석, 머신러닝 모델 학습, 장기 보관 등 엔터프라이즈 수준의 데이터 활용으로 이어질 수 있습니다. 게이트웨이는 엣지 컴퓨팅의 핵심 구성요소로서, 데이터를 클라우드 네이티브 애플리케이션과 통합하는 첫 단계를 담당합니다.
- **비-DDS 시스템과의 상호 운용성**: 자바스크립트와 같이 네이티브 DDS SDK가 제공되지 않는 프로그래밍 언어로 작성된 애플리케이션도 브리지를 통해 DDS 글로벌 데이터 공간에 참여할 수 있게 됩니다.39 이는 다양한 기술 스택을 사용하는 이기종 시스템 간의 통합을 용이하게 합니다.



### 2.2  게이트웨이 아키텍처 패턴: 필수적인 중개자



DDS와 웹소켓을 연결하는 가장 표준적인 아키텍처는 **게이트웨이(Gateway)** 또는 **프록시(Proxy)** 패턴을 사용하는 것입니다.41 이 게이트웨이는 두 프로토콜 세계의 경계에 위치하여 통신을 중개하는 독립적인 서비스입니다.

- **데이터 흐름 (DDS에서 웹으로)**:
  1. 게이트웨이는 DDS 도메인 내에서 특정 DDS `Topic`을 구독하기 위해 DDS `DataReader`를 생성합니다.
  2. `DataReader`가 DDS 네트워크로부터 데이터 샘플을 수신합니다.
  3. 게이트웨이는 수신된 데이터(CDR 형식)를 웹 클라이언트가 이해할 수 있는 형식(예: JSON)으로 변환합니다.
  4. 변환된 데이터를 해당 토픽을 구독하고 있는 웹 클라이언트들과 연결된 웹소켓을 통해 푸시(push)합니다.28
- **데이터 흐름 (웹에서 DDS로)**:
  1. 웹 클라이언트는 제어 명령이나 데이터를 JSON 객체 등의 형식으로 웹소켓 연결을 통해 게이트웨이에 전송합니다.
  2. 게이트웨이는 웹소켓 메시지를 수신하고, 이를 해당 DDS `Topic`의 데이터 타입에 맞게 변환합니다.
  3. 변환된 데이터를 CDR 형식으로 직렬화합니다.
  4. 게이트웨이 내의 DDS `DataWriter`를 사용하여 해당 데이터를 DDS 글로벌 데이터 공간에 발행합니다.28
- **상태 관리**: 게이트웨이는 본질적으로 상태를 가지는(stateful) 컴포넌트입니다. 어떤 웹소켓 클라이언트가 어떤 DDS 토픽을 구독하고 있는지, 그리고 각 클라이언트에게 어디까지 데이터를 전송했는지 등의 상태 정보를 반드시 유지하고 관리해야 합니다.44 이 상태 관리는 게이트웨이의 확장성과 장애 복원력을 설계할 때 가장 중요한 고려사항 중 하나입니다.



### 2.3  패러다임 불일치 분석: 프로토콜, 데이터, 보안 비교



DDS와 웹소켓의 근본적인 차이점과 게이트웨이의 역할을 명확히 이해하기 위해, 두 기술을 여러 차원에서 비교 분석할 수 있습니다. 이 비교는 게이트웨이가 왜 단순한 데이터 전달자가 아니라 정교한 변환 및 에뮬레이션 로직을 갖춘 지능형 중개자여야 하는지를 명확하게 보여줍니다.



#### 2.3.1 표 1: DDS 대 웹소켓 - 비교 분석



| 기능 차원             | 데이터 분산 서비스 (DDS)                                 | 웹소켓 (WebSocket)                                           | 게이트웨이의 역할 (브리지의 책임)                            |
| --------------------- | -------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **패러다임**          | 데이터 중심 발행-구독 (Data-Centric Publish-Subscribe) 1 | 연결 지향, 메시지 기반 (Connection-Oriented, Message-Based) 15 | 프록시 참여자 역할. 데이터 중심 업데이트를 특정 연결을 위한 메시지로 변환. |
| **발견 (Discovery)**  | 자동, 동적, P2P (RTPS) 2                                 | 수동, 지점 간 (클라이언트가 알려진 URL에 연결) 17            | 발견의 간극을 메움. 게이트웨이가 DDS 엔티티를 발견하고 정의된 API를 통해 클라이언트에게 노출. |
| **전송 프로토콜**     | 주로 저지연을 위한 UDP/멀티캐스트 8                      | TCP 15                                                       | UDP와 TCP 사이를 중재하며, 필요 시 패킷 손실 및 순서 차이를 처리. |
| **데이터 모델**       | 강력한 타입, 스키마 정의 (IDL), 내용 인식 7              | 불투명한 바이트 프레임 (텍스트 또는 바이너리) 17             | 데이터 변환 (예: DDS CDR to JSON) 및 타입 정의 관리.         |
| **서비스 품질 (QoS)** | 풍부하고 세분화된 정책 (신뢰성, 내구성, 이력 등) 13      | 네이티브 지원 없음 (TCP의 신뢰성에 의존)                     | DDS QoS **에뮬레이션**. 유사한 동작을 제공하기 위해 캐싱, 버퍼링, 확인 응답 로직 구현. |
| **보안 모델**         | PKI 기반, 탈중앙화, 세분화된 접근 제어 46                | WSS (TLS)를 통한 전송 보안; 애플리케이션 레벨 인증 (토큰) 26 | 보안 경계 역할. 웹 클라이언트를 인증/인가하고 그들의 권한을 DDS 접근 규칙에 매핑. |

이 표는 아키텍트에게 매우 중요한 도구입니다. 이는 "두 기술은 다르다"는 막연한 인식을 넘어 "구체적으로 이러한 측정 가능한 방식으로 다르다"는 것을 명확히 보여줍니다. 아키텍트는 이 표를 사용하여 이해관계자들에게 왜 단순한 '파이프' 연결이 불가능하며, 변환, 에뮬레이션, 매핑 로직을 갖춘 정교한 게이트웨이가 필요한지를 설명할 수 있습니다. 예를 들어, '데이터 중심'과 '연결 중심'을 나란히 보면 패러다임의 불일치가 즉각적으로 드러나고, '풍부하고 세분화된 정책'과 '없음'을 비교하면 전체 QoS 매핑 과제가 얼마나 중요한지 알 수 있습니다. 따라서 이 표는 문제 정의, 요구사항 수집, 그리고 교육 자료의 역할을 동시에 수행하며 보고서의 가치를 높입니다.



## 3.  DDS-웹소켓 브리지의 해부: 기술 심층 분석



이 섹션은 게이트웨이 구축의 세 가지 핵심 기술적 과제를 심층적으로 분석합니다. 이는 이론을 넘어 실제 구현에서 마주하게 될 문제들과 그 해결 방안을 다루는, 본 보고서에서 가장 기술 밀도가 높은 부분입니다.



### 3.1  데이터 변환: DDS CDR에서 웹 친화적인 JSON으로





#### 3.1.1 변환의 필요성



DDS는 네트워크 상에서 데이터를 효율적으로 전송하기 위해 CDR(Common Data Representation)이라는 바이너리 직렬화 형식을 사용합니다.49 CDR은 데이터 크기가 작고 파싱 속도가 빨라 임베디드 및 실시간 시스템에 최적화되어 있습니다. 반면, 웹 클라이언트, 특히 자바스크립트 환경은 사람이 읽기 쉽고 구조가 명확한 텍스트 기반의 JSON(JavaScript Object Notation) 형식을 다루는 데 최적화되어 있습니다.50 따라서 DDS의 CDR 데이터를 웹 클라이언트가 직접 사용하는 것은 거의 불가능하며, 반드시 중간에서 변환 과정이 필요합니다.



#### 3.1.2 IDL의 역할



데이터 변환의 핵심은 데이터의 구조를 이해하는 것입니다. DDS에서는 IDL(Interface Definition Language)을 사용하여 데이터 타입을 정의합니다.7 예를 들어, 센서 데이터를 나타내는 구조체를 IDL로 정의하면, DDS 미들웨어는 이 정의를 바탕으로 데이터를 직렬화하고 역직렬화합니다. 게이트웨이가 CDR 버퍼를 올바르게 해석하고 각 필드를 JSON 객체의 키-값 쌍으로 매핑하기 위해서는, 이 IDL 정의에 접근할 수 있어야 합니다. 런타임에 타입 정보를 다룰 수 있는 DDS-XTypes와 같은 표준은 이러한 동적 변환을 더욱 용이하게 합니다.51



#### 3.1.3 구현 메커니즘



데이터 변환 프로세스는 구체적으로 다음과 같은 단계를 거칩니다:

1. 게이트웨이의 `DataReader`가 DDS 네트워크로부터 원시 데이터 샘플(CDR 버퍼)을 수신합니다.
2. 게이트웨이는 해당 토픽의 `TypeCode` 정보(IDL로부터 파생됨)를 사용하여 `DDS_DynamicData`와 같은 동적 데이터 객체를 인스턴스화하고, CDR 버퍼의 내용을 이 객체에 채워 넣습니다.49
3. 채워진 `DynamicData` 객체를 JSON 포맷터(formatter)에 전달하여 JSON 문자열로 변환합니다.

RTI와 같은 상용 솔루션은 이러한 변환을 위한 내부 API(`DDS_DynamicDataFormatter_to_json`)를 제공하여 개발을 용이하게 합니다.49 NanoMQ DDS Proxy와 같은 오픈소스 솔루션은 IDL 파일로부터 필요한 JSON 직렬화/역직렬화 코드를 자동으로 생성해주는 

`idl-serial-code-gen`과 같은 유틸리티를 제공하기도 합니다.53



#### 3.1.4 역방향 변환 (JSON에서 CDR로)



웹 클라이언트가 DDS 네트워크에 데이터를 발행해야 하는 경우(예: 제어 명령 전송), 게이트웨이는 역방향 변환을 수행해야 합니다.

1. 웹소켓을 통해 들어온 JSON 메시지를 파싱합니다.
2. 게이트웨이는 알려진 타입 정의(IDL)에 따라 JSON 데이터의 유효성을 검사합니다.
3. 유효한 데이터를 `DynamicData` 객체에 채웁니다.
4. 이 `DynamicData` 객체를 CDR 버퍼로 직렬화합니다.
5. 최종적으로 게이트웨이의 `DataWriter`가 이 CDR 버퍼를 DDS 네트워크에 발행합니다.



### 3.2  QoS의 난제: DDS 정책을 게이트웨이 동작에 매핑하기



이것은 브리지 로직 설계에서 가장 어렵고 중요한 부분입니다. 웹소켓 프로토콜 자체에는 DDS와 같은 정교한 QoS 개념이 존재하지 않습니다. 따라서 게이트웨이는 DDS의 QoS 정책을 웹소켓으로 직접 '번역'할 수 없으며, 대신 게이트웨이 내부의 로직과 상태 관리를 통해 그 효과를 **'에뮬레이션(emulation)'** 해야 합니다.14 이는 각 QoS 정책의 의미를 이해하고, 그에 상응하는 동작을 게이트웨이 수준에서 구현해야 함을 의미합니다.



#### 3.2.1 표 2: DDS QoS 정책과 게이트웨이 동작 매핑



이 표는 아키텍트를 위한 실질적인 가이드입니다. 추상적인 DDS QoS 정책을 게이트웨이에서 구현해야 할 구체적인 요구사항으로 변환합니다. "만약 내 DDS 시스템이 `RELIABILITY`와 `HISTORY`를 사용한다면, 웹 클라이언트에게 이를 지원하기 위해 게이트웨이에 어떤 코드와 상태를 구축해야 하는가?"라는 질문에 대한 답을 제공합니다. 이는 즉시 실행 가능한 설계 지침이 됩니다.

| DDS QoS 정책               | DDS 정의 (DDS에서의 의미)                                    | 게이트웨이 구현 전략 (브리지가 에뮬레이트하는 방법)          | 관련 자료 |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------- |
| **RELIABILITY**            | `RELIABLE`: `DataWriter`에서 `DataReader`로의 모든 샘플 전송을 보장. ACK/NACK 메커니즘 포함. 13 | 게이트웨이의 내부 `DataReader`는 `RELIABLE`로 설정. 웹소켓 링크에 대해서는 TCP의 신뢰성에 의존하거나, 더 강력한 보장을 위해 자체적인 애플리케이션 레벨 ACK 메커니즘을 구현. 웹 클라이언트가 연결 해제된 경우 메시지를 버퍼링해야 함. | 54        |
| **HISTORY**                | `KEEP_LAST (depth)`: `DataReader` 캐시는 마지막 'depth'개의 샘플을 저장. `KEEP_ALL`: 리소스 한도까지 모든 샘플 저장. 14 | 게이트웨이는 구독된 각 `Topic`에 대한 메시지 버퍼(큐)를 유지해야 함. 이 버퍼의 크기는 `depth`를 반영하여 설정 가능해야 함. 이 버퍼는 늦게 참여하는 웹 클라이언트에게 데이터를 제공하거나 연결 재설정 후 데이터를 재전송하는 데 사용됨. | 55        |
| **DURABILITY**             | `TRANSIENT_LOCAL`: `DataWriter`는 늦게 참여하는 `DataReader`를 위해 마지막 샘플을 보관. 14 | *새로운* 웹소켓 클라이언트가 연결하고 토픽을 구독할 때, 게이트웨이는 내부 `DataReader`의 히스토리 캐시에서 `DURABILITY` 정책에 의해 보존된 데이터를 조회하여 즉시 새 클라이언트에게 전송해야 함. 이를 통해 늦은 참여자 동작을 시뮬레이션. | 14        |
| **LIFESPAN**               | 샘플은 만료 시간을 가짐. 만료된 샘플은 전달되지 않고 캐시에서 제거됨. 13 | 게이트웨이는 수신된 DDS 샘플에 타임스탬프를 기록하거나 소스 타임스탬프를 사용해야 함. 웹소켓 클라이언트로 메시지를 푸시하기 전에 `(현재 시간 - 샘플 시간) > lifespan`인지 확인. 만료된 메시지는 폐기해야 함. | 58        |
| **DEADLINE**               | 샘플 업데이트 간의 최대 시간에 대한 계약. 데드라인을 놓치면 알림을 트리거할 수 있음. 55 | 게이트웨이는 토픽의 DDS 샘플 간 시간을 모니터링할 수 있음. 데드라인이 누락되면, 웹소켓 클라이언트에게 "deadline_missed"와 같은 특별한 제어 메시지를 보내 웹 UI가 반응하도록 할 수 있음(예: 값 비활성화). | 55        |
| **CONTENT_FILTERED_TOPIC** | `DataReader`는 SQL과 유사한 필터 표현식을 지정하여 데이터의 일부만 수신할 수 있음. 5 | 게이트웨이는 클라이언트가 필터 표현식을 제공할 수 있는 API(예: 웹소켓 구독 시)를 노출해야 함. 게이트웨이의 내부 `DataReader`는 이 필터를 사용하여 DDS 도메인에서 *게이트웨이로*의 네트워크 트래픽을 줄임. | 6         |



### 3.3  보안 통합: DDS 보안 플러그인과 WSS 및 웹 인증의 결합



보안은 두 이질적인 시스템을 연결할 때 가장 중요한 고려사항 중 하나입니다. 게이트웨이는 신뢰할 수 있는 내부 DDS 네트워크와 잠재적으로 신뢰할 수 없는 외부 웹 환경 사이의 보안 경계 역할을 수행해야 합니다.



#### 3.3.1 DDS 보안 모델



DDS Security는 플러그인 기반의 포괄적인 보안 프레임워크를 정의합니다.47

- **인증(Authentication)**: 신원 CA(Identity CA)를 사용하는 PKI(Public Key Infrastructure)를 통해 DDS 참여자의 신원을 확인합니다.46 각 참여자는 고유한 인증서와 개인 키를 가집니다.
- **접근 제어(Access Control)**: 서명된 거버넌스(Governance) 및 권한(Permissions) 파일을 통해 특정 토픽에 대한 발행/구독 권한을 세밀하게 정의합니다.47
- **암호화(Cryptography)**: 네트워크를 통해 전송되는 데이터를 암호화하여 기밀성을 보장합니다.46



#### 3.3.2 웹 보안 모델



웹 환경의 보안은 주로 다음 두 가지에 의존합니다.

- **WSS (WebSocket Secure)**: TLS를 사용하여 전체 웹소켓 연결을 암호화하여 도청 및 변조를 방지합니다.17
- **인증/인가(Authentication/Authorization)**: 일반적으로 애플리케이션 계층에서 처리되며, JWT(JSON Web Token)나 OAuth 토큰과 같은 인증 토큰을 초기 HTTP 핸드셰이크 헤더에 포함시키거나, 웹소켓 연결 후 첫 메시지로 전송하여 사용자를 식별하고 권한을 부여합니다.29



#### 3.3.3 게이트웨이의 보안 초크 포인트(Choke Point) 역할



게이트웨이는 두 보안 모델을 중재하는 핵심적인 역할을 수행합니다.

1. **게이트웨이 자체의 보안**: 게이트웨이 애플리케이션은 보안이 활성화된 DDS 도메인에 참여하기 위해 필요한 모든 인증서, 개인 키, 권한 파일을 갖춘 완전한 보안 DDS 참여자여야 합니다.47
2. **안전한 웹 엔드포인트 노출**: 게이트웨이는 반드시 WSS 엔드포인트를 노출하여 웹 클라이언트와의 통신을 암호화해야 합니다.28
3. **웹 사용자 신원과 DDS 권한 매핑**: 이것이 가장 중요한 로직입니다. 게이트웨이는 웹 클라이언트로부터 인증 토큰(예: JWT)을 수신하고, 이 토큰의 유효성을 검증합니다. 그 후, 게이트웨이는 내부 정책(또는 외부 인증 서버와의 연동)을 통해 해당 웹 사용자가 수행할 수 있는 DDS 동작(특정 토픽 구독, 특정 토픽 발행 등)을 결정합니다. 예를 들어, '뷰어' 역할을 가진 사용자의 토큰은 데이터 구독만 허용하고, '운영자' 역할을 가진 사용자의 토큰은 제어 토픽에 대한 발행까지 허용할 수 있습니다. 이 권한 매핑 로직은 전적으로 게이트웨이 내에서 구현되어야 하는 핵심적인 커스텀 로직입니다.40



#### 3.3.4 취약점 고려사항



게이트웨이는 해커에게 매우 가치 있는 공격 표적이 됩니다. 게이트웨이가 손상되면 신뢰할 수 없는 웹 환경에서 중요한 실시간 DDS 네트워크로의 직접적인 침투 경로가 열릴 수 있습니다. 따라서 게이트웨이 자체의 보안을 강화하는 것이 무엇보다 중요합니다. 최근의 취약점 보고서들은 DDS 구현체 자체도 버퍼 오버플로우나 네트워크 증폭 공격과 같은 문제에 노출될 수 있음을 보여주며 61, 게이트웨이는 이러한 취약점에 대해 항상 최신 보안 패치를 적용해야 합니다.



## 4.  구현 쇼케이스: 솔루션 비교 분석



이론적 논의를 넘어, 실제 아키텍트가 선택할 수 있는 상용 및 오픈소스 솔루션들을 구체적으로 살펴보고 비교 분석합니다. 각 솔루션은 서로 다른 설계 철학과 장단점을 가지고 있어, 프로젝트의 요구사항에 따라 최적의 선택이 달라질 수 있습니다.



### 4.1  상용 등급 통합: RTI Web Integration Service (WIS)



RTI의 Web Integration Service(WIS)는 웹 서비스와 DDS 애플리케이션 간의 투명한 브리지를 제공하는 상용 제품입니다.40

- **아키텍처**: WIS는 독립 실행형 게이트웨이로, REST API와 웹소켓 API를 모두 제공하여 유연한 통합 방식을 지원합니다.40 이 이중 API 접근 방식은 시스템 관리와 실시간 데이터 스트리밍이라는 두 가지 다른 요구사항을 효과적으로 분리합니다.
- **API 설계**: DDS 엔티티(참여자, 토픽, 리더, 라이터 등)의 생성, 삭제, 조회와 같은 관리 작업은 주로 REST API를 통해 수행됩니다. 반면, 고성능의 실시간 데이터 발행 및 구독은 웹소켓 API에 최적화되어 있습니다.28
- **데이터 및 연결 흐름**: 웹소켓 클라이언트는 `BIND`라는 특별한 메시지를 사용하여 `bind_id`를 특정 DDS `DataReader` 또는 `DataWriter`에 연결합니다. `DataReader`가 바인딩되면, WIS는 수신된 DDS 데이터를 비동기적으로 클라이언트에게 `B_PUSH`합니다. 데이터를 DDS 네트워크에 쓰기 위해 클라이언트는 `B_REQUEST` 메시지를 사용합니다.28 이 바인딩 메커니즘은 웹소켓 연결의 효율성을 극대화합니다.
- **구성**: 시스템 구성은 XML 파일을 통해 이루어지며, 표준 DDS QoS 프로파일을 그대로 활용할 수 있어 기존 DDS 개발자에게 친숙한 환경을 제공합니다.65
- **라이선스 및 지원**: 상용 제품으로서, RTI는 전문적인 기술 지원과 사용자당 라이선스 모델을 제공합니다. 이는 미션 크리티컬 시스템에서 안정성과 예측 가능성을 확보해야 하는 기업에게 중요한 장점입니다.68



### 4.2  오픈소스의 유연성: eProsima Integration Service & WebSocket System Handle



eProsima의 Integration Service는 매우 유연하고 확장 가능한 플러그인 기반의 오픈소스 프레임워크입니다.52

- **아키텍처**: 이 서비스는 DDS-XTypes 기반의 공통 데이터 표현 언어를 사용하는 중앙 코어 엔진과, 각 프로토콜을 이 코어에 연결하는 '시스템 핸들(System Handles)'이라는 플러그인으로 구성됩니다.52 이 구조 덕분에 DDS와 웹소켓뿐만 아니라 FIWARE, ROS 등 다른 프로토콜까지 동일한 프레임워크 내에서 통합할 수 있습니다.
- **WebSocket System Handle**: 웹소켓 프로토콜을 위한 전용 시스템 핸들 플러그인입니다.29 이 핸들은 구성에 따라 웹소켓 서버 또는 클라이언트로 동작할 수 있어, 다양한 연결 시나리오를 지원합니다.29
- **구성**: 전체 시스템은 단일 YAML 파일을 통해 구성됩니다. 이 파일 안에 각 시스템(예: `fastdds`, `websocket_server`), 연결할 토픽, 그리고 시스템 간의 데이터 라우팅 규칙을 모두 정의합니다.29 이는 애플리케이션을 재컴파일하지 않고도 런타임에 구성을 변경할 수 있는 강력한 유연성을 제공합니다.
- **데이터 및 프로토콜**: 웹소켓을 통한 기본 통신 프로토콜은 JSON 기반이며, `advertise`, `subscribe`, `publish`와 같은 작업을 위한 명확한 메시지 구조를 정의하고 있습니다.29
- **라이선스 및 커뮤니티**: Integration Service와 관련 시스템 핸들은 Apache 2.0 라이선스 하에 배포되는 완전한 오픈소스입니다.73 이는 비용 효율적이고 높은 수준의 커스터마이징이 가능한 옵션을 제공하며, 필요 시 eProsima로부터 상용 지원을 받을 수도 있습니다.72



### 4.3  생태계 특화 솔루션: ROS 브리지 및 기타 프록시



특정 생태계나 목적에 맞춰진 브리지 솔루션들도 중요한 참고 자료가 됩니다.

- **ROS Foxglove Bridge**: ROS 2(내부적으로 DDS 사용) 환경의 데이터를 Foxglove Studio라는 시각화 도구에 연결하기 위해 특별히 설계되었습니다.75 자체적인 웹소켓 프로토콜을 사용하며, 고성능을 위해 C++ 노드로 구현되었습니다. 이는 특정 사용 사례에 최적화된 게이트웨이가 얼마나 효율적일 수 있는지를 보여주는 좋은 예입니다.
- **NanoMQ DDS Proxy**: DDS를 웹소켓에 직접 연결하는 대신, MQTT로 브리징하는 흥미로운 접근 방식을 취합니다.53 웹 클라이언트는 성숙한 MQTT 생태계의 웹소켓 지원을 통해 MQTT 브로커에 연결할 수 있습니다. 이 방식은 중간에 한 단계를 더 거치지만, MQTT의 장점을 활용할 수 있습니다. 이 프록시는 내부적으로 CycloneDDS를 사용하며, IDL-to-JSON 코드 생성기를 제공하고, MIT 라이선스로 배포됩니다.53
- **OpenDDS 기반 커스텀 빌드**: OpenDDS는 C++로 구현된 오픈소스 DDS입니다.78 RTI나 eProsima와 같이 즉시 사용 가능한 웹 통합 솔루션을 제공하지는 않지만, OpenDDS 라이브러리와 서드파티 웹소켓 라이브러리를 결합하여 완전히 맞춤화된 브리지를 구축하는 기반이 될 수 있습니다.79 이는 가장 높은 자유도를 제공하지만, 가장 많은 개발 노력을 필요로 합니다.



### 4.4 표 3: DDS-웹소켓 브리지 솔루션 - 기능 매트릭스



아키텍트가 프로젝트 요구사항에 따라 최적의 솔루션을 선택할 수 있도록, 주요 솔루션들의 특징을 표로 비교합니다. 이 표는 예산, 기술 지원 필요성, 커스터마이징 유연성, 기존 생태계와의 통합 등 핵심적인 의사결정 기준을 명확히 제시합니다.

| 기능               | RTI Web Integration Service                                  | eProsima Integration Service                                 | 커스텀 빌드 (예: OpenDDS 사용)                               |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **라이선스**       | 상용, 사용자당 라이선스 68                                   | 오픈소스 (Apache 2.0) 73                                     | 다양함 (OpenDDS는 오픈소스, 다른 구성 요소는 상용일 수 있음) |
| **아키텍처**       | REST/웹소켓 API를 갖춘 독립형 게이트웨이 40                  | 플러그인 방식 "시스템 핸들"을 갖춘 확장 가능한 프레임워크 71 | 맞춤형; DDS와 웹소켓 라이브러리의 수동 통합 필요             |
| **구성**           | XML 기반 QoS 및 애플리케이션 프로파일 65                     | 라우팅 및 시스템 구성을 위한 단일 YAML 파일 29               | 프로그래밍 방식 (코드 내) 및/또는 커스텀 구성 파일           |
| **주요 API**       | 관리용 REST, 데이터용 웹소켓 28                              | YAML 구성으로 경로 정의; 웹소켓은 JSON 기반 하위 프로토콜 사용 43 | 완전히 커스텀으로 정의된 API                                 |
| **확장성**         | DDS 통합에 제한됨                                            | 높음; 다른 프로토콜(FIWARE 등)을 위한 새 시스템 핸들 추가 가능 70 | 높음, 그러나 상당한 개발 노력 필요                           |
| **보안**           | 내장된 접근 제어 메커니즘, DDS 보안과 통합 40                | WebSocket-SH에서 TLS 및 토큰 기반 인증 지원; DDS 측에서는 DDS 보안에 의존 29 | 모든 계층에서 보안의 수동 구현 필요                          |
| **최적 사용 사례** | 웹 통합을 위한 지원되는 즉시 사용 가능한 솔루션이 필요한 기업 | 유연성, 다중 프로토콜 통합, 오픈소스 솔루션을 요구하는 시스템 | 기존 도구로 충족되지 않는 고유한 요구사항과 강력한 자체 개발 역량을 갖춘 프로젝트 |



## 5.  성능, 확장성 및 배포 고려사항



DDS-웹소켓 브리지를 프로덕션 환경에서, 특히 고빈도 데이터와 다수의 클라이언트를 대상으로 운영할 때 발생하는 실질적인 운영상의 과제들을 다룹니다. 성공적인 배포는 단순히 기능을 구현하는 것을 넘어, 성능과 안정성을 보장하는 설계를 요구합니다.



### 5.1  고빈도 데이터 및 동시 접속 클라이언트를 위한 게이트웨이 확장





#### 5.1.1 상태 기반 병목 현상(Stateful Bottleneck)



웹소켓 연결은 상태를 유지하는(stateful) 특성을 가집니다. 이는 일단 클라이언트가 특정 게이트웨이 인스턴스에 연결되면, 해당 연결이 지속되는 동안 모든 통신은 그 인스턴스를 통해야 함을 의미합니다. 따라서 단순한 라운드 로빈 방식의 로드 밸런서는 트래픽을 자유롭게 분산시킬 수 없으며, 게이트웨이가 단일 장애점(Single Point of Failure)이자 성능 병목 지점이 될 수 있습니다.44



#### 5.1.2 수직적 확장 대 수평적 확장



- **수직적 확장 (Scaling Up)**: 게이트웨이를 실행하는 단일 서버의 사양(CPU, 메모리)을 높이는 방식입니다. 관리가 단순하지만, 확장에 물리적 한계가 있고 장애에 취약합니다.44
- **수평적 확장 (Scaling Out)**: 여러 개의 게이트웨이 인스턴스를 병렬로 실행하는 방식입니다. 장애 복원력과 확장성이 뛰어나지만, 상당한 아키텍처적 복잡성을 수반합니다.44



#### 5.1.3 수평적 확장을 위한 아키텍처 패턴



수평적 확장을 성공적으로 구현하기 위해서는 다음과 같은 패턴을 고려해야 합니다.

- **스티키 세션 (Sticky Sessions)**: 로드 밸런서가 특정 클라이언트의 모든 요청을 항상 동일한 게이트웨이 인스턴스로 라우팅하도록 설정하는 방식입니다. 구현이 비교적 간단하지만, 특정 인스턴스에 부하가 집중될 수 있고, 해당 인스턴스 장애 시 세션이 유실되는 문제가 있습니다.45
- **공유 상태/백엔드 (Shared State / Backend)**: 가장 견고하고 진보된 접근 방식입니다. 각 게이트웨이 인스턴스는 상태를 가지지 않고(stateless), 모든 세션 및 구독 정보를 Redis나 다른 고성능 인메모리 데이터 스토어와 같은 공유 백엔드에 저장합니다.85 이 방식을 사용하면 어떤 게이트웨이 인스턴스든 모든 클라이언트의 요청을 처리할 수 있어, 진정한 의미의 로드 밸런싱과 원활한 장애 복구가 가능해집니다.



#### 5.1.4 데이터 압축 및 조절(Throttling)



웹 클라이언트는 종종 실시간 DDS 시스템에서 생성되는 원시 데이터 속도를 감당하지 못할 수 있습니다. 게이트웨이는 웹 클라이언트로 전송되는 데이터의 양을 줄이기 위한 전략을 구현해야 합니다.86

- **집계/다운샘플링 (Aggregation/Downsampling)**: N번째 샘플만 전송하거나 여러 샘플을 집계한 값을 전송합니다.
- **조절 (Throttling)**: 각 웹소켓 연결당 메시지 전송률을 제한합니다.
- **희소 계약 (Sparse Contracts)**: 전체 데이터 객체 대신 변경된 필드(diff)만 전송하여 데이터 크기를 줄입니다.



### 5.2  최적 성능을 위한 네트워크 및 시스템 튜닝





#### 5.2.1 커널 버퍼 크기 조정



특히 UDP를 통해 대용량 메시지를 처리할 때, 운영체제 커널의 네트워크 버퍼 크기(`net.core.rmem_max`)나 IP 단편화 타임아웃(`net.ipv4.ipfrag_time`)과 같은 파라미터를 충분히 늘려야 패킷 손실이나 전송 지연을 방지할 수 있습니다.54 게이트웨이가 실행되는 머신에서 이러한 커널 파라미터 튜닝은 매우 중요합니다.



#### 5.2.2 DDS QoS 튜닝



중요하지 않은 데이터에 대해서는 `RELIABILITY` QoS를 `BEST_EFFORT`로 설정하여 네트워크 오버헤드를 줄일 수 있습니다. 일부 DDS 구현체(예: Connext DDS)에서 제공하는 플로우 컨트롤러(flow controller)를 사용하면 대역폭 사용량을 세밀하게 조절하여 네트워크 폭주를 방지할 수 있습니다.54



#### 5.2.3 게이트웨이 리소스 관리



게이트웨이는 QoS 에뮬레이션을 위해 필요한 클라이언트별 버퍼 등 내부 리소스를 효율적으로 관리하도록 설계되어야 합니다. 특히 다수의 클라이언트가 연결될 경우, 메모리 사용량을 신중하게 관리하지 않으면 메모리 고갈로 인한 서비스 중단이 발생할 수 있습니다.



### 5.3  장애 복원력 및 고가용성 설계





#### 5.3.1 게이트웨이 장애 복구 (Failover)



공유 백엔드를 사용하는 수평 확장 아키텍처에서는, 로드 밸런서가 장애가 발생한 게이트웨이 인스턴스를 감지하고 트래픽 풀에서 자동으로 제거할 수 있어야 합니다. 클라이언트는 건강한 다른 인스턴스로 재연결하고, 새 인스턴스는 공유 백엔드에서 클라이언트의 세션 정보를 읽어와 서비스를 중단 없이 이어갈 수 있습니다.



#### 5.3.2 DDS 장애 복구 활용



게이트웨이는 DDS 자체의 강력한 장애 복구 기능을 활용해야 합니다. 예를 들어, DDS 시스템 내의 주 `DataWriter`에 장애가 발생하면 DDS는 자동으로 백업 `DataWriter`로 전환할 수 있습니다. 게이트웨이의 `DataReader`는 이러한 전환을 매끄럽게 인지하므로, 웹 클라이언트 측에서는 별도의 특별한 로직 없이도 데이터 스트림을 계속 수신할 수 있습니다.2



#### 5.3.3 웹소켓 재연결 로직



클라이언트 측 애플리케이션은 반드시 견고한 재연결 로직을 구현해야 합니다. 특히, 재연결 시도가 동시에 몰려 게이트웨이를 다운시키는 '서버 폭주(thundering herd)' 현상을 방지하기 위해, 무작위 지수 백오프(randomized exponential backoff)와 같은 전략을 사용하는 것이 필수적입니다.87



## 6.  전략적 권장사항 및 미래 전망



본 보고서의 분석을 바탕으로, DDS-웹소켓 브리지를 성공적으로 설계하고 배포하기 위한 전략적 권장사항을 제시하고, 관련 기술의 미래 발전 방향을 조망합니다.



### 6.1  DDS-웹소켓 브리지 설계 및 배포를 위한 모범 사례



- **데이터 모델 우선 설계**: 효율적인 브리지의 시작은 잘 정의된 데이터 모델에서 비롯됩니다. IDL을 사용하여 명확하고 간결한 데이터 구조를 설계하고, 데이터의 논리적 분할을 위해 키 필드(`@key`)를 적극적으로 활용하십시오.14 잘 정의된 키는 게이트웨이에서 콘텐츠 필터링을 통해 불필요한 데이터 전송을 줄이는 데 매우 효과적입니다.
- **QoS 매핑을 핵심 기능으로 취급**: QoS 지원을 부가 기능으로 취급해서는 안 됩니다. 게이트웨이의 버퍼링, 캐싱, 상태 관리 로직은 처음부터 요구되는 DDS QoS 정책을 에뮬레이션할 수 있도록 설계되어야 합니다.89 이는 브리지의 신뢰성과 성능을 결정하는 가장 중요한 요소 중 하나입니다.
- **게이트웨이 격리 및 보안 강화**: 게이트웨이는 신뢰할 수 있는 내부망과 외부망을 연결하는 중요한 보안 경계입니다. 따라서 DMZ와 같은 격리된 네트워크 세그먼트에 배포하는 것이 바람직합니다. 웹과 내부 DDS 네트워크 양쪽에서의 접근을 엄격히 통제하고, DDS 발견 프로토콜이 사용하는 포트(특히 멀티캐스트)가 절대 공용 인터넷에 노출되지 않도록 하십시오.33
- **초기 단계부터 확장성 계획**: 초기 배포 규모가 작더라도, 향후 클라이언트 연결 수나 데이터 양이 크게 증가할 것으로 예상된다면, 처음부터 게이트웨이를 상태 비저장(stateless)으로 설계하고 공유 백엔드를 사용하는 아키텍처를 고려하는 것이 장기적으로 유리합니다.44



### 6.2  올바른 솔루션 선택을 위한 의사결정 프레임워크



프로젝트의 특성에 맞는 최적의 솔루션을 선택하기 위한 가이드라인은 다음과 같습니다.

- **상용 솔루션(예: RTI WIS)을 선택해야 하는 경우**:
  - 안정적이고 전문적인 기술 지원이 보장되는 즉시 사용 가능한 솔루션이 필요할 때.
  - 라이선스 및 기술 지원에 대한 예산이 확보되어 있을 때.
  - 통합 요구사항이 순수하게 DDS를 웹 프로토콜로 브리징하는 데 집중되어 있을 때.
- **오픈소스 솔루션(예: eProsima IS)을 선택해야 하는 경우**:
  - 여러 프로토콜을 브리징해야 하는 등 최고의 유연성과 확장성이 필요할 때.
  - 오픈소스 도구를 관리하고 맞춤화할 수 있는 내부 기술 전문성을 보유하고 있을 때.
  - 특정 벤더에 대한 종속성을 피하고 싶을 때.
- **커스텀 빌드를 선택해야 하는 경우**:
  - 성능, 보안 또는 프로토콜 확장에 대해 기존 솔루션으로는 충족할 수 없는 매우 특수한 요구사항이 있을 때.
  - 게이트웨이를 직접 구축하고 장기적으로 유지보수할 수 있는 전담 개발팀이 있을 때.



### 6.3  웹에서의 DDS의 미래: 진화하는 표준과 신흥 기술



DDS와 웹의 통합은 계속해서 발전하고 있으며, 몇 가지 주목할 만한 동향이 있습니다.

- **DDS-WEB 표준**: OMG는 REST/웹소켓 게이트웨이 패턴을 표준화하는 "Web-Enabled DDS"(DDS-WEB) 사양을 제정했습니다.91 아직 보편적으로 채택되지는 않았지만, 이는 향후 벤더 솔루션 간의 상호 운용성을 향상시킬 것입니다. RTI의 WIS와 같은 구현체는 이미 이 표준에 부합하는 방향으로 발전하고 있습니다.28
- **DDS-XRCE**: IoT 센서와 같은 극도로 제한된 리소스 환경을 위한 DDS-XRCE(DDS for eXtremely Resource-Constrained Environments)는 주목할 만한 확장 기술입니다.2 XRCE 에이전트를 웹소켓 게이트웨이와 함께 배치하거나 통합함으로써, 아주 작은 임베디드 장치의 데이터까지도 웹으로 원활하게 브리징할 수 있는 가능성이 열립니다.
- **대안 전송 기술**: 현재는 웹소켓이 표준이지만, WebTransport와 같은 미래의 프로토콜은 DDS의 `BEST_EFFORT` QoS와 유사한 비신뢰성 데이터 스트림을 지원하는 등 추가적인 기능을 제공합니다.95 이러한 새로운 웹 프로토콜은 미래의 DDS-웹 브리징에서 더 효율적인 전송 계층으로 채택될 수 있습니다.

이러한 기술적 진화는 DDS의 강력한 실시간 데이터 처리 능력을 웹의 광범위한 접근성과 결합하여, 더욱 혁신적이고 연결된 시스템의 등장을 가속화할 것입니다.

#### **참고 자료**

1. DDS - velog, accessed July 4, 2025, https://velog.io/@wkdyd13/DDS
2. Data Distribution Service - Wikipedia, accessed July 4, 2025, https://en.wikipedia.org/wiki/Data_Distribution_Service
3. [ROS2]OMG Data-Distribution Service: Architectural Overview and OMG Data-Distribution Service: Architectural Update - velog, accessed July 4, 2025, https://velog.io/@protocol5/ROS2OMG-Data-Distribution-Service-Architectural-Overview-and-OMG-Data-Distribution-Service-Architectural-Update
4. What is DDS? - DDS Foundation, accessed July 4, 2025, https://www.dds-foundation.org/what-is-dds-3/
5. 데이터 분산 서비스 - 위키백과, 우리 모두의 백과사전, accessed July 4, 2025, [https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EB%B6%84%EC%82%B0_%EC%84%9C%EB%B9%84%EC%8A%A4](https://ko.wikipedia.org/wiki/데이터_분산_서비스)
6. Data Distribution Service (DDS) Architecture Overview - Twin Oaks Computing, Inc, accessed July 4, 2025, https://www.twinoakscomputing.com/Data_Distribution_Service_Overview
7. An Introduction to DDS with Examples, accessed July 4, 2025, https://dds-demonstrators.readthedocs.io/en/latest/Teams/1.Hurricane/DDSguide.html
8. [DDS] DDS와 RTPS - Interactics - 티스토리, accessed July 4, 2025, https://huroint.tistory.com/entry/DDS-DDS-RTPS
9. SDV 진짜 뼈대는 데이터 - RTI CTO가 말하는 DDS의 미래, accessed July 4, 2025, https://www.autoelectronics.co.kr/article/articleView.asp?idx=6233
10. DDS 보안기술 - 한국전자통신연구원, accessed July 4, 2025, https://ettrends.etri.re.kr/ettrends/131/0905001659/26-5_112-122.pdf
11. Interoperable Internet-Enabled DDS Applications - Object Computing, Inc., accessed July 4, 2025, https://objectcomputing.com/resources/publications/mnb/2019/06/20/interoperable-internet-enabled-dds-applications
12. DDS(Data Distribution Service)란 무엇인가? - Perbiz, accessed July 4, 2025, [http://perbiz.co.kr/data/DDS(Data%20Distribution%20Service)%EB%9E%80.pdf](http://perbiz.co.kr/data/DDS(Data Distribution Service)란.pdf)
13. OMG Data Distribution Service (DDS) - Object Management Group, accessed July 4, 2025, https://www.omg.org/spec/DDS/1.4/PDF
14. Data-Centric Programming Best Practices: - RTI, accessed July 4, 2025, https://www.rti.com/hubfs/docs/DDS_Best_Practices_WP.pdf
15. 웹소켓(WebSocket) 개념 이해하기!, accessed July 4, 2025, https://jaehoney.tistory.com/362
16. en.wikipedia.org, accessed July 4, 2025, [https://en.wikipedia.org/wiki/WebSocket#:~:text=WebSocket%20is%20a%20computer%20communications,as%20RFC%206455%20in%202011.](https://en.wikipedia.org/wiki/WebSocket#:~:text=WebSocket is a computer communications,as RFC 6455 in 2011.)
17. WebSocket - Wikipedia, accessed July 4, 2025, https://en.wikipedia.org/wiki/WebSocket
18. [기술 면접] 웹소켓 개념 및 동작 원리 - velog, accessed July 4, 2025, [https://velog.io/@jinyeong-afk/%EA%B8%B0%EC%88%A0-%EB%A9%B4%EC%A0%91-%EC%9B%B9%EC%86%8C%EC%BC%93-%EA%B0%9C%EB%85%90-%EB%B0%8F-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC](https://velog.io/@jinyeong-afk/기술-면접-웹소켓-개념-및-동작-원리)
19. WebSocket의 동작원리 - Flying Whale - 티스토리, accessed July 4, 2025, https://kellis.tistory.com/65
20. [IT기술] 웹소켓의 기본적인 이해와 활용방법 - webcodur, accessed July 4, 2025, https://webcodur.tistory.com/71
21. [Scratch.md] 웹소켓 WebSocket 에 대해서 알아보자 - 2031.10.19 - 티스토리, accessed July 4, 2025, https://vince-kim.tistory.com/40
22. What is WebSockets Protocol? Implementation and Use Cases - PubNub, accessed July 4, 2025, https://www.pubnub.com/guides/websockets/
23. 웹소켓의 동작 원리 및 실시간 애플리케이션에서의 활용 - F-Lab, accessed July 4, 2025, https://f-lab.kr/insight/understanding-websocket-and-its-application
24. [Web] 웹 소켓(Web Socket)란? (특징, 동작 과정, 사용 사례, 롱 폴링(LongPolling)과 차이점), accessed July 4, 2025, [https://yuna-ninano.tistory.com/entry/Web-%EC%9B%B9-%EC%86%8C%EC%BC%93Web-Socket%EB%9E%80-%ED%8A%B9%EC%A7%95-%EB%8F%99%EC%9E%91-%EA%B3%BC%EC%A0%95-%EC%82%AC%EC%9A%A9-%EC%82%AC%EB%A1%80-%EB%A1%B1-%ED%8F%B4%EB%A7%81LongPolling%EA%B3%BC-%EC%B0%A8%EC%9D%B4%EC%A0%90](https://yuna-ninano.tistory.com/entry/Web-웹-소켓Web-Socket란-특징-동작-과정-사용-사례-롱-폴링LongPolling과-차이점)
25. WebSocket이란? 개념과 동작 과정 (+socket.io, Polling, Streaming...) - Seize the day!, accessed July 4, 2025, [https://doozi0316.tistory.com/entry/WebSocket%EC%9D%B4%EB%9E%80-%EA%B0%9C%EB%85%90%EA%B3%BC-%EB%8F%99%EC%9E%91-%EA%B3%BC%EC%A0%95-socketio-Polling-Streaming](https://doozi0316.tistory.com/entry/WebSocket이란-개념과-동작-과정-socketio-Polling-Streaming)
26. 웹 소켓(WebSocket) 개념 이해 - 유아마루, accessed July 4, 2025, https://urmaru.com/7
27. DDS (Data Distribution Service) Core Concepts - GitHub, accessed July 4, 2025, https://github.com/jmmorato/openddsharp/blob/develop/Documentation/articles/dds_core_concepts.md/
28. 6. WebSocket API - RTI Web Integration Service 7.5.0 documentation - RTI Community, accessed July 4, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/services/web_integration_service/using_websocket_api.html
29. eProsima's Integration Service System Handle for WebSocket. - GitHub, accessed July 4, 2025, https://github.com/eProsima/WebSocket-SH
30. DDS (직접 디지털 파형합성) 이해하기 - NI, accessed July 4, 2025, https://www.ni.com/ko/shop/electronic-test-instrumentation/waveform-generators/understanding-direct-digital-synthesis--dds-.html
31. DDS 어플리케이션 개발 및 운영을 위한 플랫폼 설계 방안 - AWS, accessed July 4, 2025, https://journal-home.s3.ap-northeast-2.amazonaws.com/site/2020kics/presentation/0098.pdf
32. 소프트웨어 정의 차량을 위한 자동차 통신 프레임워크 리더 - RTI, accessed July 4, 2025, https://www.rti.com/hubfs/_Collateral/Executive-Briefs/RTI-executive-brief-Connectivity-Leader-for-SDV-KR.pdf
33. Analyzing the Data Distribution Service (DDS) Protocol for Critical Industries - Trend Micro, accessed July 4, 2025, https://www.trendmicro.com/vinfo/us/security/news/cybercrime-and-digital-threats/analyzing-the-data-distribution-service-dds-protocol-for-critical-industries
34. Revolutionize Your Dental Practice with DDS Dashboard – Subtitled for Accessibility, accessed July 4, 2025, https://www.youtube.com/watch?v=7NQ3jMvb4hg
35. Implementation of DDS Cloud Platform for Real-Time Data Acquisition of Sensors for a Legacy Machine - MDPI, accessed July 4, 2025, https://www.mdpi.com/2079-9292/11/13/2096
36. Creating a Cloud-Connected Medical Device Demo using Connext DDS, accessed July 4, 2025, https://www.rti.com/blog/cloud-connected-medical-device
37. Designing a SCADA System: A Practical Guide - Global Data Specialists - Arizona, Illinois, and Utah SCADA Solutions, accessed July 4, 2025, https://gbl-data.com/designing-a-scada-system-a-practical-guide/
38. Integration of the Data Distribution Service middleware protocol in an SCADA system - ijarcce, accessed July 4, 2025, https://ijarcce.com/wp-content/uploads/2015/10/IJARCCE-92.pdf
39. rticommunity/rtiwebintegrationservice-examples: This repository includes examples that illustrate how to use RTI Web Integration Service, an out-of-the-box solution for integrating web-based applications and services with RTI Connext™ DDS. - GitHub, accessed July 4, 2025, https://github.com/rticommunity/rtiwebintegrationservice-examples
40. RTI IS - Web Integration Service - Real-Time Innovations, accessed July 4, 2025, https://www.rti.com/products/is/web-integration-service
41. Navigating DDS: Basics, Limitations, and Integration with MQTT | by EMQ Technologies, accessed July 4, 2025, https://emqx.medium.com/navigating-dds-basics-limitations-and-integration-with-mqtt-944da6bde576
42. rticommunity/rticonnextdds-gateway: The RTI Gateway is a software component that allows integrating different types of connectivity protocols with DDS. Integration in this context means that data flows from different protocols are adapted to interface with DDS, establishing communication from and to DDS. - GitHub, accessed July 4, 2025, https://github.com/rticommunity/rticonnextdds-gateway
43. 1.1.5. ROS 2 - WebSocket bridge - Integration Service 3.1.0 ..., accessed July 4, 2025, https://integration-service.docs.eprosima.com/en/latest/examples/different_protocols/pubsub/ros2-websocket.html
44. How to scale WebSockets for high-concurrency systems - Ably Realtime, accessed July 4, 2025, https://ably.com/topic/the-challenge-of-scaling-websockets
45. Do websockets really scale badly? : r/devops - Reddit, accessed July 4, 2025, https://www.reddit.com/r/devops/comments/19624cb/do_websockets_really_scale_badly/
46. 8.1. Authentication plugin: DDS:Auth:PKI-DH - 3.2.2, accessed July 4, 2025, https://fast-dds.docs.eprosima.com/en/v3.2.2/fastdds/security/auth_plugin/auth_plugin.html
47. DDS Security - OpenDDS 3.28.1, accessed July 4, 2025, https://opendds.readthedocs.io/en/dds-3.28.1/devguide/dds_security.html
48. 웹소켓으로 개발하기 전 알아야 할 것들 - 요즘IT, accessed July 4, 2025, https://yozm.wishket.com/magazine/detail/1911/
49. Automatic DDS data to JSON for web integration service - RTI Community, accessed July 4, 2025, https://community.rti.com/forum-topic/automatic-dds-data-json-web-integration-service
50. Vodia adopts JSON for CDR and PMS integration, accessed July 4, 2025, https://web.vodia.com/blog/vodia-adopts-json-for-cdr-and-pms-integration
51. What's in the DDS Standard? - DDS Foundation, accessed July 4, 2025, https://www.dds-foundation.org/omg-dds-standard/
52. Integration Service - allowing integration of any protocol into DDS - ROS Discourse, accessed July 4, 2025, https://discourse.ros.org/t/integration-service-allowing-integration-of-any-protocol-into-dds/16135
53. DDS Proxy | NanoMQ Documentation, accessed July 4, 2025, https://nanomq.io/docs/en/latest/gateway/dds.html
54. DDS tuning information - ROS 2 Documentation: Foxy documentation, accessed July 4, 2025, https://docs.ros.org/en/foxy/How-To-Guides/DDS-tuning.html
55. 3.1.2.1. Standard QoS Policies - 3.2.2 - Fast DDS - eProsima, accessed July 4, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/dds_layer/core/policy/standardQosPolicies.html
56. Quality of Service - OpenDDS 3.28.1, accessed July 4, 2025, https://opendds.readthedocs.io/en/dds-3.28.1/devguide/quality_of_service.html
57. Setting DataWriter QosPolicies - RTI Community, accessed July 4, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/users_manual/users_manual/Setting_DataWriter_QosPolicies.htm
58. 4. Quality of Service - The Data Distribution Service Tutorial, accessed July 4, 2025, https://download.zettascale.online/www/docs/Vortex/html/ospl/DDSTutorial/qos.html
59. Interactively Configure DDS Interface - MATLAB & Simulink - MathWorks, accessed July 4, 2025, https://www.mathworks.com/help/dds/ug/interactively-configure-dds-interface.html
60. 1. Introduction - RTI Web Integration Service 7.5.0 documentation - RTI Community, accessed July 4, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/services/web_integration_service/introduction.html
61. Multiple Data Distribution Service (DDS) Implementations (Update A) - CISA, accessed July 4, 2025, https://www.cisa.gov/news-events/ics-advisories/icsa-21-315-02
62. Data Distribution Service: Exploring Vulnerabilities and Risks Part 2 | Trend Micro (US), accessed July 4, 2025, https://www.trendmicro.com/en_us/research/22/g/data-distribution-service-part-2.html
63. Security vulnerabilities on the Data Distribution Service (DDS) - Ubuntu, accessed July 4, 2025, https://ubuntu.com/blog/security-vulnerabilities-on-the-data-distribution-service-dds
64. Welcome to RTI Web Integration Service!, accessed July 4, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/services/web_integration_service/index.html
65. USER_QOS_PROFILES.xml - RTI Community, accessed July 4, 2025, https://d2vkrkwbbxbylk.cloudfront.net/sites/default/files/rti-examples/rticonnextdds-examples/examples/connext_dds/using_sequences/cs/USER_QOS_PROFILES.xml
66. USER_QOS_PROFILES.xml - RTI Community, accessed July 4, 2025, https://d2vkrkwbbxbylk.cloudfront.net/sites/default/files/rti-examples/rticonnextdds-examples/examples/connext_dds/keyed_data_advanced/c/USER_QOS_PROFILES.xml
67. RTI Connext DDS - HubSpot, accessed July 4, 2025, https://cdn2.hubspot.net/hubfs/1754418/RTI_Oct2016/PDF/RTI_CoreLibrariesAndUtilities_GettingStarted.pdf
68. Pricing | RTI - Real-Time Innovations, accessed July 4, 2025, https://www.rti.com/products/pricing
69. RTI DDS Connext® for UEI Users - Aerospace DAQ, Test, HIL - United Electronic Industries, accessed July 4, 2025, https://www.ueidaq.com/products/rti-dds-connext-for-uei-users
70. eProsima is launching Integration Service v3.0.0 - ROS Discourse, accessed July 4, 2025, https://discourse.ros.org/t/eprosima-is-launching-integration-service-v3-0-0/20515
71. Integration Service Core - Integration Service 3.1.0 documentation - eProsima, accessed July 4, 2025, https://integration-service.docs.eprosima.com/
72. eProsima/Integration-Service - GitHub, accessed July 4, 2025, https://github.com/eProsima/Integration-Service
73. Integration-Service/LICENSE at main - GitHub, accessed July 4, 2025, https://github.com/eProsima/Integration-Service/blob/main/LICENSE
74. eProsima: Middleware, Robots and AI, accessed July 4, 2025, https://www.eprosima.com/
75. ROS Foxglove bridge, accessed July 4, 2025, https://docs.foxglove.dev/docs/connecting-to-data/ros-foxglove-bridge
76. DDS and MQTT: Basics, Challenges and Integration Benefits | EMQ - EMQX, accessed July 4, 2025, https://www.emqx.com/en/blog/navigating-dds-basics-limitations-and-integration-with-mqtt
77. nanomq/nanomq_cli/dds2mqtt/dds_client.c at master - GitHub, accessed July 4, 2025, https://github.com/emqx/nanomq/blob/master/nanomq_cli/dds2mqtt/dds_client.c
78. OpenDDS, accessed July 4, 2025, https://opendds.org/
79. Introduction to OpenDDS, accessed July 4, 2025, https://opendds.org/about/articles/Article-Intro.html
80. OpenDDS Development Guidelines - Read the Docs, accessed July 4, 2025, https://opendds.readthedocs.io/en/latest/internal/dev_guidelines.html
81. Integration Service - eProsima, accessed July 4, 2025, https://www.eprosima.com/middleware/tools/dds-integration-service
82. 5. Basic QoS - RTI Connext Getting Started documentation - RTI Community, accessed July 4, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/getting_started_guide/cpp11/intro_qos.html
83. How to scale WebSockets to millions of connections - YouTube, accessed July 4, 2025, https://www.youtube.com/watch?v=vXJsJ52vwAA
84. WebSocket architecture best practices to design robust realtime system, accessed July 4, 2025, https://ably.com/topic/websocket-architecture-best-practices
85. How I Solved the WebSocket Scaling Problem Without Breaking the Bank | HackerNoon, accessed July 4, 2025, https://hackernoon.com/how-i-solved-the-websocket-scaling-problem-without-breaking-the-bank
86. Lessons Learned: WebSocketAPI at scale | by Martin Chaov | DraftKings Engineering, accessed July 4, 2025, https://medium.com/draftkings-engineering/lessons-learned-websocketapi-at-scale-604617a54cdb
87. Challenges of scaling WebSockets - DEV Community, accessed July 4, 2025, https://dev.to/ably/challenges-of-scaling-websockets-3493
88. Bridge WebSocket messages and Redux actions - GitHub, accessed July 4, 2025, https://github.com/compulim/redux-websocket-bridge
89. Best Practices Using RTI Connext DDS | PPT - SlideShare, accessed July 4, 2025, https://www.slideshare.net/slideshow/best-practices-using-rti-connext-dds/23624696
90. A Security Analysis of the Data Distribution Service (DDS) Protocol | Trend Micro, accessed July 4, 2025, https://documents.trendmicro.com/assets/white_papers/wp-a-security-analysis-of-the-data-distribution-service-dds-protocol.pdf
91. ROS and Web Clients - Google Groups, accessed July 4, 2025, https://groups.google.com/g/ros-sig-ng-ros/c/BfvcjD6Rsg0
92. About the Web-Enabled DDS Specification Version 1.0 - Object Management Group, accessed July 4, 2025, https://www.omg.org/spec/DDS-WEB/1.0/About-DDS-WEB
93. Safety Mechanisms Using the DDS Middleware in Software-Defined Cars, accessed July 4, 2025, https://www.nxp.jp/company/about-nxp/smarter-world-blog/BL-SAFETY-MECHANISMS
94. uXRCE-DDS (PX4-ROS 2/DDS Bridge) | PX4 Guide (main), accessed July 4, 2025, https://docs.px4.io/main/en/middleware/uxrce_dds.html
95. The WebSocket API (WebSockets) - Web APIs - MDN Web Docs, accessed July 4, 2025, https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API


# 차세대 IoT 통신을 위한 MQTT over QUIC

## 서론

사물 인터넷(IoT)의 폭발적인 성장은 전 세계적으로 수십억 개의 디바이스를 상호 연결하며, 산업, 생활, 교통 등 사회 전반에 걸쳐 혁신을 주도하고 있다. 이러한 거대한 네트워크를 지탱하기 위해서는 디바이스 간의 데이터를 효율적이고 신뢰성 있게 교환할 수 있는 통신 프로토콜의 역할이 절대적으로 중요하다. 특히, 제한된 전력과 낮은 연산 능력, 협소한 네트워크 대역폭을 가진 IoT 디바이스 환경에서는 경량화된 프로토콜이 필수적이다. 이러한 배경 속에서 MQTT(Message Queuing Telemetry Transport)는 발행/구독(Publish/Subscribe) 모델을 기반으로 한 단순하고 가벼운 구조 덕분에 IoT 통신의 사실상 표준(de facto standard)으로 확고히 자리매김했다.1

그러나 MQTT의 성공적인 확산 이면에는 근본적인 기술적 한계가 존재한다. MQTT 표준은 신뢰성 있는 양방향 연결을 제공하는 전송 계층 위에서 동작하도록 설계되었으며, 이로 인해 지난 수십 년간 인터넷 통신의 근간을 이루어 온 TCP/IP(Transmission Control Protocol/Internet Protocol)가 주로 사용되어 왔다.1 TCP는 안정적인 유선 네트워크 환경에서는 탁월한 성능을 보이지만, 오늘날 IoT의 주 무대가 되고 있는 모바일 및 무선 환경에서는 명백한 취약점을 드러낸다. 커넥티드 카(IoV, Internet of Vehicles), 스마트 팩토리의 산업용 IoT(IIoT)와 같이 네트워크 환경이 수시로 변하고 불안정한 시나리오에서 TCP는 잦은 연결 끊김, 높은 재연결 지연 시간, 그리고 패킷 손실 시 발생하는 HOL(Head-of-Line) 블로킹 문제로 인해 전체 시스템의 성능과 안정성을 저해하는 주요 원인이 된다.5

이러한 TCP의 고질적인 문제들을 해결하기 위한 노력의 결과로, Google이 처음 개발하고 이후 IETF(Internet Engineering Task Force)에 의해 표준화된 차세대 전송 프로토콜 QUIC(Quick UDP Internet Connections)이 강력한 대안으로 부상했다.9 QUIC는 TCP가 아닌 UDP(User Datagram Protocol)를 기반으로, 사용자 공간(User Space)에서 동작하며 현대 네트워크 환경이 요구하는 핵심 기능들을 내장하고 있다. 연결 설정에 필요한 시간을 획기적으로 단축하는 0-RTT/1-RTT 핸드셰이크, 단일 연결 내에서 여러 데이터 흐름을 독립적으로 처리하여 HOL 블로킹을 원천적으로 해결하는 다중 스트림, 클라이언트의 네트워크 주소가 변경되어도 연결을 유지하는 원활한 연결 마이그레이션(Connection Migration), 그리고 TLS 1.3을 통합한 강력한 내장 보안 기능은 QUIC를 차세대 인터넷 프로토콜 HTTP/3의 기반으로 만들었다.6

본 보고서는 경량 IoT 메시징의 표준인 MQTT를 차세대 전송 프로토콜인 QUIC 위에서 구현하는 'MQTT over QUIC' 기술에 대한 심층적인 고찰을 목적으로 한다. 각 프로토콜의 기본 원리 분석을 시작으로, 두 기술을 결합하는 구체적인 구현 방식, 다양한 네트워크 시나리오에서의 성능 벤치마크 비교 분석, 커넥티드 카와 산업용 IoT 등 핵심 응용 사례, 그리고 현재 기술 생태계의 성숙도와 향후 표준화 전망까지 다각도로 조명할 것이다. 이 종합적인 분석을 통해 MQTT over QUIC가 기존의 한계를 넘어 차세대 IoT 통신의 핵심 기술로 자리매김할 수 있는지, 그 잠재력과 현실적인 당면 과제를 종합적으로 평가하고자 한다.

## 1.  프로토콜의 이해: MQTT와 QUIC

MQTT over QUIC 기술을 정확히 이해하기 위해서는 그 근간을 이루는 두 프로토콜, MQTT와 QUIC의 핵심 원리와 특성을 먼저 파악해야 한다. MQTT는 애플리케이션 계층에서 메시지 교환의 논리를 정의하고, QUIC는 전송 계층에서 데이터 전달의 효율성과 신뢰성을 책임진다.

### 1.1  MQTT: 경량 메시징 프로토콜의 핵심

MQTT는 1999년 IBM의 Andy Stanford-Clark과 Arcom의 Arlen Nipper에 의해 처음 개발되었으며, 자원이 제한된 디바이스와 저대역폭, 고지연, 불안정한 네트워크를 위한 메시징 프로토콜로 설계되었다. 그 핵심적인 특징은 다음과 같다.

#### 1.1.1  발행/구독(Publish/Subscribe) 모델

MQTT는 전통적인 클라이언트-서버 모델과 달리, 메시지 브로커(Broker)를 중심으로 동작하는 발행/구독 패턴을 채택한다.1 이 모델에서 메시지를 보내는 클라이언트(발행자, Publisher)는 메시지를 수신하는 클라이언트(구독자, Subscriber)를 직접 알지 못한다. 대신, 발행자는 특정 주제(Topic)에 대한 메시지를 브로커에게 보내고, 브로커는 해당 주제를 미리 구독한 모든 구독자에게 메시지를 전달한다.14 이러한 비동기적 메시지 전달 방식은 송신자와 수신자 간의 시간적, 공간적, 동기적 결합도를 낮추어 시스템의 유연성과 확장성을 크게 향상시킨다. 예를 들어, 발행자는 구독자가 오프라인 상태이거나 존재하지 않더라도 메시지를 보낼 수 있으며, 하나의 발행자는 다수의 구독자에게 동시에 정보를 전파할 수 있다.

#### 1.1.2  핵심 구성요소

MQTT 아키텍처는 세 가지 주요 구성요소로 이루어진다.

- **MQTT 클라이언트 (Client)**: 네트워크를 통해 브로커에 연결되는 모든 디바이스 또는 애플리케이션을 의미한다. 클라이언트는 메시지를 발행하는 발행자의 역할과 메시지를 수신하는 구독자의 역할을 동시에 수행할 수 있다.1 예를 들어, 스마트 홈의 온도 센서는 온도 데이터를 발행하는 발행자이면서, 동시에 원격 제어 명령을 수신하는 구독자가 될 수 있다.
- **MQTT 브로커 (Broker)**: MQTT 시스템의 중심에 위치하는 서버로, 모든 메시지 흐름을 관리하고 중개한다. 브로커의 핵심 책임은 발행자로부터 모든 메시지를 수신하고, 메시지를 필터링하며, 어떤 클라이언트가 어떤 토픽을 구독하고 있는지 추적하여 정확한 수신자에게 메시지를 전달하는 것이다.1 브로커는 모든 통신이 집중되는 지점이므로, 시스템 전체의 성능과 안정성에 결정적인 영향을 미치며, 잠재적인 단일 실패 지점(Single Point of Failure)이 될 수도 있다.15
- **토픽 (Topic)**: MQTT에서 메시지를 분류하고 필터링하는 데 사용되는 주소 체계이다. 토픽은 슬래시(`/`)로 구분되는 계층적 구조를 가진 UTF-8 문자열로 표현된다 (예: `home/livingroom/temperature`). 구독자는 특정 토픽을 명시적으로 구독하거나, 와일드카드를 사용하여 여러 토픽을 한 번에 구독할 수 있다. 단일 레벨 와일드카드(`+`)는 하나의 토픽 레벨을 대체하며, 다중 레벨 와일드카드(`#`)는 여러 계층의 토픽 레벨을 대체한다. 이러한 유연한 토픽 구조는 마치 데이터베이스를 쿼리하는 것처럼 원하는 데이터를 효율적으로 필터링할 수 있게 해준다.1

#### 1.1.3  서비스 품질(Quality of Service, QoS)

MQTT는 애플리케이션의 요구사항과 네트워크의 신뢰도에 따라 메시지 전달 보장 수준을 선택할 수 있도록 세 가지 서비스 품질(QoS) 레벨을 제공한다. 이는 자원 낭비를 최소화하면서도 중요한 데이터의 전달을 보장하는 핵심적인 기능이다.1

- **`QoS 0 (At most once)`**: "최대 한 번 전달"을 의미하며, "fire and forget" 방식으로도 불린다. 클라이언트는 메시지를 한 번만 보내고 수신 확인을 기다리지 않는다. 네트워크 상황에 따라 메시지가 유실될 수 있지만, 프로토콜 오버헤드가 가장 적어 주기적으로 전송되는 비중요 센서 데이터 등에 적합하다.1
- **`QoS 1 (At least once)`**: "최소 한 번 전달"을 보장한다. 발행자는 브로커로부터 `PUBACK` 패킷을 수신할 때까지 메시지를 저장하고 재전송한다. 이로 인해 메시지는 반드시 전달되지만, 네트워크 지연 등으로 인해 동일한 메시지가 중복으로 수신될 수 있다. 수신 측은 중복 메시지를 처리할 수 있어야 한다.1
- **`QoS 2 (Exactly once)`**: "정확히 한 번 전달"을 보장하는 가장 높은 신뢰도 수준이다. 발행자와 수신자(브로커) 간에 4단계 핸드셰이크(PUBLISH, PUBREC, PUBREL, PUBCOMP) 과정을 거쳐 메시지가 유실이나 중복 없이 정확히 한 번만 전달되도록 보장한다. 가장 신뢰성이 높지만, 프로토콜 오버헤드가 가장 크고 메시지 전달 지연 시간이 길어 금융 거래나 원격 제어 명령과 같이 데이터 무결성이 매우 중요한 경우에 사용된다.1

#### 1.1.4  전통적인 전송 계층

MQTT v3.1.1 및 v5.0 공식 표준은 프로토콜이 "정렬되고(ordered), 손실 없으며(lossless), 양방향 연결을 제공하는" 신뢰성 있는 전송 프로토콜 위에서 동작해야 한다고 명시한다.4 이러한 요구사항을 만족시키는 가장 대표적인 프로토콜이 바로 TCP/IP이며, 따라서 전통적으로 MQTT는 TCP 연결 위에서 TLS를 통한 보안을 추가하는 방식으로 구현되어 왔다.1

### 1.2  QUIC: 현대 네트워크를 위한 전송 프로토콜 혁신

QUIC는 TCP가 설계될 당시에는 예상하지 못했던 현대 인터넷의 복잡하고 동적인 환경, 특히 모바일과 무선 네트워크의 특성에 대응하기 위해 탄생한 새로운 전송 프로토콜이다. HTTP/3의 기반 기술로 채택되면서 그 중요성이 더욱 부각되었다.

#### 1.2.1  UDP 기반의 사용자 공간 프로토콜

QUIC의 가장 근본적인 설계 특징 중 하나는 TCP가 아닌 UDP 위에 구축되었다는 점이다.9 TCP가 운영체제 커널에 깊숙이 통합되어 있어 수정 및 배포 주기가 매우 긴 반면, QUIC는 사용자 공간 라이브러리로 구현된다. 이러한 구조는 다음과 같은 근본적인 트레이드오프를 내포한다.

첫째, 사용자 공간 구현은 '개발 및 배포의 민첩성'을 극대화한다. 운영체제 커널을 변경하지 않고도 애플리케이션이나 라이브러리 업데이트만으로 프로토콜의 새로운 기능을 추가하거나 버그를 수정할 수 있다. 예를 들어, 새로운 혼잡 제어 알고리즘을 실험하고 적용하는 과정이 훨씬 빠르고 유연해진다.7 이는 빠르게 변화하는 IoT 환경의 요구사항에 신속하게 대응하는 데 큰 장점으로 작용한다.

둘째, 반대급부로 '성능 오버헤드'가 발생한다. TCP는 지난 수십 년간 커널 수준에서 고도로 최적화되었으며, 네트워크 인터페이스 카드(NIC)의 하드웨어 오프로딩 기능(예: TCP Segmentation Offload, Generic Receive Offload)의 혜택을 직접적으로 받는다.19 반면, 사용자 공간에서 동작하는 QUIC는 패킷 처리, 암호화, ACK 생성 등 대부분의 작업을 CPU에 의존해야 한다. 이로 인해 커널과 사용자 공간 사이를 오가는 컨텍스트 스위칭 비용과 순수 연산 비용이 증가하여, 특히 고대역폭 네트워크 환경에서 CPU가 병목 현상을 일으킬 수 있다.12

따라서 QUIC를 채택하는 것은 '이상적인 환경에서의 최대 처리량'이라는 특정 성능 지표를 '불안정한 환경에서의 지연 시간 및 연결 안정성'과 '프로토콜의 빠른 진화 가능성'이라는 다른 중요한 가치와 교환하는 전략적 결정으로 이해해야 한다. 이 트레이드오프는 MQTT over QUIC의 성능을 평가할 때 가장 핵심적인 고려사항이 된다.

#### 1.2.2  연결 설정 시간 단축 (0-RTT/1-RTT Handshake)

TCP 연결은 최소 1 RTT가 소요되는 3-way 핸드셰이크를 필요로 하며, 여기에 보안을 위한 TLS 핸드셰이크가 추가되면 총 2~3 RTT가 필요하다.22 지연 시간이 긴 모바일 네트워크에서 이는 상당한 초기 지연을 유발한다. QUIC는 전송 계층 핸드셰이크와 암호화 핸드셰이크(TLS 1.3 기반)를 통합하여 이 과정을 최적화했다. 클라이언트와 서버가 처음 연결할 때는 단 1 RTT만에 연결 설정과 암호화 협상이 완료된다. 한번 연결했던 서버에 다시 연결할 때는 이전에 교환했던 정보를 활용하여 핸드셰이크 과정 없이 즉시(0-RTT) 데이터를 전송할 수 있다. 이는 연결과 끊김이 빈번하게 발생하는 IoT 환경에서 지연 시간을 획기적으로 줄이는 핵심 기능이다.6

#### 1.2.3  스트림 다중화(Stream Multiplexing)와 HOL 블로킹 해결

TCP는 단일 바이트 스트림(single stream of bytes)으로 데이터를 전송한다. 이는 만약 스트림의 맨 앞에서 전송되던 패킷 하나가 유실되면, 그 패킷이 재전송되어 도착할 때까지 뒤따르는 모든 패킷의 처리가 지연되는 HOL(Head-of-Line) 블로킹 문제를 야기한다.13 QUIC는 이 문제를 해결하기 위해 단일 물리적 연결 내에 여러 개의 독립적인 논리적 스트림을 생성하고 관리하는 기능을 제공한다.12 각 스트림은 독립적인 흐름 제어와 순서 보장을 가진다. 따라서 한 스트림에서 패킷 손실로 인한 지연이 발생하더라도, 다른 스트림들은 영향을 받지 않고 데이터 전송을 계속할 수 있다. 이는 여러 종류의 데이터를(예: 센서 데이터, 제어 명령, 로그) 동시에 전송해야 하는 복합적인 IoT 애플리케이션에서 매우 큰 이점을 제공한다.23

#### 1.2.4  연결 마이그레이션(Connection Migration)

TCP 연결은 출발지 IP, 출발지 포트, 목적지 IP, 목적지 포트의 4-튜플(4-tuple)로 식별된다. 이 중 하나라도 변경되면 연결은 끊어진다. 이 때문에 모바일 디바이스가 Wi-Fi에서 셀룰러 네트워크로 전환하는 등 네트워크 인터페이스가 변경되면 TCP 연결을 처음부터 다시 맺어야 한다. QUIC는 이러한 문제를 해결하기 위해 각 연결을 고유한 64비트 연결 ID(Connection ID)로 식별한다.25 클라이언트의 IP 주소나 포트가 변경되더라도 연결 ID가 동일하게 유지되면 서버는 이를 동일한 연결로 인식하고 통신을 중단 없이 계속할 수 있다. 이 연결 마이그레이션 기능은 이동성이 핵심인 커넥티드 카, 드론, 웨어러블 디바이스 등의 IoT 애플리케이션에 필수적인 안정성을 제공한다.6

#### 1.2.5  내장된 보안(Built-in Security)

보안은 더 이상 선택이 아닌 필수 사항이다. QUIC는 최신 보안 표준인 TLS 1.3을 프로토콜의 핵심 구성요소로 통합하여, 별도의 설정 없이 모든 통신을 기본적으로 암호화한다.9 이는 TCP 위에서 TLS를 별도 계층으로 추가하는 방식보다 효율적이다. QUIC 핸드셰이크 과정에 TLS 암호화 협상이 포함되어 있어 연결 설정 지연을 줄일 뿐만 아니라, 전송되는 패킷의 헤더 일부를 제외한 대부분의 메타데이터까지 암호화하여 중간자 공격(Man-in-the-Middle attack)이나 트래픽 분석에 대한 저항성을 높인다.6

## 2.  MQTT over QUIC의 필요성과 기술적 구현

MQTT와 QUIC, 두 강력한 프로토콜의 결합은 단순한 기술적 호기심을 넘어, 현대 IoT 환경이 직면한 실질적인 문제들을 해결하기 위한 필연적인 과정이다. 이 장에서는 왜 MQTT에 QUIC가 필요한지를 동기와 기대효과 측면에서 분석하고, 실제 MQTT 메시지를 QUIC 스트림에 매핑하는 기술적 구현 방식을 살펴본다.

### 2.1  왜 MQTT에 QUIC가 필요한가?: 동기와 기대효과

전통적인 MQTT over TCP/TLS 방식은 수많은 IoT 프로젝트에서 성공적으로 사용되어 왔지만, 그 기반이 되는 TCP의 태생적 한계는 특정 시나리오에서 명백한 장애물로 작용한다. MQTT over QUIC는 바로 이 지점에서 출발한다.

#### 2.1.1  TCP의 근본적인 한계 극복

- **불안정한 네트워크에서의 잦은 연결 단절**: 커넥티드 카가 터널에 진입하거나, 농장의 센서가 일시적으로 통신 범위를 벗어나거나, 모바일 기기가 기지국을 전환하는 상황은 매우 흔하다. 이러한 네트워크 환경의 동적인 변화는 TCP 연결의 기반이 되는 IP 주소나 포트 정보를 변경시켜 연결을 강제로 끊어버린다. TCP는 이러한 상황을 예측하거나 유연하게 대처할 수 있는 메커니즘이 부재하다.6
- **느리고 비효율적인 재연결**: TCP 연결이 끊어지면, 애플리케이션은 이를 감지하고 재연결을 시도해야 한다. 이 과정은 운영체제의 TCP 소켓 자원이 완전히 해제되기를 기다린 후, 다시 3-way 핸드셰이크와 TLS 핸드셰이크를 처음부터 수행해야 하므로 상당한 시간과 자원을 소모한다. 수백만 대의 디바이스가 동시에 재연결을 시도하는 대규모 IoT 환경에서는 브로커에 막대한 부하를 유발하여 서비스 전체의 가용성을 위협할 수 있다.5
- **HOL(Head-of-Line) 블로킹으로 인한 지연**: MQTT는 다양한 종류의 메시지를 단일 TCP 연결을 통해 전송한다. 예를 들어, 긴급 경보 메시지(QoS 2)와 주기적인 상태 보고 메시지(QoS 0)가 같은 연결을 사용한다고 가정해보자. 만약 상태 보고 메시지의 패킷 하나가 유실되면, TCP는 해당 패킷이 재전송될 때까지 스트림 전체를 멈춘다. 이로 인해 뒤따르던 긴급 경보 메시지의 전달까지 불필요하게 지연되는 HOL 블로킹 현상이 발생한다. 이는 메시지의 중요도와 상관없이 모든 데이터가 영향을 받는 비효율적인 구조다.5

#### 2.1.2  MQTT over QUIC의 기대효과

QUIC의 고유한 특성들은 위에서 언급된 TCP의 한계점들을 직접적으로 해결하며, MQTT 통신에 다음과 같은 혁신적인 개선을 가져온다.

- **향상된 연결 안정성 및 이동성**: QUIC의 연결 마이그레이션 기능은 MQTT over QUIC의 가장 큰 장점 중 하나다. 클라이언트의 네트워크 주소가 바뀌어도 MQTT 연결은 중단 없이 유지된다. 이는 이동 중인 차량이나 모바일 기기에서 끊김 없는 데이터 스트리밍을 가능하게 할 뿐만 아니라, 불필요한 재연결 시도를 없애 브로커의 부하를 크게 줄인다. 개발자 입장에서도 복잡한 재연결 로직을 구현할 필요가 없어 애플리케이션 개발이 단순해진다.6
- **획기적인 지연 시간 감소**: 0-RTT/1-RTT 핸드셰이크는 연결 설정 시간을 극적으로 단축시킨다. 특히, 슬립 모드에서 주기적으로 깨어나 짧은 데이터를 전송하고 다시 슬립 모드로 돌아가는 저전력 IoT 디바이스나, 연결 단절이 잦은 환경에서는 이러한 빠른 연결 복구 능력이 전체 시스템의 응답성과 효율성을 결정짓는다.6
- **처리량 및 병렬 처리 능력 향상**: HOL 블로킹이 없기 때문에, 여러 토픽의 메시지들이 서로의 전송에 영향을 주지 않고 독립적으로 처리될 수 있다. 패킷 손실이 발생하더라도 해당 스트림에만 영향이 국한되므로, 손실이 많은 불안정한 무선 네트워크에서도 전체적인 처리량 저하가 적고 안정적인 성능을 유지할 수 있다.23
- **기본적으로 강화된 보안**: TLS 1.3이 프로토콜에 내장되어 있어, 별도의 보안 계층을 설정하고 관리해야 하는 복잡함 없이도 강력한 종단 간 암호화를 기본으로 제공한다. 이는 개발 부담을 줄이고 보안 취약점이 발생할 가능성을 낮춘다.16

### 2.2  기술적 구현 방식: MQTT 메시지의 QUIC 스트림 매핑

MQTT over QUIC를 구현하는 것은 단순히 전송 계층을 TCP에서 UDP로 바꾸는 것 이상의 과정이다. MQTT와 QUIC는 모두 사용자 공간에서 동작하는 프로토콜이므로, 두 프로토콜을 효과적으로 연동시키기 위한 아키텍처 설계가 필요하다.

초기 연구 단계에서는 MQTT와 QUIC 사이에 별도의 통신 계층(Inter-Process Communication, IPC)을 두거나, MQTT의 내부 데이터 구조를 QUIC와 연동되도록 재설계하는 복잡한 과정이 필요했다.22 하지만 기술이 성숙하면서, 상용 MQTT 브로커인 EMQX는 MQTT 패킷을 QUIC 스트림에 매핑하는 구체적인 두 가지 운영 모드를 제시하며 기술의 실용성을 입증했다. 이 두 가지 모드는 MQTT over QUIC의 잠재력을 어떻게 활용할 수 있는지 보여주는 중요한 사례이다.23

#### 2.2.1  단일 스트림 모드 (Single Stream Mode)

이 모드는 MQTT over QUIC를 구현하는 가장 기본적이고 직관적인 방식이다. 하나의 MQTT 연결은 하나의 양방향 QUIC 스트림에 일대일로 매핑된다. 모든 MQTT 제어 패킷(CONNECT, PUBLISH, SUBSCRIBE 등)은 이 단일 스트림을 통해 순서대로 전송된다.23

이 방식은 기존의 MQTT over TCP 동작 방식과 매우 유사하다. TCP가 제공하는 단일의 순서 보장 바이트 스트림을 QUIC의 단일 스트림이 대체하는 형태이기 때문이다. 따라서 기존 TCP 기반으로 설계된 애플리케이션 로직을 거의 변경하지 않고도 MQTT over QUIC로 전환할 수 있다는 장점이 있다. 동시에, 전송 계층은 QUIC로 대체되었으므로 빠른 핸드셰이크, 연결 마이그레이션, 향상된 혼잡 제어 등 QUIC의 핵심적인 이점들을 그대로 누릴 수 있다. 즉, 구현의 단순성과 기존 시스템과의 호환성을 유지하면서 QUIC의 성능 향상을 얻을 수 있는 가장 실용적인 접근법이다.

#### 2.2.2  다중 스트림 모드 (Multi-Stream Mode)

다중 스트림 모드는 QUIC의 핵심 기능인 스트림 다중화를 적극적으로 활용하여 MQTT 통신을 한 단계 발전시키는 고급 방식이다. 이 모드에서는 단일 MQTT 연결 내에서 목적에 따라 여러 개의 QUIC 스트림을 동시에 사용한다.23 예를 들어 다음과 같은 매핑 전략이 가능하다.

- **제어와 데이터의 분리**: MQTT 연결의 생명주기를 관리하는 제어 패킷(CONNECT, PINGREQ/PINGRESP 등)은 하나의 전용 스트림으로, 실제 데이터를 담고 있는 PUBLISH 패킷은 다른 스트림들로 분리하여 전송한다.
- **토픽별 스트림 할당**: 클라이언트가 구독하는 각 토픽 또는 토픽 그룹마다 별도의 스트림을 할당한다.
- **QoS 레벨별 스트림 할당**: QoS 2와 같이 신뢰성이 중요한 메시지는 하나의 스트림으로, QoS 0와 같이 속도가 중요한 메시지는 다른 스트림으로 분리한다.

이러한 다중 스트림 방식은 단순한 전송 계층의 교체를 넘어, MQTT 프로토콜 자체의 동작 방식을 재정의할 수 있는 강력한 잠재력을 지닌다. 전통적인 MQTT over TCP는 모든 메시지가 하나의 파이프라인(단일 바이트 스트림)을 통해 선입선출(FIFO) 방식으로 처리되므로, QoS 외에는 메시지의 우선순위를 구분할 방법이 없다.1 하지만 QUIC의 다중 스트림은 여러 개의 독립적인 파이프라인을 제공하는 것과 같다.12

이 독립적인 파이프라인들에 MQTT의 논리적 개념(토픽, QoS 레벨 등)을 매핑함으로써, 전송 계층이 애플리케이션 계층의 컨텍스트(메시지의 중요도)를 인지하고 차등적인 서비스를 제공하는 것이 가능해진다. 예를 들어, `/alerts` 토픽에 할당된 스트림은 네트워크 혼잡 시에도 `/telemetry/low_priority` 토픽에 할당된 스트림보다 높은 우선순위를 갖도록 설정할 수 있다.23 이는 기존 MQTT에서는 불가능했던 '애플리케이션 인지형(Application-Aware) 전송'을 구현하는 새로운 패러다임이다. 따라서 다중 스트림 모드는 MQTT over QUIC를 단순한 'TCP 대체재'가 아닌, 차세대 IoT 통신을 위한 근본적인 아키텍처 혁신으로 격상시킨다. 이는 향후 MQTT 표준이 진화할 때, 이러한 스트림 매핑 전략을 공식적으로 정의하는 방향으로 발전할 수 있음을 강력히 시사한다.

## 3.  성능 비교 분석: MQTT over QUIC vs. TCP/TLS

MQTT over QUIC의 이론적 장점이 실제 환경에서도 유효한지 검증하기 위해서는 기존의 TCP/TLS 방식과의 객관적인 성능 비교가 필수적이다. 다양한 연구와 벤치마크 테스트는 특정 시나리오와 평가 지표에 따라 두 기술의 우위가 달라질 수 있음을 보여준다. 따라서 단편적인 비교가 아닌, 다차원적인 관점에서의 심층 분석이 요구된다.

### 3.1  연결 수립 및 재연결 속도

연결 설정 시간은 특히 통신 세션이 짧고 빈번하게 발생하는 IoT 환경에서 전체 지연 시간에 큰 영향을 미친다.

- **초기 연결 (1-RTT Handshake)**: QUIC는 전송 계층과 암호화 계층의 핸드셰이크를 통합하여 1 RTT만에 연결을 수립한다. 반면, TCP/TLS는 각각의 핸드셰이크를 순차적으로 진행하므로 최소 2~3 RTT가 소요된다.22 이러한 차이는 네트워크의 물리적 왕복 시간(RTT)이 길수록 더욱 두드러진다. 예를 들어, RTT가 1ms인 저지연 환경에서는 두 프로토콜 간의 차이가 미미하지만, RTT가 30ms 이상인 위성 또는 셀룰러 네트워크 환경에서는 QUIC의 연결 설정 속도가 월등히 빠르다는 것이 실험적으로 입증되었다.5
- **재연결 (0-RTT Handshake)**: QUIC의 0-RTT 기능은 재연결 성능에서 압도적인 우위를 보인다. 한번 연결했던 서버에 다시 접속할 때, 클라이언트는 이전에 공유된 암호화 정보를 사용하여 핸드셰이크 과정 없이 첫 번째 패킷에 MQTT 데이터를 실어 보낼 수 있다. 이로 인해 재연결이 거의 즉각적으로 이루어진다.10 반면, TCP/TLS는 연결이 끊어질 때마다 매번 전체 핸드셰이크 과정을 반복해야 하므로 상당한 지연이 발생한다.5 이러한 특성은 네트워크 불안정으로 인해 연결/끊김이 빈번한 모바일 IoT 환경에서 사용자 경험과 시스템 효율성을 극적으로 향상시키는 핵심 요소이다. 여러 연구 결과에 따르면, QUIC는 연결 설정에 필요한 패킷 교환 수를 TCP 방식 대비 최대 56%까지 줄일 수 있다.22

### 3.2  처리량 및 지연 시간

메시지 처리량(Throughput)과 전달 지연 시간(Latency)은 통신 프로토콜의 핵심 성능 지표이지만, 이들의 평가는 네트워크 환경에 따라 크게 달라진다.

- **불안정한 네트워크 (고손실/고지연 환경)**: 패킷 손실률이 높고 지연이 심한 무선 네트워크 환경은 QUIC의 진가가 발휘되는 주 무대이다. TCP는 패킷 손실을 네트워크 혼잡의 신호로 간주하여 전송 속도를 급격히 줄이는 보수적인 혼잡 제어 메커니즘을 사용한다. 또한, HOL 블로킹 문제로 인해 단일 패킷 손실이 전체 스트림을 지연시킨다. 반면, QUIC는 더 정교한 손실 감지 및 복구 메커니즘을 사용하며, 스트림 다중화 덕분에 한 스트림의 문제가 다른 스트림으로 전파되지 않는다. 그 결과, 패킷 손실률이 5%에 달하는 열악한 환경에서도 QUIC는 TCP보다 훨씬 낮은 지연 시간과 지연 변동성(Jitter)을 유지하며 안정적인 성능을 보인다.8 한 실증 연구에서는 5% 패킷 손실률의 Wi-Fi 환경에서 MQTT over QUIC가 TCP 방식에 비해 평균 지연 시간을 25.5%, 지연 변동성을 61% 감소시켰다고 보고했다.8
- **안정적인 고속 네트워크 환경**: 역설적으로, 네트워크 조건이 매우 이상적일 경우(예: 데이터센터 내부의 고대역폭, 저손실 유선 네트워크), QUIC의 성능이 TCP/TLS보다 저하될 수 있다. 이는 QUIC의 근본적인 설계 특성에서 기인한다. QUIC의 모든 패킷 처리, 암호화/복호화, ACK 생성 등은 사용자 공간에서 CPU에 의해 수행된다. 이는 커널 수준에서 고도로 최적화되고 하드웨어 오프로딩의 지원을 받는 TCP 스택에 비해 더 많은 CPU 자원을 소모한다. 따라서 네트워크 대역폭이 매우 높아(예: 500Mbps~1Gbps) CPU가 병목 지점이 되는 시나리오에서는 TCP의 처리량이 QUIC를 능가할 수 있다.20 한 연구는 1Gbps 네트워크에서 QUIC(HTTP/3) 기반 파일 전송이 TCP(HTTP/2) 기반보다 최대 45.2% 느렸다고 보고하기도 했다.20

### 3.3  연결 마이그레이션 (NAT Rebinding) 성능

연결 마이그레이션은 이동성을 가진 IoT 디바이스에 있어 가장 중요한 기능 중 하나이다. 클라이언트의 IP 주소가 변경되는 NAT Rebinding 상황을 시뮬레이션한 여러 실증 테스트는 QUIC의 우수성을 명백하게 보여준다. 실험에서 MQTT over TCP 연결은 클라이언트의 IP 주소가 변경되자마자 즉시 끊어졌고, 메시지 전송은 완전히 중단되었다. 애플리케이션은 연결 단절을 감지하고 재연결 절차를 처음부터 다시 시작해야 했다. 반면, MQTT over QUIC 연결은 IP 주소 변경에도 불구하고 아무런 중단 없이 지속적으로 메시지를 주고받았다.25 이는 QUIC의 연결 ID 기반 마이그레이션 기능이 이론에만 그치지 않고 실제 환경에서도 완벽하게 동작함을 증명한다. 커넥티드 카, 드론, 물류 트래킹 디바이스 등 이동이 잦은 애플리케이션에서 이는 단순한 성능 향상을 넘어 서비스의 연속성을 보장하는 '킬러 기능(Killer Feature)'이라 할 수 있다.

### 3.4  자원 사용량 분석: CPU 및 메모리

MQTT over QUIC의 자원 사용량은 평가하는 관점에 따라 이중적인 특성을 보인다.

- **연결 관리 측면에서의 효율성**: 대규모 IoT 환경에서는 수많은 클라이언트가 동시에 브로커에 접속하고, 네트워크 문제로 인해 빈번하게 재접속을 시도한다. 이러한 시나리오에서 QUIC는 TCP/TLS보다 서버 자원을 더 효율적으로 사용한다. TCP는 연결 과정에서 비정상적으로 종료될 경우 'half-open connection' 상태에 빠져 서버의 메모리와 CPU 자원을 불필요하게 점유하는 문제가 있다. QUIC는 UDP 기반이므로 이러한 상태 관리 부담이 없으며, 연결 수립 과정이 더 가볍다. 연구에 따르면, QUIC는 half-open 연결 문제를 제거함으로써 서버의 프로세서 사용량을 최대 83%, 메모리 사용량을 50%까지 절감할 수 있는 것으로 나타났다.7
- **데이터 처리 측면에서의 오버헤드**: 반면, 순수한 데이터 전송 과정에서의 단위 패킷당 CPU 소모량은 QUIC가 TCP보다 높을 수 있다. 앞서 언급했듯이, QUIC의 사용자 공간 처리 방식과 모든 패킷에 대한 암호화/복호화 작업은 커널에서 최적화된 TCP에 비해 더 많은 연산을 필요로 한다.12 따라서 높은 처리량이 요구되는 시나리오에서는 QUIC의 CPU 사용량이 성능의 제약 조건이 될 수 있다.

이러한 복합적인 성능 특성은 "어느 프로토콜이 더 우수한가?"라는 단편적인 질문이 무의미함을 보여준다. 성능 평가는 반드시 구체적인 '시나리오'를 기반으로 한 '다차원적 분석'을 통해 이루어져야 한다. 즉, '처리량', '지연 시간', '연결 성공률', 'CPU 사용량' 등의 다양한 성능 지표를 '네트워크 조건(안정/불안정)', '연결 패턴(장기/단기)', '디바이스 규모'와 같은 시나리오 축과 조합하여 입체적으로 평가해야만 올바른 기술적 의사결정을 내릴 수 있다. 아래의 표는 이러한 다차원적 비교 분석 결과를 요약한 것이다.

| 성능 지표 (Metric)                     | TCP/TLS                           | MQTT over QUIC               | 주요 근거 (Supporting Snippets) |
| -------------------------------------- | --------------------------------- | ---------------------------- | ------------------------------- |
| **이상적 네트워크 (고대역폭, 저손실)** |                                   |                              |                                 |
| ▷ 최대 처리량                          | **우수**                          | 상대적 약세 (CPU 병목)       | 20                              |
| ▷ CPU 사용량 (데이터 처리)             | **낮음** (커널/하드웨어 오프로딩) | 높음 (사용자 공간 처리)      | 12                              |
| **불안정 네트워크 (고지연, 고손실)**   |                                   |                              |                                 |
| ▷ 연결 설정 시간                       | 느림 (다중 RTT)                   | **빠름** (1-RTT)             | 5                               |
| ▷ 지연 시간 및 지터                    | 높음 (HOL 블로킹, 재전송)         | **낮음** (스트림 다중화)     | 21                              |
| ▷ 처리량                               | 급격히 저하                       | **안정적 유지**              | 21                              |
| **모바일 환경 (네트워크 핸드오버)**    |                                   |                              |                                 |
| ▷ 연결 유지                            | 연결 단절                         | **연결 유지** (마이그레이션) | 25                              |
| ▷ 재연결 시간                          | 느림 (전체 핸드셰이크 필요)       | **거의 즉시** (0-RTT)        | 10                              |
| **대규모 연결 관리**                   |                                   |                              |                                 |
| ▷ 서버 자원 (연결/해제 시)             | 높음 (Half-open 연결)             | **낮음**                     | 24                              |

## 4.  주요 응용 분야 및 사례 연구

MQTT over QUIC의 기술적 장점들은 이론에만 머무르지 않고, 실제 IoT 응용 분야에서 발생하는 구체적인 문제들을 해결하는 데 직접적으로 기여한다. 특히 이동성과 네트워크 불안정성이 상존하는 커넥티드 카(IoV)와 저지연, 고신뢰성이 필수적인 산업용 IoT(IIoT)는 이 기술의 가치가 가장 크게 발휘될 수 있는 대표적인 분야이다.

### 4.1  커넥티드 카(IoV): 이동성과 불안정성을 극복하는 아키텍처

커넥티드 카는 단순한 이동 수단을 넘어, 주변 환경 및 클라우드와 끊임없이 데이터를 주고받는 '바퀴 달린 데이터 플랫폼'으로 진화하고 있다. 이러한 진화는 통신 기술에 새로운 차원의 요구사항을 제기한다.

#### 4.1.1  IoV의 통신 요구사항과 도전 과제

차량은 텔레매틱스 데이터(차량 상태, 주행 습관), V2X(Vehicle-to-Everything) 통신을 통한 안전 정보, 대용량 지도 데이터 및 펌웨어 OTA(Over-the-Air) 업데이트, 실시간 교통 정보를 반영하는 인포테인먼트 서비스 등 방대하고 다양한 종류의 데이터를 처리해야 한다. 이 데이터들은 대부분 실시간성과 높은 신뢰성을 요구한다. 하지만 차량의 운행 환경은 통신에 매우 비우호적이다. 도심의 빌딩 숲, 지하 터널, 산악 지역 등은 전파 음영 지역을 만들고, 고속 이동 중에는 여러 셀룰러 기지국 간의 핸드오버가 빈번하게 발생한다. 이러한 환경에서 TCP 기반 통신은 잦은 연결 단절과 긴 재연결 시간으로 인해 서비스의 연속성을 보장하기 어렵다.6

#### 4.1.2  MQTT over QUIC의 적용 아키텍처 및 핵심 이점

MQTT over QUIC는 IoV의 이러한 도전 과제를 해결하는 효과적인 솔루션을 제공한다. 일반적인 적용 아키텍처는 차량 내 텔레매틱스 제어 장치(TCU)나 중앙 게이트웨이에 MQTT over QUIC 클라이언트를 탑재하는 형태이다. 이 클라이언트는 클라우드에 배포된 MQTT 브로커(예: EMQX)와 QUIC 프로토콜을 통해 지속적인 연결을 맺고, 차량 내 다양한 ECU(Electronic Control Unit)로부터 수집된 데이터를 발행하거나 클라우드로부터의 명령을 수신하여 전달한다. 이 아키텍처는 다음과 같은 핵심 이점을 제공한다.

1. **끊김 없는 연결성 (Seamless Connectivity)**: 차량이 4G, 5G, Wi-Fi 등 서로 다른 네트워크 사이를 오가거나, 통신사의 NAT(Network Address Translation) 정책으로 인해 IP 주소가 변경되더라도, QUIC의 연결 마이그레이션 기능 덕분에 MQTT 세션은 중단되지 않는다. 이는 TCP 기반 솔루션이 겪는 가장 큰 문제인 빈번한 재연결과 그로 인한 데이터 유실 및 서비스 지연을 근본적으로 해결한다.25
2. **빠른 응답성 및 실시간성 보장**: 불안정한 네트워크 환경에서도 QUIC는 낮은 지연 시간으로 데이터를 전송할 수 있다. 이는 전방 차량의 급제동 신호나 교차로 충돌 경고와 같이 수십 밀리초(ms) 단위의 응답성이 요구되는 실시간 V2X 안전 애플리케이션에 매우 중요하다. 또한, 0-RTT 재연결 기능은 차량이 터널을 빠져나온 직후와 같이 통신이 복구되는 즉시 클라우드와의 연결을 재개하여 서비스 중단 시간을 최소화한다.23

#### 4.1.3  적용 사례

SAIC Volkswagen과 같은 선도적인 자동차 제조사들은 이미 IoV 플랫폼의 통신 백본을 MQTT로 전환하고 있다. 초기에는 자체 개발한 사설 TCP 프로토콜이나 HTTP 기반 시스템을 사용했지만, 차량 모델과 서비스가 다양해지면서 확장성, 유지보수 비용, 개발 속도 측면에서 한계에 부딪혔다. MQTT를 도입함으로써 표준화된 프로토콜을 통해 개발 비용을 절감하고 시스템 확장성을 확보할 수 있었다. 더 나아가, 이들 기업은 EMQX와 같은 차세대 브로커를 통해 MQTT over QUIC의 도입을 적극적으로 검토하고 있다. 이는 미래의 자율주행 및 고도화된 커넥티드 서비스를 위한 통신 인프라를 선제적으로 구축하려는 전략의 일환이다.35

### 4.2  산업용 IoT(IIoT): 저지연 및 고신뢰성 통신 요구사항 충족

스마트 팩토리로 대표되는 IIoT 환경은 생산성과 효율성을 극대화하기 위해 수많은 센서, 액추에이터, 로봇, PLC(Programmable Logic Controller) 등이 유기적으로 연결되어 동작한다. 이러한 환경은 IoV와는 또 다른 종류의 통신 요구사항을 가진다.

#### 4.2.1  IIoT의 통신 요구사항과 도전 과제

IIoT 환경은 수많은 기기가 밀집되어 있고, 모터나 용접기 등에서 발생하는 강력한 전자기파로 인해 무선 통신(Wi-Fi, 5G 등) 환경이 매우 열악하다. 전파 간섭으로 인한 패킷 손실이 빈번하게 발생할 수 있다. 그럼에도 불구하고, 정밀한 공정 제어와 실시간 모니터링을 위해서는 마이크로초(μs)에서 밀리초(ms) 단위의 매우 낮은 지연 시간과 99.999% 이상의 높은 신뢰성이 요구된다.3

#### 4.2.2  MQTT over QUIC의 적용 및 핵심 이점

- **HOL 블로킹 해결의 중요성**: 스마트 팩토리에서는 수백, 수천 개의 센서로부터 온도, 압력, 진동 등 다양한 데이터가 각기 다른 토픽으로 동시에 브로커에 발행된다. 만약 TCP를 사용한다면, 전파 간섭으로 인해 하나의 저품질 무선 링크에서 발생한 패킷 손실이 다른 모든 정상적인 링크의 데이터 스트림 처리까지 지연시키는 심각한 문제를 야기할 수 있다. 이는 중요한 로봇 제어 신호가 사소한 온도 센서 데이터의 전송 실패 때문에 지연되는 결과를 초래할 수 있다. QUIC의 스트림 다중화 기능은 이러한 HOL 블로킹 문제를 원천적으로 방지하여, 각 데이터 스트림의 독립성을 보장하고 시스템 전체의 안정성과 예측 가능성을 높인다.8
- **에너지 효율성**: IIoT 환경에서는 전원 공급이 어려운 위치에 배터리로 동작하는 무선 센서가 다수 사용된다. 통신은 배터리 소모의 주요 원인 중 하나이다. 한 실증 연구에 따르면, MQTT over QUIC는 TCP 방식에 비해 지연 시간을 크게 단축하면서도 에너지 소비량은 비슷한 수준으로 유지하는 것으로 나타났다.8 이는 서비스 품질을 향상시키면서도 디바이스의 운영 수명을 저해하지 않는다는 점에서 중요한 의미를 가진다.

이러한 특성들은 MQTT over QUIC가 IIoT 분야의 핵심 과제인 OT(운영 기술)와 IT(정보기술)의 융합을 가속화하는 핵심 인프라 기술이 될 수 있음을 시사한다. IIoT의 궁극적인 목표 중 하나는 공장 현장의 OT 시스템(PLC, 센서, 제어기 등)에서 생성되는 실시간 데이터를 클라우드의 IT 시스템(ERP, MES, 데이터 분석 플랫폼 등)과 원활하게 통합하여 데이터 기반의 의사결정을 내리는 것이다.37

이 통합 과정에서 MQTT는 서로 다른 시스템과 프로토콜을 연결하는 '통합 네임스페이스(Unified Namespace)'를 구축하는 데 이상적인 프로토콜로 주목받고 있다.37 MQTT over QUIC는 바로 이 MQTT 기반 네임스페이스를 실어 나르는 '전송 혈관' 역할을 수행한다. 공장 내부의 불안정한 무선 네트워크 구간과 외부 인터넷망으로 이어지는 구간 모두에서 발생할 수 있는 네트워크 문제(전파 간섭, 지연, 패킷 손실 등)를 효과적으로 흡수하고 완화하는 것이다. 즉, MQTT over QUIC는 단순히 디바이스와 브로커를 연결하는 것을 넘어, OT와 IT를 잇는 전체 데이터 파이프라인의 종단 간(end-to-end) 신뢰성과 성능을 보장하는 강력한 기반을 제공한다. 이는 IIoT 시스템 전체의 안정성과 효율성을 한 단계 끌어올리는 중요한 역할을 할 것이다.

## 5.  생태계 현황과 미래 전망

아무리 뛰어난 기술이라도 실제 현장에 적용하기 위해서는 성숙한 생태계의 지원이 필수적이다. MQTT over QUIC는 아직 초기 단계에 있지만, 빠르게 발전하며 미래를 준비하고 있다. 이 장에서는 브로커 및 클라이언트 라이브러리 지원 현황, 표준화 동향, 그리고 앞으로 해결해야 할 과제들을 종합적으로 살펴본다.

### 5.1  브로커 및 클라이언트 라이브러리 지원 현황

기술의 확산은 개발자들이 쉽게 사용할 수 있는 도구, 즉 브로커와 클라이언트 라이브러리의 가용성에 크게 의존한다.

- **브로커 (Broker)**: 현재 MQTT over QUIC를 상용 수준에서 공식적으로 지원하는 가장 대표적인 브로커는 **EMQX**이다. EMQ Technologies는 EMQX 5.0 버전부터 이 기능을 선도적으로 도입했으며, 관련 기술 개발과 표준화 노력을 주도하고 있다.5 다른 주요 오픈소스 및 상용 MQTT 브로커(예: HiveMQ, Mosquitto, RabbitMQ)에서는 아직 공식적인 지원 계획이 발표되지 않은 상태로, EMQX가 기술 생태계를 이끌고 있는 형국이다.

- **클라이언트 라이브러리 (Client Libraries)**: 클라이언트 측 생태계는 아직 성숙하지 않은 단계이며, 이것이 현재 기술 확산의 가장 큰 장벽 중 하나로 작용하고 있다.10

  - **EMQ 제공 라이브러리**: EMQ는 자사의 브로커와의 완벽한 호환성을 보장하는 클라이언트 SDK를 직접 개발하여 제공하고 있다. C언어로 작성된 핵심 라이브러리인 **NanoSDK**를 기반으로, 이를 Python과 Java에서 사용할 수 있도록 바인딩(binding)을 제공한다. 이 SDK들은 비동기 I/O, 0-RTT 핸드셰이크 등 MQTT over QUIC의 핵심 기능들을 지원한다.10

  - **오픈소스 커뮤니티 동향**: 가장 널리 사용되는 오픈소스 MQTT 클라이언트 라이브러리인 **Eclipse Paho** 프로젝트에서는 현재 QUIC를 공식적으로 지원할 계획이 없다고 밝힌 바 있다.38 이는 많은 개발자들이 QUIC를 도입하는 데 큰 허들로 작용한다. 다만, 커뮤니티 내에서는 

    `paho.mqtt.c` 라이브러리에 Microsoft의 QUIC 구현체인 `msquic`을 통합한 개인적인 포크(fork)가 등장하는 등 기술에 대한 관심이 점차 나타나고 있다.38 또한, Rust 언어 기반으로 

    `quinn` 라이브러리를 활용한 `mquictt`라는 실험적인 클라이언트 라이브러리도 pre-alpha 단계로 개발이 진행 중이다.39

- **현실적인 마이그레이션 경로: 엣지 브릿징**: 기존에 운영 중인 수많은 IoT 디바이스의 펌웨어를 수정하여 클라이언트 라이브러리를 교체하는 것은 현실적으로 매우 어렵다. 이러한 문제를 해결하기 위해 EMQ는 엣지 컴퓨팅용 경량 브로커인 **NanoMQ**를 통해 브릿징 솔루션을 제공한다. NanoMQ는 엣지 단에서 기존의 표준 MQTT(TCP 기반) 트래픽을 수신한 뒤, 이를 MQTT over QUIC 프로토콜로 변환하여 클라우드의 EMQX 브로커로 안전하고 효율적으로 전송하는 역할을 할 수 있다. 이는 디바이스 단의 코드 수정 없이도 QUIC의 장점(특히 불안정한 WAN 구간에서의 안정성)을 활용할 수 있는 매우 현실적이고 실용적인 마이그레이션 경로를 제시한다.10

아래 표는 현재 MQTT over QUIC를 지원하는 주요 브로커 및 라이브러리의 현황을 요약한 것이다. 이는 개발자와 시스템 아키텍트가 기술 도입을 검토할 때, 사용 가능한 도구의 성숙도와 제약 사항을 한눈에 파악하는 데 도움을 줄 것이다.

| 구분               | 이름    | 제공자             | 지원 언어       | 성숙도               | 주요 특징 및 제약                                       | 근거 |
| ------------------ | ------- | ------------------ | --------------- | -------------------- | ------------------------------------------------------- | ---- |
| **브로커**         | EMQX    | EMQ                | Erlang          | **Production Ready** | 단일/다중 스트림 모드 지원. 현재 세션 상태 보존 미지원. | 5    |
| **엣지 브릿지**    | NanoMQ  | EMQ                | C               | **Production Ready** | MQTT ↔ MQTT over QUIC 브릿징. TCP로 자동 폴백 지원.     | 10   |
| **클라이언트 SDK** | NanoSDK | EMQ                | C, Python, Java | **Production Ready** | EMQX와 완벽 호환. 비동기 I/O, 0-RTT 지원.               | 10   |
|                    | Paho    | Eclipse Foundation | C, C++, Java 등 | **미지원**           | 공식 지원 계획 없음. 커뮤니티 포크 존재.                | 38   |
|                    | mquictt | Community          | Rust            | Pre-alpha            | quinn-rs 기반. QoS2 기본 지원.                          | 39   |

### 5.2  표준화 동향: OASIS와 IETF

기술이 널리 채택되고 상호운용성을 확보하기 위해서는 표준화가 필수적이다.

- **OASIS (Organization for the Advancement of Structured Information Standards)**: MQTT 프로토콜의 공식 표준을 관리하는 기구는 OASIS이다.11 MQTT over QUIC 역시 공식 표준으로 인정받기 위한 논의가 OASIS MQTT 기술 위원회(Technical Committee) 내에서 활발하게 진행되고 있다. 특히 이 기술을 선도하고 있는 EMQ가 표준화 노력을 적극적으로 주도하고 있으며, 기술 사양을 담은 초안(draft) 문서가 위원회 내에서 공유 및 논의되고 있다.5
- **IETF (Internet Engineering Task Force)**: QUIC 프로토콜 자체는 IETF에서 개발하고 RFC 9000으로 표준화한 기술이다.10 IETF 내에 'Media over QUIC (moq)'이라는 워킹 그룹이 존재하지만, 이는 이름에서 알 수 있듯이 라이브 스트리밍, 게이밍 등 미디어 전송에 최적화된 프로토콜을 개발하는 것을 목표로 하므로, 범용적인 IoT 메시징을 다루는 MQTT over QUIC와는 직접적인 관련이 적다.41

MQTT over QUIC가 OASIS 공식 표준으로 채택된다면, 이는 기술 생태계에 매우 긍정적인 신호가 될 것이다. 표준화는 기술 도입의 불확실성을 제거하고, 다양한 벤더들이 상호운용 가능한 브로커와 클라이언트 라이브러리를 개발하도록 촉진하여 생태계를 급격히 성장시키는 기폭제가 될 것으로 예상된다.37

### 5.3  당면 과제와 향후 연구 방향

MQTT over QUIC는 유망한 기술이지만, 널리 확산되기까지는 몇 가지 기술적, 생태계적 과제를 해결해야 한다.

- **기술적 과제**:
  - **UDP 트래픽 제한 및 폴백(Fallback) 메커니즘**: 일부 기업의 내부 방화벽이나 이동통신사 네트워크 정책은 보안이나 관리상의 이유로 UDP 트래픽을 차단하거나 속도를 제한하는 경우가 있다. QUIC는 UDP 기반이므로 이러한 네트워크 환경에서는 연결 자체가 불가능할 수 있다. 따라서 QUIC 연결 시도가 실패했을 때, 기존의 TCP/TLS 방식으로 원활하게 자동 전환(fallback)하여 서비스 연속성을 보장하는 클라이언트 측 메커니즘이 필수적이다.10
  - **세션 상태 보존 문제**: 현재 EMQX의 구현에서는 QUIC 연결이 일시적으로 끊어졌다가 재개될 때 MQTT 세션 상태(클라이언트의 구독 정보, 미확인 QoS 1/2 메시지 등)가 보존되지 않는다. 이는 클라이언트가 재연결 시 모든 토픽을 다시 구독해야 함을 의미하며, 진정한 의미의 '끊김 없는(seamless)' 경험을 제공하기 위해서는 브로커와 클라이언트 양단에서 세션 상태를 복원하는 기능이 반드시 개선되어야 한다.5
  - **CPU 오버헤드 최적화**: 안정적인 고속 네트워크 환경에서 나타나는 CPU 사용량 문제를 완화하기 위한 연구가 필요하다. 사용자 공간 QUIC 스택의 내부 로직을 최적화하고, 최신 네트워크 카드에서 지원하기 시작한 UDP 하드웨어 오프로딩 기술(예: UDP Generic Receive Offload, GRO)을 적극적으로 활용하여 커널-사용자 공간 전환 비용을 줄이는 방안이 모색되어야 한다.20
- **생태계 과제**:
  - **클라이언트 라이브러리 지원 확대**: 기술의 성패는 개발자들의 손에 달려있다. Paho와 같이 널리 사용되는 주류 클라이언트 라이브러리에서 공식적으로 지원하지 않는 한, 개발자들이 프로덕션 환경에 MQTT over QUIC를 채택하기는 매우 어렵다. 커뮤니티의 참여를 독려하고 주요 라이브러리에 기여하여 생태계를 확장하는 노력이 시급하다.23
- **향후 연구 방향**:
  - **고급 스트림 매핑 전략**: 다중 스트림 모드의 잠재력을 극대화하기 위한 정교한 스트림 관리 및 매핑 전략에 대한 연구가 필요하다. 애플리케이션의 요구사항(QoS, 토픽 중요도 등)과 실시간 네트워크 상태를 기반으로 스트림을 동적으로 할당하고, 스트림별 우선순위를 제어하여 차별화된 서비스를 제공하는 알고리즘 개발이 유망한 연구 분야이다.
  - **보안 모델 심층 분석**: QUIC는 기본적으로 강력한 보안을 제공하지만, 새로운 기술에는 새로운 공격 벡터가 존재할 수 있다. 특히 0-RTT 핸드셰이크 과정에서 발생할 수 있는 재전송 공격(replay attack)에 대한 취약점과, 이를 방어하기 위한 애플리케이션 레벨에서의 안전한 데이터 처리 방안에 대한 심층적인 분석 및 검증이 요구된다.7
  - **에너지 소비 정밀 모델링**: 다양한 종류의 저전력 IoT 디바이스와 네트워크 시나리오에서 MQTT over QUIC의 에너지 소비량을 정밀하게 측정하고 모델링하는 연구가 필요하다. 이를 통해 특정 애플리케이션에서 이 기술을 도입했을 때 예상되는 배터리 수명을 예측하고, 에너지 효율을 최적화하는 방안을 제시할 수 있을 것이다.8

## 6. 결론

MQTT over QUIC는 IoT 통신 기술의 자연스럽고 필연적인 진화 방향을 제시한다. 전통적인 TCP/TLS 기반의 MQTT는 지난 십수 년간 IoT 생태계의 성장을 이끌어왔지만, 그 근간이 되는 TCP는 오늘날의 동적이고 불안정한 네트워크 환경, 특히 모바일 환경에서 명백한 한계를 드러내고 있다. MQTT over QUIC는 바로 이 지점에서 출발하여, TCP의 고질적인 문제들을 해결하기 위한 강력하고 현대적인 솔루션을 제공한다.

본 보고서에서 심층적으로 분석한 바와 같이, QUIC의 핵심 기능들—빠른 연결 설정을 위한 0-RTT/1-RTT 핸드셰이크, HOL 블로킹을 원천적으로 해결하는 스트림 다중화, 그리고 완벽한 이동성을 보장하는 연결 마이그레이션—은 기존 MQTT 통신의 패러다임을 바꿀 잠재력을 지니고 있다. 특히, 네트워크 연결이 생명선과도 같은 커넥티드 카(IoV)와, 저지연 및 고신뢰성이 생산성과 직결되는 산업용 IoT(IIoT) 분야에서 MQTT over QUIC는 기존 기술로는 달성하기 어려웠던 수준의 서비스 연속성과 안정성을 제공하며 혁신적인 가치를 창출할 수 있다.

그러나 이 기술이 모든 문제를 해결하는 만병통치약은 아니다. 그 이면에는 반드시 고려해야 할 트레이드오프와 현실적인 과제들이 존재한다. 이상적인 고속 네트워크 환경에서는 사용자 공간 처리와 암호화로 인한 CPU 오버헤드가 오히려 성능 저하를 유발할 수 있다. 또한, 아직 초기 단계에 머물러 있는 클라이언트 생태계의 미성숙함과 일부 중요한 기능(예: 세션 상태 보존)의 부재는 기술 도입을 망설이게 하는 현실적인 장벽이다.

따라서 MQTT over QUIC의 도입 결정은 애플리케이션의 핵심 요구사항과 운영 환경에 대한 면밀한 분석을 바탕으로 신중하게 이루어져야 한다. 만약 시스템이 해결해야 할 가장 큰 문제가 **'이동성'과 '네트워크 불안정성'**이라면, MQTT over QUIC는 현재의 일부 미성숙함을 감수하고서라도 도입을 적극적으로 고려할 만한 매우 매력적인 솔루션이다. 반면, 안정적인 데이터센터 환경에서 **'최대 처리량'**이 가장 중요한 성능 지표라면, 고도로 최적화된 기존의 TCP/TLS 스택이 여전히 더 효율적이고 유효한 선택일 수 있다.

궁극적으로, 기술 생태계의 성숙과 OASIS를 중심으로 한 표준화가 성공적으로 진행됨에 따라, MQTT over QUIC는 점차 더 넓은 IoT 영역에서 기본 전송 방식으로 자리 잡을 것이 자명하다. 이는 단순히 하나의 프로토콜이 다른 프로토콜로 대체되는 것을 넘어, 사물 인터넷이 한 단계 더 높은 수준의 신뢰성과 실시간성을 확보하는 새로운 시대를 여는 중요한 기술적 전환점이 될 것으로 강력히 전망한다.

#### **참고 자료**

1. Introduction to MQTT - MATLAB & Simulink - MathWorks, 8월 4, 2025에 액세스, https://la.mathworks.com/help/coder/nvidia/ug/publish-and-subscribe-to-mqtt-messages.html
2. A Comprehensive Guide to Utilizing the MQTT Protocol - C# Corner, 8월 4, 2025에 액세스, https://www.c-sharpcorner.com/article/a-comprehensive-guide-to-utilizing-the-mqtt-protocol/
3. Even Lower Latency in IIoT: Evaluation of QUIC in Industrial IoT Scenarios - PMC, 8월 4, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC8434189/
4. mqtt-v3.1.1-os.html - MQTT Version 3.1.1 - OASIS Open, 8월 4, 2025에 액세스, https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html
5. MQTT over QUIC | EMQX Docs, 8월 4, 2025에 액세스, https://docs.emqx.com/en/emqx/latest/mqtt-over-quic/introduction.html
6. MQTT Over QUIC: The Next-Generation IoT Standard Protocol, 8월 4, 2025에 액세스, https://www.iotforall.com/mqtt-over-quic-next-generation-iot-standard-protocol
7. MQTT over QUIC: Next-Generation IoT Standard Protocol | EMQ - EMQX, 8월 4, 2025에 액세스, https://www.emqx.com/en/blog/mqtt-over-quic
8. Delay and Energy Consumption of MQTT over QUIC: An Empirical ..., 8월 4, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC9145551/
9. QUIC vs TCP: The Full Story - InApp, 8월 4, 2025에 액세스, https://inapp.com/blog/quic-vs-tcp-the-full-story/
10. getting-started-with-mqtt-over-quic-from-scratch.md - GitHub, 8월 4, 2025에 액세스, https://github.com/emqx/blog/blob/main/en/202212/getting-started-with-mqtt-over-quic-from-scratch.md
11. Comparative Study of Messaging Protocols Used in Mobile Software Architecture, 8월 4, 2025에 액세스, https://mobile.fhstp.ac.at/wp-content/uploads/2023/12/paper_it231519_Gruber.pdf
12. QUIC vs TCP: Which is Better? - Fastly, 8월 4, 2025에 액세스, https://www.fastly.com/blog/measuring-quic-vs-tcp-computational-efficiency
13. QUIC vs TCP: “The Ultimate Benchmark for Modern Network Performance” | by Abhi Patel, 8월 4, 2025에 액세스, https://medium.com/@abhi.patel.7172/quic-vs-tcp-the-ultimate-benchmark-for-modern-network-performance-22be6e5af66d
14. MQTT concepts - FairCom Documentation, 8월 4, 2025에 액세스, https://docs.faircom.com/docs/en/UUID-e7ed5dcd-e121-0069-52a9-6b9dde5c2230.html
15. On MQTT Scalability in the Internet of Things: Issues, Solutions, and Future Directions, 8월 4, 2025에 액세스, https://www.researchgate.net/publication/371863086_On_MQTT_Scalability_in_the_Internet_of_Things_Issues_Solutions_and_Future_Directions
16. MQTT vs. QUIC: Which protocol is better for IoT applications? - Business software and IT blog - We design digital value creation, 8월 4, 2025에 액세스, https://blog.doubleslash.de/en/iot-and-connected-things/mqtt-vs-quic-which-protocol-is-better-for-iot-applications/
17. What is MQTT Quality of Service (QoS) 0,1, & 2? – MQTT Essentials: Part 6 - HiveMQ, 8월 4, 2025에 액세스, https://www.hivemq.com/blog/mqtt-essentials-part-6-mqtt-quality-of-service-levels/
18. RFC 9431 - Message Queuing Telemetry Transport (MQTT) and Transport Layer Security (TLS) Profile of Authentication and Authorization for Constrained Environments (ACE) Framework - IETF Datatracker, 8월 4, 2025에 액세스, https://datatracker.ietf.org/doc/rfc9431/
19. QUIC is not Quick Enough over Fast Internet : r/programming - Reddit, 8월 4, 2025에 액세스, https://www.reddit.com/r/programming/comments/1g7vv66/quic_is_not_quick_enough_over_fast_internet/
20. QUIC is not Quick Enough over Fast Internet - arXiv, 8월 4, 2025에 액세스, https://arxiv.org/html/2310.09423v2
21. Implementation and Performance Evaluation of TCP over QUIC Tunnels - arXiv, 8월 4, 2025에 액세스, https://arxiv.org/html/2504.10054v1
22. Implementation and Analysis of QUIC for MQTT - arXiv, 8월 4, 2025에 액세스, https://arxiv.org/pdf/1810.07730
23. QUIC Protocol: the Features, Use Cases and Impact for IoT/IoV | by EMQ Technologies, 8월 4, 2025에 액세스, https://emqx.medium.com/quic-protocol-the-features-use-cases-and-impact-for-iot-iov-10f27441ebe
24. Implementation and Analysis of QUIC for MQTT, 8월 4, 2025에 액세스, https://www.cse.scu.edu/~bdezfouli/publication/QUIC-MQTT-COMNET2019.pdf
25. Overcoming Address Change with MQTT over QUIC | by EMQ Technologies - Medium, 8월 4, 2025에 액세스, https://emqx.medium.com/overcoming-address-change-with-mqtt-over-quic-92cec88ba31f
26. Overcoming Address Change with MQTT over QUIC - DEV Community, 8월 4, 2025에 액세스, https://dev.to/emqx/overcoming-address-change-with-mqtt-over-quic-1542
27. MQTT over QUIC: A New Standard for Connected Vehicles | IoT For All, 8월 4, 2025에 액세스, https://www.iotforall.com/webinars/mqtt-over-quic-a-new-standard-for-connected-vehicles
28. MQTT over QUIC: The Next-Generation IoT Standard Protocol Brings New Impetus to Messaging Scenarios - SegmentFault 思否, 8월 4, 2025에 액세스, https://segmentfault.com/a/1190000042239884/en
29. Implementation and Analysis of QUIC for MQTT - ResearchGate, 8월 4, 2025에 액세스, https://www.researchgate.net/publication/328380610_Implementation_and_Analysis_of_QUIC_for_MQTT
30. A Pure HTTP/3 Alternative to MQTT-over-QUIC in Resource-Constrained IoT - arXiv, 8월 4, 2025에 액세스, https://arxiv.org/pdf/2106.12684
31. Get Started with MQTT over QUIC: A Quick Guide for The Next-generation IoT Standard Protocol - EMQX, 8월 4, 2025에 액세스, https://www.emqx.com/en/blog/getting-started-with-mqtt-over-quic-from-scratch
32. Implementation and analysis of QUIC for MQTT | Request PDF - ResearchGate, 8월 4, 2025에 액세스, https://www.researchgate.net/publication/329835020_Implementation_and_analysis_of_QUIC_for_MQTT
33. And QUIC meets IoT: performance assessment of MQTT over QUIC - ResearchGate, 8월 4, 2025에 액세스, https://www.researchgate.net/publication/346886406_And_QUIC_meets_IoT_performance_assessment_of_MQTT_over_QUIC
34. MQTT/QUIC vs MQTT/TCP Performance Demo - YouTube, 8월 4, 2025에 액세스, https://www.youtube.com/watch?v=7QAI1mTm_A4
35. MQTT in Connected Cars: Key Benefits and Real-World Applications ..., 8월 4, 2025에 액세스, https://www.emqx.com/en/blog/mqtt-for-internet-of-vehicles
36. MQTT & QUIC - The Future of Connected Vehicle Services - YouTube, 8월 4, 2025에 액세스, https://www.youtube.com/watch?v=oMr_uEirMIY
37. Shaping the Future of IoT: 7 MQTT Technology Trends in 2023 | EMQ - EMQX, 8월 4, 2025에 액세스, https://www.emqx.com/en/blog/7-mqtt-trends-in-2023
38. broker(emqx) already support QUIC, and i Hope client also supports the Quic protocol · Issue #470 · eclipse-paho/paho.mqtt.cpp - GitHub, 8월 4, 2025에 액세스, https://github.com/eclipse-paho/paho.mqtt.cpp/issues/470
39. mquictt/mquictt: MQTT over QUIC, in rust! - GitHub, 8월 4, 2025에 액세스, https://github.com/mquictt/mquictt
40. MQTT over QUIC Documentation #13934 - GitHub, 8월 4, 2025에 액세스, https://github.com/emqx/emqx/discussions/13934
41. Media Over QUIC (moq) - Datatracker - IETF, 8월 4, 2025에 액세스, https://datatracker.ietf.org/group/moq/about/


# MQTT-SN(MQTT for Sensor Networks)

## 서론: MQTT 프로토콜의 탄생과 진화

사물 인터넷(Internet of Things, IoT) 시대의 도래와 함께, 수십억 개의 디바이스가 생성하는 데이터를 효율적으로 전송하고 처리하는 기술의 중요성은 그 어느 때보다 부각되고 있다. 이러한 배경 속에서 MQTT(Message Queuing Telemetry Transport) 프로토콜은 경량 메시징 프로토콜의 표준으로 확고히 자리 잡았다. MQTT-SN(MQTT for Sensor Networks)을 깊이 이해하기 위해서는 먼저 그 모태가 된 MQTT의 역사적 배경과 핵심 설계 철학을 이해하는 것이 필수적이다.

### 0.1 MQTT의 역사적 배경 및 설계 철학

MQTT 프로토콜은 1999년 IBM의 Andy Stanford-Clark와 당시 Eurotech 소속이었던 Arlen Nipper에 의해 처음 개발되었다.1 그들의 초기 목표는 매우 구체적이고 현실적인 문제, 즉 SCADA(Supervisory Control and Data Acquisition) 산업 제어 시스템 내에서 석유 및 가스 파이프라인을 원격으로 모니터링하는 것이었다.1 당시 통신 환경은 위성 링크에 크게 의존했으며, 이는 데이터 전송 비용이 극도로 높고 대역폭이 매우 제한적임을 의미했다.1 이러한 극한의 제약 조건 하에서, 개발팀은 최소한의 대역폭을 사용하고, 디바이스의 배터리 소모를 최소화하는 것을 프로토콜 설계의 최우선 과제로 삼았다. 이처럼 MQTT는 태생부터 '자원 효율성'과 '경량성'을 핵심 가치로 내재하고 있었다.

초기에는 IBM의 MQ(Message Queue) 제품군에서 파생되었음을 암시하는 'MQ Telemetry Transport'라는 이름으로 불렸으나, 프로토콜의 본질은 메시지 큐가 아닌 발행/구독 모델에 기반한다.1 2013년 IBM이 MQTT v3.1을 표준화 기구인 OASIS에 제출한 이후, 'MQTT'는 공식적으로 어떤 단어의 약어도 아닌 고유명사로 사용되고 있다.1 현재 MQTT는 OASIS 표준이자 국제 표준화 기구(ISO)의 권고안(ISO/IEC 20922)으로 인정받으며, 자동차, 제조, 통신, 스마트 홈 등 다양한 산업 분야에서 폭넓게 활용되고 있다.1

### 0.2 발행/구독(Publish/Subscribe) 모델과 브로커 아키텍처의 이해

MQTT의 가장 큰 구조적 특징은 전통적인 클라이언트-서버 모델이 아닌 발행/구독(Publish/Subscribe) 메시징 패턴을 채택했다는 점이다.2 이 모델은 메시지를 생성하는 주체(발행자, Publisher)와 메시지를 소비하는 주체(구독자, Subscriber) 사이의 직접적인 연결을 제거하고, '메시지 브로커(Message Broker)'라는 중개자를 통해 양측을 완전히 분리(decouple)한다.

이러한 분리 구조는 세 가지 중요한 차원에서의 독립성을 보장한다.2

1. **공간적 분리(Space Decoupling):** 발행자와 구독자는 서로의 네트워크 위치(IP 주소, 포트 번호 등)를 전혀 알 필요가 없다. 모든 통신은 브로커를 통해서만 이루어진다.
2. **시간적 분리(Time Decoupling):** 발행자와 구독자가 동시에 네트워크에 연결되어 있을 필요가 없다. 발행자는 구독자의 온라인 여부와 상관없이 메시지를 브로커에 발행할 수 있으며, 브로커는 구독자가 접속했을 때 해당 메시지를 전달할 수 있다 (지속 세션 기능 활용 시).
3. **동기적 분리(Synchronization Decoupling):** 발행 작업과 구독 작업은 서로를 차단하지 않는다. 발행자는 메시지를 보내고 즉시 다른 작업을 수행할 수 있으며, 구독자는 메시지를 기다리며 대기할 필요 없이 비동기적으로 메시지를 수신할 수 있다.

이 아키텍처는 다음과 같은 핵심 구성 요소로 이루어진다.

- **MQTT 클라이언트(Client):** MQTT 라이브러리를 실행하여 네트워크 통신을 수행하는 모든 디바이스를 지칭한다. 마이크로컨트롤러부터 대규모 서버까지 그 형태는 다양하다. 클라이언트는 메시지를 보낼 때는 발행자(Publisher)로, 메시지를 받을 때는 구독자(Subscriber)로 동작한다.1
- **MQTT 브로커(Broker):** 발행/구독 모델의 중심축 역할을 하는 서버다. 모든 클라이언트로부터 메시지를 수신하고, 메시지에 포함된 '토픽(Topic)'을 기반으로 해당 토픽을 구독하고 있는 클라이언트들에게 메시지를 정확하게 라우팅(routing)한다. 이 외에도 클라이언트 인증 및 권한 부여, 메시지 필터링, 세션 관리 등 시스템의 안정성과 보안을 책임지는 중요한 역할을 수행한다.1

통신 흐름은 클라이언트가 브로커에 `CONNECT` 메시지를 보내 연결을 요청하는 것으로 시작되며, 브로커는 `CONNACK` 메시지로 응답하여 연결 수립을 확인한다. 모든 통신은 브로커를 중심으로 이루어지며, 클라이언트 간의 직접적인 통신은 허용되지 않는다. 이 모든 과정은 신뢰성 있는 양방향 연결을 제공하는 TCP/IP 스택을 기반으로 한다.1

MQTT의 근본적인 설계 원칙인 '분리(decoupling)'는 시스템의 유연성과 확장성을 극대화하는 강력한 패러다임이다. 그러나 MQTT 버전 3.1.1까지 이 철학은 TCP라는 안정적이고 연결 지향적인 전송 계층에 강하게 의존하고 있었다. TCP는 신뢰성 있는 데이터 전송을 보장하지만, 연결 설정(3-way handshake), 연결 유지(Keep-Alive), 재전송 메커니즘 등으로 인한 오버헤드가 필연적으로 발생한다. Zigbee, Bluetooth LE와 같은 무선 센서 네트워크(WSN) 환경은 본질적으로 비연결성(connectionless), 비신뢰성(unreliable), 그리고 비-IP(non-IP) 기반의 특성을 갖는다. 이러한 환경에서 TCP 스택을 구현하고 유지하는 것은 메모리, 처리 능력, 그리고 무엇보다 전력 소모 측면에서 엄청난 부담으로 작용한다.5 따라서, MQTT의 핵심 철학을 WSN의 물리적, 기술적 제약 조건에 맞게 확장하기 위해서는, TCP라는 전제 조건 자체로부터 프로토콜을 '분리'해낼 필요성이 대두되었다. 이러한 배경에서 MQTT-SN의 등장은 단순히 MQTT를 경량화한 변종이 아니라, MQTT의 핵심 철학을 전송 계층의 제약으로부터 해방시켜 UDP나 비-IP 네트워크에서도 구현할 수 있도록 만든 필연적인 진화의 산물이라 할 수 있다.

## 1. MQTT-SN의 필요성 및 등장 배경

MQTT가 IoT 메시징의 표준으로 자리 잡았음에도 불구하고, 그 적용 범위에는 명확한 한계가 존재했다. 특히 극단적인 자원 제약 환경, 즉 무선 센서 네트워크(WSN)에서는 MQTT의 기본 전제들이 오히려 큰 장벽으로 작용했다. MQTT-SN은 바로 이러한 한계를 극복하고 MQTT의 생태계를 진정한 '사물'의 영역까지 확장하기 위해 탄생했다.

### 1.1 기존 MQTT의 한계: 자원 제약적 환경에서의 도전 과제

표준 MQTT 프로토콜은 여러 측면에서 WSN 환경에 부적합했다.

- **TCP/IP 스택의 부담:** MQTT는 메시지 전달의 신뢰성을 보장하기 위해 전송 계층 프로토콜로 TCP/IP에 의존한다.1 TCP는 연결 설정, 오류 제어, 흐름 제어 등 복잡하고 정교한 메커니즘을 통해 안정적인 통신을 제공하지만, 이는 상당한 자원을 소모한다. 저사양 마이크로컨트롤러(MCU)와 같이 메모리가 수십 킬로바이트(KB)에 불과하고 처리 능력이 낮은 디바이스에게 완전한 TCP/IP 스택을 구현하고 실행하는 것은 매우 큰 부담이다.5 특히 3-way handshake 과정, 주기적인 Keep-Alive 패킷 교환, 패킷 유실 시 재전송 로직 등은 배터리로 수년 간 동작해야 하는 센서 노드의 전력 소모를 급격히 증가시키는 요인이 된다.6
- **긴 토픽 이름의 비효율성:** MQTT는 토픽(Topic)을 기반으로 메시지를 필터링하며, 이 토픽은 계층 구조를 가진 긴 UTF-8 문자열로 표현된다 (예: `factory/zone1/machineA/temperature`).1 이러한 방식은 토픽을 직관적으로 이해하고 관리하는 데 도움을 주지만, WSN 환경에서는 심각한 비효율을 초래한다. 저대역폭 무선 통신(예: Zigbee, LoRaWAN)에서는 전송할 수 있는 페이로드(payload)의 크기가 매우 제한적이다. 매번 메시지를 발행(PUBLISH)할 때마다 이 긴 토픽 문자열을 포함시키는 것은 귀중한 대역폭을 낭비하는 일이며, 디바이스의 제한된 RAM에 여러 개의 긴 토픽 문자열을 저장하는 것 또한 부담이 된다.5
- **비-IP 네트워크 지원 부재:** 사물 인터넷을 구성하는 상당수의 무선 통신 기술, 특히 Zigbee, Z-Wave, Bluetooth Low Energy(BLE) 등은 IP(Internet Protocol) 기반이 아니다.1 이들 프로토콜은 자체적인 주소 지정 방식과 네트워크 스택을 가지고 있다. TCP/IP 위에서 동작하도록 설계된 표준 MQTT는 이러한 비-IP 네트워크 환경에서 직접적으로 동작할 수 없으며, IP 네트워크와의 연동을 위해서는 별도의 복잡한 게이트웨이 로직이 필요했다.

### 1.2 무선 센서 네트워크(WSN)를 위한 경량화 프로토콜의 요구

WSN은 MQTT가 설계될 당시의 일반적인 네트워크 환경과는 근본적으로 다른 특징을 가진다.5

- **극심한 자원 제약:** 노드는 수년 간 교체 없이 동작해야 하므로 극도의 저전력 소모가 요구된다. 또한, 비용 절감을 위해 최소한의 처리 능력(CPU)과 저장 공간(RAM/Flash)을 갖춘다.
- **제한된 통신 능력:** 통신은 저대역폭, 높은 패킷 손실률, 작은 페이로드 크기라는 특징을 갖는다.
- **간헐적 연결:** 전력 소모를 줄이기 위해 대부분의 시간을 절전(sleep) 상태로 보내고, 아주 짧은 시간 동안만 깨어나 데이터를 전송하고 다시 잠드는 방식으로 동작한다.

이러한 WSN의 고유한 제약 조건들을 해결하기 위해서는 MQTT의 핵심 철학인 발행/구독 모델의 장점은 유지하면서도, 프로토콜 자체를 WSN 환경에 맞게 근본적으로 재설계할 필요가 있었다.12 바로 이 지점에서 MQTT-SN(MQTT for Sensor Networks)이 등장했다. MQTT-SN은 비-TCP/IP 기반의 임베디드 디바이스를 위한 애플리케이션 계층 통신 표준을 제공하는 것을 목표로, 기존 MQTT의 한계를 극복하도록 설계되었다.11

전통적인 네트워크 프로토콜 설계는 OSI 7계층 모델에 따라 각 계층의 독립성을 강조한다. 애플리케이션 계층(MQTT)은 하위 전송 계층(TCP)이 제공하는 신뢰성을 당연한 전제로 삼고 설계되었다. 그러나 WSN과 같은 네트워크의 가장자리(edge) 환경에서는 이러한 이상적인 계층 분리 모델이 더 이상 유효하지 않다. 물리 계층(저전력 라디오)과 데이터링크 계층(짧은 프레임)의 물리적 제약이 상위 계층 프로토콜 설계에 직접적인 영향을 미치기 때문이다. 즉, '전력'과 '대역폭'이라는 물리적 자원이 프로토콜 설계의 가장 중요한 변수가 된다.

MQTT-SN의 설계 변경점들은 이러한 관점에서 해석될 수 있다. 긴 토픽 이름을 짧은 2바이트 `Topic ID`로 대체한 것은 데이터링크 계층의 작은 페이로드 크기 제약을 극복하기 위한 필연적인 선택이었다.9 TCP 대신 UDP를 사용하고, 프로토콜 수준에서 명시적인 

`절전 모드`를 지원하는 것은 물리 계층의 저전력 요구사항을 직접적으로 반영한 결과다.9 또한, 노드가 동적으로 네트워크에 참여하고 이탈하는 WSN의 특성을 고려하여 

`게이트웨이 탐색` 기능을 추가한 것 역시 마찬가지다.9 이는 MQTT-SN이 단순히 MQTT의 기능을 축소한 '경량화 버전'이 아니라, 하위 계층의 물리적, 환경적 제약을 상위 애플리케이션 계층 프로토콜 설계에 적극적으로 통합한 '수직적 최적화(Cross-layer optimization)'의 대표적인 사례임을 보여준다. 따라서 MQTT-SN의 등장은 IoT 패러다임이 중앙 클라우드 중심에서 분산된 엣지 중심으로 확장됨에 따라, 프로토콜 설계 철학 또한 이상적인 모델에서 현실적인 제약을 고려한 통합 모델로 진화하고 있음을 보여주는 중요한 증거라 할 수 있다.

## 2. MQTT-SN 핵심 아키텍처 분석

MQTT-SN은 자원이 제한된 센서 네트워크와 기존의 강력한 MQTT 인프라를 연결하기 위해 독자적인 아키텍처를 채택했다. 이 아키텍처의 핵심에는 '게이트웨이(Gateway)'가 있으며, 이를 중심으로 클라이언트, 포워더, 그리고 MQTT 브로커가 유기적으로 상호작용한다.

### 2.1 주요 구성 요소: 클라이언트, 게이트웨이, 포워더, 브로커

MQTT-SN 생태계는 다음과 같은 네 가지 주요 구성 요소로 이루어진다.

- **MQTT-SN 클라이언트(Client):** 일반적으로 WSN 내에 존재하는 개별 센서나 액추에이터 노드를 의미한다. 이들은 제한된 메모리, 처리 능력, 배터리 용량을 가지며, TCP/IP 스택을 내장하기 어렵다. 따라서 MQTT-SN 프로토콜을 사용하여 게이트웨이와 통신하며, 이를 통해 MQTT 네트워크에 간접적으로 참여한다.5
- **MQTT-SN 게이트웨이(Gateway, GW):** MQTT-SN 아키텍처의 심장부라 할 수 있는 핵심 구성 요소다. 주된 역할은 서로 다른 두 네트워크, 즉 WSN(UDP, Zigbee 등 비-IP 기반)과 기존 IP 네트워크(TCP/IP 기반) 사이에서 다리 역할을 수행하는 것이다. 이를 위해 MQTT-SN 프로토콜로 수신된 메시지를 표준 MQTT 프로토콜로 변환하여 MQTT 브로커에 전달하고, 그 반대의 변환 작업도 수행한다.5 게이트웨이는 MQTT 브로커와 물리적으로 통합되어 하나의 서버에서 동작할 수도 있고, 완전히 독립된 장비로 존재할 수도 있다.5
- **MQTT-SN 포워더(Forwarder):** 게이트웨이가 클라이언트가 속한 WSN에 직접 연결되어 있지 않은 경우, 중계기 역할을 수행하는 선택적 구성 요소다. 포워더의 기능은 매우 단순하다. 무선 측(클라이언트)에서 수신한 MQTT-SN 프레임을 그대로 캡슐화하여 변경 없이 게이트웨이로 전달하고, 게이트웨이로부터 수신한 프레임은 캡슐화를 해제하여 클라이언트로 전달한다.5 이는 네트워크 토폴로지의 유연성을 높여준다.
- **MQTT 브로커(Broker):** 표준 MQTT 아키텍처와 동일한 역할을 수행하는 중앙 서버다. 게이트웨이를 통해 변환되어 전달된 MQTT 메시지를 수신하고, 토픽 기반으로 구독자들에게 메시지를 분배한다. 브로커는 MQTT-SN 클라이언트의 존재를 직접 인지하지 못하며, 오직 게이트웨이를 하나의 일반 MQTT 클라이언트(또는 클라이언트들의 집합)로 간주하고 통신한다.1

### 2.2 게이트웨이의 역할과 유형 심층 분석

게이트웨이는 단순한 프로토콜 변환기를 넘어, WSN 환경의 특수성을 수용하기 위한 다양한 지능적인 기능을 수행한다.

- **프로토콜 변환(Protocol Translation):** 가장 기본적인 기능이다. 예를 들어, MQTT-SN 클라이언트가 2바이트 `TopicId`를 사용하여 메시지를 발행하면, 게이트웨이는 내부 매핑 테이블을 참조하여 이를 완전한 `TopicName` 문자열로 변환한 후, 표준 MQTT `PUBLISH` 패킷을 만들어 브로커로 전송한다.5
- **게이트웨이 탐색 지원(Gateway Discovery):** WSN에 새로 참여한 클라이언트가 통신할 게이트웨이를 동적으로 찾을 수 있도록 돕는다. 게이트웨이는 주기적으로 자신의 존재를 알리는 `ADVERTISE` 메시지를 네트워크에 브로드캐스팅하거나, 클라이언트가 보낸 `SEARCHGW` 탐색 요청에 `GWINFO` 메시지로 응답한다.5
- **절전 클라이언트 지원(Sleeping Client Support):** MQTT-SN의 핵심 기능 중 하나로, 배터리로 동작하는 클라이언트의 수명을 극대화한다. 클라이언트가 절전 모드로 전환하겠다고 알리면, 게이트웨이는 해당 클라이언트로 향하는 모든 메시지를 임시 저장소(버퍼)에 보관한다. 이후 클라이언트가 깨어나 `PINGREQ` 메시지를 보내면, 게이트웨이는 버퍼에 저장해 두었던 메시지들을 순차적으로 클라이언트에게 전송한다.15

게이트웨이는 MQTT 브로커와의 연결을 관리하는 방식에 따라 두 가지 주요 유형으로 나뉜다.

- **투명 게이트웨이(Transparent Gateway):** 이 방식은 각 MQTT-SN 클라이언트에 대해 개별적인 MQTT 연결을 브로커와 수립한다. 예를 들어, 100개의 센서 노드가 투명 게이트웨이에 연결되어 있다면, 게이트웨이는 브로커와 100개의 독립적인 TCP 연결을 생성하고 유지한다. 이 방식은 각 클라이언트의 세션을 브로커가 직접 관리할 수 있어 구현이 비교적 간단하다는 장점이 있다. 하지만 클라이언트 수가 많아질수록 브로커에 가해지는 연결 부하가 커져 확장성에 한계가 있을 수 있다.5
- **집계 게이트웨이(Aggregating Gateway):** 이 방식은 게이트웨이에 연결된 모든 MQTT-SN 클라이언트들을 대표하여 단 하나의 MQTT 연결만을 브로커와 수립한다. 게이트웨이는 이 단일 연결을 통해 모든 클라이언트의 메시지를 다중화(multiplexing)하여 전송한다. 이 방식은 수천, 수만 개의 센서 노드가 있는 대규모 WSN 환경에서 브로커의 부하를 획기적으로 줄여주므로 확장성이 매우 뛰어나다. 그러나 게이트웨이가 모든 클라이언트의 세션 상태(구독 정보, Will 메시지 등)를 자체적으로 관리해야 하므로 구현이 훨씬 복잡하고, 게이트웨이 자체의 자원 요구사항이 높아진다.5

표면적으로 게이트웨이의 역할은 프로토콜 변환에 국한되는 것처럼 보일 수 있다. 하지만 그 내부 동작을 깊이 들여다보면, 게이트웨이는 표준 MQTT 브로커가 안정적인 TCP 연결 위에서 제공하던 핵심 서비스들을, 자원이 제약되고 비연결성인 WSN 환경에 맞게 재구현하는 역할을 수행한다. 예를 들어, MQTT 브로커는 TCP 세션을 통해 클라이언트의 '연결 상태'를 자연스럽게 인지한다. 반면, MQTT-SN 게이트웨이는 비연결성 UDP 위에서 `PINGREQ/PINGRESP` 교환과 `Keep Alive` 타이머를 통해 '가상 연결(virtual connection)' 상태를 능동적으로 관리해야 한다.9 또한, MQTT 브로커는 

`Clean Session=false` 옵션을 통해 클라이언트를 위한 메시지를 큐에 저장한다. MQTT-SN 게이트웨이는 한 걸음 더 나아가, 클라이언트가 `DISCONNECT` 메시지를 통해 명시한 `Duration`(절전 시간) 정보를 바탕으로 '절전 상태'의 클라이언트를 위한 메시지를 능동적으로 버퍼링한다.15 이는 단순한 큐잉을 넘어 클라이언트의 전력 상태를 인지하고 관리하는 고도의 상태 관리 기능이다. 이처럼 MQTT-SN 게이트웨이는 단순한 번역기가 아니라, WSN을 위한 '가상 브로커'이자 '상태 관리자'로서, 자원이 풍부한 IP 세계와 자원이 희소한 센서 세계를 잇는 지능적인 중재자 역할을 수행한다. 이 점을 이해하는 것이 MQTT-SN 아키텍처의 본질을 파악하는 핵심이다.

## 3. MQTT와 MQTT-SN의 프로토콜 수준 비교 분석

MQTT-SN은 MQTT의 발행/구독 모델을 계승하면서도, WSN 환경에 최적화되기 위해 프로토콜의 여러 측면에서 근본적인 변화를 주었다. 이 두 프로토콜의 기술적 차이점을 상세히 비교 분석하는 것은 특정 IoT 애플리케이션에 가장 적합한 프로토콜을 선택하는 데 있어 결정적인 정보를 제공한다.

### 3.1 전송 프로토콜과 네트워크 독립성

가장 근본적인 차이는 전송 계층에서 비롯된다. 표준 MQTT는 메시지 전달의 순서와 신뢰성을 보장하기 위해 연결 지향적인 TCP/IP 프로토콜 위에서 동작하도록 설계되었다.1 이는 안정적인 유선 또는 무선 IP 네트워크 환경에서는 큰 장점이지만, TCP 스택을 지원하지 않거나 지원하기에 부담스러운 저사양 임베디드 환경에서는 사용할 수 없다는 명백한 한계를 가진다.

반면, MQTT-SN은 이러한 의존성에서 벗어나도록 설계되었다. 주로 비연결성 프로토콜인 UDP 위에서 동작하며, 더 나아가 Zigbee, Bluetooth, 심지어 단순 직렬 통신(UART)과 같은 비-IP 기반의 다양한 통신 매체 위에서도 동작할 수 있다.5 이는 MQTT-SN이 특정 전송 기술에 얽매이지 않는 '전송 매체 독립성'을 확보했음을 의미하며, IoT의 가장자리(edge)에 존재하는 거의 모든 종류의 저전력 무선 기술과 통합될 수 있는 유연성을 제공한다.

### 3.2 메시지 구조와 토픽 표현의 최적화

네트워크 대역폭과 디바이스 메모리를 절약하기 위해 MQTT-SN은 메시지 구조를 대폭 간소화했다. 표준 MQTT의 패킷은 최소 2바이트의 고정 헤더와 가변 헤더, 그리고 페이로드로 구성된다.1 MQTT-SN 역시 유사한 구조를 가지지만, 전체적인 패킷 크기를 줄이기 위한 여러 장치를 마련했다.

가장 혁신적인 변화는 '토픽 표현' 방식이다. MQTT에서는 `home/floor1/livingroom/temperature`와 같은 긴 UTF-8 문자열을 토픽 이름으로 사용하며, 이 문자열은 메시지를 발행하거나 구독할 때마다 전송되어야 한다.1 MQTT-SN에서는 이러한 비효율을 제거하기 위해 '토픽 ID(

`TopicId`)'라는 개념을 도입했다. 클라이언트는 통신 초기에 사용하고자 하는 긴 토픽 이름을 게이트웨이에 한 번만 등록(`REGISTER`)하고, 2바이트 길이의 숫자형 `TopicId`를 할당받는다. 이후 모든 `PUBLISH` 및 `SUBSCRIBE` 메시지에는 이 짧은 `TopicId`만을 사용하여 통신한다.5 이 방식은 반복적인 메시지 전송에서 발생하는 오버헤드를 극적으로 감소시켜, 제한된 페이로드 크기를 가진 WSN 환경에서 통신 효율을 극대화한다.

### 3.3 연결 설정 및 서비스 품질(QoS)의 확장

연결 설정 과정에서도 최적화를 위한 차이가 드러난다. MQTT에서는 `CONNECT` 메시지 하나에 클라이언트 ID, 세션 정보, 인증 정보(Username/Password), 그리고 유언 메시지(Last Will and Testament) 관련 정보가 모두 포함된다.2 MQTT-SN에서는 `CONNECT` 패킷을 최대한 가볍게 만들기 위해, 선택적으로 사용되는 유언 메시지 관련 정보를 `WILLTOPIC`과 `WILLMSG`라는 별도의 메시지로 분리했다.9 클라이언트는 `CONNECT` 메시지의 'Will' 플래그를 통해서만 유언 메시지의 존재 유무를 알리고, 게이트웨이는 이 플래그가 설정된 경우에만 추가적인 요청을 보내 관련 정보를 획득한다. 이는 유언 기능을 사용하지 않는 대다수의 단순 센서들이 불필요한 데이터를 전송하지 않도록 하는 효율적인 설계다.

서비스 품질(Quality of Service, QoS) 측면에서 MQTT-SN은 MQTT의 세 가지 레벨(QoS 0: 최대 한 번, QoS 1: 최소 한 번, QoS 2: 정확히 한 번)을 모두 지원한다.3 여기에 더해, WSN 환경의 특수성을 고려한 독자적인 레벨인 **QoS -1(또는 QoS 3)**을 추가했다.4 QoS -1은 'Fire and forget' 방식으로, 클라이언트가 게이트웨이와 사전에 연결(`CONNECT`) 절차를 거치지 않고도 즉시 `PUBLISH` 메시지를 보낼 수 있게 한다. 이 메시지는 수신 확인(acknowledgement) 없이 단 한 번만 전송된다. 이는 주기적으로 상태를 보고하고 즉시 절전 모드로 돌아가야 하는 매우 단순한 센서 애플리케이션에 최적화된 기능이다.

### 3.4 아키텍처 및 부가 기능의 차이

아키텍처적으로 가장 큰 차이는 '게이트웨이'의 존재 유무다. MQTT에서는 모든 클라이언트가 브로커에 직접 연결되지만, MQTT-SN에서는 게이트웨이가 필수적인 중개자 역할을 수행한다.5 이는 프로토콜 변환을 위한 필수적인 구조이지만, 아키텍처의 복잡성을 증가시키고 게이트웨이가 단일 장애점(Single Point of Failure)이 될 수 있다는 단점을 내포한다.

이를 보완하기 위해 MQTT-SN은 MQTT에는 없는 '탐색 기능(Discovery)'을 제공한다. 클라이언트는 브로커의 주소를 사전에 알고 있어야 하는 MQTT와 달리, `SEARCHGW` 메시지를 브로드캐스팅하여 동적으로 주변의 게이트웨이를 찾거나, 게이트웨이가 주기적으로 보내는 `ADVERTISE` 메시지를 수신하여 네트워크에 참여할 수 있다.9 이는 대규모 센서 네트워크의 설치와 유지보수를 용이하게 하는 중요한 기능이다.

또한, 배터리 수명 연장을 위한 '절전 모드(Sleep Mode)' 지원은 MQTT-SN의 핵심적인 차별점이다. 클라이언트는 `DISCONNECT` 메시지를 보낼 때 자신이 절전 상태에 머무를 시간(`Duration`)을 게이트웨이에 알릴 수 있다. 게이트웨이는 이 시간 동안 해당 클라이언트로 가는 메시지를 버퍼링하고, 클라이언트가 깨어나 `PINGREQ`를 보내면 저장된 메시지를 전달해준다.12 이는 프로토콜 수준에서 저전력 동작을 체계적으로 지원하는 기능으로, 표준 MQTT에는 존재하지 않는다.

보안 측면에서는 현재 MQTT-SN v1.2 명세에 내장된 암호화나 강력한 인증 기능이 부족하다는 것이 주요 약점으로 지적된다. MQTT가 TLS/SSL을 통한 전송 계층 암호화와 Username/Password 기반 인증을 기본적으로 지원하는 것과 대조적이다.2 MQTT-SN 환경의 보안은 DTLS(Datagram TLS)와 같은 외부적인 수단에 의존해야 하며, 이는 구현의 복잡성을 가중시킨다.13 이 문제는 현재 진행 중인 v2.0 표준화 과정에서 주요 개선 과제로 다루어지고 있다.

다음 표는 MQTT와 MQTT-SN의 주요 기술적 차이점을 요약한 것이다.

**표 1: MQTT와 MQTT-SN의 핵심 기술 사양 비교**

| **항목**              | **MQTT (v3.1.1/v5)**                                         | **MQTT-SN (v1.2)**                                           | **비고 및 영향**                                             |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **전송 프로토콜**     | TCP/IP (연결 지향, 신뢰성) 1                                 | UDP, Zigbee, Bluetooth 등 (비연결성, 비-IP 지원) 5           | MQTT-SN은 TCP 스택이 없는 저사양 디바이스 및 비-IP 네트워크 지원. |
| **헤더/패킷 구조**    | 최소 2바이트 고정 헤더 + 가변 헤더 + 페이로드 1              | 2바이트 고정 헤더 (Length, MsgType) + 페이로드. 전체적으로 더 간결함 21 | 패킷 크기 최소화로 저대역폭 네트워크 효율 증대.              |
| **토픽 표현**         | UTF-8 문자열 (예: `a/b/c`) 1                                 | 2바이트 정수 `Topic ID` 5                                    | 메시지 크기 대폭 감소. 디바이스의 메모리 부담 경감.          |
| **토픽 ID 획득**      | 없음 (문자열 직접 사용)                                      | `REGISTER` 절차를 통해 동적 할당 또는 사전 정의(Pre-defined) 5 | 동적 등록 오버헤드 발생 가능성 vs. 사전 정의를 통한 최적화.  |
| **연결 설정**         | `CONNECT` 메시지 하나에 Will, 인증 정보 등 모두 포함 2       | `CONNECT` 메시지는 기본 정보만 포함. Will 관련 정보는 `WILLTOPIC`, `WILLMSG`로 분리 9 | 필요한 정보만 선택적으로 전송하여 패킷 크기 최적화.          |
| **QoS (서비스 품질)** | Level 0 (At most once), 1 (At least once), 2 (Exactly once) 3 | Level 0, 1, 2 + **Level -1 (or 3)** 추가 4                   | QoS -1: 연결 설정 없이 PUBLISH 가능. 'Fire and forget' 방식의 단순 센서에 최적화. |
| **게이트웨이**        | 불필요 (클라이언트가 브로커에 직접 연결)                     | **필수** (프로토콜 변환 및 상태 관리) 5                      | 아키텍처 복잡성 증가, 게이트웨이가 단일 장애점(SPOF)이 될 수 있음. |
| **탐색 기능**         | 없음 (브로커 주소를 미리 알아야 함)                          | 게이트웨이 탐색 (`SEARCHGW`, `ADVERTISE`) 기능 존재 9        | 동적인 네트워크 구성 및 플러그 앤 플레이(Plug-and-Play) 용이성 증대. |
| **절전 모드 지원**    | 프로토콜 자체 지원 없음 (TCP Keep-Alive에 의존)              | `DISCONNECT` 시 절전 기간(duration) 명시, `PINGREQ`로 깨어남. 게이트웨이가 메시지 버퍼링 12 | 배터리로 동작하는 디바이스의 수명 극대화.                    |
| **보안**              | TLS/SSL 통한 암호화, Username/Password 인증 지원 2           | v1.2 명세에 내장된 보안 기능 **부재**. DTLS 등 외부 수단 필요 13 | 보안 구현이 복잡하고 파편화될 수 있음. v2.0에서 개선 논의 중. |

## 4. MQTT-SN의 주요 동작 시퀀스 심층 탐구

MQTT-SN 프로토콜의 진정한 가치는 정적인 구조뿐만 아니라, 자원이 제한된 환경에서 통신을 설정하고, 데이터를 교환하며, 전력을 관리하는 동적인 동작 방식에 있다. 클라이언트가 네트워크에 참여하여 임무를 수행하고 절전 모드로 전환하는 일련의 과정을 시퀀스 다이어그램 형태로 분석하면 프로토콜의 효율성과 정교함을 깊이 이해할 수 있다.

### 4.1  게이트웨이 탐색(Gateway Discovery) 절차

WSN 환경에서는 센서 노드가 고정된 IP 주소를 갖지 않거나, 네트워크 토폴로지가 동적으로 변하는 경우가 많다. 이러한 환경에서 클라이언트가 통신할 게이트웨이를 수동으로 설정하는 것은 비효율적이다. MQTT-SN은 이를 해결하기 위해 두 가지 자동 탐색 메커니즘을 제공한다.

- **방법 1: 클라이언트 주도 탐색 (Client-initiated Search)**
  1. 새로 네트워크에 참여한 `클라이언트`는 통신할 게이트웨이를 찾기 위해 `SEARCHGW` 메시지를 네트워크 전체에 브로드캐스트한다. 이 메시지에는 탐색 범위를 나타내는 `Radius` 필드가 포함될 수 있다.15
  2. 이 메시지를 수신한 하나 이상의 `게이트웨이`는 자신의 정보(게이트웨이 ID 및 네트워크 주소)를 담은 `GWINFO` 메시지를 클라이언트에게 유니캐스트로 응답한다.19
  3. `클라이언트`는 `GWINFO` 메시지를 수신하여 통신할 게이트웨이를 선택하고 연결 절차를 시작한다.
- **방법 2: 게이트웨이 광고 (Gateway Advertisement)**
  1. `게이트웨이`는 자신의 존재를 알리기 위해 주기적으로 `ADVERTISE` 메시지를 네트워크에 브로드캐스트한다. 이 메시지에는 게이트웨이 ID와 이 광고가 유효한 시간(`Duration`)이 포함된다.5
  2. 네트워크를 감지하고 있는 `클라이언트`는 이 `ADVERTISE` 메시지를 수신하여 활성화된 게이트웨이 목록을 유지하고, 통신이 필요할 때 이 정보를 사용한다.10

이러한 탐색 메커니즘은 IoT 시스템의 '플러그 앤 플레이(Plug-and-Play)' 특성을 강화하여, 대규모 센서 네트워크의 배포 및 유지보수 편의성을 크게 향상시키는 핵심적인 기능이다.

### 4.2  클라이언트 연결 및 세션 관리

게이트웨이를 찾은 클라이언트는 가상 연결(virtual connection)을 설정하여 세션을 시작한다. 이 과정은 특히 유언(Will) 메시지 처리 방식에서 MQTT와 차이를 보인다.

1. `클라이언트`는 `게이트웨이`에게 `CONNECT` 메시지를 전송한다. 이 메시지에는 고유한 `ClientId`, 세션 유지 여부를 결정하는 `CleanSession` 플래그, 유언 메시지 사용 여부를 나타내는 `Will` 플래그, 그리고 연결 유지 시간(`KeepAlive Duration`)이 포함된다.19
2. 만약 `CONNECT` 메시지의 `Will` 플래그가 설정되어 있다면, `게이트웨이`는 유언 토픽을 요청하기 위해 `WILLTOPICREQ` 메시지를 클라이언트에게 보낸다.
3. `클라이언트`는 이에 응답하여 유언 메시지가 발행될 토픽 이름을 담은 `WILLTOPIC` 메시지를 전송한다.
4. 유언 토픽을 수신한 `게이트웨이`는 다시 유언 메시지의 내용을 요청하기 위해 `WILLMSGREQ` 메시지를 보낸다.
5. `클라이언트`는 실제 유언 메시지 내용을 담은 `WILLMSG` 메시지로 응답한다.
6. 모든 절차가 완료되면, `게이트웨이`는 연결 결과를 나타내는 `ReturnCode`를 포함한 `CONNACK` 메시지를 `클라이언트`에게 전송하며 연결이 최종적으로 수립된다.9

이처럼 유언 메시지 관련 절차를 별도의 요청/응답 과정으로 분리한 것은 MQTT-SN의 최적화 전략을 명확히 보여준다.9 대부분의 단순 센서 노드는 유언 기능을 사용하지 않으므로, 이 기능을 `CONNECT` 패킷에서 분리함으로써 초기 연결 패킷의 크기를 최소화하고 불필요한 데이터 전송을 방지할 수 있다.

### 4.3  토픽 등록 및 발행/구독 과정 (QoS 1 기준)

연결이 수립된 후, 클라이언트는 메시지를 발행하거나 구독하기 전에 토픽을 등록하는 과정을 거친다. 이는 MQTT-SN의 대역폭 효율성을 극대화하는 핵심 단계다.

1. **토픽 등록 (Topic Registration):**
   - 메시지를 발행하려는 `클라이언트`는 먼저 긴 토픽 이름(예: `sensor/1/data`)을 `REGISTER` 메시지에 담아 `게이트웨이`로 전송한다. 이 메시지에는 트랜잭션을 식별하기 위한 `MsgId`가 포함된다.5
   - `게이트웨이`는 이 토픽 이름을 수신하고, 내부적으로 이 이름에 매핑될 2바이트 `TopicId`를 할당한다. 이후 `게이트웨이`는 할당된 `TopicId`와 원래의 `MsgId`, 그리고 성공 여부를 나타내는 `ReturnCode`를 포함한 `REGACK` 메시지로 응답한다.
2. **메시지 발행 (Message Publishing):**
   - `TopicId`를 할당받은 `클라이언트`는 이제 실제 데이터를 `PUBLISH` 메시지에 담아 `게이트웨이`로 전송한다. 이때, 긴 토픽 이름 대신 할당받은 `TopicId`를 사용한다. QoS 1이므로 재전송을 관리하기 위한 `MsgId`도 함께 전송된다.6
   - `게이트웨이`는 `PUBLISH` 메시지를 성공적으로 수신했음을 알리기 위해 `PUBACK` 메시지를 `클라이언트`에게 응답한다.
   - 동시에, `게이트웨이`는 수신한 `TopicId`를 다시 긴 `TopicName`으로 변환하고, 표준 MQTT `PUBLISH` 패킷을 생성하여 `MQTT 브로커`로 전달한다.
   - `MQTT 브로커`는 이 메시지를 수신한 후, QoS 1 규칙에 따라 `PUBACK`을 `게이트웨이`에게 보낸다.

이 '등록' 과정은 클라이언트가 긴 토픽 이름을 단 한 번만 네트워크로 전송하고, 이후의 모든 반복적인 데이터 통신에서는 2바이트 `TopicId`만을 사용하게 함으로써, 통신 오버헤드를 획기적으로 줄이는 MQTT-SN의 핵심 최적화 메커니즘이다.

### 4.4  저전력 운영을 위한 절전 모드(Sleep Mode) 동작 방식

배터리로 구동되는 디바이스의 수명을 극대화하기 위한 절전 모드 동작은 MQTT-SN의 가장 정교하고 중요한 기능 중 하나다.

1. **절전 모드 진입:**
   - 데이터 전송을 마친 `클라이언트`는 전력 소모를 최소화하기 위해 절전 모드로 들어가기로 결정한다. 이때 클라이언트는 `게이트웨이`에게 `DISCONNECT` 메시지를 전송하는데, 이 메시지에는 자신이 절전 상태에 머무를 시간(`Duration`)을 명시할 수 있다.21
   - 이 메시지를 수신한 `게이트웨이`는 해당 클라이언트의 상태를 '활성(Active)'에서 '절전(Asleep)'으로 변경하고, 명시된 `Duration` 동안 이 클라이언트로 향하는 모든 메시지를 내부 버퍼에 저장하기 시작한다. 클라이언트는 메시지 전송 후 즉시 무선 통신 모듈의 전원을 차단하여 전력 소모를 최소화한다.
2. **절전 모드 해제 및 메시지 수신:**
   - 예정된 `Duration`이 지나거나, 새로운 데이터를 보낼 필요가 생겨 `클라이언트`가 깨어난다.
   - `클라이언트`는 자신의 `ClientId`를 포함한 `PINGREQ` 메시지를 `게이트웨이`에게 전송하여 자신이 다시 활성화되었음을 알린다.12
   - `PINGREQ`를 수신한 `게이트웨이`는 해당 `ClientId`의 버퍼를 확인한다. 만약 버퍼에 저장된 메시지가 있다면, `게이트웨이`는 `PINGRESP`를 즉시 보내지 않고, 대신 버퍼링된 `PUBLISH` 메시지들을 순차적으로 `클라이언트`에게 전송하기 시작한다.23
   - 버퍼에 있던 모든 메시지 전송이 완료되면, `게이트웨이`는 마지막으로 `PINGRESP` 메시지를 전송하여 버퍼가 비었음을 알린다.28
   - `클라이언트`는 `PINGRESP`를 수신함으로써 모든 밀린 메시지를 수신했음을 확인하고, 다시 절전 모드로 들어가거나 다음 작업을 수행한다.

MQTT-SN의 절전 모드는 단순한 '연결 끊기'가 아니다. 이는 클라이언트가 `Duration`이라는 명시적인 '계약 기간'을 제시하고, 게이트웨이가 이 기간 동안 클라이언트의 '대리인' 역할을 수행하기로 약속하는 '상태 기반 계약(Stateful Contract)'에 가깝다. 게이트웨이는 계약 기간 동안 클라이언트가 '살아있지만 잠들어 있는' 상태임을 인지하고, 그를 대신해 메시지를 안전하게 보관한다. 클라이언트가 `PINGREQ`로 깨어나는 행위는 이 계약에 따라 자신의 상태를 갱신하고, 계약 기간 동안 발생한 이벤트를 전달받는 과정이다. 이 정교한 상태 관리 모델은 비연결성 UDP 위에서도 신뢰성 있는 비동기 메시지 전달을 가능하게 하며, 배터리 수명을 극대화해야 하는 WSN 디바이스의 본질적인 요구사항과 완벽하게 부합한다. 이는 MQTT-SN이 단순한 통신 프로토콜을 넘어, WSN의 운영체제와 같은 역할을 일부 수행하고 있음을 시사한다.

## 5. 실제 적용 사례 및 생태계

MQTT-SN의 고유한 특징들은 이론적인 장점을 넘어, 다양한 실제 산업 현장에서 구체적인 가치를 창출하고 있다. 특히 전력 공급이 어렵고, 통신 환경이 열악하며, 수많은 소형 디바이스가 분산된 환경에서 MQTT-SN은 핵심적인 연결 기술로 부상하고 있다.

### 5.1 산업 IoT(IIoT) 및 예지 보전(Predictive Maintenance)

산업 현장은 MQTT-SN의 가치가 가장 잘 발휘될 수 있는 대표적인 분야다. 공장 내의 수많은 기계와 설비는 진동, 온도, 압력 등 다양한 상태 데이터를 지속적으로 생성하며, 이를 분석하여 고장을 사전에 예측하는 예지 보전은 IIoT의 핵심적인 응용 분야다.29

- **아키텍처:** 공장 내 회전 기계, 컨베이어 벨트, 배관 등 주요 설비에 진동 및 온도 센서가 부착된다. 이 센서들은 MQTT-SN 클라이언트로 동작하며, LoRaWAN이나 NB-IoT와 같은 저전력 광역 통신망(LPWAN) 또는 공장 내 사설 무선망을 통해 데이터를 엣지 게이트웨이로 전송한다.23 엣지 게이트웨이는 수신한 MQTT-SN 메시지를 표준 MQTT 메시지로 변환하여, 공장 내 엣지 서버나 원격 클라우드에 위치한 MQTT 브로커로 전달한다. 분석 플랫폼은 이 데이터를 구독하여 머신러닝 알고리즘을 통해 설비의 이상 징후를 감지하고 고장 시점을 예측하여 유지보수팀에 경고를 보낸다.29
- **MQTT-SN의 역할:** 많은 산업용 센서는 전원 케이블 설치가 어려운 위치에 부착되므로 배터리로 동작해야 한다. MQTT-SN의 절전 모드 지원은 이러한 무선 센서의 운영 수명을 수년 단위로 연장시켜 유지보수 비용을 획기적으로 절감한다.23 또한, 주기적으로 단순한 상태 값만 전송하는 센서의 경우, 연결 절차 없이 데이터를 전송할 수 있는 `QoS -1`을 사용하여 펌웨어 구현을 단순화하고 통신 오버헤드를 최소화할 수 있다.23

### 5.2 스마트 농업(Smart Agriculture)

광활한 농경지나 비닐하우스 단지와 같은 스마트 농업 환경은 분산된 센서 노드로부터 데이터를 수집해야 하는 전형적인 WSN 시나리오다.

- **아키텍처:** 넓은 농경지 곳곳에 설치된 토양 수분, 온도, pH 센서 노드들은 태양광 패널과 배터리로 동작하며 MQTT-SN 클라이언트 역할을 한다. 이 노드들은 수 킬로미터(km)까지 통신이 가능한 LoRaWAN을 통해 데이터를 원격 게이트웨이로 전송한다.31 LoRaWAN 네트워크 서버는 게이트웨이로부터 수신한 데이터를 MQTT 프로토콜로 변환하여 애플리케이션 서버의 MQTT 브로커로 전달하는 역할을 한다. 농부는 스마트폰 애플리케이션(MQTT 구독자)을 통해 농장 전체의 상태를 실시간으로 모니터링하고, 필요에 따라 관수 밸브나 환풍기 같은 액추에이터를 원격으로 제어할 수 있다.34

- **MQTT-SN의 역할:** LoRaWAN과 같은 LPWAN은 물리 계층과 데이터링크 계층을 정의하지만, 애플리케이션 계층 프로토콜은 별도로 정의하지 않는다. 또한, 이들 네트워크는 일반적으로 TCP/IP를 지원하지 않는다. MQTT-SN은 이러한 비-IP 네트워크 환경 위에서 경량의 발행/구독 메시징 모델을 구현할 수 있는 이상적인 솔루션이다.35 특히 

  `TopicId`를 사용하는 방식은 LoRaWAN의 제한된 페이로드 크기(수십 바이트 수준) 내에서 전송 효율을 극대화하는 데 결정적인 역할을 한다.

### 5.3 환경 모니터링 및 웨어러블 디바이스

도시의 대기 질, 하천의 수질, 외딴 지역의 산불 감시 등 공공 안전과 환경 보호를 위한 모니터링 시스템 역시 MQTT-SN의 중요한 적용 분야다.

- **사례:** 도시 전역에 설치된 미세먼지 측정기나 강에 띄워진 수질 오염 감시 센서는 배터리와 셀룰러 통신(NB-IoT, LTE-M)을 사용하여 주기적으로 데이터를 중앙 관제 센터로 전송한다.37 또한, 개인의 건강 상태를 모니터링하는 웨어러블 디바이스(심박수 측정기, 활동량 추적기 등)는 Bluetooth Low Energy(BLE)를 통해 사용자의 스마트폰과 통신하고, 스마트폰이 게이트웨이 역할을 하여 데이터를 클라우드 서버로 전송한다.39
- **MQTT-SN의 역할:** 이러한 애플리케이션들은 디바이스가 간헐적으로 네트워크에 연결되고, 매우 적은 양의 데이터를 비주기적으로 전송하는 특징을 가진다. MQTT-SN의 절전 모드와 경량 패킷 구조는 이러한 통신 패턴에 최적화되어 있으며, BLE나 Zigbee와 같은 다양한 저전력 단거리 통신 프로토콜 위에서도 유연하게 동작할 수 있는 장점을 가진다.6

### 5.4 주요 구현체 및 생태계

MQTT-SN의 도입과 확산은 강력한 오픈소스 생태계에 의해 뒷받침되고 있다. 그중 가장 대표적인 것은 이클립스 재단의 **Paho 프로젝트**다. Paho는 Java, C, Python 등 다양한 언어로 MQTT 및 MQTT-SN 클라이언트 라이브러리와 게이트웨이 구현체를 제공한다.10 특히 `paho.mqtt-sn.embedded-c`는 저사양 마이크로컨트롤러에 MQTT-SN 클라이언트를 구현하기 위한 경량 C 라이브러리를 제공하며, C++로 작성된 게이트웨이 소스 코드도 포함하고 있어 개발자들이 MQTT-SN 시스템을 구축하는 데 훌륭한 출발점을 제공한다.43 이 외에도 EMQX, HiveMQ 등 여러 상용 MQTT 브로커들이 MQTT-SN 게이트웨이 기능을 통합하여 제공함으로써, MQTT-SN 디바이스들을 기존 MQTT 인프라에 원활하게 통합할 수 있는 경로를 열어주고 있다.15

## 6. MQTT-SN의 한계, 과제 및 미래 전망

MQTT-SN은 자원 제약적 환경을 위한 강력한 솔루션을 제공하지만, 동시에 몇 가지 내재적인 한계와 해결해야 할 과제를 안고 있다. 이러한 과제를 이해하고 경쟁 기술과의 관계를 파악하며, 표준화 동향을 주시하는 것은 MQTT-SN을 성공적으로 도입하고 미래 기술 변화에 대비하는 데 필수적이다.

### 6.1 보안 문제 및 해결 방안

- **한계:** 현재 널리 사용되는 MQTT-SN v1.2 명세의 가장 큰 약점은 보안 기능의 부재다.15 표준 MQTT가 TLS/SSL을 통한 전송 계층 암호화와 

  `CONNECT` 패킷 내 Username/Password 필드를 통한 클라이언트 인증을 지원하는 반면, MQTT-SN v1.2는 프로토콜 자체에 암호화나 강력한 인증 메커니즘을 정의하고 있지 않다. `CONNECT` 메시지는 `ClientId`만을 포함하며, 이를 통해 클라이언트를 식별할 뿐 인증하지는 않는다.13 이는 특히 신뢰할 수 없는 공용망을 통해 데이터가 전송될 경우, 도청(eavesdropping), 메시지 위변조(tampering), 서비스 거부(DoS) 공격 등 심각한 보안 위협에 노출될 수 있음을 의미한다.

- **해결 방안:** 이러한 보안 공백을 메우기 위해 현재 몇 가지 우회적인 방법이 사용되고 있다.

  1. **전송 계층 보안:** UDP 위에서 동작하는 경우, DTLS(Datagram Transport Layer Security)를 적용하여 MQTT-SN 패킷 전체를 암호화하고 무결성을 보장할 수 있다. 이는 강력한 보안을 제공하지만, DTLS 핸드셰이크 과정의 오버헤드가 발생하며 저사양 디바이스에 구현하기에는 복잡성이 높다는 단점이 있다.13
  2. **애플리케이션 계층 암호화:** MQTT-SN 페이로드(실제 데이터 부분)를 애플리케이션 수준에서 직접 암호화하는 방식이다. 이는 구현이 비교적 유연하지만, 헤더 정보(TopicId 등)는 보호되지 않으며, 디바이스마다 각기 다른 암호화 방식을 사용할 경우 시스템 전체의 파편화와 관리 복잡성을 초래할 수 있다.

### 6.2 경쟁 프로토콜과의 비교: CoAP (Constrained Application Protocol)

WSN 환경을 위한 경량 프로토콜 시장에서 MQTT-SN의 주요 경쟁자는 IETF에 의해 표준화된 CoAP이다. 두 프로토콜은 유사한 목표를 가지지만, 근본적인 통신 모델과 아키텍처에서 큰 차이를 보인다.

- **공통점:** 두 프로토콜 모두 자원이 제한된 디바이스와 저전력 손실 네트워크(LLN)를 위해 설계되었으며, UDP 위에서 동작하여 TCP의 오버헤드를 피한다.44

- **차이점:**

  - **통신 모델:** 가장 큰 차이점이다. MQTT-SN은 브로커를 중심으로 하는 **발행/구독(Publish/Subscribe)** 모델을 따른다. 이는 데이터 생산자와 소비자를 분리하여 비동기적인 이벤트 기반 통신에 최적화되어 있다.46 반면, CoAP은 HTTP와 매우 유사한 

    **요청/응답(Request/Response)** 모델을 사용한다. 클라이언트는 서버의 특정 자원(Resource)에 대해 GET, POST, PUT, DELETE와 같은 메소드를 사용하여 상태를 조회하거나 변경한다.45

  - **아키텍처:** MQTT-SN은 중앙 집중적인 게이트웨이/브로커를 필요로 하는 반면, CoAP은 디바이스 간 직접 통신(peer-to-peer)이 가능한 분산형 아키텍처를 지원한다.46

  - **적합성:** MQTT-SN은 하나의 센서 데이터(이벤트)를 다수의 애플리케이션이나 디바이스에 동시에 전달해야 하는 '일대다(one-to-many)' 또는 '다대다(many-to-many)' 데이터 분배 시나리오에 매우 효과적이다. CoAP은 특정 디바이스의 현재 상태를 조회하거나(예: "전등 상태 알려줘"), 특정 동작을 명령하는(예: "전등 켜") RESTful 방식의 상호작용에 더 적합하다.44

### 6.3 OASIS 표준화 동향: MQTT-SN v2.0의 발전 방향

MQTT-SN의 한계를 극복하고 미래 IoT 환경에 대응하기 위해, OASIS MQTT 기술 위원회(TC)를 중심으로 MQTT-SN v2.0 표준화 작업이 진행 중이다.48 2021년과 2022년에 걸쳐 워킹 드래프트(Working Draft)가 공개되고 논의되는 등 활발한 활동이 이루어졌으나, 아직 최종 표준으로 확정되지는 않았다.48 v2.0의 주요 목표는 다음과 같다.

1. **MQTT v5.0과의 정렬(Alignment):** 2019년에 표준화된 MQTT v5.0은 향상된 오류 보고(Reason Code), 세션 만료, 사용자 속성(User Properties) 등 많은 고급 기능을 도입했다.51 MQTT-SN v2.0은 이러한 MQTT v5.0의 유용한 기능들을 WSN 환경에 맞게 수용하여, 두 프로토콜 간의 기능적 일관성을 높이고 게이트웨이에서의 변환 작업을 더욱 원활하게 만드는 것을 목표로 한다.48
2. **보안 강화(Security Enhancement):** v1.2의 가장 큰 약점이었던 보안 문제를 해결하는 것이 최우선 과제 중 하나다. MQTT v5.0에 도입된 확장 인증(Enhanced Authentication) 메커니즘과 유사하게, `AUTH` 패킷을 도입하여 SASL(Simple Authentication and Security Layer)과 같은 다양한 인증 방식을 지원하는 방안이 논의되고 있다.22 이를 통해 프로토콜 명세 자체에 강력하고 표준화된 보안 기능을 내장하고자 한다.
3. **구현 경험 반영 및 명확화:** 지난 수년간 v1.2를 구현하고 운영하면서 발견된 모호한 부분이나 비효율적인 절차들을 개선한다. 예를 들어, 투명 게이트웨이에서 유언 메시지를 업데이트하는 절차의 어려움, 토픽 별칭(Alias) 고갈 문제에 대한 처리, 짧은 토픽 이름에서의 와일드카드 사용 규칙 명확화 등이 개선 대상으로 논의되고 있다.50

MQTT-SN v2.0이 성공적으로 표준화되면, 강화된 보안과 MQTT 5.0과의 향상된 호환성을 바탕으로 더욱 넓은 범위의 상용 IoT 솔루션에 채택될 잠재력이 크다. 이는 MQTT-SN이 단순한 틈새 프로토콜을 넘어, 차세대 엣지 컴퓨팅 환경의 핵심 통신 규약으로 자리매김하는 중요한 전환점이 될 것이다.

## 7. 결론: 차세대 IoT 통신 프로토콜로서의 MQTT-SN의 위상

본 고찰을 통해 MQTT-SN은 단순히 표준 MQTT 프로토콜의 축소판이나 변종이 아님을 확인했다. MQTT-SN은 MQTT의 핵심 철학인 '발행/구독' 모델을 계승하면서도, TCP/IP라는 전통적인 네트워크의 굴레에서 벗어나 무선 센서 네트워크(WSN)라는 극단적인 자원 제약 환경에 대응하기 위해 근본적으로 재설계된, 고도로 전문화된 프로토콜이다.

게이트웨이 중심의 아키텍처, 긴 토픽 이름을 2바이트 `TopicId`로 대체하는 혁신, 프로토콜 수준에서 명시적으로 지원되는 절전 모드, 그리고 동적인 네트워크 구성을 위한 게이트웨이 탐색 기능은 MQTT-SN을 특징짓는 핵심적인 기술적 성취다. 이러한 기능들은 저전력, 저대역폭, 비-IP 기반 통신이라는 WSN의 본질적인 제약 조건들을 정면으로 해결하며, 수많은 센서 노드가 효율적이고 안정적으로 IoT 생태계에 참여할 수 있는 길을 열어주었다. 이는 MQTT-SN이 엣지 컴퓨팅 시대의 도래와 함께 그 중요성이 더욱 커질 수밖에 없는 이유를 명확히 보여준다.

### 7.1 도입을 위한 기술적 권고 사항

MQTT-SN을 성공적으로 도입하고 활용하고자 하는 시스템 설계자 및 개발자는 다음 사항들을 신중하게 고려해야 한다.

- **적합성 판단:** 프로젝트의 대상 디바이스가 TCP/IP 스택을 구현하기 어렵거나, 배터리 수명이 시스템의 성패를 좌우하는 매우 중요한 요소이거나, LoRaWAN, Bluetooth LE, Zigbee 등 비-IP 기반의 무선 기술을 사용하는 환경이라면, 표준 MQTT 대신 MQTT-SN 도입을 최우선으로 고려해야 한다. 반면, 안정적인 IP 네트워크에 연결되고 전력 공급이 원활한 환경이라면, 검증된 생태계를 가진 표준 MQTT가 더 나은 선택일 수 있다.
- **게이트웨이 전략 수립:** MQTT-SN 아키텍처의 핵심인 게이트웨이는 시스템 전체의 신뢰성과 확장성을 결정한다. 소규모 테스트나 프로토타이핑 단계에서는 구현이 간단한 투명 게이트웨이가 적합할 수 있다. 그러나 수천 개 이상의 노드로 확장될 상용 시스템을 염두에 둔다면, MQTT 브로커의 부하를 줄여주는 집계 게이트웨이 방식을 초기 설계 단계부터 고려하는 것이 바람직하다. 또한, 게이트웨이의 고가용성(High Availability) 및 장애 복구 전략을 반드시 수립하여 단일 장애점(SPOF) 문제를 방지해야 한다.
- **보안 대책 강구:** 현재 널리 사용되는 MQTT-SN v1.2를 기반으로 시스템을 구축할 경우, 프로토콜 자체의 보안 기능 부재를 반드시 인지하고 있어야 한다. 데이터의 기밀성과 무결성이 요구되는 애플리케이션이라면, DTLS를 적용하거나 애플리케이션 수준에서 페이로드를 암호화하는 등 추가적인 보안 계층을 구현하는 것이 필수적이다. 장기적인 관점에서는 MQTT-SN v2.0 표준화 동향을 지속적으로 주시하며, 표준에 포함될 새로운 보안 기능을 채택할 준비를 해야 한다.
- **생태계 활용:** 처음부터 모든 것을 직접 개발하기보다는, Eclipse Paho와 같이 성숙하고 검증된 오픈소스 구현체를 적극 활용하는 것이 현명하다.10 이는 개발 기간을 단축하고, 잠재적인 구현 오류를 줄이며, 광범위한 커뮤니티의 지원을 통해 문제 해결에 도움을 받을 수 있는 가장 효과적인 방법이다.

결론적으로, MQTT-SN은 IoT의 영역을 진정한 '사물'의 끝단까지 확장시키는 데 필수적인 기술이다. 그 한계와 과제에도 불구하고, WSN 환경에 대한 깊은 이해를 바탕으로 설계된 이 프로토콜은 앞으로 다가올 대규모 엣지 컴퓨팅 시대에서 더욱 중요한 역할을 수행하게 될 것이 자명하다.

#### **참고 자료**

1. MQTT - Wikipedia, 8월 4, 2025에 액세스, https://en.wikipedia.org/wiki/MQTT
2. What is MQTT? Definition of IoT Messaging Protocol - AWS, 8월 4, 2025에 액세스, https://aws.amazon.com/what-is/mqtt/
3. MQTT - The Standard for IoT Messaging, 8월 4, 2025에 액세스, https://mqtt.org/
4. MQTT beginner's guide - u-blox, 8월 4, 2025에 액세스, https://www.u-blox.com/en/blogs/insights/mqtt-beginners-guide
5. MQTT for Sensor Networks (MQTT-SN) : IoT Part 39, 8월 4, 2025에 액세스, https://www.engineersgarage.com/mqtt-for-sensor-networks-mqtt-sn-iot-part-39/
6. When MQTT-SN should be used? How is it different from MQTT? - Stack Overflow, 8월 4, 2025에 액세스, https://stackoverflow.com/questions/27560549/when-mqtt-sn-should-be-used-how-is-it-different-from-mqtt
7. Benefits of MQTT SN over MQTT | Bevywise Networks, 8월 4, 2025에 액세스, https://www.bevywise.com/blog/benefits-of-mqtt-sn-over-mqtt/
8. Introduction of Message Queue Telemetry Transport Protocol (MQTT) - GeeksforGeeks, 8월 4, 2025에 액세스, https://www.geeksforgeeks.org/computer-networks/introduction-of-message-queue-telemetry-transport-protocol-mqtt/
9. Introduction to MQTT-SN (MQTT for Sensor Networks) - Steve's Internet Guide, 8월 4, 2025에 액세스, http://www.steves-internet-guide.com/mqtt-sn/
10. MQTT-SN - Zephyr Project Documentation, 8월 4, 2025에 액세스, https://docs.zephyrproject.org/latest/connectivity/networking/api/mqtt_sn.html
11. MQTT-S – A Publish/Subscribe Protocol For Wireless Sensor Networks - UCSB CS, 8월 4, 2025에 액세스, https://sites.cs.ucsb.edu/~rich/class/cs293b-cloud/papers/mqtt-s.pdf
12. Security Analysis of the MQTT-SN Protocol for the Internet of Things - MDPI, 8월 4, 2025에 액세스, https://www.mdpi.com/2076-3417/12/21/10991
13. Exploring MQTT-SN: A Comprehensive Guide | EMQ - EMQX, 8월 4, 2025에 액세스, https://www.emqx.com/en/blog/connecting-mqtt-sn-devices-using-emqx
14. MQTT-SN(Sensor Network): A new protocol in the making? | by Rithic Hariharan - Medium, 8월 4, 2025에 액세스, https://gr8rithic.medium.com/mqtt-sn-sensor-network-a-new-protocol-in-the-making-23b17c1aa930
15. Exploring MQTT-SN: A Comprehensive Guide | by EMQ Technologies - Medium, 8월 4, 2025에 액세스, https://emqx.medium.com/exploring-mqtt-sn-a-comprehensive-guide-2a280cf2bb6c
16. MQTT-SN (MQTT For Sensor Networks) - High Voltages, 8월 4, 2025에 액세스, https://highvoltages.co/iot-internet-of-things/mqtt/mqtt-sn/
17. MQTT in Cellular IoT: Deep Dive into Protocol Variants, Advantages, and Real-World Use, 8월 4, 2025에 액세스, https://tago.io/blog/mqtt-in-cellular-iot-variants-advantages-and-usecases
18. MQTT - SN | Deep Notes - Deepak's Wiki, 8월 4, 2025에 액세스, https://deepaksood619.github.io/networking/mqtt/mqtt-sn/
19. MQTT Basics — SensiML Documentation, 8월 4, 2025에 액세스, https://sensiml.com/documentation/mqtt-specification/mqtt-basics.html
20. MQTT-SN for Cellular IoT and Micromobility - HiveMQ, 8월 4, 2025에 액세스, https://www.hivemq.com/blog/mqtt-sn-for-cellular-iot-micromobility/
21. Developing MQTT-SN Clients Guide - Bevywise, 8월 4, 2025에 액세스, https://www.bevywise.com/developing-mqtt-sn-clients/
22. mqtt-sn-implementation-guide.adoc - GitHub, 8월 4, 2025에 액세스, https://github.com/oasis-open/mqtt-sn-sample-resources/blob/main/docs/mqtt-sn-implementation-guide.adoc
23. MQTT-SN: The Smart Choice for IIoT - HiveMQ, 8월 4, 2025에 액세스, https://www.hivemq.com/blog/mqtt-sn-smart-choice-for-iiot/
24. MQTT Packets: A Comprehensive Guide - HiveMQ, 8월 4, 2025에 액세스, https://www.hivemq.com/blog/mqtt-packets-comprehensive-guide/
25. MQTT Client, MQTT Broker, and MQTT Server Connection Establishment Explained – MQTT Essentials: Part 3 - HiveMQ, 8월 4, 2025에 액세스, https://www.hivemq.com/blog/mqtt-essentials-part-3-client-broker-connection-establishment/
26. Differences between MQTT and MQTT-SN - Internet of Things for Architects [Book], 8월 4, 2025에 액세스, https://www.oreilly.com/library/view/internet-of-things/9781788470599/881de410-96e1-4771-b506-cd7450faadc3.xhtml
27. MQTT fundamentals - Navixy, 8월 4, 2025에 액세스, https://docs.navixy.com/expert-center/mqtt-fundamentals
28. Question regarding the MQTT-SN sleep mode - Google Groups, 8월 4, 2025에 액세스, https://groups.google.com/g/mqtt/c/aOnuqj41Msk
29. (PDF) IoT Based Predictive Maintenance in Industrial Machinery - ResearchGate, 8월 4, 2025에 액세스, https://www.researchgate.net/publication/393468255_IoT_Based_Predictive_Maintenance_in_Industrial_Machinery
30. TIP4.0: Industrial Internet of Things Platform for Predictive Maintenance - PMC, 8월 4, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC8309552/
31. Smart Agriculture using Internet of Things and MQTT Protocol - ResearchGate, 8월 4, 2025에 액세스, https://www.researchgate.net/publication/336437004_Smart_Agriculture_using_Internet_of_Things_and_MQTT_Protocol
32. IoT in Agriculture: Key Use Cases and Real-life Examples - Itransition, 8월 4, 2025에 액세스, https://www.itransition.com/iot/agriculture
33. Development of LoRa Multipoint Network Integrated with MQTT-SN Protocol for Microclimate Data Logging in UB Forest, 8월 4, 2025에 액세스, https://iiict.uob.edu.bh/IJCDS/papers/1570999725.pdf
34. 8 Real-World MQTT Use Cases | InfluxData, 8월 4, 2025에 액세스, https://www.influxdata.com/blog/mqtt-use-cases/
35. A LoRaWAN Network Architecture with MQTT2MULTICAST - MDPI, 8월 4, 2025에 액세스, https://www.mdpi.com/2079-9292/11/6/872
36. MQTT — Deep Dive | IoT For All, 8월 4, 2025에 액세스, https://www.iotforall.com/mqtt-a-deep-dive
37. A smart sensor and effective IoT data communication solution - u-blox, 8월 4, 2025에 액세스, https://www.u-blox.com/en/casestudies/iot-in-pollution-monitoring
38. (PDF) AIOT based Real Time Environment Monitoring and Tracking System for Soldiers using SN-MQTT Protocol - ResearchGate, 8월 4, 2025에 액세스, https://www.researchgate.net/publication/352388261_AIOT_based_Real_Time_Environment_Monitoring_and_Tracking_System_for_Soldiers_using_SN-MQTT_Protocol
39. Open-Source MQTT-Based End-to-End IoT System for Smart City Scenarios - MDPI, 8월 4, 2025에 액세스, https://www.mdpi.com/1999-5903/14/2/57
40. An MQTT-Based Context-Aware Wearable Assessment Platform for Smart Watches | Request PDF - ResearchGate, 8월 4, 2025에 액세스, https://www.researchgate.net/publication/318973662_An_MQTT-Based_Context-Aware_Wearable_Assessment_Platform_for_Smart_Watches
41. Using MQTT-SN over BLE with the BBC micro:bit - Benjamin Cabé, 8월 4, 2025에 액세스, https://blog.benjamin-cabe.com/2017/01/16/using-mqtt-sn-over-ble-with-the-bbc-microbit
42. Paho - The Eclipse Foundation, 8월 4, 2025에 액세스, https://eclipse.dev/paho/
43. eclipse-paho/paho.mqtt-sn.embedded-c - GitHub, 8월 4, 2025에 액세스, https://github.com/eclipse-paho/paho.mqtt-sn.embedded-c
44. MQTT and CoAP, IoT Protocols | The Eclipse Foundation, 8월 4, 2025에 액세스, https://www.eclipse.org/community/eclipse_newsletter/2014/february/article2.php
45. CoAP vs MQTT: Which is the definitive protocol for resource-constrained IoT devices in 2025?, 8월 4, 2025에 액세스, https://www.cloud.studio/coap-vs-mqtt/
46. What is the difference between MQTT SN and CoAP? - API Security 101 - Quora, 8월 4, 2025에 액세스, https://apisecurity101.quora.com/What-is-the-difference-between-MQTT-SN-and-CoAP
47. MQTT vs CoAP: Comparing Protocols for IoT Connectivity | EMQ - EMQX, 8월 4, 2025에 액세스, https://www.emqx.com/en/blog/mqtt-vs-coap
48. MQTT-SN Standardization Progress - Model Based Testing - Ian Craggs, 8월 4, 2025에 액세스, https://modelbasedtesting.co.uk/2021/04/17/mqtt-sn-standardization-progress/
49. OASIS Message Queuing Telemetry Transport (MQTT) TC, 8월 4, 2025에 액세스, https://www.oasis-open.org/committees/mqtt/
50. Notes on MQTT-SN v.2.0 specification WD 17, 8월 4, 2025에 액세스, [https://higherlogicdownload.s3.amazonaws.com/OASIS/cb688407-cc7a-41a4-bcb4-42a6f226281b_file.pdf?X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEC8aCXVzLWVhc3QtMSJHMEUCIF6MhSVIk98vop2lQ63eN%2BWT55aRQaUrbMco%2FrwHvXrxAiEAl%2F30wkp7w8t2zIVA%2BeHYQMd5uGl4GE%2B583CTQk5OelwqrwUISBAAGgwzODAzMzczNDA3MDYiDEmso4dAqadOIMTh9CqMBQisadgLfzjVgsMjcMKC6snTeuPzEmOXa1%2Bg44ypY0OZQ9LhC%2BvyZE5kkEBY7PWs2DW6IzVGNdkbr%2FRgi5pVCrT4m7WkDyRv9%2BRNtw96d7cWYB2vcR9vUoziCVHDKRzOaGQwUJsY9uGIzEtgaJ7dnPVJnRuCGdPO494eVHJFToRuHLHvQUhXZQ%2ByCYqQX%2BZn6nj58cgUVHXduak%2FTVE8Vk1jC4jl%2FbkXyUS22VPEigrVpfgSA%2B0rPJC50F9GYCCmUy%2BE0Wr%2F50U2URRJsB18PbZGc9eqGJkW0cgSgOQSyX6kHLpkYRrPQPfqJ9LdvzGjK8DhHma%2F430wcmYqGo%2BHAWtImmwRWfPSUDRVkQKHpTOFRnW4yxyJGfhheECQQo%2F2tBGa7b4gIL%2FztCmaxxcstTEGUuNKDX6Piz5cwef13QxjkpWKVJflknqctvQlOU%2BRD%2FTddiv6%2BMfCK%2F84CENdUE7rvbDzeDocrWNFyqb%2FW%2BiJjIudSw62C8ZmOyeETzEt102jm4QnW%2BzKwn8eLGhLYtIKSDhYsZXegTgcJDqBWts2aE3IL%2BiSUCcnu9HvJ7cjz%2BdFAzxDPNebOXFXZGiENCtCmyTBFyif%2BpZ0ynlD6yAjyU95vcCrV6T6O%2B4xZFH%2B9UPZ90C7flY03io36kXY4mXibexsvs9iaxgzmi5RQrIFcagefs%2B1f2m92YIFUPCl24qoH%2FHy%2BmJlT2wakzXAWf%2B1117p0H%2FJIF2tgbZ2lQVjyE16r0S6%2Fy%2BcQH%2BdSK%2BccFyImVhrNd7WBWc0HiVPQcFJFC6ib3l4RRmXSy71dCRwX9bgJPgUwYvIdeA9YKp4xK3AV9PQ5FSUragp8KQdyjFcec%2F1O9JXprLo4sIwxM%2FZwwY6sQFO3LFt13r41qZEP3YP9uW8tvgB8QMWXYwMfodGrbGkWGp8V5Vlgy%2FoWKEQhR59xNO5V6Qtc%2FfLDOPeSOLtyM0p3vUlgsMiL05%2BJ2p5dKxI0x5dI%2F3MiWoepaqAYrLuyHTkzkhMJF8%2FZ7%2F50FknLIflWF4S%2FomSQ0gnp2jLHzlcJxgXZ0XvnS9ud1Kk5CaBHtxsCdyvvhCyqx%2FvM%2B0xjqhp0T2oOzOmbbDu%2F3WjqKlX5hE%3D&response-content-disposition=attachment%3B%20filename%2A%3DUTF-8%27%27mqtt-sn-v2.0.pdf&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIAVRDO7IERBSUKJGOI%2F20250715%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20250715T152510Z&X-Amz-SignedHeaders=host&X-Amz-Signature=9fd1d0abaae2d3c4dd64da8dbe8b63e4e8a24c8886b22646437f1fc7af495253](https://higherlogicdownload.s3.amazonaws.com/OASIS/cb688407-cc7a-41a4-bcb4-42a6f226281b_file.pdf?X-Amz-Expires=3600&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEC8aCXVzLWVhc3QtMSJHMEUCIF6MhSVIk98vop2lQ63eN%2BWT55aRQaUrbMco/rwHvXrxAiEAl/30wkp7w8t2zIVA%2BeHYQMd5uGl4GE%2B583CTQk5OelwqrwUISBAAGgwzODAzMzczNDA3MDYiDEmso4dAqadOIMTh9CqMBQisadgLfzjVgsMjcMKC6snTeuPzEmOXa1%2Bg44ypY0OZQ9LhC%2BvyZE5kkEBY7PWs2DW6IzVGNdkbr/Rgi5pVCrT4m7WkDyRv9%2BRNtw96d7cWYB2vcR9vUoziCVHDKRzOaGQwUJsY9uGIzEtgaJ7dnPVJnRuCGdPO494eVHJFToRuHLHvQUhXZQ%2ByCYqQX%2BZn6nj58cgUVHXduak/TVE8Vk1jC4jl/bkXyUS22VPEigrVpfgSA%2B0rPJC50F9GYCCmUy%2BE0Wr/50U2URRJsB18PbZGc9eqGJkW0cgSgOQSyX6kHLpkYRrPQPfqJ9LdvzGjK8DhHma/430wcmYqGo%2BHAWtImmwRWfPSUDRVkQKHpTOFRnW4yxyJGfhheECQQo/2tBGa7b4gIL/ztCmaxxcstTEGUuNKDX6Piz5cwef13QxjkpWKVJflknqctvQlOU%2BRD/Tddiv6%2BMfCK/84CENdUE7rvbDzeDocrWNFyqb/W%2BiJjIudSw62C8ZmOyeETzEt102jm4QnW%2BzKwn8eLGhLYtIKSDhYsZXegTgcJDqBWts2aE3IL%2BiSUCcnu9HvJ7cjz%2BdFAzxDPNebOXFXZGiENCtCmyTBFyif%2BpZ0ynlD6yAjyU95vcCrV6T6O%2B4xZFH%2B9UPZ90C7flY03io36kXY4mXibexsvs9iaxgzmi5RQrIFcagefs%2B1f2m92YIFUPCl24qoH/Hy%2BmJlT2wakzXAWf%2B1117p0H/JIF2tgbZ2lQVjyE16r0S6/y%2BcQH%2BdSK%2BccFyImVhrNd7WBWc0HiVPQcFJFC6ib3l4RRmXSy71dCRwX9bgJPgUwYvIdeA9YKp4xK3AV9PQ5FSUragp8KQdyjFcec/1O9JXprLo4sIwxM/ZwwY6sQFO3LFt13r41qZEP3YP9uW8tvgB8QMWXYwMfodGrbGkWGp8V5Vlgy/oWKEQhR59xNO5V6Qtc/fLDOPeSOLtyM0p3vUlgsMiL05%2BJ2p5dKxI0x5dI/3MiWoepaqAYrLuyHTkzkhMJF8/Z7/50FknLIflWF4S/omSQ0gnp2jLHzlcJxgXZ0XvnS9ud1Kk5CaBHtxsCdyvvhCyqx/vM%2B0xjqhp0T2oOzOmbbDu/3WjqKlX5hE%3D&response-content-disposition=attachment;+filename*%3DUTF-8''mqtt-sn-v2.0.pdf&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIAVRDO7IERBSUKJGOI/20250715/us-east-1/s3/aws4_request&X-Amz-Date=20250715T152510Z&X-Amz-SignedHeaders=host&X-Amz-Signature=9fd1d0abaae2d3c4dd64da8dbe8b63e4e8a24c8886b22646437f1fc7af495253)
51. mqtt-v5.0.html - MQTT Version 5.0 - OASIS Open, 8월 4, 2025에 액세스, https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html


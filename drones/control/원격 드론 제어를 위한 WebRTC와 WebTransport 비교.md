# 원격 드론 제어를 위한 WebRTC와 WebTransport 비교

## 1. 서론: 원격 드론 제어의 통신 패러다임

### 0.1 핵심 과제 정의

원격 드론 제어 시스템의 성공은 상충하는 두 가지 핵심 요구사항을 얼마나 효과적으로 해결하느냐에 달려있다. 첫째는 조종 명령(Control Signals)을 위한 극도로 낮은 지연 시간(Ultra-low Latency)이며, 둘째는 조종사에게 상황 인식을 제공하는 고품질 실시간 영상(Video Feed) 스트리밍이다. 이 두 가지 데이터 스트림을 하나의 통신 채널 위에서 동시에, 그리고 신뢰성 있게 전송하는 것이 기술적 난제의 핵심이다.

조종사의 의사결정 과정, 즉 관찰-판단-결정-행동(Observe, Orient, Decide, Act, OODA) 루프는 통신 지연 시간에 직접적인 영향을 받는다.1 군용 무인 항공기(UAV) 연구에 따르면, 100ms의 지연 시간만으로도 조종 성능이 눈에 띄게 저하되기 시작하며, 250-300ms에 이르면 비행 제어가 거의 불가능한 수준으로 어려워진다.2 이는 1인칭 시점(FPV) 레이싱이나 정밀 항공 촬영과 같이 밀리초 단위의 빠른 반응이 요구되는 분야에서 지연 시간이 왜 그토록 중요한지를 명확히 보여준다.3 따라서, 성공적인 원격 제어 시스템은 단순히 '빠른' 연결을 넘어, 예측 가능하고 안정적인 지연 시간을 보장해야만 한다.

### 0.2 기존 프로토콜의 한계와 웹 기술의 부상

현재 드론 통신 분야는 크게 두 가지 접근 방식으로 나뉜다.

- **독점 프로토콜 (Proprietary Protocols):** DJI의 OcuSync와 같은 시스템은 하드웨어와 소프트웨어가 긴밀하게 통합되어 있다. 이를 통해 DJI O3 시스템은 1080p 라이브 뷰를 약 120ms의 놀라운 지연 시간으로 전송하는 등 뛰어난 성능을 보여준다.4 하지만 이러한 성능은 특정 제조사의 폐쇄적인 생태계에 종속된다는 대가를 치른다. 이는 서드파티 시스템과의 통합, 기능 커스터마이징, 그리고 솔루션의 전반적인 확장성을 심각하게 저해하는 요인이 된다.6
- **오픈소스 프로토콜 (Open-Source Protocols):** MAVLink는 경량 메시징과 저대역폭 환경에서의 신뢰성에 초점을 맞춘 대표적인 오픈소스 프로토콜이다.7 패킷당 오버헤드가 8바이트(MAVLink 1) 또는 14바이트(MAVLink 2)에 불과하여 64kbps와 같은 매우 제한된 대역폭에서도 안정적인 통신을 지원한다.8 그러나 MAVLink는 주로 원격 측정(Telemetry) 데이터와 제어 명령 전송에 국한되며, 고품질 비디오 스트리밍을 위한 포괄적인 솔루션을 내장하고 있지 않다. 또한, 프로토콜 자체는 기본적으로 암호화되지 않아 중간자 공격 등 보안 위협에 취약하다는 단점이 있다.10

이러한 배경 속에서 웹 기술, 특히 WebRTC와 WebTransport가 차세대 드론 제어 플랫폼의 핵심 기술 후보로 급부상했다. 브라우저 기반 제어 시스템은 별도의 클라이언트 소프트웨어 설치가 필요 없어 플랫폼 독립성을 확보할 수 있으며, 배포가 용이하고, HTML, CSS, JavaScript 등 풍부한 웹 생태계를 활용하여 정교한 사용자 인터페이스(UI)를 쉽게 개발할 수 있다는 막대한 이점을 제공한다.11

### 0.3 보고서의 목표와 범위

이 보고서는 WebRTC와 WebTransport라는 두 웹 기술을 원격 드론 제어라는 특수한 목적에 맞춰 심층적으로 비교 분석하는 것을 목표로 한다. 단순히 기술 사양을 나열하는 것을 넘어, 각 기술의 근본적인 아키텍처와 그 기반이 되는 프로토콜 스택이 드론의 비행 안정성, 제어 반응성, 그리고 운영 확장성에 구체적으로 어떤 영향을 미치는지 파헤칠 것이다. 최종 목표는 특정 개발 시나리오와 비즈니스 요구사항에 맞는 최적의 기술을 선택할 수 있도록 명확하고 데이터에 기반한 기술적 가이드를 제공하는 것이다.

## 1.  기술 심층 분석: WebRTC

### 1.1  아키텍처: P2P와 시그널링의 역할

WebRTC(Web Real-Time Communication)는 본질적으로 두 개의 브라우저, 즉 피어(Peer) 간의 직접적인 미디어 및 데이터 통신을 가능하게 하는 P2P(Peer-to-Peer) 프레임워크다.14 이 프레임워크의 핵심은 `RTCPeerConnection` API로, P2P 연결의 설정, 관리, 종료 등 모든 생명주기를 담당한다.16

그러나 WebRTC는 연결 설정을 위해 필수적인 메타데이터 교환 과정, 즉 '시그널링(Signaling)'에 대한 표준 프로토콜을 의도적으로 정의하지 않았다.16 개발자는 연결할 피어들의 네트워크 정보(ICE 후보)와 미디어 정보(SDP)를 교환하기 위해 WebSocket, HTTP 요청, 또는 다른 어떤 메커니즘이든 자유롭게 선택하여 직접 구현해야 한다. 이러한 설계는 개발자에게 높은 유연성을 제공하지만, 동시에 WebRTC 구현의 복잡성을 가중시키는 첫 번째 요인이 된다.14

### 1.2  NAT Traversal: ICE, STUN, TURN의 현실

현대의 인터넷 환경은 거의 모든 사용자가 NAT(Network Address Translation) 장비 뒤에 위치해 있기 때문에, 두 피어가 서로의 사설 IP 주소를 직접 알 수 없어 P2P 연결을 맺는 데 큰 어려움이 있다.14 WebRTC는 이 문제를 해결하기 위해 ICE(Interactive Connectivity Establishment)라는 정교한 프레임워크를 사용한다.

- **ICE (Interactive Connectivity Establishment):** ICE는 연결 가능한 모든 경로를 '후보(Candidate)' 형태로 수집하고, 이 후보들을 서로 교환하여 연결 가능성을 테스트한 뒤, 가장 효율적인 경로를 선택하는 메커니즘이다.14 ICE가 수집하는 후보의 종류는 다음과 같다.
  1. **Host Candidate:** 장치의 로컬 네트워크 주소.
  2. **Server-Reflexive Candidate:** STUN 서버를 통해 발견된 공인 IP 주소.
  3. **Relay Candidate:** TURN 서버를 통해 할당된 중계 주소.
- **STUN (Session Traversal Utilities for NAT):** STUN 서버의 역할은 간단하다. 피어가 STUN 서버에 요청을 보내면, 서버는 요청을 보낸 피어의 공인 IP 주소와 포트 정보를 응답으로 돌려준다.18 피어는 이 정보를 시그널링 채널을 통해 상대방에게 전달하고, 이를 통해 NAT 장비의 포트 매핑을 뚫고 직접적인 P2P 연결을 시도한다.14
- **TURN (Traversal Using Relays around NAT):** STUN을 통해서도 직접 연결이 불가능한 경우가 있다. 예를 들어, 일부 기업 방화벽이나 '대칭형 NAT(Symmetric NAT)' 환경에서는 외부에서의 직접적인 연결 시도를 차단한다.18 이 경우, TURN 서버가 최후의 보루 역할을 한다. TURN 서버는 두 피어 간의 모든 미디어와 데이터 트래픽을 수신하여 다른 피어에게 전달하는 '중계기' 역할을 수행한다.14 이 방식은 더 이상 순수한 P2P 통신이 아니며, 서버의 대역폭과 CPU 자원을 소모하므로 상당한 운영 비용을 발생시킨다.

이처럼 WebRTC의 'P2P'라는 이상은 실제로는 '조건부 P2P'에 가깝다. 이 조건을 충족시키기 위해 STUN/TURN 서버 인프라를 구축하고 운영해야 하는 현실적인 대가가 따르며, 이는 드론 서비스의 비즈니스 모델과 운영 비용(OPEX) 설계에 직접적인 영향을 미친다.

### 1.3  미디어 전송: 통합 미디어 엔진과 SRTP

WebRTC의 가장 큰 강점 중 하나는 미디어 처리를 위한 포괄적인 솔루션을 내장하고 있다는 점이다. `getUserMedia` API를 통해 사용자의 카메라와 마이크에 접근하는 것부터 시작하여, 캡처된 미디어를 실시간으로 인코딩(VP8, VP9, H.264, AV1 등 지원)하고, 네트워크를 통해 전송하며, 수신 측에서 디코딩하여 화면에 렌더링하는 전체 미디어 파이프라인이 브라우저에 통합되어 있다.16

보안 또한 WebRTC의 핵심 설계 원칙이다. 모든 미디어 스트림은 **SRTP(Secure Real-time Transport Protocol)**를 통해 의무적으로 암호화되어 전송된다. 이 암호화에 사용되는 키는 연결 설정 과정에서 DTLS(Datagram Transport Layer Security) 핸드셰이크를 통해 안전하게 교환된다.11

더 나아가, WebRTC 미디어 엔진은 불안정한 네트워크 환경에 대응하기 위한 다양한 기능을 내장하고 있다. 네트워크 대역폭 변화를 감지하여 비디오의 해상도나 프레임률을 동적으로 조절하는 **적응형 비트레이트(Adaptive Bitrate)** 제어, 패킷 손실이 발생했을 때 화면 깨짐을 최소화하는 **패킷 손실 은닉(Packet-loss concealment)**, 그리고 네트워크 지연 변동(지터)을 흡수하여 부드러운 재생을 돕는 **동적 지터 버퍼링(Dynamic jitter buffering)** 등의 기능이 별도의 구현 없이도 기본적으로 제공되어 일정 수준 이상의 통신 품질을 보장한다.11

### 1.4  데이터 채널: SCTP의 힘

WebRTC는 미디어뿐만 아니라 임의의 데이터를 P2P로 교환할 수 있는 `RTCDataChannel`이라는 강력한 API를 제공한다.16 이 데이터 채널은 내부적으로 **SCTP(Stream Control Transmission Protocol)**를 DTLS 위에서 실행하는 방식으로 구현된다.11 SCTP는 TCP의 신뢰성과 UDP의 메시지 기반 전송 특징을 결합한 진보된 전송 프로토콜로, 드론 제어에 특히 유용한 기능들을 제공한다.

- **다중 스트림 (Multi-streaming):** 하나의 `RTCPeerConnection` 내에 여러 개의 독립적인 데이터 스트림을 생성할 수 있다. 이는 매우 중요한 특징인데, 예를 들어 하나의 스트림에서 중요한 설정 값(Reliable)을 전송하는 동안 다른 스트림에서는 실시간 원격 측정 데이터(Unreliable)를 보낼 수 있다. 한 스트림에서 패킷 손실로 인한 Head-of-Line (HOL) 블로킹이 발생하더라도 다른 스트림의 데이터 전송에는 영향을 주지 않는다.23
- **전송 모드 (Delivery Modes):** `RTCDataChannel`의 가장 강력한 기능은 데이터의 특성에 따라 전송 방식을 메시지별로 지정할 수 있다는 점이다.
  - **신뢰성 있고 순서 보장 (Reliable & Ordered):** TCP와 동일하게 동작하며, 모든 데이터가 순서대로 빠짐없이 전달됨을 보장한다. 드론의 펌웨어 업데이트나 미션 계획 전송과 같이 데이터 무결성이 중요한 경우에 사용된다.
  - **부분 신뢰성 및 순서 보장/미보장 (Partially Reliable & Ordered/Unordered):** `maxRetransmits` (최대 재전송 횟수) 또는 `maxPacketLifeTime` (최대 패킷 생존 시간) 옵션을 통해 신뢰성 수준을 제어할 수 있다. 이것은 드론 제어의 핵심 요구사항과 완벽하게 부합한다. 드론 제어 데이터는 균일하지 않다. '비상 정지' 명령은 100% 전달되어야 하지만, 초당 수십 번씩 전송되는 드론의 자세(attitude) 정보는 최신 정보가 이전 정보를 완전히 대체하므로, 100ms 이상 지연된 과거의 자세 정보는 재전송할 가치가 없다. `maxPacketLifeTime`을 100ms로 설정하면, 100ms 내에 전달되지 못한 데이터는 자동으로 폐기되고 최신 데이터 전송에 자원을 할당하게 된다. 이 기능은 RFC 3758에 정의된 PR-SCTP(Partially Reliable SCTP) 개념을 WebRTC가 API 형태로 제공하는 것으로, 다른 어떤 범용 웹 프로토콜도 쉽게 제공하지 못하는 WebRTC만의 독보적인 장점이다.24

### 2.5. 장단점 요약

#### 1.4.1 장점

- **성숙한 생태계와 높은 호환성:** 10년 이상 발전해 온 기술로 생태계가 매우 성숙하며, 주요 데스크톱 및 모바일 브라우저에서 96% 이상의 압도적인 지원율을 보인다.27
- **잠재적인 초저지연:** 성공적인 P2P 직접 연결 시, 서버를 경유하지 않으므로 이론적으로 가장 낮은 지연 시간을 달성할 수 있으며 서버 운영 비용을 절감할 수 있다.15
- **올인원 솔루션:** 미디어 엔진과 데이터 채널이 하나의 프레임워크 안에 통합되어 있어, 복잡한 실시간 통신 기능을 비교적 일관된 방식으로 구현할 수 있다.16
- **강력한 NAT Traversal:** ICE, STUN, TURN을 통해 복잡한 실제 네트워크 환경에서도 높은 연결 성공률을 제공한다.11

#### 1.4.2 단점

- **극심한 복잡성:** ICE, DTLS, SCTP, SRTP 등 수많은 프로토콜이 얽혀있는 거대한 스택은 내부 동작을 이해하고 제어하기 어렵게 만들며, 문제 발생 시 디버깅이 매우 까다롭다.17
- **부가 인프라 요구:** 시그널링 서버는 필수적으로 직접 구현해야 하며, 안정적인 서비스를 위해서는 STUN/TURN 서버 인프라를 구축하고 운영해야 한다. 특히 트래픽을 중계하는 TURN 서버는 상당한 비용 부담을 유발한다.16
- **확장성 문제:** P2P 모델은 소수의 참여자에게는 효율적이지만, 다수의 드론을 하나의 관제 센터에서 모니터링하는 시나리오에서는 연결 복잡성이 기하급수적으로 증가하여 부적합하다. 이 경우 반드시 SFU(Selective Forwarding Unit)나 MCU(Multipoint Control Unit) 같은 서버 기반 아키텍처를 도입해야 한다.15
- **모놀리식 아키텍처:** 전체 시스템이 하나의 거대한 덩어리(Monolith)처럼 설계되어 있어, 특정 코덱을 사용하거나 전송 전략을 미세 조정하는 등 핵심 컴포넌트를 교체하거나 커스터마이징하기가 매우 어렵다.33

## 2.  기술 심층 분석: WebTransport

### 2.1  아키텍처: 클라이언트-서버와 QUIC 혁명

WebTransport는 WebSockets의 한계를 극복하기 위해 등장한 현대적인 통신 프로토콜이다. 이 기술의 핵심 철학은 WebRTC와 근본적으로 다르다. WebTransport는 P2P가 아닌, 클라이언트와 **HTTP/3 서버** 간의 양방향 통신을 위해 명확하게 설계된 클라이언트-서버(Client-Server) 모델을 따른다.29

이러한 C/S 모델 집중은 WebRTC가 겪는 복잡성의 근원인 시그널링 과정과 P2P 연결 설정을 위한 지난한 과정을 완전히 제거하는 효과를 가져왔다.15 WebTransport의 모든 것은 

**QUIC(Quick UDP Internet Connections)** 프로토콜 위에 구축된다. QUIC은 TCP의 고질적인 문제들을 해결하기 위해 구글이 주도하여 개발하고 IETF에서 표준화한, UDP 기반의 차세대 전송 프로토콜이다.12 WebTransport는 이 QUIC의 강력한 기능들을 웹 개발자들이 쉽게 사용할 수 있도록 API 형태로 노출시킨 것이다.

이러한 설계는 'P2P의 부재'가 단순한 단점이 아니라, '서버 중심 아키텍처'를 강제하는 명확한 설계 철학임을 보여준다. WebRTC는 P2P를 '시도'하다가 현실의 네트워크 제약(NAT) 때문에 결국 TURN 서버를 통한 C/S 형태로 회귀하는 경우가 많다.14 반면, WebTransport는 이러한 모호함을 제거하고 처음부터 안정적이고 예측 가능한 C/S 통신을 전제로 한다.17 이는 개발자가 '실패할 수도 있는 P2P'라는 불확실성을 고려할 필요 없이, 항상 예측 가능하게 동작하는 서버 중계 아키텍처에만 집중할 수 있게 해준다. 따라서 이는 개인 취미용 드론보다는 상업용/산업용 드론 관제 플랫폼과 같이 안정성과 확장성이 중요한 시스템에 더 적합한 모델이다.

### 2.2  QUIC의 핵심 이점

WebTransport의 우수성은 전적으로 그 기반인 QUIC에서 비롯된다.

- **HOL(Head-of-Line) 블로킹 해결:** TCP 기반의 HTTP/2는 단일 연결 위에서 여러 요청을 동시에 처리(Multiplexing)할 수 있지만, 만약 한 요청에 해당하는 패킷 하나가 유실되면 그 뒤의 모든 요청 처리가 지연되는 HOL 블로킹 문제가 발생한다. QUIC은 UDP를 기반으로 하면서 자체적으로 스트림(Stream) 개념을 구현한다. 단일 QUIC 연결 내에 다수의 독립적인 스트림을 생성할 수 있으며, 한 스트림에서 패킷이 유실되더라도 다른 스트림의 데이터 전송을 막지 않는다.34 드론 제어 시나리오에서 고화질 비디오 스트림의 패킷 하나가 유실되었다고 해서, 극도로 중요한 제어 신호 스트림의 전송이 지연되는 최악의 상황을 원천적으로 방지할 수 있다.
- **빠른 연결 설정 (0-RTT):** 기존의 TCP 연결 위에 TLS 암호화를 설정하려면 최소 2~3번의 왕복 시간(RTT)이 소요된다. QUIC은 전송 계층 핸드셰이크와 암호화(TLS 1.3) 핸드셰이크를 하나로 통합하여, 첫 연결 시 1-RTT만에 연결을 수립할 수 있다. 더욱 중요한 것은, 한번 연결했던 서버와 다시 연결할 때는 이전에 교환했던 정보를 활용하여 핸드셰이크 과정 없이 즉시 데이터를 전송하는 **0-RTT 연결 재개**가 가능하다는 점이다.37 이는 드론이 비행 중 일시적으로 통신이 두절되었다가 재연결될 때, 거의 즉시 제어권을 회복할 수 있음을 의미하며, 이는 비행 안정성에 결정적인 요소다.
- **내장된 암호화 (Built-in Encryption):** QUIC은 TLS 1.3을 프로토콜의 핵심적인 부분으로 통합하여, 일부 헤더를 제외한 모든 패킷을 기본적으로 암호화한다. WebRTC처럼 별도의 DTLS 핸드셰이크와 같은 복잡한 과정을 거칠 필요가 없어, 보안 설정이 더 단순하고 빠르다.37

### 2.3  스트림과 데이터그램: 유연한 데이터 전송

WebTransport API는 QUIC이 제공하는 두 가지 주요 전송 모드를 개발자에게 직접 노출시켜, 데이터의 특성에 맞는 최적의 전송 방식을 선택할 수 있게 한다.29

- **스트림 API (Streams API):** 신뢰성 있고(Reliable) 순서가 보장되는(Ordered) 데이터 전송을 제공한다. TCP 연결과 유사하게 동작하지만, QUIC 스트림은 훨씬 가볍게 생성하고 소멸시킬 수 있어 리소스 효율성이 높다. 서버나 클라이언트가 일방적으로 데이터를 보내는 단방향(Unidirectional) 스트림과 양쪽에서 데이터를 주고받는 양방향(Bidirectional) 스트림을 모두 지원한다. 이 모드는 드론의 설정 값 전송, 비행 로그 파일 전송, 또는 반드시 전달되어야 하는 중요한 제어 명령(예: '미션 시작')에 적합하다.34
- **데이터그램 API (Datagrams API):** 신뢰성과 순서를 보장하지 않는 최선 노력(Best-effort) 전송 방식을 제공한다. UDP 메시지와 매우 유사하지만, QUIC의 일부이므로 암호화 및 혼잡 제어 기능이 기본적으로 포함된다는 중요한 차이가 있다. 이 모드는 지연 시간에 극도로 민감하여 약간의 데이터 손실을 감수하더라도 가장 빠르게 전달되어야 하는 데이터에 이상적이다. 초당 수십 회씩 전송되는 실시간 드론 조종 신호나 최신 상태 정보가 이전 정보를 대체하는 원격 측정 데이터 전송에 완벽하게 부합한다.29

### 2.4  결정적 차이: 연결 마이그레이션

QUIC의 가장 혁신적인 기능 중 하나는 **연결 마이그레이션(Connection Migration)**이다. TCP는 '출발지 IP:포트, 목적지 IP:포트'의 4-튜플(4-tuple)로 연결을 식별한다. 따라서 클라이언트의 IP 주소가 바뀌면(예: Wi-Fi에서 LTE로 네트워크 전환) TCP 연결은 끊어지고, 처음부터 다시 연결을 수립해야 한다.

반면, QUIC은 IP 주소와 무관한 고유한 **연결 ID(Connection ID)**를 사용하여 연결을 식별한다.34 이 덕분에 클라이언트(드론)가 비행 중에 Wi-Fi 네트워크의 범위를 벗어나 5G 셀룰러 네트워크로 전환하더라도, 연결 ID가 유지되므로 통신 세션이 끊기지 않고 그대로 유지된다.37 WebRTC는 이 상황에서 ICE Restart라는 무겁고 느린 과정을 거쳐야 하지만, QUIC은 프로토콜 수준에서 이를 매끄럽고 신속하게 처리한다.

이 기능은 단순한 '연결 유지'를 넘어, 장거리 비행(BVLOS) 드론 운영의 핵심 기반 기술로 볼 수 있다. 드론이 지상의 여러 기지국(Wi-Fi, 5G)을 거치거나, 심지어 지상 통신이 불가능한 지역에서 위성 통신으로 전환해야 하는 복잡한 시나리오를 상상해 보면, 연결 마이그레이션의 가치는 명확해진다. 제어 세션이 끊기지 않는다는 것은, 이전에는 불가능했던 동적인 네트워크 환경에서의 안정적인 장거리 드론 운영을 가능하게 하는 **전략적 기술(Strategic Enabler)**이다.

### 3.5. 장단점 요약

#### 2.4.1 장점

- **우수한 전송 성능:** QUIC을 기반으로 하여 HOL 블로킹 방지, 빠른 연결 설정(0-RTT), 효율적인 혼잡 제어 등 TCP 대비 월등한 성능을 제공한다.15
- **단순한 아키텍처:** 명확한 클라이언트-서버 모델을 채택하여 WebRTC의 복잡한 시그널링 및 P2P 연결 설정 과정이 필요 없어 API가 더 단순하고 직관적이다.17
- **압도적인 이동성 지원:** 연결 마이그레이션 기능은 네트워크 환경이 계속 변하는 모바일 장치, 특히 드론에 최적의 안정성을 제공한다.37
- **유연한 전송 방식:** 신뢰성이 필요한 데이터는 스트림으로, 저지연이 중요한 데이터는 데이터그램으로 전송하는 등 데이터 특성에 맞는 유연한 선택이 가능하다.34

#### 2.4.2 단점

- **P2P 및 NAT Traversal 부재:** 서버는 반드시 공인 IP 주소를 가져야 하며, 클라이언트 간 직접 통신(P2P)은 지원하지 않는다. 모든 통신은 서버를 경유해야 한다.17
- **상대적으로 새로운 기술:** WebRTC에 비해 생태계가 아직 작고, 검증된 운영 사례나 참고 자료가 부족하다.45
- **제한적인 브라우저 지원:** 주요 브라우저에서 지원이 확대되고 있지만, WebRTC만큼 보편적이지는 않다. 특히 Safari의 지원 여부 및 안정성은 중요한 고려사항이다. 2024년 기준, 전 세계 지원율은 약 79% 수준이다.21
- **미디어 엔진 부재:** WebRTC와 달리 미디어 처리를 위한 엔진이 내장되어 있지 않다. 비디오/오디오 스트리밍을 구현하려면 WebCodecs API와 같은 다른 기술을 조합하여 지터 버퍼, 동기화 등을 직접 구현해야 하는 상당한 복잡성이 따른다.30

## 3.  핵심 요구사항 분석: 원격 드론 제어의 기술적 제약

원격 드론 제어 시스템을 위한 통신 프로토콜을 평가하기 위해서는, 먼저 드론이라는 특수한 애플리케이션이 요구하는 구체적인 기술적 제약 조건을 명확히 정의해야 한다. 이는 크게 제어 신호, 비디오 피드, 그리고 네트워크 이동성 세 가지 측면으로 나눌 수 있다.

### 3.1  제어 신호

제어 신호는 조종사의 의도를 드론에 전달하는 가장 중요한 데이터 스트림이다.

- **지연 시간 (Latency):** 조종사의 스틱 입력이 드론의 모터 반응으로 나타나기까지의 총 지연 시간, 즉 'Glass-to-Glass' 또는 'End-to-End' 지연 시간은 비행 안정성의 핵심 지표다. 군용 UAV 연구에서는 250-300ms의 지연이 조종을 수용 불가능하게 만든다고 명시하고 있으며 2, 취미용 FPV 드론 커뮤니티에서는 일반적으로 10-100ms 범위의 지연을 논하며, 이 값이 낮을수록 정밀한 제어가 가능하다고 본다.3 따라서 상업용 원격 제어 시스템의 제어 신호 지연 시간 목표는 

  **안정적으로 100ms 이하, 이상적으로는 50ms 이하**로 설정해야 한다.

- **신뢰성 및 대역폭 (Reliability & Bandwidth):** 제어 신호는 비디오 피드에 비해 요구 대역폭이 매우 낮다. 예를 들어, MAVLink 프로토콜은 패킷 크기가 20바이트 미만으로, 64kbps와 같은 저대역폭 링크에서도 안정적인 통신을 목표로 설계되었다.7 그러나 대역폭 요구사항이 낮은 대신 신뢰성 요구사항은 매우 높다. '비상 착륙'과 같은 명령은 반드시 전달되어야 한다. 반면, 초당 50회씩 전송되는 자세 정보는 최신 값이 이전 값을 대체하므로 일부 손실이 허용된다. 따라서 제어 신호 채널은 완전 신뢰성 모드와 '부분 신뢰성' 모드를 모두 지원하여, 시간 초과된 낡은 명령은 버리고 최신 명령을 우선시하는 전략을 구사할 수 있어야 한다.

### 3.2  비디오 피드

비디오 피드는 조종사에게 원격지의 상황을 인지하게 하는 '눈'의 역할을 한다.

- **지연 시간 (Latency):** 조종사가 드론의 카메라 시점에서 주변 환경을 보고 반응하기 때문에 비디오 지연 역시 제어 지연만큼 중요하다. DJI의 OcuSync 3.0 시스템은 약 120ms의 비디오 지연을 명시하고 있으며 4, FPV 비행에 특화된 DJI FPV 시스템은 고품질 모드에서 40ms, 저지연 모드에서는 28ms까지 지연 시간을 낮춘다.6 이는 웹 기술 기반 솔루션이 달성해야 할 현실적인 목표치를 제시한다. 성공적인 사용자 경험을 위해서는 

  **200ms 이하, 이상적으로는 150ms 이하**의 Glass-to-Glass 비디오 지연을 목표로 해야 한다.

- **대역폭 및 품질 (Bandwidth & Quality):** 고품질 비디오는 상당한 대역폭을 요구한다. DJI O3 시스템은 1080p/30fps 라이브 뷰를 위해 최대 18Mbps의 비디오 비트레이트를 사용하며 4, 전문 방송용 DJI Transmission 시스템은 최대 40Mbps까지 지원한다.49 드론이 비행하는 동안 무선 네트워크 환경은 계속 변하므로, 고정된 비트레이트로는 안정적인 스트리밍이 불가능하다. 따라서 네트워크 상황에 따라 해상도, 프레임률, 비트레이트를 동적으로 조절하는 **적응형 비트레이트 스트리밍(Adaptive Bitrate Streaming)** 기능이 필수적이다. WebRTC는 이 기능을 내장하고 있어 11 개발 부담을 덜어주지만, WebTransport를 사용할 경우 WebCodecs와 조합하여 개발자가 직접 이 로직을 구현해야 한다.33

### 3.3  네트워크 이동성

드론은 본질적으로 이동하는 장치(Mobile device)이므로, 네트워크 환경이 끊임없이 변한다. 특히 도심 지역에서는 수많은 Wi-Fi AP와 셀룰러(4G/5G) 기지국 사이를 오가며 네트워크 핸드오버가 빈번하게 발생할 수 있다. 핸드오버 과정에서 제어 및 비디오 스트림이 수 초간 중단되는 것은 임무 실패나 기체 추락으로 이어질 수 있으므로 절대 용납될 수 없다. 따라서 통신 프로토콜은 이러한 네트워크 변경을 감지하고 연결을 끊김 없이(Seamless) 유지하는 능력을 갖추어야 한다. 이 요구사항은 QUIC의 연결 마이그레이션 기능이 절대적으로 유리한 지점이다.37

결론적으로, 드론 제어는 '두 개의 다른 속도를 가진 심장'을 하나의 몸에 이식하는 것과 같다. 제어 신호는 작고, 빠르고, 극도로 신뢰적이거나 시간 민감해야 하는 '단거리 육상선수'와 같다. 반면, 비디오 피드는 크고, 지연에 어느 정도 관대하며, 네트워크 변화에 유연하게 적응해야 하는 '장거리 마라토너'와 같다. WebRTC와 WebTransport를 평가하는 기준은 이 두 개의 이질적인 데이터 스트림을 얼마나 효율적이고 조화롭게 처리할 수 있느냐에 달려있다.

## 4.  심층 비교 분석: 드론 제어를 위한 WebRTC vs. WebTransport

앞서 분석한 드론 제어의 핵심 요구사항을 기준으로 WebRTC와 WebTransport를 직접 비교하여, 각 기술이 특정 시나리오에서 어떤 장단점을 갖는지 명확히 한다.

### 4.1  아키텍처 대결: P2P vs. 클라이언트-서버

- **1:1 제어 (단일 조종사 - 단일 드론):**
  - **WebRTC:** NAT Traversal에 성공하여 P2P 직접 연결이 수립되면, 서버 중계 없이 드론과 조종사가 통신하므로 이론적으로 가장 낮은 지연 시간을 달성할 수 있다. 이 경우 서버 비용은 시그널링 및 STUN 서버 운영에 한정되어 저렴하다.15 하지만 연결의 성공 여부와 안정성은 전적으로 양단의 네트워크 환경에 의존하는 단점이 있다.
  - **WebTransport:** 모든 트래픽이 서버를 경유하므로 P2P 연결에 비해 최소 1 RTT(Round-Trip Time)의 물리적 지연이 추가된다. 그러나 서버를 사용자 및 드론과 가까운 곳에 배치하는 엣지 컴퓨팅(Edge Computing) 전략을 통해 이 지연을 최소화할 수 있다. 서버 경유 방식은 네트워크 환경 변화에 덜 민감하여 연결이 매우 안정적이고 예측 가능하다는 큰 장점을 가진다.15
- **N:1 제어 (관제 센터 - 다수 드론):**
  - **WebRTC:** P2P(Mesh) 아키텍처는 참여자(드론) 수가 늘어날수록 각 피어가 맺어야 하는 연결의 수가 기하급수적으로 증가(n(n−1)/2)하여 대규모 관제에는 부적합하다. 따라서 이 시나리오에서는 반드시 SFU(Selective Forwarding Unit)나 MCU(Multipoint Control Unit)와 같은 서버 기반 아키텍처로 전환해야 한다. 이 순간 WebRTC는 사실상 클라이언트-서버 모델처럼 동작하게 되며, P2P의 이론적 장점은 사라지고 복잡성만 남게 된다.15
  - **WebTransport:** 본질적으로 클라이언트-서버 모델이므로, 다수의 드론을 관리하는 관제 센터 시나리오에 매우 자연스럽게 확장된다. 서버 측에서 모든 드론의 연결 상태, 데이터 흐름, 인증 및 권한 부여를 중앙에서 효율적으로 관리할 수 있다.15

### 4.2  성능 벤치마크: 지연 시간과 패킷 손실 대응

- **연결 설정:** WebTransport는 QUIC의 빠른 핸드셰이크(1-RTT 또는 0-RTT) 덕분에 TCP+TLS 기반의 연결(예: WebSocket 시그널링)보다 초기 연결 설정 속도가 더 빠르다.29 이는 통신 두절 후 빠른 재연결에 유리하다.
- **패킷 손실 대응:** 불안정한 무선 환경은 드론 통신의 숙명이다. 벤치마크 연구에 따르면, 15%의 패킷 손실이 있는 네트워크 환경에서 WebTransport는 WebRTC 데이터 채널보다 더 많은 메시지를 더 안정적으로 전달하는 경향을 보였다.51 이는 WebTransport의 기반인 QUIC이 스트림 간 HOL 블로킹을 효과적으로 방지하는 반면, WebRTC의 SCTP 구현은 특정 조건에서 송신 측의 성능 저하를 보일 수 있음을 시사한다. 따라서 예측 불가능한 패킷 손실에 대한 강건함 측면에서는 WebTransport가 더 우수할 가능성이 높다.

### 4.3  구현 복잡성: 올인원 vs. DIY 툴킷

이 비교는 단순한 P2P 대 C/S 구도를 넘어, '통합(Integrated)' 시스템과 '모듈형(Modular)' 시스템 간의 철학적 차이를 보여준다.

- **WebRTC (통합 시스템):** WebRTC는 미디어 캡처, 인코딩/디코딩, 전송, 렌더링, 에코 캔슬링, 지터 버퍼 등 실시간 통신에 필요한 거의 모든 것을 포함하는 **올인원 솔루션**이다.21 이 덕분에 개발자는 복잡한 미디어 파이프라인을 직접 구축하지 않고도 상대적으로 쉽게 비디오 스트리밍 기능을 구현할 수 있다. 하지만 이 통합된 구조는 거대한 블랙박스처럼 느껴질 수 있으며, 내부 동작을 변경하거나 최적화하기 어렵다는 단점이 있다. 또한, 시그널링, STUN/TURN 서버 구축 등 복잡한 인프라 설정이 요구된다.17
- **WebTransport + WebCodecs (모듈형 툴킷):** 이 조합은 개발자에게 저수준의 구성 요소들을 제공하는 **DIY 툴킷**과 같다.33 WebTransport는 데이터 전송 계층을, WebCodecs는 미디어 인코딩/디코딩 기능을 각각 담당한다. 개발자는 이들을 조합하여 자신만의 실시간 통신 스택을 구축해야 한다. 이는 특정 코덱(예: HEVC)을 선택하거나, 커스텀 전송 전략을 구현하는 등 최고의 유연성과 제어권을 제공한다. API 또한 더 현대적이고 개발자 친화적이라는 평가를 받는다.29 그러나 이는 개발자가 지터 버퍼, 패킷 손실 복구(FEC/NACK), 오디오/비디오 동기화 등 매우 복잡한 미디어 처리 로직을 직접 구현해야 함을 의미하며, 이는 높은 수준의 전문성을 요구한다.21

### 4.4  연결 이동성: ICE Restart vs. Connection Migration

움직이는 드론에게 이 항목은 가장 결정적인 차이를 만들어낸다.

- **WebRTC:** 드론의 네트워크가 변경되면(예: Wi-Fi -->> 5G), 연결을 유지하기 위해 `ICE Restart` 프로세스를 시작해야 한다. 이는 새로운 네트워크 환경에서 연결 가능한 후보들을 처음부터 다시 수집하고 테스트하는 과정으로, 연결이 일시적으로 완전히 끊겼다가 재설정되는 것을 의미한다. 이 과정은 수 초의 지연을 유발할 수 있으며, 그동안 드론은 통제 불능 상태에 빠질 수 있다.
- **WebTransport:** QUIC의 **Connection Migration**은 이 문제를 프로토콜 수준에서 해결한다. 연결이 IP 주소가 아닌 고유한 연결 ID로 식별되므로, 드론의 IP 주소가 바뀌더라도 서버는 동일한 연결로 인식하고 통신을 중단 없이 이어간다.37 이 과정은 매우 매끄럽고 신속하게 이루어진다.

**평가:** 드론의 안전과 임무의 신뢰성을 고려할 때, 네트워크 핸드오버 상황에서의 끊김 없는 연결 유지는 타협할 수 없는 요구사항이다. 이 측면에서는 **WebTransport가 명백하고 압도적인 우위를 가진다.**

### 4.5  보안 모델: DTLS/SRTP vs. QUIC

- **WebRTC:** 데이터 채널은 DTLS로, 미디어 채널은 SRTP로 암호화된다. 두 프로토콜 모두 오랜 기간 검증된 강력한 보안 표준이다.11
- **WebTransport:** QUIC 자체가 TLS 1.3을 프로토콜에 내장하여, 핸드셰이크 과정부터 모든 데이터 전송까지 기본적으로 암호화한다. 더 단순하고 통합된 보안 모델을 제공한다.15

**평가:** 두 기술 모두 강력한 종단 간 암호화를 제공하므로 보안 수준 자체에 큰 차이가 있다고 보기는 어렵다. 다만, WebTransport의 모델이 더 간결하고 현대적이며, 구현상의 복잡성이 적다.

### 4.6 Table 1: WebRTC vs. WebTransport - 드론 제어를 위한 기능 비교

| 기능                 | WebRTC                                                       | WebTransport                                                 | 드론 제어에 미치는 영향                                      |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **아키텍처**         | P2P (C/S 폴백 포함)                                          | 클라이언트-서버                                              | **WebRTC:** 간단한 1:1, 저비용 시나리오에 이상적. **WebTransport:** 확장 가능한 플릿 관리 및 안정적인 기업 서비스에 더 적합. |
| **제어 지연 시간**   | 이상적인 P2P 환경에서 잠재적으로 더 낮음. TURN 중계 시 더 높음. | 예측 가능한 서버 홉 지연. 엣지 서버로 최적화 가능.           | 미션 크리티컬 제어에서는 WebTransport의 예측 가능성이 WebRTC의 잠재적(보장되지 않는) P2P 속도보다 더 가치 있을 수 있음. |
| **비디오 지연 시간** | 통합 미디어 엔진과 SRTP 덕분에 낮음.                         | DIY 미디어 파이프라인(WebCodecs)으로 인해 잠재적으로 더 높음. 전문가 수준의 구현 필요. | WebRTC는 별도 노력 없이 저지연 비디오를 구현하는 더 쉬운 경로. WebTransport는 이를 따라잡기 위해 상당한 개발 노력이 필요. |
| **제어 신뢰성**      | SCTP를 통해 우수 (신뢰성, 부분 신뢰성 모드).                 | 스트림 API(신뢰성) 및 데이터그램 API(비신뢰성)를 통해 우수.  | 둘 다 우수함. WebRTC의 내장된 부분 신뢰성 기능은 단일 채널에서 혼합된 데이터 유형을 처리하는 데 약간의 이점이 있음. |
| **네트워크 이동성**  | 취약함. 느리고 중단이 발생하는 ICE Restart 필요.             | 매우 우수함. QUIC 연결 ID를 통한 끊김 없는 연결 마이그레이션. | **명백한 승자: WebTransport.** 이는 모든 모바일 드론에 있어 핵심적인 기능임. |
| **NAT Traversal**    | 매우 우수함. ICE/STUN/TURN 내장.                             | **없음.** 서버는 반드시 공인 IP를 가져야 함.                 | **명백한 승자: WebRTC.** 중앙 서버 없이 복잡한 사설망을 통과해야 하는 애플리케이션에 적합. 하지만 드론의 경우 서버/GCS가 공인망에 있을 수 있으므로 덜 중요함. |
| **구현 복잡성**      | 복잡한 인프라 (시그널링, TURN). 단순한 미디어 처리.          | 단순한 인프라 (HTTP/3 서버). 복잡한 미디어 처리 (WebCodecs). | 트레이드오프: WebRTC는 인프라 구축이 무겁고, WebTransport는 미디어 처리 코드 작성이 무거움. 팀의 전문성에 따라 선택이 달라짐. |
| **생태계/지원**      | 성숙함, 보편적인 브라우저 지원 (>96%).                       | 신생 기술, 지원은 양호하나 Safari가 위험 요소 (~79%).        | WebRTC가 현재로서는 더 안전하고 호환성이 높은 선택. WebTransport는 특히 iOS/macOS 타겟 시 플랫폼 리스크를 안고 있음. |

## 5.  아키텍처 설계 및 권장사항

WebRTC와 WebTransport의 기술적 특성을 바탕으로, 원격 드론 제어 시스템을 위한 세 가지 가능한 아키텍처를 설계하고 각 시나리오의 적합성을 평가한다.

### 5.1  시나리오 1: 순수 WebRTC 아키텍처

- **설계:** 조종사(브라우저)와 드론(임베디드 클라이언트)이 1:1로 `RTCPeerConnection`을 맺는다. 중앙의 시그널링 서버가 두 피어의 연결 설정을 중개하고, STUN 서버가 공인 IP 주소 발견을 돕는다. P2P 연결에 실패할 경우, TURN 서버가 모든 제어 및 비디오 트래픽을 중계한다.
- **적합한 용도:** 개인 취미 프로젝트, 학술 연구, 또는 가시권 내 비행(VLOS) 환경에서 개발 및 운영 비용을 최소화하고자 할 때 가장 적합하다. 이미 WebRTC를 활용한 RC카나 드론 제어 프로젝트 사례가 다수 존재한다.13
- **장점:** P2P 연결에 성공하면 서버 자원 소모를 최소화하면서 가장 낮은 지연 시간을 기대할 수 있다. 기술이 성숙하여 구현에 참고할 수 있는 자료가 풍부하다.
- **위험 및 한계:** 네트워크 이동성에 매우 취약하여 드론이 Wi-Fi와 셀룰러 간을 이동할 때 연결이 끊길 위험이 크다. P2P 연결 실패율이 높아지면 TURN 서버 의존도가 커져 비용이 증가하고 지연 시간이 늘어난다. N:1 관제와 같은 대규모 시나리오로의 확장이 근본적으로 어렵다.31

### 5.2  시나리오 2: 순수 WebTransport 아키텍처

- **설계:** 모든 드론과 조종사는 지리적으로 최적화된 위치에 배포된 WebTransport/HTTP3 서버에 연결된다. 서버는 조종사로부터 받은 제어 신호(데이터그램)를 해당 드론으로 중계하고, 드론으로부터 받은 비디오 스트림(스트림 API)을 조종사에게 전달한다. 미디어 인코딩/디코딩은 드론과 조종사 클라이언트에서 WebCodecs를 사용하여 처리한다.
- **적합한 용도:** 상업용 드론 서비스, 다수의 드론을 동시에 관리하는 플릿(Fleet) 관제 시스템, 비가시권(BVLOS) 장거리 비행, 그리고 안정성과 예측 가능성이 최우선인 미션 크리티컬 애플리케이션에 적합하다.
- **장점:** 연결 마이그레이션 기능을 통해 압도적인 네트워크 안정성을 확보할 수 있다. 모든 통신이 서버를 경유하므로 중앙 집중식 로깅, 인증, 데이터 분석, 제어권 관리 등이 용이하여 확장성과 보안에 유리하다.15
- **위험 및 한계:** 모든 트래픽이 서버를 경유하므로 서버 구축 및 운영 비용, 그리고 고가용성 인프라 관리가 중요하다. 비디오 파이프라인(지터 버퍼, 동기화 등)을 WebCodecs를 이용해 직접 구현해야 하는 높은 기술적 장벽이 존재한다.48 또한, Safari 브라우저의 지원이 아직 완전하지 않아 iOS/macOS 기반 조종 스테이션을 고려할 경우 플랫폼 리스크가 있다.46

### 5.3  시나리오 3: 하이브리드 아키텍처

- **설계:** 두 기술의 장점을 결합하려는 시도. 예를 들어, 지연 시간에 민감하고 안정성이 중요한 제어 신호는 WebTransport를 통해 서버로 안정적으로 보내고, 대역폭을 많이 차지하는 비디오 스트림은 WebRTC를 통해 P2P로 직접 전송하는 모델을 구상할 수 있다.
- **평가:** 이론적으로는 두 기술의 장점(안정적인 제어 + 저지연 P2P 비디오)을 모두 취할 수 있을 것처럼 보이지만, **실제로는 두 기술의 단점만을 결합한 최악의 선택이 될 가능성이 매우 높다.**
  - **구현 및 관리의 지옥:** 두 개의 완전히 다른 연결 생명주기(WebRTC PeerConnection, WebTransport Session)와 보안 컨텍스트(DTLS/SRTP, QUIC/TLS1.3)를 동시에 관리해야 한다.
  - **동기화 문제:** WebTransport로 전달된 제어 신호와 WebRTC로 전달된 비디오/오디오 프레임 간의 타임스탬프를 정확히 동기화하여 'Glass-to-Glass' 지연 시간을 일관되게 측정하고 관리하는 것은 극도로 어렵다.22
  - **복잡성의 폭발:** 문제 발생 시 디버깅해야 할 대상이 두 배로 늘어난다. 하나의 문제를 해결하기 위해 두 개의 완전히 다른 프로토콜 스택과 인프라를 동시에 파헤쳐야 한다.32
- **결론:** 이 아키텍처는 얻는 이득에 비해 복잡성으로 인한 비용이 훨씬 크므로, 특별하고 압도적인 이유가 없는 한 **적극적으로 피해야 한다.**

### 5.4  최종 권장사항

기술 선택은 프로젝트의 목표와 우선순위에 따라 명확하게 내려져야 한다.

- **"신뢰성, 확장성, 이동성이 최우선인 상업용/산업용 솔루션을 구축하는가?" -> WebTransport를 선택하라.**
  - 초기 개발 비용과 WebCodecs 구현이라는 기술적 허들은 높지만, 연결 마이그레이션이 제공하는 압도적인 안정성과 서버 중심 아키텍처의 확장성은 장기적인 비즈니스 가치에 필수적이다. 이는 단순한 기술 선택을 넘어, 안정적인 서비스를 제공하기 위한 전략적 투자다.
- **"빠른 프로토타이핑, 비용 최소화, 1:1 제어가 주된 목적인가?" -> WebRTC를 선택하라.**
  - WebRTC의 올인원 솔루션은 빠르게 동작하는 프로토타입을 만드는 데 가장 효과적이다. 네트워크 이동성과 P2P 연결 실패 가능성이라는 명백한 한계를 인지하고, 주로 VLOS 환경이나 통제된 네트워크 내에서 사용하는 것을 전제로 해야 한다. 추후 확장성이 필요할 경우 SFU 아키텍처를 도입하여 일부 한계를 극복할 수 있다.

## 6.  부록: 서버 구현 고려사항

원격 드론 제어 시스템, 특히 WebTransport 기반 아키텍처에서는 서버의 역할이 절대적이다. 서버는 수많은 드론으로부터의 동시 연결을 처리하며, 저지연으로 제어 및 미디어 데이터를 중계해야 한다. 따라서 서버 구현에 사용되는 프로그래밍 언어와 라이브러리 선택은 시스템 전체 성능에 큰 영향을 미친다.

- **언어 선택 (Go vs. Rust):**

  - **Go:** `net/http`와 같은 강력한 표준 라이브러리를 내장하고 있으며, 고루틴(Goroutine)을 통해 동시성 프로그래밍을 매우 쉽게 할 수 있어 개발 생산성이 높다.55 이는 빠른 프로토타이핑과 시장 출시(Time-to-Market)에 유리하다. 

    `pion/webrtc`나 `adriancable/webtransport-go`와 같은 성숙한 실시간 통신 라이브러리 생태계를 갖추고 있다.

  - **Rust:** 가비지 컬렉터(GC)가 없어 GC로 인한 예측 불가능한 지연 시간 스파이크(latency spike)가 발생하지 않는다. 소유권(Ownership)과 빌림(Borrowing) 개념을 통해 컴파일 타임에 메모리 안전성을 보장하므로, 시스템 레벨에서 최고의 성능과 안정성을 제공한다.56 Google의 내부 연구에서도 Rust로 작성된 서비스가 Go와 동등한 개발 속도를 보이면서도 C++보다 적은 메모리를 사용하고 버그가 적었다는 결과는 시사하는 바가 크다.56

    `wtransport` 58나 

    `h3` 59와 같은 고성능 라이브러리들이 활발히 개발되고 있다.

- **권장사항:**

  - **빠른 시장 출시와 개발 용이성이 중요하다면 Go**가 현실적인 선택이다. 풍부한 웹 생태계와 쉬운 동시성 모델은 개발팀의 부담을 줄여준다.
  - **지연 시간에 극도로 민감하고, 장기적으로 최고의 성능과 안정성을 추구하며, 서버 운영 비용을 최소화해야 한다면 Rust**에 투자하는 것이 현명하다. 초기 학습 곡선은 가파르지만, 그 대가로 얻는 성능과 안정성은 미션 크리티컬 시스템에 매우 큰 가치를 제공한다.

## 7.  결론: 목적에 맞는 도구 선택

WebRTC와 WebTransport는 '어느 것이 더 우월한가'의 문제가 아니라, '어떤 목적에 더 적합한가'의 문제다. 두 기술은 서로 다른 설계 철학을 바탕으로 각기 다른 강점과 약점을 가지고 있다.

- **WebRTC**는 지난 10년간 웹 실시간 통신의 표준으로 자리 잡은, 성숙하고 보편적인 **'P2P 통신용 스위스 아미 나이프'**와 같다. 하나의 도구 안에 미디어, 데이터, P2P 연결 등 수많은 기능이 담겨있지만, 때로는 특정 작업을 위해 사용하기에는 너무 무겁고 복잡하게 느껴질 수 있다.
- **WebTransport**는 QUIC이라는 강력한 엔진 위에 구축된, 빠르고 효율적인 **'클라이언트-서버 통신용 레이저 커터'**다. 정교하고 강력하게 특정 작업을 수행하지만, 비디오 처리와 같은 부가 기능은 사용자가 직접 만들어 붙여야 한다.

원격 드론 제어의 미래는 결국 **'안정성'**과 **'이동성'**이라는 두 가지 키워드에 달려있다. 드론은 끊임없이 움직이며, 그 과정에서 발생하는 네트워크 변화에도 불구하고 제어 연결은 단 한 순간도 끊어져서는 안 된다. 이 두 가지 핵심 가치를 기준으로 판단할 때, 장기적인 관점에서는 WebTransport와 그 기반인 QUIC이 제시하는 패러다임이 더 유망해 보인다. 연결 마이그레이션이라는 '킬러 기능'은 움직이는 드론에게는 타협할 수 없는 가치를 제공한다.

물론, P2P WebTransport와 같은 미래 기술이 표준화되고 브라우저에 탑재된다면 이 구도는 다시 한번 바뀔 수 있다.30 하지만 현재 시점에서, 안정적인 상업용 드론 서비스를 구축하고자 하는 개발팀에게는 서버 중심의 예측 가능하고 강건한 제어가 가능한 WebTransport가 더 합리적인 선택지에 가깝다. 최종적인 기술 선택은 결국 프로젝트의 구체적인 목표, 가용 예산, 그리고 가장 중요하게는 개발팀의 기술적 역량과 전문성에 따라 신중하게 이루어져야 한다.

#### **참고 자료**

1. Remotely Piloted Aircraft Control Latency during Air-to-Air Combat, accessed July 29, 2025, https://www.airuniversity.af.edu/Portals/10/ASOR/Journals/Volume-1_Number-4/Thirtyacre.pdf
2. UAVs and Control Delays - DTIC, accessed July 29, 2025, https://apps.dtic.mil/sti/tr/pdf/ADA454251.pdf
3. FPV Drone Latency Explained: What It Is and How to Reduce It ..., accessed July 29, 2025, https://oscarliang.com/fpv-drone-latency/
4. Support for DJI Mini 3 Pro - DJI, accessed July 29, 2025, https://www.dji.com/support/product/mini-3-pro
5. DJI Transmission Systems – Wi-Fi, OcuSync & Lightbridge - heliguy™, accessed July 29, 2025, https://www.heliguy.com/blogs/posts/dji-transmission-systems-wi-fi-ocusync-lightbridge/
6. DJI OcuSync 3.0 (Explained For Beginners) - Droneblog, accessed July 29, 2025, https://www.droneblog.com/dji-ocusync-3-0/
7. What is MAVLink (Micro Air Vehicle Link)? - Fly Eye, accessed July 29, 2025, https://www.flyeye.io/drone-acronym-mavlink/
8. MAVLink Developer Guide | MAVLink Guide, accessed July 29, 2025, https://mavlink.io/en/
9. SiK Radio - Advanced Configuration - Blimp documentation - ArduPilot, accessed July 29, 2025, https://ardupilot.org/blimp/docs/common-3dr-radio-advanced-configuration-and-technical-information.html
10. A Novel Cipher for Enhancing MAVLink Security: Design, Security Analysis, and Performance Evaluation Using a Drone Testbed - arXiv, accessed July 29, 2025, https://arxiv.org/html/2504.20626v1
11. What is WebRTC Protocol and How does it Work? - CONTUS Tech, accessed July 29, 2025, https://www.contus.com/blog/webrtc-protocol/
12. WebTransport Explained: Low-Latency Communication over HTTP/3 - GoCodeo, accessed July 29, 2025, https://www.gocodeo.com/post/webtransport-explained-low-latency-communication-over-http-3
13. UAV-Based and WebRTC-Based Open Universal Framework to Monitor Urban and Industrial Areas - MDPI, accessed July 29, 2025, https://www.mdpi.com/1424-8220/21/12/4061
14. A Study of WebRTC Security, accessed July 29, 2025, https://webrtc-security.github.io/
15. WebRTC vs WebTransport: Comparison Guide - VideoSDK, accessed July 29, 2025, https://www.videosdk.live/developer-hub/webtransport/webrtc-vs-webtransport
16. WebRTC vs WebSockets: What Are the Differences? - GetStream.io, accessed July 29, 2025, https://getstream.io/blog/webrtc-websockets/
17. WebTransport API - Hacker News, accessed July 29, 2025, https://news.ycombinator.com/item?id=23666364
18. Introduction to WebRTC protocols - Web APIs | MDN, accessed July 29, 2025, https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Protocols
19. WebRTC Stun vs Turn Servers - GetStream.io, accessed July 29, 2025, https://getstream.io/resources/projects/webrtc/advanced/stun-turn/
20. WebRTC Latency: Comparing Low-Latency Streaming Protocols (Update) - - nanocosmos, accessed July 29, 2025, https://www.nanocosmos.de/blog/webrtc-latency/
21. Building a Web-based Audio and Video Call Engine with WebAssembly - Tencent RTC, accessed July 29, 2025, https://trtc.io/blog/details/call-engine
22. WebRTC vs gRPC: Which Communication Protocol Is Right for Your Project? - VideoSDK, accessed July 29, 2025, https://www.videosdk.live/developer-hub/webrtc/webrtc-vs-grpc-comparison
23. RFC 3286: An Introduction to the Stream Control Transmission Protocol (SCTP), accessed July 29, 2025, https://www.rfc-editor.org/rfc/rfc3286.html
24. Information on RFC 3758 - » RFC Editor, accessed July 29, 2025, https://www.rfc-editor.org/info/rfc3758
25. RFC 3758 - Stream Control Transmission Protocol (SCTP) Partial ..., accessed July 29, 2025, https://datatracker.ietf.org/doc/html/rfc3758
26. SCTP packet structure - Wikipedia, accessed July 29, 2025, https://en.wikipedia.org/wiki/SCTP_packet_structure
27. WebRTC Peer-to-peer connections | Can I use... Support tables for HTML5, CSS3, etc - CanIUse, accessed July 29, 2025, https://caniuse.com/rtcpeerconnection
28. "webrtc" | Can I use... Support tables for HTML5, CSS3, etc - CanIUse, accessed July 29, 2025, https://caniuse.com/?search=webrtc
29. Using WebTransport | Capabilities | Chrome for Developers, accessed July 29, 2025, https://developer.chrome.com/docs/capabilities/web-apis/webtransport
30. WebRTC is not the future of low latency live streaming... at least not outside o... | Hacker News, accessed July 29, 2025, https://news.ycombinator.com/item?id=31643140
31. Limitations of WebRTC - Dyte, accessed July 29, 2025, https://dyte.io/blog/webrtc-limitations/
32. WebRTC Architectures: Advantages & Limitations | by Hermes | Agora.io - Medium, accessed July 29, 2025, https://medium.com/agora-io/webrtc-architectures-advantages-limitations-7016b666e1ae
33. WebRTC unbundling: the beginning of the end for WebRTC? - BlogGeek.me, accessed July 29, 2025, https://bloggeek.me/webrtc-unbundling/
34. WebTransport API - Web APIs | MDN, accessed July 29, 2025, https://developer.mozilla.org/en-US/docs/Web/API/WebTransport_API
35. TURN servers running over webtransport / Issue #2651 / w3c/webrtc ..., accessed July 29, 2025, https://github.com/w3c/webrtc-pc/issues/2651
36. RFC 9114: HTTP/3, accessed July 29, 2025, https://www.rfc-editor.org/rfc/rfc9114.html
37. QUIC: The Secure Communication Protocol Shaping the Internet's ..., accessed July 29, 2025, https://www.zscaler.com/blogs/product-insights/quic-secure-communication-protocol-shaping-future-of-internet
38. What is QUIC? How Does It Boost HTTP/3? - CDNetworks, accessed July 29, 2025, https://www.cdnetworks.com/blog/media-delivery/what-is-quic/
39. What Is the QUIC Protocol? | EMQ - EMQX, accessed July 29, 2025, https://www.emqx.com/en/blog/quic-protocol-the-features-use-cases-and-impact-for-iot-iov
40. WebTransport: The Future of Real-Time Communication, Bridging the Gap Beyond WebRTC & Websockets | by Lakin Mohapatra, accessed July 29, 2025, https://lakin-mohapatra.medium.com/web-transport-the-future-of-real-time-communication-bridging-the-gap-beyond-webrtc-e1203f284ae9
41. An Analysis of QUIC Connection Migration in the Wild - arXiv, accessed July 29, 2025, https://arxiv.org/html/2410.06066v1
42. Use of QUIC for Mobile-Oriented Future Internet (Q-MOFI) - MDPI, accessed July 29, 2025, https://www.mdpi.com/2079-9292/13/2/431
43. Detailed Explanation of the QUIC Protocol: The Next-Generation Internet Transport Layer Protocol | by happyer | Medium, accessed July 29, 2025, https://medium.com/@threehappyer/detailed-explanation-of-the-quic-protocol-the-next-generation-internet-transport-layer-protocol-b680c0cf294a
44. Replacing WebRTC: real-time latency with WebTransport and WebCodecs - Reddit, accessed July 29, 2025, https://www.reddit.com/r/programming/comments/17juwdb/replacing_webrtc_realtime_latency_with/
45. Getting Media Over QUIC (MoQ) and WebRTC to like each other | Meetecho Blog, accessed July 29, 2025, https://www.meetecho.com/blog/moq-webrtc/
46. "webtransport" | Can I use... Support tables for HTML5, CSS3, etc - CanIUse, accessed July 29, 2025, https://caniuse.com/?search=webtransport
47. WebTransport | Can I use... Support tables for HTML5, CSS3, etc - CanIUse, accessed July 29, 2025, https://caniuse.com/webtransport
48. WebCodecs, WebTransport, and the Future of WebRTC - webrtcHacks, accessed July 29, 2025, https://webrtchacks.com/webcodecs-webtransport-and-webrtc/
49. DJI Transmission - Specs, accessed July 29, 2025, https://www.dji.com/transmission/specs
50. What is WebTransport and How Does it Work? - PubNub, accessed July 29, 2025, https://www.pubnub.com/guides/webtransport/
51. Sh3b0/realtime-web: Comparing WebSocket, WebRTC ... - GitHub, accessed July 29, 2025, https://github.com/Sh3b0/realtime-web
52. WebRTC Creeper Drone - Browser Controlled RC Car : 19 Steps - Instructables, accessed July 29, 2025, https://www.instructables.com/WebRTC-Creeper-Drone-Browser-Controlled-RC-Car/
53. WebRTC Dev Challenges & Sheerbit's Smart Solutions, accessed July 29, 2025, https://sheerbit.com/webrtc-dev-challenges-sheerbits-smart-solutions/
54. An analysis of Challenges face by WEBRTC Videoconferencing and a Remedial Architecture - ResearchGate, accessed July 29, 2025, https://www.researchgate.net/publication/312197377_An_analysis_of_Challenges_face_by_WEBRTC_Videoconferencing_and_a_Remedial_Architecture
55. I think if you want to compete with Rust, C or Zig you want to have a rich stand... | Hacker News, accessed July 29, 2025, https://news.ycombinator.com/item?id=31248091
56. Rust at Google, Outperforming C++ and Matching Go - Ardan Labs, accessed July 29, 2025, https://www.ardanlabs.com/news/2024/rust-at-google/
57. Rust vs GoLang on http/https/websocket/webrtc performance, accessed July 29, 2025, https://users.rust-lang.org/t/rust-vs-golang-on-http-https-websocket-webrtc-performance/71118
58. BiagioFesta/wtransport: Async-friendly WebTransport implementation in Rust - GitHub, accessed July 29, 2025, https://github.com/BiagioFesta/wtransport
59. HTTP server - list of Rust libraries/crates // Lib.rs, accessed July 29, 2025, https://lib.rs/web-programming/http-server




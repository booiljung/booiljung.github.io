# QUIC 프로토콜

## 1.  QUIC의 기원과 아키텍처의 근간

현대 인터넷 통신은 수십 년간 TCP(Transmission Control Protocol)라는 견고한 기반 위에 구축되어 왔다. 그러나 웹의 패러다임이 정적인 문서 교환에서 동적이고 상호작용이 풍부한 애플리케이션 중심으로 변화함에 따라, TCP가 내재한 근본적인 한계들이 점차 웹 성능의 발목을 잡는 족쇄가 되었다. QUIC(Quick UDP Internet Connections)는 이러한 한계를 극복하기 위해 등장한 단순한 프로토콜 개선이 아닌, 인터넷 전송 계층의 패러다임을 전환하는 혁신적인 시도이다. 본 보고서의 제 1부에서는 TCP 스택의 한계를 분석하여 QUIC의 등장이 필연적이었던 이유를 규명하고, 구글의 실험에서 시작하여 국제인터넷표준화기구(IETF)의 표준으로 자리 잡기까지의 발전 과정을 추적하며, UDP를 기반으로 한 QUIC의 근본적인 아키텍처 설계 철학을 심도 있게 탐구한다.

### 1.1  새로운 전송 프로토콜의 필요성: TCP 스택의 한계

QUIC의 등장은 기존 TCP 기반 프로토콜 스택이 현대 웹 환경의 요구사항을 충족시키지 못하는 여러 구조적 문제점에서 비롯되었다. 이러한 문제점들은 주로 지연 시간, 처리량 저하, 그리고 프로토콜 자체의 경직성에서 기인한다.

#### 1.1.1  TCP와 TLS의 지연 시간 문제

전통적인 보안 웹 연결(HTTPS)은 TCP와 TLS(Transport Layer Security)라는 두 개의 독립적인 프로토콜 계층 위에서 동작한다. 이 구조는 연결 설정 과정에서 필연적인 지연 시간을 발생시킨다. 클라이언트와 서버가 데이터를 교환하기 전에, 먼저 TCP 연결을 설정하기 위한 3-way 핸드셰이크(SYN, SYN-ACK, ACK)가 필요하며, 이는 최소 1 RTT(Round-Trip Time)를 소요한다.1 이 TCP 연결이 수립된 후에야 비로소 암호화 통신을 위한 TLS 핸드셰이크가 시작된다. TLS 1.2의 경우, 이 과정에서 추가적으로 2 RTT가 더 필요할 수 있다.2 따라서 사용자가 웹 페이지를 요청하고 첫 번째 데이터 바이트(Time to First Byte, TTFB)를 수신하기까지 최소 2~3 RTT의 지연이 발생하게 된다.3 이러한 순차적인 핸드셰이크 과정은 특히 지연 시간이 긴 모바일 네트워크나 위성 통신 환경에서 사용자 경험을 심각하게 저하시키는 주된 요인으로 작용한다.5

#### 1.1.2  Head-of-Line (HOL) 블로킹의 덫

HTTP/2는 단일 TCP 연결 내에서 여러 요청과 응답을 동시에 처리하는 스트림 멀티플렉싱 기능을 도입하여 HTTP/1.1의 HOL 블로킹 문제를 해결하고자 했다.6 그러나 이는 애플리케이션 계층에서의 해결책일 뿐, 전송 계층인 TCP가 가진 근본적인 HOL 블로킹 문제로부터는 자유롭지 못했다.7 TCP는 단일하고 순서가 보장된 바이트 스트림을 제공하는 프로토콜이다.5 즉, TCP 연결을 통해 전송되는 모든 데이터는 하나의 거대한 '체인'처럼 취급된다.8 만약 이 체인의 중간 링크에 해당하는 TCP 패킷 하나가 유실되거나 지연되면, TCP는 순서 보장을 위해 해당 패킷이 재전송되어 도착할 때까지 그 뒤에 오는 모든 패킷의 처리를 중단한다.3

이러한 특성은 HTTP/2의 멀티플렉싱 환경에서 치명적인 단점으로 작용한다. 예를 들어, 하나의 웹 페이지를 구성하는 이미지, CSS, 자바스크립트 파일들이 각각 독립적인 HTTP/2 스트림으로 전송되더라도, 이 모든 스트림은 단일 TCP 연결을 공유한다. 이때 이미지 파일에 해당하는 TCP 패킷 하나가 유실되면, 성공적으로 수신된 CSS나 자바스크립트 파일에 해당하는 패킷들마저도 애플리케이션 계층으로 전달되지 못하고 대기 상태에 빠진다.10 결국, 단일 스트림의 작은 문제가 전체 연결의 성능을 저하시키는 '전송 계층 HOL 블로킹' 현상이 발생하는 것이다.5

#### 1.1.3  프로토콜 경직성과 중간 장비 문제

TCP는 수십 년간 인터넷의 핵심 프로토콜로 사용되면서 운영체제 커널 깊숙이 자리 잡게 되었다. 이는 TCP 프로토콜에 새로운 기능을 추가하거나 수정하는 작업을 매우 어렵고 더디게 만들었다.10 커널 수준의 변경은 광범위한 테스트와 시스템 재부팅을 요구하며, 배포 주기가 매우 길어 빠르게 변화하는 인터넷 환경에 대응하기 어렵다.13

더욱 심각한 문제는 네트워크 경로상에 존재하는 수많은 중간 장비(Middlebox), 즉 방화벽, NAT(Network Address Translation) 장비, 로드 밸런서 등이 TCP의 발전을 가로막는다는 점이다.14 이들 중간 장비는 종종 특정 TCP 헤더 필드나 동작 방식에 대한 가정을 하드코딩하여, 표준에 부합하더라도 기존과 다른 패턴을 보이는 트래픽을 비정상으로 간주하고 차단하거나 변조하는 경우가 많다.14 이러한 현상을 '프로토콜 경직화(Ossification)'라고 부른다. 실제로 SCTP(Stream Control Transmission Protocol)와 같은 새로운 전송 프로토콜은 기술적 우수성에도 불구하고 중간 장비의 비호환성 문제로 인해 인터넷상에서 널리 배포되지 못했다.14 결국, TCP를 개선하거나 새로운 IP 기반 프로토콜을 도입하려는 시도는 이러한 현실적인 장벽에 부딪혀 좌절되기 일쑤였다.

이러한 기술적, 환경적 제약은 단순히 TCP를 개선하는 수준을 넘어, 기존의 제약에서 벗어날 수 있는 완전히 새로운 접근 방식이 필요함을 시사했다. QUIC의 설계 철학은 바로 이 지점에서 출발한다. 즉, QUIC는 커널과 중간 장비라는 '제도의 실패'를 우회하기 위한 전략적 선택의 산물이다. TCP를 직접 수정하는 대신, 거의 모든 네트워크에서 허용되는 UDP 위에 새로운 전송 계층을 구축함으로써, 프로토콜 혁신을 가로막는 구조적 장애물을 근본적으로 회피하고자 한 것이다. 이는 기술 개발의 주도권을 운영체제나 네트워크 장비 제조사가 아닌, 구글과 같은 애플리케이션 및 서비스 제공업체로 가져오는 중요한 패러다임 전환을 의미했다.

### 1.2  구글의 실험에서 IETF 표준까지: QUIC의 발전 과정

QUIC의 역사는 단일 기업의 혁신적인 실험에서 시작하여 전 세계 인터넷 커뮤니티의 협력을 통해 국제 표준으로 발전한 성공적인 사례를 보여준다. 이 과정은 크게 구글 내부에서의 개발, IETF로의 이전, 그리고 공식 표준화의 세 단계로 나눌 수 있다.

#### 1.2.1  구글에서의 기원 (2012-2013)

QUIC 프로토콜은 2012년 구글의 짐 로스킨드(Jim Roskind)에 의해 처음 설계되었으며, 같은 해에 구현 및 내부 서비스에 적용되기 시작했다.10 당시 구글의 목표는 웹 성능, 특히 지연 시간을 단축하여 사용자 경험을 개선하는 것이었다.1 초기 제안된 이름은 'Quick UDP Internet Connections'의 약어였으나, 훗날 IETF는 QUIC을 약어가 아닌 프로토콜의 고유 명사로 사용하기로 결정했다.10 2013년, 구글은 실험 범위를 확대하며 QUIC을 대외적으로 공개했고, 크롬 브라우저 버전 29부터 실험적으로 탑재하기 시작했다.10 이 시기의 QUIC은 'gQUIC'으로 불리며, 구글의 서비스(검색, 유튜브 등)와 크롬 브라우저 간의 통신을 최적화하는 데 집중되었다.18

#### 1.2.2  IETF로의 전환 (2015-2016)

gQUIC의 성공적인 실험 결과를 바탕으로, 구글은 이를 인터넷 전체의 표준으로 발전시키기 위한 노력을 시작했다. 2015년 6월, QUIC 사양에 대한 첫 번째 인터넷 드래프트(Internet-Draft)가 IETF에 제출되었다.10 이는 QUIC이 구글이라는 단일 기업의 기술을 넘어, 다양한 이해관계자들이 참여하는 개방형 표준으로 나아가는 첫걸음이었다. 이듬해인 2016년, IETF 내에 공식적으로 QUIC 워킹 그룹이 설립되면서 본격적인 표준화 작업이 시작되었다.10 이 시점부터 IETF 주도로 개발되는 QUIC은 'iQUIC'으로 불리며, 기존 gQUIC의 개념을 계승하되, 범용 전송 프로토콜로서의 역할을 강화하는 방향으로 상당한 변화와 개선을 겪게 되었다.18

#### 1.2.3  집중적인 표준화 과정 (2016-2021)

IETF 워킹 그룹에서의 표준화 과정은 매우 치열하고 집중적으로 진행되었다. 약 5년의 기간 동안 26번의 대면 회의, 1,749개에 달하는 공식 이슈 제기 및 해결, 그리고 수천 통의 이메일 토론이 이루어졌다.20 이 과정에는 구글뿐만 아니라 Cloudflare, Fastly, Akamai, Microsoft, Apple 등 다양한 기업과 학계의 전문가들이 참여하여 프로토콜의 완성도를 높였다.19 특히, 2018년 10월에는 IETF의 HTTP 워킹 그룹과 QUIC 워킹 그룹이 공동으로 QUIC 기반의 새로운 HTTP 버전을 'HTTP/3'로 명명하기로 결정하면서, QUIC이 차세대 웹 프로토콜의 기반이 될 것임을 공식화했다.18

#### 1.2.4  공식 표준화 (2021년 5월)

마침내 2021년 5월, IETF는 QUIC 버전 1을 RFC 9000으로 공식 발표했다.10 이는 QUIC의 핵심 전송 메커니즘을 정의하는 문서로, 이와 함께 관련 표준 문서들이 함께 발표되었다. 구체적으로는 QUIC의 버전 협상 및 불변 속성을 다루는 RFC 8999, TLS 1.3을 사용한 보안 메커니즘을 상세히 기술하는 RFC 9001, 그리고 손실 복구 및 혼잡 제어를 규정하는 RFC 9002가 그것이다.18 이로써 QUIC은 공식적인 인터넷 표준 프로토콜로 자리매김했으며, 이후 HTTP/3(RFC 9114)가 표준화되면서 웹 통신의 새로운 시대를 여는 기술적 토대를 마련했다.7

### 1.3  아키텍처 패러다임의 전환: UDP 기반 설계

QUIC이 TCP의 한계를 극복하기 위해 선택한 가장 근본적인 설계 결정은 전송 계층의 기반을 TCP에서 UDP(User Datagram Protocol)로 전환한 것이다. 이는 단순히 프로토콜을 바꾼 것을 넘어, 인터넷 프로토콜 설계 철학의 중요한 전환을 의미한다.

#### 1.3.1  배포 가능성을 위한 현실적 선택

QUIC 개발자들이 UDP를 선택한 것은 UDP 자체의 기술적 우월성 때문이 아니라, 앞서 언급된 '프로토콜 경직화'와 '중간 장비' 문제를 회피하기 위한 매우 전략적이고 현실적인 결정이었다.7 새로운 IP 프로토콜 번호를 사용하는 SCTP와 같은 프로토콜은 중간 장비에 의해 차단될 위험이 매우 높았지만, UDP는 DNS와 같은 필수 서비스에 널리 사용되기 때문에 거의 모든 네트워크 환경에서 허용되는 몇 안 되는 프로토콜 중 하나였다.14 QUIC은 신뢰성 있는 전송에 필요한 모든 복잡한 기능들을 암호화된 UDP 데이터그램 내부에 숨김으로써, 중간 장비의 간섭을 최소화하고 기존 인터넷 인프라에 수정 없이 배포될 수 있는 능력을 확보했다.6

#### 1.3.2  지능을 사용자 공간으로 이동

UDP는 IP 주소와 포트 번호 외에는 거의 아무런 기능을 제공하지 않는 최소한의 프로토콜이다. QUIC은 이러한 UDP의 단순함을 역이용하여, TCP가 커널 수준에서 제공하던 연결 설정, 신뢰성 보장(패킷 재전송, 순서 보장), 흐름 제어, 혼잡 제어, 그리고 보안 기능까지 모두 애플리케이션과 함께 배포되는 라이브러리, 즉 사용자 공간(User-space)에서 직접 구현했다.3

#### 1.3.3  장점과 트레이드오프

이러한 아키텍처 전환은 여러 가지 중요한 장점을 가져왔다. 가장 큰 이점은 프로토콜의 빠른 진화가 가능해졌다는 점이다. 새로운 혼잡 제어 알고리즘을 적용하거나 프로토콜의 버그를 수정하기 위해 운영체제 전체를 업데이트할 필요 없이, 애플리케이션이나 라이브러리만 업데이트하면 되기 때문이다.10 이는 구글과 같은 대규모 서비스 제공업체가 전 세계 수십억 명의 사용자에게 새로운 기능을 신속하게 배포하고 테스트할 수 있게 해주는 결정적인 요인이 되었다.

하지만 이러한 유연성은 대가를 수반한다. 사용자 공간에서의 프로토콜 처리는 커널 공간에서의 처리보다 본질적으로 더 많은 오버헤드를 발생시킨다. 커널과 사용자 공간 사이의 빈번한 데이터 복사와 컨텍스트 스위칭은 CPU 사용량을 증가시키는 주요 원인이 된다.27 또한, 수십 년간 최적화되고 네트워크 카드 하드웨어 가속(Offloading)의 혜택을 받는 TCP 스택과 달리, 초기 QUIC 구현은 이러한 최적화가 부족하여 동일한 대역폭에서 더 많은 CPU 자원을 소모하는 경향을 보였다.30 이 CPU 오버헤드 문제는 QUIC의 중요한 트레이드오프로, 보고서의 제 3부에서 심도 있게 다룰 것이다.

## 2.  QUIC 핵심 메커니즘 심층 분석

QUIC의 혁신은 단순히 UDP를 기반으로 한다는 점에 그치지 않는다. 기존 전송 프로토콜의 고질적인 문제들을 해결하기 위해 설계된 정교하고 독창적인 메커니즘들의 유기적인 결합체라는 점에서 그 진정한 가치를 찾을 수 있다. 제 2부에서는 QUIC을 구성하는 핵심 기술들을 개별적으로 분해하여 그 작동 원리를 상세히 분석한다. 연결 설정 시간을 획기적으로 단축시킨 1-RTT 및 0-RTT 핸드셰이크, 전송 계층 HOL 블로킹을 근본적으로 해결한 스트림 멀티플렉싱, 모바일 환경에서의 끊김 없는 통신을 보장하는 연결 마이그레이션, 그리고 프로토콜의 근간을 이루는 TLS 1.3의 통합 방식과 새로운 혼잡 제어 프레임워크에 이르기까지, QUIC이 어떻게 더 빠르고, 더 안정적이며, 더 안전한 인터넷을 구현하는지 기술적으로 규명한다.

### 2.1  연결 설정과 지연 시간 단축: 1-RTT 및 0-RTT 핸드셰이크

QUIC의 가장 두드러진 장점 중 하나는 연결 설정에 소요되는 시간을 극적으로 단축시킨 것이다. 이는 전송 계층 핸드셰이크와 암호화 핸드셰이크를 통합하고, 재연결 시에는 핸드셰이크 과정을 거의 생략하는 혁신적인 접근을 통해 달성된다.

#### 2.1.1  1-RTT 핸드셰이크

기존의 TCP+TLS 스택이 최소 2 RTT 이상을 소요하는 것과 달리, QUIC은 최초 연결 시 단 1 RTT만으로 보안 연결을 수립할 수 있다.1 이 과정은 다음과 같이 진행된다:

1. **클라이언트의 Initial 패킷**: 클라이언트는 서버에 연결을 시도할 때 첫 번째 패킷인 `Initial` 패킷을 전송한다. 이 패킷에는 QUIC 연결 설정을 위한 정보와 함께, 암호화 협상을 위한 TLS `ClientHello` 메시지가 포함되어 있다.1 즉, 전송 계층의 첫 통신과 암호화 계층의 첫 통신이 하나의 패킷으로 결합된다.
2. **서버의 응답**: 서버는 클라이언트의 `Initial` 패킷을 수신한 후, 필요한 암호화 키를 생성하고 자신의 `ServerHello`, 인증서(Certificate), 그리고 핸드셰이크 완료 메시지를 담은 패킷을 클라이언트로 응답한다.1
3. **연결 수립 완료**: 클라이언트는 서버의 응답을 수신하고 검증한 후, 암호화된 통신을 시작할 준비를 마친다. 이로써 클라이언트와 서버가 각각 한 번씩 패킷을 주고받는 단 한 번의 왕복(1 RTT)만으로 모든 핸드셰이크 과정이 완료된다.32 이 방식은 TCP+TLS 대비 연결 설정 시간을 약 67% 단축시키는 효과를 가져온다.32

#### 2.1.2  0-RTT 핸드셰이크 (연결 재개)

QUIC은 이전에 한 번이라도 연결했던 서버에 다시 접속할 때, 핸드셰이크에 소요되는 시간을 0으로 만드는 0-RTT 연결 재개 기능을 제공한다.32 이는 사용자 경험을 극적으로 향상시키는 핵심 기능으로, 다음과 같은 원리로 동작한다:

1. **세션 정보 캐싱**: 첫 번째 연결(1-RTT 핸드셰이크)이 성공적으로 완료되면, 서버는 클라이언트에게 세션 티켓(Session Ticket) 또는 재개 토큰(Resumption Token)이라는 암호화된 정보를 제공한다. 클라이언트는 이 정보와 함께 협상된 암호화 매개변수(예: 사전 공유 키, Pre-Shared Key, PSK)를 안전하게 캐싱한다.1
2. **데이터 즉시 전송**: 이후 클라이언트가 동일한 서버에 다시 연결할 때, 첫 번째 패킷에 캐싱해 둔 PSK를 사용하여 암호화한 애플리케이션 데이터(예: HTTP GET 요청)를 함께 담아 전송한다.1
3. **핸드셰이크 없는 응답**: 서버는 이 0-RTT 패킷을 수신하고 PSK를 검증한 후, 핸드셰이크 완료를 기다리지 않고 즉시 해당 요청을 처리하고 응답 데이터를 전송할 수 있다.1

이 0-RTT 기능은 TLS 1.3에서도 유사한 개념이 존재하지만, QUIC의 0-RTT는 TCP 핸드셰이크 과정 자체가 없기 때문에 실질적인 '제로 라운드 트립'을 구현한다는 점에서 더 뛰어나다.35 이로 인해 사용자는 이전에 방문했던 웹사이트를 다시 열 때 거의 즉각적인 로딩 속도를 경험할 수 있다. 그러나 이 기능은 재전송 공격(Replay Attack)과 같은 보안상 위험을 내포하고 있어, 신중한 적용이 필요하다 (4.2절에서 상세히 논의).

### 2.2  Head-of-Line 블로킹의 결정적 해결책

QUIC은 HTTP/2 over TCP가 해결하지 못한 전송 계층 HOL 블로킹 문제를 근본적으로 해결한다. 이는 스트림(Stream)이라는 개념을 애플리케이션 계층이 아닌 전송 계층의 핵심 요소로 설계함으로써 가능해졌다.

#### 2.2.1  전송 계층의 네이티브 스트림

HTTP/2에서 스트림은 애플리케이션 프로토콜(HTTP) 수준의 논리적 개념이었지만, QUIC에서는 스트림이 전송 프로토콜 자체의 기본 단위(First-class primitive)이다.37 각 QUIC 스트림은 독립적인 순서 보장 바이트-스트림 추상화를 제공하며, 고유한 스트림 ID를 통해 식별된다.37 클라이언트와 서버는 하나의 QUIC 연결 내에서 수많은 스트림을 생성하고 데이터를 주고받을 수 있다.

#### 2.2.2  스트림의 독립적 처리

QUIC의 HOL 블로킹 해결 원리는 바로 이 '스트림의 독립성'에 있다. 전체적인 동작 과정은 다음과 같다:

1. **프레임화 및 패킷화**: 각 스트림의 데이터는 `STREAM`이라는 프레임 단위로 분할된다.37 이 `STREAM` 프레임들은 다른 스트림의 프레임들과 함께 하나의 QUIC 패킷에 담겨 UDP 데이터그램으로 전송될 수 있다.37
2. **독립적인 손실 처리**: 수신 측에서 특정 QUIC 패킷이 유실되었다고 가정해보자. 이 패킷에는 예를 들어 스트림 A와 스트림 B의 데이터가 담겨 있었다.
3. **선택적 대기**: QUIC 수신단은 유실된 패킷에 포함된 스트림 A와 B의 데이터만 대기한다.
4. **지속적인 처리**: 그동안 다른 QUIC 패킷들이 성공적으로 도착하고, 그 패킷들에 스트림 C와 D의 데이터가 포함되어 있다면, 수신단은 스트림 A, B와는 무관하게 스트림 C, D의 데이터를 즉시 처리하여 애플리케이션 계층으로 전달할 수 있다.3

이처럼 한 스트림에서 발생한 패킷 손실이 다른 스트림의 데이터 처리에 영향을 주지 않기 때문에, TCP에서 발생하던 전송 계층 HOL 블로킹이 원천적으로 차단된다.11 이는 특히 손실이 잦은 무선 네트워크 환경에서 웹 페이지 로딩 속도와 비디오 스트리밍 품질을 획기적으로 개선하는 핵심적인 메커니즘이다.

### 2.3  연결 마이그레이션을 통한 완벽한 이동성

현대 인터넷 사용 환경의 핵심은 이동성이다. 사용자는 Wi-Fi와 셀룰러 데이터 네트워크 사이를 수시로 오가며, 이 과정에서 IP 주소가 변경되는 일은 매우 흔하다. QUIC은 이러한 네트워크 변화에도 연결이 끊기지 않는 강력한 이동성을 제공하며, 이는 '연결 ID(Connection ID)'라는 독창적인 개념을 통해 구현된다.

#### 2.3.1  연결 ID (CID)

TCP 연결은 출발지 IP, 출발지 포트, 목적지 IP, 목적지 포트로 구성된 4-튜플(4-tuple)에 의해 식별된다.13 이 4-튜플 중 하나라도 변경되면(예: Wi-Fi에서 셀룰러로 전환 시 출발지 IP 변경), TCP 연결은 유효하지 않게 되어 끊어지고, 모든 통신을 처음부터 다시 시작해야 한다. 이는 실시간 통화나 동영상 스트리밍 중에 심각한 끊김 현상을 유발한다.

QUIC은 이러한 문제를 해결하기 위해 연결을 식별하는 방식을 근본적으로 바꿨다. QUIC 연결은 네트워크 주소에 의존하는 대신, 각 엔드포인트가 생성하는 고유한 식별자인 **연결 ID(Connection ID, CID)**에 의해 식별된다.1 이 CID는 QUIC 패킷 헤더의 암호화되지 않은 부분에 포함되어, 네트워크 장비가 패킷을 올바른 목적지로 라우팅하는 데 사용된다.43

#### 2.3.2  연결과 경로의 분리

CID의 도입은 논리적인 '연결(Connection)' 상태를 물리적인 '경로(Path)'로부터 분리하는 효과를 가져온다.43 클라이언트의 IP 주소가 변경되더라도, 클라이언트는 동일한 CID를 사용하여 새로운 IP 주소에서 서버로 패킷을 보내기만 하면 된다. 서버는 패킷 헤더의 CID를 보고 이 패킷이 기존에 진행 중이던 연결에 속한다는 것을 인지하고, 중단 없이 통신을 계속할 수 있다.4 이 전체 과정은 애플리케이션에 완전히 투명하게 이루어져, 애플리케이션은 네트워크 환경이 변경되었는지조차 인지할 필요가 없다.42

#### 2.3.3  경로 검증 (Path Validation)

클라이언트가 새로운 IP 주소에서 CID를 포함한 패킷을 보내는 것만으로는 충분하지 않다. 악의적인 공격자가 다른 사용자의 CID를 도용하여 패킷을 보내는 것을 방지하고, 새로운 네트워크 경로가 실제로 양방향 통신이 가능한지 확인해야 한다. 이를 위해 QUIC은 '경로 검증'이라는 절차를 수행한다.42

1. **PATH_CHALLENGE 프레임 전송**: 클라이언트는 새로운 네트워크 경로(새로운 IP 주소)를 사용하기 전에, 해당 경로를 통해 임의의 데이터를 담은 `PATH_CHALLENGE` 프레임을 서버로 전송한다.43
2. **PATH_RESPONSE 프레임 응답**: 서버는 `PATH_CHALLENGE` 프레임을 수신한 동일한 경로를 통해, 클라이언트가 보낸 데이터를 그대로 담은 `PATH_RESPONSE` 프레임으로 응답한다.42
3. **경로 검증 완료**: 클라이언트가 이 응답을 성공적으로 수신하면, 해당 경로는 양방향 통신이 가능한 것으로 검증되며, 이후부터 이 경로를 통해 안전하게 애플리케이션 데이터를 전송할 수 있다.

이러한 연결 마이그레이션 기능은 NAT 장비가 일정 시간 비활성 상태 후 매핑 정보를 변경하는 'NAT 리바인딩(NAT Rebinding)' 현상에도 효과적으로 대응할 수 있어, 전반적인 연결 안정성을 크게 향상시킨다.42

### 2.4  설계 기반 보안: TLS 1.3의 심층 통합

QUIC은 보안을 부가 기능이 아닌 프로토콜의 핵심적인 내재 속성으로 설계했다. 이를 위해 최신 암호화 표준인 TLS 1.3을 프로토콜 깊숙이 통합하여, 모든 통신이 기본적으로 암호화되도록 보장한다.

#### 2.4.1  필수적이고 통합된 암호화

QUIC에는 암호화되지 않은 평문 통신 모드가 존재하지 않는다.4 이는 TCP와 가장 큰 차이점 중 하나로, TCP에서는 TLS가 선택적으로 적용되는 별도의 계층이지만 QUIC에서는 프로토콜의 일부이다.15 QUIC은 TLS 1.3의 핸드셰이크 메시지(예: `ClientHello`, `ServerHello`)를 그대로 사용하되, 이를 전송하는 방식에서 차이를 보인다. 기존의 TLS 레코드 계층을 사용하는 대신, QUIC은 TLS 핸드셰이크 데이터를 자체적인 `CRYPTO` 프레임에 담아 전송한다.19 이 `CRYPTO` 프레임은 QUIC 패킷에 포함되어 전송되며, 이 과정에서 전송 계층 핸드셰이크와 암호화 핸드셰이크가 동시에 이루어진다.

#### 2.4.2  인증 및 암호화된 헤더

QUIC의 보안은 페이로드에만 국한되지 않는다. QUIC 패킷 헤더의 상당 부분 또한 암호화된다. QUIC 헤더는 크게 두 부분으로 나뉜다:

- **공개 헤더 (Public Header)**: 암호화되지 않는 부분으로, 패킷 유형을 나타내는 플래그, 버전 정보, 그리고 연결을 식별하는 목적지 CID(Destination CID) 등이 포함된다. 이는 네트워크 장비가 패킷을 올바른 엔드포인트로 전달하는 데 필요한 최소한의 정보이다.
- **보호된 헤더 (Protected Header)**: 패킷 번호(Packet Number)와 같이 민감한 정보가 포함된 부분은 '헤더 보호(Header Protection)'라는 별도의 암호화 과정을 거친다.32 이는 TLS 핸드셰이크를 통해 생성된 특정 키를 사용하여 헤더의 일부를 마스킹하는 기술로, 중간 장비가 패킷 번호를 관찰하여 사용자의 트래픽 패턴을 분석하는 것을 방지하고, 프로토콜 경직화를 막는 데 기여한다.15

#### 2.4.3  키 파생 및 패킷 보호

TLS 1.3 핸드셰이크 과정에서 협상된 마스터 시크릿(Master Secret)으로부터 여러 종류의 암호화 키가 파생된다. QUIC은 통신 단계를 여러 '암호화 수준(Encryption Level)'으로 나누고, 각 수준마다 별도의 키와 패킷 번호 공간을 사용하여 패킷을 보호한다.45

- **Initial 키**: 핸드셰이크 초기에 사용되는 키로, 최소한의 보호를 제공한다.
- **0-RTT 키**: 연결 재개 시 사용되는 키로, PSK로부터 파생된다.
- **Handshake 키**: 핸드셰이크 메시지를 보호하는 데 사용되는 키이다.
- **1-RTT 키**: 핸드셰이크가 완료된 후, 실제 애플리케이션 데이터를 보호하는 데 사용되는 최종 키이다.

이처럼 단계별로 다른 키를 사용하는 정교한 키 파생 구조는 핸드셰이크 과정의 각 단계에 필요한 최소한의 보안을 제공하면서도, 최종적으로는 강력한 전방위 기밀성(Forward Secrecy)을 보장한다.

### 2.5  새로운 시대의 혼잡 제어

QUIC은 혼잡 제어 메커니즘에서도 중요한 발전을 이루었다. 사용자 공간 구현을 통해 유연성을 확보하고, 패킷 번호 체계를 개선하여 손실 감지 및 RTT 측정의 정확도를 높였다.

#### 2.5.1  유연한 사용자 공간 프레임워크

TCP의 혼잡 제어 알고리즘은 운영체제 커널에 구현되어 있어 수정 및 배포가 어려운 반면, QUIC은 혼잡 제어 로직을 사용자 공간 라이브러리에 구현했다.10 이는 다음과 같은 장점을 가진다:

- **빠른 혁신**: 새로운 혼잡 제어 알고리즘(예: BBR)을 개발하고 배포하는 속도가 훨씬 빠르다. 커널 업데이트 없이 애플리케이션 업데이트만으로 최신 기술을 적용할 수 있다.6
- **애플리케이션별 최적화**: 각 애플리케이션의 특성(예: 실시간 비디오 스트리밍, 대용량 파일 전송)에 맞는 맞춤형 혼잡 제어 알고리즘을 선택적으로 적용하는 것이 가능하다.6

#### 2.5.2  향상된 손실 감지 및 RTT 측정

QUIC의 성능 향상에 기여하는 숨은 공신은 바로 패킷 번호 체계의 혁신이다.

- **재전송 모호성 제거**: TCP에서는 패킷을 재전송할 때 원래의 시퀀스 번호를 그대로 사용한다. 이 때문에 수신 측에서 받은 ACK가 원본 패킷에 대한 것인지, 재전송된 패킷에 대한 것인지 모호해지는 '재전송 모호성(Retransmission Ambiguity)' 문제가 발생한다.13 이는 RTT 측정의 정확도를 떨어뜨리고, 불필요한 전송 지연을 유발할 수 있다.
- **고유한 패킷 번호**: QUIC에서는 모든 패킷, 심지어 재전송되는 패킷조차도 새롭고 고유한 패킷 번호를 부여받는다.13 따라서 어떤 ACK가 어떤 패킷에 대한 응답인지 명확하게 알 수 있다. 이로 인해 RTT를 훨씬 더 정확하게 측정할 수 있으며, 이는 손실을 더 빠르고 정밀하게 감지하여 혼잡 제어 알고리즘이 더 효율적으로 작동하도록 돕는다.26

#### 2.5.3  일반적인 알고리즘

QUIC 자체는 특정 혼잡 제어 알고리즘을 강제하지 않으며, '플러그형(Pluggable)' 구조를 지향한다. 현재 널리 사용되거나 연구되는 알고리즘에는 TCP에서 오랫동안 검증된 손실 기반 알고리즘인 NewReno와 CUBIC, 그리고 구글이 개발하여 지연 시간을 기반으로 대역폭을 추정하는 BBR(Bottleneck Bandwidth and Round-trip propagation time) 등이 있다.33 IETF RFC 9002에서는 NewReno와 유사한 알고리즘을 표준 혼잡 제어기로 권고하고 있다.49

이러한 핵심 메커니즘들은 개별적으로도 강력하지만, 서로 긴밀하게 연결되어 시너지를 창출한다. 예를 들어, TLS 1.3의 통합은 1-RTT 및 0-RTT 핸드셰이크를 가능하게 하는 전제 조건이며, 전송 계층의 네이티브 스트림은 HOL 블로킹을 해결하는 기반이 된다. 또한, 연결 ID는 암호화된 세션 상태와 결합하여 안전한 연결 마이그레이션을 구현한다. 이처럼 QUIC의 설계는 전통적인 OSI 모델의 엄격한 계층 구분을 넘어, 전송, 보안, 애플리케이션의 요구사항을 통합적으로 고려한 결과물이다. 이는 기존 프로토콜 스택에서는 불가능했던 수준의 최적화를 달성하게 했으며, QUIC을 단순한 TCP의 대체재가 아닌, 차세대 인터넷 전송을 위한 새로운 플랫폼으로 자리매김하게 만들었다.

## 3.  비교 성능 및 프로토콜 분석

QUIC의 이론적 우수성이 실제 네트워크 환경에서 어떻게 발현되는지를 검증하는 것은 매우 중요하다. 제 3부에서는 QUIC을 기존의 TCP+TLS+HTTP/2 스택과 직접적으로 비교하여 성능을 다각도로 분석한다. 먼저, 프로토콜의 핵심적인 차이점을 명확히 정리하고, 이후 실제 측정된 벤치마크 데이터를 기반으로 지연 시간, 처리량 등 다양한 시나리오에서의 성능을 심층적으로 평가한다. 특히, QUIC의 가장 큰 약점으로 지적되는 높은 CPU 사용량 문제의 원인을 규명하고, 이를 완화하기 위한 기술적 접근법을 모색함으로써 QUIC 도입에 따른 현실적인 트레이드오프를 조명한다.

### 3.1  QUIC 대 TCP+TLS+HTTP/2: 다각적 프로토콜 비교

QUIC과 기존 TCP 기반 스택의 차이점은 여러 측면에 걸쳐 있으며, 이는 각 프로토콜의 설계 철학과 목표가 근본적으로 다름을 보여준다. 아래 표는 두 프로토콜 스택의 핵심적인 특징을 비교하여 정리한 것이다.

**표 1: 전송 프로토콜 스택 비교 분석**

| 기능/지표                  | TCP+TLS+HTTP/2 스택            | QUIC+HTTP/3 스택                 |
| -------------------------- | ------------------------------ | -------------------------------- |
| **기반 프로토콜**          | TCP                            | UDP                              |
| **구현 위치**              | 커널 공간(Kernel-space)        | 사용자 공간(User-space)          |
| **연결 핸드셰이크 (최초)** | 2+ RTT (TCP 3-way + TLS)       | 1 RTT (통합 핸드셰이크)          |
| **연결 핸드셰이크 (재개)** | 1-2 RTT                        | 0-RTT                            |
| **멀티플렉싱**             | 애플리케이션 계층 (HTTP/2)     | 전송 계층 (QUIC)                 |
| **Head-of-Line 블로킹**    | 전송 계층 HOL 블로킹 발생      | 전송 계층 HOL 블로킹 없음        |
| **보안**                   | 선택적 (TLS는 별도 계층)       | 필수적, 내장 (TLS 1.3)           |
| **메타데이터 프라이버시**  | 전송 헤더(시퀀스 번호 등) 평문 | 대부분의 전송 헤더 암호화        |
| **연결 지속성**            | 4-튜플(IP/포트)에 종속         | 연결 ID(CID)로 분리              |
| **오류 복구**              | 재전송 모호성 존재             | 고유 패킷 번호로 RTT 정밀 측정   |
| **혼잡 제어**              | 커널에 종속                    | 사용자 공간, 플러그형(Pluggable) |

이 표는 QUIC이 지연 시간 단축(핸드셰이크), 처리량 안정성(HOL 블로킹 해결), 이동성(연결 ID), 보안(필수 암호화), 그리고 진화 가능성(사용자 공간 구현) 측면에서 구조적인 우위를 가지고 있음을 명확히 보여준다. TCP 스택이 수십 년간 개별적으로 발전해 온 기능들을 계층적으로 쌓아 올린 형태라면, QUIC은 현대 웹의 요구사항에 맞춰 이 기능들을 하나의 계층으로 통합하고 최적화한 결과물이다.

### 3.2  실제 환경에서의 성능 벤치마크: 미묘한 차이

QUIC의 실제 성능은 네트워크 환경과 워크로드의 특성에 따라 복합적인 양상을 보인다. 여러 연구와 실제 배포 사례를 종합해 보면, QUIC의 성능 우위는 특정 조건에서 극대화되지만, 모든 상황에서 절대적인 것은 아니다.

#### 3.2.1  지연 시간의 명백한 우위

QUIC의 가장 확실하고 일관된 성능 이점은 연결 설정 지연 시간의 감소이다. Cloudflare의 측정 결과에 따르면, QUIC은 TCP 대비 평균 15%의 지연 시간 감소 효과를 보였다.32 특히, 유튜브 미디어 서버를 대상으로 한 연구에서는 QUIC 핸드셰이크가 TCP+TLS 대비 인도에서 534ms, 독일에서 406ms 더 빠른 것으로 나타났다.30 이는 1-RTT 및 0-RTT 핸드셰이크가 실제 네트워크 환경에서 효과적으로 작동함을 입증하는 결과로, 사용자가 느끼는 초기 로딩 속도에 직접적인 영향을 미친다.

#### 3.2.2  손실이 많은/모바일 네트워크에서의 처리량

QUIC은 패킷 손실률이 높고 RTT가 긴 네트워크, 즉 모바일 및 무선 환경에서 진가를 발휘한다. HOL 블로킹을 해결한 덕분에, 일부 패킷이 손실되더라도 다른 스트림의 전송은 계속될 수 있다. 이는 비디오 스트리밍 시 버퍼링(Stall) 발생 횟수와 지속 시간을 크게 줄이는 효과로 이어진다.52 한 연구에서는 높은 손실률을 가진 네트워크에서 QUIC의 버퍼링 지속 시간이 TCP 대비 50%까지 짧아지는 것을 관찰했다.30 또한, Cloudflare는 모바일 환경에서 35%, 패킷 손실 환경에서는 50% 더 나은 성능을 보고했다.28 이는 QUIC의 뛰어난 손실 복구 메커니즘이 불안정한 네트워크 환경에서 전반적인 처리량을 안정적으로 유지하는 데 기여하기 때문이다.54

#### 3.2.3  고대역폭 환경의 역설

역설적이게도, 네트워크 환경이 매우 좋은 고대역폭(일반적으로 500-600 Mbps 이상) 환경에서는 QUIC의 처리량이 오히려 최적화된 TCP 스택에 미치지 못하는 현상이 다수 보고되었다.30 한 연구에서는 1Gbps 링크에서 대용량 파일을 전송할 때 QUIC이 HTTP/2보다 최대 45.2% 느렸으며, 대역폭이 증가할수록 성능 격차가 더 벌어지는 경향을 보였다.55 이는 QUIC의 성능 프로파일이 모든 시나리오에 대해 일률적이지 않음을 보여주는 중요한 결과이다. 이 현상의 주된 원인은 다음 절에서 다룰 CPU 오버헤드 문제와 직결된다.

### 3.3  CPU 오버헤드의 딜레마: QUIC의 아킬레스건

QUIC의 유연성과 풍부한 기능은 상당한 컴퓨팅 비용을 수반한다. 특히 서버 측면에서 높은 CPU 사용량은 QUIC 도입 시 반드시 고려해야 할 핵심적인 트레이드오프이다.

#### 3.3.1  비용 정량화

여러 벤치마크에서 QUIC의 높은 CPU 사용량이 일관되게 관찰되었다. 대용량 파일 전송 시 TCP 대비 2배에서 4배 더 많은 CPU 시간을 소모할 수 있으며 56, 구글 드라이브에서 200MB 파일을 다운로드할 때 CPU 사용량이 거의 두 배에 달했다는 연구 결과도 있다.51 한 실제 테스트에서는 단일 CPU 코어로 QUIC을 처리할 때 처리량이 400Mbps에서 한계에 도달한 반면, 동일한 하드웨어에서 TCP+TLS는 5Gbps를 무리 없이 처리했다.30 이는 고대역폭 환경에서 CPU가 병목 현상을 일으켜 QUIC의 잠재적 처리량을 제한할 수 있음을 시사한다.

#### 3.3.2  근본 원인 분석

QUIC의 높은 CPU 오버헤드는 여러 요인이 복합적으로 작용한 결과이다:

1. **사용자 공간 처리**: 가장 근본적인 원인이다. 모든 패킷 처리 로직이 사용자 공간에서 실행되므로, 수신된 각 UDP 데이터그램은 커널의 네트워크 스택에서 사용자 공간의 QUIC 애플리케이션으로 복사되어야 한다. 이 과정에서 발생하는 빈번한 데이터 복사와 시스템 콜, 그리고 컨텍스트 스위칭은 상당한 CPU 오버헤드를 유발한다.27 한 분석에 따르면 패킷 입출력(I/O)이 전체 CPU 사용량의 최대 50%를 차지할 수 있다.57
2. **하드웨어 오프로딩 미성숙**: 현대의 TCP 스택은 TSO(TCP Segmentation Offload), LRO(Large Receive Offload)와 같은 다양한 하드웨어 오프로딩 기능의 혜택을 본다. 이러한 기능들은 네트워크 인터페이스 카드(NIC)가 패킷 분할 및 병합과 같은 작업을 대신 처리하여 CPU의 부담을 덜어준다. 반면, UDP 기반의 QUIC은 아직 이러한 하드웨어 가속 지원이 성숙하지 않았으며, 특히 UDP GRO(Generic Receive Offload)의 부재가 성능 저하의 주요 원인으로 지목된다.30
3. **패킷 단위 암호화**: QUIC은 페이로드뿐만 아니라 헤더의 일부를 포함하여 모든 패킷을 개별적으로 암호화하고 인증한다. 이는 TCP+TLS가 바이트 스트림 단위로 암호화를 수행하고 ACK와 같은 제어 패킷은 평문으로 보내는 것과 대조적이다. 따라서 QUIC은 패킷당 더 많은 암호화 연산을 필요로 한다.27

#### 3.3.3  완화 전략 및 미래 전망

이러한 CPU 오버헤드 문제를 해결하기 위한 연구와 개발이 활발히 진행 중이다.

- **소프트웨어 최적화**: `sendmmsg`, `recvmmsg`와 같은 시스템 콜을 사용하여 여러 패킷을 한 번에 처리함으로써 시스템 콜 오버헤드를 줄일 수 있다. 또한, 커널 수준에서 UDP GSO/GRO를 구현하고 활성화하는 것이 성능 향상에 크게 기여할 수 있다.55
- **커널 바이패스(Kernel Bypass)**: DPDK(Data Plane Development Kit)와 같은 기술을 사용하여 네트워크 스택을 완전히 사용자 공간으로 가져와 커널-사용자 공간 간 데이터 복사를 원천적으로 제거하는 방법이다. 이는 최고의 성능을 제공하지만, 구현 복잡성이 매우 높다.57
- **QUIC 하드웨어 오프로딩**: 장기적인 해결책은 NIC 제조사들이 QUIC 처리에 특화된 하드웨어 오프로딩 기능을 개발하고 보급하는 것이다. 패킷 단위 암호화/복호화, 헤더 보호, CID 기반 패킷 분배 등을 하드웨어 수준에서 처리하게 되면 CPU 부담을 획기적으로 줄일 수 있을 것이다.30

결론적으로, QUIC의 성능 프로파일은 그 설계 철학을 반영한다. QUIC는 통제된 데이터센터 환경에서의 순수 처리량 극대화보다는, 예측 불가능하고 손실이 잦은 실제 인터넷 환경, 특히 모바일 '라스트 마일(last mile)'에서의 사용자 체감 지연 시간을 줄이는 데 최적화되어 있다. 구글과 같은 서비스 제공업체들은 서버 측의 CPU 비용 증가(이는 시간이 지남에 따라 최적화될 수 있는 비용)를 감수하는 대신, 수십억 최종 사용자의 경험을 개선(이는 직접적으로 사용자 참여도와 수익에 영향을 미침)하는 전략적 선택을 한 것이다. 따라서 QUIC의 성능은 단일 지표로 평가될 수 없으며, 그 목표와 트레이드오프를 종합적으로 이해해야만 올바른 평가가 가능하다.

## 4.  구현 과제와 전략적 고려사항

QUIC의 기술적 장점에도 불구하고, 이를 실제 네트워크 환경에 도입하고 운영하는 과정은 여러 가지 현실적인 도전 과제에 직면한다. 제 4부에서는 QUIC 채택 시 발생하는 주요 문제점들을 분석하고, 이에 대한 전략적 고려사항을 제시한다. 기존 네트워크 인프라와의 충돌을 야기하는 중간 장비 문제, 성능과 보안 사이의 미묘한 균형을 요구하는 0-RTT 기능의 위험성, 그리고 현재 QUIC의 생태계가 어느 정도 성숙했는지를 가늠할 수 있는 도입 현황을 종합적으로 검토한다.

### 4.1  네트워크 탐색: 중간 장비 문제의 재조명

QUIC이 UDP를 기반으로 설계된 주된 이유 중 하나는 중간 장비 문제를 회피하기 위함이었지만, 역설적으로 UDP 자체와 QUIC의 강력한 암호화 정책이 새로운 형태의 충돌을 야기하고 있다.

#### 4.1.1  UDP에 대한 도전

역사적으로 많은 기업 및 통신사 네트워크는 UDP 트래픽에 대해 엄격한 정책을 적용해왔다. UDP는 주로 DNS와 같은 단발성 요청-응답이나, 신뢰성이 덜 중요한 일부 실시간 트래픽에 사용된다는 인식 때문에, 방화벽에서 UDP 트래픽을 완전히 차단하거나, 낮은 우선순위로 처리하거나, 세션 타임아웃을 매우 짧게 설정하는 경우가 많다.14 QUIC은 장시간 지속되는 연결을 UDP 위에서 구현하므로, 이러한 네트워크 정책과 충돌하여 연결이 불안정해지거나 아예 수립되지 못하는 문제가 발생할 수 있다.

#### 4.1.2  암호화의 양날의 검

QUIC의 '모든 것을 암호화한다'는 설계 원칙은 중간 장비에 의한 프로토콜 경직화를 막는 효과적인 전략이다.14 중간 장비가 패킷의 내용을 이해할 수 없으면, 임의로 패킷을 수정하거나 차단할 수 없기 때문이다. 그러나 이는 네트워크 보안 및 운영팀에게는 심각한 도전 과제가 된다. 기존의 방화벽, 침입 탐지 시스템(IDS/IPS), 데이터 유출 방지(DLP) 솔루션 등은 암호화되지 않은 전송 계층 헤더(예: TCP 시퀀스 번호, 포트)를 분석하여 트래픽을 모니터링하고, 위협을 탐지하며, 보안 정책을 적용해왔다.4 QUIC은 이러한 정보를 모두 암호화하기 때문에, 기존의 심층 패킷 검사(DPI) 기반 보안 장비들은 사실상 무력화된다.60

#### 4.1.3  'QUIC 차단'의 딜레마

이러한 가시성 상실 문제에 대한 네트워크 관리자들의 가장 일반적인 초기 대응은 방화벽에서 QUIC이 사용하는 UDP 443번 포트를 차단하는 것이었다.4 이 경우, QUIC 연결에 실패한 클라이언트(예: 웹 브라우저)는 자동으로 기존의 TCP 기반 HTTP/2 및 TLS로 폴백(fallback)하게 된다. 이 방법은 보안 장비의 가시성을 다시 확보할 수 있다는 장점이 있지만, QUIC이 제공하는 모든 성능 및 보안상의 이점을 포기하는 결과를 초래한다.60 이는 프로토콜의 진화와 기존 보안 패러다임 간의 근본적인 충돌을 보여주는 사례로, 장기적으로는 QUIC 트래픽을 이해하고 정책을 적용할 수 있는 새로운 보안 아키텍처로의 전환이 필요함을 시사한다.

### 4.2  0-RTT의 난제: 성능과 보안의 균형

0-RTT 핸드셰이크는 지연 시간을 획기적으로 줄여주지만, 심각한 보안 위험을 내포하고 있어 신중한 설계와 적용이 필수적이다.

#### 4.2.1  재전송 공격(Replay Attack)의 위험

0-RTT의 가장 큰 보안 취약점은 재전송 공격에 있다. 0-RTT 데이터는 이전 세션에서 파생된 사전 공유 키(PSK)로 암호화된다. 경로상의 공격자는 이 암호화된 0-RTT 패킷의 내용을 해독할 수는 없지만, 패킷을 그대로 복사하여 서버에 여러 번 재전송할 수 있다.34 만약 이 0-RTT 데이터에 비멱등성(non-idempotent) 작업, 즉 반복 실행 시 부작용을 일으키는 작업(예: 온라인 결제, 게시물 작성)이 포함되어 있다면, 사용자의 의도와 달리 해당 작업이 여러 번 실행되어 심각한 문제를 야기할 수 있다.36

#### 4.2.2  순방향 비밀성(Forward Secrecy)의 부재

0-RTT로 전송되는 데이터는 순방향 비밀성을 보장하지 않는다.34 만약 서버가 세션 재개를 위해 사용하는 장기 키가 유출될 경우, 공격자는 과거에 수집해 둔 0-RTT 패킷들을 모두 해독할 수 있다. 단, 핸드셰이크가 완전히 완료된 후(1-RTT) 교환되는 데이터는 새로운 임시 키를 사용하므로 순방향 비밀성이 보장된다.

#### 4.2.3  표준화된 완화 메커니즘 (RFC 8470)

이러한 위험을 완화하기 위해, 애플리케이션 계층에서 다음과 같은 방어 메커니즘을 반드시 구현해야 한다:

- **멱등성(Idempotent) 데이터만 전송**: 애플리케이션은 0-RTT를 통해 여러 번 실행되어도 안전한 멱등성 요청(예: HTTP GET)만을 전송하도록 설계되어야 한다.36

- **`Early-Data` 헤더와 `425 (Too Early)` 상태 코드**: 클라이언트가 0-RTT로 요청을 보내면, 서버나 중간 프록시(예: CDN)는 해당 HTTP 요청에 `Early-Data: 1` 헤더를 추가하여 이것이 0-RTT 요청임을 명시할 수 있다.35 애플리케이션 서버는 이 헤더를 확인하고, 만약 해당 요청이 비멱등성 작업이라면 

  `425 (Too Early)` 상태 코드로 응답해야 한다. 이 응답을 받은 클라이언트는 1-RTT 핸드셰이크가 완전히 완료된 후에 안전하게 요청을 재시도하게 된다.35

결론적으로 0-RTT는 강력한 성능 향상 도구이지만, 그 이점을 안전하게 활용하기 위해서는 애플리케이션 수준에서의 세심한 설계와 위험 관리가 필수적이다.

### 4.3  도입 및 배포 현황 (2025년 초 기준)

QUIC과 HTTP/3는 실험 단계를 지나 인터넷의 주류 기술로 자리 잡아가고 있다. 그 도입 현황은 주요 기술 기업들의 적극적인 참여와 광범위한 생태계 지원에 힘입어 꾸준히 확대되고 있다.

#### 4.3.1  트래픽 통계

웹 기술 통계 사이트인 W3Techs에 따르면, 2025년 초 현재 전체 웹사이트의 약 35.2%가 HTTP/3를 사용하고 있다.62 이는 QUIC이 인터넷 트래픽의 상당 부분을 차지하게 되었음을 의미한다. Cloudflare Radar의 데이터에 따르면, 2023년 중반에 이미 HTTP/3 트래픽 점유율이 40% 수준에서 안정화되는 모습을 보였으며 63, 2025년 2월 기준으로는 전 세계 웹사이트의 약 31%가 HTTP/3를 사용하고 있다는 통계도 있다.63 이러한 수치는 시간이 지남에 따라 변동이 있지만, 전반적인 상승 추세는 명확하다.

#### 4.3.2  주요 도입 주체

QUIC의 확산은 인터넷 트래픽의 대부분을 차지하는 소수의 거대 기술 기업들이 주도하고 있다. 구글은 자사의 모든 서비스(검색, 유튜브, 크롬 브라우저 등)에 QUIC을 전면적으로 도입하여 프로토콜의 실효성을 입증했으며 10, 메타(페이스북) 또한 자사 트래픽의 75% 이상을 QUIC으로 처리한다고 밝힌 바 있다.63 또한 Cloudflare, Fastly, Akamai와 같은 주요 CDN(Content Delivery Network) 업체들이 자사 서비스를 통해 수많은 웹사이트에 손쉽게 HTTP/3를 활성화할 수 있는 환경을 제공하면서 QUIC의 저변 확대에 결정적인 역할을 하고 있다.28

#### 4.3.3  브라우저 및 서버 지원

QUIC 생태계의 성숙도는 클라이언트와 서버 양측의 폭넓은 지원에서 확인할 수 있다. 현재 모든 주요 웹 브라우저(크롬, 파이어폭스, 사파리, 엣지)가 QUIC 및 HTTP/3를 기본적으로 지원하고 있다.10 서버 측에서도 Nginx, LiteSpeed, Caddy 등 널리 사용되는 웹 서버 소프트웨어와 Envoy와 같은 프록시가 HTTP/3를 지원하여, 개발자들이 실제로 QUIC 기반 서비스를 구축하고 배포할 수 있는 완전한 기술 스택이 마련되었다.23 이러한 광범위한 생태계 지원은 QUIC이 더 이상 실험적인 기술이 아닌, 인터넷의 표준 인프라로 자리 잡았음을 보여준다.

## 5.  확장되는 QUIC 생태계와 미래 전망

QUIC은 단순히 HTTP를 위한 더 나은 전송 프로토콜을 넘어, 그 자체로 하나의 강력하고 유연한 플랫폼으로 진화하고 있다. QUIC이 제공하는 저지연, 보안, 이동성과 같은 핵심적인 추상화 기능들은 HTTP를 넘어선 다양한 인터넷 서비스의 기반 기술로 채택되고 있다. 제 5부에서는 QUIC을 기반으로 구축되고 있는 새로운 프로토콜 생태계를 조망하고, 현재 IETF에서 논의 중인 미래 프로토콜 확장 기능들을 통해 QUIC이 나아갈 방향을 예측한다.

### 5.1  현대 웹의 기반: HTTP/3

HTTP/3는 QUIC의 가장 중요하고 첫 번째인 '킬러 애플리케이션'이다. HTTP/3는 HTTP/2와 비교하여 애플리케이션 계층의 의미론(semantics), 즉 헤더, 메소드, 상태 코드 등에서는 큰 변화가 없다.39 본질적인 차이는 전송 계층을 TCP에서 QUIC으로 완전히 교체한 데 있다.5 즉, HTTP/3는 기존 HTTP의 의미론을 QUIC의 스트림과 다양한 기능 위에 매핑(mapping)한 프로토콜이다.66

HTTP/3의 주된 목표는 QUIC이 제공하는 모든 성능상의 이점을 그대로 상속받는 것이다. 그중에서도 가장 핵심적인 것은 전송 계층 HOL 블로킹의 완전한 해결이다. 이로써 HTTP/3는 불안정한 네트워크 환경에서도 빠르고 안정적인 웹 경험을 제공하는 차세대 웹 표준으로 자리매김하고 있다.

### 5.2  HTTP를 넘어서: QUIC의 영역 확장

QUIC의 강력한 기능들은 HTTP 외의 다른 프로토콜에도 매력적인 기반을 제공한다. 특히 DNS와 VPN 분야에서 QUIC을 활용하려는 움직임이 활발하게 진행되고 있다.

#### 5.2.1  DNS over QUIC (DoQ)

DNS over QUIC(DoQ)는 DNS 쿼리를 암호화하여 프라이버시를 보호하기 위한 새로운 프로토콜로, IETF RFC 9250으로 표준화되었다.68 DoQ는 일반적으로 UDP 853번 포트를 사용하며, 기존의 암호화 DNS 프로토콜인 DoT(DNS over TLS)나 DoH(DNS over HTTPS)에 비해 여러 장점을 가진다.

- **프로토콜 및 작동 원리**: DoQ는 DNS 쿼리와 응답을 QUIC 연결을 통해 전송한다. 클라이언트는 DoQ 해석기(resolver)와 QUIC 연결을 설정하고, 이 암호화된 터널을 통해 DNS 메시지를 교환한다.70
- **장점**: DoQ는 DoT와 DoH가 제공하는 암호화 및 프라이버시 보호 기능을 모두 제공하면서도, QUIC의 0/1-RTT 핸드셰이크 덕분에 연결 설정이 훨씬 빠르다.70 또한, HOL 블로킹이 없어 패킷 손실에 더 강건하며, 연결 마이그레이션 기능은 모바일 환경에서 DNS 해석의 안정성을 높여준다.69 특히 DoH와 비교했을 때, DoQ는 HTTP 계층을 사용하지 않으므로 HTTP 쿠키나 사용자 에이전트(User-Agent)와 같은 불필요한 메타데이터가 유출될 위험이 없어, DNS 프라이버시 보호라는 본연의 목적에 더 충실한 프로토콜로 평가받는다.72

#### 5.2.2  MASQUE (Multiplexed Application Substrate over QUIC Encryption)

MASQUE는 IETF 워킹 그룹의 이름이자, QUIC을 기반으로 차세대 프록시 및 VPN과 유사한 서비스를 구축하기 위한 프레임워크를 지칭한다.73

- **목표**: MASQUE의 핵심 목표는 HTTP/3 연결을 일종의 다목적 터널로 사용하여, 그 안에 TCP나 UDP 트래픽, 나아가 IP 패킷 전체를 캡슐화하여 전송하는 것이다.75 이를 통해 엔드-투-엔드 연결이 불가능한 환경에서 통신을 가능하게 하거나, VPN처럼 추가적인 암호화 계층을 제공하여 보안과 프라이버시를 강화한다.
- **메커니즘**: MASQUE는 HTTP의 `CONNECT` 메소드를 확장한 `CONNECT-UDP` 및 `CONNECT-IP`라는 새로운 요청 방식을 정의한다.74 클라이언트는 이 요청을 프록시 서버에 보내 터널을 생성하고, 이후의 모든 트래픽은 이 터널을 통해 전송된다. 이 과정에서 모든 데이터는 QUIC의 종단 간 암호화, 통합된 혼잡 제어, 그리고 원활한 연결 마이그레이션 기능의 혜택을 받게 되어, 기존의 VPN 기술보다 특히 모바일 환경에서 더 효율적이고 안정적인 성능을 제공할 수 있다.4

### 5.3  다음 지평: 미래의 프로토콜 확장

QUIC은 현재의 기능에 머무르지 않고, 지속적인 발전을 위한 확장 기능을 활발히 논의하고 있다. 이는 QUIC이 설계 초기부터 목표로 했던 '진화 가능한 프로토콜'로서의 면모를 보여준다.

#### 5.3.1  멀티패스 QUIC (MP-QUIC)

멀티패스 QUIC(MP-QUIC)은 현재 IETF에서 표준화가 진행 중인 중요한 확장 기능이다.77

- **개념**: MP-QUIC은 단일 QUIC 연결이 여러 네트워크 경로(예: Wi-Fi와 셀룰러 데이터)를 **동시에** 사용하여 데이터를 전송할 수 있도록 허용한다.79 이는 하나의 경로만 활성화되는 기존의 연결 마이그레이션을 넘어선 개념이다.
- **장점**: 두 개 이상의 경로를 동시에 활용함으로써, 대역폭을 결합하여 전송 처리량을 극대화할 수 있다. 또한, 한쪽 경로에 장애가 발생하더라도 다른 경로를 통해 통신이 중단 없이 즉시 이어지는 '무중단 장애 극복(Hitless Failover)'이 가능해져, 연결 안정성을 최고 수준으로 끌어올릴 수 있다. 이는 과거 MPTCP(Multipath TCP)가 시도했던 목표를 더욱 효율적이고 배포 가능한 방식으로 구현하려는 시도이다.42

#### 5.3.2  스핀 비트 (Spin Bit)

스핀 비트는 QUIC의 강력한 암호화 정책과 네트워크 운영자들의 가시성 확보 요구 사이의 절충안으로 제안된 기능이다.

- **목적**: QUIC은 거의 모든 정보를 암호화하기 때문에 네트워크 중간에서 RTT와 같은 기본적인 성능 지표를 측정하기가 매우 어렵다. 스핀 비트는 이 문제를 해결하기 위해 의도적으로 노출된 신호이다.82
- **작동 원리**: 스핀 비트는 QUIC의 짧은 헤더(Short Header)에 포함된 암호화되지 않은 단일 비트이다. 이 비트는 클라이언트와 서버 간의 상호작용을 통해 매 RTT마다 한 번씩 값이 뒤집히도록(0에서 1로, 또는 1에서 0으로) 설계되었다.82 네트워크 경로상의 관찰자는 패킷을 해독하지 않고도 이 비트의 변화 간격을 측정함으로써 종단 간 RTT를 추정할 수 있다.83 이는 개인정보를 침해하지 않으면서도 네트워크 모니터링 및 진단에 필수적인 정보를 제공하는 영리한 방법이다.

이러한 확장들은 QUIC이 단순한 HTTP 전송 프로토콜을 넘어, 인터넷의 다양한 요구사항을 수용할 수 있는 범용적인 전송 플랫폼으로 진화하고 있음을 명확히 보여준다. DoQ와 MASQUE는 QUIC의 핵심 추상화(저지연 암호화 터널, 연결 ID)가 얼마나 강력하고 일반적인 빌딩 블록인지를 증명한다. 이는 개발자들이 새로운 네트워크 서비스를 구상할 때 'TCP 위에서'나 'UDP 위에서'가 아닌, 'QUIC 위에서' 구축하는 것을 당연하게 여기는 시대로의 전환을 예고한다. QUIC은 차세대 인터넷 혁신을 위한 새로운 '플랫폼 프로토콜'로서의 역할을 수행하고 있는 것이다.

## 6.  종합 및 전략적 제언

본 보고서는 QUIC 프로토콜의 등장 배경, 핵심 아키텍처, 성능 특성, 그리고 확장 생태계에 이르기까지 다각적인 고찰을 수행했다. 제 6부에서는 지금까지의 분석을 종합하여 현대 네트워킹 환경에서 QUIC이 차지하는 역할과 의미를 재정의하고, 이를 바탕으로 인터넷 생태계의 주요 구성원들을 위한 전략적인 제언을 제시함으로써 보고서를 마무리한다.

### 6.1  현대 네트워킹에서 QUIC의 역할에 대한 종합적 시각

QUIC은 TCP의 점진적인 개선이 아닌, 인터넷 전송 패러다임의 근본적인 전환을 상징한다. 그 핵심적인 역할과 의미는 다음과 같이 종합될 수 있다.

- **사용자 중심 최적화의 산물**: QUIC의 설계 철학은 통제된 이상적인 네트워크에서의 최대 처리량보다는, 예측 불가능하고 손실이 잦은 실제 인터넷 환경, 특히 모바일 사용자의 체감 성능을 최우선으로 고려한 결과물이다. 이는 서버 측의 CPU 비용 증가라는 명백한 트레이드오프를 감수하고서라도 지연 시간 단축과 연결 안정성 향상이라는 사용자 경험의 가치를 선택한 전략적 결정이다.
- **핵심적인 기술적 긴장 관계**: QUIC은 여러 상충하는 가치 사이의 균형을 맞추려는 시도이다. **성능 향상**은 **CPU 비용 증가**를, **프로토콜 진화 가능성**은 **네트워크 가시성 상실**을, 그리고 **0-RTT의 속도**는 **재전송 공격이라는 보안 위험**을 동반한다. 따라서 QUIC을 성공적으로 도입하고 운영하기 위해서는 이러한 트레이드오프를 명확히 이해하고 관리하는 것이 필수적이다.
- **프로토콜에서 플랫폼으로의 진화**: HTTP/3를 시작으로 DoQ, MASQUE 등 다양한 애플리케이션이 QUIC을 기반으로 개발되면서, QUIC은 단일 프로토콜을 넘어 새로운 네트워크 서비스를 구축하기 위한 다목적 플랫폼으로 진화하고 있다. 이는 QUIC의 핵심 추상화 기능들이 특정 애플리케이션에 국한되지 않는 범용성과 강력함을 가지고 있음을 증명한다.

결론적으로 QUIC은 수십 년간 경직되어 있던 인터넷 전송 계층에 혁신과 유연성을 불어넣는 핵심 동력이다. 이는 인터넷의 중심축을 코어 네트워크와 운영체제에서 애플리케이션과 서비스 제공업체로 이동시키는 중요한 변화를 이끌고 있다.

### 6.2  배포, 최적화 및 향후 연구를 위한 제언

QUIC의 성공적인 안착과 발전을 위해 인터넷 생태계의 각 주체는 다음과 같은 전략적 방향을 고려해야 한다.

#### 6.2.1  네트워크 설계자 및 보안 전문가를 위한 제언

- **'차단 후 폴백'에서 '적응 및 수용'으로의 전환**: QUIC 트래픽을 무조건 차단하는 소극적인 대응에서 벗어나, 이를 수용하고 관리할 수 있는 새로운 보안 아키텍처로의 전환을 모색해야 한다. 심층 패킷 검사에 의존하는 기존 방식 대신, 엔드포인트 보안을 강화하거나 MASQUE와 같은 협력적 프록시 모델을 도입하는 방안을 검토해야 한다.
- **새로운 가시성 도구 도입**: QUIC의 암호화 정책을 존중하면서도 네트워크 상태를 파악할 수 있는 새로운 모니터링 도구에 투자해야 한다. 스핀 비트(Spin Bit)와 같은 명시적인 측정 신호를 활용하는 솔루션의 도입을 적극적으로 고려하여, 암호화된 환경에서도 네트워크 성능 진단 및 문제 해결 능력을 유지해야 한다.

#### 6.2.2  애플리케이션 개발자 및 서비스 운영자를 위한 제언

- **QUIC의 전략적 채택**: 특히 지연 시간에 민감하고 상호작용이 많은 애플리케이션, 그리고 모바일 사용자를 주 대상으로 하는 서비스에 QUIC(HTTP/3) 도입을 우선적으로 고려해야 한다.
- **워크로드별 성능 벤치마킹**: 자신의 서비스 워크로드에 특화된 환경에서 QUIC 도입에 따른 CPU 및 처리량 변화를 면밀히 벤치마킹해야 한다. 이를 통해 서버 용량 증설이나 아키텍처 변경 등 필요한 자원 계획을 수립해야 한다.
- **0-RTT의 안전한 구현**: 0-RTT 기능을 활성화할 경우, 재전송 공격을 방지하기 위해 애플리케이션 수준에서 멱등성을 보장하는 강력한 안전장치를 반드시 구현해야 한다. `Early-Data` 헤더와 `425 (Too Early)` 상태 코드를 활용하는 표준적인 방어 메커니즘을 적용하는 것이 필수적이다.

#### 6.2.3  연구자 및 프로토콜 엔지니어를 위한 제언

- **하드웨어 가속 표준화**: QUIC의 CPU 오버헤드를 근본적으로 해결하기 위해, NIC에서의 효율적인 QUIC 하드웨어 오프로딩 기술을 개발하고 표준화하는 연구에 집중해야 한다.
- **새로운 혼잡 제어 알고리즘 개발**: 사용자 공간이라는 유연성을 최대한 활용하여, 실시간 미디어, 대용량 데이터 전송, IoT 통신 등 다양한 애플리케이션 시나리오에 최적화된 새로운 혼잡 제어 알고리즘을 설계하고 평가해야 한다.
- **확장 프로토콜의 심화 연구**: MASQUE의 대규모 배포가 가져올 수 있는 보안 및 프라이버시 문제, 그리고 MP-QUIC의 스케줄링 및 혼잡 제어 알고리즘 등, 현재 진행 중인 확장 프로토콜들의 기술적 과제에 대한 심도 있는 연구를 지속해야 한다.

QUIC은 이미 인터넷의 현재를 바꾸고 있으며, 그 잠재력은 이제 막 펼쳐지기 시작했다. 위에 제시된 전략적 제언들을 통해 생태계의 모든 구성원이 협력하고 발전해 나간다면, QUIC은 더 빠르고, 더 안전하며, 더 유연한 미래 인터넷을 구축하는 핵심적인 초석이 될 것이다.

#### **참고 자료**

1. Deep Dive into QUIC: The Transport Protocol Powering Solana's Firedancer - Vic Genin, accessed July 28, 2025, https://deeprnd.medium.com/deep-dive-into-quic-the-transport-protocol-powering-solanas-firedancer-7d9e28f37ba8
2. TLS 1.2 vs. 1.3-Handshake, Performance, and Other Improvements - Catchpoint, accessed July 28, 2025, https://www.catchpoint.com/http2-vs-http3/tls1-2-vs-1-3
3. QUIC 프로토콜, accessed July 28, 2025, https://morjun.github.io/quic
4. QUIC: The Secure Communication Protocol Shaping the Internet's ..., accessed July 28, 2025, https://www.zscaler.com/blogs/product-insights/quic-secure-communication-protocol-shaping-future-of-internet
5. HTTP/3 vs. HTTP/2 - A detailed comparison - Catchpoint, accessed July 28, 2025, https://www.catchpoint.com/http3-vs-http2
6. Performance Comparison of HTTP/1.1, HTTP/2, and QUIC - Stony Brook Computer Science, accessed July 28, 2025, https://www3.cs.stonybrook.edu/~arunab/course/2017-1.pdf
7. 차세대 네트워크 프로토콜(QUIC, HTTP/3) - C's Shelter - 티스토리, accessed July 28, 2025, https://gnuhcjh.tistory.com/142
8. TCP HOL(ko/head of line) 블로킹 | HTTP/3 explained - README, accessed July 28, 2025, https://http3-explained.haxx.se/ko/why-quic/why-tcphol
9. QUIC 프로토콜 | 구글 또 너야?, accessed July 28, 2025, [https://medium.com/rate-labs/quic-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C-%EA%B5%AC%EA%B8%80-%EB%98%90-%EB%84%88%EC%95%BC-932befde91a1](https://medium.com/rate-labs/quic-프로토콜-구글-또-너야-932befde91a1)
10. QUIC - 위키백과, 우리 모두의 백과사전, accessed July 28, 2025, https://ko.wikipedia.org/wiki/QUIC
11. How does QUIC multiplexing differ from that of HTTP/2 - Stack Overflow, accessed July 28, 2025, https://stackoverflow.com/questions/54653228/how-does-quic-multiplexing-differ-from-that-of-http-2
12. Why is there a general hostility to QUIC by network engineers? - Reddit, accessed July 28, 2025, https://www.reddit.com/r/networking/comments/148qz1f/why_is_there_a_general_hostility_to_quic_by/
13. QUIC vs. TCP-Development and Monitoring Guide - Catchpoint, accessed July 28, 2025, https://www.catchpoint.com/http2-vs-http3/quic-vs-tcp
14. QUIC as a solution to protocol ossification - LWN.net, accessed July 28, 2025, https://lwn.net/Articles/745590/
15. Comparing Quic and TLS for Enterprise Security - Lightyear.ai, accessed July 28, 2025, https://lightyear.ai/tips/quic-versus-tls
16. QUIC vs. Middleboxes - FOSDEM 2025, accessed July 28, 2025, https://fosdem.org/2025/events/attachments/fosdem-2025-6261-quic-vs-middleboxes/slides/238834/2025-fosd_5SwceA4.pdf
17. QUIC (Quick UDP Internet Connection) - 도리의 디지털라이프, accessed July 28, 2025, https://blog.skby.net/quic-quick-udp-internet-connection/
18. QUIC - Wikipedia, accessed July 28, 2025, https://en.wikipedia.org/wiki/QUIC
19. QUIC을 향한 여정(번역) - velog, accessed July 28, 2025, [https://velog.io/@wsong0101/QUIC%EC%9D%84-%ED%96%A5%ED%95%9C-%EC%97%AC%EC%A0%95%EB%B2%88%EC%97%AD](https://velog.io/@wsong0101/QUIC을-향한-여정번역)
20. QUIC is now RFC 9000 - Fastly | Fastly, accessed July 28, 2025, https://www.fastly.com/blog/quic-is-now-rfc-9000
21. HTTP/3 - Wikipedia, accessed July 28, 2025, https://en.wikipedia.org/wiki/HTTP/3
22. QUIC 버전 1의 RFC 승인 - saturnsoft.net, accessed July 28, 2025, https://www.saturnsoft.net/network/2021/05/27/quic-rfc9000/
23. [nw] HTTP/3 QUIC - create value with tech, not tool - 티스토리, accessed July 28, 2025, https://gettingconnected.tistory.com/414
24. QUIC - Google Edge Network, accessed July 28, 2025, https://peering.google.com/quicfaq
25. The Road to QUIC - The Cloudflare Blog, accessed July 28, 2025, https://blog.cloudflare.com/the-road-to-quic/
26. IETF QUIC v1 Design, accessed July 28, 2025, https://www.cse.wustl.edu/~jain/cse570-21/ftp/quic/index.html
27. QUIC vs TCP: Which is Better? - Fastly, accessed July 28, 2025, https://www.fastly.com/blog/measuring-quic-vs-tcp-computational-efficiency
28. QUIC 프로토콜의 주요 특징과 장점 : HTTP/3의 핵심 기술과 실제 적용 사례 - 로미는내고양이, accessed July 28, 2025, [https://romyismycat.tistory.com/entry/QUIC-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C%EC%9D%98-%EC%A3%BC%EC%9A%94-%ED%8A%B9%EC%A7%95%EA%B3%BC-%EC%9E%A5%EC%A0%90-HTTP3%EC%9D%98-%ED%95%B5%EC%8B%AC-%EA%B8%B0%EC%88%A0%EA%B3%BC-%EC%8B%A4%EC%A0%9C-%EC%A0%81%EC%9A%A9-%EC%82%AC%EB%A1%80](https://romyismycat.tistory.com/entry/QUIC-프로토콜의-주요-특징과-장점-HTTP3의-핵심-기술과-실제-적용-사례)
29. QUIC is not Quick Enough over Fast Internet : r/programming - Reddit, accessed July 28, 2025, https://www.reddit.com/r/programming/comments/1g7vv66/quic_is_not_quick_enough_over_fast_internet/
30. Comparing TCP and QUIC - Hacker News, accessed July 28, 2025, https://news.ycombinator.com/item?id=33431669
31. HTTP/3와 QUIC 요약정리 - Easy Understanding - 티스토리, accessed July 28, 2025, https://appleg1226.tistory.com/61
32. QUIC(Quick UDP Internet Connections)에 대해 알아보겠습니다. - Feccle 의 IT자료 모음, accessed July 28, 2025, https://feccle.tistory.com/m/428
33. QUIC vs TCP+TLS - and why QUIC is not the next big thing | by Codavel - Medium, accessed July 28, 2025, https://medium.com/codavel-blog/quic-vs-tcp-tls-and-why-quic-is-not-the-next-big-thing-d4ef59143efd
34. Lightning-Fast Requests with Early Data - Akamai, accessed July 28, 2025, https://www.akamai.com/blog/edge/lightning-fast-requests-with-early-data
35. QUIC 0-RTT 재시작으로 더 빨리 연결하기 - The Cloudflare Blog, accessed July 28, 2025, https://blog.cloudflare.com/ko-kr/even-faster-connection-establishment-with-quic-0-rtt-resumption/
36. Even faster connection establishment with QUIC 0-RTT resumption - The Cloudflare Blog, accessed July 28, 2025, https://blog.cloudflare.com/even-faster-connection-establishment-with-quic-0-rtt-resumption/
37. [네트워크] QUIC 프로토콜, accessed July 28, 2025, https://globalman96.tistory.com/14
38. RFC 9000 - QUIC: A UDP-Based Multiplexed and Secure Transport - Datatracker - IETF, accessed July 28, 2025, https://datatracker.ietf.org/doc/rfc9000/
39. Comparison with HTTP/2 | HTTP/3 explained - README, accessed July 28, 2025, https://http3-explained.haxx.se/en/h3/h3-h2
40. [Network] QUIC(빠른 UDP 인터넷 연결) 개념 및 특징 - 호우동의 개발일지, accessed July 28, 2025, https://howudong.tistory.com/407
41. QUIC와 HTTP/3는 무엇인가? | F5, accessed July 28, 2025, https://www.f5.com/ko_kr/glossary/quic-http3
42. Connection Migration – quic-go docs, accessed July 28, 2025, https://quic-go.net/docs/quic/connection-migration/
43. An Analysis of QUIC Connection Migration in the Wild - arXiv, accessed July 28, 2025, https://arxiv.org/html/2410.06066v1
44. [논문 리뷰] An Analysis of QUIC Connection Migration in the Wild - Moonlight, accessed July 28, 2025, https://www.themoonlight.io/ko/review/an-analysis-of-quic-connection-migration-in-the-wild
45. RFC 9001: Using TLS to Secure QUIC, accessed July 28, 2025, https://www.rfc-editor.org/rfc/rfc9001.html
46. Google QUIC and Its Impact on Network Security - NetQuest Corporation, accessed July 28, 2025, https://netquestcorp.com/google-quic-and-network-security/
47. Improving the BBR congestion control algorithm for QUIC - DiVA portal, accessed July 28, 2025, https://www.diva-portal.org/smash/get/diva2:1767543/FULLTEXT01.pdf
48. QUIC with Deep Reinforcement Learning Congestion Control, accessed July 28, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9955954/
49. Congestion Control – quic-go docs, accessed July 28, 2025, https://quic-go.net/docs/quic/congestion-control/
50. Congestion Control Enhancements for QUIC in NGINX, accessed July 28, 2025, https://blog.nginx.org/blog/congestion-control-enhancements-for-quic-in-nginx
51. Evaluating QUIC Performance over Web, Cloud Storage and Video Workloads - Vaibhav Bajpai, accessed July 28, 2025, https://vaibhavbajpai.com/documents/papers/proceedings/quic-tnsm-2022.pdf
52. QUIC vs TCP: "The Ultimate Benchmark for Modern Network Performance" | by Abhi Patel, accessed July 28, 2025, https://medium.com/@abhi.patel.7172/quic-vs-tcp-the-ultimate-benchmark-for-modern-network-performance-22be6e5af66d
53. ELI5: if QUIC runs on top of udp then what about packet loss ( forget about streaming what abut webpages which use http3 how does packet loss handled here) - Reddit, accessed July 28, 2025, https://www.reddit.com/r/explainlikeimfive/comments/1lykqi5/eli5_if_quic_runs_on_top_of_udp_then_what_about/
54. QUIC이란 무엇입니까? HTTP/3를 어떻게 향상합니까? - 씨디네트웍스 - CDNetworks, accessed July 28, 2025, https://www.cdnetworks.com/ko/media-delivery-blog/what-is-quic/
55. QUIC is not Quick Enough over Fast Internet - arXiv, accessed July 28, 2025, https://arxiv.org/html/2310.09423v2
56. QUIC costs something like 2x to 4x as much CPU time to serve large files or stre... | Hacker News, accessed July 28, 2025, https://news.ycombinator.com/item?id=19476439
57. Making QUIC Quicker With NIC Offload - Lars Eggert, accessed July 28, 2025, https://eggert.org/papers/epiq20-final15.pdf
58. QUIC CPU Performance, accessed July 28, 2025, [https://conferences.sigcomm.org/sigcomm/2020/files/slides/epiq/0%20QUIC%20and%20HTTP_3%20CPU%20Performance.pdf](https://conferences.sigcomm.org/sigcomm/2020/files/slides/epiq/0 QUIC and HTTP_3 CPU Performance.pdf)
59. Optimizing QUIC performance - Private Octopus, Inc, accessed July 28, 2025, https://www.privateoctopus.com/2023/12/12/quic-performance.html
60. Is this sustainable though? Won't the middle boxes adapt? - Hacker News, accessed July 28, 2025, https://news.ycombinator.com/item?id=25970429
61. QUIC Will Eat the Internet - F5 Networks, accessed July 28, 2025, https://www.f5.com/company/blog/quic-will-eat-the-internet
62. Usage Statistics of HTTP/3 for Websites, July 2025 - W3Techs, accessed July 28, 2025, https://w3techs.com/technologies/details/ce-http3
63. An update on QUIC Adoption and traffic levels - CellStream, Inc., accessed July 28, 2025, https://www.cellstream.com/2025/02/14/an-update-on-quic-adoption-and-traffic-levels/
64. QUIC 프로토콜 개념과 활용: 차세대 네트워크 통신의 혁신 - 기피말고깊이, accessed July 28, 2025, https://notavoid.tistory.com/195
65. Radar Trends to Watch: July 2025 - O'Reilly Media, accessed July 28, 2025, https://www.oreilly.com/radar/radar-trends-to-watch-july-2025/
66. Deep Dive into HTTP/3: The Future of the Web - DEV Community, accessed July 28, 2025, https://dev.to/devcorner/deep-dive-into-http3-the-future-of-the-web-kjg
67. Hypertext Transfer Protocol (HTTP) over QUIC, accessed July 28, 2025, https://greenbytes.de/tech/webdav/draft-ietf-quic-http-14.html
68. New DNS over QUIC protocol makes encrypted DNS traffic faster and more efficient - SIDN, accessed July 28, 2025, https://www.sidn.nl/en/news-and-blogs/new-dns-over-quic-protocol-makes-encrypted-dns-traffic-faster-and-more-efficient
69. DNS-over-QUIC is now officially a proposed standard, accessed July 28, 2025, https://adguard-dns.io/en/blog/dns-over-quic-official-standard.html
70. DNS over QUIC (DoQ): A complete guide - NordVPN, accessed July 28, 2025, https://nordvpn.com/blog/dns-over-quic/
71. DNS over QUIC (DoQ)-Working and Implementation Guide. - Catchpoint, accessed July 28, 2025, https://www.catchpoint.com/http2-vs-http3/dns-over-quic
72. What is DNS over QUIC (DoQ)? - GreenCloud - Affordable KVM and Windows VPS, accessed July 28, 2025, https://blog.greencloudvps.com/what-is-dns-over-quic-doq.php
73. MASQUE Working Group, accessed July 28, 2025, https://ietf-wg-masque.github.io/
74. Multiplexed Application Substrate over QUIC Encryption (masque), accessed July 28, 2025, https://datatracker.ietf.org/wg/masque/about/
75. Multiplexed Application Substrate over QUIC Encryption - IETF Datatracker, accessed July 28, 2025, https://datatracker.ietf.org/doc/charter-ietf-masque/01-00/
76. Unlocking QUIC's proxying potential with MASQUE - The Cloudflare Blog, accessed July 28, 2025, https://blog.cloudflare.com/unlocking-quic-proxying-potential/
77. draft-ietf-quic-multipath-12 - Datatracker, accessed July 28, 2025, https://datatracker.ietf.org/doc/html/draft-ietf-quic-multipath-12
78. draft-ietf-quic-multipath-15 - Multipath Extension for QUIC - IETF Datatracker, accessed July 28, 2025, https://datatracker.ietf.org/doc/draft-ietf-quic-multipath/
79. Multipath Extension for QUIC - IETF, accessed July 28, 2025, https://www.ietf.org/archive/id/draft-ietf-quic-multipath-05.html
80. draft-deconinck-quic-multipath-06 - IETF Datatracker, accessed July 28, 2025, https://datatracker.ietf.org/doc/html/draft-deconinck-quic-multipath-06
81. Multipath QUIC, accessed July 28, 2025, https://multipath-quic.org/
82. draft-ietf-quic-spin-exp-01 - Datatracker, accessed July 28, 2025, https://datatracker.ietf.org/doc/html/draft-ietf-quic-spin-exp
83. The QUIC Latency Spin Bit, accessed July 28, 2025, https://greenbytes.de/tech/webdav/draft-ietf-quic-spin-exp-latest.html
84. draft-trammell-quic-spin-01 - IETF Datatracker, accessed July 28, 2025, https://datatracker.ietf.org/doc/html/draft-trammell-quic-spin-01
85. QUIC performance monitoring: implementation of Spin Bit in Chromium - POLITECNICO DI TORINO, accessed July 28, 2025, https://webthesis.biblio.polito.it/26911/1/tesi.pdf
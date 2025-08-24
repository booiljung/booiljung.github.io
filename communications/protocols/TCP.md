# 전송 제어 프로토콜(TCP)
[통신 프로토콜](./index.md)


인터넷은 현대 사회의 정보 교환을 위한 핵심 인프라로 자리 잡았으며, 그 근간에는 수많은 통신 규약, 즉 프로토콜이 존재한다. 이 프로토콜들의 복잡한 상호작용을 통해 우리는 웹 페이지를 탐색하고, 이메일을 주고받으며, 파일을 전송하는 등 다양한 온라인 활동을 영위할 수 있다. 이 거대한 시스템의 심장부에서 데이터 전송의 신뢰성과 안정성을 책임지는 핵심 프로토콜이 바로 **전송 제어 프로토콜(Transmission Control Protocol, TCP)**이다.

TCP는 단순한 데이터 전송 규약을 넘어, 불확실하고 혼잡한 네트워크 환경 속에서 데이터가 유실되거나 손상되지 않고, 보낸 순서 그대로 목적지에 정확히 도달하도록 보장하는 정교한 메커니즘의 집합체이다.1 본 보고서는 TCP의 본질을 깊이 있게 탐구하고자 한다. 먼저 TCP가 네트워크 아키텍처 내에서 차지하는 위치와 그 근본적인 설계 철학을 살펴보고, 연결 설정부터 데이터 전송, 연결 해제에 이르는 전 과정을 기계적으로 분해하여 분석한다. 나아가 흐름 제어, 혼잡 제어와 같은 핵심 메커니즘의 작동 원리를 상세히 설명하며, 현대 네트워크 환경에서 TCP가 직면하는 문제점들과 이를 해결하기 위한 진화된 알고리즘들을 고찰한다.

또한, 이론적 분석에 그치지 않고 SYN 플러드(SYN Flood)나 세션 하이재킹(Session Hijacking)과 같은 보안 취약점과 그에 대한 방어 전략을 다루며, 리눅스 시스템에서의 성능 최적화 방안과 와이어샤크(Wireshark)를 이용한 실제 트래픽 분석 방법을 제시하여 실용적인 관점을 제공한다. 마지막으로, TCP의 한계를 극복하기 위해 등장한 QUIC, MPTCP와 같은 차세대 프로토콜과 L4S, 프로그래머블 네트워크(P4, eBPF) 등 미래 전송 기술의 동향을 조망함으로써, TCP가 걸어온 길과 앞으로 나아갈 방향을 종합적으로 제시하는 것을 목표로 한다.


이 장에서는 TCP를 이해하기 위한 근본적인 맥락을 설정한다. TCP의 목적과 핵심 설계 철학을 정의하고, 현대 네트워크의 계층적 아키텍처 내에서 TCP가 수행하는 정확한 역할을 규명한다. 거시적인 개념적 관점에서 시작하여 프로토콜의 기본 데이터 단위인 TCP 세그먼트의 상세한 구조까지 분석한다.




네트워크 통신은 복잡한 과정을 여러 단계로 나눈 계층적 모델을 통해 이해할 수 있다. 현대 인터넷의 사실상 표준인 **TCP/IP 모델**에서 TCP는 **전송 계층(Transport Layer)**에 속한다.3 이 계층은 애플리케이션 계층(Application Layer)과 인터넷 계층(Internet Layer) 사이에 위치하며, 서로 다른 호스트에서 실행되는 애플리케이션 프로세스 간의 논리적 통신 채널을 제공하는 역할을 한다.5

보다 개념적이고 상세한 프레임워크를 제공하는 **OSI 7계층 모델**에서도 TCP의 기능은 동일하게 **전송 계층(Layer 4)**에 해당한다.8 비록 실제 인터넷은 TCP/IP 모델을 기반으로 구현되었지만, OSI 모델은 네트워크 기능의 각 역할을 명확히 구분하여 교육 및 개념 이해에 여전히 중요한 도구로 사용된다.6


TCP는 단독으로 동작하지 않는다. 인터넷 프로토콜(Internet Protocol, IP)과 긴밀한 공생 관계를 형성하며, 이 때문에 두 프로토콜을 묶어 **TCP/IP**라고 부르는 경우가 많다.10 IP는 인터넷 계층에서 작동하며, 논리적 주소(IP 주소)를 사용하여 네트워크를 통해 패킷(데이터그램)을 목적지까지 라우팅하는 역할을 담당한다.12 하지만 IP는 '최선 노력(best-effort)' 기반의 서비스로, 패킷의 전달, 순서, 무결성을 보장하지 않는다.10 바로 이 신뢰할 수 없는 IP 서비스 위에서 TCP가 동작하여, 애플리케이션에게 신뢰성 있는 통신 채널을 구축해주는 것이다.7

이러한 IP의 단순하고 빠른 비신뢰성 패킷 전달 기능과 TCP의 복잡하고 상태를 관리하는 신뢰성 있는 종단 간 제어 기능의 분리는 우연이 아닌, 인터넷 설계의 핵심 철학인 **종단 간 원칙(End-to-End Principle)**을 직접적으로 구현한 결과이다. 이 원칙은 네트워크의 핵심 기능은 최대한 단순하게 유지하고, 통신의 복잡성과 지능은 양 끝단(end-point)의 호스트에 두어야 한다는 사상이다. 인터넷의 핵심부(IP를 실행하는 라우터)는 패킷을 목적지로 한 단계 더 가깝게 전달하는 방법만 알면 되므로 매우 단순하고 빠르게 동작할 수 있다.10 반면, 이메일 내용이 온전히 도착하도록 보장하거나 1, 파일 전송 데이터 조각을 올바른 순서로 재조립하는 10 등 모든 복잡한 작업은 양 끝단의 호스트(TCP를 실행하는 클라이언트와 서버)가 책임진다. 이 설계 덕분에 네트워크 코어는 트래픽의 종류와 무관하게 높은 확장성과 안정성을 유지할 수 있으며, 인터넷 코어 인프라의 대대적인 변경 없이도 양 끝단에서 QUIC과 같은 새로운 전송 프로토콜이나 애플리케이션의 혁신이 가능해졌다. 따라서 TCP의 역할은 단순히 기술적인 기능을 넘어, 인터넷이 전 세계적으로 확장될 수 있었던 근본적인 설계 철학을 체현하는 것이다.


TCP가 제공하는 '신뢰성'은 막연한 약속이 아니라, 다음과 같은 구체적인 핵심 원칙들의 조합으로 이루어진다.

- **연결 지향성 (Connection-Oriented):** 데이터를 교환하기 전에, 송신자와 수신자 사이에 **3-Way Handshake**라는 절차를 통해 논리적인 연결(가상 회선)을 먼저 설정한다.1 이 과정을 통해 양측이 통신할 준비가 되었음을 확인하고, 데이터 전송에 필요한 상태 정보를 동기화한다.16
- **신뢰성 있는 데이터 전송 (Reliable Data Transfer):** TCP는 데이터의 손실, 중복, 손상을 감지하고 복구하는 메커니즘을 갖추고 있다.1 이를 위해 오류 검출을 위한 체크섬(Checksum), 수신된 데이터에 대한 확인 응답(Acknowledgement, ACK), 그리고 손실되거나 손상된 패킷의 재전송 기능을 사용한다.10
- **순서 보장 (In-Order Delivery):** 네트워크를 통해 전송되는 패킷들은 서로 다른 경로를 거치면서 순서가 뒤바뀌어 도착할 수 있다. TCP는 각 데이터 조각에 순서 번호(Sequence Number)를 부여하여, 수신 측에서 이를 기반으로 원래의 순서대로 재조립한 후 애플리케이션에 전달한다.1
- **스트림 기반 통신 (Stream-Based Communication):** TCP는 애플리케이션에게 데이터를 경계 없는 연속적인 바이트 스트림(byte stream)으로 제공한다.14 애플리케이션이 보낸 데이터는 TCP에 의해 네트워크 전송에 적합한 크기의 세그먼트(segment)로 나뉘어 전송되며, 수신 측에서는 다시 하나의 스트림으로 재구성된다.22
- **전이중 통신 (Full-Duplex):** 하나의 TCP 연결을 통해 데이터가 양방향으로 동시에 전송될 수 있다.16 이는 전화 통화처럼 양측이 동시에 말할 수 있는 것과 유사하다.


전송 계층에는 TCP 외에 또 다른 주요 프로토콜인 **사용자 데이터그램 프로토콜(User Datagram Protocol, UDP)**이 존재한다. UDP는 TCP와 정반대의 특성을 가진다. 즉, 연결 설정 없이 데이터를 전송하는 비연결성(connectionless) 프로토콜이며, 전달, 순서, 오류 수정을 보장하지 않는 비신뢰성(unreliable) 서비스를 제공한다.1

이 두 프로토콜의 가장 큰 차이는 **속도와 신뢰성 간의 트레이드오프**에 있다. TCP는 핸드셰이크, ACK, 재전송, 순서 제어 등 신뢰성을 보장하기 위한 여러 메커니즘 때문에 필연적으로 오버헤드와 지연 시간이 발생하여 UDP보다 느리다.14 반면, UDP는 이러한 기능들을 모두 생략함으로써 매우 가볍고 빠르게 데이터를 전송할 수 있다.1

어떤 프로토콜을 선택할지는 전적으로 애플리케이션의 요구사항에 달려있다. 데이터의 완벽한 무결성이 필수적인 웹 브라우징(HTTP/HTTPS), 파일 전송(FTP), 이메일(SMTP)과 같은 애플리케이션은 TCP를 사용한다.10 반면, 약간의 데이터 손실을 감수하더라도 속도와 실시간성이 더 중요한 실시간 비디오/오디오 스트리밍, 온라인 게임, DNS 조회와 같은 애플리케이션은 UDP를 선호한다.1

**표 1: TCP와 UDP의 핵심 특징 비교**

| 특징               | Transmission Control Protocol (TCP)                          | User Datagram Protocol (UDP)            |
| ------------------ | ------------------------------------------------------------ | --------------------------------------- |
| **연결 설정**      | 연결 지향형 (Connection-Oriented)                            | 비연결형 (Connectionless)               |
| **신뢰성**         | 전달 및 순서 보장                                            | 최선 노력 (Best-Effort)                 |
| **오류 제어**      | 체크섬, ACK, 재전송                                          | 선택적 체크섬                           |
| **흐름 제어**      | 지원 (슬라이딩 윈도우)                                       | 미지원                                  |
| **혼잡 제어**      | 지원                                                         | 미지원                                  |
| **전송 속도**      | 상대적으로 느림 (오버헤드 존재)                              | 상대적으로 빠름 (오버헤드 적음)         |
| **헤더 크기**      | 20 ~ 60 바이트                                               | 8 바이트                                |
| **주요 사용 사례** | 웹(HTTP/HTTPS), 파일 전송(FTP), 이메일(SMTP), 원격 접속(SSH) | DNS, VoIP, 실시간 스트리밍, 온라인 게임 |

자료: 1



TCP를 통해 전송되는 데이터의 기본 단위를 **세그먼트(Segment)**라고 한다. 모든 TCP 세그먼트는 제어 정보를 담고 있는 **헤더(Header)**와 실제 애플리케이션 데이터를 담고 있는 **페이로드(Payload)**로 구성된다.24 TCP 헤더는 최소 20바이트에서 옵션 필드에 따라 최대 60바이트까지 확장될 수 있다.24 각 필드의 역할은 다음과 같다.

- **Source Port (출발지 포트, 16비트):** 데이터를 보내는 측의 애플리케이션 프로세스를 식별하는 포트 번호.
- **Destination Port (목적지 포트, 16비트):** 데이터를 받는 측의 애플리케이션 프로세스를 식별하는 포트 번호.
- **Sequence Number (순서 번호, 32비트):** 전체 바이트 스트림 내에서 현재 세그먼트에 포함된 데이터의 첫 번째 바이트가 몇 번째 바이트인지를 나타낸다. 패킷의 순서를 재조립하고 중복을 탐지하는 데 사용된다.21
- **Acknowledgment Number (확인 응답 번호, 32비트):** ACK 플래그가 1로 설정된 경우에만 유효하며, 수신을 기대하는 다음 바이트의 순서 번호를 의미한다. 즉, 이 번호 직전까지의 모든 데이터를 성공적으로 수신했음을 알린다.21
- **Data Offset (데이터 오프셋, 4비트):** TCP 헤더의 길이를 32비트 워드(4바이트) 단위로 나타낸다. 이 값을 통해 헤더가 어디까지이고 실제 데이터가 어디서부터 시작되는지 알 수 있다.24
- **Reserved (예약 필드, 3비트):** 미래를 위해 예약된 필드로, 현재는 사용되지 않으며 항상 0으로 설정되어야 한다.
- **Flags (플래그, 9비트):** 연결의 상태를 제어하고 관리하는 9개의 1비트 플래그. 주요 플래그는 다음과 같다 24:
  - **URG (Urgent):** 긴급 포인터 필드가 유효함을 나타낸다.
  - **ACK (Acknowledgment):** 확인 응답 번호 필드가 유효함을 나타낸다. 연결 수립 후 모든 패킷에 설정된다.
  - **PSH (Push):** 수신 측에 버퍼링된 데이터를 즉시 상위 애플리케이션으로 전달하도록 요청한다.
  - **RST (Reset):** 연결을 강제로 리셋한다.
  - **SYN (Synchronize):** 연결 설정을 시작할 때 순서 번호를 동기화하기 위해 사용된다.
  - **FIN (Finish):** 데이터 전송을 마치고 연결을 종료하고자 할 때 사용된다.
- **Window Size (윈도우 크기, 16비트):** 흐름 제어를 위해 사용되며, 수신자가 한 번에 받을 수 있는 데이터의 양(바이트 단위)을 송신자에게 알린다.24
- **Checksum (체크섬, 16비트):** 헤더와 데이터 전체의 오류를 검출하기 위한 값이다. 송신 측에서 계산하여 포함시키면, 수신 측에서 동일하게 계산하여 오류 여부를 확인한다.10
- **Urgent Pointer (긴급 포인터, 16비트):** URG 플래그가 설정되었을 때, 긴급한 데이터의 마지막 바이트 위치를 가리킨다.24
- **Options (옵션, 가변 길이):** TCP의 기본 기능을 확장하기 위해 사용된다. 최대 세그먼트 크기(MSS), 윈도우 스케일(Window Scale), 선택적 확인 응답(Selective Acknowledgement, SACK), 타임스탬프(Timestamp) 등의 옵션이 포함될 수 있다.24
- **Padding (패딩):** 옵션 필드 사용 후 헤더 전체의 크기를 32비트의 배수로 맞추기 위해 0으로 채워지는 부분이다.24


애플리케이션이 생성한 데이터가 네트워크를 통해 상대방에게 전달되기까지, 데이터는 TCP/IP 스택의 각 계층을 내려가면서 **캡슐화(Encapsulation)**라는 과정을 거친다. 이 과정은 각 계층이 자신의 제어 정보를 헤더 형태로 데이터에 덧붙이는 작업이다.6

1. **애플리케이션 계층:** 사용자가 보내려는 원본 데이터는 '메시지(Message)' 형태로 존재한다.6
2. **전송 계층 (TCP):** 이 메시지를 적절한 크기로 나누고, 여기에 TCP 헤더를 붙여 '세그먼트(Segment)'를 만든다.6
3. **인터넷 계층 (IP):** TCP 세그먼트에 출발지와 목적지 IP 주소 등이 포함된 IP 헤더를 붙여 '패킷(Packet)' 또는 '데이터그램(Datagram)'을 만든다.6
4. **네트워크 액세스 계층 (링크 계층):** IP 패킷에 MAC 주소와 같은 물리적 주소 정보가 담긴 이더넷 헤더와 오류 검출을 위한 트레일러를 붙여 최종적으로 '프레임(Frame)'을 완성한다.6
5. **물리 계층:** 완성된 프레임은 전기 신호나 광 신호로 변환되어 물리적인 케이블을 통해 전송된다.3

수신 측에서는 이 과정의 역순인 **역캡슐화(Decapsulation)**를 통해 각 계층의 헤더를 순서대로 제거하며 원본 데이터를 복원한다.9

이 캡슐화 과정은 마치 러시아의 전통 인형 '마트료시카'와 같다. 각 계층은 자신보다 상위 계층에서 내려온 데이터 단위를 내용물로 취급하며, 자신의 제어 정보(헤더)라는 껍데기를 씌운다. 이 계층적 추상화는 네트워크 모듈성의 핵심이다. 예를 들어, 링크 계층은 프레임 헤더의 MAC 주소만 보고 물리적 전달을 책임질 뿐, 그 안에 담긴 IP 패킷의 내용은 전혀 신경 쓰지 않는다. 마찬가지로 IP 계층은 IP 헤더의 주소만 보고 라우팅을 수행하며, 그 안의 TCP 세그먼트는 그저 투명한 데이터 덩어리로 간주한다.

이러한 모듈성 덕분에 인터넷은 수많은 종류의 물리적 네트워크 기술(이더넷, Wi-Fi, 5G 등) 위에서 원활하게 동작할 수 있다. 하부의 물리 매체가 바뀌더라도 상위 계층인 TCP나 웹 브라우저는 아무런 변경 없이 그대로 작동할 수 있다. 이처럼 각 계층이 독립적으로 자신의 기능에만 충실하도록 만드는 엄격한 캡슐화와 역캡슐화 구조는 인터넷의 유연성, 확장성, 유지보수성을 보장하는 근본적인 원리이다. TCP 세그먼트는 이 거대한 마트료시카 인형 세트 중 하나이며, 그 신뢰성 있는 여정은 자신을 감싸고 있는 하위 계층과 자신이 담고 있는 상위 계층 데이터와의 완벽한 조화에 의존한다.


이 장에서는 TCP가 '어떻게' 동작하는지에 대해 심층적으로 분석한다. 프로토콜의 생명주기, 즉 연결의 시작, 유지, 종료 과정을 기계적으로 살펴보고, 신뢰성을 보장하는 핵심 기능들의 정교한 작동 원리를 파헤친다.



TCP의 연결 지향적 특성을 구현하는 핵심 절차는 **3-Way Handshake**이다. 이름 그대로, 세 번의 통신을 통해 클라이언트와 서버 간의 논리적 연결을 설정한다.1

- **1단계: SYN (클라이언트 -->> 서버)**
  - 연결을 시작하려는 클라이언트는 **SYN(Synchronize)** 플래그가 1로 설정된 TCP 세그먼트를 서버에 전송한다. 이 세그먼트에는 연결의 시작 순서 번호를 알리기 위한 **초기 순서 번호(Initial Sequence Number, ISN)**가 포함된다. 이 ISN은 보안상의 이유로 예측 불가능한 임의의 숫자(예: `x`)로 설정된다.1
  - 이 SYN 패킷을 보낸 후, 클라이언트는 서버의 응답을 기다리는 **`SYN-SENT`** 상태로 전환된다.30
- **2단계: SYN-ACK (서버 -->> 클라이언트)**
  - **`LISTEN`** 상태에서 대기하던 서버는 클라이언트의 SYN 패킷을 수신한다. 연결 요청을 수락하면, 서버는 **SYN** 플래그와 **ACK(Acknowledgment)** 플래그를 모두 1로 설정한 세그먼트로 응답한다.1
  - 이 응답에서, 서버는 클라이언트의 ISN(`x`)을 잘 받았다는 의미로 ACK 번호를 `x+1`로 설정한다. 동시에 서버 자신도 통신에 사용할 고유한 ISN(예: `y`)을 생성하여 순서 번호 필드에 담아 보낸다.24
  - 이 SYN-ACK 패킷을 보낸 후, 서버는 **`SYN-RECEIVED`** 상태로 전환된다.30
- **3단계: ACK (클라이언트 -->> 서버)**
  - 클라이언트는 서버로부터 받은 SYN-ACK 패킷을 확인한다. 그리고 서버의 ISN(`y`)을 잘 받았다는 최종 확인의 의미로, ACK 플래그가 1로 설정된 세그먼트를 다시 서버로 전송한다. 이때 ACK 번호는 서버의 ISN에 1을 더한 `y+1`로 설정된다.1
  - 이 마지막 ACK 패킷을 보내는 즉시 클라이언트는 **`ESTABLISHED`** 상태가 되며, 데이터를 주고받을 준비를 마친다. 서버 역시 이 ACK를 수신하면 **`ESTABLISHED`** 상태로 전환되어, 마침내 양방향 통신을 위한 연결이 완전히 수립된다.23

ISN을 예측 불가능한 난수로 사용하는 것은 매우 중요한 보안 조치이다. 만약 ISN이 순차적이거나 예측 가능하다면, 공격자는 쉽게 위조된 패킷을 만들어 기존 연결을 가로채거나(세션 하이재킹) 새로운 연결 수립을 방해할 수 있다. 난수 ISN은 과거에 사용되었던 연결의 지연된 패킷과 새로운 연결의 패킷이 혼동되는 것을 방지하는 역할도 한다.30


TCP 연결은 전이중(full-duplex) 방식이므로, 한쪽이 데이터 전송을 마쳤다고 해서 연결을 바로 끊을 수 없다. 상대방은 아직 보낼 데이터가 남아있을 수 있기 때문이다. 이 때문에 연결 종료 과정은 연결 수립보다 한 단계 더 많은 **4-Way Handshake**를 통해 이루어진다.1

- **1단계: FIN (Initiator -->> Responder)**
  - 연결 종료를 원하는 측(예: 클라이언트)은 **FIN(Finish)** 플래그가 설정된 세그먼트를 상대방(서버)에게 보낸다. 이는 "나는 더 이상 보낼 데이터가 없다"는 신호이다.
  - 이후 클라이언트는 **`FIN-WAIT-1`** 상태로 전환되어 서버의 응답을 기다린다.29
- **2단계: ACK (Responder -->> Initiator)**
  - 서버는 클라이언트의 FIN을 수신하고, 이를 확인했다는 의미로 ACK 세그먼트를 보낸다.
  - 이 ACK를 받은 클라이언트는 **`FIN-WAIT-2`** 상태로 전환된다. 서버는 **`CLOSE-WAIT`** 상태가 되는데, 이는 클라이언트의 종료 요청은 인지했지만, 아직 서버 측에서 애플리케이션에 보낼 데이터가 남아있을 수 있음을 의미한다. 이 상태에서 서버는 남은 데이터를 모두 전송할 수 있다. 이때 연결은 '반-닫힘(half-closed)' 상태가 된다.23
- **3단계: FIN (Responder -->> Initiator)**
  - 서버가 자신의 모든 데이터 전송을 마치고 연결을 종료할 준비가 되면, 서버 역시 FIN 플래그가 설정된 세그먼트를 클라이언트에게 보낸다.
  - 이후 서버는 클라이언트의 마지막 ACK를 기다리는 **`LAST-ACK`** 상태로 전환된다.29
- **4단계: ACK (Initiator -->> Responder)**
  - 클라이언트는 서버의 FIN을 수신하고, 이에 대한 최종 확인으로 ACK 세그먼트를 서버에 보낸다.
  - 이 ACK를 받은 서버는 즉시 연결을 닫고 **`CLOSED`** 상태가 된다. 반면, 클라이언트는 마지막 ACK를 보낸 후 바로 닫히지 않고 **`TIME-WAIT`** 상태로 일정 시간 대기한다.29

**`TIME_WAIT` 상태의 중요성:** 클라이언트가 마지막 ACK를 보내고 즉시 연결을 닫지 않고 `TIME_WAIT` 상태(일반적으로 2MSL, Maximum Segment Lifetime의 두 배 시간)를 유지하는 이유는 두 가지다. 첫째, 클라이언트가 보낸 마지막 ACK가 네트워크 문제로 유실되었을 경우, 서버는 ACK를 받지 못해 `LAST-ACK` 상태에서 계속 대기하며 FIN을 재전송할 수 있다. 이때 `TIME_WAIT` 상태의 클라이언트는 재전송된 FIN을 받고 다시 ACK를 보내주어 서버가 정상적으로 연결을 종료하도록 보장한다. 둘째, 네트워크에 떠돌던 이전 연결의 지연된 패킷들이, 동일한 포트 번호를 사용하는 새로운 연결에 잘못 끼어들어 오작동을 일으키는 것을 방지한다. `TIME_WAIT` 시간 동안 모든 잠재적 지연 패킷이 소멸되도록 기다리는 것이다.29

**표 2: TCP 연결 상태 전환**

| 상태               | 설명 및 트리거 이벤트                                        |
| ------------------ | ------------------------------------------------------------ |
| **`CLOSED`**       | 연결이 없는 초기 상태.                                       |
| **`LISTEN`**       | 서버가 클라이언트의 연결 요청을 기다리는 상태.               |
| **`SYN-SENT`**     | 클라이언트가 SYN 패킷을 보내고 서버의 SYN-ACK를 기다리는 상태. |
| **`SYN-RECEIVED`** | 서버가 SYN 요청을 받고 클라이언트의 마지막 ACK를 기다리는 상태. |
| **`ESTABLISHED`**  | 3-Way Handshake가 완료되어 데이터 교환이 가능한 안정된 상태. |
| **`FIN-WAIT-1`**   | 자신이 FIN을 보내고 상대방의 ACK 또는 FIN을 기다리는 상태.   |
| **`FIN-WAIT-2`**   | 자신의 FIN에 대한 ACK는 받았고, 상대방의 FIN을 기다리는 상태. |
| **`CLOSE-WAIT`**   | 상대방의 FIN 요청을 받고, 애플리케이션이 연결을 닫을 때까지 기다리는 상태. |
| **`LAST-ACK`**     | 자신의 FIN을 보내고, 이에 대한 마지막 ACK를 기다리는 상태.   |
| **`CLOSING`**      | 양측이 거의 동시에 FIN을 보내어, 자신의 FIN에 대한 ACK를 받지 못한 상태. |
| **`TIME-WAIT`**    | 연결 종료 후, 잠재적인 지연 패킷 문제를 방지하기 위해 일정 시간 대기하는 상태. |

자료: 23



TCP의 신뢰성 보장의 핵심은 순서 번호(SEQ)와 확인 응답 번호(ACK)의 정교한 상호작용에 있다.

- **바이트 단위 순서 지정:** TCP는 데이터를 개별 패킷 단위가 아닌, 연속된 바이트 스트림으로 간주한다. 세그먼트 헤더의 순서 번호는 해당 세그먼트가 담고 있는 데이터의 첫 번째 바이트가 전체 스트림에서 몇 번째 바이트인지를 나타낸다.25 이 덕분에 수신자는 네트워크에서 순서가 뒤섞여 도착한 세그먼트들을 올바르게 재조립할 수 있다.10
- **누적 확인 응답 (Cumulative Acknowledgment):** TCP의 ACK는 기본적으로 누적 방식이다. 수신자가 ACK 번호로 `N`을 보냈다면, 이는 `N-1`번째 바이트까지의 모든 데이터를 빠짐없이, 순서대로 잘 받았다는 의미이며, 이제 `N`번째 바이트부터의 데이터를 기대하고 있음을 나타낸다.25 이 방식은 매 패킷마다 ACK를 보내지 않아도 되므로 효율적이다.
- **중복 ACK를 통한 빠른 손실 감지:** 만약 중간에 세그먼트 하나가 유실되면, 수신자는 그 이후에 도착하는 세그먼트들에 대해 계속해서 마지막으로 성공적으로 수신한 바이트의 다음 번호를 ACK로 보낸다. 송신자 입장에서 동일한 ACK 번호를 여러 번(일반적으로 3번) 수신하는 것을 **중복 ACK(Duplicate ACK)**라고 하며, 이는 해당 ACK 번호의 세그먼트가 유실되었음을 암시하는 강력한 신호다. 이 경우 송신자는 재전송 타이머가 만료되기를 기다리지 않고 즉시 해당 세그먼트를 재전송한다. 이것이 바로 '빠른 재전송(Fast Retransmit)' 메커니즘의 기초가 된다.19


데이터가 전송 중에 손상될 가능성에 대비하여, TCP는 **체크섬(Checksum)** 메커니즘을 사용한다.10

- **계산 방식:** 송신자는 TCP 헤더, 데이터 페이로드, 그리고 IP 주소와 같은 추가 정보로 구성된 '가상 헤더(pseudo-header)'를 모두 포함하여 16비트 체크섬 값을 계산하고, 이를 헤더의 체크섬 필드에 기록한다.
- **검증:** 수신자는 세그먼트를 받으면 동일한 방식으로 체크섬을 다시 계산한다. 계산된 값이 헤더에 포함된 체크섬 값과 일치하지 않으면, 데이터가 전송 중에 손상되었다고 판단하고 해당 세그먼트를 폐기한다.10 폐기된 세그먼트는 송신 측의 재전송 메커니즘(타임아웃 또는 중복 ACK)에 의해 결국 다시 전송된다.


**흐름 제어(Flow Control)**는 송신 측의 전송 속도가 수신 측의 처리 속도를 초과하여 수신 버퍼가 넘쳐나는(overflow) 상황을 방지하기 위한 메커니즘이다.1 이는 전적으로 수신자가 주도하는 제어 방식이다.

- **수신 윈도우 (Receive Window, RWIN):** 수신자는 자신이 현재 수용할 수 있는 데이터의 양, 즉 버퍼의 여유 공간 크기를 TCP 헤더의 **윈도우 크기(Window Size)** 필드에 담아 송신자에게 지속적으로 알린다.24
- **슬라이딩 윈도우 (Sliding Window):** 송신자는 수신자가 알려준 RWIN 크기를 초과하지 않는 범위 내에서만 데이터를 전송할 수 있다. 이 허용된 데이터 양을 **송신 윈도우**라고 한다. 수신자가 데이터를 처리하고 버퍼 공간을 확보하면, 새로운 ACK를 통해 더 커진 윈도우 크기를 광고한다. 그러면 송신자는 더 많은 데이터를 보낼 수 있게 되는데, 이처럼 유효한 송신 데이터의 범위가 앞으로 미끄러지듯 이동한다고 해서 **슬라이딩 윈도우**라고 부른다.14
- **제로 윈도우 (Zero-Window):** 만약 수신자의 버퍼가 가득 차면, RWIN 값으로 0을 광고한다. 이 '제로 윈도우' 통보를 받은 송신자는 수신자로부터 0이 아닌 새로운 윈도우 크기를 받을 때까지 데이터 전송을 중단해야 한다.36

흐름 제어와 혼잡 제어는 둘 다 데이터 전송률을 조절하지만, 근본적으로 다른 두 가지 문제를 해결한다. 흐름 제어는 송신자와 수신자라는 **두 종단 간의 속도 차이**를 조절하여 **수신자를 보호**하는 지점 대 지점(point-to-point) 메커니즘이다.1 이는 수신자가 광고하는 RWIN에 의해 결정된다.24 반면, 혼잡 제어는 네트워크 자체의 처리 용량을 초과하지 않도록 조절하여 

**네트워크 전체를 보호**하는 전역적인 메커니즘이다.1 이는 패킷 손실이나 지연과 같은 네트워크 상태를 추론하여 결정되는 혼잡 윈도우(cwnd)에 의해 제어된다.

실제 TCP 송신자의 전송률은 이 두 가지 제어 메커니즘에 의해 동시에 제한받으며, 최종적으로 보낼 수 있는 데이터의 양은 **수신 윈도우(RWIN)와 혼잡 윈도우(cwnd) 중 더 작은 값**으로 결정된다. 이 이중 제어 시스템의 분리는 TCP의 강건함을 보여주는 중요한 설계 특징이다. 예를 들어, 네트워크 경로는 완전히 한산하여 cwnd가 매우 크더라도, 수신 애플리케이션이 데이터를 느리게 처리하여 RWIN이 작아지면 TCP는 수신자를 보호하기 위해 전송 속도를 늦춘다. 반대로, 수신자에게는 거대한 버퍼가 있어 RWIN이 매우 크더라도, 중간 경로에 혼잡이 발생하여 cwnd가 작아지면 TCP는 네트워크를 보호하기 위해 전송 속도를 줄인다. 이처럼 종단과 네트워크 내부의 제약 조건을 독립적으로 판단하고 적응하는 능력 덕분에 TCP는 다양한 환경에서 안정적으로 작동할 수 있다.


이 장은 TCP의 가장 복잡하고 중요한 기능 중 하나인 혼잡 제어에 대해 집중적으로 다룬다. 초기 반응적 접근 방식부터 고속의 복잡한 네트워크를 위해 설계된 현대의 정교한 전략에 이르기까지, 혼잡 제어 알고리즘과 그 사상의 진화 과정을 상세히 추적한다.



**혼잡(Congestion)**은 네트워크로 유입되는 데이터의 양이 라우터나 스위치와 같은 네트워크 장비의 처리 용량을 초과할 때 발생하는 현상이다.38 이로 인해 장비의 버퍼가 가득 차고, 결국 패킷이 버려지며(packet loss), 전체적인 지연 시간이 급증하게 된다.37

만약 혼잡 제어 메커니즘이 없다면, 송신자들은 유실된 패킷을 순진하게 재전송할 것이고, 이는 네트워크의 혼잡을 더욱 악화시켜 결국 네트워크 처리량이 급격히 떨어지는 **혼잡 붕괴(Congestion Collapse)** 상태에 이르게 할 수 있다.38 TCP의 혼잡 제어는 이러한 재앙을 막기 위해 모든 참여자가 네트워크 자원을 공평하고 효율적으로 사용하도록 유도하는 '선량한 시민(good citizen)' 메커니즘이다.37


초기 TCP 혼잡 제어는 손실을 혼잡의 신호로 간주하고 이에 반응하는 방식으로 설계되었다.

- **혼잡 윈도우 (Congestion Window, `cwnd`):** TCP 송신자는 **혼잡 윈도우(`cwnd`)**라는 상태 변수를 유지한다. 이 변수는 확인 응답(ACK)을 받지 않은 상태에서 네트워크에 보낼 수 있는 데이터의 최대량을 제한한다. 실제 송신 가능한 데이터 양은 `cwnd`와 수신자의 수신 윈도우(RWIN) 중 더 작은 값으로 결정된다.37
- **느린 시작 (Slow Start):** 연결이 시작되거나 타임아웃으로 인한 패킷 손실이 발생하면, TCP는 **느린 시작** 단계에 들어간다. `cwnd`는 1~10 MSS(Maximum Segment Size) 정도의 작은 값으로 초기화된다. 이후 ACK를 하나 받을 때마다 `cwnd`가 1 MSS씩 증가하여, 결과적으로 매 RTT(Round-Trip Time)마다 `cwnd`가 두 배로 늘어나는 지수적 증가를 보인다. 이를 통해 가용 대역폭을 빠르게 탐색한다.38
- **혼잡 회피 (Congestion Avoidance)와 AIMD:** `cwnd`가 특정 임계값(**`ssthresh`**, slow start threshold)에 도달하면, TCP는 **혼잡 회피** 단계로 전환한다. 이 단계에서는 `cwnd`가 매 RTT마다 1 MSS씩 선형적으로 증가한다. 이는 보다 보수적인 접근 방식이다. 만약 이 단계에서 패킷 손실(혼잡)이 감지되면, `cwnd`는 곱셈적으로(보통 절반으로) 감소한다. 이 '느리게 증가시키고, 빠르게 감소시키는' 방식이 **AIMD(Additive Increase, Multiplicative Decrease)** 알고리즘의 핵심이다.37
- **빠른 재전송 (Fast Retransmit):** 앞서 언급했듯이, 3개의 중복 ACK를 수신하면 송신자는 타임아웃을 기다리지 않고 즉시 손실된 것으로 추정되는 패킷을 재전송한다. 이는 간헐적인 패킷 손실 상황에서 성능을 크게 향상시킨다.38
- **빠른 회복 (Fast Recovery):** **TCP Tahoe**는 어떤 종류의 손실이든 감지하면 `cwnd`를 1로 줄이고 느린 시작 단계로 돌아갔다. 이는 불필요하게 전송률을 급격히 떨어뜨리는 문제를 야기했다. **TCP Reno**는 이를 개선하여 **빠른 회복** 메커니즘을 도입했다. 빠른 재전송이 발생하면, Reno는 `cwnd`를 절반으로 줄이지만(곱셈적 감소) 느린 시작으로 돌아가지 않는다. 대신, 네트워크에 아직 남아있을 중복 ACK들을 고려하여 일시적으로 `cwnd`를 부풀린 후, 바로 혼잡 회피 단계(선형 증가)로 진입한다. 이로써 사소한 혼잡 이벤트에 대해 전송률이 급감하는 것을 막을 수 있다.38



- **등장 배경:** Reno의 AIMD 방식은 대역폭-지연 곱(Bandwidth-Delay Product, BDP)이 큰 네트워크, 소위 '길고 뚱뚱한 네트워크(Long Fat Networks, LFNs)'에서 가용 대역폭을 완전히 활용하기까지 너무 오랜 시간이 걸리는 문제가 있었다.
- **작동 원리:** **CUBIC**은 이러한 환경에 더 적합하도록 윈도우 증가 함수를 수정한 알고리즘이다. 패킷 손실이 발생하면, CUBIC은 손실 직전의 `cwnd` 값을 기반으로 새로운 목표 윈도우(`W_max`)를 설정한다. 이후 `cwnd`의 성장은 마지막 손실 이벤트로부터 경과된 시간을 변수로 하는 **3차 함수(cubic function)**에 의해 결정된다. `cwnd`가 `W_max`에 가까워질 때는 안정성을 위해 천천히 증가하고, `W_max`를 넘어서면 더 많은 대역폭을 탐색하기 위해 다시 빠르게 증가한다. 이 방식은 높은 BDP 환경에서 Reno보다 훨씬 공격적이고 확장성 있게 동작하며, 현재 대부분의 리눅스 배포판에서 기본 혼잡 제어 알고리즘으로 사용된다.46


- **손실 기반 혼잡 제어의 한계:** CUBIC과 같은 전통적인 알고리즘은 **손실 기반(loss-based)**이다. 즉, 패킷 손실을 혼잡의 유일한 신호로 간주한다. 하지만 버퍼가 매우 깊은 현대 네트워크나, 혼잡과 무관한 무선 오류가 잦은 환경에서는 이 가정이 틀리는 경우가 많다. 이로 인해 비혼잡성 손실에도 불필요하게 속도를 줄여 대역폭을 낭비하거나, 손실이 발생할 때까지 깊은 버퍼를 가득 채워 심각한 지연(버퍼블로트)을 유발하는 문제가 발생한다.50

- **BBR의 핵심 원리:** **BBR(Bottleneck Bandwidth and Round-trip propagation time)**은 이러한 문제에 대한 근본적인 해결책으로 구글이 개발한 **모델 기반(model-based)** 알고리즘이다. BBR은 손실에 반응하는 대신, 네트워크 경로에 대한 모델을 능동적으로 구축한다. 이를 위해 다음 두 가지 핵심 파라미터를 지속적으로 측정한다 51:

  1. **병목 대역폭 (Bottleneck Bandwidth, `BtlBw`):** 해당 경로의 최대 데이터 전송률.
  2. **왕복 전파 시간 (Round-trip Propagation Time, `RTprop`):** 큐잉 지연이 없는 순수한 물리적 왕복 시간.

- **페이싱과 상태 머신:** BBR은 이 모델을 사용하여 경로의 BDP를 계산하고, 전송 속도를 `BtlBw`에 맞춰 **페이싱(pacing)**함으로써 데이터 패킷을 일정한 간격으로 보낸다. 이는 버퍼를 불필요하게 채우는 것을 방지하기 위함이다. BBR은 상태 머신(state machine)에 따라 동작하며, 주기적으로 더 많은 대역폭을 탐색하는 **`PROBE_BW`** 단계와, 큐를 비우고 정확한 `RTprop`을 측정하기 위해 잠시 전송률을 낮추는 **`PROBE_RTT`** 단계를 순환한다.50

- **BBRv2로의 진화:** 초기 버전인 BBRv1은 CUBIC과 같은 손실 기반 알고리즘과 공존할 때 불공평하게 대역폭을 점유하고, 특정 조건에서 높은 재전송률을 보이는 문제가 지적되었다.55 이러한 문제들을 해결하기 위해 

  **BBRv2**가 개발되었다. BBRv2는 공정성을 개선하고 손실 신호 및 ECN(Explicit Congestion Notification)에 더 잘 반응하도록 설계되어, 공유 인터넷 환경에서 더 나은 '시민'으로 동작한다.50

**표 3: 주요 혼잡 제어 알고리즘 비교**

| 특징          | Reno (AIMD)                      | CUBIC                            | BBR                                         |
| ------------- | -------------------------------- | -------------------------------- | ------------------------------------------- |
| **핵심 원리** | 손실 기반 합 증가/곱 감소        | 손실 기반 3차 함수 성장          | 모델 기반 BDP 페이싱                        |
| **혼잡 신호** | 패킷 손실 (타임아웃, 3 중복 ACK) | 패킷 손실 (타임아웃, 3 중복 ACK) | BDP 모델 (RTT 증가, 전달률 변화)            |
| **장점**      | 단순함, 표준적                   | 높은 BDP 네트워크에서 효율적     | 낮은 지연 시간, 버퍼블로트 방지             |
| **단점**      | 높은 BDP 네트워크에서 비효율적   | 여전히 손실 기반, 지연 유발 가능 | 타 흐름에 불공평할 수 있음, CPU 사용량 높음 |
| **주요 환경** | 레거시 시스템                    | 범용, 리눅스 기본값              | 구글, 유튜브, CDN 등 콘텐츠 전송            |

자료: 37


이 장에서는 프로토콜 이론과 실제 운영 환경 사이의 간극을 메운다. TCP의 설계가 실제 네트워크 인프라와 상호작용하며 발생하는 문제들을 살펴보고, 상태 기반 특성이 공격자에게 어떻게 악용되는지, 그리고 운영자가 이를 어떻게 튜닝하고 분석할 수 있는지 알아본다.



- **정의:** **버퍼블로트(Bufferbloat)**는 네트워크 장비(라우터, 스위치 등)에 설정된 과도하게 큰 버퍼로 인해 발생하는 불필요한 지연 시간(latency) 및 지터(jitter) 현상을 말한다.59
- **원인:** 링크에 혼잡이 발생했을 때, 패킷이 즉시 버려지지 않고 이 거대한 버퍼에 계속해서 쌓이게 된다. 패킷 손실을 혼잡 신호로 삼는 손실 기반 TCP 알고리즘(예: CUBIC)은 버퍼가 가득 차서 패킷이 버려지기 전까지는 혼잡을 인지하지 못한다. 그 결과, TCP는 계속해서 데이터를 전송하여 버퍼를 가득 채우고, 이로 인해 수백 밀리초에서 수 초에 달하는 큐잉 지연이 발생한다.59
- **영향:** 이렇게 급증한 지연 시간은 VoIP, 온라인 게임, 심지어 단순한 웹 브라우징과 같은 실시간 상호작용이 중요한 애플리케이션의 성능을 심각하게 저하시켜, 고속 인터넷 연결을 매우 느리게 느껴지게 만든다.59 이 문제는 해당 버퍼를 공유하는 모든 트래픽에 영향을 미친다.59
- **해결책:** 해결 방안은 네트워크 단에서의 **지능형 큐 관리(Active Queue Management, AQM)** 알고리즘(예: CoDel, FQ-CoDel) 도입과, 종단 단에서의 개선된 혼잡 제어 알고리즘 사용으로 나뉜다. 특히 BBR과 같이 버퍼를 채우는 것을 목표로 하지 않는 모델 기반 알고리즘은 버퍼블로트 문제에 대한 효과적인 해결책 중 하나이다.59


- **정의:** TCP는 데이터를 단일의, 순서가 보장된 스트림으로 처리한다. 이 때문에 스트림의 맨 앞에서 패킷 하나가 유실되면, 그 뒤에 오는 모든 패킷들은 설령 정상적으로 수신되었더라도 유실된 패킷이 재전송되어 도착할 때까지 대기해야 한다. 이를 **HOL 블로킹**이라고 한다.64
- **HTTP/2에서의 영향:** HTTP/2는 단일 TCP 연결 위에서 여러 개의 HTTP 요청과 응답을 동시에 처리하는 스트림 멀티플렉싱을 도입했다. 하지만 이 모든 논리적 스트림이 결국 하나의 TCP 바이트 스트림을 공유하기 때문에, 한 HTTP 요청(예: 이미지 파일)에 속한 패킷 하나만 유실되어도 다른 모든 요청(예: CSS, 자바스크립트 파일)의 전달이 함께 중단되는 심각한 성능 저하를 겪게 된다.64

TCP의 엄격한 순서 보장 모델에 내재된 이 HOL 블로킹 문제는 TCP를 점진적으로 개선해서는 해결할 수 없는 근본적인 한계를 드러냈고, 이는 결국 QUIC 프로토콜 개발의 핵심적인 동기가 되었다. TCP의 순서 보장은 원래 신뢰성을 위한 핵심 '기능'이었지만, 다수의 리소스를 병렬로 빠르게 가져와야 하는 현대 웹 환경에서는 오히려 '족쇄'가 된 것이다. 이 문제를 해결하기 위해서는 단일 바이트 스트림 모델 자체를 포기하고, 하나의 연결 내에서 여러 스트림이 독립적으로 전송될 수 있는 새로운 전송 계층 아키텍처가 필요했다. QUIC은 바로 이러한 요구에 부응하여 설계되었으며, 하나의 스트림에서 발생한 손실이 다른 스트림을 막지 않도록 함으로써 HOL 블로킹 문제를 근본적으로 해결했다.65



- **공격 원리:** **SYN 플러드(SYN Flood)**는 대표적인 서비스 거부(Denial-of-Service, DoS) 공격의 한 종류이다. 공격자는 주로 위조된(spoofed) IP 주소를 사용하여 서버에 대량의 SYN 패킷을 보낸다. 서버는 각 SYN 요청에 대해 SYN-ACK로 응답하고, '반-개방(half-open)' 상태의 연결을 위해 시스템 자원을 할당한다. 하지만 공격자는 마지막 ACK를 보내지 않으므로, 서버는 응답이 올 때까지 하염없이 기다리게 된다.68
- **영향:** 서버의 연결 테이블(backlog queue)이 이러한 반-개방 상태의 연결로 가득 차게 되면, 더 이상 새로운 정상적인 연결 요청을 받아들일 수 없게 되어 서비스가 마비된다.68
- **완화 기법:**
  - **SYN 쿠키 (SYN Cookies):** 서버가 SYN-ACK 응답에 연결 정보를 암호화하여 '쿠키'로 만들어 보낸다. 서버는 클라이언트로부터 이 쿠키가 포함된 유효한 ACK를 받기 전까지는 어떠한 자원도 할당하지 않는다. 이를 통해 자원 고갈을 원천적으로 방지한다.69
  - **백로그 큐 증가:** 연결 테이블의 크기를 늘리는 단순한 방법이지만, 대규모 공격에는 한계가 있다.69
  - **비율 제한 (Rate Limiting):** 특정 IP 주소로부터 오는 SYN 요청의 수를 제한한다.69
  - **방화벽/프록시/로드밸런서:** 서버 앞단에서 악성 트래픽을 필터링하거나 핸드셰이크 과정을 대신 처리하여 서버를 보호한다.69


- **공격 원리:** **TCP 세션 하이재킹(TCP Session Hijacking)**은 공격자가 이미 수립된 두 당사자 간의 TCP 세션을 가로채 제어권을 탈취하는 공격이다. 공격의 핵심은 통신에 사용되는 TCP 순서 번호를 예측하거나 도청하는 것이다.74 공격자는 정확한 순서 번호를 가진 위조 패킷을 세션에 주입하여, 서버가 이를 정상적인 클라이언트의 패킷으로 오인하게 만든다.
- **공격 단계:**
  1. **도청 (Sniffing):** 공격자는 네트워크 트래픽을 감청하여 유효한 세션 정보와 현재 사용 중인 순서 번호를 알아낸다.76
  2. **비동기화 (Desynchronization):** 공격자는 정상 클라이언트가 서버에 응답하지 못하도록 DoS 공격을 가하거나, ARP 스푸핑 같은 기법으로 트래픽의 경로를 자신에게로 돌린다.75
  3. **패킷 주입 (Injection):** 공격자는 클라이언트의 IP로 위장하고 알아낸 순서 번호를 사용하여 악의적인 명령이 담긴 패킷을 서버에 전송한다.75
- **방어 전략:** 가장 근본적이고 효과적인 방어책은 **암호화**이다. TLS/SSL 프로토콜을 TCP 위에 적용하여 HTTPS나 SSH와 같은 보안 채널을 사용하면, TCP 페이로드 전체가 암호화된다. 이 경우 공격자가 트래픽을 도청하더라도 세션 토큰이나 데이터 내용을 알아볼 수 없으며, 암호화된 스트림의 순서 번호를 추측하기도 매우 어려워진다.75 애플리케이션 수준에서 예측 불가능한 강력한 세션 ID를 생성하고 관리하는 것 또한 필수적이다.76



리눅스 커널은 `sysctl` 인터페이스(주로 `/proc/sys/net/` 경로에 위치)를 통해 TCP 동작을 제어하는 다양한 파라미터를 제공한다. 시스템 관리자는 이를 조정하여 특정 워크로드나 네트워크 환경에 맞게 TCP 성능을 최적화할 수 있다.79

- **주요 튜닝 파라미터:**
  - `net.core.rmem_max`, `net.core.wmem_max`: TCP 소켓이 사용할 수 있는 최대 수신 및 송신 버퍼 크기를 설정한다. 높은 BDP 환경에서는 이 값을 늘려야 고성능을 낼 수 있다.46
  - `net.ipv4.tcp_rmem`, `net.ipv4.tcp_wmem`: TCP 버퍼 크기 자동 튜닝의 최소, 기본, 최대값을 설정한다. `rmem_max`와 함께 조정되어야 한다.46
  - `net.ipv4.tcp_congestion_control`: 시스템의 기본 혼잡 제어 알고리즘을 `cubic`이나 `bbr` 등으로 지정한다.46
  - `net.ipv4.tcp_mtu_probing`: 점보 프레임 사용 시 발생할 수 있는 경로 MTU 블랙홀 문제를 완화하는 데 도움을 준다.46
  - `net.core.default_qdisc`: 기본 큐잉 규칙을 `fq`나 `fq_codel`과 같은 공평 큐잉(Fair Queuing) 방식으로 설정하면 버퍼블로트를 줄이고 흐름 간 공정성을 높일 수 있다.46
  - `net.ipv4.tcp_syncookies`: SYN 플러드 공격에 대한 방어책인 SYN 쿠키를 활성화한다.80
- **설정 영구화:** `sysctl -w` 명령으로 변경한 설정은 시스템 재부팅 시 사라진다. 변경 사항을 영구적으로 적용하려면 `/etc/sysctl.conf` 파일이나 `/etc/sysctl.d/` 디렉토리 내의 설정 파일에 해당 내용을 기록해야 한다.79


**와이어샤크(Wireshark)**는 네트워크 트래픽을 캡처하고 상세히 분석할 수 있는 강력한 프로토콜 분석 도구로, TCP의 동작을 시각적으로 확인하고 문제를 진단하는 데 필수적이다.36

- **3-Way Handshake 분석:** 와이어샤크는 캡처된 트래픽에서 SYN, SYN-ACK, ACK 패킷을 명확하게 구분하여 보여준다. 이를 통해 플래그 설정, 상대적 순서 번호의 변화, MSS나 윈도우 스케일링과 같은 옵션 협상 과정을 직접 확인할 수 있다.32
- **TCP 문제 해결 기능:** 와이어샤크의 전문가 정보(Expert Information) 시스템은 잠재적인 문제를 자동으로 감지하고 색상으로 표시하여 분석을 돕는다.36
  - **재전송 및 중복 ACK:** 패킷 손실과 복구 과정을 파악할 수 있다.
  - **제로 윈도우:** 수신자 버퍼가 가득 찼음을 나타낸다.
  - **이전 세그먼트가 캡처되지 않음 (Previous segment not captured):** 캡처 경로 상에서 패킷 손실이 있었음을 의미한다.
  - **순서가 맞지 않는 세그먼트 (Out-of-Order):** 네트워크에서 패킷 순서가 뒤바뀌었음을 보여준다.
- **필터링:** `tcp.flags.syn==1`이나 `tcp.analysis.retransmission`과 같은 디스플레이 필터를 사용하면, 대량의 트래픽 속에서 특정 TCP 이벤트(연결 시작, 재전송 등)만 분리하여 집중적으로 분석할 수 있다.36


이 마지막 장에서는 현재의 TCP를 넘어, 그 후계자, 확장 기술, 그리고 전송 계층의 미래를 재편하고 있는 혁신적인 기술들을 탐구한다. 네트워크 프로토콜 진화의 '다음 단계는 무엇이며, 왜 필요한가'에 대한 해답을 모색한다.



- **개발 동기:** **QUIC(Quick UDP Internet Connections)**은 구글이 현대 웹 환경에서 TCP가 가진 근본적인 한계들을 해결하기 위해 설계한 프로토콜이다.51
- **HOL 블로킹 해결:** QUIC은 TCP 대신 UDP 위에서 동작하며, 단일 연결 내에 여러 개의 독립적인 경량 스트림을 구현한다. 한 스트림에서 패킷이 손실되더라도 다른 스트림의 진행을 막지 않아, TCP의 고질적인 HOL 블로킹 문제를 효과적으로 해결한다.65
- **연결 지연 시간 단축:** QUIC은 전송 계층 핸드셰이크와 암호화 계층(TLS 1.3) 핸드셰이크를 통합하여 하나의 과정으로 처리한다. 이로 인해 새로운 연결 설정에 일반적으로 1 RTT만 소요되며, 이는 TCP+TLS의 2~3 RTT에 비해 크게 단축된 것이다. 또한, 이전에 연결했던 서버와 다시 연결할 때는 **0-RTT**를 지원하여 거의 즉각적인 연결이 가능하다.65


- **사용자 공간 구현:** QUIC은 UDP 위에서 실행되므로, 혼잡 제어, 흐름 제어 등 핵심 로직이 운영체제 커널이 아닌 사용자 공간(user-space) 라이브러리에 구현된다.22 이는 OS 업데이트를 기다릴 필요 없이 프로토콜을 신속하게 개선하고 배포할 수 있게 하여, 커널 기반 TCP의 '화석화(ossification)' 문제를 극복한다.85
- **기본적으로 암호화:** QUIC은 페이로드뿐만 아니라 전송 헤더의 대부분을 암호화한다. 이는 보안과 프라이버시를 강화하고, 중간의 네트워크 장비(middlebox)가 프로토콜의 발전을 방해하는 것을 더 어렵게 만든다.65
- **연결 마이그레이션 (Connection Migration):** QUIC 연결은 IP 주소와 포트 번호의 4-튜플이 아닌, **연결 ID(Connection ID)**로 식별된다. 덕분에 사용자가 Wi-Fi에서 셀룰러 네트워크로 전환하는 등 네트워크 환경이 바뀌더라도 연결이 끊기지 않고 원활하게 유지된다.88



- **목적:** **MPTCP(Multipath TCP)**는 단일 TCP 연결이 두 호스트 간의 여러 네트워크 경로(예: Wi-Fi와 5G를 동시에)를 사용할 수 있도록 하는 TCP 확장 기술이다.89
- **이점:**
  1. **처리량 향상:** 사용 가능한 모든 경로의 대역폭을 합산하여 전체 처리량을 높인다.89
  2. **연결 복원력 증대:** 한 경로에 장애가 발생하더라도 나머지 경로를 통해 통신이 끊김 없이 계속될 수 있어 연결의 안정성과 복원력이 크게 향상된다. 애플의 음성 비서 서비스인 Siri가 이 기술을 활용하는 대표적인 사례다.89
- **작동 원리:** MPTCP는 일종의 심(shim) 계층으로 동작한다. 먼저 하나의 TCP 연결(서브플로우, subflow)을 맺고, 새로운 TCP 옵션(`MP_CAPABLE`)을 통해 다중 경로 지원 여부를 협상한다. 이후 다른 경로를 통해 추가적인 서브플로우를 생성할 수 있다. 서로 다른 지연 시간을 가진 여러 서브플로우를 통해 도착하는 패킷들의 순서를 재조립하기 위해, 연결 전체에 걸쳐 적용되는 고유한 데이터 순서 번호를 사용한다.89
- **표준화:** MPTCP 버전 1은 **RFC 8684**로 표준화되었다.92


- **개발 동기:** 다중 경로 기술의 이점(복원력, 처리량 증대)을 QUIC 생태계에도 적용하기 위함이다.96
- **작동 원리:** IETF QUIC 워킹 그룹에서 다중 경로 기능을 위한 확장을 표준화하고 있다. 현재 논의 중인 초안은 각 경로를 식별하기 위한 명시적인 **경로 ID(Path ID)**를 도입하고, 각 경로가 독립적인 패킷 번호 공간을 갖도록 하는 방식을 제안한다. 이를 통해 경로별로 독립적인 혼잡 제어 및 손실 복구 상태를 관리하면서 QUIC의 다른 모든 장점을 그대로 계승할 수 있다.88
- **현황:** IETF QUIC 워킹 그룹 내에서 여러 초안들이 하나의 표준으로 수렴하며 활발히 작업이 진행 중이다.88



- **아키텍처:** **L4S(Low Latency, Low Loss, Scalable Throughput)**는 극심한 부하 상황에서도 모든 애플리케이션에 대해 밀리초 미만의 초저지연 큐잉을 제공하는 것을 목표로 하는 IETF 아키텍처이다 (RFC 9330, 9331).102
- **핵심 통찰:** L4S는 큐잉 지연의 근본 원인이 네트워크 큐 자체가 아니라, 송신자의 혼잡 제어 알고리즘에 있다는 통찰에서 출발한다.103
- **작동 원리:** L4S는 다음 두 요소의 조합을 필요로 한다:
  1. **송신 측의 확장형 혼잡 제어 (Scalable Congestion Control, 예: TCP Prague):** 이 알고리즘들은 기존 TCP보다 훨씬 더 정밀하고 신속하게 혼잡 신호에 반응한다.103
  2. **네트워크의 L4S 인식 AQM (예: DualQ):** 이 큐는 L4S 트래픽에 대해 더 빈번하고 정확한 **ECN(Explicit Congestion Notification)** 신호를 제공하며, 동시에 기존 트래픽이 사용하는 큰 큐와 격리시켜 지연 시간 전파를 막는다.103
- **공존 및 배포:** L4S는 점진적 배포를 위해 설계되었으며, 기존 TCP 흐름과의 공정성을 보장하고 공존하기 위한 메커니즘을 포함한다.102 그러나 부분적으로 배포된 시나리오에서의 성능과 공정성은 여전히 활발한 연구 분야로 남아있다.108


과거 네트워크의 발전은 하드웨어와 운영체제 커널의 더딘 변화 속도에 발목 잡혀 있었다. 하지만 최근 P4와 eBPF와 같은 기술의 등장은 이러한 '화석화' 현상을 종식시키고, 네트워크 혁신의 새로운 시대를 열고 있다.

- **P4 (Programming Protocol-Independent Packet Processors):** 스위치나 라우터와 같은 네트워크 장비의 **데이터 플레인(data plane)**을 프로그래밍하기 위한 도메인 특화 언어이다.111 P4를 통해 네트워크 운영자는 과거의 고정된 기능의 하드웨어에서 벗어나, 패킷 처리 로직을 직접 정의할 수 있다.113 이는 정교한 네트워크 원격 측정(telemetry), 사용자 정의 라우팅, 심지어 네트워크 장비 단에서의 보안 기능 구현까지 가능하게 한다.113
- **eBPF (extended Berkeley Packet Filter):** 안전한 샌드박스 환경에서 프로그램을 리눅스 커널 내부에서 직접 실행할 수 있게 하는 혁신적인 기술이다.115 네트워킹 분야에서 eBPF는 커널 소스 코드를 수정하지 않고도 고성능 패킷 처리, 로드 밸런싱, 심층적인 가시성 확보, 그리고 심지어 사용자 정의 TCP 혼잡 제어 알고리즘 구현까지 가능하게 한다.116

P4와 eBPF의 결합은 과거의 경직된 '블랙박스' 네트워크에서 완전히 프로그래밍 가능하고 투명하며 신속한 혁신이 가능한 생태계로의 근본적인 전환을 의미한다. 이는 **네트워크-호스트 공동 설계(network-host co-design)**라는 새로운 패러다임을 가능하게 한다. 이제 전송 프로토콜은 네트워크 패브릭과 호스트 스택이 모두 유연하다는 가정 하에 설계될 수 있다. 예를 들어, P4 스위치는 정밀한 큐잉 정보를 패킷 헤더에 직접 삽입하고(TCP-INT 118), 수신 호스트의 eBPF 프로그램은 이 정보를 읽어 TCP 혼잡 제어기에 즉시 전달할 수 있다. 이는 단순히 손실이나 RTT 변화로 혼잡을 '추론'하는 것보다 훨씬 더 정확하고 빠른 피드백 루프를 형성한다.

결론적으로, 미래의 전송 기술은 단일 프로토콜의 등장이 아니라, 애플리케이션, 호스트 스택, 그리고 네트워크 자체가 특정 성능 목표를 달성하기 위해 조화롭게 프로그래밍될 수 있는 유연한 프레임워크로 진화하고 있다. 이는 단순히 네트워크 '위에서' 프로토콜을 구축하는 시대를 넘어, 네트워크'와 함께' 프로토콜을 설계하는 패러다임의 전환을 예고한다.


전송 제어 프로토콜(TCP)은 지난 수십 년간 인터넷 통신의 근간으로서 데이터의 신뢰성 있는 전송을 책임져왔다. 3-Way/4-Way Handshake를 통한 견고한 연결 관리, 순서 번호와 확인 응답을 이용한 무결점 데이터 스트림 보장, 그리고 흐름 제어 및 혼잡 제어를 통한 안정적인 네트워크 운영은 TCP를 오늘날의 인터넷을 가능하게 한 일등공신으로 만들었다. 본 보고서는 TCP의 이러한 핵심 아키텍처와 메커니즘을 상세히 분석하였으며, CUBIC과 BBR 같은 현대적인 혼잡 제어 알고리즘으로의 진화 과정을 추적했다.

그러나 TCP의 설계 원칙은 21세기 네트워크 환경의 새로운 요구 앞에서 한계를 드러내고 있다. HOL 블로킹, 높은 연결 설정 지연, 네트워크 변화에 대한 더딘 적응력 등은 TCP가 해결해야 할 과제로 남았다. 이러한 문제들은 TCP를 점진적으로 개선하는 것을 넘어, 전송 계층에 대한 근본적인 재고를 촉발했다.

그 결과로 등장한 QUIC은 UDP를 기반으로 사용자 공간에서 혁신을 이루며 HOL 블로킹을 해결하고 연결 지연을 획기적으로 줄였다. MPTCP와 MP-QUIC은 여러 네트워크 경로를 동시에 활용하여 통신의 처리량과 복원력을 극대화하는 새로운 가능성을 열었다. 더 나아가, L4S 아키텍처는 혼잡 제어의 패러다임을 전환하여 초저지연 통신의 시대를 예고하고 있다.

이러한 프로토콜 수준의 진화와 더불어, P4와 eBPF로 대표되는 프로그래머블 네트워크 기술의 등장은 더욱 근본적인 변화를 이끌고 있다. 과거에는 불가능했던 네트워크와 호스트 간의 긴밀한 공동 설계를 통해, 이제 우리는 특정 애플리케이션의 요구에 맞춰 전송 계층의 동작을 동적으로 최적화할 수 있는 시대에 접어들고 있다.

결론적으로, TCP는 단순한 레거시 프로토콜로 남지 않을 것이다. 그것이 구축한 신뢰성, 순서 보장, 혼잡 제어라는 개념적 토대는 여전히 유효하며, 그 후계 프로토콜들에게 깊은 영향을 미치고 있다. TCP의 여정은 안정적인 통신에 대한 인류의 끊임없는 요구가 어떻게 기술적 혁신을 이끌어내는지를 보여주는 위대한 서사이며, 그 유산은 미래의 인터넷을 형성하는 데 계속해서 핵심적인 역할을 수행할 것이다.


1. [Network] Transmission Control Protocol(TCP) - 준비하는 대학생 - 티스토리, accessed July 5, 2025, https://gsbang.tistory.com/entry/Network-Transmission-Control-ProtocolTCP
2. velog.io, accessed July 5, 2025, [https://velog.io/@ppmyor/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-09.-%EC%97%B0%EA%B2%B0%EC%A7%80%ED%96%A5%ED%98%95-TCP-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C#:~:text=TCP%20%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C-,TCP%EA%B0%80%20%ED%95%98%EB%8A%94%20%EC%9D%BC,%EA%B5%90%ED%99%98%ED%95%A0%20%EC%88%98%20%EC%9E%88%EA%B2%8C%20%ED%95%9C%EB%8B%A4.](https://velog.io/@ppmyor/네트워크-09.-연결지향형-TCP-프로토콜#:~:text=TCP 프로토콜-,TCP가 하는 일,교환할 수 있게 한다.)
3. [네트워크 상식#1] 프로토콜 스택 'TCP/IP' 4계층, accessed July 5, 2025, https://another-light.tistory.com/30
4. TCP/IP 4계층의 이해, accessed July 5, 2025, [https://junu0516.github.io/posts/tcp_ip_4%EA%B3%84%EC%B8%B5/](https://junu0516.github.io/posts/tcp_ip_4계층/)
5. TCP/IP 4계층 모델 - 테드의 기술블로그 - 티스토리, accessed July 5, 2025, https://hwannny.tistory.com/117
6. [Network] TCP/IP 4계층에 대하여 - velog, accessed July 5, 2025, [https://velog.io/@dyunge_100/Network-TCPIP-4%EA%B3%84%EC%B8%B5%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC](https://velog.io/@dyunge_100/Network-TCPIP-4계층에-대하여)
7. TCP / IP 4계층 모델 - 핵심 총정리 - Inpa Dev ‍ - 티스토리, accessed July 5, 2025, [https://inpa.tistory.com/entry/WEB-%F0%9F%8C%90-TCP-IP-%EC%A0%95%EB%A6%AC-%F0%9F%91%AB%F0%9F%8F%BD-TCP-IP-4%EA%B3%84%EC%B8%B5](https://inpa.tistory.com/entry/WEB-🌐-TCP-IP-정리-👫🏽-TCP-IP-4계층)
8. [Network] TCP/IP 프로토콜 스택 (근데 이제 OSI 7계층을 곁들인) - [ Devrookie ] - 티스토리, accessed July 5, 2025, https://haeunyah.tistory.com/85
9. TCP/IP 4계층 모델, accessed July 5, 2025, [https://velog.io/@blessoms2017/TCPIP-4%EA%B3%84%EC%B8%B5-%EB%AA%A8%EB%8D%B8](https://velog.io/@blessoms2017/TCPIP-4계층-모델)
10. [Network] TCP(Transmission Control Protocol)란 무엇일까? - devkobe24.com, accessed July 5, 2025, https://www.devkobe24.com/Network/2024/2024-10-12-what-is-the-tcp.html
11. 인터넷 프로토콜 스위트 - 위키백과, 우리 모두의 백과사전, accessed July 5, 2025, [https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%EB%84%B7_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C_%EC%8A%A4%EC%9C%84%ED%8A%B8](https://ko.wikipedia.org/wiki/인터넷_프로토콜_스위트)
12. [네트워크] TCP/IP 프로토콜 스택 4계층 - velog, accessed July 5, 2025, [https://velog.io/@nayoon-kim/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-TCPIP-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C-%EC%8A%A4%ED%83%9D-4%EA%B3%84%EC%B8%B5](https://velog.io/@nayoon-kim/네트워크-TCPIP-프로토콜-스택-4계층)
13. TCP/IP란 무엇이며 어떤 원리로 작동하나요? - NordVPN, accessed July 5, 2025, https://nordvpn.com/ko/blog/tcp-ip-protocol/
14. TCP (Transmission Control Protocol) - Dev studying blog - 티스토리, accessed July 5, 2025, https://dev-studyingblog.tistory.com/8
15. [Network] TCP 프로토콜 이란? - 공대냥이 - 티스토리, accessed July 5, 2025, https://itragdoll.tistory.com/m/57?category=746355
16. 연결지향형 전송계층 프로토콜 : TCP - 까망눈연구소 - 티스토리, accessed July 5, 2025, https://wogh8732.tistory.com/26
17. 연결 지향 통신이란 무엇일까? TCP 와 connection Oriented - 프로그램잉 - 티스토리, accessed July 5, 2025, https://pro-gramm-ing.tistory.com/475
18. [정리] TCP 프로토콜 - 시나브로, accessed July 5, 2025, https://hyemsinabro.tistory.com/156
19. [TCP] 신뢰성 있는 데이터 전송의 원리 - velog, accessed July 5, 2025, [https://velog.io/@jeongbeom4693/TCP-%EC%8B%A0%EB%A2%B0%EC%84%B1-%EC%9E%88%EB%8A%94-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%A0%84%EC%86%A1%EC%9D%98-%EC%9B%90%EB%A6%AC](https://velog.io/@jeongbeom4693/TCP-신뢰성-있는-데이터-전송의-원리)
20. [Network] TCP (1) : Connection과 Handshake, 그리고 TCP Error ..., accessed July 5, 2025, https://sjh9708.tistory.com/193
21. 9. 연결지향형 TCP 프로토콜 - 각수의 창고, accessed July 5, 2025, [https://gaksu.tistory.com/entry/9-%EC%97%B0%EA%B2%B0%EC%A7%80%ED%96%A5%ED%98%95-TCP-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C](https://gaksu.tistory.com/entry/9-연결지향형-TCP-프로토콜)
22. QUIC is not Quick Enough over Fast Internet : r/programming - Reddit, accessed July 5, 2025, https://www.reddit.com/r/programming/comments/1g7vv66/quic_is_not_quick_enough_over_fast_internet/
23. [네트워크] TCP 3 way handshake & 4 way handshake - 항상 끈기있게 - 티스토리, accessed July 5, 2025, [https://nayoungs.tistory.com/entry/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-TCP-3-way-handshake-4-way-handshake](https://nayoungs.tistory.com/entry/네트워크-TCP-3-way-handshake-4-way-handshake)
24. 전송 제어 프로토콜 - 위키백과, 우리 모두의 백과사전, accessed July 5, 2025, [https://ko.wikipedia.org/wiki/%EC%A0%84%EC%86%A1_%EC%A0%9C%EC%96%B4_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C](https://ko.wikipedia.org/wiki/전송_제어_프로토콜)
25. TCP(Transmission Control Protocol), accessed July 5, 2025, https://zigispace.net/1259
26. ch09 TCP 프로토콜 / 연습문제 9장 - dma - 티스토리, accessed July 5, 2025, https://choimungu.tistory.com/49
27. TCP 3-Way 및 4-Way Handshake - velog, accessed July 5, 2025, [https://velog.io/@destiny1616/TCP-3-Way-%EB%B0%8F-4-Way-Handshake](https://velog.io/@destiny1616/TCP-3-Way-및-4-Way-Handshake)
28. TCP 3-way HandShake & 4-way HandShake - 개발일G - 티스토리, accessed July 5, 2025, https://ghs4593.tistory.com/18
29. [네트워크] | TCP 3 way handshake & 4 way handshake - velog, accessed July 5, 2025, [https://velog.io/@alkwen0996/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-TCP-3-way-handshake-4-way-handshake](https://velog.io/@alkwen0996/네트워크-TCP-3-way-handshake-4-way-handshake)
30. [네트워크] TCP 3-way & 4-way handshake란? - 조무래기 코딩 - 티스토리, accessed July 5, 2025, https://seongonion.tistory.com/74
31. [Network] TCP, 3-Way Handshake & 4-Way Handshake - Haengsin - 티스토리, accessed July 5, 2025, https://haengsin.tistory.com/47
32. TCP 3-WAY Handshake - Cisco Community, accessed July 5, 2025, https://community.cisco.com/t5/networking-blogs/tcp-3-way-handshake/ba-p/3773721
33. 쉽게 배우는 데이터 통신과 컴퓨터 네트워크 연습문제 9장 (TCP 프로토콜), accessed July 5, 2025, https://korean-otter.tistory.com/129
34. TCP ( 흐름 제어, 오류 제어) - 하루에 한 문제 - 티스토리, accessed July 5, 2025, https://dkwjdi.tistory.com/135
35. [Network]신뢰적인 데이터 전송과 TCP(1), accessed July 5, 2025, https://frog-in-well.tistory.com/48
36. 7.5. TCP Analysis - Wireshark, accessed July 5, 2025, https://www.wireshark.org/docs/wsug_html_chunked/ChAdvTCPAnalysis.html
37. [컴퓨터 네트워크] 23. Transport Layer (9) : TCP Congestion Control ..., accessed July 5, 2025, https://blog.everdu.com/378
38. TCP 혼잡 제어 - velog, accessed July 5, 2025, [https://velog.io/@mu1616/TCPIP-%ED%98%BC%EC%9E%A1-%EC%A0%9C%EC%96%B4](https://velog.io/@mu1616/TCPIP-혼잡-제어)
39. [TCP] 혼잡제어 - velog, accessed July 5, 2025, [https://velog.io/@jeongbeom4693/TCP-%ED%98%BC%EC%9E%A1%EC%A0%9C%EC%96%B4](https://velog.io/@jeongbeom4693/TCP-혼잡제어)
40. 사이 좋게 네트워크를 나눠 쓰는 방법, TCP의 혼잡 제어, accessed July 5, 2025, https://evan-moon.github.io/2019/11/26/tcp-congestion-control/
41. TCP congestion control - Wikipedia, accessed July 5, 2025, https://en.wikipedia.org/wiki/TCP_congestion_control
42. TCP 혼잡제어 - 도리의 디지털라이프, accessed July 5, 2025, [https://blog.skby.net/tcp-%ED%98%BC%EC%9E%A1%EC%A0%9C%EC%96%B4/](https://blog.skby.net/tcp-혼잡제어/)
43. [Network]TCP 계층 혼잡제어 완벽 정리 - 개구리의 개발 블로그, accessed July 5, 2025, https://frog-in-well.tistory.com/54
44. TCP 혼잡 방지 알고리즘 - 위키백과, 우리 모두의 백과사전, accessed July 5, 2025, [https://ko.wikipedia.org/wiki/TCP_%ED%98%BC%EC%9E%A1_%EB%B0%A9%EC%A7%80_%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98](https://ko.wikipedia.org/wiki/TCP_혼잡_방지_알고리즘)
45. TCP 혼잡제어 - [정보통신기술용어해설], accessed July 5, 2025, http://www.ktword.co.kr/test/view/view.php?no=5536
46. Linux Tuning - Fasterdata, accessed July 5, 2025, https://fasterdata.es.net/host-tuning/linux/
47. Linux TCP Tuning, accessed July 5, 2025, http://www.linux-admins.net/2010/09/linux-tcp-tuning.html
48. Analysis and Evaluation of TCP Congestion Algorithms - NetDev conference, accessed July 5, 2025, https://netdevconf.info/2.2/papers/brakmo-tcpcongestionanalysis-talk.pdf
49. TCP Congestion Control Algorithm Using Queueing Theory-Based Optimality Equation, accessed July 5, 2025, https://www.researchgate.net/publication/387911210_TCP_Congestion_Control_Algorithm_Using_Queueing_Theory-Based_Optimality_Equation
50. Exploring the BBRv2 Congestion Control Algorithm for use on Data Transfer Nodes - Internet2, accessed July 5, 2025, https://internet2.edu/wp-content/uploads/2022/12/techex22-AdvancedNetworking-ExploringtheBBRv2CongestionControlAlgorithm-Tierney.pdf
51. How Google Reinvented TCP for Faster Video Streaming - DEV Community, accessed July 5, 2025, https://dev.to/danielsuhett/how-google-reinvented-tcp-for-faster-video-streaming-1815
52. TCP BBR congestion control comes to GCP – your Internet just got faster - Google Cloud, accessed July 5, 2025, https://cloud.google.com/blog/products/networking/tcp-bbr-congestion-control-comes-to-gcp-your-internet-just-got-faster
53. Understanding Google's BBR for Last Mile delivery - Compira labs, accessed July 5, 2025, https://www.compiralabs.com/post/understanding-google-s-bbr-the-last-mile
54. BBR: Congestion-Based Congestion Control - ACM Queue, accessed July 5, 2025, https://queue.acm.org/detail.cfm?id=3022184
55. BBR TCP (Bottleneck Bandwidth and RTT) - GÉANT federated confluence, accessed July 5, 2025, https://wiki.geant.org/pages/viewpage.action?pageId=121340614
56. BBR Congestion Control - IETF, accessed July 5, 2025, https://www.ietf.org/archive/id/draft-cardwell-iccrg-bbr-congestion-control-01.html
57. BBR vs. BBRv2: A Performance Evaluation - Stony Brook Computer Science, accessed July 5, 2025, https://www3.cs.stonybrook.edu/~anshul/comsnets24_bbrbbrv2.pdf
58. SUSS: Improving TCP Performance by Speeding Up Slow-Start - Stanford Computer Science, accessed July 5, 2025, https://cs.stanford.edu/~keithw/sigcomm2024/sigcomm24-final280-acmpaginated.pdf
59. Bufferbloat - Wikipedia, accessed July 5, 2025, https://en.wikipedia.org/wiki/Bufferbloat
60. What is Bufferbloat? How Does Bufferbloat Impact Network Performance and User Experience? What are the Common Causes or Sources of Bufferbloat in a Network? What is Excessive Buffering? What is the Excessive Buffering Role in Bufferbloat? What Situations or Network Scenarios Is Bufferbloat More Likely to Occur? How Does Bufferbloat Impact OPNsense? How Does Bufferbloat Impact, accessed July 5, 2025, https://www.zenarmor.com/docs/network-basics/what-is-bufferbloat
61. Introduction - Bufferbloat.net, accessed July 5, 2025, https://www.bufferbloat.net/projects/bloat/wiki/Introduction/
62. Bufferbloat – what is it and why you (or your vendor) should care - Mushroom Networks, accessed July 5, 2025, https://www.mushroomnetworks.com/blog/bufferbloat-what-is-it-and-why-you-or-your-vendor-should-care/
63. What is buffer bloat - Off Topic - Netduma Forum, accessed July 5, 2025, https://forum.netduma.com/topic/9335-what-is-buffer-bloat/
64. TCP head of line blocking | HTTP/3 explained - README, accessed July 5, 2025, https://http3-explained.haxx.se/en/why-quic/why-tcphol
65. The Road to QUIC - The Cloudflare Blog, accessed July 5, 2025, https://blog.cloudflare.com/the-road-to-quic/
66. www.catchpoint.com, accessed July 5, 2025, [https://www.catchpoint.com/http2-vs-http3/quic-vs-tcp#:~:text=QUIC%20enables%200%2DRTT%20(zero,while%20TCP%20handles%20streams%20sequentially.](https://www.catchpoint.com/http2-vs-http3/quic-vs-tcp#:~:text=QUIC enables 0-RTT (zero,while TCP handles streams sequentially.)
67. QUIC vs. TCP-Development and Monitoring Guide - Catchpoint, accessed July 5, 2025, https://www.catchpoint.com/http2-vs-http3/quic-vs-tcp
68. What Is a SYN Flood Attack? - F5, accessed July 5, 2025, https://www.f5.com/glossary/syn-flood-attack
69. SYN Flood Attack: The What, Impact, and Prevention Methods, accessed July 5, 2025, https://www.indusface.com/blog/what-is-syn-synchronize-attack-how-the-attack-works-and-how-to-prevent-the-syn-attack/
70. SYN flood DDoS attack - Cloudflare, accessed July 5, 2025, https://www.cloudflare.com/learning/ddos/syn-flood-ddos-attack/
71. What is a SYN flood attack? - NetScout Systems, accessed July 5, 2025, https://www.netscout.com/what-is-ddos/syn-flood-attacks
72. What Are TCP SYN Flood DDOS Attacks & 6 Ways to Stop Them - Radware, accessed July 5, 2025, https://www.radware.com/security/ddos-knowledge-center/ddospedia/tcp-flood/
73. What Are SYN Flood DDoS Attacks? - Akamai, accessed July 5, 2025, https://www.akamai.com/glossary/what-are-syn-flood-ddos-attacks
74. What is Session Hijacking | Types, Detection & Prevention - Imperva, accessed July 5, 2025, https://www.imperva.com/learn/application-security/session-hijacking/
75. Session Hijacking Exploiting TCP, UDP and HTTP Sessions - Infopoint Security, accessed July 5, 2025, https://www.infopoint-security.de/open_downloads/alt/SessionHijacking.pdf
76. Session hijacking attack - OWASP Foundation, accessed July 5, 2025, https://owasp.org/www-community/attacks/Session_hijacking_attack
77. What is session hijacking and how does it work? - Kaspersky, accessed July 5, 2025, https://www.kaspersky.com/resource-center/definitions/what-is-session-hijacking
78. What Is Session Hijacking and How Do You Prevent It? - Keeper Security, accessed July 5, 2025, https://www.keepersecurity.com/blog/2024/04/03/what-is-the-best-way-to-prevent-session-hijacking/
79. Linux sysctl Tuning - NVIDIA Enterprise Support Portal, accessed July 5, 2025, https://enterprise-support.nvidia.com/s/article/linux-sysctl-tuning
80. Optimizing Linux Like a Pro: The SysAdmin's Guide to sysctl | by am | IT Security In Plain English | Medium, accessed July 5, 2025, https://medium.com/it-security-in-plain-english/optimizing-linux-like-a-pro-the-sysadmins-guide-to-sysctl-f4d4ce43d0fd
81. Tuning Linux Server Performance with Sysctl - HostMyCode, accessed July 5, 2025, https://www.hostmycode.in/tutorials/tuning-linux-server-performance-with-sysctl
82. TCP 3-Way Handshake, HTTP Protocol, and Packet Capture with Wireshark - Medium, accessed July 5, 2025, https://medium.com/@gulsahyarar/tcp-3-way-handshake-http-protocol-and-packet-capture-with-wireshark-7115ae62f7aa
83. TCP_3_way_handshaking - Wireshark Wiki, accessed July 5, 2025, https://wiki.wireshark.org/TCP_3_way_handshaking
84. How to capture tcp 3 way handshake - Wireshark Q&A, accessed July 5, 2025, https://osqa-ask.wireshark.org/questions/15057/how-to-capture-tcp-3-way-handshake/
85. Beyond QUIC v1 – A First Look at Recent Transport Layer IETF Standardization Efforts - arXiv, accessed July 5, 2025, https://arxiv.org/pdf/2102.07527
86. QUIC vs TCP: Which is Better? - Fastly, accessed July 5, 2025, https://www.fastly.com/blog/measuring-quic-vs-tcp-computational-efficiency
87. Beyond QUIC v1: A First Look at Recent Transport Layer IETF Standardization Efforts - Vaibhav Bajpai, accessed July 5, 2025, https://vaibhavbajpai.com/documents/papers/proceedings/quic-commag-2021.pdf
88. draft-ietf-quic-multipath-14 - Multipath Extension for QUIC - IETF Datatracker, accessed July 5, 2025, https://datatracker.ietf.org/doc/draft-ietf-quic-multipath/
89. Multipath TCP - Wikipedia, accessed July 5, 2025, https://en.wikipedia.org/wiki/Multipath_TCP
90. MultiPath TCP: From Theory to Practice., accessed July 5, 2025, https://dl.ifip.org/db/conf/networking/networking2011-1/BarrePB11.pdf
91. Understanding Multipath TCP: High availability for endpoints and the networking highway of the future - Red Hat, accessed July 5, 2025, https://www.redhat.com/en/blog/understanding-multipath-tcp-networking-highway-future
92. Using Multipath TCP to better survive outages and increase bandwidth - Red Hat, accessed July 5, 2025, https://www.redhat.com/en/blog/using-multipath-tcp-better-survive-outages-and-increase-bandwidth
93. Multipath TCP - ACM Queue, accessed July 5, 2025, https://queue.acm.org/detail.cfm?id=2591369
94. RFC 8684: TCP Extensions for Multipath Operation with Multiple ..., accessed July 5, 2025, https://www.rfc-editor.org/rfc/rfc8684.html
95. RFC 8684 - TCP Extensions for Multipath Operation with Multiple Addresses - IETF Datatracker, accessed July 5, 2025, https://datatracker.ietf.org/doc/html/rfc8684
96. draft-an-multipath-quic-application-policy-00 - IETF Datatracker, accessed July 5, 2025, https://datatracker.ietf.org/doc/html/draft-an-multipath-quic-application-policy-00
97. Multipath QUIC, accessed July 5, 2025, https://multipath-quic.org/
98. draft-dawkins-quic-multipath-questions-01 - IETF Datatracker, accessed July 5, 2025, https://datatracker.ietf.org/doc/html/draft-dawkins-quic-multipath-questions-01
99. In-progress version of draft-ietf-quic-multipath - GitHub, accessed July 5, 2025, https://github.com/quicwg/multipath
100. draft-ietf-quic-multipath-11 - IETF Datatracker, accessed July 5, 2025, https://datatracker.ietf.org/doc/html/draft-ietf-quic-multipath-11
101. QUIC (quic) - IETF Datatracker, accessed July 5, 2025, https://datatracker.ietf.org/wg/quic/about/
102. Writeups for draft-ietf-tsvwg-l4s-arch-20 - IETF Datatracker, accessed July 5, 2025, https://datatracker.ietf.org/doc/draft-ietf-tsvwg-l4s-arch/writeup/
103. RFC 9330 - Low Latency, Low Loss, and Scalable Throughput (L4S) Internet Service: Architecture - IETF Datatracker, accessed July 5, 2025, https://datatracker.ietf.org/doc/rfc9330/
104. RFC 9330: Low Latency, Low Loss, and Scalable Throughput (L4S) Internet Service: Architecture - » RFC Editor, accessed July 5, 2025, https://www.rfc-editor.org/rfc/rfc9330.html
105. draft-ietf-tsvwg-l4s-arch-11 - Low Latency, Low Loss, Scalable Throughput (L4S) Internet Service: Architecture - IETF Datatracker, accessed July 5, 2025, https://datatracker.ietf.org/doc/draft-ietf-tsvwg-l4s-arch/11/
106. RFC 9331: The Explicit Congestion Notification (ECN) Protocol for ..., accessed July 5, 2025, https://www.rfc-editor.org/rfc/rfc9331.html
107. RFC 9332: Dual-Queue Coupled Active Queue Management (AQM) for Low Latency, Low Loss, and Scalable Throughput (L4S) - » RFC Editor, accessed July 5, 2025, https://www.rfc-editor.org/rfc/rfc9332.html
108. To switch or not to switch to TCP Prague? Incentives for adoption in a partial L4S deployment - arXiv, accessed July 5, 2025, https://arxiv.org/html/2407.00464v1
109. draft-ietf-tsvwg-l4sops-07 - Operational Guidance on Coexistence with Classic ECN during L4S Deployment, accessed July 5, 2025, https://datatracker.ietf.org/doc/draft-ietf-tsvwg-l4sops/
110. heistp/l4s-tests: Tests of L4S - GitHub, accessed July 5, 2025, https://github.com/heistp/l4s-tests
111. P4sim: Programming Protocol-independent Packet Processors in ns-3 - arXiv, accessed July 5, 2025, https://arxiv.org/html/2503.17554v1
112. Your All-In-One Guide to P4 (Programming Protocol-Independent Packet Processors), accessed July 5, 2025, https://www.trentonsystems.com/en-us/resource-hub/blog/your-all-in-one-guide-to-p4
113. What is P4 Programming? And why is it important for edge computing? | Whitebox Solutions, accessed July 5, 2025, https://www.whiteboxsolution.com/blog/what-is-p4-programming-and-why-is-it-important-for-edge-computing/
114. A Review of P4 Programmable Data Planes for Network Security - Wiley Online Library, accessed July 5, 2025, https://onlinelibrary.wiley.com/doi/10.1155/2021/1257046
115. What is eBPF? An Introduction and Deep Dive into the eBPF Technology, accessed July 5, 2025, https://ebpf.io/what-is-ebpf/
116. eBPF Ecosystem Progress in 2024–2025: A Technical Deep Dive - eunomia, accessed July 5, 2025, https://eunomia.dev/blog/2025/02/12/ebpf-ecosystem-progress-in-20242025-a-technical-deep-dive/
117. Chapter 45. Understanding the eBPF networking features in RHEL 8, accessed July 5, 2025, https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/assembly_understanding-the-ebpf-features-in-rhel-8_configuring-and-managing-networking
118. TCP's Third Eye: Leveraging eBPF for Telemetry-Powered Congestion Control - Theo Jepsen, accessed July 5, 2025, https://theojepsen.dk/papers/tcpthirdeye-ebpf2023.pdf




# 개방형 시스템 상호접속(OSI) 7계층 참조 모델에 대한 고찰

## 서론

### 0.1 OSI 참조 모델의 탄생과 철학

1970년대 후반, 컴퓨터 네트워크 기술은 폭발적으로 성장하였으나 심각한 성장통을 겪고 있었다. IBM의 SNA(Systems Network Architecture), DEC의 DECnet 등 주요 제조사들은 각자의 독자적인 프로토콜 체계를 고수하였고, 이로 인해 서로 다른 시스템 간의 통신, 즉 상호운용성(Interoperability)은 거의 불가능에 가까웠다.1 이러한 기술적 파편화는 '프로토콜 전쟁(Protocol Wars)'이라 불릴 만큼 극심했으며, 네트워크의 통합적 발전을 저해하는 근본적인 장벽으로 작용했다.3 효율적인 데이터 교환을 위한 공통의 규약, 즉 표준의 필요성이 절실하게 대두된 시점이었다.1

이러한 혼란을 종식시키고 특정 제조사나 기술에 종속되지 않는 "개방형 시스템(Open Systems)" 간의 자유로운 통신을 실현하기 위해 국제표준화기구(ISO)가 나섰다.2 그 노력의 결실로, 1983년 국제전신전화자문위원회(CCITT)와의 공동 작업을 거쳐 1984년, 'OSI 기본 참조 모델(The Basic Reference Model for Open Systems Interconnection)', 즉 ISO/IEC 7498-1 표준이 세상에 공개되었다.3

OSI 모델의 핵심 설계 철학은 복잡성의 분해, 즉 '계층화(Layering)'에 있다. 네트워크 통신이라는 거대하고 복잡한 과업을, 명확하게 정의된 특정 기능을 수행하는 7개의 독립적인 계층으로 분할한 것이다.3 이 계층화는 다음과 같은 엄격한 원칙에 따라 이루어졌다.11

- 서로 다른 수준의 추상화(Abstraction)가 필요한 지점에 계층의 경계를 설정한다.
- 각 계층은 명확히 정의된 고유의 기능을 수행해야 한다.
- 계층 간의 경계는 인터페이스를 통한 정보의 흐름을 최소화하는 방향으로 선택한다.
- 계층의 수는 기능들이 불필요하게 하나의 계층에 통합되지 않을 만큼 충분히 많아야 하며, 동시에 전체 아키텍처가 비대해지지 않을 만큼 적절해야 한다.

이러한 계층적 접근 방식은 각 계층이 독립적인 모듈처럼 기능하게 하여, 한 계층의 기술적 수정이나 발전이 다른 계층에 미치는 영향을 최소화하는 '모듈화'의 이점을 제공한다.1 또한, 네트워크 장애 발생 시 문제의 원인을 특정 계층으로 국소화하여 진단하고 해결하는 과정을 획기적으로 단순화시킨다.6 더 나아가, 하위 계층의 복잡한 물리적, 전기적 구현 세부 사항을 추상화함으로써 상위 계층의 개발자들이 해당 기능에만 집중할 수 있도록 하는 강력한 개발 환경을 제공한다.16

이 모델의 탄생 배경을 더 깊이 들여다보면, 이는 단순한 기술적 해법을 넘어선다. 1970년대 네트워크 시장을 지배하던 특정 기업의 독점적 프로토콜에 대한 산업계 전반의 견제 심리가 작용했다. 즉, 모든 참여자가 공평하게 경쟁할 수 있는 '개방된' 운동장을 만들려는 산업-정치적 요구가 있었던 것이다. ISO라는 국제 '표준화' 기구가 위원회를 통해 이상적인 모델을 먼저 설계하고 현실 세계가 이를 따르도록 유도하는 '하향식(Top-down)' 접근 방식을 채택한 것은 이러한 배경과 무관하지 않다.3 이는 '법적 표준(de jure)'을 지향하는 OSI의 철학을 보여주며, 이후 실제 구현과 경험을 통해 표준을 정립해 나간 TCP/IP의 '사실상 표준(de facto)' 철학과 뚜렷한 대조를 이룬다. 결국 OSI 모델은 기술적 필요성과 더불어, 질서정연한 표준 체계를 통해 기술 발전을 이끌고자 했던 이상주의적 철학이 결합된 시대의 산물이라 할 수 있다.

### 0.2 참조 모델(Reference Model)로서의 위상

OSI 모델은 특정 프로토콜의 집합체가 아니다. 이는 모든 형태의 네트워크 통신 과정을 설명하고, 설계하며, 이해하기 위한 '개념적 프레임워크(Conceptual Framework)'이자 '참조 모델(Reference Model)'이다.3 이 모델은 각 계층이 *수행해야 할 과업(Task)*을 정의할 뿐, 그 과업을 수행하기 위해 

*어떤 프로토콜을 사용해야 하는지*를 강제하지 않는다.8 이러한 프로토콜 독립성(Protocol-Independent)은 모델의 유연성과 범용성을 보장하는 핵심 요소이다.

결과적으로 OSI 모델은 전 세계 네트워크 엔지니어, 소프트웨어 개발자, 시스템 관리자, 그리고 교육자들에게 복잡한 네트워크 시스템을 체계적으로 구성하고, 논의하며, 교육하기 위한 '공통 언어(Universal Language)'로서의 역할을 수행하게 되었다.3 이 공통의 틀을 통해 전문가들은 특정 시스템에 대한 사전 지식이 없더라도 복잡한 네트워크 아키텍처를 신속하게 이해하고, 설계하며, 장애 발생 시 논리적으로 원인을 추적할 수 있는 강력한 기반을 갖추게 된 것이다.14

## 1.  데이터 전송의 거시적 흐름: 캡슐화와 프로토콜 데이터 단위

### 1.1 캡슐화(Encapsulation)와 역캡슐화(Decapsulation)

네트워크를 통한 데이터 전송은 마치 컨베이어 벨트 위의 조립 라인과 같다. 송신 측에서 사용자가 생성한 원본 데이터(예: 이메일 본문)는 최상위 계층인 응용 계층에서 시작하여 물리 계층을 향해 한 단계씩 아래로 이동하는 여정을 거친다.20

이 과정에서 '캡슐화(Encapsulation)'라는 핵심적인 작업이 수행된다. 각 계층(Layer N)은 바로 위 상위 계층(Layer N+1)으로부터 전달받은 데이터 단위를 온전한 화물, 즉 '서비스 데이터 단위(SDU, Service Data Unit)'로 취급한다. 그리고 이 SDU에 자신의 계층에서 통제를 위해 필요한 정보, 즉 '프로토콜 제어 정보(PCI, Protocol Control Information)'를 덧붙인다. 이 제어 정보는 주로 데이터의 앞부분에 붙는 **헤더(Header)** 형태이며, 경우에 따라 뒷부분에 **트레일러(Trailer)**가 추가되기도 한다.3 이렇게 SDU와 PCI가 결합하여 만들어진 새로운 데이터 단위를 해당 계층의 **프로토콜 데이터 단위(PDU, Protocol Data Unit)**라고 부른다. 이 PDU는 다시 바로 아래 하위 계층(Layer N-1)으로 전달되는데, 이때 하위 계층의 입장에서는 이 PDU 전체가 자신의 SDU가 된다.3 이러한 캡슐화 과정은 데이터가 최하위 물리 계층에 도달할 때까지 반복된다.12

수신 측에서는 이 모든 과정이 정확히 역순으로 진행된다. 이를 '역캡슐화(Decapsulation)'라고 한다. 물리 계층에서 수신한 전기적 신호는 비트 스트림으로 변환되어 데이터 링크 계층으로 전달된다. 데이터 링크 계층은 프레임 헤더와 트레일러를 해석하여 오류 여부를 확인하고, 제어 정보를 제거한 뒤 내부의 패킷(자신의 SDU)을 네트워크 계층으로 올린다. 각 계층은 자신에게 해당하는 헤더를 분석 및 처리하고, 제어 정보를 제거한 후 페이로드를 상위 계층으로 전달하는 과정을 응용 계층까지 반복한다. 이 과정을 통해 수신자는 마침내 송신자가 보낸 원본 데이터를 얻게 된다.12

### 1.2 프로토콜 데이터 단위(PDU)의 이해

PDU는 OSI 모델의 각 계층에서 통신하는 상대방, 즉 동일 계층의 엔티티(Peer Entity) 간에 교환되는 정보의 기본 논리적 단위이다.3 이는 상위 계층으로부터 받은 SDU(실제 데이터)와 해당 계층의 프로토콜이 필요로 하는 제어 정보(PCI)의 조합으로 구성된다.3

데이터는 각 계층을 통과하면서 캡슐화 과정을 통해 서로 다른 이름의 PDU로 불리게 된다. 이는 각 계층의 역할과 처리하는 정보의 형태가 다르기 때문이다.30

- **제7, 6, 5계층 (응용, 표현, 세션):** 데이터(Data) 또는 메시지(Message) 28
- **제4계층 (전송):** 세그먼트(Segment) (TCP 사용 시) 또는 데이터그램(Datagram) (UDP 사용 시) 23
- **제3계층 (네트워크):** 패킷(Packet) 23
- **제2계층 (데이터 링크):** 프레임(Frame) 10
- **제1계층 (물리):** 비트(Bit) 또는 심볼(Symbol) 3

캡슐화 과정은 단순히 데이터를 포장하는 행위를 넘어선다. 이는 복잡한 '종단 간 데이터 전송'이라는 과업을 각 계층의 명확한 책임 단위로 분할하고 위임하는 '분산 책임 위임' 메커니즘으로 이해할 수 있다. 각 계층은 오직 자신의 헤더 정보만을 해석하고 처리할 뿐, 그 안에 담긴 상위 계층의 데이터(SDU) 내용에는 관여하지 않는다.10 예를 들어, 네트워크 계층에서 동작하는 라우터는 패킷의 IP 헤더를 보고 최적의 경로를 결정하는 임무에만 충실하며, 그 패킷 안에 어떤 TCP 세그먼트가 들어있는지는 알 필요가 없다. 이처럼 각 계층에 "이것이 당신이 처리해야 할 제어 정보(헤더)이며, 나머지는 당신의 책임이 아니니 그대로 하위 계층으로 전달하라"는 명확한 지침을 부여하는 것과 같다. 이 메커니즘 덕분에 각 계층의 프로토콜과 관련 장비들은 서로 독립적으로 개발되고 최적화될 수 있으며, 전체 네트워크 시스템의 모듈성과 확장성이 극대화된다. 이는 현대 소프트웨어 공학의 핵심 원리인 '관심사의 분리(Separation of Concerns)'가 네트워크 아키텍처에 정교하게 구현된 대표적인 사례라 할 수 있다.

### 표 1: OSI 7계층별 PDU, 주소 체계 및 관련 장비 요약

| 계층 | 명칭             | PDU (Protocol Data Unit)                   | 주요 주소 체계               | 관련 장비                             |
| ---- | ---------------- | ------------------------------------------ | ---------------------------- | ------------------------------------- |
| 7    | 응용 계층        | 데이터 (Data) / 메시지 (Message)           | 특정 주소 없음 (서비스 식별) | L7 스위치, 게이트웨이                 |
| 6    | 표현 계층        | 데이터 (Data) / 메시지 (Message)           | 특정 주소 없음               | 게이트웨이                            |
| 5    | 세션 계층        | 데이터 (Data) / 메시지 (Message)           | 특정 주소 없음               | 게이트웨이                            |
| 4    | 전송 계층        | 세그먼트 (Segment) / 데이터그램 (Datagram) | 포트 번호 (Port Number)      | L4 스위치, 게이트웨이                 |
| 3    | 네트워크 계층    | 패킷 (Packet)                              | 논리 주소 (IP Address)       | 라우터 (Router), L3 스위치            |
| 2    | 데이터 링크 계층 | 프레임 (Frame)                             | 물리 주소 (MAC Address)      | 브리지 (Bridge), L2 스위치            |
| 1    | 물리 계층        | 비트 (Bit) / 심볼 (Symbol)                 | 주소 없음                    | 리피터 (Repeater), 허브 (Hub), 케이블 |

이 표는 OSI 모델의 추상적인 계층 구조를 PDU, 주소, 장비라는 세 가지 구체적인 요소와 연결하여, 데이터가 각 계층을 거치며 어떻게 변환되고 어떤 정보에 의해 제어되며 어떤 장치에 의해 처리되는지에 대한 구조적 이해를 돕는다.10

## 2.  OSI 7계층 심층 분석

OSI 7계층은 크게 두 그룹으로 나눌 수 있다. 하위 3개 계층(물리, 데이터 링크, 네트워크)은 '미디어 계층(Media Layers)'으로, 데이터의 물리적 전송과 네트워크 간 라우팅을 담당한다. 상위 4개 계층(전송, 세션, 표현, 응용)은 '호스트 계층(Host Layers)'으로, 종단 시스템 간의 안정적인 통신과 사용자 애플리케이션 지원을 책임진다.3 이러한 구분은 데이터의 '전달 방식(How)'에 집중하는 영역과 데이터의 '의미와 활용(What)'에 집중하는 영역으로 자연스럽게 나뉘는 구조적 특성을 보여준다. 이는 네트워크 엔지니어와 애플리케이션 개발자의 주된 관심 영역이 분리되는 지점이기도 하다.41

### 2.1 하위 계층 (Media Layers): 데이터 전송의 물리적 및 논리적 기반

#### 2.1.1 제1계층: 물리 계층 (Physical Layer)

물리 계층은 네트워크 통신의 가장 근간을 이루는 단계로, 추상적인 디지털 데이터를 물리 세계의 신호로 변환하는 역할을 한다.

- **핵심 기능:** 컴퓨터 내부에서 사용되는 디지털 데이터, 즉 `$0$`과 `$1$`의 나열인 비트(Bit)를 케이블, 광섬유, 공기 등 물리적 매체를 통해 전송할 수 있는 형태의 신호(예: 전기 신호, 광 신호, 전파)로 변환(인코딩)하고, 수신 측에서는 반대로 신호를 비트로 복원(디코딩)하는 모든 과정을 책임진다.3 이 계층은 데이터의 내용, 구조, 오류 여부 등에는 전혀 관여하지 않으며, 오직 비트 스트림을 충실히 전달하는 임무만을 수행한다.34

- **주요 표준 및 규격:** 이 계층의 표준은 물리적 연결에 필요한 모든 것을 정의한다. 케이블의 종류(UTP, 동축, 광섬유), 커넥터의 기계적 형태(RJ-45 등), 핀의 배열과 각각의 기능, 신호의 전압 레벨, 케이블의 임피던스와 같은 전기적 특성을 규정한다.3 또한, 초당 전송할 수 있는 비트의 수(데이터 전송률), 신호의 타이밍, 송수신 측의 클럭을 맞추는 비트 동기화, 그리고 단방향(Simplex), 반이중(Half-duplex), 전이중(Full-duplex)과 같은 데이터 전송 모드도 이 계층에서 정의된다.3

- **PDU:** 비트(Bit) 또는 심볼(Symbol).3 다만, 일부에서는 PDU라는 구조화된 단위보다는 연속적인 신호의 흐름으로 간주하는 것이 더 정확하다고 보기도 한다.38

- **관련 프로토콜 및 장비:** 이더넷(Ethernet) 프로토콜의 물리적 규격 부분, RS-232, USB, Bluetooth, NFC 등이 대표적인 프로토콜에 해당한다.10 물리 계층에서 동작하는 장비로는 감쇠된 신호를 원래의 강도로 증폭시켜 전송 거리를 늘리는 

  **리피터(Repeater)**, 여러 장치를 연결하기 위한 다중 포트 리피터인 **허브(Hub)**, 그리고 신호를 변조하고 복조하는 **모뎀(Modem)** 및 각종 **케이블**이 있다.10

#### 2.1.2 제2계층: 데이터 링크 계층 (Data Link Layer)

물리 계층이 단순히 신호를 전달하는 역할이라면, 데이터 링크 계층은 이 신호를 의미 있는 단위로 묶고, 가까운 장치 간에 신뢰성 있는 정보를 전달하는 책임을 진다.

- **핵심 기능:** 동일한 네트워크 대역(Local Area Network, LAN) 내에서, 즉 라우터를 거치지 않고 직접 연결된 **인접 노드(Adjacent Node) 간**의 데이터 전송을 관리한다. 이를 '홉-바이-홉(Hop-by-hop)' 전송이라고도 한다.6 이 계층의 가장 중요한 목표는 불안정한 물리 계층에서 발생할 수 있는 전송 오류를 감지하고 수정하여, 상위 계층인 네트워크 계층에 오류 없는 깨끗한 전송로를 제공하는 것이다.29
- **주요 기능 상세:**
  - **프레이밍(Framing):** 네트워크 계층으로부터 전달받은 패킷의 앞뒤에 헤더와 트레일러를 붙여 **프레임(Frame)**이라는 논리적인 데이터 전송 단위를 생성한다.29
  - **물리적 주소 지정(Physical Addressing):** 모든 네트워크 인터페이스 카드(NIC)에 제조 시 부여되는 전 세계적으로 유일한 48비트 하드웨어 주소인 **MAC(Media Access Control) 주소**를 사용한다. 프레임 헤더에 출발지와 목적지의 MAC 주소를 기록하여 동일 네트워크 내의 특정 장치를 정확히 찾아간다.23
  - **오류 제어(Error Control):** 프레임의 끝부분(트레일러)에 **FCS(Frame Check Sequence)** 또는 **CRC(Cyclic Redundancy Check)**라는 오류 검출 코드를 추가한다. 수신 측은 프레임을 수신한 후 동일한 계산을 수행하여 이 값을 비교함으로써 전송 중 데이터 손상 여부를 확인한다.10 오류가 발견되면 해당 프레임은 폐기된다.
  - **흐름 제어(Flow Control):** 수신 측의 데이터 처리 능력이 송신 측의 전송 속도를 따라가지 못해 데이터가 유실되는 것을 막기 위해, 데이터의 전송량이나 속도를 조절하는 기능을 수행한다.10
  - **접근 제어(Access Control):** 하나의 통신 매체를 여러 장치가 공유하는 환경(예: 초기 이더넷, 무선랜)에서, 두 개 이상의 장치가 동시에 데이터를 전송하려 할 때 발생하는 충돌(Collision)을 방지하고 제어하는 규칙을 제공한다. 이더넷의 CSMA/CD(Carrier Sense Multiple Access with Collision Detection)가 대표적인 예이다.
- **하위 계층(Sublayers):** 데이터 링크 계층은 논리적 연결과 물리적 접근을 분리하기 위해 종종 두 개의 하위 계층으로 나뉜다.16
  - **LLC(Logical Link Control):** 상위 계층(네트워크 계층)과의 인터페이스 역할을 하며, 프로토콜을 식별하고 흐름 제어 및 오류 제어 기능을 담당한다.
  - **MAC(Media Access Control):** 매체에 대한 접근을 제어하고, MAC 주소를 이용한 물리적 주소 지정을 담당한다.
- **PDU:** 프레임(Frame).3
- **관련 프로토콜 및 장비:** LAN 환경의 **이더넷(Ethernet)**, WAN 환경의 **PPP(Point-to-Point Protocol)**, HDLC, 무선 환경의 **IEEE 802.11(WiFi)** 등이 대표적인 프로토콜이다.16 이 계층에서 동작하는 주요 장비로는 MAC 주소를 학습하여 프레임을 특정 포트로만 전달하는 **스위치(Switch)**와, 두 개의 네트워크 세그먼트를 연결하는 **브리지(Bridge)**가 있다.23

#### 2.1.3 제3계층: 네트워크 계층 (Network Layer)

네트워크 계층은 데이터가 최종 목적지에 도달하기까지의 전체 여정을 책임지는 '교통 관제사'와 같다.

- **핵심 기능:** 서로 다른 네트워크들을 넘나들며 데이터 패킷을 출발지 호스트에서 최종 목적지 호스트까지 전달하는, 즉 **종단 간(End-to-End) 전송**을 위한 경로를 설정(Routing)하고, 패킷을 해당 경로를 따라 중계(Forwarding)하는 역할을 수행한다.12 데이터 링크 계층이 '이웃집에 편지 배달'이라면, 네트워크 계층은 '서울에서 뉴욕으로 소포 보내기'에 비유할 수 있다.

- **주요 기능 상세:**

  - **논리적 주소 지정(Logical Addressing):** 전 세계 인터넷에서 각 컴퓨터를 고유하게 식별하기 위한 논리적 주소 체계인 **IP(Internet Protocol) 주소**를 사용한다. 패킷 헤더에 출발지와 최종 목적지의 IP 주소를 기록하여 전체 인터넷 망에서의 위치를 특정한다.10
  - **라우팅(Routing):** 네트워크의 상태 정보를 수집하는 라우팅 프로토콜(예: OSPF, BGP)을 통해 라우팅 테이블을 생성하고, 이를 바탕으로 패킷을 목적지까지 전달하기 위한 최적의 경로를 동적으로 결정한다.23
  - **패킷화(Packetizing):** 상위 계층인 전송 계층으로부터 받은 세그먼트에 IP 헤더를 붙여 **패킷(Packet)**이라는 전송 단위를 만든다.33
  - **분할 및 재조립(Fragmentation & Reassembly):** 패킷이 통과해야 할 하위 데이터 링크 계층의 전송 단위(MTU, Maximum Transmission Unit)보다 패킷의 크기가 클 경우, 이를 여러 개의 작은 조각으로 분할하여 전송하고, 최종 목적지 시스템의 네트워크 계층에서 이 조각들을 다시 원래의 패킷으로 재조립한다.25

- **PDU:** 패킷(Packet) 또는 데이터그램(Datagram).3

- **관련 프로토콜 및 장비:** 현재 인터넷의 핵심인 **IP(IPv4, IPv6)**, 네트워크 오류 진단 및 보고를 위한 **ICMP**, 논리 주소(IP)를 물리 주소(MAC)로 변환하는 **ARP** 등이 주요 프로토콜이다.3 이 계층의 대표적인 장비는 네트워크 간의 경로를 설정하고 패킷을 중계하는 **라우터(Router)**와, 라우팅 기능을 수행하는 고성능 스위치인 

  **L3 스위치**이다.12

### 2.2 상위 계층 (Host Layers): 사용자와 애플리케이션을 위한 통신 서비스

#### 2.2.1 제4계층: 전송 계층 (Transport Layer)

전송 계층은 하위 계층이 마련한 데이터 전달 서비스를 바탕으로, 실제 응용 프로그램 간의 논리적 연결을 맺고 데이터 전송의 신뢰성을 책임지는 역할을 한다.

- **핵심 기능:** 컴퓨터 대 컴퓨터 통신을 넘어, 양 끝단(End-to-End) 시스템에서 실행 중인 **프로세스(Process) 대 프로세스** 간의 논리적 통신 채널을 제공한다.47 이를 위해 각 응용 프로그램을 식별하는 **포트 번호(Port Number)**를 사용한다.29 이 계층은 데이터가 목적지에 정확하고 신뢰성 있게 도달하도록 보장하는 핵심적인 제어 기능들을 수행한다.30

- **주요 서비스 유형:**

  - **연결 지향형 서비스 (Connection-Oriented): TCP (Transmission Control Protocol)**
    - **특징:** 데이터를 전송하기 전에 송수신 측 간에 논리적인 연결을 설정하는 '3-way-handshake' 과정을 거치며, 데이터 전송이 완료되면 연결을 해제한다. 이 과정을 통해 매우 높은 신뢰성을 보장한다.23
    - **기능:** 대용량 데이터를 **세그먼트(Segment)**라는 단위로 분할하고 각 세그먼트에 순서 번호를 부여하여 수신 측에서 올바른 순서로 재조립할 수 있게 한다(순서 제어). 또한, 데이터 수신 확인(Acknowledgement)과 체크섬(Checksum)을 통해 전송 오류를 감지하고, 유실된 세그먼트를 재전송하여 오류를 복구한다(오류 제어). 수신 측의 버퍼 상태에 따라 전송량을 조절하는 흐름 제어(Sliding Window)와 네트워크 혼잡 상태에 따라 전송 속도를 조절하는 혼잡 제어 기능도 수행한다.34 웹 브라우징, 파일 전송, 이메일 등 데이터의 무결성이 중요한 서비스에 사용된다.
  - **비연결 지향형 서비스 (Connectionless): UDP (User Datagram Protocol)**
    - **특징:** 연결 설정 과정 없이 데이터를 **데이터그램(Datagram)** 단위로 전송한다. 신뢰성을 보장하지 않는 대신, 헤더가 단순하고 절차가 간소하여 전송 속도가 매우 빠르다. 이를 '최선 노력(Best-Effort)' 서비스라고 한다.23
    - **기능:** 출발지/목적지 포트 번호, 길이, 오류 검출을 위한 체크섬 등 최소한의 정보만을 헤더에 담는다. 순서 제어, 오류 복구, 흐름 제어 기능이 없어 데이터가 손실되거나 순서가 뒤바뀌어 도착할 수 있다.23 실시간 스트리밍, 온라인 게임, DNS 조회 등 속도가 신뢰성보다 중요한 서비스에 적합하다.

- **PDU:** 세그먼트(Segment) 또는 데이터그램(Datagram).3

- **관련 프로토콜 및 장비:** **TCP**, **UDP**, SCTP(Stream Control Transmission Protocol)가 대표적인 프로토콜이다.3 포트 번호를 기반으로 트래픽을 분산시키는 

  **L4 스위치**나 다양한 프로토콜 변환을 수행하는 **게이트웨이**가 이 계층과 관련이 있다.12

#### 2.2.2 제5계층: 세션 계층 (Session Layer)

세션 계층은 응용 프로그램 간의 '대화'를 조직하고 관리하는 역할을 한다.

- **핵심 기능:** 양 끝단의 응용 프로그램들이 통신을 시작하고, 지속하며, 종료하기까지의 일련의 과정, 즉 **통신 세션(Session)**을 수립, 관리, 동기화 및 종료하는 절차를 제공한다.23 데이터 교환을 위한 논리적인 대화 채널을 만들고 제어하는 것이다.33
- **주요 기능 상세:**
  - **세션 수립, 유지, 종료:** 두 응용 프로그램 간의 논리적 연결을 설정하고, 통신이 진행되는 동안 이를 안정적으로 유지하며, 통신이 완료되면 세션을 정상적으로 해제한다.32
  - **대화 제어(Dialogue Control):** 데이터 교환의 순서와 방식을 관리한다. 데이터를 양방향으로 동시에 전송할 수 있는 전이중(Full-duplex) 방식, 또는 한 번에 한쪽 방향으로만 전송하는 반이중(Half-duplex) 방식을 결정하고 제어한다.27
  - **동기화(Synchronization):** 대용량 데이터 전송 시, 데이터 스트림 중간중간에 **동기점(Synchronization Point)** 또는 체크포인트(Checkpoint)를 삽입한다.32 만약 전송 중에 오류가 발생하여 연결이 끊어지면, 처음부터가 아닌 마지막으로 성공한 동기점부터 데이터 전송을 재개할 수 있어 통신의 효율성을 크게 향상시킨다.
- **PDU:** 데이터(Data) 또는 메시지(Message).3
- **관련 프로토콜 및 장비:** **NetBIOS**, **RPC(Remote Procedure Call)**, PPTP, **SSH**, **TLS** 등이 이 계층의 기능을 수행하는 프로토콜들이다.3 특정 하드웨어 장비보다는 운영체제나 응용 프로그램의 일부로 소프트웨어적으로 구현되는 경우가 대부분이다.

#### 2.2.3 제6계층: 표현 계층 (Presentation Layer)

표현 계층은 서로 다른 환경의 시스템들이 데이터를 동일한 의미로 이해할 수 있도록 '통역'과 '번역'을 담당한다.

- **핵심 기능:** 한 시스템의 응용 계층에서 사용하는 데이터 표현 형식을, 네트워크를 통해 전송하기 위한 공통된 표준 형식으로 변환하거나, 수신 측에서 이를 다시 해당 시스템의 응용 계층이 이해할 수 있는 형식으로 변환하는 역할을 한다.10
- **주요 기능 상세:**
  - **데이터 변환 및 형식 지정(Translation/Formatting):** 서로 다른 문자 인코딩(예: ASCII, EBCDIC)을 사용하는 시스템 간에 데이터를 교환할 수 있도록 코드를 변환해준다.10 또한, 정수나 부동소수점 수를 표현하는 방식(예: Big-endian, Little-endian)의 차이를 해결하여 데이터의 의미가 왜곡되지 않도록 한다.
  - **암호화 및 복호화(Encryption/Decryption):** 데이터의 기밀성을 보장하기 위해 전송 전에 데이터를 암호화하고, 수신 측에서 이를 해독하여 원래의 데이터로 복원한다. **SSL(Secure Sockets Layer)/TLS(Transport Layer Security)** 프로토콜이 대표적인 예로, 이 계층에서 기능한다고 볼 수 있다.33
  - **데이터 압축(Compression):** 전송되는 데이터의 크기를 줄여 네트워크 대역폭을 효율적으로 사용하기 위해 데이터를 압축하고, 수신 측에서 압축을 해제하는 기능을 수행한다.32
- **PDU:** 데이터(Data) 또는 메시지(Message).3
- **관련 프로토콜 및 장비:** **JPEG, MPEG, GIF**와 같은 멀티미디어 압축 표준, **ASCII, EBCDIC**과 같은 문자 인코딩 표준, 그리고 **SSL/TLS**, XDR(External Data Representation) 등이 이 계층과 관련 깊다.3 세션 계층과 마찬가지로, 특정 하드웨어보다는 운영체제나 응용 프로그램의 라이브러리 형태로 구현된다.

#### 2.2.4 제7계층: 응용 계층 (Application Layer)

응용 계층은 OSI 모델의 최상위에 위치하며, 사용자가 네트워크와 직접 소통하는 창구 역할을 한다.

- **핵심 기능:** 최종 사용자가 네트워크 자원에 접근하고 다양한 네트워크 서비스를 이용할 수 있도록 인터페이스와 프로토콜을 제공한다.9 우리가 매일 사용하는 웹 브라우저, 이메일 클라이언트, 파일 전송 프로그램 등 모든 네트워크 응용 프로그램은 이 계층의 프로토콜 위에서 동작한다.
- **주요 기능 상세:**
  - **네트워크 서비스 제공:** 웹 페이지를 주고받기 위한 **HTTP/HTTPS**, 파일 전송을 위한 **FTP**, 이메일 송수신을 위한 **SMTP/POP3/IMAP**, 원격 터미널 접속을 위한 **Telnet/SSH**, 그리고 도메인 이름을 IP 주소로 변환해주는 **DNS** 등 사용자에게 친숙한 대부분의 네트워크 서비스가 이 계층에서 정의되고 제공된다.10
  - **사용자 인터페이스:** 응용 프로그램은 이 계층을 통해 사용자와 상호작용하며, 사용자의 요청(예: 웹사이트 주소 입력)을 하위 계층으로 전달하여 실제 네트워크 통신을 시작하는 관문 역할을 한다.10
  - **서비스 품질(QoS) 요구:** 응용 프로그램의 특성(예: 실시간 영상 스트리밍, 대용량 파일 전송)에 따라 하위 계층에 특정 수준의 서비스 품질(예: 낮은 지연 시간, 높은 신뢰성)을 요구하는 기능을 포함할 수 있다.42
- **PDU:** 데이터(Data) 또는 메시지(Message).28
- **관련 프로토콜 및 장비:** **HTTP, HTTPS, FTP, SMTP, POP3, IMAP, DNS, Telnet, SSH, SNMP** 등 매우 다양한 프로토콜이 존재한다.3 이 계층에서 동작하는 특수한 장비로는 애플리케이션의 트래픽을 분석하여 부하를 분산시키는 **L7 스위치(ADC, Application Delivery Controller)**나 서로 다른 프로토콜을 변환해주는 **게이트웨이(Gateway)**가 있다.47

### 2.3 표 2: 각 계층별 주요 프로토콜 및 기능 상세 비교

| 계층               | 핵심 기능                            | 주요 프로토콜                | 프로토콜의 역할 예시                                         |
| ------------------ | ------------------------------------ | ---------------------------- | ------------------------------------------------------------ |
| **7. 응용**        | 사용자에게 네트워크 서비스 제공      | HTTP, FTP, SMTP, DNS         | **HTTP:** 웹 브라우저가 웹 서버에 웹 페이지를 요청하고 응답받는 통신 규약 |
| **6. 표현**        | 데이터 형식 변환, 암호화, 압축       | SSL/TLS, JPEG, MPEG, ASCII   | **SSL/TLS:** 전송되는 데이터를 암호화하여 제3자가 엿볼 수 없도록 함 |
| **5. 세션**        | 통신 세션 수립, 관리, 종료           | NetBIOS, RPC, SSH            | **SSH:** 원격 컴퓨터에 안전하게 접속하고 명령을 실행하기 위한 세션을 설정하고 유지 |
| **4. 전송**        | 프로세스 간 신뢰성 있는 데이터 전송  | TCP, UDP                     | **TCP:** 데이터를 세그먼트로 분할하고, 순서 번호를 부여하며, 오류 발생 시 재전송하여 신뢰성 보장 |
| **3. 네트워크**    | 데이터의 최종 목적지 경로 설정       | IP, ICMP, OSPF, BGP          | **IP:** 데이터 패킷에 출발지/목적지 IP 주소를 부여하고, 라우터가 이 주소를 보고 경로를 결정 |
| **2. 데이터 링크** | 인접 노드 간 신뢰성 있는 프레임 전송 | Ethernet, PPP, MAC           | **Ethernet:** MAC 주소를 사용하여 동일 네트워크 내의 장치로 프레임을 전달하고, CRC로 오류 검출 |
| **1. 물리**        | 비트 스트림의 물리적 전송            | Ethernet (물리 규격), RS-232 | **Ethernet(케이블):** 디지털 비트(0,1)를 UTP 케이블을 통해 전송 가능한 전기 신호로 변환 |

이 표는 OSI 모델의 각 계층이 수행하는 추상적인 기능을 실제 인터넷을 구동하는 구체적인 프로토콜의 역할과 연결하여, 이론과 실제 사이의 간극을 메우고 네트워크 동작 원리에 대한 심층적인 이해를 돕는다.38

## 3.  이론과 실제의 간극: OSI 모델과 TCP/IP 모델 비교 분석

OSI 모델이 네트워크 통신의 이상적인 청사진을 제시했다면, 오늘날 인터넷을 실제로 움직이는 것은 TCP/IP 모델이다. 두 모델의 차이점을 이해하는 것은 이론적 지식과 현실 세계의 기술을 연결하는 데 매우 중요하다.

### 3.1 TCP/IP 모델의 구조와 특징

TCP/IP 모델은 1970년대 미 국방부(DoD)의 연구 프로젝트인 ARPANET에서 탄생했다.9 그 목적은 이론적 완결성보다는 당장 동작하는 견고하고 신뢰성 있는 네트워크를 구축하는 데 있었으며, 이러한 실용주의적 배경은 모델의 구조와 특징에 깊이 반영되어 있다.

- **계층 구조:** OSI 모델의 7계층 구조보다 훨씬 단순화된 4계층 또는 5계층 구조를 가진다.23
  - **4계층 모델 (전통적 모델):** 네트워크 접속 계층(Network Access), 인터넷 계층(Internet), 전송 계층(Transport), 응용 계층(Application)으로 구성된다.7
  - **5계층 모델 (현대적 모델):** 4계층 모델의 네트워크 접속 계층을 OSI 모델과 유사하게 데이터 링크 계층과 물리 계층으로 다시 분리한 형태로, 현대의 네트워크 교육에서 더 널리 사용된다.20
- **프로토콜 중심:** OSI 모델이 프로토콜에 독립적인 참조 모델인 것과 달리, TCP/IP 모델은 그 이름에서 알 수 있듯이 **TCP(Transmission Control Protocol)**와 **IP(Internet Protocol)**라는 두 개의 핵심 프로토콜을 중심으로 구성된 실제 프로토콜의 집합체, 즉 '프로토콜 스위트(Protocol Suite)' 그 자체이다.17

### 3.2 두 모델의 비교

OSI 모델과 TCP/IP 모델은 계층 구조, 설계 철학, 표준화 과정 등 여러 측면에서 뚜렷한 차이를 보인다.

- **계층 구조 매핑:**
  - OSI의 상위 3개 계층(응용, 표현, 세션)의 기능은 TCP/IP 모델에서는 하나의 **응용 계층**으로 통합된다. 이는 TCP/IP가 응용 프로그램 개발자에게 세세한 기능 분리보다는 유연성을 제공하는 데 초점을 맞췄기 때문이다.7
  - OSI의 **전송 계층**과 TCP/IP의 **전송 계층**은 역할과 기능 면에서 거의 동일하며, 종단 간 통신과 신뢰성을 담당한다.7
  - OSI의 **네트워크 계층**은 TCP/IP의 **인터넷 계층**과 대응되며, IP 주소를 이용한 라우팅을 책임진다.7
  - OSI의 하위 2개 계층(데이터 링크, 물리)은 TCP/IP의 **네트워크 접속 계층**(또는 링크 계층)으로 묶인다. 이는 TCP/IP가 특정 물리적 네트워크 기술에 종속되지 않고 다양한 기술 위에서 동작할 수 있도록 설계되었기 때문이다.7
- **설계 철학 및 접근 방식:**
  - **OSI:** 프로토콜에 독립적인(Protocol-Independent) 이론적 참조 모델로, '통신은 이렇게 이루어져야 한다'는 규범적(Prescriptive) 접근을 취한다. 서비스, 인터페이스, 프로토콜의 역할을 명확하게 구분하는 엄격한 구조를 가진다.9
  - **TCP/IP:** 이미 성공적으로 동작하고 있는 프로토콜들을 기반으로 한 실용적 모델로, '인터넷은 이렇게 동작한다'는 기술적(Descriptive) 접근을 따른다. OSI만큼 계층 간의 역할 구분이 엄격하지 않고, 유연성을 더 중시한다.7
- **표준화:**
  - **OSI:** ISO와 CCITT라는 국제 표준화 기구가 주도하여 만든 공식적인 법적 표준(de jure standard)이다.75
  - **TCP/IP:** ARPANET과 인터넷 커뮤니티(IETF)에서의 광범위한 사용과 검증을 통해 자연스럽게 시장의 표준으로 자리 잡은 사실상의 표준(de facto standard)이다.13

두 모델을 '승자'와 '패자'의 이분법적 관계로 보는 것은 피상적인 분석에 불과하다. 실제로는 상호 보완적인 관계에 가깝다. TCP/IP가 인터넷이라는 '실제로 지어진 건물'이라면, OSI 모델은 그 건물의 구조를 이해하고, 문제점을 진단하며, 더 나은 구조를 구상하게 해주는 '건축 설계도' 또는 '건축학 원론'과 같다. 현대의 네트워크 전문가는 실제 환경(TCP/IP)에서 발생하는 문제를 해결하기 위해, OSI 모델이 제공하는 명확한 계층 구분이라는 '사고의 틀'을 빌려와 "이것은 L3(네트워크 계층) 문제" 또는 "L7(응용 계층) 보안 취약점"과 같이 체계적으로 소통하고 분석한다.6 즉, OSI는 TCP/IP를 이해하고 분석하는 강력한 '사고의 틀'을 제공하며, 이론과 실제는 현장에서 함께 활용된다.

### 3.3 표 3: OSI 7계층과 TCP/IP 4계층 모델의 비교

| 구분               | OSI 모델                                                     | TCP/IP 모델                                                  |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **개발 주체/목적** | ISO/CCITT, 이론적 완결성을 갖춘 범용 통신 참조 모델 개발 4   | 미 국방부(DoD), ARPANET의 실용적이고 견고한 통신 프로토콜 개발 68 |
| **계층 수**        | 7계층 (응용, 표현, 세션, 전송, 네트워크, 데이터 링크, 물리) 76 | 4계층 (응용, 전송, 인터넷, 네트워크 접속) 70                 |
| **접근 방식**      | 하향식(Top-down), 규범적(Prescriptive), 프로토콜 독립적 9    | 상향식(Bottom-up), 기술적(Descriptive), 프로토콜 의존적 7    |
| **계층 매핑**      | 응용/표현/세션 ↔ 응용전송 ↔ 전송네트워크 ↔ 인터넷데이터링크/물리 ↔ 네트워크 접속 7 |                                                              |
| **신뢰성 보장**    | 전송 계층에서 연결형/비연결형 서비스 모두 정의 78            | 전송 계층에서 신뢰성 있는 TCP와 비신뢰성 UDP를 명확히 제공 23 |
| **주요 사용처**    | 네트워크 원리 교육, 시스템 설계, 장애 진단을 위한 참조 모델 3 | 현재 인터넷 및 대부분의 컴퓨터 네트워크에서 사용되는 실질적인 표준 9 |

이 비교표는 두 모델의 표면적인 차이를 넘어, 그 탄생 배경과 설계 철학에서 비롯된 근본적인 차이점을 조명한다. 이를 통해 왜 실용성을 앞세운 TCP/IP가 인터넷의 표준이 되었는지, 그럼에도 불구하고 왜 이론적 명확성을 갖춘 OSI 모델이 교육과 분석의 영역에서 여전히 절대적인 권위를 갖는지를 입체적으로 이해할 수 있다.

## 4.  현대 컴퓨팅 환경에서의 OSI 모델의 유효성 고찰

OSI 모델이 제정될 당시에는 상상하기 어려웠던 웹, 클라우드 컴퓨팅, 사물인터넷(IoT)과 같은 현대 기술 환경 속에서도 OSI 모델은 여전히 그 유효성을 잃지 않고 있다. 오히려 복잡한 신기술을 분석하고 체계화하는 강력한 '사고의 스캐폴딩(Scaffolding, 발판)' 역할을 수행하고 있다.

### 4.1 사례 연구: 웹 브라우징 과정 분석

사용자가 웹 브라우저 주소창에 `https://www.example.com`을 입력하고 엔터 키를 누르는 순간부터 웹 페이지가 화면에 표시되기까지의 복잡한 과정을 OSI 7계층 모델에 대입하면, 각 계층의 역할이 명확하게 드러난다.79

**송신 측 (클라이언트) 데이터 생성 및 캡슐화:**

1. **제7계층 (응용 계층):** 사용자의 입력을 받은 웹 브라우저는 URL을 해석한다. 먼저 **DNS** 프로토콜을 사용하여 도메인 이름(`www.example.com`)에 해당하는 서버의 IP 주소를 알아낸다. 그 후, 해당 서버에 웹 페이지를 요청하기 위한 **HTTP** GET 요청 메시지를 생성한다.63
2. **제6계층 (표현 계층):** 생성된 HTTP 요청 메시지를 네트워크를 통해 전송할 수 있는 표준 형식(예: ASCII)으로 인코딩한다. HTTPS 통신이므로, 이 단계에서 **TLS/SSL** 프로토콜이 동작하여 HTTP 메시지를 암호화한다.9
3. **제5계층 (세션 계층):** 웹 서버와의 안전한 통신을 위한 세션을 수립한다. TLS 핸드셰이크 과정을 통해 암호화에 사용할 키와 알고리즘을 협상하고, 보안 세션을 설정한다.22
4. **제4계층 (전송 계층):** 암호화된 데이터를 **TCP** 프로토콜을 사용하여 **세그먼트(Segment)** 단위로 분할한다. 각 세그먼트에는 출발지 포트(브라우저가 사용하는 임의의 높은 번호 포트)와 목적지 포트(HTTPS의 표준 포트인 443) 정보가 담긴 TCP 헤더가 추가된다. 데이터 전송에 앞서, TCP 3-way-handshake를 통해 서버와 신뢰성 있는 연결을 먼저 수립한다.48
5. **제3계층 (네트워크 계층):** 각 TCP 세그먼트에 IP 헤더를 추가하여 **패킷(Packet)**을 만든다. IP 헤더에는 클라이언트 자신의 IP 주소(출발지)와 DNS 조회를 통해 얻은 서버의 IP 주소(목적지)가 기록된다.50
6. **제2계층 (데이터 링크 계층):** IP 패킷에 이더넷 헤더와 트레일러를 추가하여 **프레임(Frame)**을 생성한다. 이더넷 헤더에는 자신의 NIC 카드 MAC 주소(출발지)와 다음으로 패킷을 전달해야 할 장비, 즉 로컬 네트워크의 게이트웨이(라우터)의 MAC 주소(목적지)가 포함된다. 게이트웨이의 MAC 주소를 모를 경우, **ARP** 프로토콜을 사용하여 IP 주소에 해당하는 MAC 주소를 알아낸다.51
7. **제1계층 (물리 계층):** 완성된 프레임은 최종적으로 0과 1의 비트 스트림으로 변환되고, 이는 다시 전기 신호나 광 신호, 무선 신호로 바뀌어 물리적 매체(랜선, 광케이블, 공기)를 통해 전송된다.22

이 데이터는 인터넷상의 수많은 라우터를 거쳐 목적지 서버에 도달한다. 서버는 이 과정을 정확히 역순으로(L1부터 L7까지 역캡슐화) 수행하여 클라이언트의 HTTP 요청을 해석한다. 그리고 요청받은 웹 페이지 데이터(HTML, CSS, 이미지 파일 등)를 다시 L7부터 L1까지 캡슐화 과정을 거쳐 클라이언트에게 응답으로 전송한다. 클라이언트의 컴퓨터는 이 응답 데이터를 수신하여 역캡슐화하고, 최종적으로 웹 브라우저가 이를 해석하여 사용자에게 화려한 웹 페이지를 보여주게 된다.16

### 4.2 클라우드 컴퓨팅 및 사물인터넷(IoT) 환경에서의 적용

- **클라우드 컴퓨팅:** 클라우드 환경은 고도로 추상화되고 가상화되어 있지만, 그 근간은 OSI 모델로 설명 가능한 물리적 네트워크 인프라에 의존한다. IaaS, PaaS, SaaS와 같은 클라우드 서비스 모델은 OSI 계층의 특정 부분에 대한 관리 책임이 클라우드 제공자(CSP)와 고객 사이에 어떻게 분배되는지를 명확히 설명하는 데 유용한 틀을 제공한다. 예를 들어, IaaS(Infrastructure as a Service) 사용자는 L3(네트워크) 이상의 가상 머신과 네트워크를 제어하지만, L1-L2(물리적 인프라)는 전적으로 CSP의 책임 영역이다. 또한, 가상 사설 클라우드(VPC), 로드 밸런서(L4/L7에서 동작), 웹 애플리케이션 방화벽(WAF, L7에서 동작)과 같은 복잡한 클라우드 네트워킹 서비스를 설계하고 장애를 진단하는 데 있어 OSI 모델은 필수적인 분석 프레임워크를 제공한다.3

- **사물인터넷(IoT):** 수많은 IoT 장치는 배터리로 동작하고 처리 능력이 낮은 제한된 환경에서 통신해야 한다.84 이 때문에 전력 소모가 큰 TCP/IP 스택 전체를 사용하기보다는, 경량화된 프로토콜 스택을 채택하는 경우가 많다. 예를 들어, 저전력 무선 통신을 위한 

  **IEEE 802.15.4**(L1-L2), IPv6 패킷을 작은 프레임에 담기 위한 **6LoWPAN**(L3 적응 계층), 비신뢰성 저부하 전송을 위한 **UDP**(L4), 그리고 경량 응용 프로토콜인 **CoAP/MQTT**(L7) 등이 조합되어 사용된다. OSI 모델은 이처럼 생소한 IoT 프로토콜들이 전체 네트워크 스택에서 어느 계층의 역할을 수행하며, 기존 인터넷(TCP/IP)과 어떻게 상호 연동되는지를 체계적으로 분석하고 이해하는 데 결정적인 참조점을 제공한다.84

### 4.3 확장된 관점: 보안과 관리

OSI 모델의 가치는 기능 분리를 넘어 보안과 관리 영역으로 확장된다. ISO는 기본 참조 모델(ISO/IEC 7498-1)을 보완하기 위해 다음과 같은 추가 표준들을 발표했다.

- **보안 아키텍처 (ISO/IEC 7498-2):** 이 표준은 OSI 모델의 각 7개 계층에 적용될 수 있는 보안 서비스(인증, 접근 제어, 기밀성, 무결성 등)와 메커니즘을 정의한다.3 예를 들어, 네트워크 계층(L3)에서는 

  **IPsec**을 통해 패킷 단위의 인증과 암호화를, 표현 계층(L6)에서는 **TLS/SSL**을 통해 데이터 자체의 암호화를, 응용 계층(L7)에서는 사용자 접근 제어를 구현할 수 있다. 이처럼 OSI 모델은 다양한 네트워크 공격이 어느 계층을 표적으로 하는지 식별하고, 각 계층의 특성에 맞는 적절한 보안 통제를 체계적으로 배치하는 '심층 방어(Defense-in-Depth)' 전략의 이론적 기반을 제공한다.39

- **관리 프레임워크 (ISO/IEC 7498-4):** 이 표준은 네트워크의 성능, 장애, 구성, 계정, 보안(FCAPS)을 관리하기 위한 통합 프레임워크를 정의한다.3 응용 계층 프로토콜인 SNMP(Simple Network Management Protocol)가 어떻게 네트워크 장비들을 원격으로 모니터링하고 제어하는지 이해하는 데 OSI 모델은 유용한 배경지식을 제공한다.

이처럼 OSI 모델은 어떤 새로운 기술이나 복잡한 요구사항이 등장하더라도, 그것을 체계적으로 분해하고 기존의 기술 체계 안에 위치시켜 분석할 수 있는 강력한 '분류 체계'이자 '분석의 틀'로 작용한다. OSI 모델의 진정한 현대적 가치는 특정 프로토콜의 구현 여부가 아니라, 미지의 기술적 도전에 직면했을 때 문제의 본질을 꿰뚫어 보고 해결책을 설계할 수 있는 '사고의 발판'을 제공하는 데 있다. 이는 시대를 초월하는 분석 도구로서 OSI 모델의 역할을 증명한다.

## 5.  결론

### 5.1 OSI 참조 모델의 현대적 가치 재조명

오늘날 인터넷의 실제 구현은 TCP/IP 프로토콜 스위트가 확고히 지배하고 있으며, OSI 모델을 위해 설계되었던 OSI 프로토콜 자체는 시장에서 거의 찾아볼 수 없다.3 이러한 현실 때문에 OSI 모델이 시대에 뒤떨어진 비현실적인 이론에 불과하다는 비판이 제기되기도 한다.6

그러나 본 보고서에서 심층적으로 고찰한 바와 같이, OSI 모델의 진정한 가치는 그것의 직접적인 구현 여부에 있지 않다. 그 가치는 복잡하기 그지없는 네트워크 통신의 기본 원리를 7개의 논리적 계층으로 명쾌하게 분해하여 체계적으로 학습하고 교육할 수 있는 기반을 제공한다는 점에 있다.15 또한, 끊임없이 등장하는 새로운 프로토콜과 기술들을 공통의 프레임워크 위에서 비교하고 분석하며, 네트워크 장애 발생 시 문제의 원인을 논리적으로 추적하고 격리하는 데 필수적인 **개념적 도구(Conceptual Tool)**로서의 역할을 충실히 수행하고 있다.6

### 5.2 미래 네트워크를 향한 시사점

소프트웨어 정의 네트워킹(SDN), 네트워크 기능 가상화(NFV) 등 네트워크 패러다임이 하드웨어 중심에서 소프트웨어 중심으로 빠르게 전환되고, 인공지능과 양자 통신 등 예측하기 어려운 기술들이 네트워크에 접목될 미래 환경에서, 복잡한 시스템을 기능 단위로 분해하고 각 구성 요소의 상호작용을 명확히 정의하는 계층적 사고방식의 중요성은 오히려 더욱 커질 것이다.

OSI 7계층 참조 모델은 이러한 추상화와 모듈화의 원칙을 가장 성공적으로 체계화한 지적 자산이다. 따라서 이 모델은 미래의 네트워크 전문가들이 새로운 기술적 도전에 직면했을 때, 문제의 본질을 꿰뚫어 보고 창의적인 해결책을 설계하는 데 필요한 근본적인 통찰과 지적 기반을 계속해서 제공할 것이다. 결론적으로 OSI 모델은 과거의 유물이 아니라, 끊임없이 진화하는 네트워크 기술의 미래를 탐색하는 데 필요한 지혜가 담긴 불멸의 지도라 할 수 있다.

#### **참고 자료**

1. OSI 7계층에 대하여 - 감성코드 - 티스토리, 8월 14, 2025에 액세스, https://lietenant-k.tistory.com/116
2. [네트워크 계층 모델] OSI 7계층 & TCP/IP 4계층 모델 - 개발해도유 - 티스토리, 8월 14, 2025에 액세스, https://doyu-l.tistory.com/404
3. OSI model - Wikipedia, 8월 14, 2025에 액세스, https://en.wikipedia.org/wiki/OSI_model
4. mundol-colynn.tistory.com, 8월 14, 2025에 액세스, [https://mundol-colynn.tistory.com/167#:~:text=%EA%B2%B0%EB%A1%A0%EC%A0%81%EC%9C%BC%EB%A1%9C%2C%20OSI%207%EA%B3%84%EC%B8%B5,%EA%B8%B0%EB%B3%B8%20%EA%B5%AC%EC%A1%B0%EB%A1%9C%20%EC%B1%84%ED%83%9D%EB%90%98%EC%97%88%EC%8A%B5%EB%8B%88%EB%8B%A4.](https://mundol-colynn.tistory.com/167#:~:text=결론적으로%2C OSI 7계층,기본 구조로 채택되었습니다.)
5. OSI모델이란? - 개발 완벽정리, 8월 14, 2025에 액세스, [https://westahn.com/osi%EB%AA%A8%EB%8D%B8/](https://westahn.com/osi모델/)
6. OSI Model Full Form - Open System Interconnection - GeeksforGeeks, 8월 14, 2025에 액세스, https://www.geeksforgeeks.org/computer-networks/osi-model-full-form-in-computer-networking/
7. [Network 이론] OSI 모델과 TCP/IP 모델 비교 - IT 시작해보기 - 티스토리, 8월 14, 2025에 액세스, https://easyitwanner.tistory.com/373
8. What Is the OSI Model? - 7 OSI Layers Explained - AWS, 8월 14, 2025에 액세스, https://aws.amazon.com/what-is/osi-model/
9. What is OSI Model | 7 Layers Explained - Imperva, 8월 14, 2025에 액세스, https://www.imperva.com/learn/application-security/osi-model/
10. [CS] OSI 7계층 파헤치기 / OSI 7계층이란 / 계층별 역할, 기능, 8월 14, 2025에 액세스, https://mundol-colynn.tistory.com/167
11. The OSI Model: An Overview - GIAC Certifications, 8월 14, 2025에 액세스, https://www.giac.org/paper/gsec/1417/osi-model-overview/102634
12. OSI 7 계층 - 위키원, 8월 14, 2025에 액세스, [http://wiki.hash.kr/index.php/OSI_7_%EA%B3%84%EC%B8%B5](http://wiki.hash.kr/index.php/OSI_7_계층)
13. OSI 모델과 TCP/IP 모델 - Bruders - 티스토리, 8월 14, 2025에 액세스, https://bruders.tistory.com/110
14. OSI 모델이란 무엇인가요? - IBM, 8월 14, 2025에 액세스, https://www.ibm.com/kr-ko/think/topics/osi-model
15. [OSI 7계층, TCP/IP 4계층] 네트워크의 기본 계층 구조 - 초보자 전용 마을 - 티스토리, 8월 14, 2025에 액세스, https://ryusae.tistory.com/4
16. OSI 모델이란 무엇인가요?- OSI 7계층 설명 - AWS, 8월 14, 2025에 액세스, https://aws.amazon.com/ko/what-is/osi-model/
17. Differences: TCP/IP vs OSI Model | OrhanErgun.net Blog, 8월 14, 2025에 액세스, https://orhanergun.net/tcp-ip-vs-osi-model
18. What is the OSI Model? | Cloudflare, 8월 14, 2025에 액세스, https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/
19. What Is OSI Model? Open Systems Interconnection Definition | Proofpoint US, 8월 14, 2025에 액세스, https://www.proofpoint.com/us/threat-reference/osi-model
20. OSI 7계층과 TCP/IP 4계층을 비교 설명해 주세요., 8월 14, 2025에 액세스, https://www.nossi.dev/06936fc0-eb03-4a53-8e4f-2e5dc71508dd
21. What Is the OSI Model? | IBM, 8월 14, 2025에 액세스, https://www.ibm.com/think/topics/osi-model
22. [Networking] the OSI Model. what happens when you click on a URL? | by 施靜樺 - Medium, 8월 14, 2025에 액세스, https://medium.com/@jinghua.shih/networking-the-osi-model-4a508d84444
23. OSI 7계층(ISO Standard 7498) - Rubisco's Programming Note, 8월 14, 2025에 액세스, https://huimang2.github.io/etc/iso-standard-7498
24. [Network] OSI 7계층, SDU와 PDU - yeni code - 티스토리, 8월 14, 2025에 액세스, https://yeniful.tistory.com/38
25. The OSI Model: A Comprehensive Overview - DEV Community, 8월 14, 2025에 액세스, https://dev.to/m__mdy__m/the-osi-model-a-comprehensive-overview-52j5
26. Protocol data unit - Wikipedia, 8월 14, 2025에 액세스, https://en.wikipedia.org/wiki/Protocol_data_unit
27. [네린이 공부일기] OSI 7 계층 참조모델, PDU, 계층별 통신 장비 - O! JAVA, 8월 14, 2025에 액세스, https://ojava.tistory.com/201
28. OSI 7 계층이란 - 공부 기록, 8월 14, 2025에 액세스, https://lemonandgrapefruit.tistory.com/7
29. Network 이해의 시초, OSI 7계층 정리 - velog, 8월 14, 2025에 액세스, [https://velog.io/@mooh2jj/Network-%EC%9D%B4%ED%95%B4%EC%9D%98-%EC%8B%9C%EC%B4%88-OSI-7%EA%B3%84%EC%B8%B5-%EC%A0%95%EB%A6%AC](https://velog.io/@mooh2jj/Network-이해의-시초-OSI-7계층-정리)
30. Protocol Data Unit (PDU) - GeeksforGeeks, 8월 14, 2025에 액세스, https://www.geeksforgeeks.org/computer-networks/protocol-data-unit-pdu/
31. 프로토콜 데이터 단위 - 위키백과, 우리 모두의 백과사전, 8월 14, 2025에 액세스, [https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C_%EB%8D%B0%EC%9D%B4%ED%84%B0_%EB%8B%A8%EC%9C%84](https://ko.wikipedia.org/wiki/프로토콜_데이터_단위)
32. [네트워크] OSI 참조모델 - 항상 끈기있게 - 티스토리, 8월 14, 2025에 액세스, [https://nayoungs.tistory.com/entry/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-OSI-%EC%B0%B8%EC%A1%B0%EB%AA%A8%EB%8D%B8](https://nayoungs.tistory.com/entry/네트워크-OSI-참조모델)
33. [네트워크] OSI 7계층, PDU - 뼝아리 책장 - 티스토리, 8월 14, 2025에 액세스, [https://oozoowos.tistory.com/entry/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-OSI-7%EA%B3%84%EC%B8%B5-PDU](https://oozoowos.tistory.com/entry/네트워크-OSI-7계층-PDU)
34. OSI 7 계층과 TCP/IP 계층 - velog, 8월 14, 2025에 액세스, [https://velog.io/@inyong_pang/OSI-7-%EA%B3%84%EC%B8%B5%EA%B3%BC-TCPIP-%EA%B3%84%EC%B8%B5](https://velog.io/@inyong_pang/OSI-7-계층과-TCPIP-계층)
35. From Data to Frame: The Evolution of PDUs Across the OSI Model - DEV Community, 8월 14, 2025에 액세스, https://dev.to/stungnet/from-data-to-frame-the-evolution-of-pdus-across-the-osi-model-21gd
36. OSI 7계층(OSI 7Layer) 및 계층별 프로토콜, PDU - AndrewNA - 티스토리, 8월 14, 2025에 액세스, https://it-life.tistory.com/113
37. [네트워크] OSI 7계층과 TCP/IP - 채야미의 코드레시피 - 티스토리, 8월 14, 2025에 액세스, https://chaeyami.tistory.com/189
38. OSI 7계층 :: 불곰, 8월 14, 2025에 액세스, https://brownbears.tistory.com/189
39. What is the OSI Model? Understanding the 7 Layers - Check Point Software, 8월 14, 2025에 액세스, https://www.checkpoint.com/cyber-hub/network-security/what-is-the-osi-model-understanding-the-7-layers/
40. [네트워크] 네트워크의 기능 - OSI 7계층, 프로토콜과 인터페이스 - 개발자와 코더 사이, 8월 14, 2025에 액세스, [https://bmangrok.tistory.com/entry/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC%EC%9D%98-%EA%B8%B0%EB%8A%A5-OSI-7%EA%B3%84%EC%B8%B5-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4](https://bmangrok.tistory.com/entry/네트워크-네트워크의-기능-OSI-7계층-프로토콜-인터페이스)
41. [네트워크] OSI 7계층과 TCP/IP, 8월 14, 2025에 액세스, https://chaelin1211.github.io/study/2021/05/24/OSI-layer.html
42. The OSI Model: Understanding the Layered Approach to Network Communication - Splunk, 8월 14, 2025에 액세스, https://www.splunk.com/en_us/blog/learn/osi-model.html
43. 컴퓨터 네트워크: OSI 7 계층(Layer)과 TCP/IP 모델의 이해, 8월 14, 2025에 액세스, [https://oobwrite.com/entry/%EC%BB%B4%ED%93%A8%ED%84%B0-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-OSI-7-%EA%B3%84%EC%B8%B5Layer%EA%B3%BC-TCPIP-%EB%AA%A8%EB%8D%B8%EC%9D%98-%EC%9D%B4%ED%95%B4](https://oobwrite.com/entry/컴퓨터-네트워크-OSI-7-계층Layer과-TCPIP-모델의-이해)
44. OSI Network Model Basics: Layers, Devices, & Protocols - NewServerLife, 8월 14, 2025에 액세스, https://newserverlife.com/articles/osi-network-model-for-beginners-device-layers-protocols/
45. OSI 7 (ISO 7498) - Char - 티스토리, 8월 14, 2025에 액세스, https://charstring.tistory.com/1110
46. OSI model and its 7 layers explained | A1 Digital, 8월 14, 2025에 액세스, https://www.a1.digital/knowledge-hub/osi-model-and-its-7-layers-explained/
47. OSI 7 계층 - 다락방 - 티스토리, 8월 14, 2025에 액세스, https://hojunking.tistory.com/112
48. 1. OSI 7계층, 8월 14, 2025에 액세스, https://adrian0220.tistory.com/84
49. [Q&A] 네트워크 | OSI 7계층, 8월 14, 2025에 액세스, https://m.youtube.com/watch?v=w903b3QLkrQ&pp=ygULI29zaTfqs4TsuLU%3D
50. OSI 모델이란? | OSI 7계층 - Cloudflare, 8월 14, 2025에 액세스, https://www.cloudflare.com/ko-kr/learning/ddos/glossary/open-systems-interconnection-model-osi/
51. Data journey through the Internet - The OSI model approach - DEV Community, 8월 14, 2025에 액세스, https://dev.to/amaraiheanacho/data-journey-through-the-internet-the-osi-model-approach-1n4a
52. OSI 7계층 모델 - 핵심 총정리, 8월 14, 2025에 액세스, [https://inpa.tistory.com/entry/WEB-%F0%9F%8C%90-OSI-7%EA%B3%84%EC%B8%B5-%EC%A0%95%EB%A6%AC](https://inpa.tistory.com/entry/WEB-🌐-OSI-7계층-정리)
53. OSI Model: The 7 Layers of Network Architecture – BMC Software | Blogs, 8월 14, 2025에 액세스, https://www.bmc.com/blogs/osi-model-7-layers/
54. [네트워크] OSI 7 계층 - velog, 8월 14, 2025에 액세스, [https://velog.io/@orijoon98/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-OSI-7-%EA%B3%84%EC%B8%B5](https://velog.io/@orijoon98/네트워크-OSI-7-계층)
55. [정보처리기사] OSI-7계층, TCP/IP - Devinus - 티스토리, 8월 14, 2025에 액세스, https://devinus.tistory.com/29
56. [네트워크] 쉽게 이해하는 TCP/IP 4계층 - 개발공부 블로그, 8월 14, 2025에 액세스, https://weight-devlog.tistory.com/41
57. What Is Transmission Control Protocol TCP/IP? - Fortinet, 8월 14, 2025에 액세스, https://www.fortinet.com/resources/cyberglossary/tcp-ip
58. Understanding the TCP/IP Model: The Backbone of Internet Communication - Medium, 8월 14, 2025에 액세스, https://medium.com/@gwenilorac/understanding-the-tcp-ip-model-the-backbone-of-internet-communication-aaa69b5b6595
59. OSI 모델이란? 7 Layers - 체크 포인트 소프트웨어 이해 - Check Point, 8월 14, 2025에 액세스, https://www.checkpoint.com/kr/cyber-hub/network-security/what-is-the-osi-model-understanding-the-7-layers/
60. Understanding Layer 5: The Session Layer of the OSI Model - JumpCloud, 8월 14, 2025에 액세스, https://jumpcloud.com/it-index/understanding-layer-5-the-session-layer-of-the-osi-model
61. Understanding Layer 6: The Presentation Layer of the OSI Model - JumpCloud, 8월 14, 2025에 액세스, https://jumpcloud.com/it-index/understanding-layer-6-the-presentation-layer-of-the-osi-model
62. Open Systems Interconnect Reference Model (Network Interface Guide), 8월 14, 2025에 액세스, https://docs.oracle.com/cd/E19455-01/806-1017/6jab5di2d/index.html
63. Understanding the Application Layer of the OSI Model - Coursera, 8월 14, 2025에 액세스, https://www.coursera.org/articles/application-layer
64. 웹은 어떻게 작동할까? 네트워크 OSI 7계층 모델 - 모양 - 티스토리, 8월 14, 2025에 액세스, https://gwonran.tistory.com/29
65. TCP/IP 4계층 모델 - 계층 구조 - Ga0Lee - 티스토리, 8월 14, 2025에 액세스, [https://ga0lee.tistory.com/entry/TCPIP-4%EA%B3%84%EC%B8%B5-%EB%AA%A8%EB%8D%B8-%EA%B3%84%EC%B8%B5-%EA%B5%AC%EC%A1%B0](https://ga0lee.tistory.com/entry/TCPIP-4계층-모델-계층-구조)
66. [개념 콕] OSI 7계층 - 내일배움캠프 블로그, 8월 14, 2025에 액세스, [https://nbcamp.spartacodingclub.kr/blog/%EA%B0%9C%EB%85%90-%EC%BD%95-%EC%9B%B9-%EA%B0%9C%EB%B0%9C-%EC%A7%80%EC%8B%9D-%ED%8E%B8-osi-7%EA%B3%84%EC%B8%B5%EC%9D%98-%EA%B0%9C%EB%85%90-%EA%B3%84%EC%B8%B5%EB%B3%84-%ED%8A%B9%EC%A7%95-21174](https://nbcamp.spartacodingclub.kr/blog/개념-콕-웹-개발-지식-편-osi-7계층의-개념-계층별-특징-21174)
67. What is the TCP/IP Model? The Internet Protocol Suite - Simplilearn.com, 8월 14, 2025에 액세스, https://www.simplilearn.com/tutorials/cyber-security-tutorial/what-is-tcp-ip-model
68. Internet protocol suite - Wikipedia, 8월 14, 2025에 액세스, https://en.wikipedia.org/wiki/Internet_protocol_suite
69. TCP/IP 란? - 쫌쫌따리 공부하깅 - 티스토리, 8월 14, 2025에 액세스, [https://seokhee0516.tistory.com/entry/TCPIP-%EB%9E%80](https://seokhee0516.tistory.com/entry/TCPIP-란)
70. TCP/IP 모델의 이해와 OSI와 TCP/IP 비교 - 기억에서 기록까지 - 티스토리, 8월 14, 2025에 액세스, https://jjdevelop.tistory.com/5
71. www.geeksforgeeks.org, 8월 14, 2025에 액세스, [https://www.geeksforgeeks.org/computer-networks/tcp-ip-model/#:~:text=The%20TCP%2FIP%20model%20is,Network%2FInternet%20and%20Network%20Access.](https://www.geeksforgeeks.org/computer-networks/tcp-ip-model/#:~:text=The TCP%2FIP model is,Network%2FInternet and Network Access.)
72. TCP/IP Model - GeeksforGeeks, 8월 14, 2025에 액세스, https://www.geeksforgeeks.org/computer-networks/tcp-ip-model/
73. TCP/IP 와 TCP, IP 를 다르게 읽어야한다 - velog, 8월 14, 2025에 액세스, https://velog.io/@ksi05503/tcp-ip
74. OSI and TCP/IP model: Differences explained | A1 Digital, 8월 14, 2025에 액세스, https://www.a1.digital/knowledge-hub/osi-and-tcp-ip-model-differences-explained/
75. OSI 7 Layer와 TCP/IP 비교 - 도리의 디지털라이프, 8월 14, 2025에 액세스, [https://blog.skby.net/osi-7-layer%EC%99%80-tcp-ip-%EB%B9%84%EA%B5%90/](https://blog.skby.net/osi-7-layer와-tcp-ip-비교/)
76. OSI 모델 이해와 OSI 7계층의 이해 - 기억에서 기록까지 - 티스토리, 8월 14, 2025에 액세스, https://jjdevelop.tistory.com/4
77. orhanergun.net, 8월 14, 2025에 액세스, [https://orhanergun.net/tcp-ip-vs-osi-model#:~:text=Layer%20Structure%3A%20The%20most%20apparent,functions%20are%20categorized%20and%20implemented.](https://orhanergun.net/tcp-ip-vs-osi-model#:~:text=Layer Structure%3A The most apparent,functions are categorized and implemented.)
78. Difference Between OSI Model and TCP/IP Model - GeeksforGeeks, 8월 14, 2025에 액세스, https://www.geeksforgeeks.org/computer-networks/difference-between-osi-model-and-tcp-ip-model/
79. [#. ETC] 웹 브라우저에서 URL을 입력했을 때 과정(OSI 7계층), 8월 14, 2025에 액세스, https://developer0809.tistory.com/189
80. OSI 7계층과 TCP/IP 4계층 모델 | 모든 개발자의 실무를 위한 필수 기본기 클래스 자료집, 8월 14, 2025에 액세스, [https://yansfil.github.io/awesome-class-materials/10.basic-knowledge/4.%EC%B6%94%EA%B0%80/OSI%207%EA%B3%84%EC%B8%B5%EA%B3%BC%20TCP_IP%204%EA%B3%84%EC%B8%B5%20%EB%AA%A8%EB%8D%B8.html](https://yansfil.github.io/awesome-class-materials/10.basic-knowledge/4.추가/OSI 7계층과 TCP_IP 4계층 모델.html)
81. Understanding HTTP Protocol & OSI Model | by Nisal Pubudu | Geek Culture - Medium, 8월 14, 2025에 액세스, https://medium.com/geekculture/understanding-http-protocol-osi-model-ba57cd5bda14
82. [CS 지식4.] OSI 7계층 & TCP/IP 4계층이란? - Somaz의 IT 공부 일지 - 티스토리, 8월 14, 2025에 액세스, https://somaz.tistory.com/176
83. OSI 7 Layer 쉽게 이해하기 - 네트워크 엔지니어 환영의 기술블로그, 8월 14, 2025에 액세스, https://aws-hyoh.tistory.com/50
84. 3. 사물인터넷 네트워크 - 저장소 - 티스토리, 8월 14, 2025에 액세스, https://123okk2.tistory.com/281
85. Connecting all the things in the Internet of Things - IBM Developer, 8월 14, 2025에 액세스, https://developer.ibm.com/articles/iot-lp101-connectivity-network-protocols/
86. OSI 7 Layer - 지식덤프, 8월 14, 2025에 액세스, http://jidum.com/jidums/view.do?jidumId=413
87. KS X ISO/IEC 7498-2(2022 확인) 개방형 시스템 간 상호 접속－기본 참조 모델－제2부：보안 구조 - 한국표준정보망, 8월 14, 2025에 액세스, https://www.kssn.net/search/stddetail.do?itemNo=K001010142286
88. ISO/ IEC 7498-1: Information Technology - Open Systems Interconnection - Basic reference model - Agrawal Personal, 8월 14, 2025에 액세스, http://magrawal.myweb.usf.edu/dcom/Ch1_ISO7498-1994-OSIModel.pdf
89. ISOIEC 7498-4 - iTeh Standards, 8월 14, 2025에 액세스, https://cdn.standards.iteh.ai/samples/14258/356879966ac041b7bddc5b090a8467d9/ISO-IEC-7498-4-1989.pdf
90. [Network] OSI 7계층이란? - 경험의 연장선, 8월 14, 2025에 액세스, https://livenow14.tistory.com/54
91. Role of OSI layers when we open a url in browser? - Super User, 8월 14, 2025에 액세스, https://superuser.com/questions/801788/role-of-osi-layers-when-we-open-a-url-in-browser
92. OSI Model Vs TCP/IP Model . - Cisco Learning Network, 8월 14, 2025에 액세스, https://learningnetwork.cisco.com/s/question/0D53i00000KsxUmCAJ/osi-model-vs-tcpip-model-


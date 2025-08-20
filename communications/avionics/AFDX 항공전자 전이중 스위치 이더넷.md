# 항공전자 전이중 스위치 이더넷(AFDX)

## 1. 항공전자 분야에서 결정론적 이더넷의 태동

항공전자 시스템의 역사는 기능적 요구사항의 증가와 기술적 진보가 맞물려 끊임없이 발전해 온 과정이다. 21세기에 들어서면서 항공기 시스템의 복잡성과 상호 연결성이 기하급수적으로 증가함에 따라, 기존의 항공전자 데이터버스 기술은 근본적인 한계에 직면하게 되었다. 이러한 기술적 난관을 극복하고 차세대 항공기의 요구사항을 충족시키기 위해 등장한 것이 바로 항공전자 전이중 스위치 이더넷(Avionics Full-Duplex Switched Ethernet, AFDX)이다. 본 보고서는 AFDX의 탄생 배경과 핵심 원리, 상세 기술 아키텍처, 그리고 레거시 및 최신 프로토콜과의 비교 분석을 통해 그 기술적 의의를 심도 있게 고찰한다. 나아가 현대 항공기에서의 구현 사례, 복잡한 검증 및 인증 절차, 그리고 미래 항공전자 네트워크의 진화 방향을 종합적으로 조망함으로써 AFDX에 대한 총체적이고 분석적인 이해를 제공하고자 한다.

### 1.1 연합형 아키텍처에서 통합형 아키텍처로의 패러다임 전환

AFDX의 등장을 이해하기 위해서는 먼저 항공전자 아키텍처의 근본적인 변화를 파악해야 한다. 20세기 항공기 시스템은 '하나의 박스, 하나의 기능'으로 대표되는 연합형(Federated) 아키텍처를 기반으로 설계되었다.1 이 구조에서는 관성항법장치, 자동조종장치, 비행관리컴퓨터 등 각각의 기능이 독립적인 하드웨어 장치, 즉 라인 교체 단위(Line Replaceable Unit, LRU)에 의해 수행되었다. 이러한 연합형 아키텍처는 기능의 독립성을 보장하고 고장 분석을 용이하게 하는 장점이 있었으나, 시스템이 복잡해짐에 따라 심각한 문제점을 드러냈다.

이 시기에 주로 사용된 데이터버스는 ARINC 429와 MIL-STD-1553이었다. ARINC 429는 12.5-100 kbit/s의 낮은 전송 속도를 가진 단방향 버스로, 각 연결마다 별도의 물리적 배선이 필요했다. 이로 인해 항공기 내 시스템 간 상호 연결이 복잡해질수록 케이블의 양과 무게가 엄청나게 증가하여 항공기 설계에 큰 부담으로 작용했다.2 MIL-STD-1553은 1 Mbit/s의 속도를 제공하는 양방향, 이중화 버스로 군용 항공기에서 널리 사용되었으나, 이 역시 버스 토폴로지의 한계와 최대 31개의 터미널만 연결할 수 있다는 제약으로 인해 데이터 집약적인 차세대 항공기에는 부적합했다.5

이러한 레거시 버스들의 근본적인 문제는 확장성, 대역폭, 그리고 유연성의 부족이었다.2 항공전자 기능이 점차 소프트웨어 중심으로 변화하면서, 연합형 아키텍처의 한계는 더욱 명확해졌다. 이에 대한 해결책으로 등장한 것이 바로 통합 모듈형 항공전자(Integrated Modular Avionics, IMA) 아키텍처이다.1 IMA는 여러 항공전자 기능을 소수의 고성능 공유 컴퓨터 플랫폼에서 소프트웨어적으로 구현하는 혁신적인 개념이다. IMA는 하드웨어와 소프트웨어 사이에 추상화 계층을 도입하여, 서로 다른 중요도를 가진 기능들이 동일한 하드웨어 자원을 공유할 수 있게 함으로써 항공기의 크기, 무게, 전력(Size, Weight, and Power, SWaP)을 획기적으로 줄일 수 있었다.2

그러나 IMA 개념의 실현을 위해서는 결정적인 전제 조건이 필요했다. 바로 분산된 기능과 공유 자원을 안정적으로 연결할 수 있는 고대역폭의 결정론적(deterministic) 네트워크였다. 특히 에어버스 A380과 같은 대형 항공기 개발 과정에서 기존의 데이터버스로는 배선 문제가 물리적으로, 그리고 경제적으로 감당할 수 없는 수준에 이르렀다.3 AFDX는 바로 이 IMA가 요구하는 네트워크 조건을 충족시키기 위해 에어버스에 의해 개발되었으며, 이는 단순한 속도 향상을 넘어선 항공전자 아키텍처의 혁명적 변화에 대한 필연적인 응답이었다. 이 과정에서 상용 기성품(Commercial Off-the-Shelf, COTS) 기술인 이더넷을 기반으로 채택한 것은 비용 효율성과 확장성을 확보하는 데 결정적이었지만, 동시에 비결정론적인 상용 기술을 안전 필수(safety-critical) 환경에 적용하기 위한 새로운 차원의 검증 및 인증이라는 과제를 낳았다. 이는 후술할 DO-178C, DO-254와 같은 엄격한 표준의 필요성을 예고하는 것이었다.

### 1.2 AFDX의 기본 원칙

AFDX는 레거시 버스의 한계를 극복하고 IMA의 요구사항을 충족시키기 위해 여섯 가지 핵심 원칙을 기반으로 설계되었다. 이 원칙들은 AFDX가 어떻게 표준 이더넷 기술을 항공전자 환경에 적합한 결정론적 네트워크로 변모시켰는지를 보여준다.

- **전이중(Full-Duplex):** 송신과 수신을 위한 별도의 물리적 경로를 사용하여 데이터 충돌 가능성을 원천적으로 제거한다. 이는 MIL-STD-1553과 같은 공유 버스 방식에서 발생하는 충돌 및 재전송 문제를 해결하고 예측 가능한 성능을 보장하기 위한 기본 전제 조건이다.1
- **이중화(Redundancy):** 네트워크 A와 B로 명명된 두 개의 독립적인 네트워크를 통해 모든 링크, 포트, 스위치를 완벽하게 이중화한다. 이는 단일 고장 지점을 제거하여 비행 안전에 필수적인 높은 수준의 내고장성을 제공한다.1
- **결정성(Determinism):** 데이터 전송, 지연(latency), 그리고 지터(jitter)가 사전에 정의된 범위 내에서 보장되는 것을 의미한다. 이는 안전 필수 실시간 시스템의 가장 중요한 요구사항으로, 가상 링크(Virtual Link), 대역폭 할당 갭(Bandwidth Allocation Gap) 등의 고유한 메커니즘을 통해 달성된다.1
- **고속 성능(High-Speed Performance):** 초기 표준은 10/100 Mbit/s의 속도를 지원하여, 기존 버스 대비 100배에서 1000배에 달하는 대역폭을 제공했다. 이는 IMA 아키텍처에서 요구하는 데이터 집약적인 애플리케이션을 지원하는 데 필수적이었다.5
- **스위치 기반 네트워크(Switched Network):** 중앙에 지능형 스위치를 배치하는 스타(star) 또는 계층형 스타(cascaded-star) 토폴로지를 사용한다. 이는 공유 버스를 대체하여 트래픽을 격리하고, 고장 전파를 방지하며, IMA가 요구하는 다대다(many-to-many) 통신을 가능하게 한다.5
- **프로파일 네트워크(Profiled Network):** 전체 네트워크는 설계 단계에서 사전에 완벽하게 구성되고 '프로파일링'된다. 모든 데이터 경로, 대역폭 할당, 지연 예산 등이 정적으로 정의되고 고정된다. 이는 동적 라우팅에서 발생하는 비결정성을 제거하고 네트워크가 설계 및 분석된 대로 정확하게 동작하도록 보장한다.1

## 2. AFDX의 기술 아키텍처 (ARINC 664 Part 7)

AFDX는 상용 이더넷 기술을 기반으로 하지만, 항공전자 시스템의 엄격한 실시간성과 신뢰성 요구사항을 만족시키기 위해 독자적인 메커니즘을 추가한 결정론적 네트워크이다. ARINC 664 Part 7 표준으로 정의된 AFDX의 기술 아키텍처는 기존 프로토콜 스택을 어떻게 변형하고 제약하여 결정성을 확보하는지에 대한 깊은 통찰을 제공한다.10

### 2.1 프로토콜 스택 및 OSI 모델 매핑

AFDX는 상호운용성을 보장하고 성숙한 COTS 기술을 활용하기 위해 표준 네트워킹 프로토콜을 기반으로 계층적 구조를 가진다. 이는 OSI 7계층 모델에 다음과 같이 매핑될 수 있다.

- **계층 1 (물리 계층):** IEEE 802.3 표준을 따르며, 주로 10/100 Mbit/s 속도의 구리 연선(twisted-pair) 또는 광섬유 케이블을 사용한다.10 특히 보잉 787 기종은 무게 절감과 전자기 간섭(EMI) 내성 강화를 위해 광섬유 기반 AFDX를 채택한 것으로 알려져 있다.10
- **계층 2 (데이터 링크 계층):** 표준 IEEE 802.3 MAC 프레임 형식을 사용하지만, 결정적인 변형이 가해진다. 기존의 48비트 목적지 MAC 주소 필드가 16비트의 가상 링크 ID(Virtual Link ID, VL ID)를 전송하는 데 사용된다. 이 VL ID는 AFDX 라우팅의 핵심 요소이다.3
- **계층 3 (네트워크 계층):** 표준 인터넷 프로토콜(IP)을 사용하여 종단 시스템(End System) 간의 논리적 주소 지정을 수행하며, 네트워크 진단 및 유지보수를 위해 인터넷 제어 메시지 프로토콜(ICMP)을 지원한다.10
- **계층 4 (전송 계층):** 주로 사용자 데이터그램 프로토콜(UDP)을 사용한다. UDP는 연결 설정이나 재전송 메커니즘이 없어 오버헤드가 적기 때문에 실시간 데이터 전송에 적합하다. 전송 제어 프로토콜(TCP)은 선택적으로 사용할 수 있으나, 핸드셰이킹과 재전송으로 인한 지연 때문에 비행 필수 데이터 전송에는 거의 사용되지 않는다.13
- **계층 5-7 (응용 계층):** 표준은 샘플링(Sampling), 큐잉(Queuing), 서비스 접근점(SAP), 데이터 로딩을 위한 TFTP(Trivial File Transfer Protocol), 그리고 네트워크 관리를 위한 SNMP(Simple Network Management Protocol)와 같은 필수적인 서비스들을 정의한다. 이는 시스템 통합과 유지보수 효율성을 크게 향상시킨다.16

### 2.2 가상 링크(VL) 개념: 결정성의 심장

AFDX 네트워크의 가장 핵심적인 개념은 가상 링크(Virtual Link, VL)이다. VL은 AFDX가 어떻게 비결정적인 이더넷을 결정론적으로 만드는지를 설명하는 열쇠이다.

- **VL의 정의:** VL은 하나의 송신 종단 시스템(ES)에서 하나 또는 그 이상의 수신 ES로 향하는 단방향 논리적 경로이다.5 이는 마치 ARINC 429의 전용 물리적 배선을 가상화하여 다대다 통신 환경에서 구현한 것과 같다.3
- **VL ID 기반 라우팅:** AFDX 스위치는 IP 주소나 MAC 주소로 프레임을 전달하지 않는다. 대신, 사전에 스위치에 로드된 정적 구성 테이블을 참조하여 프레임 헤더에 포함된 16비트 VL ID를 기반으로 정해진 출력 포트로만 프레임을 전달한다. 이 정적 라우팅 방식은 데이터 경로가 고정되어 있어 네트워크의 예측 가능성을 보장하는 결정성의 핵심 요소이다.10
- **대역폭 및 지연 보장:** 각 VL에는 시스템 통합 엔지니어에 의해 최대 대역폭과 종단 간 최대 지연 시간이 할당된다. 이를 통해 비행 제어와 같은 중요 데이터 스트림이 다른 트래픽에 의해 자원을 뺏기지 않도록 보장한다.1
- **확장성:** 단일 스위치는 최소 4,096개의 VL을 처리할 수 있으며, 여러 스위치를 계층적으로 연결하면 사실상 무한에 가까운 VL을 구성할 수 있어 매우 복잡한 항공기 시스템에도 적용 가능하다.1

### 2.3 트래픽 셰이핑 및 폴리싱: 규칙의 강제

AFDX는 송신단에서의 '셰이핑(shaping)'과 스위치에서의 '폴리싱(policing)'이라는 두 가지 메커니즘을 통해 네트워크 트래픽을 엄격하게 통제한다.

- **송신단에서의 트래픽 셰이핑:** 송신 ES는 각 VL에 대해 할당된 대역폭을 초과하지 않도록 스스로 트래픽을 조절할 책임이 있다. 이 조절은 **대역폭 할당 갭(Bandwidth Allocation Gap, BAG)**을 통해 이루어진다.
  - BAG는 동일한 VL에서 전송되는 연속된 두 프레임의 시작 시간 사이의 최소 간격을 의미한다. 이 값은 시스템 설계 단계에서 1, 2, 4,..., 128 ms와 같은 고정된 값으로 정의된다.3 송신 ES는 이 간격을 철저히 준수함으로써 트래픽을 예측 가능한 흐름으로 만든다.
- **스위치에서의 트래픽 폴리싱:** AFDX 스위치는 네트워크의 '경찰관' 역할을 수행한다. 스위치는 수신되는 모든 프레임에 대해 해당 VL에 정의된 BAG 규약을 준수했는지 검사한다.
  - 만약 프레임이 BAG 값보다 일찍 도착하면, 스위치는 해당 프레임을 즉시 폐기한다. 이 기능은 특정 ES가 고장 나거나 오작동하여 네트워크에 과도한 트래픽을 유발하는 '바블링 이디엇(babbling idiot)' 현상을 방지하고, 다른 중요 트래픽을 보호하는 핵심적인 역할을 한다.1 이는 AFDX 스위치를 일반 COTS 이더넷 스위치와 구별하는 가장 큰 특징 중 하나이다.
  - 이러한 트래픽 관리 개념은 비동기 전송 방식(ATM) 네트워크의 '토큰 버킷(token bucket)' 원리에서 차용하여 이더넷 환경에 맞게 변형한 것이다.10

### 2.4 이중화 및 무결성 관리: 신뢰성 확보

AFDX는 물리적, 논리적 차원에서 다층적인 메커니즘을 통해 최고의 신뢰성을 보장한다.

- **이중 네트워크 아키텍처:** 앞서 언급했듯이, 전체 네트워크는 물리적으로 완전히 분리된 네트워크 A와 B로 이중화된다. 송신 ES는 모든 프레임의 복사본을 만들어 두 네트워크를 통해 동시에 전송한다.1
- **무결성 검사:** 이중화된 데이터를 관리하는 책임은 수신 ES에 있다.
  - **순서 번호(Sequence Number, SN) 검사:** 각 AFDX 프레임에는 0부터 255까지 순환하는 8비트의 순서 번호가 포함된다. 수신 ES는 이 SN을 사용하여 프레임의 손실이나 순서 뒤바뀜을 탐지한다.1
  - **이중화 관리:** 수신 ES는 '선착순 유효 프레임 수용(First Valid Wins)' 정책을 따른다. 네트워크 A 또는 B 중 어느 한쪽에서 기대했던 SN을 가진 프레임을 먼저 수신하면, 그 프레임을 상위 애플리케이션으로 전달하고 타이머를 시작한다. `SkewMax`라는 파라미터로 정의된 시간 내에 다른 네트워크에서 동일한 VL과 SN을 가진 프레임이 도착하면, 이는 중복 프레임으로 간주되어 폐기된다.1 만약 양쪽 네트워크에서 모두 프레임이 손실되면, SN의 공백을 통해 시스템은 패킷 손실을 인지할 수 있다.

AFDX의 기술적 본질은 새로운 프로토콜의 발명이 아니라, 기존 COTS 프로토콜(이더넷, IP, UDP)을 '프로파일링'하고 '제약'하는 데 있다. 이는 표준 이더넷의 비결정적 요소들(예: CSMA/CD, 동적 라우팅, 플러딩)을 체계적으로 제거하고, VL, BAG, 정적 라우팅 테이블과 같은 엄격하고 사전에 정의된 규칙을 종단 시스템과 스위치 양단에서 강제함으로써 결정성을 달성한다. 이러한 접근 방식은 결과적으로 네트워크의 안전성과 결정성이 전적으로 모든 ES와 스위치에 로드되는 구성 파일의 정확성에 의존하게 만든다. 따라서 구성 파일의 설계, 관리, 검증 과정 자체가 안전 필수 활동으로 격상되며, 이는 DO-254/DO-178C와 같은 표준에서 요구하는 엄격한 문서화 및 프로세스 관리의 필요성과 직접적으로 연결된다. 구성 파일의 작은 오류 하나가 하드웨어 고장이나 소프트웨어 버그만큼이나 치명적인 결과를 초래할 수 있기 때문이다.

## 3. 항공전자 네트워크 프로토콜 비교 분석

AFDX의 기술적 위상과 장단점을 명확히 파악하기 위해서는 기존의 레거시 데이터버스는 물론, 새롭게 부상하는 차세대 결정론적 이더넷 표준들과의 다각적인 비교가 필수적이다. 이 분석을 통해 항공전자 네트워크 기술의 발전 궤적과 미래 방향성을 조망할 수 있다.

### 3.1 AFDX 대 레거시 데이터버스 (ARINC 429 & MIL-STD-1553)

AFDX는 ARINC 429 및 MIL-STD-1553과 비교했을 때, 단순히 속도가 빠른 것을 넘어 아키텍처 차원에서 근본적인 진화를 이루었다. 아래 표는 세 프로토콜의 핵심적인 차이점을 요약한 것이다.

| 기능                                                   | ARINC 429                                    | MIL-STD-1553                                                 | AFDX (ARINC 664 P7)                                        |
| ------------------------------------------------------ | -------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------------- |
| **토폴로지**                                           | 단방향, 점대다중점(Point-to-Multipoint) 버스 | 양방향, 다중점(Multi-drop) 버스                              | 스위치 기반 스타(Switched Star) 또는 계층형 스타           |
| **대역폭**                                             | 12.5-100 kbit/s                              | 1 Mbit/s                                                     | 10/100 Mbit/s (최대 1 Gbit/s 이상 지원 가능)               |
| **결정성 메커니즘**                                    | 단순성, 전용 물리적 링크                     | 명령/응답(Command/Response) 프로토콜, 버스 컨트롤러(BC)의 중앙 제어 | 가상 링크(VL), 대역폭 할당 갭(BAG), 스위치의 트래픽 폴리싱 |
| **이중화**                                             | 시스템별 구현 (표준 아님)                    | 이중(Dual) 버스                                              | 완전 이중화 네트워크 (네트워크 A/B)                        |
| **데이터 처리**                                        | 단방향 브로드캐스트                          | BC의 폴링에 의한 제어된 통신                                 | 스위치를 통한 VL 기반의 다중점 전송                        |
| **주요 사용처**                                        | 민간 항공기 (레거시 시스템)                  | 군용 항공기 (미션 크리티컬 시스템)                           | 최신 민간 및 군용 항공기 (IMA 백본 네트워크)               |
| *표 1: 레거시 및 최신 항공전자 데이터버스 비교 분석* 3 |                                              |                                                              |                                                            |

분석 결과, 대역폭 측면에서 AFDX는 레거시 버스 대비 100배에서 1000배에 달하는 압도적인 성능 향상을 제공한다.5 이는 고해상도 디스플레이, 플라이 바이 와이어(Fly-by-wire) 시스템, 그리고 방대한 양의 시스템 상태 모니터링 데이터를 처리하는 데 필수적이다. 토폴로지 측면에서는 AFDX의 스위치 기반 스타 구조가 핵심적인 장점을 제공한다. ARINC 429와 MIL-STD-1553의 공유 버스 구조는 단일 지점의 결함이 전체 버스에 영향을 미칠 수 있는 반면, AFDX는 스위치를 통해 각 연결을 격리함으로써 고장 전파를 효과적으로 차단한다.6

결정성 확보 방식 또한 근본적인 차이를 보인다. ARINC 429는 단순한 물리적 구조 자체로 결정성을 확보하고, MIL-STD-1553은 버스 컨트롤러의 엄격한 통제를 통해 결정성을 유지한다. 반면, AFDX는 VL, BAG, 트래픽 폴리싱이라는 복합적인 시스템을 통해 소프트웨어적으로 결정성을 구현한다.4 이는 시스템 설계 및 분석의 복잡성을 증가시키는 요인이 되기도 하지만, 동시에 훨씬 높은 수준의 유연성과 확장성을 제공한다.

### 3.2 AFDX 대 차세대 결정론적 이더넷 표준 (TTEthernet & TSN)

항공전자 네트워크 기술은 AFDX에서 멈추지 않고 계속 진화하고 있다. 현재 가장 주목받는 기술은 TTEthernet과 TSN으로, 이들은 AFDX의 개념을 계승하면서도 더 높은 수준의 결정성과 융합성을 목표로 한다.

| 속성                                          | AFDX (ARINC 664 P7)                                          | TTEthernet (SAE AS6802)                                      | TSN (IEEE 802.1)                                    |
| --------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------------------------------- |
| **주요 표준**                                 | ARINC 664 Part 7                                             | SAE AS6802                                                   | IEEE 802.1 표준 모음                                |
| **핵심 스케줄링 방식**                        | 이벤트 트리거(Event-Triggered) 방식 (BAG 기반 트래픽 셰이핑) | 시간 트리거(Time-Triggered) 방식 (전역 동기화된 스케줄 기반) | 혼합 방식 (TAS, CBS 등 다양한 스케줄러/셰이퍼 제공) |
| **클럭 동기화**                               | 불필요 (비동기 방식)                                         | 필수 (전역 클럭 동기화)                                      | 필수 (IEEE 802.1AS)                                 |
| **트래픽 종류**                               | 두 가지 우선순위 (High/Low)                                  | 세 가지 종류 (TT, RC, BE)                                    | 다중 우선순위 및 다양한 트래픽 클래스 지원          |
| **이중화 방식**                               | 네트워크 A/B 이중화, 수신단에서 처리                         | 네트워크 이중화 지원                                         | 프레임 복제 및 제거 (FRER, IEEE 802.1CB)            |
| **핵심 장점**                                 | 검증된 표준, 유연한 비동기식 통합                            | 극도로 낮은 지터, 최고의 결정성                              | 개방형 표준, 높은 유연성, 비용 효율성, 산업 간 융합 |
| *표 2: 결정론적 이더넷 프로토콜 기술 비교* 23 |                                                              |                                                              |                                                     |

**TTEthernet (Time-Triggered Ethernet)**은 결정성 측면에서 가장 진보한 기술로 평가받는다. TTEthernet의 핵심은 모든 네트워크 장치가 마이크로초 단위로 동기화된 전역 시간을 공유하고, 이를 기반으로 모든 중요 메시지(TT 트래픽)가 사전에 정의된 시간 슬롯에만 전송되도록 하는 엄격한 시간 트리거 스케줄에 있다.23 이 방식은 데이터 충돌 가능성을 완전히 배제하여 극도로 낮은 지터를 보장한다. TTEthernet은 TT 트래픽 외에도 AFDX와 유사한 속도 제한 트래픽(RC)과 일반 이더넷 트래픽(BE)을 동일한 네트워크에서 함께 지원하여 혼합 중요도(mixed-critically) 시스템에 최적화되어 있다. 이러한 특성 때문에 NASA의 오리온 우주선과 같이 절대적인 예측 가능성이 요구되는 시스템에 채택되었다.23 반면, AFDX는 프레임이 준비되고 BAG 조건만 만족하면 전송되는 이벤트 트리거 방식으로, TTEthernet만큼 엄격한 시간 동기화가 필요 없어 시스템 통합이 상대적으로 유연하다는 장점이 있다.29

**TSN (Time-Sensitive Networking)**은 단일 프로토콜이 아닌, 표준 이더넷에 결정론적 기능을 추가하는 IEEE 802.1 표준들의 '도구 상자(toolbox)'이다.23 여기에는 TTEthernet의 스케줄과 유사한 시간 인식 셰이퍼(TAS, IEEE 802.1Qbv), 스트림의 대역폭을 제어하는 크레딧 기반 셰이퍼(CBS, IEEE 802.1Qav), 그리고 AFDX의 이중화와 유사한 프레임 복제 및 제거(FRER, IEEE 802.1CB) 등의 다양한 도구들이 포함된다.31

TSN은 AFDX의 차세대 기술로 가장 유력하게 거론된다. 그 이유는 항공우주 분야뿐만 아니라 자동차, 산업 자동화 등 더 넓은 산업 분야에서 채택되고 있어 COTS 부품 생태계가 훨씬 크고, 이로 인해 비용 절감과 기술 혁신 속도에서 유리하기 때문이다.26 현재 IEEE P802.1DP 항공우주 프로파일 제정 작업을 통해 TSN의 여러 기능들을 조합하여 어떻게 항공전자 시스템의 요구사항을 충족시킬 것인지에 대한 표준화가 진행 중이다. 이는 사실상 AFDX의 핵심 개념들(VL -->> Stream, BAG -->> Shaper 등)을 개방형 IEEE 표준 프레임워크 내에서 재현하는 과정이라고 볼 수 있다.28

결정론적 네트워킹 기술의 발전은 명확한 궤적을 그리고 있다. ARINC 429나 MIL-STD-1553과 같은 *프로토콜 자체에 내재된 결정성*에서 시작하여, AFDX와 같은 *구성을 통한 결정성*으로 발전했고, 이제 TTEthernet과 TSN과 같은 *스케줄러 기반의 결정성*으로 나아가고 있다. 이러한 진화는 단일 물리 인프라에서 다양한 중요도를 가진 트래픽을 효율적으로 융합 처리할 수 있는, 더욱 강력하고 유연한 네트워크를 향한 움직임을 반영한다. 그러나 이는 동시에 새로운 인증의 과제를 제시한다. AFDX의 인증 절차는 이제 성숙 단계에 접어들었지만, 수많은 표준이 상호작용하는 TSN '도구 상자'를 DAL A 수준으로 인증하기 위해서는 트래픽 클래스 간의 복잡한 간섭 현상을 분석하고 검증할 수 있는 새로운 기법과 도구가 필요하며, 이는 현재 항공우주 산업계가 집중적으로 연구하고 있는 분야이다.28

## 4. 구현, 검증 및 인증

AFDX는 이론적 우수성을 넘어 실제 최신 항공기에 성공적으로 적용되어 그 성능과 신뢰성을 입증했다. 그러나 AFDX 네트워크를 항공기에 탑재하기까지의 과정은 설계부터 검증, 그리고 최종 인증에 이르는 길고 복잡한 여정을 거쳐야 한다. 이 섹션에서는 AFDX의 실제 구현 사례와 함께, 그 성능과 안전성을 보장하기 위한 엄격한 검증 및 인증 절차를 심층적으로 다룬다.

### 4.1 현대 항공기에서의 AFDX 적용

AFDX는 오늘날 상용, 군용, 그리고 비즈니스 항공기를 아우르는 다양한 첨단 항공 플랫폼의 핵심 데이터 백본으로 자리 잡았다.

| 항공기                                        | 제조사 | AFDX 최초 비행 | AFDX 적용 범위                                      | 물리 계층   |
| --------------------------------------------- | ------ | -------------- | --------------------------------------------------- | ----------- |
| **Airbus A380**                               | Airbus | 2005년         | 전체 시스템 백본 (비행 제어, 조종석, 전력, 연료 등) | 구리        |
| **Boeing 787**                                | Boeing | 2009년         | 전체 시스템 백본                                    | 광섬유      |
| **Airbus A350**                               | Airbus | 2013년         | 전체 시스템 백본                                    | 구리/광섬유 |
| **Airbus A400M**                              | Airbus | 2009년         | 주요 시스템 백본                                    | 구리        |
| **COMAC C919**                                | COMAC  | 2017년         | 통합 모듈형 항공전자 백본                           | 이더넷 기반 |
| **Sukhoi Superjet 100**                       | Sukhoi | 2008년         | 주요 시스템 백본                                    | 구리        |
| *표 3: 주요 항공기 플랫폼의 AFDX 적용 현황* 9 |        |                |                                                     |             |

위 표에서 보듯이, AFDX는 단순한 주변기기용 버스가 아니라 항공기의 '중추 신경계' 역할을 담당한다. 비행 제어, 조종석 디스플레이, 비행 관리 시스템과 같은 비행 안전에 직결되는 최상위 중요도(DAL A) 시스템부터 전력, 연료, 착륙 장치, 공조 시스템에 이르기까지 거의 모든 핵심 시스템 간의 데이터 교환이 AFDX 네트워크를 통해 이루어진다.13 이는 AFDX 네트워크의 아주 작은 결함이라도 항공기 전체의 안전에 치명적인 영향을 미칠 수 있음을 의미하며, 따라서 극도로 엄격한 검증 및 인증 절차가 요구되는 이유이기도 하다.

### 4.2 검증 및 확인(V&V) 워크플로우

AFDX 네트워크는 그 복잡성과 중요성 때문에 단순히 하드웨어를 제작한 후 테스트하는 방식으로는 안전성을 보장할 수 없다. 대신, 설계 초기 단계부터 전체 개발 수명 주기에 걸쳐 시뮬레이션과 정형 분석을 포함하는 체계적인 검증 및 확인(Verification & Validation, V&V) 절차를 따라야 한다.

- **방법론:** V&V 프로세스는 일반적으로 모델 기반 시스템 엔지니어링(Model-Based System Engineering, MBSE) 패러다임을 따른다.37 이는 실제 하드웨어를 제작하기 전에 네트워크의 동작을 가상 환경에서 철저히 검증하는 것을 목표로 한다.
  - **네트워크 칼큘러스(Network Calculus):** 모든 VL에 대한 최악의 경우 종단 간 지연(worst-case end-to-end delay) 및 지터의 상한값을 수학적으로 계산하는 정형 기법이다. 이를 통해 네트워크가 설계된 타이밍 요구사항을 만족한다는 것을 수학적으로 증명할 수 있다.29
  - **시뮬레이션(Simulation):** 이벤트 기반 시뮬레이터를 사용하여 전체 네트워크를 모델링하고, 다양한 트래픽 부하 및 고장 시나리오 하에서 성능을 분석한다. 이는 네트워크 칼큘러스가 제공하는 정적인 최악의 경우 분석을 보완하여, 시스템의 동적인 동작에 대한 상세한 피드백을 제공한다.37
- **도구:** 성숙한 AFDX V&V 도구 생태계가 구축되어 있다.
  - **시뮬레이션 및 분석 플랫폼:** OMNeT++, Simulink, Vector CANoe.AFDX와 같은 도구들이 네트워크 모델링 및 성능 분석에 널리 사용된다.38
  - **테스트 및 측정 하드웨어/소프트웨어:** AIM, Abaco Systems, Excalibur, TechSAT과 같은 회사들은 실험실 환경에서 실제 AFDX 네트워크를 모니터링, 분석, 시뮬레이션하고 오류를 주입할 수 있는 특수 하드웨어 카드(PCIe, PXIe 등)와 소프트웨어(예: PBA.pro, BusTools)를 제공한다.13
- **사례 연구: 이중화 관리 검증:** UPPAAL과 같은 도구를 사용한 시간 오토마타(Timed Automata)나 TLA(Temporal Logic of Actions)와 같은 정형 기법은 AFDX 이중화 관리 알고리즘을 모델링하고 검증하는 데 사용되었다. 이러한 연구를 통해 '네트워크 바블링'과 같은 특정 고장 상황에서 불필요한 시스템 리셋이 유발될 수 있는 잠재적 취약점이 발견되었으며, 이를 해결하기 위해 수신단에 우선순위 큐를 추가하는 등의 설계 개선안이 제안되기도 했다.18

### 4.3 인증의 관문: DO-254, DO-178C, 그리고 A(M)C 20-193 (CAST-32A)

AFDX 기반 시스템을 인증받는 것은 하드웨어, 소프트웨어, 그리고 시스템 수준의 여러 표준을 동시에 만족시켜야 하는 거대하고 복잡한 과제이다. 특히 비행 안전에 직결되는 시스템에는 가장 높은 설계 보증 수준인 DAL A(Design Assurance Level A - Catastrophic)가 요구된다.37

- **DO-254 (항공기 전자 하드웨어):** 이 표준은 AFDX 스위치 및 종단 시스템 인터페이스 카드에 포함된 ASIC, FPGA와 같은 복잡한 전자 하드웨어의 개발 과정을 규제한다. 요구사항 정의부터 설계, 구현, 검증에 이르는 모든 과정에 대한 엄격한 추적성을 요구한다.56 COTS IP 코어의 사용이나 부품의 단순/복잡 분류는 주요한 인증 과제 중 하나이다.61
- **DO-178C (항공기 소프트웨어):** 이 표준은 종단 시스템과 스위치 내부에서 실행되는 소프트웨어에 적용된다. 계획, 개발, 검증(코드 커버리지 분석 포함), 형상 관리, 품질 보증 등 수많은 목표를 만족시켜야 하며, DAL A 소프트웨어의 경우 코드 한 줄당 개발 비용이 100달러를 넘을 수도 있다.38
- **A(M)C 20-193 (CAST-32A 계승, 멀티코어 프로세서):** 현대 IMA 플랫폼은 멀티코어 프로세서를 사용하는데, 이는 새로운 인증 문제를 야기했다. CAST-32A와 이를 계승한 A(M)C 20-193은 서로 다른 코어에서 실행되는 소프트웨어 파티션 간의 *간섭(interference)* 문제를 해결하기 위한 지침을 제공한다. 메모리 버스, 캐시, I/O와 같은 공유 자원에 대한 경합은 비결정적인 타이밍 동작을 유발하여 시스템 전체의 안전성을 위협할 수 있다. 따라서 개발자는 모든 잠재적 간섭 채널을 식별하고, 파티셔닝, 캐시 관리, 대역폭 할당 등의 완화 기법이 효과적임을 상세한 분석을 통해 입증해야 한다. 이는 현재 항공전자 인증에서 가장 중요하고 어려운 과제 중 하나로 꼽힌다.68
- **EASA AMC 20-152A:** 이 문서는 FAA와 EASA 간의 DO-254 지침을 조화시키고, 회로 기판 어셈블리(CBA)나 복잡한 COTS 장치 등 현대적인 하드웨어에 대한 적용을 명확히 하여 AFDX 스위치 및 종단 시스템 인증에 직접적인 영향을 미친다.62

AFDX 시스템의 V&V 및 인증 과정은 기존의 부품 단위 검증에서 시스템 단위 검증으로의 패러다임 전환을 명확히 보여준다. 레거시 버스에서는 단일 LRU를 독립적으로 검증하는 것이 어느 정도 가능했지만, AFDX 환경에서는 전체 네트워크의 구성과 동작을 고려하지 않고는 단 하나의 종단 시스템도 제대로 검증할 수 없다. 즉, 네트워크 자체가 하나의 거대한 시스템인 것이다. 이러한 시스템 수준의 검증은 DO-254, DO-178C, CAST-32A와 같은 여러 표준이 상호작용하는 복합적인 문제를 해결해야 함을 의미한다. 이처럼 AFDX 네트워크를 인증하는 데 따르는 막대한 비용과 복잡성은 역설적으로 업계가 TSN과 같은 차세대 표준으로 나아가게 하는 강력한 경제적 동인이 된다. TSN 인증 역시 복잡하겠지만, 자동차 및 산업 자동화라는 더 넓은 COTS 생태계에 기반을 두고 있어 인증된 도구, 재사용 가능한 IP, 그리고 숙련된 엔지니어 풀을 활용하여 장기적으로 비반복적 엔지니어링 및 인증 비용을 절감할 수 있을 것이라는 기대를 낳고 있다.

## 5. 미래 항공전자 디지털 백본의 전망

AFDX는 지난 20년간 항공전자 네트워크의 표준으로 군림해왔지만, 기술은 끊임없이 진화하고 있다. 현재 항공우주 산업은 AFDX의 성공을 발판 삼아 더욱 유연하고, 비용 효율적이며, 지능적인 차세대 디지털 백본으로의 전환을 모색하고 있다. 이 섹션에서는 TSN으로의 진화 경로, 네트워크 인텔리전스 및 보안 강화, 그리고 미래 시스템 개발자를 위한 전략적 제언을 통해 항공전자 네트워크의 미래를 조망한다.

### 5.1 시간 민감형 네트워킹(TSN)으로의 진화 경로

업계는 TSN을 AFDX의 논리적 진화 단계로 보고 있다. TSN은 개방형 IEEE 표준을 기반으로 비행 필수 트래픽부터 기내 엔터테인먼트, 유지보수 데이터에 이르기까지 모든 종류의 트래픽을 단일 융합 네트워크에서 처리할 수 있는 가능성을 제시한다. 이는 SWaP 감소, 광범위한 COTS 생태계를 통한 비용 절감, 그리고 더 큰 아키텍처 유연성을 약속한다.26

이러한 전환의 핵심적인 가교 역할은 IEEE 802.1DP "항공우주 프로파일(Aerospace Profile)"이 담당한다. 이 프로파일은 TSN이라는 '도구 상자'를 어떻게 조합해야 엄격한 항공전자 요구사항을 만족시킬 수 있는지를 공식화하는 작업으로, 사실상 AFDX의 결정론적 개념들을 표준화되고 개방된 프레임워크로 옮겨오는 과정이다.28

물론 이 전환 과정이 간단하지는 않다. AFDX의 개념을 TSN 표준에 매핑하고(예: VL -->> Stream, BAG -->> Shaper, 이중화 -->> FRER), 기존 AFDX 시스템과의 공존을 위해 복잡한 게이트웨이 설계가 필요할 수 있다.28 그러나 장기적인 목표는 TSN 기반의 통일된 백본을 구축하는 것이며, Collins Aerospace나 Honeywell과 같은 주요 항공전자 공급업체들은 이러한 기술 진화에 적극적으로 참여하며 미래의 커넥티드 항공기 생태계를 주도하고 있다.81

### 5.2 네트워크 인텔리전스 및 보안 강화

미래의 항공전자 네트워크는 단순히 데이터를 전송하는 것을 넘어, 스스로를 진단하고 보호하는 지능을 갖추게 될 것이다. 이 과정에서 인공지능(AI)과 머신러닝(ML)이 핵심적인 역할을 할 것으로 기대된다.

- **AI/ML의 적용:**
  - **예측 정비(Predictive Maintenance):** 네트워크 트래픽, 오류 로그, 시스템 성능 데이터를 ML 알고리즘으로 분석하여 부품 고장을 사전에 예측할 수 있다. 이는 예기치 않은 결함을 방지하고 정비 효율성을 높여 항공기 가용성을 극대화하는 데 기여할 것이다.85
  - **네트워크 트래픽 분석 및 침입 탐지:** ML을 통해 정상적인 네트워크 동작의 기준선을 설정하고, 이를 벗어나는 비정상적인 패턴을 탐지하여 사이버 공격이나 시스템 오작동을 식별할 수 있다. 이는 정적인 규칙 기반 시스템보다 훨씬 동적이고 지능적인 방어 체계를 구축하게 해준다.85
- **결정론적 네트워크의 보안 태세:**
  - AFDX는 그 설계 특성상 많은 일반적인 IT 공격에 대해 강력한 내재적 보안을 제공한다. 정적으로 고정된 구성, 동적 라우팅의 부재, 스위치에서의 엄격한 트래픽 폴리싱 등은 스푸핑(spoofing), 중간자 공격(man-in-the-middle), 그리고 서비스 거부(DoS) 공격을 매우 어렵게 만든다. 인가되지 않은 장치가 네트워크에 단순히 연결하여 트래픽을 교란하는 것이 사실상 불가능하다.2
  - 이러한 특성은 기밀성(confidentiality)보다 가용성(availability)과 무결성(integrity)을 우선시하는 운영 기술(Operational Technology, OT) 또는 산업 제어 시스템(ICS)의 보안 원칙과 완벽하게 일치한다. NIST의 ICS 보안 가이드라인은 물리 세계와 상호작용하는 시스템의 고유한 보안 요구사항을 강조하며, AFDX는 이러한 시스템의 대표적인 예이다.87 따라서 AFDX의 주요 보안 과제는 전통적인 외부 해킹 방어보다는 구성 정보의 무결성을 보장하고, 내부자 위협이나 공급망의 취약점으로부터 시스템을 보호하는 데 있다.

### 5.3 시스템 통합 및 개발자를 위한 전략적 제언

이러한 기술적 변화의 흐름 속에서 시스템 통합자와 개발자는 다음과 같은 전략적 접근을 고려해야 한다.

1. **모델 기반 V&V의 적극적 수용:** 현재의 AFDX 프로젝트에서는 시뮬레이션과 정형 분석을 활용한 선행 V&V에 집중적으로 투자해야 한다. 실험실 단계에서 네트워크 설계 결함을 발견하는 비용은 시스템 통합 이후에 발견하는 것보다 수십, 수백 배 저렴하다.
2. **혼합 중요도 시대 대비:** 새로운 시스템을 설계할 때, 아키텍트는 결정론적 이더넷의 전체 스펙트럼을 고려해야 한다. AFDX가 검증된 표준이지만, 하드 실시간 제어 루프를 위해 TTEthernet을 평가하거나, 장기적인 호환성을 위해 TSN을 염두에 둔 아키텍처를 설계하는 것이 현명하다.
3. **멀티코어 간섭 분석 전문성 확보:** 멀티코어 프로세서를 사용하는 모든 시스템에서 A(M)C 20-193 준수는 필수적이다. 시스템 통합자는 간섭 분석 및 완화 기술에 대한 깊은 전문성을 자체적으로 확보하거나, 인증된 솔루션을 제공하는 RTOS 및 하드웨어 공급업체와 긴밀하게 협력해야 한다. 이는 항공전자 인증 분야에서 가장 중요한 새로운 도전 과제이다.
4. **설계 초기부터 보안 통합:** AFDX가 본질적으로 견고하더라도 보안을 나중의 문제로 미루어서는 안 된다. OT/ICS 보안 모범 사례에 따라 구성 관리의 무결성, 안전한 데이터 로딩 절차, 네트워크 구성 요소의 물리적 보안에 중점을 둔 '설계 기반 보안(Secure by Design)' 철학을 채택해야 한다.

AFDX에서 TSN으로의 진화는 단순한 기술 업그레이드를 넘어, 항공우주 산업이 공급망의 위험을 줄이고 더 넓은 산업 기술 표준과 연계하여 비용을 절감하려는 전략적 비즈니스 결정이다. 이는 항공 산업만의 고유한 솔루션에서 벗어나, 자동차 및 산업 자동화 시장의 거대한 규모와 혁신을 활용하려는 움직임이다. 이러한 융합은 전통적인 항공전자 공급업체와 기술 기업 간의 경계를 허물게 될 것이다. 핵심 네트워킹 기술이 표준화되고 상품화됨에 따라, 가치 제안의 중심은 규정을 준수하는 하드웨어를 만드는 것에서, 그 하드웨어 위에서 동작하는 정교한 소프트웨어, 서비스, 그리고 보안 계층을 제공하는 것으로 이동할 것이다. 인증된 멀티코어 RTOS, AI/ML 기반 분석, OT 시스템을 위한 사이버 보안과 같은 분야에서 탁월한 역량을 갖춘 기업들이 미래 항공전자 시장의 승자가 될 것이다. 네트워크 자체는 기반이 되고, 그 위에서 실행되는 '지능'이 핵심적인 차별화 요소가 되는 시대가 오고 있다.

#### **참고 자료**

1. Architecting of Avionics Full Duplex Ethernet (AFDX ... - IJECT, accessed July 13, 2025, https://www.iject.org/vol4/spl4/c0140.pdf
2. The Evolution of Avionics Networks From ARINC 429 to ... - CiteSeerX, accessed July 13, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=829a3d29ca6cadb114c41962792bc3e1d802ef5f
3. AFDX®: A Time-Deterministic application of ARINC 664 part 7 - Logic Fruit Technologies, accessed July 13, 2025, https://www.logic-fruit.com/blog/arinc/afdx-a-time-deterministic-application-of-arinc-664-part-7/
4. A Brief comparison between mil std 1553 and major commercial data bus systems, accessed July 13, 2025, https://www.mil-1553.com/comparison-mil-std-1553-major-data-bus-systems
5. Mastering AFDX in Avionics Systems - Number Analytics, accessed July 13, 2025, https://www.numberanalytics.com/blog/mastering-afdx-in-avionics-systems
6. Decoding ARINC 429 vs MIL-STD-1553: Key Differences - KIMDU ..., accessed July 13, 2025, https://kimdu.com/decoding-arinc-429-vs-mil-std-1553-key-differences/
7. Why are there so many avionics communications specifications? - Excalibur Systems, accessed July 13, 2025, https://www.mil-1553.com/why-many-avionics-communications-specifications
8. Avionics databus users demand more reliability and flexibility - Military Embedded Systems, accessed July 13, 2025, https://militaryembedded.com/avionics/databus/avionics-databus-users-demand-more-reliability-and-flexibility
9. The Airbus A380 - an AFDX-based flight test computer concept - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/4147050_The_Airbus_A380_-_an_AFDX-based_flight_test_computer_concept
10. Avionics Full-Duplex Switched Ethernet - Wikipedia, accessed July 13, 2025, https://en.wikipedia.org/wiki/Avionics_Full-Duplex_Switched_Ethernet
11. AFDX_Overview | PDF | Computer Network | Network Switch - Scribd, accessed July 13, 2025, https://www.scribd.com/document/865783089/AFDX-Overview
12. Design and Development of AFDX Transmitter Schedular - IJMETMR, accessed July 13, 2025, http://www.ijmetmr.com/oljune2015/SahithiPriya-BSwapnaRani-134.pdf
13. AFDX®/ARINC664P7 Tutorial - What is AFDX - AIM Online, accessed July 13, 2025, https://www.aim-online.com/products-overview/tutorials/afdx-arinc664p7-tutorial/
14. Arinc 664 p7 - AFDX Tutorial & Products - Excalibur Systems, accessed July 13, 2025, https://www.mil-1553.com/arinc-664p7-afdx
15. Testing aircraft AFDX systems - Aim Online, accessed July 13, 2025, https://www.aim-online.com/wp-content/uploads/2019/06/aim-article-afdx-aircraft-testing-12-05-01-u.pdf
16. ARINC 664 P7 - AIRCRAFT DATA NETWORK PART 7 AVIONICS ..., accessed July 13, 2025, https://standards.globalspec.com/std/1283307/arinc-664-p7
17. Testing Ethernet Based Avionics in Aircraft - Mobility Engineering ..., accessed July 13, 2025, https://www.mobilityengineeringtech.com/component/content/article/14453-testing-ethernet-based-avionics-in-aircraft
18. Formal Modeling and Analysis of the AFDX Frame Management Design ∗ - University of Pennsylvania, accessed July 13, 2025, https://repository.upenn.edu/bitstreams/aadea0d6-3ec5-40b4-8686-368e451992d8/download
19. Avionics Databus Tutorials - Astronics, accessed July 13, 2025, https://www.astronics.com/avionics-databus-tutorials
20. (PDF) Avionics data buses: An overview - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/3277898_Avionics_data_buses_An_overview
21. Quantitative Comparison of the Error-Containment Capabilities of a Bus and a Star Topology in CAN Networks - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/224084310_Quantitative_Comparison_of_the_Error-Containment_Capabilities_of_a_Bus_and_a_Star_Topology_in_CAN_Networks
22. A Practical Approach to Commercial Aircraft Data Buses, accessed July 13, 2025, https://www.ddc-web.com/resources/FileManager/dbi/Whitepapers/Commercial_Avionics_Document.pdf
23. Impact Analysis of Flow Shaping in Ethernet-AVB/TSN and AFDX from Network Calculus and Simulation Perspective, accessed July 13, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC5470926/
24. Comparison of Time Sensitive Networking (TSN) and TTEthernet | PDF - Scribd, accessed July 13, 2025, https://www.scribd.com/document/691191734/Comparison-of-Time-Sensitive-Networking-TSN-and-TTEthernet
25. On TTEthernet for Integrated Fault-Tolerant Spacecraft Networks, accessed July 13, 2025, https://ntrs.nasa.gov/api/citations/20150014489/downloads/20150014489.pdf
26. TSN: an Ethernet network, but better for critical embedded systems - IRT Saint Exupéry, accessed July 13, 2025, https://www.irt-saintexupery.com/tsn-an-ethernet-network-but-better-for-critical-embedded-systems/
27. Comparison of ieee AVB and AFDX - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/261208790_Comparison_of_ieee_AVB_and_AFDX
28. 1 Introduction - arXiv, accessed July 13, 2025, https://arxiv.org/html/2312.13969v1
29. Avionics Full Duplex Ethernet and the Time Sensitive ... - IEEE 802, accessed July 13, 2025, https://www.ieee802.org/1/files/public/docs2015/TSN-Schneele-AFDX-0515-v01.pdf
30. Time Sensitive Networking (TSN) for Aerospace & Defense - United Electronic Industries, accessed July 13, 2025, https://www.ueidaq.com/time-sensitive-networking-tsn-for-aerospace-defense
31. Time Sensitive Networking for Aerospace - Design And Reuse, accessed July 13, 2025, https://www.design-reuse.com/articles/55905/time-sensitive-networking-for-aerospace.html
32. A Comprehensive Survey of Wireless Time-Sensitive Networking (TSN): Architecture, Technologies, Applications, and Open Issues - arXiv, accessed July 13, 2025, https://arxiv.org/html/2312.01204v3
33. TSN FOR AEROSPACE - IEEE 802, accessed July 13, 2025, https://www.ieee802.org/1/files/public/docs2024/admin-tsn-aerospace-flyer-1124.pdf
34. Deployment of a Testbed for Validation of TSN Networks in Avionics - MDPI, accessed July 13, 2025, https://www.mdpi.com/2226-4310/12/3/186
35. Avionics Systems Integration & Test - TEST DRAFT - Avionics Interface Technologies - A Teradyne Company, accessed July 13, 2025, https://aviftech.com/systems-integration-test-2/
36. Comac C919 - Wikipedia, accessed July 13, 2025, https://en.wikipedia.org/wiki/Comac_C919
37. An Event-Driven Link-Level Simulator for the Validation of AFDX and Ethernet Avionics Networks - MDPI, accessed July 13, 2025, https://www.mdpi.com/2226-4310/11/4/247
38. (PDF) Verification and Validation Framework for AFDX Avionics Networks - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/363010246_Verification_and_Validation_Framework_for_AFDX_Avionics_Networks
39. Verification and Validation Framework for AFDX Avionics Networks - SciSpace, accessed July 13, 2025, https://scispace.com/pdf/verification-and-validation-framework-for-afdx-avionics-1yrd66tv.pdf
40. The Design and Implementation of the AFDX Network Simulation System - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/224193529_The_Design_and_Implementation_of_the_AFDX_Network_Simulation_System
41. Design and Analysis of AFDX Network Based High-Speed Avionics System of Civil Aircraft, accessed July 13, 2025, https://www.researchgate.net/publication/271972538_Design_and_Analysis_of_AFDX_Network_Based_High-Speed_Avionics_System_of_Civil_Aircraft
42. Temporal Analysis of AFDX Networks Aerospace Engineering - Scholar, accessed July 13, 2025, https://scholar.tecnico.ulisboa.pt/api/records/bNe0fzODMMM6Elraknl1Z9J6ArSqw3V8wYQL/file/dd7e5111b8006399128254dfd827f028bc2a557cdc9b2d6f52f5b84296251918.pdf
43. OMNeT++ Simulation Framework for Avionics Full-Duplex Switched Ethernet | Journal of Aerospace Information Systems - AIAA ARC, accessed July 13, 2025, https://arc.aiaa.org/doi/10.2514/1.I011351
44. IO781: ARINC-664 Part 7 / AFDX protocol support with Simulink - Speedgoat, accessed July 13, 2025, https://www.speedgoat.com/products/communication-protocols-arinc-afdx-io781
45. Research and Analysis on AFDX Network Configuration Based on OMNET++ Simulation, accessed July 13, 2025, https://drpress.org/ojs/index.php/ajst/article/view/31232
46. (PDF) Research and Analysis on AFDX Network Configuration Based on OMNET++ Simulation - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/392943807_Research_and_Analysis_on_AFDX_Network_Configuration_Based_on_OMNET_Simulation
47. Case Study – Modeling Avionics Full-Duplex Switched Ethernet - OMNEST, accessed July 13, 2025, https://omnest.com/casestudy-afdx
48. INVESTIGATING AND ANALYZING JITTER IN CONTROLLER AREA NETWORK - DiVA portal, accessed July 13, 2025, https://www.diva-portal.org/smash/get/diva2:1735813/FULLTEXT01.pdf
49. BusTools/AFDX-A BusTools Software Analyzer | Abaco Systems, accessed July 13, 2025, https://www.abaco.com/products/bt-afdx-bustools-software-analyzer
50. AFDX® / ARINC 664 Part 7 Switch for Flight Test - Data Device Corporation, accessed July 13, 2025, https://www.ddc-web.com/en/discontinued-1/afdx--arinc-664-part-7-switch-for-flight-test
51. AFDX® / ARINC 664 PMC Card - Data Device Corporation, accessed July 13, 2025, https://www.ddc-web.com/en/discontinued-1/dd-82101f
52. TechSAT Integrates TTTech's AFDX® and TTEthernet Network Switches for the Next Generation ADS2 Avionics Test Benches, accessed July 13, 2025, https://www.tttech.com/press/techsat-integrates-tttechs-afdx-and-ttethernet-network-switches-for-the-next-generation-ads2-avionics-test-benches
53. Formal Modeling and Analysis of AFDX Frame Management Design - CiteSeerX, accessed July 13, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=fdcc79dd3d481d2aadd58f33ef9c3a023486b065
54. Formal Modeling and Analysis of the AFDX Frame Management Design. - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/221250136_Formal_Modeling_and_Analysis_of_the_AFDX_Frame_Management_Design
55. Formal Specification and Analysis of AFDX Redundancy Management Algorithms - CiteSeerX, accessed July 13, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=28bbb9d30183ffb3bfec7f31aad129f6d1a55920
56. DO-254 Compliance Guide: Everything Avionics Hardware Teams Need to Know, accessed July 13, 2025, https://www.modernrequirements.com/blogs/do-254/
57. DO-254 Explained: Design Assurance Guidance for Airborne Electronic Hardware, accessed July 13, 2025, https://visuresolutions.com/aerospace-and-defense/do-254/
58. DO-254 Certification Kits for IP Cores - SafeCore Devices, accessed July 13, 2025, https://safecoredevices.com/do-254-ip-cores/do-254-certification-kits/
59. DO-254 IP Cores - SafeCore Devices, accessed July 13, 2025, https://safecoredevices.com/do-254-ip-cores/
60. Best DO-254 Compliance Tools, Checklists & Templates - Visure ..., accessed July 13, 2025, https://visuresolutions.com/aerospace-and-defense/do-254-checklists-and-tools/
61. DO-254 Intro, Compliance: Free Tools/ Papers / Resources - AFuzion, accessed July 13, 2025, https://afuzion.com/do-254-introduction/
62. DO-254 Update: AC/AMC 20-152A (FREE) - Airworthiness Certification Services, accessed July 13, 2025, https://airworthinesscert.com/do-254-do-178c-training-academy/do-254-update-ac-amc-20-152a-free/
63. What is the Impact of AMC 20-152A on Your DO-254 Project?, accessed July 13, 2025, https://patmos-eng.com/impact-of-amc20-152a-on-do254-project/
64. DO-178C Guidance: Introduction to RTCA DO-178 certification | Rapita Systems, accessed July 13, 2025, https://www.rapitasystems.com/do178
65. February 2017/March 2017 - DO-178C: Software for NextGen Avionics, UAVs and More, accessed July 13, 2025, https://interactive.aviationtoday.com/avionicsmagazine/february-2017-march-2017/do-178c-software-for-nextgen-avionics-uavs-and-more/
66. Software Deliverables For DO-178C DAL-B | PDF | Verification And Validation - Scribd, accessed July 13, 2025, https://www.scribd.com/document/597082960/Software-Deliverables-for-DO-178C-DAL-B
67. Deos RTOS Software | Safety Critical Multicore RTOS | DAL A, DO-178C, FACE - DDC-I, accessed July 13, 2025, https://www.ddci.com/solutions/products/deos/
68. CAST-32A Guidance for Multicore Processorss - Green Hills Software, accessed July 13, 2025, https://www.ghs.com/products/safety_critical/integrity_178_standards_cast-32A.html
69. CAST-32A: Development of avionics software for single-core processors, accessed July 13, 2025, https://www.cast32a.com/
70. Certification Authorities Software Team (CAST) : Position Paper CAST-32A | PDF | Multi Core Processor | Computer Engineering - Scribd, accessed July 13, 2025, https://www.scribd.com/document/357177001/cast-32A
71. AMC20-193 - LDRA, accessed July 13, 2025, https://ldra.com/amc20-193/
72. CAST-32A, AC 20-193, & Multi-Core Processing for Avionics - AFuzion, accessed July 13, 2025, https://afuzion.com/cast-32a-ac-20-193-multi-core-processing-for-avionics/
73. What is CAST-32A? - Rapita Systems, accessed July 13, 2025, https://www.rapitasystems.com/cast-32a
74. CAST-32A: Multi-Core ready to become Airborne | SYSGO, accessed July 13, 2025, https://www.sysgo.com/blog/article/cast-32a-multi-core-ready-to-become-airborne
75. Easy Access Rules for Acceptable Means of Compliance for Airworthiness of Products, Parts and Appliances (AMC-20) - initial issue & amendment 1 - EASA, accessed July 13, 2025, https://www.easa.europa.eu/en/document-library/easy-access-rules/online-publications/easy-access-rules-acceptable-means?page=25
76. Software & Airborne Electronic Hardware | Federal Aviation Administration, accessed July 13, 2025, https://www.faa.gov/aircraft/air_cert/design_approvals/air_software/planned
77. Avionics Hardware Development: Introduction to DO-254 and AMC 20-152A | PTC, accessed July 13, 2025, https://www.ptc.com/en/resources/application-lifecycle-management/webcast/avionics-hardware-development-do-254-amc-20-152a
78. AMC20-152A - Patmos Engineering Services, accessed July 13, 2025, https://patmos-eng.com/wp-content/uploads/2020/08/AMC20-152A.pdf
79. Specification and analysis of an extended AFDX with TSN/BLS shapers for mixed-criticality avionics applications - Theses.fr, accessed July 13, 2025, https://theses.fr/2018ESAE0010/document
80. Power of TSN and RTOS integration for aerospace - Industrial Ethernet Book, accessed July 13, 2025, https://iebmedia.com/technology/tsn/power-of-tsn-and-rtos-integration-for-aerospace/
81. Future of Flight - Collins Aerospace, accessed July 13, 2025, https://www.collinsaerospace.com/what-we-do/capabilities/future-of-flight
82. ACARS over IP - Collins Aerospace, accessed July 13, 2025, https://www.collinsaerospace.com/what-we-do/industries/commercial-aviation/connected-cockpit/acars-over-ip
83. NextGen Avionics Roadmap Version 2.0 - DTIC, accessed July 13, 2025, https://apps.dtic.mil/sti/tr/pdf/ADA561244.pdf
84. C&PS Process Roadmap - Honeywell MyAerospace, accessed July 13, 2025, https://aerospace2.honeywell.com/wps/portal/aero/help/roadmap
85. Cybersecurity in Modern Avionics - Number Analytics, accessed July 13, 2025, https://www.numberanalytics.com/blog/cybersecurity-modern-avionics-protecting-aviation
86. NetDiffusion: Network Data Augmentation Through Protocol-Constrained Traffic Generation | Request PDF - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/378376215_NetDiffusion_Network_Data_Augmentation_Through_Protocol-Constrained_Traffic_Generation
87. The NIST Guide to Industrial control systems security - Packt+, accessed July 13, 2025, https://www.packtpub.com/en-BE/product/industrial-cybersecurity-9781788395151/chapter/the-ics-cybersecurity-program-development-process-12/section/the-nist-guide-to-industrial-control-systems-security-ch12lvl1sec62
88. Final Report - IG-17-011 - NASA OIG, accessed July 13, 2025, https://oig.nasa.gov/docs/IG-17-011.pdf
89. SCADA System Cyber Security – A Comparison of ... - SCADAhacker, accessed July 13, 2025, [https://scadahacker.com/library/Documents/Standards/IEEE%20-%20Comparison%20of%20SCADA%20Security%20Standards.pdf](https://scadahacker.com/library/Documents/Standards/IEEE - Comparison of SCADA Security Standards.pdf)
90. Industrial cyber security: how to guarantee security in IIot environments., accessed July 13, 2025, https://magazine.relatech.com/en/industrial-cyber-security-how-to-guarantee-security-in-iiot-environments


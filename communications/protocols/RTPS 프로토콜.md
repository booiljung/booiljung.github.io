# RTPS 프로토콜

## 서론

### 0.1 보고서의 목적과 대상 독자

본 보고서는 시스템 아키텍트, 엔지니어 및 연구원을 대상으로 실시간 발행-구독(Real-Time Publish-Subscribe, RTPS) 프로토콜에 대한 포괄적이고 심층적인 기술 분석을 제공하는 것을 목적으로 한다. 이 문서는 RTPS의 기본 원리부터 아키텍처, 메시지 구조, 보안 취약점, 그리고 로보틱스, 자율주행차, 항공우주 등 주요 산업 분야에서의 실제 적용 사례에 이르기까지 모든 측면을 다룬다. 대상 독자는 분산 시스템, 미들웨어, 네트워크 프로토콜에 대한 기술적 배경지식을 보유하고 있으며, 단순한 개요를 넘어 시스템 설계 및 구현에 필요한 깊이 있는 통찰력을 얻고자 하는 전문가 또는 학술 연구자이다.

### 0.2 혼동하기 쉬운 프로토콜 명확화

RTPS는 이름이 유사한 다른 프로토콜과 자주 혼동되므로, 논의를 시작하기에 앞서 이들의 차이점을 명확히 하는 것이 중요하다.

- **RTPS (Real-Time Publish-Subscribe Protocol):** 본 보고서의 핵심 주제이다. 데이터 분산을 위한 P2P(Peer-to-Peer) 프로토콜로, 객체 관리 그룹(Object Management Group, OMG)에 의해 데이터 분산 서비스(Data Distribution Service, DDS)의 상호운용성을 위한 유선 프로토콜(wire protocol)로 표준화되었다.1
- **RTP (Real-time Transport Protocol):** 국제 인터넷 표준화 기구(Internet Engineering Task Force, IETF)의 표준(RFC 3550)으로, 주로 UDP 상에서 오디오 및 비디오와 같은 스트리밍 미디어의 종단 간(end-to-end) 전송을 위해 설계되었다. 미디어 스트림의 타이밍과 동기화에 중점을 둔다.4
- **RTSP (Real-Time Streaming Protocol):** 또 다른 IETF 표준(RFC 2326)으로, 스트리밍 미디어 서버를 제어(예: 재생, 일시정지)하기 위한 네트워크 제어 프로토콜이다. 스트리밍 세션을 관리하지만, 실제 미디어 데이터를 전송하지는 않으며, 이 역할은 주로 RTP가 담당한다.4

### 요약

RTPS는 데이터 분산 서비스(DDS)의 근간을 이루는 유선 프로토콜로서, 로보틱스, 자율주행 시스템, 항공우주 및 국방과 같은 미션 크리티컬 시스템에서 고성능, 고신뢰성, 상호운용 가능한 통신을 실현하는 데 핵심적인 역할을 수행한다. 이 프로토콜은 중앙 서버 없이 노드 간 직접 통신하는 탈중앙화된 발행-구독 모델을 기반으로 하며, 이를 통해 탁월한 내고장성, 확장성, 그리고 동적인 네트워크 환경에서의 '플러그 앤 플레이' 연결성을 제공한다. 본 보고서는 RTPS의 아키텍처, 서비스 품질(QoS) 정책을 통한 정교한 제어 메커니즘, 보안적 측면의 강점과 약점, 그리고 다양한 실제 산업 분야에서의 적용 사례를 심도 있게 분석하여 이 프로토콜에 대한 완전한 이해를 제공할 것이다.

------

## 1.  RTPS의 기본 원칙과 DDS 생태계에서의 역할

### 1.1  표준의 탄생: 독점적 혁신에서 개방형 상호운용성으로

RTPS 프로토콜의 역사는 분산 시스템 통신의 진화 과정을 잘 보여준다. 이 프로토콜은 본래 Real-Time Innovations (RTI)사에 의해 고성능 발행-구독 통신을 위한 독자적인 유선 프로토콜로 개발되었다.7 그 기술적 우수성을 인정받아, 산업 자동화 기업 컨소시엄인 IDA(Interface for Distributed Automation Group)에서 표준 이더넷 및 IP 네트워크를 통한 산업 통신의 기반 메커니즘으로 채택되면서 초기 성공을 거두었다.9

RTPS의 진정한 전환점은 객체 관리 그룹(OMG)에 의해 표준으로 채택되면서 이루어졌다. 공식 명칭은 "실시간 발행-구독 프로토콜 DDS 상호운용성 유선 프로토콜 명세(The Real-time Publish-Subscribe Protocol (RTPS) DDS Interoperability Wire Protocol Specification)"이며, 흔히 DDSI-RTPS로 축약된다.2 2008년 4월에 버전 2.0이 OMG 표준으로 처음 채택된 이후, 여러 차례의 개정(버전 2.1, 2.2, 2.3, 2.5 등)을 거치며 지속적으로 발전해왔다.8

이러한 표준화 과정은 단순한 기술적 승인을 넘어, 시장의 강력한 요구에 부응하는 필연적인 결과였다. DDS가 항공우주, 국방과 같은 미션 크리티컬 시스템에 점차 널리 채택되면서, 특정 벤더의 독점 기술에 종속될 경우 발생하는 위험, 즉 '벤더 종속성(vendor lock-in)'이 심각한 문제로 대두되었다. 수십 년에 이르는 시스템 수명 주기를 고려할 때, 장기적인 유지보수성, 경쟁을 통한 비용 절감, 그리고 부품의 안정적인 수급을 보장하기 위해서는 개방형 표준에 기반한 상호운용성이 필수적이었다.12 따라서 RTPS의 표준화는 RTI, eProsima, Twin Oaks 등 여러 벤더가 개발한 DDS 구현체들이 서로 원활하게 통신할 수 있는 기반을 마련함으로써, 전체 산업 생태계의 안정성과 지속 가능성을 확보하기 위한 핵심적인 전략이었다.2

### 1.2  DDS와 RTPS의 공생적 아키텍처: API 대 프로토콜

DDS와 RTPS의 관계를 이해하는 것은 분산 실시간 시스템을 설계하는 데 있어 매우 중요하다. 이 둘의 관계는 명확한 계층적 구조로 설명할 수 있다. 애플리케이션 개발자는 DDS API와 상호작용하며, DDS 구현체는 네트워크 상에서 데이터를 실제로 교환하기 위해 RTPS 프로토콜을 사용한다.17

- **DDS (The "What"):** DDS는 '무엇을 할 것인가'를 정의한다. 이것은 데이터 중심의 연결성 프레임워크, 애플리케이션 프로그래밍 인터페이스(API), 그리고 미들웨어가 보장해야 할 동작(behavior)을 명시하는 **표준 명세**이다. 개발자는 `DomainParticipant`, `Publisher`, `Subscriber`, `Topic`, `DataWriter`, `DataReader`와 같은 고수준의 추상화된 객체와 풍부한 서비스 품질(QoS) 정책을 사용하여 애플리케이션 로직을 구현한다.14
- **RTPS (The "How"):** RTPS는 '어떻게 할 것인가'를 정의한다. 이것은 네트워크 상에서 데이터를 교환하기 위한 구체적인 메시지 형식, 발견(discovery) 메커니즘, 그리고 상태 머신과 같은 동작 규칙을 정의하는 **유선 프로토콜**이다. 즉, DDS라는 추상적인 개념을 상호운용 가능한 방식으로 구현하기 위한 구체적인 방법론이다.1

DDS의 핵심 개념들은 RTPS의 구성 요소와 거의 일대일로 매핑된다. 예를 들어, DDS의 `DomainParticipant`는 RTPS의 `Participant`에, `DataWriter`는 `Writer`에, `DataReader`는 `Reader`에 직접적으로 대응된다.1 이러한 자연스러운 매핑 구조 덕분에 RTPS는 DDS를 구현하기 위한 가장 이상적인 프로토콜로 자리 잡았다.

대부분의 DDS 구현체가 내부적으로 RTPS를 사용하지만, 고급 사용자는 완전한 DDS API 계층 없이 RTPS 프로토콜 계층을 직접 사용하여 통신을 구현할 수도 있다.1 이 경우 DDS가 제공하는 풍부한 데이터 관리 기능(예: 정교한 QoS 정책 관리)의 일부를 포기해야 하지만, 프로토콜의 내부 동작에 대해 더 세밀한 제어가 가능하다는 장점이 있다.18

**표 1: DDS 엔티티와 RTPS 엔티티 매핑**

| DDS 엔티티 (API 계층) | RTPS 엔티티 (프로토콜 계층) | 설명                                                         |
| --------------------- | --------------------------- | ------------------------------------------------------------ |
| `DomainParticipant`   | `Participant`               | 도메인 내에서 애플리케이션의 참여를 나타내는 컨테이너. 엔드포인트의 팩토리 역할을 한다. |
| `Topic`               | `Topic`                     | 데이터 스트림을 고유하게 식별하는 이름, 데이터 타입, QoS의 조합. |
| `Publisher`           | (엔티티 없음)               | `DataWriter`를 그룹화하고 관리하는 논리적 객체. 프로토콜 수준에서는 `Participant`에 암시적으로 포함된다. |
| `DataWriter`          | `Writer`                    | 데이터 샘플을 네트워크에 발행하는 통신 엔드포인트.           |
| `Subscriber`          | (엔티티 없음)               | `DataReader`를 그룹화하고 관리하는 논리적 객체. 프로토콜 수준에서는 `Participant`에 암시적으로 포함된다. |
| `DataReader`          | `Reader`                    | 네트워크로부터 데이터 샘플을 수신하는 통신 엔드포인트.       |

### 1.3  핵심 설계 원칙: RTPS의 "기둥"

RTPS 프로토콜의 아키텍처와 기능은 몇 가지 핵심적인 설계 목표에 의해 결정된다. 이러한 목표들은 RTPS가 왜 특정 응용 분야에 적합한지를 설명해주며, 여러 자료에서 일관되게 강조된다.1

- **성능 및 서비스 품질 (Performance and QoS):** 실시간 애플리케이션을 위해 설계되었으며, 신뢰성과 적시성 간의 균형을 사용자가 직접 조절할 수 있는 풍부한 QoS 정책을 제공한다.3
- **내고장성 (Fault-Tolerance):** 탈중앙화된 P2P 아키텍처를 기반으로 하여 단일 장애점(Single Point of Failure)이 존재하지 않도록 설계되었다. 이는 특정 노드의 장애가 전체 시스템의 동작에 미치는 영향을 최소화한다.3
- **플러그 앤 플레이 연결성 (Plug-and-Play Connectivity):** 자동 발견(automatic discovery) 메커니즘을 통해 애플리케이션이 별도의 설정 없이 네트워크에 동적으로 참여하거나 이탈할 수 있다. 이는 시스템의 유연성과 관리 용이성을 크게 향상시킨다.1
- **확장성 (Scalability):** 잠재적으로 수천 개의 노드로 구성된 매우 큰 규모의 네트워크까지 확장될 수 있도록 설계되었다.1
- **확장 가능성 (Extensibility):** 기존 버전과의 하위 호환성을 유지하면서 새로운 서비스나 메시지 유형을 추가하여 프로토콜을 확장할 수 있다. 이는 수명이 긴 시스템의 지속적인 업그레이드와 유지보수에 필수적이다.1
- **모듈성 (Modularity):** 단순하거나 리소스가 제한된 장치도 프로토콜의 일부 기능만 구현하여 네트워크에 참여할 수 있도록 허용한다. 이는 다양한 종류의 장치가 혼재하는 이기종 시스템 구축을 용이하게 한다.1
- **타입 안전성 (Type Safety):** 데이터 타입이 일치하지 않는 등의 애플리케이션 프로그래밍 오류가 원격 노드의 동작을 손상시키는 것을 방지한다.3

이러한 설계 원칙들은 단순히 기능 목록의 나열이 아니다. 이들은 종합적으로 중앙 집중식 서버나 브로커 기반 아키텍처와는 근본적으로 다른 패러다임을 정의한다. RTPS는 노드들이 예고 없이 나타나거나 사라질 수 있고, 중앙의 통제 기관에 의존할 수 없는 동적이고 예측 불가능하며 때로는 적대적인 환경을 위해 근본적으로 설계되었다. 이러한 철학은 로보틱스, 자율주행차, 전장 시스템과 같이 예외적인 상황이 아닌 일상적인 조건에서 운영되는 시스템에서 RTPS가 탁월한 성능을 발휘하는 이유를 명확히 설명해준다.

------

## 2.  RTPS 프로토콜 아키텍처 심층 분석

### 2.1  플랫폼 독립 모델(PIM): 모듈식 청사진

RTPS 프로토콜은 플랫폼 독립 모델(Platform Independent Model, PIM)이라는 추상적인 명세로 정의된다. 이 모델은 프로토콜을 네 가지 주요 모듈로 구성하여 명확성과 확장성을 확보한다: 구조(Structure), 메시지(Messages), 동작(Behavior), 발견(Discovery).1 이러한 모듈식 접근법은 프로토콜의 핵심 설계 원칙 중 하나로, 각 부분의 역할을 명확히 하고 향후 기능 추가를 용이하게 한다.1

PIM은 이후 구체적인 전송 계층에 맞게 플랫폼 종속 모델(Platform Specific Model, PSM)로 매핑된다. 가장 보편적인 PSM은 UDP/IP를 위해 정의되었다. UDP는 비연결성, 멀티캐스트 지원, 예측 가능한 동작 특성 덕분에 실시간 시스템에 이상적인 전송 수단으로 간주된다.3 한편, UDP 트래픽이 차단되는 방화벽 환경을 통과해야 하는 시나리오를 위해 TCP PSM 또한 개발 중에 있다.3

### 2.2  구조 모듈: 통신 환경 정의

구조 모듈은 RTPS 통신에 참여하는 기본 요소들과 그들의 관계를 정의한다.

- **RTPS 도메인 (RTPS Domain):** `$domainId$`라는 정수 값으로 식별되는 격리된 통신 평면이다. 엔티티는 동일한 도메인에 속한 다른 엔티티와만 통신할 수 있으며, 이는 네트워크를 논리적으로 분할하는 역할을 한다.1
- **RTPS 참여자 (RTPS Participant):** DDS의 `DomainParticipant`에 해당하며, 하나의 애플리케이션이 특정 도메인에 참여하고 있음을 나타낸다. 참여자는 엔드포인트(Endpoint)를 생성하고 관리하는 팩토리이자 컨테이너 역할을 한다. 각 참여자는 전역 고유 식별자(Globally Unique Identifier, GUID)를 통해 시스템 전체에서 유일하게 식별된다.1
- **RTPS 엔드포인트 (RTPS Endpoints):** 데이터의 송신 및 수신 지점이다.
  - **`Writer`:** DDS의 `DataWriter`에 해당하며 데이터 변경 사항의 소스이다. 매칭된 `Reader`에게 RTPS 메시지를 전송한다.1
  - **`Reader`:** DDS의 `DataReader`에 해당하며 데이터 변경 사항의 소비자이다. 매칭된 `Writer`로부터 메시지를 수신한다.1
- **히스토리 캐시 (HistoryCache):** 애플리케이션과 네트워크 프로토콜 사이의 핵심적인 버퍼 역할을 한다. 애플리케이션이 데이터를 발행(`write`)하면, 해당 데이터는 `CacheChange`라는 객체로 `Writer`의 `HistoryCache`에 저장된다. 그 후 RTPS `Writer`는 이 변경 사항을 `Reader`들에게 전송한다. 메시지를 수신한 `Reader`는 해당 `CacheChange`를 자신의 `HistoryCache`에 추가하고 애플리케이션에 새로운 데이터가 도착했음을 알린다.1 이 메커니즘은 애플리케이션의 데이터 생성/소비 로직과 네트워크 전송 로직을 효과적으로 분리(decoupling)한다.

### 2.3  메시지 모듈: RTPS 패킷의 해부학

메시지 모듈은 `Writer`와 `Reader` 간에 교환되는 정보의 구체적인 형식을 정의한다.

- **일반적인 메시지 구조:** 하나의 RTPS 메시지는 단일 `헤더(Header)`와 그 뒤를 따르는 가변 개수의 `서브메시지(Submessages)`로 구성된다.1 이 구조는 매우 유연하여, 향후 프로토콜 버전을 확장하여 새로운 서브메시지를 추가하더라도 하위 호환성을 깨뜨리지 않는다.1
- **RTPS 헤더:** 다음 정보를 포함한다.
  - **프로토콜 식별자:** 'R', 'T', 'P', 'S' 네 개의 문자.
  - **프로토콜 버전:** 예를 들어, 2.x와 같은 버전 번호.
  - **벤더 ID:** DDS 구현체 벤더를 식별하는 고유 ID.
  - **`GuidPrefix`:** 메시지를 보내는 `Participant`의 GUID에서 고유한 접두사 부분.1
- **RTPS 서브메시지:** 각 서브메시지는 자체 헤더를 가지며, 여기에는 `submessageId`, 플래그, 길이를 포함한다. 이를 통해 수신자는 자신이 이해하지 못하는 서브메시지를 안전하게 건너뛸 수 있다.1

**표 2: 주요 RTPS 서브메시지와 기능**

| 서브메시지 이름 | ID (16진수) | 송신자   | 수신자   | 주요 기능                                                    |
| --------------- | ----------- | -------- | -------- | ------------------------------------------------------------ |
| `DATA`          | `0x15`      | `Writer` | `Reader` | 애플리케이션 데이터를 포함하는 페이로드를 전송한다.          |
| `HEARTBEAT`     | `0x07`      | `Writer` | `Reader` | `Writer`가 현재 보유한 데이터 시퀀스 번호 범위를 알려 신뢰성 있는 통신을 지원한다. |
| `ACKNACK`       | `0x06`      | `Reader` | `Writer` | 수신한 데이터(ACK)와 누락된 데이터(NACK) 정보를 `Writer`에게 알려 재전송을 요청한다. |
| `GAP`           | `0x08`      | `Writer` | `Reader` | 특정 시퀀스 번호 범위의 데이터가 더 이상 유효하지 않거나 사용할 수 없음을 알린다. |
| `INFO_TS`       | `0x09`      | `Writer` | `Reader` | 메시지 생성 시점의 타임스탬프 정보를 제공하여 시간 기반 필터링 등을 지원한다. |
| `DATA_FRAG`     | `0x16`      | `Writer` | `Reader` | 큰 데이터를 여러 조각으로 분할하여 전송할 때 사용된다.       |

- **주요 서브메시지 상세 분석:**

  - **`DATA`:** 애플리케이션 데이터를 전송하는 가장 중요한 서브메시지이다. 데이터 객체의 변경 사항에 대한 정보와 직렬화된 페이로드 자체를 포함한다.1 데이터의 값 변경뿐만 아니라, 데이터의 라이프사이클 변경(예: 폐기)을 알리는 데도 사용될 수 있다.

  - **`HEARTBEAT`:** 신뢰성 있는(reliable) `Writer`가 매칭된 `Reader`들에게 주기적으로 보내는 메시지이다. `Writer`의 `HistoryCache`에 현재 사용 가능한 시퀀스 번호(즉, `CacheChange`들)의 범위를 알려준다.1 이를 통해 

    `Reader`는 자신이 데이터를 놓쳤는지 여부를 감지할 수 있다.

  - **`ACKNACK`:** 신뢰성 있는 `Reader`가 `Writer`에게 보내는 응답 메시지이다. 수신된 데이터에 대한 긍정 확인(ACK)과 누락된 데이터에 대한 부정 확인(NACK) 역할을 동시에 수행한다. `Reader`는 비트맵(bitmap)을 사용하여 특정 시퀀스 번호 범위의 수신 상태를 효율적으로 전달하며, 이 메시지는 `Writer`의 재전송을 유발한다.1

- **CDR을 이용한 데이터 직렬화:** `DATA` 서브메시지 내의 페이로드는 OMG의 공통 데이터 표현(Common Data Representation, CDR) 표준을 사용하여 직렬화된다.7 CDR은 엔디안(endianness) 차이와 같은 문제를 해결하여 기계에 독립적인 방식으로 데이터를 표현함으로써, 이기종 시스템 간의 상호운용성을 보장한다.26 이 표준은 이후 확장 가능한 CDR(XCDR 및 XCDR2)을 포함하도록 발전하여 더욱 유연한 타입 진화를 지원한다.27

### 2.4  동작 모듈: 상호작용의 규칙

동작 모듈은 `Writer`와 `Reader` 간의 유효한 상호작용과 상태 변화를 정의하는 상태 머신을 규정한다.1

- **최선 노력 (Best-Effort) 통신:** 가장 단순한 동작 방식이다. `Writer`는 `DATA` 서브메시지를 UDP를 통해 전송하며, 전송을 보장하지 않는다. `Reader`는 수신하는 데이터만 처리한다. 최신 데이터가 과거의 개별 데이터보다 중요한 고주파수 센서 데이터 스트리밍에 적합하다.
- **신뢰성 있는 (Reliable) 통신 상태 머신:** UDP와 같은 비신뢰성 전송 계층 위에서 신뢰성을 구현하는 RTPS의 핵심 메커니즘이다.
  1. 신뢰성 있는 `Writer`는 `DATA` 서브메시지를 전송한다.
  2. `Writer`는 주기적으로 `HEARTBEAT` 메시지를 보내 자신이 보유한 데이터의 시퀀스 번호 범위를 알린다.
  3. 신뢰성 있는 `Reader`는 `HEARTBEAT`를 수신하고, `Writer`가 가진 시퀀스 번호와 자신의 `HistoryCache`에 저장된 `CacheChange`를 비교한다.
  4. 만약 `Reader`가 누락된 시퀀스 번호를 발견하면, 해당 번호를 명시하여 `ACKNACK` 서브메시지를 `Writer`에게 보낸다 (NACK).
  5. `ACKNACK`을 수신한 `Writer`는 요청받은 `DATA` 서브메시지를 재전송한다. 이때 재전송은 보통 해당 `Reader`에게만 유니캐스트로 이루어진다.
  6. `Reader`가 모든 데이터를 수신할 때까지 이 과정이 반복된다.1

이러한 자체 신뢰성 구현은 중요한 설계적 선택의 결과이다. 단순히 TCP를 사용하는 대신 UDP 위에 독자적인 신뢰성 프로토콜을 구현함으로써 RTPS는 전송 로직에 대한 완전한 제어권을 갖는다. 이는 TCP에서는 불가능한 멀티캐스트를 활용하여 효율적인 일대다(one-to-many) 데이터 배포를 가능하게 한다. 또한, 한 명의 느린 `Reader`가 과거 데이터를 수신하는 동안 다른 `Reader`들에게는 계속해서 새로운 데이터를 전송하는 등, 발행-구독 패러다임에 최적화된 유연한 신뢰성 패턴을 구현할 수 있다. 그 대가는 "신뢰성의 바퀴를 재발명"하는 복잡성이지만 30, 그로 인해 얻는 성능과 유연성은 미션 크리티컬 시스템에서 매우 중요하다.

### 2.5  발견 모듈: 플러그 앤 플레이의 실현

발견 모듈은 중앙 서버나 설정 파일 없이도 참여자들이 서로를 자동으로 찾고, 그들이 포함하는 토픽, 타입, QoS에 대한 정보를 교환할 수 있게 해준다.1 이는 '플러그 앤 플레이'라는 핵심 설계 목표를 실현하는 기반 기술이다.

- **메타트래픽 (Metatraffic):** 발견 목적으로 교환되는 데이터는 애플리케이션의 '사용자 트래픽(user traffic)'과 구분하여 '메타트래픽'이라고 부른다.1

- **단순 발견 프로토콜 (Simple Discovery Protocol, SDP):** 표준 발견 메커니즘으로, 두 단계로 나뉜다.

  1. **참여자 발견 프로토콜 (Participant Discovery Protocol, PDP):** `DomainParticipant`들이 서로를 발견하는 방법을 명시한다. 새로운 참여자가 도메인에 참여하면, 사전에 약속된 잘 알려진(well-known) 주소와 포트로 자신의 존재를 멀티캐스트한다. 다른 참여자들은 이에 응답하며, 참여자 정보의 P2P 교환이 이루어진다.1 이 단계는 흔히 SPDP(Simple PDP)라고 불린다.

  2. **엔드포인트 발견 프로토콜 (Endpoint Discovery Protocol, EDP):** PDP를 통해 두 참여자가 서로를 발견하면, EDP를 사용하여 각자의 로컬 `DataWriter`와 `DataReader`에 대한 상세 정보를 교환한다. 이 정보에는 각 엔드포인트의 토픽 이름, 데이터 타입, QoS 정책 등이 포함된다.1 미들웨어는 이 정보를 바탕으로 어떤 

     `Writer`와 `Reader`가 서로 매칭되어 통신을 시작해야 하는지 결정한다.

`HistoryCache`의 아키텍처는 `HEARTBEAT`/`ACKNACK` 메커니즘과 본질적으로 연결되어 있다. `HistoryCache`는 단순한 버퍼가 아니라, 시퀀스 번호가 부여된 데이터 변경 사항의 상태 저장소(stateful log)이다. 신뢰성 프로토콜은 본질적으로 `Writer`와 `Reader`라는 두 분산 시스템이 비신뢰성 채널을 통해 각자의 `HistoryCache` 상태를 동기화하는 메커니즘이다. `HEARTBEAT`는 `Writer`의 캐시 상태에 대한 선언이며, `ACKNACK`은 `Reader`의 상태 차이 보고서(state-diff report)로서, 목표 지향적인 업데이트를 유발한다. 이는 단순한 TCP 스타일의 패킷 기반 신뢰성 모델보다 훨씬 정교하고 데이터 중심적인 모델이다.

------

## 3.  서비스 품질(QoS)을 통한 거버넌스 및 제어

### 3.1  QoS 프레임워크: 데이터 분산의 맞춤화

서비스 품질(QoS) 정책은 개발자가 DDS/RTPS 미들웨어의 동작을 제어하는 핵심 메커니즘이다.14 QoS를 통해 성능, 신뢰성, 리소스 사용량 등 통신의 거의 모든 측면을 정밀하게 조정할 수 있다.

QoS 프레임워크의 중심에는 **요청-제공(Request-Offered, RxO) 모델**이 있다. `DataWriter`는 특정 수준의 서비스를 *제공(offer)*하고, `DataReader`는 특정 수준의 서비스를 *요청(request)*한다. 연결(매치)은 제공되는 QoS가 요청된 QoS와 호환될 때(일반적으로 제공되는 수준이 요청된 수준보다 높거나 같을 때)만 성립된다.32 이 모델은 통신 양단 간에 명시적이고 강제력 있는 "계약"을 형성한다.

QoS는 프로그래밍 코드를 통해 직접 설정할 수도 있지만, 일반적으로는 외부 XML 프로파일을 통해 설정하는 것이 권장된다. XML을 사용하면 애플리케이션을 재컴파일하지 않고도 시스템의 동작을 변경할 수 있어 유연성과 유지보수성이 크게 향상된다.31

### 3.2  실시간 시스템을 위한 핵심 QoS 정책

수많은 QoS 정책 중에서도 실시간 분산 시스템의 동작에 가장 큰 영향을 미치는 핵심 정책들은 다음과 같다.

**표 3: 주요 QoS 정책과 그 영향**

| QoS 정책 이름    | 주요 파라미터                            | 주요 영향                                                    | 일반적인 사용 사례                                           |
| ---------------- | ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `RELIABILITY`    | `kind` (BEST_EFFORT, RELIABLE)           | 데이터 샘플의 전송 보장 여부를 결정한다.                     | **RELIABLE:** 손실되면 안 되는 명령어, 상태 정보. **BEST_EFFORT:** 최신 값이 중요한 고주파 센서 데이터. |
| `DURABILITY`     | `kind` (VOLATILE, TRANSIENT_LOCAL, etc.) | 늦게 참여하는 `DataReader`에 대한 과거 데이터의 제공 여부와 방식을 제어한다. | **TRANSIENT_LOCAL:** 새로운 참여자가 즉시 최신 상태를 알 수 있도록 할 때. |
| `HISTORY`        | `kind` (KEEP_LAST, KEEP_ALL), `depth`    | `HistoryCache`에 저장할 과거 데이터 샘플의 수를 결정한다.    | **KEEP_ALL:** 신뢰성 있는 통신에서 데이터 손실을 막기 위해. **KEEP_LAST:** 최신 N개의 데이터만 유지할 때. |
| `DEADLINE`       | `period`                                 | 데이터가 명시된 주기 내에 생성/수신되어야 함을 강제하고, 위반 시 알림을 발생시킨다. | 데이터 스트림의 활성 상태 및 업데이트 주기를 감시할 때.      |
| `LATENCY_BUDGET` | `duration`                               | 데이터의 긴급성을 미들웨어에 알려 종단 간(end-to-end) 지연 시간을 최적화하도록 유도한다. | 지연 시간에 민감한 제어 루프나 사용자 인터랙션 데이터에 사용. |
| `OWNERSHIP`      | `kind` (SHARED, EXCLUSIVE), `strength`   | 동일 토픽에 여러 `Writer`가 있을 경우, 데이터 인스턴스의 소유권을 어떻게 관리할지 결정한다. | **EXCLUSIVE:** 주/백업(Primary/Backup) 구조의 내고장성 시스템을 구현할 때. |

- **RELIABILITY (신뢰성):**

  - `BEST_EFFORT`: "발송 후 망각(fire-and-forget)" 모드. 미들웨어는 손실된 샘플을 복구하려 시도하지 않는다. 최신 값이 개별 샘플의 손실보다 중요한 고주파 센서 데이터에 이상적이다.36
  - `RELIABLE`: 미들웨어가 모든 샘플의 전송을 보장한다. 이는 섹션 2.4에서 설명한 `HEARTBEAT`/`ACKNACK` 메커니즘을 통해 달성된다. 절대 손실되어서는 안 되는 명령어, 상태 변경, 이벤트 데이터에 필수적이다.32

- **DURABILITY (내구성):**

  - "늦게 참여하는(late-joining)" `DataReader`에 대한 데이터 가용성을 제어한다.
  - `VOLATILE`: `Reader`는 자신이 네트워크에 참여한 이후에 발행된 샘플만 수신한다.37
  - `TRANSIENT_LOCAL`: `DataWriter`가 최신 샘플들을 캐시에 보관하므로, 늦게 참여하는 `DataReader`는 원본 `DataWriter`가 살아있는 동안에는 가장 최근의 상태 정보를 받을 수 있다.37
  - `TRANSIENT` & `PERSISTENT`: 별도의 "영속성 서비스(Persistence Service)"를 사용하여 데이터를 각각 메모리나 디스크에 저장한다. 이를 통해 데이터가 원본 발행자의 생명 주기를 초월하여 존재할 수 있게 된다.37

- **HISTORY (이력):**

  - 각 데이터 인스턴스에 대해 `HistoryCache`에 얼마나 많은 과거 샘플을 저장할지를 결정한다.

  - `KEEP_LAST (depth)`: 가장 최근의 `N`개 샘플을 저장한다.

  - `KEEP_ALL`: `RESOURCE_LIMITS` QoS 정책에 의해 제한될 때까지 모든 샘플을 저장한다.32

    `RELIABLE` 통신에서 `Writer`가 `Reader`의 확인을 받기 전에 데이터를 버리는 것을 방지하기 위해 매우 중요하다.

- **TIMELINESS & LATENCY (적시성 및 지연시간):**

  - `DEADLINE`: 새로운 데이터 샘플이 지정된 기간 내에 발행(writer 측)되거나 수신(reader 측)되어야 한다는 계약을 설정한다. 마감 시간을 놓치면 상태 변경이 트리거되어 데이터 스트림의 상태와 업데이트 주기를 모니터링하는 데 사용된다.33
  - `LATENCY_BUDGET`: 데이터의 긴급성에 대한 힌트를 미들웨어에 제공하여, 발행부터 수신까지의 종단 간 지연 시간을 최적화하도록 유도한다.33

- **OWNERSHIP (소유권):**

  - 동일한 토픽에 대해 여러 `DataWriter`가 존재하는 시나리오를 관리한다.
  - `SHARED`: 구독자는 모든 `Writer`로부터 데이터를 수신할 수 있다.
  - `EXCLUSIVE`: 한 번에 하나의 `DataWriter`만이 데이터 인스턴스를 "소유"하고 업데이트할 수 있다. 경쟁 상황에서는 `OWNERSHIP_STRENGTH` QoS 정책이 어떤 `Writer`가 우선권을 가질지 결정하는 데 사용된다. 이는 주/백업 데이터 소스를 갖는 내고장성 시스템을 구현하는 데 매우 중요하다.42

### 3.3  QoS 설정 모범 사례 및 일반적인 패턴

실제 시스템 설계 시, 통신 패턴에 따라 QoS를 조합하여 사용하는 것이 효과적이다. RTI의 자료 등에서 제시된 일반적인 패턴은 다음과 같다.44

- **스트리밍/주기적 데이터 (예: 센서 피드):** `RELIABILITY`는 `BEST_EFFORT`로, `HISTORY`는 `KEEP_LAST`와 작은 `depth` 값으로 설정하는 경우가 많다. `DEADLINE` 정책을 사용하여 데이터가 오래되지 않았는지 모니터링한다.
- **상태 데이터 (예: 시스템 상태, 객체 위치):** 새로운 구독자가 즉시 최신 상태를 알 수 있도록 하는 것이 중요하다. 따라서 `RELIABILITY`는 `RELIABLE`로, `HISTORY`는 `KEEP_LAST` (depth=1)로, `DURABILITY`는 `TRANSIENT_LOCAL`로 설정하는 것이 일반적이다.
- **이벤트/명령어 데이터 (예: 사용자 클릭, 제어 신호):** 단 하나의 메시지도 놓치지 않아야 하므로, `RELIABILITY`는 `RELIABLE`로, `HISTORY`는 `KEEP_ALL`로 설정하여 모든 명령어가 확실히 전달되도록 보장한다.

QoS 정책들은 독립적인 설정값이 아니라, 복잡하게 상호 연결된 시스템을 형성한다. 예를 들어, 의미 있는 `RELIABLE` 통신을 달성하기 위해서는 `HISTORY` QoS를 `KEEP_ALL`이나 충분히 큰 `depth`를 가진 `KEEP_LAST`로 설정하고, `RESOURCE_LIMITS`를 통해 충분한 메모리를 할당해야 한다. 마찬가지로 `TRANSIENT_LOCAL` 이상의 `DURABILITY`는 `RELIABILITY`가 `RELIABLE`로 설정되었을 때만 효과적이다.40 이는 효과적인 QoS 설정이 단순한 체크리스트 작업이 아니라 전체적인 시스템 설계 활동임을 시사한다. 이러한 상호 의존성을 이해하지 못하면 예기치 않고 명확하지 않은 시스템 오류로 이어질 수 있다.

이러한 QoS 모델의 풍부함은 RTPS의 가장 큰 강점이자 동시에 가장 큰 진입 장벽이다. 이는 MQTT와 같은 더 단순한 프로토콜과는 비교할 수 없는 수준의 정밀한 제어 능력을 제공한다.45 그러나 이 강력한 기능은 상당한 복잡성을 동반한다.30 이는 DDS/RTPS가 모든 경우에 적합한 해결책이 아님을 의미한다. 이 기술은 이러한 수준의 제어가 

*요구사항*인 복잡한 시스템에 가장 적합하며, 해당 시스템을 구축하는 엔지니어링 팀이 이 복잡성을 관리할 전문 지식을 갖추고 있을 때 그 진가를 발휘한다.

------

## 4.  보안, 취약점 및 네트워크 제약 사항

### 4.1  DDS 보안 아키텍처: 플러그형 프레임워크

OMG DDS 보안 명세는 핵심 DDS 표준에 대한 확장 기능으로 제공된다.22 이는 본래의 RTPS 프로토콜에 내장된 기능이 아니라, 그 위에 계층적으로 추가된 서비스 집합이다. 이 아키텍처의 핵심은 **서비스 플러그인 인터페이스(Service Plugin Interface, SPI)**로, 보안 기능을 모듈식으로 구현할 수 있게 한다.47

다섯 가지 핵심 SPI는 다음과 같다:

1. **인증 (Authentication):** 참여자의 신원을 확인한다. 일반적으로 공개 키 기반 구조(PKI)와 X.509 인증서, 그리고 공유된 인증 기관(CA)을 사용한다.47
2. **접근 제어 (Access Control):** 인증된 참여자가 수행할 수 있는 DDS 관련 작업을 제한한다. 서명된 거버넌스 및 권한 파일을 사용하여 참여자가 가입할 수 있는 도메인, 발행 또는 구독할 수 있는 토픽을 정의한다.47
3. **암호화 (Cryptography):** 데이터 페이로드의 암호화/복호화(예: AES-GCM 사용) 및 메시지 무결성을 위한 서명/해싱 등 모든 암호화 작업을 처리한다.47
4. **로깅 (Logging):** 보안과 관련된 모든 이벤트를 감사하기 위한 표준화된 인터페이스를 제공한다.48
5. **데이터 태깅 (Data Tagging):** 데이터 샘플에 보안 태그를 추가하는 방법을 제공하며, 이는 접근 제어 메커니즘에서 활용될 수 있다.48

DDS 표준을 준수하는 구현체는 벤더 간 상호운용 가능한 보안을 보장하기 위해 이러한 SPI에 대한 내장(built-in) 플러그인을 의무적으로 지원해야 한다.47

### 4.2  알려진 취약점에 대한 비판적 분석 (Trend Micro 보고서 기반)

DDS/RTPS의 보안 태세는 Trend Micro의 상세한 연구를 통해 심도 있게 분석될 수 있다.50 이 분석은 프로토콜의 이론적 보안 모델과 실제 구현 및 배포 환경 간의 격차를 명확히 보여준다.

- **발견 프로토콜 악용:** RTPS 발견 프로토콜은 UDP 상에서 동작하며 통신이 활발한(chatty) 특성을 가진다. 이로 인해 네트워크 반사 및 증폭 공격에 취약하다. 보안 기능을 활성화하더라도 발견 과정에서 교환되는 메타트래픽은 완전히 인증되지 않아 스푸핑(spoofing)이 가능하다.50

- **악의적인 패킷 취약점:** 여러 DDS 구현체에서 특수하게 조작된 RTPS 패킷을 통해 서비스 거부(DoS) 공격이 가능함이 밝혀졌다. 이러한 패킷은 무한 루프, 메모리 고갈, 또는 세그멘테이션 폴트를 유발할 수 있다 (예: OpenDDS, GurumDDS, Connext DDS의 CVE).50

- **설정 파일의 공격 표면:** 많은 DDS 구현체가 QoS 및 기타 설정을 위해 XML 파일을 광범위하게 사용한다. 이는 상당한 공격 표면을 형성하며, 조작된 XML 파일을 통해 스택 오버플로나 힙 쓰기(heap-write)가 가능한 다수의 CVE가 보고되었다.50 일부 구현체에서 유지보수가 중단된 XML 파싱 라이브러리(예: GurumDDS의 

  `ezXML`)를 사용하는 것은 이 위험을 더욱 증폭시킨다.50

- **DevOps 및 배포 실패:** 실제 배포 환경에서의 보안 허점도 다수 발견되었다. 기본 자격 증명으로 인터넷에 노출된 CI/CD 파이프라인이나, 알려진 취약점을 포함한 오래된 Docker 이미지를 사용하는 사례가 있었다.50

- **정보 유출:** 공개적으로 접근 가능한 DDS 엔드포인트 중 상당수가 내부 사설 IP 주소와 같은 민감한 네트워크 정보를 유출하고 있었으며, 이는 공격자에게 중요한 정찰 데이터를 제공한다.50

### 4.3  보안 기능의 성능 오버헤드

보안 기능을 활성화하면 필연적으로 성능 저하가 발생한다.52 모든 메시지나 서브메시지에 대한 암호화 및 서명 작업은 계산 부하를 증가시켜 지연 시간을 늘리고 CPU 사용량을 높이며, 최대 처리량을 감소시킬 수 있다. 발견 과정에서의 보안 핸드셰이크 역시 초기 연결 지연 시간을 증가시킨다.

핵심은 보안과 원시 성능(raw performance) 간에는 트레이드오프 관계가 존재한다는 것이다. 시스템 설계자는 "모든 것을 암호화"하는 일괄적인 정책을 적용하기보다, 어떤 토픽에 어떤 수준의 보안이 필요한지를 신중하게 결정해야 한다. 하드 리얼타임 데이터 스트림에 과도한 보안을 적용하는 것은 시스템 요구사항을 충족하지 못하게 할 수 있다.52

### 4.4  네트워크 장벽 극복: WAN 문제

RTPS는 본래 근거리 통신망(LAN)을 위해 설계되었다. 초기 발견 단계에서 UDP 멀티캐스트에 의존하는 것은 공용 인터넷이나 광역 통신망(WAN) 환경에서의 배포에 큰 제약이 된다. 대부분의 인터넷 환경에서는 멀티캐스트가 지원되지 않기 때문이다.54

또한, 유니캐스트를 사용하더라도 네트워크 주소 변환(NAT) 방화벽은 심각한 장벽이다. NAT 뒤에 있는 참여자는 발견 과정에서 자신의 라우팅 불가능한 사설 IP 주소를 광고하게 되므로, 외부 참여자는 이 주소로 직접 도달할 수 없다.54

이러한 문제를 해결하기 위한 아키텍처적 해결책으로 **`RtpsRelay`**와 같은 개념이 등장했다.54

- `RtpsRelay`는 공개적으로 접근 가능한 중간 지점(rendezvous point) 역할을 한다.
- NAT 뒤의 참여자들은 릴레이 서버에 유니캐스트로 연결한다. 이때 NAT 방화벽은 이 아웃바운드 연결에 대한 바인딩을 생성한다.
- 릴레이는 참여자들 간의 발견 정보와 사용자 데이터를 전달함으로써, NAT를 통과하는 통신을 "중계"한다.
- 결과적으로 `RtpsRelay`는 메타트래픽 교환을 중앙에서 처리하여 발견 문제를 해결하고, 참여자들이 연결할 수 있는 안정적인 공용 주소를 제공하여 NAT 문제를 해결한다.54 RTI의 Cloud Discovery Service와 같은 다른 솔루션들도 유사한 원리로 동작한다.31

DDS 보안 명세가 핵심 RTPS 설계의 일부가 아닌 *확장 기능*이라는 역사적 맥락은 많은 취약점을 설명해준다. 기본 RTPS 프로토콜은 신뢰할 수 있는 네트워크를 가정하므로 발견 트래픽이 스푸핑될 수 있다. 보안 모델은 그 위에 계층적으로 추가되어, 보안이 적용된 요소와 그렇지 않은 요소 간의 복잡한 상호작용을 만들어낸다. 이는 DDS 시스템을 보호하는 것이 단순히 "보안 기능을 켜는" 것 이상임을 시사한다. 보안 플러그인이 무엇을 보호하는지, 그리고 더 중요하게는 기본적으로 무엇을 보호하지 *않는지*에 대한 깊은 이해가 필요하다. Trend Micro 보고서는 많은 심각한 취약점이 프로토콜 로직 자체가 아니라 구현(예: 패킷 파싱) 및 배포(예: XML 설정, 노출된 엔드포인트)에 있음을 보여준다. 이는 프로토콜 보안이 시스템 보안을 보장하지 않는다는 중요한 교훈을 준다.

------

## 5.  실제 적용 사례: 산업별 사용 사례 및 구현

### 5.1  로보틱스: ROS 2의 통신 백본

DDS/RTPS는 로봇 운영체제 ROS 2의 기본 미들웨어로 채택되어, ROS 1의 중앙 집중식 TCP 기반 아키텍처를 대체했다.30 로보틱스 분야에서 DDS/RTPS가 제공하는 핵심 이점은 다음과 같다:

- **탈중앙화된 발견:** ROS 1의 `roscore`와 같은 단일 장애점이 없어 시스템의 강건성이 향상된다.
- **실시간 성능 및 QoS:** 고대역폭 카메라 피드부터 저지연 제어 명령어에 이르기까지, 로봇 시스템의 다양한 데이터 스트림을 관리하기 위한 광범위한 QoS를 제공한다.55

**RMW(ROS Middleware Interface)**라는 추상화 계층을 통해 ROS 2 API는 특정 DDS 구현체로부터 분리된다. 이를 통해 사용자는 환경 변수 변경만으로 eProsima Fast DDS, Eclipse Cyclone DDS, RTI Connext 등 여러 벤더의 DDS 구현체를 전환하며 사용할 수 있다.55

하지만 이러한 접근 방식에는 특정 데이터 타입(예: `WString`)에 대한 벤더 간 상호운용성 문제나, ROS 2가 내부적으로 변환하는 토픽 이름 규칙으로 인해 "순수" DDS 애플리케이션과 연동하는 데 복잡성이 발생하는 등의 과제도 존재한다.55 한편, FPGA를 이용한 RTPS 하드웨어 가속(예: Robotcore RTPS)은 지연 시간을 마이크로초 단위까지 줄일 수 있어, RTPS가 극한의 실시간 제어 루프에도 적합함을 보여준다.58

### 5.2  자율 시스템: 자율주행차의 데이터 흐름

DDS/RTPS는 자율주행차 및 첨단 운전자 보조 시스템(ADAS)을 위한 핵심 기술로 자리매김하고 있다.14 주요 사용 사례는 다음과 같다:

- **고처리량 센서 데이터 융합:** 라이다(LIDAR), 레이더(RADAR), 카메라 등 다중 센서로부터의 데이터를 실시간으로 융합한다.
- **신뢰성 있는 데이터 분산:** 인지 및 계획 모듈 간의 데이터를 신뢰성 있게 분산한다.
- **저지연 차량 제어:** 차량 제어 명령어를 낮은 지연 시간으로 전달한다.53

DDS는 **AUTOSAR Adaptive**와 같은 차량용 소프트웨어 표준에서도 서비스 지향 아키텍처의 통신 바인딩으로 명시되어 있다.46 이 분야에서는 QoS 정책이 안전에 직결된다. 예를 들어, 

`DEADLINE`과 `LATENCY_BUDGET`은 장애물 감지와 같은 안전 필수 데이터가 시간 제약 내에 처리되도록 보장하는 데 사용되며 61, 

`RELIABILITY`는 제동과 같은 핵심 명령어가 절대 손실되지 않도록 보장한다.

그러나 대용량 데이터로 인한 UDP 단편화 문제나, 복잡한 차량 내 수많은 노드로 인해 도메인 당 참여자 수 제한이 문제가 될 수 있다는 점은 해결해야 할 과제이다.59

### 5.3  항공우주 및 국방: 미션 크리티컬 상호운용성

이 분야는 견고하고 신뢰성 있으며 상호운용 가능한 시스템에 대한 요구가 매우 높아 DDS/RTPS의 주요 적용 영역이 되었다.12

- **해군 시스템:** 이지스(Aegis) 전투 시스템, 함정 자체 방어 시스템(SSDS), 연안전투함(LCS), 줌왈트급 구축함(DDG 1000) 등 주요 해군 전투 관리 시스템에 적용되었다.13
- **항공전자:** **FACE(Future Airborne Capability Environment)** 표준에서 핵심적인 역할을 한다. FACE는 항공기 시스템의 모듈화 및 개방형 아키텍처를 촉진하여 비용을 절감하고 새로운 기능의 통합을 가속화하는 것을 목표로 한다.14
- **다영역 작전 (Multi-Domain Operations, MDO):** DDS는 육상, 해상, 공중, 우주 자산의 정보를 통합하는 통일된 데이터 공간을 생성하여 MDO 비전을 실현하는 데 기여한다. 다중 보안 등급 및 접근 제어를 관리하기 위해 DDS 보안 기능이 여기에서 매우 중요하다.42
- **무인 시스템 (UAVs):** 무인 항공기의 지휘 통제, 센서 데이터 전파, 탑재 장비 관리 등에 널리 사용된다.64

### 5.4  산업용 IoT, 자동화 및 스마트 그리드

- **산업 자동화:** RTPS/DDS는 PLC, 로봇, 제조 실행 시스템(MES) 등을 통합하는 "제조 서비스 버스(Manufacturing Service Bus)" 역할을 한다. 공장 현장에서 요구되는 실시간성, 확장성, 신뢰성을 제공한다.67
- **스마트 그리드:** 스마트 미터, 배전 자동화 시스템, 제어 센터를 연결하는 스마트 그리드의 복잡한 통신 문제를 해결하기 위한 솔루션으로 제안된다. QoS 정책은 높은 우선순위의 고장 데이터와 낮은 우선순위의 검침 데이터 등 다양한 데이터 유형과 신뢰성 요구사항을 관리하는 데 이상적이다.70 다만, 완전한 DDS 구현은 리소스가 제한된 그리드 말단 장치에는 부담이 될 수 있다.71

모든 사용 사례에서 반복적으로 나타나는 패턴은, RTPS가 일반적인 메시징 솔루션으로 선택된 것이 아니라, 그 특유의 아키텍처적 특징(탈중앙화, 데이터 중심, QoS 기반 제어)이 해당 분야의 근본적인 문제와 직접적으로 부합하기 때문에 채택되었다는 점이다. 예를 들어, 로보틱스는 예측 불가능한 환경에서 동적인 실시간 데이터 스트림을 관리해야 하므로 RTPS의 플러그 앤 플레이 발견과 정교한 QoS가 완벽하게 들어맞는다. 항공우주 분야는 여러 벤더로부터 공급받는 부품으로 구성된 장기 수명의 내고장성 시스템을 구축해야 하므로, RTPS의 개방형 표준과 탈중앙화 아키텍처가 필수적이다. 이처럼 RTPS는 단순한 프로토콜을 넘어, 각 시스템의 아키텍처의 핵심적인 부분인 **데이터 중심 통합 패브릭(data-centric integration fabric)**으로 기능한다.

------

## 6.  비교 분석 및 미래 방향

### 6.1  RTPS/DDS 대 브로커 중심 프로토콜: 두 아키텍처 이야기

RTPS를 더 넓은 메시징 기술 환경 내에서 이해하기 위해, MQTT 및 AMQP와 같은 브로커 중심 프로토콜과의 비교는 필수적이다. 이들의 차이점은 단순히 기능의 유무가 아니라 근본적인 아키텍처 철학에서 비롯된다.

**표 4: DDS/RTPS 대 MQTT 대 AMQP 비교 분석**

| 특징                   | DDS/RTPS                           | MQTT                                     | AMQP                                       |
| ---------------------- | ---------------------------------- | ---------------------------------------- | ------------------------------------------ |
| **아키텍처**           | 브로커리스 (Brokerless), P2P       | 브로커 기반 (Broker-based)               | 브로커 기반 (Broker-based)                 |
| **데이터 모델**        | 데이터 중심 (Data-Centric)         | 메시지 중심 (Message-Centric)            | 메시지 중심 (Message-Centric)              |
| **주요 전송 계층**     | UDP (TCP 옵션)                     | TCP                                      | TCP                                        |
| **신뢰성 모델**        | 엔드포인트 간 신뢰성 (자체 구현)   | 클라이언트-브로커 간 신뢰성              | 클라이언트-브로커 간 신뢰성, 트랜잭션 지원 |
| **발견 메커니즘**      | 동적 자동 발견 (P2P)               | 브로커를 통한 정적 연결                  | 브로커를 통한 정적 연결                    |
| **QoS 풍부함**         | 매우 풍부함 (22+ 정책)             | 기본적 (3단계)                           | 기본적 (2단계)                             |
| **보안**               | 포괄적 (인증, 접근제어, 암호화)    | 기본적 (사용자/암호, TLS)                | 유연함 (SASL, TLS)                         |
| **일반적 지연시간**    | 매우 낮음                          | 낮음-중간                                | 낮음-중간                                  |
| **이상적인 사용 사례** | 실시간 제어, 자율 시스템, 로보틱스 | IoT 센서 네트워크, 모바일, 클라우드 연동 | 금융 거래, 엔터프라이즈 메시징             |

- **아키텍처:**
  - **DDS/RTPS:** 브로커가 없는 P2P 방식으로, 참여자들이 서로 직접 통신한다. 이는 지연 시간을 최소화하고 단일 장애점을 제거하는 장점이 있다.45
  - **MQTT/AMQP:** 모든 통신이 중앙 브로커를 거치는 클라이언트-서버 모델이다. 이는 클라이언트 로직을 단순화하고 네트워크 장벽(NAT 등) 통과를 용이하게 하지만, 브로커가 잠재적인 성능 병목점이자 단일 장애점이 될 수 있다.45
- **데이터 모델:**
  - **DDS/RTPS:** **데이터 중심**이다. 미들웨어는 IDL을 통해 정의된 데이터의 구조, 내용, 생명주기를 인지한다. 이는 "글로벌 데이터 공간"이라는 추상적인 개념을 제공하여, 애플리케이션이 마치 공유 메모리에 접근하듯 데이터를 다룰 수 있게 한다.45
  - **MQTT/AMQP:** **메시지 중심**이다. 브로커는 페이로드를 의미를 알 수 없는 바이트 덩어리(opaque blob)로 취급한다. 토픽 문자열을 기반으로 메시지를 라우팅할 뿐, 그 안의 데이터가 무엇인지는 알지 못한다.45

이러한 근본적인 차이는 각 프로토콜의 이상적인 사용 사례를 결정한다. MQTT는 간단한 센서 데이터를 클라우드로 전송하는 IoT 시나리오에 최적화되어 있다. 반면, DDS/RTPS는 지능적인 피어(peer)들 간에 상태 정보를 실시간으로 동기화해야 하는 자율 시스템의 내부 통신망에 가장 적합하다. 이들은 상호 대체 가능한 기술이 아니라, 서로 다른 문제를 해결하는 도구이다.

### 6.2  다중 벤더 상호운용성: 약속과 현실

DDSI-RTPS 표준은 벤더 간 상호운용성의 핵심이다.77 OMG DDS 특별 관심 그룹(SIG)은 지속적으로 상호운용성 테스트를 수행하고 그 결과를 공개함으로써, 사용자들이 표준을 신뢰하고 여러 벤더의 제품을 혼용할 수 있도록 지원한다.78

그러나 현실에서는 몇 가지 도전 과제가 존재한다. 핵심 프로토콜은 상호운용 가능하지만, 벤더들은 독자적인 확장 기능을 제공하거나 명세의 선택적 부분에 대한 지원 수준이 다를 수 있다. ROS 2 문서에서 지적된 바와 같이, 특정 데이터 타입(`WString` 등)이나 설정 조합에서는 특정 벤더 간에 통신이 원활하지 않을 수 있다.55 이는 상호운용성이 목표이며 대부분의 경우 잘 동작하지만, 100% "플러그 앤 플레이"를 보장하는 것은 아니며, 신중한 설정과 테스트가 필요할 수 있음을 의미한다.

### 6.3  RTPS의 미래: 시간 민감형 네트워킹(TSN)과의 통합

**TSN(Time-Sensitive Networking)**은 표준 이더넷 네트워크에 결정론적 성능(보장된 대역폭, 제한된 저지연 및 지터)을 제공하는 IEEE 802 표준 집합이다.80

DDS와 TSN은 매우 상호보완적이다. DDS는 애플리케이션/전송 계층에서 동작하며 데이터의 QoS를 관리하고, TSN은 데이터 링크 계층에서 동작하며 네트워크 패킷의 QoS를 관리한다.81 이 둘을 결합하면, DDS에서 정의한 애플리케이션 수준의 QoS 요구사항(예: 

`LATENCY_BUDGET`)을 TSN이 제공하는 보장된 네트워크 리소스에 직접 매핑할 수 있다. 이는 강력한 종단 간 결정론적 통신 스택을 생성한다.80 OMG는 이 매핑을 정의하는 "DDS-TSN" 표준을 제정했다.22

이 통합 기술은 자율주행차, 산업 자동화, 항공전자 등 융합 이더넷 네트워크 상에서 결정론적 통신이 필수적인 분야의 미래에 매우 중요한 역할을 할 것으로 기대된다.80

### 6.4  진화하는 아키텍처: 엣지에서 클라우드까지

DDS/RTPS는 현대적인 엣지-투-클라우드(edge-to-cloud) 아키텍처에서 중요한 역할을 한다. DDS는 실시간 기계 간 통신이 필요한 "엣지"에서 특히 강력한 성능을 발휘한다.75

하지만 LAN 기반의 발견 메커니즘에 의존하기 때문에 클라우드와의 직접적인 통합은 어렵다. 이 때문에 엣지 DDS 도메인을 클라우드에 연결하기 위해서는 `RtpsRelay`나 RTI Cloud Discovery Service와 같은 게이트웨이나 브리징 솔루션이 필요한 경우가 많다.88 군사적 맥락에서의 "클라우드렛(Cloudlets)" 또는 "포그 컴퓨팅(Fog Computing)" 개념은 계층적 DDS 아키텍처의 완벽한 예시이다. 이 아키텍처에서 DDS는 전술적 엣지 클러스터 내에서 견고한 통신을 제공하고, 이 클러스터들은 중앙 클라우드와 간헐적으로 통신한다.86

미래에는 DDS 도메인을 WAN을 통해 연결하고 다른 클라우드 네이티브 프로토콜과 통합하는 Zenoh와 같은 더욱 정교한 게이트웨이 솔루션이 부상할 것이다. 이는 엣지에서의 DDS의 실시간 강점과 클라우드의 확장성을 결합하는 하이브리드 아키텍처를 가능하게 할 것이다.88

결론적으로, RTPS의 미래는 하이브리드 형태가 될 가능성이 높다. RTPS는 진공 상태에서 존재하지 않을 것이며, 그 미래의 강점은 다른 기술과의 통합에 달려 있다. TSN 하드웨어와의 결합은 중요 데이터에 대한 결정론적 "고속 경로"를 만들고, 클라우드 게이트웨이 기술과의 결합은 WAN을 넘어선 견고한 다리를 놓을 것이다.

------

## 7. 결론

### 7.1 핵심 결과 종합

본 보고서는 RTPS(Real-Time Publish-Subscribe) 프로토콜이 DDS(Data Distribution Service)의 상호운용성을 보장하는 고성능, 데이터 중심의 유선 프로토콜임을 종합적으로 분석했다. RTPS의 핵심적인 강점은 내고장성과 동적 '플러그 앤 플레이' 발견 메커니즘을 가능하게 하는 탈중앙화된 P2P 아키텍처, 그리고 실시간 데이터 분산에 대한 정밀한 제어를 제공하는 풍부한 QoS(서비스 품질) 모델에 있다. 이러한 특성 덕분에 RTPS는 단순한 메시지 전달을 넘어, 분산된 노드 간에 일관된 데이터 상태를 유지하고 동기화하는 강력한 메커니즘을 제공한다.

### 7.2 핵심 가치 제안의 재확인

RTPS는 단순한 메시징 프로토콜이 아니라, 복잡하고 동적인 실시간 분산 시스템을 구축하기 위한 아키텍처 프레임워크 그 자체이다. 프로토콜이 데이터의 구조와 의미를 이해하는 '데이터 중심' 모델은 RTPS를 다른 기술과 구별하는 가장 큰 차별점이며, 로보틱스, 자율주행, 국방, 산업 자동화와 같이 극도의 신뢰성과 성능을 요구하는 까다로운 분야에서 성공적으로 채택된 근본적인 이유이다.

### 7.3 미래 전망

RTPS는 지능형 시스템의 시대에 핵심적인 표준으로 계속해서 진화할 것이다. 그 미래의 중요성은 독립적인 프로토콜로서의 기능보다는 다른 기술과의 깊은 통합을 통해 정의될 것이다. 하드웨어 수준에서는 시간 민감형 네트워킹(TSN)과의 결합을 통해 종단 간 결정론적 통신을 실현하고, 시스템 아키텍처 수준에서는 게이트웨이 기술을 통해 엣지 컴퓨팅의 실시간 강점과 클라우드의 확장성을 연결하는 중추적인 역할을 수행할 것이다. 결국 RTPS는 자율 시스템과 산업용 사물 인터넷(IIoT) 시대의 지능형 엣지를 위한 최고의 통신 패브릭으로서 그 입지를 더욱 공고히 할 것이다.

#### **참고 자료**

1. RTPS Introduction - eProsima, accessed July 4, 2025, https://www.eprosima.com/developer-resources/whitepapers/rtps
2. What is RTPS and DDSI? - ZettaScale Knowledge Base, accessed July 4, 2025, https://kbase.zettascale.tech/article/what-is-rtps-and-ddsi/
3. Real-time Publish-Subscribe (RTPS) [DDS Foundation Wiki], accessed July 4, 2025, https://www.omgwiki.org/ddsf/doku.php?id=ddsf:public:guidebook:06_append:glossary:r:rtps
4. RTP Protocol - Everything You Need To Know - 100MS, accessed July 4, 2025, https://www.100ms.live/blog/rtp-protocol
5. Real-time Transport Protocol - Wikipedia, accessed July 4, 2025, https://en.wikipedia.org/wiki/Real-time_Transport_Protocol
6. The Real-time Streaming Protocol (rtsp) - NetworkSecurity Resource - Tutor Hunt, accessed July 4, 2025, https://www.tutorhunt.com/resource/26949/
7. Protocols/rtps - Wireshark Wiki, accessed July 4, 2025, https://wiki.wireshark.org/Protocols/rtps
8. What protocol did DDS use before RTPS came about? - RTI Community, accessed July 4, 2025, https://community.rti.com/forum-topic/what-protocol-did-dds-use-rtps-came-about
9. reprint rti - Real-Time Innovations, accessed July 4, 2025, https://info.rti.com/hubfs/docs/RTC_Feb02.pdf
10. Real-Time Publish-Subscribe (RTPS) Protocol - NETWORX SECURITY, accessed July 4, 2025, https://www.networxsecurity.org/members-area/glossary/r/rtps.html
11. About the DDS Interoperability Wire Protocol Specification Version 2.5, accessed July 4, 2025, https://www.omg.org/spec/DDSI-RTPS/2.5/About-DDSI-RTPS
12. The Data Distribution Service: Reducing Cost through Agile Integration - Twin Oaks Computing, Inc, accessed July 4, 2025, https://www.twinoakscomputing.com/wp/DDS_Exec_Brief_v20l-public.pdf
13. Aerospace and Defense [DDS Foundation Wiki], accessed July 4, 2025, https://www.omgwiki.org/ddsf/doku.php?id=ddsf:public:applications:aerospace_and_defense
14. DDS Standard - Data Distribution Service for Complex Systems - RTI, accessed July 4, 2025, https://www.rti.com/products/dds-standard
15. About the DDS Interoperability Wire Protocol Specification Version 2.3, accessed July 4, 2025, https://www.omg.org/spec/DDSI-RTPS/2.3/About-DDSI-RTPS
16. DDS-RTPS - ARC-IT, accessed July 4, 2025, https://www.arc-it.net/html/standards/standard1306.html
17. Difference between DDS and RTPS - Stack Overflow, accessed July 4, 2025, https://stackoverflow.com/questions/59278753/difference-between-dds-and-rtps
18. 4. RTPS Layer - 3.2.2 - Fast DDS, accessed July 4, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/rtps_layer/rtps_layer.html
19. Fast DDS Documentation, accessed July 4, 2025, https://media.readthedocs.org/pdf/eprosima-fast-rtps/latest/eprosima-fast-rtps.pdf
20. DDS API - Fast DDS 2.14.4 documentation, accessed July 4, 2025, https://fast-dds.docs.eprosima.com/en/2.14.x/
21. DDS - eProsima, accessed July 4, 2025, https://www.eprosima.com/component/content/category/english-categories/resources/dds?Itemid=102
22. What's in the DDS Standard? - DDS Foundation, accessed July 4, 2025, https://www.dds-foundation.org/omg-dds-standard/
23. 4. Understanding the RTPS Packet Format - RTI Community - Real-Time Innovations, accessed July 4, 2025, https://community.rti.com/static/documentation/wireshark/2020-07/doc/understanding_rtps.html
24. The RTPS message structure. | Download Scientific Diagram - ResearchGate, accessed July 4, 2025, https://www.researchgate.net/figure/The-RTPS-message-structure_fig2_329035319
25. 1. Introduction - 3.2.2 - Fast DDS - eProsima, accessed July 4, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastddsgen/introduction/introduction.html
26. Common Data Representation - Wikipedia, accessed July 4, 2025, https://en.wikipedia.org/wiki/Common_Data_Representation
27. 59.3 DATA_REPRESENTATION QosPolicy - RTI Community, accessed July 4, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/users_manual/users_manual/DATAREPRESENTATION_Qos.htm
28. Detecting Retransmissions in Wireshark - Eclipse Cyclone DDS - FAQ, accessed July 4, 2025, https://cyclonedds.io/content/guides/detect-retransmits.html
29. What are preemptive ACKNACK and preemptive heartbeat messages? | Data Distribution Service (DDS) Community RTI Connext Users, accessed July 4, 2025, https://community.rti.com/kb/what-are-preemptive-acknack-and-preemptive-heartbeat-messages
30. ROS on DDS - ROS2 Design, accessed July 4, 2025, https://design.ros2.org/articles/ros_on_dds.html
31. RTI Cloud Discovery Service, accessed July 4, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/addon_products/cloud_discovery_service/RTI_Cloud_Discovery_Service_UsersManual.pdf
32. Quality of Service (QoS) policies are sets of requested policies for how - entities - OpenDDS, accessed July 4, 2025, https://opendds.readthedocs.io/en/master/devguide/quality_of_service.html
33. 4. Quality of Service - The Data Distribution Service Tutorial, accessed July 4, 2025, https://download.zettascale.online/www/docs/Vortex/html/ospl/DDSTutorial/qos.html
34. 5. Basic QoS - RTI Connext Getting Started documentation - RTI Community, accessed July 4, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/getting_started_guide/cpp11/intro_qos.html
35. USER_QOS_PROFILES.xml - RTI Community, accessed July 4, 2025, https://d2vkrkwbbxbylk.cloudfront.net/sites/default/files/rti-examples/rticonnextdds-examples/examples/connext_dds/keyed_data_advanced/c/USER_QOS_PROFILES.xml
36. Quality of Service - OpenDDS 3.28.1, accessed July 4, 2025, https://opendds.readthedocs.io/en/dds-3.28.1/devguide/quality_of_service.html
37. 59.9 DURABILITY QosPolicy - RTI Community, accessed July 4, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/users_manual/users_manual/DURABILITY_QosPolicy.htm
38. QoS - Eclipse Cyclone DDS, 0.11.0, accessed July 4, 2025, https://cyclonedds.io/docs/cyclonedds/latest/api/qos.html
39. Durability - OpenDDS 3.32.0, accessed July 4, 2025, https://opendds.readthedocs.io/en/latest-release/devguide/shapes/durability.html
40. 3.1.2.1. Standard QoS Policies - eProsima Fast DDS, accessed July 4, 2025, https://fast-dds.docs.eprosima.com/en/2.x/fastdds/dds_layer/core/policy/standardQosPolicies.html
41. Relations between DDS QoS Policies for Cyber-Physical Systems - ETRI KSP, accessed July 4, 2025, https://ksp.etri.re.kr/ksp/article/read?id=10359
42. Multi-Domain Operations: Why DDS is the Right Solution for Securing Critical Data, accessed July 4, 2025, https://www.rti.com/blog/multi-domain-operations-why-dds-is-the-right-solution-for-securing-critical-data
43. RTI® Connext®-Comprehensive Summary of QoS Policies, accessed July 4, 2025, https://community.rti.com/static/documentation/connext-dds/6.0.1/doc/manuals/connext_dds/RTI_ConnextDDS_CoreLibraries_QoS_Reference_Guide.pdf
44. RTI Academy - XML and Quality of Service - 4 Configuring QoS of DDS Entities - YouTube, accessed July 4, 2025, https://www.youtube.com/watch?v=nyhvEEgf7RQ
45. Navigating IIoT Protocols: Comparing DDS and MQTT - RTI, accessed July 4, 2025, https://www.rti.com/blog/comparing-dds-and-mqtt
46. DDS Middleware. What is DDS middleware: | by Huseyin Kutluca | Software Architecture Foundations | Medium, accessed July 4, 2025, https://medium.com/software-architecture-foundations/dds-middleware-1af525f69753
47. ROS 2 DDS-Security integration - ROS2 Design, accessed July 4, 2025, https://design.ros2.org/articles/ros2_dds_security.html
48. DDS Foundation Wiki - Service Plugin Interface (SPI), accessed July 4, 2025, https://www.omgwiki.org/ddsf/doku.php?id=ddsf:public:guidebook:06_append:glossary:s:service_plugin_interface
49. DDS Security - @PROJECT_NAME@ @PROJECT_VERSION@ documentation - Eclipse Cyclone DDS, accessed July 4, 2025, https://cyclonedds.io/docs/cyclonedds/0.10.3/security.html
50. A Security Analysis of the Data Distribution Service ... - Trend Micro, accessed July 4, 2025, https://documents.trendmicro.com/assets/white_papers/wp-a-security-analysis-of-the-data-distribution-service-dds-protocol.pdf
51. Analyzing the Data Distribution Service (DDS) Protocol for Critical Industries - Trend Micro, accessed July 4, 2025, https://www.trendmicro.com/vinfo/us/security/news/cybercrime-and-digital-threats/analyzing-the-data-distribution-service-dds-protocol-for-critical-industries
52. Master Thesis - DiVA portal, accessed July 4, 2025, http://www.diva-portal.org/smash/get/diva2:1876994/FULLTEXT01.pdf
53. Designing an Autonomous Vehicle for Real-World Performance, accessed July 4, 2025, https://www.rti.com/blog/designing-an-autonomous-vehicle-for-real-world-performance
54. Internet-Enabled RTPS - OpenDDS 3.33.0-dev - Read the Docs, accessed July 4, 2025, https://opendds.readthedocs.io/en/latest/devguide/internet_enabled_rtps.html
55. About different ROS 2 DDS/RTPS vendors, accessed July 4, 2025, https://docs.ros.org/en/foxy/Concepts/About-Different-Middleware-Vendors.html
56. 16. ROS 2 using Fast DDS middleware - 3.2.2, accessed July 4, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/ros2/ros2.html
57. Ros2 and DDS messaging - ROS General - Open Robotics Discourse, accessed July 4, 2025, https://discourse.ros.org/t/ros2-and-dds-messaging/1556
58. ROBOTCORE® RTPS | An FPGA implementation of DDS-RTPS communication middleware - Acceleration Robotics, accessed July 4, 2025, https://accelerationrobotics.com/robotcore-rtps.php
59. Performance Evaluation of ROS2-DDS middleware implementations facilitating Cooperative Driving in Autonomous Vehicle. - arXiv, accessed July 4, 2025, https://arxiv.org/html/2412.07485v1
60. Requirements on Data Distribution Service - AUTOSAR.org, accessed July 4, 2025, https://www.autosar.org/fileadmin/standards/R22-11/FO/AUTOSAR_RS_DataDistributionService.pdf
61. TS-DDS: Data Distribution Service (DDS) Over In-Vehicle Time-Sensitive Networking (TSN) Mechanism Research | Request PDF - ResearchGate, accessed July 4, 2025, https://www.researchgate.net/publication/375436491_TS-DDS_Data_Distribution_Service_DDS_Over_In-Vehicle_Time-Sensitive_Networking_TSN_Mechanism_Research
62. [2305.07147] COLA: Characterizing and Optimizing the Tail Latency for Safe Level-4 Autonomous Vehicle Systems - arXiv, accessed July 4, 2025, https://arxiv.org/abs/2305.07147
63. DDS Tuning for ROS 2 - breq.dev, accessed July 4, 2025, https://breq.dev/2024/05/17/dds
64. CoreDX DDS in Aerospace and Defense - Twin Oaks Computing, Inc, accessed July 4, 2025, https://www.twinoakscomputing.com/coredx/aerospace_n_defense
65. Modular Open Systems Approach (MOSA) - RTI, accessed July 4, 2025, https://www.rti.com/industries/aerospace-defense/all-domain/mosa
66. Multi-Domain Secure DDS Networks for Aerial and Ground Vehicle Communications - Clemson OPEN, accessed July 4, 2025, https://open.clemson.edu/cgi/viewcontent.cgi?article=5326&context=all_theses
67. Designing High Performance Factory Automation Applications on Top of DDS, accessed July 4, 2025, https://www.researchgate.net/publication/237052462_Designing_High_Performance_Factory_Automation_Applications_on_Top_of_DDS
68. Industrial Automation [DDS Foundation Wiki], accessed July 4, 2025, https://www.omgwiki.org/ddsf/doku.php?id=ddsf:public:applications:industrial_automation
69. Manufacturing Process Automation | Use Case - Real-Time Innovations, accessed July 4, 2025, https://www.rti.com/developers/case-code/process-automation
70. Adopting DDS to Smart Grids: Towards Reliable Data Communication - ResearchGate, accessed July 4, 2025, https://www.researchgate.net/publication/318973016_Adopting_DDS_to_Smart_Grids_Towards_Reliable_Data_Communication
71. Configurable DDS as Uniform Middleware for Data Communication in Smart Grids - MDPI, accessed July 4, 2025, https://www.mdpi.com/1996-1073/13/7/1839
72. Comparing the Performance of Zenoh, MQTT, Kafka, and DDS, accessed July 4, 2025, https://zenoh.io/blog/2023-03-21-zenoh-vs-mqtt-kafka-dds/
73. Comparing OpenSplice DDS to RabbitMQ - ZettaScale Knowledge Base, accessed July 4, 2025, https://kbase.zettascale.tech/article/comparing-vortex-opensplice-rabbitmq/
74. AMQP vs MQTT: Messaging protocols compared - CloudAMQP, accessed July 4, 2025, https://www.cloudamqp.com/blog/amqp-vs-mqtt.html
75. What is DDS? - DDS Foundation, accessed July 4, 2025, https://www.dds-foundation.org/what-is-dds-3/
76. Taking on Industrial IoT with Real-Time DDS - EE Times, accessed July 4, 2025, https://www.eetimes.com/taking-on-industrial-iot-with-real-time-dds/
77. data distribution service - Interoperability in DDS - Stack Overflow, accessed July 4, 2025, https://stackoverflow.com/questions/34895092/interoperability-in-dds
78. Interoperability - DDS Foundation, accessed July 4, 2025, https://www.dds-foundation.org/interoperability/
79. Validation of interoperability of products compliant with OMG DDS-RTPS standard. - GitHub, accessed July 4, 2025, https://github.com/omg-dds/dds-rtps
80. DDS and TSN: Converging Two Communications Standards - Real-Time Innovations, accessed July 4, 2025, https://www.rti.com/dds-and-tsn
81. TSN and DDS: Enhancing Real-Time Communication - Pavel Pohanka, accessed July 4, 2025, https://pavelpohanka.cz/en/blog/tsn-and-dds-enhancing-real-time-communication/
82. Time Sensitive Networking (TSN) Tutorial and Reference Guide, accessed July 4, 2025, https://www.ueidaq.com/zh/tsn-tutorial-and-reference-guide
83. Critical Data Delivery with Time Sensitive Networking | Unmanned Systems Technology, accessed July 4, 2025, https://www.unmannedsystemstechnology.com/feature/critical-data-delivery-with-time-sensitive-networking/
84. Using DDS to Unlock the Power of TSN | RTI - Real-Time Innovations, accessed July 4, 2025, https://content.rti.com/whitepaper-using-dds-to-unlock-the-power-of-tsn
85. TSN and DDS® Integration for Real-Time Communication - YouTube, accessed July 4, 2025, https://www.youtube.com/watch?v=uruqV5xqoNo
86. Why DDS is Essential for the Tactical Edge - RTInsights, accessed July 4, 2025, https://www.rtinsights.com/network-centric-warfare-edge-computing-cloudlets-dds/
87. DDS at the Tactical Edge Whitepaper - Object Management Group, accessed July 4, 2025, https://www.omg.org/news/whitepapers/DDS-Tactical-Edge-Whitepaper.pdf
88. Edge, Fog and Cloud from a DDS® perspective - YouTube, accessed July 4, 2025, https://www.youtube.com/watch?v=rDl1Hf1_5CE


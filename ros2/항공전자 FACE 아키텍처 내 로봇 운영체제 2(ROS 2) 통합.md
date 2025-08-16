# 항공전자 FACE 아키텍처 내 로봇 운영체제 2(ROS 2) 통합

## 1.  기본 아키텍처: 비교 개요

이 섹션에서는 두 개의 개별 소프트웨어 생태계인 로봇 운영체제 2(ROS 2)와 미래 공수 능력 환경(Future Airborne Capability Environment, FACE)에 대한 공통된 이해를 구축합니다. 각 시스템의 목표, 설계 철학 및 핵심 아키텍처 구성 요소를 상세히 설명하여 후속 통합 분석의 기반을 마련합니다.

### 1.1  FACE 표준: 항공전자를 위한 모듈식 개방형 시스템 접근법(MOSA)

FACE 컨소시엄의 핵심 목표는 소프트웨어 이식성과 재사용성을 촉진하여 항공전자 능력의 개발 및 통합 비용을 절감하고 새로운 기능을 현장에 더 신속하게 배치하는 것입니다.1 이는 단순히 기술 표준을 넘어, 미 국방부(DoD)의 모듈식 개방형 시스템 접근법(Modular Open Systems Approach, MOSA) 지침을 지원하는 정부-산업 파트너십 기반의 비즈니스 전략이기도 합니다.2 FACE 접근법은 모듈식 설계, 핵심 인터페이스 지정, 그리고 개방형 비독점 표준 사용이라는 원칙에 기반하여 공급업체 간 경쟁을 촉진하고 특정 업체에 대한 종속성을 줄이는 것을 목표로 합니다.1

FACE 컨소시엄은 소프트웨어 구성요소(UoC, Unit of Conformance)가 표준의 인터페이스 요구사항을 준수하는지 검증하기 위한 공식적인 '적합성(Conformance) 프로그램'을 운영합니다. 이 적합성 인증은 이식성을 보장하지만, 항공기 운항에 필요한 안전성을 증명하는 감항 인증(airworthiness certification)과는 명확히 구분됩니다.3 즉, FACE 적합성은 시스템 간의 소프트웨어 재사용을 가능하게 하는 기술적 전제조건이며, 안전 인증을 위한 별도의 과정이 추가로 요구됩니다.

### 1.2  FACE 참조 아키텍처: 5개 세그먼트 해부

FACE 아키텍처는 시스템을 5개의 논리적 세그먼트로 분해하고, 이들 간의 표준화된 인터페이스를 정의함으로써 모듈성을 구현합니다.3 각 세그먼트의 역할은 다음과 같습니다.

- **운영체제 세그먼트(Operating System Segment, OSS):** 시스템의 가장 기초적인 계층으로, 파티셔닝, 프로세스 관리 등과 같은 핵심 서비스를 제공합니다. 이 세그먼트는 POSIX(Portable Operating System Interface) 기반의 '범용(General Purpose)' 프로파일, ARINC 653 표준 기반의 '안전(Safety)' 프로파일, 그리고 '보안(Security)' 프로파일로 나뉘며, 상위 모든 세그먼트는 이 OSS에 의존합니다.4
- **I/O 서비스 세그먼트(I/O Services Segment, IOSS):** 물리적인 입출력 장치에 대한 인터페이스를 표준화하여 하드웨어의 세부 사양을 상위 계층의 소프트웨어 구성요소로부터 추상화합니다.4
- **플랫폼 특정 서비스 세그먼트(Platform-Specific Services Segment, PSSS):** 데이터 로깅, 상태 관리, 그래픽 렌더링과 같이 플랫폼에 종속적인 서비스를 제공하는 역할을 담당합니다.4
- **전송 서비스 세그먼트(Transport Services Segment, TSS):** 시스템 내 통신을 담당하는 핵심 계층입니다. 다른 세그먼트들을 위해 통신 메커니즘을 추상화하고 통신 서비스를 제공하며, 특히 데이터 분산 서비스(Data Distribution Service, DDS)를 핵심 기술로 명시하고 있습니다.4
- **이식성 구성요소 세그먼트(Portable Components Segment, PCS):** 특정 플랫폼에 종속되지 않고 여러 시스템에 걸쳐 재사용되도록 설계된 이식 가능한 애플리케이션 로직, 즉 '비즈니스 로직'을 포함합니다.3

### 1.3  로봇 운영체제 2(ROS 2): 현대 로보틱스를 위한 프레임워크

ROS 2는 기존 ROS 1의 한계점, 특히 보안 부재, 실시간성 보장 미흡, 다중 로봇 시스템 지원 부족 등을 해결하기 위해 처음부터 완전히 재설계된 차세대 로봇 개발 프레임워크입니다.11 ROS 2의 핵심 아키텍처 특징은 다음과 같습니다.

- **모듈식 및 분산 구조:** 시스템은 독립적인 '노드(Node)'들로 구성되며, 이 노드들은 발행-구독(Publish-Subscribe) 메커니즘을 통해 통신합니다. 이는 시스템의 모듈성과 확장성을 크게 향상시킵니다.14
- **미들웨어 추상화:** ROS 2는 추상화된 미들웨어 인터페이스(RMW, ROS Middleware Interface)를 사용하여 기반 통신 프로토콜을 교체할 수 있도록 설계되었습니다. 기본적으로 가장 널리 사용되는 구현체는 DDS입니다.14
- **실시간 성능:** 향상된 멀티스레딩, 서비스 품질(QoS) 정책, 실시간 운영체제(RTOS)와의 호환성 등 실시간 애플리케이션을 지원하기 위한 기능들을 갖추고 설계되었습니다.12 하지만 결정론적인(deterministic) 하드 리얼타임(hard real-time) 성능을 달성하는 것은 여전히 상당한 엔지니어링 노력을 요구하는 과제입니다.20
- **보안(SROS 2):** DDS-Security 사양을 기반으로 구축된 포괄적인 보안 프레임워크를 통합하여 인증, 접근 제어, 암호화 기능을 제공합니다.11

### 1.4  ROS 2에서 데이터 분산 서비스(DDS)의 중심 역할

ROS 2의 차세대 통신 시스템으로 DDS가 채택된 것은 우연이 아닙니다. ROS 1의 전송 계층을 개선하거나 ZeroMQ와 같은 라이브러리를 사용하여 새로운 미들웨어를 구축하는 대신 DDS가 선택된 이유는, DDS가 이미 전함, 비행 시스템 등 다양한 미션 크리티컬 시스템에서 성능이 입증된 성숙한 개방형 표준이기 때문입니다.17 또한, DDS는 강력한 QoS 정책, 중앙 서버가 필요 없는 분산형 검색(discovery) 기능, 그리고 공식적인 사양을 제공합니다.17

ROS 2는 DDS와 매우 깊이 통합되어 있습니다. ROS 2의 노드는 DDS 참여자(Participant)에, ROS 2의 토픽, 서비스, 액션은 DDS의 토픽, 리더(Reader), 라이터(Writer)에 매핑됩니다. ROS 2의 QoS 설정 역시 DDS의 QoS 정책을 직접적으로 활용합니다.14 이러한 깊은 통합은 ROS 2가 DDS의 강력한 기능과 함께 그 복잡성까지 상속받는다는 것을 의미합니다. 또한, ROS 2는 eProsima Fast DDS(기본값), RTI Connext DDS, Eclipse Cyclone DDS 등 여러 DDS 벤더 구현체를 지원하도록 설계되어 특정 벤더에 대한 종속성을 피하려는 FACE의 원칙과도 일치합니다.18

이러한 배경을 통해 두 시스템의 근본적인 차이와 예상치 못한 공통점을 발견할 수 있습니다. ROS 2는 광범위한 로보틱스 애플리케이션을 위한 신속한 실험과 유연성에 철학적 뿌리를 두고 있는 반면 11, FACE는 항공전자라는 특정 도메인에서 엄격한 표준화, 이식성, 그리고 안전 인증 지원에 중점을 둡니다.1 이 두 철학은 상반되는 것처럼 보입니다. 그러나 ROS 2가 ROS 1의 한계를 극복하기 위해 강력하고 실시간성이 보장되는 미들웨어를 찾던 중 DDS를 선택했고, FACE 컨소시엄 역시 항공전자를 위한 모듈식 개방형 표준의 전송 서비스 기술로 DDS를 채택했다는 점은 주목할 만합니다.9 즉, 서로 다른 기원과 목표를 가진 두 생태계가 독립적으로 동일한 핵심 통신 기술인 DDS에 도달한 것입니다. 이 기술적 수렴은 두 시스템 간 통합을 가능하게 하는 가장 결정적인 가교 역할을 합니다.

또한, 두 표준이 모두 '이식성(portability)'을 강조하지만 그 의미에는 미묘한 차이가 있습니다. ROS 2에서 이식성은 주로 여러 운영체제(Linux, Windows, macOS)와 하드웨어 간의 호환성을 의미하며, 로봇 애플리케이션을 최소한의 코드 수정으로 다른 로봇에서 실행할 수 있게 하는 것을 목표로 합니다.12 반면, FACE에서의 이식성은 '이식성 단위(Unit of Portability, UoP)'가 서로 다른 주계약업체가 개발한 FACE 적합 시스템 간에 이동하여 올바르게 기능할 수 있음을 의미합니다. 이는 OSS, TSS와 같은 인터페이스가 표준화되어 있기 때문에 가능하며, 공식적인 적합성 프로세스를 통해 강제됩니다.3 따라서 ROS 2를 FACE에 통합하는 문제는 "ROS 2가 이식성이 있는가?"가 아니라, "ROS 2 애플리케이션을 FACE UoP로 패키징할 수 있는가?"라는 더 엄격한 기준으로 재정의되어야 합니다.

| 특징                       | 로봇 운영체제 2 (ROS 2)                                      | 미래 공수 능력 환경 (FACE)                                   |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **주요 목표**              | 로보틱스 애플리케이션 개발을 위한 유연하고 확장 가능한 프레임워크 제공 | 군용 항공전자 소프트웨어의 재사용성 및 이식성 증대를 통한 비용 절감 및 신속한 전력화 |
| **관리 주체**              | Open Robotics / 커뮤니티                                     | The Open Group 컨소시엄 (정부-산업 파트너십)                 |
| **핵심 아키텍처 패러다임** | 분산된 노드들로 구성된 계산 그래프                           | 5개 세그먼트로 구성된 계층적 아키텍처                        |
| **통신 미들웨어**          | 교체 가능한 RMW (기본값: DDS)                                | TSS (전송 서비스 세그먼트), DDS 명시                         |
| **강제 메커니즘**          | 커뮤니티 표준 및 관행                                        | 공식적인 적합성 인증 프로세스                                |
| **대상 도메인**            | 광범위한 로보틱스 (자율주행차, 드론, 산업용 로봇 등)         | 군용 항공 플랫폼                                             |
| **안전성 접근 방식**       | '실시간 지원 가능', DDS-Security 기반 보안 기능 제공         | 프로파일을 통한 '안전 인증 지원', 엄격한 인터페이스 정의     |

## 2.  결정적 연결고리: ROS 2와 FACE를 잇는 교량으로서의 DDS

이 섹션에서는 DDS를 심층적으로 분석하여, ROS 2와 FACE의 통합을 기술적으로 가능하게 하는 기반 기술임을 명확히 합니다. 추상적인 아키텍처 개념을 실제 인증된 상용 제품과 연결하여 설명합니다.

### 2.1  FACE 전송 서비스 세그먼트(TSS)의 핵심 기술로서의 DDS

TSS의 주요 목적은 이식 가능한 애플리케이션 구성요소(PCS/PSSS)로부터 통신의 세부 사항을 추상화하여, 이들 구성요소가 다른 플랫폼으로 수정 없이 이동할 수 있도록 하는 것입니다.10 FACE 참조 아키텍처는 TSS의 메시징 패러다임을 제공하는 기술로 DDS를 명시적으로 지목하고 있습니다.9 이는 DDS의 데이터 중심성(data-centricity), 풍부한 QoS 정책, 개방형 표준 기반의 상호운용성 등의 특징이 FACE의 목표를 직접적으로 지원하기 때문입니다.9 FACE 적합 TSS 구현체는 애플리케이션 구성요소에 표준화된 API를 제공하고, 내부적으로는 DDS를 사용하여 동일 프로세서 내에서든 네트워크를 통해서든 시스템 전반의 데이터 스트림을 발행하고 구독하는 방식으로 데이터 분배를 관리합니다.10

### 2.2  FACE 적합 상용 DDS 구현체 분석

FACE 생태계는 벤더들이 인증된 UoC를 제공하는 것에 의존합니다. 여러 DDS 벤더들은 이미 FACE 적합성 인증을 획득한 TSS 제품을 시장에 공급하고 있습니다. 예를 들어, RTI의 Connext TSS는 FACE 기술 표준 3.1판에 대한 적합성 인증을 획득한 성숙한 제품으로, 시장에서 활발히 판매되고 있습니다.23 또한 The Open Group의 공식 FACE 레지스트리(FACE Registry)를 살펴보면, 'ADLINK OpenSplice DDS TSS'가 전송 서비스 세그먼트용 인증 UoC로 등재되어 있음을 확인할 수 있습니다.24 CoreDX DDS 역시 FACE 참조 아키텍처에 명시된 모든 DDS 요구사항을 충족하며 FACE 준수에 매우 적합한 또 다른 상용 DDS 구현체입니다.9 이 레지스트리는 인증된 UoC에 대한 공식적인 정보 소스이며, 현재 DDS 기반 TSS 구성요소의 존재는 확인되지만 'ROS 2'나 핵심 ROS 2 패키지는 등재되어 있지 않다는 점이 중요합니다. 이는 ROS 2 자체가 FACE 구성요소로 인증된 것이 아님을 명확히 보여줍니다.

### 2.3  감항 인증으로의 경로: DDS와 DO-178C 인증 증거

FACE 적합성과 DO-178C 인증은 별개의 개념입니다. FACE 적합성은 이식성과 상호운용성을 보장하는 반면 7, DO-178C는 FAA(미 연방 항공청)나 EASA(유럽 항공 안전청)와 같은 항공 당국이 소프트웨어의 비행 안전성을 인증하는 데 사용하는 엄격한 표준입니다.3 FACE 접근법은 그 모듈성과 표준화된 인터페이스를 통해 안전 인증 과정을 

*용이하게* 하도록 설계되었습니다. 즉, 사전에 인증된 OSS나 TSS와 같은 구성요소를 사용함으로써 전체 시스템의 인증 노력을 줄일 수 있습니다.8

여기서 핵심적인 기술은 DO-178C 인증 증거(certification evidence)와 함께 제공되는 DDS 구현체입니다. RTI의 Connext Cert 제품은 DO-178C 인증이 필요한 시스템을 위해 특별히 설계되었습니다. RTI는 이미 미 육군 고객에게 인증된 RTOS(DDC-I Deos)와 하드웨어(NAI) 스택 상에서 실행되는 Connext Cert에 대한 DAL A(가장 높은 위험 등급) 인증 증거를 성공적으로 납품한 바 있습니다.25 이는 하드웨어부터 전송 계층에 이르기까지 완벽하게 인증 가능한 체인이 존재함을 의미합니다. 상용 기성품(COTS) DO-178C DAL A 인증 증거를 갖춘 DDS 구현체의 가용성은 게임 체인저와도 같습니다. 이는 잠재적인 ROS 2 통합의 통신 백본을 사전에 인증된, 비행 준비가 완료된 기반 위에 구축할 수 있음을 시사하기 때문입니다.25

이러한 분석을 통해 우리는 중요한 통합 패턴을 도출할 수 있습니다. ROS 2는 DDS 기반 RMW를 통해 통신하고, FACE 애플리케이션은 DDS 기반 TSS를 통해 통신합니다. 만약 ROS 2 RMW의 기반이 되는 DDS 구현체와 FACE TSS를 제공하는 DDS 구현체가 동일하다면, 이들은 사실상 같은 '데이터 버스' 상에서 통신하게 됩니다. 따라서 통합 지점은 ROS 2 *전체*가 아니라 통신 채널 그 자체입니다. ROS 2 노드는 동일한 FACE 적합, 안전 인증 DDS 전송 계층을 사용하는 네이티브 FACE 구성요소와 직접 데이터를 교환할 수 있습니다. 이 구조에서 TSS는 '신뢰할 수 없는' 로보틱스 세계와 '신뢰할 수 있는' 항공전자 세계 사이의 공식적이고 인증된 게이트웨이 역할을 합니다. 통합은 DDS에 의해 중재되는 데이터 수준에서 이루어집니다.

또한, 이는 오픈소스 인증에 대한 부담이 COTS 벤더에게 전가되는 효과를 낳습니다. Linux 커널, C++ 표준 라이브러리, 수많은 ROS 2 패키지로 구성된 거대하고 빠르게 변화하는 오픈소스 프로젝트 전체를 DO-178C에 따라 인증하는 것은 현실적으로 불가능합니다.12 항공 및 방위 산업은 이러한 복잡성을 관리하기 위해 인증 자료가 포함된 COTS 제품을 사용하는 잘 정립된 모델을 가지고 있습니다.8 RTI나 Wind River와 같은 벤더들은 막대한 투자를 통해 자사의 DDS 제품(Connext Cert)이나 RTOS(VxWorks)에 대한 DO-178C 인증 버전을 만들어냈습니다.25 따라서 통합 전략은 프로그램 통합업체가 오픈소스 ROS 2를 직접 인증하는 것을 요구하지 않습니다. 대신, 인증된 RTOS(OSS용)와 인증된 DDS(TSS용)를 COTS 시장에서 조달합니다. 이로써 기반 계층에 대한 인증 부담은 프로그램에서 COTS 벤더로 이전되어 리스크와 비용이 극적으로 감소합니다. 남은 인증 과제는 PCS에서 실행되는 특정 애플리케이션 코드에 대한 것으로, 이는 훨씬 더 관리하기 쉬운 문제입니다.

## 3.  FACE 시스템에 ROS 2를 통합하기 위한 전략적 프레임워크

이 섹션에서는 ROS 2 기반 기능을 FACE 적합 및 안전 인증 시스템에 통합하기 위한 구체적이고 단계적인 아키텍처 패턴을 제시합니다. 이는 본 보고서의 핵심적인 제안입니다.

### 3.1  아키텍처 패러다임: "TSS 중심" 통합 패턴

이 패턴은 전체 ROS 2 프레임워크를 FACE에 맞추려는 시도를 피합니다. 대신, ROS 2 애플리케이션을 이식 가능한 소프트웨어 구성요소로 취급하고, 인증된 TSS를 통신 백본으로 활용하여 FACE 아키텍처에 통합하는 방식을 취합니다. 핵심 원칙은 인증 가능한 기반(OSS, TSS)을 애플리케이션 로직(PCS)으로부터 격리하는 것입니다. ROS 2 API를 사용하여 개발된 애플리케이션 로직은 이 인증된 기반을 통해서만 통신하게 됩니다.

### 3.2  ROS 2 구조를 FACE 참조 아키텍처에 매핑하기

이것이 기술 전략의 핵심입니다. ROS 2 구성요소를 FACE 세그먼트 내의 논리적 등가물에 개념적으로 매핑하는 과정이 포함됩니다.

- **ROS 2 노드 -> FACE PCS UoC:** 센서 융합 알고리즘이나 임무 계획 기능과 같은 애플리케이션 로직을 포함하는 ROS 2 노드(또는 관련 노드 그룹)는 이식성 구성요소 세그먼트(PCS)에 위치하는 적합성 단위(UoC)로 패키징됩니다.
- **ROS 2 RMW 계층 -> FACE TSS API:** ROS 2 애플리케이션 코드는 표준 ROS 2 미들웨어 인터페이스(RMW)에 맞춰 작성됩니다. 시스템 통합 시, 호출을 인증된 FACE TSS로 직접 라우팅하는 특수 RMW 구현체가 사용됩니다. 이는 '어댑터' 또는 '브리지' 역할을 합니다.
- **기반 OS -> FACE OSS:** PCS UoC가 된 ROS 2 노드는 일반적인 데스크톱 Linux에서 실행되지 않습니다. 대신, VxWorks나 Deos와 같은 인증된 실시간 운영체제(RTOS) 위에서 실행되며, 이 RTOS가 FACE 운영체제 세그먼트(OSS)의 역할을 합니다.4 이러한 RTOS는 UoC가 요구하는 POSIX 또는 ARINC 653 API를 제공합니다.
- **ROS 2 토픽/서비스 -> TSS를 통한 DDS 토픽:** ROS 2 기반 UoC와 다른 FACE UoC 간의 통신은 인증된 TSS를 통해 이루어집니다. 예를 들어, ROS 2 토픽 `'/tf'`는 TSS에 의해 관리되고 보안이 적용되는 DDS 토픽이 됩니다.

### 3.3  기반 OSS로서의 인증된 실시간 운영체제(RTOS)의 역할

표준 Linux는 DO-178C 표준에 따라 인증받을 수 없으므로, 인증된 RTOS의 사용은 필수적입니다. Wind River의 VxWorks 653 및 Helix Platform은 이 분야의 시장 선두 주자로, FACE 안전 기본 프로파일(Safety Base Profile)에 대한 적합성 인증을 획득했으며 광범위한 DO-178C 인증 증거를 보유하고 있습니다.4 이들은 혼합된 위험 등급의 애플리케이션을 안전하게 실행하기 위해 필요한 파티션 환경(ARINC 653)을 제공합니다. DDC-I의 Deos 역시 주요 RTOS 중 하나로, DAL A 인증 증거를 갖춘 시공간 분할 RTOS이며 종종 RTI Connext Cert와 함께 사용됩니다.25

중요한 점은 이러한 RTOS 중 다수가 POSIX 호환 API 서브셋을 제공한다는 것입니다.4 이는 POSIX 환경을 기대하는 표준 ROS 2 클라이언트 라이브러리(rclcpp) 코드의 상당 부분을 최소한의 수정만으로 컴파일하고 실행할 수 있게 해주기 때문에 매우 중요합니다.

### 3.4  데이터 모델 상호운용성: ROS 2 메시지와 FACE 공유 데이터 모델 연결

여기에는 중요한 도전 과제가 존재합니다. ROS 2는 `.msg` 및 `.srv` 파일을 사용하여 언어별 코드를 생성하는 반면 28, FACE는 명확한 데이터 교환을 보장하기 위해 플랫폼 독립적인 공식 FACE 공유 데이터 모델(Shared Data Model, SDM)과 이식성 단위 제공 모델(Unit of Portability Supplied Model, USM)을 의무화합니다.5

이 두 시스템은 DDS 계층에서 조화될 수 있습니다. ROS 2 메시지는 DDS 전송을 위해 OMG의 인터페이스 정의 언어(Interface Description Language, IDL)로 변환되며 17, FACE 데이터 모델 역시 IDL로 변환 가능합니다. 통합 워크플로우는 다음과 같이 구성될 수 있습니다.

1. FACE 공유 데이터 모델에서 교환할 데이터를 정의합니다.
2. Ansys SCADE Architect와 같은 FACE 툴링을 사용하여 해당 IDL을 생성합니다.1
3. ROS 2 측에서는 *정확히 동일한* IDL 표현을 생성할 동등한 `.msg` 정의를 만듭니다.
4. 또는, PSSS UoC가 될 수 있는 '브리지' 구성요소를 생성할 수 있습니다. 이 구성요소의 유일한 역할은 ROS 네이티브 데이터 타입을 구독하고, 이를 FACE 네이티브 데이터 타입으로 변환하여 TSS에 다시 게시하는 것입니다. 이는 관심사의 명확한 분리를 제공합니다.

이러한 아키텍처적 접근 방식은 몇 가지 중요한 결과를 낳습니다. 첫째, ROS 2의 RMW가 교체 가능한 인터페이스라는 점을 활용하여 28, COTS 벤더들은 

`rmw_connext_cert_tss`와 같은 특수 RMW 구현체를 만들 수 있습니다. ROS 2 애플리케이션이 이 RMW에 대해 컴파일되면, `ros_publisher_create()`와 같은 호출은 표준 Fast DDS 라이브러리가 아닌 RTI Connext Cert TSS의 독점적이고 인증된 API로 라우팅됩니다. 이 RMW 계층은 '카멜레온' 또는 어댑터처럼 작동하여, ROS 2 애플리케이션 코드가 표준 ROS 2 시스템과 통신하고 있다고 믿게 만들면서 실제로는 인증된 FACE TSS와 통신하도록 합니다. 이는 애플리케이션 재작성 없이 원활한 통합을 가능하게 하는 핵심 소프트웨어 패턴이며, "ROS 2 for VxWorks"와 같은 솔루션이 바로 이 접근 방식을 취합니다.27

둘째, 이 아키텍처는 개발자가 자연스럽게 "다이어트 ROS(Diet ROS)" 접근법을 채택하도록 강제합니다. `rqt`, `rviz`와 같은 전체 ROS 2 생태계는 비행 소프트웨어가 아닌 개발 도구입니다.14 통합 패턴은 ROS 2 애플리케이션 로직을 PCS UoC 내에 배치하는데, 이 UoC는 

`rviz`를 위한 그래픽 디스플레이가 없는 인증된 RTOS 상에서 실행됩니다. 통신은 엄격하게 TSS에 의해 중재됩니다. 결과적으로, 전체 ROS 2 생태계를 기내에 탑재하는 것이 아니라, 특정 알고리즘 노드, 즉 '두뇌' 부분만 추출하여 FACE 아키텍처의 제한적이고 인증된 환경에서 실행하게 됩니다. 무거운 개발 및 시각화 도구는 지상에 남아 테스트 및 시뮬레이션 중에 동일한 DDS/TSS 백본을 통해 비행 시스템과 통신합니다. 이러한 개발 시점과 실행 시점 구성요소의 분리는 중요한 리스크 완화 전략입니다.

| ROS 2 개념                         | FACE 등가물                                        | 역할 및 통합 참고사항                                        |
| ---------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| **ROS 2 노드 (애플리케이션 로직)** | **PCS UoC**                                        | 핵심 알고리즘/비즈니스 로직으로, PCS 인터페이스 요구사항을 충족하도록 패키징됨. |
| **ROS 2 RMW API**                  | **TSS API (어댑터를 통해)**                        | 애플리케이션은 RMW API에 맞춰 작성되고, 특수 RMW 구현체가 인증된 TSS로 브리지 역할을 함. |
| **DDS 미들웨어 (예: Fast DDS)**    | **인증된 DDS (예: Connext Cert)가 제공하는 TSS**   | 기본 오픈소스 DDS가 COTS, 인증 가능한 등가물로 대체됨.       |
| **Linux/Windows OS**               | **인증된 RTOS (예: VxWorks, Deos)가 제공하는 OSS** | 범용 OS가 POSIX/ARINC 653 API를 제공하는 안전 인증 RTOS로 대체됨. |
| **ROS 2 `.msg` / `.srv`**          | **FACE USM/SDM (IDL로 정의)**                      | 데이터 모델은 IDL을 공통 언어로 사용하여 조화되어야 하며, 변환을 위해 PSSS 브리지가 필요할 수 있음. |
| **SROS 2 보안 파일**               | **TSS를 위한 DDS-Security 구성**                   | 보안 정책은 동일한 DDS-Security 표준을 사용하는 인증된 TSS 계층에서 구현됨.11 |

## 4.  지원 COTS 솔루션 및 사례 연구 분석

이 섹션에서는 제안된 이론적 프레임워크를 실제 사용 가능한 제품 및 관련 이니셔티브에 기반하여 설명함으로써, 이러한 통합이 단지 가능할 뿐만 아니라 업계 선두 주자들에 의해 적극적으로 추진되고 있음을 보여줍니다.

### 4.1  RTI Connext Suite: FACE 적합성과 DO-178C 인증을 위한 이중 경로

RTI는 FACE 적합성과 DO-178C 안전 인증이라는 두 가지 중요한 경로를 모두 지원하는 포괄적인 솔루션을 제공합니다. **RTI Connext TSS**는 FACE 3.1 기술 표준에 대한 적합성 인증을 받은 TSS UoC로, FACE 준수를 위한 '정문' 역할을 합니다.23 한편, 

**RTI Connext Cert**는 DO-178C DAL A 인증 증거와 함께 제공되는 DDS 미들웨어 버전으로, 감항 인증으로 가는 경로를 제공합니다.25 RTI의 툴링은 ROS 2를 직접 지원하여 ROS 2를 사용하는 DDS 기반 시스템의 설계와 발전을 단순화합니다.26 이는 앞서 설명한 '카멜레온' RMW 계층과 같은 통합 메커니즘이 존재함을 시사합니다. RTI는 Connext Cert (TSS)를 DDC-I Deos (OSS)와 NAI 하드웨어 위에서 실행하는 완전한 인증 가능 스택을 시연함으로써 25, ROS 2 PCS UoC를 위한 준비된 기반을 제공합니다.

### 4.2  Wind River Systems: 안전 필수 환경에서의 "ROS 2 for VxWorks"

Wind River는 FACE 적합성 및 광범위한 DO-178C 증거를 갖춘 선도적인 RTOS인 **VxWorks**를 제공하며, 이는 인증 가능한 OSS 역할을 합니다.4 특히 주목할 점은 Wind River가 **"ROS 2 for VxWorks"**라는 특정 제품/솔루션 개요를 제공한다는 것입니다.27 이는 개발자들이 VxWorks가 제공하는 결정론적이고 안전 필수적인 환경에서 실행되어야 하는 애플리케이션을 위해 ROS 2 생태계를 활용할 수 있도록 하는 것을 직접적인 목표로 합니다. 이 솔루션은 ROS 2 애플리케이션을 VxWorks 상에서 컴파일하고 실행하는 데 필요한 구성요소, 즉 특수 RMW와 빌드 도구들을 상업적으로 패키징한 것으로, 섹션 3에서 설명한 아키텍처 패턴을 실제로 구현한 것입니다.

### 4.3  Space ROS로부터의 교훈: 미션 크리티컬 시스템을 위한 ROS 2 강화

**Space ROS**는 NASA 주도의 이니셔티브로, 신뢰성과 결정론 측면에서 항공전자 시스템과 유사한 요구사항을 갖는 미션 크리티컬 우주 애플리케이션에 적합한 ROS 2의 분기 버전(fork)을 만드는 것을 목표로 합니다.29 이 프로젝트는 중요한 강화 활동을 수행하고 있으며, 이는 FACE 통합에 직접적인 참고가 됩니다.

- **메모리 안전성:** 비결정성의 주요 원인인 표준 동적 메모리 할당을 C++17 다형성 할당자(polymorphic allocator)를 사용한 사전 할당 메모리 풀로 대체합니다.29
- **결정론적 성능:** NASA의 비행 소프트웨어 표준을 충족시키기 위한 핵심 목표입니다.29
- **정적 분석 및 V&V:** 정적 분석 도구를 빌드 시스템에 통합하고 공식적인 검증 및 확인(V&V) 계획을 수립하여 인증 노력을 지원합니다.29

Space ROS 프로젝트는 완벽한 사례 연구 역할을 합니다. 이는 오픈소스 ROS 2 코드베이스를 가져와 인증 환경에 적합하도록 만드는 데 필요한 구체적인 엔지니어링 작업을 보여줍니다. 그들이 사용하는 메모리 관리와 같은 기술은 FACE 시스템을 위한 PCS UoC로 준비되는 ROS 2 애플리케이션에 직접 적용될 수 있습니다.

이러한 여러 독립적인 공급업체(RTI, Wind River)와 기관(NASA)이 모두 동일한 문제, 즉 인증된 환경에서 ROS 2를 실행하는 문제를 해결하기 위해 노력하고 있다는 사실은 이것이 단순한 이론적 연습이 아니라 실제 시장으로 부상하고 있음을 보여줍니다. 생태계는 COTS 벤더가 제공하는 '인증된 코어'(OSS + TSS) 위에, ROS 2 API와 Space ROS와 같은 프로젝트의 기술을 사용하여 개발된 '강화된 애플리케이션'을 배포하는 공통 아키텍처 패턴으로 수렴하고 있습니다. 이러한 다중 벤더 생태계는 주계약업체의 리스크를 크게 줄여줍니다. 그들은 처음부터 솔루션을 발명하는 것이 아니라, 상업적으로 이용 가능하고 사전에 인증되었으며 현장에서 입증된 구성요소들로 솔루션을 조립하게 됩니다.

| 제품/솔루션                                 | 벤더       | 구성요소 유형      | 주요 특징 및 통합 관련성                                     |
| ------------------------------------------- | ---------- | ------------------ | ------------------------------------------------------------ |
| **RTI Connext TSS / Connext Cert**          | RTI        | TSS / DDS 미들웨어 | FACE 3.1 적합성, DO-178C DAL A 증거, 보안 플러그인, ROS 2 통합 툴링 제공. |
| **Wind River VxWorks 653 / Helix Platform** | Wind River | OSS / RTOS         | FACE 안전 프로파일 적합성, DO-178C 증거, ARINC 653 파티셔닝, POSIX API 제공. |
| **Wind River "ROS 2 for VxWorks"**          | Wind River | 통합 계층          | ROS 2 애플리케이션을 VxWorks에서 실행하기 위한 브리지/RMW 및 상업적 지원 제공. |
| **DDC-I Deos**                              | DDC-I      | OSS / RTOS         | DO-178C DAL A 증거, 시공간 분할 기능 제공. 종종 RTI Connext와 함께 사용됨. |
| **Ansys SCADE Suite/Architect**             | Ansys      | 모델링/개발 도구   | FACE 데이터 모델링 지원, DO-178C를 위한 검증된 코드 생성, TSS 어댑터 생성 기능 제공. |

## 5.  구현 과제, 리스크 및 완화 전략

이 섹션에서는 제안된 아키텍처 패턴과 COTS 솔루션이 있음에도 불구하고 여전히 남아있는 중요한 엔지니어링 과제들을 균형 잡힌 시각으로 조명합니다.

### 5.1  인증 부담: 오픈소스 소프트웨어 스택 관리

OSS와 TSS가 인증되었다 하더라도, PCS UoC로서의 ROS 2 애플리케이션과 `rclcpp`와 같은 그 의존성들은 여전히 비행을 위해 인증받아야 합니다. 이는 거대하고 복잡한 오픈소스 코드베이스를 다루는 것을 포함합니다. 이를 완화하기 위해서는 최종 비행 바이너리에 포함되는 ROS 2 코드를 공격적으로 최소화하는 **코드 범위 지정(scoping)**이 필요합니다. 정적 분석을 사용하여 모든 의존성을 추적하고 불필요한 것을 제거해야 합니다. 또한, **Space ROS 활용**을 통해 이 목적을 위해 특별히 강화된 ROS 2의 패치 버전과 도구를 사용하는 것이 유리합니다.29 LDRA나 Ansys와 같은 업체에서 제공하는 **COTS 툴링**을 사용하여 FACE와 DO-178C 워크플로우를 모두 지원하는 인증 산출물을 관리하는 것도 중요합니다.1

### 5.2  성능 및 결정론: 실시간 보장 확보

ROS 2의 기본 실행기(executor)와 스레딩 모델은 하드 리얼타임 결정론을 위해 설계되지 않았으며, 기아(starvation) 현상이 없는 것을 보장하지 못하는 등의 문제를 보일 수 있습니다.20 지연 시간은 QoS 설정, 네트워크 부하, 노드 수에 따라 예측 불가능할 수 있습니다.31 이에 대한 완화책으로, 첫째, 

**인증된 RTOS 사용**이 필수적입니다. VxWorks나 Deos와 같은 RTOS의 스케줄링 및 파티셔닝 기능은 실시간 동작을 강제하는 데 가장 중요합니다.4 둘째, 

**신중한 실행기/콜백 설계**가 필요합니다. 중요한 루프에는 기본 다중 스레드 실행기를 피하고, 단일 스레드 실행기나 맞춤형 콜백 그룹 구성을 사용하여 실행 흐름과 우선순위를 제어해야 합니다.21 셋째, 

**엄격한 QoS 구성**을 통해 신뢰성, 마감 시간, 내구성 등 DDS QoS 정책을 세심하게 정의하고 테스트하여 예측 가능한 통신 동작을 보장해야 합니다.14

### 5.3  시스템 보안: SROS 2와 항공전자 보안 프로토콜의 조화

SROS 2는 DDS-Security를 기반으로 한 강력한 보안 모델(PKI, 접근 제어 목록, 암호화)을 제공하지만 11, 항공전자 시스템은 종종 그들만의 고유한 보안 요구사항과 프로토콜을 가지고 있습니다. 이 문제를 해결하기 위해, 보안 정책은 오픈소스 ROS 2 도구가 아닌 

**인증된 TSS 계층에서 구현 및 시행**되어야 합니다. 예를 들어, RTI Connext Cert의 보안 플러그인을 활용할 수 있습니다. 또한, DDS 보안 산출물에 서명하는 데 사용되는 인증 기관(CA)은 시스템 전체의 보안 및 키 관리 인프라와 통합되어야 합니다. SROS 2 도구는 개발 및 프로토타이핑에 사용될 수 있지만, 최종 키와 권한 파일은 프로그램의 공식 보안 계획에 따라 생성 및 관리되어야 합니다.11

아키텍처 패턴과 COTS 제품이 명확한 길을 제시하지만, 인증을 획득하기 위한 상세한 엔지니어링 작업은 여전히 막대합니다. RTI Connext를 사용하여 VxWorks에서 ROS 2 'hello world' 노드를 실행하는 것은 해결 가능한 문제이지만, 이 특정 노드와 그 모든 C++ 의존성이 마이크로초 단위의 실행 마감 시간을 항상 준수하고 메모리 오류가 절대 발생하지 않으며 100% MCDC 테스트 커버리지를 갖추었음을 인증 기관에 증명하는 것은 완전히 다른 차원의 노력입니다. ROS 2의 실시간 성능의 미묘함 21, 보안 구성의 복잡성 11, FACE 적합성 프로세스의 엄격함 7은 이러한 노력이 얼마나 중요한지를 보여줍니다. 따라서 프로그램 관리자는 COTS 솔루션이 기반 문제를 해결해주더라도, 최종 PCS UoC를 인증하는 책임은 통합업체에 있으며, 대부분의 상세하고 고된 엔지니어링 작업이 여기에 집중될 것이라는 점을 명심해야 합니다.

## 6.  결론 및 전략적 권장사항

이 마지막 섹션에서는 전체 분석을 종합하여 명확한 결론을 내리고, 대상 독자를 위한 실행 가능한 조언을 제공합니다.

### 6.1  최종 결론: FACE 통합을 위한 ROS 2의 실현 가능성 및 성숙도

- **ROS 2는 FACE에 포함되는가?** 직접적인 대답은 **아니요**입니다. ROS 2 프레임워크 자체는 FACE 적합성 인증을 받은 제품이 아닙니다.
- **ROS 2를 FACE 시스템에서 사용할 수 있는가?** 확정적인 대답은 **예, 아키텍처적으로 실현 가능하며 점점 더 현실화되고 있습니다**. 통합은 'ROS 2를 포함'시키는 방식이 아니라, ROS 2 기반 애플리케이션 구성요소(PCS UoC로서)를 COTS 제품으로 구축된 FACE 적합 및 안전 인증 기반(OSS 및 TSS) 위에서 실행하는 방식으로 달성됩니다.
- **성숙도 수준:** 이 접근법은 주요 항공 및 방위 산업 공급업체들의 인증된 COTS 구성요소 가용성과 Space ROS와 같은 적극적인 강화 이니셔티브에 힘입어 이론적 단계에서 실용적 단계로 나아가고 있습니다. 새로운 시스템 설계에 고려할 만큼 충분히 성숙했지만, 상당한 시스템 통합 및 안전 엔지니어링 전문 지식이 요구됩니다.

### 6.2  시스템 아키텍트 및 프로그램 관리자를 위한 실행 가능한 권장사항

1. **TSS 중심 통합 패턴 채택:** 전체 ROS 2 스택을 인증하려는 시도를 피하십시오. 인증된 OSS/TSS 스택을 조달하고 ROS 2 애플리케이션을 PCS UoC로 통합하는 데 집중해야 합니다.
2. **COTS 벤더와 조기 협력:** RTI, Wind River와 같은 벤더들의 전문 지식을 활용하십시오. 이들은 이미 많은 기반 통합 및 인증 과제를 해결했습니다.
3. **애플리케이션 수준 인증 예산 확보:** 통합업체의 주요 비용과 리스크는 자체 애플리케이션 코드의 인증에서 발생할 것입니다. 이 노력을 과소평가해서는 안 됩니다.
4. **"다이어트 ROS" 정책 수립:** 비행 소프트웨어에 포함되는 ROS 2 의존성을 최소화하는 엄격한 정책을 수립하고, 개발 도구를 비행 환경으로부터 격리해야 합니다.
5. **견고한 데이터 모델링 전략 투자:** IDL을 공통 기반으로 사용하여 ROS 2 메시지 타입과 FACE 공유 데이터 모델 간의 변환을 계획해야 합니다.

### 6.3  미래 전망: 로보틱스와 항공전자 표준의 융합

ROS 2의 FACE 시스템으로의 성공적인 통합은 빠르게 변화하는 로보틱스 산업과 안전이 최우선인 항공 및 방위 산업이 융합되는 중요한 이정표가 될 것입니다. 이는 AI/ML 기반 인식, 협력적 임무 계획과 같은 첨단 자율 능력을 항공 플랫폼에 더 신속하게 배포하는 것을 가속화할 것입니다. 앞으로 COTS 생태계는 더욱 성숙하여 ROS 2 개발 환경과 인증된 배포 플랫폼 간의 더 긴밀하고 원활한 통합을 제공할 것으로 기대됩니다. 항공 및 방위 부문의 상호 고객 요구에 따라 FACE와 ROS 2 커뮤니티의 작업은 시간이 지남에 따라 더욱 긴밀하게 연계될 가능성이 높습니다.

#### **참고 자료**

1. FACE Standard for Avionics Software | Ansys, accessed July 17, 2025, https://www.ansys.com/industries/face-standard-aviation-software
2. The FACE Approach | www.opengroup.org, accessed July 17, 2025, https://www.opengroup.org/face/approach
3. The Future Airborne Capability Environment (FACE™) - LDRA, accessed July 17, 2025, https://ldra.com/face/
4. What Is FACE™? | Wind River, accessed July 17, 2025, https://www.windriver.com/solutions/learning/face
5. Future Airborne Capability Environment - Wikipedia, accessed July 17, 2025, https://en.wikipedia.org/wiki/Future_Airborne_Capability_Environment
6. Technical Standard Edition 2.1 | www.opengroup.org, accessed July 17, 2025, https://www.opengroup.org/face/tech-standard-2.1
7. FACE™ Consortium Conformance FAQs | www.opengroup.org, accessed July 17, 2025, https://www.opengroup.org/face/conformance-FAQs
8. Leveraging Commercial Avionics Standards: Understanding the FACE Reference Architecture - RTI, accessed July 17, 2025, https://www.rti.com/blog/understanding-face-reference-architecture
9. FACE and DDS | Twin Oaks Computing, Inc, accessed July 17, 2025, https://www.twinoakscomputing.com/coredx/future_airborne_capability_environment
10. Applying Transport Services Segment (TSS) Concepts - OMG Wiki, accessed July 17, 2025, https://www.omgwiki.org/face/lib/exe/fetch.php?media=wiki:interop-wg:applying_tss_concepts_rev_b_no_comments.pdf
11. ROS 2 DDS-Security integration - ROS2 Design, accessed July 17, 2025, https://design.ros2.org/articles/ros2_dds_security.html
12. ROS 2 Key Challenges and Advances: A Survey of ROS 2 Research, Libraries, and Applications - ResearchGate, accessed July 17, 2025, https://www.researchgate.net/publication/384942758_ROS_2_Key_Challenges_and_Advances_A_Survey_of_ROS_2_Research_Libraries_and_Applications
13. [2211.07752] Robot Operating System 2: Design, Architecture, and Uses In The Wild - arXiv, accessed July 17, 2025, https://arxiv.org/abs/2211.07752
14. ROS 2: What is DDS | Data Distribution Service (DDS) Community RTI Connext Users, accessed July 17, 2025, https://community.rti.com/page/ros-2-what-dds
15. Performance Evaluation of ROS2-DDS middleware implementations facilitating Cooperative Driving in Autonomous Vehicle. - arXiv, accessed July 17, 2025, https://arxiv.org/html/2412.07485v1
16. (PDF) Towards a ROS2-based software architecture for service robots - ResearchGate, accessed July 17, 2025, https://www.researchgate.net/publication/374352169_Towards_a_ROS2-based_software_architecture_for_service_robots
17. ROS on DDS, accessed July 17, 2025, https://design.ros2.org/articles/ros_on_dds.html
18. DDS implementations — ROS 2 Documentation: Iron documentation, accessed July 17, 2025, https://docs.ros.org/en/iron/Installation/DDS-Implementations.html
19. ROS 2 in a Nutshell: A Survey[v3] - Preprints.org, accessed July 17, 2025, https://www.preprints.org/manuscript/202410.1204/v3
20. ROS2 in UR whitepaper - Documentation for Universal Robots, accessed July 17, 2025, https://docs.universal-robots.com/PolyScopeX_SDK_Documentation/build/SDK-v0.13/_downloads/305de110674c8345261f1ca772bb3c6f/ROS2inURwhitepaper.html
21. Timing Analysis and Priority-driven Enhancements of ROS 2 Multi-threaded Executors - arXiv, accessed July 17, 2025, https://arxiv.org/pdf/2408.08440
22. ROS 2 Security — ROS 2 Documentation: Rolling documentation, accessed July 17, 2025, https://docs.ros.org/en/rolling/Concepts/Intermediate/About-Security.html
23. RTI Connext TSS: Certified Conformant to FACE Technical Standard, Edition 3.1, accessed July 17, 2025, https://www.rti.com/blog/rti-connext-tss-certified-conformant
24. FACE UOC Registry, accessed July 17, 2025, https://facesoftware.org/registry
25. RTI Connext Cert: Delivering RTCA DO-178C Certification Evidence for Avionics Programs, accessed July 17, 2025, https://www.rti.com/blog/rtca-do178c-certification
26. enabling technology leadershipaward real-time innovations - Frost & Sullivan, accessed July 17, 2025, https://www.frost.com/wp-content/uploads/2023/03/Real-Time-Innovations-RTI-Final-Award-Write-up.pdf
27. VxWorks The Leading RTOS for the Intelligent Edge - Monthly archive | Wind River, accessed July 17, 2025, https://www.windriver.com/archive/202109?page=2
28. ROS 2 Design, accessed July 17, 2025, https://design.ros2.org/
29. Space ROS: An Open-Source Framework for Space Robotics and ..., accessed July 17, 2025, https://ntrs.nasa.gov/api/citations/20220017761/downloads/Space_ROS_SciTech.pdf
30. A first look at ROS 2 applications written in asynchronous Rust - arXiv, accessed July 17, 2025, https://arxiv.org/html/2505.21323v1
31. Latency Overhead of ROS2 for Modular Time-Critical Systems - arXiv, accessed July 17, 2025, http://www.arxiv.org/pdf/2101.02074v1
32. Latency Analysis of ROS2 Multi-Node Systems - arXiv, accessed July 17, 2025, https://arxiv.org/pdf/2101.02074
33. Timing Analysis and Priority-driven Enhancements of ROS 2 Multi-threaded Executors, accessed July 17, 2025, https://arxiv.org/html/2408.08440v1
34. FACE® Consortium Conformance Publications & Tools | www.opengroup.org, accessed July 17, 2025, [https://www.opengroup.org](https://www.opengroup.org/face/conformance-publications-and-tools)




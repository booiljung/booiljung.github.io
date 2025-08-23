# eProsima Fast DDS 실시간 분산 시스템을 위한 고성능 미들웨어


실시간 분산 시스템의 통신을 이해하기 위해서는 먼저 그 기반이 되는 표준 기술을 명확히 파악해야 합니다. eProsima Fast DDS는 Object Management Group (OMG)에서 제정한 두 가지 핵심 표준, 즉 데이터 분배 서비스(Data Distribution Service, DDS)와 실시간 발행-구독(Real-Time Publish-Subscribe, RTPS) 프로토콜 위에 구축되었습니다. 이 두 표준은 Fast DDS의 설계 철학, 기능, 그리고 한계를 이해하는 데 필수적인 전제 조건입니다.


DDS는 OMG가 정의한 개방형 국제 미들웨어 표준으로, 실시간 및 임베디드 시스템을 위한 데이터 중심의 발행-구독(Publish-Subscribe) 통신을 목표로 합니다.1 DDS는 단순한 메시징 라이브러리가 아니라, 분산 시스템의 구성요소들을 통합하기 위한 포괄적인 아키텍처 패턴을 제시합니다. 낮은 지연 시간의 데이터 연결성, 극도의 신뢰성, 그리고 비즈니스 및 미션 크리티컬 사물 인터넷(IoT) 애플리케이션에 요구되는 확장 가능한 아키텍처를 제공하는 것이 핵심입니다.2


DDS의 가장 핵심적인 패러다임은 데이터 중심(Data-Centric) 접근 방식입니다. 이는 전통적인 메시지 중심(Message-Centric) 미들웨어와의 근본적인 차이점입니다. 메시지 중심 미들웨어에서 애플리케이션은 내용물을 알 수 없는 바이트 덩어리(opaque blob)를 주고받으며, 수신 측 애플리케이션이 이 데이터를 해석할 책임을 집니다. 반면, DDS는 미들웨어 자체가 데이터의 구조와 의미를 인지합니다.2

이러한 데이터 중심성은 '가상 글로벌 데이터 공간(Global Data Space)'이라는 개념을 통해 구현됩니다.2 애플리케이션 개발자에게 이 공간은 마치 로컬 메모리처럼 보입니다. 데이터 생산자(Publisher)가 이 가상 공간에 데이터를 '쓰면(write)', DDS 미들웨어는 이 데이터를 관심 있는 모든 데이터 소비자(Subscriber)에게 투명하게 전파합니다. 이 모델은 시스템의 구성 요소들을 공간적(어디에 있는지), 시간적(언제 동작하는지), 그리고 플랫폼적(어떤 OS나 하드웨어를 사용하는지)으로 완벽하게 분리(decoupling)시킵니다.7

이러한 접근 방식은 시스템 설계를 근본적으로 변화시킵니다. 개발자는 더 이상 데이터 전송의 저수준 메커니즘에 대해 고민할 필요가 없습니다. 대신, 공유될 데이터의 모델을 정의하는 데 집중할 수 있습니다. 미들웨어가 데이터의 구조를 이해하기 때문에, 단순히 토픽(Topic) 기반의 필터링을 넘어 데이터의 내용(content)에 기반한 필터링, 데이터의 생명주기 관리, 데이터 일관성 보장과 같은 고수준의 기능들을 미들웨어 차원에서 직접 처리할 수 있습니다. 결과적으로 애플리케이션 코드는 더욱 단순해지고, 시스템의 복잡성은 미들웨어 계층으로 위임됩니다.2 따라서 DDS를 채택하는 것은 단순히 통신 라이브러리를 교체하는 행위를 넘어, 시스템 아키텍처를 데이터 중심으로 재구성하는 패러다임의 전환을 의미합니다.


DDS의 강력함과 유연성의 핵심은 광범위한 서비스 품질(Quality of Service, QoS) 정책에 있습니다. DDS는 22개 이상의 구성 가능한 QoS 정책을 통해 통신의 비기능적 속성을 매우 정밀하게 제어할 수 있도록 합니다.2

주요 QoS 정책은 다음과 같습니다:

- **Reliability (신뢰성):** `BEST_EFFORT` (최선 노력 전송)와 `RELIABLE` (신뢰성 보장 전송) 중에서 선택할 수 있습니다. `RELIABLE`은 데이터가 유실 없이 반드시 전달되어야 하는 중요한 데이터 스트림에 사용됩니다.
- **Durability (영속성):** 데이터 소비자가 네트워크에 늦게 참여(late-joining)하더라도 이전에 발행된 데이터를 수신할 수 있는지 여부를 결정합니다. 예를 들어, 시스템의 최신 설정값과 같은 데이터는 이 정책을 통해 새로운 노드가 시작될 때 즉시 전달받을 수 있습니다.
- **History (이력):** 데이터 생산자 또는 소비자가 얼마나 많은 데이터 샘플을 저장할지를 정의합니다. `KEEP_LAST`는 지정된 수의 최신 샘플만 유지하고, `KEEP_ALL`은 리소스가 허용하는 한 모든 샘플을 유지합니다.
- **Liveliness (활성도):** 시스템 내 참여자(participant)들의 활성 상태를 모니터링하는 메커니즘을 제공하여, 특정 노드에 문제가 발생했음을 감지할 수 있게 합니다.

개발자는 이러한 QoS 정책들을 조합하여 동일한 애플리케이션 내에서도 각기 다른 데이터 스트림의 요구사항(예: 고속의 비디오 스트림은 `BEST_EFFORT`, 중요한 제어 명령은 `RELIABLE`)에 맞춰 성능, 신뢰성, 리소스 사용량 간의 균형을 정밀하게 조절할 수 있습니다.8


DDS의 또 다른 핵심 기능은 동적 발견입니다. 시스템에 참여하는 노드들은 중앙 집중식 브로커(broker)나 IP 주소 목록이 담긴 설정 파일 없이도 런타임에 서로를 자동으로 발견합니다.2 새로운 애플리케이션이 네트워크에 추가되면, 기존의 다른 구성원들은 이를 자동으로 인지하고 필요한 데이터 통신을 시작할 수 있습니다. 이러한 '플러그 앤 플레이(plug-and-play)' 기능은 확장 가능하고 복원력 있는 분산 시스템을 구축하는 데 매우 중요합니다.


RTPS는 서로 다른 벤더가 구현한 DDS 애플리케이션 간의 상호운용성을 보장하기 위한 OMG 표준 와이어 프로토콜(wire protocol)입니다.1 와이어 프로토콜이란 네트워크 상에서 데이터를 주고받는 형식을 정의한 규약입니다. RTPS는 원래 Real-Time Innovations (RTI)사에 의해 개발되었으나, 그 우수성을 인정받아 OMG 표준으로 채택되었습니다.14


RTPS 프로토콜의 구조는 DDS의 개념과 직접적으로 매핑됩니다. 예를 들어, RTPS의 `Participant`, `Writer`, `Reader`는 각각 DDS의 `DomainParticipant`, `DataWriter`, `DataReader`에 해당합니다.16 이러한 일대일 대응 관계 덕분에 DDS 애플리케이션의 추상적인 데이터 모델이 네트워크 상의 구체적인 메시지 교환으로 원활하게 변환될 수 있습니다.


RTPS는 특정 전송 기술에 종속되지 않도록 설계되었습니다. 기본적으로 UDP와 같은 비신뢰성, 비연결성 전송 프로토콜 위에서 동작하도록 설계되었지만, TCP와 같은 연결 지향 프로토콜도 지원합니다.11 이러한 유연성 덕분에 UDP 멀티캐스트(multicast)를 활용하여 다수의 구독자에게 효율적으로 데이터를 전송할 수 있으며, 동시에 멀티캐스트가 제한되는 엔터프라이즈 네트워크 환경에서는 TCP를 대안으로 사용할 수 있습니다.


RTPS 프로토콜의 동작은 엔드포인트 간에 교환되는 몇 가지 핵심 메시지 타입에 의해 정의됩니다.

- **DATA:** 직렬화된 애플리케이션 데이터를 담고 있는 메시지입니다.16
- **HEARTBEAT:** `Writer`가 `Reader`에게 자신이 현재 전송 가능한 데이터의 시퀀스 번호 범위를 알리기 위해 주기적으로 보내는 메시지입니다.14
- **ACKNACK:** `Reader`가 `Writer`에게 어떤 데이터를 수신했는지(ACK), 그리고 어떤 데이터가 누락되었는지(NACK)를 알리는 메시지입니다. 신뢰성 있는(Reliable) 통신에서 데이터 재전송을 요청하는 데 사용됩니다.14


DDSI-RTPS 표준은 지속적으로 발전하고 있으며, 최신 버전은 2.5(2022년 4월 기준)입니다.12 이 표준은 TSN(Time-Sensitive Networking)이나 TCP와 같은 새로운 기술에 대한 매핑을 정의하는 등 새로운 요구사항을 반영하며 계속 진화하고 있습니다.12

RTPS 프로토콜은 DDS 생태계에 있어 양날의 검과 같은 역할을 합니다. 한편으로는 표준 준수를 통해 서로 다른 벤더의 제품 간 상호운용성이라는 강력한 이점을 제공합니다.9 이는 특정 벤더에 대한 종속성을 줄여주는 중요한 요소입니다. 하지만 다른 한편으로는, 벤더들이 성능 향상을 위해 추가하는 비표준 기능들이 이러한 상호운용성을 해칠 수 있다는 제약이 따릅니다. 대표적인 예가 공유 메모리 전송(Shared Memory Transport, SHM)입니다. eProsima, RTI 등 여러 벤더들은 동일 호스트 내 프로세스 간 통신 성능을 극대화하기 위해 각자 독자적인 SHM 구현을 제공합니다.21 만약 서로 다른 벤더의 애플리케이션이 동일한 머신에서 실행될 경우, 이들은 서로 호환되지 않는 SHM 전송을 시도하다가 통신에 실패할 수 있습니다.21 이는 RTPS가 보장하는 '플러그 앤 플레이'가 모든 상황에서 완벽하게 동작하는 것은 아니며, 고성능의 다중 벤더 시스템을 구축하기 위해서는 표준화된 전송 방식(UDP/TCP)만을 사용하도록 신중하게 구성해야 함을 시사합니다.


이제 DDS와 RTPS라는 기반 기술에 대한 이해를 바탕으로, 본 보고서의 핵심 주제인 eProsima Fast DDS에 대해 심층적으로 분석하겠습니다. Fast DDS는 이러한 표준들을 어떻게 구현했으며, 어떤 특징과 구조를 가지고 있는지, 그리고 어떤 비즈니스 모델을 통해 시장에서 경쟁하고 있는지 살펴보겠습니다.


eProsima는 스페인에 본사를 둔 소프트웨어 회사로, 분산 시스템을 위한 미들웨어 개발을 전문으로 하며 특히 로보틱스, 인공지능(AI), 사물 인터넷(IoT) 분야에 강점을 보이고 있습니다.22

Fast DDS(이전 명칭: Fast RTPS)는 eProsima의 주력 제품으로, OMG DDS 버전 1.4 및 RTPS 버전 2.2 표준을 C++로 구현한 고성능 미들웨어입니다.1 월 5만 회 이상의 다운로드를 기록하며 전 세계 수십만 명의 개발자가 사용하는 등, 시장에서 가장 널리 채택된 DDS 구현 중 하나로 자리매김했습니다.23


Fast DDS의 폭넓은 채택 배경에는 전략적인 라이선스 정책이 있습니다. Fast DDS는 상용 제품에도 자유롭게 통합할 수 있는 매우 허용적인 아파치 2.0 라이선스(Apache 2.0 License) 하에 배포되는 무료 오픈소스 소프트웨어입니다.24 GPL과 같은 카피레프트(copyleft) 라이선스는 이를 사용하는 애플리케이션의 소스 코드 공개를 요구할 수 있어 상용 기업들이 채택을 꺼리는 경우가 많습니다. 반면 아파치 2.0 라이선스는 이러한 부담이 없어 상용 제품 개발사들이 자신들의 독점적인 소스 코드를 공개하지 않고도 Fast DDS를 자유롭게 사용하고 수정하며 통합할 수 있게 합니다.

이러한 낮은 진입 장벽은 Fast DDS가 수많은 상용 기업들이 참여하는 ROS 2 생태계의 기본 미들웨어로 채택되는 결정적인 계기가 되었습니다. 이는 선순환 구조를 만들어냈습니다. ROS 2의 채택이 Fast DDS의 채택을 견인하고, 이는 다시 Fast DDS를 사실상의 오픈소스 DDS 표준으로 자리매김하게 하여 더 많은 사용자들과 기여자들을 유치하는 효과를 낳았습니다.

eProsima의 비즈니스 모델은 이 강력한 오픈소스 기반 위에서 상용 지원, 맞춤형 기능 개발, 아키텍처 컨설팅, 그리고 `Safe DDS`와 같은 안전 인증 버전 제공을 통해 수익을 창출하는 구조입니다.22


Fast DDS는 예측 가능성, 확장성, 유연성, 그리고 효율적인 리소스 처리를 목표로 설계되었습니다. 그 아키텍처의 핵심 요소는 다음과 같습니다.


Fast DDS는 개발자의 다양한 요구를 충족시키기 위해 두 개의 متمایز API 계층을 제공합니다.11

- **DDS API:** OMG DDS 표준을 준수하는 고수준 API로, 사용 편의성에 초점을 맞추고 있습니다. 대부분의 애플리케이션 개발자들은 이 API를 통해 Fast DDS를 사용하게 됩니다.
- **RTPS API:** 개발자가 RTPS 프로토콜의 내부 동작에 직접 접근할 수 있도록 하는 저수준 API입니다. 이를 통해 고급 사용 사례에서 매우 정밀한 제어와 최적화가 가능합니다.


Fast DDS의 아키텍처는 모듈성을 극대화하기 위해 플러그형 전송 계층 구조를 채택하고 있습니다.17

- **지원 전송 방식:** 기본적으로 UDPv4/v6, TCPv4/v6, 그리고 공유 메모리(Shared Memory, SHM) 전송을 지원합니다.17
- **제로카피(Zero-Copy):** 특히 대용량 데이터 전송 시 성능을 극대화하기 위해, 공유 메모리를 통한 제로카피 데이터 공유를 지원합니다. 이는 프로세스 간 메모리 복사를 회피함으로써 지연 시간을 획기적으로 줄이고 처리량을 높이는 기술입니다.29


Fast DDS 라이브러리는 핵심적인 DDS/RTPS 구현부 외에, 데이터 직렬화를 위한 Fast CDR 라이브러리와 같은 의존성과 `Fast DDS-Gen`과 같은 개발 도구들로 구성됩니다.28


실시간 시스템을 위해 설계된 만큼, 메모리를 사전에 할당하고 무한정 리소스를 사용하는 작업을 회피하는 등 예측 가능한 리소스 처리에 중점을 둡니다.17



Fast DDS는 다양한 네트워크 환경에 대응하기 위해 여러 발견 메커니즘을 제공합니다.

- **단순 발견(Simple Discovery):** UDP 멀티캐스트를 사용하는 표준 방식의 기본 발견 메커니즘입니다.31
- **발견 서버(Discovery Server):** 이는 표준 DDS의 한계를 극복하기 위한 eProsima의 독자적인 기능입니다. 표준적인 동적 발견 방식은 노드 수가 많아지거나 WiFi와 같이 멀티캐스트가 불안정한 환경에서는 비효율적이거나 동작하지 않을 수 있습니다.32 발견 서버는 클라이언트-서버 모델을 도입하여 이 문제를 해결합니다. 각 노드는 '클라이언트'로서 중앙의 '서버'에 접속하여 다른 노드 정보를 얻습니다. 이 방식은 멀티캐스트 트래픽을 극적으로 줄여 대규모 시스템의 확장성과 안정성을 크게 향상시킵니다.31 이는 단순한 기능 추가가 아니라, 실제 로보틱스 배포 환경에서 DDS가 마주하는 현실적인 제약을 해결하기 위한 실용적인 엔지니어링 솔루션입니다.
- **정적 발견(Static Discovery):** 동적 발견 과정을 완전히 생략하고, 통신할 상대방의 정보를 수동으로 설정하여 결정론적인 시스템을 구성할 수 있게 합니다.


데이터 발행 시 동기(synchronous) 및 비동기(asynchronous) 모드를 모두 지원합니다. 개발자는 예측 가능한 데이터 전송을 위해 블로킹(blocking) 호출을 사용하거나, 더 높은 처리량을 위해 논블로킹(non-blocking) 호출을 선택할 수 있습니다.17


매우 빠른 속도로 데이터를 생산하는 발행자로 인해 네트워크가 혼잡해지는 것을 방지하기 위해, 사용자가 직접 데이터 흐름을 제어하고 제한할 수 있는 흐름 제어기를 제공합니다.30


활성화 시, 통신 성능(지연 시간, 처리량, 패킷 손실 등)에 대한 상세한 통계 데이터를 수집하고 보고하는 기능을 제공합니다. 이 데이터는 `Fast DDS Monitor`와 같은 도구에서 시각화하여 분석할 수 있습니다.28


Fast DDS는 OMG DDS 보안 표준을 준수하는 강력한 보안 기능을 제공합니다.12


보안은 세 가지 개별적인 수준에서 플러그형 모듈로 구현됩니다.17

- **인증 (`DDS:Auth:PKI-DH`):** 공개 키 기반 구조(PKI)와 디피-헬만(Diffie-Hellman) 키 교환을 사용하여 참여자를 인증합니다. 각 참여자는 신뢰할 수 있는 인증 기관(CA)이 서명한 인증서가 필요합니다.
- **접근 제어 (`DDS:Access:Permissions`):** 특정 참여자가 어떤 도메인, 토픽, 파티션에 접근할 수 있는지를 통제합니다. 이는 서명된 거버넌스(governance) 및 권한(permissions) XML 파일을 통해 관리됩니다.
- **암호화 (`DDS:Crypto:AES-GCM-GMAC`):** 데이터 페이로드에 대한 인증된 암호화를 제공하여 기밀성(confidentiality)과 무결성(integrity)을 모두 보장합니다.


보안 기능은 기본적으로 비활성화되어 있으며, 컴파일 시 `-DSECURITY=ON` 옵션을 추가하고 애플리케이션에서 `PropertyPolicyQos`를 통해 활성화해야 합니다.35

여기서 반드시 인지해야 할 중요한 아키텍처적 트레이드오프가 존재합니다. 공식 문서에 명시된 바와 같이, 보안 기능의 사용은 최고 성능을 내는 제로카피(데이터 공유) 전송 방식과 호환되지 않습니다.35 제로카피는 공유 메모리 세그먼트를 여러 프로세스의 주소 공간에 매핑하여 성능을 극대화하는 방식입니다. 반면, DDS 보안은 각 구독자별로 고유한 세션 키를 사용하여 데이터를 암호화할 것을 요구합니다. 만약 공유 메모리에 데이터를 한 번만 암호화해서 쓴다면 모든 구독자가 동일한 키를 사용해야 하므로 보안 모델이 깨지고, 각 구독자를 위해 개별적으로 암호화한다면 한 번만 쓴다는 제로카피의 목적이 무의미해집니다. 따라서 시스템 설계자는 각 데이터 스트림에 대해 최고 수준의 지연 시간 및 처리량을 우선할 것인지(제로카피 사용), 아니면 데이터의 기밀성과 무결성을 우선할 것인지(보안 기능 사용)를 명확히 선택해야 합니다. 두 가지를 동시에 가질 수는 없습니다.


Fast DDS의 성공을 논할 때, 로봇 운영체제 ROS 2(Robot Operating System 2)와의 관계를 빼놓을 수 없습니다. Fast DDS가 오늘날과 같이 널리 사용되게 된 가장 큰 동력은 바로 ROS 2의 핵심 통신 미들웨어로 채택되었기 때문입니다.


ROS 1은 실제 산업 현장에서 사용하기에는 안정성, 보안, 실시간성 등에서 여러 한계를 가지고 있었습니다.36 이러한 문제를 해결하기 위해 등장한 ROS 2는 통신 시스템의 근간으로 DDS를 채택했습니다.


ROS 2는 RMW(ROS Middleware)라는 추상화 계층을 도입하여, 그 아래에 있는 실제 통신 미들웨어(DDS 구현체)를 컴파일 시간이나 런타임에 교체할 수 있도록 설계했습니다.33 이를 통해 개발자들은 특정 DDS 벤더에 종속되지 않고 애플리케이션을 개발할 수 있습니다.


수많은 DDS 구현체 중에서, eProsima Fast DDS는 ROS 2의 모든 장기 지원(LTS) 릴리스(예: Foxy, Galactic, Humble, Rolling) 및 대부분의 비-LTS 릴리스에서 기본 RMW 구현체로 채택되었습니다.11 이는 Open Robotics(ROS 개발 주체)가 수행한 독립적인 성능 평가에서 Fast DDS가 가장 우수한 성능을 보이는 오픈소스 옵션으로 결론 내려졌기 때문입니다.8

이러한 선택은 강력한 네트워크 효과를 창출했습니다. ROS 2는 현대 로보틱스 연구 및 개발의 사실상 표준입니다. 수많은 개발자들이 ROS 2를 배우고 사용하며, 이 과정에서 별다른 선택 없이 자연스럽게 Fast DDS를 접하게 됩니다. 이 거대한 사용자 기반은 Fast DDS에 대해 엄청난 양의 실제 환경 테스트, 버그 리포트, 커뮤니티 지원을 제공하며, 이는 다시 Fast DDS를 더욱 견고하게 만드는 피드백으로 작용합니다. 즉, ROS 2의 성공이 Fast DDS의 성공을 이끌고, Fast DDS의 발전이 다시 ROS 2 생태계를 강화하는 공생 관계가 형성된 것입니다.


ROS 2와 Fast DDS의 연동은 `rmw_fastrtps`라는 패키지를 통해 이루어집니다. 이 패키지는 실제로는 두 가지 RMW 구현을 제공하는데, 기본값인 `rmw_fastrtps_cpp`는 컴파일 타임에 타입 지원 코드를 생성하는 방식이고, `rmw_fastrtps_dynamic_cpp`는 런타임에 타입 정보를 해석하는 방식을 사용합니다.37



Fast DDS는 ROS 2의 `apt` 저장소를 통해 (`sudo apt install ros-<distro>-rmw-fastrtps-cpp`) 매우 쉽게 설치하거나, ROS 2 워크스페이스 내에서 소스 코드로부터 직접 빌드할 수 있습니다.38

사용자는 `RMW_IMPLEMENTATION` 환경 변수를 설정하는 것만으로 기본 미들웨어를 Fast DDS나 다른 DDS 구현체(예: Cyclone DDS)로 손쉽게 전환할 수 있습니다. 예를 들어, `export RMW_IMPLEMENTATION=rmw_fastrtps_cpp` 명령을 실행하면 해당 터미널 세션에서는 Fast DDS가 ROS 2의 통신 계층으로 사용됩니다.37


ROS 2 애플리케이션에서 Fast DDS의 세부적인 QoS 정책, 전송 방식 설정, 발견 메커니즘 변경 등 고급 구성을 하려면 XML 프로파일 파일을 사용합니다. 개발자는 원하는 설정을 담은 XML 파일을 작성하고, 특정 환경 변수를 통해 ROS 2 런치 시스템이 이 파일을 로드하도록 지정할 수 있습니다. 이 방식은 ROS 2 노드 코드를 재컴파일하지 않고도 통신 동작을 세밀하게 튜닝할 수 있는 유연성을 제공합니다.


앞서 언급했듯이, DDS의 기본 발견 메커니즘은 노드 수가 매우 많거나 WiFi와 같이 멀티캐스트가 불안정한 네트워크에서는 성능 저하나 발견 실패를 유발할 수 있습니다.32 이는 로봇 군집(fleet)과 같은 대규모 ROS 2 시스템에서 심각한 문제가 됩니다.

eProsima의 발견 서버는 이 문제에 대한 효과적인 해결책을 제시합니다. ROS 2 노드들을 '클라이언트'로 설정하여 하나 이상의 '서버' 프로세스에 연결되도록 구성하면, 노드 간 발견 정보 교환이 서버를 통해 중앙에서 관리됩니다. 이로써 네트워크를 범람시키던 멀티캐스트 트래픽이 사라지고, 시스템의 확장성과 안정성이 크게 향상됩니다.32 ROS 2 환경에서 발견 서버를 사용하는 것은 서버 프로세스를 실행하고 클라이언트 노드 측에서 서버 주소를 가리키는 환경 변수를 설정하는 것만으로 간단하게 적용할 수 있습니다.


Vulcanexus는 eProsima가 제공하는 '올인원 ROS 2 툴셋' 또는 ROS 2 배포판입니다.23 이는 eProsima의 전략적 방향성을 보여주는 중요한 제품입니다.

표준 ROS 2 배포판이 여러 출처의 패키지들로 구성되어 때로는 통합 수준이나 품질이 일정하지 않을 수 있는 반면, Vulcanexus는 eProsima가 직접 개발하고 유지보수하는 핵심 구성 요소들을 하나로 묶어 제공합니다. 여기에는 최신 버전의 Fast DDS뿐만 아니라, 마이크로컨트롤러를 위한 Micro-ROS, ROS 2 시스템 간 라우팅을 위한 ROS 2 Router, 모니터링 도구인 ROS 2 Monitor 등이 포함됩니다.23

이는 리눅스 세계에서 레드햇(Red Hat)이 취하는 전략과 유사합니다. 레드햇이 오픈소스 리눅스 커널과 주변 도구들을 가져와 안정적이고 지원이 보장되는 엔터프라이즈 배포판(RHEL)으로 패키징하는 것처럼, eProsima는 Vulcanexus를 통해 안정성과 통합 지원을 중시하는 상용 로보틱스 시장을 공략하고 있습니다. 이는 단순한 컴포넌트 제공자를 넘어, 잘 다듬어지고 신뢰할 수 있는 플랫폼 제공자로 나아가려는 전략적 움직임으로 해석할 수 있습니다.


성공적인 미들웨어는 그 자체의 성능뿐만 아니라, 개발자의 생산성을 높이고 디버깅을 용이하게 하는 강력한 지원 도구 생태계를 갖추고 있어야 합니다. eProsima는 Fast DDS를 중심으로 개발 라이프사이클 전반을 지원하는 풍부한 도구 모음을 구축했습니다.


`Fast DDS-Gen`은 개발자가 IDL(Interface Definition Language) 파일에 데이터 구조를 정의하면, 해당 데이터 타입을 Fast DDS 애플리케이션에서 사용하는 데 필요한 C++ 또는 파이썬 소스 코드를 자동으로 생성해주는 자바 애플리케이션입니다.28

개발자는 언어에 독립적인 IDL을 사용하여 데이터 타입을 한 번만 정의하면 됩니다. 그 후 `Fast DDS-Gen`을 실행하면 데이터의 직렬화(serialization) 및 역직렬화(deserialization)를 처리하는 `TopicDataType` 및 `TypeSupport` 클래스가 생성됩니다. 이 과정은 개발자를 복잡하고 오류가 발생하기 쉬운 직렬화 코드 작성 작업으로부터 해방시켜 줍니다.43

또한, 이 도구는 정의된 데이터 타입을 사용하는 발행기(publisher)와 구독기(subscriber) 예제 애플리케이션 골격 코드를 생성하는 기능을 제공하여, 개발 초기 단계를 크게 단축시켜 줍니다.43 파이썬 애플리케이션에서 사용할 수 있는 파이썬 바인딩 생성도 지원합니다.44


분산 시스템 개발에서 가장 어려운 점 중 하나는 시스템이 '블랙박스'처럼 동작하여 내부 상황을 파악하기 어렵다는 것입니다. eProsima는 이 문제를 해결하기 위해 강력한 모니터링 및 검사 도구를 제공합니다.

- **`eProsima Fast DDS Monitor`:** DDS 환경을 실시간으로 모니터링하기 위한 그래픽 데스크톱 애플리케이션입니다.11 이 도구를 사용하면 현재 네트워크에 어떤 참여자, 토픽, 발행기, 구독기가 활성화되어 있는지 한눈에 파악할 수 있습니다. 또한, 네트워크 토폴로지를 시각적으로 보여주고, 지연 시간(latency), 처리량(throughput), 패킷 손실률과 같은 핵심 성능 지표를 실시간으로 측정하고 그래프로 표시해 줍니다.46 이는 분산 시스템의 성능을 튜닝하고 문제를 디버깅하는 데 없어서는 안 될 필수적인 도구입니다.46
- **`eProsima Fast DDS Spy`:** DDS 네트워크를 검사하기 위한 커맨드 라인 인터페이스(CLI) 기반 도구입니다.48 GUI 환경이 없는 서버나 임베디드 환경에서 유용하며, 활성화된 엔드포인트를 쿼리하는 기능과 더불어, 특정 토픽의 실시간 데이터를 사람이 읽을 수 있는 형태로 '에코(echo)'하는 강력한 기능을 제공합니다.49 이를 통해 "내 노드들이 서로를 발견하고 있는가?", "지금 어떤 데이터가 오가고 있는가?"와 같은 질문에 신속하게 답을 얻을 수 있습니다.
- **`eProsima Fast DDS Visualizer`:** DDS 데이터를 플로팅하여 시각화하는 데 특화된 도구입니다.8

이러한 도구 모음은 DDS의 높은 학습 곡선과 복잡성을 완화하는 데 핵심적인 역할을 합니다. eProsima는 이 도구들을 오픈소스로 제공함으로써 개발자들이 DDS 플랫폼에 더 쉽게 접근하고 생산성을 높일 수 있도록 지원합니다. 이는 다른 미들웨어와의 경쟁에서 중요한 차별점이 됩니다.


eProsima는 GitHub를 통해 자사의 소프트웨어 포트폴리오를 활발하게 개발하고 공개하고 있습니다. 이는 투명성을 높이고 커뮤니티의 기여를 장려하는 전략입니다.

- **`Fast-DDS`:** 핵심 라이브러리의 메인 저장소입니다.11
- **도구 저장소:** `Fast-DDS-Gen`, `Fast-DDS-monitor`, `Fast-DDS-spy` 등 각 도구들은 별도의 저장소에서 관리되며, 활발한 커밋 활동과 다수의 스타(star), 포크(fork) 수는 높은 커뮤니티 관심도를 보여줍니다.47
- **지원 라이브러리 및 프로젝트:** 통계 데이터 처리를 위한 `Fast-DDS-statistics-backend` 34, 마이크로컨트롤러를 위한 `Micro-XRCE-DDS` 50, 그리고 서로 다른 네트워크나 프로토콜을 연결하는 `DDS-Router` 및 `Integration-Service` 8 와 같은 관련 프로젝트들은 eProsima가 단순한 라이브러리를 넘어 포괄적인 미들웨어 생태계를 구축하고 있음을 보여줍니다.


미들웨어를 선택할 때 가장 중요한 고려 사항 중 하나는 성능입니다. 이 장에서는 eProsima가 공개한 벤치마크 데이터와 독립적인 연구 결과를 바탕으로 Fast DDS의 성능 특성을 분석하고, 주요 경쟁 제품들과의 비교를 통해 그 장단점을 명확히 하겠습니다.


eProsima는 자사 웹사이트와 GitHub 저장소를 통해 지연 시간(latency)과 처리량(throughput)을 중심으로 한 상세한 성능 벤치마크 결과를 정기적으로 공개하고 있습니다.29


벤치마크 결과는 통신 환경에 따라 성능이 극적으로 달라진다는 점을 명확히 보여줍니다.

- **프로세스 내(Intra-Process):** 동일한 프로세스 내의 엔티티 간 통신은 전송 계층을 완전히 우회하므로 가장 빠릅니다. 지연 시간이 거의 0에 가깝습니다.29
- **공유 메모리(Shared Memory, SHM):** 동일한 호스트 내의 서로 다른 프로세스 간 통신에서는 SHM이 UDP보다 훨씬 낮은 지연 시간을 보입니다. 특히 대용량 페이로드의 경우, 메모리 복사를 제거하는 제로카피(Zero-Copy) SHM이 최고의 성능을 나타냅니다.29
- **UDP:** 서로 다른 머신 간의 통신에 필수적인 표준 네트워크 전송 방식입니다. 세 가지 방식 중에서는 가장 높은 지연 시간을 보이지만, 네트워크를 통한 통신에는 불가피한 선택입니다.29

이러한 성능 계층은 시스템 설계에 중요한 시사점을 줍니다. 최고의 성능을 얻기 위해서는 통신량이 많은 프로세스들을 가능한 한 동일한 머신에 배치하여 SHM이나 프로세스 내 통신을 활용하도록 아키텍처를 설계해야 합니다.


- 자율주행차 분야를 대상으로 FastDDS, Zenoh, vSomeIP를 비교한 한 연구(arXiv)에서는, FastDDS가 Zenoh에 이어 두 번째로 빠른 발견 성능을 보였으며, 테스트된 미들웨어 중 가장 낮은 리소스 사용량을 기록했습니다. 또한 최선 노력(best-effort) 통신 시나리오에서 더 나은 확장성을 보였습니다.53
- 다른 연구들 역시 DDS 구현체들이 일반적으로 효율적이지만, 구체적인 성능은 부하 조건이나 QoS 구성에 따라 크게 달라질 수 있음을 지적합니다.10

결론적으로, '가장 빠른 미들웨어'라는 단 하나의 정답은 존재하지 않습니다. 성능은 네트워크 토폴로지, 페이로드 크기, 신뢰성 요구사항, 개발 비용 등 다양한 변수에 따라 달라지는 다차원적인 트레이드오프의 결과입니다. 따라서 아키텍트는 "Fast DDS가 X보다 빠르다"와 같은 단편적인 주장이 아닌, 자신의 특정 사용 사례에 맞는 최적의 솔루션이 무엇인지 종합적으로 판단해야 합니다.


| 기능               | eProsima Fast DDS               | RTI Connext DDS                     | Eclipse Cyclone DDS      |
| ------------------ | ------------------------------- | ----------------------------------- | ------------------------ |
| **라이선스**       | Apache 2.0 (오픈소스)           | 상용                                | EPL 2.0 / MIT (오픈소스) |
| **기본 아키텍처**  | 분산형 (Decentralized)          | 분산형 (Decentralized)              | 분산형 (Decentralized)   |
| **핵심 언어**      | C++                             | C++, Java, C#, Ada                  | C                        |
| **지원 전송 방식** | UDP, TCP, SHM, Zero-Copy SHM    | UDP, TCP, SHM, Zero-Copy SHM        | UDP, TCP, SHM            |
| **안전 인증 버전** | Safe DDS (ISO 26262)            | Connext Cert                        | 없음                     |
| **고급 QoS**       | Content-Filtered Topics 등 지원 | Coherent Sets 등 더 광범위하게 지원 | 일부 지원                |
| **개발 도구**      | Monitor, Spy, Gen 등            | Admin Console, Tuner, Analyzer 등   | 일부 커뮤니티 도구       |
| **주요 생태계**    | ROS 2, 로보틱스                 | 항공우주, 국방, 의료                | ROS 2, 범용              |

표 1: DDS 구현체 기능 비교. 이 표는 21의 정보를 종합하여 구성되었습니다.

- **RTI Connext DDS:** RTI는 DDS 시장의 선두주자이자 상용 솔루션입니다. 일반적으로 Fast DDS보다 더 많은 언어 바인딩(C#, Ada), 성숙한 부가 서비스(라우팅, 영속성 서비스 등), 그리고 오랜 기간 검증된 안전 인증 버전을 제공합니다.55 아키텍처는 Fast DDS와 동일한 분산형 모델을 채택하고 있습니다.56 성능은 특정 시나리오에 따라 우위가 갈리며, 상호운용성은 RTPS/UDP 기반의 핵심 기능에서는 보장되지만, WString 타입이나 SHM과 같은 비표준 전송 방식에서는 호환성 문제가 보고된 바 있습니다.21
- **Eclipse Cyclone DDS & OpenDDS:** 이들은 Fast DDS와 함께 널리 사용되는 주요 오픈소스 DDS 구현체입니다.41 eProsima가 자체적으로 수행한 벤치마크에서는 다수의 지연 시간 및 처리량 테스트에서 Fast DDS가 Cyclone DDS와 OpenDDS보다 우수한 성능을 보였다고 주장합니다.52 과거 OpenDDS와 OpenSplice는 중앙 데몬을 사용하는 연합형(federated) 아키텍처를 주로 사용했으나, 현재는 분산형 모델도 지원합니다.56

이러한 비교는 상호운용성이 흑백 논리가 아닌 스펙트럼의 문제임을 보여줍니다. RTPS 표준이 기본적인 상호운용성을 보장하지만, 고급 기능이나 고성능을 추구할수록 벤더 종속적인 구현에 의존하게 될 가능성이 높습니다. 따라서 매우 복잡하고 높은 성능이 요구되는 시스템에서는 미묘하고 디버깅하기 어려운 상호운용성 문제를 피하기 위해 단일 벤더의 생태계에 머무는 것이 더 실용적인 선택일 수 있습니다.


| 특성                  | DDS (e.g., Fast DDS)        | MQTT                               | ZeroMQ (ZMQ)                          |
| --------------------- | --------------------------- | ---------------------------------- | ------------------------------------- |
| **아키텍처 패러다임** | 데이터 중심 (Data-Centric)  | 메시지 중심 (Message-Centric)      | 소켓 추상화 (Socket Abstraction)      |
| **통신 패턴**         | 브로커 없는 발행-구독       | 브로커 기반 발행-구독              | 패턴 라이브러리 (Pub-Sub, Req-Rep 등) |
| **발견(Discovery)**   | 자동/동적                   | 브로커가 관리                      | 수동/프로그래밍 방식                  |
| **데이터 모델**       | IDL을 통한 강력한 타입 정의 | 불투명한 페이로드 (Opaque Payload) | 원시 바이트 (Raw Bytes)               |
| **신뢰성**            | 풍부한 QoS 정책 (22+개)     | 3단계 QoS 레벨                     | 프로그래밍 방식 구현                  |
| **일반적 지연 시간**  | 매우 낮음 (로컬)            | 낮음 ~ 높음 (WAN)                  | 매우 낮음 (로컬)                      |
| **주요 사용 사례**    | 실시간 로보틱스, 산업 제어  | IoT 원격 측정, 클라우드 연동       | 고성능 컴퓨팅, 금융                   |

표 2: 미들웨어 아키텍처 비교. 이 표는 4의 정보를 종합하여 구성되었습니다.

- **MQTT:** MQTT는 중앙 브로커에 의존하는 메시지 중심 미들웨어입니다.5 저대역폭, 고지연, 불안정한 네트워크 환경의 IoT 원격 측정(telemetry) 데이터 수집이나 디바이스-클라우드 연동에 최적화되어 있습니다. 반면, DDS는 브로커가 없는 분산형 구조로, 고성능, 저지연이 요구되는 로컬 네트워크(로보틱스, 산업 제어)에 강점을 보입니다.59 QoS 측면에서도 MQTT는 3단계의 레벨을 제공하는 반면, DDS는 22개 이상의 세분화된 정책으로 훨씬 정밀한 제어가 가능합니다.54 복잡한 시스템에서는 DDS와 MQTT를 함께 사용하고 둘 사이에 브릿지를 두는 하이브리드 아키텍처가 효과적일 수 있으며, eProsima의 

  `Integration Service`는 바로 이러한 요구를 충족시키기 위한 제품입니다.51

- **ZeroMQ (ZMQ):** ZMQ는 미들웨어가 아니라, 개발자가 직접 미들웨어를 구축할 수 있는 구성 요소(패턴)를 제공하는 고성능 소켓 라이브러리입니다.4 ZMQ를 사용하면 발견, 신뢰성, 타입 관리와 같은 기능들을 개발자가 직접 구현해야 합니다. 예를 들어, 신뢰성 있는 발행-구독 통신을 구현하려면 여러 ZMQ 패턴을 애플리케이션 코드 수준에서 조합해야 합니다.4 따라서 ZMQ는 최고의 유연성을 원하는 개발자에게 적합하며, DDS는 완전한 기능이 갖춰진 미들웨어 솔루션을 원하는 개발자에게 적합합니다.


이론적인 분석을 넘어, Fast DDS가 실제 세계의 까다로운 시스템에서 어떻게 활용되고 있는지 구체적인 사례를 통해 그 역량을 입증하겠습니다. 이러한 사례들은 Fast DDS가 단순한 연구용 오픈소스 프로젝트를 넘어, 미션 크리티컬 상용 및 과학 시스템을 위한 검증된 솔루션임을 보여줍니다.


- **사례:** eProsima와 아부다비 기술 혁신 연구소(TII)의 자율주행 레이싱 차량 개발 협력.23
- **기술적 과제:** 고속 주행 중인 차량에서 4K 카메라, LiDAR 등 다수의 고대역폭 센서로부터 들어오는 방대한 데이터를 실시간으로 처리하여 인지, 판단, 제어를 수행해야 하는 극한의 실시간성, 초저지연, 고처리량 요구사항.
- **해결책:** TII는 계층화된 통신 아키텍처를 구축했습니다. 대용량 센서 데이터를 처리하는 인지 파이프라인에는 ROS 2와 Fast DDS를 함께 사용하고, 특히 제로카피 기능을 활용하여 데이터 처리 효율을 극대화했습니다. 반면, 더 낮은 대역폭의 제어 루프 통신에는 순수 Fast DDS를 직접 사용하여 시스템을 최적화했습니다.62 이는 동일한 미들웨어를 데이터 스트림의 특성에 맞게 다르게 활용하는 정교한 시스템 설계의 좋은 예시입니다.


- **사례:** Hexagon, Airbus, eProsima가 협력하여 C295 해상 감시 항공기에 탑재된 실시간 수중 감시 시스템 개발.63
- **기술적 과제:** 항공기에서 수심 측량 LiDAR 센서가 수집한 65kB 이상의 대용량 데이터 샘플을 비행 중 실시간으로 시각화 및 분석 콘솔로 안정적으로 전송해야 하는 과제.
- **해결책:** Fast DDS가 신뢰성 있는 데이터 전송을 위해 채택되었습니다. 특히 이 프로젝트의 요구사항을 충족시키기 위해, eProsima는 대용량 데이터 샘플을 처리할 수 있도록 Fast DDS의 영속성(persistence) 서비스를 맞춤형으로 개선했습니다. 이는 eProsima가 상용 고객을 위해 핵심 기능을 맞춤 개발할 수 있는 역량을 갖추고 있음을 보여주는 사례입니다.63


- **사례:** 유럽남방천문대(ESO)는 세계 최대 광학 망원경인 ELT의 제어 시스템 통신 계층으로 Fast DDS를 사용합니다.8
- **기술적 과제:** 25,000개 이상의 센서와 15,000개 이상의 액추에이터로 구성된 거대하고 분산된 제어 시스템을 단일 장애점(single point of failure) 없이 안정적이고 고성능으로 운영해야 하는 과제.
- **해결책:** 분산형의 견고한 아키텍처를 제공하는 DDS가 채택되었습니다. ESO 측의 추천사에서는 Fast DDS가 킬로헤르츠(kHz) 단위의 고주파 데이터 발행부터 감독 제어 통신에 이르기까지, 자신들의 까다로운 요구사항을 완벽하게 충족시켰다고 밝혔습니다.8


- **로보틱스:** ROS 2 생태계를 넘어, Fast DDS는 iRobot사의 Roomba 로봇 청소기와 같은 상용 제품 64, 그리고 다양한 로봇 군집 관리 및 자율주행 솔루션에 사용되고 있습니다.8

- **IIoT:** Fast DDS는 새로운 산업용 IoT 플랫폼의 기반 미들웨어로 자리매김하고 있습니다.8 또한 eProsima는 극도로 제한된 리소스를 가진 마이크로컨트롤러를 DDS 데이터 공간에 연결하기 위한 

  `Micro XRCE-DDS`를 제공하여, IoT의 가장자리(edge)까지 그 영향력을 확장하고 있습니다.31

이러한 사례들은 오픈소스 소프트웨어가 상용 솔루션보다 견고하지 않을 것이라는 일부의 인식을 정면으로 반박합니다. 안전, 비용, 과학적 발견과 같은 높은 이해관계가 걸려 있고 극한의 기술적 요구사항을 가진 조직들이 Fast DDS를 선택하고 비용을 지불하여 맞춤 개발까지 의뢰했다는 사실은, 이 기술의 성능, 신뢰성, 그리고 eProsima 팀의 역량에 대한 강력한 증거가 됩니다.


본 보고서의 분석을 종합하여, 시스템 아키텍트가 미들웨어를 선택할 때 고려해야 할 사항을 제시하고 Fast DDS의 미래 방향성을 전망하며 결론을 맺겠습니다.


최적의 미들웨어 선택은 하나의 정답을 찾는 과정이 아니라, 프로젝트의 고유한 제약 조건과 요구사항에 맞춰 일련의 질문에 답하며 최적의 균형점을 찾아가는 과정입니다. 미들웨어 선택은 한 번 결정하면 바꾸기 어려운 장기적인 아키텍처적 약속이라는 점을 인지해야 합니다. ROS 2의 RMW 계층이 이론적으로는 미들웨어 교체를 허용하지만 33, 실제 시스템은 특정 구현체의 고유 기능(예: 발견 서버)이나 성능 특성에 맞춰 튜닝되는 경우가 많고, 벤더 간 상호운용성 문제 21 또한 존재하기 때문입니다. 잘못된 초기 선택은 상당한 기술 부채나 값비싼 재설계로 이어질 수 있습니다.

아키텍트는 다음 질문들을 스스로에게 던져보아야 합니다.

- **네트워크 환경은 어떠한가?** (안정적인 로컬 LAN인가, 아니면 WAN/클라우드인가? 신뢰할 수 있는 멀티캐스트를 사용할 수 있는가?) -> 이 질문은 기본 DDS, 발견 서버를 갖춘 DDS, 또는 MQTT 사이의 선택을 좌우합니다.
- **실시간 요구사항은 무엇인가?** (경성(Hard) 실시간인가, 연성(Soft) 실시간인가, 아니면 최선 노력(Best-Effort)으로 충분한가?) -> 이는 QoS 구성과 DDS와 다른 미들웨어 간의 선택에 영향을 줍니다.
- **개발 및 지원 모델은 무엇인가?** (내부 전문성과 커뮤니티 지원으로 충분한가, 아니면 상용 수준의 기술 지원과 SLA가 필요한가?) -> 이는 오픈소스(Fast DDS)와 상용(RTI Connext) 솔루션 사이의 선택을 결정합니다.
- **시스템의 규모는 어느 정도인가?** (소수의 노드인가, 아니면 수백, 수천 개의 노드인가?) -> 이는 발견 메커니즘(P2P 대 발견 서버) 선택에 직접적인 영향을 미칩니다.
- **보안 요구사항은 무엇인가?** (데이터 암호화가 필수적인가?) -> 이는 보안 기능과 제로카피 성능 사이의 근본적인 트레이드오프를 결정하게 합니다.


- **표준의 진화:** DDS 표준은 DDS-TSN, DDS-TCP 등 계속해서 진화하고 있으며 12, Fast DDS는 이러한 새로운 표준들을 선도적으로 구현하는 역할을 계속할 것으로 보입니다.
- **안전 필수 시스템 시장의 부상:** eProsima가 ISO 26262 인증을 받은 `Safe DDS`를 출시한 것은 23, 자동차, 항공우주와 같은 안전 필수(safety-critical) 시장에서 상용 벤더와 직접 경쟁하겠다는 명확한 전략적 방향을 보여줍니다.
- **AI/로보틱스와의 심화된 통합:** 로보틱스와 AI 기술이 더욱 긴밀하게 결합됨에 따라, 모델 학습 및 추론을 위한 대용량 데이터셋을 효율적으로 전송하고 AI 프레임워크와 통합하는 미들웨어의 역할이 더욱 중요해질 것입니다. eProsima가 이 분야에 집중하고 있는 만큼 22, 이러한 미래에 유리한 위치를 점하고 있습니다.
- **오픈소스와 상용의 역학 관계:** Fast DDS의 지속적인 성공은 RTI와 같은 상용 벤더들에게 혁신과 코드 공개에 대한 압력으로 작용하는 동시에, eProsima에게는 상용 서비스 및 지원 오퍼링을 더욱 강화하도록 하는 동력이 될 것입니다.

결론적으로, eProsima Fast DDS는 강력한 국제 표준을 기반으로 하면서도, 현실 세계의 문제들을 해결하기 위한 실용적인 기능들을 더한 고성능 오픈소스 미들웨어입니다. 특히 ROS 2 생태계의 기본 통신 계층으로 자리 잡으며 막강한 영향력을 확보했으며, 풍부한 개발 도구와 검증된 실제 적용 사례들을 통해 단순한 오픈소스 프로젝트를 넘어 미션 크리티컬 시스템을 위한 신뢰할 수 있는 솔루션으로 인정받고 있습니다. 미래에도 기술 표준의 발전과 시장의 요구에 발맞춰 지속적으로 진화하며 분산 시스템 통신 분야에서 핵심적인 역할을 수행할 것으로 기대됩니다.

------


다음 표는 eProsima Fast DDS가 지원하는 운영체제, 아키텍처, 컴파일러 및 지원 등급을 요약한 것입니다. 지원 등급(Tier)은 eProsima 개발팀의 테스트 및 지원 수준을 나타내며, Tier 1이 가장 높은 수준의 지원을 의미합니다.

| 운영체제                 | 아키텍처 | 컴파일러 / 툴체인              | 지원 등급 |
| ------------------------ | -------- | ------------------------------ | --------- |
| **Ubuntu 22.04 (Jammy)** | amd64    | GCC, Clang                     | Tier 1    |
|                          | arm64    | GCC, Clang                     | Tier 1    |
| **Ubuntu 24.04 (Noble)** | amd64    | GCC                            | Tier 3    |
|                          | arm64    | GCC                            | Tier 3    |
| **Windows 10**           | amd64    | Visual Studio 2019 (MSVC v142) | Tier 1    |
|                          | amd32    | Visual Studio 2019 (MSVC v142) | Tier 3    |
| **Windows 11**           | amd64    | Visual Studio 2022 (MSVC v143) | Tier 3    |
|                          | amd32    | Visual Studio 2022 (MSVC v143) | Tier 3    |
| **macOS Mojave (10.14)** | amd64    | Clang                          | Tier 1    |
| **Debian 10 (Buster)**   | amd64    | GCC 8                          | Tier 3    |
|                          | arm64    | GCC 8                          | Tier 3    |
| **Android 12 / 13**      | amd64    | SDK 31 / 33                    | Tier 3    |
|                          | arm64    | SDK 31 / 33                    | Tier 3    |
|                          | arm32    | SDK 31 / 33                    | Tier 3    |
| **QNX 7.1**              | amd64    | QCC (GCC 8.3 기반)             | Tier 3    |
|                          | arm64    | QCC (GCC 8.3 기반)             | Tier 3    |

표 3: eProsima Fast DDS 플랫폼 지원 매트릭스. 이 표는 65의 정보를 종합하여 구성되었습니다.


1. docs.px4.io, accessed July 5, 2025, [https://docs.px4.io/v1.12/ko/dev_setup/fast-dds-installation.html#:~:text=eProsima%20Fast%20DDS%20%EB%8A%94%20OMG,%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C%EC%9D%98%20C%2B%2B%20%EA%B5%AC%ED%98%84%EC%9E%85%EB%8B%88%EB%8B%A4.](https://docs.px4.io/v1.12/ko/dev_setup/fast-dds-installation.html#:~:text=eProsima Fast DDS 는 OMG,프로토콜의 C%2B%2B 구현입니다.)
2. What is DDS? - DDS Foundation, accessed July 5, 2025, https://www.dds-foundation.org/what-is-dds-3/
3. Data Distribution Service (DDS) | Object Management Group, accessed July 5, 2025, https://www.omg.org/omg-dds-portal/
4. ZeroMQ vs. DDS Software: What's the Difference? | RTI - Real-Time Innovations, accessed July 5, 2025, https://www.rti.com/blog/2017/04/20/why-would-anyone-use-dds-over-zeromq
5. DDS and MQTT: Basics, Challenges and Integration Benefits | EMQ - EMQX, accessed July 5, 2025, https://www.emqx.com/en/blog/navigating-dds-basics-limitations-and-integration-with-mqtt
6. Title : What is DDS ? | SJ.Home, accessed July 5, 2025, https://sjang1594.github.io/study/DDS.html
7. Introduction to DDS - eProsima, accessed July 5, 2025, https://www.eprosima.com/developer-resources/whitepapers/dds
8. eProsima Fast DDS - The middleware powering the ESO ELT, accessed July 5, 2025, https://www.eso.org/sci/meetings/2023/RTC4AO/01_11_martin_losa.pdf
9. DDS와 RTPS 개념정리 - Lab Note - 티스토리, accessed July 5, 2025, [https://lab-notes.tistory.com/entry/DDS-DDS%EC%99%80-RTPS-%EA%B0%9C%EB%85%90%EC%A0%95%EB%A6%AC](https://lab-notes.tistory.com/entry/DDS-DDS와-RTPS-개념정리)
10. A Survey on Experimental Performance Evaluation of Data Distribution Service (DDS) Implementations - arXiv, accessed July 5, 2025, https://arxiv.org/pdf/2310.16630
11. eProsima/Fast-DDS: The most complete DDS - Proven: Plenty of success cases. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 5, 2025, https://github.com/eProsima/Fast-DDS
12. What's in the DDS Standard? - DDS Foundation, accessed July 5, 2025, https://www.dds-foundation.org/omg-dds-standard/
13. What is RTPS and DDSI? - ZettaScale Knowledge Base, accessed July 5, 2025, https://kbase.zettascale.tech/article/what-is-rtps-and-ddsi/
14. Protocols/rtps - Wireshark Wiki, accessed July 5, 2025, https://wiki.wireshark.org/Protocols/rtps
15. Real-Time Publish-Subscribe (RTPS) - ORTE, accessed July 5, 2025, https://orte.sourceforge.net/rtps1.2.pdf
16. RTPS Introduction - eProsima, accessed July 5, 2025, https://www.eprosima.com/developer-resources/whitepapers/rtps
17. Fast DDS Documentation, accessed July 5, 2025, https://media.readthedocs.org/pdf/eprosima-fast-rtps/latest/eprosima-fast-rtps.pdf
18. 1.1.1. The DCPS conceptual model - Fast DDS, accessed July 5, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/getting_started/definitions.html
19. Real-time Publish-Subscribe (RTPS) [DDS Foundation Wiki], accessed July 5, 2025, https://www.omgwiki.org/ddsf/doku.php?id=ddsf:public:guidebook:06_append:glossary:r:rtps
20. About the DDS Interoperability Wire Protocol Specification Version 2.5, accessed July 5, 2025, https://www.omg.org/spec/DDSI-RTPS/2.5/About-DDSI-RTPS
21. Compilibility problems of eProsima FastDDS and RTI Context DDS, accessed July 5, 2025, https://community.rti.com/forum-topic/compilibility-problems-eprosima-fastdds-and-rti-context-dds
22. Company - eProsima, accessed July 5, 2025, https://www.eprosima.com/company
23. eProsima: Middleware, Robots and AI, accessed July 5, 2025, https://www.eprosima.com/
24. Fast DDS - A standalone CPP middleware implementation providing both the OMG DDS 1.4 and the OMG RTPS 2.2 interoperable wire-protocol standards - Third-Party Products & Services - MATLAB & Simulink - MathWorks, accessed July 5, 2025, https://www.mathworks.com/products/connections/product_detail/eprosima-fast-dds.html
25. Team - eProsima, accessed July 5, 2025, https://www.eprosima.com/company/team
26. The Importance of Robust Architecture in DDS Implementations and Best Practices, accessed July 5, 2025, https://www.eprosima.com/index.php/company-all/news/338-robust-architecture-dds-implementation
27. Middleware - eProsima, accessed July 5, 2025, https://www.eprosima.com/component/content/category/english-categories/middleware?Itemid=0
28. eProsima Fast DDS, accessed July 5, 2025, https://fast-dds.docs.eprosima.com/
29. eProsima Fast DDS Performance, accessed July 5, 2025, https://www.eprosima.com/developer-resources/performance/eprosima-fast-dds-performance
30. What is Fast DDS? - 3.2.2, accessed July 5, 2025, https://fast-dds.docs.eprosima.com/en/v3.2.2/02-formalia/titlepage.html
31. DDS - eProsima, accessed July 5, 2025, https://www.eprosima.com/component/content/category/english-categories/resources/dds?Itemid=102
32. 16.2. Use ROS 2 with Fast-DDS Discovery Server - 3.2.2, accessed July 5, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/ros2/discovery_server/ros2_discovery_server.html
33. 2023-09 ROS 2 RMW alternate - AWS, accessed July 5, 2025, https://cdck-file-uploads-us1.s3.dualstack.us-west-2.amazonaws.com/flex022/uploads/ros/original/3X/a/9/a9412734d6896de23fa64665cf518f232a46a5b8.pdf
34. eProsima/Fast-DDS-statistics-backend - GitHub, accessed July 5, 2025, https://github.com/eProsima/Fast-DDS-statistics-backend
35. 8. Security - 3.2.2 - Fast DDS - eProsima, accessed July 5, 2025, http://fast-dds.docs.eprosima.com/en/latest/fastdds/security/security.html
36. [Micro-ROS] Micro XRCE-DDS 특징 - Interactics - 티스토리, accessed July 5, 2025, [https://huroint.tistory.com/entry/Micro-ROS-Micro-XRCE-DDS-%ED%8A%B9%EC%A7%95](https://huroint.tistory.com/entry/Micro-ROS-Micro-XRCE-DDS-특징)
37. 16. ROS 2 using Fast DDS middleware - 3.2.2, accessed July 5, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/ros2/ros2.html
38. eProsima Fast DDS - ROS 2 Documentation: Galactic documentation, accessed July 5, 2025, https://docs.ros.org/en/galactic/Installation/DDS-Implementations/Working-with-eProsima-Fast-DDS.html
39. eProsima Fast DDS - ROS 2 Documentation: Foxy documentation, accessed July 5, 2025, https://docs.ros.org/en/foxy/Installation/DDS-Implementations/Working-with-eProsima-Fast-DDS.html
40. eProsima Fast DDS - ROS 2 文档, accessed July 5, 2025, https://ros2docs.robook.org/rolling/Installation/DDS-Implementations/Working-with-eProsima-Fast-DDS.html
41. About different ROS 2 DDS/RTPS vendors, accessed July 5, 2025, https://docs.ros.org/en/foxy/Concepts/About-Different-Middleware-Vendors.html
42. eProsima Fast DDS - ROS 2 Documentation, accessed July 5, 2025, https://docs.ros.org/en/humble/Installation/RMW-Implementations/DDS-Implementations/Working-with-eProsima-Fast-DDS.html
43. 1. Introduction - 3.2.2 - Fast DDS - eProsima, accessed July 5, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastddsgen/introduction/introduction.html
44. 2. Usage - 3.2.2 - Fast DDS - eProsima, accessed July 5, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastddsgen/usage/usage.html
45. eProsima Fast DDS Monitor 2.0: New Release - YouTube, accessed July 5, 2025, https://www.youtube.com/watch?v=Y0jPmEtxtx8
46. eProsima Fast DDS Monitor Documentation, accessed July 5, 2025, https://fast-dds-monitor.readthedocs.io/
47. eProsima/Fast-DDS-monitor - GitHub, accessed July 5, 2025, https://github.com/eProsima/Fast-DDS-monitor
48. eProsima Fast DDS Spy, accessed July 5, 2025, https://www.eprosima.com/middleware/tools/fast-dds-spy
49. eProsima/Fast-DDS-spy - GitHub, accessed July 5, 2025, https://github.com/eProsima/Fast-DDS-spy
50. eProsima - GitHub, accessed July 5, 2025, https://github.com/eprosima
51. eProsima/FastDDS-SH: eProsima's Integration Service ... - GitHub, accessed July 5, 2025, https://github.com/eProsima/FastDDS-SH
52. Performance - eProsima, accessed July 5, 2025, https://www.eprosima.com/developer-resources/performance
53. Automotive Middleware Performance: Comparison of FastDDS, Zenoh and vSomeIP This research is accomplished within the project "AUTOtechagil" (FKZ 01IS22088x). - arXiv, accessed July 5, 2025, https://arxiv.org/html/2505.02734v1
54. Evaluating DDS, MQTT, and ZeroMQ Under Different IoT Traffic Conditions - Distributed Object Computing (DOC) Group for DRE Systems, accessed July 5, 2025, http://www.dre.vanderbilt.edu/~gokhale/WWW/papers/M4IoT2020.pdf
55. Comparing Open Source DDS to RTI Connext DDS: Considerations in Picking the Right DDS Solution to Run Your Distributed System, accessed July 5, 2025, https://www.rti.com/blog/picking-the-right-dds-solution
56. Criteria Set for Evaluation of different DDS Distributions - IJRASET, accessed July 5, 2025, https://www.ijraset.com/fileserve.php?FID=29243
57. Data Distribution Service - kjmin's devlog - 티스토리, accessed July 5, 2025, https://leichtjoon.tistory.com/54
58. eProsima/benchmarking - GitHub, accessed July 5, 2025, https://github.com/eProsima/benchmarking
59. Navigating IIoT Protocols: Comparing DDS and MQTT - RTI, accessed July 5, 2025, https://www.rti.com/blog/comparing-dds-and-mqtt
60. How Does DDS Compare to other IoT Technologies? - DDS Foundation, accessed July 5, 2025, https://www.dds-foundation.org/features-benefits/
61. ROSIN project: ROS2 Integration Service - eProsima, accessed July 5, 2025, https://www.eprosima.com/r-d-projects/rosin-project-ros2-integration-service
62. Fast DDS at Full Speed: eProsima Delivers Real-Time Comms for TII ..., accessed July 5, 2025, https://www.eprosima.com/news/fast-dds-at-full-speed-eprosima-delivers-real-time-comms-for-tii-autonomous-vehicles
63. Hexagon and Airbus: Fast DDS for real-time maritime surveillance - eProsima, accessed July 5, 2025, https://www.eprosima.com/news/hexagon-airbus-fast-dds
64. Internet of Things - eProsima, accessed July 5, 2025, https://www.eprosima.com/middleware/sectors/internet-of-things
65. Fast-DDS/QUALITY.md at master - GitHub, accessed July 5, 2025, https://github.com/eProsima/Fast-DDS/blob/master/QUALITY.md
66. Dependencies and compatibilities - 3.2.2 - Fast DDS, accessed July 5, 2025, https://fast-dds.docs.eprosima.com/en/v3.2.2/notes/versions.html
67. Fast-DDS/PLATFORM_SUPPORT.md at master - GitHub, accessed July 5, 2025, https://github.com/eProsima/Fast-DDS/blob/master/PLATFORM_SUPPORT.md


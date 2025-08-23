# DDS와 CAN 버스 게이트웨이의 아키텍처



데이터 분산 서비스(DDS)는 단순히 메시지를 전달하는 프로토콜이 아니라, 데이터 중심(data-centric) 분산 시스템을 구축하기 위한 OMG(Object Management Group) 표준 미들웨어입니다.1 DDS의 핵심 추상화 개념은 '글로벌 데이터 공간(Global Data Space, GDS)'으로, 이는 애플리케이션이 상호작용하는 가상적이고 분산된 데이터 저장소입니다.2 이 접근 방식은 DDS가 통신 세부 사항을 애플리케이션으로부터 추상화하여 개발자가 데이터의 상태와 로직에 집중할 수 있도록 한다는 점에서 메시지 중심 미들웨어와 근본적인 차이를 보입니다.2

DDS의 통신 모델은 발행-구독(publish-subscribe) 패턴을 기반으로 합니다.3 발행자(Publisher)와 구독자(Subscriber)는 특정 데이터 스트림을 위한 강력한 타입의 채널인 토픽(Topic)을 통해 익명으로 통신하며, 이로 인해 서로 분리(decoupled)됩니다.3 이러한 분리 구조는 모듈화되고 확장 가능한 아키텍처를 구현하는 핵심적인 이점입니다.1

주요 아키텍처 구성 요소는 다음과 같습니다:

- **도메인(Domain):** 도메인 ID로 식별되는 격리된 통신 평면으로, 서로 다른 시스템이 간섭 없이 통신할 수 있도록 보장합니다.2
- **도메인 참여자(DomainParticipant):** 애플리케이션이 DDS 도메인에 진입하는 관문 역할을 합니다.3
- **토픽(Topic), 데이터 작성자(DataWriter), 데이터 판독기(DataReader):** 인터페이스 정의 언어(IDL)를 통해 데이터 타입을 정의하고, 데이터 샘플을 발행하며, 이를 수신하는 기본적인 엔티티들입니다.3

DDS의 가장 두드러진 특징은 서비스 품질(Quality of Service, QoS) 정책으로, 이는 DDS를 "강화된(on steroids)" 발행-구독 기술로 만들어 줍니다.4 실시간 및 미션 크리티컬 시스템에 필수적인 주요 QoS 정책들은 통신 행위를 세밀하게 제어합니다.

- **신뢰성(RELIABILITY):** 데이터 전송을 보장하며, 제어 명령과 같이 반드시 전달되어야 하는 데이터에 필수적입니다.
- **지속성(DURABILITY):** 늦게 참여하는 구독자가 과거 데이터를 수신할 수 있게 합니다.
- **이력(HISTORY):** 저장할 과거 데이터의 양을 설정합니다.
- **마감일(DEADLINE):** 데이터 업데이트의 예상 주기를 감시합니다.
- **활성(LIVELINESS):** 참여자의 존재 여부를 모니터링합니다.
- **전송 우선순위(TRANSPORT_PRIORITY):** 하위 전송 계층의 메시지 우선순위에 영향을 줍니다.

이러한 QoS 정책들은 신뢰성, 성능, 리소스 사용에 대한 정밀한 제어를 제공하며, 복잡한 시스템에서 DDS가 선택되는 핵심적인 이유가 됩니다.1 또한, DDS는 중앙 브로커나 별도의 구성 파일 없이 런타임에 새로운 참여자, 토픽, QoS 요구사항을 자동으로 발견하는 동적 발견(Dynamic Discovery) 기능을 제공하여 진정한 '플러그 앤 플레이'를 가능하게 합니다.2


DDS의 높은 수준의 추상화와 대조적으로, CAN은 차량과 같이 전기적으로 열악한 환경에서 마이크로컨트롤러(ECU) 간의 견고하고 저비용의 실시간 통신을 위해 설계된 저수준 메시지 기반 프로토콜(ISO 11898)입니다.9 CAN의 주된 개발 목적은 차량 내 배선의 복잡성과 무게를 줄이는 것이었습니다.12

CAN의 통신 모델은 다음과 같은 특징을 가집니다:

- **방송 및 메시지 기반:** 버스 상의 모든 노드는 전송되는 모든 프레임을 수신합니다. 이후 각 지능형 노드는 프레임의 식별자(Identifier)를 기반으로 해당 프레임을 처리할지 무시할지 결정합니다.12 이는 중앙 컨트롤러가 없는 다중 마스터(multi-master) 시스템입니다.14
- **중재(Arbitration):** CAN의 결정성(determinism)을 보장하는 핵심 메커니즘입니다. CAN은 메시지 식별자를 기반으로 한 비파괴적인 비트 단위 중재(bitwise arbitration) 방식을 사용합니다. 숫자적으로 가장 낮은 ID를 가진 메시지가 항상 지연이나 손상 없이 버스 접근 권한을 획득하므로, 높은 우선순위의 메시지 전송이 보장됩니다.10

CAN 데이터 프레임은 우선순위를 결정하는 식별자, 데이터 길이 코드(DLC), 그리고 0-8 바이트의 데이터 페이로드로 구성됩니다.10 또한 데이터 요청을 위한 원격 프레임(RTR), 오류 프레임, 오버로드 프레임도 존재합니다.9 CAN은 CRC, 프레임 검사, 비트 모니터링 등 다층적인 오류 감지 메커니즘과, 결함이 있는 노드가 스스로 버스에서 연결을 끊는(Bus-Off 상태) 오류 제한 기능을 포함하고 있어 안전이 중요한 애플리케이션에 매우 높은 신뢰성을 제공합니다.11 최근에는 페이로드 크기를 64바이트로 늘리고 데이터 전송률을 크게 향상시킨 CAN FD(Flexible Data-Rate)가 등장하여 최신 애플리케이션에서도 CAN 프로토콜의 수명을 연장시키고 있습니다.9

DDS와 CAN을 연결하는 근본적인 과제는 단순히 비트와 바이트를 기술적으로 변환하는 것을 넘어, **서로 다른 두 통신 철학 사이의 개념적 변환**을 수행해야 한다는 점에 있습니다. DDS는 미들웨어 계층에서 동작하며, 네트워크를 추상화하고 풍부한 타입과 QoS로 관리되는 '글로벌 데이터 공간'에 초점을 맞춥니다. 반면, CAN은 데이터 링크 및 물리 계층에서 동작하며, 타입이 없고 우선순위에 따라 구동되는 원시적인 메시지 전송을 제공합니다. 따라서 게이트웨이는 저수준의 익명 CAN 프레임 스트림을 받아 DDS가 요구하는 풍부한 컨텍스트, 즉 토픽 이름, 정의된 데이터 타입(IDL), 적절한 QoS 설정을 부여하는 '의미론적 다리(semantic bridge)' 역할을 해야 합니다. 반대로, DDS 데이터 객체 업데이트에서 높은 수준의 컨텍스트를 제거하고, 이를 올바른 우선순위를 가진 단일 CAN 프레임으로 정제하여 적시에 방송해야 합니다. 이러한 개념적 임피던스 불일치(impedance mismatch)가 게이트웨이 설계의 모든 주요 과제의 근원입니다.

**표 1: DDS와 CAN 버스 특성 비교 개요**

| 특성           | DDS (Data Distribution Service)              | CAN (Controller Area Network)                        |
| -------------- | -------------------------------------------- | ---------------------------------------------------- |
| OSI 계층       | 미들웨어 (애플리케이션/표현 계층)            | 데이터 링크/물리 계층                                |
| 통신 패턴      | 발행-구독 (Publish-Subscribe)                | 방송 (Broadcast), 다중 마스터                        |
| 데이터 모델    | 데이터 중심 (Data-Centric), 강력한 타입      | 신호 중심 (Signal-Centric), 타입 없음                |
| 주요 추상화    | 글로벌 데이터 공간 (Global Data Space), 토픽 | 메시지 프레임, 식별자(ID)                            |
| 주소 지정      | 토픽 이름 및 키(Key)                         | 메시지 식별자 (ID)                                   |
| 페이로드 크기  | 가변적 (수 MB까지 가능)                      | 0-8 바이트 (CAN), 0-64 바이트 (CAN FD)               |
| 속도           | 고속 (Gbps 이상 가능, 네트워크에 의존)       | 최대 1 Mbps (CAN), 8 Mbps 이상 (CAN FD)              |
| QoS/우선순위   | 20개 이상의 풍부한 QoS 정책                  | 메시지 ID 기반의 고정 우선순위 중재                  |
| 발견           | 동적, 자동 발견 (플러그 앤 플레이)           | 정적, 노드 구성에 의존                               |
| 주요 사용 사례 | 자율주행, 로보틱스, 항공, 산업 IoT           | 차량 내 네트워크, 산업 자동화, 의료기기              |
| 관리 표준      | OMG (Object Management Group)                | ISO (International Organization for Standardization) |

------



차량의 전기/전자(E/E) 아키텍처는 중대한 패러다임 전환을 겪고 있습니다. 과거의 차량은 수십에서 수백 개의 전문화된 ECU가 다수의 CAN 및 LIN 버스로 연결된 분산 모델을 사용했습니다.12 반면, 현대적인 아키텍처는 ADAS, 인포테인먼트, 차량 제어와 같은 기능들을 강력한 구역 게이트웨이(Zonal Gateway)와 고성능 컴퓨터(HPC)로 통합하고 있습니다.16

이러한 새로운 아키텍처는 '연결성 격차(connectivity gap)'를 야기합니다. HPC는 복잡한 소프트웨어를 실행하기 위해 고대역폭, 확장성, 유연성을 갖춘 통신 백본을 필요로 합니다. DDS는 뛰어난 성능, 풍부한 QoS, 데이터 중심 모델 덕분에 이러한 '데이터 버스'에 이상적인 후보입니다.16 그러나 창문 모터, 휠 속도 센서, 기본적인 파워트레인 제어 등 필수적인 액추에이터와 센서들은 비용 효율성 때문에 예측 가능한 미래에도 계속해서 기존의 CAN 버스에 남아있을 것입니다.5

DDS-CAN 게이트웨이는 바로 이 격차를 해소하는 핵심적인 구성 요소입니다. 게이트웨이는 새로운 HPC 기반의 DDS 세계가 기존의 CAN 세계와 원활하게 통합하고 제어할 수 있도록 합니다. 이는 비용이 많이 들고 위험 부담이 큰 '전면 교체(rip-and-replace)' 방식 대신, 소프트웨어 정의 차량으로 점진적이고 진화적인 전환 경로를 제공하는 전략적 조력자 역할을 합니다.5


DDS-CAN 게이트웨이의 필요성은 자동차 산업에만 국한되지 않습니다.

- **산업 자동화 (인더스트리 4.0):** CANopen이나 DeviceNet과 같은 CAN 기반 프로토콜로 동작하는 기존 공장 설비 및 PLC를 최신 DDS 기반의 SCADA, 모니터링, 예측 유지보수 시스템과 통합하는 데 사용됩니다.13
- **항공우주 및 국방:** 항공기나 차량 내의 CAN 기반 항공전자 및 센서 네트워크를 상위 수준의 DDS 데이터 버스에 연결하여 임무 제어 및 데이터 분석을 수행합니다.1
- **로보틱스:** 중앙의 ROS 2/DDS 기반 로봇 두뇌가 실시간 결정성과 견고성 때문에 CAN을 사용하는 저수준 모터 컨트롤러, 액추에이터, 센서와 통신할 수 있도록 합니다.21

DDS-CAN 게이트웨이는 단순한 기술적 유틸리티를 넘어, **기술 부채를 관리하고 점진적 혁신을 가능하게 하는 핵심적인 비즈니스 및 아키텍처 전략**입니다. 기업들은 신뢰성이 입증되고 비용 효율적인 기존 CAN 기반 하드웨어 및 소프트웨어에 막대한 투자를 해왔습니다.5 자율주행, 커넥티비티, 향상된 UI와 같은 새로운 기능에 대한 시장의 요구는 DDS와 같은 더 강력한 컴퓨팅 패러다임과 통신 미들웨어를 필요로 합니다.17 모든 기존 구성 요소를 전면적으로 교체하는 것은 엄청난 비용과 높은 위험을 수반합니다. 게이트웨이는 '최소 저항 경로'를 제공합니다. 즉, 기존 시스템에 대한 투자를 보존하면서 새로운 기술의 역량을 활용할 수 있게 해줍니다. 이를 통해 기업은 기존의 방대한 CAN 구성 요소 생태계를 활용하면서 차세대 DDS 기반 시스템을 구축할 수 있습니다. 이는 개발 위험을 줄이고 새로운 기능의 시장 출시 시간을 단축시켜, 복잡한 시스템의 진화를 관리하는 강력한 도구가 됩니다.

------



게이트웨이는 CAN 버스와 DDS 글로벌 데이터 공간 사이의 다리 역할을 하며, 그 내부 구조는 세 가지 논리적 계층으로 나눌 수 있습니다.

- **CAN 인터페이스 계층(CAN Interface Layer, CIL):** CAN 버스와의 물리적 연결을 담당합니다. 하드웨어적으로는 CAN 트랜시버와 컨트롤러로 구성되며, 소프트웨어적으로는 Linux SocketCAN이나 공급업체별 API와 같은 드라이버를 사용하여 원시 CAN 프레임을 읽고 씁니다.
- **프로토콜 변환 및 데이터 매핑 엔진(Protocol Translation and Data Mapping Engine, PTDME):** 게이트웨이의 심장부입니다. 이 엔진은 양방향 변환을 수행합니다. CIL로부터 원시 CAN 프레임을 받아 파싱하고, 이를 올바른 DDS 데이터 구조에 매핑하여 DAL로 전달합니다. 반대 방향으로는 DDS 데이터 샘플을 받아 올바른 ID와 페이로드를 가진 CAN 프레임으로 직렬화하여 CIL로 전송합니다.
- **DDS 추상화 계층(DDS Abstraction Layer, DAL):** DDS 도메인 참여자와 DDS 네트워크와 상호작용하는 데 필요한 데이터 작성자 및 데이터 판독기를 포함합니다. QoS 구성, 토픽 생성, 데이터 발행/구독 등 모든 DDS 관련 로직을 처리합니다.


이것은 게이트웨이 설계에서 가장 중요한 작업입니다. 본질적으로 타입이 없는 CAN 메시지를 구조화되고 타입이 있는 표현으로 정의해야 합니다. 표준적인 접근 방식은 OMG의 인터페이스 정의 언어(IDL)를 사용하여 `struct`를 정의하는 것입니다.22

구체적인 IDL 구조를 제안하고 각 필드에 대한 근거를 설명하면 다음과 같습니다.

Code snippet

```
// 일반적인 CAN 프레임을 위한 예시 IDL
module Automotive {
    struct CANFrame {
        unsigned long long timestamp; // epoch 이후 나노초 단위 타임스탬프
        @key unsigned long can_id;    // 11비트 또는 29비트 CAN 식별자
        boolean is_extended_id;       // 표준 ID와 확장 ID 구분
        octet dlc;                    // 데이터 길이 코드 (클래식: 0-8, FD: 최대 64)
        sequence<octet, 64> payload;  // 페이로드, CAN FD에 맞춰 크기 지정
    };
};
```

이러한 데이터 모델링에는 다음과 같은 전략과 장단점이 존재합니다.

- **단순한 접근 방식 (1 CAN ID -> 1 토픽):** 이해하기는 쉽지만 규모가 커지면 실패합니다. 토픽 수가 폭발적으로 증가하여 DDS 발견 메커니즘에 과부하를 주고 시스템을 관리 불가능하게 만듭니다.24

- **정교한 접근 방식 (논리적 그룹화):** 권장되는 전략으로, 관련된 CAN 메시지들을 의미론적으로 연관된 단일 DDS 토픽으로 그룹화합니다. 예를 들어, 'WheelEncoders'라는 토픽을 만들 수 있습니다.

- **구분을 위한 DDS 키 사용:** 단일 토픽 내에서 IDL 구조체의 `can_id` 필드를 `@key`로 지정합니다.2 이를 통해 DDS는 각 

  `can_id`를 'WheelEncoders' 토픽 내의 고유한 *인스턴스*로 취급할 수 있습니다. 구독자는 전체 토픽을 구독하거나 특정 인스턴스(특정 바퀴)만 구독할 수 있습니다. 이 접근 방식은 확장 가능하고 효율적이며, 깔끔한 데이터 중심 모델을 생성합니다.


- **CAN-to-DDS:** 게이트웨이의 CIL이 CAN 프레임을 읽습니다. PTDME는 `CANFrame` IDL 구조체를 채우고, DAL의 데이터 작성자는 이를 적절한 DDS 토픽에 발행합니다. 여기서 타임스탬프는 매우 중요하며, 게이트웨이는 타이밍 정보를 보존하기 위해 하드웨어에서 읽는 시점에 최대한 가깝게 고해상도 타임스탬프를 적용해야 합니다.
- **DDS-to-CAN:** DDS 애플리케이션이 `CANFrame` 데이터 샘플을 발행합니다. 게이트웨이의 데이터 판독기가 이를 수신합니다. PTDME는 `can_id`, `dlc`, `payload`를 추출하여 원시 CAN 프레임을 구성하고, 이를 CIL로 보내 전송합니다. 게이트웨이는 `can_id`에 인코딩된 우선순위를 존중해야 합니다.
- **원격 프레임(RTR) 처리:** CAN RTR 프레임은 데이터 요청입니다. 이는 발행-구독 모델에 직접 매핑되지 않습니다. 견고한 게이트웨이는 이를 DDS-RPC(Request/Reply) 패턴에 매핑할 수 있습니다.1 즉, RTR 프레임이 DDS 요청을 트리거하고, 게이트웨이는 DDS 응답을 기다렸다가 해당 CAN 데이터 프레임을 구성하여 전송합니다.


이는 암시적 동작을 명시적 정책으로 매핑하는 미묘한 과제입니다.

- **우선순위:** CAN 메시지 ID의 암시적 우선순위는 DDS의 `TRANSPORT_PRIORITY` QoS 정책에 매핑될 수 있습니다. 이를 통해 DDS 인프라는 우선순위가 높은 CAN 메시지에서 비롯된 데이터에 우선적인 처리를 제공할 수 있습니다.
- **신뢰성:** CAN은 하드웨어 수준의 재시도 기능이 있지만 근본적으로는 비신뢰성 방송 매체입니다. 반면 DDS는 명시적인 `RELIABILITY` QoS(최선 노력 또는 신뢰성)를 제공합니다. 일반적인 전략은 안전에 중요한 CAN 데이터를 `RELIABLE` DDS 토픽에 매핑하여 IP 기반 네트워크에서의 전달을 보장하고, 중요하지 않은 데이터는 `BEST_EFFORT`를 사용하여 오버헤드를 줄이는 것입니다.
- **결정성 및 지연 시간:** 게이트웨이 자체는 지연 시간을 유발합니다.5 이는 효율적인 코드, 가능한 경우 제로 카피(zero-copy) 기술 26, 그리고 게이트웨이 프로세스를 위한 실시간 운영 체제(RTOS) 스케줄링을 통해 최소화해야 합니다. 목표는 종단 간 지연 시간을 네이티브 CAN보다 약간 높더라도 예측 가능하게 만드는 것입니다.

**표 2: CAN 프레임과 DDS IDL 구조체의 규범적 매핑**

| IDL 필드          | C++ 타입 (생성된 코드) | 설명 및 근거                                                 | CAN 프레임 소스        |
| ----------------- | ---------------------- | ------------------------------------------------------------ | ---------------------- |
| `timestamp`       | `uint64_t`             | 타이밍 정보 보존을 위한 고해상도 타임스탬프. CAN 프레임 수신 시점에 게이트웨이가 생성. | 게이트웨이             |
| `can_id` (`@key`) | `uint32_t`             | 11/29비트 CAN 식별자. 토픽 내 인스턴스를 구분하기 위해 `@key`로 지정. 확장 가능성 확보. | CAN 프레임 헤더        |
| `is_extended_id`  | `bool`                 | 표준(11비트) ID와 확장(29비트) ID를 구분.                    | CAN 프레임 헤더        |
| `is_fd_frame`     | `bool`                 | 클래식 CAN과 CAN FD 프레임을 구분. 페이로드 크기 해석에 필요. | 게이트웨이/드라이버    |
| `dlc`             | `uint8_t`              | 데이터 길이 코드(0-8 또는 0-64). 페이로드의 유효 길이를 나타냄. | CAN 프레임 헤더        |
| `payload`         | `std::vector<uint8_t>` | 최대 64바이트의 데이터 페이로드. `sequence<octet, 64>`는 가변 길이 데이터를 효율적으로 처리. | CAN 프레임 데이터 필드 |

------



Connext Drive는 RTI가 자동차 시장을 겨냥하여 Connext DDS를 기반으로 개발한 주력 제품입니다.17 이 제품의 핵심 가치는 자율주행 및 소프트웨어 정의 차량을 위해 특별히 설계된 완전한 안전 인증 프레임워크라는 점입니다.16 특히 TÜV-SÜD로부터 ISO 26262 ASIL D 인증을 획득한 것은 큰 차별점이며, 이는 시스템 통합업체가 기능 안전 인증에 드는 노력을 크게 줄여줍니다.16

Connext Drive는 AUTOSAR Classic 및 AUTOSAR Adaptive 모두에 대한 전용 통합 툴킷을 제공합니다.16 이는 표준 자동차 소프트웨어 아키텍처와의 통합에 매우 중요합니다. 이 툴킷은 AUTOSAR의 ARXML 형식과 DDS의 IDL 형식 간에 데이터 타입을 자동으로 변환하고 코드를 생성하여, 두 생태계 간의 격차를 해소합니다.16 이는 AUTOSAR Classic으로 관리되는 기존 CAN 기반 ECU를 최신 DDS 데이터 버스에 직접 연결하는 것을 용이하게 합니다.


RTI 라우팅 서비스는 DDS 도메인 간, 그리고 더 중요하게는 DDS와 비-DDS 시스템 간에 데이터를 라우팅하고 변환하기 위한 강력한 독립형 애플리케이션입니다.27 이 서비스는 매우 유연한 플러그인 아키텍처를 기반으로 동작합니다.29

비-DDS 통합의 핵심은 어댑터 SDK(Adapter SDK)입니다.27 이를 통해 개발자는 라우팅 서비스가 CAN과 같은 새로운 프로토콜과 통신하는 방법을 가르치는 맞춤형 '어댑터'를 작성할 수 있습니다. 개발자는 CAN 프레임을 읽고 쓰는 로직을 구현하기만 하면, 라우팅 서비스 엔진이 모든 복잡한 DDS 상호작용, 데이터 라우팅, 변환을 처리합니다.28

또한 라우팅 서비스는 데이터를 즉석에서 변환하는 강력한 내장 기능을 갖추고 있습니다. 토픽 이름을 변경하고, 콘텐츠 기반으로 메시지를 필터링하며, 심지어 서로 다른 IDL 구조체 간에 데이터를 재매핑할 수도 있습니다.27 이는 기존 애플리케이션을 변경하지 않고도 약간 다른 데이터 모델을 가진 시스템을 통합할 때 매우 유용합니다. 

`rticonnextdds-gateway` 프로젝트는 라우팅 서비스 SDK 위에 구축되어 프로토콜 통합 플러그인 개발을 위한 더 정제된 프레임워크와 예제를 제공함으로써 개발 과정을 단순화합니다.29

RTI의 전략은 **다양한 수준의 통합 요구에 부응하는 다층적 솔루션**을 제공하는 것입니다. 자동차 OEM은 턴키 솔루션으로 완벽하게 통합되고 안전 인증을 받은 Connext Drive 제품을 선택할 수 있고, 시스템 개발자는 더 일반적이지만 강력한 라우팅 서비스를 사용하여 고도로 맞춤화된 비자동차용 게이트웨이를 구축할 수 있습니다. 대규모 자동차 OEM의 주요 관심사는 안전 인증(ISO 26262), 표준 준수(AUTOSAR), 개발 위험 감소입니다.16 Connext Drive는 이러한 요구를 직접적으로 해결합니다. 반면, 로보틱스 회사나 산업 통합업체는 AUTOSAR 준수가 필요 없을 수 있지만, 독점적인 CAN 프로토콜을 DDS 백본에 견고하게 연결할 방법이 필요합니다.19 이들에게는 라우팅 서비스와 어댑터 SDK가 완벽한 도구입니다.27 이러한 계층화된 제품 제공은 시장에 대한 깊은 이해를 보여주며, '하나의 크기가 모든 것에 맞는다'는 접근 방식이 아님을 증명합니다.

------



Fast DDS는 ROS 2의 기본 미들웨어로 채택되어 널리 사용되는 오픈소스 C++ DDS 구현체입니다.31 고성능과 풍부한 기능을 자랑합니다. eProsima DDS 라우터는 주로 서로 다른 DDS 네트워크를 연결하기 위해 설계되었습니다. 예를 들어, 로컬 LAN을 WAN에 연결하거나 서로 다른 DDS 도메인을 브리징하는 데 사용됩니다.26 따라서 이 도구는 즉시 사용 가능한 CAN 게이트웨이는 아닙니다.

eProsima 도구를 사용하여 CAN 게이트웨이를 구축하려면, DDS 라우터를 브리지 자체로 사용하는 대신 맞춤형 C++ 애플리케이션을 설계해야 합니다. 이 애플리케이션은 DDS 측을 위해 Fast DDS 라이브러리 31와 CAN 측을 위해 CAN 라이브러리(예: `socketcan`)를 링크하여 게이트웨이 역할을 수행합니다. 필요한 경우, 이 맞춤형 애플리케이션은 더 넓은 네트워크에 연결하기 위해 DDS 라우터 인스턴스와 통신할 수 있습니다.26 eProsima의 고급 도구인 통합 서비스(Integration Service)는 서로 다른 프로토콜 간의 매개변수화된 통신 브리지를 생성하기 위한 라이브러리로, 처음부터 구축하는 것보다 더 적절한 출발점이 될 수 있습니다.37


OpenDDS는 Object Computing에서 개발한 또 다른 주요 오픈소스 C++ DDS 구현체입니다.38 이 역시 모든 기능을 갖추고 있으며 표준을 준수합니다. 구현 전략은 Fast DDS와 유사합니다. 개발자는 OpenDDS 라이브러리 39와 CAN 라이브러리를 링크하는 독립형 C++ 애플리케이션을 만들어야 합니다. 이 방식은 최대의 제어와 유연성을 제공하지만, 스레딩, 데이터 매핑, 오류 처리 등 전체 게이트웨이 로직을 처음부터 작성해야 하므로 가장 많은 개발 노력이 필요합니다.


OPC UA 41, MQTT 42, iceoryx 43와 같은 다른 게이트웨이 프로젝트를 살펴보면 공통적인 아키텍처 패턴을 발견할 수 있습니다. 이들은 종종 YAML이나 TOML 파일과 같은 구성을 통해 매핑 규칙을 정의하고, 플러그인 기반 아키텍처를 채택하며, 핵심 라우팅 로직과 프로토콜별 커넥터를 명확히 분리합니다. 이러한 패턴들은 맞춤형 DDS-CAN 게이트웨이 설계에 직접적으로 적용될 수 있습니다.

**표 3: 상용 및 오픈소스 게이트웨이 솔루션 비교**

| 기준                  | RTI Connext Drive / 라우팅 서비스 | eProsima 기반 맞춤형 게이트웨이 | OpenDDS 기반 맞춤형 게이트웨이  |
| --------------------- | --------------------------------- | ------------------------------- | ------------------------------- |
| 안전 인증 (ISO 26262) | 제공 (ASIL D)                     | 개발자 책임                     | 개발자 책임                     |
| AUTOSAR 통합          | 제공 (툴킷 포함)                  | 없음 (수동 개발 필요)           | 없음 (수동 개발 필요)           |
| 라이선스 모델         | 상용                              | Apache 2.0                      | Apache 2.0 유사                 |
| 즉시 사용 가능 기능   | 높음 (완제품/프레임워크)          | 낮음 (라이브러리 수준)          | 낮음 (라이브러리 수준)          |
| 개발 노력             | 낮음                              | 높음                            | 높음                            |
| 성능                  | 고성능, 최적화됨                  | 구현에 따라 다름                | 구현에 따라 다름                |
| 커뮤니티 및 상용 지원 | 강력한 상용 지원                  | 활발한 커뮤니티, 상용 지원 옵션 | 활발한 커뮤니티, 상용 지원 옵션 |

오픈소스 경로는 자유와 라이선스 비용 절감을 제공하지만, **시스템 통합, 검증, 그리고 가장 중요하게는 안전 인증의 부담을 전적으로 개발자에게 전가**합니다. Fast DDS나 OpenDDS를 사용하면 개발자는 강력하고 표준을 준수하는 DDS 라이브러리를 무료로 얻을 수 있습니다.32 그러나 이것들은 완전한 게이트웨이 솔루션이 아닌 라이브러리일 뿐입니다. 개발자는 복잡한 데이터 매핑, QoS 변환, 오류 처리 로직을 포함한 전체 게이트웨이 애플리케이션을 직접 설계, 구현, 테스트해야 합니다. 차량과 같은 안전이 중요한 시스템의 경우, 이렇게 맞춤 제작된 게이트웨이는 매우 시간과 비용이 많이 드는 ISO 26262 인증 과정을 거쳐야 합니다. 반면 RTI Connext Drive와 같은 상용 솔루션은 이미 이 작업을 완료하고 인증 산출물을 제공합니다.16 따라서 오픈소스의 '무료'는 라이선스에 한정됩니다. 인증된 양산 등급 시스템을 오픈소스 구성 요소로 구축하는 총 소유 비용(TCO)은 상용의 사전 인증된 솔루션의 라이선스 비용보다 잠재적으로 훨씬 더 높을 수 있습니다. 이는 단순한 기능 비교를 넘어서는 중요한 재무적, 전략적 판단을 요구합니다.

------



- **과제:** 게이트웨이는 중개자이며, 모든 처리 단계는 지연 시간을 추가합니다.25 이는 CAN 네트워크에서 기대하는 실시간 보증을 위협할 수 있습니다. 지연 시간의 원인으로는 OS 스케줄링, 계층 간 데이터 복사, 프로토콜 직렬화/역직렬화 등이 있습니다.
- **완화 전략:**
  - **제로 카피(Zero-Copy):** 데이터 복사를 최소화하거나 제거하도록 게이트웨이를 설계합니다. PTDME는 가능한 한 데이터 버퍼에 대한 포인터나 참조를 사용하여 작업해야 합니다.26
  - **RTOS:** 예측 가능한 스레드 스케줄링과 제한된 실행 시간을 보장하기 위해 실시간 운영 체제(RTOS)에 게이트웨이 소프트웨어를 배포합니다.
  - **효율적인 스레딩:** 시간적으로 민감한 CAN I/O를 처리하는 전용 고우선순위 스레드와 DDS 통신 및 기타 비핵심 작업을 처리하는 저우선순위 스레드를 사용하는 다중 스레드 아키텍처를 채택합니다.


- **과제:** 변환 과정에서 데이터의 의미가 손실될 수 있습니다. 여기에는 엔디언(바이트 순서), CAN 신호 데이터베이스(예: DBC 파일)와 DDS IDL 간의 데이터 타입 표현 차이, 정밀한 타이밍 정보 손실 등이 포함됩니다.
- **완화 전략:**
  - **타임스탬핑:** 게이트웨이의 CIL은 시간적 관계를 보존하기 위해 수신 즉시 들어오는 CAN 프레임에 고해상도 타임스탬프를 적용해야 합니다. 이 타임스탬프는 DDS IDL 구조체의 핵심 부분이 되어야 합니다.
  - **신호 패킹/언패킹:** PTDME는 DBC 파일과 같은 표준 정의에 따라 CAN 페이로드에서 비트 단위로 패킹된 신호를 정확하게 파싱하고, 이를 DDS 샘플의 올바른 타입(float, integer)으로 변환하는 견고한 로직을 포함해야 합니다. 이 과정에서 엔디언을 올바르게 처리해야 합니다.


- **과제:** 두 네트워크는 서로 다른 장애 모드를 가집니다. CAN 버스는 'Bus-Off' 상태에 들어갈 수 있고, DDS 참여자는 'not alive' 상태가 될 수 있습니다. 이러한 상태들은 시스템 수준의 인식을 유지하기 위해 브리지 전체에 전파되어야 합니다.5
- **완화 전략:**
  - **CAN 오류를 DDS로:** 게이트웨이의 CIL은 CAN 컨트롤러의 상태를 모니터링해야 합니다. Bus-Off나 높은 오류율이 감지되면, 게이트웨이는 이를 DDS 세계에 신호로 알려야 합니다. 이는 자체 `LIVELINESS` QoS 상태를 `NOT_ALIVE`로 변경하거나 전용 `GatewayStatus` 토픽을 발행하여 수행할 수 있습니다.
  - **DDS 활성을 CAN으로:** 게이트웨이가 중요한 DDS 발행자(예: 중앙 ADAS 컴퓨터)와의 활성 연결을 잃으면, 특정 CAN 메시지 전송을 중단하거나 CAN 버스의 액추에이터에 '안전 상태' 명령을 보내도록 구성할 수 있습니다.


- **과제:** 실제 차량에는 수천 개의 고유한 CAN 메시지가 있을 수 있습니다. 각각에 대한 매핑 로직을 하드코딩하는 것은 불가능합니다. 게이트웨이는 동적으로 구성 가능해야 합니다.24
- **완화 전략:**
  - **구성 기반 매핑:** 게이트웨이는 외부 구성 파일(예: XML, YAML)에서 매핑 규칙을 로드해야 합니다. 이 파일은 어떤 CAN ID가 어떤 DDS 토픽 및 키에 매핑되는지 정의하고 데이터 변환을 지정합니다. 이를 통해 코드를 다시 컴파일하지 않고도 게이트웨이의 동작을 업데이트할 수 있습니다. RTI의 라우팅 서비스는 이 점에서 뛰어납니다.27
  - **토픽/콘텐츠 필터링:** DDS의 내장 필터링 기능을 사용합니다. 게이트웨이는 콘텐츠 필터링 토픽이나 허용/거부 목록을 사용하여 5 필요한 데이터만 구독하고 전달함으로써 자체 처리 부하와 전체 네트워크 트래픽을 줄일 수 있습니다.

------



이 섹션은 DDS-CAN 브리지를 차세대 차량 네트워킹의 더 넓은 맥락에서 조명합니다. 주요 대안은 브리징이 아니라 CAN ECU를 오토모티브 이더넷으로 마이그레이션하고 SOME/IP와 같은 프로토콜을 사용하는 것입니다.44

- **SOME/IP:** AUTOSAR 프레임워크 내에서 자동차용으로 특별히 설계된 서비스 지향 미들웨어입니다.46 DDS보다 더 객체 기반이며 덜 동적입니다. 통신은 특정 클라이언트와 서비스 인스턴스 간에 긴밀하게 연결됩니다.
- **DDS:** 더 넓은 실시간 시스템에서 유래한 데이터 중심 미들웨어입니다. 완전한 피어-투-피어 방식이며, 매우 동적이고 훨씬 더 풍부한 QoS 정책을 특징으로 합니다.46

이 둘 사이의 선택은 어느 것이 '더 낫다'의 문제가 아니라, 어떤 아키텍처 목표가 최우선인지에 따라 결정됩니다. 순수한 AUTOSAR 호환 차량 내 서비스 네트워크가 목표라면 SOME/IP가 자연스러운 선택입니다. 반면, 기존 시스템을 통합하고 잠재적으로 차량 외부의 비자동차 시스템(V2X, 클라우드)과 연결해야 하는 매우 유연한 데이터 중심 아키텍처가 목표라면 DDS와 게이트웨이 접근 방식이 더 우수합니다. DDS-CAN 게이트웨이는 기존 CAN 자산을 활용하여 새로운 기능을 신속하게 추가해야 하는 복잡한 프로젝트에 최적의 솔루션이며, 이는 프로젝트의 시간 및 복잡성 제약 조건 하에서 전략적 우위를 점합니다.


미래의 차량은 단일 통신 프로토콜을 사용하지 않을 것입니다. CAN/CAN FD, LIN, FlexRay, 오토모티브 이더넷, 무선 기술이 공존하는 이기종 네트워크의 네트워크가 될 것입니다.45 이러한 미래에는 지능형 다중 프로토콜 게이트웨이의 역할이 더욱 중요해집니다. 게이트웨이는 서로 다른 프로토콜 도메인 간에 데이터를 변환하고 라우팅하여 전체 차량이 단일의 응집력 있는 시스템으로 기능하도록 보장하는 중앙 신경계 역할을 할 것입니다.

------



솔루션을 선택하기 전에 아키텍트는 다음과 같은 핵심 질문에 답해야 합니다.

- 안전 요구사항(ISO 26262 등급)은 무엇인가?
- AUTOSAR 준수는 필수적인가?
- 팀의 전문 기술은 무엇인가 (임베디드 C vs. 최신 C++/미들웨어)?
- 라이선스 비용과 내부 개발 및 검증 비용의 예산은 얼마인가?
- 요구되는 시장 출시 시간은 언제인가?

이 프레임워크는 표 3의 논리를 따라 정보에 입각한 선택을 하는 데 도움이 될 것입니다.


보고서에서 제시된 핵심 권장 사항을 실행 가능한 목록으로 요약하면 다음과 같습니다.

1. **모델 우선:** 코드를 작성하기 전에 데이터 모델(IDL)과 매핑 전략(CAN ID 대 토픽/키)을 먼저 정의하십시오.
2. **논리적 그룹화:** 1:1 CAN ID-토픽 매핑을 피하고, 확장성을 위해 논리적 그룹화와 DDS 키를 사용하십시오.
3. **QoS 우선순위 지정:** CAN 우선순위와 신뢰성 요구사항을 의식적으로 DDS QoS 정책에 매핑하고, 기본값에 의존하지 마십시오.
4. **오류 전파:** 네트워크 상태(Bus-Off, Liveliness)를 게이트웨이 전체에 전파하는 견고한 로직을 구현하십시오.
5. **하드코딩 대신 구성:** 유지보수성을 보장하기 위해 매핑 규칙에 외부 구성 파일을 사용하십시오.
6. **타임스탬프 유의:** 가능한 한 하드웨어에 가깝게 타이밍 정보를 캡처하고 보존하십시오.
7. **프로파일링 및 최적화:** 실시간 제약을 충족시키기 위해 게이트웨이 지연 시간을 지속적으로 측정하고 최적화하십시오.


DDS-CAN 브리지는 단순한 연결 장치 그 이상입니다. 이는 차세대 소프트웨어 정의 차량 및 기타 복잡한 분산 시스템의 실용적이고 진화적인 개발을 가능하게 하는 정교하고 전략적인 구성 요소입니다. 검증된 기존 기술과 현대적인 데이터 중심 미들웨어의 강력한 기능을 융합시켜, 미래 혁신을 향한 실용적인 경로를 제공합니다.


1. Data Distribution Service - Wikipedia, accessed July 5, 2025, https://en.wikipedia.org/wiki/Data_Distribution_Service
2. What is DDS? - DDS Foundation, accessed July 5, 2025, https://www.dds-foundation.org/what-is-dds-3/
3. What Is DDS? - MATLAB & Simulink - MathWorks, accessed July 5, 2025, https://www.mathworks.com/help/dds/gs/dds-conceptual-overview.html
4. 1. Foundations - The Data Distribution Service Tutorial, accessed July 5, 2025, https://download.zettascale.online/www/docs/Vortex/html/ospl/DDSTutorial/foundations.html
5. Bridging XMPP and DDS Messaging Frameworks | Object Computing, Inc., accessed July 5, 2025, https://objectcomputing.com/resources/publications/sett/april-2012-bridging-xmpp-and-dds-messaging-frameworks
6. The Data Distribution Service Tutorial - ResearchGate, accessed July 5, 2025, https://www.researchgate.net/profile/Angelo-Corsaro/publication/273136749_The_Data_Distribution_Service_Tutorial/links/54f971d20cf28d6deca482a7/The-Data-Distribution-Service-Tutorial.pdf
7. Tutorial: Create a Data Distribution Service for Real-Time Systems application in Rhapsody Developer for C++ - IBM, accessed July 5, 2025, https://www.ibm.com/docs/en/engineering-lifecycle-management-suite/design-rhapsody/10.0.1?topic=tutorials-create-data-distribution-service-real-time-systems-application
8. What can DDS do for You? - Object Management Group, accessed July 5, 2025, https://www.omg.org/hot-topics/documents/dds/CoreDX_DDS_Why_Use_DDS.pdf
9. What Is Can Bus (Controller Area Network) | Dewesoft, accessed July 5, 2025, https://dewesoft.com/blog/what-is-can-bus
10. CAN Bus Uncovered: Basics and Applications in Vehicles - EMQX, accessed July 5, 2025, https://www.emqx.com/en/blog/can-bus-how-it-works-pros-and-cons
11. What is CAN Bus? A Comprehensive Guide to the Backbone of Modern Automotive Systems, accessed July 5, 2025, https://www.sintrones.com/application/what-is-can-bus/
12. Controller Area Network (CAN) Protocol Overview - NI, accessed July 5, 2025, https://www.ni.com/en/shop/seamlessly-connect-to-third-party-devices-and-supervisory-system/controller-area-network--can--overview.html
13. GBP: How Did CAN Bus Revolutionize The Automotive Industry? - Sital Technology - Databus Communication Solutions, accessed July 5, 2025, https://sitaltech.com/gbp-how-did-can-bus-revolutionize-the-automotive-industry/
14. What is CAN Bus and how does it work? - Actisense, accessed July 5, 2025, https://actisense.com/news/what-is-can-bus-and-how-does-it-work/
15. A Guide to Controller Area Network (CAN) for Industrial Applications | Electromate Inc, accessed July 5, 2025, https://www.electromate.com/news/post/a-guide-to-controller-area-network-can-for-industrial-applications
16. RTI Connext Drive - Autonomous Vehicle Development | RTI, accessed July 5, 2025, https://www.rti.com/products/connext-drive
17. RTI Unveils Connext Drive: The First Complete Automotive-Grade Connectivity Solution for Autonomous Vehicle Development - GlobeNewswire, accessed July 5, 2025, https://www.globenewswire.com/news-release/2019/11/13/1946301/0/en/RTI-Unveils-Connext-Drive-The-First-Complete-Automotive-Grade-Connectivity-Solution-for-Autonomous-Vehicle-Development.html
18. What Is DDS In Automotive? - Talking Tech Trends - YouTube, accessed July 5, 2025, https://www.youtube.com/watch?v=QnFd_vm_9Xo
19. Why CAN Bus is Still Relevant: Understanding Its Use in Industrial and Automotive Applications - Coolgear, accessed July 5, 2025, https://www.coolgear.com/guides-informative-articles/why-can-bus-is-still-relevant-understanding-its-use-in-industrial-and-automotive-applications.html
20. RTI Connext Drive Demo - YouTube, accessed July 5, 2025, https://www.youtube.com/watch?v=rLLEzKVHO5g
21. Can Protocol vs Ros2 : r/robotics - Reddit, accessed July 5, 2025, https://www.reddit.com/r/robotics/comments/1be6sl2/can_protocol_vs_ros2/
22. Data-Centric Programming Best Practices: - RTI, accessed July 5, 2025, https://www.rti.com/hubfs/docs/DDS_Best_Practices_WP.pdf
23. A Comparison and Mapping of Data Distribution Service and High-Level Architecture - Real-Time Innovations, accessed July 5, 2025, https://info.rti.com/hubfs/whitepapers/Comparison_and_Mapping_of_DDS_and_HLA.pdf
24. Performance Evaluation of ROS2-DDS middleware implementations facilitating Cooperative Driving in Autonomous Vehicle. - arXiv, accessed July 5, 2025, https://arxiv.org/html/2412.07485v1
25. Virtualizing DDS middleware: Performance challenges and measurements - ResearchGate, accessed July 5, 2025, https://www.researchgate.net/publication/249992354_Virtualizing_DDS_middleware_Performance_challenges_and_measurements
26. eProsima DDS Router, accessed July 5, 2025, https://www.eprosima.com/middleware/tools/dds-router
27. RTI Routing Service - RTI Community, accessed July 5, 2025, https://community.rti.com/static/documentation/connext-dds/4.5e/RTI_Routing_Service_4.5e/doc/pdf/RTI_Routing_Service_GettingStarted.pdf
28. Bridging Domains: Using the RTI Routing Service Adapter to Connect MSQL Databases, accessed July 5, 2025, https://www.rti.com/blog/msql-database-routing-service-adapter
29. 1. Introduction - RTI Connext Gateway 7.3.0 documentation, accessed July 5, 2025, https://community.rti.com/static/documentation/gateway/current/introduction.html
30. rticommunity/rticonnextdds-gateway: The RTI Gateway is a software component that allows integrating different types of connectivity protocols with DDS. Integration in this context means that data flows from different protocols are adapted to interface with DDS, establishing communication from and to DDS. - GitHub, accessed July 5, 2025, https://github.com/rticommunity/rticonnextdds-gateway
31. Fast DDS - eProsima, accessed July 5, 2025, https://fast-dds.docs.eprosima.com/
32. eProsima/Fast-DDS: The most complete DDS - Proven: Plenty of success cases. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 5, 2025, https://github.com/eProsima/Fast-DDS
33. Bridge Remote DDS Networks With a DDS Router | Husarnet, accessed July 5, 2025, https://husarnet.com/blog/ros2-dds-router/
34. 1. Project Overview - 3.2.0 - DDS Router, accessed July 5, 2025, https://eprosima-dds-router.readthedocs.io/en/latest/rst/getting_started/project_overview.html
35. eProsima/DDS-Router: The DDS Router is an application developed by eProsima that allows, using Fast DDS, to communicate by DDS protocol different networks. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 5, 2025, https://github.com/eProsima/DDS-Router
36. WAN & Cloud Tools - eProsima, accessed July 5, 2025, https://www.eprosima.com/middleware/tools/wan-cloud
37. eProsima documentation index - all-docs 1.0 documentation, accessed July 5, 2025, https://docs.eprosima.com/
38. OpenDDS, accessed July 5, 2025, https://opendds.org/
39. OpenDDS is an open source C++ implementation of the Object Management Group (OMG) Data Distribution Service (DDS). OpenDDS also supports Java bindings through JNI. - GitHub, accessed July 5, 2025, https://github.com/OpenDDS/OpenDDS
40. Getting Started with the OpenDDS Project | Object Computing, Inc., accessed July 5, 2025, https://objectcomputing.com/how-we-serve/accelerators/opendds/resources/getting-started-with-opendds
41. rticommunity/rticonnextdds-gateway-opcua: The RTI OPC UA/DDS Gateway implements a standards-based transparent bridge to integrate OPC UA and DDS applications. - GitHub, accessed July 5, 2025, https://github.com/rticommunity/rticonnextdds-gateway-opcua
42. vogler75/automation-gateway: A OPC UA gateway which gives you access to your OPC UA values via MQTT or GraphQL (HTTP). If you have an OPC UA server in your PLC, or a SCADA system with an OPC UA server, you can query data from there via MQTT and GraphQL (HTTP) - GitHub, accessed July 5, 2025, https://github.com/vogler75/automation-gateway
43. eclipse-iceoryx/iceoryx-gateway-dds - GitHub, accessed July 5, 2025, https://github.com/eclipse-iceoryx/iceoryx-gateway-dds
44. SOME/IP - A Service Oriented Architecture (Intrepid Tech Day '19) - YouTube, accessed July 5, 2025, https://m.youtube.com/watch?v=T0K9oGaZJoY&pp=ygUMI2lwZGlzY292ZXJ5
45. Automotive IoT Ethernet-Based Communication Technologies ..., accessed July 5, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9460104/
46. data distribution service - What's the difference between DDS and ..., accessed July 5, 2025, https://stackoverflow.com/questions/51182471/whats-the-difference-between-dds-and-some-ip


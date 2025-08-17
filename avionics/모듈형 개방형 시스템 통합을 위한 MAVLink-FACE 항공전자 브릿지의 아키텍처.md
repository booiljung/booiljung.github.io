# 모듈형 개방형 시스템 통합을 위한 MAVLink-FACE 항공전자 브릿지의 아키텍처

## 서론

### 0.1 통합의 전략적 필요성

현대 국방 획득 환경은 수명 주기 비용을 절감하고 새로운 역량을 신속하게 전장화하기 위한 모듈형 개방형 시스템 접근법(Modular Open Systems Approach, MOSA)의 도입을 전략적 최우선 과제로 삼고 있습니다.1 이러한 패러다임 전환의 중심에는 미군 항공 플랫폼을 위한 핵심 전략으로 자리 잡은 FACE™(Future Airborne Capability Environment) 기술 표준이 있습니다.3 FACE 표준은 상호 운용성과 소프트웨어 재사용성을 극대화하여 특정 공급업체에 대한 종속성을 탈피하고, 국방 시스템의 개발 및 현대화 속도를 가속화하는 것을 목표로 합니다.

### 0.2 상용 UAS 기술의 부상

이와 동시에, 상업 및 오픈소스 무인 항공 시스템(Unmanned Aircraft System, UAS) 생태계에서는 폭발적인 기술 혁신이 이루어지고 있습니다. 이 분야에서 MAVLink(Micro Air Vehicle Link) 프로토콜은 기체와 지상 통제소(Ground Control Station, GCS) 간의 통신을 위한 사실상의 표준(de facto standard)으로 부상했습니다.6 MAVLink는 경량성, 확장성, 그리고 방대한 오픈소스 커뮤니티의 지원을 바탕으로 드론 기술 발전을 견인하고 있습니다.

### 0.3 핵심 조력자로서의 브릿지

본 보고서에서 다루는 "MAVLink-FACE 브릿지"는 이 두 개의 이질적인 세계를 연결하는 핵심적인 소프트웨어 구성 요소입니다. 이 브릿지는 단순한 프로토콜 변환기를 넘어, FACE 표준을 준수하는 시스템이 MAVLink 기반 플랫폼의 역량을 활용할 수 있도록 하는 복잡한 미들웨어입니다. 이 브릿지는 신속한 프로토타이핑, 혁신적인 상용 기성품(COTS) 및 오픈소스 소프트웨어(OSS) 센서와 플랫폼의 통합, 그리고 궁극적으로 MOSA가 추구하는 비용 절감 목표 달성을 위한 핵심 조력자로 기능합니다.1

### 0.4 논지

본 보고서는 MAVLink-FACE 브릿지가 기술적으로 실현 가능하고 전략적으로 가치가 높지만, 그 성공적인 구현은 두 표준 간의 상당한 의미론적 및 보증 격차를 극복하기 위한 엄격하고 모델 기반의 데이터 및 보안 아키텍처 접근법에 달려 있음을 주장합니다. 이러한 브릿지의 설계와 구현은 단순한 기술적 과제를 넘어, 현대 국방 항공전자 시스템의 미래를 형성하는 중요한 전략적 노력의 일환입니다.

## 1.  MAVLink 프로토콜: 무인 시스템의 사실상 표준

MAVLink 프로토콜은 무인 항공기(UAV), 지상 로봇, 수상정 등 다양한 무인 시스템에서 통신을 위해 광범위하게 사용되는 경량 메시징 프로토콜입니다. 2009년 로렌츠 마이어(Lorenz Meier)에 의해 처음 발표된 이후, ArduPilot 및 PX4와 같은 주요 오픈소스 자동조종장치 프로젝트의 핵심 통신 프로토콜로 채택되면서 사실상의 표준으로 자리 잡았습니다.7 MAVLink의 성공은 자원이 제한된 임베디드 시스템에 최적화된 설계 철학과 개발자 커뮤니티의 요구를 수용하는 유연성에 기인합니다.

### 1.1  설계 철학 및 주요 특징

MAVLink의 설계는 효율성, 확장성, 그리고 양방향 통신이라는 세 가지 핵심 원칙에 기반합니다.

- **경량성 및 효율성:** MAVLink는 마이크로컨트롤러와 같이 계산 능력과 메모리가 제한된 시스템을 위해 설계된 헤더 전용 메시지 마샬링 라이브러리입니다.6 데이터 구조를 이진 형식(stream of bytes)으로 직렬화하는 방식을 채택하여, JSON이나 XML과 같은 텍스트 기반 형식에 비해 통신 오버헤드를 최소화합니다.9 이러한 특징 덕분에 MAVLink는 대역폭이 낮고 신뢰성이 떨어지는 시리얼 통신이나 무선 주파수(RF) 링크에서도 안정적인 데이터 교환을 보장합니다.10

- **확장성:** MAVLink 프로토콜의 가장 큰 강점 중 하나는 XML 메시지 정의 파일을 통한 높은 확장성입니다.6 개발자들은 

  `common.xml`이나 각 비행 스택에 특화된 `ardupilot.xml`과 같은 파일에 새로운 메시지 유형을 정의하여 맞춤형 기능을 쉽게 추가할 수 있습니다.11 이러한 유연성은 빠르게 변화하는 드론 시장의 다양한 요구사항을 수용하고, 새로운 센서나 액추에이터를 시스템에 신속하게 통합할 수 있게 하는 원동력이 되었습니다.

- **양방향 통신:** MAVLink는 기체에서 GCS로 상태 정보를 전송하는 원격측정(telemetry) 메시지와, GCS에서 기체로 명령을 보내는 커맨드 메시지를 모두 지원하여 완전한 양방향 통신을 구현합니다.6

  `ATTITUDE`나 `GLOBAL_POSITION_INT`와 같은 메시지는 기체의 자세와 위치 정보를 실시간으로 전달하며, `COMMAND_LONG` 메시지는 이륙, 착륙, 비행 모드 변경과 같은 다양한 명령을 수행하는 데 사용됩니다. `HEARTBEAT` 메시지는 통신 링크의 활성 상태를 지속적으로 확인하는 역할을 합니다.11

MAVLink의 확장성은 상업 및 취미용 드론 분야에서 빠른 혁신을 가능하게 한 원동력이었습니다. 개발자들은 특정 페이로드나 기능을 위해 `방언(dialect)`이라 불리는 맞춤형 메시지를 자유롭게 생성할 수 있었습니다.6 그러나 이러한 유연성은 FACE와 같이 고도로 표준화된 환경과의 통합 시에는 상당한 도전 과제를 야기합니다. FACE 브릿지는 단순히 

`common.xml`에 정의된 표준 메시지 세트만을 처리하도록 설계될 수 없습니다.14 특정 기체가 사용하는 맞춤형 방언을 만났을 때 브릿지는 제대로 기능하지 못할 수 있습니다. 따라서 MAVLink-FACE 브릿지는 정적인 '일회성' 구현이 아니라, 고도로 구성 가능한(configurable) 구성 요소로 설계되어야 합니다. 브릿지의 데이터 매핑 로직은 핵심 처리 로직과 분리되어, 통합 담당자가 통합 대상 기체에 해당하는 'MAVLink 방언 프로파일'을 로드하여 사용할 수 있도록 해야 합니다. 이는 브릿지를 단순한 프로토콜 변환기에서 '데이터 모델 중재 엔진'으로 변모시킵니다. 결과적으로 브릿지의 구성 관리는 코드 자체만큼이나 중요해지며, FACE 환경에 새로운 MAVLink 시스템을 통합할 때마다 해당 시스템에 맞는 브릿지 구성 파일을 검증하고 인증해야 하는 추가적인 복잡성과 비용을 발생시킵니다.

### 1.2  MAVLink 패킷 아키텍처 (v1.0 vs. v2.0)

MAVLink 패킷은 헤더, 페이로드, 체크섬으로 구성된 명확한 구조를 가집니다. 시간이 지나면서 기능 개선을 위해 v2.0이 도입되었으며, 두 버전은 하위 호환성을 유지합니다.

- **MAVLink v1.0 구조:** v1.0 패킷은 최대 263바이트 크기를 가집니다. 패킷의 시작을 알리는 `Start-of-Frame` (0xFE) 바이트로 시작하여, 페이로드 길이, 시퀀스 번호, 시스템 ID, 컴포넌트 ID, 메시지 ID, 최대 255바이트의 페이로드, 그리고 데이터 무결성을 검증하기 위한 2바이트 CRC 체크섬으로 구성됩니다.7
- **MAVLink v2.0 향상점:** v2.0은 v1.0의 기능을 확장하면서 하위 호환성을 유지합니다. `Start-of-Frame`은 0xFD로 변경되었고, 호환성 플래그, 3바이트로 확장된 메시지 ID(255개 이상의 메시지 지원 가능), 그리고 메시지 인증을 위한 선택적 13바이트 서명(signature) 필드가 추가되었습니다.7 또한, 페이로드 끝부분의 0으로 채워진 바이트들을 전송하지 않는 페이로드 잘라내기(payload truncation) 기능을 지원하여 대역폭을 절약할 수 있습니다.15 v2.0 패킷의 최대 크기는 280바이트입니다.
- **주요 패킷 필드 설명:**
  - `System ID (sysid)`: 메시지를 보내는 시스템을 식별합니다. 일반적으로 기체는 1, GCS는 255와 같은 값을 사용합니다.7
  - `Component ID (compid)`: 동일 시스템 내에서 메시지를 보내는 구성 요소를 식별합니다. 예를 들어, 자동조종장치는 1, 동반 컴퓨터(companion computer)는 191과 같은 값을 가질 수 있습니다.7
  - `Message ID (msgid)`: 페이로드의 의미와 구조를 정의하는 고유 식별자입니다.7
  - `CRC (Cyclic Redundancy Check)`: 메시지 무결성을 보장합니다. 각 메시지 정의마다 고유한 `CRC_EXTRA` 시드(seed) 값을 체크섬 계산에 포함하여, 송신자와 수신자가 동일한 메시지 구조를 사용하고 있는지 확인하는 추가적인 검증 계층을 제공합니다.7

### 1.3  MAVLink 메시지 생태계

MAVLink의 핵심은 메시지 중심의 통신 생태계입니다. 이 생태계는 XML 정의, 코드 생성기, 그리고 마이크로서비스라는 개념을 통해 구축됩니다.

- **메시지 정의 (XML):** 모든 MAVLink 메시지는 XML 파일에 그 구조와 필드가 정의됩니다. 이 XML 파일은 MAVLink 코드 생성기에 의해 처리되어 C, Python, Java 등 다양한 프로그래밍 언어에서 사용할 수 있는 라이브러리를 자동으로 생성합니다.7 이 라이브러리는 메시지의 직렬화 및 역직렬화 과정을 처리해주므로 개발자는 저수준의 패킷 처리 작업에서 벗어나 애플리케이션 로직에 집중할 수 있습니다.
- **주요 메시지:**
  - `HEARTBEAT (#0)`: MAVLink 네트워크의 모든 구성 요소가 약 1Hz 주기로 전송하는 기본적인 "생존 신호" 메시지입니다. 이 메시지를 통해 각 구성 요소는 자신의 존재, 유형, 상태를 알립니다. GCS가 일정 시간 동안 `HEARTBEAT` 메시지를 수신하지 못하면 통신 두절로 간주하고 안전 모드(failsafe)를 발동시킬 수 있습니다.11
  - `GLOBAL_POSITION_INT (#33)`: WGS84 좌표계 기준의 위도, 경도(정수형), 해발 고도(AMSL) 및 상대 고도, 그리고 속도 벡터 정보를 제공하는 핵심적인 원격측정 메시지입니다.13
  - `ATTITUDE (#30)`: 기체의 자세 정보(롤, 피치, 요)를 라디안 단위로 제공합니다.13
  - `COMMAND_LONG (#76)`: 이륙, 착륙, 비행 모드 변경, 파라미터 설정 등 다양한 작업을 수행하기 위한 범용 명령 메시지입니다. 7개의 파라미터 필드를 통해 유연한 명령 전달이 가능합니다.13
- **마이크로서비스:** MAVLink는 단순 메시지 교환을 넘어, 여러 메시지를 조합하여 더 높은 수준의 프로토콜을 구현하는데 이를 마이크로서비스라고 합니다. 대표적인 예로, 신뢰성 있는 명령 전달을 보장하는 '커맨드 프로토콜'이 있습니다. 이 프로토콜은 `COMMAND_LONG` 메시지를 전송한 후, 수신 측으로부터 `COMMAND_ACK` 메시지를 기다리며, 응답이 없을 경우 자동으로 명령을 재전송하는 기능을 포함합니다.12

### 1.4  내재된 한계 및 보안 태세

MAVLink는 자원이 제한된 환경에서의 효율성을 최우선으로 설계되었기 때문에, 보안 기능은 부차적으로 고려되었습니다. 이는 현대적인 네트워크 환경에서 몇 가지 중요한 한계를 드러냅니다.

- **네이티브 암호화 부재:** MAVLink는 기본적으로 메시지를 암호화하지 않고 평문(plaintext)으로 전송합니다. 따라서 무선 통신 구간에서 패킷 스니핑(packet sniffing) 공격에 취약하며, 공격자는 기체의 위치나 상태와 같은 민감한 정보를 쉽게 엿볼 수 있습니다.6
- **스푸핑 및 재전송 공격에 대한 취약성:** 적절한 보안 조치가 없다면 공격자는 악의적인 `COMMAND_LONG` 메시지를 주입(spoofing)하거나, 이전에 캡처한 유효한 명령을 재전송(replay)하여 기체의 제어권을 탈취할 수 있습니다.6
- **부가 기능으로서의 인증:** MAVLink v2.0에서 도입된 서명 기능은 송신자의 신원을 확인하는 인증(authentication)은 제공하지만, 데이터의 내용을 숨기는 기밀성(confidentiality)은 보장하지 않습니다. 또한 이 기능은 선택 사항이며, 모든 시스템에서 구현되어 있지는 않습니다.6
- **구현 의존성:** MAVLink 표준은 메시지의 형식만을 정의할 뿐, 특정 메시지를 수신했을 때 기체가 어떻게 동작할지는 전적으로 ArduPilot이나 PX4와 같은 비행 스택 소프트웨어의 구현에 달려 있습니다.19 따라서 문법적으로 완벽하게 유효한 MAVLink 메시지라 할지라도, 수신 측 소프트웨어가 해당 메시지를 처리하도록 구현되어 있지 않다면 무시되거나 예기치 않은 동작을 유발할 수 있습니다.

## 2.  FACE™ 기술 표준: 군용 항공전자의 필수 요건

FACE™(Future Airborne Capability Environment) 기술 표준은 미 국방부(DoD)가 추진하는 모듈형 개방형 시스템 접근법(MOSA)의 핵심 요소로, 군용 항공전자 시스템의 개발, 획득, 유지보수 방식을 근본적으로 혁신하기 위해 탄생했습니다. 이는 비용 절감, 신기술의 신속한 전장 적용, 그리고 특정 공급업체에 대한 종속성 탈피라는 전략적 목표를 달성하기 위한 필수 요건으로 간주됩니다.

### 2.1  MOSA와 FACE의 탄생

- **정부 주도의 표준화:** FACE 표준은 공급업체 종속(vendor lock-in) 문제를 해결하고, 비반복적 공학 비용(non-recurring engineering costs)을 절감하며, 새로운 전투 능력을 신속하게 현장에 배치하기 위한 미 국방부의 강력한 의지에서 비롯되었습니다.1 이는 각 무기 체계마다 독립적으로 개발되던 소프트웨어를 재사용 가능하고 상호 운용 가능한 모듈로 전환하려는 MOSA 전략의 구체적인 실현 방안입니다.
- **표준들의 표준 (Standard of Standards):** FACE는 완전히 새로운 표준을 창조한 것이 아니라, 기존에 검증된 여러 개방형 표준들을 조합하고 특정 목적에 맞게 프로파일링한 '표준들의 표준'입니다.22 예를 들어, 운영체제 서비스를 위해서는 POSIX를, 파티셔닝을 위해서는 ARINC 653을, 그리고 인터페이스 정의를 위해서는 OMG IDL(Interface Definition Language)과 같은 산업 표준을 기반으로 하여 공통 운영 환경(Common Operating Environment)을 구축합니다.23

### 2.2  FACE 참조 아키텍처: 5개 세그먼트 모델

FACE 아키텍처의 핵심은 소프트웨어를 기능에 따라 논리적인 그룹으로 나누는 5개의 세그먼트 모델입니다. 각 세그먼트 사이에는 표준화된 인터페이스가 정의되어 있어, 소프트웨어 컴포넌트의 이식성(portability)과 상호 운용성(interoperability)을 보장합니다.22

- **5개 세그먼트:**
  1. **운영체제 세그먼트 (Operating System Segment, OSS):** 프로세스 관리, 메모리 할당 등과 같은 기본적인 운영체제 서비스를 제공합니다. 일반적으로 LynxOS-178이나 VxWorks와 같이 특정 FACE 프로파일(예: Safety, Security)을 준수하는 실시간 운영체제(RTOS)가 이 세그먼트에 해당합니다.24
  2. **I/O 서비스 세그먼트 (I/O Services Segment, IOSS):** 물리적 하드웨어 장치(예: 시리얼 포트, 이더넷 카드)에 대한 인터페이스를 정규화하여, 상위 계층 소프트웨어가 특정 장치 드라이버의 세부 사항을 알 필요 없이 I/O를 사용할 수 있도록 추상화합니다.22
  3. **플랫폼 특화 서비스 세그먼트 (Platform-Specific Services Segment, PSSS):** 특정 항공기 플랫폼에 종속적인 서비스(예: 데이터 로깅, 상태 관리, ARINC 661 기반 그래픽 렌더링)를 제공합니다.3
  4. **이식 가능 컴포넌트 세그먼트 (Portable Components Segment, PCS):** 시스템의 핵심적인 비즈니스 로직과 기능이 위치하는 '애플리케이션 계층'입니다. PCS 컴포넌트는 특정 하드웨어나 운영체제에 독립적으로 설계되어, 여러 다른 FACE 시스템 간에 높은 재사용성을 갖도록 하는 것을 목표로 합니다.22
  5. **전송 서비스 세그먼트 (Transport Services Segment, TSS):** 본 보고서의 핵심 관심사 중 하나로, PCS와 PSSS의 컴포넌트들이 서로 데이터를 교환할 수 있도록 통신 서비스를 제공하는 역할을 합니다.22

### 2.3  심층 분석: 전송 서비스 세그먼트 (TSS)

TSS는 FACE 아키텍처 내에서 데이터 유통의 중추적인 역할을 담당하며, 컴포넌트 이식성을 극대화하기 위한 핵심적인 추상화 계층을 제공합니다.

- **목적:** TSS의 주된 목표는 애플리케이션 컴포넌트(PCS/PSSS)로부터 실제 통신 메커니즘(예: 공유 메모리, 네트워크 소켓, DDS)을 추상화하는 것입니다.22 이는 PCS 컴포넌트가 데이터가 지점 A에서 지점 B로 '어떻게' 전달되는지 알 필요 없이, 단지 전달된다는 사실에만 의존하여 개발될 수 있음을 의미합니다.
- **인터페이스 정의:** TSS는 표준화된 전송 서비스 인터페이스(Transport Services Interface, TSI)를 외부에 노출합니다. 애플리케이션은 이 API를 통해 데이터를 송수신하며, 교환되는 데이터의 구조는 공식적인 데이터 모델에 의해 엄격하게 정의됩니다.23
- **이식성 확보:** TSS는 전송 방식을 추상화함으로써, 한 시스템(예: 이더넷 기반 DDS)에서 개발된 PCS 컴포넌트를 다른 전송 방식(예: 맞춤형 PCIe 백플레인)을 사용하는 시스템으로 이전할 때 PCS 컴포넌트 자체의 코드는 거의 또는 전혀 변경할 필요가 없게 만듭니다. 필요한 변경은 TSS 구현체에 국한됩니다.26

이 지점에서 중요한 점은 TSS가 다른 컴포넌트의 이식성을 가능하게 하는 *조력자*이지만, TSS 컴포넌트 *자체*는 반드시 이식 가능하지 않을 수 있다는 것입니다.26 이는 특정 플랫폼의 성능 목표를 달성하기 위해 TSS 구현이 해당 플랫폼의 전송 메커니즘(예: 특정 RTOS의 고속 메시지 큐)에 고도로 최적화될 수 있기 때문입니다. 표준화된 것은 TSS로의 

*인터페이스*이지, 그 인터페이스 뒤의 *구현*이 아닙니다. 따라서 시스템 통합 시, 목표 하드웨어와 OS에 적합한 TSS를 선택하고, 애플리케이션 컴포넌트들은 표준 TSS API에 맞춰 개발함으로써 이식성을 확보하게 됩니다. 이는 MAVLink-FACE 브릿지 설계에 직접적인 영향을 미칩니다. 브릿지 컴포넌트는 이식 가능한 PCS 컴포넌트로 설계될 수 있으며, 표준 TSS API를 사용하여 번역된 MAVLink 데이터를 발행합니다. 기반이 되는 TSS(예: DDS 기반 또는 ZeroMQ 기반)의 선택은 통합 단계의 결정 사항이 되며, 브릿지 컴포넌트 자체는 이러한 다양한 TSS 구현에 걸쳐 이식성을 유지할 수 있습니다.

### 2.4  FACE 데이터 아키텍처: 명확한 통신 강제

FACE 표준은 진정한 상호 운용성을 위해서는 단순히 함수 시그니처가 일치하는 구문적(syntactic) 상호 운용성을 넘어, 교환되는 데이터에 대한 공유되고 명확한 이해, 즉 의미론적(semantic) 상호 운용성이 필수적임을 인식하고 있습니다.28

- **모호성의 문제:** 예를 들어, 한 시스템이 '고도'를 피트 단위의 16비트 정수로 보내고 다른 시스템은 미터 단위의 32비트 부동소수점으로 받는다면, 두 시스템은 통신할 수 없습니다. FACE 데이터 아키텍처는 이러한 모호성을 제거하기 위해 설계되었습니다.
- **3계층 모델:** FACE 데이터 아키텍처는 데이터를 정의하기 위해 다계층 접근법을 사용합니다 30:
  1. **개념 모델 (Conceptual Model):** 측정 방식과 무관하게 '위치', '온도'와 같은 추상적인 개념 또는 '관찰 가능 항목(observable)'을 정의합니다.
  2. **논리 모델 (Logical Model):** 개념 모델에서 정의된 관찰 가능 항목이 어떻게 측정되는지를 명시합니다. 예를 들어, '위치'는 'WGS84' 좌표계에서 '도(degree)' 단위로 측정된다고 정의합니다.
  3. **플랫폼 모델 (Platform Model):** '32비트 부동소수점', '64비트 정수'와 같이 실제 전송되는 데이터의 구체적인 '온-더-와이어(on-the-wire)' 표현을 정의합니다.
- **주요 산출물:**
  - **공유 데이터 모델 (Shared Data Model, SDM):** FACE 컨소시엄이 제공하는 공통적이고 재사용 가능한 데이터 모델로, 기본적인 개념과 타입을 정의합니다. 모든 맞춤형 데이터 모델의 출발점이 됩니다.31
  - **이식성 단위 제공 모델 (Unit of Portability Supplied Model, USM):** 소프트웨어 컴포넌트(적합성 단위, Unit of Conformance, UoC) 개발자가 제공하는 데이터 모델입니다. 이 모델은 SDM을 기반으로 하여 해당 컴포넌트가 송수신하는 특정 데이터를 정의하며, 시스템 통합에 있어 매우 중요한 역할을 합니다.23
- **인터페이스 정의 언어 (IDL):** 컴포넌트 간의 인터페이스는 OMG의 인터페이스 정의 언어(IDL)를 사용하여 공식적으로 정의됩니다. IDL은 프로그래밍 언어에 독립적인 방식으로 데이터 구조와 오퍼레이션을 명시하는 방법을 제공하여, 서로 다른 언어로 작성된 컴포G넌트 간의 상호 운용성을 보장합니다.23

## 3.  MAVLink-FACE 브릿지의 아키텍처 설계

MAVLink-FACE 브릿지는 단순히 프로토콜을 변환하는 게이트웨이를 넘어, 두 이질적인 시스템 간의 의미론적, 보안적, 그리고 운영적 격차를 해소하는 복잡한 미들웨어 구성 요소입니다. 이 브릿지의 성공적인 설계는 FACE 참조 아키텍처 내에서의 명확한 역할 정의와 기능적으로 분리된 내부 아키텍처에 달려 있습니다.

### 3.1  브릿지의 역할 및 경계 정의

브릿지는 다음과 같은 핵심적인 책임을 수행하는 능동적인 구성 요소로 정의되어야 합니다.

- **프로토콜 종단 (Protocol Termination):** MAVLink 프로토콜 스택을 완전히 종단 처리합니다. 이는 시리얼 또는 UDP 통신을 처리하고, 패킷의 시작과 끝을 식별하며, CRC 체크섬을 검증하고, 패킷을 파싱하는 모든 과정을 포함합니다.36

- **의미론적 변환 (Semantic Translation):** MAVLink 메시지의 데이터를 FACE 데이터 모델에 정의된 구조로, 또는 그 반대로 매핑합니다. 이는 본 보고서의 섹션 4에서 자세히 다룰 핵심 과제입니다.

- **상태 관리 (State Management):** `HEARTBEAT` 메시지 수신 여부에 따라 MAVLink 시스템과의 연결 상태를 유지하고 관리합니다.17 또한, 미션 아이템 전송 시 

  `WAYPOINT_ACK`와 같은 응답 메시지를 추적하여 신뢰성 있는 명령 전달을 보장하는 상태 기계를 포함할 수 있습니다.37

- **보안 강제 (Security Enforcement):** 외부 MAVLink 네트워크와 내부 FACE 시스템 사이의 보안 경계 역할을 하며, 정의된 보안 정책을 강제하는 지점(Policy Enforcement Point)으로 기능합니다 (섹션 5에서 상세히 논의).

이러한 브릿지의 설계는 MAVLink 메시지를 다른 표준(예: STANAG 4586)으로 변환하는 기존 브릿지 시스템이나 8, MAVProxy 및 mavlink-router와 같은 소프트웨어 라우터의 아키텍처에서 영감을 얻을 수 있습니다.36 이들은 서로 다른 목적을 가지지만, 메시지 라우팅, 변환, 상태 관리라는 유사한 기능을 수행합니다.

### 3.2  FACE 참조 아키텍처 내 배치

브릿지를 FACE 참조 아키텍처의 어느 세그먼트에 배치할 것인가는 중요한 설계 결정 사항입니다. 세 가지 주요 옵션을 고려할 수 있습니다.

- **옵션 A: I/O 서비스 세그먼트 (IOSS) 컴포넌트:**
  - *근거:* MAVLink 통신이 특정 시리얼 포트나 이더넷 인터페이스와 같은 물리적 I/O 장치로 간주될 경우 이 배치가 타당합니다. 브릿지는 이 '장치'로부터 들어오는 데이터 스트림을 정규화하여 표준 I/O 서비스 인터페이스를 통해 시스템의 나머지 부분에 제공합니다.
  - *장점:* 하드웨어에 특화된 통신 로직을 시스템의 나머지 부분과 명확하게 분리할 수 있습니다.
  - *단점:* 단순한 드라이버와 자체 로직을 가진 복잡한 프로토콜 변환기 사이의 경계를 모호하게 만들 수 있습니다.
- **옵션 B: 플랫폼 특화 서비스 세그먼트 (PSSS) 컴포넌트:**
  - *근거:* 브릿지가 OSS를 통해서는 접근할 수 없는 다른 플랫폼 서비스(예: 특정 로깅 서비스, 고정밀 타임 소스)에 접근해야 할 경우 PSSS에 배치하는 것이 합리적입니다.
  - *장점:* 더 풍부한 플랫폼 수준의 서비스에 접근할 수 있습니다.
  - *단점:* 브릿지 컴포넌트 자체의 이식성을 저하시킵니다.
- **옵션 C (권장): 이식 가능 컴포넌트 세그먼트 (PCS) 컴포넌트:**
  - *근거:* 이는 가장 유연하고 바람직한 접근 방식입니다. 저수준의 I/O(시리얼/UDP)는 단순한 IOSS 컴포넌트가 처리하여 원시 데이터 스트림을 PCS에 위치한 브릿지로 전달합니다. PCS 컴포넌트인 브릿지는 모든 복잡한 파싱 및 데이터 모델 매핑 로직을 포함합니다.
  - *장점:* 브릿지 로직의 이식성을 극대화합니다. 동일한 브릿지 컴포넌트를 기반 IOSS 드라이버만 교체하여 다른 하드웨어 플랫폼에서 재사용할 수 있습니다. 이는 이식 가능한 애플리케이션 로직을 강조하는 FACE 철학과 완벽하게 일치합니다.
  - *단점:* 초기에는 IOSS 드라이버와 PCS 브릿지라는 두 개의 컴포넌트로 구성되어 더 복잡해 보일 수 있지만, 아키텍처적으로는 훨씬 더 깔끔하고 유지보수가 용이합니다.

### 3.3  브릿지의 제안 기능 아키텍처 (PCS 컴포넌트 기준)

PCS 컴포넌트로 설계된 브릿지의 내부 아키텍처는 FPGA 기반 인터페이스 설계에서 영감을 받아 37, 다음과 같은 모듈형 계층 구조로 제안될 수 있습니다.

- **계층 1: 전송 적응 계층 (Transport Adaptation Layer):**
  - IOSS와 상호 작용하여 원시 바이트 스트림(시리얼/UDP로부터)을 수신하고 송신합니다. 이 계층은 물리적 전송 매체에 대해 아는 유일한 부분입니다.
- **계층 2: MAVLink 프로토콜 엔진 (MAVLink Protocol Engine):**
  - **메시지 파서/역직렬화기:** 적응 계층으로부터 바이트 스트림을 수신합니다. 시작 플래그(0xFE/0xFD)를 사용하여 패킷 경계를 식별하고, CRC를 검증하며, 메시지 ID에 따라 페이로드를 내부 MAVLink 메시지 구조체로 역직렬화합니다.7
  - **메시지 직렬화기/포맷터:** 내부 MAVLink 메시지 구조체를 받아 바이트 스트림으로 직렬화하고, CRC를 계산한 후, 전송을 위해 적응 계층으로 전달합니다.37
- **계층 3: 데이터 모델 매퍼 및 상태 기계 (Data Model Mapper & State Machine):**
  - **MAVLink-to-FACE 매퍼:** 브릿지의 핵심 로직입니다. 파서로부터 MAVLink 메시지를 수신하면, 이 컴포넌트는 (구성 파일에서) 해당 매핑 규칙을 찾아 MAVLink 데이터를 FACE USM에 정의된 구조체로 변환합니다. 이 과정에는 단위 변환, 좌표계 변환, 열거형 매핑 등이 포함됩니다.
  - **FACE-to-MAVLink 매퍼:** FACE 시스템으로부터 (FACE USM 구조체 형태로) 수신된 명령을 적절한 MAVLink `COMMAND_LONG` 또는 다른 메시지로 변환합니다.
  - **상태 관리:** `HEARTBEAT` 메시지를 기반으로 연결 상태를 추적합니다. 또한, 미션 아이템 시퀀스를 관리하고, `WAYPOINT_ACK`와 같은 응답을 추적하여 신뢰성 있는 명령 전달을 보장할 수 있습니다. 이는 37에서 설명된 비행 상태 관리자(Flight Status Manager)의 역할과 유사합니다.
- **계층 4: FACE TSS 인터페이스 계층 (FACE TSS Interface Layer):**
  - 표준화된 IDL 정의 API를 사용하여 시스템의 TSS와 상호 작용합니다. 변환된 FACE 데이터 구조체를 발행(publish)하고, MAVLink 기체로 향하는 명령을 수신하기 위해 FACE 토픽을 구독(subscribe)합니다.

이러한 아키텍처는 브릿지를 '아키텍처 방화벽(Architectural Firewall)'으로 기능하게 합니다. MAVLink는 유연하지만 인증 관점에서는 '규율이 없는' 프로토콜인 반면 6, FACE는 엄격하고 구조화되어 있으며 인증을 위해 설계되었습니다.3 이 두 도메인 사이에 브릿지를 배치함으로써, 브릿지는 단순한 변환기 이상의 역할을 수행합니다. 즉, 인증되고 결정론적인 FACE 컴퓨팅 환경을 예측 불가능하고 인증되지 않은 MAVLink 기반 외부 시스템으로부터 격리하는 '방화벽' 역할을 합니다. 이는 브릿지 설계가 격리(isolation)와 봉쇄(containment)를 최우선으로 해야 함을 의미합니다. MAVLink 측에서 들어오는 모든 데이터는 신뢰할 수 없는 것으로 간주되어야 합니다. 브릿지는 FACE 데이터 모델로 매핑하기 전에 모든 수신 데이터에 대한 유효성 검사(범위 검사, 온전성 검사)를 수행해야 하며, 잘못된 형식이나 악의적인 MAVLink 패킷이 번역된 형태로라도 FACE TSS로 전파되는 것을 절대로 허용해서는 안 됩니다. 이는 브릿지 자체의 인증 전략에 직접적인 영향을 미칩니다. 브릿지는 인증이 필요한 중요한 구성 요소가 되며, 인증 논거는 브릿지가 '신뢰할 수 없는' MAVLink 도메인을 효과적으로 정화하고 봉쇄하여, 나머지 FACE 시스템이 TSS를 통해 브릿지로부터 수신한 모든 데이터가 유효하고 안전하다고 가정할 수 있게 한다는 점에 초점을 맞추게 될 것입니다. 브릿지는 이 신뢰 변환의 모든 부담을 지게 됩니다.

## 4.  통합의 핵심: 데이터 모델 매핑

MAVLink와 FACE의 통합에서 가장 근본적이고 어려운 과제는 두 표준 간의 '의미론적 임피던스 불일치(semantic impedance mismatch)'를 해결하는 것입니다. 이는 MAVLink의 경량적이고 암시적인 데이터 표현 방식과 FACE의 풍부하고 명시적인 데이터 아키텍처 사이의 본질적인 차이에서 비롯됩니다. 이 문제를 해결하는 것이 브릿지의 핵심 기능이며, 시스템의 상호 운용성과 신뢰성을 결정짓는 중요한 요소입니다.

### 4.1  의미론적 임피던스 불일치

이 불일치는 데이터의 의미를 표현하는 방식의 차이에서 발생합니다.

- **MAVLink의 암시적 의미론:** MAVLink의 `GLOBAL_POSITION_INT` 메시지는 위도를 `int32_t` 타입으로 전송합니다.13 이 정수 값이 실제로는 도(degree) 단위에 

  107을 곱한 값이며, WGS84 데이텀(datum)을 기준으로 한다는 사실은 메시지 자체에는 포함되어 있지 않고, MAVLink 문서나 XML 정의 파일에 암시적으로 정의되어 있습니다.15 수신 측은 이러한 '외부 정보'를 미리 알고 있어야만 데이터를 올바르게 해석할 수 있습니다.

- **FACE의 명시적 의미론:** 반면, FACE 데이터 모델은 동일한 정보를 표현하기 위해 명시적인 데이터 타입, 단위(예: `degrees`), 정밀도(10−7), 그리고 좌표 참조 프레임(WGS84)을 모두 데이터 모델 내에 명확하게 정의할 것을 요구합니다.28 모든 의미 정보가 데이터 모델 자체에 내장되어 있어, 해석의 모호성을 원천적으로 제거합니다.

- **브릿지의 역할:** MAVLink-FACE 브릿지의 핵심 역할은 바로 이 불일치를 해소하는 것입니다. 브릿지는 MAVLink로부터 수신한 암시적인 데이터에 필요한 의미론적 메타데이터(단위, 좌표계 등)를 추가하여, FACE 데이터 아키텍처가 요구하는 명시적인 형태로 변환하는 작업을 수행해야 합니다.

### 4.2  모델 기반 매핑 전략

이러한 복잡한 변환 로직을 소스 코드에 직접 하드코딩하는 것은 유지보수성과 확장성 측면에서 바람직하지 않습니다. 견고한 브릿지는 변환 규칙을 외부의, 사람이 읽고 수정하기 쉬운 구성 파일(예: XML 또는 YAML)에 정의하는 모델 기반 접근법을 채택해야 합니다.

이 구성 파일은 특정 MAVLink `msgid`와 그 필드들을 특정 FACE USM 메시지 타입과 그 필드들로 직접 매핑하는 규칙을 정의합니다. 여기에는 필요한 변환 함수(예: `int32_to_float_with_scaling`, `enum_to_string`)에 대한 명시도 포함될 수 있습니다. 이 접근법은 핵심 브릿지 코드를 재컴파일하지 않고도 새로운 MAVLink 방언이나 변경된 FACE 데이터 모델에 유연하게 적응할 수 있게 해줍니다.

### 4.3  사례 연구 1: 항법 데이터 매핑

원격측정 데이터의 핵심인 항법 정보 매핑은 브릿지의 가장 기본적인 기능입니다. 아래 표는 MAVLink의 `GLOBAL_POSITION_INT` 메시지를 가상의 FACE `NavigationSolution` 데이터 모델 요소로 변환하는 과정을 필드별로 상세히 보여줍니다. 이 표는 추상적인 이론을 구체적인 구현으로 전환하는 데 필수적인 실용적 예시를 제공합니다. 이는 단순해 보이는 데이터 전송 과정에 단위 변환, 데이터 타입 변경, 명시적 메타데이터 할당 등 복잡한 작업이 포함됨을 명확히 보여주며, 브릿지를 통해 연결해야 하는 모든 MAVLink 메시지에 대해 이와 같은 상세한 분석이 필요함을 강조합니다.

| MAVLink 필드 (`GLOBAL_POSITION_INT`, #33) | MAVLink 타입 및 단위 | 예시 값     | 변환 로직        | FACE 필드 (`NavigationSolution`) | FACE 타입 및 메타데이터               | 예시 값      |
| ----------------------------------------- | -------------------- | ----------- | ---------------- | -------------------------------- | ------------------------------------- | ------------ |
| `lat`                                     | `int32_t` (degE7)    | `473977210` | `value / 1.0e7`  | `latitude`                       | `float`, Units: `deg`, Datum: `WGS84` | `47.3977210` |
| `lon`                                     | `int32_t` (degE7)    | `85455939`  | `value / 1.0e7`  | `longitude`                      | `float`, Units: `deg`, Datum: `WGS84` | `8.5455939`  |
| `alt`                                     | `int32_t` (mm)       | `530000`    | `value / 1000.0` | `altitudeAMSL`                   | `float`, Units: `m`, Ref: `MSL`       | `530.0`      |
| `relative_alt`                            | `int32_t` (mm)       | `10000`     | `value / 1000.0` | `altitudeAGL`                    | `float`, Units: `m`, Ref: `Ground`    | `10.0`       |
| `vx`                                      | `int16_t` (cm/s)     | `500`       | `value / 100.0`  | `velocityNorth`                  | `float`, Units: `m/s`, Frame: `NED`   | `5.0`        |
| `vy`                                      | `int16_t` (cm/s)     | `100`       | `value / 100.0`  | `velocityEast`                   | `float`, Units: `m/s`, Frame: `NED`   | `1.0`        |
| `vz`                                      | `int16_t` (cm/s)     | `-50`       | `value / 100.0`  | `velocityDown`                   | `float`, Units: `m/s`, Frame: `NED`   | `-0.5`       |

### 4.4  사례 연구 2: 상태 및 열거형 매핑

상태 정보 매핑은 또 다른 종류의 도전 과제를 제시합니다. MAVLink는 비트마스크(bitmask)와 열거형(enumeration)을 사용하여 작은 페이로드에 많은 상태 정보를 압축하지만, FACE 시스템은 명확하고 해석이 용이한 개별 상태 값을 요구합니다. 아래 표는 MAVLink의 `HEARTBEAT` 메시지를 가상의 FACE `ComponentStatus` 데이터 모델 요소로 변환하는 과정을 보여줍니다. 이 표는 항공전자 시스템에서 필수적인 상태 모니터링 기능을 구축하는 데 있어 중요한 매핑 문제를 다룹니다. MAVLink의 단일 정수 값이 어떻게 FACE에서 여러 개의 불리언(boolean) 또는 열거형 필드로 확장되는지를 보여줌으로써, FACE 시스템에서 견고한 상태 모니터링을 구현하는 방법을 제시합니다. 이는 브릿지의 매핑 로직이 단순한 타입 변환을 넘어 비트 연산과 룩업 테이블(lookup table)을 포함해야 함을 시사하며, 설계 및 검증의 복잡성을 한층 더합니다.

| MAVLink 필드 (`HEARTBEAT`, #0) | MAVLink 타입 및 열거형    | 예시 값                           | 변환 로직                       | FACE 필드 (`ComponentStatus`) | FACE 타입 및 메타데이터                                      | 예시 값       |
| ------------------------------ | ------------------------- | --------------------------------- | ------------------------------- | ----------------------------- | ------------------------------------------------------------ | ------------- |
| `type`                         | `uint8_t` (MAV_TYPE)      | `2` (MAV_TYPE_QUADROTOR)          | `MAV_TYPE` 열거형에서 조회      | `componentType`               | `string`                                                     | `"Quadrotor"` |
| `autopilot`                    | `uint8_t` (MAV_AUTOPILOT) | `3` (MAV_AUTOPILOT_ARDUPILOTMEGA) | `MAV_AUTOPILOT` 열거형에서 조회 | `autopilotSoftware`           | `string`                                                     | `"ArduPilot"` |
| `system_status`                | `uint8_t` (MAV_STATE)     | `4` (MAV_STATE_ACTIVE)            | `MAV_STATE` 열거형에서 조회     | `systemState`                 | `Enum {Uninit, Boot, Calibrating, Standby, Active, Critical, Emergency}` | `Active`      |
| `mavlink_version`              | `uint8_t`                 | `3`                               | 직접 복사                       | `mavlinkVersion`              | `uint8`                                                      | `3`           |

## 5.  보안 및 인증 고려사항

상용 오픈소스 프로토콜인 MAVLink를 고도의 신뢰성이 요구되는 군용 항공전자 시스템에 통합하는 것은 심각한 보안 및 인증 문제를 야기합니다. MAVLink-FACE 브릿지는 이러한 문제를 해결하기 위한 핵심적인 역할을 수행해야 하며, 단순한 데이터 변환기를 넘어 보안 게이트웨이이자 인증의 핵심 요소로 설계되어야 합니다.

### 5.1  보안 게이트웨이로서의 브릿지

브릿지는 MAVLink 네트워크가 기본적으로 안전하지 않다는 가정 하에 설계되어야 합니다. 이는 도청, 메시지 위변조, 재전송 공격 등 다양한 위협에 노출되어 있음을 의미합니다.6 이러한 위협에 대응하기 위해 브릿지는 제로 트러스트 아키텍처(Zero Trust Architecture, ZTA) 원칙을 적용하는 이상적인 지점이 됩니다.39

- **제로 트러스트 원칙 적용:**
  - **침해 가정 (Assume Breach):** 브릿지는 연결된 MAVLink 네트워크 전체가 이미 침해되었다고 가정하고 동작해야 합니다. 외부 시스템에 대한 어떠한 암묵적인 신뢰도 부여해서는 안 됩니다.
  - **명시적 검증 (Verify Explicitly):** 수신되는 모든 MAVLink 메시지는 가능한 경우 MAVLink v2 서명 기능을 통해 인증되어야 하며, 페이로드 데이터의 유효성에 대한 엄격한 검증(예: 보고된 GPS 위치가 물리적으로 가능한 범위 내에 있는지 확인)이 이루어져야 합니다.
  - **최소 권한 접근 (Least Privilege Access):** 브릿지는 임무 수행에 필요한 특정 MAVLink 메시지만을 처리하도록 구성되어야 하며, 그 외의 모든 메시지는 폐기해야 합니다. 마찬가지로, 인가된 특정 FACE 토픽에만 데이터를 발행하도록 접근 제어가 이루어져야 합니다.
- **정책 집행 지점 (Policy Enforcement Point, PEP)으로서의 역할:** 브릿지는 보안 정책을 강제하는 역할을 수행할 수 있습니다. 예를 들어, 특정 비행 단계에서 안전하지 않다고 판단되는 `COMMAND_LONG` 메시지가 기체로 전달되는 것을 차단하거나, 비정상적인 빈도로 데이터가 수신될 경우 경고를 발생시키는 등의 규칙을 적용할 수 있습니다.

### 5.2  혼합 임계 시스템의 인증 과제

군용 항공전자 시스템은 DO-178C와 같은 엄격한 표준에 따라 소프트웨어의 안전성을 인증받아야 합니다.41 그러나 MAVLink 및 이를 사용하는 오픈소스 비행 스택은 이러한 엄격한 개발 프로세스를 따르지 않습니다. 이 둘의 통합은 인증되지 않은 구성 요소와 인증된 구성 요소가 상호 작용하는 '혼합 임계 시스템(Mixed-Criticality System)'을 만들어냅니다.41

- **핵심 문제:** 혼합 임계 시스템의 인증은 매우 복잡한 과제입니다. 인증 기관은 공유된 프로세서에서 실행되는 모든 소프트웨어가 그중 가장 높은 임계 등급(criticality level)의 기능과 동일한 수준으로 인증받을 것을 요구할 수 있습니다. 이를 '가장 높은 임계 등급으로 시스템 전체를 칠하는(coloring)' 문제라고 하며, 이는 엄청난 비용을 초래할 수 있습니다.2
- **파티셔닝의 역할:** FACE 표준이 ARINC 653 개념을 활용하여 지원하는 파티셔닝(partitioning)은 이 문제에 대한 핵심적인 해결책을 제공합니다.24 브릿지와 이 브릿지가 대표하는 MAVLink 기반 시스템은 높은 임계 등급의 파티션과 분리된 별도의 파티션에 격리될 수 있습니다. 이들 간의 상호 작용은 엄격하게 통제되고 제한됩니다.
- **브릿지의 인증 부담:** 이러한 아키텍처에서 '아키텍처 방화벽' 역할을 하는 브릿지 소프트웨어 자체는 높은 설계 보증 수준(Design Assurance Level, DAL)으로 인증받아야 할 가능성이 높습니다. 인증 논거는 브릿지가 MAVLink 측의 장애나 악의적인 행위가 높은 임계 등급의 파티션으로 전파되어 시스템을 손상시키는 것을 강력하게 방지하고 데이터를 효과적으로 격리 및 정화할 수 있다는 능력에 초점을 맞추게 될 것입니다.44

결론적으로, FACE 시스템은 MAVLink 기체를 본질적으로 신뢰할 수 없지만 6, 임무 수행을 위해 데이터를 수신하고 명령을 보내야 하는 모순적인 상황에 놓입니다. 이 신뢰 관계는 어딘가에서 설정되어야 하며, 브릿지는 그 논리적이고 유일한 지점입니다. FACE 시스템은 브릿지를 신뢰하고, 브릿지는 신뢰할 수 없는 MAVLink 링크를 관리할 책임을 집니다. 이는 브릿지의 역할을 단순한 변환기 이상으로 격상시켜, 외부 UAS 통합과 관련된 시스템의 '신뢰 컴퓨팅 기반(Trusted Computing Base, TCB)'의 일부로 만듭니다. 따라서 브릿지의 설계, 구현, 검증은 다른 어떤 안전 필수(safety-critical) 구성 요소와 마찬가지로 엄격해야 합니다. 브릿지 개발팀은 단순히 프로토콜 전문가일 뿐만 아니라, DO-178C와 같은 안전 필수 소프트웨어 개발, 보안 공학, 정형 검증 방법에 대한 전문가여야 합니다. 브릿지 개발의 비용과 일정은 프로토콜 변환이라는 기능적 요구사항보다 이러한 보증 요구사항에 의해 더 크게 좌우될 것입니다.

## 6.  전략적 분석 및 권장사항

MAVLink-FACE 브릿지를 통한 통합은 단순한 기술적 구현을 넘어, 국방 항공전자 시스템의 획득 및 개발 패러다임을 바꿀 수 있는 전략적 의미를 가집니다. 그러나 이 접근법의 잠재력을 완전히 실현하기 위해서는 그 이점과 내재된 위험을 명확히 이해하고, 체계적인 통합 전략을 수립해야 합니다.

### 6.1  MAVLink-FACE 통합 패턴의 이점

- **신속한 역량 전개:** 국방 프로그램이 방대한 상용 드론 시장에서 개발된 혁신적인 역량(예: 새로운 센서, 첨단 자율 비행 알고리즘)을 군용 플랫폼에 신속하게 통합하고 테스트할 수 있게 합니다.1 이는 기술 발전 속도가 빠른 상업 분야의 성과를 국방 분야에 효과적으로 도입하는 통로가 됩니다.
- **개발 비용 절감:** 일반적인 UAS 기능에 대해 '바퀴를 다시 발명'할 필요 없이, 오픈소스 커뮤니티의 막대한 투자와 혁신을 활용할 수 있습니다.1 이는 특히 예산이 제한된 탐색 개발이나 신속 프로토타이핑 프로젝트에서 큰 이점을 제공합니다.
- **상호 운용성 향상:** 단일 FACE 표준 준수 GCS가 군용 등급의 항공기와 상용 MAVLink 기반 드론으로 구성된 이기종(heterogeneous) 편대를 잠재적으로 통제할 수 있게 합니다. 이는 작전의 유연성을 크게 향상시킬 수 있습니다.

### 6.2  단점 및 위험

- **복잡성:** 브릿지는 상당한 보안 및 인증 부담을 지는 복잡한 소프트웨어입니다. 그 개발은 결코 사소한 엔지니어링 노력이 아니며, 높은 수준의 전문성을 요구합니다.
- **성능 오버헤드:** IOSS에서 PCS 브릿지를 거쳐 TSS로 이어지는 다계층 아키텍처는 필연적으로 지연 시간(latency)을 발생시킵니다. MAVLink의 이진 직렬화는 매우 효율적이며 9, 변환 과정에서 추가적인 처리 시간이 소요됩니다. 특히 고주파 제어 루프와 같이 시간에 민감한 애플리케이션의 경우, 이 오버헤드를 신중하게 분석하고 관리해야 합니다.
- **유지보수 및 구성 관리:** 브릿지는 특정 MAVLink 방언과 FACE 데이터 모델에 밀접하게 연결되어 있습니다. 두 표준이 발전함에 따라 브릿지도 함께 유지보수 및 업데이트되어야 하므로, 수명 주기 동안 지속적인 구성 관리의 어려움을 야기할 수 있습니다. 이는 브릿지가 정적 구성 요소가 아니라 지속적으로 관리되어야 하는 동적 자산임을 의미합니다.

### 6.3  시스템 통합자를 위한 권장사항

MAVLink-FACE 브릿지의 성공적인 구현을 위해 시스템 통합자는 다음과 같은 전략적 권장사항을 따라야 합니다.

- **1. 모델 기반 시스템 엔지니어링(MBSE) 접근법 채택:** Ansys SCADE 3 또는 다른 UML/SysML 기반 도구를 사용하여 브릿지, 인터페이스, 데이터 모델 매핑을 모델링해야 합니다. 이는 복잡성을 관리하고, 요구사항을 추적하며, 인증에 필요한 산출물을 생성하는 데 필수적입니다.
- **2. 데이터 모델 우선순위 부여:** FACE 데이터 모델, 특히 브릿지를 위한 USM은 프로젝트에서 가장 중요한 단일 산출물입니다. 개발 프로세스 초기에 이를 개발하고 검증해야 합니다. 공식적인 FACE 공유 데이터 모델(SDM)을 출발점으로 활용하여 일관성과 재사용성을 확보해야 합니다.31
- **3. 초기 단계부터 보안 설계 (Security by Design):** 브릿지를 제로 트러스트 보안 게이트웨이로 구현해야 합니다. 보안을 나중에 추가하는 기능으로 취급해서는 안 됩니다. 브릿지는 MAVLink 네트워크로부터의 위협에 대한 1차 방어선이 되어야 합니다.
- **4. 이식성을 위한 아키텍처 설계 (PCS):** 핵심 브릿지 로직을 이식 가능 컴포넌트 세그먼트(PCS)에 배치하여 재사용성을 극대화하고, 특정 하드웨어 및 운영체제 구현으로부터 분리해야 합니다. 이는 장기적인 유지보수성과 시스템 진화에 매우 유리합니다.
- **5. 인증 계획 수립:** 프로젝트 초기부터 인증 기관과 협력해야 합니다. 항공 안전성 인증(예: DO-178C)에 필요한 산출물을 생성하는 개발 프로세스를 사용하여 브릿지를 개발해야 합니다. 통합 시스템 전체의 인증 성공 여부는 브릿지 자체의 인증에 달려있다고 해도 과언이 아닙니다.

## 7. 결론

### 7.1 연구 결과 종합

본 보고서의 분석을 종합하면, 전용 브릿지를 통해 MAVLink 기반 시스템을 FACE 표준 준수 군용 항공전자 아키텍처에 통합하는 것은 기술적으로 실현 가능할 뿐만 아니라 전략적으로도 상당한 이점을 가집니다. 이는 상업 분야의 빠른 혁신 속도와 방대한 생태계를 활용하여 MOSA의 목표를 실현하는 실용적인 경로를 제시합니다. 이 통합 패턴은 국방 시스템이 더 민첩하고, 비용 효율적이며, 상호 운용 가능하도록 만드는 데 기여할 수 있습니다.

### 7.2 핵심 과제 재확인

그러나 이 접근법의 성공은 두 도메인 간의 근본적인 '의미론적 임피던스 불일치'와 '보증 격차'라는 두 가지 핵심 과제를 극복하는 데 전적으로 달려 있습니다. 첫 번째 과제는 MAVLink의 암시적이고 간결한 데이터 표현을 FACE의 명시적이고 엄격한 데이터 모델로 변환하는 기술적 복잡성을 해결하는 것입니다. 두 번째 과제는 비인증 오픈소스 구성 요소를 고신뢰성 군용 시스템에 통합하는 데 따르는 보안 및 안전성 인증의 어려움을 해결하는 것입니다.

### 7.3 최종 결론

결론적으로, 잘 설계되고 모델 기반의 보안 중심적인 MAVLink-FACE 브릿지는 신뢰할 수 없고 암시적으로 정의된 데이터를 현대 군용 항공 시스템에 적합한, 신뢰할 수 있고 검증 가능한 정보 흐름으로 변환하는 핵심적인 쐐기돌(lynchpin) 역할을 합니다. 이 브릿지는 단순한 변환 유틸리티가 아니라, 시스템의 안전성과 보안을 보장하는 중요한 미션 크리티컬 항공전자 구성 요소입니다. 따라서 그 개발은 다른 어떤 핵심 항공전자 부품과 동일한 수준의 엄격함과 규율을 가지고 접근해야 합니다. 이러한 노력을 통해 국방부는 개방형 아키텍처의 진정한 잠재력을 실현하고, 미래 전장에서의 기술적 우위를 확보할 수 있을 것입니다.

#### **참고 자료**

1. The FACE Approach | www.opengroup.org, accessed July 17, 2025, https://www.opengroup.org/face/approach
2. A Research Agenda for Mixed-Criticality Systems - UNC Computer Science, accessed July 17, 2025, [https://www.cs.unc.edu/~anderson/teach/comp790/papers/RBO-09-130%20Joint%20MCAR%20White%20Paper%20PA%20approved.doc](https://www.cs.unc.edu/~anderson/teach/comp790/papers/RBO-09-130 Joint MCAR White Paper PA approved.doc)
3. FACE Standard for Avionics Software | Ansys, accessed July 17, 2025, https://www.ansys.com/industries/face-standard-aviation-software
4. FACE Standard Compliant Software | FACE Architecture, accessed July 17, 2025, https://www.lynx.com/industry-standards/future-airborne-capability-environment-face
5. A new 'FACE' for Aviation Acquisition | Article | The United States Army, accessed July 17, 2025, https://www.army.mil/article/195215/a_new_face_for_aviation_acquisition
6. MAVLink Protocol | Drone Wolf Playbook, accessed July 17, 2025, [https://dronewolf.darkwolf.io/How%20They%20Work/Protocols/mavlink](https://dronewolf.darkwolf.io/How They Work/Protocols/mavlink)
7. MAVLink - Wikipedia, accessed July 17, 2025, https://en.wikipedia.org/wiki/MAVLink
8. Bridge between MAVLink and STANAG 4586 [54] - ResearchGate, accessed July 17, 2025, https://www.researchgate.net/figure/Bridge-between-MAVLink-and-STANAG-4586-54_fig4_333937231
9. Micro Air Vehicle Link (MAVLink) in a Nutshell: A Survey - arXiv, accessed July 17, 2025, https://arxiv.org/pdf/1906.10641
10. MAVLink wireless bridge (MAVBridge) - misc - twiddling bits and atoms, accessed July 17, 2025, https://wot.lv/mavlink-wireless-bridge-mavbridge.html
11. MAVLink Basics - Dev documentation - ArduPilot, accessed July 17, 2025, https://ardupilot.org/dev/docs/mavlink-basics.html
12. MAVLink Messaging | PX4 Guide (main), accessed July 17, 2025, https://docs.px4.io/main/en/mavlink/index.html
13. MAVLink - Clover, accessed July 17, 2025, https://clover.coex.tech/en/mavlink.html
14. MAVLINK Common Message Set (common.xml), accessed July 17, 2025, https://mavlink.io/en/messages/common.html
15. Packet Serialization | MAVLink Guide, accessed July 17, 2025, https://mavlink.io/en/guide/serialization.html
16. MAVLink Step by Step - Blog - ArduPilot Discourse, accessed July 17, 2025, https://discuss.ardupilot.org/t/mavlink-step-by-step/9629
17. MavLink Tutorial for Absolute Dummies (Part –I), accessed July 17, 2025, https://discuss.ardupilot.org/uploads/short-url/vS0JJd3BQfN9uF4DkY7bAeb6Svd.pdf
18. MUVIDS: False MAVLink Injection Attack Detection in Communication for Unmanned Vehicles, accessed July 17, 2025, https://www.ndss-symposium.org/wp-content/uploads/autosec2021_23036_paper.pdf
19. MAVLink Messages | Dissecting the Protocol - YouTube, accessed July 17, 2025, https://www.youtube.com/watch?v=Ha66uKC-od0
20. MAVLink 1.0 & 2.0 Header Structure | Download Scientific Diagram - ResearchGate, accessed July 17, 2025, https://www.researchgate.net/figure/MAVLink-10-20-Header-Structure_fig1_385670293
21. Vulnerability Analysis of the MAVLink Protocol for Command and Control of Unmanned Aircraft - AFIT Scholar, accessed July 17, 2025, https://scholar.afit.edu/context/etd/article/1613/viewcontent/AFIT_ENG_14_M_50_Marty_ADA598977.pdf
22. Leveraging Commercial Avionics Standards: Understanding the FACE Reference Architecture - RTI, accessed July 17, 2025, https://www.rti.com/blog/understanding-face-reference-architecture
23. FACE Technical Overview - The Open Group, accessed July 17, 2025, https://www.opengroup.org/sites/default/files/TWG_Overview_2021_TIM.pdf
24. What Is FACE™? | Wind River, accessed July 17, 2025, https://www.windriver.com/solutions/learning/face
25. Future Airborne Capability Environment - Wikipedia, accessed July 17, 2025, https://en.wikipedia.org/wiki/Future_Airborne_Capability_Environment
26. Applying Transport Services Segment (TSS) Concepts - OMG Wiki, accessed July 17, 2025, https://www.omgwiki.org/face/lib/exe/fetch.php?media=wiki:interop-wg:applying_tss_concepts_rev_b_no_comments.pdf
27. FACE and TPM: The Importance of the Transport Protocol Module - Real-Time Innovations, accessed July 17, 2025, https://www.rti.com/blog/face-and-tpm
28. Data Modelers | www.opengroup.org, accessed July 17, 2025, https://www.opengroup.org/face/datamodelers
29. An Introduction to Data Modeling - TES SAVi, accessed July 17, 2025, https://tes-savi.com/wp-content/uploads/2021/10/TES-SAVi-IntroToFACEDataModelingTraining-FACE-Army-TIM-Feb-2016.pdf
30. FACE™ Consortium Data Model Overview - YouTube, accessed July 17, 2025, https://www.youtube.com/watch?v=Em32ItuzeCE
31. Just Enough Data Model: A Hero's Journey Into The Modeling World - Skayl, accessed July 17, 2025, https://www.skayl.com/post/just-enough-data-model
32. DOCUMENTS & TOOLS | www.opengroup.org, accessed July 17, 2025, https://www.opengroup.org/face/docsandtools
33. Sample Code for Hello.idl, accessed July 17, 2025, https://docs.oracle.com/javase/8/docs/technotes/guides/idl/jidlSampleCode.html
34. Interface Definition Language - Object Management Group, accessed July 17, 2025, https://www.omg.org/spec/IDL/4.2/PDF
35. About the Interface Definition Language Specification Version 4.2, accessed July 17, 2025, https://www.omg.org/spec/IDL/4.2/About-IDL
36. Making a MAVLink WiFi bridge using the Raspberry Pi - Dev documentation - ArduPilot, accessed July 17, 2025, https://ardupilot.org/dev/docs/making-a-mavlink-wifi-bridge-using-the-raspberry-pi.html
37. Hardware Design and Implementation of a MAVLink ... - QUT ePrints, accessed July 17, 2025, https://eprints.qut.edu.au/79541/1/ACRA_2014b.pdf
38. Communicating with Raspberry Pi via MAVLink - Dev documentation - ArduPilot, accessed July 17, 2025, https://ardupilot.org/dev/docs/raspberry-pi-via-mavlink.html
39. What is Zero Trust Architecture? - Palo Alto Networks, accessed July 17, 2025, https://www.paloaltonetworks.com/cyberpedia/what-is-a-zero-trust-architecture
40. Department of Defense Zero Trust Reference Architecture - DoD CIO, accessed July 17, 2025, https://dodcio.defense.gov/Portals/0/Documents/Library/(U)ZT_RA_v2.0(U)_Sep22.pdf
41. What Are Mixed-Criticality OS Environments? - Wind River Systems, accessed July 17, 2025, https://www.windriver.com/solutions/learning/what-are-mixed-criticality-os-environments
42. Towards the Design of Certifiable Mixed-criticality Systems | Request PDF - ResearchGate, accessed July 17, 2025, https://www.researchgate.net/publication/220266985_Towards_the_Design_of_Certifiable_Mixed-criticality_Systems
43. Mixed Criticality Systems - A Review - CiteSeerX, accessed July 17, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=4864bab94f01dcd864e075d5c391e83868e6125d
44. Robust Mixed-Criticality Systems - University of York, accessed July 17, 2025, https://www-users.york.ac.uk/~rd17/papers/ToCRobustMCS.pdf
45. What is mixed criticality? - Trenton Systems, accessed July 17, 2025, https://www.trentonsystems.com/en-us/resource-hub/blog/what-is-mixed-criticality


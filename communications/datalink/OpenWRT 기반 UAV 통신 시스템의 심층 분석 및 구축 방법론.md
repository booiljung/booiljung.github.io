[데이터 링크 (Datalink)](./index.md)
# OpenWRT 기반 UAV 통신 시스템의 심층 분석 및 구축 방법론



OpenWRT는 2004년, Linksys사의 WRT54G 유무선 공유기 펌웨어를 커스터마이징하기 위한 작은 프로젝트에서 시작되었다.1 당시 제조사들은 GPL 라이선스를 따르는 리눅스 커널을 사용하면서도, 수정한 소스코드를 공개하지 않는 경우가 많았다. 이에 개발자들이 직접 펌웨어를 분석하고 수정하면서, OpenWRT는 특정 장비의 한계를 넘어 수많은 임베디드 장치를 지원하는 완전한 형태의 리눅스 배포판으로 진화하였다.

OpenWRT의 핵심 철학은 제조사가 제공하는 제한된 기능의 '블랙박스'에서 탈피하여, 사용자에게 시스템에 대한 완전하고 세밀한 제어권을 부여하는 것이다.1 이는 단순한 펌웨어 교체를 넘어, 평범한 가정용 공유기를 강력한 방화벽, VPN 서버, 미디어 스트리머, 또는 IoT 게이트웨이와 같은 다목적 네트워크 서버로 변모시킬 수 있는 무한한 가능성을 열어준다.1

이러한 철학은 무인 항공기(Unmanned Aerial Vehicle, UAV), 즉 드론 시스템에 적용될 때 그 가치가 극대화된다. 현대의 고성능 드론에 탑재되는 컴패니언 컴퓨터(Companion Computer)는 본질적으로 CPU, RAM, 저장 공간이 제한된 고성능 임베디드 시스템이라는 점에서 OpenWRT의 이상적인 적용 대상이다.3 OpenWRT의 경량 설계, 모듈식 아키텍처, 그리고 타의 추종을 불허하는 강력한 네트워크 기능은 비행 제어기(Flight Controller, FC)의 핵심 기능인 비행 안정성을 해치지 않으면서, 통신, 데이터 처리, 원격 제어 등 드론의 지능을 한 차원 높이는 데 최적화된 운영 환경을 제공한다.1


OpenWRT의 강력함은 단순히 기능이 많은 것에서 비롯되지 않는다. 그 힘의 원천은 제한된 자원 환경에서 최대의 성능과 유연성을 발휘하도록 세심하게 설계된 기술 아키텍처에 있다.

- **리눅스 커널 (Linux Kernel):** OpenWRT의 심장부로, 최신의 장기 지원(Long Term Support, LTS) 커널을 기반으로 한다.1 이는 최신 하드웨어에 대한 폭넓은 지원과 시스템 안정성을 보장한다. 특히, 임베디드 시스템에 특화된 다양한 패치가 적용되어 있는데, 저전력 모드 지원, 다양한 네트워크 프로토콜 스택, 그리고 실시간 제어가 중요한 드론 애플리케이션을 위한 `PREEMPT_RT` (실시간 선점) 패치 옵션 등이 포함된다.1 이를 통해 OpenWRT는 드론에 탑재된 다양한 센서와 주변 장치(GPS, 카메라, LTE 모뎀 등)의 드라이버를 안정적으로 통합하고 관리하는 견고한 기반이 된다.

- **프로세스 관리 (`procd`):** 시스템 부팅 시 가장 먼저 실행되어 다른 모든 프로세스의 생성과 관리를 책임지는 init 시스템으로, OpenWRT는 `systemd`나 `sysvinit` 같은 데스크톱 환경의 무거운 init 시스템 대신, 자체 개발한 경량의 `procd` (process daemon)를 사용한다.1

  `procd`는 빠른 부팅 속도와 현저히 낮은 리소스 점유율을 자랑하며, 이는 컴퓨팅 자원이 한정된 드론의 컴패니언 컴퓨터에서 비행 제어와 관련된 핵심 프로세스에 더 많은 자원을 할당할 수 있게 해주는 결정적인 장점이다.

- **패키지 관리 (`opkg`):** `opkg` (Open PacKaGe Management)는 Debian의 `apt`나 Red Hat의 `yum`과 유사한 역할을 하지만, 임베디드 시스템에 최적화되어 매우 가볍게 설계된 패키지 관리 시스템이다.1 다른 많은 임베디드 펌웨어들이 새로운 기능을 추가하기 위해 전체 펌웨어 이미지를 다시 컴파일하고 플래싱해야 하는 번거로움이 있는 반면 2, OpenWRT는 `opkg`를 통해 사용자가 필요에 따라 수천 개의 소프트웨어 패키지를 마치 스마트폰 앱처럼 손쉽게 설치하고 제거할 수 있다. `WireGuard` VPN, `GStreamer` 영상 처리, `ser2net` 시리얼 통신 등 드론에 필요한 거의 모든 기능을 전체 시스템 재설치 없이 동적으로 추가할 수 있게 하는 OpenWRT 유연성의 핵심이다.

- **설정 관리 (UCI, Unified Configuration Interface):** 리눅스 시스템의 복잡하고 분산된 설정 파일들을 `/etc/config/` 디렉토리 아래의 단순하고 통일된 형식으로 표준화한 시스템이다.1 네트워크, 방화벽, 시스템 서비스 등 거의 모든 설정이 UCI를 통해 관리되며, 이는 셸 명령어(`uci set`, `uci commit`)나 LuCI 웹 인터페이스를 통해 일관되고 프로그래밍 가능한 방식으로 시스템을 제어할 수 있게 한다. 드론 시스템의 구성을 스크립트 기반으로 자동화하고, 현장에서 설정을 신속하게 변경해야 할 때 매우 유용하다.

- **웹 인터페이스 (LuCI, Lua Configuration Interface):** LuCI는 Lua 프로그래밍 언어를 기반으로 한 가볍고 확장성이 뛰어난 웹 기반 사용자 인터페이스다.1 사용자는 웹 브라우저를 통해 복잡한 네트워크 라우팅, 방화벽 규칙, VPN 설정 등을 그래픽 환경에서 직관적으로 처리할 수 있다. 이는 드론을 현장에서 테스트하거나 운용할 때, 터미널에 직접 접속하는 번거로움 없이 스마트폰이나 노트북으로 빠르게 네트워크 구성을 확인하고 변경할 수 있는 큰 편의성을 제공한다.

- **네트워크 프레임워크 (`netifd`):** `netifd` (network interface daemon)는 OpenWRT의 네트워크 구성을 동적으로 관리하는 핵심 데몬이다.1 사용자가 UCI를 통해 네트워크 설정을 변경하면, `netifd`는 해당 변경 사항을 감지하여 관련 네트워크 인터페이스와 서비스를 자동으로 재시작하고 적용한다. LTE 동글을 연결했을 때 자동으로 WAN 인터페이스를 활성화하거나, Wi-Fi를 클라이언트 모드에서 AP 모드로 전환하는 등, 드론 운용 중 발생할 수 있는 다양한 네트워크 시나리오 변화에 시스템이 지능적이고 유연하게 대응할 수 있도록 하는 기반 기술이다.

데스크톱 리눅스 배포판이 `systemd`, `dbus`, `glibc`와 같이 강력하지만 상대적으로 무거운 라이브러리 스택에 의존하는 것과 대조적으로, OpenWRT는 `procd`, `ubus`, `libubox`라는 자체적인 경량화된 대안을 채택했다.5 이는 단순한 기술적 취향의 차이를 넘어, '제한된 자원 내에서 최대의 네트워크 성능과 유연성을 달성한다'는 OpenWRT의 근본적인 설계 철학을 명확히 보여준다. 예를 들어, `dbus`의 C 언어 API는 사용법이 복잡하고 많은 상용구 코드(boilerplate code)를 요구하는 것으로 알려져 있지만, `ubus`는 C 코드뿐만 아니라 일반 셸 스크립트에서도 매우 쉽게 접근하고 사용할 수 있도록 설계되었다.5 이러한 설계 철학은 OpenWRT가 단순한 '공유기 운영체제'가 아니라, IoT 게이트웨이나 드론 컴패니언 컴퓨터와 같은 '네트워크 엣지 컴퓨팅 플랫폼'으로서 최적화되었음을 증명한다. 이는 일부 데스크톱 수준의 편의 기능을 포기하는 대신, 네트워크 처리라는 핵심 임무에 대한 안정성, 성능, 그리고 확장성을 극대화하는 전략적 트레이드오프의 성공적인 결과물이라 할 수 있다.


OpenWRT는 제조사가 제공하는 기본 펌웨어와 비교할 때 명확한 장점과 함께 고려해야 할 한계점을 가지고 있다.

**장점:**

- **압도적인 확장성:** `opkg` 패키지 관리 시스템을 통해 VPN 서버, 웹 서버, 광고 차단(Ad-block), 메시 네트워킹, QoS, 실시간 네트워크 모니터링 등 상상할 수 있는 거의 모든 네트워크 관련 기능을 자유롭게 추가할 수 있다.1
- **강화된 보안:** 전 세계 개발자들로 구성된 활발한 오픈소스 커뮤니티 덕분에 보안 취약점이 발견되면 신속하게 패치가 이루어진다.1 특히 제조사가 보안 업데이트 지원을 중단한 구형 하드웨어일지라도 OpenWRT를 설치함으로써 최신 보안 상태를 유지하고 장비의 수명을 연장할 수 있다.2
- **완벽한 제어권:** 방화벽 규칙, 라우팅 테이블, VLAN, 브릿지 설정 등 네트워크의 모든 요소를 사용자가 원하는 대로 세밀하게 제어할 수 있다.2 이는 복잡한 드론 통신 아키텍처를 구현하는 데 필수적인 요소다.

**한계:**

- **높은 학습 곡선:** 강력한 기능과 높은 자유도는 필연적으로 복잡성을 동반한다. OpenWRT의 모든 기능을 활용하기 위해서는 사용자가 리눅스 운영체제와 네트워킹의 기본 원리에 대한 일정 수준 이상의 지식을 갖추어야 한다.
- **안정성 및 문서화 문제:** 커뮤니티 주도 개발 모델의 특성상, 때로는 핵심 개발팀과 커뮤니티 기여자 간의 소통 부족이나 관리 인력의 부족으로 인해 특정 개발 버전(snapshot)의 안정성이 떨어지거나 최신 기능에 대한 문서화가 미흡한 경우가 발생할 수 있다.3 따라서 실제 임무에 투입되는 드론과 같은 중요한 시스템에는 충분히 검증된 안정화 릴리즈(stable release)를 사용하는 것이 현명하다.
- **하드웨어 호환성:** OpenWRT가 수많은 하드웨어를 지원하지만, 모든 장비를 지원하는 것은 아니다. 특정 하드웨어의 경우, Wi-Fi 칩셋 드라이버나 특수 기능이 완벽하게 지원되지 않을 수 있으므로, 하드웨어 구매 전에 OpenWRT 공식 웹사이트의 'Table of Hardware'를 통해 호환성 여부를 반드시 확인해야 한다.4



드론의 두뇌 역할을 하는 비행 제어기(Flight Controller, FC)는 자이로스코프, 가속도계 등 각종 센서 데이터를 수신하여 기체의 자세를 안정적으로 유지하고, 조종사의 명령이나 자율 비행 경로에 따라 모터를 제어하는 핵심적인 임무에 모든 연산 능력을 집중하도록 설계되었다.7 이 때문에 FC는 고화질 영상 처리, 복잡한 인공지능 연산, 또는 다중 네트워크 인터페이스를 통한 통신과 같은 고수준 작업을 수행하기에는 부적합하다.8

바로 이 지점에서 컴패니언 컴퓨터(Companion Computer, CC)의 필요성이 대두된다. CC는 FC와 나란히 기체에 탑재되어, FC와는 독립적으로 고도의 연산 및 통신 작업을 전담하는 보조 컴퓨터다.8 CC는 일반적으로 FC와 시리얼(UART) 포트를 통해 연결되며, MAVLink(Micro Air Vehicle Link)라는 표준 프로토콜을 사용하여 통신한다.8 CC는 FC로부터 GPS 위치, 기체 자세, 배터리 상태, 센서 값 등 모든 종류의 텔레메트리 데이터를 실시간으로 수신하며 9, 이 데이터를 기반으로 다음과 같은 지능적인 작업을 수행한다:

- **지능형 비행 및 연산:** 카메라로부터 입력된 영상을 실시간으로 분석하여 장애물을 회피하거나, 특정 객체를 추적하는 등의 컴퓨터 비전 및 AI 기반 임무를 수행한다.8 이는 FC의 연산 능력만으로는 불가능한 고도의 자율 비행을 가능하게 한다.
- **통신 허브:** FC로부터 받은 MAVLink 텔레메트리 데이터를 Wi-Fi, LTE/5G 등 다양한 무선 통신 매체를 통해 지상관제시스템(Ground Control Station, GCS)이나 원격 클라우드 서버로 중계하는 허브 역할을 한다.12
- **페이로드 관리 및 제어:** 다수의 카메라, 고성능 센서(LiDAR, 초음파 등), 서보 모터, 그리퍼 등 복잡한 페이로드를 직접 제어하고, 이들로부터 수집된 데이터를 전처리하거나 저장하는 역할을 담당한다.9

이러한 CC의 운영체제로서 OpenWRT는 최적의 선택지다. OpenWRT는 CC 위에서 동작하며, 앞서 언급된 복잡한 네트워크 라우팅, 보안 VPN 연결, 다중 데이터 스트리밍과 같은 작업을 안정적이고 효율적으로 처리하는 강력한 기반을 제공한다.12 즉, FC가 '안정적인 비행'을 책임진다면, OpenWRT가 탑재된 CC는 '지능적인 통신과 연산'을 책임지며 드론 시스템의 전체 능력을 극대화하는 것이다.


드론 펌웨어의 양대 산맥인 ArduPilot과 PX4는 모두 MAVLink 프로토콜을 사용하여 CC와 통신한다는 공통점을 가지지만, CC를 활용하는 방식과 생태계의 철학에서 미묘한 차이를 보인다.

- **ArduPilot 생태계:** ArduPilot 커뮤니티는 `DroneKit-Python`이나 `MAVProxy`와 같이 Python을 기반으로 한 고수준 라이브러리와 도구를 활용하여 CC의 애플리케이션을 개발하는 경향이 강하다.11 ArduPilot 아키텍처에서 CC는 단순히 GCS로 데이터를 중계하는 보조적인 역할을 넘어, 비행 로직의 일부를 직접 수행하는 능동적인 주체로 활용되는 경우가 많다. 예를 들어, GCS는 단순히 기체를 모니터링하고 고수준의 명령(예: "임무 시작")만 내리고, 실제 복잡한 자율 비행 로직은 CC에 탑재된 DroneKit 스크립트가 전담하여 수행하는 아키텍처가 일반적이다.11 ArduPilot의 공식 문서는 Raspberry Pi, Odroid 등 다양한 CC 하드웨어와 DroneKit, ROS 등 여러 소프트웨어 스택을 활용하는 풍부한 예제와 가이드를 제공하며 개발자의 빠른 프로토타이핑을 장려한다.9

- **PX4 생태계:** PX4는 `MAVROS`(ROS와의 깊은 통합), `MAVLink Router`, C/C++ 기반 애플리케이션 등 보다 시스템 수준에 가깝고 정형화된 연동 방식을 강조한다.10 PX4 문서는 CC 연결을 위해 FC의 

  `TELEM2` 포트를 'Onboard' 모드로 설정하고(`MAV_1_MODE = Onboard`), 로그 스트리밍이나 FastRTPS와 같은 대용량 데이터 전송을 위해 Baudrate를 921600 bps 이상으로 설정할 것을 명확하게 가이드한다.10 또한, FC의 시리얼 포트(3.3V)와 최신 CC(1.8V) 간의 전압 레벨 차이로 인한 하드웨어 손상 가능성을 구체적으로 경고하며, 전압 레벨 시프터(level shifter)의 사용을 강력히 권장하는 등, 하드웨어 인터페이싱의 안정성과 신뢰성을 매우 중요하게 다룬다.10

ArduPilot과 PX4의 CC 연동 방식에 대한 접근법 차이는 두 프로젝트가 지향하는 철학의 차이를 반영한다. ArduPilot은 `DroneKit-Python` 11 등을 통해 개발자들이 빠르고 쉽게 아이디어를 구현할 수 있도록 하는 '유연성'과 '개발자 친화성'을 우선시한다. 이는 풍부한 커뮤니티 기반의 예제와 프로젝트로 이어져 생태계를 더욱 활성화시킨다.11 반면, PX4는 

`MAV_1_CONFIG` 파라미터 설정, Baudrate 권장, 전압 레벨 주의사항 등 '명확하고 안정적인' 시스템 통합 가이드를 제공하는 데 중점을 둔다.10 이는 PX4가 산업용 및 상업용 애플리케이션에서 요구되는 높은 수준의 신뢰성과 표준화를 지향하고 있음을 보여준다. 개발자는 자신의 프로젝트 목표가 빠른 프로토타이핑인지, 아니면 높은 신뢰성을 갖춘 시스템 통합인지에 따라 적합한 생태계를 선택해야 한다. 중요한 점은, OpenWRT가 이러한 두 가지 상이한 접근 방식을 모두 수용하고 지원할 수 있는 중립적이고 강력한 네트워크 운영 플랫폼으로서 기능한다는 것이다.


OpenWRT를 활용하면 드론 내외부의 네트워크를 체계적으로 설계하고 관리할 수 있다.

- **기체 내부 네트워크 (Onboard Network):**
  - **이상적 토폴로지 (Star Topology):** 네트워크 설계에서 가장 이상적인 형태는 스타(Star) 토폴로지다. 이는 중앙의 스위치(또는 라우터)를 중심으로 모든 장치가 직접 연결되는 구조로, 각 장치 간의 통신 지연 시간을 최소화하고 대역폭을 최대로 보장한다.19 드론 시스템에서는 OpenWRT 라우터를 이 중앙 허브로 사용한다. FC, CC, IP 카메라, LTE 모뎀 등 IP 통신이 필요한 모든 장치들이 OpenWRT 라우터에 직접 연결된다. 이때 OpenWRT 라우터는 이 내부 네트워크의 DHCP 서버 역할을 하여 각 장치에 IP 주소를 할당하고, 방화벽 기능을 통해 내부 네트워크를 외부로부터 보호한다.19
  - **실제 구성:** 시중에서 구할 수 있는 UAV 전용 OpenWRT 라우터는 이러한 스타 토폴로지를 구현할 수 있도록 FC, CC, 카메라 등을 위한 다수의 이더넷 포트와 LTE 동글 연결을 위한 USB 포트를 기본적으로 제공한다.12
- **드론-GCS-클라우드 연동 네트워크:**
  - **VPN을 통한 가상 사설망 구축:** 드론이 LTE나 5G 망을 통해 인터넷에 연결될 경우, 대부분의 통신사는 CG-NAT(Carrier-Grade NAT) 정책을 사용하기 때문에 GCS에서 드론의 IP 주소로 직접 접속하는 것이 불가능하다.20 이 문제를 해결하기 위해 `WireGuard`나 `ZeroTier`와 같은 VPN(가상 사설망) 기술을 사용한다. VPN을 통해 드론, GCS, 그리고 필요하다면 클라우드 서버까지 모두 하나의 암호화된 가상 네트워크로 묶을 수 있다.12 이렇게 구성하면, 드론과 GCS가 물리적으로 어디에 있든지 간에 마치 같은 로컬 네트워크에 있는 것처럼 서로의 IP 주소로 안전하게 직접 통신할 수 있게 된다.
  - **메시(Mesh) 네트워크를 통한 스웜 통신:** 여러 대의 드론을 동시에 운용하는 스웜(swarm) 비행 시나리오에서는, OpenWRT의 강력한 메시 네트워킹 기능을 활용할 수 있다. `B.A.T.M.A.N.-adv` (Better Approach To Mobile Ad-hoc Networking)와 같은 프로토콜을 OpenWRT에 설치하면, 드론들이 서로를 중계기 삼아 통신하는 자가 치유(self-healing) 및 자가 구성(self-configuring) 네트워크를 구축할 수 있다.21 이를 통해 일부 드론의 통신이 끊기더라도 다른 드론을 경유하는 대체 경로를 통해 전체 네트워크 연결을 유지하는 매우 강인한(robust) 통신망을 구현할 수 있으며, 이는 군집 정찰이나 재난 지역 탐사 등에서 필수적인 기능이다.



UAV 시스템에 OpenWRT를 도입하기 위한 첫 단계는 목적에 맞는 하드웨어를 선정하고 기본 설정을 완료하는 것이다.

- **선정 기준:** UAV에 탑재될 컴패니언 컴퓨터 또는 라우터는 다음과 같은 요소를 종합적으로 고려하여 신중하게 선택해야 한다.
  - **물리적 제원:** 크기, 무게, 전력 소모는 비행시간과 기체 설계에 직접적인 영향을 미치는 가장 중요한 요소다.
  - **포트 구성:** FC와의 시리얼(UART) 연결, IP 카메라 및 다른 CC와의 이더넷 연결, LTE/5G 동글을 위한 USB 포트 등 필요한 인터페이스를 충분히 갖추었는지 확인해야 한다.
  - **처리 성능:** OpenWRT 자체는 가볍지만, VPN 암호화, 고화질 영상 스트리밍, 다중 MAVLink 라우팅 등 부하가 큰 작업을 수행하려면 충분한 CPU 성능과 RAM 용량이 필요하다. OpenWRT를 원활하게 구동하기 위한 최소 사양은 8MB 플래시 메모리와 64MB RAM으로 권장된다.3
- **펌웨어 설치 및 초기 설정:**
  1. 선정한 하드웨어 모델을 OpenWRT 공식 웹사이트의 'Table of Hardware' (`openwrt.org/toh/start`)에서 검색하여 전용 펌웨어 이미지 파일을 다운로드한다.4
  2. 해당 하드웨어 제조사가 제공하는 펌웨어 업로드 가이드에 따라 OpenWRT 이미지를 플래싱한다.
  3. 설치가 완료되면, 컴퓨터와 OpenWRT 장치를 이더넷 케이블로 연결하고 웹 브라우저나 SSH를 통해 접속한다.
  4. 가장 먼저 시스템의 루트(root) 계정 비밀번호를 설정하여 보안을 강화하고, 내부 네트워크에서 사용할 LAN 인터페이스의 고정 IP 주소(예: 192.168.1.1)를 구성한다.

다음은 UAV 환경에 적용 가능한 OpenWRT 지원 하드웨어의 비교표이다.

| 모델명                    | CPU              | RAM / Flash    | 이더넷 포트   | USB                    | Wi-Fi             | 시리얼 | 크기/무게     | 주요 특징                                 | 관련 자료 |
| ------------------------- | ---------------- | -------------- | ------------- | ---------------------- | ----------------- | ------ | ------------- | ----------------------------------------- | --------- |
| **CBUnmanned UAV Router** | MediaTek MT7628  | 128MB / 32MB   | 5 (4+1)       | 1 x USB 2.0            | 2.4GHz 802.11n    | 2      | 30.5mm 표준   | UAV 전용 설계, JST-GH 커넥터, 경량        | 12        |
| **Raspberry Pi 4/CM4**    | Broadcom BCM2711 | 1/2/4/8GB / -  | 1 (Onboard)   | 2x USB 3.0, 2x USB 2.0 | 2.4/5GHz 802.11ac | UART   | 다양함        | 높은 성능, 다양한 베이스보드, GPIO 확장성 | 3         |
| **TP-Link TL-WR902AC**    | Qualcomm QCA9531 | 64MB / 8MB     | 1             | 1 x USB 2.0            | 2.4/5GHz 802.11ac | -      | 소형/경량     | 휴대성, 저렴한 가격, 듀얼밴드 지원        | 4         |
| **Alfa Tube 2H**          | Atheros AR9331   | 32MB / 8MB     | 1             | -                      | 2.4GHz 802.11n    | -      | 원통형        | 고출력(최대 27dBm), 외장 안테나, PoE 지원 | 4         |
| **Banana Pi BPI-R3**      | MediaTek MT7986  | 2GB / 8GB eMMC | 5 (2x 2.5GbE) | 1 x USB 3.0            | 2.4/5GHz Wi-Fi 6  | UART   | 상대적으로 큼 | Docker 지원, 고성능, 2.5GbE, M.2 슬롯     | 22        |


FC에서 생성되는 시리얼(UART) 기반의 MAVLink 텔레메트리 데이터를 IP 네트워크(TCP 또는 UDP)로 변환하는 것은 OpenWRT 기반 통신 시스템의 가장 기본적이면서도 중요한 단계다. 이를 'IP 브릿징'이라 하며, 이를 통해 OpenWRT에 유선 또는 무선으로 연결된 모든 장치(컴패니언 컴퓨터, GCS, IP 카메라 등)가 FC의 데이터에 접근할 수 있게 된다. 주요 방법은 두 가지다.

- 방법 1: ser2net (경량 시리얼-네트워크 브릿지)

  ser2net은 이름 그대로 시리얼 포트와 네트워크 포트를 연결해주는 매우 가볍고 단순한 유틸리티다.

  - **설치:** OpenWRT의 터미널에 접속하여 `opkg`로 간단히 설치할 수 있다.

    ```Bash
    opkg update && opkg install ser2net
    ```

    23

  - **설정:** 설정은 `/etc/ser2net.conf` (구버전) 또는 `/etc/ser2net.yaml` (신버전) 파일을 수정하여 이루어진다.24 구버전 `conf` 파일의 경우, 한 줄의 설정만으로 브릿징을 구성할 수 있다.

    ```
    <NETWORK_PORT>:<STATE>:<TIMEOUT>:<DEVICE>:<OPTIONS>
    ```

    예를 들어, FC가 `/dev/ttyS1`에 115200 Baudrate로 연결되어 있고, 이를 TCP 포트 5760으로 브릿징하려면 다음과 같이 설정한다.

    ```
    5760:raw:0:/dev/ttyS1:115200 8DATABITS NONE 1STOPBIT
    ```

    23

  - **장점:** `ser2net`은 리소스 사용량이 극히 적고 설정이 매우 간단하여, FC와 GCS 간의 1:1 텔레메트리 중계와 같이 단순한 작업에 매우 적합하다.26

- 방법 2: MAVProxy (다기능 MAVLink 라우터)

  MAVProxy는 단순한 브릿지를 넘어, MAVLink 프로토콜을 이해하고 지능적으로 라우팅할 수 있는 강력한 GCS 겸 개발 도구다.

  - **설치:** `MAVProxy`는 Python으로 작성되었으므로, OpenWRT에 Python 실행 환경을 먼저 구축해야 한다.

    ```Bash
    opkg update
    opkg install python3 python3-pip
    pip install mavproxy
    ```

    27

  - **실행:** `MAVProxy`는 커맨드라인 옵션을 통해 입력과 출력을 자유롭게 지정할 수 있다.

    ```Bash
    mavproxy.py --master=/dev/ttyUSB0 --baudrate=57600 --out=udpbcast:192.168.1.255:14550 --out=udpbcast:10.8.0.255:14550
    ```

  - **명령어 분석:**

    - `--master=/dev/ttyUSB0 --baudrate=57600`: FC가 연결된 주 MAVLink 소스(시리얼 포트와 Baudrate)를 지정한다.28
    - `--out=...`: MAVLink 데이터를 전달할 목적지를 지정한다. `--out` 옵션은 여러 번 사용하여 하나의 입력 데이터를 다수의 목적지로 동시에 전달할 수 있다.26 위 예시는 FC로부터 받은 데이터를 기체 내부 LAN(192.168.1.0/24 대역)과 VPN으로 연결된 원격 네트워크(10.8.0.0/24 대역) 양쪽으로 동시에 UDP 브로드캐스팅하는 구성이다.

  - **장점:** 단순 중계를 넘어 MAVLink 패킷의 선택적 라우팅, 필터링, 다수의 GCS 및 온보드 스크립트로의 동시 데이터 분배 등 매우 복잡하고 유연한 네트워크 구성이 가능하다.26

`ser2net`과 `MAVProxy`의 선택은 기술적 선호를 넘어 아키텍처 설계 철학의 문제와 직결된다. `ser2net` 23을 사용하는 것은 OpenWRT 라우터를 FC의 시리얼 포트를 네트워크 포트로 '투명하게' 확장해주는 '무선 시리얼 어댑터'나 '터미널 서버'로 활용하는 개념에 가깝다. 이는 가장 단순하고 효율적인 방법이다. 반면, `MAVProxy` 26를 사용하면 OpenWRT 라우터는 MAVLink 네트워크의 '지능형 허브(Intelligent Hub)'로 격상된다. FC에서 들어온 단일 데이터 스트림을 기체 내부에 있는 IP 카메라 제어 스크립트, LTE를 통해 연결된 원격 GCS, 그리고 클라우드 기반의 비행 데이터 분석 시스템 등 여러 목적지로 동시에, 그리고 선택적으로 분배하고 라우팅할 수 있다. 예를 들어, 대역폭이 제한적인 LTE 링크로는 필수적인 저속 텔레메트리만 전송하고, 근거리의 고속 Wi-Fi 링크로는 모든 데이터를 전송하는 식의 차등적 라우팅 정책을 구현할 수 있다. 따라서 프로젝트의 요구사항이 복잡해지고 확장성이 중요해질수록, `MAVProxy`의 아키텍처적 가치는 기하급수적으로 증가한다.


드론의 운용 반경을 시야 밖(Beyond Visual Line of Sight, BVLOS)으로 확장하기 위해서는 LTE나 5G와 같은 이동통신망 연동이 필수적이다. OpenWRT는 이를 위한 강력한 기능을 제공한다.

- **모뎀 준비 및 설정:** 최신 LTE/5G 모뎀은 주로 QMI(Qualcomm MSM Interface) 또는 MBIM(Mobile Broadband Interface Model) 프로토콜을 사용하여 통신한다. 모뎀을 USB 포트에 연결하기 전에, PC의 터미널 프로그램을 통해 모뎀에 접속하여 `AT` 명령어로 올바른 동작 모드를 설정하고, 사용하는 통신사의 APN(Access Point Name) 정보를 입력해야 할 수 있다.30 예를 들어, Quectel 모뎀의 경우 다음과 같은 명령어를 사용한다.

  - `AT+QCFG="usbnet",0`: QMI 모드로 설정
  - `AT+CGDCONT=1,"IP","internet"`: APN을 'internet'으로 설정

- **필수 패키지 설치:** OpenWRT가 모뎀을 인식하고 제어하기 위해 필요한 커널 드라이버와 유틸리티를 설치한다.

  - **QMI 프로토콜 모뎀:**

    ```Bash
    opkg install kmod-usb-net-qmi-wwan uqmi luci-proto-qmi
    ```

    30

  - **MBIM 프로토콜 모뎀:**

    ```Bash
    opkg install kmod-usb-net-cdc-mbim umbim luci-proto-mbim
    ```

    30

- **인터페이스 설정 (LuCI 웹 인터페이스 기준):**

  1. `Network` -> `Interfaces` 메뉴로 이동하여 `Add new interface...` 버튼을 클릭한다.
  2. 새 인터페이스의 이름을 `LTE_WAN` 등으로 지정하고, `Protocol` 드롭다운 메뉴에서 `QMI Cellular` 또는 `MBIM`을 선택한다.
  3. 생성된 인터페이스의 설정 화면에서, `Modem device` 항목에 시스템이 인식한 모뎀 장치 경로(예: `/dev/cdc-wdm0`)를 선택하고, 통신사로부터 받은 `APN`을 입력한다.
  4. `Firewall Settings` 탭으로 이동하여, 이 새로운 인터페이스를 `wan` 방화벽 존(zone)에 할당한다. 이 과정을 통해 LTE 연결을 통한 인터넷 트래픽이 방화벽에 의해 올바르게 처리된다.

- 고급 구성 (Passthrough / Half-bridge 모드):

  일반적인 라우터 모드에서는 OpenWRT 장치가 LTE 모뎀으로부터 사설 IP를 할당받고, 그 아래 연결된 장치들에게 또 다른 사설 IP를 할당하는 이중(Double) NAT 구조가 된다. 이는 포트 포워딩을 복잡하게 만들 수 있다. Passthrough 또는 Half-bridge 모드는 OpenWRT 라우터가 모뎀으로부터 받은 공인 IP 주소(또는 CG-NAT IP)를 내부 네트워크의 특정 장치(예: 주력 컴패니언 컴퓨터)에게 그대로 전달해주는 기능이다.31 이를 통해 이중 NAT 문제를 해결하고 네트워크 구성을 단순화할 수 있다. 일부 모뎀 펌웨어나 Mikrotik과 같은 특정 장비에서 이 기능을 직접 지원하며 31, OpenWRT에서는 `relayd`와 같은 패키지를 이용해 유사한 기능을 구현할 수도 있다.32


LTE/5G 망을 사용하더라도 CG-NAT 환경 때문에 외부에서 드론으로 직접 접속하는 것은 여전히 어렵다. VPN은 이 문제를 해결하고 안전한 통신 채널을 확보하는 가장 효과적인 방법이다.

- **CG-NAT 극복을 위한 간편한 솔루션 (ZeroTier):**

  - **원리:** `ZeroTier`는 복잡한 서버 설정 없이 P2P 홀 펀칭(hole punching) 기술을 사용하여 NAT나 방화벽 뒤에 있는 장치들 간에 직접 통신이 가능한 가상의 L2 스위치를 만들어주는 서비스다.20
  - **설정:** OpenWRT에 `zerotier` 패키지를 설치한 후, `ZeroTier Central` 웹사이트에서 생성한 16자리의 `Network ID`를 사용하여 가상 네트워크에 참여시키기만 하면 된다. 상세 가이드는 `https://github.com/mwarning/zerotier-openwrt/wiki`에서 확인할 수 있다.20 이 방식을 사용하면 드론, GCS, 개발자 PC가 물리적으로 어디에 있든 상관없이, 마치 같은 공유기 아래에 있는 것처럼 서로의 가상 IP 주소로 매우 쉽게 통신할 수 있다.20

- **고성능 보안 터널링 (WireGuard):**

  - **원리:** `WireGuard`는 리눅스 커널 내부에 직접 구현되어 동작하는 최신 VPN 프로토콜이다. 기존의 OpenVPN이나 IPsec에 비해 코드베이스가 매우 간결하고, 암호화 오버헤드가 적어 월등히 높은 성능과 낮은 지연 시간을 제공한다.33 이는 고화질 영상 스트리밍과 같이 대역폭과 지연 시간에 민감한 드론 애플리케이션에 가장 이상적인 VPN 솔루션으로 평가받는다.

  - **패키지 설치:**

    ```Bash
    opkg install wireguard-tools kmod-wireguard luci-proto-wireguard
    ```

    34

  - **서버-클라이언트 구성 (클라우드 VPS에 서버, 드론에 클라이언트 구성 시):**

    1. **서버 설정 (클라우드 VPS):** `wg genkey`와 `wg pubkey` 명령어로 서버의 개인키/공개키 쌍을 생성한다. `wg.conf` 설정 파일에 서버의 개인키, VPN 내부 IP 주소, 그리고 연결을 허용할 각 클라이언트(Peer)의 공개키와 할당할 IP 주소를 등록한다.
    2. **클라이언트 설정 (드론 OpenWRT):**
       - `Network` -> `Interfaces` -> `Add new interface...` 메뉴에서 프로토콜로 `WireGuard VPN`을 선택한다.34
       - 드론 자신의 개인키(마찬가지로 `wg genkey`로 생성)를 `Private Key` 필드에 입력하고, 서버로부터 할당받기로 한 VPN 내부 IP 주소를 `IP Addresses` 필드에 기입한다.
       - `Peers` 탭으로 이동하여 `Add peer`를 클릭하고, 서버의 공개키, 서버의 공인 IP 주소와 리스닝 포트(Endpoint), 그리고 `Allowed IPs`를 `0.0.0.0/0, ::/0`으로 설정한다. `Allowed IPs`를 이렇게 설정하면 드론의 모든 인터넷 트래픽이 WireGuard VPN 터널을 통해 서버로 라우팅되어, 완전한 보안 통신 및 IP 우회가 가능하다.35
       - `Persistent Keepalive` 값을 25초 정도로 설정한다. 이는 주기적으로 작은 패킷을 보내 NAT 및 방화벽의 타임아웃으로 인해 연결이 끊어지는 것을 방지한다.34
    3. **방화벽 설정 (Kill-switch):**
       - `Network` -> `Firewall` 메뉴에서 WireGuard 인터페이스를 위한 새로운 방화벽 존(예: `wg_fw`)을 생성한다.
       - 기존의 `lan` 존 설정에서, `Allow forward to destination zones` 항목의 `wan`을 제거하고 새로 만든 `wg_fw`를 대신 선택한다. 이 설정은 'Kill-switch' 역할을 하여, 만약 WireGuard VPN 연결이 예기치 않게 끊어졌을 때, 드론의 인터넷 트래픽이 암호화되지 않은 일반 WAN으로 유출되는 것을 원천적으로 차단하는 매우 중요한 보안 조치다.34


드론의 눈 역할을 하는 실시간 영상 스트리밍은 가장 많은 대역폭을 요구하는 기능이다. OpenWRT와 `GStreamer`를 활용하면 고품질, 저지연 영상 시스템을 구축할 수 있다.

- **원리 및 핵심 요소:** 드론에 장착된 카메라(USB V4L2 카메라, IP 카메라 등)로부터 영상 데이터를 받아, H.264와 같은 효율적인 코덱으로 압축(인코딩)한 후, RTP(Real-time Transport Protocol)와 같은 네트워크 프로토콜로 패킷화하여 GCS로 전송하는 일련의 과정을 '미디어 파이프라인'이라고 한다. `GStreamer`는 이러한 파이프라인을 구성하고 실행하기 위한 강력하고 유연한 오픈소스 멀티미디어 프레임워크다.38

- **하드웨어 가속의 절대적 중요성:** 1080p 이상의 고해상도 영상을 실시간으로 인코딩하는 작업은 컴패니언 컴퓨터의 CPU에 엄청난 부하를 준다. 소프트웨어 인코더(`x264enc`)만 사용할 경우, 저사양 CC에서는 프레임 드랍이나 시스템 전체의 불안정을 초래할 수 있다.41 따라서 영상 인코딩/디코딩 전용 하드웨어(GPU/VPU)를 활용하는 '하드웨어 가속'은 필수적이다. Jetson 시리즈의 `nvv4l2h264enc`나 Raspberry Pi의 `v4l2h264enc`와 같은 하드웨어 가속 인코더를 사용하면 CPU 점유율을 획기적으로 낮추고 안정적인 고화질 스트리밍이 가능하다.42

- GStreamer 파이프라인 설계 예시 (V4L2 카메라 -> H.264 -> RTP 전송):

  다음은 H.264 출력을 지원하는 USB 카메라(v4l2src)의 영상을 받아 GCS로 RTP 스트리밍하는 기본적인 GStreamer 파이프라인이다.

  ```Bash
  gst-launch-1.0 -v v4l2src device=/dev/video0! \
  video/x-h264,width=1920,height=1080,framerate=30/1! \
  h264parse! \
  rtph264pay config-interval=1 pt=96! \
  udpsink host=<GCS_IP_ADDRESS> port=5600
  ```

  - **파이프라인 분석:**

    1. `v4l2src device=/dev/video0`: V4L2(Video for Linux 2) 드라이버를 통해 `/dev/video0` 장치(USB 카메라)에서 영상 소스를 가져온다.44

    2. `video/x-h264,...`: Caps-filter를 사용하여 소스의 포맷이 H.264, 해상도가 1920x1080, 프레임률이 30fps임을 명시한다.

    3. `h264parse`: H.264 비트스트림을 분석하여 프레임 경계를 찾고, 다운스트림 엘리먼트(예: `rtph264pay`)가 처리하기 용이한 형태로 파싱한다.

    4. `rtph264pay config-interval=1 pt=96`: 파싱된 H.264 프레임을 RFC 3984 표준에 따라 RTP 패킷으로 페이로드화(payload-encode)한다. `config-interval=1` 옵션은 매 I-프레임(키프레임)마다 설정 정보(SPS/PPS)를 함께 보내, 스트림 중간에 접속하는 클라이언트가 빠르게 영상을 디코딩할 수 있도록 돕는 중요한 옵션이다.42

       `pt=96`은 동적 페이로드 타입을 지정한다.

    5. `udpsink host=... port=...`: 생성된 RTP 패킷을 지정된 GCS의 IP 주소와 포트(영상 수신용으로 약속된 포트, 예: 5600)로 UDP를 통해 전송한다.

- **네트워크 문제 해결 및 최적화:** UDP 기반의 RTP 스트리밍은 패킷 손실이나 순서 뒤바뀜에 취약하여 영상 깨짐(corruption) 현상이 발생할 수 있다.

  - **MTU 크기 조절:** 네트워크의 MTU(Maximum Transmission Unit)보다 큰 RTP 패킷은 단편화(fragmentation)되어 전송되는데, 이 과정에서 손실 확률이 높아진다. `rtph264pay` 엘리먼트에 `mtu=700`과 같이 의도적으로 패킷 크기를 작게 설정하면 단편화를 방지하여 패킷 손실률을 크게 줄일 수 있다.47
  - **Jitter Buffer 사용:** 수신 측(GCS)의 GStreamer 파이프라인에 `rtpjitterbuffer` 엘리먼트를 추가하면, 네트워크 지연 변동(jitter)으로 인해 뒤죽박죽 도착한 패킷들을 잠시 버퍼링하여 올바른 순서로 재정렬한 후 디코더로 보내주므로, 훨씬 안정적인 영상을 얻을 수 있다.45

다음은 UAV 영상 스트리밍에 적용 가능한 주요 기술 방식을 비교한 표이다.

| 방식                      | 프로토콜  | 주요 소프트웨어/플러그인            | 장점                                                         | 단점                                                         | 최적 사용 사례                              | 관련 자료 |
| ------------------------- | --------- | ----------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------- | --------- |
| **GStreamer (RTP/UDP)**   | RTP/UDP   | GStreamer (`rtph264pay`, `udpsink`) | 압도적인 저지연, 높은 유연성, 세밀한 파이프라인 제어 가능    | 파이프라인 설계가 복잡하고 전문 지식 요구, 1:1 P2P에 적합    | 저지연 FPV, 실시간 원격 조종, 로봇 비전     | 38        |
| **Nginx-RTMP (RTMP/HLS)** | RTMP, HLS | Nginx + `nginx-rtmp-module`         | 웹 브라우저 호환성 우수, Twitch 등 표준 스트리밍 플랫폼과 동일한 기술 | GStreamer 대비 지연 시간 김(HLS는 수 초 이상), 서버 설정 필요 | 1:N 방송, 인터넷을 통한 대중 공개 스트리밍  | 22        |
| **WebRTC**                | WebRTC    | `go2rtc`, Janus, Kurento            | 브라우저 네이티브 지원(플러그인 불필요), P2P 홀 펀칭으로 NAT/방화벽 우회 용이 | 미디어 서버 설정이 복잡할 수 있음, 브라우저 간 호환성 이슈 존재 | 웹 기반 GCS, 양방향 통신이 필요한 원격 제어 | 48        |



드론 기체에 OpenWRT 기반의 통신 시스템이 구축되었다면, 지상의 조종기, 즉 GCS(Ground Control Station)가 이 네트워크에 접속하여 텔레메트리를 수신하고 영상을 확인할 수 있도록 설정해야 한다.

- 텔레메트리 수신:

  대표적인 GCS인 QGroundControl(QGC)과 Mission Planner는 모두 UDP 또는 TCP를 통해 MAVLink 데이터를 수신하는 기능을 지원한다. 일반적으로 드론과 같이 이동하는 시스템에서는 연결 설정 및 재연결이 간편하고 오버헤드가 적은 UDP 프로토콜이 선호된다.49

  - **설정 방법:** GCS의 통신 링크 설정 메뉴에서 새로운 'UDP' 링크를 추가한다. GCS가 MAVLink 패킷을 수신 대기할 포트 번호(Listening Port)를 지정하는데, 관례적으로 14550번 포트가 널리 사용된다. 드론의 OpenWRT 라우터에서는 `MAVProxy`나 `ser2net`의 출력(destination)을 GCS가 실행 중인 컴퓨터의 IP 주소와 이 포트 번호(예: `192.168.1.100:14550`)로 설정하면 연결이 수립된다.

- 실시간 영상 수신:

  드론에서 GStreamer를 통해 RTP로 스트리밍되는 영상은 GCS에서 동일하게 GStreamer를 통해 수신하거나, GCS에 내장된 영상 수신 기능을 활용하여 볼 수 있다.

  - **QGroundControl 내장 기능:** QGC는 GStreamer 파이프라인을 내부적으로 지원하여 UDP 비디오 스트림을 손쉽게 수신할 수 있다. `Application Settings` -> `General` 탭의 `Video` 섹션에서 `Video Source`를 `UDP Video Stream`으로 선택하고, `UDP Port`를 드론의 GStreamer 파이프라인에서 설정한 포트 번호(예: 5600)로 지정하면 된다.40

  - **GCS 외부에서 GStreamer로 직접 수신:** 개발이나 디버깅 목적으로 GCS와 별개로 영상을 확인하고 싶을 경우, GCS 컴퓨터의 터미널에서 직접 GStreamer 수신 파이프라인을 실행할 수 있다.

    ```Bash
    gst-launch-1.0 udpsrc port=5600 caps="application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264"! rtph264depay! avdec_h264! videoconvert! autovideosink sync=false
    ```

    38

    이 파이프라인은 5600번 UDP 포트로 들어오는 RTP H.264 스트림을 받아(udpsrc), RTP 페이로드를 다시 H.264 데이터로 변환하고(rtph264depay), 디코딩한 후(avdec_h264), 화면에 표시(autovideosink)하는 역할을 한다.


전통적인 RC 송신기 대신, GCS에 연결된 USB 조이스틱이나 게임패드를 사용하여 드론을 직접 조종할 수 있다. 이는 고대역폭의 안정적인 텔레메트리 링크가 확보되었을 때 가능한 강력한 기능이다.

- **원리:** GCS 또는 관련 소프트웨어는 조이스틱의 축(axis)과 버튼 입력을 실시간으로 감지하여 `MANUAL_CONTROL`이라는 MAVLink 메시지로 변환한다. 이 메시지는 텔레메트리 링크를 통해 FC로 전송되며, FC는 이 메시지를 마치 일반 RC 수신기로부터 받은 신호처럼 해석하여 기체를 제어한다.51

- 방법 1: GCS 내장 기능 활용 (QGroundControl / Mission Planner)

  대부분의 GCS는 조이스틱 제어 기능을 내장하고 있다.

  - **설정:** GCS에 조이스틱을 USB로 연결한 후, `Joystick` 설정 메뉴로 이동한다. 여기서 롤(Roll), 피치(Pitch), 요(Yaw), 스로틀(Throttle)에 해당하는 조이스틱 축을 각각 지정하고, 각 축의 최대/최소/중립 값을 시스템에 알려주는 보정(Calibration) 작업을 수행한다. 또한, 특정 버튼에 시동(Arm), 비행 모드 변경(Change Mode) 등의 기능을 매핑할 수 있다.51
  - **활성화:** 설정이 완료되면, `Enable` 버튼을 클릭하여 조이스틱 입력의 MAVLink 메시지 전송을 시작한다.53
  - **PX4 추가 설정:** PX4 펌웨어를 사용하는 경우, 조이스틱 입력을 받기 위해서는 FC의 파라미터 중 `COM_RC_IN_MODE` 값을 `1 - Joystick/No RC Checks`로 설정해야 한다. 이 설정이 없으면 FC는 조이스틱으로부터 오는 `MANUAL_CONTROL` 메시지를 무시한다.51

- 방법 2: MAVProxy의 joystick 모듈 활용

  GCS의 UI에 의존하지 않고, 커맨드라인 기반의 경량화된 조종 시스템을 구축하고 싶을 때 MAVProxy를 사용할 수 있다.

  - **설정:** GCS 컴퓨터 또는 조종기 역할을 할 별도의 임베디드 장치(예: OpenWRT가 설치된 Raspberry Pi)에서 `MAVProxy`를 실행할 때 `joystick` 모듈을 함께 로드한다.

    ```Bash
    mavproxy.py --master=udpin:localhost:14550 --load-module=joystick
    ```

    28

  - **커스터마이징:** `MAVProxy`는 매우 유연한 조이스틱 설정 기능을 제공한다. `~/.mavproxy/joysticks/` 디렉토리 안에 YAML 형식의 설정 파일을 직접 작성하여, 특정 조이스틱 모델의 모든 버튼과 축을 원하는 RC 채널과 기능에 매우 세밀하게 매핑할 수 있다.56

  - **장점:** 이 방식은 GCS를 완전히 배제하고, OpenWRT와 `MAVProxy`가 설치된 소형 컴퓨터와 조이스틱만으로 독립적인 디지털 RC 송신기를 구현할 수 있게 해준다. 이는 시스템을 극도로 경량화하고 커스터마이징할 수 있는 길을 열어준다.

QGroundControl과 같은 GCS에는 MAVLink 데이터를 다른 네트워크 주소로 전달해주는 'MAVLink Forwarding' 기능이 내장되어 있다. 이는 이론적으로 하나의 GCS가 받은 텔레메트리를 다른 GCS나 애플리케이션으로 손쉽게 공유할 수 있게 해줄 것으로 기대된다. 그러나 실제 사용자들의 경험에 따르면, 이 기능은 종종 단방향(one-way)으로만 동작하여 드론에서 GCS로의 데이터는 전달되지만, GCS에서 드론으로 보내는 조종 명령이나 파라미터 요청과 같은 응답 패킷은 전달되지 않는 문제가 빈번하게 발생한다.57 이는 QGC의 포워딩 기능이 완전한 양방향 브릿지가 아니기 때문에 발생하는 근본적인 한계다.

이 문제에 대한 보다 전문적이고 안정적인 해결책은 `mavlink-router` 26나 `MAVProxy` 26와 같은 MAVLink 라우팅 전용 소프트웨어를 사용하는 것이다. 드론 기체 또는 별도의 게이트웨이 장비에 설치된 OpenWRT 라우터를 이러한 전문 MAVLink 라우터로 구성하면, 여러 GCS(예: 조종사용, 임무 관제사용, 개발자 디버깅용)가 하나의 드론에 안정적으로 동시 접속하고 양방향 통신을 수행하는 '다중 제어(Multi-GCS)' 아키텍처를 구현할 수 있다. 이는 GCS의 단순 포워딩 기능이 가진 한계를 극복하고, 진정한 의미의 네트워크 기반 분산 드론 제어 시스템으로 나아가는 핵심적인 단계라 할 수 있다.



지금까지 논의된 OpenWRT의 다양한 기능들을 조합하여, 특정 임무 목적에 최적화된 세 가지 종합 아키텍처를 제안한다.

- **시나리오 1: 장거리 정찰 및 감시 (LTE + WireGuard + 고해상도 스트리밍)**

  - **목표:** 이동통신망을 이용하여 수십 km 이상 떨어진 원격지에서 드론을 제어하고, 고해상도 영상을 실시간으로 전송받는다.
  - **기체 구성:**
    - **하드웨어:** QMI 또는 MBIM 프로토콜을 지원하는 LTE/5G 모뎀 30과 다중 이더넷 포트를 갖춘 OpenWRT 라우터(예: CBUnmanned UAV Router) 12를 탑재한다.
    - **네트워크:** OpenWRT 라우터는 `WireGuard` 클라이언트로 동작하여, 공인 IP를 가진 클라우드 서버의 `WireGuard` 서버와 상시 보안 터널을 유지한다.33
    - **텔레메트리:** FC의 MAVLink 데이터는 `MAVProxy`를 통해 WireGuard 터널을 거쳐 원격 GCS로 안전하게 전송된다.26
    - **영상:** 하드웨어 인코딩을 지원하는 IP 카메라 또는 CC에 연결된 카메라는 `GStreamer`를 통해 1080p 이상의 영상을 RTP로 인코딩하여, 동일한 WireGuard 터널을 통해 GCS로 스트리밍한다.38
  - **GCS 구성:** GCS PC 역시 `WireGuard` 클라이언트로 설정하여 동일한 VPN에 접속한다. GCS는 지정된 포트(예: 14550)로 MAVLink를 수신하고, 다른 포트(예: 5600)로 RTP 영상 스트림을 수신한다.

- **시나리오 2: 저지연 FPV 및 레이싱 (5GHz Wi-Fi + 최적화된 GStreamer)**

  - **목표:** 영상 지연 시간을 수십 밀리초(ms) 수준으로 최소화하여, 실시간 조종이 중요한 FPV(First-Person View) 비행이나 드론 레이싱에 활용한다.

  - **기체 구성:**

    - **하드웨어:** 고출력 5GHz Wi-Fi 모듈을 장착한 경량 OpenWRT 라우터(예: Alfa Tube 2H)를 사용한다.4

    - **네트워크:** OpenWRT는 AP(Access Point) 모드로 동작하고, GCS 측 OpenWRT 라우터는 클라이언트 모드로 동작하여 두 장치가 직접 P2P로 연결된다.

    - **텔레메트리:** 리소스 사용을 최소화하기 위해 `ser2net`을 사용하여 FC의 MAVLink를 가볍게 IP로 브릿징한다.23

    - **영상:** `GStreamer` 파이프라인에서 인코더의 `tune=zerolatency` 옵션을 사용하고, 비트레이트(bitrate)를 낮추어 인코딩 및 전송 지연을 최소화한다.50 수신 측에는 

      `rtpjitterbuffer`를 사용하여 네트워크 불안정성을 보상한다.

  - **GCS 구성:** GCS 측에도 동일한 고출력 Wi-Fi 라우터를 사용하고, 조이스틱을 연결하여 `MANUAL_CONTROL` 메시지를 통해 직접 기체를 조종한다.

- **시나리오 3: 드론 스웜 및 협력 임무 (Mesh Networking + MAVLink 라우팅)**

  - **목표:** 다수의 드론이 자율적으로 통신망을 형성하여, 일부 드론의 통신이 두절되어도 전체 임무를 지속할 수 있는 강인한 군집 시스템을 구축한다.
  - **기체 구성 (각 드론 공통):**
    - **하드웨어:** 모든 드론에 OpenWRT 라우터를 탑재한다.
    - **네트워크:** OpenWRT의 `kmod-batman-adv` 패키지를 설치하여 802.11s 표준 기반의 무선 메시(Mesh) 네트워크를 구성한다.21 드론들은 서로의 통신 범위 안에 들어오면 자동으로 네트워크를 형성하고 데이터를 중계한다.
    - **텔레메트리:** 각 드론의 `MAVProxy`는 로컬 FC의 텔레메트리를 수신하여, 메시 네트워크 전체로 브로드캐스트한다. 이를 통해 모든 드론은 다른 모든 드론의 상태 정보를 실시간으로 공유할 수 있다.26
    - **게이트웨이:** 최소 하나 이상의 드론은 '게이트웨이(Gateway)' 역할을 수행하도록 LTE 모뎀을 추가로 장착한다. 이 게이트웨이 드론은 메시 네트워크와 원격 GCS 간의 통신을 중계한다.21
  - **GCS 구성:** GCS는 게이트웨이 드론의 VPN을 통해 전체 스웜 네트워크에 접속한다. GCS에서는 전체 스웜의 상태를 종합적으로 모니터링하고, 개별 드론 또는 특정 그룹에게 임무 명령을 내릴 수 있다.


OpenWRT는 더 이상 단순한 공유기 커스텀 펌웨어가 아니다. 본 보고서에서 심층적으로 분석한 바와 같이, OpenWRT는 UAV 시스템의 통신, 제어, 그리고 연산 능력을 기존의 한계에서 벗어나 극적으로 확장할 수 있는 강력하고 성숙한 임베디드 리눅스 플랫폼이다. OpenWRT가 제공하는 모듈성, 경량 설계, 그리고 강력한 네트워킹 기능들은 LTE/5G를 통한 BVLOS 통신, WireGuard를 이용한 종단 간 암호화, GStreamer를 활용한 고화질 저지연 영상 스트리밍, 그리고 메시 네트워킹 기반의 드론 스웜 구축 등, 표준 드론 시스템으로는 구현하기 어렵거나 불가능했던 고급 기능들을 현실로 만들어준다. 이는 드론을 단순히 하늘을 나는 '비행체'에서, 인터넷에 연결되어 데이터를 생성하고 처리하는 '지능형 네트워크 엣지 노드(Intelligent Network Edge Node)'로 변모시키는 핵심적인 기술이라 결론지을 수 있다.

향후 연구는 OpenWRT의 이러한 강력한 기반 위에서 더욱 발전된 기술을 접목하는 방향으로 나아가야 할 것이다. 첫째, OpenWRT 기반 컴패니언 컴퓨터에 Docker나 기타 컨테이너 기술을 활용하여 AI/ML 모델을 손쉽게 배포하고, 비행 중에 수집되는 데이터를 실시간으로 처리하는 '엣지 AI' 분야로의 확장이 기대된다.22 둘째, 본격적으로 상용화될 5G 이동통신망의 초저지연(URLLC) 및 초연결(mMTC) 특성을 OpenWRT 기반 드론 통신 시스템에 접목하여, 원격 수술이나 정밀 제어와 같은 미션 크리티컬한 임무의 신뢰성과 성능을 한 단계 더 끌어올리는 연구가 필요하다. OpenWRT의 검증된 안정성과 무한한 유연성은 이러한 미래 기술들을 실제 드론 시스템에 통합하고 검증하는 데 가장 이상적인 테스트베드를 제공할 것이다.


1. OpenWrt: 임베디드 디바이스용 리눅스 배포판 활용 - 재능넷, accessed August 11, 2025, https://www.jaenung.net/tree/1188
2. OpenWrt - 위키백과, 우리 모두의 백과사전, accessed August 11, 2025, https://ko.wikipedia.org/wiki/OpenWrt
3. OpenWrt - 나무위키, accessed August 11, 2025, https://namu.wiki/w/OpenWrt
4. (PDF) "Design of a Communication Architecture for Unmanned Aerial Vehicle (UAV) Swarm Networks" - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/335107014_Design_of_a_Communication_Architecture_for_Unmanned_Aerial_Vehicle_UAV_Swarm_Networks
5. operating system architecture - OpenWRT, accessed August 11, 2025, https://openwrt.org/docs/techref/architecture
6. [OpenWrt Wiki] Documentation, accessed August 11, 2025, https://openwrt.org/docs/start
7. Simple Overview of ArduPilot Operation - Copter documentation, accessed August 11, 2025, https://ardupilot.org/copter/docs/common-basic-operation.html
8. Companion Computers - Copter documentation - ArduPilot, accessed August 11, 2025, https://ardupilot.org/copter/docs/common-companion-computers.html
9. Companion Computers - Dev documentation - ArduPilot, accessed August 11, 2025, https://ardupilot.org/dev/docs/companion-computers.html
10. Companion Computer for Pixhawk Series | PX4 User Guide (v1.12), accessed August 11, 2025, https://docs.px4.io/v1.12/en/companion_computer/pixhawk_companion.html
11. Understanding Architecture with Companion Computer and Dronekit ..., accessed August 11, 2025, https://discuss.ardupilot.org/t/understanding-architecture-with-companion-computer-and-dronekit-python/54192
12. OpenWRT for UAVs! : r/openwrt - Reddit, accessed August 11, 2025, https://www.reddit.com/r/openwrt/comments/1e7ikuk/openwrt_for_uavs/
13. Companion Computer / GitBook - ArduSub, accessed August 11, 2025, https://www.ardusub.com/introduction/hardware-options/required-hardware/companion-computer.html
14. OpenWRT UAV Router - About CBUnmanned | Wiki, accessed August 11, 2025, https://wiki.cbunmanned.com/wiki/products/openwrt-uav-router
15. Companion Computers - DroneKit Python 2.4.0 documentation, accessed August 11, 2025, https://dronekit-python.readthedocs.io/en/latest/develop/companion-computers.html
16. Welcome to the ArduPilot Development Site - Dev documentation, accessed August 11, 2025, https://ardupilot.org/dev/
17. Companion Computers for Sub - Sub documentation - ArduPilot, accessed August 11, 2025, https://ardupilot.org/sub/docs/sub-companions-computers.html
18. Using a Companion Computer with Pixhawk Controllers | PX4 Guide (main), accessed August 11, 2025, https://docs.px4.io/main/en/companion_computer/pixhawk_companion
19. [OpenWrt Wiki] Network Components, accessed August 11, 2025, https://openwrt.org/docs/guide-user/network/architecture/components
20. CGNAT을 해결하고 OpenWRT 라우터와 ZeroTier를 사용해서 홈 ..., accessed August 11, 2025, https://www.reddit.com/r/openwrt/comments/i8ejuu/guide_to_get_over_cgnat_and_ssh_into_home_router/?tl=ko
21. Mesh networking: A guide to using free and open-source software with common hardware, accessed August 11, 2025, https://cgomesu.com/blog/Mesh-networking-openwrt-batman/
22. Streaming RTMP with OpenWRT - Jared.geek.nz, accessed August 11, 2025, https://jared.geek.nz/2023/11/streaming-rtmp-with-openwrt/
23. Mavlink Through Serial - About CBUnmanned | Wiki, accessed August 11, 2025, https://wiki.cbunmanned.com/wiki/products/openwrt-uav-router/mavlink-through-serial
24. Using ser2net to remotely access a serial terminal or port - Tutorials - Mender Hub, accessed August 11, 2025, https://hub.mender.io/t/using-ser2net-to-remotely-access-a-serial-terminal-or-port/7807
25. Yet another Ser2Net tutorial - Wifizoo, accessed August 11, 2025, https://wifizoo.org/2023/05/12/yet-another-ser2net-tutorial/
26. MAVLink routing with a Router software - Blog - ArduPilot Discourse, accessed August 11, 2025, https://discuss.ardupilot.org/t/mavlink-routing-with-a-router-software/82138
27. Download and Installation - MAVProxy documentation - ArduPilot, accessed August 11, 2025, https://ardupilot.org/mavproxy/docs/getting_started/download_and_installation.html
28. Quickstart - MAVProxy documentation - ArduPilot, accessed August 11, 2025, https://ardupilot.org/mavproxy/docs/getting_started/quickstart.html
29. MAVProxy documentation - ArduPilot, accessed August 11, 2025, https://ardupilot.org/mavproxy/
30. [OpenWrt Wiki] How to use LTE modem in QMI mode for WAN connection, accessed August 11, 2025, https://openwrt.org/docs/guide-user/network/wan/wwan/ltedongle
31. [OpenWrt Wiki] Bridge mode, accessed August 11, 2025, https://openwrt.org/docs/guide-user/network/wan/bridge-mode
32. Using OpenWRT as an LTE half-bridge - Reddit, accessed August 11, 2025, https://www.reddit.com/r/openwrt/comments/1hirylw/using_openwrt_as_an_lte_halfbridge/
33. openwrt.org, accessed August 11, 2025, https://openwrt.org/docs/guide-user/services/vpn/wireguard/basics
34. WireGuard Setup guide for OpenWrt - IVPN, accessed August 11, 2025, https://www.ivpn.net/setup/router/openwrt-wireguard/
35. OpenWRT WireGuard VPN Server Tutorial - Reddit, accessed August 11, 2025, https://www.reddit.com/r/openwrt/comments/bahhua/openwrt_wireguard_vpn_server_tutorial/
36. OpenWRT WireGuard Simple Home VPN Tutorial - YouTube, accessed August 11, 2025, https://www.youtube.com/watch?v=BbDtc0AhGWU&pp=0gcJCfwAo7VqN5tD
37. Wireguard OpenWRT Travel Router Tutorial - Script Tactics, accessed August 11, 2025, https://scripttactics.com/self-hosted/openwrt-travel-router-tut/
38. GStreamer for video streaming - Blueye SDK, accessed August 11, 2025, https://blueye-robotics.github.io/blueye.sdk/v2.1/video/gstreamer-for-video-streaming/
39. How to Stream Live Video From Your Drone with voxl-streamer | ModalAI, Inc., accessed August 11, 2025, https://www.modalai.com/blogs/blog/how-to-stream-live-video-from-your-drone-with-voxl-streamer
40. Video Streaming | Drone Software Development - GitBook, accessed August 11, 2025, https://theiotlearninginitiative.gitbook.io/bitol/virtual-drone-solution/features/video-streaming
41. Topic: Record and stream video webcam? - OpenWrt Forum Archive, accessed August 11, 2025, https://forum.archive.openwrt.org/viewtopic.php?id=64628
42. Audio/Video over RTP With GStreamer (Linux) - Toradex Developer Center, accessed August 11, 2025, https://developer.toradex.com/linux-bsp/application-development/multimedia/audiovideo-over-rtp-with-gstreamer-linux/
43. Low latency h264 rtp video using gstreamer - Raspberry Pi Forums, accessed August 11, 2025, https://forums.raspberrypi.com/viewtopic.php?t=349669
44. Capturing h.264 stream from camera with Gstreamer - Stack Overflow, accessed August 11, 2025, https://stackoverflow.com/questions/15787967/capturing-h-264-stream-from-camera-with-gstreamer
45. How to setup gstreamer on raspberry Pi and client for rtp with H264-capable webcam?, accessed August 11, 2025, https://stackoverflow.com/questions/40921983/how-to-setup-gstreamer-on-raspberry-pi-and-client-for-rtp-with-h264-capable-webc
46. How to stream h264 video from web-cam to network? - Unix & Linux Stack Exchange, accessed August 11, 2025, https://unix.stackexchange.com/questions/57483/how-to-stream-h264-video-from-web-cam-to-network
47. [Solved] Stream H264 over RTP using nvv4l2h264enc using Gstreamer (Getting corrupted frames) - Jetson Nano - NVIDIA Developer Forums, accessed August 11, 2025, https://forums.developer.nvidia.com/t/solved-stream-h264-over-rtp-using-nvv4l2h264enc-using-gstreamer-getting-corrupted-frames/159319
48. AlexxIT/go2rtc: Ultimate camera streaming application with support RTSP, RTMP, HTTP-FLV, WebRTC, MSE, HLS, MP4, MJPEG, HomeKit, FFmpeg, etc. - GitHub, accessed August 11, 2025, https://github.com/AlexxIT/go2rtc
49. Serial Interface Guide - Doodle Labs Technical Library, accessed August 11, 2025, https://techlibrary.doodlelabs.com/serial-interface-guide
50. gstreamer send and receive h264 rtp stream - GitHub Gist, accessed August 11, 2025, https://gist.github.com/esrever10/7d39fe2d4163c5b2d7006495c3c911bb
51. Joystick Setup | QGC Guide (master) - QGroundControl Guide, accessed August 11, 2025, https://docs.qgroundcontrol.com/master/en/qgc-user-guide/setup_view/joystick.html
52. How do I use MAVLink Endpoints to get gamepad data? - Blue Robotics Community Forums, accessed August 11, 2025, https://discuss.bluerobotics.com/t/how-do-i-use-mavlink-endpoints-to-get-gamepad-data/16671
53. Joystick/Gamepad - Copter documentation - ArduPilot, accessed August 11, 2025, https://ardupilot.org/copter/docs/common-joystick.html
54. QGC does not send required joystick state (physical or virtual) for arming / Issue #11709 / mavlink/qgroundcontrol - GitHub, accessed August 11, 2025, https://github.com/mavlink/qgroundcontrol/issues/11709
55. Joystick Input - MAVProxy documentation - ArduPilot, accessed August 11, 2025, https://ardupilot.org/mavproxy/docs/modules/joystick.html
56. MAVProxy/docs/JOYSTICKS.md at master - GitHub, accessed August 11, 2025, https://github.com/ArduPilot/MAVProxy/blob/master/docs/JOYSTICKS.md
57. MAVLink Forwarding - PX4 Discussion Forum, accessed August 11, 2025, https://discuss.px4.io/t/mavlink-forwarding/32443
58. mavlink forwarding is one-way / Issue #10682 - GitHub, accessed August 11, 2025, https://github.com/mavlink/qgroundcontrol/issues/10682
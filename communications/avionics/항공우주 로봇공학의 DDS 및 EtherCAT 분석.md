---
layout: page
title: 항공우주 로봇공학의 DDS 및 EtherCAT 분석
permalink: /communications/avionics/항공우주 로봇공학의 DDS 및 EtherCAT 분석
---


최첨단 항공우주 로봇 시스템의 아키텍처를 분석할 때, 표면적으로 모순처럼 보이는 설계 선택이 관찰되곤 합니다. 사용자가 제기한 질문은 바로 이 지점을 정확히 짚고 있습니다. 즉, 높은 신뢰성을 목표로 하는 시스템이 한편으로는 데이터 분산 서비스(DDS)를 채택하여 소프트웨어 단일 장애점(SPOF)을 제거하면서, 다른 한편으로는 마스터-슬레이브 구조를 가진 EtherCAT 필드버스를 도입하여 잠재적인 하드웨어 SPOF를 만드는 것처럼 보인다는 것입니다.1 이 관찰은 매우 타당하며, 이는 단순한 모순이 아니라 현대 로봇공학의 정교하고 다층적인 신뢰성 공학을 이해하는 중요한 출발점입니다.

본 보고서의 핵심 논지는 DDS와 EtherCAT의 동시 채택이 의도적이고 시너지 효과를 내는 아키텍처 패턴이라는 것입니다. 이 두 기술은 제어 스택의 서로 다른, 특정 계층에서 최적의 솔루션으로 선택됩니다. DDS는 복잡하고 비결정적인 상위 레벨 소프트웨어 상호작용을 위한 분산형 견고성을 제공하는 반면, EtherCAT은 하위 레벨의 실시간 하드웨어 제어를 위한 독보적인 결정론적 성능을 보장합니다. EtherCAT의 본질적인 SPOF는 성능을 위한 계산된 트레이드오프이며, 이는 성숙하고 체계적인 이중화 솔루션들을 통해 시스템적으로 완화됩니다.2

궁극적으로 시스템 신뢰성은 단일 속성이 아니라, 각 계층에 적합한 각기 다른 기술을 사용하여 설계되는 공학적 특성입니다. 본 보고서는 각 기술을 개별적으로 분석하고, 이들의 통합 방법과 미션 크리티컬 시스템에서의 사용을 지배하는 고급 고려사항들을 상세히 설명함으로써 이 패러독스를 해결하고자 합니다.

| **속성**                | **DDS (Data Distribution Service)**          | **EtherCAT (Ethernet for Control Automation Technology)** |
| ----------------------- | -------------------------------------------- | --------------------------------------------------------- |
| **통신 모델**           | P2P (Peer-to-Peer), 분산형 1                 | 마스터-슬레이브, 중앙집중형 5                             |
| **주요 역할**           | 소프트웨어 미들웨어 (프로세스 간 통신) 6     | 하드웨어 필드버스 (컨트롤러-디바이스 통신) 7              |
| **일반적 지연/주기**    | 밀리초(ms) 단위 (애플리케이션에 따라 다름) 8 | 마이크로초(µs) 단위 (예: <125µs) 9                        |
| **핵심 강점**           | 데이터 중심성, QoS 유연성, 확장성 1          | 결정론, 정밀 동기화, 고성능 2                             |
| **내재된 SPOF**         | 설계상 없음 (마스터리스) 1                   | 마스터 디바이스 2                                         |
| **주요 SPOF 완화 전략** | 해당 없음                                    | 케이블, 마스터, 컨트롤러 이중화 3                         |

*표 1: 로봇공학 관점에서의 DDS와 EtherCAT 비교 개요. 이 표는 두 기술이 서로 다른 문제 영역을 해결하는 상호 보완적인 역할을 수행함을 명확히 보여줍니다.*


최신 로봇 시스템 아키텍처의 핵심적인 발전 중 하나는 소프트웨어의 단일 장애점을 제거하려는 노력이며, 이는 ROS 2(Robot Operating System 2)의 설계 철학에서 명확하게 드러납니다.


이전 세대인 ROS 1은 `roscore`라는 마스터 노드에 크게 의존했습니다.1 이 중앙집중형 구조는 시스템의 모든 노드 간의 이름 등록 및 연결 설정을 관리했습니다. 이는 개발 초기에는 편리했지만, `roscore`가 중단될 경우 전체 통신 그래프가 붕괴되는 치명적인 단일 장애점을 만들었습니다. 시스템의 어느 한 부분에서 발생한 네트워크 문제나 마스터 노드의 장애가 전체 로봇 시스템의 마비로 이어질 수 있었습니다.1

이러한 한계를 극복하기 위해, ROS 2는 통신 미들웨어로 DDS(Data Distribution Service)를 채택했습니다. DDS는 본질적으로 분산형 아키텍처를 가집니다. DDS 기반 시스템에는 중앙 마스터가 존재하지 않습니다.1 각 노드(DDS에서는 'Participant'라고 함)는 네트워크에 참여하여 다른 노드들을 자동으로 발견하고, P2P(Peer-to-Peer) 방식으로 직접 통신합니다. 이 "마스터리스(masterless)" 아키텍처는 `roscore`와 같은 중앙 집중적 장애 요소를 원천적으로 제거하여 시스템의 견고성을 극적으로 향상시킵니다.14


DDS는 통신을 특정 엔드포인트 간의 메시지 전달이 아닌, 공유된 "글로벌 데이터 공간(global data space)"을 중심으로 구성합니다.11 이 패러다임에서 데이터는 고유한 이름과 타입을 가진 "토픽(Topic)"으로 정의됩니다.1 데이터를 생성하는 노드는 '퍼블리셔(Publisher)'가 되어 특정 토픽에 데이터를 발행하고, 데이터가 필요한 노드는 '서브스크라이버(Subscriber)'가 되어 관심 있는 토픽을 구독합니다. DDS 미들웨어는 이들 사이의 라우팅, 검색, 데이터 전달의 모든 복잡성을 자동으로 처리합니다.1

이러한 데이터 중심 모델은 애플리케이션 간의 결합도를 낮추어 모듈성과 확장성을 크게 향상시킵니다. 예를 들어, 새로운 센서 시스템을 로봇에 추가할 때, 해당 시스템이 표준화된 토픽(예: `/sensor/lidar/points`)에 데이터를 발행하기만 하면, 기존의 내비게이션, 로깅, 시각화 노드들은 자신의 코드를 전혀 변경하지 않고도 이 새로운 데이터 소스를 즉시 구독하고 활용할 수 있습니다.


DDS의 가장 강력한 기능 중 하나는 풍부한 서비스 품질(QoS) 정책을 제공한다는 점입니다.1 이는 단순히 데이터를 전송하는 것을 넘어, 데이터의 전송 *방식*을 정밀하게 제어할 수 있게 해줍니다. 이는 고신뢰성 시스템 설계의 핵심 요소입니다.11 주요 QoS 정책은 다음과 같습니다.

- **신뢰성 (Reliability):** `RELIABLE` 정책은 모든 데이터가 수신자에게 반드시 전달되도록 보장하며, 손실 시 재전송합니다. 이는 로봇의 제어 명령이나 상태 머신 전환과 같은 중요한 데이터에 필수적입니다. 반면, `BEST_EFFORT` 정책은 데이터를 한 번만 전송하며, 고주파 센서 데이터(예: 카메라 이미지)처럼 최신성이 중요하여 지연보다 패킷 손실이 더 나은 경우에 적합합니다.17
- **내구성 (Durability):** 늦게 참여하는 노드가 이전에 발행된 데이터를 수신할 수 있도록 합니다. 예를 들어, 시스템 재부팅 후 상태를 복원해야 하는 노드는 이 정책을 통해 마지막으로 알려진 로봇의 위치나 상태 정보를 즉시 얻을 수 있습니다.
- **마감일 (Deadline) 및 수명 (Lifespan):** 데이터의 시간적 제약을 강제합니다. `Deadline`은 데이터가 특정 주기 내에 수신되어야 함을 보장하며, 이를 위반하면 시스템에 알립니다. `Lifespan`은 데이터가 유효한 시간을 설정하여 오래된 데이터가 사용되는 것을 방지합니다.

이러한 QoS의 유연성 덕분에 시스템 아키텍트는 단일 로봇 내에서 수십 개의 서로 다른 데이터 스트림에 대해 각각 최적의 통신 동작을 맞춤 설정하여 신뢰성과 성능을 동시에 최적화할 수 있습니다.


DDS를 선택하는 이유는 단순히 소프트웨어 SPOF를 제거하는 것 이상의 의미를 가집니다. DDS는 본질적으로 데이터 중심의 통합 프레임워크(integration framework)입니다.15 항공우주 시스템은 비행 제어, 내비게이션, 임무 컴퓨터, 지상 관제소 등 수많은 공급업체의 다양한 소프트웨어 구성 요소로 이루어진 극도로 복잡한 시스템입니다.

이러한 환경에서 ROS 1과 같은 중앙집중형 아키텍처는 통합의 병목 현상과 거대한 복잡성을 야기할 수 있습니다. 반면, DDS의 분산형 자동 검색 기능과 데이터 중심 모델은 이러한 이기종 컴포넌트들을 원활하게 통합할 수 있게 해줍니다.11 새로운 페이로드 시스템이 추가될 때, 해당 시스템이 올바른 토픽과 데이터 타입으로 정보를 발행하기만 하면, 기존의 모든 관련 시스템은 즉시 이 정보를 활용할 수 있습니다. 이러한 "플러그 앤 플레이" 기능은 개발 시간, 비용, 그리고 통합 리스크를 획기적으로 줄여주는 전략적 이점을 제공합니다.11

결론적으로, DDS의 채택은 단일 장애점 제거라는 전술적 목표를 넘어, 확장 가능하고 유지보수 용이하며 진화 가능한 소프트웨어 아키텍처를 구현하기 위한 전략적 결정입니다. 이는 시스템 견고성의 더 높은 차원을 의미합니다.


소프트웨어 계층에서 DDS가 분산형 유연성을 제공한다면, 하드웨어 제어 계층에서는 EtherCAT이 절대적인 결정론과 성능을 제공합니다. EtherCAT이 항공우주와 같은 고성능 분야에서 선택되는 이유는 그 독특한 작동 원리에 있습니다.


EtherCAT의 핵심은 "실시간 처리(processing-on-the-fly)" 원리입니다.2 표준 이더넷 기반 프로토콜(예: EtherNet/IP)에서는 마스터가 각 슬레이브에게 개별적인 패킷을 보내고, 각 슬레이브는 패킷을 수신하여 처리한 후 응답 패킷을 다시 전송합니다. 이 과정은 각 노드에서 소프트웨어 처리로 인한 지연과 지터(jitter)를 발생시킵니다.

반면, EtherCAT은 완전히 다른 방식을 사용합니다. EtherCAT 마스터는 단일 이더넷 프레임(텔레그램)을 전송하며, 이 프레임은 네트워크의 모든 슬레이브 디바이스를 순차적으로 통과합니다.10 각 슬레이브 디바이스에는 EtherCAT Slave Controller(ESC)라는 전용 하드웨어 칩이 내장되어 있습니다. 이 ESC는 프레임이 자신을 통과하는 *순간*, 자신에게 할당된 출력 데이터를 읽고 자신의 입력 데이터를 프레임에 삽입합니다.5 이 모든 과정은 하드웨어에서 나노초 수준의 지연만으로 처리되며, 프레임이 다음 노드로 즉시 전달됩니다. 네트워크의 마지막 노드는 프레임을 다시 마스터에게 반송합니다. 이 구조에서 프레임 전송을 시작할 수 있는 권한은 오직 마스터에게만 주어지므로, 예측 불가능한 지연이 원천적으로 차단됩니다.2


이 "on-the-fly" 방식은 다축 로봇 팔과 같은 정밀 모션 제어에 필수적인 성능 특성을 제공합니다.

- **극도로 짧은 사이클 타임:** EtherCAT은 125µs(마이크로초), 심지어 30µs까지의 매우 짧은 사이클 타임을 달성할 수 있습니다.9 이를 통해 수백 개의 축을 kHz 단위의 주파수로 동시에 업데이트할 수 있습니다. 이는 일반적인 이더넷/IP 구현보다 수십 배 빠른 속도입니다.22
- **낮은 지터와 정밀 동기화:** EtherCAT의 분산 클록(Distributed Clocks, DC) 메커니즘은 IEEE 1588 정밀 시간 프로토콜의 경량화 버전으로, 네트워크의 모든 슬레이브 클록을 마스터 클록에 마이크로초 미만, 종종 나노초 수준의 정확도로 동기화합니다.2 이는 여러 모터를 정밀하게 연동시켜야 하는 전자 기어링이나 캠 동기화와 같은 애플리케이션에 절대적으로 필요합니다.


EtherCAT은 단순한 공장 자동화 프로토콜이 아닙니다. 그 성능과 신뢰성은 가장 까다로운 미션 크리티컬 환경에서 이미 입증되었습니다.

- **NASA 화성 탐사 로버 "퍼서비어런스":** 이 로버에 장착된 2미터 길이의 로봇 팔은 화성 표면에서 샘플을 채취하는 핵심 임무를 수행하며, 그 정밀한 제어는 EtherCAT을 통해 이루어집니다.25
- **MDA 캐나다암3 (Canadarm3):** 달 궤도를 도는 우주정거장 '루나 게이트웨이'에 설치될 차세대 로봇 팔인 캐나다암3의 통신 백본으로 EtherCAT이 채택되었습니다.25
- **독일 항공우주 센터(DLR):** DLR은 수년 동안 다양한 우주 로봇 프로젝트에 EtherCAT을 사용해왔으며, 국제우주정거장(ISS) 내부에도 EtherCAT 기반 시스템이 이미 운영되고 있습니다.25
- **에어버스(Airbus):** 항공기 날개의 고양력 장치(플랩/슬랫) 테스트 설비와 같이 수백 개의 신호를 실시간으로 처리해야 하는 복잡한 시스템의 제어에 EtherCAT이 사용됩니다.26


EtherCAT의 설계자들은 분산형 필드버스를 만들 수도 있었지만, 의도적으로 마스터-슬레이브 아키텍처를 선택했습니다. 이는 중요한 공학적 판단을 반영합니다. 필드버스 레벨에서 해결해야 할 핵심 문제는 유연한 데이터 라우팅이나 노드 검색(DDS가 해결하는 문제)이 아니라, 모터 드라이브나 I/O 모듈과 같은 고정된 하드웨어 장치 집합에 대해 절대적이고 예측 가능한 시간 내에 데이터를 전달하는 것입니다.

어떤 분산형 프로토콜이든 통신을 위해서는 협상, 중재, 충돌 감지/회피와 같은 메커니즘이 필요하며, 이는 모두 통신 시간에 비결정성, 즉 지터(jitter)를 유발합니다. EtherCAT은 제어 권한을 단일 마스터에 중앙집중화함으로써 네트워크로 인한 모든 지터 발생 요인을 제거했습니다. 마스터는 시간의 유일한 중재자입니다. 마스터가 프레임을 보내면, 프레임은 완벽하게 예측 가능한 시간 내에 돌아옵니다.

이러한 아키텍처 선택은 분산화를 희생하여 거의 완벽한 결정론을 얻는 것입니다. 이 선택의 "비용"이 바로 단일 장애점(SPOF)의 생성입니다. 따라서 EtherCAT 마스터의 존재는 설계상의 실수가 아니라, 그 성능의 초석입니다. 이제 엔지니어링의 과제는 마스터를 *제거*하는 것이 아니라, 마스터 장애의 *위험을 완화*하는 것이 됩니다. 이는 다음 장에서 다룰 이중화 메커니즘의 필요성으로 직접 이어집니다.


EtherCAT 마스터가 단일 장애점이라는 점은 잘 알려진 사실이며, EtherCAT 기술 생태계는 이를 해결하기 위해 다층적인 이중화 전략을 개발하고 성숙시켜 왔습니다. 시스템의 요구 가용성에 따라 단순한 케이블 보호부터 전체 컨트롤러의 완벽한 페일오버에 이르기까지 다양한 수준의 결함 허용(fault tolerance)을 구현할 수 있습니다.

| **메커니즘**                                                 | **원리**                  | **보호 대상**                          | **하드웨어 요구사항**                          | **소프트웨어 요구사항**      | **장애 극복 시간**              |
| ------------------------------------------------------------ | ------------------------- | -------------------------------------- | ---------------------------------------------- | ---------------------------- | ------------------------------- |
| **케이블 이중화**                                            | 링 토폴로지 구성 2        | 단일 케이블 단선 또는 슬레이브 장애 27 | 마스터의 두 번째 NIC 28                        | 간단한 설정 활성화 29        | 끊김 없음 (Zero data loss) 4    |
| **마스터 이중화 (핫 스탠바이)**                              | 액티브/패시브 마스터 쌍 3 | 마스터 소프트웨어 또는 하드웨어 장애   | 백업 마스터 장치                               | 이중화 기능 팩 30            | 애플리케이션에 따라 다름 (빠름) |
| **컨트롤러 이중화 (예: Beckhoff TF1100)**                    | 동기화된 듀얼 컨트롤러 31 | 전체 컨트롤러(IPC) 장애                | 두 번째 컨트롤러 + 동기화 링크 (예: CU2508) 31 | 특정 라이선스 (예: TF1100) 4 | 끊김 없음 (Bumpless transfer) 4 |
| *표 2: EtherCAT 결함 허용 메커니즘. 이 표는 SPOF 위험을 체계적으로 해결하기 위한 계층적 접근 방식을 보여줍니다.* |                           |                                        |                                                |                              |                                 |


케이블 이중화는 가장 기본적이고 널리 사용되는 EtherCAT의 결함 허용 기능입니다.

- **원리:** 네트워크는 일반적으로 라인(line) 형태로 배선되지만, 마지막 슬레이브 디바이스의 출력 포트를 마스터의 두 번째 이더넷 포트에 연결하여 물리적인 링(ring)을 구성합니다.2
- **정상 작동:** 정상 작동 시, 마스터는 두 개의 포트에서 동일한 프레임을 동시에 전송합니다. 주 포트(primary port)에서 나간 프레임은 슬레이브들을 순방향으로 통과하며 데이터를 처리합니다. 반면, 이중화 포트(redundancy port)에서 나간 프레임은 슬레이브들을 역방향으로 통과하며, 이 때는 데이터 처리가 일어나지 않고 단순히 전달되기만 합니다.13
- **장애 시나리오:** 만약 두 슬레이브 사이의 케이블이 끊어지거나 한 슬레이브에 장애가 발생하면, 단선 지점 양쪽의 슬레이브들은 열린 포트를 감지하고 수신된 프레임을 즉시 자신의 다른 포트로 되돌려 보냅니다 (자동 루프백). 결과적으로 마스터는 이제 두 개의 라인 토폴로지로부터 데이터를 수신하게 되며, 이 두 라인을 합치면 단선 지점을 제외한 모든 슬레이브와의 통신이 유지됩니다. 통신은 중단 없이 모든 정상 슬레이브와 계속되며, 이는 단일 장애에 대한 완벽한 허용성을 제공합니다.13
- **구현:** 이 기능은 두 개의 이더넷 포트를 가진 마스터와 TwinCAT과 같은 마스터 소프트웨어에서 간단한 설정 옵션을 활성화하는 것만으로 구현할 수 있습니다.28


이 단계는 마스터 디바이스 자체의 장애에 대응합니다.

- **원리:** 주 마스터(active master) 외에 예비 마스터(standby master)를 네트워크에 연결합니다.3
- **정상 작동:** 주 마스터가 네트워크를 능동적으로 제어합니다. 예비 마스터는 네트워크 트래픽을 수동적으로 감청하거나, 전용 하트비트(heartbeat) 링크를 통해 주 마스터의 상태를 지속적으로 모니터링합니다.
- **장애 시나리오:** 주 마스터가 하드웨어 결함이나 소프트웨어 충돌로 인해 장애를 일으키면, 예비 마스터는 이를 감지합니다. 감지 즉시, 예비 마스터는 주 마스터의 네트워크 ID(예: MAC 주소)를 인계받아 네트워크 제어를 시작합니다. 슬레이브들은 마스터가 변경된 것을 인지하지 못하거나, 예상된 ID를 가진 새로운 마스터로 인식하여 중단 없이 작동을 계속합니다. ETG 문서에서는 이를 "비활성 마스터가 활성 마스터의 활동을 인계받고, 디바이스들은 계속 작동 상태를 유지한다"고 설명합니다.3
- **구현:** 이 기능은 일반적으로 EtherCAT 마스터 스택의 추가 기능 팩(feature pack)으로 제공되며, 예비 마스터 역할을 할 두 번째 하드웨어 장치와 핫 스탠바이 프로토콜을 지원하는 소프트웨어가 필요합니다.3


가장 높은 수준의 신뢰성을 제공하는 이 솔루션은 단순히 EtherCAT 마스터 스택만이 아닌, 컨트롤러(IPC) 전체를 이중화합니다. Beckhoff의 TF1100 기능이 이의 대표적인 예입니다.4

- **아키텍처:** 두 개의 표준 산업용 PC(IPC)가 동일한 PLC 및 모션 제어 프로그램을 절대적인 동기 상태로 실행합니다.31
- **동기화:** 두 IPC 간의 동기화는 전용 하드웨어가 아닌, 표준 고속 기가비트 이더넷 연결을 통해 이루어집니다. Beckhoff는 이 링크를 위해 프로세스 데이터를 실시간으로 교환하는 고도로 최적화된 독자적인 프로토콜을 개발했습니다. 이 소프트웨어 기반 접근 방식은 값비싼 전용 동기화 하드웨어가 필요 없다는 큰 장점이 있습니다.31
- **CU2508의 역할:** 각 컨트롤러는 CU2508 실시간 이더넷 포트 멀티플라이어에 연결됩니다. 그리고 이 두 CU2508 사이에 두 번째 독립적인 통신 링크가 설정됩니다. 이 링크의 핵심 기능은 '타이브레이커(tie-breaker)' 역할입니다. 만약 두 IPC 간의 주 동기화 링크가 끊어지면, 이 보조 링크를 통해 각 컨트롤러는 상대방 *컨트롤러*가 고장 난 것인지, 아니면 단순히 *링크*만 끊어진 것인지를 판단할 수 있습니다. 이는 두 컨트롤러가 동시에 마스터가 되려고 하는 "스플릿 브레인(split-brain)" 시나리오를 방지하는 매우 중요한 안전장치입니다.31
- **장애 시나리오:** 주 IPC에 장애가 발생하면, 이미 완벽하게 동기화되어 최신 프로세스 이미지를 가지고 있는 보조 IPC가 즉시 EtherCAT 버스의 제어권을 인계받습니다. 이 전환은 "범프리스(bumpless)", 즉 아무런 충격이나 중단 없이 이루어져 프로세스가 원활하게 계속되도록 보장합니다.4

이처럼 EtherCAT의 단일 장애점 위험은 결코 간과되지 않았으며, 시스템의 중요도와 요구 가용성에 맞춰 선택할 수 있는 성숙하고 강력한 다층적 이중화 솔루션을 통해 체계적으로 관리됩니다.


DDS와 EtherCAT은 서로 다른 계층에서 작동하며, 이 둘을 효과적으로 통합하는 것이 현대 로봇 아키텍처의 핵심입니다. 이 통합은 명확한 '관심사 분리(separation of concerns)' 원칙을 구현합니다.


일반적인 항공우주 로봇의 제어 아키텍처는 다음과 같은 계층으로 구성됩니다.

- **상위 레벨 (비실시간 / 소프트 실시간):** 이 계층은 주로 범용 운영체제(예: Linux)가 설치된 멀티코어 프로세서에서 실행됩니다. 여기에는 인지(perception), 임무 계획, 고급 행동 결정, 시각화(RVIZ) 등을 담당하는 ROS 2 노드들이 위치합니다. 이 노드들은 DDS를 통해 서로 통신하며, 복잡한 데이터 처리와 비결정적인 계산을 수행합니다.6
- **하위 레벨 (하드 실시간):** 이 계층은 실시간 운영체제(RTOS) 또는 실시간 패치(PREEMPT_RT)가 적용된 Linux 커널에서 실행됩니다. 여기에는 마이크로초 단위의 정밀한 타이밍이 요구되는 핵심적인 모션 제어 태스크와 EtherCAT 마스터 스택이 위치합니다.6
- **하드웨어 레벨:** EtherCAT 필드버스를 통해 물리적으로 연결된 액추에이터, 모터 드라이브, 센서, I/O 모듈 등이 이 계층을 구성합니다.7


이 아키텍처의 핵심은 상위 레벨과 하위 레벨을 연결하는 "브리지(bridge)" 노드입니다. 이 특화된 프로세스는 두 세계 사이에 존재하며 다음과 같은 역할을 수행합니다.

1. ROS 2의 모션 플래너(예: MoveIt)로부터 `/joint_trajectory_command`와 같은 DDS 토픽을 구독합니다.
2. EtherCAT 마스터로서, 주기적으로 EtherCAT 프레임을 생성하여 슬레이브들과 통신합니다.
3. DDS를 통해 수신한 상위 레벨의 궤적 명령(예: 관절 각도, 속도)을 하위 레벨의 프로세스 데이터 객체(PDO)로 변환하여 EtherCAT 프레임에 기록하고, 이를 각 모터 드라이브에 전달합니다.
4. 반송되는 EtherCAT 프레임에서 각 액추에이터의 현재 상태(예: 실제 위치, 속도, 토크)를 읽어들여, 이를 다시 `/joint_states`와 같은 DDS 토픽으로 발행하여 시스템의 다른 모든 노드가 로봇의 현재 상태를 알 수 있도록 합니다.6


가장 큰 기술적 과제 중 하나는 비실시간 DDS 세계와 하드 실시간 EtherCAT 세계 간의 결정론적 데이터 전송을 보장하는 것입니다. DDS를 통한 표준 ROS 2 메시징은 견고하지만, 네트워크 스택과 스케줄링으로 인해 비결정적인 지연 시간을 가질 수 있습니다.

- **공유 메모리 (Shared Memory):** 이 문제를 해결하는 가장 효과적이고 일반적인 방법은 공유 메모리 메커니즘을 사용하는 것입니다. 브리지 노드의 DDS 수신부는 명령을 공유 메모리 버퍼에 쓰고, 실시간 EtherCAT 마스터 태스크는 이 버퍼에서 직접 데이터를 읽습니다. 이 방식은 가장 중요한 데이터 경로에서 네트워크 스택의 오버헤드와 지터를 피할 수 있게 해줍니다.6 데이터 일관성을 보장하기 위해 핸드셰이크(handshake) 메커니즘이 함께 사용됩니다.6
- **실시간 커널 (PREEMPT_RT):** Linux 기반 시스템에서 EtherCAT 마스터의 하드 실시간 성능을 보장하기 위해서는 표준 커널로는 부족합니다. PREEMPT_RT 패치는 Linux 커널을 완전히 선점 가능하게 만들어, 커널 내부에서 발생하는 지연 시간을 극적으로 줄여줍니다. 이를 통해 EtherCAT 마스터 프로세스는 운영체제의 방해를 최소화하면서 마이크로초 수준의 결정론적 타이밍으로 실행될 수 있습니다.39


이러한 계층적 아키텍처는 의도적으로 시스템을 서로 다른 책임과 실시간 요구사항을 가진 부분들로 분할합니다. 3D 포인트 클라우드를 처리하는 인지 알고리즘은 마이크로초 단위의 결정론이 필요하지 않습니다. 대신 높은 처리량이 필요하며 약간의 지연은 허용됩니다. DDS를 통해 통신하는 표준 Linux 프로세스는 이러한 작업에 완벽합니다. 반대로, 서보 모터의 토크 명령을 업데이트하는 제어 루프는 마이크로초 단위의 결정론이 반드시 필요합니다. EtherCAT을 통해 통신하는 실시간 태스크는 이 작업에 완벽합니다.

만약 인지 알고리즘을 하드 실시간 루프에서 실행하려 한다면 비효율적일 뿐만 아니라 제어 태스크의 타이밍을 위협할 수 있습니다. 반대로 모터 제어를 표준 DDS를 통해 실행하려 한다면 지터로 인해 실패할 것입니다.

공유 메모리 브리지를 갖춘 계층적 아키텍처는 시스템의 각 부분이 자신에게 최적화된 환경에서 작동하도록 합니다. 이는 "무엇을 할 것인가"(계획, 소프트 실시간 또는 비실시간 태스크)와 "어떻게 정밀하게 할 것인가"(제어, 하드 실시간 태스크)를 명확하게 분리합니다. 이 우아한 분리 원칙이야말로 성숙하고 견고한 시스템 설계의 특징입니다. 사용자가 처음 제기했던 '모순'은 사실 이 정교한 설계의 한 특징인 것입니다.


고신뢰성 항공우주 시스템은 단순한 장애 복구를 넘어, 기능 안전(functional safety)과 사이버 보안(cybersecurity)이라는 두 가지 중요한 차원을 반드시 고려해야 합니다. DDS와 EtherCAT 아키텍처는 이 두 영역에서도 각기 다른 계층에서 상호 보완적인 역할을 수행합니다.


기능 안전은 시스템의 오작동으로 인해 발생할 수 있는 인간이나 환경에 대한 위험을 허용 가능한 수준으로 낮추는 것을 목표로 합니다.

- **"블랙 채널(Black Channel)" 원리:** FSoE(Safety over EtherCAT)는 안전 관련 데이터(예: 비상 정지, 안전 도어, 라이트 커튼 신호)를 표준, 비안전 인증 EtherCAT 네트워크를 통해 전송할 수 있게 해주는 프로토콜입니다.44 여기서 통신 매체는 신뢰할 수 없다고 가정되는 "블랙 채널"로 취급됩니다. 데이터의 안전성은 통신 매체가 아닌, FSoE 프로토콜 자체에 의해 종단 간(end-to-end)으로 보장됩니다.44
- **FSoE 안전 컨테이너(Safety Container):** 안전 데이터는 표준 EtherCAT 프로세스 데이터 내의 "안전 컨테이너"에 캡슐화됩니다. 이 컨테이너에는 실제 안전 데이터 외에도, 각 통신을 고유하게 식별하는 연결 ID(Connection ID), 통신 지연을 감지하는 워치독 타이머, 그리고 데이터 무결성을 검증하는 강력한 순환 중복 검사(CRC) 체크섬이 포함됩니다.44 FSoE 마스터와 슬레이브는 매 사이클마다 이 필드들을 검증하여, 주변의 표준 데이터와는 독립적으로 안전 통신의 무결성을 보장합니다.
- **아키텍처에 대한 함의:** 이 접근 방식은 단일 네트워크 케이블로 표준 제어 데이터와 SIL 3(Safety Integrity Level 3) 수준의 안전 신호를 동시에 전송할 수 있게 해줍니다. 이는 별도의 하드와이어링된 안전 시스템에 비해 배선 복잡성, 비용, 그리고 잠재적인 고장 지점을 극적으로 줄여줍니다.46


사이버 보안은 악의적인 공격으로부터 시스템을 보호하는 데 중점을 둡니다. DDS와 EtherCAT은 근본적으로 다른 보안 프로필을 가지며, 이는 다층적 방어(defense-in-depth) 전략을 요구합니다.

- **DDS 보안 (DDS-SECURITY™):** DDS는 종종 표준 IP 네트워크 위에서 실행되므로, 다양한 사이버 위협에 노출될 수 있습니다. 이를 해결하기 위해 OMG 표준인 DDS-Security는 데이터 자체에 초점을 맞춘 포괄적이고 세분화된 보안 모델을 제공합니다.11 이는 단순한 암호화 터널(SSL/TLS)이 아닙니다.
  - **인증 (Authentication):** 참여하는 애플리케이션의 신원을 확인합니다.48
  - **접근 제어 (Access Control):** 어떤 애플리케이션이 어떤 토픽을 발행하거나 구독할 수 있는지에 대한 정책을 강제합니다.48
  - **암호화 (Encryption):** 토픽 단위로 데이터의 기밀성을 보장합니다.48
- **EtherCAT 사이버 보안:** EtherCAT은 근본적으로 다른 보안 접근법을 가집니다.
  - **내재된 면역성:** 핵심 프로토콜이 IP 기반이 아니기 때문에, TCP/IP 스택을 표적으로 하는 대부분의 일반적인 네트워크 공격(예: DoS, 포트 스캐닝)에 대해 본질적으로 면역성을 가집니다.51 비-EtherCAT 프레임은 슬레이브 하드웨어에서 필터링됩니다.
  - **취약점:** EtherCAT의 취약점은 OSI 모델의 2계층(데이터 링크 계층)에 존재합니다. 표준 EtherCAT 프로토콜에는 내장된 인증이나 암호화가 없기 때문에, 네트워크에 물리적으로 접근한 공격자는 패킷 주입(packet injection)이나 중간자 공격(man-in-the-middle)을 수행할 수 있습니다.52
  - **완화 전략:** 따라서 EtherCAT의 보안은 다층적 방어 전략에 의존합니다.54
    1. **물리적 보안:** 기계와 네트워크 케이블에 대한 물리적 접근을 통제합니다.
    2. **네트워크 분할:** 퍼듀 모델(Purdue Model)에 따라, 운영 기술(OT) 네트워크인 EtherCAT망을 정보 기술(IT) 네트워크로부터 방화벽으로 격리합니다.55
    3. **강화된 마스터:** EtherCAT 마스터 역할을 하는 IPC는 EtherCAT 세그먼트와 더 넓은 IT/DDS 네트워크를 연결하는 유일한 게이트웨이 역할을 합니다. 따라서 이 마스터를 업계 표준에 따라 철저히 강화(hardening)하는 것이 무엇보다 중요합니다.51

결론적으로, 이 시스템은 DDS-Security를 통해 외부와 연결되는 상위 레벨 데이터 흐름을 보호하고, 네트워크 분할과 강화된 게이트웨이를 통해 하위 레벨의 결정론적 제어 네트워크를 보호하는 이중의 보안 구조를 형성합니다.


항공우주 분야에서 로봇 시스템을 상용화하기 위해서는 감항인증(airworthiness certification)이라는 매우 엄격한 과정을 통과해야 합니다. 이 과정에서 소프트웨어의 신뢰성을 입증하는 것은 핵심적인 과제입니다.


DO-178C/ED-12C는 미국 연방항공청(FAA), 유럽항공안전청(EASA)과 같은 인증 기관이 상업용 항공기에 탑재되는 모든 소프트웨어 기반 시스템을 승인할 때 참조하는 기본 표준입니다.58 이 표준은 소프트웨어의 전체 생명주기(계획, 개발, 검증 등)에 걸쳐 따라야 할 엄격한 프로세스를 정의합니다.


DO-178C는 소프트웨어의 고장이 항공기, 승무원, 승객에게 미치는 영향에 따라 5개의 설계 보증 수준(Design Assurance Level, DAL)을 정의합니다.59

- **DAL A:** 가장 엄격한 수준으로, 소프트웨어의 실패가 항공기의 치명적인 고장(catastrophic failure)을 유발할 수 있는 경우에 적용됩니다 (예: 비행 제어 시스템).
- **DAL B ~ E:** 위험도가 낮아질수록 요구되는 목표와 검증 활동의 엄격함이 완화됩니다.61


DAL A 수준에 맞춰 소프트웨어를 개발하고 검증하는 것은 엄청난 비용과 시간을 소요하는 작업입니다. 코드 한 줄당 약 100달러의 비용이 들 수 있다는 추정도 있습니다.62 따라서 항공우주 시스템 아키텍트에게는 이미 인증 증거(certification evidence)를 갖춘 상용 기성품(Commercial Off-The-Shelf, COTS) 구성 요소를 활용하는 것이 매우 중요한 전략이 됩니다.

- **RTI Connext DDS Cert:** 이 제품은 데이터 분산 서비스(DDS) 표준을 기반으로 하는 연결 플랫폼 중 최초로 DO-178C DAL A 인증 증거 패키지를 제공하는 COTS 솔루션입니다.15 이 패키지에는 인증 기관이 요구하는 모든 생명주기 데이터(계획서, 요구사항, 설계 문서, 테스트 결과, 분석 보고서 등)가 포함되어 있습니다.
- **아키텍처에 대한 함의:** 항공우주 회사는 이 COTS 인증 패키지를 구매함으로써, 복잡한 프로세스 간 통신을 담당하는 미들웨어 계층에 대한 인증 작업을 직접 수행할 필요 없이, 이미 검증된 구성 요소를 사용할 수 있습니다. 이는 프로젝트의 리스크, 비용, 그리고 시장 출시까지의 시간을 획기적으로 단축시켜 줍니다.63

결론적으로, COTS 인증이 가능한 DDS 미들웨어의 존재는 항공우주 비행 시스템의 상위 레벨, 비결정론적 데이터 버스로 DDS를 선택하는 강력한 이유가 됩니다. 이는 DDS와 EtherCAT을 결합한 2계층 아키텍처가 항공우주 분야에서 기술적으로나 전략적으로 얼마나 합리적인 선택인지를 다시 한번 뒷받침합니다.


본 보고서는 항공우주 로봇 시스템에서 DDS와 EtherCAT을 동시에 사용하는 것이 모순이 아니냐는 심도 있는 질문에서 시작했습니다. 분석 결과, 이 아키텍처는 모순적이기는커녕, 각기 다른 계층에서 특정 문제를 해결하기 위해 최적의 기술을 조합한 고도로 정교하고 공생적인(symbiotic) 설계임이 명확해졌습니다.


이 시스템은 신뢰성이라는 목표를 달성하기 위해 '올바른 계층에서 올바른 기술로 올바른 문제를 해결'하는 원칙을 따릅니다.

- **소프트웨어 복잡성 관리:** 상위 레벨에서는 DDS의 분산형, 데이터 중심 아키텍처를 통해 소프트웨어의 복잡성과 통합 문제를 해결하고, `roscore`와 같은 중앙 집중형 소프트웨어 단일 장애점을 제거합니다.
- **하드웨어 성능 보장:** 하위 레벨에서는 EtherCAT의 중앙집중형, 결정론적 제어 방식을 통해 다축 모션 제어에 필요한 타협 불가능한 실시간 성능과 정밀 동기화를 보장합니다.
- **계산된 위험과 체계적 완화:** EtherCAT의 마스터-슬레이브 구조가 만드는 하드웨어 단일 장애점은 성능을 위한 의도된 트레이드오프입니다. 이 위험은 케이블 이중화, 마스터 핫 스탠바이, 그리고 완전한 컨트롤러 이중화에 이르는 다층적이고 성숙한 메커니즘을 통해 체계적으로 관리되고 완화됩니다.


결론적으로, 시스템 신뢰성은 단일 기술로 얻어지는 것이 아니라, 각 계층의 잠재적인 장애 모드를 분석하고 그에 맞는 최적의 솔루션을 적용하여 설계되는 공학적 속성입니다. DDS는 소프트웨어 SPOF를 설계적으로 제거하고, EtherCAT은 성능을 확보한 뒤 이중화로 SPOF를 완화합니다. 나아가 기능 안전(FSoE)과 다층적 사이버 보안(DDS-Security + 네트워크 분할)이 전체 시스템을 보호하며, DO-178C와 같은 엄격한 인증 프로세스가 최종적인 신뢰성을 보증합니다.


기술은 끊임없이 발전하고 있으며, 항공우주 로봇 통신의 미래는 현재의 아키텍처 원칙을 기반으로 더욱 진화할 것입니다.

- **시간 민감형 네트워킹 (TSN):** TSN은 표준 이더넷을 통해 결정론적 통신을 제공하기 위한 IEEE 표준 집합입니다. DDS와 TSN의 결합은 모든 종류의 트래픽(실시간 제어, 대용량 데이터, 일반 통신)을 단일 융합 네트워크로 통합할 수 있는 강력한 미래 솔루션으로 주목받고 있습니다.16 일부에서는 TSN이 장기적으로 EtherCAT과 같은 특화된 필드버스를 대체할 수 있다고 주장하기도 합니다.66
- **EtherCAT G/G10:** EtherCAT 역시 1Gbit/s 및 10Gbit/s로 성능을 확장하는 EtherCAT G와 G10으로 진화하고 있습니다.10 이는 가장 까다로운 애플리케이션에서 성능 우위를 유지하려는 EtherCAT의 노력을 보여줍니다.

미래의 항공우주 로봇 통신은 이러한 강력한 기술들 간의 역동적인 상호작용과 잠재적인 융합을 통해 형성될 것입니다. 그러나 현재의 DDS/EtherCAT 아키텍처가 보여주는 계층적이고 목적에 맞는 신뢰성 설계의 핵심 원칙은 앞으로도 변함없이 중요한 기초가 될 것입니다.


1. DDS(Data Distribution Service) 이해하기 - Hoon's Blog - 티스토리, accessed July 13, 2025, https://yhoons.tistory.com/117
2. EtherCAT | Beckhoff USA, accessed July 13, 2025, https://www.beckhoff.com/en-us/products/i-o/ethercat/
3. EC-Product-Family-Overview_472719 | PDF | Computer Network - Scribd, accessed July 13, 2025, https://www.scribd.com/document/843681202/EC-Product-Family-Overview-472719
4. Redundancy solutions for increased plant availability | Beckhoff USA, accessed July 13, 2025, https://www.beckhoff.com/en-us/products/automation/twincat-3-redundancy/
5. EtherCAT draws ahead in the industrial ethernet competition - Motion Solutions, accessed July 13, 2025, https://www.motionsolutions.com/ethercat-draws-ahead-in-the-industrial-ethernet-competition/
6. (PDF) Develop Real-Time Robot Control Architecture Using Robot Operating System and EtherCAT - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/352732018_Develop_Real-Time_Robot_Control_Architecture_Using_Robot_Operating_System_and_EtherCAT
7. A Control System Design for 7-DoF Light-weight Robot based on EtherCAT Bus, accessed July 13, 2025, https://www.researchgate.net/publication/328185486_A_Control_System_Design_for_7-DoF_Light-weight_Robot_based_on_EtherCAT_Bus
8. Proposal for Implementation of Real-time Systems in ROS 2 - ROS2 Design, accessed July 13, 2025, https://design.ros2.org/articles/realtime_proposal.html
9. ROS 2와 결합된 소프트웨어 정의 EtherCAT | AMR 개발 가속화 | 에이디링크 ADLINK, accessed July 13, 2025, https://www.adlinktech.com/kr/software-ethercat-ros2-amr-development
10. EtherCAT - Wikipedia, accessed July 13, 2025, https://en.wikipedia.org/wiki/EtherCAT
11. How DDS Works? - DDS Foundation, accessed July 13, 2025, https://www.dds-foundation.org/why-choose-omg-dds-standard/
12. Getting on the Right Motion Bus - Tech Briefs, accessed July 13, 2025, https://www.techbriefs.com/component/content/article/9986-15124-321
13. The principle - Beckhoff Information System - English, accessed July 13, 2025, https://infosys.beckhoff.com/content/1033/ethercatsystem/2474143371.html
14. ROS 2 Architecture Overview - Automatic Addison, accessed July 13, 2025, https://automaticaddison.com/ros-2-architecture-overview/
15. Urban Air Mobility - RTI, accessed July 13, 2025, https://www.rti.com/hubfs/capability-briefs/rti-capability-brief-urban-air-mobility.pdf
16. DDS and TSN: Converging Two Communications Standards - RTI, accessed July 13, 2025, https://www.rti.com/dds-and-tsn
17. ROS 2 node graph of the UAV-ground system. The UAV's onboard nodes... | Download Scientific Diagram - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/figure/ROS-2-node-graph-of-the-UAV-ground-system-The-UAVs-onboard-nodes-publish-telemetry-and_fig2_392225374
18. RTI Connext DDS vs. MazikThings Comparison - SourceForge, accessed July 13, 2025, https://sourceforge.net/software/compare/Connext-DDS-vs-MazikThings/
19. DDS and TSN: The Future for Real-Time Data Exchange? - RTI, accessed July 13, 2025, https://www.rti.com/blog/dds-and-tsn-the-future-for-real-time-data-exchange
20. the Ethernet Fieldbus - EtherCAT Technology Group, accessed July 13, 2025, https://www.ethercat.org/en/technology.html
21. SuperCAT | EtherCAT 솔루션 | 에이디링크 ADLINK, accessed July 13, 2025, https://www.adlinktech.com/Products/Motion_Control/EtherCATSolution/SuperCAT?lang=ko
22. Which industrial network do you think is better? : r/PLC - Reddit, accessed July 13, 2025, https://www.reddit.com/r/PLC/comments/1hbvfy5/which_industrial_network_do_you_think_is_better/
23. EtherCAT vs EthernetIP for the dumb : r/PLC - Reddit, accessed July 13, 2025, https://www.reddit.com/r/PLC/comments/thmmsk/ethercat_vs_ethernetip_for_the_dumb/
24. EtherCAT : Architecture, Working, Differences & Its Applications - ElProCus, accessed July 13, 2025, https://www.elprocus.com/ethercat/
25. EtherCAT in space robotics, accessed July 13, 2025, https://www.ethercat.org/download/press/etg_052022-1_ko.pdf
26. Airbus high-lift system with high-speed EtherCAT control, accessed July 13, 2025, https://www.ethercat.org/download/documents/pcc0306_airbus_e.pdf
27. Redundancy and hot swap technology in industrial Ethernet EtherCAT - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/296322899_Redundancy_and_hot_swap_technology_in_industrial_Ethernet_EtherCAT
28. Redundancy Mode - Beckhoff Information System - English, accessed July 13, 2025, [https://infosys.beckhoff.com/english.php?content=../content/1033/tc3_io_intro/1446583307.html&id=](https://infosys.beckhoff.com/english.php?content=../content/1033/tc3_io_intro/1446583307.html&id)
29. Configuring EtherCAT cable redundancy. - Beckhoff Information System - English, accessed July 13, 2025, https://infosys.beckhoff.com/content/1033/cx51x0_hw/3674859659.html
30. Master Run-time: White Paper - koenig-pa, accessed July 13, 2025, https://koenig-pa.de/pdf/KPA_EtherCAT_Master_WhitePaper_20230328.pdf
31. "We make redundancy easy to use" | Beckhoff Worldwide, accessed July 13, 2025, https://www.beckhoff.com/en-en/company/news/we-make-redundancy-easy-to-use.html
32. EtherCAT 케이블 이중화 with TwinCAT 3(EtherCAT Cable Redundancy with TwinCAT 3) - YouTube, accessed July 13, 2025, https://www.youtube.com/watch?v=3u1WKlVhqxM
33. TF6220 | TwinCAT 3 EtherCAT Redundancy 250 | Beckhoff USA, accessed July 13, 2025, https://www.beckhoff.com/en-us/products/automation/twincat/tfxxxx-twincat-3-functions/tf6xxx-connectivity/tf6220.html
34. TS622x | TwinCAT EtherCAT Redundancy | Beckhoff USA, accessed July 13, 2025, https://www.beckhoff.com/en-us/products/automation/twincat/twincat-2/ts622x.html
35. EtherCAT cable redundancy - KINGSTAR, accessed July 13, 2025, https://help.kingstar.com/product_help/KS/Content/Concepts/Redundancy.htm
36. Journal of Beijing University of Aeronautics and Astronautics (White Paper): Redundancy and hot swap technology in Industrial Ethernet EtherCAT, accessed July 13, 2025, https://www.ethercat.org/en/downloads/downloads_8B0F4C63E67E46CA9A943BC3A5BA98F3.htm
37. Software-based solution protects plant uptime through redundant control operation | Beckhoff USA, accessed July 13, 2025, https://www.beckhoff.com/en-us/company/news/software-based-solution-protects-plant-uptime-through-redundant-control-operation.html
38. Develop Real-Time Robot Control Architecture Using Robot Operating System and EtherCAT - MDPI, accessed July 13, 2025, https://www.mdpi.com/2076-0825/10/7/141
39. (PDF) ROS2 Real-time Performance Optimization and Evaluation - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/376215526_ROS2_Real-time_Performance_Optimization_and_Evaluation
40. Real-Time EtherCAT-Based Control Architecture for Electro-Hydraulic Humanoid - MDPI, accessed July 13, 2025, https://www.mdpi.com/2227-7390/12/9/1405
41. Develop Real-Time Robot Control Architecture with ROS and EtherCAT - ADLINK Technology, accessed July 13, 2025, https://ok.adlinktech.com/social-doc/Twitter/General-ADLINK-Company-Awareness/ADLINK-Technology//ui/xWFXar/
42. The Real-Time Linux Kernel: A Survey on PREEMPT_RT, accessed July 13, 2025, https://re.public.polimi.it/retrieve/e0c31c12-9844-4599-e053-1705fe0aef77/11311-1076057_Reghenzani.pdf
43. control real-time processes with ROS/ROS2 on multicore system - Robotics Stack Exchange, accessed July 13, 2025, https://robotics.stackexchange.com/questions/97240/control-real-time-processes-with-ros-ros2-on-multicore-system
44. Safety over EtherCAT (FSoE) Introduction - EtherCAT Technology ..., accessed July 13, 2025, https://www.ethercat.org/download/documents/Safety-over-EtherCAT_Introduction.pdf
45. Safety with FSoE (FailSafe over EtherCAT) - acontis, accessed July 13, 2025, https://www.acontis.com/en/safety-with-fsoe-failsafe-over-ethercat.html
46. What is Safety over EtherCAT, FSoE? - KEB America, accessed July 13, 2025, https://www.kebamerica.com/blog/what-is-failsafe-over-ethercat-fsoe/
47. Safety-over-EtherCAT (FSoE) : How to secure your EtherCAT platform - ISIT, accessed July 13, 2025, https://isit.fr/en/article/safety-over-ethercat-how-to-secure-your-ethercat-platform.php
48. CoreDX DDS Secure | Twin Oaks Computing, Inc, accessed July 13, 2025, https://www.twinoakscomputing.com/coredx/coredx_secure
49. DDS Security - @PROJECT_NAME@ @PROJECT_VERSION@ documentation - Eclipse Cyclone DDS, accessed July 13, 2025, https://cyclonedds.io/docs/cyclonedds/0.10.3/security.html
50. DDS Security - OpenDDS 3.28.1, accessed July 13, 2025, https://opendds.readthedocs.io/en/dds-3.28.1/devguide/dds_security.html
51. Cybersecurity for Industrial Ethernet - Beckhoff USA Blog, accessed July 13, 2025, https://www.blog.beckhoffus.com/post/industrial-ethernet-cybersecurity
52. Ethernet for Control Automation Technology (EtherCAT) - ThreatNG Security - External Attack Surface Management (EASM) - Digital Risk Protection, accessed July 13, 2025, https://www.threatngsecurity.com/glossary/ethercat
53. Intrusion Detection of the ICS Protocol EtherCAT - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/315645332_Intrusion_Detection_of_the_ICS_Protocol_EtherCAT
54. Multi-Layered Security Strategy To Protect Your Business - CMIT Solutions, accessed July 13, 2025, https://cmitsolutions.com/blog/multi-layered-security/
55. What Is the Purdue Model for ICS Security? | A Guide to PERA - Palo Alto Networks, accessed July 13, 2025, https://www.paloaltonetworks.com/cyberpedia/what-is-the-purdue-model-for-ics-security
56. EtherNet/IP Vulnerability Assessment and Mitigation - Trout Software, accessed July 13, 2025, https://www.trout.software/resources/tech-blog/ethernet-ip-vulnerability-assessment-and-mitigation
57. ICS Security: Beware of Network Stack Vulnerabilities - Booz Allen, accessed July 13, 2025, https://www.boozallen.com/c/insight/blog/ics-security-beware-of-network-stack-vulnerabilities.html
58. Your Complete DO-178C Guide to Aerospace Software Compliance - LDRA, accessed July 13, 2025, https://ldra.com/do-178/
59. DO-178C Guidance: Introduction to RTCA DO-178 certification | Rapita Systems, accessed July 13, 2025, https://www.rapitasystems.com/do178
60. Making DO with Safety Standards - Aerospace Innovations, accessed July 13, 2025, https://aerospace-innovations.com/making-do-with-safety-standards/
61. RTI Connext Cert: Delivering RTCA DO-178C Certification Evidence for Avionics Programs, accessed July 13, 2025, https://www.rti.com/blog/rtca-do178c-certification
62. DO-178C: Software for NextGen Avionics, UAVs and More - Aviation Today, accessed July 13, 2025, https://interactive.aviationtoday.com/do-178c-software-for-nextgen-avionics-uavs-and-more/
63. RTI Releases First DO-178C Certification Data Package for Its Safety-Critical Connectivity Platform, accessed July 13, 2025, https://www.rti.com/news/do-178c-cert-data-package
64. accessed January 1, 1970, [https.www.rti.com/news/do-178c-cert-data-package](http://docs.google.com/https.www.rti.com/news/do-178c-cert-data-package)
65. A Comprehensive Survey of Wireless Time-Sensitive Networking (TSN): Architecture, Technologies, Applications, and Open Issues - arXiv, accessed July 13, 2025, https://arxiv.org/html/2312.01204v1
66. [1804.07643] Time-Sensitive Networking for robotics - ar5iv - arXiv, accessed July 13, 2025, https://ar5iv.labs.arxiv.org/html/1804.07643
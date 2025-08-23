# ROS 2 (Robot Operating System 2)



로봇 운영체제(Robot Operating System, ROS)는 스탠포드 대학의 초기 연구에서 시작되어 Willow Garage를 거치며 로봇 소프트웨어 개발의 사실상 표준으로 자리 잡았다.1 ROS1은 방대한 라이브러리, 시각화 및 시뮬레이션 도구, 활발한 커뮤니티를 바탕으로 로봇 연구 및 프로토타이핑을 가속화하는 데 결정적인 역할을 했다.2 그러나 ROS1은 본질적으로 연구 환경을 위해 설계되었기에, 산업 현장과 상용 제품에 적용되면서 여러 구조적 한계에 직면했다.3

ROS1의 핵심적인 한계는 중앙 집중형 마스터 노드(`roscore`)에 의존하는 아키텍처에서 비롯되었다.2 이 `roscore`는 모든 노드의 등록과 통신 연결을 관리하는 중추였지만, 동시에 시스템 전체의 단일 실패 지점(Single Point of Failure)으로 작용했다. 마스터가 중단되면 전체 ROS 네트워크가 마비되는 구조는 상업용 제품에 요구되는 안정성과 강건성을 충족시키기 어려웠다.2 또한, ROS1은 실시간(Real-time) 제어를 염두에 두고 설계되지 않았으며, 이로 인해 결정론적(deterministic)인 작업 수행이 필수적인 자율주행이나 정밀 산업 공정에는 부적합했다.2 보안 기능의 부재, 멀티 로봇 시스템 지원의 미비, 그리고 Windows나 macOS와 같은 다양한 운영체제에 대한 제한적인 지원 역시 상용화의 큰 걸림돌이었다.3

이러한 한계를 극복하기 위해 ROS2는 단순한 업그레이드가 아닌, 근본부터 다시 설계된 새로운 프레임워크로 탄생했다. ROS2의 설계 철학은 '연구 프로토타입'에서 '산업용 제품'으로의 전환을 명확히 보여준다.9 가장 큰 변화는 중앙 마스터를 제거하고, 산업 표준 통신 미들웨어인 DDS(Data Distribution Service)를 기본 통신 계층으로 채택하여 완전한 분산형 시스템을 구현한 것이다.4 이 구조적 변화는 단일 실패 지점을 제거했을 뿐만 아니라, 시스템의 확장성과 안정성을 극적으로 향상시켰다. 이를 통해 ROS2는 본질적으로 다개체 로봇 시스템과 스웜 로보틱스(Swarm Robotics)를 지원할 수 있는 토대를 마련했다.5

더 나아가 ROS2는 설계 초기부터 실시간성, 보안, 플랫폼 간 호환성을 핵심 목표로 삼았다. 실시간 운영체제(RTOS)와의 연동을 고려하고, 결정론적 실행을 보장하기 위한 메모리 관리 및 스케줄링 기법을 도입했다.6 DDS가 제공하는 보안 표준을 활용하여 통신 암호화, 인증, 접근 제어를 포괄하는 강력한 보안 프레임워크(SROS2)를 구축했다.3 또한, Linux뿐만 아니라 Windows, macOS를 1등급(Tier 1) 플랫폼으로 공식 지원함으로써 개발 환경의 유연성을 크게 높였다.8 이처럼 ROS2는 ROS1의 성공적인 개념들은 계승하되, 산업 현장의 요구사항을 충족시키기 위해 아키텍처의 근간을 재정립한 차세대 로봇 개발 플랫폼이다.


ROS1의 배포 주기가 비교적 비정기적이었던 것과 달리, ROS2는 산업계 사용자들이 장기적인 제품 개발 및 유지보수 계획을 수립할 수 있도록 예측 가능하고 안정적인 배포 정책을 채택했다.14 이 정책은 ROS Enhancement Proposal(REP) 2000에 상세히 정의되어 있으며, 로봇 소프트웨어 아키텍처의 중요한 의사결정 기준을 제공한다.15

ROS2의 배포판은 매년 5월 23일, '세계 거북이의 날(World Turtle Day)'에 맞춰 새로운 버전이 출시된다.16 이 배포 모델의 핵심은 장기 지원(Long-Term Support, LTS) 버전과 비-LTS 버전이 2년 주기로 번갈아 출시된다는 점이다.16

- **LTS (Long-Term Support) 배포판**: 짝수 해에 출시되며, 총 5년의 지원 기간을 제공한다. 이는 산업용 하드웨어나 소프트웨어의 일반적인 지원 주기와 일치하며, 상용 제품에 안정적인 기반을 제공하려는 ROS2의 전략적 목표를 반영한다. LTS 버전은 일반적으로 2년마다 출시되는 Ubuntu LTS 버전에 맞춰 개발되며, 해당 Ubuntu 버전의 표준 지원 종료 시점까지 지원이 보장된다.16
- **비-LTS 배포판**: 홀수 해에 출시되며, 1.5년의 비교적 짧은 지원 기간을 가진다. 이 버전의 주된 목적은 다음 LTS 버전까지 새로운 기능과 개선 사항을 빠르게 도입하고 커뮤니티의 피드백을 받는 것이다. 비-LTS 버전은 이전 해에 출시된 LTS 버전과 동일한 Ubuntu LTS를 대상으로 하므로, 개발자들은 OS 변경 없이 새로운 기능을 시험해볼 수 있다. 또한, 지원 기간이 다음 LTS 버전 출시 후 6개월까지 겹치도록 설계되어 충분한 마이그레이션 시간을 제공한다.16

이러한 예측 가능한 배포 주기는 기업들이 기술 로드맵을 수립하고, 개발 사이클을 ROS2 지원 기간에 맞춰 조정하며, 기술 부채와 마이그레이션 계획을 사전에 관리할 수 있게 해주는 핵심적인 비기능적 요소이다.

이와 별도로, `Rolling Ridley`라는 특별한 배포판이 존재한다.18 이는 특정 버전에 고정되지 않고 최신 개발 내용이 지속적으로 통합되는 '롤링 릴리스' 방식의 배포판이다. 

`Rolling`은 차기 안정 버전의 스테이징 영역 역할을 하며, 최신 기능을 가장 먼저 사용해보고자 하는 개발자나 차기 배포판에 포함될 패키지를 미리 테스트하려는 패키지 관리자들을 위해 제공된다. 하지만 안정성이 보장되지 않으므로, 대부분의 사용자, 특히 상용 제품 개발에는 안정된 최신 LTS 버전을 사용하는 것이 권장된다.18

다음 표는 현재까지 공식적으로 배포된 ROS2 버전들의 수명 주기를 REP 2000 정책에 기반하여 정리한 것이다. 이는 로봇 시스템 아키텍트와 시니어 엔지니어가 프로젝트에 적합한 버전을 선택하고 장기적인 유지보수 전략을 수립하는 데 필수적인 정보를 제공한다.

| 배포판 이름 (코드명)    | 출시일                 | 지원 종료(EOL)일   | 유형          | 대상 Ubuntu LTS |
| ----------------------- | ---------------------- | ------------------ | ------------- | --------------- |
| **Ardent Apalone**      | 2017년 12월 8일        | 2018년 12월        | 비-LTS (초기) | 16.04           |
| **Bouncy Bolson**       | 2018년 7월 2일         | 2019년 7월         | 비-LTS (초기) | 18.04           |
| **Crystal Clemmys**     | 2018년 12월 14일       | 2019년 12월        | 비-LTS (초기) | 18.04           |
| **Dashing Diademata**   | 2019년 5월 31일        | 2021년 5월         | LTS (초기)    | 18.04           |
| **Eloquent Elusor**     | 2019년 11월 22일       | 2020년 11월        | 비-LTS (초기) | 18.04           |
| **Foxy Fitzroy**        | 2020년 6월 5일         | 2023년 5월         | **LTS**       | 20.04           |
| **Galactic Geochelone** | 2021년 5월 23일        | 2022년 12월 9일    | 비-LTS        | 20.04           |
| **Humble Hawksbill**    | 2022년 5월 23일        | 2027년 5월         | **LTS**       | 22.04           |
| **Iron Irwini**         | 2023년 5월 23일        | 2024년 11월        | 비-LTS        | 22.04           |
| **Jazzy Jalisco**       | 2024년 5월 23일        | 2029년 5월         | **LTS**       | 24.04           |
| **Kilted Kaiju**        | 2025년 5월 23일 (예정) | 2026년 12월 (예정) | 비-LTS        | 24.04           |
| **Rolling Ridley**      | 2020년 6월             | 지속               | 개발          | 최신            |

표 1: ROS2 공식 배포판 수명 주기 15


ROS2의 발전은 단순히 기능 추가의 연속이 아니라, 핵심 아키텍처를 지속적으로 개선하고 강화하는 과정이었다. 통신, 실시간성, 보안, 개발 도구라는 네 가지 핵심 축을 중심으로 ROS2가 어떻게 더 강력하고 유연하며, 생산에 적합한 프레임워크로 진화했는지 심층적으로 분석한다.



ROS2 아키텍처의 가장 혁신적인 설계 중 하나는 ROS 미들웨어 인터페이스(ROS Middleware Interface, RMW)의 도입이다.22 이는 ROS 클라이언트 라이브러리(RCL), 즉 `rclcpp`(C++)나 `rclpy`(Python)와 같은 사용자 API 계층을 실제 데이터 전송을 담당하는 하부 통신 미들웨어로부터 분리하는 추상화 계층이다.13

ROS1에서는 TCPROS/UDPROS라는 자체 개발한 통신 프로토콜을 사용했지만, 이는 기능 확장에 한계가 있었다.22 반면, ROS2는 RMW를 통해 특정 통신 기술에 대한 종속성을 제거했다. 이 덕분에 개발자는 애플리케이션 코드를 변경하지 않고도 다양한 통신 미들웨어를 선택하거나 교체할 수 있다.24 미들웨어는 런타임에 `RMW_IMPLEMENTATION` 환경 변수를 설정하는 것만으로 간단하게 변경할 수 있다.25

이러한 설계는 ROS2에 두 가지 중요한 전략적 이점을 제공했다. 첫째, DDS와 같이 이미 산업계에서 검증된 고성능 미들웨어의 강력한 기능들, 예를 들어 다양한 QoS(Quality of Service) 정책, 실시간 통신, 내장 보안 기능 등을 ROS2가 직접 재개발할 필요 없이 그대로 활용할 수 있게 되었다.12 둘째, 기술의 발전에 따라 미래에 등장할 새로운 통신 패러다임에도 유연하게 대응할 수 있는 길을 열어두었다. 이는 최근 Zenoh가 새로운 RMW 구현체로 공식 지원되기 시작한 사례에서 명확히 증명된다.26 RMW는 ROS2를 미래 지향적이고 확장 가능한 플랫폼으로 만드는 핵심적인 아키텍처적 결정이었다.


ROS2 초기부터 현재까지 통신의 주류는 DDS(Data Distribution Service) 표준을 기반으로 한 RMW 구현체들이었다. 그중에서도 1등급(Tier 1)으로 공식 지원되는 오픈소스 구현체는 eProsima의 Fast DDS와 Eclipse의 Cyclone DDS이다.28 ROS 2 기술 운영 위원회(Technical Steering Committee, TSC)는 새로운 LTS 버전이 출시될 때마다 이들 중 하나를 기본 RMW로 선정하기 위해 심층적인 기술 평가를 수행한다.29

Humble Hawksbill 출시를 앞두고 진행된 2021년 평가 보고서는 두 미들웨어의 특성을 잘 보여준다.28

- **성능**: 빌드팜에서 수행된 벤치마크 결과, 동기식(synchronous) 퍼블리싱 모드에서는 두 구현체의 CPU 및 메모리 사용량이 거의 비슷했다. 하지만 대용량 메시지 전송 시에는 차이가 드러났다. Fast DDS는 4MB 크기의 메시지까지 손실 없이 전송한 반면, Cyclone DDS는 2MB 크기부터 메시지 손실이 발생하기 시작했다. 지연 시간(latency) 측면에서도 Fast DDS가 대용량 메시지에서 더 우수한 성능을 보였다.28
- **코드 품질 및 커뮤니티**: REP-2004 코드 품질 기준 준수 여부에서는 두 구현체가 유사한 수준을 보였으나, Fast DDS가 품질 레벨 1을, Cyclone DDS가 레벨 2를 선언하여 Fast DDS가 더 엄격한 기준을 충족하고 있음을 나타냈다.28 사용자 설문조사에서는 Cyclone DDS에 대한 선호도가 약간 더 높게 나타났는데, 이는 개발자 커뮤니티의 접근성과 응답성이 좋다는 평가에 기인했다. 반면, Fast DDS는 더 나은 코드 품질과 철저한 리뷰 프로세스, 풍부한 문서가 장점으로 꼽혔다.28

최종적으로 TSC는 Humble의 기본 RMW로 Fast DDS를 선정했다.32 이는 Foxy에 이어 연속으로 기본 채택된 것으로, 특히 대용량 데이터 처리 성능과 안정성, 그리고 엄격한 품질 관리 프로세스가 주요한 고려 사항이었음을 시사한다.28

실제 적용 환경에서는 두 미들웨어 모두 장단점을 가지며, 특히 Wi-Fi와 같은 손실이 많은 네트워크 환경에서는 추가적인 튜닝이 필요하다. 일반적으로 UDP 패킷 단편화(fragmentation)로 인해 발생하는 문제를 해결하기 위해 커널 파라미터(`net.ipv4.ipfrag_time`, `net.ipv4.ipfrag_high_thresh`)를 조정하거나, 각 DDS 벤더가 제공하는 XML 설정을 통해 버퍼 크기(`net.core.rmem_max`)를 늘리는 등의 최적화가 권장된다.35


DDS는 유선 LAN과 같이 안정적이고 손실이 적은 네트워크 환경을 가정하고 설계되었다.38 이로 인해 Wi-Fi, 5G 등 무선 환경이나 지리적으로 분산된 대규모 시스템에서는 DDS의 브로드캐스트/멀티캐스트 기반의 노드 탐색(discovery) 프로토콜이 과도한 네트워크 트래픽을 유발하는 문제가 있었다.23

이러한 DDS의 한계를 극복하기 위한 차세대 미들웨어로 Zenoh가 주목받고 있다. Zenoh는 극도로 효율적인 탐색 프로토콜을 사용하여 DDS 대비 탐색 트래픽을 90% 이상 절감할 수 있다.38 또한, 유선/무선, 로컬/광역 네트워크(WAN) 등 네트워크 토폴로지에 제약 없이 동작하도록 설계되었다. Zenoh는 Peer-to-Peer, Brokered(라우터 중계), Mesh 등 다양한 통신 모드를 지원하여, 로봇 플래투닝(platooning)과 같이 통신 환경이 동적으로 변하는 시나리오에서도 강점을 보인다.38

Zenoh의 또 다른 핵심 장점은 경량성이다. 수백 바이트의 메모리만으로 마이크로컨트롤러(MCU)에서도 동작할 수 있어, 로봇 내부의 임베디드 시스템부터 데이터센터의 클라우드까지 단일 프로토콜로 통합하는 것을 가능하게 한다.38

이러한 전략적 가치를 인정받아, ROS2 Kilted Kaiju (2025년 5월 출시 예정)부터 `rmw_zenoh_cpp`가 1등급 RMW로 공식 승격되었다.27 이는 ROS2가 단일 로봇의 로컬 네트워크를 넘어, 다개체 로봇군, IoT 연동, 클라우드 로보틱스와 같은 새로운 영역으로 확장하려는 의지를 보여주는 중요한 이정표이다. 개발자는 이제 애플리케이션의 특성에 따라 고성능 로컬 통신이 필요할 때는 DDS를, 확장성과 유연성이 중요한 분산 시스템에서는 Zenoh를 선택할 수 있게 되었다. Zenoh는 DDS를 대체하는 것이 아니라, ROS2의 적용 범위를 새로운 차원으로 확장하는 전략적 보완재인 셈이다.



ROS2는 실시간 성능을 보장하기 위해 설계되었지만, 프레임워크 자체만으로는 완전한 실시간성을 달성할 수 없다. 결정론적인 실행은 운영체제(OS) 수준의 지원이 필수적이다.46 이를 위해 ROS2 개발 환경에서는 주로 리눅스의 실시간 패치인 `RT_PREEMPT` 커널을 사용한다.46

`RT_PREEMPT` 커널은 커널 코드의 대부분을 선점 가능하게 만들어, 사용자 수준의 고우선순위 스레드가 커널 작업에 의해 지연되는 것을 최소화한다.

또 다른 중요한 비결정적 요소는 메모리 페이지 폴트(page fault)이다. 일반적인 OS는 가상 메모리를 사용하므로, 프로세스가 할당된 메모리에 처음 접근할 때 물리적 RAM에 해당 페이지를 로드하는 과정에서 예측 불가능한 지연이 발생할 수 있다. 실시간 코드 경로에서는 이러한 지연이 치명적일 수 있다. 이를 방지하기 위해 ROS2의 실시간 데모 및 가이드에서는 `mlockall` 시스템 콜을 사용할 것을 권장한다.6

`mlockall`은 프로세스가 사용하는 모든 메모리(현재 및 미래에 할당될 메모리)를 물리적 RAM에 고정시켜 페이지 폴트가 발생하지 않도록 보장한다. ROS2의 `pendulum_demo`는 이러한 실시간 프로그래밍 기법(고우선순위 스레드 설정, `mlockall` 사용 등)을 실제로 적용한 좋은 예시이다.6


OS 수준의 지원과 더불어, ROS2 노드 내부의 콜백(callback) 스케줄링 메커니즘인 Executor의 진화는 실시간성 확보를 위한 핵심적인 과정이었다. Executor는 구독, 타이머, 서비스 등에서 발생하는 이벤트를 어떤 순서로, 어떤 스레드에서 처리할지를 결정한다.48

- **초기 모델 (Foxy 이전)**: `SingleThreadedExecutor`와 `MultiThreadedExecutor`가 기본으로 제공되었다.4 이들은 `wait-set`이라는 폴링(polling) 기반 메커니즘을 사용한다. 즉, 매 주기마다 모든 엔티티(구독, 타이머 등)를 확인하여 처리할 작업이 있는지 검사한다. 이 방식은 구현이 간단하지만, 작업이 없는 엔티티까지 매번 확인해야 하므로 비효율적이며, 우선순위 역전과 같은 실시간 시스템의 고질적인 문제를 해결하지 못했다.48

- **최적화의 시작 (Galactic ~ Humble)**: `StaticSingleThreadedExecutor`가 도입되었다.48 이는 로봇 시스템의 노드 구성이 런타임 중에 거의 변하지 않는다는 점에 착안하여, 매번 엔티티 목록을 새로 구축하는 오버헤드를 제거함으로써 성능을 개선했다.

- **패러다임의 전환 (Iron ~ Jazzy)**: `EventsExecutor`가 실험적으로 도입되면서 큰 변화가 시작되었다.51

  `EventsExecutor`는 `wait-set` 대신 이벤트 큐(event-queue)를 기반으로 동작한다. 이벤트를 수신한 엔티티만 큐에 추가되므로, 작업이 없는 엔티티를 스캔하는 오버헤드가 원천적으로 사라진다. 또한, 이벤트 도착 순서에 기반한 처리가 가능해져 기존 Executor보다 더 정확하고 효율적인 스케줄링을 제공한다.

- **성숙과 안정화 (Jazzy 이후)**: Jazzy 릴리스에서는 `StaticSingleThreadedExecutor`의 최적화 개념(엔티티 목록 재사용)이 다른 `wait-set` 기반 Executor들에도 일반화되어 전반적인 성능이 향상되었다.51 이로 인해 

  `StaticSingleThreadedExecutor`는 사실상 그 역할을 다하게 되었다. 동시에, `MultiThreadedExecutor`에서 발견된 기아(starvation) 현상과 같은 미묘한 동시성 버그들이 실제 고부하 환경에서의 사용 사례를 통해 발견되고 수정되었다.54

Executor의 이러한 진화 과정은 ROS2 커뮤니티가 실시간 스케줄링 문제에 대한 이해를 점차 심화시키고 있음을 보여준다. 초기에는 기능 구현에 중점을 두었다면, 이제는 실제 산업 현장에서 요구하는 성능과 결정론성을 만족시키기 위해 아키텍처 수준의 개선이 이루어지고 있다. `EventsExecutor`를 기본 Executor로 채택하려는 논의가 진행 중이며, 이는 ROS2가 더 높은 수준의 실시간 성능을 기본으로 제공하는 방향으로 나아가고 있음을 시사한다.51


ROS1의 가장 큰 약점 중 하나는 보안 기능의 부재였다. ROS2는 DDS-Security 표준을 기반으로 한 SROS2 툴킷을 통해 이를 근본적으로 해결했다.3 SROS2는 세 가지 핵심 보안 기능을 제공한다: 인증(Authentication), 암호화(Encryption), 접근 제어(Access Control).

- **인증 및 암호화**: 공개 키 기반 구조(PKI)를 사용하여 각 ROS2 노드에 신원을 부여한다. 인증 기관(Certificate Authority, CA)이 서명한 인증서를 통해 노드들은 서로의 신원을 확인하며, 신뢰할 수 있는 노드 간의 모든 통신은 DDS 계층에서 자동으로 암호화된다.3
- **접근 제어**: 어떤 노드가 어떤 토픽에 발행(publish)하거나 구독(subscribe)할 수 있는지, 또는 어떤 서비스를 호출할 수 있는지를 세밀하게 제어한다. 이러한 규칙은 서명된 XML 형식의 권한 파일(permissions file)에 정의된다.57
- **보안 엔클레이브(Enclave)**: SROS2는 '엔클레이브'라는 개념을 도입했다.59 이는 동일한 보안 ID와 접근 제어 규칙을 공유하는 프로세스 그룹이다. 과거에는 각 노드가 개별적인 보안 주체였지만, 엔클레이브 모델을 통해 여러 노드를 하나의 보안 컨텍스트로 묶어 관리의 복잡성을 줄였다.

DDS-Security 자체는 설정이 매우 복잡하지만, SROS2의 진정한 가치는 이를 쉽게 사용할 수 있도록 만드는 '사용성 높은 도구'에 있다.3

`sros2` 커맨드 라인 도구는 CA 생성, 키와 인증서 발급, 기본 정책 파일 생성 등 복잡한 과정을 자동화한다.57 심지어 실행 중인 시스템의 통신 패턴을 분석하여 필요한 권한 정책의 초안을 자동으로 생성해주는 기능도 지원한다.3 이러한 도구들은 보안 설정의 장벽을 낮춰 개발자들이 실제로 보안 기능을 활성화하도록 유도하는 중요한 역할을 한다.

또한, ROS2는 배포판이 발전함에 따라 보안 기능도 지속적으로 강화했다. 예를 들어, Humble Hawksbill에서는 인증서 폐기 목록(Certificate Revocation List, CRL) 지원이 추가되었다.59 이를 통해 특정 노드의 키가 유출되었을 경우, 해당 노드의 인증서를 무효화하여 시스템 전체의 보안을 유지할 수 있다. 이는 장기간 운영되는 상용 로봇 시스템의 보안 생명주기 관리에 필수적인 기능이다.



ROS1의 빌드 시스템인 `catkin`은 ROS와 CMake에 깊이 의존하는 구조였다.63 ROS2는 `colcon`이라는 새로운 빌드 도구를 도입했는데, 이는 단순히 이름을 바꾼 것이 아니라 근본적인 철학의 변화를 의미한다.8

`colcon`은 ROS에 특화되지 않은 범용 빌드 도구 오케스트레이터이다. 즉, CMake 패키지뿐만 아니라 순수 Python 패키지 등 다양한 유형의 패키지를 빌드하는 방법을 알고 있다.64 이러한 분리 덕분에 ROS2의 빌드 생태계는 더 모듈화되고 확장성이 높아졌으며, 일반적인 CI/CD 시스템과의 통합도 용이해졌다.65

`colcon`은 `catkin`과 몇 가지 중요한 차이점을 가진다 8:

1. **격리된 빌드(Isolated Build)만 지원**: `catkin`은 여러 패키지를 하나의 CMake 컨텍스트에서 빌드할 수 있었지만, 이는 타겟 이름 충돌 등의 문제를 야기했다. `colcon`은 각 패키지를 독립적으로 빌드하여 이러한 문제를 원천적으로 방지한다.
2. **`devel` 공간 제거**: ROS1의 `devel` 공간은 설치 과정 없이 소스 코드를 바로 사용할 수 있게 해주는 편리한 기능이었지만, 모든 패키지가 이를 지원하기 위해 추가적인 CMake 코드를 작성해야 하는 부담이 있었다. `colcon`은 `devel` 공간을 없애고, 대신 `install` 공간에 원본 파일을 복사하는 대신 심볼릭 링크를 생성하는 `--symlink-install` 옵션을 제공한다. 이를 통해 Python 코드나 런치 파일 등을 수정한 후 다시 빌드할 필요 없이 즉시 변경 사항을 적용할 수 있어 `devel` 공간의 장점을 유지하면서 빌드 프로세스를 단순화했다.

이러한 변화들은 ROS2를 외부 개발자들에게 더 친숙하고 표준적인 개발 환경으로 만들려는 노력의 일환이다. `ament`라는 보조 패키지들은 `catkin`의 편의 기능들을 계승하여 `colcon`이 ROS 패키지를 원활하게 빌드할 수 있도록 돕는다.66


- **`ros2cli`**: ROS1의 `rostopic`, `rosnode` 등 여러 커맨드가 `ros2`라는 단일 진입점으로 통합되어 일관성 있는 사용자 경험을 제공한다. 각 명령어는 하위 동사(verb) 형태로 제공되며, 플러그인 기반으로 확장 가능하다.67
- **`launch` 시스템**: ROS1의 XML 기반 런치 파일은 정적이고 기능이 제한적이었다. ROS2는 Python 기반 런치 시스템을 도입하여 프로그래밍 방식의 동적이고 유연한 시스템 구성이 가능해졌다.4 조건문, 반복문, 이벤트 핸들러 등을 사용하여 복잡한 시나리오에 맞춰 노드를 실행하고 관리할 수 있다.
- **`rosbag2`**: 데이터 로깅 및 재생 도구인 `rosbag`은 ROS2에서 `rosbag2`로 재탄생하며 크게 발전했다.
  - **QoS 지원 (Foxy)**: Foxy Fitzroy에서는 모든 QoS 설정을 지원하여 데이터를 기록하고 재생할 수 있게 되어, 손실이 많은 네트워크 환경의 데이터도 정확하게 분석할 수 있게 되었다.13
  - **MCAP 포맷 (Iron)**: Iron Irwini부터는 MCAP이 기본 저장 포맷으로 채택되었다.69 MCAP은 메시지 정의를 파일 내에 포함하고 있어 백 파일 자체만으로 이식성이 높고, 기존 SQLite 기반 포맷보다 2~5배 높은 처리량을 보여준다.69
  - **서비스 기록/재생 (Jazzy)**: 오랫동안 요청되었던 서비스 호출을 기록하고 재생하는 기능이 Jazzy Jalisco에서 마침내 추가되었다. 이는 시스템의 동작을 디버깅하고 재현하는 데 매우 중요한 기능이다.26

이러한 툴킷의 진화는 ROS2가 단순히 통신 계층만 개선한 것이 아니라, 개발자의 생산성과 편의성을 높이기 위해 개발 워크플로우 전반을 현대화했음을 보여준다.


각 ROS2 배포판은 프레임워크의 점진적인 성숙 과정을 보여주는 이정표이다. Part II에서 다룬 아키텍처의 발전이 어떻게 각 버전에 구체적인 기능으로 구현되었는지, Foxy부터 Kilted까지의 여정을 통해 추적한다.


2020년 6월에 출시된 Foxy Fitzroy는 ROS2의 두 번째 LTS 배포판으로, 산업계와 상용 제품 개발자들이 본격적으로 ROS2를 채택할 수 있는 안정적인 기반을 마련하는 데 중점을 두었다.13 Foxy는 새로운 기능의 도입보다는 기존 기능들을 다듬고 안정화하는 데 집중했다.

주요 개선 사항은 개발자의 디버깅 및 분석 능력을 향상시키는 데 초점이 맞춰졌다. `rosbag2`는 모든 QoS 설정을 완벽하게 지원하도록 개선되었고, 데이터 압축 기능이 추가되어 대용량 센서 데이터를 효율적으로 저장하고 재생할 수 있게 되었다.13 이는 신뢰성 있는(reliable) 통신뿐만 아니라 최선형(best-effort) 통신을 사용하는 센서 데이터까지도 정확하게 로깅하고 분석할 수 있게 되었음을 의미한다.

보안 측면에서는 SROS2에 '엔클레이브(enclave)' 개념이 도입되어, 여러 노드를 하나의 보안 단위로 묶어 관리하는 것이 가능해졌다.13 이는 복잡한 시스템에서 보안 정책을 더 쉽게 적용하고 관리할 수 있도록 개선한 것이다. 또한, 핵심 클라이언트 라이브러리인 

`rclcpp`와 `rclpy`의 API가 상당 부분 안정화되었으며, Eloquent에서 Foxy로의 마이그레이션 과정에서 많은 API 변경이 이루어졌다. 이는 장기적인 안정성을 위한 기반 다지기 작업의 일환이었다.60 Foxy는 ROS2가 연구 단계를 넘어 실제 생산 환경에서 사용될 수 있는 성숙한 프레임워크임을 증명한 첫 번째 LTS 버전이라고 평가할 수 있다.


2022년 5월에 출시된 Humble Hawksbill은 ROS2의 성능과 보안 기능을 한 단계 끌어올린 중요한 LTS 배포판이다.9 이 버전의 가장 혁신적인 기능은 **타입 어댑테이션(Type Adaptation)**(REP-2007)의 도입이었다.9 이는 ROS 메시지 타입과 사용자 정의 데이터 구조(예: OpenCV의 `cv::Mat`이나 PCL의 `PointCloud`) 간의 변환을 자동화하고, 특히 공유 메모리(shared memory)를 통한 프로세스 간 통신(Intra-Process Communication)에서 불필요한 데이터 복사를 제거하는 '제로 카피(zero-copy)' 전송을 가능하게 했다. NVIDIA Jetson과 같은 하드웨어 가속기를 사용하는 비전 처리 파이프라인에서 타입 어댑테이션을 적용했을 때, 처리량이 최대 7배까지 향상되는 벤치마크 결과가 보고되기도 했다.9

Humble에서는 통신 효율성을 높이는 **콘텐츠 필터링 토픽(Content Filtered Topics)** 기능도 추가되었다.9 이는 구독자가 메시지의 내용(content)을 기반으로 특정 조건에 맞는 메시지만 수신할 수 있게 하는 기능이다. 예를 들어, 특정 ID를 가진 객체의 위치 정보만 구독하거나, 특정 임계값을 넘는 센서 데이터만 수신하는 것이 가능해져 불필요한 데이터 처리와 네트워크 대역폭 낭비를 줄일 수 있다.

보안 기능 역시 크게 강화되었다. SROS2에 **인증서 폐기 목록(Certificate Revocation List, CRL)** 지원이 추가되어, 키가 유출되거나 더 이상 신뢰할 수 없는 노드의 인증서를 무효화하여 시스템에서 퇴출시킬 수 있게 되었다.59 이는 장기간 운영되는 로봇 시스템의 보안 생명주기 관리에 필수적인 기능이다. 이 외에도 런치 파일에서 구성 가능한 노드(composable nodes)를 직접 명시할 수 있게 되는 등 개발자 편의성도 개선되었다.59 Humble은 ROS2가 고성능, 고보안을 요구하는 까다로운 애플리케이션에 대응할 수 있는 강력한 프레임워크로 발전했음을 보여주었다.


2023년 5월에 출시된 Iron Irwini는 비-LTS 배포판으로, 새로운 거대 기능보다는 개발자의 생산성과 시스템 분석 능력을 향상시키는 데 집중했다.16 이 버전의 핵심적인 변화는 시스템의 내부 동작을 더 쉽게 관찰하고 디버깅할 수 있는 도구를 제공하는 것이었다.

가장 주목할 만한 기능은 **서비스 내관(Service Introspection)**이다.69 기존에는 서비스 호출이 언제, 누구에 의해 이루어졌는지 추적하기 어려웠다. 이 기능은 각 서비스에 대해 `/_service_event`라는 시스템 토픽을 자동으로 생성하여, 서비스 요청(request)과 응답(reply)이 발생할 때마다 관련 정보를 메시지로 발행한다. 이를 통해 개발자는 `ros2 topic echo`와 같은 표준 도구를 사용하여 서비스의 상호작용을 실시간으로 모니터링하고 디버깅할 수 있게 되었다.

데이터 로깅 방식도 크게 개선되었다. `rosbag2`의 기본 저장 포맷으로 **MCAP**이 채택되었다.69 MCAP은 메시지 정의를 파일 자체에 내장하여 백 파일의 이식성을 높이고, 기존 SQLite 포맷 대비 2~5배 빠른 기록/재생 성능을 제공한다.69 이는 대규모 데이터를 다루는 자율주행 및 필드 로보틱스 분야에서 데이터 분석 워크플로우를 크게 개선했다.

또한, **파라미터 콜백(Parameter Callbacks)** 기능이 강화되어, 파라미터 변경 전(`pre_set`)과 후(`post_set`)에 실행되는 콜백을 등록할 수 있게 되었다.69 이를 통해 파라미터 값의 유효성을 검사하거나, 변경에 따른 시스템의 상태를 동기화하는 등 더 정교하고 안정적인 동적 재구성(dynamic reconfiguration) 로직을 구현할 수 있게 되었다. 이 외에도 Python API 문서가 공식 문서 사이트(docs.ros.org)로 통합되어 접근성이 향상되는 등, Iron은 개발자의 삶의 질을 높이는 데 기여한 내실 있는 릴리스였다.69


2024년 5월에 출시된 Jazzy Jalisco는 최신 LTS 배포판으로, Iron에서 시작된 개발자 경험 향상의 흐름을 이어가면서 오랫동안 커뮤니티에서 요청해 온 핵심 기능들을 완성했다.16

Jazzy의 가장 큰 성과는 `rosbag2`에서 **서비스를 기록하고 재생하는 기능**을 완벽하게 지원하게 된 것이다.26 이는 Iron의 서비스 내관 기능을 기반으로 구현되었으며, 이제 개발자는 토픽 데이터뿐만 아니라 시스템의 상태를 변화시키는 주요 원인인 서비스 호출까지 완벽하게 재현하여 디버깅할 수 있게 되었다. 이와 함께 토픽의 타입(`--topic_types`)으로 기록 대상을 필터링하는 등 `rosbag2`의 사용성이 대폭 향상되었다.71

시뮬레이션 환경과의 통합도 크게 개선되었다. ROS 버전별로 어떤 Gazebo 버전을 사용해야 할지 혼란스러웠던 문제를 해결하기 위해, Jazzy부터는 **특정 Gazebo 버전(Gazebo Harmonic)을 공식 권장**하고, 설치를 간소화하는 벤더 패키지를 함께 배포한다.26 이는 ROS와 Gazebo를 함께 사용하는 입문자들의 장벽을 크게 낮추는 중요한 변화이다.

내부적으로는 `rclcpp`의 Executor에서 발견되었던 데이터 경쟁(data race) 조건과 같은 동시성 문제들이 수정되었고, `wait-set`의 생성 및 삭제 오버헤드를 줄이는 등 성능 최적화가 이루어졌다.53 또한, RViz2가 ROS1의 RViz1과 거의 모든 기능에서 동등한 수준(feature parity)을 갖추게 되어 시각화 도구의 완성도를 높였다.76 Jazzy는 안정성과 신뢰성을 바탕으로 개발자들이 더 편리하고 효율적으로 로봇 애플리케이션을 개발할 수 있는 환경을 제공하는 데 성공한 LTS 릴리스이다.


ROS2의 발전은 현재 진행형이다. 공식 로드맵을 통해 차기 배포판들의 개발 방향을 예측할 수 있다.77

2025년 5월 출시 예정인 **Kilted Kaiju**의 가장 중요한 목표는 **`rmw_zenoh_cpp`를 1등급(Tier 1) RMW로 승격**시키는 것이다.43 이는 Zenoh가 Fast DDS, Cyclone DDS와 동등한 수준의 공식 지원을 받게 됨을 의미하며, ROS2가 DDS의 한계를 넘어 대규모 분산 시스템과 불안정한 네트워크 환경으로 본격적으로 확장됨을 알리는 신호탄이다. 이와 함께 `rclpy`의 성능 향상을 위한 이벤트 기반 Executor 도입, Conda/Pixi를 이용한 Windows 설치 경험 개선 등도 Kilted의 주요 개발 항목이다.43

2026년 출시될 LTS 배포판의 이름은 **Lyrical Luth**로 확정되었다.45 구체적인 로드맵은 아직 공개되지 않았지만, Kilted에서 도입될 Zenoh의 안정화와 확산, 그리고 지속적인 실시간 성능 및 개발 도구 개선이 주요 과제가 될 것으로 예상된다.

ROS2의 로드맵은 초기에는 ROS1과의 기능 동등성 확보와 API 안정화에 집중했다면, 이제는 커뮤니티와 산업계의 실제 요구사항, 즉 다양한 환경에서의 성능, 확장성, 사용성 문제를 해결하는 방향으로 진화하고 있다. 이는 ROS2가 성숙한 프레임워크로서 실제 현장의 복잡한 문제들을 해결하기 위해 발전하고 있음을 보여주는 긍정적인 신호이다.


ROS2의 핵심 아키텍처 개선은 단순히 프레임워크 자체의 성능을 높이는 데 그치지 않고, 생태계 전반의 패키지들이 더 안정적이고 고도화된 기능을 구현할 수 있는 토대를 제공했다. 자율 주행의 핵심인 Navigation2(Nav2)와 로봇팔 제어의 표준인 MoveIt2가 ROS2의 새로운 기능들을 어떻게 활용하여 발전했는지 살펴본다.


ROS1의 내비게이션 스택을 계승한 Nav2는 ROS2로 전환되면서 단순한 포팅을 넘어, 산업 현장에서 요구하는 수준의 안정성과 강건성을 갖춘 시스템으로 재탄생했다.82 이러한 변화의 중심에는 ROS2의 **생명주기 노드(Lifecycle Nodes)**가 있다.

ROS1에서는 여러 내비게이션 관련 노드를 동시에 실행할 때, 어떤 노드가 먼저 준비되는지에 대한 보장이 없어 경쟁 상태(race condition)가 발생하거나 불안정한 초기화 문제를 겪는 경우가 많았다. Nav2는 플래너, 컨트롤러, 맵 서버 등 핵심 구성 요소들을 모두 생명주기 노드로 구현하여 이 문제를 해결했다.84

`nav2_lifecycle_manager`라는 전용 관리자 노드는 이들 노드의 상태를 순차적으로 제어한다. 시스템이 시작되면, `nav2_lifecycle_manager`는 각 노드를 `Unconfigured` 상태에서 `Configuring` 전환을 통해 설정 파일을 로드하고, 모든 노드의 설정이 완료되면 `Activating` 전환을 통해 `Active` 상태로 만든다.86 이 과정을 통해 맵 서버가 맵을 로드하고, 위치 추정(localization) 시스템이 활성화된 후에야 경로 계획기가 목표 지점을 수신하는 등, 모든 구성 요소가 정해진 순서에 따라 완벽하게 준비된 상태에서만 전체 시스템이 동작하도록 보장한다. 이는 예측 불가능한 환경에서 로봇이 안전하게 동작해야 하는 상용 제품에 필수적인 기능이다.

`nav2_bringup` 패키지는 이러한 복잡한 생명주기 관리 과정을 추상화하여, 사용자가 단일 런치 파일 실행만으로 전체 내비게이션 스택을 안정적으로 시작할 수 있게 해준다.87 또한, Nav2는 ROS1의 유한 상태 기계(Finite State Machine) 대신 **행동 트리(Behavior Trees)**를 도입하여 내비게이션 로직을 구성한다.89 행동 트리는 '경로 계획', '경로 추종', '장애물 회피'와 같은 모듈화된 행동들을 유연하게 조합하여 복잡하고 지능적인 로봇 동작을 설계할 수 있게 해준다. 이처럼 Nav2는 ROS2의 핵심 아키텍처인 생명주기 노드를 적극적으로 활용하여, 연구용 도구를 넘어 신뢰성 있는 산업용 부품으로 진화했다.


로봇팔의 동작 계획 및 제어를 위한 통합 플랫폼인 MoveIt 역시 ROS2로 전환되면서 `MoveIt2`로 발전했다. MoveIt2의 가장 큰 구조적 개선은 **`ros2_control`** 프레임워크와의 명확하고 표준화된 통합이다.10

ROS1에서는 MoveIt과 실제 로봇 하드웨어 간의 인터페이스가 표준화되어 있지 않아 각 로봇 제조사마다 다른 방식으로 구현되는 경우가 많았다. ROS2에서는 `ros2_control`이 하드웨어 추상화 계층(Hardware Abstraction Layer)의 역할을 명확히 수행한다. MoveIt2는 상위 레벨의 동작 계획(motion planning)을 담당하고, `ros2_control`은 하드웨어에 대한 저수준 제어를 담당하는 역할 분담이 이루어진다.

MoveIt2의 **Setup Assistant**를 사용하면 로봇의 URDF 모델로부터 `JointTrajectoryController`, `GripperCommandController` 등 필요한 `ros2_control` 컨트롤러 설정을 쉽게 생성할 수 있다.93 MoveIt2는 충돌 없는 경로를 계산하여 관절 궤적(`JointTrajectory`)을 생성한 후, `FollowJointTrajectory`라는 표준 ROS 액션(Action) 인터페이스를 통해 이 궤적을 `ros2_control`에 전달한다.92 그러면 `ros2_control`은 해당 궤적을 받아 실제 로봇의 모터를 제어한다.

이러한 구조는 관심사의 분리(Separation of Concerns) 원칙을 명확히 따른다. MoveIt2는 로봇의 모터나 드라이버의 구체적인 사양을 알 필요가 없으며, 하드웨어 드라이버는 동작 계획 알고리즘에 대해 알 필요가 없다. 이 덕분에 새로운 로봇 하드웨어를 MoveIt 생태계에 통합하는 과정이 훨씬 단순해졌고, 코드의 재사용성과 유지보수성이 크게 향상되었다. 또한, MoveIt2는 복잡한 다단계 작업을 설계하기 위한 **MoveIt Task Constructor (MTC)**를 도입하여, 단순한 지점 간 이동을 넘어 '물체 집기-이동-놓기(pick-and-place)'와 같은 복잡한 매니퓰레이션 작업을 모듈화된 방식으로 구성할 수 있게 되었다.95



본 보고서에서 분석한 바와 같이, ROS2의 발전 과정은 연구 중심의 프레임워크가 상용 및 산업 애플리케이션의 엄격한 요구사항을 충족시키기 위해 체계적으로 강화되는 여정이었다. 이 진화의 핵심 동력은 다음과 같은 아키텍처적 기둥들이다.

1. **교체 가능한 RMW**: DDS라는 산업 표준을 채택하면서도 특정 벤더에 종속되지 않고, 나아가 Zenoh와 같은 차세대 프로토콜을 수용할 수 있는 유연성을 확보했다.
2. **실시간 Executor**: 단순한 폴링 방식에서 벗어나 고성능 이벤트 기반 스케줄링으로 발전하며, 결정론적 시스템을 구축할 수 있는 기반을 다졌다.
3. **포괄적인 보안**: DDS-Security를 기반으로 한 SROS2는 사용하기 쉬운 도구를 통해 인증, 암호화, 접근 제어를 제공하여 실제 제품에 적용 가능한 보안 체계를 구축했다.
4. **현대적인 툴체인**: `colcon` 빌드 시스템, Python 기반 런치 파일, 고성능 `rosbag2` 등은 개발자의 생산성을 높이고 현대 소프트웨어 개발 워크플로우에 부합하는 환경을 제공했다.

이러한 핵심적인 변화들은 Nav2와 MoveIt2와 같은 주요 생태계 패키지들이 생명주기 관리, 표준화된 하드웨어 제어 등의 기능을 통해 이전에는 불가능했던 수준의 안정성과 모듈성을 달성하도록 만들었다. ROS2는 더 이상 ROS1의 연장선이 아니라, 실제 세상의 복잡하고 까다로운 문제를 해결하기 위한 생산 등급의 플랫폼으로 자리매김했다.


ROS2의 복잡성과 빠른 발전 속도를 고려할 때, 개발자와 조직은 전략적인 접근 방식을 취해야 한다. 본 분석을 바탕으로 다음과 같은 구체적인 권장 사항을 제시한다.

- **버전 선택 전략**:

  - **생산 시스템**: 안정성과 장기적인 유지보수가 최우선인 상용 제품이나 산업용 시스템에는 반드시 최신 **LTS 배포판(현재 기준 Humble 또는 Jazzy)**을 선택해야 한다.19 5년의 지원 기간은 제품의 수명주기 동안 안정적인 보안 패치와 버그 수정을 보장한다.
  - **연구 및 개발**: Zenoh와 같은 최신 기능을 선도적으로 도입해야 하는 연구 프로젝트나 신기술 프로토타이핑의 경우, 최신 **비-LTS 버전이나 `Rolling` 배포판**을 사용하는 것을 고려할 수 있다.18 단, 이는 API 변경이 잦고 지원 기간이 짧다는 점을 명확히 인지하고, 향후 LTS 버전으로의 마이그레이션 계획을 수립해야 한다.

- **마이그레이션 계획**:

  - **LTS 간 마이그레이션**: Humble에서 Jazzy로의 이전과 같이 LTS 버전 간의 마이그레이션은 일반적으로 큰 무리 없이 진행될 수 있다. 그러나 `rclcpp`, `rclpy` 등 핵심 라이브러리의 API 변경 사항이 있을 수 있으므로, 반드시 공식 마이그레이션 가이드를 철저히 검토해야 한다.99 종종 가장 큰 변화는 ROS 자체보다 기반이 되는 Ubuntu 버전을 업그레이드하는 과정(예: 22.04에서 24.04로)에서 발생하므로, 시스템 전체의 의존성을 점검하는 것이 중요하다.102

  - **ROS1에서 마이그레이션**: ROS1 Noetic의 지원이 2025년 5월에 종료되므로, ROS1 기반 시스템의 ROS2로의 전환은 더 이상 선택이 아닌 필수 과제이다.14

    `ros1_bridge`는 점진적인 전환을 도울 수 있지만, 장기적으로는 모든 코드를 ROS2 네이티브로 재작성하는 것을 목표로 해야 한다. 생명주기 노드, `ros2_control`, Python 기반 런치 파일 등 ROS2의 새로운 패러다임을 적극적으로 수용하는 것이 마이그레이션의 성공률을 높이는 길이다.

- **미래 기술 대비**:

  - **Zenoh 도입 검토**: 다개체 로봇 시스템, 원격 제어, 불안정한 무선 네트워크 환경(Wi-Fi, 5G)을 다루는 애플리케이션을 설계하는 아키텍트는 지금부터 Zenoh의 도입을 적극적으로 검토해야 한다. Kilted Kaiju부터 1등급 RMW로 지원되므로, Zenoh는 곧 주류 기술이 될 것이다.27
  - **Executor 성능 분석**: 실시간 성능이 중요한 애플리케이션의 경우, 기본 `SingleThreadedExecutor`에만 의존하지 말고, `MultiThreadedExecutor`와 `EventsExecutor`의 성능을 실제 워크로드 환경에서 벤치마킹해야 한다. 특히 `EventsExecutor`는 많은 시나리오에서 상당한 성능 향상을 제공할 잠재력을 가지고 있다.51

결론적으로, ROS2는 로보틱스 분야의 표준 플랫폼으로서 그 입지를 공고히 하고 있다. 그 발전의 궤적을 이해하고 각 버전의 특징과 아키텍처 변화의 의미를 파악하는 것은, 모든 로봇 공학 전문가가 더 안정적이고, 효율적이며, 혁신적인 로봇 시스템을 구축하는 데 있어 필수적인 역량이 될 것이다.


1. Robot Operating System - Wikipedia, accessed July 22, 2025, https://en.wikipedia.org/wiki/Robot_Operating_System
2. The Evolution of Robot Operating Systems: A Review of ROS 2 - ResearchGate, accessed July 22, 2025, https://www.researchgate.net/publication/392620565_The_Evolution_of_Robot_Operating_Systems_A_Review_of_ROS_2
3. SROS2: Usable Cyber Security Tools for ROS 2 - Alias Robotics, accessed July 22, 2025, https://aliasrobotics.com/files/SROS2.pdf
4. ROS1 vs ROS2, Practical Overview For ROS Developers - The Robotics Back-End, accessed July 22, 2025, https://roboticsbackend.com/ros1-vs-ros2-practical-overview/
5. From ROS to ROS 2: A Comprehensive Guide to the Next Generation Robotics | by ScaleX Innovation, accessed July 22, 2025, https://scalexi.medium.com/from-ros-to-ros-2-a-comprehensive-guide-to-the-next-generation-robotics-f93a4e2e5793
6. Understanding real-time programming - ROS 2 Documentation: Foxy documentation, accessed July 22, 2025, https://docs.ros.org/en/foxy/Tutorials/Demos/Real-Time-Programming.html
7. Introduction of Robot Operating Systems 2: ROS2 and compared with ROS - XiaoR GEEK, accessed July 22, 2025, https://www.xiaorgeek.net/blogs/news/introduction-of-robot-operating-systems-2-ros2-and-compared-with-ros
8. Changes between ROS 1 and ROS 2 - ROS2 Design, accessed July 22, 2025, http://design.ros2.org/articles/changes.html
9. ROS 2 Humble Hawksbill Release! - Open Robotics, accessed July 22, 2025, https://www.openrobotics.org/blog/2022/5/24/ros-2-humble-hawksbill-release
10. The MoveIt 2 journey (part 0): why MoveIt 2? - MoveIt - ROS Discourse, accessed July 22, 2025, https://discourse.ros.org/t/the-moveit-2-journey-part-0-why-moveit-2/8461
11. Appendix: ROS 2 - 240AR060 - Introduction to ROS, accessed July 22, 2025, https://sir.upc.edu/projects/rostutorials/appendixRos2/index.html
12. ROS 2 (Robot Operating System): overview and key points for robotics software | Robotnik ®, accessed July 22, 2025, https://robotnik.eu/ros-2-robot-operating-system-overview-and-key-points-for-robotics-software/
13. ROS 2 Foxy Fitzroy: Setting a new standard for production robot development - AWS, accessed July 22, 2025, https://aws.amazon.com/blogs/robotics/ros2-foxy-fitzroy-robot-development/
14. Distributions - ROS Wiki, accessed July 22, 2025, http://wiki.ros.org/Distributions
15. REP 2000 -- ROS 2 Releases and Target Platforms (ROS.org), accessed July 22, 2025, https://www.ros.org/reps/rep-2000.html
16. ROS 2 | endoflife.date, accessed July 22, 2025, https://endoflife.date/ros-2
17. Distributions - ROS 2 Documentation: Dashing documentation, accessed July 22, 2025, https://docs.ros.org/en/dashing/Releases.html
18. Distributions - ROS 2 Documentation: Foxy documentation, accessed July 22, 2025, https://docs.ros.org/en/foxy/Releases.html
19. ROS2 Galactic EOL - Robotics Stack Exchange, accessed July 22, 2025, https://robotics.stackexchange.com/questions/99089/ros2-galactic-eol
20. Distributions - ROS 2 Documentation: Xin 文档, accessed July 22, 2025, https://daobook.github.io/ros2-docs/xin/Releases.html
21. Distributions - ROS 2 Documentation: Crystal documentation, accessed July 22, 2025, https://docs.ros.org/en/crystal/Releases.html
22. ROS 2 middleware interface - ROS2 Design, accessed July 22, 2025, https://design.ros2.org/articles/ros_middleware_interface.html
23. 2023-09 ROS 2 RMW alternate - AWS, accessed July 22, 2025, https://cdck-file-uploads-us1.s3.dualstack.us-west-2.amazonaws.com/flex022/uploads/ros/original/3X/a/9/a9412734d6896de23fa64665cf518f232a46a5b8.pdf
24. RMW implementations - ROS 2 Documentation: Humble documentation, accessed July 22, 2025, https://docs.ros.org/en/humble/Installation/RMW-Implementations.html
25. ros2/rmw_fastrtps: Implementation of the ROS Middleware (rmw) Interface using eProsima's Fast RTPS. - GitHub, accessed July 22, 2025, https://github.com/ros2/rmw_fastrtps
26. OSRF releases ROS 2 Jazzy Jalisco - The Robot Report, accessed July 22, 2025, https://www.therobotreport.com/ros-2-jazzy-jalisco-released-osrf/
27. Blog - ROS-Industrial, accessed July 22, 2025, https://rosindustrial.org/news
28. 2021 ROS Middleware Evaluation Report | TSC-RMW-Reports - GitHub Pages, accessed July 22, 2025, https://osrf.github.io/TSC-RMW-Reports/humble/
29. Default DDS provider template | TSC-RMW-Reports - GitHub Pages, accessed July 22, 2025, https://osrf.github.io/TSC-RMW-Reports/humble/dds_provider_question_template.html
30. Fast DDS TSC RMW report 2021 - GitHub Pages, accessed July 22, 2025, https://osrf.github.io/TSC-RMW-Reports/humble/eProsima-response.html
31. Iron Irwini (iron) - ROS 2 Documentation: Iron documentation, accessed July 22, 2025, https://docs.ros.org/en/iron/Releases/Release-Iron-Irwini.html
32. Fast DDS selected as the ROS 2 Humble Default Middleware - Open Robotics Discourse, accessed July 22, 2025, https://discourse.openrobotics.org/t/fast-dds-selected-as-the-ros-2-humble-default-middleware/23374
33. Fast DDS selected default RMW for next ROS 2 - eProsima, accessed July 22, 2025, https://www.eprosima.com/news/fast-dds-default-rmw-ros-2
34. FastDDS without Discovery Server? - ROS General - Open Robotics Discourse, accessed July 22, 2025, https://discourse.ros.org/t/fastdds-without-discovery-server/26117
35. DDS tuning information - ROS 2 Documentation: Eloquent documentation, accessed July 22, 2025, https://docs.ros.org/en/eloquent/Troubleshooting/DDS-tuning.html
36. DDS tuning information - ROS 2 Documentation: Foxy documentation, accessed July 22, 2025, https://docs.ros.org/en/foxy/How-To-Guides/DDS-tuning.html
37. DDS Tuning for ROS 2 - breq.dev, accessed July 22, 2025, https://breq.dev/2024/05/17/dds
38. Zenoh- Mobile Robotics through Communication and Scalability - Bisinfotech, accessed July 22, 2025, https://www.bisinfotech.com/enhancing-mobile-robotics-through-efficient-communication-and-scalability/
39. ZettaScale designs Zenoh to transcend DDS for automotive, ROS communications, accessed July 22, 2025, https://www.therobotreport.com/zettascale-designs-zenoh-to-transcend-dds-for-automotive-ros-communications/
40. eclipse-zenoh/zenoh-plugin-dds: A zenoh plug-in that allows to transparently route DDS data. This plugin can be used by DDS applications to leverage zenoh for geographical routing or for better scaling discovery. For ROS2 robotic applications, use https://github.com/eclipse-zenoh/zenoh- - GitHub, accessed July 22, 2025, https://github.com/eclipse-zenoh/zenoh-plugin-dds
41. Deployment / Zenoh - pub/sub, geo distributed storage, query, accessed July 22, 2025, https://zenoh.io/docs/getting-started/deployment/
42. Edge Robotics with Eclipse zenoh and ROS 2, accessed July 22, 2025, https://www.eclipse.org/community/eclipse_newsletter/2020/september/1.php
43. ROS 2 Kilted Kaiju Release! - Open Robotics Discourse, accessed July 22, 2025, https://discourse.ros.org/t/ros-2-kilted-kaiju-release/43902
44. Roadmap - Kilted documentation - the official ROS docs, accessed July 22, 2025, https://docs.ros.org/en/kilted/The-ROS2-Project/Roadmap.html
45. ROS 2 Kilted Kaiju released - Open Robotics, accessed July 22, 2025, https://www.openrobotics.org/blog/2025/5/23/ros-2-kilted-kaiju-released
46. Introduction to Real-time Systems - ROS 2 Design, accessed July 22, 2025, https://design.ros2.org/articles/realtime_background.html
47. Realtime ROS2 | PDF | Real Time Computing - Scribd, accessed July 22, 2025, https://www.scribd.com/document/323945751/RealtimeROS2
48. Executors - ROS 2 Documentation: Foxy documentation, accessed July 22, 2025, https://docs.ros.org/en/foxy/Concepts/About-Executors.html
49. Executors - ROS 2 Documentation: Rolling documentation, accessed July 22, 2025, https://docs.ros.org/en/rolling/Concepts/Intermediate/About-Executors.html
50. Speed Up Data Processing with Multi-Threaded Execution - The Construct, accessed July 22, 2025, https://www.theconstruct.ai/speed-up-data-processing-with-multi-threaded-execution-english-ros2-tutorial/
51. The ROS 2 C++ Executors - General - Open Robotics Discourse, accessed July 22, 2025, https://discourse.ros.org/t/the-ros-2-c-executors/38296
52. Executors in ROS 2, accessed July 22, 2025, https://roscon.ros.org/2024/talks/Executors_in_ROS_2.pdf
53. Jazzy Jalisco (jazzy) - ROS 2 Documentation: Humble documentation, accessed July 22, 2025, https://docs.ros.org/en/humble/Releases/Release-Jazzy-Jalisco.html
54. [EmSoft'24 best paper] Thread Carefully: Preventing Starvation In The ROS 2 Multi-Threaded Executor - ACM SIGBED, accessed July 22, 2025, https://sigbed.org/2025/05/01/thread-carefully-preventing-starvation-in-the-ros-2-multi-threaded-executor/
55. ros2/sros2: tools to generate and distribute keys for SROS 2 - GitHub, accessed July 22, 2025, https://github.com/ros2/sros2
56. ROS2 Security - ROS Discourse, accessed July 22, 2025, https://discourse.ros.org/t/ros2-security/2273
57. ROS 2 DDS-Security integration - ROS2 Design, accessed July 22, 2025, https://design.ros2.org/articles/ros2_dds_security.html
58. ROS 2 Access Control Policies, accessed July 22, 2025, https://design.ros2.org/articles/ros2_access_control_policies.html
59. ROS 2 Humble security, a tour of the new and improved features - Ubuntu, accessed July 22, 2025, https://ubuntu.com/blog/ros-2-humble-security-a-tour-of-the-new-and-improved-features
60. Foxy Fitzroy (foxy) - ROS 2 Documentation: Foxy documentation, accessed July 22, 2025, https://docs.ros.org/en/foxy/Releases/Release-Foxy-Fitzroy.html
61. SROS2: Usable Cyber Security Tools for ROS 2 - ResearchGate, accessed July 22, 2025, https://www.researchgate.net/publication/362469001_SROS2_Usable_Cyber_Security_Tools_for_ROS_2
62. Humble Hawksbill (humble) - ROS 2 Documentation: Foxy ..., accessed July 22, 2025, https://docs.ros.org/en/foxy/Releases/Release-Humble-Hawksbill.html
63. A universal build tool - ROS 2 Design, accessed July 22, 2025, https://design.ros2.org/articles/build_tool.html
64. Using colcon to build packages - ROS 2 Documentation, accessed July 22, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Colcon-Tutorial.html
65. ros-tooling/action-ros-ci: Github Action to build and test ROS 2 packages using colcon, accessed July 22, 2025, https://github.com/ros-tooling/action-ros-ci
66. The build system "ament_cmake" and the meta build tool "ament_tools" - ROS2 Design, accessed July 22, 2025, https://design.ros2.org/articles/ament.html
67. Features Status - ROS 2 Documentation: Iron documentation, accessed July 22, 2025, https://docs.ros.org/en/iron/The-ROS2-Project/Features.html
68. ROS Blog, accessed July 22, 2025, https://www.ros.org/blog/?utm_source=vectorlogozone&utm_medium=referrer
69. ROS 2 Iron Irwini Features: A Comprehensive Guide to the Ninth Release, accessed July 22, 2025, https://thinkrobotics.com/blogs/learn/ros-2-iron-irwini-features-a-comprehensive-guide-to-the-ninth-release
70. Open Robotics releases ROS 2 Iron Irwini - The Robot Report, accessed July 22, 2025, https://www.therobotreport.com/open-robotics-releases-ros-2-iron-irwini/
71. Jazzy Jalisco (jazzy) - ROS 2 Documentation: Rolling documentation, accessed July 22, 2025, https://docs.ros.org/en/rolling/Releases/Release-Jazzy-Jalisco.html
72. ROS 2 Jazzy Jalisco Released - Open Robotics, accessed July 22, 2025, https://www.openrobotics.org/blog/2024/5/ros-jazzy-jalisco-released
73. New ROS2 release Humble Hawksbill - The Robot Report, accessed July 22, 2025, https://www.therobotreport.com/new-ros2-release-humble-hawksbill/
74. Improve Perception Performance for ROS 2 Applications with NVIDIA Isaac Transport for ROS | NVIDIA Technical Blog, accessed July 22, 2025, https://developer.nvidia.com/blog/improve-perception-performance-for-ros-2-applications-with-nvidia-isaac-transport-for-ros/
75. Open Class - ROS2 Iron Irwini Overview - Robotics & ROS Online Courses | The Construct, accessed July 22, 2025, https://app.theconstruct.ai/live_class/549dba48-8ef7-4334-854e-31d9abd2226d/
76. Welcoming Jazzy Jalisco as the latest ROS 2 release - Intrinsic, accessed July 22, 2025, https://www.intrinsic.ai/blog/posts/welcoming-jazzy-jalisco-as-the-latest-ros-2-release
77. Roadmap - ROS 2 Documentation: Rolling documentation, accessed July 22, 2025, https://docs.ros.org/en/rolling/The-ROS2-Project/Roadmap.html
78. Roadmap - Programming Multiple Robots with ROS 2 - GitHub Pages, accessed July 22, 2025, https://osrf.github.io/ros2multirobotbook/roadmap.html
79. Roadmap - Eloquent documentation - the official ROS docs, accessed July 22, 2025, https://docs.ros.org/en/eloquent/Roadmap.html
80. Roadmap - ROS 2 Documentation: Foxy 文档, accessed July 22, 2025, https://daobook.github.io/ros2-docs/foxy/Roadmap.html
81. Lyrical Luth (codename 'lyrical'; May, 2026) - ROS 2 Documentation - the official ROS docs, accessed July 22, 2025, https://docs.ros.org/en/rolling/Releases/Release-Lyrical-Luth.html
82. Navigation2 Overview, accessed July 22, 2025, https://roscon.ros.org/2019/talks/roscon2019_navigation2_overview_final.pdf
83. ROS 1 to ROS 2 Migration - Black Coffee Robotics, accessed July 22, 2025, https://www.blackcoffeerobotics.com/blog/ros-1-to-ros-2-migration
84. Nav2 implementation - Fixposition Documentation, accessed July 22, 2025, https://docs.fixposition.com/fd/nav2-implementation
85. nav2_lifecycle_manager: Humble 1.1.18 documentation - the official ROS docs, accessed July 22, 2025, https://docs.ros.org/en/ros2_packages/humble/api/nav2_lifecycle_manager/
86. navigation2/nav2_lifecycle_manager/README.md at main - GitHub, accessed July 22, 2025, https://github.com/ros-planning/navigation2/blob/main/nav2_lifecycle_manager/README.md
87. Tuning Guide - Nav2 1.0.0 documentation, accessed July 22, 2025, https://docs.nav2.org/tuning/index.html
88. Getting Started - Nav2 1.0.0 documentation, accessed July 22, 2025, https://docs.nav2.org/getting_started/index.html
89. Navigation Concepts - Nav2 1.0.0 documentation, accessed July 22, 2025, https://docs.nav2.org/concepts/index.html
90. Navigation2 in ROS2 | Autonomous Mobile Robot | Nav2 | Behavior Trees - YouTube, accessed July 22, 2025, https://www.youtube.com/watch?v=sVUKeHMBtpQ
91. Introduction To Nav2 Specific Nodes, accessed July 22, 2025, https://docs.nav2.org/behavior_trees/overview/nav2_specific_nodes.html
92. Low Level Controllers - MoveIt Documentation: Humble documentation, accessed July 22, 2025, https://moveit.picknik.ai/humble/doc/examples/controller_configuration/controller_configuration_tutorial.html
93. MoveIt Setup Assistant, accessed July 22, 2025, https://moveit.picknik.ai/main/doc/examples/setup_assistant/setup_assistant_tutorial.html
94. How to Configure MoveIt 2 for a Simulated Robot Arm - Automatic Addison, accessed July 22, 2025, https://automaticaddison.com/how-to-configure-moveit-2-for-a-simulated-robot-arm/
95. MoveIt 2 Documentation - MoveIt Documentation: Rolling documentation, accessed July 22, 2025, https://moveit.picknik.ai/
96. Blog - Robot & Chisel, accessed July 22, 2025, https://www.robotandchisel.com/blog/
97. How to Set Up the MoveIt 2 Task Constructor – ROS 2 Jazzy - YouTube, accessed July 22, 2025, https://www.youtube.com/watch?v=oVNY3gjWIOo
98. ROS 2 Foxy and ROS Melodic EOL – Keep your robots up and running | Ubuntu, accessed July 22, 2025, https://ubuntu.com/blog/ros-foxy-ros-melodic-eol
99. Migration Guides: Humble to Jazzy - ROS2_Control, accessed July 22, 2025, https://control.ros.org/jazzy/doc/ros2_controllers/doc/migration.html
100. Does ros2 humble code works in ros2 jazzy? : r/ROS - Reddit, accessed July 22, 2025, https://www.reddit.com/r/ROS/comments/1iegfmz/does_ros2_humble_code_works_in_ros2_jazzy/
101. ROS 2 Jazzy Packages for Ubuntu 22.04, accessed July 22, 2025, https://discourse.ros.org/t/ros-2-jazzy-packages-for-ubuntu-22-04/42509
102. Upgrading to Jazzy | Clearpath Robotics Documentation, accessed July 22, 2025, https://docs.clearpathrobotics.com/docs/ros/installation/upgrading/
103. Actuate 2024 recap. - Foxglove, accessed July 22, 2025, https://foxglove.dev/blog/actuate-2024-recap


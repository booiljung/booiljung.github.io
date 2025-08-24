[로봇 통신](./index.md)
# 로봇 및 항공우주 시스템을 위한 데이터 분산 서비스(DDS)와 AFDX의 적용 가능성



본 보고서는 데이터 분산 서비스(DDS)와 항공전자 전이중 스위치 이더넷(AFDX) 기술이 현대 로보틱스 및 항공우주 시스템에 즉시 적용 가능한지에 대한 심층 분석을 제공한다. 결론적으로, DDS와 AFDX는 각기 다른 목적과 시스템 아키텍처 계층에서 이미 성숙하고 즉시 적용 가능한 기술이다. DDS는 유연한 데이터 중심 미들웨어로서의 탁월함 덕분에 로봇 운영 체제 2(ROS 2)를 통해 현대 로보틱스 분야의 사실상 표준으로 자리 잡았다. 반면 AFDX는 안전성 인증을 획득한 항공전자 시스템을 위한 검증되고 높은 결정성을 보장하지만 경직된 네트워크 계층 기술이다. 따라서 미래 시스템의 핵심은 두 기술 중 하나를 선택하는 것이 아니라, 이들의 통합 방안을 이해하고 시간 민감형 네트워킹(TSN)이라는 새로운 전략적 대안의 등장을 고려하는 데 있다.

로보틱스 분야 분석 요약:

DDS는 ROS 2의 핵심 통신 계층으로 채택되었기 때문에 로보틱스 분야에서 즉시 적용 가능하며 신규 개발 프로젝트에 권장되는 기술 경로이다.1 여기서 주요 과제는 기술 채택 자체가 아니라, 최적의 실시간 성능을 달성하기 위해 복잡한 서비스 품질(QoS) 설정을 마스터하는 것이다.3

항공우주 분야 분석 요약:

AFDX는 현재 안전 인증이 필수적인 항공전자 백본 네트워크의 확고한 현역 기술이다.5 DDS는 비(非)안전 필수 시스템이나 상대적으로 덜 중요한 시스템에 즉시 적용 가능하며, 미래 항공전자 능력 환경(FACE™) 표준의 공식 전송 서비스로 지정되어 있다.7 안전 필수 애플리케이션에 DDS를 적용하기 위해서는 RTI Connext DDS Cert와 같은 인증 가능한 구현체와 상당한 규모의 인증 투자가 요구된다.9 이 분야의 전략적 과제는 AFDX의 장기적인 존속 가능성을 평가하고, 더 유연하며 비용 효율적인 DDS-over-TSN 아키텍처로의 전환을 계획하는 것이다.

**핵심 권고 사항:**

- **로보틱스 프로젝트:** ROS 2를 통해 DDS 기반 아키텍처를 표준으로 채택하고, QoS 최적화를 위한 전문 인력 양성에 투자해야 한다.
- **항공우주 프로젝트:** 사례별 접근이 필요하다. 기존 시스템이나 높은 수준의 안전성이 요구되는 백본에는 AFDX를 활용하고, 상위 계층 시스템의 모듈성 및 상호운용성 확보를 위해서는 DDS를 적용해야 한다. 또한, 총소유비용(TCO) 절감과 유연성 증대를 위해 미래 아키텍처로서 TSN으로의 전환을 전략적으로 계획해야 한다.



데이터 분산 서비스(DDS)는 객체 관리 그룹(Object Management Group, OMG)이 제정한 미들웨어 표준으로, 발행-구독(Publish-Subscribe) 통신 모델을 구현한다.11 이는 전통적인 클라이언트-서버 모델과 근본적으로 다르다. DDS는 '글로벌 데이터 공간(Global Data Space)'이라는 추상적인 공간을 생성하여, 시스템에 참여하는 애플리케이션(Participant)들이 익명으로 특정 데이터 스트림, 즉 '토픽(Topic)'을 발행(Publish)하거나 구독(Subscribe)할 수 있게 한다.11 이 모델의 핵심은 통신에 참여하는 노드들이 서로의 존재나 위치를 알 필요 없이 오직 데이터 자체에만 집중할 수 있다는 점이다. 발행자는 특정 토픽에 대한 데이터를 생성하여 글로벌 데이터 공간에 게시하고, 미들웨어는 해당 토픽에 관심 있는 모든 구독자에게 데이터를 자동으로 전달한다.11

이 아키텍처의 가장 중요한 특징은 '데이터 중심성(Data-Centricity)'이다. DDS는 단순히 메시지 페이로드를 전달하는 데 그치지 않고, 데이터의 *내용*을 이해한다. 이는 미들웨어가 데이터의 상태 관리, 내용 기반 필터링(Content Filtering), 데이터 이력 관리 등 강력한 기능을 제공할 수 있게 하는 기반이 된다.11 결과적으로 개발자는 복잡한 네트워크 프로그래밍, 소켓 관리, 데이터 직렬화 등의 저수준 작업에서 해방되어 애플리케이션의 핵심 데이터 로직 개발에 집중할 수 있다.13

DDS의 또 다른 핵심적인 강점은 분산형 자동 발견(Decentralized Discovery) 메커니즘이다. 시스템 내 발행자와 구독자는 중앙의 마스터 노드나 브로커 서버 없이도 서로를 동적으로 발견하고 연결을 수립한다.11 이는 시스템의 단일 장애점(Single Point of Failure)을 제거하여 전체 시스템의 내고장성(Fault Tolerance)과 확장성(Scalability)을 크게 향상시킨다. 이러한 특징은 중앙 마스터에 의존했던 ROS 1과 같은 기존 시스템의 한계를 극복하는 결정적인 계기가 되었다.2

DDS의 상호작용은 표준화된 객체 모델을 통해 이루어진다. 이 모델의 핵심 엔티티는 다음과 같다: `DomainParticipant` (특정 도메인 내 애플리케이션의 참여를 표현), `Publisher` (데이터 발행을 책임지는 객체), `DataWriter` (애플리케이션이 특정 토픽의 데이터를 발행하는 인터페이스), `Subscriber` (데이터 수신을 책임지는 객체), `DataReader` (구독한 토픽의 데이터를 애플리케이션이 접근하는 인터페이스), 그리고 `Topic` (데이터의 유형과 이름을 정의)이다.12 이 표준화된 구조는 서로 다른 벤더의 DDS 구현 간에도 상호운용성을 보장하는 기반이 된다.


DDS의 가장 강력하면서도 복잡한 기능은 22개 이상으로 구성된 서비스 품질(QoS) 정책이다.15 이 정책들은 신뢰성, 적시성, 내구성, 리소스 사용량 등과 같은 비기능적 속성을 매우 세밀하게 제어할 수 있게 해준다.11 바로 이 QoS를 통해 DDS는 일반적인 통신 미들웨어를 넘어, 미션 크리티컬하고 실시간성이 요구되는 시스템에 적합하도록 튜닝될 수 있다.3 QoS 정책은 발행자와 구독자 간의 '요청(requested)'과 '제공(offered)' 모델을 기반으로 매칭되며, 호환되는 정책을 가진 엔티티들만 통신이 설정된다.

이러한 강력함은 동시에 큰 도전 과제를 제시한다. 수많은 QoS 정책과 그들 간의 복잡한 상호작용은 상당한 수준의 전문 지식을 요구한다. 잘못된 QoS 설정은 통신 실패, 예측 불가능한 동작, 성능 저하 등 심각한 문제를 야기할 수 있다.3 따라서 DDS의 '즉시 적용 가능성'을 논할 때, 이 QoS의 복잡성은 반드시 고려해야 할 핵심 요소이다. 이는 단순히 기술을 사용하는 것을 넘어, 이를 마스터하기 위한 학습 곡선이 존재함을 의미한다.18 아래 표는 가장 핵심적인 QoS 정책들과 그 전략적 의미를 요약한 것이다.

**표 1: 핵심 DDS QoS 정책 및 전략적 영향**

| QoS 정책명             | 핵심 기능                                                    | 전략적 영향 및 트레이드오프                                  | 대표 사용 사례 (로봇/항공우주)                               |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **RELIABILITY**        | 데이터 전송의 신뢰성 보장 여부를 설정한다. `RELIABLE`은 TCP와 유사하게 수신 확인 및 재전송을 통해 모든 데이터의 전달을 보장한다. `BEST_EFFORT`는 UDP와 같이 확인 절차 없이 데이터를 전송한다.11 | `RELIABLE`은 데이터 손실이 없어야 하는 핵심 명령에 필수적이지만, 지연 시간과 네트워크 오버헤드가 증가한다. `BEST_EFFORT`는 최신 데이터가 더 중요한 고속 센서 데이터 전송에 유리하며, 처리량을 극대화한다.15 | Reliable: 비행 제어 명령, 무기 발사 신호, 로봇 팔 제어 명령  Best-Effort: 비디오 스트림, LiDAR 포인트 클라우드, 비임무용 원격 측정 데이터 |
| **DURABILITY**         | 데이터의 지속성을 제어한다. `VOLATILE`은 현재 구독 중인 구독자에게만 데이터를 전달한다. `TRANSIENT_LOCAL`은 발행자가 데이터를 로컬에 캐시하여, 늦게 참여하는 구독자도 최신 데이터를 수신할 수 있게 한다.11 | `TRANSIENT_LOCAL`은 시스템 상태 동기화 및 장애 복구에 매우 중요하다. 예를 들어, 재부팅된 노드가 즉시 시스템의 현재 상태(예: 목표 위치)를 파악할 수 있게 한다. 하지만 발행자 측에 메모리 리소스 부담을 준다.19 | Transient Local: 시스템 설정값, 현재 임무 목표, 마지막으로 알려진 장애물 위치  Volatile: 일회성 이벤트 알림, 실시간 센서 판독값 |
| **DEADLINE**           | 데이터 업데이트 주기의 최댓값을 명시하는 계약이다. 정해진 시간 내에 새로운 데이터가 수신되지 않으면 미들웨어가 애플리케이션에 이를 통지한다.3 | 제어 루프와 같이 주기적인 데이터 업데이트가 필수적인 시스템의 상태를 감시하는 데 핵심적이다. 데드라인 위반은 특정 컴포넌트의 장애나 네트워크 문제를 즉시 감지할 수 있게 한다.19 | 제어 시스템의 상태 피드백, 심장 박동(heartbeat) 메시지, 안전 시스템의 주기적 상태 보고 |
| **LATENCY_BUDGET**     | 데이터의 긴급성을 미들웨어에 알려, 종단 간(end-to-end) 지연 시간을 최소화하도록 데이터 경로를 최적화하게 한다. 미들웨어는 이 정보를 활용해 리소스 할당이나 전송 우선순위를 조절할 수 있다.19 | 실시간 제어 루프의 성능을 극대화하기 위해 사용된다. 단순히 데이터를 빨리 보내는 것을 넘어, 시스템 전체의 데이터 흐름을 조율하여 가장 중요한 데이터가 가장 먼저 처리되도록 보장한다. | 원격 수술 로봇의 햅틱 피드백, 드론의 자세 제어 명령, 미사일 유도 시스템의 목표 추적 데이터 |
| **HISTORY**            | 구독자에게 전달할 데이터 샘플의 이력을 얼마나 저장할지 결정한다. `KEEP_LAST(n)`은 최근 n개의 샘플만 저장하고, `KEEP_ALL`은 모든 샘플을 저장한다.11 | `KEEP_ALL`은 `RELIABLE` QoS와 함께 사용될 때 완벽한 데이터 전달을 보장하지만, 대량의 메모리를 소모할 수 있다. `KEEP_LAST(n)`은 메모리 사용량과 신뢰성 사이의 현실적인 타협점을 제공한다. | Keep_Last(n): 상태 변화 이력을 추적해야 하지만 모든 과거 데이터가 필요 없는 경우 (예: 최근 10개의 위치 좌표)  Keep_All: 모든 이벤트 로그, 회계 데이터 등 손실이 허용되지 않는 데이터 스트림 |
| **OWNERSHIP_STRENGTH** | 동일한 토픽에 대해 여러 발행자가 존재할 경우, 어떤 발행자의 데이터를 유효한 것으로 간주할지 결정한다. 각 발행자는 `STRENGTH` 값을 가지며, 가장 높은 값을 가진 발행자가 데이터의 '소유권'을 갖는다.14 | 이중화(redundancy) 및 고가용성(high-availability) 시스템 설계에 필수적이다. 주(primary) 발행자에 장애가 발생했을 때, 더 낮은 `STRENGTH`를 가진 예비(backup) 발행자가 자동으로 데이터 발행을 이어받아 시스템 중단을 방지한다. | 주/예비 비행 제어 컴퓨터, 이중화된 센서 시스템, 내고장성 시스템의 리더 선출 |


DDS의 즉시 적용 가능성을 뒷받침하는 강력한 요소 중 하나는 성숙하고 경쟁적인 에코시스템이다. 이 에코시스템은 상용 벤더와 활발한 오픈소스 커뮤니티로 구성되어, 다양한 요구사항과 예산에 맞는 선택지를 제공한다.

상용 벤더:

상용 DDS 벤더들은 고성능, 기술 지원, 그리고 안전 인증과 같은 특정 요구사항을 충족시키는 강력한 솔루션을 제공한다. 주요 벤더는 다음과 같다.

- **RTI (Real-Time Innovations):** DDS 시장의 최대 공급업체로, `RTI Connext DDS` 제품군을 통해 광범위한 산업 분야에 솔루션을 제공한다.21 특히 항공우주 및 국방 분야에서 중요한 

  `Connext DDS Cert` (DO-178C 안전 인증 지원)와 `Connext TSS` (FACE™ 표준 준수) 같은 특화된 제품을 통해 독보적인 위치를 점하고 있다.7 RTI의 솔루션은 높은 신뢰성과 성능을 요구하는 미션 크리티컬 시스템에 널리 사용된다.23

- **ADLINK Technology:** 과거 PrismTech의 `OpenSplice DDS`를 인수하여 `ADLINK DDS` (이전 명칭 Vortex OpenSplice)로 제공하고 있다.14 이 제품은 공유 메모리(Shared Memory) 아키텍처를 통한 뛰어난 확장성과 결정론적 성능으로 잘 알려져 있으며, 대규모 분산 시스템에 강점을 보인다.14 ADLINK는 상용 버전과 함께 오픈소스 커뮤니티 에디션도 제공하여 접근성을 높이고 있다.25

- **eProsima:** `Fast DDS`라는 고성능 오픈소스 DDS 구현체를 개발하여 제공한다. Fast DDS는 특히 로보틱스 커뮤니티에서 폭넓은 지지를 받고 있으며, ROS 2의 장기 지원(LTS) 릴리스에서 기본 미들웨어로 채택되었다.26 이는 eProsima를 로보틱스 분야의 핵심 플레이어로 만들었다. 또한, 자동차 산업의 기능 안전 표준인 ISO 26262 인증을 목표로 하는 

  `Safe DDS`를 개발하는 등 새로운 영역으로 확장하고 있다.28

오픈소스 및 커뮤니티:

eProsima의 Fast DDS나 ADLINK의 Community Edition과 같은 고품질 오픈소스 DDS의 존재는 연구, 프로토타이핑, 비인증 애플리케이션 개발의 진입 장벽을 크게 낮췄다.25 특히 ROS 2 커뮤니티와의 강력한 연계는 방대한 개발자 풀과 풍부한 기술 자료를 제공하여, 문제 해결과 기술 확산에 기여하고 있다.1

이처럼 다양한 벤더와 오픈소스 옵션이 공존하는 성숙한 에코시스템은 DDS가 특정 기업에 종속되지 않는 개방형 표준 기술임을 증명한다. 사용자는 자신의 프로젝트 요구사항, 예를 들어 최고 수준의 안전 인증, 특정 플랫폼과의 통합, 개발 비용, 커뮤니티 지원 등을 종합적으로 고려하여 최적의 DDS 구현체를 전략적으로 선택할 수 있다. 이는 단순한 기술 선택을 넘어, 프로젝트의 전체 생명주기와 장기적인 목표에 부합하는 전략적 결정이 된다.



항공전자 전이중 스위치 이더넷(Avionics Full-Duplex Switched Ethernet, AFDX)은 에어버스(Airbus)가 특허를 보유한 ARINC 664 Part 7 표준의 특정 구현체이다.30 이 기술은 ARINC 429나 MIL-STD-1553B와 같은 기존의 저속, 단방향, 점대점 방식의 항공전자 데이터 버스를 대체하기 위해 개발되었다.5 기존 버스들의 데이터 전송률이 수백 kbit/s에서 최대 2 Mbit/s에 불과했던 반면, AFDX는 100 Mbit/s의 고속 통신을 제공하며 향후 1 Gbit/s로의 확장이 가능하다.5

AFDX의 핵심 설계 철학은 상용 기성품(Commercial-Off-The-Shelf, COTS) 이더넷 기술을 기반으로 개발 비용과 시간을 절감하면서도, 항공전자 시스템의 엄격한 안전성 및 실시간 요구사항을 충족시키기 위해 특별한 프로파일링과 제약을 가하는 것이다.30 즉, 표준 이더넷의 유연성과 경제성을 취하되, 비결정성(non-determinism)과 같은 단점을 보완하여 예측 가능한 통신을 보장하는 것이 목표다.

AFDX 네트워크의 핵심적인 6가지 특성은 다음과 같다: **전이중(Full Duplex)**, **이중화(Redundancy)**, **결정성(Determinism)**, **고속 성능(High-speed performance)**, **스위치 기반(Switched)**, 그리고 **프로파일된 네트워크(Profiled network)**이다.30 네트워크는 지능형 스위치를 중심으로 하는 스타(star) 토폴로지 구조를 가지며, 전이중 통신을 통해 데이터 충돌 가능성을 원천적으로 제거한다.5 이러한 구조는 통합 모듈형 항공전자(Integrated Modular Avionics, IMA) 아키텍처의 근간을 이루며, 항공기 내 배선 무게와 복잡성을 획기적으로 줄이는 데 기여했다.


AFDX가 표준 이더넷과 구별되는 결정론적 성능을 확보할 수 있는 비결은 가상 링크(Virtual Link, VL)와 대역폭 할당 갭(Bandwidth Allocation Gap, BAG)이라는 두 가지 핵심 메커니즘에 있다.

가상 링크 (Virtual Links, VLs):

VL은 AFDX 네트워크의 가장 중심적인 개념이다. 이는 하나의 송신 종단 시스템(source end-system)에서 하나 또는 그 이상의 수신 종단 시스템(destination end-systems)으로 향하는 단방향 논리적 경로를 의미한다.30 표준 이더넷 스위치가 MAC 주소를 기반으로 프레임을 라우팅하는 것과 달리, AFDX 스위치는 프레임 헤더의 특정 필드에 위치한 16비트 VL ID를 사용하여 패킷을 라우팅한다.30 각 VL은 시스템 설계 단계에서 고정된 경로를 할당받으며, 스위치는 이 설정에 따라 정해진 경로로만 패킷을 전달한다. 이는 데이터 흐름을 사전에 완전히 예측하고 제어할 수 있게 하여 네트워크의 결정성을 보장하는 근간이 된다.34

대역폭 할당 갭 (Bandwidth Allocation Gap, BAG):

BAG는 트래픽 셰이핑(traffic shaping)과 대역폭 보장을 위한 핵심 메커니즘이다. BAG는 특정 VL에 할당된 프레임들이 전송될 수 있는 최소 시간 간격을 정의한다 (예: 1 ms, 2 ms, 4 ms,... , 128 ms).30 송신 종단 시스템은 이 BAG 주기에 맞춰 프레임을 전송해야 하며, AFDX 스위치는 각 VL의 트래픽이 BAG 규약을 준수하는지 감시(policing)한다. 만약 어떤 VL이 할당된 BAG보다 빠르게 데이터를 전송하면, 스위치는 해당 초과 패킷을 폐기할 수 있다. 이 메커니즘은 특정 노드가 네트워크를 독점하는 것을 방지하고, 모든 중요 트래픽에 대해 보장된 대역폭과 예측 가능한 최대 지연 시간(latency) 및 지터(jitter)를 제공한다.30

이중화 (Redundancy):

AFDX는 시스템의 신뢰성을 극대화하기 위해 물리적으로 완전히 분리된 이중 네트워크(네트워크 A, 네트워크 B)를 의무적으로 사용한다.30 송신 종단 시스템은 동일한 프레임을 두 네트워크로 동시에 전송한다. 수신 측에서는 먼저 도착하는 유효한 프레임을 수락하고, 나중에 도착하는 중복 프레임은 폐기한다. 이를 통해 케이블 단선, 스위치 고장 등 하나의 네트워크에 장애가 발생하더라도 통신 중단 없이 서비스를 지속할 수 있다.37

이처럼 AFDX는 유연성을 희생하는 대신 VL과 BAG이라는 강력하고 경직된 메커니즘을 통해 항공전자 시스템이 요구하는 엄격한 결정성과 신뢰성을 확보한다. 네트워크의 모든 동작이 설계 단계에서부터 정적으로 정의되고 검증되기 때문에, 운항 중 발생할 수 있는 비정상적인 통신 상황을 원천적으로 차단한다.


AFDX는 표준 이더넷 기술을 기반으로 하지만, 그 결정론적 특성을 구현하기 위해서는 특수한 기능을 갖춘 하드웨어와 소프트웨어 툴이 필요하다. 따라서 AFDX 에코시스템은 일반적인 IT 네트워크 시장보다 훨씬 더 전문화되고 통합된 형태를 띤다.

특수 하드웨어:

AFDX 네트워크를 구성하기 위해서는 VL 기반 라우팅, BAG 트래픽 감시, 이중화 관리 등의 기능을 수행할 수 있는 전용 스위치와 종단 시스템용 네트워크 인터페이스 카드(NIC)가 필수적이다.30 이러한 하드웨어는 단순한 COTS 이더넷 장비가 아니며, 항공전자 환경의 혹독한 요구조건(온도, 진동 등)과 DO-254와 같은 엄격한 안전 표준을 충족하도록 설계 및 인증되어야 한다.

주요 벤더:

AFDX 하드웨어 및 관련 솔루션을 공급하는 시장은 소수의 글로벌 항공전자 전문 기업들이 주도하고 있다.

- **Collins Aerospace (구 Rockwell Collins):** 에어버스 A350, 보잉 787 등 주요 민항기 프로그램의 핵심 공급업체로서, `AFDX-3800`과 같은 AFDX 스위치 및 다양한 항공전자 시스템을 제공한다.30 이들은 항공기 통합 아키텍처 분야에서 깊은 전문성을 보유하고 있다.
- **Thales:** 에어버스 프로그램의 또 다른 핵심 파트너로, 조종석 디스플레이 시스템과 같은 복합적인 항공전자 모듈에 통합되는 AFDX 솔루션을 개발 및 공급한다.40 이들의 제품은 항공기 내 다양한 시스템 간의 데이터 집중 및 분배 역할을 수행한다.
- **TTTech Aerospace:** AFDX 및 이를 확장한 TTEthernet(Time-Triggered Ethernet) 분야의 선도적인 기술 기업이다. TTTech는 스위치 모듈, 종단 시스템용 반도체 IP, 개발 및 구성 툴 등 광범위한 포트폴리오를 제공하며, 에어버스 A380, 보잉 787 등 다수의 항공기에 이들의 기술이 적용되었다.41

개발 및 인증 툴:

AFDX 네트워크의 설계와 인증은 매우 복잡한 과정이다. 시스템 통합자는 네트워크의 종단 간 최악 지연 시간(worst-case end-to-end delay)을 정밀하게 계산하고, 모든 트래픽이 요구사항을 만족함을 증명해야 한다. 이를 위해 네트워크 구성, 시뮬레이션, 성능 분석, 검증을 위한 전문 소프트웨어 툴이 필수적이다.5 이러한 툴들은 VL과 BAG 설정을 최적화하고, 네트워크 부하에 따른 성능 변화를 예측하여 인증 기관에 제출할 객관적인 데이터를 생성하는 데 사용된다.

결론적으로, AFDX의 '즉시 적용 가능성'은 이러한 전문화된 에코시스템에 의해 제한을 받는다. 기술 자체는 성숙했지만, 이를 구현하기 위해서는 고가의 전용 하드웨어, 복잡한 설계 툴, 그리고 DO-254/DO-178C 인증에 대한 깊은 이해가 필요하다. 이는 오픈소스 DDS를 표준 이더넷 하드웨어에서 사용하는 것과는 대조적으로, 상당한 초기 투자 비용과 높은 진입 장벽을 형성하는 요인이다.




AFDX는 더 이상 새로운 실험적 기술이 아닌, 현대 항공전자 시스템의 중추로서 현장에서 그 성능과 신뢰성을 완벽하게 입증받은 기술이다. 이 기술의 성공적인 적용은 현대 항공기 설계의 패러다임을 바꾼 통합 모듈형 항공전자(IMA) 아키텍처의 실현을 가능하게 했다.

대표적인 적용 사례:

AFDX는 세계에서 가장 진보된 상용 항공기들의 데이터 백본으로 채택되었다. 대표적인 사례로는 에어버스 A380, A350, A400M 군용 수송기와 보잉 787 드림라이너가 있다.5 이 항공기들에서 AFDX 네트워크는 비행 제어, 엔진 관리, 항법, 조종석 디스플레이 등 수많은 핵심 서브시스템들을 연결하는 역할을 한다. AFDX의 도입을 통해 기존의 복잡하고 무거운 점대점 배선을 고속의 단일 네트워크로 통합함으로써 항공기 무게, 전력 소비, 그리고 배선 복잡성을 획기적으로 감소시켰다.6 예를 들어, 보잉 787의 경우, AFDX 네트워크는 커먼 코어 시스템(Common Core System, CCS)이라는 중앙 처리 장치와 항공기 내 모든 시스템을 연결하는 '중앙 신경계' 역할을 수행하며, 실시간 원격 진단과 같은 첨단 기능 구현의 기반이 되었다.6

레거시 시스템과의 통합:

현대의 항공기는 AFDX 기반의 신규 시스템과 ARINC 429, MIL-STD-1553B와 같은 기존의 레거시 데이터 버스를 사용하는 시스템이 공존하는 혼합 아키텍처를 가지는 경우가 많다. 이러한 환경에서 원활한 데이터 교환을 위해서는 서로 다른 프로토콜을 변환해주는 게이트웨이 솔루션이 필수적이다. Abaco Systems, Avionics Interface Technologies (AIT), United Electronic Industries (UEI)와 같은 전문 업체들은 이러한 요구에 부응하여 AFDX, ARINC 429, MIL-STD-1553B 등 여러 프로토콜을 동시에 지원하는 고성능 다중 프로토콜 인터페이스 카드 및 분석기를 공급하고 있다.46 이를 통해 시스템 통합 업체는 기존의 검증된 장비들을 유지하면서 새로운 AFDX 기반 시스템과 원활하게 통합할 수 있다.

AFDX의 광범위한 적용과 검증된 실적은 이 기술이 현재 항공우주 산업에서 가장 신뢰할 수 있는 고대역폭, 결정론적 네트워크 솔루션임을 명백히 보여준다. 신규 항공기 개발 프로그램에서 안전 필수 네트워크 백본을 고려할 때, AFDX는 가장 보수적이면서도 확실한 선택지이다.


항공우주 산업은 전통적으로 특정 벤더에 종속되는 폐쇄적인 독점 기술에 의존해왔다. 이는 높은 개발 및 유지보수 비용, 기술 업그레이드의 어려움, 시스템 간 상호운용성 부족 등의 문제로 이어졌다. 이러한 문제를 해결하고 소프트웨어의 재사용성과 이식성을 높이기 위해 미 국방부를 중심으로 미래 항공전자 능력 환경(Future Airborne Capability Environment, FACE™) 컨소시엄과 같은 개방형 표준화 노력이 활발히 진행되고 있다.49

FACE 아키텍처 내 DDS의 역할:

FACE 기술 표준은 여러 계층으로 구성된 참조 아키텍처를 정의하는데, 이 중 전송 서비스 세그먼트(Transport Services Segment, TSS)는 이식 가능한 소프트웨어 컴포넌트(Portable Components Segment, PCS) 간의 모든 데이터 교환을 담당하는 핵심적인 역할을 한다.51 FACE 기술 표준은 바로 이 TSS를 구현하기 위한 핵심 기술 중 하나로 OMG의 DDS를 명시적으로 지정하고 있다.7 이는 DDS의 데이터 중심적, 발행-구독 방식, 그리고 QoS 제어 기능이 현대적인 모듈형 항공전자 소프트웨어 아키텍처의 요구사항과 완벽하게 부합함을 인정한 것이다.

FACE 인증 솔루션:

FACE 표준을 준수하기 위해, DDS 벤더들은 FACE TSS API를 자사의 DDS 구현에 매핑하는 제품을 제공한다. 대표적으로 RTI는 RTI Connext TSS라는 제품을 통해 FACE 인증을 획득했다.7 이 제품은 개발자들이 FACE 표준을 준수하는 애플리케이션을 보다 쉽고 빠르게 개발할 수 있도록 지원하는 상용 기성품(COTS) 솔루션이다. 시스템 통합자는 이러한 인증된 TSS를 사용함으로써, 직접 TSS를 구현하고 인증받는 데 드는 막대한 비용과 시간을 절약하고, FACE 생태계 내 다른 인증된 컴포넌트와의 원활한 상호운용성을 보장받을 수 있다.

결론적으로, DDS는 FACE 표준을 통해 현대 항공전자 소프트웨어 아키텍처의 공식적인 일부로 자리 잡았다. 이는 DDS가 더 이상 지상 시스템이나 비핵심 애플리케이션에만 국한되지 않고, 차세대 항공기의 핵심 소프트웨어 인프라로서 '즉시 적용 가능한' 기술임을 의미한다. 특히 모듈성, 이식성, 재사용성을 강조하는 신규 항공기 개발 프로그램에서 DDS 기반의 FACE TSS를 채택하는 것은 매우 강력하고 전략적인 선택이 될 것이다.


항공우주 시스템, 특히 민항기에 탑재되는 전자 장비의 소프트웨어와 하드웨어는 비행 안전과 직결되기 때문에 세계에서 가장 엄격한 수준의 안전 인증을 통과해야 한다. 이 과정의 핵심에는 RTCA가 제정한 DO-178C (소프트웨어)와 DO-254 (복합 전자 하드웨어) 표준이 있다.54 이 표준들은 개발 프로세스의 모든 단계-계획, 요구사항 정의, 설계, 코딩, 검증, 형상 관리-에 걸쳐 수많은 목표(objective)를 제시하며, 이를 완벽하게 준수했음을 문서화된 증거로 입증해야 한다.

인증 비용과 노력은 시스템의 고장이 항공기에 미치는 영향에 따라 결정되는 설계 보증 수준(Design Assurance Level, DAL)에 따라 기하급수적으로 증가한다. DAL은 A(Catastrophic, 재앙적)부터 E(No Safety Effect, 안전 영향 없음)까지 5단계로 나뉘며, DAL A가 가장 높은 수준이다.56

AFDX 시스템의 인증:

AFDX 표준은 처음부터 DO-254 및 DO-178C 인증을 염두에 두고 개발되었다. AFDX 스위치와 종단 시스템 같은 하드웨어는 DO-254 DAL A 수준으로 개발 및 인증된다. 네트워크 전체의 구성(VL, BAG 설정 등)은 최악 상황 분석(worst-case analysis)을 통해 그 결정론적 동작이 수학적으로 증명된다. 이렇게 검증된 네트워크 위에서 동작하는 애플리케이션 소프트웨어는 DO-178C에 따라 개발되며, 네트워크의 예측 가능한 동작은 소프트웨어의 타이밍 분석을 단순화하는 데 기여한다.

DDS 시스템의 인증:

일반적인 상용 DDS 구현체는 동적 메모리 할당, 풍부한 기능으로 인한 복잡성, 방대한 코드 크기 등의 이유로 DO-178C DAL A와 같은 최고 수준의 안전 인증을 직접 통과하기 어렵다.58 이 문제를 해결하기 위해 DDS 벤더들은 안전 필수 시스템을 위한 특별한 버전의 미들웨어를 제공한다.

- **RTI Connext DDS Cert:** RTI는 자사 제품의 핵심 기능만을 추출하고, 정적 메모리 할당과 같은 안전 필수 설계 원칙을 적용하여 `Connext DDS Cert`라는 인증 가능한 버전을 개발했다.9 가장 중요한 점은, RTI가 제3의 독립 검증 기관(Verification Authority)을 통해 감사받은 완벽한 **DO-178C DAL A 인증 증거 패키지(certification evidence package)**를 함께 제공한다는 것이다.59 이 패키지에는 요구사항 추적 매트릭스, 설계 문서, 테스트 결과 등 인증에 필요한 모든 산출물이 포함되어 있다. 시스템 통합 업체는 이 COTS 인증 증거를 재사용함으로써, 미들웨어 자체를 인증하는 데 드는 수백만 달러의 비용과 수년간의 노력을 절감할 수 있다.10
- **안전 필수 실시간 운영체제(RTOS)와의 결합:** 인증 가능한 DDS 미들웨어는 반드시 DDC-I의 Deos, Wind River의 VxWorks 653 등과 같이 시간 및 공간 분할(time and space partitioning) 기능을 제공하는 DO-178C 인증 RTOS 위에서 동작해야 한다.62 이는 여러 중요도를 가진 애플리케이션들이 동일한 하드웨어에서 서로에게 영향을 주지 않고 독립적으로 실행될 수 있도록 보장하는 IMA 아키텍처의 핵심 요구사항이다.

결론적으로, 항공우주 분야에서 기술의 '즉시 적용 가능성'은 기술적 성숙도뿐만 아니라 '인증 가능성'에 의해 결정된다. AFDX는 그 자체로 인증을 고려한 설계이며, DDS는 `Connext DDS Cert`와 같은 COTS 인증 솔루션의 등장으로 인해 최고 수준의 안전 필수 시스템에도 '즉시 적용 가능한' 기술이 되었다. COTS 인증 증거의 가용성은 이 분야에서 기술 채택을 결정하는 가장 중요한 비즈니스 및 기술적 요인 중 하나이다.


DDS와 결정론적 이더넷 기술의 적용 범위는 전통적인 상용 항공기를 넘어 무인기(UAV), 도심 항공 모빌리티(UAM), 그리고 우주 탐사와 같은 새로운 영역으로 빠르게 확장되고 있다. 이들 분야는 각기 다른 요구사항을 가지지만, 공통적으로 높은 신뢰성, 실시간 데이터 처리, 그리고 모듈형 아키텍처를 필요로 한다.

무인 항공기(UAVs) 및 드론:

UAV 및 드론 시스템, 특히 지상 통제소(Ground Control Station, GCS)와 비행체 간의 통신, 그리고 비행체 내부의 서브시스템 간 데이터 교환에 DDS가 널리 사용되고 있다.63 특히 ROS 2가 UAV 개발에 많이 채택되면서, 그 기반 기술인 DDS가 자연스럽게 핵심 통신 미들웨어로 자리 잡았다.1 DDS의 발행-구독 모델은 LiDAR, IMU, 카메라 등 다양한 센서 데이터를 효율적으로 통합하고, 실시간 제어 명령을 안정적으로 전달하는 데 이상적이다. 한편, 대형 군용 UAV나 높은 수준의 안전성이 요구되는 무인기 시스템에서는 AFDX가 신뢰성 높은 데이터 링크를 보장하기 위해 적용되고 있다.64

우주 시스템(Space Systems):

우주 환경은 극도의 신뢰성과 내방사선(radiation-hardening) 특성을 요구하는 극한의 영역이다. DDS는 이미 NASA의 케네디 우주 센터 발사 통제 시스템(Launch Control System)과 같은 대규모 지상 시스템에 성공적으로 적용되어 수십만 개의 데이터 포인트를 관리하고 있다.66 또한, 위성 간 통신이나 지상국과의 데이터 분배를 위한 아키텍처로서 DDS의 잠재력이 연구된 바 있다.67

더 나아가, AFDX의 개념을 확장한 **TTEthernet (Time-Triggered Ethernet)** 기술이 우주 탐사 분야의 핵심 네트워크 기술로 부상하고 있다. TTTech가 개발한 TTEthernet은 AFDX의 결정론적 특성과 TSN의 시간 동기화 기능을 결합한 기술로, NASA의 유인 달 궤도선 '루나 게이트웨이(Lunar Gateway)'와 유럽의 차세대 발사체 '아리안 6(Ariane 6)'의 핵심 데이터 네트워크로 채택되었다.68 이는 결정론적 이더넷 기술이 지구 대기권을 넘어 우주 공간에서도 그 신뢰성과 성능을 인정받았음을 보여주는 중요한 사례이다. 유럽우주국(ESA) 역시 우주선 내부 데이터 네트워크의 신뢰성 향상과 표준화를 위해 AFDX 기술 도입을 검토한 바 있다.36

이러한 새로운 분야로의 확장은 DDS와 AFDX(및 그 후속 기술)가 특정 영역에 국한된 기술이 아니라, 복잡하고 미션 크리티컬한 분산 시스템을 구축하는 데 있어 근본적인 해결책을 제공하는 범용 기술임을 증명한다.



DDS가 첨단 로보틱스 분야에서 '즉시 적용 가능한' 기술이라고 단언할 수 있는 가장 큰 이유는, 세계에서 가장 널리 사용되는 로봇 개발 프레임워크인 로봇 운영 체제(Robot Operating System, ROS)의 차세대 버전, ROS 2의 핵심 통신 미들웨어로 채택되었기 때문이다.1

ROS 1은 중앙 집중적인 마스터 노드에 의존하고, 실시간성을 보장하지 않으며, 단일 로봇 환경을 가정하여 설계되었기 때문에 다중 로봇 시스템이나 산업용 애플리케이션으로 확장하는 데 명백한 한계가 있었다. 이러한 문제점을 근본적으로 해결하기 위해, ROS 2는 아키텍처를 전면 재설계하면서 통신 계층을 DDS로 교체했다.4

로보틱스에 가져온 이점:

ROS 2와 DDS의 결합은 로보틱스 개발에 다음과 같은 혁신적인 이점을 가져왔다.

- **분산 아키텍처:** DDS의 자동 발견 기능 덕분에 ROS 2 노드들은 중앙 마스터 없이 서로를 직접 발견하고 통신할 수 있다. 이는 다중 로봇 시스템의 구축을 용이하게 하고, 마스터 노드 고장으로 인한 전체 시스템 마비를 방지한다.2
- **실시간 성능:** DDS의 풍부한 QoS 정책을 통해 실시간 제어 루프에 필요한 데이터 전송을 보장할 수 있다. 개발자는 데이터의 특성에 따라 신뢰성, 지연 시간 등을 세밀하게 조정하여 시스템 성능을 최적화할 수 있다.15
- **상호운용성:** DDS는 개방형 표준이므로, 서로 다른 벤더(e.g., eProsima, RTI, ADLINK)의 DDS 구현체를 사용하는 ROS 2 시스템 간에도 원활한 통신이 가능하다. 이는 특정 벤더에 대한 종속성을 줄이고 생태계의 유연성을 높인다.15
- **산업 수준의 신뢰성:** DDS는 원래 국방, 항공 등 미션 크리티컬 시스템을 위해 설계된 기술이므로, 산업 현장에서 요구하는 높은 수준의 신뢰성과 내고장성을 제공한다.

이러한 통합 덕분에, 자율주행차(UGV)가 LiDAR, IMU, 카메라 등 다수의 센서로부터 들어오는 방대한 데이터를 실시간으로 처리하고, 이를 바탕으로 구동계를 제어하는 것과 같은 복잡하고 동적인 로봇 시스템의 개발이 훨씬 용이해졌다.1

벤더 지원:

ROS 2의 장기 지원(LTS) 릴리스는 기본적으로 eProsima의 Fast DDS를 사용하도록 설정되어 있다.27 이는 

`Fast DDS`가 오픈소스이며 우수한 성능을 제공하기 때문이다. 하지만 ROS 2는 미들웨어 계층(RMW)을 추상화하여 설계되었기 때문에, 사용자는 간단한 설정 변경만으로 RTI의 `Connext DDS`나 ADLINK의 `DDS`와 같은 다른 벤더의 구현체로 쉽게 전환할 수 있다. 이는 개발자에게 특정 애플리케이션의 요구사항(예: 초저지연, 안전 인증)에 가장 적합한 DDS 솔루션을 선택할 수 있는 자유를 준다.

결론적으로, DDS는 더 이상 로보틱스를 위한 여러 통신 옵션 중 하나가 아니다. ROS 2의 채택은 DDS를 현대 로보틱스 개발의 **기반(foundation)**으로 만들었다. 따라서 새로운 로봇 프로젝트를 시작하는 팀에게 DDS는 선택의 문제가 아니라, '어떻게 효과적으로 사용할 것인가'의 문제로 귀결된다.


첨단 로보틱스, 특히 자율주행차, 협동 로봇, 드론과 같은 시스템의 핵심은 외부 환경을 인식하고, 판단하며, 신속하게 반응하는 '실시간 제어 루프'의 성능에 달려있다. DDS는 이러한 까다로운 실시간 요구사항을 충족시키기 위한 강력한 기능들을 제공한다.

실시간 제어 루프 지원:

DDS는 상용 구현체에서 마이크로초(µs) 단위의 매우 낮은 지연 시간(latency)과 초당 수백만 메시지를 처리할 수 있는 높은 처리량(throughput)을 제공하도록 설계되었다.24 이는 센서 데이터가 제어기(controller)에 도달하고, 제어 명령이 구동기(actuator)에 전달되기까지의 시간을 최소화해야 하는 로봇 애플리케이션에 필수적이다. 발행-구독 모델은 데이터 생산자와 소비자 간의 결합도를 낮춰, 시스템의 각 부분이 독립적으로 빠르게 동작할 수 있도록 한다.

로보틱스 시나리오를 위한 QoS 활용:

로봇 시스템 내에는 다양한 종류의 데이터가 흐르며, 각 데이터는 서로 다른 통신 요구사항을 갖는다. DDS의 QoS 정책은 이러한 이종(heterogeneous) 데이터 트래픽을 효과적으로 관리하는 데 결정적인 역할을 한다.

- **센서 데이터:** 고해상도 카메라 이미지나 LiDAR 포인트 클라우드와 같은 대용량 센서 데이터는 최신 정보가 가장 중요하다. 오래된 데이터를 재전송하느라 지연을 유발하는 것보다, 약간의 데이터 손실을 감수하더라도 최신 데이터를 빠르게 받는 것이 유리할 수 있다. 이 경우 `RELIABILITY` QoS를 `BEST_EFFORT`로 설정하여 처리량을 극대화한다.15
- **제어 명령:** 모터의 속도나 로봇 팔의 목표 각도와 같은 제어 명령은 단 하나라도 손실되면 시스템의 오작동이나 불안정을 초래할 수 있다. 따라서 이러한 데이터는 반드시 `RELIABILITY` QoS를 `RELIABLE`로 설정하여 100% 전달을 보장해야 한다.16
- **시스템 상태 및 안전 신호:** 로봇의 각 컴포넌트가 정상적으로 동작하고 있는지를 확인하는 '심장 박동(heartbeat)' 메시지나 비상 정지 신호는 주기적으로, 그리고 반드시 전달되어야 한다. `DEADLINE` QoS를 설정하여 특정 시간 내에 메시지가 도착하지 않으면 시스템 장애로 간주하고 안전 조치를 취할 수 있으며, `DURABILITY` QoS를 `TRANSIENT_LOCAL`로 설정하여 늦게 참여한 노드도 현재의 안전 상태를 즉시 알 수 있도록 할 수 있다.

보안(Security):

로봇이 네트워크에 연결되면서 해킹이나 데이터 탈취와 같은 보안 위협이 현실적인 문제로 대두되었다. SROS2(Secure ROS 2)는 DDS 표준의 일부인 DDS-Security 규격을 활용하여 이 문제에 대응한다.1 DDS-Security는 다음과 같은 강력한 보안 기능을 제공한다.

- **인증(Authentication):** 허가된 노드만 DDS 도메인에 참여할 수 있도록 한다.
- **접근 제어(Access Control):** 특정 노드가 특정 토픽을 발행하거나 구독할 수 있는 권한을 세밀하게 제어한다.
- **암호화(Encryption):** 토픽 데이터를 암호화하여 중간에서 데이터를 가로채더라도 내용을 알 수 없게 한다.

이러한 기능들은 DDS가 단순한 데이터 전송 기술을 넘어, 신뢰성과 안전성, 보안성이 모두 요구되는 산업용 및 자율 로봇 시스템을 구축하기 위한 포괄적인 솔루션임을 보여준다.


ROS 2가 DDS의 사용을 상당 부분 추상화하고 단순화했지만, 특정 로봇 애플리케이션에서 최고의 성능을 이끌어내기 위해서는 여전히 DDS QoS와 기본 운영체제(OS) 및 네트워크 매개변수에 대한 세심한 튜닝이 필요하다.18 '동작하는' 시스템을 만드는 것은 비교적 쉽지만, 스트레스 상황에서도 '결정론적으로 최적의 성능을 내는' 시스템을 만드는 것은 전혀 다른 차원의 문제이며, 여기에 DDS 구현의 복잡성이 존재한다.

주요 이슈 및 해결 방안:

로보틱스 개발 현장에서 자주 마주치는 성능 관련 문제와 그 해결 방안은 다음과 같다.

- **대용량 데이터 전송 문제:** 고해상도 카메라 이미지나 3D 포인트 클라우드와 같이 단일 메시지의 크기가 네트워크의 최대 전송 단위(MTU)를 초과하면 데이터는 여러 개의 조각(fragment)으로 나뉘어 전송된다. 이 과정에서 단 하나의 조각이라도 유실되면 전체 메시지가 손실될 수 있다.
  - **해결책:** 리눅스 커널의 수신 버퍼 크기(`net.core.rmem_max`)나 IP 조각 재조립 시간(`net.ipv4.ipfrag_time`)과 같은 OS 레벨의 파라미터를 상향 조정해야 한다. 또한, DDS 벤더가 제공하는 흐름 제어기(flow controller) QoS나 전송 관련 설정을 통해 데이터 전송 속도를 네트워크가 감당할 수 있는 수준으로 조절하는 튜닝이 필요하다.18
- **네트워크 인터페이스 설정:** 개발용 PC나 로봇에는 유선 이더넷, 무선 Wi-Fi 등 여러 네트워크 인터페이스가 동시에 활성화된 경우가 많다. DDS 미들웨어는 기본적으로 사용 가능한 모든 인터페이스를 사용하려 할 수 있으며, 의도치 않게 느리거나 불안정한 인터페이스(예: Wi-Fi)를 통해 중요한 데이터가 전송될 수 있다.
  - **해결책:** DDS의 전송 QoS 정책(`transport QOS`) 내의 `allow_interfaces` 또는 `deny_interfaces` 속성을 사용하여 통신에 사용할 네트워크 인터페이스를 명시적으로 지정해야 한다. 이를 통해 가장 빠르고 안정적인 경로로만 데이터가 흐르도록 보장할 수 있다.70
- **QoS 정책 불일치:** 발행자와 구독자 간에 호환되지 않는 QoS 정책이 설정되면 둘 사이의 통신이 아예 이루어지지 않는다. 예를 들어, 발행자는 `RELIABLE`로 설정했는데 구독자가 `BEST_EFFORT`만 요청하면 연결이 생성되지 않는다. 이는 초보자들이 가장 흔하게 겪는 문제 중 하나다.
  - **해결책:** RTI Monitor나 Admin Console과 같은 DDS 벤더 제공 툴을 사용하여 현재 시스템의 모든 노드와 그들의 QoS 설정을 시각적으로 확인하고 불일치 지점을 찾아내야 한다. 이러한 툴은 NACK(Negative Acknowledgment) 수, 재전송 횟수, 패킷 손실률 등을 모니터링하여 성능 병목 현상의 원인을 진단하는 데에도 필수적이다.70

모범 사례 및 커뮤니티 지원:

이러한 복잡성을 해결하기 위해, DDS 벤더들과 ROS 2 커뮤니티는 방대한 양의 문서, 튜닝 가이드, 그리고 모범 사례(Best Practices) 백서를 제공한다.18 개발자는 이러한 자료를 적극적으로 활용하여 자신의 특정 사용 사례에 맞는 최적의 설정값을 찾아가는 노력을 기울여야 한다.

결론적으로, 로보틱스에서 DDS의 성공적인 적용은 단순히 API를 호출하는 것을 넘어, 시스템 전체의 동작 방식을 이해하고, 성능 병목 지점을 진단하며, 복잡한 QoS 파라미터들을 능숙하게 조율하는 엔지니어링 역량을 요구한다. 이는 프로젝트 계획 수립 시 반드시 고려되어야 할 '숨겨진' 개발 비용이자 리스크이다.




DDS, AFDX, 그리고 TSN을 비교할 때 가장 먼저 명확히 해야 할 점은 이들이 시스템 아키텍처의 서로 다른 계층에서 동작하며, 각기 다른 문제를 해결한다는 것이다. 이들은 상호 배타적인 경쟁 기술이라기보다는 상호 보완적으로 함께 사용될 수 있는 기술이다.

- **역할의 명확화:**
  - **DDS:** OSI 7계층 모델에서 주로 전송 계층(Layer 4)부터 응용 계층(Layer 7)에 걸쳐 동작하는 **미들웨어**이다. DDS의 주된 관심사는 '데이터' 자체의 관리, 즉 데이터 모델 정의, 데이터 배포, QoS 보장, 데이터 중심의 추상화 제공 등이다.72
  - **AFDX 및 TSN:** 주로 데이터 링크 계층(Layer 2)에서 동작하는 **네트워크 기술**이다. 이들의 주된 관심사는 네트워크 상에서 패킷(packet)의 '결정론적 전송'을 보장하는 것, 즉 정해진 시간 내에 예측 가능한 지연과 지터로 패킷을 전달하는 것이다.73

이러한 역할 분담은 두 기술의 통합 가능성을 시사한다. 즉, 애플리케이션은 DDS API를 통해 데이터를 주고받고, DDS 미들웨어는 이 데이터를 RTPS(Real-Time Publish-Subscribe) 프로토콜 패킷으로 변환하여 하부의 네트워크 계층으로 전달한다. 이때 하부 네트워크가 AFDX나 TSN으로 구성되어 있다면, DDS가 생성한 패킷들은 결정론적으로 전송될 수 있다.

- DDS over AFDX:

  항공우주 분야에서는 AFDX의 인증된 결정성과 DDS의 유연한 데이터 중심 모델을 결합하려는 연구와 시도가 활발히 이루어지고 있다.49 이는 FACE 표준과 같이 모듈화된 소프트웨어 아키텍처를 안전 필수 네트워크 위에 구현하기 위한 자연스러운 접근 방식이다.

  - **매핑의 도전 과제:** 그러나 이 통합은 간단하지 않다. 가장 큰 도전은 DDS의 동적이고 토픽 기반의 통신 모델과 다양한 QoS 요구사항을 AFDX의 정적이고 미리 구성된 가상 링크(VL)에 매핑하는 것이다.34 예를 들어, 어떤 DDS 토픽을 어떤 VL에 할당할 것인지, 해당 VL의 BAG(대역폭 할당 갭)을 DDS의 

    `DEADLINE`이나 `LATENCY_BUDGET` QoS를 만족시키도록 어떻게 설정할 것인지에 대한 정교한 설계가 필요하다. 또한, DDS의 자동 발견(discovery)이나 QoS 협상 과정에서 발생하는 내부 메타트래픽(metatraffic) 역시 별도의 VL에 할당하고 대역폭을 보장해주어야 한다.49

- DDS over TSN:

  DDS와 TSN의 결합은 여러 산업 분야에서 "완벽한 조합(perfect combination)" 또는 "자연스러운 궁합(natural fit)"으로 평가받는다.72 TSN은 AFDX보다 더 유연하고 표준화된 방식으로 네트워크 레벨의 결정성을 제공하며, DDS는 그 위에서 강력한 데이터 관리 기능을 제공한다.74 이 아키텍처는 애플리케이션 개발자가 네트워크의 복잡성을 신경 쓸 필요 없이 DDS의 풍부한 QoS를 통해 데이터의 실시간성을 요구하면, TSN 네트워크가 이를 물리적으로 보장해주는 이상적인 그림을 완성한다.

결론적으로, 이들 기술은 서로 다른 계층에서 각자의 역할을 수행하며, 시스템 설계자는 이들을 어떻게 조합하여 전체 시스템의 요구사항을 만족시킬 것인지를 고민해야 한다. AFDX는 경직되지만 검증된 결정성을, TSN은 유연하고 표준화된 결정성을, DDS는 그 위에서 동작하는 데이터 중심의 지능을 제공한다.


시간 민감형 네트워킹(Time-Sensitive Networking, TSN)은 표준 이더넷에 결정론적 통신 기능을 추가하기 위한 IEEE 802.1 표준들의 집합이다. AFDX가 항공전자라는 특정 도메인을 위해 만들어진 단일 규격인 반면, TSN은 자동차, 산업 자동화, 프로 오디오/비디오 등 다양한 산업의 요구를 수용하기 위해 개발된 보다 범용적인 '툴킷(toolkit)'이다.76 이러한 특성으로 인해 TSN은 AFDX의 강력한 차세대 대안으로 부상하고 있다.

TSN의 핵심 메커니즘:

TSN은 여러 표준들의 조합으로 구성되며, 대표적인 기능은 다음과 같다.

- **시간 동기화 (IEEE 802.1AS):** 네트워크에 연결된 모든 장치들이 마이크로초(µs) 이하의 정밀도로 시계를 동기화하도록 한다. 이는 스케줄 기반의 정밀한 데이터 전송을 위한 필수 전제 조건이다. AFDX는 이러한 네트워크 전반의 공통 시간 개념을 기본적으로 제공하지 않는다.77
- **트래픽 셰이핑 (IEEE 802.1Qav, Qbv, Qbu 등):** TSN은 다양한 트래픽 셰이핑 및 스케줄링 메커니즘을 제공한다.
  - **크레딧 기반 셰이퍼 (Credit-Based Shaper, CBS):** 특정 트래픽 클래스에 대해 예약된 대역폭을 보장한다.35
  - **시간 인식 셰이퍼 (Time-Aware Shaper, TAS):** 시간 동기화를 기반으로 각 트래픽 클래스에 대해 특정 시간 슬롯(time slot)을 할당하여 전송 게이트를 열고 닫는다. 이를 통해 매우 엄격한 시간 결정성을 보장할 수 있다.79
- **프레임 선점 (IEEE 802.1Qbu & 802.3br):** 긴 저우선순위 프레임(예: 파일 전송)이 전송되는 도중에 긴급한 고우선순위 프레임(예: 제어 명령)이 도착했을 때, 저우선순위 프레임 전송을 잠시 중단하고 고우선순위 프레임을 먼저 보낸 후, 중단된 지점부터 전송을 재개하는 기능이다. 이는 'Head-of-Line Blocking' 문제를 해결하여 고우선순위 트래픽의 지연 시간을 극적으로 줄여준다.20

TSN 대 AFDX 비교:

TSN은 여러 측면에서 AFDX보다 진보된 특성을 보인다.

- **유연성 및 효율성:** TSN은 여러 종류의 트래픽(실시간 제어, 오디오/비디오, 일반 데이터)을 단일 네트워크에서 효율적으로 처리할 수 있는 혼합 임계 트래픽(mixed-criticality traffic) 지원에 강점을 보인다. 반면 AFDX는 모든 트래픽을 정적인 VL로 처리해야 하므로 유연성이 떨어진다.78
- **비용:** TSN은 표준 이더넷 반도체 기술을 최대한 활용하므로, AFDX의 전용 스위치나 NIC에 비해 하드웨어 비용이 저렴해질 잠재력이 크다.76
- **성능:** TSN의 정밀한 시간 동기화와 프레임 선점 기능은 AFDX보다 더 낮은 지연 시간과 지터(jitter)를 달성할 수 있게 한다. 일부 연구에서는 AFDX의 지터가 수십 ms에 달할 수 있는 반면, TSN은 훨씬 낮은 수준의 지터를 제공할 수 있다고 분석한다.78
- **개방성:** TSN은 IEEE라는 국제 표준화 기구에서 개발한 개방형 표준으로, 특정 기업에 종속되지 않고 폭넓은 벤더 생태계를 가질 수 있다.

이러한 장점들로 인해 TSN은 항공우주 분야에서도 AFDX를 대체하거나 보완할 차세대 네트워크 기술로 진지하게 고려되고 있으며, IEEE는 항공우주 도메인을 위한 TSN 프로파일을 개발 중이다.78


DDS와 TSN의 결합은 각 기술의 장점을 극대화하는 강력한 시너지를 창출하며, 차세대 분산 실시간 시스템의 이상적인 아키텍처로 주목받고 있다. 이 조합은 "완벽한 조합(perfect combination)"으로 불리며, 데이터 중심의 유연한 애플리케이션 개발 환경과 결정론적인 네트워크 전송을 동시에 달성하는 길을 제시한다.72

자연스러운 결합(A Natural Fit):

이 아키텍처의 개념은 명확하다. 애플리케이션 계층에서는 DDS가 데이터 중심의 추상화를 제공하여 개발자가 데이터의 의미와 QoS 요구사항에만 집중할 수 있게 한다. 네트워크 계층에서는 TSN이 DDS 미들웨어가 생성한 RTPS 패킷들의 전송을 물리적으로 보장한다.74 예를 들어, 개발자가 DDS의 

`LATENCY_BUDGET` QoS를 1ms로 설정하면, DDS-TSN 통합 시스템은 해당 데이터 스트림이 TSN 네트워크를 통과하는 데 1ms를 넘지 않도록 TSN의 스케줄러(예: TAS)와 우선순위를 자동으로 구성한다. 이를 통해 소프트웨어 레벨의 요구사항이 하드웨어 레벨의 보증으로 직접 연결된다.

표준화된 통합:

이러한 통합을 체계적으로 지원하기 위해, OMG는 "DDS for TSN (DDS-TSN)"이라는 공식 표준을 발표했다.20 이 표준은 DDS의 QoS 정책을 TSN 네트워크의 구성 파라미터로 매핑하는 규칙과 절차를 정의한다. 개발자는 DDS 애플리케이션을 설계한 후, 배포 설계 단계에서 각 애플리케이션이 요구하는 네트워크 리소스(대역폭, 최대 지연 시간 등)를 명시한다. 그러면 이 정보를 바탕으로 전체 TSN 네트워크(스위치, 엔드포인트)를 설정하는 구성 파일(XML, JSON, YANG 등)이 생성될 수 있다.73 이 표준화된 접근 방식은 시스템 통합의 복잡성을 줄이고, 서로 다른 벤더의 DDS 및 TSN 제품 간의 상호운용성을 보장한다.

산업계의 채택:

DDS-over-TSN 아키텍처는 이미 여러 첨단 산업 분야에서 빠르게 채택되고 있다.

- **자동차:** 차량을 여러 구역(Zone)으로 나누는 조널 아키텍처(Zonal Architecture)와 자율주행 시스템에서 핵심 통신 기술로 부상하고 있다. ADAS 센서 데이터, 제어 신호 등 다양한 종류의 데이터를 단일 이더넷 백본을 통해 안정적으로 전송하는 데 사용된다.75
- **산업 자동화:** 스마트 팩토리에서 수많은 스마트 기기와 기계들을 연결하고, 실시간 제어 데이터와 대용량 모니터링 데이터를 동일한 네트워크에서 처리하기 위한 요구사항을 충족시킨다.75
- **군용 차량:** NATO 표준 군용 차량 아키텍처(NGVA)는 DDS와 TSN을 사용하여 중요한 트래픽에 대한 실시간 QoS를 보장하고 네트워크 구조를 단순화한다.75

이러한 추세는 항공우주 분야에도 영향을 미칠 것이 분명하다. AFDX가 현재의 표준이지만, 장기적으로는 더 유연하고, 비용 효율적이며, 성능이 뛰어난 DDS-over-TSN이 차세대 항공 플랫폼의 표준 아키텍처가 될 가능성이 매우 높다. 따라서 현재 시점에서 AFDX에 대한 투자를 결정할 때, 이러한 기술적 진화의 방향성을 반드시 고려해야 한다.


기술 채택 결정은 단순히 기술적 우수성만으로 이루어지지 않는다. 특히 항공우주나 국방과 같이 수십 년의 생명주기를 갖는 시스템에서는 초기 개발 비용뿐만 아니라 통합, 인증, 유지보수, 업그레이드 등 전체 생명주기에 걸친 총소유비용(Total Cost of Ownership, TCO)이 핵심적인 고려사항이 된다.81


아래 표는 본 보고서에서 분석한 주요 아키텍처들을 핵심적인 평가 지표에 따라 비교하여, 프로젝트의 특성에 맞는 최적의 기술을 선택하는 데 도움을 주기 위해 작성되었다.

**표 2: 로보틱스 및 항공우주를 위한 기술 의사결정 매트릭스**

| 평가 지표                          | DDS on Standard UDP/IP                         | AFDX (독립 네트워크)             | DDS over AFDX                 | DDS over TSN                                              |
| ---------------------------------- | ---------------------------------------------- | -------------------------------- | ----------------------------- | --------------------------------------------------------- |
| **결정성 및 실시간 성능**          | 낮음 (보장 불가)                               | 매우 높음 (정적 보장)            | 매우 높음 (네트워크 보장)     | 매우 높음 (동적 보장)                                     |
| **유연성 및 확장성**               | 매우 높음 (동적, P2P)                          | 매우 낮음 (정적, 경직됨)         | 중간 (데이터 모델 유연)       | 높음 (혼합 임계 지원)                                     |
| **안전 인증 경로 (DO-178C DAL A)** | 매우 어려움 (N/A)                              | 성숙/표준화됨                    | 복잡함 (통합 검증 필요)       | 개발 중 (미래 표준)                                       |
| **에코시스템 및 상호운용성**       | 매우 성숙 (ROS 2, 다수 벤더)                   | 성숙 (소수 전문 벤더)            | 성숙 (각 기술은 성숙)         | 성장 중 (표준화 진행)                                     |
| **총소유비용 (TCO)**               | 낮음 (비인증 시스템)                           | 매우 높음 (NRE, 전용 HW)         | 매우 높음 (두 기술 비용 결합) | 중간 (장기적으로 감소)                                    |
| **최적 적용 분야**                 | **로보틱스**, 비안전 필수 시스템, 프로토타이핑 | **현행 안전 필수 항공전자 백본** | **차세대 FACE 준수 항공전자** | **미래의 모든 실시간 시스템** (자동차, 산업, 차세대 항공) |

이 매트릭스는 '최고의' 기술이란 존재하지 않으며, 오직 '특정 상황에 가장 적합한' 기술만 존재함을 명확히 보여준다. 예를 들어, 빠른 프로토타이핑이 중요한 로보틱스 스타트업은 'DDS on Standard UDP/IP'를 선택할 것이고, A350과 같은 현행 항공기 프로그램을 담당하는 시스템 통합자는 'DDS over AFDX'가 가장 현실적인 경로임을 알 수 있다. 반면, 차세대 군용 차량을 설계하는 팀은 'DDS over TSN'을 전략적 목표로 설정할 것이다.


항공전자 시스템은 항공기 전체 비용의 30% 이상을 차지할 수 있으며 83, 이 비용 구조를 이해하는 것은 매우 중요하다. TCO는 단순히 부품 구매 비용(하드웨어, 소프트웨어 라이선스)을 넘어, 비반복적 엔지니어링 비용(Non-Recurring Engineering, NRE), 통합, 인증, 그리고 수십 년에 걸친 유지보수 비용을 모두 포함한다.81

- **AFDX의 비용 구조:** AFDX의 TCO는 매우 높은 NRE에 의해 주도된다. 여기에는 복잡한 네트워크 설계 및 최악 상황 분석, 고가의 전용 스위치 및 NIC 하드웨어, 그리고 시스템 변경 시마다 발생하는 막대한 재인증 비용이 포함된다.57

- **DDS의 비용 구조:** 상용 DDS는 소프트웨어 라이선스 비용이 발생한다. 더 중요한 비용 요인은 QoS 튜닝과 시스템 통합에 투입되는 상당한 엔지니어링 노력이다.3 만약 안전 인증을 처음부터 직접 수행한다면 NRE는 천문학적으로 증가한다. 하지만 

  `Connext DDS Cert`와 같이 COTS 인증 증거를 활용하면 이 NRE를 극적으로 줄일 수 있다.9 오픈소스 DDS는 초기 라이선스 비용은 없지만, 기술 지원 부재와 통합 과정에서의 숨겨진 비용이 발생할 수 있다.

- **TSN의 비용 절감 효과:** TSN은 AFDX에 비해 TCO를 낮출 잠재력을 가지고 있다. 더 표준화되고 대량 생산되는 이더넷 하드웨어를 활용하고, 혼합 임계 트래픽을 단일 네트워크에 통합하여 배선과 부품 수를 줄일 수 있기 때문이다.76

이 분석을 통해 우리는 중요한 결론에 도달할 수 있다. 항공우주 및 국방 분야에서 TCO를 지배하는 가장 큰 요인은 단위 하드웨어 비용이 아니라, 설계, 통합, 그리고 안전 인증에 소요되는 **NRE**이다. DO-178C/254 인증 프로세스는 그 자체로 막대한 노동력을 요구하며, 전체 소프트웨어 개발 비용을 35%에서 100% 이상 증가시킬 수 있다.57 따라서 부품 명세서(Bill of Materials) 상으로는 저렴해 보이는 아키텍처(예: 표준 이더넷 하드웨어 사용)가 인증 과정을 복잡하게 만든다면, 최종 TCO는 오히려 훨씬 더 비싸질 수 있다. 이것이 바로 AFDX나 COTS 인증 DDS와 같은 솔루션이 높은 초기 비용에도 불구하고, 인증 단계의 리스크와 NRE를 줄여주기 때문에 결과적으로 TCO를 낮출 수 있는 이유이다.

또한, 아키텍처의 **유연성**은 장기적인 TCO에 복리 효과를 가져온다. 항공 시스템의 수명은 수십 년에 달하며, 그 기간 동안 기능 추가나 성능 개선과 같은 업그레이드는 필연적이다. 경직된 AFDX 시스템에서는 새로운 장비(LRU) 하나를 추가하는 것이 전체 네트워크의 재설계와 재검증을 요구하는 거대한 작업이 될 수 있다. 반면, DDS 기반의 분리된(decoupled) 아키텍처에서는 새로운 애플리케이션을 기존 시스템에 거의 영향을 주지 않고 추가할 수 있다.58 TSN 역시 AFDX보다 유연한 스케줄링 메커니즘 덕분에 새로운 데이터 스트림을 추가하기가 더 용이하다.35 따라서 유연한 아키텍처에 대한 초기 투자는 시스템의 생명주기 동안 발생하는 '변경 비용'을 극적으로 줄임으로써 장기적인 TCO 절감으로 이어진다. 이는 AFDX에서 DDS와 TSN으로의 기술 전환을 지지하는 강력한 전략적 논거가 된다.



결론:

DDS는 로보틱스 분야에 단순히 '적용 가능한' 기술을 넘어, ROS 2를 통해 새로운 개발의 표준으로 자리 잡았다. 따라서 기술 채택을 망설일 이유가 없다. 이제 핵심 과제는 기본적인 기능 구현을 넘어, QoS 튜닝과 시스템 최적화에 대한 전문성을 확보하여 견고하고 실시간 성능이 보장되는 로봇을 만드는 것이다.

**로드맵:**

- **즉시 실행 (0-1년):** 모든 신규 로봇 프로젝트에 ROS 2/DDS 아키텍처를 표준으로 채택한다. 개발팀을 대상으로 DDS의 기본 원리, 핵심 QoS 정책, 그리고 성능 분석 도구 사용법에 대한 집중적인 교육에 투자한다.
- **중기 계획 (1-3년):** 자율주행차의 센서 융합, 다관절 로봇의 정밀 제어 등 특정 애플리케이션에 최적화된 사내 표준 QoS 프로파일과 개발 모범 사례를 정립한다. 이를 통해 개발 생산성을 높이고 시행착오를 줄인다.
- **장기 비전 (3년 이상):** 대규모 공장 자동화나 군집 로봇 협력과 같이 공유 네트워크 상에서 엄격한 실시간 보증이 요구되는 애플리케이션을 위해 DDS-over-TSN 아키텍처에 대한 연구개발 및 프로토타이핑을 시작한다.


결론:

항공우주 분야의 해답은 훨씬 더 복합적이다. AFDX는 안전 필수 네트워크 백본으로서 검증된 즉각적인 선택지이다. DDS는 그 네트워크 위에서 동작하는 모듈화되고 이식성 높은 애플리케이션을 구축하기 위한, 특히 FACE 표준 하에서의 즉각적인 선택지이다. 그리고 TSN은 장기적인 비용 절감과 성능 향상을 위해 반드시 준비해야 할 전략적 방향이다.

**로드맵:**

- **즉시 실행 (0-2년):** 최고 수준의 안전 인증(DAL A)이 요구되는 신규 또는 진행 중인 프로젝트의 경우, 네트워크 백본으로 AFDX를 계속 사용한다. 애플리케이션 계층에서는 소프트웨어의 모듈성과 미래 확장성을 극대화하기 위해 RTI의 인증된 TSS와 같은 솔루션을 사용하여 FACE 표준을 적극적으로 채택한다. 기존의 ARINC 429/1553 서브시스템과의 통합을 위해 게이트웨이 솔루션을 활용한다.
- **중기 계획 (2-5년):** 차세대 항공 플랫폼을 위한 DDS-over-TSN 아키텍처의 연구개발 및 프로토타이핑에 착수한다. 이 복잡하고 유연한 시스템의 통합 설계 및 인증에 필요한 툴체인과 엔지니어링 전문성을 확보하는 데 집중한다.
- **장기 비전 (5년 이상):** 신규 플랫폼의 기본 네트워크 아키텍처로 TSN을 단계적으로 도입하고, AFDX는 하위 호환성이 필요한 경우에만 제한적으로 사용한다. 미래의 표준 시스템 아키텍처는 COTS 인증 증거를 적극 활용하여 비용을 관리하는 DDS-over-TSN이 될 것이다.


사용자의 초기 질문인 "DDS와 AFDX는 즉시 적용 가능한 기술인가?"에 대한 최종 답변은 명확한 **'예(YES)'**이다. 두 기술 모두 성숙한 표준이며, 검증된 성능과 확립된 에코시스템을 갖추고 있다.

그러나 시스템 아키텍트에게 더 중요한 통찰은 '적용 가능 여부'가 아니라, **'어떻게, 그리고 어디에 적용할 것인가'**와 **'장기적인 전략적 경로는 무엇인가'**라는 질문에 있다.

본 보고서의 가장 핵심적인 결론은 하나의 기술에만 머무르는 것이 전략적 오류라는 점이다. 로보틱스와 항공우주 분야 모두에서 복잡한 실시간 시스템의 미래는 다계층, 하이브리드 아키텍처에 있다. 이 미래에서 성공하기 위한 열쇠는 데이터 중심 미들웨어 계층(DDS)을 완벽하게 마스터하고, 그 아래의 결정론적 네트워크 계층에 대해 현명하고 미래지향적인 선택(현재는 AFDX, 미래에는 TSN)을 하는 능력에 달려있다.


1. ROS2 DDS 미들웨어의 통신 오버헤드 실험 분석, accessed July 13, 2025, https://www.ki-it.com/xml/39841/39841.pdf
2. ROS on DDS - ROS2 Design, accessed July 13, 2025, https://design.ros2.org/articles/ros_on_dds.html
3. Ethernet 기반 차량 네트워크 설계에서 Functional Safety (ISO 26262) 고려사항 (SOME/IP & DDS) - 내 머리속 지우개 - 티스토리, accessed July 13, 2025, [https://habana4.tistory.com/entry/Ethernet-%EC%B0%A8%EB%9F%89-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%84%A4%EA%B3%84-Functional-Safety-ISO26262-%EA%B3%A0%EB%A0%A4%EC%82%AC%ED%95%AD-SOME-IP-DDS](https://habana4.tistory.com/entry/Ethernet-차량-네트워크-설계-Functional-Safety-ISO26262-고려사항-SOME-IP-DDS)
4. ROS1과 ROS2의 구조적 차이와 자율주행 시스템 적용 관점에서의 비교 분석 - NMLab, accessed July 13, 2025, https://nmlab.korea.ac.kr/publication/published.papers/2025/2025.04_Comparison_of_ROS1_and_ROS2-KNOM2025.pdf
5. AFDX Tap과 AFDX 프로토콜 분석기를 이용한 AFDX 네트워크 인증 ..., accessed July 13, 2025, https://koreascience.kr/article/JAKO202209858365235.pdf
6. 항공 전자 시스템의 통합 아키텍처 ✈️ - 재능넷, accessed July 13, 2025, https://www.jaenung.net/tree/20376
7. RTI Connext TSS, accessed July 13, 2025, https://www.rti.com/hubfs/docs/RTI_FACE_TSS.pdf
8. FACE and DDS | Twin Oaks Computing, Inc, accessed July 13, 2025, https://www.twinoakscomputing.com/coredx/future_airborne_capability_environment
9. Connext DDS Cert - Safety Certifiable Communications Infrastructure - Mistral Solutions, accessed July 13, 2025, https://www.mistralsolutions.com/defence_products/connext-dds-cert/
10. RTI Connext DDS Cert - Safety-Certifiable Connectivity Platform - Dedicated Systems, accessed July 13, 2025, https://dedicatedsystems.com.au/products/connext-dds-cert/
11. DDS(Data Distribution Service) 이해하기 - Hoon's Blog - 티스토리, accessed July 13, 2025, https://yhoons.tistory.com/117
12. 데이터 분산 서비스 - 위키백과, 우리 모두의 백과사전, accessed July 13, 2025, [https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EB%B6%84%EC%82%B0_%EC%84%9C%EB%B9%84%EC%8A%A4](https://ko.wikipedia.org/wiki/데이터_분산_서비스)
13. [ROS2]OMG Data-Distribution Service: Architectural Overview and OMG Data-Distribution Service: Architectural Update - velog, accessed July 13, 2025, https://velog.io/@protocol5/ROS2OMG-Data-Distribution-Service-Architectural-Overview-and-OMG-Data-Distribution-Service-Architectural-Update
14. DDS(Data Distribution Service)란 무엇인가? - Perbiz, accessed July 13, 2025, [http://perbiz.co.kr/data/DDS(Data%20Distribution%20Service)%EB%9E%80.pdf](http://perbiz.co.kr/data/DDS(Data Distribution Service)란.pdf)
15. ROS2와 DDS란? - 개발하는 핑구 - 티스토리, accessed July 13, 2025, [https://ai-sinq.tistory.com/entry/ROS2%EC%99%80-DDS%EB%9E%80](https://ai-sinq.tistory.com/entry/ROS2와-DDS란)
16. Summary and typical use cases for different Quality of Service (QoS) parameters, accessed July 13, 2025, https://community.rti.com/kb/summary-and-typical-use-cases-different-quality-service-qos-parameters
17. Modeling the QoS parameters of DDS for event-driven real-time applications | Request PDF, accessed July 13, 2025, https://www.researchgate.net/publication/274072933_Modeling_the_QoS_parameters_of_DDS_for_event-driven_real-time_applications
18. DDS tuning information - ROS 2 Documentation: Foxy documentation, accessed July 13, 2025, https://docs.ros.org/en/foxy/How-To-Guides/DDS-tuning.html
19. Data-Centric Programming Best Practices: - RTI, accessed July 13, 2025, https://www.rti.com/hubfs/docs/DDS_Best_Practices_WP.pdf
20. DDS over TSN: configuring TSN to meet DDS-level QoSes - RealTime-at-Work (RTaW), accessed July 13, 2025, https://www.realtimeatwork.com/wp-content/uploads/AEC24_RTI_RTaW_DDS-over-TSN.pdf
21. RTI | Distributed, physical AI systems work together as one, accessed July 13, 2025, https://www.rti.com/en/
22. RTI Connext® | Software Connectivity Framework - Surgical Robotics Technology, accessed July 13, 2025, https://www.surgicalroboticstechnology.com/company/real-time-innovations-rti/rti-connext/
23. Real-Time Innovations (RTI) | Ansys Technology Partner, accessed July 13, 2025, https://www.ansys.com/partner-ecosystem/technology-partners/real-time-innovations
24. Data Distribution Service | IoT Applications - ADLINK Technology, accessed July 13, 2025, https://www.adlinktech.com/en/data-distribution-service
25. ADLINK Tech | Data Distribution Service DDS Community, accessed July 13, 2025, https://www.adlinktech.com/en/data-distribution-service-dds-community
26. Fast DDS - eProsima, accessed July 13, 2025, https://fast-dds.docs.eprosima.com/
27. eProsima/Fast-DDS: The most complete DDS - Proven: Plenty of success cases. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 13, 2025, https://github.com/eProsima/Fast-DDS
28. Services - eProsima - FIWARE, accessed July 13, 2025, https://www.fiware.org/showcase/product-details/?category=services&id=eprosima-eprosima
29. Safe DDS, "certifiable" DDS (and a fork of embeddedRTPS?) - General - ROS Discourse, accessed July 13, 2025, https://discourse.ros.org/t/safe-dds-certifiable-dds-and-a-fork-of-embeddedrtps/29636
30. Avionics Full-Duplex Switched Ethernet - Wikipedia, accessed July 13, 2025, https://en.wikipedia.org/wiki/Avionics_Full-Duplex_Switched_Ethernet
31. avionics full duplex switched ethernet (afdx) data bus - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/profile/Muhammet-Yanik-2/publication/263619917_Avionics_full_duplex_switched_ethernet_AFDX_data_bus/links/0deec53b6478ab9ad0000000/Avionics-full-duplex-switched-ethernet-AFDX-data-bus.pdf
32. AFDX Tap과 AFDX 프로토콜 분석기를 이용한 AFDX 네트워크 인증 기술 - Korea Science, accessed July 13, 2025, https://koreascience.kr/article/JAKO202209858365235.pub?&lang=ko&orgId=sase
33. Implementation of the hardwired AFDX NIC, accessed July 13, 2025, http://fif.kr/AsiaNetFPGAws/paper/1-3.pdf
34. On the integration of DDS and AFDX standards | Request PDF - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/384588980_On_the_integration_of_DDS_and_AFDX_standards
35. Impact Analysis of Flow Shaping in Ethernet-AVB/TSN and AFDX from Network Calculus and Simulation Perspective, accessed July 13, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC5470926/
36. Methodology and assessment for the applicability of ARINC-664 (AFDX) in Satellite/Spacecraft on-board communicatION networks | FP7 - CORDIS, accessed July 13, 2025, https://cordis.europa.eu/project/id/312746/reporting
37. Scheduling Design and Analysis for End-to-End Heterogeneous Flows in an Avionics Network - UNL Digital Commons, accessed July 13, 2025, https://digitalcommons.unl.edu/cgi/viewcontent.cgi?article=1221&context=cseconfwork
38. Collins Aerospace AFDX-3800 Ethernet Switch Unit - Part Number: 822-1745-101, Unit Condition: New | AIR TEAM - Airteam.eu, accessed July 13, 2025, https://www.airteam.eu/p/collins-aerospace-afdx-3800-1
39. Collins Aerospace AFDX-3800 | SEAEROSPACE.COM, accessed July 13, 2025, [https://www.seaerospace.com/sales/product/Rockwell%20Collins/AFDX-3800](https://www.seaerospace.com/sales/product/Rockwell Collins/AFDX-3800)
40. Avionics | Diehl Aviation, accessed July 13, 2025, https://www.diehl.com/aviation/en/portfolio/avionics/
41. Switches - TTTECH, accessed July 13, 2025, https://www.tttech.com/products/products/aerospace/flight-rugged-hardware/switches
42. About us - Aerospace - TTTech, accessed July 13, 2025, https://www.tttech.com/aerospace/about-us
43. About | TTTech - Aviation Week Marketplace, accessed July 13, 2025, https://marketplace.aviationweek.com/suppliers/tttech/
44. A Reality-Conforming Approach for QoS Performance Analysis of AFDX in Cyber-Physical Avionics Systems - NSF-PAR, accessed July 13, 2025, https://par.nsf.gov/servlets/purl/10297125
45. AFDX Emulator for an ARINC-Based Training Platform | Request PDF - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/278031280_AFDX_Emulator_for_an_ARINC-Based_Training_Platform
46. Understanding 1553 to ARINC-429 Conversion - KIMDU Technologies, accessed July 13, 2025, https://kimdu.com/understanding-1553-to-arinc-429-conversion/
47. Avionics: MIL-STD-1553, ARINC 429 & 664, AFDX | Abaco Systems, accessed July 13, 2025, https://www.abaco.com/avionics
48. Combo MIL-STD-1553/ARINC 429 Instruments - Avionics Interface ..., accessed July 13, 2025, https://aviftech.com/combo-1553-429/
49. On the integration of DDS and AFDX standards - RTI Community, accessed July 13, 2025, https://d2vkrkwbbxbylk.cloudfront.net/sites/default/files/integrationddsafdx.pdf
50. On the Integration of DDS and AFDX standards - RTI Community, accessed July 13, 2025, https://community.rti.com/paper/integration-dds-and-afdx-standards
51. The FACE Transport Services Segment (TSS): Where Data Blossoms - RTI, accessed July 13, 2025, https://www.rti.com/blog/face-transport-services-segment-tss
52. RTI Connext TSS: Certified Conformant to FACE Technical Standard, Edition 3.1, accessed July 13, 2025, https://www.rti.com/blog/rti-connext-tss-certified-conformant
53. RTI Announces First Transport Services Segment with FACE 3.1 Conformance Certification and DO-178C DAL A Safety Evidence - GlobeNewswire, accessed July 13, 2025, https://www.globenewswire.com/news-release/2023/03/14/2626594/0/en/RTI-Announces-First-Transport-Services-Segment-with-FACE-3-1-Conformance-Certification-and-DO-178C-DAL-A-Safety-Evidence.html
54. What Is DO-178C? - Wind River Systems, accessed July 13, 2025, https://www.windriver.com/solutions/learning/do-178c
55. The Basics of DO-178C, DO-254, and AMC 20-152A | PTC, accessed July 13, 2025, https://www.ptc.com/en/resources/application-lifecycle-management/report/basics-do-178c-do-254-amc-20-152a-compliance
56. DO-178C and DO-254 Glossary - Airworthiness Certification Services, accessed July 13, 2025, https://airworthinesscert.com/do-178c-and-do-254-glossary/
57. DO-178C Costs vs Benefits Analysis - AFuzion, accessed July 13, 2025, https://afuzion.com/do-178c-costs-versus-benefits/
58. Saving Millions of Dollars in the Development and Certification of Safety-Critical Applications - Mistral Solutions, accessed July 13, 2025, https://www.mistralsolutions.com/newsletter/Jul14/Save_on_Safety_Cert.pdf
59. Connext Cert | Certification Software for Safety Critical Systems, accessed July 13, 2025, https://www.rti.com/products/connext-cert
60. RTI Connext DDS CERT - Intro to Safety Certified DDS - RTI Community, accessed July 13, 2025, https://community.rti.com/video/rti-connext-dds-cert-intro-safety-certified-dds
61. RTI Connext DDS Micro Provides FACETM Conformant, accessed July 13, 2025, https://www.cotsjournalonline.com/rti-connext-dds-micro-provides-facetm-conformant-communications-framework-for-ddc-is-deos-avionics-real-time-operating-system/
62. Deos RTOS Software | Safety Critical Multicore RTOS | DAL A, DO-178C, FACE - DDC-I, accessed July 13, 2025, https://www.ddci.com/solutions/products/deos/
63. Analysis of Design Directions for Ground Control Station (GCS), accessed July 13, 2025, https://www.scirp.org/journal/paperinformation?paperid=72306
64. Buses and Networks for Contemporary Avionics - Data Device Corporation, accessed July 13, 2025, https://www.ddc-web.com/resources/FileManager/dbi/Whitepapers/Buses_and_NetworksWP.pdf
65. A Framework for IMA-based Architecture Design with Unmanned Aerial System (UAS) as a Test Case - CEUR-WS.org, accessed July 13, 2025, https://ceur-ws.org/Vol-2814/paper-A4-1.pdf
66. Who's Using DDS? - DDS Foundation, accessed July 13, 2025, https://www.dds-foundation.org/who-is-using-dds-2/
67. Data distribution satellite - NASA Technical Reports Server (NTRS), accessed July 13, 2025, https://ntrs.nasa.gov/citations/19930003293
68. TTEthernet - Wikipedia, accessed July 13, 2025, https://en.wikipedia.org/wiki/TTEthernet
69. Space | TTTECH, accessed July 13, 2025, https://www.tttech.com/tags/space
70. Three Simple Steps to Achieving Peak DDS Performance | RTI, accessed July 13, 2025, https://www.rti.com/blog/three-simple-steps-to-achieving-peak-dds-performance/
71. Repeat Success, Not Mistakes; - Real-Time Innovations - RTI, accessed July 13, 2025, https://info.rti.com/hubfs/whitepapers/Best_Practices_Repeat_Success.pdf
72. 시간 민감형 네트워크 시리즈 | 해홍 주식회사 | TSN, accessed July 13, 2025, https://haehongtec.com/time-sensitive-network/
73. TSN and DDS: Enhancing Real-Time Communication - Pavel Pohanka, accessed July 13, 2025, https://pavelpohanka.cz/en/blog/tsn-and-dds-enhancing-real-time-communication/
74. Ensuring Critical Priority Data Delivery with Deterministic Time Sensitive Networking - Aerospace DAQ, Test, HIL - UEI - United Electronic Industries, accessed July 13, 2025, https://www.ueidaq.com/blog/ensuring-critical-priority-data-delivery-with-deterministic-time-sensitive-networking
75. DDS and TSN: Converging Two Communications Standards - RTI, accessed July 13, 2025, https://www.rti.com/dds-and-tsn
76. It's time: Time Sensitive Networking and Deterministic Networking - Military Embedded Systems, accessed July 13, 2025, https://militaryembedded.com/comms/communications/its-time-time-sensitive-networking-and-deterministic-networking
77. Power of TSN and RTOS integration for aerospace - Industrial Ethernet Book, accessed July 13, 2025, https://iebmedia.com/technology/tsn/power-of-tsn-and-rtos-integration-for-aerospace/
78. Deployment of a Testbed for Validation of TSN Networks in Avionics - MDPI, accessed July 13, 2025, https://www.mdpi.com/2226-4310/12/3/186
79. Incorporating TSN/BLS in AFDX for Mixed-Criticality Avionics Applications: Specification and Analysis - arXiv, accessed July 13, 2025, https://arxiv.org/pdf/1707.05538
80. Streamlining Zonal Architecture with DDS and TSN - RTI, accessed July 13, 2025, https://www.rti.com/blog/streamlining-zonal-architecture-with-dds-and-tsn
81. Obsolescence and Life Cycle Management for Avionics - FAA, accessed July 13, 2025, https://www.faa.gov/sites/faa.gov/files/aircraft/air_cert/design_approvals/air_software/TC-15-33.pdf
82. Through-Life Maintenance Cost of Digital Avionics - MDPI, accessed July 13, 2025, https://www.mdpi.com/2076-3417/11/2/715
83. INTRODUCTION TO AVIONICS - ACS College of Engineering, accessed July 13, 2025, https://www.acsce.edu.in/acsce/wp-content/uploads/2020/03/Avionics-Vol_1.pdf


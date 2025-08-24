# 시간 민감형 네트워킹(TSN) 종합
[모빌리티를 위한 통신](./index.md)


Industry 4.0, 산업용 사물 인터넷(IIoT), 자율주행차와 같은 차세대 기술 패러다임은 데이터 통신 인프라에 근본적인 변화를 요구하고 있다. 과거 분리되어 운영되던 정보 기술(IT)과 운영 기술(OT) 네트워크의 융합이 필수불가결해지면서, 표준 이더넷 기술의 한계를 극복하고 다양한 종류의 트래픽을 단일 네트워크에서 안정적으로 처리할 수 있는 새로운 기술의 필요성이 대두되었다. 시간 민감형 네트워킹(Time-Sensitive Networking, TSN)은 이러한 시대적 요구에 부응하기 위해 등장한 IEEE 표준 기반의 기술군으로, 표준 이더넷에 결정론적(Deterministic) 특성을 부여하여 미래 산업의 신경망을 구축하는 핵심 기술로 주목받고 있다.


표준 이더넷은 지난 수십 년간 사무 환경과 데이터 센터에서 성공적으로 사용되어 온 검증된 기술이지만, 그 태생적 한계로 인해 실시간 제어가 필수적인 산업 현장 적용에는 부적합했다. 저렴한 설치 비용과 간단한 네트워크 구조라는 장점에도 불구하고, 표준 이더넷의 통신 방식은 예측 불가능성을 내포하고 있다.1

가장 근본적인 한계는 '비결정성(Non-Determinism)'이다. 초기 이더넷이 사용했던 CSMA/CD(Carrier Sense Multiple Access with Collision Detection) 방식은 여러 장치가 동시에 데이터를 전송하려 할 때 충돌이 발생하고, 이를 해결하기 위해 무작위 시간 동안 대기 후 재전송하는 메커니즘을 사용한다.2 이러한 무작위 대기 시간은 데이터가 언제 목적지에 도착할지 예측할 수 없게 만든다.3 스위치(Switch)의 도입으로 충돌 도메인이 분리되어 이 문제는 상당 부분 완화되었지만, 또 다른 형태의 비결정성이 나타났다. 바로 스위치 내부에서 발생하는 지연 및 지터(Jitter) 문제다. 대부분의 이더넷 스위치는 '저장 후 전달(Store-and-Forward)' 방식으로 동작하는데, 이는 수신한 데이터 프레임 전체를 버퍼에 저장한 후 오류를 검사하고 목적지로 전달하는 방식이다.4 이 과정에서 프레임의 길이에 따라 처리 시간이 달라져 전송 지연 시간이 변동하는 '지터'가 발생한다. 예를 들어, 100Mbps 네트워크에서 최대 크기(1522바이트)의 이더넷 프레임 하나가 전송 경로를 점유하면, 뒤따르는 긴급 데이터는 최대 124ms까지 지연될 수 있다.5 또한, 특정 포트로 트래픽이 몰릴 경우 스위치의 버퍼 메모리가 가득 차 새로운 프레임을 폐기하는 상황도 발생할 수 있다.5

이러한 표준 이더넷의 '최선 노력(Best-Effort)' 방식은 일반적인 데이터 전송에는 충분하지만, 수 마이크로초(µs) 단위의 정밀한 제어가 요구되는 로봇 제어나 안전 시스템에서는 치명적인 결과를 초래할 수 있다. 단순히 네트워크 속도를 기가비트급으로 높이는 것만으로는 이러한 근본적인 문제를 해결할 수 없으며, 모든 예외 상황에서도 정해진 시간 내에 데이터 전송을 보장하는 '하드 실시간(Hard Real-Time)' 성능을 확보하기 위해서는 특수한 메커니즘이 필요하다.5


이러한 표준 이더넷의 한계를 극복하기 위해 산업 현장에서는 지난 수십 년간 CAN, PROFIBUS, EtherCAT, POWERLINK 등 다양한 목적의 산업용 필드버스(Fieldbus) 프로토콜이 사용되어 왔다.5 이들 프로토콜은 각자의 방식으로 실시간 통신을 보장하며 특정 분야에서 높은 성능을 발휘했지만, 이는 또 다른 문제를 야기했다. 바로 기술의 '파편화(Fragmentation)'다. 각기 다른 필드버스는 서로 호환되지 않아, 공장 내에 여러 종류의 네트워크가 공존하는 복잡한 구조를 만들었다. 이는 네트워크 접속 장치 수의 제한에 따른 확장성 문제, 상이한 기술 간 상호운용성 부재, 높은 유지보수 비용으로 이어졌다.6

Industry 4.0 시대에 접어들면서 이러한 파편화된 구조는 더 이상 지속 가능하지 않게 되었다. 스마트 팩토리는 생산 라인의 제어 데이터뿐만 아니라, 품질 검사를 위한 고해상도 비전 데이터, 설비 예지 보전을 위한 센서 데이터, 상위 MES/ERP 시스템과의 연동을 위한 IT 데이터 등 막대한 양의 다양한 데이터를 요구한다.7 자율주행차 역시 ADAS(첨단 운전자 보조 시스템)를 위한 카메라, 레이더, 라이다 데이터와 차량 제어 신호, 인포테인먼트 스트리밍 데이터가 하나의 네트워크에서 처리되어야 한다.6 이처럼 다양한 종류와 우선순위를 가진 트래픽을 단일 네트워크 인프라에서 효율적으로 처리하기 위한 '통합 네트워크(Converged Network)'의 필요성이 절실해졌다.

TSN은 바로 이 지점에서 탄생했다. TSN은 특정 기업이나 컨소시엄이 주도하는 사설 프로토콜이 아니라, 국제 표준화 기구인 IEEE 802 위원회에서 정의하는 개방형 표준 기술군이다.4 그 목표는 표준 이더넷의 개방성과 확장성, 비용 효율성을 유지하면서, 시간 동기화, 트래픽 스케줄링, 신뢰성 보장 메커니즘을 추가하여 '확정적 메시지 전송'을 구현하는 것이다.10 즉, TSN은 산업 현장을 지배하던 여러 독점적인 프로토콜들을 'TSN'이라는 거대한 표준 우산 아래로 통합하여, OT와 IT 데이터가 하나의 네트워크에서 원활하게 공존할 수 있는 기반을 마련하는 거대한 기술 표준화 프로젝트라 할 수 있다.10

이는 단순히 기존 기술을 대체하는 것을 넘어, 네트워크 패러다임 자체를 전환하는 시도다. TSN의 등장은 '비용'과 '성능' 사이의 단순한 트레이드오프를 넘어, '융합'과 '확장성'이라는 시대적 요구에 대한 필연적인 기술적 진화의 결과물이다. TSN은 IT와 OT의 완전한 융합을 위한 물리적 토대를 제공함으로써, 데이터 기반의 스마트 제조와 자율 모빌리티 혁신을 가능하게 하는 핵심 기반 기술(Enabling Technology)로 자리매김하고 있다.


TSN은 단일 표준이 아닌, 특정 기능을 수행하는 여러 IEEE 802.1 표준들의 집합체, 즉 '기술의 바구니(basket of technologies)' 또는 '툴박스(toolbox)'로 이해해야 한다.11 이 표준들은 마치 레고 블록처럼 상호 보완적으로 결합하여, 애플리케이션이 요구하는 수준의 실시간성, 신뢰성, 대역폭을 제공한다. TSN 네트워크를 구성하는 핵심 기술 요소는 크게 시간 동기화, 트래픽 스케줄링, 신뢰성 보장, 네트워크 관리의 네 가지 범주로 나눌 수 있다.12


TSN의 모든 결정론적 기능은 네트워크에 참여하는 모든 장치들이 하나의 기준 시간을 공유한다는 전제 위에서 동작한다. IEEE 802.1AS 표준은 바로 이 '시간 동기화'를 담당하는 핵심 기술이다.9 이 표준은 gPTP(generalized Precision Time Protocol)라는 프로파일을 정의하는데, 이는 널리 사용되는 IEEE 1588 PTP를 TSN 환경에 맞게 최적화한 것이다.12

gPTP의 작동 원리는 다음과 같다. 네트워크 내에서 가장 정밀한 클럭을 가진 장치가 '그랜드마스터(Grandmaster)'로 자동 선출된다.13 그랜드마스터는 자신의 시간 정보를 담은 동기화 메시지를 주기적으로 네트워크에 전파한다. 이 메시지를 수신한 다른 장치들은 자신의 클럭을 그랜드마스터의 시간에 맞춘다. 하지만 단순히 메시지 수신 시간만으로는 정밀한 동기화가 어렵다. 메시지가 네트워크 스위치를 거치면서 발생하는 처리 지연(processing delay)과 큐잉 지연(queuing delay) 때문이다. 802.1AS는 '투명 클럭(Transparent Clock)' 개념을 도입하여 이 문제를 해결한다. 스위치와 같은 중간 노드들은 동기화 메시지가 자신을 통과하는 데 걸린 시간, 즉 '체류 시간(residence time)'을 측정하여 메시지에 기록한다.12 최종적으로 메시지를 수신한 엔드포인트는 이 체류 시간 정보를 이용해 전파 과정에서 발생한 지연 오차를 보상함으로써, 모든 장치가 1 마이크로초(μs) 이하의 오차 범위 내에서 매우 정밀하게 동기화될 수 있다.13

이처럼 정밀하게 동기화된 시간은 모든 TSN 장치가 동일한 시간축 위에서 동작하게 만들어, 후술할 시간 기반 트래픽 스케줄링(TAS)을 가능하게 하는 가장 근본적인 전제 조건이 된다.


시간 동기화가 완료되면, TSN은 이 시간 정보를 기반으로 다양한 트래픽의 전송 시점을 제어하고 흐름을 조절한다. 이를 위해 여러 스케줄링 및 셰이핑 표준이 사용된다.


TAS는 TSN에서 가장 강력한 결정론적 통신을 보장하는 핵심 메커니즘이다.9 그 원리는 TDMA(Time-Division Multiple Access, 시분할 다중 접속) 방식을 이더넷 스위치에 도입하는 것이다.14 네트워크 관리자는 전체 네트워크의 시간을 잘게 쪼개어 '시간 슬롯(Time Slot)'을 만들고, 각 슬롯을 특정 종류의 트래픽에 독점적으로 할당한다.

기술적으로 이는 스위치 포트의 각 우선순위 큐 앞에 '전송 게이트(Transmission Gate)'를 설치하고, '게이트 제어 목록(Gate Control List, GCL)'이라는 스케줄 표에 따라 이 게이트들을 주기적으로 열고 닫는 방식으로 구현된다.9 예를 들어, 1ms 주기의 사이클에서 첫 100µs는 최우선순위의 실시간 제어 트래픽(예: 로봇 팔 제어)을 위해 할당하고, 나머지 900µs는 비디오 스트리밍이나 일반 데이터 전송을 위해 할당할 수 있다.14 이렇게 하면 실시간 제어 트래픽은 다른 어떤 트래픽의 방해도 받지 않고 정해진 시간에 정확히 전송될 수 있어, 예측 가능한 초저지연 통신이 가능해진다.

하지만 TAS에는 단점도 존재한다. 특정 시간 슬롯이 닫히기 직전에 전송을 시작하면 프레임이 잘릴 수 있기 때문에, 이를 방지하기 위해 슬롯 끝에 '가드 밴드(Guard Band)'라는 보호 구간을 두어야 한다.7 이 가드 밴드 동안에는 어떤 데이터도 전송할 수 없어 대역폭 낭비가 발생한다.7


프레임 선점은 TAS의 단점을 보완하고 네트워크 효율성을 높이기 위한 기술이다. 이는 긴급하지 않은 큰 크기의 프레임(Expressible Frame) 전송 중에 긴급하고 중요한 프레임(Express Frame)이 도착했을 때, 진행 중이던 전송을 잠시 멈추고 긴급 프레임을 먼저 끼워넣어 보내는(선점하는) 기능이다.4

작동 원리는 저우선순위 프레임을 전송 가능한 작은 조각(fragment)으로 나눌 수 있도록 하는 것이다. 고우선순위 프레임 전송이 끝나면, 중단되었던 저우선순위 프레임의 나머지 조각들이 이어서 전송된다.9 이 기술을 통해, 거대한 비디오 프레임 하나가 네트워크를 독점하여 긴급한 안전 신호의 전송을 수백 마이크로초씩 지연시키는 상황을 방지할 수 있다. 결과적으로 TAS에서 요구되는 가드 밴드의 크기를 최소화하여 대역폭 효율을 높이는 데 기여한다.7


CBS는 원래 AVB(Audio Video Bridging) 표준의 일부로, 주로 전문 오디오/비디오 스트림과 같이 연속적이지만 TAS처럼 엄격한 시간 보장이 필요 없는 '소프트 실시간' 트래픽을 위해 설계되었다.7 이 기술의 목표는 트래픽이 한꺼번에 몰리는 '버스트(burst)' 현상을 완화하여 네트워크를 평탄화(smoothing)하는 것이다.

이는 각 트래픽 큐에 '크레딧(credit)'을 부여하는 방식으로 동작한다. 큐가 데이터를 전송하면 크레딧이 감소하고, 데이터를 전송하지 않고 대기하면 크레딧이 정해진 비율로 증가한다. 오직 크레딧이 양수일 때만 데이터 전송이 허용되므로, 트래픽이 장기적으로 할당된 대역폭을 초과하지 않도록 제어된다.11 이를 통해 수신 측 장치의 버퍼 오버플로우를 방지하고 안정적인 스트리밍을 보장한다.

이 외에도 완벽한 결정론적 지연 시간을 보장하는 IEEE 802.1Qch(Cyclic Queuing and Forwarding)나 비동기 트래픽을 위한 IEEE 802.1Qcr(Asynchronous Traffic Shaping)과 같은 추가적인 셰이핑 표준들이 특정 애플리케이션 요구에 맞춰 개발되고 있다.4


산업 제어나 자동차 안전 시스템과 같이 고도의 신뢰성이 요구되는 애플리케이션에서는 단일 고장 지점(Single Point of Failure)도 허용되지 않는다. IEEE 802.1CB FRER(Frame Replication and Elimination for Reliability)은 네트워크 케이블 단선이나 스위치 고장과 같은 장애 상황에서도 데이터 손실 없이 통신을 지속할 수 있도록 하는 '무손실 보호절체(Seamless Redundancy)' 기능을 제공한다.9

FRER의 작동 원리는 매우 직관적이다. 송신 측(Talker)의 특정 노드는 중요한 데이터 프레임을 복제하여, 물리적으로 완전히 분리된 두 개 이상의 경로로 동시에 전송한다.12 수신 측(Listener)에서는 각 경로를 통해 들어오는 프레임들을 감시하다가, 가장 먼저 도착한 프레임을 상위 애플리케이션으로 전달한다. 동일한 시퀀스 번호(sequence number)를 가진 중복 프레임이 나중에 도착하면, 이는 단순히 폐기된다.12 이 방식을 통해 한쪽 경로에 장애가 발생하더라도 다른 경로를 통해 데이터가 끊김 없이 전달되므로, 사실상 복구 시간이 '0'에 가까운 매우 높은 수준의 통신 신뢰성을 확보할 수 있다. 실제로 Moxa와 NXP가 진행한 시연에서는 작동 중인 네트워크 케이블을 물리적으로 뽑았음에도 불구하고 제어 데이터 패킷이 손실 없이 동기화된 상태로 정상 동작하는 것을 입증했다.16


TSN의 강력한 기능들을 제대로 활용하기 위해서는 네트워크 전체의 동작을 정교하게 설계하고 구성하는 과정이 필수적이다. 이는 TSN 도입의 가장 큰 장벽 중 하나로, 이를 해결하기 위한 표준들이 개발되었다.


초기 AVB/TSN은 스트림 예약 프로토콜(SRP)이라는 분산된 방식으로 각 장치가 자원을 요청하고 예약했다. 이 방식은 소규모 네트워크에서는 유효했지만, 수백, 수천 개의 노드가 존재하는 복잡한 스마트 팩토리 환경에서는 모든 경로와 스케줄을 계산하고 관리하는 것이 거의 불가능에 가까웠다.

IEEE 802.1Qcc는 이러한 한계를 극복하기 위해 '중앙 집중식 구성 모델'을 도입했다.7 이 모델은 CUC(Centralized User Configuration)와 CNC(Centralized Network Configuration)라는 두 가지 논리적 개체를 정의한다.18 CUC는 각 엔드포인트의 통신 요구사항(예: 'A장치에서 B장치로 1ms마다 100바이트 데이터 전송 필요')을 수집하는 역할을 한다. CNC는 CUC로부터 수집된 모든 요구사항과 전체 네트워크의 토폴로지 및 성능 정보를 바탕으로, 모든 데이터 스트림의 전송 경로, 각 스위치에서의 스케줄(GCL), 대역폭 예약을 중앙에서 계산하고 설정한다.7

이 중앙 집중식 모델 덕분에, 개별 엔드포인트나 사용자는 복잡한 네트워크 내부 구조나 스케줄링 메커니즘을 전혀 알 필요 없이, 마치 클라우드 서비스를 요청하듯 필요한 통신 서비스 품질(QoS)만 요청하면 된다. 이는 네트워크 관리의 복잡성을 사용자로부터 완벽하게 추상화하는 방식으로, TSN이 실험실 기술을 넘어 실제 산업 현장에 적용 가능한 '관리 가능한(manageable)' 기술로 진화하는 데 결정적인 역할을 했다. 이는 본질적으로 소프트웨어 정의 네트워킹(SDN)의 개념을 OT 영역으로 가져온 것으로 평가할 수 있다.

이 외에도 IEEE 802.1Qca(Path Control and Reservation)는 다중 경로 설정 및 명시적 경로 제어를 위한 규칙을 제공하며 7, IEEE 802.1Qcp는 NETCONF/RESTCONF 프로토콜과 함께 YANG 데이터 모델을 사용하여 TSN 장비의 구성과 상태 정보를 표준화된 방식으로 원격 관리할 수 있는 인터페이스를 제공한다.7

TSN을 구성하는 이 표준들은 각각 독립적으로 기능하지만, 실제 애플리케이션에서는 이들을 유기적으로 조합해야만 최적의 성능을 발휘할 수 있다. 예를 들어, 엄격한 실시간성을 위해 TAS(Qbv)를 사용하면서 프레임 선점(Qbu)을 결합하여 대역폭 효율을 높이고, 여기에 FRER(CB)을 추가하여 신뢰성을 확보하는 식이다. 따라서 TSN 솔루션의 경쟁력은 단순히 개별 표준의 지원 여부가 아니라, 이 '레고 블록'들을 얼마나 정교하게 조합하고 최적화하는지에 달려있다.

| **표 1: 주요 TSN 표준 요약 및 기능** |                           |                                                              |                                                              |                                               |
| ------------------------------------ | ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| **표준 번호**                        | **주요 명칭**             | **핵심 기능**                                                | **해결 과제**                                                | **주요 적용 분야**                            |
| **IEEE 802.1AS**                     | gPTP                      | 네트워크 전체 장치의 정밀 시간 동기화                        | 시간 기반 스케줄링의 전제 조건 마련, 분산된 데이터의 시간 정렬 | 모든 TSN 애플리케이션의 기반 기술             |
| **IEEE 802.1Qbv**                    | TAS (Time-Aware Shaper)   | TDMA 기반의 엄격한 시간 슬롯 스케줄링                        | 긴급 데이터가 다른 트래픽에 의해 지연되는 문제 원천 차단     | 하드 실시간 모션 제어, 안전 필수 시스템       |
| **IEEE 802.1Qbu/802.3br**            | Frame Preemption          | 저우선순위 프레임 전송을 중단하고 고우선순위 프레임을 선점 전송 | 긴 프레임으로 인한 긴급 데이터의 지연(Head-of-Line Blocking) 방지 | TAS의 대역폭 효율성 향상, 혼합 트래픽 환경    |
| **IEEE 802.1Qav**                    | CBS (Credit-Based Shaper) | 트래픽 버스트를 제어하고 흐름을 평탄화                       | 오디오/비디오 스트림의 안정적 전송, 버퍼 오버플로우 방지     | 전문 A/V 스트리밍, 소프트 실시간 애플리케이션 |
| **IEEE 802.1CB**                     | FRER                      | 프레임 복제 및 중복 경로 전송을 통한 무손실 이중화           | 단일 링크/노드 장애 시 통신 두절 문제 해결                   | 자동차 안전 시스템, 고가용성 산업 제어        |
| **IEEE 802.1Qcc**                    | Centralized Configuration | 중앙 집중식 네트워크 구성 및 자원 예약                       | 대규모 TSN 네트워크의 복잡한 구성 및 관리 문제 해결          | 스마트 팩토리, 대규모 분산 시스템             |
| **IEEE 802.1Qcp**                    | YANG Data Model           | YANG 데이터 모델을 이용한 표준화된 장비 관리                 | 벤더 종속적인 구성 방식 탈피, 자동화된 네트워크 관리         | 네트워크 관리 자동화, SDN 연동                |


TSN의 기술적 가치는 실제 산업 현장의 문제를 해결할 때 비로소 증명된다. TSN은 이론에 머무르지 않고 산업 자동화, 자동차, 전문 오디오/비디오 등 각기 다른 요구사항을 가진 분야에서 활발하게 적용되며 그 효용성을 입증하고 있다. 특히 각 산업에서 기존 기술로는 해결이 어렵거나 비효율적이었던 가장 엄격한 요구사항을 가진 '킬러 애플리케이션'을 중심으로 도입이 가속화되고 있다.


스마트 팩토리의 궁극적인 목표는 OT(운영 기술)와 IT(정보기술)의 완벽한 융합을 통해 데이터 기반의 지능형 생산 체계를 구축하는 것이다. 이를 위해서는 실시간 모션 제어를 위한 주기적 데이터, 품질 검사를 위한 고대역폭 비전 데이터, 설비 관리를 위한 비주기적 데이터, 그리고 상위 IT 시스템과 연동되는 TCP/IP 트래픽이 하나의 통합된 네트워크에서 원활하게 공존해야 한다.8 TSN은 바로 이 '통합 네트워크'를 구현하는 핵심 기술이다.

주요 산업용 이더넷 프로토콜 진영들은 TSN을 자사 기술에 통합하는 방식으로 이러한 변화에 대응하고 있다. 예를 들어, 미쓰비시전기가 주도하는 CLPA는 'CC-Link IE TSN'을, 지멘스가 중심인 PI는 'PROFINET over TSN'을 발표했다. 이들은 TSN의 표준 메커니즘을 활용하여 자사의 실시간 프로토콜과 표준 IT 트래픽이 동일한 물리적 네트워크에서 공존할 수 있는 개방형 아키텍처를 제공한다.20

구체적인 적용 사례를 통해 TSN의 효과를 확인할 수 있다.

- **고성능 모션 제어:** 폴란드의 인쇄기 제조사 KELLER는 최대 120축의 서보 모터를 사용하는 대형 보틀 인쇄기에 CC-Link IE TSN을 적용했다. 이를 통해 과거 여러 네트워크로 분리되어 있던 제어 시스템을 단일 네트워크로 통합하고, 정밀한 동기화를 통해 생산성을 향상시켰다.20 이는 IEEE 802.1AS의 정밀 시간 동기화와 IEEE 802.1Qbv의 엄격한 시간 스케줄링이 있었기에 가능했다.
- **실시간 데이터와 비전 시스템의 결합:** 네트워크 솔루션 기업 Moxa는 대만의 한 웨이퍼 제조 공정 라인에 TSN 솔루션을 성공적으로 통합했다. 이 시스템은 웨이퍼의 미세한 오정렬을 보정하기 위해 고대역폭의 머신 비전 데이터와 마이크로초 단위의 정밀 모션 제어 데이터가 실시간으로 연동되어야 했다. TSN은 이 두 종류의 이질적인 트래픽을 단일 네트워크에서 안정적으로 처리함으로써 시스템의 신뢰성을 높였으며, 네트워크 인프라의 유지보수 및 확장을 용이하게 하여 총소유비용(TCO)을 절감하는 효과를 거두었다.17

이러한 TSN의 접근 방식은 기존의 대표적인 실시간 이더넷 기술인 EtherCAT이나 PROFINET IRT와 비교할 때 그 차별점이 명확해진다. EtherCAT은 슬레이브 장치들이 이더넷 프레임을 수신 즉시 'On-the-fly'로 처리하여 전달하는 독특한 방식으로 매우 낮은 지연 시간을 구현하지만, 마스터-슬레이브 구조로 제한되며 표준 이더넷 IT 트래픽과의 자연스러운 융합이 어렵다.5 반면, PROFINET IRT는 결정론적 통신을 제공하지만 전용 하드웨어(ASIC)를 필요로 하여 유연성과 비용 측면에서 한계가 있었다. TSN은 표준 이더넷 스위치를 기반으로 유연한 토폴로지를 구성할 수 있고 IT 트래픽과 완벽하게 공존한다는 점에서 이들 기술의 한계를 극복한다. 흥미로운 점은 TSN이 이들 기술을 완전히 '대체'하기보다는 '통합'하는 방향으로 진화하고 있다는 것이다. EtherCAT 진영은 EtherCAT 세그먼트를 TSN 네트워크에 연결하는 하이브리드 아키텍처를 제시하고 있으며 23, PROFINET 진영은 전용 하드웨어 대신 표준 TSN 기능을 활용하는 'PROFINET over TSN'으로 전환하고 있다.26 이는 TSN의 진정한 가치가 새로운 폐쇄적 시스템을 만드는 것이 아니라, 기존의 다양한 기술들을 표준 이더넷이라는 공통의 언어 위에서 포용하고 통합하는 능력에 있음을 보여준다.


자동차 산업은 전동화, 자율주행, 커넥티드 기술의 발전으로 인해 역사상 가장 큰 변화를 겪고 있으며, 이는 차량 내부의 E/E(Electrical/Electronic) 아키텍처에 혁명적인 변화를 요구하고 있다. ADAS와 자율주행 기능이 고도화되면서 카메라, 레이더, 라이다 등 각종 센서에서 생성되는 데이터의 양이 폭발적으로 증가했다. 이러한 대용량 센서 데이터와 브레이크, 조향과 같이 안전에 직결된 제어 신호, 그리고 고화질 인포테인먼트 스트리밍 데이터가 차량 내 단일 네트워크 백본을 통해 실시간으로 처리되어야 한다.6

기존 차량 네트워크의 주축이었던 CAN, LIN, FlexRay와 같은 버스 기술들은 이러한 대역폭과 융합 요구사항을 감당하기에 역부족이다.3 이에 대한 해결책으로 '차량용 이더넷(Automotive Ethernet)'이 부상했으며, TSN은 이 차량용 이더넷에 결정론적 실시간 성능과 고신뢰성을 부여하는 핵심 기술로 채택되고 있다. TSN은 여러 ECU(Electronic Control Unit) 도메인을 연결하는 고속 백본 네트워크의 역할을 수행하며, 유연하고 확장 가능한 차세대 E/E 아키텍처를 구현하는 기반이 된다.13

특히 자율주행과 같이 기능 안전성(Functional Safety)이 극도로 중요한 애플리케이션에서 TSN의 역할은 절대적이다.

- **정밀한 센서 퓨전:** 여러 센서(카메라, 레이더 등)로부터 들어온 데이터를 시간 오차 없이 정확하게 융합하여 주변 환경을 인식하기 위해서는 1마이크로초 이하의 정밀한 시간 동기화(IEEE 802.1AS)가 필수적이다.13
- **예측 가능한 제어:** 센서 퓨전 결과를 바탕으로 한 판단 및 제어 신호가 예측 가능한 시간 내에 액추에이터(브레이크, 스티어링 휠 등)에 전달되어야 한다. IEEE 802.1Qbv(TAS)는 이를 위한 결정론적 데이터 전송을 보장한다.13
- **높은 신뢰성 확보:** 통신 경로에 장애가 발생하더라도 제어 신호가 유실되어서는 안 된다. IEEE 802.1CB(FRER)는 네트워크 경로를 이중화하여 단일 고장 지점을 제거함으로써 시스템의 신뢰성을 극대화한다.13

이처럼 TSN은 ADAS 및 자율주행 기능이 요구하는 실시간성, 대역폭, 안전성, 신뢰성이라는 까다로운 요구사항들을 표준 이더넷 기술 위에서 충족시킬 수 있는 사실상 유일한 대안이다.


흥미롭게도 TSN 기술의 뿌리는 산업이나 자동차가 아닌 전문 오디오/비디오(Pro A/V) 분야에 있다. TSN은 원래 라이브 공연장, 방송 스튜디오 등에서 다수의 오디오와 비디오 스트림을 아날로그 케이블 대신 이더넷 네트워크로 전송하기 위해 개발된 AVB(Audio Video Bridging) 표준에서 진화한 것이다.9

Pro A/V 분야의 핵심 과제는 여러 채널의 오디오와 비디오 신호를 사람의 눈과 귀가 인지할 수 없는 수준의 낮은 지연 시간과 완벽한 동기화(lip-sync) 상태로 전송하는 것이다. AVB는 이를 위해 세 가지 핵심 표준을 정의했으며, 이는 그대로 TSN의 일부가 되었다.

- **IEEE 802.1AS:** 정밀 시간 동기화를 통해 모든 A/V 장비가 동일한 시간축을 공유하게 한다.
- **IEEE 802.1Qat(SRP):** 스트리밍에 필요한 대역폭을 네트워크 경로상에 미리 예약한다.
- **IEEE 802.1Qav(CBS):** 예약된 대역폭 내에서 트래픽이 버스트되지 않도록 흐름을 제어하여 안정적인 전송을 보장한다.

TSN은 AVB의 이러한 기능을 계승하고 더욱 발전시켜, 패킷 손실이나 지터 없이 고품질의 A/V 스트림을 안정적으로 전송할 수 있는 강력한 솔루션을 제공한다.23

이러한 TSN 기반 A/V 시스템의 확산에 있어 중요한 역할을 하는 기관이 바로 'Avnu Alliance'이다.31 Avnu Alliance는 Pro A/V, 자동차, 산업 자동화 등 다양한 분야의 기업들이 참여하는 비영리 단체로, TSN 장비 간의 상호운용성을 검증하고 인증하는 역할을 수행한다. Avnu 인증을 받은 제품은 제조사가 다르더라도 하나의 TSN 네트워크에서 문제없이 함께 동작할 수 있음을 보증하므로, 사용자는 특정 벤더에 종속되지 않고 자유롭게 시스템을 구성할 수 있다.31

| **표 2: 산업별 TSN 적용 효과 및 핵심 표준** |                                                              |                                                              |                             |
| ------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------------------------- |
| **산업 분야**                               | **해결 과제**                                                | **TSN 도입 효과**                                            | **핵심 활용 표준**          |
| **산업 자동화**                             | OT와 IT 네트워크의 융합, 실시간 제어와 대용량 데이터의 공존  | 단일 네트워크 인프라 구축, 고성능 모션 제어, TCO 절감        | 802.1AS, 802.1Qbv, 802.1Qcc |
| **자동차 (자율주행)**                       | 다중 센서 데이터 융합 및 실시간 제어, 기능 안전성(FuSa) 확보 | µs 단위 정밀 동기화, 보장된 저지연 통신, 고신뢰성 달성       | 802.1AS, 802.1Qbv, 802.1CB  |
| **전문 오디오/비디오**                      | 다채널 A/V 스트림의 저지연 및 동기화(Lip-Sync) 전송          | 고품질 스트리밍 보장, 케이블링 간소화, 시스템 확장성 증대    | 802.1AS, 802.1Qav, 802.1Qat |
| **항공우주 및 방위**                        | 안전 필수(Safety-Critical) 제어, 분산된 시스템의 데이터 로깅 | 결정론적 통신, 높은 신뢰성, 시스템 재사용성 및 상호운용성    | 802.1AS, 802.1CB, 802.1DP   |
| **에너지 (스마트 그리드)**                  | 분산된 전력망의 실시간 모니터링 및 제어, 보호 계전           | 저지연 통신을 통한 신속한 장애 감지 및 복구, 네트워크 신뢰성 | 802.1AS, 802.1Qbu, 802.1CB  |


TSN은 기술적 잠재력을 바탕으로 빠르게 시장을 형성하며 성장하고 있다. Industry 4.0과 IIoT의 확산, 자동차 산업의 패러다임 전환 등 거시적인 변화가 TSN 시장의 성장을 견인하는 가운데, 반도체, 네트워크 장비, 산업 자동화 솔루션 등 다양한 분야의 기업들이 참여하며 복잡하면서도 역동적인 생태계를 구축하고 있다.


글로벌 TSN 시장의 규모와 성장률에 대한 전망은 시장 조사 기관에 따라 다소 차이를 보인다. 이는 TSN 시장의 정의(하드웨어, 소프트웨어, 서비스 포함 여부)가 기관마다 다르고, 기술이 아직 초기 성장 단계에 있어 예측 변동성이 크기 때문이다. 그럼에도 불구하고 모든 보고서가 공통적으로 가파른 성장세를 예측하고 있다는 점은 주목할 만하다.

- Fortune Business Insights에 따르면, 2024년 글로벌 TSN 시장 규모는 4억 5,390만 달러로 평가되었으며, 2025년부터 2032년까지 연평균 성장률(CAGR) 29.9%를 기록하며 2032년에는 35억 1,770만 달러에 이를 것으로 전망된다.32
- Market Research Future는 2024년 시장 규모를 22억 8,000만 달러로 다소 높게 추산했으며, 2035년까지 80억 달러 규모로 성장할 것으로 예측했다.33
- Zion Market Research는 2022년 2억 3,154만 달러였던 시장이 2030년까지 CAGR 32.01%로 성장하여 21억 3,548만 달러에 이를 것으로 보았다.34
- Maximize Market Research는 53%라는 매우 높은 CAGR을 예측하며 2030년 시장 규모가 61억 5,601만 달러에 달할 것이라고 전망했다.35

이러한 수치상의 불일치는 오히려 TSN 시장이 아직 미성숙하며 빠르게 진화하고 있다는 '성장통'의 증거로 해석할 수 있다. 이는 기술 도입을 고려하는 기업들에게 단기적인 시장 규모 예측보다는 기술의 성숙도와 생태계 확장 추세에 더 주목해야 한다는 시사점을 준다.

지역별로는 현재 아시아 태평양 지역이 빠른 산업화와 정부의 스마트 인프라 투자에 힘입어 가장 큰 시장 점유율(약 36.2%)을 차지하고 있다.32 북미는 주요 기술 기업들의 존재와 Industry 4.0에 대한 적극적인 투자로, 유럽은 독일을 중심으로 한 강력한 제조업 기반과 디지털 전환 노력으로 TSN 시장의 성장을 주도하고 있다.32


TSN 시장의 고속 성장은 몇 가지 핵심 동력에 의해 뒷받침되고 있다.

- **Industry 4.0 및 IIoT 채택 가속화:** 실시간 데이터 처리와 고도화된 산업 자동화에 대한 요구는 TSN 시장의 가장 근본적인 성장 동력이다. 공장 내 로봇, 공정 제어, 분산 제어 시스템 등은 밀리초 단위의 정밀하고 안정적인 통신을 필요로 하며, TSN은 이를 위한 최적의 솔루션을 제공한다.33
- **5G와의 융합:** 5G의 초저지연/고신뢰 통신(URLLC) 기능과 TSN의 결정론적 네트워킹이 결합되면서, 유선을 넘어 무선 환경까지 실시간 통신 영역이 확장되고 있다. 이는 자율주행차, 원격 의료, 스마트 그리드 등 새로운 실시간 애플리케이션 시장을 창출하는 기폭제가 될 것으로 예상된다.32
- **자동차 이더넷 수요 증가:** ADAS, 자율주행, 차량 내 인포테인먼트 시스템이 고도화되면서 차량 내 데이터 트래픽이 폭증하고 있다. TSN은 이러한 대용량 데이터를 결정론적으로 처리하는 데 필수적인 역할을 하며, 자동차 산업의 수요를 이끌고 있다.32
- **표준화 및 상호운용성 노력:** Avnu Alliance와 같은 인증 기관의 활동과 공급업체 간의 협력은 TSN 생태계를 활성화하고 있다. 표준화된 프로토콜과 상호운용 가능한 시스템 구축 노력은 다양한 산업 분야에서 TSN 채택을 가속화하는 긍정적인 요인으로 작용한다.33
- **가상화 및 클라우드 기반 솔루션:** 확장성과 유연성을 높이기 위해 가상화 및 클라우드 기반의 TSN 솔루션으로 전환하는 추세가 나타나고 있다. 이는 소프트웨어 정의 네트워킹(SDN)과 결합하여 더욱 지능적이고 동적인 네트워크 관리를 가능하게 할 것이다.37


TSN 생태계는 특정 기업이 독점하는 구조가 아닌, 여러 계층의 플레이어들이 경쟁과 협력을 병행하는 복잡한 구조를 띤다.

- **반도체 공급업체:** 이들은 TSN 기술 구현의 가장 근간이 되는 칩을 공급한다. Texas Instruments(TI)의 Sitara 프로세서, NXP의 SJA1110 이더넷 스위치, Marvell, Broadcom, Intel, Renesas, Analog Devices, Microchip 등 대부분의 주요 반도체 기업들이 TSN 표준을 하드웨어 수준에서 지원하는 스위치 IC, 프로세서, PHY 등을 출시하며 시장을 주도하고 있다.4
- **네트워크 장비 공급업체:** 이들은 반도체 칩을 기반으로 실제 네트워크를 구성하는 장비를 만든다. Cisco, Belden, Moxa, Kontron, Amphenol, Data Device Corporation(DDC) 등은 산업용 TSN 스위치, 게이트웨이, 네트워크 인터페이스 카드(NIC) 등을 제공하며 산업 현장과 같은 열악한 환경에서도 안정적으로 동작하는 제품을 공급한다.8
- **산업 자동화 기업:** 전통적인 자동화 시장의 강자들은 자사의 산업용 이더넷 프로토콜에 TSN을 통합하여 시장 지배력을 유지하려 하고 있다. Mitsubishi Electric(CC-Link IE TSN), Siemens(PROFINET over TSN), Rockwell Automation(EtherNet/IP over TSN) 등이 대표적이다. 이들은 기존 시스템과의 호환성을 유지하면서 TSN의 장점을 활용하는 마이그레이션 경로를 제공한다.21
- **표준화 기구 및 얼라이언스:** IEEE 802.1 TSN Task Group이 기술 표준을 제정하면, Avnu Alliance, CLPA(CC-Link Partner Association), PI(PROFIBUS & PROFINET International), ETG(EtherCAT Technology Group)와 같은 산업별 컨소시엄이 해당 산업에 맞는 프로파일을 개발하고 제품 간 상호운용성을 인증하는 역할을 수행한다.31

이러한 생태계 구조는 TSN의 성공이 특정 한 기업의 기술력만으로는 불가능하며, 반도체부터 솔루션, 표준화 기구에 이르는 생태계 전반의 유기적인 협력 수준에 달려있음을 명확히 보여준다. 경쟁사들이 각자의 프로토콜에 TSN을 통합하며 경쟁하는 동시에, IEEE라는 공통 표준을 따르고 Avnu Alliance와 같은 중립적인 기구에서 상호운용성을 위해 협력하는 '협력적 경쟁(Coopetition)' 구도가 TSN 생태계의 핵심적인 특징이다.

| **표 3: TSN 시장 주요 공급업체 및 제품군** |                                                              |                                                              |                                                              |
| ------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **공급망 계층**                            | **주요 공급업체**                                            | **대표 제품/솔루션**                                         | **주요 특징**                                                |
| **반도체 (IC)**                            | Texas Instruments, NXP, Marvell, Broadcom, Intel, Analog Devices | TSN 지원 프로세서 (예: Sitara), 스위치 IC, 이더넷 PHY        | TSN 핵심 기능(TAS, CBS, Preemption 등) 하드웨어 가속, 저전력 설계 |
| **네트워크 장비**                          | Cisco, Belden, Moxa, Kontron, Amphenol, DDC                  | 산업용 이더넷 스위치, 게이트웨이, NIC 카드                   | 견고한 설계(방진/방수), 넓은 동작 온도 범위, 다양한 TSN 프로파일 지원 |
| **산업용 솔루션**                          | Mitsubishi Electric, Siemens, Rockwell Automation, Beckhoff  | CC-Link IE TSN, PROFINET over TSN, EtherNet/IP, EtherCAT-TSN Gateway | 기존 산업용 프로토콜과의 호환성, 통합 엔지니어링 툴 제공     |
| **테스트 및 계측**                         | National Instruments(NI), Spirent, Keysight                  | TSN 지원 CompactRIO, PXI 플랫폼, 트래픽 생성 및 분석기       | TSN 네트워크 성능 검증, 동기화 정확도 측정, 상호운용성 테스트 |
| **표준/인증 기관**                         | IEEE, Avnu Alliance, CLPA, PI, ETG                           | TSN 표준 제정, 산업별 프로파일 개발, 상호운용성 인증 프로그램 | 개방성 및 상호운용성 확보, 생태계 활성화                     |


TSN은 의심할 여지 없이 미래 산업 네트워크의 청사진을 제시하고 있지만, 그 잠재력을 현실화하기까지는 몇 가지 중요한 과제를 해결해야 한다. 구현의 복잡성이라는 현실적인 장벽을 넘어, 5G와 같은 차세대 기술과의 융합을 통해 새로운 가능성을 열어가는 과정에 있다.


TSN 도입을 가로막는 가장 큰 장벽은 '복잡성'이다. TSN은 기존 이더넷처럼 단순히 케이블을 연결하면 동작하는 '플러그 앤 플레이(Plug-and-Play)' 기술이 아니다.7

- **네트워크 구성의 복잡성:** TSN의 결정론적 성능을 보장하기 위해서는 네트워크에 연결된 모든 장치의 동작을 사전에 정교하게 설계해야 한다. 데이터 스트림의 전송 경로, 각 스위치에서 적용될 시간 슬롯 스케줄(802.1Qbv), 예약할 대역폭 등을 중앙 집중형 구성 툴(CNC)이나 분산형 프로토콜을 통해 설정해야 한다.18 이는 기존 네트워크 엔지니어에게는 생소한 작업이며 상당한 학습 곡선을 요구한다. PROFINET 진영이 NME(Network Management Engine)를 통해 이 과정을 자동화하려는 시도 26나 NI와 같은 기업이 LabVIEW를 통해 통합된 개발 환경을 제공하려는 노력 47은 모두 이 복잡성을 해결하기 위함이다. 미래 TSN 시장의 경쟁력은 하드웨어 성능뿐만 아니라, 이 복잡한 구성을 얼마나 간단하게 추상화하여 사용자에게 제공하는가, 즉 '사용 편의성'에 달려있을 가능성이 높다.
- **하드웨어 및 소프트웨어 요구사항:** TSN의 핵심 기능들, 특히 TAS(Time-Aware Shaper)나 프레임 선점과 같은 기능은 나노초 단위의 정밀한 타이밍 제어를 필요로 한다. 이는 일반적인 CPU만으로는 구현하기 어려우며, TSN 표준을 지원하는 전용 하드웨어(ASIC 또는 FPGA)가 내장된 스위치, 네트워크 카드(NIC), 프로세서가 필수적이다.14 또한, 이를 구성, 관리, 모니터링하기 위한 소프트웨어 스택과 엔지니어링 툴 역시 갖추어져야 한다.50
- **기술적 한계 및 상호운용성:** TSN은 표준 이더넷의 효율성 문제를 완벽하게 해결하지는 못하며, 특히 엔드 노드당 전송 데이터 양이 적은 경우 비효율적일 수 있다.23 또한, 802.1Qbv(TAS) 사용 시 발생하는 가드 밴드로 인한 대역폭 손실은 여전히 해결해야 할 과제다.7 표준 기술임에도 불구하고, 서로 다른 벤더의 장비나 다른 산업 프로파일(예: CC-Link IE TSN과 PROFINET over TSN) 간의 완벽한 상호운용성을 보장하는 것은 여전히 진행 중인 과제이다.36


이러한 과제에도 불구하고 TSN의 미래는 매우 밝으며, 그 잠재력은 5G 이동통신 기술과의 융합을 통해 극대화될 전망이다. TSN이 유선 네트워크 구간의 결정론적 통신을 책임지고, 5G가 무선 구간에서 초저지연/고신뢰 통신(URLLC)을 담당함으로써, 공장 내부에서부터 클라우드에 이르기까지 끊김 없는(Seamless) End-to-End 실시간 네트워크를 구축할 수 있게 된다.51

이러한 유무선 융합 인프라는 과거에는 상상하기 어려웠던 차세대 애플리케이션을 현실로 만들 것이다.

- **협업 모바일 로봇 및 자율 물류:** 공장 내를 자유롭게 이동하는 자율이동로봇(AMR)이나 무인운반차(AGV)가 물리적인 케이블 연결 없이도 다른 고정 설비나 로봇과 정밀하게 동기화하여 협업 작업을 수행할 수 있게 된다.51
- **원격 제어 및 디지털 트윈:** 5G와 TSN의 결합은 원격지에 있는 로봇 팔이나 중장비를 지연 없이 정밀하게 제어하는 것을 가능하게 한다. 이는 위험한 환경에서의 원격 작업이나 원격 수술 분야에 혁신을 가져올 수 있다. 또한, 물리적 자산의 디지털 트윈과 실시간으로 데이터를 동기화하여 시뮬레이션 및 예지보전의 정확도를 획기적으로 높일 수 있다.51
- **유연한 생산 시스템:** 데이터 처리 및 제어 로직을 현장 장비가 아닌 엣지 또는 클라우드 서버로 이전(Offload)하고, 그 결과를 실시간으로 현장 액추에이터에 전달하는 유연한 시스템 구축이 가능해진다. 이는 생산 라인 변경 시 하드웨어나 케이블링 변경을 최소화하고 소프트웨어 정의만으로 생산 시스템을 재구성하는, 진정한 의미의 '플러그 앤 프로듀스(Plug-and-Produce)'를 실현하는 기반이 된다.8

5G와 TSN의 융합은 네트워크를 단순히 데이터를 전달하는 고정된 '인프라'에서, 애플리케이션의 요구에 따라 실시간성, 대역폭, 신뢰도가 각기 다른 가상 네트워크를 동적으로 생성하고 제공하는 '동적 서비스 플랫폼'으로 변화시킬 것이다. 이 외에도 기술적으로는 장거리 통신을 위한 광섬유 기반 TSN 구현, 10Gbps를 넘어 400Gbps에 이르는 고대역폭 지원, AI/ML 기술을 통합하여 네트워크 상태를 스스로 진단하고 성능을 동적으로 최적화하는 방향으로 끊임없이 발전하고 있다.37


TSN은 복잡하고 도전적인 기술이지만, 그 이면에는 산업 전반의 디지털 전환을 이끌 강력한 잠재력이 있다. TSN 도입을 고려하는 기업이나 엔지니어는 다음과 같은 관점을 가질 필요가 있다.

첫째, TSN을 단기적인 문제 해결을 위한 도구가 아닌, 장기적인 표준화와 미래 확장성을 고려한 '미래 보장형(Future-proof)' 인프라 투자로 접근해야 한다.47 현재의 애플리케이션뿐만 아니라, 향후 도입될 AI 기반 비전 검사, 빅데이터 분석, 클라우드 연동 등 미래의 요구사항까지 수용할 수 있는 유연한 기반을 마련한다는 관점이 중요하다.

둘째, 성공적인 TSN 도입은 하드웨어 선택을 넘어, 전체 시스템을 아우르는 통합적인 설계와 검증 역량에 달려있다. 신뢰할 수 있는 파트너(반도체, 장비, 솔루션, 테스트 기업)와의 협력을 통해 복잡한 구성 문제를 해결하고, 실제 환경에서의 성능과 상호운용성을 철저히 검증하는 과정이 필수적이다.

결론적으로, 시간 민감형 네트워킹(TSN)은 단순한 네트워킹 기술의 발전을 넘어선다. 이는 Industry 4.0, 자율주행, 스마트 시티와 같은 미래 산업의 핵심 요구사항인 '실시간 데이터 융합'을 실현하는 가장 근본적인 기술(Foundational Technology)이다. 구현의 복잡성이라는 단기적인 과제에도 불구하고, TSN이 제공하는 개방성, 확장성, 그리고 결정론적 성능은 지난 수십 년간 파편화되어 있던 산업 네트워크 환경을 하나로 통합하는 유일한 대안이다. TSN은 OT와 IT의 장벽을 허물고, 기업 전체의 데이터를 원활하게 연결함으로써, 차세대 디지털 혁신을 이끄는 중추적인 역할을 수행할 것이며, 그 중요성은 시간이 지날수록 더욱 커질 것이다.


1. [네트워크 관리] 이더넷의 종류와 특징, accessed July 13, 2025, https://zeromin-code.tistory.com/m/30
2. IEEE 802 LAN Architecture, accessed July 13, 2025, http://kocw-n.xcache.kinxcdn.com/data/document/2020/pusancatholic/yudonghui0727/4.pdf
3. 캔통신과 이더넷통신(CAN Bus vs Ethernet Communication, 비교) - 팜테크(FAMTECH), accessed July 13, 2025, https://famtech.tistory.com/316
4. 산업 자동화를 위한 시간 민감형 네트워킹(TSN), accessed July 13, 2025, https://www.seminet.co.kr/ms_pdf/702_20180508111717_201805_ti.pdf
5. 실시간 이더넷에 대한 심층 분석 - ADI, accessed July 13, 2025, http://eewebinar.co.kr/adi/tech_view.asp?idx=445
6. 표준 이더넷 기반의 군사용 지상무인차량 실시간 네트워크 구현, accessed July 13, 2025, https://ki-it.com/xml/30747/30747.pdf
7. 시간 민감성 네트워킹 - 위키백과, 우리 모두의 백과사전, accessed July 13, 2025, [https://ko.wikipedia.org/wiki/%EC%8B%9C%EA%B0%84_%EB%AF%BC%EA%B0%90%EC%84%B1_%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%82%B9](https://ko.wikipedia.org/wiki/시간_민감성_네트워킹)
8. [이슈] 미래의 공장 위한 TSN 기반, 단일화된 네트워크 솔루션... - 인공지능신문, accessed July 13, 2025, https://www.aitimes.kr/news/articleView.html?idxno=14954
9. TSN(Time-Sensitive Networking) - ITPE * JackerLab, accessed July 13, 2025, https://itpe.jackerlab.com/m/entry/TSNTime-Sensitive-Networking
10. 회사 알아보기_230525 - ㅇ - 티스토리, accessed July 13, 2025, https://horang-guya.tistory.com/122
11. [기고] EtherNet/IP에 TSN 적용하기(1) - elec4, accessed July 13, 2025, https://www.elec4.co.kr/article/articleView.asp?idx=29617
12. Time Sensitive Network (TSN) :: 알쓸유공, accessed July 13, 2025, https://kyoko0825.tistory.com/entry/Time-Sensitive-Network-TSN
13. 차량용 Ethernet - SOA, POSIX, TSN의 미래 - Vector, accessed July 13, 2025, https://cdn.vector.com/cms/content/know-how/_technical-articles/Ethernet_Trends_AutomobilElektronik_201712_PressArticle_KO.pdf
14. Time-Sensitive Networking - Wikipedia, accessed July 13, 2025, https://en.wikipedia.org/wiki/Time-Sensitive_Networking
15. © 2017 한국전자통신연구원, accessed July 13, 2025, https://ettrends.etri.re.kr/ettrends/163/0905002179/32-1_13-24.pdf
16. 실제 산업 환경에 활용 가능한 TSN의 입증된 이점 - 오토메이션월드, accessed July 13, 2025, https://automation-world.co.kr/news/article.html?no=67259
17. 실제 산업 환경에 활용 가능한 TSN의 입증된 이점 - 헬로티, accessed July 13, 2025, https://www.hellot.net/mobile/article.html?no=86492
18. 시간 민감형 네트워킹은 어떻게 산업 자동화를 혁신시키는가 < 칼럼 ..., accessed July 13, 2025, https://www.epnc.co.kr/news/articleView.html?idxno=90786
19. 시간 민감성 네트워킹 - 오늘의AI위키, AI가 만드는 백과사전, accessed July 13, 2025, [https://wiki.onul.works/w/%EC%8B%9C%EA%B0%84_%EB%AF%BC%EA%B0%90%EC%84%B1_%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%82%B9](https://wiki.onul.works/w/시간_민감성_네트워킹)
20. CC-Link IE TSN 도입으로 스마트공장 새 시대 열다 - 인더스트리뉴스, accessed July 13, 2025, https://www.industrynews.co.kr/news/articleView.html?idxno=59284
21. PROFINET over TSN - www.profibus.com, accessed July 13, 2025, https://www.profibus.com/technologies/profinet-over-tsn
22. Simulation Framework for EtherCAT over TSN - IFIP Digital Library, accessed July 13, 2025, https://dl.ifip.org/db/conf/networking/networking2021/1570712131.pdf
23. EtherCAT 과 TSN – 산업용 이더넷 시스템 아키텍처 모범 사례, accessed July 13, 2025, https://www.ethercat.org/download/documents/Whitepaper_EtherCAT_and_TSN_KR.pdf
24. TSN - EtherCAT Technology Group, accessed July 13, 2025, https://www.ethercat.org/en/ethercat_and_tsn.htm
25. EtherCAT and TSN - Best Practices for Industrial Ethernet System Architectures, accessed July 13, 2025, https://www.ethercat.org/download/documents/whitepaper_ethercat_and_tsn.pdf
26. What Is Time Sensitive Networking (TSN)? – PI North America - PROFINET, accessed July 13, 2025, https://us.profinet.com/digital/tsn/
27. Questions and Answers about TSN - Siemens Industry Online Support, accessed July 13, 2025, https://cache.industry.siemens.com/dl/files/263/109757263/att_1075932/v1/109757263_TSN_QA_V2.0_en.pdf
28. 5G V2X 서비스를 위한 이더넷 TSN 기반 10Base-T1S 알고리즘, accessed July 13, 2025, https://conf.kics.or.kr/2024s/media?key=site/2024s/abs/0312-BVHZC.pdf
29. 차량 내부 - 한국정보통신기술협회, accessed July 13, 2025, https://tta.or.kr/data/androReport/ttaJnal/160-2-3-6.pdf
30. Avnu Alliance | Milan Advanced Certification Testing - Granite River Labs, accessed July 13, 2025, https://www.graniteriverlabs.com/ko-kr/avnu-alliance-certification-testing
31. [기술 기고] TSN이 ODVA 기술에 미치는 영향 - elec4, accessed July 13, 2025, https://www.elec4.co.kr/article/articleView.asp?idx=29392
32. Time-Sensitive Networking Market Size, Growth, Share Report 2032, accessed July 13, 2025, https://www.fortunebusinessinsights.com/time-sensitive-networking-market-109627
33. Time-Sensitive Networking Market Size, Growth Report 2035, accessed July 13, 2025, https://www.marketresearchfuture.com/reports/time-sensitive-networking-market-10524
34. Time-Sensitive Networking Market Size, Share, Trends & Forecast 2030, accessed July 13, 2025, https://www.zionmarketresearch.com/report/time-sensitive-networking-market
35. Time-Sensitive Networking Market – Global Industry analysis and Forecast (2024-2030), accessed July 13, 2025, https://www.maximizemarketresearch.com/market-report/global-time-sensitive-networking-market/8455/
36. Time-Sensitive Networking Market Size & Growth Report 2032 - SNS Insider, accessed July 13, 2025, https://www.snsinsider.com/reports/time-sensitive-networking-market-6830
37. 시간에 민감한 네트워킹 Tsn 시장: 동향 및 기회 2032 - WiseGuy Reports, accessed July 13, 2025, https://www.wiseguyreports.com/ko/reports/time-sensitive-networking-tsn-market
38. NXP, 고속 네트워크 기술 지원하는 차량용 TSN 이더넷 스위치 발표 - elec4, accessed July 13, 2025, https://elec4.co.kr/article/articleview.asp?idx=24921
39. Global TSN Ethernet Switch ICs Market Research Report 2025, accessed July 13, 2025, https://reports.valuates.com/market-reports/QYRE-Auto-11A19169/global-tsn-ethernet-switch-ics
40. NXP Semiconductors: Automotive, IoT & Industrial Solutions, accessed July 13, 2025, https://www.nxp.com/
41. Time-Sensitive Networking (TSN) 10G Ethernet Switches | Products - Amphenol Aerospace, accessed July 13, 2025, https://www.amphenol-aerospace.com/products/time-sensitive-networking-tsn-10g-ethernet-switches
42. Time-Sensitive Networking (TSN) - Data Device Corporation, accessed July 13, 2025, https://www.ddc-web.com/en/time-sensitive-networking-tsn
43. Ethernet & TSN Switching - Kontron, accessed July 13, 2025, https://www.kontron.com/en/iot/ethernet-tsnswitching
44. Time-Sensitive Network (TSN): Revolutionising Industrial Automation, accessed July 13, 2025, https://emea.mitsubishielectric.com/fa/news/blog/time-sensitive-network-tsn-revolutionising-industrial-automation
45. EtherCAT Technology Group adds TSN technologies to EtherCAT - Automation.com, accessed July 13, 2025, https://www.automation.com/en-us/articles/2017/ethercat-technology-group-adds-tsn-technologies-to
46. TSN – What's happening? - HMS Networks, accessed July 13, 2025, https://www.hms-networks.com/news/news-details/17-06-2025-tsn---what-s-happening
47. Time Sensitive Networking (TSN) Frequently Asked Questions - NI, accessed July 13, 2025, https://www.ni.com/en/shop/data-acquisition/time-sensitive-networking--tsn--frequently-asked-questions.html
48. TN1 | TSN Ethernet Switch End Point (TSN-SE) - North Atlantic Industries, accessed July 13, 2025, https://www.naii.com/model/TN1
49. TSN Network Node - NetTimeLogic GmbH, accessed July 13, 2025, https://www.nettimelogic.com/tsn-network-node.php
50. Python and TSN (Time-Sensitive Networking): An In-Depth Guide - W3computing.com, accessed July 13, 2025, https://www.w3computing.com/articles/python-tsn-time-sensitive-networking-guide/
51. 5G and TSN: Industrial Automation of the Future - Rutronik, accessed July 13, 2025, https://www.rutronik.com/article/5g-and-tsn-for-the-industrial-automation-of-the-future
52. TSN: the future of open industrial Ethernet, accessed July 13, 2025, https://iebmedia.com/tsn-the-case-for-action-now/tsn-the-future-of-open-industrial-ethernet/


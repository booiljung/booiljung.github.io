# 최신 로봇 및 우주항공 시스템을 위한 DDS와 TSN 적용 가능성

## 요약 (Executive Summary)

본 보고서는 데이터 분산 서비스(Data Distribution Service, DDS)와 시간 민감형 네트워킹(Time-Sensitive Networking, TSN) 기술이 로봇 및 우주항공이라는 두 가지 고도의 기술 집약적 분야에 즉시 적용 가능한지에 대한 전략적 질문에 답하는 것을 목표로 한다. "즉시 적용 가능성"은 단순한 기술적 구현 가능성을 넘어, 기술 성숙도, 표준화, 생태계 지원, 그리고 실제 생산 시스템에 통합할 때의 예측 가능성과 투자 수익률까지 포함하는 포괄적인 개념이다.

분석 결과, DDS는 이미 두 분야 모두에서 핵심 기술로 자리 잡은 매우 성숙한(기술 준비도 TRL 8-9) 생산 준비 완료 기술임이 확인되었다. 반면, TSN은 표준 이더넷에 결정성을 부여하는 혁신적인 기술 모음으로, 특정 기능에 따라 TRL 6-8 수준의 빠른 성숙 단계를 보이고 있으며 관련 하드웨어와 소프트웨어의 가용성이 점차 확대되고 있다. 이 두 기술의 조합인 'DDS over TSN'은 아직 보편적인 '플러그 앤 플레이' 솔루션은 아니지만, 차세대 시스템의 필연적인 진화 방향을 제시하는 강력한 신흥 아키텍처이다.

**로봇 분야 권고 사항:** 로봇 운영체제 ROS 2(Robot Operating System 2)를 중심으로 하는 로봇 분야에서 DDS는 이미 표준 통신 미들웨어로서 즉시 적용 가능하다. TSN의 통합은 하드웨어 실시간(hard real-time) 성능이 요구되는 애플리케이션을 위한 전략적 다음 단계이며, 신규 설계를 중심으로 상당한 네트워크 엔지니어링 전문성을 전제로 할 때 실현 가능하다.

**우주항공 분야 권고 사항:** 우주항공 분야에서 DDS는 인증 가능한 솔루션과 함께 임무 수행에 필수적인(mission-critical) 시스템에서 그 성능이 입증된 기술이다. TSN은 기존 항공전자 버스를 대체할 차세대 기술로 명확히 자리매김하고 있으나, 최고 안전 등급(DAL A) 시스템을 위한 인증 경로는 아직 개척 중이다. 따라서 덜 치명적인 시스템이나 지상 장비부터 시작하는 단계적 도입 전략이 권장된다.

결론적으로, '즉각적인 보편적 적용'이라는 표현은 다소 과장된 측면이 있으나, 두 기술 스택에 대한 전략적이고 체계적인 계획 기반의 도입은 단순히 실현 가능한 수준을 넘어, 두 분야의 차세대 시스템이 미래 경쟁력을 확보하기 위한 필수적인 과정이라고 할 수 있다.

------

## 1.  결정론적 통신의 근간이 되는 기술들

DDS와 TSN의 시너지를 분석하기에 앞서, 각 기술의 개별적인 원리, 강점, 그리고 현재 기술적 위치를 명확히 이해하는 것이 필수적이다. 이 장에서는 두 기술을 독립적으로 심층 분석하여 이후의 통합 아키텍처 논의를 위한 기반을 마련한다.

### 1.1  데이터 중심 미들웨어: 데이터 분산 서비스(DDS) 심층 분석

#### 1.1.1  아키텍처 원리: 발행-구독 모델에서 글로벌 데이터 공간까지

Object Management Group(OMG)에 의해 표준화된 DDS는 분산 시스템 내에서 신뢰성 있고, 고성능의, 실시간 데이터 교환을 목표로 하는 기계 대 기계(M2M) 통신 미들웨어이다.1 DDS의 핵심적인 가치는 복잡한 네트워크 프로그래밍의 세부 사항을 애플리케이션 개발자로부터 추상화하여, 개발자가 비즈니스 로직에 집중할 수 있도록 만드는 데 있다.1

DDS의 아키텍처는 '데이터 중심성(Data-Centricity)'이라는 패러다임에 기반한다. 이는 원격 객체의 메서드를 호출하는 CORBA와 같은 '객체 중심(Object-Centric)' 모델과는 근본적으로 다르다.4 DDS는 통신의 주체를 데이터 자체로 보고, 시스템의 모든 참여자가 공유하는 가상의 데이터 저장소인 '글로벌 데이터 공간(Global Data Space)' 개념을 도입한다.5 애플리케이션은 이 공간에 있는 데이터 객체(Topic)를 마치 로컬 메모리에 접근하듯 읽고 쓸 수 있으며, 미들웨어는 이 데이터의 일관성과 배포를 책임진다.3 이러한 접근 방식은 여러 하위 시스템의 상태 정보가 실시간으로 공유되어야 하는 분산 제어 시스템에 매우 적합하다.4

이러한 데이터 중심 철학은 발행-구독(Publish-Subscribe) 모델을 통해 구현된다.1 정보를 생성하는 노드(Publisher)는 특정 주제, 즉 '토픽(Topic)'을 생성하고 데이터 '샘플(Sample)'을 발행한다. 해당 토픽에 관심이 있는 노드(Subscriber)는 구독을 선언하고 샘플을 수신한다. 이 과정에서 DDS 미들웨어는 참여자 자동 발견(discovery), 데이터 직렬화/역직렬화(marshalling/demarshalling), 전달, 흐름 제어, 재시도 등 모든 중간 과정을 처리한다.1 이로 인해 발행자와 구독자는 서로의 존재나 네트워크 상의 위치를 알 필요 없이 완벽하게 분리(decoupled)될 수 있으며, 이는 시스템의 유연성과 확장성을 극대화하는 핵심적인 장점이다.1

특히 '동적 발견(Dynamic Discovery)' 기능은 DDS의 강력함을 보여주는 특징이다.5 네트워크에 새로운 참여자가 추가되면, 기존 참여자들은 이를 자동으로 인지하고 필요한 데이터 통신을 설정한다. 이는 중앙 관리자 없이도 "플러그 앤 플레이"와 같은 연결성을 제공하여, 시스템의 강건함과 유연성을 크게 향상시킨다.3

DDS 표준은 두 가지 수준의 인터페이스를 정의한다. 하위 레벨에는 효율적인 데이터 전달에 초점을 맞춘 데이터 중심 발행-구독(DCPS, Data-Centric Publish-Subscribe) 계층이 있으며, 상위 레벨에는 애플리케이션 통합을 용이하게 하는 선택적 데이터 로컬 재구성 계층(DLRL, Data-Local Reconstruction Layer)이 존재한다.1

단순한 통신 프로토콜을 넘어, DDS는 분산 시스템을 구축하는 강력한 아키텍처 패턴을 제공한다. 데이터 중심성, 동적 발견, 그리고 후술할 QoS 기반의 계약 모델은 시스템 구성 요소를 근본적으로 분리시킨다. 이는 새로운 구성 요소를 추가하거나 기존의 것을 업데이트할 때 시스템의 다른 부분에 미치는 영향을 최소화함을 의미한다. 이러한 특성은 긴 수명주기를 갖는 우주항공 프로그램이나 빠르게 진화하는 로봇 애플리케이션에서 장기적인 유지보수 비용을 절감하고 시스템의 민첩성을 높이는, 단순한 통신 기능 이상의 전략적 가치를 제공한다.

#### 1.1.2  제어의 엔진: DDS 서비스 품질(QoS) 정책의 세분화된 분석

DDS의 진정한 힘은 방대하고 세밀하게 제어 가능한 서비스 품질(QoS) 정책에 있다.4 이 QoS 정책들은 단순한 설정 값이 아니라, 데이터를 보내는 DataWriter와 받는 DataReader 간의 명시적인 '계약(contract)'으로 작용한다.11 통신은 오직 DataWriter가 제공(offered)하는 QoS가 DataReader가 요청(requested)하는 QoS와 양립 가능할 때만 성립된다.12 이 메커니즘을 통해 개발자는 시스템의 비기능적 속성, 즉 통신의 신뢰성, 실시간성, 자원 사용량 등을 매우 정밀하게 제어할 수 있다.

실시간 시스템 설계에 있어 특히 중요한 QoS 정책들은 다음과 같다:

- **RELIABILITY (신뢰성):** `BEST_EFFORT`와 `RELIABLE` 두 가지 옵션을 제공한다. `BEST_EFFORT`는 속도를 우선시하여 데이터 손실을 감수하는 반면, `RELIABLE`은 재전송 등을 통해 데이터가 반드시 전달됨을 보장한다. 중요한 점은 `RELIABLE`을 요청하는 DataReader는 `BEST_EFFORT`로 제공하는 DataWriter와 연결될 수 없다는 것이다.7 이는 시스템 설계 시 신뢰성과 성능 사이의 근본적인 트레이드오프를 명시적으로 결정하게 한다.
- **DURABILITY (지속성):** 데이터의 생명주기를 제어한다. `VOLATILE`은 현재 구독 중인 참여자에게만 데이터를 전달하고, `TRANSIENT_LOCAL`은 발행자가 최신 데이터를 캐싱하여 늦게 참여하는 구독자에게 전달한다. `TRANSIENT` 또는 `PERSISTENT`는 발행자가 사라져도 서비스가 데이터를 유지하여 시스템 전체의 상태 일관성을 보장한다.7
- **DEADLINE (마감 시간):** 데이터 샘플 간의 최대 허용 시간 간격을 정의한다. 만약 이 시간이 초과되면 미들웨어는 애플리케이션에 이를 통지할 수 있다.10 이는 주기적인 제어 루프의 정상 동작을 감시하는 데 필수적이다.
- **LATENCY_BUDGET (지연 예산):** 데이터의 긴급성을 미들웨어에 알리는 수단으로, 발행 시점부터 구독자의 데이터 캐시에 도달하기까지의 종단 간 시간 예산을 명시한다.10
- **LIVELINESS (생존성):** 참여자가 여전히 활성 상태인지 판단하기 위한 하트비트 메커니즘이다. 시스템의 결함을 감지하고 예비 시스템으로의 전환(failover)을 구현하는 데 중요하다.14
- **HISTORY (이력):** `KEEP_LAST(n)` 또는 `KEEP_ALL` 설정을 통해 구독자를 위해 과거 데이터를 몇 개나 저장할지 제어한다.7

이러한 QoS 정책들은 일반적으로 XML 파일을 통해 설정되므로, 애플리케이션 코드를 재컴파일하지 않고도 시스템의 통신 동작을 변경할 수 있다. 이는 시스템 통합 및 유지보수 과정에서 큰 유연성을 제공한다.9

DDS의 정교한 QoS 정책들은 결정론적 통신과 자원 활용 사이의 내재적 긴장 관계를 드러낸다. DDS는 `RELIABLE` 전송, `DEADLINE` 주기, `LATENCY_BUDGET`과 같은 강력한 시간 기반 정책을 제공한다.10 그러나 대부분의 DDS 구현체는 본질적으로 '최선 노력형(best-effort)' 전송 방식인 표준 UDP/IP 위에서 동작한다.16 이는 근본적인 모순을 야기한다. DDS는 네트워크 정체로 인해 발생한 마감 시간 위반을 

*감지*할 수는 있지만, 이를 *예방*할 수는 없다. 신뢰성을 위해 누락된 패킷을 재전송할 수 있지만, 이 재전송 과정 자체가 예측 불가능한 지연(jitter)을 추가한다. 따라서 DDS의 정교한 QoS 엔진은 감지를 넘어 예방의 영역으로 나아가, 그 잠재력을 완전히 발휘하기 위해 결정론적 보장을 제공할 수 있는 하위 계층 네트워크, 즉 TSN과 같은 파트너 기술을 필연적으로 요구하게 된다.

#### 1.1.3  DDS 생태계: 주요 상용 및 오픈소스 구현체 검토

다수의 상호 운용 가능한 DDS 구현체가 존재한다는 사실은 이 표준이 매우 성숙했음을 방증한다. 이 생태계는 크게 상용 솔루션과 오픈소스 솔루션으로 나뉜다. 상용 분야에서는 RTI(Real-Time Innovations)의 Connext DDS가 항공우주 및 국방과 같은 고신뢰성 산업에서 시장을 선도하고 있다.17 오픈소스 분야에서는 eProsima의 Fast DDS가 ROS 2의 기본 미들웨어로 채택되어 로봇 커뮤니티에서 폭넓게 사용되고 있으며 7, Eclipse Cyclone DDS와 같은 다른 강력한 대안도 존재한다.7

이러한 다양한 구현체들 간의 상호 운용성은 DDS 표준의 핵심적인 장점이다. 이는 DDSI-RTPS(Real-Time Publish-Subscribe Protocol)라는 와이어 프로토콜(wire protocol) 명세를 통해 보장된다.16 덕분에 개발자는 특정 벤더에 종속되지 않고, 시스템의 요구사항에 가장 적합한 구현체를 선택하거나 혼용할 수 있다. 이는 개방형 아키텍처를 지향하는 현대적인 시스템 설계 철학에 부합한다.

또한, 벤더들은 특정 요구사항을 만족시키기 위한 특화된 버전의 DDS를 제공한다. 예를 들어, 마이크로컨트롤러와 같이 자원이 극도로 제한된 환경을 위한 DDS-XRCE(DDS for eXtremely Resource Constrained Environments)나 1, 항공전자 시스템의 엄격한 안전 인증 요구사항을 만족시키기 위한 DO-178C 인증 증거 패키지를 포함한 RTI Connext DDS Cert와 같은 제품이 있다.21 이는 DDS 기술이 다양한 응용 분야의 특수한 요구에 맞춰 진화하고 적응할 수 있는 높은 유연성과 확장성을 가지고 있음을 보여준다.

### 1.2  네트워크 계층에서 결정론 강제: 시간 민감형 네트워킹(TSN) 제품군

#### 1.2.1  TSN의 사명: 표준 이더넷을 예측 가능한 패브릭으로 변환

TSN은 표준 이더넷을 결정론적(deterministic)으로 만들기 위해 고안된 IEEE 802.1 표준들의 집합이다.22 이 기술은 OSI 모델의 2계층(데이터 링크 계층)에서 동작하며, 기존 이더넷 인프라를 활용하면서 실시간 통신을 가능하게 하는 것을 목표로 한다.23

표준 이더넷은 높은 대역폭을 제공하도록 설계되었지만, 통신 매체에 대한 접근이 경쟁 기반으로 이루어지기 때문에 지연 시간이 예측 불가능하다. 이는 마이크로초 단위의 정밀한 타이밍이 요구되는 하드웨어 실시간 제어 애플리케이션에는 부적합하게 만든다.26

TSN의 핵심적인 가치 중 하나는 '네트워크 융합(Converged Networks)'을 가능하게 한다는 점이다. TSN을 통해 하나의 물리적 이더넷 네트워크 위에서 시간에 극도로 민감한 제어 데이터(OT, Operational Technology)와 일반적인 최선 노력형 데이터(IT, Information Technology)가 서로 간섭 없이 공존할 수 있다.28 이는 케이블링을 단순화하고, 시스템의 복잡성과 비용을 줄이며, 네트워크 관리를 일원화하는 등 상당한 이점을 제공한다.

기술 문서들은 주로 지연 시간과 지터 감소와 같은 기술적 측면에 초점을 맞추지만 24, 자동차나 항공우주와 같은 산업에서 TSN 도입의 전략적 동인은 SWaP(크기, 무게, 전력) 감소와 복잡성 완화라는 경제적 이점에 있다. 과거에는 비행 제어, 인포테인먼트, 진단 등 기능별로 물리적으로 분리된 여러 네트워크를 사용해야 했다.27 TSN은 이러한 이종 네트워크들을 단일 이더넷 케이블로 통합할 수 있게 해준다.28 이는 항공우주 분야에서 결정적인 무게 감소, 제조 과정에서 중요한 와이어링 하네스 단순화, 그리고 통합된 관리 접근법을 통한 운영 효율성 향상으로 이어진다.31 여기서 결정론은 목표를 달성하기 위한 *수단*이며, 네트워크 융합은 그로 인해 얻어지는 실질적인 *성과*이다.

#### 1.2.2  TSN 툴박스: 핵심 IEEE 802.1 표준의 기술적 분석

TSN은 단일 표준이 아니라, 상호 보완적인 표준들의 '툴박스(toolbox)'로 구성된다.23 이 툴박스의 핵심 구성 요소들은 다음과 같다.

- **시간 동기화 (IEEE 802.1AS):** 이는 TSN의 가장 근본적인 구성 요소이다. TSN 도메인 내의 모든 장치(엔드포인트, 스위치 등)는 마이크로초 이하의 정밀도로 동기화된 공통의 시간을 공유해야 한다.28 IEEE 802.1AS는 정밀 시간 프로토콜(PTP, IEEE 1588)의 특정 프로파일(gPTP)을 정의하며, 동기화된 시계 없이는 어떠한 시간 기반 스케줄링도 불가능하다.26
- **트래픽 셰이핑 - 시간 인식 셰이퍼 (IEEE 802.1Qbv, Time-Aware Shaper):** 하드웨어 실시간 보장을 위한 가장 핵심적인 표준이다. 이 표준은 스위치의 출력 포트에 시간 제어 게이트가 있는 여러 개의 큐를 정의한다. 게이트 제어 목록(GCL, Gate Control List)이라 불리는 스케줄에 따라 반복되는 주기 내에서 특정 시간 슬롯 동안 특정 트래픽 클래스에만 네트워크 매체에 대한 독점적인 접근 권한을 부여한다.28 이를 통해 다른 트래픽의 간섭을 원천적으로 차단하고, 스케줄된 트래픽에 대한 엄격한 결정론적 보장을 제공한다.37
- **트래픽 셰이핑 - 프레임 선점 (IEEE 802.1Qbu & 802.3br, Frame Preemption):** 네트워크에서 큰 크기의 낮은 우선순위 프레임이 전송 중일 때, 작은 크기의 높은 우선순위 프레임이 도착하면 대기해야 하는 '선두 차단(head-of-line blocking)' 문제가 발생할 수 있다. 프레임 선점은 이러한 상황에서 높은 우선순위의 '긴급(express)' 프레임이 낮은 우선순위의 '선점 가능(preemptable)' 프레임 전송을 중단시키고 먼저 지나갈 수 있도록 허용한다. 긴급 프레임 전송이 끝나면 중단되었던 프레임의 나머지 부분이 이어서 전송된다.23 이 기능은 802.1Qbv의 시간 슬롯 사이에 필요한 보호 대역(guard band)의 크기를 최소화하여 네트워크 효율을 높이는 데 기여한다.
- **신뢰성 - 프레임 복제 및 제거 (IEEE 802.1CB, Frame Replication and Elimination for Reliability):** 높은 가용성이 요구되는 안전 필수(safety-critical) 시스템을 위해, 이 표준은 동일한 프레임을 복제하여 네트워크 상의 서로 다른 경로(disjoint paths)로 동시에 전송하는 메커니즘을 제공한다. 수신 측에서는 먼저 도착한 프레임을 사용하고, 나중에 도착하는 중복 프레임은 제거한다. 이를 통해 하나의 경로에 장애가 발생하더라도 통신 중단 없이 원활한 데이터 전송을 보장한다.23
- **시스템 설정 (IEEE 802.1Qcc):** TSN 네트워크의 복잡한 스케줄과 스트림을 설정하기 위한 표준화된 모델을 제공한다. 특히 중앙 집중형 설정 모델(centralized configuration model)을 정의하여, 전체 네트워크의 결정론적 동작을 체계적으로 엔지니어링하고 관리할 수 있는 기반을 마련한다.28

TSN 표준들은 메커니즘을 정의하지만, 그 메커니즘을 어떻게 활용할지에 대한 최적의 방법을 항상 명시하는 것은 아니다. 특히, 모든 중요 트래픽 흐름에 대해 종단 간 지연 시간을 보장하는 네트워크 전반의 스케줄(모든 스위치의 GCL)을 생성하는 것은 매우 복잡한 최적화 문제이다.36 이는 시스템 전체의 트래픽 패턴에 대한 깊은 이해와 전문적인 도구를 요구한다.23 이러한 복잡성은 TSN 도입이 단순한 하드웨어 교체를 넘어, 네트워크 엔지니어링 및 관리 패러다임의 전환을 의미함을 시사한다. 바로 이 지점에서 DDS와 같은 상위 레벨 추상화 계층의 가치가 극대화된다.

| **표 1: 주요 TSN 표준과 그 기능** |
| --------------------------------- |
| **표준**                          |
| IEEE 802.1AS-2020                 |
| IEEE 802.1Qbv                     |
| IEEE 802.1Qbu/802.3br             |
| IEEE 802.1CB                      |
| IEEE 802.1Qcc                     |

#### 1.2.3  TSN 배포를 위한 인프라 및 하드웨어 전제 조건

TSN은 단순한 소프트웨어 업그레이드가 아니다. 이 기술의 잠재력을 완전히 활용하기 위해서는 TSN 표준을 지원하는 특수 하드웨어가 필수적이다. 여기에는 IEEE 802.1AS, 802.1Qbv와 같은 핵심 표준을 하드웨어 수준에서 처리할 수 있는 이더넷 스위치와 네트워크 인터페이스 카드(NIC)가 포함된다.33

또한, 네트워크 토폴로지와 설정은 매우 신중하게 설계되어야 한다. 이는 단순히 장치들을 연결하는 것을 넘어, 네트워크 관리 엔진(NME, Network Management Engine) 또는 중앙 네트워크 컨트롤러(CNC, Centralized Network Controller)와 같은 도구를 사용하여 전체 네트워크의 트래픽 스케줄을 계산하고, 각 스위치의 게이트 제어 목록(GCL)을 설정하는 복잡한 엔지니어링 과정을 포함한다.23 이는 TSN이 기존의 플러그 앤 플레이 방식의 이더넷과는 근본적으로 다른 접근 방식을 요구함을 의미한다.

------

## 2.  DDS over TSN의 아키텍처 시너지

이 장에서는 DDS와 TSN이 어떻게 결합하여 완전한 종단 간 결정론적 통신 스택을 형성하는지, 그리고 이들의 통합이 어떻게 이루어지는지를 심층적으로 분석한다.

### 2.1  종단 간 실시간 성능을 위한 융합 프레임워크

#### 2.1.1  DDS-TSN 스택: 애플리케이션 계층 요구사항이 네트워크 계층 동작을 주도하는 방식

애플리케이션/미들웨어 계층에서 동작하는 DDS와 네트워크/데이터 링크 계층에서 동작하는 TSN은 서로의 약점을 보완하고 강점을 극대화하는 자연스럽고 이상적인 조합이다.41 DDS는 통신할 *무엇*(데이터 모델과 QoS 요구사항)을 정의하고, TSN은 이를 *어떻게* 보장할 것인지(결정론적 네트워크 전송)에 대한 해답을 제공한다.

DDS 단독으로는 표준 네트워크 위에서 `DEADLINE`이나 `LATENCY_BUDGET`과 같은 시간 기반 QoS 정책을 완벽하게 보장할 수 없다. 왜냐하면 DDS는 하위 네트워크 계층을 직접 제어하지 못하기 때문이다.39 TSN은 바로 이 누락된 조각을 제공한다. TSN을 통해 DDS는 단순히 QoS 위반을 *감시*하는 수준을 넘어, 이를 적극적으로 *강제*할 수 있게 된다. 예를 들어, DDS의 `LATENCY_BUDGET` QoS 정책은 TSN의 802.1Qbv 스케줄러를 설정하여 해당 데이터 스트림이 실제로 지정된 시간 내에 전달되도록 보장하는 근거로 사용될 수 있다.42

이 조합의 가장 큰 이점 중 하나는 DDS가 TSN 설정의 복잡성을 추상화할 수 있다는 점이다. 애플리케이션 개발자는 "이 센서 데이터는 5ms 내에 도착해야 한다"와 같이 상위 수준의 QoS 요구사항만 명시하면 된다. 그러면 잘 설계된 DDS-TSN 미들웨어는 이 요구사항을 TSN 네트워크를 설정하는 데 필요한 하위 수준의 파라미터(대역폭 예약, 802.1Qbv 시간 슬롯 스케줄링, VLAN 우선순위 태깅 등)로 자동 변환할 수 있다.39 이는 복잡한 분산 시스템의 개발을 극적으로 단순화시키는 강력한 장점이다.39

#### 2.1.2  시너지 표준화: OMG "DDS for TSN" (DDS-TSN) 명세 분석

이러한 강력한 시너지를 인식한 OMG는 "DDS Extensions for Time Sensitive Networking"(DDS-TSN)이라는 공식 표준을 발표했다.41 2023년에 베타 버전이 공개된 이 표준은 DDS 개념을 TSN 개념에 매핑하는 표준화된 방법을 정의함으로써, 두 기술의 통합을 위한 공식적인 청사진을 제공한다.41

이 표준의 핵심적인 매핑 규칙은 다음과 같다:

- DDS의 데이터 흐름(Topic)을 TSN의 스트림(Stream)에 매핑한다.
- DDS의 `TRANSPORT_PRIORITY` QoS 정책을 이더넷 프레임의 우선순위 수준(예: VLAN 태그의 PCP 필드)에 매핑하여 트래픽을 구별한다.
- DDS의 `DEADLINE` 및 `LATENCY_BUDGET` QoS 정책을 TSN의 시간 인식 셰이퍼(802.1Qbv)와 같은 스케줄러를 설정하는 입력으로 사용한다.
- DDS의 신뢰성 메커니즘(예: `RELIABLE` QoS)을 TSN의 프레임 복제 및 제거(802.1CB)와 같은 하드웨어 수준의 신뢰성 메커니즘과 통합한다.

이 표준의 존재는 기술 생태계의 성숙과 상호 운용성에 매우 중요하다. 이는 각기 다른 DDS 벤더들이 TSN 네트워크와 일관된 방식으로 상호작용하도록 보장하여, 특정 벤더에 종속되는 파편화된 솔루션의 난립을 방지한다.41 공식 표준의 등장은 기술 준비도(TRL)가 한 단계 상승했음을 의미하며, 특히 보수적인 항공우주와 같은 산업 분야에 이 기술이 장기적으로 안정적인 솔루션이라는 강력한 신호를 보낸다. 이는 단순한 잠재적 시너지를 넘어, 표준화되고 상호 운용 가능한 아키텍처로의 결정적인 전환을 의미한다.

#### 2.1.3  실제 구현: 네트워크 설정, 관리 및 성능 검증

성공적인 DDS-TSN 구현은 세 가지 주요 단계를 포괄해야 한다 47:

1. **설계 단계(Design-Time):** 시스템 모델을 확장하여 네트워크 토폴로지, 데이터 흐름의 중요도 및 주기성과 같은 TSN 관련 속성을 포함시켜야 한다.
2. **실행 단계(Run-Time):** DDS 구현체는 TSN 네트워크와 상호작용하는 방법을 알아야 한다. 경우에 따라서는 표준 운영체제의 네트워크 스택 일부를 우회하여 2계층에 직접 접근해야 할 수도 있다.47
3. **배포 단계(Deployment-Time):** 설계 단계에서 정의된 시스템 요구사항에 따라 TSN 지원 스위치와 NIC를 실제로 설정하는 도구와 메커니즘이 필요하다.

설정된 네트워크가 실제로 요구되는 타이밍 제약을 만족하는지 검증하는 것은 매우 중요한 과제이다. 이는 최악의 경우 분석(worst-case analysis) 도구와 네트워크 시뮬레이션 및 테스트의 조합을 필요로 한다.43 단순한 패킷 생성기로는 불충분하며, 테스트 도구는 시간 인식이 가능해야 하고, 나노초 단위의 정밀하고 반복 가능한 패킷 주입 및 측정 기능을 갖추어야 한다.48

NXP의 ROS 2 데모와 같은 실제 시연은 이러한 통합이 실용적으로 달성 가능함을 보여준다. 이 데모에서는 DDS XML 프로파일을 사용하여 특정 DDS 트래픽을 QoS 우선순위가 할당된 VLAN에 바인딩하고, 표준 리눅스 명령어(`tc`)를 사용하여 네트워크 인터페이스를 설정함으로써 실제 하드웨어에서 개념을 증명했다.49

DDS-TSN 스택의 진정한 가치는 단일 데이터 중심 모델로부터 시스템 전반의 종단 간 결정론을 관리할 수 있는 능력에 있다. 이는 실시간 시스템 설계의 패러다임 전환을 의미한다. 과거에는 애플리케이션 팀이 처리 마감 시간을, 네트워크 팀이 네트워크 지연 시간을 각기 다른 도구와 지표로 관리했다. DDS-TSN은 이를 통합한다. 애플리케이션 개발자는 DDS 모델에 "이 센서 데이터는 10ms 주기로 1ms 미만의 지연 시간 내에 필요하다"와 같이 데이터 요구사항과 타이밍 제약을 선언하기만 하면 된다.42 그러면 DDS-TSN 인프라가 애플리케이션 스케줄링부터 네트워크 패킷 스케줄링에 이르는 전체 체인을 조율하여 이 단일 종단 간 요구사항을 충족시키는 책임을 진다. 이러한 총체적인 접근 방식은 설계를 단순화하고 통합 오류를 줄이며, 복잡한 시스템을 더욱 이해하고 관리하기 쉽게 만든다.

------

## 3.  응용 분야 분석 및 준비도 평가

이 장에서는 사용자의 질문에 직접적으로 답하기 위해, 로봇과 우주항공 각 분야에 대한 DDS와 TSN의 적용 가능성을 구체적인 사례와 함께 분석하고 기술 준비도(TRL)를 평가한다.

### 3.1  도메인 포커스: 로봇 및 자율 시스템

#### 3.1.1  ROS 2 통신 백본: DDS가 기본 미들웨어인 이유

ROS 2가 DDS를 기본 통신 미들웨어로 채택한 것은 ROS 1의 근본적인 한계를 극복하기 위한 전략적 결정이었다. ROS 1은 통신을 관리하는 중앙 마스터 노드(`roscore`)에 의존하여 단일 장애점(single point of failure) 문제를 안고 있었고, 자체 개발한 TCPROS/UDPROS 프로토콜은 실시간성과 높은 신뢰성이 요구되는 상용 애플리케이션에 적합하지 않았다.7

DDS는 ROS 2에 탈중앙화된 P2P(peer-to-peer) 아키텍처, 동적 참여자 발견, 그리고 세밀한 QoS 제어 기능을 제공함으로써 이러한 문제들을 해결했다.7 이로 인해 ROS 2는 단일 로봇 시스템을 넘어 다중 로봇 시스템이나 산업용 등급의 애플리케이션에 훨씬 더 적합한 프레임워크가 되었다. 또한, ROS 2는 `rmw`(ROS Middleware)라는 추상화 계층을 통해 특정 DDS 벤더에 종속되지 않도록 설계되었다. 개발자는 필요에 따라 eProsima Fast DDS(기본값), RTI Connext DDS, Eclipse Cyclone DDS 등 다양한 구현체를 선택하여 사용할 수 있다.7

#### 3.1.2  사례 연구: 결정론적 제어 루프를 위한 TSN 기반 ROS 2 성능 향상

고속 모션 제어나 협동 로봇의 정밀 작업과 같은 많은 로봇 애플리케이션은 결정론적이고 낮은 지연 시간을 갖는 통신을 필요로 한다. 표준 ROS 2가 사용하는 UDP/IP 통신은 운영체제 네트워크 스택의 혼잡이나 다른 프로세스의 간섭으로 인해 비결정적인 지연을 겪을 수 있다.50

NXP가 공개한 `dds-tsn` 데모는 TSN의 가치를 명확하게 보여주는 강력한 사례 연구이다.49 이 데모는 Gazebo 시뮬레이터에서 ROS 2로 제어되는 차량을 보여준다. TSN 기능이 비활성화된 상태에서 네트워크에 간섭 트래픽을 가하면, 차량 제어 명령이 지연되어 결국 장애물과 충돌한다. 하지만 동일한 간섭 조건에서 TSN의 우선순위 기반 스케줄링(Priority-Based Scheduling) 기능을 활성화하면, 중요한 제어 트래픽이 우선적으로 처리되어 차량이 충돌 없이 성공적으로 임무를 완수한다. 이는 실제 로봇 시나리오에서 TSN이 어떻게 통신의 신뢰성을 보장하는지에 대한 구체적인 증거를 제공한다.

궁극적인 결정론을 추구하기 위해 일부 기업들은 소프트웨어를 넘어 하드웨어 가속으로 나아가고 있다. Acceleration Robotics사의 ROBOTCORE는 ROS 2의 핵심 계층과 DDS/RTPS 프로토콜을 FPGA(Field-Programmable Gate Array) 하드웨어로 직접 구현하여, 소프트웨어로 인한 지터를 원천적으로 제거하고 마이크로초 수준의 통신 지연 시간을 달성했다.40 이는 고성능 로봇 기술의 최전선을 보여주는 사례이다.

#### 3.1.3  배포 시나리오: 산업 자동화에서 이동 로봇까지

DDS와 TSN 기술 스택은 다양한 로봇 분야에서 활용될 수 있다.

- **산업 자동화:** TSN은 공장 내의 파편화된 네트워크를 통합하고, 산업용 로봇과 생산 설비의 정밀한 실시간 제어를 가능하게 하는 핵심 기술로 주목받고 있다.30
- **자율주행차:** DDS와 TSN은 ADAS(첨단 운전자 보조 시스템) 및 완전 자율주행을 위한 핵심 기술이다. 방대한 양의 센서 데이터를 수집하고, 이를 기반으로 한 제어 명령을 보장된 낮은 지연 시간 내에 처리하고 전달해야 하기 때문이다.40
- **이동 로봇 (AMR/Cobot):** 유선 TSN이 공장 내 고정 설비에 적합하다면, 자율 이동 로봇(AMR)이나 협동 로봇(Cobot)은 이동성을 위해 무선 통신을 필요로 한다. 무선 TSN(Wireless TSN)이 이에 대한 해답을 제시하지만, 무선 통신 고유의 문제점(예측 불가능한 채널 상태, 전파 간섭 등)으로 인해 유선 TSN에 비해 표준화 및 구현체의 성숙도가 아직 낮은 편이다.52

#### 3.1.4  로봇 분야 준비도 판정: TRL 및 생산 적용 가능성 평가

- **DDS:** TRL 9. 완전히 성숙했으며 생산 시스템에서 검증된 기술이다. ROS 2의 표준 통신 방식으로 즉시 적용 가능하다.
- **유선 TSN:** TRL 7-8. 표준들은 대부분 성숙 단계에 있으며, 다수의 벤더가 TSN 지원 스위치와 NIC를 상용화했다. 테스트베드와 초기 배포 사례를 통해 성능이 입증되었다. 하지만 아직 범용 기술은 아니며, 네트워크 설계 및 관리에 상당한 전문 지식을 요구한다.
- **DDS over Wired TSN:** TRL 6-7. OMG 표준이 명확한 통합 경로를 제공하며, 관련 데모와 벤더 제품이 존재한다. 적절한 전문성을 갖춘 팀이 주도하는 신규 프로젝트에 도입할 준비가 되어 있다. 새로운 기술 도입 곡선의 선두에 서고자 하는 조직에게는 '즉시 적용 가능'하다고 볼 수 있다.
- **무선 TSN:** TRL 4-5. 아직 개발 단계에 가깝다. 관련 표준이 등장하고 있지만, 실제와 같이 노이즈가 많은 RF 환경에서의 상호 운용성과 성능 보장은 여전히 중요한 과제로 남아있다. 미션 크리티컬한 로봇 애플리케이션에 즉시 배포하기에는 아직 이르다.

### 3.2  도메인 포커스: 우주항공 및 국방 시스템

#### 3.2.1  타협 없는 항공전자 시스템의 요구사항: 안전성, 보안성, 그리고 인증 가능성

우주항공 시스템, 특히 항공전자(avionics) 시스템은 극도로 엄격한 요구사항을 만족해야 한다.

- **안전 최우선:** 항공전자 소프트웨어는 RTCA DO-178C와 같은 엄격한 안전 표준에 따라 인증을 받아야 한다.54 인증의 강도는 시스템 고장이 초래할 수 있는 결과의 심각성에 따라 결정되는 설계 보증 수준(DAL, Design Assurance Level)에 따라 달라지며, 가장 심각한 '재앙적(Catastrophic)' 수준인 DAL A부터 '사소함(Minor)' 수준인 DAL D까지 5단계로 나뉜다.54
- **보안성:** 국방 시스템에서는 데이터의 기밀성, 무결성, 가용성이 매우 중요하다. DDS는 인증, 접근 제어, 암호화를 포괄하는 강력하고 표준화된 보안 프레임워크(DDS Security)를 제공하여 이러한 요구사항을 충족시킨다.5
- **MOSA(모듈식 개방형 시스템 접근법) 의무화:** 현대 국방 프로그램은 비용 절감, 신기술의 신속한 도입, 그리고 특정 벤더에 대한 종속성 탈피를 위해 MOSA를 적극적으로 채택하고 있다. DDS와 TSN과 같은 개방형 표준은 이 전략의 핵심이다.21

#### 3.2.2  인증 시스템 내의 DDS: FACE™ 표준과 DO-178C의 역할

- **FACE™ 컨소시엄:** FACE(Future Airborne Capability Environment)는 군용 항공전자 소프트웨어의 이식성과 재사용성을 높이기 위해 표준 소프트웨어 아키텍처를 정의하는 정부-산업 컨소시엄이다.21
- **FACE 아키텍처 내의 DDS:** FACE 기술 표준은 컴포넌트 간 통신을 위한 전송 서비스 세그먼트(TSS, Transport Services Segment)를 정의하는데, DDS는 이 TSS를 구현하는 핵심 기술로 채택되었다. RTI와 같은 벤더들은 FACE 적합성 인증을 획득한 TSS 제품을 제공한다.21
- **DO-178C 인증 증거:** DO-178C 인증을 획득하는 과정은 시간과 비용이 많이 소요되는 매우 힘든 작업이다.58 DDS 도입의 가장 큰 장점 중 하나는 RTI와 같은 벤더들이 상용 기성품(COTS) DDS 구현체와 함께 방대한 DO-178C DAL A 인증 증거(certification evidence) 패키지를 제공한다는 점이다.21 이는 주 계약업체가 시스템을 인증받는 데 필요한 비용, 시간, 그리고 위험을 극적으로 줄여준다.

#### 3.2.3  차세대 항공전자 버스로서의 TSN: 기능 및 통합 경로

- **레거시 버스 대체:** TSN은 대역폭과 유연성에 한계가 있는 MIL-STD-1553과 같은 기존의 독점적인 항공전자 데이터버스를 대체할 현대적인 이더넷 기반 솔루션으로 자리매김하고 있다.60
- **우주항공 분야를 위한 핵심 TSN 기능:**
  - **결정론 (802.1Qbv):** 비행 제어와 같은 비행 필수(flight-critical) 기능에 필수적이다.
  - **이중화 (802.1CB):** 높은 DAL 시스템의 내고장성 요구사항을 만족시키기 위해 반드시 필요하다.
  - **네트워크 융합:** 비행 필수, 임무 필수, 그리고 정비 데이터를 단일 공통 인프라에서 공유하게 함으로써 항공기의 SWaP를 줄이는 데 기여한다.61
- **NASA의 도입 사례:** NASA는 아르테미스 임무의 발사 통제 시스템(LCS)과 같은 핵심 시스템에 DDS를 적극적으로 활용하고 있다.19 또한 NASA의 여러 로봇 플랫폼에서도 DDS가 데이터 전송 미들웨어로 사용되고 있다.63 NASA와 유럽우주국(ESA)과 같은 주요 우주 기관들은 TSN 기술에 대해서도 깊은 관심을 보이며 도입을 검토하고 있어, 이 기술에 대한 높은 신뢰도를 방증한다.64

#### 3.2.4  우주항공 분야 준비도 판정: 인증 및 임무 필수 시스템 배포 경로

- **인증 증거를 갖춘 DDS:** TRL 9. 수백 개의 임무 필수 항공우주 및 국방 프로그램에서 검증되었다.19 인증 산출물이 포함된 COTS 솔루션 덕분에 DAL A 시스템에도 즉시 적용 가능하다.
- **우주항공용 TSN:** TRL 6-7. 기술 자체는 매우 적합하지만, TSN 기반 시스템을 높은 DAL 등급으로 *인증*받기 위한 경로는 아직 개척 중이다. 이는 프로토콜 자체뿐만 아니라 네트워크 설정 및 분석에 사용되는 전체 툴체인에 대한 인증을 포함한다.
- **인증 시스템용 DDS over TSN:** TRL 5-6. 이는 명백한 미래 방향이지만, 오늘 당장 시작하는 DAL A 프로그램에 상당한 위험 부담 없이 '즉시 적용'하기는 어렵다. OMG DDS-TSN 표준의 제정은 중요한 진전이지만, 이 통합 스택 전체에 대한 인증 산출물은 아직 성숙하지 않았다. 따라서 도입은 단계적으로 이루어질 가능성이 높다. 지상 시스템이나 시뮬레이션 환경, 또는 낮은 DAL의 항공 시스템에 먼저 적용하여 경험을 쌓고 위험을 줄인 후, 점차 비행 필수 시스템으로 확대하는 전략이 유효하다.

| **표 2: DDS/TSN 기능과 DO-178C 설계 보증 수준(DAL) 매핑** |
| --------------------------------------------------------- |
| **DAL / 고장 조건**                                       |
| **A (재앙적)**                                            |
| **B (위험)**                                              |
| **C (주요)**                                              |
| **D (사소)**                                              |

이 표는 DDS와 TSN의 특정 기능이 어떻게 항공우주 분야의 핵심 과제인 안전 인증 요구사항을 만족시키는 데 직접적으로 기여하는지를 보여준다. 이는 기술 선택을 FAA나 EASA와 같은 인증 기관에 정당화하는 데 도움이 되는 분석적 도구이다. "이 기술들이 좋다"는 막연한 주장을 넘어, "이 기술들의 특정 기능을 사용하여 어떻게 안전하고 인증 가능한 시스템을 구축할 것인가"라는 항공우주 프로그램의 핵심 질문에 대한 구체적인 답변을 제시한다.

------

## 4.  전략적 평가 및 최종 권고

이 마지막 장에서는 앞선 모든 분석을 종합하여 각 분야에 대한 전략적 결론과 실행 가능한 로드맵을 제시한다.

### 4.1  구현 환경 탐색: 성숙도, 과제 및 비용

#### 4.1.1  상호 운용성 과제: 표준 및 멀티 벤더 생태계의 현황

- **DDS:** 성숙한 DDSI-RTPS 표준 덕분에 매우 높은 상호 운용성을 자랑한다.
- **TSN:** 이는 여전히 중요한 과제이다. 표준이 존재함에도 불구하고, 벤더별로 미묘하게 다른 해석으로 인해 초기 구현체들 사이에서 상호 운용성 문제가 발생할 수 있다.66 산업별 프로파일(예: 산업 자동화용 IEC/IEEE 60802) 제정과 플러그페스트(Plugfest)와 같은 상호 운용성 테스트 행사가 이러한 문제를 해결하는 데 결정적인 역할을 한다.31 진정한 멀티 벤더 상호 운용성은 개선되고 있지만, 모든 기능에 대해 아직 완벽하게 보장되지는 않는다.
- **레거시 시스템 통합:** TSN은 기존 산업용 프로토콜(예: PROFINET, OPC UA)을 대체하는 것이 아니라, 이들이 TSN 백본 위에서 동작할 수 있도록 하여 성능을 향상시키는 방식으로 설계되었다.67 기존 필드버스에서 전환하기 위해서는 게이트웨이의 역할이 중요할 것이다.68

TSN의 '준비도'는 단일 TRL 값으로 평가하기보다는, 어떤 '툴박스'의 기능이 필요한지에 따라 달라지는 스펙트럼으로 이해해야 한다. TSN은 단일 기술이 아니다.29 기본적인 시간 동기화(802.1AS)와 우선순위 큐잉은 매우 성숙한(TRL 8-9) 기술이다. 시간 인식 셰이퍼(802.1Qbv) 역시 잘 정립되어 있다(TRL 7-8). 하지만 비동기 트래픽 셰이핑(802.1Qcr)이나 복잡한 중앙 집중형 설정 모델(802.1Qcc)과 같은 더 새롭거나 진보된 기능들은 상대적으로 성숙도가 낮다(TRL 5-6). 따라서 시간 동기화와 우선순위 지정만 필요한 애플리케이션은 오늘날 TSN을 바로 적용할 수 있다. 반면, 복잡하고 동적인 네트워크 전반의 스케줄링이 필요한 애플리케이션은 더 높은 구현 위험을 감수해야 한다. 전략적 접근은 애플리케이션의 

*실제* 요구사항을 이를 충족시키는 데 필요한 특정 TSN 표준들의 성숙도와 일치시키는 것이다.

#### 4.1.2  결정론의 경제학: DDS-TSN 도입의 비용-편익 분석

- **하드웨어 비용:** TSN 지원 스위치와 NIC는 일반 기업용이나 소비자용 이더넷 하드웨어보다 가격이 높다.70 하지만 기술 채택이 확산됨에 따라 이 비용은 점차 감소할 것으로 예상된다.28 또한, 여러 개의 분리된 네트워크를 하나로 통합함으로써 얻는 비용 절감 효과가 개별 부품의 높은 가격을 상쇄할 수 있다.
- **소프트웨어 및 라이선스 비용:** RTI Connext와 같은 상용 DDS 구현체는 라이선스 비용이 발생하며, 특히 인증 패키지가 포함될 경우 상당한 비용이 들 수 있다. Fast DDS와 같은 오픈소스 대안은 이러한 비용은 없지만, 특정 애플리케이션에 필요한 기술 지원이나 고급 기능이 부족할 수 있다.7
- **개발 및 통합 비용:** 가장 큰 비용은 종종 인적 자원에서 발생한다. 복잡한 DDS-TSN 시스템을 설계, 설정, 그리고 검증하는 데는 고도로 전문화된 엔지니어링 기술이 필요하다.39 학습 곡선 또한 가파르다.

#### 4.1.3  인적 요소: 요구되는 기술 역량, 툴링 및 통합 모범 사례

- **새로운 기술 역량:** 성공적인 도입을 위해서는 애플리케이션 개발 능력뿐만 아니라, 실시간 시스템 이론, 네트워크 엔지니어링, 그리고 DDS 및 TSN 표준에 대한 깊이 있는 지식을 갖춘 팀이 필요하다.
- **고급 툴링:** 시스템 모델링, 네트워크 설정(CNC/CUC), 그리고 검증/분석을 위한 성숙한 툴체인이 성공의 관건이다.23
- **모범 사례:** 명확한 데이터 흐름과 실시간 요구사항 정의로 프로젝트를 시작해야 한다. 설계 초기 단계부터 시뮬레이션과 최악의 경우 분석을 활용해야 한다. 가능한 경우, 산업별 프로파일과 벤더의 전문 지식을 적극적으로 활용하는 것이 좋다.

DDS-TSN의 도입은 기술 역량의 융합을 촉진하며, 소프트웨어 애플리케이션 개발자와 네트워크 엔지니어 사이의 전통적인 장벽을 허물게 될 것이다. 과거에는 이 두 역할이 명확히 구분되었다. 그러나 DDS-TSN 환경에서는 애플리케이션의 QoS 요구사항(소프트웨어)이 네트워크의 하드웨어 스케줄링(하드웨어)을 직접적으로 결정한다.39 소프트웨어 개발자가 

`DEADLINE` QoS 파라미터를 변경하는 것이 전체 네트워크 스케줄을 망가뜨릴 수 있고, 네트워크 엔지니어가 시간 슬롯을 잘못 설정하면 애플리케이션의 실패를 유발할 수 있다. 이러한 긴밀한 결합은 미래의 시스템 엔지니어가 애플리케이션 로직부터 네트워크 패킷 타이밍에 이르기까지 전체 스택에 대한 총체적인 이해를 갖추어야 함을 의미한다. 이는 채용, 교육, 그리고 팀 구성 방식에 심오한 영향을 미칠 것이다.

### 4.2  결론 및 전략적 로드맵: 지금이 도입 적기인가?

#### 4.2.1  로봇 분야에 대한 종합적 결론 및 최종 판정

**판정:** DDS는 이미 확립된 현재이며, DDS-TSN 스택은 매우 가까운 미래이다. 생산을 목표로 하는 모든 신규 로봇 프로젝트에 있어 DDS는 기본 선택지이다. 특히 산업 제어나 다중 로봇 협응과 같이 높은 성능과 결정론이 요구되는 프로젝트의 경우, 유선 TSN 백본을 평가하고 통합하는 것은 건전한 전략적 결정이다. 단, 이를 위한 엔지니어링 전문성이 확보되어야 한다. ROS 2, 벤더 지원, 공개 데모 등 관련 생태계는 지금 당장 개발을 시작하기에 충분히 성숙해 있다.

#### 4.2.2  우주항공 분야에 대한 종합적 결론 및 최종 판정

**판정:** DDS는 입증되고 인증된 기술로서, 신규 프로그램에 있어 위험도가 낮은 선택이다. TSN은 레거시 버스를 대체할 명백한 기술적 후계자이며, 그 도입은 '만약'의 문제가 아니라 '언제'의 문제이다. 하지만 비행 필수(DAL A/B) 시스템에 완전한 DDS-TSN 스택을 '즉시 적용'하는 것은 아직 시기상조이다. 통합 스택에 대한 인증 경로가 아직 성숙하는 과정에 있기 때문이다. 즉각적인 기회는 지상 시스템, 시뮬레이션, 그리고 낮은 DAL 등급의 애플리케이션에 있다. 이러한 분야에서 경험을 쌓고 기술을 검증하는 것이 향후 더 중요한 시스템으로의 도입 위험을 줄이는 현명한 경로가 될 것이다.

#### 4.2.3  단계적 도입, 위험 완화, 그리고 미래 대비를 위한 실행 가능한 권고

- **전문성 투자:** 소프트웨어와 네트워크 엔지니어링 기술이 융합된 하이브리드 인력의 양성 또는 채용에 지금부터 투자해야 한다.
- **선도 프로젝트로 시작:** 비생산 환경이나 덜 치명적인 프로젝트에 DDS-TSN을 먼저 배포하라. 새로운 테스트 셀, 지상 관제소, 또는 시뮬레이션 환경에 적용하여 실질적인 경험을 축적하는 것이 중요하다.
- **벤더 및 컨소시엄과 협력:** DDS 및 TSN 하드웨어 벤더와 긴밀히 협력하고, OMG, Avnu Alliance, FACE 컨소시엄과 같은 산업 그룹에 참여하여 최신 표준과 모범 사례에 대한 정보를 지속적으로 습득해야 한다.
- **추상화를 고려한 설계:** 시스템 아키텍처를 DDS 데이터 모델과 QoS 중심으로 설계하라. 이는 하위의 네트워크 기술이 표준 이더넷에서 TSN으로, 나아가 미래의 무선 TSN으로 진화하더라도 핵심 애플리케이션 로직은 안정적이고 이식 가능하게 유지하는 최선의 방법이다. 이것이 바로 미래 변화에 유연하게 대응할 수 있는 시스템을 구축하는 궁극적인 길이다.

#### **참고 자료**

1. Architecting of Avionics Full Duplex Ethernet (AFDX ... - IJECT, accessed July 13, 2025, https://www.iject.org/vol4/spl4/c0140.pdf
2. The Evolution of Avionics Networks From ARINC 429 to ... - CiteSeerX, accessed July 13, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=829a3d29ca6cadb114c41962792bc3e1d802ef5f
3. AFDX®: A Time-Deterministic application of ARINC 664 part 7 - Logic Fruit Technologies, accessed July 13, 2025, https://www.logic-fruit.com/blog/arinc/afdx-a-time-deterministic-application-of-arinc-664-part-7/
4. A Brief comparison between mil std 1553 and major commercial data bus systems, accessed July 13, 2025, https://www.mil-1553.com/comparison-mil-std-1553-major-data-bus-systems
5. Mastering AFDX in Avionics Systems - Number Analytics, accessed July 13, 2025, https://www.numberanalytics.com/blog/mastering-afdx-in-avionics-systems
6. Decoding ARINC 429 vs MIL-STD-1553: Key Differences - KIMDU ..., accessed July 13, 2025, https://kimdu.com/decoding-arinc-429-vs-mil-std-1553-key-differences/
7. Why are there so many avionics communications specifications? - Excalibur Systems, accessed July 13, 2025, https://www.mil-1553.com/why-many-avionics-communications-specifications
8. Avionics databus users demand more reliability and flexibility - Military Embedded Systems, accessed July 13, 2025, https://militaryembedded.com/avionics/databus/avionics-databus-users-demand-more-reliability-and-flexibility
9. The Airbus A380 - an AFDX-based flight test computer concept - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/4147050_The_Airbus_A380_-_an_AFDX-based_flight_test_computer_concept
10. Avionics Full-Duplex Switched Ethernet - Wikipedia, accessed July 13, 2025, https://en.wikipedia.org/wiki/Avionics_Full-Duplex_Switched_Ethernet
11. AFDX_Overview | PDF | Computer Network | Network Switch - Scribd, accessed July 13, 2025, https://www.scribd.com/document/865783089/AFDX-Overview
12. Design and Development of AFDX Transmitter Schedular - IJMETMR, accessed July 13, 2025, http://www.ijmetmr.com/oljune2015/SahithiPriya-BSwapnaRani-134.pdf
13. AFDX®/ARINC664P7 Tutorial - What is AFDX - AIM Online, accessed July 13, 2025, https://www.aim-online.com/products-overview/tutorials/afdx-arinc664p7-tutorial/
14. Arinc 664 p7 - AFDX Tutorial & Products - Excalibur Systems, accessed July 13, 2025, https://www.mil-1553.com/arinc-664p7-afdx
15. Testing aircraft AFDX systems - Aim Online, accessed July 13, 2025, https://www.aim-online.com/wp-content/uploads/2019/06/aim-article-afdx-aircraft-testing-12-05-01-u.pdf
16. ARINC 664 P7 - AIRCRAFT DATA NETWORK PART 7 AVIONICS ..., accessed July 13, 2025, https://standards.globalspec.com/std/1283307/arinc-664-p7
17. Testing Ethernet Based Avionics in Aircraft - Mobility Engineering ..., accessed July 13, 2025, https://www.mobilityengineeringtech.com/component/content/article/14453-testing-ethernet-based-avionics-in-aircraft
18. Formal Modeling and Analysis of the AFDX Frame Management Design ∗ - University of Pennsylvania, accessed July 13, 2025, https://repository.upenn.edu/bitstreams/aadea0d6-3ec5-40b4-8686-368e451992d8/download
19. Avionics Databus Tutorials - Astronics, accessed July 13, 2025, https://www.astronics.com/avionics-databus-tutorials
20. (PDF) Avionics data buses: An overview - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/3277898_Avionics_data_buses_An_overview
21. Quantitative Comparison of the Error-Containment Capabilities of a Bus and a Star Topology in CAN Networks - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/224084310_Quantitative_Comparison_of_the_Error-Containment_Capabilities_of_a_Bus_and_a_Star_Topology_in_CAN_Networks
22. A Practical Approach to Commercial Aircraft Data Buses, accessed July 13, 2025, https://www.ddc-web.com/resources/FileManager/dbi/Whitepapers/Commercial_Avionics_Document.pdf
23. Impact Analysis of Flow Shaping in Ethernet-AVB/TSN and AFDX from Network Calculus and Simulation Perspective, accessed July 13, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC5470926/
24. Comparison of Time Sensitive Networking (TSN) and TTEthernet | PDF - Scribd, accessed July 13, 2025, https://www.scribd.com/document/691191734/Comparison-of-Time-Sensitive-Networking-TSN-and-TTEthernet
25. On TTEthernet for Integrated Fault-Tolerant Spacecraft Networks, accessed July 13, 2025, https://ntrs.nasa.gov/api/citations/20150014489/downloads/20150014489.pdf
26. TSN: an Ethernet network, but better for critical embedded systems - IRT Saint Exupéry, accessed July 13, 2025, https://www.irt-saintexupery.com/tsn-an-ethernet-network-but-better-for-critical-embedded-systems/
27. Comparison of ieee AVB and AFDX - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/261208790_Comparison_of_ieee_AVB_and_AFDX
28. 1 Introduction - arXiv, accessed July 13, 2025, https://arxiv.org/html/2312.13969v1
29. Avionics Full Duplex Ethernet and the Time Sensitive ... - IEEE 802, accessed July 13, 2025, https://www.ieee802.org/1/files/public/docs2015/TSN-Schneele-AFDX-0515-v01.pdf
30. Time Sensitive Networking (TSN) for Aerospace & Defense - United Electronic Industries, accessed July 13, 2025, https://www.ueidaq.com/time-sensitive-networking-tsn-for-aerospace-defense
31. Time Sensitive Networking for Aerospace - Design And Reuse, accessed July 13, 2025, https://www.design-reuse.com/articles/55905/time-sensitive-networking-for-aerospace.html
32. A Comprehensive Survey of Wireless Time-Sensitive Networking (TSN): Architecture, Technologies, Applications, and Open Issues - arXiv, accessed July 13, 2025, https://arxiv.org/html/2312.01204v3
33. TSN FOR AEROSPACE - IEEE 802, accessed July 13, 2025, https://www.ieee802.org/1/files/public/docs2024/admin-tsn-aerospace-flyer-1124.pdf
34. Deployment of a Testbed for Validation of TSN Networks in Avionics - MDPI, accessed July 13, 2025, https://www.mdpi.com/2226-4310/12/3/186
35. Avionics Systems Integration & Test - TEST DRAFT - Avionics Interface Technologies - A Teradyne Company, accessed July 13, 2025, https://aviftech.com/systems-integration-test-2/
36. Comac C919 - Wikipedia, accessed July 13, 2025, https://en.wikipedia.org/wiki/Comac_C919
37. An Event-Driven Link-Level Simulator for the Validation of AFDX and Ethernet Avionics Networks - MDPI, accessed July 13, 2025, https://www.mdpi.com/2226-4310/11/4/247
38. (PDF) Verification and Validation Framework for AFDX Avionics Networks - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/363010246_Verification_and_Validation_Framework_for_AFDX_Avionics_Networks
39. Verification and Validation Framework for AFDX Avionics Networks - SciSpace, accessed July 13, 2025, https://scispace.com/pdf/verification-and-validation-framework-for-afdx-avionics-1yrd66tv.pdf
40. The Design and Implementation of the AFDX Network Simulation System - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/224193529_The_Design_and_Implementation_of_the_AFDX_Network_Simulation_System
41. Design and Analysis of AFDX Network Based High-Speed Avionics System of Civil Aircraft, accessed July 13, 2025, https://www.researchgate.net/publication/271972538_Design_and_Analysis_of_AFDX_Network_Based_High-Speed_Avionics_System_of_Civil_Aircraft
42. Temporal Analysis of AFDX Networks Aerospace Engineering - Scholar, accessed July 13, 2025, https://scholar.tecnico.ulisboa.pt/api/records/bNe0fzODMMM6Elraknl1Z9J6ArSqw3V8wYQL/file/dd7e5111b8006399128254dfd827f028bc2a557cdc9b2d6f52f5b84296251918.pdf
43. OMNeT++ Simulation Framework for Avionics Full-Duplex Switched Ethernet | Journal of Aerospace Information Systems - AIAA ARC, accessed July 13, 2025, https://arc.aiaa.org/doi/10.2514/1.I011351
44. IO781: ARINC-664 Part 7 / AFDX protocol support with Simulink - Speedgoat, accessed July 13, 2025, https://www.speedgoat.com/products/communication-protocols-arinc-afdx-io781
45. Research and Analysis on AFDX Network Configuration Based on OMNET++ Simulation, accessed July 13, 2025, https://drpress.org/ojs/index.php/ajst/article/view/31232
46. (PDF) Research and Analysis on AFDX Network Configuration Based on OMNET++ Simulation - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/392943807_Research_and_Analysis_on_AFDX_Network_Configuration_Based_on_OMNET_Simulation
47. Case Study – Modeling Avionics Full-Duplex Switched Ethernet - OMNEST, accessed July 13, 2025, https://omnest.com/casestudy-afdx
48. INVESTIGATING AND ANALYZING JITTER IN CONTROLLER AREA NETWORK - DiVA portal, accessed July 13, 2025, https://www.diva-portal.org/smash/get/diva2:1735813/FULLTEXT01.pdf
49. BusTools/AFDX-A BusTools Software Analyzer | Abaco Systems, accessed July 13, 2025, https://www.abaco.com/products/bt-afdx-bustools-software-analyzer
50. AFDX® / ARINC 664 Part 7 Switch for Flight Test - Data Device Corporation, accessed July 13, 2025, https://www.ddc-web.com/en/discontinued-1/afdx--arinc-664-part-7-switch-for-flight-test
51. AFDX® / ARINC 664 PMC Card - Data Device Corporation, accessed July 13, 2025, https://www.ddc-web.com/en/discontinued-1/dd-82101f
52. TechSAT Integrates TTTech's AFDX® and TTEthernet Network Switches for the Next Generation ADS2 Avionics Test Benches, accessed July 13, 2025, https://www.tttech.com/press/techsat-integrates-tttechs-afdx-and-ttethernet-network-switches-for-the-next-generation-ads2-avionics-test-benches
53. Formal Modeling and Analysis of AFDX Frame Management Design - CiteSeerX, accessed July 13, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=fdcc79dd3d481d2aadd58f33ef9c3a023486b065
54. Formal Modeling and Analysis of the AFDX Frame Management Design. - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/221250136_Formal_Modeling_and_Analysis_of_the_AFDX_Frame_Management_Design
55. Formal Specification and Analysis of AFDX Redundancy Management Algorithms - CiteSeerX, accessed July 13, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=28bbb9d30183ffb3bfec7f31aad129f6d1a55920
56. DO-254 Compliance Guide: Everything Avionics Hardware Teams Need to Know, accessed July 13, 2025, https://www.modernrequirements.com/blogs/do-254/
57. DO-254 Explained: Design Assurance Guidance for Airborne Electronic Hardware, accessed July 13, 2025, https://visuresolutions.com/aerospace-and-defense/do-254/
58. DO-254 Certification Kits for IP Cores - SafeCore Devices, accessed July 13, 2025, https://safecoredevices.com/do-254-ip-cores/do-254-certification-kits/
59. DO-254 IP Cores - SafeCore Devices, accessed July 13, 2025, https://safecoredevices.com/do-254-ip-cores/
60. Best DO-254 Compliance Tools, Checklists & Templates - Visure ..., accessed July 13, 2025, https://visuresolutions.com/aerospace-and-defense/do-254-checklists-and-tools/
61. DO-254 Intro, Compliance: Free Tools/ Papers / Resources - AFuzion, accessed July 13, 2025, https://afuzion.com/do-254-introduction/
62. DO-254 Update: AC/AMC 20-152A (FREE) - Airworthiness Certification Services, accessed July 13, 2025, https://airworthinesscert.com/do-254-do-178c-training-academy/do-254-update-ac-amc-20-152a-free/
63. What is the Impact of AMC 20-152A on Your DO-254 Project?, accessed July 13, 2025, https://patmos-eng.com/impact-of-amc20-152a-on-do254-project/
64. DO-178C Guidance: Introduction to RTCA DO-178 certification | Rapita Systems, accessed July 13, 2025, https://www.rapitasystems.com/do178
65. February 2017/March 2017 - DO-178C: Software for NextGen Avionics, UAVs and More, accessed July 13, 2025, https://interactive.aviationtoday.com/avionicsmagazine/february-2017-march-2017/do-178c-software-for-nextgen-avionics-uavs-and-more/
66. Software Deliverables For DO-178C DAL-B | PDF | Verification And Validation - Scribd, accessed July 13, 2025, https://www.scribd.com/document/597082960/Software-Deliverables-for-DO-178C-DAL-B
67. Deos RTOS Software | Safety Critical Multicore RTOS | DAL A, DO-178C, FACE - DDC-I, accessed July 13, 2025, https://www.ddci.com/solutions/products/deos/
68. CAST-32A Guidance for Multicore Processorss - Green Hills Software, accessed July 13, 2025, https://www.ghs.com/products/safety_critical/integrity_178_standards_cast-32A.html
69. CAST-32A: Development of avionics software for single-core processors, accessed July 13, 2025, https://www.cast32a.com/
70. Certification Authorities Software Team (CAST) : Position Paper CAST-32A | PDF | Multi Core Processor | Computer Engineering - Scribd, accessed July 13, 2025, https://www.scribd.com/document/357177001/cast-32A
71. AMC20-193 - LDRA, accessed July 13, 2025, https://ldra.com/amc20-193/
72. CAST-32A, AC 20-193, & Multi-Core Processing for Avionics - AFuzion, accessed July 13, 2025, https://afuzion.com/cast-32a-ac-20-193-multi-core-processing-for-avionics/
73. What is CAST-32A? - Rapita Systems, accessed July 13, 2025, https://www.rapitasystems.com/cast-32a
74. CAST-32A: Multi-Core ready to become Airborne | SYSGO, accessed July 13, 2025, https://www.sysgo.com/blog/article/cast-32a-multi-core-ready-to-become-airborne
75. Easy Access Rules for Acceptable Means of Compliance for Airworthiness of Products, Parts and Appliances (AMC-20) - initial issue & amendment 1 - EASA, accessed July 13, 2025, https://www.easa.europa.eu/en/document-library/easy-access-rules/online-publications/easy-access-rules-acceptable-means?page=25
76. Software & Airborne Electronic Hardware | Federal Aviation Administration, accessed July 13, 2025, https://www.faa.gov/aircraft/air_cert/design_approvals/air_software/planned
77. Avionics Hardware Development: Introduction to DO-254 and AMC 20-152A | PTC, accessed July 13, 2025, https://www.ptc.com/en/resources/application-lifecycle-management/webcast/avionics-hardware-development-do-254-amc-20-152a
78. AMC20-152A - Patmos Engineering Services, accessed July 13, 2025, https://patmos-eng.com/wp-content/uploads/2020/08/AMC20-152A.pdf
79. Specification and analysis of an extended AFDX with TSN/BLS shapers for mixed-criticality avionics applications - Theses.fr, accessed July 13, 2025, https://theses.fr/2018ESAE0010/document
80. Power of TSN and RTOS integration for aerospace - Industrial Ethernet Book, accessed July 13, 2025, https://iebmedia.com/technology/tsn/power-of-tsn-and-rtos-integration-for-aerospace/
81. Future of Flight - Collins Aerospace, accessed July 13, 2025, https://www.collinsaerospace.com/what-we-do/capabilities/future-of-flight
82. ACARS over IP - Collins Aerospace, accessed July 13, 2025, https://www.collinsaerospace.com/what-we-do/industries/commercial-aviation/connected-cockpit/acars-over-ip
83. NextGen Avionics Roadmap Version 2.0 - DTIC, accessed July 13, 2025, https://apps.dtic.mil/sti/tr/pdf/ADA561244.pdf
84. C&PS Process Roadmap - Honeywell MyAerospace, accessed July 13, 2025, https://aerospace2.honeywell.com/wps/portal/aero/help/roadmap
85. Cybersecurity in Modern Avionics - Number Analytics, accessed July 13, 2025, https://www.numberanalytics.com/blog/cybersecurity-modern-avionics-protecting-aviation
86. NetDiffusion: Network Data Augmentation Through Protocol-Constrained Traffic Generation | Request PDF - ResearchGate, accessed July 13, 2025, https://www.researchgate.net/publication/378376215_NetDiffusion_Network_Data_Augmentation_Through_Protocol-Constrained_Traffic_Generation
87. The NIST Guide to Industrial control systems security - Packt+, accessed July 13, 2025, https://www.packtpub.com/en-BE/product/industrial-cybersecurity-9781788395151/chapter/the-ics-cybersecurity-program-development-process-12/section/the-nist-guide-to-industrial-control-systems-security-ch12lvl1sec62
88. Final Report - IG-17-011 - NASA OIG, accessed July 13, 2025, https://oig.nasa.gov/docs/IG-17-011.pdf
89. SCADA System Cyber Security – A Comparison of ... - SCADAhacker, accessed July 13, 2025, [https://scadahacker.com/library/Documents/Standards/IEEE%20-%20Comparison%20of%20SCADA%20Security%20Standards.pdf](https://scadahacker.com/library/Documents/Standards/IEEE - Comparison of SCADA Security Standards.pdf)
90. Industrial cyber security: how to guarantee security in IIot environments., accessed July 13, 2025, https://magazine.relatech.com/en/industrial-cyber-security-how-to-guarantee-security-in-iiot-environments




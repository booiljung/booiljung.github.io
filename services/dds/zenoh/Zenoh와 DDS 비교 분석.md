# Zenoh와 DDS 비교 분석: 아키텍처 패러다임, 성능 및 최신 분산 시스템을 위한 전략적 함의

## 서론

사물 인터넷(IoT), 로보틱스, 자율주행차와 같은 최신 분산 시스템의 복잡성이 증가함에 따라, 통신 미들웨어의 선택은 시스템 전체의 성공을 좌우하는 핵심 요소가 되었습니다. 이러한 시스템은 "클라우드에서 사물까지의 연속체(cloud-to-thing continuum)" 전반에 걸쳐 작동하며, 전례 없는 수준의 확장성, 성능, 유연성을 요구합니다.1 본 보고서는 기술 리더들이 정보에 입각한 미들웨어 선택을 할 수 있도록, 업계 표준인 데이터 분배 서비스(Data Distribution Service, DDS)와 차세대 프로토콜인 Zenoh를 심층적으로 비교 분석하는 것을 목표로 합니다.

DDS는 객체 관리 그룹(Object Management Group, OMG)에 의해 표준화된 데이터 중심 미들웨어로, 항공우주, 국방, 산업 제어 시스템과 같은 미션 크리티컬 애플리케이션에서 그 성능과 안정성을 입증받았습니다. 이는 수많은 견고한 실시간 시스템의 기반이며, 로봇 운영체제 2(ROS 2)의 기본 미들웨어이기도 합니다.3

반면, Zenoh는 DDS 전문가들이 기존 기술이 현대적이고 동적이며, 자원이 이질적인 환경에 적용될 때 발생하는 한계를 극복하기 위해 설계한 차세대 프로토콜입니다. Zenoh는 이동 중인 데이터(data in motion), 저장된 데이터(data at rest), 그리고 연산(computation)을 위한 추상화를 단일 프로토콜 내에서 통합합니다.8

본 보고서는 두 기술의 근본적인 아키텍처, 정량적 성능, 서비스 품질(QoS), 보안 모델, 생태계를 심층적으로 분석하고, 이를 바탕으로 한 전략적 통합 방안을 제시할 것입니다.

## 1.  기본 아키텍처 및 핵심 철학

이 섹션에서는 각 프로토콜의 근본적인 설계 원칙을 분석합니다. 이러한 원칙은 각 기술의 능력, 한계, 그리고 이상적인 사용 사례를 결정하기 때문입니다.

### 1.1  DDS의 데이터 중심 패러다임: 글로벌 데이터 공간

DDS의 핵심 개념은 애플리케이션이 서로에게 메시지를 보내는 대신, "글로벌 데이터 공간(Global Data Space)"이라는 가상의 공유 정보 공간에서 데이터 객체를 읽고 쓰는 방식으로 상호 작용하는 것입니다. 이 모델은 시간과 공간적으로 애플리케이션을 분리(decoupling)시킵니다.11

DDS 아키텍처는 브로커가 없는(brokerless) P2P(peer-to-peer) 구조를 기반으로 합니다. 참여자(participant)들은 기본적으로 UDP 멀티캐스트를 통해 동적으로 서로를 발견하고 데이터를 직접 교환합니다. 이 설계는 단일 장애점(single point of failure)을 제거하고 로컬 네트워크 내에서 지연 시간을 최소화합니다.4

데이터 구조는 OMG의 인터페이스 정의 언어(Interface Definition Language, IDL)를 사용하여 엄격하게 정의됩니다. IDL 컴파일러는 C++, Java 등 특정 언어에 맞는 코드를 생성하여 유형 안전성(type safety)과 플랫폼 독립적인 상호 운용성을 보장합니다.14 글로벌 데이터 공간 내의 데이터 객체는 `Topic`(이름)과 선택적으로 `Key` 필드(데이터베이스의 기본 키와 유사)로 식별됩니다.12 이 접근 방식은 강력하고 사전에 합의된 데이터 계약을 강제합니다.

### 1.2  Zenoh의 통합된 추상화: 이동, 저장, 연산 데이터

Zenoh의 철학은 세 가지 뚜렷한 데이터 상호 작용 패턴, 즉 발행/구독(Publish/Subscribe, 이동 중인 데이터), 분산 쿼리(Query, 저장된 데이터), 그리고 쿼리가능 객체(Queryable, 연산)를 하나의 간단한 기본 요소 집합으로 통합하는 것입니다.9 이는 개발자들이 통신, 저장, 원격 프로시저 호출(RPC)을 위해 여러 이질적인 기술을 통합해야 하는 부담에서 벗어나게 하는 것을 목표로 합니다.10

Zenoh는 명시적으로 토폴로지에 구애받지 않도록 설계되었습니다. P2P, 브로커(클라이언트-서버), 라우팅(routed) 통신 패턴을 모두 지원하며, 이들을 자유롭게 혼합할 수 있습니다. 이러한 적응성은 리소스가 제한된 장치부터 클라우드에 이르는 시스템에 매우 중요합니다.1 라우팅 인프라는 확장 가능하도록 설계되어 광역 네트워크나 복잡한 네트워크에서 DDS를 제한하는 토폴로지 제약을 제거합니다.8

데이터 모델링 측면에서 Zenoh는 간단한 키-값(key-value) 모델을 사용합니다. 키는 계층적인 문자열(예: `org/building/room/sensor/temp`)이며, 값은 모든 종류의 페이로드일 수 있습니다.22 데이터는 "키 표현식(Key Expression)"이라는 글로브(glob)와 유사한 언어(

`*`, `**`)를 통해 주소 지정되며, 이를 통해 키 공간 전체에 걸쳐 강력하고 유연한 구독 및 쿼리가 가능합니다.23 이 모델은 IDL과 같은 사전 컴파일 단계가 필요 없어 더 큰 동적성을 제공합니다.

### 1.3  분석과 함의: 철학적 차이

두 프로토콜의 핵심적인 차이는 "계약 대 관례(Contract vs. Convention)"로 요약될 수 있습니다. DDS는 IDL을 통해 엄격하고 사전에 정의된 *계약*을 강제하여 유형 안전성과 공식적인 상호 운용성을 우선시합니다. 반면, Zenoh는 키-값 공간을 통해 유연한 *관례*를 장려하여 동적성, 사용 편의성, 아키텍처 적응성을 우선시합니다.

이러한 차이는 각 기술의 탄생 배경에서 비롯됩니다. DDS는 국방, 항공우주와 같이 여러 공급업체가 참여하고 수명 주기가 긴 대규모 시스템에서 탄생했습니다. 이런 환경에서는 상호 운용성과 안정성이 가장 중요하며, IDL과 같은 엄격한 표준 계약은 필수적입니다.4 Zenoh는 DDS를 로보틱스나 IoT와 같은 현대적이고 빠르게 변화하는 영역에 적용할 때 발생하는 문제점에서 탄생했습니다.8 이러한 영역은 이질적인 장치(마이크로컨트롤러부터 클라우드 서버까지), 빠르게 진화하는 요구사항, 예측 불가능한 네트워크 조건을 특징으로 합니다. 유연한 키-값 관례는 타입 정의를 재컴파일하지 않고도 빠른 반복과 새로운 구성 요소의 쉬운 통합을 가능하게 합니다. 결과적으로, DDS는 초기 설계 및 데이터 모델링 단계에 복잡성을 집중시켜 더 견고하지만 덜 유연한 시스템을 만드는 반면, Zenoh는 초기 개발을 단순화하지만 데이터 검증 및 스키마 관리의 복잡성을 애플리케이션 계층으로 옮길 수 있습니다.

더 나아가, Zenoh의 설계는 DDS의 "설계 공간(design space)" 한계에 대한 직접적인 대응입니다. ZettaScale의 CTO인 Angelo Corsaro는 DDS가 원래 의도된 설계 공간, 즉 폐쇄적이고 유선이며 패킷 손실이 적고 강력하며 대칭적인 시스템 내에서는 "최고(rocks)"라고 명시적으로 언급했습니다.8 그러나 DDS를 이 공간 밖, 즉 무선, 리소스 제약, 인터넷 규모 환경으로 옮기면 "복잡해진다(gets complicated)"고 합니다.8 디스커버리 프로토콜이 네트워크 플러딩을 유발할 수 있으며 25, 마이크로컨트롤러용으로 설계되지 않았기 때문입니다. Zenoh는 이러한 한계를 "초월(transcend)"하기 위해 설계되었습니다.8 라우팅 토폴로지 지원, 최소한의 와이어 오버헤드 2, 그리고 네이티브 마이크로컨트롤러 지원(Zenoh-Pico) 27 등은 DDS를 원래 목적 이상으로 확장할 때 발생하는 특정 문제에 대한 직접적인 해결책입니다. 따라서 Zenoh는 단순히 DDS의 대안이 아니라, 동일한 전문가 커뮤니티에 의해 설계된 진화적 후계자로서 더 광범위한 현대적 문제들을 목표로 합니다.

| 특징               | DDS                              | Zenoh                                    | 함의                                                         |
| ------------------ | -------------------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| **핵심 패러다임**  | 데이터 중심 (글로벌 데이터 공간) | 통합 (이동, 저장, 연산 데이터)           | DDS는 데이터 공유에, Zenoh는 통신, 저장, RPC의 통합에 집중합니다. |
| **데이터 모델**    | 구조화된 타입 (Topic/Key)        | 계층적 키-값                             | DDS는 데이터베이스 스키마와 유사하며, Zenoh는 파일 시스템 경로와 유사한 유연성을 제공합니다. |
| **데이터 정의**    | IDL (컴파일 시점 계약)           | 키 표현식 (런타임 관례)                  | DDS는 엄격한 유형 안전성을, Zenoh는 동적성과 개발 용이성을 우선시합니다. |
| **주요 토폴로지**  | P2P (브로커 없음)                | 토폴로지 무관 (P2P, 브로커, 라우팅 혼합) | DDS는 로컬 네트워크에 최적화되어 있으며, Zenoh는 클라우드-엣지 연속체 전반에 적응 가능합니다. |
| **확장성 모델**    | 동적 디스커버리 (로컬)           | 확장 가능한 라우팅 (인터넷 규모)         | Zenoh는 DDS가 어려움을 겪는 WAN 및 복잡한 네트워크 환경을 위해 설계되었습니다. |
| **원래 목표 환경** | 미션 크리티컬 실시간 LAN         | 클라우드-마이크로컨트롤러 연속체         | DDS는 안정적인 환경에, Zenoh는 이질적이고 동적인 환경에 중점을 둡니다. |
| **통합 원칙**      | 데이터 공유 추상화               | 통신, 저장, 연산 추상화 통합             | Zenoh는 단일 기술 스택으로 더 넓은 범위의 시스템 요구사항을 해결하고자 합니다. |

**표 1: 핵심 아키텍처 비교 (Zenoh vs. DDS)**

## 2.  정량적 성능 분석: 처리량, 지연 시간 및 효율성

이 섹션에서는 아키텍처 이론을 넘어 실제 성능을 비교하기 위해 성능 벤치마크 데이터를 비판적으로 검토합니다.

### 2.1  벤치마킹 컨텍스트 및 방법론 종합

가장 포괄적인 데이터는 대만 국립대학교(NTU)의 연구에서 비롯되었으며, 이는 여러 연구 자료에서 널리 인용됩니다.21 NTU 연구는 모든 프로토콜에 대해 동일한 하드웨어(AMD Ryzen 7 5800X, 32GB RAM, 100Gb 이더넷)를 사용하여 소프트웨어 오버헤드와 프로토콜 효율성을 공정하게 비교했습니다.30 Zenoh는 P2P 및 브로커(라우팅) 모드로, DDS는 Eclipse Cyclone DDS를 사용하여 신뢰성 있는(RELIABLE) QoS로 테스트되었습니다.21 특히, NTU 연구는 와이어 프로토콜 자체의 성능을 분리하기 위해 의도적으로 iceoryx와 같은 공유 메모리 전송을 제외하고 네트워크 프로토콜 효율성에 초점을 맞췄습니다.29

### 2.2  부하 시 처리량 (Gbps 및 msg/s)

주요 발견 사항은 Zenoh가 특히 P2P 모드에서 DDS보다 지속적으로 훨씬 높은 처리량을 보인다는 것입니다. 100Gbps 이더넷을 사용한 다중 머신 환경에서 Zenoh P2P는 최대 약 50 Gbps의 처리량을 달성한 반면, Cyclone DDS는 약 14 Gbps에 그쳤습니다. 이는 Zenoh가 3배 이상의 처리량 이점을 가짐을 의미합니다.21 단일 머신(루프백) 환경에서도 Zenoh P2P는 최대 67 Gbps, Cyclone DDS는 최대 약 26 Gbps를 기록하여 Zenoh가 약 2.5배의 이점을 보였습니다.21 작은 페이로드의 메시지 전송률 측면에서도 Zenoh P2P는 4백만 msg/s 이상을 달성했지만 DDS는 2백만 msg/s에 가까웠습니다.21

### 2.3  다양한 네트워크 환경에서의 지연 시간 (마이크로초)

지연 시간 비교는 더 미묘하며 환경에 따라 크게 달라집니다. 단일 머신(루프백) 환경에서는 Cyclone DDS가 약 8µs로 매우 낮은 지연 시간을 보였고, Zenoh P2P는 약 10µs로 약간 더 높았습니다.29 그러나 주목할 점은 마이크로컨트롤러에 최적화된 버전인 

**Zenoh-Pico**가 약 5µs로 가장 낮은 지연 시간을 달성했다는 것입니다. 이는 고급 라우팅 기능을 제외했을 때 Zenoh 코어 프로토콜의 효율성을 보여줍니다.29

다중 머신(100Gbps 이더넷) 환경에서는 Zenoh P2P가 약 16µs의 지연 시간으로 선두를 차지했으며, Cyclone DDS는 약 38µs로 훨씬 높았습니다. Zenoh-Pico는 여기서도 13µs로 가장 좋은 성능을 유지했습니다.30 또한, Wi-Fi나 4G와 같은 불안정한 무선 네트워크 환경에서는 CycloneDDS가 안정적인 이더넷에서 더 나은 성능을 보이지만, Zenoh는 손실이 많은 무선 링크에서 우수한 성능을 보인다는 연구 결과가 있습니다.26

### 2.4  프로토콜 효율성 및 리소스 오버헤드

Zenoh는 최소 4-5바이트의 와이어 오버헤드를 갖도록 설계되었으며, 이는 문자열 키를 정수 ID에 매핑하여 달성됩니다.2 이는 제약된 네트워크에서의 사용을 가능하게 하는 핵심 요소입니다. 반면, DDS의 오버헤드는 RTPS 프로토콜 구조로 인해 일반적으로 더 높습니다. 리소스 사용량 측면에서, 한 연구는 동적 메시 네트워크에서 Zenoh가 DDS에 비해 CPU 사용량이 감소했다고 언급했으며 33, Zenoh-Pico가 수백 바이트의 풋프린트만으로 작동할 수 있다는 사실은 그 효율성을 입증합니다.19 공유 메모리의 경우, DDS는 iceoryx와의 성숙한 통합을 가지고 있으며, Zenoh 역시 단일 호스트에서 대용량 데이터를 지원하기 위해 자체적인 네이티브 공유 메모리 및 제로 카피 메커니즘을 개발했습니다.8

### 2.5  분석과 함의: 숫자를 넘어서

Zenoh의 성능 우위는 단순히 원시적인 속도에 관한 것이 아니라, *아키텍처 효율성*과 *적응성*에 관한 것입니다. DDS는 고성능 프로토콜이지만, Zenoh의 설계 선택(최소 오버헤드, 유연한 토폴로지)은 특히 네트워크 조건이 나빠지거나 복잡해질 때 더 효과적으로 확장할 수 있게 합니다. 지연 시간 결과는 이를 명확히 보여줍니다. DDS는 이상적인 환경, 즉 안정적이고 로컬이며 멀티캐스트 친화적인 네트워크(루프백 테스트)에서 매우 경쟁력이 있습니다.30 그러나 실제 네트워크나 라우터가 도입되는 순간(다중 머신 테스트), Zenoh의 라우팅 및 프로토콜 설계는 상당한 지연 시간 이점을 제공합니다.30 Wi-Fi/4G 테스트는 이 경향을 더욱 확증합니다. 안정적인 LAN에 최적화된 DDS의 메커니즘은 약점이 되는 반면, Zenoh의 설계는 더 견고함이 입증되었습니다.26

또한, Zenoh-Pico의 존재는 리소스가 제한된 장치에 대한 계산법을 근본적으로 바꿉니다. Zenoh-Pico가 모든 테스트 구성 중 가장 낮은 지연 시간을 달성했다는 사실은 29, 복잡한 기능을 제외한 핵심 Zenoh 프로토콜이 표준 DDS 와이어 프로토콜(RTPS)보다도 예외적으로 가볍고 효율적이라는 것을 보여줍니다. 이는 두 생태계가 "엣지"에 접근하는 방식의 철학적 차이를 드러냅니다. DDS는 엣지 장치를 브리징하기 위해 DDS-XRCE라는 확장 기능을 추가하는 반면, Zenoh의 핵심 프로토콜은 네이티브하게 축소되도록 설계되었습니다. 이는 가장 작은 마이크로컨트롤러부터 클라우드까지 원활한 통신 패브릭을 요구하는 시스템에 Zenoh가 더 통합되고 성능이 뛰어난 솔루션을 제공할 수 있음을 시사합니다.

| 메트릭 (다중 머신, 100Gbps ETH)   | Zenoh P2P    | Cyclone DDS  | 이점       |
| --------------------------------- | ------------ | ------------ | ---------- |
| **최대 처리량 (16KB 페이로드)**   | ~50 Gbps 21  | ~14 Gbps 21  | Zenoh > 3x |
| **메시지 전송률 (작은 페이로드)** | >4M msg/s 21 | ~2M msg/s 21 | Zenoh > 2x |

**표 2: 처리량 벤치마크 요약**

| 시나리오                     | Zenoh-Pico | Cyclone DDS | Zenoh P2P |
| ---------------------------- | ---------- | ----------- | --------- |
| **단일 머신 지연 시간 (µs)** | 5 30       | 8 30        | 10 30     |
| **다중 머신 지연 시간 (µs)** | 13 30      | 38 30       | 16 30     |

**표 3: 지연 시간 벤치마크 요약**

## 3.  서비스 품질(QoS) 및 데이터 전달 보장

이 섹션에서는 DDS의 복잡하고 포괄적인 QoS 모델과 Zenoh의 단순하고 직접적인 접근 방식을 대조합니다.

### 3.1  DDS QoS 프레임워크 해부

OMG DDS 표준은 통신의 거의 모든 측면을 세밀하게 제어할 수 있는 22개의 고유한 QoS 정책을 명시합니다.11 그러나 정책의 수와 그들 간의 복잡한 상호 작용은 개발자에게 상당한 어려움을 줄 수 있습니다.34 예를 들어, `RELIABILITY` QoS만으로는 TCP와 같은 신뢰성을 보장하지 않으며, `HISTORY` QoS 정책과 결합해야 합니다.34 기능적으로 그룹화하여 분석하면, `DEADLINE`이나 `TRANSPORT_PRIORITY`와 같은 시간적 속성 정책은 종종 네트워크 동작을 보장하기보다는 "힌트"나 로컬 감시 장치에 불과합니다.34 데이터 전달 정책인 `RELIABILITY`와 `HISTORY`는 밀접하게 연결되어 있으며, `OWNERSHIP`은 수신자 측에서 데이터를 필터링하므로 대역폭 절약 효과가 없습니다.34

### 3.2  Zenoh의 단순하고 직접적인 QoS 프리미티브

Zenoh는 의도적으로 훨씬 간단한 API를 제공하며, 이는 기능 부족으로 오인될 수 있지만 복잡성을 줄이기 위한 의도적인 설계 선택입니다.34 Zenoh는 더 작고 조합 가능한 기본 요소 집합으로 유사한 결과를 달성합니다. 예를 들어, 신뢰성은 간단한 플래그로 요청할 수 있습니다.29 Zenoh는 신뢰성과 혼잡 제어를 분리하여, 개발자가 신뢰성 있는 프로토콜을 선택하면서도 부하 상태에서 샘플을 차단하거나 삭제하는 혼잡 제어 알고리즘을 명시적으로 지정할 수 있게 합니다.29 이는 `RELIABILITY`와 `HISTORY`의 복잡한 상호 작용에 의존하는 DDS와 대조됩니다. 또한, Zenoh의 우선순위는 종단 간에 존중되지만, DDS의 `TRANSPORT_PRIORITY`는 보장되지 않는 힌트일 뿐입니다.34

### 3.3  분석과 함의: 제어 대 복잡성

DDS는 높은 복잡성과 가파른 학습 곡선을 대가로 철저하고 세밀한 제어 기능을 제공합니다. 반면, Zenoh는 단순성과 예측 가능한 동작에 초점을 맞춰 실용적이고 충분한 제어 기능을 제공합니다. DDS의 22개 QoS 정책은 모든 가능한 상황을 미들웨어 계층에서 구성해야 했던 시스템의 유산을 반영합니다.4 이는 전문가에게는 강력한 도구이지만, 경험이 부족한 개발자에게는 예기치 않은 동작을 유발하는 잘못된 구성의 위험을 수반합니다. Zenoh의 접근 방식은 개발자의 "인지 부하"를 줄이고 시스템 동작을 더 쉽게 추론할 수 있도록 합니다.

또한, Zenoh는 특정 QoS와 유사한 기능을 더 효과적인 아키텍처 계층에서 구현합니다. DDS의 `OWNERSHIP`이나 `TIME_BASED_FILTER`는 수신 측에서 필터링이 발생하여 네트워크 대역폭을 낭비합니다.34 Zenoh는 지능형 라우팅 인프라를 통해 네트워크 내부(라우터)나 소스에서 필터링을 수행하여 네트워크 리소스를 절약할 수 있습니다.34 마찬가지로, DDS의 `DURABILITY`는 강력하지만 복잡한 반면, Zenoh는 이를 네이티브 분산 쿼리 지원으로 해결하여 더 일반적이고 유연한 메커니즘을 제공합니다.34 이는 Zenoh의 아키텍처가 발행/구독을 라우팅 및 쿼리와 통합함으로써 QoS 관련 기능을 더 효율적이고 개념적으로 깔끔하게 구현할 수 있음을 보여줍니다.

| DDS QoS 정책              | DDS 범위             | Zenoh 동등 기능                    | Zenoh 범위           | 전문가 논평                                                  |
| ------------------------- | -------------------- | ---------------------------------- | -------------------- | ------------------------------------------------------------ |
| **Reliability + History** | Reader/Writer        | Reliability + Congestion Control   | Sample/Publisher     | Zenoh는 혼잡 제어를 명시적으로 분리하여 예측 가능성을 높입니다. |
| **Transport Priority**    | Writer/Sample        | Priority Class                     | Sample/Publisher     | Zenoh의 우선순위는 종단 간 보장되지만, DDS는 힌트일 뿐입니다. |
| **Partition**             | Publisher/Subscriber | Key Expression                     | Publisher/Subscriber | Zenoh의 키 표현식은 더 유연하고 동적인 데이터 범위 제어를 제공합니다. |
| **Deadline**              | Writer/Reader        | -                                  | API 기능             | Zenoh는 이를 API 수준의 기능으로 간주하며, Rust와 같은 언어에서는 덜 관련됩니다. |
| **Time Based Filter**     | Reader               | Interceptor (속도 제한)            | Peer/Router          | Zenoh는 소스나 라우터에서 필터링하여 네트워크 대역폭을 절약합니다. |
| **Durability**            | Reader/Writer        | Distributed Queries                | 시스템 전반          | Zenoh의 쿼리 기능은 DDS의 내구성보다 훨씬 일반적이고 강력합니다. |
| **Ownership**             | Writer/Reader        | Group Management + Leader Election | 라이브러리 확장      | Zenoh는 수신 측 필터링 대신 그룹 관리 및 리더 선출로 이를 해결합니다. |

**표 4: DDS QoS와 Zenoh 프리미티브 매핑**

## 4.  보안 모델: 인증, 무결성 및 기밀성

### 4.1  DDS 보안 표준

OMG DDS-Security 표준(DDS-Security v1.2)은 플러그형 서비스 제공자 인터페이스(SPI) 아키텍처를 기반으로 인증, 접근 제어, 암호화를 지원하는 포괄적이고 표준화된 상호 운용 가능한 보안 프레임워크를 제공합니다.14

### 4.2  Zenoh의 보안 프레임워크

Zenoh는 상호 TLS(mTLS)를 통한 인증 및 보안 채널, 플러그형 인증 메커니즘, 세분화된 접근 제어 목록(ACL)을 통한 권한 부여 등 현대적인 표준 관행을 사용합니다.1 또한 Rust로 구현되어 메모리 안전성을 기본적으로 확보하는 것이 중요한 보안적 이점입니다.1

### 4.3  비판적 분석: 종단 간 암호화 대 홉 간 암호화

핵심적인 차이점은 Zenoh의 기본 암호화가 홉 간(hop-by-hop) 방식이라는 것입니다. 데이터는 각 라우터에서 복호화된 후 다시 암호화됩니다.1 반면, DDS-Security 모델은 페이로드를 불투명한 덩어리로 취급할 수 있어 종단 간 암호화(End-to-End Encryption, E2EE)를 지원할 수 있습니다.

### 4.4  분석과 함의: 기능성과 보안의 트레이드오프

Zenoh가 네이티브 E2EE를 지원하지 않는 것은 강력한 라우팅 및 쿼리 기능의 직접적인 아키텍처적 결과입니다. 지능형 라우팅이나 네트워크 내 쿼리를 수행하려면 라우터가 암호화되지 않은 데이터나 최소한 키/메타데이터에 접근해야 합니다.8 따라서 홉 간 암호화는 이 아키텍처와 자연스럽게 일치하는 보안 모델입니다. 진정한 E2EE는 라우터의 눈을 가려 이러한 고급 기능을 비활성화시킬 것입니다.

이는 중요한 트레이드오프를 만듭니다. 시스템이 신뢰할 수 있는 단일 보안 도메인 내에서 작동하고 지능형 라우팅이 필요하다면(예: 단일 차량 내부) Zenoh의 모델은 효율적입니다. 그러나 시스템이 신뢰할 수 없는 네트워크를 통과하거나 여러 보안 도메인을 포함한다면(예: 제3자 클라우드를 통한 V2X 통신), E2EE의 부재는 애플리케이션 수준의 암호화를 필요로 하는 심각한 보안 문제가 될 수 있습니다. 더 단순한 전송 역할을 하는 DDS는 제로 트러스트 E2EE 모델을 더 자연스럽게 수용합니다.

| 보안 기능         | DDS              | Zenoh                      | 핵심 함의                                                    |
| ----------------- | ---------------- | -------------------------- | ------------------------------------------------------------ |
| **표준화**        | OMG DDS-Security | 표준 기술 활용 (mTLS, ACL) | DDS는 공식 표준을, Zenoh는 널리 쓰이는 사실상의 표준을 따릅니다. |
| **인증**          | 플러그형 SPI     | 플러그형, mTLS 기본        | 두 시스템 모두 강력한 인증을 지원합니다.                     |
| **접근 제어**     | 플러그형 SPI     | 세분화된 ACL               | 두 시스템 모두 세밀한 권한 제어가 가능합니다.                |
| **암호화 모델**   | E2EE 지원 가능   | 홉 간 암호화 (기본)        | Zenoh의 라우팅 기능은 E2EE와 상충됩니다. 신뢰 경계가 중요합니다. |
| **메모리 안전성** | 구현에 따라 다름 | Rust로 구현 (안전)         | Zenoh는 언어 수준에서 특정 종류의 취약점에 대한 방어력을 가집니다. |

**표 5: 보안 기능 비교**

## 5.  생태계, 채택 및 도메인별 적합성

### 5.1  현존하는 표준, DDS

DDS는 2004년부터 시작된 오랜 역사를 가지고 있으며 4, 국방, 항공우주, 산업 자동화, 의료, 에너지 등에서 깊이 채택되었습니다.4 ROS 2의 기본 미들웨어(예: Cyclone DDS, Fast DDS)로서의 지위는 그 영향력을 보여줍니다.7 RTI, eProsima, ADLINK/OpenSplice와 같은 여러 성숙한 상용 및 오픈 소스 공급업체가 존재합니다.37

### 5.2  떠오르는 도전자, Zenoh

Zenoh는 최근이지만 빠른 채택률을 보이고 있습니다. 주요 성과로는 공식적인 ROS 2 대체 미들웨어로 선정된 것 43, GM의 uProtocol에 채택된 것 8, 그리고 인디 자율주행 챌린지에서 사용된 것 10 등이 있습니다. Zenoh는 DDS R&D에서 분사한 ZettaScale Technology의 지원을 받고 있습니다.8

### 5.3  애플리케이션 도메인 분석

- **로보틱스 (ROS 2):** DDS가 기본이지만, `rmw_zenoh`는 무선/대규모 시나리오에서 DDS의 문제를 해결하기 위해 주목받고 있습니다.45 Zenoh-DDS 브릿지는 여기서 핵심적인 역할을 합니다.47
- **자동차:** DDS가 사용되고 있지만, Zenoh는 현대적인 구역(zonal) 아키텍처와 V2X를 위한 우수한 솔루션으로 자리매김하고 있습니다. Zenoh의 유연성과 라우팅이 핵심 장점입니다.1
- **리소스 제약 IoT:** DDS-XRCE는 에이전트를 사용하여 제약된 장치를 DDS 도메인에 브리징하는 프록시 역할을 합니다.14 반면, Zenoh-Pico는 마이크로컨트롤러에서 직접 실행되는 네이티브 경량 구현으로, 더 원활하고 에이전트 없는 통합을 제공합니다.10

### 5.4  분석과 함의: 공존과 혁신

Zenoh의 전략은 DDS에 대한 정면 공격이 아니라, DDS의 약점(무선, 확장성, 제약된 장치)을 공략하면서 거대한 기존 설치 기반과의 공존을 위한 브릿지를 제공하는 "집게 운동(pincer movement)"입니다. Zenoh는 DDS가 어려움을 겪는 사용 사례를 목표로 합니다.8 ROS 2의 *대체* 미들웨어로 선정된 것은 완벽한 예시입니다. 이는 DDS를 대체하는 것이 아니라, DDS의 한계에 부딪힌 사용자들에게 공인된 고성능 옵션을 제공하는 것입니다.43

이 전략의 가장 중요한 부분은 Zenoh-DDS 브릿지(`zenoh-plugin-dds` 및 `zenoh-plugin-ros2dds`)입니다.47 이를 통해 조직은 기존의 미션 크리티컬한 DDS 인프라를 유지하면서 Zenoh를 사용하여 WAN을 통해 시스템을 연결하거나, 마이크로컨트롤러에 연결하거나, 클라우드 서비스와 통합할 수 있습니다. 이는 채택의 위험을 줄여줍니다. 따라서 아키텍트는 이를 "DDS 또는 Zenoh"라는 이분법적 선택으로 볼 것이 아니라, 로컬 실시간 루프에는 DDS를 사용하고, 이러한 루프들을 연결하고 엣지까지 확장하며 클라우드와 통합하는 백본으로 Zenoh를 사용하는 하이브리드 전략을 고려해야 합니다.

## 6.  개발자 경험 및 전략적 통합 경로

### 6.1  API 철학 및 개발 워크플로우

DDS의 공식적이고 IDL 중심적인 다단계 워크플로우(IDL 정의 -> 컴파일 -> 코드 작성) 15와 Zenoh의 더 간단하고 코드 우선적인 API22는 대조적입니다. 이는 DDS의 선행 설계 엄격성과 Zenoh의 빠른 채택 및 사용 편의성 사이의 트레이드오프를 보여줍니다.10 개발자들은 DDS를 복잡하게 느낄 수 있다는 일화적인 증거가 있습니다.46

### 6.2  Zenoh-DDS 브릿지: 현대화를 향한 경로

`zenoh-plugin-dds`와 ROS 2 전용 `zenoh-plugin-ros2dds`는 47 DDS 토픽을 Zenoh 인프라를 통해 투명하게 라우팅하는 기능을 제공합니다. 이를 통해 DDS 애플리케이션은 애플리케이션 코드를 변경하지 않고도 지리적 라우팅, 개선된 디스커버리 확장성, 비-DDS 친화적 네트워크를 통한 통신을 위해 Zenoh를 활용할 수 있습니다.47 ROS 2 플러그인은 서비스 및 액션과 같은 ROS 개념과 더욱 긴밀한 통합을 제공합니다.47

### 6.3  커뮤니티, 벤더 지원 및 장기적 생존 가능성

DDS는 수십 년의 역사와 RTI, eProsima와 같은 주요 업체들의 지원을 받는 다중 벤더 OMG 표준입니다.5 Zenoh는 이클립스 재단 산하의 새로운 오픈 소스 프로젝트로, 주로 ZettaScale에 의해 주도되지만, 빠른 커뮤니티 성장과 상당한 산업 지원을 받고 있습니다.8

## 7. 결론 및 전략적 권장 사항

### 7.1  연구 결과 종합

본 보고서의 분석은 두 기술 간의 핵심적인 트레이드오프를 명확히 합니다.

- **DDS:** 안정적인 실시간 LAN 환경에서 입증된 표준 기술로, IDL을 통한 강력한 데이터 계약을 제공하지만, 복잡하고 동적, 무선 또는 인터넷 규모 시스템에는 덜 적합합니다.
- **Zenoh:** 유연하고 성능이 뛰어나며 모든 토폴로지에 적응 가능합니다. 데이터 패턴을 통합하고 동적/무선 환경 및 제약된 장치에서 탁월한 성능을 보이지만, 홉 간 암호화라는 다른 보안 모델을 가진 새로운 표준입니다.

### 7.2  하이브리드 미래

가장 강력한 결론은 미래가 하이브리드 형태일 가능성이 높다는 것입니다. Zenoh-DDS 브릿지는 조직이 두 세계의 장점을 모두 활용할 수 있게 하는 핵심적인 조력자입니다.

### 7.3  시나리오 기반 권장 매트릭스

다음 표는 아키텍트가 특정 시스템 요구사항에 따라 정보에 입각한 결정을 내릴 수 있도록 실용적인 프레임워크를 제공합니다.

| 사용 사례 / 시스템 특성                            | 권장 기본 미들웨어                      | 정당성 / 주요 고려사항                                       |
| -------------------------------------------------- | --------------------------------------- | ------------------------------------------------------------ |
| **미션 크리티컬 항공/해군 시스템 (유선 LAN)**      | **DDS**                                 | 입증된 안정성, 표준화, 엄격한 실시간 보장. 4                 |
| **대규모 자율주행차 플릿 (V2X, 클라우드)**         | **Zenoh**                               | 인터넷 규모 라우팅, 무선 성능, 토폴로지 유연성. 8            |
| **단일 로봇 온보드 처리 (ROS 2)**                  | **DDS (기본) 또는 Zenoh**               | DDS는 기본이며 안정적임. 무선 통신이 핵심이면 Zenoh가 우수. 26 |
| **Wi-Fi 메시를 통한 스웜 로보틱스**                | **Zenoh**                               | 동적 메시 네트워크 및 불안정한 링크에서 우수한 성능. 8       |
| **클라우드-마이크로컨트롤러 IoT 시스템**           | **Zenoh**                               | Zenoh-Pico를 통한 원활하고 에이전트 없는 통합, 최소 오버헤드. 10 |
| **기존 DDS 시스템의 WAN 확장**                     | **하이브리드 (DDS + Zenoh-DDS 브릿지)** | 기존 자산을 유지하면서 Zenoh의 라우팅 기능을 활용. 코드 변경 불필요. 47 |
| **신뢰할 수 없는 네트워크에서의 엄격한 E2EE 요구** | **DDS (DDS-Security 포함)**             | DDS-Security는 E2EE를 지원하여 종단 간 기밀성을 보장. 1      |

**표 6: 시나리오 기반 권장 매트릭스**

#### **참고 자료**

1. Zenoh Protocol Security Analysis - Census Labs, accessed July 9, 2025, https://census-labs.com/news/2025/03/17/zenoh-protocol-security-analysis/
2. Zenoh - A Protocol That Should be on Your Radar | by Jkel - Medium, accessed July 9, 2025, https://medium.com/@kelj/zenoh-a-protocol-that-should-be-on-your-radar-72befa697411
3. en.wikipedia.org, accessed July 9, 2025, [https://en.wikipedia.org/wiki/Data_Distribution_Service#:~:text=The%20Data%20Distribution%20Service%20(DDS,using%20a%20publish%E2%80%93subscribe%20pattern.](https://en.wikipedia.org/wiki/Data_Distribution_Service#:~:text=The Data Distribution Service (DDS,using a publish–subscribe pattern.)
4. Data Distribution Service - Wikipedia, accessed July 9, 2025, https://en.wikipedia.org/wiki/Data_Distribution_Service
5. The DDS Foundation Announces 20th Anniversary of the DDS Specification, accessed July 9, 2025, https://www.objectmanagementgroup.org/press-room/03-18-24-a/
6. ROS on DDS - ROS2 Design, accessed July 9, 2025, https://design.ros2.org/articles/ros_on_dds.html
7. Fast DDS Documentation, accessed July 9, 2025, https://media.readthedocs.org/pdf/eprosima-fast-rtps/latest/eprosima-fast-rtps.pdf
8. ZettaScale designs Zenoh to transcend DDS for automotive, ROS ..., accessed July 9, 2025, https://www.therobotreport.com/zettascale-designs-zenoh-to-transcend-dds-for-automotive-ros-communications/
9. www.zettascale.tech, accessed July 9, 2025, [https://www.zettascale.tech/zenoh/#:~:text=Zenoh%20is%20a%20pub%2Fsub,any%20of%20the%20mainstream%20stacks.](https://www.zettascale.tech/zenoh/#:~:text=Zenoh is a pub%2Fsub,any of the mainstream stacks.)
10. What is Zenoh? / Zenoh - pub/sub, geo distributed storage, query, accessed July 9, 2025, https://zenoh.io/docs/overview/what-is-zenoh/
11. Data Distribution Service (DDS) | Object Management Group, accessed July 9, 2025, https://www.omg.org/omg-dds-portal/
12. What is DDS? - DDS Foundation, accessed July 9, 2025, https://www.dds-foundation.org/what-is-dds-3/
13. OMG Data-Distribution Service (DDS): Architectural Overview, accessed July 9, 2025, https://archive.ll.mit.edu/HPEC/agendas/proc04/abstracts/pardo-castellote_gerardo.pdf
14. What's in the DDS Standard? - DDS Foundation, accessed July 9, 2025, https://www.dds-foundation.org/omg-dds-standard/
15. Hello World! in more detail - Eclipse Cyclone DDS 0.9.1 documentation, accessed July 9, 2025, https://cyclonedds.io/docs/cyclonedds/0.9.1/GettingStartedGuide/helloworld_indepth.html
16. 3. Data Types - RTI Connext Getting Started documentation, accessed July 9, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/getting_started_guide/csharp/intro_datatypes.html
17. Data-Centric Programming Best Practices: - RTI, accessed July 9, 2025, https://www.rti.com/hubfs/docs/DDS_Best_Practices_WP.pdf
18. Zenoh - ZettaScale Technology, accessed July 9, 2025, https://www.zettascale.tech/zenoh/
19. Zenoh - The Zero Overhead, Pub/Sub, Store, Query, and Compute Protocol., accessed July 9, 2025, https://zenoh.io/
20. Eclipse zenoh: The Edge Data Fabric, accessed July 9, 2025, https://newsroom.eclipse.org/eclipse-newsletter/2021/july/eclipse-zenoh-edge-data-fabric
21. Comparing the Performance of Zenoh, MQTT, Kafka, and DDS ..., accessed July 9, 2025, https://zenoh.io/blog/2023-03-21-zenoh-vs-mqtt-kafka-dds/
22. Your first Zenoh app / Zenoh - pub/sub, geo distributed storage, query, accessed July 9, 2025, https://zenoh.io/docs/getting-started/first-app/
23. Abstractions / Zenoh - pub/sub, geo distributed storage, query, accessed July 9, 2025, https://zenoh.io/docs/manual/abstractions/
24. Key Expressions.md - eclipse-zenoh/roadmap - GitHub, accessed July 9, 2025, [https://github.com/eclipse-zenoh/roadmap/blob/main/rfcs/ALL/Key%20Expressions.md](https://github.com/eclipse-zenoh/roadmap/blob/main/rfcs/ALL/Key Expressions.md)
25. A Performance Study on the Throughput and Latency of Zenoh, MQTT, Kafka, and DDS - arXiv, accessed July 9, 2025, https://arxiv.org/pdf/2303.09419
26. Comparison of Middlewares in Edge-to-Edge and Edge-to-Cloud Communication for Distributed ROS 2 Systems - arXiv, accessed July 9, 2025, http://arxiv.org/pdf/2309.07496
27. Top Most Requested Eclipse Zenoh Features Since Its Beginning - ZettaScale Technology, accessed July 9, 2025, https://www.zettascale.tech/news/top-most-requested-eclipse-zenoh-features-since-its-beginning/
28. Zenoh Performances : r/rust - Reddit, accessed July 9, 2025, https://www.reddit.com/r/rust/comments/123ioy1/zenoh_performances/
29. Zenoh Performance - ROS General - Open Robotics Discourse, accessed July 9, 2025, https://discourse.ros.org/t/zenoh-performance/30494
30. A Performance Study on the Throughput and Latency of Zenoh, MQTT, Kafka, and DDS, accessed July 9, 2025, https://www.alphaxiv.org/overview/2303.09419v1
31. Comparison of Middlewares in Edge-to-Edge and Edge-to-Cloud Communication for Distributed ROS2 Systems | Papers With Code, accessed July 9, 2025, https://paperswithcode.com/paper/comparison-of-dds-mqtt-and-zenoh-in-edge-to
32. Automotive Middleware Performance: Comparison of FastDDS, Zenoh and vSomeIP, accessed July 9, 2025, https://www.researchgate.net/publication/391461415_Automotive_Middleware_Performance_Comparison_of_FastDDS_Zenoh_and_vSomeIP
33. A Performance Study on the Throughput and Latency of Zenoh, MQTT, Kafka, and DDS, accessed July 9, 2025, https://www.researchgate.net/publication/369300462_A_Performance_Study_on_the_Throughput_and_Latency_of_Zenoh_MQTT_Kafka_and_DDS
34. QoS: DDS vs Zenoh - Speaker Deck, accessed July 9, 2025, https://speakerdeck.com/kydos/qos-dds-vs-zenoh
35. Why Choose DDS? - DDS Foundation, accessed July 9, 2025, https://www.dds-foundation.org/why-choose-dds/
36. Explained: Data Distribution Service (DDS) Protocol - Datatechvibe, accessed July 9, 2025, https://datatechvibe.com/data/datatechvibe-explains-data-distribution-service-dds-protocol/
37. DDS Standard - Data Distribution Service for Complex Systems - RTI, accessed July 9, 2025, https://www.rti.com/products/dds-standard
38. eProsima/Fast-DDS: The most complete DDS - Proven: Plenty of success cases. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 9, 2025, https://github.com/eProsima/Fast-DDS
39. Eclipse Cyclone DDS - ROS 2 Documentation, accessed July 9, 2025, https://docs.ros.org/en/foxy/Installation/DDS-Implementations/Working-with-Eclipse-CycloneDDS.html
40. rticommunity/rticonnextdds-usecases: Case + Code, a series of examples illustrating RTI Connext DDS use cases. These DDS use cases help take you from the problem that you have–tracking vehicles, sending real-time data over a WAN or distributing video data to many receivers–to real coding and configuration that can - GitHub, accessed July 9, 2025, https://github.com/rticommunity/rticonnextdds-usecases
41. eProsima Fast DDS, accessed July 9, 2025, https://fast-dds.docs.eprosima.com/
42. Eclipse Cyclone DDS™ | projects.eclipse.org, accessed July 9, 2025, https://projects.eclipse.org/projects/iot.cyclonedds
43. Eclipse Zenoh Selected as the Alternate ROS 2 Middleware - Eclipse News, accessed July 9, 2025, https://newsroom.eclipse.org/eclipse-newsletter/2023/october/eclipse-zenoh-selected-alternate-ros-2-middleware
44. A platform that unifies communication, storage and computation from the cloud to the micro-controller - ZettaScale Technology, accessed July 9, 2025, https://www.zettascale.tech/zetta/
45. Zenoh ROS 2 RMW: A New Middleware Implementation - ROS Developers OpenClass #194, accessed July 9, 2025, https://www.reddit.com/r/ROS/comments/1ett5rm/zenoh_ros_2_rmw_a_new_middleware_implementation/
46. What companies actually use ROS2 in production? : r/ROS - Reddit, accessed July 9, 2025, https://www.reddit.com/r/ROS/comments/1i3jogb/what_companies_actually_use_ros2_in_production/
47. eclipse-zenoh/zenoh-plugin-dds: A zenoh plug-in that ... - GitHub, accessed July 9, 2025, https://github.com/eclipse-zenoh/zenoh-plugin-dds
48. zenoh-bridge-dds - OpenEmbedded Layer Index, accessed July 9, 2025, https://layers.openembedded.org/layerindex/recipe/389394/
49. Connext Nano | RTI Labs, accessed July 9, 2025, https://www.rti.com/developers/rti-labs/connext-nano
50. Announcing the DDS-XRCE Specification: A Protocol for Sensor Networks - RTI, accessed July 9, 2025, https://www.rti.com/blog/announcing-the-dds-xrce-specification-a-protocol-for-sensor-networks
51. Just saw this quote on LinkedIn: "I've been working in robotics for over 10 years now and if there's one thing I've learned it's that robotics companies almost always start out using ROS and then spend the rest of their life trying to get the hell out of ROS". Thoughts? - Reddit, accessed July 9, 2025, https://www.reddit.com/r/robotics/comments/1bkzecp/just_saw_this_quote_on_linkedin_ive_been_working/
[GNSS](./index.md)
# u-blox PointPerfect Flex


u-blox의 PointPerfect Flex는 단순한 GNSS(Global Navigation Satellite System) 보정 서비스를 넘어, 고정밀 위치 결정 기술의 전통적인 비용 및 복잡성 장벽을 허물기 위해 전략적으로 설계된 하나의 완결된 생태계이다. 이 서비스는 유연한 사용량 기반 요금 모델을 개방형 데이터 포맷 지원(RTCM) 및 고효율 독점 기술(SPARTN)과 결합함으로써, IoT, 로보틱스, 정밀 농업 분야에서 센티미터 수준 정확도의 대중화를 공격적으로 목표로 하고 있다.

본 보고서의 핵심 분석 결과는 다음과 같다. PointPerfect Flex의 주요 차별점은 높은 고정 연간 비용에서 사용량을 분리하는 경제적 모델과, 대규모 배포를 간소화하는 '실리콘-투-클라우드(silicon-to-cloud)' 통합에 있다. IP(인터넷 프로토콜)와 L-Band 위성 전송을 모두 지원하여 서비스의 강건성을 보장하며, 'Localized Distribution' 및 'Zero Touch Provisioning (ZTP)'과 같은 기능은 데이터 비용 및 장치군 관리라는 핵심 운영 문제를 직접적으로 해결한다.

전략적으로 PointPerfect Flex는 u-blox를 신흥 자율 애플리케이션 시장의 핵심 조력자로 자리매김하게 하며, 고정밀 시장의 패러다임을 소수의 측량 등급 전문가 영역에서 확장 가능한 대중 시장의 상품으로 전환시키고 있다. 이는 기술적 성능뿐만 아니라 근본적으로 비즈니스 모델과 통합 용이성을 통해 Trimble 및 NovAtel과 같은 기존 강자들과 직접 경쟁하는 구도를 형성한다.



GNSS는 위성 시계 및 궤도 오차, 대기층(전리층 및 대류층) 통과 시 발생하는 신호 지연 등 내재적 오차 요인으로 인해 단독으로 사용할 경우 수 미터 수준의 정확도 한계를 가진다.1 이러한 정확도는 자율 주행이나 정밀 농업과 같이 센티미터 수준의 정밀도를 요구하는 애플리케이션에는 부적합하다. 이 한계를 극복하기 위해 수십 년간 다양한 보강 기술이 개발되어 왔다.


실시간 이동 측위(Real-Time Kinematic, RTK)는 관측 공간 보정(Observation Space Representation, OSR)의 대표적인 기술이다. 이 방식은 로버(이동국) 근처에 위치한 기준국에서 수신한 GNSS 위성 신호의 모든 관측 오차를 하나의 '덩어리(lump sum)'로 계산하여 로버에게 전송한다.2

- **강점**: RTK는 매우 빠른 시간(10-20초) 내에 센티미터 수준(<2 cm)의 높은 정확도를 달성할 수 있다.4
- **한계**: 이 기술은 기준국과의 거리가 약 30-40 km 이내로 제한되며, 높은 데이터 대역폭과 양방향 통신을 요구한다.2 이러한 제약은 넓은 지역에 걸쳐 서비스를 확장하는 것을 어렵고 비용이 많이 들게 만드는 근본적인 원인이 된다.


정밀 단독 측위(Precise Point Positioning, PPP)는 상태 공간 보정(State Space Representation, SSR) 기술을 기반으로 한다. 이 방식은 위성 궤도 및 시계 오차와 같은 GNSS 오차 요인들을 모델링하여 광범위한 지리적 영역에 유효한 보정 정보를 방송한다.2

- **강점**: PPP는 단방향 통신(IP 또는 위성 L-Band)만을 필요로 하므로, 지리적 제약 없이 전 세계적으로 무제한의 사용자에게 서비스를 제공할 수 있어 확장성이 매우 뛰어나다.3
- **한계**: 전통적인 PPP 방식은 수렴 시간이 매우 길어, 수 분에서 최대 30분까지 소요되어야 서브-데시미터(sub-decimeter) 수준의 정확도를 얻을 수 있다.2 이는 드론이나 자율 로봇과 같은 동적인 실시간 애플리케이션에는 부적합하게 만들며, 국지적인 대기 오차를 효과적으로 보정하는 데에도 어려움이 있었다.


PointPerfect Flex의 기술적 근간을 이루는 PPP-RTK는 RTK와 PPP의 장점을 결합한 하이브리드 기술이다. 즉, PPP의 방송 기반 확장성과 RTK의 빠른 수렴 시간 및 높은 정확도를 모두 제공한다.2

PPP-RTK는 PPP보다는 조밀하지만 RTK보다는 성긴(예: 약 150 km 간격) 기준국 네트워크를 사용하여 위성 시계, 궤도, 대기 지연 등 개별 오차 요소를 '상태(state)'로 모델링한다.4 이 SSR 보정 정보가 방송되면, 로버는 수신된 데이터를 사용하여 자신의 위치에 맞는 정밀한 지역 보정치를 계산한다. 이를 통해 PointPerfect Flex는 수 초 내에 3-6 cm 수준의 수평 정확도를 달성하며 6, 기존 두 기술의 주요 단점을 모두 해결한다.

PPP-RTK의 등장은 고정밀 GNSS의 대중화를 가능하게 한 핵심 기술적 변곡점이다. RTK의 주된 비용 및 물류 병목 현상이었던 조밀한 지역 기준국 인프라의 필요성을 제거함으로써 경제적 방정식을 근본적으로 바꾸었기 때문이다. RTK는 기준국 의존성 때문에 고가의 지역화된 애플리케이션에 국한되었고, PPP는 실시간 동적 애플리케이션에 사용하기에는 너무 느렸다. 반면, PPP-RTK는 적당한 밀도의 네트워크를 사용하여 오차 *모델*을 생성하고 이를 방송한다.3 이 모델 기반 접근법(SSR)은 단일 데이터 스트림으로 대륙 전체를 커버할 수 있게 하여, 촘촘한 RTK 기준국 네트워크에 비해 인프라 비용을 획기적으로 절감시킨다. 따라서 PPP-RTK는 PointPerfect Flex와 같이 대륙 단위의 커버리지와 대중 시장 지향적인 가격을 갖춘 서비스를 가능하게 하는 기술적 토대이다.

**표 1: RTK, PPP, PPP-RTK 기술 비교 분석**

| 기능               | 전통적 RTK (OSR)             | 전통적 PPP (SSR)      | PPP-RTK (SSR) / PointPerfect Flex |
| ------------------ | ---------------------------- | --------------------- | --------------------------------- |
| **정확도**         | <2 cm                        | <10 cm                | 3-6 cm 7                          |
| **수렴 시간**      | 10-20초 4                    | 3-30분 2              | 수 초 7                           |
| **커버리지**       | 지역적 (기준국 반경 ~30km) 4 | 전 지구적             | 대륙 단위 4                       |
| **통신 방식**      | 양방향 2                     | 단방향 3              | 단방향 2                          |
| **확장성**         | 낮음                         | 매우 높음             | 매우 높음                         |
| **대역폭 요구**    | 높음 4                       | 낮음 5                | 매우 낮음 (SPARTN) 7              |
| **인프라 밀도**    | 매우 높음 (~10km) 3          | 매우 낮음 (~1000km) 3 | 중간 (~150km) 8                   |
| **주요 사용 사례** | 측량, 건설                   | 정적 측량, 과학 연구  | 자율주행, 로보틱스, 정밀 농업     |


PointPerfect Flex는 최신 PPP-RTK 기술을 기반으로, 다양한 데이터 포맷과 전송 프로토콜을 지원하여 유연성과 강건성을 동시에 확보하는 정교한 아키텍처를 갖추고 있다.


이 서비스는 명시적으로 PPP-RTK GNSS 보정 데이터 서비스로 정의된다.4 지상 기준국 네트워크를 활용하여 위성 및 대기 오차를 모델링하고, 이를 SSR 보정 정보로 변환하여 최종 사용자 장치에 전달한다.1 로버는 이 상태 정보를 수신하여 자신의 위치에 맞는 지역 보정치를 실시간으로 계산함으로써, 수 초 내에 3-6 cm의 수평 정확도를 달성한다.7 이때 사용되는 기준 좌표계는 ITRF2020(current epoch)이다.11


PointPerfect Flex는 기술적으로 우수한 독점 형식과 산업 표준 형식을 모두 지원하는 이중 포맷 전략을 채택하고 있다. 이는 u-blox 생태계 내에서의 성능 이점을 제공하는 동시에, 외부 사용자의 진입 장벽을 낮추는 영리한 접근 방식이다.


SPARTN은 u-blox F9 및 X20 시리즈 수신기에 최적화된 기본 데이터 포맷이다.9 이 포맷은 Sapcorda(현재 u-blox에 인수됨), Bosch, Septentrio 등이 포함된 컨소시엄에 의해 개발된 개방형 표준이다.10

SPARTN의 핵심 특징은 다음과 같다:

- **고효율 압축**: 고도로 압축된 저대역폭 SSR 포맷으로, 일반적인 네트워크 RTK 서비스 대비 데이터 전송률을 최대 90%까지 절감할 수 있다고 주장한다.7 이는 최종 사용자의 모바일 데이터 비용을 직접적으로 절감시키는 중요한 이점이다.
- **강력한 보안**: 종단 간 암호화(AES) 및 인증을 지원하여 보안이 중요한 애플리케이션에 적합하다.14 PointPerfect는 SPARTN 2.0 버전을 사용한다.17


PointPerfect Flex는 산업 표준인 RTCM 포맷(특히 버전 3.3)으로도 보정 정보를 제공한다.11 이는 u-blox가 아닌 타사 GNSS RTK 수신기나 혼합된 하드웨어 장비군과의 호환성을 보장하기 위한 전략적 결정이다.7

RTCM 지원의 의미는 다음과 같다:

- **시장 확대**: 타사 하드웨어를 보유한 고객들이 값비싼 장비 교체 없이 PointPerfect 서비스를 채택할 수 있게 하여 잠재 시장을 크게 넓힌다.10
- **통합 용이성**: RTCM은 널리 지원되는 표준이므로, 기존 시스템에 통합하기 위한 장벽이 낮다. 하지만 SPARTN에 비해 대역폭 효율성은 떨어진다.7

이러한 이중 포맷 전략은 u-blox가 기술적으로 우월하고 자사 실리콘에 최적화된 SPARTN을 통해 생태계 내 사용자에게 최고의 가치를 제공하는 동시에, RTCM이라는 실용적인 '진입로'를 열어두어 경쟁사 하드웨어를 사용하는 고객들까지 유치하려는 '선점 후 확장(land and expand)' 전략의 일환으로 분석된다. 고객은 먼저 서비스의 가격과 유연성을 보고 진입한 후, 장기적으로 하드웨어를 교체할 때 SPARTN의 성능 이점을 고려하여 u-blox 모듈로 전환할 유인을 갖게 된다.



- **NTRIP**: 보정 정보는 인터넷을 통해 NTRIP(Networked Transport of RTCM via Internet Protocol) 프로토콜(버전 1.0 및 2.0 지원)로 전송된다.7 이는 IP 기반 보정 데이터 전송의 표준 방식이다.
- **MQTT**: MQTT(Message Queuing Telemetry Transport)는 주로 SPARTN 전송을 위한 레거시 옵션 및 Thingstream IoT 플랫폼과의 통합을 위해 언급된다.9 IP 기반 전송은 장치가 셀룰러 또는 Wi-Fi 연결을 필요로 한다.


인터넷 연결이 불가능한 원격 지역의 애플리케이션을 위해, 보정 정보는 정지궤도 L-Band 위성을 통해 전송될 수 있다.7

- **전용 수신기 필요**: 이 방식을 사용하려면 주 GNSS 수신기 외에 u-blox NEO-D9S와 같은 전용 L-Band 보정 수신기가 추가로 필요하다.20
- **이중화 및 가용성**: L-Band 전송은 중요한 이중화(redundancy)를 제공하며, 미션 크리티컬 애플리케이션에서 서비스 가용성을 보장하는 핵심 요소이다.9

IP와 L-Band라는 이중 전송 메커니즘은 단순한 기술적 특징을 넘어, 연결성이 불안정하지만 가동 시간이 무엇보다 중요한 자율 농업 및 중장비 자동화와 같은 시장에 대한 핵심적인 판매 포인트이다. 이는 PointPerfect Flex를 단순한 저비용 대안이 아닌, 강건한 전문가급 서비스로 포지셔닝한다.

**표 2: SPARTN 대 RTCM 기술 비교**

| 기능                          | SPARTN 2.0                            | RTCM 3.x                                           |
| ----------------------------- | ------------------------------------- | -------------------------------------------------- |
| **포맷 유형**                 | 상태 공간 보정 (SSR)                  | 주로 관측 공간 보정 (OSR) 기반이나 SSR 메시지 지원 |
| **주요 장점**                 | 고효율 압축, 저대역폭, 보안 기능 내장 | 광범위한 산업 표준, 높은 호환성                    |
| **대역폭 효율성**             | 매우 높음 (RTK 대비 최대 90% 절감) 7  | 상대적으로 낮음                                    |
| **암호화 지원**               | 기본 지원 (AES) 14                    | 프로토콜 수준에서 미지원 (전송 계층 보안 필요)     |
| **최적화된 하드웨어**         | u-blox F9/X20 시리즈 12               | 모든 표준 RTK 수신기                               |
| **PointPerfect 내 주요 용도** | u-blox 생태계 내 성능 및 비용 최적화  | 타사 및 혼합 장비군 호환성 확보 10                 |


PointPerfect Flex는 인상적인 성능 벤치마크와 함께, 대규모 IoT 배포를 염두에 둔 혁신적인 관리 기능들을 Thingstream 플랫폼을 통해 제공한다.


- **정확도**: 호환되는 수신기를 사용하고, 중단 없는 보정 데이터를 수신하며, 위상 모호성이 고정된(ambiguity-fixed) 위치를 획득했을 때, 일반적으로 3-6 cm의 수평 정확도(CEP)를 제공한다.7
- **수렴 시간**: "수 초 내 수렴(convergence in seconds)"은 이 서비스의 핵심 마케팅 포인트로, 전통적인 PPP 방식에 비해 압도적인 장점이다.7
- **가용성**: 인터넷과 L-Band 위성 전송의 이중화를 통해 99.9% 이상의 가동 시간(uptime)을 보장한다.1


이 서비스는 유럽, 미국 본토, 캐나다, 대한민국, 호주 등 대륙 단위의 균일한 커버리지를 제공한다.23 또한 해안선에서 최대 12해리(약 22 km)까지 커버리지가 확장되어 근해 해양 애플리케이션에도 유용하다.4


PointPerfect Flex는 u-blox의 엔터프라이즈급 클라우드 플랫폼인 Thingstream을 통해 제공 및 관리된다.1 Thingstream은 사용자가 직접 장치군을 관리하고, 요금을 처리하며, 이벤트를 모니터링하고, API를 통해 기능을 제어할 수 있는 셀프서비스 환경을 제공한다.4 이는 확장성과 사용 편의성을 위한 핵심 요소이며, 플랫폼 내에서 물리적 장치는 'Thing'이라는 논리적 개체로 표현된다.11


ZTP는 대규모 장치군의 배포를 간소화하기 위해 인증 정보와 설정을 자동으로 프로비저닝하는 기능이다.7 이는 생산 라인에서 각 장치를 수동으로 구성하는 지루하고 비용이 많이 드는 프로세스를 제거하여, 시장 출시 시간을 단축하고 운영 부담을 크게 줄여준다.7 이 기능은 u-blox가 목표로 하는 대중 시장 IoT 애플리케이션에 필수적이다.


이는 장치가 NTRIP 캐스터에 전송하는 NMEA-GGA 메시지를 기반으로 자신의 대략적인 위치에 따라 지역화된 보정 스트림을 구독하는 혁신적인 기능이다.23 서버는 사용자의 위치와 관련 없는 불필요한 보정 데이터를 필터링하여 전송하므로, 전체 대륙 스트림에 비해 대역폭 요구 사항을 최대 70-80%까지 줄일 수 있다.23 이는 사용자 데이터 비용을 직접적으로 절감하고, 특히 대역폭이 제한된 애플리케이션의 성능을 향상시킨다.23 또한, 이 방식은 암호화가 필요 없어 수신기 측의 처리 부담을 덜어주는 부수적인 이점도 있다.26

핵심 PPP-RTK 기술도 인상적이지만, 대규모 IoT의 성공적인 확장을 위한 진정한 비결은 Thingstream, ZTP, Localized Distribution이라는 플랫폼 수준의 기능 조합에 있다. 이 기능들은 대규모 고정밀 장치를 배포할 때 발생하는 세 가지 가장 큰 운영상의 어려움, 즉 관리, 프로비저닝, 데이터 비용을 체계적으로 해결한다. 하나의 고정밀 장치를 배포하는 것은 쉽지만, 1만 개를 배포하는 것은 운영상의 악몽이다. 전통적인 RTK 서비스는 수동 인증서 입력과 고정된 고비용 데이터 요금제를 수반하여 대규모 확장에 적합하지 않다.9 u-blox는 ZTP로 프로비저닝을 자동화하고 7, Thingstream 플랫폼으로 중앙 집중식 관리를 제공하며 4, Localized Distribution 기능과 유연한 요금제를 결합하여 총소유비용(TCO)을 예측 가능하고 확장 가능하게 만든다.23 따라서 이러한 기능들은 단순한 부가 기능이 아니라, u-blox가 목표 시장에 제공하는 핵심 가치 제안의 일부이다.

**표 3: PointPerfect Flex 공식 서비스 사양**

| 파라미터            | 사양                                                         |
| ------------------- | ------------------------------------------------------------ |
| **기술**            | PPP-RTK (SSR) 4                                              |
| **수평 정확도**     | 3-6 cm (CEP) 7                                               |
| **수렴 시간**       | 수 초 7                                                      |
| **가용성 (Uptime)** | > 99.9% 1                                                    |
| **데이터 포맷**     | SPARTN 2.0, RTCM 3.3 11                                      |
| **전송 프로토콜**   | NTRIP (IP), L-Band 위성 7                                    |
| **기준 좌표계**     | ITRF2020 (current epoch) 11                                  |
| **서비스 커버리지** | 유럽, 미국 본토, 캐나다, 대한민국, 호주 23                   |
| **주요 관리 기능**  | Thingstream 플랫폼, Zero Touch Provisioning, Localized Distribution 7 |


PointPerfect Flex는 u-blox의 '실리콘-투-클라우드' 전략의 핵심으로, 자사의 하드웨어와 긴밀하게 통합되어 최적의 성능을 발휘하도록 설계되었으며, 동시에 개방형 표준 지원을 통해 폭넓은 호환성을 제공한다.


이 서비스는 u-blox의 고정밀 GNSS 모듈에 사전 통합되어 있으며 최적화되어 있다.7 주요 호환 모듈은 다음과 같다:

- **ZED-F9P**: 표준 고정밀 GNSS 모듈 9
- **ZED-F9R**: 관성 측정 장치(IMU)가 통합되어 추측 항법(Dead Reckoning)을 지원하는 모듈 9
- **ZED-F9K**: 자동차용 추측 항법에 특화된 모듈 9
- **ZED-X20P**: 최신 세대의 고정밀 GNSS 모듈 9

이러한 모듈들을 SPARTN 포맷과 함께 사용하면 가장 원활하고 대역폭 효율적인 솔루션을 구성할 수 있다.12


L-Band 위성을 통해 보정 정보를 수신하기 위해서는 전용 보정 데이터 수신기가 필요하다.20 u-blox의 

**NEO-D9S** 모듈이 이 역할을 담당한다. NEO-D9S는 암호화된 L-Band 신호를 수신하여 SPARTN 데이터를 주 GNSS 수신기(예: ZED-F9P)로 전달하고, 주 수신기는 이를 해독하여 위치 계산에 사용한다.9 이 두 개의 칩(예: ZED-F9P + NEO-D9S)으로 구성된 아키텍처는 L-Band를 통한 이중화 통신에 필수적이다.20


PointPerfect Flex의 핵심적인 특징 중 하나는 u-blox 제품이 아닌 수신기와도 작동한다는 점이다.7 이는 표준 

**RTCM** 포맷의 보정 정보를 NTRIP을 통해 제공함으로써 가능하다.10 이 기능 덕분에 다른 공급업체의 기존 하드웨어를 보유하고 있거나 여러 종류의 장비를 혼용하는 기업들도 PointPerfect Flex 서비스를 이용할 수 있어, u-blox에게 상당한 전략적 이점을 제공한다.10


L1 및 L2/L5 주파수를 지원하는 다중 대역 안테나는 필수적이다. L1 전용 안테나를 사용하면 성능이 심각하게 저하된다.11 u-blox는 ANN-MB 시리즈와 같은 안테나를 권장한다.22 L-Band 수신을 위해서는 안테나가 L-Band 주파수(약 1540-1559 MHz)도 지원해야 한다.20 최적의 신호 품질을 위해서는 하늘이 잘 보이는 곳에 간섭 없이 안테나를 배치하고 접지면(ground plane)을 사용하는 것이 매우 중요하다.11


u-center는 u-blox가 제공하는 무료 Windows 소프트웨어로, GNSS 모듈을 구성, 모니터링 및 평가하는 데 사용된다.11 이 소프트웨어는 펌웨어 업데이트(F9 수신기는 PointPerfect 지원을 위해 펌웨어 1.3x 이상 필요), 동적 플랫폼 모델과 같은 파라미터 설정, 평가를 위한 NTRIP 캐스터 연결 등에 활용된다.11

u-blox의 '실리콘-투-클라우드' 전략은 단순히 제품을 판매하는 것을 넘어, 개발 과정의 마찰을 줄이고 전체 가치 사슬을 소유하려는 의도를 보여준다. GNSS 모듈(ZED-F9P), L-Band 수신기(NEO-D9S), 보정 서비스(PointPerfect), 관리 플랫폼(Thingstream)을 모두 제공함으로써 u-blox는 원스톱 솔루션을 구축했다.7 이는 고객의 통합 작업을 단순화하고 개발 시간과 위험을 줄여주며, u-blox에게는 더 많은 가치를 창출한다. 여러 공급업체의 부품과 서비스를 조합하는 방식으로는 달성하기 어려운 성능과 사용 편의성을 제공하는 긴밀하게 통합된 생태계를 만드는 것이다. 이러한 접근 방식은 u-blox를 단순한 부품 공급업체에서 더 높은 가치를 제공하는 서비스 제공업체로 전환시키며, 고객과의 지속적인 순환 수익 관계를 형성한다.

**표 4: PointPerfect Flex 호환 주요 u-blox 모듈**

| 모듈 이름    | 주요 기능                 | 핵심 특징                                        | 대상 애플리케이션                        |
| ------------ | ------------------------- | ------------------------------------------------ | ---------------------------------------- |
| **ZED-F9P**  | 고정밀 GNSS 수신          | 다중 대역 RTK, SPARTN/RTCM 지원                  | UAV, 로보틱스, 정밀 농업 9               |
| **ZED-F9R**  | 고정밀 추측 항법          | 통합 IMU, 센서 퓨전, GNSS 신호 단절 시 위치 유지 | 서비스 로봇, 저속 차량, 중장비 30        |
| **ZED-F9K**  | 자동차용 고정밀 추측 항법 | 자동차 등급, 차선 수준 정확도, ADAS/V2X 지원     | ADAS, 자율주행, 차량 텔레매틱스 9        |
| **ZED-X20P** | 차세대 고정밀 GNSS 수신   | 모든 대역(All-band) 지원, 향상된 성능            | 차세대 자율 시스템, 고성능 로보틱스 9    |
| **NEO-D9S**  | L-Band 보정 데이터 수신   | 위성 기반 보정 수신, ZED-F9P와 연동              | 원격지, 셀룰러 음영 지역 애플리케이션 20 |


PointPerfect Flex의 가장 파괴적인 혁신은 기술 자체보다 비즈니스 모델에 있다. "사용한 만큼만 지불하라"는 핵심 가치 제안은 기존 RTK 공급업체들의 경직되고 높은 연간 구독료 모델에 대한 정면 도전이다.9


- **개별 장치 "Thing" 요금제**: 개별 장치를 위한 월간 요금제이다. 무료 개발자 플랜(40시간)부터 시작하여, 초과 요금이 부과되는 시간제 블록(100H, 200H 등), 그리고 IP 또는 L-Band를 위한 무제한 요금제까지 다양하게 구성되어 있다.29 이 모델은 사용량이 예측 가능하고 일관된 애플리케이션에 이상적이다.
- **"Pooled" 요금제**: 대규모 장치군을 위한 핵심적인 혁신이다. 매월 대량의 시간 블록(예: 35,000시간)을 구매하면, 전체 장치군이 이 시간을 공유하여 사용할 수 있다.28 이는 로봇 잔디깎이(겨울에는 유휴 상태)나 농기계(파종/수확기에만 사용)와 같이 간헐적이거나 계절적인 사용 패턴을 가진 경우에 완벽하다. 모든 자산의 사용량을 평균화하여 장치당 비용을 극적으로 낮출 수 있기 때문이다.7


PointPerfect Flex는 여러 메커니즘을 통해 총소유비용(TCO)을 낮추는 것을 목표로 한다:

1. **구독 비용**: 사용량 기반 요금제는 간헐적 사용 사례의 경우 고정 요금 경쟁사 대비 26-87%의 비용 절감을 제공할 수 있다.28 일반적인 RTK 구독료는 연간 수천 달러에 달할 수 있지만, PointPerfect는 시간당 약 $0.04 수준으로 저렴할 수 있다.28
2. **데이터 전송 비용**: SPARTN 포맷과 Localized Distribution 기능은 주요 운영 비용인 모바일 데이터 소비를 크게 줄여준다.7
3. **하드웨어 비용**: 최적의 성능을 위해 u-blox 모듈이 권장되지만, 이들은 대중 시장을 겨냥한 가격으로 책정되어 전통적인 측량 등급 수신기의 높은 비용을 피할 수 있다.2
4. **운영 비용**: ZTP와 Thingstream 플랫폼은 배포 및 관리에 드는 인건비를 절감한다.7


ROI는 비용(하드웨어, 구독료, 데이터)을 편익과 비교하여 계산된다. 정밀 농업에서 편익은 수확량 증대, 투입 비용(비료, 농약, 연료) 절감, 작물 품질 향상 등을 포함한다.43 연구에 따르면 정밀 농업은 옥수수 수확량을 에이커당 4.3 부셸 증가시킬 수 있으며 43, 농업 기술 기업의 62% 이상이 이러한 이익을 달성하기 위해 GNSS 보정 서비스를 사용한다.45 PointPerfect의 비용 효율적인 모델은 이전에는 고정밀 솔루션의 가격 장벽 때문에 접근이 어려웠던 중소 규모 농가들도 이러한 기술적 혜택을 누릴 수 있게 한다.46

가격 모델은 u-blox의 "대중 시장" 전략의 중심 기둥이다. PPP-RTK 기술이 대륙 단위 서비스를 가능하게 했지만, 대규모 분산 IoT 장치군에 경제적으로 실현 가능하게 만든 것은 바로 유연한 공유 요금제이다. 이는 기술 혁신만큼이나 중요한 비즈니스 모델 혁신이다. 대중 시장 애플리케이션은 극도로 비용에 민감하다. 장치당 높은 고정 연간 요금(예: $1,000/년)은 $2,000짜리 로봇 잔디깎이에는 적용할 수 없는 모델이다.28 "Pooled" 요금제는 이러한 문제를 해결하는 핵심적인 혁신이다. 100대의 잔디깎이를 보유한 조경 회사는 겨울에 대부분의 장비가 유휴 상태임에도 100개의 활성 구독료를 연중 내내 지불할 필요가 없다. 대신, 활성 상태인 잔디깎이가 사용한 총 시간만큼만 비용을 지불한다.28 이는 서비스 비용을 자산의 수익 창출 활동과 직접적으로 연계시킨다. 따라서, 이 유연한 가격 모델은 u-blox가 목표로 하는 전체 유효 시장(TAM)을 열어주는 열쇠이며, 전통적인 RTK보다 훨씬 광범위한 사용 사례에서 ROI 계산을 긍정적으로 만든다.

**표 5: PointPerfect Flex 세부 가격 구조**

| 요금제 이름                                                  | 요금제 유형 | 포함 시간          | 월간 가격 (USD) | 초과 요금 (시간당 USD) | 대상 사용 사례        |      |
| ------------------------------------------------------------ | ----------- | ------------------ | --------------- | ---------------------- | --------------------- | ---- |
| **PP Flex Developer**                                        | Thing       | 40시간             | $0.00           | -                      | 개발 및 평가          |      |
| **PP Flex 100H**                                             | Thing       | 100시간            | $5.90           | $0.059                 | 저사용량 개별 장치    |      |
| **PP Flex 200H**                                             | Thing       | 200시간            | $9.70           | $0.049                 | 중간 사용량 개별 장치 |      |
| **PP Flex 300H**                                             | Thing       | 300시간            | $13.90          | $0.046                 | 중간 사용량 개별 장치 |      |
| **PP Flex 500H**                                             | Thing       | 500시간            | $21.00          | $0.042                 | 고사용량 개별 장치    |      |
| **PP Flex IP**                                               | Thing       | 무제한 (IP)        | $27.00          | -                      | 상시 접속 IP 장치     |      |
| **PP Flex L-band**                                           | Thing       | 무제한 (L-band)    | $35.00          | -                      | 상시 접속 L-band 장치 |      |
| **PP Flex L-band and IP**                                    | Thing       | 무제한 (IP+L-band) | $49.00          | -                      | IP 및 L-band 이중화   |      |
| **PP Flex 35K**                                              | Pooled      | 35,000시간         | $2,550.00       | $0.073                 | 소규모 장치군         |      |
| **PP Flex 100K**                                             | Pooled      | 100,000시간        | $6,500.00       | $0.065                 | 중규모 장치군         |      |
| **PP Flex 350K**                                             | Pooled      | 350,000시간        | $19,600.00      | $0.056                 | 대규모 장치군         |      |
| **PP Flex 650K**                                             | Pooled      | 650,000시간        | $33,800.00      | $0.052                 | 초대규모 장치군       |      |
| **PP Flex 1M**                                               | Pooled      | 1,000,000시간      | $42,000.00      | $0.042                 | 엔터프라이즈급 장치군 |      |
| 참고: L-band 관련 요금제는 별도 문의가 필요할 수 있음. 가격은 Thingstream 포털 기준이며 변경될 수 있음. 29 |             |                    |                 |                        |                       |      |


고정밀 GNSS 보정 서비스 시장은 Trimble과 NovAtel(Hexagon)과 같은 전통적인 강자들이 주도해왔다. u-blox PointPerfect Flex는 이들과 다른 전략적 접근을 통해 시장에 진입하고 있다.


- **서비스 등급**: ViewPoint RTX(약 30cm)부터 CenterPoint RTX(<2 cm)까지 다양한 등급의 PPP 기반 보정 서비스를 제공한다.48
- **성능**: CenterPoint RTX는 'RTX FAST' 지역(북미, 유럽)에서 1분 미만, 전 세계적으로는 15분 미만의 수렴 시간으로 <2 cm의 수평 정확도를 자랑한다.50 ITRF2020 기준 좌표계를 사용한다.52
- **전송 방식**: IP/셀룰러 및 위성 전송을 모두 지원하며 48, RTK 신호 단절 시 위성 PPP 보정으로 전환하는 xFill 기술이 특징이다.50
- **생태계**: Trimble의 광범위한 측량 및 농업용 하드웨어 포트폴리오와 긴밀하게 통합되어 있으며, 독점적인 보정 포맷을 사용한다.53


- **서비스 등급**: TerraStar-C PRO(2.5 cm 정확도, 3분 미만 수렴)와 지역 서비스인 TerraStar-X(2.5 cm 정확도, 1분 미만 수렴) 등 여러 PPP 서비스 레벨을 제공한다.54
- **성능**: TerraStar-X는 "하늘에서 오는 RTK(RTK from the Sky)"라는 슬로건으로 빠른 수렴 시간을 강조하며 57, 99.999%의 가동 시간을 내세운다.54
- **전송 방식**: IP 및 위성 전송을 지원한다.54
- **생태계**: 거대 기업 Hexagon 포트폴리오의 일부로, NovAtel OEM7 수신기와 긴밀하게 통합되어 있다.57


- **u-blox PointPerfect Flex**: 대중 시장 확장성, 통합 용이성, 유연하고 저렴한 가격 모델에 중점을 둔다. 개방형 RTCM 지원과 ZTP는 IoT 장치군에 대한 핵심 차별점이다. 비즈니스 모델과 운영 효율성으로 경쟁한다.
- **Trimble RTX**: 전문 측량 및 농업 시장에서 강력한 입지를 구축하고 있다. 방대한 기존 고급 하드웨어 설치 기반을 활용한다. 확고한 브랜드 신뢰도, 최고 수준의 정확도(<2cm), 전문가 사용자를 위한 xFill과 같은 기능으로 경쟁한다.
- **NovAtel TerraStar**: 특히 농업 및 자율 시스템 분야의 OEM 통합에 강점을 보인다. TerraStar-X의 빠른 수렴 시간과 99.999%라는 높은 신뢰성을 바탕으로 성능으로 경쟁한다.

시장은 두 개의 진영으로 분화되고 있다. 하나는 Trimble과 NovAtel이 주도하는 최고급 전문가 시장이고, 다른 하나는 u-blox가 개척하는 확장 가능한 대중 시장이다. 핵심 기술(PPP-RTK/PPP-AR)은 유사하지만, 비즈니스 전략, 가격 책정, 목표 고객은 뚜렷하게 갈라진다. u-blox는 측량 등급 시장에서 Trimble을 이기려는 것이 아니라, IoT 분야에서 '충분히 좋은(good enough)' 센티미터 수준 정확도를 요구하는 훨씬 더 큰 신규 시장을 창출하고 지배하려 한다. Trimble과 NovAtel의 제품은 최고 수준의 정확도를 요구하며 가격에 덜 민감한 전문가들을 대상으로 한다.48 반면 u-blox의 PointPerfect Flex는 약간 낮은 정확도(3-6 cm)를 제공하지만 7, 훨씬 다른 가격대와 유연하고 확장 가능한 비즈니스 모델을 제시한다.28 ZTP, Pooled 요금제, 혼합 장비군을 위한 RTCM 지원과 같은 기능들은 전통적인 측량 시장이 아닌 대중 시장 IoT의 과제를 해결하기 위해 명시적으로 설계되었다.7 따라서 PointPerfect Flex는 전통적인 고급 시장 아래에 새로운 시장 부문을 개척하고 있다. 이는 성능 면에서는 '충분히 좋지만' 비용과 사용 편의성 면에서 우월한 제품이 훨씬 더 큰 시장을 열 수 있다는 전형적인 '파괴적 혁신' 전략이다.

**표 6: 기능 및 사양 비교: PointPerfect Flex 대 Trimble RTX 대 NovAtel TerraStar**

| 기능                 | u-blox PointPerfect Flex                     | Trimble CenterPoint RTX                | NovAtel TerraStar-X                  |
| -------------------- | -------------------------------------------- | -------------------------------------- | ------------------------------------ |
| **수평 정확도**      | 3-6 cm 7                                     | <2 cm 48                               | 2.5 cm (95%) 54                      |
| **수렴 시간 (최상)** | 수 초 7                                      | <1 분 (RTX FAST 지역) 50               | <1 분 54                             |
| **커버리지**         | 대륙 단위 (북미, 유럽, 한국, 호주 등) 23     | 전 지구적 (FAST 지역 존재) 48          | 지역적 (전 지구적 서비스는 C-PRO) 56 |
| **전송 방식**        | IP (NTRIP), L-Band 위성 7                    | IP/셀룰러, 위성 48                     | 위성 (IP는 C-PRO) 54                 |
| **주요 데이터 포맷** | SPARTN (최적화), RTCM (호환용) 9             | 독점 포맷 53                           | 독점 포맷                            |
| **가격 모델 철학**   | 사용량 기반 (Pay-as-you-go), 공유(Pooled) 28 | 고정 연간 구독 52                      | 고정 기간 구독                       |
| **핵심 차별점**      | 경제성, 확장성, 개방형 표준 지원             | 최고 수준 정확도, xFill, 전문가 생태계 | 빠른 수렴, 높은 신뢰성, OEM 통합     |
| **목표 시장**        | 대중 시장 IoT, 로보틱스, 자율 시스템         | 전문 측량, 건설, 정밀 농업             | OEM, 정밀 농업, 자율 시스템          |


PointPerfect Flex는 그 기술적 특성과 경제적 모델을 바탕으로 다양한 신흥 시장에서 강력한 솔루션으로 부상하고 있다. u-blox는 핵심 PPP-RTK 기술 플랫폼을 기반으로 특정 수직 시장을 겨냥한 맞춤형 서비스(Flex, Live, Global, Safe)를 제공하는 '플랫폼-투-포트폴리오' 전략을 구사하고 있다.




- **과제**: 자동 조향, 파종, 방제와 같은 작업에는 센티미터 수준의 정확도가 필요하지만, 전통적인 RTK는 많은 농가, 특히 계절적으로 장비를 사용하는 농가에게 너무 비싸고 복잡하다.7
- **해결책**: PointPerfect Flex는 필요한 정확도를 제공하면서, 농업 주기에 완벽하게 부합하는 사용량 기반/계절별 요금 모델을 제공한다.7 L-Band 전송은 원격 농지에서의 커버리지를 보장한다.22
- **성과**: 사례 연구에 따르면, u-blox 기술을 사용한 로봇 시스템은 1인치 크기의 작물을 대상으로 97%의 정확도를 달성했으며, 시간당 7에이커를 처리하는 효율성을 보였다.46 비용 효율적인 모델은 중소 규모 농가도 이 기술을 도입하여 수확량을 늘리고 투입 자원을 절약할 수 있게 한다.43






- **과제**: 드론과 서비스 로봇(예: 잔디깎이, 배달 로봇)은 다양한 환경에서 안전하고 효과적으로 작동하기 위해 신뢰할 수 있고 지속적이며 정밀한 항법 정보가 필요하다.7
- **해결책**: PointPerfect Flex는 대륙 단위 커버리지를 제공하여 지역 RTK 네트워크 간의 핸드오프 문제를 제거한다.7 ZED-F9R 모듈과 결합하면 센서 퓨전(IMU + GNSS)을 통한 추측 항법이 가능해져, 나무 아래나 짧은 터널과 같이 일시적으로 GNSS 신호가 끊기는 상황에서도 위치 추정을 유지할 수 있다.31
- **성과**: 대규모 자율 장치군의 강건한 항법을 가능하게 한다. 저대역폭 SPARTN 포맷과 공유 요금제는 이러한 비용에 민감한 애플리케이션에 경제적 실현 가능성을 부여한다.7






- **과제**: ADAS 레벨 3 이상은 안전한 결정을 내리기 위해 차선 수준의 정확도(<10−20 cm)와 높은 무결성 및 기능 안전(ISO 26262)을 요구한다.61
- **해결책**: PointPerfect Flex가 핵심 PPP-RTK 기술을 제공하는 반면, u-blox의 전용 자동차 솔루션은 **PointSafe** 서비스이다. PointSafe는 동일한 PPP-RTK 기술을 기반으로 하지만, ASIL-B(ISO 26262) 인증을 받았으며 안전한 위치 엔진과 무결성 모니터링 기능을 포함한다.61 ZED-F9K는 이 분야를 위해 설계된 자동차 등급의 추측 항법 모듈이다.32
- **성과**: 카메라, 레이더와 같은 다른 센서를 보완하는 절대 위치 기준을 제공하여 차선 식별을 가능하게 한다. PointPerfect/PointSafe는 L3+ 자율 주행 기능 개발의 핵심 조력자 역할을 한다.61

이러한 포트폴리오 전략은 u-blox가 단일 R&D 투자(PPP-RTK 서비스 인프라)를 활용하여, 각기 다른 고객층을 대상으로 맞춤형 기능과 가치 제안을 담은 다양한 제품을 만들어내는 정교한 접근법임을 보여준다. 예를 들어, 대중 시장 IoT를 위해서는 유연한 가격의 **Flex**를 7, 최고 성능과 지역 좌표계를 원하는 전문가를 위해서는 

**Live**를 66, 전 세계적인 커버리지가 필요한 사용자를 위해서는 PPP-AR 기반의 

**Global**을 67, 그리고 안전이 최우선인 자동차 시장을 위해서는 ISO 26262 인증을 받은 

**PointSafe**를 제공하는 식이다.61






자율 시스템과 미션 크리티컬 IoT에서 신뢰는 가장 중요한 자산이다. u-blox는 하드웨어, 전송, 플랫폼 계층에 걸쳐 다층적인 '심층 방어(defense-in-depth)' 전략을 구현하여 PointPerfect 생태계의 보안을 강화하고 있다.






- **SPARTN 포맷**: "Secure Position Augmentation"이라는 이름에서 알 수 있듯이, SPARTN 포맷은 보안을 핵심 원칙으로 설계되었다.14 이 포맷은 종단 간 암호화를 기본적으로 지원하며, 대륙별 L-Band 스트림의 SPARTN 메시지는 CTR 모드의 AES-128을 사용하여 암호화된다.16 수신기는 동적 키를 제공받아 이 스트림을 해독한다.17
- **NTRIP 프로토콜**: 보정 데이터 자체의 암호화 여부(지역화된 스트림은 암호화되지 않음)와 관계없이 17, 통신 채널은 보호될 수 있다. PointPerfect는 NTRIP 연결에 TLS 암호화를 지원하여 인증 정보와 데이터를 도청으로부터 보호한다.12






- **보안 부팅 및 업데이트**: u-blox의 최신 수신기 플랫폼(F9 모듈 등)은 보안 부팅 기능을 통합하여 인증된 펌웨어만 장치에서 실행되도록 보장한다.69 이는 악성 펌웨어 로드를 방지한다.
- **재밍 방지**: u-blox 수신기는 고급 디지털 필터링 기술을 사용하여 재밍 신호의 영향을 감지하고 완화함으로써, 상당한 RF 간섭 환경에서도 위치 고정을 유지할 수 있다.71
- **스푸핑 방지**: 수신기는 여러 위성군과 주파수를 활용하여 단순한 스푸핑 공격을 더 어렵게 만든다. 나아가 최신 모듈은 갈릴레오 OSNMA(Open Service Navigation Message Authentication)를 지원하는데, 이는 위성 신호 자체의 진위 여부를 암호학적으로 검증하는 방법이다.69






- **Zero Touch Provisioning (ZTP)**: ZTP는 근본적으로 보안 기능이다. X.509 인증서 기반 아키텍처를 사용하여 장치를 클라우드 플랫폼(Thingstream)에 안전하고 자동으로 온보딩한다.75 이 프로세스는 u-blox 모듈 내의 하드웨어 신뢰점(Root of Trust)에 기반하여 장치 복제를 방지하고 각 장치에 고유하고 불변의 ID를 보장한다.76 이는 하드코딩된 공유 인증 정보로 장치를 출하하는 불안전한 관행을 방지한다.

이러한 포괄적이고 다층적인 접근 방식은 미션 크리티컬 및 안전 크리티컬 애플리케이션에 대한 보안 요구 사항에 대한 깊은 이해를 보여주며, PointPerfect 생태계를 신뢰할 수 있는 솔루션으로 포지셔닝한다. 자율 주행차나 중요 자산 추적기는 쉽게 재밍되거나 스푸핑될 수 있는 위치 정보에 의존할 수 없으므로, 이러한 보안은 채택의 전제 조건이다.






u-blox PointPerfect Flex는 고정밀 GNSS 시장에 상당한 영향을 미칠 잠재력을 지닌 파괴적인 서비스이다. 그 전략적 가치와 시장에서의 위치를 종합적으로 평가하면 다음과 같다.






- **강점 (Strengths)**: 파괴적이고 유연한 가격 모델; 개발을 간소화하는 긴밀하게 통합된 '실리콘-투-클라우드' 솔루션; 강력한 보안 아키텍처; 강건성을 위한 이중 IP/L-Band 전송.
- **약점 (Weaknesses)**: 측량 등급 경쟁사 대비 약간 낮은 최고 등급 정확도(3-6cm 대 <2cm); 일부 기존 사용자들이 '전문가급'이 아닌 '대중 시장용'으로 인식할 수 있는 브랜드 이미지; 최상의 성능을 위해 u-blox 생태계에 대한 의존성.
- **기회 (Opportunities)**: IoT, 로보틱스, 농업 분야에서 저렴한 고정밀 GNSS에 대한 거대하고 미개척된 시장; RTCM 지원을 활용하여 경쟁사 고객 전환 가능성; 신뢰할 수 있는 위치 정보에 대한 막대한 수요를 창출하는 자율 시스템의 성장.
- **위협 (Threats)**: Trimble/NovAtel과 같은 경쟁사들의 공격적인 가격 대응; 더 저렴한 새로운 보정 기술의 출현; 보정 서비스의 상품화 가능성으로 인한 마진 잠식.






- **대규모 IoT 장치군 운영자**: PointPerfect Flex는 매우 강력한 후보다. Pooled 요금제, ZTP, Thingstream 관리 플랫폼은 예측 가능한 낮은 TCO로 수천 개의 장치를 배포하고 관리하는 주요 과제를 해결하기 위해 특별히 설계되었다.
- **비용에 민감한 대중 시장 애플리케이션 개발자**: 저비용 F9 시리즈 하드웨어와 유연한 사용량 기반 서비스 요금제의 조합은 많은 애플리케이션에서 처음으로 센티미터 수준의 정확도를 경제적으로 실현 가능하게 한다. 무료 개발자 요금제는 평가를 위한 무위험 진입점을 제공한다.
- **미션 크리티컬 또는 원격 환경 사용자**: 이중 전송 메커니즘(IP + L-Band)은 필수적인 이중화를 제공한다. 연결성이 불안정한 자율 농업이나 원격 자산 추적과 같은 애플리케이션의 경우, L-Band 옵션(ZED-F9P + NEO-D9S 사용)은 지속적인 운영을 보장하는 중요한 기능이다.






PointPerfect Flex는 고정밀 위치 결정의 민주화를 위한 촉매제이다. 그 성공은 여러 산업에 걸쳐 자율성 채택을 가속화할 가능성이 높다. 시장은 7.0%에서 12.35%에 이르는 연평균 성장률(CAGR)로 강력한 성장을 지속할 것으로 예상된다.45 u-blox의 전략은 이 확장되는 대중 시장 부문의 상당 부분을 차지하기에 유리한 위치에 있다.

미래에는 단순히 원시적인 정확도 경쟁을 넘어, 비즈니스 모델과 사용 편의성에 대한 경쟁이 심화될 것이다. PointPerfect 포트폴리오(Flex, Live, Global, Safe)의 진화는 서비스가 특정 수직 시장의 요구 사항에 맞춰 점점 더 맞춤화되어, '하나의 크기가 모든 것에 맞는' 접근 방식에서 벗어날 것임을 시사한다. u-blox가 Sapcorda를 인수한 것은 이러한 비전을 가속화하고 기술 스택을 완전히 소유하기 위한 핵심적인 전략적 움직임이었다.39 결국 PointPerfect Flex는 고정밀 GNSS 기술이 소수의 전문가를 위한 도구에서 모든 산업이 활용할 수 있는 보편적인 유틸리티로 전환되는 중요한 이정표로 기록될 것이다.




1. Precision with PointPerfect Flex GNSS Correction Service - Symmetry Electronics, accessed July 17, 2025, https://www.symmetryelectronics.com/manufacturers/u-blox/achieve-unmatched-precision-with-pointperfect-gnss-correction-service/
2. Network RTK vs PPP-RTK: an insight into real-world performance | - u-blox, accessed July 17, 2025, https://www.u-blox.com/en/blogs/tech/network-rtk-vs-ppp-rtk
3. GNSS Corrections Demystified - Septentrio, accessed July 17, 2025, https://www.septentrio.com/en/learn-more/insights/gnss-corrections-demystified
4. PPP-RTK GNSS correction services (PointPerfect Flex) | u-blox, accessed July 17, 2025, https://www.u-blox.com/en/technologies/ppp-rtk-gnss-correction-services-pointperfect
5. PPP-RTK GNSS correction services (PointPerfect) | u-blox, accessed July 17, 2025, https://web-dev.u-blox.com/en/technologies/ppp-rtk-gnss-correction-services-pointperfect/
6. PPP-RTK with Rapid Convergence Based on SSR Corrections and Its Application in Transportation - MDPI, accessed July 17, 2025, https://www.mdpi.com/2072-4292/15/19/4770
7. PointPerfect Flex - u-blox, accessed July 17, 2025, https://www.u-blox.com/en/product/pointperfectflex
8. www.septentrio.com, accessed July 17, 2025, [https://www.septentrio.com/en/learn-more/insights/gnss-corrections-demystified#:~:text=PPP%2DRTK%20(a.k.a.%20SSR),satellite%20and%20atmospheric%20correction%20models.](https://www.septentrio.com/en/learn-more/insights/gnss-corrections-demystified#:~:text=PPP-RTK (a.k.a. SSR),satellite and atmospheric correction models.)
9. PointPerfect Flex - u-blox, accessed July 17, 2025, https://content.u-blox.com/sites/default/files/PointPerfect_ProductSummary_UBX-21024758.pdf
10. u-blox Extends Its Portfolio and Presence in the GNSS Market | Geo Week News, accessed July 17, 2025, https://www.geoweeknews.com/news/u-blox-extends-its-portfolio-and-presence-in-the-gnss-market
11. PointPerfect Flex getting started - u-blox Services - Thingstream, accessed July 17, 2025, https://support.thingstream.io/hc/en-gb/articles/18666964540060-PointPerfect-getting-started
12. PointPerfect Flex SPARTN over NTRIP Distribution - u-blox Services - Thingstream, accessed July 17, 2025, https://support.thingstream.io/hc/en-gb/articles/18667748780444-PointPerfect-SPARTN-over-NTRIP-Distribution
13. PointPerfect NTRIP Distribution - Thingstream Documentation, accessed July 17, 2025, https://developer.thingstream.io/guides/location-services/pointperfect-ntrip-distribution
14. SPARTN: Secure Position Augmentation for Real Time Navigation, accessed July 17, 2025, https://www.spartnformat.org/
15. SPARTN - New Open Data High Precision GNSS Format - xyHt, accessed July 17, 2025, https://www.xyht.com/gnsslocation-tech/spartn-new-open-data-high-precision-gnss-format/
16. APBE: Anti-Piracy Security for SPARTN Augmented GNSS Services - EWSN, accessed July 17, 2025, [https://www.ewsn.org/file-repository/ewsn2023/SPICES%202023-final9948.pdf](https://www.ewsn.org/file-repository/ewsn2023/SPICES 2023-final9948.pdf)
17. PointPerfect Flex Service Description, accessed July 17, 2025, https://support.thingstream.io/hc/en-gb/articles/18667481258140-PointPerfect-Flex-Service-Description
18. PointPerfect Service Description - Thingstream Documentation, accessed July 17, 2025, https://developer.thingstream.io/guides/location-services/pointperfect-service-description
19. PointPerfect RTCM Distribution - Thingstream Documentation, accessed July 17, 2025, https://developer.thingstream.io/guides/location-services/pointperfect-rtcm-distribution
20. PointPerfect L-band Configuration - Thingstream Documentation, accessed July 17, 2025, https://developer.thingstream.io/guides/location-services/pointperfect-getting-started/pointperfect-l-band-configuration
21. PointPerfect L-Band Corrections Receiver NEO-D9S - ArduSimple, accessed July 17, 2025, https://www.ardusimple.com/product/pointperfect-l-band-corrections-receiver-neo-d9s/
22. Precision agriculture - u-blox, accessed July 17, 2025, https://www.u-blox.com/en/precision-agriculture
23. u-blox Introduces PointPerfect Localized Distribution Feature | UST, accessed July 17, 2025, https://www.unmannedsystemstechnology.com/2023/09/u-blox-introduces-pointperfect-localized-distribution-feature/
24. u-blox and Position Partners roll out PointPerfect GNSS augmentation service to Australia, accessed July 17, 2025, https://www.u-blox.com/en/u-blox-position-partners-roll-out-point-perfect-gnss-augmentation-service-australia
25. PointPerfect Flex service coverage - u-blox, accessed July 17, 2025, https://www.u-blox.com/en/pointperfect-service-coverage
26. PointPerfect Localized Distribution - Thingstream Documentation, accessed July 17, 2025, https://developer.thingstream.io/guides/location-services/pointperfect-localized-distribution
27. New feature: PointPerfect Localized Distribution - u-blox, accessed July 17, 2025, https://content.u-blox.com/sites/default/files/documents/PointPerfect-Localization_PCN_UBXDOC-1078673104-2339.pdf
28. PointPerfect Flex usage-based plans - u-blox, accessed July 17, 2025, https://www.u-blox.com/en/PointPerfect-usage-based-plans
29. Pricing | Thingstream, accessed July 17, 2025, https://portal.thingstream.io/pricing/laas/laaspointperfect
30. ZED-F9R module - u-blox, accessed July 17, 2025, https://www.u-blox.com/en/product/zed-f9r-module
31. ZED-F9R module - u-blox, accessed July 17, 2025, https://content.u-blox.com/sites/default/files/ZED-F9R_ProductSummary_UBX-19048775.pdf
32. ZED-F9K-00B Data sheet - u-blox, accessed July 17, 2025, https://content.u-blox.com/sites/default/files/ZED-F9K-00B_DataSheet_UBX-17061422.pdf
33. Easier access to market's best high precision GNSS positioning performance - u-blox, accessed July 17, 2025, https://www.u-blox.com/en/Easier-access-to-markets-best-high-precision-GNSS-positioning-performance
34. NEO-D9S series - u-blox, accessed July 17, 2025, https://www.u-blox.com/en/product/neo-d9s-series
35. PointPerfect getting started - Thingstream Documentation, accessed July 17, 2025, https://developer.thingstream.io/guides/location-services/pointperfect-getting-started
36. GNSS Correction Data Receiver (NEO-D9S) Hookup Guide - SparkFun Learn, accessed July 17, 2025, https://learn.sparkfun.com/tutorials/gnss-correction-data-receiver-neo-d9s-hookup-guide/all
37. Precision positioning for autonomous service robots | u-blox, accessed July 17, 2025, https://www.u-blox.com/en/precision-positioning-for-autonomous-service-robots
38. Getting Started with u-blox Thingstream and PointPerfect - SparkFun Learn, accessed July 17, 2025, https://learn.sparkfun.com/tutorials/getting-started-with-u-blox-thingstream-and-pointperfect/hardware-hookup
39. u-blox acquires full ownership in Sapcorda Joint Venture |, accessed July 17, 2025, https://www.u-blox.com/en/investor-press-releases/u-blox-acquires-full-ownership-in-sapcorda-joint-venture
40. ZED-F9P Product Summary - u-blox, accessed July 17, 2025, https://content.u-blox.com/sites/default/files/ZED-F9P_ProductSummary_UBX-17005151.pdf
41. ZED-X20P-00B Data sheet - u-blox, accessed July 17, 2025, https://content.u-blox.com/sites/default/files/documents/ZED-X20P-00B_DataSheet_UBXDOC-963802114-12690.pdf
42. u-blox Brings Cost-Effective Positioning Solutions to the Agriculture Market | GOFAR, accessed July 17, 2025, https://www.agricultural-robotics.com/news/u-blox-brings-cost-effective-positioning-solutions-to-the-agriculture-market
43. Maximizing Precision Agriculture ROI - Number Analytics, accessed July 17, 2025, https://www.numberanalytics.com/blog/maximizing-precision-agriculture-roi
44. Global Navigation Satellite Systems as State-of-the-Art Solutions in Precision Agriculture: A Review of Studies Indexed in the Web of Science - ResearchGate, accessed July 17, 2025, https://www.researchgate.net/publication/372406481_Global_Navigation_Satellite_Systems_as_State-of-the-Art_Solutions_in_Precision_Agriculture_A_Review_of_Studies_Indexed_in_the_Web_of_Science
45. GNSS Correction Service Market Size and Trends Research [2033] - Global Growth Insights, accessed July 17, 2025, https://www.globalgrowthinsights.com/market-reports/gnss-correction-service-market-116501
46. How precision agriculture improves yield | u-blox, accessed July 17, 2025, https://www.u-blox.com/en/use-case/tech/precision-agriculture-improves-yield
47. Unlocking the Power of Precision Agriculture with Nordian's OEM Board and PX9 Solution, accessed July 17, 2025, https://www.u-blox.com/en/unlocking-the-power-of-precision-agriculture
48. Trimble RTX Correction Services | Trimble Positioning Services, accessed July 17, 2025, https://positioningservices.trimble.com/en/rtx
49. Correction Services, accessed July 17, 2025, https://assets.ctfassets.net/9k5dj5b59lqq/2ULag8MOBWQSrSjVyM0Wos/79dfd4d6d99512c8a09e636560ed7e80/022517-270_TAP_Collateral_Update_FAQ_USL_0124.pdf
50. Performance specifications - Trimble Field Systems Help Portal, accessed July 17, 2025, https://help.fieldsystems.trimble.com/r10/specs_all_r10-2_r12_r12i.htm
51. Trimble CenterPoint RTX Correction Service - SITECH Construction Systems, accessed July 17, 2025, https://sitechcs.com/docs/trimble-centerpoint-rtx-and-gnss-receivers-datasheet.pdf
52. RTX correction service - Trimble Field Systems Help Portal, accessed July 17, 2025, https://help.fieldsystems.trimble.com/trimble-access/latest/en/gnss-survey-rtx.htm
53. Frequently Asked Questions Correction Service - Trimble Geospatial, accessed July 17, 2025, https://geospatial.trimble.com/downloads/7EN0wFRAk234kbJgk4FZOV/b47339a9dd4437b25f9e6104050d149f/Trimble_RTX_Correction_Services_FAQ.pdf
54. TerraStar Correction Services | NovAtel, accessed July 17, 2025, https://novatel.com/products/gps-gnss-correction-services/terrastar-correction-services
55. NovAtel TerraStar GNSS Corrections - AutonomouStuff, accessed July 17, 2025, https://autonomoustuff.com/products/novatel-terrastar-gnss-corrections
56. GLOBAL CORRECTION SERVICES - Canal Geomatics, accessed July 17, 2025, https://canalgeomatics.com/wp-content/uploads/2019/12/terrastar-brochure.pdf
57. TerraStar-X Services - NovAtel, accessed July 17, 2025, https://novatel.com/products/gps-gnss-correction-services/terrastar-correction-services/terrastar-x-services
58. Correction services worldwide | Hexagon, accessed July 17, 2025, https://hexagon.com/solutions/correction-services-worldwide
59. TerraStar-C Pro Multi-Constellation Correction Service - NavtechGPS, accessed July 17, 2025, https://www.navtechgps.com/terrastar_c_pro_multi_constellation_correction_service/
60. RTK+INS simpleRTK2B-F9R hookup guide - ArduSimple, accessed July 17, 2025, https://www.ardusimple.com/rtk-ins-simplertk2b-f9r-hookup-guide/
61. PointSafe | u-blox, accessed July 17, 2025, https://www.u-blox.com/en/product/pointsafe
62. High precision positioning | u-blox, accessed July 17, 2025, https://www.u-blox.com/en/technologies/high-precision-positioning
63. PointSafe Product Summary - u-blox, accessed July 17, 2025, https://content.u-blox.com/sites/default/files/documents/PointSafe_ProductSummary_UBX-23011141.pdf
64. u-safe Product Summary - u-blox, accessed July 17, 2025, https://content.u-blox.com/sites/default/files/documents/u-safe_ProductSummary_UBX-23011142.pdf
65. ZED-F9K-02A Data sheet - u-blox, accessed July 17, 2025, https://content.u-blox.com/sites/default/files/documents/ZED-F9K-02A_DataSheet_UBXDOC-304424225-18291.pdf
66. PointPerfect Live | u-blox, accessed July 17, 2025, https://www.u-blox.com/en/product/pointperfectlive
67. u-blox Introduces PointPerfect Global GNSS Correction Service Portfolio, accessed July 17, 2025, https://insidegnss.com/u-blox-introduces-pointperfect-global-gnss-correction-service-portfolio/
68. u-blox introduces PointPerfect Global, completing its GNSS correction service portfolio with seamless worldwide coverage, accessed July 17, 2025, https://www.u-blox.com/en/u-blox-introduces-pointperfect-global
69. Positioning chips and modules | u-blox, accessed July 17, 2025, https://www.u-blox.com/en/positioning-chips-and-modules
70. GNSS Correction Module from u-blox Makes Way for Centimeter-level Positioning - News, accessed July 17, 2025, https://www.allaboutcircuits.com/news/gnss-correction-module-from-ublox-makes-way-for-centimeter-level-positioning/
71. Anti-Jamming techniques in u-blox GPS receivers, accessed July 17, 2025, [https://content.u-blox.com/sites/default/files/products/documents/u-blox-AntiJamming_WhitePaper_%28GPS-X-09008%29.pdf](https://content.u-blox.com/sites/default/files/products/documents/u-blox-AntiJamming_WhitePaper_(GPS-X-09008).pdf)
72. GNSS/GPS jamming and spoofing tests under actual conditions | - u-blox, accessed July 17, 2025, https://www.u-blox.com/en/blogs/insights/gnss-gps-jamming-spoofing
73. A new security level against GNSS spoofing - u-blox, accessed July 17, 2025, https://www.u-blox.com/en/blogs/tech/gnss-spoofing-new-security-level
74. The looming threat of GPS jamming and spoofing - u-blox, accessed July 17, 2025, https://www.u-blox.com/en/blogs/insights/looming-threat-gps-jamming-and-spoofing
75. Zero Touch Provisioning - Thingstream Documentation, accessed July 17, 2025, https://developer.thingstream.io/guides/security-services/features-overview/zero-touch-provisioning
76. IoT Security-as-a-Service - u-blox, accessed July 17, 2025, https://web-dev.u-blox.com/en/iot-security-service/
77. GNSS Correction Service Market Size, Growth Report 2033, accessed July 17, 2025, https://www.businessresearchinsights.com/market-reports/gnss-correction-service-market-102858




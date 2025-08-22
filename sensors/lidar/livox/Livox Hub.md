---
layout: page
title: Livox Hub에 대한 심층 기술 분석
permalink: /sensors/lidar/livox/Livox Hub
---



Livox Hub는 단순히 여러 LiDAR 센서를 하나의 장치에 연결하는 물리적 포트 확장기(Port Expander)의 개념을 넘어선다. 본질적으로 이 장치는 다중 LiDAR 센서 어레이(Array)를 단일의 응집력 있는 인식 시스템으로 변환하는 중앙 제어 장치(Central Control Unit)로 기능한다. 그 핵심 역할은 세 가지 주요 기능으로 요약될 수 있다: 전원 분배, 데이터 집선, 그리고 동기화.1 이 세 가지 기능이 유기적으로 결합하여 복잡한 다중 센서 시스템의 통합 및 개발 과정을 획기적으로 단순화시킨다.

첫째, **전원 분배(Power Distribution)** 기능은 시스템의 안정성과 직결된다. Hub는 최대 9개의 LiDAR 센서에 개별적으로 전원을 공급할 수 있다.1 각 포트는 독립적인 전원 제어 회로와 단락 보호(Short Circuit Protection) 기능을 내장하고 있다.4 이는 특정 센서나 케이블에 문제가 발생하더라도 전체 시스템의 다운으로 이어지는 것을 방지하며, Livox Viewer나 SDK를 통해 특정 포트의 전원을 원격으로 켜고 끌 수 있어 디버깅 및 운영 효율성을 높인다.5

둘째, **데이터 집선(Data Aggregation)** 기능은 데이터 처리 파이프라인을 간소화한다. 최대 9개의 센서에서 생성되는 방대한 양의 포인트 클라우드 데이터를 단일 1Gbps 이더넷 인터페이스로 통합하여 호스트 PC로 전송한다.1 Hub는 초당 최대 2,700,000 포인트(2.7 million points per second)의 데이터를 처리할 수 있는 성능을 갖추고 있어, 여러 고성능 LiDAR가 동시에 작동하더라도 데이터 병목 현상 없이 안정적인 스트리밍을 보장한다.1

셋째, 그리고 가장 중요한 **동기화(Synchronization)** 기능은 다중 센서 데이터의 가치를 극대화한다. Hub는 여러 센서에서 들어오는 데이터 스트림을 시간적, 공간적으로 정렬한다. IEEE 1588 프로토콜이나 GPS 신호를 이용해 모든 센서의 타임스탬프를 마이크로초 이하(μs) 수준으로 일치시키는 시간 동기화를 수행한다.1 동시에, 사용자가 사전에 캘리브레이션을 통해 획득한 각 센서의 상대적 위치 및 자세 정보(외부 파라미터, Extrinsic Parameters)를 입력받아, 이를 각 포인트 클라우드에 적용하여 공간적으로 완벽히 정합된 단일의 통합 포인트 클라우드를 생성한다.1 이로써 개발자는 복잡한 동기화 문제를 하드웨어에 위임하고, 정제된 데이터 기반의 상위 애플리케이션 개발에 집중할 수 있게 된다.


Livox Hub의 시스템 통합을 고려하는 엔지니어는 물리적, 전기적, 그리고 데이터 관련 사양을 면밀히 검토해야 한다. 이러한 사양들은 로봇이나 차량의 기구 설계, 전원 시스템 설계, 그리고 데이터 처리 아키텍처에 직접적인 영향을 미치기 때문이다.

물리적/기계적 사양

Hub는 포트 보호를 위한 커버의 장착 여부에 따라 크기와 무게가 달라진다.2 포트 커버를 모두 장착했을 때의 크기는 208 × 201 × 29 mm, 무게는 약 1352 g이다. 반면, 공간 제약으로 인해 커버를 제거할 경우 크기는 153.2 × 201 × 29 mm, 무게는 약 865 g으로 줄어든다.6 장착은 바닥면에 위치한 4개의 M3 나사 구멍(깊이 6 mm)을 통해 이루어진다.2 포트 커버는 외부 충격이나 오염으로부터 커넥터를 보호하는 역할을 하므로, 불가피한 경우가 아니라면 부착 상태로 사용하는 것이 권장된다.2 또한, 장치 상단에는 방열 영역(Dissipation Area)이 존재하므로, 설치 시 이 부분의 공기 흐름이 막히지 않도록 주의해야 한다.2

전기적 사양

Hub의 구동을 위해서는 10V에서 23V 사이의 DC 전원이 필요하며, 일반적인 사용 환경에서는 12V DC가 권장된다.2 만약 23V를 초과하는 전원을 사용해야 할 경우, 사전에 Livox 측에 문의가 필요하다는 경고가 있다.8 소모 전력은 연결된 LiDAR의 모델과 수량, 그리고 작동 온도에 따라 크게 달라지는 가변적인 값이다. 예를 들어, Livox Mid-100 센서 5대를 25°C 환경에서 12V 전원으로 구동할 경우 약 180W(15A)의 전력이 소모되며, Livox Horizon 센서 9대를 동일 조건에서 연결하면 약 130W가 소모된다.2 이는 시스템 전체의 전원 공급 장치(PSU) 용량과 케이블 굵기를 결정하는 데 있어 매우 중요한 설계 인자이다.

데이터 인터페이스 및 처리 능력

Hub는 총 9개의 Livox LiDAR 포트를 제공하며, 집선된 데이터는 단일 1Gbps 이더넷 포트를 통해 외부로 출력된다.3 안정적인 기가비트 통신을 위해 CAT5E 등급 이상의 이더넷 케이블과 DHCP 기능을 지원하는 기가비트 라우터의 사용이 필수적이다.2 장치 전면에는 3개의 상태 표시 LED가 있어 Hub의 현재 상태를 직관적으로 파악할 수 있다. LED 1과 2는 초기화, GPS 동기화, PTP 동기화 등 Hub의 전반적인 작동 상태를 나타내며, LED 3은 현재 이더넷 연결이 기가비트(녹색)인지 메가비트(청색)인지 표시해준다.2

| 기능 (Feature)                      | 사양 (Specification)                         | 출처 (Source Snippets) |
| ----------------------------------- | -------------------------------------------- | ---------------------- |
| LiDAR 포트 (LiDAR Ports)            | 9                                            | 3                      |
| 입력 전압 (Input Voltage)           | 10-23 V DC (Typical 12 V DC)                 | 2                      |
| 입력 전력 (Input Power)             | 가변적 (예: 5x Mid-100 @ 25°C = 180W)        | 2                      |
| 데이터 인터페이스 (Data Interface)  | 1Gbps Ethernet                               | 2                      |
| 시간 동기화 프로토콜 (Time Sync)    | IEEE 1588-2008 (PTPv2), GPS                  | 1                      |
| 크기 (커버 포함)                    | 208 × 201 × 29 mm                            | 2                      |
| 크기 (커버 미포함)                  | 153.2 × 201 × 29 mm                          | 2                      |
| 무게 (커버 포함)                    | 약 1352 g                                    | 2                      |
| 무게 (커버 미포함)                  | 약 865 g                                     | 2                      |
| 작동 온도 (Operating Temp.)         | -40°C to 85°C                                | 2                      |
| 호환 LiDAR 모델 (Compatible Models) | Mid-40, Mid-100, Horizon, Tele-15            | 8                      |
| 최대 케이블 길이 (Max Cable Length) | 12 m (Mid-40/Horizon/Tele-15), 8 m (Mid-100) | 8                      |

*Table 1: Livox Hub Technical Specifications Summary*


Livox Hub는 Livox의 모든 제품 라인업을 위한 범용 솔루션이 아니다. 그 위치와 호환성을 정확히 이해하는 것은 시스템 설계의 성패를 좌우하는 중요한 요소다. 공식 문서 및 리셀러 정보를 종합하면, Hub는 명시적으로 Livox Mid-40, Mid-100, Horizon, Tele-15 모델과의 호환성을 보장한다.8 이들 센서와 Hub를 연결할 때 연장 케이블을 사용할 경우, Mid-40, Horizon, Tele-15는 최대 12m, Mid-100은 최대 8m까지의 거리가 허용된다.8 이 거리를 초과하는 설계가 필요하다면 Livox 측과의 별도 협의가 필요하다.

여기서 주목해야 할 점은 Livox Hub의 '세대적 한계'이다. Hub의 사용자 매뉴얼(v1.2)은 2019년 11월에, 마지막 펌웨어(v08.09.0000)는 2020년 12월에 릴리즈된 후 업데이트가 중단되었다.10 이 시점은 Livox가 Horizon, Tele-15 등의 제품군을 주력으로 내세우던 시기와 일치한다. 이후 Livox는 HAP, Mid-360과 같은 차세대 LiDAR를 출시했다.12 이 신형 센서들은 Hub와는 다른 소프트웨어 스택을 기반으로 작동한다. 예를 들어, 이들은 'Livox Viewer 2'라는 새로운 버전의 GUI 툴과 `livox_ros_driver2`라는 별도의 ROS 2 드라이버를 사용하며, 이들의 기술 문서나 호환성 목록 어디에서도 Livox Hub는 언급되지 않는다.10

이러한 정황은 Livox Hub가 특정 세대의 제품군(2019-2020년경 주력 모델)을 위해 특별히 설계된 '세대 특화적(generation-specific)' 솔루션이라는 결론으로 이어진다. 즉, Hub는 Livox의 모든 센서를 아우르는 통합 플랫폼이 아니며, 그 역할은 과거의 특정 제품군에 국한된다. 이는 Livox의 제품 개발 전략이 Hub와 같은 중앙 집중식 통합 장치에서 벗어나, 개별 센서의 기능을 고도화하고(예: Automotive Ethernet 지원), 각 센서가 독립적으로 시스템에 통합될 수 있는 방향으로 전환되었음을 시사한다. 따라서, 신규 프로젝트에 HAP이나 Mid-360과 같은 최신 Livox 센서를 도입하려는 개발자는 Hub와의 호환성을 기대해서는 안 된다. Livox Hub는 구형 센서 재고를 활용하거나, Hub를 중심으로 이미 구축된 기존 시스템을 유지보수하는 제한적인 시나리오에서만 유효한 선택지가 될 것이다.


다중 센서 시스템에서 '동기화'는 단순히 데이터를 합치는 것을 넘어, 각기 다른 시점과 위치에서 수집된 정보를 하나의 일관된 시공간적 맥락으로 재구성하는 과정이다. Livox Hub는 이 복잡한 과제를 하드웨어 레벨에서 해결하기 위해 시간 동기화와 공간 동기화 두 가지 핵심 기능을 제공한다.


Livox Hub는 마이크로초 이하(μs) 수준의 정밀한 시간 동기화를 위해 두 가지 표준 프로토콜, PTP(Precision Time Protocol)와 GPS를 지원한다.1 이 두 방식은 서로 다른 환경과 요구사항에 대응하기 위해 상호 보완적으로 설계되었다.

PTP (Precision Time Protocol)

PTP는 네트워크를 통해 컴퓨터나 장비들의 시계를 정밀하게 맞추는 기술로, Livox Hub는 IEEE 1588-2008, 즉 PTPv2 표준을 따른다.2 PTP의 핵심 원리는 네트워크 내에 기준 시계 역할을 하는 '마스터 클럭(Master Clock)'을 두고, 다른 모든 장치('슬레이브 클럭', Slave Clock)들이 이 마스터의 시간에 자신을 맞추는 것이다. Livox Hub는 PTP 네트워크에서 슬레이브로 동작한다.15

동기화 과정은 마스터와 슬레이브 간의 양방향 메시지 교환을 통해 이루어진다. 마스터는 `Sync` 메시지와 `Follow_Up` 메시지를 보내 자신의 타임스탬프(t1, t2)를 알리고, 슬레이브는 `Delay_Req` 메시지와 `Delay_Resp` 메시지를 통해 자신의 타임스탬프(t3, t4)를 교환한다.15 Hub는 이 4개의 타임스탬프를 이용해 다음과 같은 계산을 수행한다:

1. **경로 지연 (Path Delay):** Delay=[(t4−t1)−(t3−t2)]/2
2. **시간 오프셋 (Time Offset):** Offset=(t2−t1)−Delay

이 수학적 보정을 통해 네트워크에서 발생하는 전송 지연(latency)을 정밀하게 제거하고 마스터와 슬레이브 간의 시간 차이를 마이크로초 이하로 줄일 수 있다.15 PTP는 소프트웨어 레벨의 지연을 최소화하기 위해 네트워크 인터페이스 카드(NIC) 단에서 하드웨어 타임스탬핑을 사용하는 경우가 많아, NTP(Network Time Protocol)보다 월등히 높은 정밀도를 제공한다.16

GPS 동기화

GPS 동기화는 위성항법시스템이 제공하는 절대 시간을 기준으로 동기화를 수행하는 방식이다. 이를 위해 Hub는 외부 GPS 수신기로부터 두 가지 신호를 받는다: PPS(Pulse Per Second) 신호와 GPRMC 포맷의 시간 정보 메시지.2

- **PPS 신호:** 1초에 한 번씩 매우 정확한 간격으로 발생하는 전기적 펄스다. 이 펄스의 상승 에지(rising edge)는 매초가 시작되는 정확한 물리적 시점을 알리는 하드웨어 트리거 역할을 한다.
- **GPRMC 메시지:** PPS 신호 직후에 수신되는 직렬 통신 데이터로, "이 펄스는 협정 세계시(UTC) 기준으로 몇 년, 몇 월, 며칠, 몇 시, 몇 분, 몇 초에 해당하는가"라는 시간 정보를 담고 있다.

Livox Hub는 전용 GPS Sync 포트를 통해 이 두 신호를 수신한다.2 Hub는 PPS 신호의 상승 에지를 감지하는 순간, 함께 수신된 GPRMC 메시지를 파싱하여 현재의 절대 시간을 획득하고, 이를 기준으로 내부 클럭과 연결된 모든 LiDAR의 타임스탬프를 보정한다.15

이 두 가지 동기화 방식의 지원은 Livox Hub의 적용 범위를 크게 확장한다. PTP는 GPS 신호가 도달하지 않는 실내, 지하, 도심 빌딩 숲과 같은 환경에서 다수의 로봇이나 센서 노드를 정밀하게 동기화하는 데 필수적인 솔루션이다.15 이를 위해서는 네트워크 내에 PTP 마스터 역할을 할 장비(예: 전용 그랜드마스터 클럭, PTP 지원 스위치)를 별도로 구성해야 하는 인프라 요구사항이 따른다.18 반면, GPS 동기화는 실외에서 운용되는 자율주행차나 드론 매핑 시스템에 절대적인 지리적, 시간적 기준점을 제공한다.15 여러 차량이나 장비가 동일한 UTC 시간축을 공유하게 되므로, 서로 다른 시스템에서 수집된 데이터를 융합하거나 사후 분석할 때 데이터의 일관성을 확보하는 데 매우 유리하다. 개발자는 자신의 애플리케이션 환경과 요구되는 시간 기준(상대 시간 vs 절대 시간)에 따라 적절한 동기화 인프라를 선택하고 구축해야 한다.


시간축을 정렬하는 것만으로는 완벽한 3D 환경 인식이 불가능하다. 각기 다른 위치와 방향으로 설치된 센서들의 데이터를 하나의 좌표계로 통합하는 공간 동기화가 반드시 필요하다. Livox Hub는 이 과정을 "사용자 좌표계로의 포인트 클라우드 데이터 통합" 기능으로 지원한다.1

이 기능의 핵심은 사용자가 사전에 측정한 각 LiDAR 센서의 상대적인 6자유도(6-DoF) 변환 정보, 즉 외부 파라미터(Extrinsic Parameters)를 Hub에 입력하는 것이다. 외부 파라미터는 기준 좌표계(보통 로봇의 중심) 대비 각 센서의 위치 변위(x, y, z)와 회전 변위(roll, pitch, yaw)로 구성된 변환 행렬이다.19 사용자가 Livox Viewer나 SDK를 통해 이 파라미터들을 Hub에 제공하면, Hub는 수신되는 각 센서의 포인트 클라우드 데이터에 해당 변환 행렬을 실시간으로 곱하여 모든 데이터를 단일 사용자 좌표계 기준으로 변환한 후 통합된 스트림으로 출력한다.1

여기서 매우 중요한 점은, Livox Hub가 외부 파라미터를 '계산'하거나 '추정'하는 캘리브레이션 장치가 아니라는 사실이다. Hub의 역할은 이미 정확하게 계산된 파라미터를 실시간으로 '적용'하는 하드웨어 가속 변환 엔진(transformation engine)에 가깝다. Hub의 공식 문서들은 "LiDAR 설치 파라미터를 입력한 후(After entering the LiDAR installation parameters)" 데이터를 통합한다고 명시하며, 캘리브레이션 과정 자체는 Hub의 기능 범위 밖에 있음을 분명히 한다.1

실제 다중 LiDAR 간의 외부 파라미터 캘리브레이션은 매우 비자명한(non-trivial) 과정이다. 이를 위해서는 Livox Viewer에 내장된 수동 캘리브레이션 도구를 사용하거나 20, Livox가 GitHub를 통해 제공하는 타겟 기반(체커보드 등) 또는 타겟리스(target-less) 자동 캘리브레이션 오픈소스 툴을 활용해야 한다.20 이러한 툴들은 환경 내의 평면이나 코너점, 또는 공통 특징점을 식별하고 최적화 기법을 통해 센서 간의 정밀한 변환 행렬을 계산한다.

따라서 잠재적 사용자는 Livox Hub를 도입하면 다중 LiDAR의 공간 정합 문제가 마법처럼 해결될 것이라고 오해해서는 안 된다. Hub는 캘리브레이션 *이후*의 데이터 처리 파이프라인을 단순화하고 실시간성을 보장하는 강력한 도구이지만, 그 성능을 온전히 발휘하기 위한 전제 조건인 '정밀한 외부 파라미터'를 확보하는 것은 전적으로 사용자의 책임이다. 다중 LiDAR 시스템 구축의 전체 워크플로우에서 캘리브레이션 단계의 복잡성과 중요성을 간과해서는 안 되며, 이를 성공적으로 수행하기 위한 전문 지식과 추가적인 시간 및 노력이 필요함을 반드시 인지해야 한다.


Livox Hub의 동기화 전략은 경쟁사인 Velodyne과 Ouster의 접근 방식과 비교할 때 그 특징이 더욱 명확해진다.

Velodyne

Velodyne은 LiDAR 시장의 선구자로서, 전통적으로 GPS 기반 동기화에 강점을 보여왔다. 이들의 시스템은 외부 GNSS 수신기로부터 PPS 신호와 NMEA 메시지(특히 $GPRMC 또는 $GPGGA)를 수신하여 타임스탬프를 절대 UTC 시간으로 맞추는 방식을 표준으로 채택하고 있다.24 이는 주로 옥외 자율주행 차량이나 모바일 매핑 시스템 시장에 집중했던 기술적 유산으로 볼 수 있다. PTP 지원은 후기 모델이나 특정 펌웨어 업데이트를 통해 추가된 기능으로, GPS가 기본이고 PTP는 보조적인 옵션이라는 인식이 강하다.26

Ouster

반면, Ouster는 PTP/gPTP를 기본 동기화 메커니즘으로 매우 적극적으로 채택하고 있다. 이들은 단순히 PTP를 지원하는 것을 넘어, default, gptp(IEEE 802.1AS), automotive-slave 등 다양한 PTP 프로파일을 제공하여 특정 네트워크 환경(예: Audio Visual Bridging, Automotive Ethernet)과의 상호운용성을 극대화한다.27 또한, HTTP API를 통해 실시간으로 PTP 프로파일을 변경할 수 있는 유연성을 제공하여 개발자가 다양한 테스트 환경에 신속하게 적응할 수 있도록 지원한다.27 이는 Ouster가 단순한 시간 동기화를 넘어 산업 표준과의 긴밀한 통합을 중요한 가치로 여기고 있음을 보여준다.

Livox Hub

Livox Hub는 이 두 경쟁사의 중간 지점에서 실용적인 접근 방식을 취한다. PTP와 GPS/PPS라는 두 가지 핵심 동기화 방식을 동등하게 지원함으로써 실내외를 아우르는 범용성을 확보하고자 했다.15 Ouster처럼 다양한 PTP 프로파일을 제공하는 등의 고급 기능보다는, 가장 널리 사용되는 PTPv2와 GPS 동기화라는 두 가지 핵심 기능을 안정적으로 제공하는 데 초점을 맞춘 것으로 보인다. 이는 특정 산업 표준에 깊이 통합되기보다는, 광범위한 연구 및 개발자 커뮤니티가 손쉽게 다중 LiDAR 시스템을 구축할 수 있도록 진입 장벽을 낮추려는 전략으로 해석할 수 있다.

| 제조사 (Manufacturer) | 주 동기화 방식 (Primary Sync)       | 보조 동기화 방식 (Secondary Sync) | 시간 정밀도 (Time Precision) | 주요 특징 및 철학 (Key Features/Philosophy)                  | 출처 (Source Snippets) |
| --------------------- | ----------------------------------- | --------------------------------- | ---------------------------- | ------------------------------------------------------------ | ---------------------- |
| **Livox (via Hub)**   | PTP (IEEE 1588v2), GPS (PPS+$GPRMC) | -                                 | Sub-microsecond              | PTP와 GPS를 동등하게 지원하여 실내외 범용성 확보. 통합의 편리성에 집중. | 2                      |
| **Velodyne**          | GPS (PPS+NMEA)                      | PTP (일부 모델)                   | Microsecond (GPS)            | 전통적인 실외/차량 애플리케이션에 최적화된 GPS 동기화가 표준. | 24                     |
| **Ouster**            | PTP / gPTP                          | GPS (NMEA)                        | Nanosecond (PTP)             | 다양한 PTP 프로파일 지원으로 산업 표준과의 상호운용성 및 유연성 극대화. | 27                     |

*Table 2: Multi-LiDAR Synchronization Method Comparison*

이 표는 시스템 설계자가 자신의 애플리케이션에 가장 적합한 솔루션을 선택하는 데 중요한 기준을 제공한다. 예를 들어, 고도로 표준화된 자동차 네트워크를 구축한다면 Ouster의 다양한 PTP 프로파일 지원이 결정적인 장점이 될 수 있다. 반면, 전통적인 GNSS/INS 기반의 측량 시스템과 통합해야 한다면 Velodyne의 방식이 더 익숙하고 직접적일 수 있다. Livox Hub는 이 두 접근 방식 사이에서, 복잡한 설정 없이 두 가지 핵심 동기화 기능을 모두 사용하고자 하는 개발자에게 매력적인 '올인원(All-in-one)' 솔루션으로 자리매김한다.


하드웨어의 성능은 그것을 제어하고 활용하는 소프트웨어에 의해 완성된다. Livox Hub는 전용 GUI 툴인 Livox Viewer, 개발자를 위한 SDK, 그리고 로봇 개발의 표준 플랫폼인 ROS/ROS2와의 연동을 통해 강력한 소프트웨어 생태계를 구축하고 있다.


Livox Viewer는 Livox 센서 및 Hub를 위해 특별히 설계된 GUI 소프트웨어로, Windows와 Ubuntu 운영체제를 지원한다.10 이 툴은 다중 LiDAR 시스템을 운영하고 디버깅하는 데 필수적인 창구 역할을 한다.

주요 기능은 다음과 같다:

- **데이터 시각화 및 관리:** 연결된 센서로부터 수신되는 포인트 클라우드 데이터를 실시간으로 3D 공간에 시각화한다. 데이터는 반사율, 거리, 높이 등 다양한 기준에 따라 색상을 입힐 수 있다.19 또한, 수신된 데이터를 

  `.lvx`라는 자체 포맷으로 녹화하고, 이 파일을 다시 불러와 재생하는 기능도 제공하여 오프라인 분석 및 알고리즘 테스트를 용이하게 한다.21

- **장치 제어 및 모니터링:** Viewer 내의 장치 관리자(Device Manager)를 통해 네트워크에 연결된 Hub와 그에 속한 개별 LiDAR들의 상태(펌웨어 버전, 온도, 에러 코드 등)를 실시간으로 모니터링할 수 있다.5 또한, 각 장치의 파라미터(예: 리턴 모드, IP 주소)를 설정하거나, Hub에 연결된 특정 LiDAR 포트의 전원을 개별적으로 켜고 끄는 등의 직접적인 제어가 가능하다.4

- **캘리브레이션 지원:** 다중 LiDAR 간의 외부 파라미터(Extrinsics)를 수동으로 캘리브레이션할 수 있는 인터페이스를 제공한다.20 사용자는 두 센서의 포인트 클라우드를 중첩시켜 보면서 roll, pitch, yaw, x, y, z 값을 미세 조정하여 최적의 정합 상태를 찾을 수 있다.

하지만 Livox의 소프트웨어 생태계에는 '파편화'라는 중요한 문제가 존재한다. Livox 공식 다운로드 페이지를 보면 'Livox Viewer'와 'Livox Viewer 2'가 별개의 제품으로 제공되고 있음을 알 수 있다.10 분석 결과, Hub, Mid-70, Avia, Horizon, Tele-15와 같은 구형 제품군은 2021년 3월에 마지막으로 업데이트된 Livox Viewer(버전 0.11.0)를 사용해야 한다.10 반면, HAP, Mid-360과 같은 신형 제품군은 2023년에 출시된 Livox Viewer 2를 사용하도록 안내된다.10 실제로 Reddit의 한 사용자는 Mid-70 센서가 Viewer 2에서는 인식되지 않아 문제를 겪다가, 구버전인 Viewer 1을 설치하고 나서야 문제를 해결했다는 경험을 공유했다.30

이는 Livox의 소프트웨어 도구가 세대별로 분리되어 있으며, 완벽한 하위 호환성을 보장하지 않음을 명확히 보여준다. 사용자는 자신이 보유한 센서 모델에 맞는 정확한 버전의 소프트웨어를 사용해야 하는 번거로움이 있다. 만약 구형 센서(예: Horizon)와 신형 센서(예: Mid-360)를 하나의 시스템에서 혼용하고자 한다면, 두 개의 서로 다른 Viewer를 사용해야 하는 비효율적인 상황에 직면할 수도 있다. 이는 개발 및 유지보수 과정에서 혼란을 야기할 수 있는 실질적인 문제점이므로, 시스템을 구성하기 전에 반드시 인지해야 할 부분이다.


Livox SDK(Software Development Kit)는 개발자가 Livox 장치의 모든 기능을 프로그래밍 방식으로 제어하고, 원시(raw) 포인트 클라우드 데이터를 직접 받아 커스텀 애플리케이션을 개발할 수 있도록 지원하는 C/C++ 기반 라이브러리다.7

SDK를 통해 다음과 같은 작업이 가능하다:

- **장치 제어:** Hub 및 개별 LiDAR의 파라미터 설정, 상태 쿼리, 펌웨어 업데이트 등 Viewer에서 가능한 거의 모든 제어 기능을 코드 레벨에서 수행할 수 있다.5

- **데이터 수신:** PTP 또는 GPS를 통해 정밀하게 동기화된 타임스탬프가 포함된 포인트 클라우드 데이터를 콜백(callback) 함수 형태로 수신할 수 있다. 데이터 포맷에 대한 상세한 내용은 SDK 통신 프로토콜 문서에 정의되어 있다.5

- **오픈소스 생태계:** SDK는 GitHub를 통해 소스 코드가 공개되어 있으며, 이를 기반으로 한 다양한 오픈소스 프로젝트들이 활발하게 개발되고 있다. 대표적으로 `livox_camera_lidar_calibration` 22, 

  `LIO-Livox` 32, 

  `horizon_highway_slam` 33 등은 SDK를 활용하여 센서 퓨전, SLAM, 자율주행 알고리즘을 구현한 좋은 예시다.

그러나 SDK 사용 시 잠재적인 불안정성 및 유지보수 문제를 고려해야 한다. Livox-SDK2의 GitHub 이슈 트래커에는 한 개발자가 Raspberry Pi와 같은 ARM 아키텍처 기반의 임베디드 시스템에서 SDK를 사용할 때, 메모리 정렬(memory alignment) 문제로 인해 예측 불가능한 `bus-error`가 발생하며 프로그램이 비정상 종료되는 현상을 보고한 사례가 있다.34 이 개발자는 문제의 원인으로 코드 내의 `pragma pack(1)` 지시어를 지목하며, 이를 수정한 자신의 Pull Request가 리뷰되지 않은 채 이슈가 종결된 것에 대해 "Livox-SDK2 저장소는 사실상 유지보수되지 않는 것 같다"고 비판했다.34

이는 Livox SDK가 모든 플랫폼, 특히 x86이 아닌 임베디드 환경에서 완벽한 안정성을 보장하지 않을 수 있음을 시사한다. 또한, 커뮤니티에서 제기된 문제나 기여에 대한 공식적인 기술 지원이 신속하게 이루어지지 않을 가능성도 내포한다. 따라서 상용 제품이나 고신뢰성이 요구되는 연구 프로젝트에 Livox SDK를 도입하려는 개발자는 특정 타겟 플랫폼에서의 호환성 및 안정성 검증을 철저히 수행해야 한다. 최악의 경우, SDK 내부 코드를 직접 디버깅하고 수정해야 할 가능성에 대비해야 하며, 이는 프로젝트의 개발 일정과 리스크 관리에 중요한 변수로 작용할 수 있다.


로봇 운영체제(Robot Operating System, ROS)는 로보틱스 연구 및 개발의 표준 플랫폼이다. Livox는 ROS 및 ROS2와의 원활한 통합을 위해 공식 드라이버인 `livox_ros_driver`와 `livox_ros_driver2`를 제공한다.14 Livox Hub는 구형 드라이버인 `livox_ros_driver`와 연동된다.

`livox_ros_driver` 패키지에는 Hub 연결을 위한 전용 launch 파일들이 포함되어 있어 손쉬운 설정이 가능하다. 대표적인 파일은 `livox_hub.launch`, `livox_hub_rviz.launch` (Rviz 시각화 포함), `livox_hub_msg.launch` (커스텀 메시지 포맷 사용) 등이다.36

이 launch 파일들을 사용할 때 핵심적인 파라미터는 다음과 같다:

- `data_src`: 데이터 소스를 지정한다. `0`으로 설정하면 LiDAR에 직접 연결, `1`로 설정하면 Livox Hub를 통해 연결한다.37
- `xfer_format`: 발행될 포인트 클라우드 메시지의 포맷을 결정한다. `0`은 표준 ROS 메시지인 `sensor_msgs/PointCloud2` 포맷을 사용하며, `1`은 타임스탬프 정보 등이 추가된 Livox 자체 커스텀 메시지(`livox_ros_driver/CustomMsg`) 포맷을 사용한다.35
- `multi_topic`: 여러 LiDAR의 데이터를 어떻게 발행할지 결정한다. `0`으로 설정하면 Hub에 연결된 모든 LiDAR의 포인트 클라우드가 하나의 토픽으로 통합되어 발행된다. `1`로 설정하면 각 LiDAR의 방송 코드(broadcast code)를 이름으로 하는 별도의 토픽으로 각각 발행된다.37
- `bd_list`: 연결할 LiDAR를 특정할 때 사용한다. 연결하고자 하는 LiDAR의 15자리 방송 코드를 문자열로 전달하면, 해당 장치들만 선택적으로 연결된다.36

Hub를 ROS와 연동할 때 가장 흔히 발생하는 문제는 네트워크 설정이다. Hub의 공장 출하 시 기본 IP 모드는 DHCP(동적 할당)이다.38 따라서 가장 간단하고 확실한 연결 방법은 Hub와 ROS가 실행되는 컴퓨터를 모두 DHCP 서버 기능이 있는 기가비트 라우터에 연결하는 것이다.7 만약 라우터 없이 Hub와 컴퓨터를 직접 연결하고 싶다면, 컴퓨터에 `isc-dhcp-server`와 같은 DHCP 서버 소프트웨어를 설치해야 한다.38 정적 IP 주소 모드로 사용하고 싶다면, 우선 DHCP 환경에서 Livox Viewer를 통해 Hub에 접속한 뒤, 설정 메뉴에서 IP 모드를 'Static'으로 변경하고 원하는 IP 주소를 할당해야 한다. 이 과정을 거친 후에야 정적 IP 환경에서 Hub를 사용할 수 있다.38


Livox Hub의 진정한 가치는 실제 애플리케이션, 특히 다수의 LiDAR 센서를 요구하는 복잡한 시스템에서 어떻게 활용되는지를 통해 평가될 수 있다. 자율주행, 로보틱스, SLAM 등 다양한 분야에서 Hub는 핵심적인 역할을 수행하며, 동시에 실제 운영 환경에서 발생하는 여러 문제점들을 통해 그 한계와 해결 방안을 엿볼 수 있다.


다중 LiDAR 시스템은 단일 센서로는 달성하기 어려운 넓은 시야각 확보와 높은 데이터 밀도 구현을 위해 필수적이다. Livox Hub는 이러한 시스템을 구축하는 데 있어 핵심적인 통합 솔루션으로 활용된다.

대표적인 사례로 중국의 물류 기업 JD Logistics가 구축한 L4 레벨 무인 배송 시스템이 있다.39 이 시스템은 차량 주변의 360도 환경을 완벽하게 인식하기 위해 여러 개의 Livox LiDAR를 사용하며, Hub는 이 센서들의 데이터를 통합하고 동기화하는 중추적인 역할을 담당했을 것으로 추정된다. 또한, 오픈소스 자율주행 플랫폼인 Autoware와의 통합 사례도 주목할 만하다. Livox는 5개의 Horizon 센서와 1개의 Hub를 사용하는 실험 환경을 구성하고, `livox_autoware_driver`를 통해 Autoware 플랫폼에 손쉽게 연동하여 지도 생성, 위치 추정, 경로 계획 등의 기능을 수행하는 예시를 공개했다.40 이는 Hub가 복잡한 자율주행 소프트웨어 스택과 원활하게 통합될 수 있음을 보여준다.

로보틱스 분야에서도 Hub의 필요성은 명확하다. 청소 로봇, 자율 지게차, 로봇 개, 잔디깎이 로봇 등 다양한 상용 로봇들이 주변 환경 인식 및 장애물 회피를 위해 Livox 센서를 채택하고 있다.39 이들 중 상당수는 전방위 감지나 사각지대 최소화를 위해 다중 센서 구성을 필요로 하며, 이는 Hub와 같은 통합 솔루션의 잠재적 수요를 시사한다. 연구 커뮤니티에서도 Hub는 다중 LiDAR 실험 플랫폼을 구축하는 데 널리 사용된다. 예를 들어, 5개의 Livox Horizon을 융합하여 SLAM 성능을 평가한 연구 41나, Ouster OS1과 Livox Horizon을 하나의 로버에 함께 탑재하여 성능을 비교한 연구 42 등에서 Hub는 데이터 수집 및 동기화의 복잡성을 줄여주는 중요한 도구로 기능했다.


SLAM(Simultaneous Localization and Mapping, 동시적 위치 추정 및 지도 작성)은 로봇이 미지의 환경에서 자신의 위치를 파악하고 동시에 주변 지도를 생성하는 기술이다. 다중 LiDAR를 사용하는 이유는 SLAM 시스템의 강건성(robustness)과 정밀도를 향상시키기 위함이다. 단일 센서의 제한된 시야각(FOV)을 극복하여 더 많은 환경 정보를 얻고 20, 특징이 부족한 복도나 터널, 또는 동적 객체가 많은 복잡한 환경에서도 안정적인 위치 추정을 가능하게 한다.41

이러한 다중 LiDAR SLAM 시스템에서 Livox Hub는 알고리즘의 복잡도를 크게 낮추는 역할을 한다. Hub가 시간적, 공간적으로 완벽하게 동기화된 단일 포인트 클라우드 스트림을 제공하기 때문에, SLAM 알고리즘 개발자는 여러 센서의 데이터를 개별적으로 수신하고, 타임스탬프를 맞추고, 좌표 변환을 수행하는 복잡한 전처리 과정에서 해방될 수 있다. 대신, 이미 정제된 통합 데이터에 집중하여 LIO-SAM 41, FAST-LIO 43와 같은 핵심 SLAM 알고리즘을 적용하기만 하면 된다.

`LIO-Livox` 32나 `horizon_highway_slam` 33과 같은 Livox 기반의 오픈소스 SLAM 프로젝트들은 Hub가 제공하는 통합 데이터 스트림을 기반으로 설계되었다. 특히 `horizon_highway_slam`은 고속도로 환경과 같이 매우 빠르고 동적인 시나리오에서 Hub와 다중 Horizon 센서를 사용하여 안정적인 SLAM 성능을 보여주며, 이는 Hub의 실시간 데이터 처리 및 동기화 능력의 우수성을 입증하는 사례다. 다수의 Livox Horizon을 융합하여 기존 기계식 LiDAR 기반 SLAM(LeGO-LOAM, A-LOAM)과 성능을 비교한 연구에서는, 다중 Livox 시스템이 특히 동적 객체가 많은 어려운 환경에서 더 우수한 강건성을 보임을 입증했다.41 이는 Hub를 통해 구현된 다중 센서 융합이 SLAM 성능 향상에 직접적으로 기여함을 보여준다.


Livox Hub를 실제 시스템에 통합하고 운영하는 과정에서는 여러 문제에 직면할 수 있다. 커뮤니티 포럼과 GitHub 이슈 트래커를 통해 보고된 주요 문제점과 그 해결 방안은 다음과 같다.

네트워크 설정 문제 (Network Configuration Issues)

가장 빈번하게 발생하는 문제로, Hub의 기본 IP 모드가 DHCP(동적)라는 사실을 인지하지 못하고 정적 IP 환경의 PC와 직접 연결하여 Hub가 인식되지 않는 경우다.38

- **해결책:** 가장 확실한 방법은 기가비트 라우터를 사용하여 Hub와 PC 모두 DHCP 환경에서 IP를 할당받도록 하는 것이다.7 또는, ROS를 구동하는 Ubuntu PC에 

  `isc-dhcp-server`와 같은 DHCP 서버 패키지를 설치하여 라우터 없이 직접 연결하는 방법도 유효하다.38 만약 정적 IP 사용이 반드시 필요하다면, 먼저 DHCP 환경에서 Livox Viewer로 Hub에 접속하여 IP 모드를 'Static'으로 변경하고 원하는 고정 IP를 설정한 후, 네트워크 환경을 재구성해야 한다. 또한, PC의 방화벽이 LiDAR 데이터 패킷을 차단할 수 있으므로, 테스트 시에는 방화벽을 비활성화하거나 Livox Viewer/SDK에 대한 예외 규칙을 추가하는 것이 권장된다.19

소프트웨어 버전 불일치 (Software Version Mismatch)

앞서 언급된 소프트웨어 생태계의 파편화 문제로 인해, 사용 중인 센서 모델과 호환되지 않는 버전의 Livox Viewer나 ROS 드라이버를 사용하여 장치가 인식되지 않는 문제가 발생할 수 있다.30

- **해결책:** Livox 공식 웹사이트의 다운로드 페이지에서 자신이 보유한 센서 모델(예: Horizon, Mid-70)의 제품 리소스 섹션을 확인하고, 거기에 명시된 정확한 버전의 Viewer와 드라이버를 다운로드하여 설치해야 한다.10

**물리적/환경적 문제 (Physical/Environmental Issues)**

- **광학 창 오염:** LiDAR 센서의 광학 창에 묻은 먼지나 얼룩은 포인트 클라우드의 품질을 저하시키고 노이즈를 유발한다.31 청소 시에는 먼저 압축 공기를 사용해 큰 입자들을 불어낸 후, 부드러운 렌즈 티슈로 조심스럽게 닦아내야 한다.31
- **발열:** LiDAR 센서와 Hub 모두 작동 시 상당한 열을 발생시킨다. 특히 LiDAR 센서는 알루미늄 합금과 같이 열전도율이 좋은 금속판에 장착하여 방열을 돕는 것이 좋다.44 Hub 역시 '방열 영역(Dissipation Area)'이 막히지 않도록 통풍이 잘 되는 곳에 설치해야 한다.2
- **모션 블러:** Livox 센서의 고유한 비반복 스캔 패턴은 센서가 빠르게 움직일 때 포인트 클라우드에 왜곡(motion distortion)이나 번짐(blur) 현상을 유발할 수 있다.46 이는 Hub의 문제가 아닌 센서 자체의 특성이지만, Hub를 통해 통합된 데이터에도 동일하게 나타난다. 이 문제를 해결하기 위해서는 SLAM 알고리즘 내에서 IMU 데이터를 활용한 정밀한 모션 보상(motion distortion compensation) 기법을 적용하는 것이 필수적이다.32


Livox Hub는 다중 LiDAR 시스템 구축이라는 복잡한 엔지니어링 과제에 대한 Livox의 초기 해답이었다. 그 강점과 약점을 종합적으로 평가하고, 현재 시점에서 이 장치를 어떻게 활용해야 할지에 대한 전략적 제언을 통해 이 보고서를 마무리한다.


**강점 (Strengths):**

1. **통합의 단순성 (Simplicity of Integration):** 전원 분배, 데이터 집선, 시간 및 공간 동기화라는 세 가지 복잡한 요구사항을 하나의 견고한 하드웨어 장치로 해결했다. 이는 다중 LiDAR 시스템의 물리적 구성과 초기 설정에 드는 시간과 노력을 극적으로 줄여준다. 개발자는 복잡한 케이블링과 개별적인 전원 및 데이터 관리 대신, Hub라는 단일 인터페이스에 집중할 수 있다.
2. **고정밀 시간 동기화 (High-Precision Time Synchronization):** PTP(IEEE 1588v2)와 GPS 동기화 방식을 모두 지원함으로써, 실내 로보틱스부터 실외 자율주행에 이르기까지 광범위한 애플리케이션에서 요구하는 마이크로초(μs) 수준의 시간 정밀도를 보장한다. 이는 다중 센서 데이터 융합의 신뢰성을 높이는 핵심적인 요소다.
3. **실시간 처리 성능 (Real-time Processing):** 여러 LiDAR의 데이터를 집선하고, 사용자가 입력한 외부 파라미터를 적용하여 좌표 변환을 수행하는 과정을 하드웨어 레벨에서 처리한다. 이는 호스트 PC의 CPU 부하를 크게 줄여주어, PC가 SLAM이나 객체 인식과 같은 고수준의 인식 알고리즘에 더 많은 연산 자원을 할당할 수 있도록 한다.

**약점 (Weaknesses):**

1. **세대적 한계 (Generational Limitation):** 가장 치명적인 약점으로, 2020년 이후 출시된 Livox의 신형 센서들(HAP, Mid-360 등)과 호환되지 않는다.10 이는 Hub가 미래 지향적인 솔루션이 아닌, 특정 과거 세대 제품군을 위한 레거시(legacy) 장비임을 의미한다.
2. **소프트웨어 생태계 파편화 (Fragmented Software Ecosystem):** 제품 세대에 따라 Livox Viewer와 ROS 드라이버가 다른 버전으로 나뉘어 있어 사용자에게 혼란을 준다.10 이는 일관성 있는 개발 및 유지보수 환경을 저해하는 요소다.
3. **초기 설정의 복잡성 (Initial Setup Complexity):** 특히 네트워크 설정(DHCP vs Static IP)과 관련하여 공식 문서의 설명이 부족하거나 혼선을 주어, 처음 사용하는 개발자가 연결 문제를 겪을 가능성이 높다.38
4. **'자동 캘리브레이션' 기능 부재 (Lack of 'Auto-Calibration'):** '공간 동기화'라는 용어가 마치 Hub가 자동으로 센서 간의 위치 관계를 계산해주는 것처럼 오해를 불러일으킬 수 있다. 실제로는 사용자가 별도의 툴과 노력을 통해 획득한 캘리브레이션 파라미터를 수동으로 입력해야만 하는, '적용' 장치에 불과하다.1


Livox Hub를 성공적으로 시스템에 통합하고 안정적으로 운영하기 위해서는 다음과 같은 가이드라인을 따르는 것이 좋다.

**설치 전 체크리스트:**

- **호환성 확인:** 시스템에 사용하려는 LiDAR 모델이 Hub와 공식적으로 호환되는지(Mid-40, Mid-100, Horizon, Tele-15) 다시 한번 확인한다.8
- **전원 설계:** 연결할 LiDAR의 수량과 모델을 기준으로 시스템의 최대 및 평균 전력 소모량을 계산하고, 충분한 마진을 가진 전원 공급 장치(PSU)를 선정한다.2
- **열 관리 설계:** Hub 자체의 방열 영역과 각 LiDAR 센서의 발열을 고려하여, 시스템 전체의 통풍 구조나 방열판, 냉각 팬 등의 열 관리 방안을 사전에 설계한다.2

**네트워크 설정:**

- **라우터 우선 사용:** 초기 연결 및 테스트 시에는 별도의 기가비트 라우터를 사용하여 DHCP 환경을 구성하는 것이 가장 간단하고 확실한 방법이다.7
- **방화벽 비활성화:** 호스트 PC의 방화벽이 데이터 통신을 방해할 수 있으므로, 테스트 중에는 방화벽을 비활성화하거나 Livox 관련 프로그램에 대한 예외 규칙을 명시적으로 추가한다.19
- **정적 IP 설정 절차 준수:** 정적 IP 환경이 필수적인 경우, 반드시 DHCP로 먼저 연결하여 Viewer를 통해 Hub의 IP 모드를 'Static'으로 변경하고 원하는 IP 주소를 설정한 후, 네트워크 환경을 재구성하는 절차를 따른다.38

**소프트웨어 및 캘리브레이션:**

- **정확한 소프트웨어 버전 사용:** Livox 다운로드 페이지에서 사용 중인 센서 모델에 맞는 정확한 버전의 Livox Viewer 및 ROS 드라이버를 다운로드하여 설치한다.10
- **캘리브레이션 별도 수행:** 다중 LiDAR 간의 외부 파라미터(Extrinsics)를 정밀하게 캘리브레이션하는 절차를 프로젝트의 독립적인 중요 과업으로 설정하고 계획을 수립한다. Livox가 제공하는 캘리브레이션 툴이나 다른 오픈소스 툴을 활용하여 파라미터를 획득한 후, 그 결과값을 Viewer나 SDK를 통해 Hub에 정확하게 입력한다.


Livox Hub는 분명 편리한 통합 솔루션이지만, 유일한 방법은 아니다. 특히 최신 센서를 사용하거나 더 높은 수준의 시스템 유연성이 필요할 경우, Hub의 역할을 분해하여 대안 시스템을 구축하는 것을 고려해볼 수 있다.

Hub의 세 가지 핵심 역할(전원 분배, 데이터 집선, 동기화)을 개별 구성 요소로 대체하는 시나리오는 다음과 같다:

1. **전원 분배:** 로보틱스나 드론에서 흔히 사용되는 전원 분배 보드(Power Distribution Board, PDB)나 다중 출력 채널을 가진 산업용 PSU를 사용하여 각 LiDAR에 개별적으로 안정적인 전원을 공급할 수 있다. 이는 각 센서의 전원을 더 세밀하게 제어하거나 다른 종류의 센서들과 전원 시스템을 공유하는 데 있어 더 높은 유연성을 제공한다.
2. **데이터 집선:** 시중에서 쉽게 구할 수 있는 고성능 기가비트 이더넷 스위치(Gigabit Ethernet Switch)를 사용하여 모든 LiDAR를 단일 네트워크에 연결할 수 있다. 호스트 PC는 이 스위치를 통해 모든 LiDAR의 데이터 스트림을 동시에 수신한다.
3. **동기화:** 이 부분이 가장 핵심적인 도전 과제다.
   - **시간 동기화:** PTP를 지원하는 'PTP-aware' 네트워크 스위치와 PTP 그랜드마스터 클럭(전용 장비 또는 GPS와 연동된 서버)을 사용하면, 네트워크에 연결된 각 LiDAR(PTP 슬레이브로 동작)가 개별적으로 마스터 클럭과 정밀하게 동기화될 수 있다. 이는 Hub 없이도 PTP 기반의 시간 동기화를 구현하는 표준적인 방법이다.
   - **공간 동기화:** 이 역할은 전적으로 호스트 PC의 소프트웨어가 담당해야 한다. 즉, ROS 노드나 커스텀 애플리케이션이 각 LiDAR 토픽에서 데이터를 수신할 때마다, 사전에 계산된 외부 파라미터 변환 행렬을 적용하여 좌표계를 일치시켜야 한다. 이는 Hub가 하드웨어로 처리해주던 연산 부하를 소프트웨어가 감당해야 함을 의미하며, 시스템의 전체적인 CPU 사용률을 높이는 요인이 될 수 있다.

결론적으로, Livox Hub는 다중 LiDAR 시스템 구축의 복잡성을 하나의 '플러그 앤 플레이'에 가까운 패키지로 해결했다는 점에서, 특히 해당 세대의 센서를 사용하는 개발자에게 여전히 높은 가치를 지닌다. 하지만 더 많은 수의 센서를 연결하거나, 다른 종류의 센서(카메라, IMU)와 더 긴밀하게 통합하거나, 최신 Livox 센서를 사용해야 하는 등 더 높은 유연성이 요구되는 시나리오에서는 각 기능을 분리하여 맞춤형 시스템을 구축하는 것이 더 합리적인 선택일 수 있다. 특히 PTP 지원 스위치가 보편화되고 가격이 하락한 현시점에서, Hub의 핵심 가치 중 일부는 다른 기성품으로 충분히 대체 가능하다. 따라서 Livox Hub의 도입 결정은 '통합의 편리성'과 '시스템 설계의 유연성 및 확장성' 사이의 트레이드오프를 신중하게 고려하여 이루어져야 할 것이다.


1. Livox Hub, accessed July 22, 2025, https://www.livoxtech.com/hub
2. Livox Hub, accessed July 22, 2025, [https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/20190218/Livox%20Hub%20Quick%20Start%20Guide_v1.0_20190218.pdf](https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/20190218/Livox Hub Quick Start Guide_v1.0_20190218.pdf)
3. www.livoxtech.com, accessed July 22, 2025, [https://www.livoxtech.com/hub#:~:text=The%20Livox%20Hub%20can%20access,cloud%20data%20samples%20per%20second.](https://www.livoxtech.com/hub#:~:text=The Livox Hub can access,cloud data samples per second.)
4. Livox LiDAR Hub - Connect to up to Nine sensor units (NEW in OB) - eBay, accessed July 22, 2025, https://www.ebay.com/itm/316906573208
5. Livox Hub, accessed July 22, 2025, [https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/20191128/Livox%20Hub%20Series%20User%20Manual%2020191018.pdf](https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/20191128/Livox Hub Series User Manual 20191018.pdf)
6. Livox Hub specs - Livox, accessed July 22, 2025, https://www.livoxtech.com/hub/specs
7. Livox Hub, accessed July 22, 2025, [https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/20190318/Livox%20Hub%20Quick%20Start%20Guide%20v1.0.pdf](https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/20190318/Livox Hub Quick Start Guide v1.0.pdf)
8. LIVOX Hub (P08) - koningdrone.be, accessed July 22, 2025, https://koningdrone.be/en/product/livox-hub-p08/
9. Livox Hub - AEROMOTUS, accessed July 22, 2025, https://www.aeromotus.com/product/livox-hub/
10. Download Center. - Livox, accessed July 22, 2025, https://www.livoxtech.com/downloads
11. Livox Hub Downloads - Livox, accessed July 22, 2025, https://www.livoxtech.com/hub/downloads
12. Livox: LiDAR Sensors, accessed July 22, 2025, https://www.livoxtech.com/
13. Livox HAP LiDAR, accessed July 22, 2025, https://www.livoxtech.com/hap
14. Livox-SDK/livox_ros_driver2: Livox device driver under Ros(Compatible with ros and ros2), support Lidar HAP and Mid-360. - GitHub, accessed July 22, 2025, https://github.com/Livox-SDK/livox_ros_driver2
15. livox device time synchronization manual - GitHub, accessed July 22, 2025, https://github.com/Livox-SDK/Livox-SDK/wiki/livox-device-time-synchronization-manual
16. IEEE-1588 PTP(정밀 시간 프로토콜)를 통한 정밀 시스템 동기화 - Teledyne Vision Solutions, accessed July 22, 2025, https://www.teledynevisionsolutions.com/ko-kr/learn/learning-center/machine-vision/precision-system-synchronization-with-the-ieee-1588-precision-time-protocol-ptp/
17. 시간 서버 구축하기: PTP와 NTP를 이용한 네트워크 시간 동기화 - 쵸코쿠키의 연습장, accessed July 22, 2025, https://jjeongil.tistory.com/2757
18. Welcome to LIVOX ! - Livox wiki 0.1 documentation, accessed July 22, 2025, https://livox-wiki-en.readthedocs.io/
19. Livox Viewer User Manual, accessed July 22, 2025, [https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/Livox%20Viewer/Livox%20Viewer%20User%20Manual.pdf](https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/downloads/Livox Viewer/Livox Viewer User Manual.pdf)
20. livox_wiki_en/source/tutorials/other_product/sensor_calibration.rst at master - GitHub, accessed July 22, 2025, https://github.com/Livox-SDK/livox_wiki_en/blob/master/source/tutorials/other_product/sensor_calibration.rst
21. Livox Horizon, accessed July 22, 2025, [https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/assets/horizon/Livox%20Horizon%20user%20manual%20v1.0.pdf](https://www.livoxtech.com/3296f540ecf5458a8829e01cf429798e/assets/horizon/Livox Horizon user manual v1.0.pdf)
22. Livox-SDK/livox_camera_lidar_calibration: Calibrate the extrinsic parameters between Livox LiDAR and camera - GitHub, accessed July 22, 2025, https://github.com/Livox-SDK/livox_camera_lidar_calibration
23. Multi-LiCa: A Motion- and Targetless Multi - LiDAR-to-LiDAR Calibration Framework - arXiv, accessed July 22, 2025, https://arxiv.org/html/2501.11088v1
24. manual.dewesoft.com, accessed July 22, 2025, [https://manual.dewesoft.com/x/setupmodule/devices/velodyne-lidar/velodyne-lidar-setup/velodyne-lidar-sync#:~:text=Velodyne%20can%20be%20synchronized%20with,connected%20to%20the%20GNSS%20device.](https://manual.dewesoft.com/x/setupmodule/devices/velodyne-lidar/velodyne-lidar-setup/velodyne-lidar-sync#:~:text=Velodyne can be synchronized with,connected to the GNSS device.)
25. Synchronization > Device and Software Setup > Velodyne Lidar > Devices > Setup | Dewesoft X Manual EN, accessed July 22, 2025, https://manual.dewesoft.com/x/setupmodule/devices/velodyne-lidar/velodyne-lidar-setup/velodyne-lidar-sync
26. Data synchronization based on GPS time stamps - Google Groups, accessed July 22, 2025, https://groups.google.com/g/ros-sig-drivers/c/CgX-FT3jUQ4
27. PTP Profiles Guide - Ouster Sensor Docs documentation, accessed July 22, 2025, https://static.ouster.dev/sensor-docs/image_route1/image_route2/appendix/ptp-profile-guide.html
28. Possible reasons that might cause PTP time synchronisation failure - Software - Ouster, accessed July 22, 2025, https://community.ouster.com/t/possible-reasons-that-might-cause-ptp-time-synchronisation-failure/133
29. Appendix - PTP Guide - Ouster Sensor Docs documentation, accessed July 22, 2025, https://static.ouster.dev/sensor-docs/image_route1/image_route3/appendix/appendix.html
30. Livox Mid 70 - Faulty Unit? : r/LiDAR - Reddit, accessed July 22, 2025, https://www.reddit.com/r/LiDAR/comments/1ezdmvz/livox_mid_70_faulty_unit/
31. Livox Mid-360 lidar Minimal Detection Range User Manual, accessed July 22, 2025, https://manuals.plus/livox/mid-360-lidar-minimal-detection-range-manual
32. Livox-SDK/LIO-Livox: A Robust LiDAR-Inertial Odometry for Livox LiDAR - GitHub, accessed July 22, 2025, https://github.com/Livox-SDK/LIO-Livox
33. Livox-SDK/horizon_highway_slam: Horizon_Highway_Slam Demo in Docker - GitHub, accessed July 22, 2025, https://github.com/Livox-SDK/horizon_highway_slam
34. livox_lidar_quick_start crashes on Rasbpian Bulleye 32 / Issue #33 / Livox-SDK/Livox-SDK2, accessed July 22, 2025, https://github.com/Livox-SDK/Livox-SDK2/issues/33
35. livox_ros_driver - ROS Wiki, accessed July 22, 2025, http://wiki.ros.org/livox_ros_driver
36. Livox-SDK/livox_ros_driver: Livox device driver under ros, support Lidar Mid-40, Mid-70, Tele-15, Horizon, Avia. - GitHub, accessed July 22, 2025, https://github.com/Livox-SDK/livox_ros_driver
37. ROS Package: livox_ros_driver, accessed July 22, 2025, https://index.ros.org/p/livox_ros_driver/
38. Livox Hub connection issue plz help.. / Issue #81 / Livox-SDK/livox_ros_driver - GitHub, accessed July 22, 2025, https://github.com/Livox-SDK/livox_ros_driver/issues/81
39. Robot Application Cases - LiDAR Sensors - Livox, accessed July 22, 2025, https://www.livoxtech.com/application/robotics
40. Livox Lidars fully compatible with Autoware platform, accessed July 22, 2025, https://www.livoxtech.com/news/13
41. A Robust Framework for Simultaneous Localization and Mapping with Multiple Non-Repetitive Scanning Lidars - MDPI, accessed July 22, 2025, https://www.mdpi.com/2072-4292/13/10/2015
42. Rover equipped with an Ouster OS1 32 and a Livox Horizon LiDAR for... - ResearchGate, accessed July 22, 2025, https://www.researchgate.net/figure/Rover-equipped-with-an-Ouster-OS1-32-and-a-Livox-Horizon-LiDAR-for-autonomous-navigation_fig2_372708700
43. A Novel Dataset and Comparative Study of Solid-State and Spinning Lidars - arXiv, accessed July 22, 2025, https://arxiv.org/html/2507.04321v1
44. LIVOX Mid-360 LiDAR Sensor User Guide - Manuals.plus, accessed July 22, 2025, https://manuals.plus/livox/mid-360-lidar-sensor-manual
45. Livox Mid-360 or Robosense Airy : r/LiDAR - Reddit, accessed July 22, 2025, https://www.reddit.com/r/LiDAR/comments/1li59kb/livox_mid360_or_robosense_airy/
46. Livox Mid-40 lidar first review! with pictures : r/SelfDrivingCars - Reddit, accessed July 22, 2025, https://www.reddit.com/r/SelfDrivingCars/comments/aimdqz/livox_mid40_lidar_first_review_with_pictures/


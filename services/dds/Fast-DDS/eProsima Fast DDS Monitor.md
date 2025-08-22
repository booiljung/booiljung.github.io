---
layout: page
title: eProsima Fast DDS Monitor
permalink: /services/dds/Fast-DDS/eProsima Fast DDS Monitor
---



현대 로보틱스 시스템, 특히 자율주행 자동차, 협동 로봇, 드론 군집과 같은 복잡한 시스템은 본질적으로 분산 컴퓨팅 환경입니다. 다수의 센서, 액추에이터, 연산 장치가 실시간으로 데이터를 교환하며 상호작용해야 합니다. 이러한 환경에서 각 구성 요소(노드) 간의 안정적이고 효율적인 통신은 시스템 전체의 성능과 신뢰성을 좌우하는 핵심 요소입니다. 미들웨어는 바로 이 지점에서 운영체제와 응용 프로그램 사이의 다리 역할을 수행하며, 개발자가 통신의 복잡한 저수준 세부 사항에 얽매이지 않고 응용 프로그램의 핵심 로직에 집중할 수 있도록 돕습니다.

이러한 미들웨어 중 데이터 분배 서비스(Data Distribution Service, DDS)는 객체 관리 그룹(Object Management Group, OMG)에 의해 표준화된 데이터 중심(data-centric)의 발행-구독(publish-subscribe) 통신 프로토콜입니다. DDS는 실시간성과 신뢰성이 중요한 항공, 국방, 산업 자동화 및 로보틱스 분야에서 널리 채택되고 있습니다. 로보틱스 분야의 사실상 표준 프레임워크인 로봇 운영 체제 2(Robot Operating System 2, ROS 2)는 바로 이 DDS를 핵심 통신 미들웨어로 채택하여, 노드 간의 강력하고 유연한 통신을 보장합니다.1

하지만 DDS의 강력한 기능은 복잡성을 동반합니다. 수십 가지의 서비스 품질(Quality of Service, QoS) 설정, 동적 개체 발견(dynamic discovery) 메커니즘, 다양한 전송 프로토콜 등은 분산 시스템 개발 및 디버깅 과정에서 여러 가지 어려움을 야기합니다. 통신의 비결정성, 네트워크로 인한 장애, 성능 병목 현상 등은 예측하고 해결하기 어려운 문제들입니다.


분산 시스템의 개발 및 디버깅 과정에서 발생하는 문제들은 응용 프로그램 코드 자체의 버그일 수도 있고, 미들웨어 설정(QoS)의 오류일 수도 있으며, 혹은 근본적인 네트워크 인프라의 문제일 수도 있습니다. 이러한 문제의 원인을 명확히 구분하지 못하면 디버깅은 추측과 반복적인 시도에 의존하는 비효율적인 과정이 될 수밖에 없습니다. 따라서 통신 시스템의 내부를 실시간으로 들여다볼 수 있는 능력, 즉 '내성(Introspection)'이 반드시 필요합니다.

시스템의 내성을 확보하기 위해서는 핵심 성능 지표(Key Performance Metrics, KPMs)를 지속적으로 모니터링해야 합니다. 대표적인 지표는 다음과 같습니다.1

- **지연 시간 (Latency):** 데이터가 송신자(Publisher)로부터 수신자(Subscriber)에게 도달하기까지 걸리는 시간. 실시간 제어 시스템에서 가장 중요한 지표 중 하나입니다.
- **처리율 (Throughput):** 단위 시간당 전송되는 데이터의 양. 고해상도 카메라 이미지나 라이다(LiDAR) 포인트 클라우드와 같은 대용량 데이터를 처리할 때 핵심적인 지표입니다.
- **패킷 손실 (Packet Loss):** 전송 과정에서 유실되는 데이터 패킷의 비율. 신뢰성 있는 통신을 보장하는 데 중요한 척도입니다.
- **자원 소모량 (Resource Consumption):** 통신 과정에서 소모되는 CPU 및 메모리 자원의 양. 임베디드 시스템과 같이 자원이 제한된 환경에서 중요합니다.

이러한 지표들을 정량적으로 분석할 수 있을 때, 개발자는 시스템의 병목 현상을 식별하고, QoS 설정을 최적화하며, 통신 관련 문제를 체계적으로 해결할 수 있습니다.


eProsima Fast DDS Monitor는 eProsima사의 Fast DDS 라이브러리를 사용하는 DDS 환경을 실시간으로 모니터링하기 위해 특별히 설계된 오픈 소스 그래픽 데스크톱 응용 프로그램입니다.1 이 도구의 존재 자체가 DDS와 같은 미들웨어를 '블랙박스'처럼 취급하는 것만으로는 전문적인 로보틱스 시스템 개발 및 디버깅에 충분하지 않다는 것을 시사합니다. 즉, 개발자는 데이터 플레인과 제어 플레인에 대한 가시성을 확보해야만 견고한 시스템을 구축할 수 있으며, Fast DDS Monitor는 바로 이러한 필요를 충족시키기 위해 탄생했습니다.

Fast DDS Monitor는 다음과 같은 네 가지 핵심 설계 철학을 기반으로 개발되었습니다.4

1. **모니터링 (Monitoring):** 네트워크 상태와 DDS 통신을 실시간으로 추적합니다.
2. **직관성 (Intuitive):** 사용자 경험(UX) 설계 접근법에 따라 개발된 그래픽 사용자 인터페이스(GUI)를 제공하여 복잡한 데이터를 쉽게 이해할 수 있도록 합니다.
3. **내성 (Introspection):** 배포되고 활성화된 모든 DDS 개체(entity)들을 쉽게 탐색하고, 그들의 구성 및 물리적 배포 상태를 상세히 검사할 수 있게 합니다.
4. **문제 해결 (Troubleshooting):** 통신 중에 발생할 수 있는 잠재적인 문제나 비정상적인 이벤트를 신속하게 감지할 수 있도록 지원합니다.

결론적으로, Fast DDS Monitor는 단순한 부가 기능이 아니라, 전문적인 로보틱스 개발 툴체인의 필수적인 구성 요소로 자리매김합니다. 이 도구를 통해 개발자는 추측에 기반한 디버깅에서 벗어나, 데이터에 기반한 체계적인 시스템 분석 및 문제 해결을 수행할 수 있습니다.


Fast DDS Monitor의 강력한 기능들을 효과적으로 활용하기 위해서는 그 내부 작동 원리를 이해하는 것이 필수적입니다. Monitor는 독립적으로 작동하는 응용 프로그램이 아니라, Fast DDS 라이브러리 자체의 특정 기능과 긴밀하게 연동되는 클라이언트 응용 프로그램입니다. 이 아키텍처를 이해하는 것은 가장 흔한 사용자 문제의 근본 원인을 파악하는 열쇠가 됩니다.


Fast DDS Monitor 기능의 근간은 Fast DDS 라이브러리 내부에 내장된 **통계 모듈(Statistics Module)**입니다.6 이 모듈은 DDS 통신과 관련된 다양한 성능 지표와 메타데이터를 수집하는 역할을 담당합니다. 여기서 가장 중요한 사실은, 이 통계 모듈이 기본적으로 비활성화되어 있다는 점입니다. 따라서 Monitor가 어떤 데이터라도 수신하기 위해서는, 응용 프로그램이 사용하는 Fast DDS 라이브러리를 컴파일할 때 반드시 통계 모듈을 명시적으로 활성화해야 합니다. 이는 CMake 빌드 옵션에서 `-DFASTDDS_STATISTICS=ON` 플래그를 설정함으로써 이루어집니다.8

이러한 설계는 성능에 민감한 사용자를 위한 배려입니다. 통계 데이터를 수집하고 발행하는 과정은 필연적으로 약간의 CPU 및 네트워크 오버헤드를 발생시킵니다. 따라서 모니터링이 필요 없는 배포 환경에서는 이 기능을 완전히 비활성화하여 오버헤드를 원천적으로 차단할 수 있습니다.


통계 모듈이 활성화되면, 이 모듈은 자체적인 내부 DDS 토픽들을 생성하여 수집된 통계 데이터를 발행합니다.7 Fast DDS Monitor는 이 특별한 통계 토픽들을 구독하는 단순한 DDS 구독자 응용 프로그램처럼 작동합니다. 이 방식은 Monitor가 사용자의 주 응용 프로그램 토픽과 완전히 분리되어 작동하도록 보장하며, 모니터링 행위가 응용 프로그램의 데이터 흐름에 직접적인 영향을 미치지 않게 합니다.

통계 모듈이 발행하는 주요 내장 토픽들은 다음과 같습니다 9:

- `HISTORY_LATENCY_TOPIC`: 개별 데이터 샘플의 종단 간(end-to-end) 지연 시간 정보를 담고 있습니다.
- `PUBLICATION_THROUGHPUT_TOPIC`: 각 DataWriter의 초당 발행 데이터양(bytes/sec)을 보고합니다.
- `SUBSCRIPTION_THROUGHPUT_TOPIC`: 각 DataReader의 초당 수신 데이터양(bytes/sec)을 보고합니다.
- `RTPS_SENT_TOPIC`: RTPS(Real-Time Publish-Subscribe Protocol) 계층에서 전송된 패킷 수를 보고합니다.
- `RTPS_LOST_TOPIC`: RTPS 계층에서 손실된 패킷 수를 보고하여 네트워크 신뢰성 문제를 진단하는 데 사용됩니다.
- `HEARTBEAT_COUNT_TOPIC`: 신뢰성 있는(Reliable) 통신에서 DataWriter가 보내는 Heartbeat 메시지의 수를 집계합니다.
- `ACKNACK_COUNT_TOPIC`: 신뢰성 있는 통신에서 DataReader가 보내는 ACKNACK/NACK 메시지의 수를 집계하여 재전송 요청 빈도를 파악하게 합니다.
- `PHYSICAL_DATA_TOPIC`: DDS 개체가 실행되고 있는 물리적 환경 정보(호스트 이름, 사용자, 프로세스 ID 등)를 전달합니다.

이처럼 시스템이 자기 자신의 상태를 모니터링하기 위해 바로 그 자신(DDS 프로토콜)을 사용하는 것은 매우 우아하고 자기 참조적인(self-referential) 설계입니다. 이는 Monitor가 표준 DDS 응용 프로그램임을 의미하며, 동시에 Monitor 자체의 통신 또한 진단 대상이 되는 네트워크 문제의 영향을 받을 수 있음을 시사합니다.


Fast DDS Monitor의 내부 아키텍처는 다음과 같은 구성 요소들의 유기적인 결합으로 이루어집니다:

1. **DDS 클라이언트 엔진:** 응용 프로그램의 핵심부에는 지정된 DDS 도메인에 참여하는 `DomainParticipant`가 있습니다. 이 Participant는 위에서 언급된 모든 통계 토픽을 구독하기 위한 `DataReader`들을 생성합니다.7
2. **인메모리 데이터베이스:** `DataReader`를 통해 수신된 시계열(time-series) 통계 데이터는 Monitor 내부의 인메모리 데이터베이스에 저장됩니다. 이를 통해 과거 데이터 조회 및 실시간 데이터 분석이 가능해집니다.7
3. **GUI 프론트엔드:** 사용자가 보는 모든 화면, 즉 패널과 차트는 Qt 프레임워크 기반의 GUI 응용 프로그램입니다.12 이 프론트엔드는 내부 인메모리 데이터베이스에 질의(query)하여 얻은 데이터를 시각적으로 렌더링하는 역할을 합니다.

이 아키텍처는 "통계 모듈이 없으면, 통계 토픽도 없고, 따라서 Monitor에 표시할 데이터도 없다"는 명확한 인과 관계를 보여줍니다. 이 간단한 논리 사슬은 사용자들이 겪는 가장 흔한 문제, 즉 "왜 아무것도 보이지 않나요?"에 대한 근본적인 해답입니다. 많은 사용자들이 Monitor 응용 프로그램을 다운로드하여 실행하기만 하면 모든 것이 마법처럼 작동할 것이라 기대하지만, 이는 사실이 아닙니다.8 Monitor는 수동적인 시각화 도구일 뿐이며, Fast DDS 라이브러리가 통계 토픽을 통해 데이터를 능동적으로 발행해야만 그 기능을 발휘할 수 있습니다.


Fast DDS Monitor는 분산 시스템을 다층적인 관점에서 이해할 수 있도록 계층적인 뷰를 제공합니다. 이 뷰는 `PHYSICAL_DATA_TOPIC`과 표준 DDS 발견 프로토콜(PDP/EDP)을 통해 수집된 정보를 바탕으로 구성됩니다.5

- **DDS 개체 (DDS Entities):** DDS 표준에 정의된 핵심 구성 요소들입니다. 네트워크에서 발견된 `DomainParticipant`와 그에 속한 `DataWriter`, `DataReader`들이 이 뷰에 표시됩니다. 이는 통신의 가장 기본적인 단위들을 보여줍니다.
- **논리적 개체 (Logical Entities):** 통신을 개념적으로 분할하는 추상적인 단위들입니다. `Domain`은 독립적인 통신 공간을, `Topic`은 데이터가 흐르는 채널을 나타냅니다. 이 뷰는 데이터 공간이 어떻게 구성되어 있는지를 이해하는 데 도움을 줍니다.
- **물리적 개체 (Physical Entities):** DDS 개체들이 실제로 어디에서 실행되고 있는지를 보여주는 물리적 배포 정보입니다. `Host`(머신) 안에 특정 `User`가 있고, 그 사용자가 실행한 `Process` 안에 DDS `DomainParticipant`가 존재하는 계층 구조를 명확히 보여줍니다.9 이 정보는 여러 장비에 걸쳐 시스템이 배포되었을 때 문제의 물리적 위치를 파악하는 데 매우 중요합니다.

이러한 다층적 뷰는 개발자가 추상적인 통신 구조(논리적 뷰)와 실제 하드웨어 배포(물리적 뷰), 그리고 그 둘을 연결하는 미들웨어 개체(DDS 뷰) 간의 관계를 직관적으로 파악할 수 있도록 돕는 강력한 기능입니다.


Fast DDS Monitor를 성공적으로 사용하기 위한 첫걸음은 올바른 설치와 시스템 환경 설정입니다. 이 장에서는 운영체제별 설치 방법과, 가장 중요하지만 종종 간과되는 ROS 2 연동을 위한 시스템 준비 과정을 상세히 다룹니다.


Fast DDS Monitor를 소스 코드로부터 직접 빌드하기 위해서는 여러 가지 의존성 패키지가 필요합니다. 바이너리 설치 프로그램을 사용하는 경우에도, 특정 런타임 라이브러리가 필요할 수 있습니다.

**Table 3.1: 설치 전제 조건 및 의존성**

| 항목                        | 요구 사항                      | 설명 및 설치 방법 (Ubuntu 기준)                              |
| --------------------------- | ------------------------------ | ------------------------------------------------------------ |
| **컴파일러**                | C++11 이상을 지원하는 컴파일러 | `sudo apt install build-essential`                           |
| **CMake**                   | 버전 3.16 이상                 | 빌드 시스템. `sudo apt install cmake`                        |
| **Qt**                      | 버전 5.15 이상                 | GUI 프레임워크. `sudo apt install qtbase5-dev qt5-qmake` 14  |
| **Gradle**                  | 버전 6.0 이상                  | Fast DDS-Gen 빌드에 필요. `sdkman`을 통한 설치 권장 2        |
| **Fast DDS-Gen**            | 버전 1.0.4 이상                | IDL 파일로부터 C++ 코드를 생성하는 도구. Gradle 설치 후 빌드 2 |
| **Visual Studio (Windows)** | Visual Studio 2019 이상        | Windows 환경에서의 빌드에 필요. C++ 재배포 가능 패키지 포함 2 |

이 표는 여러 문서에 흩어져 있는 정보를 통합하여 사용자가 설치 전에 시스템을 점검하고 준비할 수 있는 체크리스트를 제공합니다.2


가장 간단하고 권장되는 설치 방법은 eProsima 공식 웹사이트에서 제공하는 바이너리 설치 프로그램을 사용하는 것입니다.5

- **Windows:**

  1. eProsima 웹사이트의 다운로드 섹션에서 최신 버전의 "eProsima Fast DDS Monitor" Windows 설치 프로그램 (`.exe` 파일)을 다운로드합니다.
  2. 다운로드한 `.exe` 파일을 실행합니다.
  3. 설치 마법사의 지시에 따라 설치를 완료합니다.
  4. 설치 완료 후 나타나는 창에서 즉시 응용 프로그램을 실행할 수 있습니다.

- Linux:

  Linux 사용자는 두 가지 방식 중 하나를 선택할 수 있습니다.

  1. **.run 설치 프로그램:**
     - eProsima 웹사이트에서 `.run` 형식의 설치 파일(예: `eProsima_Fast-DDS-Monitor-x.x.x-Linux.run`)을 다운로드합니다.
     - 터미널을 열고 다운로드한 파일에 실행 권한을 부여합니다: `chmod +x eProsima_Fast-DDS-Monitor-x.x.x-Linux.run`.
     - 파일을 실행하여 (`./eProsima_Fast-DDS-Monitor-x.x.x-Linux.run`) 설치를 시작하고, 지시에 따라 설치를 완료합니다.
  2. **AppImage 휴대용 형식:**
     - eProsima 웹사이트에서 `.AppImage` 형식의 파일을 다운로드합니다.
     - 이 파일은 설치 과정 없이 직접 실행할 수 있는 휴대용 응용 프로그램입니다. 필요 시 `chmod +x` 명령으로 실행 권한을 부여한 후, 더블 클릭하거나 터미널에서 직접 실행하여 Monitor를 시작할 수 있습니다.


사용자 정의 빌드가 필요하거나 최신 개발 버전을 사용하고자 하는 개발자는 소스 코드로부터 직접 빌드할 수 있습니다. 다음은 Linux 환경에서의 상세한 빌드 과정입니다.5

1. **의존성 설치:** 먼저 위의 Table 3.1에 명시된 모든 의존성(cmake, g++, Qt5, gradle 등)을 설치합니다.

2. **Fast DDS 및 Fast DDS-Gen 설치:** Fast DDS Monitor는 Fast DDS 라이브러리에 의존하므로, 먼저 Fast DDS를 소스로부터 빌드하여 설치해야 합니다. 이 과정에서 **가장 중요한 단계는 CMake 실행 시 `-DFASTDDS_STATISTICS=ON` 옵션을 추가하여 통계 모듈을 활성화하는 것입니다.**

   ```Bash
   # Fast DDS 소스 코드 클론
   git clone https://github.com/eProsima/Fast-DDS.git
   mkdir Fast-DDS/build && cd Fast-DDS/build
   # 통계 모듈을 활성화하여 빌드 및 설치
   cmake.. -DFASTDDS_STATISTICS=ON
   sudo cmake --build. --target install
   ```
   
3. **Fast DDS Monitor 소스 코드 클론:**

   ```Bash
   git clone https://github.com/eProsima/Fast-DDS-monitor.git
   ```
   
4. **빌드 및 설치:**

   ```Bash
   cd Fast-DDS-monitor
   mkdir build && cd build
   cmake..
   cmake --build. --target install
   ```
   
5. **실행:** 설치가 완료되면 빌드 디렉터리 내의 `fastddsmonitor` 실행 파일을 통해 응용 프로그램을 시작할 수 있습니다.


공식 문서와 GitHub 저장소를 기준으로, eProsima는 **macOS용 공식 바이너리 설치 프로그램이나 소스 빌드 가이드를 제공하지 않습니다**.5 커뮤니티에서 macOS에서 사용하려는 시도나 관심이 있다는 증거는 있으나 12, 이는 공식적인 지원 범위 밖입니다.

따라서 macOS 사용자를 위한 가장 현실적이고 권장되는 해결책은 **Docker를 사용하는 것**입니다. eProsima는 Fast DDS Monitor를 포함한 여러 도구가 사전 설치된 Docker 이미지를 제공합니다.16 이 Docker 이미지를 사용하면 운영체제에 구애받지 않고 일관된 환경에서 Fast DDS Monitor를 실행할 수 있습니다.


ROS 2 사용자가 Fast DDS Monitor로 자신의 시스템을 모니터링하기 위해 반드시 거쳐야 하는 가장 중요한 설정 단계입니다. 이 단계를 생략하면 Monitor는 ROS 2 노드들을 발견할 수는 있어도, 어떠한 통계 데이터(지연 시간, 처리율 등)도 표시하지 못합니다.

활성화 방법은 `FASTDDS_STATISTICS` 환경 변수를 설정하는 것입니다. 이 변수는 세미콜론(;)으로 구분된 통계 토픽 이름들의 목록을 값으로 가집니다. ROS 2 노드를 실행하기 **전**에, 해당 노드를 실행할 터미널에서 이 환경 변수를 반드시 `export` 해야 합니다.11

다음은 모든 통계 토픽을 활성화하는 전체 예시입니다:

```Bash
export FASTDDS_STATISTICS="HISTORY_LATENCY_TOPIC;NETWORK_LATENCY_TOPIC;PUBLICATION_THROUGHPUT_TOPIC;SUBSCRIPTION_THROHPUT_TOPIC;RTPS_SENT_TOPIC;RTPS_LOST_TOPIC;HEARTBEAT_COUNT_TOPIC;ACKNACK_COUNT_TOPIC;NACKFRAG_COUNT_TOPIC;GAP_COUNT_TOPIC;DATA_COUNT_TOPIC;RESENT_DATAS_TOPIC;SAMPLE_DATAS_TOPIC;PDP_PACKETS_TOPIC;EDP_PACKETS_TOPIC;PHYSICAL_DATA_TOPIC"
```

이후, 동일한 터미널에서 `ros2 run` 또는 `ros2 launch` 명령을 실행하면 해당 노드들은 통계 데이터를 발행하기 시작하고, Fast DDS Monitor는 이 데이터를 수신하여 시각화할 수 있게 됩니다.


Fast DDS Monitor의 그래픽 사용자 인터페이스(GUI)는 복잡한 분산 시스템의 상태를 직관적으로 파악할 수 있도록 체계적으로 설계되었습니다. 각 패널의 기능과 그들 간의 상호작용을 이해하는 것은 효율적인 분석과 디버깅의 핵심입니다. 이 장에서는 GUI의 각 구성 요소를 상세히 설명하며, 이를 통해 사용자가 시스템을 진단하는 논리적인 작업 흐름을 제시합니다.

이 UI 구조는 사용자가 문제의 원인을 체계적으로 좁혀나가는, 이른바 '탑다운(top-down)' 방식의 진단 워크플로우를 자연스럽게 유도합니다. 예를 들어, 사용자는 먼저 **Physical Panel**에서 호스트 간의 물리적 연결을 확인하고, **Logical Panel**에서 도메인 참여와 토픽 인지를 점검합니다. 그 후 **DDS Panel**과 **Domain View**를 통해 개체 간의 매칭 여부를 확인하고, 만약 매칭에 실패했다면 **Info Panel**에서 QoS 설정을 비교하여 불일치를 찾아냅니다. 마지막으로, 통신은 되지만 성능이 저하될 경우 **Chart View**를 통해 지연 시간과 처리율을 정량적으로 분석할 수 있습니다.


응용 프로그램을 처음 실행하면 마주하는 전체적인 레이아웃은 다음과 같은 주요 부분으로 구성됩니다 5:

- **Application Menu:** 화면 상단의 메뉴 바로, `File`(파일), `Edit`(편집), `View`(보기), `Help`(도움말) 그룹으로 구성되어 응용 프로그램의 모든 기능에 접근할 수 있습니다.
- **Shortcuts Bar:** Application Menu 아래에 위치한 수평 바(bar)로, `Start Monitoring`, `Create Chart` 등 자주 사용하는 기능들에 빠르게 접근할 수 있는 아이콘들을 제공합니다.


화면 왼쪽에 위치한 Explorer Panel은 발견된 모든 개체들을 계층적인 트리 구조로 보여주는 기본 탐색 창입니다. 이 패널은 여러 하위 패널로 구성되어 있으며, 각 패널은 시스템을 다른 관점에서 조명합니다.5

- **DDS Panel:** DDS 통신의 핵심 구성 요소인 `DomainParticipant`, 그리고 그 하위에 중첩된 `DataReader`와 `DataWriter`를 보여줍니다. 각 항목을 클릭하여 확장하거나 축소하면서 개체 간의 소속 관계를 파악할 수 있습니다.
- **Physical Panel:** DDS 개체들이 실행되고 있는 물리적 배포 환경을 보여줍니다. `Host`(컴퓨터) -> `User`(사용자 계정) -> `Process`(프로세스) 순서의 계층 구조를 통해, 특정 `DomainParticipant`가 어느 컴퓨터의 어떤 프로세스에서 실행 중인지 명확하게 확인할 수 있습니다.
- **Logical Panel:** DDS 통신의 추상적인 구조를 보여줍니다. `Domain`(독립적인 통신 공간) -> `Topic`(데이터 채널)의 계층 구조를 통해, 어떤 토픽들이 어떤 도메인 내에서 사용되고 있는지 파악할 수 있습니다.


Explorer Panel에서 특정 개체(예: `DataReader`)를 더블 클릭하여 선택하면, 화면 왼쪽 하단에 위치한 이 패널들이 해당 개체에 대한 상세 정보로 채워집니다.5

- **Info Panel:** 선택된 개체의 정적인 설정 정보를 표시합니다. 여기에는 개체의 고유 식별자(GUID), 프로세스 ID(PID), 그리고 가장 중요하게는 **QoS 설정**이 포함됩니다. 두 개체의 QoS가 호환되지 않아 통신이 실패하는 경우, 이 패널에서 각 개체의 QoS 설정을 비교하여 문제의 원인을 직접 확인할 수 있습니다.
- **Statistics Panel:** 선택된 개체와 관련된 주요 성능 지표(KPI)를 실시간으로 요약하여 보여줍니다. 예를 들어, 특정 `DataReader`를 선택하면 해당 리더의 평균 구독 처리율, `Host`를 선택하면 해당 호스트 간의 평균 지연 시간 등을 수치로 확인할 수 있습니다.


화면 오른쪽 하단에 위치한 이 패널들은 Monitor 응용 프로그램 자체의 상태와 네트워크에서 발생하는 이벤트를 실시간으로 보여줍니다.5

- **Status Sub-Panel:** 현재 모니터링 중인 `Domain`의 수와 발견된 총 `Entity`의 개수와 같은 고수준의 요약 정보를 제공합니다.
- **Log Sub-Panel:** Monitor가 네트워크에서 새로운 개체를 발견하거나 상태 변경을 감지할 때마다 발생하는 콜백(callback) 이벤트를 시간 순서대로 기록합니다. "새로운 `DataWriter` 발견", "참여자 연결 끊김"과 같은 저수준의 이벤트 로그를 통해 네트워크에서 무슨 일이 일어나고 있는지 원시적인 형태로 확인할 수 있습니다.10


화면 중앙의 가장 넓은 영역을 차지하는 Main Panel은 실제 분석이 이루어지는 곳으로, 여러 탭을 통해 다양한 뷰를 제공합니다.5

- **Domain View:** DDS 네트워크의 연결 관계를 강력한 그래프 형태로 시각화합니다. 이 뷰는 `DataWriter`와 `DataReader`가 특정 `Topic`을 통해 어떻게 연결되는지, 그리고 이 모든 통신 개체들이 어떻게 물리적인 `Process`와 `Host`에 포함되는지를 한눈에 보여줍니다. 특정 토픽을 기준으로 그래프를 필터링하여 복잡한 네트워크에서 원하는 부분만 집중적으로 분석할 수 있습니다.20
- **Chart View:** 성능 데이터를 시간의 흐름에 따라 플로팅하는 기본 인터페이스입니다. 사용자는 `Chartbox`라는 개별 차트 창을 생성하고, 여기에 지연 시간, 처리율 등 원하는 메트릭을 시리즈(series)로 추가하여 시각적으로 분석할 수 있습니다.


이 패널들은 통신 문제를 진단하는 데 매우 중요한 역할을 합니다.

- **Issues Panel:** Monitor 응용 프로그램 자체에서 발생하는 오류를 기록합니다. 예를 들어, 이미 모니터링 중인 도메인을 다시 모니터링하려고 시도하는 등의 사용자 오류가 여기에 표시됩니다.5
- **Problem Summary Panel:** DDS 프로토콜 수준에서 감지된 문제들을 개체별로 요약하여 목록으로 보여주는 핵심적인 문제 해결 기능입니다. 여기에는 **`incompatible QoS`(호환되지 않는 QoS)**, **`missed deadlines`(데드라인 놓침)** 등과 같은 구체적인 문제들이 보고됩니다. 예를 들어, `DataWriter`와 `DataReader`가 매칭되지 않을 때 이 패널을 확인하면 "incompatible QoS" 경고와 함께 어떤 QoS 정책(예: `RELIABILITY`)이 불일치하는지 직접적으로 알려주어 문제 해결 시간을 크게 단축시킬 수 있습니다.


이론적인 지식을 바탕으로, 이제 Fast DDS Monitor를 실제 시나리오에 적용하는 방법을 단계별로 살펴봅니다. 기본적인 DDS 네트워크 모니터링부터 시작하여, 로보틱스 개발자에게 가장 중요한 ROS 2 시스템 분석까지 구체적인 예제를 통해 학습합니다.


이 예제는 Fast DDS Monitor의 가장 기본적인 사용법을 보여주며, 공식 튜토리얼을 기반으로 합니다.9

1. **Monitor 시작 및 도메인 초기화:**
   - Fast DDS Monitor를 실행하고 초기 화면에서 `Start monitoring!` 버튼을 클릭합니다.
   - 도메인을 초기화하라는 대화 상자가 나타나면, 기본값인 도메인 ID `0`을 입력하고 `OK`를 클릭하여 모니터링을 시작합니다.
2. **구독자(Subscriber) 실행:**
   - 별도의 터미널을 열고, Fast DDS 예제에 포함된 "Hello World" 구독자 응용 프로그램을 실행합니다. 이 응용 프로그램은 통계 모듈이 활성화된 Fast DDS 라이브러리로 컴파일되어야 합니다.
   - **관찰:** Monitor의 Explorer Panel을 확인합니다. `DDS Panel`에 `RTPSParticipant`라는 이름의 새로운 `DomainParticipant`와 그 하위에 `DataReader`가 나타나는 것을 볼 수 있습니다. `Physical Panel`에는 이 구독자 프로세스가 실행 중인 `Host`, `User`, `Process` 정보가 표시됩니다. `Logical Panel`에는 `hello_world_topic`이라는 새로운 `Topic`이 도메인 0 아래에 생성된 것을 확인할 수 있습니다.
3. **발행자(Publisher) 실행:**
   - 또 다른 터미널에서 "Hello World" 발행자 응용 프로그램을 실행합니다.
   - **관찰:** Monitor에 새로운 `DomainParticipant`와 그 하위의 `DataWriter`가 나타납니다. 이제 발행자와 구독자가 모두 실행 중이므로, Main Panel의 `Domain View` 탭을 클릭합니다.
4. **연결 관계 분석:**
   - `Domain View`에는 발행자(`DataWriter`)와 구독자(`DataReader`)가 `hello_world_topic`을 통해 연결된 모습이 그래프로 명확하게 시각화됩니다. 각 개체가 어떤 프로세스와 호스트에 속해 있는지도 한눈에 파악할 수 있습니다.

이 간단한 예제를 통해 사용자는 DDS 개체들이 네트워크상에서 어떻게 발견되고 연결되는지, 그리고 Monitor가 이 과정을 어떻게 시각적으로 표현하는지에 대한 기본적인 감각을 익힐 수 있습니다.


이 예제는 ROS 2 개발자에게 초점을 맞춘 실용적인 시나리오입니다. ROS 2 노드와 토픽이 Fast DDS Monitor에서 어떻게 표현되는지 이해하는 것이 핵심입니다.1

1. **환경 설정:**

   - 모니터링을 수행할 터미널과 ROS 2 노드를 실행할 각 터미널에서 ROS 2 환경을 소싱(source)합니다. (예: `source /opt/ros/humble/setup.bash`)

   - **가장 중요한 단계:** ROS 2 노드를 실행할 각 터미널에서 `FASTDDS_STATISTICS` 환경 변수를 `export`하여 통계 데이터 발행을 활성화합니다.

     ```Bash
  export FASTDDS_STATISTICS="HISTORY_LATENCY_TOPIC;PUBLICATION_THROUGHPUT_TOPIC;PHYSICAL_DATA_TOPIC"
     ```
     
   - Fast DDS Monitor를 실행하고 도메인 0을 모니터링합니다.

2. **ROS 2 노드 실행:**

   - 한 터미널에서 `ros2 run demo_nodes_cpp talker`를 실행합니다.
   - 다른 터미널에서 `ros2 run demo_nodes_cpp listener`를 실행합니다.

3. **ROS 2 개체 분석:**

   - **DDS Panel 분석:** Monitor의 `DDS Panel`을 보면, `talker`와 `listener`라는 이름의 ROS 2 노드가 각각 별개의 DDS `DomainParticipant`로 표시되는 것을 확인할 수 있습니다. 이는 ROS 2의 각 노드가 내부적으로 하나의 DDS Participant에 해당함을 보여줍니다.
   - **Logical Panel 및 이름 맹글링(Name Mangling) 확인:** `Logical Panel`에서 토픽 목록을 확인합니다. ROS 2에서 `/chatter`라는 이름으로 사용되는 토픽이 Monitor에서는 `rt/chatter`라는 이름으로 표시되는 것을 볼 수 있습니다. 이는 ROS 2가 순수 DDS 응용 프로그램과의 의도치 않은 충돌을 피하기 위해 토픽 이름 앞에 `rt/`와 같은 접두사를 붙이는 '이름 맹글링'을 수행하기 때문입니다.23 이 맹글링 규칙을 모르는 사용자는 자신의 토픽이 제대로 발견되지 않는다고 오해할 수 있으나, Monitor는 이 변환된 실제 DDS 토픽 이름을 명확히 보여줌으로써 혼란을 방지합니다.
   - **Info Panel 및 타입 이름 확인:** `rt/chatter` 토픽을 선택하고 `Info Panel`을 보면, 데이터 타입 이름 역시 `std_msgs/msg/String`이 아닌 `std_msgs::msg::dds_::String_`과 같이 변환되어 있는 것을 확인할 수 있습니다.23 이 또한 이름 맹글링의 일부이며, Monitor는 ROS 2와 순수 DDS 시스템을 연동하려는 개발자에게 필수적인 정보인 실제 타입 이름을 알려줍니다.


성능 분석을 위해서는 데이터를 시각화하는 것이 필수적입니다. Main Panel의 `Chart View`는 이를 위한 강력한 도구를 제공합니다.9

- **과거 데이터 플로팅 (Historical Data):**
  1. 상단 메뉴에서 `Edit -> Display Historical Data`를 선택합니다.
  2. 플로팅할 데이터 종류를 선택합니다. 예를 들어, 전송된 총 메시지 수를 보려면 `DATA_COUNT`를, 지연 시간을 분석하려면 `FASTDDS_LATENCY`를 선택합니다.
  3. 새로운 대화 상자에서 차트 시리즈를 상세히 설정합니다.
     - **Source Entity:** 데이터를 어떤 개체로부터 가져올지 선택합니다. (예: `DATA_COUNT`는 `DataWriter`에서)
     - **Number of bins:** 데이터를 몇 개의 구간으로 나누어 집계할지 결정합니다. (0으로 설정 시 모든 데이터 포인트 표시)
     - **Statistics kind:** `SUM`(합계), `MEAN`(평균), `MEDIAN`(중앙값), `MAX`(최댓값), `MIN`(최솟값) 등 어떤 통계치를 플로팅할지 선택합니다.
  4. `Add` 또는 `Add & Close`를 클릭하여 차트를 생성합니다. 예를 들어, `FASTDDS_LATENCY`에 대해 `MEDIAN`, `MAX`, `MIN` 시리즈를 각각 추가하면 지연 시간의 분포를 한눈에 파악할 수 있습니다.
- **실시간 데이터 플로팅 (Real-Time Data):**
  1. Shortcuts Bar에서 `Display Real-Time Data` 버튼을 클릭합니다.
  2. `FASTDDS_LATENCY`와 같은 데이터 종류를 선택합니다.
  3. **Time window**(데이터를 집계할 시간 창, 예: 1분)와 **Update period**(차트를 갱신할 주기, 예: 5초)를 설정합니다.
  4. 차트 시리즈 설정 대화 상자에서 `MEAN`, `MAX`, `MIN` 등의 통계 종류를 선택하여 실시간으로 변하는 성능 추이를 모니터링할 수 있습니다. `RAW_DATA`를 선택하면 수신되는 모든 데이터 포인트를 실시간으로 점으로 찍어 보여줍니다.

이러한 시각화 기능들을 통해 개발자는 시스템의 성능을 정량적으로 평가하고, 문제 발생 시 데이터에 기반한 근거를 가지고 원인을 추적할 수 있습니다.


Fast DDS Monitor는 강력한 도구이지만, 그 기능을 제대로 활용하기 위해서는 몇 가지 흔한 문제들과 그 해결책을 숙지하고 있어야 합니다. 이 장에서는 커뮤니티에 보고된 실제 사례들을 바탕으로, 사용자들이 자주 겪는 문제들에 대한 체계적인 진단 및 해결 가이드를 제공합니다.


가장 빈번하게 발생하는 문제로, Monitor를 실행했지만 Explorer Panel에 아무런 개체도 나타나지 않는 경우입니다. 원인은 다양하며, 다음 순서대로 점검해 볼 수 있습니다.

- **원인 1: 도메인 ID 불일치:**

  - **진단:** ROS 2는 `ROS_DOMAIN_ID` 환경 변수를 통해 DDS 도메인을 설정합니다. 이 값이 설정되어 있지 않으면 기본값은 0입니다. Monitor가 다른 도메인 ID를 수신 대기하고 있다면 노드를 발견할 수 없습니다.

  - **해결책:** ROS 2 노드를 실행하는 터미널과 Monitor가 동일한 `ROS_DOMAIN_ID`를 사용하고 있는지 확인합니다.13

    `export ROS_DOMAIN_ID=42`와 같이 명시적으로 ID를 설정하고, Monitor에서도 해당 도메인(42)을 모니터링하도록 초기화합니다.

- **원인 2: RMW(ROS MiddleWare) 구현체 문제:**

  - **진단:** ROS 2는 여러 DDS 벤더(eProsima Fast DDS, RTI Connext, Eclipse Cyclone DDS 등)를 지원합니다. Fast DDS Monitor는 오직 Fast DDS를 RMW로 사용할 때만 작동합니다.
  - **해결책:** `RMW_IMPLEMENTATION` 환경 변수를 확인하여 `rmw_fastrtps_cpp`로 설정되어 있는지 확인합니다. 만약 `rmw_cyclonedds_cpp` 등으로 설정되어 있다면, `export RMW_IMPLEMENTATION=rmw_fastrtps_cpp` 명령으로 변경한 후 노드를 다시 실행합니다.25

- **원인 3: Fast DDS 버전 비호환성:**

  - **진단:** ROS 2 배포판(Foxy, Humble 등)은 특정 버전의 `rmw_fastrtps` 패키지와 Fast DDS 라이브러리에 의존합니다. 사용자가 임의로 Fast DDS 버전을 업데이트하거나, ROS 2 배포판과 호환되지 않는 버전을 사용하면 예기치 않은 문제가 발생할 수 있습니다. 특히 통계 모듈과 관련된 API는 버전에 따라 변경될 수 있습니다.
  - **해결책:** GitHub 이슈 등에서 여러 사용자들이 최신 Fast DDS 버전 대신 특정 안정 버전(예: v2.6.2)으로 다운그레이드하여 문제를 해결한 사례가 보고되었습니다.13 문제가 지속될 경우, 사용 중인 ROS 2 배포판과 공식적으로 호환되는 Fast DDS 버전을 확인하고 재설치하는 것을 고려해야 합니다.

- **원인 4: 네트워크 구성 문제:**

  - **진단:** 방화벽, Docker의 브리지 네트워크, 복잡한 서브넷 환경 등은 DDS의 자동 발견(discovery)에 사용되는 멀티캐스트 패킷 전파를 방해할 수 있습니다.
  - **해결책:** 로컬 방화벽 설정을 확인하고, Docker를 사용하는 경우 `--net=host` 옵션을 사용하여 호스트 네트워크를 직접 사용하도록 설정해 볼 수 있습니다. 보다 근본적인 해결책으로, 멀티캐스트가 제한된 환경에서는 Fast DDS의 Discovery Server 기능을 사용하여 중앙 서버를 통해 노드들이 서로를 발견하도록 구성할 수 있습니다.6


노드와 토픽은 정상적으로 보이지만, Statistics Panel이나 Chart View에 아무런 데이터도 나타나지 않는 경우입니다. 이 문제의 원인은 거의 두 가지 중 하나입니다.

- **가장 주된 원인: `FASTDDS_STATISTICS` 환경 변수 미설정:**
  - **진단:** 이 문제는 사용자가 ROS 2 노드를 실행하기 전에 통계 발행을 활성화하는 환경 변수를 설정하는 것을 잊었을 때 발생합니다.
  - **해결책:** ROS 2 노드를 종료하고, 노드를 실행할 터미널에서 `export FASTDDS_STATISTICS="..."` 명령을 실행하여 환경 변수를 올바르게 설정한 후, 노드를 다시 시작합니다.11
- **두 번째 원인: 통계 모듈이 비활성화된 Fast DDS 라이브러리 사용:**
  - **진단:** ROS 2의 기본 바이너리 설치나 Fast DDS의 일부 배포판은 성능 최적화를 위해 통계 모듈이 비활성화된 상태(`-DFASTDDS_STATISTICS=OFF`)로 컴파일되어 있을 수 있습니다.
  - **해결책:** 이 경우, 유일한 해결책은 Fast DDS 라이브러리를 소스 코드로부터 직접 재컴파일하는 것입니다. 이 때 반드시 CMake 옵션에 `-DFASTDDS_STATISTICS=ON`을 추가해야 합니다.8


- **QoS 불일치 진단:**

  1. `Domain View`에서 발행자와 구독자 간의 연결이 보이지 않는 경우, QoS 불일치를 의심할 수 있습니다.
  2. `Explorer Panel`에서 해당 토픽의 `DataWriter`를 선택하고 `Info Panel`에서 QoS 설정을 확인합니다.
  3. 같은 방식으로 `DataReader`를 선택하고 `Info Panel`에서 QoS 설정을 확인합니다.
  4. 두 개체의 QoS 설정(예: `Reliability`, `Durability`, `Deadline`)을 직접 비교하여 불일치하는 부분을 찾습니다.
  5. 더 간단하게는, `Problem Summary Panel`에 `incompatible QoS`라는 경고가 자동으로 표시되는지 확인합니다. 이 패널은 어떤 QoS 정책이 문제인지 명시적으로 알려줍니다.5

- 데이터 손실 진단:

  Chart View를 활용하여 데이터 손실을 정량적으로 진단할 수 있습니다. 이를 위해서는 FASTDDS_STATISTICS 환경 변수에 관련 통계 토픽들이 포함되어 있어야 합니다.

  1. **송수신 데이터량 비교:** `DataWriter`의 `PUBLICATION_THROUGHPUT_TOPIC` 또는 `DATA_COUNT`와 `DataReader`의 `SUBSCRIPTION_THROUGHPUT_TOPIC` 또는 `SAMPLE_DATAS_TOPIC`을 동일한 차트에 플로팅합니다. 두 그래프 사이에 지속적으로 차이가 발생하고 그 차이가 누적된다면 데이터 손실을 의미합니다.

  2. **직접적인 손실 측정:** `RTPS_LOST_TOPIC`을 플로팅합니다. 이 값은 RTPS 프로토콜 수준에서 감지된 손실 패킷의 수를 직접 보여주는 가장 명확한 지표입니다.

  3. **신뢰성 통신 분석:** 신뢰성(Reliable) QoS를 사용하는 경우, `HEARTBEAT_COUNT_TOPIC`과 `ACKNACK_COUNT_TOPIC`을 모니터링합니다.7

     `ACKNACK` 메시지의 수가 비정상적으로 많다면, 이는 네트워크 상태가 좋지 않아 수신자가 데이터 재전송을 빈번하게 요청하고 있음을 의미하며, 잠재적인 데이터 손실 및 높은 지연 시간의 원인이 됩니다.

아래 표는 `FASTDDS_STATISTICS` 환경 변수에 사용할 수 있는 주요 통계 토픽과 그 의미를 요약한 것입니다. 이 표를 통해 사용자는 불필요한 오버헤드를 줄이고, 진단 목적에 맞는 특정 통계만 선택적으로 활성화할 수 있습니다.

**Table 6.1: 주요 `FASTDDS_STATISTICS` 토픽과 그 의미**

| 통계 토픽 이름                 | 설명 및 진단적 가치                                          |
| ------------------------------ | ------------------------------------------------------------ |
| `HISTORY_LATENCY_TOPIC`        | 개별 데이터 샘플의 종단 간 지연 시간. 시스템의 실시간성 평가에 핵심적입니다. |
| `PUBLICATION_THROUGHPUT_TOPIC` | 발행자의 초당 데이터 전송량. 대역폭 사용량 및 발행 성능 병목을 식별합니다. |
| `RTPS_LOST_TOPIC`              | 손실된 RTPS 패킷 수. 데이터 손실 여부를 직접적으로 확인하는 가장 중요한 지표입니다. |
| `HEARTBEAT_COUNT_TOPIC`        | 신뢰성 통신에서 발행자가 보내는 Heartbeat 메시지 수. 통신 활성 상태를 나타냅니다. |
| `ACKNACK_COUNT_TOPIC`          | 신뢰성 통신에서 구독자가 보내는 ACKNACK/NACK 메시지 수. 높은 값은 잦은 재전송 요청을 의미하며, 네트워크 품질 저하의 신호입니다. |
| `DATA_COUNT_TOPIC`             | 발행된 총 데이터 메시지 수. 예상되는 메시지 수와 비교하여 누락 여부를 간접적으로 추정할 수 있습니다. |
| `PHYSICAL_DATA_TOPIC`          | 개체가 실행되는 호스트, 사용자, 프로세스 정보. 분산 환경에서 문제의 물리적 위치를 파악하는 데 필수적입니다. |


eProsima Fast DDS Monitor는 강력한 도구이지만, 유일한 선택지는 아닙니다. 로보틱스 개발자는 프로젝트의 요구사항, 사용 중인 DDS 구현체, 예산 등에 따라 다양한 모니터링 도구를 고려할 수 있습니다. 이 장에서는 Fast DDS Monitor를 RTI Administration Console 및 ROS 2의 RQt 도구 모음과 비교하여 각 도구의 장단점을 명확히 분석합니다.


- **강점:**
  - **깊은 통합:** eProsima Fast DDS와 완벽하게 통합되어, Heartbeat, ACKNACK 수와 같은 프로토콜 수준의 상세하고 독점적인 통계 정보에 접근할 수 있습니다.1
  - **계층적 뷰:** 물리적 계층(Host, Process)부터 논리적 계층(Domain, Topic), DDS 계층(Participant, Endpoint)까지 다층적인 뷰를 제공하여 시스템을 종합적으로 이해하는 데 매우 유용합니다.5
  - **오픈 소스 및 무료:** 라이선스 비용 없이 자유롭게 사용할 수 있어 학계 및 개인 개발자에게 접근성이 높습니다.
- **약점:**
  - **벤더 종속성(Vendor Lock-in):** 이 도구의 가장 큰 한계는 이름에서 알 수 있듯이 오직 eProsima Fast DDS를 사용하는 시스템만 모니터링할 수 있다는 점입니다. Eclipse Cyclone DDS나 RTI Connext DDS와 같은 다른 DDS 구현체와는 호환되지 않습니다.


- **강점:**
  - **성숙하고 풍부한 기능:** RTI Connext DDS를 위한 상용 제품으로, 매우 성숙하고 강력한 기능들을 제공합니다. 수십 년간 축적된 노하우가 집약된 도구입니다.
  - **자동 QoS 매치 분석:** 시스템 내의 모든 발행자와 구독자 간의 QoS 호환성을 실시간으로 자동 분석하여 불일치 항목을 즉시 보고하는 강력한 기능을 갖추고 있습니다.27
  - **원격 서비스 관리:** Routing Service와 같은 RTI 인프라 서비스를 원격으로 시작, 중지, 재구성할 수 있습니다.27
  - **그래픽 QoS 편집기:** XML 파일을 직접 수정하는 대신, 그래픽 인터페이스를 통해 QoS 프로파일을 쉽고 안전하게 편집할 수 있는 기능을 제공합니다.28
- **약점:**
  - **상용 제품:** 사용을 위해서는 상용 라이선스가 필요하며, 이는 상당한 비용을 수반할 수 있습니다.
  - **벤더 종속성:** RTI Connext DDS 생태계에 종속되어 있습니다.


- **강점:**

  - **ROS 2 네이티브 통합:** ROS 2 생태계에 기본적으로 포함된 도구 모음으로, 설치 및 사용이 간편합니다.

  - **벤더 독립성:** RMW(ROS MiddleWare) 추상화 계층 위에서 작동하므로, Fast DDS, Cyclone DDS, Connext DDS 등 어떤 DDS 구현체를 사용하든 동일하게 작동합니다.

  - **경량 및 목적 중심:** `rqt_graph`는 노드와 토픽 간의 연결 관계를 시각화하는 데 탁월하고 29, 

    `rqt_plot`은 특정 토픽의 숫자 데이터를 실시간으로 플로팅하는 데 유용하며 30, 

    `rqt_console`은 로그 메시지를 필터링하고 확인하는 데 강력한 기능을 제공합니다.31

- **약점:**

  - **ROS 추상화 수준에서 작동:** RQt 도구들은 DDS 프로토콜의 세부 사항을 볼 수 없습니다. DDS `DomainParticipant`나 `DataReader`/`DataWriter`와 같은 DDS 고유의 개체를 직접 인식하지 못하며, DDS QoS 설정을 검사하거나 프로토콜 수준의 통계(예: Heartbeat 수)에 접근할 수 없습니다.32
  - **범용 도구의 한계:** RQt는 DDS 네트워크를 심층적으로 분석하기 위해 특화된 솔루션이 아니라, ROS 시스템을 위한 범용적인 GUI 도구 모음입니다. 따라서 Fast DDS Monitor나 RTI Admin Console이 제공하는 깊이 있는 분석 기능은 부족합니다.

아래 표는 세 가지 도구의 주요 특징을 한눈에 비교하여, 사용자가 자신의 상황에 가장 적합한 도구를 선택할 수 있도록 돕습니다.

**Table 7.1: 모니터링 도구 기능 비교**

| 기능                                  | eProsima Fast DDS Monitor         | RTI Administration Console | ROS 2 RQt Suite        |
| ------------------------------------- | --------------------------------- | -------------------------- | ---------------------- |
| **비용**                              | 무료 (오픈 소스)                  | 상용 (라이선스 필요)       | 무료 (오픈 소스)       |
| **지원 DDS 벤더**                     | eProsima Fast DDS 전용            | RTI Connext DDS 전용       | 모든 RMW (벤더 독립적) |
| **작동 수준**                         | DDS 프로토콜 계층                 | DDS 프로토콜 계층          | ROS 추상화 계층        |
| **실시간 데이터 플로팅**              | 예                                | 예                         | 예 (`rqt_plot`)        |
| **DDS QoS 설정 검사**                 | 예 (Info Panel)                   | 예                         | 아니요                 |
| **자동 QoS 불일치 분석**              | 아니요 (Problem Summary에서 보고) | 예 (Match Analysis 기능)   | 아니요                 |
| **프로토콜 수준 통계 (Heartbeat 등)** | 예                                | 예                         | 아니요                 |
| **원격 서비스 관리**                  | 아니요                            | 예                         | 아니요                 |
| **그래픽 QoS 편집기**                 | 아니요                            | 예                         | 아니요                 |
| **ROS 2 통합 수준**                   | 높음 (ROS 2 특화 기능 지원)       | 중간 (ROS 2 연동 가능)     | 최상 (ROS 2 네이티브)  |

결론적으로, 어떤 도구를 선택할지는 사용자의 구체적인 목표에 따라 달라집니다. eProsima Fast DDS를 주력으로 사용하며 깊이 있는 디버깅과 성능 분석이 필요하다면 **Fast DDS Monitor**가 최적의 선택입니다. 미션 크리티컬한 상용 시스템에서 RTI Connext DDS를 사용하고 있다면, 강력한 기능과 기술 지원을 제공하는 **RTI Admin Console**이 필수적입니다. 반면, DDS 구현체에 상관없이 ROS 노드와 토픽의 연결 관계를 빠르게 확인하거나 특정 토픽의 데이터를 간단히 시각화하는 것이 목적이라면, 가볍고 범용적인 **RQt 도구 모음**이 가장 효율적일 수 있습니다.


이 보고서는 eProsima Fast DDS Monitor의 개념적 배경부터 아키텍처, 설치, 사용법, 고급 진단 기법, 그리고 경쟁 도구와의 비교 분석에 이르기까지 다각적인 고찰을 제공했습니다. 이 마지막 장에서는 전체 분석을 종합하여 도구의 핵심 역량과 한계를 요약하고, 개발 워크플로우에 통합하기 위한 모범 사례를 제시하며, 향후 발전 방향을 조망합니다.


eProsima Fast DDS Monitor는 eProsima Fast DDS 기반의 분산 시스템을 위한 필수적인 분석 도구입니다.

- **핵심 역량:**
  - **심층 내성(Deep Introspection):** Fast DDS와의 긴밀한 통합을 통해 다른 도구들이 접근할 수 없는 프로토콜 수준의 상세한 통계 데이터(지연 시간, 처리율, 패킷 손실, Heartbeat/ACKNACK 카운트 등)를 제공합니다.
  - **다층적 시각화:** 물리적, 논리적, DDS적 관점을 아우르는 계층적 뷰를 통해 복잡한 시스템의 구조를 직관적으로 파악하게 해줍니다.
  - **데이터 기반 문제 해결:** 차트와 통계 패널을 통해 성능 병목과 통신 이상 현상을 정량적으로 분석하고, 추측이 아닌 데이터에 기반한 체계적인 디버깅을 가능하게 합니다.
- **명백한 한계:**
  - **벤더 종속성:** 오직 eProsima Fast DDS 환경에서만 작동하며, 다른 DDS 구현체와는 호환되지 않습니다.
  - **설정 의존성:** 그 기능은 전적으로 Fast DDS 라이브러리의 통계 모듈 활성화(`-DFASTDDS_STATISTICS=ON`)와 ROS 2 노드 실행 시 `FASTDDS_STATISTICS` 환경 변수 설정에 의존합니다. 이 설정이 누락되면 도구는 제 기능을 발휘하지 못합니다.


Fast DDS Monitor는 단순히 문제가 발생했을 때만 사용하는 사후 대응적 도구가 아닙니다. 개발 주기 전반에 걸쳐 능동적으로 활용할 때 그 진정한 가치가 발휘됩니다.

1. **통합 테스트 단계에서의 필수 사용:** 새로운 노드나 기능을 시스템에 통합하는 과정에서 항상 Monitor를 실행하십시오. 이를 통해 노드 간의 발견(discovery) 문제, 토픽 매칭 실패, 초기 QoS 불일치 등을 즉시 시각적으로 확인하여 문제를 조기에 해결할 수 있습니다.
2. **성능 기준선(Baseline) 설정:** 응용 프로그램이 정상적으로 작동하는 상태에서 Monitor의 차트 기능을 사용하여 주요 성능 지표(평균 지연 시간, 최대 처리율 등)의 기준선을 측정하고 기록해 두십시오. 이는 향후 시스템 변경 후 성능 저하 여부를 객관적으로 판단하는 중요한 근거가 됩니다.
3. **체계적인 결함 격리(Fault Isolation):** 통신 관련 버그가 발생했을 때, Monitor의 다층적 뷰를 활용하여 체계적으로 문제의 범위를 좁혀나가십시오. 물리적 뷰에서 네트워크 연결을 확인하고, 논리적 뷰에서 도메인 및 토픽 구성을 점검한 후, DDS 뷰에서 개체 매칭과 QoS를 상세히 분석하는 탑다운 접근법을 사용하십시오.
4. **CI/CD 파이프라인에의 통합:** 고급 사용 사례로, Monitor의 핵심 엔진인 `statistics_backend` 라이브러리를 직접 사용하는 스크립트를 작성하여 지속적 통합/지속적 배포(CI/CD) 파이프라인에 통합할 수 있습니다.15 이를 통해 새로운 코드가 커밋될 때마다 자동으로 성능 회귀 테스트(performance regression test)를 수행하여, 잠재적인 성능 저하를 조기에 감지하고 방지할 수 있습니다.

이러한 접근법은 Fast DDS Monitor를 단순한 디버깅 도구에서 시스템의 품질을 사전에 보증하는 **시스템 분석 및 품질 보증 도구**로 격상시킵니다.


eProsima Fast DDS Monitor는 활발하게 개발이 진행 중인 프로젝트입니다. 최근 릴리스 노트와 로드맵 발표를 통해 향후 발전 방향을 엿볼 수 있습니다.33

- **기능 강화:** 최근 릴리스에서는 호환되지 않는 QoS 문제에 대한 모니터링 기능이 확장되었고, TCP 전송 계층 처리 방식이 개선되었으며, 동적 네트워크 인터페이스 변경에 대한 대응력이 향상되었습니다. 이는 도구가 더욱 견고해지고 더 넓은 범위의 시나리오를 지원하고 있음을 보여줍니다.
- **사용성 개선:** Discovery Server 로케이터를 이름으로 설정하는 기능, 런타임에 원격 서버 목록을 수정하는 기능 등이 추가되어 복잡한 네트워크 환경에서의 설정 편의성이 증대되었습니다.33
- **장기적 비전:** Content Filtered Topic과 같은 고급 DDS 기능에 대한 지원, Python 바인딩 제공 등은 Fast DDS 생태계 전반의 발전에 발맞추어 Monitor의 분석 능력 또한 지속적으로 확장될 것임을 시사합니다.33

결론적으로, eProsima Fast DDS Monitor는 현대 로보틱스 시스템 개발의 복잡성을 관리하기 위한 강력하고 필수적인 도구입니다. 이 보고서에서 제시된 심층적인 분석과 실용적인 가이드를 통해 개발자들은 이 도구를 최대한 활용하여, 더 안정적이고 성능이 뛰어난 분산 로봇 응용 프로그램을 구축할 수 있을 것입니다. Fast DDS 생태계가 계속 발전함에 따라, Monitor의 역할과 중요성 또한 더욱 커질 것으로 기대됩니다.


1. Fast DDS Monitor demo using ROS 2 - YouTube, accessed July 6, 2025, https://www.youtube.com/watch?v=OYibnUnMIlc
2. Fast DDS 설치 | PX4 오토파일럿 사용자 설명서 (v1.13), accessed July 6, 2025, https://docs.px4.io/v1.13/ko/dev_setup/fast-dds-installation.html
3. eProsima Fast DDS Monitor 2.0: New Release - YouTube, accessed July 6, 2025, https://www.youtube.com/watch?v=Y0jPmEtxtx8
4. eProsima Fast DDS Monitor Documentation, accessed July 6, 2025, https://fast-dds-monitor.readthedocs.io/
5. eProsima Fast DDS Monitor Documentation, accessed July 6, 2025, https://fast-dds-monitor.readthedocs.io/en/latest/
6. eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/
7. ROS 2 Network Monitoring, accessed July 6, 2025, [http://download.ros.org/downloads/roscon/2022/ROS%202%20network%20monitoring.pdf](http://download.ros.org/downloads/roscon/2022/ROS 2 network monitoring.pdf)
8. Fast-DDS Monitor did not pick the traffic data. / Issue #151 - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-monitor/issues/151
9. 3. Example of usage - 3.2.0, accessed July 6, 2025, https://fast-dds-monitor.readthedocs.io/en/latest/rst/getting_started/tutorial.html
10. 3. Example of usage - Fast DDS Monitor 1.5.0 documentation, accessed July 6, 2025, https://fast-dds-monitor.readthedocs.io/en/v1.5.0/rst/getting_started/tutorial.html
11. 1.1. Monitor Tutorial with ROS 2 Jazzy - 3.2.0, accessed July 6, 2025, https://fast-dds-monitor.readthedocs.io/en/latest/rst/ros/jazzy/jazzy.html
12. frontiers-robotics-ai-rep-pkg/data-analysis/repos.csv at main - GitHub, accessed July 6, 2025, https://github.com/IntelAgir-Research-Group/frontiers-robotics-ai-rep-pkg/blob/main/data-analysis/repos.csv
13. Fast-DDS-monitor ros2 FOXY / Issue #153 - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-monitor/issues/153
14. Shapes Demo Documentation - Read the Docs, accessed July 6, 2025, https://readthedocs.org/projects/eprosima-shapes-demo/downloads/pdf/latest/
15. 2.2. Monitoring ROS 2 with Fast DDS Statistics Backend ..., accessed July 6, 2025, https://docs.vulcanexus.org/en/iron/rst/tutorials/tools/monitor_backend/monitor_backend.html
16. eProsima/Fast-DDS: The most complete DDS - Proven: Plenty of success cases. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS
17. 1.3. Fast DDS Suite Image - 3.2.2, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/stable/docker/fastdds_suite/fast_dds_suite.html
18. 1. Docker Images - 3.2.2 - eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/v3.2.2/docker/docker.html
19. Fast DDS Monitor and ROS 2 Humble / Issue #202 - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-monitor/issues/202
20. eProsima Fast DDS Monitor - Graph View Demo - YouTube, accessed July 6, 2025, https://www.youtube.com/watch?v=YxIklcaJV48
21. 7. Issues Panel - eProsima Fast DDS Monitor Documentation, accessed July 6, 2025, https://fast-dds-monitor.readthedocs.io/en/v1.5.0/rst/user_manual/issues_panel.html
22. 1. Fast DDS Monitor with ROS 2 - 3.2.0, accessed July 6, 2025, https://fast-dds-monitor.readthedocs.io/en/latest/rst/ros/ros.html
23. Subscribe to ROS2 topic from native ePromisa DDS program - Stack Overflow, accessed July 6, 2025, https://stackoverflow.com/questions/68272082/subscribe-to-ros2-topic-from-native-epromisa-dds-program
24. 1.6.1. Communication between Vulcanexus and Fast DDS using ROS 2 generated IDLs -, accessed July 6, 2025, https://docs.vulcanexus.org/en/latest/rst/tutorials/core/deployment/dds2vulcanexus/dds2vulcanexus_ros2idl.html
25. Invisible topic between ROS2 local and ROS2 Isaac SIM - NVIDIA Developer Forums, accessed July 6, 2025, https://forums.developer.nvidia.com/t/invisible-topic-between-ros2-local-and-ros2-isaac-sim/231837
26. Galactic Geochelone changelog - ROS 2 Documentation: Rolling documentation, accessed July 6, 2025, https://docs.ros.org/en/rolling/Releases/Galactic-Geochelone-Complete-Changelog.html
27. 2.3. Features Overview - RTI Admin Console 7.5.0 documentation, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/tools/admin_console/p2_administration/features_overview/features_overview_all.html
28. AdminConsoleOverview - YouTube, accessed July 6, 2025, https://www.youtube.com/watch?v=pQtOP28K094
29. RoArm-M1 Tutorial VIII: Use of rqt in ROS2 - Waveshare Wiki, accessed July 6, 2025, https://www.waveshare.com/wiki/RoArm-M1_Tutorial_VIII:_Use_of_rqt_in_ROS2
30. Implementing GUI Based Tools Using rqt - Machine Intelligence Laboratory, accessed July 6, 2025, https://mil.ufl.edu/docs/software/rqt.html
31. Using rqt_console to view logs - ROS 2 Documentation: Rolling documentation, accessed July 6, 2025, https://docs.ros.org/en/rolling/Tutorials/Beginner-CLI-Tools/Using-Rqt-Console/Using-Rqt-Console.html
32. Unlocking the potential of Fast DDS middleware [community ..., accessed July 6, 2025, https://docs.ros.org/en/humble/Tutorials/Advanced/FastDDS-Configuration.html
33. Fast DDS TSC RMW report 2021 - GitHub Pages, accessed July 6, 2025, https://osrf.github.io/TSC-RMW-Reports/humble/eProsima-response.html
34. Supported versions - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/stable/notes/previous_versions/supported_versions.html
35. Releases / eProsima/Fast-DDS - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-RTPS/releases


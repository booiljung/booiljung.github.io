---
layout: page
title: eProsima Fast DDS Statistics Backend
permalink: /services/dds/Fast-DDS/eProsima Fast DDS Statistics Backend
---


분산 시스템의 복잡성이 증가함에 따라, 시스템의 상태, 성능, 동작을 투명하게 관찰하는 능력은 단순한 편의 기능을 넘어 필수적인 요구사항이 되었습니다. eProsima Fast DDS 환경에서 이러한 관찰 가능성(Observability)을 제공하는 핵심 도구가 바로 Fast DDS Statistics Backend입니다. 그러나 이 백엔드를 효과적으로 활용하기 위해서는 그것이 독립적인 개체가 아니라, Fast DDS 내부에 계층적으로 설계된 더 큰 관찰 아키텍처의 사용자 지향적 구성요소라는 점을 이해하는 것이 중요합니다. 본 파트에서는 이 아키텍처의 근간을 이루는 핵심 개념들을 탐구합니다.


데이터 분산 서비스(Data Distribution Service, DDS)는 객체 관리 그룹(Object Management Group, OMG)이 정의한 표준으로, 실시간 시스템에서 데이터 중심의 발행-구독(Publish-Subscribe) 통신을 위한 프로토콜입니다.1 이 모델은 데이터를 생성하는 애플리케이션(Publisher)과 데이터를 소비하는 애플리케이션(Subscriber) 간의 효율적이고 신뢰성 있는 정보 분배를 목적으로 하며, 양측을 깔끔하게 분리하는 다대다(many-to-many) 분산 네트워크 패러다임에 기반합니다.1

eProsima Fast DDS는 이러한 DDS 사양을 고성능 C++로 구현한 라이브러리입니다.1 Fast DDS는 DDS API 구현 외에도, 타입 인터페이스와 미들웨어 구현을 연결하는 생성 도구인 Fast DDS-Gen, 그리고 표준 네트워크를 통해 메시지를 교환하기 위한 기저의 RTPS(Real-Time Publish-Subscribe) 와이어 프로토콜 구현을 포함합니다.1 RTPS는 DDS를 위한 상호운용성 프로토콜로, OMG 컨소시엄에 의해 정의되고 유지됩니다.1 모든 RTPS 엔티티들은 격리된 통신 평면을 나타내는 RTPS 도메인(Domain)에 연결되어 엔드포인트 간의 매칭이 이루어집니다.1 이러한 구조를 통해 Fast DDS는 예측 가능성, 확장성, 유연성 및 효율적인 리소스 처리를 달성합니다.


모든 모니터링의 기반은 **Fast DDS Statistics Module**에 있습니다. 이 모듈은 Fast DDS의 핵심 기능 중 하나로, 활성화될 경우 DDS 통신 코어를 계측하여 시스템 내부에서 교환되는 데이터에 대한 정보를 수집하고 제공합니다.1

이 모듈의 아키텍처는 본질적으로 "DDS-네이티브(DDS-native)"라는 점이 특징입니다. 즉, 시스템 상태를 보고하기 위해 별도의 외부 통신 채널을 사용하지 않고, Fast DDS 자체의 통신 메커니즘을 활용합니다. 모듈이 활성화되면, 내부적으로 자체 DDS 데이터 기록기(DataWriter)를 생성하여 미리 정의된 특수 목적의 토픽(Topic)들을 통해 통계 데이터를 발행(publish)합니다.2 이 통계 토픽들을 통해 발행되는 원시 데이터 스트림이 바로 Statistics Backend를 포함한 모든 모니터링 도구가 소비하는 정보의 원천이 됩니다.

이러한 설계는 매우 효율적입니다. DDS 시스템의 상태를 DDS 메시지로 전달함으로써, 모니터링 시스템은 표준 DDS 구독자(Subscriber)로 구현될 수 있습니다. 이론적으로 DDS 토픽을 구독할 수 있는 모든 도구는 이 데이터에 접근할 수 있지만, 원시 데이터는 매우 빈번하게, 그리고 파편화된 형태로 전달되므로 직접 처리하기에는 복잡성이 높습니다. 바로 이 지점에서 Statistics Backend의 필요성이 대두됩니다.


**Fast DDS Statistics Backend**는 핵심 통계 모듈이 생성하는 원시 데이터를 사용자가 쉽게 활용할 수 있도록 설계된 전용 C++ 라이브러리입니다.3 이 백엔드의 핵심 역할은 통계 모듈의 토픽들을 구독하는 특수하고 미리 구성된 DDS 구독자 애플리케이션으로 동작하는 것입니다.5

백엔드는 통계 토픽으로부터 원시 데이터를 수신한 후, 이를 처리하고 집계하여 내부적으로 쿼리가 가능한 "데이터베이스와 유사한(database-like)" 구조에 저장합니다.5 그 다음, 이 가공된 정보를 사용하기 쉬운 C++ API를 통해 노출시킵니다.4 이 과정을 통해 개발자는 수동으로 DDS 구독자를 생성하고, 통계 데이터 타입을 위한 코드를 생성하며(IDL로부터), 복잡한 데이터들을 상호 연관시키는 번거로움에서 해방됩니다.4

결론적으로 Statistics Backend의 가장 큰 가치는 추상화와 단순화에 있습니다. 이는 실시간으로 쏟아지는 원시적이고 단편적인 통계 이벤트 스트림을, 구조화되고 질의 가능한 '네트워크의 상태' 정보로 변환해주는 역할을 합니다. 개발자는 더 이상 원시 토픽 데이터를 다루는 대신, 시스템의 구조화된 모델에 질의하는 방식으로 상호작용하게 됩니다. 이를 통해 DDS 엔티티의 상태 모니터링을 강화하고, 시스템의 구조와 동작을 그래프와 같은 형태로 시각화하여 쉽게 이해할 수 있도록 돕습니다.5


Fast DDS Statistics Backend를 사용하여 모니터링 시스템을 구축하는 과정은 단순히 라이브러리를 설치하는 것 이상을 포함하는 다단계 프로세스입니다. 컴파일 시점부터 런타임에 이르기까지 여러 계층에서 정확한 설정이 요구됩니다. 본 파트에서는 개발자가 모니터링을 시작하기 위해 필요한 모든 설치 및 구성 절차를 포괄적으로 안내합니다.


Statistics Backend를 사용하기 위한 첫 단계는 소스 코드로부터 라이브러리를 빌드하고 시스템에 설치하는 것입니다. Linux 환경을 기준으로, 다음의 실용적인 단계별 가이드를 따릅니다.

먼저, 빌드에 필요한 도구들이 시스템에 설치되어 있어야 합니다. 주요 의존성은 `CMake`, `g++`, `pip3`, `wget`, `git` 등입니다.6 Ubuntu와 같은 Debian 기반 배포판에서는 다음 명령어를 사용하여 패키지를 설치할 수 있습니다.

```Bash
sudo apt install cmake g++ python3-pip wget git
```

또한, C++ 유닛 테스트 라이브러리인 Gtest는 선택적으로 설치할 수 있습니다. 기본적으로 테스트는 컴파일되지 않지만, CMake 옵션을 통해 활성화할 수 있습니다.6

다음으로, 여러 소프트웨어 패키지 세트를 빌드하기 위한 명령줄 도구인 `colcon`을 설치합니다. `colcon`과 관련 개발 도구는 `pip3`를 통해 설치할 수 있습니다.6

```Bash
pip3 install -U colcon-common-extensions vcstool
```

만약 권한 오류가 발생하면, `--user` 플래그를 추가하여 사용자 디렉토리에 설치할 수 있습니다.

이제 Statistics Backend와 그 의존성들을 다운로드하고 빌드할 작업 공간을 설정합니다. 제공되는 `.repos` 파일을 사용하여 관련 저장소를 한 번에 클론(clone)할 수 있습니다.6

```Bash
# 작업 공간 디렉토리 생성
mkdir -p Fast-DDS-statistics-backend-ws/src
cd Fast-DDS-statistics-backend-ws

# repos 파일 다운로드 및 의존성 소스 코드 가져오기
wget https://raw.githubusercontent.com/eProsima/Fast-DDS-statistics-backend/main/fastdds_statistics_backend.repos
vcs import src < fastdds_statistics_backend.repos

# colcon을 사용하여 빌드 및 설치
colcon build --symlink-install
```

이 과정을 완료하면, 빌드된 라이브러리와 실행 파일이 `install` 디렉토리에 생성되며, 개발 환경에서 이를 사용할 준비가 완료됩니다.


통계 데이터 발행을 활성화하는 방법은 여러 가지가 있으며, 각 방법은 서로 다른 개발 및 배포 시나리오에 적합합니다. 이들은 상호 배타적이지 않으며 조합하여 사용할 수도 있습니다.


가장 기본적인 단계는 Fast DDS 라이브러리를 빌드할 때 통계 모듈 자체를 포함시키는 것입니다. 이는 `FASTDDS_STATISTICS` CMake 옵션을 통해 제어됩니다. 중요한 점은 Fast DDS 버전 2.9.0부터 이 옵션이 **기본적으로 활성화**되어 있다는 것입니다.2 따라서 최신 버전의 Fast DDS를 사용하는 개발자는 별도의 조치 없이 이 단계를 통과할 수 있습니다. 구버전을 사용하거나 특정 이유로 이 옵션을 비활성화했다면, 빌드 시 `-DFASTDDS_STATISTICS=ON` 플래그를 명시적으로 추가해야 합니다.


가장 빠르고 간편하게 통계 데이터 발행을 활성화하는 방법은 `FASTDDS_STATISTICS` 환경 변수를 사용하는 것입니다.2 이 방법은 임시 테스트나 Docker와 같은 컨테이너 환경에서 특히 유용합니다. 환경 변수에 활성화하고자 하는 통계 토픽의 키워드를 세미콜론(;)으로 구분하여 나열하면 됩니다.8

예를 들어, 지연 시간, 처리량, 물리적 데이터 토픽을 활성화하려면 다음과 같이 설정합니다.

```Bash
export FASTDDS_STATISTICS="HISTORY_LATENCY_TOPIC;PUBLICATION_THROUGHPUT_TOPIC;PHYSICAL_DATA_TOPIC"
```

활성화할 수 있는 전체 토픽 키워드 목록은 `types.idl` 파일에 정의되어 있으며, 주요 항목은 다음과 같습니다 9: 

`HISTORY2HISTORY_LATENCY`, `NETWORK_LATENCY`, `PUBLICATION_THROUGHPUT`, `SUBSCRIPTION_THROUGHPUT`, `RTPS_SENT`, `RTPS_LOST`, `RESENT_DATAS`, `HEARTBEAT_COUNT`, `DISCOVERED_ENTITY`, `PHYSICAL_DATA` 등.


애플리케이션 코드 내에서 직접 통계 데이터 기록기를 활성화하여 더 세밀한 제어를 할 수도 있습니다. 이 방법은 애플리케이션의 내부 상태에 따라 동적으로 모니터링을 켜고 끌 때 유용합니다. 예를 들어, 특정 오류 조건이 감지되었을 때만 상세 로깅을 활성화하는 시나리오에 적합합니다.

이를 위해서는 `DomainParticipant` 객체를 `eprosima::fastdds::statistics::dds::DomainParticipant`로 내로우 캐스팅(narrow casting)한 후, `enable_statistics_datawriter` 메서드를 호출해야 합니다.2

```C++
// 기존 DomainParticipant 생성
eprosima::fastdds::dds::DomainParticipant* participant =...;

// 통계 Participant로 내로우 캐스팅
eprosima::fastdds::statistics::dds::DomainParticipant* statistics_participant =
    eprosima::fastdds::statistics::dds::DomainParticipant::narrow(participant);

if (statistics_participant)
{
    // 특정 통계 토픽(예: GAP_COUNT_TOPIC) 활성화
    eprosima::fastdds::dds::ReturnCode_t ret =
        statistics_participant->enable_statistics_datawriter(
            eprosima::fastdds::statistics::GAP_COUNT_TOPIC,
            eprosima::fastdds::statistics::dds::STATISTICS_DATAWRITER_QOS);

    if (ret!= eprosima::fastdds::dds::ReturnCode_t::RETCODE_OK)
    {
        // 오류 처리
    }
}
```


재현 가능하고 버전 관리가 가능한 방식으로 통계 설정을 영구적으로 정의하려면 XML 프로필을 사용하는 것이 가장 좋습니다. 이 방법은 특히 ROS 2와 같이 시스템 동작을 파일로 정의하는 것이 표준인 환경에서 선호됩니다.10

XML 프로필 내에서 `DomainParticipant`의 속성(property)을 설정하여 통계 모듈을 활성화할 수 있습니다. 이는 `fastdds.statistics`라는 속성 키를 사용하며, 환경 변수와 동일한 기능을 제공합니다. 다음은 `participant` 프로필 내에서 통계 토픽을 활성화하는 예시입니다.10

```XML
<?xml version="1.0" encoding="UTF-8"?>
<dds xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles">
    <profiles>
        <participant profile_name="participant_with_statistics" is_default_profile="true">
            <rtps>
                <propertiesPolicy>
                    <properties>
                        <property>
                            <name>fastdds.statistics</name>
                            <value>HISTORY_LATENCY_TOPIC;PUBLICATION_THROUGHPUT_TOPIC</value>
                        </property>
                    </properties>
                </propertiesPolicy>
            </rtps>
        </participant>
    </profiles>
</dds>
```

이렇게 작성된 XML 파일은 애플리케이션이 시작될 때 로드되어야 합니다. `DomainParticipantFactory::load_XML_profiles_file()` 함수를 코드에서 직접 호출하거나, `FASTRTPS_DEFAULT_PROFILES_FILE` 환경 변수에 XML 파일의 경로를 지정하여 자동으로 로드할 수 있습니다.11 이 방법은 시스템 구성을 코드와 분리하여 관리할 수 있게 해주는 강력한 기능입니다.


Fast DDS Statistics Backend의 설정이 완료되면, 개발자는 수집된 데이터를 활용하여 시스템을 분석하고 디버깅할 수 있습니다. 백엔드가 제공하는 통계 데이터와 상호작용하는 방법은 크게 세 가지로 나눌 수 있습니다: C++ API를 통한 프로그래밍 방식의 접근, Fast DDS Monitor를 사용한 실시간 시각화, 그리고 Prometheus/Grafana 스택과의 연동을 통한 대규모 모니터링입니다. 본 파트에서는 이 세 가지 핵심적인 사용 패턴을 상세히 다룹니다.


C++ API는 개발자가 자신만의 커스텀 모니터링 애플리케이션을 구축하거나, 기존 애플리케이션에 모니터링 로직을 통합할 수 있는 가장 유연한 방법을 제공합니다. API는 비동기적 알림과 동기적 데이터 쿼리라는 두 가지 주요 상호작용 모델을 중심으로 설계되었습니다.

API의 주 진입점은 `StatisticsBackend` 싱글턴(singleton) 클래스입니다.14 이 클래스를 통해 특정 도메인에 대한 모니터링을 시작하고 중지하며, 엔티티 정보와 통계 데이터를 추출할 수 있습니다.

**비동기적 알림 (리스너 모델):** 이 모델은 네트워크의 변화나 새로운 통계 데이터의 도착과 같은 이벤트를 실시간으로 처리하는 데 적합합니다. 애플리케이션은 `IListener` 인터페이스를 구현한 리스너 객체를 `StatisticsBackend`에 등록합니다. 이벤트가 발생하면 백엔드는 해당 리스너의 콜백 함수를 호출합니다.4 이는 GUI를 실시간으로 업데이트하거나 특정 조건(예: 지연 시간 임계값 초과)에 따라 경고를 발생시키는 로직에 효율적입니다.

**동기적 쿼리 (요청-응답 모델):** 이 모델은 애플리케이션이 필요할 때 특정 시점의 시스템 상태를 조회하는 데 사용됩니다. `StatisticsBackend`는 `get_domain_view_graph()`, `get_info()`, `get_data()`와 같은 다양한 `Get...` 함수를 제공합니다.14 이를 통해 특정 엔티티 간의 지연 시간, 토픽의 처리량 등 원하는 데이터를 언제든지 요청하고 응답받을 수 있습니다. 이는 시스템 상태를 스냅샷으로 저장하거나, 주기적으로 보고서를 생성하는 작업에 유용합니다.

다음은 도메인 0을 모니터링하고, 새로운 데이터가 도착했을 때 이를 출력하는 간단한 C++ 예제입니다.4

```C++
#include <fastdds_statistics_backend/StatisticsBackend.hpp>
#include <fastdds_statistics_backend/listener/DomainListener.hpp>
#include <fastdds_statistics_backend/types/types.hpp>

#include <iostream>
#include <thread>
#include <chrono>

// 사용자 정의 리스너 클래스
class MyListener : public eprosima::statistics_backend::DomainListener
{
public:
    // 데이터 도착 시 호출되는 콜백
    void on_domain_data_available(
            eprosima::statistics_backend::EntityId domain_id) override
    {
        std::cout << "Data available on domain " << domain_id << std::endl;

        // 실제 데이터 쿼리 및 처리 로직 추가 가능
    }
};

int main()
{
    // StatisticsBackend 싱글턴 인스턴스 가져오기
    auto backend = eprosima::statistics_backend::StatisticsBackend::get_instance();

    // 사용자 리스너 생성 및 등록
    auto listener = std::make_shared<MyListener>();
    backend->set_domain_listener(listener.get());

    // 도메인 0에 대한 모니터링 시작
    eprosima::statistics_backend::EntityId domain_id = backend->init_monitor(0);

    if (!domain_id.is_valid())
    {
        std::cerr << "Error: Could not start monitoring on domain 0" << std::endl;
        return 1;
    }

    std::cout << "Monitoring started on domain 0. Press CTRL+C to exit." << std::endl;

    // 애플리케이션을 계속 실행 상태로 유지
    while (true)
    {
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }

    // 모니터링 중지 (실제로는 도달하지 않음)
    backend->stop_monitor(domain_id);

    return 0;
}
```

이 코드는 API의 기본적인 사용법을 보여주며, 이를 확장하여 복잡한 커스텀 모니터링 도구를 개발할 수 있습니다.


**Fast DDS Monitor**는 eProsima가 제공하는 공식 그래픽 사용자 인터페이스(GUI) 도구로, 코드를 한 줄도 작성하지 않고 DDS 환경을 실시간으로 모니터링하고 분석할 수 있게 해줍니다.17 이 도구는 통계 데이터가 올바르게 발행되고 있는지 확인하고, 시스템 동작에 대한 직관적인 이해를 얻는 데 가장 이상적인 출발점입니다.

Fast DDS Monitor 사용법은 다음과 같은 단계로 이루어집니다 8:

1. **모니터 시작 및 도메인 연결:** 애플리케이션을 실행하고 "Start monitoring!" 버튼을 클릭합니다. 모니터링할 DDS 도메인 ID(예: 0)를 입력하여 연결을 시작합니다.
2. **패널 및 엔티티 탐색:** 기본적으로 DDS 패널이 표시됩니다. 상단의 '///' 버튼을 통해 논리적(Logical) 패널과 물리적(Physical) 패널을 추가하여 전체 정보를 시각화할 수 있습니다. 왼쪽의 탐색기 패널에는 발견된 호스트, 사용자, 프로세스, 참여자(Participant), 데이터 기록기(DataWriter), 데이터 판독기(DataReader) 등 모든 DDS 엔티티가 계층 구조로 표시됩니다.
3. **그래프 뷰(Graph View):** `Domain View` 탭을 클릭하면 네트워크 토폴로지가 그래프 형태로 시각화됩니다. 이 뷰를 통해 어떤 퍼블리셔가 어떤 토픽에 데이터를 쓰고, 어떤 서브스크라이버가 읽어 가는지 한눈에 파악할 수 있습니다. 특정 토픽을 기준으로 필터링하는 것도 가능합니다.20
4. **통계 데이터 차트 생성:**
   - **과거 데이터 플롯:** `Chart View`에서 `Edit -> Display Historical Data`를 선택하여 과거 데이터를 기반으로 한 정적 차트를 생성할 수 있습니다. 예를 들어, `FASTDDS_LATENCY` 토픽을 선택하고, 소스 엔티티와 대상 엔티티를 지정한 후, 통계 종류(예: `MEAN`, `MAX`, `MIN`)를 선택하면 해당 기간의 지연 시간 통계 그래프가 그려집니다.
   - **실시간 동적 차트:** 상단의 실시간 데이터 표시 아이콘을 클릭하여 동적 차트를 생성할 수 있습니다. 시간 창(예: 1 minute)과 업데이트 주기(예: 5 seconds)를 설정하면, 차트가 주기적으로 자동 업데이트되면서 실시간 데이터를 보여줍니다.

Fast DDS Monitor는 개발 초기 단계에서 시스템의 동작을 검증하고, 복잡한 통신 문제를 직관적으로 진단하는 데 매우 강력한 도구입니다.


대규모 시스템이나 프로덕션 환경에서는 일시적인 시각화 도구를 넘어, 장기적인 데이터 수집, 저장, 경고 기능을 갖춘 견고한 모니터링 스택이 필요합니다. Fast DDS Statistics Backend는 업계 표준 관찰 가능성 도구인 **Prometheus** 및 **Grafana**와 원활하게 연동되도록 설계되어 이러한 요구를 충족합니다.21

이 연동 패턴은 Fast DDS 통계 생태계가 폐쇄적인 구조가 아님을 보여줍니다. DDS 성능 데이터를 서버, 데이터베이스, 다른 애플리케이션의 메트릭과 함께 기존의 중앙 집중식 모니터링 대시보드에 통합할 수 있게 함으로써, 전사적인 관찰 가능성 전략의 일부로 활용될 수 있습니다.

연동 과정은 Vulcanexus 튜토리얼에 잘 설명되어 있으며, 핵심적인 역할은 `ROS2Prometheus` 예제 애플리케이션이 담당합니다.21

1. **Prometheus Exporter 설정:** 통계 데이터를 Prometheus가 수집할 수 있는 형식으로 노출시키기 위해 C++ Prometheus exporter 라이브러리를 설치합니다.
2. **`ROS2Prometheus` 애플리케이션 빌드 및 실행:** 이 예제 애플리케이션은 Fast DDS Statistics Backend를 사용하여 통계 데이터를 수집하고, 이를 Prometheus exporter를 통해 HTTP 엔드포인트로 노출시키는 '브릿지' 역할을 합니다.21
3. **Prometheus 구성:** `prometheus.yml` 설정 파일에 `ROS2Prometheus` 애플리케이션이 노출하는 엔드포인트를 스크레이프(scrape)하도록 작업을 추가합니다.
4. **Prometheus에서 데이터 확인:** Prometheus를 실행하고 웹 UI에 접속하면, `fastdds_latency`, `publication_throughput`과 같은 메트릭이 수집되고 있는 것을 확인할 수 있습니다.
5. **Grafana 대시보드 생성:** Grafana를 설치하고 Prometheus를 데이터 소스로 추가합니다. 그 다음, Prometheus에 저장된 메트릭을 사용하여 풍부하고 상호작용적인 시각화 대시보드를 구축할 수 있습니다.23 이를 통해 시간 경과에 따른 성능 추이를 분석하고, 임계값 기반의 경고 규칙을 설정하는 등 고급 모니터링 기능을 구현할 수 있습니다.


Robot Operating System 2 (ROS 2)는 내부 통신 미들웨어로 DDS를 채택했으며, eProsima Fast DDS는 여러 ROS 2 배포판의 기본 미들웨어로 사용되고 있습니다.25 따라서 Fast DDS Statistics Backend는 ROS 2 개발자들에게 매우 강력하고 유용한 도구입니다. ROS 2는 DDS의 복잡성을 추상화하여 사용 편의성을 높이지만, 이로 인해 때로는 성능 저하의 원인이나 통신 문제를 진단하기 어려워집니다.26 Statistics Backend는 이 추상화 계층 아래를 들여다볼 수 있는 "창"을 제공하여, ROS 2 개발자가 시스템의 실제 동작을 이해하고 최적화할 수 있도록 돕습니다.


ROS 2 환경에서 커스텀 모니터링 노드를 개발하기 위한 첫 단계는 `fastdds_statistics_backend` 라이브러리에 의존하는 새로운 ROS 2 패키지를 생성하는 것입니다. Vulcanexus 문서에 설명된 절차를 기반으로, 다음과 같은 실용적인 튜토리얼을 따를 수 있습니다.15

1. ROS 2 작업 공간 및 패키지 생성:

   먼저, ament_cmake 빌드 타입을 사용하는 새로운 ROS 2 패키지를 생성합니다. ros2 pkg create 명령어에 --dependencies 옵션을 사용하여 fastdds_statistics_backend에 대한 의존성을 명시합니다.

   ```Bash
   mkdir -p ros2_ws/src
   cd ros2_ws/src
   ros2 pkg create --build-type ament_cmake monitor_tutorial --dependencies fastdds_statistics_backend rclcpp
   ```
   
2. package.xml 수정:

   생성된 monitor_tutorial 패키지 안의 package.xml 파일을 열어, fastdds_statistics_backend에 대한 <depend> 태그가 올바르게 추가되었는지 확인합니다. 이는 colcon이 빌드 시 필요한 의존성을 해결하는 데 사용됩니다.

   ```XML
   <depend>fastdds_statistics_backend</depend>
   <depend>rclcpp</depend>
   ```
   
3. CMakeLists.txt 수정:

   CMakeLists.txt 파일은 빌드 구성을 정의합니다. find_package를 사용하여 fastdds_statistics_backend 패키지를 찾고, 실행 파일을 생성할 때 해당 라이브러리를 링크해야 합니다.

   ```CMake
   #... (상단부 생략)
   
   find_package(ament_cmake REQUIRED)
   find_package(rclcpp REQUIRED)
   find_package(fastdds_statistics_backend REQUIRED)
   
   add_executable(monitor_node src/monitor.cpp)
   ament_target_dependencies(monitor_node rclcpp)
   
   # fastdds_statistics_backend 라이브러리 링크
   target_link_libraries(monitor_node fastdds_statistics_backend::fastdds_statistics_backend)
   
   install(TARGETS
     monitor_node
     DESTINATION lib/${PROJECT_NAME}
   )
   
   #... (하단부 생략)
   ```
   
4. 모니터링 노드 소스 코드 작성 (src/monitor.cpp):

   이제 src 디렉토리 안에 monitor.cpp 파일을 생성하고, 제 3.1절에서 설명한 C++ API를 사용하여 모니터링 로직을 구현합니다. 이 노드는 ROS 2 노드이면서 동시에 Statistics Backend를 사용하여 DDS 통계를 수집하는 역할을 합니다.

이러한 과정을 통해 ROS 2 개발자는 자신만의 모니터링 노드를 쉽게 구축하고, 이를 기존 ROS 2 시스템에 통합하여 특정 통신 성능을 측정하거나 디버깅할 수 있습니다.


커스텀 모니터링 패키지가 준비되면, 이를 활용하여 표준 ROS 2 노드 간의 통신을 분석할 수 있습니다. 예를 들어, ROS 2 `demo_nodes_cpp` 패키지에 포함된 `talker`와 `listener` 노드를 실행하고, 이들의 통신 상태를 Statistics Backend를 통해 관찰하는 시나리오를 생각해 볼 수 있습니다.15

ROS 2 개발자가 `talker` 노드를 실행하면, ROS 2의 미들웨어 계층(RMW)은 내부적으로 Fast DDS `DomainParticipant`를 생성하고, `/chatter` 토픽에 대한 `DataWriter`를 만듭니다. 마찬가지로 `listener` 노드는 `DataReader`를 생성합니다. ROS 2의 추상화된 개념인 '노드'와 '토픽'은 실제로는 DDS의 'Participant', 'DataWriter', 'DataReader'와 같은 구체적인 엔티티에 매핑됩니다.

표준 ROS 2 도구인 `ros2 topic echo`나 `rqt_graph`는 이러한 기저의 DDS 엔티티에 대한 상세 정보를 제공하지 않습니다. 그러나 Statistics Backend를 사용하면 다음과 같은 심층적인 분석이 가능합니다:

- **엔티티 발견:** 모니터링 노드는 `talker`와 `listener`에 해당하는 `DomainParticipant`가 네트워크에 나타나는 것을 감지할 수 있습니다.
- **QoS 설정 확인:** 각 `DataWriter`와 `DataReader`의 실제 QoS(Quality ofService) 설정을 조회하여, 신뢰성(reliability), 내구성(durability) 등의 설정이 의도대로 적용되었는지 확인할 수 있습니다. 이는 QoS 불일치로 인한 통신 실패를 디버깅하는 데 매우 중요합니다.
- **성능 메트릭 측정:** `talker`의 `DataWriter`와 `listener`의 `DataReader` 사이의 실제 종단 간 지연 시간(`HISTORY2HISTORY_LATENCY`)을 마이크로초 단위로 측정할 수 있습니다. 또한, `talker`가 발행하는 실제 데이터 처리량(`PUBLICATION_THROUGHPUT`)도 확인할 수 있습니다.
- **네트워크 트래픽 분석:** RTPS 레벨에서 실제로 전송된 패킷 수(`RTPS_SENT`), 유실된 패킷 수(`RTPS_LOST`), 재전송된 데이터 수(`RESENT_DATAS`) 등을 관찰하여 네트워크 상태를 진단할 수 있습니다.

이처럼 Statistics Backend는 ROS 2의 추상화 계층을 투과하여 DDS 레벨의 실제 동작을 직접 관찰할 수 있게 해주는 강력한 디버깅 도구입니다. 이를 통해 개발자는 "왜 내 노드가 통신하지 못하는가?" 또는 "왜 통신이 느린가?"와 같은 어려운 질문에 대한 답을 찾을 수 있습니다.


Fast DDS Statistics Backend는 강력한 모니터링 기능을 제공하지만, 모든 기능에는 비용이 따릅니다. 통계 모듈을 활성화하는 것이 시스템 성능에 미치는 영향을 이해하는 것은 매우 중요합니다. 또한, 수집된 다양한 통계 메트릭을 정확하게 해석하기 위해서는 각 데이터가 무엇을 의미하는지에 대한 명확한 이해가 필요합니다. 본 파트에서는 통계 기능 사용에 따른 성능 고려사항을 분석하고, 주요 통계 메트릭에 대한 상세한 참조 가이드를 제공합니다.


Fast DDS는 그 자체로 고성능을 목표로 설계된 미들웨어입니다. 특히 프로세스 내(intra-process) 통신이나 공유 메모리(Shared Memory, SHM) 전송을 사용할 때 매우 낮은 지연 시간을 보이며, 제로 카피(zero-copy) 메커니즘을 통해 데이터 복사로 인한 오버헤드를 최소화합니다.27

통계 모듈을 활성화하면 시스템에 필연적으로 오버헤드가 추가됩니다. 현재 제공된 자료에서는 이 오버헤드를 정량적으로 측정한 구체적인 수치는 없지만 28, 그 성격은 논리적으로 추론할 수 있습니다. 오버헤드는 주로 다음 두 가지 요인에서 발생합니다:

1. **계측 및 데이터 수집:** 통신 파이프라인의 여러 지점에서 지연 시간, 처리량 등을 측정하고 데이터를 수집하는 데 필요한 CPU 사이클이 소모됩니다.
2. **통계 데이터 발행:** 수집된 통계 데이터는 그 자체로 또 다른 DDS 데이터입니다. 이를 직렬화(serialization)하고, 네트워크를 통해 전송하며(자체 통계 DataWriter 사용), 모니터링 애플리케이션에서 수신하여 역직렬화(deserialization)하는 과정에서 CPU 및 네트워크 리소스가 추가로 사용됩니다.2

이 오버헤드는 고정된 비용이 아니라, 사용자가 제어할 수 있는 가변적인 비용입니다. 오버헤드의 크기는 활성화된 통계 토픽의 수와 데이터 발행 빈도에 정비례합니다. 따라서 성능에 미치는 영향을 최소화하기 위한 가장 중요한 전략은 **필요한 통계 토픽만 선택적으로 활성화**하는 것입니다. 예를 들어, 애플리케이션 간의 종단 간 지연 시간만 디버깅하고 싶다면 `HISTORY2HISTORY_LATENCY` 토픽만 활성화하고, 처리량이나 패킷 손실과 관련된 다른 모든 토픽은 비활성화하여 모니터링 시스템 자체의 부하를 줄일 수 있습니다.

또한, 통계 데이터가 제대로 수신되지 않거나 일부만 도착하는 경우, 통계 데이터를 발행하는 엔드포인트의 메모리 제약이 너무 엄격하게 설정되었을 수 있습니다. 이 경우, 관련 엔드포인트의 리소스 제한(예: 버퍼 크기)을 더 여유롭게 설정해야 할 수 있다는 경고가 문서에 명시되어 있습니다.5 이는 통계 데이터 자체도 DDS의 QoS 정책과 리소스 관리의 영향을 받는다는 점을 시사하는 중요한 문제 해결 팁입니다.


Statistics Backend가 제공하는 데이터를 효과적으로 활용하려면 각 메트릭이 무엇을 의미하는지 정확히 알아야 합니다. 다음 표는 `types.idl` 파일에 정의된 주요 통계 토픽과 그 의미를 정리한 데이터 사전입니다.9 이는 Fast DDS Monitor에서 보거나 API를 통해 수신한 데이터를 해석하는 데 필수적인 참조 자료가 될 것입니다.

**표 1: 주요 통계 토픽과 그 의미**

| 메트릭 이름 / EventKind   | 데이터 구조             | 설명                                                         | 주요 사용 사례                                               |
| ------------------------- | ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `HISTORY2HISTORY_LATENCY` | `WriterReaderData`      | 데이터 샘플이 DataWriter의 `write()` 호출에 의해 히스토리 캐시에 기록된 시점부터, 해당 샘플이 매칭되는 DataReader의 히스토리 캐시에 수신될 때까지 경과된 시간(종단 간 지연 시간). | 애플리케이션 레벨의 종단 간 성능 측정 및 디버깅.             |
| `NETWORK_LATENCY`         | `Locator2LocatorData`   | 특정 네트워크 로케이터(IP 주소 및 포트)에서 다른 로케이터로 데이터가 전송되는 데 걸리는 네트워크 전송 지연 시간. | 물리적 네트워크 경로의 성능 문제 진단.                       |
| `PUBLICATION_THROUGHPUT`  | `EntityData`            | 특정 DataWriter가 초당 발행하는 데이터의 양 (보통 바이트/초 단위). | 퍼블리셔의 데이터 전송 성능 및 병목 현상 분석.               |
| `SUBSCRIPTION_THROUGHPUT` | `EntityData`            | 특정 DataReader가 초당 수신하는 데이터의 양 (보통 바이트/초 단위). | 서브스크라이버의 데이터 수신 성능 및 처리 능력 확인.         |
| `RTPS_SENT`               | `Entity2LocatorTraffic` | 특정 엔티티(DataWriter)에서 특정 로케이터로 보낸 RTPS 패킷의 수와 총 바이트. | 네트워크 트래픽 발생량 모니터링.                             |
| `RTPS_LOST`               | `Entity2LocatorTraffic` | 특정 엔티티(DataWriter)에서 보냈지만 특정 로케이터에 도달하지 못한 것으로 간주되는 RTPS 패킷의 수. (신뢰성 통신에서만 의미 있음) | 패킷 손실률을 통한 네트워크 안정성 평가.                     |
| `RESENT_DATAS`            | `EntityCount`           | 신뢰성 통신(Reliable QoS)에서 유실된 데이터를 재전송한 횟수. | 네트워크 문제로 인한 재전송 오버헤드 분석.                   |
| `HEARTBEAT_COUNT`         | `EntityCount`           | DataWriter가 자신의 상태를 알리기 위해 보낸 하트비트(Heartbeat) 메시지의 총 수. | 신뢰성 통신 프로토콜의 동작 상태 확인.                       |
| `ACKNACK_COUNT`           | `EntityCount`           | DataReader가 수신한 데이터에 대해 긍정/부정 응답(ACK/NACK)을 보낸 횟수. | 신뢰성 통신 프로토콜의 피드백 메커니즘 분석.                 |
| `DISCOVERED_ENTITY`       | `DiscoveryTime`         | 로컬 참여자가 원격 엔티티(다른 참여자, 기록기, 판독기)를 발견했을 때의 타임스탬프와 관련 정보. | 시스템의 동적 발견 프로세스 추적 및 디버깅.                  |
| `PHYSICAL_DATA`           | `PhysicalData`          | 특정 참여자가 실행되고 있는 물리적 환경 정보(호스트 이름, 사용자 이름, 프로세스 ID). | DDS 엔티티가 어떤 물리적 머신과 프로세스에 배포되었는지 매핑. |


지금까지 eProsima Fast DDS Statistics Backend의 기초 개념, 아키텍처, 설정 방법, 그리고 다양한 활용 패턴에 대해 심도 있게 탐구했습니다. 본 마지막 파트에서는 전체 분석을 종합하여, 개발자가 자신의 요구사항에 가장 적합한 모니터링 전략을 수립할 수 있도록 돕는 비교 분석과 실용적인 최적 활용 방안을 제시합니다.


Statistics Backend를 활용하는 세 가지 주요 방법(C++ API, Fast DDS Monitor, Prometheus/Grafana 연동)은 각각 뚜렷한 장단점을 가집니다. 어떤 방법을 선택할지는 당면 과제의 성격, 요구되는 유연성 수준, 그리고 시스템의 규모에 따라 달라집니다. 다음 표는 각 방법론을 여러 기준에 따라 비교하여, 사용자가 정보에 입각한 결정을 내릴 수 있도록 돕습니다.8

**표 2: 모니터링 방법론 비교**

| 기준                       | C++ API를 통한 프로그래밍                                    | Fast DDS Monitor (GUI)                                   | Prometheus/Grafana 연동                                      |
| -------------------------- | ------------------------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------ |
| **주요 사용 사례**         | 커스텀 경고, 자동화된 테스트, 애플리케이션 로직과의 긴밀한 통합. | 대화형 디버깅, 실시간 시각화, 시스템 동작의 직관적 탐색. | 장기적인 프로덕션 모니터링, 추세 분석, 중앙 집중식 대시보드. |
| **설정 용이성**            | 높음 (C++ 개발 및 빌드 필요)                                 | 낮음 (설치 후 즉시 사용 가능)                            | 중간 (Exporter, Prometheus, Grafana 설정 필요)               |
| **유연성 및 커스터마이징** | 최대 (모든 로직을 코드로 구현)                               | 보통 (제공되는 뷰와 차트 기능 내에서 커스터마이징)       | 높음 (PromQL 쿼리와 Grafana 대시보드를 통한 높은 유연성)     |
| **확장성**                 | 애플리케이션에 의존적                                        | 데스크톱 수준 (대규모 데이터 시각화 시 성능 저하 가능)   | 높음 / 엔터프라이즈급 (분산 수집 및 저장을 위해 설계됨)      |
| **리소스 사용량**          | 최소 (필요한 로직만 구현)                                    | 보통 (GUI 렌더링 및 데이터 처리에 리소스 소모)           | 높음 (전체 스택이 상당한 리소스 요구)                        |

이 비교를 통해 명확한 의사결정 프레임워크를 도출할 수 있습니다. "내 퍼블리셔가 데이터를 보내고 있는지 빠르게 확인하고 싶다"면 Fast DDS Monitor가 최적의 선택입니다. "우리 로봇 플릿의 전반적인 통신 상태를 보여주는 영구적인 상태 대시보드를 구축해야 한다"면 Prometheus/Grafana 연동이 정답입니다. "지연 시간이 100ms를 초과하면 애플리케이션 코드 내에서 특정 복구 로직을 트리거하고 싶다"면 C++ API를 사용해야 합니다.


Fast DDS Statistics Backend의 잠재력을 최대한 활용하기 위해, 다음과 같은 실용적인 최적 활용 방안을 권장합니다.

1. **Fast DDS Monitor로 시작하십시오:** 모든 모니터링 작업의 첫 단계는 항상 Fast DDS Monitor를 사용하는 것이 좋습니다. 이를 통해 통계 데이터가 올바르게 생성되고 있는지 시각적으로 즉시 확인하고, 코드를 작성하기 전에 시스템 동작에 대한 직관을 얻을 수 있습니다.
2. **토픽을 선택적으로 사용하십시오:** 성능 오버헤드를 최소화하기 위해, 당면 과제에 필요한 특정 통계 토픽만 활성화하십시오. 모든 토픽을 무분별하게 활성화하는 것은 불필요한 시스템 부하를 유발할 수 있습니다.
3. **재현 가능한 구성을 위해 XML을 사용하십시오:** 테스트 또는 프로덕션 환경을 대상으로 하는 모든 시스템에서는, 통계 구성을 XML 프로필에 명시적으로 정의하고 이를 소스 코드와 함께 버전 관리 시스템에 커밋하십시오. 이는 배포의 일관성과 재현성을 보장합니다.
4. **디버깅 도구로써 활용하십시오:** Statistics Backend는 단순히 성능 메트릭을 위한 도구가 아닙니다. 복잡한 ROS 2/DDS 상호작용 문제나 QoS 불일치 문제를 해결하는 데 매우 귀중한 디버깅 도구라는 점을 기억하십시오. 추상화 계층 아래의 실제 동작을 보여줌으로써 문제의 근본 원인을 파악하는 데 도움을 줍니다.
5. **메모리 제약을 염두에 두십시오:** 통계 데이터가 유실되는 현상이 발생하면, 관련 엔드포인트의 메모리 제약이 원인일 수 있습니다.5 XML 프로필이나 QoS 설정을 통해 관련 버퍼 크기를 늘리는 것을 고려해야 합니다. 이는 종종 간과되지만 매우 중요한 문제 해결 단계입니다.

결론적으로, eProsima Fast DDS Statistics Backend는 단순한 데이터 수집기를 넘어, 현대 분산 시스템에 필수적인 깊이 있는 관찰 가능성을 제공하는 정교한 도구 모음입니다. 그 아키텍처를 이해하고, 다양한 구성 및 사용 패턴을 숙지함으로써 개발자는 더 안정적이고, 예측 가능하며, 고성용의 실시간 애플리케이션을 구축할 수 있을 것입니다.


1. eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/
2. 15.10. Statistics module - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/use_cases/statistics_module/statistics_module.html
3. Fast-DDS-statistics-backend/docs/conf.py at main - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-statistics-backend/blob/main/docs/conf.py
4. eProsima/Fast-DDS-statistics-backend - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-statistics-backend
5. eProsima Fast DDS Statistics Backend Documentation - Fast DDS Statistics Backend 2.2.0 documentation, accessed July 6, 2025, https://fast-dds-statistics-backend.readthedocs.io/
6. eProsima Fast DDS Statistics Backend Documentation, accessed July 6, 2025, https://fast-dds-statistics-backend.readthedocs.io/_/downloads/en/latest/pdf/
7. Releases / eProsima/Fast-DDS-statistics-backend - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-statistics-backend/releases
8. 3. Example of usage - 3.2.0, accessed July 6, 2025, https://fast-dds-monitor.readthedocs.io/en/latest/rst/getting_started/tutorial.html
9. Fast-DDS/include/fastdds/statistics/types.idl at master - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS/blob/master/include/fastdds/statistics/types.idl
10. Unlocking the potential of Fast DDS middleware [community ... - ROS 2, accessed July 6, 2025, https://docs.ros.org/en/rolling/Tutorials/Advanced/FastDDS-Configuration.html
11. 1.3.1. Configuring Fast-DDS QoS via XML profiles - Vulcanexus Documentation, accessed July 6, 2025, https://docs.vulcanexus.org/en/iron/rst/tutorials/core/qos/xml_profiles/xml_profiles.html
12. Fast-DDS-docs/docs/fastdds/xml_configuration/making_xml_profiles.rst at master - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-docs/blob/master/docs/fastdds/xml_configuration/making_xml_profiles.rst
13. 10.1. Creating an XML profiles file - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/2.x/fastdds/xml_configuration/making_xml_profiles.html
14. 1. StatisticsBackend - Fast DDS Statistics Backend 2.2.0 ..., accessed July 6, 2025, https://fast-dds-statistics-backend.readthedocs.io/en/latest/rst/statistics_backend/statistics_backend.html
15. 2.2. Monitoring ROS 2 with Fast DDS Statistics Backend - Vulcanexus Documentation, accessed July 6, 2025, https://docs.vulcanexus.org/en/iron/rst/tutorials/tools/monitor_backend/monitor_backend.html
16. 4. Full example - eProsima Fast DDS Statistics Backend Documentation - Read the Docs, accessed July 6, 2025, https://fast-dds-statistics-backend.readthedocs.io/en/latest/rst/full_example.html
17. Monitor Your DDS Network in Real-Time with eProsima Fast DDS Monitor! - YouTube, accessed July 6, 2025, https://www.youtube.com/watch?v=KMEDt1fedI8
18. README.md - eProsima/Fast-DDS-monitor - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS-monitor/blob/main/README.md
19. 3. Example of usage - Fast DDS Monitor 1.5.0 documentation, accessed July 6, 2025, https://fast-dds-monitor.readthedocs.io/en/v1.5.0/rst/getting_started/tutorial.html
20. eProsima Fast DDS Monitor - Graph View Demo - YouTube, accessed July 6, 2025, https://www.youtube.com/watch?v=YxIklcaJV48
21. 2.5. ROS 2 network statistics inspection with Prometheus ..., accessed July 6, 2025, https://docs.vulcanexus.org/en/iron/rst/tutorials/tools/prometheus/prometheus.html
22. DDS Prometheus Exporter - ROS General - Open Robotics Discourse, accessed July 6, 2025, https://discourse.ros.org/t/dds-prometheus-exporter/22006
23. Grafana integrations | Grafana Cloud documentation, accessed July 6, 2025, https://grafana.com/docs/grafana-cloud/monitor-infrastructure/integrations/
24. Data sources | Grafana documentation, accessed July 6, 2025, https://grafana.com/docs/grafana/latest/datasources/
25. 16. ROS 2 using Fast DDS middleware - 3.2.2, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/ros2/ros2.html
26. robotics - What's the difference between ROS2 and DDS? - Stack ..., accessed July 6, 2025, https://stackoverflow.com/questions/51187676/whats-the-difference-between-ros2-and-dds
27. eProsima Fast DDS Performance, accessed July 6, 2025, https://www.eprosima.com/developer-resources/performance/eprosima-fast-dds-performance
28. Fast DDS TSC RMW report 2021 - GitHub Pages, accessed July 6, 2025, https://osrf.github.io/TSC-RMW-Reports/humble/eProsima-response.html


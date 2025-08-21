# eProsima Integration Service 시스템 아키텍처 및 통합

## 1.  현대적 통합의 과제와 eProsima 패러다임

현대의 분산 시스템은 로보틱스, 사물 인터넷(IoT), 자동차, 항공우주 등 다양한 분야에서 점점 더 복잡해지고 있습니다. 이러한 시스템들은 종종 서로 다른 통신 프로토콜, 데이터 모델, 프로그래밍 언어를 사용하는 이기종 하위 시스템들로 구성됩니다. 이러한 이질성은 시스템 통합을 매우 어려운 과제로 만들며, 확장성, 유지보수성, 개발 비용 측면에서 심각한 문제를 야기합니다. eProsima Integration Service는 이러한 근본적인 문제에 대한 전략적 해결책으로 등장했습니다.

### 1.1  N² 문제: 포인트-투-포인트 브리지의 확장성 위기

분산 시스템 통합에서 전통적으로 사용되던 방식은 각 시스템 쌍을 직접 연결하는 '포인트-투-포인트(Point-to-Point)' 브리지를 개발하는 것이었습니다. 이 방식은 연결해야 할 시스템의 수가 적을 때는 직관적이고 간단해 보일 수 있습니다. 그러나 시스템의 규모가 커짐에 따라 이 접근법의 근본적인 한계가 드러납니다.

N개의 서로 다른 시스템을 상호 연결해야 할 경우, 이론적으로 최대 N×(N−1)/2개의 개별 브리지가 필요하게 됩니다. 이는 시스템의 수가 증가함에 따라 개발 및 유지보수해야 할 브리지의 수가 기하급수적으로 늘어나는 '조합적 폭발(combinatorial explosion)'을 의미합니다.1 예를 들어, 5개의 시스템을 통합하려면 최대 10개의 브리지가 필요하지만, 10개의 시스템을 통합하려면 최대 45개의 브리지가 필요합니다.

이러한 접근법은 여러 가지 심각한 문제를 야기합니다 2:

- **개발 비용 증가**: 새로운 시스템이 추가될 때마다 기존의 모든 시스템과 연결하기 위한 새로운 브리지를 개발해야 합니다.
- **유지보수 복잡성**: 각 브리지는 독립적인 소프트웨어 구성 요소이므로, 프로토콜이 변경되거나 버그가 발생했을 때 수정하고 테스트해야 할 대상이 많아집니다.
- **낮은 재사용성**: 특정 시스템 쌍을 위해 개발된 브리지는 다른 시스템에 재사용하기 어렵습니다.
- **비확장성**: 시스템의 복잡성이 증가할수록 통합에 드는 노력과 비용이 감당할 수 없는 수준으로 증가하여 사실상 확장이 불가능해집니다.

이러한 N² 문제는 복잡한 시스템의 진화를 가로막는 주요 장애물이며, 보다 효율적이고 확장 가능한 통합 패러다임의 필요성을 명확히 보여줍니다.

### 1.2  Integration Service 소개: 중앙 집중식 허브-앤-스포크 아키텍처

eProsima Integration Service는 앞서 언급된 N² 문제에 대한 근본적인 해결책을 제시합니다.1 이 서비스는 개별 시스템들이 서로 직접 통신하는 대신, 중앙의 허브(Hub)를 통해 메시지를 교환하는 '허브-앤-스포크(Hub-and-Spoke)' 모델을 채택합니다. 이 아키텍처에서 Integration Service는 서로 다른 언어를 사용하는 시스템들 사이에서 메시지를 중개하고 전달하는 중앙 집중식 메시지 전달 도구 역할을 합니다.3

이 모델의 핵심적인 장점은 확장성 문제를 해결하는 방식에 있습니다. 모든 시스템은 오직 중앙의 코어(Core)와만 통신하면 됩니다. 따라서 N개의 시스템을 통합하기 위해 필요한 소프트웨어 구성 요소(플러그인)의 수는 N개에 비례하여 선형적으로 증가합니다.1 이는 포인트-투-포인트 방식의 N² 증가와 극명한 대조를 이룹니다.

이러한 구조적 이점은 단순히 기술적 세부사항에 그치지 않고, 대규모 시스템 통합의 핵심적인 난제인 비용, 복잡성, 장기적인 유지보수성을 직접적으로 해결하는 전략적 결정입니다. 새로운 기술을 시스템에 도입할 때, 해당 기술을 위한 플러그인 하나만 개발하면 기존에 연결된 모든 시스템과 즉시 통신할 수 있게 됩니다. 이는 시스템의 민첩성을 극대화하고 기술 부채를 줄이는 효과적인 방법론을 제공합니다.

### 1.3  프로젝트의 기원과 진화: SOSS에서 성숙한 eProsima 제품으로

eProsima Integration Service는 하루아침에 만들어진 것이 아니라, 실제 산업 현장의 요구를 반영하며 점진적으로 발전해 온 역사를 가지고 있습니다.

이 프로젝트의 전신은 'SOSS(System-Of-Systems Synthesizer)'로, 현재도 Integration Service의 핵심 개념을 형성하고 있습니다.1 SOSS는 eProsima와 로봇 운영체제(ROS)의 관리 기관인 Open Robotics 간의 공동 노력으로 탄생했습니다.6 이는 프로젝트 초기부터 로보틱스 분야의 복잡한 통합 요구사항을 해결하는 데 중점을 두었음을 시사합니다.

초기 개발은 유럽 연합(EU)의 Horizon 2020 연구 혁신 프로그램의 일환인 ROS-Industrial(ROSIN)로부터 자금을 지원받는 집중 기술 프로젝트(FTP)로 시작되었습니다. 당시 프로젝트명은 "ROS2 Integration Service"였으며, 주요 목표는 ROS 2를 MQTT, ZeroMQ와 같은 다른 프로토콜과 쉽게 연결하고, WAN/인터넷을 통한 ROS 2 메시지 통신을 가능하게 하는 것이었습니다.8

현재는 eProsima가 단독으로 프로젝트의 유지보수 및 개발을 책임지고 있습니다.6 특히 버전 3.0.0은 이 프로젝트의 중요한 전환점이었습니다. 이 버전에서 SOSS라는 이름에서 공식적으로 eProsima Integration Service로 명칭이 변경되었고, C++ 네임스페이스가 `soss::core`에서 `eprosima::is::core`로 변경되는 등 전체적인 프로젝트 구조가 재편되었습니다.5

이러한 발전 과정은 Integration Service가 학술적 연구에 머무르지 않고, 로보틱스 및 산업 자동화라는 까다로운 분야의 실제 문제를 해결하기 위해 설계된, 검증된 솔루션임을 보여줍니다. Open Robotics 및 ROS-Industrial과의 협력은 이 도구의 신뢰성을 높이고, 잠재적 사용자들이 그 성숙도와 적합성을 평가하는 데 중요한 근거를 제공합니다.

## 2.  아키텍처 심층 분석: 코어, 시스템 핸들, 그리고 xTypes

eProsima Integration Service의 강력함과 유연성은 그 독창적인 아키텍처에 기인합니다. 이 아키텍처는 프로토콜별 상호작용, 일반적인 메시지 라우팅, 데이터 표현이라는 세 가지 핵심 관심사를 능숙하게 분리하여 높은 수준의 모듈성과 확장성을 달성합니다. 이 섹션에서는 Integration Service를 구성하는 핵심 요소인 코어 엔진, 시스템 핸들, xTypes 공통 언어, 그리고 Mix 파일의 역할과 상호작용을 심층적으로 분석합니다.

### 2.1  Integration Service 코어 엔진: 중앙 메시지 전달 허브

Integration Service의 심장부에는 **코어 엔진(Core Engine)**이 자리 잡고 있습니다.3 코어는 임의의 수의 미들웨어를 연결할 수 있는 플러그인 기반 플랫폼을 제공하는 중앙 허브입니다.3 코어의 주된 책임은 서로 다른 시스템 핸들(System Handle) 간의 메시지를 라우팅하는 것입니다. 중요한 점은 코어 자체가 ROS, DDS, WebSocket과 같은 개별 프로토콜의 세부 사항을 이해하지 못한다는 것입니다. 코어는 오직 모든 시스템 핸들과 공유하는 '공통 언어'만을 사용합니다.11

메시지 전달 과정은 다음과 같이 이루어집니다. 소스 시스템 핸들로부터 메시지를 수신하면, 코어의 라우팅 로직이 YAML 설정 파일에 정의된 경로(route)에 따라 목적지 시스템 핸들을 결정하고 해당 메시지를 전달합니다. 예를 들어, `Middleware_1`에서 `Middleware_2`로의 경로가 정의된 경우, 코어는 내부적으로 `Middleware_1`을 위한 `TopicSubscriber`와 `Middleware_2`를 위한 `TopicPublisher`를 등록하여 데이터 흐름을 관리합니다.11 이러한 구조 덕분에 코어는 특정 프로토콜에 종속되지 않고 범용적인 메시지 중개자 역할을 수행할 수 있습니다.

### 2.2  시스템 핸들: 모듈식 플러그인 기반 프로토콜 변환 접근법

**시스템 핸들(System Handle, SH)**은 특정 미들웨어나 프로토콜과 Integration Service 코어 사이를 연결하는 프로토콜별 플러그인입니다.3 각 시스템 핸들의 핵심 역할은 대상 프로토콜의 고유한 데이터 타입과 통신 방식을 코어가 이해할 수 있는 공통 표현 언어인 xTypes로 변환하고, 그 반대의 변환도 수행하는 것입니다.3

시스템 핸들은 사용자 애플리케이션의 통신 엔티티(발행자, 구독자, 서버, 클라이언트)에 대한 '미러(mirror)' 프록시를 생성하는 방식으로 작동합니다.6 예를 들어, 사용자 애플리케이션에 발행자(publisher)가 있다면, 시스템 핸들은 해당 발행자의 메시지를 수신하기 위한 구독자(subscriber)를 생성합니다. 반대로 사용자 애플리케이션에 구독자가 있다면, 시스템 핸들은 해당 구독자에게 메시지를 전달하기 위한 발행자를 생성합니다. 서비스 클라이언트와 서버에 대해서도 동일한 원리가 적용됩니다.6

이 플러그인 기반 프레임워크는 시스템 확장을 매우 용이하게 만듭니다. 새로운 프로토콜을 통합해야 할 경우, 개발자는 해당 프로토콜에 대한 시스템 핸들만 새로 작성하면 됩니다. 일단 시스템 핸들이 개발되면, 해당 프로토콜은 이미 지원되는 다른 모든 프로토콜과 즉시 상호 운용성을 갖게 됩니다.1 이는 개발자가 특정 프로토콜을 xTypes로 변환하는 방법에만 집중하면 되도록 하여, 새로운 기술을 통합하는 데 필요한 진입 장벽을 극적으로 낮춥니다.

### 2.3  공통 언어: eProsima xTypes와 동적 타입 표현의 이해

Integration Service 코어와 모든 시스템 핸들 간의 원활한 통신을 가능하게 하는 기반 기술은 **eProsima xTypes**라는 공통 언어입니다.1 eProsima xTypes는 Object Management Group(OMG)의 **DDS-XTypes (Extensible and Dynamic Topic Types for DDS)** 표준을 C++17 기반의 헤더 전용 라이브러리로 구현한 것으로, 빠르고 가벼운 것이 특징입니다.6

xTypes의 가장 중요한 특징은 **런타임에 동적 타입(Dynamic Type) 표현을 생성**할 수 있는 능력입니다.11 이는 사용자가 YAML 설정 파일에 인터페이스 정의 언어(Interface Definition Language, IDL) 형식으로 데이터 타입을 정의하면, xTypes 파서가 이를 런타임에 해석하여 동적인 데이터 구조를 만들어내는 방식으로 이루어집니다.10

이 동적 타입 기능은 Integration Service 유연성의 핵심입니다. 컴파일 시점에 알지 못하는 임의의 사용자 정의 데이터 구조를 처리할 수 있게 해주기 때문입니다. 정적 타입 시스템을 사용했다면 새로운 메시지 타입이 추가될 때마다 전체 서비스를 다시 컴파일해야 했을 것입니다. 하지만 동적 타입을 사용함으로써, 사용자는 단순히 YAML 설정 파일만 수정하여 새로운 데이터 타입을 시스템에 알리고, 재컴파일 없이 즉시 사용할 수 있습니다. 또한, 시스템은 xTypes 라이브러리의 `TypeConsistency` 정의를 사용하여 서로 다른 시스템 엔드포인트 간의 타입 호환성을 검사할 수 있습니다.14

### 2.4  "Mix" 파일의 역할: 타입 변환 및 동적 라이브러리 관리

아키텍처의 또 다른 중요한 구성 요소는 "Mix" 파일입니다. Mix는 `MiddlewareInterfaceExtension`의 약어로, 특정 시스템 핸들이 필요로 하는 동적 라이브러리(Linux의 `.so`, Windows의 `.dll`) 목록을 담고 있는 파일입니다.4

이 동적 라이브러리에는 미들웨어의 고유 타입(예: ROS 2의 `.msg` 파일)을 공통 언어인 xTypes 표현으로 변환하거나 그 반대의 변환을 수행하는 구체적인 로직이 포함되어 있습니다.4 즉, 타입 변환의 실제 구현은 이 라이브러리 안에 존재합니다.

Mix 파일은 일반적으로 빌드 과정에서 `is_mix_generator`라는 CMake 함수에 의해 자동으로 생성되며, Integration Service가 시작될 때 런타임에 로드됩니다.4 이 메커니즘을 통해 시스템은 필요한 타입 변환 기능을 동적으로 로드하여 사용할 수 있습니다.

| 표 1: 아키텍처 구성 요소 개요 |
| ----------------------------- |
| **구성 요소**                 |
| Integration Service 코어      |
| 시스템 핸들 (SH)              |
| eProsima xTypes               |
| Mix 파일                      |

## 3.  구성 및 배포를 위한 실용 가이드

eProsima Integration Service의 가장 큰 장점 중 하나는 코드 수정이나 재컴파일 없이 텍스트 기반 설정 파일 하나로 전체 시스템의 통합 동작을 정의할 수 있다는 점입니다. 이 섹션에서는 Integration Service 인스턴스를 구성, 설치 및 실행하는 방법에 대한 구체적이고 실행 가능한 지침을 제공합니다.

### 3.1  YAML 설정 파일: 구조와 문법

Integration Service 인스턴스의 모든 동작은 런타임에 로드되는 단일 YAML 파일에 의해 제어됩니다.3 이 접근 방식은 설정과 코드를 명확하게 분리하여, 대규모 시스템에서 다양한 통합 시나리오를 재컴파일 없이 유연하게 적용할 수 있게 해주는 강력한 이점을 가집니다.3

YAML 파일은 여러 개의 필수 및 선택적 최상위 섹션으로 구성됩니다.3 각 섹션은 통합 시스템의 특정 측면을 정의하는 역할을 합니다.

### 3.2  시스템, 타입, 경로 정의

YAML 설정의 핵심은 `systems`, `types`, `routes` 섹션을 통해 통합의 기본 골격을 정의하는 것입니다.

- **`types` (선택 사항)**: 이 섹션에서는 사용자가 IDL(Interface Definition Language) 문법을 사용하여 커스텀 메시지 타입을 YAML 파일 내에 직접 정의할 수 있습니다. 여기에 정의된 내용은 런타임에 파싱되어 동적 타입으로 생성됩니다.10

- **`systems` (필수 사항)**: 통신에 참여하는 각 미들웨어를 나열하고 설정하는 필수 섹션입니다. 각 시스템에는 고유한 별칭(예: `ros2_system`, `dds_system`)이 부여되며, 해당 시스템의 `type`(예: `ros2`, `fastdds`, `websocket_server`)을 명시해야 합니다.6 또한, 

  `types-from` 옵션을 사용하여 한 시스템의 타입 정의를 다른 시스템에서 상속받아 타입 재정의를 피할 수 있습니다.6

- **`routes` (필수 사항)**: 시스템 간의 통신 다리, 즉 경로를 정의하는 필수 섹션입니다. 각 경로는 발행/구독(publish/subscribe) 패턴의 경우 `from`과 `to` 시스템을, 서비스 요청/응답(server/client) 패턴의 경우 `server`와 `clients`를 지정하여 데이터 흐름의 방향을 정의합니다.14

### 3.3  통신을 위한 토픽 및 서비스 지정

기본 골격이 정의되면, `topics`와 `services` 섹션을 통해 실제 데이터를 교환할 대상을 구체적으로 명시해야 합니다.

- **`topics`**: 브리징할 특정 토픽을 정의합니다. 각 토픽 항목은 반드시 데이터의 `type`과 사용할 `route`를 지정해야 합니다. 여기서 `type`은 `types` 섹션에 정의되었거나 Mix 파일을 통해 로드된 타입이어야 합니다.6
- **`services`**: 브리징할 특정 서비스를 정의합니다. 토픽과 유사하게 `type`과 `route`를 지정합니다.14
- **`remap` (선택 사항이지만 강력함)**: `topics` 또는 `services` 정의 내에서 사용할 수 있는 매우 강력한 기능입니다. `remap`을 사용하면 경로에 포함된 각 시스템에 대해 서로 다른 토픽/서비스 이름이나 심지어 다른 타입 이름(단, 타입 구조는 호환 가능해야 함)을 지정할 수 있습니다. 이 기능은 애초에 함께 작동하도록 설계되지 않은 이기종 시스템을 통합할 때 매우 중요합니다.6

이처럼 YAML 설정 파일은 단순한 설정 모음이 아니라, 복잡한 다중 프로토콜 시스템의 전체 데이터 흐름과 변환 로직을 하나의 인간이 읽을 수 있는 문서로 선언하는 '선언적 통합 언어'의 역할을 합니다. 이는 시스템 문서화, 버전 관리(YAML 파일을 Git으로 관리), 런타임 유연성 측면에서 막대한 이점을 제공합니다.

### 3.4  설치, 의존성 및 `colcon` 빌드 프로세스

Integration Service를 사용하기 위해서는 먼저 필요한 의존성을 설치하고 소스 코드를 빌드해야 합니다.

- **코어 의존성**: 코어 엔진은 `yaml-cpp` (YAML 파서)와 `boost-program-options` (설정 파일 옵션 처리) 라이브러리를 필요로 합니다.6 eProsima xTypes 역시 필수 의존성이지만, 일반적으로 Git 서브모듈로 포함되어 있어 별도 설치 없이 함께 클론됩니다.15
- **시스템 핸들 의존성**: 각 시스템 핸들은 고유한 의존성을 가집니다. 예를 들어:
  - `FastDDS-SH`는 eProsima Fast DDS 라이브러리가 필요합니다.6
  - `ROS2-SH`는 ROS 2 배포판(예: Foxy, Galactic)이 필요합니다.6
  - `WebSocket-SH`는 OpenSSL과 WebSocket++ 라이브러리가 필요합니다.6
- **빌드 프로세스**: 빌드를 위해 권장되는 도구는 ROS 생태계의 표준 빌드 도구인 `colcon`입니다.6 전체 프로세스는 다음과 같습니다:
  1. `colcon` 작업 공간(workspace)을 생성합니다.
  2. `Integration-Service` 리포지토리를 `--recursive` 플래그와 함께 클론하여 서브모듈(xTypes)까지 다운로드합니다.
  3. 사용하고자 하는 시스템 핸들의 리포지토리를 작업 공간의 `src` 디렉터리에 클론합니다.
  4. 작업 공간의 루트에서 `colcon build` 명령을 실행하여 전체 프로젝트를 빌드합니다.17

Integration Service가 독립적인 제품임에도 불구하고 `colcon`을 권장 빌드 도구로 채택한 것은 이 프로젝트가 ROS 및 로보틱스 생태계와 깊이 연계되어 있음을 보여줍니다. 이는 주된 대상 사용자인 로봇 공학자들의 진입 장벽을 낮추는 효과가 있지만, ROS 환경에 익숙하지 않은 개발자에게는 약간의 학습 곡선이 존재할 수 있습니다.

## 4.  내장 시스템 핸들 심층 분석

eProsima Integration Service의 실제적인 힘은 다양한 프로토콜을 지원하는 시스템 핸들(System Handle)에서 나옵니다. 각 시스템 핸들은 특정 미들웨어와의 통신을 전담하는 플러그인 역할을 합니다. 이 섹션에서는 공식적으로 지원되는 각 내장 시스템 핸들의 목적, 주요 설정 옵션, 그리고 활용 사례를 상세히 분석합니다. 이 시스템 핸들들의 조합은 로보틱스와 산업용 사물 인터넷(IIoT)의 교차점에서 발생하는 복잡한 통합 요구사항을 해결하기 위한 전략적 선택을 반영합니다.

### 4.1  Fast DDS 시스템 핸들 (`fastdds`)

- **목적**: 고성능 DDS(Data Distribution Service) 구현체인 eProsima Fast DDS를 사용하는 시스템을 통합합니다.13 DDS는 ROS 2의 기본 미들웨어이며 산업, 자동차, 국방 등 미션 크리티컬 시스템에서 널리 사용됩니다.12
- **주요 사용 사례**:
  - DDS 애플리케이션과 비-DDS 시스템(예: ROS, WebSocket) 간의 브리징.13
  - 서로 다른 `domain_id`에서 실행되는 두 DDS 애플리케이션 연결. 이는 대규모 시스템에서 네트워크를 논리적으로 분리하는 데 사용됩니다.13
  - WAN 통신을 위한 TCP 터널 생성.13
- **설정**: YAML 설정 파일에서 `participant.domain_id`를 통해 DDS 도메인을 지정할 수 있으며, `participant.file_path`와 `profile_name`을 사용하여 XML 기반의 커스텀 QoS(Quality of Service) 프로파일을 적용할 수 있습니다.13
- **예제**: Fast DDS 발행자의 데이터를 ROS 2 구독자로 전송하는 `DDS-ROS 2 브리지` 예제가 제공됩니다.13

### 4.2  ROS 1 & ROS 2 시스템 핸들 (`ros1`, `ros2`)

- **목적**: 로봇 운영체제(Robot Operating System) 기반 애플리케이션을 통합합니다. ROS는 로보틱스 연구 및 산업 분야의 사실상 표준입니다.3
- **주요 사용 사례**:
  - 기존의 ROS 1 시스템과 최신 ROS 2 시스템 간의 데이터 및 서비스를 변환하는 강력한 `ROS 1-ROS 2 브리지` 생성.20
  - 서로 다른 `domain_id`를 사용하는 ROS 2 노드들 간의 토픽 브리징. 이는 복잡한 로봇 시스템에서 흔히 요구되는 기능입니다.5
  - ROS 시스템을 WebSocket 시스템 핸들을 통해 웹 인터페이스에 연결하거나, DDS 시스템 핸들을 통해 다른 백엔드 시스템에 연결.
- **설정**: `ros2` 시스템 타입은 YAML 파일에서 `domain_id` 매개변수를 받을 수 있습니다.14
- **의존성**: 해당 ROS 배포판 환경이 활성화되어 있어야 합니다(예: ROS 1의 경우 Melodic/Noetic, ROS 2의 경우 Foxy/Galactic).6

### 4.3  WebSocket 시스템 핸들 (`websocket_server`, `websocket_client`)

- **목적**: WebSocket 프로토콜을 사용하는 웹 기반 애플리케이션이나 다른 원격 서비스와의 양방향 통신을 가능하게 합니다.15

- **주요 사용 사례**:

  - ROS 또는 DDS 시스템의 상태를 실시간으로 모니터링하는 웹 기반 대시보드 구축.
  - 원격 웹 클라이언트가 로봇 시스템에 명령(예: 서비스 호출)을 전송하도록 허용.22

- **설정**: 보안 통신을 위한 `TLS`와 비보안 통신을 위한 `TCP` 엔드포인트를 모두 지원합니다. YAML에서 `port`, `security`, 인증서 경로 등을 설정할 수 있습니다.22 WebSocket을 통한 통신 프로토콜은 JSON 기반이며, `publish`, `subscribe`, `call_service`와 같은 동작을 위한 특정 `op` 코드를 사용합니다.22

### 4.4  FIWARE 시스템 핸들 (`fiware`)

- **목적**: 스마트 시티 및 IoT 애플리케이션에서 널리 사용되는 오픈소스 플랫폼인 FIWARE, 특히 Orion Context Broker와의 통합을 지원합니다.1
- **주요 사용 사례**: 로봇 시스템(ROS/DDS)에서 생성된 데이터를 FIWARE가 관리하는 더 큰 규모의 IoT 생태계로 전송하거나, 반대로 FIWARE 시스템이 로봇에게 작업을 지시하는 시나리오에 사용됩니다.2
- **의존성**: `Asio`, `cURLpp`, `cURL` 라이브러리가 필요합니다.6

| 표 2: 내장 시스템 핸들 및 주요 세부 정보 |
| ---------------------------------------- |
| **시스템 핸들 (`type` 이름)**            |
| `fastdds`                                |
| `ros1`                                   |
| `ros2`                                   |
| `websocket_server` / `websocket_client`  |
| `fiware`                                 |

## 5.  주요 적용 시나리오 및 산업 활용 사례

eProsima Integration Service의 진정한 가치는 이론적인 아키텍처를 넘어 실제 산업 현장의 복잡한 문제들을 해결하는 능력에 있습니다. 이 도구는 세 가지 대표적인 적용 시나리오를 통해 그 실용성을 입증하며, 이 시나리오들은 상호 배타적이지 않고 조합하여 더욱 정교한 시스템 아키텍처를 구축하는 빌딩 블록 역할을 합니다.

### 5.1  시나리오 1: 이기종 시스템 통합 (Heterogeneous System Integration)

이는 Integration Service의 가장 일반적이고 기본적인 사용 사례로, 서로 다른 프로토콜을 사용하며 호환되지 않는 타입, 토픽, 서비스를 처리하는 시스템 간의 통신을 가능하게 합니다.3 포인트-투-포인트 브리지의 한계를 극복하고 중앙 집중식 허브를 통해 원활한 데이터 교환을 구현합니다.

- **예제: `ROS 1 - ROS 2 브리지`**: 많은 로보틱스 프로젝트는 기존의 방대한 ROS 1 자산을 활용하면서 최신 ROS 2의 기능(실시간성, 보안 등)을 도입하고자 합니다. Integration Service는 레거시 ROS 1 시스템과 최신 ROS 2 시스템 간의 토픽과 서비스를 변환하는 안정적인 브리지를 구성하여 점진적인 마이그레이션을 지원합니다.20
- **예제: `FIWARE - ROS 2 브리지`**: 스마트 팩토리나 스마트 시티 환경에서는 FIWARE 기반의 IoT 플랫폼이 로봇과 상호작용해야 하는 경우가 많습니다. Integration Service는 로봇의 센서 데이터를 FIWARE 컨텍스트 브로커로 전송하거나, IoT 플랫폼의 명령을 ROS 2 로봇이 수행하도록 연결하는 역할을 할 수 있습니다.2
- **예제: `DDS - ROS 2 브리지` (커스텀 QoS 포함)**: 이 예제는 단순한 데이터 전달을 넘어, 두 미들웨어 간의 QoS(Quality of Service) 정책을 존중하고 매핑하는 고급 기능을 보여줍니다. 예를 들어, DDS의 `RELIABLE` QoS를 ROS 2의 신뢰성 있는 통신에 매핑하여 데이터 전달의 신뢰성을 보장할 수 있습니다.20

### 5.2  시나리오 2: 동종 시스템 브리징 (Homogeneous System Bridging)

이 시나리오는 동일한 프로토콜을 사용하는 시스템들이지만, 해당 프로토콜의 특정 기능(예: 도메인 ID, 파티션)에 의해 논리적으로 격리된 경우, 이들 간의 통신을 중개하는 데 사용됩니다.3 이는 시스템의 모듈성과 보안을 강화하는 데 중요한 역할을 합니다.

- **예제: `ROS 2 도메인 ID 변경`**: ROS 2는 DDS의 도메인 ID를 사용하여 논리적으로 분리된 네트워크를 생성합니다. 예를 들어, 자율주행 차량에서 인식 시스템(perception system)과 임무 계획 시스템(mission planning system)을 서로 다른 도메인 ID로 분리하여 상호 간섭을 막을 수 있습니다. 이때 Integration Service는 두 도메인 간의 게이트웨이 역할을 하여, `차량 위치`와 같은 특정 토픽만 선택적으로 브리징하고 나머지 트래픽은 격리된 상태로 유지할 수 있습니다.21
- **예제: `DDS 도메인 ID 변경`**: 순수 DDS 시스템에서도 동일한 개념이 적용됩니다. Integration Service를 사용하면 서로 다른 도메인에 속한 DDS 애플리케이션 간에 제어된 데이터 교환을 구현할 수 있습니다.13

### 5.3  시나리오 3: WAN 및 인터넷 통신 (WAN and Internet Communication)

이 시나리오는 지리적으로 다른 지역에 위치하거나 논리적으로 분리된 광역 통신망(WAN)에 호스팅된 시스템 간의 통신을 가능하게 합니다.3 이는 원격 제어, 클라우드 기반 로봇 관리, 분산 데이터 수집 등 현대적인 애플리케이션에 필수적입니다.

- **예제: `DDS를 통한 WAN-TCP 터널링`**: 일반적으로 DDS에서 사용하는 UDP 프로토콜은 인터넷과 같은 WAN 환경에서 방화벽이나 NAT 문제로 인해 불안정할 수 있습니다. Integration Service는 이러한 문제를 해결하기 위해 WAN의 양단에 각각 인스턴스를 실행하고, 이들 간의 통신을 TCP 기반 전송으로 설정하여 안정적인 터널을 생성할 수 있습니다. 이를 통해 DDS 트래픽을 인터넷을 통해 안정적으로 터널링할 수 있습니다.13 이 기능은 ROSIN 프로젝트 제안서에서도 핵심 기능으로 강조되었습니다.8

이러한 시나리오들의 조합을 통해 Integration Service는 로보틱스, IoT, 자동차, 그리고 기타 미션 크리티컬 애플리케이션에 매우 적합한 도구임이 입증되었습니다.12 eProsima가 유럽남방천문대(ESO), 르네사스(Renesas), Husarion, RRAI 등 다양한 산업 분야의 고객 및 파트너와 협력하고 있다는 사실은 Integration Service를 포함한 자사 기술 스택의 산업적 중요성을 뒷받침합니다.24

## 6.  성능 및 보안 분석

프로덕션 환경에 시스템을 배포할 때, 기능적 요구사항만큼이나 성능 및 보안과 같은 비기능적 특성이 중요합니다. eProsima Integration Service는 다른 시스템들을 연결하는 중개자 역할을 하므로, 이 서비스가 추가하는 오버헤드와 보안 모델을 이해하는 것은 아키텍트에게 필수적입니다.

### 6.1  성능 프로파일: 지연 시간 및 처리량 분석

Integration Service 자체의 성능은 단일 수치로 정의되기보다는, 연결되는 전체 데이터 경로의 함수로 이해해야 합니다. 메시지가 거치는 전체 과정, 즉 `소스 앱 -->> 소스 미들웨어 -->> 소스 SH -->> 코어 -->> 목적지 SH -->> 목적지 미들웨어 -->> 목적지 앱`의 각 단계에서 발생하는 지연 시간의 총합이 최종 성능을 결정합니다.

- **직접적인 벤치마크의 부재**: 현재 제공된 자료에는 Integration Service 자체에 대한 직접적인 성능 벤치마크(지연 시간, 처리량) 데이터가 포함되어 있지 않습니다.11
- **추론된 성능**: 성능은 근본적으로 사용되는 시스템 핸들과 xTypes 변환 프로세스의 효율성에 의해 결정됩니다. Integration Service 코어 자체는 "빠르고 가벼운" 엔진으로 설계되었으므로 3, 주된 오버헤드는 시스템 핸들 내에서 발생하는 데이터 직렬화/역직렬화 및 프로토콜별 처리 과정에서 발생합니다. 따라서 브리지의 전체 성능은 연결된 두 미들웨어 중 성능이 더 낮은 쪽의 성능에 데이터 변환 오버헤드가 더해진 값에 가깝게 됩니다.
- **Fast DDS를 통한 성능 예측**: DDS가 포함된 브리지의 성능은 eProsima Fast DDS의 성능을 분석함으로써 예측할 수 있습니다. Fast DDS는 CycloneDDS, OpenDDS와 같은 대안들과 비교했을 때, 특히 큰 데이터 크기나 제로 카피(zero-copy) 데이터 공유와 같은 고급 기능을 사용할 때 지연 시간과 처리량 측면에서 우수한 성능을 보인다는 광범위한 벤치마크 결과가 존재합니다.27
- **지연 시간 요인**: Integration Service를 통과하는 메시지의 종단 간 지연 시간은 (1) 소스 시스템 핸들의 수신 및 xTypes 변환 시간, (2) 코어의 라우팅 시간, (3) 목적지 시스템 핸들의 xTypes 역변환 및 전송 시간의 합입니다. Fast DDS 벤치마크는 전송 방식(UDP vs. 공유 메모리), 데이터 크기, 동기/비동기 발행 모드와 같은 요소들이 지연 시간을 결정하는 주요 변수임을 보여줍니다.27 따라서 아키텍트는 "Integration Service의 지연 시간은 얼마인가?"라는 질문 대신, "4KB 메시지에 대한 ROS 2와 DDS 간 브리지의 지연 시간은 얼마인가?"와 같이 구체적인 경로에 대해 질문해야 합니다.

### 6.2  보안 프레임워크: 시스템 핸들을 통한 상속된 보안

Integration Service 코어 자체는 독립적인 보안 계층을 구현하지 않습니다. 대신, 보안은 개별 시스템 핸들의 기능에 의해 제공되는 '상속된 보안 모델'을 따릅니다.30 이는 강력한 유연성을 제공하지만, 전체 시스템의 보안을 보장해야 하는 아키텍트에게 중요한 책임을 부여합니다.

- **DDS 보안**: `FastDDS-SH`는 Fast DDS가 제공하는 포괄적인 OMG 표준 DDS 보안 플러그인을 활용할 수 있습니다. 이는 업계 최고 수준의 보안 기능을 제공합니다.
  - **인증 (`DDS:Auth:PKI-DH`)**: 신뢰할 수 있는 인증 기관(CA)을 사용하여 DDS 도메인에 참여하는 각 `DomainParticipant`를 인증합니다.31
  - **접근 제어 (`DDS:Access:Permissions`)**: 서명된 거버넌스 및 권한 파일을 통해 각 참여자가 어떤 토픽을 읽고 쓸 수 있는지에 대한 세분화된 권한을 정의합니다.31
  - **암호화 (`DDS:Crypto:AES-GCM-GMAC`)**: 메시지 페이로드에 대한 인증된 암호화를 제공하여 데이터의 기밀성과 무결성을 보장합니다.31
- **WebSocket 보안**: `WebSocket-SH`는 YAML 설정 파일에서 구성할 수 있는 TLS(Transport Layer Security)를 통해 보안 통신을 지원합니다.22

이러한 상속된 보안 모델은 강력하지만, 아키텍트가 종단 간 보안을 신중하게 설계해야 함을 의미합니다. 예를 들어, DDS 보안 기능으로 완벽하게 보호되는 시스템의 특정 토픽을 Integration Service를 통해 보안이 설정되지 않은(`security: none` 옵션 사용 가능 22) WebSocket 서버로 브리징할 수 있습니다. 이 경우, 해당 데이터 경로에 대한 DDS 보안은 사실상 무력화됩니다. Integration Service는 연결성을 제공하는 도구일 뿐, 전역적인 보안 정책을 강제하지 않습니다. 따라서 아키텍트는 허브에 연결된 각 '스포크'의 보안을 개별적으로 구성하고, 전체 시스템-오브-시스템즈의 보안을 종합적으로 분석해야 합니다.

## 7.  비교 분석: 미들웨어 생태계 속 Integration Service의 위치

eProsima Integration Service의 가치를 명확히 이해하기 위해서는 다른 일반적인 통합 기술과의 비교를 통해 그 고유한 위치를 파악하는 것이 중요합니다. 이 서비스는 특정 프로토콜도, 저수준 라이브러리도 아닌, 이들을 아우르는 상위 계층의 솔루션입니다. 아키텍트는 이 비교를 통해 자신의 문제에 가장 적합한 도구를 정보에 입각하여 선택할 수 있습니다.

### 7.1  Integration Service vs. 메시지 브로커 (예: MQTT)

- **MQTT (Message Queuing Telemetry Transport)**: 제한된 성능의 장치와 신뢰성 없는 네트워크를 위해 설계된 경량의 브로커 기반 발행-구독 프로토콜입니다.32 OASIS 표준이며, 주로 간단한 원격 측정 데이터(telemetry)를 다수의 장치에서 중앙 지점으로 전송하는 데 이상적입니다.
- **비교**:
  - **아키텍처**: 둘 다 중앙 허브(브로커) 기반이지만, MQTT는 그 자체가 프로토콜인 반면, Integration Service는 **다중 프로토콜 게이트웨이**입니다. 즉, Integration Service는 내부에 MQTT 시스템 핸들을 포함하여 MQTT를 다른 시스템(예: DDS, ROS)과 연결하는 역할을 수행할 수 있습니다 (실제로 ROSIN 프로젝트에서 구상됨 8).
  - **데이터 타이핑**: MQTT는 페이로드를 바이트 덩어리로 취급하여 데이터 타입에 거의 관여하지 않습니다. 반면, Integration Service는 xTypes/IDL을 통해 강력한 타입 시스템을 제공합니다. 이는 미들웨어 계층에서 데이터 거버넌스와 유효성 검사를 가능하게 하여 시스템의 안정성을 높입니다.
  - **사용 사례**: MQTT는 간단한 센서 데이터를 전달하는 데 최적화되어 있고, Integration Service는 데이터 구조와 타입 안전성이 중요한 DDS, ROS와 같은 풍부한 데이터 시스템 간의 복잡한 통합을 위해 설계되었습니다.

### 7.2  Integration Service vs. 메시징 라이브러리 (예: ZeroMQ)

- **ZeroMQ**: 고성능 비동기 메시징 라이브러리로, 브로커가 없는 것이 특징입니다. 다양한 전송 계층 위에서 원자적 메시지를 전달하는 소켓을 제공하며, 맞춤형 프로토콜을 구축하기 위한 '부품 키트'에 가깝습니다.
- **비교**:
  - **추상화 수준**: ZeroMQ는 저수준 라이브러리입니다. 개발자는 메시징 패턴, 데이터 직렬화, 서비스 탐색(discovery) 등 모든 것을 직접 구현해야 합니다. 반면, Integration Service는 이러한 기능들을 즉시 사용 가능한 완전한 애플리케이션 형태로 제공하는 고수준 솔루션입니다.
  - **서비스 탐색 및 타이핑**: Integration Service는 시스템 핸들(예: DDS)로부터 상속받은 내장 서비스 탐색 기능과 풍부한 동적 타입 시스템(xTypes)을 갖추고 있습니다. ZeroMQ에는 이러한 기능이 내장되어 있지 않습니다.
  - **사용 사례**: ZeroMQ는 최고의 성능과 제어 능력을 바탕으로 완전히 새로운 통신 프로토콜을 구축해야 하는 개발자에게 적합합니다. Integration Service는 기존의 표준 기반 시스템들을 최소한의 코드로 연결해야 하는 시스템 통합 전문가에게 적합합니다.

### 7.3  Integration Service vs. 다른 데이터 중심 미들웨어

- **비교**: 많은 DDS 벤더들은 도메인을 브리징하거나 WAN을 통해 연결하기 위한 라우팅 서비스를 제공합니다. 이들과 Integration Service를 비교할 때 핵심적인 차별점은 바로 **이기종 프로토콜 지원**입니다.
- **핵심 차별점**: 다른 DDS 라우터들이 DDS 트래픽을 라우팅하는 데 탁월한 성능을 보이는 반면, 이들은 범용 프로토콜 게이트웨이로 설계되지 않았습니다. Integration Service의 핵심 사명은 DDS, ROS, WebSocket 등 근본적으로 다른 시스템들 간의 이질성(heterogeneity)을 해결하는 것입니다.3 개방적인 플러그인 아키텍처와 ROS에 대한 최고 수준의 지원은 다른 솔루션과 구별되는 가장 큰 특징입니다.

결론적으로, eProsima Integration Service는 독특한 생태학적 지위를 차지합니다. 이는 프로토콜도, 저수준 라이브러리도, 단순한 라우터도 아닌, **"메타-미들웨어(meta-middleware)"** 또는 **"시스템-오브-시스템즈 통합 플랫폼(system-of-systems integration platform)"**으로 정의할 수 있습니다. 기존 미들웨어들 *위에서* 동작하며 이들이 서로 상호 운용되도록 만드는 것이 그 목적이기 때문입니다. 아키텍트가 "풍부한 데이터 타입을 가진, 여러 개의 기존 이기종 시스템을 연결해야 하는가?"라는 질문에 '예'라고 답한다면, Integration Service는 매우 강력한 후보가 될 것입니다.

| 표 3: 통합 기술 비교 분석 |
| ------------------------- |
| **특징**                  |
| **주요 추상화**           |
| **아키텍처**              |
| **데이터 타이핑**         |
| **서비스 탐색**           |
| **주요 사용 사례**        |
| **사용 용이성**           |

## 8.  전략적 전망 및 권장 사항

eProsima Integration Service는 복잡한 분산 시스템 통합이라는 현대적 과제에 대한 강력하고 성숙한 솔루션입니다. 이 마지막 섹션에서는 프로젝트의 현재 상태, 이상적인 채택 시나리오, 그리고 미래 잠재력을 종합적으로 평가하여 아키텍트와 기술 관리자를 위한 전략적 지침을 제공합니다.

### 8.1  현재 상태, 유지보수 및 커뮤니티

프로젝트의 건전성과 지속 가능성은 산업계에서 기술을 채택할 때 가장 중요한 고려사항 중 하나입니다. Integration Service는 이 측면에서 매우 긍정적인 신호를 보냅니다.

- **전문 기업의 유지보수**: 이 프로젝트는 분산 시스템 미들웨어 전문 기업인 eProsima에 의해 적극적으로 개발 및 유지보수되고 있습니다.6 이는 프로젝트가 방치되지 않고 지속적으로 개선될 것임을 보장합니다.
- **개방성과 상업적 지원**: Integration Service는 허용적인 아파치 2.0(Apache 2.0) 라이선스 하에 오픈소스로 제공됩니다.12 이는 개발자와 연구자들이 초기 비용 부담 없이 기술을 도입하고 실험할 수 있게 합니다. 동시에, eProsima는 산업 사용자들이 요구하는 보증과 전문가 지원을 위해 상업적 지원, 아키텍처 컨설팅, 기능 개발 가속화 등의 유료 서비스를 제공합니다.6 이러한 이중 모델은 넓은 사용자 기반을 확보하는 동시에 프로젝트의 장기적인 재정적 지속 가능성을 담보하는 검증된 전략입니다.
- **풍부한 문서와 활발한 개발**: 사용자 매뉴얼, API 레퍼런스, 수많은 예제를 포함한 포괄적인 문서가 제공되어 학습 곡선을 완화합니다.16 또한, 릴리스 노트를 통해 지속적인 기능 개선과 버그 수정이 이루어지고 있음을 확인할 수 있습니다.5

### 8.2  채택 시나리오: 언제 Integration Service를 선택해야 하는가

Integration Service는 모든 문제에 대한 만병통치약이 아니며, 그 가치가 가장 빛을 발하는 특정 시나리오가 존재합니다.

- **이상적인 시나리오**:
  - **ROS 기반 시스템 통합**: 로보틱스 시스템(ROS 1 또는 ROS 2)을 다른 산업용(DDS) 또는 엔터프라이즈 시스템(FIWARE, 웹 서비스)과 통합해야 할 때 가장 강력한 선택지 중 하나입니다.
  - **점진적 마이그레이션**: 기존의 ROS 1 시스템 자산을 유지하면서 ROS 2로 점진적으로 전환하는 프로젝트에 이상적인 브리징 솔루션을 제공합니다.
  - **진화하는 시스템 아키텍처**: 장기적으로 새로운 기술(예: 새로운 센서, AI 프레임워크, 클라우드 서비스)이 추가될 것으로 예상되는 복잡한 로봇 또는 IoT 시스템에 적합합니다. 플러그인 아키텍처 덕분에 미래의 통합 요구에 유연하게 대응할 수 있습니다.
  - **지리적으로 분산된 시스템**: 인터넷을 통해 안정적으로 통신해야 하는 원격 제어 또는 분산 시스템에 검증된 WAN 터널링 기능을 제공합니다.
- **적합하지 않은 시나리오**:
  - 모든 구성 요소가 이미 동일한 미들웨어를 사용하고 있으며 외부 프로토콜과 통합할 계획이 없는 동종 시스템.
  - 단순한 포인트-투-포인트 통신만 필요하며, 간단한 사용자 정의 스크립트로 해결할 수 있는 경우.
  - 서비스 자체의 오버헤드가 문제가 될 수 있는 극도로 제한된 리소스 환경 (단, Fast DDS와 같은 의존성은 성능에 최적화되어 있습니다).

### 8.3  미래 궤도 및 새로운 시스템 핸들의 잠재력

Integration Service의 미래는 그 확장 가능한 플러그인 아키텍처에 달려 있습니다.1 프레임워크는 이미 새로운 프로토콜을 지원할 준비가 되어 있습니다.

- **잠재적 시스템 핸들**: 원래 ROSIN 프로젝트에서 구상했던 것처럼 MQTT나 ZeroMQ에 대한 공식 시스템 핸들이 추가될 수 있습니다.8 또한, 산업 자동화 분야의 주요 프로토콜인 OPC UA나, 데이터베이스(DB)와의 직접적인 연결을 위한 커넥터도 유용한 확장이 될 수 있습니다.
- **AI 및 로보틱스 연계**: eProsima가 AI와 로보틱스에 중점을 두고 있다는 점은 18, 향후 개발이 AI/ML 프레임워크 및 데이터 파이프라인을 로봇 시스템과 더욱 원활하게 통합해야 하는 필요성에 의해 주도될 수 있음을 시사합니다.

### 8.4  최종 평가

eProsima Integration Service는 현대 분산 시스템에서 발생하는 복잡한 통합 문제를 해결하기 위해 전략적으로 설계된 성숙하고 강력한 프레임워크입니다. 표준화되고 동적인 xTypes 공통 언어를 기반으로 하는 중앙 집중식 플러그인 아키텍처는, 깨지기 쉬운 포인트-투-포인트 브리지 방식에 대한 확장 가능하고 유지보수가 용이한 대안을 제공합니다.

특히 로보틱스와 산업용 IoT(IIoT) 영역에서 그 가치가 두드러지며, 차세대 복합 시스템을 구축하는 아키텍트에게 신뢰할 수 있는 솔루션을 제공합니다. 이 도구는 단순히 시스템을 연결하는 것을 넘어, 시스템이 진화하고 성장할 수 있는 견고한 기반을 마련해 줍니다.

#### **참고 자료**

1. Integration Service - allowing integration of any protocol into DDS - ROS Discourse, accessed July 5, 2025, https://discourse.ros.org/t/integration-service-allowing-integration-of-any-protocol-into-dds/16135
2. Integration Service - Integrate a complex system into DDS - General - ROS Discourse, accessed July 5, 2025, https://discourse.ros.org/t/integration-service-integrate-a-complex-system-into-dds/16250
3. Integration Service Core - Integration Service 3.1.0 documentation - eProsima, accessed July 5, 2025, https://integration-service.docs.eprosima.com/
4. 1.1.5. Mix - Integration Service 3.1.0 documentation, accessed July 5, 2025, https://integration-service.docs.eprosima.com/en/latest/api_reference/core/core/runtime/middlewareinterfaceextension.html
5. eProsima is launching Integration Service v3.0.0 - ROS Discourse, accessed July 5, 2025, https://discourse.ros.org/t/eprosima-is-launching-integration-service-v3-0-0/20515
6. eProsima/Integration-Service - GitHub, accessed July 5, 2025, https://github.com/eProsima/Integration-Service
7. github.com, accessed July 5, 2025, [https://github.com/eProsima/Integration-Service#:~:text=eProsima%20Integration%20Service%20is%20a,in%20charge%20of%20maintaining%20it.](https://github.com/eProsima/Integration-Service#:~:text=eProsima Integration Service is a,in charge of maintaining it.)
8. ROSIN project: ROS2 Integration Service - eProsima, accessed July 5, 2025, https://www.eprosima.com/r-d-projects/rosin-project-ros2-integration-service
9. ROS2 Integration Service - ROSIN, accessed July 5, 2025, https://www.rosin-project.eu/ftp/ros2-integration-service
10. Latest version - Integration Service 3.1.0 documentation, accessed July 5, 2025, https://integration-service.docs.eprosima.com/en/latest/release_notes/release_notes.html
11. 1. Integration Service Core - Integration Service 3.1.0 documentation, accessed July 5, 2025, https://integration-service.docs.eprosima.com/en/latest/user_manual/is_core.html
12. Integration Service - eProsima, accessed July 5, 2025, https://www.eprosima.com/middleware/tools/dds-integration-service
13. eProsima's Integration Service System Handle for Fast DDS. - GitHub, accessed July 5, 2025, https://github.com/eProsima/FastDDS-SH
14. 1.1.1. Config - Integration Service 3.1.0 documentation, accessed July 5, 2025, https://integration-service.docs.eprosima.com/en/v3.1.0/api_reference/core/core/config.html
15. 1. Dependencies - Integration Service 3.1.0 documentation, accessed July 5, 2025, https://integration-service.docs.eprosima.com/en/latest/installation_manual/external_dep.html
16. eProsima/Integration-Service-docs: Documentation related to Integration-Service - GitHub, accessed July 5, 2025, https://github.com/eProsima/Integration-Service-docs
17. 2. Installation - Integration Service 3.1.0 documentation, accessed July 5, 2025, https://integration-service.docs.eprosima.com/en/latest/installation_manual/installation.html
18. Company - eProsima, accessed July 5, 2025, https://www.eprosima.com/company
19. Robotics Middleware Essentials - Number Analytics, accessed July 5, 2025, https://www.numberanalytics.com/blog/robotic-middleware-essentials
20. 1. Different Protocols - Integration Service 3.1.0 documentation, accessed July 5, 2025, https://integration-service.docs.eprosima.com/en/latest/examples/different_protocols/examples_different_protocols.html
21. Integration Service New Version 3.1.0 - eProsima, accessed July 5, 2025, https://www.eprosima.com/news/integration-service-new-version-310
22. eProsima's Integration Service System Handle for WebSocket. - GitHub, accessed July 5, 2025, https://github.com/eProsima/WebSocket-SH
23. Examples - Integration Service 3.1.0 documentation, accessed July 5, 2025, https://integration-service.docs.eprosima.com/en/latest/examples/examples.html
24. Developers - eProsima, accessed July 5, 2025, https://www.eprosima.com/developer-resources
25. eProsima Fast DDS - The middleware powering the ESO ELT, accessed July 5, 2025, https://www.eso.org/sci/meetings/2023/RTC4AO/01_11_martin_losa.pdf
26. User Manual - Integration Service 3.1.0 documentation, accessed July 5, 2025, https://integration-service.docs.eprosima.com/en/latest/user_manual/user_manual.html
27. eProsima Fast DDS Performance, accessed July 5, 2025, https://www.eprosima.com/developer-resources/performance/eprosima-fast-dds-performance
28. Fast DDS TSC RMW report 2021 - GitHub Pages, accessed July 5, 2025, https://osrf.github.io/TSC-RMW-Reports/humble/eProsima-response.html
29. eProsima/benchmarking - GitHub, accessed July 5, 2025, https://github.com/eProsima/benchmarking
30. Middleware - eProsima, accessed July 5, 2025, https://www.eprosima.com/middleware
31. 8. Security - 3.2.2 - Fast DDS - eProsima, accessed July 5, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/security/security.html
32. Comparison of MQTT implementations - Wikipedia, accessed July 5, 2025, https://en.wikipedia.org/wiki/Comparison_of_MQTT_implementations
33. OPC UA versus ROS, DDS, and MQTT: Performance Evaluation of Industry 4.0 Protocols - mediaTUM, accessed July 5, 2025, https://mediatum.ub.tum.de/doc/1470362/document.pdf
34. Services - eProsima, accessed July 5, 2025, https://www.eprosima.com/services
35. eProsima: Middleware, Robots and AI, accessed July 5, 2025, https://www.eprosima.com/


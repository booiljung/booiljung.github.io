---
layout: page
title: eProsima Integration Service를 활용한 Fast DDS와 WebSocket 브리지
permalink: /services/dds/Fast-DDS/eProsima Integration Service를 활용한 Fast DDS와 WebSocket 브리지
---


현대의 분산 시스템은 종종 서로 다른 통신 프로토콜을 사용하는 이기종 하위 시스템들로 구성됩니다. 이러한 시스템들을 통합하는 과정은 복잡하며, 각 프로토콜 쌍에 대한 전용 브리지를 개발하는 전통적인 방식은 확장성과 유지보수성 측면에서 한계를 가집니다. eProsima Integration Service는 이러한 문제를 해결하기 위해 설계된 강력하고 유연한 통합 솔루션으로, 프로토콜에 구애받지 않는 중앙 허브 역할을 수행하여 시스템 간의 원활한 데이터 교환을 가능하게 합니다.1


eProsima Integration Service의 핵심은 중앙 집중식 메시지 전달 도구로 기능하는 코어 엔진입니다.2 이 엔진의 가장 큰 특징은 특정 통신 프로토콜의 세부 사항에 대해 알지 못한다는 점입니다. 대신, 모든 프로토콜별 복잡성은 '시스템 핸들(System Handle)'이라는 플러그인 모듈에 위임됩니다. 이러한 설계는 각 시스템 쌍에 대해 N-대-N 방식의 전용 브리지를 만들어야 하는 취약하고 비확장적인 모델에서 벗어나, 중앙 집중화를 통해 통합을 단순화합니다.2

코어 엔진의 주요 책임은 다음과 같습니다 5:

1. **YAML 설정 파일 파싱**: 런타임 시 제공된 YAML 설정 파일을 분석하여 전체 통합 시나리오를 구성합니다.
2. **시스템 핸들 플러그인 로딩**: 설정에 명시된 시스템(예: Fast DDS, WebSocket)에 해당하는 동적 라이브러리(플러그인)를 로드합니다.
3. **내부 타입 레지스트리 관리**: 설정 파일에 정의된 데이터 타입(IDL 기반)을 파싱하여 내부 데이터베이스에 등록합니다.
4. **데이터 라우팅 실행**: 설정된 '경로(routes)'에 따라 시스템 핸들 간의 데이터 흐름을 관리하고 중개합니다.

이러한 아키텍처의 본질적인 장점은 **복잡성의 분리(Complexity Segregation)**에 있습니다. 코어 엔진은 오직 일반화된 데이터의 라우팅에만 집중하며, 프로토콜별 데이터 변환 및 통신 로직은 각 시스템 핸들 내에 완벽하게 캡슐화됩니다. 이 설계 패턴 덕분에 시스템은 견고하면서도 확장이 용이해집니다. 예를 들어, MQTT와 같은 새로운 프로토콜 지원을 추가해야 할 경우, 기존 코어나 다른 시스템 핸들을 수정할 필요 없이 MQTT와 Integration Service의 공통 언어 간의 변환을 담당하는 새로운 시스템 핸들만 개발하면 됩니다. 이는 시스템 통합의 'N-제곱 문제'를 근본적으로 해결하며, N개의 시스템을 통합하는 데 N개의 시스템 핸들만 필요하게 만들어 장기적인 유지보수 비용을 크게 절감합니다.


Integration Service의 강력함은 모든 시스템 핸들이 소통하는 공통의 내부 언어, 즉 OMG(Object Management Group)의 DDS-XTypes(Extensible and Dynamic Topic Types for DDS) 표준 구현체를 사용한다는 점에서 비롯됩니다.1 이 공용어를 통해 서로 다른 프로토콜이 매끄럽게 상호 운용될 수 있습니다.

데이터 변환 과정은 다음과 같이 이루어집니다 5:

1. **수신 및 변환**: 소스 시스템 핸들(예: Fast DDS 핸들)은 자신의 네이티브 포맷(예: CDR 바이너리)으로 데이터를 수신합니다.
2. **xTypes 변환**: 수신된 데이터를 프로토콜에 독립적인 일반적인 xTypes `DynamicData` 표현으로 변환합니다.
3. **라우팅**: 코어 엔진은 이 `DynamicData` 객체를 설정된 경로에 따라 목적지 시스템 핸들로 전달합니다.
4. **재변환 및 전송**: 목적지 시스템 핸들(예: WebSocket 핸들)은 전달받은 `DynamicData` 객체를 자신의 네이티브 포맷(예: JSON 문자열)으로 다시 변환하여 최종 수신자에게 전송합니다.

모든 데이터가 이 xTypes라는 공용어를 거치도록 강제함으로써, 코어 엔진의 API는 안정적으로 유지됩니다. 이는 Integration Service를 미래 지향적인 아키텍처 선택으로 만듭니다. 새로운 통신 프로토콜이 등장하더라도 핵심 프레임워크에 대한 투자는 유효하게 유지되며, 새로운 '드라이버'(시스템 핸들)만 개발하면 되기 때문입니다.


시스템 핸들은 전체 시스템을 작동시키는 프로토콜별 플러그인입니다.2 이는 특정 미들웨어(Fast DDS, ROS 2, WebSocket 등)의 미묘한 차이를 이해하고 처리하는 '드라이버'와 같습니다. 각 시스템 핸들은 대상 프로토콜 및 Integration Service 코어와 인터페이스하기 위해 필요한 발행/구독(pub/sub) 또는 클라이언트/서버(service) 패턴을 구현하는 '미러' 프록시 역할을 합니다.1

eProsima는 다음과 같은 주요 프로토콜에 대한 공식 시스템 핸들을 제공합니다 1:

- Fast DDS System Handle
- ROS 2 System Handle
- ROS 1 System Handle
- WebSocket System Handle (서버 및 클라이언트 모드)
- FIWARE System Handle


Integration Service의 핵심 장점 중 하나는 모든 통합 토폴로지가 런타임에 YAML 파일을 통해 정의된다는 것입니다. 이는 통신 구조를 변경할 때마다 코드를 재컴파일할 필요가 없음을 의미하며, 이는 엄청난 아키텍처 유연성을 제공합니다.1

전체 워크플로우는 매우 간단합니다. 터미널에서 `integration-service` 실행 파일에 설정 YAML 파일의 경로를 인자로 전달하여 실행하기만 하면 됩니다.1 이 접근 방식은 개발 및 배포 주기를 크게 단축시키고, 다양한 환경에 대한 구성을 손쉽게 관리할 수 있게 합니다.


이제 일반적인 프레임워크에서 벗어나, 본 보고서의 핵심인 Fast DDS와 WebSocket 시스템 핸들의 구체적인 기능과 설정 옵션을 심층적으로 살펴보겠습니다. 각 시스템 핸들의 설정 옵션은 해당 기술 도메인의 핵심적인 관심사를 명확하게 반영합니다.


`fastdds` 시스템 핸들은 DDS 네트워크 내에서 표준 DDS Participant(참가자)로 동작하도록 설계되었습니다.9 이를 통해 Integration Service는 DDS 토픽을 발행(publish)하거나 구독(subscribe)하고, DDS 서비스를 제공하거나 호출할 수 있습니다.

주요 설정 매개변수는 다음과 같습니다 9:

- `type: fastdds`: 이 시스템 핸들을 사용하기 위한 필수 식별자입니다.9
- `domain_id`: 서비스가 참여할 DDS 도메인을 지정합니다. 이는 서로 다른 DDS 네트워크를 격리하거나 연결하는 데 매우 중요한 매개변수입니다.
- `participant` 프로파일: `file_path`와 `profile_name`을 사용하여 외부 XML 설정 파일을 로드할 수 있습니다. 이는 DDS의 핵심 기능인 QoS(Quality of Service) 정책(예: 신뢰성, 내구성, 이력 등)을 세밀하게 제어하는 가장 강력한 방법입니다. 실시간 시스템에서 데이터 전송의 신뢰성과 성능을 보장하기 위해 이 기능은 필수적입니다.

Fast DDS 핸들의 설정은 `domain_id`와 `QoS`와 같이 DDS의 데이터 중심 실시간 네트워킹 모델에 중점을 둡니다. 웹 개발자는 일반적으로 이러한 매개변수에 익숙하지 않을 수 있지만, 실시간 시스템 엔지니어에게는 매우 중요한 요소입니다.


WebSocket 시스템 핸들은 서버 또는 클라이언트로 작동할 수 있으며, 본 보고서에서는 주로 `websocket_server` 역할에 초점을 맞춥니다.11 이 핸들은 웹 클라이언트(예: 브라우저, 모바일 앱)와의 양방향 통신 채널을 설정하는 역할을 합니다.

주요 설정 매개변수는 다음과 같습니다 11:

- `type: websocket_server` 또는 `websocket_client`: 핸들의 역할을 정의합니다.
- `port`: `websocket_server`가 클라이언트 연결을 수신 대기할 포트를 지정합니다.
- `security`: 연결 엔드포인트의 보안을 설정합니다. 옵션으로는 TLS/SSL을 사용하는 보안 연결(인증서 및 키 파일 경로 필요)과, `security: none`으로 지정하는 비보안 TCP 연결이 있습니다.6
- `encoding`: 수신된 xTypes 데이터를 WebSocket을 통해 전송하기 위해 어떤 형식으로 인코딩할지를 결정하는 매우 중요한 매개변수입니다. 문서에서는 내장된 `json` 인코딩 프로토콜을 명시적으로 언급하고 있으며, 이것이 기본값으로 사용됩니다.11

WebSocket 핸들의 설정은 `port`, `security`, `encoding`과 같이 모든 웹 기반 통신의 주요 관심사를 다룹니다. 이는 DDS 시스템 엔지니어에게는 생소할 수 있는 영역입니다. 이처럼 Integration Service의 YAML 설정 파일은 두 개의 서로 다른 기술 도메인이 만나는 지점 역할을 하며, 아키텍트가 단일 선언적 파일에서 두 엔드포인트의 동작을 모두 정의할 수 있게 해줍니다. 이는 단순한 설정 파일을 넘어, 두 전문 분야 팀 간의 협업을 촉진하는 **선언적 형식의 인터페이스 제어 문서(Interface Control Document, ICD)** 로 기능한다고 볼 수 있습니다.

또한, WebSocket 시스템 핸들은 토픽/서비스를 광고하고 구독을 관리하기 위해 간단한 JSON 기반 핸드셰이크 프로토콜을 사용합니다. 예를 들어, 클라이언트가 연결되면 서버는 사용 가능한 토픽을 알리는 `advertise` 메시지를 보내고, 클라이언트는 특정 토픽을 수신하기 위해 `subscribe` 메시지를 보냅니다.13


이 섹션에서는 완전한 YAML 설정 파일을 작성하는 실용적이고 단계적인 가이드를 제공하여, 사용자가 직접 데이터 흐름을 설계할 수 있도록 돕습니다.


모든 통신의 기초는 명확한 데이터 스키마를 정의하는 것입니다. Integration Service에서 이는 YAML 파일의 `types` 섹션에서 이루어지며, 일반적으로 인터페이스 정의 언어(Interface Definition Language, IDL) 파일을 포함하거나 직접 내용을 작성하는 방식을 사용합니다.5

Integration Service의 내장 xTypes 파서는 런타임에 이러한 IDL 정의를 처리하여 동적 타입 레지스트리를 생성하며, 이 레지스트리는 모든 후속 데이터 라우팅 및 변환에 사용됩니다.5 예를 들어, 센서 데이터를 위한 간단한 IDL `struct`는 다음과 같이 정의할 수 있습니다.

```YAML
types:
  idls:
    - >
      struct SensorData {
        string sensor_id;
        double temperature;
        int64 timestamp;
      };
```


`systems` 블록의 각 항목은 통합에 참여할 시스템 핸들의 인스턴스를 나타냅니다. 여기서는 각 시스템에 고유한 별칭을 부여하고 필요한 설정을 지정합니다. 앞선 예시에 이어, `fastdds` 시스템과 `websocket_server` 시스템을 다음과 같이 정의할 수 있습니다.

```YAML
systems:
  # DDS 백엔드 시스템 정의
  dds_backend:
    type: fastdds
    domain_id: 5

  # 웹 게이트웨이 시스템 정의
  web_gateway:
    type: websocket_server
    port: 8888
    security: none
    encoding: json
```


실제 데이터 경로를 정의하는 부분입니다. `routes` 섹션은 `systems` 섹션에서 정의한 시스템들 간의 연결을 설정합니다.8

`topics` 섹션은 특정 토픽 이름과 타입을 정의된 경로에 바인딩하는 역할을 합니다. 양방향 통신을 위해서는 각 방향에 대한 경로를 모두 정의해야 합니다.

```YAML
routes:
  # DDS에서 웹으로의 데이터 흐름 경로
  dds_to_web:
    from: dds_backend
    to: web_gateway

  # 웹에서 DDS로의 데이터 흐름 경로
  web_to_dds:
    from: web_gateway
    to: dds_backend

topics:
  # 'SensorData' 타입을 사용하는 'sensor_topic' 정의
  # 이 토픽은 dds_to_web 경로를 사용
  sensor_topic:
    type: SensorData
    route: dds_to_web

  # 'ControlCommand' 타입을 사용하는 'command_topic' 정의
  # 이 토픽은 web_to_dds 경로를 사용
  command_topic:
    type: ControlCommand # 다른 IDL 구조체로 가정
    route: web_to_dds
```


지금까지의 모든 부분을 종합하여, `fastdds_to_websocket.yaml`이라는 완전하고 주석이 달린 예제 파일을 구성해 보겠습니다. 이 예제는 공식 문서의 `ros2_websocket__helloworld.yaml` 13 및 `websocket_server__addtwoints.yaml` 15과 같은 실제 예제를 참조하여 현실성을 높였습니다.

```YAML
# 1. 데이터 타입 정의 (IDL)
types:
  idls:
    - >
      struct SensorData {
        string sensor_id;
        double temperature;
        int64 timestamp;
      };
    - >
      struct ControlCommand {
        string target_id;
        string command;
        sequence<string> args;
      };

# 2. 통신 시스템 정의
systems:
  # Fast DDS 백엔드 시스템 (로봇, 센서 네트워크 등)
  dds_robot:
    type: fastdds
    domain_id: 5

  # 웹 클라이언트 연결을 위한 WebSocket 서버
  web_frontend:
    type: websocket_server
    port: 9090
    security: none # 개발 환경에서는 none, 프로덕션에서는 TLS 설정 권장
    encoding: json

# 3. 데이터 라우팅 규칙 정의
routes:
  # DDS -> WebSocket 경로 (센서 데이터 전송용)
  robot_to_frontend:
    from: dds_robot
    to: web_frontend

  # WebSocket -> DDS 경로 (명령어 전송용)
  frontend_to_robot:
    from: web_frontend
    to: dds_robot

# 4. 토픽과 경로 매핑
topics:
  # DDS의 'robot_sensors' 토픽을 WebSocket에 동일한 이름으로 브리징
  robot_sensors:
    type: SensorData
    route: robot_to_frontend

  # WebSocket의 'robot_command' 토픽을 DDS에 동일한 이름으로 브리징
  robot_command:
    type: ControlCommand
    route: frontend_to_robot
    # 'remap'을 사용하여 프로토콜별로 다른 토픽 이름을 사용할 수도 있음
    # remap:
    #   dds_robot:
    #     topic: /robot/commands_dds
```

이 YAML 구성은 **토픽 및 타입 리매핑(remapping)** 이라는 강력한 패턴을 가능하게 합니다. `remap` 옵션을 사용하면 1 단일 데이터 스트림을 서로 다른 프로토콜에서 다른 토픽 이름이나 호환되는 타입 정의로 노출할 수 있습니다. 예를 들어, ROS 2 백엔드는 `/robot/imu/data`라는 토픽을 사용하지만, 웹 프론트엔드 팀은 `imu`라는 더 간단한 이름을 선호할 수 있습니다. `remap` 기능은 이러한 매핑을 YAML 파일에서 직접 선언적으로 정의할 수 있게 해줍니다. 이는 시스템 간의 강력한 **디커플링 계층**을 제공하여, 백엔드와 프론트엔드 팀이 Integration Service YAML에 정의된 "계약"을 유지하는 한, 내부 토픽 이름과 데이터 구조를 독립적으로 발전시킬 수 있게 합니다. 이는 개발 마찰을 줄이고 팀 간의 병렬 작업을 보다 효과적으로 만듭니다.


Fast DDS와 WebSocket을 브리징할 때 가장 중요한 아키텍처 결정 중 하나는 전송할 데이터의 인코딩 방식을 선택하는 것입니다. Integration Service는 주로 두 가지 접근 방식을 지원하며, 각각 명확한 장단점을 가집니다.


이 방식은 WebSocket 시스템 핸들의 내장 `json` 인코딩 기능을 활용합니다.11 Integration Service가 내부 xTypes 표현에서 JSON 문자열로의 변환을 자동으로 처리합니다.

- **장점**:
  - **가독성**: 데이터 구조가 사람이 읽을 수 있는 텍스트 형식이므로 디버깅이 매우 용이합니다.
  - **상호운용성**: 거의 모든 웹 브라우저(`JSON.parse()`) 및 모바일 플랫폼(Gson/Moshi 등)에서 추가 라이브러리 없이 네이티브하게 지원됩니다(사용자 질의).
- **단점**:
  - **성능 오버헤드**: 텍스트를 파싱하고 생성하는 과정에서 바이너리 형식보다 더 많은 CPU 자원을 소모합니다.
  - **메시지 크기**: 동일한 데이터를 표현하더라도 바이너리보다 크기가 커서 네트워크 대역폭을 더 많이 차지합니다.
  - **정밀도 손실**: 특정 부동소수점 값(예: `Inf`, `NaN`)을 표현할 때 문제가 발생할 수 있었으나, 최신 버전의 Integration Service는 이러한 특수 값 처리를 개선했습니다.6
- **주요 사용 사례**: 상태 업데이트, 설정 변경과 같은 저빈도 데이터 전송, 원격 제어 인터페이스, 그리고 원시 성능보다 개발 속도와 디버깅 용이성이 우선시되는 시나리오에 이상적입니다.


이 아키텍처 패턴은 DDS의 네이티브 직렬화 형식인 CDR(Common Data Representation) 바이너리 페이로드를 WebSocket을 통해 그대로 전달하는 방식입니다. WebSocket 프로토콜은 바이너리 프레임을 기본적으로 지원하므로 이 방식이 가능합니다.

- **장점**:
  - **최소 지연 시간**: 데이터 변환 오버헤드가 거의 없어 매우 낮은 지연 시간을 보장합니다.
  - **고효율**: 데이터 크기가 매우 작아 네트워크 대역폭 소모가 적고, 처리량이 높습니다.
  - **데이터 정밀도**: DDS의 네이티브 형식을 그대로 사용하므로 데이터 손실이 없습니다.
- **단점**:
  - **가독성 부재**: 데이터가 바이너리 스트림이므로 사람이 직접 내용을 파악할 수 없습니다.
  - **클라이언트 복잡성 증가**: 클라이언트 측(웹 또는 모바일)에서 이 바이너리 데이터를 의미 있는 객체로 역직렬화하려면, 데이터 구조에 대한 스키마(IDL)를 기반으로 하는 맞춤형 파서가 필요합니다. 이는 클라이언트 개발의 복잡성을 크게 증가시킵니다.
- **주요 사용 사례**: 수백 Hz 단위의 고주파 센서 데이터 스트리밍, 비디오 또는 포인트 클라우드 데이터 전송 등 성능이 가장 중요한 모든 애플리케이션에 필수적입니다.


아래 표는 두 인코딩 형식 간의 주요 특성을 비교하여 아키텍트가 정보에 기반한 결정을 내릴 수 있도록 돕습니다. 이 표는 복잡하고 다면적인 결정을 구조화된 형식으로 요약하여 강력한 의사결정 도구 역할을 합니다.

| 특성                     | JSON                                     | 바이너리 (CDR)                                     |
| ------------------------ | ---------------------------------------- | -------------------------------------------------- |
| **네트워크 처리량**      | 중간                                     | 높음                                               |
| **지연 시간**            | 중간 (텍스트 파싱으로 인한 오버헤드)     | 낮음 (최소 변환)                                   |
| **CPU 오버헤드**         | 높음 (직렬화/역직렬화 시)                | 낮음 (네이티브 처리)                               |
| **대역폭 소비**          | 높음 (텍스트 기반의 큰 크기)             | 낮음 (고도로 압축된 형식)                          |
| **클라이언트 개발 노력** | 최소 (네이티브 `JSON.parse()` 지원)      | 상당함 (맞춤형 스키마 기반 파서 필요)              |
| **디버깅/가독성**        | 매우 높음                                | 매우 낮음                                          |
| **툴링 지원**            | 범용적 (모든 플랫폼에서 지원)            | 제한적 (`Fast DDS-Gen` 등 특정 도구 필요)          |
| **최적 사용 사례**       | 웹 대시보드, 원격 제어, 개발 편의성 중시 | 고주파 데이터 스트리밍, 실시간 시각화, 성능 극대화 |


이 섹션에서는 데이터가 클라이언트에 도착했을 때 이를 처리하는 실용적인 "방법"을 코드 예제와 함께 제공합니다.


- **JSON 데이터 처리**: WebSocket을 통해 JSON 형식의 텍스트 메시지를 수신하면, JavaScript에서는 `JSON.parse()` 함수 한 줄로 간단하게 네이티브 객체로 변환할 수 있습니다.

  ```JavaScript
  const socket = new WebSocket("ws://localhost:9090");
  
  socket.onmessage = function(event) {
    // event.data는 서버로부터 받은 JSON 문자열
    // 예: '{"sensor_id": "temp_sensor_01", "temperature": 25.5, "timestamp": 1678886400}'
    console.log("수신된 텍스트:", event.data);
  
    try {
      // JSON 문자열을 JavaScript 객체로 변환
      const sensorData = JSON.parse(event.data);
  
      // 변환된 객체의 속성에 직접 접근
      console.log("변환된 객체:", sensorData);
      console.log("센서 ID:", sensorData.sensor_id); // "temp_sensor_01"
      console.log("온도:", sensorData.temperature); // 25.5
    } catch (e) {
      console.error("JSON 파싱 오류:", e);
    }
  };
  
  socket.onopen = function(event) {
    console.log("WebSocket 연결 성공");
    // 필요 시, 구독 메시지 전송
    // const subscriptionMsg = { op: "subscribe", topic: "robot_sensors" };
    // socket.send(JSON.stringify(subscriptionMsg));
  };
  ```
  
- **바이너리 데이터 처리**: 바이너리 경로를 웹 클라이언트에서 사용하는 것은 상당한 도전 과제입니다. eProsima는 공식적인 JavaScript용 CDR 파서를 제공하지 않으므로 18, 개발자는 다음 단계를 직접 수행해야 합니다:

  1. 데이터에 대한 IDL 스키마를 확보합니다.
  2. WebSocket을 통해 수신된 `ArrayBuffer`를 CDR 사양과 IDL 스키마에 따라 역직렬화하는 JavaScript 함수를 직접 구현합니다. 이는 상당한 노력이 필요한 작업이며, 바이너리 접근 방식을 웹 환경에서 채택하는 데 가장 큰 장애물입니다. Node.js 환경을 위한 `is-web-api` 20 프로젝트가 참고 자료가 될 수는 있지만, 브라우저 클라이언트에 직접 적용하기에는 한계가 있습니다.


- **JSON 데이터 처리**: 안드로이드의 주력 언어인 Kotlin을 사용하여 JSON을 처리하는 것은 웹만큼이나 간단합니다. **Gson**이나 **Moshi**와 같은 표준 라이브러리를 사용하면 JSON 문자열을 Kotlin `data class` 인스턴스로 손쉽게 변환할 수 있습니다.

  ```Kotlin
  // 1. 수신할 데이터 구조와 일치하는 데이터 클래스 정의
  data class SensorData(val sensor_id: String, val temperature: Double, val timestamp: Long)
  
  // 2. Gson 라이브러리를 사용하여 변환
  val gson = com.google.gson.Gson()
  
  fun handleWebSocketMessage(jsonString: String) {
      // jsonString은 서버로부터 받은 JSON 문자열
      try {
          // JSON 문자열을 SensorData 객체로 자동 변환
          val sensorDataObject = gson.fromJson(jsonString, SensorData::class.java)
  
          // 변환된 객체 사용
          println("수신된 객체: $sensorDataObject")
          println("온도 값 접근: ${sensorDataObject.temperature}")
      } catch (e: com.google.gson.JsonSyntaxException) {
          println("JSON 파싱 오류: $e")
      }
  }
  ```
  
- **바이너리 데이터 처리와 `Fast DDS-Gen`**: `Fast DDS-Gen` 도구의 존재는 안드로이드에서 바이너리 경로를 훨씬 더 실용적이고 매력적인 선택으로 만듭니다. 이 도구는 IDL로부터 바이너리 데이터를 처리하는 Java 코드를 자동으로 생성해주기 때문입니다.

  **단계별 가이드:**

  1. **IDL 파일 정의**: 웹 클라이언트 예제와 동일한 `SensorData.idl` 파일을 사용합니다.

  2. **`Fast DDS-Gen` 실행**: 터미널에서 다음 명령을 실행하여 IDL로부터 Java 코드를 생성합니다. `Fast DDS-Gen`은 Java를 네이티브로 지원합니다.21

     ```Bash
     fastddsgen -language java SensorData.idl
     ```
     
  3. **생성된 코드 통합**: 이 명령은 `SensorData.java` (데이터 타입 클래스), `SensorDataPubSubType.java` (직렬화/역직렬화 로직 포함) 등의 파일을 생성합니다. 이 `.java` 파일들을 안드로이드 스튜디오 프로젝트의 `src/main/java` 폴더에 포함시킵니다.

  4. **Kotlin에서 사용**: Kotlin은 Java와 100% 상호 운용 가능하므로, 생성된 Java 클래스를 마치 네이티브 Kotlin 클래스처럼 사용할 수 있습니다. 다음은 WebSocket으로부터 수신한 `ByteArray`를 타입-안전한 `SensorData` 객체로 변환하는 예제입니다.

  ```Kotlin
  import com.your.package.SensorData // 생성된 Java 클래스
  import com.your.package.SensorDataPubSubType // 생성된 Java 클래스
  
  fun handleWebSocketBinaryMessage(data: ByteArray) {
      // 1. 생성된 PubSubType 인스턴스 생성
      val pubSubType = SensorDataPubSubType()
  
      // 2. 역직렬화를 위한 데이터 객체 생성
      val receivedData = SensorData()
  
      // 3. eProsima Fast CDR을 사용하여 ByteArray를 객체로 역직렬화
      // 생성된 코드 내부에 이 로직이 포함되어 있음
      val dataRepresentation = eprosima.fastcdr.Cdr.DEFAULT_REPRESENTATION
      val cdr = eprosima.fastcdr.Cdr(
          java.nio.ByteBuffer.wrap(data),
          dataRepresentation
      )
      pubSubType.deserialize(cdr, receivedData)
  
      // 4. 완벽하게 타입이 지정된 객체 사용
      println("역직렬화된 객체: $receivedData")
      println("센서 ID: ${receivedData.sensor_id}")
  }
  ```

이처럼 `Fast DDS-Gen`의 가용성은 안드로이드 클라이언트에서 바이너리/CDR 경로를 웹 클라이언트보다 훨씬 더 실현 가능하고 매력적인 옵션으로 만듭니다. 안드로이드에서 바이너리 데이터를 처리하는 데 드는 개발 노력은 웹에 비해 현저히 낮습니다. 이는 아키텍트가 하이브리드 접근 방식을 선택하도록 유도할 수 있습니다. 예를 들어, 간단한 웹 기반 대시보드에는 **JSON**을 사용하고, 더 높은 성능이 요구되는 기능이 풍부한 안드로이드 원격 제어 애플리케이션에는 **바이너리/CDR**을 사용하는 것입니다. eProsima의 툴체인은 이러한 플랫폼별 최적화를 자연스럽게 장려합니다.


이 마지막 섹션에서는 시야를 넓혀 일반적인 고급 사용 사례를 논의하고, 아키텍처 수준의 조언을 제공합니다.


이 기술이 ROS 2 생태계에서 특히 널리 사용되는 이유는 ROS 2의 기본 미들웨어가 Fast DDS이기 때문입니다.25 따라서 Integration Service는 ROS 2의 토픽, 서비스, 액션을 웹 세계로 거의 투명하게 브리징하는 역할을 합니다. 공식 문서에는 ROS 2 발행자를 WebSocket 클라이언트에 연결하거나 13, WebSocket에서 ROS 2 서비스를 호출하는 등 15 수많은 ROS 2 관련 예제가 포함되어 있습니다. 이를 통해 로봇의 상태를 웹에서 실시간으로 모니터링하거나, 웹 인터페이스를 통해 로봇을 원격으로 제어하는 강력한 애플리케이션을 손쉽게 구축할 수 있습니다.


고급 사용 사례 중 하나는 두 개의 Integration Service 인스턴스를 사용하여 인터넷을 통해 DDS 도메인을 브리징하는 것입니다.9 이는 지리적으로 분산된 로봇 시스템이나 관제 센터를 연결하는 강력한 패턴입니다.

아키텍처는 다음과 같습니다:

DDS 앱 (LAN 1) <-> IS (WAN 게이트웨이 1) <---인터넷(TCP)---> IS (WAN 게이트웨이 2) <-> DDS 앱 (LAN 2)

이 구성에서 각 Integration Service 인스턴스는 로컬 DDS 네트워크와 통신하는 동시에, 다른 인스턴스와 WAN을 통해 TCP 연결을 설정합니다. 이를 통해 물리적으로 떨어진 두 개의 독립적인 DDS 네트워크가 마치 하나의 네트워크인 것처럼 데이터를 교환할 수 있습니다.2


브리지의 성능과 안정성을 최적화하기 위한 몇 가지 실용적인 권장 사항은 다음과 같습니다.

- **하이브리드 인코딩 사용**: 고주파 데이터(센서, 영상)에는 바이너리/CDR을, 저빈도 데이터(제어, 상태)에는 JSON을 사용하여 성능과 개발 편의성의 균형을 맞춥니다.
- **DDS QoS 정책 활용**: 실시간 측의 데이터 신뢰성, 내구성, 이력 등을 관리하기 위해 Fast DDS 시스템 핸들의 XML 프로파일 기능을 적극적으로 활용합니다.
- **엔드포인트 보안**: 웹에 노출되는 WebSocket 엔드포인트는 반드시 TLS를 사용하여 암호화해야 합니다.
- **명확한 IDL 계약 설계**: 모든 통신의 근간이 되는 IDL 파일을 명확하게 설계하고 버전을 관리하여, 모든 시스템이 따르는 "진실의 원천(source of truth)"으로 삼습니다.


결론적으로, eProsima Integration Service는 실시간 DDS 기반 시스템과 표준 웹 기술의 세계를 잇는 성숙하고 유연하며 고성능의 도구입니다. 그 강점은 DDS 및 xTypes와 같은 표준 기반의 플러그인 주도 아키텍처와 선언적인 런타임 설정에 있습니다. 데이터 인코딩 형식(JSON 대 바이너리) 간의 장단점을 이해하고 `Fast DDS-Gen`과 같은 제공된 툴체인을 활용함으로써, 아키텍트는 현대적인 상호 연결 애플리케이션의 요구 사항을 효과적으로 충족하는 견고하고 확장 가능하며 유지보수 용이한 시스템을 설계할 수 있습니다. 이 서비스는 단순히 두 프로토콜을 연결하는 것을 넘어, 서로 다른 기술 스택을 사용하는 팀들이 원활하게 협력할 수 있는 공통의 기반을 제공합니다.


1. eProsima/Integration-Service - GitHub, accessed July 15, 2025, https://github.com/eProsima/Integration-Service
2. Integration Service Core - Integration Service 3.1.0 documentation - eProsima, accessed July 15, 2025, https://integration-service.docs.eprosima.com/
3. Integration Service - eProsima, accessed July 15, 2025, https://www.eprosima.com/middleware/tools/dds-integration-service
4. Integration Service - allowing integration of any protocol into DDS - ROS Discourse, accessed July 15, 2025, https://discourse.ros.org/t/integration-service-allowing-integration-of-any-protocol-into-dds/16135
5. 1. Integration Service Core, accessed July 15, 2025, https://integration-service.docs.eprosima.com/en/latest/user_manual/is_core.html
6. Latest version - Integration Service 3.1.0 documentation, accessed July 15, 2025, https://integration-service.docs.eprosima.com/en/latest/release_notes/release_notes.html
7. 2. System Handle - Integration Service 3.1.0 documentation, accessed July 15, 2025, https://integration-service.docs.eprosima.com/en/latest/user_manual/systemhandle/sh.html
8. 3. YAML Configuration - Integration Service 3.1.0 documentation, accessed July 15, 2025, https://integration-service.docs.eprosima.com/en/latest/user_manual/yaml_config.html
9. eProsima/FastDDS-SH: eProsima's Integration Service ... - GitHub, accessed July 15, 2025, https://github.com/eProsima/FastDDS-SH
10. 2.1.1. Fast DDS System Handle - eProsima Integration Service, accessed July 15, 2025, https://integration-service.docs.eprosima.com/en/latest/user_manual/systemhandle/fastdds_sh.html
11. 2.1.5. WebSocket System Handle - eProsima Integration Service, accessed July 15, 2025, https://integration-service.docs.eprosima.com/en/latest/user_manual/systemhandle/websocket_sh.html
12. Integration-Service/examples/basic/ros1_server__addtwoints.yaml at main - GitHub, accessed July 15, 2025, https://github.com/eProsima/Integration-Service/blob/main/examples/basic/ros1_server__addtwoints.yaml
13. 1.1.5. ROS 2 - WebSocket bridge - Integration Service 3.1.0 documentation, accessed July 15, 2025, https://integration-service.docs.eprosima.com/en/latest/examples/different_protocols/pubsub/ros2-websocket.html
14. eProsima's Integration Service System Handle for WebSocket. - GitHub, accessed July 15, 2025, https://github.com/eProsima/WebSocket-SH
15. Integration-Service-docs/docs/examples/different_protocols/services/websocket-server.rst at master - GitHub, accessed July 15, 2025, https://github.com/eProsima/Integration-Services-docs/blob/master/docs/examples/different_protocols/services/websocket-server.rst
16. Integration-Service/examples/basic/websocket_server__addtwoints.yaml at main - GitHub, accessed July 15, 2025, https://github.com/eProsima/Integration-Service/blob/main/examples/basic/websocket_server__addtwoints.yaml
17. 1.1.1. Config - Integration Service 3.1.0 documentation, accessed July 15, 2025, https://integration-service.docs.eprosima.com/en/latest/api_reference/core/core/config.html
18. The WebSocket API (WebSockets) - Web APIs - MDN Web Docs, accessed July 15, 2025, https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API
19. websocket - NPM, accessed July 15, 2025, https://www.npmjs.com/package/websocket
20. eProsima/is-web-api - GitHub, accessed July 15, 2025, https://github.com/eProsima/is-web-api
21. Fast DDS - eProsima, accessed July 15, 2025, https://fast-dds.docs.eprosima.com/
22. Fast DDS Installation | PX4 User Guide (v1.12), accessed July 15, 2025, https://docs.px4.io/v1.12/en/dev_setup/fast-dds-installation.html
23. 1. Introduction - 3.3.0 - Fast DDS - eProsima, accessed July 15, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastddsgen/introduction/introduction.html
24. Fast-DDS-Gen/src/main/java/com/eprosima/fastdds/fastddsgen.java at master - GitHub, accessed July 15, 2025, https://github.com/eProsima/Fast-DDS-Gen/blob/master/src/main/java/com/eprosima/fastdds/fastddsgen.java
25. Fast DDS Documentation, accessed July 15, 2025, https://media.readthedocs.org/pdf/eprosima-fast-rtps/stable/eprosima-fast-rtps.pdf
26. eProsima/Fast-DDS: The most complete DDS - Proven: Plenty of success cases. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 15, 2025, https://github.com/eProsima/Fast-DDS
27. eProsima's Integration Service System Handle for ROS 2. - GitHub, accessed July 15, 2025, https://github.com/eProsima/ROS2-SH
28. 1.2.4. WebSocket Service Server - Integration Service 3.1.0 documentation, accessed July 15, 2025, https://integration-service.docs.eprosima.com/en/latest/examples/different_protocols/services/websocket-server.html
29. Examples - Integration Service 3.1.0 documentation, accessed July 15, 2025, https://integration-service.docs.eprosima.com/en/latest/examples/examples.html


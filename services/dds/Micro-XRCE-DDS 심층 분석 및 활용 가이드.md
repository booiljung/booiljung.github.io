# Micro-XRCE-DDS 심층 분석 및 활용 가이드

## 1.  Micro-XRCE-DDS의 기본 원리

### 1.1  OMG DDS-XRCE 표준: 자원 제약 환경으로의 교량

표준 데이터 분산 서비스(Data Distribution Service, DDS)는 실시간 분산 시스템을 위한 강력한 데이터 중심 통신 미들웨어이지만, 높은 성능을 위해 상당한 메모리와 처리 능력을 요구합니다. 이로 인해 마이크로컨트롤러와 같은 자원이 극도로 제한된 장치에서는 직접 사용하기 어렵습니다.1 이러한 근본적인 문제를 해결하기 위해 Object Management Group (OMG)는 'eXtremely Resource Constrained Environments'를 위한 DDS, 즉 DDS-XRCE라는 공식 프로토콜 표준을 제정했습니다.1 이 표준의 핵심 목표는 RAM 32 KB, 플래시 메모리 250 KB 수준의 초소형 장치도 완전한 DDS 통신 네트워크에 참여할 수 있도록 하는 것입니다.1

eProsima사의 Micro-XRCE-DDS는 바로 이 OMG DDS-XRCE 표준을 구현한 구체적인 오픈소스 소프트웨어 솔루션입니다.2 즉, Micro-XRCE-DDS는 특정 기업의 독자적인 기술이 아니라, 산업 표준에 기반한 개방형 기술이라는 점에서 중요한 의미를 가집니다.

### 1.2  핵심 철학: 마이크로컨트롤러를 위한 데이터 중심(Data-Centricity)

DDS의 가장 중요한 철학은 '데이터 중심' 통신 모델입니다. 이는 단순히 메시지를 주고받는 '메시지 중심(Message-centric)' 모델(예: MQTT)과 근본적인 차이를 보입니다.7 데이터 중심 모델에서 통신의 주체는 메시지가 아니라, 공유되는 '데이터' 그 자체입니다. 각 애플리케이션은 특정 '토픽(Topic)'의 데이터를 발행(Publish)하거나 구독(Subscribe)함으로써, 마치 공유된 데이터 공간(Global Data Space)의 상태를 읽고 쓰는 것처럼 동작합니다.7

Micro-XRCE-DDS는 이 강력하고 신뢰성 높은 데이터 중심 패러다임을 마이크로컨트롤러까지 확장합니다. 전통적으로 임베디드 장치들은 간단한 커스텀 직렬 프로토콜에 의존해왔지만, Micro-XRCE-DDS를 통해 복잡한 분산 시스템의 일원으로서 정형화되고 안정적인 데이터 공유가 가능해집니다.9

### 1.3  클라이언트-에이전트 패러다임: 아키텍처 개요

DDS-XRCE 표준의 핵심은 클라이언트-서버(Client-Server) 아키텍처에 기반합니다.2 이 아키텍처는 표준 DDS의 자원 요구량과 마이크로컨트롤러의 제약 사이의 간극을 메우는 핵심적인 해결책입니다. 표준 DDS 네트워크에서 모든 참여자(Participant)는 동등한 피어(Peer)로서 서로를 발견하고 연결을 관리해야 하므로 높은 자원을 소모합니다.1 마이크로컨트롤러는 이러한 복잡성을 감당할 수 없으며, 특히 동적 네트워크 환경이나 절전 모드(sleep cycle)가 필요한 경우 더욱 그렇습니다.1

DDS-XRCE의 클라이언트-에이전트 모델은 이러한 복잡성을 에이전트에게 위임함으로써 문제를 해결합니다.

- **Micro-XRCE-DDS Client**: 자원이 제한된 장치(예: 마이크로컨트롤러)에서 실행되도록 설계된 경량 C99 라이브러리입니다. 클라이언트는 매우 가볍고, 동적 메모리 할당 없이 동작할 수 있도록 설계되었습니다.2
- **Micro-XRCE-DDS Agent**: 상대적으로 자원이 풍부한 컴퓨터(예: 컴패니언 컴퓨터, PC)에서 실행되는 C++11 애플리케이션입니다. 에이전트는 클라이언트를 대신하여 DDS 네트워크에 참여하는 브로커 또는 프록시(Proxy) 역할을 수행합니다.2

에이전트는 클라이언트와 주 DDS 네트워크(DDS Global Data Space)를 연결하는 다리 역할을 하며, 이 과정에서 eProsima의 Fast DDS와 같은 표준 DDS 구현체를 사용합니다.3 결국, 클라이언트-에이전트 아키텍처는 단순한 설계 선택이 아니라, 마이크로컨트롤러에서 DDS를 실현 가능하게 만드는 본질적인 메커니즘입니다.

### 1.4  핵심 용어: 오퍼레이션, 엔티티, 그리고 전역 데이터 공간

Micro-XRCE-DDS를 이해하기 위해서는 몇 가지 핵심 용어에 대한 명확한 정의가 필요합니다.

- **DDS 전역 데이터 공간 (DDS Global Data Space)**: 모든 DDS 데이터가 존재하는 개념적인 공간입니다. 권한이 있는 모든 DDS 참여자는 이 공간에 접근하여 데이터를 발행하거나 구독할 수 있습니다.2
- **오퍼레이션 (Operations)**: 클라이언트와 에이전트 간의 근본적인 통신 방식입니다. 클라이언트는 에이전트에게 특정 작업(예: 엔티티 생성, 데이터 쓰기, 데이터 읽기)을 수행하도록 '요청(request)'하고, 에이전트는 그 결과 상태를 담은 '응답(response)'을 보냅니다.2
- **엔티티 (Entities)**: DDS의 객체들(Participant, Topic, Publisher, Subscriber, DataWriter, DataReader 등)을 의미합니다. 매우 중요한 점은, 이 엔티티들이 클라이언트의 요청에 의해 *에이전트 상에* 생성되고 관리된다는 것입니다.2 클라이언트는 이 엔티티들을 직접 소유하는 것이 아니라, 에이전트에게 생성을 요청하고 원격으로 제어하는 개념입니다. 이는 자원 제약을 해결하는 프록시 모델의 핵심입니다.

## 2.  아키텍처 심층 분석: 클라이언트-에이전트 상호작용 모델

### 2.1  Micro-XRCE-DDS Client: 경량 액터

클라이언트는 극도의 자원 제약 환경을 위해 세심하게 설계되었습니다. 그 핵심 설계 원칙은 다음과 같습니다.

- **최소 메모리 사용량**: 간단한 발행/구독 애플리케이션의 경우, 플래시 메모리 75KB 미만, RAM 약 3KB 정도의 매우 적은 자원만을 소모합니다.5
- **높은 이식성**: 표준 C99로 작성되어 다양한 마이크로컨트롤러 및 RTOS 환경에 쉽게 이식할 수 있습니다.5
- **동적 메모리 할당 없음**: 초기화 단계 이후, 런타임 중에는 동적 메모리 할당(`malloc` 등)을 전혀 사용하지 않습니다. 이는 실시간 시스템의 예측 가능성과 안정성을 보장하는 데 매우 중요합니다.13

클라이언트의 주된 역할은 애플리케이션의 요청과 데이터를 Micro-CDR 라이브러리를 사용해 직렬화하고, 이를 에이전트에게 전송한 뒤, 에이전트로부터의 응답을 처리하는 것입니다.5 클라이언트는 DDS 네트워크와 직접 상호작용하지 않으며, 모든 복잡한 작업은 에이전트에게 위임합니다.

### 2.2  Micro-XRCE-DDS Agent: 브로커이자 프록시

에이전트는 연결된 클라이언트들을 대신하여 완전한 기능을 갖춘 DDS 참여자(Participant)로서 동작합니다.2 에이전트의 핵심 기능은 다음과 같습니다.

1. 클라이언트로부터 UDP, TCP, 직렬 통신 등 다양한 전송 계층을 통해 오퍼레이션 요청(예: "Publisher를 생성하라")을 수신합니다.2

2. 수신한 XRCE 요청을 Fast DDS와 같은 표준 DDS 구현체를 사용하여 실제 DDS 동작으로 변환합니다.3 예를 들어, 클라이언트의 '토픽 생성' 요청은 에이전트 내에서 실제 

   `dds::topic::Topic` 객체를 생성하는 것으로 이어집니다.

3. 클라이언트들을 위해 생성된 모든 DDS 엔티티의 생명주기(lifecycle)를 관리합니다. 클라이언트와의 연결이 끊어지면 관련 엔티티들을 안전하게 정리합니다.17

### 2.3  `ProxyClient`의 역할: 에이전트가 클라이언트를 대리하는 방법

에이전트 아키텍처의 핵심에는 `ProxyClient`라는 내부 엔티티가 있습니다.2

`ProxyClient`는 에이전트 내에서 원격의 물리적 클라이언트를 1:1로 대변하는 소프트웨어 객체입니다. 이는 전체 아키텍처의 핵심적인 연결고리 역할을 합니다.

`ProxyClient`는 각 클라이언트 세션의 상태를 유지하며, 해당 클라이언트가 소유한 모든 DDS 엔티티(Participant, Topic 등)의 목록과 고유한 클라이언트 키(Client Key)를 관리합니다.2 예를 들어, 한 클라이언트가 '토픽 생성' 요청을 보내면, 에이전트는 해당 클라이언트와 연결된 

`ProxyClient` 인스턴스를 통해 실제 Fast DDS 토픽 객체를 생성합니다. 이 `ProxyClient`는 클라이언트의 대리인 역할을 하여, 자원이 부족한 물리적 클라이언트가 마치 완전한 기능을 갖춘 DDS 참여자인 것처럼 보이게 만듭니다. 이러한 프록시 패턴의 적용은 DDS-XRCE가 성공적으로 동작하게 하는 핵심적인 아키텍처 혁신입니다.

### 2.4  통신 흐름: 요청-응답 주기의 단계별 분석

클라이언트와 에이전트 간의 일반적인 통신 흐름은 다음과 같은 단계로 이루어집니다.

1. **세션 생성**: 클라이언트는 고유한 `client_key`를 포함한 `Create session` 오퍼레이션을 에이전트에게 전송하여 통신을 시작합니다.16 에이전트는 이 키를 등록하고 해당 클라이언트를 위한 

   `ProxyClient` 인스턴스를 내부적으로 생성합니다.

2. **엔티티 생성**: 클라이언트는 생성된 세션을 통해 `create_participant_xml`과 같은 `Create entity` 오퍼레이션을 전송합니다. 이 요청에는 클라이언트가 나중에 해당 엔티티를 참조하기 위한 Object ID와 엔티티의 정의(예: XML 문자열)가 포함됩니다.20

3. **에이전트 처리**: 에이전트는 요청을 수신하고, 해당 `ProxyClient`는 요청에 따라 실제 DDS 엔티티(예: `dds::domain::DomainParticipant`)를 생성합니다.

4. **상태 응답**: 에이전트는 오퍼레이션의 성공 또는 실패 여부를 알리는 상태 메시지를 클라이언트에게 다시 보냅니다.2

5. **데이터 교환**: 클라이언트는 이전에 생성한 DataWriter/DataReader의 Object ID를 참조하여 `Write Data` 또는 `Read Data` 오퍼레이션을 사용해 데이터를 발행하거나 구독합니다.20 실제 DDS 

   `write()` 또는 `take()` 호출은 에이전트의 `ProxyClient`가 대신 처리합니다.

### 2.5  지원 전송 방식: 기술적 검토

Micro-XRCE-DDS의 주요 강점 중 하나는 특정 전송 방식에 종속되지 않는다는 점입니다. 이는 TCP에 의존하는 MQTT와 같은 프로토콜에 비해 큰 유연성을 제공합니다.5

- **내장 전송 방식**: UDPv4/v6, TCPv4/v6, 직렬(Serial) 통신이 공식적으로 지원됩니다.2 CAN FD 또한 지원 목록에 언급됩니다.15
- **커스텀 전송 방식**: 사용자가 직접 자신만의 전송 계층을 구현할 수 있는 인터페이스를 명시적으로 제공합니다. 이를 통해 블루투스, 6LoWPAN, 또는 기타 커스텀 무선 링크와 같은 다양한 통신 환경에 적용할 수 있습니다.2
- **스트림 지향 전송을 위한 프레이밍**: 직렬 통신과 같이 패킷의 경계가 명확하지 않은 스트림 지향 전송을 위해, Micro-XRCE-DDS는 HDLC(High-Level Data Link Control) 기반의 프레이밍 프로토콜을 구현합니다. 이 프레이밍은 메시지를 명확하게 구분하고 CRC(Cyclic Redundancy Check)를 통해 데이터 무결성을 보장합니다.9

## 3.  실용적 구현: 설치 및 환경 설정

### 3.1  전제 조건 및 의존성

Micro-XRCE-DDS를 사용하기 위해서는 몇 가지 기본적인 도구와 라이브러리가 필요합니다.

- **필수 도구**: C++11을 지원하는 컴파일러, CMake, Git이 필요합니다.24
- **Agent 의존성**: Agent를 빌드하기 위해서는 eProsima의 Fast DDS와 그 의존성인 Fast CDR이 시스템에 설치되어 있어야 합니다.25
- **Gen 도구 의존성**: IDL 파일로부터 코드를 생성하는 `Micro-XRCE-DDS-Gen` 도구를 사용하려면 Java Development Kit (JDK)와 Gradle이 필요합니다.26

### 3.2  방법 1: 소스 코드로부터 직접 빌드하기

이 방법은 빌드 플래그(프로파일)를 직접 제어하거나 커스텀 빌드 시스템에 통합해야 하는 개발자에게 가장 적합합니다. 최대의 유연성을 제공하지만 설정이 가장 복잡합니다.

리눅스 환경에서의 일반적인 빌드 과정은 다음과 같습니다.22

Bash

```
# Agent 빌드 예시
git clone -b v2.4.2 https://github.com/eProsima/Micro-XRCE-DDS-Agent.git
cd Micro-XRCE-DDS-Agent
mkdir build
cd build
cmake..
make
sudo make install
sudo ldconfig /usr/local/lib/
```

윈도우 환경에서는 Visual Studio용 프로젝트 파일을 생성하여 빌드할 수 있습니다 (`cmake -G "Visual Studio..."`).26

### 3.3  방법 2: Snap 패키지 및 Docker를 이용한 간편 배포

- **Snap**: Ubuntu 사용자는 `sudo snap install micro-xrce-dds-agent` 명령 하나로 Agent를 간편하게 설치할 수 있습니다.17 포트 설정 등은 

  `snap set micro-xrce-dds-agent port=8888`과 같은 명령으로 수행합니다.17

- **Docker**: eProsima는 Agent와 Client를 위한 사전 빌드된 Docker 이미지를 제공합니다. 이는 격리된 테스트 환경을 구성하거나 CI/CD 파이프라인에 통합할 때 매우 유용하며, 일관된 실행 환경을 보장합니다.26

### 3.4  방법 3: ROS 2 작업 공간에 통합하기 (Colcon)

micro-ROS 개발자에게 가장 일반적인 방법입니다. Agent 소스 코드를 ROS 2 작업 공간의 `src` 디렉토리에 복제하고 `colcon build` 명령으로 빌드합니다.19 이 방법은 Agent를 다른 ROS 2 노드처럼 실행하고 관리할 수 있게 하여 ROS 2 생태계와의 완벽한 통합을 보장합니다. 빌드 후에는 

`source install/setup.bash` 명령으로 환경을 설정해야 합니다.25

### 3.5  에이전트 설정 및 실행: 명령줄 인터페이스(CLI) 활용

`MicroXRCEAgent` 실행 파일은 다양한 명령줄 인자를 통해 설정할 수 있습니다.15

- **전송 방식 선택**: `udp4`, `tcp4`, `serial` 등 통신 방식을 지정합니다.15
- **필수 인자**: `-p` 또는 `--port`를 사용하여 수신 대기할 포트를 지정해야 합니다.15
- **선택적 인자**: `-v` (상세 로깅 수준), `-d` (Discovery 서버 활성화), `-r` (참조 기반 엔티티 생성 파일 로드) 등이 있습니다.15

**일반적인 실행 예시:**

- UDP 포트 8888에서 수신 대기: `MicroXRCEAgent udp4 -p 8888` 27
- 직렬 포트와 전송 속도 지정: `MicroXRCEAgent serial --dev /dev/ttyUSB0 -b 921600` 27

### 3.6 표 1: 설치 방법 비교

| 방법                | 장점                                                         | 단점                                                         | 주요 사용 사례                                               |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **소스 빌드**       | 빌드 옵션(프로파일)에 대한 완전한 제어, 디버깅 용이, 커스텀 라이브러리와의 연동 가능 | 가장 복잡한 설정, 수동 의존성 관리 필요, 시스템 라이브러리와의 충돌 가능성 28 | 핵심 라이브러리 개발자, 고도로 최적화/커스텀된 빌드가 필요한 사용자, 비표준 플랫폼 통합 |
| **Snap 패키지**     | 매우 간단한 설치 (`sudo snap install...`), 자동 업데이트     | Ubuntu에 한정, 설정 유연성 제한적                            | 빠른 테스트, 데모, 표준 환경에서의 간단한 배포               |
| **Docker**          | 일관되고 격리된 실행 환경 보장, 의존성 문제 해결, CI/CD에 이상적 | Docker에 대한 이해 필요, 네트워킹 설정이 복잡할 수 있음      | 개발 환경 표준화, 테스트 자동화, 클라우드 배포               |
| **ROS 2 작업 공간** | ROS 2 생태계와의 완벽한 통합, `colcon` 및 `ros2 launch`와 같은 도구 활용 가능 | ROS 2 개발 환경 설정이 선행되어야 함                         | micro-ROS 애플리케이션 개발                                  |

## 4.  개발자를 위한 Micro-XRCE-DDS 애플리케이션 구축 가이드

### 4.1  인터페이스 정의 언어(IDL)를 이용한 데이터 구조 정의

DDS에서는 인터페이스 정의 언어(Interface Definition Language, IDL)를 사용하여 통신에 사용될 데이터의 구조를 공식적으로 정의합니다. 이는 데이터 타입을 명확히 하고, 서로 다른 시스템 간의 상호 운용성을 보장하는 표준적인 방법입니다.11

예를 들어, 간단한 "Hello World" 메시지를 위한 IDL 파일은 다음과 같이 작성할 수 있습니다.20

Code snippet

```
struct HelloWorld {
    unsigned long index;
    string message;
};
```

### 4.2  `Micro-XRCE-DDS-Gen`을 이용한 코드 생성

`microxrceddsgen`이라는 도구는 IDL 파일을 입력으로 받아, 해당 데이터 구조를 C 언어에서 직렬화(serialize)하고 역직렬화(deserialize)하는 데 필요한 코드를 자동으로 생성해줍니다.5 예를 들어, 위 IDL로부터 `HelloWorld_size_of_topic`, `HelloWorld_serialize_topic`과 같은 함수들이 포함된 C 헤더 파일이 생성됩니다. 이 과정은 지루하고 오류가 발생하기 쉬운 수동 코딩 작업을 제거하여 개발 효율성을 크게 높여줍니다.

### 4.3  클라이언트 애플리케이션 생명주기: 단계별 가이드

클라이언트 애플리케이션을 개발하는 과정은 일련의 정형화된 단계를 따릅니다. 이 과정은 API의 설계 철학을 잘 보여주는데, 각 API 호출이 즉시 네트워크 통신을 유발하는 것이 아니라, 내부 스트림에 작업을 '버퍼링'하고, `uxr_run_session...` 계열의 함수가 호출될 때 일괄적으로 처리하는 효율적인 방식을 채택하고 있습니다. 이는 네트워크 통신 횟수를 최소화하여 임베디드 장치의 부하를 줄이는 중요한 최적화 기법입니다.

다음은 공식 `getting_started` 문서를 기반으로 한 단계별 개발 과정입니다.20

1. **전송 및 세션 초기화**: 먼저 사용할 전송 방식을 초기화하고(예: `uxr_init_udp_transport`), 이어서 세션을 초기화합니다(`uxr_init_session`). 세션 초기화 시에는 전송 객체, 고유한 클라이언트 키, 그리고 선택적인 콜백 함수들을 전달합니다. 마지막으로 `uxr_create_session`을 호출하여 에이전트와의 실제 세션을 설정합니다.20
2. **통신 채널 설정**: 입출력 스트림을 생성합니다. `uxr_create_output_best_effort_stream` (최선 노력)과 `uxr_create_output_reliable_stream` (신뢰성 보장) 중에서 선택할 수 있으며, 신뢰성 스트림의 경우 `history` 파라미터를 통해 버퍼 크기를 조절하여 신뢰성과 메모리 사용량 사이의 균형을 맞춥니다.20
3. **DDS 계층 구조 생성**: Participant, Topic, Publisher/Subscriber 순서로 DDS 엔티티 생성을 요청합니다. 각 생성 함수(`uxr_buffer_create_...`)는 요청을 추적하기 위한 ID를 반환합니다.20
4. **DataWriter 및 DataReader 인스턴스화**: 실제 데이터 송수신을 담당할 DataWriter(Publisher 및 Topic에 연결)와 DataReader(Subscriber 및 Topic에 연결) 생성을 요청합니다.20

### 4.4  발행/구독 패턴 구현

- **Publisher (발행자)**:
  1. `uxr_prepare_output_stream` 함수를 사용하여 데이터를 쓸 출력 스트림을 준비합니다. 이때 DataWriter의 ID와 데이터 크기를 지정합니다.22
  2. `Micro-XRCE-DDS-Gen`이 생성한 `_serialize_topic` 함수를 사용하여 C 구조체 데이터를 버퍼에 직렬화합니다.
  3. `uxr_run_session_...` 함수를 호출하여 버퍼링된 데이터를 에이전트로 전송합니다.20
- **Subscriber (구독자)**:
  1. `uxr_set_topic_callback` 함수를 사용하여 데이터 수신 시 호출될 콜백 함수를 등록합니다.20
  2. `uxr_buffer_request_data` 함수를 사용하여 에이전트에게 데이터 수신을 요청합니다.22
  3. 메인 루프에서 `uxr_run_session_...` 함수를 주기적으로 호출하면, 데이터가 도착했을 때 등록된 `on_topic` 콜백이 실행됩니다. 콜백 함수 내에서는 `_deserialize_topic` 함수를 사용하여 수신된 데이터를 C 구조체로 역직렬화하여 사용합니다.20

### 4.5  요청/응답 패턴 구현

Micro-XRCE-DDS는 DDS-RPC 표준을 통해 요청/응답(Request/Reply) 통신도 지원합니다.2 이를 위해 Requester와 Replier 엔티티를 생성합니다.20 Requester는 요청을 보내고, Replier는 `on_request` 콜백을 통해 요청을 수신하여 처리한 후 응답을 되돌려주는 방식으로 동작합니다.22

### 4.6  에이전트 응답 및 상태 처리

엔티티 생성과 같은 중요한 작업은 비동기적으로 처리될 수 있으므로, 에이전트로부터의 상태 응답을 확인하는 것이 중요합니다. `uxr_run_session_until_all_status` 함수는 여러 생성 요청이 모두 성공적으로 처리되었는지 확인하는 편리한 방법을 제공합니다. 이 함수는 모든 요청에 대한 상태 응답을 받거나 타임아웃이 발생할 때까지 세션을 실행합니다.20

## 5.  micro-ROS와의 공생 관계: 마이크로컨트롤러에서 ROS 2 활성화

### 5.1  micro-ROS의 기본 미들웨어로서의 Micro-XRCE-DDS

Micro-XRCE-DDS의 가장 중요하고 널리 사용되는 실제 적용 사례는 단연 micro-ROS입니다. micro-ROS는 ROS 2를 마이크로컨트롤러로 확장하기 위한 공식 프로젝트로, 기본 통신 미들웨어로서 Micro-XRCE-DDS를 채택하고 있습니다.5 Micro-XRCE-DDS는 micro-ROS의 단순한 구성 요소가 아니라, 그 핵심 기능인 성능, 신뢰성, 그리고 ROS 2 생태계와의 완벽한 통합을 가능하게 하는 근본적인 기술입니다.

### 5.2  micro-ROS 스택 분석: `rclc`에서 통신망까지

micro-ROS는 ROS 2의 아키텍처를 따르면서 마이크로컨트롤러 환경에 최적화된 계층 구조를 가집니다.18

- **애플리케이션 계층**: 개발자는 `rclc` (ROS Client Library for C) API를 사용하여 사용자 코드를 작성합니다.
- **클라이언트 라이브러리 계층**: `rclc`는 `rcl` (ROS Client Library) 위에 구축되어 편리한 함수와 실행 관리자(Executor)를 제공합니다.18
- **미들웨어 인터페이스 (RMW)**: ROS 2의 핵심인 RMW(ROS Middleware Interface) 계층에는 `rmw_microxrcedds`라는 micro-ROS 전용 구현체가 사용됩니다.5
- **미들웨어 계층**: `rmw_microxrcedds`는 내부적으로 eProsima Micro-XRCE-DDS Client 라이브러리를 호출합니다.
- **전송 계층**: 최종적으로 UDP, 직렬 통신 등의 물리적인 전송 방식을 통해 데이터가 전송됩니다.

### 5.3  ROS 2 개념이 XRCE 오퍼레이션으로 매핑되는 방식

micro-ROS는 ROS 2의 추상화된 API 호출을 내부적으로 XRCE 오퍼레이션으로 변환합니다. 예를 들어, 개발자가 `rclc_publisher_init_default` 함수를 호출하면, `rmw_microxrcedds` 계층은 이를 `uxr_buffer_create_publisher_xml`, `uxr_buffer_create_datawriter_xml`과 같은 일련의 Micro-XRCE-DDS 오퍼레이션으로 변환하여 에이전트에게 전송합니다. 마찬가지로 `rcl_publish` 호출은 `Write Data` 오퍼레이션으로 매핑됩니다. 이처럼 micro-ROS는 친숙한 ROS 2 API 뒤에서 복잡한 XRCE 프로토콜 처리를 완벽하게 추상화하여 개발자가 저수준 통신에 신경 쓰지 않도록 합니다.

### 5.4  실시간 성능을 위한 메모리 관리

micro-ROS 스택의 핵심 장점 중 하나는 실시간 애플리케이션에 적합하도록 런타임 중 동적 메모리 할당을 회피하는 것입니다.18

`rmw_microxrcedds` 계층은 노드, 퍼블리셔, 구독자와 같은 모든 ROS 엔티티를 위한 정적 메모리 풀을 사용합니다. 이 풀의 크기는 `RMW_UXRCE_MAX_NODES`, `RMW_UXRCE_MAX_PUBLISHERS`와 같은 CMake 플래그를 통해 컴파일 시점에 미리 설정됩니다.35 이는 시스템의 메모리 사용량을 예측 가능하게 하고, 런타임 중 예기치 않은 지연을 방지합니다.

### 5.5  비교 분석: `rosserial`을 뛰어넘는 micro-ROS와 XRCE-DDS

`rosserial`은 ROS 1에서 마이크로컨트롤러를 연결하기 위해 사용되었던 솔루션이지만, 비표준 커스텀 프로토콜에 기반하고 기능적 한계가 명확했습니다.9 Micro-XRCE-DDS를 기반으로 하는 micro-ROS는 여러 면에서 

`rosserial`을 월등히 뛰어넘는 차세대 솔루션입니다.

- **표준 프로토콜**: OMG의 DDS-XRCE라는 산업 표준을 따르므로 상호 운용성과 신뢰성이 높습니다.9
- **전송 유연성**: 직렬 통신에 국한되지 않고 UDP, TCP 등 다양한 전송 방식을 지원합니다.9
- **QoS 지원**: 신뢰성(Reliability)과 같은 DDS의 풍부한 QoS 개념을 상속받아 통신의 질을 제어할 수 있습니다.9
- **효율성 및 안정성**: HDLC 프레이밍과 CRC를 통한 무결성 검사 등 더 견고한 통신 메커니즘을 제공하며, 메모리 사용도 더 효율적입니다.9
- **API 통합**: 표준 `rcl` 기반으로 구축되어 ROS 2와의 일관된 개발 경험을 제공합니다.32

결론적으로, Micro-XRCE-DDS의 채택은 micro-ROS가 `rosserial`과 같은 단순한 브릿지가 아니라, ROS 2 생태계의 '진정한 일원'으로 자리매김하게 한 결정적인 요인입니다.

## 6.  비교 프로토콜 분석: IoT 환경에서의 Micro-XRCE-DDS

### 6.1  아키텍처 이분법: 브로커 기반 vs. 에이전트-프록시

- **MQTT/MQTT-SN**: 모든 메시지를 중개하는 중앙 집중형 '브로커'에 의존합니다. 클라이언트는 브로커에 연결하여 토픽을 발행하거나 구독하며, 브로커가 메시지 라우팅을 전담합니다.7
- **Micro-XRCE-DDS**: '에이전트'를 사용하지만, 이는 단순한 메시지 라우터를 넘어선 '프록시' 역할을 합니다. 에이전트는 클라이언트를 '대신하여' DDS 네트워크에 능동적으로 참여하며, 데이터 중심 세계로의 직접적인 다리를 놓습니다.1

### 6.2  데이터 모델링: 불투명한 페이로드 vs. 강력한 타입의 IDL

- **MQTT/CoAP**: 메시지 페이로드(payload)는 의미를 알 수 없는 데이터 덩어리(opaque blob)입니다. 데이터의 구조와 의미는 발행자와 구독자 간에 사전에 약속되어야 합니다.7
- **Micro-XRCE-DDS**: OMG의 IDL을 사용하여 데이터 구조를 공식적으로 정의합니다. 이는 강력한 타입 검사(strong typing)를 가능하게 하고, 직렬화 코드를 자동으로 생성하며, 모호함 없이 상호 운용성을 보장합니다.5

### 6.3  서비스 품질(QoS): 세분화된 비교

- **MQTT**: 클라이언트와 브로커 간의 메시지 전달을 보장하기 위해 3단계의 QoS(0, 1, 2)를 제공합니다.11
- **CoAP**: UDP 위에서 Confirmable 메시지를 통해 기본적인 신뢰성 있는 메시징을 제공합니다.36
- **Micro-XRCE-DDS**: 최선 노력(Best-Effort) 및 신뢰성(Reliable) 스트림 모드를 지원합니다.5 여기서 결정적인 차이점은, 에이전트가 완전한 DDS 참여자이기 때문에 클라이언트가 에이전트 측의 설정을 통해 간접적으로 DDS의 풍부한 QoS 정책(예: Durability, Deadline, Lifespan 등)을 활용할 수 있다는 점입니다.5 이는 실시간 시스템에서 매우 중요한 장점입니다.

### 6.4  오버헤드 및 효율성 분석

- **헤더 크기**: DDS-XRCE는 12-16 바이트의 고정적인 헤더 오버헤드를 가집니다. 반면 MQTT는 최소 5바이트로 시작하지만, 토픽 문자열의 길이에 따라 오버헤드가 증가하여, 긴 토픽 이름을 사용할 경우 XRCE의 오버헤드를 초과할 수 있습니다.11
- **자원 사용량**: 일부 연구에 따르면 특정 임베디드 환경에서 DDS 미들웨어가 MQTT 설정보다 메모리 및 전력 소비가 더 적을 수 있음이 나타났습니다.39 Micro-XRCE-DDS는 특히 저자원 소모를 목표로 설계되었습니다.5
- **성능 (지연 시간/처리량)**: DDS는 고성능, 저지연 실시간 시스템을 위해 설계된 아키텍처입니다.7 반면 MQTT는 저대역폭, 고지연, 비신뢰성 네트워크에 최적화되어 있습니다.7 DDS-XRCE에 대한 성능 평가는 다양한 조건 하에서 지연 시간과 처리량을 중심으로 이루어졌으며, IoT 애플리케이션에 적합하다는 결론을 내렸습니다.40

### 6.5 표 2: 프로토콜 상세 비교: Micro-XRCE-DDS vs. MQTT vs. CoAP

| 기능               | Micro-XRCE-DDS                                               | MQTT                               | CoAP                                      |
| ------------------ | ------------------------------------------------------------ | ---------------------------------- | ----------------------------------------- |
| **아키텍처**       | 클라이언트-에이전트 (프록시)                                 | 클라이언트-서버 (브로커)           | 클라이언트-서버 (요청/응답)               |
| **주요 사용 사례** | 실시간 로보틱스, IIoT, 자율 시스템                           | 경량 IoT 메시징, 원격 모니터링     | 제약된 노드 간의 웹 통신                  |
| **데이터 모델**    | 강력한 타입 (OMG IDL)                                        | 불투명한 페이로드 (Opaque Payload) | 불투명한 페이로드 (Opaque Payload)        |
| **QoS**            | Best-Effort, Reliable 스트림. 에이전트를 통해 풍부한 DDS QoS 활용 가능 | QoS 0, 1, 2 (메시지 전달 보장)     | Confirmable/Non-confirmable (기본 신뢰성) |
| **전송**           | 전송 계층에 독립적 (UDP, TCP, Serial, Custom 등 지원)        | 주로 TCP에 의존                    | 주로 UDP에 의존                           |
| **보안**           | DDS-Security 표준 연동 가능                                  | 사용자 정의 (TLS, 인증 등)         | DTLS                                      |
| **Discovery**      | 에이전트 Discovery 기능 지원                                 | 브로커 주소를 알아야 함            | 리소스 Discovery 기능 지원                |
| **오버헤드**       | 고정적 (12-16 바이트)                                        | 가변적 (토픽 길이에 따라 증가)     | 매우 낮음 (4바이트 고정 헤더)             |
| **확장성 모델**    | 데이터 중심의 분산 확장성 (에이전트를 통해)                  | 중앙 브로커의 성능에 의존          | 점대점(Point-to-point) 또는 그룹 통신     |

제 7장: 고급 개념 및 성능 튜닝

### 6.6  신뢰성 보장: 세션 관리와 스트림

- **세션 상태 유지**: 에이전트는 클라이언트의 세션을 유지 관리합니다. 간헐적인 연결 끊김 후 동일한 `client_key`로 클라이언트가 재연결하면, 이전에 에이전트 상에 생성된 엔티티들을 그대로 재사용할 수 있습니다. 이는 절전 모드를 사용하는 장치에 매우 중요합니다.1

- **신뢰성 스트림**: 신뢰성 있는 통신은 클라이언트와 에이전트 양측의 히스토리 버퍼(슬라이딩 윈도우)를 통해 구현됩니다. 이는 UDP와 같은 비신뢰성 전송 위에서도 메시지의 순서와 전달을 보장합니다.21

  `history` 크기는 메모리 사용량과 통신 복원력 사이의 균형을 맞추는 핵심 튜닝 파라미터입니다.30

- **하트비트 (Heartbeats)**: 에이전트와 클라이언트는 주기적으로 하트비트 메시지를 교환하여 세션의 활성 상태를 확인하고 신뢰성 프로토콜의 상태를 관리합니다.23

### 6.7  플러그형 전송 계층: 커스텀 전송 구현

Micro-XRCE-DDS는 플랫폼별 송수신 함수를 추상화하는 전송 API를 제공하여 사용자가 직접 전송 계층을 구현할 수 있도록 합니다.23 커스텀 전송은 두 가지 모드로 구현할 수 있습니다.

- **패킷 지향 (Packet-oriented)**: UDP처럼 메시지 단위 전송이 가능한 통신 방식에 사용됩니다.
- **스트림 지향 (Stream-oriented)**: 직렬 통신이나 SPI처럼 데이터가 연속적인 스트림으로 전달되는 경우 사용되며, 이 경우 내장된 HDLC 프레이밍이 자동으로 적용됩니다.23

이 기능은 개발자가 Micro-XRCE-DDS를 거의 모든 하드웨어 통신 인터페이스에 적용할 수 있도록 하는 강력한 확장성을 제공합니다.

### 6.8  엔티티 생성 전략: XML vs. 참조(Reference)

- **XML에 의한 생성**: 클라이언트가 엔티티를 정의하는 XML 문자열을 에이전트로 보내는 기본 방식입니다. 유연하지만, 클라이언트 측에서 XML 문자열을 저장하기 위한 메모리와 이를 전송하기 위한 대역폭을 소모합니다.5
- **참조에 의한 생성**: 고도로 최적화된 방식입니다. 필요한 엔티티들을 에이전트 측에 미리 파일 등으로 설정해두고, 클라이언트는 단지 짧은 참조 레이블(예: "my_imu_publisher")만 전송하여 엔티티를 생성합니다. 이는 클라이언트의 메모리와 대역폭을 크게 절약해주므로, 극도로 자원이 제한된 장치에 이상적입니다.5

### 6.9  자원 제약 환경을 위한 튜닝: 컴파일 프로파일 활용

클라이언트 라이브러리는 CMake 플래그(`-DUCLIENT_PROFILE_...`)를 통해 컴파일 시점에 고도로 설정 가능합니다.4 개발자는 애플리케이션에 필요 없는 전송 방식(예: 직렬 통신만 사용 시 TCP 지원 제거), Discovery 기능 등을 비활성화하여 최종 바이너리의 크기를 크게 줄일 수 있습니다. 이는 애플리케이션의 요구사항에 정확히 맞춰 라이브러리를 재단하는 것을 가능하게 합니다.5

'참조에 의한 생성'과 '컴파일 프로파일'의 조합은 극단적인 최적화를 위한 핵심 설계 철학을 보여줍니다. 이는 복잡성을 자원이 부족한 클라이언트에서 자원이 풍부한 개발/배포 단계로 옮기는 전략입니다. 고정된 기능을 가진 임베디드 장치의 경우, 필요한 모든 DDS 엔티티와 QoS 설정을 설계 시점에 미리 에이전트에 정의하고, 클라이언트는 최소한의 참조 정보와 기능만을 탑재한 채로 컴파일될 수 있습니다. 이는 '극도로 자원 제약이 있는 환경'이라는 목표를 달성하는 가장 효과적인 방법론입니다.

## 7.  실제 적용 사례 및 커뮤니티 자원

### 7.1  항공우주 및 드론: PX4 및 ArduPilot과의 통합

Micro-XRCE-DDS(이전 명칭: micro-RTPS)는 PX4 자동 비행 조종 장치를 ROS 2 / DDS 네트워크에 연결하는 표준 인터페이스입니다.10 차량 상태, 주행 기록계와 같은 텔레메트리를 스트리밍하고, 오프보드(offboard) 제어 명령을 전송하는 데 사용됩니다.25 ArduPilot 커뮤니티 또한 Micro-XRCE-DDS 통합을 활발히 진행하고 있습니다.28 이러한 주요 오픈소스 자동 비행 프로젝트들의 채택은 Micro-XRCE-DDS의 안정성과 중요성을 강력하게 입증합니다.

### 7.2  로보틱스: micro-ROS 생태계

수천 개의 마이크로컨트롤러 기반 장치들이 micro-ROS를 사용하며, 이는 곧 Micro-XRCE-DDS를 사용한다는 의미입니다.33 상용 로봇인 iRobot Roomba(상위 레벨에서 ROS 2 및 Fast DDS 사용)나 저수준 부품에 micro-ROS를 사용하는 Husarion ROSbot과 같은 오픈소스 플랫폼이 대표적인 예입니다.33 이러한 생태계 중심의 통합은 기술의 성숙도를 높이고 개발을 촉진하는 긍정적인 피드백 루프를 형성합니다.

### 7.3  산업용 IoT(IIoT) 및 센서 네트워크

이 프로토콜은 간헐적인 연결성을 가지며 전력 효율이 중요한 센서 네트워크를 위해 명시적으로 설계되었습니다.1 엔터프라이즈 DDS 시스템과 연결하는 능력 덕분에, 저수준 센서들을 대규모 산업 모니터링 및 제어 시스템에 통합하는 데 적합합니다.33

### 7.4  커뮤니티 및 지원 채널

- **공식 문서**: Read the Docs에서 호스팅되며, 매뉴얼과 API 레퍼런스를 제공합니다.2
- **GitHub 저장소**: 소스 코드, 이슈 트래킹, 풀 리퀘스트를 위한 기본 채널입니다. 주요 저장소로는 `Micro-XRCE-DDS`, `Micro-XRCE-DDS-Agent`, `Micro-XRCE-DDS-Client`, `Micro-XRCE-DDS-Gen`이 있습니다.2
- **상업적 지원**: eProsima는 기업 사용자를 위해 이메일(info@eprosima.com, support@eprosima.com)을 통한 상업적 지원을 제공합니다.4
- **커뮤니티 포럼**: 전용 Slack 채널은 없지만, ROS Discourse 포럼(특히 micro-ROS, PX4/ArduPilot 관련)과 ROS 2 임베디드 워킹 그룹 회의 등을 통해 커뮤니티 토론이 활발하게 이루어집니다.28

## 8.  결론

eProsima Micro-XRCE-DDS는 단순한 통신 라이브러리를 넘어, 자원이 극도로 제한된 임베디드 장치를 표준화되고 강력한 DDS 및 ROS 2 데이터 중심 세계로 통합하기 위한 포괄적인 솔루션입니다. 본 보고서에서 심층적으로 분석한 바와 같이, Micro-XRCE-DDS의 핵심적인 성공 요인은 그 아키텍처 자체에 있습니다.

**클라이언트-에이전트 패러다임**은 표준 DDS의 높은 자원 요구량과 마이크로컨트롤러의 제약이라는 근본적인 충돌을 해결하는 독창적인 해법입니다. 에이전트가 클라이언트의 프록시 역할을 수행하며 DDS 통신의 모든 복잡성을 위임받음으로써, 클라이언트는 동적 메모리 할당 없이 극도로 가볍고 예측 가능하게 동작할 수 있습니다. 이는 실시간 임베디드 시스템에서 필수적인 요구사항을 충족시킵니다.

또한, Micro-XRCE-DDS는 **OMG 표준**에 기반하여 기술적 신뢰성과 상호 운용성을 보장하며, **플러그형 전송 계층**과 **고도로 설정 가능한 컴파일 프로파일**을 통해 거의 모든 하드웨어와 애플리케이션 요구사항에 맞춰 최적화될 수 있는 유연성을 제공합니다.

특히 **micro-ROS의 기본 미들웨어**로서의 역할은 이 기술의 중요성을 단적으로 보여줍니다. Micro-XRCE-DDS는 micro-ROS가 `rosserial`을 뛰어넘어, QoS, 신뢰성, 다양한 전송 방식을 지원하는 진정한 ROS 2의 확장으로 자리매김할 수 있게 한 기반 기술입니다.

결론적으로, Micro-XRCE-DDS는 로보틱스, 드론, IIoT 분야에서 저수준 센서와 액추에이터를 상위 시스템과 안정적이고 효율적으로 연결하고자 하는 개발자들에게 가장 강력하고 표준화된 도구 중 하나입니다. 이 기술에 대한 깊이 있는 이해는 차세대 분산 임베디드 시스템을 설계하고 구현하는 데 있어 필수적인 역량이 될 것입니다.

#### **참고 자료**

1. Announcing the DDS-XRCE Specification: A Protocol for Sensor Networks, accessed July 6, 2025, https://www.rti.com/blog/announcing-the-dds-xrce-specification-a-protocol-for-sensor-networks
2. GitHub - eProsima/Micro-XRCE-DDS-Agent, accessed July 6, 2025, https://github.com/eProsima/Micro-XRCE-DDS-Agent
3. Overview - eProsima Micro XRCE-DDS 3.0.1 documentation, accessed July 6, 2025, https://micro-xrce-dds.docs.eprosima.com/en/latest/introduction.html
4. eProsima/Micro-XRCE-DDS - GitHub, accessed July 6, 2025, https://github.com/eProsima/Micro-XRCE-DDS
5. Micro XRCE-DDS, accessed July 6, 2025, https://micro.ros.org/docs/concepts/middleware/Micro_XRCE-DDS/
6. eProsima/Micro-XRCE-DDS-docs - GitHub, accessed July 6, 2025, https://github.com/eProsima/Micro-XRCE-DDS-docs
7. Navigating IIoT Protocols: Comparing DDS and MQTT - RTI, accessed July 6, 2025, https://www.rti.com/blog/comparing-dds-and-mqtt
8. DDS – The Proven Data Connectivity Standard for IoT TM - Twin Oaks Computing, Inc, accessed July 6, 2025, https://www.twinoakscomputing.com/datasheets/DDS-Brochure.pdf
9. Micro XRCE-DDS compared to rosserial, accessed July 6, 2025, https://micro.ros.org/docs/concepts/middleware/rosserial/
10. Bringing Micro XRCE-DDS & micro-ROS to PX4-based flying systems - PX4 Autopilot, accessed July 6, 2025, https://px4.io/wp-content/uploads/2020/07/microROS-presentation-PX4DevSummit-2020.pdf
11. DDS-XRCE, MQTT & IoT | micro-ROS, accessed July 6, 2025, https://micro.ros.org/docs/concepts/middleware/IoT/
12. Intro to DDS (Data Distribution Service), accessed July 6, 2025, https://www.ics.uci.edu/~cs237/guestspeakers2022/DDS-Sp22.pdf
13. [Micro-ROS] Micro XRCE-DDS 특징 - Interactics - 티스토리, accessed July 6, 2025, [https://huroint.tistory.com/entry/Micro-ROS-Micro-XRCE-DDS-%ED%8A%B9%EC%A7%95](https://huroint.tistory.com/entry/Micro-ROS-Micro-XRCE-DDS-특징)
14. eProsima/Micro-XRCE-DDS-Client - GitHub, accessed July 6, 2025, https://github.com/eProsima/Micro-XRCE-DDS-Client
15. eProsima Micro XRCE-DDS Agent, accessed July 6, 2025, https://micro-xrce-dds.docs.eprosima.com/en/latest/agent.html
16. Overview - eProsima Micro XRCE-DDS 2.1.1 documentation, accessed July 6, 2025, https://micro-xrce-dds.docs.eprosima.com/en/stable/introduction.html
17. Install Micro XRCE-DDS Agent on Linux | Snap Store - Snapcraft, accessed July 6, 2025, https://snapcraft.io/micro-xrce-dds-agent
18. Features and Architecture - micro-ROS, accessed July 6, 2025, https://micro.ros.org/docs/overview/features/
19. 6. micro-ROS Documentation - Vulcanexus 1.0.0 documentation, accessed July 6, 2025, https://docs.vulcanexus.org/en/iron/rst/microros_documentation/
20. Getting started - eProsima Micro XRCE-DDS 3.0.1 documentation, accessed July 6, 2025, https://micro-xrce-dds.docs.eprosima.com/en/latest/getting_started.html
21. Client API - eProsima Micro XRCE-DDS 3.0.1 documentation, accessed July 6, 2025, https://micro-xrce-dds.docs.eprosima.com/en/latest/client_api.html
22. Quick start - eProsima Micro XRCE-DDS, accessed July 6, 2025, https://micro-xrce-dds.docs.eprosima.com/en/latest/quickstart.html
23. Transport - eProsima Micro XRCE-DDS 3.0.1 documentation, accessed July 6, 2025, https://micro-xrce-dds.docs.eprosima.com/en/latest/transport.html
24. Micro XRCE-DDS Agent Installation Problem : r/ROS - Reddit, accessed July 6, 2025, https://www.reddit.com/r/ROS/comments/1l1flxh/micro_xrcedds_agent_installation_problem/
25. uXRCE-DDS (PX4-ROS 2/DDS Bridge) | PX4 Guide (main), accessed July 6, 2025, https://docs.px4.io/main/en/middleware/uxrce_dds.html
26. Installation - eProsima Micro XRCE-DDS 2.1.1 documentation, accessed July 6, 2025, https://micro-xrce-dds.docs.eprosima.com/en/v2.2.0/installation.html
27. uXRCE-DDS (PX4-ROS 2/DDS Bridge) | PX4 User Guide (v1.14) - PX4 docs, accessed July 6, 2025, https://docs.px4.io/v1.14/en/middleware/uxrce_dds.html
28. MicroXRCE DDS on Ardupilot - Embedded - ROS Discourse, accessed July 6, 2025, https://discourse.ros.org/t/microxrce-dds-on-ardupilot/29972
29. eProsima/Micro-XRCE-DDS-Gen - GitHub, accessed July 6, 2025, https://github.com/eProsima/Micro-XRCE-DDS-Gen
30. eProsima Micro XRCE-DDS Client - GitHub, accessed July 6, 2025, https://github.com/eProsima/Micro-XRCE-DDS-docs/blob/master/docs/client.rst
31. eProsima Shapes Demo with Micro XRCE-DDS - YouTube, accessed July 6, 2025, https://www.youtube.com/watch?v=Twc9VENhlxk
32. [Micro-ROS] 특징 - Interactics - 티스토리, accessed July 6, 2025, [https://huroint.tistory.com/entry/Micro-ROS-%ED%8A%B9%EC%A7%95](https://huroint.tistory.com/entry/Micro-ROS-특징)
33. Internet of Things - eProsima, accessed July 6, 2025, https://www.eprosima.com/middleware/sectors/internet-of-things
34. micro-ROS on FreeRTOS, accessed July 6, 2025, https://www.freertos.org/Community/Blogs/2020/micro-ros-on-freertos
35. Middleware Configuration - micro-ROS, accessed July 6, 2025, https://micro.ros.org/docs/tutorials/advanced/microxrcedds_rmw_configuration/
36. CoAP, MQTT, AMQP, XMPP & DDS: Which Protocol Should You Choose for IoT? - NexPCB, accessed July 6, 2025, https://www.nexpcb.com/blog/different-data-protocols-which-one-to-choose
37. How Does DDS Compare to other IoT Technologies? - DDS Foundation, accessed July 6, 2025, https://www.dds-foundation.org/features-benefits/
38. MicroXRCE-DDS Documentation, accessed July 6, 2025, https://micro-xrce-dds.docs.eprosima.com/_/downloads/en/latest/pdf/
39. data sharing using mqtt and zigbee-based dds on - Middle East Technical University, accessed July 6, 2025, https://etd.lib.metu.edu.tr/upload/12625183/index.pdf
40. CoAP vs. MQTT-SN: Comparison and Performance Evaluation in Publish-Subscribe Environments | Request PDF - ResearchGate, accessed July 6, 2025, https://www.researchgate.net/publication/356076177_CoAP_vs_MQTT-SN_Comparison_and_Performance_Evaluation_in_Publish-Subscribe_Environments
41. (PDF) DDS-XRCE Standard Performance Evaluation of Different Communication Scenarios in IoT Technologies - ResearchGate, accessed July 6, 2025, https://www.researchgate.net/publication/365702304_DDS-XRCE_Standard_Performance_Evaluation_of_Different_Communication_Scenarios_in_IoT_Technologies
42. Micro XRCE-DDS - Auterion Documentation, accessed July 6, 2025, https://docs.auterion.com/hardware-integration/flight-controller-customization/micro-xrce-dds
43. eProsima Fast DDS - The middleware powering the ESO ELT, accessed July 6, 2025, https://www.eso.org/sci/meetings/2023/RTC4AO/01_11_martin_losa.pdf
44. Micro-XRCE-DDS-Agent/README.md at master - GitHub, accessed July 6, 2025, https://github.com/eProsima/Micro-XRCE-DDS-Agent/blob/master/README.md
45. Issues / eProsima/Micro-XRCE-DDS-Client - GitHub, accessed July 6, 2025, https://github.com/eProsima/Micro-XRCE-DDS-Client/issues
46. eProsima Micro XRCE-DDS, accessed July 6, 2025, https://micro-xrce-dds.docs.eprosima.com/
47. eProsima Micro XRCE-DDS - YouTube, accessed July 6, 2025, https://www.youtube.com/playlist?list=PL-Kh3H15FsNQ5ksTjp-WutVnPoOq6iyoM


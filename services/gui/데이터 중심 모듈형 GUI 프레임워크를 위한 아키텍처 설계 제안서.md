# 데이터 중심 모듈형 GUI 프레임워크를 위한 아키텍처 설계 제안서



------



## 1.  기본 원칙 및 선행 기술 분석



본 설계 제안서의 목표는 데이터 분산 서비스(DDS)와 작은 실행 단위(small execution unit)라는 두 가지 핵심 요소를 기반으로 하는 새로운 GUI 프레임워크의 아키텍처를 설계하는 것입니다. 이 프레임워크는 ROS2(Robot Operating System 2)와 유사한 철학을 공유하지만, GUI 애플리케이션의 고유한 요구사항에 최적화된 독자적인 구조를 가집니다. 제 1부에서는 이러한 아키텍처를 구성하는 기본 원칙을 확립하고, DDS, ROS2, 그리고 마이크로 프론트엔드와 같은 선행 기술들을 심층적으로 분석하여 우리가 나아갈 방향의 이론적 토대를 마련합니다.



### 1.1  데이터 중심 패러다임: 시스템 전역 데이터 버스로서의 DDS



새로운 프레임워크의 근간은 데이터 분산 서비스(DDS)입니다. DDS는 단순한 메시징 프로토콜을 넘어, 분산 시스템 전체에 '글로벌 데이터 공간(global data space)'을 형성하는 패러다임 전환적 기술입니다.1 이 패러다임의 핵심은 데이터 자체가 애플리케이션 간의 인터페이스 역할을 한다는 것입니다.1 이러한 접근 방식은 분산 애플리케이션, 특히 UI의 설계 방식을 근본적으로 변화시킵니다. DDS의 핵심 구성 요소는 발행-구독(Publish-Subscribe) 모델, 중앙 브로커가 없는 탈중앙화된 검색(decentralized discovery), 그리고 시스템의 비기능적 속성을 제어하는 서비스 품질(QoS, Quality of Service) 정책입니다.1



#### 1.1.1  DDS와 타 통신 기술 비교 분석



본 아키텍처에서 DDS가 갖는 독보적인 가치를 명확히 하기 위해, 다른 일반적인 통신 기술들과 비교 분석합니다.

- **gRPC (gRPC Remote Procedure Call)**: gRPC는 고성능 원격 프로시저 호출(RPC) 프레임워크입니다.4 이는 본질적으로 클라이언트-서버 구조를 따르는 명령 지향적(command-oriented) 프로토콜로, DDS의 익명적 발행-구독 모델보다 더 강한 결합(tighter coupling)을 형성합니다. 서비스 간의 명시적 호출에는 뛰어나지만, UI를 위한 실시간 공유 상태 공간을 만드는 데는 적합하지 않습니다.
- **웹소켓 (WebSockets)**: 웹소켓은 단일 TCP 연결을 통해 전이중(full-duplex) 통신을 제공하는 프로토콜입니다.6 실시간 양방향 통신을 제공하지만, 근본적으로는 점대점(point-to-point) 파이프에 가깝습니다. DDS가 제공하는 내용 기반 필터링(content filtering), 시간 기반 필터링(time-based filtering)과 같은 풍부한 데이터 중심 기능과 포괄적인 QoS 시스템이 부재합니다.



#### 1.1.2  UI의 재정의: 클라이언트가 아닌 동료(Peer)로서의 UI



전통적인 UI 아키텍처는 gRPC나 웹소켓을 사용하더라도 암묵적으로 클라이언트-서버 모델을 따릅니다. UI는 '백엔드'로부터 데이터를 요청하는 '클라이언트'로 간주됩니다.4 그러나 DDS의 탈중앙화된 브로커리스 아키텍처는 이러한 모델을 근본적으로 파괴합니다.1 DDS를 사용하는 애플리케이션은 클라이언트나 서버가 아닌, 분산 데이터 공간에 참여하는 하나의 '동료(peer)'입니다.

따라서 DDS를 기반으로 구축된 GUI 프레임워크는 더 이상 클라이언트로 설계되어서는 안 됩니다. 이는 글로벌 데이터 공간에 직접 참여하는 그래픽 표현을 가진 동료 컴포넌트들의 집합으로 설계되어야 합니다. 이러한 관점의 전환은 전체 아키텍처, 특히 상태 관리 방식에 지대한 영향을 미치며, 이는 제 2부에서 상세히 논의될 것입니다.



### 1.2  ROS2 모델: 통합의 사례 연구



ROS2는 DDS를 성공적으로 활용한 대표적인 로봇 프레임워크입니다.1 ROS2가 어떻게 DDS를 통합하고 활용하는지 분석하는 것은 우리 프레임워크 설계에 중요한 시사점을 제공합니다. 핵심은 ROS 미들웨어 인터페이스(RMW, ROS Middleware Interface)로, 이는 ROS2가 eProsima Fast DDS, RTI Connext, Eclipse Cyclone DDS 등 다양한 DDS 벤더를 지원할 수 있도록 하는 중요한 추상화 계층입니다.8



#### 1.2.1  ROS 개념의 DDS 매핑



ROS2는 자체 개념을 DDS 엔티티에 매핑하는 구체적인 규칙을 사용합니다. 예를 들어, ROS 노드(Node)는 DDS 참여자(Participant)에, ROS 토픽(Topic)은 DDS 토픽에 매핑됩니다.8 여기서 주목할 점은 ROS2가 사용하는 이름 변환(name-mangling) 체계입니다. ROS 토픽은 

`rt/`, 서비스는 `rq/`와 `rr/` 같은 접두사가 붙어 DDS 토픽으로 변환됩니다.11 이 방식은 ROS 시스템과 비-ROS DDS 시스템 간의 공존을 가능하게 하지만, 동시에 직접적인 통신에 장벽을 만들기도 합니다.



#### 1.2.2  ROS2 GUI 도구 분석



- **rqt**: `rqt`는 Qt 기반의 플러그인 프레임워크로, 개발자가 Python이나 C++로 쉽게 새로운 플러그인을 만들 수 있다는 유연성이 강점입니다.12 하지만 모든 플러그인이 단일 프로세스 내에서 실행되는 구조는 결함 격리(fault isolation) 측면에서 약점을 보입니다.13

- **RViz2**: `RViz2`는 3D 시각화에 특화된 도구로, 역시 플러그인 아키텍처를 기반으로 합니다.15 RViz의 각 '디스플레이(Display)'는 

  `LaserScan`이나 `PointCloud2`와 같은 특정 ROS2 토픽을 구독하고 이를 3D 뷰에 렌더링하는 플러그인입니다.16 데이터와 시각화의 직접적인 바인딩이 그 강력함의 원천입니다.



#### 1.2.3  추상화와 기능 활용의 트레이드오프



ROS2의 RMW 인터페이스는 벤더 중립성이라는 중요한 목표를 달성하기 위해 설계되었습니다.8 이를 위해 RMW API는 모든 지원 DDS 구현체에서 공통적으로 사용할 수 있는 최소 공통분모(least common denominator) 기능만을 노출합니다. 그러나 DDS는 22개 이상의 QoS 정책과 다양한 벤더별 확장 기능을 갖춘 방대한 표준입니다.17 DDS의 진정한 힘은 이러한 정책들을 섬세하게 적용하는 데서 나옵니다.

RMW 추상화 계층은 필연적으로 이러한 미묘한 기능 대부분을 일반 ROS2 개발자로부터 숨기고, 단순화된 QoS 설정의 일부만을 노출합니다.8 "DDS + GUI"를 지향하는 프레임워크가 단순히 ROS2의 RMW 접근 방식을 모방한다면, 이는 GUI 컴포넌트가 강력한 DDS 기능을 활용하는 것을 막는 실수가 될 것입니다. 예를 들어, 데이터 폭주를 막기 위한 

`TIME_BASED_FILTER`나 일시적인 팝업을 위한 `LIFESPAN`과 같은 QoS 정책은 고성능 UI 구축에 필수적입니다. 따라서 우리가 설계할 프레임워크는 기저의 DDS 미들웨어에 대해 더 직접적이고 덜 추상화된 인터페이스를 제공해야 합니다.



### 1.3  작은 실행 단위 패러다임: 마이크로 프론트엔드로부터의 교훈



사용자가 요구한 "작은 실행 단위"는 마이크로 프론트엔드 아키텍처의 원칙과 직접적으로 연결됩니다.20 이 아키텍처 스타일은 거대한(monolithic) 프론트엔드 애플리케이션을 독립적으로 배포 가능한 더 작은 서비스들로 분해하며, 이는 백엔드의 마이크로서비스 패턴을 반영합니다.21



#### 1.3.1  마이크로 프론트엔드의 상태 관리 도전 과제



마이크로 프론트엔드의 주된 과제는 독립적인 단위들 간의 상태를 관리하고 통신하는 것입니다. 연구 자료들은 다음과 같은 패턴들을 제시합니다.

- **지역 상태 (Local State)**: 각 마이크로 프론트엔드가 자체 상태를 독립적으로 관리합니다. 이는 결합도를 낮추지만 컴포넌트 간 통신을 어렵게 만듭니다.20
- **공유 전역 스토어 (Shared Global Store, 예: Redux)**: 중앙 집중식 스토어는 단일 진실 공급원(single source of truth)을 제공하지만, 모듈성의 목적을 저해하는 강한 결합을 다시 도입할 수 있습니다.21
- **이벤트 버스 / 커스텀 이벤트 (Event Bus / Custom Events)**: 느슨하게 결합된 접근 방식이지만, 시스템이 커짐에 따라 디버깅이 어려워지고 "이벤트 충돌"이 발생할 수 있습니다.23
- **웹 스토리지 / URL (Web Storage / URL)**: 구현은 간단하지만 기능이 제한적이며, 복잡한 상태나 반응형 업데이트에는 적합하지 않습니다.23



#### 1.3.2  이상적인 마이크로 프론트엔드 백엔드로서의 DDS



마이크로 프론트엔드 아키텍트들이 직면하는 문제들(상태 동기화, 서비스 간 통신, 디커플링)은 정확히 DDS가 분산 시스템에서 해결하기 위해 설계된 문제들입니다.21 마이크로 프론트엔드의 '공유 전역 스토어' 패턴은 웹 기술을 사용하여 데이터 중심의 '진실 공급원'을 구축하려는 시도입니다. 반면, DDS는 그 자체가 분산 환경을 위한 네이티브 실시간 '진실 공급원'입니다.1 '이벤트 버스' 패턴은 발행-구독 모델의 임시방편적 구현이지만, DDS는 이 모델의 견고하고 표준화된, 풍부한 기능을 제공합니다.1

더 나아가, DDS의 `DURABILITY`와 같은 QoS 정책은 늦게 참여하는(late-joining) 컴포넌트를 위한 상태 지속성 문제를 기본적으로 해결해 줍니다.25 이는 마이크로 프론트엔드 솔루션이 수동으로 해결해야 하는 복잡한 문제입니다.

결론적으로, 우리는 두 패러다임의 장점만을 취하는 프레임워크를 설계할 수 있습니다. 마이크로 프론트엔드의 구성(composition) 및 격리(isolation) 이점을 채택하면서, 취약한 웹 중심의 통신 및 상태 관리 패턴을 DDS라는 산업 등급의 견고한 기반으로 대체하는 것입니다. 이 종합적인 접근 방식이야말로 제안하는 설계의 핵심 혁신입니다.

------



## 2.  DDS-네이티브 GUI 프레임워크를 위한 아키텍처 설계



제 1부에서 확립된 원칙들을 바탕으로, 새로운 프레임워크의 상세한 설계를 제시합니다. 이 아키텍처는 DDS를 단순한 통신 계층이 아닌, 프레임워크의 핵심 구조와 상태 관리의 근간으로 활용합니다.



### 2.1  핵심 아키텍처: 셸(Shell)과 마이크로-위젯(Micro-Widget)



프레임워크는 두 가지 주요 아키텍처 컴포넌트로 구성됩니다. 이 이원적 모델은 유연성과 견고성을 동시에 확보하기 위해 설계되었습니다.

- **셸 (The Shell)**: 경량 호스트 애플리케이션입니다. 셸의 유일한 책임은 마이크로-위젯을 검색하고, 로드하며, 그 생명주기를 관리하는 것입니다. 셸은 메인 윈도우와 레이아웃 관리자(예: 도킹 프레임워크)를 제공하지만, 애플리케이션별 비즈니스 로직은 전혀 포함하지 않습니다. 즉, 셸은 순수한 '애플리케이션 컨테이너' 역할을 합니다.
- **마이크로-위젯 (The Micro-Widget)**: 프레임워크의 기본이 되는, 자립적이고 독립적으로 배포 가능한 "작은 실행 단위"입니다. 각 마이크로-위젯은 UI의 한 조각(버튼, 플롯, 텍스트 입력, 3D 뷰 등)을 나타내며, 해당 기능에 필요한 모든 로직을 캡슐화합니다.



### 2.2  마이크로-위젯: 캡슐화된 GUI-활성 DDS 노드



마이크로-위젯은 `rqt` 플러그인과 같은 단순한 코드 조각이 아닙니다. 이는 분산 시스템 내에서 일급 시민(first-class citizen)으로 동작하는 독립적인 개체입니다.



#### 2.2.1  전용 DDS 참여자(Participant)



본 아키텍처의 가장 중요한 결정 중 하나는, 각 마이크로-위젯 인스턴스가 자신만의 `DDSDomainParticipant`를 갖는다는 것입니다.2 이는 일반적인 ROS2 모델에서 벗어난 중대한 설계 변경이며, 진정한 격리를 달성하는 핵심 열쇠입니다. 이 설계는 다음과 같은 이점을 제공합니다.

- **세분화된 QoS (Granular QoS)**: 각 위젯은 자신의 특정 기능에 맞춰진 고유한 DDS QoS 설정을 가질 수 있습니다. 예를 들어, 고주파 센서 데이터를 시각화하는 위젯은 `BEST_EFFORT` 신뢰성을, 중요한 명령을 전송하는 버튼 위젯은 `RELIABLE` 신뢰성을 사용할 수 있습니다.
- **세분화된 보안 (Granular Security)**: DDS-Security를 사용하면 각 위젯이 고유한 신원, 접근 제어 규칙, 암호화 설정을 가질 수 있습니다.1 이는 시스템의 보안을 매우 정교하게 제어할 수 있게 합니다.
- **결함 격리 (Fault Isolation)**: 만약 하나의 마이크로-위젯이 충돌하거나 응답 불능 상태에 빠져도, 그 실패는 해당 위젯 내에 국한됩니다. 셸이나 다른 마이크로-위젯에는 영향을 주지 않습니다. 셸은 DDS의 Liveliness QoS 등을 통해 위젯의 실패를 감지하고 재시작을 시도할 수 있습니다.



### 2.3  통신 및 상태: 단일 진실 공급원으로서의 DDS 글로벌 데이터 공간



애플리케이션의 모든 상태는 위젯 내부에 저장되는 것이 아니라, DDS 글로벌 데이터 공간에 존재합니다. UI는 이 데이터 공간을 위한 순수한 *시각화 및 상호작용 계층*의 역할을 합니다. 이는 시스템의 상태가 더 이상 특정 UI 컴포넌트에 종속되지 않음을 의미합니다.



#### 2.3.1  데이터 바인딩 예시



'스로틀 슬라이더' 마이크로-위젯의 동작 방식을 통해 이 개념을 구체화할 수 있습니다.

1. 슬라이더 위젯은 내부에 `float value;`와 같은 상태 변수를 가지지 않습니다.
2. 대신, 이 위젯은 `vehicle/throttle_command` 토픽에 대한 `DDSDataWriter`와 `vehicle/throttle_status` 토픽에 대한 `DDSDataReader`를 생성합니다.
3. 사용자가 슬라이더를 조작하면, 위젯의 이벤트 핸들러는 새로운 값을 `throttle_command` 토픽에 발행(`writer.write()`)합니다.
4. 차량의 모터 제어기(별도의 DDS 애플리케이션)는 이 토픽을 구독하여 명령을 수신하고 실행합니다.
5. 모터 제어기는 자신의 실제 스로틀 상태를 `throttle_status` 토픽에 발행합니다.
6. 슬라이더 위젯의 `DataReader`는 이 상태 업데이트를 수신하고, 리스너 콜백 함수는 슬라이더 손잡이의 시각적 위치를 업데이트합니다.

이러한 접근 방식은 시스템 전체를 투명하게 만듭니다. UI의 상태는 RTI Admin Console이나 `ros2 topic echo`와 같은 표준 DDS 도구를 사용하여 모니터링, 디버깅, 심지어 조작까지 가능해집니다. 더 이상 '숨겨진' UI 상태는 존재하지 않습니다.



### 2.4  이벤트 전파: 사용자 상호작용의 DDS 메시지 표준화



사용자 인터페이스에서 발생하는 모든 상호작용(클릭, 키 입력 등)을 표준화된 DDS 메시지로 변환하여 '가상 UI 이벤트 버스'를 구축합니다. 이는 시스템의 유연성과 테스트 용이성을 극대화하는 핵심적인 설계입니다. 이를 위해 OMG IDL(Interface Definition Language)을 사용하여 UI 이벤트를 위한 표준 스키마를 정의합니다.3



#### 2.4.1  제안된 IDL 스키마 (`UIEvents.idl`)



Code snippet

```
// UIEvents 모듈
module UIEvents {
    struct Header {
        string source_widget_id; // 이벤트를 발생시킨 위젯의 고유 ID
        // timestamp 등 추가 정보
    };

    enum Button { NONE, LEFT, MIDDLE, RIGHT };
    enum Action { PRESS, RELEASE, CLICK, MOVE, SCROLL };

    struct MouseEvent {
        Header header;
        Button button;
        Action action;
        float x;
        float y;
        float scroll_delta;
    };

    struct KeyEvent {
        Header header;
        enum KeyAction { PRESS, RELEASE };
        KeyAction action;
        long key_code;
        // modifiers (shift, ctrl, alt)
    };

    // FocusEvent, DragDropEvent 등 추가 가능
};
```



#### 2.4.2  이벤트 흐름 및 파생 효과



마이크로-위젯 내의 버튼에서 클릭이 발생하면, 해당 위젯의 로컬 GUI 이벤트 핸들러가 트리거됩니다. 이 핸들러는 비즈니스 로직을 직접 실행하는 대신, `UIEvents::MouseEvent` 구조체를 채우고 이를 `framework/events/mouse`와 같은 잘 알려진 DDS 토픽에 발행합니다.

이 단순해 보이는 간접적인 방식은 다음과 같은 혁신적인 효과를 가져옵니다.

- **원격 제어 (Remote Control)**: 네트워크상의 어떤 DDS 애플리케이션이든 이벤트 토픽에 메시지를 발행함으로써 UI 요소를 프로그래밍 방식으로 '클릭'할 수 있게 됩니다.
- **테스트 자동화 (Test Automation)**: 테스트 스크립트는 복잡하고 깨지기 쉬운 UI 자동화 프레임워크 대신 간단한 DDS 발행기가 됩니다.
- **기록 및 재현 (Record & Replay)**: 표준 DDS 로깅 도구로 이벤트 스트림을 기록할 수 있습니다. 이 로그 파일을 재현하면 사용자의 GUI 상호작용이 완벽하게 재현되어, 간헐적으로 발생하는 복잡한 문제를 디버깅하는 데 매우 귀중한 도구를 제공합니다. 이는 대부분의 GUI 프레임워크가 제공하지 못하는 강력한 기능입니다.

------



## 3.  구현 전략 및 기술 선택



추상적인 아키텍처 설계를 구체적인 구현 모델로 전환하고, 각 전략의 장단점을 분석하여 실제 프로젝트에 적용할 수 있는 가이드를 제공합니다.



### 3.1  구현 모델 비교 분석



세 가지 주요 구현 전략을 비교하여 각 접근 방식의 핵심적인 트레이드오프를 명확히 합니다. 이 비교는 프로젝트의 요구사항에 가장 적합한 기술 스택을 선택하는 데 도움을 줄 것입니다.

**표 1: GUI 구현 전략 비교**

| 특징              | 전략 1: 통합 프로세스 모델        | 전략 2: 다중 프로세스 모델        | 전략 3: 선언적 UI 모델          |
| ----------------- | --------------------------------- | --------------------------------- | ------------------------------- |
| **핵심 기술**     | Qt / GTK                          | Electron / 네이티브 윈도우 임베딩 | Flutter / 선언적 프레임워크     |
| **프로세스 모델** | 단일 프로세스, 다중 스레드        | 다중 프로세스                     | 단일 프로세스, 다중 스레드      |
| **격리 수준**     | 낮음 (플러그인 충돌 시 셸에 영향) | 높음 (위젯 충돌이 격리됨)         | 낮음 (위젯 오류 시 셸에 영향)   |
| **성능**          | 매우 높음 (공유 메모리 활용 가능) | 중간 (IPC/네트워크 오버헤드)      | 높음 (최적화된 렌더링 엔진)     |
| **자원 사용량**   | 낮음                              | 높음                              | 중간                            |
| **개발 복잡도**   | 중간 (성숙한 툴킷)                | 높음 (IPC/윈도잉 복잡성)          | 중간 (현대적이지만 발전 중)     |
| **주요 장점**     | 최대 성능, 성숙한 생태계          | 최대 복원력 및 유연성             | 최고의 크로스플랫폼 UX, 성능    |
| **주요 단점**     | 낮은 결함 격리 수준               | 높은 자원 오버헤드                | 상대적으로 작은 데스크톱 생태계 |

이 표는 각 전략이 성능, 복원력, 비용(자원/복잡성), 유연성이라는 실제 프로젝트에서 중요한 차원들을 어떻게 만족시키는지를 요약하여 보여줍니다. 이를 통해 기술적 의사결정의 핵심 트레이드오프를 한눈에 파악할 수 있습니다.



### 3.2  전략 1: 통합 프로세스 모델 (예: Qt/GTK 기반)



이 모델은 `rqt` 아키텍처와 가장 유사합니다.12 셸은 Qt 또는 GTK 애플리케이션으로, 공유 라이브러리 파일(

`.so`/`.dll`)에서 마이크로-위젯을 동적으로 로드합니다. 모든 위젯은 동일한 프로세스에서 실행되며 동일한 UI 이벤트 루프를 공유합니다.27

- **구현 세부사항**: Qt의 `QPluginLoader`나 GTK의 GModule과 같은 네이티브 플러그인 메커니즘을 활용합니다. DDS 데이터 수신 스레드와 메인 UI 스레드 간의 통신은 경쟁 상태(race condition)를 피하기 위해 Qt의 시그널과 슬롯 또는 GTK의 `g_idle_add`와 같은 메커니즘을 사용하여 신중하게 관리해야 합니다.29
- **장단점 분석**:
  - **장점**: 동일 머신 내 위젯 간 통신에서 최고의 성능을 발휘할 수 있으며(DDS 공유 메모리 전송 사용 가능), 메모리 사용량이 가장 낮고, 성숙한 위젯 생태계에 접근할 수 있습니다.
  - **단점**: 하나의 위젯이 오작동하거나 충돌하면 전체 애플리케이션이 다운됩니다. 한 위젯에서 장시간 연산을 수행하면 전체 UI가 멈추게 됩니다.31
- **고주파 데이터 처리**: 이 모델의 핵심 과제는 고주파 DDS 데이터가 UI 스레드를 잠식하는 것을 방지하는 것입니다. 해결책은 DDS QoS 정책을 필터로 사용하는 것입니다. 예를 들어, 1kHz 데이터 스트림을 시각화하는 마이크로-위젯은 `DataReader`의 `TIME_BASED_FILTER` QoS 정책을 33ms(~30Hz)의 최소 간격으로 설정합니다.25 이렇게 하면 미들웨어 자체가 초과 샘플을 폐기하여 UI 스레드는 관리 가능한 속도로만 데이터를 수신하게 됩니다. 이는 애플리케이션 코드에서 초당 1000개의 샘플을 모두 수신한 후 조절하려는 시도보다 훨씬 효율적인 접근 방식입니다.



### 3.3  전략 2: 다중 프로세스 모델 (예: Electron 또는 네이티브 임베딩)



이 모델에서 셸 프로세스의 주 역할은 각 마이크로-위젯을 위한 자식 프로세스를 시작하고 관리하는 것입니다. 각 자식 프로세스의 UI는 셸의 메인 윈도우에 '재부모화(reparented)'되어 통합된 것처럼 보입니다.

- **구현 변형**:

  1. **Electron 기반**: 셸은 Electron의 `main` 프로세스가 되고, 각 마이크로-위젯은 별도의 `renderer` 프로세스가 됩니다.33 표준 Electron 앱과의 결정적인 차이점은, 프로세스 간 통신(IPC)에 Electron의 내장 IPC 대신 DDS를 사용한다는 것입니다. 이를 통해 웹 기술로 구현된 위젯이 비-웹 기반의 전체 분산 시스템과 원활하게 통합될 수 있습니다.35

  2. **네이티브 윈도우 임베딩**: 셸은 C++/Qt 등으로 작성된 네이티브 애플리케이션입니다. 셸은 다른 프로세스(어떤 언어나 툴킷으로 작성되었든 무관)를 실행하고, 운영체제 수준의 API를 사용하여 해당 프로세스의 최상위 윈도우를 자신의 윈도우 안에 포함시킵니다. Qt에서는 `QWindow::fromWinId()`와 `createWindowContainer`를 사용하고 37, Windows에서는 

     `SetParent` API를 39, Linux에서는 XEmbed 프로토콜을 활용합니다.40 이는 매우 고급 기술이지만 강력한 유연성을 제공합니다.

- **장단점 분석**:

  - **장점**: 최고의 결함 격리 및 복원력을 제공합니다. 한 위젯이 충돌하거나, 메모리 누수가 발생하거나, 무한 루프에 빠져도 셸과 다른 위젯들은 안정적으로 유지됩니다. 또한 Python/GTK 위젯, C++/Qt 위젯, Rust 위젯이 공존하는 진정한 기술적 이질성(heterogeneity)을 가능하게 합니다.
  - **단점**: 위젯당 상당한 메모리 및 CPU 오버헤드가 발생합니다. 윈도우 관리, 포커스 처리, 입력 이벤트 라우팅이 극도로 복잡해집니다.39 시각적 결함이 발생하거나 '네이티브'한 느낌이 저하될 위험이 있습니다.



### 3.4  전략 3: 현대적 선언적 UI 모델 (예: Flutter 기반)



이 모델은 Flutter와 같이 화면의 모든 픽셀을 직접 제어하고 네이티브 OS 위젯을 우회하는 현대적인 크로스플랫폼 UI 툴킷을 활용합니다. 아키텍처는 여전히 단일 프로세스 모델이지만, 내부 작동 방식은 Qt나 GTK와 같은 전통적인 툴킷과 근본적으로 다릅니다.

- **구현 세부사항**: Flutter의 계층적 아키텍처 43, 고도로 최적화된 렌더링 파이프라인 44, 그리고 다중 스레드 설계(UI 스레드, GPU 스레드, I/O 스레드)를 분석하고 활용합니다.46
- **반응형 프로그래밍과 DDS의 시너지**: Flutter의 반응형 프로그래밍 모델("UI는 상태의 함수다")과 우리의 데이터 중심 DDS 아키텍처 간의 시너지가 이 전략의 핵심 장점입니다.
  1. Flutter 마이크로-위젯은 `StatefulWidget`으로 구현됩니다.
  2. 위젯의 `State` 객체는 DDS `DataReader`들을 포함합니다.
  3. `DataReader`의 리스너 콜백은 DDS 토픽으로부터 새로운 데이터를 수신합니다.
  4. 리스너 내부에서, 이 콜백은 `setState()`를 호출하여 새로운 데이터를 위젯의 상태 객체에 전달합니다.
  5. 이는 자동으로 `build()` 메서드 호출을 트리거하고, Flutter 엔진은 새로운 위젯 하위 트리와 이전 트리를 효율적으로 비교하여 최소한의 변경 사항만을 화면에 렌더링합니다.45
- **장단점 분석**:
  - **장점**: 고도로 최적화된 Skia 렌더링 엔진 덕분에 잠재적으로 최고의 성능을 제공합니다. Windows, macOS, Linux 등 모든 플랫폼에서 일관되고 고품질의 UI/UX를 보장합니다. 생산성이 높은 현대적인 선언적 개발 모델을 채택합니다.
  - **단점**: 데스크톱 생태계가 Qt/GTK보다 덜 성숙합니다. 고급 과학 차트와 같은 특화된 서드파티 위젯이 상대적으로 적습니다. 네이티브 플랫폼 컴포넌트와의 상호 운용성은 가능하지만 복잡성을 추가합니다.

------



## 4.  고급 주제 및 전략적 권장사항



프레임워크의 고유한 기능을 활용하고 중요한 기술 선택을 내리는 데 필요한 실행 가능한 지침을 제공합니다.



### 4.1  DDS QoS를 활용한 실시간 시각화 마스터하기



이 섹션은 DDS QoS를 사용하여 일반적이고 어려운 UI 문제들을 해결하는 실용적인 가이드입니다. 이론에서 벗어나 프레임워크의 강력함을 실제로 적용하는 방법을 보여줍니다. 포괄적인 QoS 문서들을 적극적으로 참조합니다.26



#### 4.1.1  UI 패턴을 위한 QoS 레시피



- **사용자 입력 (명령)**: 명령을 전송하는 버튼 클릭의 경우, `RELIABILITY.kind = RELIABLE`과 `HISTORY.kind = KEEP_LAST`를 사용합니다. 이는 명령이 확실히 전송되도록 보장하고 가장 최근의 명령만이 의미를 갖도록 합니다.
- **상태 동기화**: 구성 슬라이더의 경우, `DURABILITY.kind = TRANSIENT_LOCAL`을 사용합니다. 이는 새로운 위젯이 생성될 때, 원래 발행자가 사라졌더라도 마지막으로 알려진 슬라이더 값을 즉시 수신하도록 보장합니다.
- **고주파 데이터**: 3D 포인트 클라우드 시각화의 경우, `RELIABILITY.kind = BEST_EFFORT`와 `TIME_BASED_FILTER.minimum_separation`을 UI의 목표 프레임 간격(예: 33ms)으로 설정합니다. 이는 낮은 지연 시간을 우선시하고 UI가 데이터로 인해 압도되는 것을 방지합니다.
- **활성 및 상태 모니터링**: `LIVELINESS.kind`와 `DEADLINE.period`를 사용하여 데이터 소스의 상태를 모니터링합니다. 만약 데드라인을 놓치면, UI는 상태 표시기를 '녹색'에서 '적색'으로 업데이트할 수 있습니다.



#### 4.1.2  GUI 컴포넌트를 위한 권장 DDS QoS 프로파일



**표 2: GUI 컴포넌트를 위한 권장 DDS QoS 프로파일**

| UI 컴포넌트       | QoS 정책            | 권장 설정                         | 근거                                                         |
| ----------------- | ------------------- | --------------------------------- | ------------------------------------------------------------ |
| **명령 버튼**     | `RELIABILITY`       | `RELIABLE`                        | 중요한 사용자 명령이 네트워크에서 손실되지 않도록 보장합니다. |
|                   | `HISTORY`           | `KEEP_LAST`, `depth=1`            | 가장 최근의 명령만이 유효하며, 오래된 명령이 큐에 쌓이는 것을 방지합니다. |
| **구성 설정**     | `DURABILITY`        | `TRANSIENT_LOCAL`                 | 늦게 참여하는 위젯이 현재 구성 상태를 수신하도록 보장합니다. |
| **실시간 플롯**   | `RELIABILITY`       | `BEST_EFFORT`                     | 고속 데이터에 대해 보장된 전달보다 낮은 지연 시간을 우선시합니다. |
| (예: 센서 데이터) | `HISTORY`           | `KEEP_ALL`                        | 정확한 이력 플로팅을 위해 데이터 포인트가 누락되지 않도록 보장합니다. |
|                   | `TIME_BASED_FILTER` | `minimum_separation=33ms`         | UI 재생률(예: ~30fps)에 맞춰 데이터 소스에서 데이터를 조절합니다. |
| **상태 표시기**   | `LIVELINESS`        | `AUTOMATIC`, `lease_duration=...` | 모니터링되는 컴포넌트의 연결이 끊어졌는지 감지합니다.        |
|                   | `DEADLINE`          | `period=...`                      | 주기적인 데이터 업데이트가 제시간에 도착하지 않는 것을 감지합니다. |

이 표는 DDS에 대한 전문가 지식을 실용적인 '레시피' 집합으로 체계화하여 개발자에게 강력한 가속기를 제공합니다. 고주파 비디오 스트림에 `RELIABLE`을 사용하거나(엄청난 지연 시간 유발) 중요한 명령에 `BEST_EFFORT`를 사용하는(데이터 손실 가능) 것과 같은 일반적인 함정을 방지함으로써, 검증된 출발점을 제공하여 복잡한 시스템 튜닝 문제를 직접적으로 해결합니다.



### 4.2  성능 공학 및 벤치마킹



본 프레임워크의 핵심 성능 지표(KPI)는 다음과 같이 정의됩니다.

1. **Glass-to-Glass 지연 시간**: 사용자 행동(예: 마우스 클릭)부터 DDS 네트워크를 통해 전파된 후 화면에 시각적 결과가 나타나기까지의 시간.
2. **UI 응답성**: 특히 과도한 DDS 부하 하에서의 GUI 프레임 속도(FPS).
3. **DDS 처리량**: 응답성이 저하되기 전에 GUI가 시각화할 수 있는 최대 데이터 속도.

단순한 `localhost` 테스트의 함정을 피하는 벤치마킹 전략을 제안합니다.49 이 전략은 네트워크 시뮬레이터를 사용하여 현실적인 지연, 지터, 패킷 손실을 도입하는 것을 포함합니다. 기존 DDS 벤치마크 스위트의 개념을 활용하여 51, 우리가 제안한 

`UIEvent` 전파 패턴과 마이크로-위젯의 데이터 시각화 성능을 구체적으로 측정하는 테스트를 설계합니다.



### 4.3  올바른 DDS 벤더 선택



우리의 아키텍처는 직접적인 DDS API 접근을 권장하므로, 벤더 선택은 매우 중요한 전략적 결정이 됩니다. 이 섹션은 그 선택을 위한 체계적인 프레임워크를 제공합니다. 주요 DDS 구현체들(eProsima Fast DDS, RTI Connext, Eclipse Cyclone DDS 등)을 수집된 연구 자료를 바탕으로 비교합니다.17

**표 3: DDS 벤더 선택 매트릭스**

| 기준               | eProsima Fast DDS                 | Eclipse Cyclone DDS                          | RTI Connext DDS                                           |
| ------------------ | --------------------------------- | -------------------------------------------- | --------------------------------------------------------- |
| **라이선스**       | Apache 2.0                        | EPL 2.0                                      | 상용 (커뮤니티/연구용 옵션 제공)                          |
| **성능 특징**      | 우수; 제로-카피 지원 53           | 우수; 결정론적 성능에 중점 57                | 탁월; 배치(batching) 등 고급 기능 17                      |
| **고급 도구**      | 기본적                            | 기본적                                       | 광범위함 (Admin Console, Analyzer 등) 17                  |
| **보안**           | 표준 DDS-Security                 | 표준 DDS-Security                            | 성숙하고 인증된 DDS-Security 구현 55                      |
| **생태계 및 지원** | 강력함 (ROS2 기본), 커뮤니티 주도 | 강력함, 커뮤니티 주도                        | 상용 지원, 방대한 문서 55                                 |
| **최적 활용 분야** | 오픈소스 프로젝트, ROS2 통합      | 단순성과 결정론적 성능을 우선시하는 프로젝트 | 미션 크리티컬 시스템, 고급 도구 및 지원이 필요한 프로젝트 |

이 매트릭스는 중요한 구현 질문에 직접적으로 답합니다. 라이선스, 기능, 커뮤니티 인식에 대한 이질적인 사실들을 하나의 실행 가능한 비교표로 종합합니다. 이는 프로젝트의 비기능적 요구사항(예: 예산, 보안 요구 수준, 지원 수준)을 각 주요 벤더의 제공 사항과 일치시키는 데 도움을 주어, 개발 후반에 발생할 수 있는 비용이 많이 드는 불일치를 방지합니다.

------



## 5.  결론 및 향후 전망





### 5.1  종합 및 최종 권장사항



제안된 아키텍처는 마이크로-위젯 개념에서 비롯된 독보적인 모듈성과 복원력, 데이터 중심 설계에서 비롯된 투명하고 테스트 가능한 시스템, 그리고 `UIEvent` 버스에서 비롯된 내재적인 확장성과 원격 제어 가능성이라는 독특한 장점들을 가집니다.

최종적으로, 프로젝트의 특성에 따른 미묘한 권장사항을 제시합니다. 예를 들어, "복원력이 가장 중요하고 이기종 기술 스택이 사용될 가능성이 높은 복잡한 미션 크리티컬 제어 시스템을 구축하는 팀에게는 **다중 프로세스 모델(전략 2)**이 가장 견고한 기반을 제공합니다. 일관된 사용자 경험을 갖춘 고성능 크로스플랫폼 시각화 도구의 신속한 개발에 중점을 둔 팀에게는 **선언적 UI 모델(전략 3)**이 가장 미래 지향적이고 효율적인 경로를 제시합니다."



### 5.2  향후 연구 및 개발 방향



- **원활한 보안 통합**: DDS-Security 표준을 완벽하게 통합하여 1, 각 마이크로-위젯이 자신을 인증하고 중앙 보안 정책에 따라 데이터 접근이 통제되는 시스템을 탐구합니다.
- **웹 및 클라우드 통합**: DDS-over-WebSockets 3 또는 RTI의 Web Integration Service 17와 같은 기술을 사용하여 프레임워크를 웹으로 확장합니다. 이를 통해 웹 기반 마이크로-위젯이 데스크톱 위젯과 동일한 데이터 공간에 원활하게 참여할 수 있습니다.
- **AI 주도 및 적응형 UI**: AI 에이전트가 DDS 데이터 공간의 참여자가 되는, 투기적이지만 강력한 비전입니다. 시스템 상태 토픽과 `framework/events/*` 토픽을 모두 구독함으로써 AI는 사용자 패턴을 학습할 수 있습니다. 그런 다음 자체 `UIEvent` 메시지를 발행하여 작업을 자동화하거나, `framework/layout_config` 토픽에 발행하여 레이아웃을 동적으로 조정함으로써 진정으로 지능적이고 적응력 있는 사용자 인터페이스를 만들 수 있습니다.

#### **참고 자료**

1. ROS 2: What is DDS | Data Distribution Service (DDS) Community RTI Connext Users, accessed July 14, 2025, https://community.rti.com/page/ros-2-what-dds
2. robotics - What's the difference between ROS2 and DDS? - Stack Overflow, accessed July 14, 2025, https://stackoverflow.com/questions/51187676/whats-the-difference-between-ros2-and-dds
3. ROS on DDS - ROS2 Design, accessed July 14, 2025, https://design.ros2.org/articles/ros_on_dds.html
4. The Duel of Data: gRPC vs WebSockets - Apidog, accessed July 14, 2025, https://apidog.com/blog/grpc-vs-websockets/
5. gRPC vs. WebSocket: Key differences and which to use - Ably Realtime, accessed July 14, 2025, https://ably.com/topic/grpc-vs-websocket
6. WebSocket vs gRPC: Detailed Comparison and Implementation Guide - VideoSDK, accessed July 14, 2025, https://www.videosdk.live/developer-hub/websocket/websocket-vs-grpc
7. gRPC vs WebSocket | When Is It Better To Use? - Wallarm, accessed July 14, 2025, https://www.wallarm.com/what/grpc-vs-websocket-when-is-it-better-to-use
8. ROS 2 middleware interface - ROS2 Design, accessed July 14, 2025, https://design.ros2.org/articles/ros_middleware_interface.html
9. About ROS 2 middleware implementations, accessed July 14, 2025, https://docs.ros.org/en/foxy/Concepts/About-Middleware-Implementations.html
10. DDS implementations — ROS 2 Documentation: Iron documentation, accessed July 14, 2025, https://docs.ros.org/en/iron/Installation/DDS-Implementations.html
11. Topic and Service name mapping to DDS - ROS2 Design, accessed July 14, 2025, https://design.ros2.org/articles/topic_and_service_names.html
12. Implementing GUI Based Tools Using rqt - Machine Intelligence Laboratory, accessed July 14, 2025, https://mil.ufl.edu/docs/software/rqt.html
13. Overview and usage of RQt — ROS 2 Documentation: Foxy ..., accessed July 14, 2025, https://docs.ros.org/en/foxy/Concepts/About-RQt.html
14. Overview and usage of RQt — ROS 2 Documentation: Foxy 文档, accessed July 14, 2025, https://daobook.github.io/ros2-docs/foxy/Concepts/About-RQt.html
15. ROS 2 - Data display with RViz2 - Stereolabs, accessed July 14, 2025, https://www.stereolabs.com/docs/ros2/rviz2
16. RViz User Guide — ROS 2 Documentation: Humble documentation, accessed July 14, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/RViz/RViz-User-Guide/RViz-User-Guide.html
17. Comparing Open Source DDS to RTI Connext DDS: Considerations in Picking the Right DDS Solution to Run Your Distributed System, accessed July 14, 2025, https://www.rti.com/blog/picking-the-right-dds-solution
18. QoS Policy List - RTI Community, accessed July 14, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/qos_reference/qos_reference/qos_guide_all_in_one.htm
19. Formalizing the Semantics of DDS QoS Policies for Improved Communications in Distributed Smart Grid Applications - MDPI, accessed July 14, 2025, https://www.mdpi.com/2079-9292/12/10/2246
20. State Management in Micro Frontends: Tips and Strategies - PixelFreeStudio Blog, accessed July 14, 2025, https://blog.pixelfreestudio.com/state-management-in-micro-frontends-tips-and-strategies/
21. State Management in Micro Frontend Architectures | by Kaushal Dhakal | Medium, accessed July 14, 2025, https://kaushaldhakal40.medium.com/state-management-in-micro-frontend-architectures-23d6f827c918
22. Connecting The Dots: Data Communication Methods for Micro ..., accessed July 14, 2025, https://www.fabricgroup.com.au/blog/connecting-the-dots-data-communication-methods-for-micro-frontends
23. Mastering Cross-Framework State Management in Micro-Frontends, accessed July 14, 2025, https://frontenddogma.com/posts/2025/mastering-cross-framework-state-management-in-micro-frontends/
24. Unlock Cross-Micro-Frontend State Sharing! - DEV Community, accessed July 14, 2025, https://dev.to/trinly01/unlock-cross-micro-frontend-state-sharing-1clh
25. 4. Quality of Service — The Data Distribution Service Tutorial, accessed July 14, 2025, https://download.zettascale.online/www/docs/Vortex/html/ospl/DDSTutorial/qos.html
26. 3.1.2.1. Standard QoS Policies - 3.3.0 - Fast DDS - eProsima, accessed July 14, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/dds_layer/core/policy/standardQosPolicies.html
27. The Event System | Qt Core | Qt 6.9.1, accessed July 14, 2025, https://doc.qt.io/qt-6/eventsandfilters.html
28. The Main Event Loop - GLib – 2.0 - GTK Documentation, accessed July 14, 2025, https://docs.gtk.org/glib/main-loop.html
29. Working With Qt Events: A Comprehensive Guide, accessed July 14, 2025, https://www.learnqt.guide/working-with-events
30. Responsive Applications - gtksharp - Mono Project, accessed July 14, 2025, https://www.mono-project.com/docs/gui/gtksharp/responsive-applications/
31. Qt & OpenGL: how to render as frequent as possible? - Stack Overflow, accessed July 14, 2025, https://stackoverflow.com/questions/9698322/qt-opengl-how-to-render-as-frequent-as-possible
32. The Main Event Loop - GUI development with Rust and GTK 4, accessed July 14, 2025, https://gtk-rs.org/gtk4-rs/stable/latest/book/main_event_loop.html
33. Inter-Process Communication - Electron, accessed July 14, 2025, https://electronjs.org/docs/latest/tutorial/ipc
34. Distinction between the renderer and main processes in Electron - Stack Overflow, accessed July 14, 2025, https://stackoverflow.com/questions/37669727/distinction-between-the-renderer-and-main-processes-in-electron
35. Electron – building a desktop application using web technologies | JazzTeam technical articles, accessed July 14, 2025, https://jazzteam.org/technical-articles/technical-articles-electron-creating-a-desktop-application-using-web-technologies/
36. Electron: Communicate from Renderer to Main Process | Firmansyah Yanuar, accessed July 14, 2025, https://fyfirman.com/blog/communicate-from-renderer-to-main-process
37. Window embedding in Qt Quick, accessed July 14, 2025, https://www.qt.io/blog/window-embedding-in-qt-quick
38. Window Embedding | Qt 6.9 - Qt Documentation, accessed July 14, 2025, https://doc.qt.io/qt-6/qtdoc-demos-windowembedding-example.html
39. Embedding HWND into external process using SetParent - Stack Overflow, accessed July 14, 2025, https://stackoverflow.com/questions/170800/embedding-hwnd-into-external-process-using-setparent
40. X Window System protocols and architecture - Wikipedia, accessed July 14, 2025, https://en.wikipedia.org/wiki/X_Window_System_protocols_and_architecture
41. Rationale and discussion | XEmbed Protocol Specification, accessed July 14, 2025, https://specifications.freedesktop.org/xembed-spec/0.5/rationale.html
42. Xlib usage examples - Victor Gaydov, accessed July 14, 2025, https://gavv.net/articles/xlib-usage-examples/
43. Flutter architectural overview, accessed July 14, 2025, https://docs.flutter.dev/resources/architectural-overview
44. Understanding Flutter rendering pipeline: Build phase | by Roman Ismagilov - Medium, accessed July 14, 2025, https://medium.com/@pomis172/understanding-flutter-rendering-pipeline-build-phase-cf7e05aa12f1
45. The Ultimate Guide to Screen Rendering in Flutter UI - DhiWise, accessed July 14, 2025, https://www.dhiwise.com/post/exploring-screen-rendering-in-flutter-ui
46. Exploration of the Flutter Rendering Mechanism from Architecture to Source Code, accessed July 14, 2025, https://www.alibabacloud.com/blog/exploration-of-the-flutter-rendering-mechanism-from-architecture-to-source-code_597285
47. flutterExplain the concept of rendering pipeline in Flutter. | by Karthick Raja - Medium, accessed July 14, 2025, https://medium.com/@karthickrajaraja424/explain-the-concept-of-rendering-pipeline-in-flutter-d5f756f46ab9
48. Manage QoS - MATLAB & Simulink - MathWorks, accessed July 14, 2025, https://www.mathworks.com/help/dds/ug/manage-qos.html
49. Is it possible for me to do the performance testing in localhost with actual network environment? - Stack Overflow, accessed July 14, 2025, https://stackoverflow.com/questions/2135551/is-it-possible-for-me-to-do-the-performance-testing-in-localhost-with-actual-net
50. eProsima/benchmarking - GitHub, accessed July 14, 2025, https://github.com/eProsima/benchmarking
51. Is There a Benchmarking Tool That Tests the Performance of Multiple DDS Products As Well As RTI | Data Distribution Service (DDS) Community RTI Connext Users, accessed July 14, 2025, https://community.rti.com/forum-topic/there-benchmarking-tool-tests-performance-multiple-dds-products-well-rti
52. Performance Testing - OpenDDS, accessed July 14, 2025, https://opendds.org/about/performance.html
53. DDS implementation performance benchmark - ROS General - Open Robotics Discourse, accessed July 14, 2025, https://discourse.ros.org/t/dds-implementation-performance-benchmark/19343
54. ROS 2 / Fast-DDS Inter-Process Performance Test Result - GitHub Pages, accessed July 14, 2025, https://osrf.github.io/TSC-RMW-Reports/humble/sony-test-results/ROS-2_Fast-DDS_interprocess_performance-test-result.pdf
55. c# - DDS - Which one is recommended - OpenSplice or CoreDX? - Stack Overflow, accessed July 14, 2025, https://stackoverflow.com/questions/38220008/dds-which-one-is-recommended-opensplice-or-coredx
56. What are alternatives to OpenSplice and RTI DDS implementations? - Stack Overflow, accessed July 14, 2025, https://stackoverflow.com/questions/9501499/what-are-alternatives-to-opensplice-and-rti-dds-implementations
57. The DDS Race: what we've seen so far - General - ROS Discourse, accessed July 14, 2025, https://discourse.ros.org/t/the-dds-race-what-weve-seen-so-far/27421


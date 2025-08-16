# ROS2 리소스를 활용한 GUI 애플리케이션 개발에 대한 종합적 분석

## 1.  GUI 개발을 위한 ROS2의 기초 원리

### 1.1  UI 백엔드로서의 ROS2 통신 그래프

성공적인 로봇 운영체제(Robot Operating System, ROS) 기반 그래픽 사용자 인터페이스(Graphical User Interface, GUI) 개발의 핵심은 ROS2의 통신 아키텍처를 UI의 백엔드로 활용하는 데 있습니다. 잘 설계된 ROS2 시스템은 그 자체로 GUI가 상호작용할 수 있는 구조화된 데이터 및 제어 계층을 제공합니다.1 ROS 그래프를 구성하는 노드, 토픽(Topics), 서비스(Services), 액션(Actions)은 GUI의 다양한 기능을 구현하는 근간이 되는 API 역할을 수행합니다. 각 통신 방식의 고유한 특성을 이해하고 UI 상호작용의 성격에 맞게 적절히 선택하는 것은 반응성 높고 직관적인 GUI를 설계하기 위한 첫 번째 아키텍처 원칙입니다.

#### 1.1.1  연속적인 데이터 시각화를 위한 토픽 (Topics)

토픽은 단방향, 비동기식, 다대다(many-to-many) 발행/구독(publish/subscribe) 통신 모델을 따릅니다. 이는 센서 데이터, 로봇 상태, 로그 메시지와 같이 연속적으로 발생하는 데이터 스트림을 처리하는 데 가장 이상적인 방식입니다.3 GUI의 관점에서 토픽은 실시간 데이터 시각화의 중추적인 역할을 담당합니다. 예를 들어, 카메라의 이미지 데이터(`sensor_msgs/msg/Image`), 로봇의 위치 및 속도 정보(`nav_msgs/msg/Odometry`), 각 관절의 상태(`sensor_msgs/msg/JointState`), 시스템 로그(`rcl_interfaces/msg/Log`) 등을 GUI의 비디오 피드, 플로터, 상태 대시보드 등에 표시하는 데 사용됩니다.4

구현 관점에서 GUI 애플리케이션 내의 ROS2 노드는 특정 토픽을 구독(subscribe)하고, 메시지가 수신될 때마다 등록된 콜백 함수를 통해 데이터를 전달받습니다. 이 콜백 함수 내에서 수신된 데이터를 파싱하여 GUI 위젯을 업데이트하는 로직이 수행됩니다. 예를 들어, `rqt_image_view`는 이미지 토픽을 구독하여 화면에 비디오 스트림을 렌더링하고 6, 

`rqt_plot`과 같은 도구는 수치 데이터를 구독하여 시간에 따른 변화를 그래프로 시각화합니다.5 여기서 핵심적인 기술적 과제는 ROS2 콜백이 실행되는 스레드와 GUI가 실행되는 메인 스레드 간의 데이터를 안전하게 전달하는 것이며, 이는 후속 섹션에서 심도 있게 다룰 것입니다.

#### 1.1.2  동기적이고 빠른 상호작용을 위한 서비스 (Services)

서비스는 동기식 요청/응답(request/response) 통신 모델을 제공하며, 원격 프로시저 호출(Remote Procedure Call, RPC)과 유사한 개념입니다.3 사용자가 GUI에서 특정 버튼을 클릭했을 때, 빠르고 명확한 결과가 반환되어야 하는 상호작용에 적합합니다. 예를 들어, "현재 설정값 조회", "센서 캘리브레이션 트리거", "특정 좌표에 대한 역기구학(Inverse Kinematics) 계산"과 같은 작업들은 서비스로 구현하기에 이상적입니다.3

서비스 클라이언트는 서버에 요청을 보낸 후, 응답이 올 때까지 대기(blocking)합니다. 이 특성은 작업이 신속하게 완료된다는 보장이 있을 때 유용하지만, 동시에 잠재적인 위험을 내포합니다. 만약 서비스 서버가 요청을 처리하는 데 오랜 시간이 걸린다면, 서비스를 호출한 클라이언트 스레드는 그대로 멈추게 됩니다. 만약 이 스레드가 GUI의 이벤트 루프를 처리하는 스레드와 동일하다면, 애플리케이션 전체가 '얼어붙는(freeze)' 현상이 발생하여 사용자 경험을 심각하게 저해할 수 있습니다.3 따라서 공식 문서에서도 장시간 실행되는 프로세스에 서비스를 사용하는 것을 지양하도록 경고하고 있으며, 이러한 작업에는 다음에 설명할 액션을 사용해야 합니다.

#### 1.1.3  장시간 실행되는 비동기 작업을 위한 액션 (Actions)

액션은 ROS2에서 제공하는 가장 정교한 통신 방식으로, 장시간이 소요될 수 있는 비동기 작업을 위해 특별히 설계되었습니다. 액션은 단순히 요청과 응답을 넘어, 작업의 진행 상황을 지속적으로 피드백하고, 사용자가 원할 때 작업을 중간에 취소할 수 있는 기능을 제공합니다.3 이러한 특성 덕분에 액션은 복잡한 로봇 GUI에서 필수적인 상호작용을 구현하는 데 가장 적합한 백엔드 메커니즘입니다. "지정된 위치로 이동(Navigate to Point)", "로봇 팔 경로 실행(Execute a Manipulation Trajectory)", "특정 영역 스캔(Scan an Area)"과 같은 작업들이 대표적인 예시입니다.

액션은 세 가지 주요 구성요소로 이루어집니다.8

1. **목표 (Goal):** 클라이언트가 서버에게 달성하고자 하는 작업의 목표를 명시하여 전송합니다. GUI에서는 사용자가 "시작" 버튼을 누르는 행위에 해당합니다.
2. **피드백 (Feedback):** 서버는 목표를 수행하는 동안 주기적으로 작업의 진행 상태를 클라이언트에게 전송합니다. GUI에서는 이 피드백 메시지를 받아 프로그레스 바(progress bar)를 업데이트하거나 현재 상태를 텍스트로 표시할 수 있습니다.
3. **결과 (Result):** 작업이 완료되면(성공, 실패, 또는 취소), 서버는 최종 결과를 클라이언트에게 한 번 전송합니다. GUI에서는 이 결과를 바탕으로 성공 또는 실패를 알리는 다이얼로그 박스를 표시할 수 있습니다.

또한, 액션 클라이언트는 언제든지 목표 취소(cancel goal) 요청을 보낼 수 있어, 사용자에게 높은 수준의 제어권을 부여합니다.8 내부적으로 액션은 토픽과 서비스의 조합으로 구현되어 있으나(`_action/send_goal`, `_action/get_result` 등의 내부 서비스와 토픽 생성), 이러한 복잡성은 `rclpy`나 `rclcpp`와 같은 클라이언트 라이브러리 수준에서 추상화되어 개발자는 간단한 API를 통해 액션을 사용할 수 있습니다.8

이러한 통신 방식의 선택은 단순히 기술 구현의 문제가 아니라, 사용자 경험을 결정짓는 중요한 아키텍처 설계 결정입니다. 예를 들어, 사용자가 GUI에서 "내비게이션 시작" 버튼을 클릭하는 시나리오를 생각해 보겠습니다.

1. 만약 개발자가 이 기능을 **서비스**로 구현한다면, GUI 스레드 또는 서비스를 호출한 ROS 스레드는 로봇이 목적지에 도착할 때까지 완전히 멈추게 됩니다.3 UI는 응답 불능 상태가 되며, 진행 상황을 표시하거나 중간에 취소할 방법이 없습니다. 이는 매우 나쁜 사용자 경험입니다.
2. 만약 **토픽**을 사용해 목표 지점을 발행(publish)한다면, 이는 "쏘고 잊어버리는(fire-and-forget)" 방식입니다. GUI는 목표가 제대로 수신되었는지, 현재 처리 중인지, 성공했는지, 실패했는지 전혀 알 수 없습니다.
3. 반면 **액션**을 사용하면, GUI는 비동기적으로 목표를 전송합니다. 그 후, 주기적인 `피드백` 메시지(예: 목표까지 남은 거리)를 수신하여 프로그레스 바를 업데이트할 수 있으며, 사용자가 "취소" 버튼을 누르면 언제든지 `cancel_goal` 요청을 보낼 수 있습니다. 작업이 완료되면 최종 `결과`를 받아 사용자에게 알립니다.8

결론적으로, 액션 패턴은 로보틱스 UI에서 흔히 발생하는 비동기적이고, 상태를 가지며, 장시간 실행되는 상호작용을 위해 만들어졌습니다. 개발자는 UI 상호작용의 본질(단순 조회인지, 연속 데이터 스트림인지, 장시간 작업인지)을 정확히 분석하고 그에 맞는 ROS2 통신 프리미티브를 선택해야 합니다. 이는 ROS2 기반 GUI 개발의 가장 기초적이면서도 핵심적인 원칙입니다.

### 1.2  실행 모델과 GUI 반응성에 미치는 영향

ROS2 애플리케이션의 콜백 처리 방식, 즉 실행 모델(Execution Model)은 GUI의 반응성에 직접적인 영향을 미칩니다. ROS2의 실행자(Executors)와 콜백 그룹(Callback Groups)에 대한 깊이 있는 이해는 UI 멈춤 현상을 방지하고 안정적인 애플리케이션을 구축하는 데 필수적입니다.

#### 1.2.1  콜백 처리의 심장, 실행자 (Executors)

ROS2 노드는 스스로 콜백을 처리하지 않습니다. 실행자가 노드를 "스핀(spin)"시켜야만, 웨이트셋(wait set)에서 들어오는 이벤트(새로운 메시지, 서비스 요청, 타이머 만료 등)를 확인하고 해당 콜백을 실행할 수 있습니다.9 주요 클라이언트 라이브러리인 `rclcpp`와 `rclpy`는 두 가지 기본 실행자를 제공합니다.9

- **단일 스레드 실행자 (Single-Threaded Executor):** 하나의 스레드를 사용하여 모든 콜백을 순차적으로 처리합니다. 구현이 간단하고 스레드 안전성(thread safety)에 대한 고민이 적지만, 하나의 장기 실행 콜백이 다른 모든 콜백의 실행을 막는 병목 현상을 유발할 수 있습니다.
- **다중 스레드 실행자 (Multi-Threaded Executor):** 스레드 풀(thread pool)을 사용하여 여러 콜백을 병렬로 처리할 수 있습니다. 시스템의 처리량을 극대화할 수 있지만, 여러 스레드에서 공유 데이터에 동시에 접근할 때 발생할 수 있는 경쟁 상태(race condition)를 개발자가 직접 관리해야 합니다.

#### 1.2.2  세밀한 동시성 제어를 위한 콜백 그룹 (Callback Groups)

콜백 그룹은 특히 다중 스레드 실행자 환경에서 콜백들이 서로 어떻게 실행될지를 제어하는 강력한 도구입니다.12

- **상호 배제 콜백 그룹 (Mutually Exclusive Callback Group):** 이 그룹에 속한 콜백들은 절대로 동시에 실행될 수 없습니다. 하나의 콜백이 실행 중이면 같은 그룹의 다른 콜백은 대기해야 합니다. 이는 노드의 기본 콜백 그룹 유형이며, 공유 자원에 대한 안전한 접근을 보장하지만 성능 저하의 원인이 될 수 있습니다.12
- **재진입 가능 콜백 그룹 (Reentrant Callback Group):** 이 그룹에 속한 콜백들은 아무런 제약 없이 병렬로 실행될 수 있습니다. 심지어 동일한 콜백의 여러 인스턴스도 동시에 실행될 수 있습니다. 이는 최대의 병렬성을 제공하지만, 개발자가 뮤텍스(mutex) 등을 사용하여 공유 데이터에 대한 스레드 안전성을 직접 보장해야 합니다.12

#### 1.2.3  블로킹으로 인한 GUI 멈춤 문제

ROS2 GUI 개발에서 마주하는 가장 근본적인 문제는 GUI 프레임워크(예: Qt의 `app.exec()`)와 ROS2 노드(`rclpy.spin()`)가 모두 자신의 스레드를 블로킹(blocking)하는 이벤트 루프를 가지고 있다는 점입니다.14 이 두 개의 블로킹 함수를 같은 스레드에서 실행하려고 하면, 필연적으로 하나가 다른 하나를 막게 됩니다. 만약 장시간 실행되는 ROS 콜백이 처리 중이라면 GUI는 사용자의 입력에 반응하지 않고 멈춰버릴 것입니다. 반대로 GUI가 복잡한 렌더링이나 이벤트 처리로 바쁘다면, ROS 메시지 처리가 지연될 것입니다.

특히 위험한 시나리오는 동기식 서비스 호출에서 발생합니다. 예를 들어, 단일 스레드 실행자 환경에서 타이머 콜백이 동기식으로 다른 서비스를 호출하면, 해당 콜백은 응답이 올 때까지 실행자 전체를 점유하며 블로킹합니다. 그런데 서비스의 응답 처리 역시 동일한 실행자에 의해 수행되어야 하므로, 실행자는 영원히 응답을 기다리는 교착 상태(deadlock)에 빠지게 됩니다.7 이는 왜 순진한 단일 스레드 접근 방식이 위험한지를 명확히 보여줍니다.

이러한 문제를 해결하기 위한 아키텍처적 접근 방식은 다음과 같은 논리적 흐름을 따릅니다.

1. 개발자는 독립형 Qt GUI를 만들고 ROS 메시지를 처리해야 합니다. 가장 간단한 방법은 메인 스레드에서 `rclpy.spin(node)`를 호출하는 것입니다.14
2. 이 방법은 즉시 실패합니다. 왜냐하면 Qt의 `app.exec()` 역시 메인 스레드에서 실행되어야 하기 때문입니다. 두 호출은 상호 배타적입니다.
3. 다음 논리적 단계는 `rclpy.spin(node)`를 별도의 스레드(예: `QThread` 또는 `std::thread`)에서 실행하는 것입니다.14 이것이 초기 블로킹 문제를 해결하는 표준적인 방법입니다.
4. 이제 ROS 스레드에서 실행된 콜백이 데이터를 수신하고 GUI 위젯(예: `QLabel`)을 업데이트해야 하는 상황이 발생합니다. GUI가 아닌 스레드에서 Qt 위젯을 직접 조작하는 것은 금지되어 있으며, 애플리케이션 충돌을 유발합니다.
5. 따라서 ROS 스레드에서 GUI 스레드로 데이터를 안전하게 전달하기 위한 스레드 안전(thread-safe) 통신 메커니즘이 필요합니다. Qt의 시그널-슬롯(signals and slots) 메커니즘이 바로 이 역할을 수행합니다. ROS 콜백은 수신한 데이터를 담아 시그널을 `emit`하고, 이 시그널은 GUI 위젯의 슬롯(메서드)에 연결되어 GUI 메인 스레드에서 안전하게 위젯을 업데이트합니다.17
6. 여기서 더 깊은 아키텍처적 고려가 필요합니다. 만약 ROS 스레드에 여러 개의 고주파 토픽 콜백과 하나의 느린 서비스 클라이언트 콜백이 있다면 어떻게 될까요? ROS 스레드에서 단일 스레드 실행자를 사용하면, 느린 서비스 호출이 고주파 데이터 처리를 지연시킬 수 있습니다. 바로 이 지점에서 다중 스레드 실행자와 콜백 그룹이 필수적이 됩니다. 개발자는 고주파 토픽 콜백들을 재진입 가능 그룹에 배치하여 처리량을 극대화하고, 느린 서비스 클라이언트는 별도의 상호 배제 그룹에 배치하여 다른 작업에 영향을 주지 않도록 격리할 수 있습니다.11

결론적으로, 성공적인 ROS2 GUI 아키텍처는 본질적으로 동시성 시스템 설계 문제입니다. 개발자는 GUI와 ROS 노드를 별개의 독립체로 취급할 수 없습니다. UI 멈춤 현상과 ROS 처리 병목 현상을 모두 방지하기 위해 스레딩 모델, 실행자 전략, 콜백 그룹핑을 함께 설계해야만 합니다. ROS2의 실행 모델은 단순한 성능 튜닝 도구가 아니라, GUI의 반응성을 보장하기 위해 GUI의 스레딩 모델과 함께 공동으로 설계되어야 하는 핵심 아키텍처 구성 요소입니다.

## 2.  GUI 개발 접근 방식 비교 연구

ROS2 환경에서 GUI를 개발하는 방법은 크게 세 가지로 나눌 수 있습니다: `rqt` 프레임워크를 활용한 플러그인 개발, 독립 실행형(standalone) 애플리케이션 개발, 그리고 웹 기술을 이용한 인터페이스 개발입니다. 각각의 접근 방식은 개발 속도, 성능, 유연성, 배포 복잡성 등에서 뚜렷한 장단점을 가지므로, 프로젝트의 목표와 요구사항에 따라 신중하게 선택해야 합니다.

### 2.1  rqt 플러그인 프레임워크

`rqt`는 Qt 기반의 그래픽 사용자 인터페이스 프레임워크로, 다양한 GUI 도구들을 플러그인 형태로 구현하고, 이들을 하나의 메인 윈도우 안에 도킹 가능한 위젯으로 실행할 수 있게 해줍니다.18 이는 특히 로봇 시스템의 디버깅, 모니터링, 인트로스펙션을 위한 도구를 개발할 때 모듈화되고 통합된 개발 경험을 제공합니다.20

#### 2.1.1  장점

- **신속한 개발 및 재사용성:** `rqt`의 가장 큰 장점은 `rqt_plot`, `rqt_image_view`, `rqt_graph` 등 이미 존재하는 풍부한 플러그인 생태계를 활용하여 복잡한 커스텀 대시보드를 신속하게 구축할 수 있다는 점입니다.5 새로운 플러그인을 개발하는 과정 역시 공통 API와 표준화된 시작/종료 절차, 상태 복원 기능 등을 통해 간소화됩니다.19
- **간소화된 생명주기 관리:** `rqt` 프레임워크가 ROS 노드 초기화(플러그인 코드에서 `rclpy.init()` 등을 직접 호출하지 않음)와 메인 GUI 이벤트 루프 처리를 담당합니다. 따라서 개발자는 위젯의 핵심 로직 구현에만 집중할 수 있습니다.23

#### 2.1.2  단점

- **UI/UX 제약:** 플러그인은 `rqt` 컨테이너와 그 고유의 디자인 스타일에 종속됩니다. 이로 인해 특히 우분투(Ubuntu)의 GNOME 데스크톱 환경에서는 다소 "구식"으로 보이거나 현대적인 애플리케이션과 비교해 일관성이 떨어질 수 있습니다.25 사용자가 원하는 대로 UI를 자유롭게 커스터마이징하는 데 한계가 있습니다.
- **모놀리식 프로세스:** 모든 플러그인은 동일한 `rqt` 프로세스 내에서 실행됩니다. 이는 하나의 플러그인에서 발생한 오류나 충돌이 `rqt` 애플리케이션 전체를 중단시킬 수 있는 잠재적 위험을 의미합니다.26 (참고: Qt5부터는 별도 프로세스의 위젯을 임베딩하는 기능이 제거되어 프로세스 수준의 격리가 더 어려워졌습니다.)

#### 2.1.3  플러그인 개발 과정 (Python 기준)

1. **패키지 생성:** `rqt_gui`, `rqt_gui_py`, `rclpy` 등에 대한 의존성을 갖는 ROS2 패키지를 생성합니다.23

2. **`plugin.xml` 작성:** 이 매니페스트 파일은 플러그인을 ROS 생태계에 알리는 핵심적인 역할을 합니다. 파일 내에는 플러그인의 클래스 이름, 타입(예: `rqt_mypkg.my_module.MyPlugin`), 베이스 클래스(`rqt_gui_py::Plugin`), 그리고 `rqt` 메뉴에 표시될 메타데이터(레이블, 아이콘, 상태 팁 등)를 명시해야 합니다.27

3. **플러그인 클래스 구현:** 핵심 로직은 `rqt_gui_py.plugin.Plugin`을 상속받는 클래스 내에 구현됩니다. `__init__` 메서드에서 `QWidget`을 생성하고, `context.add_widget()`을 호출하여 `rqt` 프레임에 통합합니다.24 ROS 구독자/발행자 등은 이 메서드 내에서 생성되며, 

   `shutdown_plugin` 메서드는 플러그인이 종료될 때 리소스를 정리하는 데 사용됩니다.24

### 2.2  독립 실행형 GUI 애플리케이션

독립 실행형 애플리케이션은 PyQt/PySide(Python) 또는 C++/Qt 등으로 개발된 표준 데스크톱 프로그램이면서 동시에 하나의 완전한 ROS2 노드로서 기능하는 방식입니다. 이 접근법은 애플리케이션의 외형, 동작, 생명주기에 대한 완전한 제어권을 개발자에게 부여합니다.29

#### 2.2.1  장점

- **완전한 UI/UX 자유도:** 개발자는 애플리케이션의 디자인, 브랜딩, 사용자 경험을 완전히 통제할 수 있습니다. 이는 최종 사용자나 고객에게 제공되는 상용 제품을 개발할 때 필수적인 요소입니다.32
- **분리된 배포:** 애플리케이션은 자체적으로 실행 가능한 파일로 만들어지므로, 여러 `rqt` 플러그인들의 집합을 관리하는 것보다 배포가 간결할 수 있습니다. 표준 `ros2 launch` 파일을 통해 다른 ROS 노드들과 함께 실행될 수 있습니다.31

#### 2.2.2  단점

- **동시성 처리의 복잡성:** 1.2절에서 논의했듯이, 개발자는 GUI 스레드와 ROS2 통신 스레드를 분리하고 관리할 전적인 책임을 집니다. 이는 GUI 멈춤이나 교착 상태와 같은 버그의 주된 원인이 되며, 이 접근법의 가장 큰 기술적 난관입니다.7
- **기능의 재구현:** `rqt` 플러그인으로 손쉽게 사용할 수 있는 그래프 플로팅이나 이미지 뷰어와 같은 공통 기능들을 처음부터 직접 구현하거나 외부 라이브러리를 통합해야 하는 부담이 있습니다.33

#### 2.2.3  핵심 구현 패턴 (Python/PyQt 기준)

이 문제에 대한 표준적인 해결책은 스레딩(threading)입니다.

1. 애플리케이션의 **메인 스레드**는 Qt의 이벤트 루프(`app.exec()`)를 실행하여 GUI의 반응성을 책임집니다.
2. ROS2 관련 로직을 실행하기 위한 별도의 **워커 스레드**(worker thread, 종종 `QThread`를 상속받아 구현)를 생성합니다.
3. 워커 스레드의 `run()` 메서드 내에서 `rclpy.init()`를 호출하고, ROS2 노드를 인스턴스화한 후, `rclpy.spin(node)`을 실행하여 ROS2 이벤트 루프를 시작합니다.15
4. 워커 스레드의 ROS 콜백 함수가 GUI를 업데이트해야 할 데이터를 수신하면, 해당 데이터를 담아 Qt **시그널(signal)을 발생(emit)**시킵니다.
5. 이 시그널은 메인 GUI 클래스에 정의된 **슬롯(slot)**에 연결됩니다. Qt의 스레드 간 시그널-슬롯 메커니즘 덕분에, 이 슬롯은 GUI의 메인 스레드에서 안전하게 실행되어 위젯을 직접 조작할 수 있게 됩니다.14

### 2.3  웹 기반 인터페이스

이 접근법은 UI를 ROS 시스템으로부터 완전히 분리합니다. 로봇이나 연결된 컴퓨터에서 웹소켓(WebSocket) 서버가 실행되어 ROS2 메시지를 웹 브라우저에서 실행되는 표준 웹 프론트엔드와 중계합니다.25

#### 2.3.1  브릿지: `rosbridge_server` 대 `foxglove_bridge`

- **`rosbridge_server`:** 전통적인 솔루션으로, 웹소켓을 통해 JSON 기반 프로토콜을 제공합니다. 기능적으로는 충분하지만 성능상의 한계를 가질 수 있습니다.36
- **`foxglove_bridge`:** 현대적이고 고성능의 C++ 기반 대안입니다. 더 효율적인 바이너리 프로토콜을 사용하며, ROS2의 IDL 메시지 정의, 파라미터, 그래프 인트로스펙션과 같은 ROS2 고유 기능을 네이티브로 지원하여 새로운 프로젝트에 강력히 권장됩니다.36

#### 2.3.2  장점

- **궁극의 이식성:** UI는 ROS 설치가 필요 없는 모든 최신 웹 브라우저(데스크톱, 태블릿, 스마트폰)에서 접근 가능합니다. 이는 원격 모니터링 및 제어에 이상적입니다.25
- **웹 기술 생태계 활용:** 개발팀은 React, Vue 등 방대하고 성숙한 웹 개발 프레임워크와 관련 인력을 활용할 수 있습니다.32
- **디커플링 및 보안:** UI가 로봇의 핵심 프로세스와 물리적으로 분리됩니다. 웹소켓 브릿지는 단일화되고 보안 설정이 용이한 진입점 역할을 합니다.37

#### 2.3.3  단점

- **네트워크 의존성:** 성능이 전적으로 네트워크 연결 품질에 좌우됩니다. 지연 시간(latency)과 대역폭(bandwidth)이 중요한 문제가 될 수 있습니다.
- **브릿지 오버헤드:** 웹소켓 브릿지 자체가 로봇의 컴퓨팅 자원을 소모하며, 시스템에 또 다른 복잡성 계층을 추가합니다.
- **개발 오버헤드:** ROS2 개발 외에 프론트엔드 프레임워크, 빌드 도구, 웹 서버 등 완전한 웹 개발 스택이 필요합니다.40

### 2.4  프레임워크 선택 매트릭스

다음 표는 세 가지 접근 방식을 다각적으로 비교하여 개발자가 아키텍처를 선택하는 데 도움을 주기 위해 작성되었습니다. 이는 복잡한 트레이드오프를 명확하고 실행 가능한 형식으로 요약합니다.

| 기준                   | `rqt` 플러그인                                               | 독립 실행형 애플리케이션                                     | 웹 기반 UI                                                   |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **개발 속도**          | **높음.** 기존 프레임워크와 플러그인을 활용하여 신속한 개발 가능.19 | **중간~낮음.** 스레딩 등 기반 작업에 상당한 노력이 필요.14   | **중간~낮음.** 완전한 풀스택 웹 개발 필요.40                 |
| **성능**               | **중간.** 공유 프로세스에서 실행되어 다른 플러그인의 영향을 받을 수 있음.26 | **높음.** C++/Python 네이티브 실행. 성능은 GUI 툴킷과 하드웨어에 의해 결정됨. | **가변적.** 네트워크 지연 및 브릿지 성능에 크게 의존 (`foxglove_bridge`가 우수).36 |
| **UI/UX 유연성**       | **낮음.** `rqt`의 외형과 도킹 시스템에 제약됨.25             | **높음.** 디자인, 레이아웃, 브랜딩에 대한 완전한 제어권.     | **높음.** 현대 웹 디자인(CSS, JS 프레임워크)의 모든 기능 활용 가능. |
| **확장성**             | **높음 (ROS 내에서).** 다른 `rqt` 플러그인과 쉽게 통합하여 복잡한 대시보드 구성 가능.5 | **중간.** 표준 소프트웨어 방식으로 확장 가능하나, ROS 도구 통합은 직접 구현해야 함. | **높음 (일반적).** ROS뿐만 아니라 모든 웹 API 또는 서비스와 통합 가능. |
| **배포 복잡성**        | **낮음.** 표준 ROS2 워크스페이스의 일부로 배포됨.            | **중간.** 독립 실행 파일로 배포되나, 의존성 관리가 필요함.   | **높음.** ROS 브릿지, 웹 서버, 프론트엔드 애플리케이션의 배포가 모두 필요.41 |
| **요구되는 팀 전문성** | **ROS 개발자.** 기본적인 Qt 지식으로 충분.                   | **ROS 개발자 + 동시성/GUI 전문가.** 스레딩과 GUI 이벤트 루프에 대한 깊은 이해 필요. | **ROS 개발자 + 웹 개발팀 (프론트엔드/백엔드).**              |
| **이상적인 사용 사례** | 개발자 도구, 디버깅 대시보드, 신속한 프로토타이핑, 내부 유틸리티.6 | 잘 다듬어진 고객 대면 애플리케이션, 고도로 특화된 UI, 성능이 중요한 인터페이스. | 원격 모니터링/제어, 크로스플랫폼 접근, 클라우드 통합 대시보드.25 |

## 3.  고급 아키텍처 패턴 및 모범 사례

효과적인 ROS2 GUI 애플리케이션을 개발하기 위해서는 단순히 ROS와 GUI 프레임워크를 연결하는 것을 넘어, 유지보수 가능하고 확장 가능하며 안정적인 소프트웨어를 만들기 위한 검증된 아키텍처 패턴을 적용해야 합니다. 이 장에서는 모델-뷰-컨트롤러(MVC) 패턴, 상태 관리 기법, 데이터 동기화 및 스레드 안전성 확보를 위한 모범 사례를 심도 있게 탐구하고, 실제 사례 연구를 통해 이러한 패턴들이 어떻게 적용되는지 분석합니다.

### 3.1  ROS2 맥락에서의 모델-뷰-컨트롤러(MVC) 패턴

MVC 패턴은 사용자 인터페이스를 세 가지 상호 연결된 구성 요소로 분리하여 관심사 분리(Separation of Concerns)를 달성하는 고전적인 소프트웨어 디자인 패턴입니다. Qt 자체는 모델-뷰(Model-View) 아키텍처를 내장하고 있지만 43, 이를 확장하여 완전한 MVC 패턴을 ROS2 GUI에 적용하면 코드의 구조를 명확하게 하고, 테스트 용이성을 높이며, 유지보수성을 크게 향상시킬 수 있습니다.44

ROS2 GUI 애플리케이션에서 MVC 패턴은 다음과 같이 적용될 수 있습니다.

- **모델 (Model):** 로봇의 상태, 센서 데이터, 파라미터 등 시스템의 데이터를 관리하는 ROS2 노드(들)가 모델의 역할을 합니다. 모델은 시스템 상태에 대한 '단일 진실 공급원(Single Source of Truth)'이며, UI에 대해 완전히 독립적이어야 합니다. 즉, 모델은 자신이 어떤 뷰에 의해 표시되는지 알지 못합니다.45
- **뷰 (View):** 사용자와 직접 상호작용하는 Qt 또는 GTK 위젯들이 뷰를 구성합니다. 뷰의 역할은 모델의 데이터를 사용자에게 시각적으로 표현하고, 사용자의 입력(버튼 클릭, 텍스트 입력 등)을 받아 컨트롤러에 전달하는 것에 국한됩니다. 뷰는 비즈니스 로직을 포함하지 않는 '멍청한(dumb)' 컴포넌트여야 합니다.43
- **컨트롤러 (Controller):** 모델과 뷰 사이의 중재자 역할을 하는 전용 클래스 또는 함수들의 집합입니다. 컨트롤러는 ROS 토픽을 구독하고 서비스/액션 클라이언트를 통해 모델의 상태 변화를 감지한 후, 뷰에 업데이트를 요청합니다(주로 시그널-슬롯을 통해). 또한, 뷰로부터 사용자 입력 이벤트를 받아 이를 해석하고, ROS 메시지를 발행하거나 서비스/액션을 호출하여 모델의 상태를 변경하도록 지시합니다.46

이러한 구조는 UI 로직과 ROS 통신 및 비즈니스 로직이 하나의 거대한 클래스에 뒤섞이는 '신의 객체(God Object)' 안티패턴을 방지합니다.29 예를 들어, ROS 파라미터(모델)의 변경이 

`QLineEdit` 위젯(뷰)에 반영되는 과정은 컨트롤러를 통해 이루어집니다. 컨트롤러는 파라미터 변경 콜백을 수신하고, 뷰의 특정 메서드를 호출하여 `QLineEdit`의 텍스트를 업데이트합니다.

MVC 패턴을 적용하는 것은 단순히 스타일적인 선택이 아닙니다. 이는 유지보수 및 테스트 가능한 ROS2 GUI 애플리케이션을 만드는 핵심 전략입니다. 순진한 구현 방식은 모든 로직을 하나의 `QMainWindow` 클래스에 넣는 경향이 있습니다.14 이 경우 클래스는 금방 비대해지고 이해하기 어려워지며, UI 변경이 ROS 로직을 망가뜨리거나 그 반대의 경우가 발생하기 쉽습니다. 또한, 단위 테스트가 거의 불가능해집니다. ROS 로직을 테스트하려면 전체 GUI를 인스턴스화해야 하고, GUI 로직을 테스트하려면 실제 ROS 시스템이 필요하기 때문입니다.23 MVC를 채택하면 모델(ROS 노드)은 독립적으로 테스트할 수 있고, 뷰(Qt 위젯)는 모의(mock) 데이터로 테스트할 수 있으며, 둘을 연결하는 컨트롤러의 로직 또한 격리된 상태에서 테스트가 가능합니다. 이러한 아키텍처적 분리는 UI/UX 전문가와 로보틱스 엔지니어의 병렬 개발을 가능하게 하여 팀의 작업 흐름을 개선하고 더 견고한 최종 제품을 만드는 데 직접적으로 기여합니다.29

### 3.2  복잡한 사용자 인터페이스를 위한 상태 관리

단순한 GUI는 불리언(boolean) 플래그 몇 개로 상태를 관리할 수 있지만, 애플리케이션이 복잡해지면 이러한 방식은 금방 관리 불가능하고 버그를 유발하는 코드로 변질됩니다 (예: `if (isNavigating &&!isCharging && isConnected)...`).47 따라서 복잡한 UI 흐름을 제어하기 위해서는 정형화된 상태 관리 패턴을 도입해야 합니다.

- **유한 상태 기계 (Finite State Machines, FSMs):** FSM은 시스템이 가질 수 있는 상태가 `IDLE`, `NAVIGATING`, `DOCKING`, `ERROR`처럼 명확하게 정의된 유한한 개수이고, 상태 간의 전이 조건이 명확할 때 이상적입니다.47 GUI는 FSM을 사용하여 로봇의 현재 상태에 따라 특정 위젯을 활성화하거나 비활성화할 수 있습니다. 예를 들어, 로봇의 상태가 이미 

  `NAVIGATING`이라면 "내비게이션 시작" 버튼을 비활성화하는 식입니다. ROS2 환경에서는 `YASMIN`과 같은 라이브러리가 FSM 구현을 제공하며, 각 상태가 토픽, 서비스, 액션과 상호작용할 수 있도록 지원합니다.49 산업 자동화 분야의 표준인 PackML 역시 FSM을 기반으로 하며, ROS2로 포팅되었습니다.50

- **행동 트리 (Behavior Trees, BTs):** BT는 복잡하고 반응적인 작업을 처리하는 데 있어 FSM보다 더 유연하고 확장성이 뛰어납니다. BT는 액션, 조건 등 다양한 노드들로 구성된 트리 구조이며, 주기적으로 "틱(tick)"을 받아 로봇의 행동을 결정합니다. 본질적으로 모듈화 및 조합이 용이합니다.48 주로 로봇 자율주행 로직에 사용되지만, 각 단계가 트리의 노드에 해당하는 복잡한 UI 마법사(wizard)를 구동하는 데에도 영감을 줄 수 있습니다. 

  `BehaviorTree.CPP` 라이브러리는 트리를 시각화하고 편집할 수 있는 `Groot`라는 GUI 도구를 함께 제공하는데, 이는 커스텀 GUI 개발에 참고할 만한 좋은 예시입니다.48

- **ROS2 생명주기 노드 (Lifecycle Nodes):** 이는 ROS2에 내장된 특수한 형태의 FSM으로, `UNCONFIGURED`, `INACTIVE`, `ACTIVE`, `FINALIZED`와 같은 표준화된 상태를 가집니다. GUI는 이러한 생명주기 노드와 상호작용하여 복잡한 시스템의 시작 및 종료 시퀀스를 안전하고 예측 가능하게 관리할 수 있습니다.50

### 3.3  데이터 동기화 및 스레드 안전성 확보

앞서 여러 번 강조했듯이, GUI가 아닌 스레드에서 GUI 요소를 직접 업데이트하는 것은 불안전하며 반드시 피해야 합니다.17

- **표준적인 해결책: 시그널과 슬롯 (Qt):** Qt의 시그널-슬롯 메커니즘은 스레드 안전성을 보장하는 콜백 시스템입니다. 워커 스레드에서 실행되는 ROS 콜백은 수신한 데이터를 담아 시그널을 `emit`합니다. 이 시그널은 메인 `QWidget` 또는 `QMainWindow` 클래스에 정의된 슬롯에 연결됩니다. Qt의 이벤트 루프는 이 슬롯이 GUI의 메인 스레드에서 실행되도록 보장하므로, 위젯을 안전하게 업데이트할 수 있습니다.14 이 보고서는 이 패턴을 명시적으로 보여주는 C++ 및 Python 코드 예제를 제공할 것입니다.
- **데이터 동기화와 시간:**
  - **`use_sim_time`:** 시뮬레이션 데이터나 `rosbag` 재생 데이터로 작업할 때, GUI를 포함한 모든 노드가 공통된 시간 소스를 사용하는 것이 매우 중요합니다. `use_sim_time` 파라미터를 `true`로 설정하면 노드들은 시스템의 실제 시간(wall clock) 대신 `/clock` 토픽을 구독하여 시간을 동기화합니다.51 타임스탬프를 표시하는 GUI는 이 파라미터를 반드시 존중해야 시스템의 다른 부분과 정확하게 동기화될 수 있습니다.
  - **`message_filters`:** GUI가 서로 다른 시간에 발행되는 여러 토픽의 데이터를 동기화하여 표시해야 할 경우(예: 이미지와 해당 이미지의 카메라 정보를 함께 처리), `message_filters` 라이브러리를 사용할 수 있습니다. 이 라이브러리는 각 메시지의 타임스탬프를 기반으로 여러 토픽의 메시지들을 동기화하여 하나의 콜백 함수에서 함께 처리할 수 있도록 해줍니다.52

### 3.4  사례 연구: MoveIt2 RViz 플러그인 아키텍처

MoveIt2의 RViz용 모션 플래닝 플러그인은 대규모 프레임워크에 통합된 복잡한 GUI의 대표적인 예시입니다. 이 플러그인은 `move_group` 노드에 대한 풍부한 기능의 프론트엔드 역할을 합니다.53

- **인터랙티브 마커를 통한 사용자 상호작용:**

  - **메커니즘:** 플러그인은 `visualization_msgs/InteractiveMarker`를 사용하여 로봇의 엔드 이펙터(end-effector)에 6자유도(6-DOF) 제어 핸들을 생성합니다.56
  - **데이터 흐름:** 사용자가 이 마커를 드래그하면, RViz 플러그인은 로봇을 직접 움직이는 것이 아니라 마커의 새로운 포즈(pose)를 캡처합니다. 이 포즈는 모션 플래닝 요청의 `goal_state`가 됩니다.58
  - **"스냅백(Snap-Back)" 문제:** 사용자가 마커를 놓았을 때 마커가 원래 위치로 돌아가는 흔한 문제는 이 아키텍처를 잘 보여줍니다. 이 현상은 모션 플래닝 요청이 실패했을 때(예: 목표 지점이 도달 불가능하거나 충돌 상태일 때) 또는 설정 문제(예: 키네마틱스 플러그인이 로드되지 않았거나 `use_sim_time` 설정 오류)가 있을 때 발생합니다. 마커의 위치는 그 자체로 진실이 아니라, 백엔드에 의해 검증되어야 할 '요청'임을 의미합니다.58

- **`move_group`과의 통신:**

  - 플러그인은 ROS2 액션(예: `MoveGroupAction`)을 사용하여 인터랙티브 마커로부터 얻은 목표 포즈가 포함된 플래닝 요청을 `move_group` 노드로 전송합니다.
  - `move_group`은 플래닝을 수행하고, 성공하면 경로(trajectory)를 반환합니다. 플러그인은 이 경로를 수신하여 RViz에 시각화합니다(초록색 시작 상태, 주황색 목표 상태, 그리고 계획된 경로).53

- **코드 수준 분석:** 이 플러그인은 `rviz_common::Panel`을 상속받는 C++ 클래스로 구현되어 있습니다. 내부적으로 `rclcpp`를 사용하여 ROS2 노드를 생성하고 통신하며 61, 

  `moveit_visual_tools` 라이브러리를 활용하여 다양한 시각화 및 상호작용 기능을 구현합니다.63 소스 코드를 분석하면, 플러그인이 

  `/monitored_planning_scene`과 같은 토픽을 구독하여 현재 월드 상태를 표시하는 방식을 확인할 수 있습니다.57

## 4.  배포 및 최종 고려사항

ROS2 GUI 애플리케이션의 개발이 완료되면, 이를 실제 로봇이나 원격 시스템에 안정적으로 배포하고 운영하는 단계가 남아있습니다. 이 장에서는 애플리케이션의 패키징, 의존성 관리, 컨테이너화를 통한 배포 전략을 다루고, 전체적인 분석을 종합하여 결론과 향후 전망을 제시합니다.

### 4.1  패키징 및 배포 전략

성공적인 배포는 개발 환경의 복잡성을 안정적으로 재현하는 것에서 시작됩니다. ROS2 생태계는 이를 위한 표준화된 도구와 워크플로우를 제공합니다.

- **빌드 및 의존성 관리:**
  - **`colcon`:** ROS2 워크스페이스를 빌드하는 표준 명령줄 도구입니다. GUI 패키지 역시 `colcon`을 통해 컴파일되고 설치됩니다.28
  - **`package.xml`:** 패키지의 메타데이터를 정의하는 핵심 파일입니다. 이 파일에는 빌드, 실행, 테스트에 필요한 모든 의존성을 정확하게 명시해야 합니다. 여기에는 `rclpy`, `rqt_gui_py`와 같은 ROS 패키지뿐만 아니라, `python3-pyqt5`와 같은 시스템 라이브러리도 포함됩니다.27
  - **`rosdep`:** `package.xml`에 명시된 시스템 수준의 의존성(예: Qt, GTK 라이브러리)을 다양한 운영체제에서 자동으로 설치해주는 도구입니다. 이를 통해 다른 환경에서도 손쉽게 빌드 환경을 구축할 수 있습니다.31
- **컨테이너화를 통한 배포 (Docker):**
  - **장점:** 도커(Docker)는 애플리케이션과 그 모든 의존성을 컨테이너라는 격리된 환경에 패키징하여, "내 컴퓨터에서는 잘 됐는데..." 문제를 근본적으로 해결합니다. 이는 일관되고 이식성 높은 배포를 가능하게 하여, 복잡한 의존성을 가진 ROS2 애플리케이션 배포에 매우 효과적입니다.42
  - **GUI 애플리케이션의 과제:** 도커 컨테이너 내부에서 GUI 애플리케이션을 실행하는 것은 추가적인 설정이 필요합니다. 호스트 시스템의 X11 소켓을 컨테이너에 전달하거나, VNC, Xpra와 같은 다른 디스플레이 서버 기술을 사용해야 합니다. 이는 초보자에게는 다소 까다로운 작업일 수 있습니다.66
  - **표준 배포 워크플로우:** 일반적인 프로덕션 배포 과정은 다음과 같습니다.
    1. 로컬 환경에서 개발 및 테스트를 완료합니다.
    2. `Dockerfile`을 작성합니다. 이 파일은 베이스 ROS2 이미지를 기반으로, 애플리케이션 소스 코드를 복사하고, `rosdep`으로 의존성을 설치한 후, `colcon`으로 빌드하는 과정을 자동화합니다.42
    3. 다단계 빌드(multi-stage builds)를 사용하여 최종 배포용 이미지를 가볍고 안전하게 만듭니다.65
    4. 빌드된 이미지를 도커 허브(Docker Hub)나 AWS ECR과 같은 컨테이너 레지스트리에 푸시합니다.
    5. 목표 로봇 또는 원격 스테이션에서 해당 이미지를 풀(pull)하여 컨테이너를 실행합니다.42
- **보안 고려사항:** 보안이 활성화된 ROS2 시스템과 연동되는 GUI를 배포할 때는, 필요한 보안 관련 파일(키, 인증서, 권한 파일 등)이 최소 권한 원칙에 따라 컨테이너 또는 호스트 시스템에 정확하게 패키징되고 배포되어야 합니다. 민감한 개인 키 등은 배포 대상에 포함되지 않도록 주의해야 합니다.68

### 4.2  결론 및 향후 전망

본 보고서는 ROS2 리소스를 활용한 GUI 애플리케이션 개발에 대한 다각적이고 심층적인 분석을 제공했습니다. ROS2의 통신 및 실행 모델을 이해하는 것부터 시작하여, `rqt` 플러그인, 독립 실행형 애플리케이션, 웹 기반 인터페이스라는 세 가지 주요 개발 접근법의 아키텍처와 장단점을 비교 분석했습니다. 또한, MVC, 상태 기계와 같은 고급 아키텍처 패턴과 스레딩을 통한 UI 반응성 확보 기법을 탐구했으며, 최종적으로 배포 전략까지 살펴보았습니다.

분석을 통해 도출된 핵심 결론은 '최고의 단일 해결책'은 존재하지 않는다는 것입니다. 최적의 아키텍처 선택은 프로젝트의 구체적인 요구사항, 개발팀의 기술 역량, 그리고 최종 배포 환경이라는 세 가지 변수에 의해 결정되는 함수입니다.

- **개발자 도구나 빠른 프로토타입**이 목적이라면, `rqt` 플러그인은 기존 생태계를 활용하여 가장 높은 생산성을 제공합니다.
- **고도로 맞춤화된 사용자 경험과 성능이 중요한 상용 제품**을 개발한다면, 동시성 처리의 복잡성을 감수하고 독립 실행형 애플리케이션을 선택하는 것이 타당합니다.
- **원격 접근성, 플랫폼 독립성, 클라우드 통합**이 핵심 요구사항이라면, 웹 기반 인터페이스가 가장 강력한 솔루션입니다.

로보틱스 분야의 GUI 개발은 계속해서 진화하고 있으며, 다음과 같은 미래 동향을 주목할 필요가 있습니다.

- **웹 UI의 부상:** 웹 기술의 보편성과 접근성, 그리고 React, Vue와 같은 강력한 프레임워크 생태계 덕분에 웹 기반 UI가 점차 표준으로 자리 잡고 있습니다. `foxglove_bridge`와 같은 고성능 브릿지는 이러한 추세를 더욱 가속화하고 있습니다.25

- **선언적 UI (Declarative UI):** Qt의 QML이나, 다른 생태계의 SwiftUI/Jetpack Compose와 같은 선언적 UI 프레임워크는 UI의 상태와 모양을 코드로 직접 기술하여, 기존의 명령형 위젯 조작 방식보다 UI 개발과 상태 관리를 훨씬 간결하게 만듭니다. 이러한 패러다임이 ROS 생태계에도 점차 영향을 미칠 것으로 예상됩니다.

- **Rust의 도입:** 안전성과 성능을 강점으로 하는 Rust 언어가 로보틱스 스택에서 주목받고 있습니다. `Iced`, `Slint`, `Tauri`와 같은 Rust 네이티브 GUI 프레임워크가 성숙하고 있으며, Qt 바인딩도 존재합니다.69

  `r2r`과 같은 새로운 클라이언트 라이브러리는 기존과는 다른 실행 모델을 제시하며, 이는 향후 GUI 아키텍처에도 새로운 과제를 던져줄 것입니다.9

궁극적으로, 이 보고서가 제공하는 심층적인 분석과 비교는 개발자가 단순히 코드를 작성하는 것을 넘어, 자신의 프로젝트에 가장 적합한 아키텍처를 정보에 입각하여 현명하게 선택하는 데 필요한 지식과 통찰력을 제공하는 것을 목표로 합니다. 성공적인 로봇 애플리케이션은 강력한 기능뿐만 아니라, 그 기능을 사용자와 효과적으로 연결하는 잘 설계된 인터페이스 위에서 완성되기 때문입니다.

#### **참고 자료**

1. Concepts - ROS 2 Documentation, accessed July 16, 2025, https://ros2-documentation-zh-cn.readthedocs.io/en/rolling/source/Concepts.html
2. Concepts — ROS 2 Documentation: Foxy documentation, accessed July 16, 2025, https://docs.ros.org/en/foxy/Concepts.html
3. Topics vs Services vs Actions — ROS 2 Documentation: Foxy ..., accessed July 16, 2025, https://docs.ros.org/en/foxy/How-To-Guides/Topics-Services-Actions.html
4. antmicro/ros2-gui-node - GitHub, accessed July 16, 2025, https://github.com/antmicro/ros2-gui-node
5. Creating a RQT Dashboard — ROS Tutorials 0.5.2 documentation - Clearpath Robotics, accessed July 16, 2025, [https://www.clearpathrobotics.com/assets/guides/kinetic/ros/Creating%20RQT%20Dashboard.html](https://www.clearpathrobotics.com/assets/guides/kinetic/ros/Creating RQT Dashboard.html)
6. ROS's RQt - PAL OS 25.01 documentation, accessed July 16, 2025, https://docs.pal-robotics.com/edge/management/rqt
7. Synchronous vs. asynchronous service clients — ROS 2 Documentation, accessed July 16, 2025, https://docs.ros.org/en/foxy/How-To-Guides/Sync-Vs-Async.html
8. Actions - ROS2 Design, accessed July 16, 2025, https://design.ros2.org/articles/actions.html
9. A first look at ROS 2 applications written in asynchronous Rust - arXiv, accessed July 16, 2025, https://arxiv.org/html/2505.21323v1
10. ROS2 - Manual Composition with Multi Threaded Executor - YouTube, accessed July 16, 2025, https://www.youtube.com/watch?v=amQzXVkR7lY
11. DLu/toy_threads: Some toy examples to demonstrate threading in `rclpy`. - GitHub, accessed July 16, 2025, https://github.com/DLu/toy_threads
12. Using Callback Groups — ROS 2 Documentation: Galactic documentation, accessed July 16, 2025, https://docs.ros.org/en/galactic/How-To-Guides/Using-callback-groups.html
13. Using Callback Groups — ROS 2 Documentation: Iron documentation, accessed July 16, 2025, https://docs.ros.org/en/iron/How-To-Guides/Using-callback-groups.html
14. ROS2 + Qt: any good solution? - Stack Overflow, accessed July 16, 2025, https://stackoverflow.com/questions/61060188/ros2-qt-any-good-solution
15. making a ros node from a qt thread - Stack Overflow, accessed July 16, 2025, https://stackoverflow.com/questions/41041341/making-a-ros-node-from-a-qt-thread
16. ROS-Qt GUI - How to distribute the Threads? - Stack Overflow, accessed July 16, 2025, https://stackoverflow.com/questions/54464488/ros-qt-gui-how-to-distribute-the-threads
17. Thread: How to solve GUI freezing problem - Qt Centre Forum, accessed July 16, 2025, https://www.qtcentre.org/threads/53697-How-to-solve-GUI-freezing-problem
18. rqt - ROS Wiki, accessed July 16, 2025, http://wiki.ros.org/rqt
19. ros2_documentation/source/Concepts/Intermediate/About-RQt.rst at rolling - GitHub, accessed July 16, 2025, https://github.com/ros2/ros2_documentation/blob/rolling/source/Concepts/Intermediate/About-RQt.rst
20. Learn ROS 2 - Intro: RQt - Hadabot, accessed July 16, 2025, https://www.hadabot.com/learn-ros2-intro-rqt.html?step=what-is-rqt
21. Implementing GUI Based Tools Using rqt - Machine Intelligence Laboratory, accessed July 16, 2025, https://mil.ufl.edu/docs/software/rqt.html
22. Overview and usage of RQt — ROS 2 Documentation: Foxy documentation, accessed July 16, 2025, https://docs.ros.org/en/foxy/Concepts/About-RQt.html
23. Create your rqt plugin package - ROS Wiki, accessed July 16, 2025, [http://wiki.ros.org/rqt/Tutorials/Create%20your%20new%20rqt%20plugin](http://wiki.ros.org/rqt/Tutorials/Create your new rqt plugin)
24. rqt/Tutorials/Writing a Python Plugin - ROS Wiki, accessed July 16, 2025, [http://wiki.ros.org/rqt/Tutorials/Writing%20a%20Python%20Plugin](http://wiki.ros.org/rqt/Tutorials/Writing a Python Plugin)
25. Development of a user-friendly ROS2 GTK4 GUI with Adwaita theme - ROS Discourse, accessed July 16, 2025, https://discourse.ros.org/t/development-of-a-user-friendly-ros2-gtk4-gui-with-adwaita-theme/41605
26. RQT in ROS2 - ROS General - Open Robotics Discourse, accessed July 16, 2025, https://discourse.ros.org/t/rqt-in-ros2/6428
27. ChoKasem/rqt_tut: custom rqt plugin tutorial - GitHub, accessed July 16, 2025, https://github.com/ChoKasem/rqt_tut
28. How can I create a simple GUI using an RQT plugin? - Stack Overflow, accessed July 16, 2025, https://stackoverflow.com/questions/77934193/how-can-i-create-a-simple-gui-using-an-rqt-plugin
29. A simple way to create GUIs for your ROS 2 project, accessed July 16, 2025, https://discourse.ros.org/t/a-simple-way-to-create-guis-for-your-ros-2-project/41847
30. Simple ROS2-PyQt-GUI for robot control - GitHub, accessed July 16, 2025, https://github.com/KimJiHong190/ros2_controller_gui
31. Abishalini/ROS2QtGui: Creating custom GUI with Qt and ROS2 - GitHub, accessed July 16, 2025, https://github.com/Abishalini/ROS2QtGui
32. ROS2 User Interface : r/ROS - Reddit, accessed July 16, 2025, https://www.reddit.com/r/ROS/comments/1j8t8px/ros2_user_interface/
33. Ros2 custom gui with pyqt5 : r/ROS - Reddit, accessed July 16, 2025, https://www.reddit.com/r/ROS/comments/1iuu0m4/ros2_custom_gui_with_pyqt5/
34. Threads when using ros2 and python - ROS Answers archive, accessed July 16, 2025, https://answers.ros.org/question/386193/
35. ROS2 how to do multi-threading, subscription callbacks, spinning , executors, accessed July 16, 2025, https://robotics.stackexchange.com/questions/101788/ros2-how-to-do-multi-threading-subscription-callbacks-spinning-executors
36. Using Rosbridge with ROS 2 - Foxglove, accessed July 16, 2025, https://foxglove.dev/blog/using-rosbridge-with-ros2
37. ROS 2 | Foxglove Docs, accessed July 16, 2025, https://docs.foxglove.dev/docs/getting-started/ros2
38. ROS Foxglove bridge, accessed July 16, 2025, https://docs.foxglove.dev/docs/getting-started/ros-foxglove-bridge
39. Visualizing ROS 2 data with Foxglove, accessed July 16, 2025, https://docs.ros.org/en/rolling/Related-Projects/Visualizing-ROS-2-Data-With-Foxglove.html
40. How to start creating a GUI in Ros? - Reddit, accessed July 16, 2025, https://www.reddit.com/r/ROS/comments/r0yuny/how_to_start_creating_a_gui_in_ros/
41. User Interfaces - Programming Multiple Robots with ROS 2 - GitHub Pages, accessed July 16, 2025, https://osrf.github.io/ros2multirobotbook/ui.html
42. The Complete Beginner's Guide to Docker for ROS 2 Deployment (2025) - Robotair, accessed July 16, 2025, https://blog.robotair.io/the-complete-beginners-guide-to-using-docker-for-ros-2-deployment-2025-edition-0f259ca8b378
43. Model/View Programming | Qt Widgets | Qt 6.9.1, accessed July 16, 2025, https://doc.qt.io/qt-6/model-view-programming.html
44. MVC Example in Qt - Qt Forum, accessed July 16, 2025, https://forum.qt.io/topic/65074/mvc-example-in-qt
45. Has anyone created a proper MVC pattern using Qt? : r/QtFramework - Reddit, accessed July 16, 2025, https://www.reddit.com/r/QtFramework/comments/fmlw2o/has_anyone_created_a_proper_mvc_pattern_using_qt/
46. The Model View Controller (MVC) Pattern - YouTube, accessed July 16, 2025, https://www.youtube.com/watch?v=cZfcI6GyDxU
47. State · Design Patterns Revisited - Game Programming Patterns, accessed July 16, 2025, https://gameprogrammingpatterns.com/state.html
48. State Machines vs Behavior Trees: designing a decision-making architecture for robotics, accessed July 16, 2025, https://www.polymathrobotics.com/blog/state-machines-vs-behavior-trees
49. uleroboticsgroup/yasmin: YASMIN (Yet Another State MachINe) - GitHub, accessed July 16, 2025, https://github.com/uleroboticsgroup/yasmin
50. PACKML2: STATE MACHINE BASED SYSTEM PROGRAMMING, MONITORING AND CONTROL IN ROS2, accessed July 16, 2025, https://roscon.ros.org/2019/talks/roscon2019_packml2.pdf
51. Clock and Time - ROS2 Design, accessed July 16, 2025, https://design.ros2.org/articles/clock_and_time.html
52. alsora/ros2-code-examples: Collection of tutorials and examples for the Robot Operating System ROS 2 - GitHub, accessed July 16, 2025, https://github.com/alsora/ros2-code-examples
53. Concepts | MoveIt, accessed July 16, 2025, https://moveit.ai/documentation/concepts/
54. The move_group node — MoveIt Documentation: Rolling ..., accessed July 16, 2025, https://moveit.picknik.ai/main/doc/concepts/move_group.html
55. MoveIt Quickstart in RViz — moveit_tutorials Kinetic documentation - ROS 2, accessed July 16, 2025, http://docs.ros.org/en/kinetic/api/moveit_tutorials/html/doc/quickstart_in_rviz/quickstart_in_rviz_tutorial.html
56. rviz/Tutorials/Interactive Markers: Getting Started - ROS Wiki, accessed July 16, 2025, [http://wiki.ros.org/rviz/Tutorials/Interactive%20Markers%3A%20Getting%20Started](http://wiki.ros.org/rviz/Tutorials/Interactive Markers%3A Getting Started)
57. MoveIt Quickstart in RViz — MoveIt Documentation: Humble documentation, accessed July 16, 2025, https://moveit.picknik.ai/humble/doc/tutorials/quickstart_in_rviz/quickstart_in_rviz_tutorial.html
58. Unable to move MoveIt2 interactive markers - Robotics Stack Exchange, accessed July 16, 2025, https://robotics.stackexchange.com/questions/116945/unable-to-move-moveit2-interactive-markers
59. (Humble, Jazzy, Rolling) Interactive Markers do not display · Issue #3339 · moveit/moveit2, accessed July 16, 2025, https://github.com/moveit/moveit2/issues/3339
60. Interactive Marker does not update pose in RViz (returns to initial position on release) · Issue #3475 · moveit/moveit2 - GitHub, accessed July 16, 2025, https://github.com/moveit/moveit2/issues/3475
61. moveit2/moveit_ros/visualization/motion_planning_rviz_plugin/src/motion_planning_frame.cpp at main - GitHub, accessed July 16, 2025, https://github.com/ros-planning/moveit2/blob/master/moveit_ros/visualization/motion_planning_rviz_plugin/src/motion_planning_frame.cpp
62. moveit2: moveit_setup_assistant/moveit_setup_framework/src/rviz_panel.cpp Source File - MoveIt Documentation, accessed July 16, 2025, https://moveit.picknik.ai/main/api/html/rviz__panel_8cpp_source.html
63. Visualizing In RViz - MoveIt Documentation - PickNik Robotics, accessed July 16, 2025, https://moveit.picknik.ai/main/doc/tutorials/visualizing_in_rviz/visualizing_in_rviz.html
64. ros2_documentation/source/Tutorials/Beginner-Client-Libraries/Custom-ROS2-Interfaces.rst at rolling - GitHub, accessed July 16, 2025, https://github.com/ros2/ros2_documentation/blob/rolling/source/Tutorials/Beginner-Client-Libraries/Custom-ROS2-Interfaces.rst
65. The Complete Guide to Deploying ROS Applications in Production - Robotair, accessed July 16, 2025, https://blog.robotair.io/the-complete-guide-to-deploying-ros-applications-in-production-07a652a2e2f8
66. Automatic Deployment of ROS2 Based System to remote devices: Dual Copy or Containers?, accessed July 16, 2025, https://discourse.ros.org/t/automatic-deployment-of-ros2-based-system-to-remote-devices-dual-copy-or-containers/33884
67. MOGI-ROS/Week-1-2-Introduction-to-ROS2 - GitHub, accessed July 16, 2025, https://github.com/MOGI-ROS/Week-1-2-Introduction-to-ROS2
68. Deployment Guidelines — ROS 2 Documentation: Humble documentation, accessed July 16, 2025, https://docs.ros.org/en/humble/Tutorials/Advanced/Security/Deployment-Guidelines.html
69. I'm working on a GUI app. How does Qt compare to GTK in Rust? - Reddit, accessed July 16, 2025, https://www.reddit.com/r/rust/comments/1j0gwb2/im_working_on_a_gui_app_how_does_qt_compare_to/
70. A 2025 Survey of Rust GUI Libraries | boringcactus, accessed July 16, 2025, https://www.boringcactus.com/2025/04/13/2025-survey-of-rust-gui-libraries.html




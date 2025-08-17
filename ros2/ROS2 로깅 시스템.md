# ROS2 로깅 시스템

## 서론

### 0.1 ROS2 시스템에서 로깅의 본질적 가치와 역할 재정의

로보틱스 시스템, 특히 ROS2(Robot Operating System 2)와 같이 다수의 노드가 비동기적으로 통신하는 분산 환경에서 로깅(logging)은 단순한 `printf` 디버깅의 확장을 넘어선다. 이는 시스템의 상태를 실시간으로 관찰하고, 예측 불가능한 환경에서 발생하는 이벤트의 순서를 재구성하며, 전체 시스템의 동작을 검증하는 핵심적인 관측 가능성(Observability) 도구이다. 효과적인 로깅 전략은 개발 및 디버깅 시간을 단축시킬 뿐만 아니라, 배포된 시스템의 안정성과 유지보수성을 결정하는 중요한 요소로 작용한다. 로깅은 시스템의 내부 상태를 외부로 투영하는 유일한 창이며, 이를 통해 개발자와 운영자는 복잡한 상호작용의 결과를 이해하고 잠재적인 문제를 사전에 식별할 수 있다.

### 0.2 보고서의 목적과 구조

현재 ROS2 로깅에 대한 정보는 공식 문서, 튜토리얼, 커뮤니티 포럼 등 여러 곳에 파편적으로 흩어져 있어 시스템의 전체적인 그림을 이해하기 어렵다. 본 보고서는 이러한 파편화된 정보를 종합하고, ROS2 로깅 시스템의 근본 원리부터 내부 아키텍처, 다양한 제어 방법론, 그리고 대규모 시스템을 위한 고급 활용 전략까지 체계적으로 분석하는 것을 목표로 한다. 이를 통해 독자는 ROS2 로깅 시스템을 완벽하게 통제하고, 자신의 프로젝트에 최적화된 로깅 전략을 수립하며, 발생 가능한 문제를 효율적으로 해결할 수 있는 깊이 있는 지식을 습득하게 될 것이다.

보고서는 총 5개의 장으로 구성된다. 제1장에서는 로깅의 기본 개념인 심각도 수준, 로거 이름, 레벨 상속을 다룬다. 제2장에서는 `rclcpp`와 `rclpy` 클라이언트 라이브러리를 사용한 실제 로깅 코드 작성법과 고급 필터링 기법을 분석한다. 제3장에서는 환경 변수, 명령줄 인수, 런치 파일, 런타임 API 등 다양한 구성 방법과 그들 사이의 우선순위를 명확히 규명한다. 제4장에서는 로깅 스택의 내부 아키텍처를 심층적으로 분석하고 ROS1과의 차이점을 비교한다. 마지막으로 제5장에서는 대규모 시스템에서의 실용적인 디버깅 및 분석 전략과 모범 사례를 제시한다.

------

## 1.  ROS2 로깅의 근본 원리 (Fundamental Principles of ROS2 Logging)

ROS2 로깅 시스템의 효과적인 활용은 그 근간을 이루는 세 가지 핵심 개념, 즉 '심각도 수준(Severity Levels)', '로거 이름(Logger Names)', 그리고 '레벨 상속(Level Inheritance)'에 대한 깊은 이해에서 시작된다. 이 세 요소는 서로 유기적으로 작용하여 개발자가 시스템의 어느 부분에서, 얼마나 상세한 정보를, 어떤 목적으로 기록할지를 정밀하게 제어할 수 있도록 지원한다.

### 1.1  심각도 수준(Severity Levels)의 이해와 전략적 활용

ROS2의 모든 로그 메시지는 심각도 수준과 연관되며, 이는 메시지의 중요성과 의도를 나타내는 핵심적인 속성이다. 로거는 설정된 심각도 수준 이상의 메시지만 처리하므로, 이를 통해 출력되는 정보의 양을 효과적으로 제어할 수 있다.1 ROS2는 다음과 같이 오름차순으로 정렬된 5가지 표준 심각도 수준을 정의한다.2

- **`DEBUG`**: 시스템의 내부 상태, 변수 값, 함수의 단계별 실행 과정 등 개발 및 디버깅 단계에서만 필요한 매우 상세한 정보를 기록하는 데 사용된다.5 이 레벨의 로그는 기본적으로 비활성화되어 있으며, 운영 환경의 성능에 영향을 미치지 않도록 설계되었다. 문제가 발생했을 때 특정 모듈의 동작을 면밀히 추적하기 위해 일시적으로 활성화된다.2
- **`INFO`**: 노드의 시작 및 종료, 주요 상태 전이, 서비스 요청 처리 완료 등 시스템의 정상적인 동작 과정에서 발생하는 중요한 이벤트를 알리는 데 사용된다.5 운영자가 시스템이 의도대로 동작하고 있음을 시각적으로 확인하는 용도로 적합하며, 기본 로깅 레벨로 설정되어 있다.2
- **`WARN`**: 예상치 못한 상황이나 잠재적인 문제를 나타내지만, 시스템의 기능이 즉각적으로 중단되지는 않는 경우에 사용된다.6 예를 들어, 파라미터 값이 권장 범위를 벗어났거나, 특정 센서 데이터가 일시적으로 수신되지 않는 경우 등이 해당된다. 즉각적인 조치는 필요 없으나 운영자의 주의가 요구되는 상황을 알린다.
- **`ERROR`**: 특정 기능이 명백히 실패했으나, 전체 노드나 시스템은 계속 동작을 시도하는 심각한 문제를 보고하는 데 사용된다.6 예를 들어, 필수적인 파일 로드 실패, 네트워크 연결 끊김 등이 여기에 해당한다.
- **`FATAL`**: 노드 또는 시스템 전체의 즉각적인 종료를 유발하거나, 더 이상의 정상적인 작동을 보장할 수 없는 치명적인 오류를 나타낸다.6 이 레벨의 로그가 발생하면 시스템은 스스로를 보호하기 위해 종료 절차에 들어갈 수 있다.

이러한 심각도 수준은 단순히 정보의 중요도를 나누는 필터링 기준을 넘어, '로그의 소비자'를 정의하는 일종의 커뮤니케이션 프로토콜로 이해해야 한다. 각 레벨은 서로 다른 대상과 목적을 가진다. `DEBUG` 레벨은 오직 개발자를 위한 것이며, 시스템의 가장 깊은 내부를 들여다보기 위한 상세한 정보를 담는다. `INFO` 레벨은 시스템을 운영하고 모니터링하는 운영자를 대상으로 하며, 시스템이 정상적으로 살아있고 주요 작업을 수행하고 있음을 알리는 '심장 박동'과 같은 역할을 한다. `WARN`, `ERROR`, `FATAL` 레벨은 시스템 관리자나 자동화된 알림 시스템을 대상으로 하며, 인간의 개입이나 즉각적인 조치가 필요한 비정상 상황을 알리는 신호로 작용한다. 이러한 관점에서 로깅 정책을 수립하면, 코드를 작성하는 개발자는 "이 메시지는 누가, 어떤 목적으로 소비할 것인가?"라는 질문을 통해 자연스럽게 가장 적절한 심각도 수준을 선택할 수 있다. 이는 결과적으로 코드의 가독성을 높이고, 대규모 시스템의 유지보수성을 크게 향상시키는 실용적인 설계 원칙이 된다.

### 1.2  로거 이름과 계층 구조: 모듈성의 핵심

ROS2 로깅 시스템의 또 다른 핵심 축은 '로거 이름'과 그 이름이 형성하는 '계층 구조'이다. 모든 로그 메시지는 특정 이름을 가진 로거에 의해 생성되며, 이 이름은 로그의 출처를 명확히 식별하는 역할을 한다.

- **노드 기반 로거**: 모든 ROS2 노드는 `get_logger()` 메서드를 호출할 때 자신의 노드 이름과 네임스페이스를 자동으로 포함하는 로거를 얻는다.1 예를 들어, 

  `my_namespace` 네임스페이스에서 `my_node`라는 이름으로 실행되는 노드의 로거 이름은 `my_namespace.my_node`가 될 수 있다. 만약 `ros2 run`이나 런치 파일에서 노드 이름이 외부적으로 재매핑(remapping)된다면, 이 변경 사항은 로거 이름에도 동적으로 반영되어 코드 수정 없이도 일관성을 유지한다.7

- **사용자 정의 로거**: `rclcpp::get_logger("이름")` (C++) 또는 `rclpy.logging.get_logger("이름")` (Python)을 통해 특정 노드에 종속되지 않는 독립적인 로거를 생성할 수 있다.9 이는 노드 컨텍스트 외부의 라이브러리나 유틸리티 클래스에서 로깅을 수행할 때 유용하다.

- **계층 구조**: 로거 이름에서 점(`.`)은 디렉터리 경로의 슬래시(`/`)와 유사하게 계층 구조를 정의하는 구분자로 사용된다.1 예를 들어, 

  `control.motion_planner.local`이라는 이름의 로거는 `control.motion_planner` 로거의 자식이며, 이는 다시 `control` 로거의 자식이 된다.10

이러한 로거의 계층 구조는 ROS2의 컴포넌트 기반 설계 철학을 로깅 시스템에 직접적으로 반영한 결과물이다. 이는 단순히 로그를 그룹화하는 기능을 넘어, 시스템의 논리적 구조를 로깅 시스템에 그대로 투영시키는 강력한 아키텍처 패턴이다. 예를 들어, 하나의 `navigation` 노드 프로세스 내에 `global_planner`와 `local_planner`라는 두 개의 논리적 컴포넌트가 함께 실행될 수 있다. 이때 각 컴포넌트가 `navigation.global_planner`와 `navigation.local_planner`라는 계층적 로거 이름을 사용한다면, 개발자는 출력된 로그 메시지만 보고도 어떤 서브 모듈에서 문제가 발생했는지 즉시 파악할 수 있다. 더 나아가, 이 계층 구조는 다음 섹션에서 설명할 '레벨 상속' 메커니즘과 결합하여, `navigation`이라는 상위 로거의 레벨을 `DEBUG`로 설정하는 단 한 번의 조작으로 전체 내비게이션 스택의 상세 동작을 관찰하는 등, 특정 서브시스템 전체의 동작을 세밀하게 관찰하고 제어하는 것을 가능하게 한다. 결국, 로거 계층은 소프트웨어의 모듈성과 시스템의 관측 가능성을 연결하는 다리 역할을 하며, 복잡성이 높은 대규모 로봇 시스템을 관리하는 데 필수적인 도구가 된다.

### 1.3  레벨 상속(Level Inheritance) 메커니즘

레벨 상속은 로거의 계층 구조를 더욱 강력하게 만드는 핵심 메커니즘이다. 이를 통해 개발자는 최소한의 설정으로 시스템 전체의 로깅 상세도를 유연하게 조절할 수 있다.

상속 규칙은 간단하다. 특정 로거의 심각도 수준이 명시적으로 설정되지 않은 경우(이 상태를 `UNSET`이라 한다), 해당 로거는 자신의 부모 로거에 설정된 심각도 수준을 그대로 물려받는다.1 만약 부모 로거 역시 `UNSET` 상태라면, 다시 그 부모의 부모 로거 설정을 확인하는 과정이 최상위 로거에 도달할 때까지 연쇄적으로 일어난다.4 만약 최상위 로거까지 모두 `UNSET` 상태라면, 시스템에 설정된 기본 로거 레벨(기본값은 `INFO`)이 최종적으로 적용된다.1

이 메커니즘의 가장 큰 장점은 중앙 집중적인 제어 가능성이다. 예를 들어, `control.motion_planner.local`과 `control.motion_planner.global` 로거의 레벨이 모두 `UNSET`이라고 가정하자. 이 상태에서 `control.motion_planner` 로거의 레벨을 `DEBUG`로 변경하면, 두 자식 로거는 이 설정을 상속받아 모두 `DEBUG` 레벨로 동작하게 된다.4 반대로, `control.motion_planner.local` 로거의 레벨만 `INFO`로 명시적으로 설정하면, 이 로거는 더 이상 부모의 설정을 상속받지 않고 독립적으로 동작한다.

이처럼 레벨 상속은 특정 서브시스템 전체의 로그 상세도를 한 번에 높이거나 낮추는 것을 매우 용이하게 만든다. 개발자는 평소에는 상위 로거(예: `control`)를 `INFO` 레벨로 유지하여 간결한 로그를 확인하다가, 특정 기능(예: `motion_planner`)에 문제가 발생하면 해당 로거의 레벨만 `DEBUG`로 동적으로 변경하여 문제의 원인을 심층적으로 분석할 수 있다. `rclpy.logging.get_logger_effective_level()`과 같은 API를 사용하면 특정 로거에 상속 규칙을 통해 최종적으로 적용된 유효 심각도 수준(effective level)을 확인할 수 있어, 현재 로깅 상태를 명확히 파악하는 데 도움이 된다.8

------

## 2.  ROS2 로깅 시스템 활용 (Practical Application of the ROS2 Logging System)

ROS2 로깅 시스템의 근본 원리를 이해했다면, 다음 단계는 이를 실제 코드에 효과적으로 적용하는 것이다. 이 장에서는 C++(`rclcpp`)과 Python(`rclpy`) 클라이언트 라이브러리에서 제공하는 로깅 API의 구체적인 사용법을 분석하고, 실시간 시스템의 특성을 고려한 고급 필터링 기법, 그리고 ROS 프레임워크 외부의 코드와 로깅 시스템을 통합하는 전략에 대해 논한다.

### 2.1  코드 레벨 로깅 API 분석: `rclcpp`와 `rclpy`

ROS2는 C++과 Python 개발자 모두에게 일관된 경험을 제공하기 위해 노력하지만, 각 언어의 특성에 맞는 로깅 API를 제공한다.

C++ (rclcpp):

C++ 환경에서는 주로 전처리기 매크로(preprocessor macro)를 통해 로깅 기능을 사용한다. 이는 컴파일 타임에 로그 레벨을 확인하여, 비활성화된 로그 호출(예: DEBUG 레벨이 비활성화된 상태에서의 RCLCPP_DEBUG 호출)과 관련된 코드(메시지 포맷팅 등)가 아예 컴파일 결과물에서 제외되도록 하여 런타임 성능 저하를 최소화한다.

- **printf-style 매크로**: `RCLCPP_<SEVERITY>(logger, "format string %d", value)` 형태로, C언어의 `printf` 함수와 유사한 포맷 문자열을 사용한다.11
- **Stream-style 매크로**: `RCLCPP_<SEVERITY>_STREAM(logger, "message " << value)` 형태로, C++의 `std::ostream`과 유사한 스트림 연산자를 사용하여 메시지를 구성한다.11

두 스타일 모두 메시지 끝에 개행 문자(`\n`)를 추가할 필요가 없다. 로깅 인프라가 자동으로 처리해주기 때문이다.11

Python (rclpy):

Python 환경에서는 노드의 로거 객체가 제공하는 메서드를 통해 로깅을 수행한다.

- **메서드 호출**: `node.get_logger().<severity>('message %d' % value)`와 같이 각 심각도 수준에 해당하는 메서드를 호출한다.11 f-string (

  `f'message {value}'`) 이나 `.format()` 메서드도 사용할 수 있다.

- **키워드 인자**: C++의 다양한 매크로 변형(예: `_ONCE`, `_THROTTLE`)에 해당하는 기능들이 `once`, `skip_first`, `throttle_duration_sec`과 같은 키워드 인자(keyword arguments) 형태로 제공되어 유연성을 높인다.8

다음 표는 `rclcpp`와 `rclpy`의 주요 로깅 API를 기능별로 비교하여 보여준다.

| 기능               | 설명                                                     | `rclcpp` 예제                                                | `rclpy` 예제                                                 |
| ------------------ | -------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **기본 로깅**      | 메시지를 매번 출력한다.                                  | `RCLCPP_INFO(node->get_logger(), "Value: %d", 42);`          | `node.get_logger().info(f'Value: {42}')`                     |
| **1회 로깅**       | 해당 코드 라인이 처음 실행될 때만 메시지를 출력한다.     | `RCLCPP_INFO_ONCE(node->get_logger(), "Initialization complete.");` | `node.get_logger().info("Initialization complete.", once=True)` |
| **최초 제외 로깅** | 두 번째 실행부터 메시지를 출력한다.                      | `RCLCPP_WARN_SKIPFIRST(node->get_logger(), "Connection lost.");` | `node.get_logger().warning("Connection lost.", skip_first=True)` |
| **주기적 로깅**    | 지정된 시간(초) 간격으로 최대 한 번만 메시지를 출력한다. | `RCLCPP_DEBUG_THROTTLE(node->get_logger(), *node->get_clock(), 1000, "Processing data...");` | `node.get_logger().debug("Processing data...", throttle_duration_sec=1.0)` |

이러한 API 비교는 다중 언어 환경에서 작업하는 개발자가 두 클라이언트 라이브러리 간의 미묘한 차이점(매크로 vs. 키워드 인자)을 명확히 인지하고, 프로젝트 전반에 걸쳐 일관된 로깅 패턴을 적용하는 데 도움을 준다.

### 2.2  조건부 및 필터링 로깅 기법

ROS2 로깅 API는 단순한 메시지 출력을 넘어, 특정 조건 하에서만 로그를 남기는 다양한 필터링 메커니즘을 내장하고 있다. 이는 불필요한 로그 출력을 줄여 가독성을 높이고, 실시간 시스템의 성능 저하를 방지하는 데 매우 중요하다.2

- **1회성 로깅 (`_ONCE` / `once=True`)**: 노드 초기화, 파라미터 로딩, 첫 번째 데이터 수신과 같이 시스템 생명주기 동안 단 한 번만 발생하는 이벤트나 상태를 기록하는 데 이상적이다. 이 기능을 사용하면 루프나 콜백 함수 내부에 있더라도 해당 로그는 단 한 번만 출력되어 로그 창이 불필요한 정보로 가득 차는 것을 방지한다.11
- **최초 제외 로깅 (`_SKIPFIRST` / `skip_first=True`)**: 반복적인 작업에서 첫 번째 시도는 정상적일 수 있으나(예: 초기 연결 시도), 두 번째 시도부터 발생하는 이벤트가 문제 상황을 의미할 때 유용하다. 예를 들어, 재연결 시도 로직에서 첫 번째 시도는 무시하고, 두 번째 시도부터 경고 로그를 남겨 지속적인 문제를 알릴 수 있다.2
- **주기적 로깅 (`_THROTTLE` / `throttle_duration_sec`)**: 이 기능은 실시간 로봇 시스템의 특성을 가장 잘 반영하는 기능 중 하나이다. 로봇의 제어 루프나 센서 데이터 처리 콜백은 수십에서 수백 헤르츠(Hz)의 높은 주기로 실행된다. 이러한 고주파 루프 내에서 일반적인 로깅 함수를 호출하면, 매초 수백 개의 로그가 생성되는 '로그 폭증(log flooding)' 현상이 발생한다. 이는 콘솔 가독성을 해칠 뿐만 아니라, CPU 자원을 소모하고 시스템 타이밍에 미세한 영향을 주어 실시간 성능을 저해할 수 있다. `THROTTLE` 기능은 지정된 시간 간격(예: 1초) 내에 최대 한 번만 로그를 출력하도록 제한함으로써, 최소한의 오버헤드로 시스템이 정상적으로 동작하고 있음을 나타내는 '심장 박동(heartbeat)' 로그를 남길 수 있게 해준다.2 이는 단순한 편의 기능을 넘어, 실시간 시스템의 성능과 안정성을 보장하기 위한 필수적인 디버깅 전략이다.

### 2.3  비-ROS 컴포넌트에서의 로깅 통합 전략

대부분의 로봇 시스템은 순수 ROS 코드로만 구성되지 않는다. 외부 라이브러리, 하드웨어 드라이버, 또는 ROS에 독립적인 비즈니스 로직을 담은 클래스 등 다양한 비-ROS 컴포넌트와 통합된다. ROS2 로거는 `rclpy.node.Node` 또는 `rclcpp::Node` 객체와 강하게 결합되어 있어(`node.get_logger()`), 이러한 비-ROS 컴포넌트에서 직접 ROS 로깅을 사용하는 것은 도전적인 과제이다.9 다음은 이 문제를 해결하기 위한 몇 가지 효과적인 전략이다.

- **전략 1: 의존성 주입 (Dependency Injection)**: 가장 권장되는 객체 지향 설계 패턴이다. 로깅이 필요한 비-ROS 클래스의 생성자나 초기화 메서드를 통해 `rclcpp::Logger` 객체나 Python 로거 객체를 외부(주로 해당 클래스의 인스턴스를 생성하는 ROS 노드)에서 전달받는 방식이다.

  ```C++
  // MyLibrary.hpp
  class MyLibrary {
  public:
      explicit MyLibrary(const rclcpp::Logger& logger);
      void do_work();
  private:
      rclcpp::Logger logger_;
  };
  
  // MyRosNode.cpp
  MyRosNode::MyRosNode() : Node("my_ros_node") {
      my_lib_instance_ = std::make_unique<MyLibrary>(this->get_logger());
      my_lib_instance_->do_work();
  }
  ```

  이 방식은 비-ROS 컴포넌트가 ROS에 대한 직접적인 의존성을 갖지 않게 하여 코드의 모듈성과 테스트 용이성을 극대화한다.

- **전략 2: 전역 로거 획득 (Global Logger Acquisition)**: 의존성 주입이 구조적으로 어려운 경우, `rclpy.logging.get_logger('my_library_logger')` 와 같이 명시적인 이름으로 로거를 직접 생성하여 사용할 수 있다.10 이 로거는 특정 노드에 귀속되지는 않지만, ROS2 로깅 시스템의 일부로 동작하여 심각도 레벨 설정, 출력 핸들러 등 ROS2 로깅의 모든 기능을 활용할 수 있다. 이는 라이브러리 코드 내에서 ROS 로깅을 사용하기 위한 간편한 방법이다.

- **전략 3: Python 표준 로깅 통합 (Standard Logging Integration)**: 많은 Python 기반 외부 라이브러리는 파이썬의 표준 `logging` 모듈을 사용한다. 이 경우, ROS2의 `rclpy` 로깅 시스템과 표준 `logging` 모듈을 연결하는 커스텀 핸들러를 구현할 수 있다.9 즉, 표준 

  `logging` 모듈로 보내진 로그 메시지를 가로채서 `rclpy` 로거로 다시 보내는 것이다. 이를 통해 외부 라이브러리의 코드를 전혀 수정하지 않고도 해당 라이브러리에서 발생하는 로그를 ROS2 생태계(예: `rqt_console`, `/rosout`) 내에서 통합적으로 관리할 수 있다.

------

## 3.  로깅 동작의 제어 및 구성 (Control and Configuration of Logging Behavior)

ROS2 로깅 시스템의 진정한 강력함은 다양한 계층에서 그 동작을 세밀하게 제어할 수 있다는 점에 있다. 개발자는 시스템의 전체적인 기본 동작부터 특정 노드의 특정 로거에 이르기까지, 여러 메커니즘을 조합하여 원하는 로깅 정책을 구현할 수 있다. 이 장에서는 로깅을 구성하는 다양한 방법론을 살펴보고, 이들 간의 상호작용과 우선순위를 명확히 규명한다.

### 3.1  구성 방법론 개요 및 우선순위

ROS2 로깅은 환경 변수, 명령줄 인수, 런치 파일, 그리고 런타임에 호출 가능한 서비스 API 등 여러 계층에서 구성할 수 있다. 여러 설정이 충돌할 경우, 더 구체적이고(specific) 더 늦게 적용되는(late-binding) 설정이 우선권을 갖는 일반적인 원칙을 따른다. 여러 공식 문서와 커뮤니티 논의를 종합하면, 로깅 설정의 적용 우선순위는 다음과 같이 명확하게 정리할 수 있다.8

1. **런타임 서비스 호출 (가장 높음)**: `set_logger_levels`와 같은 서비스를 통해 실행 중인 노드의 로거 레벨을 변경하는 것은 노드의 현재 메모리 상태를 직접 수정하는 것이므로, 다른 모든 초기 설정 값을 무시하고 즉시 적용된다.14 이는 실시간 디버깅 시 가장 강력한 제어 수단이다.
2. **명령줄 인수 / 런치 파일 (중간)**: `ros2 run` 또는 `ros2 launch` 시점에 `--ros-args --log-level <logger_name>:=<LEVEL>` 형식으로 특정 로거에 대해 명시적으로 지정된 레벨은 그보다 범위가 넓은 전역 설정보다 우선한다.16 이는 특정 노드나 컴포넌트의 동작을 정밀하게 제어하기 위한 것이다.
3. **전역 명령줄 인수 (낮음)**: `--ros-args --log-level <LEVEL>`과 같이 특정 로거를 명시하지 않고 전역적으로 지정된 레벨은, 개별적으로 레벨이 설정되지 않은 모든 로거의 기본값으로 사용된다.16
4. **환경 변수 (기본값, 가장 낮음)**: `RCUTILS_...`로 시작하는 환경 변수들은 로깅의 출력 형식, 색상, 대상 스트림(stdout/stderr) 등 프로세스가 시작될 때의 가장 기본적인 동작 환경을 정의한다.8 이는 가장 낮은 우선순위를 가지며, 다른 더 구체적인 설정에 의해 쉽게 덮어쓰일 수 있다.

이러한 우선순위 계층을 이해하는 것은 복잡한 시스템에서 로깅 동작이 예상과 다를 때 문제의 원인을 체계적으로 추적하는 데 매우 중요하다. 예를 들어, 런치 파일에서 로그 레벨을 `DEBUG`로 설정했음에도 불구하고 `DEBUG` 메시지가 보이지 않는다면, 런타임 서비스에 의해 레벨이 `INFO`로 다시 변경되었을 가능성을 먼저 의심해볼 수 있다.

### 3.2  환경 변수를 활용한 프로세스-단위 설정

환경 변수는 특정 터미널 세션이나 전체 시스템에서 실행되는 모든 ROS2 프로세스에 대해 일관된 로깅 환경을 설정하는 데 사용된다. 이는 주로 출력 형식이나 파일 저장 위치와 같은 전역적인 동작을 제어한다.

다음 표는 로깅과 관련된 주요 환경 변수와 그 기능을 정리한 것이다.

| 환경 변수                         | 기능 설명                                                   | 설정 값 예시                                                 | 관련 정보                                                    |
| --------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `RCUTILS_CONSOLE_OUTPUT_FORMAT`   | 콘솔에 출력되는 로그 메시지의 형식을 지정한다.              | `export RCUTILS_CONSOLE_OUTPUT_FORMAT="[{severity} {time}][{name}]: {message} ({function_name}() at {file_name}:{line_number})"` | 사용 가능한 토큰: `{severity}`, `{name}`, `{message}`, `{time}`, `{function_name}`, `{file_name}`, `{line_number}` 등 8 |
| `RCUTILS_COLORIZED_OUTPUT`        | 콘솔 출력 시 색상 사용 여부를 제어한다.                     | `export RCUTILS_COLORIZED_OUTPUT=0`                          | `0`: 비활성화, `1`: 강제 활성화, 설정 안함: 터미널(TTY) 여부에 따라 자동 결정 12 |
| `RCUTILS_LOGGING_USE_STDOUT`      | 로그 출력 스트림을 지정한다.                                | `export RCUTILS_LOGGING_USE_STDOUT=1`                        | `0` 또는 설정 안함: `stderr` (Foxy 이후 기본값), `1`: `stdout` 8 |
| `RCUTILS_LOGGING_BUFFERED_STREAM` | 출력 스트림의 버퍼링 방식을 제어한다.                       | `export RCUTILS_LOGGING_BUFFERED_STREAM=1`                   | `0`: 버퍼링 없음, `1`: 라인 버퍼링. 로그 순서가 섞이는 문제를 해결하는 데 유용할 수 있다.8 |
| `ROS_LOG_DIR`                     | 로그 파일이 저장될 디렉터리를 직접 지정한다.                | `export ROS_LOG_DIR=~/my_ros_logs`                           | 이 변수가 설정되면 `ROS_HOME` 설정보다 우선한다.8            |
| `ROS_HOME`                        | ROS 관련 파일(로그 포함)이 저장될 기본 디렉터리를 지정한다. | `export ROS_HOME=~/.my_ros`                                  | `ROS_LOG_DIR`이 설정되지 않은 경우, 로그는 `$ROS_HOME/log`에 저장된다.8 |

이러한 환경 변수들은 특히 CI/CD 파이프라인이나 표준화된 개발 환경을 구축할 때 유용하다. 예를 들어, 모든 개발자가 디버깅 시 파일명과 줄 번호를 포함한 로그를 볼 수 있도록 `RCUTILS_CONSOLE_OUTPUT_FORMAT`을 공통 셸 설정 파일에 추가할 수 있다.

### 3.3  명령줄 인수를 통한 동적 제어

명령줄 인수는 노드를 실행하는 시점에 일회성으로 로깅 설정을 변경하는 가장 직접적인 방법이다. 모든 ROS2 관련 인수는 `--ros-args` 플래그 뒤에 위치하여 사용자 정의 인수와 충돌하는 것을 방지한다.16

- **전역 로그 레벨 설정**: `ros2 run my_package my_node --ros-args --log-level DEBUG` 명령은 `my_node` 프로세스 내에서 명시적으로 레벨이 설정되지 않은 모든 로거의 기본 심각도 수준을 `DEBUG`로 변경한다.12 이는 특정 노드의 동작을 빠르게 디버깅하고자 할 때 유용하다.

- **특정 로거 레벨 설정**: `ros2 run my_package my_node --ros-args --log-level my_node:=DEBUG --log-level rcl:=INFO`와 같이 `<logger_name>:=<LEVEL>` 구문을 사용하면 특정 로거의 레벨을 정밀하게 제어할 수 있다.8 이 예제에서는 

  `my_node` 로거는 `DEBUG` 레벨로, ROS 클라이언트 라이브러리 자체의 로거인 `rcl`은 `INFO` 레벨로 설정된다. 이 방식은 여러 번 반복하여 다수의 로거를 동시에 설정할 수 있으며, 시스템의 특정 부분만 상세히 관찰하고 나머지 부분은 조용하게 유지하여 로그 가독성을 높이는 데 매우 효과적이다.

- **출력 대상 제어**: `--disable-stdout-logs`, `--disable-rosout-logs`, `--disable-external-lib-logs`와 같은 인수를 사용하여 콘솔, `/rosout` 토픽, 파일 로깅을 각각 비활성화할 수 있다.8 이는 네트워크 대역폭이 제한적이거나 디스크 I/O를 최소화해야 하는 임베디드 환경에서 유용하다.

### 3.4  런치 파일(Launch Files)을 이용한 체계적 구성

대규모 시스템에서는 수십 개의 노드를 동시에 실행해야 하므로, 명령줄으로 일일이 옵션을 지정하는 것은 비현실적이다. 런치 파일은 이러한 복잡한 시스템의 구성을 체계적으로 관리하는 표준적인 방법이다.

하지만 한 가지 중요한 차이점이 있다. `ros2 run` 명령어는 `--ros-args`를 직접 해석하지 않고 그대로 실행할 노드 프로세스에 전달하지만, `ros2 launch` 명령어는 자체적인 인수 해석 규칙을 가지며 `--ros-args`를 모든 노드에 자동으로 전파하지 않는다.17 이는 ROS2 런치 시스템이 시스템 전체의 구성을 명시적으로 프로그래밍하는 프레임워크로 설계되었기 때문이다. 암묵적인 전역 설정 대신, 개발자가 각 노드의 실행 환경을 명확하게 제어하도록 유도하여 구성의 예측 가능성을 높이려는 설계 철학이 반영된 것이다.17

따라서 런치 파일에서 로그 레벨을 설정하려면, `launch_ros.actions.Node`의 `arguments` 파라미터를 통해 각 노드에 개별적으로 `--ros-args`와 `--log-level`을 전달해야 한다.17 Python 런치 파일에서 

`LaunchConfiguration`을 사용하면 이 과정을 더욱 유연하게 만들 수 있다.

```Python
# my_package/launch/my_launch.py
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration
from launch_ros.actions import Node

def generate_launch_description():
    # 런치 파일 실행 시 외부에서 주입 가능한 'log_level' 인수를 선언한다.
    log_level_arg = DeclareLaunchArgument(
        'log_level',
        default_value='INFO',
        description='Logging severity level'
    )

    return LaunchDescription([
        log_level_arg,
        Node(
            package='my_package',
            executable='node_A',
            name='node_A_name',
            output='screen',
            arguments=[
                '--ros-args',
                '--log-level',
                ['node_A_name:=', LaunchConfiguration('log_level')]
        ),
        Node(
            package='another_package',
            executable='node_B',
            name='node_B_name',
            output='screen',
            arguments=
        )
    ])
```

위 예제는 `ros2 launch my_package my_launch.py log_level:=DEBUG`와 같이 실행될 수 있다. 이 경우 `node_A_name` 로거는 `DEBUG` 레벨로, `node_B_name` 로거는 `WARN` 레벨로 설정되어, 시스템의 각기 다른 부분에 대해 차등적인 로깅 정책을 런치 파일 내에서 체계적으로 관리할 수 있다.

### 3.5  런타임 API: 서비스를 통한 실시간 제어

시스템을 재시작하지 않고 실행 중에 로깅 동작을 변경하는 기능은 실시간 디버깅의 효율성을 극대화한다. 최신 ROS2 배포판(Iron 이상)은 이를 위한 표준 서비스 인터페이스를 제공한다.

먼저, 노드를 생성할 때 `rclcpp::NodeOptions().enable_logger_service(true)` (C++) 또는 동등한 `rclpy` 옵션을 통해 로거 설정 서비스를 활성화해야 한다.14 이 옵션은 기본적으로 비활성화되어 있는데, 이는 모든 노드가 불필요하게 추가적인 서비스를 생성하는 것을 방지하기 위함이다.14

서비스가 활성화되면, 해당 노드는 `/<node_name>/get_logger_levels`와 `/<node_name>/set_logger_levels`라는 두 가지 서비스를 제공하게 된다.14 이 서비스들은 `rcl_interfaces/srv/GetLoggerLevels`와 `rcl_interfaces/srv/SetLoggerLevels` 타입의 표준 인터페이스를 사용한다.14

예를 들어, `NodeWithLoggerService`라는 노드에서 이 기능이 활성화되었다고 가정하자. 다른 터미널에서 다음 명령어를 실행하여 `NodeWithLoggerService` 로거와 `rcl` 로거의 레벨을 각각 `INFO`(20)와 `DEBUG`(10)으로 동적으로 변경할 수 있다.

```Bash
ros2 service call /NodeWithLoggerService/set_logger_levels rcl_interfaces/srv/SetLoggerLevels \
'{levels:}'
```

이 기능은 운영 중인 로봇에서 특정 서브시스템의 동작이 의심스러울 때, 전체 시스템을 중단시키지 않고 해당 부분의 상세 로그만 일시적으로 활성화하여 문제를 진단하고, 진단이 끝나면 다시 원래 레벨로 복원하는 강력하고 유연한 디버깅 워크플로우를 가능하게 한다.

### 3.6  파라미터 파일(YAML)을 통한 구성의 한계

ROS2 개발자들 사이에서 흔히 발생하는 오해 중 하나는 표준 ROS 파라미터 시스템, 즉 YAML 파일을 `--params-file` 인수로 로드하는 방식으로 로거의 심각도 수준을 설정할 수 있을 것이라는 기대이다. 결론부터 말하자면, 이는 **불가능하다**.8

이러한 제약은 ROS2의 설계상 의도된 결과이며, 그 이유는 다음과 같다.

1. **초기화 순서의 문제**: 로깅 시스템(`rcutils`, `rcl`)은 노드의 핵심 기능, 특히 파라미터 서버가 완전히 초기화되기 이전에 설정되어야 한다. 왜냐하면 로깅 시스템은 파라미터 파일을 로드하고 파싱하는 과정 자체에서 발생하는 오류나 경고를 기록해야 할 수도 있기 때문이다. 만약 로그 레벨이 파라미터 파일 내에 정의된다면, 이 순서가 역전되는 논리적 모순이 발생한다.
2. **설계적 관심사의 분리**: ROS2는 로깅과 파라미터를 서로 다른 관심사로 명확히 분리한다. 로깅은 노드의 실행 환경과 관련된 기반 기능으로 간주되어, `NodeOptions`의 일부로 처리되고 명령줄 인수나 환경 변수를 통해 제어된다.8 반면, 파라미터는 노드가 실행된 후 그 애플리케이션의 구체적인 동작 로직(예: 게인 값, 속도 제한 등)을 설정하는 데 사용된다.

이러한 설계적 차이를 이해하지 못하면, 개발자는 YAML 파일에 로그 레벨을 설정하고 왜 동작하지 않는지 원인을 찾느라 불필요한 시간을 낭비할 수 있다. 로깅 레벨 구성은 파라미터 시스템이 아닌, 본 장에서 설명한 다른 메커니즘(명령줄 인수, 런치 파일의 `arguments` 등)을 통해 이루어져야 한다는 점을 명확히 인지하는 것이 중요하다.

------

## 4.  ROS2 로깅 아키텍처 심층 분석 (In-depth Analysis of the ROS2 Logging Architecture)

ROS2 로깅 시스템의 다양한 기능을 온전히 활용하기 위해서는 그 내부 아키텍처에 대한 이해가 필수적이다. 로깅은 단순한 단일 라이브러리가 아니라, 여러 계층으로 구성된 정교한 시스템이다. 이 장에서는 로깅 스택의 각 계층의 역할과 상호작용을 분석하고, 로그 메시지가 생성되어 최종 목적지까지 전달되는 경로를 추적하며, ROS1과의 근본적인 아키텍처 차이점을 비교한다.

### 4.1  로깅 스택의 계층 구조

ROS2 로깅 시스템은 하위 수준의 유틸리티 라이브러리부터 상위 수준의 사용자 친화적 API에 이르기까지 여러 레이어로 구성되어 있다.

- **`rcutils` (ROS C Utilities)**: 로깅 스택의 가장 기반이 되는 레이어이다.8 이 C 라이브러리는 플랫폼 독립적인 기본 유틸리티를 제공하며, 로깅과 관련해서는 로그 메시지의 최종 포맷팅과 콘솔(stdout/stderr) 출력을 담당한다.19

  `RCUTILS_CONSOLE_OUTPUT_FORMAT`과 같은 환경 변수를 파싱하여 출력 형식을 결정하는 로직이 이 레이어에 포함되어 있다.

- **`rcl_logging_spdlog`**: ROS2는 파일 로깅을 위해 `spdlog`라는 고성능의 외부 C++ 로깅 라이브러리를 백엔드로 사용한다.8

  `rcl_logging_spdlog` 패키지는 `rcl_logging_interface`라는 추상화된 인터페이스를 구현하여 `spdlog`의 기능을 ROS2 생태계에 통합하는 역할을 한다.19 즉, 상위 레이어로부터 포맷팅된 로그 메시지를 받아 

  `spdlog`를 통해 디스크에 효율적으로 기록한다.

- **`rcl` (ROS Client Library)**: ROS2의 핵심 C API 레이어인 `rcl`은 `rcutils`와 `rcl_logging_spdlog`를 통합하여 중앙 제어 역할을 수행한다.29 상위 레이어로부터 로깅 요청이 들어오면, 

  `rcl`은 현재 설정을 확인하여 해당 메시지를 어떤 출력 핸들러(console handler, file handler, rosout handler)로 전달할지 결정하고 분배한다.

- **`rclcpp` / `rclpy`**: 개발자가 직접 상호작용하는 최상위 레이어로, 각각 C++과 Python을 위한 클라이언트 라이브러리이다.1

  `RCLCPP_...` 매크로나 `logger.info()`와 같은 사용자 친화적인 API를 제공하며, 내부적으로는 `rcl`의 함수들을 호출하여 실제 로깅 작업을 위임한다. 특히 `rclcpp`의 경우, 여러 스레드에서 동시에 로깅 함수가 호출될 때 메시지가 깨지는 것을 방지하기 위해 전역 뮤텍스(global mutex)를 사용하여 로깅 호출을 직렬화한다.31 이는 스레드 안전성을 보장하지만, 고도로 병렬화된 애플리케이션에서는 성능 병목 지점이 될 수 있다는 점을 인지해야 한다.

### 4.2  로그 메시지 전달 경로: 콘솔, 파일, 그리고 `/rosout`

개발자가 코드에서 `RCLCPP_INFO(...)`와 같은 단일 로깅 함수를 호출하면, 해당 메시지는 기본적으로 동시에 세 가지 목적지로 전달되도록 설계되어 있다.19

1. **콘솔 (Console)**: `rcl`은 메시지를 `rcutils`의 콘솔 출력 핸들러(`rcutils_logging_console_output_handler`)로 전달한다. `rcutils`는 `RCUTILS_LOGGING_USE_STDOUT` 환경 변수 설정에 따라 `stderr`(기본값) 또는 `stdout`으로 메시지를 출력한다.29

2. **파일 (File)**: `rcl`은 메시지를 `rcl_logging_spdlog`의 파일 출력 핸들러(`rcl_logging_ext_lib_output_handler`)로 전달한다. `spdlog` 백엔드는 이 메시지를 `~/.ros/log` 또는 `ROS_LOG_DIR` 환경 변수로 지정된 디렉터리 내의 로그 파일에 비동기적으로 기록한다.19

3. **`/rosout` 토픽**: `rcl`은 메시지를 `rcl_logging_rosout_output_handler`로 전달한다. 이 핸들러는 RMW(ROS Middleware Interface) 레이어를 통해 `rcl_interfaces/msg/Log` 타입의 메시지를 생성하고, 이를 `/rosout`이라는 특수 토픽으로 퍼블리시한다.29 이를 통해 

   `rqt_console`과 같은 외부 GUI 도구나 다른 노드에서 시스템 전체의 로그를 원격으로 구독하고 모니터링할 수 있다.

이 세 가지 출력 대상은 각각 독립적으로 제어될 수 있다. 노드 생성 시 `--disable-stdout-logs`, `--disable-external-lib-logs`, `--disable-rosout-logs`와 같은 명령줄 인수를 사용하면, 특정 노드에 대해 불필요한 출력 경로를 비활성화하여 성능을 최적화하거나 보안 요구사항을 만족시킬 수 있다.8

### 4.3  ROS1 (`rosconsole`)과의 비교 분석

ROS1에서 ROS2로 마이그레이션하는 개발자에게 로깅 시스템의 변화는 가장 혼란스러운 부분 중 하나일 수 있다. 두 시스템은 표면적으로 유사해 보이지만, 근본적인 아키텍처와 동작 방식에서 큰 차이를 보인다.

| 비교 항목          | ROS1 (`rosconsole`)                                          | ROS2 (`rcl_logging`)                                         |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **아키텍처**       | 중앙 집중형 (ROS Master 의존)                                | 완전 분산형 (DDS 기반, Master 없음) 33                       |
| **중앙 관리**      | `roscore`에 포함된 `rosout` 노드가 모든 로그를 집계하고 파일로 기록 | 각 노드가 독립적으로 파일 로깅을 처리. `/rosout`은 원격 모니터링을 위한 선택적 채널 29 |
| **구성 방식**      | 주로 `rosconsole.config` 파일 또는 `ROS_ROOT/config/rosconsole.config`를 통해 전역적으로 설정 35 | 환경 변수, 명령줄 인수, 런치 파일, 런타임 서비스 등 다계층적이고 분산된 구성 방식 37 |
| **`/rosout` 역할** | 모든 노드의 로그가 전송되는 중앙 집계 토픽. `rosout` 노드가 이를 구독하여 처리. | 각 노드가 직접 퍼블리셔가 되는 토픽. 중앙 집계 노드 없음. 원격 모니터링용. 29 |
| **클라이언트 API** | `ros/console.h` 헤더. `ROS_INFO`, `ROS_INFO_NAMED` 등 다양한 매크로 제공 38 | `rclcpp/logging.hpp` 헤더. `RCLCPP_INFO` 등 유사한 매크로 제공. `rclpy`는 객체 메서드 방식 39 |
| **성능 고려사항**  | `rosout` 노드가 병목 지점이 될 수 있음.                      | 각 노드가 직접 파일 I/O 수행. `rclcpp`의 전역 뮤텍스가 멀티스레드 환경에서 병목이 될 수 있음.31 |

가장 근본적인 차이는 **중앙 집중성**의 제거에 있다. ROS1에서는 `roscore`의 일부인 `rosout` 노드가 시스템 전체의 로그를 `/rosout` 토픽을 통해 수신하고, 이를 집계하여 파일에 기록하는 중앙 허브 역할을 했다.40 이 구조는 간단했지만, `rosout` 노드 자체가 단일 실패 지점(Single Point of Failure)이 될 수 있었다.

반면, ROS2는 ROS Master를 제거한 분산 아키텍처를 채택함에 따라 로깅 시스템도 완전히 분산되었다.41 각 ROS2 노드는 자체적으로 파일 로깅(`spdlog` 백엔드 사용)과 콘솔 로깅을 처리하는 독립적인 주체이다. `/rosout` 토픽은 여전히 존재하지만, 그 역할은 원격 모니터링을 위한 데이터 스트림으로 축소되었다. 각 노드는 필요에 따라 `/rosout` 토픽의 퍼블리셔가 되며, 중앙에서 이를 집계하는 단일 노드는 더 이상 존재하지 않는다.

구성 방식 또한 ROS1의 `rosconsole.config`라는 단일 설정 파일 중심에서, ROS2에서는 환경 변수, 노드별 명령줄 인수, 런치 파일 내 인수 지정 등 훨씬 더 유연하고 세분화된 방식으로 진화했다. 이러한 변화는 각 노드의 로깅 동작을 독립적으로 제어할 수 있게 하여, 대규모 모듈형 시스템의 관리를 용이하게 만든다.

------

## 5.  고급 로깅 및 디버깅 전략 (Advanced Logging and Debugging Strategies)

지금까지 논의된 ROS2 로깅 시스템의 기본 원리와 아키텍처에 대한 지식은 실제 로봇 애플리케이션의 복잡한 문제를 해결하기 위한 실용적인 전략의 기반이 된다. 이 장에서는 대규모 시스템에서 로깅을 효과적으로 설계하고 활용하는 모범 사례, `rqt_console`과 `ros2 bag`을 이용한 심층 분석 기법, 그리고 로깅을 통한 효과적인 디버깅 패턴과 피해야 할 안티-패턴을 제시한다.

### 5.1  대규모 시스템을 위한 로깅 설계 패턴 및 모범 사례

복잡한 시스템일수록 체계적인 로깅 정책의 수립이 중요하며, 이는 프로젝트 초기부터 고려되어야 한다.

- **일관된 로거 이름 규칙 수립**: 시스템의 논리적 구조를 반영하는 계층적 로거 이름 규칙을 프로젝트 전체에 적용하는 것이 매우 중요하다. 예를 들어, `perception.lidar.filter`, `control.motion_planner.local`과 같은 명명 규칙은 로그 메시지만으로도 문제의 출처를 명확히 알려준다.42 이는 특정 서브시스템 전체의 로그 레벨을 한 번에 제어하는 것을 용이하게 하여 디버깅 효율성을 극대화한다.

- **구조화된 로그 메시지 (Structured Logging)**: `"Sensor data processed: sensor_id=lidar_front, points=1024, processing_time_ms=15"`와 같이 `key=value` 쌍이나 JSON 형식을 사용하여 기계가 파싱하기 쉬운 형태로 로그를 작성하는 것이 좋다. 이는 `grep`, `awk` 같은 전통적인 텍스트 처리 도구뿐만 아니라, Elasticsearch, Splunk와 같은 전문 로그 분석 플랫폼과의 연동을 용이하게 하여 데이터 기반의 시스템 성능 분석 및 이상 탐지를 가능하게 한다.44

- **로그 레벨의 동적 관리 워크플로우 구축**: 운영 중인 시스템에서 문제가 발생했을 때, 시스템을 재시작하지 않고도 원격으로 `set_logger_levels` 서비스를 호출하여 특정 서브시스템의 로그 레벨을 `DEBUG`로 일시적으로 전환하는 절차를 마련해야 한다. 문제의 원인이 파악되면 다시 `INFO` 레벨로 복원하여 불필요한 로그 생성을 막는다.45 이러한 동적 제어 능력은 다운타임을 최소화해야 하는 상용 시스템에서 필수적이다.

- **런치 파일 모듈화와 로깅 설정**: 대규모 프로젝트에서는 기능 단위(예: 인식, 내비게이션, 조작)로 런치 파일을 분리하고, 최상위 런치 파일에서 이들을 `IncludeLaunchDescription`으로 포함하는 구조가 권장된다.42 로그 레벨과 같이 시스템 전반에 걸쳐 변경될 수 있는 설정은 최상위 런치 파일에서 

  `DeclareLaunchArgument`와 `LaunchConfiguration`을 사용하여 파라미터화한다. 이를 통해 하위 런치 파일을 수정하지 않고도 전체 시스템의 로깅 정책을 일관되게 변경할 수 있다.17

### 5.2  `rqt_console`과 `ros2 bag`을 활용한 로그 분석

ROS2는 로그를 분석하기 위한 강력한 도구를 기본적으로 제공한다.

- **`rqt_console`을 이용한 실시간 분석**: `rqt_console`은 단순한 로그 뷰어를 넘어선 강력한 실시간 분석 도구이다.46

  - **고급 필터링**: 심각도 수준별로 메시지를 제외하는 기본 필터 외에도, 'Filter' 영역에서 정규 표현식(regex)을 사용하여 특정 로거 이름 패턴(예: `.*planner.*`)을 가진 로그만 분리하여 볼 수 있다. 'Highlight' 영역에서는 특정 문자열(예: "goal reached")을 포함하는 메시지를 강조하여 중요한 이벤트를 시각적으로 추적할 수 있다.46
  - **사후 분석**: `rqt_console`은 현재 표시되는 로그를 파일로 저장하고, 나중에 다시 로드하는 기능을 제공한다.46 이를 통해 특정 이벤트가 발생한 시점의 로그를 캡처하여 동료와 공유하거나 오프라인에서 심층 분석을 수행할 수 있다.

- **`ros2 bag`을 이용한 사후 분석 (Post-mortem Analysis)**: `ros2 bag`은 ROS2 시스템을 위한 '블랙박스' 또는 '비행 데이터 기록기' 역할을 수행할 수 있다. 분산 시스템에서 발생하는 예측 불가능한 오류는 종종 여러 컴포넌트에 걸친 비동기적 상호작용의 결과로 나타나며 재현이 매우 어렵다. 개별 노드의 로그 파일만으로는 전체 시스템의 시간적 맥락을 파악하기 어렵지만, `/rosout` 토픽은 모든 노드의 로그를 타임스탬프와 함께 단일 스트림으로 집계한다.29

  ros2 bag record -a 또는 구체적으로 ros2 bag record /rosout 명령을 통해 시스템 운영 중 /rosout 토픽을 포함한 모든 데이터를 기록하면 2, 시스템 장애 발생 후 해당 bag 파일을 분석하여 모든 노드의 마지막 메시지들을 시간 순서대로 재구성할 수 있다. 이를 통해 "A 노드에서 

  `ERROR`가 발생하기 직전, B 노드는 어떤 `INFO` 메시지를 보냈는가?"와 같은 복잡한 인과 관계를 추적할 수 있으며, 이는 디버깅의 난이도를 획기적으로 낮추는 매우 강력한 전략이다. 기록된 bag 파일은 `ros2 bag play`로 재생하면서 `rqt_console`로 다시 분석하거나, `rosbag2_py` API를 사용하여 특정 조건(예: `FATAL` 레벨 메시지)을 만족하는 로그만 추출하는 자동화된 분석 스크립트를 작성하는 데 활용될 수 있다.50

### 5.3  로깅 기반 디버깅 기법 및 안티-패턴

효과적인 로깅은 체계적인 디버깅의 초석이다. 다음은 권장되는 디버깅 기법과 피해야 할 안티-패턴이다.

- **효과적인 디버깅 기법**:

  - **상태 전이 로깅**: 로봇의 동작을 제어하는 상태 머신(State Machine)이나 비헤이비어 트리(Behavior Tree)의 모든 상태 전이를 `INFO` 또는 `DEBUG` 레벨로 명확하게 기록한다. "Entering state: NAVIGATING", "Transitioning to RECOVERY"와 같은 로그는 시스템의 논리적 흐름을 추적하는 데 결정적인 단서를 제공한다.
  - **예외 처리와 로깅의 결합**: 모든 `try-catch` 블록에서 예외가 발생했을 때, 단순히 예외를 무시(swallow)하지 말고 반드시 스택 트레이스(stack trace)와 함께 `ERROR` 레벨로 로그를 남겨야 한다.53 이는 예외가 발생한 정확한 코드 위치와 호출 스택을 알려주어 문제 해결 시간을 크게 단축시킨다.
  - **입력과 출력 로깅**: 중요한 콜백 함수나 핵심 알고리즘 메서드의 시작 지점에서는 입력 파라미터 값을, 종료 지점에서는 반환 값을 `DEBUG` 레벨로 로깅한다. 이를 통해 특정 모듈이 예상대로 데이터를 수신하고 올바른 결과를 생성하는지 검증할 수 있다.

- **피해야 할 안티-패턴 (Anti-patterns)**:

  - **민감 정보 로깅**: 비밀번호, API 키, 사용자 개인정보 등 보안에 민감한 데이터를 로그에 평문으로 기록하는 것은 심각한 보안 취약점을 유발한다.

  - **과도한 `DEBUG` 로깅과 로그 폭증**: `DEBUG` 레벨은 기본적으로 비활성화되어 있지만, 활성화했을 때 시스템 성능에 심각한 영향을 줄 정도로 많은 로그를 남기는 것은 피해야 한다.53 특히 고주파 루프 내에서는 반드시 

    `THROTTLE` 기능을 사용하여 로그 발생 빈도를 제어해야 한다.

  - **모호하고 무의미한 메시지**: "Here", "Done", "step 1"과 같이 맥락 없이는 이해할 수 없는 메시지는 디버깅에 아무런 도움이 되지 않는다.44 로그 메시지는 그 자체로 완전한 문장이 되어, "Successfully planned path with 125 points."와 같이 어떤 일이 일어났는지 명확히 설명해야 한다.

  - **로깅을 제어 흐름에 사용**: 로그 레벨 설정에 따라 프로그램의 핵심 로직이 분기되거나 동작이 달라지도록 코드를 작성하는 것은 매우 위험한 설계이다. 로깅은 시스템을 관찰하기 위한 부가적인 기능이어야 하며, 프로그램의 핵심 제어 흐름에 영향을 주어서는 안 된다.

------

## 6. 결론

### ROS2 로깅 시스템의 핵심 철학 요약

본 보고서를 통해 분석한 바와 같이, ROS2 로깅 시스템은 단순한 메시지 출력 도구가 아니다. 이는 **모듈성(Modularity)**, **제어 가능성(Controllability)**, 그리고 **관측 가능성(Observability)**이라는 핵심 철학을 바탕으로 설계된 정교한 분산 시스템 진단 프레임워크이다. 로거의 계층 구조는 소프트웨어의 모듈성을 로깅 시스템에 투영하며, 다계층적인 구성 방법론은 개발자에게 시스템의 동작을 세밀하게 제어할 수 있는 권한을 부여한다. 콘솔, 파일, 그리고 `/rosout`으로 이어지는 다중 출력 경로는 다양한 시나리오에서 시스템의 상태를 효과적으로 관찰할 수 있도록 지원한다. 이러한 요소들이 결합하여 ROS2 로깅은 복잡한 로봇 시스템의 개발, 디버깅, 운영 전반에 걸쳐 필수적인 역할을 수행한다.

### 6.1 효과적인 로깅 시스템 구축을 위한 최종 제언

안정적이고 신뢰성 높은 로봇 시스템을 구축하기 위해서는 로깅을 부수적인 작업이 아닌, 설계의 핵심 요소로 간주해야 한다. 이를 위한 최종적인 제언은 다음과 같다.

1. **프로젝트 초기에 로깅 가이드라인을 수립하라**: 모든 팀원이 따를 수 있는 일관된 로거 이름 규칙, 심각도 수준별 사용 기준, 메시지 형식 등을 정의하여 프로젝트 전반의 로그 품질을 높여야 한다.
2. **다양한 구성 방법을 숙지하고 목적에 맞게 사용하라**: 개발 중에는 명령줄 인수를 통한 신속한 레벨 변경을, 시스템 통합 및 배포 시에는 런치 파일을 통한 체계적인 구성을, 그리고 운영 중에는 런타임 서비스를 통한 동적 제어를 적극적으로 활용해야 한다.
3. **분석 도구를 적극적으로 활용하라**: `rqt_console`의 실시간 필터링 기능과 `ros2 bag`을 이용한 `/rosout` 사후 분석은 복잡한 문제를 해결하는 데 매우 강력한 무기이다. 이러한 도구의 사용법을 숙달하여 디버깅 워크플로우에 통합해야 한다.

결론적으로, ROS2 로깅 시스템이 제공하는 풍부한 기능과 유연성을 깊이 이해하고 전략적으로 활용함으로써, 개발자는 더욱 견고하고 예측 가능하며 유지보수가 용이한 로봇 시스템을 구축할 수 있을 것이다.

#### **참고 자료**

1. About logging and logger configuration - ROS 2 Documentation, accessed July 22, 2025, https://docs.ros.org/en/crystal/Concepts/About-Logging.html
2. ROS2 Part 11 - Rosbag2, Ros2doctor and Logging Severity Levels - RoboticsUnveiled, accessed July 22, 2025, https://www.roboticsunveiled.com/ros2-rosbag2-ros2doctor-and-logging-severity-levels/
3. docs.ros.org, accessed July 22, 2025, [https://docs.ros.org/en/foxy/Concepts/About-Logging.html#:~:text=Log%20messages%20have%20a%20severity,or%20FATAL%20%2C%20in%20ascending%20order.](https://docs.ros.org/en/foxy/Concepts/About-Logging.html#:~:text=Log messages have a severity,or FATAL %2C in ascending order.)
4. About logging and logger configuration - ROS 2 Documentation, accessed July 22, 2025, https://docs.ros.org/en/eloquent/Concepts/Logging.html
5. RCLCPP_INFO VS RCLCPP_DEBUG ROS2 Logging | by makarand mandolkar - Medium, accessed July 22, 2025, https://medium.com/@mandolkarmakarand94/rclcpp-info-vs-rclcpp-debug-ros2-logging-413c3480a478
6. Console logging - Autoware Universe Documentation, accessed July 22, 2025, https://autowarefoundation.github.io/autoware-documentation/pr-366/contributing/coding-guidelines/ros-nodes/console-logging/
7. About logging and logger configuration - ROS 2 Documentation: Foxy 文档, accessed July 22, 2025, https://daobook.github.io/ros2-docs/foxy/Concepts/About-Logging.html
8. About logging and logger configuration - ROS 2 Documentation, accessed July 22, 2025, https://docs.ros.org/en/foxy/Concepts/About-Logging.html
9. Logging from non-ros components without access to node - Robotics Stack Exchange, accessed July 22, 2025, https://robotics.stackexchange.com/questions/114497/logging-from-non-ros-components-without-access-to-node
10. rclcpp::Logger Class Reference - ROS Documentation, accessed July 22, 2025, https://docs.ros2.org/ardent/api/rclcpp/classrclcpp_1_1_logger.html
11. Logging - ROS 2 Documentation: Jazzy documentation, accessed July 22, 2025, https://docs.ros.org/en/jazzy/Tutorials/Demos/Logging-and-logger-configuration.html
12. Logging - ROS 2 Documentation: Foxy documentation, accessed July 22, 2025, https://docs.ros.org/en/foxy/Tutorials/Demos/Logging-and-logger-configuration.html
13. The Python Node, explained - ROS2 Tutorial July 07, 2025 documentation, accessed July 22, 2025, https://ros2-tutorial.readthedocs.io/en/latest/python_node_explained.html
14. External logging level configuration / Issue #1355 / ros2/ros2 - GitHub, accessed July 22, 2025, https://github.com/ros2/ros2/issues/1355
15. Logging - ROS 2 Documentation: Rolling documentation, accessed July 22, 2025, https://docs.ros.org/en/rolling/Tutorials/Demos/Logging-and-logger-configuration.html
16. ROS Command Line Arguments, accessed July 22, 2025, https://design.ros2.org/articles/ros_command_line_arguments.html
17. Support setting log-level via command-line / Issue #383 / ros2/launch - GitHub, accessed July 22, 2025, https://github.com/ros2/launch/issues/383
18. ros - how to set up log level by package at startup? - Robotics Stack Exchange, accessed July 22, 2025, https://robotics.stackexchange.com/questions/97650/how-to-set-up-log-level-by-package-at-startup
19. About logging and logger configuration - ROS 2 Documentation, accessed July 22, 2025, https://docs.ros.org/en/galactic/Concepts/About-Logging.html
20. Logging and logger configuration demo - ROS 2 Documentation, accessed July 22, 2025, https://docs.ros.org/en/dashing/Tutorials/Logging-and-logger-configuration.html
21. Logging and logger configuration demo - ROS 2 Documentation, accessed July 22, 2025, https://docs.ros.org/en/eloquent/Tutorials/Logging-and-logger-configuration.html
22. ROS 2 Launch System, accessed July 22, 2025, https://design.ros2.org/articles/roslaunch.html
23. Selecting log level in ROS2 launch file - Robotics Stack Exchange, accessed July 22, 2025, https://robotics.stackexchange.com/questions/89933/selecting-log-level-in-ros2-launch-file
24. Understanding parameters - ROS 2 Documentation: Foxy ..., accessed July 22, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Parameters/Understanding-ROS2-Parameters.html
25. ROS2 YAML For Parameters - The Robotics Back-End, accessed July 22, 2025, https://roboticsbackend.com/ros2-yaml-params/
26. How to Use ROS 2 Parameters - Foxglove, accessed July 22, 2025, https://foxglove.dev/blog/how-to-use-ros2-parameters
27. How to read specific parameter from YAML in ROS2 .py Launch file? - Stack Overflow, accessed July 22, 2025, https://stackoverflow.com/questions/73579586/how-to-read-specific-parameter-from-yaml-in-ros2-py-launch-file
28. rcutils/logging.h File Reference - ROS Documentation, accessed July 22, 2025, https://docs.ros2.org/ardent/api/rcutils/logging_8h.html
29. Logging and logger configuration - ROS 2 Documentation: Iron ..., accessed July 22, 2025, https://docs.ros.org/en/iron/Concepts/Intermediate/About-Logging.html
30. Comprehensive code walkthrough of the rclcpp / rcutils / rcl logging subsystems - General, accessed July 22, 2025, https://discourse.ros.org/t/comprehensive-code-walkthrough-of-the-rclcpp-rcutils-rcl-logging-subsystems/30173
31. Documentation for the existing logging system / Issue #2026 / ros2/ros2_documentation, accessed July 22, 2025, https://github.com/ros2/ros2_documentation/issues/2026
32. INFO and ERROR logging should come out in order by default / Issue #168 / ros2/rcutils, accessed July 22, 2025, https://github.com/ros2/rcutils/issues/168
33. ROS 1 vs ROS 2 What are the Biggest Differences? - Model Prime, accessed July 22, 2025, https://www.model-prime.com/blog/ros-1-vs-ros-2-what-are-the-biggest-differences
34. What is the difference between ROS and ROS2? - Robotics Stack Exchange, accessed July 22, 2025, https://robotics.stackexchange.com/questions/86338/what-is-the-difference-between-ros-and-ros2
35. ROS and ROS2 Logging Severity Level - Open Robotics Discourse, accessed July 22, 2025, https://discourse.ros.org/t/ros-and-ros2-logging-severity-level/45217
36. How to enable debug logging for the DWALocalPlanner and the SimpleScoredSamplingPlanner - Robotics Stack Exchange, accessed July 22, 2025, https://robotics.stackexchange.com/questions/101327/how-to-enable-debug-logging-for-the-dwalocalplanner-and-the-simplescoredsampling
37. Logging and logger configuration demo - ROS 2 Documentation - daobook 0.0.1 文档, accessed July 22, 2025, https://daobook.github.io/ros2-docs/xin/Tutorials/Logging-and-logger-configuration.html
38. roscpp/Overview/Logging - ROS Wiki, accessed July 22, 2025, http://wiki.ros.org/roscpp/Overview/Logging
39. what is replacement for ros/console.hpp in ROS2 ? - ROS Answers archive, accessed July 22, 2025, https://answers.ros.org/question/410082/
40. ROS/Technical Overview, accessed July 22, 2025, [http://wiki.ros.org/ROS/Technical%20Overview](http://wiki.ros.org/ROS/Technical Overview)
41. ROS1 vs ROS2, Practical Overview For ROS Developers - The Robotics Back-End, accessed July 22, 2025, https://roboticsbackend.com/ros1-vs-ros2-practical-overview/
42. Managing large projects - ROS 2 Documentation, accessed July 22, 2025, https://docs.ros.org/en/rolling/Tutorials/Intermediate/Launch/Using-ROS2-Launch-For-Large-Projects.html
43. Better Console Logging - ROS General - Open Robotics Discourse, accessed July 22, 2025, https://discourse.ros.org/t/better-console-logging/44012
44. Logging anti-patterns - Gojko Adzic, accessed July 22, 2025, https://gojko.net/2006/12/09/logging-anti-patterns/
45. How to Set Default Logging Verbosity in ROS 2? - Robotics Stack Exchange, accessed July 22, 2025, https://robotics.stackexchange.com/questions/113529/how-to-set-default-logging-verbosity-in-ros-2
46. Using rqt_console to view logs - ROS 2 Documentation: Foxy ..., accessed July 22, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-CLI-Tools/Using-Rqt-Console/Using-Rqt-Console.html
47. ROS2 Basics - CLI Tools - rqt console - YouTube, accessed July 22, 2025, https://www.youtube.com/watch?v=Fc1dWutBPlw
48. Analyzing Your System Using rqt_console - Automatic Addison, accessed July 22, 2025, https://automaticaddison.com/analyzing-your-system-using-rqt_console/
49. Recording and playing back data - ROS 2 Documentation, accessed July 22, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-CLI-Tools/Recording-And-Playing-Back-Data/Recording-And-Playing-Back-Data.html
50. ros2/rosbag2 - GitHub, accessed July 22, 2025, https://github.com/ros2/rosbag2
51. Reading from a bag file (Python) - ROS 2 Documentation, accessed July 22, 2025, https://docs.ros.org/en/rolling/Tutorials/Advanced/Reading-From-A-Bag-File-Python.html
52. [Documentation/Example] Please add example on how to read a rosbag with python / Issue #1692 / ros2/rosbag2 - GitHub, accessed July 22, 2025, https://github.com/ros2/rosbag2/issues/1692
53. What are some patterns and anti-patterns of application logging? [closed] - Software Engineering Stack Exchange, accessed July 22, 2025, https://softwareengineering.stackexchange.com/questions/112402/what-are-some-patterns-and-anti-patterns-of-application-logging
54. [ROS2] Best practice for catching all exceptions for logging - Robotics Stack Exchange, accessed July 22, 2025, https://robotics.stackexchange.com/questions/100491/ros2-best-practice-for-catching-all-exceptions-for-logging


# ROS2 Humble 로깅 시스템에 대한 심층 분석 및 활용 전략
[ROS2 Humble](./index.md)


로봇 운영 체제(Robot Operating System, ROS)는 로봇 애플리케이션 개발을 위한 표준 프레임워크로 자리매김했으며, 분산 시스템 환경에서 디버깅과 상태 모니터링의 중요성은 아무리 강조해도 지나치지 않다. 이러한 시스템의 핵심적인 부분은 바로 로깅 시스템이다. 로깅은 단순히 프로그램의 상태를 출력하는 것을 넘어, 시스템의 동작을 추적하고, 예기치 않은 문제를 진단하며, 성능을 분석하는 데 필수적인 데이터를 제공한다. ROS 2는 이전 버전인 ROS 1의 교훈을 바탕으로 더욱 견고하고 유연하며 실시간성을 고려한 시스템으로 재설계되었으며, 로깅 시스템 역시 이러한 설계 철학을 반영하여 상당한 발전을 이루었다.1

ROS 2의 여러 배포판 중에서도 Humble Hawksbill은 장기 지원(Long-Term Support, LTS) 버전으로서 산업계와 학계에서 널리 채택되어 안정성과 성숙도를 대표한다.3 따라서 Humble의 로깅 시스템을 깊이 있게 이해하는 것은 ROS 2 기반의 복잡하고 신뢰성 높은 로봇 시스템을 개발하는 데 있어 필수적인 역량이다. 사용자는 종종 로깅을 단순히 `printf`와 같은 출력문으로 간주하는 경향이 있지만, ROS 2의 로깅 시스템은 그보다 훨씬 정교하고 다층적인 구조를 가지고 있다.

본 보고서는 ROS 2 Humble의 로깅 시스템에 대한 심층적인 고찰을 목적으로 한다. 이를 위해 로깅 시스템의 근간을 이루는 아키텍처와 핵심 구성 요소부터 시작하여, 로그 메시지가 생성되어 다양한 목적지로 전달되기까지의 전 과정을 추적할 것이다. 또한, 심각도 수준(severity level)과 계층적 로거(hierarchical logger)와 같은 핵심 개념을 탐구하고, 환경 변수, 커맨드 라인, 서비스, 그리고 프로그래밍 API를 통한 다각적인 설정 및 제어 방법을 상세히 분석한다. `rclcpp`(C++)와 `rclpy`(Python)에서 제공하는 다양한 로깅 API, 특히 조건부 로깅과 같은 고급 기능들의 활용법과 성능적 이점을 심도 있게 다룰 것이다.

나아가, 파일 로깅을 담당하는 `spdlog` 백엔드의 역할과 한계를 조명하고, 실시간 시스템에서의 로깅 성능 최적화 방안과 같은 고급 주제를 탐구한다. ROS 1의 `rosconsole`과의 비교를 통해 ROS 2 로깅 시스템의 설계 철학과 차별점을 명확히 하고, Humble 이후 배포판인 Iron, Jazzy에서의 개선 사항을 추적하여 로깅 시스템의 발전 방향을 조망한다. 마지막으로, 이러한 모든 기술적 분석을 종합하여 대규모 분산 시스템에서 로깅을 효과적으로 관리하고 활용하기 위한 체계적인 전략과 모범 사례를 제시하고자 한다. 본 보고서는 ROS 2 개발자가 로깅 시스템을 단순한 디버깅 도구를 넘어, 시스템의 상태를 투명하게 관찰하고 제어하는 강력한 수단으로 활용할 수 있도록 깊이 있는 통찰과 실질적인 가이드를 제공하는 것을 목표로 한다.


ROS 2 로깅 시스템은 단일체(monolithic)가 아닌, 명확한 역할 분담을 가진 여러 라이브러리 계층으로 구성된 정교한 스택이다. 이러한 모듈식 설계는 ROS 2의 핵심 철학인 유연성과 확장성을 반영하며, 각 계층은 특정 수준의 추상화에서 로깅 기능을 처리한다. 이 구조를 이해하는 것은 고급 디버깅 및 사용자 정의의 첫걸음이다.


ROS 2 로깅 시스템의 아키텍처는 가장 낮은 수준의 C 유틸리티부터 사용자가 직접 상호작용하는 클라이언트 라이브러리에 이르기까지 여러 계층으로 나뉜다. 각 계층은 하위 계층의 기능을 기반으로 더 높은 수준의 추상화를 제공한다.4

- **`rcutils` (ROS 2 C Utilities):** 이 라이브러리는 로깅 시스템의 가장 근간을 이루는 토대이다. 미들웨어에 독립적인 핵심 기능들을 C 언어로 제공하며, 여기에는 기본적인 로깅 매크로, 오류 처리 메커니즘, 커맨드 라인 인자 파싱 등이 포함된다.4

  `rcutils`는 기본적인 로깅 API와 출력 핸들러(output handler) 메커니즘을 정의한다. 이 핸들러는 로그 메시지를 특정 목적지로 보낼 함수를 지정하는 역할을 하며, 기본적으로는 콘솔(stderr 또는 stdout)로 출력을 보낸다.5

- **`rcl` (ROS Client Library Interface):** `rcutils` 위에 위치하는 `rcl`은 클라이언트 라이브러리 수준의 로직을 담당한다. `rcl`의 중요한 역할 중 하나는 여러 개의 로깅 출력 핸들러를 조합하여 하나의 통합된 핸들러로 만든 뒤, 이를 `rcutils`에 등록하는 것이다.7 이 계층에서 로깅은 ROS 그래프와 본격적으로 통합된다. 예를 들어, 로거를 특정 노드와 연관시키고, 노드의 이름과 네임스페이스를 로거 이름에 자동으로 반영하는 기능이 

  `rcl` 수준에서 처리된다.4

- **`rcl_logging_spdlog`:** 이것은 ROS 2 Humble에서 기본으로 제공되는 외부 로깅 구현체(external logging implementation)이다. `rcl_logging_interface`라는 추상 API를 구현하며, 주된 역할은 로그 메시지를 디스크의 파일에 기록하는 것이다.6 이 구성 요소의 존재는 ROS 2 로깅 시스템의 백엔드가 교체 가능(pluggable)하다는 점을 명확히 보여준다. 즉, 

  `spdlog` 대신 다른 로깅 라이브러리(예: `log4cxx`)를 백엔드로 사용하고자 할 경우, 해당 인터페이스만 구현하면 시스템에 통합할 수 있다.6

- **`rmw` (ROS Middleware Interface):** 이 계층은 DDS(Data Distribution Service)와 같은 미들웨어를 통해 네트워크 통신을 추상화한다. 로깅의 관점에서 `rmw`의 핵심 역할은 로그 메시지를 `/rosout`이라는 특수 토픽으로 발행하는 것이다.4 이를 통해 ROS 2 네트워크에 연결된 모든 노드는 다른 노드들의 로그를 구독하여 시스템 전반의 상태를 분산 환경에서 모니터링할 수 있게 된다.

- **클라이언트 라이브러리 (`rclcpp`, `rclpy`):** 개발자가 코드 내에서 직접 상호작용하는 가장 상위 계층이다. C++를 위한 `rclcpp`와 Python을 위한 `rclpy`는 각 언어에 맞는 편리한 인터페이스를 제공한다. 예를 들어, `rclcpp`는 `RCLCPP_INFO`와 같은 매크로를, `rclpy`는 `get_logger().info()`와 같은 객체 지향 메소드를 제공한다.8 이 라이브러리들은 내부적으로 

  `rcl`과 `rcutils`의 기능을 감싸고 있으며, 여러 스레드에서 동시에 로깅 호출이 발생하더라도 안전하도록 스레드 동기화(thread-safety)를 처리한 후 하위 계층으로 메시지를 전달하는 역할도 수행한다.9

이러한 계층적 구조는 문제 해결에 중요한 단서를 제공한다. 예를 들어, 로그가 콘솔에는 표시되지만 파일에는 기록되지 않는다면, 문제는 애플리케이션 코드나 `rcutils`보다는 `rcl_logging_spdlog` 계층이나 관련 설정에 있을 가능성이 높다. 반대로, 콘솔 출력의 포맷팅에 문제가 있다면 `RCUTILS_CONSOLE_OUTPUT_FORMAT` 환경 변수와 관련된 `rcutils` 계층을 살펴보아야 한다. 이처럼 각 계층의 역할을 명확히 이해하면 문제의 원인을 체계적으로 좁혀나갈 수 있다.


개발자가 코드 한 줄로 남긴 로그 메시지는 위에서 설명한 계층적 구조를 따라 여러 목적지로 전달되는 여정을 거친다. 이 과정을 단계별로 추적하면 시스템의 동작을 더욱 명확하게 이해할 수 있다.

1. **생성 (Generation):** 모든 것은 개발자의 코드에서 시작된다. C++에서는 `RCLCPP_INFO(node->get_logger(), "Log message");`와 같은 매크로를, Python에서는 `node.get_logger().info('Log message')`와 같은 메소드를 호출하여 로그 메시지를 생성한다.11
2. **클라이언트 라이브러리 처리 (Client Library Handling):** 이 호출은 해당 언어의 클라이언트 라이브러리(`rclcpp` 또는 `rclpy`)에 의해 처리된다. 라이브러리는 메시지, 심각도 수준, 로거 이름 등의 정보를 취합하고, 멀티스레드 환경에서의 안전성을 보장하기 위해 내부적인 잠금(lock) 메커니즘을 사용하여 호출을 감싼 뒤, 이를 `rcl` 계층으로 전달한다.9
3. **`rcl` 디스패칭 (Dispatching):** `rcl` 계층은 전달받은 로그 메시지를 사전에 등록된 여러 출력 핸들러에게 분배(dispatch)하는 역할을 한다.7 ROS 2 Humble의 기본 설정에서는 다음 세 가지 핸들러가 활성화되어 있다 6:
   - **콘솔 핸들러 (Console Handler):** 이 핸들러는 메시지를 `rcutils`의 콘솔 출력 함수로 전달한다. 이 함수는 최종적으로 메시지를 터미널에 출력하며, Humble에서는 기본적으로 `stderr`(표준 에러) 스트림을 사용한다.6
   - **파일 핸들러 (File Handler):** 이 핸들러는 메시지를 외부 로깅 라이브러리 인터페이스를 통해 `rcl_logging_spdlog` 구현체로 전달한다. `spdlog`는 이 메시지를 받아 지정된 로그 디렉토리 내의 파일에 기록한다.6
   - **`/rosout` 핸들러:** 이 핸들러는 메시지를 `rmw` 계층으로 보낸다. `rmw`는 이 정보를 `rosgraph_msgs/Log` 타입의 메시지로 직렬화(serialize)한 후, DDS 미들웨어를 통해 `/rosout` 토픽으로 네트워크에 발행한다.6

이 세 가지 출력 경로는 서로 독립적으로 동작하며, 노드 실행 시 커맨드 라인 인자를 통해 각각을 개별적으로 활성화하거나 비활성화할 수 있다.14 예를 들어, `--disable-rosout-logs` 인자를 사용하면 네트워크 대역폭을 절약하기 위해 `/rosout` 발행을 중단하면서도 콘솔과 파일 로깅은 유지할 수 있다. 이러한 독립성은 디버깅 시 중요한 고려사항이다. 만약 로그가 콘솔에는 나타나지만 `rqt_console`이나 로그 파일에서는 보이지 않는다면, 이는 애플리케이션의 로깅 호출 자체가 잘못된 것이 아니라, 특정 출력 경로(예: 파일 시스템 권한 문제, 네트워크 설정 오류)에 문제가 발생했음을 시사한다.


ROS 2 로깅 시스템의 설계는 ROS 1의 `rosconsole`과 비교할 때 근본적인 철학의 변화를 보여준다. 이는 로깅 시스템에 국한된 것이 아니라 ROS 2 전체의 설계 방향을 반영한다.

- **모듈성 및 추상화 (Modularity and Abstraction):** ROS 1의 `rosconsole`은 비교적 단일체적이고 강하게 결합된 구조였던 반면, ROS 2는 `rcutils`(기본 기능), `rcl`(ROS 통합), `rmw`(네트워크) 등 명확하게 분리된 계층을 통해 관심사를 분리했다.4 이는 추상 인터페이스를 통해 다양한 하위 구현을 지원하려는 ROS 2의 핵심 설계 목표와 일치한다.2

- **교체 가능한 백엔드 (Pluggable Backends):** ROS 2 아키텍처는 `rcl_logging_interface`라는 외부 로깅 라이브러리를 위한 명시적인 인터페이스를 포함한다.5 기본 구현은 

  `spdlog`이지만, 이 설계 덕분에 `log4cxx`나 사용자가 직접 개발한 고성능 로깅 솔루션과 같은 다른 백엔드로 교체할 수 있는 가능성이 열려 있다. 이는 ROS 1에서는 상상하기 어려웠던 중요한 발전이다.5

- **DDS 통합 (DDS Integration):** `/rosout` 토픽을 `rmw` 인터페이스를 통해 DDS 통신 시스템의 일부로 만든 것은 또 다른 핵심적인 차이점이다. 이로 인해 로깅은 ROS 2의 분산 발견(decentralized discovery) 및 통신 메커니즘의 혜택을 직접적으로 받게 되었다. ROS 1에서 모든 노드 발견과 통신 초기화를 중재하던 ROS Master라는 단일 실패 지점(single point of failure)이 제거된 것이다.2

결론적으로, ROS 2 Humble의 로깅 시스템은 단순한 출력 기능이 아니라, ROS 2의 핵심 설계 원칙인 모듈성, 유연성, 분산 시스템 지원을 충실히 반영하는 정교한 하위 시스템이다. 개발자는 이 구조를 이해함으로써 시스템의 동작을 더 깊이 파악하고, 문제 발생 시 원인을 정확히 진단하며, 나아가 특정 요구사항에 맞게 시스템을 확장하거나 최적화할 수 있는 기반을 마련하게 된다.


ROS 2 로깅 시스템의 강력함은 단순히 메시지를 출력하는 기능을 넘어, 어떤 메시지를, 언제, 어디서 보여줄지를 세밀하게 제어할 수 있는 능력에서 나온다. 이러한 제어의 중심에는 '로거(logger)'라는 개념이 있으며, 로거는 심각도 수준과 계층 구조라는 두 가지 핵심적인 속성을 통해 관리된다.


심각도 수준(Severity Level)은 로그 메시지의 중요도를 분류하는 데 사용되는 열거형(enumeration)이다. 이를 통해 개발자는 메시지의 성격에 따라 필터링하여 원하는 정보에 집중할 수 있다.

- **정의 및 순서:** ROS 2는 다섯 가지의 심각도 수준을 정의하며, 심각도가 낮은 순서부터 나열하면 다음과 같다: `DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL`.8
- **필터링 로직:** 각 로거는 최소 심각도 수준을 설정값으로 가진다. 로거는 자신이 처리하는 메시지의 심각도가 설정된 수준보다 같거나 높을 경우에만 해당 메시지를 처리하여 상위 핸들러(콘솔, 파일, `/rosout`)로 전달한다.8
- **기본 수준:** 특별히 설정하지 않은 로거의 기본 심각도 수준은 `INFO`이다. 이는 일반적인 실행 환경에서 `INFO`, `WARN`, `ERROR`, `FATAL` 수준의 메시지는 모두 출력되지만, `DEBUG` 수준의 상세한 메시지는 무시된다는 것을 의미한다.18 디버깅이 필요할 때만 특정 로거의 수준을 `DEBUG`로 낮추어 상세 정보를 확인할 수 있다.
- **각 수준의 의미:**
  - **`DEBUG`:** 개발자가 시스템의 내부 실행 흐름을 단계별로 추적하기 위한 매우 상세한 정보를 담는다. 이 메시지들은 주로 개발 및 디버깅 단계에서만 활성화된다.18
  - **`INFO`:** 시스템의 주요 이벤트나 상태 변경을 알려주는 정보성 메시지다. 시스템이 예상대로 정상 동작하고 있음을 확인하는 용도로 사용된다.18
  - **`WARN`:** 예상치 못한 상황이나 이상적인 상태는 아니지만, 즉각적인 기능 장애를 일으키지는 않는 경우에 사용된다. 예를 들어, 타임아웃이 임박했거나, 권장되지 않는 파라미터 값이 사용되었을 때 경고를 발생시킬 수 있다. 이는 더 심각한 문제의 전조일 수 있다.18
  - **`ERROR`:** 복구 가능한 오류가 발생했음을 나타낸다. 일부 기능에 영향을 미치지만 노드 전체가 중단되지는 않는 상황이다. 예를 들어, 특정 센서 데이터를 수신하지 못했지만 다른 기능은 계속 수행될 수 있는 경우에 해당한다.
  - **`FATAL`:** 복구가 불가능한 치명적인 오류가 발생했음을 의미한다. 이 수준의 로그가 발생하면 해당 노드는 일반적으로 곧 종료된다.


로거 계층 구조(Logger Hierarchy)는 복잡한 시스템의 로깅을 체계적으로 관리하기 위한 매우 강력한 메커니즘이다. 이는 점(`.`)으로 구분된 이름을 통해 로거들 간의 부모-자식 관계를 형성하는 방식으로 동작한다.6

- **개념:** 로거 이름은 계층적 네임스페이스를 나타낸다. 예를 들어, `control.motor.driver`라는 이름의 로거는 `control.motor` 로거의 자식이며, `control.motor`는 `control` 로거의 자식이다. 이러한 구조는 Python의 `logging` 모듈이나 Java의 `Log4j`와 유사하다.6

- **수준 상속 (Level Inheritance):** 계층 구조의 핵심 기능은 심각도 수준의 상속이다. 만약 특정 로거의 심각도 수준이 명시적으로 설정되지 않았다면(즉, `UNSET` 상태라면), 그 로거는 계층 구조를 거슬러 올라가면서 가장 가까운 부모 중 명시적으로 수준이 설정된 부모의 수준을 상속받는다. 만약 모든 상위 부모의 수준도 설정되지 않았다면, 시스템의 기본 로거 수준(기본값 `INFO`)을 따르게 된다.6

- **변경 사항 전파 (Propagation of Changes):** 부모 로거의 수준을 변경하면, 그 변경 사항은 명시적으로 자신의 수준을 설정하지 않은 모든 자손 로거들에게 자동으로 전파된다.6 예를 들어, 

  `control` 로거의 수준을 `DEBUG`로 설정하면, `control.motor`나 `control.motor.driver` 로거의 수준을 별도로 설정하지 않은 한, 이들 모두 `DEBUG` 수준으로 동작하게 된다. 이 기능 덕분에 단 하나의 명령으로 전체 하위 시스템의 로깅 상세도를 조절하는 것이 가능해진다.

이러한 계층 구조는 대규모 시스템에서 로깅을 관리하는 데 있어 전략적인 접근을 가능하게 한다. 예를 들어, 개발자는 각 노드 내에서 기능 단위로 자식 로거(예: `get_logger().get_child("planning")`, `get_logger().get_child("perception")`)를 생성하여 코드의 논리적 구조를 로거 이름에 반영할 수 있다. 이렇게 하면 평상시에는 노드 전체를 `INFO` 수준으로 운영하다가, 특정 기능(예: `planning`)에 문제가 발생했을 때 `my_node_name.planning` 로거의 수준만 `DEBUG`로 동적으로 변경하여 다른 부분의 로그는 그대로 둔 채 필요한 부분의 상세 정보만 집중적으로 확인할 수 있다. 이는 불필요한 로그의 홍수 속에서 중요한 정보를 놓치는 것을 방지하는 매우 효율적인 디버깅 전략이다.


ROS 2에서는 로거를 생성하고 사용하는 방식에 따라 크게 두 가지 종류로 나눌 수 있다. 이 둘의 차이는 특히 분산 시스템 환경에서 중요한 영향을 미친다.

- **노드 연관 로거 (Node-Associated Loggers):** 모든 ROS 2 노드는 생성 시 자동으로 자신과 연관된 로거를 갖게 된다. 이 로거의 이름은 노드의 네임스페이스와 이름을 조합하여 만들어진다 (예: `/my_namespace/my_node`).6 만약 

  `ros2 launch`나 커맨드 라인을 통해 노드의 이름이 재매핑(remapping)되면, 로거의 이름도 이를 따라 변경된다. 이는 ROS 그래프 상의 노드와 로깅 시스템 간의 일관성을 유지해주는 중요한 특징이다.6

- **이름 지정 로거 (Named Loggers / Non-Node Loggers):** 노드 객체에 직접 접근할 수 없는 라이브러리나 일반 함수 등에서도 로깅을 할 수 있도록, 특정 이름을 지정하여 로거를 생성하는 것도 가능하다. C++에서는 `rclcpp::get_logger("my_custom_name")`을, Python에서는 `rclpy.logging.get_logger('my_custom_name')`을 사용하여 생성할 수 있다.6

이 두 로거 타입의 가장 중요하고 종종 간과되는 차이점은 `/rosout` 토픽으로의 발행 여부이다. 오직 노드와 연관된 로거(그리고 그 자식 로거들)만이 자신의 로그 메시지를 `/rosout` 토픽으로 발행한다.20 이름 지정 로거를 통해 생성된 로그는 콘솔이나 파일에는 정상적으로 기록될 수 있지만, 

`/rosout` 토픽으로는 발행되지 않는다. 이는 해당 로거가 특정 노드에 속해 있지 않아 메시지를 발행할 `Publisher`를 가지고 있지 않기 때문이다.20

이러한 동작 방식은 분산 시스템 디버깅 시 혼란을 야기할 수 있다. 개발자가 라이브러리 코드에서 `rclcpp::get_logger("my_library_logger")`를 사용해 로그를 남겼을 때, 로컬 콘솔에서는 로그가 잘 보이지만 `rqt_console`과 같이 `/rosout`을 구독하여 시스템 전체의 로그를 모니터링하는 도구에서는 해당 로그가 보이지 않는 현상이 발생할 수 있다. 따라서, 시스템의 다른 부분에서도 관찰되어야 할 중요한 로그는 반드시 노드 연관 로거를 통해 생성되어야 한다는 점을 명심해야 한다. 이는 로그가 시스템의 관찰 가능한, 네트워크화된 상태의 일부가 되기 위해서는 의미론적으로 '노드'라는 엔티티에 귀속되어야 함을 시사한다.


ROS 2 Humble은 개발자와 시스템 운영자에게 로깅 시스템의 동작을 세밀하게 제어할 수 있는 다양한 방법을 제공한다. 이러한 제어 메커니즘은 시스템 전체에 영향을 미치는 전역 설정부터 개별 노드의 특정 로거에 대한 런타임 동적 변경에 이르기까지 여러 계층에 걸쳐 존재한다. 각 설정 방법의 역할과 우선순위를 이해하는 것은 시스템을 효율적으로 디버깅하고 운영하는 데 필수적이다.


환경 변수는 프로세스 단위로 로깅의 전반적인 동작 방식을 설정하는 데 사용된다. 즉, 하나의 터미널에서 설정된 환경 변수는 해당 터미널에서 실행되는 모든 ROS 2 노드에 동일하게 적용된다.6

**표 1: ROS 2 로깅 관련 주요 환경 변수**

| 변수 이름                         | 설명                                                         | 값 예시 및 의미                                              | 기본 동작                                                    |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `ROS_LOG_DIR`                     | 로그 파일이 저장될 디렉토리를 직접 지정한다. 이 변수가 설정되면 가장 높은 우선순위를 갖는다. | `~/my_ros_logs`                                              | 설정되지 않음                                                |
| `ROS_HOME`                        | `ROS_LOG_DIR`이 설정되지 않았을 때, 로그 파일을 `$ROS_HOME/log` 디렉토리에 저장한다. | `~/.my_ros_home`                                             | `~/.ros`                                                     |
| `RCUTILS_LOGGING_USE_STDOUT`      | 로그 출력 스트림을 제어한다.                                 | `1`: 모든 로그를 `stdout`으로 보낸다. `0` 또는 미설정: 모든 로그를 `stderr`로 보낸다. | `stderr` 사용 6                                              |
| `RCUTILS_LOGGING_BUFFERED_STREAM` | 콘솔 출력의 버퍼링 방식을 제어한다.                          | `1`: 라인 버퍼링을 강제한다. `0`: 버퍼링을 사용하지 않도록 강제한다. | 스트림의 기본값 사용 (`stdout`은 라인 버퍼링, `stderr`는 버퍼링 없음) 6 |
| `RCUTILS_COLORIZED_OUTPUT`        | 콘솔 출력의 색상 사용 여부를 제어한다.                       | `1`: 색상 사용을 강제한다. `0`: 색상 사용을 비활성화한다.    | TTY(터미널) 연결 여부에 따라 자동 결정 6                     |
| `RCUTILS_CONSOLE_OUTPUT_FORMAT`   | 콘솔에 출력되는 각 로그 메시지의 형식을 지정하는 템플릿 문자열. | `[{severity}] {message}`                                     | `[{severity}][{time}][{name}]: {message}` 6                  |

이러한 환경 변수들은 로깅의 '어떻게'를 제어한다. 예를 들어, `RCUTILS_CONSOLE_OUTPUT_FORMAT`은 로그의 모양을, `ROS_LOG_DIR`은 로그의 저장 위치를 결정한다. 특히 `ros2 launch`를 사용할 때 각 노드의 출력이 직접적인 TTY에 연결되지 않을 수 있으므로, `RCUTILS_COLORIZED_OUTPUT=1`로 설정하여 색상 출력을 강제하는 것이 유용할 수 있다.22

**표 2: `RCUTILS_CONSOLE_OUTPUT_FORMAT`의 주요 플레이스홀더**

| 플레이스홀더            | 설명                                               | 출력 예시              |
| ----------------------- | -------------------------------------------------- | ---------------------- |
| `{severity}`            | 로그의 심각도 수준                                 | `INFO`                 |
| `{name}`                | 로거의 전체 이름                                   | `my_node.control`      |
| `{message}`             | 개발자가 작성한 실제 로그 메시지                   | `Motor command sent.`  |
| `{function_name}`       | 로그가 호출된 함수의 이름                          | `send_motor_command`   |
| `{file_name}`           | 로그가 호출된 소스 파일의 이름                     | `motor_controller.cpp` |
| `{line_number}`         | 로그가 호출된 소스 파일의 라인 번호                | `123`                  |
| `{time}`                | Epoch 이후의 시간을 초 단위로 나타낸 부동소수점 값 | `1678886400.123456`    |
| `{time_as_nanoseconds}` | Epoch 이후의 시간을 나노초 단위로 나타낸 정수 값   | `1678886400123456789`  |

이 플레이스홀더들을 조합하여 개발자는 디버깅 목적에 최적화된 맞춤형 로그 포맷을 만들 수 있다. 예를 들어, 개발 중에는 `[{severity}][{name}] {message} ({file_name}:{line_number})` 와 같이 파일명과 라인 번호를 포함시켜 IDE에서 클릭 한 번으로 해당 코드로 바로 이동할 수 있도록 설정하는 것이 매우 유용하다.12


`ros2 run`이나 `ros2 launch` 사용 시 `--ros-args` 플래그를 통해 개별 노드의 로깅 동작을 미세 조정할 수 있다. 이는 환경 변수보다 더 구체적인 제어를 제공한다.18

- `--log-level <level>`: 해당 노드 내에서 명시적으로 수준이 설정되지 않은 모든 로거의 기본 심각도 수준을 지정한다. 예를 들어, `ros2 run my_pkg my_node --ros-args --log-level DEBUG`는 해당 노드의 모든 출력을 `DEBUG` 수준으로 설정한다.24
- `--log-level <logger_name>:=<level>`: 특정 이름을 가진 로거의 심각도 수준을 개별적으로 설정한다. 이는 계층 구조와 결합하여 매우 강력한 제어 기능을 제공한다. 예를 들어, `ros2 run my_pkg my_node --ros-args --log-level my_pkg.planning:=DEBUG`는 `my_pkg` 노드 내의 `planning` 하위 로거만 `DEBUG` 수준으로 설정한다.19
- `--disable-*-logs`: 특정 출력 싱크(sink)를 비활성화한다. `--disable-stdout-logs`(콘솔), `--disable-rosout-logs`(`/rosout`), `--disable-external-lib-logs`(파일)와 같은 플래그를 사용하여 불필요한 로깅 오버헤드를 줄일 수 있다.14


코드 내에서 런타임에 로거의 수준을 동적으로 변경하는 것도 가능하다. 이는 특정 조건이 충족되었을 때 디버깅 상세도를 높이는 등의 시나리오에서 유용하다.

- **C++ (`rclcpp`):** `rcutils_logging_set_logger_level()` 함수를 직접 호출하여 로거 수준을 변경할 수 있다. `logging_demo` 예제에서는 타이머 콜백을 사용하여 프로그램 시작 후 일정 시간이 지나면 노드의 로거 수준을 `DEBUG`로 변경하는 것을 보여준다.11

  ```C++
  // 예시: 5.5초 후에 로거 수준을 DEBUG로 변경
  auto ret = rcutils_logging_set_logger_level(get_logger().get_name(), RCUTILS_LOG_SEVERITY_DEBUG);
  ```

- **Python (`rclpy`):** `rclpy.logging.set_logger_level()` 함수를 사용한다.19

  ```Python
  # 예시: 로거 수준을 DEBUG로 변경
  from rclpy.logging import LoggingSeverity
  rclpy.logging.set_logger_level('my_node_logger', LoggingSeverity.DEBUG)
  ```


ROS 2는 실행 중인 시스템을 재시작하지 않고도 원격으로 로거 수준을 확인하고 변경할 수 있는 표준화된 방법을 서비스(service)를 통해 제공한다. 이 기능은 Humble 배포판에서 안정적으로 지원된다.27

- **제공되는 서비스:** 로깅 제어를 활성화한 노드는 `/node_name/get_logger_levels`와 `/node_name/set_logger_levels`라는 두 개의 서비스를 제공한다.27

- **서비스 타입:** 이 서비스들은 각각 `rcl_interfaces/srv/GetLoggerLevels`와 `rcl_interfaces/srv/SetLoggerLevels`라는 서비스 정의를 사용한다.27

- **사용법:** `ros2 service call` 커맨드를 사용하여 이 서비스들을 호출할 수 있다. 로거 이름 목록과 원하는 수준을 인자로 전달하여 해당 노드가 실행되는 프로세스 내의 모든 로거(ROS 2 코어 라이브러리인 `rcl`의 로거 포함)를 제어할 수 있다.28

  ```Bash
  # talker 노드의 로거 수준을 DEBUG로 설정
  ros2 service call /talker/set_logger_levels rcl_interfaces/srv/SetLoggerLevels "{levels: [{name: 'talker', level: 10}]}"
  ```

- **활성화 방법:** 이 기능은 기본적으로 비활성화되어 있다. 노드 옵션에서 활성화하거나, 컴포넌트 기반 시스템의 경우 `LoggerConfig` 컴포넌트를 컨테이너에 로드하여 사용할 수 있다.27

이러한 다층적인 설정 방식은 명확한 우선순위 체계를 형성한다. 가장 구체적인 설정이 가장 높은 우선순위를 갖는다. 즉, 런타임 서비스 호출이나 프로그래밍 방식의 변경은 모든 이전 설정을 덮어쓴다. 그 다음으로는 특정 로거를 지정한 커맨드 라인 인자(`--log-level name:=level`), 노드 전체의 기본 수준을 지정한 커맨드 라인 인자(`--log-level level`), 그리고 마지막으로 시스템 기본값(`INFO`) 순이다. 환경 변수는 이러한 심각도 수준 설정(무엇을 보여줄지)과는 별개로 출력의 형식과 위치(어떻게 보여줄지)를 제어한다.

Humble 배포판의 중요한 한계점 중 하나는 대규모 시스템의 모든 노드에 대한 로깅 설정을 통합적으로 관리할 수 있는 단일 설정 파일(configuration file)이 없다는 점이다. `ros2 launch`에서 파라미터를 YAML 파일로부터 로드할 수는 있지만 30, 로거 수준, 싱크 활성화 여부, 포맷 등을 체계적으로 기술하는 표준화된 로깅 설정 스키마는 존재하지 않는다. 

`--external_log_config_file` 인자가 존재하지만 기본 백엔드인 `spdlog`에서는 구현되어 있지 않다는 공식 문서의 언급이 이를 뒷받침한다.19 이로 인해 Humble 사용자는 여러 노드의 로깅을 일관되게 설정하기 위해 복잡한 런치 파일 스크립팅에 의존해야 하는 불편함이 있다. 이러한 기능은 이후 배포판인 Iron에서 "External logger configuration"이라는 이름으로 주요 기능으로 추가되었으며 31, 이는 Humble의 중요한 제약 사항임을 보여준다.


ROS 2 로깅 시스템의 모든 기능은 결국 `rclcpp`(C++)와 `rclpy`(Python)에서 제공하는 API를 통해 사용된다. 이 API들은 단순한 메시지 출력을 넘어, 코드의 가독성과 성능을 크게 향상시킬 수 있는 다양한 조건부 로깅 기능을 제공한다. 이러한 고급 기능들을 이해하고 활용하는 것은 전문적인 로봇 소프트웨어 개발의 핵심 요소이다.


`rclcpp`는 전처리기 매크로(preprocessor macro)를 통해 로깅 API를 제공한다. 이는 C++ 개발자에게 익숙한 방식이며, 컴파일 타임에 최적화될 수 있는 여지를 제공한다.

- **기본 매크로:** 가장 기본적으로 사용되는 매크로는 `RCLCPP_DEBUG`, `RCLCPP_INFO`, `RCLCPP_WARN`, `RCLCPP_ERROR`, `RCLCPP_FATAL`이다.14 이들은 각각  `printf` 스타일의 포맷팅과 C++ 스트림(`<<`) 스타일의 포맷팅(`RCLCPP_*_STREAM`)을 모두 지원한다.11

  ```C++
  // printf 스타일
  RCLCPP_INFO(this->get_logger(), "Sensor value: %f", 3.14);
  // 스트림 스타일
  RCLCPP_INFO_STREAM(this->get_logger(), "Sensor value: " << 3.14);
  ```

- **조건부 매크로:** 이 매크로들은 로깅의 성능 오버헤드를 최소화하고 특정 조건에서만 로그를 남기기 위한 강력한 도구다.

  - `RCLCPP_*_ONCE`: 해당 코드 라인이 처음 실행될 때 단 한 번만 로그를 출력한다. 초기화 완료 메시지 등에 유용하다.11

  - `RCLCPP_*_SKIPFIRST`: 처음 실행될 때를 제외하고, 이후 모든 실행에서 로그를 출력한다. 반복문 내에서 첫 번째 반복을 건너뛰고 싶을 때 사용될 수 있다.11

  - `RCLCPP_*_THROTTLE`: 지정된 주기(duration) 내에서 최대 한 번만 로그를 출력한다. 고속으로 반복되는 루프 안에서 주기적으로 상태를 확인하고 싶을 때 매우 유용하다. 이 매크로는 `rclcpp::Clock::SharedPtr` 타입의 시계 객체를 인자로 요구하며, Humble에서는 주기를 밀리초 단위의 정수(integer)로 받는다.11

    ```C++
    // 1초(1000ms)에 한 번씩만 로그 출력
    RCLCPP_INFO_THROTTLE(this->get_logger(), *this->get_clock(), 1000, "Loop is running.");
    ```

    이 API는 사용이 다소 직관적이지 않다는 피드백이 있었고, 이후 버전에서는 `rclcpp::Duration` 객체를 직접 받을 수 있도록 개선되었다.34

  - `RCLCPP_*_EXPRESSION`: 주어진 C++ 표현식(expression)이 `true`로 평가될 때만 로그를 출력한다. 가장 중요한 특징은 **해당 심각도 수준이 활성화되어 있을 때만 표현식이 평가된다**는 점이다. 이는 계산 비용이 비싼 디버그 메시지를 비활성화했을 때 성능 저하가 전혀 없도록 보장한다.14

    ```C++
    // count가 짝수일 때만 로그 출력. DEBUG 레벨이 꺼져있으면 count % 2 연산조차 하지 않음.
    RCLCPP_DEBUG_EXPRESSION(this->get_logger(), (count_ % 2) == 0, "Count is even.");
    ```

  - `RCLCPP_*_FUNCTION`: `_EXPRESSION`과 유사하지만, `bool`을 반환하는 함수를 인자로 받는다. 해당 심각도 수준이 활성화되어 있을 때만 함수가 호출된다.26


`rclpy`는 Python의 관용적인(idiomatic) 스타일을 따라 객체 지향적인 메소드와 키워드 인자(keyword arguments)를 통해 로깅 기능을 제공한다.

- **기본 사용법:** `node.get_logger()`를 통해 로거 객체를 얻은 후, `logger.debug()`, `logger.info()` 등의 메소드를 호출한다.11

  ```Python
  self.get_logger().info(f'Sensor value: {3.14}')
  ```

- **조건부 로깅을 위한 키워드 인자:** C++의 다양한 매크로에 해당하는 기능들이 로깅 메소드의 선택적 키워드 인자로 제공된다.6

  - `once=True`: `RCLCPP_*_ONCE`와 동일한 기능이다.11

  - `skip_first=True`: `RCLCPP_*_SKIPFIRST`와 동일한 기능이다.11

  - `throttle_duration_sec=<seconds>`: 스로틀 주기를 초 단위의 부동소수점 값으로 지정한다. 이는 C++의 밀리초 정수 방식보다 훨씬 직관적이고 사용하기 편리하다.11

    ```Python
    # 1.5초에 한 번씩만 로그 출력
    self.get_logger().info('Loop is running.', throttle_duration_sec=1.5)
    ```

  - `throttle_time_source_type`: 스로틀링에 사용할 시계 타입을 지정할 수 있다.


`rclcpp`와 `rclpy`는 동일한 로깅 철학을 공유하지만, 각 언어의 특성에 맞게 다른 방식으로 API를 제공한다.

**표 3: C++ (`rclcpp`) vs. Python (`rclpy`) 로깅 API 비교**

| 기능             | C++ (`rclcpp`) 구문                                 | Python (`rclpy`) 구문                                       | 주요 차이점                                                  |
| ---------------- | --------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| **기본 로깅**    | `RCLCPP_INFO(logger, "Msg %d", 1);`                 | `logger.info("Msg %d" % 1)`                                 | 매크로 기반 vs. 메소드 기반                                  |
| **스로틀 로깅**  | `RCLCPP_INFO_THROTTLE(logger, clock, 1000, "Msg");` | `logger.info("Msg", throttle_duration_sec=1.0)`             | C++은 시계 객체와 밀리초 정수 필요, Python은 초 단위 부동소수점 사용 |
| **한 번만 로깅** | `RCLCPP_INFO_ONCE(logger, "Msg");`                  | `logger.info("Msg", once=True)`                             | 매크로 이름에 기능 명시 vs. 키워드 인자로 기능 지정          |
| **조건부 로깅**  | `RCLCPP_INFO_EXPRESSION(logger, expr, "Msg");`      | (별도 기능 없음, `if`문 사용) `if expr: logger.info("Msg")` | C++은 표현식 평가를 지연시키는 매크로 제공, Python은 일반적인 `if`문 사용 |

**모범 사례:**

- **C++:**
  - `printf` 포맷팅 오류를 방지하고 타입 안전성을 높이기 위해 스트림 기반(`RCLCPP_*_STREAM`) 로깅을 선호하는 것이 좋다.
  - 성능에 민감한 루프 내에서 복잡한 디버그 메시지를 생성할 때는, 반드시 `RCLCPP_DEBUG_EXPRESSION`이나 `RCLCPP_DEBUG_FUNCTION`으로 감싸거나, `if (get_logger().get_effective_level() <= RCUTILS_LOG_SEVERITY_DEBUG)`와 같이 수동으로 레벨을 확인해야 한다. 이렇게 하지 않으면 `DEBUG` 레벨이 비활성화된 상태에서도 메시지 문자열을 생성하는 데 드는 비용이 그대로 발생하여 심각한 성능 저하를 유발할 수 있다.
  - 스로틀 매크로 사용 시, `this->get_clock()`의 반환 타입은 `SharedPtr`이므로, 매크로에 전달하기 전에 `*this->get_clock()`과 같이 역참조(dereference)하여 실제 시계 객체를 전달해야 한다. 이는 매크로 구현의 특성 때문이다.33
- **Python:**
  - 현대적이고 가독성이 높은 f-string을 사용하여 메시지 포맷팅을 하는 것이 권장된다: `logger.info(f'Value is {my_var}')`.
  - `throttle_duration_sec` 인자는 C++의 방식보다 직관적이며 오류 발생 가능성이 적다.

조건부 로깅 기능들은 단순히 편의를 위한 것이 아니라, 중요한 성능 최적화 도구라는 점을 인식하는 것이 매우 중요하다. 로깅으로 인한 오버헤드는 실시간 시스템에서 예측 불가능한 지연(jitter)을 유발할 수 있으며, 이러한 조건부 매크로/인자들은 로깅 기능이 비활성화되었을 때의 비용을 거의 0에 가깝게 만들어준다. 이는 ROS 2 API가 실시간성을 염두에 두고 설계되었음을 보여주는 좋은 예이다.35


ROS 2 로깅 시스템의 표면 아래에는 로그를 실제로 처리하고 저장하는 백엔드 구현이 존재한다. Humble에서는 `rcl_logging_spdlog`가 이 역할을 담당하며, 시스템의 유연성과 성능 한계를 결정하는 중요한 요소이다. 더 나아가, 실시간 요구사항을 충족시키기 위해서는 기본 로깅 시스템의 동작 방식을 이해하고 이를 최적화하는 고급 기법이 필요하다.


`rcl_logging_spdlog`는 ROS 2의 "외부 로깅(external logging)" 핸들러에 대한 기본 구현체로서, 디스크에 로그 파일을 쓰는 책임을 진다.6

- **`spdlog` 라이브러리:** 이 백엔드는 `spdlog`라는 매우 유명하고 성능이 뛰어난 C++ 로깅 라이브러리를 기반으로 한다. `spdlog`는 빠른 속도, 스레드 안전성, 유연한 포맷팅 등으로 널리 알려져 있으며, ROS 2는 이 라이브러리의 장점을 활용하여 효율적인 파일 로깅을 구현한다.9

- **Humble에서의 설정 한계:** `spdlog` 자체는 로그 파일 로테이션(오래된 로그 파일을 자동으로 삭제하거나 압축하는 기능), 파일 크기 제한, 출력 포맷 커스터마이징 등 다양한 고급 설정 기능을 제공한다. 하지만 ROS 2 Humble의 `rcl_logging_spdlog` 구현에서는 이러한 기능들을 ROS 수준에서 표준화된 방식으로 제어할 방법이 없다. `--external_log_config_file`이라는 커맨드 라인 인자가 존재하기는 하지만, 이는 `spdlog` 백엔드에 대해 구현되어 있지 않다.19 따라서 로그 파일 로테이션과 같은 고급 기능을 사용하고자 하는 개발자는 

  `rcl_logging_spdlog` 패키지의 소스 코드를 직접 수정하고 재컴파일해야 하는 실질적인 제약이 있다.7


ROS 2 로깅 시스템의 설계는 처음부터 다양한 백엔드를 지원할 수 있도록 확장성을 염두에 두었다.

- **초기 논의 (`log4cxx`):** ROS 2 개발 초기에는 `log4cxx`를 백엔드로 사용하는 방안이 비중 있게 논의되었고, 실제 구현체도 만들어졌다. `log4cxx`는 Java의 `log4j`에 기반한 강력한 기능(XML 기반 설정 등)을 제공했지만, 종료 시 메모리 안전성 문제로 인해 간헐적으로 세그멘테이션 폴트(segmentation fault)를 일으키는 이슈가 발견되어 최종적으로 기본 백엔드로 채택되지는 않았다.5 이 이력은 ROS 2가 강력한 서드파티 라이브러리를 적극적으로 통합하려는 의지를 가지고 있음을 보여준다.

- **컴파일 타임 선택:** Humble을 포함한 이전 버전들에서 로깅 백엔드는 **컴파일 타임**에 결정된다. `rcl` 패키지를 빌드하기 전에 `RCL_LOGGING_IMPLEMENTATION` 환경 변수를 설정함으로써 사용할 백엔드를 선택할 수 있다 (예: `export RCL_LOGGING_IMPLEMENTATION=rcl_logging_log4cxx`).7 이는 매우 큰 제약 사항인데, 백엔드 선택이 ROS 2의 핵심 라이브러리에 고정되어 전체 워크스페이스에 동일하게 적용되기 때문이다.

- **동적 로딩으로의 전환 요구:** 이러한 제약을 극복하기 위해 커뮤니티에서는 로깅 백엔드를 **런타임**에 동적으로 선택할 수 있게 해달라는 요구가 꾸준히 제기되었다. 이는 `RMW_IMPLEMENTATION` 환경 변수를 통해 DDS 미들웨어를 런타임에 교체하는 방식과 유사하다.38 이 기능이 구현된다면, 개발자는 ROS를 재빌드하지 않고도 애플리케이션의 요구에 따라 각기 다른 로깅 백엔드를 사용할 수 있게 된다. 예를 들어, 대부분의 노드는 기본 

  `spdlog` 파일 로거를 사용하고, 성능이 매우 중요한 제어 노드는 고속 인메모리(in-memory) 로거를 사용하며, 시스템 관리와 연동이 필요한 노드는 `syslog` 전달자(forwarder)를 사용하는 등의 유연한 구성이 가능해진다.37 이러한 변화는 ROS 2가 정적이고 컴파일 타임에 결정되던 선택들을 점차 동적이고 런타임에 결정되는 유연한 구조로 발전해 나가는 더 큰 흐름의 일부이다.


실시간(Real-Time) 시스템에서는 예측 가능하고 결정론적인(deterministic) 시간 내에 작업을 완료하는 것이 매우 중요하다. 하지만 일반적인 로깅 작업은 이러한 요구사항에 정면으로 위배되는 특성을 많이 가지고 있다.

- **문제점:** 로그 메시지를 생성하고 출력하는 과정에는 다음과 같은 비결정론적 요소들이 포함될 수 있다.35
  - **문자열 포맷팅:** `printf` 계열 함수나 스트림 연산은 내부적으로 복잡한 문자열 처리와 동적 메모리 할당을 수행할 수 있다.
  - **I/O 작업:** 로그를 콘솔이나 파일에 쓰는 것은 운영체제의 스케줄링에 따라 지연 시간이 크게 변동될 수 있는 대표적인 블로킹(blocking) I/O 작업이다.
  - **뮤텍스 잠금:** 멀티스레드 환경에서 로깅의 안전성을 보장하기 위한 뮤텍스(mutex) 잠금은 우선순위 역전(priority inversion)과 같은 문제를 야기하여 실시간 스레드의 실행을 예기치 않게 지연시킬 수 있다.
- **완화 기법:** 이러한 문제를 완화하고 실시간 성능에 대한 로깅의 영향을 최소화하기 위해 다음과 같은 기법들을 사용할 수 있다.
  - **불필요한 싱크 비활성화:** 실시간성이 중요한 노드에서는 `--disable-rosout-logs`와 `--disable-external-lib-logs` 같은 커맨드 라인 인자를 사용하여 네트워크 및 파일 I/O 오버헤드를 원천적으로 차단하는 것이 좋다.14
  - **조건부 로깅 적극 활용:** 4장에서 설명한 `RCLCPP_*_EXPRESSION`과 같은 조건부 매크로를 사용하여, 비활성화된 로그 레벨에 대해서는 어떠한 연산도 수행되지 않도록 보장해야 한다.26
  - **비동기 로깅 (Asynchronous Logging):** 기본 로깅은 동기식(synchronous)으로, 로깅 호출이 완료될 때까지 해당 스레드는 대기한다. 고급 기법으로, ROS 2의 전역 로깅 핸들러를 직접 교체하여 비동기 싱크를 구현할 수 있다. 이는 실시간 스레드에서는 로그 메시지를 락프리(lock-free) 큐에 넣는 최소한의 작업만 수행하고, 실제 포맷팅과 I/O 작업은 별도의 낮은 우선순위 스레드에서 처리하는 방식이다. 이 방법은 로깅이 실시간 루프에 미치는 영향을 극적으로 줄일 수 있다.37
  - **메모리 관리:** 실시간 경로에서는 `new`, `malloc`과 같은 동적 메모리 할당을 피해야 한다. 일부 로깅 호출이 내부적으로 메모리를 할당할 수 있다는 점에 유의해야 한다.21 이를 피하기 위해 메모리 풀(memory pool)이나 정적으로 할당된 버퍼를 사용하는 등의 고급 메모리 관리 기법이 필요할 수 있다.35
  - **시스템 수준 최적화:** `PREEMPT_RT` 패치가 적용된 실시간 리눅스 커널을 사용하고, `SCHED_FIFO`와 같은 실시간 스케줄링 정책을 적용하며, 메모리 잠금(`mlockall`)을 통해 페이지 폴트(page fault)를 방지하는 등 시스템 전반의 최적화가 병행되어야 한다.36

결론적으로, ROS 2 Humble의 기본 로깅 시스템은 일반적인 용도로는 매우 편리하지만, 고성능 및 실시간 애플리케이션의 엄격한 요구사항을 충족시키기에는 한계가 명확하다. 커뮤니티에서는 이러한 한계를 극복하기 위해 독자적인 락프리 인메모리 백엔드를 개발하거나 37, 기본 핸들러를 커스텀 비동기 싱크로 교체하는 등 37 다양한 노력이 이루어지고 있다. 이는 안전이 중요하거나 고주파 제어가 필요한 시스템을 구축하는 개발자에게 기본 로깅 시스템이 최종 해결책이 아니라, 필요에 따라 수정하거나 교체해야 할 시작점임을 시사하는 중요한 사실이다.


ROS 2 Humble의 로깅 시스템을 올바르게 평가하기 위해서는 이를 더 넓은 맥락, 즉 ROS 1과의 비교 및 ROS 2 자체의 발전 과정 속에서 조망할 필요가 있다. 이러한 비교는 시스템의 설계 의도를 파악하고 현재 기능의 장단점을 이해하는 데 도움을 준다.


ROS 1의 로깅 시스템인 `rosconsole`과 ROS 2의 `rcl_logging`은 표면적으로 유사한 기능을 제공하지만, 그 기반이 되는 아키텍처와 철학에는 근본적인 차이가 있다.

**표 4: ROS 1 (`rosconsole`) vs. ROS 2 Humble (`rcl_logging`) 로깅 시스템 비교**

| 기능                   | ROS 1 (`rosconsole`)                                         | ROS 2 Humble (`rcl_logging`)                                 |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **핵심 아키텍처**      | `log4cxx` 라이브러리를 직접 래핑하는 비교적 단일체적인 구조 16 | `rcutils`, `rcl`, `rmw` 등 여러 계층으로 분리된 모듈식 구조 4 |
| **기본 백엔드**        | `log4cxx` (파일 로깅, XML 설정 등 강력한 기능 제공) 16       | `rcl_logging_spdlog` (파일 로깅) + `rcutils` (콘솔 출력) 6   |
| **설정 방법**          | `ROSCONSOLE_CONFIG_FILE` 환경 변수를 통해 `log4cxx` XML/properties 설정 파일 지정 16 | 환경 변수, 커맨드 라인 인자, 서비스 호출의 조합. 통합 설정 파일 부재 19 |
| **`/rosout` 메커니즘** | 중앙 집중형 ROS Master를 통해 `/rosout` 토픽에 대한 정보를 중개하고 발행 13 | DDS 미들웨어를 통해 분산 방식으로 `/rosout` 토픽을 직접 발행 및 발견 2 |
| **API 스타일**         | `ROS_` 접두사를 사용하는 C++ 매크로 중심 (`roscpp`) 41       | C 기반의 `rcutils` 위에 `rclcpp` 매크로와 `rclpy` 객체를 구현하여 언어 간 일관성 추구 2 |

이 비교를 통해 몇 가지 핵심적인 차이점을 도출할 수 있다. 첫째, ROS 1은 `log4cxx`라는 강력하지만 무거운 단일 백엔드에 깊이 의존했던 반면, ROS 2는 경량의 `rcutils`를 기반으로 백엔드를 교체할 수 있는 유연한 구조를 채택했다. 둘째, 설정 방식에서 ROS 1은 `log4cxx`의 강력한 파일 기반 설정 기능을 그대로 활용할 수 있었던 반면, ROS 2 Humble은 이 부분이 약점이다. 대신 ROS 2는 커맨드 라인과 서비스를 통한 동적 제어 기능을 강화했다. 마지막으로, `/rosout` 메커니즘의 변화는 ROS 1의 중앙 집중식 마스터-슬레이브 구조에서 ROS 2의 완전한 분산 시스템으로의 전환을 상징적으로 보여준다. ROS 1에서 마이그레이션하는 개발자는 이러한 철학적, 실용적 차이점을 명확히 인지해야 혼란을 줄이고 ROS 2의 장점을 온전히 활용할 수 있다.


ROS 2 로깅 시스템은 단번에 완성된 것이 아니라, 여러 배포판을 거치며 점진적으로 발전해왔다.

- **Dashing/Eloquent:** 이 초기 배포판들은 로깅 시스템의 핵심 개념(심각도 수준, 계층 구조, `rcutils` 기반 구조)을 정립했다. 하지만 기능적으로는 미완성인 부분이 많았다. 예를 들어, Dashing에서는 심각도 수준에 따라 로그 출력 스트림이 `stdout`과 `stderr`로 나뉘는 등 현재와 다른 동작을 보였다.24 Eloquent의 공식 문서에서는 파일 출력과 런타임 외부 설정 기능이 여전히 "향후 제공될 예정(forthcoming)"이라고 명시되어 있었다.8

- **Foxy:** Foxy에 이르러 로깅 시스템은 상당히 안정화되었다. 모든 레벨의 로그 출력이 `stderr`로 통일되었고 12, 

  `rcl_logging_spdlog`를 통한 파일 로깅이 기본 기능으로 자리 잡았다. 전반적으로 Foxy의 로깅 시스템은 Humble과 매우 유사하며, Humble은 Foxy에서 다져진 기능들을 더욱 안정화하고 성숙시킨 버전이라고 볼 수 있다. 다만, 외부 설정 파일 미구현과 같은 한계점은 여전히 남아있었다.19

- **Humble (본 보고서의 초점):** Foxy 시대의 기능들이 안정화된 LTS 버전이다. 프로그래밍 방식 및 서비스를 통한 동적 제어 API가 안정적으로 제공되며, 산업 현장에서 신뢰하고 사용할 수 있는 수준의 성숙도에 도달했다.6


Humble 이후의 배포판들은 사용자의 피드백을 반영하여 중요한 사용성 개선을 이루었다.

- **Iron Irwini:** 로깅 시스템에 있어 중요한 이정표가 된 배포판이다. 가장 큰 변화는 **외부 로거 설정(External logger configuration)** 기능의 도입이다.31 이는 Humble의 가장 큰 약점이었던 통합 설정 파일의 부재를 해결하는 기능으로, 파일 기반으로 여러 노드의 로거 수준 등을 정교하게 제어할 수 있게 되었다.
- **Jazzy Jalisco:** 가장 최신 LTS 배포판인 Jazzy는 편의성을 높이는 개선 사항을 추가했다. 대표적으로 `--log-file-name` 커맨드 라인 인자가 추가되어, 사용자가 `ROS_LOG_DIR`과 같은 환경 변수에 의존하지 않고도 각 노드의 로그 파일 이름을 쉽게 지정할 수 있게 되었다.42
- **미래 방향:** 로깅 시스템의 발전은 여기서 멈추지 않을 것이다. 백엔드의 동적 로딩 기능에 대한 지속적인 요구 38와 로그 로테이션, 포맷팅 등 백엔드 자체에 대한 고급 설정 기능의 표준화 요구 9는 시스템이 앞으로 나아갈 방향을 시사한다. 이는 로깅 시스템이 일반적인 개발 편의성을 넘어, 산업용 및 실시간 시스템의 더욱 엄격한 요구사항을 충족시키기 위해 지속적으로 발전하고 있음을 보여준다.

이러한 발전 과정을 살펴보면, ROS 2 로깅 시스템이 핵심 기능 구현(Eloquent), 기능 안정화(Foxy/Humble), 주요 사용성 격차 해소(Iron), 그리고 편의성 개선(Jazzy)이라는 단계를 거쳐 성숙해왔음을 알 수 있다. Humble 사용자의 관점에서 이러한 맥락을 이해하는 것은 현재 시스템의 한계를 인지하고, 필요할 경우 최신 배포판으로의 업그레이드를 고려하는 데 중요한 기준이 될 수 있다.


지금까지 분석한 ROS 2 Humble 로깅 시스템의 기술적 세부 사항들을 실제 복잡한 로봇 애플리케이션에 효과적으로 적용하기 위해서는 체계적인 전략이 필요하다. 단순히 로그를 출력하는 것을 넘어, 시스템의 상태를 효율적으로 모니터링하고, 문제를 신속하게 진단하며, 장기적인 분석을 위한 데이터를 축적하는 종합적인 접근법이 요구된다.


`rqt_console`은 단순히 `/rosout` 토픽의 내용을 보여주는 뷰어를 넘어, 능동적인 디버깅 도구로 활용될 수 있다.18

- **필터링:** 대규모 시스템에서는 수많은 `INFO` 메시지가 쏟아져 나와 정작 중요한 `WARN`이나 `ERROR` 메시지를 놓치기 쉽다. `rqt_console`의 제외(exclude) 필터 기능을 사용하여 특정 심각도 수준(예: `Info`, `Debug`)이나 특정 로거 이름의 메시지를 숨김으로써 노이즈를 제거하고 문제 상황에 집중할 수 있다.18
- **하이라이팅:** 강조(highlight) 필터는 특정 문자열(예: "timeout", "failed", "goal reached")을 포함하는 메시지를 눈에 띄게 표시해준다. 이를 통해 복잡한 로그 스트림 속에서 특정 이벤트의 발생을 즉시 인지할 수 있다.18
- **전략적 활용:** 이 두 기능을 조합하면 특정 디버깅 시나리오에 최적화된 뷰를 만들 수 있다. 예를 들어, 내비게이션 스택의 실패 원인을 분석할 때, `Info`와 `Debug` 메시지는 모두 제외하고, "nav", "planner", "failed", "exception"과 같은 키워드를 포함하는 메시지만 하이라이트하도록 설정하면 문제와 관련된 로그에만 집중할 수 있다.


기본 콘솔 로그 포맷 `[{severity}][{time}][{name}]: {message}`는 많은 정보를 담고 있지만, 때로는 너무 장황하여 가독성을 해칠 수 있다.23

- **정보 밀도 조절:** `RCUTILS_CONSOLE_OUTPUT_FORMAT` 환경 변수를 활용하여 로그의 목적에 맞게 출력 형식을 조절하는 것이 모범 사례이다.
  - **개발 단계:** `[{severity}][{name}]: {message} ({file_name}:{line_number})`와 같이 파일명과 라인 번호를 포함시키면, VS Code와 같은 최신 IDE의 터미널에서 해당 로그를 클릭했을 때 소스 코드의 해당 위치로 바로 이동하는 매우 편리한 기능을 활용할 수 있다.12
  - **배포 및 운영 단계:** 현장 운영 시에는 파일명이나 라인 번호보다는 간결한 메시지가 더 유용할 수 있다. `[{severity}][{name}]: {message}`와 같이 단순화하여 운영자가 시스템 상태를 빠르게 파악할 수 있도록 돕는다.
- **고급 포맷팅:** 일부 고급 사용자는 ANSI 이스케이프 시퀀스를 포맷 문자열에 직접 삽입하여 색상, 굵기, 밑줄 등을 제어하거나 심지어 커서 위치를 조작하여 매우 가독성 높은 출력을 만들기도 한다. 이는 표준 기능은 아니지만, 터미널 환경에 익숙한 사용자에게는 강력한 커스터마이징 옵션을 제공한다.23


`ros2 launch`는 여러 노드를 실행하고 설정하는 중심적인 역할을 하므로, 대규모 시스템의 로깅을 체계적으로 관리하는 핵심 지점이다.44

- **중앙 집중식 제어:** 각 노드의 로깅 수준을 런치 파일 내에서 명시적으로 설정하는 것이 좋다. 특히, 로깅 수준을 런치 인자(Launch Argument)로 노출시키고 기본값을 지정하면, 커맨드 라인에서 `log_level:=DEBUG`와 같이 특정 실행에 대해서만 설정을 쉽게 덮어쓸 수 있어 유연성이 크게 향상된다.
- **구조적 로거 이름 활용:** 런치 파일에서 노드 이름과 네임스페이스를 체계적으로 부여하여, 2.2절에서 설명한 로거 계층 구조를 의도적으로 설계해야 한다. 예를 들어, 모든 인식 관련 노드를 `perception` 네임스페이스에 배치하면, `ros2 service call`을 통해 `perception` 로거의 수준을 변경함으로써 하위의 모든 인식 노드들의 로깅 상세도를 한 번에 제어할 수 있다.
- **YAML 파라미터 활용:** 비록 Humble에 표준 로깅 설정 파일은 없지만, YAML 파일을 통해 노드 파라미터를 전달하는 기능은 잘 지원된다.30 만약 특정 노드가 내부적으로 로깅 관련 설정을 파라미터로 받도록 직접 구현했다면, 런치 파일에서 이 YAML 파일을 로드하여 로깅을 설정하는 방식을 사용할 수 있다.


`/rosout`은 실시간 모니터링에는 훌륭하지만, 영구적이고 통합된 로깅 솔루션은 아니다. 네트워크 상태가 좋지 않으면 메시지가 유실될 수 있으며, 수십, 수백 개의 노드가 고주파로 로그를 쏟아내는 환경에서는 확장성에 한계가 있다.

- **ROS 외부 도구와의 연동:** 상용 수준의 로봇 시스템에서는 ROS 네이티브 도구를 넘어, 업계 표준의 외부 로깅 인프라와 연동하는 것이 일반적이다.
- **통합 전략:**
  1. 각 ROS 노드는 로그를 로컬 파일에 기록하도록 설정한다 (`--disable-rosout-logs`를 사용하여 네트워크 부하를 줄일 수 있다).
  2. 각 머신(로봇, 서버 등)에 `Fluentd`, `Logstash`, `rsyslog`와 같은 로그 수집 에이전트를 설치한다.
  3. 이 에이전트들이 ROS 로그 파일의 변경 사항을 감지(tail)하여, 파싱하고 구조화한 뒤 중앙 로그 서버(예: Elasticsearch, Graylog, Loki)로 전송한다.
  4. 개발자와 운영자는 Kibana, Grafana와 같은 대시보드 도구를 통해 중앙 서버에 수집된 모든 시스템의 로그를 검색, 분석, 시각화한다.

이러한 접근 방식은 ROS 커뮤니티 내에서도 대규모 시스템의 로그 관리를 위한 견고한 해결책으로 인식되고 있으며, `rsyslog`나 `FluentBit`과 ROS 2를 통합하는 것에 대한 논의가 이를 뒷받침한다.7

결론적으로, 대규모 ROS 2 시스템에서 효과적인 로깅은 단일 도구나 기술에 의존하는 것이 아니라, 시스템의 생애주기와 목적에 따라 다양한 도구를 조합하는 시스템 수준의 설계 문제이다. 실시간 디버깅에는 `rqt_console`과 동적 서비스 호출을, 정적 설정에는 런치 파일을, 그리고 장기적인 분석과 운영에는 외부 로그 통합 시스템을 사용하는 다층적인 전략이 필요하다. 개발자는 인간의 가독성을 위한 콘솔 출력과 기계의 파싱 용이성을 위한 파일 출력이 서로 다른 요구사항을 가질 수 있음을 인지하고, 각 출력의 소비자가 누구인지를 고려하여 로깅 전략을 수립해야 한다.


본 보고서는 ROS 2 Humble Hawksbill의 로깅 시스템에 대한 다각적이고 심층적인 분석을 제공했다. 분석 결과, Humble의 로깅 시스템은 ROS 1에서 비약적인 발전을 이루어, 모듈식 아키텍처, 다양한 출력 경로, 그리고 강력한 런타임 제어 기능을 갖춘 정교한 하위 시스템으로 자리 잡았음을 확인했다.

**ROS 2 Humble 로깅 시스템의 강점**은 명확하다. 첫째, `rcutils`, `rcl`, `rmw`로 이어지는 **계층적이고 모듈화된 구조**는 시스템의 유연성과 확장성을 보장하며, 문제 발생 시 원인 분석을 용이하게 한다. 둘째, 콘솔, 파일, 그리고 `/rosout` 토픽이라는 **세 가지 독립적인 출력 경로**는 개발자에게 실시간 모니터링, 영구 기록, 분산 환경 관찰이라는 각기 다른 목적에 맞춰 로깅을 활용할 수 있는 선택지를 제공한다. 셋째, 커맨드 라인 인자와 서비스를 통한 **강력한 런타임 제어 기능**은 시스템을 재시작하지 않고도 로깅 상세도를 동적으로 조절할 수 있게 하여 디버깅 효율성을 극대화한다. 마지막으로, `RCLCPP_*_EXPRESSION`과 같은 **성능 최적화 API**는 로깅이 실시간 성능에 미치는 영향을 최소화하려는 ROS 2의 설계 철학을 명확히 보여준다.

반면, **Humble 로깅 시스템의 한계와 과제** 또한 존재한다. 가장 두드러지는 약점은 대규모 시스템에서 여러 노드의 로깅 설정을 일관되게 관리할 수 있는 **표준화된 통합 설정 파일의 부재**이다. 이는 이후 Iron 배포판에서 해결되었지만, Humble 사용자에게는 여전히 불편함으로 남아있다. 또한, 로깅 백엔드가 **컴파일 타임에 정적으로 선택**된다는 점은 다양한 요구사항을 가진 애플리케이션을 단일 환경에서 운영해야 할 때 유연성을 저해하는 요소이다. 마지막으로, 기본 로깅 시스템은 일반적인 용도에는 충분하지만, 엄격한 **실시간 요구사항을 충족시키기에는 본질적인 오버헤드**를 가지고 있어, 고성능 시스템을 위해서는 비동기 로깅과 같은 고급 사용자 정의 기법이 요구된다.

본 고찰을 통해 얻은 핵심적인 통찰은 ROS 2 로깅 시스템이 단순한 출력 도구가 아니라, **시스템의 상태를 관찰하고 제어하는 능동적인 인터페이스**라는 점이다. 로거의 계층 구조를 체계적으로 설계하고, 다양한 설정 방법을 목적에 맞게 조합하며, 외부 도구와의 연동을 통해 로그 데이터를 자산으로 활용하는 전략적 접근이 필요하다.

ROS 2 로깅 시스템은 Humble 이후에도 Iron과 Jazzy를 거치며 꾸준히 발전하고 있다. 외부 설정 파일 지원, 편의 기능 추가, 그리고 동적 백엔드 로딩과 같은 향후 발전 방향은 로깅 시스템이 산업 현장의 복잡하고 엄격한 요구사항을 수용하기 위해 지속적으로 진화하고 있음을 시사한다. 따라서 ROS 2 개발자는 현재 배포판의 기능을 최대한 활용하면서도, 이러한 발전 동향을 주시하며 자신의 시스템에 가장 적합한 로깅 전략을 끊임없이 모색해야 할 것이다. 결국, 잘 설계된 로깅 시스템은 견고하고 신뢰성 높은 로봇 시스템을 구축하는 데 있어 가장 중요한 초석 중 하나가 될 것이다.


1. ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/index.html
2. ROS 1 vs ROS 2 What are the Biggest Differences? - Model Prime, accessed July 26, 2025, https://www.model-prime.com/blog/ros-1-vs-ros-2-what-are-the-biggest-differences
3. Concepts - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Concepts.html
4. Internal ROS 2 interfaces - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Concepts/Advanced/About-Internal-Interfaces.html
5. ROS2 Logging - ROS General - Open Robotics Discourse, accessed July 26, 2025, https://discourse.ros.org/t/ros2-logging/6469
6. Logging and logger configuration - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Concepts/Intermediate/About-Logging.html
7. ROS2 Logging - Page 2 - ROS Discourse, accessed July 26, 2025, https://discourse.ros.org/t/ros2-logging/6469?page=2
8. About logging and logger configuration - ROS 2 Documentation, accessed July 26, 2025, https://docs.ros.org/en/eloquent/Concepts/Logging.html
9. proposal for how to unify logging and make it configurable with a file / Issue #92 / ros2/rcl_logging - GitHub, accessed July 26, 2025, https://github.com/ros2/rcl_logging/issues/92
10. About logging and logger configuration - ROS 2 Documentation: Foxy 文档, accessed July 26, 2025, https://daobook.github.io/ros2-docs/foxy/Concepts/About-Logging.html
11. Logging - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Demos/Logging-and-logger-configuration.html
12. Logging - ROS 2 Documentation: Foxy documentation, accessed July 26, 2025, https://docs.ros.org/en/foxy/Tutorials/Demos/Logging-and-logger-configuration.html
13. rosout - ROS Wiki, accessed July 26, 2025, http://wiki.ros.org/rosout
14. ros2_documentation/source/Concepts/Intermediate/About-Logging.rst at rolling - GitHub, accessed July 26, 2025, https://github.com/ros2/ros2_documentation/blob/rolling/source/Concepts/Intermediate/About-Logging.rst
15. Changes between ROS 1 and ROS 2 - ROS2 Design, accessed July 26, 2025, http://design.ros2.org/articles/changes.html
16. rosconsole - ROS Wiki, accessed July 26, 2025, http://wiki.ros.org/rosconsole
17. What is the difference between ROS and ROS2? - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/86338/what-is-the-difference-between-ros-and-ros2
18. Using rqt_console to view logs - ROS 2 Documentation: Humble ..., accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Using-Rqt-Console/Using-Rqt-Console.html
19. About logging and logger configuration - ROS 2 Documentation: Foxy documentation, accessed July 26, 2025, https://docs.ros.org/en/foxy/Concepts/About-Logging.html
20. ROS2 - rosout using rclpy.logging.get_logger() - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/116880/ros2-rosout-using-rclpy-logging-get-logger
21. ROS2_assorted_docs/ROS2_core/ROS2_logging.md at main - GitHub, accessed July 26, 2025, https://github.com/jrutgeer/ROS2_assorted_docs/blob/main/ROS2_core/ROS2_logging.md
22. Logging - ROS 2 Documentation: Jazzy documentation, accessed July 26, 2025, https://docs.ros.org/en/jazzy/Tutorials/Demos/Logging-and-logger-configuration.html
23. Better Console Logging - ROS General - Open Robotics Discourse, accessed July 26, 2025, https://discourse.ros.org/t/better-console-logging/44012
24. Logging and logger configuration demo - ROS 2 Documentation, accessed July 26, 2025, https://docs.ros.org/en/dashing/Tutorials/Logging-and-logger-configuration.html
25. Problem when trying to start a ROS2 node and trying to set the log level, accessed July 26, 2025, https://robotics.stackexchange.com/questions/115099/problem-when-trying-to-start-a-ros2-node-and-trying-to-set-the-log-level
26. demos/logging_demo/src/logger_usage_component.cpp at rolling / ros2/demos - GitHub, accessed July 26, 2025, https://github.com/ros2/demos/blob/rolling/logging_demo/src/logger_usage_component.cpp
27. External logging level configuration / Issue #1355 / ros2/ros2 - GitHub, accessed July 26, 2025, https://github.com/ros2/ros2/issues/1355
28. Logging - ROS 2 Documentation: Rolling documentation, accessed July 26, 2025, https://docs.ros.org/en/rolling/Tutorials/Demos/Logging-and-logger-configuration.html
29. Logging and logger configuration demo - ROS 2 Documentation, accessed July 26, 2025, https://docs.ros.org/en/eloquent/Tutorials/Logging-and-logger-configuration.html
30. ROS2 YAML For Parameters - The Robotics Back-End, accessed July 26, 2025, https://roboticsbackend.com/ros2-yaml-params/
31. Open Robotics releases ROS 2 Iron Irwini - The Robot Report, accessed July 26, 2025, https://www.therobotreport.com/open-robotics-releases-ros-2-iron-irwini/
32. rclcpp "throttle" logging / Issue #797 - GitHub, accessed July 26, 2025, https://github.com/ros2/rclcpp/issues/797
33. ros2 throttle logs - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/95855/ros2-throttle-logs
34. Make throttle log macros accept a rclcpp::Duration / Issue #1929 - GitHub, accessed July 26, 2025, https://github.com/ros2/rclcpp/issues/1929
35. Advanced ROS2 Topics for Robotics - Number Analytics, accessed July 26, 2025, https://www.numberanalytics.com/blog/advanced-ros2-topics-for-robotics
36. Understanding real-time programming - ROS 2 Documentation: Foxy documentation, accessed July 26, 2025, https://docs.ros.org/en/foxy/Tutorials/Demos/Real-Time-Programming.html
37. Improving ROS2 logging with spdlog and fmt - Projects - ROS Discourse - Open Robotics, accessed July 26, 2025, https://discourse.openrobotics.org/t/improving-ros2-logging-with-spdlog-and-fmt/38817
38. [FR] select the logger implementation without rebuilding / Issue ..., accessed July 26, 2025, https://github.com/ros2/rcl/issues/1178
39. Better Console Logging - ROS General - Open Robotics Discourse, accessed July 26, 2025, https://discourse.openrobotics.org/t/better-console-logging/44012
40. (PDF) ROS2 Real-time Performance Optimization and Evaluation - ResearchGate, accessed July 26, 2025, https://www.researchgate.net/publication/376215526_ROS2_Real-time_Performance_Optimization_and_Evaluation
41. roscpp/Overview/Logging - ROS Wiki, accessed July 26, 2025, http://wiki.ros.org/roscpp/Overview/Logging
42. Jazzy Jalisco (jazzy) - ROS 2 Documentation, accessed July 26, 2025, https://docs.ros.org/en/jazzy/Releases/Release-Jazzy-Jalisco.html
43. OSRF releases ROS 2 Jazzy Jalisco - The Robot Report, accessed July 26, 2025, https://www.therobotreport.com/ros-2-jazzy-jalisco-released-osrf/
44. ROS 2 Launch System - ROS2 Design, accessed July 26, 2025, https://design.ros2.org/articles/roslaunch.html
45. How to Use ROS 2 Launch Files - Foxglove, accessed July 26, 2025, https://foxglove.dev/blog/how-to-use-ros2-launch-files
46. Managing large projects - ROS 2 Documentation: Rolling documentation, accessed July 26, 2025, https://docs.ros.org/en/rolling/Tutorials/Intermediate/Launch/Using-ROS2-Launch-For-Large-Projects.html


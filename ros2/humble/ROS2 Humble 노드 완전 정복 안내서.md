# ROS 2 Humble 노드 완전 정복 안내서

## 서론: 왜 노드를 이해해야 하는가?

ROS 2(Robot Operating System 2)로 구축된 모든 로봇 시스템의 근간에는 **노드(Node)** 가 있다. 노드는 로봇 애플리케이션을 구성하는 가장 기본적인 실행 및 연산 단위다.1 복잡하고 거대한 로봇 시스템을 작고, 독립적이며, 재사용 가능한 모듈로 분해하는 것이 바로 노드의 핵심 철학이다.3 따라서 노드를 얼마나 잘 이해하고, 설계하며, 관리하는지가 전체 시스템의 성능, 안정성, 확장성을 결정짓는다고 해도 과언이 아니다.

이 안내서는 ROS 2 Humble Hawksbill 배포판에서 노드의 개념을 단순히 소개하는 것을 넘어선다. 커맨드 라인 인터페이스(CLI)를 이용한 기본적인 관리 방법부터 시작하여, C++(`rclcpp`)와 Python(`rclpy`)을 사용한 구체적인 프로그래밍 기법, 그리고 대규모 시스템의 성능과 안정성을 보장하는 컴포지션(Composition)과 라이프사이클(Lifecycle) 같은 고급 아키텍처 패턴까지 깊이 있게 파고든다. 이 안내서의 목표는 개발자가 실무에서 마주할 수 있는 모든 시나리오에 효과적으로 대응할 수 있는 포괄적이고 전문적인 지식을 제공하는 것이다.4

## 1. 부: ROS 2 노드의 본질

### 1.1  ROS 그래프와 노드의 역할

노드를 이해하기 위해서는 먼저 노드가 활동하는 무대인 **ROS 그래프(ROS Graph)** 를 알아야 한다.

#### 1.1.1 정의와 역할

노드는 ROS 2 그래프에 참여하는 하나의 개체(participant)이며, 일반적으로 하나의 논리적인 작업(a single, modular purpose)을 수행하는 최소 계산 단위로 정의된다.1 예를 들어, 로봇 시스템은 카메라 드라이버 노드, 라이다 데이터 처리 노드, 모터 제어 노드, 경로 계획 노드 등으로 구성될 수 있다. 각 노드는 독립적으로 개발, 테스트, 실행될 수 있어 전체 시스템의 모듈성을 크게 향상시킨다.6

ROS 그래프는 이러한 모든 노드들과 그들 간의 통신 연결(토픽, 서비스 등)을 시각적으로 표현한 네트워크다.2 이 그래프는 정적인 다이어그램이 아니라, 노드가 실행되고 종료됨에 따라 실시간으로 변화하는 동적인 네트워크다. `rqt_graph`는 이 동적 네트워크의 구조를 시각화하여 노드 간의 데이터 흐름을 직관적으로 파악하게 해주는 필수적인 디버깅 도구다.3

#### 1.1.2 분산 디스커버리와 `ros2 daemon`

ROS 1이 중앙의 `roscore`(마스터)를 통해 모든 노드 간의 연결을 관리했던 것과 달리, ROS 2는 **분산 디스커버리 프로세스(distributed discovery process)** 를 채택했다.1 이는 중앙 집중적인 관리자 없이 각 노드가 네트워크상의 다른 노드들을 스스로 발견하고 연결을 설정하는 방식이다. 이 설계는 마스터 노드가 다운되면 전체 시스템이 마비되는 ROS 1의 단일 실패점(Single Point of Failure) 문제를 해결하여 시스템의 견고성을 크게 높였다.

이러한 분산 시스템 철학은 강력하지만, 한 가지 트레이드오프가 존재한다. 새로운 노드가 전체 그래프의 정보를 파악하는 데 시간이 걸릴 수 있다는 점이다. 이로 인해 `ros2 node list`와 같은 명령어가 즉각적으로 반응하지 않을 수 있다. ROS 2는 이 문제를 해결하기 위해 **`ros2 daemon`** 이라는 백그라운드 프로세스를 도입했다.7 데몬은 사용자가 `ros2` 명령어를 처음 실행할 때 자동으로 시작되며, ROS 그래프에 대한 정보를 지속적으로 수집하고 캐싱한다. 덕분에 사용자는 분산 시스템의 견고성을 유지하면서도 CLI를 통해 시스템 정보를 신속하게 조회할 수 있다. 즉, 노드는 단순히 독립적인 프로그램이 아니라, ROS 2의 분산적이고 견고한 시스템 철학을 구현하는 핵심 추상화(abstraction)이며, `ros2 daemon`은 이러한 철학을 실용적으로 만들기 위한 보완 장치인 셈이다.

### 1.2  노드 통신 패러다임

노드는 고립된 개체가 아니며, 다른 노드와 끊임없이 상호작용한다. 이를 위해 노드는 다양한 통신 인터페이스를 소유하는 일종의 '컨테이너' 역할을 한다.1

`ros2 node info` 명령어는 특정 노드가 소유한 모든 통신 인터페이스(퍼블리셔, 서브스크라이버, 서비스 서버/클라이언트, 액션 서버/클라이언트 등)를 정확히 보여주어 이 개념을 명확히 한다.2 ROS 2는 크게 네 가지 통신 방식을 제공한다.

- **토픽 (Topics):** 비동기식, 다대다(N:M) 퍼블리시/서브스크라이브(Publish/Subscribe) 모델이다. 어떤 노드(퍼블리셔)가 특정 이름의 '토픽'에 데이터를 계속해서 발행하면, 해당 토픽에 관심 있는 다른 모든 노드(서브스크라이버)들이 그 데이터를 수신한다. 데이터의 흐름이 단방향이며, 주로 센서 데이터나 로봇의 상태처럼 지속적으로 발생하는 데이터 스트림을 전달하는 데 적합하다.3
- **서비스 (Services):** 동기 또는 비동기식, 일대일(1:1) 요청/응답(Request/Response) 모델이다. 클라이언트 노드가 특정 작업을 서비스 서버 노드에 요청하면, 서버는 그 요청을 처리한 후 단일 응답을 반환한다. 이는 특정 계산을 요청하고 그 결과를 즉시 받아야 하는 빠른 원격 프로시저 호출(RPC)에 유용하다. 동기식으로 호출할 경우, 작업이 완료될 때까지 클라이언트는 다음 코드로 진행하지 않고 기다린다(blocking).1
- **액션 (Actions):** 비동기식, 일대일(1:1) 목표/피드백/결과(Goal/Feedback/Result) 모델이다. 로봇 팔을 특정 위치로 이동시키거나, 지정된 경로를 따라 내비게이션하는 것처럼 실행에 시간이 오래 걸리는 작업을 위해 설계되었다. 서비스와 가장 큰 차이점은 두 가지다. 첫째, 작업이 진행되는 동안 주기적인 **피드백**을 클라이언트에게 보낼 수 있다. 둘째, 클라이언트는 진행 중인 작업을 언제든지 **취소(preemption)** 할 수 있다.9
- **파라미터 (Parameters):** 노드의 설정을 위한 키-값(key-value) 저장소다. 각 노드는 자신만의 파라미터 집합을 유지하며, 이 값들은 런타임에 동적으로 조회하거나 변경할 수 있다. 예를 들어, 로봇의 최고 속도, 카메라의 해상도 등을 파라미터로 관리할 수 있다. 흥미롭게도 파라미터 시스템 자체는 ROS 2 서비스 위에 구현되어 있어, 외부에서 서비스 호출을 통해 파라미터를 제어할 수 있다. 이는 노드의 동작을 코드 수정 없이 유연하게 조정할 수 있게 해주는 강력한 메커니즘이다.1

개발자는 특정 통신 요구사항에 가장 적합한 도구를 선택해야 하며, 이는 효율적이고 올바른 시스템 설계를 위한 첫걸음이다. 아래 표는 각 통신 방식의 특징과 사용 사례를 요약한 것이다.

**Table 1: ROS 2 통신 방식 비교**

| 구분               | 토픽 (Topics)                                                | 서비스 (Services)                                            | 액션 (Actions)                                               |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **통신 모델**      | 퍼블리시/서브스크라이브 (N:M)                                | 요청/응답 (1:1)                                              | 목표/피드백/결과 (1:1)                                       |
| **동기성**         | 비동기 (Asynchronous)                                        | 동기/비동기 (Sync/Async)                                     | 비동기 (Asynchronous)                                        |
| **주요 특징**      | - 지속적인 데이터 스트림 - 단방향 통신 - 송신자와 수신자 간의 시간적 결합 없음 | - 빠른 원격 프로시저 호출(RPC) - 요청-응답이 한 쌍으로 이루어짐 - 작업 완료 시 단일 결과 반환 | - 장시간 실행되는 작업 - 주기적인 피드백 제공 - 작업 취소(Preemption) 가능 |
| **주요 사용 사례** | - 센서 데이터 (카메라, 라이다) - 로봇 상태 (위치, 속도) - 연속적인 제어 명령 | - 현재 로봇의 설정값 요청 - 특정 좌표에 대한 역기구학(IK) 계산 - 간단한 상태 변경 요청 | - 내비게이션 목표 지점까지 이동 - 로봇 팔로 물체 집기 - 복잡한 데이터 처리 작업 |

Source: 9

## 2. 부: 노드 관리를 위한 핵심 도구: `ros2` CLI

ROS 2는 커맨드 라인 인터페이스(CLI)를 통해 노드를 관리하고 시스템을 분석할 수 있는 강력한 도구 모음을 제공한다.7 `ros2`라는 메인 명령어를 시작으로 다양한 하위 명령어를 사용할 수 있다.

### 2.1  노드 실행 및 탐색

- `ros2 run <package_name> <executable_name>`: 패키지에 포함된 노드 실행 파일을 실행하는 가장 기본적인 명령어다. 이 명령어는 단순히 프로세스를 시작하는 것을 넘어, 해당 노드를 ROS 2 네트워크에 참여시켜 다른 노드들과 통신할 수 있게 만든다. 예를 들어, `ros2 run turtlesim turtlesim_node`는 `turtlesim` 패키지의 `turtlesim_node`를 실행하여 거북이 시뮬레이션 창을 띄운다.2
- `ros2 node list`: 현재 ROS 2 그래프에서 실행 중인 모든 노드의 이름을 나열한다. 복잡한 시스템에서 어떤 노드들이 활성화되어 있는지 한눈에 파악하는 데 필수적이다. 이 명령어는 백그라운드의 `ros2 daemon` 덕분에 신속하게 결과를 반환한다.2
- `ros2 node info <node_name>`: 특정 노드에 대한 상세 정보를 출력한다. 이 정보에는 해당 노드가 발행(Publishers)하는 토픽, 구독(Subscribers)하는 토픽, 제공하는 서비스(Service Servers) 및 사용하는 서비스(Service Clients), 그리고 액션(Action Servers/Clients) 목록이 모두 포함된다. 노드 간의 연결 관계를 파악하고 문제를 진단하는 디버깅의 첫걸음과도 같은 명령어다.2

### 2.2  노드 속성 재정의: 리매핑(Remapping)과 파라미터 로딩

`ros2 run` 명령어는 단순히 노드를 실행하는 것 이상의 기능을 제공한다. `--ros-args` 플래그를 사용하면 노드의 동작을 런타임에 유연하게 변경할 수 있다.

- **리매핑 (Remapping):** `--remap` 옵션은 노드가 사용하는 이름(노드 이름, 토픽 이름, 서비스 이름 등)을 실행 시점에 다른 이름으로 변경하는 기능이다. 예를 들어, `ros2 run turtlesim turtlesim_node --ros-args --remap __node:=my_turtle` 명령은 `turtlesim_node`의 기본 이름인 `/turtlesim`을 `/my_turtle`로 변경하여 실행한다. 이를 통해 동일한 노드 코드를 재사용하여 여러 개의 인스턴스를 서로 다른 이름으로 실행할 수 있다.2
- **파라미터 로딩 (Parameter Loading):** `--params-file` 옵션은 YAML 파일에 정의된 파라미터들을 노드 시작 시 한 번에 로드하는 기능이다. 예를 들어, `ros2 run turtlesim turtlesim_node --ros-args --params-file./turtlesim.yaml` 명령은 `turtlesim.yaml` 파일에 저장된 설정값으로 노드를 초기화한다. 이는 노드의 설정을 소스 코드와 완전히 분리하여 관리할 수 있게 해주므로, 코드 재사용성과 설정의 유연성을 극대화한다.15

`--ros-args` 플래그는 이처럼 리매핑, 파라미터 로딩, 로그 레벨 설정 등 ROS 2 시스템에 특화된 '메타 설정'을 주입하는 표준화된 게이트웨이 역할을 한다. ROS 2 클라이언트 라이브러리(RCL)는 사용자가 작성한 `main` 함수가 실행되기 전에 이 인자들을 먼저 파싱하여 노드의 내부 상태를 설정한다. 이는 개발자가 작성한 비즈니스 로직과 ROS 프레임워크의 설정을 명확하게 분리하는 중요한 경계선 역할을 하며, 어떤 노드든 일관된 방식으로 런타임 설정을 주입할 수 있게 해준다. 50에서 사용자가 이 개념을 혼동하여 겪는 문제는 이 분리의 중요성을 잘 보여준다.

### 2.3  다중 노드 관리: `ros2 launch`

실제 로봇 시스템은 수십, 수백 개의 노드로 구성될 수 있다. 이 많은 노드들을 터미널에서 `ros2 run`으로 하나씩 실행하고 각각의 리매핑과 파라미터를 수동으로 입력하는 것은 비효율적일 뿐만 아니라 오류 발생 가능성이 매우 높다.17

`ros2 launch`는 이러한 문제를 해결하는 강력한 도구다. Python, XML, 또는 YAML 형식으로 작성된 **런치 파일(launch file)** 을 사용하여 여러 노드와 그에 필요한 모든 설정(노드 이름, 네임스페이스, 파라미터, 리매핑 등)을 한 번의 명령으로 실행, 관리, 모니터링할 수 있다. 런치 시스템은 단순히 노드를 실행하는 것을 넘어, 노드 간의 실행 순서를 제어하거나 특정 조건에 따라 노드를 실행하는 등 복잡한 시스템 구동 로직을 기술할 수 있다.17

### 2.4  노드 인터페이스와 상호작용

`ros2` CLI는 실행 중인 노드의 통신 인터페이스와 직접 상호작용할 수 있는 다양한 하위 명령어를 제공한다. 이 명령어들은 시스템을 디버깅하고 테스트하는 데 매우 유용하다.

- `ros2 topic`: 토픽과 관련된 모든 작업을 수행한다.
  - `pub`: 특정 토픽에 메시지를 한 번 또는 주기적으로 발행한다.
  - `echo`: 특정 토픽의 데이터를 실시간으로 화면에 출력한다.
  - `hz`/`bw`: 특정 토픽의 메시지 발행 주기(Hz)와 대역폭(bandwidth)을 측정한다.
  - `find`: 특정 메시지 타입을 사용하는 토픽을 검색한다.
  - Source: 3
- `ros2 service`: 서비스와 관련된 작업을 수행한다.
  - `call`: 특정 서비스를 인자와 함께 호출하고 응답을 받는다.
  - `type`: 특정 서비스의 타입을 확인한다.
  - `find`: 특정 서비스 타입을 사용하는 서비스를 검색한다.
  - Source: 10
- `ros2 action`: 액션과 관련된 작업을 수행한다.
  - `send_goal`: 특정 액션 서버에 목표를 전송하고 피드백과 결과를 수신한다.
  - `info`: 특정 액션의 상세 정보를 확인한다.
  - Source: 12
- `ros2 param`: 노드의 파라미터를 동적으로 관리한다.
  - `get`/`set`: 특정 노드의 파라미터 값을 조회하거나 변경한다.
  - `list`: 특정 노드의 모든 파라미터 목록을 확인한다.
  - `dump`: 노드의 현재 파라미터 상태를 YAML 파일로 저장한다.
  - Source: 14

## 3. 부: C++를 이용한 노드 프로그래밍 (`rclcpp`)

`rclcpp`는 C++로 ROS 2 노드를 작성하기 위한 클라이언트 라이브러리다. 모던 C++의 기능을 적극적으로 활용하여 높은 성능과 안정성을 제공한다.

### 3.1  개발 환경 설정

C++ 노드를 개발하기 위해서는 `ament_cmake` 빌드 시스템을 사용해야 한다.

1. **패키지 생성:** `ros2 pkg create --build-type ament_cmake --license <LICENSE> <package_name> --dependencies rclcpp std_msgs...` 명령어를 사용하여 패키지를 생성한다. `--dependencies` 플래그는 필요한 의존성을 `package.xml`과 `CMakeLists.txt`에 자동으로 추가해준다.19
2. **`package.xml` 설정:** 이 파일은 패키지의 이름, 버전, 설명, 라이선스 등 메타 정보와 빌드 및 실행 시 필요한 의존성 패키지 목록을 정의한다. `ament_cmake` 빌드 시스템은 이 파일을 참조하여 패키지 간의 의존성 관계를 파악한다.20
3. **`CMakeLists.txt` 설정:** 이 파일은 C++ 소스 코드를 컴파일하고 링크하여 실행 파일을 만드는 방법을 정의하는 빌드 스크립트다. 주요 단계는 다음과 같다 20:
   - `find_package(rclcpp REQUIRED)`: `rclcpp`와 같은 의존성 패키지를 찾는다.
   - `add_executable(node_name src/source_file.cpp)`: 소스 파일을 컴파일하여 실행 파일을 생성한다.
   - `ament_target_dependencies(node_name rclcpp std_msgs)`: 생성된 실행 파일이 의존하는 라이브러리들을 링크한다.
   - `install(TARGETS node_name DESTINATION lib/${PROJECT_NAME})`: 빌드가 완료된 후, 생성된 실행 파일을 `install/package_name/lib` 디렉토리에 복사하여 `ros2 run` 명령어가 찾을 수 있도록 한다.

### 3.2  기본 퍼블리셔/서브스크라이버 노드 작성

다음은 "Hello World" 메시지를 주기적으로 발행하는 퍼블리셔 노드의 전체 코드 예시다.

```C++
// publisher_member_function.cpp
#include <chrono>
#include <functional>
#include <memory>
#include <string>

#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"

using namespace std::chrono_literals;

/* rclcpp::Node를 상속받는 MinimalPublisher 클래스 정의 */
class MinimalPublisher : public rclcpp::Node
{
public:
  MinimalPublisher()
  : Node("minimal_publisher"), count_(0)
  {
    // 'topic'이라는 이름의 토픽에 String 메시지를 발행하는 퍼블리셔 생성. QoS 큐 사이즈는 10.
    publisher_ = this->create_publisher<std_msgs::msg::String>("topic", 10);
    // 500ms(0.5초) 주기로 timer_callback 함수를 호출하는 타이머 생성.
    timer_ = this->create_wall_timer(
      500ms, std::bind(&MinimalPublisher::timer_callback, this));
  }

private:
  void timer_callback()
  {
    // 발행할 메시지 객체 생성
    auto message = std_msgs::msg::String();
    message.data = "Hello, world! " + std::to_string(count_++);
    // RCLCPP_INFO 매크로를 사용하여 콘솔에 로그 출력
    RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
    // 메시지 발행
    publisher_->publish(message);
  }
  // 멤버 변수 선언
  rclcpp::TimerBase::SharedPtr timer_;
  rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
  size_t count_;
};

int main(int argc, char * argv)
{
  // ROS 2 초기화
  rclcpp::init(argc, argv);
  // MinimalPublisher 노드의 인스턴스를 생성하고, spin을 통해 콜백 처리 시작
  rclcpp::spin(std::make_shared<MinimalPublisher>());
  // ROS 2 종료
  rclcpp::shutdown();
  return 0;
}
```

Source: 21

**코드 해설:**

- **클래스 구조:** 노드는 `rclcpp::Node`를 상속받는 클래스로 구현된다. 노드의 모든 기능(퍼블리셔, 타이머 등)은 클래스의 멤버 변수로 관리된다.
- **생성자:** 노드 이름(`minimal_publisher`)을 부모 생성자에 전달하여 초기화한다. `create_publisher()`를 호출하여 퍼블리셔를, `create_wall_timer()`를 호출하여 타이머를 생성한다. `std::bind`는 멤버 함수인 `timer_callback`을 타이머에 콜백으로 등록하기 위해 사용된다.
- **`timer_callback()`:** 타이머에 의해 주기적으로 호출되는 함수다. 이 함수 안에서 `std_msgs::msg::String` 타입의 메시지를 생성하고, 데이터를 채운 뒤, `publisher_->publish()`를 통해 메시지를 발행한다.
- **`main()` 함수:** 프로그램의 진입점이다. `rclcpp::init()`으로 ROS 2 시스템을 초기화하고, `rclcpp::spin()`을 호출하여 노드를 활성화하고 콜백 함수들이 호출될 수 있도록 이벤트 루프를 시작한다. `spin` 함수는 `Ctrl+C` 등으로 종료 신호를 받을 때까지 블로킹된다. `rclcpp::shutdown()`은 `spin`이 종료된 후 모든 ROS 2 자원을 정리한다.

서브스크라이버 노드는 유사한 구조를 가지며, `create_subscription()`을 사용하고 메시지를 수신했을 때 호출될 콜백 함수를 정의하는 점이 다르다.21

### 3.3  요청-응답 통신: 서비스 서버/클라이언트

다음은 두 정수를 더하는 서비스 서버의 핵심 코드다.

```C++
// add_two_ints_server.cpp
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/srv/add_two_ints.hpp"
#include <memory>

// 서비스 콜백 함수
void add(const std::shared_ptr<example_interfaces::srv::AddTwoInts::Request> request,
          std::shared_ptr<example_interfaces::srv::AddTwoInts::Response> response)
{
  response->sum = request->a + request->b;
  RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Incoming request\na: %ld, b: %ld",
                request->a, request->b);
  RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "sending back response: [%ld]", (long int)response->sum);
}

int main(int argc, char **argv)
{
  rclcpp::init(argc, argv);
  // 'add_two_ints_server' 이름의 노드 생성
  std::shared_ptr<rclcpp::Node> node = rclcpp::Node::make_shared("add_two_ints_server");
  // 'add_two_ints' 이름의 서비스를 생성하고 'add' 함수를 콜백으로 등록
  rclcpp::Service<example_interfaces::srv::AddTwoInts>::SharedPtr service =
    node->create_service<example_interfaces::srv::AddTwoInts>("add_two_ints", &add);
  RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Ready to add two ints.");
  // 노드를 스핀하여 서비스 요청을 기다림
  rclcpp::spin(node);
  rclcpp::shutdown();
}
```

Source: 20

클라이언트 노드는 `create_client()`로 클라이언트를 생성하고, `async_send_request()`로 비동기 요청을 보낸다. 반환된 `std::future`를 `rclcpp::spin_until_future_complete()`로 기다려 결과를 얻는다.20

### 3.4  장기 실행 작업: 액션 서버/클라이언트

액션 서버는 목표, 취소, 수락에 대한 콜백을 각각 등록해야 한다. 특히 장기 실행 작업은 메인 스레드를 블로킹하지 않도록 별도의 스레드에서 처리하는 것이 중요하다.

```C++
// fibonacci_action_server.cpp (핵심 부분)
void handle_accepted(const std::shared_ptr<GoalHandleFibonacci> goal_handle)
{
  using namespace std::placeholders;
  // 실행 함수가 메인 스레드를 막지 않도록 새 스레드에서 실행
  std::thread{std::bind(&FibonacciActionServer::execute, this, _1), goal_handle}.detach();
}

void execute(const std::shared_ptr<GoalHandleFibonacci> goal_handle)
{
  RCLCPP_INFO(this->get_logger(), "Executing goal");
  rclcpp::Rate loop_rate(1);
  const auto goal = goal_handle->get_goal();
  auto feedback = std::make_shared<Fibonacci::Feedback>();
  //... (피보나치 수열 계산 로직)...

  for (int i = 1; (i < goal->order) && rclcpp::ok(); ++i) {
    // 취소 요청이 있는지 확인
    if (goal_handle->is_canceling()) {
      //... (취소 처리)...
      return;
    }
    //... (수열 업데이트)...
    // 피드백 발행
    goal_handle->publish_feedback(feedback);
    loop_rate.sleep();
  }

  // 목표 성공 시 결과 전송
  if (rclcpp::ok()) {
    auto result = std::make_shared<Fibonacci::Result>();
    result->sequence = feedback->partial_sequence;
    goal_handle->succeed(result);
    RCLCPP_INFO(this->get_logger(), "Goal succeeded");
  }
}
```

Source: 23

액션 클라이언트는 `create_client()`로 클라이언트를 생성하고, `async_send_goal()`을 호출할 때 목표 응답, 피드백, 최종 결과에 대한 콜백 함수들을 옵션으로 함께 전달한다.23

`rclcpp` API 전반에 걸쳐 `std::shared_ptr`와 같은 스마트 포인터와 `std::bind`, 람다(lambda) 같은 모던 C++ 기능이 적극적으로 사용되는 것을 볼 수 있다.20 이는 단순히 코드를 간결하게 만드는 것을 넘어, 메모리 누수를 방지하고 객체의 소유권을 명확히 하며, 특히 컴포지션 환경에서 Zero-copy 통신을 가능하게 하는 등 자원 관리를 강제하고 비동기 처리를 안전하게 만들기 위한 의도적인 설계 철학이다. 따라서 `rclcpp`를 효과적으로 사용하기 위해서는 C++11/14 이상의 모던 C++ 패러다임에 대한 이해가 필수적이다.

## 4. 부: Python을 이용한 노드 프로그래밍 (`rclpy`)

`rclpy`는 Python으로 ROS 2 노드를 작성하기 위한 클라이언트 라이브러리다. C++에 비해 개발 속도가 빠르고 문법이 간결하여 프로토타이핑과 상위 레벨 로직 구현에 널리 사용된다.

### 4.1  개발 환경 설정

Python 노드를 개발하기 위해서는 `ament_python` 빌드 타입을 사용한다.

1. **패키지 생성:** `ros2 pkg create --build-type ament_python --license <LICENSE> <package_name> --dependencies rclpy std_msgs...` 명령어로 패키지를 생성한다.25
2. **`package.xml` 설정:** C++ 패키지와 동일하게 메타 정보와 의존성을 정의한다. `rclpy` 의존성을 명시해야 한다.26
3. **`setup.py` 설정:** 이 파일은 Python 패키징 표준(setuptools)에 따라 작성되며, `ament_python` 빌드 시스템의 핵심이다. 가장 중요한 부분은 `entry_points` 딕셔너리다. 여기에 `'node_name = package_name.file_name:main'` 형식으로 실행 가능한 노드를 등록해야 `ros2 run` 명령어가 해당 노드를 찾고 실행할 수 있다.26
4. **`setup.cfg` 설정:** 이 파일은 `setup.py`에서 정의한 스크립트가 설치될 경로를 지정한다. 보통 `install/package_name/lib/package_name` 경로에 설치되도록 설정한다.29

### 4.2  기본 퍼블리셔/서브스크라이버 노드 작성

다음은 Python으로 작성된 퍼블리셔 노드의 전체 코드 예시다. C++ 버전과 구조적으로 매우 유사함을 알 수 있다.

```Python
# publisher_member_function.py
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MinimalPublisher(Node):
    def __init__(self):
        super().__init__('minimal_publisher')
        # 'topic' 이름의 토픽에 String 메시지를 발행하는 퍼블리셔 생성. QoS 큐 사이즈는 10.
        self.publisher_ = self.create_publisher(String, 'topic', 10)
        timer_period = 0.5  # seconds
        # 0.5초 주기로 timer_callback 함수를 호출하는 타이머 생성.
        self.timer = self.create_timer(timer_period, self.timer_callback)
        self.i = 0

    def timer_callback(self):
        msg = String()
        msg.data = f'Hello World: {self.i}'
        self.publisher_.publish(msg)
        self.get_logger().info(f'Publishing: "{msg.data}"')
        self.i += 1

def main(args=None):
    # rclpy 라이브러리 초기화
    rclpy.init(args=args)
    minimal_publisher = MinimalPublisher()
    # 노드를 스핀하여 콜백 처리 시작
    rclpy.spin(minimal_publisher)
    # 노드 명시적 파괴 (선택 사항)
    minimal_publisher.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

Source: 26

**코드 해설:**

- **클래스 구조:** `rclpy.node.Node` 클래스를 상속받아 노드를 정의한다.
- **생성자:** `super().__init__()`을 호출하여 부모 클래스를 초기화하고 노드 이름을 전달한다. `create_publisher()`와 `create_timer()`를 사용하여 퍼블리셔와 타이머를 생성한다.
- **`timer_callback()`:** 타이머에 의해 주기적으로 호출되는 콜백 함수다. 메시지를 생성하고 `publish()` 메서드로 발행한다.
- **`main()` 함수:** C++과 동일하게 `rclpy.init()`으로 초기화하고, 노드 인스턴스를 생성한 뒤, `rclpy.spin()`을 호출하여 이벤트 루프를 시작한다.

### 4.3  요청-응답 통신: 서비스 서버/클라이언트

Python 서비스 서버는 `create_service()`를 사용하여 서비스를 생성하고, 요청을 처리하는 콜백 함수를 등록한다. 콜백 함수는 `request`와 `response` 객체를 인자로 받아, 요청 데이터를 읽고 `response` 객체에 결과값을 채워 반환한다.28

클라이언트는 `create_client()`로 클라이언트를 생성하고, `call_async()` 메서드를 사용하여 비동기적으로 서비스를 호출한다. 이 메서드는 `future` 객체를 반환하며, 이 `future`가 완료될 때까지 기다렸다가 결과를 얻는다.28

### 4.4  장기 실행 작업: 액션 서버/클라이언트

Python 액션 서버는 `ActionServer` 클래스를 사용하여 구현한다. 생성자에서 액션 타입, 액션 이름, 그리고 목표를 실행할 `execute_callback` 함수를 등록한다. `execute_callback`은 장기 실행 작업이므로, `async`와 `await`를 사용하거나 별도의 스레드를 활용하여 비동기적으로 구현해야 한다.32 콜백 함수 내에서 `goal_handle.publish_feedback()`으로 피드백을 보내고, 작업 완료 시 `goal_handle.succeed()`를 호출하며 결과 객체를 반환한다.32

클라이언트는 `ActionClient` 클래스를 사용한다. `send_goal_async()`를 통해 목표를 비동기적으로 전송하며, 피드백을 받기 위한 콜백 함수를 등록할 수 있다. 반환된 `goal_handle` 객체를 통해 최종 결과를 비동기적으로 얻을 수 있다.32

## 5. 부: 고급 노드 아키텍처

단순한 노드 작성법을 넘어, 실제 로봇 시스템의 성능과 안정성을 극대화하기 위해서는 ROS 2가 제공하는 고급 아키텍처 패턴인 **컴포지션(Composition)** 과 **라이프사이클(Lifecycle)** 을 이해하고 적용해야 한다.

### 5.1  노드 컴포지션: 성능 최적화 기법

#### 5.1.1 개념 및 배경

ROS 1에서는 노드를 만드는 두 가지 방식, 즉 독립적인 실행 파일인 '노드'와 컨테이너 프로세스에 로드되는 공유 라이브러리인 '노드렛'이 존재했다. 노드렛은 프로세스 간 통신 오버헤드를 줄여 성능을 높일 수 있었지만, 노드와 API가 달라 개발과 배포의 유연성을 저해하는 문제가 있었다.35

ROS 2는 이 문제를 해결하기 위해 모든 노드를 재사용 가능한 **컴포넌트(Component)** 라는 형태로 작성하도록 권장한다. 컴포넌트는 `main` 함수가 없는 공유 라이브러리 형태로 빌드된다. 이렇게 만들어진 컴포넌트들은 배포 시점에 개발자의 선택에 따라 여러 개를 하나의 프로세스에 묶어서 실행하거나(composed), 각각을 별도의 프로세스로 실행할 수 있다. ROS 2는 이를 **"배포 시점 결정(deploy-time decision)"** 이라고 부르며, 이는 개발의 유연성과 배포의 효율성을 동시에 달성하는 핵심적인 개념이다.35

#### 5.1.2 장점: IPC와 Zero-Copy의 원리

컴포지션의 가장 큰 장점은 압도적인 통신 성능 향상에 있다. 이는 **프로세스 내 통신(Intra-Process Communication, IPC)** 과 **Zero-Copy** 메커니즘을 통해 달성된다.

- **일반적인 프로세스 간 통신:** 서로 다른 프로세스에서 실행되는 두 노드가 통신할 때는 다음과 같은 과정을 거친다.

  1. 송신 노드가 메시지 데이터를 메모리상의 연속된 버퍼로 변환한다 (직렬화, Serialization).

  2. 이 버퍼를 운영체제의 네트워크 스택을 통해 다른 프로세스로 전달한다 (이 과정에서 여러 번의 메모리 복사가 발생).

  3. 수신 노드는 전달받은 버퍼를 다시 원래의 메시지 데이터 구조로 복원한다 (역직렬화, Deserialization).

     이 과정, 특히 대용량 데이터(카메라 이미지, 포인트 클라우드 등)를 다룰 때 발생하는 메모리 복사와 직렬화/역직렬화는 상당한 CPU 오버헤드와 지연 시간(latency)을 유발한다.38

- **IPC / Zero-Copy 통신:** 반면, 하나의 프로세스 내에 함께 로드된 컴포넌트들은 이 모든 과정을 생략할 수 있다.

  1. 송신 컴포넌트(퍼블리셔)가 `publish()` 메서드를 호출할 때, 메시지 데이터를 직렬화하는 대신 데이터가 담긴 메모리의 주소(포인터)를 전달한다.

  2. 수신 컴포넌트(서브스크라이버)는 이 포인터를 직접 받아 데이터에 접근한다.

     이 방식에서는 데이터의 복사가 전혀 발생하지 않으므로 'Zero-Copy' 라고 부른다. 이는 통신 오버헤드를 극적으로 줄여, 이미지나 포인트 클라우드 같은 대용량 데이터를 거의 지연 없이 주고받을 수 있게 한다.38

     `rclcpp` API가 메시지를 전달할 때 `std::unique_ptr`나 `std::shared_ptr` 같은 스마트 포인터를 사용하는 이유가 바로 이 Zero-Copy 통신을 가능하게 하기 위함이다.

이러한 성능 향상은 단순한 최적화를 넘어, ROS 2가 자율주행이나 고정밀 조작과 같이 고대역폭 센서 데이터를 실시간으로 처리해야 하는 차세대 로봇 애플리케이션의 플랫폼이 될 수 있게 하는 핵심 기술이다. 따라서 복잡한 센서 데이터 처리 파이프라인을 구축할 때 컴포지션을 사용하지 않는 것은 ROS 2의 잠재력을 완전히 활용하지 못하는 것과 같다.

#### 5.1.3 구현 방법

컴포지션을 구현하는 방법은 크게 세 가지다.

1. **런타임 컴포지션 (Runtime Composition):** `ros2 run rclcpp_components component_container` 명령으로 범용 컨테이너 프로세스를 실행한 뒤, `ros2 component load` 명령어를 사용하여 컴포넌트(공유 라이브러리)를 필요에 따라 동적으로 로드하거나 언로드하는 방식이다. 가장 유연하다.37
2. **컴파일타임 컴포지션 (Compile-time Composition):** `main` 함수에서 직접 여러 컴포넌트 클래스의 인스턴스를 생성하고, 이를 하나의 실행 파일로 컴파일하는 방식이다. 실행할 컴포넌트가 컴파일 시점에 고정된다.37
3. **런치 파일을 이용한 컴포지션 (Launch-based Composition):** Python 런치 파일 내에서 `launch_ros.actions.ComposableNodeContainer`와 `launch_ros.descriptions.ComposableNode`를 사용하여 런치 타임에 컨테이너와 그 안에서 실행될 컴포넌트들을 선언적으로 정의하는 방식이다. 유연성과 자동화를 모두 잡을 수 있어 가장 널리 사용된다.42

### 5.2  라이프사이클 노드: 시스템 견고성 확보

#### 5.2.1 개념 및 상태 머신

일반적인 ROS 노드는 실행되는 즉시 활성화(active) 상태가 되어 작업을 시작한다. 하지만 복잡한 로봇 시스템에서는 모든 노드가 준비되기 전에 특정 노드가 먼저 동작하는 것을 원치 않을 수 있다. **라이프사이클 노드(Lifecycle Node)** 는 이러한 문제를 해결하기 위해 외부에서 제어 가능한 **상태 머신(State Machine)** 을 내장한 특수한 노드다.6

라이프사이클 노드는 다음과 같은 4개의 주요 상태(Primary States)를 가진다 44:

- **`Unconfigured` (미설정):** 노드가 생성되었지만, 아직 아무런 설정이나 자원 할당이 이루어지지 않은 초기 상태.
- **`Inactive` (비활성):** 설정이 완료되고 자원도 할당되었지만, 아직 데이터 처리나 통신은 하지 않는 준비 상태. 이 상태에서 파라미터 변경 등 재설정이 가능하다.
- **`Active` (활성):** 모든 기능이 정상적으로 동작하며, 데이터를 처리하고 통신하는 주된 작동 상태.
- **`Finalized` (종료됨):** 노드가 종료되기 전 모든 자원을 해제한 상태. 이 상태에서는 파괴되는 것 외에 다른 상태로 전환할 수 없다.

이 상태들 사이의 전환(Transition)은 `configure`, `activate`, `deactivate`, `cleanup`, `shutdown`과 같은 외부 명령을 통해 이루어진다.43

#### 5.2.2 장점 및 사용 사례: `Nav2`

라이프사이클 노드의 진정한 가치는 여러 노드가 상호 의존적으로 동작하는 대규모 시스템에서 드러난다. 대표적인 예가 바로 ROS 2의 표준 내비게이션 스택인 **`Nav2`** 다.45

1. **결정론적 시스템 구동:** `Nav2` 스택은 컨트롤러, 플래너, 지도 서버 등 수많은 노드로 구성된다. 이 노드들은 `lifecycle_manager`라는 관리자 노드에 의해 정해진 순서대로 `configure`되고 `activate`된다. 예를 들어, 장애물 회피를 담당하는 `controller_server`가 완전히 활성화된 후에야 경로 계획 노드가 동작을 시작하도록 제어할 수 있다. 이는 시스템이 예측 가능하고 안전하게 시작되도록 보장한다.45
2. **견고한 오류 처리:** 만약 `controller_server`가 하드웨어 문제 등으로 `activate`에 실패하면, `lifecycle_manager`는 다른 내비게이션 관련 노드들의 활성화를 막는다. 이는 시스템이 불완전하고 위험한 상태로 동작하는 것을 원천적으로 방지하여 전체 시스템의 안정성과 신뢰성을 크게 높인다.45
3. **효율적인 자원 관리:** 로봇이 내비게이션을 사용하지 않을 때는 관련 노드들을 `Inactive` 상태로 전환하여 CPU와 메모리 자원을 절약할 수 있다. 필요할 때만 `Active` 상태로 전환하여 사용하므로, 특히 자원이 제한적인 임베디드 환경에서 유리하다.6
4. **온라인 업그레이드 및 재시작:** 시스템 전체를 중단하지 않고, 특정 노드(예: 경로 계획기)만 `deactivate`, `cleanup`한 후 새로운 버전의 컴포넌트로 교체하고 다시 `configure`, `activate`하는 것이 가능하다. 이는 시스템의 가용성을 높여준다.6

단일 노드의 상태 관리는 간단한 `if/else` 문으로도 가능하지만, 라이프사이클 노드는 개별 노드를 넘어 '시스템 전체의 상태'를 관리하고 제어하는, 한 차원 높은 수준의 아키텍처 도구다. `cascade_lifecycle` 패키지49와 같이 한 노드의 상태 전환이 의존하는 다른 노드의 상태 전환을 자동으로 유발하는 기능은 이러한 시스템 수준의 상태 동기화를 더욱 쉽게 만들어준다.

#### 5.2.3 구현 및 관리

- **구현:** C++에서는 `rclcpp_lifecycle::LifecycleNode`를, Python에서는 `rclpy.lifecycle.LifecycleNode`를 상속받아 노드를 작성한다. 그리고 `on_configure()`, `on_activate()` 등 각 전환에 해당하는 콜백 함수를 재정의(override)하여 해당 상태 전환 시 수행할 작업을 구현한다.43
- **관리:** `ros2 lifecycle` CLI를 사용하여 실행 중인 라이프사이클 노드의 현재 상태를 확인(`get`)하거나, 특정 상태로 전환하도록 명령(`set`)할 수 있다.43

## 6. 결론: 상황에 맞는 최적의 노드 설계 전략

지금까지 살펴본 것처럼, ROS 2의 노드는 단순한 실행 파일이 아니다. 개발자는 애플리케이션의 구체적인 요구사항, 즉 성능, 안정성, 복잡도, 재사용성 등을 종합적으로 고려하여 **표준 노드, 컴포저블 노드, 라이프사이클 노드**라는 세 가지 아키텍처 중 가장 적합한 것을 선택해야 하는 설계자의 역할을 수행해야 한다.

아래 표는 각 노드 유형의 장단점과 추천 사용 사례를 요약하여, 개발자가 실제 프로젝트에서 올바른 의사결정을 내리는 데 도움을 주는 프레임워크를 제공한다.

**Table 2: 노드 유형별 장단점 및 추천 사용 사례**

| 노드 유형                              | 장점                                                         | 단점                                                         | 추천 사용 사례                                               |
| -------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **표준 노드** (Standard Node)          | - 구현이 가장 간단하고 직관적 - 개별 노드의 디버깅이 용이 - 프로세스 격리로 인한 안정성 | - 프로세스 간 통신 오버헤드가 큼 - 대용량 데이터 처리 시 성능 저하 - 시스템 전체의 상태 관리가 어려움 | - 간단한 프로토타이핑 - GUI 도구나 Rviz 플러그인 - 시스템의 다른 부분과 독립적인 단일 기능 노드 |
| **컴포저블 노드** (Composable Node)    | - IPC/Zero-Copy를 통한 압도적인 통신 성능 - 낮은 지연 시간과 높은 처리량 - 단일 프로세스 실행으로 자원 효율성 증대 | - 모든 노드가 단일 프로세스에 있어 한 노드의 충돌이 전체에 영향 - 개별 노드 디버깅의 복잡성 증가 | - 카메라, 라이다 등 고대역폭 센서 데이터 처리 파이프라인 - 여러 알고리즘이 순차적으로 데이터를 처리하는 인식/제어 시스템 - 자원이 제한된 임베디드 시스템 |
| **라이프사이클 노드** (Lifecycle Node) | - 결정론적 시스템 시작/종료 보장 - 견고한 오류 처리 및 시스템 안정성 향상 - 자원 관리 및 온라인 업그레이드 용이 | - 상태 머신 관리로 인한 코드 복잡성 증가 - 모든 노드에 적용하기에는 과도할 수 있음 - 외부 관리자(e.g., lifecycle_manager) 필요 | - Nav2, MoveIt2와 같이 여러 노드가 상호 의존적인 대규모 시스템 - 안전이 중요한 시스템(Safety-critical systems) - 시스템의 특정 부분을 동적으로 활성화/비활성화해야 하는 경우 |

Source: 6

이 안내서에서 다룬 개념들을 바탕으로 실제 로봇이나 시뮬레이션 환경에 직접 적용하며 경험을 쌓는 것이 무엇보다 중요하다. 특히 `Nav2` 45, 

`MoveIt2` 47와 같은 실제 대규모 ROS 2 프로젝트의 소스 코드를 분석하며, 이들이 어떻게 노드 아키텍처를 구성하고 문제를 해결했는지 학습하는 것은 매우 유용한 실전 훈련이 될 것이다. ROS 2의 세계는 끊임없이 발전하고 있으므로, 공식 문서 4와 커뮤니티 포럼 4을 통해 최신 동향을 주시하는 자세 또한 훌륭한 ROS 2 개발자가 되기 위한 필수적인 덕목이다.

#### **참고 자료**

1. Nodes - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Concepts/Basic/About-Nodes.html
2. Understanding nodes - ROS 2 Documentation: Humble ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Nodes/Understanding-ROS2-Nodes.html
3. Understanding topics - ROS 2 Documentation: Humble ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html
4. ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/index.html
5. Tutorials - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials.html
6. Introduction to ROS 2 and Dynamic Composition: A Gateway to Advanced Robotics, accessed July 27, 2025, https://betterprogramming.pub/introduction-to-ros-2-and-dynamic-composition-a-gateway-to-advanced-robotics-782b459215ee
7. Introspection with command line tools - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Concepts/Basic/About-Command-Line-Tools.html
8. Understanding ROS 2 topics - ROS documentation, accessed July 27, 2025, https://docs.ros.org/en/dashing/Tutorials/Topics/Understanding-ROS2-Topics.html
9. Topics vs Services vs Actions - ROS 2 Documentation: Foxy ..., accessed July 27, 2025, https://docs.ros.org/en/foxy/How-To-Guides/Topics-Services-Actions.html
10. Understanding services - ROS 2 Documentation: Foxy documentation, accessed July 27, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Services/Understanding-ROS2-Services.html
11. Actions - ROS2 Design, accessed July 27, 2025, https://design.ros2.org/articles/actions.html
12. Understanding actions - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Actions/Understanding-ROS2-Actions.html
13. Understanding actions - ROS 2 Documentation: Foxy documentation, accessed July 27, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Actions/Understanding-ROS2-Actions.html
14. Understanding ROS 2 parameters - daobook 0.0.1 文档, accessed July 27, 2025, https://daobook.github.io/ros2-docs/foxy/Tutorials/Parameters/Understanding-ROS2-Parameters.html
15. Understanding parameters - ROS 2 Documentation: Humble ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Parameters/Understanding-ROS2-Parameters.html
16. Create a ROS2 Node with Python - ROS2 Humble Tutorial 6 - YouTube, accessed July 27, 2025, https://www.youtube.com/watch?v=EnkW5L-nZRw
17. Launching nodes - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Launching-Multiple-Nodes/Launching-Multiple-Nodes.html
18. Understanding services - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Services/Understanding-ROS2-Services.html
19. Writing a simple publisher and subscriber (C++) - ROS 2 ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Cpp-Publisher-And-Subscriber.html
20. Writing a simple service and client (C++) - ROS 2 Documentation ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Cpp-Service-And-Client.html
21. Robotics crash course (Part 8 - Writing a ROS 2 publisher and subscriber in C++), accessed July 27, 2025, https://jeremypedersen.com/posts/2024-08-15-pt8-ros2-cpp-pubsub/
22. ROS2 C++ Node for Beginners - Robotisim, accessed July 27, 2025, https://robotisim.com/ros2-cpp-node-for-begginers/
23. Writing an action server and client (C++) - ROS 2 Documentation ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Writing-an-Action-Server-Client/Cpp.html
24. ROS2 C++ Nodes Development in Depth - YouTube, accessed July 27, 2025, https://www.youtube.com/watch?v=cjckvOHo8B4
25. Creating your first node (Python) / User Manual - GitHub Pages, accessed July 27, 2025, https://turtlebot.github.io/turtlebot4-user-manual/tutorials/first_node_python.html
26. Writing a simple publisher and subscriber (Python) - ROS 2 Documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html
27. ROS2 Programming Basics Using Python | The Construct, accessed July 27, 2025, https://www.theconstruct.ai/ros2-programming-basics-using-python/
28. Writing a simple service and client (Python) - ROS 2 Documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Service-And-Client.html
29. Robotics crash course (Part 9 - Writing a ROS 2 publisher and subscriber in Python), accessed July 27, 2025, https://jeremypedersen.com/posts/2024-08-16-pt9-ros2-python-pubsub/
30. Ros 2 Publisher and Subscriber Python | by Poommin - Medium, accessed July 27, 2025, https://medium.com/@poommin2543/ros-2-publisher-and-subscriber-python-af299fc834a6
31. Writing a simple service and client (Python) - ROS 2 Documentation, accessed July 27, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Service-And-Client.html
32. Writing an action server and client (Python) - ROS 2 Documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Writing-an-Action-Server-Client/Py.html
33. ROS2 Actions [1H Crash Course] - YouTube, accessed July 27, 2025, https://www.youtube.com/watch?v=X7YSnDbKMWo
34. ROS2 Part 10 - ROS2 Actions in Python and C++ - RoboticsUnveiled, accessed July 27, 2025, https://www.roboticsunveiled.com/ros2-actions-in-python-and-cpp/
35. Composition - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Concepts/Intermediate/About-Composition.html
36. About Composition - ROS 2 Documentation: Xin 文档, accessed July 27, 2025, https://daobook.github.io/ros2-docs/xin/Concepts/About-Composition.html
37. Composing multiple nodes in a single process - ROS 2 Documentation, accessed July 27, 2025, https://docs.ros.org/en/crystal/Tutorials/Composition.html
38. Impact of ROS 2 Node Composition in Robotics | by makarand mandolkar | Medium, accessed July 27, 2025, https://medium.com/@mandolkarmakarand94/impact-of-ros-2-node-composition-in-robotics-8ad2295ea0fe
39. ROS 2 composition and ZED ROS 2 Wrapper - Stereolabs, accessed July 27, 2025, https://www.stereolabs.com/docs/ros2/ros2-composition
40. Zero-copy data transfer in ROS 2 - YouTube, accessed July 27, 2025, https://www.youtube.com/watch?v=gbSBRqLCVjE
41. Composing multiple nodes in a single process - ROS 2 Documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Composition.html
42. composition: Humble 0.20.5 documentation - the official ROS docs, accessed July 27, 2025, https://docs.ros.org/en/humble/p/composition/
43. lifecycle - lifecycle: Humble 0.20.5 documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/p/lifecycle/
44. Managed nodes - ROS2 Design, accessed July 27, 2025, https://design.ros2.org/articles/node_lifecycle.html
45. How to Use ROS 2 Lifecycle Nodes - Foxglove, accessed July 27, 2025, https://foxglove.dev/blog/how-to-use-ros2-lifecycle-nodes
46. What are LifeCycle Nodes and Composable nodes for? - Robotics Stack Exchange, accessed July 27, 2025, https://robotics.stackexchange.com/questions/115208/what-are-lifecycle-nodes-and-composable-nodes-for
47. MoveIt 2 Documentation - MoveIt Documentation: Humble documentation, accessed July 27, 2025, https://moveit.picknik.ai/humble/index.html
48. Tutorials - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://ftp.udx.icscoe.jp/ros/ros_docs_mirror/en/humble/Tutorials.html
49. fmrico/cascade_lifecycle: Managed nodes (or lifecycle nodes, LN) package provides a mechanism to create LifecycleNode activation trees - GitHub, accessed July 27, 2025, https://github.com/fmrico/cascade_lifecycle
50. How to make a command line argument on Ros2 Humble at launch - Stack Overflow, accessed July 27, 2025, https://stackoverflow.com/questions/73570800/how-to-make-a-command-line-argument-on-ros2-humble-at-launch


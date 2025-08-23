# ROS 2 Humble 메시징 시스템 완벽 가이드


ROS2(Robot Operating System 2)의 심장은 메시징 시스템이다. 이 시스템은 로봇 애플리케이션을 구성하는 분산된 노드(소프트웨어 모듈)들이 서로 통신하는 방식을 정의한다. 이는 단순히 데이터를 한 곳에서 다른 곳으로 보내는 것을 넘어, 전체 시스템의 강건함, 유연성, 그리고 확장성을 결정하는 핵심적인 아키텍처다.1

ROS2의 메시징 시스템은 ROS1의 한계를 극복하기 위해 근본부터 재설계되었다. 가장 큰 변화는 산업 표준 미들웨어인 DDS(Data Distribution Service)를 통신 계층의 기반으로 채택한 것이다.2 이 결정으로 인해 ROS2는 이전 버전과 비교할 수 없는 강력한 기능들을 갖추게 되었다. 본 가이드는 ROS2 Humble의 메시징 시스템을 구성하는 핵심 요소들을 깊이 있게 탐구한다.

이 가이드는 다음과 같은 구조로 진행된다.

1. **통신 패러다임:** 데이터의 흐름과 목적에 따라 토픽(Topics), 서비스(Services), 액션(Actions) 중 무엇을 선택해야 하는지 명확한 기준을 제시한다.
2. **인터페이스 정의:** 통신에 사용될 데이터의 구조를 설계하고, 표준 인터페이스를 활용하며, 필요에 따라 자신만의 커스텀 인터페이스를 만드는 방법을 다룬다.
3. **구현:** 명령줄 도구(CLI)와 Python(rclpy), C++(rclcpp) 코드를 통해 실제로 메시징 시스템을 사용하는 방법을 실습한다.
4. **서비스 품질(QoS):** 통신의 신뢰성, 내구성 등을 제어하여 다양한 네트워크 환경에 대응하는 방법을 알아본다.
5. **아키텍처:** 메시징 시스템의 근간을 이루는 DDS와 RMW(ROS Middleware Interface) 아키텍처를 이해하고, 이것이 시스템 전체에 어떤 영향을 미치는지 분석한다.

이 요소들을 깊이 있게 이해하는 것은 곧 견고하고 효율적인 ROS2 시스템을 설계하는 첫걸음이 될 것이다.4

------


ROS2는 세 가지 주요 통신 패러다임을 제공한다: 토픽, 서비스, 액션. 이들은 각각 다른 목적을 위해 설계되었으며, 어떤 것을 선택하는가는 시스템 전체의 아키텍처에 큰 영향을 미친다. 따라서 각 방식의 개념적 차이를 명확히 이해하고, 개발 중인 시스템의 요구사항에 가장 적합한 방식을 선택하는 것이 매우 중요하다.



- **발행/구독 모델 (Publish/Subscribe Model):** 토픽은 노드 간의 비동기적, 비연결성 데이터 버스(bus) 역할을 한다. 데이터를 생산하는 노드(발행자, Publisher)가 특정 이름의 '토픽'으로 데이터를 발행하면, 해당 토픽에 관심이 있다고 등록한 모든 노드(구독자, Subscriber)가 그 데이터를 수신한다. 이는 일대다(one-to-many), 다대일(many-to-one), 또는 다대다(many-to-many) 통신을 가능하게 한다.7
- **익명성 (Anonymous):** 발행자는 구독자가 누구인지, 몇 명이나 있는지 알 필요가 없다. 마찬가지로 구독자 역시 발행자가 누구인지 신경 쓰지 않는다. 이 느슨한 결합(loose coupling)은 시스템의 유연성과 확장성을 극대화한다. 예를 들어, 실행 중인 시스템에 영향을 주지 않고 `ros2 bag record`와 같은 디버깅 도구를 특정 토픽에 붙여 데이터를 기록하거나, `rqt_graph`로 노드 관계를 시각화하는 것이 가능하다.8
- **강타입 (Strongly-Typed):** 모든 토픽은 특정 메시지 타입(`.msg`)을 가진다. ROS2 시스템은 발행자와 구독자 간에 이 메시지 타입이 정확히 일치하는지를 강제한다. 이는 데이터의 구조와 의미에 대한 일관성을 보장하여, 개발 과정에서의 실수를 줄여준다.8


토픽은 지속적으로 데이터가 흐르는 상황에 가장 이상적이다.

- **실시간 센서 데이터:** LiDAR 센서의 거리 측정값(`sensor_msgs/LaserScan`), 카메라의 이미지 프레임(`sensor_msgs/Image`), IMU의 관성 측정값(`sensor_msgs/Imu`) 등과 같이, 로봇이 환경을 인지하기 위해 지속적으로 생성되는 데이터 스트림을 전달하는 데 사용된다.9
- **로봇 상태 브로드캐스팅:** 로봇의 현재 위치 및 자세(`nav_msgs/Odometry`), 각 관절의 각도와 속도(`sensor_msgs/JointState`), 좌표계 간의 변환 정보(tf2) 등 시스템의 여러 부분이 참조해야 하는 상태 정보를 주기적으로 모든 노드에 알리는 데 효과적이다.10



- **요청/응답 모델 (Request/Response Model):** 서비스는 원격 프로시저 호출(RPC)과 유사한 통신 모델을 따른다. 클라이언트(Client) 노드가 특정 작업을 요청(Request)하면, 서버(Server) 노드가 그 요청을 처리하고 단 하나의 응답(Response)을 반환한다.14
- **동기식 상호작용 (Synchronous-like Interaction):** 클라이언트는 일반적으로 서버로부터 응답을 받을 때까지 대기한다(blocking). 물론 비동기 클라이언트를 구현하여 대기 없이 다른 작업을 수행할 수도 있지만, 서비스의 근본적인 패러다임은 동기식 RPC에 가깝다.10
- **빠른 응답성:** 서비스는 장시간이 소요되는 작업에는 부적합하다. 서비스 콜은 빠르게 완료될 수 있는 계산이나 상태 쿼리를 위해 설계되었다. 만약 작업이 오래 걸린다면, 클라이언트는 응답이 올 때까지 멈춰있게 되어 시스템 전체의 반응성을 해칠 수 있다.10


서비스는 즉각적인 응답이 필요한 단발성 상호작용에 적합하다.

- **상태 쿼리:** "로봇 팔의 현재 엔드 이펙터(end-effector) 좌표는?" 또는 "현재 로딩된 맵의 이름은 무엇인가?"와 같이 특정 노드의 현재 상태를 질의하고 즉시 답을 얻고자 할 때 사용된다.10
- **트리거 및 설정 변경:** "시뮬레이션을 리셋해라" 또는 "카메라의 노출 값을 100으로 변경해라"와 같이 시스템의 상태를 변경하거나 특정 동작을 촉발하는 단발성 명령에 사용된다.11
- **빠른 계산:** 로봇 팔의 목표 자세를 주고 역기구학(IK) 계산을 요청하여 관절 각도를 즉시 받아야 하는 경우와 같이, 입력값을 주고 계산 결과를 바로 활용해야 할 때 유용하다.10



- **목표/피드백/결과 모델 (Goal/Feedback/Result Model):** 액션은 서비스와 같이 목표(Goal)를 요청하고 최종 결과(Result)를 받는다는 점에서는 유사하지만, 결정적인 차이점이 있다. 목표가 실행되는 동안 서버는 클라이언트에게 지속적으로 진행 상황에 대한 피드백(Feedback)을 보낼 수 있다.17
- **비동기 실행 (Asynchronous Execution):** 액션 클라이언트는 서버에 목표를 보낸 후, 작업이 완료될 때까지 기다릴 필요 없이 즉시 다른 작업을 수행할 수 있다. 작업의 진행 상황은 피드백 콜백을 통해 비동기적으로 전달받는다.18
- **선점 가능성 (Preemptible):** 클라이언트는 진행 중인 작업을 언제든지 취소(cancel)할 수 있다. 이는 내비게이션이나 복잡한 조작과 같이, 예상치 못한 상황이 발생했을 때 동작을 즉시 중단시켜야 하는 장시간 실행 작업에서 필수적인 기능이다.10


액션은 시간이 오래 걸리고, 중간 과정에 대한 모니터링이 필요하며, 취소 기능이 요구되는 복잡한 작업에 사용된다.

- **내비게이션:** 로봇에게 "(x, y) 좌표로 이동하라"는 목표를 부여한다. 로봇은 이동하는 동안 현재 위치, 목표 지점까지의 남은 거리 등을 피드백으로 계속 보내고, 클라이언트는 이 정보를 바탕으로 진행 상황을 UI에 표시하거나, 장애물이 나타나면 이동 명령을 취소할 수 있다.11
- **로봇 팔 조작:** "테이블 위의 물체를 집어서 상자 안에 넣어라"와 같은 복잡한 매니퓰레이션 작업을 액션으로 모델링할 수 있다. '물체로 접근 중', '그리퍼 파지 중', '목표 지점으로 이동 중', '물체 놓는 중'과 같은 각 단계의 진행 상황을 피드백으로 받는다.10
- **복잡한 작업 시퀀스:** "주방을 청소하라"와 같이 여러 단계(바닥 스캔, 쓰레기 인식, 쓰레기 수거, 충전 스테이션 복귀)로 구성된 장시간 작업을 하나의 액션으로 정의하고 관리할 수 있다.22


토픽, 서비스, 액션의 선택은 단순히 데이터 전달 방식을 넘어, 노드 간의 **결합도(Coupling)**와 **상태 관리(State Management)**에 대한 근본적인 아키텍처 설계 결정이다.

[표 1-1]은 세 가지 통신 패러다임의 주요 특징을 비교한다.

| 특성               | 토픽 (Topic)          | 서비스 (Service)       | 액션 (Action)             |
| ------------------ | --------------------- | ---------------------- | ------------------------- |
| **통신 모델**      | 발행/구독             | 요청/응답              | 목표/피드백/결과          |
| **동기성**         | 비동기                | 동기 (클라이언트 관점) | 비동기                    |
| **실행 시간**      | 제약 없음             | 짧은 시간              | 긴 시간                   |
| **피드백**         | 없음                  | 없음                   | 있음                      |
| **취소(선점)**     | 불가                  | 불가                   | 불가                      |
| **연결 관계**      | 다대다 (M:N)          | 일대일 (1:1)           | 일대일 (1:1)              |
| **결합도**         | 매우 낮음 (Decoupled) | 높음 (Tightly Coupled) | 중간 (Statefully Coupled) |
| **주요 사용 사례** | 센서 데이터/상태 방송 | 빠른 RPC/트리거        | 내비게이션/조작           |

이 표에서 '결합도'는 특히 중요한 개념이다. ROS2의 통신 방식은 단순한 API가 아니라, 시스템의 모듈성, 견고성, 확장성을 결정하는 아키텍처 패턴이다.

- **토픽**은 '익명성'을 통해 극도의 **디커플링(decoupling)**을 제공한다.8 발행자는 구독자의 존재 자체를 몰라도 되므로, 시스템의 일부를 교체하거나 새로운 모니터링 노드를 추가하는 것이 매우 쉽다. 이는 유연하고 확장 가능한 시스템을 만드는 데 유리하지만, 데이터 전달을 보장하지 않을 수 있다(QoS 설정에 따라 다름).
- **서비스**는 클라이언트가 서버를 호출하고 응답을 기다리는 RPC 패턴이다. 이는 클라이언트 코드가 서버의 존재와 즉각적인 응답을 가정하게 되므로 노드 간에 **강한 결합(tight coupling)**을 유발한다. 서버가 다운되거나 응답이 늦어지면 클라이언트의 동작에 직접적인 영향을 미친다. 이는 시스템의 유연성을 저해할 수 있지만, 강력한 동기적 보장을 제공한다.14
- **액션**은 이 둘의 절충안이다. 액션은 목표 ID를 통해 특정 작업의 상태(진행 중, 성공, 실패, 취소됨)를 추적한다.18 이는 클라이언트와 서버가 '목표'라는 특정 컨텍스트 하에서 장시간 상태를 공유함을 의미한다. 이는 단순한 디커플링이나 강한 결합이 아닌, **상태 기반 결합(stateful coupling)**이라는 제3의 형태로 볼 수 있다. 클라이언트의 비동기성을 보장하면서도 복잡한 로봇 동작을 우아하게 모델링할 수 있게 해준다.

결론적으로, 어떤 통신 방식을 선택하는가는 "데이터를 어떻게 보낼까?"라는 단순한 질문이 아니다. 이는 "노드들이 서로에 대해 얼마나 알아야 하고, 상호작용의 생명주기를 어떻게 관리할 것인가?"라는 아키텍처 수준의 질문에 답하는 과정이다.

------


ROS2 통신에서 주고받는 모든 데이터는 '인터페이스(Interface)'라는 명확한 구조를 통해 정의된다. 인터페이스를 잘 설계하고 관리하는 것은 시스템의 안정성과 재사용성을 높이는 데 매우 중요하다. 이 장에서는 ROS2가 제공하는 표준 인터페이스를 효과적으로 활용하는 방법과, 필요에 따라 애플리케이션에 특화된 커스텀 인터페이스를 설계하고 빌드하는 전 과정을 상세히 다룬다.


ROS2는 재사용성과 표준화를 위해 `common_interfaces`라는 패키지 그룹을 통해 다양한 분야에서 널리 사용되는 메시지 타입들을 미리 정의하여 제공한다.23 새로운 인터페이스를 만들기 전에, 이미 원하는 목적에 맞는 표준 인터페이스가 있는지 확인하는 것이 좋다. 이는 다른 ROS2 패키지(예: RViz, Navigation2)와의 호환성을 보장하는 가장 좋은 방법이다.

- **`std_msgs`:** 문자열(`String`), 정수(`Int32`), 불리언(`Bool`) 등 프로그래밍의 기본적인 데이터 타입을 포함한다. 주로 간단한 예제나 디버깅 목적으로 사용된다. 실제 애플리케이션에서는 데이터의 의미를 명확하게 나타내는 커스텀 메시지를 정의하는 것이 권장된다.12
- **`geometry_msgs`:** 로봇의 기하학적 정보를 표현하는 데 필수적인 메시지들을 포함한다. 3차원 공간상의 위치(`Point`), 방향(`Quaternion`), 그리고 이 둘을 합친 자세(`Pose`)와 속도(`Twist`) 등이 여기에 속한다. 로봇의 움직임을 제어하거나 상태를 표현할 때 가장 빈번하게 사용된다.12
- **`sensor_msgs`:** 다양한 센서의 데이터를 표현하기 위한 표준 메시지들을 포함한다. 카메라 이미지(`Image`), LiDAR 스캔 데이터(`LaserScan`), 관성 측정 장치 데이터(`Imu`), GPS 데이터(`NavSatFix`) 등이 대표적이다. 표준 센서 메시지를 사용하면 RViz와 같은 시각화 도구에서 별도의 설정 없이 바로 센서 데이터를 확인할 수 있는 장점이 있다.12

[표 2-1]은 다양한 ROS2 애플리케이션에서 자주 사용되는 주요 표준 인터페이스 메시지 타입들을 요약한 것이다.

| 패키지          | 메시지 타입   | 설명                                   | 주요 필드                                                    | 사용 예시                                       |
| --------------- | ------------- | -------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------- |
| `std_msgs`      | `String`      | 간단한 문자열 데이터                   | `string data`                                                | 디버깅 메시지 출력, 간단한 명령 전달            |
| `geometry_msgs` | `PoseStamped` | 타임스탬프와 좌표계 정보가 포함된 자세 | `std_msgs/Header header`, `Pose pose`                        | 로봇의 목표 자세 지정, 객체의 위치 및 방향 표현 |
| `geometry_msgs` | `Twist`       | 선속도와 각속도                        | `Vector3 linear`, `Vector3 angular`                          | 로봇의 이동 속도 제어 명령 (`/cmd_vel` 토픽)    |
| `sensor_msgs`   | `Image`       | 카메라 이미지 데이터                   | `Header header`, `uint32 height`, `uint32 width`, `uint8 data` | 카메라 드라이버, 이미지 처리 노드               |
| `sensor_msgs`   | `LaserScan`   | 2D LiDAR 스캔 데이터                   | `Header header`, `float32 angle_min`, `float32 angle_max`, `float32 ranges` | 장애물 회피, SLAM                               |
| `sensor_msgs`   | `Imu`         | 관성 측정 장치 데이터                  | `Header header`, `Quaternion orientation`, `Vector3 angular_velocity`, `Vector3 linear_acceleration` | 로봇의 자세 추정                                |
| `sensor_msgs`   | `JointState`  | 로봇 관절들의 상태                     | `Header header`, `string name`, `float64 position`, `float64 velocity` | 로봇 팔 제어, 로봇 상태 시각화                  |


표준 인터페이스만으로는 애플리케이션에 특화된 모든 데이터를 표현하기 어렵다. 예를 들어, '여러 개의 LED 색상과 밝기를 동시에 제어하는 명령'이나 '로봇 그리퍼의 현재 상태(열림, 닫힘, 파지력)'와 같은 데이터는 커스텀 인터페이스로 정의해야 한다.

- **인터페이스 전용 패키지 (Interface-only Package):** 커스텀 인터페이스는 반드시 `ament_cmake` 빌드 타입의 전용 패키지에서 관리하는 것이 모범 사례(best practice)이다. 이렇게 하면 인터페이스의 정의와 이를 사용하는 노드의 로직을 분리하여 코드의 재사용성과 관리 용이성을 높일 수 있다. 패키지 이름은 관례적으로 `[project_name]_interfaces`와 같이 짓는다.26

- **디렉토리 구조:** 생성한 인터페이스 패키지 내부에 `msg/`, `srv/`, `action/` 디렉토리를 만들고, 각 타입에 맞는 정의 파일을 위치시킨다.26

- **파일 구조:**

  - **`.msg` 파일:** 메시지를 구성하는 필드들의 타입과 이름을 한 줄에 하나씩 정의한다. 다른 패키지의 메시지 타입을 포함하여 복합적인 데이터 구조를 만들 수 있다.27

    - *예시 (`Sphere.msg`):*

      ```
      geometry_msgs/Point center
      float64 radius
      ```

  - **`.srv` 파일:** 서비스의 요청(request) 부분과 응답(response) 부분을 `---` 구분자로 분리하여 정의한다.27

    - *예시 (`AddThreeInts.srv`):*

      ```
      int64 a
      int64 b
      int64 c
      ---
      int64 sum
      ```

  - **`.action` 파일:** 액션의 목표(goal), 결과(result), 피드백(feedback) 부분을 각각 `---` 구분자로 분리하여 정의한다.29

    - *예시 (`Fibonacci.action`):*

      ```
      int32 order
      ---
      int32 sequence
      ---
      int32 partial_sequence
      ```


커스텀 인터페이스를 정의하는 것만으로는 충분하지 않다. ROS2 빌드 시스템에 이 인터페이스의 존재를 알리고, C++이나 Python과 같은 프로그래밍 언어에서 사용할 수 있는 코드로 변환하는 과정이 필요하다.


`package.xml` 파일은 패키지의 메타 정보와 의존성을 정의한다. 인터페이스 패키지에서는 다음 태그들이 특히 중요하다.

- `<buildtool_depend>rosidl_default_generators</buildtool_depend>`: 이 태그는 빌드 시점에 `.msg`, `.srv`, `.action` 파일로부터 각 언어에 맞는 코드를 생성하는 `rosidl` 생성기 도구가 필요함을 명시한다.27
- `<exec_depend>rosidl_default_runtime</exec_depend>`: 이 태그는 실행 시점에 생성된 인터페이스 코드를 사용하기 위해 `rosidl` 런타임 라이브러리가 필요함을 명시한다.27
- `<member_of_group>rosidl_interface_packages</member_of_group>`: 이 패키지가 인터페이스를 제공하는 패키지 그룹의 일원임을 선언한다. 이 선언이 있어야 다른 패키지에서 이 인터페이스를 쉽게 찾고 의존성을 설정할 수 있다.27
- `<depend>geometry_msgs</depend>`: 만약 커스텀 인터페이스가 `Sphere.msg`의 예시처럼 `geometry_msgs`와 같은 다른 인터페이스 패키지의 메시지를 사용한다면, 해당 패키지에 대한 일반 의존성을 추가해야 한다.27


`ament_cmake` 타입의 인터페이스 패키지에서는 `CMakeLists.txt` 파일이 빌드 과정을 제어한다.

- `find_package(rosidl_default_generators REQUIRED)`: 빌드에 필요한 `rosidl` 코드 생성기 패키지를 찾는다.27
- `find_package(geometry_msgs REQUIRED)`: `Sphere.msg`가 의존하는 `geometry_msgs` 패키지를 찾는다.27
- `rosidl_generate_interfaces(${PROJECT_NAME} "msg/Sphere.msg" "srv/AddThreeInts.srv" DEPENDENCIES geometry_msgs)`: 이 함수가 인터페이스 빌드의 핵심이다.
  - 첫 번째 인자 `${PROJECT_NAME}`은 생성될 라이브러리의 이름으로, 현재 패키지 이름과 동일해야 한다.
  - 그 다음으로 변환할 모든 `.msg`, `.srv`, `.action` 파일들의 목록을 나열한다.
  - `DEPENDENCIES` 키워드 뒤에는 이 인터페이스들이 의존하는 다른 메시지 패키지들(예: `geometry_msgs`)을 명시해야 한다. 이 부분이 누락되면 빌드 에러가 발생한다.27

그렇다면 왜 인터페이스 패키지는 반드시 `ament_cmake` 타입이어야만 할까? 이는 ROS2의 다국어 지원이라는 핵심 철학과 깊은 관련이 있다. ROS2에서 인터페이스 정의와 코드 생성은 특정 프로그래밍 언어에 종속되지 않는 근본적인 빌드 과정이다. `rosidl`이라는 도구는 `.msg`와 같은 추상적인 인터페이스 정의를 읽어들여, 이를 먼저 DDS가 사용하는 표준 언어인 IDL(`.idl`) 파일로 변환한다.35 그 다음, 이 IDL 파일을 기반으로 C++ 헤더 파일, Python 클래스 파일 등 각 언어에 맞는 구체적인 코드를 생성한다.

이 변환 및 생성 과정 전체를 관장하는 `rosidl` 도구 모음 자체가 C/C++ 기반으로 구축되어 있다. 따라서 이 과정을 실행하고 제어하는 빌드 시스템은 C++ 빌드 도구인 `CMake`를 사용하는 `ament_cmake`여야만 한다. 이는 Python만 사용하는 개발자에게는 다소 번거롭게 느껴질 수 있지만, ROS2의 가장 큰 장점 중 하나인 다국어 지원을 가능하게 하는 의도적인 설계 결정이다. 이 아키텍처는 데이터의 '명세(specification)'와 '구현(implementation)'을 명확히 분리하여, 언어에 구애받지 않는 견고하고 재사용 가능한 시스템을 구축하도록 유도한다.

------


개념을 이해하고 인터페이스를 정의했다면, 이제 실제로 노드를 만들고 통신을 구현할 차례다. ROS2는 강력한 명령줄 도구(CLI)를 제공하여 시스템을 실시간으로 분석하고 디버깅할 수 있게 해주며, Python과 C++를 위한 직관적인 클라이언트 라이브러리(RCL)를 제공한다.


`ros2` CLI는 실행 중인 ROS 시스템을 들여다보고(introspect), 직접 상호작용할 수 있는 가장 기본적이면서도 강력한 도구다. 코드를 한 줄도 작성하지 않고도 시스템의 상태를 확인하고, 데이터를 주입하며, 문제를 진단할 수 있다.36

- **`ros2 topic`:** 토픽 관련 작업을 위한 명령어 모음이다.
  - `list [-t]`: 현재 활성화된 모든 토픽의 목록 [과 해당 메시지 타입]을 확인한다.7
  - `info <topic_name>`: 특정 토픽의 메시지 타입, 현재 발행자(Publisher) 수와 구독자(Subscriber) 수를 보여준다.7
  - `echo <topic_name>`: 특정 토픽으로 발행되는 메시지의 내용을 터미널에 실시간으로 출력한다. 데이터가 제대로 발행되는지 확인할 때 가장 유용하다.7
  - `pub <topic_name> <msg_type> '<args>' [--once]`: 특정 토픽으로 메시지를 한 번(`--once`) 또는 1Hz 주기로 반복해서 발행한다. 센서 노드 없이 특정 입력을 테스트할 때 매우 유용하다. 인자는 YAML 형식으로 입력한다.7
  - `hz <topic_name>`: 특정 토픽의 메시지 발행 주기(Hz)를 측정한다. 센서 데이터가 원하는 주기로 들어오는지 확인할 때 사용한다.7
- **`ros2 service`:** 서비스 관련 작업을 위한 명령어 모음이다.
  - `list [-t]`: 현재 활성화된 모든 서비스의 목록 [과 해당 서비스 타입]을 확인한다.14
  - `type <service_name>`: 특정 서비스의 타입을 확인한다.14
  - `find <service_type>`: 특정 서비스 타입을 제공하는 모든 서비스를 검색한다.14
  - `call <service_name> <srv_type> '<args>'`: 특정 서비스를 직접 호출하고 서버로부터의 응답을 터미널에서 확인한다. 서비스가 정상적으로 동작하는지 테스트할 때 사용한다.14
- **`ros2 action`:** 액션 관련 작업을 위한 명령어 모음이다.
  - `list [-t]`: 현재 활성화된 모든 액션의 목록 [과 해당 액션 타입]을 확인한다.17
  - `info <action_name>`: 특정 액션의 서버와 클라이언트 정보를 보여준다.17
  - `send_goal <action_name> <action_type> '<goal>' [--feedback]`: 특정 액션 서버에 목표를 전송한다. `--feedback` 옵션을 추가하면 목표가 실행되는 동안 서버가 보내는 피드백을 실시간으로 확인할 수 있다.17
- **`ros2 interface show <interface_type>`:** 모든 종류의 인터페이스(`.msg`, `.srv`, `.action`)의 상세한 구조(필드 이름과 타입)를 확인한다. `pub`이나 `call` 명령을 사용하기 전에 데이터 구조를 확인할 때 필수적이다.7


`rclpy`는 ROS2의 공식 Python 클라이언트 라이브러리로, Python을 사용하여 ROS2 노드를 쉽게 작성할 수 있게 해준다.

- **발행자/구독자 (Publisher/Subscriber):**
  - 모든 노드는 `rclpy.node.Node` 클래스를 상속받아 생성한다.
  - 발행자는 `self.create_publisher(MessageType, 'topic_name', qos_profile)`를 호출하여 생성한다. `qos_profile`은 통신 품질을 결정하며, 보통 10과 같은 정수값으로 큐 크기를 지정한다.39
  - 구독자는 `self.create_subscription(MessageType, 'topic_name', self.callback_function, qos_profile)`를 호출하여 생성한다. 메시지가 수신될 때마다 지정된 `callback_function`이 자동으로 호출된다.39
  - 발행자는 `publisher.publish(msg)` 메서드를 사용하여 준비된 메시지 객체를 토픽으로 발행한다.39
- **서비스 서버/클라이언트 (Service Server/Client):**
  - 서비스 서버는 `self.create_service(ServiceType, 'service_name', self.callback_function)`를 호출하여 생성한다. 콜백 함수는 `request`와 `response` 객체를 인자로 받아, 요청을 처리한 후 응답 객체를 채워 반환해야 한다.16
  - 서비스 클라이언트는 `self.create_client(ServiceType, 'service_name')`를 호출하여 생성한다.16
  - 클라이언트는 요청을 보내기 전에 `while not self.cli.wait_for_service(timeout_sec=1.0):`과 같은 코드를 사용하여 서버가 준비될 때까지 기다리는 것이 안전하다.16
  - 요청은 `future = self.cli.call_async(req)`를 통해 비동기적으로 전송된다. `rclpy.spin_until_future_complete(self, future)`를 사용하여 응답이 올 때까지 대기하고, `future.result()`로 응답을 얻을 수 있다.16
- **액션 서버/클라이언트 (Action Server/Client):**
  - 액션 서버는 `rclpy.action.ActionServer` 클래스를 사용하여 생성하며, 목표 요청을 처리할 `execute_callback`을 등록한다.20
  - `execute_callback` 내에서는 `goal_handle` 객체를 통해 목표의 최종 상태를 관리한다. `goal_handle.succeed()`는 성공, `goal_handle.abort()`는 실패를 의미한다. `goal_handle.publish_feedback(feedback_msg)`를 사용하여 클라이언트에게 진행 상황을 주기적으로 알릴 수 있다.20
  - 액션 클라이언트는 `rclpy.action.ActionClient` 클래스를 사용하여 생성한다.20
  - `future = self.client.send_goal_async(goal_msg, feedback_callback=self.feedback_callback)`를 사용하여 목표를 비동기적으로 전송한다. `send_goal_async`는 목표가 수락되었는지 여부를 알려주는 `future`를 반환하며, 피드백과 최종 결과를 처리하기 위한 콜백 함수들을 함께 등록할 수 있다.20


`rclcpp`는 ROS2의 공식 C++ 클라이언트 라이브러리다. `rclpy`와 개념적으로는 동일한 기능들을 제공하지만, C++의 특징인 정적 타입, 컴파일 타임 검사, 스마트 포인터(`std::shared_ptr`) 등을 활용하여 더 높은 성능과 안정성을 제공한다.

- **발행자/구독자:** Python과 개념은 동일하다. `create_publisher<MessageType>()`와 `create_subscription<MessageType>()` 템플릿 함수를 사용하며, 콜백 함수는 `std::bind`를 사용하여 멤버 함수를 바인딩하는 경우가 많다.
- **서비스 서버/클라이언트:**
  - 서버는 `create_service<ServiceType>()`로 생성한다. 콜백 함수는 `const std::shared_ptr<Request>`와 `std::shared_ptr<Response>`를 인자로 받는다.41
- **액션 서버/클라이언트:**
  - 서버는 `rclcpp_action::create_server<ActionType>()`를 사용하여 생성한다. 목표 수락 여부를 결정하는 `handle_goal`, 취소 요청을 처리하는 `handle_cancel`, 실제 목표를 실행하는 `handle_accepted` 등 여러 콜백을 등록하여 상태를 정밀하게 관리한다.42

------


ROS2의 가장 강력한 기능 중 하나는 서비스 품질(Quality of Service, QoS)을 세밀하게 제어할 수 있다는 점이다. QoS는 노드 간의 통신이 얼마나 신뢰성 있게, 그리고 어떤 방식으로 이루어질지를 정의하는 규칙들의 집합이다. 이를 통해 개발자는 다양한 네트워크 환경과 애플리케이션의 요구사항에 맞춰 통신을 최적화할 수 있다.


ROS1은 주로 안정적인 유선 LAN 환경을 가정하고 설계되었으며, 기본 통신 프로토콜로 신뢰성 있는 TCP(TCPROS)를 사용했다.9 이 방식은 데이터 유실이 거의 없는 환경에서는 잘 작동했지만, 손실이 잦은 무선 네트워크나, 데이터의 최신성이 신뢰성보다 더 중요한 실시간 시스템에서는 한계가 있었다.

ROS2는 이러한 한계를 극복하기 위해 DDS를 통신 기반으로 채택했다. DDS는 기본적으로 비연결성 프로토콜인 UDP 위에서 동작하므로, 개발자는 통신의 신뢰성 수준을 직접 선택할 수 있다. QoS는 바로 이 유연성을 개발자가 제어할 수 있도록 제공되는 강력한 도구다. QoS를 통해 ROS2 통신은 TCP처럼 신뢰성을 보장할 수도, UDP처럼 최선을 다해 전달할 수도 있으며, 그 사이의 수많은 상태를 구현할 수 있다.9


QoS 프로파일은 여러 QoS '정책(Policy)'들의 조합으로 구성된다. 가장 중요하고 자주 사용되는 정책들은 다음과 같다.

- **`Reliability` (신뢰성):** 메시지 전달을 보장할지 여부를 결정한다.
  - `RELIABLE`: TCP처럼 메시지 전달을 보장한다. 수신 확인(acknowledgement)과 재전송 메커니즘을 통해 메시지가 반드시 전달되도록 시도한다. 서비스 요청/응답이나 액션 목표처럼 단 한 번의 유실도 허용되지 않는 중요한 통신에 사용된다.9
  - `BEST_EFFORT`: UDP처럼 메시지를 최선을 다해 한 번만 보낸다. 네트워크 상황에 따라 메시지가 유실될 수 있다. 과거 데이터보다는 최신 데이터가 더 중요하고, 일부 데이터 유실은 감수할 수 있는 고주파 센서 데이터(예: 카메라 영상, LiDAR 스캔)에 적합하다. 신뢰성을 위한 오버헤드가 없어 더 빠르고 효율적이다.9
- **`Durability` (내구성):** 발행자가 메시지를 보낸 후, 뒤늦게 구독을 시작한 노드에게 메시지를 전달할지 여부를 결정한다.
  - `VOLATILE`: '휘발성'이라는 의미 그대로, 발행 시점에 이미 구독하고 있는 노드에게만 메시지를 전달한다. 구독자가 없으면 메시지는 그냥 사라진다. 대부분의 통신에 사용되는 기본값이다.9
  - `TRANSIENT_LOCAL`: 발행자가 보낸 메시지를 로컬 버퍼에 저장하고 있다가, 나중에 구독을 시작하는("late-joining") 구독자에게도 가장 최근에 발행된 메시지들을 전달해준다. 이는 ROS1의 'latching' 기능과 유사하다. 로봇의 정적인 설정값, SLAM으로 생성된 지도 데이터처럼 한 번 발행된 후 시스템이 실행되는 동안 계속 유효해야 하는 데이터를 전달하는 데 매우 유용하다.9
- **`History` & `Depth` (이력 & 깊이):** 발행자 측의 메시지 큐를 어떻게 관리할지 결정한다.
  - `KEEP_LAST` (History) + `depth` (N): 발행자가 메시지를 보내는 속도가 구독자가 처리하는 속도보다 빠를 때, 발행자 측 큐(queue)에 가장 최신 N개의 메시지만을 유지하고 오래된 것은 버린다. 이는 메모리 사용량을 제한하는 데 효과적이다. 대부분의 경우 이 설정을 사용한다.9
  - `KEEP_ALL` (History): 리소스 한도 내에서 발행된 모든 메시지를 큐에 유지한다. `RELIABLE` 신뢰성 정책과 함께 사용될 때, 모든 메시지가 순서대로 유실 없이 전달되도록 보장하는 데 중요한 역할을 한다.43


매번 모든 QoS 정책을 수동으로 설정하는 것은 복잡하고 오류가 발생하기 쉽다. 이를 위해 ROS2는 일반적인 사용 사례에 맞춰 미리 정의된 QoS 프로파일들을 제공한다.9

[표 4-1]은 주요 사전 정의 QoS 프로파일의 설정과 사용 사례를 보여준다.

| 프로파일           | Reliability       | Durability        | History           | Depth | 주요 사용 사례                                               |
| ------------------ | ----------------- | ----------------- | ----------------- | ----- | ------------------------------------------------------------ |
| **Default**        | `RELIABLE`        | `VOLATILE`        | `KEEP_LAST`       | 10    | 일반적인 발행/구독. ROS1과 유사한 동작을 제공하여 마이그레이션을 돕는다.9 |
| **Services**       | `RELIABLE`        | `VOLATILE`        | `KEEP_LAST`       | 10    | 서비스 통신. 재시작된 서버가 오래된 요청을 받지 않도록 `VOLATILE` 설정이 중요하다.9 |
| **Sensor Data**    | `BEST_EFFORT`     | `VOLATILE`        | `KEEP_LAST`       | 5     | 고주파 센서 데이터. 최신성이 신뢰성보다 중요하며, 약간의 데이터 유실은 허용된다.9 |
| **Parameters**     | `RELIABLE`        | `VOLATILE`        | `KEEP_LAST`       | 1000  | 파라미터 통신. 통신이 잠시 끊겨도 파라미터 설정 요청이 유실되지 않도록 큰 큐 크기를 가진다.9 |
| **System Default** | RMW 구현체 기본값 | RMW 구현체 기본값 | RMW 구현체 기본값 | -     | RMW 벤더마다 설정이 다를 수 있어 이식성을 위해 명시적인 프로파일 사용이 권장된다.9 |


발행자와 구독자 간의 통신이 이루어지려면, 양측의 QoS 프로파일이 서로 '호환(compatible)'되어야 한다. 만약 호환되지 않는 QoS 설정으로 통신을 시도하면, 아무런 에러 메시지 없이 조용히 연결이 실패하고 데이터가 전달되지 않는다. 이는 ROS2 초보자들이 가장 흔하게 겪는 문제 중 하나다.

- **호환성 원칙:** 구독자는 발행자가 제공하는 것보다 더 높은 수준의 서비스를 요구할 수 없다. 예를 들어, 발행자가 `BEST_EFFORT`로 데이터를 보내고 있는데, 구독자가 `RELIABLE`을 요구하면 연결은 성립되지 않는다. 반대로, 발행자가 `RELIABLE`로 보내고 구독자가 `BEST_EFFORT`로 받는 것은 가능하다.9
- **디버깅:** 노드 간 통신이 안 될 때는 가장 먼저 QoS 호환성을 의심해야 한다. `ros2 topic info <topic_name> -v` 명령어를 사용하면 해당 토픽에 연결된 각 발행자와 구독자의 상세한 QoS 설정을 확인할 수 있다. 이 정보를 비교하여 불일치하는 부분을 찾아 수정하면 문제를 해결할 수 있다.45

------


ROS2 메시징 시스템의 강력함과 유연성은 그 기반이 되는 아키텍처에서 비롯된다. 왜 ROS2가 DDS를 선택했는지, 그리고 RMW라는 추상화 계층이 어떤 역할을 하는지를 이해하는 것은 ROS2의 동작 방식을 깊이 있게 파악하고, 복잡한 문제를 해결하는 데 필수적이다.


ROS1은 `roscore`라는 중앙 집중식 마스터 노드와 자체적으로 개발한 TCPROS/UDPROS 통신 프로토콜에 의존했다. 이는 단일 로봇 환경에서는 잘 작동했지만, 여러 로봇으로 구성된 대규모 시스템이나 불안정한 네트워크 환경에서는 확장성과 강건성에 한계를 보였다.

ROS2는 이러한 문제를 해결하기 위해 DDS(Data Distribution Service)를 핵심 미들웨어로 채택했다. DDS는 OMG(Object Management Group)에서 제정한 산업 표준으로, 항공, 국방, 금융 등 높은 신뢰성과 성능이 요구되는 분야에서 수십 년간 검증된 기술이다. ROS2가 DDS를 선택한 이유는 다음과 같다.2

- **분산 발견(Distributed Discovery):** 중앙 마스터 없이 노드들이 서로를 자동으로 발견하여 통신망을 형성한다. 이는 단일 실패 지점(Single Point of Failure)을 제거하여 시스템의 강건성을 높인다.
- **풍부한 QoS 정책:** 통신의 신뢰성, 내구성, 실시간성 등을 매우 세밀하게 제어할 수 있다.
- **보안:** 암호화, 인증, 접근 제어 등 강력한 보안 기능을 표준으로 제공한다.
- **상호 운용성:** 다른 벤더의 DDS 구현체와도 표준 프로토콜(RTPS)을 통해 통신할 수 있다.


DDS는 매우 강력하지만, 그 자체로 복잡하고 다양한 벤더(e.g., eProsima Fast DDS, RTI Connext DDS, Eclipse Cyclone DDS)들이 존재한다. 만약 ROS2가 특정 벤더의 DDS API에 직접적으로 의존했다면, ROS2 생태계 전체가 해당 벤더에 종속되었을 것이다.

이를 피하기 위해 ROS2는 RMW(ROS Middleware Interface)라는 독창적인 추상화 계층을 도입했다. RMW는 ROS2의 상위 API(rclcpp, rclpy)와 특정 DDS 구현체 사이의 '어댑터' 또는 '플러그인 인터페이스' 역할을 한다.35

- **역할:** 개발자가 `create_publisher`와 같은 ROS2 API를 호출하면, RMW 구현체는 이 호출을 자신이 지원하는 특정 DDS 벤더의 API(예: `create_datawriter`)로 변환해준다.
- **장점 (벤더 독립성):** 개발자는 DDS 벤더의 세부적인 API를 알 필요 없이 일관된 ROS2 API로 코드를 작성할 수 있다. 환경 변수 설정만으로 기본 RMW 구현체를 다른 것으로 쉽게 교체할 수 있어, 애플리케이션의 요구사항(성능, 라이선스 등)에 가장 적합한 DDS를 선택할 수 있는 유연성을 제공한다.47
- **단점 (기능 제한):** RMW는 모든 DDS 벤더가 공통적으로 지원하는 기능의 교집합을 중심으로 설계되었다. 따라서 특정 벤더만 제공하는 고유의 고급 기능을 ROS2 API를 통해 직접 사용하기는 어렵다. 또한, ROS2의 개념(예: 토픽 네이밍 규칙)을 DDS의 개념으로 변환하는 과정에서 '네이티브' DDS 시스템과의 완벽한 상호 운용성에 일부 제약이 생길 수 있다.48


이 DDS/RMW 아키텍처는 ROS2 메시징 시스템의 핵심 동작 방식에 직접적인 영향을 미친다.

- **분산 발견 (Discovery):** ROS1의 `roscore`가 사라졌다. ROS2 노드(내부적으로는 DDS Participant)는 시작과 동시에 DDS의 내장된 분산 발견 프로토콜(Simple Discovery Protocol)을 통해 네트워크상의 다른 노드와 토픽 정보를 자동으로 교환한다. RMW는 이 복잡한 과정을 ROS 사용자에게는 투명하게 만들어, 마치 노드들이 마법처럼 서로를 찾는 것처럼 보이게 한다.2
- **메시지 직렬화와 타입 서포트 (Serialization & Type Support):**
  1. 개발자는 언어에 독립적인 `.msg` 파일에 데이터 구조를 정의한다.
  2. 빌드 시, `rosidl` 도구는 이 `.msg` 파일을 DDS가 표준으로 사용하는 인터페이스 정의 언어인 `.idl` 파일로 변환한다.35
  3. RMW의 일부인 '타입 서포트(Type Support)' 라이브러리는 이 `.idl` 파일을 기반으로 생성된 C++ 또는 Python 코드를 사용하여, 프로그램 내의 ROS 메시지 객체와 네트워크를 통해 전송될 DDS 데이터 형식 간의 변환(직렬화/역직렬화)을 책임진다. 이 과정 덕분에 개발자는 저수준의 데이터 변환에 대해 신경 쓸 필요가 없다.35
- **QoS 변환:** 개발자가 ROS2 코드에서 지정한 QoS 프로파일(예: `SensorDataQoS()`)은 RMW 계층을 통해 실제 DDS 구현체가 이해할 수 있는 세부적인 QoS 정책 값들(예: `RELIABILITY_QOS_POLICY_BEST_EFFORT`)로 변환되어 통신 채널에 적용된다.43


RMW의 존재는 ROS2에 엄청난 유연성을 주지만, 한편으로는 '의도된 불완전함'을 내포한다. ROS2의 설계 목표는 '완벽한 DDS 래퍼(wrapper)'가 되는 것이 아니라, '일관성 있고 사용하기 쉬운 ROS 생태계'를 우선시하는 것이다. 이 추상화는 DDS의 복잡성을 숨겨 개발자 경험을 향상시키지만, 동시에 DDS의 모든 잠재력을 활용하는 것을 막고 네이티브 시스템과의 통합을 더 복잡하게 만드는 '의도된 트레이드오프'이다.

예를 들어, ROS2의 토픽 이름 규칙(예: `/my_topic`)과 DDS의 토픽 이름 규칙이 다르거나, ROS2가 사용하는 DDS 파티션(Partition) 명명 규칙 때문에, 순수한 DDS로만 작성된 애플리케이션과 ROS2 노드가 직접 통신하는 데 어려움을 겪을 수 있다.48

이는 ROS2 기반 시스템을 다른 산업용 DDS 시스템과 통합하려는 개발자가 반드시 이해해야 할 중요한 아키텍처적 함의다. 문제 해결을 위해서는 RMW 계층을 우회하여 네이티브 DDS 객체에 직접 접근하는 방법을 고려하거나, 양쪽 시스템이 모두 동의하는 특정 토픽 이름과 QoS 규칙을 사용하도록 구성해야 할 수 있다.48 RMW는 단순한 어댑터가 아니라, ROS 생태계의 철학을 강제하는 '필터'이자 '게이트웨이'이며, 이로 인한 호환성 문제는 버그가 아니라 더 높은 수준의 목표(개발자 경험, 벤더 독립성)를 위한 의도된 설계의 결과물로 해석해야 한다.

------


지금까지 ROS2 Humble의 메시징 시스템을 구성하는 핵심 요소들을 다각도에서 깊이 있게 살펴보았다. ROS2 메시징 시스템은 토픽, 서비스, 액션이라는 명확한 통신 패러다임, 커스텀 가능한 인터페이스, 강력한 QoS 제어, 그리고 DDS/RMW라는 견고한 아키텍처 기반 위에 구축되어 있다.

이러한 이해를 바탕으로, 더 안정적이고 효율적이며 확장 가능한 로봇 시스템을 설계하기 위한 몇 가지 제언을 하고자 한다.

1. **인터페이스를 먼저 설계하라:** 노드 코드를 작성하기 전에, 시스템의 노드들이 어떤 데이터를 주고받을지 먼저 고민하고, 이를 `_interfaces` 패키지에 명확히 정의하라. 이는 시스템의 API를 설계하는 것과 같다. 잘 정의된 인터페이스는 시스템의 구조를 명확하게 하고, 팀원 간의 협업을 원활하게 한다.
2. **통신 방식을 신중히 선택하라:** 데이터의 특성(지속성, 요청/응답, 장기 작업)뿐만 아니라, 노드 간의 결합도와 상태 관리 측면에서 토픽, 서비스, 액션을 전략적으로 선택하라. 잘못된 선택은 시스템을 불필요하게 복잡하게 만들거나 성능을 저하시킬 수 있다.
3. **QoS를 적극적으로 활용하되, 명시적으로 하라:** 모든 통신에는 의도가 있다. 데이터의 중요도와 네트워크 환경을 고려하여 `SensorData`나 `Default` 같은 사전 정의된 프로파일을 코드에 명시적으로 사용하라. RMW 구현체마다 기본값이 다를 수 있는 `SystemDefault`에 의존하는 것은 이식성을 해칠 수 있다.
4. **CLI는 당신의 가장 친한 친구다:** 시스템이 예상대로 동작하지 않을 때, 가장 먼저 `ros2 topic/service/action info -v`와 같은 CLI 도구를 사용하여 노드 간의 연결 상태와 QoS 설정을 확인하라. 대부분의 통신 문제는 이 단계에서 원인을 찾을 수 있다.
5. **DDS/RMW를 이해하면 디버깅이 쉬워진다:** 노드 간 연결이 안 되거나 메시지가 유실될 때, 문제의 원인이 애플리케이션 로직이 아닌 QoS 호환성이나 RMW의 동작 방식에 있을 수 있음을 인지하라. 이 근본적인 아키텍처에 대한 이해는 해결할 수 없는 것처럼 보였던 문제의 실마리를 제공할 것이다.

ROS2 메시징 시스템은 단순한 데이터 파이프라인이 아니다. 그것은 분산 로봇 시스템이라는 복잡한 문제를 해결하기 위한 정교한 철학과 아키텍처의 집약체다. 이 가이드가 그 깊이를 이해하고, 당신의 다음 로봇 프로젝트를 성공으로 이끄는 데 든든한 발판이 되기를 바란다.


1. ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/index.html
2. Decentralized Information-Flow Control for ROS2 - Network and Distributed System Security (NDSS) Symposium, accessed July 27, 2025, https://www.ndss-symposium.org/wp-content/uploads/2024-101-paper.pdf
3. ROS on DDS - ROS2 Design, accessed July 27, 2025, https://design.ros2.org/articles/ros_on_dds.html
4. Tutorials - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://ftp.udx.icscoe.jp/ros/ros_docs_mirror/en/humble/Tutorials.html
5. Beginner: CLI tools - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools.html
6. Tutorials - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials.html
7. Understanding topics - ROS 2 Documentation: Humble ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html
8. Topics - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Concepts/Basic/About-Topics.html
9. Quality of Service settings - ROS 2 Documentation: Humble ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Concepts/Intermediate/About-Quality-of-Service-Settings.html
10. Topics vs Services vs Actions - ROS 2 Documentation: Foxy ..., accessed July 27, 2025, https://docs.ros.org/en/foxy/How-To-Guides/Topics-Services-Actions.html
11. Topics vs. Services vs. Actions in ROS2-Based Projects - Automatic Addison, accessed July 27, 2025, https://automaticaddison.com/topics-vs-services-vs-actions-in-ros2-based-projects/
12. Built-in Message Type | Introduction to ROS2 and Robotics, accessed July 27, 2025, https://www.learnros2.com/ros/ros2-building-blocks/built-in-types/built-in-message-type
13. ROS2 in UR whitepaper - Documentation for Universal Robots, accessed July 27, 2025, https://docs.universal-robots.com/PolyScopeX_SDK_Documentation/build/SDK-v0.13/_downloads/305de110674c8345261f1ca772bb3c6f/ROS2inURwhitepaper.html
14. Understanding services - ROS 2 Documentation: Humble ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Services/Understanding-ROS2-Services.html
15. Understanding services - ROS 2 Documentation: Foxy documentation, accessed July 27, 2025, https://docs.ros.org/en/foxy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Services/Understanding-ROS2-Services.html
16. Writing a simple service and client (Python) - ROS 2 Documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Service-And-Client.html
17. Understanding actions - ROS 2 Documentation: Humble ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Actions/Understanding-ROS2-Actions.html
18. Actions - ROS2 Design, accessed July 27, 2025, https://design.ros2.org/articles/actions.html
19. 5.3.1.7. Understanding actions - Vulcanexus 1.0.0 documentation, accessed July 27, 2025, https://docs.vulcanexus.org/en/iron/ros2_documentation/source/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Actions/Understanding-ROS2-Actions.html
20. Writing an action server and client (Python) - ROS 2 ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Writing-an-Action-Server-Client/Py.html
21. ROS2 Basics for Python (Humble) Course - The Construct, accessed July 27, 2025, https://www.theconstruct.ai/robotigniteacademy_learnros/ros-courses-library/ros2-basics-python/
22. [ROS2 How-to] ROS2 Topics vs Services vs Actions #5 - The Construct, accessed July 27, 2025, https://www.theconstruct.ai/ros2-how-to-ros2-topics-vs-services-vs-actions-5/
23. common_interfaces/sensor_msgs/msg/Imu.msg at rolling - GitHub, accessed July 27, 2025, https://github.com/ros2/common_interfaces/blob/rolling/sensor_msgs/msg/Imu.msg
24. ROS Package: std_msgs, accessed July 27, 2025, https://index.ros.org/p/std_msgs/
25. Tutorial 2: Communications using topics - 240AR060 - Introduction to ROS, accessed July 27, 2025, https://sir.upc.edu/projects/ros2tutorials/2-communications_using_topics/index.html
26. ROS2 Create Custom Message (Msg/Srv) - The Robotics Back-End, accessed July 27, 2025, https://roboticsbackend.com/ros2-create-custom-message/
27. Creating custom msg and srv files - ROS 2 Documentation ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Custom-ROS2-Interfaces.html
28. Creating a dedicated package for custom interfaces - (Murilo's) ROS2 Tutorial, accessed July 27, 2025, https://ros2-tutorial.readthedocs.io/en/latest/create_interface_package.html
29. Creating an action - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Creating-an-Action.html
30. Expanding on ROS 2 interfaces - ROS documentation, accessed July 27, 2025, https://docs.ros.org/en/crystal/Tutorials/Single-Package-Define-And-Use-Interface.html
31. Robotics crash course (Part 13 - Creating a custom interface) - Jeremy Pedersen, accessed July 27, 2025, https://jeremypedersen.com/posts/2024-08-22-pt13-custom-interfaces/
32. ROS2 Tutorials #7: How to create a ROS2 Custom Message - The Construct, accessed July 27, 2025, https://www.theconstruct.ai/ros2-tutorials-7-how-to-create-a-ros2-custom-message-new/
33. Creating custom msg and srv. Python - ROS 2 course 0.1 documentation - Read the Docs, accessed July 27, 2025, [https://ros2course.readthedocs.io/en/latest/Creating%20custom%20msg%20and%20srv.%20Python.html](https://ros2course.readthedocs.io/en/latest/Creating custom msg and srv. Python.html)
34. ament_cmake user documentation - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/How-To-Guides/Ament-CMake-Documentation.html
35. About internal ROS 2 interfaces - ROS documentation, accessed July 27, 2025, https://docs.ros.org/en/dashing/Concepts/About-Internal-Interfaces.html
36. Introspection with command line tools - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Concepts/Basic/About-Command-Line-Tools.html
37. ros2/ros2cli: ROS 2 command line interface tools - GitHub, accessed July 27, 2025, https://github.com/ros2/ros2cli
38. ROS 2 Commands Cheat Sheet - The Robotics Space, accessed July 27, 2025, https://www.theroboticsspace.com/assets/article3/ros2_humble_cheat_sheet2.pdf
39. Writing a simple publisher and subscriber (Python) - ROS 2 ..., accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html
40. ROS2 Actions [1H Crash Course] - YouTube, accessed July 27, 2025, https://www.youtube.com/watch?v=X7YSnDbKMWo
41. Writing a simple service and client (C++) - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Cpp-Service-And-Client.html
42. Writing an action server and client (C++) - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Writing-an-Action-Server-Client/Cpp.html
43. ROS 2 Quality of Service policies - ROS2 Design, accessed July 27, 2025, https://design.ros2.org/articles/qos.html
44. Quality of Service (QoS) - Steven Gong, accessed July 27, 2025, https://stevengong.co/notes/Quality-of-Service
45. ROS 2 Quality of Service (QoS) - Isaac Sim Documentation - NVIDIA, accessed July 27, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.2.0/ros2_tutorials/tutorial_ros2_qos.html
46. Quality of Service | Introduction to ROS2 and Robotics, accessed July 27, 2025, https://www.learnros2.com/ros/ros2-building-blocks/quality-of-service
47. ROS 2 middleware interface - ROS2 Design, accessed July 27, 2025, https://design.ros2.org/articles/ros_middleware_interface.html
48. Ros2 and DDS messaging - ROS General - Open Robotics Discourse, accessed July 27, 2025, https://discourse.ros.org/t/ros2-and-dds-messaging/1556
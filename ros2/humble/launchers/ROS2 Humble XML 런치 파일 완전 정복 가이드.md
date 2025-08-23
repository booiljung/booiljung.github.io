# ROS2 Humble XML 런처 완벽 정복 가이드


로봇 시스템이 복잡해질수록 수많은 ROS 2 노드를 동시에 실행하고 관리해야 하는 과제에 직면한다. 각 노드를 별도의 터미널에서 `ros2 run` 명령어로 일일이 실행하는 것은 소규모 프로젝트에서는 가능할지 몰라도, 노드 개수가 수십 개에 이르고 각각의 파라미터, 리매핑, 네임스페이스 설정이 필요한 실제 로봇 애플리케이션에서는 비효율적이고 오류를 유발하기 쉽다.1 바로 이 지점에서 ROS 2 런치 시스템의 필요성이 대두된다.


ROS 2 런치 시스템은 여러 노드와 그 구성을 단일 명령어로 한 번에 실행하고 관리하기 위해 설계된 자동화 도구다.3 런치 시스템의 핵심 역할은 단순히 노드를 실행하는 것을 넘어, 전체 시스템의 구성을 체계적으로 '기술(describe)'하고, 기술된 내용에 따라 정확하게 '실행(execute)'하는 데 있다. 여기에는 어떤 프로그램을 어디서 실행할지, 어떤 인자를 전달할지, 그리고 프로세스의 상태를 모니터링하고 변화에 대응하는 것까지 포함된다.3 즉, 런치 파일은 복잡한 로봇 시스템의 청사진이자 실행 계획서 역할을 한다.


ROS 1이 XML 형식만 지원했던 것과 달리, ROS 2는 개발자의 필요와 선호에 따라 Python, XML, YAML 세 가지 형식의 런치 파일을 지원한다.5 각 포맷은 고유한 철학과 장단점을 가지고 있다.

- **XML (eXtensible Markup Language)**: 선언적(Declarative) 포맷으로, '무엇을' 실행할지를 명시적으로 기술한다. 태그 기반의 계층적 구조는 시스템의 구성을 명확하고 직관적으로 보여주어 가독성이 높다. 특히 시스템 구성이 대부분 정적인 경우에 강력한 장점을 보인다. ROS 1 사용자에게는 가장 익숙한 형식이며, ROS 2 공식 문서에서도 일반적인 사용 사례(typical use cases)에는 Python보다 XML이나 YAML을 권장한다.7

- **Python**: 프로그래밍적(Programmatic) 포맷으로, Python 언어가 제공하는 모든 기능을 활용하여 동적이고 복잡한 런치 로직을 구현할 수 있다. 런치 시점의 조건문, 반복문, 외부 파일 파싱, 라이브러리 연동 등 XML로는 불가능하거나 매우 복잡한 작업을 수행할 수 있다.5

  `launch_ros` API의 저수준 기능에 직접 접근할 수 있다는 점도 큰 장점이다.6

- **YAML (YAML Ain't Markup Language)**: XML과 마찬가지로 선언적 포맷이지만, 태그 대신 들여쓰기 기반의 문법을 사용하여 훨씬 간결하고 사람이 읽고 쓰기 편하다. 주로 파라미터 파일을 정의하는 데 널리 사용되지만, 간단한 노드 실행을 위한 런치 파일로도 활용될 수 있다.2

이러한 포맷의 차이는 런치 시스템의 근본적인 작동 방식에서 비롯된다. ROS 2 런치 파일은 직접 프로세스를 실행하는 스크립트가 아니다. 대신, 시스템의 구성을 기술하는 추상적인 데이터 구조인 **런치 디스크립션(Launch Description)**을 생성하는 역할을 한다.11

`ros2 launch` 명령어는 이렇게 생성된 런치 디스크립션을 입력받아, 이를 해석하고 실제 노드들을 실행하는 실행자(executor)로 작동한다.12

이 구조를 이해하면 왜 Python 런치 파일이 일반적인 Python 스크립트와 다르게 느껴지는지 알 수 있다. 예를 들어, Python의 `if` 문 대신 `IfCondition`과 같은 클래스를 사용하는 이유는, 실행 시점의 로직을 직접 수행하는 것이 아니라 '어떤 조건에서 무엇을 실행하라'는 **설명**을 런치 디스크립션에 추가하기 위함이다.11 따라서 Python 런치 파일은 실행 스크립트라기보다 런치 디스크립션을 구축하는 빌더(builder)에 가깝다. 이 관점에서 보면 XML은 단순히 오래되거나 제한적인 포맷이 아니라, 선언적 시스템 기술이라는 목적에 가장 충실한 일급 시민(first-class citizen)으로 볼 수 있다.


XML, Python, YAML 중 어떤 것을 선택할지는 프로젝트의 요구사항과 개발자의 선호에 따라 달라진다.

**XML이 유리한 경우**는 다음과 같다:

- 시스템 구성이 대부분 정적이며, 런치 시점에 복잡한 로직이 필요 없을 때.
- ROS 1에서 마이그레이션하는 프로젝트로, 기존 `.launch` 파일과의 유사성이 중요할 때.7
- 팀원들과의 협업에서 런치 파일의 명확한 구조와 높은 가독성을 최우선으로 고려할 때.8

**Python이 필요한 경우**는 다음과 같다:

- 런치 인자 값에 따라 실행되는 노드나 구성이 동적으로 변경되어야 할 때.
- IP 주소 형식 검사나 파일 존재 여부 확인 등 런치 인자에 대한 복잡한 유효성 검사가 필요할 때.9
- ROS 2의 컴포저블 노드(Composable Nodes)나 라이프사이클 노드(Lifecycle Nodes)와 같은 고급 기능을 정교하게 제어해야 할 때.15

대부분의 일반적인 애플리케이션(약 95%)에서는 XML만으로도 충분하며, Python보다 훨씬 빠르고 간결하게 작성할 수 있다는 것이 중론이다.16 만약 복잡성이 필요한 부분이 생긴다면, 해당 부분만 Python 런치 파일로 작성하고 메인 XML 런치 파일에서 `<include>` 태그를 사용해 포함시키는 현명한 조합도 가능하다.17

다음 표는 세 가지 런치 파일 포맷의 특징을 요약하여 보여준다.

| 기준                 | XML                             | Python                            | YAML                            |
| -------------------- | ------------------------------- | --------------------------------- | ------------------------------- |
| **포맷 유형**        | 선언적 (Declarative)            | 프로그래밍적 (Programmatic)       | 선언적 (Declarative)            |
| **가독성**           | 높음 (계층적 구조)              | 중간 (Python 문법 지식 필요)      | 매우 높음 (간결함)              |
| **유연성/동적 기능** | 제한적 (조건부 실행, 치환 등)   | 매우 높음 (언어의 모든 기능 사용) | 매우 제한적                     |
| **학습 곡선**        | 낮음 (특히 ROS 1 경험자)        | 높음 (런치 API 학습 필요)         | 매우 낮음                       |
| **생태계 지원**      | 우수 (ROS 1부터의 전통)         | 매우 우수 (모든 고급 기능 지원)   | 보통 (주로 파라미터용)          |
| **최적 사용처**      | 정적인 시스템 구성, 가독성 중시 | 동적인 시스템 구성, 복잡한 로직   | 파라미터 파일, 매우 간단한 런치 |


이론을 넘어 실제 코드를 작성하며 XML 런치 파일을 사용하는 전체 개발 워크플로우를 단계별로 익혀보자.


전문적인 ROS 개발에서는 노드 소스 코드와 런치 파일을 같은 패키지에 두지 않는 것이 모범 사례(Best Practice)로 꼽힌다. 이는 불필요한 의존성 결합을 막고 시스템의 모듈성과 재사용성을 높이기 위함이다.19 런치 파일, 설정 파일(YAML), URDF 등 시스템 실행과 관련된 파일들을 모아두는 전용 패키지를 만드는 것이 좋다. 관례적으로 이 패키지의 이름은 `<project_name>_bringup`으로 명명한다.20

다음 명령어를 통해 `ament_cmake` 타입의 `bringup` 패키지를 생성하고 초기 구조를 설정할 수 있다.

```Bash
# 워크스페이스의 src 디렉토리로 이동
cd ~/ros2_ws/src

# 런치 파일 전용 패키지 생성
ros2 pkg create --build-type ament_cmake my_robot_bringup

# 생성된 패키지 디렉토리로 이동
cd my_robot_bringup

# 불필요한 소스 및 헤더 디렉토리 삭제
rm -rf src/ include/

# 런치 파일을 저장할 디렉토리 생성
mkdir launch
```

19

이러한 "Bringup 패키지" 아키텍처는 단순히 파일을 정리하는 것을 넘어, 시스템의 **구현(Implementation)**을 **구성(Configuration)**으로부터 분리하는 중요한 설계 패턴이다. 예를 들어, `my_robot_nodes` 패키지는 순수하게 노드 코드에만 집중하고, `my_robot_sim_bringup`과 `my_robot_real_bringup`이라는 별도의 패키지들이 각각의 상황에 맞게 `my_robot_nodes`의 노드들을 조합하여 실행하는 방식이다. 이는 대규모 프로젝트의 유지보수성을 극대화하는 전문가 수준의 접근 방식이므로 처음부터 따르는 것이 좋다.20


앞서 생성한 `launch/` 디렉토리 안에 `my_first.launch.xml` 파일을 생성한다. 파일 확장자는 `.launch.xml`을 사용하는 것이 관례다.19

가장 기본적인 XML 런치 파일의 구조는 다음과 같다.

```XML
<?xml version="1.0" encoding="UTF-8"?>
<launch>
    <node pkg="demo_nodes_cpp" exec="talker" name="my_talker"/>
    <node pkg="demo_nodes_cpp" exec="listener" name="my_listener"/>
</launch>
```

7

- `<?xml...?>`: 이 파일이 XML 문서임을 선언하는 부분이다.
- `<launch>`: 런치 파일의 루트(root) 요소로, 모든 다른 태그들은 이 태그 내부에 위치해야 한다.
- `<node>`: ROS 2 노드를 실행하는 가장 핵심적인 태그다.
  - `pkg`: 실행할 노드가 포함된 패키지의 이름을 지정한다.
  - `exec`: `colcon`으로 빌드했을 때 생성되는 실행 파일의 이름을 지정한다.
  - `name`: 실행되는 노드에 고유한 이름을 부여한다 (선택 사항). 지정하지 않으면 노드 코드에 정의된 기본 이름이 사용된다.


런치 파일을 `ros2 launch` 명령어로 찾아서 실행하려면, 빌드 시스템에 해당 파일의 존재를 알려주어야 한다.

- **`CMakeLists.txt` 수정**: `colcon build` 시 `launch/` 디렉토리와 그 안의 모든 파일들을 `install/` 디렉토리로 복사하도록 `CMakeLists.txt` 파일에 다음 구문을 추가한다. 이 구문은 `ament_package()` 호출 전에 위치해야 한다.

  ```CMake
  install(
    DIRECTORY launch
    DESTINATION share/${PROJECT_NAME}
  )
  ```

  20

- **`package.xml` 수정**: 런치 파일이 실행하는 노드들이 다른 패키지에 있다면, 해당 패키지에 대한 실행 시간 의존성(`exec_depend`)을 `package.xml`에 추가해야 한다. 이는 `colcon`이 패키지들의 빌드 순서를 올바르게 결정하는 데 도움을 준다.

  ```XML
  <exec_depend>demo_nodes_cpp</exec_depend>
  <exec_depend>demo_nodes_py</exec_depend>
  ```

  19


이제 모든 준비가 끝났다. 다음 단계를 통해 런치 파일을 실행하고 결과를 확인할 수 있다.

1. 워크스페이스의 최상위 디렉토리로 이동하여 `colcon`으로 `bringup` 패키지를 빌드한다.

   ```Bash
   cd ~/ros2_ws
   colcon build --packages-select my_robot_bringup
   ```

2. 새 터미널을 열고 빌드된 워크스페이스의 환경을 설정(source)한다.

   ```Bash
   source install/setup.bash
   ```

3. `ros2 launch` 명령어를 사용하여 런치 파일을 실행한다.

   ```Bash
   ros2 launch my_robot_bringup my_first.launch.xml
   ```

   6

4. 실행 후, 다른 터미널에서 `ros2 node list` 명령어를 입력하면 `/my_talker`와 `/my_listener` 노드가 목록에 나타나는 것을 확인할 수 있다.


XML 런치 파일의 강력함은 다양한 태그를 조합하여 노드의 동작을 세밀하게 제어하는 데 있다. 각 핵심 태그의 속성과 사용법을 ROS 1과의 비교를 통해 심층적으로 분석해 본다. 이러한 문법적 차이는 단순한 이름 변경을 넘어, ROS 2의 핵심 설계 원칙인 캡슐화, 명확성, 탈중앙화를 반영하고 있다.


`<node>` 태그는 지정된 패키지의 실행 파일을 새로운 프로세스로 실행하여 ROS 2 노드를 구동시킨다.

- **주요 속성** 8:
  - `pkg` (필수): 노드가 속한 패키지 이름.
  - `exec` (필수): 실행 파일 이름. ROS 1의 `type` 속성이 `exec`으로 변경되었다.7
  - `name` (선택): 노드에 부여할 이름.
  - `namespace` (선택): 노드가 속할 네임스페이스. ROS 1의 `ns` 속성이 `namespace`로 변경되었다.7
  - `output` (선택): 노드의 표준 출력(stdout/stderr)을 어디로 보낼지 결정한다. `screen`으로 설정하면 런치 파일을 실행한 터미널에 로그가 직접 표시되어 디버깅에 매우 유용하다. 기본값은 `log`로, 로그 파일에만 기록된다.8
  - **지원 중단된 속성**: ROS 1의 `respawn`, `respawn_delay`, `machine` 등은 ROS 2에서 직접 지원되지 않는다. 노드 재시작과 같은 기능은 라이프사이클 노드(Lifecycle Nodes)와 같은 더 정교한 관리 메커니즘을 통해 다루어진다.


`<param>` 태그는 노드 실행 시 파라미터 값을 설정하는 데 사용된다.

- **핵심 변경점**: ROS 2에는 더 이상 전역(global) 파라미터 서버가 없다. 모든 파라미터는 특정 노드에 귀속(node-local)된다. 이 설계 철학의 변화로 인해, **`<param>` 태그는 반드시 `<node>` 태그 내부에 중첩되어야 한다**.7 ROS 1처럼 `<node>` 태그 밖에 선언하면 오류가 발생한다.

- **기본 사용법**: `<param name="param_name" value="param_value"/>` 형식으로 사용한다.16

- **값의 타입 추론**: `value` 속성의 값은 특정 규칙에 따라 타입이 추론된다. `value="1.0"`은 부동소수점, `value="10"`은 정수, `value="true"`는 불리언으로 인식된다. 문자열임을 명시하려면 `value="'hello'"`와 같이 작은따옴표로 감싸는 것이 안전할 수 있다.7

- **파라미터 그룹화**: `<param>` 태그를 중첩하여 계층적인 파라미터 구조를 만들 수 있다. 예를 들어, `<param name="group1"><param name="my_param" value="1"/></param>`은 코드에서 `group1.my_param`으로 접근할 수 있다.7

- **파일에서 로드**: `<param from="/path/to/params.yaml"/>` 구문을 사용하면 YAML 파일에 정의된 수많은 파라미터를 한 번에 로드할 수 있어, 복잡한 파라미터 관리에 매우 효율적이다.8


`<remap>` 태그는 노드가 사용하는 토픽, 서비스, 액션 등의 이름을 동적으로 변경(리매핑)하여 노드 간의 연결을 유연하게 만든다.

- **역할**: 코드에 하드코딩된 이름을 바꾸지 않고도, 런치 시점에서 통신 경로를 자유롭게 바꿀 수 있게 해준다.25
- **사용법**: `<remap from="original_name" to="new_name"/>` 형식으로 사용하며, `<param>` 태그와 마찬가지로 반드시 `<node>` 태그 내부에 위치해야 한다.7
- **예시**: `turtlesim` 예제에서 `mimic` 노드는 기본적으로 `/input/pose` 토픽을 구독하고 `/output/cmd_vel` 토픽을 발행한다. 이를 `<remap from="/input/pose" to="/turtle1/pose"/>`와 같이 리매핑하여 특정 거북이의 위치를 구독하도록 만들 수 있다.6


`<arg>` 태그는 런치 파일 내부에서 사용할 변수를 선언하고, 이 변수 값을 커맨드라인에서 덮어쓸 수 있게 하여 런치 파일의 재사용성과 유연성을 크게 향상시킨다.

- **역할**: 런치 파일의 동작을 외부에서 제어하기 위한 인터페이스를 제공한다.
- **속성** 8:
  - `name` (필수): 인수의 이름.
  - `default` (선택): 기본값. 커맨드라인에서 값이 주어지지 않으면 이 값이 사용된다.
  - `description` (선택): 인수에 대한 설명. ROS 1의 `doc` 속성이 `description`으로 변경되었다.7
- **ROS 1과의 차이점**:
  - 고정값을 설정하던 `value` 속성이 사라졌다. 런치 파일 내부에서만 사용하는 변수는 `<let>` 태그를 사용하도록 역할이 분리되었다.7
- **값 사용하기**: `<arg>`로 선언된 변수의 값은 `$(var arg_name)` 치환(substitution) 구문을 통해 런치 파일 내 다른 태그의 속성값으로 전달할 수 있다.6
- **커맨드라인에서 값 전달**: `ros2 launch my_package my_launch.xml my_arg:=new_value`와 같이 `key:=value` 형식으로 값을 전달하여 `default` 값을 덮어쓸 수 있다.6

다음 표는 ROS 1과 ROS 2의 XML 런치 파일에서 주요 태그의 변경점을 요약한 것이다.

| 태그        | ROS 1                            | ROS 2                                            | 주요 변경점 및 이유                                          |
| ----------- | -------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| `<node>`    | `<node pkg="..." type="..."/>`   | `<node pkg="..." exec="..."/>`                   | `type` -> `exec`. 실행 파일임을 명확히 함.                   |
| `<node>`    | `ns="..."`                       | `namespace="..."`                                | `ns` -> `namespace`. 약어 대신 완전한 단어 사용.             |
| `<param>`   | 전역/노드 내부 선언 가능         | `<node>` 내부에서만 선언 가능                    | 탈중앙화 설계. 전역 파라미터 서버를 제거하여 노드의 캡슐화 강화. |
| `<arg>`     | `<arg name="..." value="..."/>`  | `<arg name="..." default="..."/>`                | `value` 속성 제거. 내부 변수(`let`)와 외부 인자(`arg`)의 역할을 명확히 분리. |
| `<include>` | `<include file="..." ns="..."/>` | `<include file="..."/>` + `<push_ros_namespace>` | `ns` 속성 제거. 네임스페이스 적용을 명시적인 액션으로 처리하여 가독성 향상. |


단일 런치 파일의 한계를 넘어, 여러 런치 파일을 조합하고 고급 태그들을 활용하여 복잡하고 유지보수하기 쉬운 대규모 시스템을 구축하는 방법을 알아본다. 모듈화와 계층적 설계가 핵심이다.


`<include>` 태그는 다른 런치 파일을 현재 런치 파일에 포함시키는 기능으로, 복잡한 시스템을 기능 단위로 분리하여 관리할 수 있게 해주는 가장 중요한 도구다.8

- **핵심 기능**: `<include>`를 사용하면, 예를 들어 '카메라 시스템'과 관련된 모든 노드(드라이버, 이미지 처리 등)를 `camera.launch.xml`로, 'LIDAR 시스템'은 `lidar.launch.xml`로 분리할 수 있다. 그리고 최상위 런치 파일인 `robot.launch.xml`에서는 이 모듈들을 단순히 `<include>` 하기만 하면 된다. 이는 유지보수성을 극적으로 향상시킨다.22
- **형식의 유연성**: XML 런치 파일에서 Python이나 YAML 형식의 런치 파일을 포함하는 것이 가능하다. 이는 기존에 Python으로 작성된 런치 파일을 수정 없이 재사용하면서 새로운 시스템을 가독성 좋은 XML로 구성할 수 있는 강력한 유연성을 제공한다.6
- **인자 전달**: `<include>` 태그 내부에 `<arg>` 태그를 사용하여 포함되는 런치 파일에 파라미터를 전달할 수 있다. 이를 통해 모듈화된 런치 파일의 동작을 외부에서 제어할 수 있다.6

이러한 `<include>`와 `<arg>`의 조합은 단순한 파일 포함을 넘어, 소프트웨어 공학의 '의존성 주입(Dependency Injection)'과 유사한 패턴을 구현하게 해준다. 예를 들어, 시뮬레이션용 `sim_camera.launch.xml`과 실제 로봇용 `real_camera.launch.xml`을 만들어두고, 최상위 런치 파일에서는 외부에서 받은 `use_sim_time` 같은 인자에 따라 조건부로 둘 중 하나를 포함하도록 구성할 수 있다. 이는 시스템의 특정 부분을 쉽게 교체(swap out)할 수 있게 만들어, 시뮬레이션과 실제 로봇 간의 전환 비용을 극적으로 낮추고 시스템의 확장성을 보장하는 고급 아키텍처 기법이다.


`<group>` 태그는 여러 액션(`node`, `include` 등)을 논리적으로 묶는 역할을 하며, `<push_ros_namespace>`와 함께 사용될 때 강력한 계층 구조 설계 도구가 된다.

- **ROS 1과의 결정적 차이**: ROS 1에서는 `<group>`이나 `<include>` 태그에 `ns` 속성을 직접 부여하여 네임스페이스를 적용했다. 하지만 ROS 2에서는 이 `ns` 속성이 제거되었다.7
- **명시적 액션**: ROS 2에서는 `<push_ros_namespace>`라는 별도의 태그를 `<group>` 내부에 사용해야만 해당 그룹 내의 모든 액션에 네임스페이스가 적용된다. 이는 네임스페이스 적용을 숨겨진 속성이 아닌 명시적인 '액션'으로 처리하여 런치 파일의 동작을 더 명확하게 파악할 수 있게 하려는 설계 의도가 담겨있다.6

```XML
<group>
    <push_ros_namespace namespace="robot1"/>
    <include file="$(find-pkg-share my_robot_driver)/launch/driver.launch.xml"/>
    <include file="$(find-pkg-share my_robot_nav)/launch/navigation.launch.xml"/>
</group>
```

위 예시는 `driver`와 `navigation` 런치 파일에 포함된 모든 노드들을 `robot1`이라는 네임스페이스 하위에 배치하여, 다른 로봇과의 이름 충돌을 방지한다.6


ROS 2는 ROS 1에는 없던 새로운 태그들을 도입하여 런치 파일의 표현력을 확장했다.

- `<let>`: 런치 파일 내에서만 사용되는 지역 변수를 선언한다. `value` 속성을 가진 ROS 1의 `<arg>`를 대체하며, '외부에서 주입받는 인자(`arg`)'와 '내부에서 정의하는 변수(`let`)'의 역할을 명확히 구분한다.7
- `<executable>`: ROS 노드가 아닌 일반적인 실행 파일(예: `ls`, `bash` 스크립트, 데이터 로깅 프로그램 등)을 실행할 수 있게 해준다. `cmd`, `cwd`(작업 디렉토리) 등의 속성을 가진다.7
- `<env>`, `<set_env>`, `<unset_env>`: 환경 변수를 제어하는 태그들이다.
  - `<env>`: `<node>`나 `<executable>` 태그 내부에 사용하여 해당 프로세스에만 적용될 환경 변수를 설정한다.7
  - `<set_env>`, `<unset_env>`: 런치 파일의 전역 범위나 `<group>` 범위에서 환경 변수를 설정하거나 해제한다. 조건부 실행(`if`/`unless`)이 가능하여 더욱 유연한 제어를 제공한다.7


치환은 런치 파일이 실행되는 시점에 동적으로 값을 결정하게 해주는 강력한 기능이다.

- `$(var arg_name)`: `<arg>` 태그로 선언된 변수의 값을 가져온다. (ROS 1의 `$(arg...)` 대체).7
- `$(find-pkg-share pkg_name)`: 패키지의 `install/<pkg_name>/share` 디렉토리 경로를 찾아준다. 설정 파일이나 다른 런치 파일을 참조할 때 절대 경로를 하드코딩하지 않도록 해주는 필수 기능이다. (ROS 1의 `$(find...)` 대체).6
- `$(env VAR_NAME)`: 환경 변수 값을 가져온다.7
- `$(eval '...')`: Python 표현식을 평가하여 그 결과를 사용한다. 간단한 조건부 로직을 구현할 때 유용하지만, 복잡한 로직은 Python 런치 파일을 사용하는 것이 좋다. 문자열 비교 시 `if="$(eval '\'$(var my_str_arg)\' == \'some_value\'')"`와 같이 이스케이프 문자(`\'`) 사용에 주의해야 한다.7


다음은 앞서 설명한 모든 고급 태그를 활용한 복잡한 런치 파일의 예제다.6

```XML
<?xml version="1.0" encoding="UTF-8"?>
<launch>
  <arg name="background_r" default="0" description="Red background for turtlesim2"/>
  <arg name="background_g" default="84" description="Green background for turtlesim2"/>
  <arg name="background_b" default="140" description="Blue background for turtlesim2"/>
  <arg name="chatter_ns" default="my_chatter_ns" description="Namespace for included nodes"/>

  <include file="$(find-pkg-share demo_nodes_cpp)/launch/topics/talker_listener.launch.py" />

  <group>
    <push_ros_namespace namespace="$(var chatter_ns)" />
    <include file="$(find-pkg-share demo_nodes_cpp)/launch/topics/talker_listener.launch.py" />
  </group>

  <node pkg="turtlesim" exec="turtlesim_node" name="sim" namespace="turtlesim1" />

  <node pkg="turtlesim" exec="turtlesim_node" name="sim" namespace="turtlesim2">
    <param name="background_r" value="$(var background_r)" />
    <param name="background_g" value="$(var background_g)" />
    <param name="background_b" value="$(var background_b)" />
  </node>

  <node pkg="turtlesim" exec="mimic" name="mimic">
    <remap from="/input/pose" to="/turtlesim1/turtle1/pose" />
    <remap from="/output/cmd_vel" to="/turtlesim2/turtle1/cmd_vel" />
  </node>
</launch>
```

이 런치 파일은 `talker`/`listener` 예제를 두 번(한 번은 전역, 한 번은 특정 네임스페이스) 실행하고, 두 개의 `turtlesim` 창을 띄운다. 두 번째 창의 배경색은 외부에서 인자로 제어할 수 있으며, `mimic` 노드가 리매핑을 통해 첫 번째 거북이를 따라다니도록 설정되어 있다. 이는 ROS 2 XML 런치 파일의 유연성과 강력함을 잘 보여주는 사례다.


개발 과정에서 마주치는 문제는 피할 수 없다. 효과적인 디버깅을 위해서는 문제의 발생 지점(런치 시점 vs. 실행 시점)과 종류(문법 오류 vs. 런타임 충돌 vs. 로직 오류)를 먼저 판별하고, 그에 맞는 도구를 사용하는 체계적인 접근이 필요하다.


- **XML 문법 오류**: 태그가 제대로 닫히지 않았거나(`</node>`), 필수 속성이 누락된 경우(`pkg` 또는 `exec`). `ros2 launch` 실행 시 "malformed XML"과 같은 파서 에러가 발생한다. 이 경우, 에러 메시지가 가리키는 줄 번호를 확인하여 수정하면 된다.26
- **"Executable... not found"**: `pkg`나 `exec` 속성의 이름이 잘못되었거나, 해당 패키지가 `colcon build`로 빌드되지 않았거나, `source install/setup.bash` 명령어로 환경 설정이 제대로 되지 않은 경우에 발생한다. `ros2 pkg executables <pkg_name>` 명령어로 패키지 내의 사용 가능한 실행 파일 목록을 확인해볼 수 있다.10
- **"InvalidLaunchFileError"**: `<include>`된 파일의 경로가 잘못되었거나, 포함된 런치 파일 자체에 문제가 있을 때 발생한다. ROS 2의 초기 버전에서는 오류가 발생한 파일이 어떤 것인지 명확히 알려주지 않아 디버깅이 어려웠으나, Humble 이후 버전에서는 개선되어 문제의 파일 경로를 알려주는 경우가 많아졌다.26
- **ROS 1 문법 사용**: ROS 1에 익숙한 사용자들이 `type` 속성을 사용하거나 `<param>`을 `<node>` 태그 밖에 선언하는 실수를 저지르곤 한다. 이는 ROS 2의 변경된 스키마와 맞지 않아 오류를 발생시킨다.7


런치 파일 실행 시 문제가 발생했을 때 가장 먼저 시도해야 할 방법은 `ros2 launch` 명령어의 디버깅 옵션을 사용하는 것이다.

- `ros2 launch... -d` 또는 `ros2 launch... --debug`: 런치 시스템의 디버그 로그를 활성화한다.

- 만약 실행 시 `[launch]: Caught exception in launch (see debug for traceback)`와 같은 메시지가 출력된다면, `-d` 옵션을 붙여 다시 실행해야 한다. 그러면 숨겨져 있던 상세한 Python 트레이스백(traceback)이 출력되어, 어떤 파일의 어떤 부분에서 예외가 발생했는지 추적하는 데 결정적인 단서를 제공한다.27

  `-a` 옵션을 함께 사용하면 모든 노드의 디버그 출력을 볼 수 있다.


런치 파일 자체의 문제가 아니라, 실행된 C++ 노드가 세그멘테이션 폴트(segmentation fault) 등으로 비정상 종료될 때 GDB(GNU Debugger)를 연동하여 심층 분석을 할 수 있다.

1. **디버그 빌드**: 먼저, 디버깅하려는 패키지를 디버그 정보와 함께 빌드해야 한다.

   ```Bash
   colcon build --packages-select <pkg_name> --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo
   ```

   28

2. **`launch-prefix` 사용**: 디버깅할 노드의 `<node>` 태그에 `launch-prefix` 속성을 추가한다.

   ```XML
   <node pkg="my_pkg" exec="my_node" name="crasher"
         launch-prefix="gdbserver localhost:3000" output="screen"/>
   ```

   28

   

   이 설정은 my_node를 실행하기 전에 gdbserver를 localhost의 3000번 포트에서 실행하라는 의미다.

3. **GDB 연결**: 런치 파일을 실행한 뒤, 다른 터미널에서 GDB를 실행하고 원격 서버에 연결한다.

   ```Bash
   gdb
   (gdb) target remote localhost:3000
   ```

   28

   연결이 성공하면 continue 명령으로 프로그램을 진행시키고, 크래시가 발생했을 때 bt(backtrace) 명령으로 콜 스택을 확인하여 문제의 원인이 된 코드 위치를 찾을 수 있다.

   **대안**: `launch-prefix="xterm -e gdb -ex run --args"`를 사용하면 런치 시 새로운 xterm 터미널 창에 GDB가 바로 실행되어 더 편리하게 디버깅할 수 있다.28


VS Code와 같은 현대적인 IDE를 사용하면 디버깅 과정을 더욱 효율적으로 만들 수 있다.

- **`--symlink-install` 옵션**: 워크스페이스를 빌드할 때 `colcon build --symlink-install` 옵션을 사용하면, `install` 디렉토리에 파일이 복사되는 대신 원본 소스 파일에 대한 심볼릭 링크가 생성된다. 이를 통해 IDE에서 소스 코드를 수정하고 저장하면, 별도의 빌드 과정 없이도 다음 런치 실행 시 변경 사항이 즉시 반영된다.30
- **`launch.json` 설정**: VS Code의 "Run and Debug" 기능을 사용하여 GDB 디버깅을 자동화할 수 있다. `launch.json` 파일에 `gdbserver`에 연결하는 'attach' 타입의 설정을 추가하면, 버튼 클릭 한 번으로 실행 중인 노드에 디버거를 붙일 수 있다.31


지금까지 다룬 내용을 바탕으로, 실제 프로젝트에 적용할 수 있는 구체적인 전략과 최종적인 조언을 제시한다.


- **단순 프로토타입**: 노드 수가 적고 구성이 간단한 초기 프로토타입 단계에서는 단일 XML 런치 파일로 모든 것을 관리하는 것이 빠르고 효율적이다.
- **단일 로봇 시스템**: 로봇 시스템이 복잡해지면 기능별(센서, 액추에이터, 인식, 내비게이션 등)로 런치 파일을 모듈화하고, 최상위 런치 파일에서는 이 모듈들을 `<include>`하는 계층적 구조를 채택하는 것이 바람직하다.22
- **다중 로봇 또는 시뮬레이션/실제 병행**: `_bringup` 패키지를 분리하고, `<arg>`와 조건부 실행(`if`/`unless`)을 활용하여 환경에 따라 다른 구성을 로드하는 유연한 구조를 설계해야 한다. 이는 코드 중복을 최소화하고 유지보수성을 극대화한다.


모든 것을 하나의 포맷으로 통일할 필요는 없다. 두 형식의 장점을 모두 취하는 하이브리드 접근 방식이 가장 실용적일 수 있다.

1. 시스템의 기본 골격과 정적인 부분은 가독성이 뛰어난 **XML**로 구성한다.
2. 동적인 로직이나 복잡한 연산이 필요한 특정 부분만 별도의 **Python** 런치 파일로 작성한다.
3. 메인 XML 런치 파일에서 해당 Python 런치 파일을 `<include>`하여 전체 시스템을 완성한다.17

이러한 접근법은 전체 시스템의 가독성을 유지하면서도 필요한 부분에 Python의 유연성을 더할 수 있는 최적의 절충안이다.


ROS 2 런치 시스템은 계속 발전하고 있으므로, 공식 문서를 통해 최신 정보를 확인하는 것이 중요하다.

- **ROS 2 공식 튜토리얼**: Launch System에 대한 기본 개념과 실습을 제공한다.21
- **ROS 1 마이그레이션 가이드**: ROS 1과 2의 차이점을 상세히 설명하여 마이그레이션 시 발생하는 실수를 줄여준다.7
- **ROS 2 디자인 문서 (`roslaunch_xml.html`)**: XML 스키마의 원본 정의와 설계 철학을 담고 있어, 가장 근본적인 정보를 제공한다.8


ROS 2 Humble에서 XML 런치 파일은 결코 구시대의 유물이 아니다. 오히려 선언적인 특성을 통해 시스템의 구조를 명확하게 표현하고, 팀원 간의 협업과 장기적인 유지보수를 용이하게 만드는 강력하고 실용적인 도구다. Python의 무한한 유연성이 필요한 특정 고급 사용 사례를 제외하고, 대부분의 로봇 애플리케이션에서는 XML이 제공하는 명료함과 안정성이 더 큰 가치를 가질 수 있다. 이 가이드에서 다룬 기본 원칙과 고급 기법, 디버깅 방법론을 숙지한다면, 어떤 복잡한 로봇 시스템이라도 자신 있게 구성하고 관리할 수 있는 역량을 갖추게 될 것이다.


1. How to Use ROS 2 Launch Files - Foxglove, accessed July 27, 2025, https://foxglove.dev/blog/how-to-use-ros2-launch-files
2. ROS2 Launch Files - Dave's RoboShack, accessed July 27, 2025, https://davesroboshack.com/the-robot-operating-system-ros/ros2-launch-files/
3. Launch - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Concepts/Basic/About-Launch.html
4. Creating a launch file - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Launch/Creating-Launch-Files.html
5. Using Python, XML, and YAML for ROS 2 Launch Files, accessed July 27, 2025, https://docs.ros.org/en/foxy/How-To-Guides/Launch-file-different-formats.html
6. Using XML, YAML, and Python for ROS 2 Launch Files - ROS 2 ..., accessed July 27, 2025, https://docs.ros.org/en/humble/How-To-Guides/Launch-file-different-formats.html
7. Migrating Launch Files - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/How-To-Guides/Migrating-from-ROS1/Migrating-Launch-Files.html
8. ROS 2 Launch XML Format - ROS2 Design, accessed July 27, 2025, https://design.ros2.org/articles/roslaunch_xml.html
9. Input validation for XML launch files / Issue #731 / ros2/launch - GitHub, accessed July 27, 2025, https://github.com/ros2/launch/issues/731
10. ROS2 Part 9 - ROS2 Launch Files - RoboticsUnveiled, accessed July 27, 2025, https://www.roboticsunveiled.com/ros2-launch-files/
11. Why does the ROS2 Python launch files look like XML written in Python? - ROS Discourse, accessed July 27, 2025, https://discourse.openrobotics.org/t/why-does-the-ros2-python-launch-files-look-like-xml-written-in-python/23679
12. Why does the ROS2 Python launch files look like XML written in Python? - ROS Discourse, accessed July 27, 2025, https://discourse.ros.org/t/why-does-the-ros2-python-launch-files-look-like-xml-written-in-python/23679
13. ROS 2 Launch System - ROS2 Design, accessed July 27, 2025, https://design.ros2.org/articles/roslaunch.html
14. Which ROS 2 launch file format do you prefer Python or XML and why? - Reddit, accessed July 27, 2025, https://www.reddit.com/r/ROS/comments/1j6cu02/which_ros_2_launch_file_format_do_you_prefer/
15. ROS 2 launch: Python vs XML - Robotics Stack Exchange, accessed July 27, 2025, https://robotics.stackexchange.com/questions/96149/ros-2-launch-python-vs-xml
16. ROS2 - Create a Launch File with XML - YouTube, accessed July 27, 2025, https://www.youtube.com/watch?v=Le1vx1_KUDQ
17. ROS2 - Include a Launch File in Another Launch File (Python, XML, YAML) - YouTube, accessed July 27, 2025, https://www.youtube.com/watch?v=sl0exwcg3o8&pp=0gcJCfwAo7VqN5tD
18. How to create ros2 XML launch files - The Construct, accessed July 27, 2025, https://www.theconstruct.ai/how-to-create-ros2-xml-launch-files/
19. ROS2 XML Launch File Example - The Robotics Back-End, accessed July 27, 2025, https://roboticsbackend.com/ros2-xml-launch-file/
20. ros2-examples/launch_files.md at humble - GitHub, accessed July 27, 2025, https://github.com/IntelligentSystemsLabUTV/ros2-examples/blob/humble/launch_files.md
21. Integrating launch files into ROS 2 packages, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Launch/Launch-system.html
22. Using ROS 2 launch for large projects - daobook 0.0.1 文档, accessed July 27, 2025, https://daobook.github.io/ros2-docs/galactic/Tutorials/Launch-Files/Using-ROS2-Launch-For-Large-Projects.html
23. ROS2 Python Launch File Example - How to Start All Your Nodes at Once, accessed July 27, 2025, https://roboticsbackend.com/ros2-launch-file-example/
24. ROS2: Adding parameters to YAML launch file - Robotics Stack Exchange, accessed July 27, 2025, https://robotics.stackexchange.com/questions/105910/ros2-adding-parameters-to-yaml-launch-file
25. Creating a launch file - ROS 2 Documentation: Foxy 文档, accessed July 27, 2025, https://daobook.github.io/ros2-docs/foxy/Tutorials/Launch-Files/Creating-Launch-Files.html
26. On error, ros2 launch command should point to the problematic launch file #637 - GitHub, accessed July 27, 2025, https://github.com/ros2/launch/issues/637
27. ROS2 Humble Launch - How to Get Log Level to Output Launch File ..., accessed July 27, 2025, https://robotics.stackexchange.com/questions/112612/ros2-humble-launch-how-to-get-log-level-to-output-launch-file-error-traceback
28. Using GDB with ROS2, a reference - Juraph, accessed July 27, 2025, https://juraph.com/miscellaneous/ros2_and_gdb/
29. Debugging - ROS2_Control: Humble Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/humble/doc/ros2_control/doc/debugging.html
30. IDEs and Debugging [community-contributed] - ROS 2 Documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/How-To-Guides/ROS-2-IDEs.html
31. Debug ROS2 C++ node on VSCode (Ubuntu) - GitHub Gist, accessed July 27, 2025, https://gist.github.com/JADC362/a4425c2d05cdaadaaa71b697b674425f
32. Debug ROS 2 C++ Node with Breakpoint in VS Code by Running Node or Launch File (WSL and Ubuntu) - Reddit, accessed July 27, 2025, https://www.reddit.com/r/ROS/comments/1ghp2ta/debug_ros_2_c_node_with_breakpoint_in_vs_code_by/
33. ROS2: Including xml launch files in a python launch file - Robotics Stack Exchange, accessed July 27, 2025, https://robotics.stackexchange.com/questions/102348/ros2-including-xml-launch-files-in-a-python-launch-file
34. Launch - ROS 2 Documentation: Humble documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Launch/Launch-Main.html


---
layout: page
title: ROS 2 Humble 파이썬 런치 파일 작성 완벽 가이드
permalink: /ros2/humble/launchers/ROS2 Humble 파이썬 런치 파일 작성 완벽 가이드
---


이 섹션에서는 ROS 2 런치 시스템의 근본적인 목적을 확립하고, 사용 가능한 형식들을 비교하며, 궁극적으로 이 보고서가 왜 파이썬 기반 접근 방식에 초점을 맞추는지 설명한다.


전형적인 ROS 2 시스템은 다수의 프로세스, 심지어 여러 머신에 걸쳐 실행되는 수많은 노드로 구성된다. 로봇 애플리케이션이 복잡해질수록 센서, 액추에이터, 인식, 계획 등 다양한 기능을 담당하는 노드의 수는 기하급수적으로 증가한다. 이러한 각 노드를 터미널에서 수동으로 하나씩 실행하고 필요한 파라미터를 설정하는 작업은 매우 지루하고 실수를 유발하기 쉽다.

런치 시스템은 이러한 복잡성을 관리하기 위한 핵심 도구다. 런치 파일은 시스템 구성을 기술하는 파일로, 어떤 프로그램을 실행할지, 어디서 실행할지, 어떤 인자를 전달할지 등을 명시한다. 단일 `ros2 launch` 명령어 하나로 전체 시스템, 즉 모든 노드와 그 설정을 한 번에 구동할 수 있게 함으로써, 개발자는 반복적이고 예측 가능한 방식으로 복잡한 로봇 시스템을 안정적으로 시작할 수 있다.


ROS 2는 파이썬(Python), XML, YAML의 세 가지 형식으로 런치 파일을 작성할 수 있도록 지원한다. 각 형식은 고유한 장단점을 가지며, 개발자는 프로젝트의 요구사항에 따라 적절한 형식을 선택해야 한다.

- **XML (eXtensible Markup Language):** ROS 1의 `roslaunch` 시스템에서 사용되던 표준 형식이었기 때문에, ROS 1에서 마이그레이션하는 사용자에게 가장 친숙할 수 있다. XML은 선언적(declarative) 접근 방식을 취하며, 정적인 구성을 기술할 때 "보는 것이 얻는 것(WYSIWYG)"과 유사한 명료함을 제공한다.
- **YAML (YAML Ain't Markup Language):** YAML 역시 선언적 형식으로, XML보다 더 간결하고 사람이 읽기 쉬운 문법을 제공한다는 장점이 있다. 간단한 노드 실행과 파라미터 설정에 적합하다.
- **Python:** 파이썬은 앞선 두 형식과 달리 완전한 프로그래밍 언어다. 이는 런치 파일 내에서 조건문, 반복문, 함수, 외부 라이브러리 활용 등 강력한 프로그래밍 로직을 사용할 수 있음을 의미한다. 커뮤니티의 논의를 보면, 단순한 구성에는 XML의 간결함을 선호하는 의견도 있지만, 동적인 로직이 필요할 때는 파이썬이 압도적인 지지를 받는다.


이 보고서가 파이썬에 집중하는 이유는 그것이 제공하는 압도적인 유연성 때문이다. 하지만 파이썬 런치 파일의 진정한 힘을 이해하려면 그 설계 철학을 깊이 파고들 필요가 있다.

표면적으로 파이썬 런치 파일은 때때로 "파이썬답지 않다(un-pythonic)"고 느껴질 수 있다. 예를 들어, 파이썬의 기본 `if` 문 대신 `IfCondition`이라는 클래스를 사용해야 하는 경우가 그렇다. 이는 파이썬 런치 파일의 역할이 명령을 *즉시 실행*하는 것이 아니라, 실행될 시스템의 *설명(description)*을 생성하는 데 있기 때문이다.

`generate_launch_description` 함수는 `LaunchDescription`이라는 객체, 즉 시스템의 청사진(blueprint)을 반환하는 공장(factory)과 같다. 이 함수가 하는 일은 `Node` 액션, `IncludeLaunchDescription` 액션 등 다양한 부품을 조립하여 이 청사진을 만드는 것이다. 실제 시스템을 구동하는 것은 `ros2 launch`라는 이름의 건설팀이며, 이 팀은 완성된 청사진을 받아 그대로 시스템을 구축한다.

이러한 설계는 시스템의 정적 분석과 시각화를 가능하게 하고, 실행 전 단계에서 시스템의 구성을 검증할 수 있게 한다. 따라서 파이썬 런치 파일의 진정한 패러다임은 **"선언적 런치 설명을 프로그래밍 방식으로 생성하는 것"**이다. 이는 단순한 스크립팅을 넘어, 복잡하고 동적인 로봇 시스템을 체계적으로 기술하고 관리하는 강력한 방법을 제공한다.


| 특징               | Python                                                       | XML                                               | YAML                                 |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------- | ------------------------------------ |
| **문법 스타일**    | 프로그래밍 (Python 스크립트)                                 | 마크업 (태그 기반)                                | 마크업 (들여쓰기 기반)               |
| **유연성/능력**    | 최상. 모든 프로그래밍 로직 사용 가능                         | 제한적. 정적 선언에 국한됨                        | 제한적. 정적 선언에 국한됨           |
| **조건부 로직**    | `IfCondition`, `PythonExpression` 등 강력한 조건부 실행 지원 | `if`/`unless` 속성으로 제한적 지원                | 직접적인 조건부 로직 지원 미비       |
| **가독성 (단순)**  | 상대적으로 장황할 수 있음                                    | 명확하고 구조적임                                 | 가장 간결하고 읽기 쉬움              |
| **주요 사용 사례** | 동적 설정, 복잡한 로직, 조건부 노드 실행, 대규모 프로젝트    | 정적 시스템, ROS 1 마이그레이션, 간단한 노드 그룹 | 파라미터 파일, 매우 간단한 런치 구성 |


이 섹션에서는 파이썬 런치 파일과 이를 포함하는 패키지를 구조화하는 필수 구성 요소와 모범 사례를 분석한다.


소규모 프로젝트에서는 기존 패키지 내에 런치 파일을 두는 것이 편리할 수 있지만, 프로젝트가 성장함에 따라 런치 파일과 설정 파일들을 중앙에서 관리하는 것이 중요해진다. 이를 위한 모범 사례는 프로젝트의 모든 런치 파일을 전담하는 별도의 패키지를 만드는 것이다.

이 패키지는 관례적으로 <robot_name>_bringup과 같은 이름으로 명명된다.

이 패키지의 일반적인 구조는 다음과 같다:

1. **패키지 생성:** `ament_cmake` 빌드 타입으로 새 패키지를 생성한다.
2. **디렉토리 정리:** 코드 파일이 없으므로 `src/`와 `include/` 디렉토리는 제거한다.
3. **디렉토리 생성:** 런치 파일을 위한 `launch/` 디렉토리와 파라미터 YAML 파일을 위한 `config/` 디렉토리를 생성한다.
4. **의존성 명시 (`package.xml`):** 이 런치 파일에서 실행할 노드가 포함된 모든 패키지에 대해 실행 시간 의존성(`<exec_depend>`)을 추가한다. 이는 `colcon`이 빌드 순서를 올바르게 결정하는 데 도움을 준다.
5. **설치 규칙 정의:** `launch/`와 `config/` 디렉토리의 파일들이 `colcon build` 시 설치(install) 디렉토리에 복사되도록 설정해야 한다.
   - **ament_cmake (`CMakeLists.txt`):** `install(DIRECTORY launch config DESTINATION share/${PROJECT_NAME})`와 같은 명령어를 추가한다.
   - **ament_python (`setup.py`):** `data_files` 목록에 `('share/' + package_name + '/launch', glob('launch/*.py'))`와 같은 형식으로 런치 파일과 설정 파일을 포함시킨다.1

이러한 구조는 코드(다른 패키지)와 설정 및 실행(bringup 패키지)을 명확히 분리하여, 확장 가능하고 유지보수하기 쉬운 프로젝트를 만든다.


모든 파이썬 런치 파일은 특정 구조를 따라야 한다. 다음은 두 개의 `turtlesim` 노드를 실행하는 최소 기능의 런치 파일 예제다.

```Python
# 1. 필수 모듈 임포트
from launch import LaunchDescription
from launch_ros.actions import Node

# 2. 필수 함수 정의
def generate_launch_description():
    # 3. LaunchDescription 객체 생성
    ld = LaunchDescription()

    # 4. Node 액션 정의
    talker_node = Node(
        package="demo_nodes_cpp",
        executable="talker"
    )
    listener_node = Node(
        package="demo_nodes_py",
        executable="listener"
    )

    # 5. 액션을 LaunchDescription에 추가
    ld.add_action(talker_node)
    ld.add_action(listener_node)

    # 6. LaunchDescription 객체 반환
    return ld
```

1. **필수 모듈 임포트:** `launch` 모듈에서 `LaunchDescription` 클래스를, `launch_ros.actions` 모듈에서 `Node`와 같은 ROS 관련 액션을 임포트한다.
2. **`generate_launch_description` 함수:** 모든 파이썬 런치 파일은 반드시 `generate_launch_description`이라는 이름의 함수를 포함해야 한다. `ros2 launch` 시스템은 이 함수를 호출하여 런치 설명을 얻는다.
3. **`LaunchDescription` 객체 생성:** 이 객체는 실행할 모든 액션(노드, 다른 런치 파일 포함 등)을 담는 최상위 컨테이너다.
4. **`Node` 액션 정의:** `Node` 클래스는 단일 ROS 노드를 실행하기 위한 액션이다. 최소한 노드 실행 파일이 포함된 `package`와 `executable` 이름을 지정해야 한다.
5. **액션 추가:** 생성된 `Node` 객체들을 `LaunchDescription` 객체의 `add_action()` 메소드를 사용해 추가한다. 추가되지 않은 액션은 실행되지 않는다.
6. **객체 반환:** 모든 액션이 추가된 `LaunchDescription` 객체를 반환하면 런치 파일의 역할은 끝난다.


개발자들이 흔히 겪는 혼란 중 하나는 `IncludeLaunchDescription`을 통해 다른 런치 파일을 포함시킬 때, 포함된 파일의 `generate_launch_description` 함수가 두 번 호출되는 현상이다. `print` 문을 넣어보면 이 현상을 명확히 확인할 수 있다.

이는 버그가 아니라 ROS 2 런치 시스템의 의도된 동작이다. 런치 시스템은 실제 실행에 앞서 전체 런치 파일 트리를 분석하고 설명하는 "인트로스펙션(introspection)" 단계를 거친다. 이 과정에서 포함된 파일들을 파싱하며 `generate_launch_description`이 처음 호출된다. 그 후, 실제 시스템을 구동하는 "실행(execution)" 단계에서 다시 한번 평가가 이루어지며 두 번째 호출이 발생할 수 있다.

이러한 동작 방식이 시사하는 바는 매우 중요하다. `generate_launch_description` 함수는 **멱등성(idempotent)**을 가져야 하며, **부수 효과(side effect)가 없어야 한다.** 즉, 이 함수는 몇 번을 호출되더라도 동일한 `LaunchDescription` 객체를 생성해야 하며, 파일 쓰기나 외부 명령어 실행과 같은 실제적인 변화를 일으켜서는 안 된다.

만약 런치 과정에서 특정 명령어를 실행해야 한다면, `print`나 `os.system`과 같은 파이썬 함수를 직접 사용하는 대신, `ExecuteProcess`와 같은 `launch.Action` 객체로 감싸야 한다. 이렇게 함으로써 해당 동작이 인트로스펙션 단계에서는 무시되고 오직 실행 단계에서만 한 번 수행되도록 보장할 수 있다. 이는 견고하고 예측 가능한 런치 파일을 작성하기 위한 핵심 설계 패턴이다.


`launch_ros.actions.Node`는 가장 빈번하게 사용되는 핵심 액션이다. 이 섹션에서는 `Node` 액션의 주요 파라미터를 예제와 함께 심층적으로 분석한다.


가장 기본적인 노드 실행을 위해서는 다음 네 가지 파라미터에 대한 이해가 필수적이다.

- `package`: 노드의 실행 파일이 포함된 ROS 2 패키지의 이름 (문자열).
- `executable`: 패키지 내에서 실행할 실행 파일의 이름 (문자열).
- `name`: ROS 계산 그래프 상에 표시될 노드의 고유한 이름 (문자열). 이 값을 지정하지 않으면 `executable` 이름이 기본값으로 사용된다. 동일한 노드를 여러 개 실행할 때 충돌을 피하기 위해 반드시 다른 `name`을 부여해야 한다.
- `namespace`: 노드와 관련된 모든 토픽, 서비스, 파라미터 이름 앞에 붙는 접두사 (문자열). 여러 로봇을 동시에 구동하거나 동일한 서브시스템을 여러 개 실행할 때 이름 충돌을 방지하는 핵심적인 기능이다.

**예제:**

```Python
Node(
    package='turtlesim',
    executable='turtlesim_node',
    namespace='turtlesim1',
    name='sim'
)
```

위 코드는 `turtlesim` 패키지의 `turtlesim_node` 실행 파일을 `/turtlesim1` 네임스페이스 하에서 `/turtlesim1/sim`이라는 이름으로 실행한다.


노드의 동작을 동적으로 변경하기 위해 파라미터를 사용한다. 런치 파일에서 파라미터를 설정하는 방법은 크게 두 가지다.

1. **인라인 설정:** `parameters` 인자에 파라미터 이름과 값을 담은 딕셔너리의 리스트를 직접 전달한다.

   ```Python
   Node(
       package='turtlesim',
       executable='turtlesim_node',
       name='sim',
       parameters=[
           {'background_r': 255},
           {'background_g': 0},
           {'background_b': 0}
       ]
   )
   ```

   이 방식은 소수의 파라미터를 설정할 때 간편하지만, 파라미터 수가 많아지면 런치 파일이 지저분해진다.

2. **YAML 파일 로딩:** 더 확장 가능하고 권장되는 방법은 파라미터를 별도의 YAML 파일에 정의하고, 이 파일의 경로를 `parameters` 인자에 전달하는 것이다.

   - **`config/my_params.yaml`:**

     ```YAML
     # YAML 파일 최상위에 노드의 최종 이름이 와야 한다.
     /my_robot/controller_node:
       ros__parameters:
         wheel_radius: 0.15
         max_velocity: 2.5
     ```

     YAML 파일의 구조는 `node_name: ros__parameters: {key: value}` 형식을 엄격하게 따라야 한다.

   - **런치 파일:**

     ```Python
     import os
     from ament_index_python.packages import get_package_share_directory
     
     #...
     param_file = os.path.join(
         get_package_share_directory('my_robot_bringup'),
         'config',
         'my_params.yaml'
     )
     Node(
         package='my_robot_driver',
         executable='controller',
         name='controller_node',
         namespace='my_robot',
         parameters=[param_file]
     )
     ```

여기서 매우 중요한 점이 있다. 파라미터 로딩은 단순히 런치 파일과 노드 간의 2단계 과정이 아니다. 이는 **런치 파일-YAML-노드 코드 간의 3자간의 약속(Three-Way Handshake)**이다.

1. **런치 파일**은 YAML 파일의 정확한 경로를 제공한다.
2. **YAML 파일**은 정확한 노드 이름 아래에 파라미터 값들을 정의한다.
3. **노드의 소스 코드(C++ 또는 Python)**는 `declare_parameter()` 또는 `declare_parameters()` 함수를 사용해 자신이 받을 파라미터들을 **반드시 사전에 선언**해야 한다.

이 세 가지 중 하나라도 어긋나면(경로 오류, YAML 내 노드 이름 불일치, 코드 내 미선언), 파라미터는 종종 아무런 에러 메시지 없이 조용히 로드에 실패한다. 이는 초보자들이 흔히 겪는 문제의 원인이므로 반드시 기억해야 한다.


리매핑은 ROS의 강력한 기능 중 하나로, 노드가 사용하는 토픽, 서비스, 액션의 이름을 런치 시점에 변경할 수 있게 해준다. 이는 서로 다른 이름 규칙을 사용하는 패키지들을 통합하거나, 다중 로봇 시스템에서 데이터 흐름을 제어하는 데 필수적이다.

`remappings` 인자는 `('원본 이름', '새 이름')` 형태의 튜플 리스트를 받는다.

**예제:** `turtlesim`의 두 번째 거북이(`turtlesim2`)가 첫 번째 거북이(`turtlesim1`)의 움직임을 따라 하도록 `mimic` 노드를 설정하는 경우다.

```Python
Node(
    package='turtlesim',
    executable='mimic',
    name='mimic',
    remappings=[
        ('/input/pose', '/turtlesim1/turtle1/pose'),      # 입력 포즈 토픽을 turtlesim1의 포즈로
        ('/output/cmd_vel', '/turtlesim2/turtle1/cmd_vel') # 출력 속도 명령 토픽을 turtlesim2의 명령으로
    ]
)
```

이 설정을 통해 `mimic` 노드는 `turtlesim1`의 위치를 구독하여 `turtlesim2`의 속도 명령으로 발행하게 된다.


노드의 동작을 확인하고 문제를 해결하기 위한 두 가지 핵심 파라미터가 있다.

- `output`: 노드의 표준 출력(stdout)과 표준 에러(stderr)를 어디로 보낼지 결정한다. `output='screen'`으로 설정하면 `RCLCPP_INFO`, `RCLCPP_ERROR` 등의 로그 메시지가 런치 파일을 실행한 터미널에 직접 표시된다. 이는 가장 기본적인 디버깅 방법이다.2
- `prefix`: 노드 실행 명령어 앞에 추가적인 명령어를 붙일 수 있게 해준다. 이는 GDB와 같은 디버거를 붙이는 주된 방법이다. 예를 들어 `prefix='xterm -e gdb --args'`는 새로운 xterm 터미널을 열고 그 안에서 GDB를 실행한 뒤 노드를 시작시킨다. 이 고급 디버깅 기법은 6장에서 자세히 다룬다.


대규모 프로젝트에서는 수십 개의 노드와 파라미터를 단일 런치 파일에서 관리하는 것은 비효율적이고 유지보수를 어렵게 만든다. 이 섹션에서는 런치 시스템을 작고 재사용 가능한 구성 요소로 분해하여 확장 가능하고 유지보수하기 쉬운 시스템을 구축하는 기술을 다룬다.


`IncludeLaunchDescription` 액션은 하나의 런치 파일이 다른 런치 파일을 포함할 수 있게 해주는 가장 기본적인 모듈화 도구다. 이를 통해 전체 시스템을 기능 단위(예: 센서, 인식, 내비게이션)로 나눌 수 있다.

포함할 런치 파일의 경로는 하드코딩하는 대신, 이식성을 위해 `FindPackageShare`와 `PathJoinSubstitution`을 함께 사용하는 것이 일반적이다.3

- `FindPackageShare('package_name')`: 지정된 패키지의 공유(share) 디렉토리 경로를 찾아준다.
- `PythonLaunchDescriptionSource`: 파이썬 런치 파일을 포함할 때 사용한다.
- `PathJoinSubstitution([...])`: 여러 경로 구성 요소를 운영체제에 맞게 결합하여 전체 경로를 만든다.

**예제:** 최상위 런치 파일에서 센서 런치 파일과 내비게이션 런치 파일을 포함하는 경우.

```Python
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.substitutions import PathJoinSubstitution
from launch_ros.substitutions import FindPackageShare

def generate_launch_description():
    sensors_launch_path = PathJoinSubstitution()
    navigation_launch_path = PathJoinSubstitution()

    return LaunchDescription()
```


런치 파일을 재사용 가능하게 만들려면 하드코딩된 값을 외부에서 주입할 수 있어야 한다. 이 역할을 하는 것이 `DeclareLaunchArgument`와 `LaunchConfiguration`이다.

- `DeclareLaunchArgument`: 런치 파일이 받을 수 있는 인자를 선언한다. 인자의 이름, 기본값, 설명을 지정할 수 있다.1
- `LaunchConfiguration`: 선언된 런치 인자의 값을 런타임에 가져오는 대체(substitution) 객체다.1

이 인자들은 커맨드 라인(`ros2 launch... my_arg:=value`)이나 부모 런치 파일의 `IncludeLaunchDescription`의 `launch_arguments` 파라미터를 통해 전달될 수 있다.

**예제:** `use_sim_time` 파라미터를 런치 인자로 받아 설정하는 경우.

```Python
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration
from launch_ros.actions import Node

def generate_launch_description():
    # 1. 런치 인자 선언
    use_sim_time_arg = DeclareLaunchArgument(
        'use_sim_time',
        default_value='true',
        description='Use simulation (Gazebo) clock if true'
    )

    # 2. LaunchConfiguration으로 값 가져오기
    use_sim_time = LaunchConfiguration('use_sim_time')

    my_node = Node(
        package='my_package',
        executable='my_node_exec',
        parameters=[{'use_sim_time': use_sim_time}] # 파라미터로 전달
    )

    return LaunchDescription([
        use_sim_time_arg,
        my_node
    ])
```

이렇게 하면 동일한 런치 파일을 시뮬레이션과 실제 로봇 환경 모두에서 재사용할 수 있다.


런치 파일을 포함할 때 흔히 발생하는 문제 중 하나는 부모 런치 파일에서 설정한 네임스페이스나 리매핑 규칙이 포함된 런치 파일의 노드들에게 적용되지 않는 것이다. 이는 포함된 런치 파일이 자신만의 독립적인 스코프(scope)를 가지기 때문이다.

이 문제를 해결하고 특정 설정을 포함된 런치 파일에 적용하려면 `GroupAction`을 사용해야 한다. `GroupAction`으로 `IncludeLaunchDescription`을 감싸고, 그룹 내에서 `PushRosNamespace`나 `SetRemap`과 같은 액션을 사용하면 해당 설정이 그룹 내의 모든 액션에 적용된다.

**예제:** `robot1` 네임스페이스를 `nav2_bringup` 런치 파일에 적용하는 경우.

```Python
from launch.actions import GroupAction, IncludeLaunchDescription, SetRemap
from launch_ros.actions import PushRosNamespace

#...
nav2_launch = IncludeLaunchDescription(...)

# 잘못된 방법: 네임스페이스가 적용되지 않음
# Node(..., namespace='robot1', actions=[nav2_launch]) # 이런 문법은 존재하지 않음

# 올바른 방법: GroupAction 사용
robot1_nav_group = GroupAction(
    actions=
)

return LaunchDescription([robot1_nav_group])
```

스코프 규칙을 이해하고 `GroupAction`을 올바르게 사용하는 것은 복잡한 시스템을 구성할 때 발생하는 많은 문제를 예방해준다.


이 섹션에서는 런치 파일에 로직을 추가하여 다양한 시나리오에 적응할 수 있도록 만드는 방법을 다룬다.


런치 파일의 가장 기본적인 동적 기능은 특정 조건에 따라 노드나 다른 액션을 실행하거나 실행하지 않는 것이다. 모든 `launch.Action` 객체는 `condition`이라는 파라미터를 가지고 있으며, 여기에 조건 객체를 전달할 수 있다.

- `IfCondition(value)`: `value`가 참(`'true'`, `'1'`)으로 평가될 때 액션을 실행한다.4
- `UnlessCondition(value)`: `value`가 거짓(`'false'`, `'0'`)으로 평가될 때 액션을 실행한다.

일반적으로 조건의 `value`로는 `LaunchConfiguration`을 사용하여 런치 인자를 전달한다.

**예제:** `launch_rviz` 인자가 `true`일 때만 RViz를 실행하는 경우.

```Python
from launch.actions import DeclareLaunchArgument, IncludeLaunchDescription
from launch.substitutions import LaunchConfiguration
from launch.conditions import IfCondition

def generate_launch_description():
    # RViz 실행 여부를 결정하는 런치 인자
    launch_rviz_arg = DeclareLaunchArgument(
        'launch_rviz',
        default_value='true'
    )
    
    rviz_launch_include = IncludeLaunchDescription(
        #... rviz 런치 파일 경로...
        condition=IfCondition(LaunchConfiguration('launch_rviz')) # 조건 설정
    )

    return LaunchDescription([
        launch_rviz_arg,
        rviz_launch_include
    ])
```


단순한 참/거짓 확인을 넘어 여러 조건을 조합(AND, OR)하거나 문자열을 비교해야 할 경우, `IfCondition`만으로는 부족하다. 이때 `PythonExpression` 대체(substitution)를 사용해야 한다.

`PythonExpression`은 문자열과 다른 대체 객체들의 리스트를 받아, 이들을 모두 합친 후 파이썬의 `eval()` 함수로 평가한다. 그 결과가 참이면 조건이 만족된다.

**예제:** `mode` 인자가 `'slam'`이고 `use_gui` 인자가 `'true'`일 때만 특정 노드를 실행하는 경우.

```Python
from launch.substitutions import LaunchConfiguration, PythonExpression
from launch.conditions import IfCondition

mode = LaunchConfiguration('mode')
use_gui = LaunchConfiguration('use_gui')

slam_toolbox_node = Node(
    #...
    condition=IfCondition(
        PythonExpression([
            "'", mode, "' == 'slam' and '", use_gui, "' == 'true'"
        ])
    )
)
```

`PythonExpression`은 강력하지만, 문법이 복잡하고 따옴표와 공백 처리에 매우 민감하여 오류가 발생하기 쉽다. 각 `LaunchConfiguration`은 런타임에 그 값(문자열)으로 치환되므로, 최종적으로 `eval()`에 전달될 파이썬 표현식이 올바른 문법을 갖도록 신중하게 구성해야 한다. 이는 런치 시스템의 선언적 세계에 명령형 로직을 주입하는 강력한 탈출구인 동시에, 가독성을 해치고 디버깅을 어렵게 만들 수 있는 양날의 검이다.


이 섹션에서는 런치 파일을 실행하고, 테스트하며, 문제를 해결하는 실용적인 워크플로우를 제공한다.


런치 파일을 실행하는 기본 명령어는 `ros2 launch <package_name> <launch_file_name>`이다. 런치 인자는 `arg_name:=value` 형식으로 전달한다.

```Bash
# 기본 실행
ros2 launch my_robot_bringup robot.launch.py

# 인자 전달
ros2 launch my_robot_bringup robot.launch.py use_sim_time:=true launch_rviz:=false
```

런치 파일이 어떤 인자를 받는지 확인하려면 `--show-args` 플래그를 사용하면 유용하다.

```Bash
ros2 launch my_robot_bringup robot.launch.py --show-args
```


문제 해결의 첫 단계는 로그를 확인하는 것이다.

- **`output="screen"`:** `Node` 액션에 이 옵션을 추가하면 해당 노드의 모든 로그가 터미널에 직접 출력되어 즉각적인 피드백을 얻을 수 있다.2
- **`rqt_console`:** 여러 노드가 동시에 실행될 때는 `rqt_console`이 필수적이다. 이 GUI 도구는 시스템의 모든 노드에서 발생하는 로그 메시지를 한 곳에 모아 보여주며, 심각도 수준에 따른 필터링, 특정 문자열 검색 등 강력한 기능을 제공한다.2


세그멘테이션 폴트(segmentation fault)와 같이 로그만으로 해결하기 어려운 심각한 버그는 GDB(GNU Debugger)와 같은 디버거를 사용하여 해결해야 한다.

**GDB 사용 절차:**

1. **디버그 심볼과 함께 컴파일:** 디버깅하려는 패키지를 디버그 정보와 함께 빌드해야 한다. `RelWithDebInfo`(최적화 포함 디버그) 또는 `Debug`(최적화 없음) 빌드 타입을 사용한다.

   ```Bash
   colcon build --packages-select my_package --cmake-args -DCMAKE_BUILD_TYPE=Debug
   ```

2. **런치 파일 수정 (`prefix` 사용):** 디버깅할 `Node` 액션에 `prefix` 인자를 추가한다.

   - **xterm 사용:** `prefix=['xterm -e gdb --args']`를 추가하면, 런치 시 새로운 터미널 창이 열리고 그 안에서 GDB가 해당 노드를 실행한다. 여기서 중단점(breakpoint) 설정, 변수 확인 등이 가능하다.
   - **gdbserver 사용:** `prefix=['gdbserver localhost:3000']`를 추가하면, 노드가 GDB 서버와 함께 실행된다. 이후 다른 터미널이나 VSCode와 같은 IDE에서 이 서버에 원격으로 접속하여 디버깅할 수 있다.

**복잡한 시스템에서의 디버깅:**

만약 디버깅하려는 노드가 다른 패키지의 런치 파일에 의해 포함(include)되어 있어 직접 수정하기 어렵다면 어떻게 해야 할까? 예를 들어, `robot.launch.py`가 `navigation.launch.py`를 포함하고, 실제 디버깅 대상은 `navigation.launch.py` 안의 `planner_server` 노드인 경우다.

이러한 상황에서의 실용적인 접근법은 시스템을 일시적으로 **분해(de-compose)**하는 것이다.

1. 부모 런치 파일(`robot.launch.py`)에서 `navigation.launch.py`를 포함하는 `IncludeLaunchDescription` 부분을 일시적으로 주석 처리한다.
2. 부모 런치 파일을 실행하여 내비게이션을 제외한 나머지 시스템을 구동한다.
3. 별도의 터미널에서, `navigation.launch.py`를 `ros2 launch`로 직접 실행하되, 이번에는 `planner_server` 노드에 GDB `prefix`를 추가한 수정된 버전의 런치 파일을 사용하거나, 커맨드 라인에서 `prefix`를 적용하여 실행한다.

이처럼 모듈성은 시스템 구성에는 유리하지만, 심층 디버깅을 위해서는 때때로 관심 있는 구성 요소를 분리하여 독립적으로 실행하는 것이 더 효율적이다.


이 마지막 섹션에서는 앞서 다룬 모든 개념을 종합하여, 자율주행 모바일 로봇을 위한 현실적인 런치 시스템을 구성하는 전체 과정을 보여준다.


- **로봇:** Lidar, 카메라, 차동 구동 기반을 갖춘 모바일 로봇
- **소프트웨어:** Nav2 스택을 이용한 자율 주행, 시뮬레이션 및 실제 로봇 구동 지원
- **요구사항:**
  - 최상위 런치 파일 하나로 전체 시스템 구동
  - 시뮬레이션 모드와 실제 로봇 모드 전환 가능 (`use_sim_time`)
  - RViz 시각화 도구 실행 여부 선택 가능 (`launch_rviz`)
  - 모듈화된 구조 (센서, 내비게이션, 시각화 분리)


`my_robot_bringup` 패키지 내에 다음과 같이 파일을 생성한다.

```
my_robot_bringup/
├── launch/
│   ├── robot.launch.py         # 최상위 런치 파일
│   ├── sensors.launch.py       # 센서 드라이버 런치 파일
│   ├── navigation.launch.py    # Nav2 런치 파일
│   └── rviz.launch.py          # RViz 런치 파일
├── config/
│   ├── nav2_params.yaml        # Nav2 파라미터 파일
│   └── my_robot.rviz           # RViz 설정 파일
├── package.xml
└── CMakeLists.txt
```



Lidar와 카메라 드라이버 노드를 실행한다.

```Python
# launch/sensors.launch.py
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    lidar_node = Node(
        package='rplidar_ros',
        executable='rplidar_node',
        name='lidar_node',
        parameters=[{'frame_id': 'laser_frame'}]
    )
    camera_node = Node(
        package='usb_cam',
        executable='usb_cam_node_exe',
        name='camera_node'
    )
    return LaunchDescription([lidar_node, camera_node])
```


Nav2 패키지의 메인 런치 파일을 포함하고, 파라미터 파일과 `use_sim_time` 인자를 전달한다.

```Python
# launch/navigation.launch.py
import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, IncludeLaunchDescription
from launch.substitutions import LaunchConfiguration, PathJoinSubstitution
from launch.launch_description_sources import PythonLaunchDescriptionSource

def generate_launch_description():
    use_sim_time = LaunchConfiguration('use_sim_time')
    params_file = LaunchConfiguration('params_file')

    nav2_bringup_dir = get_package_share_directory('nav2_bringup')
    nav2_launch_dir = os.path.join(nav2_bringup_dir, 'launch')

    return LaunchDescription()
```


지정된 설정 파일로 RViz2를 실행한다.

```Python
# launch/rviz.launch.py
import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    rviz_config_file = os.path.join(get_package_share_directory('my_robot_bringup'),
                                    'config', 'my_robot.rviz')
    rviz_node = Node(
        package='rviz2',
        executable='rviz2',
        name='rviz2',
        arguments=['-d', rviz_config_file],
        output='screen'
    )
    return LaunchDescription([rviz_node])
```


모든 것을 하나로 묶는다. 인자를 선언하고, 조건부로 다른 런치 파일들을 포함한다.

```Python
# launch/robot.launch.py
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, IncludeLaunchDescription
from launch.conditions import IfCondition
from launch.substitutions import LaunchConfiguration, PathJoinSubstitution
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch_ros.substitutions import FindPackageShare

def generate_launch_description():
    # 런치 인자 선언
    use_sim_time_arg = DeclareLaunchArgument('use_sim_time', default_value='true')
    launch_rviz_arg = DeclareLaunchArgument('launch_rviz', default_value='true')

    # LaunchConfiguration으로 값 가져오기
    use_sim_time = LaunchConfiguration('use_sim_time')
    launch_rviz = LaunchConfiguration('launch_rviz')

    # 각 서브시스템 런치 파일 경로 정의
    bringup_pkg_path = FindPackageShare('my_robot_bringup')
    sensors_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(PathJoinSubstitution([bringup_pkg_path, 'launch', 'sensors.launch.py']))
    )
    navigation_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(PathJoinSubstitution([bringup_pkg_path, 'launch', 'navigation.launch.py'])),
        launch_arguments={'use_sim_time': use_sim_time}.items()
    )
    rviz_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(PathJoinSubstitution([bringup_pkg_path, 'launch', 'rviz.launch.py'])),
        condition=IfCondition(launch_rviz) # 조건부 실행
    )

    return LaunchDescription([
        use_sim_time_arg,
        launch_rviz_arg,
        sensors_launch,
        navigation_launch,
        rviz_launch
    ])
```

이 종합 예제는 모듈성, 재사용성, 동적 구성, 파라미터 관리 등 이 보고서에서 다룬 모든 핵심 개념을 실제 프로젝트에 적용하는 방법을 명확하게 보여준다. 이 구조를 기반으로 개발자는 더 복잡하고 정교한 로봇 시스템을 위한 런치 파일을 체계적으로 구축해 나갈 수 있다.


1. Using substitutions - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Launch/Using-Substitutions.html
2. Launching nodes - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Launching-Multiple-Nodes/Launching-Multiple-Nodes.html
3. Managing large projects - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/Launch/Using-ROS2-Launch-For-Large-Projects.html
4. Equivalents of "if" and "unless" ROS1 XML for ROS2 python launch ..., accessed July 26, 2025, https://answers.ros.org/question/410161/


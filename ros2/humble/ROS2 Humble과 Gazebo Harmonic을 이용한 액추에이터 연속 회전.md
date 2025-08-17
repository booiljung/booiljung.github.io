# ROS2 Humble과 Gazebo Harmonic을 이용한 액추에이터 연속 회전

이 보고서는 ROS2 Humble과 최신 Gazebo 시뮬레이터인 Gazebo Harmonic을 사용하여 단일 액추에이터(회전 관절)를 지속적으로 회전시키는 완전한 예제를 제공한다. 전체 시스템 아키텍처, 환경 설정, 필수 코드 파일(URDF, YAML, Launch, Python), 실행 및 검증 절차, 그리고 시스템 동작의 이론적 배경까지 심도 있게 다룬다.

## 1. 시스템 아키텍처 및 환경 설정

성공적인 시뮬레이션을 위해서는 먼저 구성 요소들이 어떻게 상호작용하는지 이해하고, 호환되는 환경을 올바르게 구축해야 한다.

### 1.1 제어 파이프라인 개요

본 예제에서 구현할 제어 시스템의 데이터 흐름은 다음과 같다.

1. **Python 퍼블리셔 노드**: 일정한 속도 명령(`std_msgs/msg/Float64MultiArray` 타입)을 생성하여 특정 토픽으로 발행한다.
2. **ROS2 토픽 (`/velocity_controller/commands`)**: 퍼블리셔 노드와 컨트롤러 사이의 통신 채널 역할을 한다.
3. **`ros2_control` 컨트롤러 (`velocity_controllers/JointGroupVelocityController`)**: 토픽을 구독하여 속도 명령을 수신한다.1
4. **컨트롤러 매니저 (`ros2_control_node`)**: 컨트롤러의 생명주기를 관리하고, 실시간 제어 루프를 실행하여 컨트롤러의 `update()` 함수를 주기적으로 호출한다.2
5. **Gazebo 시스템 플러그인 (`gz_ros2_control`)**: `ros2_control`을 위한 하드웨어 인터페이스 역할을 한다. 컨트롤러로부터 받은 명령을 Gazebo 시뮬레이션이 이해할 수 있는 형태로 변환하여 전달한다.4
6. **Gazebo Harmonic**: 물리 엔진을 통해 로봇 모델의 관절에 명령을 적용하고, 그 결과를 시뮬레이션한다.
7. **상태 피드백**: `gz_ros2_control` 플러그인은 시뮬레이션으로부터 관절의 현재 상태(위치, 속도 등)를 읽어와 `ros2_control`의 상태 인터페이스(state interface)를 통해 컨트롤러 매니저와 다른 ROS 노드에 제공한다. 이 정보는 `/joint_states`와 같은 토픽으로 발행된다.6

### 1.2 핵심 사항: ROS2 Humble & Gazebo Harmonic 호환성 매트릭스 탐색

가장 먼저 해결해야 할 중요한 문제는 ROS2 Humble과 Gazebo Harmonic의 호환성이다. 공식적으로 ROS2 Humble은 Gazebo Fortress를 지원한다.7 따라서 표준적인 `sudo apt install ros-humble-ros-gz` 명령은 Gazebo Fortress를 설치하게 된다.

하지만 사용자의 요구사항은 Gazebo Harmonic이다. 이 비표준 조합을 사용하기 위해서는 Gazebo 프로젝트에서 직접 제공하는 패키지 저장소를 이용해야 한다. 이 방법은 Humble의 공식 Gazebo 패키지(`ros-humble-ros-gz-fortress` 등)와 충돌을 일으킬 수 있으므로, 기존에 Gazebo Fortress 관련 패키지를 설치했다면 제거하거나 깨끗한 환경에서 시작하는 것이 좋다.7

다음 단계에 따라 ROS2 Humble 환경에 Gazebo Harmonic을 설치한다.

1. Gazebo 패키지 저장소 추가:

   Gazebo의 GPG 키를 등록하고 패키지 소스를 시스템에 추가한다.

   ```Bash
   sudo wget https://packages.osrfoundation.org/gazebo.gpg -O /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null
   ```

2. Gazebo Harmonic 설치:

   패키지 목록을 업데이트하고 Gazebo Harmonic을 설치한다.

   ```Bash
   sudo apt-get update
   sudo apt-get install gz-harmonic
   ```

3. 비공식 ROS2-Gazebo 연동 패키지 설치:

   Gazebo Harmonic과 ROS2 Humble을 연결하는 비공식 브릿지 패키지를 설치한다.7

   ```Bash
   sudo apt-get install ros-humble-ros-gzharmonic
   ```

### 1.3 `ros2_control` 및 필수 패키지 설치

다음으로 `ros2_control` 프레임워크와 컨트롤러, 그리고 Gazebo 연동을 위한 플러그인을 설치해야 한다. `ros2_control`은 로봇 하드웨어를 추상화하고 제어기들을 관리하는 프레임워크다.8

Gazebo 연동 플러그인은 두 가지가 존재하는데, `gazebo_ros2_control`은 구버전인 Gazebo Classic(11.x 이하)용이며 더 이상 유지보수되지 않는다.10 최신 Gazebo(Garden, Harmonic, Ionic 등)에서는 `gz_ros2_control`을 사용해야 한다.4 이 예제는 Gazebo Harmonic을 사용하므로 `gz_ros2_control` 관련 패키지를 설치한다.

```Bash
sudo apt install ros-humble-ros2-control ros-humble-ros2-controllers ros-humble-gz-ros2-control ros-humble-gz-ros2-control-demos
```

이 명령은 다음을 설치한다:

- `ros-humble-ros2-control`: `ros2_control`의 핵심 구성요소(컨트롤러 매니저, 리소스 매니저 등).
- `ros-humble-ros2-controllers`: `ForwardCommandController`, `JointTrajectoryController` 등 널리 사용되는 표준 컨트롤러 모음.8
- `ros-humble-gz-ros2-control`: 최신 Gazebo와 `ros2_control`을 연동하는 플러그인.
- `ros-humble-gz-ros2-control-demos`: `gz_ros2_control`의 사용법을 보여주는 예제 패키지.

## 2. 물리적 구현: 로봇 URDF 정의

URDF(Unified Robot Description Format)는 로봇의 물리적 구조(링크, 관절)와 `ros2_control` 및 Gazebo와의 연동 설정을 정의하는 XML 파일이다.

### 2.1 패키지 및 파일 구조 생성

먼저 이 예제를 위한 ROS2 패키지를 생성한다.

```Bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_python simple_velocity_robot
cd simple_velocity_robot
mkdir urdf config launch
```

생성된 패키지는 다음과 같은 파일 구조를 갖게 된다.

```
simple_velocity_robot/
├── config/
│   └── controllers.yaml
├── launch/
│   └── robot_launch.py
├── src/
│   └── simple_velocity_robot/
│       └── velocity_publisher.py
├── urdf/
│   └── robot.urdf.xacro
├── package.xml
└── setup.py
```

### 2.2 로봇의 청사진: `robot.urdf.xacro`

`urdf/robot.urdf.xacro` 파일에 로봇의 구조를 정의한다. 이 로봇은 고정된 `base_link`와 회전하는 `rotating_link`, 그리고 두 링크를 연결하는 `revolute_joint`로 구성된 매우 간단한 모델이다.

```XML
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro" name="simple_velocity_robot">

  <link name="base_link">
    <visual>
      <geometry>
        <box size="0.5 0.5 0.2"/>
      </geometry>
      <origin xyz="0 0 0.1" rpy="0 0 0"/>
      <material name="grey">
        <color rgba="0.5 0.5 0.5 1.0"/>
      </material>
    </visual>
    <collision>
      <geometry>
        <box size="0.5 0.5 0.2"/>
      </geometry>
      <origin xyz="0 0 0.1" rpy="0 0 0"/>
    </collision>
    <inertial>
      <mass value="1.0"/>
      <inertia ixx="0.1" ixy="0.0" ixz="0.0" iyy="0.1" iyz="0.0" izz="0.1"/>
    </inertial>
  </link>

  <link name="rotating_link">
    <visual>
      <geometry>
        <cylinder radius="0.05" length="0.4"/>
      </geometry>
      <origin xyz="0 0 0.2" rpy="0 0 0"/>
      <material name="blue">
        <color rgba="0.1 0.1 0.8 1.0"/>
      </material>
    </visual>
    <collision>
      <geometry>
        <cylinder radius="0.05" length="0.4"/>
      </geometry>
      <origin xyz="0 0 0.2" rpy="0 0 0"/>
    </collision>
    <inertial>
      <mass value="0.5"/>
      <inertia ixx="0.01" ixy="0.0" ixz="0.0" iyy="0.01" iyz="0.0" izz="0.01"/>
    </inertial>
  </link>

  <joint name="revolute_joint" type="revolute">
    <parent link="base_link"/>
    <child link="rotating_link"/>
    <origin xyz="0 0 0.2" rpy="0 0 0"/>
    <axis xyz="0 0 1"/>
    <limit lower="-3.14" upper="3.14" effort="10.0" velocity="10.0"/>
  </joint>

  <xacro:include filename="$(find simple_velocity_robot)/urdf/robot.ros2_control.xacro" />
  <xacro:rrbot_ros2_control />

</robot>
```

편의를 위해 `ros2_control`과 Gazebo 관련 태그는 별도의 파일(`robot.ros2_control.xacro`)로 분리하여 포함시킨다.

### 2.3 `ros2_control` 인터페이스 태그

`urdf/robot.ros2_control.xacro` 파일을 생성하고 `ros2_control`과의 연동을 위한 핵심 설정을 추가한다.

```XML
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro">

  <xacro:macro name="rrbot_ros2_control">
    <ros2_control name="GazeboSimSystem" type="system">
      <hardware>
        <plugin>gz_ros2_control/GazeboSimSystem</plugin>
      </hardware>
      <joint name="revolute_joint">
        <command_interface name="velocity">
          <param name="min">-10.0</param>
          <param name="max">10.0</param>
        </command_interface>
        <state_interface name="position"/>
        <state_interface name="velocity"/>
      </joint>
    </ros2_control>
  </xacro:macro>

</robot>
```

이 설정의 각 부분은 다음과 같은 의미를 가진다.

- `<ros2_control name="GazeboSimSystem" type="system">`: `ros2_control` 설정 블록을 정의한다. `type="system"`은 여러 관절과 센서를 포함하는 로봇 전체 시스템을 위한 하드웨어 인터페이스를 의미한다.3
- `<hardware><plugin>gz_ros2_control/GazeboSimSystem</plugin></hardware>`: 컨트롤러 매니저의 리소스 매니저가 로드할 하드웨어 인터페이스 플러그인을 지정한다. `GazeboSimSystem`은 최신 Gazebo 시뮬레이터와 연동하기 위한 올바른 플러그인이다.4
- `<joint name="revolute_joint">`: 이 제어 설정을 URDF에 정의된 `revolute_joint`에 연결한다.
- `<command_interface name="velocity">`: 이 관절에 대해 "velocity" 타입의 명령 인터페이스를 노출시킨다. 컨트롤러는 이 인터페이스를 '점유(claim)'하여 제어 명령을 내릴 수 있다. `position`, `effort` 등 다른 타입도 가능하다.6
- `<param name="min/max">`: 명령 인터페이스에 대한 최소/최대값을 설정한다.
- `<state_interface name="position/velocity">`: 하드웨어 인터페이스(Gazebo 플러그인)가 시뮬레이터로부터 관절의 위치와 속도 상태를 읽어와 ROS2 시스템에 제공하도록 지시한다. 이 값들은 `joint_state_broadcaster`에 의해 토픽으로 발행된다.6

### 2.4 Gazebo 시뮬레이션 플러그인 태그

마지막으로, Gazebo가 시뮬레이션을 시작할 때 `ros2_control` 시스템을 로드하도록 지시하는 플러그인을 `robot.urdf.xacro`의 `<xacro:macro>` 내부에 추가한다.

```XML
<gazebo>
  <plugin filename="libgz_ros2_control-system.so" name="gz_ros2_control::GazeboSimROS2ControlPlugin">
    <parameters>$(find simple_velocity_robot)/config/controllers.yaml</parameters>
  </plugin>
</gazebo>
```

- `<plugin filename="libgz_ros2_control-system.so" name="gz_ros2_control::GazeboSimROS2ControlPlugin">`: Gazebo 시뮬레이션 내에서 실행될 플러그인을 지정한다. 이 플러그인은 URDF 내의 `<ros2_control>` 태그를 파싱하고, 컨트롤러 매니저를 포함한 `ros2_control` 프레임워크를 시뮬레이션 환경 내에 인스턴스화하는 역할을 한다.4
- `<parameters>...</parameters>`: 컨트롤러 매니저가 로드될 때 필요한 컨트롤러 설정 파일의 경로를 전달한다. 이를 통해 런치 파일에서 별도로 파라미터를 로드할 필요 없이 URDF만으로 컨트롤러 설정까지 한 번에 처리할 수 있어 편리하다.4

## 3. 제어 로직: 컨트롤러 설정

컨트롤러의 종류, 파라미터, 그리고 컨트롤러 매니저의 동작 방식을 YAML 파일을 통해 설정한다.

### 3.1 `controllers.yaml` 파일

`config/controllers.yaml` 파일을 생성하고 다음 내용을 작성한다.

```YAML
controller_manager:
  ros__parameters:
    update_rate: 100  # Hz

    # The list of controllers to be loaded and activated
    # This is not strictly necessary, as we spawn them from the launch file
    # but it's good practice to list them here.
    # joint_state_broadcaster:
    #   type: joint_state_broadcaster/JointStateBroadcaster
    # velocity_controller:
    #   type: velocity_controllers/JointGroupVelocityController

# Controller configurations
joint_state_broadcaster:
  type: joint_state_broadcaster/JointStateBroadcaster

velocity_controller:
  type: velocity_controllers/JointGroupVelocityController
  ros__parameters:
    joints:
      - revolute_joint
    # The 'interface_name' is implicitly 'velocity' for this controller type
    # interface_name: velocity
```

### 3.2 컨트롤러 매니저 설정

- `controller_manager: ros__parameters: update_rate: 100`: 컨트롤러 매니저의 주된 `read-update-write` 제어 루프의 실행 주기를 100 Hz로 설정한다. 이 값이 높을수록 로봇의 반응성이 향상되지만, 계산 부하도 증가한다.14

### 3.3 브로드캐스터 설정

- `joint_state_broadcaster: type: joint_state_broadcaster/JointStateBroadcaster`: `joint_state_broadcaster`는 `ros2_control`의 필수적인 유틸리티 컨트롤러다. URDF의 `<state_interface>`에 등록된 모든 관절의 상태(위치, 속도 등)를 읽어와 표준 ROS 메시지 형태로 `/joint_states` 및 `/dynamic_joint_states` 토픽에 발행하는 역할을 한다. RViz나 MoveIt과 같은 다른 ROS 도구들이 로봇의 현재 자세를 인지하기 위해 이 토픽을 구독한다.16

### 3.4 속도 컨트롤러 설정

- `velocity_controller: type: velocity_controllers/JointGroupVelocityController`: 목표를 달성하기 위해 사용할 컨트롤러다. `ros2_controllers` 패키지는 `forward_command_controller`라는 범용 컨트롤러를 제공하지만 18, 

  `velocity`, `position`, `effort` 등 특정 인터페이스에 특화된 컨트롤러들도 제공한다. `velocity_controllers/JointGroupVelocityController`는 속도 인터페이스를 사용하도록 미리 설정된 특화 컨트롤러로, 코드가 더 명시적이고 설정이 간결해지는 장점이 있다.1

- `ros__parameters: joints: - revolute_joint`: 이 컨트롤러가 제어할 관절의 목록을 지정한다. 여기서는 `revolute_joint` 하나만 제어한다.1

- `interface_name: velocity`: 이 컨트롤러 타입은 내부적으로 `velocity` 인터페이스를 사용하도록 기본 설정되어 있어 명시적으로 작성할 필요는 없다. 이 설정은 컨트롤러가 URDF에 정의된 `revolute_joint`의 `command_interface` 중 `name="velocity"`인 것을 찾아 점유하도록 지시한다.

### 핵심 설정 파라미터 요약

다음 표는 로봇을 제어하기 위해 URDF와 YAML 파일에서 설정한 핵심 파라미터들을 요약한 것이다.

| 파라미터            | 파일 위치          | 섹션                       | 목적                                                         |
| ------------------- | ------------------ | -------------------------- | ------------------------------------------------------------ |
| `plugin`            | `robot.urdf.xacro` | `<ros2_control><hardware>` | 하드웨어 인터페이스 플러그인(`gz_ros2_control/GazeboSimSystem`)을 지정한다. |
| `command_interface` | `robot.urdf.xacro` | `<ros2_control><joint>`    | 관절이 수용할 명령의 종류(예: `velocity`)를 정의한다.        |
| `state_interface`   | `robot.urdf.xacro` | `<ros2_control><joint>`    | 하드웨어로부터 읽어올 상태 변수(예: `position`, `velocity`)를 정의한다. |
| `plugin`            | `robot.urdf.xacro` | `<gazebo>`                 | Gazebo가 `ros2_control`을 로드하기 위해 사용할 플러그인(`libgz_ros2_control-system.so`)을 지정한다. |
| `update_rate`       | `controllers.yaml` | `controller_manager`       | `ros2_control` 제어 루프의 주기를 설정한다.                  |
| `type`              | `controllers.yaml` | `velocity_controller`      | 사용할 컨트롤러 알고리즘(`velocity_controllers/JointGroupVelocityController`)을 정의한다. |
| `joints`            | `controllers.yaml` | `velocity_controller`      | 해당 컨트롤러가 관리할 관절 목록을 지정한다.                 |

## 4. 오케스트레이터: 시스템 런치 파일

런치 파일은 Gazebo 시뮬레이터, 로봇 모델, `ros2_control` 노드 등 여러 구성요소를 순서에 맞게 실행시키는 역할을 한다.

### 4.1 `robot_launch.py` 파일

`launch/robot_launch.py` 파일을 생성하고 다음 코드를 작성한다.

```Python
import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription, RegisterEventHandler
from launch.event_handlers import OnProcessExit
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch_ros.actions import Node
import xacro

def generate_launch_description():
    pkg_path = os.path.join(get_package_share_directory('simple_velocity_robot'))

    # Process the URDF file
    xacro_file = os.path.join(pkg_path, 'urdf', 'robot.urdf.xacro')
    robot_description_config = xacro.process_file(xacro_file)
    robot_description = {'robot_description': robot_description_config.toxml()}

    # Create a robot_state_publisher node
    robot_state_publisher_node = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        output='screen',
        parameters=[robot_description]
    )

    # Launch Gazebo
    gazebo = IncludeLaunchDescription(
        PythonLaunchDescriptionSource([os.path.join(
            get_package_share_directory('ros_gz_sim'), 'launch', 'gz_sim.launch.py')]),
        launch_arguments={'gz_args': '-r -v 4 empty.sdf'}.items()
    )

    # Spawn robot
    spawn_entity = Node(
        package='ros_gz_sim',
        executable='create',
        arguments=['-topic', 'robot_description',
                   '-entity', 'simple_robot'],
        output='screen'
    )

    # Spawn controllers
    joint_state_broadcaster_spawner = Node(
        package="controller_manager",
        executable="spawner",
        arguments=["joint_state_broadcaster", "--controller-manager", "/controller_manager"],
    )

    velocity_controller_spawner = Node(
        package="controller_manager",
        executable="spawner",
        arguments=["velocity_controller", "--controller-manager", "/controller_manager"],
    )
    
    # Ensure controllers are spawned after Gazebo is ready
    # This is a robust way to handle startup race conditions.
    # We wait for the spawn_entity node to finish, which implies Gazebo is running and the robot model is loaded.
    # Then we spawn the controllers.
    delay_broadcaster_after_spawn = RegisterEventHandler(
        event_handler=OnProcessExit(
            target_action=spawn_entity,
            on_exit=[joint_state_broadcaster_spawner],
        )
    )

    delay_controller_after_broadcaster = RegisterEventHandler(
        event_handler=OnProcessExit(
            target_action=joint_state_broadcaster_spawner,
            on_exit=[velocity_controller_spawner],
        )
    )


    return LaunchDescription([
        gazebo,
        robot_state_publisher_node,
        spawn_entity,
        delay_broadcaster_after_spawn,
        delay_controller_after_broadcaster,
    ])
```

### 4.2 로봇 디스크립션 로딩

- XACRO 파일을 처리하여 완전한 URDF XML 문자열을 생성하고, 이를 `robot_description` 파라미터로 만든다. 이는 ROS2에서 로봇 모델을 전달하는 표준 방식이다.20
- `robot_state_publisher` 노드는 `/joint_states` 토픽을 구독하고 `robot_description`을 사용하여 로봇의 각 링크 간의 좌표 변환 정보(TF)를 `/tf` 토픽으로 발행한다. 이 정보는 RViz와 같은 시각화 도구에서 로봇 모델을 올바르게 표시하는 데 필수적이다.21

### 4.3 Gazebo 실행 및 로봇 스폰

- `IncludeLaunchDescription`을 사용하여 `ros_gz_sim` 패키지에서 제공하는 런치 파일을 포함시켜 Gazebo 시뮬레이터를 실행한다.22
- `ros_gz_sim`의 `create` 실행 파일을 사용하는 `spawn_entity` 노드는 `robot_description` 토픽을 구독하여 해당 모델을 실행 중인 Gazebo 월드에 스폰(생성)한다.21

### 4.4 `ros2_control` 시스템 시작

- `controller_manager`의 `spawner` 실행 파일은 컨트롤러 매니저의 서비스(`/controller_manager/load_and_start_controller`)를 호출하여 YAML 파일에 정의된 컨트롤러를 로드하고 활성화하는 유틸리티다.6

- 단순히 모든 노드를 동시에 시작하면, 컨트롤러 매니저가 준비되기 전에 스포너가 실행되어 실패하는 등의 경쟁 상태(race condition)가 발생할 수 있다. `ros2_control_demos`의 런치 파일들에서 볼 수 있듯이 24, 

  `RegisterEventHandler`와 `OnProcessExit`를 사용하는 것이 훨씬 안정적이다. 이 예제에서는 로봇 스폰(`spawn_entity`)이 완료된 후에 `joint_state_broadcaster`를 스폰하고, 그것이 완료된 후에 `velocity_controller`를 스폰하도록 하여, 각 구성요소가 순서대로 안전하게 시작되도록 보장한다.

## 5. 원동력: 속도 명령 퍼블리셔

이제 로봇을 실제로 움직이게 할 속도 명령을 보내는 간단한 Python 노드를 작성한다.

### 5.1 `velocity_publisher.py` 스크립트

`src/simple_velocity_robot/velocity_publisher.py` 파일을 생성하고 다음 코드를 작성한다.

```Python
import rclpy
from rclpy.node import Node
from std_msgs.msg import Float64MultiArray

class VelocityPublisher(Node):
    def __init__(self):
        super().__init__('velocity_publisher')
        self.publisher_ = self.create_publisher(
            Float64MultiArray,
            '/velocity_controller/commands',
            10)
        timer_period = 0.1  # 10 Hz
        self.timer = self.create_timer(timer_period, self.timer_callback)
        self.get_logger().info('Velocity publisher has been started.')

    def timer_callback(self):
        msg = Float64MultiArray()
        # Set a constant velocity command of 1.5 rad/s
        msg.data = [1.5]
        self.publisher_.publish(msg)
        # self.get_logger().info('Publishing velocity: "%s"' % msg.data)

def main(args=None):
    rclpy.init(args=args)
    velocity_publisher = VelocityPublisher()
    rclpy.spin(velocity_publisher)
    velocity_publisher.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### 5.2 코드 분석

- **임포트**: `rclpy`와 `Node`, 그리고 컨트롤러가 수신할 메시지 타입인 `Float64MultiArray`를 `std_msgs.msg`로부터 임포트한다.25
- **퍼블리셔 생성**: `create_publisher`를 사용하여 퍼블리셔를 초기화한다. 토픽 이름은 `<controller_name>/commands` 규칙에 따라 `/velocity_controller/commands`로 지정해야 한다. 메시지 타입은 `Float64MultiArray`이며, 이는 여러 관절을 동시에 제어할 수 있도록 설계되었기 때문이다.16
- **타이머 콜백**: `create_timer`를 사용하여 0.1초(10 Hz)마다 `timer_callback` 함수를 호출한다. 이를 통해 지속적인 명령 발행이 가능하다.27
- **메시지 발행**: 콜백 함수 내에서 `Float64MultiArray` 메시지 객체를 생성하고, `data` 필드에 원하는 속도값을 리스트 형태로 할당한다. 관절이 하나뿐이라도 반드시 리스트(`[1.5]`)로 전달해야 한다. `publish` 함수를 호출하여 메시지를 발행한다.29
- **main 함수**: ROS2 Python 노드의 표준적인 초기화, 실행(spin), 종료 코드로 구성된다.27

## 6. 조립 및 실행

모든 구성 파일이 준비되었으므로, 패키지를 빌드하고 시뮬레이션을 실행한다.

### 6.1 ROS2 패키지 최종 설정

1. package.xml 설정:

   패키지의 의존성을 명시한다. rclpy, std_msgs와 ros2_control 관련 패키지들을 추가한다.

   ```XML
   <?xml version="1.0"?>
   <package format="3">
     <name>simple_velocity_robot</name>
     <version>0.0.0</version>
     <description>Example of continuous actuator rotation with ros2_control and Gazebo</description>
     <maintainer email="user@todo.todo">user</maintainer>
     <license>Apache-2.0</license>
   
     <buildtool_depend>ament_python</buildtool_depend>
   
     <exec_depend>rclpy</exec_depend>
     <exec_depend>std_msgs</exec_depend>
     <exec_depend>ros2_control</exec_depend>
     <exec_depend>ros2_controllers</exec_depend>
     <exec_depend>gz_ros2_control</exec_depend>
     <exec_depend>robot_state_publisher</exec_depend>
     <exec_depend>ros_gz_sim</exec_depend>
   
     <test_depend>ament_copyright</test_depend>
     <test_depend>ament_flake8</test_depend>
     <test_depend>ament_pep257</test_depend>
     <test_depend>python3-pytest</test_depend>
   </package>
   ```

2. setup.py 설정:

   Python 노드를 실행 파일로 등록하기 위해 entry_points를 설정한다.

   ```Python
   from setuptools import find_packages, setup
   import os
   from glob import glob
   
   package_name = 'simple_velocity_robot'
   
   setup(
       name=package_name,
       version='0.0.0',
       packages=find_packages(exclude=['test']),
       data_files=[
           ('share/ament_index/resource_index/packages',
               ['resource/' + package_name]),
           ('share/' + package_name, ['package.xml']),
           (os.path.join('share', package_name, 'launch'), glob('launch/*.py')),
           (os.path.join('share', package_name, 'urdf'), glob('urdf/*')),
           (os.path.join('share', package_name, 'config'), glob('config/*.yaml')),
       ],
       install_requires=['setuptools'],
       zip_safe=True,
       maintainer='user',
       maintainer_email='user@todo.todo',
       description='Example of continuous actuator rotation with ros2_control and Gazebo',
       license='Apache-2.0',
       tests_require=['pytest'],
       entry_points={
           'console_scripts': [
               'velocity_publisher = simple_velocity_robot.velocity_publisher:main',
           ],
       },
   )
   ```

### 6.2 시뮬레이션 빌드 및 실행

1. 워크스페이스 빌드:

   ros2_ws 디렉토리로 이동하여 colcon으로 패키지를 빌드한다.

   ```Bash
   cd ~/ros2_ws
   colcon build --symlink-install --packages-select simple_velocity_robot
   ```

2. 워크스페이스 소싱:

   빌드된 패키지를 현재 터미널 환경에 등록한다.

   ```Bash
   source install/setup.bash
   ```

3. 시뮬레이션 실행:

   작성한 런치 파일을 실행하여 Gazebo와 ros2_control 시스템을 시작한다.

   ```Bash
   ros2 launch simple_velocity_robot robot_launch.py
   ```

   Gazebo 창이 열리고, 로봇 모델이 스폰되는 것을 확인할 수 있다.

4. 명령 퍼블리셔 실행:

   별도의 터미널을 열고 워크스페이스를 소싱한 후, 속도 명령 퍼블리셔 노드를 실행한다.

   ```Bash
   source ~/ros2_ws/install/setup.bash
   ros2 run simple_velocity_robot velocity_publisher
   ```

   이제 Gazebo 시뮬레이션에서 로봇의 `rotating_link`가 일정한 속도로 회전하기 시작한다.

### 6.3 검증 및 내부 상태 확인

시스템이 올바르게 동작하는지 확인하기 위해 `ros2 control` 및 `ros2 topic` CLI 도구를 사용한다.

- **컨트롤러 상태 확인**:

  ```Bash
  ros2 control list_controllers
  ```

  `joint_state_broadcaster`와 `velocity_controller`가 모두 `active` 상태로 표시되어야 한다.16

- **하드웨어 인터페이스 상태 확인**:

  ```Bash
  ros2 control list_hardware_interfaces
  ```

  `revolute_joint`의 `velocity` 명령 인터페이스가 `[claimed]` 상태로 표시되어 `velocity_controller`에 의해 점유되었음을 확인해야 한다.16

- **관절 상태 토픽 확인**:

  ```Bash
  ros2 topic echo /joint_states
  ```

  `position` 값이 지속적으로 증가(또는 감소)하는 것을 통해 관절이 실제로 회전하고 있음을 확인할 수 있다.

## 7. 이론적 토대 및 심층 분석

이 간단한 예제는 `ros2_control`과 Gazebo의 상호작용에 대한 중요한 이론적 개념을 내포하고 있다.

### 7.1 `ros2_control` 실시간 루프

컨트롤러 매니저는 `update_rate`에 맞춰 주기적으로 실시간 제어 루프를 실행한다. 이 루프는 세 단계로 구성된다.2

1. **`read()`**: `gz_ros2_control` 플러그인이 Gazebo의 물리 엔진으로부터 현재 관절 상태(위치, 속도)를 읽어와 `ros2_control`의 상태 인터페이스를 갱신한다.
2. **`update()`**: 활성화된 모든 컨트롤러의 `update()` 함수가 호출된다. 이 예제에서 `velocity_controller`는 `/velocity_controller/commands` 토픽에서 수신한 최신 명령값을 자신의 출력으로 설정한다.
3. **`write()`**: `gz_ros2_control` 플러그인이 컨트롤러의 출력(명령 인터페이스의 값)을 읽어와 Gazebo의 관절에 적용한다.

### 7.2 숨겨진 제어 법칙: 시뮬레이션 내부의 PID

여기서 한 가지 중요한 질문이 생긴다. 우리는 `velocity` (rad/s)를 명령했지만, 물리 시뮬레이터인 Gazebo는 실제로 관절에 `force`나 `torque`를 가해야 움직임을 만들어낼 수 있다. 그렇다면 속도 명령은 어떻게 토크로 변환되는가?

그 해답은 `gz_ros2_control` 플러그인 내부에 숨겨진 PID(Proportional-Integral-Derivative) 제어기에 있다.11 사용자가 속도, 위치 등 상위 레벨의 명령을 내리면, Gazebo 플러그인은 내부적으로 이 목표값을 달성하기 위한 저수준의 힘/토크를 계산한다.

이 과정은 다음과 같다.

1. 사용자가 `velocity_controller`를 통해 목표 속도(q˙desired)를 명령한다.

2. `gz_ros2_control` 플러그인은 `read()` 단계에서 Gazebo 물리 엔진으로부터 현재 실제 속도(q˙actual)를 측정한다.

3. 플러그인은 목표 속도와 실제 속도 간의 오차 `$e(t) = \dot{q}_{desired}(t) - \dot{q}_{actual}(t)$`를 계산한다.

4. 내장된 PID 제어기는 이 오차를 기반으로 관절에 적용할 토크 `$u(t)$`를 다음의 표준적인 PID 제어 법칙에 따라 계산한다.
   $$
   u(t) = K_p e(t) + K_i \int_0^t e(\tau) d\tau + K_d \frac{de(t)}{dt}
   $$
   계산된 토크 `$u(t)$`가 `write()` 단계에서 Gazebo의 관절에 최종적으로 적용된다.

여기서 `$K_p$`, `$K_i$`, `$K_d$`는 각각 비례, 적분, 미분 이득(gain)으로, 플러그인 내부에 기본값이 설정되어 있다. 이처럼 `ros2_control`과 Gazebo 시뮬레이션의 조합은 사용자가 고수준의 제어(속도 제어)에 집중할 수 있도록 저수준의 동역학 제어(토크 제어)를 추상화하여 제공한다. 이는 시뮬레이션 환경을 실제 하드웨어와 유사하게 만들어주며, 제어기 개발 및 테스트를 매우 효율적으로 만들어주는 핵심적인 기능이다.

#### **참고 자료**

1. velocity_controllers - ROS2_Control: Rolling Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/rolling/doc/ros2_controllers/velocity_controllers/doc/userdoc.html
2. Getting Started - ROS2_Control: Humble Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/humble/doc/getting_started/getting_started.html
3. Getting Started - ROS2_Control: Foxy Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/foxy/doc/getting_started/getting_started.html
4. gz_ros2_control - ROS2_Control: Rolling Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/rolling/doc/gz_ros2_control/doc/index.html
5. gz_ros2_control - ROS2_Control: Humble Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/humble/doc/gz_ros2_control/doc/index.html
6. ros2_control Concepts & Simulation - Articulated Robotics, accessed July 27, 2025, https://articulatedrobotics.xyz/tutorials/mobile-robot/applications/ros2_control-concepts/
7. Installing Gazebo with ROS - Gazebo ionic documentation, accessed July 27, 2025, https://gazebosim.org/docs/latest/ros_installation/
8. Welcome to the ros2_control documentation - Humble!, accessed July 27, 2025, https://control.ros.org/humble/index.html
9. Welcome to the ros2_control documentation! - ROS2_Control: Rolling Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/
10. ros-controls/gazebo_ros2_control: Wrappers, tools and additional API's for using ros2_control with Gazebo Classic - GitHub, accessed July 27, 2025, https://github.com/ros-controls/gazebo_ros2_control
11. gazebo_ros2_control - ROS2_Control: Humble Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/humble/doc/gazebo_ros2_control/doc/index.html
12. Demos - ROS2_Control: Rolling Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/rolling/doc/ros2_control_demos/doc/index.html
13. Example 7: Full tutorial with a 6DOF robot - ROS2_Control, accessed July 27, 2025, https://control.ros.org/rolling/doc/ros2_control_demos/example_7/doc/userdoc.html
14. Using a URDF in Gazebo - ROS 2 Documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Using-a-URDF-in-Gazebo.html
15. Controller Manager - ROS2_Control: Humble Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/humble/doc/ros2_control/controller_manager/doc/userdoc.html
16. Example 3: Robots with multiple interfaces - ROS2_Control: Rolling Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/rolling/doc/ros2_control_demos/example_3/doc/userdoc.html
17. Example 9: Simulation with RRBot - ROS2_Control: Humble Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/humble/doc/ros2_control_demos/example_9/doc/userdoc.html
18. forward_command_controller - ROS2_Control: Humble Jul 2025 ..., accessed July 27, 2025, https://control.ros.org/humble/doc/ros2_controllers/forward_command_controller/doc/userdoc.html
19. velocity_controllers - ROS2_Control: Humble Jul 2025 documentation, accessed July 27, 2025, https://control.ros.org/humble/doc/ros2_controllers/velocity_controllers/doc/userdoc.html
20. ros2_control extra bits - Articulated Robotics, accessed July 27, 2025, https://articulatedrobotics.xyz/tutorials/mobile-robot/applications/ros2_control-extra/
21. Tutorial: How to Spawn a Robot in a Custom Gazebo World using ROS2 Humble - Medium, accessed July 27, 2025, https://medium.com/@ayaan.cdry/tutorial-how-to-spawn-a-robot-in-a-custom-gazebo-world-using-ros2-humble-e4a2f3c6e95b
22. Spawn a Gazebo model from ROS 2, accessed July 27, 2025, https://gazebosim.org/docs/latest/ros2_spawn_model/
23. Launch Gazebo from ROS 2, accessed July 27, 2025, https://gazebosim.org/docs/latest/ros2_launch_gazebo/
24. rrbot.launch.py - ros-controls/ros2_control_demos - GitHub, accessed July 27, 2025, https://github.com/ros-controls/ros2_control_demos/blob/master/example_1/bringup/launch/rrbot.launch.py
25. std_msgs/msg/Float64MultiArray Message - ROS Documentation, accessed July 27, 2025, https://docs.ros2.org/foxy/api/std_msgs/msg/Float64MultiArray.html
26. Float64MultiArray - std_msgs: Humble 4.9.0 documentation - ROS 2, accessed July 27, 2025, https://docs.ros.org/en/humble/p/std_msgs/msg/Float64MultiArray.html
27. Writing a simple publisher and subscriber (Python) - ROS 2 Documentation, accessed July 27, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html
28. ROS2 Python Publisher Example - The Robotics Back-End, accessed July 27, 2025, https://roboticsbackend.com/ros2-python-publisher-example/
29. ROS publishing array from python node - Stack Overflow, accessed July 27, 2025, https://stackoverflow.com/questions/31369934/ros-publishing-array-from-python-node
30. How To Write a ROS2 Publisher and Subscriber (Python) – Foxy, accessed July 27, 2025, https://automaticaddison.com/how-to-write-a-ros2-publisher-and-subscriber-python-foxy/
31. ros2_control_demos/example_2/doc/userdoc.rst at master / ros-controls ... - GitHub, accessed July 27, 2025, https://github.com/ros-controls/ros2_control_demos/blob/master/example_2/doc/userdoc.rst
32. Example 7: Full tutorial with a 6DOF robot - ROS2_Control, accessed July 27, 2025, https://control.ros.org/humble/doc/ros2_control_demos/example_7/doc/userdoc.html
33. Tutorial : ROS control - Gazebo Classic, accessed July 27, 2025, https://classic.gazebosim.org/tutorials?tut=ros_control


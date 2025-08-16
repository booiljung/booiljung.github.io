# ROS 2 Humble 기반 전방향 4륜 스위블 드라이브 로봇 개발: 종합 시뮬레이션 프로젝트





## 1.  스위블 드라이브 시스템의 기초 운동학





### 1.1  스위블 드라이브 운동학 소개



로봇 공학에서 이동 플랫폼의 설계는 그 응용 범위를 결정하는 핵심 요소입니다. 일반적인 차동 구동(Differential Drive) 방식은 구현이 간단하지만 측면 이동이 불가능하며, 애커만 조향(Ackermann Steering) 방식은 고속 주행 시 안정적이지만 제자리 회전이나 전방향 이동에 제약이 있습니다. 이러한 한계를 극복하기 위해 고안된 구동 방식 중 하나가 바로 **스위블 드라이브(Swerve Drive)**입니다.

스위블 드라이브는 각 바퀴 모듈이 독립적인 구동 모터와 조향 모터를 가지는 구조입니다. 이 구조 덕분에 각 바퀴의 방향과 속도를 개별적으로 제어할 수 있어, 로봇은 전후좌우 직선 이동, 제자리 회전, 그리고 이 모든 움직임을 조합한 복합적인 궤적 이동, 즉 **전방향(Omnidirectional) 이동**이 가능해집니다. 이러한 탁월한 기동성은 물류 창고, 병원, 서비스 환경 등 복잡하고 좁은 공간에서 로봇의 활용성을 극대화합니다.

본 프로젝트의 목표는 이러한 스위블 드라이브 로봇을 ROS 2 환경에서 시뮬레이션하고 제어하는 것입니다. 이를 위해 가장 먼저 해결해야 할 과제는 운동학(Kinematics) 모델을 정립하는 것입니다. 구체적으로, 로봇 전체에 원하는 선속도(vx,vy)와 각속도(ωz)가 주어졌을 때, 각 네 개의 스위블 모듈이 가져야 할 조향 각도(ϕi)와 구동 속도(vi)를 계산하는 **역운동학(Inverse Kinematics)**을 유도하는 것이 이 섹션의 핵심 목표입니다.



### 1.2  좌표계 및 시스템 변수 정의



정확한 운동학 모델을 수립하기 위해, 먼저 로봇 시스템의 좌표계와 변수들을 명확히 정의해야 합니다. 본 보고서에서는 ROS Enhancement Proposals(REP) 103에 명시된 좌표계 규칙을 준수하여 일관성을 유지합니다.1

- **좌표계:**
  - `base_link`: 로봇의 기하학적 중심에 원점을 둔 로봇 고정 좌표계입니다. 로봇의 전방을 x축, 좌측을 y축, 상방을 z축으로 정의합니다.
  - `odom`: 로봇의 초기 위치에 고정된 월드 좌표계(World-fixed frame)로, 로봇의 주행 거리(Odometry)를 계산하는 기준이 됩니다.
- **로봇 전체의 운동 변수:**
  - vb,x: `base_link` 좌표계 기준 로봇의 x축 방향 선속도 (m/s)
  - vb,y: `base_link` 좌표계 기준 로봇의 y축 방향 선속도 (m/s)
  - ωb,z: `base_link` 좌표계 기준 로봇의 z축 중심 회전 각속도 (rad/s)
  - V![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.471em" height="0.714em" style="width:0.471em" viewBox="0 0 471 714" preserveAspectRatio="xMinYMin"><path d="M377 20c0-5.333 1.833-10 5.5-14S391 0 397 0c4.667 0 8.667 1.667 12 5 3.333 2.667 6.667 9 10 19 6.667 24.667 20.333 43.667 41 57 7.333 4.667 11 10.667 11 18 0 6-1 10-3 12s-6.667 5-14 9c-28.667 14.667-53.667 35.667-75 63 -1.333 1.333-3.167 3.5-5.5 6.5s-4 4.833-5 5.5c-1 .667-2.5 1.333-4.5 2s-4.333 1 -7 1c-4.667 0-9.167-1.833-13.5-5.5S337 184 337 178c0-12.667 15.667-32.333 47-59 H213l-171-1c-8.667-6-13-12.333-13-19 0-4.667 4.333-11.333 13-20h359 c-16-25.333-24-45-24-59z"></path></svg>)b=[vb,xvb,yωb,z]T: 로봇의 목표 속도 벡터
- **로봇의 물리적 상수:**
  - L: 축간 거리(Wheelbase), 즉 앞바퀴와 뒷바퀴 축 사이의 거리 (m)
  - W: 윤거(Track width), 즉 왼쪽 바퀴와 오른쪽 바퀴 사이의 거리 (m)
  - r: 바퀴의 반지름 (m)
- **개별 스위블 모듈의 변수 (i = 1, 2, 3, 4):**
  - (li,x,li,y): `base_link` 원점으로부터 각 스위블 모듈 i의 중심까지의 위치 벡터.
    - 모듈 1 (Front-Right): (L/2,−W/2)
    - 모듈 2 (Front-Left): (L/2,W/2)
    - 모듈 3 (Back-Left): (−L/2,W/2)
    - 모듈 4 (Back-Right): (−L/2,−W/2)
  - ϕi: 스위블 모듈 i의 조향 각도 (rad). `base_link`의 x축을 기준으로 반시계 방향을 양수로 정의합니다.
  - vi: 스위블 모듈 i의 구동 바퀴 속도 (m/s)
  - ωi: 스위블 모듈 i의 구동 바퀴 각속도 (rad/s), vi=ωi⋅r



### 1.3  역운동학 상세 유도



역운동학은 로봇 전체의 목표 속도(V![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.471em" height="0.714em" style="width:0.471em" viewBox="0 0 471 714" preserveAspectRatio="xMinYMin"><path d="M377 20c0-5.333 1.833-10 5.5-14S391 0 397 0c4.667 0 8.667 1.667 12 5 3.333 2.667 6.667 9 10 19 6.667 24.667 20.333 43.667 41 57 7.333 4.667 11 10.667 11 18 0 6-1 10-3 12s-6.667 5-14 9c-28.667 14.667-53.667 35.667-75 63 -1.333 1.333-3.167 3.5-5.5 6.5s-4 4.833-5 5.5c-1 .667-2.5 1.333-4.5 2s-4.333 1 -7 1c-4.667 0-9.167-1.833-13.5-5.5S337 184 337 178c0-12.667 15.667-32.333 47-59 H213l-171-1c-8.667-6-13-12.333-13-19 0-4.667 4.333-11.333 13-20h359 c-16-25.333-24-45-24-59z"></path></svg>)b)를 개별 모듈의 조향각(ϕi)과 구동 속도(vi)로 변환하는 과정입니다. 이는 제어기의 핵심 로직이 됩니다. 유도 과정은 다음 두 단계로 나뉩니다.1

**1단계: 각 스위블 모듈의 속도 벡터 계산**

로봇의 특정 지점에서의 속도는 로봇 전체의 병진 운동(translation)에 의한 속도와 회전 운동(rotation)에 의한 접선 속도의 벡터 합으로 표현됩니다. 따라서 `base_link` 원점을 기준으로 (li,x,li,y) 위치에 있는 스위블 모듈 i의 속도 벡터 v![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.471em" height="0.714em" style="width:0.471em" viewBox="0 0 471 714" preserveAspectRatio="xMinYMin"><path d="M377 20c0-5.333 1.833-10 5.5-14S391 0 397 0c4.667 0 8.667 1.667 12 5 3.333 2.667 6.667 9 10 19 6.667 24.667 20.333 43.667 41 57 7.333 4.667 11 10.667 11 18 0 6-1 10-3 12s-6.667 5-14 9c-28.667 14.667-53.667 35.667-75 63 -1.333 1.333-3.167 3.5-5.5 6.5s-4 4.833-5 5.5c-1 .667-2.5 1.333-4.5 2s-4.333 1 -7 1c-4.667 0-9.167-1.833-13.5-5.5S337 184 337 178c0-12.667 15.667-32.333 47-59 H213l-171-1c-8.667-6-13-12.333-13-19 0-4.667 4.333-11.333 13-20h359 c-16-25.333-24-45-24-59z"></path></svg>)i=[vi,xvi,y]T는 다음과 같이 계산됩니다.

[vi,xvi,y]=[vb,xvb,y]+[−ωb,z⋅li,yωb,z⋅li,x]=[vb,x−ωb,zli,yvb,y+ωb,zli,x]

이 식은 각 모듈의 속도가 로봇의 선속도(vb,x,vb,y)와 각속도(ωb,z)에 의해 어떻게 결정되는지를 명확히 보여줍니다.

**2단계: 바퀴 속도 및 조향각 계산**

1단계에서 계산된 각 모듈의 속도 벡터 v![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="0.471em" height="0.714em" style="width:0.471em" viewBox="0 0 471 714" preserveAspectRatio="xMinYMin"><path d="M377 20c0-5.333 1.833-10 5.5-14S391 0 397 0c4.667 0 8.667 1.667 12 5 3.333 2.667 6.667 9 10 19 6.667 24.667 20.333 43.667 41 57 7.333 4.667 11 10.667 11 18 0 6-1 10-3 12s-6.667 5-14 9c-28.667 14.667-53.667 35.667-75 63 -1.333 1.333-3.167 3.5-5.5 6.5s-4 4.833-5 5.5c-1 .667-2.5 1.333-4.5 2s-4.333 1 -7 1c-4.667 0-9.167-1.833-13.5-5.5S337 184 337 178c0-12.667 15.667-32.333 47-59 H213l-171-1c-8.667-6-13-12.333-13-19 0-4.667 4.333-11.333 13-20h359 c-16-25.333-24-45-24-59z"></path></svg>)i는 해당 모듈의 바퀴가 향해야 할 방향과 그 방향으로의 속력을 의미합니다. 따라서, 이 벡터의 크기와 방향을 계산하면 우리가 원하는 바퀴의 구동 속도와 조향 각도를 얻을 수 있습니다.

- **바퀴 구동 속도 (vi):** 속도 벡터의 크기(magnitude)로 계산합니다.

  vi=vi,x2+vi,y2![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="400em" height="1.88em" viewBox="0 0 400000 1944" preserveAspectRatio="xMinYMin slice"><path d="M983 90 l0 -0 c4,-6.7,10,-10,18,-10 H400000v40 H1013.1s-83.4,268,-264.1,840c-180.7,572,-277,876.3,-289,913c-4.7,4.7,-12.7,7,-24,7 s-12,0,-12,0c-1.3,-3.3,-3.7,-11.7,-7,-25c-35.3,-125.3,-106.7,-373.3,-214,-744 c-10,12,-21,25,-33,39s-32,39,-32,39c-6,-5.3,-15,-14,-27,-26s25,-30,25,-30 c26.7,-32.7,52,-63,76,-91s52,-60,52,-60s208,722,208,722 c56,-175.3,126.3,-397.3,211,-666c84.7,-268.7,153.8,-488.2,207.5,-658.5 c53.7,-170.3,84.5,-266.8,92.5,-289.5z M1001 80h400000v40h-400000z"></path></svg>)

- **바퀴 조향 각도 (ϕi):** 속도 벡터의 방향(angle)으로 계산하며, `atan2` 함수를 사용하여 사분면을 정확히 고려합니다.

  ϕi=atan2(vi,y,vi,x)

이 두 공식을 네 개의 바퀴에 대해 각각 적용하면, 로봇의 목표 움직임을 구현하기 위한 8개(조향 4, 구동 4)의 액추에이터 명령값을 모두 얻을 수 있습니다. 예를 들어, 전방-우측(Front-Right, i=1) 모듈의 경우, $(l_{1,x}, l_{1,y}) = (L/2, -W/2)$를 대입하여 다음과 같이 계산합니다.

- v1,x=vb,x−ωb,z(−W/2)=vb,x+ωb,zW/2
- v1,y=vb,y+ωb,z(L/2)=vb,y+ωb,zL/2
- v1=(vb,x+ωb,zW/2)2+(vb,y+ωb,zL/2)2![img](data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="400em" height="1.28em" viewBox="0 0 400000 1296" preserveAspectRatio="xMinYMin slice"><path d="M263,681c0.7,0,18,39.7,52,119 c34,79.3,68.167,158.7,102.5,238c34.3,79.3,51.8,119.3,52.5,120 c340,-704.7,510.7,-1060.3,512,-1067 l0 -0 c4.7,-7.3,11,-11,19,-11 H40000v40H1012.3 s-271.3,567,-271.3,567c-38.7,80.7,-84,175,-136,283c-52,108,-89.167,185.3,-111.5,232 c-22.3,46.7,-33.8,70.3,-34.5,71c-4.7,4.7,-12.3,7,-23,7s-12,-1,-12,-1 s-109,-253,-109,-253c-72.7,-168,-109.3,-252,-110,-252c-10.7,8,-22,16.7,-34,26 c-22,17.3,-33.3,26,-34,26s-26,-26,-26,-26s76,-59,76,-59s76,-60,76,-60z M1001 80h400000v40h-400000z"></path></svg>)
- ϕ1=atan2(vb,y+ωb,zL/2,vb,x+ωb,zW/2)

나머지 세 바퀴에 대해서도 동일한 방식으로 계산할 수 있습니다.

여기서 스위블 드라이브와 애커만 조향의 근본적인 차이점을 인지하는 것이 중요합니다. 애커만 조향은 모든 바퀴가 하나의 공통된 순간 회전 중심(Instantaneous Center of Rotation, ICR)을 가지도록 조향각을 조절하여 타이어 슬립을 최소화하는 비-홀로노믹(non-holonomic) 제약 조건을 가집니다.2 이로 인해 전방향 이동이 불가능합니다. 반면, 스위블 드라이브는 각 바퀴의 독립적인 제어를 통해 이러한 제약에서 벗어나 완전한 홀로노믹(holonomic) 또는 전방향 움직임을 구현합니다. 따라서 본 프로젝트에서는 애커만 조향 컨트롤러(

`ackermann_steering_controller`)가 아닌, 위에서 유도한 스위블 드라이브 역운동학을 직접 구현하는 방식을 채택합니다.3



### 1.4  주행 거리 계산을 위한 순운동학 개념



역운동학이 제어에 필수적이라면, 순운동학(Forward Kinematics)은 로봇의 현재 상태, 특히 주행 거리(Odometry)를 추정하는 데 사용됩니다. 순운동학은 역운동학의 반대 과정으로, 각 바퀴의 실제 조향각과 구동 속도(엔코더 센서로부터 측정된 값)를 바탕으로 로봇 전체의 속도(vb,x,vb,y,ωb,z)를 추정합니다.

스위블 드라이브의 경우, 4개의 조향각과 4개의 구동 속도, 총 8개의 센서 입력으로부터 3개의 변수(vb,x,vb,y,ωb,z)를 추정해야 합니다. 이는 과결정 시스템(overdetermined system)으로, 모든 센서 값이 이상적이라면 여러 조합으로 해를 구할 수 있지만 실제로는 센서 오차와 슬립 등으로 인해 값이 일치하지 않습니다.1 따라서 일반적으로 최소 자승법(Least Squares Method)과 같은 통계적 기법을 사용하여 가장 가능성 높은 로봇의 속도를 추정하게 됩니다. 본 프로젝트의 제어 노드에서는 순운동학을 직접 구현하지는 않지만, 향후 로봇의 주행 거리를 정밀하게 추정하고 Nav2 스택과 연동하기 위해서는 이 순운동학에 대한 이해가 필수적입니다.



## 2.  ROS 2 프로젝트 환경 구축



이론적 기반을 다졌으니, 이제 실제 코드를 작성하고 시뮬레이션을 실행할 프로젝트 환경을 구축합니다. 잘 구조화된 ROS 2 패키지는 코드의 재사용성, 유지보수성, 협업 효율성을 크게 향상시킵니다.



### 2.1  워크스페이스 및 패키지 생성



먼저, 프로젝트를 담을 ROS 2 워크스페이스를 생성합니다. 그 후, 워크스페이스의 `src` 디렉토리로 이동하여 `ros2 pkg create` 명령어를 사용해 새로운 패키지를 만듭니다.5

Bash

```
# 1. 워크스페이스 디렉토리 생성 및 이동
mkdir -p ~/swerve_ws/src
cd ~/swerve_ws/src

# 2. ament_cmake 타입의 ROS 2 패키지 생성
ros2 pkg create --build-type ament_cmake --license Apache-2.0 \
  --node-name swerve_controller_node swerve_bot_description
```

여기서 몇 가지 중요한 선택이 이루어졌습니다.

- **패키지 이름:** `swerve_bot_description`으로 지정했습니다. 단일 로봇 시뮬레이션 프로젝트의 경우, 로봇 모델, 제어 설정, 런치 파일 등을 하나의 패키지에 통합 관리하는 것이 효율적입니다. 이는 `ros2_control_demos` 등 여러 예제에서 볼 수 있는 실용적인 접근 방식입니다.7
- **빌드 타입:** `ament_cmake`를 선택했습니다. 운동학 계산과 같이 실시간성과 성능이 중요한 제어 로직은 C++로 작성하는 것이 유리하기 때문입니다.8 런치 파일은 Python으로 작성되지만, 패키지 전체의 빌드 시스템은 C++ 코드를 컴파일하기 위해 CMake를 기반으로 해야 합니다.
- **노드 이름:** `--node-name swerve_controller_node` 옵션을 통해 `src` 디렉토리에 C++ 노드 템플릿 파일을 자동으로 생성했습니다. 이 파일은 섹션 5에서 수정하여 사용할 것입니다.



### 2.2  표준 시뮬레이션 패키지 구조 정의



패키지 내부에 체계적인 디렉토리 구조를 만드는 것은 프로젝트 관리에 매우 중요합니다. ROS 2 시뮬레이션 프로젝트에서 일반적으로 사용되는 구조는 다음과 같습니다.9

```
swerve_bot_description/
├── CMakeLists.txt
├── package.xml
├── src/
│   └── swerve_controller_node.cpp
├── launch/
│   └── launch_sim.launch.py
├── urdf/
│   ├── swerve_bot.urdf.xacro
│   ├── swerve_module.xacro
│   └── swerve_bot.ros2_control.xacro
├── config/
│   └── swerve_bot_controllers.yaml
├── worlds/
│   └── empty.world
└── rviz/
    └── swerve_bot.rviz
```

각 디렉토리의 역할은 다음과 같습니다.

- `src/`: C++ 소스 코드를 저장합니다.
- `launch/`: 시스템을 실행하는 런치 파일(`.launch.py`)을 저장합니다.
- `urdf/`: 로봇 모델을 정의하는 URDF 및 Xacro 파일(`.urdf.xacro`)을 저장합니다.
- `config/`: 컨트롤러 파라미터와 같은 YAML 설정 파일(`.yaml`)을 저장합니다.
- `worlds/`: Gazebo 시뮬레이션 환경 파일(`.world`)을 저장합니다.
- `rviz/`: RViz 시각화 설정 파일(`.rviz`)을 저장합니다.

`cd ~/swerve_ws/src/swerve_bot_description` 명령으로 패키지 디렉토리로 이동한 후, `mkdir` 명령을 사용하여 위 구조대로 디렉토리를 생성합니다.



### 2.3  `package.xml` 설정



`package.xml` 파일은 패키지의 메타 정보(이름, 버전, 작성자 등)와 의존성을 명시합니다. `ros2 pkg create` 명령으로 기본 파일이 생성되었지만, 우리 프로젝트에 필요한 의존성을 추가해야 합니다.

**`swerve_bot_description/package.xml`**

XML

```
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>swerve_bot_description</name>
  <version>0.0.0</version>
  <description>ROS 2 simulation package for a 4-wheel swerve drive robot.</description>
  <maintainer email="user@todo.todo">user</maintainer>
  <license>Apache-2.0</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <depend>rclcpp</depend>
  <depend>geometry_msgs</depend>
  <depend>std_msgs</depend>
  <depend>xacro</depend>
  <depend>robot_state_publisher</depend>
  <depend>joint_state_publisher_gui</depend>
  <depend>rviz2</depend>
  <depend>gazebo_ros</depend>
  <depend>ros2_control</depend>
  <depend>ros2_controllers</depend>
  <depend>gazebo_ros2_control</depend>

  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
```

이 파일은 C++ 노드 개발(`rclcpp`, `geometry_msgs`), 로봇 모델링(`xacro`), 시뮬레이션(`gazebo_ros`), 제어(`ros2_control`, `ros2_controllers`, `gazebo_ros2_control`), 시각화(`robot_state_publisher`, `rviz2`)에 필요한 모든 패키지를 의존성으로 선언합니다.9



### 2.4  `CMakeLists.txt` 설정



`CMakeLists.txt` 파일은 패키지를 빌드하는 방법을 정의합니다. C++ 실행 파일 컴파일, 라이브러리 링크, 그리고 `launch`, `urdf`, `config` 등의 리소스 파일들을 설치하는 규칙을 포함해야 합니다.

**`swerve_bot_description/CMakeLists.txt`**

CMake

```
cmake_minimum_required(VERSION 3.8)
project(swerve_bot_description)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(std_msgs REQUIRED)
find_package(xacro REQUIRED)

# Build the C++ node
add_executable(swerve_controller_node src/swerve_controller_node.cpp)
ament_target_dependencies(swerve_controller_node rclcpp geometry_msgs std_msgs)

# Install the C++ node
install(TARGETS
  swerve_controller_node
  DESTINATION lib/${PROJECT_NAME}
)

# Install resource files
install(
  DIRECTORY urdf launch config worlds rviz
  DESTINATION share/${PROJECT_NAME}
)

# Install package.xml
install(FILES package.xml
  DESTINATION share/${PROJECT_NAME}
)

# Ament linting for tests
if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
```

이 설정 파일의 핵심적인 부분은 다음과 같습니다.

- `find_package(...)`: `package.xml`에 명시된 의존성 패키지들을 찾습니다.
- `add_executable(...)`: `src/swerve_controller_node.cpp` 파일을 컴파일하여 `swerve_controller_node`라는 실행 파일을 생성합니다.
- `ament_target_dependencies(...)`: 생성된 실행 파일이 `rclcpp`와 같은 ROS 2 라이브러리에 링크되도록 합니다.
- `install(TARGETS...)`: 빌드된 실행 파일을 `install/swerve_bot_description/lib` 디렉토리에 설치하여 `ros2 run` 명령으로 실행할 수 있게 합니다.
- `install(DIRECTORY...)`: `urdf`, `launch`, `config` 등의 리소스 디렉토리들을 `install/swerve_bot_description/share` 디렉토리에 설치합니다. 이 과정은 런치 파일이 런타임에 모델 및 설정 파일을 찾을 수 있도록 하는 매우 중요한 단계입니다.

이제 워크스페이스 루트(`~/swerve_ws`)로 돌아가 `colcon build` 명령을 실행하면, 설정한 구조와 내용에 따라 패키지가 성공적으로 빌드될 것입니다.



## 3.  URDF와 Xacro를 이용한 로봇 모델링



로봇의 물리적 형상, 관절 구조, 센서 위치 등을 정의하는 로봇 모델은 시뮬레이션과 제어의 근간을 이룹니다. ROS에서는 URDF(Unified Robot Description Format)라는 XML 기반 형식으로 로봇을 기술합니다. 하지만 복잡한 로봇을 순수 URDF로 작성하는 것은 매우 번거롭고 반복적인 작업이 많습니다.13 이를 해결하기 위해 XML 매크로 언어인 **Xacro(XML Macros)**를 사용하여 모듈화되고 유지보수하기 쉬운 로봇 모델을 작성하는 것이 표준적인 방법입니다.14



### 3.1  베이스 섀시 및 링크 설계



먼저 로봇의 몸통이 되는 `base_link`와, 이 섀시에 연결되어 회전하는 4개의 조향 링크(예: `front_left_steering_link`)를 정의합니다. 각 링크는 시각적 표현을 위한 `<visual>` 태그, 물리 엔진과의 상호작용을 위한 `<collision>` 태그, 그리고 질량과 관성 모멘트를 정의하는 `<inertial>` 태그를 가집니다.

정확한 시뮬레이션을 위해서는 이 세 가지 요소를 모두 충실하게 정의하는 것이 매우 중요합니다. `<visual>`만 정의하면 로봇이 보이기만 할 뿐, 물리적으로 존재하지 않는 유령처럼 동작하게 됩니다. `<collision>`은 물리 엔진이 충돌을 계산할 때 사용하는 기하학적 형태로, 성능을 위해 보통 `<visual>`보다 단순한 형태(실린더, 박스 등)를 사용합니다. `<inertial>`은 로봇의 동역학적 거동을 결정하므로, 실제 로봇과 유사한 시뮬레이션 결과를 얻기 위해 필수적입니다.13



### 3.2  Xacro의 핵심: 속성과 재사용 가능한 매크로



Xacro의 가장 강력한 기능은 상수(속성)와 매크로를 사용하는 것입니다. 이를 통해 반복적인 코드를 줄이고 모델의 파라미터를 한 곳에서 관리할 수 있습니다.14

**표 1: 주요 로봇 물리 파라미터 (Xacro 속성)**

| 속성 이름 (`name`)   | 값 (`value`) | 설명                                    |
| -------------------- | ------------ | --------------------------------------- |
| `chassis_length`     | 0.4          | 섀시의 길이 (x축 방향) [m]              |
| `chassis_width`      | 0.3          | 섀시의 폭 (y축 방향) [m]                |
| `chassis_height`     | 0.1          | 섀시의 높이 (z축 방향) [m]              |
| `chassis_mass`       | 5.0          | 섀시의 질량 [kg]                        |
| `wheelbase`          | 0.35         | 축간 거리 (앞/뒤 바퀴 중심 간 거리) [m] |
| `track_width`        | 0.25         | 윤거 (좌/우 바퀴 중심 간 거리) [m]      |
| `steering_link_mass` | 0.2          | 조향 링크의 질량 [kg]                   |
| `wheel_radius`       | 0.05         | 바퀴의 반지름 [m]                       |
| `wheel_thickness`    | 0.04         | 바퀴의 두께 [m]                         |
| `wheel_mass`         | 0.5          | 바퀴의 질량 [kg]                        |

이러한 속성들은 메인 Xacro 파일 상단에 `<xacro:property>` 태그로 정의되며, 파일 내에서는 `${property_name}` 구문을 통해 참조됩니다. 예를 들어, 바퀴의 반지름은 `<cylinder radius="${wheel_radius}"... />`와 같이 사용됩니다. 이렇게 하면 로봇의 크기나 질량을 변경하고 싶을 때, 파일 상단의 속성 값만 수정하면 되므로 매우 편리합니다.



### 3.3  파라미터화된 스위블 모듈 Xacro 매크로 구현



스위블 드라이브 로봇은 4개의 동일한 구조를 가진 모듈로 구성됩니다. 이를 각각 URDF로 작성하는 대신, 하나의 모듈을 정의하는 Xacro 매크로를 만들고 이를 4번 호출하는 것이 훨씬 효율적입니다. 이 접근법은 `ros2_control_demos`와 같은 고급 예제에서 널리 사용되는 모범 사례입니다.15

먼저, `urdf/swerve_module.xacro` 파일을 생성하고 다음과 같이 매크로를 정의합니다.

**`urdf/swerve_module.xacro`**

XML

```
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro">

  <xacro:macro name="swerve_module" params="prefix parent *origin">
    <link name="${prefix}_steering_link">
      <visual>
        <geometry>
          <box size="0.02 0.02 0.05"/>
        </geometry>
        <material name="grey"/>
      </visual>
      <collision>
        <geometry>
          <box size="0.02 0.02 0.05"/>
        </geometry>
      </collision>
      <xacro:inertial_box mass="${steering_link_mass}" x="0.02" y="0.02" z="0.05">
        <origin xyz="0 0 0" rpy="0 0 0"/>
      </xacro:inertial_box>
    </link>

    <joint name="${prefix}_steering_joint" type="revolute">
      <parent link="${parent}"/>
      <child link="${prefix}_steering_link"/>
      <xacro:insert_block name="origin"/>
      <axis xyz="0 0 1"/>
      <limit lower="-3.1415" upper="3.1415" effort="10" velocity="10"/>
    </joint>

    <link name="${prefix}_wheel">
      <visual>
        <origin xyz="0 0 0" rpy="1.5707 0 0"/>
        <geometry>
          <cylinder radius="${wheel_radius}" length="${wheel_thickness}"/>
        </geometry>
        <material name="black"/>
      </visual>
      <collision>
        <origin xyz="0 0 0" rpy="1.5707 0 0"/>
        <geometry>
          <cylinder radius="${wheel_radius}" length="${wheel_thickness}"/>
        </geometry>
      </collision>
      <xacro:inertial_cylinder mass="${wheel_mass}" radius="${wheel_radius}" length="${wheel_thickness}">
        <origin xyz="0 0 0" rpy="1.5707 0 0"/>
      </xacro:inertial_cylinder>
    </link>

    <joint name="${prefix}_drive_joint" type="continuous">
      <parent link="${prefix}_steering_link"/>
      <child link="${prefix}_wheel"/>
      <origin xyz="0 0 -0.025" rpy="0 0 0"/>
      <axis xyz="0 1 0"/>
    </joint>
  </xacro:macro>

</robot>
```

이 매크로는 `prefix`(예: "front_left"), `parent`(부모 링크), `origin`(모듈 위치)을 파라미터로 받습니다. 이제 메인 로봇 파일인 `swerve_bot.urdf.xacro`에서 이 매크로를 포함하고 호출합니다.

**`urdf/swerve_bot.urdf.xacro` (일부)**

XML

```
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro" name="swerve_bot">

  <xacro:include filename="$(find swerve_bot_description)/urdf/materials.xacro" />
  <xacro:include filename="$(find swerve_bot_description)/urdf/inertial_macros.xacro" />
  <xacro:include filename="$(find swerve_bot_description)/urdf/swerve_module.xacro" />
  <xacro:include filename="$(find swerve_bot_description)/urdf/swerve_bot.ros2_control.xacro" />

  <xacro:property name="chassis_length" value="0.4" />
  <xacro:property name="wheel_radius" value="0.05" />

  <link name="base_link">
    </link>
  <link name="base_footprint"/>
  <joint name="base_footprint_joint" type="fixed">
    <parent link="base_link"/>
    <child link="base_footprint"/>
    <origin xyz="0 0 0" rpy="0 0 0"/>
  </joint>

  <xacro:swerve_module prefix="front_right" parent="base_link">
    <origin xyz="${wheelbase/2} ${-track_width/2} 0" rpy="0 0 0"/>
  </xacro:swerve_module>
  <xacro:swerve_module prefix="front_left" parent="base_link">
    <origin xyz="${wheelbase/2} ${track_width/2} 0" rpy="0 0 0"/>
  </xacro:swerve_module>
  <xacro:swerve_module prefix="back_left" parent="base_link">
    <origin xyz="${-wheelbase/2} ${track_width/2} 0" rpy="0 0 0"/>
  </xacro:swerve_module>
  <xacro:swerve_module prefix="back_right" parent="base_link">
    <origin xyz="${-wheelbase/2} ${-track_width/2} 0" rpy="0 0 0"/>
  </xacro:swerve_module>

</robot>
```



### 3.4  `ros2_control` 및 Gazebo 플러그인 연동



로봇 모델이 시뮬레이터와 제어 프레임워크와 통신하기 위해서는 URDF에 관련 플러그인 정보를 추가해야 합니다. 이 정보는 별도의 `swerve_bot.ros2_control.xacro` 파일에 정의하여 모듈성을 높이는 것이 좋습니다. 이 방식은 시뮬레이션용 설정과 실제 하드웨어용 설정을 쉽게 전환할 수 있게 해주는 고급 설계 패턴입니다.16

**`urdf/swerve_bot.ros2_control.xacro`**

XML

```
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro">

  <xacro:macro name="swerve_bot_ros2_control" params="name">
    <ros2_control name="${name}" type="system">
      <hardware>
        <plugin>gazebo_ros2_control/GazeboSystem</plugin>
      </hardware>

      <joint name="front_right_steering_joint">
        <command_interface name="position">
          <param name="min">-3.1415</param>
          <param name="max">3.1415</param>
        </command_interface>
        <state_interface name="position"/>
        <state_interface name="velocity"/>
      </joint>
      <joint name="front_left_steering_joint">
        <command_interface name="position">
          <param name="min">-3.1415</param>
          <param name="max">3.1415</param>
        </command_interface>
        <state_interface name="position"/>
        <state_interface name="velocity"/>
      </joint>
      <joint name="front_right_drive_joint">
        <command_interface name="velocity">
          <param name="min">-10</param>
          <param name="max">10</param>
        </command_interface>
        <state_interface name="velocity"/>
        <state_interface name="position"/>
      </joint>
      <joint name="front_left_drive_joint">
        <command_interface name="velocity">
          <param name="min">-10</param>
          <param name="max">10</param>
        </command_interface>
        <state_interface name="velocity"/>
        <state_interface name="position"/>
      </joint>
      </ros2_control>

    <gazebo>
      <plugin name="gazebo_ros2_control" filename="libgazebo_ros2_control.so">
        <parameters>$(find swerve_bot_description)/config/swerve_bot_controllers.yaml</parameters>
      </plugin>
    </gazebo>
  </xacro:macro>

</robot>
```

이 파일은 두 가지 중요한 역할을 합니다.17

1. `<ros2_control>` 태그: `ros2_control` 프레임워크에게 하드웨어 인터페이스(여기서는 `GazeboSystem` 플러그인)와 제어할 조인트 목록, 그리고 각 조인트가 제공하는 명령/상태 인터페이스(`position`, `velocity`)를 알려줍니다.
2. `<gazebo>` 플러그인: Gazebo 시뮬레이터에게 `gazebo_ros2_control` 플러그인을 로드하라고 지시합니다. 이 플러그인은 URDF의 `<ros2_control>` 태그를 파싱하고, 컨트롤러 매니저를 Gazebo 내에서 실행하며, 컨트롤러 설정 파일(`swerve_bot_controllers.yaml`)을 로드하는 역할을 합니다.

마지막으로, 메인 URDF 파일(`swerve_bot.urdf.xacro`)에서 이 매크로를 호출합니다.

XML

```
<xacro:swerve_bot_ros2_control name="GazeboSwerveSystem"/>
```



## 4.  `ros2_control` 시스템 설정



`ros2_control`은 ROS 2의 표준 실시간 제어 프레임워크입니다. 하드웨어(또는 시뮬레이터)와 제어 알고리즘을 분리하여 코드의 모듈성과 재사용성을 높입니다. URDF에 정의된 하드웨어 인터페이스를 어떤 컨트롤러로 제어할지는 YAML 설정 파일을 통해 결정됩니다.12



### 4.1  `controllers.yaml` 설정 파일



`config/swerve_bot_controllers.yaml` 파일은 컨트롤러 매니저가 어떤 컨트롤러들을 로드하고 활성화할지, 그리고 각 컨트롤러의 파라미터는 무엇인지를 정의합니다.

**`config/swerve_bot_controllers.yaml`**

YAML

```
controller_manager:
  ros__parameters:
    update_rate: 100
    use_sim_time: true

    joint_state_broadcaster:
      type: joint_state_broadcaster/JointStateBroadcaster

    steering_controller:
      type: position_controllers/JointGroupPositionController

    drive_controller:
      type: velocity_controllers/JointGroupVelocityController

steering_controller:
  ros__parameters:
    joints:
      - front_right_steering_joint
      - front_left_steering_joint
      - back_left_steering_joint
      - back_right_steering_joint
    command_interfaces:
      - position
    state_interfaces:
      - position
      - velocity

drive_controller:
  ros__parameters:
    joints:
      - front_right_drive_joint
      - front_left_drive_joint
      - back_left_drive_joint
      - back_right_drive_joint
    command_interfaces:
      - velocity
    state_interfaces:
      - velocity
      - position
```

이 파일은 `controller_manager`와 3개의 컨트롤러(`joint_state_broadcaster`, `steering_controller`, `drive_controller`)를 설정합니다.



### 4.2  `joint_state_broadcaster` 설정



`joint_state_broadcaster`는 `ros2_control` 시스템의 필수적인 구성 요소입니다. 이 컨트롤러는 하드웨어 인터페이스(URDF의 `<ros2_control>` 태그에 정의된)로부터 모든 조인트의 현재 상태(위치, 속도 등)를 읽어와서, 이를 `/joint_states`라는 토픽으로 발행(broadcast)하는 역할을 합니다.17

`robot_state_publisher` 노드는 이 토픽을 구독하여 로봇의 TF(Transform) 트리를 계산하고, RViz는 이를 이용해 로봇의 현재 자세를 시각화합니다.



### 4.3  조향 조인트 위치 컨트롤러 설정



4개의 조향 조인트는 특정 각도로 제어되어야 하므로 위치 컨트롤러가 필요합니다. 여기서 `position_controllers/JointGroupPositionController`를 사용한 것은 중요한 설계 결정입니다.20 4개의 개별적인 위치 컨트롤러를 사용하는 대신, 

`JointGroup` 컨트롤러를 사용하면 단일 토픽(`/steering_controller/commands`)에 `std_msgs/Float64MultiArray` 타입의 메시지로 4개 조인트의 목표 위치를 한 번에 전달할 수 있습니다. 이는 제어 노드의 구현을 단순화하고 ROS 네트워크상의 부하를 줄여주는 효율적인 방법입니다.

`steering_controller`의 `joints` 파라미터에는 제어할 4개 조향 조인트의 이름을 정확히 나열해야 합니다. 이 이름들은 `swerve_bot.ros2_control.xacro` 파일에 정의된 조인트 이름과 반드시 일치해야 합니다. 이러한 URDF와 YAML 간의 이름 일치는 `ros2_control`이 올바르게 동작하기 위한 암묵적인 "계약"이며, 불일치 시 컨트롤러 로딩에 실패하므로 주의해야 합니다.



### 4.4  구동 바퀴 속도 컨트롤러 설정



마찬가지로 4개의 구동 바퀴는 특정 속도로 제어되어야 합니다. 이를 위해 `velocity_controllers/JointGroupVelocityController`를 사용합니다.20 이 컨트롤러 역시 

`/drive_controller/commands`라는 단일 토픽을 통해 4개 바퀴의 목표 속도를 배열 형태로 수신합니다.

**표 2: 컨트롤러 설정 요약**

| 컨트롤러 이름             | 컨트롤러 타입                                       | 관리 조인트                 | 명령 토픽                       |
| ------------------------- | --------------------------------------------------- | --------------------------- | ------------------------------- |
| `joint_state_broadcaster` | `joint_state_broadcaster/JointStateBroadcaster`     | 모든 조인트(상태 읽기 전용) | N/A (상태 발행)                 |
| `steering_controller`     | `position_controllers/JointGroupPositionController` | 4개의 조향 조인트           | `/steering_controller/commands` |
| `drive_controller`        | `velocity_controllers/JointGroupVelocityController` | 4개의 구동 조인트           | `/drive_controller/commands`    |

이러한 컨트롤러 설정은 시뮬레이션뿐만 아니라 실제 하드웨어에도 거의 동일하게 적용될 수 있으며, 이것이 `ros2_control`의 강력한 추상화 능력입니다.



## 5.  스위블 드라이브 운동학 노드 구현



이제 이론과 설정을 코드로 구현할 차례입니다. 우리는 외부(예: 조이스틱, 내비게이션 스택)로부터 로봇 전체의 목표 속도를 담은 `Twist` 메시지를 받아, 섹션 1에서 유도한 역운동학을 계산하고, 그 결과를 섹션 4에서 설정한 조인트 그룹 컨트롤러들에게 전달하는 C++ 노드를 작성할 것입니다.



### 5.1  노드 아키텍처



우리가 만들 `SwerveControllerNode`의 역할과 인터페이스는 다음과 같습니다.

- **구독(Subscription):** `/cmd_vel` 토픽을 구독하여 `geometry_msgs/msg/Twist` 메시지를 수신합니다.21 이 메시지의 

  `linear.x`, `linear.y`, `angular.z` 필드를 사용합니다.

- **발행(Publication):**

  - `/steering_controller/commands` 토픽으로 `std_msgs/msg/Float64MultiArray` 메시지를 발행하여 4개 조향 조인트의 목표 각도를 전달합니다.
  - `/drive_controller/commands` 토픽으로 `std_msgs/msg/Float64MultiArray` 메시지를 발행하여 4개 구동 바퀴의 목표 속도를 전달합니다.

- **파라미터(Parameters):** 운동학 계산에 필요한 로봇의 물리적 치수(`wheelbase`, `track_width`)를 ROS 파라미터 서버로부터 동적으로 읽어옵니다. 이렇게 하면 코드를 재컴파일하지 않고도 다른 크기의 로봇에 동일한 제어 노드를 적용할 수 있습니다.

**표 3: Swerve Controller Node API**

| 인터페이스 타입 | 토픽 이름                       | 메시지 타입                      | 설명                              |
| --------------- | ------------------------------- | -------------------------------- | --------------------------------- |
| Subscriber      | `/cmd_vel`                      | `geometry_msgs/msg/Twist`        | 로봇의 목표 선속도 및 각속도 명령 |
| Publisher       | `/steering_controller/commands` | `std_msgs/msg/Float64MultiArray` | 4개 조향 조인트의 목표 위치(각도) |
| Publisher       | `/drive_controller/commands`    | `std_msgs/msg/Float64MultiArray` | 4개 구동 바퀴의 목표 속도         |
| Parameter       | `wheelbase`                     | `double`                         | 로봇의 축간 거리                  |
| Parameter       | `track_width`                   | `double`                         | 로봇의 윤거                       |



### 5.2  C++ 전체 구현 (`swerve_controller_node.cpp`)



이제 `src/swerve_controller_node.cpp` 파일을 열고 다음 코드를 작성합니다. 이 코드는 위에서 설계한 아키텍처를 충실히 따르며, 특히 실용적인 성능을 위한 **조향 최적화 로직**을 포함합니다.

조향 최적화란, 목표 조향각으로 이동할 때 현재 각도에서 가장 가까운 경로를 선택하는 기법입니다. 예를 들어, 현재 각도가 10도이고 목표 각도가 170도일 때, 160도를 회전하는 대신 반대 방향으로 20도만 회전하고 바퀴의 구동 방향을 반대로 뒤집는 것이 훨씬 빠르고 효율적입니다. 이 로직은 로봇의 움직임을 부드럽게 만들고 불필요한 조향 동작을 줄여줍니다.23

C++

```
#include <rclcpp/rclcpp.hpp>
#include <geometry_msgs/msg/twist.hpp>
#include <std_msgs/msg/float64_multi_array.hpp>
#include <vector>
#include <cmath>
#include <memory>

class SwerveControllerNode : public rclcpp::Node
{
public:
    SwerveControllerNode() : Node("swerve_controller_node")
    {
        // 파라미터 선언 및 초기화
        this->declare_parameter<double>("wheelbase", 0.35);
        this->declare_parameter<double>("track_width", 0.25);
        this->get_parameter("wheelbase", wheelbase_);
        this->get_parameter("track_width", track_width_);

        RCLCPP_INFO(this->get_logger(), "Wheelbase: %f, Track Width: %f", wheelbase_, track_width_);

        // 퍼블리셔 및 서브스크라이버 생성
        steering_pub_ = this->create_publisher<std_msgs::msg::Float64MultiArray>("/steering_controller/commands", 10);
        drive_pub_ = this->create_publisher<std_msgs::msg::Float64MultiArray>("/drive_controller/commands", 10);
        cmd_vel_sub_ = this->create_subscription<geometry_msgs::msg::Twist>(
            "/cmd_vel", 10, std::bind(&SwerveControllerNode::cmdVelCallback, this, std::placeholders::_1));

        // 현재 조향각 초기화 (실제로는 joint_states를 구독해야 함)
        current_steering_angles_.resize(4, 0.0);
    }

private:
    void cmdVelCallback(const geometry_msgs::msg::Twist::SharedPtr msg)
    {
        double v_bx = msg->linear.x;
        double v_by = msg->linear.y;
        double w_bz = msg->angular.z;

        // 각 모듈의 위치 (FR, FL, BL, BR 순서)
        std::vector<std::pair<double, double>> module_positions = {
            {wheelbase_ / 2, -track_width_ / 2},
            {wheelbase_ / 2, track_width_ / 2},
            {-wheelbase_ / 2, track_width_ / 2},
            {-wheelbase_ / 2, -track_width_ / 2}
        };

        std_msgs::msg::Float64MultiArray steering_commands;
        std_msgs::msg::Float64MultiArray drive_commands;
        steering_commands.data.resize(4);
        drive_commands.data.resize(4);

        for (size_t i = 0; i < module_positions.size(); ++i)
        {
            double l_x = module_positions[i].first;
            double l_y = module_positions[i].second;

            // 1. 역운동학 계산
            double v_ix = v_bx - w_bz * l_y;
            double v_iy = v_by + w_bz * l_x;

            double target_speed = std::sqrt(v_ix * v_ix + v_iy * v_iy);
            double target_angle = std::atan2(v_iy, v_ix);
            
            // 2. 조향 최적화 (Shortest Path)
            double current_angle = current_steering_angles_[i];
            double angle_diff = target_angle - current_angle;

            // 각도 차이를 -pi ~ pi 범위로 정규화
            while (angle_diff > M_PI) angle_diff -= 2 * M_PI;
            while (angle_diff < -M_PI) angle_diff += 2 * M_PI;

            if (std::abs(angle_diff) > M_PI / 2.0)
            {
                // 90도 이상 차이나면 반대 방향으로 회전하고 속도를 뒤집음
                target_angle += (angle_diff > 0)? -M_PI : M_PI;
                target_speed = -target_speed;
            }
            
            steering_commands.data[i] = target_angle;
            drive_commands.data[i] = target_speed;

            // 다음 계산을 위해 현재 각도 업데이트 (간단한 버전)
            current_steering_angles_[i] = target_angle;
        }

        steering_pub_->publish(steering_commands);
        drive_pub_->publish(drive_commands);
    }

    rclcpp::Publisher<std_msgs::msg::Float64MultiArray>::SharedPtr steering_pub_;
    rclcpp::Publisher<std_msgs::msg::Float64MultiArray>::SharedPtr drive_pub_;
    rclcpp::Subscription<geometry_msgs::msg::Twist>::SharedPtr cmd_vel_sub_;
    
    double wheelbase_;
    double track_width_;
    std::vector<double> current_steering_angles_;
};

int main(int argc, char * argv)
{
    rclcpp::init(argc, argv);
    rclcpp::spin(std::make_shared<SwerveControllerNode>());
    rclcpp::shutdown();
    return 0;
}
```

**참고:** 위 코드의 `current_steering_angles_`는 이상적으로는 `/joint_states` 토픽을 구독하여 실제 로봇의 조향각을 피드백 받아야 합니다. 하지만 본 예제에서는 개념 설명을 위해 이전 명령값으로 간단히 대체하였습니다. 실제 적용 시에는 피드백 루프를 구현하는 것이 좋습니다.



### 5.3  커스텀 노드 컴파일



이 C++ 노드는 섹션 2.4에서 설정한 `CMakeLists.txt`에 의해 자동으로 컴파일됩니다. `add_executable`과 `ament_target_dependencies` 규칙이 `swerve_controller_node.cpp`를 컴파일하고 필요한 라이브러리와 링크하는 역할을 합니다. 워크스페이스 루트에서 `colcon build --packages-select swerve_bot_description` 명령을 실행하면 이 노드가 빌드되어 실행 가능한 상태가 됩니다.



## 6.  시스템 통합 및 시뮬레이션



이제 개별적으로 개발한 운동학, 로봇 모델, 제어 설정, 제어 노드를 하나로 통합하여 완전한 시뮬레이션 환경을 구동할 차례입니다. 이 모든 과정을 자동화하고 조율하는 것이 바로 ROS 2 런치 시스템의 역할입니다.



### 6.1  마스터 런치 파일 작성 (`launch_sim.launch.py`)



Python 기반의 런치 파일은 단순한 노드 실행을 넘어, 파라미터 설정, 조건부 실행, 다른 런치 파일 포함 등 복잡한 로직을 구현할 수 있게 해줍니다. 우리는 `launch/launch_sim.launch.py` 파일을 생성하여 Gazebo, RViz, 그리고 필요한 모든 ROS 2 노드를 한 번에 실행하는 마스터 런치 파일을 작성할 것입니다.7



### 6.2  노드 오케스트레이션: Gazebo, RViz, `robot_state_publisher`, 컨트롤러 스포너



마스터 런치 파일은 다음과 같은 구성 요소들을 순차적으로, 그리고 체계적으로 실행시키는 역할을 합니다.

**`launch/launch_sim.launch.py`**

Python

```
import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch.actions import IncludeLaunchDescription, DeclareLaunchArgument
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.substitutions import LaunchConfiguration, Command
from launch_ros.actions import Node

def generate_launch_description():
    pkg_share = get_package_share_directory('swerve_bot_description')
    
    # use_sim_time 파라미터는 시뮬레이션에서 절대적으로 중요합니다.
    # 모든 노드가 Gazebo의 시뮬레이션 시간(/clock)을 사용하도록 합니다.
    use_sim_time = LaunchConfiguration('use_sim_time', default='true')

    # 1. Gazebo 실행
    gazebo_params_file = os.path.join(pkg_share, 'config', 'gazebo_params.yaml')
    gazebo = IncludeLaunchDescription(
        PythonLaunchDescriptionSource([os.path.join(
            get_package_share_directory('gazebo_ros'), 'launch'), '/gazebo.launch.py']),
        launch_arguments={'extra_gazebo_args': '--ros-args --params-file ' + gazebo_params_file}.items(),
    )

    # 2. URDF 처리 및 robot_state_publisher 실행
    xacro_file = os.path.join(pkg_share, 'urdf', 'swerve_bot.urdf.xacro')
    robot_description_content = Command(['xacro ', xacro_file])
    
    robot_state_publisher_node = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        output='screen',
        parameters=[{'robot_description': robot_description_content,
                     'use_sim_time': use_sim_time}]
    )

    # 3. Gazebo에 로봇 스폰
    spawn_entity_node = Node(
        package='gazebo_ros',
        executable='spawn_entity.py',
        arguments=['-topic', 'robot_description', '-entity', 'swerve_bot'],
        output='screen'
    )

    # 4. 컨트롤러 로드 및 활성화 (Spawner)
    # ros2_control이 실행된 후 컨트롤러들을 로드해야 하므로, spawn_entity가 성공한 후에 실행되도록 설정할 수 있습니다.
    # (본 예제에서는 동시 실행)
    joint_state_broadcaster_spawner = Node(
        package="controller_manager",
        executable="spawner",
        arguments=["joint_state_broadcaster", "--controller-manager", "/controller_manager"],
    )

    steering_controller_spawner = Node(
        package="controller_manager",
        executable="spawner",
        arguments=["steering_controller", "--controller-manager", "/controller_manager"],
    )

    drive_controller_spawner = Node(
        package="controller_manager",
        executable="spawner",
        arguments=["drive_controller", "--controller-manager", "/controller_manager"],
    )

    # 5. 커스텀 스위블 컨트롤러 노드 실행
    swerve_controller_node = Node(
        package='swerve_bot_description',
        executable='swerve_controller_node',
        name='swerve_controller_node',
        output='screen',
        parameters=[{'wheelbase': 0.35, 'track_width': 0.25, 'use_sim_time': use_sim_time}]
    )
    
    # 6. RViz 실행
    rviz_config_file = os.path.join(pkg_share, 'rviz', 'swerve_bot.rviz')
    rviz_node = Node(
        package='rviz2',
        executable='rviz2',
        name='rviz2',
        arguments=['-d', rviz_config_file],
        output='screen'
    )

    return LaunchDescription()
```

이 런치 파일에서 가장 중요한 부분 중 하나는 `use_sim_time` 파라미터를 `true`로 설정하고 이를 모든 관련 노드에 전달하는 것입니다.16 시뮬레이션 환경에서는 Gazebo가 자체적인 시간(

` /clock` 토픽)을 발행합니다. 만약 ROS 노드들이 시스템의 실제 시간(wall clock)을 사용하면, 타임스탬프 불일치로 인해 TF 오류 등 심각한 동기화 문제가 발생합니다. 따라서 모든 노드가 시뮬레이션 시간을 사용하도록 통일하는 것은 안정적인 시뮬레이션을 위한 필수 조건입니다.

또한, `spawner` 노드의 역할에 대한 이해가 필요합니다. `spawner`는 단순히 컨트롤러를 실행하는 것이 아니라, `controller_manager`의 ROS 2 서비스와 통신하여 컨트롤러 플러그인을 동적으로 로드하고, YAML 파일의 파라미터로 설정한 후, 라이프사이클 상태를 'active'로 전환하는 역할을 합니다.12 이는 

`ros2_control`의 핵심적인 클라이언트-서버 아키텍처입니다.



### 6.3  시뮬레이션 실행 및 원격 조작 테스트



이제 모든 준비가 끝났습니다. 워크스페이스를 빌드하고 소싱한 후, 다음 명령어로 전체 시뮬레이션 환경을 실행할 수 있습니다.

Bash

```
# 1. 워크스페이스 루트로 이동
cd ~/swerve_ws

# 2. 패키지 빌드
colcon build --packages-select swerve_bot_description

# 3. 워크스페이스 환경 설정
source install/setup.bash

# 4. 마스터 런치 파일 실행
ros2 launch swerve_bot_description launch_sim.launch.py
```

이 명령을 실행하면 Gazebo와 RViz 창이 뜨고, 시뮬레이션 환경에 스위블 드라이브 로봇이 나타날 것입니다.

로봇을 움직여보기 위해, 새 터미널을 열고 `teleop_twist_keyboard` 패키지를 사용하거나 `ros2 topic pub` 명령으로 직접 `/cmd_vel` 토픽에 `Twist` 메시지를 발행합니다.

**키보드로 조작하기:**

Bash

```
# teleop_twist_keyboard 노드 실행
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

**직접 토픽 발행으로 테스트하기:**

Bash

```
# 전진 (linear.x = 0.5)
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.5, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}" -1

# 측면 이동 (linear.y = 0.5)
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.0, y: 0.5, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}" -1

# 제자리 회전 (angular.z = 1.0)
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.0}}" -1
```

RViz와 Gazebo에서 로봇이 발행된 명령에 따라 정확하게 전방향으로 움직이는 것을 확인할 수 있다면, 프로젝트는 성공적으로 완성된 것입니다.



## 7.  결론 및 향후 발전 방향





### 7.1. 프로젝트 요약 및 핵심 학습 내용



본 보고서는 ROS 2 Humble 환경에서 4륜 구동 및 4륜 조향이 가능한 스위블 드라이브 로봇을 개발하는 전 과정을 상세히 다루었습니다. 이 프로젝트를 통해 다음과 같은 핵심적인 기술과 개념을 학습할 수 있었습니다.

1. **운동학 모델링:** 로봇의 목표 움직임을 개별 액추에이터 명령으로 변환하는 스위블 드라이브의 역운동학을 수학적으로 유도하고, 조향 최적화와 같은 실용적인 기법을 C++ 코드로 구현했습니다.
2. **모듈식 로봇 설계:** Xacro의 속성과 매크로 기능을 활용하여 파라미터화되고 재사용 가능한 로봇 모델(URDF)을 작성했습니다. 이는 복잡한 로봇을 효율적으로 관리하는 표준적인 방법입니다.
3. **`ros2_control` 프레임워크 활용:** `ros2_control`을 사용하여 제어 로직과 하드웨어(시뮬레이터)를 분리하는 현대적인 제어 아키텍처를 구축했습니다. `JointGroup` 컨트롤러와 YAML 설정을 통해 다수의 조인트를 효율적으로 제어하는 방법을 익혔습니다.
4. **시뮬레이션 통합:** Gazebo 시뮬레이터를 ROS 2와 연동하고, `gazebo_ros2_control` 플러그인을 통해 로봇 모델과 제어 시스템을 통합했습니다. `use_sim_time`과 같은 핵심 파라미터의 중요성을 이해했습니다.
5. **시스템 오케스트레이션:** ROS 2 런치 시스템을 사용하여 복잡한 시뮬레이션 환경의 모든 구성 요소를 단일 명령으로 실행하고 관리하는 방법을 습득했습니다.

결론적으로, 이 프로젝트는 단순한 코드 예제를 넘어, 이론적 배경, 설계 철학, 그리고 실제 구현 기술이 유기적으로 결합된 종합적인 로봇 시스템 개발 경험을 제공합니다.



### 7.1  향후 발전 방향



본 프로젝트는 스위블 드라이브 로봇 개발의 견고한 출발점이며, 다음과 같은 방향으로 확장 및 발전시킬 수 있습니다.

- **주행 거리(Odometry) 구현:** 현재는 제어만 구현되었지만, 실제 자율 주행을 위해서는 로봇의 위치를 추정하는 것이 필수적입니다. 섹션 1.4에서 개념적으로 다룬 순운동학을 `ros2_control`의 하드웨어 인터페이스 내에 구현하거나 별도의 오도메트리 노드를 작성하여, `nav_msgs/msg/Odometry` 메시지를 발행하고 이를 TF로 변환하여 게시할 수 있습니다.
- **Nav2 스택 연동:** 정밀한 주행 거리 정보가 확보되면, ROS 2의 표준 내비게이션 프레임워크인 Nav2와 연동할 수 있습니다. 스위블 드라이브의 전방향 이동 능력을 최대한 활용하기 위해서는 SMAC Planner나 MPPI Controller와 같이 홀로노믹 움직임을 지원하는 플래너와 컨트롤러를 선택하고 설정해야 합니다.26
- **실제 하드웨어 적용:** 본 프로젝트의 아키텍처는 시뮬레이션과 실제 하드웨어 간의 전환을 용이하게 설계되었습니다. `swerve_bot.ros2_control.xacro` 파일에서 `GazeboSystem` 플러그인 부분을 실제 모터 드라이버(예: Arduino, Maxon, Roboteq)와 통신하는 커스텀 하드웨어 인터페이스 플러그인으로 교체하면, 동일한 상위 제어 노드와 설정을 사용하여 실제 로봇을 구동할 수 있습니다. 이는 `ros2_control`의 가장 큰 장점 중 하나입니다.

이러한 확장 과정은 로봇 공학의 더 깊은 영역으로 나아가는 도전적인 과제가 될 것이며, 본 프로젝트에서 다진 기반은 그 여정의 훌륭한 디딤돌이 될 것입니다.



#### **참고 자료**

1. Wheeled Mobile Robot Kinematics — ROS2_Control: Humble Jul ..., accessed July 14, 2025, https://control.ros.org/humble/doc/ros2_controllers/doc/mobile_robot_kinematics.html
2. Wheeled Mobile Robot Kinematics — ROS2_Control: Iron Jul 2025 documentation, accessed July 14, 2025, https://control.ros.org/iron/doc/ros2_controllers/doc/mobile_robot_kinematics.html
3. ackermann_steering_controller — ROS2_Control: Humble Jul 2025 documentation, accessed July 14, 2025, https://control.ros.org/humble/doc/ros2_controllers/ackermann_steering_controller/doc/userdoc.html
4. ackermann_steering_controller — ROS2_Control: Rolling Jul 2025 documentation, accessed July 14, 2025, https://control.ros.org/rolling/doc/ros2_controllers/ackermann_steering_controller/doc/userdoc.html
5. Creating a package — ROS 2 Documentation: Humble documentation, accessed July 14, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html
6. How to install ROS2 Humble and create Publisher and Subscriber in Python - YouTube, accessed July 14, 2025, https://www.youtube.com/watch?v=NVoZlrlyVBY
7. ros-controls/ros2_control_demos: This repository aims at providing examples to illustrate ros2_control and ros2_controllers - GitHub, accessed July 14, 2025, https://github.com/ros-controls/ros2_control_demos
8. Example 7: Full tutorial with a 6DOF robot - ROS2_Control, accessed July 14, 2025, https://control.ros.org/humble/doc/ros2_control_demos/example_7/doc/userdoc.html
9. Setting up a robot simulation (Basic) — ROS 2 Documentation: Humble documentation, accessed July 14, 2025, https://docs.ros.org/en/humble/Tutorials/Advanced/Simulators/Webots/Setting-Up-Simulation-Webots-Basic.html
10. Ali-Hossam/ROS2_4wheel_autonomous_robot: Four ... - GitHub, accessed July 14, 2025, https://github.com/Ali-Hossam/ROS2_4wheel_autonomous_robot
11. Tutorial : ROS 2 overview - Gazebo Classic, accessed July 14, 2025, https://classic.gazebosim.org/tutorials?tut=ros2_overview
12. Getting Started — ROS2_Control: Humble Jul 2025 documentation, accessed July 14, 2025, https://control.ros.org/humble/doc/getting_started/getting_started.html
13. How to Create a URDF for a Mobile Robot That Can Paint Like in the Video? - ROS General, accessed July 14, 2025, https://discourse.ros.org/t/how-to-create-a-urdf-for-a-mobile-robot-that-can-paint-like-in-the-video/42567
14. Using Xacro to clean up your code — ROS 2 Documentation ..., accessed July 14, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Using-Xacro-to-Clean-Up-a-URDF-File.html
15. 11. Simulate a Mobile Robot in ROS - Part 2 - Robodev Blog, accessed July 14, 2025, https://robodev.blog/simulate-a-mobile-robot-in-ros-part-2
16. ros2_control on the real robot - Articulated Robotics, accessed July 14, 2025, https://articulatedrobotics.xyz/tutorials/mobile-robot/applications/ros2_control-real/
17. ros2_control Concepts & Simulation - Articulated Robotics, accessed July 14, 2025, https://articulatedrobotics.xyz/tutorials/mobile-robot/applications/ros2_control-concepts/
18. gazebo_ros2_control — ROS2_Control: Humble Jul 2025 ..., accessed July 14, 2025, https://control.ros.org/humble/doc/gazebo_ros2_control/doc/index.html
19. ros2_control_demos/example_11/doc/userdoc.rst at master · ros ..., accessed July 14, 2025, https://github.com/ros-controls/ros2_control_demos/blob/master/example_11/doc/userdoc.rst
20. velocity_controllers — ROS2_Control: Humble Jul 2025 documentation, accessed July 14, 2025, https://control.ros.org/humble/doc/ros2_controllers/velocity_controllers/doc/userdoc.html
21. c++ - Subscribing and publishing geometry/Twist messages from Turtlesim - Stack Overflow, accessed July 14, 2025, https://stackoverflow.com/questions/43515772/subscribing-and-publishing-geometry-twist-messages-from-turtlesim
22. Subscribing and publishing geometry/Twist messages from Turtlesim, accessed July 14, 2025, https://robotics.stackexchange.com/questions/80316/subscribing-and-publishing-geometry-twist-messages-from-turtlesim
23. Swerve steering ROS controller - Mark Naeem, accessed July 14, 2025, https://marknaeem.github.io/pages/swervesteering.html
24. Demos — ROS2_Control: Humble Jul 2025 documentation, accessed July 14, 2025, https://control.ros.org/humble/doc/ros2_control_demos/doc/index.html
25. Launch Gazebo from ROS 2, accessed July 14, 2025, https://gazebosim.org/docs/latest/ros2_launch_gazebo/
26. Four wheel independent steering robot using Nav2 and ros2-control - ROS Discourse, accessed July 14, 2025, https://discourse.ros.org/t/four-wheel-independent-steering-robot-using-nav2-and-ros2-control/34587


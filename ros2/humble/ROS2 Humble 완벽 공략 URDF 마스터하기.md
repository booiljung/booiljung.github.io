# ROS2 Humble 완벽 공략 URDF 마스터하기

## 1.  URDF, 로봇을 디지털로 되살리는 설계도

### 1.1  URDF의 핵심 철학: 링크(Link)와 조인트(Joint)

URDF(Unified Robot Description Format)는 로봇의 물리적 구조를 XML(Extensible Markup Language) 기반 포맷으로 설명하기 위한 표준이다.1 이는 단순히 로봇의 3D 모델을 렌더링하는 것을 넘어, 로봇의 기구학적(kinematic) 및 동역학적(dynamic) 특성을 정의하여 시뮬레이션과 제어에 활용할 수 있도록 하는 디지털 설계도와 같다.4 이 설계도의 핵심을 이루는 두 가지 기본 구성 요소는 바로 **링크(Link)**와 **조인트(Joint)**다.

**링크(Link)**는 로봇의 개별적인 강체(rigid body) 부품을 의미한다.2 강체란 외부에서 힘이 가해져도 그 모양이나 크기가 변하지 않는 이상적인 물체를 말하며, 로봇의 몸통, 팔의 마디, 다리, 바퀴 등이 모두 링크에 해당한다.1 각 링크는 고유의 질량, 관성, 형태, 색상 등의 물리적 및 시각적 속성을 가질 수 있다.

**조인트(Joint)**는 이러한 링크들을 서로 연결하는 관절의 역할을 한다.1 조인트는 두 링크가 서로에 대해 어떻게 움직일 수 있는지를 정의하며, 이는 로봇의 움직임을 구현하는 핵심 요소다. 예를 들어, 모터에 의해 회전하는 로봇 팔의 관절이나, 직선 운동을 하는 액추에이터 등이 조인트로 표현된다.2

결론적으로, 로봇의 전체 모델은 하나의 루트 링크(root link)에서 시작하여 여러 개의 링크와 조인트가 번갈아 가며 사슬(chain) 또는 트리(tree) 구조로 연결된 형태로 구성된다. 이 구조를 통해 로봇의 각 부분이 어떻게 연결되어 있고, 각 관절이 움직일 때 전체 로봇의 자세가 어떻게 변하는지를 수학적으로 계산할 수 있게 된다. 이것이 바로 URDF가 로봇의 움직임을 기술하는 기본 원리다.

로봇 모델링을 처음 접하는 개발자들은 종종 URDF를 RViz와 같은 시각화 도구에서 로봇의 3D 모델을 보여주기 위한 파일 정도로만 생각하는 경향이 있다.6 물론 시각화는 URDF의 중요한 기능 중 하나지만, 이는 빙산의 일각에 불과하다. URDF는 시각적 속성(`<visual>`) 외에도 물리적 충돌 속성(`<collision>`)과 관성 속성(`<inertial>`)을 상세히 정의할 수 있다.1 이러한 정보는 Gazebo와 같은 고성능 물리 시뮬레이터에서 로봇이 중력의 영향을 받고, 다른 물체와 충돌하며, 관성에 따라 움직이는 등 실제 세계와 유사한 물리 법칙을 따르도록 하는 데 필수적이다.1 따라서 URDF를 작성하는 행위는 단순히 3D 모델을 만드는 것을 넘어, 실제 로봇의 물리적 특성을 디지털 공간에 정밀하게 복제하여 현실적인 테스트와 검증이 가능한 '가상 프로토타입(virtual prototype)'을 제작하는 과정이라고 이해해야 한다. 이 개념을 명확히 인지하고 있어야 `<inertial>`이나 `<collision>` 같은 태그의 중요성을 제대로 이해하고, 시뮬레이션의 정확도를 높이는 방향으로 URDF를 설계할 수 있다.

### 1.2  모든 것의 기준: URDF 좌표계(xyz, rpy)와 표준 단위

URDF는 로봇의 각 구성 요소의 위치와 방향을 명확하게 기술하기 위해 표준화된 좌표계와 단위를 사용한다. 이를 정확히 이해하지 못하면 로봇이 의도치 않은 위치에 조립되거나 비정상적인 방향으로 움직일 수 있다.

URDF에서 사용하는 좌표계는 3차원 공간상의 위치를 나타내는 **직교 좌표계(Cartesian coordinates)**와 방향을 나타내는 **오일러 각(Euler angles)**의 조합으로 이루어진다.2

- **위치(Translation):** `xyz` 속성을 사용하여 3차원 공간상의 위치를 표현한다. 각 값은 x, y, z축 방향으로의 이동 거리를 나타내며, 표준 단위는 **미터(m)**다.2 예를 들어, `xyz="0.1 0.0 -0.2"`는 x축으로 10cm, z축으로 -20cm 이동한 위치를 의미한다.

- **방향(Orientation):** `rpy` 속성을 사용하여 3차원 공간상의 방향(자세)을 표현한다. `rpy`는 각각 롤(roll), 피치(pitch), 요(yaw)를 의미하며, 이는 고정된 축(fixed-axis)을 기준으로 x축, y축, z축 순서로 회전한 각도를 나타낸다. 여기서 가장 중요한 점은 각도의 단위로 60분법(degree)이 아닌 **라디안(radian)**을 사용한다는 것이다.2

  `180도`는 `π` 라디안(약 3.14159)과 같다는 관계를 항상 기억해야 한다. 예를 들어, y축으로 90도 회전한 방향을 표현하려면 `rpy="0 1.5708 0"`과 같이 지정해야 한다.

이 외에도 URDF에서 사용되는 물리량들은 다음과 같은 표준 단위를 따른다 9:

- **질량(Mass):** 킬로그램(kg)
- **시간(Time):** 초(s)
- **선속도(Linear Velocity):** 미터/초(m/s)
- **각속도(Angular Velocity):** 라디안/초(rad/s)
- **힘(Force):** 뉴턴(N)
- **토크(Torque):** 뉴턴미터(Nm)

이러한 표준 단위를 일관되게 사용해야만 ROS 시스템 내의 다른 노드들(특히 물리 시뮬레이터)과 데이터를 원활하게 주고받을 수 있으며, 계산 오류를 방지할 수 있다.

## 2.  로봇의 뼈대와 살: `<link>` 태그 심층 분석

`<link>` 태그는 로봇의 물리적인 부품, 즉 강체(rigid body)를 정의하는 가장 기본적인 블록이다. 각 링크는 고유한 이름을 가지며, 그 안에는 시각적, 물리적, 동역학적 특성을 정의하는 세 가지 중요한 하위 태그, 즉 `<visual>`, `<collision>`, `<inertial>`이 포함될 수 있다.5

### 2.1  시각적 표현: `<visual>`

`<visual>` 태그는 RViz와 같은 3D 시각화 도구에서 로봇이 어떻게 보일지를 결정하는 역할을 한다.1 사용자는 이 태그를 통해 링크의 모양, 크기, 색상, 텍스처 등을 정의할 수 있다. 하나의 링크는 여러 개의 `<visual>` 태그를 가질 수 있는데, 이 경우 각각의 시각적 요소들이 합쳐져서 하나의 복합적인 형태로 보이게 된다. 예를 들어, 몸체와 렌즈로 구성된 카메라를 표현할 때, 몸체용 `<visual>`과 렌즈용 `<visual>`을 각각 정의할 수 있다.5

`<visual>` 태그의 주요 하위 태그는 다음과 같다.

- **`<origin xyz="..." rpy="...">`**: 이 태그는 링크의 기준 좌표계로부터 시각적 형상이 얼마나 떨어져 있고(offset) 어떻게 회전해 있는지를 정의한다.5 기본적으로 모든 형상은 자신의 기하학적 중심에 원점을 두고 생성된다. 예를 들어, `<box>`는 상자의 정중앙에, `<cylinder>`는 원기둥의 길이와 지름의 중심에 원점이 있다. `<origin>` 태그를 사용하면 이 원점을 이동시키거나 회전시켜서 링크의 기준 좌표계에 정확히 맞출 수 있다.
- **`<geometry>`**: 링크의 실제 모양을 정의하는 부분이다. 두 가지 방식으로 정의할 수 있다.
  - **기본 도형(Primitives):** 간단한 기하학적 형태로, `box`, `cylinder`, `sphere` 세 가지를 지원한다.11
    - `<box size="width height depth"/>`: 직육면체.
    - `<cylinder radius="..." length="..."/>`: 원기둥.
    - `<sphere radius="..."/>`: 구.
  - **메쉬(Mesh):** 복잡한 형상을 표현하기 위해 외부 3D 모델링 파일을 불러오는 방식이다. STL, DAE(Collada), OBJ 등 다양한 형식을 지원한다.6 파일 경로는 `package://` 스킴을 사용하는 것이 표준이다. 이는 ROS가 빌드 시간에 패키지의 실제 경로를 찾아주기 때문에 이식성이 매우 높아진다.13
    - 예시: `<mesh filename="package://my_robot_description/meshes/arm_part.stl"/>`
- **`<material name="...">`**: 형상의 표면 재질, 즉 색상이나 텍스처를 정의한다.
  - **`<color rgba="R G B A"/>`**: Red, Green, Blue, Alpha(투명도) 값을 각각 0.0에서 1.0 사이의 값으로 지정하여 색상을 정의한다.5 예를 들어, 불투명한 파란색은 `rgba="0 0 1 1"`로 표현한다.
  - **`<texture filename="..."/>`**: 이미지 파일을 형상의 표면에 입혀 텍스처를 표현한다.5

### 2.2  물리적 상호작용: `<collision>`

`<collision>` 태그는 Gazebo와 같은 물리 시뮬레이터에서 해당 링크가 다른 물체와 어떻게 충돌할지를 계산하는 데 사용되는 물리적 경계(collision geometry)를 정의한다.1 이 태그의 구조는 `<visual>` 태그와 거의 동일하며, `<origin>`과 `<geometry>` 하위 태그를 가진다.5

여기서 가장 중요한 개념은 **`<visual>`과 `<collision>`은 서로 다를 수 있다**는 점이다. 시각적으로는 매우 정교하고 복잡한 메쉬 파일을 사용하더라도, 충돌 계산에는 그보다 훨씬 단순한 기본 도형(box, cylinder 등)의 조합을 사용하는 것이 일반적이다. 이는 복잡한 메쉬 간의 충돌 검사가 엄청난 계산 부하를 유발하여 시뮬레이션 속도를 저하시키기 때문이다. 따라서 성능 최적화를 위해 충돌 모델은 로봇의 실제 외형을 충분히 근사하면서도 최대한 단순하게 만드는 것이 모범 사례로 꼽힌다.5 예를 들어, 수만 개의 폴리곤으로 이루어진 로봇 팔 링크의 시각 모델이 있더라도, 충돌 모델은 몇 개의 원기둥과 구의 조합으로 단순화할 수 있다.

### 2.3  동역학의 핵심: `<inertial>`

`<inertial>` 태그는 링크의 관성 속성, 즉 질량(mass)과 관성 텐서(inertia tensor)를 정의한다. 이 정보는 동역학 시뮬레이션에서 링크가 힘과 토크에 어떻게 반응할지를 계산하는 데 필수적이다. 만약 이 태그가 생략되면 해당 링크는 질량과 관성이 0인, 즉 물리적으로 존재하지 않는 가상의 링크로 간주된다.2

`<inertial>` 태그의 하위 태그는 다음과 같다.

- **`<origin xyz="..." rpy="...">`**: 이 태그는 링크의 기준 좌표계에 대한 **질량 중심(Center of Mass, CoM)**의 위치와 방향을 지정한다.5 질량 중심은 물체의 질량이 균등하게 분포되어 있다고 가정할 수 있는 가상의 점으로, 시각적 중심이나 기하학적 중심과 반드시 일치하지는 않는다. 예를 들어, 한쪽 끝에 무거운 모터가 달린 막대 링크의 질량 중심은 모터 쪽에 치우쳐 있을 것이다.
- **`<mass value="..."/>`**: 링크의 총 질량을 킬로그램(kg) 단위로 지정한다.5
- **`<inertia ixx="..." ixy="..." ixz="..." iyy="..." iyz="..." izz="..."/>`**: 질량 중심을 원점으로 하는 좌표계에서 계산된 3x3 회전 관성 행렬(rotational inertia matrix) 또는 관성 텐서를 지정한다.2
  - `ixx`, `iyy`, `izz`: 주 관성 모멘트(moments of inertia)로, 각각 x, y, z축에 대한 회전 저항을 나타낸다.
  - `ixy`, `ixz`, `iyz`: 관성곱(products of inertia)으로, 서로 다른 축 간의 회전 운동이 어떻게 연관되는지를 나타낸다.
  - 관성 텐서는 링크의 질량 분포와 형상에 따라 결정되며, CAD 소프트웨어에서 자동으로 계산해주는 경우가 많다. 여기서 중요한 점은, 계산 오류를 줄이고 시뮬레이션의 안정성을 높이기 위해 관성곱(`ixy`, `ixz`, `iyz`)이 0이 되도록 관성 주축(principal axes of inertia)에 맞춰 좌표계를 정렬하는 것이 매우 좋은 설계 방식이라는 것이다.5

URDF를 작성할 때 `<origin>` 태그의 다중성과 그로 인한 좌표계의 계층 구조를 이해하는 것은 매우 중요하다. 많은 개발자들이 `<link>`와 `<joint>` 태그에 각각 `<origin>`이 있고, 심지어 `<visual>`, `<collision>`, `<inertial>` 내부에도 `<origin>`이 존재한다는 사실에 혼란을 겪는다.5 이들의 관계를 명확히 파악하는 것이 디버깅의 핵심이다. 먼저, `<joint>` 태그 내부의 `<origin>`은 **부모 링크의 좌표계**를 기준으로 **자식 링크의 좌표계**가 어디에 위치하는지를 정의한다. 이것이 바로 로봇의 전체적인 뼈대를 이루는 TF(Transform) 트리의 기본 연결 관계를 설정하는 것이다.10 반면, 

`<link>` 태그 내부의, 즉 `<visual>`, `<collision>`, `<inertial>` 태그 안에 있는 `<origin>`은, 방금 `<joint>`에 의해 정의된 **해당 링크 자신의 좌표계**를 기준으로 각 요소(시각적 형상, 충돌 형상, 질량 중심)가 얼마나 떨어져 있는지를 정의하는 로컬 오프셋이다.12

이 계층 구조를 이해하지 못하면 고질적인 문제에 부딪히게 된다. 예를 들어, 로봇의 바퀴를 10cm 옆으로 옮기고 싶을 때, `<visual>` 내부의 `<origin>`만 수정했다고 가정해보자. 이 경우 RViz에서는 바퀴가 움직인 것처럼 보일 것이다. 하지만 실제 바퀴 링크의 TF 프레임과 충돌 모델, 관성 중심은 원래 위치에 그대로 남아있게 된다. 그 결과, Gazebo 시뮬레이션에서는 로봇이 공중에서 움직이는 것처럼 보이거나, 충돌이 엉뚱한 곳에서 발생하는 등 비정상적인 동작을 보이게 된다. 따라서 URDF 디버깅의 핵심은 단순히 XML 문법을 확인하는 것을 넘어, `tf2_tools`나 RViz의 TF 디스플레이 기능을 사용하여 이 보이지 않는 좌표계들의 계층 구조(`부모_링크_프레임` -> `자식_링크_프레임` -> `visual/collision_프레임`)가 자신의 의도대로 정확하게 구성되었는지 시각적으로 확인하는 것이다. 이 점을 명심해야 "RViz에서는 멀쩡한데 시뮬레이션이 이상해요"와 같은 난해한 문제의 근본 원인을 파악하고 해결할 수 있다.

## 3.  로봇에 움직임을 불어넣기: `<joint>` 태그 완전 정복

`<joint>` 태그는 정적인 링크들을 연결하여 움직이는 로봇을 만드는 핵심 요소다. 이 태그는 두 링크 간의 상대적인 운동을 정의하며, 로봇의 자유도(Degrees of Freedom, DoF)를 결정한다.

### 3.1  조인트 타입 완벽 가이드: `revolute`, `continuous`, `prismatic`, `fixed` 등

`<joint>` 태그의 `type` 속성은 관절이 어떤 방식으로 움직일지를 결정한다.16 각 타입의 특징과 사용 사례를 정확히 이해하고 선택하는 것이 중요하다.

| 타입 (Type)  | 운동 방식 (Motion)             | 한계 (`<limit>`)    | 축 (`<axis>`)          | 주요 사용 예시                                               |
| ------------ | ------------------------------ | ------------------- | ---------------------- | ------------------------------------------------------------ |
| `revolute`   | 축 중심 회전 (한계 있음)       | 필수 (lower, upper) | 필수                   | 로봇 팔 관절, 특정 각도로 여닫는 그리퍼 2                    |
| `continuous` | 축 중심 무한 회전              | 불필요              | 필수                   | 구동 바퀴, 계속 회전하는 라이다(LIDAR) 센서 2                |
| `prismatic`  | 축 방향 직선 운동 (한계 있음)  | 필수 (lower, upper) | 필수                   | 선형 액추에이터, 상하로 움직이는 리프트 메커니즘 2           |
| `fixed`      | 움직임 없음 (두 링크를 고정)   | 불필요              | 불필요                 | 카메라나 센서를 로봇 몸체에 단단히 부착할 때 6               |
| `floating`   | 6-DOF (3D 공간 자유 이동/회전) | 불필요              | 불필요                 | 월드(world) 프레임과 로봇의 베이스 링크를 연결하여 공중에 떠 있는 것처럼 표현할 때 2 |
| `planar`     | 평면상 이동 및 회전            | 불필요              | 필수 (평면의 법선벡터) | 평면 위를 움직이는 로봇 (예: 호버크래프트) 2                 |

### 3.2  링크 연결의 정석: `<parent>`, `<child>`, `<origin>` 태그

모든 조인트는 반드시 두 개의 링크를 연결해야 하며, 이 연결 관계는 명확한 부모-자식 관계로 정의된다. 이는 로봇의 기구학적 구조가 트리(tree) 형태를 이루도록 보장한다.

- **`<parent link="..."/>`**: 부모가 되는 링크의 이름을 지정한다. 로봇 모델의 모든 링크(루트 링크 제외)는 반드시 하나의 부모 조인트를 가져야 한다.12
- **`<child link="..."/>`**: 자식이 되는 링크의 이름을 지정한다.12 하나의 부모 링크는 여러 개의 자식 조인트를 가질 수 있지만, 하나의 자식 링크는 오직 하나의 부모 조인트만 가질 수 있다.
- **`<origin xyz="..." rpy="...">`**: 이 태그는 조인트의 위치와 방향을 정의하며, 그 기준은 **부모 링크의 좌표계**다. 더 정확히 말하면, 부모 링크의 좌표계를 기준으로 자식 링크의 좌표계가 어디에 위치하고 어떤 방향을 갖는지를 정의하는 변환(transform) 정보이다.10 조인트 자체의 회전축이나 이동축은 이 자식 링크의 좌표계 원점에 위치하게 된다.

### 3.3  운동의 방향과 한계 설정: `<axis>`와 `<limit>`

움직이는 조인트(`revolute`, `continuous`, `prismatic`)는 운동의 방향과 범위를 정의하는 추가적인 정보가 필요하다.

- **`<axis xyz="...">`**: 조인트의 운동 축을 3차원 벡터로 지정한다. 이 벡터는 조인트의 좌표계, 즉 **자식 링크의 좌표계**를 기준으로 정의된다는 점에 유의해야 한다.

  - `revolute`와 `continuous` 조인트에서는 회전축이 된다.

  - `prismatic` 조인트에서는 직선 운동의 방향이 된다.

  - 이 벡터는 반드시 크기가 1인 단위 벡터(normalized vector)여야 한다.16 만약 지정하지 않으면 기본값으로 

    `xyz="1 0 0"` (x축)이 사용된다.

- **`<limit lower="..." upper="..." effort="..." velocity="..."/>`**: `revolute`와 `prismatic` 조인트의 운동 범위를 제한하는 데 사용된다. `continuous` 조인트는 무한 회전하므로 이 태그를 사용하지 않는다.16

  - `lower`와 `upper`: 조인트가 움직일 수 있는 범위의 하한값과 상한값을 지정한다. `revolute` 조인트는 라디안(radian) 단위, `prismatic` 조인트는 미터(m) 단위다.16
  - `effort`: 조인트가 발휘할 수 있는 최대 힘(prismatic) 또는 토크(revolute)를 지정한다. 단위는 각각 뉴턴(N)과 뉴턴미터(Nm)다. 시뮬레이션과 제어에 중요한 파라미터다.16
  - `velocity`: 조인트가 움직일 수 있는 최대 속도를 지정한다. 단위는 각각 미터/초(m/s)와 라디안/초(rad/s)다.16

이러한 태그들을 조합하여 로봇의 모든 관절을 정밀하게 정의함으로써, 비로소 디지털 로봇 모델에 생명력을 불어넣을 수 있게 된다.

## 4.  전문가의 코드 작성법: Xacro로 URDF 재사용성 극대화하기

순수한 XML 형식의 URDF 파일은 로봇이 복잡해질수록 길이가 기하급수적으로 늘어나고, 반복적인 코드가 많아져 가독성과 유지보수성이 급격히 떨어진다. Xacro(XML Macros)는 이러한 문제를 해결하기 위해 등장한 강력한 도구다. Xacro는 URDF 파일 내에서 변수, 수학식, 매크로, 조건문 등을 사용할 수 있게 해주는 전처리기(preprocessor)로, 코드를 더 간결하고, 모듈화하고, 재사용 가능하게 만들어준다.18

### 4.1  변수, 상수, 수학식 활용법

Xacro의 가장 기본적이면서도 강력한 기능은 반복되는 값을 변수로 만들어 한 곳에서 관리하는 것이다.

- **속성(Properties) / 변수 정의:** `<xacro:property>` 태그를 사용하여 변수(또는 상수)를 정의한다.

  ```XML
  <xacro:property name="wheel_radius" value="0.05" />
  <xacro:property name="wheel_length" value="0.02" />
  <xacro:property name="PI" value="3.1415926535" />
  ```

  이렇게 정의된 변수는 파일 내 어디서든 `${변수명}` 형식으로 참조할 수 있다.11

  ```XML
  <cylinder radius="${wheel_radius}" length="${wheel_length}" />
  ```

  이를 통해 바퀴의 반지름과 같은 중요한 설계 치수를 파일 상단에 모아두고, 변경이 필요할 때 한 곳만 수정하면 관련된 모든 부분이 자동으로 업데이트되도록 할 수 있다.

- **수학식(Math Expressions):** Xacro는 `${}` 구문 안에서 간단한 수학 연산을 지원한다. 사칙연산(+, -, *, /)은 물론, `pi`, `sin`, `cos`, `sqrt` 등 파이썬의 `math` 모듈에서 제공하는 대부분의 함수와 상수를 사용할 수 있다.11

  ```XML
  <origin xyz="0 0 ${-wheel_length / 2}" rpy="${PI/2} 0 0" />
  ```

  이 기능은 링크의 크기나 다른 변수에 따라 위치나 방향을 동적으로 계산해야 할 때 매우 유용하다.

### 4.2  궁극의 모듈화: 매크로(`<xacro:macro>`) 작성 및 활용

매크로는 Xacro의 꽃이라 할 수 있는 기능으로, 코드의 재사용성을 극대화한다. C언어의 함수처럼, 파라미터를 받는 재사용 가능한 XML 코드 블록을 정의할 수 있다.18

- **매크로 정의:** `<xacro:macro>` 태그를 사용하며, `name` 속성으로 매크로 이름을, `params` 속성으로 전달받을 파라미터 목록을 지정한다.

  ```XML
  <xacro:macro name="wheel" params="prefix parent reflect">
    <link name="${prefix}_wheel_link">
      </link>
    <joint name="${prefix}_wheel_joint" type="continuous">
      <parent link="${parent}"/>
      <child link="${prefix}_wheel_link"/>
      <origin xyz="0 ${reflect * 0.1} 0" rpy="0 0 0"/>
      <axis xyz="0 1 0"/>
    </joint>
  </xacro:macro>
  ```

- **매크로 호출:** 정의된 매크로는 태그 형태로 호출하며, 파라미터 값을 속성으로 전달한다.

  ```XML
  <xacro:wheel prefix="left" parent="base_link" reflect="1" />
  <xacro:wheel prefix="right" parent="base_link" reflect="-1" />
  ```

  위 예제는 좌우 바퀴처럼 구조는 동일하지만 이름(`prefix`)이나 위치(`reflect` 파라미터를 이용한 y축 대칭)가 다른 부분을 단 몇 줄의 코드로 생성하는 방법을 보여준다. 이는 수십, 수백 줄의 반복적인 코드를 획기적으로 줄여준다.11

- **고급 기법:** 매크로의 일부를 외부에서 XML 블록 형태로 주입받는 것도 가능하다. 파라미터 이름 앞에 `*`를 붙여 블록 파라미터로 선언하고, 매크로 내부에서 `<xacro:insert_block name="블록_파라미터명" />`으로 삽입할 수 있다.19 이는 매크로의 유연성을 더욱 높여준다.

### 4.3  조건부 블럭(`<xacro:if>`, `<xacro:unless>`)을 이용한 동적 모델링

Xacro는 조건에 따라 특정 XML 블록을 포함하거나 제외하는 기능도 제공한다. 이는 런치 파일에서 전달된 인자값에 따라 URDF의 구성을 동적으로 변경하고자 할 때 매우 유용하다.15

- **`<xacro:if value="${condition}">`**: `condition`이 참(true 또는 1)일 경우, 내부의 XML 블록을 URDF에 포함시킨다.
- **`<xacro:unless value="${condition}">`**: `condition`이 거짓(false 또는 0)일 경우, 내부의 XML 블록을 포함시킨다.

활용 사례:

런치 파일에서 use_gazebo_plugin이라는 인자를 받아, 이 값이 true일 때만 Gazebo 시뮬레이션에 필요한 센서 플러그인 관련 코드를 URDF에 추가하도록 만들 수 있다.

```XML
<xacro:arg name="use_gazebo_plugin" default="false"/>

<xacro:if value="$(arg use_gazebo_plugin)">
  <gazebo reference="camera_link">
    <sensor type="camera" name="camera_sensor">
      </sensor>
  </gazebo>
</xacro:if>
```

이러한 방식은 실제 로봇을 구동할 때(플러그인이 필요 없을 때)와 시뮬레이션을 할 때(플러그인이 필요할 때) 동일한 Xacro 파일을 사용하면서도, 불필요한 의존성을 제거하고 상황에 맞는 최적의 URDF를 생성할 수 있게 해준다.8

Xacro의 진정한 가치는 단순한 코드 중복 제거를 넘어선다. 속성, 수학식, 매크로, 조건문을 전략적으로 조합하면, 로봇의 모든 핵심 설계 파라미터(치수, 질량, 부품의 유무 등)를 `.xacro` 파일 상단의 `<xacro:property>` 블록으로 모두 끌어올릴 수 있다. 이렇게 되면 `.xacro` 파일 자체가 로봇의 모든 설계 변수를 정의하는 일종의 '마스터 설정 파일(Design Configuration File)' 역할을 하게 된다. 개발자는 더 이상 복잡한 XML 구조를 헤맬 필요 없이, 파일 상단의 설정 값 몇 개만 변경하여 로봇의 다양한 변형(예: 팔 길이 변경, 그리퍼 부착/제거, 센서 추가)을 손쉽게 생성하고 테스트할 수 있다. 이 관점을 적용하면, 단 하나의 잘 설계된 `.xacro` 파일로부터 실제 로봇 구동용 URDF, Gazebo 시뮬레이션용 URDF, MoveIt! 경로 계획용 URDF 등 목적에 맞는 다양한 버전의 로봇 설명을 효율적으로 생성하는 것이 가능해진다. 이는 단순한 코드 정리를 넘어, 로봇 개발 파이프라인 전체의 생산성을 극대화하는 핵심 전략이 된다.8

## 5.  실전! ROS2 워크플로우: 런치 파일부터 RViz2 시각화까지

URDF 파일을 작성했다면, 이제 이를 ROS2 시스템에 로드하고 3D로 시각화하여 검증하는 과정이 필요하다. 이 워크플로우는 몇 가지 핵심 노드와 런치 파일의 유기적인 연동으로 이루어진다.

### 5.1  `robot_state_publisher`와 `joint_state_publisher`의 역할과 연동

URDF 모델을 RViz2에 띄우기 위해서는 최소한 세 개의 노드가 필요하다: `robot_state_publisher`, `joint_state_publisher`, 그리고 `rviz2`. 이들의 역할과 데이터 흐름은 다음과 같다.

- **`robot_state_publisher` (RSP)**: 이 노드는 URDF 시각화의 심장과도 같은 역할을 한다. RSP는 두 가지 중요한 정보를 입력받는다. 첫째는 `robot_description`이라는 파라미터로 전달되는 URDF 파일의 내용이고, 둘째는 `/joint_states`라는 토픽으로 전달되는 로봇의 현재 관절 상태(각도, 위치 등)다. RSP는 이 두 정보를 종합하여, 로봇의 모든 링크들 간의 상대적인 위치와 방향 관계, 즉 **TF(Transform) 트리**를 계산하고, 이를 `/tf` (움직이는 조인트)와 `/tf_static` (고정된 조인트) 토픽으로 발행한다.13
- **`joint_state_publisher` (JSP)**: 이 노드는 로봇의 각 관절이 현재 어떤 상태인지를 `sensor_msgs/msg/JointState` 타입의 메시지로 만들어 `/joint_states` 토픽에 발행하는 역할을 한다.13 실제 로봇이라면 하드웨어 드라이버가 센서 값을 읽어 이 정보를 발행하겠지만, 단순히 URDF 모델을 시각화하고 테스트하는 단계에서는 이 노드가 가상의 관절 상태 값을 제공해준다.
- **`joint_state_publisher_gui`**: `joint_state_publisher`의 GUI 버전으로, URDF에 정의된 모든 비고정(non-fixed) 조인트에 대한 슬라이더를 자동으로 생성해준다. 사용자는 이 슬라이더를 마우스로 조작하여 각 조인트의 값을 실시간으로 변경하고, 그에 따라 RViz2에 표시된 로봇 모델이 어떻게 움직이는지 직관적으로 확인할 수 있다.6

이 세 노드의 데이터 흐름을 요약하면 다음과 같다:

1. `joint_state_publisher_gui`가 사용자의 슬라이더 조작에 따라 조인트 상태 값을 `/joint_states` 토픽으로 발행한다.
2. `robot_state_publisher`는 `/joint_states` 토픽을 구독하여 현재 조인트 값을 얻고, `robot_description` 파라미터의 URDF 정보와 결합하여 TF 트리를 계산한다.
3. 계산된 TF 정보는 `/tf`와 `/tf_static` 토픽으로 발행된다.
4. `rviz2`는 `/tf` 토픽을 구독하여 TF 트리를 수신하고, 이를 바탕으로 `robot_description`의 시각 정보를 3D 공간에 렌더링한다.

### 5.2  파이썬 런치 파일(`launch.py`) 작성의 모든 것

ROS2에서는 여러 노드를 동시에 실행하고, 각 노드의 파라미터를 설정하며, 실행 순서를 제어하기 위해 런치 파일 시스템을 사용한다.26 ROS2 Humble에서는 Python 스크립트를 이용한 런치 파일 작성이 표준적인 방법이다.

URDF를 시각화하기 위한 런치 파일 작성 과정은 다음과 같다.

1. **패키지 및 디렉토리 생성:** URDF 관련 파일을 저장할 ROS2 패키지를 생성하고, 그 안에 `launch`, `urdf`, `meshes`, `rviz` 등의 디렉토리를 만든다.23

   ```Bash
   cd ~/ros2_ws/src
   ros2 pkg create my_robot_description --build-type ament_python
   cd my_robot_description
   mkdir launch urdf meshes rviz
   ```

2. **`setup.py` 설정:** 생성한 패키지의 `setup.py` 파일을 열어, `launch`, `urdf`, `meshes`, `rviz` 디렉토리 안의 파일들이 빌드 및 설치 과정에 포함되도록 `data_files` 항목을 수정해야 한다. 이는 `ros2 launch` 명령어가 해당 파일들을 찾을 수 있도록 하기 위함이다.13

   ```Python
   # setup.py
   import os
   from glob import glob
   from setuptools import setup
   
   package_name = 'my_robot_description'
   
   setup(
       #... 기타 설정...
       data_files=[
           ('share/ament_index/resource_index/packages',
               ['resource/' + package_name]),
           ('share/' + package_name, ['package.xml']),
           (os.path.join('share', package_name, 'launch'), glob('launch/*.launch.py')),
           (os.path.join('share', package_name, 'urdf'), glob('urdf/*')),
           (os.path.join('share', package_name, 'meshes'), glob('meshes/*')),
           (os.path.join('share', package_name, 'rviz'), glob('rviz/*')),
       ],
   )
   ```

3. **런치 파일(`display.launch.py`) 작성:** `launch` 디렉토리 안에 런치 파일을 생성하고, `generate_launch_description()` 함수 내에 노드 실행 로직을 작성한다.23

   다음은 `robot_state_publisher`, `joint_state_publisher_gui`, `rviz2`를 실행하는 완전한 런치 파일 예시다.

   ```Python
   import os
   from ament_index_python.packages import get_package_share_directory
   from launch import LaunchDescription
   from launch.substitutions import Command
   from launch_ros.actions import Node
   
   def generate_launch_description():
   
       # 패키지 경로 찾기
       pkg_path = get_package_share_directory('my_robot_description')
   
       # Xacro 파일 경로 설정
       xacro_file = os.path.join(pkg_path, 'urdf', 'my_robot.urdf.xacro')
   
       # Xacro 파일을 로드하여 robot_description 파라미터 생성
       robot_description_config = Command(['xacro ', xacro_file])
   
       # robot_state_publisher 노드
       robot_state_publisher_node = Node(
           package='robot_state_publisher',
           executable='robot_state_publisher',
           name='robot_state_publisher',
           output='screen',
           parameters=[{'robot_description': robot_description_config}]
       )
   
       # joint_state_publisher_gui 노드
       joint_state_publisher_gui_node = Node(
           package='joint_state_publisher_gui',
           executable='joint_state_publisher_gui',
           name='joint_state_publisher_gui',
           output='screen'
       )
   
       # RViz2 노드
       rviz_config_file = os.path.join(pkg_path, 'rviz', 'display.rviz')
       rviz_node = Node(
           package='rviz2',
           executable='rviz2',
           name='rviz2',
           output='screen',
           arguments=['-d', rviz_config_file]
       )
   
       # 런치 설명 반환
       return LaunchDescription([
           robot_state_publisher_node,
           joint_state_publisher_gui_node,
           rviz_node
       ])
   ```

   이 코드에서 가장 중요한 부분은 `robot_state_publisher`에 `robot_description` 파라미터를 전달하는 부분이다. `launch.substitutions.Command`를 사용하여 `xacro` 명령어를 런치 타임에 실행하고, 그 출력(순수 URDF XML 문자열)을 파라미터 값으로 직접 전달한다.19 이 방식을 사용하면 

   `.xacro` 파일을 수정할 때마다 수동으로 `.urdf` 파일을 생성할 필요가 없어 매우 편리하다.

   더 나아가, `urdf_launch` 패키지를 사용하면 위와 같은 런치 파일 작성을 더욱 간소화할 수 있다. 이 패키지는 URDF 시각화에 필요한 일반적인 런치 로직을 재사용 가능한 형태로 제공한다.29

### 5.3  RViz2에서 로봇 모델을 띄우고 상호작용하기

런치 파일을 실행하면 RViz2 창이 열린다. 로봇 모델이 정상적으로 표시되게 하려면 몇 가지 설정을 확인해야 한다.31

1. **RViz2 실행 및 설정:**

   ```Bash
   cd ~/ros2_ws
   colcon build --packages-select my_robot_description
   source install/setup.bash
   ros2 launch my_robot_description display.launch.py
   ```

2. **RViz2 Displays 패널 설정:**

   - **Global Options -> Fixed Frame:** RViz2 3D 뷰의 기준이 되는 좌표계를 설정하는 항목이다. 일반적으로 로봇의 루트 링크(예: `base_link`)나, 로봇이 존재하는 가상의 월드 프레임(예: `odom` 또는 `world`)으로 설정한다. 만약 여기에 설정된 프레임이 현재 TF 트리에 존재하지 않으면, "Fixed Frame [frame_name] does not exist"라는 오류가 발생하며 아무것도 표시되지 않는다. 따라서 가장 먼저 확인해야 할 부분이다.13
   - **RobotModel Display:** 왼쪽 하단의 'Add' 버튼을 눌러 'RobotModel' 디스플레이를 추가한다. 추가된 디스플레이의 속성에서 `Description Topic`이 `robot_description`으로 올바르게 설정되어 있는지 확인한다. 이 토픽을 통해 `robot_state_publisher`가 발행하는 URDF 정보를 수신한다.32
   - **TF Display:** 디버깅을 위해 'TF' 디스플레이를 추가하는 것이 매우 유용하다. 이를 통해 로봇의 모든 링크에 해당하는 좌표계(프레임)들이 어떻게 연결되어 있는지, 그리고 실시간으로 어떻게 움직이는지 시각적으로 확인할 수 있다.6

모든 설정이 완료되면, `joint_state_publisher_gui` 창의 슬라이더를 움직여보자. RViz2 화면의 로봇 모델이 슬라이더의 움직임에 따라 실시간으로 부드럽게 움직인다면, URDF 작성부터 런치 파일 실행까지의 모든 워크플로우가 성공적으로 완료된 것이다.6

## 6.  문제 해결 및 모범 사례: 버그는 잡고, 퀄리티는 높이고

URDF를 다루다 보면 다양한 문제에 직면하게 된다. 여기서는 흔히 발생하는 오류들의 원인과 해결책, 그리고 더 나은 URDF 설계를 위한 모범 사례를 다룬다.

### 6.1  URDF 문법 검사 유틸리티: `check_urdf`

`check_urdf`는 URDF 파일의 XML 문법과 링크-조인트 트리 구조가 유효한지 검사하는 간단하지만 유용한 명령줄 도구다.36 모델이 RViz에 나타나지 않거나 파싱 오류가 발생할 때 가장 먼저 시도해볼 수 있는 방법이다.

- **사용법:**

  ```Bash
  check_urdf /path/to/your/robot.urdf
  ```

  문제가 없다면 성공적으로 파싱되었다는 메시지와 함께 로봇의 트리 구조를 출력한다. 오류가 있다면 어느 부분에서 문제가 발생했는지 알려준다.

- **Xacro 파일 검사:** `check_urdf`는 순수 URDF 파일만 검사할 수 있으므로, Xacro 파일은 먼저 URDF로 변환한 후 검사해야 한다. 다음 명령어를 사용하면 변환과 검사를 한 번에 수행할 수 있다.20

  ```Bash
  xacro your_robot.urdf.xacro > temp.urdf && check_urdf temp.urdf && rm temp.urdf
  ```

- **한계:** `check_urdf`는 어디까지나 문법적인 오류(예: 닫히지 않은 태그, 존재하지 않는 부모 링크 참조)만 검출할 뿐, 논리적인 오류(예: 잘못 입력된 질량 값, 비현실적인 링크 크기, 잘못된 조인트 타입 선택)는 잡아내지 못한다는 점을 명심해야 한다.36

### 6.2  흔한 RViz2 오류 완벽 해부 ("Fixed Frame", "No transform", 메쉬 누락 등)

URDF를 처음 다룰 때 겪는 대부분의 문제는 RViz2에서 모델이 제대로 보이지 않는 현상이다. 다음은 대표적인 오류 메시지와 그 해결책이다.

- **문제 1: "Fixed Frame [map] does not exist" 또는 "No tf data"**
  - **원인:** RViz2의 'Global Options'에 설정된 `Fixed Frame`이 현재 ROS 시스템의 TF 트리에 존재하지 않는 프레임이기 때문이다. RViz는 이 프레임을 기준으로 모든 것을 그리는데, 기준점 자체가 없으니 아무것도 표시할 수 없는 것이다.33
  - **해결:** `Fixed Frame`을 현재 TF 트리에 존재하는 프레임, 보통 로봇의 루트 링크(예: `base_link`)나 월드 프레임(`world`, `odom`)으로 변경한다. `ros2 run tf2_ros tf2_echo <source_frame> <target_frame>`이나 `ros2 run tf2_tools view_frames` 명령어로 현재 어떤 프레임들이 존재하는지 확인할 수 있다.
- **문제 2: "No transform from [child_link] to [base_link]"**
  - **원인:** 이 오류는 `child_link`에서 `base_link`까지 이어지는 TF 경로가 끊어졌다는 의미다. 거의 대부분의 경우, `robot_state_publisher`가 TF를 제대로 발행하지 못하고 있기 때문이다. 그리고 그 주된 원인은 `robot_state_publisher`가 `/joint_states` 토픽을 수신하지 못하는 데 있다.34
  - **해결:** 다음을 순서대로 점검한다.
    1. `joint_state_publisher` 또는 `joint_state_publisher_gui` 노드가 실행 중인지 확인한다.
    2. `ros2 topic echo /joint_states` 명령어로 `/joint_states` 토픽에 데이터가 실제로 발행되고 있는지 확인한다.
    3. `rqt_graph`를 실행하여 `joint_state_publisher` -> `/joint_states` -> `robot_state_publisher` -> `/tf`로 이어지는 데이터 흐름이 정상적으로 연결되어 있는지 시각적으로 확인한다.
    4. 런치 파일에서 토픽 이름을 `remap` 했다면, 그 설정이 올바른지 다시 한번 확인한다.
- **문제 3: 메쉬 파일 로드 실패 ("Could not load mesh resource...")**
  - **원인:** `<mesh>` 태그의 `filename` 경로가 잘못되었을 때 발생한다. 흔한 원인으로는 `package://` 스킴의 오타(예: `package:/`), 패키지 이름 오타, 파일 경로 오타, 메쉬 파일 이름의 대소문자 불일치 등이 있다.14
  - **해결:**
    1. `filename="package://<package_name>/<path_to_mesh>"` 형식이 정확한지 철자 하나하나 확인한다.
    2. `setup.py`의 `data_files` 항목에 `meshes` 디렉토리가 올바르게 포함되어 있는지 확인한다. 이 설정이 누락되면 빌드 시 메쉬 파일이 설치 디렉토리로 복사되지 않아 찾을 수 없게 된다.13
    3. 파일 이름의 대소문자는 리눅스에서 구분되므로, 실제 파일명과 URDF에 적힌 이름이 정확히 일치하는지 확인한다.
- **문제 4: URDF 파일을 수정했는데 RViz에 반영되지 않음**
  - **원인:** ROS2의 빌드 시스템은 소스 코드를 수정했다고 해서 자동으로 변경사항을 반영하지 않는다. 사용자가 직접 빌드하고, 환경을 설정하고, 노드를 재시작해야 한다.33
  - **해결:** URDF/Xacro 파일을 수정한 후에는 반드시 다음의 3단계를 거쳐야 한다.
    1. **빌드:** `cd ~/ros2_ws && colcon build --packages-select <your_package_name>`
    2. **환경 설정:** `source install/setup.bash`
    3. **재시작:** 실행 중이던 런치 파일을 `Ctrl+C`로 종료하고 다시 실행한다.

효과적인 URDF 트러블슈팅은 단순히 `check_urdf`를 실행하는 '파일 검증'의 차원을 넘어선다. 이는 ROS2 시스템 전체의 데이터 흐름을 이해하고 진단하는 '시스템 디버깅'에 가깝다. RViz에서 모델이 보이지 않을 때, URDF 파일 자체의 문법 오류보다는 노드 간의 통신 문제, 빌드 시스템 설정 오류, 런치 파일 구성 오류일 가능성이 훨씬 높다. 따라서 `check_urdf`는 첫 단추일 뿐, 진짜 문제 해결은 `rqt_graph`로 노드 연결을 시각화하고, `ros2 topic echo`로 메시지 내용을 직접 들여다보며, RViz의 TF 디스플레이로 좌표계 트리의 연결 상태를 추적하는 등 ROS2의 강력한 디버깅 도구들을 종합적으로 활용하는 데서 시작된다. 이러한 시스템 수준의 접근 방식이야말로 초보자와 전문가를 가르는 결정적인 차이점이다.

### 6.3  유지보수와 확장을 고려한 URDF 설계 패턴

단순히 동작하는 것을 넘어, 다른 사람이 이해하기 쉽고, 수정하기 편하며, 확장이 용이한 URDF를 작성하는 것은 전문가의 중요한 덕목이다.

- **시뮬레이션과 실제 로봇 설정 분리:** 실제 로봇을 구동할 때는 필요 없는 Gazebo 플러그인이나 시뮬레이션용 센서 모델이 URDF에 포함되어 있으면 불필요한 의존성을 유발한다. 이를 해결하기 위해, 시뮬레이션 전용 설정(예: `<gazebo>` 태그)을 `my_robot.gazebo.xacro`와 같은 별도의 파일로 분리하고, 메인 Xacro 파일에서는 `<xacro:include>`와 `<xacro:if>`를 사용하여 필요할 때만 이 파일을 포함시키는 패턴을 사용하는 것이 좋다.8
- **체계적인 파일 구조화:** 프로젝트가 커질수록 파일 관리가 중요해진다. 모든 URDF/Xacro 파일은 `urdf/` 또는 `description/` 폴더에, 3D 메쉬 파일은 `meshes/`, RViz 설정 파일은 `rviz/`, Gazebo 관련 설정은 `config/` 폴더에 모아두는 등 표준적인 디렉토리 구조를 따르면 프로젝트의 가독성과 유지보수성이 크게 향상된다.13
- **Xacro의 적극적인 활용:** 코드의 재사용성과 가독성을 극대화하기 위해 Xacro를 최대한 활용해야 한다. 로봇의 모든 고정 수치(길이, 반경, 질량, 색상 등)는 파일 상단에 `<xacro:property>`로 정의하고, 두 번 이상 반복되는 구조(바퀴, 다리, 센서 마운트 등)는 무조건 `<xacro:macro>`로 만들어야 한다. 이는 URDF를 단순한 XML 문서가 아닌, 파라미터화된 설계도로 만들어준다.11

#### **참고 자료**

1. Lesson 9: Introduction to URDF - MiRobot, accessed July 26, 2025, https://www.mirobot.ai/ieee-beginner/chapter2/lesson9
2. URDF(Unified Robot Description Format)란?, accessed July 26, 2025, https://duvallee.tistory.com/11
3. URDF - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/URDF-Main.html
4. ROS URDF 작성하기. URDF | by brewmaster | newworld-kim - Medium, accessed July 26, 2025, https://medium.com/newworld-kim/ros-urdf-b6979bfa31aa
5. urdf/XML/link - ROS Wiki, accessed July 26, 2025, http://wiki.ros.org/urdf/XML/link
6. ROS2 URDF Tutorial - Describe Any Robot (Links and Joints) - YouTube, accessed July 26, 2025, https://www.youtube.com/watch?v=LsKL8N5Iwkw
7. Basics of Robot URDF Modeling in ROS2 (Humble Hawksbill) and How to Write Launch Files from Scratch : r/ControlRobotics - Reddit, accessed July 26, 2025, https://www.reddit.com/r/ControlRobotics/comments/1bfz5wf/basics_of_robot_urdf_modeling_in_ros2_humble/
8. Best Practice: URDF descriptions, real robots, gazebo plugins and dependencies, accessed July 26, 2025, https://robotics.stackexchange.com/questions/37682/best-practice-urdf-descriptions-real-robots-gazebo-plugins-and-dependencies
9. [ROS] URDF 알아보기, accessed July 26, 2025, [https://velog.io/@dazi2_2/ROS-URDF-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0](https://velog.io/@dazi2_2/ROS-URDF-알아보기)
10. How is the coordinate system working in URDF - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/85761/how-is-the-coordinate-system-working-in-urdf
11. urdf_example/description/example_robot.urdf.xacro at main - GitHub, accessed July 26, 2025, https://github.com/joshnewans/urdf_example/blob/main/description/example_robot.urdf.xacro
12. The many origins of URDF - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/82452/the-many-origins-of-urdf
13. HaofeiMa/urdf_ros2_rviz2: Visualize urdf model in ros2 using rviz2, a example ros2 package with modify tutorial. - GitHub, accessed July 26, 2025, https://github.com/HaofeiMa/urdf_ros2_rviz2
14. Did anybody know how to solve this problem >> URDF Errors loading geometries: : r/ROS, accessed July 26, 2025, https://www.reddit.com/r/ROS/comments/1cic4ja/did_anybody_know_how_to_solve_this_problem_urdf/
15. ros2_control extra bits - Articulated Robotics, accessed July 26, 2025, https://articulatedrobotics.xyz/tutorials/mobile-robot/applications/ros2_control-extra/
16. urdf/XML/joint - ROS Wiki, accessed July 26, 2025, http://wiki.ros.org/urdf/XML/joint
17. Help: URDF joint axis : r/ROS - Reddit, accessed July 26, 2025, https://www.reddit.com/r/ROS/comments/s5nlvq/help_urdf_joint_axis/
18. ROS2 - Create a Xacro Macro - YouTube, accessed July 26, 2025, https://www.youtube.com/watch?v=XH6Fvv7p5n0
19. Using Xacro to clean up your code - ROS 2 Documentation ..., accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Using-Xacro-to-Clean-Up-a-URDF-File.html
20. xacro (XML macro) - 노땅엔진니어의 로봇 이야기 - 티스토리, accessed July 26, 2025, https://duvallee.tistory.com/17
21. URDF, XACRO 마스터 하기 ! (ADDBOT Rviz에서 구현하기) (2/2) - 자동차 설계하기.., accessed July 26, 2025, https://kimbrain.tistory.com/409
22. URDF improvements - ROS General - Open Robotics Discourse, accessed July 26, 2025, https://discourse.openrobotics.org/t/urdf-improvements/30520
23. Using URDF with robot_state_publisher (C++) - ROS 2 Documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Using-URDF-with-Robot-State-Publisher-cpp.html
24. ros/robot_state_publisher: Allows you to publish the state of a robot (i.e the position of its base and all joints) via the "tf" transform library - GitHub, accessed July 26, 2025, https://github.com/ros/robot_state_publisher
25. How to Create URDF and Launch Files in ROS2 and Display Them in Rviz, accessed July 26, 2025, https://aleksandarhaber.com/how-to-create-urdf-and-launch-files-in-ros2-and-display-them-in-rviz/
26. [ROS2] launch file 작성하기 - velog, accessed July 26, 2025, [https://velog.io/@gaebalsebal/ROS2-launch-file-%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0](https://velog.io/@gaebalsebal/ROS2-launch-file-작성하기)
27. Using URDF with robot_state_publisher (Python) - ROS 2 ..., accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Using-URDF-with-Robot-State-Publisher-py.html
28. How to use XACRO files with Gazebo in ROS2 - The Construct, accessed July 26, 2025, https://www.theconstruct.ai/how-to-use-xacro-in-ros2-gazebo/
29. urdf_launch/README.md at main / ros/urdf_launch / GitHub, accessed July 26, 2025, https://github.com/ros/urdf_launch/blob/main/README.md
30. ros/urdf_launch: Launch files for common URDF operations - GitHub, accessed July 26, 2025, https://github.com/ros/urdf_launch
31. RViz User Guide - ROS 2 Documentation: Humble documentation, accessed July 26, 2025, https://docs.ros.org/en/humble/Tutorials/Intermediate/RViz/RViz-User-Guide/RViz-User-Guide.html
32. ROS2 Rviz2 Tutorial - Robot Simulation - YouTube, accessed July 26, 2025, https://www.youtube.com/watch?v=qWoGkPDg4N8
33. rviz - How to force update after updating urdf file? - Robotics Stack ..., accessed July 26, 2025, https://robotics.stackexchange.com/questions/76358/how-to-force-update-after-updating-urdf-file
34. error in urdf file - rviz - Robotics Stack Exchange, accessed July 26, 2025, https://robotics.stackexchange.com/questions/96118/error-in-urdf-file
35. Displaying two robots in the same RViz window - ros2 - Stack Overflow, accessed July 26, 2025, https://stackoverflow.com/questions/76276272/displaying-two-robots-in-the-same-rviz-window
36. 2.2.3 Checking for Correctness - TU Delft OCW, accessed July 26, 2025, https://ocw.tudelft.nl/course-lectures/2-2-3-checking-for-correctness/
37. urdf - ROS Wiki, accessed July 26, 2025, http://wiki.ros.org/urdf
38. urdf/Tutorials/Create your own urdf file - ROS Wiki, accessed July 26, 2025, [http://wiki.ros.org/urdf/Tutorials/Create%20your%20own%20urdf%20file](http://wiki.ros.org/urdf/Tutorials/Create your own urdf file)
39. ros kinetic - Why does RVIZ complain that I have to transform from ..., accessed July 26, 2025, https://robotics.stackexchange.com/questions/93253/why-does-rviz-complain-that-i-have-to-transform-from-right-wheel-to-base-link


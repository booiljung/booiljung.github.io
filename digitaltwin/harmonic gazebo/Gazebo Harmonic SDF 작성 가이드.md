# Gazebo Harmonic SDF 작성 가이드
[Gazebo Harmonic](./index.md)


이 섹션에서는 시뮬레이션 기술의 핵심인 SDFormat(SDF)의 기초를 다룬다. SDF의 목적과 핵심 구조를 이해하는 것은 Gazebo Harmonic에서 정교한 시뮬레이션 환경과 로봇을 구축하기 위한 첫걸음이다.


SDFormat(Simulation Description Format), 줄여서 SDF는 로봇 시뮬레이터를 위한 객체와 환경을 기술하는 XML 기반의 파일 포맷이다.1 본래 Gazebo 로봇 시뮬레이터의 일부로 개발되었지만, 현재는 과학 및 로봇 공학 애플리케이션 전반에 걸쳐 사용될 수 있도록 설계된 독립적인 사양이다. SDF는 로봇의 기구학적, 동역학적 속성뿐만 아니라, 정적 및 동적 객체, 조명, 지형, 물리 엔진의 모든 측면을 기술할 수 있는 강력하고 확장 가능한 포맷이다.2

SDF와 자주 비교되는 URDF(Unified Robot Description Format)와의 차이점을 이해하는 것이 중요하다. URDF는 단일 로봇의 기구학적 구조(링크와 조인트의 연결 관계)와 시각적 형태를 기술하는 데 중점을 둔다. 반면, SDF는 시뮬레이션 '월드' 전체를 기술할 수 있다. 여기에는 여러 로봇, 다양한 객체, 조명, 센서, 심지어 물리 법칙까지 포함된다. 특히 SDF는 URDF에서는 표현하기 어려운 폐쇄 루프 기구학(closed-loop kinematics)이나 상세한 물리 속성(마찰, 감쇠 등)을 네이티브로 지원하여 훨씬 더 현실적인 시뮬레이션을 가능하게 한다.3 이러한 이유로 SDF는 Gazebo의 네이티브 언어로 사용된다.

모든 SDF 파일은 XML 선언으로 시작하며, 최상위 루트 요소인 `<sdf>` 태그를 가진다.6

```XML
<?xml version="1.0"?>
<sdf version="1.9">
</sdf>
```

여기서 가장 중요한 속성은 `version`이다. 예를 들어 `<sdf version="1.9">`는 이 파일이 SDF 사양 1.9 버전을 준수함을 명시한다. Gazebo의 SDF 파싱 라이브러리인 `libsdformat`은 이 버전을 기준으로 파일의 유효성을 검사한다.1 Gazebo Harmonic은 최신 SDF 버전들을 지원하며, 새로운 버전일수록 더 많은 기능이 추가된다. 예를 들어, 

`libsdformat 14.0.0`에서 `<mimic>` 조인트 제약 조건이 추가되었으며, 이는 SDF 1.11 사양의 일부이다.8 만약 `<mimic>` 태그를 사용하면서 SDF 파일에 `<sdf version="1.7">`이라고 선언하면, 파서는 이를 유효하지 않은 요소로 간주하여 오류를 발생시킨다. 이처럼 `version` 속성은 단순한 메타데이터가 아니라, 파일의 내용과 기능 가용성을 결정하는 엄격한 계약과 같다. 따라서 사용하려는 기능에 맞는 올바른 버전을 명시하는 것이 매우 중요하다.


SDF 파일은 크게 두 가지 목적 중 하나를 위해 작성된다. 하나는 완전한 시뮬레이션 환경을 정의하는 것이고, 다른 하나는 재사용 가능한 단일 객체나 로봇을 정의하는 것이다. 이 두 목적은 각각 `<world>`와 `<model>` 태그에 의해 구분된다.9

월드 파일 (.world 또는 .sdf)

월드 파일은 하나 이상의 <world> 태그를 포함하며, 시뮬레이션 환경 전체를 캡슐화한다. <world> 태그 안에는 물리 엔진 설정, 조명, GUI 레이아웃, 그리고 시뮬레이션에 등장하는 모든 모델들이 포함된다.6 일반적으로 `gz sim my_world.sdf`와 같이 Gazebo를 실행할 때 이 파일을 인자로 전달한다.

```xml
<?xml version="1.0"?>
<sdf version="1.9">
  <world name="my_simulation_world">
    <physics... />
    <light... />
    <gui... />

    <model name="robot1">... </model>
    <include>
      <uri>model://ground_plane</uri>
    </include>
  </world>
</sdf>
```

모델 파일 (model.sdf)

모델 파일은 <sdf> 태그의 직계 자식으로 단 하나의 <model> 태그만을 가진다. 이 파일들은 특정 로봇이나 객체를 독립적으로 정의하며, 재사용성을 극대화하도록 설계되었다. 보통 meshes, materials 폴더와 함께 구조화된 디렉터리에 model.sdf라는 이름으로 저장되며, 월드 파일에서 <include> 태그와 URI(Uniform Resource Identifier)를 통해 참조된다.11

```XML
<?xml version="1.0"?>
<sdf version="1.9">
  <model name="reusable_robot">
    <link name="base_link">... </link>
    <joint name="wheel_joint">... </joint>
    </model>
</sdf>
```


이 섹션에서는 시뮬레이션 공간 자체, 즉 모델들이 상호작용하는 방식을 지배하는 물리, 조명, 그리고 전역 파라미터를 정의하는 방법을 상세히 다룬다.


`<world>` 요소는 시뮬레이션 장면 전체를 담는 최상위 컨테이너다. `name` 속성은 필수이며, 해당 월드를 고유하게 식별하는 데 사용된다.6


`<physics>` 태그는 시뮬레이션의 동역학 엔진을 설정하는 데 사용된다.6

`type` 속성을 통해 `ode`, `bullet`, `dart`와 같은 물리 엔진을 명시할 수 있다.6 하지만 Gazebo Harmonic과 같은 최신 버전에서는 물리 엔진 자체가 시스템 플러그인으로 로드되기 때문에, 이 `type` 속성은 `ignored`로 설정되는 경우가 많다.6

**주요 물리 파라미터:**

- `<max_step_size>`: 시뮬레이션의 시간 간격(time step)을 초 단위로 설정한다 (예: 1ms의 경우 `0.001`). 값이 작을수록 계산의 정확도는 높아지지만 더 많은 컴퓨팅 자원을 소모한다.6
- `<real_time_factor>`: 시뮬레이션 시간 대비 실제 시간의 목표 비율이다. `1.0`은 실시간, `1.0` 미만은 슬로우 모션, `1.0` 초과는 패스트 포워드를 의미한다.6

**중력 및 환경 설정:**

- `<gravity>`: 중력 가속도를 3차원 벡터로 정의한다. 일반적으로 `0 0 -9.8` m/s²를 사용한다.9
- `<wind>`: 전역적으로 적용되는 바람의 선형 속도를 정의할 수 있다.9
- `<atmosphere>`: 온도, 압력과 같은 대기 속성을 모델링한다. 이는 고도에 민감한 센서나 공기역학을 시뮬레이션할 때 중요한 요소가 된다.9

Gazebo Harmonic의 모듈식 아키텍처는 중요한 변화를 가져왔다. Gazebo Classic에서는 물리 엔진과 같은 핵심 기능이 암묵적으로 로드되었지만, Harmonic에서는 사용자가 월드 파일 내에서 이를 명시적으로 로드해야 한다. 만약 월드 파일에 물리 시스템 플러그인이 포함되어 있지 않으면, 모델들은 중력의 영향을 받지 않고 공중에 떠 있게 된다. 마찬가지로, 씬 브로드캐스터 플러그인이 없으면 GUI 클라이언트에서 시뮬레이션 장면을 볼 수 없다.15 따라서, 기능적으로 완전한 최소한의 월드 파일은 다음과 같이 핵심 시스템 플러그인들을 반드시 포함해야 한다.

```XML
<world name="minimal_functional_world">
  <plugin filename="gz-sim-physics-system" name="gz::sim::systems::Physics"></plugin>

  <plugin filename="gz-sim-user-commands-system" name="gz::sim::systems::UserCommands"></plugin>

  <plugin filename="gz-sim-scene-broadcaster-system" name="gz::sim::systems::SceneBroadcaster"></plugin>

  </world>
```


시뮬레이션 환경의 시각적 사실감을 높이기 위해 조명 설정은 필수적이다.

**전역 조명:**

- `<scene>`: 장면 전반에 걸친 시각적 속성을 담는 컨테이너다.
- `<ambient_light>`와 `<background_color>`: 각각 전역 주변광과 배경색을 RGBA(Red, Green, Blue, Alpha) 값으로 설정한다.6 값의 범위는 0에서 1 사이다.

광원 (<light>)

개별 광원을 추가하여 특정 영역을 비추거나 그림자를 생성할 수 있다.

- `name`과 `type`(`directional`, `point`, `spot`) 속성으로 광원을 정의한다.6
- `<pose>`: 광원의 위치와 방향을 설정한다.
- `<cast_shadows>`: `true`로 설정하면 해당 광원이 그림자를 생성한다.6
- `<diffuse>` 및 `<specular>`: 빛의 확산광과 정반사광 색상을 RGBA로 정의한다.6
- `<attenuation>`: 빛이 거리에 따라 어떻게 감쇠하는지를 `range`(도달 범위), `constant`, `linear`, `quadratic` 계수를 통해 설정한다.6


미리 정의된 모델들을 월드에 추가하는 가장 일반적인 방법은 `<include>` 태그를 사용하는 것이다.6

- `<uri>`: 모델을 가리키는 URI를 지정하는 핵심 요소다.
  - **Gazebo Fuel:** 온라인 모델 저장소인 Gazebo Fuel의 모델을 직접 참조할 수 있다. 예: `https://fuel.gazebosim.org/1.0/OpenRobotics/models/Coke`.6
  - **로컬 파일:** `model://` 접두사를 사용하여 로컬 시스템에 저장된 모델을 참조할 수 있다. 예: `model://my_robot_model`. Gazebo는 `GZ_SIM_RESOURCE_PATH` 환경 변수에 지정된 경로들에서 해당 모델 디렉터리를 검색한다.11

이 `GZ_SIM_RESOURCE_PATH` 환경 변수의 올바른 설정은 커스텀 모델을 사용하는 프로젝트의 성패를 좌우하는 핵심 요소다. 많은 사용자들이 "모델을 찾을 수 없다"는 오류를 겪는 주된 원인은 이 환경 변수가 설정되지 않았거나 잘못된 경로를 가리키고 있기 때문이다. 프로젝트를 시작할 때, 모델 파일들이 위치한 디렉터리를 이 환경 변수에 추가하는 것을 습관화해야 한다.11

```Bash
# ~/.bashrc 또는 쉘 설정 파일에 추가
export GZ_SIM_RESOURCE_PATH=$GZ_SIM_RESOURCE_PATH:/path/to/your/project/models
```

`<include>` 태그 내에서 모델의 `<name>`이나 `<pose>`와 같은 속성을 직접 덮어쓸 수도 있어, 동일한 모델을 다른 이름과 위치로 여러 번 재사용할 수 있다.10


`<gui>` 태그를 사용하면 월드 파일을 로드할 때 Gazebo 클라이언트의 레이아웃과 플러그인을 미리 정의할 수 있다.6 이를 통해 시뮬레이션을 시작할 때마다 수동으로 창을 배열할 필요 없이 일관된 작업 환경을 구성할 수 있다.

```XML
<gui fullscreen="false">
  <plugin filename="MinimalScene" name="3D View">
    <gz-gui>
      <title>3D View</title>
      <property type="string" key="state">docked</property>
    </gz-gui>
    <engine>ogre2</engine>
    <scene>scene</scene>
  </plugin>

  <plugin filename="WorldControl" name="World control">
    <gz-gui>
      <title>World control</title>
      <property type="bool" key="showTitleBar">false</property>
      <property type="string" key="state">docked_collapsed</property>
    </gz-gui>
  </plugin>
</gui>
```

위 예제에서는 3D 뷰와 월드 제어 플러그인을 로드하고, 각각의 제목, 상태(도킹됨, 플로팅 등)를 `<property>` 태그를 통해 설정한다.6


이 섹션은 로봇 모델을 구성하는 각 요소를 세분화하여 설명하는 이 가이드의 핵심 부분이다. 링크, 조인트, 센서 등 로봇을 구성하는 모든 요소를 SDF로 정의하는 방법을 다룬다.


`<model>` 태그는 링크, 조인트, 플러그인 및 다른 중첩된 모델들의 집합을 담는 컨테이너다.4

`name` 속성은 해당 범위 내에서 고유해야 한다.10

**주요 속성:**

- `<static>`: `true`로 설정하면 모델이 움직이지 않게 고정된다. 이는 지면이나 건물과 같은 정적 환경 객체에 사용하여 시뮬레이션 성능을 최적화하는 데 유용하다.10
- `<self_collide>`: `true`로 설정하면 동일한 모델 내의 링크들이 서로 충돌할 수 있게 된다. 기본값은 `false`이며, 조인트로 연결된 링크들은 이 설정과 관계없이 서로 충돌하지 않는다.10
- `<pose>`: 월드와 같은 부모 프레임에 대한 모델의 초기 위치와 방향을 정의한다.19

SDF의 포즈 시스템은 좌표 변환의 계층적 트리 구조를 따른다. 포즈를 잘못 이해하면 모델이 의도치 않은 형태로 조립될 수 있다. 각 요소의 `<pose>` 태그는 기본적으로 자신의 부모 XML 요소의 프레임을 기준으로 정의된다. 예를 들어, `<link>`의 포즈는 `<model>` 프레임을 기준으로, `<visual>`의 포즈는 `<link>` 프레임을 기준으로 정의된다. 하지만 `<pose relative_to="...">` 속성을 사용하면 이 기준 프레임을 명시적으로 변경할 수 있어 복잡한 모델링을 가능하게 한다.19 특히, `<joint>`의 포즈는 기본적으로 *자식 링크*의 프레임을 기준으로 정의된다는 점은 매우 중요하며, 자주 혼동을 일으키는 부분이다.22


`<link>`는 물리적 속성을 가진 단일 강체(rigid body)를 나타낸다. 로봇의 몸통, 팔, 바퀴 등이 모두 링크에 해당하며, 모델을 구성하는 가장 기본적인 빌딩 블록이다.4


`<inertial>` 태그는 링크의 동역학적 특성을 정의한다. 물리 엔진은 이 값을 사용하여 링크의 움직임을 계산한다.4

- `<mass>`: 링크의 질량을 킬로그램(kg) 단위로 지정한다.
- `<pose>`: 링크의 원점 프레임에 대한 질량 중심(center of mass)의 상대적인 위치와 방향을 정의한다.
- `<inertia>`: 3x3 관성 행렬을 정의한다. 대칭 행렬이므로 6개의 값(`<ixx>`, `<iyy>`, `<izz>`, `<ixy>`, `<ixz>`, `<iyz>`)으로 지정한다.23 이 값들을 수동으로 계산하는 것은 복잡하며, 종종 CAD 소프트웨어나 외부 도구를 사용한다.


`<collision>`은 물리 엔진이 충돌 감지를 위해 사용하는 형상을 정의한다. 시뮬레이션 성능에 직접적인 영향을 미치므로 가능한 한 단순한 형태로 유지하는 것이 매우 중요하다.4

- `<geometry>`: 형상의 모양을 정의한다.
  - `<box>`, `<sphere>`, `<cylinder>`, `<capsule>`, `<ellipsoid>`: 단순한 기하학적 원시 형태(primitive shape)를 사용한다. 이들은 계산 비용이 매우 저렴하다.4
  - `<mesh>`: 복잡한 형상을 위해 `.dae`, `.stl`, `.obj`와 같은 3D 모델 파일을 사용한다. 충돌 메시(mesh)는 시각 메시보다 훨씬 낮은 폴리곤 수를 갖도록 최적화해야 한다.4
- `<surface>`: 표면의 물리적 특성을 정의한다.
  - `<friction>`: 마찰 계수(`mu`, `mu2`)와 같은 마찰 속성을 설정한다.
  - `<contact>`: 접촉 시의 강성(`kp`) 및 감쇠(`kd`), 반발 계수 등을 설정하여 표면의 부드러움이나 탄성을 조절할 수 있다.25


`<visual>`은 GUI에 렌더링되는 형상을 정의한다. 이는 충돌 지오메트리와 독립적이므로, 시뮬레이션 성능에 영향을 주지 않으면서 시각적으로 복잡하고 상세한 모델을 사용할 수 있다.4

- `<geometry>`: `<collision>`에서와 동일한 태그들을 사용하지만, 보통 더 높은 폴리곤의 고품질 메시 파일을 가리킨다.

- `<material>`: 링크의 색상과 질감을 정의한다. 최신 Gazebo에서는 PBR(Physically-Based Rendering) 재질을 위해 `<ambient>`, `<diffuse>`, `<specular>` 태그를 사용하는 것이 표준이다.13 Gazebo Classic과의 호환성을 위해 

  `<script>` 태그를 사용하여 `Gazebo/SkyBlue`와 같은 미리 정의된 색상을 사용할 수도 있다.3

- `<cast_shadows>`와 `<transparency>`: 각각 그림자 생성 여부와 투명도를 제어한다.27

`<visual>`과 `<collision>` 지오메트리를 분리하는 것은 SDF의 핵심적인 최적화 전략이다. 많은 개발자들이 실수로 동일한 고품질 CAD 메시를 두 요소 모두에 사용하는데, 이는 불필요한 물리 계산을 유발하여 시뮬레이션 성능을 심각하게 저하시키는 주된 원인이다. 로봇 팔을 모델링할 때, `<collision>`은 단순한 박스와 캡슐의 조합으로 만들고, `<visual>`은 상세한 CAD 메시를 사용하는 것이 모범 사례다.4


`<joint>`는 두 링크 사이의 기구학적 제약을 정의하여 이들의 상대적인 움직임을 제한한다.19


- `<parent>`와 `<child>`: 연결될 두 링크의 이름을 지정한다. `parent` 링크의 이름으로 `world`를 사용하면 해당 조인트의 `child` 링크를 월드 좌표계에 고정시킬 수 있다.19
- `<pose>`: 조인트의 좌표 프레임을 정의한다. 기본적으로 자식 링크의 프레임을 기준으로 표현된다.19


`type` 속성은 필수이며, 조인트의 운동 방식을 결정한다. 아래 표는 자주 사용되는 조인트 유형을 요약한 것이다.

| 조인트 유형 (Joint Type) | 자유도 (DoF) | 주요 운동             | 주요 SDF 요소                             |
| ------------------------ | ------------ | --------------------- | ----------------------------------------- |
| `revolute`               | 1            | 축 중심 회전          | `<axis>`, `<limit>`, `<lower>`, `<upper>` |
| `continuous`             | 1            | 축 중심 무한 회전     | `<axis>`                                  |
| `prismatic`              | 1            | 축 방향 직선 운동     | `<axis>`, `<limit>`, `<lower>`, `<upper>` |
| `fixed`                  | 0            | 두 링크를 완전히 고정 | -                                         |
| `ball`                   | 3            | 모든 방향 회전        | -                                         |
| `screw`                  | 1            | 회전과 직선 운동 결합 | `<axis>`, `<thread_pitch>`                |

자료 출처: 19

`<axis>` (그리고 `revolute2`나 `universal` 조인트의 경우 `<axis2>`)는 운동 축을 정의한다.

- `<xyz>`: 조인트 프레임 내에서 운동 축의 방향을 나타내는 단위 벡터를 지정한다. `expressed_in` 속성을 사용하여 다른 기준 프레임을 명시할 수도 있다.19


- `<limit>`: 조인트의 운동 범위를 제한한다.
  - `<lower>` 및 `<upper>`: 최소 및 최대 위치를 지정한다 (회전 조인트는 라디안, 직선 조인트는 미터).22
  - `<effort>`: 조인트가 발휘할 수 있는 최대 힘 또는 토크를 뉴턴 또는 뉴턴-미터 단위로 제한한다.22
  - `<velocity>`: 조인트의 최대 속도를 제한한다.22
- `<dynamics>`: 조인트 운동의 물리적 특성을 정의한다.
  - `<damping>`: 속도에 비례하는 점성 감쇠 계수. 조인트의 움직임을 부드럽게 만드는 데 사용된다.22
  - `<friction>`: 정지 마찰 계수.22


`<sensor>` 태그는 링크에 센서를 부착하여 시뮬레이션된 월드로부터 데이터를 수집할 수 있게 한다.4

**일반적인 센서 속성:**

- `name`과 `type`은 필수 속성이다. `type`은 `camera`, `imu`, `lidar`, `contact`, `gps` 등이 될 수 있다.28
- `<update_rate>`: 센서가 데이터를 생성하는 빈도를 헤르츠(Hz) 단위로 설정한다.28
- `<visualize>`: `true`로 설정하면 센서의 측정 범위나 형태가 GUI에 시각화된다.28
- `<topic>`: 센서 데이터가 발행될 Gazebo Transport 토픽의 이름을 지정한다.28

주요 센서별 설정:

아래 표는 자주 사용되는 센서들의 핵심 설정 파라미터를 요약한 것이다.

| 센서 유형 (Sensor Type) | 주요 파라미터 요소      | 설명                                      | 예시 값                                                      |
| ----------------------- | ----------------------- | ----------------------------------------- | ------------------------------------------------------------ |
| `camera`                | `<horizontal_fov>`      | 수평 시야각 (라디안)                      | `1.047`                                                      |
|                         | `<image>`               | 이미지 너비, 높이, 픽셀 포맷              | `<width>640</width> <height>480</height>`                    |
|                         | `<clip>`                | 렌더링할 최소/최대 거리                   | `<near>0.1</near> <far>100</far>`                            |
|                         | `<noise>`               | 이미지에 추가할 노이즈 모델 (가우시안 등) | `<type>gaussian</type> <stddev>0.007</stddev>`               |
| `imu`                   | `<angular_velocity>`    | 각속도 측정에 대한 노이즈 모델            | `<x><noise type="gaussian">...</noise></x>`                  |
|                         | `<linear_acceleration>` | 선형 가속도 측정에 대한 노이즈 모델       | `<z><noise type="gaussian">...</noise></z>`                  |
| `lidar` (or `ray`)      | `<scan>`                | 수평/수직 스캔 설정 (샘플 수, 각도 범위)  | `<horizontal><samples>360</samples>...</horizontal>`         |
|                         | `<range>`               | 측정 가능한 최소/최대 거리                | `<min>0.1</min> <max>30.0</max>`                             |
|                         | `<noise>`               | 거리 측정값에 대한 노이즈 모델            | `<type>gaussian</type> <mean>0.0</mean> <stddev>0.01</stddev>` |

자료 출처: 28


이 섹션에서는 Gazebo Harmonic에서 새롭게 도입되거나 크게 개선된 고급 기능들을 다룬다. 이러한 기능들은 더 현실적이고 복잡한 시뮬레이션을 더 쉽게 만들 수 있도록 돕는다.


**개념:** Gazebo Harmonic은 링크의 충돌 지오메트리를 기반으로 질량, 질량 중심, 관성 텐서를 자동으로 계산하는 기능을 제공한다. 이는 MeshLab과 같은 외부 도구를 사용하여 수동으로 관성 값을 계산하고 SDF 파일에 복사/붙여넣기하던 기존의 번거로운 작업을 대체한다.23

**구현 방법:**

1. `<inertial>` 태그에 `auto="true"` 속성을 추가한다: `<inertial auto="true" />`.32
2. 이 속성이 설정되면, 해당 `<inertial>` 태그 내에 수동으로 입력된 `<mass>`, `<pose>`, `<inertia>` 값들은 무시되고 자동 계산된 값으로 덮어쓰여진다.33
3. 계산의 기반이 되는 각 `<collision>` 요소 내부에 `<density>` 태그를 추가하여 재료의 밀도를 kg/m³ 단위로 지정한다.32 만약 하나의 링크에 여러 개의 충돌 지오메트리가 있다면, Gazebo는 각 지오메트리의 밀도를 고려하여 전체 링크의 관성 속성을 종합한다.32
4. 만약 `auto="true"`와 함께 `<inertial>` 태그 내에 `<mass>` 값이 명시적으로 주어지면, Gazebo는 `<density>`로 정의된 질량 분포 비율을 유지하면서 전체 질량이 명시된 값과 일치하도록 관성 속성들을 스케일링한다.32

예제: 자동 관성 계산을 적용한 진자 모델

아래는 두 개의 동일한 구조를 가진 진자 모델을 포함하는 월드 SDF 예제다. 왼쪽 진자는 피봇(pivot) 부분의 밀도가 높고, 오른쪽 진자는 추(bob) 부분의 밀도가 높다. 이 밀도 분포의 차이로 인해 두 진자의 질량 중심과 관성 모멘트가 달라져 서로 다른 동적 행동을 보인다. 이 예제는 auto_inertia_pendulum.sdf 파일로 제공되며, gz sim auto_inertia_pendulum.sdf 명령으로 실행해볼 수 있다.32

```XML
<?xml version="1.0"?>
<sdf version="1.11">
  <world name="pendulum_world">
    <plugin filename="gz-sim-physics-system" name="gz::sim::systems::Physics"/>
    <plugin filename="gz-sim-scene-broadcaster-system" name="gz::sim::systems::SceneBroadcaster"/>
    
    <model name="pendulum_pivot_heavy">
      <pose>-1 0 2 0 0 0</pose>
      <link name="pendulum_link">
        <inertial auto="true"/>
        <collision name="pivot_collision">
          <pose>0 0 0.5 0 0 0</pose>
          <density>8000</density> <geometry><cylinder><radius>0.1</radius><length>0.2</length></cylinder></geometry>
        </collision>
        <collision name="rod_collision">
          <pose>0 0 0 0 0 0</pose>
          <density>1000</density>
          <geometry><cylinder><radius>0.05</radius><length>1.0</length></cylinder></geometry>
        </collision>
        <collision name="bob_collision">
          <pose>0 0 -0.5 0 0 0</pose>
          <density>2000</density>
          <geometry><sphere><radius>0.15</radius></sphere></geometry>
        </collision>
        </link>
      <joint name="pivot_joint" type="revolute">
        <parent>world</parent>
        <child>pendulum_link</child>
        <axis><xyz>1 0 0</xyz></axis>
      </joint>
    </model>

    <model name="pendulum_bob_heavy">
      <pose>1 0 2 0 0 0</pose>
      <link name="pendulum_link">
        <inertial auto="true"/>
        <collision name="pivot_collision">
          <pose>0 0 0.5 0 0 0</pose>
          <density>2000</density>
          <geometry><cylinder><radius>0.1</radius><length>0.2</length></cylinder></geometry>
        </collision>
        <collision name="rod_collision">
          <pose>0 0 0 0 0 0</pose>
          <density>1000</density>
          <geometry><cylinder><radius>0.05</radius><length>1.0</length></cylinder></geometry>
        </collision>
        <collision name="bob_collision">
          <pose>0 0 -0.5 0 0 0</pose>
          <density>8000</density> <geometry><sphere><radius>0.15</radius></sphere></geometry>
        </collision>
        </link>
      <joint name="pivot_joint" type="revolute">
        <parent>world</parent>
        <child>pendulum_link</child>
        <axis><xyz>1 0 0</xyz></axis>
      </joint>
    </model>
  </world>
</sdf>
```


**개념:** 과거 `<gearbox>` 조인트를 대체하는 `<mimic>` 제약은 두 조인트 축의 위치 사이에 선형 관계를 설정한다. 이는 하나의 구동 조인트가 다른 조인트들의 움직임을 결정하는 평행 그리퍼, 가위 리프트, 4절 링크와 같은 메커니즘을 모델링하는 데 필수적이다.31

**구현 방법:**

1. `<mimic>` 태그는 구동될 조인트, 즉 *팔로워(follower)* 조인트의 `<axis>` 태그 내부에 위치한다.
2. **속성:**
   - `joint`: 움직임을 따라갈 *리더(leader)* 조인트의 이름을 지정한다.
   - `axis` (선택 사항): 리더 조인트가 다축 조인트일 경우, 모방할 특정 축의 이름을 지정한다 (`axis` 또는 `axis2`).22
3. **자식 요소:**
   - `<multiplier>`: 움직임의 비율. `-1.0`으로 설정하면 팔로워 조인트는 리더와 반대 방향으로 움직인다.
   - `<offset>`: 팔로워 조인트의 위치에 더해지는 상수 오프셋.
   - `<reference>` (선택 사항): 리더 조인트의 위치에서 `multiplier`를 적용하기 전에 빼는 오프셋.
   - 이들의 관계는 다음 수식으로 정의된다: follower_pos=multiplier×(leader_pos−reference)+offset.22

예제: 미믹 조인트를 사용한 간단한 그리퍼

아래는 한쪽 손가락(leader)의 움직임이 다른 쪽 손가락(follower)을 대칭적으로 움직이게 하는 간단한 2지 그리퍼 모델이다. gz sim joint_mimic.sdf와 같은 명령으로 예제 월드를 실행할 수 있다.35

```XML
<?xml version="1.0"?>
<sdf version="1.11">
  <model name="mimic_gripper">
    <link name="base">
      <pose>0 0 0.5 0 0 0</pose>
      <collision name="base_collision"><geometry><box><size>0.1 0.2 0.1</size></box></geometry></collision>
      <visual name="base_visual"><geometry><box><size>0.1 0.2 0.1</size></box></geometry></visual>
    </link>

    <link name="left_finger">
      <pose>0 0.15 0 0 0 0</pose>
      <collision name="left_collision"><geometry><box><size>0.1 0.1 0.1</size></box></geometry></collision>
      <visual name="left_visual"><geometry><box><size>0.1 0.1 0.1</size></box></geometry></visual>
    </link>

    <link name="right_finger">
      <pose>0 -0.15 0 0 0 0</pose>
      <collision name="right_collision"><geometry><box><size>0.1 0.1 0.1</size></box></geometry></collision>
      <visual name="right_visual"><geometry><box><size>0.1 0.1 0.1</size></box></geometry></visual>
    </link>

    <joint name="left_finger_joint" type="prismatic">
      <parent>base</parent>
      <child>left_finger</child>
      <axis>
        <xyz>0 1 0</xyz>
        <limit><lower>0</lower><upper>0.05</upper></limit>
      </axis>
    </joint>

    <joint name="right_finger_joint" type="prismatic">
      <parent>base</parent>
      <child>right_finger</child>
      <axis>
        <xyz>0 1 0</xyz>
        <limit><lower>-0.05</lower><upper>0</upper></limit>
        <mimic joint="left_finger_joint" multiplier="-1" offset="0"/>
      </axis>
    </joint>
  </model>
</sdf>
```

이러한 내장 기능들은 Gazebo가 외부 도구나 복잡한 플러그인에 대한 의존도를 줄이고, SDF 파일 자체에 더 많은 물리적 시스템을 선언적으로 기술할 수 있도록 발전하고 있음을 보여준다. 이는 개발자가 물리 구현의 복잡성보다 로봇 설계 자체에 더 집중할 수 있게 해주는 중요한 패러다임의 전환이다.


기존 Gazebo 사용자를 위해 이전 버전에서 Harmonic으로 전환할 때 필요한 주요 변경 사항을 요약한다.

| 기능/개념                   | Gazebo Classic / Ignition             | Gazebo Harmonic                                    | 참고사항                                                    |
| --------------------------- | ------------------------------------- | -------------------------------------------------- | ----------------------------------------------------------- |
| **시뮬레이터 실행 파일**    | `gazebo` / `ign gazebo`               | `gz sim`                                           | `ign`에서 `gz`로의 전면적인 이름 변경이 있었다.36           |
| **리소스 경로 환경 변수**   | `GAZEBO_MODEL_PATH`                   | `GZ_SIM_RESOURCE_PATH`                             | 모델, 월드 등 모든 리소스 경로를 통합한다.15                |
| **플러그인 경로 환경 변수** | `GAZEBO_PLUGIN_PATH`                  | `GZ_SIM_SYSTEM_PLUGIN_PATH`                        | 시스템 플러그인 경로가 분리되었다.15                        |
| **플러그인 SDF 문법**       | `<plugin filename="..." name="..."/>` | `<plugin filename="..." name="C++::Class::Name"/>` | `name` 속성에 C++ 클래스 전체 이름을 명시해야 한다.15       |
| **재질(Material) 정의**     | `<script>` (Ogre 전용)                | `<material>` (PBR 기반)                            | PBR 기반의 `<ambient>`, `<diffuse>` 사용이 표준이다.15      |
| **핵심 C++/CLI 네이밍**     | `ignition`, `ign`                     | `gazebo`, `gz`                                     | 라이브러리, 네임스페이스, 명령어 등 모든 것이 변경되었다.36 |

자료 출처: 15

Gazebo Classic에서 Harmonic으로의 마이그레이션은 단순한 이름 변경 이상이다. 플러그인 아키텍처가 완전히 다르기 때문에, Classic용으로 작성된 플러그인은 Harmonic에서 작동하지 않으며, Harmonic의 시스템 플러그인으로 대체하거나 새로 작성해야 한다.


플러그인은 시뮬레이션에 동적으로 코드를 주입하여 기본 기능을 확장하거나 외부 시스템과 연동하는 강력한 메커니즘이다. 이 섹션에서는 SDF를 통해 플러그인을 로드하고 설정하는 방법을 설명한다.


`<plugin>`은 동적으로 로드되는 공유 라이브러리를 시뮬레이션에 추가한다.38 이 태그는 `<world>`, `<model>`, `<link>`, `<sensor>`, `<gui>` 등 다양한 요소의 자식으로 위치할 수 있으며, 어느 요소에 속하느냐에 따라 플러그인이 접근할 수 있는 API의 범위가 달라진다.10

**주요 속성:**

- `filename`: 로드할 공유 라이브러리의 이름이다 (예: `gz-sim-diff-drive-system`). 플랫폼 간 호환성을 위해 `lib` 접두사나 `.so` 확장자는 생략하는 것이 일반적이다.39
- `name`: 플러그인 인스턴스의 고유한 이름이다. 최신 Gazebo 시스템 플러그인의 경우, 이 이름은 플러그인의 C++ 클래스 이름과 일치해야 한다 (예: `gz::sim::systems::DiffDrive`).15


`<plugin>` 태그 내부에 중첩된 모든 커스텀 XML 요소들은 해당 플러그인의 설정값으로 전달된다. 이는 플러그인을 재사용 가능하고 유연하게 만드는 핵심적인 방법이다.39

예제: 차동 구동(DiffDrive) 시스템

차동 구동 로봇을 제어하는 내장 플러그인의 SDF 설정 예제는 다음과 같다.13

```XML
<model name="diff_drive_robot">
 ...
  <plugin filename="gz-sim-diff-drive-system" name="gz::sim::systems::DiffDrive">
    <left_joint>left_wheel_joint</left_joint>
    <right_joint>right_wheel_joint</right_joint>
    <wheel_separation>1.2</wheel_separation>
    <wheel_radius>0.4</wheel_radius>
    <topic>/cmd_vel</topic>
  </plugin>
</model>
```

`left_joint`, `right_joint`와 같은 태그들은 `gz-sim-diff-drive-system` 플러그인 내부에서 파싱되어 사용될 커스텀 파라미터다.

C++에서 파라미터 파싱 (Configure 메서드)

설정값이 필요한 시스템 플러그인은 ISystemConfigure 인터페이스를 구현한다.41 이 인터페이스는 `Configure`라는 메서드를 제공하며, 시뮬레이션 시작 전에 Gazebo에 의해 호출된다.

```
void Configure(const Entity &_entity, const std::shared_ptr<const sdf::Element> &_sdf,...)
```

여기서 `_sdf` 파라미터는 SDF 파일의 `<plugin>` 요소를 가리키는 포인터다. 개발자는 이 `sdf::Element` 객체의 API를 사용하여 커스텀 파라미터를 읽어올 수 있다. `sdf::Element` API는 단순한 키-값 쌍을 넘어 중첩된 복잡한 XML 구조도 파싱할 수 있는 범용 XML DOM(Document Object Model) 인터페이스를 제공한다.42

아래는 `DiffDrive` 플러그인이 파라미터를 파싱하는 방식을 개념적으로 보여주는 C++ 코드 스니펫이다.44

```C++
#include <gz/sim/System.hh>
#include <sdf/Element.hh>

class MyDiffDrive : public gz::sim::System,
                    public gz::sim::ISystemConfigure
{
  //...

  void Configure(const gz::sim::Entity &_entity,
                 const std::shared_ptr<const sdf::Element> &_sdf,
                 gz::sim::EntityComponentManager &_ecm,
                 gz::sim::EventManager &_eventMgr) override
  {
    // Get<T> 템플릿을 사용하여 값을 읽는다. 두 번째 인자는 기본값이다.
    this->leftJointName = _sdf->Get<std::string>("left_joint", "default_left_joint").first;
    this->rightJointName = _sdf->Get<std::string>("right_joint", "default_right_joint").first;
    this->wheelSeparation = _sdf->Get<double>("wheel_separation", 1.0).first;
    this->wheelRadius = _sdf->Get<double>("wheel_radius", 0.2).first;

    // HasElement로 요소 존재 여부 확인 가능
    if (_sdf->HasElement("topic"))
    {
      this->topic = _sdf->Get<std::string>("topic");
    }
  }

private:
  std::string leftJointName;
  std::string rightJointName;
  double wheelSeparation;
  double wheelRadius;
  std::string topic;
};
```

이처럼 플러그인에 전달되는 `sdf::Element`는 강력한 XML 데이터 인터페이스 역할을 한다. 개발자는 자신의 플러그인이 복잡한 리스트, 중첩된 속성, 혹은 자신만의 "미니 언어"를 SDF 내에서 파싱하도록 설계할 수 있으며, 이를 통해 매우 유연하고 재사용성 높은 플러그인을 만들 수 있다.


Gazebo Harmonic에서의 SDF 작성은 단순한 파일 생성을 넘어, 시뮬레이션의 물리적, 시각적, 기능적 모든 측면을 정교하게 설계하는 과정이다. 이 가이드는 SDF의 기본 구조부터 Gazebo Harmonic의 최신 고급 기능에 이르기까지 포괄적인 내용을 다루었다.

핵심적인 사항들은 다음과 같다:

1. **버전의 중요성:** SDF 파일 상단의 `version` 속성은 단순한 주석이 아니다. 이는 사용 가능한 기능과 문법을 결정하는 엄격한 계약이므로, 사용하려는 기능에 맞는 버전을 명시해야 한다.
2. **모듈성과 명시성:** Gazebo Harmonic은 모듈식 아키텍처를 채택했다. 이는 물리 엔진이나 씬 렌더링과 같은 핵심 기능조차도 월드 파일 내에서 `<plugin>` 태그를 통해 명시적으로 로드해야 함을 의미한다.
3. **성능 최적화:** `<visual>`과 `<collision>` 지오메트리를 분리하는 것은 효율적인 시뮬레이션을 위한 필수적인 모범 사례다. 물리 계산에는 단순한 형태를, 시각화에는 복잡한 메시를 사용해야 한다.
4. **선언적 모델링:** 관성 자동 계산(`<inertial auto="true">`)과 조인트 미믹 제약(`<mimic>`)과 같은 Harmonic의 새로운 기능들은 복잡한 물리 시스템을 외부 도구나 커스텀 코드 없이 SDF 내에서 직접 선언적으로 기술할 수 있게 해준다. 이는 개발 생산성을 크게 향상시킨다.
5. **확장성:** 플러그인 시스템과 `sdf::Element` API는 Gazebo의 기능을 무한히 확장할 수 있는 통로를 제공한다. SDF 내에 정의된 커스텀 파라미터를 통해 C++ 플러그인의 동작을 세밀하게 제어할 수 있다.

이 가이드에서 제공된 지식과 예제들을 바탕으로, 개발자는 Gazebo Harmonic의 모든 잠재력을 활용하여 현실에 가까운, 복잡하고 유용한 로봇 시뮬레이션을 구축할 수 있을 것이다.


1. sdformat - Gazebo documentation, accessed July 26, 2025, https://gazebosim.org/libs/sdformat/
2. SDFormat Home, accessed July 26, 2025, http://sdformat.org/
3. SDFormat extensions to URDF (the 'gazebo' tag) - Documentation, accessed July 26, 2025, http://sdformat.org/tutorials?tut=sdformat_urdf_extensions&cat=specification&
4. Tutorial : Make a model - Gazebo Classic, accessed July 26, 2025, https://classic.gazebosim.org/tutorials?tut=build_model
5. [Gazebo in 5 minutes] 004 - How to create a gazebo model using SDF - YouTube, accessed July 26, 2025, https://www.youtube.com/watch?v=3YhW04wIjEc&pp=0gcJCfcAhR29_xXO
6. SDF worlds - Gazebo ionic documentation, accessed July 26, 2025, https://gazebosim.org/docs/latest/sdf_worlds/
7. Quickstart - Documentation - SDFormat, accessed July 26, 2025, http://sdformat.org/tutorials?tut=quickstart&cat=developers&
8. sdformat/Changelog.md at sdf15 - GitHub, accessed July 26, 2025, https://github.com/gazebosim/sdformat/blob/sdf15/Changelog.md
9. SDFormat Specification, accessed July 26, 2025, http://sdformat.org/spec?ver=1.12&elem=world
10. SDFormat Specification, accessed July 26, 2025, http://sdformat.org/spec?ver=1.12&elem=model
11. docs/harmonic/Model_insertion_fuel.md at master / gazebosim/docs - GitHub, accessed July 26, 2025, https://github.com/gazebosim/docs/blob/master/harmonic/Model_insertion_fuel.md
12. How to Load a Robot Model (SDF Format) into Gazebo – ROS 2 - Automatic Addison, accessed July 26, 2025, https://automaticaddison.com/how-to-load-a-robot-model-sdf-format-into-gazebo-ros-2/
13. docs/harmonic/tutorials/moving_robot/moving_robot.sdf at master / gazebosim/docs - GitHub, accessed July 26, 2025, https://github.com/gazebosim/docs/blob/master/harmonic/tutorials/moving_robot/moving_robot.sdf
14. Gazebo Harmonic slow real time factor on baylands world - ArduPilot Discourse, accessed July 26, 2025, https://discuss.ardupilot.org/t/gazebo-harmonic-slow-real-time-factor-on-baylands-world/125284
15. Migration from Gazebo classic: SDF, accessed July 26, 2025, https://gazebosim.org/api/sim/8/migrationsdf.html
16. docs/harmonic/sdf_worlds.md at master / gazebosim/docs - GitHub, accessed July 26, 2025, https://github.com/gazebosim/docs/blob/master/harmonic/sdf_worlds.md
17. GUI Configuration - Gazebo Sim, accessed July 26, 2025, https://gazebosim.org/api/sim/7/gui_config.html
18. Server Configuration - Gazebo Sim, accessed July 26, 2025, https://gazebosim.org/api/sim/7/server_config.html
19. sdf_tutorials/spec_model_kinematics/tutorial.md at master - GitHub, accessed July 26, 2025, https://github.com/gazebosim/sdf_tutorials/blob/master/spec_model_kinematics/tutorial.md
20. Tutorial : Building a world - Gazebo Classic, accessed July 26, 2025, https://classic.gazebosim.org/tutorials?tut=build_world
21. SDFormat Specification - Documentation, accessed July 26, 2025, http://sdformat.org/tutorials
22. 
23. Tutorial : Inertial parameters of triangle meshes - Gazebo Classic, accessed July 26, 2025, https://classic.gazebosim.org/tutorials?tut=inertia
24. Gazebo Release Features - Gazebo ionic documentation, accessed July 26, 2025, https://gazebosim.org/docs/latest/release-features/
25. Tutorial : Physics Parameters - Gazebo Classic, accessed July 26, 2025, https://classic.gazebosim.org/tutorials?tut=physics_params
26. Migration support for models with material scripts - General - Gazebo Community, accessed July 26, 2025, https://community.gazebosim.org/t/migration-support-for-models-with-material-scripts/2471
27. SDFormat Specification, accessed July 26, 2025, http://sdformat.org/spec?elem=visual
28. SDFormat Specification, accessed July 26, 2025, http://sdformat.org/spec?elem=sensor
29. Gazebo Sim (Harmonic) Plugins and Sensors for ROS2 | by Ali Tekeş - Medium, accessed July 26, 2025, https://medium.com/@alitekes1/gazebo-sim-plugin-and-sensors-for-acquire-data-from-simulation-environment-681d8e2ad853
30. Sensors - Gazebo ionic documentation, accessed July 26, 2025, https://gazebosim.org/docs/latest/sensors/
31. Gazebo Harmonic Released! - Open Robotics, accessed July 26, 2025, https://www.openrobotics.org/blog/2023/9/26/gazebo-harmonic-released
32. Automatic Inertia Calculation for Links - Gazebo Sim, accessed July 26, 2025, https://gazebosim.org/api/sim/9/auto_inertia_calculation.html
33. GSoC 2023: Automatically Computing Moments of Inertia for SDFormat Links - Gazebo General - ROS Discourse, accessed July 26, 2025, https://discourse.openrobotics.org/t/gsoc-2023-automatically-computing-moments-of-inertia-for-sdformat-links/49133
34. Proposal for Automatic Moments of Inertia Calculations - Documentation - SDFormat, accessed July 26, 2025, http://sdformat.org/tutorials?tut=auto_inertial_params_proposal
35. Implementing Mimic joint plugin - Help - Gazebo Community, accessed July 26, 2025, https://community.gazebosim.org/t/implementing-mimic-joint-plugin/1362
36. docs/harmonic/migration_from_ignition.md at master / gazebosim/docs - GitHub, accessed July 26, 2025, https://github.com/gazebosim/docs/blob/master/harmonic/migration_from_ignition.md
37. Migrating ROS 2 packages that use Gazebo Classic - GitHub, accessed July 26, 2025, https://github.com/gazebosim/docs/blob/master/migrating_gazebo_classic_ros2_packages.md
38. Tutorial : Plugins 101 - Gazebo, accessed July 26, 2025, https://classic.gazebosim.org/tutorials?tut=plugins_hello_world
39. Tutorial : Intermediate: Control plugin - Gazebo Classic, accessed July 26, 2025, https://classic.gazebosim.org/tutorials?tut=guided_i5
40. Failed to load system plugin gz::sim::Systems::DiffDrive / Issue #2715 - GitHub, accessed July 26, 2025, https://github.com/gazebosim/gz-sim/issues/2715
41. ISystemConfigure Class Reference - Gazebo, accessed July 26, 2025, https://gazebosim.org/api/gazebo/6/classignition_1_1gazebo_1_1ISystemConfigure.html
42. sdf::v9::Element Class Reference - Gazebo, accessed July 26, 2025, https://gazebosim.org/api/sdformat/9/classsdf_1_1v9_1_1Element.html
43. ShaderParam Class Reference - Gazebo Sim, accessed July 26, 2025, https://gazebosim.org/api/sim/8/classgz_1_1sim_1_1systems_1_1ShaderParam.html
44. gz-sim/src/systems/diff_drive/DiffDrive.cc at gz-sim9 / gazebosim/gz ..., accessed July 26, 2025, https://github.com/gazebosim/gz-sim/blob/gz-sim9/src/systems/diff_drive/DiffDrive.cc






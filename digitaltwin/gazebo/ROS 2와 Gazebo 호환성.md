[Gazebo](./index.md)
# ROS 2와 Gazebo 호환성



로봇 운영 체제(Robot Operating System, ROS) 생태계에서 Gazebo는 오랫동안 사실상의 표준 3D 물리 시뮬레이터로 자리매김해왔다. Gazebo는 ROS를 개발하는 Open Robotics에서 관리하지만, 본질적으로 ROS에 속한 일부가 아닌 독립적인 애플리케이션이다.1 역사적으로 이 두 시스템 간의 통합은 `gazebo_ros_pkgs`라는 전용 메타패키지를 통해 이루어졌다.1

`gazebo_ros_pkgs`는 네 가지 핵심 패키지로 구성된다: `gazebo_dev`, `gazebo_msgs`, `gazebo_ros`, 그리고 `gazebo_plugins`.1 이 통합 모델의 핵심 메커니즘은 `libgazebo_ros_camera.so`와 같은 ROS 전용 플러그인(공유 라이브러리)을 Gazebo 시뮬레이션 프로세스에 직접 로드하는 방식이었다. 이 플러그인들은 Gazebo의 C++ API에 직접 접근하여 시뮬레이션 데이터를 ROS 토픽으로 발행하거나 ROS로부터 커맨드를 수신하는 역할을 수행했다.5

ROS 1과 함께 성장해 온 이 긴밀한 결합의 역사는 Gazebo를 ROS 개발자들에게 가장 친숙한 시뮬레이터로 만들었고, 방대한 양의 튜토리얼과 커뮤니티 지식이 축적되는 결과를 낳았다.2 그러나 이는 동시에 오래되거나 더 이상 유효하지 않은 정보의 범람을 야기하여, 새로운 사용자들에게 혼란을 주는 원인이 되기도 했다.2 이 시기의 중요한 특징은 `gazebo_ros_pkgs`가 특정 Gazebo 메이저 버전에 강하게 종속되었다는 점이다. 예를 들어, ROS Kinetic은 Gazebo 7, ROS Melodic과 Crystal은 Gazebo 9, 그리고 ROS Noetic과 Foxy는 Gazebo 11과 연동되었다.1


15년 이상 개발이 지속되면서 Gazebo Classic의 단일체(monolithic) 아키텍처는 유지보수와 확장이 점점 더 어려워졌다.9 이에 따라 Gazebo는 전면적인 재설계를 거치게 되었다. 새로운 아키텍처의 목표는 `gz-math`, `gz-transport`, `gz-rendering`과 같이 독립적으로 개발하고 사용할 수 있는 느슨하게 결합된 라이브러리 모음으로 전환하는 것이었다.10

개발팀은 점진적인 "Gazebo 12"로의 업데이트 대신 "Ignition"이라는 새로운 이름으로 코드베이스를 완전히 새로 작성하는 길을 택했다. 이는 기존 코드베이스와의 API/ABI 호환성을 유지해야 하는 엄청난 부담에서 벗어나, 엔티티-컴포넌트-시스템(Entity-Component-System, ECS) 모델과 같은 근본적인 아키텍처 개선을 단행하기 위한 전략적 결정이었다.10

이후 커뮤니티의 혼란을 줄이고 브랜드를 통합하기 위해 "Ignition Gazebo"는 다시 "Gazebo"로 리브랜딩되었으며, 새로운 버전들은 Fortress, Harmonic과 같은 알파벳 코드명을 사용하게 되었다.9 이로써 기존 시뮬레이터는 공식적으로 "Gazebo Classic"으로 명명되었다.9

이러한 Gazebo의 아키텍처 재설계는 단순히 기술적 업그레이드를 넘어, 시뮬레이터를 ROS로부터 철학적으로 분리하는 중요한 과정이었다. 모듈식 라이브러리(`gz-math`, `gz-transport` 등)로의 전환은 시뮬레이션 컴포넌트들을 Gazebo라는 단일 애플리케이션 외부에서도 재사용 가능하게 만들려는 전략적 결정이었다. 실제로 RViz2와 같은 다른 ROS 도구들이 현재 `gz-math`를 사용하고 있다는 사실이 이를 증명한다.11 이 분리 과정은 ROS 통합 모델의 변화를 필연적으로 요구했다. 시뮬레이터 내부에 ROS 특정 코드(플러그인)를 주입하는 방식에서 벗어나, 보다 표준화된 메시지 전달 인터페이스(브리지)로 전환하게 된 것이다. 이는 소프트웨어 공학적으로 더 견고하고 유지보수하기 용이한 패턴이다. 즉, 통합 아키텍처의 변화(`플러그인` -->> `브리지`)는 Gazebo 자체 아키텍처의 변화(`단일체` -->> `라이브러리`)에서 비롯된 직접적이고 필연적인 결과였다. 이는 새로운 Gazebo가 기술적으로는 독립적이었던 Gazebo Classic보다 실질적으로 더욱 "ROS에 구애받지 않는(agnostic)" 구조가 되었음을 의미하고, 통합이 ROS와 같은 특정 미들웨어가 아닌 전송 계층에서 이루어지므로 이론적으로 다른 프레임워크와의 통합도 더 용이해졌다.2


전환기 동안 ROS 2 생태계에서는 Gazebo Classic과 현대적 Gazebo가 공존하며 지원되었다. ROS 2 Humble과 같은 배포판은 Gazebo Classic 11과 Gazebo Fortress를 모두 지원했다.16

그러나 이 두 스택의 경로는 ROS 2 Jazzy Jalisco에서 결정적으로 분기되었다. Jazzy는 Gazebo Classic에 대한 지원을 중단한 최초의 장기 지원(Long Term Support, LTS) 릴리스다.17 이러한 결정의 주된 이유는 Gazebo Classic이 의존하는 라이브러리들이 Jazzy의 대상 플랫폼인 Ubuntu 24.04 (Noble)에서는 더 이상 제공되지 않기 때문이다.17

Gazebo Classic의 공식 지원 종료(End-Of-Life, EOL)는 2025년 1월, ROS 1 Noetic의 EOL은 2025년 5월로 예정되어 있으며, 이는 레거시 스택의 시대가 공식적으로 막을 내림을 의미한다.3 EOL 이후에는 새로운 기능 추가, 보안 업데이트, 버그 수정이 중단되므로, 사용자들에게는 새로운 스택으로의 마이그레이션이 강력히 요구된다.20



안정적이고 신뢰할 수 있는 사용자 경험을 보장하기 위해, 각 ROS 2 배포판은 특정 버전의 Gazebo와 함께 테스트되고 공식적으로 지원된다.21 이러한 "공식 페어링"은 개발 환경 설정의 복잡성을 크게 줄여준다.

주요 공식 페어링은 다음과 같다:

- **ROS 2 Humble Hawksbill (LTS):** Gazebo Fortress와 공식적으로 페어링된다.21
- **ROS 2 Iron Irwini:** Gazebo Classic 11에 대한 공식 바이너리 지원을 제공한 마지막 릴리스였으며, Gazebo Fortress도 함께 지원했다.17
- **ROS 2 Jazzy Jalisco (LTS):** Gazebo Harmonic과 공식적으로 페어링된다.21
- **ROS 2 Rolling Ridley (및 차기 K-turtle):** 가장 최신 Gazebo 릴리스(현재 Ionic)와 페어링된다.21


ROS 2 Jazzy부터 도입된 "벤더 패키지(vendor package)"는 ROS와 Gazebo의 통합 방식을 근본적으로 바꾼 중요한 변화다.24 ROS 벤더 패키지는 ROS가 의존하는 소프트웨어를 제공하는 패키지로, 특히 시스템의 기본 버전이 부적합하거나 사용 불가능할 때 유용하다.11

이는 과거의 고질적인 문제를 해결한다. 이전에는 Ubuntu의 업스트림 저장소에 포함된 Gazebo 패키지가 업데이트되지 않아 ROS 사용자들이 구버전을 사용해야 하는 경우가 많았다.26 벤더 패키지는 이 문제를 해결하기 위해, ROS 빌드팜 프로세스의 일부로서 공식적으로 페어링된 Gazebo 버전(예: Jazzy를 위한 Harmonic)의 소스 코드를 직접 가져와 빌드한다.24

이 새로운 모델은 각 ROS 릴리스에 대해 일대일로 완벽하게 테스트된 페어링을 보장하고, 대다수 사용자의 설치 과정을 극적으로 단순화하고 초기 사용 경험을 개선한다.11 벤더 패키지의 도입은 ROS-Gazebo 통합에 대한 거버넌스 및 유지보수 책임의 근본적인 전환을 의미한다. Jazzy 이전에는 Gazebo가 Boost나 Ogre와 같은 다른 외부 의존성 라이브러리처럼 취급되었고, ROS는 대상 운영체제의 저장소에서 제공되는 버전에 단순히 의존했다.21 그러나 벤더 패키지를 통해 ROS 빌드팜은 이제 특정 버전의 Gazebo 소스 코드를 능동적으로 가져와 컴파일한다.24 이는 ROS 유지보수팀이 ROS 생태계를 위한 Gazebo 패키징 및 배포 프로세스에 대한 직접적인 소유권을 갖게 되었음을 의미한다. 더 이상 업스트림 의존성을 수동적으로 소비하는 것이 아니라, 적극적으로 큐레이팅하고 배포하는 것이다. 이는 사용성 측면에서 큰 이점을 제공하지만, 두 프로젝트의 릴리스 주기를 더욱 긴밀하게 만들고 ROS 빌드팜 팀의 유지보수 부담을 증가시키는 양면성을 가진다.


벤더 패키지가 표준 경로를 제시하지만, 프레임워크는 여전히 다른 ROS/Gazebo 조합을 필요로 하는 전문가 사용 사례를 허용한다.21

- **방법 1: OSRF 바이너리 사용:** ROS 2 Humble과 Gazebo Harmonic 같은 조합을 사용하려면, 기본 ROS 저장소를 우회해야 한다. 이를 위해 `packages.osrfoundation.org` 저장소를 시스템에 추가하고, 원하는 Gazebo 버전(예: `gz-harmonic`)과 `ros-humble-ros-gzharmonic`과 같은 특정 호환성 패키지를 설치해야 한다.21 이 패키지들은 기본 `ros-humble-ros-gz` 패키지와 충돌하므로 기존 패키지를 먼저 제거해야 한다.21
- **방법 2: 소스에서 빌드:** 최신 Gazebo 버전을 사용하거나 Gazebo 라이브러리 자체를 개발하는 등 최고의 유연성이 필요한 경우, `ros_gz`와 관련 의존성을 colcon 작업 공간에서 소스로부터 직접 빌드해야 한다.21 이 과정에서는 다음과 같은 중요한 환경 변수 설정이 필요하다:
  - `export GZ_RELAX_VERSION_MATCH=1`: 벤더 패키지가 시스템에 설치된 Gazebo 라이브러리의 정확한 버전 대신 메이저 버전만 일치하도록 허용한다.26
  - `export GZ_BUILD_FROM_SOURCE=1`: 벤더 패키지가 동일한 작업 공간 내에 소스 코드로 존재하는 Gazebo 라이브러리를 대상으로 빌드되도록 강제하여, 올바른 빌드 순서와 의존성을 보장한다.26


초보자에게는 항상 최신 LTS 버전의 ROS 2와 Gazebo를 Tier 1 지원 운영체제(예: Humble을 위한 Ubuntu 22.04, Jazzy를 위한 Ubuntu 24.04)에서 함께 사용하는 것이 권장된다.21 오래된 튜토리얼을 따르는 사용자들이 흔히 겪는 문제, 예를 들어 Ubuntu 22.04에서 Gazebo Classic 11을 표준 저장소에서 설치할 수 없는 문제 등은 이러한 권장 사항을 따름으로써 피할 수 있다.16

다음 표는 ROS 2와 Gazebo 버전 간의 호환성 및 권장 설치 방법을 요약한 것이다.

| ROS 2 배포판     | 대상 Ubuntu | Gazebo 버전 | 지원 상태     | 설치 방법                                   | 주요 참고사항                      |
| ---------------- | ----------- | ----------- | ------------- | ------------------------------------------- | ---------------------------------- |
| **Humble (LTS)** | 22.04       | Fortress    | **공식 지원** | `sudo apt install ros-humble-ros-gz`        | 가장 안정적인 조합.                |
| Humble (LTS)     | 22.04       | Harmonic    | 고급 사용자   | OSRF 저장소 + `ros-humble-ros-gzharmonic`   | 기본 `ros-gz` 패키지와 충돌. 21    |
| Humble (LTS)     | 22.04       | Classic 11  | 비권장        | 소스 빌드 또는 `libgazebo-dev`              | 공식 지원 중단. 16                 |
| **Iron**         | 22.04       | Fortress    | **공식 지원** | `sudo apt install ros-iron-ros-gz`          | Classic 11을 지원한 마지막 릴리스. |
| Iron             | 22.04       | Classic 11  | 레거시 지원   | `sudo apt install ros-iron-gazebo-ros-pkgs` | EOL 임박. 17                       |
| **Jazzy (LTS)**  | 24.04       | Harmonic    | **공식 지원** | `sudo apt install ros-jazzy-ros-gz`         | 벤더 패키지 모델 도입. 24          |
| Jazzy (LTS)      | 24.04       | Classic 11  | **미지원**    | -                                           | Ubuntu 24.04에서 빌드 불가. 18     |
| **Rolling**      | 24.04       | Ionic       | **공식 지원** | `sudo apt install ros-rolling-ros-gz`       | 최신 기능, 벤더 패키지 사용. 21    |



`gazebo_ros_pkgs` 아키텍처의 핵심은 공유 라이브러리 형태의 플러그인을 Gazebo 서버 프로세스에 직접 로드하는 것이었다.5 이 플러그인들은 Gazebo의 C++ API와 전송 시스템에 직접 접근할 수 있었고, 동시에 ROS 노드를 초기화하여 ROS 네트워크와 통신했다.1 특히 `gazebo_ros_api_plugin`은 Gazebo 내에서 ROS를 초기화하기 위해 필수적인 대형 시스템 플러그인이었고, 다양한 기능을 단일 번들로 제공하여 사용자가 특정 기능을 선택적으로 사용하기 어려운 구조였다.1


현대적 Gazebo는 자체 내부 미들웨어인 Gazebo Transport(구 Ignition Transport)를 사용하여 컴포넌트 간 통신을 수행한다.5 ROS 2와의 통합은 더 이상 ROS를 인지하는 플러그인을 시뮬레이터에 로드하는 방식으로 이루어지지 않는다. 대신, `ros_gz_bridge`라는 독립적인 ROS 2 노드가 별도의 프로세스로 실행된다.5

이 브리지 노드는 번역기 역할을 한다. Gazebo Transport 토픽을 구독하고, 메시지 형식을 변환한 후, 해당 ROS 2 토픽으로 발행한다(또는 그 반대).5

`parameter_bridge` 실행 구문은 `ros2 run ros_gz_bridge parameter_bridge /gz_topic@ros_msg_type@gz_msg_type`와 같다.33 이 아키텍처는 시뮬레이터를 ROS 미들웨어로부터 완전히 분리시켜, Gazebo가 ROS의 존재를 전혀 인지하지 않고도 실행될 수 있게 한다. 이는 더 모듈화되고 견고한 설계다.10 고대역폭 데이터의 효율적인 전송을 위해 `ros_gz_image`와 같은 특화된 브리지도 존재한다.5


브리지 아키텍처의 주요 성능 고려 사항은 프로세스 간 통신(IPC) 및 메시지 직렬화/역직렬화에 따른 오버헤드다. 이를 완화하기 위해 `ros_gz` 스택은 ROS 2의 컴포저블 노드(composable nodes) 기능을 적극 활용한다.32 런치 파일에서 `use_composition:=True` 파라미터를 설정하면, 사용자는 Gazebo 서버 노드와 `ros_gz_bridge` 노드를 ROS 컨테이너 내의 동일한 프로세스에 로드할 수 있다.32

이는 프로세스 내 통신(intra-process communication)을 가능하게 하여 직렬화 및 전송 오버헤드를 제거하고, 이미지나 레이저 스캔과 같은 고주파 토픽의 성능을 크게 향상시킨다.11 한 예시 그래프는 컴포지션을 사용했을 때 1000개의 이미지를 수신하는 데 필요한 시간이 극적으로 감소함을 보여준다.35


로봇 제어 통합은 `gz_ros2_control` 패키지를 통해 이루어진다. 이 패키지는 `ros2_control` 프레임워크와 Gazebo 사이의 다리 역할을 한다.28 로봇의 URDF 파일을 통해 로드되는 Gazebo 시스템 플러그인을 제공하고, 이 플러그인은 `ros2_control` 컨트롤러 매니저를 인스턴스화하고 Gazebo의 시뮬레이션된 조인트 상태 및 커맨드 인터페이스를 `JointStateInterface`, `EffortJointInterface`와 같은 `ros2_control` 하드웨어 인터페이스에 연결한다.28 이를 통해 `joint_trajectory_controller`나 `diff_drive_controller`와 같은 표준 `ros2_control` 컨트롤러가 실제 로봇을 제어하는 것과 동일한 방식으로 시뮬레이션된 로봇을 제어할 수 있다.28

브리지 기반 시스템으로의 아키텍처 전환은 복잡성을 외부화하지만, 메시지 변환 및 전송 구성과 관련된 새로운 유형의 실패 모드와 디버깅 과제를 야기한다. 과거의 플러그인 시스템은 단일 코드 단위였지만, 새로운 시스템은 Gazebo 센서, Gazebo Transport 토픽, 그리고 `ros_gz_bridge` 구성이라는 세 가지 개별 부분으로 이루어져 있다.1 따라서 디버깅은 다단계 프로세스가 되었다. 사용자는 1) Gazebo가 내부 토픽으로 데이터를 올바르게 발행하는지(`gz topic -l`), 2) 브리지가 정확한 토픽 이름과 메시지 유형으로 구성되었는지, 3) 브리지가 실행되어 메시지를 올바르게 변환하는지, 4) ROS 2 노드가 최종 토픽을 올바르게 구독하는지를 순차적으로 확인해야 한다. 이는 아키텍처적으로는 더 깔끔하지만, 설정 및 디버깅 시 개발자의 인지 부하를 증가시킨다. 최근 릴리스에서 개선된 런치 파일이 도입된 것은 바로 이러한 "구성의 복잡성"을 완화하기 위한 직접적인 시도다.27



모든 마이그레이션의 첫 단계는 패키지 의존성을 업데이트하는 것이다. `package.xml` 파일에서 `gazebo_ros_pkgs` 또는 `gazebo_ros`에 대한 의존성은 `ros_gz` 메타패키지에 포함된 `ros_gz_sim`, `ros_gz_bridge`, `ros_gz_image`와 같은 더 세분화된 패키지로 대체되어야 한다.5 마찬가지로, `CMakeLists.txt`에서는 `find_package(gazebo_ros REQUIRED)`가 `find_package(ros_gz_sim REQUIRED)` 등으로 변경되어야 한다.36


런치 파일은 상당한 변경이 필요하다. `gazebo_ros`의 런치 파일(예: `empty_world.launch.py`)은 `ros_gz_sim`에서 제공하는 동등한 파일로 대체된다.5 새로운 `ros_gz_sim.launch.py`는 `gz_args` 인수를 통해 Gazebo 실행 파일에 직접 명령줄 인자(예: `-r` 즉시 실행, `-s` 서버만 실행)를 전달할 수 있다.5

로봇 모델을 스폰하는 방식도 `gazebo_ros`의 `spawn_entity.py` 스크립트에서 `ros_gz_sim`의 `create` 실행 파일이나 런치 액션으로 변경된다.36 가장 중요한 변화는, 이제 런치 파일이 `ros_gz_bridge` 노드를 명시적으로 실행하고, 브리징할 토픽 목록이 정의된 설정 파일을 제공해야 한다는 점이다.5


URDF/SDF 파일 내의 플러그인 정의는 핵심적인 수정 대상이다. `<plugin name="gazebo_ros_diff_drive" filename="libgazebo_ros_diff_drive.so">`와 같은 클래식 ROS 플러그인은 `<plugin name="gz::sim::systems::DiffDrive" filename="gz-sim-diff-drive-system">`과 같은 현대적 Gazebo Sim의 "시스템" 플러그인으로 대체되어야 한다.36 이때 네임스페이스는 `gz::sim::systems`로 변경된다.36

예를 들어, LiDAR 센서의 경우 `libgazebo_ros_ray_sensor.so`가 네이티브 시스템으로 대체되고, 성능 향상을 위해 센서 유형이 `ray`에서 `gpu_lidar`로 변경되는 경우가 많다.36 월드 파일(`.world` 또는 `.sdf`) 또한 `physics`, `scene_broadcaster`와 같은 필수 기본 시스템과 원하는 렌더링 엔진을 포함하도록 업데이트되어야 한다.5


차동 구동 로봇의 마이그레이션 사례를 통해 전체 과정을 살펴본다.

- **클래식 접근 방식:** `gazebo_ros_diff_drive` 플러그인이 포함된 URDF를 사용한다. 이 플러그인은 내부적으로 `/cmd_vel` ROS 토픽을 구독하고 `/odom` 토픽을 발행했다.

- **현대적 접근 방식:**

  1. URDF는 `gz::sim::systems::DiffDrive` 시스템 플러그인을 사용하도록 업데이트된다. 이 시스템은 Gazebo Transport 토픽(예: `/model/my_robot/cmd_vel`)을 구독하고, 다른 Gazebo Transport 토픽(예: `/model/my_robot/odom`)으로 주행 거리계를 발행한다.

  2. ROS 토픽과 Gazebo 토픽을 매핑하기 위한 `bridge_config.yaml` 파일을 생성한다.5

     YAML

     ```
     - ros_topic_name: "/cmd_vel"
       gz_topic_name: "/model/my_robot/cmd_vel"
       ros_type_name: "geometry_msgs/msg/Twist"
       gz_type_name: "gz.msgs.Twist"
       direction: ROS_TO_GZ
     - ros_topic_name: "/odom"
       gz_topic_name: "/model/my_robot/odom"
       ros_type_name: "nav_msgs/msg/Odometry"
       gz_type_name: "gz.msgs.Odometry"
       direction: GZ_TO_ROS
     ```

  3. 메인 런치 파일은 Gazebo, 로봇 스폰 노드, 그리고 위에서 정의한 YAML 설정 파일을 사용하는 `ros_gz_bridge` 노드를 실행하도록 수정된다.5

이 마이그레이션 과정은 개발자의 책임이 "암묵적인" 관례 기반 설정에서 "명시적인" 구성 기반 설정으로 전환되었음을 보여준다. Gazebo Classic에서는 `gazebo_ros_diff_drive` 플러그인을 추가하는 것만으로도 내부 관례에 따라 `/cmd_vel`과 `/odom` 같은 ROS 토픽이 암묵적으로 생성되었다.1 반면, Gazebo Sim에서는 개발자가 1) Gazebo 네이티브 시스템 플러그인을 추가하고, 2) 별도의 YAML 파일에 ROS 토픽과 Gazebo 토픽 간의 매핑을 명시적으로 정의하며, 3) 이 구성을 사용하여 브리지 노드를 실행하는 세 가지 명시적인 단계를 거쳐야 한다.5 이러한 변화는 초기 설정의 복잡성을 증가시키지만, 두 통신 계층(ROS DDS와 Gazebo Transport)과 그 사이의 변환 계층을 개발자가 명확하게 인지하게 만든다. 이는 고급 사용자에게 훨씬 더 큰 유연성과 명확성을 제공하고, 네임스페이스 관리가 중요한 복잡한 다중 로봇 시나리오에서 시스템을 더 쉽게 사용자 정의할 수 있게 해준다.



고주파 센서 데이터, 특히 `gpu_lidar` 센서의 데이터를 브리징할 때 심각한 성능 저하가 발생하는 문제가 문서화되어 있다.37 이 문제는 ROS 2 토픽의 주파수가 센서에 설정된 `update_rate`보다 현저히 낮게 나타나고(예: 목표 속도의 60%), 동시에 Gazebo의 실시간 계수(Real-Time Factor, RTF)가 급격히 하락하는 현상으로 나타난다.37

문제의 원인은 브리지 자체에 국한된다. 센서에 대한 `ros_gz_bridge` 노드를 비활성화하면 RTF가 즉시 정상 수준으로 회복되기 때문이다.37 이는 특정 데이터 유형에 대한 브리지 내부의 메시지 처리 또는 변환 과정에 병목이 있음을 시사한다. 이 문제는 분리된 아키텍처가 고대역폭 데이터 스트림에 대해 가질 수 있는 성능 비용을 명확히 보여주고, 3.3절에서 논의된 `use_composition` 최적화의 중요성을 강조한다.


`ros2_control`을 Gazebo와 함께 사용할 때 센서 데이터와 주행 거리계 변환 정보 간에 시간 동기화 문제가 발생하는 미묘하지만 치명적인 버그가 보고되었다.38

- **증상:** 로봇이 회전할 때 RViz에서 LaserScan과 같은 센서 데이터가 로봇의 회전을 "앞서가는" 것처럼 보인다. 이는 센서 데이터의 타임스탬프와 `ros2_control` 스택이 발행하는 TF 변환 정보의 타임스탬프가 일치하지 않음을 나타낸다.38 이 문제는 클래식 Gazebo 플러그인을 사용할 때는 발생하지 않아, `ros2_control` 통합 경로에 문제가 있음을 시사한다.38

- **의심되는 원인:** 문제는 바퀴 미끄러짐과 같은 물리적 현상이 아닌 타이밍 또는 동기화 문제로 추정된다.38 논의는 

  `diff_drive_controller` 자체의 문제, 특히 가속도 제한을 처리하거나 주행 거리계를 계산할 때 타임스탬프를 찍는 방식과 관련이 있을 가능성을 제기한다.38

핵심 과제는 시뮬레이션 시간(`/clock`), 센서 데이터 타임스탬프, 그리고 `ros2_control` 업데이트 루프가 모두 완벽하게 동기화되도록 보장하는 것이다. 여기서 `use_sim_time` 파라미터가 매우 중요하다. ROS 2에서는 이 파라미터를 노드별로 설정해야 하므로, 런치 파일에서 신중하게 관리하지 않으면 시스템 전체에 걸쳐 일관되지 않은 시간 소스가 발생할 수 있다.40

`No clock received, using time argument instead!`와 같은 경고는 이러한 설정 오류의 명백한 신호다.44


특정 버전 조합에서 특정 센서가 부정확한 데이터를 제공하는 사례도 보고되었다. 대표적인 예로 ROS 2 Humble과 함께 사용된 Gazebo Classic 11의 힘/토크(Force/Torque) 센서가 있다. 이 센서는 "매우 부정확한" 데이터를 생성하고, 측정값이 틀릴 뿐만 아니라 로봇의 스폰 높이에 따라 달라지는 문제를 보였다.45 동일한 설정이 ROS 1이나 ROS 2의 Gazebo Fortress에서는 올바르게 작동하므로, 이는 `libgazebo_ros_ft_sensor.so` 플러그인과 ROS 2 Humble 환경 간의 특정 상호작용 버그임을 시사한다.45 이 외에도 `gazebo_ros_pkgs` 저장소에는 IMU 센서의 스파이크 발생, 레이 센서의 미발행, 차동 구동 플러그인의 부정확한 주행 거리계 데이터 등 여러 문제가 보고되어 레거시 스택의 지속적인 유지보수 어려움을 보여준다.46


기술적인 문제 외에도, 문서와 커뮤니티 지식의 상태는 중요한 도전 과제다. 오랜 역사와 여러 차례의 주요 아키텍처 변경(Classic -->> Ignition -->> Gazebo Sim)으로 인해, 온라인에 존재하는 수많은 튜토리얼, 포럼 답변, 문서들이 현재는 시대에 뒤떨어져 있다.2 이러한 "문서 부채"는 새로운 사용자들이 올바른 정보를 찾는 것을 어렵게 만들고, 결국 잘못되거나 폐기된 방법에 의존하게 만든다.2 혼란스러운 이름 변경의 역사는 관련 정보 검색을 더욱 어렵게 하여 이 문제를 악화시킨다.47

이러한 문서 파편화는 ROS 생태계 전반에서 나타나는 문제이지만, 두 프로젝트가 병렬적으로, 때로는 혼란스럽게 진화해 온 Gazebo 통합의 경우 특히 두드러진다.2

이러한 문제들은 단순히 개별적인 버그가 아니라, 서로 다른 두 아키텍처 패러다임(분리된 브리지 대 단일체 제어 루프)의 본질적인 복잡성에서 비롯된 증상들이다. `gpu_lidar` 문제는 메시지 전달 파이프라인의 처리량 문제이고, `ros2_control` 버그는 분산 시스템의 타이밍 문제다. 두 문제 모두 긴밀하게 결합된 단일체 아키텍처에서는 발생할 가능성이 훨씬 낮다. 이는 새로운 모듈식 아키텍처의 유연성이 성능 튜닝과 시간 동기화의 복잡성 증가라는 대가를 치른다는 것을 의미한다.



Gazebo의 공식 로드맵은 시뮬레이터의 미래 방향을 예측하는 데 중요한 단서를 제공한다.48

- **기술 업그레이드:** GUI 툴킷을 현대화하기 위한 Qt5에서 Qt6로의 마이그레이션과 빌드 성능 및 재현성 향상을 위한 Bazel 빌드 시스템 채택이 계획되어 있다.48
- **통신 계층 혁신:** `gz-transport`에 Zenoh를 통합하려는 계획은 매우 중요하다. Zenoh는 로보틱스와 IoT를 위해 설계된 현대적인 통신 프로토콜로, 이를 채택하면 Gazebo의 핵심 통신 계층에서 상당한 성능 및 확장성 향상을 기대할 수 있다.48
- **고급 애플리케이션 지원:** 강화학습(Reinforcement Learning, RL) 인터페이스 개선에 중점을 두고 있다. 이는 커뮤니티의 요구와 Isaac Sim과 같은 경쟁 시뮬레이터에 대응하기 위한 직접적인 조치다.2
- **생태계 성장:** "연합된 서드파티 플러그인 생태계" 구축 계획은 커뮤니티 기여를 위한 보다 접근성 높고 분산된 모델로의 전환을 시사한다.48


ROS 2 기술 운영 위원회(Technical Steering Committee, TSC)는 프로젝트의 기술적 방향, 로드맵, 릴리스 일정을 책임진다.49 최근 릴리스(Iron, Humble 등)의 로드맵은 DDS 사용자 경험 개선, 성능 향상(`rclcpp` 실행기, 대용량 메시지 처리 등), 그리고 `rosbag2`, `launch`와 같은 도구 개선에 초점을 맞추어 왔다.52

ROS 2를 "제품 수준으로 준비"시키고 ROS 1로부터의 마이그레이션을 용이하게 하려는 명시적인 목표는, 벤더 패키지를 통해 Gazebo 통합을 강화하는 전략과 완벽하게 일치한다. 안정적이고 사용하기 쉬운 시뮬레이션 환경은 제품 수준의 로보틱스 프레임워크의 초석이기 때문이다. Microsoft, Bosch, Amazon 등 주요 산업체들의 자원 투입에 기반한 TSC의 의사결정 구조는, 향후 통합 노력이 이들 기업의 요구에 의해 주도될 가능성이 높음을 시사한다.49


- **신규 프로젝트:** 현재 최신 ROS 2 LTS 릴리스(Jazzy Jalisco)와 공식적으로 페어링된 Gazebo 버전(Harmonic)을 권장 Ubuntu 버전(24.04)에서 새로운 벤더 패키지를 통해 설치하는 것이 가장 확실한 선택이다.18 이 경로는 가장 긴 지원 기간과 가장 안정적이고 간소화된 경험을 제공한다.

- **레거시 프로젝트:** Gazebo Classic 기반 프로젝트의 경우, 마이그레이션은 더 이상 선택이 아닌 필수다. EOL이 임박함에 따라 18, 팀은 현대적 Gazebo로의 전환을 위한 자원을 할당해야 한다. 

  `ros1_bridge`를 사용하여 컴포넌트를 하나씩 점진적으로 마이그레이션하는 접근 방식이 권장된다.55

- **모범 사례:**

  - 모든 노드에 대해 모든 런치 파일에서 `use_sim_time` 파라미터를 명시적으로 관리하여 일관된 시간 동기화를 보장해야 한다.40
  - 고대역폭 토픽의 `ros_gz_bridge` 성능을 극대화하기 위해 가능한 모든 곳에서 `use_composition`을 활용해야 한다.32
  - 마이그레이션 시, 암묵적인 플러그인 동작에 의존하기보다 모든 토픽 브리지를 YAML 파일에 명시적으로 정의하여 명확성과 유지보수성을 확보하는 것이 좋다.5
  - 제어에는 `gz_ros2_control`을 사용하는 것이 ROS 2 제어 생태계와의 원활한 통합을 위한 표준적이고 권장되는 접근 방식이다.28

ROS 2와 Gazebo의 병렬적인 로드맵은 더 깊고 성능 지향적인 통합의 미래를 예고한다. 특히, 양쪽 프로젝트 모두에서 차세대 통신 기술로 Zenoh를 검토하고 있다는 점은 주목할 만하다.48 만약 ROS 2와 Gazebo가 Zenoh를 네이티브 전송 계층으로 채택한다면, 서로 다른 미들웨어 프로토콜 간의 변환을 담당하는 현재의 `ros_gz_bridge`는 불필요해질 수 있다. 이는 ROS-Gazebo 통합의 궁극적인 형태로, 성능 병목 현상을 해결하고 아키텍처를 극적으로 단순화할 수 있는 잠재력을 가진다. 현재의 브리지 아키텍처와 `use_composition` 최적화는 진정으로 원활한 상호 운용성을 향한 과도기적 단계일 수 있고, 개발자들은 이러한 통합 방식이 미래에 근본적으로 다시 한번 진화할 가능성이 있음을 인지해야 한다.


1. gazebo_tutorials/ros2_overview/tutorial.md at master - GitHub, accessed July 22, 2025, https://github.com/osrf/gazebo_tutorials/blob/master/ros2_overview/tutorial.md
2. Why is Gazebo very famous in the ROS community? what about Webots?, accessed July 22, 2025, https://discourse.ros.org/t/why-is-gazebo-very-famous-in-the-ros-community-what-about-webots/42459
3. gazebo_ros_pkgs - ROS Wiki, accessed July 22, 2025, http://wiki.ros.org/gazebo_ros_pkgs
4. Tutorial : ROS 2 overview - Gazebo Classic, accessed July 22, 2025, https://classic.gazebosim.org/tutorials?tut=ros2_overview
5. Migrating ROS 2 packages that use Gazebo Classic, accessed July 22, 2025, https://gazebosim.org/docs/latest/migrating_gazebo_classic_ros2_packages/
6. Tutorial: ROS integration overview - Gazebo, accessed July 22, 2025, https://classic.gazebosim.org/tutorials?tut=ros_overview
7. 2. Gazebo - ROS Notes 0.9.0 documentation, accessed July 22, 2025, https://ros-notes.readthedocs.io/en/latest/chapters/ROS/15_mobile_robot/gazebo.html
8. Tutorial : Installing gazebo_ros_pkgs (ROS 2) - Gazebo Classic, accessed July 22, 2025, https://classic.gazebosim.org/tutorials?tut=ros2_installing
9. Gazebo Classic Migration, accessed July 22, 2025, https://gazebosim.org/docs/latest/gazebo_classic_migration/
10. Ignition Gazebo motivation, accessed July 22, 2025, https://community.gazebosim.org/t/ignition-gazebo-motivation/1240
11. State of Gazebo 2024 - ROSCon 2025, accessed July 22, 2025, https://roscon.ros.org/2024/talks/The_State_of_Gazebo.pdf
12. Gazebo Sim: Migration from Gazebo Classic: Plugins, accessed July 22, 2025, https://gazebosim.org/api/sim/9/migrationplugins.html
13. Migration Guide - Gazebo ionic documentation, accessed July 22, 2025, https://gazebosim.org/docs/latest/migration_from_ignition/
14. Move/rename osrf/gazebo repository to gazebosim/gazebo-classic - Open Robotics Discourse, accessed July 22, 2025, https://discourse.openrobotics.org/t/move-rename-osrf-gazebo-repository-to-gazebosim-gazebo-classic/48180
15. A new era for Gazebo - Open Robotics Discourse, accessed July 22, 2025, https://discourse.openrobotics.org/t/a-new-era-for-gazebo/48995
16. Gazebo Classic (gazebo11) can't be installed on Ubuntu 22.04 Jammy with ROS2 Humble. / Issue #3357 - GitHub, accessed July 22, 2025, https://github.com/gazebosim/gazebo-classic/issues/3357
17. Gazebo Classic End-Of-Life & ROS 2 Jazzy - General, accessed July 22, 2025, https://discourse.ros.org/t/gazebo-classic-end-of-life-ros-2-jazzy/36239
18. Gazebo Classic End-of-Life and Migration Survey - Open Robotics Discourse - ROS, accessed July 22, 2025, https://discourse.ros.org/t/gazebo-classic-end-of-life-and-migration-survey/35690
19. Gazebo Classic to Gazebo Sim: Essential Upgrade Before Jan 2025! - YouTube, accessed July 22, 2025, https://www.youtube.com/watch?v=KVe2u_2igkc&pp=0gcJCfwAo7VqN5tD
20. Gazebo Classic and Citadel End of Life - General, accessed July 22, 2025, https://community.gazebosim.org/t/gazebo-classic-and-citadel-end-of-life/3242
21. Installing Gazebo with ROS - Gazebo ionic documentation, accessed July 22, 2025, https://gazebosim.org/docs/latest/ros_installation/
22. For ROS2 Beginners - Install ROS 2 and Gazebo on Ubuntu 22.04 Jammy (LTS) Step-by-Step Instruction | by Claudia Yao | Medium, accessed July 22, 2025, https://medium.com/@claudia.yao2012/for-ros2-beginners-install-ros-2-and-gazebo-on-ubuntu-22-04-jammy-lts-step-by-step-instruction-08327c3764a5
23. Getting Started with Gazebo? - Gazebo ionic documentation, accessed July 22, 2025, https://gazebosim.org/docs/latest/getstarted/
24. ROS 2 Jazzy Jalisco Released - Open Robotics, accessed July 22, 2025, https://www.openrobotics.org/blog/2024/5/ros-jazzy-jalisco-released
25. Compatibility of Gazebo and TurtleBot3 with Ubuntu 24.04 and ROS 2 Jazzy, accessed July 22, 2025, https://community.gazebosim.org/t/compatibility-of-gazebo-and-turtlebot3-with-ubuntu-24-04-and-ros-2-jazzy/3533
26. ROS 2 Gazebo Vendor Packages - Gazebo ionic documentation, accessed July 22, 2025, https://gazebosim.org/docs/latest/ros2_gz_vendor_pkgs/
27. ROS 2 Jazzy Jalisco Released! - Open Robotics Discourse, accessed July 22, 2025, https://discourse.ros.org/t/ros-2-jazzy-jalisco-released/37862
28. gz_ros2_control - ROS2_Control: Humble Jul 2025 documentation, accessed July 22, 2025, https://control.ros.org/humble/doc/gz_ros2_control/doc/index.html
29. gazebosim.org, accessed July 22, 2025, [https://gazebosim.org/docs/latest/ros_installation/#:~:text=Gazebo%20Harmonic%20can%20be%20used,Humble%20officially%20supports%20Gazebo%20Fortress).](https://gazebosim.org/docs/latest/ros_installation/#:~:text=Gazebo Harmonic can be used,Humble officially supports Gazebo Fortress).)
30. ros_gz/README.md at ros2 / gazebosim/ros_gz / GitHub, accessed July 22, 2025, https://github.com/gazebosim/ros_gz/blob/ros2/README.md
31. 10 Problems Every Beginner Roboticist Faces (And How to Solve Them) - RoboSmiths, accessed July 22, 2025, https://www.robosmiths.com/blog/ten-problems-beginner-roboticists
32. ROS 2 integration overview - Gazebo ionic documentation, accessed July 22, 2025, https://gazebosim.org/docs/latest/ros2_overview/
33. Use ROS 2 to interact with Gazebo, accessed July 22, 2025, https://gazebosim.org/docs/latest/ros2_integration/
34. robotic_notes/docs/ros2_gazebo_integration.md at master - GitHub, accessed July 22, 2025, https://github.com/behnamasadi/robotic_notes/blob/master/docs/ros2_gazebo_integration.md
35. Ros_gz bridge improvements - General - Gazebo Community, accessed July 22, 2025, https://community.gazebosim.org/t/ros-gz-bridge-improvements/3135
36. Migrating from Gazebo Classic to Gazebo Sim: A Practical Guide - Ibrahim Bin Mansur, accessed July 22, 2025, https://ibrahimmansur4.medium.com/migrating-from-gazebo-classic-to-gazebo-sim-a-practical-guide-804af2011011
37. Performance issues with bridging gpu_lidar data / Issue #368 ..., accessed July 22, 2025, https://github.com/gazebosim/ros_gz/issues/368
38. Sensor synchronisation bug between Gazebo and ros2_control with diff_drive_controller, accessed July 22, 2025, https://answers.ros.org/question/387856/
39. joshnewans/mwe_diffdrive_bug: Minimum Working Example for a bug in differential drive ROS/Gazebo simulations - GitHub, accessed July 22, 2025, https://github.com/joshnewans/mwe_diffdrive_bug
40. Ros2: use_sim_time leads to inconsistent clocks - ROS General - Open Robotics Discourse, accessed July 22, 2025, https://discourse.ros.org/t/ros2-use-sim-time-leads-to-inconsistent-clocks/42030
41. Clock and Time - ROS 2 Design, accessed July 22, 2025, https://design.ros2.org/articles/clock_and_time.html
42. Time synchronisation between ROS2 and Gazebo - LXRobotics, accessed July 22, 2025, https://lxrobotics.com/blog/time-synchronisation-ros2-gazebo/
43. Simulating with Gazebo | Articulated Robotics, accessed July 22, 2025, https://articulatedrobotics.xyz/tutorials/ready-for-ros/gazebo/
44. Gazebo No clock received Use_Sim_time Error : r/ROS - Reddit, accessed July 22, 2025, https://www.reddit.com/r/ROS/comments/1i06kb3/gazebo_no_clock_received_use_sim_time_error/
45. Inaccurate FT Sensor Values in Gazebo Classic 11 (22.04 ROS2 ..., accessed July 22, 2025, https://github.com/gazebosim/gazebo-classic/issues/3398
46. Issues / ros-simulation/gazebo_ros_pkgs - GitHub, accessed July 22, 2025, https://github.com/ros-simulation/gazebo_ros_pkgs/issues
47. RANT: Gazebo (new) is a terrible piece of software and I don't think we should recommend it to newbies : r/ROS - Reddit, accessed July 22, 2025, https://www.reddit.com/r/ROS/comments/1jqn8wb/rant_gazebo_new_is_a_terrible_piece_of_software/
48. Gazebo Roadmap - Gazebo ionic documentation, accessed July 22, 2025, https://gazebosim.org/docs/latest/roadmap/
49. ROS 2 Technical Steering Committee Charter, accessed July 22, 2025, https://docs.ros.org/en/foxy/The-ROS2-Project/Governance/ROS2-TSC-Charter.html
50. Open Robotics names ROS 2 Technical Steering Committee - The Robot Report, accessed July 22, 2025, https://www.therobotreport.com/ros-2-steering-committee-open-robotics/
51. Introducing the ROS 2 Technical Steering Committee - ROS General - ROS Discourse, accessed July 22, 2025, https://discourse.ros.org/t/introducing-the-ros-2-technical-steering-committee/6132
52. Roadmap - Eloquent documentation - the official ROS docs, accessed July 22, 2025, https://docs.ros.org/en/eloquent/Roadmap.html
53. Roadmap - ROS 2 Documentation: Foxy 文档, accessed July 22, 2025, https://daobook.github.io/ros2-docs/foxy/Roadmap.html
54. Roadmap - ROS 2 Documentation: Galactic documentation, accessed July 22, 2025, https://docs.ros.org/en/galactic/The-ROS2-Project/Roadmap.html
55. Migration of a Gazebo ROS simulation to a ROS2 Ignition Gazebo - YouTube, accessed July 22, 2025, https://www.youtube.com/watch?v=DY7QTtDpVzw
56. GSoC 2024 - Proposal writing - General - Gazebo Community, accessed July 22, 2025, https://community.gazebosim.org/t/gsoc-2024-proposal-writing/2666


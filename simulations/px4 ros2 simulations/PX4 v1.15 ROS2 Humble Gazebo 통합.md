---
layout: page
title: PX4 v1.15, ROS 2 Humble, Gazebo 통합
permalink: /simulations/px4 ros2 simulations/PX4 v1.15 ROS2 Humble Gazebo 통합
---


본 보고서는 PX4 오토파일럿 버전 1.15.4, Gazebo 시뮬레이터, 그리고 ROS 2 Humble Hawksbill의 호환성에 대한 포괄적이고 심층적인 분석을 제공한다. 분석 결과, 이 기술 스택은 공식적으로 지원되며 매우 강력한 기능을 제공하지만, 성공적인 구현은 다섯 가지 핵심 구성 요소의 정확한 버전 의존성, 올바른 미들웨어 구성, 그리고 특정 빌드 시스템 및 시뮬레이션 브리징의 미묘한 차이에 대한 정밀한 이해에 달려 있음을 확인했다. PX4 v1.15에서 도입된 uXRCE-DDS 미들웨어의 안정화와 실험적인 PX4 ROS 2 인터페이스 라이브러리는 기존의 오프보드 제어를 넘어, ROS 2를 비행 컨트롤러의 네이티브 확장 기능으로 활용하는 새로운 패러다임을 제시한다. 본 문서는 개발자가 이 복잡한 시스템을 안정적으로 구축하고 활용하는 데 필요한 환경 설정, 구성 요소 설치, 고급 애플리케이션 개발 및 문제 해결에 이르는 전 과정을 상세히 안내하는 것을 목표로 한다.

------


성공적인 드론 자율 비행 시스템을 구축하기 위해서는 각 구성 요소의 역할과 버전을 명확히 이해하고, 이들이 어떻게 상호 작용하는지에 대한 깊이 있는 통찰이 필요하다. 이들은 독립적인 세 개의 소프트웨어가 아니라, 버전 정렬이 무엇보다 중요한 긴밀하게 결합된 하나의 시스템으로 보아야 한다.


PX4는 전문가 수준의 오픈소스 오토파일럿 소프트웨어로, 산업계와 학계의 세계적 수준의 개발자들에 의해 개발되고 있다.1 PX4 v1.15는 v1.14에서 이루어진 ROS 2 통합의 기반 위에 중요한 개선 사항들을 추가한 릴리스이며, v1.15.4는 이 시리즈의 안정적인 패치 릴리스이다.3

v1.14부터 시작되어 v1.15에서 더욱 정제된 가장 중요한 아키텍처 변화는 ROS 2 통신을 위해 기존의 Fast-RTPS 브리지에서 uXRCE-DDS 미들웨어로 완전히 전환한 것이다.5 이 변화는 PX4와 ROS 2 간의 통신 안정성과 성능을 크게 향상시켰다.

v1.15 릴리스의 가장 획기적인 특징은 실험적으로 도입된 **PX4 ROS 2 인터페이스 라이브러리**이다. 이 라이브러리는 개발자가 커스텀 비행 모드를 PX4 펌웨어 내부 모드와 동등한 수준의 ROS 2 애플리케이션으로 작성할 수 있게 해주며, 이는 전통적인 오프보드 제어 방식을 뛰어넘는 중대한 발전이다.4

또한, 시뮬레이션 측면에서도 중요한 개선이 있었다. PX4 SITL(Software-In-The-Loop) 인스턴스와 시뮬레이션 백엔드(Gazebo)가 분리되어, 각각 독립적으로 실행하거나 심지어 다른 호스트에서 네트워크를 통해 실행할 수 있게 되었다.4


ROS 2 Humble Hawksbill은 2027년까지 지원되는 장기 지원(Long-Term Support, LTS) 릴리스로, 안정적인 개발 환경을 구축하기 위한 권장 버전이다.6

가장 중요한 점은 ROS 2 Humble이 공식적으로 **Ubuntu 22.04 (Jammy Jellyfish)** 운영체제에서만 지원된다는 것이다.6 다른 버전의 운영체제를 사용하는 것은 수많은 예기치 않은 오류의 원인이 되므로 반드시 준수해야 한다.

ROS 2는 RMW(ROS Middleware Layer)라는 추상화 계층을 통해 다양한 DDS(Data Distribution Service) 구현체를 지원한다. Humble의 기본 RMW는 `rmw_cyclonedds_cpp`이지만, PX4와의 연동 시에는 다른 구현체를 사용하는 것이 더 안정적일 수 있으며, 이는 섹션 2에서 자세히 다룬다.


Gazebo 시뮬레이터는 최근 이름 변경으로 인해 혼란을 야기할 수 있다. 현재 "Gazebo"는 과거 "Ignition Gazebo"로 불리던 최신 버전을 지칭하며, 레거시 버전은 "Gazebo Classic"으로 불린다.5 PX4 v1.14 이후부터는 이 변경 사항이 빌드 타겟 이름에 반영되어, 최신 Gazebo용 타겟은 `$gz_` 접두사를, Gazebo Classic용 타겟은 `$gazebo-classic_` 접두사를 사용한다.7

ROS 2 Humble과 공식적으로 호환되는 Gazebo 버전은 **Gazebo Fortress**이다.16 Gazebo Harmonic을 Humble과 함께 사용할 수도 있지만, 이는 비표준 패키지를 필요로 하므로 권장되지 않는다.16

PX4에서 `$make px4_sitl gz_x500` 명령을 실행하면, PX4는 ROS 2를 통하지 않고 Gazebo Transport를 통해 Gazebo와 직접 통신한다.18 이는 많은 사용자들이 오해하는 부분으로, 시뮬레이터의 센서 데이터를 ROS 2에서 받기 위해서는 별도의 브리지가 필요하다는 것을 의미한다.


성공적인 통합은 단순히 각 구성 요소를 설치하는 것을 넘어, 아래의 다섯 가지 핵심 요소가 정확하게 정렬될 때 비로소 가능하다. 개발 과정에서 발생하는 수많은 문제를 예방하기 위해, 프로젝트 시작 전에 이 "골든 패스"를 확인하는 것은 필수적이다. 이 버전 조합을 따르지 않을 경우, 한 구성 요소의 사소한 불일치가 전혀 다른 부분에서 예측 불가능한 오류를 발생시키는 연쇄 실패로 이어질 수 있다. 예를 들어, `px4_msgs`의 `main` 브랜치를 사용하는 실수는 PX4 v1.15 펌웨어와의 불일치로 인해 `colcon` 빌드 실패나 런타임 통신 오류를 유발한다.12

| **구성 요소**     |
| ----------------- |
| 운영체제 (OS)     |
| ROS 2 배포판      |
| PX4 오토파일럿    |
| `px4_msgs` 패키지 |
| Gazebo 시뮬레이터 |

표 1.1: PX4/ROS 2 Humble 통합 스택 (골든 패스)

------


PX4와 ROS 2 간의 원활한 통신은 uXRCE-DDS 미들웨어에 대한 정확한 이해와 구성에 달려있다. 이 섹션에서는 단순히 설정 방법을 나열하는 것을 넘어, 통신 실패의 근본적인 원인을 파악하고 해결하는 데 초점을 맞춘다. 성공적인 통신은 PX4 전용 에이전트 설정과 ROS 2 네트워크 환경 구성이라는 두 가지 축을 모두 고려해야만 가능하다.


PX4와 ROS 2의 통신은 클라이언트/에이전트 모델을 기반으로 하는 Micro XRCE-DDS를 통해 이루어진다.6

- **`uxrce_dds_client`**: PX4 비행 컨트롤러(또는 SITL)에서 실행되며, uORB 토픽 데이터를 DDS 세계로 보내거나 받는 역할을 한다.
- **`MicroXRCEAgent`**: 컴패니언 컴퓨터(ROS 2가 실행되는 PC)에서 실행되며, 클라이언트의 프록시 역할을 하여 글로벌 DDS 데이터 공간과 데이터를 교환한다.

이 구조에서 핵심은 `uxrce_dds_client`가 PX4 펌웨어 빌드 시점에 PX4-Autopilot 소스 트리 내의 uORB 메시지 정의(`.msg` 파일)로부터 생성된다는 점이다.6 이는 ROS 2 워크스페이스에서 사용되는 `px4_msgs` 패키지의 버전이 펌웨어 빌드에 사용된 메시지 정의와 정확히 일치해야 하는 이유를 설명한다. 브리지 될 uORB 토픽의 목록은 소스 트리 내의 `dds_topics.yaml` 파일에 명시되어 있으며, 이 파일이 사실상 브리지의 "매니페스트" 역할을 한다.6


가장 안정적인 방법은 `Micro-XRCE-DDS-Agent`를 소스에서 직접 클론하여 빌드하는 것이다.9 설치가 완료되면, SITL 환경과의 UDP 통신을 위해 다음 명령어를 사용하여 에이전트를 실행한다.

```Bash
MicroXRCEAgent udp4 -p 8888
```

이 에이전트는 PX4와 ROS 2 간의 모든 통신을 중계하므로, 별도의 터미널에서 항상 실행 상태를 유지해야 한다.22


많은 개발자들이 ROS 2의 내부 네트워킹 설정 문제로 인해 통신 실패를 겪는다. 사용자가 `ros2 topic list` 명령으로 토픽이 보이지 않을 때, SITL이나 XRCE 에이전트의 문제라고 단정하기 쉽다. 하지만 근본 원인은 종종 보이지 않는 ROS 2 DDS 네트워크 구성의 불일치에 있다.

- **RMW 구현체 (`RMW_IMPLEMENTATION`)**: 이 환경 변수는 ROS 2가 사용할 DDS 벤더를 선택한다. Humble의 기본값은 `rmw_cyclonedds_cpp`이지만, 커뮤니티의 여러 사례에 따르면 eProsima Fast DDS 구현체인 `rmw_fastrtps_cpp`가 PX4 연동 시 더 안정적인 성능을 보이는 경우가 많다.24

  `.bashrc` 파일에 다음 라인을 추가하여 RMW를 명시적으로 설정하는 것이 권장된다.

  ```Bash
  export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
  ```

- **ROS 도메인 ID (`ROS_DOMAIN_ID`)**: 이 환경 변수는 DDS 네트워크를 분할하는 데 사용된다. 동일한 네트워크상의 모든 노드(XRCE-DDS 에이전트 포함)는 동일한 도메인 ID를 가져야만 서로 통신할 수 있다. "토픽이 보이지 않는" 문제에 대한 가장 일반적인 해결책 중 하나는 모든 터미널의 `.bashrc` 파일에 도메인 ID를 명시적으로 설정하는 것이다.24

  ```Bash
  export ROS_DOMAIN_ID=0
  ```

이러한 네트워크 설정은 PX4 문서에서 명시적으로 강조되지 않는 경우가 많아 사용자들이 간과하기 쉽다. 따라서 문제 해결 시, PX4 관련 설정뿐만 아니라 ROS 2 네트워크 환경 자체를 최우선으로 점검하는 체계적인 접근이 필요하다.

------


이 섹션은 완전한 기능의 개발 및 시뮬레이션 환경을 구축하기 위한 실용적이고 단계적인 가이드를 제공한다. 핵심은 오토파일럿 제어를 위한 데이터 흐름과 시뮬레이션 센싱을 위한 데이터 흐름이라는 두 개의 병렬적인 경로를 이해하고, 이를 각각 관리하는 방법을 제시하는 것이다.


ROS 2 워크스페이스는 컴패니언 컴퓨터에서 실행될 모든 노드와 로직의 집합소이다.

1. **워크스페이스 생성**:

   ```Bash
   mkdir -p ~/px4_ros_ws/src
   cd ~/px4_ros_ws/src
   ```

2. **`px4_msgs` 클론**: PX4 펌웨어와의 메시지 호환성을 위해 정확한 브랜치를 지정하는 것이 매우 중요하다.

   ```Bash
   git clone https://github.com/PX4/px4_msgs.git -b release/1.15
   ```

   `release/1.15` 브랜치는 PX4 v1.15.x 릴리스와 호환되는 메시지 정의를 포함한다.9

3. **`px4_ros_com` 클론 (선택 사항이지만 권장)**: v1.14 이후 브리지 자체에는 더 이상 필요하지 않지만, 이 패키지에는 오프보드 제어 및 리스너에 대한 매우 유용한 예제 코드가 포함되어 있다.6

   ```Bash
   git clone https://github.com/PX4/px4_ros_com.git
   ```

4. **워크스페이스 빌드**:

   ```Bash
   cd ~/px4_ros_ws
   colcon build
   ```

   빌드 중 Python 의존성 문제(`empy`, `setuptools` 등)가 발생할 수 있으며, 이 경우 관련 패키지의 버전을 조정해야 할 수 있다.17


1. **PX4-Autopilot 클론 및 버전 확인**: 재현성을 위해 정확한 버전을 체크아웃하는 것이 중요하다.

   ```Bash
   git clone https://github.com/PX4/PX4-Autopilot.git --recursive
   cd PX4-Autopilot
   git checkout v1.15.4
   git submodule update --init --recursive
   ```

2. **SITL 타겟 빌드**:

   ```Bash
   make px4_sitl_default
   ```

3. **시뮬레이션 실행**: 두 개의 터미널이 필요하다.

   - **터미널 1 (에이전트)**:

     ```Bash
     MicroXRCEAgent udp4 -p 8888
     ```

   - **터미널 2 (SITL 및 Gazebo)**:

     ```Bash
     make px4_sitl gz_x500
     ```

   이 명령어는 PX4 비행 스택을 SITL 모드로 시작하고, `x500` 모델과 함께 Gazebo GUI를 실행한다.7


개발자가 겪는 가장 큰 혼란 중 하나는 두 가지 다른 통신 경로를 구분하지 못하는 것이다.

1. **제어 경로**: ROS 2 노드 -->> `MicroXRCEAgent` -->> PX4 SITL -->> Gazebo (모터 제어)
2. **센싱 경로**: Gazebo (카메라 등) -->> `ros_gz_bridge` -->> ROS 2 노드 (센서 데이터 수신)

카메라와 같은 Gazebo 센서의 데이터는 PX4의 uORB 토픽이 아니므로 `MicroXRCEAgent`를 통해 전달되지 않는다.18 이 데이터를 ROS 2에서 사용하려면 `ros_gz_bridge`라는 별도의 도구가 필요하다.

- **브리지 설치**:

  ```Bash
  sudo apt install ros-humble-ros-gz
  ```

- **브리지 사용 예시 (카메라)**: Gazebo의 토픽 이름은 모델의 SDF 파일에 따라 길고 복잡할 수 있다. `gz topic -l` 명령어로 정확한 토픽 이름을 확인한 후 브리지를 실행해야 한다.

  ```Bash
  ros2 run ros_gz_bridge parameter_bridge /world/default/model/x500_mono_cam/link/camera_link/sensor/camera/image@sensor_msgs/msg/Image@gz.msgs.Image
  ```

  이 명령어는 커뮤니티 포럼에서 공유된 성공적인 사례이다.28 포인트 클라우드와 같은 다른 센서들도 유사한 패턴으로 브리징할 수 있다.29


PX4와 ROS 2는 서로 다른 좌표계를 표준으로 사용하며, 이는 데이터 교환 시 반드시 변환이 필요함을 의미한다.6

- **PX4**: 월드 프레임은 **NED** (North, East, Down), 바디 프레임은 **FRD** (Forward, Right, Down)를 사용한다.
- **ROS 2**: 관례적으로 월드 프레임은 **ENU** (East, North, Up), 바디 프레임은 **FLU** (Forward, Left, Up)를 사용한다.

따라서 PX4로부터 위치 데이터를 구독하거나 ROS 2에서 제어 목표점을 발행하는 모든 노드는 이 좌표계 간의 변환을 수행해야 한다. 예를 들어, FLU에서 FRD로 변환하려면 X축(전방)을 기준으로 π 라디안 회전이 필요하다.6 이러한 변환은 쿼터니언 연산을 통해 코드 내에서 구현되어야 한다.

------


이 섹션에서는 기본 설정을 넘어, PX4 v1.15가 제공하는 혁신적인 기능들을 탐구한다. 핵심은 v1.15가 ROS 2를 단순한 원격 제어 도구가 아닌, 비행 컨트롤러 자체의 네이티브 확장 기능으로 취급하기 시작했다는 패러다임의 전환을 이해하는 것이다.


전통적인 오프보드 제어는 컴패니언 컴퓨터에서 위치, 속도 등의 목표점(setpoint) 스트림을 PX4의 일반적인 모드로 전송하는 방식이었다. 하지만 PX4 v1.15에서 실험적으로 도입된 **PX4 ROS 2 인터페이스 라이브러리**는 완전히 새로운 접근 방식을 제공한다. Auterion의 지원으로 개발된 이 라이브러리를 통해, ROS 2 노드는 자신만의 비행 모드를 정의하고 이를 PX4 비행 컨트롤러에 동적으로 등록할 수 있다.4

이 새로운 패러다임의 주요 이점은 다음과 같다 8:

- **명명된 모드(Named Modes)**: 커스텀 모드가 QGroundControl과 같은 GCS에 이름으로 표시되며, 사용자가 직접 선택할 수 있다.
- **안전 장치 통합(Failsafe Integration)**: 모드가 PX4의 안전 장치 상태 머신(failsafe state machine)에 완벽하게 통합된다.
- **무장 점검(Arming Checks)**: 모드 활성화에 필요한 자체적인 사전 무장 점검 조건을 정의할 수 있다.
- **모드 실행기(Mode Executors)**: 복잡한 임무를 위해 모드 시퀀스를 관리하는 상위 수준의 상태 머신(`executor`)을 생성할 수 있다.

이 기능은 v1.15에서 **실험적(experimental)**으로 분류되며, API는 향후 변경될 수 있다는 점에 유의해야 한다.4 그럼에도 불구하고, 이는 복잡한 자율성을 구현하는 데 있어 기존 오프보드 모드의 한계를 극복하는 강력한 대안을 제시한다.


커스텀 비행 모드를 만드는 과정은 고수준의 로직을 리소스가 제한된 비행 컨트롤러에서 강력한 성능의 컴패니언 컴퓨터로 옮기면서도, PX4 비행 스택의 안전성과 구조는 유지할 수 있게 해준다.8

개념적인 개발 과정은 다음과 같다:

1. `px4_msgs`와 `px4-ros2-interface-lib`가 포함된 ROS 2 워크스페이스를 준비한다.
2. 라이브러리의 `ModeBase` 클래스를 상속받는 C++ 노드를 생성한다.
3. `on_activate()` (모드 활성화 시 호출) 및 `update()` (주기적으로 호출)와 같은 가상 함수를 구현하여 모드의 동작을 정의한다.
4. 생성된 모드를 PX4 인스턴스에 등록하여 GCS에서 선택 가능하게 만든다.

이러한 접근 방식은 MAVLink/Offboard 프로토콜의 한계에 대한 PX4 개발 커뮤니티의 직접적인 응답이다. 이는 PX4가 내부적으로 모든 기능을 복제하려는 대신, ROS라는 더 넓은 로봇 생태계와 더 깊이 통합하려는 전략적 결정을 보여준다. ROS 커뮤니티에서도 "더 나은 통합"과 "불필요한 브리지 제거"에 대한 요구가 지속적으로 제기되어 왔으며 32, 이 라이브러리는 그 방향으로 나아가는 중요한 첫걸음이다.

------


이 섹션은 연구 과정에서 발견된 실제 문제들을 직접적으로 다루는 상세한 문제 해결 매뉴얼 역할을 한다. 대부분의 오류는 예측 가능한 범주(빌드, 미들웨어, 시뮬레이션)에 속하며, 체계적으로 진단할 수 있다.


- **Humble에서의 `ModuleNotFoundError`**:
  - **증상**: `$colcon build`는 성공하지만, `$ros2 run` 또는 `$ros2 topic echo` 실행 시 `ModuleNotFoundError: No module named 'px4'` 또는 "The message type 'px4/msg/...' is invalid" 오류가 발생한다.33
  - **근본 원인**: 이는 Humble 환경에서 PX4 소스 트리 내부에서 메시지를 빌드할 때 발생하는 CMake 범위 지정 문제이다. 생성된 메시지에 대한 Python 경로가 환경에 올바르게 내보내지지 않는다.33
  - **해결책**: 가장 확실한 해결책은 **PX4 트리 내부에서 메시지를 빌드하지 않는 것**이다. 항상 별도의 ROS 2 워크스페이스를 사용하고, 섹션 3.1에서 설명한 대로 버전이 일치하는 `px4_msgs` 패키지를 클론하여 사용해야 한다. 이는 `colcon`이 이해하는 방식으로 메시지 생성을 격리시킨다.
- **`px4_msgs` 버전 불일치**:
  - **증상**: `$colcon build`가 실패하거나, 메시지 직렬화/역직렬화와 관련된 런타임 오류가 발생한다.34
  - **근본 원인**: ROS 2 워크스페이스의 `px4_msgs` 브랜치가 사용 중인 PX4 펌웨어 버전과 일치하지 않는다.
  - **해결책**: `px4_msgs` 브랜치가 PX4 버전과 엄격하게 일치하는지 확인한다 (예: PX4 v1.15.x의 경우 `release/1.15`).


- **증상**: PX4 SITL과 `MicroXRCEAgent`가 실행 중이지만, `$ros2 topic list`에 `/fmu/`로 시작하는 토픽이 표시되지 않는다.25
  - **진단 단계**:
    1. 에이전트 터미널에서 TX/RX 통계 확인. 트래픽이 없다면 SITL과 에이전트 간의 UDP 연결 문제(방화벽 등)이다.
    2. 에이전트에 트래픽이 표시된다면, 에이전트와 ROS 2 노드 간의 문제이다.
    3. **해결책 1 (RMW)**: `.bashrc`에 `export RMW_IMPLEMENTATION=rmw_fastrtps_cpp`를 설정하고 모든 터미널을 다시 시작한다.24
    4. **해결책 2 (도메인 ID)**: `.bashrc`에 `export ROS_DOMAIN_ID=0`을 설정하고 모든 터미널을 다시 시작한다.24


- **증상 1**: `$ros_gz_bridge`가 토픽 브리징에 실패하거나 오류를 보고한다.
  - **근본 원인**: 잘못된 토픽 이름 또는 메시지 유형. Gazebo 토픽 이름은 길고 모델에 따라 다르다.
  - **해결책**: 실행 중인 시뮬레이션에서 `$gz topic -l`을 사용하여 **정확한** 토픽 이름을 확인한다. 28의 예제를 템플릿으로 사용한다.
- **증상 2**: 데이터는 올바르게 브리징되지만, Rviz2에서 "No tf data" 오류가 발생하고 모델이 올바르게 표시되지 않는다.29
  - **근본 원인**: `$ros_gz_bridge`는 센서 데이터만 브리징한다. 센서 간의 관계 및 월드 프레임과의 관계를 나타내는 좌표 프레임 변환(TF 트리)은 브리징하지 않는다.
  - **해결책**: 이 변환 정보를 발행하는 별도의 ROS 2 노드가 필요하다. 표준 도구는 URDF 모델 설명과 조인트 상태를 입력받아 TF 트리를 발행하는 `robot_state_publisher`이다. 간단한 경우, `map`, `odom`, `base_link`와 같은 프레임 간의 관계를 정의하기 위해 정적 변환 발행기(static transform publisher)를 사용할 수 있다.


문제가 발생했을 때 무작위로 추측하는 대신, 구조화된 접근 방식을 따르면 문제의 원인을 신속하게 파악할 수 있다. 아래의 진단 흐름도는 커뮤니티 포럼에서 논의된 다양한 문제 해결 경험을 체계적인 프로세스로 정리한 것이다.2

| **증상**                    |
| --------------------------- |
| `colcon build` 실패         |
| `/fmu/` 토픽 없음           |
| Rviz2에서 "No tf data" 오류 |
| Gazebo 센서 토픽 없음       |

표 5.1: PX4-ROS 2 통합 문제 진단 흐름도

------



결론적으로, PX4 v1.15.4, ROS 2 Humble, 그리고 Gazebo Fortress 스택은 고급 드론 개발을 위한 강력하고 실행 가능한 조합이다. 이 시스템의 안정성은 본 보고서에서 제시한 "골든 패스" 버전 관리 및 구성 지침을 엄격하게 따를 때에만 보장된다. 주요 과제는 핵심 구성 요소의 버그가 아니라, 그들 간의 통합 복잡성과 환경 설정에 요구되는 정밀성이다.


1. **버전 관리는 타협 불가**: 항상 프로젝트 시작 시 전체 스택(OS, ROS, PX4, Gazebo, `px4_msgs`)을 정의하고 해당 버전을 고정하라. 섹션 1.4의 표를 지침으로 삼아야 한다.
2. **워크스페이스 격리**: PX4-Autopilot 소스 트리와 분리된 깨끗한 ROS 2 워크스페이스를 유지하라. 이 둘을 병합하려고 시도하지 말아야 한다.
3. **환경 표준화**: `.bashrc` 파일이나 소싱 스크립트를 사용하여 모든 터미널에서 `RMW_IMPLEMENTATION`과 `ROS_DOMAIN_ID`를 일관되게 설정하라.
4. **체계적인 디버깅**: 문제가 발생하면 섹션 5.4의 진단 흐름도를 사용하라. 문제를 빌드, 미들웨어, 시뮬레이션 브리징 문제로 분리하여 접근해야 한다.
5. **새로운 패러다임 수용**: 새로운 프로젝트의 경우, PX4 ROS 2 인터페이스 라이브러리를 학습하는 데 시간을 투자하라. v1.15에서는 실험적이지만, 이는 고수준 제어의 미래이며 레거시 오프보드 모드에 비해 상당한 이점을 제공한다.
6. **커뮤니티에 기여**: 이 생태계는 지식 공유를 통해 성장한다. GitHub에 문제를 명확하게 문서화하고33, PX4 Discuss 포럼에서 해결책을 공유하라.26


1. PX4 Autopilot Software - GitHub, accessed July 25, 2025, https://github.com/PX4/PX4-Autopilot
2. Px4 simulation with ROS2 Humble and Gazebo, accessed July 25, 2025, https://discuss.px4.io/t/px4-simulation-with-ros2-humble-and-gazebo/32661
3. Releases / PX4/PX4-Autopilot - GitHub, accessed July 25, 2025, https://github.com/PX4/PX4-Autopilot/releases
4. PX4-Autopilot v1.15 Release Notes | PX4 Guide (main), accessed July 25, 2025, https://docs.px4.io/main/en/releases/1.15
5. PX4-Autopilot v1.14 Release Notes, accessed July 25, 2025, https://px-4.com/v1.14/en/releases/1.14.html
6. ROS 2 User Guide | PX4 Guide (v1.15), accessed July 25, 2025, https://docs.px4.io/v1.15/en/ros2/user_guide.html
7. ROS 2 User Guide | PX4 Guide (main) - PX4 docs, accessed July 25, 2025, https://docs.px4.io/main/en/ros2/user_guide
8. PX4 ROS 2 Control Interface | PX4 Guide (main), accessed July 25, 2025, https://docs.px4.io/main/en/ros2/px4_ros2_control_interface
9. Custom flight modes using PX4 and ROS 2 - RIIS LLC, accessed July 25, 2025, https://www.riis.com/blog/custom-flight-modes-using-px4-and-ros2
10. ROS 2 Documentation: Humble documentation, accessed July 25, 2025, https://docs.ros.org/en/humble/index.html
11. Distributions - ROS 2 Documentation: Humble documentation, accessed July 25, 2025, https://docs.ros.org/en/humble/Releases.html
12. PX4/px4_msgs: ROS/ROS2 messages that match the uORB messages counterparts on the PX4 Firmware - GitHub, accessed July 25, 2025, https://github.com/PX4/px4_msgs
13. For ROS2 Beginners - Install ROS 2 and Gazebo on Ubuntu 22.04 Jammy (LTS) Step-by-Step Instruction | by Claudia Yao | Medium, accessed July 25, 2025, https://medium.com/@claudia.yao2012/for-ros2-beginners-install-ros-2-and-gazebo-on-ubuntu-22-04-jammy-lts-step-by-step-instruction-08327c3764a5
14. Hi everyone. Can i install ros2 on Ubuntu 24.04? : r/ROS - Reddit, accessed July 25, 2025, https://www.reddit.com/r/ROS/comments/1crwzw9/hi_everyone_can_i_install_ros2_on_ubuntu_2404/
15. PX4-Autopilot v1.14 Release Notes | PX4 Guide (main), accessed July 25, 2025, https://docs.px4.io/main/en/releases/1.14
16. Installing Gazebo with ROS - Gazebo ionic documentation, accessed July 25, 2025, https://gazebosim.org/docs/latest/ros_installation/
17. ROS2 Humble, Gazebo Harmonic, PX4, QGroundControl, Micro XRCE-DDS Agent & Client Installation | by Erdem | Medium, accessed July 25, 2025, https://medium.com/@erdem.ku.3.14/ros2-humble-gazebo-harmonic-px4-ve-micro-xrce-dds-agent-client-installation-aad32d8f5669
18. Difficulties with gazebo version used by PX4- SITL - PX4 Discussion Forum, accessed July 25, 2025, https://discuss.px4.io/t/difficulties-with-gazebo-version-used-by-px4-sitl/34137
19. Gazebo Simulation | PX4 Guide (main), accessed July 25, 2025, https://docs.px4.io/main/en/sim_gazebo_gz/
20. ROS 2 User Guide - GitHub, accessed July 25, 2025, https://github.com/PX4/PX4-user_guide/blob/main/ja/ros2/user_guide.md
21. monemati/PX4-ROS2-Gazebo-YOLOv8 - GitHub, accessed July 25, 2025, https://github.com/monemati/PX4-ROS2-Gazebo-YOLOv8
22. ROS 2 User Guide (PX4-ROS 2 Bridge) - PX4 docs, accessed July 25, 2025, https://docs.px4.io/v1.12/en/ros/ros2_comm.html
23. SathanBERNARD/PX4-ROS2-Gazebo-Drone-Simulation-Template - GitHub, accessed July 25, 2025, https://github.com/SathanBERNARD/PX4-ROS2-Gazebo-Drone-Simulation-Template
24. ROS2 - PX4 - Gazebo - QGroundControl does not work - #15 by mdobrea, accessed July 25, 2025, https://discuss.px4.io/t/ros2-px4-gazebo-qgroundcontrol-does-not-work/30316/15
25. PX4 to ROS2 connection does not work - ROS 1 / ROS 2 - PX4 Discussion Forum, accessed July 25, 2025, https://discuss.px4.io/t/px4-to-ros2-connection-does-not-work/31275
26. ROS2 - PX4 - Gazebo - QGroundControl does not work, accessed July 25, 2025, https://discuss.px4.io/t/ros2-px4-gazebo-qgroundcontrol-does-not-work/30316
27. PX4/px4_ros_com: ROS2/ROS interface with PX4 through a Fast-RTPS bridge - GitHub, accessed July 25, 2025, https://github.com/PX4/px4_ros_com
28. Ros_gz_bridge not working. Unable to use rviz2 to fetch PX4 ..., accessed July 25, 2025, https://discuss.px4.io/t/ros-gz-bridge-not-working-unable-to-use-rviz2-to-fetch-px4-x500-mono-cam-image-topic/44447
29. How to get depth data from the PX4 Gazebo simulation and visualize it in RVIZ2?, accessed July 25, 2025, https://discuss.px4.io/t/how-to-get-depth-data-from-the-px4-gazebo-simulation-and-visualize-it-in-rviz2/37916
30. ROS 2 User Guide - PX4 docs, accessed July 25, 2025, https://docs.px4.io/v1.14/en/ros/ros2_comm.html
31. PX4 ROS 2 Control Interface | PX4 Guide (v1.15), accessed July 25, 2025, https://docs.px4.io/v1.15/en/ros2/px4_ros2_control_interface.html
32. Gauging the aerial robotics community - Aerial Vehicles - ROS Discourse, accessed July 25, 2025, https://discourse.ros.org/t/gauging-the-aerial-robotics-community/29757
33. building PX4 with colcon on ubuntu22 - ros2 humble: ROS2 ... - GitHub, accessed July 25, 2025, https://github.com/PX4/PX4-Autopilot/issues/21128
34. [RTPS_READER_HISTORY Error] Gazebo & Error When Running offboard_control.py / Issue #53 / PX4/px4_msgs - GitHub, accessed July 25, 2025, https://github.com/PX4/px4_msgs/issues/53
35. Issues Running ROS2-PX4 - ROS 1 / ROS 2 - Discussion Forum for PX4, Pixhawk, QGroundControl, MAVSDK, MAVLink, accessed July 25, 2025, https://discuss.px4.io/t/issues-running-ros2-px4/28400
36. Please help me out with a SITL for drones. PX4, ROS2 Humble, Gazebo Harmonic, Ubuntu 22.04 - Reddit, accessed July 25, 2025, https://www.reddit.com/r/drones/comments/1lam6c1/please_help_me_out_with_a_sitl_for_drones_px4/
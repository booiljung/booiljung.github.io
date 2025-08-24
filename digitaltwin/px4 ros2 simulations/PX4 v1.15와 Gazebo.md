# PX4 v1.15와 Gazebo 고급 드론 시뮬레이션
[PX4 ROS2 Simulation](./index.md)



본 보고서는 드론용 오픈소스 비행 제어 펌웨어인 PX4 버전 1.15와 로보틱스 시뮬레이터 Gazebo 버전별 호환성을 심층적으로 분석하고, 개발자를 위한 명확한 기술 가이드를 제공하는 것을 목적으로 한다. 이 분석은 단순한 호환성 목록을 넘어, 운영체제(OS) 및 로봇 운영 체제(ROS)와의 상호 의존성까지 포함한다. 본 보고서는 개발자가 안정적이고 기능적인 시뮬레이션 환경을 구축하는 데 필요한 모든 정보를 제공하는 권위 있는 자료가 되는 것을 목표로 한다.


본 보고서의 핵심 분석 결과는 다음과 같다.

- PX4 v1.15를 위한 주력 개발 경로는 Ubuntu 22.04 LTS 운영체제에서 Gazebo Harmonic과 ROS 2 Humble을 함께 사용하는 것이다. 이 조합은 가장 안정적이며 최신 기능을 활용할 수 있는 환경을 제공한다.
- Ubuntu 20.04 LTS와 Gazebo Classic 11을 사용하는 레거시 경로는 여전히 유효하지만, 관련 소프트웨어들의 서비스 종료(EOL)가 임박함에 따라 새로운 프로젝트에는 권장되지 않는다.
- ROS 2 Humble이 공식적으로 지원하는 Gazebo 버전(Fortress)과 PX4 커뮤니티가 권장하는 버전(Harmonic) 사이에 불일치가 존재한다. 이는 개발 환경 구축 시 특별한 주의와 특정 설치 절차를 요구하는 주요 마찰 지점이다.1
- PX4 v1.14에서 시작되어 v1.15에서 더욱 개선된 아키텍처 변경 사항들은 최신 Gazebo와의 통합을 통해 더 유연하고 강력한 시뮬레이션 경험을 가능하게 한다.3


개발자가 자신의 요구사항에 맞는 최적의 환경을 신속하게 결정할 수 있도록, PX4 v1.15 기준의 마스터 호환성 매트릭스를 아래 표와 같이 제공한다. 이 표는 운영체제, ROS 2, Gazebo 버전 간의 복잡한 관계를 시각적으로 요약하여 의사결정을 돕는다.

**표 1: PX4 v1.15 마스터 호환성 매트릭스**

| OS + ROS 2 버전                 | Gazebo Classic 11         | Gazebo Garden              | Gazebo Fortress            | Gazebo Harmonic |
| ------------------------------- | ------------------------- | -------------------------- | -------------------------- | --------------- |
| **Ubuntu 22.04 + ROS 2 Humble** | 권장하지 않음             | 레거시/지원 중단           | 우회 방안과 함께 사용 가능 | **권장 경로**   |
| **Ubuntu 20.04 + ROS 2 Foxy**   | **레거시/지원 중단 예정** | 우회 방안과 함께 사용 가능 | 권장하지 않음              | 권장하지 않음   |

**매트릭스 해설:**

- **권장 경로 (Recommended Path):** 가장 안정적이고 충분히 테스트되었으며, 모든 기능을 활용할 수 있는 최상의 조합이다. 새로운 프로젝트는 이 경로를 따라야 한다.
- **우회 방안과 함께 사용 가능 (Viable with Workarounds):** 작동은 가능하지만, 비표준적인 설정이나 특정 문제 해결 과정이 필요하며 일부 기능 제약이 있을 수 있다. 예를 들어, ROS 2 Humble의 공식 지원 버전인 Fortress는 PX4와 연동 시 `ros_gz_bridge`를 소스에서 직접 빌드해야 하는 등의 추가 작업이 필요하다.2
- **레거시/지원 중단 (Legacy / Deprecated):** 작동은 하지만, 기반 소프트웨어가 서비스 종료(EOL)되었거나(예: Gazebo Garden 4) 곧 종료될 예정인(예: Gazebo Classic 11 5) 조합이다. 기존 프로젝트 유지보수 외에는 사용을 지양해야 한다.
- **권장하지 않음 (Not Recommended):** 불안정하거나, 공식적으로 지원되지 않거나, 심각한 호환성 문제가 알려진 조합이다. 예를 들어, Ubuntu 22.04에서는 Gazebo Classic을 설치 및 실행할 수 없다.6


PX4와 Gazebo의 호환성을 정확히 이해하기 위해서는 먼저 Gazebo 시뮬레이터 자체가 'Gazebo Classic'과 '최신 Gazebo'라는 두 개의 주요 갈래로 나뉘어 발전해왔다는 사실을 인지해야 한다. 이 두 버전은 아키텍처, 기능, 지원 주기 등 모든 면에서 근본적인 차이를 가지며, 이는 PX4 개발 환경 선택에 직접적인 영향을 미친다.


Gazebo는 2002년부터 개발된 유서 깊은 3D 로보틱스 시뮬레이터이다. 초기의 Gazebo는 모든 기능이 하나의 거대한 애플리케이션에 통합된 모놀리식(monolithic) 아키텍처를 가지고 있었다.8 그러나 소프트웨어의 복잡성이 증가함에 따라, 2017년을 기점으로 프로젝트는 두 방향으로 분기되었다. 기존의 모놀리식 버전은 계속 유지되었고, 이와 별개로 'Ignition'이라는 새로운 이름 아래 현대적인 소프트웨어 설계 원칙에 따라 여러 개의 독립적인 라이브러리 모음으로 재구성된 버전이 개발되기 시작했다.8

2022년, 'Ignition'이라는 이름에 대한 상표권 문제로 인해 개발팀은 명칭을 재정리했다. 새로운 라이브러리 기반 버전은 다시 'Gazebo'라는 이름을 사용하게 되었고, 기존의 모놀리식 버전은 혼동을 피하기 위해 'Gazebo Classic'으로 명명되었다.8 따라서 현재 'Gazebo'는 최신 버전을, 'Gazebo Classic'은 레거시 버전을 지칭한다.


Gazebo Classic은 버전 11을 마지막으로 하는 레거시 시뮬레이터이다. 수년 동안 로보틱스 커뮤니티의 표준 시뮬레이션 도구로 사용되어 왔으며, ODE(Open Dynamics Engine) 물리 엔진과 OpenGL 렌더링을 기반으로 한다.8 특히 Gazebo 9와 11은 장기 지원(LTS) 버전으로 널리 사용되었다.9

그러나 Gazebo Classic 11은 2025년 1월을 기점으로 공식적인 서비스 지원이 종료(End-of-Life, EOL)되었다.5 이는 더 이상 새로운 기능 추가, 보안 업데이트, 버그 수정이 제공되지 않음을 의미한다. 또한 Gazebo Classic은 Ubuntu 20.04(Focal Fossa)와 같은 구형 운영체제에 의존적이며, Ubuntu 22.04(Jammy Jellyfish) 이상에서는 공식적으로 지원되지 않는다.6 따라서 Gazebo Classic은 기존 프로젝트의 유지보수 목적 외에는 더 이상 권장되지 않는다.


최신 Gazebo는 완전히 새로운 아키텍처를 기반으로 한다. `gz-sim`(시뮬레이션 서버), `gz-transport`(통신), `gz-rendering`(렌더링), `gz-physics`(물리) 등 기능별로 세분화된 `gz-*` 라이브러리들의 집합으로 구성되어 있다.12 이러한 모듈식 설계는 유연성과 확장성을 크게 향상시켰다.

최신 Gazebo는 알파벳 순서의 코드명으로 릴리스되며(예: Citadel, Fortress, Garden, Harmonic, Ionic), 일반 릴리스와 장기 지원(LTS) 릴리스로 구분된다.14 각 릴리스는 주목할 만한 기능 개선을 포함한다.

- **Gazebo Garden (EOL):** Python 바인딩 도입, 디지털 고도 모델(DEM) 지원, glTF/GLB 메시 지원 등 사용자 경험을 크게 향상시킨 첫 릴리스였다.4
- **Gazebo Harmonic (LTS):** 순수 Python으로 플러그인을 작성할 수 있게 되었고, 렌더링 기능(광각 렌즈, 렌즈 플레어), 유체 역학 시뮬레이션(부가 질량), 기어박스 조인트 개선 등 사실성을 높이는 기능들이 대거 추가되었다.13

이러한 지속적인 발전은 개발자들이 더 현실적이고 복잡한 시나리오를 시뮬레이션할 수 있게 해주며, PX4 커뮤니티가 Gazebo Classic에서 최신 Gazebo로 전환하는 주된 이유가 된다.

특히 주목해야 할 점은 생태계 전반에 걸친 서비스 종료(EOL)의 연쇄 효과이다. Gazebo Classic(2025년 1월 EOL)과 ROS 1 Noetic(2025년 5월 EOL)의 지원 종료 시점은 거의 일치한다.5 더 나아가 이 기술 스택의 기반이 되는 운영체제인 Ubuntu 20.04 역시 자체적인 지원 종료 일정을 가지고 있다. 이는 개별 소프트웨어의 업그레이드 권고를 넘어, 해당 기술 스택 전체가 수명을 다했음을 의미하는 강력한 신호이다. PX4 프로젝트 역시 이러한 흐름에 발맞춰 버전 1.14부터 최신 Gazebo를 기본 시뮬레이터로 채택함으로써 기술적 전환을 공식화했다.16 따라서 개발자에게 Gazebo Classic을 계속 사용하는 결정은 단순한 선호의 문제가 아니라, 정의된 만료일이 있는 프로젝트 위험 관리의 영역에 속하게 된다. 이 보고서는 이러한 관점에서 최신 Gazebo로의 전환을 단순한 권장 사항이 아닌, 필수적인 전략으로 간주한다.


PX4 v1.15의 향상된 시뮬레이션 기능은 이전 버전인 v1.14에서 이루어진 근본적인 아키텍처 변화 위에 구축되었다. 이 변화의 핵심은 최신 Gazebo와의 통신 방식을 현대화하여 효율성과 유연성을 극대화한 것이다.


PX4 v1.15 시뮬레이션 환경의 토대는 v1.14 릴리스에서 마련되었다. 이 버전의 가장 중요한 변화 중 하나는 `PX4-Autopilot#20319` 풀 리퀘스트를 통해 최신 Gazebo의 핵심 통신 계층인 `gz-transport`를 네이티브로 지원하기 시작했다는 점이다.16

이전까지 Gazebo Classic과의 연동은 MAVLink 프로토콜을 기반으로 하는 `gazebo-mavlink-sitl` 플러그인을 통해 이루어졌다. 이는 시뮬레이터와 PX4 SITL(Software-in-the-Loop) 간의 데이터를 MAVLink 메시지로 변환하여 주고받는 방식이었다. 그러나 `gz-transport`의 네이티브 지원은 이러한 중간 변환 과정 없이 PX4가 시뮬레이터와 직접적이고 효율적인 통신 채널을 형성할 수 있게 만들었다. 이는 성능 저하를 유발하는 오버헤드를 줄이고, 시뮬레이션 데이터의 지연 시간을 최소화하며, 최신 Gazebo가 제공하는 풍부한 센서 데이터와 물리 정보를 온전히 활용할 수 있는 길을 열었다. 이 변화로 인해 PX4의 기본 시뮬레이션 환경은 Gazebo Classic에서 최신 Gazebo로 공식적으로 전환되었다.16


PX4 v1.15 릴리스는 v1.14에서 마련된 기반 위에서 시뮬레이션 경험을 더욱 향상시키는 데 중점을 두었다. 릴리스 노트에서 언급된 "Gazebo 시뮬레이션 개선(Gazebo Simulation Improvements)"은 몇 가지 중요한 실제적 변화를 포함한다.3

가장 주목할 만한 개선은 "Gazebo와 PX4 SITL의 분리(separation of Gazebo and PX4 SITL)"이다.3 이는 개발자가 PX4 SITL 프로세스와 Gazebo 시뮬레이션 서버/클라이언트를 독립적으로 실행할 수 있게 되었음을 의미한다. 구체적으로, 

`PX4_GZ_STANDALONE=1` 환경 변수를 사용하면 Gazebo 인스턴스를 먼저 실행해 둔 상태에서 PX4 SITL 프로세스만 별도로 시작하거나 재시작할 수 있다.7

이러한 아키텍처의 '분리'는 단순한 기능 추가 이상의 의미를 갖는다. 이는 시뮬레이션 환경을 보다 모듈화된 마이크로서비스 스타일로 전환하는 근본적인 변화이다. 과거 Gazebo Classic 환경에서는 `make` 명령 하나로 PX4와 시뮬레이터가 강하게 결합된 형태로 함께 실행되었다.6 이 경우, 비행 제어 코드의 작은 변경 사항을 테스트하기 위해서도 리소스 소모가 큰 전체 시뮬레이션 환경을 재시작해야만 했다.

그러나 최신 Gazebo의 `gz-transport`는 프로세스 간 통신을 기본으로 지원하며 10, PX4 v1.14는 이를 통합했고 16, v1.15는 

`PX4_GZ_STANDALONE=1`과 같은 기능을 통해 이 분리를 공식화했다.7 이로 인해 개발자는 무거운 Gazebo 시뮬레이터를 한 번만 실행한 후, 가벼운 PX4 SITL 프로세스만 반복적으로 컴파일하고 재실행하며 코드 변경 사항을 신속하게 테스트할 수 있다. 이는 문서에서도 명시적으로 언급된 장점이며 6, 개발자의 코드 테스트 및 디버깅 속도, 즉 '개발자 생산성(Developer Velocity)'을 획기적으로 향상시키는 실질적인 이점을 제공한다.

또한, PX4 v1.15는 비선형 공기역학을 모델링하는 "고급 비행기 시뮬레이션(Advanced Plane simulation)"과 같은 새로운 시뮬레이션 모델을 지원하여 더욱 정밀하고 현실적인 테스트를 가능하게 했다.3


PX4 v1.15와 Gazebo의 호환성은 개발자가 선택하는 운영체제와 ROS 버전에 따라 크게 달라진다. 본 섹션에서는 주요 개발 환경 시나리오별로 호환성을 심층 분석하고, 각 경로의 장단점과 설정 방법을 제시한다.


이 경로는 기존 프로젝트를 유지보수하거나 특정 레거시 의존성이 필요한 경우에 고려할 수 있는, 안정적이지만 현재는 지원이 중단된 환경이다.

- **PX4 v1.15 지원:** PX4 v1.15는 여전히 Ubuntu 20.04에서 Gazebo Classic 11과의 호환성을 유지하고 있다. 관련 문서는 이 조합을 설정하는 방법을 명시적으로 안내한다.6
- **ROS 통합:** 이 환경은 ROS 1 Noetic 또는 ROS 2 Foxy와 통합될 수 있다.22 중요한 점은 ROS와 함께 Gazebo Classic을 사용할 경우, ROS 설치 지침에 따라 Gazebo를 설치해야 한다는 것이다. 이는 ROS 패키지와의 의존성 충돌을 피하기 위함이다.6
- **설정 및 실행:** 시뮬레이션 실행은 `make px4_sitl gazebo-classic_iris`와 같은 `gazebo-classic_` 접두사가 붙은 `make` 타겟을 통해 이루어진다.6
- **한계:** 가장 큰 한계는 Gazebo Classic 11과 ROS 1 Noetic이 모두 서비스 지원이 종료(EOL)되었다는 점이다.5 이는 보안 위협에 노출될 수 있으며, 최신 Gazebo에서 제공되는 새로운 물리 모델, 센서, 렌더링 기능을 사용할 수 없음을 의미한다. 따라서 신규 프로젝트에는 이 경로를 선택해서는 안 된다.


이 경로는 PX4 v1.15를 사용한 모든 신규 개발 프로젝트에 대해 공식적으로 권장되는 최상의 경로이다. 최신 기능, 장기 지원, 활발한 커뮤니티를 통해 가장 안정적이고 강력한 개발 환경을 제공한다.

- **OS/시뮬레이터 요구사항:** Ubuntu 22.04 운영체제는 최신 Gazebo 사용을 강제한다. Gazebo Classic은 이 버전에서 지원되지 않기 때문이다.6 PX4 공식 문서는 여러 최신 Gazebo 버전 중에서도 LTS 버전인 Gazebo Harmonic 사용을 구체적으로 권장하고 있다.7
- **ROS 2 Humble 통합:** 이 경로에서 가장 복잡하고 주의가 필요한 부분이다. ROS 2 Humble은 공식적으로 Gazebo Fortress를 기본 시뮬레이터로 지정하고 지원한다.1 그러나 PX4 v1.15는 Gazebo Harmonic에 최적화되어 테스트되었다.2 이 불일치는 개발 환경 구축 시 주요 혼란의 원인이 된다.
  - **충돌의 원인:** 사용자가 ROS 2 Humble을 먼저 설치하면 `ros-humble-ros-gz` 패키지가 의존성으로 Gazebo Fortress를 시스템에 설치한다. 이후 PX4의 표준 설치 스크립트(`Tools/setup/ubuntu.sh`)를 실행하면, 이 스크립트는 Gazebo Harmonic을 설치하려고 시도하여 패키지 충돌을 일으킬 수 있다.1
  - **해결 방안:** 이 문제를 해결하기 위해서는 ROS의 공식 패키지 저장소 대신 Open Robotics가 제공하는 `packages.osrfoundation.org` 저장소의 Gazebo 패키지를 사용해야 한다. 또한, ROS 2와 Gazebo Harmonic 간의 통신을 위해 `ros-humble-ros-gzharmonic` 브리지 패키지를 설치해야 한다.1 이 방법은 두 시스템 간의 호환성을 보장하지만, 표준 ROS 설치 절차와는 다르다는 점을 명확히 인지해야 한다.

이러한 버전 불일치는 PX4 개발 생태계가 ROS 주류 생태계보다 더 빠르게 최신 시뮬레이션 기술을 채택하고 있음을 보여준다. ROS LTS 릴리스(예: Humble)는 장기적인 안정성을 위해 특정 버전의 Gazebo(예: Fortress)에 고정된다.1 반면, 비행 동역학 및 제어 알고리즘 개발에 중점을 두는 PX4 프로젝트는 Harmonic에서 제공하는 향상된 물리 및 센서 모델과 같은 최신 기능의 이점을 최대한 활용하고자 한다.13 결과적으로 PX4 개발팀은 동일한 OS(Ubuntu 22.04)에 대해 더 새로운 버전의 Gazebo를 테스트하고 권장하게 되었다.2 이는 'PX4 개발자'가 단순히 'ROS 개발자'의 하위 집합이 아니며, PX4 고유의 툴체인과 환경 권장 사항을 숙지해야 하는 전문적인 역할임을 시사한다.

**표 2: ROS 2와 Gazebo 버전 조합 비교**

| ROS 2 배포판           | ROS 공식 Gazebo 버전 | PX4 권장 Gazebo 버전 | 주요 고려사항                                                |
| ---------------------- | -------------------- | -------------------- | ------------------------------------------------------------ |
| **ROS 2 Humble (LTS)** | Gazebo Fortress      | Gazebo Harmonic      | 공식 버전과 권장 버전이 달라 충돌 가능성 존재. `osrfoundation` 패키지 및 `ros-humble-ros-gzharmonic` 브리지 사용 필요. |
| **ROS 2 Jazzy (LTS)**  | Gazebo Harmonic      | Gazebo Harmonic      | 버전이 일치하여 호환성 문제가 적고 설정이 더 간편함. PX4 v1.16 이상에서 주력 경로가 될 것으로 예상됨. |


이 경로는 ROS 2 Humble의 공식 페어링을 따르려는 전문가나 특정 요구사항이 있는 개발자가 시도할 수 있는 실험적인 경로이다.

- **보고된 문제점:** 이 조합을 시도하는 사용자들이 가장 흔하게 마주치는 문제는 PX4 SITL 실행 시 `Preflight Fail: Accel Sensor 0 missing` 또는 `Gyro Sensor 0 missing`과 같은 프리플라이트 체크 실패 오류이다.2 이 오류는 시뮬레이터의 센서 데이터가 PX4 펌웨어로 제대로 전달되지 않고 있음을 나타낸다.
- **근본 원인:** 문제의 핵심은 Gazebo(Ignition)의 `gz-transport`와 ROS 2의 DDS 통신 시스템 간에 토픽을 변환해주는 `ros_gz_bridge`에 있다. ROS 2 Humble과 함께 배포되는 사전 빌드된 `ros_gz_bridge`는 Gazebo Fortress 용으로 만들어졌지만, PX4가 사용하는 특정 센서 토픽의 형식이나 통신 방식과 완벽하게 호환되지 않을 수 있다.
- **우회 방안:** 커뮤니티에서 검증된 해결책은 `ros_gz_bridge`를 소스 코드에서 직접 빌드하는 것이다.2 이를 통해 현재 시스템에 설치된 정확한 라이브러리와 메시지 정의에 맞춰 브리지를 컴파일함으로써 통신 불일치 문제를 해결할 수 있다. 이 방법은 성공률이 높지만, 추가적인 빌드 과정과 의존성 관리가 필요하다.


성공적인 시뮬레이션 환경 구축을 위해서는 올바른 버전 조합을 선택하는 것만큼이나, 설정 과정에서 발생할 수 있는 일반적인 오류들을 사전에 방지하고 신속하게 해결하는 것이 중요하다. 본 섹션에서는 개발자가 흔히 겪는 문제들과 그 해결책을 체계적으로 정리하여 제공한다.


첫 `make` 명령을 실행하기 전에 다음 사항들을 점검하면 많은 시간을 절약할 수 있다.

- **Git 저장소 클론:** PX4-Autopilot 저장소는 반드시 `--recursive` 플래그와 함께 클론해야 한다: `git clone https://github.com/PX4/PX4-Autopilot.git --recursive`. 만약 이 플래그를 빠뜨렸다면, 저장소 디렉토리 내에서 `git submodule update --init --recursive` 명령을 실행하여 누락된 하위 모듈들을 받아와야 한다. 시뮬레이션 모델(`PX4-SITL_gazebo-classic`, `PX4-gazebo-models` 등)은 모두 하위 모듈로 관리되므로, 이 과정이 누락되면 빌드 시 `ninja: error: unknown target 'gz_x500'`과 같은 대상을 찾을 수 없다는 오류가 발생한다.25
- **컴파일러 환경 확인:** Anaconda나 Miniconda와 같은 Python 가상 환경은 시스템의 `PATH` 환경 변수 앞부분에 자체 경로를 추가하여, 시스템 기본 컴파일러(GCC) 대신 자체적으로 포함된 컴파일러를 사용하게 만들 수 있다. 이는 예기치 않은 컴파일 오류나 런타임 오류의 원인이 될 수 있다. 이 문제를 방지하기 위해, PX4 빌드 전 터미널에서 `export CC=gcc`와 `export CXX=g++` 명령을 실행하여 시스템의 기본 GCC 툴체인을 사용하도록 명시적으로 지정하는 것이 매우 효과적인 해결책이다.25
- **PX4 설치 스크립트 실행:** PX4 소스 코드 내의 `Tools/setup/ubuntu.sh` 스크립트는 시뮬레이션을 포함한 PX4 빌드에 필요한 모든 의존성 패키지를 설치하는 역할을 한다. 저장소를 클론한 후 가장 먼저 이 스크립트를 실행해야 한다.25


아래 표는 PX4 v1.15 SITL 환경 설정 및 실행 중 발생하는 가장 빈번한 문제들과 그 해결책을 정리한 것이다. 이는 커뮤니티 포럼과 GitHub 이슈 트래커에 흩어져 있는 집단 지성을 체계화한 빠른 진단 도구이다.

**표 3: PX4 v1.15 SITL 문제 해결 가이드**

| 증상 / 오류 메시지                                           | 유력한 원인                                                  | 권장 해결책 / 우회 방안                                      | 관련 자료 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | --------- |
| `ninja: error: unknown target 'gz_x500'`                     | 1. Git 하위 모듈(submodule)이 초기화되지 않음.2. Conda 환경의 컴파일러가 시스템 GCC와 충돌. | 1. git submodule update --init --recursive 실행.2. export CC=gcc CXX=g++ 실행 후 빌드. | 25        |
| `Preflight Fail: Accel Sensor 0 missing`                     | 권장되지 않는 Gazebo 버전(예: Fortress) 사용으로 인한 `ros_gz_bridge` 통신 오류. | 1. (권장) Gazebo Harmonic으로 전환.2. (우회) ros_gz_bridge를 소스에서 직접 빌드. | 2         |
| Protobuf 버전 충돌 오류                                      | 시스템에 여러 버전의 Protocol Buffers가 설치되어 충돌 발생. (WSL2 환경에서 흔함) | `protoc --version`으로 현재 활성화된 버전을 확인하고, 필요한 버전에 맞게 시스템 환경을 정리. (예: `apt`로 설치된 버전과 `pip`로 설치된 버전 간 충돌 확인) | 26        |
| 시뮬레이션은 시작되나 기체가 보이지 않음                     | `PX4_SIM_MODEL`과 `PX4_GZ_MODEL_NAME` 환경 변수가 잘못 설정되었거나 충돌. 이 두 변수는 상호 배타적임. | 사용 목적에 맞는 변수 하나만 설정했는지 확인. (예: `make` 타겟에 모델이 명시된 경우, 이 변수들을 설정할 필요 없음) | 7         |
| `[health_and_arming_checks] Preflight Fail: ekf2 missing data` | 시뮬레이션된 기체의 위치가 월드 원점에서 너무 멀리 떨어져 EKF2가 초기 위치를 수신하지 못함. | `PX4_GZ_MODEL_POSE`를 사용하여 기체를 월드 원점(`0,0,0`) 근처에 스폰시키거나, 월드 파일의 원점 설정을 확인. | 28        |


PX4 시뮬레이션을 효과적으로 제어하기 위해 사용되는 주요 명령어와 환경 변수에 대한 이해는 필수적이다.

- **실행 명령어:** PX4 v1.14부터 시뮬레이터 종류에 따라 `make` 타겟의 접두사가 명확히 구분되었다.
  - **최신 Gazebo:** `gz_` 접두사 사용 (예: `make px4_sitl gz_x500`).7
  - **Gazebo Classic:** `gazebo-classic_` 접두사 사용 (예: `make px4_sitl gazebo-classic_iris`).6 이 명명 규칙은 어떤 시뮬레이션 백엔드가 사용되는지를 명확히 하여 혼동을 줄여준다.16
- **주요 환경 변수:**
  - `PX4_SIMULATOR=gz`: 사용할 시뮬레이터가 최신 Gazebo임을 명시한다. 보통 `make` 타겟에 의해 자동으로 설정된다.7
  - `PX4_GZ_STANDALONE=1`: 이미 실행 중인 Gazebo 시뮬레이션에 PX4 SITL 인스턴스를 연결하고자 할 때 사용한다. 이는 Gazebo를 재시작하지 않고 PX4 코드만 빠르게 테스트할 때 매우 유용하다.7
  - `PX4_SIM_MODEL`: `make` 명령을 사용하지 않고 PX4 SITL을 직접 실행할 때, 스폰할 기체 모델을 지정한다 (예: `gz_x500`). `PX4_GZ_MODEL_NAME`과 함께 사용할 수 없다.21
  - `PX4_GZ_MODEL_NAME`: 이미 Gazebo 월드에 존재하는 특정 이름의 모델에 PX4 SITL 인스턴스를 바인딩(연결)하고자 할 때 사용한다. `PX4_SIM_MODEL`과 함께 사용할 수 없다.7
  - `PX4_GZ_WORLD`: 시뮬레이션을 시작할 때 로드할 월드 파일을 지정한다. 이미 실행 중인 시뮬레이션에는 영향을 주지 않는다.21
  - `PX4_GZ_MODEL_POSE`: 기체가 스폰될 초기 위치와 자세를 `x,y,z,roll,pitch,yaw` 형식으로 지정한다.21


본 보고서의 심층 분석을 바탕으로, PX4 v1.15를 사용하는 개발자를 위한 전략적 권고와 향후 생태계의 발전 방향을 제시한다.


모든 신규 드론 개발 프로젝트는 **Ubuntu 22.04 LTS + ROS 2 Humble + Gazebo Harmonic + PX4 v1.15** 기술 스택을 채택할 것을 강력히 권고한다. 이 조합을 추천하는 이유는 다음과 같다.

- **성능 및 기능:** Gazebo Harmonic은 최신 물리 엔진, 향상된 렌더링, 다양한 센서 모델을 제공하여 가장 현실적이고 정밀한 시뮬레이션이 가능하다.13
- **장기 지원(LTS):** 스택의 핵심 요소들(Ubuntu 22.04, ROS 2 Humble, Gazebo Harmonic)이 모두 LTS 버전이므로, 장기간 안정적인 기술 지원과 보안 업데이트를 보장받을 수 있다.1
- **미래 호환성:** 이 경로는 PX4와 ROS 2 생태계의 주력 발전 방향과 일치하므로, 향후 릴리스될 버전(예: ROS 2 Jazzy)으로의 마이그레이션이 상대적으로 용이하다.


현재 Ubuntu 20.04와 Gazebo Classic 기반으로 진행 중인 프로젝트는 계획적인 마이그레이션 전략을 수립해야 한다. Gazebo Classic과 ROS 1 Noetic의 서비스 지원이 공식적으로 종료된 현시점에서, 레거시 스택에 대한 신규 기능 개발은 중단하고 가용한 리소스를 최신 스택으로의 포팅 작업에 할당하는 것이 바람직하다.5 이는 잠재적인 보안 위협을 회피하고, 프로젝트의 기술적 생명력을 연장하며, 최신 시뮬레이션 기술의 이점을 활용하기 위한 필수적인 조치이다.


PX4와 Gazebo 시뮬레이션 생태계는 복잡성을 내포하고 있지만, 그 발전 방향은 명확하며 더욱 강력하고 유연한 개발 환경으로 나아가고 있다.

- **차세대 조합:** ROS 2 Jazzy가 Gazebo Harmonic과 공식적으로 페어링되면서 1, PX4와의 호환성 문제는 점차 해소될 것이다. 향후 ROS 2 Kilted와 Gazebo Ionic의 조합은 더욱 향상된 기능을 제공할 것으로 기대된다.1
- **지속적인 개발:** PX4-Autopilot 및 관련 모델 저장소의 활발한 풀 리퀘스트 활동은 프로젝트가 건강하게 발전하고 있음을 보여준다. 새로운 기체, 센서, 월드 모델이 지속적으로 추가되고 있으며 29, 이는 개발자들이 더욱 다양한 시나리오를 테스트할 수 있게 해준다.

결론적으로, PX4 v1.15와 Gazebo의 호환성 환경은 과도기적 복잡성을 일부 가지고 있으나, 이는 더 나은 시뮬레이션 기술로 나아가는 과정에서 발생하는 자연스러운 현상이다. 본 보고서에서 제시된 가이드를 따르면 개발자들은 이러한 복잡성을 극복하고, 강력하고 효율적인 시뮬레이션 기반의 첨단 드론 시스템을 성공적으로 개발할 수 있을 것이다.


1. Installing Gazebo with ROS - Gazebo ionic documentation, accessed July 2, 2025, https://gazebosim.org/docs/latest/ros_installation/
2. PX4 simulation in gazebo fortress support, accessed July 2, 2025, https://discuss.px4.io/t/px4-simulation-in-gazebo-fortress-support/45249
3. PX4 Autopilot Release v1.15: What You Need To Know, accessed July 2, 2025, https://px4.io/px4-autopilot-release-v1-15-what-you-need-to-know/
4. Gazebo Garden officially end-of-life [x-post Gazebo Sim Community] - ROS Discourse, accessed July 2, 2025, https://discourse.ros.org/t/gazebo-garden-officially-end-of-life-x-post-gazebo-sim-community/41044
5. Gazebo Classic 11 has reached end-of-life [x-post Gazebo Sim Community] - ROS Discourse, accessed July 2, 2025, https://discourse.ros.org/t/gazebo-classic-11-has-reached-end-of-life-x-post-gazebo-sim-community/41852
6. Gazebo Classic Simulation | PX4 Guide (v1.15), accessed July 2, 2025, https://docs.px4.io/v1.15/en/sim_gazebo_classic/index.html
7. Gazebo Simulation | PX4 Guide (main), accessed July 2, 2025, https://docs.px4.io/main/en/sim_gazebo_gz/
8. Gazebo (simulator) - Wikipedia, accessed July 2, 2025, https://en.wikipedia.org/wiki/Gazebo_(simulator)
9. Blog : Gazebo 11.0.0 release, accessed July 2, 2025, https://classic.gazebosim.org/blog/gazebo11
10. Blog : Gazebo 9.0.0 Release, accessed July 2, 2025, https://classic.gazebosim.org/blog/gazebo9
11. Gazebo Classic robotics simulator reaches end of life, accessed July 2, 2025, https://www.therobotreport.com/gazebo-classic-robotics-simulator-reaches-end-of-life/
12. Gazebo garden documentation, accessed July 2, 2025, https://gazebosim.org/docs/garden/install/
13. Gazebo Harmonic Released! - Open Robotics, accessed July 2, 2025, https://www.openrobotics.org/blog/2023/9/26/gazebo-harmonic-released
14. Gazebo Releases - Gazebo ionic documentation, accessed July 2, 2025, https://gazebosim.org/docs/latest/releases/
15. Gazebo Garden Released! - Open Robotics, accessed July 2, 2025, https://www.openrobotics.org/blog/2022/10/3/gazebo-garden-released
16. PX4-Autopilot v1.14 Release Notes | PX4 Guide (v1.15), accessed July 2, 2025, https://docs.px4.io/v1.15/en/releases/1.14.html
17. PX4-Autopilot v1.14 Release Notes, accessed July 2, 2025, https://px-4.com/v1.14/en/releases/1.14.html
18. PX4-Autopilot v1.14 Release Notes | PX4 Guide (main), accessed July 2, 2025, https://docs.px4.io/main/en/releases/1.14.html
19. Gazebo Classic Simulation | PX4 Guide (main), accessed July 2, 2025, https://docs.px4.io/main/en/sim_gazebo_classic/
20. Gazebo Vehicles | PX4 Guide (v1.15), accessed July 2, 2025, https://docs.px4.io/v1.15/en/sim_gazebo_gz/vehicles.html
21. Gazebo Simulation | PX4 Guide (v1.15), accessed July 2, 2025, https://docs.px4.io/v1.15/en/sim_gazebo_gz/
22. ROS 2 User Guide | PX4 Guide (v1.15), accessed July 2, 2025, https://docs.px4.io/v1.15/en/ros2/user_guide.html
23. ROS2 Humble, Gazebo Harmonic, PX4, QGroundControl, Micro XRCE-DDS Agent & Client Installation | by Erdem | Medium, accessed July 2, 2025, https://medium.com/@erdem.ku.3.14/ros2-humble-gazebo-harmonic-px4-ve-micro-xrce-dds-agent-client-installation-aad32d8f5669
24. [Bug] PX4 (v1.16.0-alpha2) + Ubuntu 24.04 + ROS2 Jazzy + Gazebo Harmonic leads to error #24159 - GitHub, accessed July 2, 2025, https://github.com/PX4/PX4-Autopilot/issues/24159
25. [Bug] ninja: error: unknown target 'gz_x500' / Issue #22728 / PX4/PX4-Autopilot - GitHub, accessed July 2, 2025, https://github.com/PX4/PX4-Autopilot/issues/22728
26. [Bug] About compiler issue / Issue #22695 / PX4/PX4-Autopilot - GitHub, accessed July 2, 2025, https://github.com/PX4/PX4-Autopilot/issues/22695
27. Issue running PX4 SITL in a container - MAVLink, accessed July 2, 2025, https://discuss.px4.io/t/issue-running-px4-sitl-in-a-container/46059
28. How to change PX4 Gazebo (last version) home point when launching a simulation, accessed July 2, 2025, https://discuss.px4.io/t/how-to-change-px4-gazebo-last-version-home-point-when-launching-a-simulation/39497
29. Pull requests 6 - PX4/PX4-gazebo-models - GitHub, accessed July 2, 2025, https://github.com/PX4/PX4-gazebo-models/pulls
30. Pull requests / PX4/PX4-Autopilot - GitHub, accessed July 2, 2025, https://github.com/PX4/PX4-Autopilot/pulls


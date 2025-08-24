[Webots](./index.md)
# Webots 로봇 시뮬레이터

Webots 로봇 시뮬레이터에 대한 포괄적이고 심층적인 분석을 제공한다. Webots는 성숙하고 안정적이며 전문가 수준의 로봇 시뮬레이터로, 사용자 친화적인 그래픽 사용자 인터페이스(GUI), 뛰어난 리소스 효율성, 그리고 다중 프로그래밍 언어에 대한 강력한 지원으로 차별화된다. 1996년 스위스 로잔 연방 공과대학교(EPFL)에서 학술 연구용으로 시작하여 1998년부터 Cyberbotics사에 의해 상용 소프트웨어로 개발되다가, 2018년 아파치 2.0 라이선스로 전환하며 오픈 소스 생태계에 합류했다. 이러한 독특한 발전 경로는 20년간의 상용 개발을 통해 축적된 안정성과 완성도를 오픈 소스의 유연성 및 접근성과 결합시키는 결과를 낳았다.

**주요 분석 결과:**

- **성능 및 효율성:** 정량적 벤치마크 분석 결과, Webots는 장시간 시뮬레이션에서의 안정성과 리소스 효율성(낮은 CPU 및 메모리 사용량) 측면에서 최상위 성능을 보인다.1 이는 특수 하드웨어가 아닌 일반적인 컴퓨팅 환경에서도 복잡한 실험을 안정적으로 수행할 수 있음을 의미한다.
- **사용자 경험:** 오랜 기간 ROS 생태계를 주도해 온 Gazebo와 비교했을 때, Webots는 초보자 친화성, 체계적인 문서화, 그리고 세련된 GUI 측면에서 뚜렷한 우위를 점하고 있다.3 이는 개발 과정의 마찰을 줄여 생산성을 높이는 중요한 요소로 작용한다.
- **적용 분야:** Webots는 학술 연구, 교육, 산업계의 신속한 프로토타이핑 등 광범위한 분야에서 활용 가치가 높다. 특히, 최근에는 심층 강화학습(Deep Reinforcement Learning, DRL) 분야에서의 활용도가 증가하고 있으며, 시뮬레이션 결과를 실제 로봇에 이식하는 Sim-to-Real 전환을 위한 성숙하고 강력한 프레임워크를 제공한다.4

**핵심 권장 사항:**

안정성, 크로스플랫폼 호환성, 사용 편의성, 그리고 리소스 효율성을 우선시하는 연구자 및 개발자에게 Webots는 매우 강력한 선택지가 될 수 있다. 특히 로봇 시뮬레이션에 새로 입문하거나 전통적인 ROS 중심의 파이프라인에서 벗어난 프로젝트를 진행하는 경우, Webots는 Gazebo의 매력적인 대안으로서 적극적으로 고려될べき이다.


Webots의 설계 철학과 기술적 특성을 이해하기 위해서는 그 역사적 발전 과정과 핵심 기술 스택에 대한 깊이 있는 분석이 선행되어야 한다. 본 섹션에서는 Webots의 기원부터 오픈 소스로의 전환 과정, 그리고 시뮬레이터를 구성하는 기술적 기반을 상세히 다룬다.


- **학문적 뿌리:** Webots 프로젝트는 1996년, 세계적인 명성을 자랑하는 스위스 로잔 연방 공과대학교(EPFL)의 올리비에 미셸(Olivier Michel) 박사에 의해 시작되었다.6 이는 Webots가 태생적으로 엄격한 학술 연구 환경에 기반을 두고 있음을 시사한다.
- **상용화 및 전문화:** 1998년, Cyberbotics Ltd.가 설립되어 Webots를 핵심 제품으로 상용화하고 개발을 주도했다.6 이후 20년간 Webots는 독점 라이선스 소프트웨어로서 전문가용으로 지속적으로 유지보수 및 개선되었다.9
- **2018년의 오픈 소스 전환:** 2018년 12월, Cyberbotics는 Webots를 허용적인 아파치 2.0(Apache 2.0) 라이선스로 공개하는 중대한 결정을 내렸다.6 이 전환을 통해 Webots의 모든 기능이 커뮤니티에 무료로 제공되었으며, 다른 오픈 소스 시뮬레이터와 직접 경쟁하는 발판을 마련했다. 이러한 전략적 전환은 Webots가 2002년부터 오픈 소스였던 Gazebo가 주도하는 ROS(Robot Operating System) 커뮤니티의 거대한 흐름에 합류하기 위한 필연적인 선택이었다. 독점 소프트웨어라는 장벽은 Webots가 광범위한 사용자층, 특히 학계와 ROS 커뮤니티로 확산되는 데 큰 제약이었기 때문이다. 이 결정은 Webots에 독특한 정체성을 부여했는데, 20년간의 상업적 개발을 통해 축적된 안정성, 세련미, 그리고 포괄적인 문서화의 장점과 오픈 소스 프로젝트의 유연성, 커뮤니티 기반 협력, 무료라는 이점을 모두 갖추게 된 것이다.3
- **현대의 비즈니스 모델:** Cyberbotics는 현재 산업 및 학술 고객을 대상으로 유료 기술 지원, 교육, 컨설팅 서비스를 제공함으로써 수익을 창출하고, 이를 통해 Webots의 지속적인 개발을 이끌고 있다.7


Webots의 핵심은 현대적인 GUI, 맞춤형 물리 엔진, 그리고 고성능 렌더링 엔진이라는 세 가지 기술적 기둥의 조합으로 이루어져 있다.8

- **그래픽 사용자 인터페이스 (GUI):** 현대적이고 크로스플랫폼을 지원하는 **Qt 프레임워크**를 기반으로 구축되었다.8 이 덕분에 Windows, macOS, Linux 등 다양한 운영체제에서 일관되고 세련된 사용자 경험을 제공한다.12 GUI는 크게 3D 뷰(3D View), 씬 트리(Scene Tree), 텍스트 편집기(Text Editor), 콘솔(Console)의 네 가지 창으로 구성된다.13
- **물리 엔진:** 충돌 감지 및 강체 동역학 시뮬레이션을 위해 **Open Dynamics Engine (ODE)의 자체 포크(fork) 버전**을 사용한다.6 표준 ODE를 그대로 사용하지 않고 포크 버전을 유지하는 것은 중요한 공학적 결정이다. 이는 표준 ODE가 전문적인 로봇 시뮬레이션에 요구되는 결정론적(deterministic)이고 강건한(robust) 동작을 보장하기에 충분하지 않았음을 시사한다. Cyberbotics는 이를 통해 질량, 관절, 마찰, 스프링/댐핑 상수와 같은 핵심 물리 속성뿐만 아니라 간단한 유체 역학까지 시뮬레이션할 수 있도록 ODE를 맞춤화하고 최적화했다.6 이러한 제어권 확보는 과학 실험이나 산업 검증에서 반복성과 신뢰성을 보장하는 데 결정적인 요소로 작용한다.
- **렌더링 엔진:** **`wren`**이라는 이름의 전용 렌더링 엔진을 사용하며, 이는 **OpenGL 3.3**에 기반한다.8 이를 통해 고품질의 3D 시각화와 현대적인 그래픽 효과를 지원한다.


- **계층적 씬 트리:** Webots 월드(`.wbt` 파일)는 VRML97 표준과 유사하게 "노드(Node)"들의 계층적 트리 구조로 구성된다. 이 구조 덕분에 로봇이 관절을 포함하고, 관절이 다시 센서를 포함하는 것과 같이 복잡한 객체를 단순한 객체들의 조합으로 쉽게 구성할 수 있다.6

- **VRML 기반 파일 형식:** 월드 파일(`.wbt`)은 플랫폼 간 호환성을 가지며, 사람이 읽고 편집할 수 있는 VRML(Virtual Reality Modeling Language)에 기반하여 직접적인 텍스트 편집과 상호 운용성을 보장한다.6

- **PROTO: 모듈화 및 재사용 가능한 자산:** Webots는 **`PROTO`**라는 강력한 시스템(`.proto` 파일)을 통해 재사용 가능하고 매개변수화할 수 있는 객체, 센서, 액추에이터, 심지어 로봇 전체를 정의할 수 있게 한다.10 이는 모듈성을 촉진하고 복잡한 씬의 생성을 가속화한다. 

  `PROTO` 파일은 Blender와 같은 3D CAD 소프트웨어나 URDF 파일로부터 가져올 수도 있다.8

| 속성                     | 사양                                                         | 출처 |
| ------------------------ | ------------------------------------------------------------ | ---- |
| **최초 출시**            | 1996                                                         | 6    |
| **오픈 소스 전환**       | 2018년 12월                                                  | 6    |
| **라이선스**             | Apache 2.0                                                   | 6    |
| **개발사**               | Cyberbotics Ltd.                                             | 6    |
| **핵심 구성요소**        | GUI (Qt), 물리 엔진 (ODE 포크), 렌더링 엔진 (wren/OpenGL 3.3) | 8    |
| **지원 운영체제**        | Windows, macOS, Linux                                        | 6    |
| **주요 프로그래밍 언어** | C, C++, Python, Java, MATLAB                                 | 8    |
| **월드 파일 형식**       | `.wbt` (VRML 기반)                                           | 6    |


Webots에서 로봇을 프로그래밍하고 제어하는 아키텍처는 다중 언어 지원과 명확한 API 구조를 통해 개발자에게 높은 유연성과 안정성을 제공한다. 본 섹션에서는 컨트롤러 아키텍처, API 구현 방식, 그리고 외부 시스템과의 연동성을 분석한다.


- **분리된 프로세스 모델:** 각 로봇 컨트롤러는 Webots에 의해 생성된 별도의 자식 프로세스(child process)로 실행된다.15 이러한 견고한 설계는 특정 컨트롤러의 충돌이 메인 시뮬레이션이나 다른 컨트롤러에 영향을 미치지 않도록 보장하여 전체 시스템의 안정성을 크게 향상시킨다.
- **컨트롤러-로봇 연결:** 월드 내의 로봇 모델과 그 동작을 정의하는 코드는 `.wbt` 파일의 로봇 노드(Robot node)에 있는 `controller` 필드를 통해 연결된다.14 여러 로봇이 동일한 컨트롤러 코드를 공유할 수 있지만, 각각은 독립된 프로세스에서 실행된다.
- **슈퍼바이저 컨트롤러:** 로봇 노드의 `supervisor` 필드를 `TRUE`로 설정하면 해당 로봇은 특별한 권한을 가진 슈퍼바이저 컨트롤러로 지정된다.14 슈퍼바이저는 일반 로봇 컨트롤러가 접근할 수 없는 기능들을 수행할 수 있는데, 예를 들어 시뮬레이션 내 모든 객체의 위치를 읽거나 수정하고, 시뮬레이션을 시작하거나 정지하며, 동영상을 녹화하는 등의 작업이 가능하다. 이러한 기능은 축구 심판과 같은 복잡한 실험을 조율하거나 17, 강화학습에서 훈련 루프를 관리하는 데 필수적이다.13 슈퍼바이저의 존재는 Webots 아키텍처가 단순한 시뮬레이션을 넘어, 정교한 연구 패러다임을 지원하도록 설계되었음을 보여주는 핵심적인 증거이다. 강화학습과 같은 분야에서 외부 학습 알고리즘은 환경을 초기화하고, 보상을 계산하기 위해 객체의 상태를 정확히 알아야 하는데, 슈퍼바이저 API는 바로 이러한 요구사항을 완벽하게 충족시키는 통로를 제공한다.


- **지원 언어:** Webots는 **C, C++, Python, Java, MATLAB**을 위한 네이티브 API를 제공한다.8 이러한 유연성 덕분에 개발자는 자신에게 가장 익숙한 환경에서 작업할 수 있다. Webots 소프트웨어 자체는 주로 C++ (54.2%)와 C (24.7%)로 작성되었다.7

- **핵심 API 구조:**

  - 컨트롤러의 기본적인 제어 루프는 `wb_robot_step(time_step)` 함수를 호출하는 `while` 또는 `for` 반복문으로 구성된다. 이 함수는 컨트롤러의 심장 박동과 같아서, 시뮬레이터와 데이터를 동기화하고 시뮬레이션 시간을 `time_step`만큼 진행시키는 역할을 한다.18

  - C 언어에서는 `wb_robot_init()`과 `wb_robot_cleanup()` 함수를 통해 초기화와 정리를 명시적으로 수행한다. Python, C++, Java와 같은 객체 지향 언어에서는 이러한 과정이 클래스의 생성자(constructor)와 소멸자(destructor)를 통해 추상화된다.18

  - **장치 접근:** 센서를 사용하려면 먼저 `wb_*_enable(sampling_period)` 함수를 호출하여 활성화해야 한다. `sampling_period`(밀리초 단위)는 센서 데이터의 갱신 주기를 결정한다. 이는 실제 세계의 느린 장치를 모델링하거나, 카메라처럼 CPU 소모가 큰 센서의 부하를 줄여 시뮬레이션 성능을 향상시키는 데 사용될 수 있다.18 데이터는 

    `wb_*_get_value()` 형태의 함수를 통해 검색된다.

- **사용자 입력:** `Keyboard` API는 컨트롤러가 직접 키보드 입력을 읽을 수 있게 하여, 시뮬레이션 중에 로봇을 실시간으로 조작하는 상호작용을 가능하게 한다.19

다양한 언어를 지원하고 컨트롤러를 독립 프로세스로 실행하는 아키텍처는 진입 장벽을 낮추고 시스템의 견고성을 높이는 이중의 효과를 가진다. AI/ML 연구자(주로 Python 사용)부터 제어 시스템 엔지니어(MATLAB, C++)까지 다양한 기술 스택을 가진 연구팀이 단일 플랫폼에서 협업할 수 있다. 이러한 유연성과 안정성의 조합은 Webots를 다학제적 팀에게 매우 실용적이고 합리적인 선택으로 만든다.


- **ROS/ROS2 통합:** Webots는 ROS 및 ROS2를 완벽하게 지원하여, ROS 생태계 내에서 주요 시뮬레이션 환경으로 사용될 수 있다.3 Vulcanexus와 같은 프로젝트는 Webots를 완전한 ROS2 소프트웨어 스택의 일부로 포함하여 제공하기도 한다.8
- **외부 소프트웨어 통신:** 컨트롤러는 TCP/IP를 통해 제3자 소프트웨어와 인터페이스할 수 있어, 복잡한 공동 시뮬레이션(co-simulation) 설정이 가능하다.20
- **환경 구성:** `runtime.ini` 파일은 컨트롤러를 위한 환경 변수를 설정하는 강력한 메커니즘을 제공한다. 특히 Windows, macOS, Linux 등 플랫폼별로 다른 구성을 지정할 수 있어 복잡한 라이브러리 경로와 의존성을 관리하는 데 필수적이다.18

| 기능                   | C++ 예시                                                     | Python 예시                        | Java 예시                                             |
| ---------------------- | ------------------------------------------------------------ | ---------------------------------- | ----------------------------------------------------- |
| **헤더/모듈 임포트**   | `#include <webots/Robot.hpp>`                                | `from controller import Robot`     | `import com.cyberbotics.webots.controller.Robot;`     |
| **로봇 인스턴스 생성** | `webots::Robot *robot = new webots::Robot();`                | `robot = Robot()`                  | `Robot robot = new Robot();`                          |
| **메인 루프**          | `while (robot->step(timeStep)!= -1) {... }`                  | `while robot.step(timeStep)!= -1:` | `while (robot.step(timeStep)!= -1) {... }`            |
| **장치 핸들 획득**     | `webots::DistanceSensor *ds = robot->getDistanceSensor("ds0");` | `ds = robot.getDevice("ds0")`      | `DistanceSensor ds = robot.getDistanceSensor("ds0");` |
| **센서 활성화**        | `ds->enable(timeStep);`                                      | `ds.enable(timeStep)`              | `ds.enable(timeStep);`                                |
| **센서 값 읽기**       | `double value = ds->getValue();`                             | `value = ds.getValue()`            | `double value = ds.getValue();`                       |


Webots의 진정한 가치는 물리적 현상을 시뮬레이션하는 능력과 이를 뒷받침하는 방대한 자산 라이브러리에 있다. 본 섹션에서는 Webots가 제공하는 물리 시뮬레이션의 깊이, 센서와 액추에이터의 충실도, 그리고 즉시 사용 가능한 환경 및 자산 생태계를 상세히 분석한다.


- **강체 속성:** 사용자는 모든 객체(Solid 노드)에 대해 질량, 무게 중심, 마찰 계수, 스프링/댐핑 상수 등 상세한 물리적 속성을 정의할 수 있다.6 `Physics` 노드가 없는 객체는 정적이며 운동학적으로만 처리된다.13
- **관절 및 다관절 시스템:** `HingeJoint`, `SliderJoint`, `BallJoint` 등 다양한 종류의 관절을 사용하여 로봇 팔이나 다족 보행 로봇과 같은 다관절 시스템을 정교하게 모델링할 수 있다.13
- **충돌 감지:** ODE 엔진이 씬 내의 모든 물리 객체 간의 충돌을 감지하고 처리한다.6
- **유체 역학:** Webots는 간단한 유체 역학을 지원하여, 항력과 같은 요소를 고려한 수중 자율 이동체(AUV)나 비행 드론의 시뮬레이션을 가능하게 한다.4
- **물리 플러그인:** 고급 사용자는 C/C++로 사용자 정의 물리 플러그인을 작성하여 기본 물리 엔진의 동작을 수정하거나 확장할 수 있다.10


- **광범위한 센서 라이브러리:** Webots는 현대 로보틱스에 필수적인 방대한 종류의 센서 모델을 기본으로 제공하며, 이들은 자유롭게 수정 가능하다. **라이다(Lidar), 레이더(Radar), 카메라, 근접/거리 센서**와 같은 외부 환경 인식 센서뿐만 아니라, **GPS, 가속도계, 자이로스코프, 관성 측정 장치(IMU)**와 같은 자기 상태 인식 센서까지 모두 포함된다.6
- **다양한 액추에이터 라이브러리:** 회전 및 선형 서보 모터, 그리퍼, 시각적 피드백을 위한 LED, 통신 시뮬레이션을 위한 송수신기 등 광범위한 액추에이터를 사용할 수 있다.6
- **매개변수화:** 각 센서와 액추에이터 모델은 고도로 매개변수화되어 있어 실제 하드웨어의 특성과 한계를 현실적으로 시뮬레이션할 수 있다. 예를 들어, `DistanceSensor`의 동작은 거리와 센서 값을 매핑하는 조회 테이블(lookup table)로 정의되어 비선형적인 반응을 모델링할 수 있다.23


- **기본 제공 환경:** Webots는 가구가 비치된 아파트, 주방, 공장 등 즉시 사용 가능한 풍부한 예제 환경을 포함하고 있다.24 이러한 환경들은 모듈화된 `PROTO` 객체들로 구성되어 있어 쉽게 변형하고 확장할 수 있다.
- **상호작용 가능한 객체:** 제공되는 많은 객체들은 상호작용이 가능하다. 예를 들어, 로봇이 캐비닛 문을 열거나, 밸브 핸들을 돌리거나, 도구를 집는 등의 복잡한 조작 작업을 프로그래밍하고 시뮬레이션할 수 있다.24
- **가져오기 기능:** 사용자는 빈 공간에서 새로운 월드를 구축하거나 외부 소스로부터 자산을 가져올 수 있다. 3D CAD 소프트웨어(VRML 또는 STL/DAE 변환)나 로봇 모델을 위한 URDF 파일 가져오기를 지원한다.6 또한, OpenStreetMap에서 지도를 가져와 자율 주행 시뮬레이션을 위한 현실적인 대규모 환경을 생성하는 것도 가능하다.8
- **온라인 공유 및 경쟁:** Cyberbotics는 `webots.cloud`와 `robotbenchmark.net`이라는 웹 플랫폼을 운영한다.6 사용자들은 이곳에서 WebGL 스트리밍 기술을 통해 웹 브라우저에서 직접 시뮬레이션을 실행하고, 자신의 모델과 시뮬레이션을 공유하며, 경쟁에 참여할 수 있다.

Webots의 자산 생태계는 신속한 프로토타이핑을 지원하고 복잡한 시뮬레이션의 진입 장벽을 낮추기 위해 의도적으로 설계되었다. 방대한 센서, 액추에이터, 상호작용 객체, 그리고 완성된 환경 라이브러리는 3D 모델링 전문가가 아니더라도 누구나 정교한 시뮬레이션을 빠르게 구성할 수 있게 한다.6 이는 교육 현장이나 애자일 개발 방식의 산업계에서 "첫 실험까지 걸리는 시간"을 극적으로 단축시키는 핵심 요소이다.

더 나아가, `PROTO` 파일의 메타데이터 시스템은 이러한 자산 생태계를 확장 가능하고 검색 가능한 라이브러리로 만드는 기반이 된다. `PROTO` 설계 가이드라인에는 `license`, `documentation url`, `tags`(`deprecated` 등), `keywords`(`robot/wheeled` 등)와 같은 구조화된 메타데이터 스키마가 포함되어 있다.25 이는 단순한 파일 정리를 넘어, 

`webots.cloud`와 같은 웹 기반 자산 데이터베이스의 백엔드 데이터 모델 역할을 한다. Cyberbotics는 개발자들이 이러한 메타데이터를 추가하도록 장려함으로써, 방대하지만 때로는 정돈되지 않은 Gazebo의 모델 저장소와 경쟁할 수 있는, 잘 정리된 커뮤니티 기여 자산 라이브러리를 구축하는 장기적인 전략을 실행하고 있다.


Webots는 그 유연성과 안정성을 바탕으로 학계, 산업계, 교육 현장 전반에 걸쳐 광범위한 영향력을 미치고 있다. 본 섹션에서는 구체적인 프로젝트와 연구 사례를 통해 Webots가 각 분야에서 어떻게 활용되고 있는지 심도 있게 조명한다.


- **핵심 연구 분야:** Webots는 **군집 지능(다개체 로봇 시뮬레이션), 로봇 이동(다족 및 차륜 로봇), 인공 생명, 진화 로보틱스, 적응 행동** 등 수많은 학술 연구 분야에서 핵심 도구로 자리 잡았다.6
- **구체적인 연구 사례:**
  - **수중 로봇 군집:** 150대의 자율 수중 이동체(AUV)로 구성된 군집을 위한 퍼지 협력 측위 프레임워크 연구에서, Webots의 유체 역학 및 수중 음향 통신 시뮬레이션 기능이 알고리즘 검증에 활용되었다.4
  - **보행 패턴 전이:** 다족 보행 로봇의 보행 패턴을 단순한 지형에서 복잡한 지형으로 전이시키는 연구에서, Webots 시뮬레이션을 통해 전이 학습 프레임워크가 진화 과정을 3-4배 가속화할 수 있음을 입증했다.4
  - **제어 시스템 검증:** 한 연구에서는 Webots에서 로봇 매니퓰레이터의 제어 시스템을 설계하고, 그 결과를 Matlab/Simulink의 결과와 직접 비교함으로써 시뮬레이션의 물리적 정확성과 제어 시스템 설계의 타당성을 검증했다.13
- **기관 채택 현황:** 전 세계 930개 이상의 대학 및 연구 센터에서 연구 및 교육 목적으로 Webots를 사용하고 있다.13


- **일반적인 산업 활용:** Webots는 산업 현장에서 이동 로봇의 신속한 프로토타이핑과 AI 및 제어 알고리즘의 개발, 테스트, 검증을 위해 널리 사용된다.8
- 주요 산업 프로젝트 8:
  - **자동차:** **르노(Renault)**는 Webots 기반 주행 플랫폼을 사용하여 차량 내 안내 시스템에 대한 운전자의 반응을 연구했다. **페론 로보틱스(Perrone Robotics)**는 자율 주행 차량용 소프트웨어 개발에 Webots를 활용하고 있으며, 광산 대기업 **BHP**는 자율 주행 채굴 차량 간의 상호작용을 모델링하는 데 사용한다.
  - **항공우주 및 원자력:** 프랑스의 **INTRA 그룹(EDF, CEA, Areva)**과 독일의 **KHG**는 원자력 사고 대응용 원격 조종 로봇의 조종사 훈련을 위해 맞춤형 Webots 시뮬레이터를 개발했다.
  - **수술 로봇:** 스위스 기업 **디스탈모션(Distalmotion)**이 개발한 **Dexter 수술 로봇**은 Webots에서 시뮬레이션되었다.
  - **제조/물류:** **SeRoNet** 프로젝트는 물류 및 산업 분야 서비스 로봇의 개발을 단순화하기 위해 Webots를 활용하며, 한 연구 논문에서는 제조 작업을 수행하는 UR3e 협동 로봇이 Webots 가상 환경에서 시뮬레이션되는 모습을 보여준다.27

Webots의 가치는 아이디어 구상부터 실제 구현에 이르는 전체 파이프라인에 걸쳐 있다. 학술 연구(군집 지능)에서 시작된 아이디어가 산업 프로토타입(자율 채굴 차량)으로 발전하고, 산업 표준 도구(Matlab/Simulink)로 검증되며, 최종 제품의 운용자 훈련(원자력 로봇)에까지 사용된다. 이처럼 연구, 개발, 훈련 단계에서 도구를 전환할 필요가 없다는 연속성은 마찰을 줄이고 오류 가능성을 낮추는 강력한 이점이다. 또한, 르노(자동차 안전), 디스탈모션(수술 로봇), INTRA 그룹(원자력 안전)과 같은 고위험 산업 분야에서의 성공적인 도입 사례는 Webots의 안정성, 결정론적 특성, 물리적 충실도에 대한 높은 신뢰를 방증한다. 이는 Webots가 사용자 친화적인 인터페이스를 가졌음에도 불구하고 전문가 수준의 "진지한" 프로젝트에 충분히 신뢰할 수 있는 도구임을 증명하는 강력한 실증적 증거이다.


- **교육 도구:** Webots는 입문 프로그래밍 강의부터 고급 AI 과정에 이르기까지 로봇 공학 교육을 위해 명시적으로 설계되고 사용된다.6 초보자 친화적인 특성과 방대한 튜토리얼은 교육 현장에서의 핵심 자산이다.7

- **로봇 경진대회:** Webots는 로봇 경진대회를 위한 인기 있는 플랫폼이다. 특히 **2021년 로보컵(RoboCup) 가상 휴머노이드 축구 대회**에 공식 시뮬레이터로 사용되었다.8 기본 제공되는 

  `soccer.wbt` 데모는 이러한 활용의 직접적인 예시를 보여준다.17

- **온라인 학습 플랫폼:** **robotbenchmark.net** 웹사이트는 클라우드에서 실행되는 Webots 시뮬레이션에 기반한 일련의 무료 로봇 프로그래밍 챌린지를 제공하여, 단계적인 학습 경로를 제시한다.6


로봇 시뮬레이터 선택은 프로젝트의 성패를 좌우할 수 있는 중요한 결정이다. 본 섹션에서는 정성적인 사용자 경험과 정량적인 성능 벤치마크 데이터를 종합하여 Webots를 주요 경쟁 시뮬레이터들과 비교 분석한다.


- **역사적 맥락:** Gazebo는 2002년부터 오픈 소스로 개발되어 ROS와 깊이 통합되면서 ROS 커뮤니티의 사실상 표준으로 자리 잡았다. 반면 Webots는 2018년에야 오픈 소스로 전환되어 커뮤니티 규모나 ROS 관련 튜토리얼의 양에서 Gazebo가 상당한 우위를 점하고 있었다.3
- **초보자 친화성 및 UI:** 사용자 포럼에서는 **Webots가 Gazebo보다 훨씬 초보자 친화적**이라는 의견이 지배적이다. Webots의 UI는 "더 세련되고", "체계적"이라고 평가받으며, 사용자가 실시간으로 변경 사항을 확인하며 프로토타입을 만들 수 있어 Gazebo의 XML 기반 씬 정의 방식보다 훨씬 쉽다는 평이다.3 반면 Gazebo는 사용자 친화적이지 않다는 평가를 종종 받는다.29
- **문서화:** Webots의 문서는 "더 완전하고 체계적"이라는 찬사를 받는다. 이와 대조적으로, Gazebo는 오랜 역사 때문에 온라인에 오래된 정보가 많아 신규 사용자에게 혼란을 줄 수 있으며, 이는 "Ignition"으로의 리브랜딩과 번복 과정에서 더욱 악화되었다.3 다만, 일부 사용자는 Webots의 문서가 복잡한 주제에 대해서는 부족할 수 있다고 지적하기도 한다.9
- **ROS 통합:** 두 시뮬레이터 모두 ROS2를 지원하지만, Gazebo가 더 많은 직접적인 튜토리얼과 역사적 유대감을 가지고 있는 것으로 인식된다. 그럼에도 불구하고 Webots의 ROS2 통합은 견고하며 활발하게 유지보수되고 있다.3

이러한 정성적인 사용자 경험의 차이는 실질적인 개발 효율성으로 이어진다. Webots의 직관적인 UI와 명확한 문서는 개발자의 "학습 곡선"과 "프로토타이핑 시간"을 직접적으로 단축시킨다. 혼란스럽고 오래된 문서나 투박한 씬 편집기와 씨름하는 개발자는 로봇 공학 문제를 해결하는 대신 도구와 싸우는 데 시간을 낭비하게 된다. 따라서 벤치마크가 CPU 사이클과 메모리를 측정하는 동안, 정성적인 장점은 개발자의 시간을 절약하여 프로젝트 완료를 앞당기고 비용을 절감하는, 정량화하기 어렵지만 매우 중요한 실질적 가치를 창출한다.


- **리소스 사용량 (CPU & 메모리):** 2020년 NAO 로봇 내비게이션 작업을 통해 Webots, Gazebo, V-REP(現 CoppeliaSim)을 비교한 연구에 따르면, **Webots가 가장 낮은 리소스 사용량**을 보였다. Webots는 평균 CPU 약 11%, 메모리 약 177MB를 사용한 반면, Gazebo는 CPU 약 42%, 메모리 약 204MB를 사용했다.2
- **안정성 및 반복성 (ROS2 환경):** 2022년 5개의 ROS2 호환 시뮬레이터(Ignition, Webots, Isaac Sim, PyBullet, CoppeliaSim)를 대상으로 장시간 집기-놓기 및 동적 던지기 작업을 평가한 체계적인 벤치마크 연구가 수행되었다.1 이 연구는 **Webots와 Ignition이 장기 작동에서 가장 높은 안정성**을 보였다고 결론지었다. 특히 **Webots는 모든 작업에서 가장 균형 잡히고 일관된 성능을 제공**하여 그 신뢰성과 뛰어난 반복성을 입증했다.1
- **모션 정확도:** 2022년 실제 Husky A200 로봇과 비교하여 CoppeliaSim, Gazebo, MORSE, Webots의 모션 시뮬레이션 정확도를 평가한 연구에서는, **CoppeliaSim이 정확도 측면에서 가장 우수한 성능을 보였고 Gazebo가 그 뒤를 이었다**.32 이는 특정 실제 로봇의 동역학을 매우 정밀하게 일치시키는 것이 최우선 과제인 응용 분야에서는 다른 시뮬레이터가 더 나은 선택일 수 있음을 시사한다.

정량적 데이터는 "최고의" 시뮬레이터가 단 하나가 아님을 명확히 보여준다. 각 시뮬레이터는 특정 영역에서 강점을 보인다. CoppeliaSim은 모션 정확도에서, Isaac Sim은 GPU 가속 강화학습에서, Gazebo는 깊은 ROS 통합에서 두각을 나타낸다. 이러한 맥락에서 Webots의 핵심 가치는 **균형 잡힌 성능과 효율성**에 있다. 모든 단일 지표에서 최고는 아닐지라도, 안정성, 리소스 사용량, 사용 편의성 등 전반적인 측면에서 일관되게 우수한 성능을 보여주어, 실용적이고 범용적인 선택지로서의 입지를 확고히 한다.

| 기능               | Webots                                             | Gazebo (Ignition)                                 | Isaac Sim                                       | CoppeliaSim                        |
| ------------------ | -------------------------------------------------- | ------------------------------------------------- | ----------------------------------------------- | ---------------------------------- |
| **라이선스**       | Apache 2.0                                         | Apache 2.0                                        | NVIDIA License                                  | Proprietary (Edu/Free versions)    |
| **주요 물리 엔진** | ODE (Custom Fork)                                  | ODE, Bullet, DART, Simbody                        | NVIDIA PhysX                                    | Bullet, ODE, Vortex, Newton        |
| **ROS/ROS2 지원**  | 우수                                               | 매우 우수 (네이티브)                              | 우수                                            | 우수                               |
| **주요 강점**      | 안정성, 리소스 효율성, 사용자 친화성, 크로스플랫폼 | 깊은 ROS 통합, 방대한 커뮤니티, 모듈성            | GPU 가속, RL/AI 최적화, 사실적 렌더링           | 높은 유연성, 스크립팅, 모션 정확도 |
| **주요 약점**      | 복잡한 주제에 대한 문서 부족 가능성                | 가파른 학습 곡선, 오래된 정보, 높은 리소스 사용량 | 높은 하드웨어 요구사항 (NVIDIA GPU), Linux 전용 | 일부 기능은 유료, 학습 곡선        |
| **주요 사용 사례** | 교육, 학술 연구, 신속 프로토타이핑, 군집 로봇      | ROS 기반 연구 및 개발, 실외 환경                  | 대규모 RL 훈련, 합성 데이터 생성, 디지털 트윈   | 산업 자동화, 정밀 모션 시뮬레이션  |

| 지표                               | Webots                 | Gazebo                       | CoppeliaSim (V-REP) | 출처 연구 |
| ---------------------------------- | ---------------------- | ---------------------------- | ------------------- | --------- |
| **CPU 사용량 (NAO 내비게이션)**    | **~11.05% (최저)**     | ~42.38%                      | ~20.65%             | 2         |
| **메모리 사용량 (NAO 내비게이션)** | **~176.6 MB (최저)**   | ~203.64 MB                   | ~212 MB             | 2         |
| **안정성/반복성 (ROS2 조작)**      | **높음 (최상위 그룹)** | 높음 (Ignition, 최상위 그룹) | -                   | 1         |
| **모션 정확도 (Husky A200)**       | -                      | 좋음 (2위)                   | **매우 좋음 (1위)** | 32        |


현대 로보틱스의 가장 중요한 두 가지 과제는 지능형 에이전트를 훈련시키는 것과 시뮬레이션된 결과를 현실 세계로 이전하는 것이다. 본 섹션에서는 이 두 가지 첨단 분야에서 Webots가 제공하는 역량과 프레임워크를 탐구한다.


- **구조적 적합성:** 앞서 분석했듯이, 슈퍼바이저 노드, 결정론적 물리 엔진, 그리고 실제 시간보다 빠르게 시뮬레이션을 실행할 수 있는 능력은 Webots의 핵심 아키텍처를 DRL 훈련 파이프라인에 매우 적합하게 만든다.14

- **커뮤니티 주도 프레임워크:** 연구 커뮤니티는 Webots를 DRL에 보다 효율적으로 사용하기 위한 프레임워크를 구축해왔다.

  - **`deepbots`:** Webots를 위한 표준화된 OpenAI Gym 인터페이스를 제공하는 오픈 소스 프레임워크이다.35 이는 연구자들이 Webots 환경을 Stable-Baselines3와 같은 인기 있는 DRL 라이브러리에 쉽게 연결할 수 있도록 하는 핵심적인 기반 시설이다. 

    `deepbots`는 저수준의 통신을 처리하고 개발 노력을 줄여준다.

  - **컨테이너화 아키텍처:** 또 다른 접근법은 Docker와 같은 컨테이너 기술을 사용하여 Webots 시뮬레이션 환경과 모델 개발 환경을 분리하는 것이다.38 이를 통해 데이터 과학자들은 Webots 자체에 대한 깊은 지식 없이도 ROS와 같은 깔끔한 API를 통해 시뮬레이션과 상호작용할 수 있으며, 이는 무인 자동 훈련 파이프라인 구축을 용이하게 한다.

- **연구 적용 사례:** Webots는 DRL 연구에 활발히 사용되고 있다. 예를 들어, 조밀한 환경에서 항공 로봇의 안전한 종단 간(end-to-end) 경로 계획 정책을 훈련시키는 데 사용되었으며, 이 과정에서 일반화 성능을 높이기 위해 고도로 무작위화된 환경 구성을 활용했다.4

Webots는 DRL 채택의 "선순환 구조"에 진입했다. 그 핵심 아키텍처(슈퍼바이저, 결정론)가 DRL에 적합했기 때문에 초기 연구자들이 이를 DRL 프로젝트에 사용하기 시작했다. 이는 더 나은 통합의 필요성을 낳았고, `deepbots`와 같은 커뮤니티 도구의 개발로 이어졌다. 이 도구들은 다시 새로운 연구자들이 Webots를 DRL에 채택하는 것을 더 쉽게 만들어, 긍정적인 피드백 루프를 형성하고 있다. 비록 Isaac Sim이 GPU 가속 DRL의 선두주자로 여겨지지만, Webots는 CPU 기반의 리소스 효율적이고 접근성 높은 DRL 플랫폼으로서 강력한 틈새 시장을 구축하고 있다.


- **핵심 설계 철학:** 컨트롤러를 실제 로봇으로 이전하는 기능은 최근에 추가된 것이 아니라, Webots의 초기 시절부터 이어진 근본적인 설계 개념이다.5
- 두 가지 주요 메커니즘 5:
  - **원격 제어(Remote Control):** 컨트롤러 프로그램은 호스트 PC에서 실행되지만, 시뮬레이션된 로봇과 통신하는 대신 사용자 정의 라이브러리가 API 호출(예: `wb_motor_set_velocity`)을 실제 로봇으로 리디렉션한다. 이 통신은 주로 직렬 포트나 다른 인터페이스를 통해 이루어진다. 실제 로봇의 센서 데이터도 같은 방식으로 다시 읽어온다. 이는 실제 환경에서 직접 테스트하기 위한 가장 간단한 방법이다.
  - **교차 컴파일(Cross-Compilation):** 진정한 임베디드 배포를 위해, C, C++, Python 또는 Java로 작성된 컨트롤러 소스 코드를 실제 로봇의 타겟 프로세서용으로 다시 컴파일한다. 그러면 로봇은 PC와의 영구적인 연결 없이 자율적으로 제어 프로그램을 실행하게 된다. Webots는 e-puck과 같은 로봇을 위한 교차 컴파일 예제를 제공한다.
- **튜닝의 중요성:** 문서는 시뮬레이션이 현실의 근사치일 뿐임을 명시적으로 인정한다. 성공적인 Sim-to-Real 전환을 위해서는 시뮬레이션된 모델의 매개변수를 실제 로봇의 동작과 더 잘 일치하도록 "튜닝"하는 과정이 필수적이며, 이는 "현실과의 간극(reality gap)"을 줄이는 데 결정적인 단계이다.5

Webots의 성숙한 Sim-to-Real 프레임워크는 완전한 시스템 개발에 있어 핵심적인 차별점이다. 많은 시뮬레이터가 제어 알고리즘을 생성할 수 있지만, Webots는 그 알고리즘을 실제 하드웨어에 *배포*하기 위한 완전하고 문서화된, 오랜 역사를 지닌 툴체인을 제공한다. 디버깅과 테스트를 위한 원격 제어, 최종 배포를 위한 교차 컴파일을 모두 명시적으로 지원하는 것은 전체 워크플로우를 포괄한다. 현실에 맞게 시뮬레이션을 "튜닝"해야 한다는 실용적인 조언은 이 분야의 어려움에 대한 깊고 실질적인 이해를 보여준다. 이는 최종 목표가 물리적 프로토타입인 프로젝트에 Webots를 특히 강력한 선택지로 만든다.


본 보고서의 분석을 종합하여, Webots에 대한 최종적인 전문가 평가와 다양한 사용자 프로필에 맞춘 권장 사항을 제시한다. 이는 새로운 데이터를 도입하기보다는, 제시된 증거들을 바탕으로 고차원적인 전략적 결론을 도출하는 데 중점을 둔다.


Webots는 틈새 독점 도구에서 주요 오픈 소스 경쟁자로 성공적으로 진화했으며, 20년간의 전문적인 완성도와 현대적인 오픈 소스의 유연성을 독특하게 결합했다. 그 핵심 강점은 사용자 친화적인 GUI, 견고한 안정성, 높은 리소스 효율성, 그리고 폭넓은 다중 언어 지원이라는 균형 잡힌 특징에 있다. 모든 단일 지표에서 최고는 아니지만, 안정성과 효율성의 정량적 벤치마크에서는 지속적으로 최상위 성능을, 사용 편의성과 문서 명확성의 정성적 평가에서는 선두를 차지한다. 또한, 그 아키텍처는 DRL과 같은 고급 응용 분야에 근본적으로 적합하며, Sim-to-Real 전환이라는 핵심 과제를 위한 성숙하고 검증된 프레임워크를 갖추고 있다.


- **교육자 및 학생:** Webots는 **매우 탁월한 선택이며 강력히 추천된다**. 낮은 학습 곡선, 우수한 문서, 방대한 튜토리얼, 무료 오픈 소스 정책, 그리고 e-puck과 같은 교육용 로봇에 대한 내장 지원은 로봇 공학, 프로그래밍, AI의 기초를 가르치고 배우는 데 이상적인 플랫폼을 제공한다. `robotbenchmark.net` 플랫폼은 이미 준비된 교육과정을 제공하는 것과 같다.
- **학술 연구자:** Webots는 **강력하고 다재다능한 옵션이며, 특정 분야에 특히 유용하다**. 다수의 에이전트를 효율적으로 시뮬레이션할 수 있으므로 **군집 지능, 다개체 시스템, 진화 로보틱스** 연구에 선호되는 도구가 되어야 한다. 안정성과 반복성이 무엇보다 중요한 경우, 대규모 GPU 병렬화가 필요하지 않은 **DRL 연구**에 탁월한 선택이다. 물리적 로봇으로의 명확한 경로를 가진 연구자들은 성숙한 Sim-to-Real 툴체인으로부터 큰 이점을 얻을 것이다.
- **산업 R&D 전문가:** Webots는 **신속한 프로토타이핑과 알고리즘 검증을 위한 매우 효과적인 도구**이다. 복잡한 시나리오를 신속하게 구축하고 테스트할 수 있는 능력과 낮은 리소스 요구사항은 개발 주기를 크게 단축시킬 수 있다. Matlab/Simulink와 같은 표준에 대해 제어 시스템을 검증할 수 있는 기능은 중요한 자산이다. 자동차, 수술, 원자력 로봇과 같은 고위험 산업에서의 성공 사례는 전문가 수준의 프로젝트에 대한 신뢰성을 입증한다.



Webots는 아파치 2.0(Apache 2.0) 라이선스 하에 배포된다.6 이 라이선스는 사용자에게 소프트웨어와 그 소스 코드를 어떤 목적으로든, **상업적 응용을 포함하여**, 무료로 사용, 수정, 배포할 권한을 부여한다. 이는 파생 저작물에 대해 동일한 오픈 소스 라이선스를 강제하지 않는 "허용적(permissive)" 라이선스이다. 따라서 Webots는 독점적인 상용 제품을 개발하는 데 안전하게 사용될 수 있다. 단, 일부 특정 자산은 더 제한적인 "Webots Assets license" 하에 배포될 수 있으나, 이는 관련 파일에 명확히 명시되어 있다.11


Webots는 Windows, macOS, Linux 운영체제를 모두 지원하며, 공식 웹사이트에서 각 플랫폼에 맞는 설치 파일을 다운로드할 수 있다.8 설치 과정은 대체로 간단하지만, 몇 가지 유의사항이 있다.

- **macOS:** 애플리케이션이 서명되지 않았기 때문에 첫 실행 시 보안 경고가 발생할 수 있다. 이를 피하려면 Control 키를 누른 상태에서 Webots 앱을 클릭하고 '열기'를 선택해야 한다.12
- **GPU 요구사항:** Webots는 3D 렌더링을 위해 적당한 수준의 GPU 성능을 요구한다. 그래픽 성능이 부족하다는 경고가 나타날 경우, 렌더링 옵션을 조정하여 성능을 향상시킬 수 있다.12
- **외부 Python 설정:** Python 컨트롤러를 사용하려면 시스템에 Python 3가 설치되어 있어야 한다. Webots는 시스템에 설치된 외부 Python 인터프리터를 사용하도록 설정해야 할 수 있다.12


Webots 사용자는 다양한 공식 및 커뮤니티 채널을 통해 지원을 받거나 커뮤니티에 참여할 수 있다.

- **공식 채널:**
  - **GitHub 저장소:** 소스 코드 접근, 이슈 추적, 버그 리포트를 위한 공식 채널이다.7
  - **공식 문서:** 사용자 가이드(User Guide)와 참조 매뉴얼(Reference Manual)은 가장 신뢰할 수 있는 정보원이다.8
  - **Discord 서버:** 빠른 질문과 커뮤니티 구성원들과의 실시간 토론을 위한 공간이다.8
- **커뮤니티 허브:**
  - **Stack Overflow:** `webots` 태그를 사용하여 질문을 게시하면 개발팀과 커뮤니티가 답변을 제공한다.40
  - **Reddit:** r/Webots 서브레딧은 사용자 간의 정보 공유와 토론을 위한 공간이다.23
  - **YouTube:** Cyberbotics 공식 채널이나 Kajal Gada와 같은 커뮤니티 제작자들이 제공하는 다양한 비디오 튜토리얼을 찾아볼 수 있다.28


1. A Systematic Comparison of Simulation Software for Robotic Arm Manipulation using ROS2, accessed July 22, 2025, https://www.alphaxiv.org/overview/2204.06433v1
2. (PDF) A Comparison of Humanoid Robot Simulators: A Quantitative ..., accessed July 22, 2025, https://www.researchgate.net/publication/347629578_A_Comparison_of_Humanoid_Robot_Simulators_A_Quantitative_Approach
3. Why is Gazebo very famous in the ROS community? what about Webots?, accessed July 22, 2025, https://discourse.ros.org/t/why-is-gazebo-very-famous-in-the-ros-community-what-about-webots/42459
4. Research papers relying on Webots #2621 - GitHub, accessed July 22, 2025, https://github.com/cyberbotics/webots/discussions/2621
5. Webots documentation: Transfer to your own Robot, accessed July 22, 2025, https://cyberbotics.com/doc/guide/transfer-to-your-own-robot
6. Webots - Wikipedia, accessed July 22, 2025, https://en.wikipedia.org/wiki/Webots
7. cyberbotics/webots: Webots Robot Simulator - GitHub, accessed July 22, 2025, https://github.com/cyberbotics/webots
8. Cyberbotics: Robotics simulation with Webots, accessed July 22, 2025, https://cyberbotics.com/
9. Cyberbotics Ltd. Webots Reviews, Price, Use-cases, Compare... - QVIRO, accessed July 22, 2025, https://qviro.com/product/cyberbotics-ltd/webots-cyberbotics
10. 3.8. Webots - Vulcanexus 1.0.0 documentation, accessed July 22, 2025, https://docs.vulcanexus.org/en/humble/rst/introduction/tools/webots.html
11. Webots documentation: License Agreement - Cyberbotics, accessed July 22, 2025, https://cyberbotics.com/doc/guide/webots-license-agreement
12. Webots Robot Simulator - Robotics for Creative Practice - Fall 2022, accessed July 22, 2025, https://courses.ideate.cmu.edu/16-375/f2022/ref/text/resources/Webots.html
13. An Application Example of Webots in Solving Control Tasks of Robotic System, accessed July 22, 2025, https://www.mas.bg.ac.rs/_media/istrazivanje/fme/vol41/2/10_pmandic.pdf
14. Introduction to Webots, accessed July 22, 2025, https://cyberbotics.com/doc/guide/introduction-to-webots
15. Tutorial 1: Your First Simulation in Webots (30 Minutes), accessed July 22, 2025, https://cyberbotics.com/doc/guide/tutorial-1-your-first-simulation-in-webots
16. Webots documentation: Technical-Questions 2021, accessed July 22, 2025, https://cyberbotics.com/doc/discord/technical-questions-2021
17. Webots documentation: Demos, accessed July 22, 2025, https://cyberbotics.com/doc/guide/samples-demos
18. Webots documentation: Controller Programming - Cyberbotics, accessed July 22, 2025, https://cyberbotics.com/doc/guide/controller-programming
19. Webots documentation: Keyboard, accessed July 22, 2025, https://cyberbotics.com/doc/reference/keyboard
20. (PDF) WebotsTM: Professional Mobile Robot Simulation - ResearchGate, accessed July 22, 2025, https://www.researchgate.net/publication/221702459_WebotsTM_Professional_Mobile_Robot_Simulation
21. Cyberbotics Ltd. Webots : Professional Mobile Robot Simulation - arXiv, accessed July 22, 2025, https://arxiv.org/pdf/cs/0412052
22. webots.cloud - home, accessed July 22, 2025, https://webots.cloud/
23. Webots: Robot Simulator - Reddit, accessed July 22, 2025, https://www.reddit.com/r/Webots/
24. Environments - Webots documentation, accessed July 22, 2025, https://cyberbotics.com/doc/guide/samples-environments
25. Webots documentation: PROTO Design Guidelines, accessed July 22, 2025, https://cyberbotics.com/doc/reference/proto-design-guidelines
26. An application example of webots in solving control tasks of robotic ..., accessed July 22, 2025, https://www.researchgate.net/publication/290321475_An_application_example_of_webots_in_solving_control_tasks_of_robotic_system
27. The virtual environment built on Webots: it consists of a table, a UR3e... - ResearchGate, accessed July 22, 2025, https://www.researchgate.net/figure/The-virtual-environment-built-on-Webots-it-consists-of-a-table-a-UR3e-cobot-left-with_fig5_359384664
28. how to install and run your first simulation in 10 min | Webots Tutorial 1 - YouTube, accessed July 22, 2025, https://www.youtube.com/watch?v=luyg3plGujg&pp=0gcJCfwAo7VqN5tD
29. Webots, Mujoco...Which robot simulation software is better? - EEWorld, accessed July 22, 2025, https://en.eeworld.com.cn/news/robot/eic691307.html
30. A Comparison of Humanoid Robot Simulators: A Quantitative Approach - Francisco Cruz, accessed July 22, 2025, https://franciscocruz.org/publications/Ayala_et_al_ICDL_2020.pdf
31. A Systematic Comparison of Simulation Software for Robotic Arm Manipulation using ROS2 - arXiv, accessed July 22, 2025, https://arxiv.org/pdf/2204.06433
32. (PDF) How to pick a mobile robot simulator: A quantitative comparison of CoppeliaSim, Gazebo, MORSE and Webots with a focus on the accuracy of motion simulations - ResearchGate, accessed July 22, 2025, https://www.researchgate.net/publication/361876364_How_to_pick_a_mobile_robot_simulator_A_quantitative_comparison_of_CoppeliaSim_Gazebo_MORSE_and_Webots_with_a_focus_on_the_accuracy_of_motion_simulations
33. How to pick a mobile robot simulator: A quantitative comparison of CoppeliaSim, Gazebo, MORSE and Webots with a focus on accuracy of motion - GitHub, accessed July 22, 2025, https://github.com/offroad-robotics/robot-simulator-comparison
34. Comparing Popular Simulation Environments in the Scope of Robotics and Reinforcement Learning - arXiv, accessed July 22, 2025, https://arxiv.org/pdf/2103.04616
35. Framework for deep reinforcement learning in Webots virtual ..., accessed July 22, 2025, https://journals.vilniustech.lt/index.php/NTCS/article/view/23668
36. Deepbots: A Webots-Based Deep Reinforcement Learning Framework for Robotics, accessed July 22, 2025, https://www.researchgate.net/publication/341730698_Deepbots_A_Webots-Based_Deep_Reinforcement_Learning_Framework_for_Robotics
37. Deepbots: A Webots-Based Deep Reinforcement Learning Framework for Robotics - PMC, accessed July 22, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7256566/
38. [2403.00765] An Architecture for Unattended Containerized (Deep) Reinforcement Learning with Webots - arXiv, accessed July 22, 2025, https://arxiv.org/abs/2403.00765
39. Cabinet - Webots documentation, accessed July 22, 2025, https://cyberbotics.com/doc/guide/object-cabinet?tab-os=linux
40. Webots documentation: Technical-Questions 2019, accessed July 22, 2025, https://cyberbotics.com/doc/discord/technical-questions-2019?version=R2021b
41. Cyberbotics Webots - YouTube, accessed July 22, 2025, https://www.youtube.com/user/cyberboticswebots/videos
42. Webots Tutorial Series in Python // Get started with Robotics using Webots - YouTube, accessed July 22, 2025, https://www.youtube.com/playlist?list=PLbEU0vp_OQkUwANRMUOM00SXybYQ4TXNF


# ROS2 Humble 환경에서의 OMPL

## 서론

### 0.1 배경 및 중요성

현대 로보틱스 시스템, 특히 자율 매니퓰레이션(autonomous manipulation) 분야에서 모션 플래닝(motion planning)은 가장 근본적이고 도전적인 과제 중 하나로 자리 잡고 있다. 로봇이 주어진 작업을 성공적으로 수행하기 위해서는 자신의 기구학적, 동역학적 제약 조건을 만족시키면서 작업 공간 내의 장애물과 충돌하지 않는 유효한 경로를 생성해야 한다. 로봇 운영체제(Robot Operating System, ROS)의 두 번째 버전인 ROS2, 그중에서도 Humble Hawksbill 배포판은 강건하고 실시간성 높은 로보틱스 애플리케이션 개발을 위한 표준 플랫폼으로 부상하였다. 이 생태계의 중심에는 로봇 매니퓰레이션을 위한 통합 소프트웨어 프레임워크인 MoveIt2가 있으며, MoveIt2의 핵심적인 경로 계획 기능은 OMPL(Open Motion Planning Library)이라는 강력한 백엔드(backend) 라이브러리를 통해 구현된다.1

### 0.2 샘플링 기반 플래닝 패러다임

전통적인 그리드 기반(grid-based) 또는 조합적(combinatorial) 플래닝 방법들은 로봇의 자유도(Degrees of Freedom, DoF)가 증가함에 따라 설정 공간(Configuration Space, C-Space)의 크기가 기하급수적으로 팽창하는 '차원의 저주(curse of dimensionality)' 문제에 직면한다.3 이로 인해 고차원 공간에서는 현실적인 시간 내에 해를 찾는 것이 거의 불가능해진다. 이러한 한계를 극복하기 위해 등장한 패러다임이 바로 샘플링 기반 플래닝(Sampling-Based Motion Planning, SBMP)이다. SBMP는 설정 공간 전체를 명시적으로 구성하는 대신, 무작위 샘플링을 통해 공간의 연결성을 확률적으로 탐색한다. OMPL은 이러한 샘플링 기반 플래닝 패러다임의 정점에 있는 라이브러리로서, 확률적 로드맵(Probabilistic Roadmap, PRM), 빠른 탐색 랜덤 트리(Rapidly-exploring Random Tree, RRT)와 같은 최첨단 알고리즘 및 그 변형들을 집대성한 포괄적인 도구 모음이다.2

### 0.3 보고서의 목적과 구성

본 보고서는 ROS2 Humble 환경에서 OMPL을 활용하고자 하는 로보틱스 연구자 및 고급 개발자를 대상으로 한다. 단순한 API 사용법이나 튜토리얼 수준을 넘어, OMPL의 근본적인 아키텍처, 핵심 이론, 그리고 MoveIt2와의 통합 과정에서 발생하는 미묘한 상호작용에 대한 심층적인 고찰을 제공하는 것을 목적으로 한다. 보고서는 다음과 같이 구성된다. 제1장에서는 MoveIt2의 플러그인 아키텍처 내에서 OMPL이 어떻게 통합되고 데이터가 처리되는지 구조적 관점에서 분석한다. 제2장에서는 OMPL 라이브러리 자체의 설계 철학과 핵심 구성 요소를 탐구하여 그 이론적 기반을 다진다. 제3장에서는 실제 설정 파일인 `ompl_planning.yaml`을 통해 플래너의 동작을 세밀하게 제어하고 튜닝하는 방법을 상세히 기술한다. 제4장에서는 RRT, PRM, FMT* 등 MoveIt2에서 널리 사용되는 주요 플래너들의 작동 원리와 장단점을 심도 있게 비교 분석한다. 제5장에서는 샘플링 기반 플래닝의 근본적인 난제인 차원의 저주와 좁은 통로 문제를 OMPL의 관점에서 고찰하고, 각 알고리즘의 계산 복잡도를 분석한다. 제6장에서는 운동학적 제약 조건 및 동적 환경과 같은 고급 플래닝 시나리오에서 OMPL의 적용 가능성과 한계를 논한다. 마지막으로, 결론에서는 전체 내용을 요약하고 머신러닝과의 통합 등 향후 발전 방향을 전망한다.

## 1.  ROS2 Humble과 MoveIt2에서의 OMPL 통합 아키텍처

OMPL은 그 자체로 독립적인 라이브러리이지만, ROS2 생태계에서는 MoveIt2라는 거대한 프레임워크 내에서 핵심적인 플래닝 엔진으로 기능한다. 이 장에서는 OMPL이 MoveIt2의 유연한 아키텍처에 어떻게 통합되어 작동하는지를 구조적 관점에서 상세히 해부한다. 이 과정의 핵심에는 MoveIt2의 플러그인 기반 설계와, 두 시스템을 매개하는 `ompl_interface` 패키지의 역할이 있다.

### 1.1  MoveIt2 플래너 플러그인 아키텍처

MoveIt2의 모션 플래닝 프레임워크는 높은 수준의 모듈성과 확장성을 지향하도록 설계되었다. 이러한 설계 철학의 핵심은 플래너 플러그인 아키텍처에 있다. MoveIt2는 특정 플래닝 알고리즘에 종속되지 않고, `planning_interface::PlannerManager`라는 추상 기반 클래스를 통해 다양한 모션 플래너 라이브러리를 동적으로 로드하고 사용할 수 있는 인터페이스를 제공한다.1

이 아키텍처 덕분에 OMPL은 MoveIt2의 '기본(default)' 플래너로 기능하면서도, 필요에 따라 CHOMP(Covariant Hamiltonian Optimization for Motion Planning), STOMP(Stochastic Trajectory Optimization for Motion Planning) 등 다른 최적화 기반 플래너들과 동일한 인터페이스 상에서 교체되거나 함께 사용될 수 있다.7 ROS2 시스템의 중심 노드인 `move_group`은 이 플러그인 인터페이스를 통해 로드된 플래너들에게 ROS 액션(`action`) 또는 서비스(`service`) 형태로 모션 플랜 요청(`MotionPlanRequest`)을 전달한다. 사용자는 설정 파일을 통해 어떤 플래너 플러그인을 로드할지, 그리고 각 플래너를 어떻게 설정할지를 지정할 수 있다. 이와 같은 모듈식 설계는 MoveIt2가 특정 알고리즘의 발전에 얽매이지 않고 지속적으로 최신 플래닝 기술을 수용할 수 있게 하는 기반이 된다.

### 1.2  `ompl_interface` 패키지: MoveIt2와 OMPL의 교량

`ompl_interface` 패키지는 MoveIt2 프레임워크와 OMPL 라이브러리 사이의 핵심적인 교량 역할을 수행한다. 이 패키지는 두 시스템 간의 개념적, 데이터 구조적 차이를 해소하는 추상화 계층(abstraction layer)으로 기능한다.1 OMPL은 순수한 알고리즘 라이브러리로서 로봇의 기구학, 작업 공간의 장애물, 혹은 충돌이라는 구체적인 개념을 내장하고 있지 않다.2 반면 MoveIt2는 로봇 모델(`moveit::core::RobotModel`), 현재의 월드 상태를 나타내는 플래닝 씬(`planning_scene::PlanningScene`) 등 로보틱스 문제에 특화된 풍부한 컨텍스트를 가지고 있다.

`ompl_interface`는 바로 이 지점에서 다음과 같은 핵심적인 변환 및 관리 작업을 수행한다:

- **문제 변환:** MoveIt2의 `MotionPlanRequest`와 `PlanningScene` 정보를 OMPL이 이해할 수 있는 추상적인 문제 정의, 즉 `ompl::base::StateSpace`, `ompl::base::StateValidityChecker`, `ompl::base::Goal` 등으로 변환한다.1
- **상태 유효성 검사:** OMPL의 `StateValidityChecker`가 호출될 때, 내부적으로 MoveIt2의 `PlanningScene`을 이용해 해당 설정(configuration)에서의 충돌 여부 및 관절 한계 준수 여부를 검사하고 그 결과를 OMPL에 반환한다.
- **결과 변환:** OMPL 플래너가 성공적으로 경로(`ompl::geometric::PathGeometric`)를 찾으면, `ompl_interface`는 이 경로를 MoveIt2가 사용할 수 있는 `moveit_msgs::msg::RobotTrajectory` 메시지 형식으로 다시 변환한다.

이처럼 `ompl_interface`는 OMPL의 추상적인 알고리즘을 실제 로봇 문제에 적용 가능하게 만드는 필수적인 매개체이다.

### 1.3  `OMPLPlannerManager` 클래스 심층 분석

`ompl_interface` 패키지의 심장부에는 `ompl_interface::OMPLPlannerManager` 클래스가 있다. 이 클래스는 앞서 언급한 MoveIt2의 `planning_interface::PlannerManager` 추상 클래스를 상속받아 구체적으로 구현한 것으로, OMPL 기반 플래너들을 MoveIt2 프레임워크에 공식적으로 등록하고 그 생명주기를 관리하는 책임을 진다.9

이 클래스의 주요 메서드와 그 역할은 다음과 같다:

- `initialize()`: `move_group` 노드가 시작될 때 호출되며, MoveIt2의 `RobotModel` 정보를 받아 OMPL 플래닝을 위한 전반적인 환경을 초기화한다.
- `setPlannerConfigurations()`: `ompl_planning.yaml` 파일로부터 읽어들인 각 플래너의 설정 정보(파라미터 등)를 `planning_interface::PlannerConfigurationMap` 형태로 받아 내부적으로 저장하고 관리한다.9
- `getPlanningContext()`: 이 클래스의 가장 핵심적인 메서드이다. 실제 플래닝 요청이 들어왔을 때, 요청 메시지(`MotionPlanRequest`)와 현재 플래닝 씬(`PlanningScene`)을 분석하여 가장 적합한 OMPL 플래너 인스턴스와 해당 플래닝 문제에 대한 구체적인 컨텍스트(`PlanningContext`)를 생성하여 반환한다. 이 과정에서 요청된 플래너 ID에 해당하는 설정을 `setPlannerConfigurations()`를 통해 저장된 맵에서 조회하여 적용한다.9
- `canServiceRequest()`: 주어진 `MotionPlanRequest`를 OMPL 플래너들이 처리할 수 있는지 여부를 판단한다. 예를 들어, 요청에 포함된 제약 조건이 OMPL에서 지원되지 않는 종류일 경우 `false`를 반환할 수 있다.

`OMPLPlannerManager`는 단순히 플래너를 생성하는 것을 넘어, MoveIt2의 동적인 플래닝 요청에 맞추어 적절한 OMPL 자원을 할당하고 설정하는 동적인 관리자의 역할을 수행한다.

### 1.4  데이터 흐름: 요청부터 실행까지

ROS2 Humble 환경에서 OMPL을 이용한 모션 플래닝의 전체 데이터 흐름은 다음과 같은 단계로 요약할 수 있다.

1. **요청 (Request):** 사용자(예: RViz의 인터랙티브 마커, C++/Python으로 작성된 노드)가 특정 목표 자세(pose goal)나 관절 공간 목표(joint space goal)를 담은 `MotionPlanRequest`를 `move_group` 노드의 액션 서버로 전송한다.
2. **컨텍스트 생성 (Context Generation):** `move_group`은 등록된 플래너 플러그인 중 OMPL을 담당하는 `OMPLPlannerManager`에게 `getPlanningContext()`를 호출한다. `OMPLPlannerManager`는 요청 내용과 현재 `PlanningScene`을 바탕으로 특정 OMPL 플래너(예: `RRTConnect`)와 문제 정의(`ProblemDefinition`)가 포함된 `PlanningContext` 객체를 생성한다.
3. **플래닝 실행 (Planning Execution):** `PlanningContext` 내에서 `ompl_interface`가 MoveIt2의 로봇 상태(관절 값)와 월드 정보(장애물)를 OMPL의 `State`와 `StateValidityChecker`로 변환한다. 이후 선택된 OMPL 플래너의 `solve()` 메서드가 호출되어 경로 탐색을 수행한다.
4. **후처리 및 반환 (Post-Processing & Return):** OMPL이 경로를 반환하면, `ompl_interface`는 이를 MoveIt2의 `RobotTrajectory` 형식으로 변환한다. 이 시점에서 MoveIt2의 플래닝 어댑터(Planning Request Adapters) 체인이 작동하여, 경로에 시간 정보를 입히는 시간 매개변수화(`AddTimeParameterization`)나 경로를 부드럽게 만드는 스무딩(smoothing) 등의 후처리 작업이 수행될 수 있다.1
5. **실행 (Execution):** 최종적으로 생성된 궤적(`trajectory`)은 `move_group`을 통해 사용자에게 반환되며, 사용자가 실행을 명령하면 `FollowJointTrajectoryAction` 인터페이스를 통해 해당 로봇의 컨트롤러로 전송되어 실제 로봇이 움직이게 된다.11

이러한 일련의 과정은 잘 정의된 인터페이스와 추상화 계층을 통해 이루어지므로, 사용자는 OMPL의 복잡한 내부 동작을 모두 알지 못하더라도 MoveIt2를 통해 강력한 플래닝 기능을 활용할 수 있다.

이 아키텍처는 상당한 유연성을 제공하지만, 동시에 고려해야 할 점을 내포한다. OMPL 라이브러리는 독립적으로 매우 빠르게 발전하며 새로운 플래너나 최적화 기법이 지속적으로 추가된다.12 그러나 이러한 최신 기능이 MoveIt2 사용자가 즉시 활용 가능함을 의미하지는 않는다. 

`ompl_interface`라는 추상화 계층이 해당 기능을 MoveIt2의 설정 파일 및 API에 명시적으로 "노출(expose)"하도록 업데이트되어야만 비로소 사용이 가능해진다. 실제로 MoveIt2 문서에서도 OMPL에 존재하지만 아직 MoveIt2에서는 사용할 수 없는 최적화 플래너들이 언급되기도 한다.13 이는 고급 기능을 사용하려는 개발자가 MoveIt2와 OMPL의 버전 호환성 및 `ompl_interface`의 지원 범위를 반드시 확인해야 함을 시사한다. 이 간극은 때로는 최신 연구 결과를 실제 로봇에 적용하고자 할 때 기술적 장벽으로 작용할 수 있으며, 이 경우 `ompl_interface` 패키지를 직접 수정하거나 확장해야 할 필요성이 발생할 수 있다.

## 2.  OMPL의 핵심 개념과 구성 요소

MoveIt2라는 프레임워크를 통해 OMPL을 사용하는 것을 넘어, 그 성능을 극한으로 끌어올리고 예상치 못한 문제에 대응하기 위해서는 OMPL 라이브러리 자체의 내부 철학과 핵심 구성 요소를 이해하는 것이 필수적이다. 이 장에서는 MoveIt2라는 래퍼(wrapper)를 벗겨내고 OMPL의 순수한 아키텍처를 탐구한다. 이 근본적인 이해는 이후 논의될 고급 튜닝과 문제 해결의 견고한 기반이 된다.

### 2.1  OMPL의 설계 철학: 추상화와 모듈성

OMPL의 설계 철학의 핵심은 '알고리즘'과 '문제 정의'의 철저한 분리에 있다. OMPL은 특정 로봇, 특정 충돌 검사기, 혹은 특정 시각화 도구에 종속되지 않는, 순수한 모션 플래닝 알고리즘의 집합체로 설계되었다.2 라이브러리의 내용은 샘플링 기반 알고리즘 자체에 국한되며, 환경 명세나 충돌 감지와 같은 구체적인 구현은 외부에서 제공되어야 한다.

이러한 높은 수준의 추상성은 OMPL의 가장 큰 강점이다. 사용자는 자신의 문제, 예를 들어 특정 로봇의 기구학이나 작업 환경의 장애물 정보에 맞는 구체적인 구성 요소(예: 상태 유효성 검사기)를 마치 부품처럼 OMPL 프레임워크에 "주입(inject)"하여 사용할 수 있다.14 이 모듈성 덕분에 OMPL은 로봇 팔 매니퓰레이션뿐만 아니라, 모바일 로봇 내비게이션, 심지어는 단백질 접힘(protein folding)과 같은 비-로보틱스 분야의 고차원 경로 탐색 문제에도 성공적으로 적용될 수 있다. 플래닝 알고리즘의 로직과 문제의 특성이 코드 수준에서 명확히 분리되어 있기 때문에, 동일한 RRT* 알고리즘 코드가 아무런 변경 없이 다양한 도메인의 문제 해결에 재사용될 수 있다. 이는 OMPL을 단순한 '플래너 모음'이 아닌, 범용적인 '플래닝 프레임워크'로 만드는 핵심적인 설계 원칙이다.

### 2.2  플래닝 문제 정의의 3요소

OMPL에서 모션 플래닝 문제를 정의하기 위해서는 최소한 세 가지 핵심 요소가 필요하다. 이들은 플래닝 알고리즘이 작동할 운동장과 규칙을 설정한다.

- **`ompl::base::StateSpace` (상태 공간):** 플래닝이 수행될 공간, 즉 로봇의 모든 가능한 설정(configuration)의 집합인 설정 공간(C-Space)을 수학적으로 정의한다. 이 클래스는 단순히 상태를 저장하는 것을 넘어, 해당 공간의 위상(topology)에 특화된 필수적인 연산을 제공해야 한다. 여기에는 두 상태 간의 거리를 계산하는 함수(`distance`), 두 상태 사이를 보간(interpolate)하는 함수, 그리고 상태를 위한 메모리를 할당하고 해제하는 기능이 포함된다.14 OMPL은 

  `SE(3)`(3차원 강체 변환), `SO(2)`(2차원 회전), `RealVectorStateSpace`(N차원 실수 벡터 공간) 등 로보틱스에서 자주 사용되는 다양한 상태 공간 구현체를 기본적으로 제공하여 사용자의 편의를 돕는다.2

- **`ompl::base::StateValidityChecker` (상태 유효성 검사기):** 주어진 상태(`State`)가 유효한지를 판별하는 역할을 한다. '유효성'의 기준은 문제에 따라 달라지며, 로보틱스에서는 주로 로봇이 자기 자신이나 환경과 충돌하지 않는지, 그리고 모든 관절이 기구학적 한계 내에 있는지를 검사하는 것을 의미한다. OMPL의 설계 철학에 따라, 이 구성 요소는 전적으로 문제에 종속적(problem-specific)이므로 OMPL 자체는 어떠한 기본 구현도 제공하지 않는다. 사용자는 반드시 자신의 문제에 맞는 유효성 검사 로직을 클래스 또는 함수 형태로 구현하여 OMPL에 제공해야 한다.14 MoveIt2 환경에서는 

  `ompl_interface`가 `PlanningScene`의 충돌 검사 기능을 이용하여 이 `StateValidityChecker`를 자동으로 생성하고 OMPL에 주입한다.

- **`ompl::base::Goal` (목표):** 플래닝의 성공 조건을 정의한다. 가장 간단한 형태는 특정 목표 상태에 도달하는 것이지만, OMPL은 더 유연한 목표 정의를 지원한다. 예를 들어, 목표 상태에 특정 허용 오차(tolerance) 반경 내로 진입하는 것, 설정 공간 내의 특정 영역(region)에 도달하는 것, 또는 사용자가 정의한 복잡한 조건을 만족하는 상태 집합에 도달하는 것 등이 가능하다.14

### 2.3  플래닝 프로세스의 구성 요소

위에서 정의된 3요소를 바탕으로, 실제 플래닝 프로세스를 구동하기 위한 추가적인 구성 요소들이 존재한다.

- **`ompl::base::SpaceInformation`:** `StateSpace`, `StateValidityChecker` 등 플래닝에 필요한 모든 기본 정보들을 하나로 묶어 관리하는 통합 클래스다. 플래너는 이 `SpaceInformation` 객체를 통해 상태 공간의 속성을 파악하고 상태의 유효성을 검사하는 등 필요한 모든 연산을 수행한다. 이는 플래너와 문제 정의 사이의 주된 인터페이스 역할을 한다.14
- **`ompl::base::ProblemDefinition`:** 해결하고자 하는 구체적인 플래닝 "인스턴스"를 명시한다. 여기에는 하나 이상의 시작 상태(start states), 목표 정의(`Goal`), 그리고 선택적으로 경로의 품질을 평가하기 위한 최적화 목적(`OptimizationObjective`)이 포함된다.14
- **`ompl::base::Planner`:** `SpaceInformation`과 `ProblemDefinition`을 입력으로 받아, `solve()` 메서드를 통해 실제 경로 탐색 알고리즘을 수행하는 주체다. OMPL에 구현된 모든 플래너(예: `ompl::geometric::RRT`, `ompl::geometric::PRM`)는 이 `ompl::base::Planner` 추상 기반 클래스를 상속받아 구현된다. 플래너는 자신이 어떤 종류의 상태 공간에서 작업하는지 알 필요 없이, `SpaceInformation`이 제공하는 추상화된 인터페이스에만 의존하여 작동한다.14
- **`ompl::base::OptimizationObjective`:** 경로의 "비용(cost)" 또는 "품질(quality)"을 정량적으로 정의하는 객체다. 기본적으로는 경로의 길이를 비용으로 사용하지만(`PathLengthOptimizationObjective`), 사용자는 장애물로부터의 최소 거리를 최대화하거나(`MaximizeMinClearanceObjective`), 로봇이 경로를 따라 움직일 때 소모되는 기계적인 일(`MechanicalWorkOptimizationObjective`)을 최소화하는 등 다양한 최적화 기준을 정의하고 적용할 수 있다. RRT*나 PRM*과 같은 최적화 플래너(optimizing planner)들은 이 `OptimizationObjective`를 참조하여 더 나은 품질의 경로를 탐색한다.8

### 2.4  `SimpleSetup`: 사용자 편의를 위한 통합 클래스

앞서 설명한 여러 구성 요소들을 개별적으로 생성하고 설정하는 과정은 번거로울 수 있다. 이를 위해 OMPL은 `ompl::geometric::SimpleSetup` (운동학적 플래닝용) 및 `ompl::control::SimpleSetup` (동역학적 플래닝용)이라는 유틸리티 클래스를 제공한다. `SimpleSetup` 클래스는 플래닝 문제 설정에 필요한 상용구 코드(boilerplate code)를 상당 부분 자동화해준다.

사용자는 `StateSpace`와 `StateValidityChecker`와 같은 최소한의 정보만 `SimpleSetup` 객체에 제공하면 된다. 그러면 `SimpleSetup`이 내부적으로 `SpaceInformation`, `ProblemDefinition` 등의 객체를 자동으로 생성하고 올바르게 연결해준다. 또한 플래너 설정, 시작 및 목표 상태 설정, 플래닝 실행(`solve()`), 결과 경로 확인(`getSolutionPath()`) 등의 과정을 간결한 API를 통해 수행할 수 있게 해준다.14 이는 OMPL을 독립적으로 사용하여 빠르게 프로토타이핑하거나 특정 문제를 테스트할 때 매우 유용하다.

## 3.  `ompl_planning.yaml`을 통한 상세 설정 및 튜닝

OMPL의 이론적 개념과 구성 요소들은 ROS2 Humble 환경에서 `ompl_planning.yaml`이라는 구체적인 설정 파일을 통해 제어된다. 이 파일은 MoveIt2 사용자가 OMPL의 강력하고 다양한 기능들을 자신의 로봇과 애플리케이션에 맞게 조정할 수 있는 핵심적인 인터페이스다. 각 파라미터가 플래닝의 속도, 경로의 품질, 그리고 안정성에 미치는 영향을 깊이 이해하는 것은 OMPL을 효과적으로 활용하기 위한 필수 과정이다.

### 3.1  `ompl_planning.yaml` 파일 구조

MoveIt 설정 어시스턴트(`moveit_setup_assistant`)를 통해 생성된 `moveit_config` 패키지 안에는 `config/ompl_planning.yaml` 파일이 포함되어 있다. 이 파일의 구조는 계층적이며 직관적으로 설계되어 있다.

최상위 레벨에서는 로봇의 플래닝 그룹(planning group), 예를 들어 매니퓰레이터 팔을 지칭하는 `panda_arm`이나 그리퍼를 의미하는 `hand` 등으로 설정이 구분된다.18 각 플래닝 그룹은 독립적인 플래닝 파라미터를 가질 수 있다.

각 그룹 내부에는 두 가지 주요 섹션이 존재한다:

- `default_planner_config`: 해당 그룹에 대해 플래닝 요청 시 플래너가 명시적으로 지정되지 않았을 때 사용될 기본 플래너 설정을 지정한다.
- `planner_configs`: 해당 그룹에서 사용 가능한 모든 플래너 설정들의 목록을 정의한다. 각 설정은 고유한 이름(예: `RRTConnectkConfigDefault`, `RRTstarkConfigDefault`)을 가지며, 이 이름은 코드나 RViz에서 특정 플래너를 호출할 때 사용된다.18

각 개별 플래너 설정은 `type` 필드를 통해 사용할 OMPL 플래너의 클래스 이름을 지정하고(예: `geometric::RRTConnect`), 그 외에는 해당 플래너의 동작을 제어하는 다양한 파라미터들을 포함한다.18

### 3.2  공통 및 핵심 파라미터 분석

대부분의 샘플링 기반 플래너, 특히 RRT 계열에서 공통으로 사용되는 핵심 파라미터들은 다음과 같으며, 이들의 상호작용을 이해하는 것이 튜닝의 첫걸음이다.

- `type`: 사용할 OMPL 플래너의 전체 클래스 이름을 지정한다. 예를 들어, `geometric::RRTConnect`는 운동학적(kinematic) 플래닝을 위한 RRT-Connect 플래너를 의미한다. `geometric` 네임스페이스는 OMPL 내에서 기하학적 플래너들을 그룹화하는 데 사용된다.19
- `range`: 트리 기반 플래너(RRT, RRT*, RRT-Connect 등)가 한 번의 확장 단계에서 전진할 수 있는 최대 거리를 상태 공간 단위로 정의한다. 이 값은 탐색의 '보폭'에 해당하며, 플래닝 성능에 직접적인 영향을 미친다.
  - **작은 `range` 값:** 트리가 더 촘촘하게 성장하여 좁은 통로를 통과할 확률이 높아지고 생성된 경로가 더 부드러워지는 경향이 있다. 하지만 전체 공간을 탐색하는 데 더 많은 반복이 필요하므로 플래닝 시간이 길어진다.20
  - **큰 `range` 값:** 트리가 더 빨리 넓은 영역으로 뻗어 나가므로 초기 해를 찾는 속도가 빨라질 수 있다. 그러나 너무 크면 장애물이 많은 복잡한 환경이나 좁은 통로 주변에서 유효한 확장을 생성하기 어려워 플래닝에 실패할 수 있다.20
- `goal_bias`: 샘플링 과정에서 완전히 무작위로 상태를 선택하는 대신, 명시된 목표 상태를 직접 샘플로 선택할 확률을 지정한다. 이 값은 0.0에서 1.0 사이이며, 일반적으로 0.05에서 0.1 사이의 작은 값이 권장된다.20
  - **적절한 `goal_bias`:** 트리가 목표 방향으로 탐색을 집중하도록 유도하여 수렴 속도를 높이는 데 도움이 된다.
  - **과도한 `goal_bias`:** 트리가 목표를 향해 너무 탐욕적(greedy)으로만 확장하게 만들어, 복잡한 장애물(예: 'C'자 형태의 장애물) 뒤에 목표가 있을 경우 지역 최적해(local minima)에 빠져 해를 찾지 못하게 될 위험이 있다.
- `longest_valid_segment_fraction` & `maximum_waypoint_distance`: OMPL은 기본적으로 경로의 유효성을 검사할 때, 경로를 구성하는 두 웨이포인트(상태) 사이의 모든 지점을 검사하는 연속적인 충돌 검사를 수행하지 않는다. 대신, 두 웨이포인트가 모두 유효하고 그 사이의 거리가 특정 임계값보다 작으면, 그 사이의 경로 또한 유효하다고 가정한다. 이 임계값을 설정하는 파라미터가 바로 이 두 가지다.19
  - `longest_valid_segment_fraction`: 상태 공간의 전체 범위에 대한 비율로 임계값을 설정한다. 예를 들어 0.01은 상태 공간 대각선 길이의 1%를 의미한다.
  - `maximum_waypoint_distance`: 상태 공간 거리의 절대값으로 임계값을 설정한다.
  - 이 값들은 안전성과 성능 간의 중요한 트레이드오프를 결정한다. 너무 큰 값(느슨한 검사)은 얇거나 작은 장애물과의 충돌을 놓칠 위험을 증가시키고, 너무 작은 값(촘촘한 검사)은 충돌 검사 횟수를 급격히 늘려 플래닝 시간을 매우 느리게 만든다.13

성능 튜닝은 단순히 개별 파라미터를 조정하는 것을 넘어, 이들 간의 복잡한 상호작용을 이해하는 과정이다. 예를 들어, `range` 값을 키워 탐색 속도를 높이는 공격적인 전략을 선택했다면, 두 노드 사이의 거리가 멀어지므로 그 사이에 위치한 장애물과의 충돌을 놓칠 위험이 커진다. 이러한 위험을 상쇄하기 위해서는 `longest_valid_segment_fraction` 값을 더 작게 설정하여 충돌 검사의 정밀도를 높이는 보수적인 전략을 함께 취해야 한다. 이처럼 플래닝의 '속도', '경로 품질', '안전성'이라는 세 가지 상충하는 목표 사이에서, 주어진 로봇과 작업 환경에 맞는 최적의 균형점을 찾는 것은 과학적 분석과 경험적 직관이 모두 요구되는 섬세한 과정이다.

### 3.3  경로 최적화를 위한 고급 설정

단순히 가능한 경로를 찾는 것을 넘어, '더 좋은' 경로를 찾기 위해 OMPL은 다양한 최적화 기능을 제공하며, 이는 `ompl_planning.yaml`을 통해 설정할 수 있다.

- `optimization_objective`: RRT*, PRM*, FMT*와 같은 점근적 최적(asymptotically optimal) 플래너들이 어떤 기준에 따라 경로를 최적화할지를 명시적으로 지정한다.19 이 파라미터를 설정하지 않으면 기본적으로 경로 길이 최소화가 적용된다.
  - `PathLengthOptimizationObjective`: 경로의 총 길이를 최소화하는 가장 일반적인 목적 함수다.8
  - `MaximizeMinClearanceObjective`: 경로상의 모든 지점에서 장애물과의 최소 거리를 최대화한다. 이는 로봇이 장애물로부터 최대한 멀리 떨어져 움직이는, 더 '안전한' 경로를 생성하는 데 유용하다.8
  - 기타 목적 함수: 로봇의 움직임에 필요한 `MechanicalWorkOptimizationObjective`나, 상태 자체에 비용을 부여하는 `StateCostIntegralObjective` 등 다양한 기준을 적용할 수 있다.8
- `AnytimePathShortening`: 단일 플래너가 아닌, 여러 플래너를 병렬로 실행하고 그 결과들을 결합하여 경로를 개선하는 메타 알고리즘이다. 이 플래너는 경로 단축(path shortcutting)과 경로 하이브리드화(path hybridization) 기법을 반복적으로 적용하여, 주어진 시간 내에 점진적으로 더 나은 품질의 경로를 찾아낸다. 이론적인 최적성을 보장하지는 않지만, 실제 많은 문제에서 실용적으로 우수한 품질의 경로를 빠르게 찾는 데 효과적이다.19

### 3.4  플래너 종료 조건 및 후처리

플래닝 프로세스의 종료 시점과 결과 경로의 최종 품질은 다음 파라미터들에 의해 결정된다.

- `allowed_planning_time`: `move_group`에 의해 플래너에게 주어지는 최대 허용 시간(초)이다. OMPL 플래너는 이 시간 내에 해를 찾아야 한다.19

- `termination_condition`: 시간 제한 외에 플래닝을 조기에 종료할 수 있는 추가적인 조건을 설정한다. 플래너는 시간 제한과 이 조건 중 어느 것이든 먼저 만족되면 종료된다.19

  - `Iteration[num]`: 지정된 반복 횟수에 도달하면 종료한다. 플래닝 시간을 예측하기 어려울 때 일관된 탐색량을 보장하는 데 유용하다.
  - `CostConvergence`: 최적화 플래너가 생성하는 해의 비용이 더 이상 개선되지 않고 수렴 상태에 도달했다고 판단되면 종료한다. 이는 불필요한 추가 계산을 방지하여 효율성을 높인다. (OMPL 1.5.0 이상에서 사용 가능)
  - `ExactSolution`: 최적화 플래너가 '최초의' 유효한 해를 찾자마자 즉시 종료하도록 한다. 이는 RRT*와 같은 플래너의 점진적 개선(anytime) 특성을 비활성화하고, RRT-Connect처럼 가장 빠른 해를 원할 때 사용될 수 있다.

- **후처리 스무딩 (Post-Processing Smoothing):** 많은 샘플링 기반 플래너가 생성하는 초기 경로는 불필요한 굴곡이 많고 각진 형태를 띤다. MoveIt2는 `allowed_planning_time` 중 플래닝이 일찍 끝나고 남은 시간을 활용하여, 이 경로를 더 부드럽고 짧게 만드는 후처리 스무딩 단계를 자동으로 수행한다.19 또한, 

  `num_planning_attempts` 파라미터를 1보다 큰 값으로 설정하면 MoveIt2는 여러 번의 플래닝을 시도하고, 그 결과로 나온 다양한 경로들의 좋은 부분들을 조합(hybridization)하여 더 우수한 품질의 최종 경로를 생성할 수 있다.19

아래 표는 `ompl_planning.yaml`에서 사용되는 주요 파라미터들을 요약하고, 그 기능과 성능에 미치는 영향을 정리한 것이다.

| 파라미터 이름                    | 설명                                   | 적용 플래너             | 일반적인 값                          | 성능 영향                                                    |
| -------------------------------- | -------------------------------------- | ----------------------- | ------------------------------------ | ------------------------------------------------------------ |
| `type`                           | 사용할 OMPL 플래너 클래스 지정         | 모든 플래너             | `geometric::RRTConnect` 등           | 플래닝의 기본 전략(속도, 최적성 등)을 결정한다.              |
| `range`                          | 트리 확장 시 최대 전진 거리            | RRT 계열                | 0.0 ~ (상태 공간에 따라 다름)        | **(+)**: 값이 크면 탐색 속도 증가. **(-)**: 값이 크면 좁은 통로 통과 실패율 증가. |
| `goal_bias`                      | 목표 상태를 직접 샘플링할 확률         | RRT 계열                | 0.0 ~ 1.0 (주로 0.05)                | **(+)**: 수렴 속도 향상. **(-)**: 값이 크면 지역 최적해에 빠질 위험 증가. |
| `longest_valid_segment_fraction` | 경로 유효성 검사 시 최대 간격 (비율)   | 모든 플래너             | 0.001 ~ 0.1                          | **(+)**: 값이 크면 충돌 검사 감소로 속도 향상. **(-)**: 값이 크면 충돌 놓칠 위험 증가. |
| `maximum_waypoint_distance`      | 경로 유효성 검사 시 최대 간격 (절대값) | 모든 플래너             | 0.0 ~ (상태 공간에 따라 다름)        | `longest_valid_segment_fraction`과 유사. 더 보수적인 쪽이 선택된다. |
| `projection_evaluator`           | 저차원 투영에 사용할 관절/링크 지정    | KPIECE, PDST 계열       | `joints(j1, j2)`                     | 특정 플래너의 탐색 효율성을 높이는 데 사용된다.              |
| `optimization_objective`         | 경로 최적화 기준                       | 최적화 플래너 (RRT* 등) | `PathLengthOptimizationObjective` 등 | 경로의 품질(길이, 안전성 등)을 결정한다.                     |
| `termination_condition`          | 추가적인 플래너 종료 조건              | 최적화 플래너           | `CostConvergence[...]` 등            | 불필요한 계산을 줄여 플래닝 효율성을 높일 수 있다.           |

## 4.  주요 샘플링 기반 플래너 심층 분석

OMPL은 수십 가지의 다양한 플래닝 알고리즘을 제공하지만, 실제 MoveIt2 환경에서는 몇 가지 핵심적인 플래너들이 주로 사용된다. 이 장에서는 그중 가장 대표적인 RRT, PRM 계열 및 최신 최적화 플래너인 FMT*의 내부 작동 원리를 수학적, 알고리즘적 관점에서 깊이 있게 탐구한다. 각 플래너가 가진 고유의 철학과 특성을 이해하는 것은, 주어진 플래닝 문제에 가장 적합한 도구를 전략적으로 선택하기 위한 필수적인 과정이다.

### 4.1  RRT (Rapidly-exploring Random Tree) 계열

RRT 계열 알고리즘들은 단일 질의(single-query) 문제, 즉 주어진 시작점과 목표점에 대한 하나의 경로를 찾는 데 특화되어 있다. 이들은 트리를 점진적으로 확장해 나가며 설정 공간을 탐색하는 공통된 구조를 가진다.

#### 4.1.1  기본 RRT

- **원리:** RRT 알고리즘의 작동 원리는 직관적이고 간단하다. 시작점(`$q_{start}$`)을 루트 노드로 하는 트리(`$\mathcal{T}$`)로 시작하여 다음 과정을 반복한다.22

  1. **샘플링(Sampling):** 설정 공간 `$\mathcal{C}$`에서 임의의 상태 `$q_{rand}$`를 균일하게 샘플링한다.

  2. **최근접 이웃 탐색(Nearest Neighbor):** 현재 트리 `$\mathcal{T}$`에 있는 모든 노드 중에서 `$q_{rand}$`와 가장 가까운 노드 `$q_{near}$`를 찾는다. 여기서 '거리'는 상태 공간에 정의된 거리 척도(metric)를 사용한다. `$q_{near} = \arg\min_{q \in \mathcal{T}} \text{distance}(q, q_{rand})$`.

  3. **확장(Steering):** `$q_{near}$`에서 `$q_{rand}$` 방향으로 미리 정의된 최대 거리(`range`, `$\epsilon$`)만큼 전진하여 새로운 상태 `$q_{new}$`를 생성한다. 만약 `$q_{near}$`와 `$q_{rand}$` 사이의 거리가 `$\epsilon$`보다 작으면 `$q_{new} = q_{rand}$`가 된다.

  4. 유효성 검사 및 추가(Validation & Addition): $q_{near}$에서 $q_{new}$로 이동하는 경로가 충돌이 없는 경우, $q_{new}$를 새로운 노드로 트리에 추가하고 $q_{near}$와 $q_{new}$를 엣지로 연결한다.

     이 과정은 트리의 노드 중 하나가 목표 영역에 도달하거나 지정된 시간이 초과될 때까지 계속된다.

- **장점:** RRT의 가장 큰 장점은 탐색의 효율성이다. 트리는 자연스럽게 탐색이 덜 된 넓은 공간으로 뻗어 나가려는 경향(Voronoi bias)을 보이므로, 고차원 공간을 매우 빠르게 탐색할 수 있다.26 또한, 구현이 비교적 간단하다.

- **단점:** RRT는 경로의 최적성을 전혀 고려하지 않는다. 따라서 생성된 경로는 매우 각지고 비효율적이며, 실행할 때마다 결과가 달라지는 등 성능 편차가 크다.27 특히, 설정 공간 내의 좁은 통로(narrow passage) 영역은 샘플링될 확률이 낮아 통과에 어려움을 겪는 고질적인 문제가 있다.24

#### 4.1.2  RRT-Connect

- **원리:** RRT-Connect는 기본 RRT의 느린 수렴 속도를 개선하기 위해 양방향 탐색(bidirectional search)과 탐욕적 휴리스틱(greedy heuristic)을 결합한 알고리즘이다. 시작점(`$q_{start}$`)에서 시작하는 트리 `$\mathcal{T}_a$`와 목표점(`$q_{goal}$`)에서 시작하는 트리 `$\mathcal{T}_b$`를 동시에 확장한다.25

  

  알고리즘은 두 트리를 번갈아 가며 확장한다. 예를 들어, $\mathcal{T}_a$를 확장할 차례가 되면, 임의의 샘플 $q_{rand}$를 생성하고 $\mathcal{T}_a$에서 가장 가까운 노드 $q_{near}$를 찾아 $q_{new}$를 생성한다. 그 후, 단순히 $q_{new}$를 추가하는 데 그치지 않고, 다른 트리인 $\mathcal{T}_b$가 $q_{new}$를 향해 '연결(Connect)'을 시도한다. 이 연결 과정은 $\mathcal{T}_b$가 $q_{new}$에 도달하거나 장애물에 막힐 때까지 반복적으로 확장을 시도하는 탐욕적인 방식으로 이루어진다. 만약 두 트리가 성공적으로 만나면, 두 트리를 이어붙여 최종 경로를 생성한다.

- **장점:** 양쪽에서 동시에 탐색이 진행되므로, 특히 시작점과 목표점 사이에 복잡한 장애물 군집이 있는 경우 단방향 RRT에 비해 훨씬 빠른 속도로 해를 찾는다. 두 트리가 '중간에서 만날' 확률이 높기 때문이다.25 이러한 높은 효율성 덕분에 MoveIt2에서 가장 널리 사용되는 기본 플래너 중 하나로 채택되었다.18

- **단점:** 속도에 초점을 맞춘 '만족해(satisficing)' 플래너로서, 경로의 최적성은 보장하지 않는다. 생성된 경로는 여전히 비효율적일 수 있다.32

#### 4.1.3  RRT*

- **원리:** RRT*는 RRT에 점근적 최적성(asymptotic optimality) 보장을 추가한 변형 알고리즘이다. 이는 샘플 수가 무한대로 갈수록 생성된 경로의 비용이 최적 경로의 비용으로 수렴함을 의미한다. RRT*는 기본 RRT의 확장 과정에 두 가지 핵심적인 최적화 단계를 추가한다.25

  1. **최적 부모 선택(Choose Parent):** 새로운 노드 `$q_{new}$`가 생성되면, 기본 RRT처럼 가장 가까운 `$q_{near}$`를 무조건 부모로 삼지 않는다. 대신, `$q_{new}$` 주변의 일정 반경 내에 있는 모든 이웃 노드(`$\mathcal{Q}_{near}$`)를 탐색한다. 그리고 그 이웃 노드들 중에서, 해당 노드를 경유하여 시작점(`$q_{start}$`)으로부터 `$q_{new}$`까지 도달하는 총 경로 비용(cost)을 최소화하는 노드 `$q_{min}$`을 찾아 `$q_{new}$`의 부모로 연결한다.

  $$
  q_{min} = \arg\min_{q' \in \mathcal{Q}_{near}} (\text{Cost}(q') + \text{Cost}(q', q_{new}))
  $$

  1. **재연결(Rewiring):** `$q_{new}$`가 트리에 추가된 후, 다시 주변 이웃 노드 `$q' \in \mathcal{Q}_{near}$`들을 검토한다. 만약 기존 경로 대신 `$q_{new}$`를 경유하여 `$q'$`에 도달하는 경로의 비용이 더 저렴하다면, `$q'$`의 기존 부모-자식 관계를 끊고 `$q_{new}$`를 새로운 부모로 연결하는 '재연결' 과정을 수행한다.

  $$
  \text{if } \text{Cost}(q_{new}) + \text{Cost}(q_{new}, q') < \text{Cost}(q') \text{ then } \text{parent}(q') \leftarrow q_{new}
  $$

  **장점:** 충분한 플래닝 시간이 주어지면 최적 경로로 수렴함이 수학적으로 증명되었다.36 초기 해를 찾은 후에도 플래닝을 계속 진행하면 점진적으로 경로의 품질이 개선되는 '애니타임(anytime)' 알고리즘으로서의 특성을 가진다.

- **단점:** 최적 부모 선택과 재연결 과정에서 다수의 최근접 이웃 탐색과 추가적인 충돌 검사가 필요하다. 이로 인해 기본 RRT나 RRT-Connect에 비해 계산 복잡도가 훨씬 높고, 초기 해를 찾는 속도 및 전체적인 수렴 속도가 현저히 느리다.28

RRT-Connect와 RRT*의 비교는 샘플링 기반 플래닝이 직면한 근본적인 딜레마, 즉 '탐색(exploration)'과 '최적화(optimization)' 사이의 상충 관계를 명확히 보여준다. RRT-Connect는 '어떻게든 길을 찾는 것'에 집중하여 속도를 극대화하는 반면, RRT*는 '가장 좋은 길을 찾는 것'에 집중하여 경로 품질을 보장하지만 속도를 희생한다. 이 둘 사이의 선택은 단순히 기술적 선호의 문제가 아니라, 애플리케이션의 핵심 요구사항(예: 실시간 반응성이 중요한 동적 장애물 회피 vs. 에너지 효율이 중요한 반복적인 제조 공정)에 따라 결정되어야 하는 근본적인 설계상의 결정이다.

### 4.2  PRM (Probabilistic Roadmap) 계열

PRM 계열 알고리즘들은 주로 다중 질의(multi-query) 문제, 즉 동일한 환경 내에서 여러 다른 시작점-목표점 쌍에 대한 경로를 반복적으로 찾아야 하는 상황에 적합하다.

#### 4.2.1  기본 PRM

- **원리:** PRM은 명시적인 두 단계로 구성된다.25
  1. **학습/구축 단계(Learning/Construction Phase):** 설정 공간 전역에 걸쳐 다수의 무작위 샘플, 즉 '마일스톤(milestones)'을 생성한다. 각 마일스톤에 대해 충돌 여부를 검사하여 유효한 마일스톤들만 유지한다. 그 후, 서로 일정 거리 내에 있는 마일스톤 쌍들에 대해, 그들을 직접 잇는 '지역 경로(local path)'(주로 직선)가 충돌이 없는지 검사한다. 충돌이 없다면 두 마일스톤을 엣지로 연결하여 그래프, 즉 '로드맵'을 구축한다.
  2. **질의 단계(Query Phase):** 새로운 경로 탐색 요청이 들어오면, 주어진 시작점 `$q_{start}$`와 목표점 `$q_{goal}$`을 로드맵 상의 가장 가까운 마일스톤에 각각 연결한다. 그 후, 로드맵 그래프 상에서 두 연결 지점 사이의 최단 경로를 다익스트라나 A*와 같은 그래프 탐색 알고리즘을 이용해 찾는다.
- **장점:** 한 번 로드맵을 구축해두면, 질의 단계는 매우 빠른 속도로 수행된다. 따라서 공장 자동화와 같이 정적인 환경에서 반복적으로 다른 경로를 계획해야 하는 시나리오에 매우 효과적이다.42
- **단점:** 초기 로드맵 구축 단계에 상당한 계산 시간이 소요될 수 있다. 또한, RRT와 마찬가지로 좁은 통로 영역은 샘플링될 확률이 낮아, 로드맵이 여러 개의 연결되지 않은 컴포넌트(disconnected components)로 나뉠 수 있다. 이 경우, 시작점과 목표점이 서로 다른 컴포넌트에 속하면 해를 찾지 못하는 문제가 발생한다.43

#### 4.2.2  PRM* 및 Lazy 변형

- **PRM\*:** 점근적 최적성을 보장하기 위해, 샘플 수가 증가함에 따라 각 마일스톤을 연결하려는 이웃 탐색 반경(또는 이웃의 수)을 점진적으로 늘려나가는 전략을 사용한다. 이는 `$n \to \infty$`일 때 로드맵이 설정 공간의 연결성을 완벽하게 반영하도록 하여 최적 경로로의 수렴을 보장한다.25
- **LazyPRM / LazyPRM\*:** 이 변형들은 계산 비용이 가장 큰 충돌 검사를 최대한 지연시키는 '게으른 평가(lazy evaluation)' 전략을 사용한다. 로드맵 구축 단계에서는 충돌 여부를 전혀 검사하지 않고, 모든 잠재적 마일스톤과 엣지를 일단 로드맵에 추가한다. 질의 단계에서 A* 등으로 최단 경로 후보를 찾은 뒤, 비로소 그 경로를 구성하는 마일스톤과 엣지들에 대해서만 충돌 검사를 수행한다. 만약 충돌이 발견되면, 해당 노드나 엣지를 로드맵에서 제거하고 다른 경로를 다시 탐색한다. 이 방식은 설정 공간의 대부분이 비어있어 충돌 확률이 낮은 문제에서 충돌 검사 횟수를 극적으로 줄여 전체적인 플래닝 시간을 단축시킨다.25

### 4.3  FMT* (Fast Marching Tree)

- **원리:** FMT*는 PRM의 초기 일괄 샘플링 방식과 RRT*의 점진적 트리 확장 및 최적화 방식을 결합한 하이브리드 알고리즘이다.
  1. 먼저 PRM처럼 설정 공간에 `$N$`개의 샘플을 미리 생성한다.
  2. 그 후, 'Fast Marching' 기법에 영감을 받아, 다익스트라 알고리즘과 유사하게 비용 우선순위 큐(priority queue)를 사용하여 항상 비용(cost-to-come)이 가장 낮은 '열린(open)' 노드를 선택하여 트리를 확장해 나간다.
  3. 새로운 노드를 트리에 연결할 때, RRT*처럼 주변 이웃 중에서 최적의 부모를 선택한다. 하지만 RRT*와 달리, 한 번 최적 경로가 결정된 노드는 다시 방문하지 않는 'one-pass' 특성을 가진다. 충돌 검사 또한 연결이 실제로 고려될 때만 수행하는 게으른 방식을 채택한다.47
- **장점:** RRT*와 PRM*의 장점을 효과적으로 결합하여, 두 알고리즘보다 더 빠른 속도로 최적 해에 수렴하는 것으로 알려져 있다. 특히 장애물이 많고 복잡한 고차원 환경에서 뛰어난 성능을 보인다.48
- **단점:** 알고리즘의 개념과 구현이 RRT*나 PRM*에 비해 상대적으로 복잡하다.

아래 표는 본 장에서 분석한 주요 플래닝 알고리즘들의 핵심적인 특성을 비교 요약한 것이다.

| 플래너          | 기본 패러다임      | 질의 유형 | 최적성 보장 | 핵심 장점                                  | 핵심 단점                             | 대표적인 ROS2 Humble 사용 사례                               |
| --------------- | ------------------ | --------- | ----------- | ------------------------------------------ | ------------------------------------- | ------------------------------------------------------------ |
| **RRT**         | 트리 기반          | 단일 질의 | 아니요      | 빠른 탐색 속도, 고차원 공간에 효율적       | 경로 품질 낮음, 좁은 통로에 취약      | 동적 장애물 회피를 위한 빠른 초기 경로 생성                  |
| **RRT-Connect** | 트리 기반 (양방향) | 단일 질의 | 아니요      | 매우 빠른 수렴 속도, MoveIt2 기본 플래너   | 경로 품질 낮음, 최적성 보장 안됨      | 대부분의 일반적인 단일 경로 계획 문제                        |
| **RRT***        | 트리 기반          | 단일 질의 | 점근적 최적 | 경로 품질 우수, 애니타임(Anytime) 특성     | 계산 비용 높음, 수렴 속도 느림        | 에너지 효율이 중요한 반복 작업의 오프라인 경로 최적화        |
| **PRM**         | 로드맵 기반        | 다중 질의 | 아니요      | 다중 질의에 매우 빠름, 환경 재사용         | 초기 구축 비용 높음, 좁은 통로에 취약 | 정적 환경(예: 공장)에서 다양한 작업을 위한 경로 라이브러리 구축 |
| **LazyPRM**     | 로드맵 기반        | 다중 질의 | 아니요      | 충돌 검사 최소화로 매우 빠름(특히 빈 공간) | 복잡한 환경에서는 비효율적일 수 있음  | 장애물이 적고 넓은 공간에서의 다중 질의                      |
| **FMT***        | 하이브리드         | 단일 질의 | 점근적 최적 | RRT*보다 빠른 최적 경로 수렴 속도          | 알고리즘 복잡도 높음                  | 복잡하고 장애물이 많은 환경에서의 고품질 경로 계획           |

## 5.  고급 주제 및 성능 고찰

샘플링 기반 플래닝 알고리즘은 고차원 공간 문제에 대한 실용적인 해법을 제시했지만, 그 자체로 모든 문제를 해결하는 만병통치약은 아니다. 이 장에서는 샘플링 기반 플래너들이 공통적으로 직면하는 근본적인 난제들, 즉 '차원의 저주'와 '좁은 통로 문제'를 심도 있게 다룬다. 또한, OMPL 내에서 이러한 문제들을 완화하기 위한 접근법을 살펴보고, 주요 알고리즘들의 이론적 계산 복잡도를 분석하여 성능의 한계를 명확히 한다.

### 5.1  차원의 저주와 고차원 공간에서의 플래닝

- **문제 정의:** '차원의 저주(curse of dimensionality)'는 로봇의 자유도(dimension, `$d$`)가 증가함에 따라 설정 공간의 부피가 `$L^d$` (여기서 `$L$`은 각 차원의 길이)로 기하급수적으로 커지는 현상을 지칭한다. 이로 인해 발생하는 문제는 다음과 같다.4

  1. **샘플링 밀도 저하:** 동일한 수의 샘플을 사용하더라도, 차원이 높아질수록 샘플들은 공간 내에서 점점 더 희소(sparse)해진다. 공간을 충분히 촘촘하게 커버하기 위해 필요한 샘플의 수는 차원에 따라 폭발적으로 증가한다.

  2. **'가장자리' 현상:** 고차원 공간에서는 부피의 대부분이 단위 초입방체의 '가장자리' 근처에 집중된다. 이는 무작위 샘플이 대부분 공간의 중심부가 아닌 외곽에 위치하게 만들어, 내부의 복잡한 구조를 탐색하기 어렵게 만든다.

  3. 거리 척도의 무의미화: 고차원 공간에서는 모든 점들이 서로 거의 비슷한 거리에 위치하게 되는 경향이 있어, '최근접 이웃'이라는 개념이 무의미해지고 탐색의 방향성을 잃기 쉽다.

     이러한 현상들은 그리드 기반 탐색을 수치적으로 불가능하게 만들 뿐만 아니라, 샘플링 기반 방법의 효율성 또한 심각하게 저하시킨다.

- **OMPL의 대응 전략 - 다단계 플래닝 (Multilevel Planning):** OMPL은 이 근본적인 난제에 대응하기 위해 '다단계 플래닝'이라는 정교한 프레임워크를 제공한다. 이 접근법의 핵심 아이디어는 고차원의 복잡한 문제를 직접 해결하는 대신, 문제를 더 단순한 여러 개의 저차원 '추상 공간(abstraction levels)'으로 분해하여 계층적으로 해결하는 것이다.12

  예를 들어, 양팔을 가진 휴머노이드 로봇이 긴 복도를 통과하는 문제를 생각해보자. 복도를 통과하는 동안 팔의 정밀한 자세는 중요하지 않을 수 있다. 이 경우, 다단계 플래닝은 다음과 같이 작동할 수 있다.

  1. **추상 레벨 1:** 로봇의 팔을 없는 것으로 간주하고, 몸통의 이동(예: 3D 위치 및 1D 회전)에 대해서만 경로를 계획한다. 이는 저차원 문제이므로 매우 빠르게 해결할 수 있다.

  2. 추상 레벨 2: 레벨 1에서 찾은 몸통 경로를 '가이드' 삼아, 이제 팔의 움직임을 포함한 전체 자유도에 대한 경로를 탐색한다. 탐색 공간이 저차원 해에 의해 크게 제약을 받으므로, 전체 공간을 무작위로 탐색하는 것보다 훨씬 효율적이다.

     OMPL은 이러한 추상화를 사용자가 직접 정의할 수 있는 유연한 인터페이스를 제공하며, QRRT*, QMP*와 같은 다단계 플래너들은 이 계층적 구조를 활용하여 기존 플래너들이 수십 분 이상 걸리거나 아예 풀지 못하는 100차원 이상의 문제도 수 초 내에 해결하는 성능을 보여준다.51

### 5.2  좁은 통로 문제 (The Narrow Passage Problem)

- **문제 정의:** '좁은 통로 문제'는 설정 공간 내에서 해 경로가 반드시 통과해야 하는 매우 좁은 영역이 존재할 때 발생하는 어려움을 말한다. 이 좁은 통로 영역은 전체 설정 공간에서 차지하는 부피가 극히 작기 때문에, 균일한 무작위 샘플링으로 해당 영역 내에 샘플이 생성될 확률이 매우 낮다. 이로 인해 플래너는 통로를 '발견'하지 못하고 플래닝에 실패하거나, 우연히 통로를 찾을 때까지 엄청나게 많은 샘플을 생성해야 하므로 플래닝 시간이 기하급수적으로 길어진다.24
- **해결을 위한 접근법:** 이 문제를 해결하기 위한 연구는 주로 샘플링 전략을 개선하여 좁은 통로와 같은 '중요한' 영역에 샘플을 의도적으로 집중시키는 방향으로 진행되어 왔다.
  - **장애물 근처 샘플링 (Obstacle-Based Sampling):** 좁은 통로는 정의상 장애물 경계에 의해 형성된다. 이 점에 착안하여, 먼저 임의의 샘플을 생성하고 만약 그 샘플이 장애물 내부에 있다면, 그 샘플을 장애물 표면으로 이동시키거나 그 주변의 자유 공간에 새로운 샘플을 생성하는 전략이다. 이는 장애물 경계 근처의 샘플 밀도를 높여 좁은 통로를 발견할 확률을 증가시킨다.3
  - **브릿지 테스트 (Bridge Test):** 이 기법은 좁은 통로의 기하학적 특성을 직접적으로 활용한다. 무작위로 두 점을 선택하고, 만약 두 점이 모두 장애물 내부에 있지만 그들을 잇는 직선의 중간 지점은 자유 공간에 있다면, 그 중간 지점은 좁은 통로 내부에 있을 가능성이 매우 높다고 판단한다. 이 중간 지점을 유효한 샘플로 추가함으로써 통로 내부를 효과적으로 샘플링할 수 있다.54
  - **머신러닝 기반 편향 샘플링 (Learning-Based Biased Sampling):** 최근 가장 활발히 연구되는 접근법으로, 과거의 성공적인 플래닝 경험 데이터로부터 '좋은' 경로가 존재할 가능성이 높은 영역의 확률 분포를 학습한다. 예를 들어, 변분 오토인코더(Variational Autoencoder, VAE)와 같은 생성 모델을 사용하여, 주어진 시작점, 목표점, 장애물 배치에 대해 유망한 샘플링 분포를 생성할 수 있다. 새로운 플래닝 문제에 직면했을 때, 이 학습된 분포로부터 샘플링함으로써 탐색을 훨씬 효율적으로 유도할 수 있다.56

이러한 고급 주제들은 모든 샘플링 기반 플래너의 성능이 알고리즘 자체의 우수성뿐만 아니라, 해결하고자 하는 '문제의 기하학적 구조'에 얼마나 깊이 의존하는지를 명확히 보여준다. 따라서 "모든 상황에 최적인 만능 플래너"는 존재하지 않는다. 성공적인 플래닝은 문제의 특성(고차원, 좁은 통로 존재 여부 등)을 정확히 파악하고, 그에 맞는 샘플링 전략이나 다단계 플래닝과 같은 문제 분해 기법을 선택적으로 적용하는 능력에 달려있다. 이는 플래닝 문제를 단순히 '어떤 플래너를 쓸 것인가'의 문제를 넘어, '문제를 어떻게 구조화하여 플래너에게 힌트를 줄 것인가'의 문제로 전환시킨다.

### 5.3  계산 복잡도 비교 분석

알고리즘의 성능을 평가하는 또 다른 중요한 척도는 계산 복잡도이다. 샘플링 기반 플래너의 복잡도는 주로 샘플의 수 `$n$`에 대한 함수로 표현된다.

- **PRM:** PRM의 복잡도는 주로 로드맵 구축 단계에 의해 결정된다. `$n$`개의 샘플 각각에 대해 `$k$`개의 최근접 이웃을 찾아야 한다. k-d 트리와 같은 효율적인 자료구조를 사용하면 이 과정은 `$O(n \log n)$`의 시간 복잡도를 가진다. 그러나 모든 샘플 쌍을 고려하는 단순한 구현은 `$O(n^2)$`의 복잡도를 가질 수 있다.42 질의 단계의 복잡도는 로드맵 그래프의 크기와 밀도에 따라 달라지며, 일반적으로 구축 단계보다 훨씬 빠르다.
- **RRT:** RRT는 매 반복마다 새로운 샘플에 대해 트리 전체에서 최근접 이웃을 찾아야 한다. 트리가 균형 잡혀 있다고 가정하고 k-d 트리를 사용하면, `$n$`개의 노드를 가진 트리에 대한 탐색은 평균적으로 `$O(\log n)$` 시간이 걸린다. 따라서 총 시간 복잡도는 `$O(n \log n)$`이 된다.59
- **RRT\*:** RRT*는 RRT의 최근접 이웃 탐색 외에도, 새로운 노드 주변의 일정 반경 내 모든 이웃을 찾는 범위 탐색(range search)과 재연결 과정을 추가적으로 수행한다. 이론적으로, 점근적 최적성을 보장하기 위해 필요한 이웃의 수는 `$O(\log n)$`개이다. 따라서 RRT*의 시간 복잡도는 `$O(n \log n)$`으로 RRT와 점근적으로 동일하다. 하지만, 실제로는 이 과정에 포함된 상수 인자(constant factor)가 훨씬 크기 때문에 RRT보다 현저히 느리다.36
- **공간 복잡도:** RRT와 RRT*는 트리 구조만을 저장하므로, `$n$`개의 노드에 대해 `$O(n)$`의 공간 복잡도를 가진다. 반면, PRM은 로드맵 그래프를 저장해야 하므로, 엣지의 수에 따라 최악의 경우 `$O(n^2)$`의 공간이 필요할 수 있다.60

이러한 복잡도 분석은 알고리즘의 확장성(scalability)을 이해하는 데 도움을 주지만, 실제 성능은 구현 방식, 자료구조의 효율성, 그리고 무엇보다도 문제 자체의 복잡성에 의해 크게 좌우된다는 점을 유념해야 한다.

## 6.  제약 조건 및 동적 환경에서의 OMPL

지금까지의 논의는 주로 장애물 회피라는 기본적인 제약만을 고려한 정적 환경에서의 플래닝에 초점을 맞추었다. 그러나 실제 로보틱스 애플리케이션은 훨씬 더 복잡한 요구사항을 가진다. 이 장에서는 기본 OMPL 플래너들의 가정을 넘어서는 두 가지 중요한 시나리오, 즉 운동학적 제약 조건이 있는 플래닝과 동적 장애물이 존재하는 환경에서의 플래닝에 대해 OMPL의 능력과 근본적인 한계를 논한다.

### 6.1  제약 조건부 플래닝 (Constrained Planning)

- **문제 정의:** 많은 로봇 작업들은 단순한 시작-목표점 연결 이상의 복잡한 제약 조건을 수반한다. 예를 들어, 액체가 담긴 컵을 옮길 때 로봇의 말단 장치(end-effector)는 항상 수평을 유지해야 하거나, 유리창을 닦을 때 특정 평면 위에서만 움직여야 한다. 이러한 제약 조건이 있는 경우, 로봇이 취할 수 있는 유효한 상태들의 집합은 전체 설정 공간(`$\mathcal{C}$`) 내에서 더 낮은 차원을 갖는 부분다양체(submanifold)를 형성한다. 일반적인 무작위 샘플링 방법으로는 이 얇은 다양체 위에 정확히 놓이는 유효한 샘플을 생성하는 것이 확률적으로 거의 불가능하다.

- **OMPL의 접근법:** OMPL은 이러한 제약 조건부 플래닝 문제를 해결하기 위한 정교한 프레임워크를 별도로 제공한다.2 이 프레임워크의 핵심은 

  `ompl::base::ConstrainedStateSpace`이다. 이 특수한 상태 공간은 일반적인 상태 공간 위에 제약 함수 `$f(q)=0$`을 추가로 정의한다.

  - **샘플링:** 새로운 샘플을 생성할 때, 먼저 제약이 없는 공간에서 샘플링한 후, 수치 최적화 기법(예: 뉴턴-랩슨 방법)을 사용하여 이 샘플을 제약 조건을 만족하는 가장 가까운 다양체 위의 점으로 '투영(projection)'한다.
  - **보간(Interpolation):** 두 개의 유효한 제약 상태 사이를 연결할 때, 단순히 직선으로 보간하는 대신 다양체 위를 따라가는 '측지선(geodesic)' 경로를 근사적으로 계산한다. 이는 접선 공간(tangent space)에서의 이동과 재투영 과정을 반복하여 이루어진다.
  - **MoveIt2에서의 활용:** MoveIt2 사용자는 `MotionPlanRequest` 메시지 내의 `path_constraints` 필드를 통해 이러한 제약 조건을 명시할 수 있다. MoveIt2는 위치, 방향, 가시성 등 다양한 종류의 제약 조건을 지원하며, 이를 OMPL의 제약 조건부 플래닝 백엔드로 전달한다. 제공되는 튜토리얼에서는 상자(box), 평면(plane), 선(line), 그리고 방향(orientation) 제약 조건 하에서 팬더(Panda) 로봇의 경로를 계획하는 구체적인 예제를 보여준다.61

- **한계 및 고려사항:** 제약 조건부 플래닝은 매우 강력한 도구이지만, 몇 가지 내재적인 한계를 가진다. 제약 함수는 일반적으로 연속적이고 미분 가능해야 한다. 제약 다양체에 특이점(singularity)이 존재하거나 곡률이 매우 높은 경우, 투영이나 보간 과정이 수치적으로 불안정해지거나 실패할 수 있다. 이러한 실패는 플래닝 과정 전체의 실패로 이어질 수 있으며, 이를 해결하기 위해서는 제약 만족 허용 오차나 보간 스텝 크기와 같은 하이퍼파라미터에 대한 세심한 튜닝이 필요할 수 있다.62

### 6.2  동적 장애물 회피의 한계와 접근법

- **근본적 한계:** OMPL을 포함한 대부분의 샘플링 기반 플래너와 MoveIt2의 기본 플래닝 파이프라인은 환경이 정적(static)이라는 근본적인 가정 하에 설계되었다. 즉, 플래닝은 특정 시점의 환경에 대한 '스냅샷(snapshot)'을 기반으로 수행된다. 플래너는 이 스냅샷 정보를 바탕으로 시작점부터 목표점까지의 전체 경로를 한 번에 생성한다. 따라서, 플래닝이 진행되는 도중이나 계산된 경로를 로봇이 실행하는 중에 장애물이 움직이면, 생성된 경로는 더 이상 안전성을 보장할 수 없게 된다.63 OMPL 자체는 시간에 따라 변화하는 장애물을 직접적으로 모델링하는 기능을 내장하고 있지 않다.
- **대응 접근법 - 재플래닝 (Replanning):** 동적 환경에 대응하기 위한 가장 보편적이고 실용적인 접근법은 '재플래닝' 루프를 구현하는 것이다.
  - **개념:** 로봇의 센서를 통해 환경 변화(예: 새로운 장애물의 등장, 기존 장애물의 이동)가 감지되면, 현재 진행 중인 계획을 즉시 폐기하고 새로운 플래닝을 시작한다. 이때, 로봇의 현재 상태가 새로운 시작점이 되고, 업데이트된 환경 정보를 바탕으로 목표점까지의 경로를 다시 계산한다.63
  - **전략:**
    1. **전체 재플래닝:** 가장 간단한 방법으로, 환경이 변할 때마다 전체 경로를 처음부터 다시 계산한다.
    2. **부분 재플래닝/경로 복구:** 효율성을 높이기 위해, 기존 경로에서 충돌이 예상되는 부분만 식별하고 해당 부분만 수정하여 새로운 경로를 생성하는 방법도 있다. 예를 들어, DBG-RRT와 같은 알고리즘은 기존 경로를 최대한 재사용하면서 필요한 부분만 빠르게 재생성하는 전략을 제안한다.63
    3. **예측 기반 플래닝:** 움직이는 장애물의 속도와 방향을 예측하고, 이 예측된 미래 궤적을 시간 차원이 포함된 설정 공간(`$\mathcal{C} \times T$`) 상의 정적 장애물로 간주하여 플래닝을 수행하는 접근법도 있다.65
  - **OMPL에서의 도전 과제:** 재플래닝은 매우 짧은 시간 안에 새로운 경로를 생성해야 하므로, RRT-Connect와 같이 초기 해를 빠르게 찾는 플래너가 유리하다. RRT*와 같은 최적화 플래너는 재플래닝 주기가 길어져 동적인 환경 변화를 따라잡지 못할 수 있다. 중요한 점은, OMPL 자체는 이러한 재플래닝 로직이나 환경 변화 감지 메커니즘을 제공하지 않는다는 것이다. 재플래닝을 언제, 어떻게 촉발할 것인지를 결정하는 상위 레벨의 제어 루프는 전적으로 MoveIt2나 사용자 애플리케이션 수준에서 구현되어야 한다.

이러한 고급 주제들을 통해 OMPL의 역할과 위치가 더욱 명확해진다. OMPL은 주어진 정적인 문제 정의 하에서 매우 효율적으로 경로를 생성하는 강력한 '저수준 플래닝 엔진'이다. 그러나 문제 자체를 동적으로 수정하거나(예: 움직이는 장애물 반영), 복잡한 제약 조건을 해석하고 공식화하는 것은 OMPL의 책임 범위를 벗어난다. 이는 MoveIt2와 OMPL을 사용하는 개발자가 이들을 모든 것을 자동으로 해결해주는 '블랙박스'로 여겨서는 안 되며, 자신의 복잡한 문제에 맞게 상위 레벨에서 적절히 문제를 모델링하고 제어 로직을 구현해야 하는 '도구 상자'로 이해해야 함을 시사한다.

## 7. 결론 및 향후 전망

### 핵심 요약

본 보고서는 ROS2 Humble 환경에서 MoveIt2 프레임워크를 통해 사용되는 OMPL에 대한 심층적인 기술적 고찰을 제공하였다. 분석을 통해 다음과 같은 핵심 사항들을 도출하였다.

첫째, OMPL은 MoveIt2의 유연한 플러그인 아키텍처 내에서 `ompl_interface` 패키지를 통해 긴밀하게 통합된다. 이 아키텍처는 높은 모듈성을 제공하지만, 동시에 OMPL 라이브러리의 최신 기능과 MoveIt2에서 실제 사용 가능한 기능 간에 시간적, 기능적 간극이 존재할 수 있음을 인지해야 한다.

둘째, OMPL의 강력함은 개별 알고리즘뿐만 아니라, '알고리즘'과 '문제 정의'를 철저히 분리하는 모듈식 설계 철학에서 비롯된다. 이는 OMPL을 다양한 로보틱스 문제에 적용할 수 있는 범용적인 플래닝 프레임워크로 만든다.

셋째, `ompl_planning.yaml` 파일은 OMPL의 성능을 사용자의 요구에 맞게 튜닝할 수 있는 강력한 인터페이스다. `range`, `goal_bias`와 같은 파라미터들은 속도, 경로 품질, 안전성 간의 복잡한 상충 관계를 가지며, 최적의 설정은 문제의 특성에 따라 달라지므로 경험적인 튜닝이 필수적이다.

넷째, RRT-Connect와 RRT*와 같은 주요 플래너들은 각각 '빠른 만족해 탐색'과 '점근적 최적해 탐색'이라는 뚜렷한 철학적 차이를 가지며, 애플리케이션의 근본적인 요구사항에 따라 전략적으로 선택되어야 한다.

다섯째, '차원의 저주'와 '좁은 통로 문제'는 모든 샘플링 기반 플래너가 직면하는 근본적인 난제이며, OMPL의 성능은 알고리즘 자체뿐만 아니라 문제의 기하학적 구조에 깊이 의존한다. 다단계 플래닝이나 편향 샘플링과 같은 기법은 이러한 문제를 완화하는 데 효과적이다.

마지막으로, OMPL은 본질적으로 정적 환경을 가정한 저수준 플래닝 엔진이다. 제약 조건부 플래닝이나 동적 환경 대응과 같은 고급 기능은 상위 레벨의 애플리케이션(예: MoveIt2, 사용자 노드)에서 문제를 적절히 모델링하고 제어 루프를 구현함으로써 달성된다.

### 7.1 향후 전망

OMPL과 샘플링 기반 모션 플래닝 분야는 앞으로도 계속해서 빠르게 발전할 것으로 예상되며, 다음과 같은 방향이 주요 흐름이 될 것으로 전망된다.

- **머신러닝과의 통합:** 미래의 모션 플래닝은 경험으로부터 학습하는 방향으로 더욱 깊이 발전할 것이다. 과거의 성공적인 플래닝 경험 데이터베이스를 활용하여 새로운 문제에서 해가 존재할 가능성이 높은 영역을 예측하고, 그곳에 샘플링을 집중하는 학습 기반 샘플링(learning-based sampling) 기법이 일반화될 것이다.56 또한, 강화학습(Reinforcement Learning)을 통해 동적이고 불확실한 환경에서 실시간으로 반응적인 회피 정책을 학습하고, 이를 OMPL의 지역 플래너(local planner)와 결합하는 하이브리드 접근법이 더욱 정교해질 것이다.65
- **안전성과 강건성(Robustness) 강화:** 현재의 OMPL은 결정론적인 충돌 모델에 기반하지만, 실제 세계는 센서 노이즈, 로봇 모델의 불확실성, 예측 불가능한 외부 요인으로 가득 차 있다. 이러한 불확실성을 고려하여, 특정 확률 이상의 안전을 보장하는 확률론적 플래닝(probabilistic planning) 66이나, 어떠한 경우에도 안전을 보장하는 형식 검증(formal verification) 기반의 안전 보장(safety guarantees) 기법 64과의 통합이 중요한 연구 방향이 될 것이다.
- **커뮤니티와 생태계의 발전:** OMPL과 MoveIt2는 전 세계의 수많은 연구자와 개발자들이 참여하는 활발한 오픈소스 커뮤니티에 의해 유지되고 발전하고 있다.67 ROS2 생태계가 산업계와 학계 전반으로 확산됨에 따라, 더 많은 최신 알고리즘들이 OMPL에 기여되고 MoveIt2에 통합될 것이다. 또한, 벤치마킹 도구의 발전과 표준화된 문제 세트의 공유를 통해 다양한 플래너들의 성능을 객관적으로 비교하고, 특정 애플리케이션에 가장 적합한 솔루션을 선택하는 과정이 더욱 체계화될 것으로 기대된다.

결론적으로, ROS2 Humble 환경에서의 OMPL은 이미 성숙하고 강력한 도구이지만, 여전히 해결해야 할 도전 과제와 무한한 발전 가능성을 동시에 가지고 있다. 머신러닝 및 안전성 공학과 같은 인접 분야와의 융합을 통해, OMPL은 미래의 자율 로봇이 더욱 복잡하고 동적인 환경에서 지능적으로 움직일 수 있게 하는 핵심 기술로서 그 역할을 계속해서 확장해 나갈 것이다.

#### 7.1.1 Works cited

1. Motion Planning — MoveIt Documentation: Humble documentation, accessed August 6, 2025, https://moveit.picknik.ai/humble/doc/concepts/motion_planning.html
2. The Open Motion Planning Library, accessed August 6, 2025, https://ompl.kavrakilab.org/
3. Sampling-Based Motion Planning: A Comparative Review - Kavraki Lab, accessed August 6, 2025, https://www.kavrakilab.org/publications/orthey2024-review-sampling.pdf
4. Chapter 10. Motion planning in higher dimensions, accessed August 6, 2025, https://motion.cs.illinois.edu/RoboticSystems/MotionPlanningHigherDimensions.html
5. Search-based versus Sampling-based Robot Motion Planning: A Comparative Study - arXiv, accessed August 6, 2025, https://arxiv.org/html/2406.09623v2
6. OMPL - Wikipedia, accessed August 6, 2025, https://en.wikipedia.org/wiki/OMPL
7. Error while Demo launch - on ROS2 Humble and 22.04 Ubuntu · Issue #1814 · moveit/moveit2 - GitHub, accessed August 6, 2025, https://github.com/moveit/moveit2/issues/1814
8. OMPL Planner — moveit_tutorials Kinetic documentation, accessed August 6, 2025, http://docs.ros.org/en/kinetic/api/moveit_tutorials/html/doc/ompl_interface/ompl_interface_tutorial.html
9. ompl: ompl_interface::OMPLPlannerManager Class Reference, accessed August 6, 2025, http://docs.ros.org/en/noetic/api/moveit_planners_ompl/html/classompl__interface_1_1OMPLPlannerManager.html
10. moveit_planners/ompl/ompl_interface/src/planning_context_manager.cpp File Reference - moveit2, accessed August 6, 2025, https://moveit.picknik.ai/humble/api/html/planning__context__manager_8cpp.html
11. Concepts | MoveIt, accessed August 6, 2025, https://moveit.ai/documentation/concepts/
12. Release Notes - The Open Motion Planning Library, accessed August 6, 2025, https://ompl.kavrakilab.org/releaseNotes.html
13. OMPL Planner — MoveIt Documentation - PickNik Robotics, accessed August 6, 2025, https://moveit.picknik.ai/main/doc/examples/ompl_interface/ompl_interface_tutorial.html
14. API Overview - The Open Motion Planning Library - Kavraki Lab, accessed August 6, 2025, https://ompl.kavrakilab.org/api_overview.html
15. Member List - Kavraki Lab, accessed August 6, 2025, https://ompl.kavrakilab.org/classompl_1_1base_1_1Planner-members.html
16. Optimal Planning - Kavraki Lab, accessed August 6, 2025, https://ompl.kavrakilab.org/optimalPlanning.html
17. Python Bindings - The Open Motion Planning Library, accessed August 6, 2025, https://ompl.kavrakilab.org/python.html
18. Build a MoveIt! Package - ROS Industrial Training - Read the Docs, accessed August 6, 2025, https://industrial-training-master.readthedocs.io/en/humble/_source/session3/3-Build-a-MoveIt-Package.html
19. OMPL Planner — MoveIt Documentation: Humble documentation, accessed August 6, 2025, https://moveit.picknik.ai/humble/doc/examples/ompl_interface/ompl_interface_tutorial.html
20. Tuning Parameters OMPL (ompl_planning.yaml) - Google Groups, accessed August 6, 2025, https://groups.google.com/g/moveit-users/c/u5FroQ0UnsQ
21. Benchmarking planners — The Kautham Project Tutorials, accessed August 6, 2025, https://sir.upc.edu/projects/kautham_tutorials/intermediate/benchmarking/benchmarking.html
22. ompl::geometric::RRT Class Reference - Kavraki Lab, accessed August 6, 2025, https://ompl.kavrakilab.org/classompl_1_1geometric_1_1RRT.html
23. ompl::control::RRT Class Reference - Kavraki Lab, accessed August 6, 2025, https://ompl.kavrakilab.org/classompl_1_1control_1_1RRT.html
24. RRT–Path: a guided Rapidly exploring Random Tree - Computation Robotics Laboratory, accessed August 6, 2025, https://comrob.fel.cvut.cz/papers/romoco09rrt.pdf
25. Available Planners - Kavraki Lab, accessed August 6, 2025, https://ompl.kavrakilab.org/planners.html
26. Rapidly-Exploring Random Trees: A New Tool for Path Planning - Steven M. LaValle, accessed August 6, 2025, https://msl.cs.illinois.edu/~lavalle/papers/Lav98c.pdf
27. Robotic Path Planning using Rapidly exploring Random Trees, accessed August 6, 2025, https://www.araa.asn.au/acra/acra2003/papers/36.pdf
28. Fast-RRT: A RRT-Based Optimal Path Finding Method - MDPI, accessed August 6, 2025, https://www.mdpi.com/2076-3417/11/24/11777
29. ompl::geometric::RRTConnect Class Reference - Kavraki Lab, accessed August 6, 2025, https://ompl.kavrakilab.org/classompl_1_1geometric_1_1RRTConnect.html
30. RRT-Connect: An Efficient Approach to Single ... - Steven M. LaValle, accessed August 6, 2025, https://lavalle.pl/papers/KufLav00.pdf
31. HyRRT-Connect: An Efficient Bidirectional Rapidly-Exploring Random Trees Motion Planning Algorithm for Hybrid Dynamic, accessed August 6, 2025, https://hybrid.soe.ucsc.edu/sites/default/files/preprints/315.pdf
32. : Almost-Surely Asymptotically Optimal Planning with RRT-Connect - arXiv, accessed August 6, 2025, https://arxiv.org/html/2505.10542v2
33. ompl::geometric::RRTstar Class Reference - Kavraki Lab, accessed August 6, 2025, https://ompl.kavrakilab.org/classompl_1_1geometric_1_1RRTstar.html
34. Robotic Path Planning: RRT and RRT* | by Tim Chinenov | Medium, accessed August 6, 2025, https://theclassytim.medium.com/robotic-path-planning-rrt-and-rrt-212319121378
35. Near vertices and rewiring operations in RRT*; a finding near vertices - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/figure/Near-vertices-and-rewiring-operations-in-RRT-a-finding-near-vertices-b-selection-of_fig3_318896231
36. [1105.1186] Sampling-based Algorithms for Optimal Motion Planning - arXiv, accessed August 6, 2025, https://arxiv.org/abs/1105.1186
37. Sampling-based Algorithms for Optimal Motion Planning - People @EECS, accessed August 6, 2025, https://people.eecs.berkeley.edu/~pabbeel/cs287-fa19/optreadings/rrtstar.pdf
38. RRT* for Robotics: A Linear Algebra Perspective - Number Analytics, accessed August 6, 2025, https://www.numberanalytics.com/blog/rrt-star-linear-algebra-robotics-path-planning
39. ompl::geometric::PRM Class Reference - Kavraki Lab, accessed August 6, 2025, https://ompl.kavrakilab.org/classompl_1_1geometric_1_1PRM.html
40. Probabilistic Roadm IEEE Trans. on Robotics and Automation, 12(4 ..., accessed August 6, 2025, https://www.kavrakilab.org/publications/kavraki-svestka1996probabilistic-roadmaps-for.pdf
41. Probabilistic roadmaps for path planning in high-dimensional configuration spaces - SciSpace, accessed August 6, 2025, https://scispace.com/pdf/probabilistic-roadmaps-for-path-planning-in-high-dimensional-45s4l7x8cl.pdf
42. Difference between Single-Query and Multiple Query Algorithms?, accessed August 6, 2025, https://robotics.stackexchange.com/questions/18433/difference-between-single-query-and-multiple-query-algorithms
43. Probabilistic Roadmaps - KAIST, accessed August 6, 2025, https://sgvr.kaist.ac.kr/~sungeui/MPA_F15/Slides/Lecture5.pdf
44. Sensory Steering for Sampling-Based Motion Planning - University of Pennsylvania, accessed August 6, 2025, https://repository.upenn.edu/server/api/core/bitstreams/9a910ad6-4136-4ca4-bdba-62e7481667e2/content
45. ompl::geometric Namespace Reference - Kavraki Lab, accessed August 6, 2025, https://ompl.kavrakilab.org/namespaceompl_1_1geometric.html
46. ompl::geometric::LazyPRM Class Reference - Kavraki Lab, accessed August 6, 2025, https://ompl.kavrakilab.org/classompl_1_1geometric_1_1LazyPRM.html
47. ompl::geometric::FMT Class Reference - The Open Motion Planning Library, accessed August 6, 2025, https://ompl.kavrakilab.org/classompl_1_1geometric_1_1FMT.html
48. Fast Marching Trees: A Fast Marching Sampling-Based Method for ..., accessed August 6, 2025, https://stanfordasl.github.io/wp-content/papercite-data/pdf/Janson.Pavone.ISRR13.pdf
49. Motion Planning for Reconfigurable Mobile Robots Using Hierarchical Fast Marching Trees - Parasol Lab, accessed August 6, 2025, https://parasollab.web.illinois.edu/events/wafr/wafr2016/papers/WAFR_2016_paper_130.pdf
50. a Fast Marching Sampling-Based Method for Optimal Motion Planning in Many Dimensions - Lucas Janson, accessed August 6, 2025, https://lucasjanson.fas.harvard.edu/papers/Fast_Marching_Tree_A_Fast_Marching_Sampling_Based_Method-Janson_ea-2015.pdf
51. Multilevel Planning Framework - The Open Motion Planning Library, accessed August 6, 2025, https://ompl.kavrakilab.org/multiLevelPlanning.html
52. Minimising Computational Complexity of the RRT Algorithm A Practical Approach - Aalborg Universitets forskningsportal, accessed August 6, 2025, https://vbn.aau.dk/files/52820007/minimise_complexity_of_rrt.pdf
53. An Empirical Study of Optimal Motion Planning, accessed August 6, 2025, https://motion.cs.illinois.edu/papers/IROS2014-OptimalBenchmarking.pdf
54. Redalyc.Sampling-Based Motion Planning: A Survey, accessed August 6, 2025, https://www.redalyc.org/pdf/615/61513253002.pdf
55. Narrow Passage Sampling for Probabilistic Roadmap Planning - Duke Computer Science, accessed August 6, 2025, https://users.cs.duke.edu/~reif/paper/sunz/bridge/bridge.pub.pdf
56. Learning Sampling Distributions for Robot Motion Planning - Autonomous Systems Laboratory, accessed August 6, 2025, https://stanfordasl.github.io/wp-content/papercite-data/pdf/Ichter.Harrison.Pavone.ICRA18.pdf
57. Improving Path Planning Methods Using Machine Learning - ČVUT DSpace, accessed August 6, 2025, https://dspace.cvut.cz/bitstream/handle/10467/115117/F3-BP-2024-Tsoy-Artyom-main.pdf
58. Machine learning for fast motion planning - ČVUT DSpace, accessed August 6, 2025, https://dspace.cvut.cz/bitstream/handle/10467/108552/F3-BP-2023-Kriz-Jonas-Thesis.pdf
59. 7.3 Path planning algorithms: A*, RRT, and PRM - Robotics - Fiveable, accessed August 6, 2025, https://library.fiveable.me/robotics/unit-7/path-planning-algorithms-a-rrt-prm/study-guide/2Y6L25zBjDRe24Df
60. Time and space complexities of RRT and RRT* - Stack Overflow, accessed August 6, 2025, https://stackoverflow.com/questions/49482108/time-and-space-complexities-of-rrt-and-rrt
61. Using OMPL Constrained Planning — MoveIt Documentation: Humble documentation, accessed August 6, 2025, https://moveit.picknik.ai/humble/doc/how_to_guides/using_ompl_constrained_planning/ompl_constrained_planning.html
62. Constrained Planning - The Open Motion Planning Library, accessed August 6, 2025, https://ompl.kavrakilab.org/constrainedPlanning.html
63. A Dynamic Multiple-Query RRT Planning Algorithm for Manipulator Obstacle Avoidance, accessed August 6, 2025, https://www.mdpi.com/2076-3417/13/6/3394
64. OMPL Blog - The Open Motion Planning Library, accessed August 6, 2025, https://ompl.kavrakilab.org/blog.html
65. Reinforcement learning-based dynamic obstacle avoidance and integration of path planning, accessed August 6, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC8493784/
66. Roadmap Learning for Probabilistic Occupancy Maps with Topology-Informed Growing Neural Gas - Oregon State University, accessed August 6, 2025, https://research.engr.oregonstate.edu/rdml/sites/research.engr.oregonstate.edu.rdml/files/20-2050_03_ms.pdf
67. moveit/moveit2: :robot: MoveIt for ROS 2 - GitHub, accessed August 6, 2025, https://github.com/moveit/moveit2
68. The Open Motion Planning Library (OMPL) - GitHub, accessed August 6, 2025, https://github.com/ompl/ompl
69. MoveIt 2 Documentation — MoveIt Documentation: Rolling documentation, accessed August 6, 2025, https://moveit.picknik.ai/


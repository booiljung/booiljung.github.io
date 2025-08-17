# ROS2 Humble Nav2

## 서론

로봇 운영 체제(Robot Operating System, ROS)는 로봇 응용 프로그램 개발을 위한 표준 플랫폼으로 자리매김했다. 특히, 자율 이동 로봇의 핵심 기능인 내비게이션은 ROS1의 Navigation Stack을 통해 널리 구현되었으며, 이는 수많은 로봇 프로젝트의 성공에 기여했다. 그러나 ROS1의 시대가 저물고 ROS2로의 전환이 가속화되면서, 내비게이션 스택 또한 근본적인 재설계를 거쳐 Nav2로 재탄생했다. 본 고찰은 ROS2의 Long-Term Support (LTS) 릴리즈인 Humble Hawksbill 버전에 탑재된 Nav2를 중심으로, 그 설계 철학, 핵심 아키텍처, 주요 알고리즘, 성능 최적화 전략 및 고급 주제들을 심층적으로 분석하고 기술하는 것을 목표로 한다.

### 0.1 ROS1 Navigation에서 Nav2로의 진: 패러다임의 전환

ROS1 Navigation Stack의 중심에는 `move_base`라는 단일 노드가 있었다.1 이 노드는 전역 및 지역 경로 계획, 비용 지도(Costmap) 관리, 복구 행동 등 내비게이션에 필요한 모든 기능을 포함하는 모놀리식(monolithic) 구조를 가졌다. 

`move_base`는 그 자체로 매우 성공적인 패키지였으나, 몇 가지 근본적인 한계점을 내포하고 있었다. 첫째, 내부 로직이 유한 상태 머신(Finite State Machine, FSM)으로 고정되어 있어 사용자가 내비게이션 동작을 세밀하게 수정하거나 확장하기 어려웠다.1 둘째, 차동 구동(differential-drive) 및 홀로노믹(holonomic) 로봇 유형에 대한 지원에 초점이 맞춰져 있어, 애커만 스티어링(Ackermann steering)이나 다리형 로봇 등 다양한 기구학을 가진 로봇에 적용하기에는 제약이 따랐다.1

ROS2의 등장은 이러한 한계를 극복할 기회를 제공했다. ROS2는 실시간성, 신뢰성, 보안 등 산업 현장에서 요구하는 높은 수준의 요구사항을 만족시키기 위해 설계되었다.3 Nav2는 이러한 ROS2의 철학을 계승하여 단순한 ROS1 코드의 포팅(porting)을 넘어, 내비게이션 프레임워크의 패러다임을 근본적으로 전환하는 것을 목표로 했다.2

`move_base`의 모놀리식 구조는 해체되었고, 각 기능은 독립적인 모듈로 분리되어 유연성과 확장성을 극대화하는 방향으로 재설계되었다.

### 0.2 Nav2의 핵심 목표: 모듈성, 신뢰성, 확장성

Nav2 개발팀은 ROS 커뮤니티의 피드백을 바탕으로 세 가지 핵심 목표를 설정했다: 모듈성(Modularity), 신뢰성(Reliability), 그리고 확장성(Extensibility)이다.2

- **모듈성**: 사용자가 특정 알고리즘(예: 경로 플래너, 로컬 컨트롤러)을 쉽게 교체하거나 추가할 수 있도록 시스템을 설계했다. 이는 `move_base`의 고정된 구조와 대비되는 가장 큰 특징이다.2
- **신뢰성**: 시스템의 각 구성요소가 예측 가능하게 동작하고, 오류 발생 시 시스템 전체가 안전하게 대응할 수 있도록 하는 것이 중요했다. 이를 위해 ROS2의 생명주기 노드(Lifecycle Node)를 적극적으로 도입하여 각 서버의 상태를 명시적으로 관리하고, 오류 발생 시 시스템을 안전한 상태로 전환하는 메커니즘을 구현했다.2
- **확장성**: C++뿐만 아니라 Python 등 다른 프로그래밍 언어를 통해서도 시스템의 동작을 제어하고 새로운 기능을 추가할 수 있도록 설계했다.2 이는 행동 트리(Behavior Tree)와 같은 유연한 도구를 통해 내비게이션 로직을 정의함으로써 가능해졌다.

이러한 목표를 달성하기 위해 Nav2는 시스템 테스트와 지속적 통합(Continuous Integration, CI) 프로세스를 도입하여 코드의 품질과 유지보수성을 높이는 데에도 많은 노력을 기울였다.2

### 0.3 Humble Hawksbill 버전의 의의와 주요 개선 사항

ROS2 Humble Hawksbill은 2027년까지 지원되는 LTS 릴리즈로서, 안정성과 성숙도 측면에서 중요한 의미를 갖는다.6 특히 Nav2에 있어 Humble 버전은 단순한 기능 추가를 넘어, 프레임워크가 연구 단계를 지나 산업 현장에 적용될 수 있는 수준으로 발전했음을 보여주는 이정표와 같다. 많은 개발팀이 이전 버전인 Galactic에서 Humble로의 마이그레이션을 서두른 이유도 "매력적인 새로운 기능과 수정 사항" 때문이었다.6

Galactic 대비 Humble 버전의 Nav2는 여러 핵심적인 개선 사항을 포함한다. 대표적으로 Smac Planner의 데이터 구조 최적화를 통해 10-30%의 런타임 성능 향상을 이루었고 7, MPPI 컨트롤러는 45%에 달하는 획기적인 성능 개선을 달성했다.8 또한, 경로 계획과 독립적으로 임박한 충돌을 감지하고 대응하는 'Collision Monitor'라는 새로운 안전 기능이 추가되었다.6 Python 기반의 `Simple Commander` API가 개선되어 외부에서 Nav2를 제어하기가 더욱 용이해졌으며, 다양한 버그 수정과 안정성 향상이 이루어졌다.9

이러한 변화들은 Humble 버전의 Nav2가 이전 버전들과 비교하여 기능적으로 풍부하고, 성능적으로 우수하며, 실제 제품에 적용하기에 충분히 안정적이고 성숙한 프레임워크임을 시사한다. 따라서 본 고찰은 이 Humble 버전을 기준으로 Nav2의 기술적 깊이를 탐구하고자 한다.

## 1.  Nav2의 설계 철학과 아키텍처

Nav2의 아키텍처는 ROS1 Navigation Stack의 한계를 극복하고 모듈성, 신뢰성, 확장성이라는 핵심 목표를 달성하기 위해 근본적으로 재설계되었다. 그 중심에는 모놀리식 구조를 탈피하고, 각 기능을 독립적인 서버로 분리하며, 이들 간의 상호작용을 ROS2의 표준 통신 방식인 액션(Action)과 생명주기(Lifecycle) 관리로 조율하는 설계 철학이 자리 잡고 있다.

### 1.1  모놀리식(Monolithic)에서 모듈러(Modular)로: `move_base`와 Nav2의 근본적 차이

ROS1의 `move_base`는 내비게이션의 모든 로직을 단일 노드 내에 포함하는 모놀리식 구조였다.1 이는 초기 설정과 사용을 간편하게 만들었지만, FSM 기반의 경직된 로직으로 인해 사용자가 특정 동작을 수정하거나 새로운 알고리즘을 통합하는 데 큰 제약을 주었다. 예를 들어, 특정 상황에서만 다른 지역 플래너를 사용하고 싶어도 `move_base`의 코드를 직접 수정하지 않고서는 불가능했다.

Nav2는 이 문제를 해결하기 위해 `move_base`의 기능을 여러 개의 독립적인 서버(Server)로 분할했다. 각 서버는 명확하게 정의된 단일 책임을 갖는다.10

- **Planner Server**: 전역 경로(global path)를 계산하는 임무를 수행한다.
- **Controller Server**: 계산된 전역 경로를 따라 로봇을 제어하고 지역 장애물을 회피하는 임무를 수행한다.
- **Smoother Server**: 전역 경로를 더 부드럽게 다듬는 역할을 한다.
- **Behavior Server**: 복구 행동(recovery behavior)과 같은 특수 동작을 수행한다.
- **BT Navigator**: 이 모든 서버들을 조율하여 전체 내비게이션 작업을 지휘하는 두뇌 역할을 한다.

이러한 모듈러 아키텍처는 각 구성 요소의 독립적인 개발, 테스트, 교체를 가능하게 한다. 개발자는 전체 Nav2 스택을 수정하지 않고도 자신만의 플래너나 컨트롤러를 플러그인(plugin) 형태로 개발하여 기존 서버에 쉽게 통합할 수 있다.2 이는 Nav2의 확장성을 극대화하는 핵심적인 설계 결정이었다.

### 1.2  Nav2의 핵심 구성요소: 액션 기반 서버와 생명주기(Lifecycle) 관리

분리된 서버들은 서로 유기적으로 통신하며 협력해야 한다. Nav2는 이를 위해 ROS2의 두 가지 핵심 기능을 적극적으로 활용한다: 액션(Action)과 생명주기 노드(Lifecycle Node)이다.

**액션(Action) 기반 통신**: 내비게이션 작업, 예를 들어 "목표 지점까지 경로를 계산하라" 또는 "계산된 경로를 따라 이동하라"와 같은 명령은 완료하는 데 수 초에서 수 분까지 걸릴 수 있는 장시간 실행 작업이다. 이러한 비동기 작업을 효과적으로 관리하기 위해 Nav2의 서버 간 통신은 거의 모두 ROS2 액션을 통해 이루어진다.5

`BT Navigator`는 `Planner Server`에게 `ComputePathToPose` 액션을 요청하고, `Controller Server`에게 `FollowPath` 액션을 요청한다. 액션 클라이언트(BT Navigator)는 액션 서버(Planner/Controller Server)에게 목표를 전달한 후, 작업이 완료될 때까지 기다리거나(blocking), 주기적으로 상태를 확인하며 다른 작업을 수행할 수 있다. 또한, 작업 중간에 피드백(feedback)을 받아 진행 상황을 모니터링할 수도 있다.5 이 액션 기반 통신 방식은 각 서버의 독립성을 보장하면서도 복잡한 작업을 안정적으로 조율할 수 있게 해준다.

**생명주기(Lifecycle) 관리**: Nav2의 모든 핵심 서버는 ROS2의 생명주기 노드로 구현된다.2 생명주기 노드는 `Unconfigured`, `Inactive`, `Active`, `Finalized`와 같은 명시적인 상태를 가지며, 외부의 관리자(manager)에 의해 상태 전이가 제어된다.

1. **Configuring**: 노드가 생성된 후, 파라미터를 읽고 ROS 인터페이스(publisher, subscriber 등)를 설정하는 단계.
2. **Activating**: 설정이 완료된 노드가 실제 데이터 처리를 시작하도록 활성화하는 단계.
3. **Deactivating**: 노드의 작동을 일시 중지하는 단계.
4. **Cleaning Up**: 할당된 자원을 해제하는 단계.

`nav2_lifecycle_manager`는 이 모든 서버들의 생명주기를 순차적으로 관리하여, 전체 시스템이 예측 가능하고 결정론적인 순서로 시작되고 종료되도록 보장한다.11 예를 들어, 코스트맵이 완전히 활성화되기 전에는 플래너가 경로 계산을 시도하지 않도록 제어할 수 있다. 또한, `bond`라는 연결 메커니즘을 통해 서버의 비정상적인 종료(crash)를 감지하고, 이 경우 시스템 전체를 안전하게 비활성화하여 추가적인 문제를 방지한다.5 이러한 생명주기 관리는 Nav2 시스템의 전반적인 신뢰성을 크게 향상시킨다.

### 1.3  시스템 아키텍처 개요: 데이터 흐름과 노드 간 상호작용

Nav2 시스템의 전체 아키텍처는 `BT Navigator`를 중심으로 각 모듈러 서버들이 유기적으로 연결된 형태를 띤다.10 전체적인 데이터 흐름과 상호작용은 다음과 같이 요약할 수 있다.

1. **입력(Inputs)**: Nav2 시스템은 작동을 위해 몇 가지 필수적인 입력을 요구한다.1
   - **TF (Transformations)**: 로봇의 각 좌표계(`base_link`, `odom`, `map` 등) 간의 관계를 나타내는 TF 정보. REP-105 표준을 준수해야 한다.10
   - **Map Source**: 정적 레이어를 사용하는 경우, `map_server`를 통해 제공되는 환경 지도.
   - **Sensor Data**: Lidar, Depth Camera 등에서 오는 센서 데이터. 코스트맵 생성에 사용된다.
   - **Behavior Tree XML File**: `BT Navigator`가 실행할 내비게이션 로직을 정의한 XML 파일.10
2. **중앙 제어 (Central Control)**: 외부(예: RViz, 사용자 프로그램)에서 `NavigateToPose` 액션을 통해 목표 지점을 전달하면, `BT Navigator`가 이를 수신한다. `BT Navigator`는 로드된 행동 트리의 로직에 따라 각 서버에 순차적 또는 병렬적으로 작업을 지시한다.
3. **상호작용 및 데이터 흐름**:
   - **위치 추정**: `AMCL` 또는 `SLAM Toolbox` 노드는 센서 데이터와 맵을 비교하여 로봇의 현재 위치(`map` -> `odom` TF)를 추정하고 게시한다.11
   - **환경 모델링**: `Planner Server`와 `Controller Server` 내의 `Costmap2D` 인스턴스들은 `map_server`가 제공하는 정적 지도와 센서 데이터를 결합하여 전역 및 지역 코스트맵을 생성하고 지속적으로 업데이트한다.11
   - **경로 계획**: `BT Navigator`의 지시에 따라 `Planner Server`는 현재 로봇 위치에서 목표 지점까지의 전역 경로를 계산하여 `BT Navigator`에게 반환한다.
   - **경로 추종**: `BT Navigator`는 계산된 경로를 `Controller Server`에 전달한다. `Controller Server`는 이 경로를 따라가면서 지역 코스트맵을 참조하여 동적 장애물을 회피하고, 최종적으로 로봇이 따라야 할 속도 명령(`geometry_msgs/Twist`)을 계산하여 게시한다.10
   - **복구 행동**: 경로 계획이나 추종 과정에서 문제가 발생하면, 행동 트리의 로직에 따라 `Behavior Server`가 호출되어 회전, 후진 등의 복구 행동을 수행한다.
4. **출력(Output)**: Nav2 시스템의 최종 출력은 로봇의 베이스 컨트롤러가 수신하여 실행할 수 있는 속도 명령(`cmd_vel`)이다.10

이처럼 Nav2의 아키텍처는 각 기능의 책임을 명확히 분리하고, ROS2의 표준 통신 및 관리 메커니즘을 통해 이들을 유기적으로 통합함으로써, ROS1 `move_base`의 한계를 극복하고 높은 수준의 유연성, 신뢰성, 확장성을 달성했다. 이는 개발자가 단순히 주어진 내비게이션 시스템을 사용하는 것을 넘어, 다양한 부품을 조합하여 자신만의 맞춤형 내비게이션 시스템을 '구성(compose)'할 수 있게 만드는 강력한 프레임워크를 제공한다.

## 2.  행동 트리(Behavior Tree)를 통한 지능적 내비게이션

Nav2의 가장 혁신적인 변화 중 하나는 내비게이션 로직을 제어하는 방식으로 유한 상태 머신(FSM) 대신 행동 트리(Behavior Tree, BT)를 채택한 것이다. 이 결정은 Nav2에 전례 없는 수준의 유연성과 표현력을 부여했으며, 단순한 A 지점에서 B 지점으로의 이동을 넘어 복잡한 자율 작업을 설계할 수 있는 기틀을 마련했다.

### 2.1  유한 상태 머신(FSM)을 넘어: 왜 행동 트리인가?

전통적인 로봇 제어에 널리 사용되던 FSM은 상태(State)와 전이(Transition)로 시스템의 동작을 모델링한다. 이는 단순한 작업에는 효과적이지만, 내비게이션과 같이 복잡한 작업에서는 여러 한계에 부딪힌다. 예를 들어, '경로 계획 중', '경로 추종 중', '장애물로 인해 멈춤', '복구 행동 1 수행 중', '복구 행동 2 수행 중' 등 수많은 상태가 필요하게 되고, 이들 간의 전이 규칙은 기하급수적으로 복잡해져 '상태 폭발(state explosion)' 문제를 야기한다.5 이러한 FSM은 한번 설계되면 수정하거나 확장하기가 매우 어렵다.

반면, 행동 트리는 작업을 계층적인 트리 구조로 표현한다. 트리의 각 노드는 간단한 행동(Action), 조건(Condition), 또는 다른 노드들의 실행 순서를 제어하는 제어 흐름 노드(Control Flow Node)를 나타낸다. 이러한 구조는 다음과 같은 장점을 가진다.5

- **모듈성(Modularity)**: "경로 계산하기", "회전하기"와 같은 기본 행동들을 독립적인 노드로 만들어 재사용할 수 있다.
- **가독성(Readability)**: 트리 구조는 작업의 전체적인 흐름과 논리를 인간이 직관적으로 이해하기 쉽게 표현한다.
- **확장성(Scalability)**: 새로운 행동이나 복잡한 로직을 추가할 때, 기존 트리의 구조를 크게 변경하지 않고 새로운 서브트리(subtree)를 추가하는 방식으로 쉽게 확장할 수 있다.

Nav2는 이러한 BT의 장점을 활용하여, 내비게이션의 복잡한 의사결정 과정을 명시적이고 설정 가능한 방식으로 구현했다. 내비게이션 "소프트웨어"의 로직이 더 이상 C++ 코드 안에 숨겨져 있는 것이 아니라, 사용자가 직접 보고 수정할 수 있는 XML 파일로 정의되는 패러다임의 전환이 일어난 것이다.14

### 2.2  기본 행동 트리 심층 분석: `navigate_to_pose_w_replanning_and_recovery.xml`

Nav2는 `nav2_bt_navigator` 패키지에 몇 가지 기본 행동 트리를 제공하며, 그중 가장 대표적인 것이 `navigate_to_pose_w_replanning_and_recovery.xml`이다.14 이 BT는 목표 지점까지 이동하면서 지속적으로 경로를 재계획하고, 문제가 발생하면 여러 단계의 복구 행동을 수행하도록 설계되었다. 이 트리는 크게 두 개의 서브트리, 즉 `Navigation` 서브트리와 `Recovery` 서브트리로 구성된다.14

전체 트리의 최상위에는 `RecoveryNode`가 위치한다. 이 노드는 두 개의 자식 노드를 가지는데, 첫 번째는 주된 작업을 수행하는 노드(여기서는 `Navigation` 서브트리)이고 두 번째는 첫 번째가 실패했을 때 실행될 복구 노드(여기서는 `Recovery` 서브트리)이다. `number_of_retries`라는 파라미터를 통해, 주된 작업이 실패했을 때 복구 작업을 몇 번이나 시도할지를 결정한다. 기본값은 6으로, `Navigation` 서브트리가 최종적으로 실패하더라도 `Recovery` 서브트리를 통해 최대 6번까지 복구를 시도하고 다시 내비게이션을 재시도할 수 있음을 의미한다.14

### 2.3  주요 노드의 역할과 기능: `PipelineSequence`, `RecoveryNode`, `RateController` 등

기본 행동 트리의 정교한 동작은 몇 가지 핵심적인 제어 흐름 노드들에 의해 구현된다.

- **`PipelineSequence`**: 이 노드는 `Navigation` 서브트리의 핵심이다. 일반적인 `Sequence` 노드가 자식 노드들을 순차적으로 하나씩 실행하는 반면, `PipelineSequence`는 첫 번째 자식(`ComputePathToPose`)이 성공적으로 실행을 시작하면 즉시 두 번째 자식(`FollowPath`)을 실행한다. 그리고 두 번째 자식이 실행되는 동안에도 첫 번째 자식을 계속해서 실행(tick)한다.14 이 독특한 동작 방식 덕분에 로봇이 경로를 따라 이동하는 중에도 계속해서 주변 상황 변화를 반영하여 새로운 최적의 경로를 계산하는 '지속적인 재계획(continuous replanning)'이 가능해진다.
- **`RecoveryNode` (내부)**: `Navigation` 서브트리 내에서 `ComputePathToPose`와 `FollowPath` 액션은 각각 별도의 `RecoveryNode`로 감싸여 있다. 이는 '상황별 복구(Contextual Recovery)'를 구현하기 위함이다. 예를 들어, `ComputePathToPose` 액션이 실패하면(예: 경로를 찾을 수 없음), 이와 연결된 복구 로직(`ClearEntireCostmap` - 전역 코스트맵 정리)이 실행된다. 마찬가지로 `FollowPath` 액션이 실패하면(예: 경로 추종 중 장애물에 막힘), 그에 맞는 복구 로직(`ClearEntireCostmap` - 지역 코스트맵 정리)이 실행된다.14 이처럼 문제의 원인에 따라 다른 복구 전략을 적용하는 정교한 오류 처리가 가능하다.
- **`RateController`**: 이 노드는 데코레이터(Decorator) 노드의 일종으로, 자식 노드의 실행 빈도를 제어한다. `ComputePathToPose` 서브트리는 이 노드로 감싸여 있어, 기본적으로 1Hz의 빈도로만 실행된다.14 만약 이 제어가 없다면, 행동 트리의 기본 업데이트 주기(예: 100Hz)마다 경로 계획 요청이 발생하여 플래닝 서버에 엄청난 부하를 줄 것이다.  `RateController`는 이러한 불필요한 계산을 막아 시스템의 효율성을 높인다.

### 2.4  내비게이션 및 복구 서브트리의 유기적 작동 원리

`Navigation` 서브트리와 `Recovery` 서브트리는 Nav2의 2단계 방어 전략을 구성한다.

1단계: Navigation Subtree (상황별 복구)

이 서브트리는 실제 주행을 담당하는 1차 방어선이다. PipelineSequence를 통해 경로 계산과 추종을 동시에 수행하며 목표를 향해 나아간다. 이 과정에서 ComputePathToPose나 FollowPath가 일시적인 문제로 실패하면, 각각에 연결된 RecoveryNode가 즉시 상황에 맞는 복구(예: 해당 코스트맵 정리)를 시도한다.14 대부분의 사소한 문제는 이 단계에서 해결된다. 그러나 이 상황별 복구를 여러 번 시도했음에도 불구하고 문제가 해결되지 않으면, `Navigation` 서브트리는 최종적으로 `FAILURE`를 반환하고 2단계 방어선으로 제어권을 넘긴다.

2단계: Recovery Subtree (시스템 전역 복구)

Navigation 서브트리가 실패했을 때만 트리거되는 2차 방어선이다. 이 서브트리의 핵심은 RoundRobin 노드이다. RoundRobin은 자식 노드들을 순서대로 돌아가면서 한 번씩 실행한다. 기본 BT에는 네 가지 시스템 전역 복구 행동이 등록되어 있다 14:

1. **코스트맵 정리 (`ClearingActions`)**: 전역 및 지역 코스트맵을 모두 정리한다.
2. **회전 (`Spin`)**: 제자리에서 회전하여 주변을 다시 인식하거나 좁은 공간에서 탈출을 시도한다.
3. **대기 (`Wait`)**: 잠시 동안 기다려 동적 장애물이 지나가기를 기대한다.
4. **후진 (`BackUp`)**: 뒤로 약간 이동하여 교착 상태에서 벗어난다.

`RoundRobin` 노드는 이 복구 행동들을 하나씩 시도한다. 만약 한 복구 행동이 성공(`SUCCESS`)하면, 시스템은 다시 `Navigation` 서브트리로 돌아가 내비게이션을 재시도한다. 만약 재시도마저 실패하면, `RoundRobin`은 다음 순서의 복구 행동을 시도한다. 이 과정은 최상위 `RecoveryNode`에 설정된 `number_of_retries` 횟수만큼 반복된다.

이처럼 행동 트리를 통해 Nav2는 정교하고 체계적인 다단계 오류 처리 및 복구 전략을 구현한다. 더 중요한 것은, 이 모든 로직이 XML 파일에 명시적으로 정의되어 있어 사용자가 자신의 로봇과 환경에 맞게 쉽게 수정하고 확장할 수 있다는 점이다. 새로운 복구 행동을 추가하거나, 재계획 주기를 변경하거나, 심지어 완전히 새로운 내비게이션 전략을 가진 BT를 작성하는 것 모두 C++ 코드를 재컴파일할 필요 없이 가능하다.13 이는 Nav2를 단순한 내비게이션 도구가 아닌, 복잡한 자율 시스템을 구축하기 위한 강력한 프레임워크로 만드는 핵심 요소이다.

## 3.  환경 표현: Costmap 2D 심층 분석

자율 이동 로봇이 성공적으로 내비게이션을 수행하기 위해서는 주변 환경을 정확하게 인식하고 이해하는 것이 필수적이다. Nav2에서 이 역할을 담당하는 핵심 구성요소가 바로 `Costmap 2D`이다. Costmap 2D는 단순히 장애물의 위치를 나타내는 2차원 지도를 넘어, 다양한 정보를 여러 계층(layer)으로 통합하여 로봇이 더 지능적인 결정을 내릴 수 있도록 돕는 정교한 환경 모델이다.

### 3.1  전역(Global) 및 지역(Local) 코스트맵의 역할과 차이

Nav2는 일반적으로 두 종류의 코스트맵 인스턴스를 동시에 운영한다: 전역 코스트맵과 지역 코스트맵이다.1

- **전역 코스트맵 (Global Costmap)**: 이 코스트맵은 전체 작업 공간에 대한 경로를 계획하는 데 사용된다. 보통 SLAM을 통해 사전에 생성된 정적 지도(static map)를 기반으로 하며, 그 위에 센서 데이터를 통해 감지된 장애물 정보를 갱신한다. 로봇의 현재 위치부터 최종 목표 지점까지의 장거리 경로를 계산하는 `Planner Server`가 주로 이 코스트맵을 참조한다. 업데이트 주기는 비교적 낮게 설정하여 계산 부하를 줄인다.18

- **지역 코스트맵 (Local Costmap)**: 이 코스트맵은 로봇의 바로 주변 영역에 대한 실시간 정보에 초점을 맞춘다. 로봇이 이동함에 따라 함께 움직이는 '롤링 윈도우(rolling window)' 방식으로 작동하는 경우가 많다.18

  `Controller Server`는 이 지역 코스트맵을 사용하여 전역 경로를 따라가면서 실시간으로 나타나는 동적 장애물을 회피하는 제어 명령을 생성한다. 따라서 전역 코스트맵보다 훨씬 높은 빈도로 업데이트되어야 한다.18

이 두 코스트맵은 서로 다른 목적을 위해 존재하며, 각각 다른 파라미터와 레이어 구성을 가질 수 있다.

### 3.2  핵심 파라미터 설정: `plugins`, `footprint`, `global_frame` 등

모든 코스트맵 관련 설정은 `nav2_params.yaml` 파일 내에서 각 코스트맵(예: `global_costmap`, `local_costmap`)의 네임스페이스 아래에 정의된다.18 주요 파라미터는 다음과 같다.

- **`plugins`**: 사용할 레이어들의 목록과 순서를 정의하는 가장 중요한 파라미터이다. 여기에 명시된 순서대로 레이어들이 중첩되어 최종 코스트맵이 생성된다.18

- **`footprint` vs. `robot_radius`**: 로봇의 물리적 형태를 정의한다. 로봇이 완벽한 원형이 아니라면, `robot_radius` 대신 꼭짓점 좌표의 배열로 표현되는 `footprint`를 사용하는 것이 매우 중요하다.19 정확한 

  `footprint`를 사용하면, 원형으로 근사했을 때는 통과할 수 없다고 판단되었을 좁은 통로를 안전하게 통과하는 경로를 계획할 수 있다. 특히 Smac Hybrid-A*와 같은 운동학적으로 실현 가능한 플래너는 이 `footprint` 정보를 활용하여 훨씬 더 정밀하고 현실적인 경로를 생성한다.19

- **`global_frame`**: 코스트맵이 기준하는 좌표계(예: `map` 또는 `odom`)를 지정한다.18

- **`robot_base_frame`**: 로봇의 기준 좌표계(예: `base_link`)를 지정한다.18

- **`update_frequency`**: 코스트맵이 센서 데이터를 반영하여 업데이트되는 주기(Hz).18

- **`publish_frequency`**: 시각화 등을 위해 코스트맵 데이터를 토픽으로 발행하는 주기(Hz).18

- **`resolution`**: 코스트맵 그리드 한 칸의 실제 크기(m/pixel).18

### 3.3  코스트맵 레이어 분석: Static, Obstacle, Inflation 레이어의 기능과 파라미터

Costmap 2D의 진정한 강력함은 여러 레이어를 조합하여 풍부한 정보를 담은 지도를 만드는 데 있다. 각 레이어는 특정 종류의 정보를 처리하여 코스트맵에 기여한다.

- **StaticLayer (정적 레이어)**: `map_server`를 통해 제공되는 정적 지도를 코스트맵의 가장 기본 바탕으로 설정하는 역할을 한다. 이 레이어는 벽, 기둥 등 변하지 않는 환경 구조물을 나타낸다.18

- **ObstacleLayer / VoxelLayer (장애물 레이어)**: 2D 센서(Lidar)나 3D 센서(Depth Camera)로부터 받은 데이터를 처리하여 동적인 장애물을 코스트맵에 반영한다. `observation_sources` 파라미터를 통해 어떤 센서 토픽을 구독할지 지정한다. 센서 데이터가 감지된 위치에는 높은 비용(장애물)을 표시(`marking`)하고, 센서 빔이 통과한 자유 공간은 비용을 낮추는(`clearing`) 작업을 수행한다.18

  `VoxelLayer`는 3D 포인트 클라우드 데이터를 복셀 그리드로 처리하여 2D 코스트맵으로 투영하는, 더 정교한 버전의 장애물 레이어이다.18

- **InflationLayer (팽창 레이어)**: 이 레이어는 코스트맵의 지능을 한 단계 높이는 핵심적인 역할을 한다. 장애물로 표시된 셀(lethal obstacle) 주변으로 점진적으로 감소하는 비용 값을 '팽창'시킨다. 이는 로봇에게 장애물에 가까워질수록 위험 부담(비용)이 커진다는 것을 알려주는 '잠재 필드(potential field)'를 생성하는 것과 같다.18

  `inflation_radius` 파라미터는 이 팽창 영역의 반경을, `cost_scaling_factor`는 비용이 감소하는 기울기를 결정한다. 잘 튜닝된 팽창 레이어는 로봇이 단순히 장애물을 피하는 것을 넘어, 가능한 한 안전 거리를 유지하며 통로의 중앙으로 주행하도록 유도하는 효과를 낳는다.19

이러한 레이어 구조는 환경에 대한 다양한 정보를 분리하여 처리하고, 이를 체계적으로 결합할 수 있게 해준다. 정적 지도, 실시간 센서 데이터, 안전을 위한 가상 버퍼 존 등 서로 다른 성격의 정보가 하나의 코스트맵에 통합되어, 플래너와 컨트롤러가 더 현명한 결정을 내릴 수 있는 기반을 제공한다.

### 3.4  고급 기능: Costmap Filters (Keepout/Speed Zones)

Nav2 Humble에서는 코스트맵 필터라는 강력한 기능이 도입되었다. 이는 기본 레이어들이 생성한 최종 코스트맵 위에 추가적인 의미론적(semantic) 정보를 덧씌우는 역할을 한다.18

- **Keepout Filter (진입 금지 구역)**: 특정 영역을 로봇이 절대로 진입해서는 안 되는 구역으로 설정한다. 이는 물리적인 장애물은 아니지만, 운영상의 규칙(예: 위험 구역, 작업자 전용 공간)을 코스트맵에 반영하는 데 사용된다.18
- **Speed Filter (속도 제한 구역)**: 특정 영역에 진입했을 때 로봇의 최대 속도를 제한하도록 설정한다. 예를 들어, 사람이 많은 복도나 좁은 통로에서 자동으로 속도를 줄이도록 하는 데 사용할 수 있다.18

이 필터들은 흑백 이미지(filter mask)와 해당 이미지의 메타데이터를 담은 YAML 파일을 통해 정의된다. 이 기능의 도입은 코스트맵이 단순한 물리적 공간의 표현을 넘어, 인간이 정의한 규칙과 제약 조건까지 포함하는 '의미론적 월드 모델'로 발전하고 있음을 보여준다. 행동 트리는 이러한 필터 정보를 조회하여, 특정 구역에 진입했을 때 "저속 모드" BT로 전환하는 등 더욱 지능적인 행동을 수행할 수 있다. 이처럼 Costmap 2D는 Nav2 스택 전체에 공간 지능을 제공하는 핵심 허브 역할을 수행한다.

## 4.  로봇 위치 추정: AMCL과 SLAM

자율 내비게이션의 가장 기본적인 전제 조건은 로봇이 지도 상에서 자신의 현재 위치와 방향(자세, pose)을 아는 것이다. Nav2에서는 이 위치 추정(localization) 문제를 해결하기 위해 주로 적응형 몬테카를로 위치 추정(Adaptive Monte Carlo Localization, AMCL) 알고리즘을 사용한다. 또한, 내비게이션을 시작하기에 앞서 환경 지도를 생성하기 위해 SLAM(Simultaneous Localization and Mapping) 기술이 활용된다.

### 4.1  적응형 몬테카를로 위치 추정(AMCL)의 원리

AMCL은 알려진 지도(known map) 위에서 로봇의 자세를 추정하는 확률적 위치 추정 알고리즘이다.24 그 이름에서 알 수 있듯이, 이 알고리즘은 몬테카를로 방법, 즉 무작위 샘플링에 기반한 파티클 필터(particle filter)를 사용한다.

AMCL의 작동 원리는 다음과 같은 단계로 이루어진다 26:

1. **초기화 (Initialization)**: 로봇의 위치가 불확실할 때, AMCL은 지도 전역에 수천 개의 '파티클(particle)'을 무작위로 흩뿌린다. 각 파티클은 '로봇이 (x, y, θ) 위치에 있을 것이다'라는 하나의 가설을 나타낸다.
2. **예측 (Prediction)**: 로봇이 움직이면, 오도메트리(odometry) 정보를 바탕으로 모든 파티클의 위치를 동일하게 이동시킨다. 이 과정에서 로봇의 움직임에 내재된 불확실성(노이즈)이 추가된다.
3. **업데이트 (Update)**: 로봇의 센서(주로 Lidar)가 주변 환경을 스캔하면, 각 파티클의 위치에서 관측되었을 센서 값과 실제 센서 값을 비교한다. 예를 들어, 특정 파티클의 위치에서는 전방 3m에 벽이 있어야 하는데 실제 Lidar는 2.9m에서 벽을 감지했다면, 이 파티클은 '그럴듯한' 가설로 판단된다. 이 '그럴듯함'의 정도를 계산하여 각 파티클에 가중치(weight)를 부여한다. 실제 관측과 예측이 잘 맞을수록 높은 가중치를 받는다.
4. **재샘플링 (Resampling)**: 가중치가 높은 파티클(즉, 더 그럴듯한 가설)은 살아남을 확률이 높아지고, 가중치가 낮은 파티클은 제거될 확률이 높아진다. 전체 파티클 수(또는 밀도)를 유지하기 위해, 높은 가중치를 가진 파티클 주변에서 새로운 파티클들이 복제되어 생성된다. 이 과정을 반복하면 파티클들은 점차 실제 로봇의 위치 주변으로 수렴하게 된다.

'적응형(Adaptive)'이라는 이름은 KLD-샘플링(Kullback-Leibler Divergence sampling) 기법에서 유래한다.25 로봇의 위치가 불확실할 때는 더 많은 파티클을 사용하여 넓은 영역을 탐색하고, 위치가 어느 정도 수렴되어 확신이 높아지면 파티클 수를 줄여 계산 효율성을 높이는 방식으로 파티클의 수를 동적으로 조절한다.26

### 4.2  주요 AMCL 파라미터 튜닝 가이드

Nav2에 포함된 `nav2_amcl` 패키지는 ROS1 AMCL의 코드를 ROS2에 맞게 리팩토링한 것으로, 알고리즘 자체에는 큰 변화가 없다.24 따라서 기존의 튜닝 경험이 대부분 유효하다. 한편, 호환 가능한 대안으로 Beluga 라이브러리 기반의 

`beluga_amcl`도 존재한다.27 성공적인 위치 추정을 위해서는 로봇과 환경의 특성에 맞게 주요 파라미터를 정밀하게 튜닝하는 것이 매우 중요하다.

- **모션 모델 파라미터 (`alpha1` ~ `alpha5`)**: 이 파라미터들은 오도메트리 센서의 노이즈를 모델링한다. 예를 들어, `alpha1`은 회전 운동으로 인한 회전 오차를, `alpha3`는 직진 운동으로 인한 직진 오차를 나타낸다. 로봇의 휠 엔코더가 부정확하거나 바닥이 미끄러워 슬립이 잦다면 이 값들을 높여 예측 단계의 불확실성을 크게 하고, 센서 업데이트에 더 의존하도록 만들어야 한다.26
- **센서 모델 파라미터 (`laser_model_type`)**: Lidar 데이터를 어떻게 해석할지를 결정한다. `likelihood_field` 모델은 미리 계산된 가능성 필드를 사용하여 계산 효율이 높고 대부분의 경우에 잘 작동하는 기본 옵션이다. 반면 `beam` 모델은 각 레이저 빔을 개별적으로 시뮬레이션하여 더 정확한 결과를 낼 수 있지만 계산 비용이 훨씬 높다. 동적 장애물이 많은 환경이 아니라면 `likelihood_field`를 사용하는 것이 권장된다.26
- **파티클 필터 파라미터 (`min_particles`, `max_particles`)**: 필터가 유지할 파티클의 최소 및 최대 개수를 지정한다. 파티클 수가 많을수록 더 정확하고 강건한 위치 추정이 가능하지만, CPU 사용량이 증가한다. 로봇이 납치(kidnapped)된 상황에서 다시 위치를 찾아야 하는 등 어려운 문제에 대응하려면 `max_particles`를 충분히 높게 설정해야 한다.26
- **업데이트 파라미터 (`update_min_d`, `update_min_a`)**: 필터의 업데이트 단계를 트리거하기 위한 최소 이동 거리(d)와 최소 회전 각도(a)를 설정한다. 이 값들을 너무 작게 설정하면 불필요한 업데이트가 자주 발생하여 계산 자원을 낭비할 수 있고, 너무 크게 설정하면 로봇의 움직임을 제대로 반영하지 못할 수 있다. 로봇의 일반적인 주행 속도와 환경의 동적인 정도를 고려하여 조절해야 한다.26

### 4.3  SLAM Toolbox를 이용한 초기 맵 생성 및 연동

AMCL은 '알려진 지도'가 있다는 전제 하에 작동한다. 따라서 내비게이션을 시작하기 전에 로봇이 활동할 환경의 지도를 만드는 과정이 선행되어야 한다. 이 과정을 SLAM(동시적 위치 추정 및 지도 작성)이라고 하며, Nav2에서는 주로 `slam_toolbox` 패키지를 사용한다.28

SLAM 과정은 다음과 같다 28:

1. `slam_toolbox` 노드를 실행하고, 로봇을 수동으로 조작(예: 조이스틱)하여 환경의 모든 영역을 구석구석 돌아다닌다.
2. 로봇이 이동하면서 Lidar 센서로 주변을 스캔하고, `slam_toolbox`는 이 정보를 누적하여 실시간으로 지도를 그려나간다. 동시에 지도 내에서 로봇 자신의 위치를 계속해서 추정한다.
3. 맵 생성이 완료되면, `nav2_map_saver` CLI(Command-Line Interface) 도구를 사용하여 생성된 지도를 파일로 저장한다. 이 때 두 개의 파일이 생성된다: 지도의 이미지 데이터가 담긴 `.pgm` 파일과, 해상도(resolution), 원점(origin) 등의 메타데이터가 담긴 `.yaml` 파일이다.
4. 이렇게 저장된 맵은 내비게이션 실행 시 `nav2_map_server` 노드에 의해 로드된다. `map_server`는 이 맵 데이터를 토픽으로 발행하고, `AMCL`과 `Planner Server`의 `StaticLayer`가 이 토픽을 구독하여 위치 추정과 경로 계획에 사용한다.11

이처럼 SLAM을 통한 지도 생성과 AMCL을 통한 위치 추정은 Nav2 자율 내비게이션을 위한 필수적인 선행 과정이며, 이 두 기능의 안정성과 정확도가 전체 내비게이션 성능을 좌우하는 중요한 기반이 된다.

## 5.  경로 계획 알고리즘 비교 분석

Nav2의 핵심 기능은 로봇의 현재 위치에서 목표 지점까지 안전하고 효율적인 경로를 생성하는 것이다. 이 임무는 크게 두 단계로 나뉜다: 전체적인 경로의 개요를 그리는 '전역 경로 계획(Global Planning)'과, 그 경로를 따라가면서 실시간으로 장애물을 피하는 '지역 경로 계획(Local Planning)'이다. Nav2는 플러그인 기반 아키텍처를 통해 다양한 특성을 가진 여러 알고리즘을 제공하며, 사용자는 로봇의 기구학적 특성과 임무 목표에 따라 최적의 조합을 선택해야 한다.

### 5.1  전역 경로 계획 (Global Planning)

전역 플래너는 전역 코스트맵을 기반으로 출발점에서 목적지까지의 전체 경로를 계산한다. Nav2에서 제공하는 대부분의 전역 플래너는 A* 탐색 알고리즘에 기반을 두고 있다.

#### 5.1.1  탐색 기반 알고리즘의 기초: A*와 그 평가 함수 $f(n) = g(n) + h(n)$

A*(A-Star) 알고리즘은 그래프 탐색에서 최단 경로를 찾는 데 가장 널리 사용되는 알고리즘 중 하나이다. 이는 다익스트라(Dijkstra) 알고리즘의 '지금까지 온 거리'를 고려하는 특성과 탐욕적 최상 우선 탐색(Greedy Best-First Search)의 '목표까지의 추정 거리'를 고려하는 특성을 결합하여 효율성과 최적성을 모두 달성한다.29

A*는 각 노드 $n$을 평가하기 위해 다음과 같은 평가 함수 $f(n)$을 사용한다.29
$$
f(n) = g(n) + h(n)
$$

- $g(n)$: 출발 노드에서 현재 노드 $n$까지의 실제 누적 비용(거리)이다. 다익스트라 알고리즘이 사용하는 정보와 같다.
- $h(n)$: 현재 노드 $n$에서 목표 노드까지의 남은 비용을 추정한 값으로, 휴리스틱(heuristic) 함수라고 불린다. 이 휴리스틱은 실제 비용보다 항상 작거나 같아야 하는 '허용성(admissibility)' 조건을 만족해야 A*가 최적의 경로를 보장할 수 있다.29

A*는 탐색할 노드들의 우선순위 큐(open list)에서 $f(n)$ 값이 가장 작은 노드를 선택하여 탐색을 확장해 나간다. 이를 통해 무작정 목표를 향해 돌진하지도, 맹목적으로 사방을 탐색하지도 않고, 가장 유망한 방향으로 탐색을 효율적으로 이끌어간다.

#### 5.1.2  플래너 비교: NavFn, Theta*, Smac Planner (2D, Hybrid-A*, State Lattice)

Nav2는 A*를 기반으로 한 여러 전역 플래너 플러그인을 제공한다.

- **NavFn**: ROS1부터 사용된 전통적인 플래너로, 다익스트라 알고리즘(A*에서 `h(n)=0`인 경우와 유사)을 사용하여 코스트맵 상에서 최단 경로를 계산한다. 결과 경로는 일반적으로 넓고 완만한 곡선 형태를 띤다.19
- **Theta\***: A*의 변형으로, '임의 각도 계획(any-angle planning)'을 지원한다. 일반적인 A*가 그리드의 인접 셀로만 이동하는 반면, Theta*는 현재 노드와 조부모 노드 사이에 장애물이 없다면 직선으로 연결하여 더 자연스럽고 짧은 경로를 생성할 수 있다.19
- **Smac Planner 2D**: 고전적인 A* 알고리즘에 코스트맵의 비용 값을 고려하는 페널티를 추가하여, 단순히 최단 거리가 아니라 장애물로부터 안전 거리를 확보하는 경로를 선호하도록 설계된 플래너이다.19
- **Smac Planner Hybrid-A\***: 이 플래너는 Nav2의 가장 중요한 발전 중 하나이다. 일반적인 A*가 2D 그리드 공간에서 탐색하는 것과 달리, Hybrid-A*는 `(x, y, θ)`의 SE(2) 상태 공간에서 탐색을 수행한다. 이는 로봇의 방향과 최소 회전 반경과 같은 운동학적 제약(kinematic constraints)을 탐색 과정 자체에 포함시키는 것을 의미한다. Dubin 또는 Reeds-Shepp 경로와 같은 모션 프리미티브(motion primitives)를 사용하여, 자동차와 같이 제자리 회전이 불가능한 비홀로노믹(non-holonomic) 로봇을 위한 현실적으로 주행 가능한(kinematically feasible) 경로를 생성할 수 있다.34
- **Smac Planner State Lattice**: 또 다른 운동학적 제약 고려 플래너로, 미리 정의된 제어 입력 세트(minimum control sets)를 기반으로 도달 가능한 상태들의 격자(lattice)를 만들어 탐색한다. Hybrid-A*보다 더 다양한 비정형 기구학을 가진 로봇에 유연하게 적용할 수 있다.34

#### 5.1.3  로봇 기구학에 따른 플래너 선택 가이드

전역 플래너의 선택은 로봇의 기구학적 특성에 가장 크게 좌우된다. 잘못된 플래너 선택은 비효율적이거나 심지어 실행 불가능한 경로를 생성할 수 있다.

**Table 1: 전역 플래너 선택 가이드** 19

| 플러그인 이름              | 지원되는 로봇 유형                       | 주요 특징                                                    |
| -------------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| NavFn Planner              | 원형 차동, 원형 옴니디렉셔널             | 전통적인 Dijkstra 기반 플래너. 넓고 완만한 곡선 경로 생성.   |
| Theta* Planner             | 원형 차동, 원형 옴니디렉셔널             | 임의 각도 계획 지원. 직선 경로 선호.                         |
| Smac Planner 2D            | 원형 차동, 원형 옴니디렉셔널             | 비용 인지 A* 플래너. 안전 거리 확보 경향.                    |
| Smac Hybrid-A* Planner     | 비원형/원형 애커먼, 비원형/원형 다리형   | SE(2) 상태 공간 탐색. 운동학적으로 주행 가능한 경로 생성.    |
| Smac State Lattice Planner | 비원형 차동, 비원형 옴니디렉셔널, 비정형 | 제어 세트 기반 상태 격자 탐색. 다양한 기구학에 유연하게 적용. |

### 5.2  지역 경로 계획 (Local Planning / Control)

지역 플래너, 또는 컨트롤러(Controller)는 전역 플래너가 생성한 경로를 입력받아, 로봇의 현재 상태와 지역 코스트맵의 실시간 장애물 정보를 고려하여 실제 로봇에 전달할 속도 명령(`cmd_vel`)을 매 순간 계산한다.

#### 5.2.1  동적 창 접근법(DWA)의 원리와 평가 함수

동적 창 접근법(Dynamic Window Approach, DWA)은 로봇의 동역학(dynamics)을 직접적으로 고려하는 고전적이면서도 효과적인 지역 플래너이다. DWA는 위치 공간이 아닌 속도 공간, 즉 가능한 선속도($v$)와 각속도($w$)의 조합$(v, w)$ 내에서 직접 최적의 제어 명령을 탐색한다.12

DWA는 탐색 공간을 '동적 창(Dynamic Window)'으로 제한하여 실시간성을 확보한다.38 이 창은 현재 로봇의 속도와 가속도 제한을 고려하여 다음 시간 간격 내에 도달 가능한 속도들의 집합이다. 이 창 내의 각 속도$(v, w)$에 대해 가상의 원형 궤적을 시뮬레이션하고, 다음 세 가지 기준을 포함하는 평가 함수 $G(v, w)$를 최대화하는 속도 명령을 선택한다 12:

- **목표 방향성 (Heading)**: 생성된 궤적이 전역 경로 또는 목표 지점을 얼마나 잘 향하는가.
- **장애물과의 거리 (Clearance/Dist)**: 궤적 상의 가장 가까운 장애물까지의 거리. 멀수록 좋다.
- **전진 속도 (Velocity)**: 로봇의 선속도. 빠를수록 좋다.

#### 5.2.2  시간 탄성 밴드(TEB)의 원리와 다중 목표 최적화

시간 탄성 밴드(Timed Elastic Band, TEB)는 최적화(optimization) 기반의 지역 플래너이다. TEB는 전역 경로의 일부를 가져와 이산적인 로봇 자세(pose)들의 시퀀스로 표현하고, 이 시퀀스를 '탄성 밴드'처럼 간주한다.39 이 밴드는 장애물에 의해 밀려나고, 경로의 길이와 실행 시간을 최소화하려는 힘에 의해 당겨지는 것처럼 변형된다.

TEB는 이 과정을 다중 목표 최적화 문제로 공식화한다. 목표 함수는 다음과 같은 여러 항목들의 가중 합으로 구성된다 40:

- 경로 실행 시간 최소화
- 장애물과의 안전 거리 최대화
- 경로 평활도(smoothness) 최대화
- 로봇의 속도 및 가속도 등 운동학적/동역학적 제약 조건 준수

이 최적화 문제를 실시간으로 풀어 로봇의 궤적을 생성하므로, 매우 부드럽고 동역학적으로 안정적인 움직임을 만들어낼 수 있다. 하지만 계산 비용이 상대적으로 높고, 복잡한 환경에서는 지역 최적해(local minimum)에 빠질 위험이 있다.40

#### 5.2.3  컨트롤러 비교: DWB, TEB, MPPI, RPP

Nav2는 다양한 특성을 가진 컨트롤러 플러그인을 제공한다.

- **DWB (DWBLocalPlanner)**: DWA를 기반으로 ROS2 환경에 맞게 구현된 컨트롤러이다. 여러 비평가(critics) 플러그인을 조합하여 평가 함수를 유연하게 구성할 수 있는 것이 특징이다. 견고하고 널리 사용된다.12
- **TEB (TEBLocalPlanner)**: ROS1에서 포팅된 TEB 컨트롤러. 최적화 기반으로 매우 부드러운 경로를 생성하지만, 파라미터 튜닝이 민감하고 지역 최적해 문제에 취약할 수 있다.39
- **MPPI (Model Predictive Path Integral)**: 모델 예측 제어(MPC)에 기반한 최신 고급 컨트롤러이다. 수많은 무작위 궤적을 시뮬레이션하고 비용 함수를 기반으로 최적의 제어 시퀀스를 찾는다. 특히 고속 주행이나 동적인 환경에서 뛰어난 성능을 보이며, Nav2 커뮤니티에서 활발하게 개발되고 있다.8
- **RPP (Regulated Pure Pursuit)**: 순수 추종(Pure Pursuit) 알고리즘에 기반한 기하학적 컨트롤러이다. 로봇 전방의 경로 상에 '당근(carrot)'을 설정하고, 이 당근을 향해 이동하도록 제어한다. 주어진 경로를 매우 정확하게 추종하는 데 특화되어 있지만, 경로 상에 없는 동적 장애물을 회피하는 능력은 제한적이다.19

#### 5.2.4  임무 목표에 따른 컨트롤러 선택 가이드

컨트롤러 선택은 로봇이 보여주길 바라는 반응적 행동(reactive behavior)에 따라 결정된다.

**Table 2: 지역 컨트롤러 선택 가이드** 19

| 플러그인 이름            | 지원되는 로봇 유형                 | 주요 임무              | 장점                               | 단점                         |
| ------------------------ | ---------------------------------- | ---------------------- | ---------------------------------- | ---------------------------- |
| DWB Controller           | 차동, 옴니디렉셔널                 | 동적 장애물 회피       | 유연한 비평가 구성, 안정적         | MPPI보다 반응성 낮을 수 있음 |
| TEB Controller           | 차동, 옴니디렉셔널, 애커먼         | 부드러운 궤적 생성     | 매우 부드러운 경로, 최적화 기반    | 지역 최적해 위험, 튜닝 복잡  |
| MPPI Controller          | 차동, 옴니디렉셔널, 애커먼, 다리형 | 고속/동적 환경 주행    | 뛰어난 반응성, 예측 제어           | 계산 비용 높음               |
| RPP Controller           | 차동, 애커먼, 다리형               | 정확한 경로 추종       | 간단하고 예측 가능, 경로 이탈 적음 | 동적 장애물 회피 능력 부족   |
| Rotation Shim Controller | 차동, 옴니디렉셔널                 | 경로 추종 전 방향 정렬 | 주력 컨트롤러 튜닝 단순화          | 추가적인 회전 시간 소요      |

결론적으로, 전역 플래너와 지역 컨트롤러는 독립적인 요소가 아니라 서로의 특성을 보완하는 공생 관계에 있다. 예를 들어, Hybrid-A*와 같이 운동학적으로 실현 가능한 경로를 생성하는 플래너는 경로를 정확히 따라가기만 하면 되는 RPP 컨트롤러와 좋은 조합을 이룬다.19 반면, NavFn과 같이 기구학을 고려하지 않는 플래너가 생성한 경로는 DWB나 MPPI처럼 경로에서 벗어나 장애물을 회피하고 제약을 만족시킬 유연성이 있는 컨트롤러를 필요로 한다. 성공적인 Nav2 시스템 구축은 개별 알고리즘의 최적화뿐만 아니라, 이러한 플래너-컨트롤러 간의 시너지를 고려한 종합적인 설계에서 비롯된다.

## 6.  성능 최적화 및 실용적 튜닝 전략

Nav2는 매우 유연하고 설정 가능한 프레임워크이지만, 그만큼 최적의 성능을 이끌어내기 위해서는 신중한 튜닝 과정이 필수적이다. 이 과정은 단순히 파라미터 값을 조정하는 기술적인 작업을 넘어, 로봇의 물리적 특성, 작업 환경, 그리고 임무 목표 간의 복잡한 상호작용을 이해하고 균형점을 찾아가는 과정이다.

### 6.1  튜닝의 철학: 과학과 예술의 경계

Nav2 튜닝에는 정해진 공식이나 정답이 없다. 이는 과학보다는 예술에 가깝다고 할 수 있으며, 수많은 시행착오와 실험을 통해 특정 로봇과 환경에 가장 적합한 파라미터 조합을 찾아가는 과정이다.26 예를 들어, 반응성을 높이기 위해 컨트롤러 주파수를 올리면 CPU 사용량이 증가하고, 안전성을 높이기 위해 팽창 반경을 늘리면 통과할 수 있는 좁은 공간이 줄어드는 등 모든 파라미터는 트레이드오프 관계에 있다. 따라서 튜닝의 목표는 모든 지표를 최대로 만드는 것이 아니라, 주어진 제약 조건 하에서 가장 중요한 성능 지표를 우선순위에 두고 균형을 맞추는 것이다.

### 6.2  주요 성능 파라미터: 주파수, 안전, 정확도 관련 설정

Nav2의 성능에 큰 영향을 미치는 파라미터들은 크게 세 가지 범주로 나눌 수 있다.44

- **주파수 파라미터 (Frequency Parameters)**:
  - `controller_frequency`: 컨트롤러가 속도 명령을 계산하는 주기. 높을수록 로봇의 반응성이 좋아져 동적 장애물에 더 잘 대응할 수 있지만, 계산 부하가 증가한다.
  - `planner_frequency`: 전역 플래너가 새로운 경로를 계산하는 주기. 환경이 매우 동적이어서 경로를 자주 갱신해야 하는 경우가 아니라면, 너무 높게 설정할 필요는 없다. 오히려 너무 높으면 경로 진동의 원인이 될 수 있다.
  - `update_frequency` (Costmap): 코스트맵이 센서 정보를 반영하여 업데이트되는 주기. 지역 코스트맵의 경우, 로봇의 속도와 환경의 복잡도에 맞춰 충분히 높게 설정해야 한다.
- **안전 파라미터 (Safety Parameters)**:
  - `inflation_radius` (InflationLayer): 장애물 주변에 설정할 안전 버퍼의 크기. 로봇이 장애물로부터 얼마나 떨어져 주행할지를 결정하는 핵심 파라미터이다.
  - `cost_scaling_factor` (InflationLayer): 팽창된 비용이 거리에 따라 얼마나 급격하게 감소할지를 결정한다. 이 값이 높을수록 로봇은 장애물을 더욱 강하게 회피하려는 경향을 보인다.
  - `max_vel_x`, `max_vel_theta`: 로봇의 최대 선속도 및 각속도를 제한하여 안전을 확보한다.
- **정확도 파라미터 (Accuracy Parameters)**:
  - `xy_goal_tolerance`: 로봇이 목표 지점의 xy 좌표에 얼마나 가깝게 도달해야 성공으로 간주할지를 결정한다.45
  - `yaw_goal_tolerance`: 로봇이 목표 지점의 방향(yaw)과 얼마나 가깝게 정렬되어야 성공으로 간주할지를 결정한다.45 이 값들을 너무 작게 설정하면 로봇이 목표 지점 근처에서 계속 맴도는 현상이 발생할 수 있다.
  - `resolution` (Costmap): 코스트맵의 해상도. 높을수록 더 정밀한 환경 표현이 가능하지만, 메모리 사용량과 계산 부하가 증가한다.

### 6.3  로봇 형태(`footprint` vs. `radius`)가 계획에 미치는 영향

로봇의 물리적 형태를 어떻게 정의하는가는 경로 계획의 질에 직접적인 영향을 미친다. 많은 사용자들이 편의상 로봇을 원으로 근사하는 `robot_radius` 파라미터를 사용하지만, 로봇이 직사각형이나 다른 비원형 형태를 가졌다면 이는 최적이 아니다.19

정확한 꼭짓점 좌표로 정의된 `footprint`를 사용하면, 시스템은 로봇의 실제 모양을 고려하여 충돌을 검사한다. 이는 특히 좁은 공간을 통과할 때 큰 차이를 만든다. 예를 들어, 폭이 좁고 길이가 긴 로봇의 경우, `robot_radius`를 사용하면 실제로는 통과할 수 있는 좁은 문을 통과하지 못한다고 판단할 수 있다. 반면, `footprint`를 사용하면 로봇의 방향을 조절하여 문을 통과하는 정밀한 경로를 생성할 수 있다. 2021년 12월 이후, Nav2의 모든 컨트롤러 플러그인은 전체 `footprint`를 이용한 충돌 검사를 지원하므로, 비원형 로봇의 경우 반드시 `footprint`를 사용하는 것이 권장된다.19

### 6.4  제자리 회전 동작(Rotation Shim Controller)의 전략적 활용

`Rotation Shim Controller`는 Nav2의 튜닝 과정을 훨씬 수월하게 만들어주는 영리한 도구이다.19 이 컨트롤러는 주력 컨트롤러(예: DWB, MPPI)가 작동하기 전에, 먼저 로봇을 제자리에서 회전시켜 다가올 경로의 초기 방향과 대략적으로 정렬시키는 역할을 한다.

많은 지역 컨트롤러들은 경로를 부드럽게 추종하도록 튜닝하면, 출발점에서 갑자기 큰 각도로 회전하는 동작을 잘 수행하지 못하고 어색하게 휘청거리거나 나선형으로 움직이는 경향이 있다. `Rotation Shim Controller`는 이러한 "출발 시의 까다로운 회전" 문제를 전담하여 해결해준다. 덕분에 개발자는 DWB나 MPPI와 같은 주력 컨트롤러를 튜닝할 때, 복잡한 초기 회전 동작을 고려할 필요 없이 순수하게 '경로를 잘 따라가는' 성능에만 집중할 수 있다. 이는 전체적인 튜닝의 복잡도를 낮추고, 로봇의 움직임을 훨씬 더 직관적이고 안정적으로 만드는 효과를 가져온다. 단, 운동학적으로 실현 가능한 플래너(예: Hybrid-A*)를 사용하는 경우, 플래너가 이미 로봇의 초기 방향을 고려하여 주행 가능한 경로를 생성하므로 이 컨트롤러는 필수가 아닐 수 있다.19

## 7.  주요 문제 해결 및 고급 주제

Nav2를 실제 로봇에 적용하고 운영하는 과정에서는 다양한 문제와 도전 과제에 직면하게 된다. 경로 진동과 같은 고질적인 문제부터, TF 오류와 같은 일반적인 설정 문제, 그리고 새로운 버전으로의 마이그레이션까지, 이러한 문제들을 해결하는 능력은 성공적인 Nav2 시스템 구축에 필수적이다. 또한, Humble 버전에서 새롭게 도입된 고급 기능들을 이해하고 활용하는 것은 시스템의 성능과 안전성을 한 차원 높일 수 있다.

### 7.1  경로 진동(Path Oscillation) 문제와 행동 트리 기반 해결책

경로 진동은 로봇이 비슷한 비용을 가진 두 개 이상의 경로 사이에서 결정을 내리지 못하고 왕복하는 현상을 말한다.46 예를 들어, 넓은 복도에서 목표 지점으로 가는 경로를 계획할 때, 복도 왼쪽으로 가는 경로와 오른쪽으로 가는 경로의 비용이 거의 같다면, 재계획 주기가 너무 짧을 경우 로봇이 한쪽 경로로 조금 나아가다가 다시 다른 쪽 경로로 재계획하여 방향을 트는 행동을 반복할 수 있다.

이 문제를 해결하기 위해 과거에는 단순히 재계획 빈도를 낮추는 방법을 사용했지만, 이는 동적 환경에 대한 반응성을 떨어뜨리는 부작용을 낳았다. Nav2에서는 행동 트리(BT)를 활용한 더 정교한 해결책이 제안되었다.46

새로운 기본 BT에서는 `RateController`를 사용하여 일반적인 재계획 주기를 10~15초 정도로 길게 설정한다. 대신, `IsPathValid`와 같은 조건 노드를 통해 현재 경로가 여전히 유효한지(예: 새로운 장애물과 충돌하지 않는지)를 훨씬 더 높은 빈도로 확인한다. 만약 경로가 유효하다면, 로봇은 더 나은 경로가 있을 가능성이 있더라도 현재 경로를 계속해서 "신뢰하고" 따라간다. 경로가 무효화되었을 때만 즉시 재계획을 트리거한다. 이 방식은 로봇이 한번 결정한 경로에 대해 "끈기"를 갖게 하여 불필요한 진동을 막는 동시에, 실제 위험에 대해서는 신속하게 반응할 수 있는 균형 잡힌 접근법이다.

### 7.2  일반적인 오류와 해결 방안: TF 오류, 플러그인 로딩 실패 등

Nav2를 처음 설정하거나 디버깅할 때 자주 발생하는 몇 가지 일반적인 오류와 해결 방안은 다음과 같다.

- **TF 오류 (`Lookup would require extrapolation into the future`, `map` 또는 `odom` 프레임 없음)**:
  - `use_sim_time` 파라미터 불일치: 시뮬레이션을 사용할 때, Nav2 관련 모든 노드와 시뮬레이터(Gazebo)의 `use_sim_time` 파라미터가 `true`로 일관되게 설정되었는지 확인해야 한다.48
  - TF 트리 단절: `map` -> `odom` -> `base_link`로 이어지는 TF 트리가 완전한지 확인해야 한다. `map` -> `odom`은 AMCL/SLAM 노드가, `odom` -> `base_link`는 로봇의 오도메트리 소스(드라이버 또는 시뮬레이터)가 발행해야 한다.48
  - 생명주기 노드 비활성화: TF를 발행하는 노드(AMCL, 드라이버 등)가 `active` 상태로 제대로 전환되었는지 확인해야 한다. `autostart:=true` 옵션을 사용하지 않았다면 수동으로 활성화해야 한다.49
- **플러그인 로딩 실패**: `nav2_params.yaml` 파일에 정의된 플러그인 이름(`plugin: "..."`)이 실제 플러그인 클래스 이름과 정확히 일치하는지, 그리고 해당 플러그인을 포함하는 패키지가 제대로 빌드되고 설치되었는지 확인해야 한다.44
- **빌드 오류**:
  - `rosdep` 실행: `rosdep install` 명령을 사용하여 Nav2와 관련된 모든 의존성 패키지가 현재 ROS 배포판에 맞게 설치되었는지 확인해야 한다.49
  - 올바른 브랜치 사용: Nav2 소스 코드를 직접 빌드하는 경우, 사용 중인 ROS 배포판(예: Humble)에 맞는 브랜치를 사용하고 있는지 확인해야 한다. `main` 브랜치는 `rolling` 배포판을 기준으로 한다.49

### 7.3  ROS2 버전 마이그레이션: Galactic에서 Humble로의 전환 사례

ROS2 버전 간 마이그레이션, 특히 Galactic에서 Humble로의 전환은 새로운 기능을 활용하고 버그 수정을 받기 위해 필수적이다. 실제 마이그레이션 사례를 보면, 이 과정은 몇 가지 도전 과제를 동반한다.6

- **의존성 패키지 호환성**: Nav2가 의존하는 외부 패키지가 새로운 ROS 배포판(및 해당 Ubuntu 버전)에서 제대로 빌드되거나 지원되지 않는 경우가 있다. 예를 들어, Spatio-Temporal Voxel Layer (STVL) 패키지는 Humble이 사용하는 Ubuntu 22.04의 OpenVDB 라이브러리 버전과 호환성 문제가 있었다. 이 경우, 커뮤니티에서 제공하는 패치된 라이브러리를 사용하거나 직접 소스를 수정하여 해결해야 했다.6
- **점진적 마이그레이션 전략**: 전체 시스템을 한 번에 마이그레이션하는 대신, 컨테이너화(Docker) 기술을 활용하여 점진적으로 전환하는 전략이 유용할 수 있다. 예를 들어, 먼저 내비게이션 관련 기능만 Humble 기반의 Docker 컨테이너로 분리하고, 나머지 시스템은 Galactic 컨테이너에서 실행하면서 둘 사이의 통신을 통해 테스트를 진행할 수 있다. 이는 마이그레이션 과정의 복잡성을 줄이고 문제를 단계적으로 해결하는 데 도움이 된다.6

### 7.4  Humble의 새로운 기능: Collision Monitor 및 Smac Planner 개선 사항

Humble 버전은 Nav2가 안전성과 성능 측면에서 크게 성숙했음을 보여주는 여러 고급 기능을 포함한다. 이러한 기능들은 Nav2가 학술 연구 도구를 넘어 실제 산업 현장에서 사용될 수 있는 강력한 프레임워크로 발전하고 있음을 시사한다.

- **Collision Monitor**: 이 기능은 Nav2의 안전성을 한 단계 끌어올린 중요한 추가 사항이다.6 Collision Monitor는 플래너나 컨트롤러와는 별개의 독립적인 노드로 작동하며, 로봇의 현재 속도와 센서 데이터를 지속적으로 감시한다. 만약 현재 궤적을 따라갔을 때 가까운 미래에 충돌이 예상되면, Collision Monitor는 즉시 로봇을 감속시키거나 정지시킨다. 또한, 행동 트리에 신호를 보내 특정 복구 행동(예: 후진)을 트리거할 수도 있다. 이처럼 계획-실행 루프와 분리된 다중 안전 장치를 두는 것은 실제 자율 시스템에서 매우 중요한 설계 패턴이다.
- **Smac Planner 성능 개선**: Smac Planner 내부에서 사용하는 데이터 구조(예: 노드 맵)를 더 효율적인 구현체로 교체함으로써, 전반적인 계획 시간을 10-30% 단축했다. 이는 특히 복잡한 환경에서 더 빠른 재계획을 가능하게 하여 로봇의 반응성을 높인다.7
- **MPPI Controller 성능 개선**: MPPI 컨트롤러의 내부 로직을 대대적으로 최적화하여, 동일한 벤치마크에서 계산 시간을 약 45% 단축했다.8 이는 개발자가 동일한 계산 자원으로 컨트롤러의 예측 시간(prediction horizon)을 두 배로 늘리거나, 궤적 샘플 수를 두 배로 늘려 더 나은 제어 성능을 달성할 수 있음을 의미한다. 이는 로봇의 최대 주행 속도를 높이거나 더 복잡하고 동적인 환경에 대응하는 데 직접적으로 기여한다.

이러한 개선 사항들은 Nav2가 커뮤니티의 요구와 실제 현장의 문제들을 적극적으로 해결하며 진화하고 있음을 보여준다. 특히 안전과 성능에 대한 근본적인 개선은 Humble 버전으로의 업그레이드를 고려해야 하는 강력한 이유가 된다.

## 8. 결론

ROS2 Humble Nav2는 ROS1 Navigation Stack의 성공적인 유산을 계승하면서도, 그 한계를 극복하기 위해 근본적인 아키텍처 재설계를 감행한 차세대 로봇 내비게이션 프레임워크이다. 본 고찰을 통해 분석한 바와 같이, Nav2는 모듈성, 신뢰성, 확장성이라는 명확한 설계 목표 아래, 단순한 A-to-B 이동 기능을 넘어 복잡하고 지능적인 자율 시스템을 구축하기 위한 강력한 기반을 제공한다.

### ROS2 Humble Nav2의 핵심 강점 요약

Nav2의 핵심 강점은 다음과 같이 요약될 수 있다.

- **행동 트리를 통한 유연하고 지능적인 작업 조율 능력**: 경직된 유한 상태 머신을 탈피하고 행동 트리를 채택함으로써, Nav2는 내비게이션 로직을 사용자가 직관적으로 이해하고 쉽게 수정할 수 있는 XML 파일로 분리했다. 이는 상황별 복구, 다단계 실패 처리 등 정교한 동작을 구현할 수 있게 할 뿐만 아니라, 내비게이션을 더 큰 규모의 자율 작업의 일부로 손쉽게 통합할 수 있는 길을 열었다.
- **플러그인 기반의 모듈식 아키텍처가 제공하는 최고의 확장성**: `move_base`의 모놀리식 구조를 해체하고, 플래너, 컨트롤러, 스무더, 비헤이비어 등 각 기능을 독립적인 서버로 분리한 것은 Nav2의 가장 중요한 아키텍처 결정이다. 이는 각 기능의 독립적인 개발과 교체를 가능하게 하여, 전 세계 개발자들이 자신만의 알고리즘을 플러그인 형태로 Nav2 생태계에 기여하고 공유할 수 있게 만들었다.
- **운동학적으로 실현 가능한 플래너와 고성능 컨트롤러를 통한 다양한 로봇 지원**: Smac Hybrid-A*와 State Lattice 플래너는 자동차나 다리형 로봇과 같이 운동학적 제약이 심한 로봇도 현실적으로 주행 가능한 경로를 계획할 수 있게 해주었다. 또한, MPPI와 같은 고성능 컨트롤러는 동적인 환경에서 로봇이 더 빠르고 안전하게 주행할 수 있도록 지원한다. 이는 Nav2가 특정 로봇 유형에 국한되지 않고 광범위한 모바일 로봇 플랫폼에 적용될 수 있음을 의미한다.
- **Humble 버전에서 강화된 성능, 안정성, 그리고 안전 기능**: Humble LTS 릴리즈는 Nav2가 성숙기에 접어들었음을 보여주는 이정표이다. Smac Planner와 MPPI 컨트롤러의 획기적인 성능 향상, Collision Monitor와 같은 새로운 안전 기능의 도입, 그리고 생명주기 관리와 행동 트리를 통한 시스템 안정성 강화는 Nav2가 연구실을 넘어 산업 현장의 요구사항을 만족시킬 준비가 되었음을 증명한다.

### 8.1 미래 발전 방향과 커뮤니티의 역할 조망

Nav2는 이미 강력한 프레임워크이지만, 그 발전은 현재진행형이다. 앞으로 Nav2는 더욱 지능적이고 강건한 방향으로 진화할 것으로 예상된다. 예를 들어, 다양한 내비게이션 전략의 성능을 객관적으로 비교하고 튜닝 과정을 자동화하기 위한 정량적 성능 평가 및 벤치마킹 프레임워크 구축이 중요한 과제로 논의되고 있다.50 또한, 내비게이션 그래프를 활용한 경로 계획(Route Server), 동적 도킹 기능 등 새로운 기능들이 지속적으로 추가되고 있다.

이러한 발전의 중심에는 활발한 오픈소스 커뮤니티가 있다. Nav2는 특정 기업의 전유물이 아니라, 전 세계 수많은 연구자, 개발자, 기업들의 기여로 함께 만들어가는 프로젝트이다.51 커뮤니티의 지속적인 참여와 피드백, 그리고 새로운 플러그인과 아이디어의 공유는 Nav2가 미래의 로봇 기술 변화에 발맞춰 계속해서 진화해 나갈 수 있는 가장 큰 원동력이 될 것이다.

결론적으로, ROS2 Humble Nav2는 단순한 내비게이션 스택을 넘어, 모바일 로봇 자율성을 위한 표준 플랫폼이자 강력한 개발 프레임워크로 확고히 자리매김하고 있다. 그 정교한 아키텍처와 풍부한 기능, 그리고 활발한 커뮤니티를 바탕으로, Nav2는 앞으로 다가올 자율 이동 로봇의 시대를 이끌어 나갈 핵심 기술이 될 것으로 기대된다.

#### **참고 자료**

1. Navigation (ROS 1) Vs Navigation 2 (ROS 2) | by Sharad ... - Medium, 8월 5, 2025에 액세스, https://medium.com/@thehummingbird/navigation-ros-1-vs-navigation-2-ros-2-12398b64cd
2. Matt Hansen, Sr. SW Architect Open Source Robotics Intel ROS2 Robot Dev Kit Featuring Navigation2 Overview - ROS-Industrial, 8월 5, 2025에 액세스, https://rosindustrial.squarespace.com/s/01_Matt_Hansen_ROS-I-Conf-Europe-2019-Dev-Kit-Navigation2-Overview-bd9w.pdf
3. ROS 2 Documentation: Humble documentation, 8월 5, 2025에 액세스, https://docs.ros.org/en/humble/index.html
4. Features Status - ROS 2 Documentation: Humble documentation, 8월 5, 2025에 액세스, https://docs.ros.org/en/humble/The-ROS2-Project/Features.html
5. Navigation Concepts - Nav2 1.0.0 documentation, 8월 5, 2025에 액세스, https://docs.nav2.org/concepts/index.html
6. ROS 2 migration stories: The struggles of moving to Humble - Karelics, 8월 5, 2025에 액세스, https://karelics.fi/blog/2023/08/16/ros-2-migration-stories-the-struggles-of-moving-to-humble/
7. [Nav2] Major run-time improvement on Smac Planners (10-30%)! - ROS Discourse, 8월 5, 2025에 액세스, https://discourse.openrobotics.org/t/nav2-major-run-time-improvement-on-smac-planners-10-30/27396
8. Nav2 MPPI - 45% Performance Boost - Beta Testing Requested - ROS Discourse, 8월 5, 2025에 액세스, https://discourse.openrobotics.org/t/nav2-mppi-45-performance-boost-beta-testing-requested/36652
9. Costmap 2D - Nav2 1.0.0 documentation, 8월 5, 2025에 액세스, https://ros.ncnynl.com/en/nav2/configuration/packages/configuring-costmaps.html
10. Nav2 - Nav2 1.0.0 documentation, 8월 5, 2025에 액세스, https://docs.nav2.org/
11. 9.About ROS2 Navigation - Yahboom, 8월 5, 2025에 액세스, [https://www.yahboom.net/public/upload/upload-html/1699598914/9.About%20ROS2%20Navigation.html](https://www.yahboom.net/public/upload/upload-html/1699598914/9.About ROS2 Navigation.html)
12. 8. Navigation and obstacle avoidance - Yahboom, 8월 5, 2025에 액세스, http://www.yahboom.net/build/id/7468/cid/310
13. Behavior tree with NAV2 - Floris Jousselin, 8월 5, 2025에 액세스, https://robotcopper.github.io/ROS/behavior_tree.html
14. Detailed Behavior Tree Walkthrough - Nav2 1.0.0 documentation, 8월 5, 2025에 액세스, https://docs.nav2.org/behavior_trees/overview/detailed_behavior_tree_walkthrough.html
15. Navigation and Behaviour trees for task execution | by Rushikesh Halle | Thoughtworks: e4r™ Tech Blogs | Medium, 8월 5, 2025에 액세스, https://medium.com/e4r/navigation-and-behaviour-trees-for-task-execution-798ba999f8f6
16. nav2_behavior_tree: Humble 1.1.18 documentation, 8월 5, 2025에 액세스, https://docs.ros.org/en/ros2_packages/humble/api/nav2_behavior_tree/
17. Getting Started with Custom Behavior Tree Plugins in Nav2: Seeking Guidance and Resources : r/ROS - Reddit, 8월 5, 2025에 액세스, https://www.reddit.com/r/ROS/comments/1ii9qso/getting_started_with_custom_behavior_tree_plugins/
18. Costmap 2D - Nav2 1.0.0 documentation, 8월 5, 2025에 액세스, https://docs.nav2.org/configuration/packages/configuring-costmaps.html
19. Tuning Guide - Nav2 1.0.0 documentation, 8월 5, 2025에 액세스, https://docs.nav2.org/tuning/index.html
20. nav2_costmap_2d / humble / Enrico Schütz / navigation2 / GitLab - at https://git.tu, 8월 5, 2025에 액세스, https://git.tu-berlin.de/ecschuetz/navigation2/-/tree/humble/nav2_costmap_2d
21. Obstacle Layer Parameters - Nav2 1.0.0 documentation, 8월 5, 2025에 액세스, https://docs.nav2.org/configuration/packages/costmap-plugins/obstacle.html
22. costmap_2d - ROS Wiki, 8월 5, 2025에 액세스, http://wiki.ros.org/costmap_2d
23. Navigating with Speed Limits - Nav2 1.0.0 documentation, 8월 5, 2025에 액세스, https://docs.nav2.org/tutorials/docs/navigation2_with_speed_filter.html
24. nav2_amcl: Humble 1.1.18 documentation, 8월 5, 2025에 액세스, https://docs.ros.org/en/ros2_packages/humble/api/nav2_amcl/
25. amcl - ROS Wiki, 8월 5, 2025에 액세스, http://wiki.ros.org/amcl
26. ROS 2 Navigation Tuning Guide – Nav2 - Automatic Addison, 8월 5, 2025에 액세스, https://automaticaddison.com/ros-2-navigation-tuning-guide-nav2/
27. beluga_amcl: Humble 2.0.2 documentation, 8월 5, 2025에 액세스, https://docs.ros.org/en/ros2_packages/humble/api/beluga_amcl/
28. ROS2 Nav2 Tutorial - The Robotics Back-End, 8월 5, 2025에 액세스, https://roboticsbackend.com/ros2-nav2-tutorial/
29. AI | Search Algorithms | A* Search | Codecademy, 8월 5, 2025에 액세스, https://www.codecademy.com/resources/docs/ai/search-algorithms/a-star-search
30. The A* Algorithm: A Complete Guide - DataCamp, 8월 5, 2025에 액세스, https://www.datacamp.com/tutorial/a-star-algorithm
31. Introduction to A* - Stanford CS Theory, 8월 5, 2025에 액세스, http://theory.stanford.edu/~amitp/GameProgramming/AStarComparison.html
32. A* search algorithm - Wikipedia, 8월 5, 2025에 액세스, https://en.wikipedia.org/wiki/A*_search_algorithm
33. ROS and ROS2 Navigation Stacks: A performance review | by Gaurav Gupta | Black Coffee Robotics - Medium, 8월 5, 2025에 액세스, https://medium.com/black-coffee-robotics/ros-and-ros2-navigation-stacks-a-performance-review-facb4d6c15a1
34. rapyuta-robotics/smac_planner - GitHub, 8월 5, 2025에 액세스, https://github.com/rapyuta-robotics/smac_planner
35. Smac Hybrid-A* Planner - Nav2 1.0.0 documentation, 8월 5, 2025에 액세스, https://docs.nav2.org/configuration/packages/smac/configuring-smac-hybrid.html
36. Open-Source, Cost-Aware Kinematically Feasible Planning for Mobile and Surface Robotics, 8월 5, 2025에 액세스, https://arxiv.org/html/2401.13078v1
37. Local planning methods for autonomous navigation on sidewalks: a comparative survey - -ORCA, 8월 5, 2025에 액세스, [https://orca.cardiff.ac.uk/id/eprint/167063/7/Local%20planning%20methods%20for%20autonomous%20navigation%20on%20sidewalks%20-%20a%20comparative%20survey.pdf](https://orca.cardiff.ac.uk/id/eprint/167063/7/Local planning methods for autonomous navigation on sidewalks - a comparative survey.pdf)
38. The Dynamic Window Approach to Collision Avoidance 1 Introduction, 8월 5, 2025에 액세스, https://www.ri.cmu.edu/pub_files/pub1/fox_dieter_1997_1/fox_dieter_1997_1.pdf
39. Improvement of the TEB Algorithm for Local Path Planning of Car-like Mobile Robots Based on Fuzzy Logic Control - MDPI, 8월 5, 2025에 액세스, https://www.mdpi.com/2076-0825/14/1/12
40. (PDF) An Improved Timed Elastic Band (TEB) Algorithm of ..., 8월 5, 2025에 액세스, https://www.researchgate.net/publication/356997131_An_Improved_Timed_Elastic_Band_TEB_Algorithm_of_Autonomous_Ground_Vehicle_AGV_in_Complex_Environment
41. teb_local_planner - ROS Wiki, 8월 5, 2025에 액세스, http://wiki.ros.org/teb_local_planner
42. rst-tu-dortmund/teb_local_planner: An optimal trajectory planner considering distinctive topologies for mobile robots based on Timed-Elastic-Bands (ROS Package) - GitHub, 8월 5, 2025에 액세스, https://github.com/rst-tu-dortmund/teb_local_planner
43. Working list of algorithms to include / Issue #1710 / ros-navigation/navigation2 - GitHub, 8월 5, 2025에 액세스, https://github.com/ros-planning/navigation2/issues/1710
44. ROS 2 Navigation2 Configuration: Complete Guide to Optimizing Your Rob - Think Robotics, 8월 5, 2025에 액세스, https://thinkrobotics.com/blogs/learn/ros-2-navigation2-configuration-complete-guide-to-optimizing-your-robot-navigation-stack
45. Navigation in ROS2 - Robot & Chisel, 8월 5, 2025에 액세스, https://www.robotandchisel.com/2020/09/01/navigation2/
46. [Nav2] Feedback Required: Potential Change to Default Behavior - ROS Discourse, 8월 5, 2025에 액세스, https://discourse.openrobotics.org/t/nav2-feedback-required-potential-change-to-default-behavior/24687
47. [Nav2] Feedback Required: Potential Change to Default Behavior ..., 8월 5, 2025에 액세스, https://discourse.ros.org/t/nav2-feedback-required-potential-change-to-default-behavior/24687
48. Discussion - Nav2 Introduction (Making a Mobile Robot Pt 16), 8월 5, 2025에 액세스, https://discourse.articulatedrobotics.xyz/t/discussion-nav2-introduction-making-a-mobile-robot-pt-16/199
49. Build Troubleshooting Guide - Nav2 1.0.0 documentation, 8월 5, 2025에 액세스, https://docs.nav2.org/development_guides/build_docs/build_troubleshooting_guide.html
50. [Nav2][Discussion] Metrics / framework for quantitative evaluation of navigation performance, 8월 5, 2025에 액세스, https://discourse.ros.org/t/nav2-discussion-metrics-framework-for-quantitative-evaluation-of-navigation-performance/28082
51. [Nav2] A Time for Change - ROS Discourse - Open Robotics, 8월 5, 2025에 액세스, https://discourse.openrobotics.org/t/nav2-a-time-for-change/30525


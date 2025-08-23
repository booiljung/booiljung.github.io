# ROS2 Humble의 PlanSys2



로봇 자율성의 핵심은 주어진 목표를 달성하기 위해 행동 순서를 스스로 결정하는 능력, 즉 계획(Planning)에 있다. ROS(Robot Operating System) 생태계에서 이러한 기호 계획(Symbolic Planning)의 문을 연 것은 ROS1의 ROSPlan이었다.1 ROSPlan은 PDDL(Planning Domain Definition Language)을 사용하여 로봇의 작업 영역을 모델링하고 계획을 생성하는 기능을 제공함으로써, 많은 연구자와 개발자에게 강력한 도구를 제공했다. 하지만 ROSPlan은 ROS1이라는 기반 위에서 동작했기에, ROS1이 가진 근본적인 한계를 그대로 물려받을 수밖에 없었다. TCPROS/UDPROS 기반 통신의 실시간성 보장 부재, 보안 기능의 취약성, 그리고 노드 시작 순서에 따른 비결정론적 동작 가능성 등은 산업 현장이나 장시간 안정적인 운영이 필수적인 환경에서 ROSPlan을 적용하는 데 걸림돌이 되었다.2

이러한 배경 속에서 등장한 PlanSys2는 단순한 ROSPlan의 ROS2 버전으로의 이식(Porting)이 아니다.4 PlanSys2는 ROS2가 제공하는 새로운 핵심 개념들을 설계 철학의 근간으로 삼아 근본적으로 재설계된 프레임워크다. 개발팀은 ROSPlan을 수년간 사용하며 얻은 경험을 바탕으로, 기존의 한계를 극복하고 로봇 개발에 필수적인 기능들을 새롭게 디자인하여 통합했다.5 PlanSys2의 설계는 ROS2의 핵심 기능인 DDS(Data Distribution Service) 기반의 통신과 Lifecycle Node(Managed Node)를 적극적으로 활용한다. DDS는 실시간 데이터 전송을 위한 QoS(Quality of Service) 설정, 보안 통신, 그리고 효율적인 네트워크 발견 메커니즘을 제공하며, Lifecycle Node는 시스템의 각 컴포넌트(노드)가 예측 가능한 상태(state)를 가지도록 강제한다.3 이처럼 PlanSys2는 ROS1의 한계를 답습하는 대신, ROS2의 장점을 극대화하여 신뢰성, 효율성, 그리고 사용 편의성을 모두 향상시키는 것을 목표로 탄생했다. 이는 PlanSys2가 단순한 '후속작'이 아닌, ROS2 네이티브 환경을 위한 '차세대' 계획 프레임워크임을 명확히 보여준다.


PlanSys2의 핵심에는 PDDL이 자리 잡고 있다. PDDL은 AI 계획 커뮤니티에서 수십 년간 발전해 온 표준 언어로, 로봇이 수행할 작업과 그 작업이 이루어지는 환경을 선언적으로(declaratively) 기술하는 데 사용된다.3 PDDL의 가장 큰 특징은 "어떻게(How)"가 아닌 "무엇을(What)"에 집중한다는 점이다. 개발자는 로봇이 할 수 있는 행동(actions)과 세상의 속성(predicates)을 정의하고, 현재 상태(initial state)와 원하는 목표 상태(goal state)를 기술하기만 하면 된다. 그러면 PDDL 플래너가 이 정보를 바탕으로 목표를 달성하기 위한 최적의 행동 순서, 즉 계획(plan)을 찾아낸다.7

PDDL은 크게 두 개의 파일로 구성된다.7

1. **도메인 파일 (Domain File):** 해당 영역의 일반적인 규칙을 정의한다. 여기에는 타입(types, 예: robot, room), 술어(predicates, 예: `(at?robot?room)`), 그리고 행동(actions, 예: `move`, `pick`)의 정의가 포함된다. 행동은 그것이 실행되기 위한 전제조건(preconditions)과 실행 후 세상이 어떻게 변하는지를 나타내는 효과(effects)로 구성된다.
2. **문제 파일 (Problem File):** 특정 문제를 기술한다. 여기에는 세상에 존재하는 구체적인 객체(objects, 예: r2d2, kitchen), 시스템의 초기 상태(initial state), 그리고 달성하고자 하는 최종 목표(goal)가 명시된다.

PlanSys2는 이 표준화된 PDDL을 채택함으로써, 로봇 공학의 세계와 AI 계획 연구의 세계를 잇는 강력한 다리 역할을 수행한다. POPF, TFD와 같은 검증된 외부 플래너들을 쉽게 통합하여 로봇에 적용할 수 있게 해주며, 이는 로봇이 더욱 복잡하고 지능적인 작업을 수행할 수 있는 기반을 마련한다.1


자율 로봇의 행동은 대부분 Sense-Plan-Act라는 순환적인 패러다임으로 설명할 수 있다.12

- **Sense (인식):** 로봇이 센서(카메라, LiDAR 등)를 통해 주변 환경과 자신의 상태에 대한 정보를 수집하고 이해하는 단계.
- **Plan (계획):** 수집된 정보를 바탕으로 현재 상태에서 목표 상태에 도달하기 위한 일련의 행동 순서를 결정하는 단계.
- **Act (실행):** 결정된 계획에 따라 액추에이터(모터, 그리퍼 등)를 움직여 실제 세계에서 행동을 수행하는 단계.

PlanSys2는 이 패러다임에서 'Plan'과 'Act' 단계를 체계적으로 구현하고 관리하는 역할을 담당한다. 시스템의 각 컴포넌트는 이 패러다임의 특정 부분에 명확하게 대응된다. 'Sense' 단계에서 수집된 정보는 **Problem Expert**에 의해 세계의 현재 상태로 반영된다. 이 정보를 바탕으로 **Planner**는 'Plan' 단계, 즉 행동 계획을 생성한다. 마지막으로, **Executor**는 생성된 계획을 받아 실제 로봇의 행동을 유발하는 'Act' 단계를 책임진다. 이처럼 PlanSys2는 추상적인 Sense-Plan-Act 패러다임을 구체적인 ROS2 노드 아키텍처로 구현하여, 개발자가 복잡한 자율 시스템을 보다 구조적이고 체계적으로 개발할 수 있도록 지원한다.



PlanSys2의 설계는 '관심사의 분리(Separation of Concerns)' 원칙을 철저히 따른다. 시스템의 핵심 기능은 네 개의 독립적인 ROS2 노드로 명확하게 분리되어 있으며, 각 노드는 고유한 책임을 가진다. 이러한 모듈식 설계는 시스템의 유연성, 확장성, 그리고 유지보수성을 극대화한다.4

- **Domain Expert:** 이 노드는 PDDL 도메인 파일에 정의된 정적(static) 지식을 관리한다.4 도메인이란 로봇이 활동하는 세계의 물리 법칙과 같다. 예를 들어, '로봇은 한 번에 한 장소에만 있을 수 있다' 또는 '물건을 집으려면 손이 비어 있어야 한다'와 같은 규칙들이 여기에 해당한다. Domain Expert는 하나 이상의 PDDL 도메인 파일을 읽어들여 내부적으로 병합할 수 있는 기능을 제공한다.3 이는 여러 패키지가 각자 자신의 기능에 해당하는 PDDL 조각과 액션 구현을 제공하여 하나의 통합된 애플리케이션을 구성하는 모듈식 개발을 가능하게 한다.

- **Problem Expert:** 이 노드는 현재 세계의 동적(dynamic)이고 휘발성(volatile)인 상태 정보를 관리한다.4 '로봇 r2d2가 현재 주방에 있다`(at r2d2 kitchen)`' 또는 '배터리가 70% 남았다 `(= (battery_level r2d2) 70)`'와 같은 구체적인 사실(facts)들이 여기에 해당한다. 이 정보는 센서 데이터나 다른 노드의 행동 결과에 따라 계속해서 변한다. Problem Expert는 자신의 상태가 변경될 때마다 `std_msgs::msg::Empty` 타입의 토픽을 발행하여, 시스템의 다른 부분에 "세계가 변했다"는 사실을 알리는 역할을 한다.4 이는 재계획(replanning)을 유발하는 중요한 트리거가 된다.

- **Planner:** 이 노드는 실제 계획을 생성하는 두뇌 역할을 한다.4 Planner는 계획 요청을 받으면, Domain Expert로부터 세계의 규칙(도메인)을, Problem Expert로부터 현재 상황과 목표(문제)를 PDDL 형식으로 전달받는다. 그리고 이 정보를 바탕으로 지정된 외부 PDDL 솔버를 호출하여 목표를 달성하기 위한 행동 계획(plan)을 생성한다.3 PlanSys2의 Planner는 플러그인 기반으로 설계되어, 사용자는 POPF, TFD 외에도 자신의 문제에 더 적합한 다른 PDDL 솔버를 쉽게 통합할 수 있다.18

- **Executor:** 이 노드는 Planner가 생성한 추상적인 계획을 현실 세계의 행동으로 옮기는 실행자다.4 Executor는 계획을 받아, 각 행동을 수행하도록 구현된 ROS2 노드(Action Performer)를 순서에 맞게 호출한다. 단순히 순서대로 호출하는 것을 넘어, 행동 실행 중에 전제조건이 계속 만족되는지 감시하고, 만약 조건이 깨지면 즉시 계획 실행을 중단하여 시스템을 안전한 상태로 유지한다.19

이 네 가지 노드의 명확한 역할 분리는 복잡한 자율 시스템을 개발하고 디버깅하는 과정을 훨씬 단순하게 만든다. 예를 들어, 계획이 잘못 생성된다면 Planner 노드와 그 플러그인을 집중적으로 살펴보면 되고, 로봇의 행동이 이상하다면 Executor나 해당 Action Performer 노드를 조사하면 된다.


PlanSys2의 강건성(robustness)과 예측 가능성(predictability)은 모든 핵심 컴포넌트가 ROS2의 `rclcpp_lifecycle::LifecycleNode`로 구현되었다는 사실에 깊이 뿌리내리고 있다.3 ROS1에서는 여러 노드를 동시에 실행할 때 어떤 노드가 먼저 활성화될지 보장할 수 없어, 초기화 순서에 따른 경쟁 상태(race condition)가 빈번하게 발생했다. 이는 시스템의 불안정성을 야기하는 주요 원인이었다.

Lifecycle Node는 이러한 문제를 해결하기 위해 노드의 생명주기를 명시적인 상태 머신(State Machine)으로 관리한다.3 각 노드는 `Unconfigured`, `Inactive`, `Active`, `Finalized`와 같은 상태를 가지며, 외부의 트리거에 의해서만 상태 전이(transition)가 일어난다. PlanSys2 시스템을 시작할 때, 시스템 관리자(예: 런치 파일의 `LifecycleManager`)는 다음과 같은 결정론적 순서로 노드들을 활성화할 수 있다.

1. 모든 노드(Domain Expert, Problem Expert, Planner, Executor)를 생성한다. 이 때 노드들은 `Unconfigured` 상태다.
2. 각 노드에 대해 `configure` 전이를 트리거한다. 노드들은 파라미터를 읽고 필요한 메모리를 할당하는 등의 초기 설정을 수행하고 `Inactive` 상태로 전환된다.
3. 모든 노드가 성공적으로 `Inactive` 상태가 되면, 각 노드에 대해 `activate` 전이를 트리거한다. 노드들은 ROS2 통신(퍼블리셔, 구독자, 서비스 등)을 시작하고 실제 작업을 수행할 준비가 된 `Active` 상태로 전환된다.

이러한 단계적이고 통제된 활성화 과정은 모든 컴포넌트가 완전히 준비된 상태에서 시스템이 동작을 시작하도록 보장한다. 또한, 에러가 발생했을 때나 시스템을 종료할 때도 `on_cleanup`, `on_shutdown`과 같은 콜백을 통해 리소스를 안전하게 해제할 수 있다. 이처럼 Lifecycle Node의 채택은 PlanSys2가 연구실 수준의 프로토타입을 넘어, 장시간 안정적으로 운영되어야 하는 산업용 애플리케이션의 요구사항을 충족시키기 위한 핵심적인 설계 결정이다.


PlanSys2, 특히 Planner 노드는 플러그인 아키텍처를 채택하여 뛰어난 확장성을 제공한다.3 이는 PlanSys2가 특정 계획 알고리즘에 종속되지 않는 유연한 '프레임워크'로서의 정체성을 갖게 하는 핵심 요소다.

AI 계획 분야는 끊임없이 새로운 알고리즘이 등장하며 빠르게 발전하고 있다. 만약 PlanSys2가 특정 플래너(예: POPF)를 시스템 내부에 하드코딩했다면, 새로운 플래너를 사용하기 위해 프레임워크 자체를 수정해야 하는 불편함이 있었을 것이다. 하지만 플러그인 구조 덕분에 사용자는 자신의 애플리케이션 요구사항에 가장 적합한 PDDL 솔버를 선택하여 쉽게 교체하거나 추가할 수 있다.11

예를 들어, 시간적 제약이 중요한 문제(temporal planning)를 다루고 있다면 TFD 플러그인을 사용하고, 수치 자원 관리(numeric fluents)가 중요하다면 POPF를 선택할 수 있다. 심지어 LPG-TD와 같이 공식적으로 지원되지 않는 플래너도, 사용자가 직접 플러그인 인터페이스에 맞춰 래퍼(wrapper) 클래스를 작성하면 PlanSys2에 통합하는 것이 가능하다.20

이러한 확장성은 PlanSys2를 단순한 '도구'가 아닌, 다양한 AI 계획 기술을 로봇 공학에 접목시키는 '플랫폼'으로 만들어준다. 연구자들은 자신이 개발한 새로운 계획 알고리즘을 PlanSys2 플러그인으로 만들어 실제 로봇이나 고충실도 시뮬레이터에서 손쉽게 테스트하고 검증할 수 있다. 이는 AI 계획 연구와 로봇 공학 응용 사이의 선순환을 촉진하는 중요한 역할을 한다.



PlanSys2가 분산된 ROS2 노드 환경에서 원활하게 동작하기 위해서는 PDDL이라는 텍스트 기반의 추상적 언어를 ROS2 통신 시스템이 이해할 수 있는 구체적인 이진 데이터 구조로 변환하는 과정이 필수적이다. 이 중요한 역할을 담당하는 것이 바로 `plansys2_msgs` 패키지다. 이 패키지는 PDDL의 복잡한 논리 구조를 효율적이고 타입-세이프(type-safe)한 ROS2 메시지로 인코딩하는 정교한 직렬화(serialization) 계층을 제공한다.

이 변환의 핵심에는 `plansys2_msgs/Node`와 `plansys2_msgs/Tree`라는 두 가지 메시지가 있다.21

- **`plansys2_msgs/Node`:** PDDL 표현식 트리의 개별 노드를 나타낸다. 이 메시지는 다양한 필드를 통해 PDDL의 여러 구문을 표현한다.
  - `node_type`: 노드가 나타내는 PDDL 구문의 종류를 정의한다. 예를 들어, `ACTION`, `PREDICATE`, `FUNCTION`, `AND`, `OR`, `NOT` 등이 있다.
  - `expression_type`: `node_type`이 `EXPRESSION`일 때, 어떤 종류의 수식인지를 명시한다. `COMP_GE` (>=), `COMP_LT` (<), `ARITH_ADD` (+) 등이 포함된다.
  - `modifier_type`: `node_type`이 `FUNCTION_MODIFIER`일 때, 함수 값의 변경 방식을 정의한다. `ASSIGN` (:=), `INCREASE` (`increase`), `DECREASE` (`decrease`) 등이 있다.
  - `name`: 술어나 함수의 이름을 저장한다 (예: `at`, `battery_level`).
  - `parameters`: 술어나 함수에 사용되는 파라미터 목록을 `plansys2_msgs/Param` 메시지 배열로 저장한다.
  - `value`: 숫자(literal) 노드나 함수(function)의 값을 저장한다.
  - `children`: 해당 노드의 자식 노드들의 ID(배열 내 인덱스)를 저장하여 트리 구조를 형성한다.
- **`plansys2_msgs/Tree`:** PDDL 표현식 전체를 나타내는 트리 구조 자체다. 이 메시지는 단순히 `plansys2_msgs/Node` 타입의 배열(`nodes`) 하나만을 필드로 가진다. 트리의 모든 노드들은 이 배열에 저장되며, 각 `Node` 메시지의 `children` 필드를 통해 서로 연결된다.

예를 들어, `(and (at r2d2 kitchen) (not (has_object r2d2)))`와 같은 PDDL 목표는 하나의 `Tree` 메시지로 변환된다. 이 `Tree`의 `nodes` 배열에는 `AND` 노드, `at` 술어 노드, `NOT` 노드, `has_object` 술어 노드 등이 포함되고, 이들의 부모-자식 관계가 `children` 필드를 통해 명시된다. 이 방식은 텍스트 파싱의 비효율성을 제거하고, 노드 간에 복잡한 논리 구조를 명확하고 구조화된 방식으로 전달할 수 있게 해준다.


PlanSys2의 각 컴포넌트는 ROS2의 표준 통신 메커니즘인 서비스(Services)와 액션(Actions)을 통해 자신의 기능을 외부에 노출하고 서로 상호작용한다. `plansys2_msgs` 패키지는 이러한 상호작용에 필요한 모든 인터페이스를 정의한다.21 개발자는 컨트롤러 노드와 같은 애플리케이션 코드에서 이 인터페이스들을 사용하여 PlanSys2 시스템을 제어하고 모니터링한다.

PlanSys2는 이러한 서비스와 액션 호출의 복잡성을 숨기기 위해 편리한 클라이언트 라이브러리(`DomainExpertClient`, `ProblemExpertClient`, `PlannerClient`, `ExecutorClient`)를 제공한다.4 개발자는 ROS2 서비스/액션의 저수준(low-level) 상세 구현을 알 필요 없이, 마치 로컬 라이브러리의 함수를 호출하듯 쉽게 PlanSys2의 기능에 접근할 수 있다.

아래 표는 PlanSys2의 핵심적인 지식 상호작용을 위한 주요 서비스와 액션을 요약한 것이다.

| 인터페이스 이름 (Interface Name)            | 타입 (Type) | 메시지 타입 (Message Type)                   | 대상 노드 (Target Node) | 설명 (Description)                                           |
| ------------------------------------------- | ----------- | -------------------------------------------- | ----------------------- | ------------------------------------------------------------ |
| `/domain_expert/get_domain`                 | Service     | `plansys2_msgs::srv::GetDomain`              | Domain Expert           | 현재 로드된 PDDL 도메인 전체를 문자열로 반환한다.            |
| `/domain_expert/get_domain_actions`         | Service     | `plansys2_msgs::srv::GetDomainActions`       | Domain Expert           | 도메인에 정의된 모든 행동(action)의 이름 목록을 반환한다.    |
| `/problem_expert/add_problem_instance`      | Service     | `plansys2_msgs::srv::AffectParam`            | Problem Expert          | 문제에 새로운 객체 인스턴스(예: 로봇 `r2d2`)를 추가한다.     |
| `/problem_expert/add_problem_goal`          | Service     | `plansys2_msgs::srv::AddProblemGoal`         | Problem Expert          | 문제에 새로운 목표(goal)를 추가한다. 목표는 `Tree` 메시지 형태로 표현된다. |
| `/problem_expert/affect_node`               | Service     | `plansys2_msgs::srv::AffectNode`             | Problem Expert          | 술어(predicate)나 함수(function)를 추가, 제거 또는 수정한다. |
| `/problem_expert/get_problem`               | Service     | `plansys2_msgs::srv::GetProblem`             | Problem Expert          | 현재 문제 상황(객체, 초기 상태, 목표) 전체를 PDDL 문자열로 반환한다. |
| `/problem_expert/is_problem_goal_satisfied` | Service     | `plansys2_msgs::srv::IsProblemGoalSatisfied` | Problem Expert          | 주어진 목표가 현재 상태에서 만족되었는지 여부를 확인한다.    |
| `/planner/get_plan`                         | Service     | `plansys2_msgs::srv::GetPlan`                | Planner                 | 현재 도메인과 문제를 기반으로 목표를 달성하기 위한 계획을 생성하여 반환한다. |
| `/executor/execute_plan`                    | Action      | `plansys2_msgs::action::ExecutePlan`         | Executor                | 주어진 계획의 실행을 시작하고, 실행 과정 동안 피드백을 제공하며, 최종 결과를 반환한다. |

이처럼 잘 정의된 인터페이스와 클라이언트 라이브러리는 PlanSys2를 강력하면서도 사용하기 쉬운 프레임워크로 만드는 핵심 요소다. 개발자는 이러한 고수준 API를 통해 PlanSys2의 내부 동작 원리를 깊이 이해하지 않고도 자신의 애플리케이션에 정교한 계획 및 실행 기능을 통합할 수 있다.



PlanSys2에서 계획이 생성되는 과정은 명확하게 정의된 파이프라인을 따른다. 이 과정은 외부 컨트롤러 노드가 Planner 노드의 `/get_plan` 서비스를 호출하는 것으로 시작된다.17 이 호출은 PlanSys2의 내부 컴포넌트들이 협력하여 최종적인 행동 계획을 만들어내는 일련의 단계를 촉발시킨다.

1. **지식 수집:** `/get_plan` 서비스 요청을 받은 Planner 노드는 먼저 계획 생성에 필요한 정보를 수집한다. 이를 위해 Domain Expert 클라이언트를 통해 현재 로드된 PDDL 도메인 정의를, Problem Expert 클라이언트를 통해 현재의 객체 인스턴스, 술어, 그리고 목표가 포함된 PDDL 문제 정의를 각각 문자열 형태로 요청한다.3
2. **임시 파일 생성:** Planner 노드는 수집한 도메인과 문제 문자열을 로컬 파일 시스템의 임시 디렉토리(일반적으로 `/tmp/${namespace}/`)에 각각 `domain.pddl`과 `problem.pddl`이라는 이름의 파일로 저장한다.13 이 파일들은 외부 PDDL 솔버가 입력으로 사용할 표준 형식의 파일이다. 이 방식은 PlanSys2가 외부 솔버의 구현 방식에 의존하지 않고 독립적으로 동작할 수 있게 해준다. 또한, 디버깅 시 개발자가 이 파일들을 직접 확인하여 플래너에 어떤 정보가 전달되었는지 명확히 파악할 수 있다는 장점이 있다.
3. **플래너 플러그인 호출:** Planner 노드는 설정 파일(`plansys2_params.yaml`)에 지정된 플래너 플러그인을 로드하고, 이 플러그인의 `getPlan` 메소드를 호출한다.18 이 때, 방금 생성한 `domain.pddl`과 `problem.pddl` 파일의 경로를 인자로 전달한다.
4. **외부 솔버 실행 및 결과 파싱:** 플러그인 내부에서는 전달받은 파일 경로를 사용하여 외부 PDDL 솔버의 실행 파일을 시스템 콜(system call) 등으로 호출한다. 외부 솔버는 계획 생성을 수행하고, 그 결과를 텍스트 파일(예: `/tmp/${namespace}/plan.pddl`)로 출력한다. 플러그인은 이 출력 파일을 읽고 파싱하여, 시간 정보, 행동 이름, 행동 파라미터 등으로 구성된 구조화된 계획 데이터(예: `plansys2_msgs::msg::Plan`)로 변환한다.13
5. **계획 반환:** 플러그인이 파싱한 계획 데이터를 반환하면, Planner 노드는 이를 `/get_plan` 서비스의 응답에 담아 최초 요청을 보냈던 컨트롤러 노드에게 전달한다. 이로써 하나의 계획 생성 사이클이 완료된다.

이 파이프라인의 설계는 PlanSys2가 AI 계획 알고리즘 자체를 구현하는 대신, 기존의 강력한 외부 도구들을 '호출'하고 그 결과를 '해석'하는 '어댑터(Adapter)' 또는 '브릿지(Bridge)' 역할에 충실하도록 만든다. 이 덕분에 PlanSys2는 최신 AI 계획 연구 성과를 신속하게 흡수하며 발전할 수 있는 유연하고 강력한 구조를 갖추게 되었다.


PlanSys2는 기본적으로 두 가지의 검증된 PDDL 플래너를 플러그인 형태로 제공하여, 사용자가 문제의 특성에 맞게 선택할 수 있도록 한다.3

- **POPF (Partial-Order Planning Forwards):** PlanSys2의 기본 플래너로, PDDL 2.1 표준의 일부인 수치적 유창성(numeric fluents, 예: 배터리 잔량, 거리)을 지원하는 강력한 플래너다.9 POPF는 전방 탐색(forward-chaining)과 부분 순서 계획(partial-order planning)을 결합한 방식을 사용한다. 부분 순서 계획은 행동들 간의 엄격한 전체 순서를 강요하지 않고, 반드시 지켜져야 하는 인과 관계만을 정의하기 때문에 더 유연한 계획을 생성할 수 있다. 자원 관리나 수치적 조건이 중요한 문제에 적합하다.
- **TFD (Temporal Fast Downward):** 시간적 제약(temporal constraints)과 지속적 행동(durative actions)을 지원하는 플래너다.3 PDDL 2.1의 시간적 모델링 기능을 완벽하게 활용할 수 있게 해준다. 예를 들어, 'A 행동은 B 행동이 시작된 후 5초에서 10초 사이에 시작되어야 한다' 또는 'C 행동은 30초 동안 지속되어야 한다'와 같은 복잡한 시간적 요구사항을 처리할 수 있다. 로봇의 행동이 즉각적으로 끝나지 않고 일정 시간 지속되는 현실적인 시나리오를 모델링하는 데 필수적이다.

아래 표는 PlanSys2에서 사용 가능한 주요 플래너들의 특징을 비교한 것이다.

| 플래너 (Planner) | PDDL 지원 (PDDL Support)                           | 계획 접근 방식 (Planning Approach)  | 강점 (Strengths)                                             | 고려사항 (Weaknesses/Considerations)                         |
| ---------------- | -------------------------------------------------- | ----------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **POPF**         | PDDL 2.1, Numeric Fluents                          | Forward-chaining partial-order      | 수치 자원 관리, 유연한 부분 순서 계획 생성                   | 시간적 제약(Temporal constraints) 지원 미흡, 최적성(optimality) 보장 안됨 3 |
| **TFD**          | PDDL 2.1, Durative Actions, Timed Initial Literals | Heuristic Forward Search (Temporal) | 복잡한 시간적 제약 조건 처리, 지속적 행동 모델링             | 일부 오래된 버전은 Python 2.7 의존성 존재 26, 수치 계획 능력은 POPF 대비 제한적일 수 있음 |
| **LPG-TD**       | PDDL 2.1, Numeric, Temporal                        | Local Search, Heuristic             | 수치 및 시간적 문제 모두에 대해 우수한 성능, 품질과 속도 간의 균형 조절 가능 | PlanSys2에 기본 포함되지 않으며, 별도 플러그인 설치 필요 20  |
| **OPTIC**        | PDDL 3.0 (Preferences), Temporal                   | Anytime Heuristic Search            | 시간적 계획과 선호도(preferences) 기반 계획 지원, 시간에 따라 더 나은 계획을 점진적으로 찾아냄 | PlanSys2 테스트에서 제거됨 27, 복잡한 설정이 필요할 수 있음  |

사용자는 자신의 애플리케이션이 주로 자원 관리에 초점을 맞추는지, 아니면 복잡한 시간 동기화에 초점을 맞추는지에 따라 적절한 플래너를 선택해야 한다. 이러한 선택의 유연성은 PlanSys2의 가장 큰 장점 중 하나다.


PlanSys2의 진정한 강력함은 사용자가 자신만의 커스텀 PDDL 플래너를 시스템에 통합할 수 있는 능력에서 나온다. 이는 연구자가 새로운 계획 알고리즘을 개발했거나, 특정 문제에 고도로 최적화된 상용 솔버를 사용하고자 할 때 매우 유용하다. 통합 과정은 다음 세 단계로 이루어진다.

1단계: 플래너 클래스 구현

먼저, plansys2::PlanSolverBase라는 추상 базовый 클래스를 상속받는 새로운 C++ 클래스를 만들어야 한다. 이 클래스는 플래너 플러그인의 인터페이스 역할을 한다. 가장 중요한 것은 getPlan이라는 가상 함수를 재정의(override)하는 것이다.28

```C++
#include "plansys2_planner/PlanSolverBase.hpp"

class MyCustomPlanner : public plansys2::PlanSolverBase
{
public:
  void configure(
    rclcpp_lifecycle::LifecycleNode::SharedPtr node,
    std::string & plugin_name) override
  {
    // 플러그인 초기 설정 (필요 시)
  }

  std::optional<plansys2_msgs::msg::Plan> getPlan(
    const std::string & domain,
    const std::string & problem,
    const std::string & node_namespace = "") override
  {
    // 1. domain과 problem 문자열을 임시 파일로 저장
    // 2. system() 이나 popen() 등을 사용하여 커스텀 플래너 실행 파일 호출
    //    (예: /path/to/my_planner -d domain.pddl -p problem.pddl -o plan.txt)
    // 3. 생성된 계획 파일(plan.txt)을 읽고 파싱
    // 4. 파싱된 결과를 plansys2_msgs::msg::Plan 메시지에 채워서 반환
    // 5. 계획 생성 실패 시 std::nullopt 반환
  }
};
```

이 코드에서 핵심은 `getPlan` 함수 내의 로직이다. 이 함수는 PlanSys2의 Planner 노드로부터 도메인과 문제 PDDL 텍스트를 받아, 이를 커스텀 플래너가 이해할 수 있는 방식으로 전달하고(주로 파일 사용), 플래너의 출력을 다시 PlanSys2가 이해할 수 있는 `Plan` 메시지 형태로 변환하는 책임을 진다.

2단계: 플러그인 등록

작성한 클래스를 ROS2 시스템이 런타임에 동적으로 로드할 수 있도록 pluginlib을 사용하여 등록해야 한다.28 이는 소스 파일 하단에 매크로를 추가하고, 플러그인 정보를 담은 XML 파일을 작성하는 것으로 이루어진다.

- **소스 파일(.cpp)에 매크로 추가:**

  ```C++
  #include "pluginlib/class_list_macros.hpp"
  PLUGINLIB_EXPORT_CLASS(my_planning_plugin::MyCustomPlanner, plansys2::PlanSolverBase)
  ```

- **플러그인 설명 XML 파일 생성 (예: `my_planner_plugin.xml`):**

  

  ```XML
  <library path="my_planning_plugin">
    <class name="my_planning_plugin/MyCustomPlanner"
           type="my_planning_plugin::MyCustomPlanner"
           base_class_type="plansys2::PlanSolverBase">
      <description>My custom PDDL planner plugin.</description>
    </class>
  </library>
  ```

이 XML 파일은 패키지의 `package.xml` 파일에 `export` 태그를 통해 명시되어야 한다.

3단계: 설정 파일 수정

마지막으로, PlanSys2가 새로운 플러그인을 인식하고 사용하도록 설정 파일을 수정해야 한다. plansys2_bringup 패키지 등에 포함된 plansys2_params.yaml 파일을 열어 다음과 같이 수정한다.20

```YAML
planner:
  ros__parameters:
    plan_solver_plugins: # 사용할 플래너 이름 지정

    POPF:
      plugin: "plansys2/POPFPlanSolver"
    TFD:
      plugin: "plansys2/TFDPlanSolver"
    MY_CUSTOM_PLANNER: # 새로운 플래너 섹션 추가
      plugin: "my_planning_plugin/MyCustomPlanner" # 2단계에서 등록한 클래스 이름
```

`plan_solver_plugins` 파라미터에 새로 추가한 플러그인의 이름(`MY_CUSTOM_PLANNER`)을 명시하면, Planner 노드는 시작 시 이 플러그인을 로드하여 사용하게 된다. 이처럼 체계적인 플러그인 시스템은 PlanSys2의 확장성을 보장하는 핵심적인 기술이다.



PlanSys2의 가장 혁신적이고 강력한 기능 중 하나는 Planner가 생성한 선형적인(linear) PDDL 계획을 병렬 실행이 가능한 비선형적(non-linear) 행동 트리(Behavior Tree, BT)로 자동 변환하는 능력이다.2 이는 단순한 계획 실행을 넘어, 실행 자체를 최적화하고 강건하게 만드는 핵심 메커니즘이다.

전통적인 플래너가 생성하는 계획은 보통 시간 순서에 따른 행동의 목록이다 (예: 1. `move A to B`, 2. `pick C`, 3. `move B to D`). 이를 그대로 순차적으로 실행하면, 서로 독립적인 행동이라 할지라도 앞선 행동이 완료될 때까지 기다려야 하는 비효율이 발생한다. PlanSys2의 Executor는 이러한 비효율을 극복하기 위해 다음과 같은 과정을 거친다.

1. **실행 그래프 분석:** 먼저, 주어진 계획에 포함된 모든 행동들의 전제조건(preconditions)과 효과(effects)를 분석하여 행동 간의 인과 관계(causal links)를 파악한다. 이를 통해 어떤 행동이 다른 행동의 선행 조건이 되는지를 나타내는 의존성 그래프, 즉 실행 그래프(execution graph)를 구축한다.2
2. **병렬 실행 흐름 식별:** 실행 그래프를 분석하면, 서로 의존성이 없는 행동들을 식별할 수 있다. 예를 들어, '왼팔로 물건 A 집기'와 '오른팔로 물건 B 집기'는 동시에 수행될 수 있다. PlanSys2는 이러한 병렬 실행이 가능한 행동 그룹들을 찾아낸다.
3. **행동 트리 생성:** 식별된 의존성과 병렬 실행 흐름을 바탕으로, `BehaviorTree.CPP` 라이브러리 30의 제어 노드(Control Nodes)를 사용하여 최적화된 실행 트리를 동적으로 생성한다.
   - 반드시 순서대로 실행되어야 하는 행동들은 `Sequence` 노드로 연결된다.
   - 동시에 실행될 수 있는 행동들은 `Parallel` 노드로 묶인다.
   - 조건에 따라 다른 행동을 선택해야 하는 경우 `Fallback`(또는 `Selector`) 노드가 사용될 수 있다.

이 변환 과정의 결과물은 단순한 행동 목록이 아닌, 실행 효율성과 환경 변화에 대한 반응성이 극대화된 정교한 실행 로직이다. 학술 논문에서 제시된 벤치마크 결과에 따르면, 이러한 BT 기반 병렬 실행은 다중 로봇 시나리오에서 순차 실행 방식에 비해 실행 시간을 극적으로 단축시키는 것으로 나타났다.29 이처럼 PlanSys2는 PDDL의 정적인 세계와 로봇 공학의 동적인 세계 사이의 간극을 '최적화된 행동 트리'라는 강력한 매개체를 통해 효과적으로 연결한다.


PlanSys2는 단일 로봇뿐만 아니라 다중 로봇 시스템에서의 협력 작업을 지원하도록 설계되었다. 이를 가능하게 하는 핵심 기술이 바로 '액션 오션 프로토콜(Action Auction Protocol)'이다.3

Executor가 계획에 따라 특정 행동(예: `(move r2d2 kitchen hall)`)을 실행해야 할 때, 네트워크 상에는 해당 `move` 행동을 수행할 수 있는 능력을 가진 노드(Action Performer)가 여러 개 존재할 수 있다. 예를 들어, 로봇 `r2d2`와 `c3po`가 각각 자신의 `move` 액션 노드를 가지고 있는 상황을 생각할 수 있다. 이때 Executor는 단순히 아무 노드에게나 명령을 내리는 것이 아니라, 일종의 '경매(auction)'를 시작한다.

이 경매 시스템은 각 Action Performer 노드가 자신의 전문 분야를 선언하는 방식으로 동작한다. 이는 ROS 파라미터인 `specialized_arguments`를 통해 설정된다.6 예를 들어, `r2d2` 로봇의 `move` 액션 노드는 자신의 `specialized_arguments` 파라미터에 `{"r2d2"}`라고 설정할 수 있다.

Executor가 `(move r2d2 kitchen hall)` 행동에 대한 경매를 시작하면, 모든 `move` 액션 노드들이 이 요청을 받는다. 하지만 각 노드는 행동의 인자(`r2d2`, `kitchen`, `hall`)와 자신의 `specialized_arguments`를 비교하여, 자신이 이 행동의 '전문가'인지를 판단한다. 위 예시에서는 `r2d2`의 `move` 노드만이 첫 번째 인자가 `r2d2`와 일치하므로 경매에 '입찰'하고, `c3po`의 노드는 입찰하지 않는다. Executor는 이 입찰에 성공한 노드에게 최종적으로 행동 실행을 위임한다.

이 프로토콜은 단순히 로봇을 구분하는 것을 넘어, 더 세분화된 전문화도 가능하다. 예를 들어, `(pick apple)`과 `(pick cup)`이라는 행동에 대해, 하나는 부드러운 물체를 집는 데 특화된 그리퍼를 제어하는 노드가, 다른 하나는 단단한 물체를 집는 데 특화된 그리퍼를 제어하는 노드가 각각 응답하도록 만들 수 있다. 이처럼 액션 오션 프로토콜은 분산된 환경에서 유연하고 확장 가능한 방식으로 작업 할당을 수행하는 강력한 메커니즘을 제공한다.


로봇이 동작하는 현실 세계는 불확실성으로 가득 차 있다. 따라서 계획 실행 시스템은 예기치 않은 실패에 강건하게 대처할 수 있어야 한다. PlanSys2의 Executor는 여러 메커니즘을 통해 실행 중 실패를 감지하고 처리한다.

- **전제조건 및 효과 검사:** Executor는 생성된 행동 트리를 실행하면서, 각 행동의 시작 전(`at start`), 실행 중(`over all`), 그리고 종료 후(`at end`)에 PDDL에 명시된 조건들이 만족되는지를 지속적으로 검사한다.19 예를 들어, `move` 행동이 실행되는 동안 `over all` 조건으로 '로봇의 바퀴가 땅에 닿아 있어야 한다'가 명시되어 있다면, Executor는 이 조건이 계속 참인지 확인한다. 만약 로봇이 장애물에 걸려 바퀴가 공중에 뜨게 되면, Executor는 이 조건 위반을 감지한다.
- **계획 취소:** 조건 위반이 감지되면, Executor는 즉시 현재 진행 중인 계획 실행을 실패로 간주하고 전체 행동 트리의 실행을 중단한다.19 이는 시스템이 위험하거나 비효율적인 상태로 계속 진행되는 것을 막는 중요한 안전장치다. 계획 실행이 실패했다는 결과는 Executor의 액션 클라이언트(주로 컨트롤러 노드)에게 전달되며, 이는 컨트롤러가 재계획(replanning)과 같은 후속 조치를 취하는 계기가 된다.
- **액션 타임아웃:** 특정 행동이 비정상적으로 오래 걸리는 경우를 대비하여, `action_timeouts` 파라미터를 통해 시간제한을 설정할 수 있다.13 예를 들어, `move` 액션의 예상 소요 시간이 10초이고 `duration_overrun_percentage`를 20%로 설정했다면, 이 액션이 12초를 초과하여 실행될 경우 Executor는 이를 강제로 중단시키고 실패로 처리한다. 이 기능은 시스템이 특정 행동에서 무한정 대기하는 교착 상태(deadlock)에 빠지는 것을 방지한다.

이러한 다층적인 실패 처리 메커니즘은 PlanSys2가 예측 불가능한 환경에서도 안정적으로 동작할 수 있도록 보장하는 핵심 요소다.


PlanSys2는 개발자에게 PDDL에 정의된 추상적인 행동을 실제 로봇의 동작으로 연결하는 두 가지 유연한 방법을 제공한다.

1. C++ ActionExecutorClient를 이용한 직접 구현:

   이 방식은 plansys2::ActionExecutorClient 클래스를 상속받는 C++ 노드를 작성하는 것이다.19 개발자는 

   `do_work()`라는 가상 함수를 재정의(override)하여 행동의 핵심 로직을 구현한다. 이 `do_work()` 함수는 생성자에서 지정한 주기(예: 250ms, 즉 4Hz)마다 반복적으로 호출된다.

   ```C++
   class MoveAction : public plansys2::ActionExecutorClient
   {
   public:
     MoveAction() : plansys2::ActionExecutorClient("move", 250ms) {... }
   private:
     void do_work() {
       // (move r2d2 kitchen hall) 에서 {"r2d2", "kitchen", "hall"} 가져오기
       const auto& args = get_arguments(); 
       // Nav2 액션 클라이언트 호출 등 실제 이동 로직 수행
      ...
       // 진행률 피드백 전송
       send_feedback(progress_percent, "Moving...");
       // 이동 완료 시
       if (is_done) {
         finish(true, 1.0, "Move completed");
       }
     }
   };
   ```

   이 방식은 행동의 로직이 비교적 단순하거나, 다른 ROS2 시스템(예: Nav2, MoveIt)과 직접적으로 상호작용해야 할 때 유용하다. `send_feedback()`과 `finish()`와 같은 내장 함수를 통해 Executor에게 진행 상황과 최종 결과를 쉽게 알릴 수 있다.

2. 행동 트리(BT)를 이용한 모듈식 구현:

   행동의 내부 로직 자체가 여러 단계의 순차적 또는 조건부 작업으로 구성될 경우, 이를 하나의 C++ 함수로 구현하는 것은 복잡하고 재사용성이 떨어질 수 있다. PlanSys2는 이러한 문제를 해결하기 위해 plansys2_bt_actions 패키지를 제공한다.31

   이 패키지의 bt_action_node라는 제네릭(generic) 실행 파일을 사용하면, PDDL 행동 하나를 별도의 행동 트리(BT)로 정의할 수 있다. 개발자는 C++ 코드를 작성하는 대신, 행동의 로직을 기술하는 XML 파일을 작성한다.

   예를 들어, (transport?piece?from?to)라는 PDDL 행동은 다음과 같은 BT로 구현될 수 있다.31

   ```XML
   <root main_tree_to_execute="MainTree">
     <BehaviorTree ID="MainTree">
       <Sequence name="transport_sequence">
         <Move goal="{arg2}"/>      <OpenGripper/>
         <ApproachObject object="{arg1}"/>
         <CloseGripper/>
         <Move goal="{arg3}"/>      <OpenGripper/>
       </Sequence>
     </BehaviorTree>
   </root>
   ```

   이 방식의 가장 큰 장점은 **모듈성**과 **재사용성**이다. `Move`, `OpenGripper`와 같은 기본 BT 노드들을 한 번 구현해두면, 다양한 PDDL 행동(예: `transport`, `fetch`, `tidy_up`)의 BT 구현에서 이들을 레고 블록처럼 조합하여 재사용할 수 있다. 이는 복잡한 행동 로직을 보다 직관적으로 설계하고 유지보수하는 데 매우 효과적이다.

개발자는 구현하려는 행동의 복잡도와 재사용 필요성에 따라 이 두 가지 방식 중 적절한 것을 선택할 수 있다. 이러한 유연성은 PlanSys2의 실용성을 높이는 중요한 특징이다.



자율 로봇 시스템이 현실 세계에서 유용하기 위해서는, 초기에 수립한 계획이 더 이상 유효하지 않게 되었을 때 이를 인지하고 새로운 계획을 수립하는 능력, 즉 재계획(Replanning)이 필수적이다. PlanSys2는 재계획이 필요한 다양한 상황을 감지하고 이에 대응할 수 있는 메커니즘을 제공한다. 재계획을 촉발하는 주요 트리거는 다음과 같다.

- **실행 실패 (Execution Failure):** 가장 명백한 트리거는 Executor가 현재 계획의 실행에 실패했을 때다. 이는 행동의 전제조건(precondition)이 만족되지 않거나(예: 집으려던 물체가 사라짐), 행동이 타임아웃되거나, 하위 시스템(예: Nav2)이 실패를 보고하는 경우에 발생한다.19 Executor는 계획 실패를 상위의 컨트롤러 노드에 알리고, 컨트롤러는 이 정보를 바탕으로 재계획을 결정할 수 있다.
- **예기치 않은 환경 변화 (Unexpected Environmental Change):** 로봇이 계획을 수행하는 동안에도 세상은 계속 변한다. 예를 들어, 로봇이 복도를 지나가려는데 갑자기 사람이 나타나 길을 막는 상황이 발생할 수 있다. 이러한 변화는 센서 데이터를 처리하는 노드에 의해 감지되고, 이 노드는 Problem Expert의 상태(예: `(path_blocked hall)` 술어를 추가)를 업데이트할 수 있다. Problem Expert는 자신의 지식 베이스가 변경될 때마다 토픽을 발행하여 이 사실을 시스템 전체에 알린다.4 컨트롤러 노드는 이 토픽을 구독하고 있다가, 현재 계획의 유효성에 영향을 미치는 중요한 변화가 감지되면 재계획을 시작할 수 있다.
- **새로운 목표 또는 우선순위 변경 (New Goal or Priority Change):** 시스템 운영자나 상위 레벨의 의사결정 시스템이 로봇에게 새로운 임무를 부여하거나, 현재 임무보다 더 시급한 임무를 지시할 수 있다. 이 경우, 컨트롤러 노드는 현재 실행 중인 계획을 취소하고, 새로운 목표를 Problem Expert에 설정한 뒤, 이 새로운 목표를 달성하기 위한 재계획을 Planner에 요청한다.5

이처럼 PlanSys2는 재계획을 위한 '신호'와 '정보'를 제공하는 다양한 채널을 마련해두고 있다. 하지만 재계획을 실제로 '결정'하고 '시작'하는 책임은 애플리케이션의 특정 로직을 담고 있는 컨트롤러 노드에 있다는 점을 이해하는 것이 중요하다.


PlanSys2는 단순히 계획을 다시 세우는 것을 넘어, 상황에 따라 더 정교하게 대응할 수 있는 여러 재계획 전략을 지원하거나 논의하고 있다. 이는 프레임워크가 실세계의 복잡성에 얼마나 잘 대처할 수 있는지를 보여주는 지표다.5

- **기본 재계획 (Basic Replanning):** 가장 간단한 전략이다. 컨트롤러가 새로운 계획을 Executor에게 보내면, Executor는 현재 실행 중이던 계획을 즉시 중단 및 취소하고, 새로운 계획을 처음부터 실행하기 시작한다. 이는 가장 구현하기 쉽고 예측 가능한 방식이지만, 로봇의 행동이 갑자기 끊기는 것처럼 보일 수 있다.
- **점진적 재계획 (Smooth Replanning):** 이 전략은 로봇의 행동을 더 부드럽고 자연스럽게 만들기 위해 고안되었다. 새로운 계획이 도착했을 때, Executor는 현재 실행 중인 행동이 새 계획의 첫 번째 행동과 동일한지 확인한다. 만약 동일하다면, 현재 행동을 중단하지 않고 계속해서 실행을 완료한 뒤, 새 계획의 두 번째 행동부터 이어서 수행한다. 예를 들어, 로봇이 `(move kitchen living_room)`을 수행하던 중, `(move kitchen living_room)`으로 시작하는 새 계획을 받으면, 현재의 이동을 마친 후에 다음 행동으로 넘어간다. 이는 불필요한 중단과 재시작을 줄여 효율성과 사용자 경험을 향상시킨다.
- **개선된 취소 (Improved Cancellation):** 현재 개발 중인 아이디어로, 취소되는 행동이 안전하게 종료될 시간을 보장하는 메커니즘이다. 예를 들어, 로봇 팔이 움직이는 도중에 취소 명령을 받으면, 즉시 전원을 차단하는 것이 아니라 현재 동작을 안전하게 멈추고 초기 자세로 돌아가는 등의 '정리' 작업을 수행할 시간을 주는 것이다. 이를 위해 새 계획의 시작이 다소 지연될 수 있지만, 시스템의 안전성과 안정성을 크게 높일 수 있다.

이러한 다양한 재계획 전략은 PlanSys2가 단순한 계획-실행 시스템을 넘어, 동적이고 예측 불가능한 환경에 지능적으로 적응하는 자율 시스템을 구축하기 위한 정교한 도구들을 제공하고 있음을 보여준다.


PlanSys2 아키텍처에서 재계획의 '지휘자' 역할을 하는 것은 바로 사용자 정의 컨트롤러 노드(Controller Node)다.6 PlanSys2의 핵심 컴포넌트들(Domain Expert, Problem Expert, Planner, Executor)은 재계획에 필요한 기능적 '부품'들을 제공하지만, 이 부품들을 언제 어떻게 조립하여 재계획이라는 전체 프로세스를 완성할지를 결정하는 것은 컨트롤러의 몫이다.

컨트롤러 노드의 핵심 책임은 다음과 같다.

1. **미션 관리:** 애플리케이션의 전체적인 목표와 임무를 관리한다. 예를 들어, '모든 방을 순서대로 순찰하라'는 임무가 있다면, 컨트롤러는 현재 어떤 방을 순찰할 차례인지 상태를 추적한다. `plansys2_patrol_navigation_example`의 `patrolling_controller_node`는 이러한 로직을 FSM(Finite State Machine) 형태로 구현하여, 하나의 웨이포인트 순찰이 완료되면 다음 웨이포인트를 새로운 목표로 설정한다.22
2. **실행 모니터링:** Executor의 액션 서버에 목표를 전달하고, 실행 과정 동안 피드백(feedback)과 최종 결과(result)를 지속적으로 수신한다. 이를 통해 현재 계획이 성공적으로 진행되고 있는지, 아니면 실패했는지를 파악한다.
3. **상태 감지 및 지식 업데이트:** 센서 데이터 처리 노드나 다른 시스템으로부터 정보를 받아, Problem Expert의 지식 베이스를 최신 상태로 유지한다. 예를 들어, 카메라를 통해 새로운 물체를 발견하면 `(object_at...)` 술어를 Problem Expert에 추가한다.
4. **재계획 결정 및 트리거:** 실행 실패, 환경 변화, 새로운 목표 등 재계획 트리거를 감지하면, "지금 재계획이 필요하다"고 판단한다. 이 판단에 따라 새로운 목표를 Problem Expert에 설정하고, Planner에 `/get_plan` 서비스를 호출하여 새로운 계획을 요청한다. 그리고 새로 받은 계획을 다시 Executor에게 전달하여 실행을 시작시킨다.

이러한 설계는 엄청난 유연성을 제공한다. 재계획을 얼마나 자주 할지, 어떤 종류의 실패에서 재계획을 시도할지, 재계획 시도 전 어떤 복구 절차를 수행할지 등 애플리케이션의 특성에 맞는 모든 정책(policy)을 컨트롤러 노드에 자유롭게 구현할 수 있다. PlanSys2는 정책을 강요하는 대신, 정책을 구현하는 데 필요한 강력한 도구들을 제공하는 데 집중하는 것이다.



PlanSys2의 성능과 확장성은 실제 로봇 애플리케이션에 적용 가능성을 평가하는 중요한 척도다. 학술 논문과 커뮤니티의 보고를 통해 그 성능을 다각도로 분석할 수 있다.

주요 성능 향상 요인은 Executor가 선형적인 계획을 병렬 실행이 가능한 행동 트리(BT)로 변환하는 데 있다. IROS 2021 논문에서 제시된 다중 로봇 요리 실험 벤치마크는 이를 명확히 보여준다.3 3대의 로봇을 사용한 시뮬레이션에서, 모든 행동을 순차적으로 실행했을 때보다 BT를 통해 독립적인 행동들을 병렬로 실행했을 때 전체 임무 완료 시간이 현저하게 단축되었다.29 이는 로봇의 유휴 시간을 최소화하고 시스템의 전체 처리량(throughput)을 극대화하는 PlanSys2의 핵심적인 장점이다.

그러나 대규모 문제(large-scale problems)에서는 성능 병목 현상이 발생할 수 있다. GitHub 이슈 트래커에는 PlanSys2의 확장성과 관련된 중요한 논의들이 기록되어 있다. 예를 들어, '비효율적인 존재 조건부 전제조건 해결(Inefficient existential preconditions resolution)'이라는 제목의 이슈(#328)는 PDDL의 `(exists...)` 구문을 처리하는 과정에서 발생하는 성능 문제를 지적한다.35 수백 개의 객체와 복잡한 조건이 얽힌 문제에서, 계획을 생성하기 전 문제 파일을 검증하고 전처리하는 단계에서 상당한 시간이 소요될 수 있다는 것이다. 이는 Problem Expert가 모든 가능한 인스턴스 조합을 확인해야 하기 때문에 발생하는 조합 폭발(combinatorial explosion) 문제와 관련이 있다.

또한, PlanSys2는 ROS2 위에서 동작하므로, 기반이 되는 통신 미들웨어인 DDS의 성능 특성에 간접적인 영향을 받는다. 예를 들어, 일부 DDS 구현체는 매우 큰 메시지(예: 고해상도 지도나 포인트 클라우드)를 높은 빈도로 전송할 때 성능 저하를 보일 수 있다는 보고가 있다.36

`ros2 topic hz`와 같은 모니터링 도구조차도 대용량 메시지를 처리할 때 실제 주파수보다 낮게 측정되는 경우가 있어, 성능 분석 시 주의가 필요하다.37 PlanSys2 자체가 대용량 센서 데이터를 직접 처리하지는 않지만, PlanSys2의 상태를 업데이트하는 노드들이 이러한 데이터를 다룬다면 전체 시스템의 반응성에 영향을 미칠 수 있다.

결론적으로, PlanSys2는 일반적인 작업 계획 시나리오에서 BT 기반 최적화를 통해 뛰어난 실행 성능을 보이지만, 수백 개 이상의 객체를 다루는 매우 복잡한 도메인에서는 계획 생성 전 단계에서 성능 병목이 발생할 수 있다. 따라서 대규모 애플리케이션에 적용하기 전에는 목표 도메인에 대한 충분한 성능 테스트가 선행되어야 한다.


PlanSys2는 강력하고 성숙한 프레임워크이지만, 완벽하지는 않으며 몇 가지 명확한 한계점을 가지고 있다. 개발자는 이러한 한계점을 명확히 인지하고, 필요에 따라 애플리케이션 수준에서 보완적인 로직을 구현해야 한다.

- **PDDL 지원 범위의 한계:** PlanSys2는 PDDL 2.1의 주요 기능(지속적 행동, 수치적 유창성 등)을 지원하지만, 최신 PDDL 표준의 모든 기능을 완벽하게 지원하지는 않는다.3 특히 

  `forall` (전체 한정자)이나 `imply` (함의)와 같은 더 복잡한 논리 표현식을 행동 트리로 변환하는 데 어려움이 있다는 이슈가 제기되었다.35 이는 모델링의 표현력을 제한할 수 있으며, 복잡한 도메인을 PDDL로 표현하려는 개발자에게 장벽이 될 수 있다.

- **플래너 다양성 부족:** 공식적으로는 POPF와 TFD 두 가지 플래너만이 안정적으로 지원되고 테스트된다.3 플러그인 시스템을 통해 다른 플래너를 통합할 수는 있지만, 이는 전적으로 사용자나 커뮤니티의 노력에 달려있다. 다양한 계획 문제 유형(예: 확률적 계획, 인식 기반 계획)에 맞는 최적의 플래너를 '즉시' 사용하기에는 생태계가 아직 부족하다.

- **행동 중단 시 상태 일관성 문제:** 이는 실용적인 관점에서 가장 중요한 한계 중 하나다. 만약 로봇이 행동을 수행하던 중 실패하거나 외부 요인에 의해 취소될 경우, PlanSys2는 시스템의 상태(Problem Expert의 지식)를 행동 시작 전으로 되돌리는 '트랜잭션 롤백(transaction rollback)' 기능을 자동으로 제공하지 않는다.3 예를 들어, 로봇이 물건을 집는 `pick` 행동이 '그리퍼를 물건으로 이동' -> '그리퍼 닫기' -> '팔 들어올리기'의 3단계로 구성되어 있고, '그리퍼 닫기' 단계에서 실패했다고 가정해보자. 이때 `at start` 효과로 적용된 '로봇 팔이 물건 위치에 있다'는 술어는 참이 되지만, `at end` 효과인 '로봇이 물건을 들고 있다'는 거짓이 된다. 이로 인해 Problem Expert의 지식은 실제 세계와 불일치하는 비일관적(inconsistent) 상태에 빠질 수 있다.38 이 문제를 해결하는 책임은 전적으로 애플리케이션 컨트롤러에 있으며, 컨트롤러는 실패를 감지한 후 센서 정보 등을 통해 실제 세계를 다시 파악하고 Problem Expert의 상태를 수동으로 보정해주는 복잡한 로직을 구현해야 한다.

- **비결정성 처리의 부재:** 현재 PlanSys2는 기본적으로 결정론적(deterministic) 환경을 가정한다. 센서의 노이즈, 행동의 불확실한 결과(예: 물건을 집을 때 5% 확률로 떨어뜨림)와 같은 비결정적 요소를 PDDL 모델에 직접 표현하고 이를 처리하는 기능은 부족하다. ROS Discourse 포럼에서도 이와 관련된 질문이 제기된 바 있다.5

이러한 한계들은 PlanSys2가 모든 문제를 해결해주는 '만능 열쇠'가 아님을 보여준다. PlanSys2는 강력한 '엔진'을 제공하지만, 이 엔진을 실제 도로에서 안전하게 운전하기 위한 '섀시'와 '에어백' 같은 안전장치들은 상당 부분 개발자의 몫으로 남아있다.


PDDL을 사용하여 복잡한 로봇 작업을 모델링하는 것은 매우 어렵고 오류가 발생하기 쉬운 과정이다. 특히 계획 생성이 실패했을 때, 그 원인을 찾는 것은 숙련된 개발자에게도 상당한 도전이 될 수 있다.23 "문법 오류인가? 도메인 논리에 모순이 있는가? 아니면 단순히 문제 파일에 필요한 술어나 객체가 누락된 것인가?"와 같은 질문에 답하기는 쉽지 않다.9

PlanSys2는 이러한 디버깅의 어려움을 완화하기 위해 몇 가지 보조 장치를 제공한다. Planner 노드가 계획 생성을 시도할 때마다, 외부 솔버를 호출하는 데 사용된 PDDL 파일들을 임시 디렉토리(예: `/tmp/plansys2/`)에 그대로 남겨둔다.23

- `/tmp/plansys2/domain.pddl`: Domain Expert로부터 받은 도메인 파일.
- `/tmp/plansys2/problem.pddl`: Problem Expert로부터 받은 문제 파일.
- `/tmp/plansys2/plan.pddl`: 외부 플래너가 생성한 원시(raw) 출력 파일.

계획이 생성되지 않는다면, 개발자는 터미널을 열고 이 파일들을 사용하여 외부 플래너를 직접 실행해볼 수 있다.

```Bash
ros2 run popf popf /tmp/plansys2/domain.pddl /tmp/plansys2/problem.pddl
```

이 명령을 실행하면 플래너가 출력하는 상세한 오류 메시지나 경고를 직접 확인할 수 있어, 문제의 원인을 파악하는 데 큰 도움이 된다.

더 나아가, Visual Studio Code의 PDDL 플러그인과 같은 외부 개발 도구를 활용하는 것이 매우 효과적이다.1 이러한 도구들은 PDDL 구문 강조(syntax highlighting), 실시간 오류 검사, 그리고 편집기 내에서 직접 플래너를 실행하고 결과를 확인하는 기능 등을 제공하여 개발 및 디버깅 생산성을 크게 향상시킨다.39 그럼에도 불구하고, PDDL 디버깅은 여전히 PlanSys2를 사용하는 데 있어 가장 큰 진입 장벽 중 하나로 남아있다.



PlanSys2의 능력과 복잡성을 가장 잘 보여주는 사례 중 하나는 IROS 2021 학회 논문에서 소개된 '다중 로봇 요리 실험'이다.6 이 실험은 단순히 하나의 목표를 달성하는 것을 넘어, 여러 로봇이 협력하여 주방 환경에서 지속적으로 요리를 만들어내는 동적인 시나리오를 다룬다.

이 시나리오의 PDDL 도메인에는 `move`, `pick_ingredient`, `cook`, `serve`와 같은 행동들이 정의되어 있다. 컨트롤러 노드는 지속적으로 새로운 요리 주문을 생성하고, 이를 달성하기 위한 목표를 Problem Expert에 설정한다. 그러면 Planner는 현재 주방의 상태(로봇 위치, 재료 유무 등)를 바탕으로 요리를 완성하기 위한 계획을 생성한다.

이 사례 연구는 PlanSys2의 여러 핵심 기능을 실제로 어떻게 활용하는지 구체적으로 보여준다.

- **다중 로봇 협응:** 시뮬레이션에서는 최대 3대의 로봇이 동시에 작업을 수행한다.40 Executor는 '액션 오션 프로토콜'을 사용하여 각 행동(예: 

  `(pick_ingredient robot1 tomato)`)을 해당 로봇(`robot1`)에게 정확히 할당한다.

- **외부 시스템 연동:** 로봇의 이동은 ROS2의 표준 내비게이션 스택인 Nav2와 연동된다. `move` PDDL 행동을 구현하는 Action Performer 노드는 내부적으로 Nav2의 액션 서버를 호출하여 실제 이동을 수행한다.40 또한, 실험에 사용된 TIAGo 로봇의 일부 드라이버가 ROS1 기반이었기 때문에, 

  `ros1_bridge`를 사용하여 ROS1과 ROS2 시스템을 연결하는 현실적인 통합 과제도 다루고 있다.

- **지속적인 작업 수행:** 컨트롤러 노드는 하나의 요리가 끝나면 즉시 다음 요리 주문을 생성하여 시스템이 멈추지 않고 계속해서 작업을 수행하도록 한다.41 이는 PlanSys2가 서비스 로봇과 같이 장시간 자율적으로 임무를 수행해야 하는 애플리케이션에 어떻게 적용될 수 있는지를 보여준다.

이 요리 실험은 PlanSys2가 복잡하고 동적인 환경에서 다중 로봇의 작업을 조율하고, 기존 ROS 생태계의 강력한 기능들과 통합될 수 있는 성숙한 프레임워크임을 입증하는 중요한 실증 사례다.


또 다른 중요한 적용 사례는 `plansys2-hospital-l4ros2` 예제 프로젝트에서 찾아볼 수 있다.42 이 프로젝트는 병원과 같은 대규모 실내 환경에서 서비스 로봇이 자율적으로 이동하며 특정 작업을 수행하는 시나리오를 시뮬레이션한다. 이는 물류, 안내, 소독 등 다양한 병원 내 로봇 애플리케이션의 기반이 될 수 있는 기술을 보여준다.43

이 예제에서 TIAGo 로봇은 Gazebo 시뮬레이터로 구현된 병원 환경을 Nav2 스택을 사용하여 항해한다. `plansys2_hospital_launch.py` 런치 파일은 PlanSys2의 핵심 노드들과 함께 병원 환경, 로봇 모델, 그리고 각 PDDL 행동을 구현하는 액션 노드들을 실행시킨다.42

이 사례의 핵심은 `hospital_controller_node`라는 컨트롤러 노드의 역할에 있다.42 이 컨트롤러는 로봇에게 일련의 목표를 순차적으로 부여하는 역할을 한다. 예를 들어, "1. 접수처로 이동하라. 2. 302호로 이동하라. 3. 약제실로 이동하라."와 같은 임무가 주어졌을 때, 컨트롤러는 먼저 `(robot_at tiago reception)`을 목표로 설정하고 PlanSys2에 계획 실행을 요청한다. 이 계획이 성공적으로 완료되면, 다음 목표인 `(robot_at tiago room302)`를 설정하여 다시 계획 실행을 요청하는 방식으로 전체 임무를 관리한다.

이 병원 활용 사례는 PlanSys2가 어떻게 고수준의 작업 계획(Task Planning)을 담당하고, Nav2와 같은 저수준의 동작 계획(Motion Planning) 및 실행 시스템과 계층적으로 결합될 수 있는지를 명확하게 보여준다. 이는 복잡한 서비스 로봇 애플리케이션을 구조적으로 설계하고 개발하는 데 있어 매우 효과적인 패턴이다.


다양한 예제와 실제 적용 사례를 통해 PlanSys2 기반의 애플리케이션을 효과적으로 구조화하기 위한 몇 가지 모범 사례를 도출할 수 있다.

- **전용 패키지 구성:** 특정 PDDL 도메인과 관련된 모든 요소를 하나의 ROS2 패키지에 모아 관리하는 것이 좋다.33 이 패키지에는 일반적으로 다음이 포함된다.
  - `pddl/` 디렉토리: 도메인 PDDL 파일 (`domain.pddl`).
  - `src/` 디렉토리: 각 PDDL 행동을 구현하는 C++ 액션 노드들의 소스 코드.
  - `launch/` 디렉토리: PlanSys2 시스템과 액션 노드들을 함께 실행하는 런치 파일.
  - `config/` 디렉토리: 노드들의 파라미터 설정을 담은 YAML 파일.
- **적절한 행동 구현 방식 선택:** 행동의 복잡도에 따라 구현 방식을 달리하는 것이 효율적이다.
  - 단순한 행동(예: 그리퍼 열기/닫기)이나 다른 ROS2 시스템과의 직접적인 연동이 필요한 행동(예: Nav2 호출)은 C++ `ActionExecutorClient`로 직접 구현하는 것이 간결하다.33
  - 내부적으로 여러 단계를 거치거나 복잡한 로직을 가진 행동(예: 물건 운반)은 `plansys2_bt_actions`를 사용하여 행동 트리(BT)로 구현하는 것이 모듈성과 재사용성 측면에서 유리하다.31
- **구조화된 컨트롤러 설계:** 애플리케이션의 전체 미션을 관장하는 중앙 컨트롤러 노드는 즉흥적으로 코딩하기보다, FSM(Finite State Machine)이나 상위 레벨의 행동 트리(BT)와 같은 검증된 설계 패턴을 사용하여 구조화하는 것이 바람직하다.22 이는 목표 설정, 실행 모니터링, 실패 처리, 재계획 요청 등 복잡한 로직을 명확하게 분리하고 관리하는 데 도움을 준다.

이러한 모범 사례들은 PlanSys2가 제공하는 유연성을 체계적으로 활용하여, 확장 가능하고 유지보수가 용이한 고품질의 로봇 애플리케이션을 구축하는 데 중요한 지침이 된다. PlanSys2는 '무엇을, 어떤 순서로' 할지를 결정하는 상위 수준의 '작업 계획'을 담당하고, Nav2나 MoveIt과 같은 하위 시스템은 '어떻게' 그 행동을 수행할지를 결정하는 '동작 계획'을 담당하는 계층적 분리(Hierarchical Decomposition) 구조는 특히 강력하다. PlanSys2는 이 구조에서 최상위 '의사결정 레이어' 또는 '조율자(Orchestrator)'로서의 역할을 완벽하게 수행하며, 이는 PlanSys2가 단독으로 사용되기보다는 기존의 강력한 ROS2 패키지들과 결합될 때 그 가치가 극대화됨을 시사한다.



PlanSys2의 미래 발전 방향에서 가장 주목할 만한 움직임은 유럽의 AI 계획 이니셔티브인 AIPlan4EU의 일환으로 진행되는 UPF4ROS2 프로젝트와의 통합이다.45 UPF(Unified Planning Framework)는 다양한 종류와 버전의 AI 플래너들을 단일화된 API로 추상화하여 제공하는 것을 목표로 하는 라이브러리다.

현재 PlanSys2는 POPF, TFD 등 소수의 플래너만을 직접 지원하며, 다른 플래너를 사용하려면 개발자가 직접 플러그인을 작성해야 하는 수고가 필요하다. UPF4ROS2는 이 문제를 해결하기 위해 UPF를 PlanSys2의 플래너 플러그인 형태로 통합한다.46 이것이 완성되면, 개발자는 PlanSys2의 설정 파일에서 플래너 이름만 바꾸는 것만으로도 UPF가 지원하는 수많은 최신 플래너(시간 계획, 수치 계획, 계층적 계획, 최적화 계획 등)를 거의 즉시 활용할 수 있게 된다.

이는 PlanSys2의 확장성을 수평적으로 크게 확장하는 중요한 진보다. 더 이상 특정 플래너의 API나 출력 형식을 파싱하는 코드를 작성할 필요 없이, AI 계획 커뮤니티의 최신 연구 성과를 로봇 애플리케이션에 신속하게 적용할 수 있게 되는 것이다. UPF4ROS2는 PlanSys2가 명실상부한 ROS2 생태계의 표준 계획 '플랫폼'으로 자리매김하는 데 결정적인 역할을 할 것으로 기대된다.


PlanSys2의 또 다른 발전 축은 계획 실행의 정확성과 강건성을 수직적으로 심화시키는 학계의 연구 노력에서 찾아볼 수 있다. PlanSys2의 초기 행동 트리(BT) 변환 알고리즘은 많은 시나리오에서 효율적이지만, PDDL 2.1이 정의하는 복잡한 시간적 제약(temporal constraints)의 모든 의미를 완벽하게 포착하지는 못한다는 한계가 지적되었다.49 예를 들어, 한 행동의 효과가 다른 행동이 진행되는 '중간'에 발생해야 하는 등의 미묘한 시간적 관계를 현재의 BT 변환 로직이 놓칠 수 있다는 것이다.

이러한 한계를 극복하기 위해, 최근 연구들은 보다 정교한 알고리즘을 제안하고 있다. 대표적인 접근법 중 하나는 PDDL 계획을 먼저 STN(Simple Temporal Network)으로 변환하는 것이다.12 STN은 노드(시간점)와 간선(시간 제약)으로 구성된 그래프로, 행동들의 시작과 끝 시간점 사이의 모든 시간적 관계를 명시적으로 표현하는 데 매우 효과적이다. 이렇게 생성된 STN을 다시 분석하여 행동 트리로 변환함으로써, 원래 PDDL 계획이 의도했던 복잡한 시간적 의미를 훨씬 더 정확하게 반영하는 실행 트리를 만들 수 있다.

이러한 연구들은 PlanSys2의 Executor가 단순한 병렬 실행을 넘어, 엄격한 시간적 요구사항이 있는 미션 크리티컬한 애플리케이션(예: 정밀한 협동 조립 작업)에서도 신뢰성 있게 동작할 수 있도록 만드는 것을 목표로 한다. 이는 프레임워크가 기능적으로 '성숙'해감에 따라, '기능의 확장'과 함께 '기능의 심화'가 동시에 이루어지고 있음을 보여주는 좋은 예다.


PlanSys2는 ROS2 Humble 및 이후 버전을 위한 기호 계획 프레임워크로서, 의심할 여지 없이 가장 성숙하고 강력한 솔루션으로 자리매김했다. ROS1 시대의 ROSPlan에서 영감을 받았지만, ROS2의 핵심 철학(Lifecycle Node, DDS)을 기반으로 완전히 재설계됨으로써 비교할 수 없는 수준의 신뢰성, 예측 가능성, 그리고 확장성을 확보했다.3

**강점:**

- **강건한 모듈식 아키텍처:** 4개의 노드(Domain Expert, Problem Expert, Planner, Executor)로 명확히 분리된 책임과 Lifecycle Node 기반의 결정론적 상태 관리는 복잡한 시스템의 개발과 유지보수를 용이하게 한다.
- **최적화된 실행 모델:** 선형적인 PDDL 계획을 병렬 실행이 가능한 행동 트리(BT)로 변환하는 혁신적인 접근법은 계획 실행 시간을 크게 단축시키고 시스템의 반응성을 높인다.
- **뛰어난 확장성:** 플러그인 기반의 Planner와 액션 오션 프로토콜은 다중 로봇 시나리오와 새로운 AI 계획 알고리즘의 통합을 유연하게 지원한다.

**약점:**

- **제한적인 PDDL 지원 및 디버깅의 어려움:** 최신 PDDL 표준의 모든 기능을 지원하지 않으며, PDDL 모델링 및 디버깅은 여전히 높은 전문성을 요구하는 진입 장벽이다.
- **상태 일관성 문제:** 행동 실행이 중간에 실패했을 때, 시스템의 상태를 자동으로 복구해주지 않아 컨트롤러 수준에서 복잡한 예외 처리 로직이 요구된다.

이러한 몇 가지 한계점에도 불구하고, PlanSys2의 근본적인 아키텍처의 우수성, 활발한 개발 커뮤니티의 지속적인 개선 노력 35, 그리고 UPF4ROS2와 같은 미래 발전 가능성을 고려할 때, PlanSys2는 현재 ROS2 생태계에서 복잡한 자율 로봇의 작업 계획(task planning)을 위한 사실상의 표준(de facto standard) 프레임워크라 결론 내릴 수 있다.12 PlanSys2는 로봇에게 "무엇을 할지" 지능적으로 결정하는 능력을 부여하고자 하는 모든 ROS2 개발자에게 가장 먼저 고려해야 할 강력하고 필수적인 도구다.


1. Additional Resources - Planning.wiki, accessed July 23, 2025, https://planning.wiki/extras
2. Optimized Execution of PDDL Plans using Behavior Trees - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2101.01964
3. PlanSys2: A Planning System Framework for ROS2 - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2107.00376
4. ros2_planning_system - ROS Index, accessed July 23, 2025, https://index.ros.org/r/ros2_planning_system/
5. ROS2 Planning System - Projects - ROS Discourse, accessed July 23, 2025, https://discourse.ros.org/t/ros2-planning-system/11844
6. PlanSys2: A Planning System Framework for ROS2 - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/353055585_PlanSys2_A_Planning_System_Framework_for_ROS2
7. An Introduction to PDDL, accessed July 23, 2025, http://www.cs.toronto.edu/~sheila/2542/w09/A1/introtopddl2.pdf
8. Mastering PDDL in Robotics - Number Analytics, accessed July 23, 2025, https://www.numberanalytics.com/blog/mastering-pddl-in-robotics
9. ROS2 Planning System 1st Anniversary - ROS Projects - Open Robotics Discourse, accessed July 23, 2025, https://discourse.ros.org/t/ros2-planning-system-1st-anniversary/18224
10. Fast and Accurate Task Planning using Neuro-Symbolic Language Models and Multi-level Goal Decomposition - arXiv, accessed July 23, 2025, https://arxiv.org/html/2409.19250v1
11. ROS2 Planning System 1st Anniversary - Projects - Open Robotics Discourse, accessed July 23, 2025, https://discourse.openrobotics.org/t/ros2-planning-system-1st-anniversary/18224
12. PlanSys2: A Planning System Framework for ROS2 | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/357109831_PlanSys2_A_Planning_System_Framework_for_ROS2
13. PlanSys2 design - ROS2 Planning System 2 1.0.0 documentation, accessed July 23, 2025, https://plansys2.github.io/design/index.html
14. PlanSys2 architecture. | Download Scientific Diagram - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/figure/PlanSys2-architecture_fig1_353055585
15. plansys2_problem_expert 2.0.9 documentation - ROS 2, accessed July 23, 2025, https://docs.ros.org/en/humble/p/plansys2_problem_expert/
16. ROS Package: plansys2_problem_expert, accessed July 23, 2025, https://index.ros.org/p/plansys2_problem_expert/
17. plansys2_planner 2.0.18 documentation - the official ROS docs, accessed July 23, 2025, https://docs.ros.org/en/jazzy/p/plansys2_planner/
18. plansys2_planner - plansys2_planner 2.0.9 documentation, accessed July 23, 2025, https://docs.ros.org/en/humble/p/plansys2_planner/
19. plansys2_executor 2.0.18 documentation - the official ROS docs, accessed July 23, 2025, https://docs.ros.org/en/jazzy/p/plansys2_executor/
20. Nuriadj/plansys2_lpgtd_plugin - GitHub, accessed July 23, 2025, https://github.com/Nuriadj/plansys2_lpgtd_plugin
21. plansys2_msgs 2.0.9 documentation - the official ROS docs, accessed July 23, 2025, https://docs.ros.org/en/ros2_packages/humble/api/plansys2_msgs/
22. Using a planning controller - ROS2 Planning System 2 1.0.0 documentation, accessed July 23, 2025, https://plansys2.github.io/tutorials/docs/controller_example.html
23. ros2_planning_system/plansys2_docs/FAQ.md at rolling - GitHub, accessed July 23, 2025, https://github.com/IntelligentRoboticsLabs/ros2_planning_system/blob/master/plansys2_docs/FAQ.md
24. Terminal Usage - ROS2 Planning System 2 1.0.0 documentation, accessed July 23, 2025, https://plansys2.github.io/tutorials/docs/terminal_usage.html
25. What are the best planners for PDDL2.1? - Stack Overflow, accessed July 23, 2025, https://stackoverflow.com/questions/71410330/what-are-the-best-planners-for-pddl2-1
26. Issues runing TFD with plansys2 #7 - GitHub, accessed July 23, 2025, https://github.com/IntelligentRoboticsLabs/plansys2_tfd_plan_solver/issues/7
27. ROS Package: plansys2_planner, accessed July 23, 2025, https://index.ros.org/p/plansys2_planner/
28. Writing a New Planner Plugin - Nav2 1.0.0 documentation, accessed July 23, 2025, https://docs.nav2.org/plugin_tutorials/docs/writing_new_nav2planner_plugin.html
29. Optimized Execution of PDDL Plans using Behavior Trees, accessed July 23, 2025, https://archive.illc.uva.nl/AAMAS-2021/pdfs/p1596.pdf
30. Developing BDI-based Robotic Systems with ROS2 - iris@unitn, accessed July 23, 2025, https://iris.unitn.it/retrieve/handle/11572/364845/604413/PAAMS_22_ROS2BDI.pdf
31. Implement actions as Behavior Trees - ROS2 Planning System, accessed July 23, 2025, https://plansys2.github.io/tutorials/docs/bt_actions.html
32. ROS Package: plansys2_executor - ROS Index, accessed July 23, 2025, https://index.ros.org/p/plansys2_executor/
33. The first planning package - ROS2 Planning System 2 1.0.0 documentation, accessed July 23, 2025, https://plansys2.github.io/tutorials/docs/simple_example.html
34. ROS Package: plansys2_bt_actions, accessed July 23, 2025, https://index.ros.org/p/plansys2_bt_actions/
35. Issues / PlanSys2/ros2_planning_system - GitHub, accessed July 23, 2025, https://github.com/PlanSys2/ros2_planning_system/issues
36. slow publishing and performance for custom messages with large arrays / Issue #346 / ros2/rmw_cyclonedds - GitHub, accessed July 23, 2025, https://github.com/ros2/rmw_cyclonedds/issues/346
37. Regarding slow output rate in ROS2 / autowarefoundation / Discussion #2555 - GitHub, accessed July 23, 2025, https://github.com/orgs/autowarefoundation/discussions/2555
38. Infinite loop when checking requirement satisfaction fails in wait_atstart / Issue #218 / PlanSys2/ros2_planning_system - GitHub, accessed July 23, 2025, https://github.com/PlanSys2/ros2_planning_system/issues/218
39. Episode 6: Multiple Planner Configuration - PDDL Tooling - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=mH1J56abKxE
40. IntelligentRoboticsLabs/plansys2_cooking_experiment - GitHub, accessed July 23, 2025, https://github.com/IntelligentRoboticsLabs/plansys2_cooking_experiment
41. Cooking Experiment - PlanSys2 - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=2yCuZLhCzKk
42. Docencia-fmrico/plansys2-hospital-l4ros2 - GitHub, accessed July 23, 2025, https://github.com/Docencia-fmrico/plansys2-hospital-l4ros2
43. Behavior Trees in Robotics Online Seminars | Petter Ögren - KTH, accessed July 23, 2025, https://www.kth.se/profile/petter/page/behavior-trees-in-robotics-online-seminars
44. Runtime Architecture and Task Plan Co-Adaptation for Autonomous Robots with Metaplan, accessed July 23, 2025, https://arxiv.org/html/2312.10525v1
45. REAP: A Flexible Real-World Simulation Framework for AI-Planning of UAVs - ICAPS 2023, accessed July 23, 2025, https://icaps23.icaps-conference.org/demos/papers/3216_paper.pdf
46. PlanSys2/UPF4ROS2 - GitHub, accessed July 23, 2025, https://github.com/PlanSys2/UPF4ROS2
47. UPF4ROS2 - robotica.unileon.es, accessed July 23, 2025, https://robotica.unileon.es/index.php?title=UPF4ROS2
48. UPF4ROS2 | AI-on-Demand - AI4Europe, accessed July 23, 2025, https://www.ai4europe.eu/business-and-industry/case-studies/upf4ros2
49. Constructing Behavior Trees from Temporal Plans for Robotic Applications - arXiv, accessed July 23, 2025, https://arxiv.org/html/2406.17379v1
50. ROS2 Planning System, accessed July 23, 2025, https://plansys2.github.io/


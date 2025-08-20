# PlanSys2 BehaviorTree.ROS2 SMACC2 통합 설계

## 1. 계층적 자율성의 필요성

현대 로봇 공학은 동적이고 비정형적인 환경에서 작동하는 자율 로봇을 요구하며, 이는 제어 아키텍처에 중대한 도전을 제기한다. 과거의 단일체(monolithic) 시스템이나 순수 반응형 시스템은 복잡성이 증가함에 따라 확장성과 유지보수성 측면에서 한계를 드러냈다. 특히 유한 상태 기계(FSM)는 상태 전이 로직이 각 개별 상태에 분산되어 있어, 복잡한 AI 에이전트 개발 시 확장, 적응, 재사용이 어렵다는 문제가 제기되었다.1 이러한 문제를 해결하기 위해, 로봇의 의사결정 과정을 여러 추상화 계층으로 분리하는 계층적 제어 아키텍처가 필수적인 해결책으로 부상했다.

본 보고서는 ROS2 환경에서 세 가지 강력한 프레임워크를 통합하는 3계층 제어 아키텍처를 심층적으로 분석하고, 그 구체적인 구현 방안을 제시한다. 이 아키텍처는 다음과 같이 구성된다.

- **전략 계층 (Strategy - Why):** 임무 수준의 목표를 분해하고 장기적인 계획을 수립한다. 이 역할은 PDDL(Planning Domain Definition Language) 기반의 기호적 계획(symbolic planning)을 제공하는 **PlanSys2**가 담당한다.3
- **전술 계층 (Tactics - How):** 전략적 단계 하나를 실행하기 위해 반응적인 작업 순서를 정의한다. 이 역할은 모듈성, 조합성, 반응성을 갖춘 실행 로직을 제공하는 **BehaviorTree.ROS2**가 담당한다.1
- **반응 계층 (Reaction - What):** 개별 하드웨어 구성요소의 저수준 실시간 제어를 담당한다. 이 역할은 복잡한 다중 컴포넌트 로봇을 위해 설계된 비동기, 이벤트 기반 FSM 라이브러리인 **SMACC2**가 담당한다.8

현재 ROS2 생태계에서 PlanSys2, BehaviorTree.ROS2, SMACC2를 즉시 통합해주는 단일 프레임워크는 존재하지 않는다.10 그러나 본 보고서는 이 세 가지 구성요소를 하나의 지능형 시스템으로 통합하기 위해 필요한 제어 흐름, 통신 프로토콜, 그리고 핵심 설계 패턴을 상세히 기술함으로써, 이러한 통합을 위한 명확하고 실행 가능한 청사진을 제공하고자 한다.

## 2.  아키텍처 청사진: 3계층 심층 분석

본 섹션에서는 각 계층을 구성하는 프레임워크를 심층적으로 분석하고, 전체 아키텍처 내에서 각각의 역할과 핵심 기능을 해부한다.

### 2.1  전략 계층: 임무 수준 심의를 위한 PlanSys2

PlanSys2의 핵심 기능은 "현재 세계의 상태와 주어진 목표가 있을 때, 어떤 행동 순서를 수행해야 하는가?"라는 질문에 답하는 것이다. 이를 위해 PDDL을 사용하여 세계 모델과 로봇의 행동을 정의한다.4 PlanSys2는 네 가지 핵심 노드로 구성된 모듈식 설계를 채택하고 있다.5

- **Domain Expert:** 타입, 술어(predicate), 액션 스키마 등 정적인 PDDL 모델 정보를 관리한다. 여러 도메인 파일을 결합하여 모듈식 애플리케이션 설계를 지원하는 강력한 기능을 제공한다.5
- **Problem Expert:** 객체 인스턴스, 현재 술어 값, 활성 목표 등 세계의 동적인 상태를 유지 관리한다. 이는 실행 중에 지속적으로 업데이트되는 시스템의 "지식 베이스(knowledge base)" 역할을 한다.11
- **Planner:** 계획 수립의 핵심 두뇌 역할을 한다. Domain Expert와 Problem Expert로부터 도메인과 문제 정의를 받아 POPF나 TFD와 같은 외부 PDDL 솔버 플러그인을 호출하고, 그 결과로 생성된 계획 파일(`plan.pddl`)을 파싱한다.13
- **Executor:** 계획을 실행하는 주체이다. Planner로부터 계획을 받아 액션 수행자(action performer)를 호출하고 그 진행 상황을 모니터링하며 실행을 총괄한다.11

PlanSys2의 설계에서 주목할 점은 Executor가 단순히 선형적인 액션 시퀀스를 실행하지 않는다는 것이다. 대신, PDDL 계획을 행동 트리(Behavior Tree, BT)로 변환하여 실행을 최적화한다. 이 변환 과정은 액션 간의 의존성을 분석하여 병렬 실행이 가능한 흐름을 식별하고, 이를 통해 전체 임무 수행 시간을 단축한다.5 이는 BT가 단순히 하위 전술 계층의 구성요소일 뿐만 아니라, 이미 전략 계층의 실행 모델 내에 깊숙이 통합된 핵심 개념임을 시사한다.

### 2.2  전술 계층: 반응형 작업 순서를 위한 BehaviorTree.ROS2

전술 계층의 핵심 기능은 `transport`와 같은 PlanSys2의 단일 PDDL 액션을 `Navigate`, `Grasp`, `Navigate`, `Release`와 같은 더 상세하고 반응적인 하위 작업들의 순서로 분해하여 실행하는 것이다. BT는 FSM이 확장성과 모듈성 측면에서 겪는 어려움을 효과적으로 해결한다.1

BT의 실행 원리는 다음과 같다.

- **틱 (Tick):** 루트 노드에서 시작하여 트리 아래로 전파되는 기본적인 실행 신호로, 특정 주기로 발생한다.17
- **노드 상태 (Node Status):** 모든 노드는 실행 후 `SUCCESS`, `FAILURE`, 또는 `RUNNING` 상태 중 하나를 반환하며, 이 상태 값은 제어 흐름을 결정한다.17
- **블랙보드 (Blackboard):** 트리 내의 노드 간에 데이터를 공유하기 위한 키-값 저장소로, 파라미터와 결과 전달에 필수적이다.19
- **제어 흐름 노드 (Control Flow Nodes):** `Sequence` (논리적 AND), `Fallback` (논리적 OR), `Parallel`, `Decorator`와 같은 노드들이 트리의 논리 구조를 형성한다.17

여기서 `BehaviorTree.ROS2` 패키지는 `BehaviorTree.CPP`라는 핵심 라이브러리 7를 ROS2와 통합하는 결정적인 접착제 역할을 한다.6 이 패키지는 ROS2의 기본 통신 메커니즘과 상호작용하기 위한 사전 구축된 `TreeNode` 래퍼들을 제공한다.

- `RosActionNode`: ROS2 액션 호출용
- `RosServiceNode`: ROS2 서비스 호출용

이러한 래퍼들은 BT가 SMACC2 FSM을 포함한 다른 ROS2 노드에 명령을 내릴 수 있게 하는 필수적인 도구이다. ROS2의 내비게이션 스택인 Nav2는 이러한 BT의 활용을 보여주는 성숙하고 대표적인 사례이다.19

### 2.3  반응 계층: 비동기 장치 제어를 위한 SMACC2

반응 계층의 핵심 기능은 그리퍼, 매니퓰레이터 암, 센서 모듈 등 단일의 복잡한 하드웨어 구성요소의 상태와 동작을 관리하는 것이다. SMACC2는 BT나 플래너에서 구현하기에는 번거롭고 불안정한 저수준의 이벤트 기반 로직을 처리하는 데 특화되어 있다.

SMACC2 애플리케이션의 구조는 다음과 같다.8

- **상태 (`SmaccState`):** 행동의 기본 단위로, 전이 테이블과 로직을 포함한다.
- **이벤트 (Events):** 상태 전이의 주요 동인이다. 이벤트는 상태 머신에 의해 게시(post)되고 소비(consume)된다.25
- **클라이언트 (`smacc2_client_library`):** Nav2나 MoveIt2와 같은 외부 ROS2 노드와의 통신을 관리하는 영속적인 객체이다.8 이들은 외부 이벤트의 주요 소스이다.
- **오쏘고널 (Orthogonals):** 복잡성 관리를 위한 핵심 개념이다. 오쏘고널은 `OrthoGripper`, `OrthoDriveBase`와 같이 명확히 구분되는 기능 또는 하드웨어 구성요소를 나타낸다. 이는 클라이언트를 포함하며 주 상태 머신 로직과 병렬로 실행되어, 구성요소별 로직을 완벽하게 캡슐화한다.8
- **컴파일 타임 검증:** `Boost.Statechart`로부터 물려받은 주요 특징으로, 상태 머신의 구조와 전이가 컴파일 시점에 검증되어 런타임 오류를 획기적으로 줄여준다.8

이 세 프레임워크는 추상적인 "가정(what-if)" 기반의 계획(HTN)에서 반응적인 작업 실행(BT), 그리고 비동기적인 하드웨어 관리(FSM)에 이르기까지 로봇 공학 문제의 자연스러운 분해 과정을 그대로 반영한다. 이는 임의의 도구 조합이 아니라, 문제 해결을 위한 잘 정립된 아키텍처 패턴임을 보여준다. 하지만 SMACC2의 외부 상호작용 패턴에 대한 문서가 부족하다는 점 24은, 성공적인 통합을 위해 기성 예제에 의존하기보다는 명확하고 재사용 가능한 맞춤형 인터페이스를 설계해야 함을 시사한다.

## 3.  계층 간 제어 흐름 및 통신 프로토콜

본 섹션에서는 계층 구조를 따라 하향식으로 전달되는 명령 흐름과, 상향식으로 전달되는 피드백 및 세계 상태 업데이트 흐름을 추적하여 시스템의 작동 방식을 구체적으로 설명한다.

### 3.1  하향식 명령 전파: 계획에서 행동까지

#### 3.1.1 HTN -->> BT: `bt_action_node` 브리지

명령 흐름은 PlanSys2 Executor가 계획에서 `(transport r2d2 kitchen assembly_station)`과 같은 액션을 식별하면서 시작된다.

1. Executor는 `plansys2_msgs::action::ExecuteAction`이라는 ROS2 액션을 통해 해당 액션 수행자(action performer)를 호출한다.16
2. BT 기반 액션의 경우, 이 수행자는 `plansys2_bt_actions` 패키지에서 제공하는 범용 노드인 `bt_action_node`이다.29 이 노드는 자신이 처리할 PDDL 액션의 이름(`transport`)과 해당 BT의 XML 파일 경로(`transport.xml`)를 파라미터로 받아 실행된다.29
3. `ExecuteAction` 목표를 수신하면, `bt_action_node`는 `transport.xml` 트리를 동적으로 로드하고, PDDL 인자들(예: `arg0: "r2d2"`, `arg1: "kitchen"`, `arg2: "assembly_station"`)을 블랙보드에 채운 뒤, 트리를 틱(tick)하기 시작한다.29

#### 3.1.2 BT -->> FSM: `RosActionNode`와 SMACC2 클라이언트 패턴

`transport.xml` 트리 내에서 `<GraspPart/>`와 같은 리프 노드는 그리퍼의 FSM을 트리거해야 한다.

1. 이 리프 노드는 맞춤형 `BT::RosActionNode<...>`로 구현된다.6 이 노드의 목적은 `/gripper_control/grasp`와 같은 전용 ROS2 액션 서버를 호출하는 것이다.
2. `/gripper_control/grasp` 액션 서버는 `OrthoGripper`라는 오쏘고널 내의 SMACC2 `Client` 안에서 구현된다. 이 클라이언트의 생성자는 `rclcpp_action::Server`를 생성한다.
3. BT 노드가 틱되면 액션 목표를 전송한다. SMACC2 클라이언트의 목표 콜백 함수가 이 목표를 수신하고, `EvGraspRequest`와 같은 내부 SMACC2 이벤트를 상태 머신에 게시(post)한다.24 이 이벤트가 그리핑 상태 시퀀스로의 전이를 유발한다.
4. FSM이 작업을 수행하는 동안 BT 노드는 `RUNNING` 상태를 유지한다.

### 3.2  상향식 피드백 및 세계 상태 동기화

#### 3.2.1 FSM -->> BT: ROS 액션을 통한 결과 보고

SMACC2 FSM이 작업을 성공 또는 실패로 완료하면, 액션 서버를 관리하는 `Client`가 액션 핸들의 결과를 설정한다 (예: `handle->succeed()` 또는 `handle->abort()`).

1. 이 결과는 `BT::RosActionNode`의 `onResultReceived` 또는 `onFailure` 콜백 함수에서 수신된다.6
2. BT 노드는 이 액션 결과를 BT 상태인 `NodeStatus::SUCCESS` 또는 `NodeStatus::FAILURE`로 변환한다. 이 상태는 BT 상위로 전파되어 전술적 실행 흐름에 영향을 미친다.

#### 3.2.2 FSM/BT -->> HTN: 재계획을 위한 피드백 루프

이 경로는 지능적 자율성을 위한 가장 중요한 피드백 경로이다. 하위 계층에서의 실패는 반드시 전략 계층에 전달되어야 한다.

1. **액션 실패:** BT로 구현된 PDDL 액션이 PlanSys2 Executor에게 `FAILURE`를 반환하면, Executor는 이를 인지하고 기본적으로 전체 계획 실행을 중단시킨다.16 `plansys2::ExecutorClient`를 기반으로 구축된 상위 제어 노드는 이 실패를 감지하고 새로운 계획을 요청할 수 있다.
2. **세계 상태 업데이트:** 더 견고한 방법은 `ProblemExpert`를 직접 업데이트하는 것이다. 예를 들어, 그리퍼 FSM이 객체 파지에 실패하고 이를 BT에 보고하면, "World State Monitor"와 같은 전용 노드가 `ProblemExpert`의 서비스를 호출하여 `(part_at part1 locationA)` 술어를 `(grasp_failed part1)`으로 변경하거나 `(gripper_free r2d2)`를 제거할 수 있다.
3. Executor는 다음 액션을 실행하기 전에, 해당 액션의 전제 조건(`at_start` 요구사항)을 업데이트된 `ProblemExpert`와 비교하여 확인한다.16 만약 실패로 인해 전제 조건이 더 이상 충족되지 않으면, 현재 계획은 무효화되고 취소된다. 그러면 제어 애플리케이션은 새로운 세계 상태를 기반으로 재계획을 요청하여 동적인 대응을 할 수 있게 된다.5

### 3.3 표 2.1: 계층 간 통신 매트릭스

아래 표는 전체 통신 아키텍처를 간결하게 요약한 것이다. 이는 설계에 대한 빠른 이해와 구현 시 참조 자료로서 매우 유용하다.

| 소스 계층                  | 타겟 계층                  | 주요 ROS2 메커니즘 | 핵심 패키지/노드/인터페이스                                  | 목적                                                         |
| -------------------------- | -------------------------- | ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **HTN (PlanSys2)**         | **BT (BehaviorTree.ROS2)** | ROS2 액션          | `plansys2_executor` -> `plansys2_bt_actions::bt_action_node` (`plansys2_msgs::action::ExecuteAction` 사용) | BT로 구현된 단일 PDDL 액션의 실행을 명령                     |
| **BT (BehaviorTree.ROS2)** | **FSM (SMACC2)**           | ROS2 액션          | 맞춤형 `BT::RosActionNode` -> `rclcpp_action::Server`를 갖춘 맞춤형 SMACC2 `Client` | 하드웨어 구성요소로부터 특정 상태 관리 기능(예: 파지, 보관, 충전)을 요청 |
| **FSM (SMACC2)**           | **BT (BehaviorTree.ROS2)** | ROS2 액션 결과     | SMACC2 `Client` -> `BT::RosActionNode`의 `onResultReceived`/`onFailure` 콜백 | 요청된 기능의 최종 상태(성공/실패)를 전술 계층으로 보고      |
| **FSM/BT**                 | **HTN (PlanSys2)**         | ROS2 서비스/토픽   | 맞춤형 "World State Monitor" 노드 -> `plansys2_problem_expert` 서비스 (예: `add_predicate`, `remove_predicate`) | 저수준 결과를 PDDL 세계 모델에 업데이트하여 전제 조건 확인 및 재계획을 유발 |
| **애플리케이션**           | **HTN (PlanSys2)**         | ROS2 액션/서비스   | 맞춤형 제어 노드 -> `plansys2_executor` (`ExecutePlan` 액션) 또는 `plansys2_planner` (`GetPlan` 서비스) | 목표를 설정하고 계획을 요청하여 전체 전략 프로세스를 시작    |

## 4.  제어 프리미티브의 생명주기 관리

본 섹션에서는 사용자의 질문에 직접적으로 답하며, 각 계층의 제어 구성요소들이 영속적인지 아니면 일시적인지에 대해 설명한다.

### 4.1  HTN 계획의 생명주기: 일시적인 아티팩트

PlanSys2의 계획은 본질적으로 일시적인(ephemeral) 객체이다. 클라이언트가 `GetPlan` 서비스를 호출하거나 `ExecutePlan` 액션을 요청할 때 필요에 따라 생성된다.14 계획 자체는 임시 파일(`plan.pddl`)에 저장된 후 `plansys2_msgs::msg::Plan` 메시지 구조로 파싱되는 액션의 시퀀스이다.13 Executor가 계획을 소모하면(완료, 실패, 또는 취소), 해당 계획 객체는 폐기된다. 다음 목표를 위해서는 반드시 새로운 계획이 생성되어야 한다. 즉, 계획의 생명주기는 단일의 상위 임무 목표에 종속된다.

### 4.2  행동 트리의 생명주기: 범위가 지정된 일시적 객체

이 아키텍처의 맥락에서 BT의 생명주기는 단일 PDDL 액션의 실행 범위로 한정된다. `bt_action_node`는 `ExecuteAction` 액션 서버에서 목표를 수신할 때마다 XML 파일로부터 BT의 새로운 인스턴스를 생성한다.29 BT가 종료 상태(`SUCCESS` 또는 `FAILURE`)를 반환하면, `bt_action_node`는 그 결과를 반환하고 BT 객체는 소멸된다. 이는 각 PDDL 액션 실행이 깨끗하고 새로운 전술적 상태에서 시작됨을 보장한다.

### 4.3  FSM의 생명주기: 영속적인 서비스

이와는 대조적으로, SMACC2 상태 머신은 장시간 실행되는 영속적인(persistent) ROS2 노드로 설계되었다.8 그리퍼와 같은 특정 구성요소를 위한 상태 머신은 로봇 작동 시간 동안 단 한 번 실행된다. 각 요청에 따라 생성되고 소멸되는 것이 아니라, 수신되는 ROS 메시지(액션, 서비스, 토픽)에 의해 트리거된 이벤트에 반응하여 `Idle`, `Grasping`, `Holding`, `Error` 등 내부적으로 정의된 상태들 사이를 전이한다.24

이러한 생명주기의 차이는 근본적인 아키텍처 원칙을 반영한다. 전략 및 전술 계층은 동적이고 임무 지향적이므로 그 결과물은 일시적이다. 반면, 반응 계층은 로봇의 물리적 능력을 나타내며 이는 영속적이므로, 그 소프트웨어 추상화 역시 영속적이다. 이러한 분리는 모든 하위 작업마다 하드웨어 인터페이스를 재초기화하는 오버헤드를 방지한다.

## 5.  실용적인 통합 가이드 및 통합 프레임워크

본 섹션은 "통합된 프레임워크가 있는가?"라는 질문에 "없지만, 이렇게 만들 수 있다"고 답하며, 이전의 모든 분석을 실행 가능한 구현 계획으로 종합한다.

### 5.1  통합의 간극과 맞춤형 솔루션의 필요성

앞서 언급했듯이, 이 세 가지 라이브러리를 아우르는 단일 프레임워크는 존재하지 않는다.10 따라서 다음 내용은 이 강력하지만 분리된 라이브러리들 사이의 간극을 메우기 위한 제안된 아키텍처이다.

### 5.2  통합 프레임워크를 위한 청사진

#### 5.2.1 BT-FSM 액션 인터페이스

대부분의 BT-FSM 상호작용에 재사용할 수 있는 범용 ROS2 액션 파일(예: `ComponentControl.action`)을 정의하여 통신 계약을 표준화한다.

```
# ComponentControl.action
string command  # 예: "GRASP", "STOW", "SELF_CHECK"
string params # 명령어별 파라미터
---
bool success
string message
---
float32 progress
```

#### 5.2.2 재사용 가능한 SMACC2 액션 클라이언트

오쏘고널 내에서 인스턴스화될 `SmaccActionServerClient`를 위한 C++ 템플릿/예제를 제공한다. 이 클라이언트는 `rclcpp_action::Server`를 포함하며, 콜백 함수는 수신된 `command` 문자열을 파싱하여 적절한 타입의 SMACC2 이벤트(예: `EvGraspRequest`, `EvStowRequest`)를 게시하는 역할을 담당한다. 이것이 BT와 FSM 계층을 연결하는 핵심 맞춤형 코드이다.

#### 5.2.3 세계 상태 모니터 및 업데이트 노드

중앙 집중식 Python 또는 C++ 노드를 설계한다. 이 노드는 BT 및 FSM 계층에서 발행하는 상태 토픽(예: `/gripper/status`, `/navigation/status`)을 구독한다. 또한, BT 노드가 중요한 결과를 보고할 수 있도록 서비스를 제공한다. 파지 실패와 같은 중요한 이벤트가 발생하면, 이 노드는 `plansys2_problem_expert` 서비스를 호출하여 PDDL 세계 모델을 업데이트함으로써 재계획 루프를 완성시킨다.

### 5.3  사례 연구: "자율적 인출 및 전달"

아래는 전체 아키텍처가 실제로 작동하는 과정을 단계별로 서술한 것이다.

1. **목표 설정:** 애플리케이션 노드가 `ProblemExpert`에 `(and (part_delivered part1 assembly_station))`라는 목표를 설정한다.

2. **HTN 계획:** 애플리케이션이 계획을 요청하면, PlanSys2는 다음과 같은 계획을 생성한다.

   Code snippet

   ```
   0.001: (move r2d2 depot kitchen)
   10.000: (pickup r2d2 part1 kitchen)
   20.000: (move r2d2 kitchen assembly_station)
   30.000: (drop r2d2 part1 assembly_station)
   ```

3. **전술적 실행 (BT):** Executor는 `(move...)` 액션부터 시작한다. `move` `bt_action_node`를 호출하고, 이 BT는 Nav2 액션을 래핑하여 실행한다.19 성공하면 Executor는 

   `(pickup...)`으로 넘어간다.

4. **반응적 실행 (FSM):** `pickup` `bt_action_node`가 트리거된다. 이 노드의 BT는 `<GraspObject>` 노드를 포함하는 시퀀스로 구성된다. 이 노드는 `RosActionNode`로서, SMACC2 FSM 내에서 실행 중인 `/gripper_control/grasp` 액션 서버를 호출한다.

5. **FSM 내부 로직:** SMACC2 FSM은 요청을 받고 `StOpeningGripper -> StApproachingPart -> StClosingGripper -> StConfirmingGrasp -> StLifting`과 같은 내부 상태들을 순차적으로 전이한다. 각 상태는 센서 클라이언트로부터 오는 내부 이벤트(예: `EvProximity`, `EvContact`)에 의해 구동된다.

6. **상향식 피드백:** FSM이 성공하고 `Client`가 액션 결과로 `success=true`를 보고한다. `<GraspObject>` BT 노드는 이 결과를 받고 `SUCCESS`를 반환한다. `pickup` BT가 성공적으로 완료되고, Executor는 다음 계획 단계로 이동한다.

7. **실패 시나리오 및 재계획:** `StConfirmingGrasp` 상태가 실패했다고 가정하자(예: 힘 센서에 접촉 없음). FSM은 에러 상태로 전이하고, `Client`는 액션 결과로 `success=false`를 보고한다. `<GraspObject>` BT 노드는 `FAILURE`를 반환하고, `pickup` BT 전체가 실패한다. Executor는 계획을 중단한다. 이때 World State Monitor 노드가 그리퍼 상태가 "파지 실패"로 변경된 것을 감지하고 `ProblemExpert`에 `(grasp_failed part1)`을 업데이트한다. 계획 실패를 인지한 애플리케이션 노드는 새로운 계획을 요청한다. 이제 플래너는 `(grasp_failed part1)`이라는 새로운 사실을 알고 있으므로, 다른 계획(예: 다른 부품을 시도하거나 도움 요청)을 생성하게 된다.

## 6.  고급 아키텍처 고려사항 및 권장사항

### 6.1  시스템 전반의 결함 허용 및 복구

- **FSM 계층:** SMACC2 내부에 `StRetryGrasp`와 같은 자체 복구 상태를 구현하여 내부적인 오류를 처리한다.
- **BT 계층:** `Fallback` 노드를 사용하여 대안적인 전술(예: 한 파지 자세가 실패하면 다른 자세 시도)을 정의하고, `Retry` 데코레이터를 사용하여 일시적인 실패에 대응한다.
- **HTN 계층:** 전술적으로 복구 불가능한 실패는 이 계층에서 처리한다. 주요 복구 메커니즘은 `ProblemExpert`를 업데이트하는 피드백 루프에 의해 트리거되는 재계획(replanning)이다.31

### 6.2  동시성 및 병렬성 관리

PlanSys2의 Executor는 계획 그래프에서 상호 의존성이 없는 액션들에 대해 `Parallel` 노드를 포함하는 BT를 생성할 수 있다.5 아키텍처는 이를 지원해야 한다. 예를 들어, 계획에 `(move_arm_to_stow)`와 `(navigate_to_chargestation)`이 병렬로 포함될 수 있다. 이는 팔 제어 FSM과 내비게이션 FSM이 동시에 작동할 수 있어야 함을 의미하며, 이는 SMACC2의 오쏘고널 모델이 정확히 해결하고자 하는 문제이다.8

### 6.3  확장성 및 유지보수성

이 계층적 아키텍처는 확장성이 매우 뛰어나다. 예를 들어, 드릴과 같은 새로운 기능을 추가하는 과정은 다음과 같이 모듈화된다.

1. 드릴을 위한 SMACC2 FSM을 포함하는 `sm_drill` 패키지를 생성한다.
2. 도메인 파일에 `(drill_hole)` PDDL 액션을 정의한다.
3. ROS 액션을 통해 드릴의 FSM을 호출하는 `drill_hole.xml` BT를 생성한다.
4. 새로운 노드들을 실행한다.

이 과정에서 핵심 전략 및 전술 로직은 변경되지 않는다. 이러한 모듈성은 이 아키텍처의 핵심 강점이다.2

## 7. 결론: 견고하고 지능적인 제어 시스템의 합성

본 보고서는 ROS2 환경에서 PlanSys2(HTN), BehaviorTree.ROS2(BT), SMACC2(FSM)를 통합하는 계층적 제어 아키텍처를 심층적으로 분석하고, 그 구체적인 구현 방안을 제시했다.

분석 결과, 이 아키텍처는 다음과 같은 핵심 원칙에 기반한다. 첫째, 3계층 구조는 로봇의 의사결정 과정을 자연스럽게 추상화한다. 둘째, 하향식 명령 흐름은 ROS2 액션의 연쇄적인 호출을 통해 이루어진다. 셋째, 상향식 피드백은 세계 상태를 업데이트하여 재계획을 유발하는 루프를 통해 지능적인 대응을 가능하게 한다. 넷째, 각 계층의 제어 프리미티브는 일시적이거나 영속적인, 뚜렷하게 구분되는 생명주기를 가진다.

현재 이들을 즉시 통합하는 단일 프레임워크는 부재하지만, 본 보고서에서 제안한 구체적이고 견고한 설계 패턴들-`bt_action_node` 브리지, `RosActionNode`-SMACC 클라이언트 패턴, 그리고 World State Monitor-을 통해 성공적인 통합이 가능하다. 결론적으로, HTN-BT-FSM 아키텍처는 올바르게 구현될 경우, 차세대 자율 로봇 시스템을 구축하기 위한 매우 강력하고 유연하며 확장 가능한 기반을 제공한다.

#### **참고 자료**

1. A survey of Behavior Trees in robotics and AI | Request PDF - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/359928525_A_survey_of_Behavior_Trees_in_robotics_and_AI
2. Integration of an Automated Hierarchical Task Planner in ROS Using Behaviour Trees | Request PDF - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/322004012_Integration_of_an_Automated_Hierarchical_Task_Planner_in_ROS_Using_Behaviour_Trees
3. plansys2_core 2.0.18 documentation - ROS 2, accessed July 24, 2025, https://docs.ros.org/en/jazzy/p/plansys2_core/
4. PlanSys2/ros2_planning_system: This repo contains a PDDL-based planning system for ROS2. - GitHub, accessed July 24, 2025, https://github.com/PlanSys2/ros2_planning_system
5. PlanSys2: A Planning System Framework for ROS2 - arXiv, accessed July 24, 2025, https://arxiv.org/pdf/2107.00376
6. Integration with ROS2 | BehaviorTree.CPP, accessed July 24, 2025, https://www.behaviortree.dev/docs/ros2_integration/
7. behaviortree_cpp 4.7.0 documentation - the official ROS docs, accessed July 24, 2025, https://docs.ros.org/en/rolling/p/behaviortree_cpp/
8. robosoft-ai/SMACC2: An Event-Driven, Asynchronous, Behavioral State Machine Library for ROS2 (Robotic Operating System) applications written in C++ - GitHub, accessed July 24, 2025, https://github.com/robosoft-ai/SMACC2
9. smacc2 1.22.1 documentation - the official ROS docs, accessed July 24, 2025, https://docs.ros.org/en/rolling/p/smacc2/
10. A curated list of awesome tools and libraries for deliberation in ROS 2. - GitHub, accessed July 24, 2025, https://github.com/ros-wg-delib/awesome-ros-deliberation
11. PlanSys2 design - ROS2 Planning System 2 1.0.0 documentation, accessed July 24, 2025, https://plansys2.github.io/design/index.html
12. PlanSys2 architecture. | Download Scientific Diagram - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/figure/PlanSys2-architecture_fig1_353055585
13. ros2_planning_system/plansys2_docs/FAQ.md at rolling - GitHub, accessed July 24, 2025, https://github.com/IntelligentRoboticsLabs/ros2_planning_system/blob/master/plansys2_docs/FAQ.md
14. plansys2_planner 2.0.18 documentation - the official ROS docs, accessed July 24, 2025, https://docs.ros.org/en/jazzy/p/plansys2_planner/
15. ROS Package: plansys2_planner, accessed July 24, 2025, https://index.ros.org/p/plansys2_planner/
16. plansys2_executor - plansys2_executor 2.0.18 documentation, accessed July 24, 2025, https://docs.ros.org/en/jazzy/p/plansys2_executor/
17. Behavior Trees for Robotic Applications (ROS2) in C++ | by Markus Buchholz | Medium, accessed July 24, 2025, https://markus-x-buchholz.medium.com/behavior-trees-in-c-for-robotic-applications-ros2-775ec0e97856
18. Future of BehaviorTree.CPP - General - ROS Discourse, accessed July 24, 2025, https://discourse.ros.org/t/future-of-behaviortree-cpp/16927
19. nav2_behavior_tree: Jazzy 1.3.7 documentation - the official ROS docs, accessed July 24, 2025, https://docs.ros.org/en/jazzy/p/nav2_behavior_tree/
20. behaviortree_cpp_v3: Rolling 3.8.7 documentation - the official ROS docs, accessed July 24, 2025, https://docs.ros.org/en/rolling/p/behaviortree_cpp_v3/
21. Nav2 Behavior Trees - Nav2 1.0.0 documentation, accessed July 24, 2025, https://docs.nav2.org/behavior_trees/index.html
22. ROS2 example project / Issue #412 / BehaviorTree/BehaviorTree.CPP - GitHub, accessed July 24, 2025, https://github.com/BehaviorTree/BehaviorTree.CPP/issues/412
23. Python Behavior Tree for ROS2 ? / iRobotEducation create3_docs / Discussion #285, accessed July 24, 2025, https://github.com/iRobotEducation/create3_docs/discussions/285
24. Intro to Substate Objects | SMACC – State Machine Asynchronous C++, accessed July 24, 2025, https://smacc.dev/intro-to-substate-objects/
25. State Reactors | SMACC – State Machine Asynchronous C++, accessed July 24, 2025, https://smacc.dev/state-reactors/
26. 10. State Machines for Complex Robot Behavior, with Brett Aldrich - Sense Think Act Podcast, accessed July 24, 2025, https://www.sensethinkact.com/episodes/10-brett-aldrich
27. Introducing the SMACC State Machine Library - Page 2 - ROS General, accessed July 24, 2025, https://discourse.ros.org/t/introducing-the-smacc-state-machine-library/14963?page=2
28. Partial SMACC2 documentation - General - ROS Discourse, accessed July 24, 2025, https://discourse.ros.org/t/partial-smacc2-documentation/29810
29. Implement actions as Behavior Trees - ROS2 Planning System, accessed July 24, 2025, https://plansys2.github.io/tutorials/docs/bt_actions.html
30. ROS Package: plansys2_bt_actions, accessed July 24, 2025, https://index.ros.org/p/plansys2_bt_actions/
31. ROSA: a knowledge-based solution for robot self ... - Frontiers, accessed July 24, 2025, https://www.frontiersin.org/articles/10.3389/frobt.2025.1531743/pdf
32. Implementing BDI Continual Temporal Planning for Robotic Agents - arXiv, accessed July 24, 2025, https://arxiv.org/pdf/2309.00327
33. plansys2_msgs 2.0.18 documentation - the official ROS docs, accessed July 24, 2025, https://docs.ros.org/en/jazzy/p/plansys2_msgs/
34. Using a planning controller - ROS2 Planning System 2 1.0.0 documentation, accessed July 24, 2025, https://plansys2.github.io/tutorials/docs/controller_example.html
35. SMACC demonstrates concurrent control of two arms in ROS 2 using MoveIt2 and ros2_control, accessed July 24, 2025, https://discourse.ros.org/t/smacc-demonstrates-concurrent-control-of-two-arms-in-ros-2-using-moveit2-and-ros2-control/28793
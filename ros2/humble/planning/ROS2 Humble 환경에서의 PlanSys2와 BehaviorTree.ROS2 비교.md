# ROS2 Humble 환경에서의 PlanSys2와 BehaviorTree.ROS2 심층 비교 분석
[ROS2 Humble 행동(임무) 계획 및 관리](./index.md)



현대의 로봇 공학은 정적이고 예측 가능한 환경에서의 반복 작업을 넘어, 불확실하고 동적인 실제 세계에서 복잡한 임무를 자율적으로 수행하는 방향으로 나아가고 있다. 이러한 고도화된 자율성을 달성하기 위해서는 로봇이 단순히 사전 프로그래밍된 동작을 수행하는 것을 넘어, 현재 상황을 인식하고, 목표를 이해하며, 목표 달성을 위한 최적의 행동 순서를 스스로 추론하고 실행하는 고수준의 '태스크 지능(Task Intelligence)'을 갖추어야 한다.

과거에 널리 사용되었던 유한 상태 기계(Finite State Machine, FSM)와 같은 전통적인 제어 방식은 시스템의 모든 가능한 상태와 그 사이의 전이를 명시적으로 설계해야 했다. 이는 시스템의 복잡도가 증가함에 따라 상태의 수가 기하급수적으로 폭발하여 설계, 유지보수, 확장이 극도로 어려워지는 한계를 가진다.1 로봇이 직면할 수 있는 모든 예외 상황을 사전에 예측하고 코드로 구현하는 것은 사실상 불가능에 가깝다.

이러한 한계를 극복하기 위해 로봇 운영 체제(Robot Operating System, ROS)의 차세대 버전인 ROS2는 실시간성, 보안성, 그리고 분산 시스템의 안정성을 대폭 강화하여 산업 현장과 같이 높은 신뢰성을 요구하는 환경에 적용될 수 있는 견고한 기반을 제공한다.3 이처럼 고도화된 프레임워크 위에서, 로봇의 지능을 보다 유연하고 확장 가능하게 구현하기 위한 새로운 패러다임이 요구되었으며, 이것이 바로 AI 태스크 플래닝(Task Planning)과 반응형 행동 실행(Reactive Behavior Execution) 기술의 등장을 촉진했다.


ROS2 Humble 생태계에서 로봇의 태스크 지능을 구현하는 두 가지 핵심적인 접근법으로 `PlanSys2`와 `BehaviorTree.ROS2`를 꼽을 수 있다. 이 둘은 로봇의 행동을 제어한다는 공통점을 가지지만, 그 철학과 역할에서 근본적인 차이를 보인다.

`PlanSys2`는 인공지능 계획(AI Planning) 기술에 기반한 기호적(symbolic) 태스크 플래닝 프레임워크이다.5 이는 로봇에게 "무엇을(what)" 해야 하는지를 알려주는 것이 아니라, 달성해야 할 '목표(goal)'와 로봇이 할 수 있는 '행동(action)'의 목록을 선언적으로 정의하면, 목표 달성을 위한 최적의 행동 순서, 즉 '계획(plan)'을 동적으로 생성해주는 역할을 한다.4 이를 위해 표준 계획 언어인 PDDL(Planning Domain Definition Language)을 사용하며, 복잡한 문제 공간에서 최적의 해결책을 탐색하는 데 중점을 둔다.

반면, `BehaviorTree.ROS2`는 `BehaviorTree.CPP`라는 고성능 C++ 라이브러리를 ROS2 환경에 통합한 패키지로, 로봇이 특정 작업을 "어떻게(how)" 수행할지를 절차적으로 명시하는 반응형 행동 실행 프레임워크이다.7 이는 '액션', '조건', '제어 흐름' 등의 기본 단위인 노드(Node)들을 계층적인 트리 구조로 조합하여 복잡한 행동 로직을 모듈화하고 재사용성을 높이는 데 강점을 가진다.8 행동 트리는 실행 흐름을 명시적으로 설계하여 환경 변화에 대한 즉각적인 반응성을 구현하는 데 초점을 맞춘다.


본 보고서는 `PlanSys2`와 `BehaviorTree.ROS2`의 단순한 기능 나열을 넘어, 두 패키지의 근본적인 설계 철학, 아키텍처, 기능적 범위, 그리고 실제 로봇 애플리케이션에 적용할 때 발생하는 장단점과 트레이드오프를 심층적으로 비교 분석하는 것을 목표로 한다.

특히, 이 두 기술이 서로를 대체하는 경쟁 관계가 아니라, 로봇 지능 아키텍처의 서로 다른 추상화 계층에서 상호 보완적으로 작동하며 강력한 시너지를 창출하는 공생 관계임을 규명하는 데 중점을 둔다. 이러한 관점의 핵심 근거는 PlanSys2가 AI 플래너를 통해 생성된 추상적인 계획을 실행하는 단계에서, 그 실행의 견고함과 효율성을 높이기 위해 내부적으로 행동 트리(Behavior Tree)를 동적으로 생성하고 활용한다는 사실에 있다.4 이는 ROS2 생태계가 거대한 단일 로직으로 제어되던 과거의 방식에서 벗어나, 고수준의 '숙고(deliberation)'와 저수준의 '실행(execution)'을 분리하는 성숙한 아키텍처 패턴으로 발전하고 있음을 보여주는 명백한 증거이다. 본 보고서는 이 핵심적인 연결고리를 출발점으로 삼아 두 시스템의 관계를 다각적으로 분석할 것이다.


`PlanSys2`와 `BehaviorTree.ROS2`의 차이를 이해하기 위해서는 그 근간을 이루는 프로그래밍 패러다임인 '선언적(Declarative)' 방식과 '명령형(Imperative)' 방식의 본질적인 차이를 먼저 이해해야 한다. 이 두 패러다임의 선택은 단순히 코딩 스타일의 문제가 아니라, 시스템의 복잡성을 어디에서 어떻게 관리할 것인지에 대한 근본적인 아키텍처 설계 결정과 직결된다.


선언적 프로그래밍은 최종적으로 '무엇(What)'을 원하는지에 집중하고, 그것을 달성하기 위한 구체적인 '방법(How)'은 시스템이나 프레임워크에 위임하는 접근 방식이다.11 가장 대표적인 예는 데이터베이스 쿼리 언어인 SQL이다. 사용자는 `SELECT name FROM users WHERE age > 30`과 같이 원하는 데이터의 조건을 명시할 뿐, 데이터베이스가 어떤 인덱스를 사용하고 어떤 순서로 테이블을 스캔하여 결과를 가져올지에 대한 절차는 전혀 관여하지 않는다.14

PDDL은 이러한 선언적 패러다임을 로봇의 태스크 플래닝 문제에 적용한 표준 언어이다.16 개발자는 PDDL을 통해 다음과 같은 세 가지 핵심 요소를 정의한다:

1. **도메인(Domain):** 로봇이 활동하는 세계의 일반적인 규칙을 정의한다. 여기에는 로봇이 수행할 수 있는 모든 행동(Actions), 그 행동의 전제조건(Preconditions), 그리고 행동의 결과(Effects)가 포함된다. 예를 들어, `move`라는 행동은 로봇이 현재 위치에 있어야 하고(전제조건), 행동이 끝나면 로봇은 목표 위치에 있게 된다(결과)는 식으로 기술된다.17
2. **문제(Problem):** 특정 시나리오를 기술한다. 여기에는 세계에 존재하는 구체적인 객체(Objects), 시스템의 초기 상태(Initial State), 그리고 로봇이 궁극적으로 달성해야 할 목표 상태(Goal)가 포함된다.16
3. **플래너(Planner):** 도메인과 문제 정의를 입력받아, 초기 상태에서 목표 상태로 전이시키기 위한 유효한 행동들의 순서, 즉 '계획(Plan)'을 찾아내는 알고리즘이다.

이 방식의 가장 큰 장점은 복잡성 관리의 위임에 있다. 개발자는 방대한 상태 공간을 탐색하며 모든 경우의 수를 고려할 필요 없이, 문제의 본질(행동, 상태, 목표)을 정확하게 모델링하는 데에만 집중하면 된다. 전략적 복잡성은 검증된 플래너 알고리즘(예: POPF, TFD)이 처리한다.20 또한, 목표가 변경되더라도(예: "A 방 청소"에서 "A, B 방 모두 청소"로 변경) 도메인 파일은 그대로 둔 채 문제 파일의 목표 정의만 수정하면 되므로 높은 유연성과 재사용성을 보장한다.


명령형 프로그래밍은 목표를 달성하기 위해 컴퓨터가 수행해야 할 모든 단계를 명시적이고 순차적으로 지시하는, '어떻게(How)'에 초점을 맞추는 전통적인 프로그래밍 방식이다.11 C++, Python, Java 등 대부분의 프로그래밍 언어가 이 패러다임에 속한다.

행동 트리(Behavior Tree, BT)는 이러한 명령형 로직을 로봇 행동 제어에 맞게 특화시킨, 그래픽하고 계층적인 데이터 구조이다. BT는 개발자가 로봇의 행동 흐름을 직접 설계하고 제어할 수 있는 강력한 도구를 제공한다. 핵심 구성 요소는 다음과 같다:

- **제어 흐름 노드(Control Flow Nodes):** 자식 노드들의 실행 순서와 조건을 결정한다.
  - **Sequence (`->`):** 자식들을 왼쪽에서 오른쪽으로 순차적으로 실행한다. 모든 자식이 성공(`SUCCESS`)해야 성공을 반환하고, 하나라도 실패(`FAILURE`)하면 즉시 실행을 중단하고 실패를 반환한다. 이는 "그리고(AND)" 논리와 유사하다.22
  - **Fallback (`?`):** 자식들을 순서대로 실행하며, 하나라도 성공하면 즉시 실행을 중단하고 성공을 반환한다. 모든 자식이 실패해야 실패를 반환한다. 이는 "또는(OR)" 논리와 유사하며, 예외 처리나 대안 행동을 구현하는 데 주로 사용된다.22
- **실행 노드(Execution Nodes / Leaf Nodes):** 트리의 가장 말단에서 실제 작업을 수행한다.
  - **Action:** 로봇의 모터를 움직이거나, 계산을 수행하는 등 시스템의 상태를 변경시키는 구체적인 작업을 담당한다. 작업이 진행 중일 때는 `RUNNING` 상태를 반환할 수 있다.22
  - **Condition:** 특정 조건을 검사하여(예: `is_battery_low?`) 즉시 `SUCCESS` 또는 `FAILURE`를 반환한다.22

이 방식은 실행의 모든 세부 사항을 개발자가 직접 제어하므로, 예측 불가능한 환경 변화에 즉각적으로 반응하는 로직(reactivity)을 정교하게 설계할 수 있다.1 예를 들어, "이동" 액션 중에 장애물이 나타나면 즉시 "정지"하고 "회피"하는 로직을 BT 내에 명시적으로 구현할 수 있다. 전술적 복잡성은 개발자의 명시적인 설계를 통해 관리된다.


`PlanSys2`와 `BehaviorTree.ROS2`는 이 두 패러다임을 상호 보완적으로 통합하여 현대 로봇 아키텍처의 이상적인 모델을 제시한다. 로봇의 지능을 계층적으로 분리하여, 각 패러다임이 가장 잘 할 수 있는 영역에 집중하도록 하는 것이다.

- 상위 계층 (전략적 숙고): PlanSys2

  로봇의 장기적이고 전략적인 목표를 수립하는 역할을 한다. "집 전체를 청소하라"와 같은 추상적인 목표가 주어지면, PlanSys2는 PDDL 모델을 기반으로 현재 로봇의 상태(위치, 배터리)와 환경(각 방의 청소 상태)을 고려하여 최적의 행동 순서(예: 거실 청소 -> 부엌 청소 -> 충전 스테이션으로 이동)를 동적으로 생성한다. 이는 로봇의 '숙고(deliberation)' 과정에 해당한다.

- 하위 계층 (전술적 실행): Behavior Tree

  상위 계층에서 생성된 계획의 각 단계를 실행하거나, 개발자가 직접 설계한 특정 임무를 수행한다. 예를 들어, PlanSys2가 생성한 (vacuum living_room)이라는 단일 행동은, 실제로는 장애물 회피, 배터리 잔량 실시간 확인, 흡입력 조절 등 복잡한 로직을 포함하는 하나의 정교한 행동 트리로 구현될 수 있다.9 이는 로봇의 '실행(execution)'과 '반응(reaction)' 과정에 해당한다.

결론적으로, 가장 효과적인 로봇 아키텍처는 선언적 방식을 사용하여 '전략적 복잡성'(무엇을 어떤 순서로 할 것인가)을 관리하고, 명령형 방식을 사용하여 '전술적 복잡성'(하나의 작업을 어떻게 견고하게 수행할 것인가)을 관리한다. PlanSys2의 Executor가 PDDL 계획을 행동 트리로 변환하여 실행하는 것은 바로 이 통합 패러다임의 가장 강력하고 명백한 증거이다.4


PlanSys2는 단순히 PDDL 플래너를 ROS2에 포팅한 것을 넘어, 로봇의 지식 관리, 계획 생성, 실행 모니터링을 포괄하는 완전한 분산형 프레임워크이다. 그 아키텍처는 마이크로서비스 패러다임을 연상시키는, 고도로 분리되고 확장 가능한 설계를 특징으로 한다.


PlanSys2는 기능적으로 분리된 네 개의 독립적인 ROS2 노드로 구성되어 있으며, 각 노드는 시스템의 특정 역할을 담당한다.4 이러한 분산 구조는 각 컴포넌트의 독립적인 개발, 테스트, 배포를 가능하게 하여 시스템 전체의 유연성과 확장성을 높인다. 또한, 각 노드는 ROS2의 관리형 노드인 `LifecycleNode`로 구현될 수 있어, 시스템의 시작, 종료, 설정 단계를 체계적으로 관리하고 안정성을 확보할 수 있다.26

1. **Domain Expert (도메인 전문가)**
   - **역할:** PDDL 도메인 파일(`.pddl`)을 파싱하여 시스템의 정적인 '능력'에 대한 정보를 관리하고 제공한다. 이 정보에는 로봇이 사용할 수 있는 객체의 유형(types), 세계의 상태를 표현하는 술어(predicates)와 함수(functions), 그리고 로봇이 수행할 수 있는 모든 행동(actions)의 정의가 포함된다.5
   - **특징:** 이 정보는 시스템이 실행되는 동안 일반적으로 변하지 않는 정적 데이터이다. 다른 노드들은 ROS2 서비스(Service)를 통해 Domain Expert에게 특정 행동의 파라미터가 무엇인지, 혹은 특정 술어가 유효한지 등을 질의할 수 있다. `transient_local` QoS 프로파일을 사용하여 도메인 정보를 발행함으로써, 나중에 시스템에 합류하는 노드도 즉시 최신 도메인 정보를 획득할 수 있도록 설계되어 있다.17
2. **Problem Expert (문제 전문가)**
   - **역할:** 로봇의 '지식 베이스' 또는 '믿음(belief)'에 해당하는 동적인 정보를 관리한다. 여기에는 현재 세계에 존재하는 구체적인 객체 인스턴스(instances), 각 술어의 참/거짓 상태, 함수의 현재 값, 그리고 달성해야 할 목표(goal)가 포함된다.4
   - **특징:** 이 정보는 로봇의 센서 데이터, 행동의 결과, 또는 외부 명령에 의해 실시간으로 변한다. 예를 들어, 로봇이 `(move r2d2 kitchen)` 행동을 성공적으로 마치면, Problem Expert는 `(robot_at r2d2 kitchen)` 술어를 `true`로 업데이트한다. 상태 변경이 있을 때마다 토픽을 통해 이를 알려, 다른 노드들이 변경 사항을 인지하고 반응할 수 있도록 하는 이벤트 기반 아키텍처를 따른다.5
3. **Planner (계획기)**
   - **역할:** 실제 계획 생성을 담당하는 핵심 두뇌이다. 계획 요청이 들어오면, Domain Expert로부터 도메인 정보를, Problem Expert로부터 현재 상태와 목표 정보를 가져온다.4
   - **특징:** PlanSys2 자체는 계획 알고리즘을 포함하지 않는다. 대신, 획득한 도메인과 문제 정보를 PDDL 파일 형식으로 변환하여 외부 PDDL 솔버(Solver)를 호출하고 그 결과를 파싱하는 역할을 한다.20 기본적으로 POPF(Partial-Order Plan-Forward) 플래너를 사용하며, TFD(Temporal Fast Downward)와 같은 다른 플래너도 플러그인(plugin) 형태로 쉽게 통합할 수 있도록 설계되어 있다.17 이 플러그인 기반 아키텍처 덕분에 사용자는 특정 문제에 더 적합한 최신 계획 알고리즘으로 쉽게 교체할 수 있다.
4. **Executor (실행기)**
   - **역할:** Planner가 생성한 추상적인 행동 계획을 실제 로봇의 동작으로 변환하고 실행을 총괄한다. 계획(일련의 행동 시퀀스)을 입력받아, 각 행동을 순서대로 또는 병렬로 실행한다.5
   - **특징:** 각 PDDL 행동을 실제로 수행하는 ROS2 노드, 즉 '액션 수행자(Action Performer)'에게 ROS2 액션(Action)을 통해 실행을 요청한다. 실행기는 행동 시작 전 전제조건이 충족되는지 확인하고, 행동이 진행되는 동안에도 전제조건이 계속 유효한지 감시한다. 만약 조건이 깨지면 현재 계획의 실행을 즉시 중단하고 실패를 보고하는 등 견고한 실행 관리를 담당한다.28


PlanSys2를 이용한 로봇 애플리케이션 개발은 다음과 같은 명확한 생명주기를 따른다.

1. **모델링 (Modeling):** 개발자는 로봇이 활동할 환경과 로봇 자체의 능력을 PDDL 도메인 파일로 추상화하여 정의한다. 이는 애플리케이션의 가장 근본적인 설계 단계에 해당한다.18
2. **지식 초기화 (Knowledge Initialization):** 시스템이 시작될 때, Problem Expert에 세계의 초기 상태를 설정한다. 예를 들어, 로봇의 초기 위치, 존재하는 방들의 목록, 방들의 연결 관계 등을 인스턴스와 술어로 추가한다. 이 과정은 개발 및 테스트 시에는 `plansys2_terminal`이라는 CLI 도구를 통해 수동으로 수행할 수 있으며, 실제 애플리케이션에서는 전용 지식 관리 노드가 센서 데이터를 해석하여 자동으로 수행한다.18
3. **계획 요청 (Planning Request):** 사용자가 명령을 내리거나 특정 조건이 충족되면, Problem Expert에 목표(goal)가 설정된다. 예를 들어, `(set goal (and (robot_at leia bathroom)))`과 같이 목표를 설정할 수 있다. 목표가 설정되면 클라이언트 노드는 Planner에게 계획 생성을 요청한다.18
4. **계획 실행 (Plan Execution):** Planner가 성공적으로 계획(예: `(charge)`, `(move)`, `(move)`)을 생성하면, 이 계획은 Executor에게 전달된다. Executor는 계획의 첫 번째 행동인 `charge`를 수행할 Action Performer 노드를 찾아 ROS2 액션으로 실행을 요청한다. `charge` 액션이 성공적으로 완료되면, 다음 행동인 `move`를 실행하는 식으로 계획을 순차적으로 진행한다.28
5. **상태 업데이트 및 모니터링 (State Update and Monitoring):** Action Performer는 행동 수행이 완료되면 그 결과를(성공/실패) Executor에게 피드백한다. Executor는 이 결과를 바탕으로 Problem Expert의 상태를 업데이트하여 세계 모델을 현실과 동기화한다. 예를 들어, `move` 행동이 성공하면 로봇의 위치 술어가 변경된다. 이 과정은 계획의 마지막 행동이 완료될 때까지 반복된다.


PlanSys2의 Executor는 단순히 계획을 순차적으로 실행하는 데 그치지 않는다. 그 핵심에는 생성된 PDDL 계획을 내부적으로 최적화된 행동 트리로 변환하여 실행하는 정교한 메커니즘이 존재한다. 이는 PlanSys2의 성능과 유연성을 극대화하는 핵심적인 설계 철학이다.4

이 변환 과정은 다음과 같이 이루어진다:

1. **실행 그래프 생성 (Execution Graph Generation):** 먼저, PDDL 계획에 포함된 각 행동들의 전제조건(preconditions)과 효과(effects)를 분석하여 행동 간의 인과 관계(causal relationships)를 파악한다. 이를 통해 어떤 행동이 다른 행동보다 반드시 먼저 수행되어야 하는지를 나타내는 '실행 그래프'를 구축한다.10
2. **병렬 실행 흐름 식별 (Identifying Parallel Execution Flows):** 실행 그래프에서 서로 의존성이 없는 행동들을 식별한다. 예를 들어, 로봇 A가 부품 1을 가져오는 행동과 로봇 B가 부품 2를 가져오는 행동은 서로 독립적이므로 동시에 수행될 수 있다.
3. **행동 트리 동적 생성 (Dynamic Behavior Tree Generation):** 식별된 실행 흐름을 바탕으로 행동 트리를 동적으로 생성한다.
   - 서로 의존성이 없는 병렬 실행 가능한 행동들은 BT의 `Parallel` 노드의 자식으로 배치된다.
   - 반드시 순서대로 실행되어야 하는 행동들은 BT의 `Sequence` 노드의 자식으로 배치된다.
   - 이 과정을 통해 생성된 행동 트리는 원본 PDDL 계획의 논리적 제약은 모두 만족시키면서도, 가능한 모든 병렬성을 활용하여 전체 임무 수행 시간을 크게 단축시킨다.10

이처럼 PlanSys2는 AI 플래닝의 '전략적 최적화'와 행동 트리의 '전술적 실행 효율성'을 결합하여, 단순한 계획 실행기를 넘어선 고성능 태스크 오케스트레이션 프레임워크로서의 면모를 보여준다.


`BehaviorTree.ROS2`는 로봇의 행동 로직을 설계하고 실행하기 위한 강력하고 유연한 도구이다. 이는 단순히 ROS2용 라이브러리를 제공하는 것을 넘어, 복잡한 로봇 소프트웨어를 체계적으로 구성하기 위한 특정 아키텍처 패턴, 즉 '중앙 집중형 코디네이터(Centralized Coordinator)' 패턴을 강력하게 유도한다.


`BehaviorTree.ROS2`의 근간은 Davide Faconti가 개발한 고성능 C++17 라이브러리인 `BehaviorTree.CPP`이다.7

`BehaviorTree.ROS2`는 이 핵심 라이브러리의 기능들을 ROS2 통신 메커니즘과 자연스럽게 통합시켜주는 래퍼(wrapper)와 유틸리티의 집합이다.7

행동 트리의 실행은 '틱(tick)'이라는 주기적인 신호에 의해 구동된다. 틱 신호는 트리의 루트(root) 노드에서 시작하여 자식 노드로 전파되며, 각 노드는 틱을 받을 때마다 자신의 로직을 실행하고 세 가지 상태 중 하나를 반환한다 22:

- `SUCCESS`: 작업이 성공적으로 완료되었음을 의미한다.
- `FAILURE`: 작업이 실패했음을 의미한다.
- `RUNNING`: 작업이 아직 진행 중이며, 완료되기까지 더 많은 시간이 필요함을 의미한다.

트리를 구성하는 노드는 기능에 따라 크게 네 가지 유형으로 분류된다.

1. **제어 흐름 노드 (Control Flow Nodes):** 자식 노드들의 실행 순서와 방식을 결정하며, 트리의 논리적 구조를 형성한다.
   - **`Sequence`:** "AND" 로직을 구현한다. 자식 노드들을 순서대로 틱하며, 하나라도 `FAILURE`를 반환하면 즉시 중단하고 `FAILURE`를 반환한다. 모든 자식이 `SUCCESS`해야만 `SUCCESS`를 반환한다.22
   - **`Fallback` (또는 `Selector`):** "OR" 로직을 구현한다. 자식 노드들을 순서대로 틱하며, 하나라도 `SUCCESS`를 반환하면 즉시 중단하고 `SUCCESS`를 반환한다. 모든 자식이 `FAILURE`해야만 `FAILURE`를 반환한다. 주로 오류 처리나 대체 행동 선택에 사용된다.22
   - **`Parallel`:** 여러 자식 노드를 동시에 틱한다. 성공 및 실패 조건을 유연하게 설정할 수 있다(예: N개의 자식이 성공하면 `SUCCESS` 반환).
2. **실행 노드 (Execution Nodes / Leaf Nodes):** 트리의 말단에서 실제 작업을 수행한다.
   - **`Action`:** 로봇의 상태를 변경하는 실질적인 작업을 수행한다. 예를 들어, `MoveBase` 액션을 호출하여 로봇을 이동시키거나, 그리퍼를 제어하는 등의 작업을 포함한다. 비동기적인(asynchronous) 작업의 경우, 작업이 완료될 때까지 `RUNNING` 상태를 반환하는 것이 일반적이다.22
   - **`Condition`:** 시스템의 특정 상태를 확인하는 역할을 한다. 예를 들어, `IsBatteryLow` 조건은 배터리 잔량을 확인하여 임계값보다 낮으면 `SUCCESS`를, 아니면 `FAILURE`를 즉시 반환한다. 조건 노드는 `RUNNING` 상태를 반환해서는 안 된다.22
3. **데코레이터 노드 (Decorator Nodes):** 오직 하나의 자식 노드만 가지며, 그 자식 노드의 동작을 수정하거나 강화하는 역할을 한다.
   - **`Inverter`:** 자식 노드의 결과를 반전시킨다 (`SUCCESS` -> `FAILURE`, `FAILURE` -> `SUCCESS`).
   - **`Retry`:** 자식 노드가 `FAILURE`를 반환했을 때, 지정된 횟수만큼 재시도한다.
   - **`Timeout`:** 자식 노드가 지정된 시간 내에 완료되지 않으면 `FAILURE`를 반환하고 자식의 실행을 중단시킨다.


`BehaviorTree.ROS2`의 가장 큰 가치는 ROS2의 핵심 통신 메커니즘, 특히 비동기 통신 방식인 액션(Action)과 서비스(Service)를 행동 트리 노드로 완벽하게 추상화하여 제공한다는 점에 있다.7 이를 통해 개발자는 복잡한 비동기 콜백 처리나 스레드 관리 없이도 장시간 소요되는 작업을 간결하게 BT 로직에 통합할 수 있다.

- **ROS 액션 클라이언트 노드 (`RosActionNode`):** ROS2 액션을 호출하는 BT 노드를 생성하기 위한 강력한 템플릿 클래스를 제공한다. 개발자는 이 템플릿을 상속받아 몇 가지 핵심 콜백 함수만 구현하면 된다 8:

  - `setGoal()`: 노드가 처음 틱될 때 호출되며, ROS2 액션 서버에 보낼 목표(goal) 메시지를 설정한다.

  - `onResultReceived()`: 액션 서버가 최종 결과를 반환했을 때 호출되며, 그 결과에 따라 노드의 최종 상태를 `SUCCESS` 또는 `FAILURE`로 결정한다.

  - `onFeedback()`: 액션이 실행되는 동안 서버가 보내는 피드백 메시지를 처리한다.

  - onFailure(): 통신 오류 등으로 액션 자체가 실패했을 때 호출된다.

    이 노드는 액션이 실행되는 동안 RUNNING 상태를 유지하며, ROS2의 비동기 통신을 BT의 동기적인 틱 모델에 자연스럽게 통합시킨다. 이 패턴은 ROS2 Nav2 스택에서 경로 추종, 연산 등 핵심 기능을 오케스트레이션하는 데 널리 사용된다.35

- **ROS 서비스 클라이언트 노드 (`RosServiceNode`):** ROS2 서비스를 비동기적으로 호출하는 BT 노드를 제공한다. `setRequest()` 콜백으로 요청 메시지를 채우고 `onResponseReceived()` 콜백으로 응답을 처리하는 유사한 구조를 가진다.8

- **토픽 발행/구독 노드 (Topic Publisher/Subscriber Nodes):** 특정 토픽에 메시지를 발행하거나, 토픽에서 메시지를 수신하여 블랙보드(Blackboard)에 저장하는 등의 간단한 노드들을 제공하여 BT가 ROS2 네트워크와 유연하게 상호작용할 수 있도록 지원한다.7


`BehaviorTree.ROS2`는 효율적인 개발 및 디버깅 워크플로우를 지원하는 도구와 철학을 제공한다.

- **XML 기반 트리 정의:** 행동 트리의 구조, 즉 노드들의 연결 관계는 C++ 코드에 직접 작성하는 것이 아니라 별도의 XML 파일을 통해 정의된다.33 이는 컴파일 없이 로봇의 행동 로직을 수정, 재구성, 확장할 수 있게 하여 개발 유연성을 극대화한다. 행동 설계자와 노드 구현자의 역할을 명확히 분리할 수 있는 장점도 있다.

- **노드의 모듈화와 재사용성:** `BehaviorTree.CPP`의 핵심 설계 원칙은 각 노드를 재사용 가능한 독립적인 빌딩 블록으로 만드는 것이다.2 잘 설계된 

  `MoveTo` 액션 노드나 `IsObjectDetected` 조건 노드는 하나의 프로젝트에 국한되지 않고, 다양한 로봇과 애플리케이션에서 레고 블록처럼 재사용될 수 있다. 이는 개발 효율성을 크게 향상시키고 코드의 모듈성을 유지하는 데 기여한다.39

- **Groot (시각적 GUI 도구):** `BehaviorTree.ROS2` 개발 경험의 핵심 요소는 `Groot`이라는 강력한 GUI 도구이다.

  - **에디터(Editor):** 개발자는 코드를 직접 작성하는 대신, 드래그-앤-드롭 인터페이스를 통해 직관적으로 노드를 배치하고 연결하여 행동 트리를 시각적으로 설계할 수 있다.41

  - **모니터(Monitor):** `Groot`의 가장 강력한 기능으로, 실제 로봇이나 시뮬레이션에서 실행 중인 행동 트리에 실시간으로 연결할 수 있다. 모니터 화면에서는 각 노드의 상태(`SUCCESS`, `FAILURE`, `RUNNING`)가 색상으로 표시되며, 틱이 전파되는 과정을 실시간으로 확인할 수 있다.43 이는 복잡한 로직의 흐름을 파악하고 예상치 못한 동작의 원인을 찾는 디버깅 과정에 결정적인 도움을 준다. 

    `BehaviorTree.ROS2`는 `Groot`과의 연동을 위한 ZeroMQ 통신 기능을 내장하고 있어 쉽게 모니터링 환경을 구축할 수 있다.17

이러한 도구와 워크플로우는 `BehaviorTree.ROS2`가 단순한 라이브러리를 넘어, 로봇의 복잡한 행동을 체계적으로 설계하고 관리하기 위한 성숙한 개발 생태계를 제공함을 보여준다. 특히 중앙 코디네이터 패턴을 통해 시스템의 전체적인 행동 로직을 하나의 BT에 집중시키고, 나머지 ROS 노드들은 특정 기능만 제공하는 단순한 서비스 서버로 구현하도록 유도함으로써, 분산 시스템에서 발생하기 쉬운 로직 파편화 문제를 해결하고 시스템 전체의 이해도와 유지보수성을 크게 향상시킨다.8


`PlanSys2`와 `BehaviorTree.ROS2`는 로봇 지능을 구현하는 데 있어 서로 다른 추상화 수준에서 문제를 접근한다. 이들의 차이점과 상호 관계를 명확히 이해하는 것은 효과적인 로봇 아키텍처를 설계하는 데 필수적이다.


두 시스템의 근본적인 차이점은 다음 표에 요약되어 있다. 이 표는 개발자가 특정 애플리케이션에 어떤 도구가 더 적합한지, 또는 어떻게 조합해야 할지를 판단하는 데 유용한 기준을 제공한다.

**Table 1: PlanSys2 vs. BehaviorTree.ROS2 핵심 비교**

| **항목 (Axis)**                       | **PlanSys2**                                       | **BehaviorTree.ROS2**                               |
| ------------------------------------- | -------------------------------------------------- | --------------------------------------------------- |
| **주요 역할 (Primary Role)**          | 태스크 플래닝 프레임워크 (Task Planning Framework) | 행동 실행 프레임워크 (Behavior Execution Framework) |
| **핵심 패러다임 (Core Paradigm)**     | 선언적 (Declarative) - "무엇을 할 것인가"          | 명령형 (Imperative) - "어떻게 할 것인가"            |
| **추상화 수준 (Abstraction Level)**   | 높음 (High) - 기호적 수준의 행동과 세계 모델       | 낮음 (Low) - 구체적인 실행 로직과 ROS 통신          |
| **입력 (Input)**                      | PDDL 도메인/문제 파일, 목표 상태                   | XML로 정의된 행동 트리, ROS2 메시지/서비스          |
| **출력 (Output)**                     | 행동 순서 계획 (Plan), 실행 결과                   | 행동의 성공/실패/진행 상태                          |
| **반응성 모델 (Reactivity Model)**    | 계획 실패 시 재계획 (Replanning on plan failure)   | 내장된 제어 흐름을 통한 즉각적 반응 (In-tree logic) |
| **모듈화 단위 (Unit of Modularity)**  | PDDL Action, Domain                                | Behavior Tree Node, Sub-Tree                        |
| **개발 워크플로우 (Dev. Workflow)**   | PDDL 모델링 -> 지식 설정 -> 계획 -> 실행           | C++ 노드 구현 -> XML 트리 설계 -> 실행              |
| **핵심 도구 (Key Tooling)**           | `plansys2_terminal`, PDDL 검증기                   | `Groot` (시각적 편집기/모니터)                      |
| **주요 사용 사례 (Primary Use Case)** | 복잡한 순서의 장기 목표 달성, 동적 작업 할당       | 반응성이 중요한 단일 임무 수행, 상태 관리           |


분석의 핵심은 두 시스템이 상호 배타적인 선택지가 아니라, 서로의 강점을 활용하여 더욱 강력한 시스템을 구축할 수 있는 공생 관계에 있다는 점이다. 이들의 통합은 여러 수준에서 이루어질 수 있다.

- 시나리오 1: PlanSys2가 BT를 사용하는 경우 (PlanSys2 Uses BTs - 기본 통합)

  가장 기본적인 통합 형태로, PlanSys2의 표준 동작 방식이다. Executor는 Planner가 생성한 PDDL 계획을 단순히 순차적으로 실행하는 것이 아니라, 계획 내 행동들의 인과 관계를 분석하여 최적화된 행동 트리로 동적으로 변환한 후 실행한다.4 이 방식은 PlanSys2가 행동 트리의 병렬 실행 능력과 견고한 실행 모델의 이점을 기본적으로 활용하고 있음을 보여준다. 개발자는 별도의 작업 없이도 계획 실행의 효율성을 높일 수 있다.

- 시나리오 2: BT가 PlanSys2를 사용하는 경우 (BTs Use PlanSys2 - 동적 계획 통합)

  더 큰 규모의 행동 트리 내에서, 특정 상황에 맞는 계획이 동적으로 필요할 때 PlanSys2를 하나의 '서비스'처럼 호출하는 고급 아키텍처이다. 예를 들어, '순찰' 임무를 수행하는 BT가 있다고 가정하자. 이 BT는 평소에는 순찰 경로를 따라 이동하지만, IsAnomalyDetected 조건 노드가 SUCCESS를 반환하면, 순찰 로직을 중단하고 PlanResponse라는 커스텀 액션 노드를 틱할 수 있다. 이 PlanResponse 노드는 내부적으로 PlanSys2의 Problem Expert에 현재 상황(예: 이상 현상 위치)과 새로운 목표(예: (and (anomaly_reported) (robot_at safe_zone)))를 설정한 뒤, Planner에 계획 생성을 요청한다. PlanSys2가 계획을 반환하면, PlanResponse 노드는 해당 계획을 실행하는 서브트리를 동적으로 로드하여 실행한다. 이는 정적인 BT 로직에 동적인 계획 능력을 부여하는 강력한 하이브리드 방식이다.

- 시나리오 3: PDDL Action을 BT로 구현 (Implementing PDDL Actions as BTs - 계층적 추상화)

  두 시스템을 결합하는 가장 실용적이고 강력한 패턴이다. 이는 PlanSys2의 plansys2_bt_actions 패키지를 통해 직접적으로 지원된다.45 이 시나리오에서는 PlanSys2가 다루는 PDDL Action을 매우 추상적인 수준(예: `(transport object from_loc to_loc)`)으로 유지한다. 그리고 이 추상적인 Action 하나하나를, 실제 세계의 복잡성과 불확실성을 다루는 정교한 행동 트리로 구현한다.9

  예를 들어, (transport...) PDDL Action은 다음과 같은 로직을 포함하는 BT로 구현될 수 있다:

  1. `MoveTo` 액션 노드를 사용해 `from_loc`으로 이동한다.

  2. `ApproachObject` 액션 노드로 객체에 정밀하게 접근한다.

  3. `OpenGripper` -> `CloseGripper` 시퀀스로 객체를 집는다.

  4. `IsGraspSuccessful` 조건 노드로 파지 성공 여부를 확인한다. 실패 시 `Fallback`을 통해 재시도 로직을 수행한다.

  5. `MoveTo` 액션 노드를 사용해 `to_loc`으로 이동한다.

  6. PlaceObject 서브트리를 실행하여 객체를 내려놓는다.

     이때 PlanSys2는 PDDL Action의 파라미터(예: object, from_loc, to_loc)를 행동 트리가 공유하는 데이터 공간인 '블랙보드(Blackboard)'에 arg0, arg1, arg2와 같은 키-값 형태로 자동으로 전달해준다. BT 내의 각 노드들은 이 블랙보드 값을 읽어 자신의 파라미터로 사용하여 동작한다.9 이 방식은 계획의 '전략적 추상성'과 실행의 '전술적 견고함'을 완벽하게 결합하여, 복잡한 로봇 시스템을 위한 이상적인 계층적 아키텍처를 구현한다.


- **PlanSys2:** PDDL이라는 선언적 언어와 자동 계획(Automated Planning) 분야의 이론적 배경에 대한 이해가 필요하므로, 관련 경험이 없는 개발자에게는 초기 학습 곡선이 상대적으로 가파를 수 있다. 디버깅의 초점은 "왜 플래너가 계획을 생성하지 못하는가?" 또는 "왜 생성된 계획이 예상과 다른가?"에 맞춰진다. 이는 `plansys2_terminal`을 사용하여 현재 Problem Expert의 지식 상태(술어, 인스턴스)가 정확한지 확인하고, PDDL 도메인/문제 파일의 논리적 오류를 찾는 과정으로 이루어진다.18
- **BehaviorTree.ROS2:** C++과 ROS2에 대한 기본적인 지식이 있다면, 명령형 로직을 다루는 방식이 일반적인 프로그래밍과 유사하여 비교적 학습 곡선이 완만하다. 특히 `Groot`의 존재는 학습과 디버깅 경험을 획기적으로 개선한다.41 복잡한 행동 트리의 전체 구조와 실행 흐름을 시각적으로 파악할 수 있으며, 실시간 모니터링 기능을 통해 어느 노드에서 `FAILURE`가 발생했는지, 또는 어떤 로직 분기가 예상과 다르게 동작하는지를 직관적으로 추적할 수 있다.43 이는 복잡한 상호작용과 동시성 문제를 디버깅하는 데 매우 강력한 장점이다.


이론적 분석을 바탕으로, 실제 로봇 애플리케이션에 `PlanSys2`와 `BehaviorTree.ROS2`를 적용할 때 고려해야 할 시나리오별 아키텍처와 일반적인 설계 원칙을 제시한다.


- **시나리오 A: 동적 물류 창고 로봇**
  - **문제 정의:** 수시로 변경되는 주문에 따라 창고 내 여러 위치에서 물품을 픽업하여 지정된 포장 스테이션으로 운반해야 한다. 작업 공간에는 다른 로봇이나 작업자가 존재하며, 이동 경로는 동적으로 변할 수 있다.
  - **추천 아키텍처: PlanSys2 주도형 계층 구조**
    1. **상위 제어 (PlanSys2):** 시스템의 최상위 컨트롤러로 PlanSys2를 사용한다. 새로운 주문이 접수되면, 이는 `(item_at item_X packing_station)`과 같은 PDDL 목표로 변환되어 Problem Expert에 추가된다. PlanSys2의 Planner는 현재 로봇의 위치, 각 물품의 위치, 창고 내 경로 정보 등을 고려하여 가장 효율적인 픽업 및 이동 순서를 동적으로 계획한다.
    2. **하위 실행 (BehaviorTree.ROS2):** PlanSys2가 생성한 `(move...)`, `(pickup...)`, `(place...)`과 같은 각 PDDL Action은 정교한 행동 트리로 구현된다. 예를 들어, `(move robot loc_A loc_B)` Action은 ROS2 Nav2 스택을 호출하는 BT로 구현되며, 이 BT는 동적 장애물 회피, 경로 막힘 시 대기 또는 우회와 같은 반응적인 행동을 내장한다.
    3. **예외 처리:** `(pickup...)` BT가 물품 집기에 여러 번 실패하면 `FAILURE`를 반환한다. Executor는 이 실패를 감지하고 현재 계획을 중단시킨다. 상위 제어 로직은 Problem Expert의 상태(예: `(grasp_failed item_X)`)를 업데이트하고 PlanSys2에 재계획(replanning)을 요청하여, 해당 물품을 건너뛰고 다음 작업을 수행하는 등의 새로운 대안 계획을 수립하도록 한다.
- **시나리오 B: 복잡한 조립 작업을 수행하는 매니퓰레이터**
  - **문제 정의:** 정해진 순서와 절차에 따라 여러 부품을 정밀하게 조립해야 한다. 각 단계는 힘/토크 센서나 비전 시스템의 피드백에 따라 실시간으로 미세 조정이 필요하며, 실패 시 정교한 복구 절차를 따라야 한다.
  - **추천 아키텍처: BehaviorTree.ROS2 주도형 구조**
    1. **핵심 제어 (BehaviorTree.ROS2):** 조립 순서가 대부분 고정되어 있으므로, 동적 계획의 필요성이 낮다. 따라서 전체 조립 프로세스를 하나의 거대한 행동 트리로 설계하는 것이 더 효율적이다.
    2. **모듈화:** "나사 조이기", "부품 삽입", "커넥터 체결"과 같은 각 조립 단위는 서브트리(Sub-tree)로 모듈화하여 재사용성을 높인다. 각 서브트리 내에서는 `ApplyForceUntilContact` 액션 노드, `IsAlignmentCorrect` 조건 노드, `RetryWithOffset` 액션 노드 등이 `Sequence`와 `Fallback`으로 정교하게 조합되어 센서 기반의 반응형 제어를 구현한다.
    3. **계획의 부재:** 이 시나리오에서는 "어떤 순서로 조립할까?"를 고민할 필요가 없으므로 PlanSys2는 사용되지 않거나, 특정 부품이 누락되었을 때 "누락 부품 요청"과 같은 매우 제한적인 상황에서만 보조적으로 호출될 수 있다.
- **시나리오 C: 순찰 및 이상 감지 보안 로봇**
  - **문제 정의:** 평소에는 정해진 경로를 순찰하지만, 열 감지 센서나 카메라를 통해 이상 현상(예: 미등록 인물, 화재 징후)을 감지하면 즉시 순찰 임무를 중단하고 사전에 정의된 대응 프로토콜을 수행해야 한다.
  - **추천 아키텍처: 하이브리드 접근법**
    1. **기본 행동 (BehaviorTree.ROS2):** 로봇의 기본 행동은 "순찰"과 "이상 감지"라는 두 가지 병렬적인 작업을 수행하는 BT로 관리한다. `Parallel` 노드 아래에 `PatrolRoute` 서브트리와 `MonitorForAnomalies` 서브트리를 둔다. `MonitorForAnomalies`는 센서 데이터를 지속적으로 확인하는 `Condition` 노드들을 포함한다.
    2. **상황 전환:** 평소에는 `PatrolRoute` 서브트리가 활성화되어 순찰 지점들을 순차적으로 이동한다. 만약 `MonitorForAnomalies` 서브트리 내의 `IsFireDetected` 조건이 `SUCCESS`를 반환하면, `Parallel` 노드의 정책에 따라 `PatrolRoute` 서브트리의 실행이 즉시 중단(halt)되고, 제어 흐름은 `RespondToAnomaly` 서브트리로 넘어간다.
    3. **동적 대응 (PlanSys2):** `RespondToAnomaly` 서브트리 내에서는 PlanSys2를 호출하여 현재 상황에 가장 적합한 대응 계획을 동적으로 생성할 수 있다. 예를 들어, "가장 가까운 소화기의 위치를 파악하고, 본부에 경보를 전송하며, 안전한 대피 경로를 계산"하는 복잡한 목표를 PlanSys2에 전달하면, 플래너는 이를 달성하기 위한 최적의 행동 순서를 생성하여 반환한다. BT는 이 계획을 받아 실행한다.


위 시나리오들을 바탕으로, 두 시스템을 효과적으로 통합하기 위한 일반적인 설계 원칙은 다음과 같다.

- **계층적 설계 (Hierarchical Design):** 로봇의 지능을 '숙고(Deliberation)' 계층과 '실행(Execution)' 계층으로 명확히 분리하라. 숙고 계층은 PlanSys2를 통해 "무엇을, 왜" 할 것인지를 결정하고, 실행 계층은 BehaviorTree.ROS2를 통해 "어떻게" 할 것인지를 구체화한다. 이 '추상화 샌드위치' 구조는 복잡성을 체계적으로 관리하는 핵심이다.

- **추상화 유지 (Maintain Abstraction):** PDDL Action은 최대한 추상적인 수준에서 의도를 표현하도록 설계하라. `(move_forward_1_meter)`와 같은 저수준 명령이 아닌, `(navigate robot from_loc to_loc)`과 같이 고수준의 목표를 나타내는 행동을 정의해야 한다. 구체적인 이동 방법, 장애물 회피, 실패 복구 등은 모두 하위 BT 구현에 위임하여 계획의 유연성을 확보하라.

- **반응성과 재계획의 균형 (Balance Reactivity and Replanning):** 모든 예외 상황을 BT 내에서 처리하려고 시도하는 것은 트리를 불필요하게 복잡하게 만든다.

  - **BT 내 처리:** 일시적이고 예측 가능한 실패(예: 경로 상의 사소한 장애물, 물체 파지 일시 실패)는 BT의 `Fallback`이나 `Retry` 데코레이터를 사용하여 즉각적으로 처리함으로써 빠른 반응성을 확보하라.
  - **PlanSys2로 위임:** 계획의 근본적인 가정을 무너뜨리는 중대한 실패(예: 목표 지점 영구적 접근 불가, 로봇의 특정 기능 고장)는 현재 BT 실행을 `FAILURE`로 종료하고, 상위 계층인 PlanSys2가 변경된 세계 모델을 기반으로 완전히 새로운 계획을 수립(재계획)하도록 위임하라.

- **도구의 적극적 활용 (Leverage Tooling):** 개발 생산성과 시스템의 안정성을 극대화하기 위해 각 시스템이 제공하는 도구를 적극적으로 활용하라. PDDL 모델의 논리적 정합성은 VSCode PDDL 확장 기능과 같은 검증기(Validator)를 통해 사전에 충분히 검증하라.49 복잡한 행동 트리의 로직은 

  `Groot`을 통해 시각적으로 설계하고, 실시간 모니터링 기능을 활용하여 철저히 디버깅하라.



본 보고서는 ROS2 Humble 환경에서 로봇의 고수준 지능을 구현하는 두 가지 핵심 프레임워크인 `PlanSys2`와 `BehaviorTree.ROS2`에 대한 심층적인 비교 분석을 수행했다. 분석을 통해 도출된 핵심 결론은 다음과 같다.

- **패러다임의 분리:** `PlanSys2`는 PDDL을 사용하는 선언적(Declarative) 태스크 플래닝 시스템으로, 로봇이 달성해야 할 '무엇(What)'을 정의하면 최적의 행동 계획을 동적으로 생성한다. 반면, `BehaviorTree.ROS2`는 명령형(Imperative) 행동 실행 프레임워크로, 로봇이 작업을 '어떻게(How)' 수행할지를 명시적으로 설계하여 반응성과 견고함을 확보한다.
- **경쟁이 아닌 공생 관계:** 두 시스템은 서로를 대체하는 기술이 아니라, 로봇 아키텍처의 서로 다른 계층에서 상호 보완적으로 작동하는 강력한 공생 관계를 형성한다. 이 관계의 가장 명백한 증거는 PlanSys2가 생성된 계획을 실행할 때, 내부적으로 이를 최적화된 행동 트리로 변환하여 `BehaviorTree.CPP`의 실행 모델을 활용한다는 점이다.
- **계층적 아키텍처의 효용성:** 이들의 조합은 복잡한 로봇 임무를 '전략적 계획 수립(PlanSys2)'과 '전술적 행동 실행(BehaviorTree.ROS2)'이라는 두 개의 명확한 계층으로 분리하여 관리할 수 있게 한다. 이러한 계층적 아키텍처는 시스템의 모듈성, 확장성, 그리고 유지보수성을 크게 향상시키는 현대 로봇 소프트웨어 설계의 핵심 원칙이다.


로봇 기술이 점차 더 복잡하고 예측 불가능한 환경으로 확장됨에 따라, 하드코딩된 로직만으로는 한계가 명확하다. 미래의 자율 로봇은 주어진 목표를 달성하기 위해 스스로 계획을 수립하고(deliberation), 예기치 못한 상황에 유연하게 대처하며(reaction), 실패로부터 학습하여 계획을 수정(replanning)하는 능력이 필수적으로 요구될 것이다.

이러한 맥락에서 PlanSys2와 같은 AI 플래닝 기술과 BehaviorTree.ROS2와 같은 반응형 실행 기술을 통합하는 프레임워크의 중요성은 앞으로 더욱 커질 것이다. 이 두 기술의 결합은 로봇에게 장기적인 목표 지향성과 단기적인 환경 적응성을 동시에 부여하는 가장 효과적인 방법론 중 하나이다.

따라서 로봇 시스템 설계자가 내려야 할 결정은 "PlanSys2 *또는* BehaviorTree.ROS2 중 무엇을 선택할 것인가?"가 아니다. 대신, "주어진 애플리케이션의 요구사항을 충족시키기 위해 PlanSys2*와* BehaviorTree.ROS2를 *어떻게* 최적으로 조합하여, 견고하고 유연하며 확장 가능한 계층적 지능 시스템을 구축할 것인가?"가 되어야 한다. 이 질문에 대한 답을 찾는 과정이 곧 차세대 자율 로봇의 지능을 설계하는 핵심적인 여정이 될 것이다.


1. State Machines vs Behavior Trees: designing a decision-making architecture for robotics, accessed August 18, 2025, https://www.polymathrobotics.com/blog/state-machines-vs-behavior-trees
2. About - BehaviorTree.CPP, accessed August 18, 2025, https://www.behaviortree.dev/docs/intro/
3. ROS 2 Documentation: Humble documentation, accessed August 18, 2025, https://docs.ros.org/en/humble/index.html
4. PlanSys2: A Planning System Framework for ROS2 - arXiv, accessed August 18, 2025, https://arxiv.org/pdf/2107.00376
5. ros2_planning_system - ROS Index, accessed August 18, 2025, https://index.ros.org/r/ros2_planning_system/
6. PlanSys2/ros2_planning_system: This repo contains a PDDL-based planning system for ROS2. - GitHub, accessed August 18, 2025, https://github.com/PlanSys2/ros2_planning_system
7. BehaviorTree.ROS2/README.md at humble · BehaviorTree/BehaviorTree.ROS2 · GitHub, accessed August 18, 2025, https://github.com/BehaviorTree/BehaviorTree.ROS2/blob/humble/README.md
8. Integration with ROS2 | BehaviorTree.CPP, accessed August 18, 2025, https://www.behaviortree.dev/docs/ros2_integration/
9. Implement actions as Behavior Trees — ROS2 Planning System 2 ..., accessed August 18, 2025, https://plansys2.github.io/tutorials/docs/bt_actions.html
10. Optimized Execution of PDDL Plans using Behavior Trees, accessed August 18, 2025, https://archive.illc.uva.nl/AAMAS-2021/pdfs/p1596.pdf
11. What are some examples of imperative vs. declarative programming? - Quora, accessed August 18, 2025, https://www.quora.com/What-are-some-examples-of-imperative-vs-declarative-programming
12. Can you explain the differences between imperative and declarative programming languages? What are the advantages and disadvantages of each? How do they differ from object-oriented programming languages? - Quora, accessed August 18, 2025, https://www.quora.com/Can-you-explain-the-differences-between-imperative-and-declarative-programming-languages-What-are-the-advantages-and-disadvantages-of-each-How-do-they-differ-from-object-oriented-programming-languages
13. Declarative vs. Imperative Programming: 4 Key Differences | Codefresh, accessed August 18, 2025, https://codefresh.io/learn/infrastructure-as-code/declarative-vs-imperative-programming-4-key-differences/
14. How do you conceptualize the difference between 'Imperative' vs 'Declarative' Programming? : r/compsci - Reddit, accessed August 18, 2025, https://www.reddit.com/r/compsci/comments/17wv25o/how_do_you_conceptualize_the_difference_between/
15. Imperative vs Declarative Programming - DEV Community, accessed August 18, 2025, https://dev.to/askharley/imperative-vs-declarative-programming-4eba
16. Planning Domain Definition Language - Wikipedia, accessed August 18, 2025, https://en.wikipedia.org/wiki/Planning_Domain_Definition_Language
17. PlanSys2 design — ROS2 Planning System 2 1.0.0 documentation, accessed August 18, 2025, https://plansys2.github.io/design/index.html
18. ros2_planning_system/plansys2_docs/tutorials/tut_1_terminal.md at rolling - GitHub, accessed August 18, 2025, https://github.com/PlanSys2/ros2_planning_system/blob/master/plansys2_docs/tutorials/tut_1_terminal.md
19. Integrating AI Planning Semantics into SysML System Models for Automated PDDL File GenerationThis research paper [project iMOD and LaiLa] is funded by dtec.bw - arXiv, accessed August 18, 2025, https://arxiv.org/html/2506.06714v1
20. plansys2_planner 2.0.9 documentation, accessed August 18, 2025, https://docs.ros.org/en/humble/p/plansys2_planner/
21. plansys2_popf_plan_solver 2.0.9 documentation, accessed August 18, 2025, https://docs.ros.org/en/ros2_packages/humble/api/plansys2_popf_plan_solver/index.html
22. Introduction to BTs | BehaviorTree.CPP, accessed August 18, 2025, https://www.behaviortree.dev/docs/learn-the-basics/bt_basics/
23. Behavior Tree Tutorial - Community & Industry Discussion - Unreal Engine Forums, accessed August 18, 2025, https://forums.unrealengine.com/t/behavior-tree-tutorial/105
24. Your first Behavior Tree | BehaviorTree.CPP, accessed August 18, 2025, https://www.behaviortree.dev/docs/tutorial-basics/tutorial_01_first_tree/
25. Constructing Behavior Trees from Temporal Plans for Robotic Applications - arXiv, accessed August 18, 2025, https://arxiv.org/html/2406.17379v1
26. ROS Package: plansys2_planner, accessed August 18, 2025, https://index.ros.org/p/plansys2_planner/
27. ROS Package: plansys2_problem_expert, accessed August 18, 2025, https://index.ros.org/p/plansys2_problem_expert/
28. plansys2_executor 2.0.18 documentation, accessed August 18, 2025, https://docs.ros.org/en/jazzy/p/plansys2_executor/
29. The first planning package — ROS2 Planning System 2 1.0.0 documentation, accessed August 18, 2025, https://plansys2.github.io/tutorials/docs/simple_example.html
30. PlanSys2: A Planning System Framework for ROS2 - ResearchGate, accessed August 18, 2025, https://www.researchgate.net/publication/353055585_PlanSys2_A_Planning_System_Framework_for_ROS2
31. [2101.01964] Optimized Execution of PDDL Plans using Behavior Trees - arXiv, accessed August 18, 2025, https://arxiv.org/abs/2101.01964
32. Optimized Execution of PDDL Plans using Behavior Trees - arXiv, accessed August 18, 2025, https://arxiv.org/pdf/2101.01964
33. behaviortree_cpp_v3 3.8.7 documentation, accessed August 18, 2025, https://docs.ros.org/en/iron/p/behaviortree_cpp_v3/
34. BehaviorTree/BehaviorTree.CPP: Behavior Trees Library in C++. Batteries included. - GitHub, accessed August 18, 2025, https://github.com/BehaviorTree/BehaviorTree.CPP
35. nav2_behavior_tree: Humble 1.1.18 documentation, accessed August 18, 2025, https://docs.ros.org/en/humble/p/nav2_behavior_tree/
36. README — nav2_behavior_tree: Humble 1.1.18 documentation - the official ROS docs, accessed August 18, 2025, https://docs.ros.org/en/humble/p/nav2_behavior_tree/__README.html
37. behaviortree_cpp_v3: Jazzy 3.8.7 documentation, accessed August 18, 2025, https://docs.ros.org/en/jazzy/p/behaviortree_cpp_v3/
38. ROS Package: behaviortree_cpp - ROS Index, accessed August 18, 2025, https://index.ros.org/p/behaviortree_cpp/
39. Main Concepts | BehaviorTree.CPP, accessed August 18, 2025, https://www.behaviortree.dev/docs/4.0.2/learn-the-basics/main_concepts/
40. Behavior Trees in Action:A Study of Robotics Applications, accessed August 18, 2025, https://pure.itu.dk/files/85635298/2020_sle_razan_behaviortrees.pdf
41. Groot Tutorials — Nav2 1.0.0 documentation, accessed August 18, 2025, https://docs.nav2.org/tutorials/docs/using_groot.html
42. Groot - Interacting with Behavior Trees — Nav2 1.0.0 documentation, accessed August 18, 2025, https://docs.nav2.org/tutorials/docs/groot.html
43. Groot | BehaviorTree.CPP, accessed August 18, 2025, https://www.behaviortree.dev/groot/
44. Smart Turtle Powered by Behavior Trees, ROS2 & Groot! - ROS General, accessed August 18, 2025, https://discourse.openrobotics.org/t/smart-turtle-powered-by-behavior-trees-ros2-groot/43449
45. plansys2_bt_actions 2.0.18 documentation, accessed August 18, 2025, https://docs.ros.org/en/jazzy/p/plansys2_bt_actions/
46. berkeleyauv/behavior_tree: ROS2 package for managing the AUV's behavior using BTCPP behavior trees. - GitHub, accessed August 18, 2025, https://github.com/berkeleyauv/behavior_tree
47. ROS Package: plansys2_bt_actions, accessed August 18, 2025, https://index.ros.org/p/plansys2_bt_actions/
48. plansys2_terminal - ROS Package, accessed August 18, 2025, https://index.ros.org/p/plansys2_terminal/
49. PDDL - Visual Studio Marketplace, accessed August 18, 2025, https://marketplace.visualstudio.com/items?itemName=jan-dolejsi.pddl
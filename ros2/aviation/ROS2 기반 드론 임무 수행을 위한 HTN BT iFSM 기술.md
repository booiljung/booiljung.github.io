---
layout: page
title: ROS2 기반 드론 임무 수행을 위한 HTN, BT, iFSM 기술
permalink: /ros2/aviation/ROS2 기반 드론 임무 수행을 위한 HTN BT iFSM 기술
---


현대의 드론(UAV) 자율 임무는 단순한 경로점 비행을 넘어, 동적인 환경 변화에 실시간으로 대응하고, 복잡한 작업을 순차적으로 수행하며, 예기치 않은 예외 상황을 스스로 처리하는 고도의 지능을 요구한다.1 물류 배송, 시설 점검, 수색 및 구조와 같은 다양한 시나리오에서 드론은 예측 불가능한 변수들 속에서 임무를 완수해야 하는 도전에 직면해 있다.4 이러한 복잡성을 효과적으로 관리하기 위해서는 로봇의 행동을 결정하는 '두뇌', 즉 체계적인 소프트웨어 아키텍처가 필수적이다.

자율 시스템의 의사결정 아키텍처는 크게 두 가지 핵심 축으로 나눌 수 있다. 첫째는 **심의적 계획(Deliberative Planning)**으로, 장기적인 목표를 달성하기 위해 필요한 행동의 순서를 사전에 깊이 숙고하여 생성하는 능력이다. 이는 "무엇을, 어떤 순서로 할 것인가?"라는 전략적 질문에 대한 해답을 제공한다.6 둘째는 **반응형 실행(Reactive Execution)**으로, 수립된 계획을 실행하는 도중 발생하는 예기치 않은 상황에 즉각적으로 대응하는 능력이다. 이는 "지금 당장 어떻게 행동할 것인가?"라는 전술적 질문을 해결한다.6

본 보고서는 드론의 자율 임무 제어를 위한 세 가지 핵심 인공지능(AI) 패러다임-심의적 계획을 대표하는 **계층적 임무 네트워크(Hierarchical Task Network, HTN)**, 반응형 실행의 표준으로 자리 잡은 **행동 트리(Behavior Tree, BT)**, 그리고 고전적 제어 모델의 지능적 진화 형태인 **지능형 유한 상태 머신(intelligent Finite State Machine, iFSM)**-을 심층적으로 분석한다. 각 기술의 이론적 기반과 핵심 원리를 탐구하고, Robot Operating System 2 (ROS2) 생태계 내에서의 구체적인 구현 방법론을 제시하며, 나아가 이들을 통합한 하이브리드 아키텍처의 설계 방안까지 포괄적으로 고찰할 것이다. 이를 통해, 실제 드론 시스템 개발 현장에서 마주하는 복잡한 문제들을 해결하고, 견고하며 지능적인 자율 임무 시스템을 구축하는 데 필요한 실질적인 아키텍처 설계 지침을 제공하는 것을 궁극적인 목표로 한다.




계층적 임무 네트워크(HTN)는 복잡한 문제를 인간이 사고하는 방식과 유사하게 해결하는 AI 계획 기법이다. 즉, '패키지 배송'과 같이 추상적이고 높은 수준의 목표가 주어지면, 이를 '목적지까지 비행', '패키지 투하', '기지로 복귀'와 같이 더 작고 구체적인 하위 작업들로 계층적으로 분해(decomposition)하여 최종적인 행동 계획을 수립한다.8

HTN의 핵심 구성 요소는 **작업(Task)**과 **메소드(Method)**다. 작업은 그 성격에 따라 두 가지로 분류된다.

- **복합 작업 (Compound Task):** '패키지 배송'이나 '건물 정찰'과 같이 그 자체로는 직접 실행할 수 없으며, 추가적인 하위 작업으로 분해되어야 하는 추상적인 목표를 의미한다.8
- **원시 작업 (Primitive Task):** '모터 RPM 설정', '카메라 셔터 누르기', '그리퍼 열기'와 같이 시스템이 더 이상 분해하지 않고 직접 실행할 수 있는 가장 작은 단위의 행동이다. HTN 플래너가 최종적으로 생성하는 계획은 바로 이 원시 작업들의 실행 가능한 순차열(sequence)이다.8

**메소드(Method)**는 하나의 복합 작업을 특정 조건 하에서 어떻게 하위 작업들로 분해할 것인지를 정의하는 '방법' 또는 '레시피'다.10 각 메소드는 자신을 적용할 수 있는 **선행 조건(precondition)**과, 그 결과로 생성되는 **하위 작업 네트워크(subtask network)**로 구성된다. 예를 들어, '목표 지점 이동'이라는 복합 작업은 '비행 경로 이용' 메소드와 '지상 주행 경로 이용' 메소드를 가질 수 있다. '비행 경로 이용' 메소드의 선행 조건이 `(날씨 == 맑음)`이라면, 이 메소드는 날씨가 맑을 때만 선택될 수 있다.

HTN의 계획 수립 과정은 초기 상태(initial state)와 달성하고자 하는 최상위 작업 네트워크가 주어졌을 때 시작된다. 플래너는 깊이 우선 탐색(Depth-First Search)과 유사한 방식으로, 최상위 복합 작업을 시작으로 재귀적으로 메소드를 적용하며 분해를 진행한다.8 만약 특정 분해 경로에서 유효한 메소드를 찾지 못하거나 원시 작업의 선행 조건이 충족되지 않으면, 플래너는 **백트래킹(backtracking)**을 통해 이전 분기점으로 돌아가 다른 메소드를 시도한다.14 이 과정은 모든 복합 작업이 원시 작업의 실행 가능한 시퀀스로 완전히 변환될 때까지 계속된다.

이러한 접근 방식은 순수한 절차적 코딩과 선언적 계획의 중간 지점에 위치한다. 개발자는 "이런 상황에서는 대략 이런 절차를 따르라"는 절차적 지식을 '메소드'라는 형태로 시스템에 주입한다.17 그러면 플래너는 주어진 현재 상태에 맞춰 그 절차 내의 구체적인 행동 순서나 매개변수를 '발견'해낸다.18 이는 개발자의 도메인 지식을 활용하여 탐색 공간을 효율적으로 제한하면서도, 상황에 맞는 유연한 계획을 생성할 수 있게 하는 HTN의 핵심적인 특징이다. 드론의 정찰 임무처럼 표준 운영 절차(SOP)가 존재하는 도메인에서 특히 강력한 성능을 발휘하는 이유가 바로 여기에 있다.


HTN 계획 문제는 수학적으로 튜플 $(s_0, T_0, D)$로 정의할 수 있다. 여기서 $s_0$는 초기 상태(initial state)를 나타내는 술어(predicate)의 집합, $T_0$는 초기에 주어진 작업 네트워크(initial task network), 그리고 $D$는 도메인 지식으로, 원시 작업, 복합 작업, 그리고 메소드의 집합을 포함한다. 플래너의 목표는 이 튜플이 주어졌을 때, $s_0$에서 실행 가능한 원시 작업의 시퀀스인 최종 계획 $\pi$를 찾는 것이다.

메소드 $m$은 형식적으로 튜플 $(name(m), task(m), pre(m), network(m))$으로 표현된다.13

- $name(m)$: 메소드의 고유 이름
- $task(m)$: 이 메소드가 분해하는 대상 복합 작업
- $pre(m)$: 메소드가 적용되기 위해 현재 상태에서 반드시 참이어야 하는 선행 조건들의 집합
- $network(m)$: 메소드가 성공적으로 적용되었을 때 생성되는 하위 작업들의 부분 순서 그래프(partially ordered graph)

HTN의 표현력은 고전적인 STRIPS 계획 형식보다 엄격하게 더 높다($HTN > STRIPS$). STRIPS가 목표 상태(goal state)라는 '무엇(what)'을 달성할 것인지만을 명시하는 반면, HTN은 메소드를 통해 '어떻게(how)' 그 목표를 달성할 것인지에 대한 절차적, 제어적 지식을 도메인에 직접 인코딩할 수 있기 때문이다. 이로 인해 HTN은 훨씬 더 복잡하고 현실적인 문제들을 모델링할 수 있다.

하지만 이러한 강력한 표현력에는 대가가 따른다. 일반적인 HTN 계획 문제, 특히 메소드가 재귀적으로 자신을 호출할 수 있는 경우에는 계획의 존재 여부를 판별하는 것이 튜링 기계의 정지 문제와 동등한 수준으로 어려워져, 알고리즘적으로 해결 불가능(undecidable)하다.8 따라서 실제 HTN 플래너들은 대부분 메소드의 재귀를 금지하거나 분해 깊이를 제한하는 등의 구문적 제약을 가하여 문제를 결정 가능(decidable)한 범위 내로 한정시킨다.16


HTN 계획 패러다임은 풍부한 도메인 지식을 활용하는 데서 비롯되는 명확한 장점과 그에 따른 본질적인 한계를 동시에 지닌다.

**장점:**

- **효율성 및 확장성:** HTN의 가장 큰 장점은 도메인 전문가의 지식을 활용하여 거대한 탐색 공간(search space)을 효과적으로 가지치기(pruning)할 수 있다는 점이다. 이는 복잡한 실제 문제에서 고전 플래너가 겪는 조합 폭발(combinatorial explosion) 문제를 회피하고, 매우 빠른 속도로 실용적인 계획을 생성하게 해준다.10
- **가독성 및 유지보수성:** HTN의 계층적 구조는 인간이 문제를 인식하고 해결하는 방식과 매우 유사하다. 이 덕분에 개발자나 도메인 전문가가 계획 과정을 직관적으로 이해하고, 도메인 지식을 수정하거나 확장하기가 용이하다.10
- **표현력:** 단순히 목표 상태를 만족시키는 것을 넘어, 특정 절차나 제약 조건을 따르는 계획을 생성하도록 강제할 수 있다. 예를 들어, "A 작업을 수행한 후에는 반드시 B 작업을 수행해야 한다"와 같은 절차적 제약을 자연스럽게 모델링할 수 있다.15

**한계:**

- **도메인 지식 의존성:** HTN의 성능은 전적으로 사전에 제공되는 도메인 지식의 질과 양에 달려있다. 고품질의, 잘 구조화된 메소드를 작성하는 것은 상당한 전문성과 엔지니어링 비용을 요구하는 어려운 작업이다. 잘못 설계된 도메인 지식은 오히려 계획 생성을 방해하거나 비효율적인 계획을 만들 수 있다.20
- **최적성 보장 부재:** HTN 플래너는 기본적으로 '실행 가능한(feasible)' 계획을 찾는 데 중점을 둔다. 깊이 우선 탐색 방식으로 첫 번째로 발견된 해결책을 반환하는 경우가 많으므로, 그 계획이 이동 거리나 에너지 소모 같은 비용 측면에서 최적이라는 보장은 없다.16
- **반응성 부족:** HTN은 본질적으로 오프라인(off-line) 심의 시스템이다. 계획은 실행 전에 미리 전체가 생성된다. 따라서 계획을 실행하는 도중에 예상치 못한 환경 변화(예: 갑작스러운 장애물 출현)가 발생하면, 기존 계획은 무용지물이 될 수 있다. 이 경우, 시스템은 일반적으로 실행을 멈추고 현재 상태를 기반으로 처음부터 다시 계획(re-planning)을 수립해야 하므로, 실시간 대응 능력이 떨어진다.22


행동 트리(BT)는 원래 비디오 게임 AI에서 유한 상태 머신(FSM)의 복잡성 문제를 해결하기 위해 등장했으며, 현재는 로보틱스 분야에서 반응형 행동 제어를 위한 표준 아키텍처로 널리 사용되고 있다.23 BT의 핵심 철학은 시스템의 '상태(State)'가 아닌 '작업(Task)'을 기본 구성 요소로 삼아, 복잡한 행동 로직을 모듈화하고 계층적으로 구성하는 것이다.


BT의 동작은 **'틱(Tick)'** 이라는 주기적인 신호에 의해 구동된다. 루트(root) 노드에서 일정한 주파수(예: 10Hz, 30Hz)로 틱 신호가 발생하면, 이 신호는 트리를 따라 자식 노드로 전파된다. 틱을 받은 노드는 자신의 로직을 실행하고, 그 결과를 세 가지 상태 중 하나로 부모 노드에게 반환한다.23

- `SUCCESS`: 작업이 성공적으로 완료되었음을 의미한다.
- `FAILURE`: 작업이 실패했음을 의미한다.
- `RUNNING`: 작업이 아직 진행 중이며, 다음 틱에서도 계속해서 실행되어야 함을 의미한다. 이 상태 덕분에 BT는 비동기적인 장기 실행 작업을 자연스럽게 처리할 수 있다.

BT는 목적에 따라 여러 유형의 노드로 구성된다.

- **제어 흐름 노드 (Control Flow Node):** 자식 노드들의 실행 순서와 방식을 결정하며, BT의 논리적 흐름을 제어한다.
  - **Sequence (?):** 자식 노드들을 왼쪽에서 오른쪽 순서로 실행한다. 실행 중인 자식 노드가 `FAILURE`를 반환하면, 즉시 실행을 중단하고 부모에게 `FAILURE`를 반환한다. 모든 자식 노드가 `SUCCESS`를 반환해야만 부모에게 `SUCCESS`를 반환한다. 이는 논리적 AND 연산과 유사하다.23
  - **Selector / Fallback (?):** 자식 노드들을 왼쪽에서 오른쪽 순서로 실행한다. 실행 중인 자식 노드가 `SUCCESS`를 반환하면, 즉시 실행을 중단하고 부모에게 `SUCCESS`를 반환한다. 모든 자식 노드가 `FAILURE`를 반환해야만 부모에게 `FAILURE`를 반환한다. 이는 논리적 OR 연산과 유사하며, 우선순위에 따른 행동 선택이나 예외 처리에 주로 사용된다.23
- **실행 노드 (Execution Node / Leaf Node):** 트리의 가장 말단에 위치하며, 실제 작업을 수행하거나 조건을 검사한다.
  - **Action:** 로봇의 구체적인 행동(예: '경로점으로 이동', '사진 촬영')을 수행하는 노드. ROS 액션 서버를 호출하거나 특정 함수를 실행하는 등의 작업을 담당한다. 실행에 시간이 걸리는 작업의 경우 `RUNNING`을 반환할 수 있다.24
  - **Condition:** 특정 조건(예: '배터리 잔량이 20% 이상인가?')을 검사하는 노드. `RUNNING` 상태를 반환하지 않고, 즉시 `SUCCESS` 또는 `FAILURE`를 반환해야 한다.24
- **데코레이터 노드 (Decorator Node):** 단 하나의 자식 노드만을 가지며, 그 자식 노드의 행동을 수정하거나 실행 흐름을 제어하는 특별한 노드다.
  - `Inverter`: 자식 노드의 결과(`SUCCESS` 또는 `FAILURE`)를 반대로 뒤집는다.
  - `RetryUntilSuccessful`: 자식 노드가 `FAILURE`를 반환할 경우, 지정된 횟수(N)만큼 재시도한다.
  - `Timeout`: 자식 노드가 지정된 시간 내에 완료되지 않으면 강제로 중단시키고 `FAILURE`를 반환한다.24


BT는 반응형 시스템 설계에 있어 수많은 장점을 제공하지만, 동시에 명확한 한계도 가지고 있다.

**장점:**

- **모듈성 및 재사용성:** 각 서브트리(subtree)는 하나의 독립적인 행동 단위를 캡슐화한다. '물체 집기'나 '장애물 회피'와 같은 서브트리는 한번 잘 설계해두면, 다른 여러 BT에서 그대로 가져다 재사용할 수 있다. 이는 복잡한 시스템의 개발 속도를 높이고 유지보수를 용이하게 한다.23
- **반응성:** 매 틱마다 트리의 루트부터 상태를 재평가하는 구조 덕분에, 환경 변화에 매우 신속하게 반응할 수 있다. 예를 들어, 순찰 임무를 수행하던 드론이 갑자기 적을 발견하면, 우선순위가 더 높은(트리의 왼쪽에 위치한) '공격' 서브트리가 즉시 활성화될 수 있다.32
- **가독성 및 직관성:** 행동의 흐름이 계층적이고 시각적인 트리 구조로 표현되기 때문에, 프로그래머가 아닌 기획자나 디자이너도 AI의 의사결정 로직을 쉽게 이해하고 디버깅할 수 있다.23
- **확장성:** 새로운 행동이나 조건을 추가하는 것이 매우 용이하다. 기존 로직을 대대적으로 수정할 필요 없이, 트리의 적절한 위치에 새로운 노드나 서브트리를 추가하는 것만으로 시스템의 기능을 확장할 수 있다.32

**한계:**

- **계획 능력 부재:** BT는 본질적으로 사전에 잘 설계된 '실행 스크립트'다. 주어진 상황에 따라 어떤 행동을 할지 결정할 수는 있지만, 장기적인 목표를 달성하기 위해 새로운 행동 순서를 스스로 '발견'하거나 '계획'하지는 못한다. 모든 가능한 시나리오와 그에 대한 대응책은 개발자가 미리 트리에 모두 설계해 넣어야 한다.15
- **복잡성 관리:** 임무가 매우 복잡해지면 BT의 깊이와 너비가 과도하게 커져, 전체 구조를 한눈에 파악하고 관리하기 어려워질 수 있다. 이는 BT의 장점인 가독성을 해칠 수 있다.31
- **최적화의 어려움:** BT는 우선순위에 기반하여 행동을 결정하기 때문에, 선택된 행동이 자원 소모나 시간 효율성 측면에서 전역적으로 최적이라는 보장을 하기 어렵다.

특히, BT의 `Fallback` 노드는 소프트웨어 공학의 예외 처리(Exception Handling) 메커니즘과 본질적으로 동일한 구조적 패턴을 제공한다. 프로그래밍에서 `try-catch` 구문이 주된 로직을 시도(`try`)하고 실패 시 대체 로직(`catch`)을 실행하는 것처럼, `Fallback` 노드는 가장 우선순위가 높은 자식(이상적인 행동)을 먼저 시도하고, 그것이 `FAILURE`를 반환하면 차선책인 다음 자식(복구 행동)을 실행한다.24 이 패턴은 "A를 시도하라. 실패하면 B를, 그것도 실패하면 C를 시도하라"와 같은 견고한(robust) 복구 로직을 매우 직관적으로 구현하게 해준다. 드론 임무에서 '주 임무 수행' 실패 시 '안전 지점 복귀', 이마저도 실패하면 '비상 착륙'으로 이어지는 연속적인 안전 절차를 구현하는 데 이 `Fallback` 노드는 핵심적인 역할을 수행한다.24



유한 상태 머신(FSM)은 시스템이 가질 수 있는 유한한 개수의 **'상태(State)'**와, 특정 입력(Input)이나 이벤트(Event)에 의해 발생하는 상태 간의 **'전이(Transition)'**로 시스템의 동작을 모델링하는 고전적인 계산 모델이다.37 단순하고 명확하여 초기 AI나 제어 시스템에서 널리 사용되었지만, 복잡한 시스템을 모델링하는 데에는 명백한 한계를 드러냈다.

**FSM의 근본적인 한계:**

- **상태 폭발 (State Explosion):** 시스템의 복잡도가 증가함에 따라 필요한 상태의 수가 기하급수적으로 늘어나는 문제다. 예를 들어, 4개의 독립적인 on/off 스위치를 모델링하려면 $2^4 = 16$개의 상태가 필요하다. 드론과 같이 수많은 변수(위치, 속도, 배터리, 임무 단계 등)를 가진 시스템에서는 상태의 수가 감당할 수 없을 정도로 커지게 된다. 이는 상태 간의 전이 관계를 스파게티 코드처럼 복잡하게 얽히게 만들어 시스템의 이해, 디버깅, 유지보수를 극도로 어렵게 만든다.39
- **확장성 및 재사용성 부족:** FSM에 새로운 상태를 하나 추가하려면, 그 상태와 관련된 모든 기존 상태들의 전이를 일일이 수정해야 할 수 있다. 또한, 각 상태는 특정 FSM의 맥락에 강하게 결합(tightly coupled)되어 있어, 다른 FSM에서 특정 상태 로직을 재사용하기가 매우 어렵다.33

이러한 문제를 해결하기 위해 **계층적 FSM (Hierarchical FSM, HFSM)**이 등장했다. HFSM은 상태 안에 또 다른 FSM(sub-state machine)을 중첩시키는 계층적 구조를 도입한다.33 이를 통해 관련된 상태들을 하나의 상위 상태(super-state)로 묶어 추상화할 수 있다.

예를 들어, '비행 중'이라는 상위 상태 안에 '경로점 비행', '호버링', '장애물 회피'와 같은 여러 하위 상태들을 포함시킬 수 있다. 이때 '배터리 부족'이라는 치명적인 이벤트가 발생하면, 드론이 현재 어떤 하위 비행 상태에 있든지 상관없이, '비행 중' 상위 상태에 연결된 단 하나의 전이를 통해 '기지 복귀' 상태로 즉시 전환될 수 있다. 이는 수많은 중복된 전이를 제거하여 상태 폭발 문제를 상당 부분 완화하고, 시스템의 모듈성과 가독성을 향상시킨다.


HFSM은 상태 폭발 문제를 완화하는 데 기여했지만, 근본적인 해결책을 제시하지는 못했다. 상태의 수가 여전히 많아질 수 있으며, 계층이 깊어질수록 복잡성은 다시 증가한다. 행동 트리(BT)는 상태와 전이 로직을 분리하여 트리 구조로 중앙에서 관리함으로써 이 문제를 보다 근본적으로 해결했다.44

그러나 FSM 패러다임 자체도 진화하고 있다. 본 보고서에서 **'지능형 FSM (iFSM)'**은 단순히 HFSM을 넘어, 외부의 지능적인 모듈과 결합하여 상태나 전이를 동적으로 수정하거나 생성할 수 있는 능력을 갖춘 FSM을 지칭하는 개념으로 사용한다.

- **Self-Adjusting FSM:** 환경 변화에 대응하여 FSM의 구조 자체(상태, 전이)를 동적으로 추가, 제거, 수정하는 별도의 모듈을 가진 FSM이다. 이를 통해 사전 정의된 정적인 구조를 넘어, 학습과 적응을 통해 로봇의 행동을 근본적으로 변화시킬 수 있다.45
- **LLM 기반 FSM 생성:** 최근에는 대규모 언어 모델(LLM)을 활용하여 "드론이 순찰하다가 침입자를 발견하면 추적 모드로 전환하고, 놓치면 다시 순찰 모드로 돌아가라"와 같은 자연어 지시로부터 FSM의 상태와 전이를 자동으로 생성하거나 수정하는 연구가 활발히 진행되고 있다.46 이는 FSM 설계 및 유지보수에 드는 개발자의 부담을 획기적으로 줄여줄 수 있는 새로운 접근법으로, FSM이 다시금 복잡한 AI 시스템의 제어 아키텍처로서 주목받을 가능성을 시사한다.



세 가지 패러다임은 드론의 의사결정 문제를 각기 다른 철학적 관점에서 접근한다. 이들의 핵심적인 차이점을 이해하는 것은 특정 임무에 가장 적합한 아키텍처를 선택하는 데 매우 중요하다.


- **HTN (심의적/계획 중심):** HTN의 핵심은 "어떻게 목표를 달성할 것인가?"라는 질문에 대한 최적의 또는 실행 가능한 '행동 순서'를 미리 찾아내는 것이다. 계획 생성 과정 자체가 지능의 핵심이며, 실행은 단지 사전에 생성된 청사진을 따르는 부차적인 과정으로 간주된다. 이는 장기적인 관점에서 자원의 효율적 사용과 목표 달성을 보장하는 데 초점을 맞춘다.6
- **BT (반응형/실행 중심):** BT의 철학은 "현재 상태에서 무엇을 해야 하는가?"라는 질문에 대한 즉각적인 '행동 결정'에 있다. 매 순간(tick)마다 우선순위에 따라 가장 적절한 행동을 선택하며, 장기적인 계획보다는 현재 환경에 대한 반응성과 견고함이 더 중요하다. 계획은 사전에 개발자에 의해 트리 구조 자체에 하드코딩된 것으로 볼 수 있다.6
- **iFSM (상태 중심):** iFSM은 "현재 시스템은 어떤 '상태'에 있으며, 어떤 이벤트가 발생하면 다음 '상태'로 가야 하는가?"라는 질문에 집중한다. 시스템의 동작을 명시적인 상태와 그 상태들 사이의 전이 규칙으로 모델링한다. 지능은 상태 전이 로직의 정교함이나, 외부 요인에 의해 이 로직이 어떻게 동적으로 변화하는가에 달려 있다.37


- **유연성(Flexibility):** 환경 변화에 대한 적응 능력 면에서는 **BT**가 단연 가장 뛰어나다. 매 틱마다 전체 상황을 재평가하고, `Fallback` 노드를 통해 다양한 대체 행동을 쉽게 추가할 수 있어 예측하지 못한 상황에 유연하게 대처할 수 있다.32 반면 

  **HTN**은 계획이 현재 상황과 맞지 않게 되면 전체 재계획(re-planning)이 필요하므로 반응이 느리고 경직될 수 있다. **FSM**은 모든 가능한 전이를 사전에 명시해야 하므로 가장 경직된 구조를 가진다.10

- **확장성(Scalability):** **BT**와 **HTN**은 모두 모듈식 설계가 가능하여 확장성이 우수하다. BT는 잘 정의된 서브트리를 재사용하고, HTN은 작업과 메소드 라이브러리를 재사용하여 시스템을 확장한다.10 반면, 

  **FSM**은 '상태 폭발' 문제로 인해 시스템이 복잡해질수록 확장성이 급격히 저하된다.39

- **유지보수성(Maintainability):** **BT**의 시각적이고 모듈적인 특성은 AI의 행동 로직을 이해하고 수정하기 쉽게 만들어 유지보수성을 높인다.23

  **HTN**은 도메인 지식이 잘 구조화되어 있다면 유지보수가 용이하지만, 복잡한 메소드 간의 의존성이 얽히면 어려워질 수 있다. **FSM**은 스파게티처럼 얽힌 전이 관계 때문에 세 패러다임 중 유지보수가 가장 어렵다.39

이러한 특성들을 종합적으로 비교하면 다음 표와 같다. 이 표는 시스템 아키텍트가 드론 시스템의 각 구성 요소에 대한 요구사항을 평가하고, 가장 적합한 패러다임 조합을 선택하는 데 도움을 줄 수 있다. 예를 들어, 고수준의 임무 전략은 심의적 능력이 뛰어난 HTN이, 저수준의 실시간 장애물 회피는 반응성이 높은 BT가, 드론의 비행 모드(수동, 자동, 귀환 등) 관리는 명확한 상태 정의가 가능한 FSM이 적합할 수 있다. 이는 단일 패러다임이 아닌, 하이브리드 아키텍처가 최적의 해결책인 경우가 많음을 시사한다.

| 특징 (Feature)         | 계층적 임무 네트워크 (HTN)    | 행동 트리 (BT)             | 지능형 FSM (iFSM/HFSM)       |
| ---------------------- | ----------------------------- | -------------------------- | ---------------------------- |
| **주요 초점**          | 심의적 계획 (Deliberative)    | 반응형 실행 (Reactive)     | 상태 기반 제어 (State-based) |
| **계획 범위**          | 장기적 (Long-term)            | 단기적/즉각적 (Short-term) | 상태에 따라 가변적           |
| **반응성**             | 낮음 (재계획 필요)            | 매우 높음 (매 틱 재평가)   | 중간 (이벤트 기반)           |
| **도메인 지식**        | 매우 많이 필요 (메소드)       | 개발자가 트리에 직접 코딩  | 상태 전이 로직으로 코딩      |
| **최적성**             | 보장 안 됨 (실행 가능성 우선) | 보장 안 됨 (우선순위 기반) | 보장 안 됨                   |
| **모듈성**             | 높음 (작업/메소드 단위)       | 매우 높음 (서브트리 단위)  | 중간 (HFSM으로 개선)         |
| **가독성**             | 중간 (인간의 사고와 유사)     | 매우 높음 (시각적, 직관적) | 낮음 (상태 폭발 시 저하)     |
| **확장성**             | 높음                          | 매우 높음                  | 낮음 (상태 폭발 문제)        |
| **결함 허용 메커니즘** | 재계획 (Re-planning)          | Fallback 노드, 데코레이터  | 예외 상태로의 전이           |


각 패러다임의 장점은 취하고 단점은 보완하기 위해, 이들을 결합한 하이브리드 아키텍처가 복잡한 자율 시스템의 실질적인 해결책으로 부상하고 있다. 특히 HTN의 계획 능력과 BT의 반응형 실행 능력을 결합하는 것은 매우 자연스럽고 강력한 조합이다.22


이 통합 모델은 로봇의 의사결정을 '전략'과 '전술'의 두 계층으로 명확히 분리한다.

- **워크플로우:**
  1. **계획 단계 (전략 계층 - HTN):** 먼저, HTN 플래너가 고수준의 임무 목표(예: '지역 A를 정찰하고 발견된 목표물을 지역 B로 운송하라')를 입력받는다. 플래너는 도메인 지식(메소드)을 활용하여 이 추상적인 목표를 저수준의 원시 작업 시퀀스(예: `(fly_to A)`, `(scan_area A)`, `(fly_to target)`, `(pickup target)`, `(fly_to B)`, `(dropoff target)`)로 구성된 계획을 생성한다. 이 단계는 임무 전체의 큰 그림을 그리는 전략적 의사결정에 해당한다.22
  2. **실행 단계 (전술 계층 - BT):** HTN이 생성한 계획의 각 원시 작업(예: `(fly_to A)`)은 해당 작업을 실제로 수행하도록 미리 설계된 특정 BT를 트리거하는 역할을 한다. 이 BT는 실제 비행 제어, 실시간 장애물 회피, 통신 상태 모니터링 등 구체적이고 전술적인 실행을 담당한다. 즉, `fly_to` BT는 단순히 목적지로 이동하는 것을 넘어, 비행 중 마주치는 다양한 예기치 않은 상황에 대처하는 로직을 포함하고 있다.22

이 모델은 "무엇을 할지(What)"는 HTN이 결정하고, "그것을 어떻게 할지(How)"는 BT가 결정하는 명확한 역할 분담을 통해, 시스템이 장기적인 목표를 향해 나아가면서도 단기적인 환경 변화에 유연하게 대처할 수 있도록 한다.


하이브리드 아키텍처의 진정한 강점은 실패 상황을 다루는 방식에 있다.

1. **전술적 복구 (BT 계층):** BT 실행 중 작업 실패(예: `fly_to` BT가 좁은 통로를 통과하지 못하고 `FAILURE`를 반환)가 발생하면, 먼저 BT 내부의 `Fallback` 구조를 통해 1차적인 복구가 시도된다. 예를 들어, '좁은 통로 통과' 행동이 실패하면 '우회 경로 탐색'이라는 대체 행동을 시도할 수 있다.24
2. **전략적 재계획 (HTN 계층):** 만약 BT 내의 모든 복구 시도가 실패하여 BT 자체가 최종적으로 `FAILURE`를 반환하면, 이 실패 정보는 상위의 계획 계층(HTN)으로 전파된다.22
3. 계획 계층은 이 실패를 인지하고, 현재 세계 상태(예: '통로 A 통과 불가')를 반영하여 새로운 계획을 생성(재계획)한다. 예를 들어, 원래 계획을 폐기하고 '지역 A 정찰' 목표를 달성하기 위한 완전히 다른 접근법(예: '드론 C에게 임무 인계')을 포함하는 새로운 계획을 수립할 수 있다.22

이처럼 실패를 다단계로 처리함으로써, 시스템은 사소한 문제에 대해서는 빠르고 효율적으로 현장에서 대응하고, 근본적인 계획 수정이 필요한 심각한 문제에 대해서는 심의적인 재계획을 통해 대응하는, 매우 견고하고 지능적인 결함 허용(fault-tolerant) 동작을 구현할 수 있다.



ROS2는 로봇 애플리케이션 개발을 위한 차세대 표준 프레임워크로, 모듈성, 확장성, 그리고 실시간 성능을 핵심 가치로 삼는다. 드론과 같은 자율 시스템을 구축하기 위해서는 ROS2의 통신 아키텍처와 실시간 설계 원칙에 대한 깊은 이해가 필수적이다.


ROS2 시스템은 독립적으로 실행되는 여러 개의 프로세스인 **'노드(Node)'**들의 집합으로 구성된다.49 각 노드는 특정 기능(예: 카메라 드라이버, 경로 계획, 모터 제어)을 담당하며, 이들 간의 데이터 교환은 DDS(Data Distribution Service)를 기반으로 한 표준화된 통신 메커니즘을 통해 이루어진다.

- **토픽 (Topics):** 발행/구독(Publish/Subscribe) 모델을 사용하는 비동기식 일대다(one-to-many) 통신 방식이다. 센서 데이터(`sensor_msgs/Image`, `sensor_msgs/LaserScan`)나 로봇의 상태(`tf2_msgs/TFMessage`)처럼 연속적으로 발생하는 데이터 스트림을 전송하는 데 적합하다.50
- **서비스 (Services):** 요청/응답(Request/Response) 모델을 사용하는 동기식 일대일(one-to-one) 통신 방식이다. 특정 작업을 요청하고 그 결과를 즉시 받아야 할 때 사용된다(예: '현재 지도 저장 요청').50
- **액션 (Actions):** 목표/피드백/결과(Goal/Feedback/Result) 모델을 사용하는 비동기식 일대일(one-to-one) 통신 방식이다. '경로점까지 이동'과 같이 실행에 긴 시간이 소요되고, 중간 진행 상황을 모니터링해야 하며, 중간에 취소(cancel)할 수도 있어야 하는 장기 실행 작업에 최적화되어 있다. 이는 BT나 FSM에서 비동기 작업을 처리하는 데 핵심적인 역할을 한다.50


드론 제어와 같이 밀리초 단위의 반응 속도가 중요한 미션 크리티컬 애플리케이션은 **실시간(Real-time)** 성능을 요구한다. 이는 단순히 '빠른' 것을 넘어, 정해진 시간(deadline) 내에 작업 완료를 '보장'하는 **결정성(determinism)**을 의미한다.52

ROS2는 실시간성을 염두에 두고 설계되었으며, 다음과 같은 원칙을 고려하여 시스템을 구축해야 한다.

- **QoS (Quality of Service) 설정:** DDS의 QoS 정책을 통해 통신의 신뢰성(Reliability), 내구성(Durability), 데드라인(Deadline) 등을 세밀하게 조정할 수 있다. 예를 들어, 중요한 제어 명령은 'Reliable' QoS로 설정하여 반드시 전달되도록 보장하고, 고주파의 영상 데이터는 'Best Effort'로 설정하여 최신 데이터를 우선적으로 처리하도록 할 수 있다.54
- **실시간 안전(Real-time Safe) 프로그래밍:** 실시간 실행 경로(real-time code path)에서는 예측 불가능한 지연을 유발하는 작업을 피해야 한다. 여기에는 동적 메모리 할당/해제(`new`, `delete`), 파일 I/O, 무한정 블로킹될 수 있는 동기화 메커니즘 등이 포함된다.52
- **운영체제 및 스케줄링:** 실시간 성능을 극대화하기 위해 RT_PREEMPT 패치가 적용된 리눅스 커널을 사용하고, 중요한 노드의 스레드 우선순위와 스케줄링 정책(예: `SCHED_FIFO`)을 적절히 설정해야 한다.53


`BehaviorTree.CPP`는 C++17 기반의 고성능 BT 라이브러리로, 유연성과 속도, 쉬운 사용성 덕분에 ROS2 생태계에서 사실상의 표준으로 자리 잡았다.56 이 라이브러리의 가장 큰 특징은 BT의 구조를 C++ 코드에 하드코딩하는 대신, XML 형식의 스크립트 파일로 정의하고 런타임에 동적으로 로드할 수 있다는 점이다. 이는 코드 재컴파일 없이 AI의 행동 로직을 쉽게 수정하고 테스트할 수 있게 해준다.


`BehaviorTree.ROS2` 패키지는 `BehaviorTree.CPP`를 ROS2의 통신 메커니즘과 매끄럽게 통합하기 위한 C++ 래퍼(wrapper) 클래스들을 제공한다.56 이 래퍼들은 ROS2의 비동기 통신(특히 액션과 서비스)을 BT의 `RUNNING` 상태와 자연스럽게 연결하여, 메인 틱(tick) 루프를 차단(blocking)하지 않으면서도 장기 실행 작업을 효과적으로 처리할 수 있게 해준다.56

아키텍처 설계 관점에서, BT를 실행하는 중앙 **조정자(Coordinator)** 또는 **'Task Planner'** 노드를 하나 두고, 로봇의 다른 모든 기능(예: 이동, 인식, 조작)은 독립적인 서비스 지향(service-oriented) 컴포넌트(주로 액션 서버나 서비스 서버)로 구현하는 것이 권장된다. BT는 이 컴포넌트들을 조율하여 전체 임무를 수행하는 역할을 맡는다.56


`BehaviorTree.ROS2`는 ROS2의 주요 통신 패턴을 BT의 실행 노드(leaf node)로 쉽게 변환할 수 있는 템플릿 클래스들을 제공한다.

- **`RosActionNode<ActionT>`:** ROS2 액션 클라이언트를 BT 노드로 구현하기 위한 핵심 템플릿 클래스다. 사용자는 이 클래스를 상속받아 자신만의 액션 노드를 정의한다.

  - `setGoal(Goal& goal)`: 틱을 받았을 때 액션 목표를 설정하고 서버로 전송하는 로직을 구현한다.

  - `onResult(const WrappedResult& result)`: 액션이 성공적으로 완료되었을 때 호출되며, 결과를 처리하고 `NodeStatus::SUCCESS`를 반환한다.

  - `onFailure(ActionNodeErrorCode error)`: 액션이 실패했을 때 호출되며, 실패 원인을 처리하고 `NodeStatus::FAILURE`를 반환한다.

  - onFeedback(const Feedback::ConstSharedPtr feedback): 액션 실행 중 서버로부터 피드백을 받을 때마다 호출된다.

    이 노드는 목표를 전송한 후 액션이 완료될 때까지 자동으로 NodeStatus::RUNNING 상태를 유지하므로, 개발자는 비동기 처리의 복잡함을 신경 쓸 필요가 없다.56

- **`RosServiceNode<ServiceT>`:** ROS2 서비스 클라이언트를 위한 유사한 래퍼 클래스다. 틱을 받으면 서비스를 요청하고, 응답이 올 때까지 `RUNNING` 상태를 유지한다.56

이러한 래퍼들을 사용하면, 예를 들어 드론의 `NavigateToPose` 액션을 호출하는 `FlyToWaypointAction` 노드를 단 몇 줄의 코드로 간결하게 구현할 수 있으며, 이는 BT의 모듈성과 재사용성을 극대화한다.


PlanSys2는 ROS1 시절의 대표적인 계획 시스템이었던 ROSPlan의 경험을 바탕으로, ROS2의 현대적인 설계 철학에 맞춰 완전히 재구현된 PDDL(Planning Domain Definition Language) 기반 계획 시스템이다.60 HTN과 마찬가지로 심의적 계획을 수행하지만, AI 계획 커뮤니티의 표준 언어인 PDDL을 사용함으로써 범용성과 호환성을 높였다.


PlanSys2의 전체 워크플로우는 계획과 실행을 명확히 분리하는 하이브리드 아키텍처의 전형을 보여준다.

1. **PDDL 모델링:** 먼저, 드론의 능력과 세상의 규칙을 PDDL로 기술한다. **도메인 파일(.pddl)**에는 드론이 수행할 수 있는 행동(actions)인 `fly`, `pickup`, `dropoff` 등과, 세상의 상태를 나타내는 술어(predicates)인 `(robot_at?r?wp)`, `(holding?r?obj)` 등을 정의한다.63
2. **문제 정의:** 현재 상황과 목표를 **문제 파일(.pddl)**에 기술한다. 여기에는 존재하는 객체(objects/instances)인 `r2d2`, `waypoint1`, 초기 상태(init)인 `(robot_at r2d2 base)`, 그리고 최종 목표(goal)인 `(robot_at r2d2 destination)` 등이 포함된다.63
3. **계획 생성:** `Planner` 노드가 `Domain Expert`와 `Problem Expert` 노드로부터 최신 도메인 및 문제 정보를 받아, 외부 PDDL 솔버(기본값: POPF)를 호출한다. 솔버는 이 정보를 바탕으로 목표를 달성하기 위한 행동 시퀀스(plan)를 찾아 반환한다.64
4. **BT 변환 및 실행:** `Executor` 노드가 생성된 계획(예: `(fly base wp1)`, `(pickup item1)`)을 입력받는다. PlanSys2의 핵심적인 기능 중 하나는, 이 순차적인 계획을 분석하여 행동 간의 인과 관계(causal relationship)를 나타내는 **실행 그래프(execution graph)**를 생성하고, 이를 기반으로 최적화된 BT를 동적으로 구축하는 것이다. 이 BT는 서로 의존성이 없는 행동들을 병렬로 실행하도록 구성되어 임무 수행 시간을 단축시킨다.62
5. 생성된 BT가 실행되면서, 각 행동에 해당하는 BT 노드는 실제 행동을 수행하는 ROS2 노드, 즉 **Action Performer**를 호출하여 임무를 수행한다.68

이러한 구조는 HTN-BT 하이브리드 아키텍처의 철학을 PDDL이라는 표준화된 언어를 통해 구현한 매우 실용적인 사례다. 이는 '계획'과 '실행'을 분리하는 강력한 개념을 ROS2 환경에서 효과적으로 실현하며, 복잡한 자율 드론 임무를 구현할 때 가장 먼저 고려해야 할 강력한 아키텍처 패턴 중 하나로 평가받는다.


PlanSys2는 기능적으로 분리된 4개의 핵심 노드로 구성된 모듈식 아키텍처를 채택하여 유연성과 확장성을 확보했다.64

- **Domain Expert:** PDDL 도메인 파일들을 읽고 파싱하여, 시스템의 정적인 규칙(행동, 술어 등)에 대한 정보를 관리하고 다른 노드들에게 서비스 형태로 제공한다.
- **Problem Expert:** 시스템의 동적인 상태(현재 로봇의 위치, 들고 있는 물건, 목표 등)를 관리한다. 다른 노드로부터의 업데이트를 받아 지식 베이스를 최신 상태로 유지한다.
- **Planner:** 계획이 필요할 때, `Domain Expert`와 `Problem Expert`로부터 현재의 정적/동적 정보를 받아 PDDL 문제 파일을 생성하고, 이를 플래닝 솔버에 전달하여 계획을 생성하는 역할을 한다.
- **Executor:** `Planner`로부터 계획을 받아 이를 실행 가능한 BT로 변환하고, 전체 실행 과정을 감독한다. Action Performer들과 통신하며 실제 행동을 트리거하고, 실행 상태를 모니터링하며, 계획이 성공 또는 실패했는지를 판별한다.


SMACC2는 ROS1의 SMACH에서 영감을 받아 C++로 재작성된, ROS2를 위한 강력한 이벤트 기반 비동기 상태 머신 라이브러리다.71 단순한 FSM을 넘어, 복잡한 로봇 시스템의 동시성(concurrency) 문제를 해결하는 데 특화된 고급 기능을 제공한다.


SMACC2는 전통적인 시간 기반 폴링(polling) 방식 대신 **이벤트 주도(event-driven)** 아키텍처를 채택했다.71 이는 상태 전이가 단순히 시간이 흐르거나 조건을 계속 확인하는 것이 아니라, 시스템 내에서 발생하는 명시적인 '이벤트'에 의해 트리거됨을 의미한다. 예를 들어, 센서로부터 `EvObstacleDetected` 이벤트가 발생하거나, 네비게이션 클라이언트로부터 `EvGoalReached` 이벤트가 발생했을 때 상태 전이가 일어난다. 이 방식은 불필요한 연산을 줄여 리소스를 효율적으로 사용하게 하고, 시스템의 반응성을 높인다.


SMACC2의 가장 독특하고 강력한 기능은 **'Orthogonal'**이라는 개념이다.71 이는 복잡한 다중 컴포넌트 로봇의 병렬 동작을 체계적으로 모델링하기 위해 도입된 Harel의 상태도(Statecharts) 개념에 기반한다.

- **Orthogonal의 개념:** Orthogonal은 상태 머신 전체의 생명주기 동안 유지되는 독립적인 '실행 컨텍스트' 또는 '슬롯'으로 생각할 수 있다. 일반적으로 로봇의 각 물리적 하위 시스템(예: `OrNavigation` - 네비게이션 시스템, `OrArm` - 로봇 팔, `OrGripper` - 그리퍼, `OrPerception` - 인식 시스템)에 하나씩 할당된다.71
- **동시성 관리:** Orthogonal을 사용하면, 하나의 상위 상태 내에서 여러 하위 시스템이 동시에, 그리고 독립적으로 동작하는 것을 매우 직관적으로 표현할 수 있다. 예를 들어, '물품 픽업'이라는 상태에 진입했을 때, `OrNavigation` Orthogonal은 목표 물체로 정밀하게 접근하는 행동을 수행하고, 동시에 `OrPerception` Orthogonal은 물체를 계속 추적하며, `OrArm` Orthogonal은 팔을 뻗을 준비를 하는 등의 병렬적인 작업 흐름을 자연스럽게 구현할 수 있다.76

이는 수많은 스레드나 콜백을 수동으로 관리할 때 발생하는 복잡성과 경쟁 조건(race condition) 문제를 상태 머신 프레임워크 수준에서 체계적으로 해결해준다. 따라서 SMACC2는 여러 액추에이터와 센서가 복합적으로 상호작용하는 정교한 드론 임무를 설계할 때 매우 효과적인 도구가 될 수 있다.



지금까지 논의된 이론과 기술들을 종합하여, 자율 물류 드론의 임무를 설계하는 구체적인 사례를 통해 각 아키텍처의 실제 적용 방안을 고찰한다.


- **기본 임무:** 드론은 지정된 기지(base)에서 이륙하여, 물품이 있는 픽업 장소(pickup)로 이동한다. 픽업 장소에서 물품을 성공적으로 수령한 후, 최종 목적지(destination)로 비행하여 물품을 배송한다. 배송 완료 후에는 안전하게 기지로 복귀(return-to-home)한다.
- **핵심 제약 조건 및 예외 상황:**
  1. **장애물 회피:** 모든 비행 구간에서 실시간으로 정적 및 동적 장애물을 감지하고 안전하게 회피해야 한다.
  2. **배터리 관리:** 비행 중 배터리 잔량이 사전에 설정된 임계치(예: 30%) 이하로 떨어지면, 현재 진행 중인 모든 작업을 즉시 중단하고 가장 가까운 충전소 또는 기지로 복귀해야 한다.
  3. **통신 두절:** 지상 관제소(GCS)와의 통신이 일정 시간 이상 두절될 경우, 사전에 정의된 안전 절차(예: 호버링 대기, 지정된 안전 지점으로 이동 후 착륙)를 따라야 한다.
  4. **작업 실패:** 물품 픽업이나 투하와 같은 핵심 작업이 실패할 경우, 몇 차례 재시도하고, 계속 실패하면 임무 실패를 보고하고 복귀해야 한다.


이러한 복합적인 요구사항을 만족시키기 위해, 심의적 계획과 반응형 실행을 결합한 **PlanSys2-BT 하이브리드 아키텍처**를 채택한다.

- **계획 계층 (PlanSys2):**
  - **PDDL 도메인 정의:** 드론의 기본 능력을 `(:actions...)` 섹션에 정의한다.
    - `(fly?from?to)`: 한 지점에서 다른 지점으로 비행.
    - `(pickup?item?loc)`: 특정 위치에서 아이템을 집음.
    - `(dropoff?item?loc)`: 특정 위치에 아이템을 내려놓음.
  - **PDDL 문제 정의:** `Planner` 노드는 이 도메인과 현재 문제(예: `(at drone base)`, `(at item pickup_loc)`, goal: `(delivered item destination)`)를 기반으로 전체 임무의 거시적인 순서, 예를 들어 `(fly base pickup_loc)`, `(pickup item pickup_loc)`, `(fly pickup_loc destination)`, `(dropoff item destination)`, `(fly destination base)`와 같은 계획을 생성한다.
- **실행 계층 (BehaviorTree.CPP):**
  - PlanSys2의 `Executor`는 생성된 계획을 받아, 이를 실행할 BT를 동적으로 구성한다.
  - 계획의 각 단계는 특정 BT 서브트리에 매핑된다. 예를 들어, `(fly base pickup_loc)` 액션은 ROS2의 표준 네비게이션 스택인 **Nav2**를 사용하는 BT를 트리거한다.78
  - `nav2_bt_navigator`는 `ComputePathToPose`, `FollowPath`, 그리고 다양한 복구 행동(recovery behaviors) 노드들을 포함하는 정교한 BT를 사용하여 실제 네비게이션을 수행한다. 이 과정에서 실시간 장애물 회피는 Nav2의 컨트롤러 및 코스트맵 레이어에서 처리된다.78


견고한 임무 수행을 위해 BT를 이용한 예외 처리 설계는 필수적이다. 이는 주로 `Fallback` 노드와 `Decorator` 노드를 활용한 특정 디자인 패턴을 통해 구현된다.

- **전역 안전 감시 (Global Safety Monitor):** 전체 임무 BT의 최상위 루트 노드는 `ReactiveFallback`으로 구성한다. 이 노드는 매 틱마다 자식들을 왼쪽에서 오른쪽으로 평가하며, 하나가 `RUNNING` 또는 `SUCCESS`를 반환하면 나머지 자식들은 평가하지 않는다.
  - 가장 왼쪽(최고 우선순위)에는 **전역 안전 상태**를 지속적으로 확인하는 `Condition` 노드들을 배치한다.
    - `IsBatteryLow?`: `sensor_msgs/BatteryState` 토픽을 구독하여 배터리 전압이 임계치 이하인지 확인하는 노드.81
    - `IsConnectionLost?`: GCS와의 마지막 통신 시간을 확인하여 타임아웃이 발생했는지 검사하는 노드.
  - 이러한 조건 노드들 다음에 주 임무를 수행하는 서브트리를 배치한다.
- **저전력 복구 (Low Battery Recovery):**
  - `ReactiveFallback`의 구조상, 만약 `IsBatteryLow?` 조건이 `SUCCESS`를 반환하면(즉, 배터리가 부족해지면), `ReactiveFallback`은 즉시 현재 실행 중이던 주 임무 서브트리를 **중단(halt)**시키고, 그 다음 자식인 'Return To Home' 서브트리를 실행한다. 이것이 BT의 반응적 특성을 활용한 핵심적인 **결함 허용(fault tolerance)** 패턴이다.34
- **액션 레벨 재시도 및 타임아웃 (Action-Level Retry & Timeout):**
  - **재시도:** '물품 픽업 확인'과 같이 일시적인 통신 오류나 물리적 오차로 실패할 수 있는 작업의 경우, 해당 액션 노드를 `RetryUntilSuccessful` 데코레이터로 감싸준다. 이 데코레이터는 자식 노드가 `FAILURE`를 반환하면, 지정된 횟수(예: `num_attempts="3"`)만큼 자동으로 재시도한다.30
  - **타임아웃:** 특정 액션이 응답이 없거나 무한정 `RUNNING` 상태에 빠지는 것을 방지하기 위해, `Timeout` 데코레이터를 사용한다. 예를 들어, `<Timeout msec="5000">`로 액션 노드를 감싸면, 5초 내에 작업이 완료되지 않을 경우 해당 노드는 강제로 중단되고 `FAILURE`를 반환한다. 그러면 상위의 `Fallback` 노드가 이 실패를 감지하고 대체 행동(예: 임무 실패 알림 후 복귀)을 실행하도록 유도할 수 있다.84

아래 표는 이 사례 연구에서 사용될 수 있는 주요 BT 노드들과 그에 대한 ROS2 구현 세부 사항을 요약한 것이다. 이는 추상적인 아키텍처 설계를 실제 코드로 변환하는 과정에 대한 구체적인 청사진을 제공한다.

| 임무 단계/결함                 | BT 노드 이름         | BT 노드 유형                                       | ROS2 구현 세부 사항                                   | 목적                                        |
| ------------------------------ | -------------------- | -------------------------------------------------- | ----------------------------------------------------- | ------------------------------------------- |
| **(임무) 경로점 비행**         | `NavigateToWaypoint` | `RosActionNode<nav2_msgs::action::NavigateToPose>` | `nav2_bt_navigator`의 `NavigateToPose` 액션 서버 호출 | 목표 지점까지 자율 비행                     |
| **(임무) 물품 픽업**           | `PickupItem`         | `RosActionNode<custom_msgs::action::Pickup>`       | 그리퍼 제어 및 확인을 위한 커스텀 액션 서버 호출      | 물품을 안전하게 집는 동작 수행              |
| **(결함) 배터리 부족 감지**    | `IsBatteryLow`       | `ConditionNode`                                    | `/battery_state` 토픽을 구독하여 전압/잔량 확인       | 배터리 상태가 임계치 이하인지 실시간 검사   |
| **(결함) 배터리 부족 시 복귀** | `ReturnToHome`       | `Sequence` (in `Fallback`)                         | `ClearCostmaps`, `GoHomeAction` 노드들의 순차 실행    | 비상 상황 시 안전하게 기지로 복귀           |
| **(결함) 픽업 실패 시 재시도** | `RetryPickup`        | `RetryUntilSuccessful`                             | `PickupItem` 액션 노드를 감싸서 3회 재시도 설정       | 일시적인 실패에 대한 견고성 확보            |
| **(결함) 통신 두절 감지**      | `IsConnectionLost`   | `ConditionNode`                                    | GCS Heartbeat 토픽의 마지막 수신 시간 확인            | GCS와의 통신 상태 실시간 모니터링           |
| **(결함) 통신 두절 시 대기**   | `HoverAndWait`       | `Sequence` (in `Fallback`)                         | `SetFlightModeHover`, `Wait` 노드들의 순차 실행       | 통신 재연결을 위해 안전하게 호버링하며 대기 |


본 보고서는 드론의 자율 임무 수행을 위한 세 가지 핵심 AI 아키텍처인 HTN, BT, iFSM을 이론적, 실용적 관점에서 심층적으로 고찰하고, ROS2 환경에서의 구현 방안을 제시했다. 분석 결과, 각 기술은 뚜렷한 장단점을 가지며, 특정 시나리오에 따라 그 유용성이 달라짐을 확인했다.

- **각 기술의 최적 활용 시나리오 요약:**

  - **HTN/PDDL (PlanSys2):** 잘 정의된 절차와 복잡한 작업 의존성을 가진, 예측 가능한 환경에서의 고수준 임무 계획에 최적이다. 예를 들어, 공장 내부의 정해진 경로를 따라 부품을 운반하는 물류 임무처럼, '어떻게' 해야 하는지에 대한 명확한 절차적 지식이 존재할 때 강력한 성능을 발휘한다.
  - **행동 트리 (BehaviorTree.CPP):** 동적이고 예측 불가능한 환경에서의 반응형 실행 및 견고한 예외 처리에 가장 적합하다. 수색 및 구조 임무처럼 시시각각 변하는 상황에 대응하고, 다양한 실패 시나리오에 대한 복구 절차를 명확하게 정의해야 할 때 필수적이다.
  - **iFSM (SMACC2):** 로봇의 근본적인 운영 모드(예: 이륙, 비행, 착륙, 충전)를 관리하거나, 네비게이션, 매니퓰레이션, 인식 등 여러 하드웨어 서브시스템의 동시 동작을 명확하고 안전하게 모델링해야 할 때 최적의 선택이다.

- **추천 아키텍처:** 현대의 복잡한 드론 임무는 심의적 계획 능력과 반응형 실행 능력을 모두 요구한다. 따라서 단일 패러다임에 의존하기보다는, **PlanSys2(PDDL)와 BehaviorTree.CPP를 결합한 하이브리드 아키텍처**를 구축하는 것을 가장 강력하게 추천한다. 이 아키텍처는 PlanSys2를 통해 전략적인 장기 계획을 수립하고, BT를 통해 그 계획을 전술적으로, 그리고 반응적으로 실행함으로써, 드론 시스템의 지능성과 견고성을 동시에 극대화하는 가장 성숙하고 실용적인 접근법이다.

- 미래 연구 방향: LLM 기반 계획 생성과의 통합 가능성:

  최근 대규모 언어 모델(LLM)을 활용하여 인간의 자연어 지시로부터 직접 작업 계획(PDDL 문제 파일)이나 실행 가능한 행동 트리(BT XML)를 생성하는 연구가 폭발적인 관심을 받고 있다.85 이는 자율 시스템의 패러다임을 바꿀 잠재력을 지닌다.

  미래의 드론 시스템은 사용자가 "창고에서 가장 가벼운 상자를 집어서 3번 게이트 앞 책상으로 가져다줘"와 같은 모호하고 복잡한 자연어 명령을 내리면, LLM이 이를 해석하여 PlanSys2를 위한 PDDL 문제 파일을 동적으로 생성하거나, 직접 실행 가능한 BT XML을 생성하는 아키텍처로 발전할 것이다. 이러한 통합은 드론의 자율성을 한 차원 더 높이고, 비전문가도 복잡한 로봇 임무를 쉽게 지시할 수 있게 만들어, 드론 기술의 활용 범위를 획기적으로 넓히는 핵심 동력이 될 것이다. 따라서 향후 연구는 이러한 LLM 기반 동적 계획 생성 기술을 기존의 견고한 계획/실행 프레임워크와 안정적으로 통합하는 데 집중되어야 할 것이다.


1. Behavior Tree Capabilities for Dynamic Multi-Robot Task Allocation with Heterogeneous Robot Teams - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2402.02833
2. arXiv:2111.06714v1 [cs.NI] 12 Nov 2021, accessed July 23, 2025, https://arxiv.org/pdf/2111.06714
3. Functional Architecture using ROS for Autonomous UAVs - SciTePress, accessed July 23, 2025, https://www.scitepress.org/Papers/2020/98883/98883.pdf
4. Mission Control over Multi UAVs in the Real-time Distributed Hardware-In-the-Loop Environment, accessed July 23, 2025, [https://meral.edu.mm/record/2513/files/Mission%20Control%20over%20Multi%20UAVs....pdf](https://meral.edu.mm/record/2513/files/Mission Control over Multi UAVs....pdf)
5. AI-Powered Drone Surveying for Smarter Utility Monitoring, accessed July 23, 2025, https://flyghtcloud.ideaforgetech.com/resources/blogs/ai-powered-drone-surveying-utilities
6. What is the difference between reactive and deliberative robotic control? - Milvus, accessed July 23, 2025, https://milvus.io/ai-quick-reference/what-is-the-difference-between-reactive-and-deliberative-robotic-control
7. Deliberative vs Reactive Control - Number Analytics, accessed July 23, 2025, https://www.numberanalytics.com/blog/deliberative-vs-reactive-control-robotics
8. Hierarchical task network - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Hierarchical_task_network
9. www.geeksforgeeks.org, accessed July 23, 2025, [https://www.geeksforgeeks.org/artificial-intelligence/hierarchical-task-network-htn-planning-in-ai/#:~:text=HTN%20planning%20is%20a%20form,also%20known%20as%20primitive%20tasks).](https://www.geeksforgeeks.org/artificial-intelligence/hierarchical-task-network-htn-planning-in-ai/#:~:text=HTN planning is a form,also known as primitive tasks).)
10. Hierarchical Task Network (HTN) Planning in AI - GeeksforGeeks, accessed July 23, 2025, https://www.geeksforgeeks.org/artificial-intelligence/hierarchical-task-network-htn-planning-in-ai/
11. 계층적 네트워크: 코어, 배포 및 액세스 레이어 | 파이버몰 - FiberMall, accessed July 23, 2025, https://www.fibermall.com/ko/blog/core-distribution-and-assess-layer.htm
12. 지식관리 방법론 - 계층적 네트워크 - 노션 일잘러 Jeremy - 티스토리, accessed July 23, 2025, [https://kys4620.tistory.com/entry/%EC%A7%80%EC%8B%9D%EA%B4%80%EB%A6%AC-%EB%B0%A9%EB%B2%95%EB%A1%A0-%EA%B3%84%EC%B8%B5%EC%A0%81-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC](https://kys4620.tistory.com/entry/지식관리-방법론-계층적-네트워크)
13. (PDF) On Hierarchical Task Networks - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/309588967_On_Hierarchical_Task_Networks
14. A HTN Planner For A Real-Time Strategy Game - CiteSeerX, accessed July 23, 2025, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=2e991c2c94cb53d2db144a5e869e781dce69c443
15. Exploring HTN Planners through Example - Game AI Pro, accessed July 23, 2025, https://www.gameaipro.com/GameAIPro/GameAIPro_Chapter12_Exploring_HTN_Planners_through_Example.pdf
16. HTN Planning as Heuristic Progression Search - Journal of Artificial Intelligence Research, accessed July 23, 2025, https://jair.org/index.php/jair/article/download/11282/26578/23423
17. HATP: Hierarchical Agent-based Task Planner - IFAAMAS, accessed July 23, 2025, https://ifaamas.org/Proceedings/aamas2018/pdfs/p1823.pdf
18. What's the difference between Behavior Trees and Hierarchical Task Networks - Reddit, accessed July 23, 2025, https://www.reddit.com/r/gamedev/comments/1ozugf/whats_the_difference_between_behavior_trees_and/
19. DEVELOPING AN HTN PLANNER FOR VIRTUAL ECOSYSTEMS - UPCommons, accessed July 23, 2025, https://upcommons.upc.edu/bitstream/handle/2117/399252/178088.pdf;jsessionid=47F187246930B4D97258C6100ADFDBA9?sequence=2
20. (PDF) An Overview of Hierarchical Task Network Planning - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/261217494_An_Overview_of_Hierarchical_Task_Network_Planning
21. University of Groningen HTN planning: Overview, comparison, and beyond Georgievski, I. - CORE, accessed July 23, 2025, https://core.ac.uk/download/pdf/232493354.pdf
22. A Hybrid Approach to Planning and Execution in Dynamic Environments Through Hierarchical Task Networks and Behavior Trees, accessed July 23, 2025, https://cdn.aaai.org/ojs/13044/13044-52-16561-1-2-20201228.pdf
23. Behavior tree (artificial intelligence, robotics and control) - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Behavior_tree_(artificial_intelligence,_robotics_and_control)
24. Introduction to behavior trees - Robohub, accessed July 23, 2025, https://robohub.org/introduction-to-behavior-trees/
25. Unlocking the Power of Behavior Trees in AI and Robotics: A Guide by Curate Consulting Services, accessed July 23, 2025, https://curatepartners.com/blogs/skills-tools-platforms/unlocking-the-power-of-behavior-trees-in-ai-and-robotics-a-guide-by-curate-consulting-services/
26. Behavior Tree 개념 및 동작 - 그냥 그런 블로그 - 티스토리, accessed July 23, 2025, https://lifeisforu.tistory.com/327
27. Introduction to BTs | BehaviorTree.CPP, accessed July 23, 2025, https://www.behaviortree.dev/docs/learn-the-basics/bt_basics/
28. Behavior Tree를 알아봅시다, accessed July 23, 2025, https://engineering.linecorp.com/ko/blog/behavior-tree/
29. Unity Behavior Tree 시스템 개요 - 1 ( Behavior Tree 란? ) - Def's Programming And Life, accessed July 23, 2025, [https://defs-program.tistory.com/entry/Unity-Behavior-Tree-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EA%B0%9C%EC%9A%94-1-Behavior-Tree-%EB%9E%80](https://defs-program.tistory.com/entry/Unity-Behavior-Tree-시스템-개요-1-Behavior-Tree-란)
30. Decorators | BehaviorTree.CPP, accessed July 23, 2025, https://www.behaviortree.dev/docs/nodes-library/decoratornode/
31. Building The Foundation of Robot Explanation Generation Using Behavior Trees - HARP Lab, accessed July 23, 2025, https://harplab.github.io/assets/pubs/lee_THRI2021.pdf
32. State Machines vs Behavior Trees: designing a decision-making architecture for robotics, accessed July 23, 2025, https://www.polymathrobotics.com/blog/state-machines-vs-behavior-trees
33. 2.4 Hierarchical Finite State Machine (HFSM) & Behavior Tree (BT) - Stanford University, accessed July 23, 2025, https://web.stanford.edu/class/cs123/lectures/CS123_lec08_HFSM_BT.pdf
34. Behavior Trees in Robotics - DiVA portal, accessed July 23, 2025, https://www.diva-portal.org/smash/get/diva2:1078940/FULLTEXT01.pdf
35. Behavior tree - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Behavior_tree
36. Behavior Tree - Lark, accessed July 23, 2025, https://www.larksuite.com/en_us/topics/ai-glossary/behavior-tree
37. Finite-state machine - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Finite-state_machine
38. Using Finite State Automata in Robotics - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/profile/Richard-Balogh/publication/327389755_Using_Finite_State_Machines_in_Introductory_Robotics_Methods_and_Applications_for_Teaching_and_Learning/links/5e95ce01a6fdcca78915778f/Using-Finite-State-Machines-in-Introductory-Robotics-Methods-and-Applications-for-Teaching-and-Learning.pdf
39. Limitations of FSM | PDF | Applied Mathematics | Computer ... - Scribd, accessed July 23, 2025, https://www.scribd.com/document/822927174/Limitations-of-FSM
40. Advantages & Disadvantages of Finite State Machines: Switch-cases, C/C++ Pointers & Lookup Tables (Part II) - Ubidots, accessed July 23, 2025, https://ubidots.com/blog/advantages_and_disadvantages_of_finite_state_machines/
41. Hierarchical Finite State Machine for AI Acting Engine | Towards Data Science, accessed July 23, 2025, https://towardsdatascience.com/hierarchical-finite-state-machine-for-ai-acting-engine-9b24efc66f2/
42. An Overview of XRobots: A Hierarchical State Machine Based Language - College of Science and Engineering - University of Minnesota, accessed July 23, 2025, https://www-users.cse.umn.edu/~gini/publications/papers/Stousig11icraw.pdf
43. State Machine for Drone Control, accessed July 23, 2025, https://dspace.cvut.cz/bitstream/handle/10467/80869/F8-BP-2019-Gura-Matus-thesis.pdf?sequence=-1&isAllowed=y
44. A Hybrid Approach to Planning and Execution in Dynamic Environments Through Hierarchical Task Networks and Behavior Trees | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/364455552_A_Hybrid_Approach_to_Planning_and_Execution_in_Dynamic_Environments_Through_Hierarchical_Task_Networks_and_Behavior_Trees
45. Self-Adjusting Finite State Machines: an approach to Real-Time Autonomous Behavior in Robots - Digital Commons @ Trinity, accessed July 23, 2025, https://digitalcommons.trinity.edu/cgi/viewcontent.cgi?article=1008&context=compsci_honors
46. [2412.05625] Can Large Language Models Help Developers with Robotic Finite State Machine Modification? - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2412.05625
47. Finite State Machine (FSM) : Types, Properties, Design and Applications - ElProCus, accessed July 23, 2025, https://www.elprocus.com/finite-state-machine-mealy-state-machine-and-moore-state-machine/
48. Behavior Trees in Robotics and AI: An Introduction - Barnes & Noble, accessed July 23, 2025, https://www.barnesandnoble.com/w/behavior-trees-in-robotics-and-ai-michele-colledanchise/1136790529
49. Trustworthy ROS Software Architecture for Autonomous Drones Missions: From RoboChart Modelling to ROS Implementation - White Rose Research Online, accessed July 23, 2025, https://eprints.whiterose.ac.uk/id/eprint/222167/1/MESA2024_paper_92.pdf
50. Robot Operating System 2 (ROS 2) Architecture | by Huseyin Kutluca - Medium, accessed July 23, 2025, https://medium.com/software-architecture-foundations/robot-operating-system-2-ros-2-architecture-731ef1867776
51. A ROS-Based GNC Architecture for Autonomous Surface Vehicle Based on a New Multimission Management Paradigm - MDPI, accessed July 23, 2025, https://www.mdpi.com/2504-446X/6/12/382
52. Introduction to Real-time Systems - ROS 2 Design, accessed July 23, 2025, https://design.ros2.org/articles/realtime_background.html
53. Understanding real-time programming - ROS 2 Documentation: Rolling documentation, accessed July 23, 2025, https://docs.ros.org/en/rolling/Tutorials/Demos/Real-Time-Programming.html
54. (PDF) ROS2 Real-time Performance Optimization and Evaluation - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/376215526_ROS2_Real-time_Performance_Optimization_and_Evaluation
55. ROS 2 (Robot Operating System): overview and key points for robotics software | Robotnik ®, accessed July 23, 2025, https://robotnik.eu/ros-2-robot-operating-system-overview-and-key-points-for-robotics-software/
56. Integration with ROS2 | BehaviorTree.CPP, accessed July 23, 2025, https://www.behaviortree.dev/docs/ros2_integration/
57. BehaviorTree/BehaviorTree.CPP: Behavior Trees Library in C++. Batteries included. - GitHub, accessed July 23, 2025, https://github.com/BehaviorTree/BehaviorTree.CPP
58. BehaviorTree.CPP, accessed July 23, 2025, https://www.behaviortree.dev/
59. BehaviorTree.CPP utilities to work with ROS2 - GitHub, accessed July 23, 2025, https://github.com/BehaviorTree/BehaviorTree.ROS2
60. ROS2 Planning System - Projects - ROS Discourse, accessed July 23, 2025, https://discourse.ros.org/t/ros2-planning-system/11844
61. PlanSys2/ros2_planning_system: This repo contains a PDDL-based planning system for ROS2. - GitHub, accessed July 23, 2025, https://github.com/PlanSys2/ros2_planning_system
62. PlanSys2: A Planning System Framework for ROS2 - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2107.00376
63. ros2_planning_system/plansys2_docs/FAQ.md at rolling - GitHub, accessed July 23, 2025, https://github.com/IntelligentRoboticsLabs/ros2_planning_system/blob/master/plansys2_docs/FAQ.md
64. PlanSys2 design - ROS2 Planning System 2 1.0.0 documentation, accessed July 23, 2025, https://plansys2.github.io/design/index.html
65. ROS Package: plansys2_planner, accessed July 23, 2025, https://index.ros.org/p/plansys2_planner/
66. Optimized Execution of PDDL Plans using Behavior Trees, accessed July 23, 2025, https://archive.illc.uva.nl/AAMAS-2021/pdfs/p1596.pdf
67. Autonomous Robot Task Execution in Flexible Manufacturing: Integrating PDDL and Behavior Trees in ARIAC 2023 - PubMed Central, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC11504948/
68. Implement actions as Behavior Trees - ROS2 Planning System, accessed July 23, 2025, https://plansys2.github.io/tutorials/docs/bt_actions.html
69. plansys2.github.io, accessed July 23, 2025, [https://plansys2.github.io/design/index.html#:~:text=PlanSys2%20has%20a%20modular%20design,goals%20that%20compose%20the%20model.](https://plansys2.github.io/design/index.html#:~:text=PlanSys2 has a modular design,goals that compose the model.)
70. PlanSys2 architecture. | Download Scientific Diagram - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/figure/PlanSys2-architecture_fig1_353055585
71. robosoft-ai/SMACC2: An Event-Driven, Asynchronous, Behavioral State Machine Library for ROS2 (Robotic Operating System) applications written in C++ - GitHub, accessed July 23, 2025, https://github.com/robosoft-ai/SMACC2
72. SMACC – State Machine Asynchronous C++ | An Event-Driven, Asynchronous, Behavioral State Machine Library for real-time ROS (Robotic Operating System) applications written in C++, accessed July 23, 2025, https://smacc.dev/
73. robosoft-ai/SMACC: An Event-Driven, Asynchronous, Behavioral State Machine Library for ROS (Robotic Operating System) applications written in C++ - GitHub, accessed July 23, 2025, https://github.com/robosoft-ai/SMACC
74. Orthogonals | SMACC – State Machine Asynchronous C++, accessed July 23, 2025, https://smacc.dev/orthogonals/
75. Intro to Substate Objects | SMACC – State Machine Asynchronous C++, accessed July 23, 2025, https://smacc.dev/intro-to-substate-objects/
76. 10. State Machines for Complex Robot Behavior, with Brett Aldrich - Sense Think Act Podcast, accessed July 23, 2025, https://www.sensethinkact.com/episodes/10-brett-aldrich
77. Introducing the SMACC State Machine Library - ROS General - Open Robotics Discourse, accessed July 23, 2025, https://discourse.ros.org/t/introducing-the-smacc-state-machine-library/14963
78. navigation2/nav2_behavior_tree/README.md at main / ros ... - GitHub, accessed July 23, 2025, https://github.com/ros-planning/navigation2/blob/main/nav2_behavior_tree/README.md
79. nav2_bt_navigator: Humble 1.1.18 documentation - the official ROS docs, accessed July 23, 2025, https://docs.ros.org/en/ros2_packages/humble/api/nav2_bt_navigator/
80. Detailed Behavior Tree Walkthrough - Nav2 1.0.0 documentation, accessed July 23, 2025, https://docs.nav2.org/behavior_trees/overview/detailed_behavior_tree_walkthrough.html
81. A Behavior Tree Approach for Battery-Aware Inspection of Large Structures Using Drones - Guilherme Pereira - West Virginia University, accessed July 23, 2025, https://guilhermepereira.faculty.wvu.edu/files/d/7918e5c9-89a0-4cf3-9765-0241c1f2ba61/icuas24b.pdf
82. Human-Multi-Drone Interaction in Search and Rescue Systems under High Cognitive Workload - Umeå University, accessed July 23, 2025, https://umu.diva-portal.org/smash/get/diva2:1832071/FULLTEXT01.pdf
83. Decorators - BehaviorTree.CPP, accessed July 23, 2025, https://www.behaviortree.dev/docs/4.0.2/nodes-library/decoratornode/
84. Race condition in timeout decorator node? / Issue #57 / BehaviorTree/BehaviorTree.CPP - GitHub, accessed July 23, 2025, https://github.com/BehaviorTree/BehaviorTree.CPP/issues/57
85. LLM-BRAIn: AI-driven Fast Generation of Robot Behaviour Tree ..., accessed July 23, 2025, https://arxiv.org/pdf/2305.19352
86. [2502.03814] Large Language Models for Multi-Robot Systems: A Survey - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2502.03814
87. Safety Aware Task Planning via Large Language Models in Robotics - arXiv, accessed July 23, 2025, https://arxiv.org/html/2503.15707v1
88. Robo-Troj: Attacking LLM-based Task Planners - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2504.17070
89. arXiv:2503.18971v1 [cs.AI] 22 Mar 2025, accessed July 23, 2025, [https://arxiv.org/pdf/2503.18971?](https://arxiv.org/pdf/2503.18971)


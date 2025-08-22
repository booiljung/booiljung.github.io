---
layout: page
title: 행동트리(Behavior Tree)
permalink: /planning/mission planning/행동트리(Behavior Tree)
---



행동트리(Behavior Tree, 이하 BT)는 단순한 의사결정 구조를 넘어, 태스크(task) 실행을 위한 모듈식 계층 구조 모델이다.1 2000년대 초반, 게임 산업은 유한 상태 머신(Finite State Machine, FSM)이 가진 확장성의 한계에 직면했고, BT는 이를 극복하기 위한 핵심 대안으로 부상했다. 

`Halo`, `Grand Theft Auto`, `Bioshock`과 같은 기념비적인 게임 타이틀에서 그 효용성이 입증되면서 BT는 비플레이어 캐릭터(Non-Player Character, NPC)의 행동을 모델링하는 주류 기술로 확고히 자리 잡았다.1

BT의 적용 범위는 비디오 게임에 국한되지 않는다. 그 구조적 우수성, 즉 복잡한 행동을 단순한 태스크의 조합으로 구성할 수 있는 능력은 군사 시뮬레이션, 로보틱스 제어 시스템 등 자율 에이전트의 의사결정이 필요한 모든 분야로 빠르게 확산되었다.2

본 보고서는 BT의 기본 원리를 단순히 나열하는 것을 넘어선다. BT의 구조적 우수성과 실제 적용 시 발생하는 한계를 비판적으로 분석하고, 나아가 기계학습(Machine Learning, ML)과의 융합을 통해 끊임없이 진화하는 BT의 미래를 심도 있게 조망하는 것을 목표로 한다. 이는 단순한 기술 설명서를 넘어, 현대 AI 시스템 설계의 핵심 패러다임으로서 BT의 본질을 파헤치는 심층 분석 보고서가 될 것이다.



행동트리의 모든 실행은 '틱(tick)'이라는 개념에서 시작한다. 틱은 트리의 루트 노드(root node)에 주기적으로 전송되는 활성화 신호(enabling signal)이며, 이 신호는 트리의 말단인 리프 노드(leaf node)에 도달할 때까지 아래로 전파된다.4 이 틱은 게임 엔진의 프레임 업데이트 주기와 동기화되거나 6, 특정 시간 간격에 따라 발생하며 8, 이를 통해 BT는 에이전트의 상태와 주변 환경을 지속적으로 재평가하여 적절한 행동을 결정한다.

이러한 틱의 전파 방식은 깊이 우선 탐색(Depth-First Search, DFS)을 따른다.9 루트에서 시작된 틱은 각 노드의 자식들을 왼쪽에서 오른쪽 순서로 순회한다. 이 순서는 단순한 구현상의 편의가 아니라, 행동의 우선순위를 정의하는 핵심적인 설계 요소다.11 트리의 왼쪽에 위치한 행동일수록 더 높은 우선순위를 가지며, 먼저 평가될 기회를 얻는다.

BT 실행 모델의 본질은 상태를 명시적으로 저장하고 관리하는 것이 아니라, 매 틱마다 현재 월드의 상태를 기반으로 어떤 행동을 실행할지 루트에서부터 다시 결정하는 지극히 반응적인(reactive) 모델이라는 점이다.3 이는 BT가 본질적으로 상태가 없는(stateless) 구조를 가지면서도, 동적인 환경 변화에 매우 민첩하게 대응할 수 있게 만드는 근간이다.

이 틱 기반 모델은 단순한 함수 호출의 반복이 아니다. 이는 '상태를 인지하는 폴링(State-Aware Polling)' 메커니즘으로 이해해야 한다. 각 틱은 시스템의 현재 상태를 확인하는 '심장 박동'과 같다. 만약 특정 행동 노드가 완료되기까지 시간이 더 필요한 경우, 해당 노드는 `Running` 상태를 반환한다. 이는 시스템에 "나는 아직 실행 중이니, 다음 심장 박동(tick) 때 나를 다시 깨워달라"는 신호를 보내는 것과 같다.4 이 메커니즘 덕분에, 본질적으로 상태 정보를 저장하지 않는 트리 구조 내에서도 시간의 흐름에 따라 지속되는 상태를 가진(stateful) 행동의 실행 흐름을 효과적으로 관리할 수 있다. 이는 명시적인 이벤트에 의해 상태 전이가 발생하는 FSM과의 근본적인 차이점이다.


BT는 명확한 역할 분담을 가진 세 가지 주요 노드 유형의 계층적 조합으로 구성된다: 루트 노드, 제어 흐름 노드, 그리고 실행 노드다.5

- **루트 노드 (Root Node):** 트리의 유일한 진입점으로, 부모 노드가 없으며 단 하나의 자식 노드만을 가진다.5 외부 시스템으로부터 틱 신호를 받아 트리 전체의 실행을 시작시키는 역할을 한다.

- **제어 흐름 노드 (Control Flow / Composite Nodes):** 하나 이상의 자식 노드를 가지며, 이들의 실행 순서와 조건을 제어하는 중간 관리자 역할을 한다. 이 노드들은 디자인 패턴의 컴포지트 패턴(Composite Pattern)과 유사하게 동작하여, 여러 자식 노드를 하나의 논리적 단위로 묶어 처리할 수 있게 해준다.9 대표적인 예로 

  `Sequence`와 `Selector`가 있으며, 이들은 BT의 논리적 흐름을 구성하는 뼈대가 된다.

- **실행 노드 (Execution / Leaf Nodes):** 트리의 가장 말단에 위치하며, 실제적인 작업을 수행하거나 월드의 상태를 검사하는 역할을 한다.9 이들은 더 이상 자식 노드를 가지지 않으며, AI 에이전트와 실제 게임 월드 또는 로봇 시스템 간의 구체적인 상호작용이 일어나는 인터페이스 지점이다.4 실행 노드는 크게 

  `Action`과 `Condition` 두 종류로 나뉜다.


모든 노드는 자신의 실행이 끝난 후, 그 결과를 부모 노드에게 세 가지 상태 중 하나로 반환해야 한다: `Success` (성공), `Failure` (실패), 그리고 `Running` (실행 중)이다.4 일부 구현에서는 

`Aborted` (외부 요인에 의한 중단) 11나 

`Suspended` (일시 정지) 16와 같은 추가 상태를 정의하기도 하지만, 이 세 가지가 가장 보편적이고 핵심적인 상태 값이다.

- **`Success`와 `Failure`:** 이 두 상태는 태스크가 한 틱 내에 완료되었음을 의미하며, 그 결과가 성공적이었는지 실패했는지를 나타낸다. 부모인 제어 흐름 노드는 이 반환값을 기반으로 다음 자식을 실행할지, 아니면 현재 브랜치의 실행을 중단하고 자신의 상태를 결정할지를 판단한다.4

- **`Running`의 중요성:** 이 상태는 BT의 표현력을 극적으로 향상시키는 핵심 요소다. `Running`은 "목표 지점으로 이동 중" 또는 "공격 애니메이션 재생 중"과 같이, 태스크가 완료되기까지 한 틱 이상의 시간이 필요함을 의미한다.4

  `Running`을 반환한 노드는 다음 틱에서 다시 실행될 우선권을 가지게 된다. 이 메커니즘이 없다면 BT는 단순히 매 틱마다 즉시 완료되는 조건과 행동만을 처리하는 제한적인 의사결정 트리 15에 머물렀을 것이다.

`Running` 상태의 존재는 행동트리를 단순한 의사결정 구조에서 '행동 실행 관리자'로 격상시킨다. 의사결정 트리는 한 번의 평가를 통해 최종 결론(Action)에 도달하지만, BT는 `Running`을 통해 '결정'과 '실행'을 분리한다. 예를 들어, 첫 틱에서 "플레이어를 공격하라"는 결정이 내려지고 `MoveToPlayer` 액션이 `Running`을 반환하면, BT는 다음 틱들에서 계속 `MoveToPlayer`를 실행하며 그 상태를 추적하고 관리한다. 즉, BT는 무엇을 할지 결정할 뿐만 아니라, 그 행동이 완료될 때까지의 전 과정을 관리하는 역할을 겸하는 것이다. 이러한 관점은 BT를 단순한 로직 분기 구조가 아닌, 비동기 태스크를 관리하는 경량 스케줄러로 이해하게 하며, 복잡한 행동 조합 능력을 파악하는 데 매우 중요하다.


행동트리의 논리적 흐름은 주로 제어 흐름 노드(Composite Nodes)에 의해 결정된다. 이 노드들은 자식 노드들의 실행을 조율하며, 이를 통해 복잡하고 지능적인 행동 패턴을 만들어낸다.


`Sequence` 노드는 논리적 AND 연산과 유사하게 동작하며, 자식 노드들을 정해진 순서대로 모두 성공적으로 실행하는 것을 목표로 한다.12

- **작동 원리:** `Sequence`는 자신의 자식 노드들을 왼쪽에서 오른쪽 순서로 하나씩 실행한다. 현재 실행한 자식이 `Success`를 반환하면, 다음 자식을 실행한다. 모든 자식이 순서대로 `Success`를 반환했을 때만, `Sequence` 노드 자신도 부모에게 `Success`를 반환한다.1
- **실패 처리:** 만약 자식 노드 중 하나라도 `Failure`를 반환하면, `Sequence`는 즉시 전체 실행을 중단하고 부모에게 `Failure`를 반환한다. 실패한 노드 이후의 자식들은 실행될 기회조차 얻지 못한다.9
- **실행 중 처리:** 자식 중 하나가 `Running`을 반환하면, `Sequence` 역시 즉시 부모에게 `Running`을 반환한다. 이 경우, `Sequence`는 현재 실행 중인 자식의 위치를 기억하고 있다가 다음 틱에서 해당 자식부터 실행을 재개한다.4
- **사용 예시:** "문으로 이동한다", "문을 연다", "방으로 들어간다"와 같이 반드시 순서대로, 그리고 모두 성공해야만 의미가 있는 일련의 행동들을 묶는 데 이상적이다.2


`Selector` 노드는 `Fallback` 노드라고도 불리며, 논리적 OR 연산과 유사하게 동작한다. 여러 자식 노드 중 실행 가능한 첫 번째 행동을 찾아 수행하는 것을 목표로 한다.4

- **작동 원리:** `Selector`는 자신의 자식 노드들을 왼쪽에서 오른쪽 순서로 하나씩 실행한다. 만약 자식 중 하나가 `Success`나 `Running`을 반환하면, `Selector`는 즉시 전체 실행을 중단하고 해당 상태를 그대로 부모에게 반환한다. 성공한 노드 이후의 자식들은 평가되지 않는다.9
- **실패 처리:** 만약 자식 노드가 `Failure`를 반환하면, `Selector`는 포기하지 않고 다음 자식 노드를 시도한다. 모든 자식 노드가 `Failure`를 반환했을 때 비로소 `Selector` 자신도 부모에게 `Failure`를 반환한다.4
- **사용 예시:** "적이 보이는가? (보이면) 공격한다", "(안 보이면) 마지막 목격 장소로 이동한다", "(그것도 실패하면) 순찰한다"와 같이 여러 행동 대안 중 우선순위에 따라 가능한 하나를 선택하는 데 사용된다. 트리의 왼쪽에 배치된 자식일수록 더 높은 우선순위를 갖게 된다.11

이러한 `Sequence`와 `Selector`는 단순한 제어 흐름 도구가 아니다. 이들은 '내재된 우선순위'를 통해 AI의 성격을 규정하는 핵심적인 설계 도구다. 모든 자료에서 노드의 실행 순서는 '왼쪽에서 오른쪽'으로 명시되어 있다.9 이는 단순한 구현 디테일이 아니라 AI의 행동 철학을 결정한다. 예를 들어, 

`Selector`의 자식으로 `(공격, 후퇴, 경계)`를 배치하는 것과 `(후퇴, 경계, 공격)`을 배치하는 것은 완전히 다른 성격의 AI를 창조한다. 전자는 우선적으로 공격을 시도하는 공격적인 AI이며, 후자는 위험을 먼저 회피하려는 소극적인 AI다. 이처럼 개발자는 노드의 순서를 조정하는 것만으로 AI의 '성격(personality)'과 '전술적 선호도(tactical preference)'를 직접적으로 코딩하고 미세하게 튜닝할 수 있다.

**Table 1: 주요 행동트리 노드 유형 및 기능**

| 노드 유형     | 아이콘 (일반적) | 핵심 기능          | 실행 규칙 (자식 노드 처리 방식)     | Success 반환 조건                  | Failure 반환 조건                   | Running 반환 조건                       |
| ------------- | --------------- | ------------------ | ----------------------------------- | ---------------------------------- | ----------------------------------- | --------------------------------------- |
| **Root**      | ▶               | 트리의 시작점      | 단일 자식을 틱(tick)함              | 자식이 Success 반환 시             | 자식이 Failure 반환 시              | 자식이 Running 반환 시                  |
| **Sequence**  | -->>               | 순차적 실행 (AND)  | 자식들을 왼쪽부터 순서대로 실행     | 모든 자식이 Success 반환 시        | 자식 중 하나가 Failure 반환 시      | 자식 중 하나가 Running 반환 시          |
| **Selector**  | ?               | 우선순위 선택 (OR) | 자식들을 왼쪽부터 순서대로 실행     | 자식 중 하나가 Success 반환 시     | 모든 자식이 Failure 반환 시         | 자식 중 하나가 Running 반환 시          |
| **Parallel**  | ⇉               | 동시 실행          | 모든 자식을 동시에 틱함             | 정책에 따라 다름 (예: N개 성공 시) | 정책에 따라 다름 (예: M개 실패 시)  | 정책에 따라 다름                        |
| **Decorator** | ◇               | 자식 노드 변형     | 단일 자식을 특정 조건/방식으로 실행 | 자식의 결과를 변형하거나 강제      | 자식의 결과를 변형하거나 강제       | 자식의 Running 상태를 그대로 반환       |
| **Action**    | □               | 실제 행동 수행     | 자식 없음. 월드 상태 변경           | 행동이 성공적으로 완료되었을 때    | 행동을 수행할 수 없거나 실패했을 때 | 행동이 완료되기까지 시간이 더 필요할 때 |
| **Condition** | ○               | 월드 상태 검사     | 자식 없음. 월드 상태 확인           | 조건이 참일 때                     | 조건이 거짓일 때                    | (반환해서는 안 됨)                      |


`Decorator` 노드는 단 하나의 자식 노드만을 가지며, 그 자식의 실행에 개입하여 행동을 변조하거나 제어하는 특수한 노드다.4 이름에서 알 수 있듯이, 이 노드는 디자인 패턴의 데코레이터 패턴(Decorator Pattern)과 유사한 역할을 수행하며, 기존 행동 노드의 기능을 동적으로 확장하거나 변경한다.9

- **주요 유형:**
  - **Inverter:** 자식 노드의 결과를 반전시킨다. `Success`는 `Failure`로, `Failure`는 `Success`로 바꾼다. `Running`은 그대로 유지된다. "적이 보이지 않으면 성공"과 같은 논리를 구현하는 데 사용된다.4
  - **Repeater/Loop:** 자식 노드를 지정된 횟수만큼, 혹은 무한히 반복 실행한다. `Repeat` 노드는 정해진 횟수만큼 반복하며, 반복이 끝나면 `Success`를 반환한다.9
  - **Conditional Logic:** 특정 조건을 만족할 때만 자식을 실행하거나, 특정 조건이 만족될 때까지 자식을 반복하는 등의 복잡한 제어 흐름을 만든다. `If` 데코레이터는 조건이 참일 때만 자식을 실행하고, `While` 데코레이터는 조건이 참인 동안 자식을 계속 반복 실행한다.9
  - **Force Success/Failure:** 자식 노드의 실제 반환값과 상관없이, 부모에게 항상 `Success` 또는 `Failure`를 반환하도록 강제한다. 특정 행동의 실패 여부가 전체 시퀀스의 실패로 이어지지 않게 만들고 싶을 때 유용하다.18

데코레이터의 존재는 BT의 표현력을 단순한 순차/선택 구조에서 프로그래밍 언어의 제어문 수준으로 끌어올리는 결정적인 역할을 한다. `Sequence`와 `Selector`만으로는 "체력이 50% 이상인 동안 계속 공격하라"와 같은 복잡한 로직을 간결하게 표현하기 어렵다. 하지만 데코레이터를 사용하면 이러한 논리를 쉽게 구현할 수 있다. `If` 데코레이터는 `if` 문, `While` 데코레이터는 `while` 루프, `Repeat` 데코레이터는 `for` 루프에 해당하는 기능을 제공한다.9 이를 통해 BT는 단순한 태스크의 나열이 아니라, 월드의 상태에 따라 복잡한 논리적 흐름을 생성할 수 있는 완전한 '행동 스크립팅 언어'로서의 잠재력을 갖추게 된다.


실행 노드는 트리의 가장 끝에서 실질적인 기능을 수행하는 노드들이다.

- **`Action` 노드:** 게임 월드의 상태를 변경하는 모든 종류의 '행동'을 담당한다. "플레이어 공격", "특정 위치로 이동", "아이템 사용" 등이 여기에 해당한다.15

  `Action` 노드는 비동기적인 작업(예: 이동)을 처리하기 위해 `Running` 상태를 반환할 수 있다.4 좋은 설계 원칙 중 하나는 

  `Action` 노드가 대부분 `Success`를 반환하도록 만드는 것이다. 만약 특정 액션이 자주 실패한다면, 이는 해당 액션을 실행하기 전에 필요한 전제 조건이 충족되지 않았다는 의미이므로, 액션 실행 이전에 `Condition` 노드를 통해 실행 가능 여부를 먼저 확인하는 것이 바람직하다.15

- **`Condition` 노드:** 게임 월드의 현재 상태를 '질의(query)'하고, 그 결과에 따라 `Success`(참) 또는 `Failure`(거짓)를 즉시 반환한다.15 "플레이어가 시야에 있는가?", "남은 총알이 있는가?", "체력이 50% 미만인가?"와 같은 조건 검사를 수행한다. 

  `Condition` 노드는 월드의 상태를 절대 변경해서는 안 되며, 완료에 시간이 걸리는 작업이 아니므로 `Running` 상태를 반환해서도 안 된다.4


행동트리를 깊이 이해하기 위해서는 그 이전의 지배적인 패러다임이었던 유한 상태 머신(FSM)과의 비교가 필수적이다. BT는 FSM이 가진 근본적인 문제들을 해결하기 위해 등장했으며, 두 기술은 AI 설계에 대한 서로 다른 철학을 반영한다.


FSM은 '상태(state)'와 상태 간의 '전이(transition)'로 시스템을 모델링한다.19 이 구조는 상태의 수가 적을 때는 직관적이지만, AI의 행동이 복잡해져 상태 수가 증가하면 상태 간의 전이 관계가 기하급수적으로 늘어나 소위 '스파게티' 구조가 되어버린다.20 새로운 상태를 하나 추가하기 위해서는, 그 상태와 연결될 가능성이 있는 기존의 모든 상태에 새로운 전이 로직을 추가해야 할 수도 있다. 이는 상태 간의 높은 결합도(coupling)를 유발하며, 시스템의 유지보수와 확장을 극도로 어렵게 만든다.13

**Table 2: FSM vs. Behavior Tree 핵심 특성 비교**

| 평가 기준         | 유한 상태 머신 (FSM)            | 행동트리 (BT)                      |
| ----------------- | ------------------------------- | ---------------------------------- |
| **기본 단위**     | 상태 (State)                    | 행동 (Task/Behavior)               |
| **모듈성**        | 낮음 (상태 간 강한 결합)        | 높음 (독립적인 서브트리)           |
| **확장성**        | 낮음 (상태 증가 시 복잡도 폭발) | 높음 (트리 구조로 용이하게 확장)   |
| **재사용성**      | 낮음 (전이 로직에 종속)         | 높음 (독립적 노드/서브트리 재사용) |
| **반응성**        | 제한적 (명시적 전이 필요)       | 높음 (틱 기반 재평가 및 중단)      |
| **설계 패러다임** | 상태 중심 (State-centric)       | 행동 중심 (Behavior-centric)       |
| **디버깅**        | 복잡 (상태 전이 추적 어려움)    | 용이 (시각적 트리 흐름 추적)       |
| **핵심 비유**     | `GoTo` 문                       | `함수 호출`                        |

반면, BT는 상태 전이라는 개념 자체가 없다.19 각 서브트리(subtree)는 그 자체로 완결된 행동 모듈로 기능하며, 다른 부분과 독립적으로 설계되고 테스트될 수 있다.3 이 차이는 프로그래밍 언어의 제어 흐름에 비유할 수 있다. FSM의 상태 전이는 특정 코드 라인으로 무조건 분기하는 

`GoTo` 문과 유사하다.21 일단 다른 상태로 '점프'하면 제어 흐름이 원래 위치로 돌아온다는 보장이 없어 전체적인 흐름을 추적하기 어렵다. 이와 대조적으로, BT에서 서브트리를 실행하는 것은 '함수 호출'과 같다.21 서브트리의 실행이 끝나면(

`Success` 또는 `Failure`를 반환하면), 제어권은 자신을 호출했던 부모 노드로 반드시 돌아온다. 이러한 구조 덕분에 특정 서브트리를 마치 라이브러리 함수처럼 다른 위치에 쉽게 재사용하거나, 새로운 행동 서브트리를 기존 트리에 '플러그인'처럼 끼워 넣어 기능을 확장하는 것이 매우 용이하다.3

계층적 유한 상태 머신(Hierarchical Finite State Machine, HFSM)은 상태를 계층적으로 묶어 FSM의 복잡성을 일부 완화하려 시도했지만 13, 전이를 기반으로 하는 근본적인 결합도 문제는 해결하지 못했다. 오히려 반응성을 높이기 위해 HFSM에 전이를 과도하게 추가하면, 결국 모든 상태가 서로 연결된 그래프(fully connected graph)와 다름없게 되어 계층 구조의 이점을 상실하게 된다.21


- **반응성:** BT의 가장 큰 장점 중 하나는 높은 반응성이다. 매 틱마다 루트에서부터 트리를 재평가하기 때문에, 환경 변화에 거의 즉각적으로 대응할 수 있다. 우선순위가 높은 행동(트리의 왼쪽에 위치한)은 언제든지 현재 실행 중인 낮은 우선순위의 행동을 중단시키고 제어권을 가져올 수 있다.13 이러한 메커니즘은 '조건부 중단(Conditional Aborts)' 12 또는 '관찰자 중단(Observer Aborts)' 11이라 불리며, 동적인 게임 환경에서 AI가 멍청하게 행동하지 않도록 보장하는 핵심 기능이다. 반면 FSM은 현재 상태의 로직에 묶여 있기 때문에, 다른 상태로 전환하기 위한 명시적인 전이 조건이 충족될 때까지는 새로운 상황에 반응할 수 없다. FSM의 반응성을 높이려면 더 많은 전이를 추가해야 하는데, 이는 앞서 언급했듯 모듈성을 해치는 근본적인 트레이드오프 관계에 있다.21
- **재사용성:** BT의 노드와 서브트리는 특정 상태에 종속되지 않은 독립적인 로직 블록이기 때문에 재사용성이 매우 뛰어나다.3 예를 들어, "안전한 엄폐물 찾기"라는 서브트리는 어떤 종류의 AI 캐릭터에게든 거의 수정 없이 재사용될 수 있다. 반면 FSM의 '상태'는 특정 전이 로직과 강하게 결합되어 있기 때문에, 한 FSM에 속한 상태를 다른 FSM에 그대로 가져다 사용하는 것은 거의 불가능하다.20

BT의 진정한 혁신은 AI 설계의 패러다임을 '상태' 중심에서 '행동' 중심으로 전환시켰다는 데 있다. FSM 설계 시 개발자는 "AI가 지금 어떤 상태에 있는가?"를 먼저 고민하고, 그 상태에서 할 수 있는 행동과 다른 상태로 넘어갈 조건을 정의한다. 반면 BT 설계 시 개발자는 "이 AI가 할 수 있는 행동에는 무엇이 있는가?"를 먼저 정의하고, `Selector`와 `Sequence`를 이용해 이 행동들의 우선순위와 실행 순서를 조합한다.15 이러한 사고방식의 전환은 AI 설계 과정을 훨씬 더 직관적으로 만들며, 기획자나 디자이너가 "이 캐릭터가 이런 상황에서는 이런 행동을 먼저 했으면 좋겠다"는 요구사항을 트리의 노드 순서를 조정하는 것만으로 직접적으로 반영할 수 있게 해준다.1

그러나 BT의 뛰어난 반응성은 공짜가 아니다. 이는 '계산 비용(computational cost)'과의 트레이드오프 결과다. FSM은 현재 활성화된 상태의 로직만 실행하면 되므로 일반적으로 연산 부하가 낮다.3 하지만 BT는 매 틱마다 루트에서부터 트리의 일부 또는 전체를 순회하며 실행 가능한 행동을 찾아야 한다.13 특히 '조건부 중단'과 같이 높은 반응성을 보장하는 기능은 현재 실행 중인 브랜치와는 별개로 다른 브랜치의 조건들을 계속해서 검사해야 하므로 추가적인 연산 비용을 발생시킨다. 따라서 BT의 뛰어난 반응성은 매 틱마다 우선순위가 높은 행동부터 다시 검사하는 '지속적인 재평가' 비용을 지불함으로써 얻어지는 것이다. 이는 수백, 수천 개의 AI 에이전트를 동시에 처리해야 하는 대규모 시뮬레이션이나 RTS 장르의 게임에서 중요한 성능 고려사항이 될 수 있다.


FSM과 BT는 상호 배타적인 관계가 아니다. 두 기술의 장단점을 이해하고 전략적으로 조합하면 더욱 견고하고 효율적인 AI 아키텍처를 구축할 수 있다.

- **상황에 따른 선택:** 모든 AI에 BT가 최선은 아니다. 문, 엘리베이터, 자동 포탑과 같이 상태가 명확하고 전이 조건이 단순한 객체에는 FSM이 오히려 더 직관적이고 효율적일 수 있다.13 반면, 주변 환경과 동적으로 상호작용하며 복잡한 의사결정을 내려야 하는 NPC와 같은 에이전트에게는 BT의 유연성과 확장성이 훨씬 더 적합하다.

- **FSM 내부에 BT 사용:** 가장 일반적인 하이브리드 접근법은 상위 레벨의 큰 상태 구분은 FSM으로 관리하고, 각 상태 내부의 세부적인 행동 로직은 BT로 구현하는 방식이다.22 예를 들어, AI의 최상위 상태를 

  `Idle`, `Patrol`, `Combat`, `Flee` 등으로 FSM으로 정의한다. 그리고 AI가 `Combat` 상태로 전이하면, 해당 상태는 내부적으로 `Combat_BehaviorTree`를 틱하기 시작한다. 이 방식은 FSM의 명확한 상태 구분 능력과 BT의 유연한 행동 표현 능력을 모두 활용할 수 있는 매우 강력하고 실용적인 아키텍처다. 이를 통해 `Combat` 상태일 때는 순찰이나 탐색과 관련된 불필요한 트리 브랜치를 아예 평가하지 않음으로써 성능상의 이점도 얻을 수 있다.


행동트리는 이론적 개념을 넘어, 주요 게임 엔진과 로보틱스 프레임워크에서 강력한 도구로 구현되어 널리 사용되고 있다. 각 프레임워크는 BT의 핵심 원리를 공유하면서도 고유한 철학과 추가 기능을 통해 차별화된 개발 경험을 제공한다.


언리얼 엔진(UE)은 행동트리를 엔진의 핵심 AI 기능으로 통합하여 제공한다. UE의 BT 시스템은 두 가지 주요 에셋, 즉 `Behavior Tree`와 `Blackboard`의 긴밀한 협력을 기반으로 설계되었다.23

- **블랙보드 (Blackboard):** 블랙보드는 AI의 '뇌' 또는 '메모리' 역할을 하는 중앙 데이터 저장소다.23

  `TargetPlayer`(플레이어 액터 참조), `LastKnownLocation`(벡터 값), `HasLineOfSight`(불리언 값)과 같은 '키(Key)'들을 정의하고, AI 컨트롤러나 다른 시스템이 이 키들의 값을 동적으로 업데이트한다. BT는 의사결정을 내리기 위해 이 블랙보드의 데이터를 참조하지만, 데이터를 직접 소유하지는 않는다. 이 구조는 BT의 실행 로직과 AI의 상태 데이터를 완벽하게 분리(decoupling)하여, 동일한 BT 로직을 서로 다른 상태를 가진 여러 AI에 재사용할 수 있게 하는 핵심적인 역할을 한다.24

- **서비스 (Services):** 서비스는 `Sequence`나 `Selector`와 같은 컴포지트 노드에 부착되어, 해당 노드의 브랜치가 활성화되어 있는 동안 지정된 주기마다 실행되는 특수한 노드다.11 주로 주변 환경을 감지하여 블랙보드의 값을 업데이트하는 데 사용된다. 예를 들어, "플레이어 탐색" 서비스는 0.2초마다 시야 검사를 수행하여 

  `HasLineOfSight` 키와 `TargetPlayer` 키의 값을 갱신할 수 있다.26 이는 전통적인 BT 시스템의 

  `Parallel` 노드 기능을 대체하며, 지속적인 상태 확인 로직을 행동 실행 로직과 깔끔하게 분리할 수 있게 해준다.

- **데코레이터 (Decorators)와 태스크 (Tasks):** UE에서 데코레이터는 주로 블랙보드의 키 값을 확인하여 브랜치의 실행 여부를 결정하는 '관문(gate)' 역할을 한다. 예를 들어, "Blackboard" 데코레이터는 `HasLineOfSight` 키가 `true`일 때만 하위 브랜치의 실행을 허용한다.27 태스크는 트리의 리프 노드로서, "Move To"나 "Play Animation"과 같은 실제 행동을 수행한다. 이 태스크들은 블랙보드의 값을 입력으로 받아 행동을 수행하고, 그 결과를 다시 블랙보드에 쓸 수 있다.

- **시각적 편집 및 디버깅:** UE의 비헤이비어 트리 에디터는 이러한 모든 노드를 시각적으로 배치하고 연결할 수 있는 강력한 그래프 기반 인터페이스를 제공한다.28 특히, 게임이 실행되는 동안 에디터 상에서 각 노드의 실행 흐름이 실시간으로 하이라이트되고, 블랙보드 패널에서 키 값의 변화를 직접 확인할 수 있어 디버깅이 매우 직관적이고 효율적이다.13

이러한 블랙보드 아키텍처는 행동트리를 단순한 '소프트웨어 컴포넌트'에서 '재사용 가능한 디자인 패턴'으로 승격시킨 핵심적인 추상화 계층이다. 블랙보드가 없다면, "플레이어가 보이는가?"라는 조건은 게임 엔진의 특정 함수(`GetPlayerCharacter()->IsVisible()`)를 직접 호출하는 코드를 노드 내에 포함해야 한다. 이는 해당 BT를 특정 게임 로직에 강하게 결합시킨다. 하지만 블랙보드를 사용하면, BT는 단지 "블랙보드의 'IsVisible' 키가 true인가?"라고만 묻는다. `IsVisible` 키의 값을 채우는 책임은 서비스와 같은 외부 시스템에 위임된다. 이로써 BT는 특정 데이터 소스로부터 완전히 독립적인, 순수한 '논리 흐름 템플릿'이 된다. 이 추상화 덕분에 동일한 `Combat.btasset`을 저격수 AI와 돌격병 AI에 공통으로 적용하면서 블랙보드에 설정된 파라미터(사정거리, 공격 딜레이 등)만 다르게 가져가는 것이 가능해진다. 이는 BT를 단순한 코딩 기법이 아닌 강력한 아키텍처 패턴으로 만드는 핵심 요소다.


유니티는 오랫동안 내장 BT 기능을 제공하지 않았기 때문에, 강력한 서드파티 에셋 생태계가 그 자리를 채워왔다. 최근에는 유니티 자체적으로도 공식 패키지를 출시하며 지원을 강화하고 있다.

- **공식 패키지 (`com.unity.behavior`):** 유니티는 최근 공식 BT 패키지를 도입했다.29 이 패키지는 그래프 기반의 시각적 에디터, 블랙보드 시스템, 커스텀 노드 생성 기능, 실시간 디버깅 등 현대적인 BT 프레임워크가 갖춰야 할 필수 기능들을 제공한다. 특히, 생성형 AI인 Unity Muse를 활용하여 자연어 프롬프트로부터 BT를 생성하는 혁신적인 기능도 포함하고 있다.29
- **서드파티 에셋 생태계:** 공식 패키지 이전부터 `Behavior Designer` 7나 `Opsive Behavior Tree` 31와 같은 고품질의 서드파티 에셋들이 유니티 개발자들 사이에서 널리 사용되어 왔다. 이들은 UE의 BT 시스템과 유사한 기능(시각적 에디터, 블랙보드, 강력한 디버깅 도구)을 제공하며, 수년간의 개발을 통해 안정성과 풍부한 기능 라이브러리를 축적해왔다.
- **구현 방식:** 유니티에서의 BT 개발은 일반적으로 C# 스크립트를 사용하여 개별 노드(`Action`, `Condition` 등)의 로직을 작성하고, 이를 에디터의 그래프 뷰에서 시각적으로 조립하는 방식으로 이루어진다.31

시각적 에디터의 존재는 단순한 개발 편의성 이상의 의미를 갖는다. 이는 '기획-개발 협업 모델'을 근본적으로 바꾸는 역할을 한다. 전통적인 AI 개발 워크플로우에서는 기획자가 AI의 행동을 문서로 상세히 명세하면, 프로그래머가 이를 코드로 구현했다. 이 과정에서는 기획자의 의도가 정확히 전달되지 않거나 구현상의 미묘한 차이가 발생하기 쉬웠다. 그러나 시각적 BT 에디터가 도입되면서, 프로그래머는 재사용 가능한 저수준 행동(Action, Condition 노드)을 '벽돌'처럼 만들고, 기획자는 이 벽돌들을 직접 조립하여 고수준의 행동 로직(BT)을 '건축'할 수 있게 되었다.1 디버깅 과정에서도 기획자는 코드 한 줄 보지 않고도 실행 흐름을 시각적으로 따라가며 "왜 AI가 여기서 이런 행동을 하지?"를 직접 파악하고 노드의 순서를 바꾸거나 파라미터를 수정하는 등의 빠른 수정을 할 수 있다.13 이는 기획과 개발 사이의 반복(iteration) 주기를 극적으로 단축시키고, 최종적으로 AI의 품질을 높이는 핵심적인 워크플로우 혁신이다.


BT는 게임 AI뿐만 아니라 로보틱스 분야에서도 복잡한 작업을 순차적으로 관리하고, 예외 상황에 대응하기 위한 강력한 도구로 각광받고 있다. 특히 로봇 운영체제의 표준인 ROS2 환경에서 그 활용도가 높다.

- **ROS2에서의 BT 활용:** ROS2의 주요 프레임워크, 특히 자율 주행 및 네비게이션을 담당하는 `Nav2`와 로봇 팔 조작(manipulation)을 위한 `MoveIt2`에서 BT는 태스크 플래닝과 실행을 제어하는 핵심 아키텍처로 채택되었다.34 예를 들어, "목표 지점까지 이동" -> "물체 탐색" -> "물체 집기" -> "지정된 위치로 이동" -> "물체 놓기"와 같은 복잡한 시나리오를 모듈화된 서브태스크의 조합으로 명확하게 표현하고 실행하는 데 매우 효과적이다.
- **`BehaviorTree.CPP` 라이브러리:** ROS 커뮤니티에서 사실상의 표준으로 사용되는 C++ 기반 BT 라이브러리다. XML 파일을 통해 트리 구조를 정의하므로, 코드를 재컴파일하지 않고도 로봇의 행동을 수정할 수 있다. 또한, 커스텀 노드를 플러그인 형태로 쉽게 추가할 수 있는 유연한 구조를 제공하여, 특정 로봇이나 센서에 맞는 행동을 쉽게 통합할 수 있다.35
- **ROS2 통합의 특징:** 로보틱스 환경에서 BT 노드는 단순히 내부 로직을 실행하는 것을 넘어, ROS2의 통신 메커니즘을 통해 다른 시스템과 상호작용한다. 예를 들어, `MoveTo` 액션 노드는 내부적으로 ROS2의 액션 클라이언트(Action Client)를 사용하여 `Nav2` 서버에 목표 지점을 전송하고, `Nav2`로부터 작업의 진행 상황(`Running`)이나 결과(`Success`/`Failure`)를 피드백 받는다.37 블랙보드는 ROS2 파라미터 서버나 특정 토픽(Topic)의 메시지를 통해 외부 시스템과 데이터를 동기화하는 데 사용될 수 있다.
- **과제:** 로보틱스 분야에서 BT를 설계할 때는 게임 AI와는 다른 차원의 고려사항이 추가된다. 하드웨어의 물리적 제약, 센서의 노이즈, 통신 지연, 그리고 무엇보다 안전성(safety)이 매우 중요하다. 또한, `ros2_control`과 같은 실시간 제어 루프와의 정밀한 상호작용이 요구된다.39 따라서 로보틱스 BT 설계는 논리적 흐름뿐만 아니라, 실제 물리 세계와의 상호작용에서 발생하는 다양한 불확실성과 제약을 모두 고려해야 하는 복잡한 엔지니어링 과제다.


행동트리는 매우 유연하고 강력한 도구지만, 그 유연성 때문에 잘못 사용될 경우 오히려 유지보수가 어려운 복잡한 구조를 낳을 수 있다. 견고하고 확장 가능한 BT를 설계하기 위해서는 검증된 설계 패턴을 따르고 흔한 안티패턴을 피하는 것이 중요하다.


- **서브트리(Subtree)를 통한 모듈화:** 복잡한 행동을 하나의 거대한 트리로 구현하는 것은 최악의 안티패턴이다. 대신, 전체 행동을 논리적인 기능 단위로 나누어 작은 서브트리로 분해해야 한다.24 예를 들어, "전투"라는 복잡한 행동은 "공격", "방어", "엄폐물 찾기", "재장전"과 같은 독립적인 서브트리로 나눌 수 있다. 이렇게 잘 정의된 서브트리는 다른 상황이나 다른 유형의 AI에서 쉽게 재사용될 수 있어 개발 효율성을 크게 높인다.24 언리얼 엔진의 

  `Run Behavior` 노드나 다른 프레임워크의 서브트리 노드가 바로 이 패턴을 지원하기 위해 존재한다.

- **데이터-로직 분리 (블랙보드 활용):** BT의 재사용성을 극대화하기 위해서는 행동 로직과 데이터를 철저히 분리해야 한다. 노드 내부에 특정 값(예: 이동 속도 `5.0`, 공격 사정거리 `1000.0`)을 직접 하드코딩하는 대신, 블랙보드 키(예: `MovementSpeed`, `AttackRange`)를 통해 값을 참조하도록 설계해야 한다.24 이렇게 하면 동일한 BT 에셋을 사용하면서 블랙보드에 다른 값을 설정하는 것만으로 다양한 성격과 능력을 가진 AI(예: 느리고 강력한 브루트, 빠르고 약한 스카웃)를 만들어낼 수 있다.

- **조건부 중단(Conditional Aborts)의 전략적 사용:** BT의 높은 반응성을 구현하는 핵심 패턴이다. 예를 들어, AI가 긴 순찰 경로를 따라 이동하는 `Sequence`를 실행하고 있다고 가정하자. 이 `Sequence`에 "적이 시야에 들어왔는가?"라는 조건을 확인하는 조건부 중단 데코레이터를 부착하면, 순찰 중 어느 시점에서든 적을 발견하는 즉시 순찰 행동을 중단하고 즉시 전투 브랜치로 전환할 수 있다.12 이는 AI가 변화하는 상황에 둔감하게 반응하는 것을 막고, 훨씬 더 지능적으로 보이게 만든다.

- **액션과 조건의 명확한 분리:** `Action` 노드는 월드의 상태를 변경하는 '행동'에만 집중해야 하고, `Condition` 노드는 월드의 상태를 확인하는 '판단'에만 집중해야 한다. `Action` 노드 내부에 복잡한 `if-else` 분기문을 넣어 의사결정을 처리하는 것은 BT의 시각적 흐름을 무의미하게 만들고 디버깅을 어렵게 하는 대표적인 안티패턴이다.15 모든 의사결정은 `Selector`, `Sequence`, `Decorator`와 같은 제어 흐름 노드를 통해 트리 구조상에서 명확하게 드러나야 한다.


안티패턴은 일반적으로 좋은 의도에서 시작되지만 장기적으로는 시스템을 취약하게 만드는 잘못된 설계 관행을 의미한다.40 BT 설계에서도 여러 흔한 안티패턴이 존재한다.

**Table 3: 행동트리 설계 안티패턴 및 해결책**

| 안티패턴 (Anti-Pattern)                                 | 문제점 (Symptoms)                                            | 해결책 (Solution)                                            |
| ------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **거대 트리 (Monolithic Tree)**                         | 낮은 가독성, 디버깅 어려움, 재사용 불가, 사소한 변경이 전체 트리에 영향 | 기능 단위로 서브트리(Subtree)로 분해. `Run Behavior` (UE) 또는 서브트리 노드를 사용하여 계층적으로 구성. |
| **데이터 하드코딩 (Hardcoded Values)**                  | 재사용성 저하, 유연성 부족, 값 변경 시 BT 에셋을 직접 수정해야 함. | 블랙보드(Blackboard)를 사용하여 데이터를 외부화. 노드는 블랙보드 키를 통해 값을 참조. |
| **액션 노드 내 복잡한 로직 (Complex Logic in Actions)** | BT의 시각적 흐름이 실제 로직을 반영하지 못함. 디버깅 시 코드 내부를 파고들어야 함. | 조건 판단은 데코레이터(Decorator), 지속적인 상태 확인은 서비스(Service)로 분리. 액션은 최소 단위의 실행에만 집중. |
| **실패 경로 미고려 (Ignoring Failure Paths)**           | '냉장고 문' 문제처럼, 예기치 않은 상태에서 시스템이 멈추거나 리소스가 정리되지 않음. | `Selector`(Fallback)나 `Parallel` 노드, 또는 `Force Success/Failure` 데코레이터를 활용하여 실패 시의 대체 행동이나 정리(cleanup) 로직을 명시적으로 설계. |

- **거대 트리 (Monolithic Tree):** 모든 가능한 행동 로직을 하나의 거대한 BT 파일에 담으려는 시도다. 이는 BT의 가장 큰 장점인 모듈성과 재사용성을 완전히 파괴하며, 누구도 전체 구조를 이해할 수 없는 스파게티 코드를 만들어낸다.24
- **과도한 틱 연산 (Performance Bottlenecks):** 모든 노드, 특히 복잡한 연산을 수행하는 노드를 매 프레임마다 틱하게 만들면 심각한 성능 저하를 유발할 수 있다.7 언리얼 엔진의 `Service`와 같은 기능의 실행 주기(tick interval)를 0.1초나 0.5초 등으로 적절히 조절하고, 자주 확인하지만 값이 자주 변하지 않는 조건의 결과는 캐싱하는 등의 최적화 기법을 적극적으로 사용해야 한다.24

이러한 BT 안티패턴들은 대부분 기술적인 오류라기보다는 '설계 철학의 부재'에서 비롯된다. 개발자가 BT를 단순히 거대한 `switch-case` 문 43 이나 `if-else` 블록 42의 시각적 버전으로만 간주할 때 이러한 문제가 발생한다. 올바른 BT 설계를 위해서는, 각 서브트리를 독립적인 '함수'로, 블랙보드를 '파라미터'로, `Action` 노드를 '부수 효과(side effect)'가 있는 최소 단위 작업으로 여기는 소프트웨어 공학적 설계 원칙(단일 책임 원칙, 관심사 분리 등)을 트리 구조에 적용해야 한다.


견고한 BT 설계를 위해서는 성공 경로뿐만 아니라 모든 잠재적인 실패 경로를 신중하게 고려해야 한다. 이를 잘 보여주는 고전적인 예시가 '냉장고 문' 문제다.4

- **문제 상황:** "냉장고 문 열기 -->> 맥주 꺼내기 -->> 냉장고 문 닫기"라는 일련의 행동을 `Sequence` 노드로 구현했다고 가정하자. 만약 중간 단계인 "맥주 꺼내기" 액션이 실패하면(예를 들어, 냉장고에 맥주가 없어서), `Sequence` 노드는 즉시 `Failure`를 반환하고 실행을 중단한다. 그 결과, 마지막 단계인 "냉장고 문 닫기" 액션은 결코 실행되지 않고, 냉장고 문은 영원히 열려있는 상태로 남게 된다.4
- **해결 방안:** 이 문제를 해결하기 위해서는 실패 시에도 반드시 실행되어야 하는 '정리(cleanup)' 로직을 보장하는 구조를 만들어야 한다. `4`에서 제안하는 한 가지 견고한 해결책은 다음과 같다.
  1. 최상위에 `Sequence` 노드를 배치하고, 그 자식으로 `OpenDoor` 액션과 `Fallback` 노드를 둔다.
  2. `Fallback` 노드의 자식으로, 첫 번째에는 `(Sequence: GrabBeer, CloseFridge)`를, 두 번째에는 `CloseFridge` 액션만을 둔다.
- **작동 방식:**
  - **성공 시나리오:** `OpenDoor`가 성공하고, `GrabBeer`도 성공하면, `Fallback`의 첫 번째 자식인 `Sequence`가 실행되어 `CloseFridge`까지 성공적으로 마친다. `Fallback`은 `Success`를 반환하고, 전체 트리도 `Success`로 끝난다.
  - **실패 시나리오:** `OpenDoor`는 성공했지만 `GrabBeer`가 실패하면, `Fallback`의 첫 번째 자식인 `Sequence`가 `Failure`를 반환한다. 그러면 `Fallback`은 다음 자식인 `CloseFridge` 액션을 시도하고, 이 액션이 성공하면 `Fallback`은 `Success`를 반환한다. 결과적으로 맥주를 꺼내지는 못했지만, 문은 안전하게 닫히게 된다.

이 '냉장고 문' 문제는 BT가 FSM과 달리 '실행 컨텍스트'를 명시적으로 관리하지 않기 때문에 발생하는 전형적인 문제다. FSM에서는 '문이 열린 상태(DoorOpenState)'라는 명시적인 상태가 존재하며, 이 상태에서는 특정 행동이 강제되거나 타임아웃 등의 메커니즘으로 안전한 상태로 복귀할 수 있다. 하지만 BT는 상태가 없기 때문에, `Sequence`가 실패하면 '문이 열려 있다'는 컨텍스트는 그냥 소멸하고 제어는 부모로 넘어가 버린다. 이 때문에 개발자는 실패 경로를 명시적으로 설계하여 '컨텍스트 정리(context cleanup)'를 수동으로 보장해야 한다. 이는 BT의 유연성이 공짜가 아니며, 상태를 명시적으로 관리하지 않는 데서 오는 책임을 개발자가 직접 져야 함을 명확히 보여준다.


수작업으로 정교한 행동트리를 설계하는 것은 여전히 많은 시간과 노력을 요구하는 장인(artisan)의 영역에 가깝다. 현대 AI 연구의 흐름은 이러한 과정을 자동화하고, 데이터로부터 최적의 행동을 학습시키기 위해 기계학습(ML) 기술을 BT에 접목하는 방향으로 나아가고 있다.


강화학습(RL)은 에이전트가 환경과의 상호작용을 통해 보상을 최대화하는 행동 정책을 학습하는 패러다임이다. 이를 BT 최적화에 적용하려는 시도는 크게 두 가지 방향으로 진행된다.

- **파라미터 최적화:** 이 접근법에서는 BT의 전체적인 구조는 인간 전문가가 설계하고, 각 노드에 포함된 구체적인 파라미터(예: `Condition` 노드의 거리 임계값, `Wait` 액션의 대기 시간, `Move` 액션의 속도 등)를 RL 알고리즘을 통해 자동으로 튜닝한다. 이는 BT의 높은 해석 가능성과 모듈성을 그대로 유지하면서, 인간이 일일이 조정하기 어려운 미세한 파라미터들을 데이터 기반으로 최적화할 수 있는 매우 실용적이고 강력한 하이브리드 방식이다.44 한 연구에서는 블랙박스 최적화 기법인 CMA-ES를 사용하여 시뮬레이션 환경에서 로봇 팔을 제어하는 BT의 파라미터를 성공적으로 학습시켰고, 이를 실제 로봇에 추가 학습 없이 이전(sim-to-real transfer)하는 데 성공했다.44

- **구조적 학습:** 더 나아가, RL을 이용해 BT의 구조 자체를 학습하려는 연구도 활발히 진행되고 있다. 예를 들어, `Selector` 노드의 자식들 중 어떤 것을 먼저 시도하는 것이 장기적으로 가장 높은 보상을 가져올지 학습하여 자식의 순서를 동적으로 바꾸거나 46, 

  `Learning Selector`와 같은 새로운 노드 유형을 도입하여 Q-러닝과 같은 알고리즘으로 최적의 자식 노드를 선택하게 하는 방식이다.10

BT와 ML의 결합은 '설명 가능한 AI(Explainable AI, XAI)'라는 현대 AI의 가장 큰 난제를 해결하는 실용적인 접근법을 제시한다. 심층 신경망(Deep Neural Network, DNN)으로 대표되는 순수 RL 정책은 왜 특정 상황에서 그런 행동을 했는지 설명하기가 거의 불가능한 '블랙박스' 모델이다. 이는 특히 안전이 중요한 로보틱스나 예측 가능성이 중요한 게임 디자인에서 치명적인 단점이 될 수 있다. 하지만 RL을 BT의 파라미터 '튜닝'에 사용하면, RL은 '무엇을 할지(what to do)'를 처음부터 배우는 것이 아니라, 이미 인간이 설계한 행동 구조 내에서 '어떻게 더 잘할지(how to do it better)'를 최적화하는 역할을 하게 된다. 만약 AI가 이상하게 행동하더라도, 개발자는 BT의 실행 흐름을 시각적으로 디버깅하며 "아, RL이 '공격 거리' 파라미터를 너무 짧게 학습했구나"와 같이 그 원인을 직관적으로 파악하고 수정할 수 있다. 이는 블랙박스 모델을 해석 가능한 프레임워크의 '튜닝 도구'로 활용함으로써, 학습의 강력함과 명시적 설계의 안정성을 모두 얻는 매우 현실적인 절충안이다.


BT 설계를 자동화하는 또 다른 흐름은 인간의 시연이나 지시로부터 직접 트리를 생성하는 것이다.

- **모방학습 (Imitation Learning, IL):** 모방학습은 전문가(주로 인간)의 시연 데이터를 모방하여 행동 정책을 학습하는 기법이다. `ILBERT` 프레임워크와 같은 연구에서는, 인간이 몇 차례 작업을 시연하면 그 과정에서 행동의 순서, 그리고 각 행동이 시작되고 끝나는 조건들을 추출하여 초기 BT를 자동으로 생성한다.47 이후 로봇이 실행 중에 실패하면, 인간과의 상호작용을 통해 BT를 점진적으로 수정하고 개선해 나간다.

- **거대 언어 모델 (Large Language Models, LLM):** 최근에는 "로봇, 테이블 위의 캔을 집어서 쓰레기통에 버려줘"와 같은 자연어 명령을 이해하여 이를 수행하기 위한 BT의 구조(주로 XML 형식)를 LLM이 직접 생성하는 연구가 큰 주목을 받고 있다.49

  `BTGenBot` 프로젝트는 경량 LLM을 특정 작업 데이터셋으로 파인튜닝(fine-tuning)하여, 자연어 지시로부터 로봇 제어용 BT를 생성하고 이를 실제 로봇에서 검증하는 구체적인 사례를 보여준다.35

LLM을 통한 BT 생성은 AI 개발의 패러다임을 '구현(Implementation)'에서 '명세(Specification)'로 전환시킬 엄청난 잠재력을 가진다. 미래의 개발자나 디자이너는 더 이상 노드를 하나하나 끌어다 놓거나 코드를 작성할 필요 없이, 원하는 AI의 행동을 자연어로 상세하게 '명세'하기만 하면 될지 모른다. 그러면 LLM이 그 명세를 만족하는 가장 적절한 BT 구조를 자동으로 생성해줄 것이다. 이는 개발의 추상화 수준을 한 단계 끌어올리는 것으로, 마치 고급 프로그래밍 언어가 어셈블리어를 대체했듯이, 자연어 기반 명세가 직접적인 BT 설계를 대체할 수 있음을 시사한다. 이 시나리오에서 BT는 인간의 '의도'와 기계가 실행 가능한 '로직' 사이를 잇는 완벽한 '중간 표현(intermediate representation)'으로서 기능하게 된다. 이는 AI 개발의 민주화를 가속화하고 생산성을 극적으로 향상시킬 수 있는 미래의 청사진이다.


엔드-투-엔드(end-to-end) 딥러닝이 모든 AI 문제를 해결할 것이라는 일부의 예측과 달리, BT는 데이터 기반 AI 시대에 오히려 더 중요한 역할을 맡게 될 것이다.

- **하이브리드 AI의 허브 (Hub for Hybrid AI):** BT는 규칙 기반의 명시적 로직과 데이터 기반의 학습된 로직을 결합하는 이상적인 '허브' 아키텍처를 제공한다. 예를 들어, AI의 전체적인 전략적 흐름(예: 공격할 것인가, 후퇴할 것인가)은 BT의 `Selector`를 통해 명시적으로 제어하되, "최적의 엄폐물을 찾아라"와 같은 특정 `Action` 노드의 내부 결정은 심층 신경망 모델이 내리도록 위임할 수 있다.2
- **해석 가능한 AI (XAI)의 발판:** BT는 신경망과 같은 블랙박스 모델의 의사결정 과정을 인간이 이해할 수 있도록 설명하는 데 중요한 도구가 될 수 있다. 신경망 정책이 내린 결정을 BT의 조건이나 행동으로 변환하여 시각화함으로써, 왜 AI가 특정 행동을 선택했는지 그 이유를 추적하고 분석하는 데 도움을 줄 수 있다.24

결론적으로, BT는 모듈성, 해석 가능성, 디버깅 용이성, 그리고 디자이너 친화성이라는, 순수 데이터 기반 모델이 쉽게 대체할 수 없는 강력한 장점들을 가지고 있다.2 BT는 앞으로도 AI 개발의 핵심적인 도구로 남을 것이며, 그 역할은 더욱 정교한 하이브리드 AI 시스템을 구축하기 위한 필수적인 '중간 계층 아키텍처'로서 더욱 중요해질 것이다.


행동트리는 지난 20년간 게임 AI와 로보틱스 분야에서 가장 중요한 의사결정 아키텍처 중 하나로 발전해왔다. 유한 상태 머신(FSM)의 확장성 문제를 해결하기 위해 등장한 BT는 모듈성, 확장성, 재사용성, 그리고 직관적인 시각적 설계라는 본질적인 강점을 바탕으로 복잡한 AI 행동을 설계하는 산업 표준으로 자리매김했다. 틱(tick) 기반의 반응형 실행 모델과 `Success`, `Failure`, `Running` 상태를 통한 비동기 작업 처리는 BT가 동적인 환경에 민첩하게 대응할 수 있는 능력의 근간을 이룬다.

그러나 BT의 유연성은 설계자에게 더 큰 책임을 요구한다. 상태 컨텍스트의 명시적 관리 부재로 인해 실패 경로와 정리(cleanup) 로직을 신중하게 설계해야 하는 복잡성이 따르며, 매 틱마다 트리를 재평가하는 구조는 잠재적인 연산 비용 문제를 안고 있다. '거대 트리'나 '데이터 하드코딩'과 같은 안티패턴은 BT의 핵심 철학을 이해하지 못했을 때 쉽게 빠질 수 있는 함정이다.

행동트리의 미래는 기계학습과의 깊은 융합에 있다. 강화학습을 통한 파라미터 최적화는 BT의 성능을 데이터 기반으로 극대화하며, 모방학습과 거대 언어 모델을 통한 자동 생성 기술은 BT 설계의 패러다임을 수작업 '공예'에서 자동화된 '생산'으로 전환시키고 있다. 이 과정에서 BT는 학습된 블랙박스 정책에 구조와 해석 가능성을 부여하는 이상적인 뼈대로서의 역할을 수행할 것이다.

결론적으로, 행동트리는 사라지지 않을 것이다. 오히려 더욱 정교하고 지능적인 자율 에이전트를 구축하기 위한 필수적인 '중간 계층 아키텍처'로서 그 중요성이 더욱 커질 전망이다. BT는 인간 디자이너의 명시적 의도와 기계학습 모델의 강력한 패턴 인식 능력 사이를 잇는 핵심적인 다리 역할을 수행하며, 앞으로도 오랫동안 지능형 에이전트 설계의 중심에 굳건히 서 있을 것이다.


1. Behavior tree (artificial intelligence, robotics and control) - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Behavior_tree_(artificial_intelligence,_robotics_and_control)
2. A behavior tree including selector, sequence, condition and action nodes - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/figure/A-behavior-tree-including-selector-sequence-condition-and-action-nodes_fig4_312869797
3. State Machines vs Behavior Trees: designing a decision-making architecture for robotics, accessed July 23, 2025, https://www.polymathrobotics.com/blog/state-machines-vs-behavior-trees
4. Introduction to BTs | BehaviorTree.CPP, accessed July 23, 2025, https://www.behaviortree.dev/docs/learn-the-basics/bt_basics/
5. Behaviour Trees versus State Machines | Queen Of Squiggles's Blog, accessed July 23, 2025, https://queenofsquiggles.github.io/guides/fsm-vs-bt/
6. 액터 틱 | 언리얼 엔진 5.6 문서 | Epic Developer Community, accessed July 23, 2025, https://dev.epicgames.com/documentation/ko-kr/unreal-engine/actor-ticking-in-unreal-engine
7. UNSEEN 5. 게임 알고리즘 (2): FSM, BT - 나는 뉴비다 개발자편, accessed July 23, 2025, https://dev-nicitis.tistory.com/54
8. 행동 트리를 활용한 AI 만들기, accessed July 23, 2025, https://maplestoryworlds-creators.nexon.com/ko/docs?postId=562
9. Behavior Tree를 알아봅시다 - line engineering - LINE Plus, accessed July 23, 2025, https://engineering.linecorp.com/ko/blog/behavior-tree/
10. A Reinforcement Learning Behavior Tree Framework for Game AI - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/308464470_A_Reinforcement_Learning_Behavior_Tree_Framework_for_Game_AI
11. [Study] Part 2 - 행동 트리 모델의 이해 (11/15), accessed July 23, 2025, https://j1y00h4.tistory.com/115
12. Behavior Tree Flow - Opsive, accessed July 23, 2025, https://opsive.com/support/documentation/behavior-designer/what-is-a-behavior-tree/behavior-tree-flow/
13. Should I use behavior trees or Finite state machines? : r/unrealengine - Reddit, accessed July 23, 2025, https://www.reddit.com/r/unrealengine/comments/1eskk42/should_i_use_behavior_trees_or_finite_state/
14. 시스템 아키텍처 패턴 언어 - 시스템 설계 문제에 대한 재사용 가능한 해결책을 체계화한 패턴 언어. 패턴 카탈로그, 패턴 조합, 안티패턴, 패턴 적용 컨텍스트 및 패턴 기반 설계 방법론을 다룸. | Flashcards World, accessed July 23, 2025, https://flashcards.world/flashcards/sets/9f6bd960-5f21-4938-a3c8-d66bfd15aab1/
15. Behavior Tree (1) - 기초 - 게임 클라 개발 - 티스토리, accessed July 23, 2025, https://tsyang.tistory.com/123
16. The Behavior Tree Starter Kit - Game AI Pro, accessed July 23, 2025, https://www.gameaipro.com/GameAIPro/GameAIPro_Chapter06_The_Behavior_Tree_Starter_Kit.pdf
17. [SWTT] 유니티의 Behaviour-tree (행동트리) 개념 소개 및 'thekiwicoder Behaviour-tree editor' 사용법 - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=UysoNIRgks8
18. Behavior Tree Tutorial - Community & Industry Discussion - Unreal Engine Forums, accessed July 23, 2025, https://forums.unrealengine.com/t/behavior-tree-tutorial/105
19. Behavior Tree 개념 및 동작 - 그냥 그런 블로그 - 티스토리, accessed July 23, 2025, https://lifeisforu.tistory.com/327
20. 2.4 Hierarchical Finite State Machine (HFSM) & Behavior Tree (BT) - Stanford University, accessed July 23, 2025, https://web.stanford.edu/class/cs123/lectures/CS123_lec08_HFSM_BT.pdf
21. Comparison between Behavior Trees and Finite State Machines - arXiv, accessed July 23, 2025, https://arxiv.org/html/2405.16137v1
22. How best to use behavior trees? : r/gamedev - Reddit, accessed July 23, 2025, https://www.reddit.com/r/gamedev/comments/13gwgeo/how_best_to_use_behavior_trees/
23. Behavior Trees in Unreal Engine - Epic Games Developers, accessed July 23, 2025, https://dev.epicgames.com/documentation/en-us/unreal-engine/behavior-trees-in-unreal-engine
24. Rediscovering Behavior Trees: A Powerful AI Tool for Modern Games - Wayline, accessed July 23, 2025, https://www.wayline.io/blog/rediscovering-behavior-trees-ai-tool
25. Do you use Behavior Tree or just write code? : r/Unity3D - Reddit, accessed July 23, 2025, https://www.reddit.com/r/Unity3D/comments/1ifbfk8/do_you_use_behavior_tree_or_just_write_code/
26. Unreal Engine Behavior Tree Node Reference: Services - Epic Games Developers, accessed July 23, 2025, https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-behavior-tree-node-reference-services
27. [UE4 입문] Behavior Tree #2 - erikanes's Macchiato - 티스토리, accessed July 23, 2025, https://erikanes.tistory.com/325
28. Behavior Tree in Unreal Engine - Quick Start Guide - Epic Games Developers, accessed July 23, 2025, https://dev.epicgames.com/documentation/en-us/unreal-engine/behavior-tree-in-unreal-engine---quick-start-guide
29. Behavior - Unity - Manual, accessed July 23, 2025, https://docs.unity3d.com/6000.0/Documentation/Manual/com.unity.behavior.html
30. I created an Open-Source Behavior Tree package for Unity : r/Unity3D - Reddit, accessed July 23, 2025, https://www.reddit.com/r/Unity3D/comments/19462gu/i_created_an_opensource_behavior_tree_package_for/
31. Build AI Behaviour Trees FAST in Unity with C# - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=aR6wt5BlE-E
32. How to Code Behaviour Trees in Unity C# - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=lusROFJ3_t8&pp=0gcJCfwAo7VqN5tD
33. Building Utility Decisions into Your Existing Behavior Tree - Game AI Pro, accessed July 23, 2025, http://www.gameaipro.com/GameAIPro/GameAIPro_Chapter10_Building_Utility_Decisions_into_Your_Existing_Behavior_Tree.pdf
34. Behavior Trees for ROS2 | ROS2 Developers Open Class #162 - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=KO4S0Lsba6I
35. BTGenBot: a system to generate behavior trees for robots using lightweight - GitHub, accessed July 23, 2025, https://github.com/AIRLab-POLIMI/BTGenBot
36. Getting Started with Custom Behavior Tree Plugins in Nav2: Seeking Guidance and Resources : r/ROS - Reddit, accessed July 23, 2025, https://www.reddit.com/r/ROS/comments/1ii9qso/getting_started_with_custom_behavior_tree_plugins/
37. py_trees_ros_tutorials 2.1.0 documentation - PyTrees ROS Tutorials - Read the Docs, accessed July 23, 2025, https://py-trees-ros-tutorials.readthedocs.io/en/devel/tutorials.html
38. Writing a New Behavior Tree Plugin - Nav2 1.0.0 documentation, accessed July 23, 2025, https://docs.nav2.org/plugin_tutorials/docs/writing_new_bt_plugin.html
39. Example 7: Full tutorial with a 6DOF robot - ROS2_Control, accessed July 23, 2025, https://control.ros.org/rolling/doc/ros2_control_demos/example_7/doc/userdoc.html
40. 정적 분석 vs. 숨겨진 안티패턴: 무엇을 보고 무엇을 놓치는가, accessed July 23, 2025, https://www.in-com.com/ko/blog/static-analysis-vs-hidden-anti-patterns-what-it-sees-and-what-it-misses/
41. Anti-Patterns vs. Patterns: What Is the Difference? – BMC Software | Blogs, accessed July 23, 2025, https://www.bmc.com/blogs/anti-patterns-vs-patterns/
42. Behavior Trees – Breaking the cycle of misuse - Taking Initiative, accessed July 23, 2025, https://takinginitiative.net/2020/01/07/behavior-trees-breaking-the-cycle-of-misuse/
43. Behavior Trees : r/gamedev - Reddit, accessed July 23, 2025, https://www.reddit.com/r/gamedev/comments/2dvb1s/behavior_trees/
44. Learning of Parameters in Behavior Trees for Movement Skills, accessed July 23, 2025, https://arxiv.org/abs/2109.13050
45. Improving the performance of Learned Controllers in Behavior Trees using Value Function Estimates at Switching Boundaries - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2305.18903
46. Improving the Performance of Backward Chained Behavior Trees that use Reinforcement Learning - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2112.13744
47. Interactively learning behavior trees from imperfect human demonstrations - Frontiers, accessed July 23, 2025, https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2023.1152595/full
48. Interactively learning behavior trees from imperfect human demonstrations - PMC, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC10368948/
49. arxiv.org, accessed July 23, 2025, https://arxiv.org/html/2403.12761v1
50. [2302.12927] Robot Behavior-Tree-Based Task Generation with Large Language Models, accessed July 23, 2025, https://arxiv.org/abs/2302.12927
51. Optimizing Interpretable Decision Tree Policies for Reinforcement Learning - arXiv, accessed July 23, 2025, https://arxiv.org/html/2408.11632v1
52. Behaviour Trees: The Cornerstone of Modern Game AI | AI 101 - YouTube, accessed July 23, 2025, https://www.youtube.com/watch?v=6VBCXvfNlCM


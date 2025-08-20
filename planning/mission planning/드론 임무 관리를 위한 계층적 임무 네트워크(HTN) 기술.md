# 드론 임무 관리를 위한 계층적 임무 네트워크(HTN) 기술

## 1. 목표 지향에서 절차 수행으로의 패러다임 전환

자동화된 계획(automated planning) 분야에서 계층적 임무 네트워크(Hierarchical Task Network, HTN)는 고전적인 계획 패러다임과는 근본적으로 다른 접근 방식을 제시한다. STRIPS와 같은 고전적 플래너가 초기 상태(initial state)에서 목표 상태(goal state)로 전환하는 일련의 행동(action) 순서를 찾는 데 집중하는 반면, HTN 플래너는 상위 수준의 추상적인 임무(task)를 점진적으로 구체화하고 분해하여 실행 가능한 기본 행동들의 순서로 만드는 과정을 통해 계획을 수립한다.1 이는 단순히 '무엇을 달성할 것인가'의 문제를 넘어 '어떻게 수행할 것인가'라는 절차적 지식(procedural knowledge)을 계획 과정의 핵심에 두는 패러다임의 전환을 의미한다.

이러한 접근법은 복잡하고 다단계의 절차를 요구하는 드론 임무 관리에 특히 유용하다. 드론의 임무는 단순한 상태 변화의 집합이 아니라, 정찰, 배송, 협력 작업 등과 같이 명확한 절차와 제약 조건을 갖는 구조화된 활동이기 때문이다. 본 보고서는 드론 임무 관리를 위한 HTN 계획 기술의 핵심 원리, 형식적 정의, 타 기술과의 비교 분석 및 실제 적용 방안을 심도 있게 고찰하고자 한다. 이를 통해 HTN이 현대 자율 시스템, 특히 무인 항공기(UAV)의 지능적 임무 수행 능력을 어떻게 향상시킬 수 있는지에 대한 기술적 통찰을 제공하는 것을 목표로 한다.

### 1.1  HTN 프레임워크의 해부

HTN의 핵심은 임무를 계층적으로 구조화하고, 이를 분해하는 방법을 정의하는 데 있다. 이 프레임워크는 다음과 같은 핵심 구성요소로 이루어진다.1

- **원시 임무 (Primitive Tasks):** 이는 시스템이 직접 실행할 수 있는 가장 기본적인 행동 단위이다. 원시 임무는 고전적 계획에서의 연산자(operator)와 유사하며, 행동을 수행하기 위한 전제조건(preconditions)과 행동 수행 후 세계 상태의 변화를 기술하는 효과(effects)를 가진다.1 예를 들어, 드론의 원시 임무는 `takeoff()`, `moveTo(waypoint)`, `activate_sensor(camera)`, `land()` 등이 될 수 있다. 이들은 더 이상 분해될 수 없는 계획의 최종 실행 단위이다.
- **복합 임무 (Compound Tasks):** 이는 직접 실행할 수 없는 추상적이거나 복합적인 상위 수준의 임무를 의미한다.1 복합 임무는 그 자체로 세계의 상태를 바꾸지 않으며, 반드시 하나 이상의 하위 임무 네트워크로 분해(decompose)되어야 한다. 드론 임무에서 `perform_reconnaissance(area)`(지역 정찰 수행), `deliver_package(destination)`(목적지로 소포 배송) 등은 대표적인 복합 임무이다. 이들은 계획의 계층적 구조를 형성하는 뼈대가 된다.
- **메서드 (Methods):** 메서드는 HTN의 핵심적인 절차적 지식을 담고 있는 요소로, 특정 복합 임무를 어떻게 더 작은 하위 임무들의 네트워크로 분해할 수 있는지에 대한 한 가지 방법을 정의한다.4 하나의 복합 임무는 여러 개의 메서드를 가질 수 있으며, 이는 동일한 목표를 달성하기 위한 다양한 전략이나 절차가 존재함을 의미한다.4 예를 들어, `perform_reconnaissance(area)`라는 복합 임무는 '신속 스캔'을 위한 `fast_scan` 메서드와 '정밀 스캔'을 위한 `detailed_scan` 메서드를 가질 수 있다. 플래너는 현재 상황과 제약 조건에 가장 적합한 메서드를 선택하여 임무를 분해한다.
- **임무 네트워크 (Task Networks):** 임무 네트워크는 임무들의 집합과 그들 사이의 제약 조건(주로 순서 제약)으로 구성된다.1 메서드는 복합 임무를 단순한 순차적 행동 목록이 아닌, 잠재적으로 부분 순서(partially ordered)를 갖는 임무 네트워크로 분해할 수 있다.11 이는 하위 임무들 간의 병렬 수행 가능성을 열어주어 계획의 유연성과 효율성을 높인다. 예를 들어, 다수의 드론이 각자 다른 구역을 동시에 정찰하는 계획은 부분 순서 임무 네트워크로 자연스럽게 표현될 수 있다.

### 1.2  HTN 계획 프로세스: 메커니즘적 관점

HTN 플래너의 작동 원리는 초기 임무 네트워크에서 시작하여 모든 복합 임무가 원시 임무로 분해될 때까지 재귀적으로 메서드를 적용하는 과정으로 요약할 수 있다.4 이 과정은 가능한 모든 분해 방법을 탐색하는 과정이며, 탐색 트리의 노드는 부분 계획(task network)에, 간선은 분해 단계(decomposition step)에 해당한다.12

플래너는 주어진 초기 상태와 초기 임무 네트워크를 입력받아, 해결해야 할 복합 임무를 선택한다. 그리고 해당 임무를 분해할 수 있는 여러 메서드 중 하나를 비결정론적으로 선택하여 적용한다. 이 과정에서 복합 임무는 메서드에 정의된 하위 임무 네트워크로 대체된다. 이 과정을 모든 복합 임무가 사라지고 실행 가능한 원시 임무들의 순서만 남을 때까지 반복한다. 최종적으로 생성된 원시 임무 시퀀스가 바로 계획의 해(solution)가 된다.1

이 탐색 과정에는 크게 두 가지 접근 방식이 존재한다.

1. **계획 공간 탐색 (Plan-Space Search):** SIPE-2, O-Plan과 같은 플래너들이 사용하는 방식으로, 부분적으로 순서가 정해진 계획들의 공간을 탐색한다.12 이 방식은 계획의 결함을 해결하기 위해 새로운 임무나 제약 조건을 추가하며 진행된다. 임무 간의 순서를 유연하게 결정할 수 있다는 장점이 있지만, 탐색 과정에서 현재 상태에 대한 정보가 제한적이라는 단점이 있다.
2. **전향적 상태 공간 탐색 (Progression Search):** SHOP2와 같은 플래너들이 사용하는 방식으로, 계획의 앞부분부터 순차적으로 생성하며 세계 상태를 전진(progress)시킨다.12 이 방식은 현재 상태를 명확하게 알 수 있어 고전적 계획의 휴리스틱 함수를 적용하기 용이하지만, 계획 공간 탐색에 비해 임무 순서 결정의 유연성은 떨어진다.

HTN의 진정한 가치는 단순히 임무를 계층적으로 나누는 것을 넘어, 도메인 전문가의 절차적 지식을 명시적으로 인코딩하고 활용하는 데 있다. 고전적 계획이 원자적 행동들의 집합과 목표 상태만으로 처음부터 모든 절차를 '발견'해야 하는 반면, HTN은 복잡한 임무를 해결하는 방법에 대한 '조리법(recipe)'에 해당하는 메서드를 제공받는다.7 이 절차적 지식은 매우 강력한 휴리스틱으로 작용하여, 전문가가 "타당하다"고 정의한 행동 순서만을 고려하도록 탐색 공간을 극적으로 가지치기한다.10

드론 임무 관리의 관점에서 이는 결정적인 이점을 제공한다. 임무 설계자는 "격자형 탐색 방법", "역풍을 고려한 착륙 접근법"과 같은 숙련된 조종사의 지식을 메서드로 직접 인코딩할 수 있다. 이를 통해 생성된 계획은 기술적으로 올바를 뿐만 아니라, 실제 운용상으로도 타당하며 인간이 사용하는 전략과 유사해진다.2 이는 자율 시스템에서 가장 중요한 과제 중 하나인 시스템 행동의 예측 가능성과 인간 감독관의 신뢰 확보 문제를 해결하는 데 기여한다.15

## 2.  형식주의와 표현력-결정가능성 스펙트럼

HTN 계획을 엄밀하게 분석하고 다른 패러다임과 비교하기 위해서는 수학적으로 명확하게 정의된 형식적 모델이 필수적이다. HTN의 형식주의는 다양한 연구에서 조금씩 다르게 정의되지만, 핵심적인 개념들은 공통적으로 나타난다.5

### 2.1  HTN 계획의 형식적 정의

HTN 계획 문제는 일반적으로 초기 상태, 초기 임무 네트워크, 그리고 도메인 지식의 세 부분으로 정의된다.

- **상태와 연산자 (State and Operators):** 상태 $s$는 참인 명제(ground atom)들의 집합으로 표현된다. 원시 임무, 즉 연산자 $o$는 이름, 전제조건, 효과로 구성된 튜플로 정의할 수 있다.
  $$
  o = (\text{name}(o), \text{pre}(o), \text{eff}(o))
  $$
  여기서 $\text{pre}(o)$는 연산자 적용을 위해 상태 $s$에서 반드시 참이어야 하는 조건들의 집합이며, $\text{eff}(o)$는 연산자 적용 후 상태에 추가(add-list)되거나 삭제(delete-list)되는 명제들의 집합이다.

- **계획 도메인 (Planning Domain):** HTN 계획 도메인 $D$는 연산자들의 집합 $O$와 메서드들의 집합 $M$으로 구성된 튜플이다.
  $$
  D = (O, M)
  $$
  **임무 네트워크 (Task Network):** 임무 네트워크 $w$는 임무 식별자들의 집합 $T$, $T$에 대한 엄격한 부분 순서(strict partial order) 관계 $\prec$, 그리고 각 식별자를 임무 이름에 매핑하는 함수 $\alpha$로 구성된 튜플로 정의된다.11
  $$
  w = (T, \prec, \alpha)
  $$
  여기서 $\prec$는 임무들 간의 선행 관계를 나타낸다.

- **메서드 (Method):** 메서드 $m$은 이름, 이 메서드가 분해하는 복합 임무, 그리고 분해 결과로 생성되는 하위 임무 네트워크로 구성된 튜플이다.10
  $$
  m = (\text{name}(m), \text{task}(m), tn_m)
  $$
  여기서 $\text{task}(m)$은 복합 임무 이름이고 $tn_m$은 해당 임무를 대체할 임무 네트워크이다.

- **계획 문제 (Planning Problem):** HTN 계획 문제 $P$는 초기 상태 $s_0$, 초기 임무 네트워크 $w_0$, 그리고 계획 도메인 $D$로 구성된 튜플이다.10
  $$
  P = (s_0, w_0, D)
  $$
  **해 (Solution):** 계획 문제 $P$에 대한 해(solution)인 계획 $\pi$는 초기 상태 $s_0$에서 시작하여 초기 임무 네트워크 $w_0$를 성공적으로 분해하고 실행할 수 있는 원시 임무(연산자)들의 순차열(sequence) $\pi = \langle o_1, o_2,..., o_k \rangle$ 이다.

### 2.2  표현력과 계산 복잡도

HTN 계획은 이론적으로 STRIPS 스타일의 고전적 계획보다 표현력이 더 높다(strictly more expressive).1 이는 HTN이 이론적 모델에서 재귀(recursion)와 무한한 임무 레이블을 사용할 수 있기 때문인데, 이로 인해 HTN은 문맥 자유 문법(context-free grammar)에, STRIPS는 정규 문법(regular grammar)에 대응되는 것으로 분석된다.18

그러나 이러한 강력한 표현력은 큰 대가를 수반한다. 일반적인(unrestricted) HTN 계획 문제는 결정 불가능(undecidable)하다.11 즉, 해가 존재하는지를 판별하는 알고리즘이 존재하지 않을 수 있다. 심지어 상태 공간이 유한하고 변수가 없는 경우에도, 임무 네트워크가 순서가 정해지지 않은 두 개 이상의 복합 임무를 포함할 수 있다면 문제는 여전히 결정 불가능하다.18 따라서 이 이론적 모델은 실제 시스템에 그대로 적용하기 어렵다.

실용적이고 해결 가능한 플래너를 만들기 위해, HTN 모델에는 다음과 같은 제약들이 가해진다 1:

1. 메서드 간의 재귀적 호출을 금지하거나, 도메인 지식이 비순환적(acyclic)임을 보장한다.
2. 생성될 계획의 최대 길이를 제한한다.
3. 복합 임무들 간에는 완전 순서(total order)를 강제한다.

이러한 제약 조건들은 HTN 계획 문제를 결정 가능하게 만들며, 그 계산 복잡도는 문제의 형태에 따라 NP-완전(NP-complete)에서 2-EXPSPACE-완전(2-EXPSPACE-complete)에 이르기까지 다양하다.1 많은 실용적인 HTN 문제들은 고전적 계획 언어인 PDDL(Planning Domain Definition Language)로 효율적으로 컴파일될 수도 있다.1

HTN의 실용적 가치는 이론적인 튜링 완전성(Turing-completeness)에 있는 것이 아니라, 원하는 계획의 구조를 기술하는 '도메인 특화 언어(domain-specific language)'로서 기능하여 플래너의 탐색을 효과적으로 안내하고 제약하는 능력에 있다. HTN이 STRIPS보다 '표현력이 높다'는 이론적 결과는 종종 인용되지만 18, 다른 연구에서는 일반적인 확장 기능들을 포함하면 두 형식론의 실제 구현체 모두 튜링 완전성을 가질 수 있어 순수한 계산 능력은 동등하다고 지적하기도 한다.18

이러한 관점의 차이는 '무엇을 계산할 수 있는가'에서 '어떻게 명시하는가'로 초점을 옮김으로써 해소된다. HTN의 계층 구조와 메서드는 단순히 계산 능력을 추가하는 것이 아니라, *선언적 제어 지식(declarative control knowledge)*을 추가하는 것이다.8 도메인 설계자는 메서드를 통해 플래너에게 "X라는 임무를 해결하려면, 이 절차들 중 하나를 사용해야 한다"고 지시한다. 이는 평평한 행동 목록에서 절차를 스스로 발견해야 하는 STRIPS와 근본적으로 다른 점이다.

드론 편대 임무를 설계하는 엔지니어에게는 이론적으로 가장 표현력이 풍부한 언어보다, 복잡한 운용 제약, 교전 규칙(rules of engagement), 표준 운용 절차(standard operating procedures)를 자연스럽게 명시할 수 있는 언어가 더 필요하다. HTN은 이러한 요구를 계층 구조를 통해 충족시킨다. 계층 구조는 임무 수준, 팀 수준, 개별 드론 수준 등 인간이 복잡한 문제를 구조화하는 방식과 유사하게 여러 추상화 수준에서 추론을 가능하게 한다.4 HTN을 결정 가능하게 만드는 '제약'들은 단지 한계가 아니라, 계획 수립 과정이 종료됨을 보장하고 생성된 계획이 사전 정의된 구조를 따르도록 만드는 유용한 '기능'으로 작용한다.

## 3.  계획 및 제어 아키텍처 비교 분석

HTN을 자율 시스템을 위한 AI 기술의 더 넓은 맥락에 위치시키기 위해, 가장 관련성 높은 대안인 고전적 계획(STRIPS) 및 반응형 제어(행동 트리)와 비교하는 것은 필수적이다. 이러한 비교는 주어진 문제에 적합한 아키텍처를 선택해야 하는 엔지니어에게 중요한 지침을 제공한다.

**표 1: 계획 및 제어 아키텍처 비교: HTN vs. STRIPS vs. 행동 트리**

| 차원            | 계층적 임무 네트워크 (HTN)           | STRIPS (고전적 계획)                      | 행동 트리 (Behavior Tree)              |
| --------------- | ------------------------------------ | ----------------------------------------- | -------------------------------------- |
| **목표 명시**   | 수행해야 할 임무 (Task to perform)   | 달성해야 할 세계 상태 (Goal state)        | 내재된 목표 없음 (행동이 목표를 암시)  |
| **계획 생성**   | 임무 분해 (Task decomposition)       | 상태 공간 탐색 (State-space search)       | 계획 생성 없음 (트리 자체가 계획)      |
| **지식 형태**   | 절차적 (Procedural): '방법(Methods)' | 선언적 (Declarative): '연산자(Operators)' | 명령형 (Imperative): '스크립트된 논리' |
| **반응성**      | 심의적 (Deliberative): 실행 전 계획  | 심의적 (Deliberative): 실행 전 계획       | 반응형 (Reactive): 실시간 결정         |
| **탐색 복잡도** | 계층 구조로 탐색 공간 축소           | 잠재적으로 지수적 증가                    | 탐색 없음 (단순 트리 순회)             |
| **주요 응용**   | 로보틱스, 물류, 군사 계획, 게임 AI   | 퍼즐 해결, 경로 탐색, 고전적 문제         | 게임 AI, 반응형 로봇 제어              |

### 3.1  HTN 대 고전적 계획(STRIPS): 절차적 안내 대 원리 기반 발견

HTN과 STRIPS의 가장 근본적인 차이는 HTN이 무언가를 '어떻게' 할지를 계획하는 반면, STRIPS는 '무엇을' 달성할지를 계획한다는 점에 있다.2 HTN 문제의 해는 초기 임무 네트워크를 구체화하는 행동 순서이며, STRIPS 문제의 해는 초기 상태에서 목표 상태에 도달하는 행동 순서이다.6

이 차이는 필요한 입력 지식의 형태에서 비롯된다. HTN은 메서드라는 형태의 추가적인 도메인 특화 제어 지식을 요구한다.8 문제를 해결하는 방법에 대한 이 '조언'은 계획 수립을 훨씬 효율적으로 만들지만, 상당한 양의 사전 지식 공학(knowledge engineering)을 필요로 한다.22 반면 STRIPS는 덜 구체적인 지식을 요구하지만, 훨씬 더 큰 탐색 공간에 직면하게 된다.13

### 3.2  HTN 대 행동 트리(BT): 심의적 계획 대 반응형 제어

HTN과 행동 트리의 차이는 계획과 제어의 차이로 요약될 수 있다. HTN은 실행에 앞서 행동의 미래 효과를 추론하여 행동 순서, 즉 '계획'을 생성하는 *계획 패러다임*이다.23 반면, 행동 트리는 미래에 대한 예측 없이 현재 상태에 기반하여 매 순간 결정을 내리는 *제어 아키텍처*이다.25

HTN 플래너는 가능한 분해 방법들의 집합으로부터 계획을 '발견'한다.25 행동 트리는 사전에 스크립트된 명령어 트리를 단순히 '실행'할 뿐이다. 행동 트리의 지능은 트리 자체의 설계에 있는 반면, HTN의 지능은 플래너의 탐색 과정에 있다.

각각의 한계는 하이브리드 시스템의 개발로 이어졌다. HTN 플래너가 상위 수준의 전략적 계획을 생성하면, 하위 수준의 반응형 행동 트리가 이를 실행하는 방식이다. 이는 HTN의 장기적인 추론 능력과 행동 트리의 동적인 환경에 대한 강건함 및 반응성을 결합하는 강력한 패턴이다.26

HTN, STRIPS, 행동 트리 사이의 선택은 어느 것이 '최고'인가의 문제가 아니라, 문제의 특성, 특히 *제어*, *최적성*, *반응성* 간의 균형을 어떻게 맞출 것인가의 문제이다.

만약 도메인 전문가가 임무 수행 방식에 대한 최대의 *제어*를 원하고, 생성된 계획이 합리적이고 구조적이기를 보장하고 싶다면 HTN이 탁월한 선택이다. 메서드는 플래너를 올바른 방향으로 안내하는 '가이드라인' 역할을 한다.2

만약 절차 자체는 중요하지 않고, 주어진 제약 하에서 *최적*의 해(예: 최단 경로)를 찾는 것이 목표라면, 좋은 휴리스틱을 갖춘 고전적 플래너가 더 나을 수 있다. HTN은 본질적으로 최적성을 보장하지 않는다. HTN은 주어진 절차 중 하나를 따르는 유효한 계획을 찾을 뿐이며, 이것이 전역 최적해가 아닐 수도 있다.10

만약 환경이 매우 동적이고 예측 불가능하여 장기적인 계획보다 즉각적인 *반응성*이 더 중요하다면, 행동 트리가 종종 더 적합하다. 행동 트리는 세계가 약간 변할 때마다 재계획(re-planning)을 수행하는 오버헤드가 없기 때문이다.26

복잡한 드론 임무는 이러한 아키텍처들의 *계층적* 조합을 필요로 할 가능성이 높다. 예를 들어, 며칠에 걸친 감시 임무의 경우, HTN 플래너는 상위 수준의 임무 계획(예: 어느 날 어떤 지역을 정찰할지, 배터리와 데이터 제약을 고려하여)을 생성하는 데 이상적이다. 그러나 새떼를 피하거나 돌풍에 맞춰 자세를 조정하는 등 순간적인 비행 제어는 행동 트리나 피드백 제어기 같은 반응형 시스템이 더 잘 처리한다. 가장 강건한 드론 아키텍처는 전략적 심의를 위해 HTN을, 전술적 실행을 위해 행동 트리를 사용하는 형태가 될 것이다.26

## 4.  다중 드론 임무 관리를 위한 HTN

HTN 프레임워크를 실제 드론 임무에 적용하는 구체적인 예시를 통해 추상적인 개념을 실용적인 구현으로 전환할 수 있다. 이 과정은 다양한 연구에서 제시된 개념들을 종합하여 구성된다.27

### 4.1  드론 임무의 계층적 임무 모델링

다수의 드론을 이용한 협력 정찰 임무를 HTN으로 모델링하는 과정은 다음과 같이 계층적으로 분해될 수 있다.

**표 2: '다중 드론 협력 정찰' 임무의 HTN 분해 예시**

| 수준               | 임무/메서드                                         | 전제조건 (Preconditions)                    | 하위 임무 네트워크 (Subtasks & Constraints)                  |
| ------------------ | --------------------------------------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| **0**              | **복합 임무:** `CoordinatedRecon(Area_A, {D1, D2})` | -                                           | 초기 임무 네트워크                                           |
| **1**              | **메서드:** `DivideAndConquerScan`                  | `(num_drones >= 2)`, `(area_is_large)`      | **하위 임무:** `{AssignSubArea(D1, A1), AssignSubArea(D2, A2), ExecuteRecon(D1, A1), ExecuteRecon(D2, A2), ConsolidateData({D1, D2})}` **제약:** `(AssignSubArea(D1, A1) ≺ ExecuteRecon(D1, A1))`, `(AssignSubArea(D2, A2) ≺ ExecuteRecon(D2, A2))`, `(ExecuteRecon(D1, A1) ≺ ConsolidateData)`, `(ExecuteRecon(D2, A2) ≺ ConsolidateData)` |
| **2**              | **복합 임무:** `ExecuteRecon(D1, A1)`               | -                                           | -                                                            |
|                    | **메서드:** `StandardGridScan`                      | `(battery(D1) > 50%)`, `(weather == clear)` | **하위 임무:** `{(CalculateGridPath(A1)), (FlyPath(D1, Path)), (ReturnToBase(D1))}` **제약:** `(CalculateGridPath ≺ FlyPath ≺ ReturnToBase)` |
| **3**              | **복합 임무:** `FlyPath(D1, Path)`                  | -                                           | -                                                            |
|                    | **메서드:** `RecursivePathTraversal`                | `(Path is not empty)`                       | **하위 임무:** `{(Primitive: FlyTo(D1, next_wp)), (Primitive: ActivateSensor(Camera)), (Compound: FlyPath(D1, remaining_path))}` **제약:** `(FlyTo ≺ ActivateSensor ≺ FlyPath)` |
| **최종 계획 (D1)** | -                                                   | -                                           | `FlyTo(WP1)`, `ActivateSensor()`, `FlyTo(WP2)`, `ActivateSensor()`,... `ReturnToBase()` |

이 표는 단일 상위 명령(`CoordinatedRecon`)이 어떻게 여러 에이전트(드론 1, 드론 2)를 위한 실행 가능한 계획으로 분해되는지를 명확히 보여준다. 각 수준에서 메서드는 특정 조건 하에 상위 임무를 하위 임무 네트워크로 분해하고, 이들 간의 순서 제약을 정의한다. `ExecuteRecon`과 같은 임무는 병렬로 실행될 수 있으며, 이는 HTN의 부분 순서 계획 능력을 보여준다.

### 4.2  HTN 확장을 통한 드론 운용 핵심 과제 해결

기본적인 HTN 형식만으로는 실제 드론 운용에서 발생하는 모든 문제를 다루기 어렵다. 배터리, 연료, 데이터 저장 공간과 같은 소모성 자원이나 시간 제약, 다중 에이전트 간의 충돌 회피 등은 HTN의 확장을 통해 해결될 수 있다.

- **자원 관리 (타임라인 및 제약 조건):**
  - **타임라인 기반 HTN (T-HTN):** 이 접근법은 HTN과 타임라인 기반 계획을 통합한다.28 드론의 배터리나 공유 공역과 같은 자원을 타임라인으로 모델링하고, 행동(action)을 이 타임라인 상의 특정 시간 간격에 배치한다. 이를 통해 자원 소모와 상태 변화를 시간의 흐름에 따라 추적하고, 다중 드론 시나리오에서 공유 자원(예: 충전 패드, 좁은 통로)에 대한 충돌을 효과적으로 방지할 수 있다.28
  - **수치적 상태 변수 (Numeric Fluents):** SHOP2나 HATP와 같은 현대적인 HTN 플래너들은 수치적 상태 변수를 지원한다.13 이를 통해 배터리 잔량, 연료 소모량, 데이터 저장량과 같은 연속적인 값을 행동의 전제조건과 효과에 직접 모델링할 수 있다(예: `precondition: (battery_level > 20)`, `effect: (decrease battery_level by 5)`). 이는 장거리 비행이나 에너지 제약이 심한 임무를 계획하는 데 필수적이다.31
- **다중 에이전트 협력:**
  - HTN의 계층 구조는 팀 수준의 임무가 개별 에이전트의 임무로 자연스럽게 분해되기 때문에 다중 에이전트 계획에 매우 적합하다.22
  - **HATP (Hierarchical Agent-based Task Planner):** 이 프레임워크는 에이전트를 '일급 객체(first-class entity)'로 취급하여 로보틱스에 더 적합하게 HTN을 확장한다.35 HATP는 에이전트 간의 허용 가능한 행동을 정의하는 '사회적 규칙(social rules)'을 명시할 수 있으며, 기호적 계획과 기하학적 추론을 연동하여 3D 세계에서 행동의 유효성을 검증한다. 이는 물리적 상호작용이 중요한 로봇 시스템에 매우 중요하다.27
  - 인간 운용자는 이러한 계층적 계획의 여러 추상화 수준에서 개입할 수 있다. 예를 들어, 팀 전체에 상위 수준의 임무를 할당하면, 플래너가 이를 각 드론을 위한 구체적인 계획으로 자동 분해한다.30
- **동적 재계획 및 실행 모니터링:**
  - 실제 임무 환경은 예측 불가능한 변화로 가득하다. 실행 중 원시 임무가 실패하면(예: 드론이 목표 지점에 도달하지 못함), 실행 모니터링 시스템이 현재 계획의 실패를 감지하고 재계획을 촉발할 수 있다.36
  - HTN의 계층적 구조 덕분에, 전체 상위 계획을 폐기하지 않고도 실패한 하위 임무에 대한 대안적인 메서드를 찾아 계획을 수정하는 것이 가능하다. 이는 훨씬 효율적인 복구 전략으로 이어진다.24

드론을 위한 HTN의 진정한 잠재력은 기본적인 형식론이 아니라, T-HTN이나 HATP와 같이 자원을 인식하고 에이전트 중심적인 확장 기능들을 통해 실현된다. 이러한 확장 기능들은 추상적인 계획을 드론 임무의 물리적 현실에 접목시킨다. 기본적인 HTN 플래너는 `비행-스캔-복귀`와 같은 논리적으로 올바른 순서를 생성할 수 있지만 1, 비행 도중에 배터리가 소진되거나 두 드론이 같은 공역에서 충돌한다면 그 계획은 무용지물이다.31 따라서 배터리를 위한 수치적 상태 변수 27, 공역 충돌 방지를 위한 타임라인 28, 에이전트 특화 추론 35과 같은 확장 기능들은 단순한 '부가 기능'이 아니라, 실행 가능하고 강건한 계획을 생성하기 위한 '필수 구성요소'이다. 따라서 드론에 HTN을 적용하려는 실무자는 SHOP2와 같은 일반적인 플래너에서 시작하기보다는, T-HTN, HATP와 같이 시간, 자원, 다중 에이전트 지원이 강력한 프레임워크를 우선적으로 고려해야 한다.

## 5.  첨단 방법론 및 미래 전망

HTN은 강력한 패러다임이지만, 지식 공학의 병목 현상, 불확실성 처리의 한계 등 본질적인 약점도 가지고 있다. 현대 AI 연구는 이러한 한계를 극복하기 위해 기계 학습, 확률적 추론 등 다양한 기술을 HTN과 융합하는 방향으로 나아가고 있다.

### 5.1  지식 공학 병목 현상 극복

HTN의 가장 큰 단점은 전문가가 수동으로 완벽한 메서드 라이브러리를 구축해야 한다는 점이다.14 이는 시간이 많이 소요될 뿐만 아니라, 예측하지 못한 상황에 시스템이 취약해지는 원인이 된다.

- **시연 기반 HTN 학습 (Learning HTN from Demonstration):** 메서드를 수동으로 작성하는 대신, 전문가의 임무 수행 시연을 관찰하여 학습하는 접근법이다. 알고리즘은 실행 궤적(execution trace)을 분석하여 계층적 임무 구조와 행동의 전제조건을 추론하고, 이를 통해 도메인 지식을 자동으로 구축한다.14

- **거대 언어 모델(LLM)과의 하이브리드 계획: ChatHTN 접근법:** 이 혁신적인 접근법은 전통적인 HTN 플래너와 ChatGPT와 같은 거대 언어 모델을 결합한다.22

  1. HTN 플래너는 평소와 같이 작동한다.

  2. 만약 지식 베이스에 없는 새로운 복합 임무를 분해해야 하는 상황에 직면하면, 플래너는 LLM에게 질의한다.

  3. 플래너는 LLM에게 임무, 현재 상태 등의 맥락 정보를 제공하고, 이 임무를 알려진 원시 임무들의 순서로 분해하는 그럴듯한 방법을 요청한다.

  4. LLM이 제안한 (잠재적으로 불완전하거나 부정확할 수 있는) 분해 방법은 다시 기호적 플래너에 의해 검증되고 계획에 통합된다.

     이 방식은 HTN의 지식 병목 현상(LLM이 지식의 공백을 메움)과 LLM의 부정확성(기호적 플래너가 검증자 역할을 함)을 동시에 우아하게 해결한다.22

### 5.2  불확실성 및 선호도 기반 계획

실제 드론 임무는 결정론적인 경우가 거의 없다. 행동의 결과가 불확실하거나, 여러 가능한 계획 중 더 '선호되는' 계획이 존재할 수 있다.

- **확률적 HTN (Probabilistic HTN):** 행동 결과의 불확실성을 다루기 위해 HTN 형식론을 확률적으로 확장한다. 이는 플래너가 행동의 성공 확률을 추론하고, 신뢰할 수 없는 센서나 액추에이터가 있는 상황에서도 임무 성공 가능성을 최대화하는 계획을 생성하도록 돕는다.38 예를 들어, 짧은 경로가 탐지될 위험이 높다면, 플래너는 약간 더 길지만 안전한 경로를 선택하는 계획을 생성할 수 있다.29
- **선호도 기반 계획 (Preference-Based Planning):** 표준 HTN은 '유효한' 계획을 찾지만, 반드시 '최선'의 계획을 찾는 것은 아니다. PDDL3 확장과 같은 선호도 언어를 도입하여 사용자는 계획에 대한 선호도를 명시할 수 있다.10 예를 들어, "정찰 임무에는 드론1을 사용하는 것을 선호한다" 또는 "총 비행 시간을 최소화하는 계획을 선호한다"와 같은 조건을 추가할 수 있다. 그러면 플래너는 분기 한정법(branch-and-bound)이나 휴리스틱 탐색을 사용하여 이러한 선호도에 따라 최적인 유효 계획을 찾는다.10

### 5.3  종합 및 드론 임무 플래너를 위한 제언

HTN은 드론 임무 관리를 위한 강력하고 유연한 프레임워크를 제공하지만, 그 효과는 임무의 특성에 맞는 적절한 형태의 HTN 플래너를 선택하고 구성하는 데 달려 있다.

- **고도로 구조화되고 예측 가능한 임무 (예: 물류 배송, 정기 시설 점검):** 수치 및 시간 제약 기능이 강화된 SHOP2와 같은 전통적인 제약적 HTN 플래너가 매우 효과적이다. 계획의 효율성과 검증 가능성이 큰 자산이다.
- **동적이고 준구조화된 임무 (예: 수색 및 구조, 적대적 환경에서의 정찰):** 하이브리드 접근법이 권장된다. 여기에는 다음이 포함될 수 있다.
  1. 전략적 계획과 반응형 실행을 결합하기 위한 **계층적 HTN + 행동 트리 아키텍처**.26
  2. 사전 정의된 메서드가 불충분할 때 유연성을 확보하기 위한 **임무 삽입 HTN (TIHTN)**.5
  3. 환경 및 행동 결과의 내재적 불확실성을 처리하기 위한 **확률적 HTN 플래너**.38
- **탐색적이거나 급변하는 도메인 (완벽한 지식 베이스 구축이 비현실적인 경우):** **ChatHTN**과 같은 신기술이 미래 방향을 제시한다. 이러한 기술은 강력하고 유연한 HTN 기반 시스템을 배포하는 데 필요한 엔지니어링 노력을 크게 줄여줄 것으로 기대된다.22

결론적으로, HTN 계획의 미래는 단일한 형태가 아니라 하이브리드 형태일 것이다. 이는 HTN의 한계를 극복하기 위해 기계 학습, 반응형 제어, 고전적 계획 등 다른 기술들을 통합하면서도 그 핵심 강점은 유지하는 방향이다. 고전적 계획의 주요 약점은 복잡한 도메인에서의 조합 폭발이고 13, 반응형 시스템의 약점은 미래 예측 능력의 부재이다.26 가장 진보된 연구들은 HTN을 단독으로 '수리'하려 하기보다, 다른 기술들과 결합하여 약점을 보완하고 있다. TIHTN은 고전적 계획의 유연성을, 확률적 HTN은 결정 이론의 강건함을, ChatHTN은 LLM을 활용하여 지식의 공백을 메우는 방식을 취한다.

이는 드론 자율성 스택의 설계가 '만능 플래너'를 찾는 것이 아니라, 이러한 다양한 구성요소들을 의식적으로 선택하고 통합하는 과정이어야 함을 시사한다. 이 시스템에서 HTN의 역할은 임무의 '구조적 사고' 또는 '절차적 뼈대'를 제공하는 것이며, 다른 구성요소들은 불확실성, 반응성, 지식 격차를 처리하기 위해 통합된다. 궁극적인 자율 드론 시스템은 HTN 플래너가 전략적 지휘자 역할을 하는 전문화된 에이전트들의 사회(society of specialized agents)와 같은 형태가 될 것이다.

#### **참고 자료**

1. Hierarchical task network - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Hierarchical_task_network
2. Introduction to HTN Planning by Dr. Pascal Bercher, accessed July 23, 2025, https://bercher.net/data/teaching/2018/HTN-Lecture/4-on-1/06--IntroductionToHTNPlanning--handout.pdf
3. Bridging the Gap Between Planning and Scheduling - NASA Technical Reports Server, accessed July 23, 2025, https://ntrs.nasa.gov/api/citations/20000115877/downloads/20000115877.pdf
4. Hierarchical Task Network (HTN) Planning in AI - GeeksforGeeks, accessed July 23, 2025, https://www.geeksforgeeks.org/artificial-intelligence/hierarchical-task-network-htn-planning-in-ai/
5. Hierarchical Task Network Planning with Task Insertion and State Constraints - IJCAI, accessed July 23, 2025, https://www.ijcai.org/proceedings/2017/0623.pdf
6. Hierarchical task network planning, accessed July 23, 2025, http://www.cs.hut.fi/~sto/planning-seminaari/hagstrom/hierarchical-planning.html
7. HTN | gurubehc - WordPress.com, accessed July 23, 2025, https://gurubehc.wordpress.com/2013/04/24/htn/
8. 계층적 작업 망 계획을 위한 변환 기반의 접근법 489 - TKIPS, accessed July 23, 2025, https://tkips.kips.or.kr/digital-library/manuscript/file/13072/B-2009-16-6-489.pdf
9. On Bringing HTN Domains Closer to Reality - The Case of Satellite and Rover Domains - ICAPS 2022, accessed July 23, 2025, https://icaps22.icaps-conference.org/workshops/SPARK/papers/spark2022_paper_9.pdf
10. HTN Planning with Preferences - IJCAI, accessed July 23, 2025, https://www.ijcai.org/Proceedings/09/Papers/298.pdf
11. On the Decidability of HTN Planning with Task Insertion - IJCAI, accessed July 23, 2025, https://www.ijcai.org/Proceedings/11/Papers/327.pdf
12. HTN Planning as Heuristic Progression Search - Journal of Artificial Intelligence Research, accessed July 23, 2025, https://jair.org/index.php/jair/article/download/11282/26578/23423
13. Hierarchical Task Network (HTN) Planning, accessed July 23, 2025, https://pages.mtu.edu/~nilufer/classes/cs5811/2012-fall/lecture-slides/cs5811-ch11b-htn.pdf
14. Decision Support for Beyond Visual Range Air Combat Based on HTN Planning and Machine Learning | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/382350938_Decision_Support_for_Beyond_Visual_Range_Air_Combat_Based_on_HTN_Planning_and_Machine_Learning
15. Autonomous Systems - DTIC, accessed July 23, 2025, https://apps.dtic.mil/sti/pdfs/AD1010077.pdf
16. Hierarchical Task Network Plan Reuse for Video Games - Maastricht University, accessed July 23, 2025, https://dke.maastrichtuniversity.nl/m.winands/documents/CIG2016_HTN.pdf
17. (PDF) Grounding of HTN Planning Domain - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/320492086_Grounding_of_HTN_Planning_Domain
18. Expressivity of STRIPS-like and HTN-like Planning - www3.fiit.stuba.sk, accessed July 23, 2025, [https://www2.fiit.stuba.sk/~lekavy/publikacie/2007%20Expressivity%20of%20STRIPS-Like%20and%20HTN-Like%20Planning/strips_vs_htn_lekavy.pdf](https://www2.fiit.stuba.sk/~lekavy/publikacie/2007 Expressivity of STRIPS-Like and HTN-Like Planning/strips_vs_htn_lekavy.pdf)
19. (PDF) Expressivity of STRIPS-Like and HTN-Like Planning - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/221254628_Expressivity_of_STRIPS-Like_and_HTN-Like_Planning
20. A Translation-based Approach to Hierarchical Task Network Planning - Korea Science, accessed July 23, 2025, https://koreascience.kr/article/JAKO200903538424563.page
21. 계층적 네트워크: 코어, 배포 및 액세스 레이어 | 파이버몰 - FiberMall, accessed July 23, 2025, https://www.fibermall.com/ko/blog/core-distribution-and-assess-layer.htm
22. ChatHTN: Interleaving Approximate (LLM) and Symbolic HTN Planning - arXiv, accessed July 23, 2025, https://arxiv.org/html/2505.11814v1
23. Exploring HTN Planners through Example - Game AI Pro, accessed July 23, 2025, https://www.gameaipro.com/GameAIPro/GameAIPro_Chapter12_Exploring_HTN_Planners_through_Example.pdf
24. Hierarchical Task Network Planning - UE Marketplace - Unreal Engine Forums, accessed July 23, 2025, https://forums.unrealengine.com/t/hierarchical-task-network-planning/148010
25. What's the difference between Behavior Trees and Hierarchical Task Networks - Reddit, accessed July 23, 2025, https://www.reddit.com/r/gamedev/comments/1ozugf/whats_the_difference_between_behavior_trees_and/
26. A Hybrid Approach to Planning and Execution in Dynamic Environments Through Hierarchical Task Networks and Behavior Trees, accessed July 23, 2025, https://cdn.aaai.org/ojs/13044/13044-52-16561-1-2-20201228.pdf
27. (PDF) HATP: An HTN Planner for Robotics - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/262525949_HATP_An_HTN_Planner_for_Robotics
28. T-HTN: Timeline Based HTN Planning for Multiple Robots, accessed July 23, 2025, https://icaps22.icaps-conference.org/workshops/HPlan/papers/paper-09.pdf
29. Optimal routing and transmission strategies for UAV reconnaissance missions with detection threats - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2406.19001
30. Mixed-Initiative Planning and Execution for Multiple Drones in Search and Rescue Missions, accessed July 23, 2025, https://www.researchgate.net/publication/282439405_Mixed-Initiative_Planning_and_Execution_for_Multiple_Drones_in_Search_and_Rescue_Missions
31. Drone Routing for Drone-Based Delivery Systems: A Review of Trajectory Planning, Charging, and Security - PMC - PubMed Central, accessed July 23, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9919662/
32. [1703.10049] Autonomous Recharging and Flight Mission Planning for Battery-operated Autonomous Drones - arXiv, accessed July 23, 2025, https://arxiv.org/abs/1703.10049
33. Autonomously Constructing Hierarchical Task Networks for Planning and Human-Robot Collaboration, accessed July 23, 2025, https://scazlab.yale.edu/sites/default/files/files/hayes_icra16.pdf
34. Hierarchical Task Assignment for Multi-UAV System in Large-Scale Group-to-Group Interception Scenarios - MDPI, accessed July 23, 2025, https://www.mdpi.com/2504-446X/7/9/560
35. [1405.5345] HATP: An HTN Planner for Robotics - arXiv, accessed July 23, 2025, https://arxiv.org/abs/1405.5345
36. Exploring HTN Planners through Example - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/335744815_Exploring_HTN_Planners_through_Example
37. arXiv:2505.11814v1 [cs.AI] 17 May 2025, accessed July 23, 2025, http://www.arxiv.org/pdf/2505.11814
38. Integration of Online Learning into HTN Planning for Robotic Tasks, accessed July 23, 2025, https://cdn.aaai.org/ocs/4265/4265-19460-1-PB.pdf
39. HTN Planning for Robotics: A Deep Dive - Number Analytics, accessed July 23, 2025, https://www.numberanalytics.com/blog/htn-planning-for-robotics-deep-dive


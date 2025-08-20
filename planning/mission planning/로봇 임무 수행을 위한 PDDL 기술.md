# 로봇 임무 수행을 위한 PDDL 기술

## 1.  기호주의 계획의 귀환과 PDDL의 역할

### 1.1 로봇 자율성의 핵심 과제로서의 작업 계획(Task Planning)

로봇이 단순히 정해진 동작을 반복하는 기계를 넘어, 주어진 환경에서 유의미한 목표를 자율적으로 달성하기 위해서는 개별 행동(action)을 넘어선 장기적인 행동 순서(sequence of actions)를 생성하는 능력이 필수적이다. 이는 로봇 팔이 컵을 집어 테이블 위로 옮기는 단순한 작업부터, 여러 방을 이동하며 물건을 수거하고 정리하는 복잡한 임무에 이르기까지 모든 지능적 행동의 근간을 이룬다. 이처럼 목표를 달성하기 위해 필요한 행동의 순서를 결정하는 과정을 '작업 계획(Task Planning)'이라 하며, 이는 단순한 반응형 제어(reactive control)를 넘어선 '숙고(deliberation)'의 영역에 속한다.1 인공지능(AI)의 한 분야인 '자동화 계획(Automated Planning)'은 바로 이 문제의 핵심을 다루며, 본 보고서는 이 분야의 사실상 표준 언어인 PDDL(Planning Domain Definition Language)을 중심으로 로봇의 자율적 임무 수행 능력을 기술적으로 심층 고찰하고자 한다.

### 1.2 PDDL의 탄생과 철학

PDDL은 1998년 드류 맥더멋(Drew McDermott)과 그의 동료들에 의해 국제 계획 대회(International Planning Competition, IPC)를 가능하게 할 목적으로 개발되었다.3 이전까지 다양한 연구 그룹들은 각자의 고유한 언어로 계획 문제를 기술하였기에, 플래너(planner)들의 성능을 객관적으로 비교하고 연구 결과를 공유하는 데 큰 어려움이 있었다. PDDL은 이러한 파편화된 연구 환경에 표준을 제시했다.

그 핵심 철학은 특정 문제 영역의 일반적인 '물리 법칙'과 '행동 규칙'을 기술하는 **도메인(domain)**과, 해결해야 할 구체적인 '상황 설정'을 기술하는 **문제(problem)**를 명확히 분리하는 것이다.3 예를 들어, '물류 운송'이라는 도메인에서는 "트럭은 도시 사이를 이동할 수 있다" 또는 "트럭에 화물을 실을 수 있다"와 같은 일반적인 규칙을 정의한다. 그리고 '파리의 화물을 베를린으로 옮기기'라는 특정 문제에서는 구체적인 객체(트럭1, 화물A, 도시 파리, 도시 베를린)와 초기 상태, 그리고 목표 상태를 기술한다. 이 표준화는 연구의 재사용성과 비교 가능성을 획기적으로 높이는 데 기여했으나, 동시에 특정 도메인에 고도로 특화된 시스템에 비해 표현력의 일부를 희생하는 대가를 치렀다.3

### 1.3 보고서의 핵심 논지와 구조

본 보고서는 PDDL이 단순히 20세기 후반의 학술적 유산을 넘어, 로봇 운영체제(Robot Operating System, ROS)와의 긴밀한 통합 6과 최신 머신러닝(Machine Learning, ML) 기술과의 창의적인 융합 8을 통해 어떻게 현대 로봇 공학에서 새로운 생명력을 얻고 있는지를 체계적으로 추적한다. 이를 위해 본 보고서는 다음과 같은 구조로 전개된다. 첫째, PDDL의 기본 구조와 역사적 진화를 해부하여 그 근본 원리를 파악한다. 둘째, 시간, 자원, 불확실성과 같은 현실 세계의 복잡한 제약을 모델링하기 위한 핵심적인 언어 확장을 분석한다. 셋째, ROS2 PlanSys2 프레임워크를 사례로 들어 PDDL이 실제 로봇 시스템에 어떻게 통합되고 작동하는지 구체적으로 살핀다. 넷째, 계층적 작업 네트워크(HTN), 행동 트리(Behavior Trees) 등 대안적인 접근법과 비교하며 PDDL의 상대적 위치와 장단점을 명확히 한다. 다섯째, PDDL의 내재적 한계와 이를 극복하려는 최신 신경-기호주의(Neuro-Symbolic) 접근법의 최전선을 탐사한다. 이 과정을 통해 본 보고서는 로봇 지능의 청사진으로서 PDDL의 현재와 미래를 종합적으로 제시할 것이다.

이러한 분석의 기저에는 PDDL의 역설적인 부활에 대한 통찰이 자리 잡고 있다. PDDL의 현대적 부활은 그 언어 자체가 완벽하기 때문이 아니라, 오히려 그 한계가 명확하고 잘 정의되어 있기 때문에 가능했다. PDDL의 핵심적인 약점으로 지적되는 '기호 접지(symbol grounding)' 문제, 즉 추상적인 기호를 실제 세계의 감각 데이터와 연결하는 문제와, 복잡한 현실을 PDDL 문제 파일로 기술하는 것의 어려움은 역설적으로 현대 머신러닝, 특히 비전-언어 모델(Vision-Language Models, VLM)이 가장 잘 해결할 수 있는 문제 영역이다. 순수 기호 시스템인 PDDL은 `(on blockA table)`과 같은 기호가 실제 세계의 픽셀 데이터나 물리적 상태와 어떻게 연결되는지에 대한 내장 메커니즘을 가지고 있지 않다.10 또한, 복잡한 실제 환경을 PDDL 문제 파일로 정확하게 수동으로 작성하는 것은 엄청난 노력이 필요한 전문가의 영역이다.9 반면, VLM과 같은 최신 AI 모델들은 이미지와 자연어로부터 상황을 이해하고 구조화된 텍스트를 생성하는 데 탁월한 능력을 보인다.9 따라서 VLM이 로봇의 '지각(perception)'과 '이해(understanding)'를 담당하고, 그 결과를 PDDL이라는 '정형화된 기호 언어'로 변환하여, PDDL 플래너가 '논리적 추론(logical reasoning)'을 수행하는 명확한 분업 구조가 자연스럽게 형성된다. 결국, PDDL의 기호주의적 엄격함과 명확한 한계는 머신러닝과의 '역할 분담'을 위한 완벽한 인터페이스를 제공하며, 이는 PDDL이 과거의 유물로 남지 않고 신경-기호주의 아키텍처의 핵심 요소로 부활하는 핵심적인 원동력이 되고 있다.

## 2.  PDDL의 해부: 언어의 구조와 의미론

### 2.1 핵심 철학: 도메인과 문제의 분리 (Separation of Domain and Problem)

PDDL의 가장 근본적이고 중요한 설계 원칙은 재사용 가능한 '규칙의 집합'인 **도메인(domain)**과, 일회성의 '상황 설정'인 **문제(problem)**를 엄격하게 분리하는 것이다.3 도메인은 해당 세계의 물리 법칙, 즉 어떤 행동이 가능하고 그 행동이 어떤 결과를 낳는지를 정의한다. 문제는 그 세계 안에서 해결해야 할 특정 과제를 기술하며, 구체적인 객체들, 그들의 초기 상태, 그리고 달성해야 할 목표 상태로 구성된다.

이러한 분리는 객체 지향 프로그래밍(OOP)에서 일반적인 속성과 메소드를 정의하는 클래스(Class)와, 그 클래스로부터 생성된 구체적인 실체인 인스턴스(Instance)를 분리하는 것과 철학적으로 유사하다.3 이 덕분에 연구자나 개발자는 하나의 잘 정의된 도메인 파일(예: 로봇 팔 조작 도메인)을 가지고 수많은 다른 문제 파일(예: 블록 쌓기 문제, 컵 옮기기 문제, 문 열기 문제)을 적용하여 다양한 시나리오를 효율적으로 생성하고 테스트할 수 있다.3 이는 코드의 재사용성을 극대화하고 모델링의 효율성을 높이는 핵심적인 특징이다.

### 2.2 도메인 파일 (`domain.pddl`) 상세 분석

도메인 파일은 해당 세계의 '물리 법칙'과 '가능한 행동'을 정의하는, 계획 문제의 핵심적인 뼈대이다.5 그 구조는 리스프(Lisp)와 유사한 괄호 표현식을 사용하며, 주요 구성 요소는 다음과 같다.

- **`(:requirements)`**: 이 섹션은 해당 도메인 파일이 사용하는 PDDL 언어의 확장 기능을 명시한다. 이는 프로그래밍 언어에서 특정 라이브러리나 모듈을 가져오는 `import` 또는 `include` 구문과 유사한 역할을 한다.17 플래너는 이 정보를 보고 어떤 기능들을 지원해야 하는지 파악한다. 일반적인 요구사항으로는 다음과 같은 것들이 있다 15:

  - `:strips`: 가장 기본적인 요구사항으로, 부정적 전제조건(negative preconditions)과 조건부 효과(conditional effects)가 없는 간단한 행동 모델을 의미한다.
  - `:typing`: 객체에 타입을 부여할 수 있게 한다.
  - `:equality`: 객체들이 서로 같은지 다른지 비교(`=`)할 수 있게 한다.
  - `:adl`: 부정, 논리합(disjunction), 보편/존재 한정사(quantifiers) 등 더 풍부한 논리 표현을 허용하는 ADL(Action Description Language) 기능을 사용한다.
  - `:durative-actions`: 행동이 일정 시간 동안 지속되는 것을 모델링한다.
  - `:fluents`: 수치적 속성(numeric fluents)을 사용한다.

- **`(:types)`**: 도메인에 존재하는 객체들의 유형과 그 계층 구조를 정의한다.3 예를 들어, 

  `truck airplane - vehicle` 와 같이 트럭과 비행기는 모두 운송수단의 한 종류임을 명시할 수 있다. 이렇게 타입을 정의하면 행동의 매개변수나 술어의 인자에 대한 제약을 더 명확하게 할 수 있어, 모델의 가독성과 정확성을 높이고 플래너의 탐색 공간을 줄이는 데 도움이 된다.

- **`(:constants)`**: 모든 문제 인스턴스에 걸쳐 항상 존재하고 변하지 않는 고정된 객체를 선언한다.17 예를 들어, 모든 물류 문제에 '본사(mainsite)'나 '중앙 창고(central-depot)'와 같은 고정된 장소가 존재한다고 가정할 때 유용하다.

- **`(:predicates)`**: 세상의 상태를 나타내는 참/거짓 명제(논리적 사실)의 템플릿을 정의한다.5 각 술어는 이름과 물음표(

  `?`)로 시작하는 매개변수 목록으로 구성되며, 매개변수에는 타입이 지정될 수 있다. 예를 들어, `(at?obj - object?loc - location)`는 '객체 `?obj`가 장소 `?loc`에 있다'는 사실을 나타내는 템플릿이다. 술어는 행동에 의해 그 진리값이 변하는 '동적(dynamic) 술어'와, 변하지 않는 '정적(static) 술어'로 구분될 수 있다.15 예를 들어, 

  `(is-heavy?obj)`와 같은 객체의 고유 속성은 정적 술어일 수 있다.

- **`(:action)`**: 상태를 한 상태에서 다른 상태로 변화시키는 연산자를 정의하며, PDDL 도메인 모델링의 핵심이다.14 각 행동은 다음과 같은 하위 요소들로 구성된다.

  - **`:parameters`**: 해당 행동에 사용될 객체 변수들을 정의한다. `?r - robot`와 같이 변수명과 타입을 명시하여, 이 행동이 어떤 종류의 객체들에 적용될 수 있는지를 한정한다.5

  - **`:precondition`**: 이 행동이 실행되기 위해 반드시 참이어야 하는 조건들의 논리적 결합(conjunction, `and`) 또는 선언(disjunction, `or`)이다.5 예를 들어, 로봇이 물건을 집기 위해서는 

    `(and (robot-free?r) (at?obj?loc) (at?r?loc))`와 같이 로봇의 손이 비어 있어야 하고, 로봇과 물건이 같은 장소에 있어야 한다는 조건이 만족되어야 한다.

  - **`:effect`**: 행동이 성공적으로 실행된 후 세상의 상태가 어떻게 변하는지를 기술한다.5 효과는 주로 원자 명제(atomic proposition)를 추가(add effect)하거나 삭제(delete effect)하는 것으로 구성된다. 삭제 효과는 

    `(not...)` 구문을 사용하여 표현한다. 예를 들어, 로봇이 물건을 집는 행동의 효과는 `(and (holding?r?obj) (not (robot-free?r)) (not (at?obj?loc)))` 와 같이, 로봇이 물건을 들고 있는 상태가 되고, 손이 비어 있다는 사실과 물건이 원래 위치에 있다는 사실은 거짓이 되는 것으로 표현할 수 있다.

### 2.3 문제 파일 (`problem.pddl`) 상세 분석

문제 파일은 도메인 파일에 정의된 일반적인 규칙들을 바탕으로, 해결해야 할 특정 시나리오를 구체화하는 역할을 한다.5

- **`(:domain <domain-name>)`**: 이 문제가 어떤 도메인 규칙을 따르는지를 명시한다. 여기에 명시된 이름은 반드시 대응하는 도메인 파일의 `(domain...)`에 정의된 이름과 일치해야 한다.5

- **`(:objects)`**: 해당 문제에 등장하는 모든 구체적인 객체(instance)들을 나열한다.5 예를 들어, 

  `robot1 - robot`, `roomA roomB - room`, `apple - food`와 같이 각 객체의 이름과 도메인에 정의된 타입을 함께 선언한다.

- **`(:init)`**: 계획이 시작되는 시점의 초기 상태를 정의한다.15 이 섹션에는 초기 상태에서 참인 모든 구체화된(grounded) 술어들을 나열한다. 예를 들어, 

  `(at robot1 roomA)`, `(robot-free robot1)`, `(at apple roomA)`와 같이 기술한다. 여기에 명시되지 않은 모든 술어는 **'닫힌 세계 가정(Closed-World Assumption, CWA)'**에 따라 거짓으로 간주된다. 이는 모델링을 간결하게 해주는 중요한 원칙이다.

- **`(:goal)`**: 계획이 성공적으로 종료되었을 때 만족해야 할 목표 상태를 논리식으로 기술한다.15 목표는 일반적으로 

  `(and...)` 구문을 사용하여 여러 조건의 논리곱으로 표현된다. 예를 들어, `(and (at robot1 roomB) (holding robot1 apple))`은 로봇이 B 방에 있으면서 사과를 들고 있는 상태를 목표로 설정한다.

### 2.4 Table 1: PDDL 주요 버전별 핵심 추가 기능 및 특징 비교 (PDDL 1.2 ~ PDDL 3.1)

PDDL은 정적인 언어가 아니라, IPC를 거치며 현실 세계의 복잡성을 더 잘 표현하기 위해 지속적으로 진화해왔다. 이 진화의 역사를 한눈에 파악하는 것은 PDDL의 현재 기능과 그 배경을 이해하는 데 필수적이다. 각 버전이 어떤 '현실의 문제'를 해결하기 위해 도입되었는지를 명확히 보여주는 로드맵은 다음과 같다.3

| 버전 (IPC)               | 핵심 추가 기능                                               | 해결하고자 하는 문제                                         | 로보틱스에서의 의미                                          |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **PDDL 1.2** (1998/2000) | STRIPS 기반, 타입, 조건부 효과(when-effects)                 | 결정론적, 단일 에이전트, 이산적(discrete) 세계의 기본 계획 문제 표준화 | 로봇 행동의 기본 논리(선행조건/효과) 모델링의 기초를 제공함. |
| **PDDL 2.1** (2002)      | **수치적 유동량(Numeric Fluents)**, **지속 행동(Durative Actions)**, 계획 메트릭(Plan Metrics) | 시간, 연료, 에너지 등 연속적 자원 관리. 동시 실행 및 시간 소요 작업 모델링. 계획의 질적 평가. | 로봇의 에너지 소비, 작업 소요 시간, 동시 작업 등 현실적 제약조건 모델링의 시작. 로보틱스 적용의 중요한 분기점. **(본 보고서의 III장에서 상세히 다룸)** |
| **PDDL 2.2** (2004)      | 파생 술어(Derived Predicates), 시간 초기 리터럴(Timed Initial Literals) | 사실들 간의 종속성(e.g., 전이성) 표현. 계획과 무관하게 발생하는 외부 사건(Exogenous Events) 모델링. | 로봇의 센서 데이터로부터 파생되는 복합 상태 추론(e.g., "A와 B가 연결되고 B와 C가 연결되면, A와 C는 연결되어 있다"). 예측된 환경 변화(e.g., "문이 10초 후에 자동으로 닫힌다")에 대응하는 계획 수립. |
| **PDDL 3.0** (2006)      | 상태-궤적 제약(State-Trajectory Constraints), 선호도(Preferences) | 계획 실행 '과정'에 대한 제약(e.g., "위험 구역에 절대 들어가지 말 것"). 최적은 아니더라도 '더 나은' 계획 선호. | 안전 제약(Safety Constraints)을 강화하여 로봇의 안전한 작동 보장. "가급적 빨리", "가급적 에너지를 적게"와 같은 연성 제약(soft constraints)을 고려한 질적으로 우수한 계획 생성. |
| **PDDL 3.1** (2008/2011) | 객체 유동량(Object-Fluents)                                  | 함수의 결과값이 숫자가 아닌 다른 객체가 될 수 있도록 확장 (e.g., `(location-of?robot)`의 결과가 `roomA` 객체). | 로봇이 현재 들고 있는 객체, 현재 연결된 도구 등, 동적으로 변하는 객체 관계를 더 직관적이고 유연하게 표현 가능. |

이러한 진화 과정을 통해 PDDL은 초기의 단순한 논리적 모델에서 벗어나, 시간, 자원, 외부 사건, 선호도 등 현실 세계의 다양한 측면을 포괄하는 풍부한 표현력을 갖춘 언어로 발전해왔다. 특히 PDDL 2.1의 등장은 로보틱스 분야에서 PDDL의 실용성을 크게 높이는 결정적인 계기가 되었다.

## 3.  현실 세계로의 확장: 시간, 자원, 불확실성 모델링

고전적인 STRIPS 스타일의 PDDL은 모든 행동이 즉각적으로, 비용 없이 발생한다고 가정한다. 이러한 가정은 이론적인 문제에서는 유용할 수 있지만, 물리적 세계와 상호작용하는 로봇에게는 심각한 한계로 작용한다.21 로봇의 모든 움직임은 시간을 소요하고 에너지를 소비하며, 때로는 행동의 결과가 예측과 다를 수 있다. PDDL은 이러한 현실 세계의 복잡성을 모델링하기 위해 여러 중요한 확장들을 도입했다.

### 3.1 PDDL 2.1: 시간과 숫자의 도입 - 로보틱스를 위한 혁명

DDL 2.1은 PDDL을 단순한 학술적 도구에서 실용적인 로봇 계획 언어로 전환시킨 결정적인 계기였다.3 이 버전에서 도입된 `numeric fluents`와 `durative actions`는 시간과 자원이라는, 로보틱스에서 가장 중요한 두 가지 제약을 정면으로 다루었다.

#### 3.1.1 수치적 유동량 (Numeric Fluents)

- **개념 및 문법**: Numeric fluents, 또는 간단히 'fluents'는 도메인 내에서 시간에 따라 변할 수 있는 연속적인 수치 값을 나타낸다. 이는 도메인 파일의 `(:functions)` 블록에 선언되며, 연료량, 배터리 잔량, 이동 거리, 누적 비용 등 다양한 양적 속성을 모델링하는 데 사용된다.16 예를 들어, `(fuel-level?v - vehicle)`은 차량 `?v`의 연료 수준을 나타내는 함수이다. 행동의 `:effect` 부분에서는 `(increase (total-cost) 10)`이나 `(decrease (fuel-level?v) 5)`와 같은 연산자를 사용하여 이 값들을 동적으로 변경할 수 있다.16
- **로봇 응용**: 수치적 유동량은 로봇의 물리적 제약을 모델링하는 데 필수적이다. 로봇의 배터리 잔량을 fluent로 모델링하고, 이동 행동의 전제조건으로 `(>= (battery-level) 10)`을 설정하여 최소한의 에너지가 있어야만 움직일 수 있도록 강제할 수 있다. 더 나아가, 문제 파일의 `(:metric)` 섹션을 통해 계획의 최적화 기준을 명시할 수 있다.16 예를 들어, `(:metric minimize (total-energy-consumed))`는 단순히 목표를 달성하는 것을 넘어, 총 에너지 소비를 최소화하는 '가장 효율적인' 계획을 찾도록 플래너에게 지시한다. 이는 로봇의 운영 효율성을 극대화하는 데 결정적인 역할을 한다.

#### 3.1.2 지속 행동 (Durative Actions)

- **개념 및 문법**: Durative actions는 즉각적인 상태 변화가 아닌, 특정 기간 동안 지속되는 행동을 모델링한다. 이는 `(:durative-action)` 키워드로 정의되며, 행동이 지속되는 시간을 명시하는 `(:duration)` 필드를 포함한다.16 이 기간은 고정된 값일 수도 있고, fluent에 기반한 가변적인 값일 수도 있다. 가장 큰 특징은 조건과 효과를 시간적 구간에 따라 다르게 적용할 수 있다는 점이다 16:
  - `(at start...)`: 행동이 시작되는 시점에 만족해야 하는 조건 또는 발생하는 효과.
  - `(over all...)`: 행동이 지속되는 전체 기간 동안(시작과 끝 제외) 계속해서 만족해야 하는 불변 조건(invariant).
  - `(at end...)`: 행동이 끝나는 시점에 만족해야 하는 조건 또는 발생하는 효과.
- **로봇 응용**: 이 기능은 로봇의 실제 행동을 매우 현실적으로 모델링할 수 있게 해준다. 예를 들어, '로봇 팔로 문 열기'라는 행동은 5초가 걸린다고 `(:duration (=?duration 5))`로 정의할 수 있다. 행동이 시작될 때 `(at start (gripper-on-handle))`라는 조건이 필요하고, 행동이 끝날 때 `(at end (door-is-open))`이라는 효과가 발생하도록 모델링할 수 있다. 특히 `(over all...)`로 정의된 불변 조건은 안전성을 높이는 데 매우 중요하다. 예를 들어, 로봇이 물건을 옮기는 동안에는 `(over all (not (gripper-is-slipping)))`과 같은 조건을 통해 물건을 놓치지 않아야 함을 강제할 수 있다. 이러한 시간적 제약 조건의 도입은 여러 행동이 시간적으로 겹쳐서 실행되는 동시성(concurrency) 계획의 유효성을 검증하고, 더 효율적인 병렬 실행 계획을 생성하는 데 핵심적인 역할을 한다.16

### 3.2 불확실성 대응을 위한 변종들 (Variants for Uncertainty)

실제 로봇 환경은 결정론적이지 않으며 예측 불가능한 요소로 가득하다. 로봇의 행동이 의도와 다르게 실패할 수도 있고(stochasticity), 주변 환경에 대한 정보가 불완전할 수도 있다(partial observability). 이러한 불확실성을 다루기 위해 PDDL의 여러 변종들이 개발되었다.

- **PPDDL (Probabilistic PDDL)**: 행동의 효과가 확률적으로 결정되는 상황을 모델링한다. 예를 들어, 로봇이 미끄러운 바닥에서 물건을 집으려 할 때 80% 확률로 성공하고 20% 확률로 실패(물건을 떨어뜨림)할 수 있다. 이는 `(probabilistic 0.8 (effect_success) 0.2 (effect_failure))`와 같이 각 효과에 확률을 부여하여 표현된다.3 PPDDL은 이러한 불확실한 행동 결과를 고려하여 목표 달성 성공 확률을 극대화하는 정책(policy)을 찾는 문제로 귀결되며, 이는 주로 마르코프 결정 과정(Markov Decision Process, MDP) 프레임워크를 통해 해결된다.26 ROSPlan과 같은 로봇 프레임워크는 PPDDL을 지원하여 로봇이 불확실한 환경에서 더 강건한 계획을 수립할 수 있도록 돕는다.27
- **RDDL (Relational Dynamic Influence Diagram Language)**: PPDDL에서 한 걸음 더 나아가, 부분 관찰 가능성(partial observability)과 동시성(concurrency)을 더 정교하게 다루는 언어이다. RDDL에서는 모든 상태 속성을 유동량(fluent)으로 표현하며, 이를 통해 MDP뿐만 아니라, 로봇이 자신의 정확한 상태를 알지 못하는 상황을 모델링하는 부분 관찰 가능 MDP(Partially Observable MDP, POMDP)를 기술하는 데 사용된다.3
- **PDDL+**: 물리 법칙에 따른 연속적인 상태 변화와 외부에서 발생하는 사건(exogenous events)을 모델링하는 데 특화되어 있다.3 PDDL+는 두 가지 중요한 개념을 도입한다:
  - **`(:process)`**: 시간이 흐름에 따라 지속적으로 상태 변수를 변화시키는 과정을 정의한다. 예를 들어, 로봇이 아무것도 하지 않아도 배터리가 서서히 방전되는 현상이나, 뜨거운 물체가 서서히 식는 과정을 모델링할 수 있다.25
  - **`(:event)`**: 특정 조건이 만족되면 계획과 무관하게 즉시 발생하는 외부 사건을 정의한다. 예를 들어, "온도가 100도가 되면 물이 끓기 시작한다" 또는 "문 앞에 사람이 감지되면 문이 자동으로 열린다"와 같은 현상을 모델링할 수 있다.25 PDDL+는 로봇이 자신의 행동뿐만 아니라, 통제할 수 없는 환경의 동적인 변화까지 고려하여 계획을 세워야 하는 매우 복잡하고 현실적인 시나리오에 필수적이다.21

이러한 확장과 변종들의 개발은 PDDL을 현실 세계의 복잡성에 한 걸음 더 다가서게 했다. 그러나 이러한 표현력의 증가는 필연적으로 계산적 대가를 수반한다. 현실 세계를 더 정확하게 '표현'하려는 노력은, 동시에 계획 문제를 '해결'하는 것을 기하급수적으로 어렵게 만드는 과정이기도 하다. 기본 STRIPS PDDL조차 계산적으로 다루기 어려운 문제인데(PSPACE-hard) 8, 여기에 시간, 연속적인 자원, 확률, 외부 사건과 같은 요소들이 추가되면 상태 공간은 상상 이상으로 거대해진다. PDDL+와 같이 가장 표현력이 풍부한 언어는 심지어 계획 문제의 결정 가능성(decidability) 자체를 잃어버리기도 한다. 즉, 어떤 문제에 대해서는 해가 존재하는지조차 이론적으로 판별할 수 없게 된다.21

이러한 표현력과 해결 가능성 사이의 근본적인 상충 관계(Expressiveness vs. Solvability Trade-off)는 AI 계획 분야의 오랜 딜레마이다. 이 때문에 많은 실제 로봇 시스템들은 PDDL+의 모든 기능을 활용하기보다는, PDDL 2.1 수준의 검증된 하위 집합을 중심으로 시스템을 구축하거나, 계획 단계에서는 단순화된 모델을 사용하고 실행 단계에서 행동 트리(BT)와 같은 다른 기술을 결합하는 하이브리드 접근법을 채택한다. 이는 순수한 이론적 최적성보다는 실용적인 강건성(robustness)과 예측 가능성을 우선시하는 공학적 타협의 결과이며, 왜 PDDL의 모든 기능이 실제 현장에서 동일한 빈도로 사용되지 않는지를 설명해준다.

## 4.  PDDL의 실제: ROS2 생태계와의 통합 (PlanSys2 사례 연구)

PDDL이 이론적 언어에 머무르지 않고 실제 로봇을 구동하는 핵심 두뇌로 작동하기 위해서는 로봇 운영체제(ROS)와의 긴밀한 통합이 필수적이다. ROS2 Planning System, 약칭 PlanSys2는 ROS2 생태계에서 PDDL 기반 계획을 구현한 가장 대표적이고 현대적인 사례이다.

### 4.1 ROSPlan에서 PlanSys2로의 진화

ROSPlan은 ROS 환경에 PDDL 기반 계획을 도입한 선구적인 프레임워크였다.30 ROSPlan은 지식 관리, 계획, 실행의 파이프라인을 구축하여 많은 로봇 연구에 기여했지만, 몇 가지 한계점을 가지고 있었다. 주로 단일 로봇 제어에 초점이 맞춰져 있었고, 복잡한 시나리오에서의 실행 효율성 문제가 제기되었다.31

PlanSys2는 이러한 한계를 극복하고 ROS2의 현대적인 아키텍처 위에서 새롭게 설계된 시스템이다.6 ROS2는 실시간 시스템, 보안, 다중 로봇 통신 등 산업 현장의 요구사항을 충족하도록 개발되었으며, PlanSys2는 이러한 ROS2의 장점을 최대한 활용하여 신뢰성, 성능, 유연성을 대폭 개선했다.31 특히 다중 로봇 계획 및 실행, 행동 트리(Behavior Trees)를 통한 강건한 실행, 플러그인 기반의 유연한 플래너 연동 등에서 큰 발전을 이루었다.

### 4.2 PlanSys2 아키텍처 심층 분석

PlanSys2의 아키텍처는 PDDL의 핵심 철학인 '도메인과 문제의 분리'를 ROS2의 노드(node) 기반 분산 시스템 아키텍처로 매우 명확하게 구현한 탁월한 예시이다.6 시스템은 크게 4개의 핵심 노드로 구성된다.

- **Domain Expert (도메인 전문가)**: 이 노드는 PDDL 도메인 파일(`domain.pddl`)을 로드하고 파싱하여, 해당 세계의 모든 '규칙'과 '가능한 행동'에 대한 정보를 관리하는 역할을 한다. 즉, 시스템이 무엇을 할 수 있는지에 대한 정적인 지식 베이스(static knowledge base)이다.6 다른 노드들은 ROS2 서비스(service)를 통해 Domain Expert에게 "어떤 종류의 객체(types)가 있는가?", "move 액션의 전제조건(precondition)은 무엇인가?"와 같은 질의를 보낼 수 있다. 여러 도메인 파일을 콜론(

  `:`)으로 구분하여 동시에 로드하고 병합하는 기능도 지원하여, 모듈식 애플리케이션 개발을 용이하게 한다.32

- **Problem Expert (문제 전문가)**: 이 노드는 PDDL 문제 파일(`problem.pddl`)의 내용을 관리하며, 로봇의 현재 상태와 목표를 동적으로 유지한다. 즉, 세계가 '지금 어떤 상태인지'에 대한 동적인 지식 베이스(dynamic knowledge base)이다.6 로봇의 센서 데이터가 처리되어 "로봇은 지금 roomA에 있다" 

  `(at robot1 roomA)`와 같은 새로운 사실(predicate instance)이 Problem Expert에 추가될 수 있다. 또한, 사용자의 명령이나 외부 시스템의 요청에 따라 목표(`:goal`)가 실시간으로 변경될 수도 있다. Problem Expert는 상태가 변경될 때마다 토픽(topic)을 통해 다른 노드들에게 이를 알린다.

- **Planner (플래너)**: 계획 요청이 들어오면, Planner 노드는 Domain Expert로부터 도메인 정보를, Problem Expert로부터 현재 문제(초기 상태, 목표) 정보를 ROS2 서비스를 통해 받아온다. 이 정보들을 바탕으로 임시 PDDL 도메인 및 문제 파일을 생성하고, 이 파일들을 외부 PDDL 플래너에 전달한다.31 PlanSys2는 POPF, Temporal Fast Downward (TFD) 등 다양한 플래너를 플러그인(plugin) 형태로 지원하므로, 문제의 특성에 맞는 최적의 플래너를 선택하여 사용할 수 있다.31 플래너가 성공적으로 해(plan), 즉 행동 시퀀스를 찾아내면, Planner 노드는 그 결과를 파싱하여 Executor 노드에게 전달한다.

- **Executor (실행기)**: Executor는 Planner로부터 받은 추상적인 행동 시퀀스(예: `(move robot1 roomA roomB)`, `(pickup robot1 ball1 roomB)`)를 실제 로봇의 구체적인 행동으로 변환하여 실행하는, 가장 복잡하고 중요한 노드이다.6 PlanSys2는 단순히 계획을 순서대로 실행하는 데 그치지 않는다. 계획을 행동 트리(Behavior Trees, BT)로 변환하여 실행함으로써, 실행 중 발생하는 작은 오류에 강건하게 대처하고 유연성을 높인다.31 또한, 다중 로봇 환경에서는 '액션 경매(action auction)'라는 독특한 프로토콜을 사용한다. 특정 행동을 실행해야 할 때, Executor는 해당 행동을 수행할 수 있는 모든 로봇(action performer)에게 "이 일을 할 수 있는가?"라고 묻고, 가장 적합한 로봇을 선택하여 임무를 할당한다.31

### 4.3 Sense-Plan-Act 루프의 구현

PlanSys2의 아키텍처는 로봇 공학의 고전적인 '감지(Sense) - 계획(Plan) - 행동(Act)' 패러다임을 분산 시스템 환경에서 효과적으로 구현한다.30

1. **Sense (감지)**: 로봇의 카메라, 라이다 등 센서가 주변 환경을 감지한다. 이 원시 데이터는 다른 노드(e.g., SLAM, 객체 인식 노드)에 의해 처리되어 "로봇의 현재 위치는 (x, y)이다", "테이블 위에 컵이 있다"와 같은 기호적 정보로 변환되고, 이 정보는 **Problem Expert**의 상태를 업데이트한다.
2. **Plan (계획)**: 사용자로부터 "컵을 부엌으로 가져와라"는 새로운 목표가 주어지거나, 현재 상태가 예상과 크게 달라져 기존 계획이 무효화되면, **Planner** 노드가 호출된다. Planner는 Domain Expert와 업데이트된 Problem Expert의 정보를 바탕으로 새로운 계획을 생성한다.
3. **Act (행동)**: **Executor**는 생성된 계획의 첫 번째 단계를 실행한다. 예를 들어 `(navigate robot1 table)`이라는 행동을 로봇의 내비게이션 스택에 전달하여 로봇을 테이블로 이동시킨다. 행동의 실행 결과(성공, 실패)는 다시 Problem Expert의 상태에 반영되며, 이 순환 구조를 통해 로봇은 동적인 환경 변화에 지속적으로 대응하며 임무를 수행하게 된다.

### 4.4 Table 2: PlanSys2 아키텍처 구성 요소 및 역할

PlanSys2의 아키텍처는 PDDL이라는 추상적인 언어가 어떻게 실제 로봇 시스템의 구체적인 소프트웨어 모듈로 구현되는지를 보여주는 가장 좋은 예시다. 각 노드의 역할과 그들이 처리하는 PDDL 요소를 명확히 매핑하면, 이론과 실제 사이의 연결고리를 직관적으로 이해할 수 있다.

| 구성 요소 (Node)   | 핵심 역할                   | 관련 PDDL 요소                                               | 주요 특징                                                    |
| ------------------ | --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Domain Expert**  | 도메인 지식 관리 및 제공    | `domain.pddl` 전체 (`:requirements`, `:types`, `:predicates`, `:functions`, `:action` 등) | 정적(Static) 지식 베이스. 시스템의 '규칙'과 '능력'을 정의하며, 한 번 로드되면 거의 변하지 않음. |
| **Problem Expert** | 현재 문제 상태 및 목표 관리 | `problem.pddl` 전체 (`:objects`, `:init`, `:goal`)           | 동적(Dynamic) 지식 베이스. 실시간 센서 데이터와 외부 명령에 의해 `(:init)` 상태와 `(:goal)`이 계속 업데이트됨. |
| **Planner**        | 계획(행동 순서) 생성        | `domain.pddl`과 `problem.pddl`을 입력으로 사용               | 외부 PDDL 솔버(POPF, TFD 등)를 플러그인으로 활용. 계획 생성을 온디맨드 서비스로 제공. |
| **Executor**       | 생성된 계획의 실행 및 감독  | 생성된 Plan (Action Sequence)                                | 행동 트리(Behavior Trees)를 이용한 유연하고 강건한 실행. 다중 로봇 조정을 위한 액션 경매(Auction) 메커니즘. |

이 표는 복잡한 ROS2 시스템 아키텍처를 PDDL의 개념적 틀에 맞춰 명료하게 정리해준다. PDDL의 `domain`과 `problem`이 각각 `Domain Expert`와 `Problem Expert`라는 별도의 노드로 관리되는 이유는 정적 지식과 동적 상태를 분리하기 위함이며, 이는 시스템의 모듈성과 확장성을 크게 향상시킨다.

## 5.  대안적 접근법과의 비교 고찰

PDDL은 강력한 표준이지만, 로봇의 작업 계획을 위한 유일한 방법론은 아니다. 문제의 성격과 요구사항에 따라 다른 접근법이 더 효과적일 수 있다. PDDL을 다른 주요 방법론들과 비교함으로써 그 특징과 적합한 적용 분야를 더 명확히 이해할 수 있다.

### 5.1 PDDL vs. ASP (Answer Set Programming): 논리적 추론의 두 갈래

- **핵심 차이**: PDDL은 본질적으로 계획 문제 해결을 위해 명시적으로 설계된 액션 기술 언어인 반면, ASP는 더 넓은 범위의 지식 표현 및 추론 문제를 위한 선언적 논리 프로그래밍 패러다임이다.1 두 방법론의 가장 근본적인 차이는 추론 방식에 있다. PDDL의 공리 기반 추론은 **단조적(monotonic)**이다. 즉, 새로운 정보가 추가되어도 이전에 유도된 결론은 변하지 않는다. 반면, ASP는 

  **비단조적(non-monotonic)** 추론을 지원하여, 새로운 정보가 주어지면 이전에 참이었던 결론을 철회하고 새로운 결론을 도출할 수 있다.1 이는 '달리 명시되지 않는 한, 사실은 변하지 않는다'와 같은 관성(inertia)에 대한 상식적 추론을 더 자연스럽게 표현하게 해준다.

- **성능 비교**: 경험적 연구에 따르면, 어떤 종류의 문제인지에 따라 두 시스템의 성능이 명확하게 갈린다.1

  - **PDDL의 강점**: **계획의 길이가 긴 문제(long solutions)**에서 더 나은 성능을 보인다. 이는 국제 계획 대회(IPC)의 벤치마크 문제들처럼, 복잡한 추론보다는 긴 순차적 행동을 효율적으로 생성하는 데 최적화된 플래너들이 많기 때문이다.1
  - **ASP의 강점**: **객체의 수가 많은 문제(large number of objects)**나, 행동의 전제조건 및 효과를 추론하기 위해 **복잡한 논리적 추론이 필요한 문제**에서 우세하다. 특히 재귀적인 효과(e.g., "A를 옮기면 연결된 B도 옮겨지고, B에 연결된 C도 옮겨진다")나 복잡한 제약조건이 있는 도메인에서 강력한 성능을 발휘한다.1

- **로봇 공학적 함의**: 이 비교는 로봇 공학자에게 중요한 선택 기준을 제공한다. 만약 문제가 주로 "어떤 순서로 여러 장소를 방문할 것인가?"와 같이 긴 행동 시퀀스를 찾는 것이라면 PDDL 기반 플래너가 더 적합할 수 있다. 반면, 문제가 "수많은 부품들 중, 특정 조건을 만족하는 부품들만을 사용하여 조립을 완료하라"와 같이 복잡한 환경 규칙과 제약조건 속에서 유효한 행동을 찾아야 한다면 ASP 기반 플래너가 더 나은 선택일 수 있다.

### 5.2 PDDL vs. 행동 트리 (BT) vs. HTN: 계획, 반응, 절차의 스펙트럼

이 세 가지 방법론은 로봇의 행동을 결정하는 서로 다른 철학을 대표하며, 계획(deliberation), 반응(reaction), 절차(procedure)라는 스펙트럼 위에 위치한다.

- **근본적 철학의 차이**:
  - **PDDL (계획 기반)**: '무엇을(What)' 달성할지(목표, goal)를 명시하면, '어떻게(How)' 할지(계획, plan)를 **탐색(search)하고 발견(discover)**하는 접근법이다.33 플래너는 주어진 초기 상태에서 목표 상태에 도달하는 유효한 행동 경로를 상태 공간에서 찾아낸다. 이는 매우 유연하고 새로운 문제에 대한 창의적인 해결책을 찾을 수 있지만, 탐색 공간이 클 경우 계산 비용이 기하급수적으로 증가하는 단점이 있다.35
  - **행동 트리 (BT) (반응/스크립트 기반)**: '어떻게' 행동할지를 **미리 설계(design)하고 규정(prescribe)**하는 접근법이다.33 BT는 "만약 A 조건이 참이면 B 행동을 하라"와 같은 규칙들을 계층적인 트리 구조로 구성한다. 시스템은 매 순간(tick) 트리의 루트부터 탐색하여 현재 상황에 가장 적절한 행동을 '선택'하여 실행한다. 이는 예측 불가능한 환경 변화에 즉각적으로 반응할 수 있어 강건하고, 모듈화가 용이하여 널리 사용되지만, 사전에 정의되지 않은 장기적인 전략이나 새로운 목표를 달성하기 위한 계획을 생성할 수는 없다.36
  - **HTN (Hierarchical Task Network) (절차/분해 기반)**: PDDL과 BT의 중간적 성격을 띤다. HTN은 고수준의 추상적인 작업(e.g., "집 청소하기")을 더 구체적인 하위 작업들(e.g., "거실 청소", "부엌 청소")로 분해하는 '방법(method)' 또는 '레시피'를 제공한다.33 플래너는 이 레시피에 따라 주어진 작업을 계층적으로 분해하여 최종적인 원시 행동(primitive action) 계획을 생성한다. 이는 순수 PDDL보다 더 구조화되고 예측 가능한 계획을 유도할 수 있어, 전문가의 작업 절차 지식을 시스템에 효과적으로 인코딩할 수 있다.34

이러한 방법론들 간의 논쟁은 점차 '어느 것이 우월한가'에서 '어떻게 각자의 강점을 융합할 것인가'로 변화하고 있다. 성공적인 최신 로봇 시스템들은 이들을 상호 배타적인 선택지로 보지 않고, 아키텍처의 다른 계층에서 서로를 보완하는 도구로 활용하는 경향이 뚜렷하다. 이는 '전략(Strategy)'을 담당하는 상위 계획 계층과 '전술(Tactics)'을 담당하는 하위 실행 계층을 분리하는 고전적인 군사 및 경영 아이디어의 현대적 공학 구현으로 볼 수 있다.

예를 들어, PDDL 플래너는 장기적이고 목표 지향적인 '전략적 계획'(e.g., "A블록을 B위로 옮기고, 그 다음에 C블록을 A위로 옮겨라")을 생성하는 데 탁월하다.36 그러나 이 계획은 장애물이 없고 모든 행동이 성공하는 이상적인 환경을 가정한다. 행동 트리(BT)는 예측 불가능한 현실 세계에서 이 계획의 각 단계를 '전술적으로 실행'하는 데 강점을 보인다.36 "A블록을 B위로 옮겨라"는 전략적 지시를 받았을 때, 만약 A블록 앞에 예기치 않은 장애물이 나타난다면, BT는 '장애물 피하기' 행동을 먼저 실행한 후 목표를 재시도하는 등 강건하고 유연한 실행을 보장할 수 있다.

이러한 융합의 구체적인 예시는 다수 발견된다. 본 보고서에서 분석한 PlanSys2는 PDDL로 계획을 생성하고 BT로 실행하는 대표적인 사례이다.31 한 연구는 HTN으로 고수준의 협력 계획을 짜고, 각 에이전트의 개별 행동 단계를 BT로 실행하여 반응성을 높이는 하이브리드 접근법을 제안한다.36 또 다른 연구는 BT와 HTN의 장점을 결합한 eBT(Extended Behavior Trees)라는 새로운 모델을 제안하며, 심지어 PDDL 플래너를 초기 계획 생성에 보조적으로 사용하기도 한다.34

결론적으로, 미래의 지능형 로봇 아키텍처는 단일 방법론에 의존하기보다, PDDL이나 HTN과 같은 계획 시스템이 '무엇을, 어떤 순서로 할 것인가'라는 전략적 지침을 제공하고, BT와 같은 반응형 시스템이 '지금 당장 어떻게 할 것인가'라는 전술적 실행을 책임지는 다층적(multi-layered) 구조로 수렴될 가능성이 높다. 이는 각 기술의 장점을 극대화하고 단점을 상쇄하는 가장 실용적이고 강력한 설계 패턴으로 자리 잡고 있다.

### 5.3 Table 3: 로봇 작업 계획 방법론 비교 (PDDL, ASP, BT, HTN)

네 가지 주요 방법론은 각기 다른 철학과 장단점을 가지고 있어, 로봇 공학자가 자신의 문제에 가장 적합한 도구를 선택하는 데 어려움을 겪을 수 있다. 다음 표는 각 방법론의 핵심 특징을 정량적, 정성적으로 비교하여 정보에 입각한 의사결정을 지원한다.

| 기준               | PDDL (STRIPS-style)                                          | ASP (Answer Set Programming)                                 | 행동 트리 (Behavior Tree)                              | HTN (Hierarchical Task Network)                              |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------------ |
| **패러다임**       | 상태 공간 탐색 기반 계획                                     | 논리 프로그래밍 기반 추론                                    | 반응형, 스크립트 기반 제어                             | 계층적 작업 분해 기반 계획                                   |
| **계획 생성**      | **발견(Discover):** 초기 상태에서 목표 상태까지의 경로를 탐색 | **발견(Discover):** 논리적 제약을 만족하는 해(answer set)를 추론 | **규정(Prescribe):** 미리 설계된 트리를 따라 행동 선택 | **분해(Decompose):** 추상적 작업을 구체적 행동으로 계층적 분해 |
| **유연성/적응성**  | 낮음 (계획은 정적이며, 재계획 필요)                          | 중간 (비단조 추론으로 일부 적응성 제공)                      | **높음 (상태 변화에 매 순간 즉각 반응)**               | 중간 (미리 정의된 분해 방법에 강하게 의존)                   |
| **전략적 깊이**    | **높음 (전역 최적해 또는 만족해 탐색)**                      | 높음 (복잡한 논리적 제약 하의 전역적 해 탐색)                | 낮음 (지역적 결정의 연속, 장기 전략 부재)              | 중간 (계층적 구조를 통해 전략 표현 가능)                     |
| **표현력**         | 중간 (PDDL 확장으로 강화 가능)                               | **높음 (복잡한 논리, 비단조 추론, 제약 표현)**               | 낮음 (단순한 조건-행동의 조합)                         | 중간 (절차적(procedural) 지식 표현에 강점)                   |
| **주요 강점**      | 긴 순차 계획, 표준화, 다양한 플래너 존재                     | 다수의 객체, 복잡한 제약/규칙, 비단조 추론                   | 강건한 실행, 모듈성, 높은 반응성, 직관적 설계          | 전문가 지식 활용, 구조화되고 예측 가능한 계획                |
| **주요 약점**      | 상태 공간 폭발, 기호 접지 문제, 동적 환경 취약               | 특정 유형 문제 외 성능 저하 가능성, 덜 직관적                | 목표 지향적 계획 생성 불가, 장기 전략 부재             | 정의되지 않은 새로운 문제 해결 불가                          |
| **주요 적용 분야** | 물류, 고전적 로봇 조작, 퍼즐 문제                            | 자원 할당, 스케줄링, 복잡한 규칙 기반 시스템                 | 게임 AI, 로봇의 저수준 반응형 제어, 안전 시스템        | 복잡한 조립 공정, 제조, 다단계 임무 수행                     |

## 6.  한계와 도전: PDDL이 넘어야 할 산

PDDL은 AI 계획 분야에 큰 기여를 했지만, 복잡하고 동적인 현실 세계의 로봇 문제에 직접 적용하기에는 여러 근본적인 한계와 도전을 안고 있다. 이러한 한계들은 PDDL의 실용성을 저해하는 주요 요인으로 작용한다.

### 6.1 계산 복잡성 및 확장성 (Computational Complexity & Scalability)

PDDL 기반 계획은 근본적으로 계산적으로 매우 어려운 문제이다. 가장 단순한 형태인 STRIPS 계획 문제조차 **PSPACE-hard**라는 높은 계산 복잡도 등급에 속한다.8 이는 문제의 크기, 즉 객체의 수나 가능한 행동의 종류가 조금만 증가해도, 해를 찾는 데 필요한 시간과 메모리가 최악의 경우 기하급수적으로 증가할 수 있음을 의미한다. 이 현상을 **'상태 공간 폭발(state-space explosion)'**이라고 부르며, 이는 PDDL을 복잡한 실제 로봇 문제에 적용하는 데 있어 가장 큰 기술적 걸림돌이다.8 예를 들어, 10개의 블록을 쌓는 문제의 가능한 상태 수는 수백만 개에 달하며, 이는 플래너가 모든 가능성을 탐색하는 것을 사실상 불가능하게 만든다.

### 6.2 기호 접지 문제 (The Symbol Grounding Problem)

PDDL은 본질적으로 추상적인 기호의 세계에 머물러 있다. 플래너는 `(at robot waypoint1)`이나 `(holding cup)`과 같은 기호적 사실(symbolic fact)을 다룬다. 그러나 로봇이 실제로 상호작용하는 세계는 연속적이고 노이즈가 낀 센서 데이터로 이루어져 있다. '기호 접지 문제'는 바로 이 추상적인 기호와 로봇의 구체적인 센서-모터 데이터 사이의 간극을 어떻게 메울 것인가에 대한 문제이다.10

예를 들어, `(at robot waypoint1)`이라는 기호적 사실은 로봇의 실제 물리적 위치(GPS 좌표, SLAM 맵 상의 포즈)와 어떻게 연결되는가? `(holding cup)`이라는 사실은 로봇 그리퍼의 힘 센서 값이나 카메라 이미지로부터 어떻게 판단할 수 있는가? 이 문제를 해결하지 못하면, 플래너가 아무리 훌륭한 계획을 생성하더라도 로봇에게는 아무 의미 없는 기호의 나열에 불과하다.11 많은 연구들이 이 문제를 해결하기 위해 자연어 처리나 딥러닝과 같은 기술을 활용하여, 비정형적인 환경 정보로부터 의미 있는 기호를 추출하고 이를 로봇 간에 공유하는 방법을 제안하고 있다.10

### 6.3 동적 환경과 재계획 (Dynamic Environments & Replanning)

로봇이 작동하는 실제 환경은 실험실처럼 통제되어 있지 않고, 예측 불가능하게 계속해서 변한다. 로봇이 복도를 따라 이동하는 중에 사람이 갑자기 나타나 길을 막거나, 로봇이 옮기던 물건이 미끄러져 떨어지는 등 예기치 않은 사건이 발생하면, 기존에 수립된 계획은 즉시 무효가 된다.39

- **도전 과제**: 이러한 상황에서 로봇은 다음과 같은 일련의 과정을 신속하게 처리해야 한다: 1) 센서를 통해 환경의 변화를 감지하고, 2) 현재 계획이 더 이상 유효하지 않음을 인지하며, 3) 새로운 상황에 맞춰 기존 계획을 부분적으로 수정(plan repair)하거나, 완전히 새로운 계획을 생성(replanning)해야 한다.39
- **어려움**: 실시간 재계획은 매우 어려운 과제이다. 첫째, 새로운 계획을 생성하는 데 시간이 너무 오래 걸리면 로봇은 그저 멈춰 서 있게 되어, 특히 인간과의 상호작용에서 자연스러움(fluency)과 효율성을 크게 해친다.40 둘째, 재계획의 기반이 되는 센서 데이터 자체가 불완전하거나 부정확할 수 있다.42 잘못된 정보에 기반한 재계획은 오히려 상황을 더 악화시키는 결과를 초래할 수도 있다. 이러한 문제를 해결하기 위해, PDDL+와 같이 외부 사건을 모델링하여 계획에 미리 반영하거나 21, 계획과 실행을 긴밀하게 연동하여 온라인으로 지속적인 계획 수정 및 생성을 수행하는 아키텍처 28가 필수적으로 요구된다.

이러한 PDDL의 주요 한계들-계산 복잡성, 기호 접지, 재계획의 어려움-은 서로 독립적인 문제가 아니라, 서로를 악화시키는 악순환 구조를 형성한다. 이 상호 증폭 효과가 순수 PDDL 기반 시스템이 실제 동적 환경에서 종종 실패하는 근본적인 원인이다. 이 악순환의 고리는 다음과 같이 작동한다.

1. **시작점은 기호 접지의 어려움이다.** 동적인 실제 환경에서 로봇은 센서 데이터를 통해 현재 세계 상태를 파악해야 한다. 하지만 기호 접지 문제 때문에, 노이즈가 끼고 연속적인 센서 데이터를 PDDL의 깨끗하고 이산적인 `(:init)` 상태로 완벽하게 변환하는 것 자체가 매우 어렵고 불완전하다.42
2. **이것이 1차적으로 부정확한 계획을 낳는다.** 부정확하거나 불완전한 세계 상태(`(:init)`)를 기반으로 계획을 시작하면, 플래너는 애초에 잘못된 전제 위에서 작동하게 된다. 아무리 강력한 플래너라도 '쓰레기 입력(garbage in)'에서는 '쓰레기 출력(garbage out)'을 내놓을 가능성이 높다.
3. **2차적으로 재계획의 실패를 유발한다.** 예기치 않은 사건이 발생하여 재계획이 필요해졌을 때, 로봇은 다시 현재 상태를 파악해야 하지만 기호 접지 문제는 여전히 존재한다. 따라서 재계획의 시작점 역시 부정확할 가능성이 높다.
4. **3차적으로 계산 복잡성으로 인한 지연이 상황을 악화시킨다.** 설상가상으로, 재계획은 신속해야 한다. 그러나 상태 공간 폭발 문제 때문에, 복잡한 상황에서는 새로운 계획을 계산하는 데 수 초에서 수 분이 걸릴 수 있다.8 이 시간 동안 환경은 더 변할 수 있고, 힘들게 계산한 새로운 계획은 생성되자마자 다시 구식이 되어 무효화될 수 있다.
5. **결과적으로 악순환이 완성된다.** (1) 기호 접지의 어려움으로 인해 (2) 부정확한 계획이 생성되고, (3) 동적 환경 변화로 재계획이 필요해지면, (4) 계산 복잡성으로 인한 지연과 (1)의 문제가 결합되어 (5) 또다시 부정확하고 시대에 뒤떨어진 계획이 생성된다. 이 순환 고리가 바로 순수 PDDL 시스템이 실제 로봇에 널리 적용되기 어려웠던 핵심적인 이유이며, 현대 로봇 공학이 PDDL을 다른 기술과 융합하려는 강력한 동기가 되었다.

## 7.  미래를 향한 도약: 신경-기호주의적 접근 (Neuro-Symbolic Approaches)

### 7.1 배경: 한계 극복을 위한 패러다임 전환

앞서 논의된 PDDL의 근본적인 한계들, 특히 기호 접지와 확장성 문제는 순수한 기호주의(symbolicism) 접근만으로는 해결하기 어렵다는 공감대를 형성했다. 이에 대한 가장 유망한 해답으로, 기호주의 계획(PDDL)이 가진 논리적 엄밀함과 설명 가능성, 그리고 신경망(neural networks)으로 대표되는 머신러닝의 유연한 패턴 인식 및 일반화 능력을 결합하려는 **'신경-기호주의(Neuro-Symbolic)'** 접근법이 부상했다.8 이 패러다임은 각 접근법의 장점을 취하고 단점을 보완함으로써, 로봇 지능을 한 단계 끌어올릴 잠재력을 가지고 있다.

### 7.2 VLM/LLM을 이용한 PDDL 문제 자동 생성

신경-기호주의 접근법 중 가장 직접적이고 실용적인 응용은 기호 접지 문제와 PDDL 파일 작성의 어려움을 해결하는 것이다. 이 분야에서 비전-언어 모델(VLM)과 대규모 언어 모델(LLM)이 핵심적인 역할을 한다.

- **Image2PDDL 및 유사 프레임워크**: 이들 프레임워크의 핵심 아이디어는 VLM을 사용하여 이미지(초기 상태)와 자연어 또는 이미지(목표 상태)로부터 PDDL 문제 파일(`problem.pddl`)을 자동으로 생성하는 것이다.8 이는 PDDL 사용의 가장 큰 진입 장벽 중 하나였던 수동 모델링 작업을 자동화한다.

- **작동 원리**: 이 과정은 일반적으로 다음과 같은 단계로 이루어진다.

  1. **상태 인식 및 번역**: VLM(예: GPT-4o)이 입력된 이미지(예: 로봇의 카메라가 촬영한 테이블 위 장면)를 분석하여, 이미지 내의 객체들(블록, 컵 등), 그들의 속성(색상, 크기), 그리고 객체들 간의 공간적 관계(`on`, `in`, `clear` 등)를 텍스트 형태로 추출한다.8

  2. **PDDL 초기 상태 생성**: 이 추출된 텍스트 정보는 미리 정의된 PDDL 도메인 파일과 결합되어, 문제 파일의 `(:objects)`와 `(:init)` 섹션을 자동으로 채우는 데 사용된다.9 예를 들어, VLM이 "파란색 블록 위에 빨간색 블록이 있다"고 인식하면, 이는 

     `(on red-block blue-block)`이라는 PDDL 술어로 변환된다.

  3. **PDDL 목표 상태 생성**: 사용자가 "블록들을 알파벳 순서로 쌓아라"와 같이 자연어로 목표를 제시하거나 목표 상태의 이미지를 보여주면, 모델은 이를 분석하여 `(:goal)` 섹션을 생성한다.9

- **의의**: 이 접근법은 로봇의 '지각(perception)'과 '계획(planning)'이라는, 전통적으로 분리되어 있던 두 영역 사이의 간극을 메우는 강력한 다리 역할을 한다. 더 이상 인간 전문가가 센서 데이터를 보고 수동으로 세계 상태를 기호화할 필요가 없어져, PDDL의 접근성과 확장성을 획기적으로 향상시킨다.12

### 7.3 LLM을 활용한 계획 프로세스 가속 및 효율화

상태 공간 폭발이라는 계산 복잡성 문제를 완화하기 위해 LLM의 상식적 추론 능력과 패턴 인식 능력을 활용하는 연구도 활발히 진행되고 있다.

- **계층적 계획 및 부 목표 생성 (Hierarchical Planning & Subgoal Generation)**: 복잡하고 긴 작업(long-horizon task)을 한 번에 해결하려는 대신, LLM을 이용해 더 작고 관리하기 쉬운 하위 목표(subgoal)들로 문제를 분해한다. 그 후, PDDL 플래너는 각 하위 목표에 대한 짧고 간단한 계획들을 순차적으로 해결한다. 예를 들어, "방을 청소하고 요리하기"라는 큰 목표를 "1. 바닥의 물건 줍기", "2. 진공청소기 돌리기", "3. 재료 손질하기", "4. 냄비에 넣고 끓이기"와 같은 하위 목표들로 나누는 것이다. 이 방식은 전체 문제의 탐색 공간을 효과적으로 줄여 계획 시간을 극적으로 단축시킬 수 있다.8
- **경험 기반 계획 (Experience-based Planning)**: LLM을 특정 도메인에 대한 수많은 문제-해결 쌍 데이터로 훈련시켜, 해당 도메인에 대한 '상식'이나 '경험적 지식'을 갖게 한다. 이를 통해 LLM은 주어진 상황에서 성공할 가능성이 높은 유망한 행동들을 제안하거나, 명백히 비효율적인 탐색 경로를 미리 제거(pruning)함으로써 PDDL 플래너의 탐색 부담을 덜어줄 수 있다.40 예를 들어, Teriyaki와 같은 연구는 LLM을 PDDL과 호환되는 형식의 플래너로 직접 훈련시켜, 기존 기호주의 플래너보다 계획 시간을 단축하고, 특히 인간-로봇 상호작용에서 응답 지연을 줄여 상호작용의 유창함(fluency)을 개선하는 데 성공적인 결과를 보였다.40

이러한 신경-기호주의 패러다임의 발전은 PDDL 플래너의 역할을 근본적으로 재정의하고 있다. 전통적인 AI 계획에서 플래너는 문제 정의부터 해결책 탐색까지 모든 것을 책임지는 '창의적인 문제 해결사'였다. 그러나 이제 그 역할은 '엄격한 논리 검증기 및 시퀀서'로 점차 변화하고 있다. LLM이 인간의 직관이나 상식에 기반하여 '그럴듯한(plausible)' 계획의 초안이나 아이디어를 생성하면, PDDL 플래너는 그 아이디어가 주어진 도메인의 엄격한 물리 법칙 하에서 실제로 '가능한지(valid)'를 검증하고, 가능하다면 구체적인 행동 순서로 정교하게 다듬는 역할을 맡게 되는 것이다.

이러한 역할 분담의 논리는 명확하다. LLM이 생성한 계획 아이디어는 종종 '환각(hallucination)'을 포함하거나 물리 법칙을 무시하는 등 논리적 결함을 가질 수 있다.8 예를 들어, LLM은 "벽을 통과해서 공을 집어라"와 같이 문법적으로는 그럴듯하지만 물리적으로 불가능한 계획을 제안할 수 있다. 바로 이 지점에서 PDDL 플래너의 새로운 가치가 드러난다. PDDL 도메인은 해당 세계의 '불변의 물리 법칙'을 형식 논리로 명확하게 정의한다. PDDL 플래너는 LLM이 제안한 (하위) 목표나 계획 스케치를 입력받아, 이 '물리 법칙' 하에서 해당 목표가 실제로 달성 가능한지, 가능하다면 어떤 정확한 행동 순서로 가능한지를 엄격하게 검증하고 계산한다.

ViPlan 벤치마크 연구는 이러한 역할 분담의 효과를 잘 보여준다.49 이 연구는 'VLM-as-planner'(VLM이 직접 행동을 제안) 방식과 'VLM-as-grounder'(VLM이 PDDL 술어의 참/거짓을 판단하고 기호주의 플래너가 계획) 방식을 비교했다. 그 결과, 정확한 시각적 접지(visual grounding)가 중요한 블록 쌓기 문제에서는 후자가 더 우수한 성능을 보였고, 상식과 오류 회복 능력이 중요한 가정 환경 시뮬레이션에서는 전자가 더 나은 성능을 보였다. 이는 두 접근법의 강점이 서로 다르며, 문제의 특성에 따라 상호 보완적으로 사용되어야 함을 시사한다.

결론적으로, 미래의 신경-기호주의 시스템에서 PDDL은 LLM의 창의적이지만 종종 비논리적인 출력을 '필터링'하고 '구체화'하는 핵심적인 역할을 맡게 될 것이다. 이는 PDDL의 활용 범위를 좁히는 것이 아니라, 오히려 전체 지능형 시스템에서 가장 신뢰할 수 있는 '논리적 앵커(logical anchor)'로서 그 중요성을 더욱 공고히 하는 방향으로의 진화라고 할 수 있다.

## 8.  결론: 로봇 지능의 청사진으로서 PDDL의 현재와 미래

### 보고서 핵심 요약

본 보고서는 로봇의 자율적 임무 수행을 위한 핵심 언어인 PDDL에 대한 기술적 심층 고찰을 수행했다. PDDL은 1998년 학술적 연구의 표준화를 위해 탄생하여 3, 도메인과 문제를 분리하는 명확한 철학을 제시했다. 이후 국제 계획 대회를 거치며 PDDL 2.1에서 시간과 자원을, PDDL 2.2와 PDDL+ 등에서 외부 사건과 불확실성을 모델링하는 기능을 추가하며 현실 세계의 복잡성에 점차 가까워졌다.3 이러한 이론적 발전은 ROS2 PlanSys2와 같은 현대적 프레임워크를 통해 실제 로봇 시스템에 깊숙이 통합되었으며, 이는 PDDL이 더 이상 실험실의 도구가 아님을 증명한다.6

또한, ASP, 행동 트리, HTN과 같은 대안적 접근법과의 비교 분석을 통해, 각 방법론이 계획, 추론, 반응, 절차라는 스펙트럼 위에서 서로 다른 강점과 약점을 가지며, 현대 로봇 시스템은 이들을 융합하는 방향으로 발전하고 있음을 확인했다.1 그러나 PDDL은 계산 복잡성, 기호 접지, 동적 환경 대응의 어려움이라는 근본적인 한계를 내포하고 있으며, 이 문제들이 서로를 증폭시키는 악순환 구조를 형성함도 명확히 지적했다.8

### 8.1 PDDL의 현재적 가치와 미래 전망

이러한 한계에도 불구하고 PDDL의 현재적 가치는 그 '엄격한 형식주의(formalism)'에 있다. PDDL로 기술된 도메인과 계획은 그 의미가 명확하며, 이는 시스템의 행동에 대한 **해석 가능성(interpretability)**, **검증 가능성(verifiability)**, 그리고 **예측 가능성(predictability)**을 제공한다.50 이러한 특성은 안전이 최우선시되는 로봇 시스템, 특히 인간과 함께 작업하는 환경에서 무엇보다 중요한 가치이다.

PDDL의 미래는 단독으로 모든 것을 해결하는 '만능 해결사'가 되는 것이 아니라, 신경망 모델과의 융합을 통해 새로운 역할을 찾는 데 있다. 본 보고서에서 심도 있게 논의한 신경-기호주의 아키텍처가 가장 유망한 미래상이다. 이 구조에서 VLM/LLM은 로봇의 '눈과 귀, 그리고 직관' 역할을 하여 비정형적인 시각 및 언어 데이터를 이해하고 문제 상황을 정의한다. 그러면 PDDL 플래너는 시스템의 '이성적 두뇌' 역할을 하여, 주어진 물리 법칙과 제약 조건 하에서 논리적으로 타당하고 실행 가능한 행동 계획을 생성하고 검증한다.8 PDDL은 LLM의 창의적이지만 때로는 비논리적인 제안들을 걸러내는 신뢰할 수 있는 '논리적 앵커'가 되는 것이다.

### 8.2 최종 통찰 및 향후 연구 방향

결론적으로, PDDL은 죽은 언어가 아니라, AI의 새로운 패러다임 속에서 자신의 위치를 성공적으로 재정의하며 진화하고 있다. 향후 연구는 다음과 같은 방향으로 집중될 것이다.

1. **효율적인 계획 알고리즘**: PDDL의 풍부한 표현력(특히 PDDL+ 수준)을 완전히 활용하면서도, 상태 공간 폭발 문제를 완화하여 실시간에 가까운 계획 및 재계획을 가능하게 하는 새로운 플래닝 알고리즘 개발이 시급하다.
2. **매끄러운 신경-기호 인터페이스**: LLM/VLM의 출력(자연어, 구조화된 텍스트)과 PDDL 플래너의 입력(형식적 기호) 사이의 변환을 더욱 정교하고 강건하게 만드는 인터페이스 설계가 중요하다. 오류 감지 및 자동 수정 메커니즘을 포함하여 양방향 소통을 원활하게 해야 한다.
3. **실패로부터의 학습**: 로봇이 PDDL 계획을 실행하다 실패했을 때, 그 실패의 원인을 분석하여 PDDL 도메인 모델 자체(행동의 전제조건이나 효과)를 자동으로 수정하고 개선하는 '모델 학습(model learning)' 연구가 필요하다. 이는 시스템이 경험을 통해 점차 더 똑똑해지게 만들 것이다.

궁극적으로 PDDL은 로봇이 어떻게 세상을 인식하고, 목표를 설정하며, 그 목표를 달성하기 위해 '생각하고 행동하는지' 그 과정을 이해하고 설계하기 위한 가장 강력하고 투명한 청사진 중 하나로써, 앞으로도 로봇 지능 연구의 중심에서 그 가치를 계속해서 증명해 나갈 것이다.

#### **참고 자료**

1. Task planning in robotics: an empirical comparison of PDDL- and ..., accessed July 23, 2025, https://www.cs.utexas.edu/~pstone/Papers/bib2html-links/FITEE19-jiang.pdf
2. Task & Motion Planning and Behaviour Trees in Robotics | by Sunny Katyara | Medium, accessed July 23, 2025, https://medium.com/@sunny.sunny_13233/task-motion-planning-and-behaviour-trees-in-robotics-e00faeeb6cb1
3. Planning Domain Definition Language - Wikipedia, accessed July 23, 2025, https://en.wikipedia.org/wiki/Planning_Domain_Definition_Language
4. Mastering PDDL in Robotics - Number Analytics, accessed July 23, 2025, https://www.numberanalytics.com/blog/mastering-pddl-in-robotics
5. PDDL Tutorial - PDDL4J 4.0 documentation, accessed July 23, 2025, http://pddl4j.imag.fr/pddl_tutorial.html
6. ros2_planning_system - ROS Index, accessed July 23, 2025, https://index.ros.org/r/ros2_planning_system/
7. PlanSys2/ros2_planning_system: This repo contains a PDDL-based planning system for ROS2. - GitHub, accessed July 23, 2025, https://github.com/PlanSys2/ros2_planning_system
8. Fast and Accurate Task Planning using Neuro-Symbolic Language ..., accessed July 23, 2025, https://arxiv.org/pdf/2409.19250
9. Planning with Vision-Language Models and a Use Case in Robot-Assisted Teaching - arXiv, accessed July 23, 2025, https://arxiv.org/html/2501.17665v1
10. PDDL Planning with Natural Language-Based Scene Understanding for UAV-UGV Cooperation - MDPI, accessed July 23, 2025, https://www.mdpi.com/2076-3417/9/18/3789
11. Combining Planning and Motion Planning - DROPS, accessed July 23, 2025, https://drops.dagstuhl.de/storage/16dagstuhl-seminar-proceedings/dsp-vol10081/DagSemProc.10081.6/DagSemProc.10081.6.pdf
12. Planning with Vision-Language Models and a Use Case in Robot-Assisted Teaching, accessed July 23, 2025, https://www.researchgate.net/publication/388495284_Planning_with_Vision-Language_Models_and_a_Use_Case_in_Robot-Assisted_Teaching
13. [Literature Review] Planning with Vision-Language Models and a Use Case in Robot-Assisted Teaching - Moonlight | AI Colleague for Research Papers, accessed July 23, 2025, https://www.themoonlight.io/en/review/planning-with-vision-language-models-and-a-use-case-in-robot-assisted-teaching
14. An Introduction to PDDL, accessed July 23, 2025, http://www.cs.toronto.edu/~sheila/2542/w09/A1/introtopddl2.pdf
15. Writing Planning Domains and Problems in PDDL, accessed July 23, 2025, https://users.cecs.anu.edu.au/~patrik/pddlman/writing.html
16. PDDL2.1 : An Extension to PDDL for Expressing Temporal Planning ..., accessed July 23, 2025, https://gki.informatik.uni-freiburg.de/teaching/ws0607/aip/pddl2.1.pdf
17. PDDL Domain - Planning.wiki, accessed July 23, 2025, https://planning.wiki/ref/pddl/domain
18. PDDL 2.1 - Planning.wiki, accessed July 23, 2025, https://planning.wiki/ref/pddl21
19. Getting Started with PDDL | LearnPDDL - GitHub Pages, accessed July 23, 2025, https://fareskalaboud.github.io/LearnPDDL/
20. We'll describe PDDL, a standard for representing planning problems - UMBC, accessed July 23, 2025, https://courses.cs.umbc.edu/undergraduate/471/spring23/02/notes/13_planning/13.2_pddl.pdf
21. Real-World Planning with PDDL+ and Beyond - arXiv, accessed July 23, 2025, https://arxiv.org/html/2402.11901v1
22. pddl2.1 : An Extension to pddl for Expressing Temporal Planning Domains, accessed July 23, 2025, https://planning.wiki/_citedpapers/pddl212003.pdf
23. View of PDDL2.1: An Extension to PDDL for Expressing Temporal Planning Domains - Journal of Artificial Intelligence Research, accessed July 23, 2025, https://jair.org/index.php/jair/article/view/10352/24759
24. (PDF) PDDL2.1: An extension to PDDL for expressing temporal planning domains - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/220543188_PDDL21_An_extension_to_PDDL_for_expressing_temporal_planning_domains
25. AAAI-17 Tutorial on Planning and Robotics - GitHub Pages, accessed July 23, 2025, http://kcl-planning.github.io/ROSPlan//demos/conference_pages/slides/AAAI17_PlanningAndRobotics.pdf
26. Probabilistic Planning in Robotics - Number Analytics, accessed July 23, 2025, https://www.numberanalytics.com/blog/probabilistic-planning-robotics-guide
27. Probabilistic Planning for Robotics with ROSPlan - UPCommons, accessed July 23, 2025, https://upcommons.upc.edu/bitstream/handle/2117/180760/2190-Probabilistic-Planning-for-Robotics-with-ROSPlan.pdf;sequence=1
28. Tutorial 12 RDDL and Probabilistic Planning - GitHub Pages, accessed July 23, 2025, https://kcl-planning.github.io/ROSPlan//tutorials/tutorial_12
29. Nyx: Planning for Emerging Problems with PDDL+ and Beyond - ICAPS 2024, accessed July 23, 2025, https://icaps24.icaps-conference.org/program/workshops/keps-papers/KEPS-24_paper_18.pdf
30. PlanSys2: A Planning System Framework for ROS2 | Request PDF - ResearchGate, accessed July 23, 2025, https://www.researchgate.net/publication/357109831_PlanSys2_A_Planning_System_Framework_for_ROS2
31. PlanSys2: A Planning System Framework for ROS2 - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2107.00376
32. PlanSys2 design - ROS2 Planning System 2 1.0.0 documentation, accessed July 23, 2025, https://plansys2.github.io/design/index.html
33. What's the difference between Behavior Trees and Hierarchical Task Networks - Reddit, accessed July 23, 2025, https://www.reddit.com/r/gamedev/comments/1ozugf/whats_the_difference_between_behavior_trees_and/
34. (PDF) Extended behavior trees for quick definition of flexible robotic ..., accessed July 23, 2025, https://www.researchgate.net/publication/321820909_Extended_behavior_trees_for_quick_definition_of_flexible_robotic_tasks
35. LLM-as-BT-Planner: Leveraging LLMs for Behavior Tree Generation in Robot Task Planning - arXiv, accessed July 23, 2025, https://arxiv.org/html/2409.10444v2
36. A Hybrid Approach to Planning and Execution in Dynamic Environments Through Hierarchical Task Networks and Behavior Trees, accessed July 23, 2025, https://cdn.aaai.org/ojs/13044/13044-52-16561-1-2-20201228.pdf
37. A Survey of Behavior Trees in Robotics and AI - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2005.05842
38. Challenges in Modelling and Solving Plotting with PDDL *, accessed July 23, 2025, https://icaps23.icaps-conference.org/program/workshops/keps/KEPS-23_paper_9200.pdf
39. A Real-Time PDDL-Based Planning Component for Video Games - AAAI, accessed July 23, 2025, https://cdn.aaai.org/ojs/12370/12370-52-15898-1-2-20201228.pdf
40. A framework for neurosymbolic robot action planning using large language models - Frontiers, accessed July 23, 2025, https://www.frontiersin.org/journals/neurorobotics/articles/10.3389/fnbot.2024.1342786/pdf
41. MIT Open Access Articles Online Replanning in Belief Space for Partially Observable Task and Motion Problems, accessed July 23, 2025, https://dspace.mit.edu/bitstream/handle/1721.1/130011/1911.04577.pdf?sequence=2&isAllowed=y
42. Towards Safe and Reliable Robot Task Planning - CEUR-WS.org, accessed July 23, 2025, https://ceur-ws.org/Vol-2640/paper_25.pdf
43. 인공지능이 결합된 산업용로봇 의 제조 현장 적용방안, accessed July 23, 2025, https://www.hellot.net/data/banner/202209/20220916546828.pdf
44. Planning with Vision-Language Models and a Use Case in Robot-Assisted Teaching - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2501.17665
45. Planning with Vision-Language Models and a Use Case in Robot, accessed July 23, 2025, https://chatpaper.com/chatpaper/paper/103463
46. [2311.00967] Vision-Language Interpreter for Robot Task Planning - arXiv, accessed July 23, 2025, https://arxiv.org/abs/2311.00967
47. From Language to Action with Object-Level Planning, accessed July 23, 2025, https://dyalab.mines.edu/2025/icra-workshop/7.pdf
48. Achieving Scalable Robot Autonomy via neurosymbolic planning using lightweight local LLM - arXiv, accessed July 23, 2025, https://arxiv.org/html/2505.08492v1
49. ViPlan: A Benchmark for Visual Planning with Symbolic Predicates and Vision-Language Models - arXiv, accessed July 23, 2025, https://arxiv.org/pdf/2505.13180
50. A Survey of Robotic Language Grounding: Tradeoffs between Symbols and Embeddings, accessed July 23, 2025, https://arxiv.org/html/2405.13245v2


# 계획 영역 정의 언어(Planning Domain Definition Language, PDDL)
[임무 계획 (Mission Planning)](./index.md)



계획 영역 정의 언어(Planning Domain Definition Language, PDDL)는 인공지능(AI) 계획 문제의 명세를 표준화하기 위해 설계된 형식적이고 인간이 읽을 수 있는 언어이다.1 PDDL의 핵심 목표는 주어진 세계의 가능한 상태들, 수행 가능한 행동의 집합, 특정 초기 상태, 그리고 달성하고자 하는 목표 상태의 집합을 명확하게 기술하는 것이다.1 구문적으로 PDDL은 리스프(Lisp) 프로그래밍 언어에서 영감을 받아 괄호를 사용한 전위 표기법(prefix notation)을 채택하고 있다.4

PDDL이 등장하기 이전, AI 계획 연구 커뮤니티는 STRIPS, ADL과 같은 각기 다른 형식의 언어들을 사용하였다. 이러한 언어들은 서로 호환되지 않았기 때문에, 연구자들은 각기 다른 플래너(planner)의 성능을 객관적으로 비교하거나 문제 집합을 공유하는 데 큰 어려움을 겪었다.5 PDDL은 이러한 파편화된 환경에 대한 해결책으로 제시되었다. 즉, AI 계획 분야의 '공용어(lingua franca)' 역할을 수행함으로써, 연구자들이 동일한 문제 정의를 사용하여 자신의 플래너를 테스트하고 그 결과를 다른 연구자들의 것과 직접 비교할 수 있는 기반을 마련한 것이다.

이러한 표준화는 단순히 편의성을 높이는 차원을 넘어, AI 계획 연구 분야의 발전을 촉진하는 결정적인 촉매제가 되었다. PDDL을 통해 플래너 성능에 대한 객관적이고 경험적인 평가가 가능해졌으며, 이는 건전한 경쟁과 협력을 장려하는 연구 환경을 조성하는 데 기여했다.1 결과적으로 PDDL의 도입은 AI 계획을 개별적인 연구의 집합에서 벤치마킹과 정량적 진보 측정이 가능한 통일된 과학 분야로 전환시키는 데 핵심적인 역할을 수행했다.


PDDL의 탄생과 발전은 국제 계획 대회(International Planning Competition, IPC)와 불가분의 관계에 있다. PDDL은 1998년 드류 맥더못(Drew McDermott)과 그의 동료들에 의해 개발되었는데, 그 주된 목적은 1998년과 2000년에 개최된 IPC를 성공적으로 운영하기 위함이었다.1 IPC는 전 세계의 다양한 AI 플래너들이 정해진 문제들을 해결하며 성능을 겨루는 대회로, 모든 참가 플래너가 내부 알고리즘의 차이와 무관하게 공통된 입력 형식을 이해하고 처리할 수 있어야 했다. PDDL은 바로 이 공통 입력 언어의 역할을 수행하기 위해 탄생했다.7

이후 PDDL은 각각의 IPC가 개최될 때마다 대회의 요구사항을 반영하며 지속적으로 발전해왔다.1 초기 IPC가 고전적인 논리 기반의 문제에 집중했다면, 시간이 흐르면서 대회는 시간, 수치 자원, 선호도, 외부 사건 등 현실 세계의 복잡성을 더 많이 반영하는 문제들을 출제하기 시작했다. 이러한 대회의 진화는 PDDL의 진화를 직접적으로 견인했다. 예를 들어, 시간과 자원의 개념을 다루는 문제가 등장하면서 PDDL 2.1이 개발되었고, 최적화와 선호도를 다루기 위해 PDDL 3.0이 등장하는 등, IPC는 PDDL의 표현력을 확장시키는 가장 중요한 동력이었다.7 이처럼 PDDL의 역사는 IPC라는 구체적인 필요성에 의해 시작되었고, 대회의 발전에 발맞추어 그 기능과 표현력이 확장되어 온 역사라 할 수 있다.


PDDL의 가장 근본적이고 강력한 설계 철학은 계획 문제를 '도메인(domain)'과 '문제(problem)'라는 두 가지 독립적인 부분으로 명확하게 분리하는 것이다.1

- **도메인 기술(Domain Description):** 도메인 파일은 특정 문제 영역의 '물리학' 또는 '법칙'을 정의한다. 이는 해당 영역의 모든 문제에 보편적으로 적용되는 요소들을 포함한다. 예를 들어, 객체의 종류(types), 객체들이 가질 수 있는 속성이나 관계(predicates), 그리고 상태를 변화시킬 수 있는 행동(actions)과 그 행동의 결과(effects) 등이 여기에 해당한다.1 물류 도메인을 예로 들면, '트럭'과 '화물'이라는 객체 유형, '트럭이 특정 도시에 있다'는 상태, '트럭에 화물을 싣는다'는 행동 등은 어떤 특정 배송 시나리오와 무관하게 항상 존재하는 규칙이다.
- **문제 기술(Problem Description):** 문제 파일은 도메인에 정의된 규칙 내에서 해결해야 할 구체적인 시나리오, 즉 하나의 인스턴스(instance)를 정의한다. 여기에는 해당 문제에 등장하는 구체적인 객체들(objects), 계획이 시작되는 초기 상태(initial state), 그리고 계획을 통해 도달해야 하는 목표 상태(goal)가 포함된다.1 앞선 물류 도메인의 예에서, '트럭A와 트럭B', '화물1', '서울과 부산'이라는 구체적인 객체들과, '초기 상태에서 트럭A와 화물1은 서울에 있다', '목표 상태는 화물1이 부산에 있는 것이다'와 같은 내용이 문제 파일에 기술된다.

이러한 분리 구조는 객체 지향 프로그래밍(Object-Oriented Programming, OOP)에서 클래스(class)와 인스턴스(instance)의 관계와 유사하다.1 도메인은 재사용 가능한 청사진인 클래스에 해당하고, 문제 파일은 그 청사진을 바탕으로 생성된 구체적인 데이터 객체인 인스턴스에 해당한다.

이러한 설계 원칙의 중요성은 재사용성과 확장성에 있다. 잘 정의된 하나의 물류 도메인 파일만 있다면, 행동의 핵심 로직을 전혀 수정하지 않고도 수천, 수만 가지의 각기 다른 배송 문제(문제 파일)를 해결할 수 있다.1 이러한 모듈성은 PDDL이 학계를 넘어 산업 현장에서도 성공적으로 적용될 수 있었던 핵심적인 이유이다. 더 나아가, 이 분리 구조는 AI 계획 문제를 '모델링'과 '해결'이라는 두 단계로 자연스럽게 나누게 한다. 이는 먼저 세계의 규칙을 정의하는 지식 표현(knowledge representation) 과제를 수행하고, 그 다음 그 규칙 안에서 특정 도전 과제를 해결하는 방식으로 문제에 접근하도록 유도한다. 이는 현대 AI 연구에서 강건한 세계 모델을 구축하는 것이 핵심적인 과제로 부상하고 있는 것과 맥을 같이 한다. 최근 대규모 언어 모델(LLM)을 이용해 PDDL 모델 생성을 자동화하려는 연구가 활발히 진행되고 있는데 13, 이는 PDDL이 수십 년 전에 형식화했던 '모델링의 어려움'이라는 근본적인 문제가 오늘날 AI의 최전선에서도 여전히 중요한 과제임을 보여준다.


PDDL 모델은 앞서 설명한 설계 원칙에 따라 도메인 파일과 문제 파일이라는 두 개의 텍스트 파일로 구성된다. 이 두 파일이 결합하여 하나의 완전한 계획 과제를 형성하며, 이는 AI 플래너의 입력으로 사용된다.




도메인 파일은 계획 환경의 보편적인 측면, 즉 특정 시나리오에 관계없이 항상 참인 규칙들을 정의한다.9 이 파일은 다음과 같은 핵심적인 섹션들로 구성된다.

- `(define (domain <domain-name>))`: 모든 도메인 파일의 시작 부분으로, 해당 도메인의 고유한 이름을 선언한다.11
- `(:requirements...)`: 해당 도메인 모델이 사용하는 PDDL의 특정 기능들을 명시하는 부분이다. 예를 들어, `:strips`, `:typing`, `:adl` 등이 있으며, 이를 통해 플래너는 자신이 해당 도메인을 처리할 수 있는 능력을 갖추었는지 사전에 확인할 수 있다.2 대부분의 플래너는 PDDL의 전체 기능 중 일부만을 지원하기 때문에 이 요구사항 명시는 매우 중요하다.5
- `(:types...)`: 도메인에 존재하는 객체들의 계층 구조를 정의한다. OOP의 클래스 상속과 유사하게, 상위 타입을 정의하고 이를 상속받는 하위 타입을 정의할 수 있다 (예: `truck airplane - vehicle`, `vehicle - physobj`). 타이핑을 통해 행동의 매개변수 타입을 제한함으로써 계획 탐색 공간을 줄이고 모델의 명확성을 높일 수 있다.2
- `(:predicates...)`: 객체들의 속성이나 객체들 간의 관계를 나타내는 술어(predicate)를 선언한다. 술어는 참(true) 또는 거짓(false)의 값을 가지는 불리언(boolean) 명제이다. 예를 들어, `(at?t - truck?l - location)`는 '트럭 `?t`가 장소 `?l`에 있다'는 관계를 나타낸다. 물음표(`?`)는 변수를 의미한다.2
- `(:action...)`: 세계의 상태를 변화시키는 행위자(operator)를 정의하는 가장 핵심적인 부분이다. 각 행동은 다음 요소들을 포함한다.
  - **이름(Name):** 행동을 식별하는 고유한 이름.
  - **매개변수(Parameters):** 행동에 관여하는 객체 변수들과 그 타입을 정의한다 (예: `?t - truck`).
  - **전제조건(Precondition):** 해당 행동이 실행되기 위해 반드시 참이어야 하는 조건들의 논리식이다.
  - **효과(Effect):** 행동이 성공적으로 실행된 후 세계의 상태가 어떻게 변하는지를 기술한다. 즉, 어떤 술어가 참이 되고 어떤 술어가 거짓이 되는지를 명시한다.

도메인 파일은 본질적으로 인과 관계에 대한 선언적 모델(declarative model)이다. 이는 플래너에게 문제를 '어떻게' 해결해야 하는지를 알려주는 절차적 코드가 아니라, 이 세계에서 '무엇이 가능한지'만을 정의한다. 바로 이러한 도메인 독립적(domain-independent) 특성 덕분에, 단일 AI 플래ナー는 내부 코드를 변경하지 않고도 물류, 로보틱스, 게임 등 전혀 다른 영역의 문제들을 해결할 수 있다.1


문제 파일은 도메인 파일에 정의된 일반적인 규칙들을 바탕으로, 해결해야 할 특정 시나리오를 구체화한다.9 이 파일은 다음의 구성 요소들을 포함한다.

- `(define (problem <problem-name>))`: 문제의 고유한 이름을 선언한다.2
- `(:domain <domain-name>)`: 이 문제가 어떤 도메인에 속하는지를 명시하여 해당 도메인 파일과 연결한다.2
- `(:objects...)`: 이 문제에 등장하는 구체적인 객체들의 목록을 나열하고 각 객체의 타입을 지정한다. 예를 들어, `truck1 truck2 - truck`은 `truck1`과 `truck2`가 `truck` 타입의 객체임을 선언한다.5
- `(:init...)`: 계획이 시작되는 초기 상태를 기술한다. 초기 상태에서 참인 모든 구체화된(grounded) 술어들의 리스트를 나열한다. PDDL은 기본적으로 '닫힌 세계 가정(Closed World Assumption)'을 따르므로, `:init` 섹션에 명시적으로 언급되지 않은 모든 술어는 초기 상태에서 거짓(false)으로 간주된다.5
- `(:goal...)`: 계획이 성공적으로 완료되기 위해 최종 상태에서 반드시 만족되어야 하는 조건들을 논리식 형태로 기술한다.5

결론적으로, 문제 파일은 플래너의 탐색 알고리즘에 필요한 구체적인 데이터, 즉 출발점(`:init`)과 도착점(`:goal`)을 제공한다. 도메인 파일과 문제 파일의 조합은 AI 플래너가 해결할 수 있는 완전하고 명확한 하나의 계획 과제를 구성한다.1


PDDL의 구조를 구체적으로 이해하기 위해, AI 계획 분야의 고전적인 예제인 '블록 월드(Blocks World)'를 모델링하는 과정을 살펴본다. 이 세계에는 여러 개의 블록이 있고, 로봇 팔이 블록을 집거나 내려놓거나 다른 블록 위에 쌓을 수 있다.17


```
(define (domain blocksworld)
  (:requirements :strips :typing)
  (:types block)
  (:predicates 
    (on?x - block?y - block)      ; 블록 x가 블록 y 위에 있음
    (ontable?x - block)            ; 블록 x가 테이블 위에 있음
    (clear?x - block)              ; 블록 x 위에 아무것도 없음
    (handempty)                     ; 로봇 팔이 비어 있음
    (holding?x - block)            ; 로봇 팔이 블록 x를 들고 있음
  )

  (:action pick-up
    :parameters (?x - block)
    :precondition (and (clear?x) (ontable?x) (handempty))
    :effect (and (not (ontable?x)) (not (clear?x)) (not (handempty)) (holding?x))
  )

  (:action put-down
    :parameters (?x - block)
    :precondition (holding?x)
    :effect (and (not (holding?x)) (clear?x) (handempty) (ontable?x))
  )

  (:action stack
    :parameters (?x - block?y - block)
    :precondition (and (holding?x) (clear?y))
    :effect (and (not (holding?x)) (not (clear?y)) (handempty) (clear?x) (on?x?y))
  )

  (:action unstack
    :parameters (?x - block?y - block)
    :precondition (and (on?x?y) (clear?x) (handempty))
    :effect (and (not (on?x?y)) (not (clear?x)) (not (handempty)) (holding?x) (clear?y))
  )
)
```


```
(define (problem blocksworld-problem-1)
  (:domain blocksworld)
  (:objects a b c - block)
  (:init
    (ontable a)
    (ontable b)
    (on c a)
    (clear b)
    (clear c)
    (handempty)
  )
  (:goal (and (on a b) (on b c)))
)
```

이 예시는 PDDL의 핵심 원리를 명확하게 보여준다. 도메인 파일은 `pick-up`, `put-down`, `stack`, `unstack`이라는 네 가지 보편적인 행동을 정의한다. 문제 파일은 `a`, `b`, `c`라는 세 개의 구체적인 블록을 소개하고, 이들의 초기 배치 상태를 기술하며, 최종적으로 `c` 위에 `b`가, `b` 위에 `a`가 쌓인 탑을 만드는 것을 목표로 설정한다. AI 플래너는 이 두 파일을 입력받아 목표를 달성하기 위한 행동 순서(예: `unstack(c, a)`, `put-down(c)`, `pick-up(b)`, `stack(b, c)`, `pick-up(a)`, `stack(a, b)`)를 찾아낸다. 이처럼 구체적인 예제는 도메인과 문제의 추상적인 개념을 실체화하고, 더 복잡한 모델을 이해하는 데 필요한 기초를 제공한다.


PDDL은 단일한 언어가 아니라, AI 계획 커뮤니티의 요구사항 변화를 반영하며 지속적으로 진화해 온 언어들의 집합체이다. 각 버전은 이전 버전의 한계를 극복하고 더 현실적이고 복잡한 문제를 모델링할 수 있도록 새로운 기능들을 도입했다. 이러한 진화의 과정은 AI 계획 연구자들이 '현실 세계의 문제'를 어떻게 이해하고 정의해왔는지를 보여주는 연대기와 같다.


PDDL의 초기 버전인 PDDL 1.2는 최초의 IPC(1998, 2000)에서 사용되었으며, 이전의 대표적인 계획 형식 언어였던 STRIPS(Stanford Research Institute Problem Solver)와 ADL(Action Description Language)의 핵심 개념을 표준화했다.1

- **STRIPS:** 가장 기본적인 PDDL의 하위 집합으로, 매우 제한적인 표현력만을 가진다. STRIPS에서는 전제조건과 목표가 오직 긍정 논리 리터럴(positive literal)의 논리곱(conjunction)으로만 구성될 수 있다. 행동의 효과는 단순히 상태에 추가될 사실들의 목록(add list)과 삭제될 사실들의 목록(delete list)으로 표현된다.5
- **ADL:** PDDL 1.2는 `:adl` 요구사항을 통해 STRIPS의 한계를 크게 확장했다. ADL 수준의 표현력은 다음을 포함한다 5:
  - **부정 전제조건(Negative Preconditions):** `(not (predicate))`와 같이 어떤 사실이 거짓이어야 함을 조건으로 명시할 수 있다.
  - **선언적 전제조건(Disjunctive Preconditions):** `(or...)`를 사용하여 여러 조건 중 하나만 만족해도 됨을 표현할 수 있다.
  - **정량화 공식(Quantified Formulas):** `(forall...)` (모든 객체에 대해) 또는 `(exists...)` (어떤 객체에 대해)를 사용하여 더 일반적인 조건을 기술할 수 있다.
  - **조건부 효과(Conditional Effects):** `(when...)` 구문을 사용하여 특정 조건이 만족될 때만 효과가 발생하도록 할 수 있다.

PDDL 1.2는 결정론적(deterministic)이고, 완전 관찰 가능(fully observable)하며, 정적(static)인 세계에서 행동이 즉각적으로(instantaneous) 일어나는 '고전 계획(classical planning)'의 토대를 마련했다. 현대의 많은 플래너들이 더 발전된 기능을 지원함에도 불구하고, 여전히 대다수의 플래너들은 이 STRIPS/ADL 하위 집합을 핵심으로 작동한다.5 이는 고전 계획 모델이 여전히 많은 문제의 핵심을 포착하는 데 유용하며, 계산적으로 다루기 용이하기 때문이다.


2002년 IPC-3를 위해 도입된 PDDL 2.1은 표현력 측면에서 기념비적인 도약을 이루었다.10 이 버전은 이전까지의 순수 논리적 세계관에서 벗어나, 현실 세계의 두 가지 중요한 차원인 시간과 자원을 모델링에 통합했다.

- **수치 플루언트(Numeric Fluents):** `:functions` 키워드를 통해 도입된 수치 플루언트는 연료, 에너지, 거리, 비용 등 연속적인 양을 모델링할 수 있게 했다. 이 변수들은 전제조건에서 비교 연산에 사용될 수 있으며(예: `(> (fuel-level) 10)`), 행동의 효과를 통해 그 값이 증가하거나 감소할 수 있다(예: `(decrease (fuel-level) 10)`).9 이를 통해 자원 소비와 같은 개념을 자연스럽게 표현할 수 있게 되었다.
- **지속 행동(Durative Actions):** `:durative-action`은 행동이 즉각적으로 끝나는 것이 아니라 일정 시간 동안 지속된다는 개념을 도입했다. 이로써 행동의 시작, 끝, 그리고 지속되는 동안에 각각 다른 전제조건과 효과를 명시할 수 있게 되었다. 예를 들어, 비행기 이륙 행동은 '시작 시점(`at start`)'에 활주로가 비어 있어야 하고, '끝나는 시점(`at end`)'에 목적지 공항의 활주로가 비어 있어야 하며, '지속되는 동안(`over all`)'에는 연료가 충분해야 한다는 복합적인 시간적 제약을 모델링할 수 있다.9

PDDL 2.1의 등장은 AI 계획의 범위를 순수 논리 퍼즐에서 벗어나, 시간과 자원이 핵심 제약 조건인 스케줄링, 자원 관리 등 훨씬 더 광범위한 현실 문제로 확장시켰다.22 하지만 이러한 표현력의 증가는 계획 문제의 계산 복잡성을 기하급수적으로 증가시키는 대가를 치렀다.26


PDDL 2.2는 PDDL 2.1을 기반으로 한 점진적인 개선 버전으로, 모델링의 간결성과 표현력을 더욱 향상시키는 두 가지 주요 기능을 추가했다.

- **파생 술어(Derived Predicates):** `:derived-predicates` 요구사항을 통해 도입된 이 기능은, 다른 술어들의 논리적 조합으로 정의되는 새로운 술어를 만들 수 있게 한다. 예를 들어, `(train-usable?t)`라는 파생 술어를 `(and (has-driver?t) (has-guard?t))`로 정의할 수 있다. 이는 복잡한 논리식을 반복적으로 작성할 필요 없이, 재사용 가능한 논리적 매크로처럼 사용하여 도메인 정의를 더 간결하고 이해하기 쉽게 만든다.9
- **시간 제약 초기 리터럴(Timed Initial Literals, TILs):** `:timed-initial-literals` 요구사항으로 활성화되는 이 기능은, 특정 사실이 계획 시작 시점이 아닌 미래의 특정 시간에 참이 되도록 명시할 수 있게 한다. 예를 들어, `(at 10.0 (market-open))`은 계획 시간 10.0에 '시장이 열린다'는 외부 사건이 발생함을 의미한다. 플래너는 이러한 외부 사건에 맞춰 자신의 행동을 계획하거나, 특정 사건이 발생할 때까지 기다려야 한다.9

이 두 기능은 모델링의 우아함과 효율성을 높였다. 파생 술어는 행동 전제조건의 중복을 줄여주고, TILs는 플래너가 수동적으로 반응해야 하는 외부의 스케줄된 이벤트를 모델링하는 강력한 도구를 제공했다.


PDDL 3.0은 계획의 패러다임을 '가능한 계획을 찾는 것(satisfiability)'에서 '최상의 계획을 찾는 것(optimization)'으로 전환시켰다. 이를 위해 '소프트 제약(soft constraints)'과 복잡한 계획 품질 지표의 개념을 도입했다.9

- **선호도(Preferences):** `:preferences` 요구사항을 통해, 계획의 유효성에 필수적이지는 않지만 만족되면 더 좋은 것으로 간주되는 '소프트 목표'를 정의할 수 있게 되었다. 선호도를 위반하는 계획은 유효하지만, 특정 비용(cost)을 수반하게 된다.27
- **제약(Constraints):** `:constraints`는 계획이 실행되는 전 과정, 즉 상태 궤적(state-trajectory)에 걸쳐 만족되어야 하는 조건을 명시한다. 예를 들어, `(always (safe-mode-on))`는 계획의 모든 상태에서 안전 모드가 켜져 있어야 함을, `(sometime-after (pickup-package) (deliver-package))`는 화물을 픽업한 후에만 배송이 이루어져야 한다는 시간적 순서를 강제한다.9
- **지표(Metric):** 계획의 품질을 평가하는 `:metric` 함수가 확장되었다. `(is-violated <preference-name>)`이라는 특수 함수를 사용하여 특정 선호도를 위반한 횟수나 여부를 비용 계산에 포함시킬 수 있게 되었다. 이를 통해 계획의 총 소요 시간, 자원 사용량, 그리고 다양한 선호도 만족도를 가중치를 두어 조합하는 복잡한 다중 목표 최적화 함수를 정의할 수 있다.27

PDDL 3.0을 통해 사용자는 단순히 목표를 달성하는 계획을 넘어, 다양한 질적 기준에 따라 가장 바람직한 계획을 찾도록 플래너에게 요구할 수 있게 되었다. 이는 현실 세계의 의사결정 과정이 단일 목표 달성이 아닌, 여러 상충하는 목표들 사이의 균형을 찾는 과정이라는 점을 반영한 중요한 진화였다.28


PDDL+는 이산적(discrete) 상태 변화와 연속적(continuous) 동역학이 혼합된 하이브리드 시스템(hybrid system)을 모델링하기 위해 설계되었다.32 이 버전은 플래너가 직접 제어할 수 없는 외부 세계의 변화를 포착하는 두 가지 핵심적인 개념을 도입했다.

- **프로세스(Processes):** `:process`는 전제조건이 만족되는 동안 지속적으로 상태에 영향을 미치는, 제어 불가능한 효과를 정의한다. 가장 대표적인 예는 중력이다. 로봇이 공을 들고 있지 않은 동안(`(not (held?ball))`), 중력 프로세스가 활성화되어 공의 속도를 시간에 따라 지속적으로 증가시킨다. 프로세스는 보통 `#t`라는 특수 시간 변수를 사용하여 수치 플루언트의 연속적인 변화를 모델링한다.1
- **이벤트(Events):** `:event`는 전제조건이 만족되는 순간 즉각적으로 발생하는, 제어 불가능한 행동을 정의한다. 예를 들어, 앞서 중력의 영향을 받던 공의 높이가 0 이하가 되는 순간, '바닥 충돌' 이벤트가 발생하여 공의 속도 방향을 반대로 바꾸는 효과를 일으킬 수 있다.1

PDDL+는 에이전트의 행동 외에도 외부 환경 자체의 변화(외생적 변화, exogenous change)가 발생하는 동적인 세계를 모델링할 수 있게 해준다. 이는 에이전트가 정적인 세계에 단순히 행동을 가하는 것이 아니라, 끊임없이 변하는 환경에 반응하고 적응해야 하는 현실적인 로보틱스, 물리 기반 게임, 우주 탐사 등의 도메인을 모델링하는 데 필수적이다.32 PDDL+는 PDDL 표현력의 최전선에 있지만, 그 극심한 복잡성으로 인해 이를 완벽하게 지원하는 플래너는 극소수에 불과하다.32

이러한 PDDL의 연대기적 진화는 AI 계획 커뮤니티가 현실 세계를 모델링하기 위해 점진적으로 단순화 가정을 완화해 온 과정을 보여준다. 논리(1.2)에서 시간/자원(2.1)으로, 다시 선호도(3.0)와 제어 불가능한 동역학(PDDL+)으로 나아가는 이 궤적은, 언어의 표현력을 높이는 동시에 계획 문제의 계산적 난이도를 급격히 증가시키는 근본적인 트레이드오프 관계를 내포하고 있다.


PDDL의 발전 과정은 새로운 기능의 점진적인 추가로 요약될 수 있다. 각 버전은 이전 버전의 기능을 포함하면서 표현력을 확장하는 방식으로 설계되었다. 이 섹션에서는 주요 버전별 핵심 기능과 그에 따른 구문 및 의미론적 변화를 체계적으로 비교 분석한다.


다음 표는 PDDL의 주요 버전별로 도입된 핵심 기능들을 요약하여 보여준다. 이 표는 특정 계획 문제를 모델링할 때 어떤 버전의 PDDL을 사용해야 할지 결정하는 데 유용한 참조 자료가 될 수 있다.

| 기능 (Feature)                                               | PDDL 1.2 | PDDL 2.1 | PDDL 2.2 | PDDL 3.0 | PDDL+ |
| ------------------------------------------------------------ | -------- | -------- | -------- | -------- | ----- |
| **기본 STRIPS** (Positive Preconditions/Effects)             | ✔️        | ✔️        | ✔️        | ✔️        | ✔️     |
| **타이핑 (Typing)**                                          | ✔️        | ✔️        | ✔️        | ✔️        | ✔️     |
| **ADL 기능** (Negation, Disjunction, Quantifiers, Conditional Effects) | ✔️        | ✔️        | ✔️        | ✔️        | ✔️     |
| **수치 플루언트 (Numeric Fluents)**                          | ❌        | ✔️        | ✔️        | ✔️        | ✔️     |
| **지속 행동 (Durative Actions)**                             | ❌        | ✔️        | ✔️        | ✔️        | ✔️     |
| **파생 술어 (Derived Predicates)**                           | ❌        | ❌        | ✔️        | ✔️        | ✔️     |
| **시간 제약 초기 리터럴 (Timed Initial Literals)**           | ❌        | ❌        | ✔️        | ✔️        | ✔️     |
| **제약 (Constraints)** (State-Trajectory)                    | ❌        | ❌        | ❌        | ✔️        | ✔️     |
| **선호도 (Preferences)** (Soft Goals)                        | ❌        | ❌        | ❌        | ✔️        | ✔️     |
| **프로세스 (Processes)** (Uncontrollable, Continuous)        | ❌        | ❌        | ❌        | ❌        | ✔️     |
| **이벤트 (Events)** (Uncontrollable, Instantaneous)          | ❌        | ❌        | ❌        | ❌        | ✔️     |

*표 1: PDDL 버전별 주요 기능 도입 매트릭스*


PDDL의 버전이 올라가면서 단순히 새로운 키워드가 추가되는 것을 넘어, '계획(plan)'의 근본적인 의미 자체가 변화했다.

- **PDDL 1.2 (고전적 계획):** 이 버전에서 계획은 **즉각적인 행동들의 순차적인 나열(sequence of actions)**이다. 시간의 개념이 없으며, 한 행동이 끝나면 즉시 다음 상태로 전이되고, 그 상태에서 다음 행동이 이어진다. 계획의 유효성은 초기 상태에서 시작하여 행동 순서를 따랐을 때 최종적으로 목표 상태에 도달하는지로 판단된다.
- **PDDL 2.1 (시간적 계획):** 지속 행동의 도입으로 계획은 더 이상 단순한 순서가 아니라, **시간 축 위에 배치된 행동들의 스케줄(schedule)**이 된다. 각 행동은 시작 시간과 종료 시간을 가지며, 여러 행동이 동시에 실행될 수 있다. 계획의 유효성은 모든 행동의 시간적 제약조건(시작/종료/지속 조건)이 만족되고, 자원 제약(수치 플루언트)이 위반되지 않으며, 최종적으로 목표를 달성하는지로 판단된다. 이로 인해 계획 문제는 순서 찾기 문제에서 복잡한 시간 및 자원 제약 만족 문제로 확장되었다.
- **PDDL 3.0 (최적화 계획):** 선호도와 제약의 도입은 계획의 개념을 **유틸리티(utility)를 최대화하는 스케줄**로 발전시켰다. 이제 플래너는 단순히 유효한 계획을 찾는 것을 넘어, 수많은 유효한 계획들 중에서 주어진 지표(`:metric`)에 따라 가장 '좋은' 계획을 찾아야 한다. 계획의 품질은 소요 시간, 자원 소비뿐만 아니라, 얼마나 많은 선호도를 만족시키고 제약을 준수했는지에 따라 결정된다.
- **PDDL+ (반응적 계획):** 프로세스와 이벤트의 도입은 계획을 고정된 행동 스케줄이 아닌, **외부 환경 변화에 대응하는 정책(policy)**의 개념으로까지 확장시킨다. 플래너가 생성한 계획은 제어 불가능한 외부 사건(이벤트)이나 지속적인 환경 변화(프로세스)가 발생할 것을 예측하고, 그에 맞춰 행동을 수행해야 한다. 계획의 유효성은 에이전트의 행동과 외부 환경의 동역학이 상호작용하는 시뮬레이션 과정에서 목표를 달성할 수 있는지에 따라 결정된다.

이처럼 PDDL의 진화는 계획의 의미를 단순한 행동 순서에서 복잡한 시간적 스케줄로, 그리고 더 나아가 동적인 환경에 대응하는 최적의 정책으로 점차 확장시켜왔다. 이는 AI 계획이 추상적인 논리 문제에서 벗어나 현실 세계의 복잡성과 불확실성을 포용하려는 지속적인 노력을 반영한다.


PDDL의 강력한 표현력과 표준화된 형식은 이론적 연구를 넘어 다양한 실제 응용 분야에서 복잡한 의사결정 문제를 해결하는 데 사용되고 있다. 각 분야는 PDDL의 특정 기능들을 활용하여 고유의 문제들을 모델링하고 해결책을 모색한다.


로보틱스 분야에서 PDDL은 로봇의 작업 계획 및 실행을 위한 핵심 도구로 자리 잡았다.3 로봇이 수행해야 할 복잡한 임무(예: 부품 조립, 물체 조작, 자율 주행)를 관리 가능한 하위 작업들로 분해하고, 목표 달성을 위한 최적의 행동 순서를 추론하는 데 PDDL이 사용된다.3 예를 들어, PR2 로봇이 물건을 가져오거나 문을 여는 작업을 계획하고, NASA의 로보넛(Robonaut)이 우주 정거장에서 정교한 조작 작업을 수행하는 데 PDDL 기반 플래너가 활용되었다.3

**모델링 기법:** 로보틱스 도메인에서는 로봇 팔의 상태(예: `(handempty)`), 물체의 위치(`(at?obj?loc)`), 로봇이 쥔 물체(`(holding?obj)`) 등을 술어로 표현한다. `pick-up`, `put-down`, `move`와 같은 행동들은 로봇 팔의 상태와 물체의 위치를 확인하는 전제조건과, 이 상태들을 변경하는 효과로 정의된다.3 특히 로봇의 물리적 움직임이나 배터리 소모와 같은 연속적인 변화를 모델링하기 위해 PDDL+의 프로세스와 이벤트 기능이 매우 유용하게 사용될 수 있다.36 생성된 PDDL 계획은 ROS(Robot Operating System)와 같은 로보틱스 프레임워크와 통합되어 실제 로봇의 모터 제어 명령으로 변환되어 실행된다.3

**사례 연구: 로봇 팔의 블록 이동:** 방 사이를 오가며 블록을 옮기는 로봇 팔을 모델링하는 경우를 생각할 수 있다. 도메인 파일에는 `move`, `pick-up`, `drop` 행동이 정의된다. `pick-up` 행동의 전제조건은 '로봇이 블록과 같은 방에 있고', '로봇 팔이 비어 있으며', '블록 위에 아무것도 없는 것'이다. 효과는 '로봇 팔이 블록을 쥐게 되고', '블록은 더 이상 원래 위치에 있지 않으며', '로봇 팔은 더 이상 비어있지 않게 되는 것'이다. 문제 파일은 특정 방들의 배치, 블록들의 초기 위치, 그리고 목표 위치를 정의한다. 플래너는 이 정보를 바탕으로 모든 블록을 목표 위치로 옮기기 위한 최단 행동 순서를 계산한다.11


물류 및 공급망 관리 분야는 PDDL이 가장 성공적으로 적용된 영역 중 하나이다. PDDL은 배송 차량의 경로를 최적화하고, 운송 스케줄을 수립하며, 재고를 관리하는 등 복잡한 물류 문제를 해결하는 데 널리 사용된다.41 PDDL 기반 플래너를 통해 생성된 효율적인 계획은 운송 비용을 절감하고 배송 시간을 단축하는 데 직접적으로 기여한다.42

**모델링 기법:** 전형적인 물류 도메인은 `truck`, `airplane`, `package`, `location`, `city`와 같은 타입을 정의한다. `(at?vehicle?loc)` (차량이 특정 위치에 있음), `(in?package?vehicle)` (화물이 특정 차량 안에 있음)과 같은 술어들이 상태를 표현한다. `load`, `unload`, `drive`, `fly`와 같은 행동들은 차량과 화물의 위치를 변경하는 역할을 한다. 예를 들어, `drive` 행동은 같은 도시 내의 두 장소 사이를 이동하게 하며, `fly` 행동은 공항 사이를 이동하게 한다.2 특히 PDDL 3.0의 선호도 기능은 '특정 경로를 선호'하거나 '정해진 시간 내 배송'과 같은 소프트 제약을 모델링하는 데 매우 유용하다. 이를 통해 단순히 가능한 경로가 아닌, 비즈니스 요구사항을 만족하는 최적의 경로를 찾을 수 있다.27

**사례 연구: 다중 운송 수단을 이용한 화물 배송:** 여러 도시에 흩어져 있는 화물들을 각기 다른 목적지로 배송하는 고전적인 물류 문제를 PDDL로 모델링할 수 있다. 도시 내에서는 트럭을 이용해 화물을 운송하고, 도시 간에는 비행기를 이용해 공항으로 화물을 운송한다. `load-truck` 행동은 트럭과 화물이 같은 장소에 있어야 한다는 전제조건을 가지며, `fly-airplane` 행동은 비행기가 공항에 있어야 한다는 전제조건을 가진다. 플래너는 이러한 제약 조건들을 모두 고려하여, 모든 화물을 최종 목적지까지 가장 효율적으로 운송하기 위한 트럭 운행과 항공편 이용의 조합으로 이루어진 전체 계획을 수립한다.2


인더스트리 4.0 시대의 스마트 팩토리에서 PDDL은 자동화된 제조 공정 계획 및 최적화에 활용된다. 복잡한 조립 라인의 작업 순서를 결정하거나, 여러 로봇과 장비의 협업을 스케줄링하는 데 사용될 수 있다.46 특히 디지털 트윈(Digital Twin) 기술과 결합하여, PDDL 플래너가 생성한 최적의 작업 계획을 가상 환경의 디지털 트윈에서 먼저 시뮬레이션하고 검증한 후, 실제 물리적 시스템에 적용하는 방식이 연구되고 있다.48 하지만 제조 공정은 매우 복잡하여 PDDL 모델을 수동으로 작성하는 것이 큰 병목으로 작용하며, 이 때문에 SysML이나 Semantic Web Services 같은 다른 산업 표준 형식에서 PDDL 모델을 자동으로 생성하는 연구가 활발히 진행 중이다.12

**모델링 기법:** 제조 도메인은 `robot`, `conveyor`, `unit`(작업 스테이션), `piece`(부품), `tray`(부품 운반대) 등의 타입을 가진다. 행동은 `pickup_tray_on_unit`(유닛에서 트레이 픽업), `drop_tray_on_conveyor`(컨베이어에 트레이 드롭) 등 구체적인 제조 단계를 나타낸다.2 문제는 종종 각 트레이에 수행해야 할 작업 목록(스택)이 주어지고, 로봇이 트레이를 적절한 유닛으로 운반하여 모든 작업을 완료하는 생산자-소비자 문제로 형식화된다.2

**사례 연구: 로봇 기반 제조 셀:** 여러 로봇이 컨베이어 벨트와 가공 유닛 사이에서 부품이 담긴 트레이를 운반하는 제조 셀을 모델링한다. 각 트레이에는 수행해야 할 가공 작업(예: 드릴링, 폴리싱)의 순서가 정의되어 있다. 로봇은 트레이를 집어 해당 작업을 수행할 수 있는 유닛으로 가져가야 한다. 플래너의 목표는 모든 트레이의 모든 작업을 완료하는 가장 빠른 로봇들의 행동 및 이동 순서를 찾는 것이다.2


게임 개발 분야에서 PDDL은 NPC(Non-Player Character)와 같은 AI 에이전트의 지능적인 행동을 계획하는 데 사용될 수 있다. 에이전트는 PDDL을 사용하여 게임 환경 내에서 전략을 수립하고, 퍼즐을 풀며, 플레이어나 다른 객체와 지능적으로 상호작용할 수 있다.52 더 중요한 응용 분야는 자동화된 게임 테스팅이다. 플래너를 사용하여 가능한 모든 플레이 시나리오를 탐색하는 테스트 스크립트를 자동으로 생성하고, 게임을 더 이상 진행할 수 없는 막다른 상태(dead-end state)를 찾아내거나, 의도하지 않은 지름길(shortcut)이 없는지 검증할 수 있다.53 그러나 게임의 복잡한 규칙을 PDDL로 모델링하는 '저작 부담(authorial burden)'이 커서, 실제 게임 플레이 로그를 분석하여 PDDL 모델을 자동으로 학습하는 워크플로우가 제안되고 있다.53

**모델링 기법:** 게임 세계의 상태는 `(location hero tile23)`, `(has-key hero)`와 같은 술어로 표현된다. 플레이어가 할 수 있는 행동(예: 이동, 아이템 줍기, 문 열기)은 PDDL 행동으로 모델링된다. 특히 물리 엔진이 중요한 게임에서는 PDDL+를 사용하여 연속적인 동역학을 모델링하는 것이 효과적이다.33

**사례 연구: 아케이드 게임 AI:** 고전 아케이드 게임인 '아이스블록스(Iceblox)'에 PDDL 플래너를 연결한 사례가 있다. 게임의 목표는 펭귄 캐릭터를 조종하여 모든 동전을 모으는 것이다. 플래너는 '교차로 A에서 B로 이동', '얼음 블록 밀기'와 같은 상위 수준의 계획을 생성한다. 별도의 계획 실행 모듈은 이 상위 수준의 계획을 실제 게임에서 필요한 저수준의 키 입력(예: 위, 아래, 왼쪽, 오른쪽)으로 변환하여 실행한다. 이를 통해 AI는 장애물과 적을 피해 레벨을 해결하는 논리적 추론을 수행할 수 있다.55

이러한 다양한 응용 사례들은 PDDL의 공통적인 가치를 보여준다. PDDL은 단순히 '계획'을 찾는 도구를 넘어, 시스템의 의사결정 논리를 형식적이고, 검증 가능하며, 설명 가능한 모델로 제공한다. 특히 로보틱스나 제조업과 같이 안전이 중요한 분야에서, 결과물의 행동 과정을 단계별 인과관계로 설명할 수 있다는 점은 종단간(end-to-end) 딥러닝과 같은 블랙박스 모델에 비해 PDDL이 갖는 중요한 장점이다. 계획 자체가 목표 달성 과정에 대한 명확한 설명이 되기 때문이다.


PDDL이 이론적 언어를 넘어 실용적인 도구로 자리매김할 수 있었던 배경에는 이를 지원하는 풍부한 소프트웨어 생태계가 있다. 이 생태계는 PDDL 모델을 해석하고 계획을 생성하는 플래너부터, 개발 과정을 돕는 편집기 및 검증 도구에 이르기까지 다양한 구성 요소들을 포함한다.

- **플래너(Planners):** 플래너는 PDDL 도메인 및 문제 파일을 입력으로 받아, 초기 상태에서 목표 상태로 이르는 행동 순서, 즉 계획을 출력하는 핵심 소프트웨어이다.1 수많은 플래너가 개발되었으며, 각기 다른 알고리즘과 지원하는 PDDL 기능의 범위에 따라 특징이 나뉜다. 대표적인 고전적 플래너로는 FF(Fast Forward), FD(Fast Downward), HSP 등이 있으며, 특정 목적을 위해 설계된 다양한 플래너와 툴킷도 존재한다. 예를 들어, PDDL4J는 자바(Java) 기반의 오픈소스 툴킷으로, 다양한 최신 플래너를 제공하고 새로운 플래너 연구를 촉진하는 것을 목표로 한다.2 Pyperplan은 파이썬(Python)으로 구현된 플래너로, 교육 및 연구 목적으로 널리 사용된다.43
- **검증기(Validators):** 계획 검증기는 주어진 계획이 특정 PDDL 문제에 대한 올바른 해답인지를 자동으로 확인하는 도구이다. 즉, 계획의 각 행동이 전제조건을 만족하며 실행되는지, 그리고 최종적으로 목표 상태에 도달하는지를 검사한다. 이 분야의 표준 도구는 VAL(Validator)이다. VAL은 PDDL의 거의 모든 버전을 지원하며, 계획의 유효성을 빠르고 정확하게 검증하는 데 사용된다.15 검증기는 플래너 개발 과정에서 버그를 찾거나, 복잡한 계획의 정당성을 보장하는 데 필수적인 역할을 한다.
- **통합 개발 환경(IDE) 지원:** 현대적인 PDDL 개발은 전문화된 IDE 확장 기능에 의해 크게 도움을 받는다. 특히 Visual Studio Code(VS Code)를 위한 PDDL 확장 기능은 PDDL을 C#이나 Python과 같은 일급 언어 수준으로 지원한다.16 이 확장 기능은 다음과 같은 강력한 기능들을 제공한다:
  - **구문 강조(Syntax Highlighting):** PDDL 코드의 가독성을 높인다.
  - **자동 완성(Auto-completion):** 도메인, 문제, 행동 등의 템플릿을 자동으로 생성하여 작성 시간을 단축한다.
  - **오류 검사 및 정보 제공:** 술어나 타입 위에 마우스를 올리면 정의를 보여주는 등 실시간 피드백을 제공한다.
  - **플래너 연동 및 계획 시각화:** 편집기 내에서 직접 플래너를 실행하고, 생성된 계획을 시각적으로 확인하여 디버깅을 용이하게 한다.
- **라이브러리 및 프로그래밍 인터페이스:** PDDL.jl과 같은 라이브러리는 줄리아(Julia)와 같은 최신 프로그래밍 언어 환경에서 PDDL을 직접 다룰 수 있는 인터페이스를 제공한다.17 이러한 라이브러리를 사용하면, PDDL 파일을 단순히 플래너의 입력으로 사용하는 것을 넘어, 프로그램 내에서 PDDL 도메인을 동적으로 로드하고, 상태를 조작하며, 행동을 실행하는 등 PDDL을 더 큰 소프트웨어 시스템의 일부로 통합할 수 있다. 이는 PDDL을 이용한 시뮬레이션, 강화 학습 환경 구축 등 더 복잡하고 통합적인 응용을 가능하게 한다.

이처럼 플래너, 검증기, IDE, 라이브러리로 구성된 풍부한 생태계는 PDDL의 진입 장벽을 낮추고 개발 생산성을 향상시킨다. 이러한 도구들이 없었다면 PDDL은 학술적인 형식 언어에 머물렀을 것이다. 이 생태계 덕분에 PDDL은 연구실을 넘어 실제 산업 현장에서 복잡한 문제를 해결하는 실용적인 기술로 발전할 수 있었다.


PDDL은 지난 수십 년간 AI 계획 분야의 발전을 이끌어왔지만, 여전히 해결해야 할 근본적인 도전 과제들을 안고 있다. 이러한 과제들은 PDDL의 미래 연구 방향을 제시하며, AI 기술의 전반적인 발전과 맞물려 새로운 가능성을 열고 있다.


PDDL의 가장 큰 실용적 장벽은 '지식 공학 병목 현상(Knowledge Engineering Bottleneck)'으로 알려져 있다. 이는 현실 세계의 복잡한 문제를 정확하고 효율적인 PDDL 도메인 모델로 변환하는 과정이 막대한 시간과 전문 지식을 요구하는 수작업에 의존한다는 문제이다.12 PDDL 모델링은 종종 명확한 방법론이 없는 '블랙 아트(black art)'로 묘사될 정도로, 도메인 전문가의 직관과 경험에 크게 의존한다.54

이 병목 현상은 PDDL의 광범위한 산업적 채택을 가로막는 핵심적인 요인이다. 아무리 빠른 플래너가 존재하더라도, 플래너가 이해할 수 있는 형식으로 문제를 모델링하는 과정 자체가 너무 어렵고 비용이 많이 든다면 그 효용성은 제한될 수밖에 없다. 따라서 응용 AI 계획 분야에서 가장 영향력 있는 미래 연구는 단순히 더 빠른 플래너를 개발하는 것을 넘어, PDDL 모델 획득 과정을 자동화하여 인간 전문가와 형식적 플래너 사이의 간극을 메우는 데 집중될 것이다.


최근 이 지식 공학 병목 현상을 해결할 가장 유망한 기술로 대규모 언어 모델(Large Language Model, LLM)이 부상하고 있다. 연구자들은 인간의 자연어(natural language)로 기술된 문제 설명을 LLM을 이용해 PDDL 도메인 및 문제 파일로 자동 번역하는 접근법을 활발히 연구하고 있다.13

이러한 하이브리드 접근법은 LLM의 유연한 자연어 이해 능력과 고전적 플래너의 형식적 정확성 및 계획의 정당성 보장이라는 두 가지 장점을 결합한다.13 초기 연구들은 LLM이 생성한 PDDL 코드의 오류를 인간 전문가가 수정하는 방식에 의존했지만, 최근에는 실제 환경과의 상호작용을 통해 얻은 피드백을 바탕으로 LLM이 스스로 PDDL 모델을 반복적으로 개선하는, 인간의 개입이 없는 자동화된 정제 프로세스가 제안되고 있다.13

LLM 기반 PDDL 생성 기술이 성공적으로 정착된다면, 이는 AI 계획의 민주화를 가져올 수 있다. 형식적 모델링에 대한 전문 지식이 없는 현장 전문가들도 자연어를 통해 자신들의 문제를 정의하고 AI 플래너의 도움을 받을 수 있게 될 것이다. 이는 모델링의 패러다임을 '인간이 모델러'인 현재에서 '인간이 감독관'인 미래로 전환시키는 중요한 변화를 의미한다.


PDDL의 역사에서 지속적으로 관찰되는 근본적인 딜레마는 **표현력(expressiveness)**과 **계산적 처리 가능성(tractability)** 사이의 상충 관계이다. PDDL 버전이 진화하며 현실 세계를 더 정확하게 묘사하기 위해 새로운 기능(예: 시간, 자원, 연속적 변화)을 추가할수록, 계획을 찾는 문제의 계산 복잡성은 기하급수적으로 증가한다.

- 기본적인 STRIPS 수준의 계획 문제의 복잡도는 **PSPACE-완전(PSPACE-complete)**이다.
- PDDL 2.1과 같이 조밀한 시간(dense time)을 다루는 시간적 계획은 **EXPSPACE-완전(EXPSPACE-complete)**으로 훨씬 더 어렵다.26
- PDDL+가 다루는 하이브리드 시스템에서의 계획 문제는 심지어 **결정 불가능(undecidable)**하다. 즉, 어떤 경우에는 해답을 찾는 알고리즘 자체가 존재하지 않을 수 있다.32

이것이 바로 AI 계획의 근본적인 트레이드오프이다. 더 현실적인 모델(표현력이 높은 모델)은 본질적으로 해결하기가 더 어렵다. 이 사실은 왜 많은 실용적인 플래너들이 여전히 STRIPS와 같은 단순한 PDDL 하위 집합만을 지원하는지를 설명해준다.5 또한 PDDL+를 지원하는 플래너가 극히 드물고 고도로 전문화된 이유이기도 하다.32 따라서 어떤 버전의 PDDL을 사용할 것인가의 선택은 '무엇을 모델링할 수 있는가'의 문제일 뿐만 아니라, '주어진 시간과 자원 내에서 현실적으로 해결할 수 있는가'의 문제이기도 하다. AI 계획의 미래는 모든 것을 표현할 수 있는 단일 언어가 아니라, 문제의 본질을 포착하는 가장 단순하고 효율적인 형식 언어를 선택하여 사용하는 도구 포트폴리오의 형태가 될 가능성이 높다.


계획 영역 정의 언어(PDDL)는 지난 수십 년간 인공지능 계획 연구 분야를 정의하고 발전시키는 데 중추적인 역할을 수행했다. 다양한 형식 언어들로 파편화되어 있던 분야에 표준을 제시함으로써, PDDL은 플래너 간의 객관적인 성능 비교를 가능하게 했고, 이는 국제 계획 대회(IPC)를 중심으로 한 경험적 연구의 기틀을 마련했다. PDDL의 가장 핵심적인 철학인 '도메인'과 '문제'의 분리는 모델의 재사용성과 확장성을 극대화하는 강력한 설계 원칙으로, 이는 AI 계획을 선언적인 지식 표현 문제로 접근하게 하는 패러다임을 확립했다.

PDDL은 정적인 논리 세계를 다루는 PDDL 1.2에서 시작하여 시간과 자원을 포함하는 PDDL 2.1, 선호도와 최적화를 다루는 PDDL 3.0, 그리고 제어 불가능한 연속적 변화까지 모델링하는 PDDL+에 이르기까지, 현실 세계의 복잡성을 포용하기 위해 끊임없이 진화해왔다. 이러한 진화의 역사는 표현력과 계산 복잡성 사이의 근본적인 상충 관계를 명확히 보여주며, AI 계획 연구의 핵심적인 도전 과제가 무엇인지를 시사한다.

딥러닝과 같은 다른 AI 패러다임이 부상하고 있음에도 불구하고, PDDL의 가치는 여전히 확고하다. 형식적 검증 가능성, 계획 과정의 설명 가능성, 그리고 명시적인 인과 관계 모델링이라는 PDDL의 특징은 특히 로보틱스, 첨단 제조, 물류와 같이 안전성과 신뢰성이 무엇보다 중요한 고위험(high-stakes) 도메인에서 대체 불가능한 강점을 제공한다.

PDDL의 미래는 지식 공학 병목 현상의 극복에 달려 있다. 대규모 언어 모델(LLM)을 활용한 자동 모델 생성 기술의 발전은 이 오랜 난제를 해결하고, 비전문가도 AI 계획 기술의 혜택을 누릴 수 있도록 하는 새로운 지평을 열고 있다. 결국 PDDL의 지속적인 발전과 그 생태계의 성숙은, 복잡하고 동적인 세계에서 자율적으로 추론하고 행동하는 차세대 지능형 시스템을 구현하는 데 있어 핵심적인 역할을 계속해서 수행할 것이다.


1. Planning Domain Definition Language - Wikipedia, accessed August 18, 2025, https://en.wikipedia.org/wiki/Planning_Domain_Definition_Language
2. PDDL Tutorial — PDDL4J 4.0 documentation, accessed August 18, 2025, http://pddl4j.imag.fr/pddl_tutorial.html
3. Mastering PDDL in Robotics - Number Analytics, accessed August 18, 2025, https://www.numberanalytics.com/blog/mastering-pddl-in-robotics
4. Automatic Planning – Chapter 3: PDDL - Foundations of Artificial Intelligence (FAI) Group, accessed August 18, 2025, https://fai.cs.uni-saarland.de/teaching/winter18-19/planning-material/planning03-pddl-post-handout.pdf
5. TDDD48 > Labs - IDA.LiU.SE, accessed August 18, 2025, https://www.ida.liu.se/~TDDD48/labs/2023/pddl.en.shtml
6. PDDL | The Planning Domain De nition Language - CMU School of Computer Science, accessed August 18, 2025, https://www.cs.cmu.edu/~mmv/planning/readings/98aips-PDDL.pdf
7. Planning.wiki - The AI Planning & PDDL Wiki: Home, accessed August 18, 2025, https://planning.wiki/
8. Writing Planning Domains and Problems in PDDL - IDA.LiU.SE, accessed August 18, 2025, https://www.ida.liu.se/~TDDC17/info/labs/planning/writing.html
9. What is PDDL? - Planning.wiki - The AI Planning & PDDL Wiki, accessed August 18, 2025, https://planning.wiki/guide/whatis/pddl
10. History and evolutions of the PDDL language - ResearchGate, accessed August 18, 2025, https://www.researchgate.net/figure/History-and-evolutions-of-the-PDDL-language_fig4_321632092
11. An Introduction to PDDL, accessed August 18, 2025, http://www.cs.toronto.edu/~sheila/2542/w09/A1/introtopddl2.pdf
12. Integrating AI Planning Semantics into SysML System Models for Automated PDDL File GenerationThis research paper [project iMOD and LaiLa] is funded by dtec.bw - arXiv, accessed August 18, 2025, https://arxiv.org/html/2506.06714v1
13. Leveraging Environment Interaction for Automated PDDL Generation and Planning with Large Language Models - arXiv, accessed August 18, 2025, https://arxiv.org/html/2407.12979v1
14. Large Language Models as Planning Domain Generators - arXiv, accessed August 18, 2025, https://arxiv.org/html/2405.06650v1
15. Writing Planning Domains and Problems in PDDL, accessed August 18, 2025, https://users.cecs.anu.edu.au/~patrik/pddlman/writing.html
16. Getting Started with PDDL | LearnPDDL - GitHub Pages, accessed August 18, 2025, https://fareskalaboud.github.io/LearnPDDL/
17. Getting Started · PDDL.jl, accessed August 18, 2025, https://juliaplanners.github.io/PDDL.jl/stable/tutorials/getting_started/
18. Modeling in PDDL - Episode1 - Blocksworld - YouTube, accessed August 18, 2025, https://www.youtube.com/watch?v=_NOVa4i7Us8
19. Mastering PDDL for Automated Reasoning - Number Analytics, accessed August 18, 2025, https://www.numberanalytics.com/blog/mastering-pddl-for-automated-reasoning
20. Class Notes on STRIPS - College of Science and Engineering, accessed August 18, 2025, https://classpages.cselabs.umn.edu/Spring-2020/csci4511W/strips.html
21. Classical STRIPS Planning, accessed August 18, 2025, https://web.engr.oregonstate.edu/~afern/classes/cs533/notes/strips-intro.pdf
22. PDDL 2.1 - Planning.wiki, accessed August 18, 2025, https://planning.wiki/ref/pddl21
23. PDDL 2.1 Domain - Planning.wiki, accessed August 18, 2025, https://planning.wiki/ref/pddl21/domain
24. Temporal Planning, accessed August 18, 2025, https://cw.fel.cvut.cz/b222/_media/courses/pui/tp.pdf
25. A brief overview of temporal planning, accessed August 18, 2025, https://users.aalto.fi/~rintanj1/temporalplanning.html
26. Decidability and Complexity of Action-Based Temporal Planning over Dense Time | Proceedings of the AAAI Conference on Artificial Intelligence, accessed August 18, 2025, https://ojs.aaai.org/index.php/AAAI/article/view/6539
27. PDDL 3 Problem - Planning.wiki, accessed August 18, 2025, https://planning.wiki/ref/pddl3/problem
28. Preferences and Soft Constraints in PDDL3 - UNIBS, accessed August 18, 2025, https://artificial-intelligence.unibs.it/gerevini/papers/WS-PREF-ICAPS06.pdf
29. Plan Constraints and Preferences in PDDL3, accessed August 18, 2025, https://ipc06.icaps-conference.org/deterministic/booklet/deterministic00.pdf
30. PDDL 3 Domain - Planning.wiki, accessed August 18, 2025, https://planning.wiki/ref/pddl3/domain
31. Plan Constraints and Preferences in PDDL3 The Language of the Fifth International Planning Competition - ResearchGate, accessed August 18, 2025, https://www.researchgate.net/publication/237534864_Plan_Constraints_and_Preferences_in_PDDL3_The_Language_of_the_Fifth_International_Planning_Competition
32. Nyx: Planning for Emerging Problems with PDDL+ and Beyond - ICAPS 2024, accessed August 18, 2025, https://icaps24.icaps-conference.org/program/workshops/keps-papers/KEPS-24_paper_18.pdf
33. Real-World Planning with PDDL+ and Beyond - arXiv, accessed August 18, 2025, https://arxiv.org/html/2402.11901v1
34. PDDL+ - Planning.wiki, accessed August 18, 2025, https://planning.wiki/ref/pddlplus
35. PDDL+ Domain - Planning.wiki, accessed August 18, 2025, https://planning.wiki/ref/pddlplus/domain
36. (PDF) PDDL+ : Modelling Continuous Time-dependent Effects - ResearchGate, accessed August 18, 2025, https://www.researchgate.net/publication/2836948_PDDL_Modelling_Continuous_Time-dependent_Effects
37. Heuristic Planning for PDDL+ Domains - IJCAI, accessed August 18, 2025, https://www.ijcai.org/Proceedings/16/Papers/455.pdf
38. Planning for Continuous Domains, accessed August 18, 2025, https://boa.unimib.it/retrieve/e39773b1-e9a1-35a3-e053-3a05fe0aac26/planning_for_continuous_domains.pdf
39. www.numberanalytics.com, accessed August 18, 2025, [https://www.numberanalytics.com/blog/mastering-pddl-in-robotics#:~:text=Examples%20of%20PDDL%20in%20Robotics%20Applications&text=Task%20planning%3A%20PDDL%20has%20been,interactions%20between%20humans%20and%20robots.](https://www.numberanalytics.com/blog/mastering-pddl-in-robotics#:~:text=Examples of PDDL in Robotics Applications&text=Task planning%3A PDDL has been,interactions between humans and robots.)
40. PDDL+ : Modelling Continuous Time-dependent Effects, accessed August 18, 2025, https://planning.wiki/_citedpapers/pddlplus2003.pdf
41. www.numberanalytics.com, accessed August 18, 2025, [https://www.numberanalytics.com/blog/mastering-pddl-for-automated-reasoning#:~:text=PDDL%20has%20been%20used%20in,delivery%20trucks%20and%20other%20vehicles.](https://www.numberanalytics.com/blog/mastering-pddl-for-automated-reasoning#:~:text=PDDL has been used in,delivery trucks and other vehicles.)
42. PDDL for Automated Reasoning: A Deep Dive - Number Analytics, accessed August 18, 2025, https://www.numberanalytics.com/blog/pddl-for-automated-reasoning-deep-dive
43. Classical Planning with LLM-Generated Heuristics: Challenging the State of the Art with Python Code - arXiv, accessed August 18, 2025, https://arxiv.org/html/2503.18809v1
44. Optimal Solutions to Large Logistics Planning Domain Problems, accessed August 18, 2025, https://ai.dmi.unibas.ch/papers/paul-et-al-socs2017.pdf
45. A Realistic Multi-Modal Cargo Routing Benchmark - AAAI, accessed August 18, 2025, https://cdn.aaai.org/ocs/ws/ws0087/10144-45991-1-PB.pdf
46. 9.0 Planning Domain Description Languages (PDDL) | by Ahmad Humaizi | Medium, accessed August 18, 2025, https://medium.com/@maizi5469/9-0-planning-domain-description-languages-pddl-0879646de46c
47. 5.0 Advancing Manufacturing Process Planning in Industry 4.0: The SWS2PDDL Conversion Challenge | by Ahmad Humaizi | Medium, accessed August 18, 2025, https://medium.com/@maizi5469/5-0-advancing-manufacturing-process-planning-in-industry-4-0-the-sws2pddl-conversion-challenge-2c9d70e7ec49
48. Production planning with a PDDL-enhanced digital twin. - ResearchGate, accessed August 18, 2025, https://www.researchgate.net/figure/Production-planning-with-a-PDDL-enhanced-digital-twin_fig1_350911320
49. Automated PDDL Domain File Generation for Enhancing Production System Development based on SysML Models - CEUR-WS.org, accessed August 18, 2025, https://ceur-ws.org/Vol-3883/paper3_IPS2.pdf
50. Manufacrturing Robot Planning in PDDL - Engineering AI Agents, accessed August 18, 2025, https://pantelis.github.io/aiml-common/lectures/planning/pddl/manufacturing/
51. Fully Automated Cyclic Planning for Large-Scale Manufacturing Domains - AAAI, accessed August 18, 2025, https://cdn.aaai.org/ojs/13620/13620-40-17138-1-2-20201228.pdf
52. Planning Domain Description Language (PDDL) | by Pranav Patki - Medium, accessed August 18, 2025, https://medium.com/@pranavrp16/planning-domain-description-language-pddl-635116a19a58
53. On Automating Video Game Testing by Planning and Learning - arXiv, accessed August 18, 2025, https://arxiv.org/html/2402.12393v1
54. On Automating Video Game Regression Testing by Planning and Learning - ICAPS 2024, accessed August 18, 2025, https://icaps24.icaps-conference.org/program/workshops/keps-papers/KEPS-24_paper_10.pdf
55. A Real-Time PDDL-Based Planning Component for Video Games - AAAI, accessed August 18, 2025, https://cdn.aaai.org/ojs/12370/12370-52-15898-1-2-20201228.pdf
56. Validating Domains and Plans for Temporal Planning via Encoding into Infinite-State Linear Temporal Logic, accessed August 18, 2025, https://ojs.aaai.org/index.php/AAAI/article/view/11018/10877
57. PDDL - Visual Studio Marketplace, accessed August 18, 2025, https://marketplace.visualstudio.com/items?itemName=jan-dolejsi.pddl
58. PDDL.jl: A Fast and Flexible Interface for Automated Planning | Xuan (Tan Zhi Xuan) | JuliaCon 2022 - YouTube, accessed August 18, 2025, https://www.youtube.com/watch?v=S757gSLScwM
59. Foundation Models for Logistics: Toward Certifiable, Conversational Planning Interfaces, accessed August 18, 2025, https://arxiv.org/html/2507.11352v1
60. Optimizing the Optimization of Planning Domains by Automatic Action Schema Splitting, accessed August 18, 2025, https://users.aalto.fi/~rintanj1/jussi/papers/ElahiRintanen24aaai.pdf
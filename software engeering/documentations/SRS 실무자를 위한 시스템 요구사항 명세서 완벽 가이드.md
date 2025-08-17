# 실무자를 위한 시스템 요구사항 명세서(SRS) 완벽 가이드

## 서론: 실패하지 않는 프로젝트의 초석, SRS

시스템 요구사항 명세서(Software Requirements Specification, SRS)는 단순한 기술 문서를 넘어선다. 이것은 소프트웨어 개발 프로젝트의 성공과 실패를 가르는 가장 중요한 산출물이자 프로젝트의 심장과 같다.1 소프트웨어 개발이라는 복잡한 퍼즐에서 모든 팀원이 같은 그림을 보게 만드는 유일한 청사진(Blueprint)이자 로드맵(Roadmap)이다.2 SRS는 프로젝트의 기둥 1 이자 종합 설계도 4 로서, 기획부터 분석, 설계, 구현, 시험, 그리고 유지보수까지 전 과정에서 의사결정의 핵심 기준이 된다.5

프로젝트 실패의 근본 원인은 기술적 문제보다 잘못된 요구사항에서 비롯되는 경우가 압도적으로 많다. 통계에 따르면, 프로젝트 재작업 비용은 전체 개발 비용의 30~50%를 차지하며, 이 중 무려 70~85%가 바로 부정확하거나 누락된 요구사항에서 발생한다.1 개발 후반 단계에서 요구사항 오류를 수정하는 비용은 초기 분석 단계에서 바로잡는 비용의 수십, 수백 배에 달하는 기하급수적인 증가를 보인다.1 국내 SI 프로젝트에서 제안요청서(RFP)부터 모호하게 정의되어 프로젝트 시작 후에야 산출물을 기반으로 요구사항을 역으로 정의하는 관행은 이러한 비용 폭증과 실패의 직접적인 원인이 된다.8 이 보고서는 이러한 재앙을 피하고, 프로젝트 성공 확률을 극적으로 높이는 SRS 작성의 모든 것을 다룬다.

SRS는 개발자만을 위한 문서가 아니다. 오히려 개발자, 기획자, 디자이너, 테스터, 마케팅 담당자, 그리고 최종적으로는 고객에 이르기까지 프로젝트에 관련된 모든 이해관계자(Stakeholder)가 동일한 목표를 향해 나아가게 하는 핵심적인 소통의 도구다.2 잘 작성된 SRS는 팀 간의 오해를 줄이고, 불필요한 논쟁을 없애며, 프로젝트가 올바른 방향으로 나아가고 있다는 확신을 심어준다. 이 문서는 단순한 지침을 넘어, 성공적인 프로젝트를 위한 가장 강력한 무기다.

------

## 1. 부: 요구사항 정의 - SRS 작성을 위한 준비 운동

훌륭한 SRS 문서는 그 자체로 프로젝트의 성숙도를 반영하는 지표다. 많은 프로젝트, 특히 국내 SI 환경에서는 모호한 제안요청서(RFP)로 계약을 체결한 뒤, 개발이 어느 정도 진행된 후에야 형식적으로 SRS를 작성하는 경우가 비일비재하다.5 이는 문서를 단지 행정적 절차로 취급하는 미성숙한 프로젝트 관리의 전형이다. 반면, 성공적인 프로젝트는 개발 시작 전, 모든 이해관계자가 참여하여 요구사항을 도출하고 분석하는 데 충분한 시간과 노력을 투자한다.1 이 과정 자체가 프로젝트의 숨겨진 리스크를 발견하고, 팀의 공감대를 형성하며, 명확한 목표를 설정하는 핵심 활동이기 때문이다. 따라서 SRS 작성법을 배우는 것은 단순히 문서 양식을 익히는 것이 아니라, 성공적인 프로젝트를 수행하기 위한 환경과 문화를 조성하는 방법을 배우는 것과 같다.

### 1.1  요구사항 도출: 숨겨진 요구사항을 찾아내는 기술

요구사항 도출(Elicitation)은 단순히 고객의 말을 받아 적는 수동적인 행위가 아니다. 이는 고객조차 명확히 인지하지 못하는 숨겨진 니즈, 당연하게 여기는 암묵적인 가정, 그리고 잠재적인 제약사항을 능동적으로 파헤치는 탐정 활동에 가깝다.11 고객은 종종 자신이 무엇을 원하는지 정확히 모르거나, 기술적인 언어로 표현하는 데 어려움을 겪는다.14 따라서 분석가는 다양한 기법을 활용하여 요구사항의 본질에 접근해야 한다.

프로젝트의 성격, 이해관계자의 특성, 개발 단계 등에 따라 최적의 도출 기법을 단독으로 또는 조합하여 사용해야 한다.15

- **인터뷰 (Interviews)**: 소수의 핵심 이해관계자(업무 전문가, 주요 사용자 등)로부터 깊이 있고 상세한 정보를 얻는 데 가장 효과적인 전통적 기법이다.11 인터뷰는 크게 두 가지 방식으로 나뉜다. 사전에 정의된 질문 목록 없이 자유롭게 대화하는 **개방형 인터뷰(Open-ended Interview)**는 문제의 전반적인 맥락을 파악하는 데 유용하다. 반면, 구체적인 질문지를 준비하여 진행하는 **구조적 인터뷰(Structured Interview)**는 특정 주제에 대한 명확한 답변을 얻는 데 효과적이다.13 성공적인 인터뷰를 위해서는 단순히 질문하고 답을 듣는 것을 넘어, 상대방의 답변을 요약하여 재확인하고, 모호한 부분은 구체적인 예시를 들어 파고드는 후속 질문을 던지는 기술이 필요하다.17
- **워크숍 (Workshops / Facilitated Application Specification Technique)**: 개발자, 기획자, 디자이너, 현업 사용자 등 다양한 배경을 가진 이해관계자들을 한자리에 모아 집중적인 토론을 통해 요구사항을 정의하고 즉석에서 합의를 도출하는 매우 강력한 기법이다.11 워크숍은 개별 인터뷰에서 발생할 수 있는 정보의 파편화를 막고, 서로 다른 관점을 조율하여 갈등을 조기에 해결하며, 참여자들의 소속감과 공감대를 형성하는 데 탁월한 효과를 보인다. 성공적인 워크숍을 위해서는 명확한 목표 설정, 경험 많은 진행자(Facilitator), 그리고 토론 규칙 준수가 필수적이다.11
- **프로토타이핑 (Prototyping)**: "백문이 불여일견"이라는 말을 가장 잘 실천하는 방법이다. 초기 요구사항을 바탕으로 와이어프레임, 목업(Mockup), 또는 실제 동작하는 간단한 시제품을 만들어 사용자에게 직접 보여주고 피드백을 받는다.12 프로토타입은 추상적인 텍스트로만 이루어진 요구사항 문서를 사용자가 실제로 만지고 경험할 수 있는 구체적인 형태로 바꿔준다. 이를 통해 사용자는 자신이 정말로 원했던 것이 무엇인지 깨닫게 되고, 개발팀은 잠재적인 설계 오류나 사용성 문제를 개발 초기에 발견하여 막대한 재작업 비용을 절감할 수 있다.9
- **사용자 관찰 (Observation / Ethnography)**: 분석가가 사용자의 실제 업무 환경에 직접 들어가 그들이 시스템을 어떻게 사용하고 업무를 처리하는지 면밀히 관찰하는 기법이다.11 인터뷰나 설문으로는 결코 드러나지 않는 암묵적인 지식, 비효율적인 업무 프로세스, 그리고 사용자들이 겪는 실제적인 어려움을 발견하는 데 매우 유용하다. 사용자가 "당연하게" 여기기 때문에 말하지 않는 중요한 요구사항을 발굴할 수 있는 독보적인 방법이다.
- **설문조사 (Surveys/Questionnaires)**: 지리적으로 분산되어 있거나 수가 많은 사용자 그룹으로부터 정량적인 데이터나 일반적인 의견을 효율적으로 수집할 때 유용하다.12 특정 기능의 선호도, 사용 빈도, 만족도 등을 통계적으로 분석하여 요구사항의 우선순위를 정하는 데 활용할 수 있다. 다만, 깊이 있는 정보를 얻기에는 한계가 있어 다른 기법과 병행하는 것이 좋다.
- **브레인스토밍 (Brainstorming)**: 정해진 규칙이나 비판 없이 참여자들이 자유롭게 아이디어를 교환하며 창의적인 해결책이나 잠재적 요구사항을 탐색하는 기법이다.13 기존의 틀에서 벗어난 혁신적인 아이디어를 얻거나, 프로젝트 초기에 가능한 모든 요구사항을 넓게 펼쳐보는 데 효과적이다.
- **유스케이스 분석 (Use Case Analysis)**: 사용자가 시스템과 상호작용하는 다양한 시나리오를 '액터(Actor)'와 '유스케이스(Use Case)' 관계로 구체적으로 기술하여 기능 요구사항을 체계적으로 도출하는 방법이다.11 "사용자가 시스템으로 무엇을 하는가"에 초점을 맞춰 기능의 범위와 흐름을 명확히 정의하고, 누락되는 기능이 없도록 돕는다.

### 1.2  요구사항 분석과 우선순위 결정

도출된 요구사항들은 날것 그대로의 원석과 같다. 이들 중에는 서로 모순되거나, 내용이 불완전하거나, 기술적으로 실현 불가능한 것들이 섞여 있다. 요구사항 분석 단계는 이 원석들을 정제하고 구조화하여 보석으로 만드는 과정이다.19

- **요구사항 모델링**: 복잡한 요구사항을 텍스트만으로 이해하는 데는 한계가 있다. 유스케이스 다이어그램(Use Case Diagram), 데이터 흐름도(Data Flow Diagram, DFD), 상태 전이 다이어그램(State Transition Diagram, STD), 클래스 다이어그램(Class Diagram) 등 다양한 모델링 도구를 사용하여 요구사항을 시각화하면, 시스템의 구조와 동작 방식을 보다 명확하게 이해하고 이해관계자 간의 소통 오류를 줄일 수 있다.11
- **우선순위 결정**: "모든 요구사항은 똑같이 중요하지 않다." 제한된 시간과 예산, 인력이라는 현실 속에서 모든 요구사항을 동시에 개발하는 것은 불가능하다.20 어떤 기능을 먼저 개발하고 어떤 기능을 나중에 할지, 혹은 아예 제외할지를 결정하는 우선순위 결정은 프로젝트 성공의 핵심 열쇠다.13
  - **MoSCoW 기법**: 요구사항을 네 가지 범주로 분류하여 개발 범위를 명확히 하는 가장 대중적인 기법 중 하나다.22
    - **Must-have**: 이 기능이 없으면 시스템 출시가 불가능하다. 반드시 구현해야 하는 핵심 요구사항.
    - **Should-have**: 매우 중요하지만, 대안이 있거나 다음 릴리즈로 미룰 수 있는 요구사항.
    - **Could-have**: 있으면 좋지만 없어도 큰 문제가 되지 않는, 사용자 경험을 약간 향상시키는 정도의 요구사항.
    - **Won't-have (this time)**: 이번 프로젝트 범위에서는 명확히 제외하기로 합의한 요구사항.
  - **가치 기반 우선순위 (Value-Based Prioritization)**: 각 요구사항이 고객에게 제공하는 비즈니스 가치(Value)와 구현에 따르는 기술적 위험(Risk) 및 비용(Cost)을 종합적으로 평가하여 우선순위를 결정하는 방식이다.11 가치는 높고 리스크와 비용은 낮은 요구사항이 가장 높은 우선순위를 갖는다.

### 1.3 Table 1: 요구사항 도출 기법 비교

| 기법 (Technique)                 | 개요 (Description)                                           | 장점 (Pros)                                                  | 단점 (Cons)                                                  | 최적 활용 사례 (Best For)                                    |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **인터뷰 (Interviews)**          | 이해관계자와의 일대일 또는 소그룹 대화를 통해 정보를 수집 12 | 깊이 있는 정보 획득, 민감한 주제 논의 가능, 유연한 질의응답  | 시간 소모적, 다수 의견 수렴 어려움, 인터뷰어의 편향 가능성   | 소수의 핵심 업무 전문가나 의사결정자로부터 상세한 요구사항을 도출할 때 |
| **워크숍 (Workshops)**           | 다양한 이해관계자가 모여 집중 토론을 통해 요구사항을 정의하고 합의 11 | 신속한 의사결정, 이해관계자 간 합의 형성, 갈등 조기 해결     | 많은 인원의 일정 조율 어려움, 숙련된 진행자 필수, 지배적인 의견에 쏠릴 위험 | 복잡한 요구사항에 대해 여러 부서의 합의가 필요하거나, 프로젝트 초기에 공감대를 형성하고자 할 때 |
| **프로토타이핑 (Prototyping)**   | 시제품을 만들어 사용자 피드백을 통해 요구사항을 구체화하고 검증 19 | 요구사항의 시각화, 잠재적 문제 조기 발견, 사용자 참여도 증진 | 개발 비용 발생, 시제품을 최종 제품으로 오인할 가능성, 범위 확장의 위험 | 사용자 인터페이스(UI/UX)가 중요한 시스템이나, 요구사항이 매우 추상적이고 불명확할 때 |
| **사용자 관찰 (Observation)**    | 사용자의 실제 업무 환경에서 행동을 관찰하여 암묵적 요구사항 발견 17 | 실제 업무 흐름 파악, 문서화되지 않은 니즈 발견, 객관적 데이터 확보 | 관찰자의 주관 개입 가능성, 사용자가 관찰을 의식하여 평소와 다르게 행동할 수 있음 | 기존 시스템을 개선하거나, 복잡한 업무 프로세스를 자동화하는 프로젝트에서 현황을 파악할 때 |
| **설문조사 (Surveys)**           | 표준화된 질문지를 통해 다수의 이해관계자로부터 정량적 데이터 수집 16 | 적은 비용으로 대규모 데이터 수집 가능, 통계 분석 용이        | 깊이 있는 정보 획득 한계, 낮은 응답률, 질문 설계의 어려움    | 다수의 사용자 그룹으로부터 기능 선호도, 사용 빈도 등 일반적인 경향을 파악하고자 할 때 |
| **브레인스토밍 (Brainstorming)** | 자유로운 아이디어 교환을 통해 창의적인 해결책이나 잠재적 요구사항 탐색 16 | 다양한 아이디어 생성, 창의성 촉진, 참여자들의 적극적 참여 유도 | 아이디어의 질이 보장되지 않음, 논점에서 벗어날 수 있음, 아이디어 정리 및 평가에 시간 소요 | 프로젝트 초기 단계에서 가능한 모든 요구사항을 탐색하거나, 새로운 기능에 대한 아이디어를 모을 때 |
| **유스케이스 분석 (Use Cases)**  | 사용자와 시스템 간의 상호작용 시나리오를 기술하여 기능 요구사항 도출 12 | 기능적 요구사항을 체계적으로 정리, 누락 방지, 테스트 케이스 작성의 기반 제공 | 비기능적 요구사항 표현의 한계, 작성에 시간과 노력이 소요됨   | 시스템이 수행해야 할 기능의 범위와 흐름을 명확하게 정의하고 싶을 때 |

------

## 2. 부: SRS 문서화 - 표준 구조 완전 정복

요구사항을 성공적으로 도출하고 분석했다면, 이제 그것을 체계적인 문서로 만드는 단계다. SRS 문서는 단순히 내용을 나열하는 것이 아니라, 전 세계적으로 통용되는 표준 구조를 따름으로써 그 자체로 하나의 잘 설계된 시스템처럼 기능해야 한다. 이 구조를 따르는 행위는 요구사항의 누락을 방지하고, 논리적 일관성을 유지하며, 모든 이해관계자가 동일한 틀 안에서 정보를 이해하도록 돕는다.

특히 IEEE 830 표준 22은 단순한 목차 그 이상이다. 각 섹션은 프로젝트 팀이 반드시 거쳐야 할 비판적 사고의 과정을 안내하는 질문지와 같다. 예를 들어, '제품의 관점(Product Perspective)' 섹션을 채우는 것은 "우리가 만드는 시스템이 더 큰 생태계에서 어떤 위치를 차지하는가?"라는 전략적 질문에 답하는 과정이다. '제약 조건(Constraints)' 섹션은 "우리의 발목을 잡을 수 있는 기술적, 예산적, 법적 한계는 무엇인가?"라는 리스크 관리 활동이다. 이처럼 SRS를 작성하는 과정은 단순히 문서를 채우는 행정 업무가 아니라, 프로젝트의 성공을 위해 잠재된 가정과 리스크를 수면 위로 끌어올려 해결하는 핵심적인 전략 활동이다.

### 2.1  IEEE 830 표준 기반 SRS 템플릿 해부

소프트웨어 개발 회사마다 사용하는 템플릿은 약간씩 다를 수 있지만, 그 근간은 대부분 IEEE 830 표준에 두고 있다.1 이 표준 구조는 수많은 프로젝트 경험을 통해 정제된, 요구사항 명세의 '모범 사례(Best Practice)'라 할 수 있다. 22

#### 1. 서론 (Introduction)

이 섹션은 독자가 SRS 문서를 이해하는 데 필요한 기본적인 배경 정보와 맥락을 제공한다.

- **1.1 목적 (Purpose)**: 이 SRS 문서가 왜 작성되었으며, 누구를 위한 문서인지(개발자, 기획자, QA, 고객 등) 명확히 기술한다. 이 문서를 통해 무엇을 달성하고자 하는지를 정의한다. 4
- **1.2 범위 (Scope)**: 개발할 소프트웨어의 이름과 버전을 명시하고, 시스템의 핵심 목표와 주요 기능을 간략하게 설명한다. 여기서 가장 중요한 것은 시스템이 **무엇을 할 것인지(in-scope)**와 더불어 **무엇을 하지 않을 것인지(out-of-scope)**를 명확히 선언하는 것이다. 범위의 경계를 명확히 긋는 것은 프로젝트 내내 끊임없이 발생하는 '범위蔓延(Scope Creep)'을 방지하는 첫 번째 방어선이다. 4
- **1.3 정의, 약어 (Definitions, Acronyms, and Abbreviations)**: 문서 전체에서 사용되는 모든 기술 용어, 비즈니스 용어, 약어 등을 명확하게 정의한다. 이는 이해관계자 간의 오해를 막고 원활한 소통을 보장한다. 22
- **1.4 참조 (References)**: 이 SRS를 작성하는 데 참고한 모든 외부 문서를 목록화한다. 예를 들어, 상위 수준의 시스템 요구사항 명세서, 프로젝트 계획서, 사용자 인터페이스 스타일 가이드, 관련 법규 및 표준 문서 등이 포함될 수 있다. 각 참조 문서에 쉽게 접근할 수 있도록 충분한 정보(제목, 버전, 날짜, 출처 등)를 제공해야 한다. 22
- **1.5 개요 (Overview)**: 이 문서의 나머지 부분이 어떻게 구성되어 있는지, 각 섹션이 어떤 내용을 담고 있는지 간략하게 안내하여 독자가 문서의 전체 구조를 파악할 수 있도록 돕는다. 22

#### 2.1.1  전반적인 설명 (Overall Description)

이 섹션은 세부적인 요구사항을 기술하기에 앞서, 제품에 영향을 미치는 일반적인 요소들과 전체적인 그림을 설명한다.

- **2.1 제품의 관점 (Product Perspective)**: 개발할 시스템이 어떤 맥락에 놓여 있는지를 설명한다. 완전히 새로운 독립형 제품인지, 기존 제품군의 다음 버전인지, 더 큰 시스템을 구성하는 하나의 모듈인지, 혹은 낡은 시스템을 대체하는 것인지 등을 기술한다. 만약 더 큰 시스템의 일부라면, 전체 시스템과의 관계 및 주요 인터페이스를 다이어그램과 함께 설명하는 것이 효과적이다. 22
- **2.2 제품 기능 (Product Functions)**: 시스템이 수행해야 할 핵심적인 기능들을 높은 수준에서 요약하여 설명한다. 이 부분은 상세한 기능 명세가 아니라, 독자가 시스템의 주요 역량을 빠르게 파악할 수 있도록 돕는 요약본이다. 유스케이스 다이어그램이나 최상위 데이터 흐름도 같은 시각 자료를 활용하면 이해도를 높일 수 있다. (상세한 기능 요구사항은 3장에서 다룬다.) 4
- **2.3 사용자 특성 (User Characteristics)**: 시스템을 사용하게 될 다양한 사용자 그룹(User Classes 또는 Persona)을 정의하고 각 그룹의 특징을 기술한다. 예를 들어, '관리자', '일반 회원', '콘텐츠 편집자' 등으로 나누고, 각 그룹의 기술 숙련도, 교육 수준, 주요 사용 기능, 권한 수준 등을 설명한다. 이는 사용자 중심의 설계와 사용성 요구사항을 정의하는 데 중요한 기초 자료가 된다. 4
- **2.4 제약 조건 (Constraints)**: 프로젝트의 선택지를 제한하는 모든 기술적, 환경적, 정책적 제약 사항을 명시한다. 이는 개발팀이 임의로 결정할 수 없는, 반드시 지켜야 할 규칙이다. 예를 들면 다음과 같다.
  - 특정 프로그래밍 언어(예: Java 11 이상)나 프레임워크 사용 의무
  - 특정 데이터베이스(예: Oracle, MS-SQL)와의 호환성 요구
  - 정부 규제 또는 산업 표준(예: 개인정보보호법, HIPAA) 준수
  - 하드웨어 제약(예: 특정 임베디드 장비에서 동작)
  - 예산 및 개발 일정 제약 4
- **2.5 가정 및 의존성 (Assumptions and Dependencies)**: 프로젝트의 성공이 달려있는 외부 요인이나, 참이라고 가정하는 사항들을 기술한다. 만약 이 가정들이 깨질 경우 프로젝트에 큰 위험이 될 수 있으므로, 이를 명시적으로 관리하는 것은 매우 중요하다. 예를 들면 다음과 같다.
  - *가정*: "사용자 인증은 사내 통합 인증(SSO) 시스템을 통해 이루어지며, 이 시스템은 99.9%의 가용성을 보장한다고 가정한다."
  - *의존성*: "결제 기능은 외부 PG사 'X'의 API에 의존하며, 해당 API의 변경 사항은 프로젝트에 직접적인 영향을 미친다." 22

#### 2.1.2  세부 요구사항 (Specific Requirements)

이 섹션은 SRS 문서의 심장부로, 시스템이 충족해야 할 모든 요구사항을 구체적이고 상세하게 기술한다. 일반적으로 기능 요구사항, 비기능 요구사항, 인터페이스 요구사항 등으로 나누어 작성한다.

### 2.2  기능 요구사항(Functional Requirements) 명세화

기능 요구사항은 시스템이 구체적으로 '무엇을 하는가(what the system does)'를 정의하는 부분이다. 사용자가 시스템을 통해 수행할 수 있는 모든 작업, 시스템이 처리해야 할 데이터, 그리고 그 결과가 여기에 해당한다. 4

- **작성 방법**: 명확하고 체계적인 기능 요구사항을 작성하기 위해 다음 원칙을 따라야 한다.
  - **고유 ID 부여**: 모든 요구사항에는 추적이 가능한 고유 식별자(ID)를 부여해야 한다. 예를 들어 `FR-AUTH-001` (Functional Requirement - Authentication - 001)과 같은 명명 규칙을 정하면, 요구사항이 변경되거나 테스트 케이스와 연동될 때 혼란 없이 관리할 수 있다. 이는 요구사항 추적성의 핵심이다. 21
  - **유스케이스/사용자 스토리 기반 작성**: "사용자는 [어떤 목표]를 달성하기 위해, [어떤 기능]을 할 수 있다"와 같은 사용자 스토리 형식으로 기능을 서술하면, 개발자가 기능의 목적과 맥락을 더 쉽게 이해할 수 있다. 이는 기술 중심이 아닌 사용자 중심의 사고를 촉진한다. 11
  - **입력-처리-출력(IPO) 명시**: 각 기능에 대해 사용자가 어떤 입력(Input)을 제공하고, 시스템이 그 입력을 받아 어떤 처리(Process)를 수행하며, 그 결과로 어떤 출력(Output)을 보여주는지를 명확히 기술해야 한다. 특히, 정상적인 흐름(Happy Path)뿐만 아니라, 잘못된 입력이나 시스템 오류가 발생했을 때의 예외 처리(Exception Handling) 시나리오도 반드시 포함해야 한다. 이는 시스템의 견고성을 결정하는 중요한 요소다. 27
- **예시 (온라인 뱅킹 시스템)** 30:
  - **요구사항 ID**: `FR-AUTH-001`
  - **요구사항 명**: 사용자 로그인
  - **설명**: 등록된 사용자는 자신의 아이디와 비밀번호를 사용하여 시스템에 접근할 수 있어야 한다.
  - **사전 조건**: 사용자는 인터넷에 연결된 기기를 통해 로그인 페이지에 접근한 상태이다.
  - **입력**: 사용자 아이디(문자열), 사용자 비밀번호(문자열)
  - **처리**:
    1. 시스템은 입력된 아이디와 비밀번호를 수신한다.
    2. 시스템은 데이터베이스에서 해당 아이디의 정보와 일치하는지 확인한다. 비밀번호는 저장된 해시값과 비교한다.
    3. 인증에 성공하면, 사용자 세션을 생성하고 개인화된 메인 페이지로 리디렉션한다.
  - **출력**:
    - (성공 시) 사용자의 계좌 정보를 볼 수 있는 메인 대시보드 화면.
    - (실패 시) "아이디 또는 비밀번호가 일치하지 않습니다."라는 오류 메시지.
  - **예외 처리**:
    - `FR-AUTH-002`: 로그인 시도 5회 연속 실패 시, 해당 계정은 보안을 위해 30분간 잠금 처리되며, 사용자에게 이메일로 관련 사실을 통지해야 한다.

### 2.3  비기능 요구사항(Non-functional Requirements) 명세화

기능 요구사항이 '무엇을'에 대한 것이라면, 비기능 요구사항은 시스템의 품질과 제약 조건, 즉 '어떻게 동작하는가(how the system operates)'를 정의한다..4 이는 종종 간과되지만, 사용자 만족도와 시스템의 성패를 좌우하는 결정적인 요소다. 예를 들어, 기능이 아무리 완벽해도 시스템이 너무 느리거나, 자주 멈추거나, 보안에 취약하다면 아무도 사용하지 않을 것이다.

비기능 요구사항은 프로젝트의 기술 스택, 아키텍처, 그리고 팀 구성을 결정하는 중요한 근거가 된다. 예를 들어, "초당 10,000건의 트랜잭션 처리"라는 성능 요구사항은 고성능 서버와 분산 아키텍처, 그리고 부하 테스트 전문가의 필요성을 의미한다. "모든 금융 데이터는 암호화되어야 한다"는 보안 요구사항은 보안 전문가와 암호화 라이브러리, 그리고 정기적인 모의 해킹의 필요성을 암시한다. 이처럼 비기능 요구사항을 명확히 정의하는 것은 프로젝트의 예산, 일정, 인력 구성을 현실적으로 계획하기 위한 필수 과정이다.

- **주요 유형 및 작성법**: 비기능 요구사항은 '좋다', '빠르다'와 같은 모호한 표현을 절대 사용해서는 안 된다. 반드시 **측정 가능(Measurable)**하고 **검증 가능(Verifiable)**한 구체적인 수치로 표현해야 한다. 21
  - **성능 (Performance)**: 시스템의 응답 시간, 처리량(Throughput), 동시 접속자 수 등을 정의한다.
    - *나쁜 예*: "시스템은 빨라야 한다." 36
    - *좋은 예*: `NFR-PERF-001`: 상품 목록 페이지는 피크 타임(오후 8-10시)에 500명의 동시 접속자 환경에서 95%의 요청에 대해 3초 이내에 로드되어야 한다. 22
  - **보안 (Security)**: 데이터 암호화 수준, 접근 제어 정책, 인증 방식, 외부 공격(예: SQL Injection, XSS) 방어 등 보안 관련 요구사항을 정의한다.
    - *나쁜 예*: "시스템은 안전해야 한다."
    - *좋은 예*: `NFR-SEC-001`: 사용자의 개인정보(이름, 주소, 연락처)와 비밀번호는 데이터베이스에 저장될 때 반드시 AES-256 알고리즘으로 암호화되어야 한다. 22
  - **가용성 (Availability)**: 시스템이 장애 없이 정상적으로 운영되어야 하는 시간을 비율(%)로 명시한다. 'Five Nines'(99.999%)와 같이 업계 표준을 사용하기도 한다.
    - *나쁜 예*: "시스템은 멈추면 안 된다."
    - *좋은 예*: `NFR-AVAIL-001`: 시스템은 연간 99.9%의 가용성을 보장해야 한다 (계획된 유지보수 시간을 제외한 연간 총 장애 시간은 8.76시간을 초과할 수 없다). 22
  - **사용성 (Usability)**: 사용자가 시스템을 얼마나 쉽게 배우고 효율적으로 사용할 수 있는지를 정의한다.
    - *나쁜 예*: "사용자 친화적인 인터페이스여야 한다." 22
    - *좋은 예*: `NFR-USABL-001`: 처음 방문한 사용자는 별도의 교육이나 도움말 없이 5분 이내에 원하는 상품을 찾아 장바구니에 담는 작업을 완료할 수 있어야 한다. 22
  - **확장성 (Scalability)**: 미래에 사용자 수, 데이터 양, 트래픽이 증가했을 때 시스템이 어떻게 대응해야 하는지를 정의한다.
    - *나쁜 예*: "나중에 사용자가 늘어나도 잘 동작해야 한다."
    - *좋은 예*: `NFR-SCALE-001`: 시스템은 현재 사용자 수의 3배까지 증가하더라도, 웹 서버와 애플리케이션 서버의 수평적 확장(Scale-out)만으로 응답 시간을 현재 수준으로 유지할 수 있어야 한다. 27

### 2.4 Table 2: 주요 비기능 요구사항 유형 및 측정 지표 예시

| 유형 (Category)                  | 설명 (Description)                                           | 핵심 측정 지표 (Key Metrics)                                 | 좋은 작성 예시 (Good Example)                                |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **성능 (Performance)**           | 시스템의 응답 속도, 처리 용량, 자원 사용 효율성              | 응답 시간(초), 처리량(TPS, 초당 트랜잭션), 동시 사용자 수, CPU/메모리 사용률(%) | `NFR-PERF-001`: 사용자 검색 기능은 95%의 경우 2초 이내에 결과를 반환해야 한다. |
| **보안 (Security)**              | 데이터 보호, 비인가 접근 방지, 시스템 무결성 유지            | 암호화 알고리즘(예: AES-256), 접근 제어 규칙, OWASP Top 10 준수 여부, 인증 실패율 | `NFR-SEC-001`: 모든 사용자 비밀번호는 SHA-256 이상의 해시 함수와 Salt를 사용하여 암호화되어 저장되어야 한다. |
| **가용성 (Availability)**        | 시스템이 장애 없이 지속적으로 서비스를 제공할 수 있는 능력   | 가용성 비율(%), 평균 무고장 시간(MTBF), 평균 수리 시간(MTTR), 연간 허용 다운타임(시간) | `NFR-AVAIL-001`: 시스템은 연 99.95%의 가용성을 보장해야 한다 (연간 총 장애 시간 4.38시간 이내). |
| **사용성 (Usability)**           | 사용자가 시스템을 얼마나 쉽고 편리하게 배우고 사용할 수 있는지 | 작업 완료 시간(초/분), 에러 발생률(%), 학습 시간(분), 사용자 만족도 점수(예: SUS) | `NFR-USABL-001`: 신규 사용자는 별도의 교육 없이 10분 이내에 핵심 기능(상품 검색 및 주문)을 완료할 수 있어야 한다. |
| **유지보수성 (Maintainability)** | 시스템을 얼마나 쉽게 수정, 개선, 확장할 수 있는지            | 코드 복잡도(Cyclomatic Complexity), 코드 커버리지(%), 신규 기능 추가 소요 시간(시간/일) | `NFR-MAINT-001`: 새로운 결제 모듈을 추가하는 데 필요한 개발 공수는 5인/일(Man-Day)을 초과하지 않아야 한다. |
| **확장성 (Scalability)**         | 부하 증가에 대응하여 시스템 성능을 얼마나 유연하게 확장할 수 있는지 | 지원 가능한 최대 사용자/데이터 양, 서버 추가 시 성능 향상률(선형적/비선형적) | `NFR-SCALE-001`: 사용자 수가 2배 증가하더라도 서버 추가만으로 선형적인 성능 확장이 가능해야 한다. |
| **이식성 (Portability)**         | 시스템을 다른 환경(OS, 하드웨어, 브라우저)으로 얼마나 쉽게 이전할 수 있는지 | 지원 OS/브라우저 목록, 환경 이전 소요 시간(시간/일)          | `NFR-PORT-001`: 웹 애플리케이션은 최신 버전의 Chrome, Firefox, Safari, Edge 브라우저에서 모든 기능이 동일하게 동작해야 한다. |

------

## 3. 부: 명품 요구사항 작성법 - 명확함과 구체성의 기술

요구사항의 품질이 프로젝트의 품질을 결정한다. 잘 쓰인 요구사항 하나가 잘못 쓰인 요구사항 열 개보다 훨씬 더 큰 가치를 지닌다. '명품' 요구사항은 개발팀에게 명확한 목표를 제시하고, 테스터에게 정확한 검증 기준을 제공하며, 모든 이해관계자에게 프로젝트가 올바른 방향으로 가고 있다는 신뢰를 준다. 이를 위해서는 단순히 내용을 채우는 것을 넘어, 명확함과 구체성의 기술을 연마해야 한다.

### 3.1  좋은 요구사항의 7가지 조건

성공적인 프로젝트에서 발견되는 좋은 요구사항들은 공통적으로 다음과 같은 7가지 특성을 만족한다. SRS를 작성하거나 검토할 때, 이 기준들을 체크리스트처럼 활용하면 요구사항의 품질을 크게 향상시킬 수 있다. 6

1. **완전성 (Complete)**: 요구사항을 이해하고 구현하는 데 필요한 모든 정보가 빠짐없이 기술되어야 한다. 시스템이 특정 조건에서 어떻게 반응해야 하는지, 어떤 오류 메시지를 보여줘야 하는지 등 모든 시나리오가 포함되어야 한다. '나중에 결정함(To Be Determined, TBD)'과 같은 표기는 가능한 한 피하고, 만약 불가피하다면 누가, 언제까지 결정할 것인지를 명시해야 한다. 21
2. **명확성/비모호성 (Unambiguous)**: 요구사항은 모든 사람이 읽었을 때 단 하나의 의미로만 해석될 수 있어야 한다. '빠른 응답', '효율적인 처리', '사용자 친화적인 디자인'과 같은 주관적이고 애매한 표현은 절대 금물이다. 이러한 표현은 사람마다 다르게 해석하여 결국 잘못된 결과물을 낳게 된다. 21
3. **일관성 (Consistent)**: 문서 내의 요구사항들이 서로 모순되거나 충돌해서는 안 된다. 예를 들어, 한 요구사항에서는 '모든 사용자는 실명 인증을 해야 한다'고 명시하고, 다른 요구사항에서는 '익명으로 게시글을 작성할 수 있다'고 기술하면 혼란을 야기한다. 용어의 사용 또한 문서 전체에 걸쳐 일관되어야 한다. 21
4. **정확성 (Correct)**: 모든 요구사항은 고객이나 최종 사용자가 실제로 원하는 바를 정확하게 반영해야 한다. 분석가의 잘못된 해석이나 가정으로 인해 실제 필요와 다른 요구사항이 정의되면, 프로젝트는 처음부터 잘못된 방향으로 나아가게 된다. 22
5. **검증 가능성 (Verifiable/Testable)**: 모든 요구사항은 테스트나 분석, 검증 등의 방법을 통해 시스템이 해당 요구사항을 만족하는지 여부를 객관적으로 확인할 수 있어야 한다. "시스템은 2초 이내에 응답해야 한다"는 검증 가능하지만, "시스템은 빨라야 한다"는 검증이 불가능하다. 모든 요구사항은 결국 테스트 케이스로 변환될 수 있어야 한다. 14
6. **수정 용이성 (Modifiable)**: 프로젝트 환경은 끊임없이 변하며, 요구사항 변경은 필연적이다. 따라서 SRS 문서는 변경이 발생했을 때 쉽게 수정하고 관리할 수 있도록 구조화되어야 한다. 요구사항들이 서로 너무 복잡하게 얽혀있지 않고, 각 요구사항이 독립적으로 기술되어 있다면 수정이 용이해진다. 목차와 색인을 잘 구성하는 것도 중요하다. 6
7. **추적 가능성 (Traceable)**: 각 요구사항은 그 출처(예: 비즈니스 목표, 상위 요구사항, 고객 인터뷰)부터 시작하여 설계 문서, 소스 코드, 테스트 케이스, 최종 기능에 이르기까지 양방향으로 추적이 가능해야 한다. 이를 위해 모든 요구사항에 고유한 ID를 부여하는 것이 필수적이다. 추적성이 확보되면, 특정 요구사항 변경 시 어떤 부분에 영향을 미치는지(Impact Analysis) 신속하게 파악할 수 있다. 6

### 3.2  SMART 원칙을 활용한 측정 가능한 요구사항 작성

좋은 요구사항의 조건, 특히 '명확성'과 '검증 가능성'을 만족시키기 위한 강력한 도구가 바로 SMART 목표 설정 프레임워크다. 본래 경영학에서 목표 관리를 위해 사용되던 이 원칙은, 모호한 요구사항을 구체적이고 측정 가능한 명세로 바꾸는 데 매우 효과적이다. 23

- **S (Specific - 구체적인)**: 요구사항은 누가, 무엇을, 왜, 어떻게 할 것인지 명확하고 구체적으로 기술해야 한다. "사용자 관리 기능"이 아니라 "관리자는 신규 사용자를 등록하고, 기존 사용자의 권한을 수정하며, 퇴사한 사용자를 비활성화할 수 있다"와 같이 구체적으로 정의해야 한다.
- **M (Measurable - 측정 가능한)**: 요구사항의 달성 여부를 객관적으로 판단할 수 있는 정량적인 지표를 포함해야 한다. "성능 개선"이 아니라 "페이지 로딩 시간을 현재 5초에서 2초로 단축한다"와 같이 측정 가능한 목표를 제시해야 한다.
- **A (Achievable - 달성 가능한)**: 요구사항은 주어진 기술, 자원, 시간 내에서 현실적으로 달성 가능해야 한다. "100% 버그 없는 시스템"이나 "24/7/365 무중단 운영"과 같은 목표는 이론적으로는 훌륭하지만, 현실적으로는 막대한 비용이 들거나 달성이 불가능할 수 있다. 37
- **R (Relevant - 관련성 있는)**: 모든 요구사항은 프로젝트의 전체 목표 및 상위 비즈니스 목표와 직접적인 관련이 있어야 한다. 단순히 기술적으로 흥미롭거나 특정 이해관계자의 개인적인 선호 때문에 추가된 요구사항은 프로젝트의 본질을 흐리고 자원을 낭비하게 만든다. 45
- **T (Time-bound - 시간제한이 있는)**: 요구사항이 언제까지 달성되어야 하는지 명확한 기한이나 특정 시점을 명시해야 한다. 이는 프로젝트 계획 수립과 우선순위 결정에 중요한 정보를 제공한다. "1단계 오픈 시까지" 또는 "2024년 4분기까지"와 같이 구체적인 시간 제약을 두어야 한다.

### 3.3  Bad vs. Good: 실제 사례로 배우는 요구사항 개선법

이론만으로는 부족하다. 실제 사례를 통해 나쁜 요구사항이 어떻게 개선되는지 직접 보는 것이 가장 효과적인 학습 방법이다. 아래 표는 현장에서 흔히 발견되는 모호하고 검증 불가능한 요구사항과, 이를 명확하고 구체적으로 개선한 사례를 비교하여 보여준다. 22

요구사항을 작성할 때 특히 피해야 할 단어들이 있다. **'등(etc.)', '포함하여(including)', '~와 같은(such as)', '다양한(various)'** 등은 요구사항의 범위를 모호하게 만든다. 또한 **'빠른(fast)', '쉬운(easy)', '효율적인(efficient)', '유연한(flexible)', '안정적인(robust)'** 과 같은 주관적인 형용사나, **'지원하다(support)', '처리하다(handle)', '관리하다(manage)', '개선하다(improve)'** 와 같은 포괄적인 동사는 구체적인 행위와 기준을 명시하지 않아 해석의 여지를 남긴다. 이러한 '위험한 단어'들을 발견하면, 즉시 구체적인 수치와 명확한 행위로 바꾸려는 노력을 해야 한다. 36

### 3.4 Table 3: 잘못된 요구사항과 개선된 요구사항 비교

| 구분 (Category)                    | 나쁜 요구사항 (Bad Requirement)                              | 문제점 (Problem)                                             | 좋은 요구사항 (Good Requirement)                             |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **기능 (Functionality)**           | 시스템은 사용자 정보를 관리해야 한다.                        | '관리'의 범위가 모호함. 생성, 조회, 수정, 삭제 중 무엇을 의미하는지 불분명함. | `FR-USER-001`: 관리자는 신규 사용자 계정을 생성할 수 있다. `FR-USER-002`: 관리자는 기존 사용자의 이름과 연락처를 수정할 수 있다. `FR-USER-003`: 관리자는 퇴사한 사용자의 계정을 비활성화할 수 있다. |
| **성능 (Performance)**             | 검색 응답은 빨라야 한다.                                     | '빠르다'는 주관적이며 측정 불가능함. 어떤 상황에서의 응답 시간인지 명시되지 않음. | `NFR-PERF-002`: 키워드 검색 시, 동시 접속자 100명 환경에서 95%의 요청에 대해 1초 이내에 검색 결과를 반환해야 한다. |
| **사용성 (Usability)**             | 시스템은 사용하기 쉬워야 한다.                               | '쉽다'는 사용자의 경험과 숙련도에 따라 달라지는 주관적인 개념임. 검증할 방법이 없음. | `NFR-USABL-002`: 처음 접속한 사용자는 별도의 매뉴얼 없이 3분 이내에 '비밀번호 찾기' 기능을 성공적으로 완료할 수 있어야 한다. |
| **보안 (Security)**                | 시스템은 안전해야 한다.                                      | '안전'의 기준이 무엇인지 전혀 알 수 없음. 모든 종류의 위협을 막는 것은 불가능함. | `NFR-SEC-002`: 시스템은 OWASP Top 10에 명시된 모든 웹 취약점에 대해 방어책을 마련해야 하며, 정기적인 취약점 점검을 통과해야 한다. |
| **구현 명시 (Over-specification)** | 파이 차트는 `highcharts.js` 라이브러리를 사용해서 그려야 한다. 48 | '어떻게(How)'를 명시하여 개발팀의 기술 선택 자유도를 제한함. 더 나은 대안이 있어도 사용하지 못하게 만듦. | `FR-RPRT-001`: 사용자는 월별 매출 현황을 카테고리별 비중으로 보여주는 파이 차트 형태로 조회할 수 있어야 한다. |
| **모호한 범위 (Ambiguous Scope)**  | 사용자는 CSV, Excel 등 다양한 형식으로 보고서를 내보낼 수 있다. | '등(etc.)'이라는 표현이 지원해야 할 파일 형식의 범위를 불명확하게 만듦. | `FR-RPRT-002`: 사용자는 보고서를 CSV와 PDF 두 가지 형식으로 내보낼 수 있어야 한다. |

------

## 4. 부: 관리와 활용 - 살아있는 SRS 만들기

SRS는 한 번 작성하고 책장에 꽂아두는 기념비적인 문서가 아니다. 프로젝트가 끝나는 날까지 계속해서 참조되고, 수정되며, 진화해야 하는 '살아있는 문서(Living Document)'다.38 프로젝트의 현실은 끊임없이 변한다. 시장 상황이 바뀌고, 고객의 요구가 추가되며, 기술적인 제약이 발견되기도 한다. 이러한 변화를 효과적으로 관리하고 SRS를 프로젝트의 실질적인 중심축으로 만들기 위해서는 적절한 도구와 프로세스가 필수적이다.

특히, 현대 소프트웨어 개발에서 '추적성(Traceability)'은 살아있는 SRS를 구현하는 핵심 엔진이다.22 추적성이란, 하나의 요구사항이 어떤 비즈니스 목표에서 시작되어, 어떤 설계와 코드로 구현되고, 어떤 테스트를 통해 검증되었는지를 명확하게 연결하는 것을 의미한다. 과거에는 이 모든 연결을 수작업으로 관리해야 했지만, 이제는 Jira와 Confluence 같은 도구들을 통해 이 과정을 자동화하고, SRS를 프로젝트의 정적인 설계도에서 동적인 중앙 신경계로 탈바꿈시킬 수 있다.

### 4.1  요구사항 관리 도구 활용법: Jira와 Confluence

SRS를 효과적으로 관리하기 위해 가장 널리 사용되는 조합은 Atlassian의 Confluence와 Jira다. 이 두 도구는 각각의 목적을 가지고 있지만, 서로 긴밀하게 연동될 때 강력한 시너지를 발휘한다. 50

- **Confluence: 지식과 요구사항의 중앙 저장소**: Confluence는 위키(Wiki) 기반의 협업 도구로, SRS 문서를 작성하고, 공유하며, 버전 관리하는 데 최적화되어 있다.50

  - **협업 편집**: 여러 팀원이 동시에 문서에 접근하여 편집하고 댓글을 통해 의견을 나눌 수 있다. 이를 통해 모든 이해관계자가 SRS 작성 과정에 참여하고 최신 정보에 기반한 의사결정을 내릴 수 있다.
  - **템플릿 활용**: 표준 SRS 템플릿을 만들어두면, 새로운 프로젝트마다 일관된 구조의 문서를 쉽게 생성할 수 있다.
  - **버전 관리**: 모든 변경 이력이 자동으로 기록되므로, 누가, 언제, 무엇을 수정했는지 쉽게 추적할 수 있으며, 필요 시 이전 버전으로 복원할 수도 있다.51

- **Jira: 작업 추적과 실행의 엔진**: Jira는 애자일 프로젝트 관리 도구로, 개별 요구사항(기능)을 '이슈(Issue)' 또는 '티켓(Ticket)'이라는 작업 단위로 만들어 개발 진행 상황을 추적하고 관리하는 데 사용된다.50

  - **작업 분할**: Confluence에 기술된 '사용자 로그인 기능'이라는 큰 요구사항을 Jira에서는 '로그인 UI 개발', '백엔드 인증 API 개발', '로그인 실패 시나리오 테스트' 등 구체적인 작업들로 나눌 수 있다.
  - **워크플로 관리**: 각 작업이 '할 일(To Do)', '진행 중(In Progress)', '완료(Done)' 등 어떤 상태에 있는지 칸반 보드나 스크럼 보드를 통해 시각적으로 관리할 수 있다.
  - **보고 및 분석**: 번다운 차트(Burndown Chart), 속도 보고서(Velocity Report) 등 다양한 보고 기능을 통해 프로젝트의 진행 상황과 팀의 생산성을 분석할 수 있다.51

- **연동의 힘 - 완벽한 추적성 구축**: Confluence와 Jira를 연동하는 것이 바로 '살아있는 SRS'의 핵심이다.

  1. **요구사항 정의 (Confluence)**: 먼저 Confluence 페이지에 `FR-AUTH-001: 사용자 로그인`이라는 요구사항을 상세히 기술한다.
  2. **작업 생성 및 연결 (Jira)**: Confluence 페이지에서 바로 Jira 이슈를 생성한다. 'DEV-123: 로그인 기능 개발'이라는 이슈가 만들어지고, 이 이슈는 자동으로 `FR-AUTH-001` 요구사항과 연결된다.
  3. **개발 및 코드 커밋**: 개발자는 'DEV-123' 이슈를 해결하기 위해 코드를 작성하고, 코드 커밋 메시지에 'DEV-123' 이슈 키를 포함하여 Git에 푸시한다.
  4. **테스트 케이스 연결**: QA 엔지니어는 'DEV-123' 이슈에 대한 테스트 케이스를 작성하고, 테스트 결과를 Jira 이슈에 업데이트한다.

  - 이러한 흐름을 통해, Confluence의 요구사항 페이지에서 이 요구사항과 관련된 모든 Jira 작업, 코드 커밋, 테스트 결과를 한눈에 확인할 수 있는 완벽한 추적성이 확보된다. 50

### 4.2  실전 SRS 템플릿 및 작성 예시

이론을 실제 프로젝트에 적용할 수 있도록, 다양한 도메인의 SRS 작성 예시를 분석하고 바로 활용 가능한 템플릿을 제공한다.

- **범용 SRS 템플릿 (Markdown 형식)**: 이 보고서에서 다룬 모든 원칙과 IEEE 830 표준 구조를 반영한 범용 템플릿이다. 이 템플릿을 기반으로 각 프로젝트의 특성에 맞게 수정하여 사용할 수 있다. 28

  # [프로젝트명] 시스템 요구사항 명세서 (SRS)

  - **버전**: 1.0
  - **작성일**: YYYY-MM-DD
  - **작성자**: [이름]

  ------

  ## 1. 서론 (Introduction)

  ### 1.1. 목적 (Purpose)

  ### 1.2. 범위 (Scope)

  ### 1.3. 정의, 약어 (Definitions, Acronyms, and Abbreviations)

  ### 1.4. 참조 (References)

  ### 1.5. 개요 (Overview)

  ## 2. 전반적인 설명 (Overall Description)

  ### 2.1. 제품의 관점 (Product Perspective)

  ### 2.2. 제품 기능 (Product Functions)

  ### 2.3. 사용자 특성 (User Characteristics)

  ### 2.4. 제약 조건 (Constraints)

  ### 2.5. 가정 및 의존성 (Assumptions and Dependencies)

  ## 3. 세부 요구사항 (Specific Requirements)

  ### 3.1. 기능 요구사항 (Functional Requirements)

  #### 3.1.1. [기능 그룹 1: 예) 사용자 관리]

  - **FR-USER-001**: [요구사항명]

  ### 3.2. 비기능 요구사항 (Non-functional Requirements)

  #### 3.2.1. 성능 (Performance)

  - **NFR-PERF-001**: [요구사항명]

  #### 3.2.2. 보안 (Security)

  - **NFR-SEC-001**: [요구사항명]

  ### 3.3. 외부 인터페이스 요구사항 (External Interface Requirements)

  #### 3.3.1. 사용자 인터페이스 (User Interfaces)

  #### 3.3.2. 소프트웨어 인터페이스 (Software Interfaces)

  ## 4. 부록 (Appendices)

  **도메인별 예시 분석**:

  - **이커머스 플랫폼 (E-commerce Platform)**: 상품 검색(필터, 정렬), 장바구니, 보안 결제(PG 연동), 주문 관리, 재고 관리 등의 핵심 기능 요구사항이 중심이 된다. 비기능적으로는 대규모 트래픽을 처리하기 위한 성능 및 확장성, 개인정보 및 결제 정보 보호를 위한 강력한 보안 요구사항이 필수적이다. 54
  - **모바일 애플리케이션 (Mobile App)**: 사용자 등록/로그인, 프로필 관리, 콘텐츠 피드 등 일반적인 기능 외에, 푸시 알림, 카메라/GPS/연락처 등 디바이스 하드웨어 접근, 오프라인 모드 지원 등 모바일 환경에 특화된 요구사항이 중요하다. 또한, iOS와 Android 양대 플랫폼에서의 일관된 사용자 경험을 위한 이식성 요구사항도 고려해야 한다. 55
  - **의료 정보 시스템 (Health Information System, HIS)**: 환자 정보 관리(EHR), 원격 진료, 의료 영상 전송(DICOM) 등의 기능이 필요하다. 무엇보다 중요한 것은 비기능 요구사항으로, 환자의 민감한 의료 정보를 보호하기 위한 HIPAA와 같은 강력한 법적 규제 준수, 의료 데이터 표준(HL7, DICOM) 준수, 그리고 시스템 장애가 환자의 생명과 직결될 수 있으므로 매우 높은 수준의 신뢰성과 가용성이 요구된다. 61
  - **금융 시스템 (Financial System)**: 계좌 조회, 자금 이체, 대출 신청 등 금융 거래 기능이 핵심이다. 비기능적으로는 단 1원의 오차도 허용하지 않는 데이터 정합성, 모든 거래 기록을 추적할 수 있는 감사 추적(Audit Trail) 기능, 그리고 금융 사기를 방지하기 위한 최고 수준의 보안(다중 인증, 이상 거래 탐지 등)이 요구된다. 34

### 4.3  변경 관리와 버전 관리

"요구사항은 변한다"는 소프트웨어 개발의 유일한 불변의 진리다. 중요한 것은 변경을 막는 것이 아니라, 변경이 발생했을 때 혼란을 최소화하고 프로젝트를 안정적으로 유지하는 체계적인 프로세스를 갖추는 것이다. 20

- **변경 요청 프로세스 수립**: 모든 요구사항 변경은 구두가 아닌 공식적인 절차를 따라야 한다. **변경 요청서(Change Request Form)**를 사용하여 변경의 내용, 요청 사유, 기대 효과, 긴급도 등을 명확히 문서화하도록 한다.
- **영향 분석 (Impact Analysis)**: 변경 요청이 접수되면, 분석가나 아키텍트는 해당 변경이 프로젝트의 다른 부분에 미칠 영향을 분석해야 한다.
  - **기술적 영향**: 어떤 다른 요구사항, 모듈, 인터페이스에 영향을 주는가?
  - **자원 영향**: 추가적인 개발 공수, 테스트 시간, 비용은 얼마나 드는가?
  - **일정 영향**: 프로젝트 전체 일정에 어떤 지연을 초래하는가?
  - 요구사항 추적성이 잘 구축되어 있다면 이 영향 분석 과정이 훨씬 빠르고 정확해진다.
- **변경 통제 위원회 (Change Control Board, CCB)**: 프로젝트 관리자, 핵심 개발자, 기획자, 고객 대표 등 주요 이해관계자로 구성된 위원회를 운영한다. CCB는 접수된 변경 요청서와 영향 분석 결과를 검토하고, 변경을 승인할지, 기각할지, 아니면 보류할지를 결정한다. 이는 특정 개인의 판단이 아닌, 합의에 기반한 체계적인 의사결정을 보장한다. (공공 SW 사업의 과업변경심의위원회와 유사한 역할을 한다.) 5
- **버전 관리 및 소통**: 승인된 모든 변경 사항은 SRS 문서에 즉시 반영되어야 한다. Confluence와 같은 도구의 히스토리 기능을 활용하여 모든 변경 이력을 투명하게 관리하고, 문서의 버전을 명확히 명시한다(예: v1.0 -> v1.1). 그리고 가장 중요한 것은, 변경된 내용이 모든 관련 팀원에게 신속하고 명확하게 전파되도록 하는 것이다.

------

## 5. 결론: SRS, 성공적인 소프트웨어 개발의 나침반

SRS는 프로젝트의 방향을 제시하는 나침반이자, 성공적인 소프트웨어 개발의 가장 단단한 초석이다. 이 문서는 단순히 '무엇을 만들 것인가'를 정의하는 것을 넘어, '왜 만드는가', '누구를 위해 만드는가', '어떤 제약 조건 하에서 만드는가'에 대한 깊은 고민과 이해관계자 간의 사회적 합의를 담아내는 그릇이다.

잘 만들어진 SRS는 개발팀에게는 명확한 구현 목표를, 테스트팀에게는 정확한 검증 기준을, 프로젝트 관리자에게는 범위와 일정 관리의 근거를 제공한다. 더 나아가 고객과 모든 이해관계자에게는 프로젝트가 투명하고 체계적으로 관리되고 있다는 신뢰를 심어주며, 필연적으로 발생하는 변경과 갈등 상황에서 합리적인 의사결정을 내릴 수 있는 기준점 역할을 한다.

프로젝트 실패의 가장 큰 원인이 기술 부족이 아닌 소통의 부재와 잘못된 요구사항 정의에 있다는 사실을 기억해야 한다.1 SRS를 작성하는 과정은 그 자체로 가장 중요한 소통 활동이며, 이 과정에 쏟는 노력과 시간은 프로젝트 후반부에 발생할 수 있는 막대한 재작업 비용과 실패의 위험을 막는 최고의 투자다.

이 보고서에서 제시한 요구사항 도출 기법, IEEE 830 표준 구조, 명품 요구사항 작성 원칙, 그리고 관리 및 활용 방안을 당신의 다음 프로젝트에 꾸준히 실천하라. SRS를 단순한 산출물이 아닌, 프로젝트의 성공을 이끄는 핵심 전략으로 활용한다면, 당신의 프로젝트는 혼란의 안개를 헤치고 성공이라는 목적지를 향해 흔들림 없이 나아갈 것이다.

### 5.1 최종 체크리스트

SRS 작성을 완료한 후, 배포하기 전에 아래 항목들을 통해 최종적으로 점검하라.

- **[ ] 구조와 형식**: IEEE 830 표준 구조를 따랐는가? 목차와 페이지 번호가 정확한가?
- **[ ] 완전성**: 모든 섹션이 채워졌는가? '나중에 결정(TBD)' 항목이 남아있다면, 책임자와 기한이 명시되었는가?
- **[ ] 고유 ID**: 모든 개별 요구사항(기능/비기능)에 추적 가능한 고유 ID가 부여되었는가?
- **[ ] 명확성**: '등', '빠른', '쉬운', '효율적인'과 같은 모호한 표현이 사용되지 않았는가? 모든 요구사항이 단 하나의 의미로만 해석되는가?
- **[ ] 검증 가능성**: 모든 요구사항은 테스트를 통해 검증할 수 있도록 구체적인 수치나 조건으로 명시되었는가?
- **[ ] 비기능 요구사항**: 성능, 보안, 가용성, 사용성 등 중요한 비기능 요구사항이 빠짐없이 정의되었는가?
- **[ ] 일관성**: 요구사항 간에 서로 모순되는 내용이 없는가? 문서 전체에서 용어가 일관되게 사용되었는가?
- **[ ] 이해관계자 검토**: 모든 핵심 이해관계자가 이 문서를 검토하고 내용에 동의했는가? 검토 이력이 기록되었는가?

#### **참고 자료**

1. SRS(Software Requirements Specification)의 중요성, accessed July 29, 2025, https://www.allofsoftware.net/2008/11/srssoftware-requirements-specification.html
2. 소프트웨어 엔지니어링의 SRS: 개발자에게 필요한 이유 - Ranktracker, accessed July 29, 2025, https://www.ranktracker.com/ko/blog/srs-in-software-engineering-why-developers-need-it/
3. Write a Software Requirement Document (With Template) [2025] - Asana, accessed July 29, 2025, https://asana.com/resources/software-requirement-document-template
4. [Project] 사용자 요구사항 정의서 (SRS)란? - velog, accessed July 29, 2025, [https://velog.io/@bagt/%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%9A%94%EA%B5%AC%EC%82%AC%ED%95%AD-%EC%A0%95%EC%9D%98%EC%84%9C-SRS%EB%9E%80](https://velog.io/@bagt/사용자-요구사항-정의서-SRS란)
5. 요구명세(SRS)의 중요성과 제도화 방향 - SPRi - 소프트웨어정책연구소, accessed July 29, 2025, https://spri.kr/posts/view/22234?code=data_all&study_type=ai_brief
6. SRS (SW Requirements Specification) - 요구공학 - Char - 티스토리, accessed July 29, 2025, https://charstring.tistory.com/982
7. Requirements Elicitation Techniques: Comparative Study - IJRDET, accessed July 29, 2025, https://www.ijrdet.com/files/Volume1Issue3/IJRDET_1213_01.pdf
8. 요구명세 - SRS (Software Requirement Specipication) 의 중요성과 SW진흥법 전면 개정, accessed July 29, 2025, https://it-license.tistory.com/30
9. 6 Steps to Requirements Gathering for Project Success [2025] - Asana, accessed July 29, 2025, https://asana.com/resources/requirements-gathering
10. Requirement Gathering Methods - UMSL, accessed July 29, 2025, [https://www.umsl.edu/~sauterv/analysis/F2015/Requirement%20Gathering%20Methods.html.htm](https://www.umsl.edu/~sauterv/analysis/F2015/Requirement Gathering Methods.html.htm)
11. CH9. 소프트웨어 요구사항 개요 및 명세 방법 - velog, accessed July 29, 2025, [https://velog.io/@eukddan/CH9.-%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4-%EC%9A%94%EA%B5%AC%EC%82%AC%ED%95%AD-%EA%B0%9C%EC%9A%94-%EB%B0%8F-%EB%AA%85%EC%84%B8-%EB%B0%A9%EB%B2%95](https://velog.io/@eukddan/CH9.-소프트웨어-요구사항-개요-및-명세-방법)
12. Requirements Elicitation in Software Engineering: A Complete Guide - Testbytes, accessed July 29, 2025, https://www.testbytes.net/blog/requirements-elicitation/
13. Requirements Elicitation - Software Engineering - GeeksforGeeks, accessed July 29, 2025, https://www.geeksforgeeks.org/software-engineering/software-engineering-requirements-elicitation/
14. How to avoid "bad" requirements [closed] - Stack Overflow, accessed July 29, 2025, https://stackoverflow.com/questions/1691270/how-to-avoid-bad-requirements
15. Comparison of Various Requirements Elicitation Techniques, accessed July 29, 2025, https://www.ijcaonline.org/archives/volume116/number4/20322-2408/
16. Requirements Gathering – How We Solve The Biggest Problems With Consulting, accessed July 29, 2025, https://nmgtechnologies.com/blog/requirement-gathering-solve-biggest-problems-consulting
17. Requirements Gathering Techniques for Agile Product Teams - Jama Software, accessed July 29, 2025, https://www.jamasoftware.com/requirements-management-guide/requirements-gathering-and-management-processes/11-requirements-gathering-techniques-for-agile-product-teams/
18. 15 Best Requirements Gathering Techniques Explained - Plaky, accessed July 29, 2025, https://plaky.com/blog/requirements-gathering-techniques/
19. 정보처리기사 2020 필기 정리 [1] 소프트웨어 설계 1장 요구 사항 확인(2), accessed July 29, 2025, https://jjeny0911.tistory.com/7
20. 요구사항 명세서 작성법 - ernest Dev - 티스토리, accessed July 29, 2025, https://ernest-o.tistory.com/35
21. 소프트웨어 개발 요구사항 명세서 작성 방법 - Dev Inventory - 티스토리, accessed July 29, 2025, https://devinventory.tistory.com/20
22. 소프트웨어 요구사항 명세서(SRS) 작성법 | PromleeBlog, accessed July 29, 2025, https://www.promleeblog.com/blog/post/308-effective-software-requirements
23. How to Write Good Software Requirements (with Examples), accessed July 29, 2025, https://www.modernrequirements.com/blogs/good-software-requirements/
24. IEEE Std 830 스타일의 소프트웨어 요구사항 명세서 - 팬더의 프로그래밍 - 티스토리, accessed July 29, 2025, https://chlalgud8505.tistory.com/56
25. SRS(Software requirements specification, 요구사항명세서) 표준 IEEE-STD-830 - 미처서의미처서만든블로그 - 티스토리, accessed July 29, 2025, [https://kmckmc.tistory.com/entry/SRSSoftware-requirements-specification-%EC%9A%94%EA%B5%AC%EC%82%AC%ED%95%AD%EB%AA%85%EC%84%B8%EC%84%9C-%ED%91%9C%EC%A4%80-IEEESTD830](https://kmckmc.tistory.com/entry/SRSSoftware-requirements-specification-요구사항명세서-표준-IEEESTD830)
26. pers0n4/srs-kor: Korean translation of IEEE Std 830. - GitHub, accessed July 29, 2025, https://github.com/pers0n4/srs-kor
27. 소프트웨어 요구사항 명세서(SRS) 완벽 정리 - velog, accessed July 29, 2025, [https://velog.io/@neverleaveualong/%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4-%EC%9A%94%EA%B5%AC%EC%82%AC%ED%95%AD-%EB%AA%85%EC%84%B8%EC%84%9C](https://velog.io/@neverleaveualong/소프트웨어-요구사항-명세서)
28. jam01/SRS-Template: A template for Software ... - GitHub, accessed July 29, 2025, https://github.com/jam01/SRS-Template
29. 요구사항 명세서 작성방법 - 개발학습 블로그, accessed July 29, 2025, https://do-devel.tistory.com/16
30. Functional vs. Non Functional Requirements - GeeksforGeeks, accessed July 29, 2025, https://www.geeksforgeeks.org/software-engineering/functional-vs-non-functional-requirements/
31. [Stack-Overflow 클론] 요구사항 명세서 - i'm done - 티스토리, accessed July 29, 2025, https://all-done.tistory.com/108
32. Functional & Non-Functional Requirements | PDF | Mobile App - Scribd, accessed July 29, 2025, https://www.scribd.com/document/502962211/Functional-Non-functional-Requirements
33. Functional Requirement - Glossary - DevX, accessed July 29, 2025, https://www.devx.com/terms/functional-requirement/
34. What are Non-functional Requirements: Types, Examples & Approaches - Visure Solutions, accessed July 29, 2025, https://visuresolutions.com/alm-guide/non-functional-requirements/
35. NFRs: What is Non Functional Requirements (Example & Types) - BrowserStack, accessed July 29, 2025, https://www.browserstack.com/guide/non-functional-requirements-examples
36. Bad Requirements - Devopedia, accessed July 29, 2025, https://devopedia.org/bad-requirements
37. 10 of the Worst Requirements I have Ever Seen - Reqtest, accessed July 29, 2025, https://reqtest.com/en/knowledgebase/10-of-the-worst-requirements-i-have-ever-seen/
38. How to Write a Software Requirements Specification (SRS ..., accessed July 29, 2025, https://www.perforce.com/blog/alm/how-write-software-requirements-specification-srs-document
39. 요구사항 명세서(Software Requirement Specification) - Goooooood - 티스토리, accessed July 29, 2025, https://seing.tistory.com/203
40. 소프트웨어 요구 사항 사양 (SRS) : 팁 및 템플릿 - Visure Solutions, accessed July 29, 2025, [https://visuresolutions.com/ko/%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4-%EC%9A%94%EA%B5%AC-%EC%82%AC%ED%95%AD-%EC%82%AC%EC%96%91-srs-%ED%8C%81-%ED%85%9C%ED%94%8C%EB%A6%BF/](https://visuresolutions.com/ko/소프트웨어-요구-사항-사양-srs-팁-템플릿/)
41. interaction.html - FSU Computer Science, accessed July 29, 2025, https://www.cs.fsu.edu/~baker/swe1/restricted/notes/SRS.html
42. Testable & Non-Testable Requirements - Not Just A Tester !, accessed July 29, 2025, https://not-just-a-tester.blogspot.com/2012/01/testable-non-testable-requirements.html
43. Ensuring Requirements are Testable in Agile - Pluralsight, accessed July 29, 2025, https://www.pluralsight.com/resources/blog/guides/ensuring-requirements-are-testable-in-agile
44. SMART 목표란 무엇이며 나만의 SMART 목표 세우는 방법 - Tableau, accessed July 29, 2025, https://www.tableau.com/ko-kr/learn/articles/smart-goals-criteria
45. SMART 목표란 무엇이고 어떻게 작성하는가 - Slingshot, accessed July 29, 2025, https://www.slingshotapp.io/ko/blog/how-to-write-smart-goals-a-step-by-step-guide
46. 예시와 팁을 통해 살펴보는 더 좋은 SMART 목표 작성하기 - Asana, accessed July 29, 2025, https://asana.com/ko/resources/smart-goals
47. 정량적 성과 측정이 어려운 부서를 위한 목표설정 전략 - 오픈애즈, accessed July 29, 2025, https://openads.co.kr/content/contentDetail?contsId=12628
48. Business Requirements Examples for Software / AnvilEight Blog, accessed July 29, 2025, https://anvileight.com/blog/posts/software-requirements-specifications-good-and-bad-examples/
49. Writing Better Requirements - University of Ottawa, accessed July 29, 2025, [https://www.site.uottawa.ca/~bochmann/SEG3101/Notes/SEG3101-ch3-3%20-%20WritingBetterRequirements.pdf](https://www.site.uottawa.ca/~bochmann/SEG3101/Notes/SEG3101-ch3-3 - WritingBetterRequirements.pdf)
50. Confluence와 Jira의 심층 비교 - Lark, accessed July 29, 2025, https://www.larksuite.com/ko_kr/blog/confluence-vs-jira
51. Confluence 대 Jira: 2025년에 가장 적합한 프로젝트 관리 도구는 무엇입니까? - ClickUp, accessed July 29, 2025, https://clickup.com/ko/blog/66467/confluence-vs-jira
52. Confluence 대 Jira [심층 비교 및 기능] - TextCortex, accessed July 29, 2025, https://textcortex.com/ko/post/confluence-vs-jira
53. Jira와 Confluence: 더 나은 프로젝트 관리 도구는 무엇입니까? - 비주얼 솔루션, accessed July 29, 2025, https://visuresolutions.com/ko/jira-guide/jira-vs-confluence/
54. Software Requirements Specification Sample Format & Document Guide with Examples, accessed July 29, 2025, https://paceai.co/software-requirements-specification-sample-format/
55. Requirements specification document example, accessed July 29, 2025, https://assets-global.website-files.com/66f3d20188e193c145e94eb2/67f7671a5593a44f3a51effa_xexodusasebeledu.pdf
56. Creativa Membership Mobile App. SRS - TIEC, accessed July 29, 2025, [https://tiec.gov.eg/English/Proposals/Documents/Creativa_Membership%20App%20SRS.pdf](https://tiec.gov.eg/English/Proposals/Documents/Creativa_Membership App SRS.pdf)
57. Software Requirements Specification - Webkul, accessed July 29, 2025, https://webkul.com/srs-builder/assets/Software-Requirements-Specification.pdf
58. Software Requirements Specification - METU Ceng Demo Day 2025, accessed July 29, 2025, https://senior.ceng.metu.edu.tr/2016/newline/SRS.pdf
59. Software Requirements Specification for Scan&Cook - Cloudfront.net, accessed July 29, 2025, https://d3jqtupnzefbtn.cloudfront.net/andersenlab/new-andersensite/SRS.pdf
60. Software Requirements Specification - Bellevue College, accessed July 29, 2025, https://www.bellevuecollege.edu/wp-content/uploads/sites/135/2019/04/SRS_RoadTrip.pdf
61. 건강 정보 시스템에 대한 심층 가이드 | 스코픽 - Scopic, accessed July 29, 2025, https://scopicsoftware.com/ko/blog/health-information-systems/
62. 병원정보시스템 (HIS): 개념 및 주요 서브 시스템, accessed July 29, 2025, https://www.ominext.com/kr/blog/basic-components-of-hospital-information-system
63. A manager's guide for monitoring data integrity in financial systems - GovInfo, accessed July 29, 2025, https://www.govinfo.gov/content/pkg/GOVPUB-C13-d57e13513900beb646b81776f3728939/pdf/GOVPUB-C13-d57e13513900beb646b81776f3728939.pdf
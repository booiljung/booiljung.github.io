# YAGNI(You Aren't Gonna Need It) 원칙에 대한 심층 고찰


소프트웨어 시스템은 본질적으로 시간이 흐름에 따라 복잡성이 증가하는 경향을 보인다. 이러한 현상을 '소프트웨어 엔트로피'라 칭하며, 이는 유지보수 비용의 증가, 변경의 어려움, 그리고 예측 불가능한 버그 발생 가능성의 증대로 이어진다.1 현대 소프트웨어 공학의 핵심 과제 중 하나는 이 불가피한 복잡성을 관리하고 제어하는 것이다. YAGNI(You Aren't Gonna Need It) 원칙은 이러한 복잡성의 핵심 원인 중 하나인 '추측에 기반한 미래 기능의 선제적 구현'을 원천적으로 차단하려는 강력하고도 단순한 시도이다.

YAGNI는 익스트림 프로그래밍(Extreme Programming, XP)의 핵심 원칙 중 하나로, "현재 명확하게 필요하다고 간주될 때까지 기능을 추가하지 말라"는 명제로 요약된다.3 이 원칙의 철학적 배경은 XP의 공동 설립자인 론 제프리스(Ron Jeffries)가 남긴 "실제로 필요할 때 무조건 구현하되, 그저 필요할 것이라고 예상할 때에는 절대 구현하지 말라"는 말에 명확히 드러난다.4 YAGNI는 단독으로 존재하는 원칙이 아니라, XP의 더 큰 지침인 '동작 가능한 가장 단순한 것을 하라(DTSTTCPW, Do The Simplest Thing That Could Possibly Work)'를 구체화하는 핵심적인 실천 방안이다.4

표면적으로 YAGNI는 단순히 기능을 구현하지 않는 수동적인 태도나 심지어 게으름으로 오해받을 수 있다.6 그러나 XP가 강조하는 지속적인 리팩토링, 자동화된 테스트, 그리고 빈번한 통합이라는 맥락 속에서 YAGNI를 이해할 때, 그 본질은 전혀 다른 차원에서 해석된다.5 YAGNI는 무작정 구현을 회피하는 것이 아니라, 불확실성이 높은 상황에서 섣부른 결정을 내리는 대신 의사결정 자체를 '지연'시키는 전략에 가깝다. 미래의 요구사항이 보다 명확해지는 시점, 즉 정보의 가치가 가장 높아지는 순간까지 구현을 미루는 것은 초기 추측에 기반한 개발보다 실패 확률을 현저히 낮추는 합리적인 선택이다.2 따라서 YAGNI는 '일을 하지 않는 것'이 아니라, '가장 적절한 시점까지 구현을 의도적으로 미루는' 적극적 지연(Active Deferral) 전략이자 정교한 리스크 관리 기법으로 재정의될 수 있다. 이는 낭비를 제거하고 가치 흐름을 최적화하려는 린(Lean) 사고방식과도 깊은 연관을 맺는다.1

본 보고서는 YAGNI를 단순한 코딩 지침을 넘어, 불확실성 하에서 최적의 가치를 창출하기 위한 경제적, 심리학적, 공학적 전략으로 심층 분석하고자 한다. 이를 위해 경제학의 비용 모델, 인지심리학의 편향 이론, 그리고 소프트웨어 설계론의 핵심 원칙들을 통합하여 YAGNI 원칙이 지닌 다층적 의미와 실용적 가치를 탐색할 것이다.


YAGNI 원칙의 핵심은 제한된 개발 자원을 가장 가치 있는 곳에 집중시키는 경제적 합리성에 있다. 이 장에서는 YAGNI 원칙을 위반하고 불필요한 기능을 미리 구현하는 행위, 즉 '추측성 기능(Presumptive Feature)' 개발이 야기하는 다양한 경제적 비용을 분석한다. 이를 통해 YAGNI가 단순한 프로그래밍 스타일이 아니라, 프로젝트의 총소유비용(TCO)을 최적화하고 기회비용을 관리하는 핵심적인 경제 원칙임을 입증한다.


마틴 파울러(Martin Fowler)를 비롯한 여러 전문가들은 당장 필요하지 않은 기능을 구현할 때 발생하는 비용이 단순히 개발에 투입된 시간과 노력에 그치지 않는다고 지적한다.1 추측성 기능은 네 가지 유형의 복합적인 비용을 발생시키며, 이들은 종종 눈에 보이지 않게 프로젝트의 건전성을 잠식한다.1

- **구축 비용 (Cost of Build):** 이는 가장 명시적인 비용으로, 특정 기능을 분석, 설계, 구현, 테스트하는 데 직접적으로 소요되는 모든 자원을 포함한다. 개발팀의 시간, 인프라 비용 등이 여기에 해당한다.1
- **지연 비용 (Cost of Delay):** 추측성 기능을 개발하는 데 자원을 소모함으로써, 현재 시장에 더 시급하고 명확한 가치를 제공할 수 있는 다른 핵심 기능의 출시를 지연시키는 데서 발생하는 기회비용이다. 이는 시장 선점 기회의 상실, 경쟁 우위 약화, 예상 매출의 지연 등 측정 가능하거나 불가능한 형태로 나타난다.1
- **유지/소유 비용 (Cost of Carry):** 일단 시스템에 추가된 코드는 그 자체로 부채가 된다. 불필요한 기능은 코드베이스의 복잡도를 높이고, 이로 인해 향후 모든 신규 기능 개발, 디버깅, 리팩토링 작업의 난이도와 소요 시간을 증가시킨다. 이는 마치 금융 부채에 대한 이자처럼, 기능이 존재하는 동안 지속적으로 발생하는 숨겨진 비용이다.1
- **수리 비용 (Cost of Repair):** 미래에 대한 추측이 빗나갔을 때 발생하는 비용이다. 구현된 기능이 실제 요구사항과 다르거나 아예 불필요해졌을 경우, 이를 수정하거나 안전하게 제거하는 데 상당한 노력이 필요하다. 특히 잘못된 추상화는 단순한 기능 제거보다 훨씬 더 큰 규모의 재설계와 재작업을 요구할 수 있다.1

이러한 다차원적인 비용 구조를 이해하는 것은 YAGNI의 경제적 타당성을 평가하는 데 필수적이다. 다음 표는 각 비용의 특성을 체계적으로 요약한 것이다.

**표 1: 추측성 기능의 비용 분석**

| 비용 유형                  | 정의                                              | 발생 시점                              | 예시                                                         |
| -------------------------- | ------------------------------------------------- | -------------------------------------- | ------------------------------------------------------------ |
| 구축 비용 (Cost of Build)  | 기능을 개발하는 데 드는 직접적 노력과 자원        | 개발 초기                              | 기능 구현에 20 Person-Day 소요                               |
| 지연 비용 (Cost of Delay)  | 다른 중요 기능 출시 지연으로 인한 기회 손실       | 기능 개발 기간 동안                    | 핵심 기능 출시 지연으로 월 2천만 원의 예상 매출 손실 발생    |
| 유지 비용 (Cost of Carry)  | 코드 복잡도 증가로 인한 지속적인 생산성 저하 비용 | 기능이 코드베이스에 존재하는 기간 내내 | 신규 기능 개발 시 복잡도 증가로 인해 평균 15%의 추가 공수 발생 |
| 수리 비용 (Cost of Repair) | 요구사항 불일치로 인한 기능 수정 또는 제거 비용   | 실제 요구사항이 명확해졌을 때          | 잘못 구현된 기능을 제거하고 재설계하는 데 30 Person-Day 소요 |


경제학의 근본 개념인 기회비용은 '여러 대안 중 하나를 선택했을 때 포기해야만 하는 나머지 대안들 중 가장 가치가 큰 것'을 의미한다.12 소프트웨어 개발에서 모든 결정은 자원 배분에 관한 것이며, 필연적으로 기회비용을 수반한다.

YAGNI의 관점에서, 불확실한 미래 가치를 지닌 추측성 기능 A를 개발하기로 한 결정의 기회비용은, 그 시간과 자원을 투입하여 개발할 수 있었던, 현재 명확한 요구사항에 기반한 기능 B가 창출할 수 있었던 순가치이다.14 이를 간단한 모델로 표현하면 다음과 같다.
$$
\text{OC}_{A} = E - E[V_A]
$$
여기서 `$\text{OC}_{A}$`는 기능 A를 선택했을 때의 기회비용, `E`는 포기한 대안 B의 기대 가치(Expected Value), `E[V_A]`는 선택한 대안 A의 기대 가치를 의미한다. YAGNI 원칙은 추측성 기능의 기대 가치 `E[V_A]`가 미래의 불확실성으로 인해 매우 낮거나 심지어 음수일 수 있음을 경고한다. 반면, 현재 요구사항에 기반한 기능 B의 기대 가치 `E`는 상대적으로 확실하고 높다. 따라서 YAGNI는 기회비용을 최소화하는 합리적인 의사결정 프레임워크를 제공한다.


지연 비용(CoD)은 제품이나 기능의 출시가 지연될 때 조직이 감수해야 하는 경제적 손실을 정량화하는 린-애자일(Lean-Agile)의 핵심 지표이다.15 제품 개발 전문가인 돈 라이너츤(Don Reinertsen)은 "만약 단 하나만 정량화해야 한다면, 지연 비용을 정량화하라"고 강조하며 그 중요성을 역설했다.16

CoD는 일반적으로 특정 기능이 제공하는 '사용자-비즈니스 가치(Value)'와 그 가치가 시간에 따라 얼마나 민감하게 변하는지를 나타내는 '긴급성(Urgency)'의 함수로 모델링된다.18
$$
\text{CoD} = f(\text{Value, Urgency})
$$
YAGNI는 이 모델에서 '가치(Value)'가 불확실하거나 현재 '0'에 가까운 항목에 귀중한 개발 자원을 투입하는 것을 방지하는 역할을 한다. 추측성 기능에 시간을 낭비하면, 필연적으로 CoD가 높은(즉, 가치와 긴급성이 모두 높은) 핵심 기능들의 출시가 지연된다. 이는 프로젝트 전체의 경제적 손실을 극대화하는 결과를 초래한다.1 결국 YAGNI는 팀이 CoD가 가장 높은 작업에 집중하도록 자연스럽게 유도함으로써, 전체적인 가치 전달 속도를 최적화하는 데 기여한다.

소프트웨어 개발에서 미래의 요구사항은 본질적으로 불확실하다.19 이러한 불확실성 하에서 의사결정을 내리는 것은 금융 공학의 '실물 옵션(Real Options)' 이론과 유사한 측면을 가진다. 실물 옵션 이론은 불확실한 미래에 특정 행동을 할 수 있는 '권리'(옵션)의 가치를 평가하는 방법론이다.20 YAGNI 원칙에 따라 특정 기능의 구현을 '지연'하는 것은, 해당 기능을 미래에 더 확실한 정보를 가지고 구현할 수 있는 '콜 옵션(Call Option)'을 보유하는 것과 같다. 지금 당장 불확실성에 투자(개발 비용 지불)하는 대신, 시장의 반응과 실제 사용자 데이터를 확인한 후 그 기능의 가치가 충분히 높다고 판단될 때만 옵션을 행사(개발을 시작)할 수 있는 유연성의 가치를 확보하는 것이다. 반대로, 추측에 기반해 미리 개발하는 것은 불확실한 기초자산에 대해 옵션의 시간 가치와 유연성의 가치를 포기하고 즉시 권리를 행사하는 비합리적인 투자 결정일 수 있다. 이 관점에서 YAGNI는 단순한 개발 원칙을 넘어, 프로젝트의 유연성이라는 무형 자산의 가치를 극대화하고 불확실성이라는 리스크를 효과적으로 헤지(hedge)하는 정교한 재무적 의사결정 프레임워크로 볼 수 있다.


YAGNI 원칙을 위반하는 근본적인 원인은 단순히 기술적 판단 착오에만 있지 않다. 그 이면에는 인간의 사고 과정에 내재된 체계적인 오류, 즉 '인지 편향(Cognitive Bias)'이 깊숙이 자리 잡고 있다. 이 장에서는 YAGNI 위반을 부추기는 주요 인지 편향을 분석하고, YAGNI가 이러한 심리적 함정에 대한 효과적인 방어기제로서 어떻게 작동하는지를 논증한다.


소프트웨어 개발은 기계적인 작업이 아니라, 개발자의 직관과 판단에 크게 의존하는 고도의 지적 활동이다.22 이로 인해 개발자들은 인간이라면 누구나 겪는 인지 편향에 매우 취약하다.23 소프트웨어 공학 분야의 연구에 따르면, 개발 과정 전반에 걸쳐 최소 37가지의 각기 다른 인지 편향이 관찰되었으며, 이는 요구사항 분석의 오류, 설계의 비효율성, 테스트의 부실함으로 이어져 프로젝트 실패의 주요 원인으로 작용한다.24 한 연구에서는 개발 과정에서 되돌려지거나 폐기된(reverted) 작업의 약 70%가 최소 하나 이상의 인지 편향과 관련이 있었다고 보고하며, 그 심각성을 뒷받침한다.26


과잉 엔지니어링(Over-engineering) 또는 과잉 기능화(Over-featuring)는 여러 인지 편향이 복합적으로 작용한 결과물이다. 그중 YAGNI 원칙 위반과 특히 밀접한 관련이 있는 편향들은 다음과 같다.

- **과신 편향 (Overconfidence Bias):** 자신의 지식, 예측 능력, 또는 문제 해결 능력을 실제보다 높게 평가하는 경향이다.27 소프트웨어 개발에서 이는 "미래에 이 기능이 분명히 필요할 것이며, 나는 그것을 완벽하게 예측하고 이상적으로 설계할 수 있다"는 착각으로 이어진다. 이러한 과신은 불확실성을 과소평가하게 만들고, 아직 존재하지도 않는 문제에 대한 정교한 해결책, 즉 불필요한 기능을 미리 구현하게 만든다.24 또한, 자신의 이해도를 과신하여 요구사항 분석을 소홀히 하는 원인이 되기도 한다.26
- **확증 편향 (Confirmation Bias):** 자신의 기존 신념이나 가설을 지지하는 정보는 적극적으로 찾고 수용하는 반면, 그에 반하는 정보는 무시하거나 외면하는 경향이다.27 개발자가 "이 추상화 계층은 미래에 유용할 것이다"라는 가설을 세우면, 확증 편향은 그 가설을 뒷받침하는 가상의 시나리오에만 집중하게 만든다. 이 기능이 실제로는 필요 없을 수 있다는 반박 증거나 동료의 비판적인 의견은 평가절하될 가능성이 높다.26
- **가용성 편향 (Availability Heuristic):** 판단을 내릴 때, 머릿속에 쉽게 떠오르는 최신 정보나 생생한 경험에 과도하게 의존하는 경향이다.26 예를 들어, "이전 프로젝트에서 마이크로서비스 아키텍처를 성공적으로 도입했으니, 이번의 작은 프로젝트에도 동일하게 적용하는 것이 좋겠다"고 섣불리 판단하는 것이다. 이는 특정 디자인 패턴이나 기술에 대한 비판적 검토 없이, 과거의 성공 경험을 맹신하여 현재 맥락에 맞지 않는 과잉 설계를 유발하는 주요 원인이 된다.26
- **기준점 편향 (Anchoring Bias):** 처음 접한 정보(기준점, anchor)가 이후의 모든 판단에 부적절한 영향을 미치는 현상이다.29 프로젝트 초기에 제시된 거창한 아키텍처 청사진이나 방대한 기능 목록이 기준점으로 작용하면, 팀은 실제 사용자의 필요와 무관하게 그 초기 계획을 실현하는 데 매몰될 수 있다. "초기 회의에서 나온 '모든 것을 플러그인화하자'는 아이디어"에 고착되어, 간단한 기능조차 불필요하게 복잡한 플러그인 구조로 개발하는 경우가 이에 해당한다.27

이러한 인지 편향들이 어떻게 과잉 엔지니어링으로 이어지는지를 다음 표에 요약하였다.

**표 2: 과잉 엔지니어링을 유발하는 인지 편향**

| 인지 편향                  | 정의                                            | 개발 과정에서의 발현 예시                                    | YAGNI 위반 결과                                              |
| -------------------------- | ----------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 과신 편향 (Overconfidence) | 자신의 능력이나 예측을 과대평가함               | "6개월 뒤에 필요할 기능을 나는 완벽하게 예측해서 지금 미리 만들 수 있다." | 불필요한 기능의 시기상조 구현 (Premature feature implementation) |
| 확증 편향 (Confirmation)   | 자신의 가설을 지지하는 정보만 선택적으로 수용함 | "이 추상화가 유용할 것이라는 내 생각에 부합하는 시나리오만 고려하고, 불필요하다는 의견은 무시한다." | 과잉 일반화 및 추상화 (Over-generalization)                  |
| 가용성 편향 (Availability) | 쉽게 떠올릴 수 있는 정보에 의존해 판단함        | "최근에 성공적으로 사용했던 복잡한 디자인 패턴을 이번의 간단한 문제에도 그대로 적용하자." | 불필요한 복잡성 도입 (Unnecessary complexity)                |
| 기준점 편향 (Anchoring)    | 초기 정보에 과도하게 의존함                     | "초기 아키텍처 회의에서 나온 '모든 것을 확장 가능하게'라는 원칙에 꽂혀서, 간단한 기능조차 과도하게 설계한다." | 과잉 설계 (Over-design)                                      |


YAGNI 원칙은 이러한 인지적 함정에 빠지지 않도록 돕는 강력한 심리적 안전장치 역할을 한다. YAGNI는 개발자의 의사결정 과정에서 '미래에 대한 주관적 예측'과 '불확실한 가정'이라는 변수가 개입될 여지를 근본적으로 최소화한다.

"이것이 지금 당장, 명백하게 필요한가?"라는 YAGNI의 핵심 질문은, 개발자의 판단 기준을 모호한 직관이나 과거 경험이 아닌, '현재 존재하는 구체적인 증거(요구사항, 사용자 데이터 등)'로 강제 전환시킨다.2 이는 과신 편향이나 확증 편향에 빠져 미래를 섣불리 재단하려는 충동을 억제한다. 또한, YAGNI는 제품 책임자나 동료가 제기하는 추측성 기능 요구에 대해 "아니오"라고 말할 수 있는 명확하고 합리적인 근거를 제공하며, 팀 전체가 현재 시점에서 가장 중요한 가치를 창출하는 데 집중하도록 돕는다.30


YAGNI는 고립된 원칙이 아니라, 소프트웨어 설계를 구성하는 다른 핵심 원칙들과 복잡한 상호작용을 통해 그 의미가 완성된다. 이 장에서는 YAGNI를 KISS, DRY, SOLID와 같은 주요 원칙들과의 관계 속에서 조명한다. 각 원칙과의 유사점, 차이점, 그리고 잠재적 충돌 지점을 분석함으로써, 개발자가 특정 상황에서 어떤 원칙을 우선해야 하는지에 대한 균형 잡힌 시각을 제공한다.


KISS(Keep It Simple, Stupid) 원칙은 주어진 문제를 해결하는 데 있어 불필요하게 복잡한 방법을 피하고 가장 단순한 해결책을 선택하라는 지침이다.32 YAGNI와 KISS는 모두 소프트웨어의 복잡성을 줄이고 단순함을 지향한다는 근본적인 목표를 공유한다.1

이 둘의 관계는 문제 해결의 두 가지 다른 차원에서 이해할 수 있다. YAGNI는 **'무엇을 만들지 않을 것인가'**를 결정함으로써 문제의 범위 자체를 단순화한다. 즉, 해결해야 할 문제의 개수를 줄이는 데 초점을 맞춘다. 반면, KISS는 **'만들기로 결정한 것을 어떻게 단순하게 만들 것인가'**에 대한 지침을 제공한다. 즉, 해결책의 구현 방식의 복잡도를 낮추는 데 집중한다. YAGNI가 적용되어 구현할 기능의 수가 줄어들면, 자연스럽게 시스템 전체의 구조가 단순해져 KISS 원칙을 지키기가 더 용이해진다. 요컨대, YAGNI는 기능의 부재(absence)를 통해, KISS는 구현의 간결함(simplicity)을 통해 시스템의 단순성을 추구한다.


DRY(Don't Repeat Yourself) 원칙은 시스템 내 모든 지식(코드, 로직, 데이터)은 단일하고, 명확하며, 권위 있는 표현을 가져야 한다는 원칙이다.32 이는 코드의 중복을 피하고 재사용성을 높여 유지보수성을 향상시키기 위해 종종 추상화(abstraction)를 권장한다.

바로 이 지점에서 YAGNI와 DRY 사이에 미묘한 긴장 관계가 발생한다. DRY를 준수하기 위한 추상화가 '아직 발생하지 않은 미래의 중복'을 제거하기 위한 목적일 경우, 이는 YAGNI 원칙과 정면으로 충돌할 수 있다. 이는 '시기상조의 추상화(Premature Abstraction)' 또는 '과잉 일반화(Over-generalization)'라는 대표적인 안티패턴으로 이어진다.

경험 많은 개발자들은 이 딜레마를 해결하기 위해 '세 번의 법칙(Rule of Three)'과 같은 실용적인 휴리스틱을 사용한다.6 이 법칙은 특정 코드 조각이나 로직이 세 번 이상 반복되기 전까지는 섣불리 추상화하지 말라는 지침이다. 이는 처음에는 YAGNI를 따라 약간의 중복을 허용하다가, 중복이 실질적인 문제로 부상하는 명확한 증거가 나타났을 때 DRY 원칙을 적용하여 리팩토링하는 균형 잡힌 접근법이다. 전략적으로 볼 때, YAGNI는 시스템을 구성하는 컴포넌트의 수를 줄여 복잡도를 낮추고, DRY는 시스템을 관리 가능한 여러 컴포넌트로 분할하여 각 부분의 복잡도를 낮추는 역할을 한다.34


SOLID는 객체지향 설계의 다섯 가지 핵심 원칙(단일 책임 원칙, 개방-폐쇄 원칙, 리스코프 치환 원칙, 인터페이스 분리 원칙, 의존 역전 원칙)을 집대성한 것으로, 유지보수 가능하고 유연하며 확장성 있는 소프트웨어를 설계하는 **'방법(How)'**에 대한 구체적인 지침이다.34

YAGNI와 SOLID 원칙 사이에서 가장 흥미롭고 중요한 긴장 관계는 OCP(Open-Closed Principle, 개방-폐쇄 원칙)와의 관계에서 나타난다. OCP는 "소프트웨어 개체(클래스, 모듈 등)는 확장에 대해서는 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다"고 정의한다.34 이를 실현하기 위해서는 미래의 변경 가능성을 예측하고, 인터페이스나 추상 클래스를 통해 변경이 발생할 지점을 격리하는 설계가 필요하다.36 이는 "미래를 예측하여 설계하지 말라"는 YAGNI의 핵심 메시지와 표면적으로 충돌하는 것처럼 보인다.36

이 명백해 보이는 갈등은 두 원칙이 다루는 문제의 차원을 명확히 구분함으로써 해소될 수 있다. YAGNI는 주로 **'무엇을(What)'** 만들 것인가, 즉 기능의 범위(feature scope)에 대한 의사결정 원칙이다. 반면, SOLID는 일단 만들기로 결정한 그 기능을 **'어떻게(How)'** 구현할 것인가, 즉 설계의 품질(design quality)에 대한 원칙이다.38

예를 들어, '사용자 데이터를 CSV 파일로 내보내는' 기능 요구사항이 있다고 가정해보자.

- **YAGNI의 적용 (What):** "나중에 PDF나 Excel 내보내기 기능도 필요할지 모르니, 지금 범용 데이터 Export 프레임워크를 만들자"는 유혹을 뿌리치고, 오직 현재 요구된 CSV 내보내기 기능만을 구현 범위로 한정한다.
- **SOLID/OCP의 적용 (How):** 이렇게 결정된 CSV 내보내기 기능을 구현할 때, 향후 다른 포맷이 추가될 *가능성*을 염두에 두고 좋은 설계를 적용한다. 예를 들어, `IExporter`라는 인터페이스를 정의하고 `CsvExporter`라는 구체적인 구현체를 만드는 방식이다. 이는 현재의 CSV 내보내기 기능 구현을 불필요하게 복잡하게 만들지 않으면서도, 나중에 `PdfExporter`를 추가해야 할 때 기존 코드를 수정하지 않고 새로운 클래스를 추가하는 것만으로 확장이 가능하게 한다.

마틴 파울러는 이 관계를 "YAGNI는 추측성 기능을 지원하기 위해 만들어진 기능에만 적용되며, 소프트웨어를 수정하기 쉽게 만들기 위한 노력(즉, 좋은 설계)에는 적용되지 않는다"고 명확히 설명했다.40 좋은 설계를 위한 적절한 추상화는 YAGNI 위반이 아니라, 오히려 YAGNI를 성공적으로 실천하기 위한 기반이 된다.

다음 표는 YAGNI를 포함한 주요 개발 원칙들의 핵심 목표와 관심사를 비교하여, 그들의 상호보완적인 관계를 명확히 보여준다.

**표 3: 주요 개발 원칙 비교 분석**

| 원칙      | 핵심 목표                      | 주요 관심사                                  | 적용 대상                                  |
| --------- | ------------------------------ | -------------------------------------------- | ------------------------------------------ |
| **YAGNI** | 낭비 제거, 가치 전달 속도 향상 | **What**: 무엇을 만들지 않을 것인가?         | 기능 범위 (Feature Scope)                  |
| **KISS**  | 이해 및 유지보수 용이성        | **How**: 어떻게 단순하게 만들 것인가?        | 구현 복잡도 (Implementation Complexity)    |
| **DRY**   | 지식의 단일 출처 보장          | **Where**: 정보/로직을 어디에 둘 것인가?     | 코드 중복 (Code Duplication)               |
| **SOLID** | 변경에 유연한 구조적 설계      | **How**: 어떻게 구조적으로 잘 설계할 것인가? | 의존성 및 결합도 (Dependencies & Coupling) |


이 장에서는 YAGNI 원칙을 실제 개발 현장에서 어떻게 적용할 수 있는지, 그리고 어떤 상황에서는 이 원칙을 의도적으로 위반해야 하는지에 대한 구체적이고 실용적인 지침을 제공한다. 이론을 넘어선 실천의 영역에서 YAGNI를 현명하게 사용하는 기술을 다룬다.


YAGNI 원칙의 적용 여부는 코드 수준에서 명확한 차이를 만들어낸다.

- YAGNI 위반 사례: 불필요한 필드 추가

  사용자 정보를 관리하는 Customer 클래스를 설계한다고 가정하자. 현재 요구사항에는 사용자의 성(LastName)과 이름(FirstName)만 필요하다. 그러나 개발자는 미래에 중간 이름(MiddleName)이나 별명(Alias)이 필요할 것이라고 예측하여 다음과 같이 클래스를 설계할 수 있다.41

  ```C#
  // YAGNI 위반 사례
  public class Customer
  {
      public int Id { get; set; }
      public string FirstName { get; set; }
      public string LastName { get; set; }
      public string MiddleName { get; set; } // YAGNI: Who needs this now?
      public string Alias { get; set; }      // YAGNI: Do we need this?
  }
  ```

  이러한 설계는 당장 사용되지 않는 필드를 추가함으로써 클래스와 데이터베이스 스키마를 불필요하게 복잡하게 만들고, 데이터 모델에 대한 오해를 유발하며, 장기적인 유지보수 비용을 증가시킨다.35

- YAGNI 준수 사례: 명확한 요구사항의 단순한 구현

  반면, YAGNI를 준수하는 개발자는 현재의 요구사항에만 집중한다.

  ```C#
  // YAGNI 준수 사례
  public class Customer
  {
      public int Id { get; set; }
      public string FirstName { get; set; }
      public string LastName { get; set; }
  }
  ```

  이 접근법은 코드를 간결하고 이해하기 쉽게 유지한다. 만약 미래에 중간 이름이 정말로 필요해진다면, 그 시점에 필드를 추가하는 것이 훨씬 경제적이고 정확하다. 이처럼 YAGNI는 '가장 단순한 것이 작동한다'는 원칙을 코드 수준에서 구체화한다. 다만, 이러한 접근법이 성공적으로 작동하기 위해서는 요구사항 변경 시 코드를 쉽게 수정하고 확장할 수 있는 지속적인 리팩토링 문화가 반드시 전제되어야 한다.5


YAGNI는 만병통치약이 아니며, 맹목적으로 적용할 경우 오히려 장기적인 기술 부채를 야기할 수 있다. 특히 한번 결정하면 되돌리기 어렵거나 변경 비용이 매우 높은 영역에서는 YAGNI 원칙을 신중하게 적용하거나 의도적으로 배제해야 한다.

- **API 및 프레임워크/라이브러리 설계:** 조직 내부에서 사용하는 코드와 달리, 외부에 공개되어 여러 클라이언트에 의해 사용되는 API나 라이브러리는 하위 호환성을 유지하는 것이 매우 중요하다. 한번 배포된 공개 API를 변경하는 것(Breaking Change)은 막대한 파급 효과를 낳기 때문에, 초기 설계 단계에서 잠재적인 사용 사례를 신중하게 예측하고 확장성을 충분히 고려해야 한다.11 이 영역에서는 YAGNI보다 미래를 대비하는 설계가 더 높은 우선순위를 가질 수 있다.
- **핵심 아키텍처 및 인프라 결정:** 시스템의 근간을 이루는 아키텍처 패턴(예: 모놀리식 vs. 마이크로서비스)이나 핵심 인프라(예: 데이터베이스 선택, 로깅/모니터링 시스템 구축, 네트워크 구성)에 대한 결정은 나중에 변경하기가 극히 어렵고 비용이 많이 든다. 예를 들어, 초기 트래픽이 적다고 해서 로깅 시스템 구축을 완전히 배제하는 것은 YAGNI의 오용이다. 이러한 근본적인 결정들은 장기적인 관점에서 신중하게 이루어져야 한다.43
- **비기능적 요구사항 (Non-Functional Requirements, NFRs):** 보안, 법규 준수(Compliance), 안정성, 확장성, 접근성과 같은 비기능적 요구사항은 대부분 '나중에 추가'하는 것이 거의 불가능하다. 보안 취약점은 시스템 설계 초기부터 고려되어야 하며, 특정 산업(금융, 의료 등)의 규제 준수는 아키텍처 자체에 깊숙이 통합되어야 한다. 이러한 명백하고 필수적인 요구사항들은 YAGNI의 적용 대상이 아니다.19


YAGNI와 추상화에 대한 접근 방식은 개발자의 경험 수준에 따라 뚜렷한 차이를 보인다.6

- **주니어 개발자:** 원칙의 존재를 모르거나 무시하고, 당장의 문제 해결에만 집중하여 확장성이 부족한 코드를 작성하는 경향이 있다.
- **미드레벨 개발자:** 다양한 디자인 패턴과 추상화 기법을 학습하면서, 이를 실제 필요 이상으로 과도하게 적용하려는 경향(Over-engineering)이 강하게 나타난다. 이 단계의 개발자들이 YAGNI 원칙을 가장 빈번하게 위반할 수 있다.
- **시니어 개발자/아키텍트:** 수많은 프로젝트를 통해 시기상조의 최적화와 불필요한 추상화가 얼마나 큰 비용을 초래하는지 경험으로 체득했기 때문에, 다시 의식적으로 단순한 접근법으로 회귀한다. 이들은 YAGNI를 '모든 추상화를 반대하는 원칙'이 아니라 '불필요한 추상화를 반대하는 원칙'으로 정확히 이해한다.6 이들의 전문성은 '언제 추상화가 현재의 복잡성 증가 비용보다 미래의 유연성 확보 이득이 더 클지'를 분별하는 직관과 경험에 있다.

결국 YAGNI의 현명한 적용 여부는 '변경 비용 곡선(Cost of Change Curve)'에 대한 깊은 이해와 판단에 달려있다. YAGNI의 근본적인 가정은 "나중에 기능을 추가하는 비용이 지금 추가하는 비용과 비슷하거나, 오히려 더 저렴하다(요구사항이 명확해지므로)"는 것이다.40 이 가정이 성립하려면, 소프트웨어의 변경 비용 곡선이 시간이 지나도 가파르게 상승하지 않고 완만하게 유지되어야 한다.46 애자일 방법론이 강조하는 지속적인 리팩토링, TDD, CI/CD 파이프라인 등은 바로 이 곡선을 의도적으로 평탄하게 유지하기 위한 공학적 실천들이다.5 이러한 환경에서는 YAGNI를 적극적으로 적용하는 것이 매우 합리적이다. 그러나 API 설계나 핵심 아키텍처 결정처럼 변경 비용 곡선이 매우 가파른 영역에서는, YAGNI를 적용하는 대신 신중한 예측과 선제적인 설계가 필요하다. 숙련된 개발자의 진정한 역량은 특정 결정이 이 두 가지 곡선 중 어디에 위치하는지를 정확히 판단하는 능력에서 나온다.


YAGNI(You Aren't Gonna Need It)는 소프트웨어 개발의 복잡성을 관리하기 위한 강력한 원칙이지만, 그 본질을 이해하지 못하면 오용되기 쉽다. 본 보고서는 YAGNI를 경제학, 인지심리학, 소프트웨어 설계론의 다각적인 관점에서 분석함으로써, 이 원칙이 단순한 '기능 삭제' 규칙을 넘어선 정교한 전략임을 보였다.

첫째, YAGNI는 경제적 합리성에 깊이 뿌리내리고 있다. 불필요한 기능을 선제적으로 구현하는 행위는 명시적인 구축 비용뿐만 아니라, 더 중요한 다른 기능의 출시를 막는 지연 비용, 시스템 전체의 복잡도를 높이는 유지 비용, 그리고 추측이 틀렸을 때 발생하는 수리 비용까지 초래한다. YAGNI는 이러한 보이지 않는 비용들을 최소화하고, 기회비용과 지연 비용을 고려하여 제한된 자원을 가장 가치 있는 곳에 집중시키는 경제적 의사결정 프레임워크이다.

둘째, YAGNI는 개발자의 인지적 한계에 대한 효과적인 방어기제이다. 과신 편향, 확증 편향, 가용성 편향 등 인간의 체계적인 사고 오류는 종종 과잉 엔지니어링으로 이어진다. YAGNI는 "지금 당장 필요한가?"라는 질문을 통해 개발자의 판단 근거를 불확실한 미래 예측에서 현재의 명확한 증거로 전환시킴으로써, 이러한 인지적 함정에 빠질 위험을 줄여준다.

셋째, YAGNI는 다른 핵심 설계 원칙들과의 관계 속에서 그 역할이 명확해진다. KISS와는 단순성이라는 목표를 공유하지만, YAGNI는 '무엇을'의 영역을, KISS는 '어떻게'의 영역을 다룬다. DRY와는 시기상조의 추상화라는 긴장 관계를 형성하며, '세 번의 법칙'과 같은 실용적 절충을 통해 조화를 이룬다. SOLID, 특히 OCP와는 '기능의 범위(What)'와 '설계의 품질(How)'이라는 서로 다른 차원의 문제를 다루며 상호 보완적으로 작용한다.

궁극적으로 YAGNI는 모든 상황에 맹목적으로 적용해야 할 절대 법칙이 아니라, 팀이 지속적으로 "이 작업이 지금 정말로 필요한가? 이것이 사용자에게 전달하는 가치는 무엇인가?"라는 근본적인 질문을 던지게 만드는 강력한 발견적 학습 도구(Heuristic)이다.

그러나 YAGNI의 성공적인 실천에는 중요한 전제 조건이 따른다. 이 원칙은 고립되어 존재할 수 없다. 그 가치를 온전히 발휘하기 위해서는 변경 비용 곡선을 완만하게 유지하려는 조직적 노력이 필수적이다. 즉, 지속적인 리팩토링 문화, 견고한 자동화 테스트 스위트, 그리고 점진적 설계에 대한 팀의 헌신이 반드시 뒷받침되어야 한다.5 이러한 공학적 안전망 없이는 YAGNI의 실천이 기술 부채(Technical Debt)를 방치하고 누적시키는 지름길로 변질될 수 있다.5

결론적으로 YAGNI의 최종 목표는 코드의 양을 줄이는 미니멀리즘 그 자체가 아니라, 불확실한 미래를 예측하는 데 드는 모든 형태의 낭비를 제거하고, 현재 고객에게 가장 큰 가치를 제공하는 일에 조직의 모든 역량을 집중함으로써 전체적인 가치 전달 흐름(Value Stream)을 최적화하는 것이다.31 이는 단순한 기술적 원칙을 넘어, 급변하는 시장 환경에서 비즈니스의 민첩성과 생존 가능성을 높이는 핵심적인 전략이라 할 수 있다.


1. What is YAGNI principle (You Aren't Gonna Need It)? - GeeksforGeeks, accessed August 16, 2025, https://www.geeksforgeeks.org/software-engineering/what-is-yagni-principle-you-arent-gonna-need-it/
2. Understanding the principle of software development with DRY, KISS, YAGNI, and the criticisms of SOLID | by ISHII (石井) - schimizu.com, accessed August 16, 2025, https://schimizu.com/understanding-the-principle-software-development-with-dry-kiss-yagni-and-the-criticisms-of-solid-036dbe28e649
3. ko.wikipedia.org, accessed August 16, 2025, [https://ko.wikipedia.org/wiki/YAGNI#:~:text=YAGNI(You%20aren't%20gonna,(XP)%EC%9D%98%20%EC%9B%90%EC%B9%99%EC%9D%B4%EB%8B%A4.](https://ko.wikipedia.org/wiki/YAGNI#:~:text=YAGNI(You aren't gonna,(XP)의 원칙이다.)
4. YAGNI - 위키백과, 우리 모두의 백과사전, accessed August 16, 2025, https://ko.wikipedia.org/wiki/YAGNI
5. You aren't gonna need it - Wikipedia, accessed August 16, 2025, [https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it](https://en.wikipedia.org/wiki/You_aren't_gonna_need_it)
6. "YAGNI" is a good principle, but many devs miss the point and conflate it with being anti-abstraction. - Reddit, accessed August 16, 2025, https://www.reddit.com/r/ExperiencedDevs/comments/11vonwg/yagni_is_a_good_principle_but_many_devs_miss_the/
7. Practicing YAGNI - Jason McCreary, accessed August 16, 2025, https://jasonmccreary.me/articles/practicing-yagni/
8. What Is YAGNI ("You Aren't Gonna Need It") Principle? | Built In, accessed August 16, 2025, https://builtin.com/software-engineering-perspectives/yagni
9. YAGNI: You Ain't Gonna Need It - Better Programming, accessed August 16, 2025, https://betterprogramming.pub/yagni-you-aint-gonna-need-it-f9a178cd8e1
10. Yagni - Martin Fowler, accessed August 16, 2025, https://martinfowler.com/bliki/Yagni.html
11. YAGNI revisited / Enterprise Craftsmanship, accessed August 16, 2025, https://enterprisecraftsmanship.com/posts/yagni-revisited/
12. 경제개념 | KDI 경제교육/정보센터, accessed August 16, 2025, [https://eiec.kdi.re.kr/material/conceptList.do?depth01=&idx=6](https://eiec.kdi.re.kr/material/conceptList.do?depth01&idx=6)
13. How to calculate opportunity cost (with examples) - Rho, accessed August 16, 2025, https://rho.co/blog/opportunity-cost-formula/
14. Opportunity Cost: Definition, Examples, and Applications | LaunchNotes, accessed August 16, 2025, https://www.launchnotes.com/glossary/opportunity-cost-in-product-management-and-operations
15. What Is the Cost of Delay? - Businessmap, accessed August 16, 2025, https://businessmap.io/lean-management/value-waste/cost-of-delay
16. What is the Cost of Delay framework? | Definition and Overview - ProductPlan, accessed August 16, 2025, https://www.productplan.com/glossary/cost-of-delay/
17. Cost of Delay - PMI, accessed August 16, 2025, https://www.pmi.org/disciplined-agile/what-is-the-economic-cost-of-delay-for-software-delivery
18. SAFe & Cost of Delay: a suggested improvement | Black Swan Farming, accessed August 16, 2025, https://blackswanfarming.com/safes-cost-of-delay-a-suggested-improvement/
19. Why YAGNI matters in Software Development and Architecture | HackerNoon, accessed August 16, 2025, https://hackernoon.com/why-yagni-matters-in-software-development-and-architecture
20. Keep Your Options Open: Extreme Programming and the Economics of Flexibility - To favaro.net, accessed August 16, 2025, https://www.favaro.net/john/home/publications/xpecon.pdf
21. Keep Your Options Open: Extreme Programming and the Economics of Flexibility, accessed August 16, 2025, https://nrc-publications.canada.ca/eng/view/accepted/?id=e690c7e2-793c-4ba2-adb4-869ba3105c64
22. 개발자 인지편향 - sookiwi, accessed August 16, 2025, https://sookiwi.com/posts/tech/2018/01/23/cognitive-bias-of-developers/
23. Cognitive Biases and Mitigation Strategies in Software Engineering - IJRAR, accessed August 16, 2025, https://www.ijrar.org/papers/IJRAR1944546.pdf
24. Cognitive Biases in Software Engineering: A Systematic Mapping ..., accessed August 16, 2025, https://www.researchgate.net/publication/318418537_Cognitive_Biases_in_Software_Engineering_A_Systematic_Mapping_Study
25. Cognitive Biases in Software Engineering: A Systematic Mapping Study - ResearchGate, accessed August 16, 2025, https://www.researchgate.net/publication/328410759_Cognitive_Biases_in_Software_Engineering_A_Systematic_Mapping_Study
26. A Tale from the Trenches: Cognitive Biases and Software ..., accessed August 16, 2025, https://web.engr.oregonstate.edu/~sarmaa/wp-content/uploads/2020/08/icse20-chattopadhyay.pdf
27. Mitigation of Cognitive Biases in Innovative Software Engineering Projects: A Proposal, accessed August 16, 2025, https://sol.sbc.org.br/index.php/cibse/article/download/35314/35104/
28. "보통(Normal)" 엔지니어는 훌륭한 팀의 핵심임 - GeekNews, accessed August 16, 2025, https://news.hada.io/topic?id=19754
29. 소프트웨어 개발과 인지 편향 현상(Cognitive Bias), accessed August 16, 2025, https://code-run.tistory.com/88
30. 코딩 중 자꾸 다른 것도 만들고 싶다면: YAGNI - Studying S, accessed August 16, 2025, https://studyingandsuccess.tistory.com/11
31. You Ain't Gonna Need It (YAGNI), Explained in Depth - Plutora, accessed August 16, 2025, https://www.plutora.com/blog/you-aint-gonna-need-it-yagni
32. OOP 개발 원칙 - YAGNI - 파란하늘의 지식창고, accessed August 16, 2025, https://luvstudy.tistory.com/102
33. Clean Code Essentials: YAGNI, KISS, DRY - DEV Community, accessed August 16, 2025, https://dev.to/juniourrau/clean-code-essentials-yagni-kiss-and-dry-in-software-engineering-4i3j
34. [Back-end] SOLID 원칙 (+DRY, YAGNI, KISS) - velog, accessed August 16, 2025, [https://velog.io/@ragi/Back-end-SOLID-%EC%9B%90%EC%B9%99-DRY-YAGNI-KISS](https://velog.io/@ragi/Back-end-SOLID-원칙-DRY-YAGNI-KISS)
35. YAGNI ("You Aren't Gonna Need It,"): helps software engineers build with clarity, not clutter | by Saurabh Gupta | Medium, accessed August 16, 2025, https://medium.com/@saurabh.engg.it/yagni-you-arent-gonna-need-it-helps-software-engineers-build-with-clarity-not-clutter-02464d0b7a63
36. How To Choose Between Conflicting Programming Principles - C# and Unity development, accessed August 16, 2025, https://giannisakritidis.com/blog/Dealing-With-Conflicting-Principles/
37. 3 software development principles I wish I knew earlier in my career, and the power of YAGNI, KISS, and DRY : r/programming - Reddit, accessed August 16, 2025, https://www.reddit.com/r/programming/comments/1bmicj0/3_software_development_principles_i_wish_i_knew/
38. SOLID vs. YAGNI [closed] - Stack Overflow, accessed August 16, 2025, https://stackoverflow.com/questions/3921904/solid-vs-yagni
39. design - SOLID principles vs YAGNI - Software Engineering Stack ..., accessed August 16, 2025, https://softwareengineering.stackexchange.com/questions/32618/solid-principles-vs-yagni
40. I'm probably a minority, but I really hate YAGNI. It is based on the assumpti... - DEV Community, accessed August 16, 2025, https://dev.to/idanarye/comment/al2f
41. C# YAGNI Principle (You Aren't Gonna Need It!) (2023) - ByteHide, accessed August 16, 2025, https://www.bytehide.com/blog/yagni-principle-csharp
42. Opposite of YAGNI - Applied Mathematics Consulting, accessed August 16, 2025, https://www.johndcook.com/blog/2012/02/29/opposite-of-yagni/
43. YAGNI (You Aren't Gonna Need It): A Key Principle in Software Development, accessed August 16, 2025, https://dev.to/alisamir/yagni-you-arent-gonna-need-it-a-key-principle-in-software-development-167m
44. Practicing YAGNI - DEV Community, accessed August 16, 2025, https://dev.to/gonedark/practicing-yagni-3n1d
45. accessed January 1, 1970, https://www.reddit.com/r/ExperiencedDevs/comments/11vonwg/yagni_is_a_good_principle_but_many_devs_miss_the_point_and_conflate_it_with_being_antiabstraction/
46. Developers on teams that consistently ignored tech debt, what was the ultimate outcome? : r/ExperiencedDevs - Reddit, accessed August 16, 2025, https://www.reddit.com/r/ExperiencedDevs/comments/1c2c06b/developers_on_teams_that_consistently_ignored/
47. Role of Automation in Reducing Software Refactoring Costs - Carnegie Mellon University, accessed August 16, 2025, https://insights.sei.cmu.edu/library/role-of-automation-in-reducing-software-refactoring-costs/
48. A Case Study on the Impact of Technical Debt Management Efforts on Code Quality - UTEP, accessed August 16, 2025, https://www.cs.utep.edu/vladik/utepnmsu19pinto.pdf
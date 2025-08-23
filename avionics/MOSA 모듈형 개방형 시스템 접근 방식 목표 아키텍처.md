# MOSA 모듈형 개방형 시스템 접근 방식 목표 아키텍처


본 보고서는 모듈형 개방형 시스템 접근 방식(Modular Open Systems Approach, MOSA)을 심층적으로 분석하여, 미국 국방법(U.S. law)과 국방부(DoD) 정책에 의해 의무화된 이 패러다임 전환적 전략의 핵심을 탐구한다. MOSA는 단순히 기술적 선호도를 넘어, 빠르게 진화하는 위협에 대응하고, 전력화 주기를 단축하며, 무기체계의 총수명주기비용을 통제하기 위한 통합된 비즈니스 및 기술 전략이다. 이 보고서는 MOSA의 핵심 원칙, 이를 뒷받침하는 표준 생태계, 그리고 장기적 이점과 단기적 비용 사이의 긴장 관계에서 비롯되는 중대한 실행 장벽을 상세히 기술한다. 특히, 지적재산권(IP) 전략의 결정적 역할과 MOSA가 디지털 엔지니어링 및 인공지능(AI) 기술 통합의 핵심 기반으로서 갖는 미래 가치를 조명한다. 결론적으로, MOSA의 성공적인 구현은 문화적, 조직적, 재정적 장벽을 극복하기 위한 정부와 산업계 전반의 일관되고 강력한 의지에 달려 있음을 명확히 한다.


이 파트에서는 MOSA의 근본적인 '무엇'과 '왜'를 규명하며, 이를 단순한 기술적 선택이 아닌 현대전과 국방 획득을 위한 법적으로 의무화된 전략적 필수 요소로 자리매김한다.


MOSA 도입을 추동하는 전략적 배경에는 기술과 위협의 급격한 진화가 자리 잡고 있다. 이러한 환경은 기존의 단일체(monolithic), 독점적 개발 모델로는 더 이상 감당할 수 없는 빠른 전력화 및 개량 주기를 요구한다.1 2019년 미 육해공군이 공동으로 발표한 "우리 무기체계를 위한 모듈형 개방형 시스템 접근 방식은 전력 우위를 위한 필수 과제(Modular Open Systems Approaches for our Weapon Systems is a Warfighting Imperative)"라는 제목의 메모는 이러한 최고위급의 의지를 명확히 보여준다. 이 메모는 미래 분쟁에서의 승리가 정보를 신속하게 공유하고 시스템을 효과적으로 적응시키는 능력에 달려있다고 강조한다.2 이는 전통적인 국방 획득 방식이 직면한 근본적인 문제, 즉 '공급자 종속(vendor lock)' 현상을 해결하기 위한 것이다. 과거에는 단일 계약업체가 특정 무기체계의 후속 군수지원과 성능개량을 독점함으로써 경쟁과 혁신이 제한되고, 최신 기술을 통합하는 데 큰 장벽으로 작용했다.3

MOSA로의 전환은 가상 적국에 대한 기술적 우위 상실에 대한 직접적인 대응으로 해석될 수 있다. 이는 폐쇄형 시스템을 개발하는 데 수십 년이 걸리는 전통적인 획득 주기가 신속하고 소프트웨어 중심적인 현대전 환경에서는 더 이상 유효하지 않다는 점을 인정한 것이다. 따라서 MOSA는 단순한 효율성 개선 이니셔티브가 아니라, 경쟁 우위를 회복하고 유지하기 위한 국가 안보 전략의 핵심 구성 요소이다. 이는 '역량'의 정의를 정적인 플랫폼에서 적응 가능하고 진화하는 시스템으로 재정의하는 전략적 전환을 의미한다.


MOSA는 공식적으로 "통합된 비즈니스 및 기술 전략(integrated business and technical strategy)"으로 정의된다.1 이 전략은 미국법 타이틀 10 U.S.C. 4401(b) (이전 2446a)에 명시된 법적 요구사항에 근거한다. 이 법률은 모든 주요 국방 획득 프로그램(MDAP)이 모듈식 설계를 채택하고, 주요 인터페이스에 대해 검증 가능한 표준을 사용하며, 분리 가능한 구성요소를 허용하는 MOSA를 의무적으로 사용하도록 규정한다.2 이러한 요구사항의 궁극적인 목표는 경쟁과 혁신의 기회를 확대하는 것이다.

여기서 '비즈니스' 측면은 기술적 설계만큼이나 중요하다. MOSA는 투명성, 협력, 그리고 위험 공유를 장려하는 "개방형 비즈니스 모델(open business model)"을 필수적으로 요구한다.1 기술적으로 완벽한 모듈형 아키텍처라 할지라도, 계약 메커니즘, 지적재산권(IP) 전략, 그리고 비즈니스 인센티브가 이를 뒷받침하지 못하면 실패할 수밖에 없다. 이러한 '비즈니스'와 '기술'의 이중적 성격은 MOSA의 가장 핵심적인 특징이자 가장 어려운 과제이다. 이는 프로그램의 성공이 단순히 엔지니어링 지표만으로 측정될 수 없으며, 경쟁 심화나 방위 산업 기반 다각화와 같은 비즈니스 성과로도 평가되어야 함을 시사한다. 따라서 MOSA 실행 계획은 그에 상응하는 상세한 비즈니스 및 IP 계획과 통합되어야 하며, 이 둘을 통합된 전체로 다루지 못하는 것이 실행 단계에서 발생하는 여러 문제의 근본 원인이다.


미 국방부는 MOSA의 성공적인 실행을 위해 5가지 기본 원칙을 제시했다. 이 원칙들은 단순한 점검 목록이 아니라, 지속적이고 반복적인 순환 과정의 일부로 이해되어야 한다.2

1. **조성 환경 구축(Establish an Enabling Environment):** 프로그램 관리자(PM)는 MOSA 개발을 지원하는 요구사항, 비즈니스 관행, 그리고 기술 개발, 획득, 시험평가 및 제품 지원 전략을 수립하여 우호적인 환경을 조성할 책임이 있다.10
2. **모듈식 설계 적용(Employ Modular Design):** 시스템은 응집성(cohesive), 캡슐화(encapsulated), 자립성(self-contained), 그리고 고도로 분류된(highly binned) 모듈로 설계되어야 한다.10
3. **주요 인터페이스 지정(Designate Key Interfaces):** 모듈 간의 상호작용을 정의하는 핵심 인터페이스를 전략적으로 식별하고 관리하는 것이 중요하다.10
4. **개방형 표준 사용(Use Open Standards):** 지정된 주요 인터페이스에는 널리 지원되고 합의에 기반한 개방형 표준을 적용해야 한다.10
5. **적합성 인증(Certify Conformance):** 개발된 인터페이스와 모듈이 선택된 표준을 준수하는지 검증하고 확인하는 절차가 필수적이다.10

이 5대 원칙은 본질적으로 애자일(Agile) 및 반복적 개발 방법론과 맞닿아 있다. 예를 들어, '적합성 인증' 단계에서 얻은 피드백은 다음 시스템 증분(increment)을 위한 '조성 환경 구축'과 '주요 인터페이스 지정' 과정을 개선하는 데 활용된다. 즉, 프로그램은 이 원칙들을 한 번만 수행하고 끝내는 것이 아니라, 환경 조성, 설계, 인증의 순환 주기를 반복하며 시스템을 점진적으로 발전시킨다. 이러한 순환적 특성은 본 보고서의 Part VII에서 논의될 애자일 및 데브섹옵스(DevSecOps) 개념과 직접적으로 연결된다.


미 국방부는 MOSA를 통해 5가지 주요 이점을 달성하고자 하며, 이는 각 프로그램의 MOSA 실행을 위한 상위 목표 역할을 한다.1

1. **경쟁 강화(Enhance Competition):** 분리 가능한 모듈을 통해 구성요소를 개별적으로 경쟁시켜 공급자 종속을 타파한다.1
2. **기술 진부화 해소 촉진(Facilitate Technology Refresh):** 전체 시스템을 재설계하지 않고도 새로운 기술을 신속하게 도입하거나 교체할 수 있다.1
3. **혁신 통합(Incorporate Innovation):** 새로운 위협과 작전 요구에 신속하게 대응하기 위해 자산을 재구성할 수 있는 유연성을 확보한다.1
4. **비용 절감/회피(Enable Cost Savings/Cost Avoidance):** 수명주기 전반에 걸쳐 모듈과 구성요소를 재사용하여 비용을 절감한다.1
5. **상호운용성 향상(Improve Interoperability):** 하드웨어 및 소프트웨어 모듈을 독립적으로 변경할 수 있게 하여 체계 간 통합을 개선한다.1

이 5가지 이점은 종종 서로 상충 관계에 있어 프로그램이 전략적인 절충을 해야 한다. 예를 들어, '경쟁 강화'를 위해 수많은 소형 모듈을 만들면 통합 복잡성이 증가하여 단기적으로 '비용 절감'에 해가 될 수 있다. 반대로, 단일 구성요소의 광범위한 재사용을 통해 '비용 절감'을 극대화하면 새로운 공급업체로부터의 '혁신 통합' 기회가 제한될 수 있다. 따라서 성공적인 MOSA 전략은 단순히 이점들을 나열하는 것을 넘어, 특정 프로그램의 임무, 기술 환경, 수명주기 단계를 고려하여 이점들의 우선순위를 명확히 정의하고 정당화해야 한다. 미 회계감사원(GAO)이 지적했듯이, 프로그램들이 공식적인 비용-편익 분석을 수행하지 않는다는 사실은 이러한 절충이 명시적으로나 엄격하게 이루어지지 않고 있음을 방증한다.4


이 파트는 모듈화라는 추상적인 개념에서 벗어나, 시스템 아키텍처 수준에서 모듈화가 어떻게 작동하는지 설명하기 위해 기존의 접근 방식과 MOSA 패러다임을 대조하며 핵심적인 기술적 기반을 제공한다.


이 섹션에서는 단일체(monolithic) 아키텍처와 모듈식(modular) 아키텍처를 상세히 비교 분석한다.

- **단일체 아키텍처:** 모든 구성요소가 상호 의존적인 단일하고 긴밀하게 결합된 단위로 정의된다.14 이러한 시스템의 수명주기는 느리고 묶음 형태의 개발, 전체 코드베이스에 대한 복잡한 테스트, 그리고 어렵고 위험 부담이 큰 배포 과정을 특징으로 한다.17 초기 개발은 단순할 수 있지만, 시간이 지남에 따라 확장성과 유지보수가 심각한 문제로 대두된다.15
- **모듈식 아키텍처 (MOSA):** 잘 정의된 인터페이스를 가진 독립적이고 느슨하게 결합된 모듈들의 집합으로 정의된다.14 이러한 시스템의 수명주기는 병렬 개발, 모듈의 독립적인 테스트, 그리고 더 빠르고, 빈번하며, 위험이 적은 배포를 가능하게 한다.14 이는 유연성, 확장성, 코드 재사용을 촉진하지만, 모듈 간 통신 및 분산 시스템 관리의 복잡성을 야기할 수 있다.15

이 두 아키텍처 사이의 선택은 단순한 기술적 결정이 아니라, 프로그램이 추구하는 *속도*에 대한 근본적인 선택이다. 단일체 아키텍처는 단순한 프로젝트의 초기 개발 속도를 최적화한다. 반면, 모듈식 아키텍처는 장기적인 *적응성과 진화 속도*를 최적화한다. 수십 년의 수명주기를 가지며 급변하는 위협에 직면하는 국방 시스템에게는, 비록 초기에 더 많은 아키텍처적 규율이 요구되더라도 모듈식 접근 방식이 유일하게 논리적인 선택이다. 즉, MOSA는 국방부가 초기 일회성 개발 속도보다 지속적인 민첩성에 전략적으로 투자하고 있음을 보여준다.


이 섹션에서는 잘 구조화된 모듈식 시스템을 정의하는 구체적인 기술적 속성들을 심층적으로 다룬다.

- **높은 응집성(High Cohesion):** 모듈은 집중되고 잘 정의된 기능을 포함해야 한다.1 즉, 하나의 모듈은 하나의 기능을 잘 수행해야 한다.
- **느슨한 결합(Loose Coupling):** 모듈은 독립적이어야 하며 다른 모듈을 제약해서는 안 된다.1 한 모듈의 변경이 다른 모듈에 미치는 영향을 최소화해야 한다. 이는 단일체 시스템의 '긴밀한 결합'과 정반대되는 개념이다.15
- **캡슐화(Encapsulation):** 모듈은 자신의 내부 작동 방식(행위 및 데이터)을 다른 모듈로부터 숨기고, 오직 잘 정의된 인터페이스를 통해서만 기능을 노출해야 한다.10
- **분리 가능성(Severable):** 수명주기 동안 모듈을 물리적으로나 논리적으로 추가, 제거 또는 교체할 수 있는 능력을 의미한다.1

이러한 기술적 특성들은 MOSA의 비즈니스 사례를 뒷받침하는 기반이다. 모듈이 느슨하게 결합되고 캡슐화되어 있기 때문에 다른 공급업체에 의해 개발될 수 있다. 또한, 응집성이 높고 분리 가능하기 때문에 독립적으로 경쟁하고 업그레이드할 수 있다. 이러한 기술적 속성이 없다면, 경쟁 강화, 기술 진부화 해소 등 MOSA의 5대 핵심 이점은 아키텍처적으로 달성 불가능하다. 즉, 아키텍처 규율의 실패는 비즈니스 목표 달성의 실패로 직접 이어진다.


이 섹션에서는 MOSA가 시스템 엔지니어링의 초점을 어떻게 변화시키는지 논의한다. 시스템 엔지니어링의 역할은 완성된 정적 시스템을 설계하는 것에서, 진화를 가능하게 하는 *아키텍처*와 *인터페이스*를 설계하는 것으로 전환된다. 프로그램 관리자(PM)와 시스템 엔지니어는 시스템 엔지니어링 계획(SEP)의 일부로서 MOSA 실행 계획을 개발할 책임이 있다.6 여기에는 주요 인터페이스 지정, 표준 선정, 적합성 인증 계획 등이 포함된다.10 이는 주 계약업체에게 아키텍처 설계를 위임하는 것이 아니라, 정부가 아키텍처를 정의하는 데 있어 주도적인 역할을 할 것을 요구한다.19

이는 정부 시스템 엔지니어의 역할을 요구사항 관리자 및 감독자에서 진정한 시스템 아키텍트로 격상시킨다. 전통적인 모델에서 정부는 시스템이 '무엇을' 하는지 정의하고, 주 계약업체는 '어떻게' 할지를 결정했다. MOSA 모델에서는 정부가 시스템이 개방성을 유지하도록 '어떻게'의 상당 부분, 특히 아키텍처 프레임워크, 주요 인터페이스, 표준을 직접 정의해야 한다. 이는 정부 프로그램 사무소 내에 더 높은 수준의 기술 전문성을 요구하며, 기술적 의사결정에서 더욱 적극적인 자세를 필요로 한다.

| **속성**      | **단일체 아키텍처**                                          | **모듈식 아키텍처 (MOSA)**                                   |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **개발**      | 순차적, 통합된 코드베이스 개발. 초기에는 단순하나 점차 복잡해짐.15 | 독립 모듈의 병렬 개발 가능. 코드 재사용성 높음.14            |
| **배포**      | 전체 애플리케이션을 한 번에 배포. 배포 주기가 길고 위험 부담이 큼.18 | 모듈 단위의 독립적이고 빈번한 배포 가능. 배포 속도가 빠르고 위험이 적음.14 |
| **확장성**    | 전체 시스템을 복제해야 하므로 비효율적이고 어려움. 특정 기능만 확장하기 힘듦.17 | 필요한 모듈만 독립적으로 확장 가능. 높은 확장성과 유연성 제공.14 |
| **유지보수**  | 코드베이스가 커지면서 수정 및 버그 해결이 어려움. 변경의 파급 효과가 큼.15 | 모듈이 독립적이므로 수정 및 업데이트가 용이. 다른 모듈에 미치는 영향이 적음.14 |
| **유연성**    | 기술 스택이나 구조 변경이 어려워 경직됨.17                   | 새로운 기술이나 기능을 모듈 단위로 쉽게 추가, 교체, 제거할 수 있어 유연성이 높음.14 |
| **기술 채택** | 새로운 기술을 도입하려면 전체 시스템을 재구성해야 할 수 있어 느림.17 | 새로운 기술을 특정 모듈에 신속하게 적용하거나 새로운 기술 모듈을 통합하기 용이함.18 |
| **팀 구조**   | 대규모의 상호 의존적인 팀이 단일 코드베이스에서 작업.18      | 소규모의 독립적인 팀이 각자 맡은 모듈을 개발 및 관리.18      |


이 파트는 모듈화라는 추상적인 개념에서 벗어나, 현실 세계에서 이를 가능하게 하는 구체적인 표준들을 다루며, 이 '생태계'가 어떻게 작동하는지 설명한다.


MOSA는 단일 표준이 아니라, "조력 표준(enabling standards)"의 집합을 활용하는 접근 방식이다.6 이 표준들은 널리 인정받고, 합의에 기반하며, 공인된 표준화 기구나 시장에 의해 제정된 것들이다.6 미 국방부는 모듈형 개방형 시스템 워킹 그룹(MOSWG)과 그 산하의 타이거 팀을 통해 이러한 표준들을 식별, 개발, 그리고 장려하는 역할을 수행한다.1 FACE, SOSA, OMS, VICTORY 등 복잡하게 얽힌 표준들은 MOSA 원칙을 특정 도메인에 맞게 구현한 사례들이다.13

미 국방부의 전략은 모든 표준을 처음부터 만드는 것이 아니라, 기존의 성숙한 정부 및 산업 표준 포트폴리오를 *선별*하고 *의무화*하는 것이다. 이는 더 넓은 시장의 혁신을 활용하는 실용적인 접근법이다. 그러나 이는 동시에 중대한 거버넌스 과제를 낳는다. 즉, 이질적인 표준들이 상호 보완적으로 유지되도록 하고, 중복을 피하며, 그 진화를 조화롭게 관리해야 하는 것이다. MOSWG는 바로 이러한 거버넌스 과제에 대한 조직적 해답이다. 국방부는 표준 제정자라기보다는 '표준 수용자' 및 '표준 통합자'의 역할을 수행하며, 기술 생태계에서 최적의 도구를 선택한 후 국방 목적에 맞게 통합하는 프레임워크를 구축하고 있다.


미래 공중 능력 환경(Future Airborne Capability Environment, FACE)은 군용 항공 솔루션을 위한 소프트웨어 표준이자 비즈니스 전략이다.22 그 목표는 소프트웨어의 이식성(portability)과 재사용성을 높여 개발 시간과 비용을 절감하는 것이다.12 FACE 참조 아키텍처는 운영체제 세그먼트(OSS), 이식 가능 컴포넌트 세그먼트(PCS), 전송 서비스 세그먼트(TSS), 플랫폼 특정 서비스 세그먼트(PSSS), 입출력 서비스 세그먼트(IOSS)의 5개 세그먼트로 구성되며, 공통 운영 환경의 개념을 기반으로 한다.23 이 표준은 정부, 산업계, 학계가 참여하는 오픈 그룹 FACE 컨소시엄에 의해 관리되며 12, 미 3군 합동 메모에서 의무화된 MOSA 표준 중 하나이다.22

FACE는 항공전자 분야에서 하드웨어 중심적 사고에서 소프트웨어 중심적 사고로의 중대한 전환을 의미한다. 표준화된 소프트웨어 아키텍처를 만듦으로써, FACE는 소프트웨어(즉, 능력)를 그것이 실행되는 특정 하드웨어로부터 효과적으로 분리시킨다. 이것이 바로 이식성의 핵심이며, 새로운 항공 컴퓨터가 나올 때마다 소프트웨어를 다시 작성해야 했던 악순환을 끊는 열쇠이다. 이는 소프트웨어를 일회성의 플랫폼 종속적 비용에서 재사용 가능한 국방 자산으로 전환시킨다. 이를 통해 한 항공기용으로 개발된 첨단 알고리즘을 최소한의 재작업으로 다른 항공기에 이식할 수 있게 되어, 국방부가 하드웨어 제조 속도가 아닌 소프트웨어 개발 속도로 새로운 능력을 전개할 수 있게 한다.


센서 개방형 시스템 아키텍처(Sensor Open Systems Architecture, SOSA) 표준은 C5ISR(지휘, 통제, 통신, 컴퓨터, 사이버, 정보, 감시, 정찰) 시스템을 위한 표준이자 비즈니스 전략으로, 하드웨어, 소프트웨어, 전기/기계 인터페이스를 포괄한다.25 그 목적은 여러 공급업체로부터 레이더, 전자전(EW), 신호 정보(SIGINT) 등 다양한 센서와 하위 시스템을 유연하게 획득할 수 있도록 하는 것이다.25 SOSA는 기존 표준들을 활용하고 프로파일링하는데, 특히 하드웨어 모듈, 백플레인, 섀시를 위해 VITA의 OpenVPX 표준을 광범위하게 참조한다.13 이 표준 역시 오픈 그룹 산하의 SOSA 컨소시엄에 의해 관리된다.25

SOSA는 MOSA 생태계의 물리적, 논리적 중추 역할을 한다. FACE가 공중 소프트웨어를 다루는 반면, SOSA는 많은 플랫폼의 '두뇌'에 해당하는 센서와 프로세서를 위한 하드웨어 및 저수준 소프트웨어 프레임워크를 제공한다. 하드웨어 프로파일(예: 슬롯 프로파일, 핀 구성)을 표준화함으로써, SOSA는 C5ISR 하드웨어 카드에 대한 진정한 '플러그 앤 플레이' 환경을 조성한다.26 이는 하드웨어 계층을 상품화(commoditize)하여, 경쟁의 가치를 해당 하드웨어에서 실행되는 소프트웨어와 알고리즘으로 이동시킨다. 즉, SOSA는 고성능 국방 전자장비를 위한 표준화된 '앱 스토어' 모델을 창출하고 있다. 섀시는 '스마트폰', SOSA 정렬 슬롯은 '포트' 역할을 하며, 공급업체들은 쉽게 통합될 수 있는 경쟁적인 플러그인 카드(PIC)를 만들 수 있다.


개방형 임무 시스템(Open Mission Systems, OMS) 표준은 주로 미 공군이 주도하는 정부 소유 아키텍처 사양이다.30 이는 소프트웨어 *서비스*와 하드웨어 *하위 시스템* 간의 인터페이스 및 데이터 교환에 중점을 둔다.30 범용 지휘통제 인터페이스(Universal Command and Control Interface, UCI)는 OMS 내에서 사용되는 핵심 메시지 형식이지만, OMS는 다른 데이터 교환 방식도 포함하는 더 넓은 개념이다.30 OMS는 '전부 아니면 전무' 방식이 아닌 계층적 적합성(tiered compliance) 개념을 제공하며, 레거시 시스템을 위한 어댑터 사용도 허용한다.30 그 목표는 논리적 '플러그 앤 토크(plug and talk)' 상호운용성을 구현하는 것이다.30

OMS/UCI는 SOSA나 FACE보다 더 높은 추상화 수준에서 작동한다. FACE가 소프트웨어 운영 환경을, SOSA가 하드웨어/소프트웨어 구성 요소를 정의한다면, OMS는 임무 서비스들이 서로 대화하는 데 사용하는 *언어* 또는 프로토콜을 정의한다. 이는 임무 지휘의 의미론(semantics), 즉 센서에 임무를 부여하고, 결과물을 수신하며, 상태를 보고하는 행위에 초점을 맞춘다. 이로 인해 OMS는 체계 간 상호운용성의 핵심 요소가 되며, 예를 들어 항공작전센터가 OMS 호환 항공기에 임무를 부여하고, 이 항공기는 다시 OMS 호환 센서에 임무를 부여하는 모든 과정이 표준화된 언어를 통해 이루어지게 한다.30


C5ISR/EW 모듈형 개방형 표준 스위트(C5ISR/EW Modular Open Suite of Standards, CMOSS)는 미 육군이 주도하는 이니셔티브로, FACE, SOSA/OpenVPX, VICTORY, MORA 등 여러 표준을 지상 차량용 공통 섀시로 통합하기 위한 것이다.2 주된 목표는 중복되는 '무전기 박스'들을 제거하고, 단일 섀시 내에서 플러그인 카드를 통해 기능을 공유함으로써 크기, 중량, 전력 및 비용(SWaP-C)을 절감하는 것이다.33 CMOSS 장착형 폼팩터(CMFF)가 공식적인 기록 프로그램(program of record)으로 지정된 것은 이 능력을 신속하게 전력화하려는 육군의 강력한 의지를 보여주는 중요한 이정표이다.34

CMOSS는 특정하고 시급한 문제, 즉 차량 탑재 장비의 과도한 SWaP 부담을 해결하기 위한 MOSA 구현의 가장 공격적이고 구체적인 사례이다. 공식 프로그램(PdM CMFF)을 만듦으로써, 육군은 산업계에 강력한 '수요 신호'를 보냈다. 이는 PNT, EW, 무선 파형 등을 위한 경쟁력 있고 상호운용 가능한 CMOSS 호환 카드 시장을 촉진시켰다.35 CMOSS는 정부가 명확하게 정의한 아키텍처가 어떻게 산업 기반을 성공적으로 형성하여 개방형 모듈식 솔루션을 제공하게 하는지를 보여주는 사례 연구이다.


이 섹션에서는 앞선 논의들을 종합하여, 이 표준들이 경쟁 관계가 아닌 상호 보완 관계임을 설명한다. 가상의 항공기 탑재 C5ISR 포드를 예로 들어 계층적 구조를 설명할 수 있다. 포드의 물리적 카드와 백플레인은 SOSA에 맞춰 설계될 수 있다. 이 카드들의 소프트웨어는 FACE 표준을 사용하여 분할될 수 있다. 그리고 포드 전체는 OMS/UCI 메시지를 사용하여 항공기의 임무 컴퓨터 및 다른 시스템과 통신할 수 있다. 이는 표준들이 시스템 스택의 여러 다른 계층에서 작동하여 포괄적인 개방형 아키텍처를 달성하는 방식을 보여준다.37

MOSA 표준 생태계는 국방 전체를 위한 다층적 참조 아키텍처를 형성한다. SOSA는 하드웨어/섀시 계층을, FACE는 소프트웨어 이식성/운영체제 계층을, OMS는 임무 애플리케이션/의미론적 계층을 제공한다. 이 계층적 모델을 이해하는 것은 시스템 아키텍트에게 매우 중요하다. 프로그램은 모든 표준을 채택하지 않고 일부(예: 소프트웨어 업그레이드를 위한 FACE)만 선택할 수도 있지만, 최대의 이점은 이 계층들을 함께 사용하여 실리콘 칩부터 임무 논리까지 완전히 개방된 시스템을 만들 때 달성된다.

| **표준**    | **주관 기관/군**                | **주요 도메인**           | **핵심 목표**                                                |
| ----------- | ------------------------------- | ------------------------- | ------------------------------------------------------------ |
| **FACE**    | The Open Group (정부/산업/학계) | 항공전자 소프트웨어       | 소프트웨어 이식성, 재사용성, 비용 절감 22                    |
| **SOSA**    | The Open Group (정부/산업/학계) | C5ISR 하드웨어/소프트웨어 | 센서 및 하위 시스템의 유연한 획득, 상호운용성 25             |
| **OMS/UCI** | 미 공군                         | 임무 시스템 C2            | 서비스와 하위 시스템 간의 표준화된 데이터 교환, 논리적 상호운용성 30 |
| **CMOSS**   | 미 육군                         | 육군 지상/공중 플랫폼     | SWaP-C 절감, 여러 표준을 공통 섀시로 통합 2                  |


이 파트는 MOSA의 '무엇'과 '어떻게'에서 '왜 그렇게 어려운가'로 초점을 전환하여, GAO 보고서 및 기타 비판적 분석을 바탕으로 MOSA 채택을 방해하는 도전 과제들을 증거 기반으로 분석한다.


이 섹션은 미 회계감사원(GAO)의 주요 조사 결과를 중심으로 도전 과제들을 소개한다. 법적 의무에도 불구하고 MOSA의 실행이 일관되지 않다는 점을 먼저 지적한다.4 많은 프로그램이 일부 MOSA를 구현했다고 보고하지만, 비용 및 일정과 같은 장벽을 이유로 전략을 완전히 수용하는 경우는 드물다.4 이 섹션에서는 GAO가 제시하고 국방부가 동의한 14개의 권고안을 이 파트에서 논의할 문제들의 로드맵으로 제시한다.38

GAO 보고서는 MOSA 이니셔티브에 대한 체계적인 감사를 통해 정책적 의도와 프로그램 실행 사이의 심각한 격차를 드러낸다. 국방부가 모든 권고안에 동의했다는 사실은 이러한 문제들이 현실적이고 체계적이라는 것을 암묵적으로 인정한 것이다. 여기서 부상하는 핵심 주제는 MOSA가 예산이 책정되지 않은 의무 사항이나 단순한 점검 목록 항목으로 성공적으로 구현될 수 없으며, 국방부의 기획, 예산 편성 및 감독 프로세스에 근본적인 변화가 필요하다는 점이다.


이 섹션은 GAO가 지적한 가장 큰 장벽, 즉 공식적인 비용-편익 분석의 부재와 그로 인한 단기 비용에 대한 집중 문제를 다룬다. 조사 대상 프로그램 중 어느 곳도 공식적인 분석을 수행하지 않았는데, 이는 국방부 정책이 이를 명시적으로 요구하지 않기 때문이다.4 이로 인해 단기 획득 성과로 평가받는 프로그램 담당자들은 장기적인 후속 군수지원 비용 절감과 같은 MOSA의 이점을 간과하게 된다.4 예를 들어, B-52 레이더 현대화 프로그램이 인지된 위험 때문에 MOSA 요소를 포기한 사례는 이러한 행태를 잘 보여준다.38 또한, MOSA 비용이 초기에 집중된다는 경제적 현실도 논의된다.40

이는 MOSA 실행의 핵심적인 '악순환'이다. 프로그램들은 단기 비용을 최소화하도록 인센티브를 받는다. MOSA는 종종 장기적인 절약을 위해 단기적인 투자(기획, 인터페이스 설계, 데이터 권리 획득 등)를 필요로 한다. 이러한 장기적 이점을 정량화하고 가치를 평가하는 공식적이고 의무적인 프로세스가 없다면, 단기 비용이 논쟁에서 거의 항상 이기게 되어, 총수명주기 관점에서 차선책을 선택하게 된다. 이 순환을 끊기 위해서는 프로그램이 공식적인 비즈니스 사례 분석을 수행하도록 강제하는 정책 변경이 필요하며, 이를 통해 장기적인 이점을 예산 논의에서 가시적이고 방어 가능하게 만들어야 한다.


이 섹션에서는 기획 및 조정의 결함을 탐구한다. GAO는 대부분의 프로그램이 시스템 엔지니어링 계획(SEP)과 같은 획득 문서에서 모든 핵심 MOSA 기획 요소를 다루지 않았다고 지적했다.4 더 나아가, 프로그램 집행관(PEO) 수준에서 프로그램 간 MOSA 실행을 조정하기 위한 공식적인 프로세스가 부족하다는 점도 강조한다.4 이러한 파편화는 공통 부품 사용과 간소화된 설계를 통한 비용 절감 기회를 제한한다.38 표준 저장소나 디지털 도구와 같은 MOSA 조력자에 대한 전사적 수준의 투자가 핵심적인 누락 부분으로 논의된다.4

MOSA의 가장 큰 이점은 개별 프로그램 수준이 아닌 전사적 수준에서 실현된다. 재사용 가능한 소프트웨어 모듈이나 공통 하드웨어 카드의 가치는 다른 프로그램에서 재사용될 때마다 증폭된다. 그러나 프로그램은 개별적으로 관리되고 예산이 편성된다. 이는 구조적인 불일치를 야기한다. 이러한 교차 프로그램 기회를 식별, 자금 지원 및 관리할 PEO 또는 전사적 수준의 기관이 없다면, 이러한 기회는 결코 실현되지 않을 것이다. 각 프로그램은 자신의 이익에 따라 행동하며, 자체적인 고유 솔루션을 개발하여 공통성과 재사용의 목적을 무력화시킬 것이다.


이 섹션에서는 비기술적 장벽을 다룬다. MOSA가 독점 시스템과 공급자 종속적 후속 군수지원에 의존하는 전통적인 비즈니스 모델을 파괴하기 때문에, 일부 기존 산업계의 자연스러운 저항을 야기한다는 점을 논의한다.5 또한, 국방 획득 인력 내에서 요구되는 문화적 변화, 즉 규정 준수 중심의 문서 기반 사고방식에서 보다 기술적으로 적극적이고 아키텍처 중심적인 사고방식으로의 전환에 대해 다룬다.19 인력의 비즈니스 및 기술 전문성 개발의 어려움도 강조된다.4 성공적인 MOSA 노력을 '깨뜨리지' 않으면서 활용하는 것의 어려움과 규정 준수 지표 정의의 어려움도 다룬다.41

MOSA의 채택은 파괴적 혁신의 전형적인 사례이며, 전형적인 형태의 조직적 저항에 직면하고 있다. 산업계에게는 기존의 수익원을 위협한다. 정부 인력에게는 새로운 기술과 보다 적극적이고 위험을 감수하는 자세를 요구한다. 이를 극복하기 위해서는 정책 메모 이상의 것이 필요하다. 즉, 정부와 산업계 모두에게 개방적인 행동을 보상하는 새로운 인센티브를 창출하고, 소통, 훈련을 포함하는 조정된 변화 관리 전략이 필요하다.

| **장벽 범주**       | **구체적 GAO 조사 결과**                                     | **시사점**                                                   | **주요 GAO 권고안**                                          |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **비용-편익 분석**  | 공식적인 비용-편익 분석이 수행되지 않음.4                    | 단기 비용을 우선시하여 장기적 이점을 간과함.39               | MOSA 비용과 편익을 분석하는 프로세스를 개발할 것.39          |
| **프로그램 기획**   | 대부분의 프로그램이 핵심 MOSA 기획 요소를 획득 문서에 반영하지 않음.4 | 프로그램이 MOSA의 잠재적 이점을 달성할 준비가 되어 있는지에 대한 가시성 부족. | 군별로 MOSA 기획 문서의 질을 보장하는 프로세스를 개선할 것.39 |
| **포트폴리오 조정** | 포트폴리오 전반에 걸쳐 MOSA를 조정하기 위한 공식적인 프로세스 부재.4 | 공통 부품 사용을 통한 비용 절감 및 업그레이드 가속화 기회 상실.39 | 프로그램 간 MOSA 실행 조정을 위한 프로세스를 개선할 것.39    |
| **자원 할당**       | 군별로 MOSA 구현에 필요한 투자를 완전히 식별하고 우선순위를 정하지 않음.4 | 프로그램이 MOSA의 잠재적 이점을 달성하기에 불충분한 자원과 전문성 위험. | 군별로 MOSA 요구사항을 평가하고 자원을 조정할 것.39          |
| **정책 및 지침**    | 중기획득(MTA) 경로 프로그램에 대한 MOSA 요구사항 적용 정책 공백.4 | MOSA 구현을 방해할 수 있는 정책 및 지침의 격차 존재.         | MOSA 정책 및 지침의 격차를 해소할 것.39                      |


이 파트는 통합 전략의 '비즈니스' 측면을 심층적으로 다루며, MOSA와 지적재산권(IP) 사이의 복잡하지만 중요한 관계에 초점을 맞춘다.


이 섹션에서는 좋은 IP 전략과 MOSA가 상호 보완적이라는 핵심 전제를 설정한다.42 IP는 "통합된 비즈니스 및 기술 전략"의 "비즈니스 측면"이다.42 정부 계약에서 IP의 기본 개념을 소개한다: 정부는 거의 

*소유권*을 취득하지 않고, 기술 데이터(TD)와 컴퓨터 소프트웨어(CS)에 대한 *사용 허가권*(데이터 권리)을 획득한다.43 목표는 산업계의 투자를 보호하는 것과 정부가 경쟁 및 후속 군수지원을 위해 필요한 권리를 확보하는 것 사이에서 "올바른 균형"을 찾는 것이다.44

MOSA와 IP의 관계는 공생적이다. 강력한 MOSA 기술 아키텍처(잘 정의된 모듈식 경계)는 세분화된 IP 전략을 구현하기 쉽게 만든다. 반대로, 잘 정의된 IP 전략(어떤 모듈에 어떤 데이터 권리가 필요한지 식별)은 기술 아키텍처를 알리고 추진한다. 이 둘은 순차적이 아니라 병렬적으로 개발되어야 한다. IP를 고려하지 않고 아키텍처를 설계하는 프로그램은 나중에 필요한 권리를 획득하는 것이 불가능하다는 것을 알게 될 것이며, 아키텍처를 이해하지 않고 IP 전략을 정의하는 프로그램은 필요하지 않은 권리를 획득하거나 중요한 권리를 놓치게 될 것이다.


이 섹션에서는 MOSA와 IP 전략이 협력하여 공급자 종속을 깨는 주요 메커니즘을 설명한다. 정부의 데이터 권리가 시스템의 가장 낮은 실행 가능한 수준에서 평가되는 "분리 가능성의 원칙(doctrine of segregability)"을 논의한다.42 잘 구현된 MOSA는 본질적으로 이러한 분리 가능한 모듈을 생성한다. 그러나 핵심은 단지 모듈을 갖는 것이 아니라, 그것들을 

*분리하고 재통합하는 데 필요한 데이터*에 대한 IP 권리를 획득하는 것이다.42 새로운 모듈을 "연결"하는 방법을 정의하는 이 데이터는 종종 미래 경쟁을 가능하게 하는 가장 중요한 IP 조각이다. "스위스 치즈 문제"는 MOSA 없이는 계약업체가 핵심 부품을 독점적으로 만들어 나머지 시스템의 데이터 권리를 무용지물로 만들 수 있는 방법을 보여주는 예로 사용된다.42

MOSA 생태계에서 가장 가치 있는 IP는 반드시 모듈의 내부 작동 방식이 아니라, *그 경계의 정의*이다. 공급업체는 정부가 경쟁사의 제품으로 해당 모듈을 교체할 수 있도록 하는 인터페이스 데이터에 대한 필요한 권리를 가지고 있는 한, 모듈 내부의 "비법 소스"를 독점적으로 유지할 수 있다. 이는 산업계는 핵심 경쟁 우위를 보호하고, 정부는 경쟁력 있고 업그레이드 가능한 시스템이라는 목표를 달성하는 "윈-윈" 시나리오를 만든다. 따라서 IP 협상의 초점은 전체 시스템에서 모듈 간의 인터페이스로 이동한다.


이 섹션에서는 국방 연방 획득 규정 보충(DFARS)에 정의된 다양한 유형의 데이터 권리 라이선스와 MOSA와의 관련성에 대한 실질적인 개요를 제공한다. 무제한 권리(Unlimited Rights), 정부 목적 권리(Government Purpose Rights, GPR), 제한적/한정적 권리(Limited/Restricted Rights)를 설명하고, 이를 "자금 지원 테스트"(즉, 개발 비용을 누가 지불했는지가 정부의 권리를 결정)와 연결한다.42 "올바른 균형"을 달성하기 위한 유연한 도구로서 특별 협상 라이선스 권리(Specially Negotiated License Rights)의 중요성이 강조될 것이다. MOSA를 더 잘 구현하기 위해 DFARS를 업데이트하는 데 따르는 지속적인 과제도 논의된다.45

기존의 DFARS 데이터 권리 프레임워크는 "자금 지원 테스트"와 "분리 가능성의 원칙"을 통해 MOSA 세계에 놀라울 정도로 잘 부합한다. MOSA는 분리 가능성이라는 법적 원칙이 효과적으로 적용될 수 있도록 물리적/논리적 모듈성을 제공한다. 문제는 규칙이 존재하지 않는 것이 아니라, 복잡하고 종종 프로그램 사무소에서 제대로 이해하고 적용하지 못한다는 것이다. 효과적인 MOSA 구현을 위해서는 획득 인력이 이러한 기존 도구, 특히 GPR과 특별 협상 라이선스를 사용하여 모듈식 후속 군수지원에 필요한 특정 권리를 확보하는 데 전문가가 되어야 한다.


이 섹션에서는 전체 획득 전략의 일부로서 IP 전략을 개발하기 위한 실행 가능한 권고안을 제공한다. RFP가 발표되기 전, 즉 초기 기획의 필요성을 강조한다.5 국방획득대학(DAU)의 IP 전략 직무 지원 도구나 IP 및 데이터 권리 실무 커뮤니티와 같은 자원을 프로그램 사무소를 위한 귀중한 도구로 참조한다.46 전략은 미래 업그레이드를 위한 핵심 모듈을 식별하고, 이를 위해 필요한 분리 및 재통합 데이터와 권리를 명시적으로 정의해야 한다.42 IP/데이터 권리 요구사항을 조달 요청서와 계약서에 직접 포함하는 것의 중요성이 강조될 것이다.44

MOSA 프로그램을 위한 IP 전략은 수명주기 위험 관리 계획으로 기능하는 살아있는 문서이다. 이는 미래의 기술 진부화 해소, 잠재적인 단종 문제, 후속 군수지원 요구를 예측하고, 공급자 종속으로 인해 이러한 사건에 대응할 수 없는 위험을 완화하기 위한 주요 도구로서 데이터 권리 획득을 사용한다. 반응적인 IP 접근 방식은 실패할 수밖에 없다. 데이터의 필요성이 확인될 때쯤이면 정부는 모든 협상력을 잃게 된다.

| **라이선스 유형**      | **정의**                                                     | **일반적 자금 출처** | **MOSA에 대한 시사점**                                       |
| ---------------------- | ------------------------------------------------------------ | -------------------- | ------------------------------------------------------------ |
| **무제한 권리**        | 정부가 어떠한 목적으로든 사용, 수정, 공개할 수 있는 권리.    | 완전 정부 자금 지원  | 완전한 자체 군수지원 및 경쟁을 가능하게 함.                  |
| **정부 목적 권리**     | 정부 목적(경쟁 조달 포함)으로 사용, 수정, 공개할 수 있는 권리. | 혼합 자금 지원       | 정부 목적을 위한 경쟁을 허용하며, 인터페이스 데이터에 이상적임.44 |
| **제한적/한정적 권리** | 정부의 사용이 엄격하게 제한됨.                               | 완전 민간 자금 지원  | 경쟁을 방지하므로, 구획화된 독점 모듈에 가장 적합함.         |
| **특별 협상 라이선스** | 당사자 간의 합의에 따라 권리가 맞춤형으로 정의됨.            | 다양함               | "올바른 균형"을 찾기 위한 유연한 도구로, 특정 모듈의 업그레이드 권리 확보에 유용함.43 |


이 파트는 이론과 정책에서 벗어나 실제 적용 사례를 통해 MOSA 구현의 구체적인 증거를 제시한다. 각 사례 연구는 특정 군이 MOSA 원칙을 어떻게 적용하여 고유한 문제를 해결하고 있는지 보여주며, 성공 사례와 교훈을 함께 조명한다.


이 섹션에서는 국방부에서 가장 초기의 성공적인 대규모 개방형 아키텍처 구현 사례 중 하나를 상세히 다룬다. 버지니아(VIRGINIA)급 잠수함 프로그램이 비용 절감과 노후화 방지를 목표로 어떻게 독점 시스템에서 상용 기성품(COTS) 기술과 모듈식 설계를 기반으로 한 아키텍처로 전환했는지 설명한다.48 특히 음향 탐지 체계(sonar system)와 음향 신속 COTS 도입(Acoustic Rapid COTS Insertion, A-RCI) 프로그램에 초점을 맞춘다. A-RCI는 독점 프로세서를 COTS 컴퓨터로 교체하고, 점진적인 소프트웨어 빌드(APB)와 하드웨어 갱신(TI)을 통해 지속적으로 성능을 업그레이드했다.52 설계를 이끈 핵심 아키텍처 원칙인 모듈성, 공통성, 표준 기반 개방성도 상세히 기술된다.48

A-RCI 프로그램은 현대적이고 공식화된 MOSA 이니셔티브의 "정신적 선구자"이다. 이는 하드웨어와 소프트웨어를 분리하고 신속하고 반복적인 COTS 기반 업그레이드를 사용하는 것이 가능할 뿐만 아니라, 전통적인 개발 모델보다 훨씬 우수하다는 것을 증명했다. A-RCI의 교훈-경쟁적인 비즈니스 모델의 중요성, 소프트웨어를 재사용 가능한 모듈로 분할하는 것, 기술 도입을 위한 정기적인 주기 설정-은 현재 국방부 전체의 MOSA를 정의하는 원칙과 정책에 직접적인 정보를 제공했다. 버지니아급은 MOSA의 한 예일 뿐만 아니라, 국방부가 전사적으로 이 접근 방식을 의무화하는 데 필요한 자신감을 준 근본적인 증거이다.


이 섹션에서는 F-22 랩터라는 레거시 플랫폼에 MOSA를 적용한 사례에 초점을 맞춘다. 록히드 마틴이 어떻게 MOSA를 사용하여 5세대 전투기에 대한 신속하고 반복적인 업데이트를 가능하게 하여, 12~18개월의 능력 전력화 주기로 전환했는지 상세히 설명한다.58 분석은 MOSA와 모델 기반 시스템 엔지니어링(MBSE)과 같은 디지털 엔지니어링 관행 간의 시너지를 강조한다. 이는 센서 및 방어 시스템과 같은 새로운 구성 요소를 디지털 방식으로 설계하고 통합하는 데 사용된다.58 레드햇 엔터프라이즈 리눅스와 같은 개방형 표준을 사용하여 친숙한 개발 환경을 조성하고, 신속한 프로토타이핑을 위한 "샌드박스" 접근 방식이 핵심 조력자로 논의된다.58 그 결과, 주요 구조적 재설계 없이 F-22의 작전 수명을 2040년대까지 연장할 수 있게 되었다.58

F-22 사례 연구는 MOSA가 신규 개발 프로그램뿐만 아니라 기존 플랫폼의 현대화를 위한 중요한 전략임을 보여준다. 이는 모듈식 접근 방식이 어떻게 이전에 폐쇄적이고 고도로 통합된 플랫폼을 "열어서" 진화하는 위협에 보조를 맞출 수 있게 하는지를 보여준다. F-22 프로그램에서 MOSA를 디지털 엔지니어링 및 애자일 개발과 명시적으로 연결한 것은 "디지털 삼위일체(Digital Trinity)"(MOSA, DE, Agile)가 주요 무기 체계에서 능력 전달을 가속화하기 위해 실제로 어떻게 작동하는지에 대한 구체적인 본보기를 제공한다.


이 섹션에서는 CMOSS와 CMFF 프로그램을 통한 육군의 MOSA 구현을 검토한다. CMOSS가 SOSA, FACE, VICTORY 등을 활용하여 지상 차량을 위한 공통 컴퓨팅 환경을 만드는 "표준 스위트"임을 설명한다.2 실현되고 있는 주요 이점은 수많은 개별 "박스"를 CMOSS 호환 플러그인 카드로 다양한 기능을 호스팅하는 단일 섀시로 교체함으로써 SWaP-C를 크게 줄이는 것이다.34 신속한 프로토타이핑을 위해 OTA 계약을 사용하는 새로운 기록 프로그램으로 PdM CMFF를 공식화한 것은 이 능력을 신속하게 전력화하려는 육군의 의지를 보여주는 증거로 제시된다.34

육군의 CMOSS 이니셔티브는 차량 탑재 장비의 과도한 SWaP 부담이라는 특정하고 시급한 문제에 대한 MOSA의 가장 공격적이고 구체적인 구현이라고 할 수 있다. 공식 기록 프로그램(CMFF)을 만듦으로써, 육군은 산업계에 강력한 "수요 신호"를 보냈다. 이는 PNT, EW, 무선 파형 등을 위한 경쟁력 있고 상호운용 가능한 CMOSS 호환 카드 시장을 촉진시켰다.35 CMOSS는 정부가 명확하게 정의한 아키텍처가 어떻게 산업 기반을 성공적으로 형성하여 개방형 모듈식 솔루션을 제공하게 하는지를 보여주는 사례 연구이다.


이 파트는 MOSA가 최종 상태가 아니라 차세대 국방 기술 및 획득 프로세스를 위한 근본적인 조력자임을 분석하며 미래를 조망한다.


이 섹션에서는 MOSA, 디지털 엔지니어링(DE), 애자일 개발 방법론 간의 강력한 시너지를 탐구한다. "디지털 삼위일체(Digital Trinity)" 개념을 인용하고 59, 이 세 요소가 어떻게 근본적으로 상호 관련되어 있는지 설명한다.60

- **MOSA**는 모듈식 개방형 아키텍처 구조를 제공한다.
- **DE**(MBSE, 디지털 트윈, 시뮬레이션 포함)는 가상 환경에서 해당 구조를 설계, 테스트 및 유지하여 일정을 단축하고 비용을 절감하는 도구를 제공한다.58
- **애자일**은 모듈식 아키텍처에 능력을 신속하게(스프린트 또는 증분 단위로) 전력화하기 위한 반복적 개발 프로세스를 제공한다.60

이러한 융합은 전체 획득 및 엔지니어링 수명주기의 총체적인 변화를 나타낸다. 이는 선형적이고 문서 기반인 프로세스(폭포수 모델)에서 지속적이고 모델 기반이며 반복적인 순환으로의 전환이다. 여기서 MOSA는 핵심축이다. 배포할 모듈식 아키텍처가 없다면 애자일 개발의 영향은 제한적이다. 복잡한 상호작용을 모델링할 디지털 도구가 없다면 대규모 모듈식 시스템을 관리하기 어렵다. 이 세 가지는 단순히 상호 보완적인 것이 아니라, 신속하고 지속적인 능력 전달이라는 비전을 달성하기 위해 상호 의존적이다.


이 섹션에서는 MOSA가 인공지능(AI), 머신러닝(ML), 자율성을 국방 시스템에 효과적으로 통합하기 위한 중요한 전제 조건이라고 주장한다. MOSA의 모듈식 특성은 AI/ML 소프트웨어 구성 요소를 교체 가능한 모듈로 취급할 수 있게 하여, 기술과 위협이 진화함에 따라 알고리즘을 신속하게 업데이트할 수 있게 한다. MOSA 기반 디지털 테스트베드가 시뮬레이션에서 자율성 소프트웨어를 개발하고 평가하는 데 어떻게 사용되고 있는지 논의한다.64 AI를 시스템에 안전하게 통합할 필요성과 62, 모듈식 아키텍처 기반의 AI/ML 지원 EW 시스템 개발이 강조된다.35

AI/ML 기술은 전통적인 하드웨어 개발보다 훨씬 빠른 속도로 진화한다. 깊이 내장된 독점 AI 알고리즘을 가진 단일체 시스템은 거의 즉시 구식이 될 것이다. MOSA는 이러한 빠른 진화를 수용할 수 있는 유일한 아키텍처 접근 방식이다. 이는 시스템의 AI "두뇌"를 프로세서 카드처럼 교체 가능한 구성 요소로 만들어, 경쟁 시장에서 가장 진보된 알고리즘을 현장 시스템에 지속적으로 통합하여 효과를 유지할 수 있도록 하는 "최고 품종(best-of-breed)" 접근 방식을 가능하게 한다.


이 섹션에서는 MOSA가 국방 획득 시스템에 미치는 광범위한 영향을 논의한다. MOSA가 DE 및 애자일과 결합하여 어떻게 획득의 초점을 완성된 플랫폼 획득에서 지속적이고 업그레이드 가능한 능력 획득으로 바꾸는지 탐구한다. 유연한 설계로 구조화된 요구사항을 대체하고, 전술적 최전선에서 MOSA 호환 구성 요소의 디지털 라이브러리를 사용하여 첨단 제조를 활용하는 등의 개념을 다룬다.60 이러한 역동적인 접근 방식을 지원하기 위해 보다 유연한 예산 책정 및 재프로그래밍 권한의 필요성도 언급된다.60

완전히 실현된 MOSA 기반 획득 시스템은 전통적인 공장 생산 라인보다 현대 소프트웨어 회사의 지속적 통합/지속적 전달(CI/CD) 파이프라인과 더 유사해 보인다. "제품"은 정적인 하드웨어가 아니라 지속적으로 진화하는 시스템이다. 이는 요구사항(더 유연해야 함), 예산(더 적응 가능해야 함), 테스트(더 지속적이어야 함), 계약(더 작고 빈번한 수주를 지원해야 함) 등 획득 시스템의 모든 측면에 심대한 영향을 미친다. MOSA는 단지 시스템을 바꾸는 것이 아니라, 그것을 구축하는 비즈니스 프로세스의 근본적인 변화를 강요하고 있다.


이 마지막 파트는 전체 보고서를 종합하여 국방 분야의 다양한 이해관계자들을 위한 명확하고 실행 가능한 권고안을 제시한다.


이 섹션에서는 높은 수준의 전략적 권고안을 제공한다. 주요 내용은 다음과 같다:

- 모든 주요 프로그램에서 MOSA 구현을 위한 공식적인 총수명주기 비용-편익 분석을 의무화하고 예산을 지원한다.
- 교차 프로그램 MOSA 조력자(공통 구성 요소, 표준 저장소, 디지털 도구)를 관리할 책임이 있는 전사적 수준의 PEO 또는 기관을 설립하고 자금을 지원한다.
- 반복적인 개발을 지원할 수 있도록 보다 유연하고 포트폴리오 기반의 자금 지원을 허용하도록 예산 프로세스를 개혁한다.
- MOSA 관련 정책 및 규정(DFARS, MTA 경로 지침 등)을 지속적으로 개선하고 명확히 한다.


이 섹션에서는 PM과 그 팀을 위한 전술적 조언을 제공한다. 주요 내용은 다음과 같다:

- 프로그램 시작부터 SEP와 IP 전략을 통합되고 동등한 문서로 개발한다.
- 사내 시스템 엔지니어링 및 IP 전문성 구축에 투자한다.
- 주요 인터페이스를 정의하고 통제하는 데 적극적으로 나서며, 아키텍처 통제권을 주 계약업체에 위임하지 않는다.
- 분리 및 재통합 데이터 권리 획득을 우선시한다.
- "기어가기, 걷기, 뛰기(crawl, walk, run)" 접근 방식을 사용하여, 몇 개의 핵심 모듈로 시작하여 시간이 지남에 따라 MOSA 적용 범위를 확장한다.


이 섹션에서는 산업계를 위한 지침을 제공한다. 주요 내용은 다음과 같다:

- 독점적인 폐쇄형 시스템에서 개방형 모듈식 솔루션으로의 전환을 장기적인 비즈니스 기회로 받아들인다.
- 주요 MOSA 표준(FACE, SOSA 등)에 대한 전문성 개발에 투자한다.
- 표준 컨소시엄에 적극적으로 참여하여 그 진화를 형성하는 데 도움을 준다.
- 공급자 종속적 후속 군수지원이 아닌 개별 모듈의 가치와 혁신에 기반한 수익성 있는 비즈니스 모델을 개발한다.
- 중소 및 비전통적 기업의 경우, MOSA 생태계 내 특정 모듈을 위한 동급 최고의 솔루션 개발에 집중한다.


1. Modular Open Systems Approach – DoD Research & Engineering ..., accessed July 9, 2025, https://www.cto.mil/sea/mosa/
2. A guide to the DoD's Modular Open Systems Approach - Systel, accessed July 9, 2025, https://systelusa.com/blog/guide-dod-modular-open-systems-approach-mosa/
3. Benefits of MOSA - Curtiss-Wright Defense Solutions, accessed July 9, 2025, https://www.curtisswrightds.com/media-center/blog/what-is-mosa-benefits
4. GAO-25-106931, WEAPON SYSTEMS ACQUISITION: DOD Needs ..., accessed July 9, 2025, https://www.gao.gov/assets/gao-25-106931.pdf
5. Impacts and Recommendations for Achieving Modular Open Systems Architectures --Fifth Post in a Series - SEI Blog, accessed July 9, 2025, https://insights.sei.cmu.edu/blog/impacts-and-recommendations-for-achieving-modular-open-systems-architectures-fifth-post-in-a-series/
6. Modular Open Systems Approach (MOSA) | www.dau.edu, accessed July 9, 2025, https://www.dau.edu/acquipedia-article/modular-open-systems-approach-mosa
7. CLE019 Modular Open Systems Approach (MOSA) - DAU, accessed July 9, 2025, [https://www.dau.edu/sites/default/files/Migrated/CopDocuments/CLE019_pf_Mods%200-6.pdf](https://www.dau.edu/sites/default/files/Migrated/CopDocuments/CLE019_pf_Mods 0-6.pdf)
8. Modular Open Systems Approach (MOSA) - NAVAIR, accessed July 9, 2025, https://www.navair.navy.mil/MOSA
9. Modular Open Systems Approach (MOSA) - Defense Standardization Program, accessed July 9, 2025, https://www.dsp.dla.mil/Programs/MOSA/
10. Modular Open Systems Approach (MOSA) - AcqNotes, accessed July 9, 2025, https://acqnotes.com/acqnote/careerfields/modular-open-systems-approach
11. What is MOSA? - everything RF, accessed July 9, 2025, https://www.everythingrf.com/community/what-is-mosa
12. FACE Standard for Avionics Software | Ansys, accessed July 9, 2025, https://www.ansys.com/industries/face-standard-aviation-software
13. What is MOSA? - BAE Systems, accessed July 9, 2025, https://www.baesystems.com/en-us/definition/what-is-mosa
14. Frappe's modular architecture versus monolithic applications, accessed July 9, 2025, https://hybrowlabs.com/blog/frappes-modular-architecture-versus-monolithic-applications
15. Architectural Design - Monolithic versus Modular Monolithic versus Microservices, accessed July 9, 2025, https://medium.com/codenx/architectural-design-monolithic-versus-modular-monolithic-versus-microservices-9bd318941cfd
16. Comparing Monolith, Microservices, and Modular Monolith: Communications, Data, Development, and Deployments | by Mehmet Ozkaya, accessed July 9, 2025, https://mehmetozkaya.medium.com/comparing-monolith-microservices-and-modular-monoliths-communications-data-development-and-5ebf643191fd
17. Monolithic Application vs Microservices Architecture Guide - OpenLegacy, accessed July 9, 2025, https://www.openlegacy.com/blog/monolithic-application
18. Monolith vs microservices: Comparing architectures for software delivery - Chronosphere, accessed July 9, 2025, https://chronosphere.io/learn/comparing-monolith-and-microservice-architectures-for-software-delivery/
19. Get the Most Out of MOSA: Begin With the End in Mind - Events, accessed July 9, 2025, https://events.techconnect.org/MOSA_2023/slides/Get-the-Most-Out-of-MOSA-Begin-with-the-End-in-Mind.pdf
20. The Future of Standardized Defense Platforms Using MOSA, SOSA and VPX Open Architectures - Vicor Corporation, accessed July 9, 2025, https://www.vicorpower.com/resource-library/white-papers/defense-platforms-using-mosa-sosa-and-vpx
21. The Future of Standardized Defense Platforms Using MOSA, SOSA, and VPX Open Architectures - White Paper - All About Circuits, accessed July 9, 2025, https://www.allaboutcircuits.com/industry-white-papers/the-future-of-standardized-defense-platforms-using-mosa-sosa-and-vpx-open-architectures/
22. The FACE Approach | www.opengroup.org, accessed July 9, 2025, https://www.opengroup.org/face/approach
23. What Is FACE™? | Wind River, accessed July 9, 2025, https://www.windriver.com/solutions/learning/face
24. FACE Standard Compliant Software | FACE Architecture, accessed July 9, 2025, https://www.lynx.com/industry-standards/future-airborne-capability-environment-face
25. What is SOSA? - everything RF, accessed July 9, 2025, https://www.everythingrf.com/community/what-is-sosa
26. Sensor Open Systems Architecture (SOSA) | Curtiss-Wright, accessed July 9, 2025, https://www.curtisswrightds.com/capabilities/open-architectures/mosa/sensor-open-systems-architecture
27. The SOSA™ standard – definition, objectives and benefits, accessed July 9, 2025, https://www.reflexces.com/newsroom/the-sosa-standard-definition-objectives-and-benefits
28. Understanding SOSA - everything RF, accessed July 9, 2025, https://cdn.everythingrf.com/live/SOSA_ebook_Final_1_638575831347426778_2_3_638761395097370396.pdf
29. How MOSA Is Redefining Defense Development and Procurement, accessed July 9, 2025, https://www.daisydata.com/blog/how-mosa-is-redefining-defense-development-and-procurement/
30. Open Mission Systems (OMS) in a Nutshell - VDL - AF.mil, accessed July 9, 2025, https://www.vdl.afrl.af.mil/programs/oam/OMS_Marketing.pdf
31. Open Missions Systems (OMS) - VDL, accessed July 9, 2025, https://www.vdl.afrl.af.mil/programs/oam/oms.php
32. Use of Open Mission Systems/Universal Command and Control Interface, accessed July 9, 2025, [https://www.afacpo.com/AQDocs/AQMemos/20181009%20-%20OMS%20UCI%20Memo.pdf](https://www.afacpo.com/AQDocs/AQMemos/20181009 - OMS UCI Memo.pdf)
33. Curtiss-Wright and C5ISR/EW Modular Open Suite of Standards (CMOSS), accessed July 9, 2025, https://www.curtisswrightds.com/capabilities/open-architectures/mosa/army-cmoss
34. CMOSS Mounted Form Factor becomes new Army program | Article ..., accessed July 9, 2025, https://www.army.mil/article/285078/cmoss_mounted_form_factor_becomes_new_army_program
35. Pacific Defense Selected for U.S. Army CMFF Program - Business Wire, accessed July 9, 2025, https://www.businesswire.com/news/home/20250327094467/en/Pacific-Defense-Selected-for-U.S.-Army-CMFF-Program
36. Press - Perceptronics Solutions, accessed July 9, 2025, https://www.percsolutions.com/press
37. Sensor Open Systems Architecture (SOSA), an overview, accessed July 9, 2025, https://militaryembedded.com/comms/communications/sensor-open-systems-architecture-sosa-an-overview
38. Evaluating the DOD's Modular Open Systems Approach: Progress, Challenges, and Recommendations - Fed Contract Pros, accessed July 9, 2025, https://www.fedcontractpros.com/blog/evaluating-the-dods-modular-open-systems-approach-progress-challenges-and-recommendations
39. Weapon Systems Acquisition: DOD Needs Better Planning to Attain Benefits of Modular Open Systems - GAO, accessed July 9, 2025, https://www.gao.gov/products/gao-25-106931
40. Readiness for Open Systems: How Prepared Are the Pentagon and ..., accessed July 9, 2025, https://www.csis.org/analysis/readiness-open-systems-how-prepared-are-pentagon-and-defense-industry-coordinate
41. Modular Open Systems Approach Overview and Efforts - Defense Standardization Program, accessed July 9, 2025, [https://www.dsp.dla.mil/Portals/26/Documents/Publications/Journal/200101-DSPJ%20-%2001.pdf](https://www.dsp.dla.mil/Portals/26/Documents/Publications/Journal/200101-DSPJ - 01.pdf)
42. In-Brief: Modular Open Systems Approaches and Intellectual ... - DAU, accessed July 9, 2025, [https://www.dau.edu/sites/default/files/2024-11/MOSA%20Flyer%2020241105_Final.pdf](https://www.dau.edu/sites/default/files/2024-11/MOSA Flyer 20241105_Final.pdf)
43. DOD Releases Intellectual Property Guidebook: Key Insights for Defense Contractors, Part 1, accessed July 9, 2025, https://www.jdsupra.com/legalnews/dod-releases-intellectual-property-2431831/
44. The Right Balance: Ensuring the right balance in negotiating intellectual property in Army contracting | Article | The United States Army, accessed July 9, 2025, https://www.army.mil/article/285512/the_right_balance_ensuring_the_right_balance_in_negotiating_intellectual_property_in_army_contracting
45. DOD's Proposed Data Rights Regulations For MOSA Undercut Its Pursuit Of Commercial Innovation | Government Contracts Insights, accessed July 9, 2025, https://govcon.mofo.com/topics/dod-s-proposed-data-rights-regulations-for-mosa-undercut-its-pursuit-of-commercial-innovation
46. Intellectual Property (IP) & Data Rights | www.dau.edu, accessed July 9, 2025, https://www.dau.edu/cop/IPDR
47. Department Issues New Guide for Implementing MOSA in DoD Programs | www.dau.edu, accessed July 9, 2025, https://www.dau.edu/blogs/department-issues-new-guide-implementing-mosa-dod-programs
48. Virginia Class Submarine - SEBoK, accessed July 9, 2025, https://sebokwiki.org/wiki/Virginia_Class_Submarine
49. Virginia-class submarine - Wikipedia, accessed July 9, 2025, https://en.wikipedia.org/wiki/Virginia-class_submarine
50. DEFENSE STANDARDIZATION PROGRAM, accessed July 9, 2025, [https://share.ansi.org/shared%20documents/Other%20Services/SBB/Case-Study-Virginia-Class-Submarine.pdf](https://share.ansi.org/shared documents/Other Services/SBB/Case-Study-Virginia-Class-Submarine.pdf)
51. The Future of U.S. Nuclear Submarines - Stanford University, accessed July 9, 2025, http://large.stanford.edu/courses/2016/ph241/yachanin1/
52. AN/BYG-1 Combat Control System - Director Operational Test and Evaluation, accessed July 9, 2025, https://www.dote.osd.mil/Portals/97/pub/reports/FY2011/navy/2011anbyg-1.pdf?ver=2019-08-22-112032-270
53. Acoustic Rapid Commercial Off-the-Shelf (COTS) Insertion (A-RCI ..., accessed July 9, 2025, https://www.dote.osd.mil/Portals/97/pub/reports/FY2010/navy/2010arci.pdf?ver=2019-08-22-112818-050
54. U.S. attack submarine fleet expands and gets sonar, COTS upgrades, accessed July 9, 2025, https://militaryembedded.com/radar-ew/sensors/u-sonar-cots-upgrades
55. The How and Why of Open Architecture - Issuu, accessed July 9, 2025, https://issuu.com/julianne.m.johnson/docs/usw_spring_2008/s/10628300
56. The A-RCI Process - Leadership and Management Principles | Acquisition Talk, accessed July 9, 2025, [https://acquisitiontalk.com/wp-content/uploads/2022/06/Naval-Engineers-Journal-2008-Johnson-The-A%E2%80%90RCI-Process-Leadership-and-Management-Principles.pdf](https://acquisitiontalk.com/wp-content/uploads/2022/06/Naval-Engineers-Journal-2008-Johnson-The-A‐RCI-Process-Leadership-and-Management-Principles.pdf)
57. Professional Note: Acoustic Rapid Commercial Off-the-Shelf Insertion: A Model for the Future - U.S. Naval Institute, accessed July 9, 2025, https://www.usni.org/magazines/proceedings/2004/january/professional-note-acoustic-rapid-commercial-shelf-insertion
58. The Lockheed Martin F-22 Raptor: MOSA in flight - Military ..., accessed July 9, 2025, https://militaryembedded.com/avionics/computers/the-lockheed-martin-f-22-raptor-mosa-in-flight
59. Modular Open Systems Approach (MOSA) | www.dau.edu, accessed July 9, 2025, https://www.dau.edu/cop/mosa
60. VIEWPOINT: Accelerating Defense Innovation Requires Changes to Acquisition Approach, accessed July 9, 2025, https://www.nationaldefensemagazine.org/articles/2024/9/20/viewpoint-accelerating-defense-innovation-requires-changes-to-acquisition-approach
61. Digital Engineering - BAE Systems, accessed July 9, 2025, https://www.baesystems.com/en-us/who-we-are/electronic-systems/engineering-careers/digital-engineering
62. 2024 Predictions for Aerospace & Defense Product, Systems, and Software Development, accessed July 9, 2025, https://www.jamasoftware.com/blog/2024-predictions-for-aerospace-defense-product-systems-and-software-development/
63. status of adoption and implementation of digital engineering infrastructure and workforce development within the department of defense - USD(R&E), accessed July 9, 2025, https://ac.cto.mil/wp-content/uploads/2022/12/Status-of-Adoption-and-Implementation-of-Digital-Engineering-Infrastructure-and-Workforce-Development-Within-the-Department-of-Defense.pdf
64. A MODULAR OPEN SYSTEM APPROACH TO A DIGITAL AUTONOMY TESTBED - NDIA Michigan, accessed July 9, 2025, [https://ndia-mich.org/images/events/gvsets/2024/papers/MOSA/4%2040PM%20A%20Modular%20Open%20System%20Approach%20to%20Digital%20Autonomy%20Testbed.PDF](https://ndia-mich.org/images/events/gvsets/2024/papers/MOSA/4 40PM A Modular Open System Approach to Digital Autonomy Testbed.PDF)
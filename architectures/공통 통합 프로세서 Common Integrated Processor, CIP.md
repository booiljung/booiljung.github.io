# 공통 통합 프로세서(Common Integrated Processor, CIP)
[컴퓨터 아키텍처 (Computer Architectures)](./index.md)


현대 항공우주 및 국방 기술의 정점에 있는 5세대 전투기 F-35 라이트닝 II의 성공은 단순히 스텔스 성능이나 기체 역학적 우수성에만 기인하지 않는다. 그 핵심에는 항공기의 '중추 신경계' 역할을 수행하는 고도로 통합된 컴퓨팅 시스템, 즉 공통 통합 프로세서(Common Integrated Processor, CIP)가 자리 잡고 있다. CIP는 F-35가 전례 없는 수준의 상황 인식, 정보 융합, 네트워크 중심 작전 능력을 발휘할 수 있게 하는 기술적 기반이다. 이 보고서는 F-35의 CIP를 다각적으로 심층 분석하여, 그 탄생 배경이 된 항공전자 아키텍처의 패러다임 변화부터 기술적 구성, 핵심 기능, 현대화 과정에서 직면한 도전과제, 그리고 미래 항공전 및 방위 산업에 미치는 전략적 함의까지 포괄적으로 고찰하는 것을 목표로 한다.

본 보고서는 먼저 CIP의 등장을 필연적으로 만든 기존 연합형(Federated) 아키텍처의 한계와 이를 극복하기 위한 통합 모듈형 항공전자(Integrated Modular Avionics, IMA) 개념의 대두를 분석한다. 이후 F-35의 CIP가 IMA 철학을 어떻게 구현했으며, 항공기의 '두뇌'로서 센서 융합과 같은 핵심 기능을 어떻게 수행하는지 상세히 설명한다. 또한, 기술 재생 3(Technology Refresh 3, TR-3)로 대표되는 CIP의 현대화 과정과 이 과정에서 발생한 기술적, 프로그램적 난제 및 복잡한 인증 절차를 심도 있게 다룬다. 마지막으로, CIP가 열어젖힌 소프트웨어 정의 항공기 시대의 도래와 인공지능(AI)의 통합, 그리고 방위 산업 생태계의 변화 등 미래 전략적 전망을 제시함으로써 CIP에 대한 종합적이고 통찰력 있는 이해를 제공하고자 한다.


공통 통합 프로세서(CIP)의 등장은 단순한 기술적 진보가 아니라, 항공전자 시스템 설계 철학의 근본적인 혁명이었다. 이 혁명을 이해하기 위해서는 CIP가 해결하고자 했던 문제, 즉 기존 연합형 아키텍처의 지속 불가능한 복잡성과 물리적 한계를 먼저 살펴봐야 한다.


전통적인 항공전자 시스템은 연합형(Federated) 아키텍처를 기반으로 구축되었다.1 이 방식은 항공기의 각 기능, 예를 들어 항법, 통신, 레이더, 비행 제어 등이 각각의 독립적인 하드웨어 장치, 즉 라인 교체 단위(Line Replaceable Unit, LRU)에 탑재되는 구조를 의미한다.2 각 LRU는 자체 프로세서, 메모리, 전원 공급 장치 및 입출력 인터페이스를 갖춘 독립적인 컴퓨터 시스템으로, 특정 기능만을 전담하여 수행했다.

이러한 분산형 접근 방식은 초기에는 각 기능의 독립성을 보장하고 개발 및 테스트를 용이하게 한다는 장점이 있었으나, 항공기 시스템의 복잡성이 기하급수적으로 증가함에 따라 심각한 한계에 봉착했다.

- **물리적 제약 (크기, 중량 및 전력 - SWaP):** 항공기에 탑재되는 센서와 기능이 늘어날수록 LRU의 수도 비례하여 증가했다. 이는 항공기 전체의 중량, 부피, 전력 소모(Size, Weight, and Power, SWaP)를 급격히 증가시키는 결과를 초래했다.1 과도한 중량은 연료 효율을 악화시키고 항속 거리를 단축시켰으며, 제한된 내부 공간에 수많은 LRU를 배치하는 것은 물리적으로 불가능한 수준에 이르렀다.
- **복잡성과 비용 증가:** 각 LRU는 고유한 배선을 필요로 했으며, 이는 항공기 내부를 "무겁고 복잡한" 케이블링으로 가득 채웠다.1 수많은 연결 지점은 잠재적인 고장 지점을 늘리고, 통합 과정에서의 복잡성과 비용을 증대시켰다. 시스템 통합은 각기 다른 공급업체가 개발한 수십, 수백 개의 LRU를 물리적으로 연결하고 소프트웨어적으로 연동시키는 지난한 과정이었다.
- **경직성과 노후화 문제:** 연합형 아키텍처의 가장 큰 문제점 중 하나는 근본적인 경직성이었다.2 단일 기능을 업그레이드하기 위해서도 해당 LRU 전체의 하드웨어 재설계가 필요한 경우가 많았다. 이는 새로운 기술을 신속하게 도입하거나 부품 단종과 같은 노후화 문제에 대응하는 것을 매우 어렵고 비용이 많이 드는 일로 만들었다.2

이러한 연합형 아키텍처의 문제점은 IT 산업에서 대규모 시스템이 겪는 모놀리식(Monolithic) 아키텍처의 문제점과 유사하다. 모놀리식 소프트웨어도 시스템 규모가 커짐에 따라 코드베이스가 비대해지고, 작은 변경 사항이 전체 시스템의 빌드와 재배포를 요구하며, 유지보수가 어려워지는 등의 문제를 겪는다.5 항공전자 시스템 역시 더 많은 센서와 처리 능력을 요구하게 되면서, 연합형 아키텍처는 물리적, 재정적으로 지속 불가능한 막다른 길에 다다랐다.


연합형 아키텍처의 한계를 극복하기 위한 대안으로 등장한 것이 바로 통합 모듈형 항공전자(Integrated Modular Avionics, IMA)이다. IMA는 항공전자 시스템 설계의 패러다임 자체를 바꾸는 혁신적인 개념으로, "서로 다른 중요도 수준의 수많은 애플리케이션을 지원할 수 있는 공통 하드웨어 모듈의 집합체"로 정의된다.3 즉, 각 기능별로 전용 하드웨어를 사용하는 대신, 유연하고 재사용 가능한 하드웨어 및 소프트웨어 자원의 공유 풀(pool)을 만들어 여러 기능을 하나의 공통 플랫폼에서 실행하는 방식이다.1

IMA 철학의 핵심 원칙은 다음과 같다.

- **자원 공유:** 각 기능별 전용 프로세서를 사용하는 대신, 중앙 집중화된 고성능 컴퓨팅 자원 풀을 여러 기능이 공유한다.1 이를 통해 하드웨어의 수를 획기적으로 줄이고 자원 활용률을 극대화할 수 있다.
- **모듈화와 표준화:** 표준화된 하드웨어 모듈과 소프트웨어 인터페이스(API)를 사용하여 상호운용성, 이식성, 재사용성을 보장한다.3 이는 유지보수와 업그레이드를 단순화한다. 특정 모듈에 문제가 발생하면 동일한 규격의 다른 모듈로 쉽게 교체할 수 있으며, 하드웨어 교체 없이 소프트웨어 업그레이드만으로 새로운 기능을 추가하는 것이 가능해진다.3
- **파티셔닝(Partitioning):** IMA 개념의 안전성을 보장하는 가장 핵심적인 기술이다. 파티셔닝은 하나의 프로세서에서 여러 애플리케이션을 실행할 때, 각 애플리케이션을 서로 완벽하게 격리시키는 기술을 의미한다. 이는 **시간 분할(Temporal Partitioning)**과 **공간 분할(Spatial Partitioning)**로 나뉜다.13
  - **공간 분할:** 각 애플리케이션(파티션)에 독립적인 메모리 공간을 할당하고, 한 파티션의 소프트웨어가 다른 파티션의 메모리 영역을 침범하지 못하도록 운영체제와 하드웨어 수준에서 강제한다. 이를 통해 특정 애플리케이션의 오류가 다른 애플리케이션으로 전파되는 것을 원천적으로 차단한다.15
  - **시간 분할:** 각 파티션에 미리 정해진 CPU 실행 시간(타임 슬롯)을 고정적으로 할당한다. 특정 파티션의 소프트웨어가 무한 루프에 빠지거나 비정상적으로 많은 연산을 수행하더라도, 할당된 시간만 사용하고 강제로 제어권을 다른 파티션에 넘겨주게 된다. 이는 하나의 애플리케이션이 CPU를 독점하여 다른 중요 기능(예: 비행 제어)의 실행을 방해하는 것을 막는다.14

이러한 견고한 파티셔닝 덕분에, 비행에 치명적인(flight-critical) 애플리케이션과 상대적으로 중요도가 낮은 임무(mission-critical) 또는 정보(information-level) 애플리케이션을 동일한 프로세서에서 안전하게 함께 실행하는 '혼합 중요도(mixed-criticality)' 시스템의 구현이 가능해졌다.

| 특성                     | 연합형 아키텍처 (Federated Architecture)              | 통합 모듈형 항공전자 (IMA) 아키텍처                         |
| ------------------------ | ----------------------------------------------------- | ----------------------------------------------------------- |
| **하드웨어**             | 기능별 전용 하드웨어 (LRU) 2                          | 표준화된 공통 컴퓨팅 모듈의 공유 풀 3                       |
| **소프트웨어**           | 하드웨어에 강하게 종속된 맞춤형 소프트웨어            | 하드웨어로부터 추상화된 이식 가능한 애플리케이션 9          |
| **자원 관리**            | 각 LRU가 자체 자원 독립적으로 관리                    | 중앙 집중화된 자원 공유 및 파티셔닝 기반 할당 1             |
| **데이터 통신**          | 점대점(Point-to-Point) 방식의 복잡한 물리적 배선 3    | 고속 데이터 버스 또는 스위치 네트워크를 통한 통신 9         |
| **업그레이드 및 확장성** | 하드웨어 교체를 동반하여 매우 어렵고 비용이 많이 듦 2 | 소프트웨어 업그레이드 및 모듈 추가/교체로 유연하고 용이함 3 |
| **SWaP**                 | 기능 증가에 따라 급격히 증가 (높음) 1                 | 자원 통합으로 획기적 절감 (낮음) 10                         |
| **개발 및 통합**         | 각 LRU는 독립 개발, 전체 시스템 통합은 매우 복잡 1    | 공급업체별 독립 개발 후 표준 인터페이스로 통합 용이 9       |
| **결함 허용**            | 채널 이중화/삼중화 등 물리적 중복성에 의존            | 동적 재구성, 예비 모듈 활용 등 유연한 결함 관리 가능 9      |
| **생애주기비용**         | 높은 개발, 통합, 유지보수 비용 17                     | 부품 공용화, 재사용성, 용이한 업그레이드로 비용 절감 17     |


IMA로의 전환은 단순히 기술적 이상을 추구한 것이 아니라, 명확하고 설득력 있는 경제적, 성능적 이점을 기반으로 한 전략적 결정이었다. 여러 연구와 실제 적용 사례는 IMA가 제공하는 정량적 이점을 명확히 보여준다.

- **SWaP(크기, 중량, 전력) 감소:** IMA 아키텍처 도입의 가장 직접적이고 강력한 동인은 SWaP의 획기적인 절감이다. 28개의 서로 다른 항공기 프로젝트에서 연합형 시스템을 IMA로 교체한 사례를 분석한 한 연구에 따르면, 크기는 28%에서 50%, 전력 소모는 38%에서 60%, 그리고 중량은 25%에서 50%까지 감소하는 효과를 보였다.16 보잉 777ER 항공기에 하니웰이 IMA 시스템을 적용했을 때, 기존 항공전자 시스템 대비 50%의 SWaP 절감을 달성했으며 18, 에어버스 A380의 경우 프로세서 유닛의 부품 번호(part number) 수를 절반으로 줄였다.19 이러한 중량 감소는 직접적으로 연료 효율 향상과 운용 비용 절감으로 이어진다.
- **생애주기비용(Lifecycle Cost) 절감:** 항공전자 시스템의 비용은 초기 개발 비용뿐만 아니라 수십 년에 걸친 운용 및 유지보수 비용을 포함하는 생애주기비용 관점에서 평가해야 한다. IMA는 이러한 생애주기비용을 크게 절감한다. COTS(Commercial-Off-The-Shelf, 상용 기성품) 부품 활용과 표준화된 모듈은 부품 공용화를 통해 예비 부품 재고 및 관리 비용을 줄인다. 또한, 소프트웨어 재사용과 간소화된 업그레이드 절차는 '변경 비용(cost of change)'을 낮춘다.17 하니웰의 777ER 사례에서는 소프트웨어 생애주기비용이 50%나 감소하는 성과를 거두었다.18
- **개발 효율성 향상:** IMA의 모듈화 및 표준화된 접근 방식은 개발 과정에도 긍정적인 영향을 미친다. 777ER 프로그램에서는 소프트웨어 개발 노력이 20% 감소했다.18 서로 다른 공급업체들이 표준화된 인터페이스(예: ARINC 653)를 준수하며 각자의 애플리케이션을 독립적으로 개발하고 사전 검증할 수 있기 때문에, 최종 시스템 통합 단계의 복잡성과 리스크가 크게 줄어든다.9

결론적으로, 항공전자 시스템의 성능 요구사항이 급증하면서 연합형 아키텍처는 SWaP와 복잡성 측면에서 한계에 도달했다. 이러한 위기 상황은 IMA라는 새로운 패러다임의 등장을 촉발하는 직접적인 원인이 되었다. IMA가 제공하는 SWaP 및 생애주기비용의 획기적인 절감 효과는, 새로운 아키텍처를 도입하고 인증하는 데 따르는 막대한 초기 투자를 정당화하는 강력한 비즈니스 케이스를 제공했다. 이는 항공우주 분야가 고유의 엄격한 안전 및 인증 요구사항에도 불구하고, IT 산업의 마이크로서비스 아키텍처와 같은 검증된 개념을 적극적으로 수용하고 적응하여 유사한 확장성 및 복잡성 문제를 해결하고 있음을 보여준다.5 이러한 경향은 미래 항공전자 시스템이 분산 IMA(DIMA)나 클라우드 네이티브 항공전자(Cloud-Native Avionics)와 같은 더욱 진보된 분산 컴퓨팅 패러다임으로 진화할 가능성을 시사한다.21


IMA라는 혁신적인 설계 철학은 F-35 전투기에서 공통 통합 프로세서(Common Integrated Processor, CIP)라는 구체적인 형태로 구현되었다. CIP는 단순히 강력한 컴퓨터가 아니라, F-35의 모든 감각과 반응을 관장하는 중앙 신경계이자, 항공기의 전투 능력을 정의하는 핵심 요소이다.


F-35 프로그램에서는 '공통 통합 프로세서(CIP)'라는 용어를, 주요 공급업체인 L3Harris에서는 '통합 코어 프로세서(ICP)'라는 용어를 사용하지만, 이 둘은 동일한 시스템을 지칭한다.24 CIP는 단일 칩이 아닌, 여러 처리 모듈로 구성된 하나의 거대한 시스템으로, F-35에 탑재된 IMA 아키텍처의 물리적 실체이다. 이 시스템은 명실상부한 "항공기의 두뇌(brains of the aircraft)"로 묘사되며, 임무 수행에 필요한 거의 모든 핵심 데이터를 처리하는 책임을 진다.24

CIP는 특정 기능에 종속되지 않는 개방형 시스템 아키텍처(Open System Architecture, OSA)를 기반으로 설계되어 탁월한 모듈성과 유연성을 자랑한다.25 이는 미래의 위협 변화에 대응하여 새로운 하드웨어나 소프트웨어를 보다 쉽게 통합할 수 있도록 하는 핵심적인 설계 사상이다.


CIP의 가장 중요한 임무는 과거 연합형 아키텍처에서는 수십 개의 개별 LRU가 처리하던 기능들을 소프트웨어 애플리케이션 형태로 통합하여 단일 플랫폼에서 실행하는 것이다. CIP는 항공기 내 거의 모든 핵심 하위 시스템으로부터 들어오는 데이터를 통합 처리하는 중앙 허브 역할을 수행한다.25

- **신호 및 데이터 처리의 중앙화:** CIP는 레이더, 전자전 장비 등 개별 센서 시스템에 자체 프로세서를 둘 필요 없이, 모든 센서와 임무 항공전자 장비의 신호 처리(signal processing)와 데이터 처리(data processing)를 전담한다.25 이를 통해 하드웨어의 중복을 제거하고 데이터 흐름을 일원화하여 효율성을 극대화한다.
- **핵심 통합 기능:** CIP가 처리하는 주요 기능들은 F-35의 전투 능력을 구성하는 거의 모든 요소를 포함한다.24
  - **센서 시스템:**
    - AN/APG-81 능동 전자주사 배열(AESA) 레이더
    - AN/ASQ-239 전자전(EW) 시스템
    - AN/AAQ-37 분산 개구 시스템(DAS)
    - AN/AAQ-40 전자광학 표적 시스템(EOTS)
  - **통신, 항법, 식별(CNI) 시스템:** Northrop Grumman이 개발한 CNI 스위트는 음성/데이터 통신, 피아식별(IFF), Link-16 등 수십 가지 기능을 그 자체로 통합한 소형 IMA 시스템이며, 이 또한 CIP와 연동된다.26
  - **유도 및 제어:** 비행 제어 시스템을 위한 연산을 처리한다.
  - **디스플레이 시스템:** 조종석의 파노라마형 칵핏 디스플레이(PCD)와 헬멧 장착 시현기(HMD)에 시현될 그래픽과 정보를 생성하고 전송한다.
  - **진단 및 상태 관리:** CIP는 기체 상태를 실시간으로 모니터링하는 자가 진단 기능을 수행하며, 비행 중 수집된 상태 보고 코드를 ALIS(Autonomic Logistics Information System)/ODIN(Operational Data Integrated Network) 지상 시스템으로 전송하여 예측 정비를 지원한다.27

이처럼 CIP는 항공기의 감각(센서), 소통(통신), 판단(처리), 표현(디스플레이)에 이르는 모든 과정을 총괄하는 명실상부한 F-35의 핵심이다.


CIP의 막대한 처리 능력이 만들어내는 가장 중요한 결과물이자 F-35의 전술적 우위를 정의하는 핵심 능력은 바로 '센서 융합(Sensor Fusion)'이다. 센서 융합은 단순히 여러 센서 정보를 한 화면에 보여주는 것을 넘어, F-35의 모든 센서로부터 수집된 원시 데이터를 실시간으로 자동 분석 및 연관 분석하여, 조종사에게 직관적이고 통합된 단일 전장 상황도를 제공하는 과정이다.29

- **데이터 처리 방식의 근본적 변화:** 4세대 전투기까지의 센서 융합은 각 센서가 독립적으로 표적을 탐지하고 '트랙(track)'을 생성한 후, 이 트랙 정보들을 중앙 컴퓨터에서 통합하는 방식이었다. 반면, F-35의 CIP는 각 센서로부터 트랙 파일이 아닌 가공되지 않은 '원시 측정 데이터(raw sensor measurements)'를 직접 받아, 이들을 한꺼번에 융합하여 단일 트랙을 생성한다.31 이 방식은 훨씬 더 풍부하고 정확하며 모호성이 적은 전장 상황을 제공하지만, 비교할 수 없을 정도로 막대한 연산 능력을 요구한다. 연합형 아키텍처에서는 데이터가 각 LRU에 분산되어 있어 이러한 방식의 융합이 물리적으로 불가능하다.
- **인공지능의 초기 형태:** 전문가들은 이처럼 방대한 데이터를 자동으로 분석, 연관 분석하고 종합하여 조종사에게 의미 있는 정보로 변환하는 과정을 "인공지능의 초기 반복(early iteration of artificial intelligence)"이라고 평가한다.32 CIP 내부의 정교한 컴퓨터 알고리즘은 수신되는 위협 정보를 기체에 내장된 '임무 데이터 파일(Mission Data Files)'이라는 방대한 위협 라이브러리와 실시간으로 대조하여, 조종사의 개입 없이도 표적을 식별하고 위협 수준을 판단하는 등의 자동화된 기능을 수행한다.33
- **전술적 효과:** 이 모든 과정은 조종사의 '인지 부하(cognitive load)'를 극적으로 줄여준다. 조종사는 더 이상 여러 센서 디스플레이를 번갈아 보며 상충되는 정보를 해석하는 '센서 관리자'가 아니라, CIP가 제공하는 통합된 정보를 바탕으로 전술적 결심에만 집중하는 '진정한 전술가(true tactician)'가 될 수 있다.32 F-35 조종사들이 이구동성으로 꼽는 F-35의 가장 "게임 체인징한 차이점"이 바로 이 센서 융합을 통한 압도적인 상황 인식 능력이다.32

결론적으로, IMA 아키텍처를 채택하고 모든 처리를 CIP로 중앙화한 F-35의 설계 결정은, 항공기의 가장 핵심적인 경쟁력인 센서 융합 능력을 구현하기 위한 필연적인 선택이었다. 센서 융합은 CIP 위에서 실행되는 여러 애플리케이션 중 하나가 아니라, CIP라는 아키텍처의 존재 이유이자 그 목적의 최종적인 발현이라고 할 수 있다. 이러한 능력은 F-35를 단순한 전투기를 넘어, 스텔스 성능을 갖춘 고기동성 비행 데이터 센터이자 네트워크 노드로 재정의한다.29 F-35의 핵심 임무는 단순히 운동에너지를 투사하는 것을 넘어, 전장의 정보를 수집, 처리, 공유하여 자기 자신은 물론 네트워크에 연결된 모든 아군 자산의 치명성과 생존성을 극대화하는 정보 우위의 확보로 확장된다.30 이는 제공권의 개념을 재정립하고 합동 전영역 작전(JADO)의 핵심 자산으로서 F-35의 전략적 가치를 규정하는 중요한 함의를 가진다.


F-35의 CIP가 전례 없는 임무를 수행하기 위해서는 하드웨어부터 운영체제, 소프트웨어 프레임워크에 이르기까지 모든 계층에서 극도의 성능과 신뢰성을 보장해야 한다. 이 장에서는 CIP를 구성하는 기술 스택을 심층적으로 해부하여, 어떻게 그토록 복잡하고 중요한 임무를 안정적으로 수행하는지 분석한다.


- **처리 하드웨어: COTS 기술의 적극적 활용:** CIP는 군용으로 특별 제작된 프로세서가 아닌, 상용 기성품(COTS) 기술을 적극적으로 활용한다. 특히 인텔(Intel)사의 칩셋을 포함한 고성능 멀티코어 프로세서가 그 기반을 이룬다.17 이는 완전 맞춤형 군용 하드웨어 개발에 따르는 막대한 비용과 시간을 절감하고, 상용 시장의 빠른 기술 발전 속도를 활용하기 위한 전략적 선택이다.38
- **이기종 컴퓨팅(Heterogeneous Computing):** CIP 아키텍처는 단순히 여러 개의 동일한 CPU 코어를 사용하는 것을 넘어, 그래픽 처리 장치(GPU), 필드 프로그래머블 게이트 어레이(FPGA) 등 서로 다른 종류의 처리 장치를 통합하는 이기종 컴퓨팅 환경을 지원한다.14 이를 통해 파노라마 칵핏 디스플레이의 복잡한 3D 그래픽 렌더링이나 레이더 신호 처리와 같은 특정 작업은 그에 최적화된 전문 프로세서에 맡겨 전체 시스템의 효율을 극대화한다.
- **열과의 전쟁: 첨단 열관리 솔루션:** 고성능 멀티코어 프로세서들을 좁고 밀폐된 군용 항공전자 장비함에 집적하면 엄청난 양의 열이 발생한다. 전통적인 팬(fan) 방식의 공랭식 냉각은 부피가 크고 소음이 심하며, 현대 항공전자가 요구하는 냉각 성능을 감당할 수 없다.41 따라서 F-35와 같은 첨단 항공기는 정교한 열관리 솔루션에 의존한다.
  - **액체 냉각 시스템(Liquid Cooling Systems):** 폴리알파올레핀(Polyalphaolefin, PAO)과 같은 냉각재를 순환시켜 발열 부품의 열을 직접 흡수하고 열교환기를 통해 외부로 방출하는 방식이 핵심이다.42 액체 냉각액이 흐르는 냉각판(cold plate)을 발열량이 큰 회로 기판에 직접 부착하거나, 200와트 이상의 고전력 카드를 냉각할 수 있도록 섀시 측면에 액체 냉각 채널을 설계한다.44
  - **직접 분사 냉각(Direct-Spray Cooling):** 비전도성 액체 냉매를 미세한 안개 형태로 고열 부품에 직접 분사하는 2상(two-phase) 냉각 기술이다. 액체가 기화하면서 부품의 열을 매우 효율적으로 빼앗아 냉각시킨다.44
  - **히트 파이프 및 루프 히트 파이프(LHP):** 진공관 내부의 작동 유체가 증발과 응축을 반복하며 열을 전달하는 수동적(passive) 2상 냉각 장치다. 별도의 펌프나 동력 없이 모세관 현상만으로 작동하며, 전자장비에서 발생한 열을 항공기 연료와 같은 다른 열 흡수원(heat sink)으로 효과적으로 전달하는 데 사용된다.46


CIP의 강력한 하드웨어를 제어하고 그 위에서 실행되는 수많은 소프트웨어의 안전과 보안을 보장하는 것은 실시간 운영체제(Real-Time Operating System, RTOS)의 몫이다. F-35는 이 중차대한 역할을 Green Hills Software사의 INTEGRITY-178B와 그 멀티코어 버전인 INTEGRITY-178 tuMP에 맡겼다.47

- **전례 없는 수준의 보안 인증:** INTEGRITY-178B는 미국 국가안보국(NSA)으로부터 EAL 6+ High Robustness 등급을 인증받은 유일한 운영체제다.47 이는 운영체제가 받을 수 있는 가장 높은 수준의 보안 평가로, 막대한 자금과 기술을 동원하는 국가 수준의 정교한 사이버 공격으로부터 기밀 정보를 보호할 수 있도록 설계되었음을 의미한다. F-35가 네트워크 중심전의 핵심 노드라는 점을 감안할 때, 이는 필수적인 요건이다.
- **최고 수준의 안전성 인증:** 동시에 이 RTOS는 미국 연방항공청(FAA)의 항공기 소프트웨어 안전성 표준인 RTCA/DO-178B/C의 가장 엄격한 등급인 레벨 A(DAL A) 인증을 준수한다.48 DAL A는 소프트웨어의 실패가 항공기 추락과 같은 치명적인(catastrophic) 결과를 초래할 수 있는 시스템에 요구되는 최고 안전 등급이다.
- **견고한 파티셔닝 강제:** INTEGRITY-178B 커널은 IMA 아키텍처가 요구하는 견고한 시간 및 공간 분할을 강제하도록 특별히 설계되었다. 특히, 예측 불가능성을 유발할 수 있는 동적 메모리 할당(dynamic memory allocation)과 같은 기능을 원천적으로 배제하여 모든 연산이 정해진 시간 안에 완료됨을 보장(bounded compute times)한다.48 이는 시스템의 결정론적(deterministic) 작동을 보장하는 핵심 요소다.


IMA가 개념적 틀을 제공하고 INTEGRITY-178B가 신뢰할 수 있는 실행 환경을 제공한다면, ARINC 653은 여러 공급업체가 개발한 다양한 소프트웨어 애플리케이션들이 이 환경에서 질서정연하게 공존할 수 있도록 하는 표준화된 규칙, 즉 '헌법'의 역할을 한다.9 ARINC 653은 애플리케이션 소프트웨어와 RTOS 간의 표준화된 인터페이스(API)를 정의한다.

- **견고한 파티셔닝의 표준화:** ARINC 653의 핵심은 혼합 중요도 시스템을 위한 파티셔닝 메커니즘을 표준화하는 것이다.
  - **공간 분할(Spatial Partitioning):** 표준은 각 애플리케이션(파티션)이 자신에게 할당된 보호된 메모리 공간만을 사용하도록 명시한다. 운영체제는 이 규칙을 강제하여 한 파티션의 버그나 악성 코드가 다른 파티션의 코드나 데이터를 손상시키는 것을 방지한다.14
  - **시간 분할(Temporal Partitioning):** 표준은 정적인 스케줄링 테이블에 따라 각 파티션에 고정된 CPU 시간 창(time window)을 할당하도록 규정한다. 이를 통해 특정 파티션이 과도한 연산으로 CPU를 독점하여 다른 파티션의 실시간성을 해치는 것을 막는다.14
- **모듈성 및 재사용성 촉진:** ARINC 653이라는 공통의 규칙이 존재함으로써, 서로 다른 공급업체들은 자사의 애플리케이션을 이 표준에 맞춰 독립적으로 개발하고 사전 검증(pre-qualify)할 수 있다.9 시스템 통합업체는 이렇게 개발된 '부품화된' 소프트웨어 파티션들을 가져와 마치 레고 블록처럼 조립하여 전체 시스템을 구성할 수 있다. 이는 전체 시스템의 통합 및 인증 과정을 크게 단순화하며, IMA의 핵심적인 비즈니스 모델을 가능하게 한다.

결론적으로 CIP의 기술 아키텍처는 그 자체로 하나의 거대한 시스템적 트레이드오프(trade-off)를 보여준다. 성능과 비용 효율성을 위해 상용 멀티코어 프로세서(COTS)를 채택하는 동시에 17, 이로 인해 발생하는 예측 불가능성과 간섭 문제를 해결하기 위해 막대한 노력을 기울여야 하는 것이다. 상용 칩은 본질적으로 항공전자 시스템이 요구하는 완벽한 결정론적 작동을 위해 설계되지 않았기 때문에 52, 코어 간 공유 캐시나 메모리 컨트롤러 같은 공유 자원은 '간섭 채널(interference channels)'로 작용하여 ARINC 653이 보장해야 할 시간적 격리성을 위협할 수 있다.52 이 근본적인 기술적 긴장 관계가 바로 F-35 프로그램이 첨단 열관리 시스템 44, EAL 6+ 등급의 초고신뢰성 운영체제 47, 그리고 비결정론적 하드웨어 위에서 최악 실행 시간(WCET)을 증명하기 위한 복잡한 분석 기법 56에 막대한 투자를 해야 하는 이유다. 이는 CIP 아키텍처가 시스템 수준에서 소프트웨어 정의 능력이라는 혁신적인 가치를 얻기 위해, 하드웨어와 운영체제 같은 하위 계층에서 엄청난 통합의 복잡성과 리스크를 감수하는 F-35 프로그램 전체의 철학을 압축적으로 보여준다. 이 도박의 성공은 전적으로 중간 계층(RTOS, ARINC 653, 인증 프로세스)이 하위 계층에서 발생하는 내재적 복잡성을 얼마나 성공적으로 통제하고 관리할 수 있느냐에 달려 있다.


CIP의 아키텍처가 아무리 혁신적이라 할지라도, 그 가치는 실제 개발, 통합, 현대화 과정의 성공 여부에 따라 결정된다. F-35의 핵심 현대화 프로그램인 기술 재생 3(Technology Refresh 3, TR-3)는 CIP의 기술적 도약과 동시에, 이러한 복잡한 시스템을 현실화하는 과정에서 발생하는 심각한 프로그램적 난제와 미로와도 같은 인증 절차를 극명하게 보여주는 사례다.


TR-3는 단순히 기존 시스템을 개선하는 수준을 넘어, F-35의 컴퓨팅 능력을 한 세대 발전시키는 핵심적인 하드웨어 및 소프트웨어 현대화 프로그램이다. 이는 향후 수십 년간 F-35의 전투 능력을 보장할 블록 4(Block 4) 업그레이드를 위한 필수적인 "개방형 아키텍처 기반(open architecture backbone)"으로 불린다.35 TR-3는 75개의 신규 프로그램을 포함하는 F-35 역사상 가장 공격적인 업데이트로 평가된다.35

TR-3의 핵심은 L3Harris가 공급하는 주요 항공전자 부품의 전면적인 업그레이드에 있다.24

- **통합 코어 프로세서(ICP):** 기존 TR-2 시스템 대비 **37배**에 달하는 압도적인 연산 능력을 제공한다. 이는 TR-3 업그레이드의 심장부라 할 수 있다.
- **항공기 메모리 시스템(AMS):** 저장 용량을 **20배**로 확장하여, 더욱 정교한 센서로부터 수집되는 방대한 양의 데이터를 저장하고 처리할 수 있게 한다.
- **파노라마 칵핏 디스플레이 전자 장치(PCD EU):** 조종석 디스플레이 처리 속도를 **5배** 향상시켜 조종사에게 더 빠르고 부드러운 시각 정보를 제공한다.

이처럼 막대한 처리 능력의 향상은 단순히 속도를 높이는 것을 넘어, 블록 4에서 계획된 새로운 첨단 무기체계, 차세대 센서, 그리고 고도의 인공지능 기반 알고리즘을 실행할 수 있는 컴퓨팅 마력을 제공하는 데 그 목적이 있다.58 또한, 향상된 소프트웨어 안정성, 신뢰성, 진단 기능을 통해 장기적으로는 유지보수 비용을 절감하는 효과도 기대된다.58

| 기능                              | 레거시 시스템 (TR-2) | 업그레이드 시스템 (TR-3) | 성능 향상 | 전략적 목적                                                  |
| --------------------------------- | -------------------- | ------------------------ | --------- | ------------------------------------------------------------ |
| **코어 프로세싱 (ICP)**           | 기준 처리 능력       | 기준 대비 37배 증가 58   | **37x**   | 블록 4의 첨단 알고리즘, 신규 무장 및 센서 구동을 위한 연산 능력 확보 59 |
| **데이터 저장 (AMS)**             | 기준 저장 용량       | 기준 대비 20배 증가 58   | **20x**   | 향상된 센서로부터 수집되는 방대한 데이터 저장 및 상황 인식 능력 확대 58 |
| **칵핏 디스플레이 처리 (PCD EU)** | 기준 처리 속도       | 기준 대비 5배 증가 58    | **5x**    | 진화하는 위협에 대응하기 위한 조종사 디스플레이 정보의 신속한 처리 및 시현 24 |


TR-3의 야심찬 계획과 달리, 실제 개발 및 통합 과정은 심각한 난관에 부딪혔다.

- **상당한 일정 지연:** 당초 2023년 7월부터 TR-3가 탑재된 기체의 인도가 시작될 예정이었으나, 실제로는 거의 1년 가까이 지연된 2024년 중반에야 첫 인도가 이루어졌다.59
- **지연의 근본 원인:**
  - **공급업체 개발 지연:** 핵심 부품인 ICP를 개발하는 L3Harris에서 하드웨어 및 소프트웨어 개발과 관련하여 "예상치 못한 문제"가 발생하며 1차적인 지연을 초래했다.59
  - **소프트웨어 불안정성:** 하드웨어가 납품된 이후에도 록히드 마틴은 소프트웨어를 안정적으로 통합하는 데 어려움을 겪었다. 2024년 초, 미군 합동 시험팀(UOTT)은 불안정한 소프트웨어 성능, 핵심적인 결함, 지속적인 임무 시스템 오류 등을 이유로 시험 중단(stop test) 명령을 내리기까지 했다.60
  - **기능 축소 인도:** 결국 미 공군은 생산 라인을 계속 가동시키기 위해, 기존 TR-2 항공기에서는 정상 작동하던 일부 핵심 전투 기능이 비활성화된 "기능이 축소된 소프트웨어(truncated software)"가 탑재된 TR-3 기체를 인수하기 시작했다.60 이는 단기적인 전투 능력의 후퇴를 감수하면서까지 하드웨어를 우선적으로 현장에 배치하려는 고육지책으로, 통합 문제의 심각성을 방증한다.

이러한 문제는 F-35의 자율군수정보시스템(ALIS)과 그 후속 시스템인 ODIN에서 나타난 문제와 놀라울 정도로 유사하다. ALIS는 비직관적인 사용자 인터페이스, 부정확한 데이터, 높은 오경보율, 거대한 물리적 서버 등으로 인해 지속적으로 비판받아왔다.61 ODIN으로의 전환 역시 예산 부족과 기술적 문제로 지연되고 있으며, 초기 단계는 문제가 많은 ALIS 소프트웨어를 새로운 클라우드 기반 인프라로 단순히 옮기는 '리프트 앤 시프트(lift and shift)'에 그쳐 근본적인 문제 해결과는 거리가 멀다는 지적을 받고 있다.60


CIP와 같은 복잡한 시스템이 실제 항공기에 탑재되어 비행하기까지는 여러 단계의 엄격하고 복잡한 인증 과정을 통과해야 한다.

- **DO-297: 프로세스 청사진:** 이 표준은 IMA 시스템 개발 생태계에 참여하는 다양한 주체들의 **역할과 책임**을 정의하는 지침서다. 플랫폼 공급업체, 애플리케이션 공급업체, 시스템 통합업체, 최종 인증 신청자 등 여러 회사가 협업하는 IMA 환경에서 각자의 역할을 명확히 규정하여 원활한 개발과 인증을 돕는다.64

- **DO-178C: 소프트웨어의 바이블:** 항공용 소프트웨어 인증을 위한 핵심 표준이다. 시스템에 미치는 고장의 심각도에 따라 설계 보증 수준(DAL)을 A등급(치명적)부터 E등급(영향 없음)까지 5단계로 나눈다. CIP의 핵심 RTOS와 비행 제어와 같은 중요 애플리케이션은 가장 엄격한 DAL A 요건을 충족해야 한다.67

- **DO-254: 하드웨어의 규칙서:** FPGA, ASIC과 같은 복잡한 전자 하드웨어에 대한 설계 보증 지침을 제공하는 표준이다. 이 역시 DAL 프레임워크를 사용하며, CIP에 포함된 다양한 맞춤형 반도체 부품들이 이 표준에 따라 개발되고 검증된다.70

- **멀티코어 인증의 도전:** 멀티코어 프로세서를 안전필수 시스템에 사용하는 것은 인증 분야의 가장 큰 난제 중 하나다. FAA의 CAST-32A와 같은 지침서는 코어 간 간섭 채널, 자원 경합 등 멀티코어 고유의 문제를 해결하기 위한 지침을 제공한다.55 특히, 다른 코어의 활동으로 인해 특정 작업의 실행 시간이 예측 불가능하게 늘어나는 문제를 분석하는 

  **최악 실행 시간(Worst-Case Execution Time, WCET) 분석**이 핵심 과제다. 간섭으로 인해 WCET가 "터무니없이 커질(absurdly large)" 수 있기 때문에 57, 정적 분석만으로는 한계가 있으며 '간섭 생성기(interference generators)'를 이용한 측정 기반의 새로운 검증 기법이 요구된다.56


F-35의 막강한 능력은 소프트웨어와 네트워크 중심 설계에서 비롯되지만, 바로 그 지점이 가장 큰 취약점이 될 수도 있다.

- **양날의 검:** CIP를 중심으로 고도로 연결된 F-35의 아키텍처는 전례 없는 성능을 제공하는 동시에, 해커에게는 광범위한 공격 표면(attack surface)을 노출한다.34 하나의 노드(node) 침투가 여러 시스템에 연쇄적인 영향을 미칠 수 있는 잠재적 위험을 내포하고 있다.34
- **내재된 보안(Baked-In Security):** 이러한 위험에 대응하기 위해 F-35 프로그램은 사이버 보안을 나중에 추가하는 IT 문제로 취급하지 않고, 설계 초기 단계부터 시스템에 "내재화(bake-in)"하는 접근법을 채택한 대표적인 사례로 꼽힌다.34
- **아키텍처 수준의 방어 메커니즘:**
  - **고신뢰 운영체제(High-Assurance RTOS):** EAL 6+ 인증을 받은 INTEGRITY-178B RTOS는 시스템의 가장 낮은 수준에서부터 비인가 접근을 차단하고 시스템 무결성을 보장하는 근본적인 보안 통제 수단이다.47
  - **ARINC 653 파티셔닝:** ARINC 653이 제공하는 강력한 시간 및 공간 분할은 그 자체로 효과적인 보안 메커니즘이다. 특정 파티션에서 발생한 오류나 악성 코드가 다른 파티션으로 전파되는 것을 원천적으로 차단함으로써, 일종의 마이크로 세분화(micro-segmentation)를 구현하여 피해 확산을 막는다.15
  - **개방형 아키텍처와 디지털 엔지니어링:** 모델 기반 시스템 엔지니어링(MBSE)과 같은 디지털 엔지니어링 기법의 도입은 시스템 설계에 대한 깊은 통찰력을 제공하여, 개발 초기 단계부터 위협을 모델링하고 완화 전략을 통합하는 것을 가능하게 한다.76

결론적으로 F-35 프로그램의 고도로 통합된 소프트웨어 중심 접근 방식은 TR-3 ICP와 같은 단일 핵심 부품의 개발 지연이 블록 4 현대화 전체를 지연시키는 연쇄적이고 프로그램 전반에 걸친 영향을 초래하는 '취약한(brittle)' 개발 경로를 만들어냈다. 이는 F-35의 상호의존적인 시스템 아키텍처에서 단일 장애점(single point of failure)이 얼마나 불균형적으로 큰 파급 효과를 낳을 수 있는지 보여주는 명백한 증거다. TR-3와 ALIS/ODIN에서 지속적으로 나타나는 문제들은 단순한 개발 지연을 넘어, IMA와 소프트웨어 정의 시스템이 약속하는 막대한 잠재력을 실현하기 위해 치러야 하는 '복잡성 세금(complexity tax)'을 상징한다. 이러한 시스템들을 검증, 인증, 통합하는 데 드는 비용과 시간, 그리고 리스크는 지속적으로 과소평가되어 왔으며, 아키텍처의 비전(원활한 통합)과 프로그램의 현실(지속적인 버그, 지연, 임시방편) 사이의 격차는 F-35 프로그램의 핵심적인 과제로 남아있다. 이는 미래 6세대 전투기 개발의 가장 큰 도전이 첨단 기능을 구상하는 것 자체가 아니라, 이를 실제로 감당 가능한 비용과 일정 내에 구현할 수 있는 공학 및 프로그램 관리 방법론을 확립하는 것임을 시사한다.


CIP와 그 기반이 되는 IMA 철학은 F-35라는 특정 플랫폼을 넘어, 항공전의 미래와 방위 산업 생태계 전반에 걸쳐 근본적인 변화를 이끌고 있다. 이는 소프트웨어 정의 전쟁, 인공지능의 역할, 그리고 새로운 국방 획득 비즈니스 모델의 등장을 예고한다.


CIP에 의해 구동되는 F-35는 '소프트웨어 정의 항공기(Software-Defined Aircraft)'라는 개념을 완벽하게 구현한 사례다.78 이제 항공기의 성능은 기체의 물리적 형상이나 엔진의 추력만으로 결정되는 것이 아니라, 그 위에서 실행되는 소프트웨어에 의해 정의된다.

- **지속적인 능력 향상:** 이러한 패러다임은 완전히 새로운 개발 모델을 가능하게 한다. 과거처럼 수십 년 주기로 완전히 새로운 기체를 개발하는 대신, 소프트웨어 '드롭(drop)' 또는 '블록(Block)' 단위의 반복적인 업그레이드를 통해 지속적으로 새로운 능력을 추가하고 위협에 대응한다.34 F-35의 블록 4 현대화가 바로 그 예다.
- **애자일(Agile) 및 데브섹옵스(DevSecOps) 도입:** 이러한 빠른 반복 주기를 지원하기 위해, F-35 프로그램은 전통적인 폭포수(waterfall) 방식에서 벗어나 애자일 소프트웨어 개발 방법론과 데브섹옵스(DevSecOps)를 도입했다. 이를 통해 소프트웨어 배포 주기를 기존 18개월에서 수개월, 심지어 6주까지 단축하는 것을 목표로 하고 있다.79 이는 기존의 경직된 국방 획득 절차에 대한 근본적인 문화적, 절차적 도전이다.
- **디지털 엔지니어링(Digital Engineering):** 이 모든 변화의 기저에는 문서 중심의 설계에서 모델 기반 시스템 엔지니어링(MBSE)과 디지털 트윈(Digital Twin)으로의 전환이 있다.76 설계부터 생산, 운용, 유지보수에 이르는 전 과정을 '디지털 스레드(digital thread)'로 연결함으로써, 변경 사항의 영향을 신속하고 정확하게 분석하고, 여러 팀 간의 협업을 강화하며, 개발 속도를 획기적으로 높일 수 있다.81


TR-3와 그 이후 세대의 IMA 플랫폼이 제공하는 막대한 처리 능력은 인공지능(AI)과 머신러닝(ML) 애플리케이션을 항공기에 본격적으로 탑재하기 위한 기반이다.82

- **미래 AI 적용 분야:**
  - **지능형 센서 융합 및 결심 지원:** 현재의 센서 융합을 넘어, AI를 활용하여 위협을 예측하고, 최적의 교전 순서를 추천하며, 조종사의 전술적 결심을 지원하는 단계로 발전할 것이다.82
  - **자율 시스템(Autonomous Systems):** IMA는 무인 항공기(UAV)나 '충성스러운 윙맨(loyal wingman)' 개념의 핵심 기술이다. 자율 항법과 제어에 필요한 고신뢰성 인증 컴퓨팅 플랫폼을 제공함으로써 유무인 복합 전투체계의 구현을 가능하게 한다.21
  - **예측 정비(Predictive Maintenance):** 운용 및 상태 데이터를 AI로 분석하여 부품의 고장을 사전에 예측하고 정비 소요를 최적화함으로써, 항공기 가용성을 극대화하고 군수 지원 체계를 혁신할 수 있다.21
- **신뢰의 문제: 설명가능 AI (Explainable AI, XAI):** AI가 조종사의 생명과 직결되는 중요한 결정을 내리게 되면서, '블랙박스(black box)'와 같은 AI의 의사결정 과정을 인간이 이해하고 신뢰할 수 있도록 만드는 것이 핵심 과제로 떠올랐다. XAI는 AI의 판단 근거를 인간이 이해할 수 있는 형태로 제시하는 기술 분야로 85, 자율 전투 시스템의 윤리적, 법적, 안전성 문제를 해결하고 최종적으로 인증을 획득하기 위한 필수 전제 조건이다.88 이와 더불어, 신경망의 안전성과 강건성을 수학적으로 증명하는 형식 검증(Formal Verification) 기술 또한 첨단 항공전자 시스템의 신뢰성을 확보하기 위한 중요 연구 분야로 부상하고 있다.89


IMA와 개방형 아키텍처 표준의 결합은 방위 산업의 구조 자체를 근본적으로 바꾸고 있다.

- **FACE™ 컨소시엄:** 미래 공중 능력 환경(Future Airborne Capability Environment, FACE)은 미 정부와 방산업계가 협력하여 군용 항공전자를 위한 표준화된 개방형 아키텍처를 만드는 컨소시엄이다.92
- **새로운 비즈니스 모델:** FACE는 모바일 '앱 스토어'와 유사한 비즈니스 모델을 추구한다.95
  - **표준화된 아키텍처:** 운영체제 세그먼트(OSS), 플랫폼 특화 서비스 세그먼트(PSSS) 등 표준화된 소프트웨어 계층과 그 사이의 인터페이스를 정의한다.92
  - **재사용 가능한 소프트웨어 부품:** 제3의 소프트웨어 공급업체들은 이 표준에 맞춰 이식성과 재사용성이 높은 소프트웨어 부품, 즉 '적합성 단위(Unit of Conformance, UoC)'를 개발할 수 있다.92
  - **디지털 라이브러리:** 개발된 UoC는 적합성 인증을 거쳐 FACE 레지스트리(Registry)라는 일종의 디지털 라이브러리에 등록된다.92
  - **신속한 통합:** 체계 통합업체나 군 프로그램 관리자는 이 레지스트리에서 필요한 기능의 UoC를 선택하여, 마치 앱을 다운로드하듯 신속하게 새로운 능력을 조합하고 전장에 배치할 수 있다. 이는 특정 업체에 대한 종속성(vendor lock-in)을 깨고 중복 투자를 줄이는 효과를 가져온다.98
- **중소기업을 위한 기회:** 이러한 모델은 거대 방산기업이 아니더라도 혁신적인 기술을 가진 중소기업(SME)들이 전체 항공전자 스택을 개발할 필요 없이 특정 기능의 소프트웨어 '앱'을 개발하여 시장에 진입할 수 있는 기회를 창출한다.99 이는 방위 산업 생태계의 다양성을 증진하고 경쟁을 촉진하여 혁신을 가속화하는 원동력이 될 수 있다.

결론적으로, CIP로 대표되는 기술적 아키텍처(IMA), 애자일/데브섹옵스로 대표되는 개발 방법론, 그리고 FACE로 대표되는 비즈니스 모델은 서로 분리된 주제가 아니라, 하나의 통일된 전략을 구성하는 세 개의 기둥이다. FACE의 '앱 스토어' 모델은 IMA와 ARINC 653이라는 표준화된 기술 아키텍처 없이는 불가능하며 9, 소프트웨어 정의 항공기가 추구하는 신속하고 반복적인 능력 업그레이드는 애자일 방법론 없이는 달성할 수 없다.79 따라서 CIP는 단순한 하드웨어 부품이 아니라, 새로운 소프트웨어 개발 방식과 새로운 산업 협력 모델이 구축되는 기술적 토대 그 자체다.

궁극적인 전략적 함의는 미래 공중전의 우위가 더 이상 기체의 물리적 성능 경쟁에만 머무르지 않고, 새로운 소프트웨어 능력을 얼마나 빠르고 정교하게 개발, 인증, 배포할 수 있는가 하는 '소프트웨어 개발 및 통합 파이프라인'의 속도와 효율성 경쟁으로 전환되고 있다는 점이다. TR-3와 ODIN에서 F-35 프로그램이 겪고 있는 어려움은 이 파이프라인을 구축하는 것이 얼마나 힘든지를 보여주지만, 그 잠재적 보상은 군사 항공우주 분야에서 새로운 전략적 고지를 점령하는 것이다. 향후 10년간의 핵심 과제는 단순히 더 나은 전투기를 만드는 것이 아니라, 더 나은 '능력 공장(capability factory)'을 구축하는 것이 될 것이다.


F-35의 공통 통합 프로세서(CIP)는 5세대 전투기의 능력을 정의하는 핵심 기술을 넘어, 현대 항공전자 시스템 설계와 국방 획득 패러다임의 근본적인 전환을 상징하는 결정체다. 본 보고서는 CIP의 기원, 기술적 구조, 기능, 그리고 전략적 함의를 다각도로 분석하였으며, 다음과 같은 핵심 결론을 도출하였다.

첫째, **CIP의 등장은 필연적이었다.** 과거의 연합형 아키텍처는 기능이 추가될수록 무게, 부피, 전력 소모, 배선 복잡성이 기하급수적으로 증가하여 물리적, 경제적 한계에 도달했다. CIP의 기반이 된 통합 모듈형 항공전자(IMA)는 이러한 지속 불가능한 위기를 해결하기 위한 유일한 대안이었으며, 자원 공유, 모듈화, 표준화를 통해 SWaP 및 생애주기비용을 획기적으로 절감하는 정량적 이점을 제공했다.

둘째, **CIP는 단순한 중앙 처리 장치가 아니라, F-35의 전투 능력을 창출하는 '융합 엔진'이다.** CIP의 중앙 집중식 아키텍처는 모든 센서의 원시 데이터를 한곳에서 실시간으로 처리하는 것을 가능하게 했고, 이는 F-35의 가장 큰 전술적 우위인 '센서 융합'을 구현하는 핵심 전제 조건이 되었다. 이 과정은 초기 형태의 인공지능으로 평가받으며, 조종사의 인지 부하를 줄이고 그를 '센서 관리자'에서 '전술가'로 변모시켰다. 즉, F-35는 CIP를 통해 비행하는 컴퓨터이자 정보 네트워크의 핵심 노드로 재탄생했다.

셋째, **CIP의 혁신은 막대한 '복잡성 세금'을 동반한다.** 고성능 상용 멀티코어 프로세서의 성능과 안전필수 시스템이 요구하는 결정론적 신뢰성 사이의 내재적 긴장 관계는 CIP 아키텍처의 핵심적인 기술적 난제다. 이를 해결하기 위해 첨단 열관리 기술, 세계 최고 수준의 보안 및 안전 인증을 받은 RTOS(INTEGRITY-178B), 그리고 ARINC 653과 같은 복잡한 표준과 인증 절차가 동원되었다. TR-3 프로그램에서 나타난 심각한 개발 및 통합 지연은 이러한 복잡성을 관리하는 것이 얼마나 어려운지를 명백히 보여주며, 이는 F-35 프로그램 전반의 비용 증가와 일정 지연의 주요 원인으로 작용하고 있다.

넷째, **CIP는 '소프트웨어 정의 전쟁' 시대를 여는 기술적 토대다.** CIP의 개방형 아키텍처와 강력한 처리 능력은 항공기의 성능이 하드웨어가 아닌 소프트웨어에 의해 정의되는 새로운 시대를 열었다. 이는 애자일, 데브섹옵스와 같은 신속한 개발 방법론의 도입을 촉진하고, FACE 컨소시엄이 주도하는 '앱 스토어' 형태의 새로운 비즈니스 모델을 가능하게 한다. 이를 통해 방위 산업 생태계는 더욱 개방적이고 경쟁적으로 변화하며, 중소기업에게도 새로운 혁신의 기회를 제공할 것이다.

궁극적으로, F-35와 CIP의 사례는 미래 공중전의 승패가 더 이상 기체의 성능만으로 결정되지 않음을 보여준다. 이제 경쟁의 핵심은 위협 변화에 대응하여 얼마나 빠르고 효율적으로 새로운 소프트웨어 능력을 개발, 인증, 그리고 전장에 배포할 수 있는가 하는 '능력 개발 파이프라인'의 우위에 달려있다. F-35 프로그램이 겪고 있는 고난은 이 파이프라인을 구축하는 것이 얼마나 어려운지를 증명하지만, 동시에 그 파이프라인을 성공적으로 구축하는 국가와 동맹이 미래의 하늘을 지배하게 될 것임을 명확히 예고하고 있다. 따라서 CIP에 대한 고찰은 단순히 하나의 부품에 대한 기술 분석을 넘어, 미래 국방력의 향방을 가늠하는 중요한 시금석이 된다.


1. 항공용 운영체제 기술 동향, accessed July 12, 2025, https://ettrends.etri.re.kr/ettrends/140/0905001805/28-2_020-028.pdf
2. Transitioning from federated avionics architectures to Integrated ..., accessed July 12, 2025, https://www.researchgate.net/publication/4295132_Transitioning_from_federated_avionics_architectures_to_Integrated_Modular_Avionics
3. 항공 전자 시스템의 통합 아키텍처 ✈️ - 재능넷, accessed July 12, 2025, https://www.jaenung.net/tree/20376
4. Obsolescence and Life Cycle Management for Avionics - FAA, accessed July 12, 2025, https://www.faa.gov/sites/faa.gov/files/aircraft/air_cert/design_approvals/air_software/TC-15-33.pdf
5. [MSA 살펴보기] (feat. Monolithic) - 한 걸음씩 기록하며 - 티스토리, accessed July 12, 2025, https://haksae.tistory.com/216
6. [MSA] MSA란 무엇인가? 개념 이해하기 - 개발개발 울었다 - 티스토리, accessed July 12, 2025, https://wooaoe.tistory.com/57
7. MSA 아키텍쳐와 모놀리식 아키텍쳐 - velog, accessed July 12, 2025, [https://velog.io/@jung5318/MSA-%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90%EC%99%80-%EB%AA%A8%EB%86%80%EB%A6%AC%EC%8B%9D-%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90](https://velog.io/@jung5318/MSA-아키텍쳐와-모놀리식-아키텍쳐)
8. [아키텍처] 모놀리식 아키텍처 VS 마이크로 서비스 아키텍처(MSA) - 개발 메모용 블로그, accessed July 12, 2025, https://memodayoungee.tistory.com/155
9. Integrated modular avionics - Wikipedia, accessed July 12, 2025, https://en.wikipedia.org/wiki/Integrated_modular_avionics
10. Improving Hazard Analysis and Certification of Integrated Modular Avionics | Journal of Aerospace Information Systems, accessed July 12, 2025, https://arc.aiaa.org/doi/10.2514/1.I010164
11. (PDF) Integrated Modular Avionics - Past, present, and future - ResearchGate, accessed July 12, 2025, https://www.researchgate.net/publication/284113137_Integrated_Modular_Avionics_-_Past_present_and_future
12. IMA in Avionics: A Comprehensive Guide, accessed July 12, 2025, https://www.numberanalytics.com/blog/ultimate-guide-ima-avionics
13. A Streamlined Approach Toward Automated Generation and Validation of ARINC 653–Compliant Avionics Configurations | Journal of Aerospace Information Systems, accessed July 12, 2025, https://arc.aiaa.org/doi/10.2514/1.I011439
14. Safety-Critical Software Development for Integrated Modular Avionics - Wind River Systems, accessed July 12, 2025, https://www.windriver.com/resource/safety-critical-software-development-for-integrated-modular-avionics
15. ARINC 653 API and its application – An insight into Avionics System Case Study, accessed July 12, 2025, https://publications.drdo.gov.in/ojs/index.php/dsj/article/download/4268/2490/0
16. (PDF) SWaP Reduction: Vital for Choice of Avionics Architecture, accessed July 12, 2025, https://www.researchgate.net/publication/308344854_SWaP_Reduction_Vital_for_Choice_of_Avionics_Architecture
17. Safety-Critical Software Development for Integrated Modular Avionics - Wind River, accessed July 12, 2025, https://www.windriver.com/sites/default/files/2022-02/458226011.pdf
18. Orion Integrated Modular Avionics (IMA) - Honeywell Aerospace, accessed July 12, 2025, https://aerospace.honeywell.com/content/dam/aerobt/en/documents/learn/platforms/brochures/N61-1331-000-000_Orion_IMA.pdf
19. Integrated Modular Avionics: Less Is More | PDF | Aeronautics ..., accessed July 12, 2025, https://www.scribd.com/doc/134630616/ATA-42-IMA
20. Modular Avionics Safety-Critical Software Development for Integrated - Wind River, accessed July 12, 2025, [https://www.windriver.com/sites/default/files/2023-06/514450%20-%20Safety-Critical%20Software%20Development%20for%20Integrated%20Modular%20Avionics_Whitepaper.pdf](https://www.windriver.com/sites/default/files/2023-06/514450 - Safety-Critical Software Development for Integrated Modular Avionics_Whitepaper.pdf)
21. The Future of Avionics: IMA Technology - Number Analytics, accessed July 12, 2025, https://www.numberanalytics.com/blog/future-avionics-ima-technology
22. The main design challenge of modern avionics - ResearchGate, accessed July 12, 2025, https://www.researchgate.net/figure/The-main-design-challenge-of-modern-avionics_fig1_269695371
23. A Distributed Integrated Modular Avionics platform for Multi-Mission Vehicles - Fenix, accessed July 12, 2025, https://fenix.tecnico.ulisboa.pt/downloadFile/281870113705295/Thesis.pdf
24. F-35 Lightning II Avionics | L3Harris® Fast. Forward., accessed July 12, 2025, https://www.l3harris.com/all-capabilities/f-35-lightning-ii-avionics
25. F-22 CIP mission computers upgraded with open systems architecture modules - Reddit, accessed July 12, 2025, https://www.reddit.com/r/FighterJets/comments/1hgnzmo/f22_cip_mission_computers_upgraded_with_open/
26. F-35 Lightning II - Northrop Grumman, accessed July 12, 2025, https://www.northropgrumman.com/what-we-do/aircraft/f35-lightning
27. F-35 Joint Strike Fighter JSF Autonomic Logistics Information System ALIS | www.dau.edu, accessed July 12, 2025, https://www.dau.edu/cop/log/documents/f-35-joint-strike-fighter-jsf-autonomic-logistics-information-system-alis
28. F-35 Joint Strike Fighter Autonomic Logistics and Prognostics & Health Management F-35 Joint Strike Fighter Autonomic Logist - SAE International, accessed July 12, 2025, https://www.sae.org/events/dod/presentations/2006nealmccollom.pdf
29. F-35 Lightning II - Lockheed Martin, accessed July 12, 2025, https://lockheedmartin.com/content/dam/lockheed-martin/aero/f35/documents/F35Brochure9_2022.pdf
30. Outsight In: F-35 Sensor Fusion in Focus, accessed July 12, 2025, https://www.f35.com/f35/news-and-features/f35-sensor-fusion-in-focus.html
31. F-35 Sensor Fusion in UNPRECEDENTED Detail - YouTube, accessed July 12, 2025, https://www.youtube.com/watch?v=brY2wzVAcVM
32. Pilots praise F-35 for advanced sensor fusion - DevX, accessed July 12, 2025, https://www.devx.com/daily-news/pilots-praise-f-35-for-advanced-sensor-fusion/
33. Sensor Fusion: The Secret Sauce that Makes the F-35 Revolutionary - The National Interest, accessed July 12, 2025, https://nationalinterest.org/blog/buzz/sensor-fusion-secret-sauce-makes-f-35-revolutionary-192873
34. The F-35's Cyber Reliance Makes It Powerful-But Also Vulnerable to Attack, accessed July 12, 2025, https://nationalinterest.org/blog/buzz/f-35s-cyber-reliance-makes-it-powerful-also-vulnerable-attack-198014/
35. Lockheed Martin F-35 TR3 Upgrade Approaches Combat Capability ..., accessed July 12, 2025, https://www.ainonline.com/aviation-news/defense/2025-06-16/f-35-upgrade-approaches-combat-capability
36. How the F-35 Connects the Battlespace - Lockheed Martin, accessed July 12, 2025, https://www.lockheedmartin.com/en-us/news/features/2025/how-the-f35-connects-the-battlespace.html
37. F-35 Integrated Core Processors - GlobalSpec, accessed July 12, 2025, https://www.globalspec.com/industrial-directory/f-35_integrated_core_processors
38. Life cycle management: The COTS perspective - Military Embedded Systems, accessed July 12, 2025, https://militaryembedded.com/avionics/computers/life-cycle-management-cots-perspective
39. IMA architecture for avionics systems Two classes of major modules have... - ResearchGate, accessed July 12, 2025, https://www.researchgate.net/figure/MA-architecture-for-avionics-systems-Two-classes-of-major-modules-have-been-defined-for_fig2_220833356
40. Safety Island in a Mixed Safety-Critical Heterogeneous Architecture - Arm Developer, accessed July 12, 2025, [https://developer.arm.com/Additional%20Resources/Video%20Tutorials/DevHub/Safety%20Island%20in%20a%20Mixed%20Safety-Critical%20Heterogeneous%20Architecture](https://developer.arm.com/Additional Resources/Video Tutorials/DevHub/Safety Island in a Mixed Safety-Critical Heterogeneous Architecture)
41. Thermal Management Solutions for High-Performance Aviation and Military Electronics, accessed July 12, 2025, https://imtron.com/blogs/blog/thermal-management-solutions-for-high-performance-aviation-and-military-electronics
42. Liquid Cooling Systems for Aerospace - AMETEK PDT, accessed July 12, 2025, https://www.pd-tech.com/products/thermal-management-systems
43. Liquid Cooling for Defense Market - AMETEK PDT, accessed July 12, 2025, https://www.pd-tech.com/markets/defense
44. Aerospace/ Military Thermal Management Systems Selection Guide - Electronics Cooling, accessed July 12, 2025, http://s3.electronics-cooling.com/wp-content/uploads/2015/03/Thermal-Solutions.pdf
45. Liquid cooling - Enercon Technologies, Ltd. | Military Power Supplies and Networking Field-Proven Solutions, accessed July 12, 2025, https://enercon.co.il/products/thermal-management/liquid-cooling
46. Passive Thermal Management for Avionics in High Temperature Environments - Advanced Cooling Technologies, accessed July 12, 2025, https://www.1-act.com/wp-content/uploads/2015/05/Passive-Thermal-Management-for-a-Avionics-in-High-Temperature-Environments.pdf
47. INTEGRITY: The First EAL6+ Operating System Technology - Green Hills Software, accessed July 12, 2025, https://www.ghs.com/security/security_home.html
48. Integrity (operating system) - Wikipedia, accessed July 12, 2025, https://en.wikipedia.org/wiki/Integrity_(operating_system)
49. Lockheed Martin is using Green Hills Software's INTEGRITY-178 RTOS, AdaMULTIIDE for the F35 JSF, accessed July 12, 2025, https://www.ghs.com/customers/lockheedf35.html
50. INTEGRITY-178 tuMP RTOS: Proven Expertise - Green Hills Software, accessed July 12, 2025, https://www.ghs.com/products/safety_critical/integrity_178_pedigree.html
51. Green Hills Software Announces Additional High Assurance Software Components for its Certified and Field-Proven INTEGRITY-178B RTOS - Vita Technologies, accessed July 12, 2025, https://vita.militaryembedded.com/2445-green-certified-field-proven-integrity-178b-rtos/
52. Update on using multicore processors with a ... - Wind River, accessed July 12, 2025, https://www.windriver.com/sites/default/files/2022-02/422204601.pdf
53. Assurance of Multicore Processors in Airborne Systems - FAA, accessed July 12, 2025, https://www.faa.gov/sites/faa.gov/files/aircraft/air_cert/design_approvals/air_software/TC-16-51.pdf
54. Challenges in Future Avionic Systems on Multi-Core Platforms - DiVA portal, accessed July 12, 2025, https://www.diva-portal.org/smash/get/diva2:929730/FULLTEXT01.pdf
55. Assured Multicore Partitioning for FACE Systems - Rapita Systems, accessed July 12, 2025, https://www.rapitasystems.com/blog/assured-multicore-partitioning-face-systems
56. Measuring the Impact of Interference Channels on Multicore Avionics - ResearchGate, accessed July 12, 2025, https://www.researchgate.net/publication/347045024_Measuring_the_Impact_of_Interference_Channels_on_Multicore_Avionics
57. Paper Title (use style - arXiv, accessed July 12, 2025, https://arxiv.org/pdf/2101.02204
58. TR-3: THE OPEN ARCHITECTURE BACKBONE OF THE ... - L3Harris, accessed July 12, 2025, https://www.l3harris.com/sites/default/files/2023-02/F-35-TR-3-Infographic-sas-62431_web.pdf
59. Deliveries of modernized F-35 TR-3 to be postponed until 2024 - Aviacionline, accessed July 12, 2025, https://www.aviacionline.com/deliveries-of-modernized-f-35-tr-3-to-be-postponed-until-2024
60. F-35 program faces delays and software challenges - Defence Blog, accessed July 12, 2025, https://defence-blog.com/f-35-program-faces-delays-and-software-challenges/
61. The F-35: ALIS in the Looking-Glass | U.S. GAO, accessed July 12, 2025, https://www.gao.gov/blog/f-35-alis-looking-glass
62. F-35 Testing Report Reveals Problems with Production Decisions, accessed July 12, 2025, https://www.pogo.org/analysis/f-35-testing-report-reveals-problems-with-production-decisions
63. Transition to New F-35 Logistics System Hits Headwinds - National Defense Magazine, accessed July 12, 2025, https://www.nationaldefensemagazine.org/articles/2021/5/26/transition-to-new-f-35-logistics-system-hits-headwinds
64. DO-297 - Wikipedia, accessed July 12, 2025, https://en.wikipedia.org/wiki/DO-297
65. DO-297 Training | IMA Development Guidance - Tonex Training, accessed July 12, 2025, https://www.tonex.com/training-courses/do-297-training/
66. Why Aerospace Still Needs DO-297 - Tonex Training, accessed July 12, 2025, https://www.tonex.com/why-aerospace-still-needs-do-297/
67. What Is DO-178C? - Wind River Systems, accessed July 12, 2025, https://www.windriver.com/solutions/learning/do-178c
68. DO-178C - Wikipedia, accessed July 12, 2025, https://en.wikipedia.org/wiki/DO-178C
69. AMETEK Abaco Systems FORCE2C Mission Computer Achieves DO-254 and DO-178C Safety Audit Certifications, accessed July 12, 2025, https://www.abaco.com/news/ametek-abaco-systems-force2c-mission-computer-achieves-do-254-and-do-178c-safety-audit
70. DO-178C and DO-254 Explained - PTC, accessed July 12, 2025, https://www.ptc.com/en/blogs/alm/do178c-and-do254-explained
71. DO-254 Explained: Design Assurance Guidance for Airborne Electronic Hardware, accessed July 12, 2025, https://visuresolutions.com/aerospace-and-defense/do-254
72. What does it take to certify multicore avionics for DO-178C?, accessed July 12, 2025, https://www.his-conference.co.uk/session/what-does-it-take-to-certify-multicore-avionics-for-do-178c
73. CCC_12_006898-REV07 - MULCORS Final Report.pdf - EASA, accessed July 12, 2025, [https://www.easa.europa.eu/sites/default/files/dfu/CCC_12_006898-REV07%20-%20MULCORS%20Final%20Report.pdf](https://www.easa.europa.eu/sites/default/files/dfu/CCC_12_006898-REV07 - MULCORS Final Report.pdf)
74. Cyber-Security Challenges in Aviation Industry: A Review of Current and Future Trends, accessed July 12, 2025, https://www.mdpi.com/2078-2489/13/3/146
75. ARINC 653 | 235 Publications | 1490 Citations | Top Authors | Related Topics - SciSpace, accessed July 12, 2025, https://scispace.com/topics/arinc-653-29tjgo7b
76. Empowering F-35 Digital Engineering Transformation - Booz Allen, accessed July 12, 2025, https://www.boozallen.com/insights/digital-engineering/empowering-f-35-digital-engineering-transformation.html
77. An Introduction to Model-Based Systems Engineering (MBSE) - SEI Blog, accessed July 12, 2025, https://insights.sei.cmu.edu/blog/introduction-model-based-systems-engineering-mbse/
78. Software-defined Defence: Algorithms at War - The International Institute for Strategic Studies, accessed July 12, 2025, https://www.iiss.org/globalassets/media-library---content--migration/files/research-papers/iiss_software-defined-defence_17022023.pdf
79. What F-35 Can Teach us About Global Software Delivery for Impact - HAW Hamburg, accessed July 12, 2025, https://www.fzt.haw-hamburg.de/pers/Scholz/dglr/hh/text_2024_05_16_F35_Software.pdf
80. Model Based Systems Engineering - BAE Systems, accessed July 12, 2025, https://www.baesystems.com/en-us/who-we-are/intelligence-and-security/mbse
81. Air Force Digital Transformation: Strategy & Impact - WalkMe, accessed July 12, 2025, https://www.walkme.com/blog/air-force-digital-transformation/
82. Diehl Aviation Powers Next-Generation Avionics for Europe's FCAS ..., accessed July 12, 2025, https://www.diehl.com/aviation/en/press-and-media/press/diehl-aviation-powers-next-generation-avionics-for-europes-fcas-fighter-program/
83. CJADC2 interoperability: AI-/ML-based sensor fusion at the edge ..., accessed July 12, 2025, https://militaryembedded.com/ai/machine-learning/cjadc2-interoperability-ai-ml-based-sensor-fusion-at-the-edge
84. Autonomous aircraft - Wikipedia, accessed July 12, 2025, https://en.wikipedia.org/wiki/Autonomous_aircraft
85. Explainable AI in Autonomous Vehicles: Building Transparency and Trust on the Road, accessed July 12, 2025, https://smythos.com/managers/ops/explainable-ai-in-autonomous-vehicles/
86. A Guide to Explainable AI (XAI) - Unaligned Newsletter, accessed July 12, 2025, https://www.unaligned.io/p/guide-explainable-ai-xai
87. The Need for Explainability in Autonomous Systems: Enhancing Transparency and Trust with Explainable AI (XAI) | by Siddhartha Pramanik | Medium, accessed July 12, 2025, https://medium.com/@siddharthapramanik771/the-need-for-explainability-in-autonomous-systems-enhancing-transparency-and-trust-with-83336b6640bd
88. Navigating ethical challenges of explainable ai in autonomous systems - International Journal of Science and Research Archive, accessed July 12, 2025, https://ijsra.net/sites/default/files/IJSRA-2024-1872.pdf
89. Formal Verification of Deep Neural Networks for Object Detection - arXiv, accessed July 12, 2025, https://arxiv.org/html/2407.01295
90. Formal Verification of a Neural Network Based Prognostics System for Aircraft Equipment, accessed July 12, 2025, https://www.researchgate.net/publication/376515383_Formal_Verification_of_a_Neural_Network_Based_Prognostics_System_for_Aircraft_Equipment
91. Formal Analysis of Neural Network-based Systems in the Aircraft Domain? - Department of Computing, accessed July 12, 2025, https://www.doc.ic.ac.uk/~alessio/papers/21/fm21-KKLLMOZ.pdf
92. What Is FACE™? | Wind River, accessed July 12, 2025, https://www.windriver.com/solutions/learning/face
93. What is FACE? - everything RF, accessed July 12, 2025, https://www.everythingrf.com/community/what-is-face
94. The Future Airborne Capability Environment (FACE™) - LDRA, accessed July 12, 2025, https://ldra.com/face/
95. An app store for military avionics: FACE expo to showcase promising new standard for unifying DOD aviation systems | NAVAIR, accessed July 12, 2025, https://www.navair.navy.mil/node/17441
96. Future Airborne Capability Environment - Wikipedia, accessed July 12, 2025, https://en.wikipedia.org/wiki/Future_Airborne_Capability_Environment
97. FACE Software Architecture Will Play An Important Role In The Future of Process Automation - ARC Advisory Group, accessed July 12, 2025, https://www.arcweb.com/industry-best-practices/face-software-architecture-will-play-important-role-future-process
98. The FACE Approach | www.opengroup.org, accessed July 12, 2025, https://www.opengroup.org/face/approach
99. How To Start an Avionics Systems Manufacturer Business | ClickUp™, accessed July 12, 2025, https://clickup.com/p/small-business/how-to-start-avionics-systems-manufacturer-business
100. Contracts for July 9, 2025 - Department of Defense, accessed July 12, 2025, https://www.defense.gov/News/Contracts/Contract/Article/4238652/
101. Integrating Software Data Loaders into ATE Systems - Avionics Interface Technologies - A Teradyne Company, accessed July 12, 2025, https://aviftech.com/integrating-software-data-loaders-into-ate-systems/


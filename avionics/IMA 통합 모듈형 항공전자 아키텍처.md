# IMA 통합 모듈형 항공전자 아키텍처

## 1.  통합 모듈형 항공전자의 기초

항공전자 시스템의 설계 패러다임은 지난 수십 년간 근본적인 변화를 겪었습니다. 개별 기능이 독립된 하드웨어에 의존하던 전통적인 연합형(Federated) 아키텍처에서 벗어나, 고도로 통합되고 소프트웨어 중심적인 플랫폼으로 진화했습니다. 이 혁신의 중심에는 통합 모듈형 항공전자(Integrated Modular Avionics, IMA) 아키텍처가 있습니다. 본 파트에서는 IMA의 등장을 촉발한 기술적, 역사적 배경을 추적하고, 그 핵심 원리를 분석하며, 항공전자 공학의 지형을 어떻게 재정의했는지 탐구합니다.

### 1.1  연합형에서 통합형 아키텍처로의 진화

IMA의 혁신성을 이해하기 위해서는 그 이전의 지배적인 패러다임이었던 연합형 아키텍처의 원리와 한계를 먼저 이해해야 합니다. 연합형 시스템의 본질적인 제약은 IMA라는 새로운 아키텍처의 등장을 필연적으로 만들었습니다.

#### 1.1.1  연합형 패러다임: 원리와 한계

연합형 아키텍처는 "하나의 기능, 하나의 컴퓨터(one function — one computer)"라는 단순하고 직관적인 원칙에 기반합니다.1 이 모델에서 항공기의 각 기능, 예를 들어 항법, 통신, 비행 제어 등은 각각의 독립적이고 자족적인 하드웨어 장치에 탑재됩니다. 이 장치들은 라인 교체 단위(Line Replaceable Unit, LRU) 또는 라인 교체 모듈(Line Replaceable Module, LRM)로 알려져 있으며, ARINC 429와 같은 표준화된 데이터 버스를 통해 상호 연결됩니다.1 1980년대 후반부터 널리 사용된 이 아키텍처는 F-16 MLU와 같은 군용기나 초기 에어버스 및 보잉 기종에서 찾아볼 수 있습니다.1

초기에는 기능의 물리적 분리가 오류 격리(fault containment)에 명백한 이점을 제공했지만, 항공전자 기능의 수와 복잡성이 기하급수적으로 증가함에 따라 연합형 아키텍처는 심각한 한계에 직면했습니다.

- **SWaP-C (크기, 중량, 전력 및 비용) 부담 증가:** 새로운 기능이 추가될 때마다 새로운 LRU가 필요했고, 이는 항공기 내부에 더 많은 공간, 배선, 전력 공급 장치를 요구했습니다. 이러한 하드웨어의 증가는 항공기의 총 중량을 늘려 연료 효율성을 저하시키고, 복잡한 배선으로 인해 유지보수 비용을 상승시키는 직접적인 원인이 되었습니다.2
- **자원 비효율성:** 연합형 시스템의 각 LRU에 탑재된 프로세서는 최대 부하(peak load) 상황을 기준으로 사양이 결정되었습니다. 하지만 실제 비행 임무의 대부분 시간 동안 이 프로세서들은 자신의 최대 처리 능력의 일부만을 사용하게 되어, 시스템 전체적으로 컴퓨팅 자원이 심각하게 낭비되는 결과를 초래했습니다.7
- **유연성 부족과 노후화:** 기존 시스템을 업그레이드하거나 새로운 기능을 추가하는 작업은 매우 어렵고 비용이 많이 들었습니다. 이는 단순히 새로운 하드웨어를 추가하는 것을 넘어, 물리적 통합, 배선 변경, 그리고 때로는 항공기 구조 변경까지 요구했기 때문입니다. 이러한 경직성은 기술 발전을 항공기 시스템에 신속하게 적용하는 것을 방해하는 "기술적 천장"을 만들었습니다.2

이러한 한계들은 단순히 크기와 무게를 줄이는 최적화의 문제를 넘어섰습니다. 연합형 모델은 소프트웨어 복잡성이 폭발적으로 증가하는 시대에 더 이상 지속 가능하지 않은 패러다임이었습니다. 항공전자 시스템의 발전은 더 많은 기능을 요구했고, 각 기능은 더 정교한 소프트웨어를 필요로 했습니다. 연합형 구조 하에서 이는 곧 하드웨어의 무한한 증식을 의미했으며, 이는 물리적으로나 경제적으로 불가능한 길이었습니다. 따라서 IMA로의 전환은 단순한 개선이 아니라, 항공전자 시스템이 다음 단계로 나아가기 위한 필수적인 아키텍처 혁명이었습니다. 설계의 핵심 과제가 물리적 통합(상자와 전선을 위한 공간 찾기)에서 논리적 통합(공유 자원을 안전하게 관리하기)으로 이동했으며, 이는 완전히 새로운 표준과 인증 접근법을 요구하게 되었습니다.

#### 1.1.2  IMA의 기원: 군용기에서의 시작과 상용기에서의 채택

IMA 개념은 1990년대 초 4세대 전투기 설계를 통해 처음 구체화된 것으로 알려져 있습니다.8 이 혁신적인 접근법의 대표적인 사례는 F-22 랩터(Raptor)에 탑재된 통합 항공전자 시스템(Integrated Avionics System, IAS)과 그 핵심인 공통 통합 프로세서(Common Integrated Processor, CIP)입니다. 이 시스템은 전통적인 단일 기능 "블랙박스"들을 공통의 모듈형 처리 아키텍처로 대체함으로써 항공전자 시스템의 통합 수준을 한 차원 높였습니다.9 F-35와 프랑스의 다쏘 라팔(Dassault Rafale) 전투기 역시 초창기부터 이 개념을 도입한 선구자들입니다.8

군용 분야에서 그 가능성을 입증한 IMA 개념은 1990년대 후반 상용 항공기 시장으로 빠르게 확산되었습니다. 보잉 777에 탑재된 하니웰(Honeywell)의 항공기 정보 관리 시스템(Airplane Information Management System, AIMS)은 상용기에 IMA 원리가 부분적으로 혹은 독점적인 형태로 구현된 최초의 사례로 평가받습니다.1

진정한 의미의 "개방형 IMA(Open IMA)" 시대를 연 것은 에어버스 A380입니다. 탈레스(Thales), 디일(Diehl) 등과 협력하여 개발된 A380의 항공전자 시스템은 조종석뿐만 아니라 항공기 전체 시스템 영역에 걸쳐 ARINC 653, ARINC 664(AFDX)와 같은 공개 표준을 전면적으로 채택한 최초의 사례였습니다.1 이는 특정 공급업체에 종속되지 않는 유연하고 확장 가능한 시스템 구축의 가능성을 열었으며, 이후 보잉 787의 공통 핵심 시스템(Common Core System, CCS)과 같은 후속 기종들에게 표준 모델을 제시했습니다.4

#### 1.1.3  IMA의 핵심 원리: 모듈성, 확장성, 이식성, 재구성

IMA 아키텍처는 연합형 시스템의 한계를 극복하기 위해 몇 가지 핵심적인 원리를 기반으로 설계되었습니다.

- **모듈성 및 통합:** IMA는 복잡한 시스템을 더 작고 독립적인 하드웨어 및 소프트웨어 모듈로 분해합니다.13 그리고 여러 항공전자 기능들을 공유된 하드웨어 및 소프트웨어 플랫폼 위에 통합합니다.14
- **자원 공유:** 전용 자원을 할당하는 대신, IMA는 프로세서, 메모리, 네트워크 대역폭과 같은 컴퓨팅 자원을 공유 풀(pool) 형태로 관리하고, 각 애플리케이션의 요구에 따라 동적으로 할당합니다.2
- **확장성 및 유연성:** 이 아키텍처는 높은 확장성을 가지도록 설계되어, 다양한 임무 요구사항이나 미래의 기술 발전에 대응하기 위해 모듈과 기능을 쉽게 추가하거나 제거할 수 있습니다.13
- **소프트웨어 이식성:** 핵심 목표 중 하나는 애플리케이션 소프트웨어가 다양한 공통 하드웨어 모듈에서 이식 가능하도록 만드는 것입니다. 이는 하드웨어의 세부 사항을 애플리케이션으로부터 숨기는 표준화된 API(Application Programming Interface)와 추상화 계층을 통해 달성됩니다.8 이를 통해 개발자들은 하위 시스템의 복잡성에서 벗어나 애플리케이션 계층에 집중할 수 있습니다.8
- **재구성:** IMA는 정적(지상에서) 또는 동적(비행 중) 재구성을 지원합니다. 주 모듈에 장애가 발생할 경우, 해당 모듈에서 실행되던 애플리케이션을 예비 모듈로 이전하여 실행함으로써 시스템 전체의 가용성과 내고장성을 크게 향상시킬 수 있습니다.16

#### 1.1.4  아키텍처 비교 분석: 연합형 대 IMA

연합형 아키텍처와 IMA 아키텍처의 차이점은 항공전자 시스템의 설계, 개발, 유지보수 방식에 근본적인 변화를 가져왔습니다. 아래 표는 두 패러다임의 핵심적인 특징을 비교하여 그 차이를 명확히 보여줍니다.

| 특징                  | 연합형 아키텍처                                     | 통합 모듈형 항공전자(IMA) 아키텍처                      |
| --------------------- | --------------------------------------------------- | ------------------------------------------------------- |
| **핵심 철학**         | "하나의 기능, 하나의 상자(One function, one box)" 1 | 공유 자원 기반의 통합 플랫폼 2                          |
| **하드웨어**          | 전용, 특수화된 LRU                                  | 공통, 표준화된 모듈 (CPM, IOM 등) 14                    |
| **소프트웨어**        | 하드웨어와 긴밀하게 결합                            | 이식 가능하고 하드웨어에 독립적인 애플리케이션 8        |
| **자원 활용도**       | 낮음 (대부분의 시간 동안 미활용) 7                  | 높음 (효율적인 공유)                                    |
| **오류 격리**         | 내재적인 물리적 분리                                | 논리적 파티셔닝 (예: ARINC 653) 18                      |
| **SWaP-C**            | 높음                                                | 대폭 감소 6                                             |
| **업그레이드/확장성** | 어렵고 비용이 많이 듦                               | 높은 유연성과 확장성 13                                 |
| **통합 복잡성**       | 주로 물리적 (배선)                                  | 주로 논리적/소프트웨어적 (자원 관리, 스케줄링, 검증) 19 |
| **핵심 과제**         | 물리적 제약 및 노후화                               | 시스템 복잡성 및 인증 20                                |

### 1.2  IMA 시스템의 해부학

IMA 시스템은 물리적 하드웨어와 논리적 소프트웨어 계층이 정교하게 결합된 구조를 가집니다. 이 섹션에서는 IMA 아키텍처를 구성하는 핵심적인 물리적, 논리적 빌딩 블록들을 상세히 분석합니다.

#### 1.2.1  하드웨어 플랫폼 아키텍처: 캐비닛, 모듈, 백플레인

IMA 시스템의 물리적 기반은 종종 핵심 부품들을 수용하는 모듈형 캐비닛을 중심으로 구축됩니다. 다쏘(Dassault) 제트기의 하니웰 모듈형 항공전자 장치(Modular Avionics Units, MAU)나 보잉 787의 공통 컴퓨팅 자원(Common Computing Resource, CCR) 캐비닛이 대표적인 예입니다.8

이러한 캐비닛은 다양한 라인 교체 모듈(LRM)을 위한 슬롯을 포함하고 있습니다. 여기에는 처리 모듈, 입출력(I/O) 모듈, 전원 공급 모듈 등이 포함됩니다.14 예를 들어, F-22의 CIP는 표준 전자 모듈 크기 E(Standard Electronics Module Size E, SEM-E) 규격의 모듈을 사용합니다.9 이러한 모듈성은 전체 LRU를 교체하는 대신 개별 카드만 교체할 수 있게 하여 유지보수와 업그레이드를 간소화합니다.6

모듈들은 캐비닛 내의 고속 백플레인에 연결되어 모듈 간 통신과 전력 분배를 담당합니다. 초기 IMA 시스템은 SAFEbus와 같은 독점적인 백플레인을 사용했지만, 현대 시스템은 종종 주 항공기 네트워크(AFDX)를 모듈 간 통신에도 활용합니다.12

#### 1.2.2  핵심 처리 모듈(CPM)과 입출력 모듈(IOM)

- **핵심 처리 모듈(Core Processing Modules, CPMs):** CPM은 IMA 시스템의 '두뇌' 역할을 하며, 여러 항공전자 애플리케이션을 호스팅하고 실행하는 데 필요한 컴퓨팅 성능을 제공합니다.13 B787 CCS에서는 이를 일반 프로세서 모듈(General Processor Modules, GPMs)이라고도 부릅니다.21 핵심 특징은 공통 하드웨어를 사용하여 서로 다른 애플리케이션을 동일한 프로세서 카드에서 실행할 수 있다는 점입니다.8
- **입출력 모듈(Input/Output Modules, IOMs):** IOM은 IMA 처리 플랫폼과 항공기의 외부 장치(센서, 액추에이터, 디스플레이 등) 사이의 인터페이스 역할을 합니다.14 이들은 데이터 변환, 오류 감지, 주변 장치와의 통신을 처리하며, 고속 내부 네트워크와 다양한 레거시 또는 특수 데이터 형식 간의 격차를 해소합니다.23
- **통합 모듈(Core Processor Input/Output Modules, CPIOMs):** 탈레스의 아키텍처와 같은 일부 시스템에서는 처리 기능과 I/O 기능을 단일 모듈에 통합한 CPIOM을 사용합니다. 이는 긴밀하게 결합된 제어 루프를 구현하거나 별도의 I/O 장치의 필요성을 줄이는 데 사용될 수 있습니다.25

#### 1.2.3  네트워크 백본: ARINC 664 (AFDX) 심층 분석

ARINC 664 파트 7로 표준화된 항공전자 전이중 교환 이더넷(Avionics Full-Duplex Switched Ethernet, AFDX)은 현대 IMA 시스템의 지배적인 고속, 결정론적 네트워크 백본입니다.27 에어버스가 A380을 위해 개척한 이 기술은 현재 B787 및 다른 여러 플랫폼에서 널리 사용되고 있습니다.27 AFDX는 상용 기성품(COTS)인 IEEE 802.3 이더넷 기술을 활용하지만, 안전 필수 애플리케이션에 요구되는 결정성과 신뢰성을 보장하기 위해 특정 프로파일과 프로토콜을 추가합니다.27

##### 1.2.3.1  결정성, 이중화, 그리고 서비스 품질(QoS)

- **전이중 교환 네트워크:** AFDX는 교환형(switched), 전이중(full-duplex) 스타 토폴로지를 사용하여 구형 버스 시스템에서 발생하던 충돌을 제거하고 전용 대역폭을 제공합니다.27
- **이중화:** 네트워크는 물리적으로 이중화됩니다. 엔드 시스템은 동일한 프레임을 두 개의 중복된 네트워크(네트워크 A와 네트워크 B)를 통해 동시에 전송합니다. 수신 측 엔드 시스템은 먼저 도착한 유효한 프레임을 수락하고 두 번째 프레임은 폐기하여, 하나의 네트워크 경로에 장애가 발생하더라도 지속적인 작동을 보장합니다.28
- **결정성:** AFDX는 제한된 지연 시간(latency)과 지터(jitter) 내에서 메시지 전달을 보장합니다. 이는 실시간 비행 제어 및 기타 안전 필수 기능에 매우 중요합니다.27

##### 1.2.3.2  가상 링크(VL)와 대역폭 할당 갭(BAG)

- **가상 링크(Virtual Links, VLs):** AFDX는 물리적 MAC 주소를 기반으로 패킷을 라우팅하지 않습니다. 대신 16비트의 가상 링크 ID를 사용합니다. VL은 하나의 소스 엔드 시스템에서 하나 이상의 목적지 엔드 시스템으로 가는 단방향 논리적 경로를 정의합니다.27 스위치는 VL ID를 기반으로 프레임을 라우팅하도록 미리 구성되어, 고정되고 예측 가능한 데이터 경로를 생성합니다.
- **대역폭 할당 갭(Bandwidth Allocation Gap, BAG):** 이는 트래픽 제어(traffic policing)와 결정성 보장의 핵심 메커니즘입니다. BAG는 특정 VL에 대해 두 개의 연속된 프레임 사이의 최소 시간 간격(예: 1ms, 2ms, 4ms...)을 정의합니다.27 소스 엔드 시스템은 이 간격을 강제하여 단일 VL이 네트워크를 독점하는 것을 방지하고, 다른 모든 중요한 VL을 위한 대역폭이 항상 보장되도록 합니다.

#### 1.2.4  원격 데이터 집중기(RDC)의 역할

원격 데이터 집중기(Remote Data Concentrators, RDC)는 IMA 아키텍처, 특히 분산형 IMA(DIMA)에서 중요한 역할을 합니다. RDC는 항공기 곳곳의 센서와 액추에이터에 물리적으로 가깝게 배치되는 소형 장치입니다. 여러 로컬 소스로부터 데이터를 수집하고, 이를 디지털화하여 패키징한 후, 고속 AFDX 네트워크를 통해 중앙 IMA 캐비닛으로 전송합니다.32

RDC의 가장 큰 이점은 데이터 집중을 통해 점대점(point-to-point) 배선의 양과 길이를 극적으로 줄일 수 있다는 것입니다. 이는 상당한 중량 감소와 설치 간소화로 이어집니다. 예를 들어, 보잉 787의 CCS는 21개의 RDC를 활용하여 이러한 이점을 극대화합니다.32

IMA 하드웨어 아키텍처는 "관심사 분리(separation of concerns)" 원칙의 물리적 구현이라고 할 수 있습니다. CPM은 일반적인 계산을 처리하고, IOM/RDC는 특정 물리적 인터페이스를 담당하며, AFDX 네트워크는 통신을 관리합니다. 이러한 분리는 모듈성과 확장성을 가능하게 하는 핵심 요소입니다. CPM은 센서의 세부 사항을 알 필요 없이 IOM/RDC로부터 표준화된 데이터 패킷만 받으면 되고, IOM은 데이터가 어떻게 처리되는지 알 필요 없이 센서 신호를 표준 AFDX 프레임으로 변환하기만 하면 됩니다. 이러한 아키텍처적 분리는 독립적인 개발과 진화를 가능하게 합니다. 새로운 센서는 해당 로컬 RDC만 업데이트하거나 교체하면 추가할 수 있으며, 중앙 CPM이나 그 위에서 실행되는 애플리케이션에는 영향을 미치지 않습니다. 마찬가지로, 수백 개의 센서나 액추에이터를 변경하지 않고도 CPM을 더 강력한 프로세서로 업그레이드할 수 있습니다. 이 물리적, 논리적 분리는 연합형 아키텍처의 경직성과 노후화 문제를 직접적으로 해결하며, IMA가 약속하는 장기적인 유지보수성과 업그레이드 가능성의 핵심 동력입니다.

## 2.  소프트웨어, 표준, 그리고 인증

IMA의 진정한 힘은 하드웨어 플랫폼뿐만 아니라, 그 위에서 실행되는 소프트웨어 아키텍처와 이를 규제하는 엄격한 표준 및 인증 프레임워크에서 나옵니다. 이 파트에서는 IMA를 가능하게 하는 비물리적 요소들, 즉 소프트웨어의 핵심 원리와 안전성 및 상호운용성을 보장하는 규제 환경을 심층적으로 탐구합니다.

### 2.1  소프트웨어의 초석: ARINC 653과 강력한 파티셔닝

ARINC 653 표준은 IMA의 소프트웨어적 심장부입니다. 이 표준이 없었다면, 하나의 프로세서에서 서로 다른 중요도를 가진 여러 애플리케이션을 안전하게 공유하는 것은 불가능했을 것입니다. 이 섹션에서는 ARINC 653이 어떻게 이러한 혁신을 가능하게 하는지 그 핵심 메커니즘을 분석합니다.

#### 2.1.1  APEX (APplication/EXecutive) 인터페이스

ARINC 653 사양은 운영체제("Executive")와 애플리케이션 소프트웨어("Application") 사이의 표준 소프트웨어 인터페이스를 정의합니다.34 APEX(APplication/EXecutive)로 알려진 이 API는 파티션 관리, 프로세스 관리, 시간 관리, 메모리 관리, 그리고 파티션 간 통신을 위한 표준화된 서비스 세트를 제공합니다.34

APEX의 주된 목표는 강력한 시간 및 공간 파티셔닝을 위한 프레임워크를 제공하는 것입니다. 이를 통해 서로 다른 공급업체가 개발하고, 서로 다른 안전성 등급(예: DAL A부터 DAL D까지)을 가진 소프트웨어들이 단일 프로세서 상에서 서로 간섭 없이 공존할 수 있게 됩니다.8 이는 각 소프트웨어 구성 요소(파티션)를 독립적으로 테스트, 검증 및 인증할 수 있는 기반을 마련합니다.8

#### 2.1.2  공간 파티셔닝: 메모리 보호와 격리

공간 파티셔닝은 각 애플리케이션(파티션)이 자신만의 독립적이고 보호된 메모리 공간을 갖도록 보장하는 핵심 개념입니다.34 이 메커니즘을 통해 하나의 파티션이 다른 파티션의 메모리에 접근하거나, 데이터를 손상시키거나, 어떤 형태로든 간섭하는 것을 원천적으로 차단합니다.15

이러한 격리는 일반적으로 프로세서의 메모리 관리 장치(Memory Management Unit, MMU)를 통해 구현됩니다. ARINC 653 호환 실시간 운영체제(RTOS)는 MMU를 설정하여 각 파티션에 대해 서로 다른 가상 메모리 컨텍스트를 생성합니다.15 각 파티션은 자신만의 스택과 힙을 가지며, 이 메모리 영역은 다른 파티션으로부터 완벽하게 보호됩니다.15 강력한 파티셔닝의 궁극적인 목표는 물리적으로 분리된 연합형 시스템과 동등한 수준의 기능적 격리를 논리적으로 구현하는 것입니다.18

#### 2.1.3  시간 파티셔닝: 계층적 스케줄링과 결정성

시간 파티셔닝은 단일 애플리케이션이 CPU를 독점하는 것을 방지하여, 모든 애플리케이션이 할당된 처리 시간을 보장받도록 하는 메커니즘입니다.15 이는 시스템의 실시간 성능과 예측 가능성을 보장하는 데 필수적입니다.

ARINC 653은 이를 위해 두 단계의 계층적 스케줄링 모델을 정의합니다 34:

- **1단계 (파티션 스케줄링):** RTOS는 고정된 순환 스케줄을 사용하여 파티션들에게 CPU 시간을 할당합니다. 이 스케줄은 주 시간 프레임(Major Time Frame, MaF)이라는 큰 주기를 기준으로 반복됩니다. MaF 내에서 각 파티션은 고정된 길이의 시간 슬롯, 즉 파티션 시간 창(Partition Time Window)을 할당받습니다. 이는 비선점형(non-preemptive) 라운드 로빈 방식과 유사하게 동작합니다.
- **2단계 (프로세스 스케줄링):** 각 파티션은 자신에게 할당된 시간 창 내에서 하나 이상의 프로세스(또는 스레드)를 실행할 수 있습니다. 이 프로세스들의 스케줄링은 일반적으로 선점형(preemptive) 우선순위 기반으로 이루어지며, 해당 파티션의 내부 로직에 의해 관리됩니다.

이러한 계층적 모델은 한 파티션 내의 높은 우선순위 프로세스가 다른 파티션의 실행을 지연시킬 수 없도록 보장합니다. 결과적으로 시스템 수준에서 예측 가능하고 결정론적인 타이밍 동작을 제공하게 됩니다.37

#### 2.1.4  파티션 간 통신: 샘플링 및 큐잉 포트

파티션들은 격리되어 있지만, 기능 수행을 위해 서로 통신해야 합니다. ARINC 653은 통제되지 않는 데이터 흐름을 방지하기 위해 엄격하게 제어되는 통신 메커니즘을 정의합니다.37 파티션 간의 직접적인 메모리 공유는 금지됩니다.

통신은 논리적이고 단방향인 포트(port)를 통해 이루어집니다. 주요 포트 유형은 다음과 같습니다:

- **샘플링 포트(Sampling Ports):** 상태 기반 데이터(state-based data)에 사용됩니다. 새로운 메시지는 이전 메시지를 덮어씁니다. 수신자는 항상 가장 최신 메시지를 받지만, 송신자가 수신자보다 빠를 경우 중간 메시지가 손실될 수 있습니다.
- **큐잉 포트(Queuing Ports):** 이벤트 기반 데이터(event-based data)에 사용됩니다. 메시지는 FIFO(First-In, First-Out) 큐에 버퍼링됩니다. 버퍼가 오버플로우되지 않는 한 메시지 손실이 없습니다.

이러한 포트와 그 연결은 시스템의 구성 파일에 정적으로 정의되어, 통신 경로가 사전에 알려지고 검증 가능하도록 보장합니다.38

#### 2.1.5  상태 감시 및 오류 관리

ARINC 653 표준은 각 파티션의 상태를 감시하고 오류를 통제된 방식으로 처리하기 위한 메커니즘을 포함합니다.36 각 파티션은 전용의 높은 우선순위를 가진 오류 처리기(error handler) 프로세스를 가질 수 있습니다. 만약 파티션 내의 프로세스에서 오류(예: 불법적인 메모리 접근)가 발생하면, 이 오류 처리기가 활성화됩니다. 오류 처리기는 오류에 대한 정보를 얻고, 오류가 발생한 프로세스를 중지시키는 등의 조치를 취할 수 있습니다.34 여기서 핵심은 한 파티션 내에서 발생한 오류가 해당 파티션 내에 격리되어 다른 파티션으로 전파되지 않는다는 점입니다.37

#### 2.1.6  상용 구현 사례: 윈드리버 VxWorks 653

윈드리버(Wind River)의 VxWorks 653은 ARINC 653 표준을 구현한 대표적인 상용 기성품(COTS) RTOS입니다.15 이 RTOS는 보잉 787, 에어버스 A400M 등 수많은 주요 항공우주 프로그램에 채택되어 그 신뢰성과 성능을 입증했습니다.40

VxWorks 653은 강력한 시간 및 공간 파티셔닝, 멀티코어 프로세서 지원, 그리고 윈드리버 워크벤치(Wind River Workbench)라는 통합 개발 환경을 제공합니다. 이 개발 환경에는 구성, 디버깅, 그리고 DO-178C와 같은 표준에 대한 인증 증거 생성을 지원하는 도구들이 포함되어 있습니다.39 시스템 및 파티션 구성은 XML 파일을 통해 정의되며, 이 모델 기반 접근 방식은 복잡한 구성 과정을 자동화하고 검증하는 데 도움을 줍니다.42

ARINC 653은 단순한 기술 사양을 넘어, IMA 비즈니스 모델의 근본적인 동력입니다. 이 표준은 항공기 제작사(예: 보잉, 에어버스)가 시스템 통합자 역할을 수행하며, 수많은 경쟁 관계에 있는 공급업체들의 소프트웨어를 단일 하드웨어 플랫폼 위에 안전하게 통합할 수 있는 '가상 백플레인'을 제공합니다. 여러 회사가 개발한 구성 요소를 통합하는 것은 IMA의 핵심 과제이며 8, 다양한 공급업체의 소프트웨어를 지원하는 것은 IMA의 주요 장점입니다.26 만약 파티셔닝 동작과 포트 기반 통신에 대한 엄격하고 강제적인 표준이 없다면, A 공급업체의 항법 애플리케이션이 B 공급업체의 비행 제어 애플리케이션과 동일한 프로세서에서 실행될 때 서로 간섭하지 않을 것이라고 보장할 수 없을 것입니다. 이 경우 통합 및 검증 노력은 천문학적으로 증가하고 각 항공기마다 고유한 작업이 될 것입니다. ARINC 653은 모든 소프트웨어 공급업체를 위한 '교통 규칙'을 제공합니다. 애플리케이션을 이러한 파티션 컨테이너와 제어된 인터페이스 안에 강제로 위치시킴으로써, 개별 애플리케이션의 개발 및 인증 과정을 서로 분리시킵니다. 공급업체는 자신의 ARINC 653 파티션을 독립적으로 개발하고 사전 인증할 수 있습니다.8 이는 항공전자 기능에 대한 경쟁 시장을 창출하고, 항공기 제작사의 특정 공급업체에 대한 종속성을 줄이며, 변경에 따르는 비용과 위험을 감소시킵니다. 항공기 제작사는 두 공급업체가 모두 ARINC 653 '계약'을 준수하는 한, 전체 시스템을 재인증할 필요 없이 한 공급업체의 기능을 다른 공급업체의 기능으로 교체할 수 있습니다.15 이러한 모듈식 인증이야말로 IMA를 이끄는 주요 경제적 원동력입니다.

### 2.2  IMA 생태계: 상호운용성과 디스플레이 표준

IMA 시스템은 ARINC 653과 664만으로 완성되지 않습니다. 완벽하고 상호운용 가능한 생태계를 구축하기 위해서는 조종석 디스플레이와 인증 절차를 포함한 다른 핵심 표준들이 조화롭게 작동해야 합니다.

#### 2.2.1  ARINC 661: 조종석 디스플레이 시스템(CDS) 표준화

ARINC 661은 IMA 철학을 조종석 인터페이스까지 확장한 중요한 표준입니다. 이는 항공전자 시스템의 논리와 그래픽 표현을 분리하여 모듈성과 유연성을 극대화합니다.

##### 2.2.1.1  항공전자 로직(사용자 애플리케이션)과 GUI(CDS)의 분리

ARINC 661 이전에는 조종석 디스플레이가 각 공급업체의 독점 기술이었습니다. 비행 관리 시스템(FMS)과 같은 항공전자 시스템의 로직은 화면에 그래픽을 그리는 코드와 긴밀하게 결합되어 있었습니다.44 이는 디스플레이 레이아웃을 조금만 변경해도 전체 항공전자 애플리케이션을 재인증해야 하는, 즉 비용과 시간이 많이 소요되는 문제를 야기했습니다.

ARINC 661은 이러한 문제를 해결하기 위해 항공전자 기능(사용자 애플리케이션, User Applications, UA)과 조종석 디스플레이 시스템(Cockpit Display System, CDS) 간의 인터페이스를 표준화합니다.45 이 모델에서 CDS는 그래픽 서버 역할을 하고, UA는 클라이언트 역할을 합니다. UA는 시스템 로직을 포함하며 CDS에게  *무엇을* 표시할지 지시하지만, *어떻게* 표시할지는 CDS가 결정합니다.

##### 2.2.1.2  정의 파일(DF), 위젯, 그리고 런타임 프로토콜

- **정의 파일(Definition Files, DF):** 디스플레이의 그래픽 레이아웃은 이진(binary) 형식의 정의 파일에 명시됩니다. 이 파일은 화면의 그래픽 객체(위젯) 계층 구조, 속성, 레이아웃 등을 정의하며, CDS가 초기화될 때 로드됩니다.44 도구 기반의 조작을 용이하게 하기 위해 XML 형식의 정의도 표준에 포함되어 있습니다.45
- **위젯(Widgets):** 표준은 버튼, 레이블, 목록, 메뉴 및 복잡한 지도 요소와 같은 풍부한 공통 그래픽 위젯 라이브러리를 정의합니다.44 이는 여러 UA에 걸쳐 일관된 모양과 느낌(look and feel)을 보장합니다.
- **런타임 프로토콜(Runtime Protocol):** 런타임 시, UA는 CDS에 메시지를 보내 위젯 속성을 수정합니다(예: "이 레이블을 새로운 속도 값으로 업데이트하라"). 반대로 CDS는 UA에 사용자 상호작용을 보고하는 메시지를 보냅니다(예: "조종사가 이 버튼을 클릭했다").44

ARINC 661은 IMA 철학을 완벽하게 보완합니다. IMA 시스템에서는 강력한 단일 그래픽 생성기 모듈이 CDS 역할을 수행하며, 네트워크 전반의 다양한 CPM에서 파티션된 애플리케이션으로 실행되는 여러 UA에 서비스를 제공할 수 있습니다. 이는 모듈성을 더욱 강화하고 변경 비용을 절감하는 효과를 가져옵니다.46

#### 2.2.2  인증 프레임워크: DO-178C, DO-297, 그리고 FAA/EASA의 역할

IMA 시스템의 안전성은 엄격한 인증 프레임워크를 통해 보장됩니다. 이는 소프트웨어 개발 프로세스부터 시스템 통합에 이르기까지 모든 단계를 포괄합니다.

- **DO-178C (소프트웨어 고려사항):** 이는 항공용 소프트웨어를 인증하기 위한 기본 표준입니다. IMA 애플리케이션은 기능의 중요도에 따라 결정되는 설계 보증 수준(Design Assurance Level, DAL A-E)에 맞춰 DO-178C를 준수하여 개발되어야 합니다.47 ARINC 653은 동일한 프로세서 상의 다른 파티션들이 서로 다른 DAL로 인증받는 것을 허용합니다.48
- **DO-297/ED-124 (IMA 개발 지침):** 이 문서는 IMA 시스템의 개발 및 인증에 대한 구체적인 지침을 제공합니다. 자원 관리, 여러 공급업체의 구성 요소 통합, 통합된 시스템의 안전성 보장과 같은 IMA 고유의 과제들을 다룹니다.38
- **규제 감독:** 미국 연방 항공청(FAA)과 유럽 항공 안전청(EASA)은 인증 과정을 감독하는 주요 규제 기관입니다. 이들은 IMA 아키텍처를 채택한 항공기에 대한 지침 자료를 발행하고 인증 계획을 승인합니다.18

아래 표는 IMA 생태계를 구성하는 주요 항공전자 표준들의 역할과 목적을 요약한 것입니다. 이 표준들은 각각 계산, 네트워킹, 디스플레이, 안전 프로세스라는 특정 영역을 담당하며, 상호 보완적으로 작동하여 IMA라는 복잡한 시스템을 실현합니다.

| 표준                              | 전체 명칭                                                    | IMA 컨텍스트에서의 목적                                      |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **ARINC 653**                     | Avionics Application Software Standard Interface             | 시간 및 공간 파티셔닝 규칙을 정의하여, 공유 하드웨어에서 혼합된 중요도의 애플리케이션들이 안전하게 공존할 수 있도록 함. "OS 계약". 34 |
| **ARINC 664 Part 7**              | Aircraft Data Network (AFDX)                                 | IMA 모듈과 엔드 시스템 간의 신뢰성 있는 통신을 위한 결정론적, 이중화, 고속 네트워크를 정의함. "네트워크 계약". 27 |
| **ARINC 661**                     | Cockpit Display System Interfaces to User Systems            | 항공전자 로직(UA)과 그래픽 디스플레이(CDS) 간의 표준 인터페이스를 정의하여 기능과 표현을 분리함. "HMI 계약". 44 |
| **RTCA DO-178C / EUROCAE ED-12C** | Software Considerations in Airborne Systems and Equipment Certification | 소프트웨어 안전성 보장을 위한 개발 프로세스와 목표를 규정하는 기본 표준. |
| **RTCA DO-297 / EUROCAE ED-124**  | Integrated Modular Avionics (IMA) Development Guidance and Certification Considerations | IMA 시스템의 고유한 과제에 안전 및 개발 프로세스를 적용하는 방법에 대한 구체적인 지침을 제공함. |

## 3.  구현, 과제, 그리고 미래 동향

이론적 기반을 넘어, 이 파트에서는 IMA가 실제 항공기에 어떻게 구현되고 있는지, 그리고 멀티코어 프로세서와 같은 현대 기술이 제기하는 첨단 과제들은 무엇인지 살펴봅니다. 또한, 분산형 IMA(DIMA), 확장된 모듈형 항공전자(EMA), 그리고 인공지능의 통합과 같은 미래 기술의 방향성을 조망합니다.

### 3.1  실제 적용된 IMA: 랜드마크 사례 연구

이 섹션에서는 IMA 원리와 표준이 세계에서 가장 진보된 항공기들에 어떻게 적용되었는지 분석합니다. 이를 통해 이론이 실제 엔지니어링 솔루션으로 변환되는 과정을 구체적으로 이해할 수 있습니다.

#### 3.1.1  군용기 선구자: F-22의 공통 통합 프로세서(CIP)와 F-35 아키텍처

- **F-22 공통 통합 프로세서(CIP):**
  - **아키텍처:** F-22의 통합 항공전자 시스템(IAS)은 레이시온(Raytheon)이 개발한 2개의 공통 통합 프로세서(CIP)를 중심으로 구축된 고도로 통합된 시스템입니다 (세 번째 CIP를 위한 공간 포함). 이는 이전 세대 전투기의 연합형 시스템에서 급진적으로 벗어난 것이었습니다.9
  - **처리 능력:** 각 CIP는 당시 기준으로 크레이(Cray) 슈퍼컴퓨터 2대에 해당하는 처리 능력을 갖춘 것으로 묘사될 만큼, 계산 능력에서 엄청난 도약을 이루었습니다.50
  - **구성 요소:** CIP는 범용 계산을 위한 인텔 i960 프로세서 기반의 이중 데이터 처리 요소(DDPE)와 센서 데이터 처리를 위한 디지털 신호 처리 요소(DSPE)를 포함한 공통의 범용 SEM-E 모듈을 사용합니다.9
  - **기능:** CIP는 레이더, 전자전(EW), 통신/항법/식별(CNI), 임무 소프트웨어 등 거의 모든 항공전자 기능의 데이터 및 신호 처리를 수행합니다. 원시 센서 데이터는 400 Mbps 광섬유 버스를 통해 CIP로 전송됩니다.9
- **F-35 라이트닝 II 아키텍처:**
  - **고도 통합:** F-35는 IMA 철학을 계승하여, 레거시 항공기에 비해 부품 수, 크기, 중량을 줄이기 위해 고도로 통합된 서브시스템 아키텍처를 특징으로 합니다.51
  - **중앙 집중식 처리:** F-22와 마찬가지로, 중앙 처리 플랫폼을 사용하여 여러 연합형 항공전자 애플리케이션을 호스팅함으로써 전체 항공기 시스템에 걸쳐 균일한 하드웨어 및 OS 환경을 조성합니다.11

#### 3.1.2  상용 항공의 게임 체인저

- **5.2.1. 보잉 787의 공통 핵심 시스템(CCS) by GE Aviation**
  - **아키텍처:** CCS는 GE Aviation이 개발한 787의 "중추 신경계"입니다. 이는 내고장성을 위해 물리적으로 분리된 두 개의 중복 공통 컴퓨팅 자원(CCR) 캐비닛으로 구성됩니다.32
  - **구성 요소:** 각 CCR 캐비닛은 일반 프로세서 모듈(GPM), ARINC 664 캐비닛 스위치(ACS), 광섬유 변환기(FOX)와 같은 모듈을 수용합니다. GPM은 윈드리버의 파티션된 OS(VxWorks 653)와 함께 프리스케일(Freescale) 프로세서에서 실행됩니다.21
  - **개방형 아키텍처:** CCS의 핵심 특징은 "개방형 아키텍처" 접근 방식으로, 이를 통해 여러 다른 공급업체의 소프트웨어를 공통 플랫폼에서 호스팅할 수 있습니다. 이는 향후 변경 및 업그레이드 비용을 절감하기 위해 설계되었습니다.32
  - **분산형 특성:** CCS는 분산형 IMA(DIMA) 시스템의 대표적인 예로, 21개의 원격 데이터 집중기(RDC)를 사용하여 항공기 전체의 센서와 시스템을 AFDX 네트워크를 통해 중앙 CCR에 연결합니다.32
- **5.2.2. 에어버스 A380/A350의 IMA 플랫폼 by Thales**
  - **"개방형 IMA" 개척:** A380은 모든 영역(조종석, 유틸리티 등)에 걸쳐 ARINC 표준을 기반으로 한 개방형 IMA 아키텍처를 완전히 구현한 최초의 항공기였습니다.1 탈레스가 주요 공급업체이자 통합자 역할을 수행했습니다.8
  - **구성 요소:** 이 아키텍처는 이중 중복 AFDX(ARINC 664) 네트워크로 연결된 공통 처리 및 I/O 모듈(CPIOM)을 중심으로 구축되었습니다.26
  - **공급업체 통합:** A380에서 입증된 핵심 장점 중 하나는 항공기 제작사(에어버스)가 다양한 공급업체의 소프트웨어를 탈레스가 제공한 플랫폼에 통합할 수 있는 능력입니다.26 이 모델은 A350과 A400M에서도 계속되었습니다.26

#### 3.1.3  비즈니스 및 지역 제트기: 하니웰 EASy와 록웰 콜린스 Pro Line Fusion

- **하니웰 EASy 플랫폼:** 다쏘 팔콘 제트기에 사용되는 이 플랫폼은 두 개의 이중 채널 모듈형 항공전자 장치(MAU)를 중심으로 구축되어 있으며, 여러 애플리케이션을 위한 기능 카드를 단일 모듈에 통합합니다.8
- **록웰 콜린스 Pro Line Fusion:** 이 IMA 시스템은 봄바르디어 글로벌 시리즈 및 걸프스트림 G280과 같은 다양한 비즈니스 제트기에 사용됩니다.8
- **ATR -600 시리즈:** 탈레스는 ATR 지역 터보프롭 항공기를 위한 IMA 스위트를 제공하며, A380과 같은 대형 여객기를 위해 개발된 AFDX 및 글래스 칵핏과 같은 기술을 지역 항공기 부문에 도입했습니다.56

이러한 사례 연구들은 IMA 철학의 명확한 진화 추세를 보여줍니다. 고성능이지만 독점적인 군용 시스템에서 시작하여, 경쟁력 있고 유연한 공급망을 육성하기 위해 설계된 개방형 표준 기반의 상용 플랫폼으로 발전했습니다. F-22의 CIP는 혁신적이고 고도로 통합된 시스템이었지만, 그 구성 요소와 아키텍처는 해당 프로그램에 특화된 것이었습니다.9 반면, 에어버스 A380은 독점 표준을 명시적으로 버리고 개방형 ARINC 표준(653, 664)을 채택했습니다.12 보잉 787의 CCS 역시 여러 공급업체의 소프트웨어를 호스팅하도록 설계된 "개방형 아키텍처"로 명시적으로 설명됩니다.32 이는 군의 주요 동인이 원초적인 성능과 기능 통합이었던 반면, 상용 항공 산업은 수명 주기 비용, 유지보수성, 공급업체 유연성에 훨씬 더 높은 가치를 두었음을 시사합니다. 개방형 표준 기반 접근 방식은 항공기 제작사가 단일 항공전자 공급업체에 종속되는 것을 피하고, 경쟁을 촉진하며, 다양한 공급망에서 기능을 통합하는 과정을 단순화합니다.8 결국, "IMA"는 단일 개념이 아니라, 프로그램의 비즈니스 및 전략적 목표에 따라 그 구현 방식이 크게 달라지는 유연한 패러다임임을 알 수 있습니다.

| 항공기 플랫폼        | IMA 시스템 명칭          | 주요 공급업체             | 핵심 아키텍처 특징                                           |
| -------------------- | ------------------------ | ------------------------- | ------------------------------------------------------------ |
| **F-22 랩터**        | 공통 통합 프로세서 (CIP) | 레이시온                  | 고도로 통합된 중앙 집중식 처리, 공통 SEM-E 모듈, 독점 고속 버스 사용 9 |
| **F-35 라이트닝 II** | 통합 코어 프로세서       | 록히드 마틴/노스롭 그루먼 | 고도로 통합, 다중 기능 호스팅, IMA 철학 계승 51              |
| **보잉 787**         | 공통 핵심 시스템 (CCS)   | GE Aviation               | 개방형 아키텍처, 2개의 중복 CCR 캐비닛, GPM, RDC의 광범위한 사용(DIMA), ARINC 653/664 21 |
| **에어버스 A380**    | IMA 플랫폼               | 탈레스                    | 최초의 완전한 "개방형 IMA" 구현, CPIOM, 모든 항공기 영역에 ARINC 653/664(AFDX) 적용 12 |
| **다쏘 팔콘 7X**     | EASy 비행 갑판           | 하니웰                    | 모듈형 항공전자 장치(MAU) 기반 8                             |
| **ATR 72-600**       | IMA 스위트               | 탈레스                    | A380 수준의 IMA 기술(AFDX 포함)을 지역 터보프롭 시장에 적용 56 |

### 3.2  멀티코어의 난제: 성능, 간섭, 그리고 인증

이 섹션에서는 IMA가 직면한 가장 중요한 현대적 과제, 즉 멀티코어 프로세서의 사용 문제를 다룹니다. 멀티코어는 엄청난 성능 향상을 약속하지만, 그 이면에는 예측 불가능성이라는 치명적인 문제가 도사리고 있습니다.

#### 3.2.1  항공전자 시스템의 멀티코어 프로세서(MCP)로의 전환

항공전자 시스템이 멀티코어 프로세서(MCP)로 전환하는 것은 몇 가지 강력한 요인에 의해 주도되고 있습니다. 복잡한 애플리케이션을 처리하기 위한 컴퓨팅 성능 요구 증대, 더 나은 SWaP 특성, 그리고 상용 기성품(COTS) 시장에서 고성능 단일 코어 프로세서의 가용성 감소가 바로 그것입니다.57

그러나 이 전환에는 근본적인 충돌이 존재합니다. COTS MCP는 일반 소비자나 기업용 컴퓨팅 환경에 맞춰 *평균적인 경우(average-case)*의 성능을 최적화하도록 설계되었습니다. 반면, 안전 필수 항공전자 시스템은 결정론적이고 예측 가능한 동작을 요구하며, 이는 *최악의 경우(worst-case)*에도 성능이 보장되어야 함을 의미합니다.58 이 근본적인 불일치가 바로 멀티코어 인증의 핵심적인 난제입니다.59

#### 3.2.2  간섭 문제: 공유 자원에 대한 경합

MCP에서 가장 큰 문제는 여러 프로세서 코어가 공유 하드웨어 자원을 동시에 사용하려고 경쟁할 때 발생하는 '간섭(interference)'입니다. 이 경쟁은 '간섭 채널(interference channels)'을 생성하며, 시스템의 예측 가능성을 심각하게 훼손할 수 있습니다.57

주요 공유 자원은 다음과 같습니다:

- 최종 레벨 캐시 (L2 또는 L3 캐시)
- 주 메모리 (DRAM)
- 메모리 컨트롤러 및 상호연결/버스
- 입출력(I/O) 장치

한 코어에서 실행되는 소프트웨어가 공유 자원에 접근할 때, 다른 코어에서 실행되는 소프트웨어의 접근을 지연시키거나 방해할 수 있습니다. 이로 인해 특정 작업의 실행 시간이 예측 불가능하게 변동하여, 실시간 시스템에서 치명적인 마감 시간 위반(missed deadline)으로 이어질 수 있습니다.57 단독으로 실행될 때는 완벽하게 작동하던 작업이, 인접 코어에서 특정 다른 작업이 실행될 때 10배나 더 오래 걸리는 현상이 발생할 수도 있습니다.60

#### 3.2.3  간섭 채널 분석 및 완화 기법

멀티코어 시스템의 안전성을 입증하기 위해서는 간섭 채널을 식별, 분석하고 그 영향을 완화해야 합니다. 이는 하향식 분석(top-down analysis)으로 잠재적 간섭 경로를 식별하고, 상향식 분석(bottom-up analysis)으로 그 영향을 정량화하는 2단계 프로세스를 포함합니다.59 이를 위해서는 프로세서의 마이크로아키텍처에 대한 깊은 이해가 필수적입니다.61

주요 완화 기법은 다음과 같습니다:

- **캐시 파티셔닝(Cache Partitioning):** 공유 캐시를 분할하여 각 코어에 전용 영역을 할당하는 기술입니다. 이는 하드웨어적으로(NXP T2080과 같이 프로세서가 지원하는 경우) 또는 소프트웨어적으로("캐시 컬러링") 구현될 수 있습니다. 이를 통해 한 코어의 애플리케이션이 다른 코어의 캐시 라인을 밀어내는 것을 방지합니다.60 하지만 캐시 파티셔닝은 트레이드오프 관계에 있습니다. 간섭의 주요 원인을 제거하여 타이밍 

  *변동성*은 줄일 수 있지만, 각 코어가 사용할 수 있는 캐시 크기가 줄어들어 캐시 미스(cache miss)가 증가하고, 결과적으로 *전체적인 성능*이 저하될 수 있습니다.62

- **메모리/버스 조절(Memory/Bus Regulation):** 특정 "시끄러운 이웃" 코어가 메모리 버스를 독점하는 것을 방지하기 위해 메모리 버스의 대역폭을 조절하는 메커니즘을 구현할 수 있습니다.63

- **스케줄링(Scheduling):** 위상차 실행(phased execution)이나 시분할 다중 접속(TDMA)과 같은 스케줄링 모델을 사용하여, 한 번에 하나의 코어만 공유 메모리에 접근하도록 제한할 수 있습니다. 이는 간섭을 효과적으로 제거하지만, 병렬성의 이점을 희생시켜 전체 처리량을 감소시킵니다.64

#### 3.2.4  인증 지침: CAST-32A와 AC/AMC 20-193 해부

DO-178C와 같은 전통적인 인증 표준은 단일 코어 프로세서를 기준으로 작성되었기 때문에, MCP의 고유한 문제를 다루기 위한 새로운 지침이 필요했습니다.

- **CAST-32A:** 인증 기관 소프트웨어 팀(CAST)은 MCP 인증에 대한 사실상의 전 세계적인 지침이 된 입장 문서, CAST-32A를 발표했습니다.65 이 문서는 간섭 채널과 같은 핵심 문제 영역과 신청자가 충족해야 할 목표들을 명시했습니다.
- **AC 20-193 (FAA) 및 AMC 20-193 (EASA):** FAA의 권고 회람(Advisory Circular)과 EASA의 수용 가능한 규정 준수 수단(Acceptable Means of Compliance)인 이 공식 문서들은 CAST-32A의 지침을 공식화했습니다.57 이들 문서는 신청자에게 모든 간섭 채널을 식별하고, 그 영향을 분석하며, 간섭에도 불구하고 시스템이 안전하고 예측 가능하게 작동할 것이라는 증거를 제출하도록 요구합니다.

#### 3.2.5  멀티코어 검증을 위한 도구 및 기술

- **최악 실행 시간(WCET) 분석:** 핵심 목표 중 하나는 소프트웨어 작업의 WCET를 결정하는 것입니다. MCP에서는 다른 코어의 활동에 따라 실행 시간이 달라지기 때문에 이는 매우 어렵습니다. 정적 분석은 지나치게 비관적인 결과를 낼 수 있고, 측정 기반 접근 방식은 실제 최악의 경우를 실행하지 못할 수 있습니다.67 따라서 정적 분석과 측정 기반 분석을 결합한 하이브리드 접근 방식이 종종 필요합니다.67
- **간섭 생성 도구:** 시스템의 견고성을 테스트하기 위해, 테스트 중에 공유 자원에 의도적으로 경합을 일으키는 특수 도구가 사용됩니다. 예를 들어, 라피타 시스템즈(Rapita Systems)의 RapiDaemon 기술은 테스트 대상 소프트웨어가 실행되는 동안 다른 코어에서 마이크로벤치마크를 실행하여 메모리 버스와 같은 공유 자원에 부하를 줍니다.60 이는 간섭의 영향을 경험적으로 측정하는 데 도움을 줍니다.

멀티코어 문제는 항공우주 산업의 두 가지 주요 트렌드, 즉 비용 절감을 위한 COTS 부품 사용 추세와 타협할 수 없는 결정론적, 인증 가능한 안전성 요구사항 사이의 근본적인 충돌을 나타냅니다. 업계는 시장 가용성과 성능 요구로 인해 MCP를 사용할 수밖에 없는 상황에 처해 있습니다.57 그러나 이 COTS MCP들은 DO-178C가 요구하는 최악의 경우의 결정성이 아닌, 평균적인 경우의 성능에 최적화되어 있어 '예측 가능성 격차'를 만듭니다.58 CAST-32A, AC 20-193, 분석 도구, 완화 기법 등 멀티코어 인증을 둘러싼 모든 노력은 이 격차를 메우기 위한 시도입니다. 즉, 비결정론적인 COTS 플랫폼에 결정론적인 동작을 강제하려는 노력입니다. 이는 항공전자 개발자에게 막대한 기술적, 재정적 부담을 안겨줍니다. 그들은 간섭이 제한되고 수용 가능하다는 것을 증명하기 위해 복잡한 분석에 수백만 달러를 지출하거나, 캐시 파티셔닝이나 TDMA 스케줄링과 같은 기술을 사용하여 MCP의 성능을 의도적으로 저하시켜야 합니다. 이는 MCP를 채택하게 만든 바로 그 성능상의 이점을 희생시키는 것입니다. 이것이 바로 현대 IMA 개발의 중심적인 트레이드오프이자 '난제'입니다.

| 과제 (AC/AMC 20-193 기준) | 설명                                                         | 잠재적 완화 기법                                             |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **공유 캐시 간섭**        | 한 코어가 다른 코어의 데이터를 밀어내는 등 공유 L2/L3 캐시에 대한 경합. | 하드웨어 또는 소프트웨어 캐시 파티셔닝(컬러링). 60           |
| **공유 메모리/버스 간섭** | 메모리 컨트롤러 및 시스템 버스에 대한 경합으로 인한 접근 지연. | 메모리 대역폭 조절, 위상차/TDMA 스케줄링. 63                 |
| **I/O 간섭**              | 여러 코어가 공유 I/O 주변 장치에 대한 접근을 위해 경합.      | I/O 가상화, 전용 I/O 컨트롤러, 접근 스케줄링.                |
| **WCET 예측 불가능성**    | "시끄러운 이웃"으로 인한 간섭으로 작업의 실행 시간이 매우 가변적이 됨. | 변동성을 줄이기 위한 위 기법들의 조합, 간섭 생성을 포함한 강력한 하이브리드 타이밍 분석. 60 |
| **데이터/제어 커플링**    | 공유 하드웨어 상태를 통해 다른 코어의 파티션 간에 의도치 않은 통신 또는 영향 발생. | 강력한 파티셔닝, 은밀한 채널을 식별하고 제거하기 위한 상세한 마이크로아키텍처 분석. |

### 3.3  차세대 기술: DIMA, EMA, 그리고 지능형 항공전자

현재의 IMA를 넘어, 이 섹션에서는 그 진화 과정과 새로운 기술의 통합을 탐구합니다. 이는 IMA가 어떻게 더 분산되고, 지능화되며, 효율적으로 개발될 수 있는지에 대한 비전을 제시합니다.

#### 3.3.1  분산형 IMA (DIMA): 아키텍처의 탈중앙화

DIMA(Distributed IMA) 또는 2세대 IMA(IMA2G)는 중앙 집중식 IMA 개념의 진화된 형태입니다. 모든 처리를 하나 또는 두 개의 대형 캐비닛에 집중하는 대신, DIMA는 더 작은 하드웨어 장치들을 항공기 전체에 분산시키고, 이를 안전 필수 네트워크로 연결합니다.17

보잉 787이 수많은 RDC를 사용하는 것이 DIMA 아키텍처의 대표적인 예이며 25, 에어버스 A350 역시 A380보다 더 분산된 모델을 채택하고 있습니다.25

DIMA의 장점은 다음과 같습니다:

- **유연성 및 SWaP:** DIMA는 하드웨어 배치에 더 큰 유연성을 제공하며, I/O 및 처리를 관련 센서 및 액추에이터에 더 가깝게 배치함으로써 배선 무게와 길이를 더욱 줄여줍니다.17
- **향상된 내고장성:** DIMA는 동적 재구성을 통해 내고장성을 향상시킵니다. 분산된 공유 예비 컴퓨팅 자원 풀을 통해, 고장 난 모듈의 기능들을 항공기 다른 곳에 있는 정상적인 예비 모듈로 동적으로 재할당하고 재시작할 수 있습니다.17

#### 3.3.2  확장된 모듈형 항공전자 (EMA): 2세대 IMA의 비전

EMA(Extended Modular Avionics)는 탈레스가 에어버스 등과 협력하여 주도하는 차세대 IMA를 정의하기 위한 연구 개발 이니셔티브입니다.26

이 프로젝트는 비행 필수 기능부터 승객 서비스에 이르기까지 기내 전자 장치의 모든 측면을 포괄하여 IMA2G의 기반을 마련하는 것을 목표로 합니다. 핵심 목표 중 하나는 현재의 IMA 대비 크기, 무게, 전력(SWaP) 및 케이블링 비용을 50% 절감하여 시스템 통합에 새로운 가능성을 여는 것입니다.26

#### 3.3.3  IMA 개발 및 인증을 위한 모델 기반 시스템 엔지니어링 (MBSE)

DIMA와 같이 압도적으로 많은 구성 가능성을 가진 IMA 시스템의 복잡성은 전통적인 문서 기반 설계 프로세스를 비효율적이고 오류에 취약하게 만듭니다.25

이러한 문제에 대한 해결책으로 모델 기반 시스템 엔지니어링(Model-Based Systems Engineering, MBSE)이 부상하고 있습니다. MBSE는 정적인 문서 대신, 공식적이고 상호 연결된 디지털 모델(예: SysML 사용)을 시스템 설계, 분석 및 검증의 중심적인 정보 소스로 사용하는 패러다임입니다.72

IMA에 대한 MBSE의 적용 분야는 다음과 같습니다:

- **자동화된 구성:** MBSE 도구를 사용하여 전체 IMA 아키텍처(모듈, 파티션, 네트워크 연결)를 모델링할 수 있습니다. 이 단일 소스 모델로부터 복잡한 ARINC 653 및 ARINC 664 XML 구성 파일을 자동으로 생성하여 수동 오류를 줄이고 일관성을 보장할 수 있습니다.38
- **검증 및 확인:** 모델을 초기 단계의 분석 및 시뮬레이션에 사용하여, 설계자가 하드웨어가 준비되기 훨씬 전에 요구사항을 검증하고 통합 문제를 발견할 수 있도록 합니다.70
- **인증 지원:** MBSE는 상위 수준의 안전 요구사항부터 특정 설계 요소 및 검증 결과물까지 명확한 추적성을 제공하며, 이는 인증에 필요한 증거를 생성하는 데 매우 중요합니다.70

#### 3.3.4  AI의 여명: 미래 항공전자 시스템에서의 인공지능과 머신러닝의 역할

인공지능(AI)과 머신러닝(ML)은 효율성, 자율성, 안전성을 향상시키기 위해 항공전자 시스템에 점점 더 많이 탐색되고 있는 기술입니다.75

잠재적인 적용 분야는 다음과 같습니다:

- **예측 정비:** AI 알고리즘은 실시간 센서 데이터를 분석하여 부품 고장을 사전에 예측하고, 정비 일정을 최적화하며 안전성을 향상시킬 수 있습니다.75
- **상황 인식 강화:** AI는 여러 센서의 데이터를 융합하여 조종사에게 전투 공간이나 비행 환경에 대한 더 명확하고 직관적인 이해를 제공하고, 경보의 우선순위를 정하여 인지 부하를 관리하는 데 도움을 줄 수 있습니다.76
- **자율 시스템:** AI/ML은 무인 항공기(UAV) 및 미래의 자율 항공기에서 자율 항법, 의사 결정, 제어의 핵심 동력입니다.51

그러나 많은 AI/ML 알고리즘의 비결정론적 특성은 전통적인 항공전자 인증(DO-178C)의 결정론적 원칙과 정면으로 충돌합니다. 안전 필수 애플리케이션에 머신러닝 시스템을 인증하는 문제를 해결하기 위해 새로운 방법론과 참조 아키텍처가 제안되고 있습니다.78

IMA의 미래 궤적은 1세대의 성공과 도전 과제에 대한 직접적인 대응입니다. DIMA는 초기 IMA의 물리적 중앙 집중화 한계를 해결하고, MBSE는 IMA의 통합 능력이 낳은 막대한 설계 복잡성을 해결합니다. AI/ML은 IMA가 구축된 바로 그 결정론적 기반에 도전하는 잠재적인 패러다임 전환을 나타냅니다. 1세대 IMA가 프로세서를 대형 캐비닛에 중앙 집중화했다면 25, DIMA는 이를 분산시킵니다.17 이는 명백한 아키텍처 진화입니다. IMA가 가능하게 한 통합은 시스템을 너무 복잡하게 만들어 수동적인 문서 기반 설계가 주요 병목 현상이 되었습니다.70 이 문제는 MBSE라는 더 강력한 설계 방법론에 대한 수요를 창출했습니다.71 즉, 새로운 기능이 새로운 복잡성을 낳고, 이는 다시 이를 관리하기 위한 새로운 도구와 방법을 요구하는 순환 관계가 존재합니다. DIMA와 MBSE는 별개의 추세가 아니라, 원래의 IMA 개념이 만든 문제와 기회에 대한 공동 진화적 해결책입니다. AI/ML의 잠재적 통합은 다음의 주요 파괴적 변화입니다. DIMA와 MBSE가 기존의 결정론적 프레임워크 내에서의 진화적 단계라면, AI는 비결정론을 도입합니다. 이는 DO-178C의 탄생 이래 항공전자 인증 철학에 가장 중요한 변화를 강요할 수 있습니다. 업계는 필요한 안전 보증을 제공하기 위해 학습 시스템의 동작을 어떻게 제한할 것인지 알아내야 하며, 이는 아직 해결되지 않은 문제입니다.

## 4.  종합 및 제언

이 마지막 파트에서는 보고서의 핵심적인 발견들을 종합하여, IMA 패러다임과 그것이 항공우주 산업에 미친 영향에 대한 전체적인 평가를 제공합니다.

### 4.1  IMA 패러다임에 대한 종합적 평가

#### 4.1.1  이점의 종합: SWaP-C 및 신뢰성 향상에 대한 정량적 고찰

IMA는 항공전자 시스템 설계에 혁명을 가져왔으며, 그 이점은 질적인 측면을 넘어 정량적으로도 입증되었습니다.

- **SWaP 감소:** 연합형 시스템을 IMA로 교체한 28개 프로젝트에 대한 연구에 따르면, 크기는 28-50%, 전력 소비는 38-60%, 중량은 25-50% 감소한 것으로 나타났습니다.7 이는 항공기의 연료 효율성과 탑재 능력에 직접적인 긍정적 영향을 미칩니다.
- **신뢰성 및 유지보수성 향상:** LRU, 커넥터, 전원 공급 장치의 수를 줄임으로써, IMA는 고장 날 수 있는 부품의 수를 본질적으로 감소시켜 하드웨어 신뢰성을 향상시킵니다.47 공통 모듈의 사용은 군수 지원과 유지보수를 간소화합니다.6 연구에 따르면 평균 고장 간격(MTBF)이 2배에서 3.5배까지 개선된 것으로 나타났습니다.33
- **유연성 및 수명 주기 비용 절감:** IMA의 모듈성과 개방형 표준 사용은 더 쉬운 업그레이드를 촉진하고, 변경 비용을 절감하며, 하드웨어 노후화 문제를 더 효과적으로 관리하는 데 도움을 줍니다.13

#### 4.1.2  복잡성 탐색: 개발 및 통합 과제 관리 전략

IMA가 제공하는 수많은 이점에도 불구하고, 그 구현은 상당한 도전을 수반합니다.

- **시스템 복잡성:** 높은 수준의 통합은 예측하고 분석하기 어려운 의도치 않은 상호작용과 복잡한 연쇄 고장 모드의 가능성을 만듭니다.20
- **통합 및 인증:** 여러 공급업체의 구성 요소를 통합하고 인증하는 것은 주요 과제이며, 표준에 대한 엄격한 준수와 상당한 검증 노력을 요구합니다.19
- **멀티코어 장벽:** 6절에서 자세히 설명했듯이, MCP를 인증하는 것은 현대 IMA 프로그램에 있어 가장 심각한 기술적, 재정적 과제로 남아 있습니다.

이러한 과제들을 성공적으로 해결하기 위해서는 강력한 아키텍처 설계(DIMA), 진보된 방법론(MBSE), 그리고 엄격하고 증거 기반의 인증 전략을 결합한 총체적인 접근 방식이 필요합니다.

#### 4.1.3  IMA의 지속적인 영향과 미래 진화에 대한 결론

결론적으로, 통합 모듈형 항공전자(IMA)는 더 이상 단순한 아키텍처가 아니라, 미래 항공의 기반이 되는 핵심 플랫폼으로 자리 잡았습니다. 이는 자율성 증대, AI 기반 의사 결정 지원, 초연결 항공기와 같은 차세대 역량이 구축될 반석입니다.

연합형 아키텍처의 물리적 한계를 극복하며 등장한 IMA는 SWaP-C 절감과 신뢰성 향상이라는 명백한 이점을 제공하며 항공전자 시스템의 표준이 되었습니다. 그러나 IMA가 가능하게 한 고도의 통합은 전례 없는 시스템 복잡성이라는 새로운 과제를 낳았습니다. 이러한 복잡성을 관리하기 위해 DIMA와 같은 분산형 아키텍처와 MBSE와 같은 설계 방법론이 등장했으며, 이는 기술 발전의 자연스러운 순환 과정을 보여줍니다.

현재 업계는 멀티코어 프로세서 인증이라는 또 다른 변곡점에 서 있습니다. 이는 COTS 기술 활용이라는 경제적 요구와 항공우주의 절대적인 안전성 요구 사이의 긴장을 해결해야 하는 과제입니다. 이 문제를 어떻게 해결하느냐가 향후 수십 년간 항공전자 시스템의 발전 방향을 결정할 것입니다.

궁극적으로, IMA에서 DIMA로, 그리고 MBSE에 의해 관리되며 결국 AI를 통합하게 될 진화의 과정은 현대 항공우주 시스템 엔지니어링의 중심 서사를 구성합니다. 이 여정은 계속해서 더 안전하고, 더 효율적이며, 더 유능한 항공기를 향한 혁신을 이끌어 갈 것입니다.

#### **참고 자료**

1. Metrics for Integrated Modular Avionics Architecture - Lund University Publications, accessed July 6, 2025, https://lup.lub.lu.se/student-papers/record/8923451/file/8923454.pdf
2. Transitioning from federated avionics architectures to Integrated Modular Avionics - ResearchGate, accessed July 6, 2025, https://www.researchgate.net/publication/4295132_Transitioning_from_federated_avionics_architectures_to_Integrated_Modular_Avionics
3. CIVIL AIRCRAFT ADVANCED AVIONICS ARCHITECTURES – AN INSIGHT INTO SARAS AVIONICS, PRESENT AND FUTURE PERSPECTIVE CM Ananda - CORE, accessed July 6, 2025, https://core.ac.uk/download/pdf/11874268.pdf
4. Avionics Architectures - Civil Avionics Systems - Wiley Online Library, accessed July 6, 2025, https://onlinelibrary.wiley.com/doi/10.1002/9781118536704.ch5
5. Resource partitioning for Integrated Modular Avionics: comparative study of implementation alternatives - Han - 2014 - Software: Practice and Experience - Wiley Online Library, accessed July 6, 2025, https://onlinelibrary.wiley.com/doi/abs/10.1002/spe.2210
6. 항공 전자 시스템의 통합 아키텍처 ✈️ - 재능넷, accessed July 6, 2025, https://www.jaenung.net/tree/20376
7. SWaP Reduction: Vital for Choice of Avionics Architecture - ResearchGate, accessed July 6, 2025, https://www.researchgate.net/publication/308344854_SWaP_Reduction_Vital_for_Choice_of_Avionics_Architecture
8. Integrated modular avionics - Wikipedia, accessed July 6, 2025, https://en.wikipedia.org/wiki/Integrated_modular_avionics
9. Lockheed F-22 Raptor - Helitavia, accessed July 6, 2025, https://helitavia.com/avionics/TheAvionicsHandbook_Cap_32.pdf
10. Lockheed F22 Raptor The Definition of Stealth. - Full Afterburner, accessed July 6, 2025, http://fullafterburner.weebly.com/aerospace/lockheed-f22-raptor-the-definition-of-stealth
11. A Framework for IMA-based Architecture Design with Unmanned Aerial System (UAS) as a Test Case - CEUR-WS, accessed July 6, 2025, https://ceur-ws.org/Vol-2814/paper-A4-1.pdf
12. A Distributed Integrated Modular Avionics platform for Multi-Mission Vehicles - Fenix, accessed July 6, 2025, https://fenix.tecnico.ulisboa.pt/downloadFile/281870113705295/Thesis.pdf
13. Modular Avionics Mastery - Number Analytics, accessed July 6, 2025, https://www.numberanalytics.com/blog/ultimate-guide-modular-avionics
14. IMA in Avionics: A Comprehensive Guide, accessed July 6, 2025, https://www.numberanalytics.com/blog/ultimate-guide-ima-avionics
15. Safety-Critical Software Development for Integrated Modular Avionics - Wind River Systems, accessed July 6, 2025, https://www.windriver.com/resource/safety-critical-software-development-for-integrated-modular-avionics
16. (PDF) Integrated Modular Avionics — Past, present, and future - ResearchGate, accessed July 6, 2025, https://www.researchgate.net/publication/284113137_Integrated_Modular_Avionics_-_Past_present_and_future
17. DYNAMIC RECONFIGURATION MECHANISM FOR DISTRIBUTED INTEGRATED MODULAR AVIONICS SYSTEM - Aerospace Research Central, accessed July 6, 2025, https://arc.aiaa.org/doi/pdf/10.2514/6.2015-3285
18. AMC 20-170 'Integrated modular avionics (IMA)' - EASA, accessed July 6, 2025, https://www.easa.europa.eu/sl/downloads/48264/en
19. Real Time Operating Systems and Component Integration Considerations for Integrated Modular Avionics - FAA, accessed July 6, 2025, https://www.faa.gov/sites/faa.gov/files/aircraft/air_cert/design_approvals/air_software/AR-07-48.pdf
20. Certification concerns of Integrated Modular Avionics (IMA) systems - ResearchGate, accessed July 6, 2025, https://www.researchgate.net/publication/224357947_Certification_concerns_of_Integrated_Modular_Avionics_IMA_systems
21. Boeing 787 Common Core System (CCS) : A Technical Breakdown, accessed July 6, 2025, https://oat.aero/2024/04/17/boeing-787-common-core-system-ccs/
22. Common Core System (CCS) in Boeing 787-8: A Summary - Online Aviation Training, accessed July 6, 2025, https://oat.aero/2023/09/14/common-core-system-ccs-in-boeing-787-8-a-summary/
23. I/O Modules: Enabling Device Connectivity and Control - Machine Metrics, accessed July 6, 2025, https://www.machinemetrics.com/connectivity/hardware/io-modules
24. The Critical Role of I/O Modules - Global Electronic Services, accessed July 6, 2025, https://gesrepair.com/what-is-i-o-module/
25. A Survey of Optimal Hardware and Software Mapping for Distributed Integrated Modular Avionics Systems - MDPI, accessed July 6, 2025, https://www.mdpi.com/2076-3417/10/8/2675
26. Taking integration and connectivity to the next level - Thales ..., accessed July 6, 2025, https://onboard.thalesgroup.com/taking-integration-connectivity-next-level/
27. Avionics Full-Duplex Switched Ethernet - Wikipedia, accessed July 6, 2025, https://en.wikipedia.org/wiki/Avionics_Full-Duplex_Switched_Ethernet
28. AFDX®/ARINC664P7 Tutorial - What is AFDX - AIM Online, accessed July 6, 2025, https://www.aim-online.com/products-overview/tutorials/afdx-arinc664p7-tutorial/
29. Arinc 664 p7 - AFDX Tutorial & Products - Excalibur Systems, accessed July 6, 2025, https://www.mil-1553.com/arinc-664p7-afdx
30. The Evolution and Significance of AFDX in Modern Aircraft Systems, accessed July 6, 2025, https://www.logic-fruit.com/blog/avionics/afdx/
31. A Comprehensive Guide to ARINC 664 Part 7 Standard, accessed July 6, 2025, https://arincinsider.com/navigating-the-skies-a-comprehensive-guide-to-arinc-664-part-7-standard/
32. GE Aviation Common Core System for the 787 Dreamliner marks debut of open architecture approach for this type of commercial aviation system - Green Car Congress, accessed July 6, 2025, https://www.greencarcongress.com/2011/10/geccs-20111028.html
33. A Distributed Integrated Modular Avionics platform for Multi-Mission Vehicles - Fenix, accessed July 6, 2025, https://fenix.tecnico.ulisboa.pt/downloadFile/281870113705296/ExtendedAbstract.pdf
34. ARINC 653 - Wikipedia, accessed July 6, 2025, https://en.wikipedia.org/wiki/ARINC_653
35. Space &Time Partitioning with ARINC 653 and pragma Profile - SIGAda, accessed July 6, 2025, http://sigada.org/ada_letters/dec2003/11_Tokar_final.pdf
36. ARINC 653 on PikeOS - SYSGO, accessed July 6, 2025, https://www.sysgo.com/arinc-653
37. What Are ARINC 653–Compliant Safety-Critical Applications? - Wind River Systems, accessed July 6, 2025, https://www.windriver.com/solutions/learning/arinc-653-compliant-safety-critical-applications
38. A Streamlined Approach Toward Automated Generation and Validation of ARINC 653–Compliant Avionics Configurations | Journal of Aerospace Information Systems, accessed July 6, 2025, https://arc.aiaa.org/doi/10.2514/1.I011439
39. VxWorks 653 Multi-core Edition Product Overview - Wind River Systems, accessed July 6, 2025, https://www.windriver.com/resource/vxworks-653-product-overview
40. Updates made to VxWorks RTOS by Wind River - Military Embedded Systems, accessed July 6, 2025, https://militaryembedded.com/avionics/safety-certification/updates-made-to-vxworks-rtos-by-wind-river
41. VxWorks 653: ARINC 653–Compliant Platform for IMA Marketplace - Mistral Solutions, accessed July 6, 2025, https://www.mistralsolutions.com/defence_products/wind-river-vxworks-653/
42. Example ARINC 653 Module and VxWorks Application Description configuration, accessed July 6, 2025, https://www.researchgate.net/figure/Example-ARINC-653-Module-and-VxWorks-Application-Description-configuration_fig11_271468795
43. model-driven development of arinc 653 configuration tables - GMV, accessed July 6, 2025, https://www.gmv.com/sites/default/files/content/file/2020/06/03/1/dasc_paper.pdf
44. ARINC 661: the standard behind modern cockpit display systems | Ansys Knowledge, accessed July 6, 2025, https://innovationspace.ansys.com/knowledge/forums/topic/arinc-661-the-standard-behind-modern-cockpit-display-systems/
45. ARINC 661 - Wikipedia, accessed July 6, 2025, https://en.wikipedia.org/wiki/ARINC_661
46. Cockpit display system - Wikipedia, accessed July 6, 2025, https://en.wikipedia.org/wiki/Cockpit_display_system
47. How does Integrated Modular Avionics (IMA) improve reliability and obsolescence mitigation over the federated architecture? - Aviation Stack Exchange, accessed July 6, 2025, https://aviation.stackexchange.com/questions/87321/how-does-integrated-modular-avionics-ima-improve-reliability-and-obsolescence
48. ARINC 653, accessed July 6, 2025, http://retis.sssup.it/~giorgio/slides/cbsd/Biondi3-ARINC.pdf
49. 항공기 임베디드 소프트웨어 기술의 동향 - 테크월드뉴스, accessed July 6, 2025, https://www.epnc.co.kr/news/articleView.html?idxno=45112
50. A close-up view of the Common Integrated Processor which was developed for the F-22 jet fighter and is the equivalent of two Cray supercomputers. The CIP is a little larger than a 20-in television. It is the world's most advanced, high-speed computer system for use on board a fighter aircraft - NARA & DVIDS Public Domain Archive, accessed July 6, 2025, https://nara.getarchive.net/media/a-close-up-view-of-the-common-integrated-processor-which-was-developed-for-704456
51. The Future of Avionics: IMA Technology - Number Analytics, accessed July 6, 2025, https://www.numberanalytics.com/blog/future-avionics-ima-technology
52. F-35_Air_Vehicle_Technology_Overview.pdf - Lockheed Martin, accessed July 6, 2025, https://www.lockheedmartin.com/content/dam/lockheed-martin/eo/documents/webt/F-35_Air_Vehicle_Technology_Overview.pdf
53. Simulation of Aircraft Parameters in High Speed IMA Systems - Avionics Interface Technologies — A Teradyne Company, accessed July 6, 2025, https://aviftech.com/simulating-aircraft-parameters-in-high-speed-ima-systems/
54. Boeing 787 Common Core System - Trans Global Training, accessed July 6, 2025, https://www.transglobaltraining.com/boeing-787-common-core-system/
55. Flight deck, Avionics Equipment & Functions | Thales Group, accessed July 6, 2025, https://www.thalesgroup.com/en/markets/aerospace/flight-deck-avionics-equipment-functions
56. Thales technologies on board ATR 72–600 - Thales Aerospace Blog, accessed July 6, 2025, https://onboard.thalesgroup.com/thales-technologies-on-board-atr-72-600/
57. What does it take to certify multicore avionics for DO-178C?, accessed July 6, 2025, https://www.his-conference.co.uk/session/what-does-it-take-to-certify-multicore-avionics-for-do-178c
58. Enabling Multi-Core Processing in DO-254/178 Safety-Certifiable Avionics - Mistral Solutions, accessed July 6, 2025, https://mistralsolutions.com/newsletter/Jul20/Enabling-Multi-Core-Processing-in-DO-254-178-Safety-Certifiable-Avionics-white-paper.pdf
59. Assurance of Multicore Processors in Airborne Systems - FAA, accessed July 6, 2025, https://www.faa.gov/sites/faa.gov/files/aircraft/air_cert/design_approvals/air_software/TC-16-51.pdf
60. Multicore Timing Solution | Rapita Systems, accessed July 6, 2025, https://www.rapitasystems.com/services/multicore-timing
61. Mitigation of interference in Multicore Processors | Page A - Wind River Systems, accessed July 6, 2025, https://www.windriver.com/sites/default/files/2024-10/656381-mitigation-of-interference-in-MC-processors.pdf
62. Is cache partitioning helpful or harmful to multicore timing performance? - Rapita Systems, accessed July 6, 2025, https://www.rapitasystems.com/products/faqs/cache-partitioning-helpful-or-harmful-multicore-timing-performance
63. A Survey of Techniques for Reducing Interference in Real-Time Applications on Multicore Platforms - SciSpace, accessed July 6, 2025, https://scispace.com/pdf/a-survey-of-techniques-for-reducing-interference-in-real-e9msev69.pdf
64. Execution Model to Reduce the Interference of Shared Memory in ARINC 653 Compliant Multicore RTOS - MDPI, accessed July 6, 2025, https://www.mdpi.com/2076-3417/10/7/2464
65. AMC 20-193 & CAST-32A Multi-Core Processing Training - AFuzion, accessed July 6, 2025, https://afuzion.com/aviation-safety-private-training/amc-20-193-multi-core-processing-with-cast-32a-training/
66. AC 20-193 - Use of Multi-Core Processors - FAA, accessed July 6, 2025, https://www.faa.gov/regulations_policies/advisory_circulars/index.cfm/go/document.information/documentID/1036408
67. Multicore Processors in Avionics - Visure Solutions, accessed July 6, 2025, https://visuresolutions.com/aerospace-and-defense/multicore-processors/
68. A Distributed Platform for Integrated Modular Avionics, accessed July 6, 2025, https://my.svh-verlag.de/catalogue/details/de/978-3-8381-0049-4/a-distributed-platform-for-integrated-modular-avionics
69. Integrated Modular Avionics Archives - Thales Aerospace Blog, accessed July 6, 2025, https://onboard.thalesgroup.com/tag/integrated-modular-avionics/
70. Model-Based IMA Platform Development and Certification Ecosystem, accessed July 6, 2025, https://www.researchgate.net/publication/375570374_Model-Based_IMA_Platform_Development_and_Certification_Ecosystem
71. Model-Based Systems Engineering and Model-Based Safety Analysis: Final Report - ROSA P, accessed July 6, 2025, https://rosap.ntl.bts.gov/view/dot/57838/dot_57838_DS1.pdf
72. Model Based System Engineering (MBSE) with SysML Certification Training, accessed July 6, 2025, https://apmg-international.com/product/model-based-system-engineering-mbse-sysml-certification-training
73. Enroll in MIT's Model-based Systems Engineering Online Course - MIT xPRO, accessed July 6, 2025, https://xpro.mit.edu/courses/course-v1:xPRO+SysEngx3/
74. Applied Model-Based Systems Engineering (MBSE) – Online Short Course (17–20 November 2025) - AIAA - Shaping the future of aerospace, accessed July 6, 2025, https://aiaa.org/courses/applied-model-based-systems-engineering-mbse/
75. The Future of Avionics Modernization - Number Analytics, accessed July 6, 2025, https://www.numberanalytics.com/blog/future-of-avionics-modernization
76. How AI Is Transforming Avionics in North America - MarketsandMarkets, accessed July 6, 2025, https://www.marketsandmarkets.com/ResearchInsight/avionics-in-north-america.asp
77. Smarter skies: How AI and open architectures are changing military avionics, accessed July 6, 2025, https://militaryembedded.com/avionics/displays/smarter-skies-how-ai-and-open-architectures-are-changing-military-avionics
78. AI in Avionics - AVweb, accessed July 6, 2025, https://avweb.com/features/ai-in-avionics/
79. The Future of Avionics - Number Analytics, accessed July 6, 2025, https://www.numberanalytics.com/blog/future-avionics-aerospace-engineering
80. The Future of AI and Machine Learning in Avionics Systems - Tech Briefs, accessed July 6, 2025, https://www.techbriefs.com/component/content/article/47715-the-future-of-ai-and-machine-learning-in-avionics-systems
81. Improving Hazard Analysis and Certification of Integrated Modular Avionics - Nancy Leveson, accessed July 6, 2025, http://sunnyday.mit.edu/STAMP/Final-IMA.pdf


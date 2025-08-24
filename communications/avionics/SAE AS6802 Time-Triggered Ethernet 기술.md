[항공을 위한 통신](./index.md)
# SAE AS6802 Time-Triggered Ethernet 기술



표준 IEEE 802.3 이더넷은 본질적으로 '최선 노력(best-effort)' 기반의 전달 모델을 따릅니다.1 이는 데이터가 제시간에, 올바른 순서로, 혹은 반드시 도착한다는 것을 보장하지 않음을 의미합니다. 과거의 CSMA/CD와 같은 이더넷의 매체 접근 제어(MAC) 메커니즘은 예측 가능한 지연 시간(latency)보다는 공정한 중재와 높은 평균 처리량에 초점을 맞춰 설계되었습니다.3

이러한 최선 노력 방식의 특성은 표준 이더넷을 플라이 바이 와이어(fly-by-wire) 항공기나 자율주행차 제어 시스템과 같은 하드 실시간(hard real-time) 제어 루프에 근본적으로 부적합하게 만듭니다.4 이러한 시스템에서는 메시지 하나가 누락되거나 지연되는 것이 치명적인 결과를 초래할 수 있습니다. 네트워크의 타이밍은 예측 불가능하며, 네트워크 부하와 경합으로 인해 발생하는 지터(jitter)에 취약합니다.2

TTEthernet, TSN, EtherCAT 등을 포함한 '산업용 이더넷' 및 결정론적 프로토콜 제품군의 등장은 이더넷의 경제적, 물리적 이점(예: 높은 대역폭, 상용 기성품(COTS) 구성 요소, 폭넓은 기술 인지도)과 운영 기술(OT) 및 안전 최우선 시스템의 엄격한 시간적 요구사항을 조화시키려는 근본적인 동기에서 비롯되었습니다. 이 중에서도 TTEthernet은 본질적으로 확률적인 이더넷의 기반 위에 절대적인 예측 가능성을 부여하려는 가장 야심 찬 시도 중 하나를 대표합니다. 표준 이더넷은 보편적이고 비용 효율적이지만 8, 그 핵심 설계는 기계 제어(예측 가능성, 보장된 지연 시간)가 아닌 사무 환경(공정성, 평균 속도)에 맞춰져 있습니다.1 이로 인해 근본적인 충돌이 발생하며, 사무용 LAN으로 비행기를 조종할 수는 없습니다. 따라서 이더넷의 동작을 '길들이기' 위한 새로운 서비스 계층이 필요하며, SAE AS6802는 이더넷의 물리 및 데이터 링크 계층 위에 결정론적인 시간 트리거 제어 계층을 추가하는 서비스 집합입니다.3 이는 TTEthernet을 이더넷의 대체 기술이 아닌, 그 동작을 근본적으로 수정하는 기술로 자리매김하게 합니다.


시간 트리거(Time-Triggered, TT) 패러다임은 시스템 활동, 특히 통신이 전역적인 시스템 전체 스케줄에 따라 미리 정해진 시점에 시작되어야 한다고 규정합니다. 이는 이벤트(예: 메시지 도착)에 대한 응답으로 활동이 시작되는 이벤트 트리거(event-triggered) 시스템과 대조됩니다.13

시간 트리거 아키텍처는 '시간적 분할(temporal partitioning)'을 제공하여 한 구성 요소의 동작이 다른 구성 요소의 시간적 동작에 영향을 미치지 않도록 보장하며, 이는 혼합 중요도 시스템(mixed-criticality systems)을 통합하는 데 매우 중요합니다. 이 접근 방식은 예측 가능한 지연 시간과 지터를 보장하고, 시스템 구성 요소를 독립적으로 개발 및 검증한 후 예기치 않은 시간적 간섭 없이 통합할 수 있는 '구성 가능성(composability)'을 가능하게 합니다.14 TTEthernet은 공유된 전역 시간 개념을 사용하여 모든 핵심 통신을 조율함으로써 이 패러다임을 네트워킹에 적용합니다.2

TTEthernet은 TTP(SAE AS6003) 및 FlexRay와 같은 프로토콜의 원칙을 이더넷의 더 높은 대역폭 영역으로 확장하여 네트워크 시스템에 대한 시간 트리거 철학을 기술적으로 구현한 것입니다. 그 목표는 단순히 '빠른 실시간 네트워크'가 되는 것을 넘어, 네트워크의 시간적 동작이 시스템의 우발적 속성이 아닌 설계 단계에서 이미 알려진 매개변수가 되는, 구성 가능하고 검증 가능한 시스템 아키텍처를 만드는 데 있습니다. 현대 항공기나 자동차와 같은 복잡한 시스템은 비행 제어와 기내 엔터테인먼트처럼 중요도가 다른 기능들을 통합합니다.8 이벤트 트리거 시스템에서는 중요하지 않은 기능의 '떠드는 바보(babbling idiot)' 결함이 네트워크를 마비시켜 중요한 기능들을 방해할 수 있습니다.2 시간 트리거 패러다임은 중요한 메시지에 미리 할당된 독점적인 시간 슬롯을 제공함으로써 이러한 문제를 방지합니다. 중요하지 않은 기능은 해당 시간에 전송하도록 스케줄되지 않았기 때문에 중요한 메시지의 슬롯을 간섭할 수 없습니다.14 따라서 TTEthernet이 이 패러다임을 채택한 것은 안전하고 통합된 혼합 중요도 시스템을 구축하는 과제에 대한 직접적인 해결책이며, 최대 대역폭 활용이나 유연성보다 예측 가능하고 검증 가능한 안전성을 우선시합니다.


공식 명칭이 "Time-Triggered Ethernet"인 SAE AS6802는 SAE International의 AS-2D2 위원회에 의해 2011년 11월에 처음 발표되었습니다.8 이후 재확인 및 개정을 거쳐 2023년 2월에 최신 버전인 AS6802A가 발표되었습니다.16 이 표준은 TTTech Computertechnik AG와 주요 산업 파트너들에 의해 개발되었습니다.11

이 표준의 목표는 이더넷의 비동기 통신 기능을 유지하면서 시간, 안전, 임무에 민감한 모든 임베디드 애플리케이션에 이더넷을 적용할 수 있도록 하는 것이었습니다.3 이 표준은 고장 감내 동기화 및 동기식 시간 트리거 패킷 스위칭에 필요한 서비스를 명시적으로 정의합니다.8 항공우주 및 자동차 산업에 중점을 둔 표준화 기구인 SAE 내에서 제정되었다는 점은 이 기술의 주요 적용 분야를 명확히 보여줍니다.18

SAE AS6802의 비전은 또 다른 독점적인 산업용 이더넷을 만드는 것이 아니라, 표준 이더넷에 추가할 수 있는 보편적이고 결정론적인 '서비스'를 창출하는 것이었습니다. 이를 통해 하드 실시간 제어, 스트리밍 데이터, 표준 LAN 트래픽이 동일한 물리적 회선에 공존하는 통합 네트워크 아키텍처를 구현할 수 있게 되며, 이는 항공우주 및 자동차 설계에서 주요 동인인 시스템 설계 단순화, 무게 및 비용 절감을 가능하게 합니다.3 전통적인 연합형 항공전자 시스템은 각 기능마다 MIL-STD-1553(제어용)이나 ARINC 429(데이터용)와 같은 별도의 전용 버스를 사용하여 복잡한 배선, 높은 무게, 그리고 복잡성을 야기했습니다.6 통합 모듈형 항공전자(Integrated Modular Avionics, IMA) 개념은 공유된 컴퓨팅 및 네트워킹 자원을 사용하여 이러한 복잡성을 줄이고자 합니다.5 IMA를 실현하기 위한 핵심 요소는 다양한 중요도를 가진 여러 기능을 안전하게 호스팅할 수 있는 네트워크입니다. SAE AS6802는 단일 고대역폭 이더넷 네트워크에서 이러한 안전한 통합을 가능하게 하는 규칙 집합(동기화, 스케줄링, 분할)을 제공함으로써 IMA 비전을 직접적으로 실현합니다.2



TTEthernet은 분산된 마스터 없는(master-less) 클럭 동기화 프로토콜을 통해 전역 시간 기반을 구축합니다. 이 프로토콜은 동기화 마스터(Synchronization Master, SM), 압축 마스터(Compression Master, CM), 그리고 동기화 클라이언트(Synchronization Client, SC)라는 세 가지 역할을 포함합니다.5

이 동기화 과정은 "통합 주기(integration cycles)" 내에서 발생합니다. 일반적으로 종단 시스템(end system)인 SM은 프로토콜 제어 프레임(Protocol Control Frame, PCF)에 자신의 로컬 시간을 담아 브로드캐스트합니다. 일반적으로 스위치인 CM은 이러한 PCF들을 수신하고, 도착 시간에 고장 감내 압축 함수(예: 중앙값 또는 평균화 알고리즘)를 적용한 후, 보정된 새로운 PCF를 브로드캐스트합니다. 그러면 SM과 SC를 포함한 모든 노드는 이 압축된 값을 사용하여 자신의 로컬 클럭을 보정합니다.5

이 메커니즘은 고장 시에도 운영이 가능한(fail-operational) 시스템을 위해 명시적으로 설계되었습니다. 단일 고장 구성에서는 단일 SM의 비잔틴(Byzantine, 임의의) 고장이나 단일 CM의 불일치 생략(inconsistent-omission) 고장을 허용할 수 있습니다. 이중 고장 구성에서는 여러 개의 생략 고장을 허용할 수 있습니다.12 이러한 견고성은 TTEthernet의 안전성 제안의 핵심입니다. 이 기능은 TTEthernet의 가장 중요하고 독보적인 특징입니다. TSN에서 사용되는 IEEE 1588/802.1AS와 같은 마스터-슬레이브 프로토콜이 '최상의' 마스터 클럭을 선출하는 데 의존하는 반면, TTEthernet의 접근 방식은 단일 장애점(single point of failure) 없이 복원력 있고 고장 감내가 가능한 시스템 시간을 생성합니다. 이러한 알고리즘의 형식적 검증 가능성은 항공우주 분야(예: DO-254/DO-178B) 인증의 핵심 요구사항입니다.3

분산형 비잔틴 고장 감내 동기화 프로토콜의 선택은 가장 까다로운 안전 최우선 항공우주 애플리케이션에서 비롯된 TTEthernet의 기원을 직접적으로 반영합니다. 이 시스템은 구성 요소가 고장 날 것이며, 예측 불가능한 방식(비잔틴 고장)으로 고장 날 수 있다고 가정합니다. 전체 아키텍처는 이러한 고장에도 불구하고 일관된 시간 상태를 유지하도록 구축되었으며, 이는 단순히 백업 마스터 클럭을 두는 것보다 훨씬 강력한 보증입니다. 유인 우주 비행과 같이 생명이 직결된 시스템에서는 단일 장애점이 용납될 수 없습니다. 마스터 클럭은 핫 스탠바이(hot standby)가 있더라도 잠재적인 단일 장애점 또는 복잡한 페일오버(failover) 시나리오를 나타냅니다. 더욱이, 결함이 있는 구성 요소는 충돌하거나 악의적인 타이밍 정보(비잔틴 고장)를 보낼 수 있으며, 단순한 마스터-슬레이브 시스템은 이를 원활하게 처리하지 못할 수 있습니다. SM/CM 아키텍처 19는 시간 유지 책임을 분산시켜 이 문제를 해결합니다. CM은 '투표자' 역할을 하여 압축 함수를 사용해 결함 있는 SM의 비정상적인 값을 폐기하고, 이를 통해 민주적이고 고장 감내가 가능한 시간 기반을 만듭니다. 이러한 설계 선택은 복잡하지만, 안전한 시간 트리거 통신을 위한 절대적인 전제 조건인 시스템의 전역 시간에 대한 최고 수준의 신뢰를 제공합니다. 이는 단순성을 포기하고 궁극적인 견고성을 선택한 직접적인 트레이드오프입니다.


TTEthernet은 가장 중요한 트래픽을 스케줄링하기 위해 시분할 다중 접속(Time-Division Multiple Access, TDMA) 접근 방식을 사용합니다. 통신 스케줄은 시스템 설계 단계에서 오프라인으로 정의되며, 모든 네트워크 장치(스위치 및 종단 시스템)에 로드됩니다.2

시스템 설계자는 네트워크의 각 링크에서 각 시간 트리거(TT) 메시지에 대한 정확한 전송 창을 오프라인으로 정의합니다. 이 스케줄은 런타임에 TT 메시지에 대한 충돌이나 경합이 없도록 보장합니다.2 스케줄은 주기적이며, "통합 주기(integration cycles)"와 모든 메시지 주기의 최소공배수인 더 큰 "클러스터 주기(cluster cycle)"로 구성됩니다.19

스케줄이 경합이 없고 전역 시간과 동기화되어 있기 때문에, TT 메시지는 고정되고 예측 가능한 종단 간 지연 시간과 마이크로초 단위의 극히 낮은 지터를 달성합니다.2 스위치는 프레임을 언제 수신하고 언제 전달해야 하는지 정확히 알고 있습니다.3 이 정적이고 미리 구성된 스케줄링은 TTEthernet에 하드 실시간 결정성을 부여하며, 네트워크를 확률적 시스템에서 결정론적 시스템으로 변환합니다. 이는 긴밀한 폐쇄 루프 제어 애플리케이션에 필수적이며, 최악의 통신 지연이 설계 단계에서 이미 알려진 값이므로 시스템 분석 및 검증을 단순화합니다.

오프라인 TDMA 스케줄링에 대한 의존은 예측 가능성과 유연성 사이의 근본적인 설계 트레이드오프를 나타냅니다. 이는 (인증에 큰 이점이 되는) 타의 추종을 불허하는 결정성을 제공하는 반면, 시스템을 경직되게 만듭니다. 새로운 장치를 추가하거나 메시지의 타이밍을 변경하는 것과 같은 어떠한 변경이라도 전체 네트워크 스케줄을 재계산하고 재배포해야 합니다. 이러한 복잡성과 런타임 유연성의 부족은 보다 동적인 프로토콜인 TSN과의 주요 대조점입니다. 안전을 보장하기 위해 시스템은 분석 가능해야 하며, 최악의 경우의 동작이 알려져 있고 제한되어야 합니다. 오프라인 스케줄링은 시스템이 구축되기 전에도 네트워크의 시간적 동작에 대한 완전한 형식적 분석을 가능하게 하며, 스케줄 가능성은 수학적으로 증명될 수 있습니다.20 이는 시스템의 동작이 완전히 명시되고 검증 가능하므로 FAA나 ESA와 같은 기관의 인증 절차를 훨씬 간단하게 만듭니다. 그러나 이는 운영상의 유연성을 희생하는 대가로 이루어집니다. 시스템은 예기치 않은 변경이나 동적 조건에 쉽게 적응할 수 없습니다. 이러한 경직성은 항공기와 같이 매우 정적인 시스템에서는 수용 가능하지만, 산업 자동화나 소비자 차량과 같이 보다 동적인 환경에서는 상당한 단점이 되며, 이는 TSN이 대안으로 부상한 핵심 이유 중 하나입니다.


TTEthernet은 단일 물리적 네트워크를 통해 세 가지 고유한 트래픽 클래스를 전송하도록 설계되었으며, 이들 사이에는 엄격한 우선순위와 분할이 적용됩니다.8

- **1. 시간 트리거(Time-Triggered, TT) 트래픽:** 가장 높은 우선순위를 가집니다. 독점적으로 미리 스케줄된 TDMA 슬롯에서 전송됩니다. 비행 제어와 같은 안전 최우선, 하드 실시간 기능에 사용됩니다. 고정된 지연 시간과 마이크로초 수준의 지터를 보장합니다.2
- **2. 속도 제한(Rate-Constrained, RC) 트래픽:** 중간 우선순위를 가집니다. TT 트래픽이 스케줄되지 않았을 때 전송됩니다. 대역폭 및 지연 시간 보장을 제공하지만, TT보다 지터가 더 큽니다. 기능적으로 AFDX(ARINC 664p7) 트래픽과 동일하며, 트래픽을 제어하기 위해 대역폭 할당 간격(Bandwidth Allocation Gap, BAG) 개념을 사용합니다.12 오디오/비디오 스트리밍이나 시스템 모니터링과 같이 덜 중요한 실시간 애플리케이션에 사용됩니다.
- **3. 최선 노력(Best-Effort, BE) 트래픽:** 가장 낮은 우선순위를 가집니다. TT 또는 RC 트래픽에 의해 소비되지 않은 나머지 대역폭을 사용합니다. 시간적 보장이 없는 표준 IEEE 802.3 이더넷 트래픽에 해당합니다.14 진단, 파일 전송 또는 인포테인먼트와 같은 비핵심 기능에 사용됩니다.

이 세 계층의 트래픽 시스템은 통합 모듈형 항공전자(IMA)를 가능하게 하는 핵심 메커니즘입니다. 이를 통해 시스템 설계자는 이전에 분리되었던 여러 네트워크를 단일 백본으로 통합하여 무게, 전력 및 비용을 절감하는 동시에, 중요하지 않은 BE 트래픽이 중요한 TT 트래픽의 결정론적 전달을 방해하지 않도록 보장할 수 있습니다.2

AFDX와 기능적으로 동일한 RC 트래픽 클래스를 포함시킨 것은 탁월한 전략적 결정이었습니다. 이는 TTEthernet을 항공전자 분야에서 확립된 AFDX 표준에 대한 파괴적인 경쟁자가 아닌, 호환 가능하고 진화적인 업그레이드로 자리매김하게 했습니다. 항공우주 제조업체는 기존 AFDX 기반 시스템(RC 트래픽으로 실행됨)을 원활하게 통합하면서 새로운 고도의 시간 트리거 기능을 추가하기 위해 TTEthernet을 채택할 수 있었습니다. 이는 보수적인 산업에서 보다 원활한 채택 경로를 만들었습니다. 항공우주 산업은 이미 에어버스 A380과 같은 시스템에서 AFDX(ARINC 664p7)에 막대한 투자를 했습니다.6 완전히 새롭고 호환되지 않는 프로토콜을 도입하는 것은 엄청난 저항에 부딪혔을 것입니다. TTEthernet의 RC 클래스를 AFDX와 호환되도록 설계함으로써 TTTech는 점진적인 채택을 가능하게 했습니다. 시스템은 TTEthernet과 AFDX 장치의 혼합이 될 수 있었습니다. 그런 다음 TT 클래스는 이더넷을 통한 플라이 바이 와이어와 같은 차세대 기능을 위해 AFDX보다 높은 수준의 결정성을 제공하는 핵심적인 부가 가치였습니다. 이는 TTEthernet을 단순한 대안이 아닌 AFDX의 상위 집합으로 만들어, 항공우주 분야에서의 시장 침투에 결정적인 역할을 했습니다.


TTEthernet은 '간섭으로부터의 자유(freedom from interference)'를 보장하기 위해 견고한 분할을 구현합니다. 낮은 중요도의 애플리케이션이나 장치에서 발생한 결함은 봉쇄되어 중요한 시스템에 연쇄적으로 영향을 미치지 않습니다.2

- **시간적 분할:** TT 트래픽에 대한 TDMA 스케줄은 본질적으로 시간적 분할을 제공합니다. 장치는 할당된 슬롯에서만 전송할 수 있습니다.2
- **공간적 분할 / 버스 가디언:** TTEthernet 스위치는 '중앙 버스 가디언(central bus guardian)' 역할을 합니다. 이들은 전역 스케줄을 알고 있으며, 지정된 창 밖에서 도착하거나 잘못된 소스에서 온 프레임을 폐기합니다. 이는 '떠드는 바보' 종단 시스템이 네트워크를 마비시키고 다른 통신을 방해하는 것을 방지합니다.2
- **대역폭 분할:** RC 트래픽의 제어(BAG를 통해)와 BE 트래픽의 낮은 우선순위는 이들이 TT 트래픽에 예약된 대역폭을 소비할 수 없도록 보장합니다.8 만약 TT 슬롯이 사용되지 않으면, 대역폭은 비동기 트래픽을 위해 동적으로 해제되어 효율성을 향상시킬 수 있습니다.2

고장 봉쇄는 안전 인증의 초석입니다. 시스템 설계자는 시스템의 한 부분에서 발생한 고장이 봉쇄됨을 증명할 수 있어야 합니다. TTEthernet은 이러한 봉쇄 기능을 네트워크 하드웨어와 프로토콜에 직접 내장하여 애플리케이션 수준의 고장 관리와 시스템의 전반적인 안전성 주장을 단순화합니다.2

'버스 가디언' 기능이 스위치에 중앙 집중화된 것은 핵심적인 아키텍처 선택입니다. TTP나 FlexRay와 같은 다른 시간 트리거 프로토콜에서는 버스 가디언이 종종 각 노드의 버스 인터페이스 컨트롤러의 일부입니다. 이 중요한 기능을 스위치에 배치함으로써 TTEthernet은 스위치드 이더넷의 스타 토폴로지를 활용하여 더 강력하고 중앙에서 관리 가능한 고장 봉쇄 도메인을 만듭니다. 단일 스위치가 연결된 모든 종단 시스템을 감시할 수 있습니다. 핵심적인 고장 모드 중 하나는 종단 시스템이 '떠드는 바보'가 되어 지속적으로 또는 잘못된 시간에 전송하는 것입니다. 버스 토폴로지에서는 이것이 전체 버스를 다운시킬 수 있습니다. 스위치드 스타 토폴로지에서는 고장이 초기에 결함 있는 노드와 스위치 사이의 링크로 격리됩니다. 스위치에 지능(스케줄을 인지하는 버스 가디언)을 추가함으로써 TTEthernet은 고장이 스위치를 통해 다른 노드로 전파되는 것을 방지합니다. 스위치는 단순히 불법적인 프레임을 폐기합니다.13 이는 스위치를 단순한 패킷 전달자에서 시스템의 안전 아키텍처의 중요한 요소로 변환시켜 전체 시스템의 규칙을 강제합니다.



SAE AS6802는 OSI 모델의 계층 2(데이터 링크 계층)에서 작동하는 서비스 집합을 정의하며, 새로운 계층 3 프로토콜이 아닙니다.3 이는 중요한 아키텍처 세부 사항입니다. TTEthernet이 계층 2에서 작동하고 표준 IEEE 802.3 이더넷 프레임을 사용하기 때문에 TCP/IP와 같은 상위 계층 프로토콜에 투명합니다. 이는 표준 IT 애플리케이션, Wireshark와 같은 도구 및 스택이 수정 없이 TTEthernet 네트워크(BE 트래픽으로)를 통해 실행될 수 있음을 의미하며, 이는 시스템 통합 및 진단에 큰 이점입니다.3 TTEthernet 관련 정보는 표준 프레임 구조 내에 포함됩니다.

TTEthernet을 완전히 새로운 프로토콜 스택이 아닌 계층 2 QoS 향상 기능으로 구현하기로 한 결정은 호환성을 극대화하고 채택을 용이하게 하기 위한 실용적인 선택이었습니다. 이는 설계자가 중요한 제어 루프를 위한 결정론적, 시간 트리거 동작과 그 외 모든 것을 위한 표준 IP 기반 네트워킹의 풍부한 생태계라는 '두 마리 토끼를 모두 잡을 수 있게' 해줍니다. 이 모든 것이 단일 물리적 네트워크에서 가능합니다. 시스템 설계자는 하드 실시간 제어(TT), 스트리밍(RC), 그리고 진단, 파일 전송 또는 웹 인터페이스(BE)와 같은 표준 네트워크 서비스 등 다양한 애플리케이션을 실행해야 합니다. 완전히 새롭고 IP와 호환되지 않는 프로토콜을 만드는 것은 모든 표준 네트워크 애플리케이션과 도구를 다시 작성하거나 포팅해야 하는 엄청난 노력을 요구했을 것입니다. TTEthernet을 계층 2 서비스로 만듦으로써, 표준 IP 패킷(이더넷 프레임에 캡슐화됨)을 BE 트래픽으로 전달할 수 있습니다. TTEthernet 스위치는 이를 단순히 낮은 우선순위 데이터로 취급하고 네트워크가 비어 있을 때 전달합니다.3 이를 통해 예를 들어, 안전 최우선 비행 제어 루프는 TT 트래픽으로 실행되는 동시에, 유지보수 기술자는 표준 노트북을 연결하여 BE 트래픽을 통해 장치의 웹 기반 진단 인터페이스에 접근할 수 있습니다. 이 모든 것이 동일한 네트워크에서 이루어지며, 이것이 바로 '통합 네트워킹' 비전의 본질입니다.3


TTEthernet을 구현하려면 주로 TTEthernet 스위치와 TTEthernet 종단 시스템(네트워크 인터페이스 카드 또는 컨트롤러) 형태의 특수 하드웨어가 필요합니다.8

- **TTE-스위치:** 이는 표준 이더넷 스위치가 아닙니다. AS6802 동기화 프로토콜을 실행하고(CM 역할), TDMA 스케줄을 강제하며, 트래픽 클래스를 감시하고, 버스 가디언 역할을 수행하는 하드웨어와 펌웨어를 포함합니다. 격리를 보장하기 위해 다양한 트래픽 클래스를 위한 전용 메모리 파티션을 가지고 있습니다.13
- **TTE-종단 시스템:** 이는 호스트 컴퓨터/프로세서를 네트워크에 연결하는 네트워크 인터페이스 컨트롤러(NIC) 또는 FPGA/ASIC용 IP 코어입니다. 동기화에서 SM/SC 역할을 구현하고 스케줄에 따라 정확한 순간에 TT 프레임을 전송하는 역할을 합니다.26

특수 하드웨어 요구사항은 TTEthernet의 비용, 복잡성 및 생태계에 중요한 요소입니다. 표준 상용 이더넷 하드웨어에서는 실행할 수 없습니다. 이 하드웨어는 표준이 보장하는 마이크로초 수준의 정밀도와 고장 감내 기능을 제공하는 데 필수적입니다. 특수 하드웨어에 대한 의존성은 표준 하드웨어에서 실행될 수 있는 기술에 비해 더 폐쇄적이고 통제된 생태계를 만듭니다. 이는 고전적인 트레이드오프입니다. 특수 하드웨어는 성능과 신뢰성을 보장하고 인증할 수 있게 하지만, 공급업체 선택을 제한하고 비용을 증가시킬 수 있습니다. 고정밀 클럭 동기화, 스케줄 강제 및 트래픽 감시와 같은 작업은 타임스탬핑 및 의사 결정을 위한 하드웨어 수준의 지원을 필요로 합니다.8 범용 CPU에서의 소프트웨어 전용 구현은 너무 많은 지터와 예측 불가능성을 야기할 것입니다. 이는 맞춤형 ASIC, FPGA 및 전용 스위치 하드웨어의 개발을 필요로 합니다.26 이 하드웨어는 주로 기술의 창시자인 TTTech와 그 긴밀한 파트너들에 의해 개발 및 판매됩니다.27 이는 더 작고 전문화된 시장을 형성합니다. 이는 높은 수준의 품질과 지원을 보장하지만, TSN이 활용하고자 하는 방대하고 상품화된 표준 이더넷 하드웨어 시장에 비해 경쟁이 적고 잠재적으로 가격이 더 높다는 것을 의미합니다.


TTEthernet 생태계는 기술의 원 개발사인 **TTTech Computertechnik AG**(및 그 사업부인 TTTech Aerospace, TTTech Industrial)가 주도하고 있습니다.8 주요 파트너 및 사용자로는 **Honeywell**, **Boeing**, **Airbus**와 같은 주요 항공우주 및 방위 산업체 6, **NASA**와 **ESA**와 같은 우주 기관 8, **RUAG Space**(우주 인증 하드웨어용) 30, **NAI(North Atlantic Industries)** 7, **Abaco Systems**(구 GE Fanuc) 9와 같은 하드웨어 제조업체, 그리고 **Audi**와 같은 자동차 업계의 주자들 32이 있습니다.

이 생태계는 고위험 산업 분야에서의 깊고 장기적인 파트너십을 특징으로 합니다. 기업들은 TTEthernet을 단순한 부품으로 선택하는 것이 아니라, 가장 중요한 시스템을 위한 기본 플랫폼으로 선택하며, 종종 TTTech의 공동 개발 및 광범위한 통합 지원을 받습니다.27 TTEthernet 생태계는 '대량 생산' 생태계라기보다는 '고신뢰' 생태계입니다. 그 성공은 실패가 용납되지 않는 프로그램(Orion, Airbus, Boeing)에서 그 가치를 입증하고, 그 명성을 활용하여 다른 중요한 시장에 진입하는 데 기반을 두고 있습니다. 이는 칩 제조업체, 산업 자동화 회사, 자동차 제조업체의 훨씬 더 광범위한 컨소시엄에 의해 주도되어 대중 시장의 상호 운용성을 목표로 하는 TSN 생태계와는 극명한 대조를 이룹니다.

TTEthernet은 설계, 스케줄링 및 검증에 상당한 초기 투자가 필요한 복잡한 기술입니다. 이는 우주선이나 상업용 항공기와 같은 대규모, 장기 수명 주기 프로젝트에 가장 적합하게 만듭니다. 이러한 프로젝트에서는 초기 비용이 수십 년에 걸쳐 상각되고 주요 관심사는 안전과 신뢰성입니다. 이러한 프로젝트에 참여하는 회사들(예: Honeywell, Maxar, RUAG)은 네트워크가 전체 시스템 아키텍처에 필수적이므로 TTTech와 깊은 파트너가 됩니다.29 이는 TTTech에게 선순환을 만듭니다. NASA의 Orion과 같은 유명 프로젝트에서의 성공 8은 다음 유명 프로젝트를 수주하기 위한 주요 마케팅 도구가 되어, 초고신뢰성 결정론적 네트워킹 분야의 선두 주자로서의 입지를 강화합니다.




TTEthernet은 NASA와 ESA에 의해 인간의 심우주 임무를 위해 설계된 오리온 우주선의 핵심 데이터 백본으로 선정되었습니다. 이는 우주선의 '신경계'로 묘사됩니다.8 약 50개의 종단점을 연결하고, 이전 우주왕복선 시스템보다 1,000배 빠른 속도로 데이터를 전송합니다.8 이 네트워크는 유도, 항법 및 제어(GNC)와 생명 유지 시스템과 같은 가장 중요한 기능(TT 트래픽으로)을 우선순위가 낮은 원격 측정 및 탑재체 데이터(RC 및 BE 트래픽으로)와 단일의 고장 감내 삼중 이중화 네트워크에 통합합니다. 이 통합 접근 방식은 과거의 연합형 아키텍처에 비해 질량, 전력 및 비용을 크게 절감합니다.6

오리온에 TTEthernet이 선정된 것은 이 기술에 대한 가장 중요한 단일 검증 사례입니다. 유인 우주선의 통신 네트워크는 고장 시에도 운영 가능함이 입증되어야 합니다. TTEthernet의 견고한 비잔틴 고장 감내 동기화 및 분할 기능은 이러한 타의 추종을 불허하는 안전 및 신뢰성 요구사항을 충족하는 데 필수적이었습니다. 화성이나 달로 가는 임무는 비행 중에 수리할 수 없으므로, 시스템은 여러 고장을 견디고 계속 작동할 수 있어야 합니다. 항공전자 네트워크는 중요한 시스템들의 통합자 역할을 하므로 네트워크 장애는 우주선과 승무원의 손실로 이어질 수 있습니다. 따라서 NASA와 ESA는 가능한 최고 수준의 인증된 신뢰성을 갖춘 네트워크 기술을 요구했습니다. TTEthernet의 AS6802 표준은 고장 감내 시간 및 분할을 위한 형식적으로 검증 가능한 알고리즘을 통해 이러한 요구사항을 충족했습니다.3 오리온에 선정되고 이후 심우주를 위한 "국제 항공전자 시스템 상호운용성 표준(IASIS)"의 핵심 표준으로 자리 잡음으로써 8, 이 분야에서 황금 표준으로서의 지위를 확고히 했습니다.


TTEthernet은 현대 상업용 항공기의 다양한 중요 시스템에 사용됩니다. 예를 들어, Boeing 787 Dreamliner의 전력 관리 및 압력 제어 시스템과 Airbus A380의 기체 압력 제어 시스템이 있습니다.6 또한 F-16 전투기의 GE F110 엔진 제어 시스템에도 사용됩니다.6 이러한 애플리케이션에서 TTEthernet은 분산 제어 기능을 위한 신뢰성 있고 결정론적이며 무게를 절감하는 통신 백본을 제공합니다. 이를 통해 여러 센서와 액추에이터를 일관된 시스템으로 통합할 수 있으며, 이 시스템은 DO-254/178B 레벨 A와 같은 인증 표준을 충족하기 위해 입증된 신뢰성으로 작동해야 합니다.3

항공기에서 유일한 네트워크는 아니지만(AFDX도 광범위하게 사용됨), 이러한 특정 고도의 중요 하위 시스템에서 TTEthernet을 사용하는 것은 표준 AFDX가 제공하는 것 이상의 결정성과 고장 감내가 필요한 기능에 대한 가치를 입증합니다. 이는 종종 새롭고 복잡한 분산 제어 애플리케이션을 위해 선택됩니다. 객실 압력 제어나 복잡한 전력 분배와 같은 기능은 안전에 매우 중요합니다. 이러한 시스템이 더 복잡해지고 분산됨에 따라, 레거시 버스보다 더 유능한 네트워크가 필요합니다. TTEthernet은 이더넷의 높은 대역폭과 함께 필요한 결정성과 고장 감내 기능을 제공하는 COTS 기반 솔루션을 제공합니다.6 이러한 인증된 항공기 플랫폼에서의 채택은 그 성숙도와 신뢰성에 대한 추가적인 증거를 제공하며, 다른 항공우주 프로그램에서의 채택에 긍정적인 피드백 루프를 만듭니다.



아우디는 TTTech와의 파트너십을 통해 첨단 운전자 보조 시스템(ADAS)을 위한 업계 최초의 중앙 집중식 도메인 컨트롤러 중 하나인 zFAS(zentrales Fahrerassistenzsteuergerät)를 개척했습니다. 이 플랫폼은 아우디 A8과 같은 차량의 자율 주행 기능을 위한 '두뇌' 역할을 합니다.32 zFAS 플랫폼은 TTEthernet 기술 기반의 결정론적 이더넷 백본을 사용하여 NVIDIA, Mobileye의 고성능 프로세서와 Infineon Aurix 안전 마이크로컨트롤러를 연결합니다. 이 네트워크는 센서 융합을 위해 다양한 센서(카메라, 레이더, 라이다)로부터의 방대한 데이터 흐름을 처리하며, 단일 ECU에서 다양한 안전 수준(ISO 26262 ASIL D까지)을 가진 30개 이상의 다양한 애플리케이션을 안전하게 실행할 수 있게 합니다. 네트워크의 분할 기능은 덜 중요한 애플리케이션의 버그가 안전 최우선 의사 결정 기능에 영향을 미치지 않도록 하는 데 매우 중요합니다.25 TTTech의 미들웨어인 MotionWise는 이 플랫폼의 핵심 구성 요소입니다.34

zFAS는 수십 개의 개별 ECU에서 강력한 중앙 집중식 도메인 컨트롤러로 이동하는 자동차 E/E 아키텍처의 패러다임 전환을 나타냅니다. TTEthernet은 이러한 복잡하고 혼합된 중요도 시스템이 작동하는 데 필요한 안전하고, 고대역폭이며, 결정론적인 통신을 제공함으로써 이러한 전환을 가능하게 한 핵심 기술이었습니다. 이는 항공우주 등급의 안전 개념이 자동차 세계에 성공적으로 적용될 수 있음을 증명했습니다. 자율 주행은 환경 모델을 구축하기 위해 많은 센서의 데이터를 실시간으로 융합해야 하며, 이는 엄청난 데이터 및 처리 과제입니다.34 이 융합과 후속 의사 결정은 최고 수준(ASIL D)의 안전이 요구되는 작업입니다. 중앙 집중식 아키텍처는 더 강력하고 효율적이지만, 올바르게 설계되지 않으면 단일 장애점이 될 수 있습니다. 내부 통신 네트워크는 이 아키텍처의 백본입니다. 아우디와 TTTech는 TTEthernet의 핵심 원칙-중요 데이터에 대한 보장된 지연 시간, 애플리케이션 간의 견고한 분할, 고장 감내-이 안전한 도메인 컨트롤러의 요구사항과 완벽하게 일치했기 때문에 TTEthernet을 선택했습니다. 이는 zFAS '두뇌'를 위한 신뢰할 수 있는 '신경계'를 제공했습니다.32



세계 최고의 풍력 터빈 제조업체 중 하나인 Vestas는 결정론적 이더넷을 기반으로 하는 TTTech Industrial의 확장 가능한 분산 제어 시스템(DCS)을 사용합니다. 이 시스템은 세계 최초의 15 MW 터빈을 포함하여 10,000기 이상의 터빈에 통합되었습니다.36 DCS는 안전 및 비안전 기능을 단일 하드웨어 플랫폼과 네트워크에 통합합니다. 이는 안전과 제어 기능이 물리적으로 분리되었던 이전 시스템의 복잡한 배선을 단순화합니다. 결정론적 이더넷을 사용하면 중요하지 않은 모니터링 및 진단 데이터와 동일한 네트워크를 통해 안전 최우선 데이터(예: 강풍 시 블레이드 피치 제어)를 안정적으로 전송할 수 있습니다. 이로 인해 새로운 터빈 모델에서 제어 아키텍처의 70% 이상을 재사용할 수 있게 되었고, 유지보수가 감소했으며, 터빈 가동 시간이 증가했습니다.36

Vestas 사례 연구는 TTEthernet의 통합된 혼합 중요도 아키텍처의 이점이 운송 분야에만 국한되지 않음을 보여줍니다. 풍력 발전소와 같은 대규모 산업 시스템에서 결정성과 신뢰성은 높은 효율성, 더 큰 가동 시간, 낮은 유지보수 비용이라는 경제적 이익으로 직접 이어집니다. 솔루션의 확장성 또한 핵심 요소입니다. 풍력 터빈은 혹독한 환경에서 안정적이고 안전하게 작동해야 하는 복잡한 제어 시스템입니다. 전통적인 제어 시스템은 안전과 제어를 위해 별도의 네트워크를 사용하여 높은 복잡성과 배선 비용을 초래했습니다.36 Vestas는 비용을 절감하고 더 크고 새로운 터빈 개발을 가속화하기 위해 더 유연하고 통합되며 확장 가능한 플랫폼으로 전환하고자 했습니다. 결정론적 이더넷(TTEthernet)을 사용하는 TTTech Industrial의 DCS가 그 해결책을 제공했습니다. 이는 다양한 기능을 하나의 네트워크에 안전하게 통합하여 복잡성을 줄이고 아키텍처 재사용을 가능하게 했습니다. 결정성은 블레이드를 페더링하는 것과 같은 중요한 제어 조치가 필요할 때 정확하게 발생하도록 보장하여 안전과 에너지 생산량을 모두 극대화합니다.36 이는 운영 신뢰성이 가장 중요한 산업 애플리케이션에서도 TTEthernet의 가치 제안을 보여줍니다.



TSN은 이더넷에 결정론적 기능을 추가하기 위해 설계된 IEEE 802.1 표준의 집합입니다. TTEthernet은 SAE의 단일 종합 표준입니다. 둘 다 결정성을 목표로 하지만, 서로 다른 메커니즘을 통해 이를 달성합니다.40

- **시간 동기화:** TTE는 견고하고 분산된 비잔틴 고장 감내 AS6802 프로토콜(SM/CM/SC)을 사용합니다.19 TSN은 단일 그랜드마스터 클럭을 선출하는 마스터-슬레이브 IEEE 1588 PTP 프로토콜의 프로파일인 IEEE 802.1AS를 사용합니다.2

- **스케줄링:** TTE는 엄격한 오프라인 TDMA 스케줄을 사용하여 개별 프레임을 스케줄링합니다.40 TSN의 주요 스케줄링 메커니즘인 시간 인식 셰이퍼(IEEE 802.1Qbv)는 스케줄에 따라 매체에 대한 게이트를 열고 닫음으로써 

  *큐* 또는 트래픽 클래스를 스케줄링합니다. 이는 더 유연하지만 잠재적으로 덜 세분화될 수 있습니다.23

- **고장 감내:** 고장 감내는 TTEthernet 설계의 근본적이고 통합된 부분입니다.13 TSN에서 고장 감내는 선택적인 추가 표준(예: 신뢰성을 위한 프레임 복제 및 제거를 위한 IEEE 802.1CB)입니다.40

- **표준화 및 생태계:** TTE는 SAE의 단일, 하향식 표준으로, 보다 통제되지만 고도로 통합된 생태계를 이끌어냅니다.18 TSN은 많은 개별 IEEE 표준의 "도구 상자"로, 보다 개방적이고 유연하며 광범위한 생태계를 이끌지만 잠재적으로 더 복잡한 통합을 요구합니다.40

TTEthernet과 TSN은 두 가지 다른 철학을 대표합니다. TTEthernet은 최고의 안전성과 검증 가능성을 위해 설계된 "고장 시 운영 가능한 시스템" 접근 방식으로, 절대적인 예측 가능성을 위해 유연성을 희생합니다. TSN은 유연성, 상호 운용성 및 융합을 위해 설계된 "실시간 네트워킹" 접근 방식으로, 산업 자동화에서 자동차 및 프로 오디오/비디오에 이르기까지 훨씬 더 넓은 범위의 애플리케이션을 지원하는 것을 목표로 합니다. TTEthernet의 설계 선택(비잔틴 동기화, 프레임별 스케줄링)은 고장 비용이 무한대인 시스템(예: 유인 우주선)에 최적입니다. 복잡성과 경직성은 수용 가능한 트레이드오프입니다. TSN의 설계 선택(PTP 동기화, 큐 기반 스케줄링, 모듈식 표준)은 상호 운용성, 비용 및 유연성이 핵심 동인인 시장(예: 다중 공급업체 공장 현장 또는 자동차의 인포테인먼트 및 ADAS 도메인)에 최적입니다. 이는 이들이 순전히 경쟁 관계에 있지 않다는 결론으로 이어집니다. TTEthernet은 전문화된 고신뢰성 솔루션인 반면, TSN은 범용적이고 유연한 솔루션입니다. TTEthernet의 강점은 타의 추종을 불허하는 검증 가능한 견고성이고, TSN의 강점은 개방적이고 적응 가능하며 광범위한 시장 매력입니다.

| 기능            | SAE AS6802 (TTEthernet)                                      | IEEE 802.1 (TSN)                                             | 시스템 설계자에 대한 시사점                                  |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **표준화**      | SAE의 단일 종합 표준 (AS6802).                               | IEEE의 여러 모듈식 표준 "도구 상자" (802.1Qbv, AS 등).       | TTE: 완전하고 통합된 솔루션. TSN: 더 유연하지만, 올바른 표준 집합을 선택하고 통합해야 함. |
| **핵심 철학**   | 안전 우선, 고장 시 운영 가능한 시스템 설계. 예측 가능성과 검증 가능성 우선. | 유연성 우선, 실시간 네트워킹. 융합과 상호 운용성 우선.       | TTE: 최고 등급(예: DAL A) 시스템에 이상적. TSN: 광범위한 요구사항을 가진 혼합 중요도 시스템에 이상적. |
| **시간 동기화** | 분산형, 비잔틴 고장 감내 프로토콜 (SM/CM/SC). 단일 마스터 없음. | 단일 그랜드마스터 클럭을 사용하는 마스터-슬레이브 프로토콜 (IEEE 802.1AS/gPTP). | TTE: 클럭 고장에 대한 본질적인 견고성이 더 높음. TSN: 개념은 더 간단하지만, 그랜드마스터가 중요한 요소. |
| **스케줄링**    | 개별 프레임의 오프라인, 정적 스케줄링 (TDMA).                | 게이트 제어 목록(GCL)이 큐/트래픽 클래스를 스케줄링 (예: 802.1Qbv). 더 동적임. | TTE: 절대적인 결정성을 보장하지만 경직됨. TSN: 더 유연하고 효율적인 대역폭 사용, 그러나 분석이 더 복잡함. |
| **고장 감내**   | 처음부터 통합 설계됨 (동기화 프로토콜, 버스 가디언).         | 별도 표준을 통한 추가 기능 (예: 이중화를 위한 IEEE 802.1CB). | TTE: 고장 감내가 내재되어 있음. TSN: 특정 구성 요소를 사용하여 이중화를 명시적으로 설계해야 함. |
| **생태계**      | TTTech 및 주요 항공우주/자동차 파트너가 주도하는 통제된 생태계. | 산업, 자동차, 프로 AV 전반에 걸친 광범위하고 개방적인 다중 공급업체 생태계. | TTE: 높은 수준의 통합 및 지원, 그러나 공급업체 수가 적음. TSN: 다양한 공급업체와 구성 요소 선택 가능, 저렴한 비용. |


TTEthernet의 속도 제한(RC) 트래픽 클래스는 기능적으로 AFDX와 동일하며 호환됩니다.7 TTEthernet은 ARINC 664p7을 준수합니다.2 이 섹션에서는 이것이 '대결' 비교가 아니라 '상위 집합' 관계임을 명확히 할 것입니다. TTEthernet은 AFDX의 모든 기능(가상 링크, 대역폭 할당 간격)을 포함하고, 그 위에 더 높은 우선순위의 완전 결정론적인 시간 트리거(TT) 클래스를 추가합니다.

TTEthernet은 AFDX의 다음 논리적 진화로 볼 수 있으며, 항공우주 시스템이 기존의 방대한 AFDX 시스템 기반과의 하위 호환성을 유지하면서 새롭고 더 까다로운 애플리케이션을 위해 훨씬 더 높은 수준의 결정성을 달성할 수 있는 경로를 제공합니다. AFDX는 좋은 수준의 제한된 지연 시간을 제공하지만, 진정한 시간 트리거 시스템의 고정된 지연 시간과 마이크로초 수준의 지터는 제공하지 못합니다. 플라이 바이 와이어와 같은 기능이 이더넷으로 이동함에 따라 더 높은 수준의 결정성이 요구되었습니다. TTEthernet은 TT 클래스로 이를 제공했습니다. 또한 AFDX 호환 RC 클래스를 포함함으로써 설계자들은 "오래된" AFDX 스타일 애플리케이션과 "새로운" TT 스타일 애플리케이션 모두에 하나의 네트워크를 사용하여 아키텍처를 단순화할 수 있었습니다.11


EtherCAT(Ethernet for Control Automation Technology)은 또 다른 인기 있는 산업용 이더넷 프로토콜이지만, TTEthernet과는 근본적으로 다른 작동 원리를 사용합니다.43

- **아키텍처:** TTEthernet은 스타 또는 복잡한 토폴로지를 가진 스위치드 네트워크로, 대규모 혼합 중요도 시스템의 백본 통신을 위해 설계되었습니다.8 EtherCAT은 논리적 링 토폴로지(물리적 토폴로지는 라인, 트리 또는 스타가 될 수 있음)를 가진 마스터-슬레이브 아키텍처를 사용하며, 기계 수준에서 고속 I/O 및 모션 제어를 위해 설계되었습니다.47
- **통신 모델:** TTEthernet 스위치는 스케줄이나 우선순위에 따라 전체 프레임을 저장하고 전달합니다. EtherCAT은 마스터가 보낸 단일 프레임이 모든 슬레이브를 통과하는 독특한 "실시간 처리(processing on the fly)" 방식을 사용합니다. 각 슬레이브는 프레임이 통과할 때 자신의 데이터를 읽고 자신의 데이터를 프레임에 삽입하며, 이 모든 것이 하드웨어에서 이루어집니다. 이는 매우 빠르며, 100 마이크로초 미만의 사이클 타임을 가집니다.47
- **하드웨어:** TTEthernet은 특수 스위치와 종단 시스템이 필요합니다. EtherCAT은 표준 이더넷 하드웨어에 마스터가 필요하지만, 모든 슬레이브 장치에는 전용 EtherCAT 슬레이브 컨트롤러(ASIC)가 필요합니다.50

TTEthernet과 EtherCAT은 직접적인 경쟁자가 아니며, 자동화 피라미드의 다른 계층을 위해 설계되었습니다. TTEthernet은 복잡한 컨트롤러를 통합하기 위한 시스템 수준 또는 백본 네트워크입니다. EtherCAT은 극히 낮은 지연 시간으로 센서, 액추에이터 및 드라이브를 연결하기 위한 필드 수준 또는 기계 수준 네트워크입니다. 예를 들어, 복잡한 자동화 공장을 생각해 봅시다. 여러 기계 셀과 컨트롤러를 연결하는 전체 공장 제어 및 안전 시스템은 견고하고 혼합된 중요도 통신을 보장하기 위해 TTEthernet 백본에서 실행될 수 있습니다. 단일 로봇 조립 셀 내에서 로봇 컨트롤러(마스터)는 정밀한 모션 제어에 필요한 마이크로초 수준의 동기화를 달성하기 위해 EtherCAT을 사용하여 관절용 서보 드라이브(슬레이브)와 통신할 것입니다. 따라서 시스템은 감독 수준에는 TTEthernet을, 필드 수준에는 EtherCAT을 모두 사용할 수 있습니다. 이들은 서로 다른 규모에서 다른 문제를 해결하는 보완적인 기술입니다.

| 기능                   | SAE AS6802 (TTEthernet)                                    | EtherCAT                                         | 시스템 설계자에 대한 시사점                                  |
| ---------------------- | ---------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| **주요 사용 사례**     | 혼합 중요도 시스템 통합 백본.                              | 고속, 동기화된 모션 제어 및 I/O.                 | TTE: 컨트롤러와 하위 시스템 연결용. EtherCAT: 컨트롤러를 드라이브 및 센서에 연결용. |
| **네트워크 아키텍처**  | 스위치드 이더넷 (스타/메시 토폴로지).                      | 논리적 링의 마스터-슬레이브.                     | TTE: 유연한 토폴로지, 대규모 분산 시스템에 적합. EtherCAT: 기계 수준의 데이지 체인에 최적화됨. |
| **통신 원리**          | 전역 스케줄 기반의 저장 후 전달(Store-and-forward) 스위칭. | 슬레이브의 "실시간 처리(Processing on the fly)". | TTE: 대규모 네트워크 전반에 걸쳐 예측 가능한 종단 간 지연 시간. EtherCAT: 단일 기계 세그먼트 내에서 극히 낮은 지연 시간. |
| **결정성**             | 오프라인 TDMA 스케줄과 고장 감내 동기화로 보장.            | 마스터 주도의 단일 프레임 처리로 보장.           | TTE: 시스템 전체의 시간적 분할에 중점. EtherCAT: 마이크로초 수준의 장치 동기화에 중점. |
| **일반적인 적용 분야** | 항공전자/자동차 백본, 대규모 산업 제어 시스템.             | CNC 기계, 로봇 공학, 포장 기계, 사출 성형.       | 이들은 대체로 경쟁 관계가 아닌 보완 관계임.                  |



TTEthernet의 지속적인 가치는 안전과 결정성에 대한 타협 없는 접근 방식에 있습니다. 이는 비잔틴 고장 감내 클럭 동기화, 정적 TDMA 스케줄링, 그리고 견고한 분할이라는 세 가지 기둥을 기반으로, 입증 가능하게 견고하고, 고장 시 운영 가능하며, 시간적으로 예측 가능한 통신 플랫폼을 제공합니다. 이는 유인 우주 비행 및 인증된 상업용 항공기에서의 채택으로 입증되었듯이, 고장 비용이 용납할 수 없을 정도로 높은 애플리케이션을 위한 황금 표준으로 자리매김하게 합니다.6


업계는 개방성, 유연성 및 광범위한 공급업체 지원으로 인해 특히 자동차 및 산업 자동화 분야에서 TSN으로의 강력한 추세를 보이고 있습니다.44 그러나 TTEthernet의 원칙은 802.1Qbv와 같은 TSN 표준에 큰 영향을 미쳤습니다.23 미래는 "승자 독식" 시나리오가 될 가능성은 낮습니다. TSN이 광범위한 시간 민감형 애플리케이션을 위한 지배적인 표준이 될 것으로 예상되지만, TTEthernet은 TSN의 보다 유연한 접근 방식이 쉽게 충족할 수 없는 특정하고 검증 가능한 고장 감내 모델이 필수적인 초고신뢰성 분야에서 그 틈새 시장을 유지할 가능성이 높습니다.

가장 가능성 있는 미래는 **공존과 하이브리드화**의 하나입니다. 우리는 미래의 항공기나 자율주행차와 같은 복잡한 시스템이 이기종 네트워크 아키텍처를 사용하는 것을 보게 될 것입니다. 매우 견고한 TTEthernet 또는 TTEthernet과 유사한 백본이 절대적인 안전 최우선 핵심(예: 차량 동역학 제어)에 사용될 수 있으며, 이는 센서 처리, 인포테인먼트 또는 텔레매틱스와 같은 다른 기능을 위한 여러 TSN 하위 도메인으로 게이트웨이 역할을 할 것입니다. TSN이 더 발전된 신뢰성 기능을 계속 채택함에 따라 그들 사이의 경계는 흐려질 수 있습니다. TSN은 IEEE의 지원 덕분에 광범위한 시장 채택 경쟁에서 이기고 있으며, 이는 경쟁력 있고 저렴하며 다중 공급업체 생태계를 조성합니다.46 그러나 TSN 표준의 "도구 상자"를 기반으로 한 복잡한 시스템을 최고 안전 수준(예: DAL A)으로 인증하는 것은 단일의, 통합적이며 형식적으로 검증된 AS6802 표준을 기반으로 한 시스템을 인증하는 것보다 더 복잡할 수 있습니다. 따라서 가장 위험을 회피하는 산업 및 애플리케이션의 경우, TTEthernet은 더 비싸거나 덜 유연하더라도 설득력 있고 위험이 낮은 선택으로 남을 것입니다. 이는 계층화된 미래로 이어집니다. 안전 최우선 핵심의 "요새"를 위한 TTEthernet과, 다른 실시간 기능의 주변 "도시"를 위한 TSN, 그리고 그들 사이의 안전한 게이트웨이가 있는 미래입니다.


결정론적 네트워킹 기술의 선택은 "어느 것이 더 나은가"의 단순한 문제가 아니라 "애플리케이션의 특정 요구사항에 어느 것이 적합한가"의 문제입니다.

- **SAE AS6802 (TTEthernet)를 선택해야 할 경우:** 주요 시스템 동인이 최고 무결성 수준(예: 항공우주 DAL A, 자동차 ASIL D)의 고장 시 운영 가능한 안전성일 때. 시스템 아키텍처가 비교적 정적이고, 형식적 검증 및 인증 비용이 런타임 유연성의 필요성을 능가할 때. 단일 공급업체의 고도로 통합된 솔루션이 수용 가능하거나 선호될 때.
- **Time-Sensitive Networking (TSN)을 선택해야 할 경우:** 주요 시스템 동인이 유연성, 상호 운용성, 비용 및 여러 실시간 및 비실시간 트래픽 유형의 융합일 때. 시스템이 더 동적인 구성을 요구하고, 광범위한 다중 공급업체 생태계가 바람직할 때. 안전 요구사항이 높을 수 있지만, TTEthernet에 내재된 특정 비잔틴 고장 감내 모델을 요구하지 않을 때.

연구자로서 TTEthernet에서 TSN으로의 진화는 기술 확산의 매력적인 사례 연구입니다. TTEthernet이 개척한 엄격하고 하향식이며 안전 중심적인 원칙은 검증되었으며, 이제 TSN의 보다 유연하고 시장 중심적이며 상향식 접근 방식에 의해 채택되고 대중화되고 있습니다. 내일의 중요한 시스템을 설계하는 모든 설계자에게 이 두 가지를 모두 이해하는 것은 필수적입니다.


1. time-triggered ethernet (TTE) #networkengineering #ethernet - YouTube, accessed July 5, 2025, https://www.youtube.com/watch?v=LJmMezD0s24
2. On TTEthernet for Integrated Fault- Tolerant Spacecraft Networks, accessed July 5, 2025, https://ntrs.nasa.gov/api/citations/20170009923/downloads/20170009923.pdf
3. Deterministic Ethernet - SAE AS6802 (TTEthernet) | PDF - Scribd, accessed July 5, 2025, https://www.scribd.com/document/73788160/Deterministic-Ethernet-SAE-AS6802-TTEthernet
4. A time-predictable open-source TTEthernet end-system - DTU Inside, accessed July 5, 2025, https://backend.orbit.dtu.dk/ws/files/209339901/1_s2.0_S1383762120300382_main.pdf
5. Time-Triggered Ethernet Metamodel: Design and Application - Journal of Software, accessed July 5, 2025, https://www.jsoftware.us/vol11/202-JSW15135.pdf
6. Ethernet for Aerospace Applications - NASA Technical Reports ..., accessed July 5, 2025, https://ntrs.nasa.gov/api/citations/20150011061/downloads/20150011061.pdf
7. Time-Triggered Ethernet / ARINC 664 Part 7 (AFDX®) / IEEE 802.3 Ethernet Deterministic Communications, accessed July 5, 2025, [https://www.tptech.co.jp/documents/uploads/TE2%20Data%20Sheet.pdf](https://www.tptech.co.jp/documents/uploads/TE2 Data Sheet.pdf)
8. TTEthernet (Time-Triggered Ethernet) - eoPortal, accessed July 5, 2025, https://www.eoportal.org/other-space-activities/ttethernet-1
9. GE Fanuc Intelligent Platforms Announces TTEthernet For Safety-Critical Avionics Applications | GE News, accessed July 5, 2025, https://www.ge.com/news/press-releases/ge-fanuc-intelligent-platforms-announces-ttethernet-safety-critical-avionics
10. Time-Triggered Ethernet - ResearchGate, accessed July 5, 2025, https://www.researchgate.net/profile/Michael-Paulitsch/publication/330310738_Time-triggered_ethernet/links/616025ede7993f536ca439ba/Time-triggered-ethernet.pdf
11. August 2009 - TTEthernet – A Powerful Network Solution for Advanced Integrated Systems, accessed July 5, 2025, https://www.abaco.com/news/august-2009-ttethernet-a-powerful-network-solution-for-advanced-integrated-systems/n2614
12. Deterministic Ethernet SAE AS6802 | PPT - SlideShare, accessed July 5, 2025, https://www.slideshare.net/mja70/deterministic-ethernet-sae-as6802
13. TTEthernet - Wikipedia, accessed July 5, 2025, https://en.wikipedia.org/wiki/TTEthernet
14. Time-Triggered Ethernet – A Powerful Network Solution for Multiple Purpose - TTTECH, accessed July 5, 2025, https://www.tttech.com/sites/default/files/documents/TTEthernet_Technical-Whitepaper_TTTech.pdf
15. On TTEthernet for Integrated Fault-Tolerant Spacecraft Networks - NASA Technical Reports Server (NTRS), accessed July 5, 2025, https://ntrs.nasa.gov/api/citations/20150014489/downloads/20150014489.pdf
16. Time-Triggered Ethernet AS6802 - SAE International, accessed July 5, 2025, https://www.sae.org/standards/content/as6802/
17. A Time-Triggered Ethernet (TTE) Switch, accessed July 5, 2025, http://www.ann.ece.ufl.edu/courses/eel6686_15spr/papers/tte_switch.pdf
18. SAE International releases deterministic Ethernet standard - TTTech Computertechnik AG, accessed July 5, 2025, https://www.tttech.com/press/sae-international-releases-deterministic-ethernet-standard-sae-as6802
19. Integrated Formal Analysis of Timed-Triggered Ethernet - NASA Technical Reports Server (NTRS), accessed July 5, 2025, https://ntrs.nasa.gov/api/citations/20120003666/downloads/20120003666.pdf
20. Synthesis of communication schedules for TTEthernet-based mixed-criticality systems | Request PDF - ResearchGate, accessed July 5, 2025, https://www.researchgate.net/publication/262233805_Synthesis_of_communication_schedules_for_TTEthernet-based_mixed-criticality_systems
21. Incremental Scheduling of the Time-triggered Traffic on TTEthernet Network - SciTePress, accessed July 5, 2025, https://www.scitepress.org/Papers/2022/109537/109537.pdf
22. TTEthernetTutorial - VisualSimDocs, accessed July 5, 2025, https://www.mirabilisdesign.com/doc_files/doc/CodeDoc/WebHelp/VisualSimDocs/TTEthernetTutorial.html
23. Impact Analysis of Flow Shaping in Ethernet-AVB/TSN and AFDX ..., accessed July 5, 2025, https://www.mdpi.com/1424-8220/17/5/1181
24. Comparison of ieee AVB and AFDX - ResearchGate, accessed July 5, 2025, https://www.researchgate.net/publication/261208790_Comparison_of_ieee_AVB_and_AFDX
25. Time-Triggered Ethernet - TTTECH, accessed July 5, 2025, https://www.tttech.com/know-how/time-triggered-ethernet
26. TTE Controller Development Status, accessed July 5, 2025, http://www.sephy.eu/mt-content/uploads/2018/06/2018-dasia_tttech_tte-controller-development-status_v1.2.pdf
27. TTTech Aerospace: certifiable solutions for safety-critical aviation systems - FlightGlobal, accessed July 5, 2025, https://www.flightglobal.com/paid-content/tttech-aerospace-certifiable-solutions-for-safety-critical-aviation-systems/153549.article
28. Audi Selects Altera SoC FPGA for Production Vehicles with 'Piloted Driving' Capability | Intel Newsroom, accessed July 5, 2025, https://download.intel.com/newsroom/2021/archive/2015-01-05-news-releases-audi-selects-altera-soc-fpga-production-vehicles-piloted-driving-capability.pdf
29. TTTech Aerospace's mature TTEthernet® network solutions enable Honeywell Anthem's™ system architecture, accessed July 5, 2025, https://www.tttech.com/press/tttech-aerospace-honeywell-anthem-flight-deck
30. TTTech Aerospace and RUAG Space selected by Maxar to supply TTEthernet network platform for NASA's Gateway, accessed July 5, 2025, https://www.spacefoundation.org/2021/03/04/tttech-aerospace-and-ruag-space-selected-by-maxar-to-supply-ttethernet-network-platform-for-nasas-gateway/
31. TTTech Aerospace, in partnership with Réaltra, releases its new, compact Ethernet switch for space transportation, accessed July 5, 2025, https://www.tttech.com/compact-deterministic-ethernet-switch-for-space-transportation
32. Multi-disciplinary partnership pushes the envelope for autonomous cars - AI Online -, accessed July 5, 2025, https://www.ai-online.com/2015/01/multi-disciplinary-partnership-pushes-the-envelope-for-autonomous-cars/
33. Scalable ECU Platform for Autonomous Driving is Going into Series Production and is Now Available Worldwide - TTTech, accessed July 5, 2025, https://www.tttech.com/scalable-ecu-platform-for-autonomous-driving-is-going-into-series-production-and-is-now-available-worldwide
34. Why Audi's zFAS is a blueprint for next-gen domain architectures - eeNews Europe, accessed July 5, 2025, https://www.eenewseurope.com/en/why-audis-zfas-is-a-blueprint-for-next-gen-domain-architectures/
35. Cutting-edge technology solutions ensuring safety and electronic robustness for a more connected, automated and sustainable world. - EFFRA, accessed July 5, 2025, https://www.effra.eu/wp-content/uploads/2025/04/Francesca-Flmigni_-TTTech.pdf
36. Vestas - TTTech Industrial, accessed July 5, 2025, https://www.tttech-industrial.com/resources/case-studies/vestas
37. Industrial Automation in the Wind Energy Sector, accessed July 5, 2025, https://www.tttech-industrial.com/resource-library/blog-posts/industrial-automation-in-the-wind-energy-sector
38. Successful partnership in the wind energy sector - TTTech, accessed July 5, 2025, https://www.tttech.com/successful-partnership-in-the-wind-energy-sector
39. Shaping the future of open, autonomous industry, accessed July 5, 2025, https://iebmedia.com/buyers-guide/shaping-the-future-of-open-autonomous-industry/
40. TSN vs. TTE: Choosing the Right Ethernet Networking Standard for Critical Missions, accessed July 5, 2025, https://www.dornerworks.com/blog/tsn-vs-tte/
41. Comparison of Time Sensitive Networking (TSN) and TTEthernet | PDF - Scribd, accessed July 5, 2025, https://www.scribd.com/document/691191734/Comparison-of-Time-Sensitive-Networking-TSN-and-TTEthernet
42. An Overview of Scheduling Mechanisms for Time-sensitive Networks - Computer Science, accessed July 5, 2025, https://www.cs.uni-salzburg.at/~scraciunas/pdf/techreports/craciunas_ETR17.pdf
43. Comparison of Time Sensitive Networking (TSN) and TTEthernet | Request PDF, accessed July 5, 2025, https://www.researchgate.net/publication/329561589_Comparison_of_Time_Sensitive_Networking_TSN_and_TTEthernet
44. Time sensitive networking: Revolutionizing industrial automation and automotive systems - | World Journal of Advanced Research and Reviews, accessed July 5, 2025, https://journalwjarr.com/sites/default/files/fulltext_pdf/WJARR-2025-1594.pdf
45. TSN: an Ethernet network, but better for critical embedded systems - IRT Saint Exupéry, accessed July 5, 2025, https://www.irt-saintexupery.com/tsn-an-ethernet-network-but-better-for-critical-embedded-systems/
46. Time-Sensitive Networking (TSN) for Industrial Automation: Current Advances and Future Directions - arXiv, accessed July 5, 2025, https://arxiv.org/pdf/2306.03691
47. Ethernet vs. EtherCAT - Astrodyne TDI, accessed July 5, 2025, https://www.astrodynetdi.com/blog/ethernet-vs.-ethercat
48. Exploring use of ethernet for in-vehicle control applications: AFDX, TTEthernet, EtherCAT, and AVB | Request PDF - ResearchGate, accessed July 5, 2025, https://www.researchgate.net/publication/277638810_Exploring_use_of_ethernet_for_in-vehicle_control_applications_AFDX_TTEthernet_EtherCAT_and_AVB
49. Ethernet vs EtherCAT: Key Differences Explained for Engineers! #instrumentation - YouTube, accessed July 5, 2025, https://www.youtube.com/watch?v=pxFNc5WiokM
50. EtherCAT vs. EtherNet/IP - Real Time Automation, Inc., accessed July 5, 2025, https://www.rtautomation.com/rtas-blog/ethercat-vs-ethernet-ip/
51. Design Methodology of Automotive Time-Sensitive Network System Based on OMNeT++ Simulation System - PMC - PubMed Central, accessed July 5, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9229697/
52. Title TSN-based industrial network performance analysis Authors Seliem, Mohamed - UCC: CORA, accessed July 5, 2025, https://cora.ucc.ie/bitstreams/5f7e9640-8af8-4318-a189-52932cad28af/download


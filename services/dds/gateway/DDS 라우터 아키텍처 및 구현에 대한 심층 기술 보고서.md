# 세상을 연결하다: DDS 라우터 아키텍처 및 구현에 대한 심층 기술 보고서





## 1.  데이터 분산 서비스(DDS) 패러다임: 실시간 시스템의 초석





### 1.1  데이터 중심(Data-Centricity)의 철학



현대의 분산 시스템 설계에서 통신 미들웨어는 애플리케이션 간의 원활한 정보 교환을 위한 핵심적인 역할을 수행한다. 전통적인 메시지 중심(Message-Centric) 또는 객체 중심(Object-Centric) 패러다임이 특정 수신자에게 메시지를 보내는 행위 자체에 초점을 맞추는 반면, 데이터 분산 서비스(DDS)는 근본적으로 다른 접근 방식을 채택한다.1 DDS의 핵심 철학은 '데이터 중심(Data-Centricity)'으로, 이는 통신의 주체를 애플리케이션이 아닌 '데이터' 그 자체로 간주하는 패러다임의 전환을 의미한다.

이러한 철학을 구현하는 핵심 개념이 바로 '글로벌 데이터 공간(Global Data Space)'이다.3 이는 물리적으로 분산된 노드들이 공유하는 가상의 거대 데이터베이스와 유사하다. 각 애플리케이션(DDS에서는 'Participant'로 지칭)은 이 공간에 데이터를 '발행(Publish)'하거나 필요한 데이터를 '구독(Subscribe)'함으로써 통신에 참여한다. 개발자는 더 이상 소켓 프로그래밍, 데이터 직렬화, 네트워크 주소 관리와 같은 저수준의 네트워크 프로그래밍에 대해 고심할 필요가 없다.3 DDS 미들웨어가 데이터의 효율적인 배포를 전적으로 책임지기 때문이다. 애플리케이션은 단지 "어떤 데이터를 원하는가" 또는 "어떤 데이터를 제공할 것인가"에만 집중하면 된다.

이러한 추상화는 개발 과정을 극적으로 단순화하지만, 동시에 새로운 과제를 제시한다. '글로벌 데이터 공간'이라는 강력한 추상화는 그 아래에 놓인 복잡한 네트워크의 실체를 감춘다. 개발자는 네트워크 동작을 직접 제어하는 대신 미들웨어의 '발견(Discovery)' 메커니즘에 의존하게 된다.3 이 발견 메커니즘이 원활하게 작동하는 환경에서는 문제가 없지만, 서브넷 경계를 넘어서거나 방화벽이 존재하는 등 네트워크 환경이 복잡해지면 '글로벌 데이터 공간'은 분절되고 추상화는 깨지기 시작한다. 이처럼 추상화로 인해 발생하는 문제는 종종 더 지능적이고 관리되는 계층의 추상화, 즉 '라우터(Router)'를 통해 해결된다. 이 지점에서 DDS 라우터의 필요성이 대두되며, 본 보고서의 핵심 주제로 이어진다.



### 1.2  핵심 아키텍처 구성 요소



DDS의 데이터 중심 아키텍처를 이해하기 위해서는 그를 구성하는 핵심 객체들의 역할과 관계를 명확히 파악해야 한다. 이 구성 요소들은 이후 라우터 설정에서 반복적으로 등장하므로 그 개념을 숙지하는 것이 필수적이다.

- **도메인(Domain)과 도메인 참여자(DomainParticipant):** DDS 통신은 '도메인'이라는 논리적인 파티션 내에서 이루어진다. 서로 다른 도메인에 속한 참여자들은 기본적으로 통신할 수 없다.3 '도메인 참여자'는 애플리케이션이 특정 도메인에 참여하기 위한 진입점(Entry Point) 역할을 하는 객체이다.5 시스템을 기능별로 분리하거나 트래픽을 격리할 목적으로 여러 도메인을 사용할 수 있다.
- **토픽(Topic):** 토픽은 DDS 통신의 가장 중심적인 개념으로, 데이터 그 자체를 식별하는 역할을 한다. 이는 특정 데이터 타입(Data Type)과 고유한 이름(Topic Name), 그리고 서비스 품질(QoS) 정책의 집합으로 정의된다.4 예를 들어, 항공 관제 시스템에서 'AircraftTrack'이라는 토픽은 항공기의 위치, 속도, 고도 등의 정보를 담는 데이터 구조와 연결될 수 있다.
- **발행자(Publisher)와 데이터 작성자(DataWriter):** '발행자'는 하나 이상의 '데이터 작성자'를 관리하는 객체이다. 실제 데이터 샘플을 글로벌 데이터 공간에 쓰는 역할은 '데이터 작성자'가 수행한다.5 데이터 작성자는 특정 토픽에 바인딩되어 해당 토픽의 데이터를 발행한다.
- **구독자(Subscriber)와 데이터 판독기(DataReader):** '구독자'는 하나 이상의 '데이터 판독기'를 관리하며, '데이터 판독기'는 특정 토픽의 데이터를 수신하는 역할을 한다.5 데이터 판독기는 관심 있는 토픽을 구독하고, 해당 토픽으로 새로운 데이터가 발행되면 이를 수신하여 애플리케이션에 전달한다.
- **데이터 타입(Data-Type)과 키(Key):** DDS는 강력한 타입 시스템을 지원한다. 통신에 사용될 데이터 구조는 인터페이스 정의 언어(IDL)와 같은 형식으로 명확하게 정의된다.6 또한, 토픽 내에서 데이터의 특정 인스턴스를 구분하기 위해 '키'를 사용할 수 있다. 예를 들어, 'AircraftTrack' 토픽에서 각 항공기를 고유하게 식별하기 위해 '항공편 번호' 필드를 키로 지정할 수 있다.3 이를 통해 애플리케이션은 특정 키 값을 가진 데이터만 선택적으로 읽거나 갱신할 수 있어 확장성과 데이터 관리 효율성을 높인다.



### 1.3  발행-구독 통신 모델



DDS는 발행-구독(Publish-Subscribe) 통신 모델을 기반으로 한다.4 이 모델의 가장 큰 특징은 발행자와 구독자 간의 '완전한 분리(Decoupling)'이다.

데이터를 생성하는 발행자는 자신의 데이터를 누가, 어디서, 몇 명이나 구독하는지 알 필요가 없다.3 단지 정의된 토픽에 데이터를 발행할 뿐이다. 마찬가지로, 데이터를 소비하는 구독자 역시 데이터가 어디서, 누구에 의해 생성되었는지 알 필요 없이 관심 있는 토픽을 구독하기만 하면 된다. 이들 사이의 연결, 즉 메시지 라우팅은 전적으로 DDS 미들웨어가 자동으로 처리한다.5

이러한 분리(Decoupling)는 시스템의 유연성과 확장성을 극대화한다. 새로운 애플리케이션(참여자)이 시스템에 추가되거나 기존 애플리케이션이 제거되어도, 다른 애플리케이션의 코드를 수정할 필요가 없다. 이는 '플러그 앤 플레이(Plug-and-Play)' 특성을 가능하게 하여, 동적으로 변화하는 대규모 분산 시스템 구축을 용이하게 한다.2



### 1.4  서비스 품질(QoS)의 핵심적 역할



서비스 품질(QoS)은 DDS의 단순한 부가 기능이 아니라, 데이터 전달의 '방법'을 제어하는 핵심적인 표준 기능이다.7 DDS는 20개 이상의 세분화된 QoS 정책을 제공하여 개발자가 애플리케이션의 요구사항에 맞춰 통신의 신뢰성, 실시간성, 자원 사용량 등을 정밀하게 제어할 수 있도록 한다. 라우팅과 관련하여 특히 중요한 몇 가지 QoS 정책은 다음과 같다.

- **신뢰성(RELIABILITY):** 데이터가 반드시 전달되어야 하는지(RELIABLE) 또는 최선 노력(BEST_EFFORT)으로 전달되어도 되는지를 결정한다. 금융 거래나 제어 명령과 같이 데이터 손실이 치명적인 경우 RELIABLE을 사용한다.
- **지속성(DURABILITY):** 구독자가 참여하기 전에 발행된 데이터를 받을 수 있는지 여부를 결정한다. 시스템의 현재 상태 값처럼 늦게 참여한 구독자도 최신 값을 알아야 할 경우 이 정책이 유용하다.
- **이력(HISTORY):** 데이터 작성자 또는 판독기가 얼마나 많은 데이터 샘플을 보관할지를 결정한다.
- **소유권(OWNERSHIP):** 하나의 토픽에 대해 여러 발행자가 존재할 경우, 어떤 발행자가 데이터 발행의 소유권을 가질지에 대한 규칙을 정의한다. 이를 통해 시스템의 이중화(Redundancy)를 구현할 수 있다.

발행-구독 모델에서 통신이 성립되기 위해서는 데이터 작성자와 데이터 판독기가 토픽의 이름과 타입뿐만 아니라, 호환되는 QoS 정책을 가져야만 한다. 예를 들어, 작성자는 RELIABLE로 데이터를 보내는데 판독자가 BEST_EFFORT로 받으려고 하면 둘 사이의 연결은 성립되지 않는다. 이러한 QoS 불일치는 다른 모든 설정이 완벽하더라도 통신 실패의 주된 원인이 된다.8 DDS 라우터는 시스템에 도입될 때 내부적으로 입력단에서는 구독자(DataReader)로, 출력단에서는 발행자(DataWriter)로 동작한다. 따라서 라우터의 QoS 설정은 원본 발행자와 최종 구독자 양쪽 모두와 호환 가능해야 한다. 이는 라우터 설정에서 QoS 관리가 매우 중요하고 복잡한 작업이 되는 이유이며, 5장과 7장에서 심도 있게 다룰 것이다.



### 1.5  RTPS 유선 프로토콜과 상호운용성



DDS는 표준화된 API 명세이며, RTPS(Real-Time Publish-Subscribe)는 DDS 구현체 간의 상호운용성을 보장하기 위한 표준 유선 프로토콜(Wire Protocol)이다.9 즉, RTI, eProsima, OpenSplice 등 서로 다른 벤더가 만든 DDS 구현체들이 RTPS라는 공통의 언어를 사용하기 때문에 서로 데이터를 주고받을 수 있다. DDS 라우터가 서로 다른 네트워크나 도메인을 연결하는 역할을 수행할 수 있는 근본적인 이유도 바로 이 RTPS 프로토콜의 존재 덕분이다. 라우터는 이 표준 프로토콜을 기반으로 데이터를 수신하고 전달함으로써 벤더에 종속되지 않는 유연한 시스템 통합을 가능하게 한다.



## 2.  분산 DDS 시스템의 연결성 문제: 라우터의 필요성



DDS의 데이터 중심 발행-구독 모델은 단일 로컬 네트워크(LAN) 환경에서는 매우 효율적이고 강력한 성능을 발휘한다. 그러나 시스템의 규모가 커지고 네트워크 경계를 넘어서면서 DDS의 기본 발견(Discovery) 메커니즘은 명백한 한계에 부딪힌다. 이러한 한계점들이 바로 DDS 라우터의 필요성을 역설한다.



### 2.1  P2P(Peer-to-Peer) 발견 메커니즘의 한계



DDS의 표준 발견 메커니즘은 기본적으로 P2P 방식에 기반한다. 각 도메인 참여자는 자신의 존재(자신이 발행하거나 구독하는 토픽 정보 등)를 네트워크에 알리고, 동시에 다른 참여자들을 탐색한다. 이 과정은 주로 IP 멀티캐스트를 통해 이루어지며, 이를 '자동 발견(Automatic Discovery)'이라고 부른다.2

이 P2P 방식은 중앙 관리 서버가 없어 단일 장애점(Single Point of Failure)이 없고 내구성이 높다는 장점을 가진다.2 하지만 시스템이 복잡해지면서 다음과 같은 약점들이 드러난다 2:

- **관리 및 보안의 어려움:** 중앙 관리 주체가 없기 때문에 전체 시스템의 참여자를 일관되게 관리하거나 보안 정책을 적용하기 어렵다.
- **불필요한 트래픽 증가:** 새로운 참여자가 네트워크에 참여하면, 이웃 참여자를 찾기 위해 수많은 다른 참여자들과 정보를 교환해야 하므로 불필요한 검색 트래픽이 발생하고 비용이 증가한다.
- **자원 소모:** 각 참여자는 발견된 모든 다른 참여자에 대한 정보를 스스로 관리해야 하므로, 참여자 수가 늘어날수록 메모리와 CPU 자원 소모가 커진다.



### 2.2  WAN 문제: 지리적으로 분산된 네트워크 연결



표준 DDS 발견 메커니즘이 가장 큰 난관에 부딪히는 시나리오는 바로 광역 네트워크(WAN)를 통해 지리적으로 분산된 시스템을 연결해야 할 때이다.

- **멀티캐스트의 한계:** 자동 발견에 주로 사용되는 IP 멀티캐스트는 대부분의 인터넷 라우터나 기업 방화벽에서 보안상의 이유로 차단된다. 따라서 서로 다른 LAN에 위치한 DDS 참여자들은 멀티캐스트를 통해 서로를 발견할 수 없다.
- **NAT(Network Address Translation) 문제:** 각 로컬 네트워크의 참여자들은 NAT 장비 뒤에 위치하여 사설 IP 주소를 사용한다. 외부에서는 이 사설 IP로 직접 접근할 수 없으므로, 특별한 조치 없이는 P2P 방식의 직접적인 연결이 불가능하다.12
- **대역폭 및 지연 시간:** WAN 링크는 LAN에 비해 상대적으로 대역폭이 좁고 지연 시간이 길며, 데이터 손실률이 높다.13 이러한 환경에 일반적인 DDS 설정을 그대로 적용하면 네트워크 포화 상태를 유발하거나 통신 타임아웃을 초래할 수 있다.

이러한 WAN 환경의 문제점을 해결하기 위해 DDS 라우터는 통신 패러다임을 근본적으로 변환한다. 라우터는 일반적으로 WAN 통신을 위해 TCP 프로토콜을 사용한다.9 반면, DDS는 본래 실시간성과 낮은 지연 시간을 위해 UDP를 선호한다.13 라우터를 사용함으로써 시스템 아키텍처는 효과적으로 분리된다. 각 LAN 내부의 참여자들은 효율적인 UDP 멀티캐스트를 사용하여 통신을 계속할 수 있다. 각 LAN의 경계에 위치한 라우터는 이 로컬 UDP 트래픽을 구독하는 '프록시' 또는 '게이트웨이' 역할을 한다.13 그리고 수신한 데이터를 안정적이고 방화벽 통과가 용이한 TCP 터널을 통해 원격지의 다른 라우터로 전송한다. 원격지 라우터는 이 데이터를 다시 자신의 로컬 LAN에 효율적인 UDP로 재발행한다. 결과적으로 DDS 라우터는 각 네트워크 구간의 특성에 가장 적합한 전송 프로토콜을 선택적으로 사용하는 '최적의 전송 전략'을 가능하게 한다. 이는 표준 DDS 참여자만으로는 쉽게 구현하기 어려운 수준의 제어 능력이다.



### 2.3  확장성과 성능 병목 현상



시스템에 참여하는 노드 수가 수백, 수천 개로 증가하면 P2P 발견 메커니즘은 성능 병목을 유발한다.

- **디스커버리 스톰(Discovery Storms):** 대규모 시스템에서 다수의 참여자가 동시에 서로를 발견하려 할 때 발생하는 과도한 양의 P2P 발견 트래픽은 '디스커버리 스톰' 현상을 일으킬 수 있다.4 이는 네트워크 대역폭과 모든 노드의 CPU 자원을 심각하게 소모시켜 실제 데이터 통신을 방해한다. 보안 기능이 추가된 DDS(Secure DDS)에서는 인증서 교환 등으로 인해 이 부하가 더욱 커진다.4
- **불필요한 데이터 트래픽:** 필터링 기능이 없는 순수 P2P 모델에서는 특정 토픽으로 발행된 모든 데이터가 해당 토픽을 구독하는 모든 구독자에게 전달된다. 이는 네트워크 전체에 걸쳐 불필요하고 중복된 데이터 흐름을 야기한다.
- **자원 소모 증가:** P2P 시스템에서는 모든 참여자가 전체 시스템의 토폴로지 정보를 유지해야 하므로, 시스템 규모가 커질수록 각 노드의 부담이 기하급수적으로 증가한다.2

이러한 문제들은 P2P DDS 모델에 '관리자' 또는 '브로커' 역할의 부재에서 기인한다.2 DDS 라우터는 전략적으로 배치될 때 바로 이 부재한 관리자 역할을 수행한다. 네트워크 간 트래픽을 라우터라는 중앙 지점(또는 몇 개의 핵심 지점)을 통과하도록 제어함으로써 다음과 같은 이점을 얻을 수 있다. 첫째, 

**트래픽 제어**가 가능해진다. 라우터는 네트워크 경계에서 데이터를 필터링하여 필요한 데이터만 네트워크 간에 전달할 수 있다.9 둘째, 

**발견 범위(Discovery Scope)를 제한**할 수 있다. 발견 트래픽을 각 로컬 네트워크 내부에 국한시켜 디스커버리 스톰이 전체 시스템으로 확산되는 것을 방지한다.9 마지막으로, 

**보안 관리가 단순화**된다. WAN을 통해 연결될 수 있는 모든 P2P 링크에 대해 보안 자격 증명과 접근 제어를 관리하는 대신, 라우터 대 라우터로 연결되는 지점에서 보안을 집중적으로 강화할 수 있다.



## 3.  DDS 라우팅의 아키텍처 원칙



DDS 라우터는 일반적인 네트워크 라우터와는 근본적으로 다르다. 프로토콜에 관계없이 IP 패킷을 전달하는 네트워크 장비가 아니라, DDS 생태계 내에서 동작하는 특수한 목적의 DDS 애플리케이션이다. 라우터의 동작 원리를 이해하기 위해서는 이러한 핵심 원칙들을 먼저 파악해야 한다.



### 3.1  특수 목적의 DDS 참여자로서의 라우터



DDS 라우터의 가장 기본적인 정체성은 하나 이상의 DDS 도메인에 동시에 참여하는 '특수 목적의 DDS 참여자'라는 점이다. 라우터는 내부에 여러 개의 `DomainParticipant`를 가지며, 이를 통해 서로 다른 도메인이나 네트워크에 연결된다. 데이터 흐름의 관점에서 보면, 라우터는 한쪽 도메인에서 데이터를 구독하기 위한 내부적인 `DataReader`를 생성하고, 이 데이터를 다른 쪽 도메인으로 발행하기 위한 내부적인 `DataWriter`를 생성하여 둘을 연결하는 역할을 수행한다. 이 개념은 라우터의 설정 파일이 왜 `DomainParticipant`와 관련된 파라미터(예: 도메인 ID, QoS)를 정의하는지를 설명해준다.



### 3.2  핵심 기능: 전달, 필터링, 변환



DDS 라우터는 단순히 데이터를 한 곳에서 다른 곳으로 전달하는 것 이상의 정교한 기능을 제공한다.

- **전달(Forwarding):** 입력(Input)으로 데이터를 수신하여 출력(Output)으로 보내는 가장 기본적인 기능이다.
- **필터링(Filtering):** 특정 기준에 따라 데이터를 선택적으로 전달하는 기능이다. 이는 단순히 토픽 이름을 기반으로 허용 목록(`allowlist`)을 만드는 것부터 14, 데이터의 내용(값)을 기반으로 필터링하는 복잡한 기능까지 포함할 수 있다.9 이를 통해 네트워크 대역폭을 절약하고 수신측 애플리케이션의 부하를 줄일 수 있다.
- **변환(Transformation):** 데이터가 라우터를 통과할 때 내용을 수정하는 기능으로, 이는 라우터의 가장 강력한 기능 중 하나이다. 간단하게는 토픽 이름을 변경하는 것부터 시작하여, 서로 다른 데이터 타입(Type)을 사용하는 이종 시스템 간의 통신을 위해 데이터 구조를 변환하는 것까지 가능하다.9



### 3.3  일반적인 라우팅 토폴로지 및 활용 사례



DDS 라우터는 다양한 아키텍처 패턴을 구현하는 데 사용될 수 있다.

- **도메인 브리지(Domain Bridge):** 두 개 이상의 DDS 도메인을 연결하여 마치 하나의 거대한 논리적 도메인처럼 보이게 만드는 가장 간단한 토폴로지이다.9 개발팀별, 또는 기능별로 분리된 도메인 간의 데이터 공유를 위해 널리 사용된다.
- **계층적 데이터버스 / 시스템-오브-시스템즈(Layered Databus / System-of-Systems):** 대규모 국방, 항공, 산업 시스템에서 필수적인 아키텍처이다. 각 하위 시스템은 자체적인 '사설(private)' 도메인 내에서 통신하며, 라우터는 이 하위 시스템들의 데이터 중 일부 선택된 데이터만을 상위의 '글로벌 시스템-오브-시스템즈' 도메인으로 노출시키는 역할을 한다.9 이를 통해 하위 시스템의 자율성을 보장하면서 전체 시스템의 통합을 이룰 수 있다.
- **클라우드/엣지 배포(Cloud/Edge Deployment):** 여러 지역에 분산된 엣지 디바이스들이 중앙 클라우드 서비스와 통신하는 현대적인 IoT 아키텍처이다. 각 엣지 네트워크의 경계와 클라우드에 라우터를 배포하여 엣지와 클라우드 간의 통신을 연결한다.14
- **레거시 시스템 통합(Legacy System Integration):** DDS 라우터는 커스텀 어댑터(Adapter)를 통해 MQTT, 파일 기반 시스템 등 DDS를 사용하지 않는 기존 레거시 시스템과 최신 DDS 애플리케이션 간의 데이터 다리 역할을 수행할 수 있다.13 이는 기존 시스템을 점진적으로 현대화하는 데 매우 유용한 전략이다.



## 4.  심층 분석: eProsima Fast DDS Router



eProsima Fast DDS Router는 오픈 소스 DDS 구현체인 Fast DDS를 기반으로 하는 라우팅 솔루션이다. 성능, 효율성, 그리고 사용 편의성에 중점을 두고 설계되었으며, 특히 로보틱스 및 자율 시스템 커뮤니티에서 널리 채택되고 있다.



### 4.1  개요 및 설계 철학



eProsima Fast DDS Router는 분산된 DDS 네트워크를 연결하기 위해 설계된 최종 사용자 소프트웨어 애플리케이션이다.14 이 라우터의 핵심 설계 철학은 '경량화'와 '고속 데이터 전달'에 있다. 공식 문서에서 강조하는 '데이터 내성(introspection) 회피' 및 '제로카피(zero-copy) 시스템'과 같은 특징들은 라우터가 데이터를 복잡하게 처리하기보다는 최대한 빠른 속도로 전달하는 데 집중하고 있음을 보여준다.14 이러한 접근 방식은 실시간성이 중요한 시스템에서 라우터 자체의 오버헤드를 최소화하는 데 기여한다.



### 4.2  주요 특징



eProsima Fast DDS Router는 다음과 같은 핵심적인 특징을 제공한다.

- **TCP 기반 WAN 통신:** 인터넷을 통한 안정적인 DDS 통신을 위해 TCP 전송을 지원한다.14
- **분산형 특성:** 중앙 집중식 노드 없이, 중간 라우터 노드를 배포하여 동적으로 네트워크에 참여하거나 이탈하는 새로운 개체(entity)를 자동으로 발견할 수 있는 메시(mesh) 네트워크 구성을 지원한다.14
- **효율적인 데이터 라우팅:** 데이터의 내용을 들여다보지 않고 전달하는 제로카피 시스템을 구현하여 데이터 포워딩 성능을 극대화한다.14
- **손쉬운 배포:** 네트워크에 대한 전문 지식이 없는 사용자도 쉽게 설정할 수 있는 모듈식 YAML 설정 파일을 기반으로 한다.14
- **토픽 필터링:** 사용자가 지정한 토픽(이름 및 타입 기준)에 속하는 데이터만 전달하도록 허용/차단 목록(allow/blocklist)을 설정할 수 있다.14
- **동적 토픽 발견:** 허용 목록 규칙과 일치하는 토픽이 발견되면, 통신에 필요한 모든 DDS 개체(DataReader/DataWriter)를 자동으로 생성하여 통신을 시작한다.14



### 4.3  내부 아키텍처 및 참여자 유형



eProsima DDS Router의 아키텍처는 여러 종류의 '참여자(Participant)'를 설정하는 것을 기반으로 한다. 참여자는 라우터가 외부 네트워크와 통신하는 인터페이스 역할을 한다.22 설정 파일에서 정의할 수 있는 주요 참여자 종류(

`kind`)는 다음과 같다.

- **`local` (또는 `simple`):** 라우터가 실행되는 로컬 머신 또는 로컬 네트워크의 표준 DDS 도메인에 연결하기 위해 사용된다. 특정 도메인 ID를 지정하여 해당 도메인에 참여한다.
- **`wan`:** WAN(Wide Area Network)을 통해 다른 DDS 라우터와 통신하기 위해 사용된다. 주로 Discovery Server 메커니즘을 기반으로 동작하며, 원격 라우터의 주소와 포트를 지정하여 연결을 설정한다.

데이터는 한 참여자의 내부 엔드포인트(DataReader)에서 수신되어, 설정된 토픽 허용 목록에 따라 다른 참여자의 내부 엔드포인트(DataWriter)로 전달된다. '제로카피' 메커니즘은 이 과정에서 데이터가 메모리상에서 복사되지 않고 참조를 통해 전달됨을 의미하며, 이는 지연 시간을 줄이고 처리량을 높이는 데 결정적인 역할을 한다.21



### 4.4  라이선스 및 생태계



eProsima Fast DDS와 DDS Router는 아파치 2.0 라이선스(Apache 2.0 License) 하에 배포된다.19 이는 상업적 사용, 수정, 배포가 자유로운 매우 허용적인 오픈 소스 라이선스로, 사용자는 라이선스 비용 부담 없이 솔루션을 도입하고 필요에 따라 코드를 수정할 수 있다.

eProsima의 솔루션은 특히 로봇 운영체제(ROS 2) 생태계와 매우 밀접한 관련이 있다. eProsima Fast DDS는 ROS 2의 기본(default) DDS 미들웨어로 채택되었으며 11, DDS Router는 eProsima가 제공하는 로보틱스 개발 플랫폼인 Vulcanexus의 핵심 구성 요소 중 하나이다.25 이러한 긴밀한 통합은 DDS Router가 로보틱스 및 자율 시스템 분야의 요구사항을 충족시키기 위해 적극적으로 개발되고 검증되었음을 의미한다. 예를 들어, 여러 대의 로봇으로 구성된 로봇 플릿(fleet)을 클라우드 백엔드와 연결하는 시나리오에서 DDS Router는 단일 LAN을 넘어 ROS 2 시스템을 확장하기 위한 자연스럽고 강력한 솔루션을 제공한다.26 따라서 ROS 2 기반의 시스템을 개발하는 아키텍트에게 eProsima DDS Router는 매우 매력적인 선택지가 될 수 있다.



## 5.  실전 가이드: eProsima Fast DDS Router 구현



이 장에서는 eProsima Fast DDS Router를 실제 시스템에 설치하고, 설정하며, 실행하는 구체적인 방법을 단계별로 안내한다. 개발자 중심의 소스 코드 컴파일 방식부터 다양한 시나리오에 맞는 YAML 설정 파일 작성법까지 다룬다.



### 5.1  리눅스 환경에서 설치



eProsima DDS Router는 소스 코드를 직접 컴파일하여 설치하는 방식을 권장한다. 이를 통해 최신 기능을 사용하고 시스템에 최적화된 빌드를 생성할 수 있다.

1. 사전 요구사항 설치:

먼저, 컴파일에 필요한 기본적인 도구들을 설치해야 한다. 우분투(Ubuntu) 기반 시스템에서는 다음 명령어를 사용한다.27

Bash

```
sudo apt update
sudo apt install cmake g++ python3-pip wget git
```

또한, 의존성 관리 및 빌드 도구인 `colcon`을 설치한다.27

Bash

```
pip3 install -U colcon-common-extensions vcstool
```

2. 소스 코드 다운로드:

작업 공간을 생성하고, ddsrouter.repos 파일을 이용해 DDS Router와 모든 의존성 패키지(Fast DDS, Fast CDR 등)의 소스 코드를 다운로드한다.27

Bash

```
mkdir -p ~/DDS-Router-Workspace/src
cd ~/DDS-Router-Workspace
wget https://raw.githubusercontent.com/eProsima/DDS-Router/main/ddsrouter.repos
vcs import src < ddsrouter.repos
```

3. 컴파일 및 설치:

colcon을 사용하여 모든 패키지를 한 번에 빌드하고 설치한다.

Bash

```
cd ~/DDS-Router-Workspace
colcon build --symlink-install
```

4. 환경 설정:

컴파일된 실행 파일을 사용하기 위해, 터미널 세션마다 다음 명령어를 실행하여 환경을 설정해야 한다. 편의를 위해 .bashrc 파일에 추가하여 자동으로 로드되게 할 수 있다.28

Bash

```
source ~/DDS-Router-Workspace/install/setup.bash
```



### 5.2  YAML 설정 파일: 상세 분석



eProsima DDS Router의 모든 동작은 YAML 형식의 설정 파일을 통해 제어된다. 파일의 구조는 직관적이며, 주요 섹션은 다음과 같다.21

- `version`: 설정 파일의 버전 (예: `v4.0`).
- `participants`: 라우터의 통신 인터페이스를 정의하는 배열. 라우터의 핵심 설정 부분이다.
- `allowlist` / `blocklist`: 특정 토픽의 라우팅을 허용하거나 차단하는 규칙을 정의한다.
- `builtin-topics`: DDS의 내장 토픽(Discovery 정보 등)을 라우팅할지 여부를 설정한다.

다음 표는 `participants` 설정에 사용되는 주요 파라미터를 요약한 것이다. 이 표는 개발자가 빠르게 설정을 구성할 수 있도록 돕는 빠른 참조 가이드 역할을 한다.

**표 5.2.1: eProsima DDS Router Participant 주요 설정 파라미터**

| 파라미터               | 설명                                                       | 예시 값                            |
| ---------------------- | ---------------------------------------------------------- | ---------------------------------- |
| `name`                 | 참여자의 고유한 이름.                                      | `local_participant_0`              |
| `kind`                 | 참여자의 종류. 로컬 DDS 도메인, WAN 연결 등을 결정한다.    | `local`, `wan`, `discovery-server` |
| `domain`               | `kind`가 `local`일 때, 참여할 DDS 도메인 ID.               | `0`, `1`,...                       |
| `listening-addresses`  | WAN 참여자가 외부 연결을 수신 대기할 주소와 포트 목록.     | `ip: 0.0.0.0`, `port: 11666`       |
| `connection-addresses` | WAN 참여자가 연결을 시도할 원격 라우터의 주소와 포트 목록. | `ip: 203.0.113.10`, `port: 11666`  |
| `transport`            | 사용할 전송 프로토콜.                                      | `udp`, `tcp`                       |
| `external-port`        | NAT 환경에서 외부에서 접근 가능한 공용 포트.               | `34567`                            |



### 5.3  시나리오별 설정 레시피



다음은 일반적인 사용 사례에 대한 완전한 YAML 설정 파일 예제이다.



#### 5.3.1  단순 도메인 브리지 (LAN)



가장 기본적인 시나리오로, 로컬 네트워크 내의 DDS 도메인 0과 도메인 1을 연결한다. 이 설정은 두 도메인 간의 모든 토픽 트래픽을 양방향으로 전달한다.

YAML

```
# file: simple-bridge.yaml
version: v4.0

participants:
  - name: participant_domain_0
    kind: local
    domain: 0

  - name: participant_domain_1
    kind: local
    domain: 1
```



#### 5.3.2  선택적 토픽 필터링



위의 도메인 브리지 시나리오에서, 'HelloWorldTopic'이라는 특정 토픽만 라우팅하도록 `allowlist`를 추가한다. `blocklist`가 명시되지 않으면, `allowlist`에 없는 모든 토픽은 차단된다.

YAML

```
# file: selective-filter.yaml
version: v4.0

allowlist:
  - name: HelloWorldTopic
    type: HelloWorld     # 데이터 타입 이름

participants:
  - name: participant_domain_0
    kind: local
    domain: 0

  - name: participant_domain_1
    kind: local
    domain: 1
```



#### 5.3.3  WAN 링크 설정



지리적으로 떨어진 두 개의 LAN을 연결하는 시나리오이다. 각 사이트에 라우터가 하나씩 배포된다고 가정한다.

**라우터 1 (Site A) 설정:**

- 공인 IP: 203.0.113.10
- 로컬 LAN은 도메인 0을 사용한다.
- 외부 연결을 위해 포트 11666에서 수신 대기한다.
- 라우터 2(Site B)의 주소로 연결을 시도한다.

YAML

```
# file: router-A.yaml
version: v4.0

participants:
  # 로컬 LAN (도메인 0) 인터페이스
  - name: lan_participant
    kind: local
    domain: 0

  # WAN 인터페이스
  - name: wan_participant
    kind: wan
    transport: tcp
    listening-addresses:
      - ip: 0.0.0.0
        port: 11666
    connection-addresses:
      - ip: 198.51.100.20   # 라우터 2의 공인 IP
        port: 11666
```

**라우터 2 (Site B) 설정:**

- 공인 IP: 198.51.100.20
- 로컬 LAN은 도메인 5를 사용한다.
- 외부 연결을 위해 포트 11666에서 수신 대기한다.
- 라우터 1(Site A)의 주소로 연결을 시도한다.

YAML

```
# file: router-B.yaml
version: v4.0

participants:
  # 로컬 LAN (도메인 5) 인터페이스
  - name: lan_participant
    kind: local
    domain: 5

  # WAN 인터페이스
  - name: wan_participant
    kind: wan
    transport: tcp
    listening-addresses:
      - ip: 0.0.0.0
        port: 11666
    connection-addresses:
      - ip: 203.0.113.10   # 라우터 1의 공인 IP
        port: 11666
```



### 5.4  기본 실행 및 검증



DDS Router는 터미널에서 간단한 명령어로 실행할 수 있다. `-c` 또는 `--config-path` 옵션으로 설정 파일의 경로를 지정한다.30

Bash

```
ddsrouter -c /path/to/your/config.yaml
```

라우터가 정상적으로 동작하는지 검증하기 위해, `ShapesDemo`와 같은 시각화 도구나 간단한 발행/구독 애플리케이션을 사용할 수 있다.31 예를 들어, '단순 도메인 브리지' 시나리오에서는 한 터미널에서 도메인 0으로 

`ShapesDemo` 발행자를 실행하고, 다른 터미널에서 도메인 1로 구독자를 실행한다. 라우터가 실행되면, 이전에는 통신할 수 없었던 두 `ShapesDemo` 인스턴스 간에 도형 데이터가 전달되는 것을 확인할 수 있다.



## 6.  심층 분석: RTI Connext Routing Service



RTI Connext Routing Service는 상용 DDS 솔루션의 선두주자인 RTI(Real-Time Innovations)가 제공하는 라우팅 솔루션이다. 대규모의 복잡하고 지리적으로 분산된 시스템을 확장하고 통합하는 데 초점을 맞춘 강력한 기능을 제공하며, 고도의 신뢰성과 성능이 요구되는 미션 크리티컬 산업 분야에서 널리 사용된다.



### 6.1  개요 및 Connext 생태계 내에서의 위치



RTI Routing Service는 단순한 데이터 전달 도구가 아니라, RTI Connext 제품군(Product Suite)의 핵심 구성 요소이다.9 이 제품군에는 시스템 모니터링을 위한 

`Admin Console`, 데이터 기록 및 재생을 위한 `Recording Service`, 그리고 강력한 보안 기능을 제공하는 `Security Plugins` 등이 포함되어 있으며, Routing Service는 이 모든 도구와 긴밀하게 통합되어 동작한다.32 이러한 생태계 접근 방식은 개발, 배포, 운영, 유지보수에 이르는 시스템의 전체 수명 주기에 걸쳐 일관되고 강력한 관리 기능을 제공하는 것을 목표로 한다.



### 6.2  고급 기능



RTI Routing Service는 eProsima DDS Router와 같은 오픈 소스 솔루션과 차별화되는 여러 고급 기능을 제공한다.

- **데이터 변환(Data Transformation):** 이는 RTI Routing Service의 가장 강력하고 독보적인 기능이다. 라우터는 데이터를 전달하는 과정에서 실시간으로 내용을 변환할 수 있다.9 단순히 토픽 이름을 바꾸는 수준을 넘어, 서로 다른 데이터 타입(Type)을 사용하는 시스템 간의 통신을 가능하게 한다.15 예를 들어, 구형 시스템에서 사용하는 

  `LegacySensorData` 타입을 신형 시스템에서 사용하는 `ModernSensorData` 타입으로 필드를 매핑하고 단위를 변환하여 전달할 수 있다.

- **콘텐츠 필터링(Content Filtering):** 단순한 토픽 기반 필터링을 넘어, 데이터의 내용(값)에 기반한 필터링을 지원하고 이 필터 조건을 원격 네트워크로 전파할 수 있다.17 예를 들어, 원격지의 구독자가 "온도가 100도 이상인 데이터만 필요하다"는 필터를 설정하면, 라우터는 이 조건에 맞는 데이터만 WAN을 통해 전송한다. 이는 불필요한 네트워크 대역폭 낭비를 최소화하는 데 매우 효과적이다.

- **플러그형 아키텍처 (어댑터 및 프로세서):** Routing Service는 플러그인을 통해 기능을 무한히 확장할 수 있는 구조를 가지고 있다. RTI는 MQTT, 파일, JMS 등 비-DDS 시스템과 연동하기 위한 다양한 어댑터(Adapter)를 제공하거나 개발할 수 있도록 SDK를 지원한다.18 또한, 사용자는 커스텀 프로세서(Processor)를 C/C++/Java 등으로 직접 개발하여 복잡한 라우팅 로직이나 데이터 처리 알고리즘을 라우터 내부에 구현할 수 있다.18 이를 통해 Routing Service는 단순한 DDS 브리지를 넘어 강력한 '통합 허브(Integration Hub)'로 기능할 수 있다.

이러한 고급 기능, 특히 데이터 변환 기능은 시스템의 수명 주기 관리 측면에서 매우 중요한 의미를 가진다. 국방, 항공우주와 같이 수십 년간 운영되는 시스템에서는 필연적으로 구형 컴포넌트와 신형 컴포넌트가 공존하게 된다. 9 문서에서 언급된 "데이터 모델의 진화(Data model evolution)"와 17의 "송신과 수신 애플리케이션이 동일한 데이터 구조를 사용할 필요가 없도록 데이터 타입을 변경할 수 있다"는 설명은 바로 이 점을 지적한다. 신형 컴포넌트가 새로운 데이터 모델을 사용하더라도, 기존의 구형 컴포넌트를 수정하는 것이 불가능하거나 엄청난 비용이 드는 경우가 많다. 이때 RTI Routing Service를 '번역 계층(Translation Layer)'으로 배치하면, 구형과 신형 데이터 모델 간의 변환을 라우터가 담당하게 된다. 이는 각 하위 시스템이 서로의 개발 주기나 데이터 모델에 종속되지 않고 독립적으로 발전할 수 있도록 하는 '디커플링 에이전트(Decoupling Agent)' 역할을 수행하는 것이다. 이는 단순한 데이터 전달과는 차원이 다른, 아키텍처 수준의 유연성을 제공하는 핵심적인 가치이다.



### 6.3  RTI 툴링 및 보안과의 통합



Routing Service는 RTI의 강력한 툴링 생태계와 완벽하게 연동된다. `RTI Administration Console`을 사용하면 원격에서 실행 중인 Routing Service에 접속하여 라우트 상태, 데이터 처리량, 지연 시간 등의 성능 지표를 실시간으로 모니터링하고, 동적으로 설정을 변경할 수도 있다.33

또한, RTI Security Plugins와 완벽하게 통합되어 인증, 접근 제어, 암호화 등 DDS-Security 표준을 준수하는 강력한 보안 기능을 라우팅 경로 전체에 적용할 수 있다.34 이를 통해 민감한 데이터가 여러 네트워크 경계를 넘어 전달될 때 종단 간(end-to-end) 보안을 보장할 수 있다.



### 6.4  상용 솔루션으로서의 고려사항



RTI Connext Routing Service는 상용 소프트웨어로, 사용을 위해서는 라이선스 구매가 필요하다.15 이는 초기 도입 비용이 발생함을 의미하지만, 동시에 미션 크리티컬 시스템에 필수적인 여러 이점을 제공한다. RTI는 전문적인 기술 지원(Professional Support), 상세하고 체계적인 공식 문서, 그리고 제품의 지속적인 유지보수 및 업데이트를 보장한다.17 안정성과 신뢰성이 최우선인 시스템에서는 이러한 상용 지원의 가치가 매우 크다.



## 7.  실전 가이드: RTI Connext Routing Service 구현



이 장에서는 RTI Connext Routing Service를 시스템에 배포하고 설정하는 실질적인 절차를 다룬다. 환경 설정부터 복잡한 XML 파일 작성, 그리고 배포 후 모니터링까지의 과정을 상세히 설명한다.



### 7.1  설치 및 환경 설정



RTI Routing Service는 RTI Connext DDS Professional 제품군에 포함되어 배포된다. 설치 과정 자체는 일반적인 소프트웨어 설치와 유사하지만, 실행을 위해서는 몇 가지 중요한 환경 설정이 필요하다.

1. NDDSHOME 환경 변수 설정:

RTI Connext DDS가 설치된 디렉토리 경로를 NDDSHOME 환경 변수로 지정해야 한다. Routing Service 실행 파일과 관련 라이브러리들이 이 경로를 기준으로 동작한다.36

Bash

```
# 예시: Linux 환경
export NDDSHOME=/opt/rti_connext_dds-6.1.1
```

2. 라이선스 파일 설치:

RTI 제품은 유효한 라이선스 파일(rti_license.dat)을 필요로 한다. 이 파일은 RTI로부터 발급받아 지정된 위치에 두거나, RTI_LICENSE_FILE 환경 변수를 통해 파일의 경로를 명시해야 한다.15



### 7.2  XML 설정 파일: 구조 및 주요 태그



RTI Routing Service의 모든 설정은 XML 파일을 통해 이루어진다. 그 구조는 계층적이며, 주요 태그들의 역할을 이해하는 것이 중요하다.34

- `<routing_service>`: 특정 라우팅 서비스 구성을 정의하는 최상위 태그. `name` 속성으로 고유한 이름을 가진다.
- `<domain_route>`: 데이터가 흐르는 경로의 양 끝, 즉 도메인 또는 커넥션을 정의한다.
- `<participant>`: `<domain_route>` 내에서 DDS 도메인에 대한 연결을 정의한다. `domain_id`와 QoS 설정을 포함한다.
- `<session>`: 동일한 QoS와 처리 방식을 공유하는 라우트들의 그룹이다.
- `<topic_route>`: 실제 데이터 라우팅 규칙을 정의하는 가장 핵심적인 태그. 입력(input)과 출력(output)을 지정한다.

다음 표는 RTI Routing Service의 핵심 XML 태그들을 요약하여 구조를 쉽게 파악할 수 있도록 돕는다.

**표 7.2.1: RTI Routing Service 핵심 XML 설정 태그**

| 태그                | 설명                                                   | 주요 속성/하위 태그                                 |
| ------------------- | ------------------------------------------------------ | --------------------------------------------------- |
| `<routing_service>` | 라우팅 서비스 전체 설정을 감싸는 루트 요소.            | `name`                                              |
| `<domain_route>`    | 두 개 이상의 참여자(네트워크) 간의 라우팅 경로를 정의. | `name`, `<participant>`, `<connection>`             |
| `<participant>`     | DDS 도메인에 대한 참여를 정의.                         | `name`, `domain_id`, `<participant_qos>`            |
| `<session>`         | 라우트들의 논리적 그룹.                                | `name`, `<topic_route>`                             |
| `<topic_route>`     | 특정 토픽에 대한 라우팅 규칙을 상세히 정의.            | `name`, `<input>`, `<output>`, `<transformation>`   |
| `<input>`           | 데이터 소스를 정의.                                    | `participant`, `topic_name`, `registered_type_name` |
| `<output>`          | 데이터 목적지를 정의.                                  | `participant`, `topic_name`, `registered_type_name` |
| `<transformation>`  | 데이터 변환 로직을 정의.                               | `<script>`, `<transformation_plugin>`               |



### 7.3  시나리오별 설정 레시피



다음은 RTI Routing Service의 강력한 기능을 보여주는 XML 설정 예제들이다.



#### 7.3.1  서로 다른 도메인 연결하기



도메인 0에서 발행되는 `Square` 토픽을 도메인 1로 라우팅하는 기본 설정이다.16

XML

```
<dds>
    <routing_service name="SimpleDomainBridge">
        <domain_route name="Route_0_to_1">
            <participant name="Domain0_Participant">
                <domain_id>0</domain_id>
            </participant>
            <participant name="Domain1_Participant">
                <domain_id>1</domain_id>
            </participant>
        </domain_route>
        <session name="ForwardingSession">
            <topic_route name="Route_Square">
                <input participant="Domain0_Participant">
                    <topic_name>Square</topic_name>
                    <registered_type_name>ShapeType</registered_type_name>
                </input>
                <output participant="Domain1_Participant">
                    <topic_name>Square</topic_name>
                    <registered_type_name>ShapeType</registered_type_name>
                </output>
            </topic_route>
        </session>
    </routing_service>
</dds>
```



#### 7.3.2  데이터 변환하기



도메인 0의 `Square` 토픽을 수신하여, 토픽 이름을 `Circle`로 변경하고 데이터 내용의 `x` 좌표 값을 100만큼 더해서 도메인 1로 발행하는 예제이다. 이는 `<transformation>` 태그의 강력함을 보여준다.39

XML

```
<dds>
    <routing_service name="TransformingBridge">
        <session name="TransformationSession">
            <topic_route name="Transform_Square_To_Circle">
                <input participant="Domain0_Participant">
                    <topic_name>Square</topic_name>
                    <registered_type_name>ShapeType</registered_type_name>
                </input>
                <output participant="Domain1_Participant">
                    <topic_name>Circle</topic_name>
                    <registered_type_name>ShapeType</registered_type_name>
                </output>
                <transformation>
                    <script>
                        <language>lua</language>
                        <code>
                            -- 입력 데이터의 x 좌표에 100을 더함
                            output.x = input.x + 100
                            -- 나머지 필드는 그대로 복사
                            output.y = input.y
                            output.shapesize = input.shapesize
                            output.color = input.color
                            return output
                        </code>
                    </script>
                </transformation>
            </topic_route>
        </session>
    </routing_service>
</dds>
```



#### 7.3.3  보안 TCP 기반 WAN 라우팅



두 개의 라우터가 WAN을 통해 보안 TCP 연결을 맺는 설정이다. 멀티캐스트를 비활성화하고, TCP 전송 속성을 설정하며, 보안 플러그인을 활성화하는 위치를 보여준다.16

**라우터 1 (Site A) 설정:**

XML

```
<dds>
    <routing_service name="Secure_WAN_Router_A">
        <domain_route name="WAN_Link">
            <participant name="Local_LAN">
                <domain_id>0</domain_id>
            </participant>
            <participant name="WAN_TCP">
                <domain_id>99</domain_id> <participant_qos>
                    <discovery>
                        <initial_peers>
                            <element>tcpv4://198.51.100.20:7401</element>
                        </initial_peers>
                    </discovery>
                    <transport_builtin>
                        <mask>TCPV4_WAN</mask>
                    </transport_builtin>
                    <property>
                        <value>
                            <element>
                                <name>com.rti.serv.secure.enable</name>
                                <value>true</value>
                            </element>
                            </value>
                    </property>
                </participant_qos>
            </participant>
        </domain_route>
        </routing_service>
</dds>
```

(라우터 B는 `initial_peers`에 라우터 A의 주소를 명시하여 유사하게 설정)



### 7.4  배포 및 모니터링



Routing Service는 다음 명령어를 통해 실행한다. `-cfgFile`로 XML 파일 경로를, `-cfgName`으로 실행할 `<routing_service>`의 이름을 지정한다.

Bash

```
rtiroutingservice -cfgFile my_config.xml -cfgName Secure_WAN_Router_A
```

실행 후, `RTI Admin Console`을 실행하여 라우터의 관리 도메인(설정 파일의 `<administration>` 태그에 정의된 `domain_id`)에 접속하면, 라우터의 상태, 데이터 흐름, 성능 지표 등을 그래픽 인터페이스를 통해 실시간으로 확인하고 관리할 수 있다.33



## 8.  비교 분석 및 고급 주제



지금까지 eProsima Fast DDS Router와 RTI Connext Routing Service의 아키텍처와 사용법을 각각 살펴보았다. 이 장에서는 두 솔루션을 직접적으로 비교 분석하고, 라우팅 시스템 구축 시 공통적으로 고려해야 할 보안, 성능, 문제 해결과 같은 고급 주제들을 다룬다.



### 8.1  직접 비교: eProsima vs. RTI



두 라우터는 분산된 DDS 네트워크를 연결한다는 동일한 목표를 가지지만, 그 철학과 기능, 대상 시장에서 뚜렷한 차이를 보인다. 시스템 아키텍트가 프로젝트의 요구사항에 가장 적합한 솔루션을 선택하기 위해서는 이러한 차이점을 명확히 이해해야 한다.

**표 8.1.1: 기능 및 아키텍처 비교: eProsima vs. RTI**

| 항목              | eProsima Fast DDS Router                            | RTI Connext Routing Service                             |
| ----------------- | --------------------------------------------------- | ------------------------------------------------------- |
| **라이선스**      | Apache 2.0 (오픈 소스, 무료) 19                     | 상용 라이선스 (유료, 기술 지원 포함) 32                 |
| **설정 방식**     | YAML 14                                             | XML 34                                                  |
| **핵심 철학**     | 경량, 고성능 데이터 포워더 14                       | 엔터프라이즈급 통합 플랫폼 9                            |
| **데이터 변환**   | 지원하지 않음 (단순 전달에 집중)                    | 강력한 기능 제공 (타입, 토픽, 값 변환) 9                |
| **확장성**        | 제한적 (기본 기능에 충실)                           | 플러그인 아키텍처 (Adapter, Processor)로 높은 확장성 18 |
| **툴링/모니터링** | `Fast DDS Monitor` 등 별도 오픈 소스 도구와 연동 41 | `Admin Console`과 완벽히 통합된 원격 관리/모니터링 33   |
| **주요 생태계**   | ROS 2, 로보틱스, 오픈 소스 커뮤니티 25              | 국방, 항공, 의료, 산업 자동화 등 미션 크리티컬 산업 9   |
| **보안 통합**     | Fast DDS 보안 기능 활용 (XML 프로파일) 42           | Connext Security Plugins와 완벽 통합 (XML 직접 설정) 34 |

이 비교표는 의사 결정의 핵심적인 기준을 제공한다. **eProsima Fast DDS Router**는 라이선스 비용이 없고 ROS 2와의 통합이 뛰어나므로, 오픈 소스 기반의 프로젝트, 특히 로보틱스나 자율주행 연구 개발 단계에 매우 적합하다. 핵심 기능인 데이터 전달 성능에 집중하여 가볍고 빠르게 동작하는 것이 장점이다.

반면, **RTI Connext Routing Service**는 이종 시스템 간의 데이터 변환, 레거시 시스템 통합, 그리고 강력한 중앙 관리 및 모니터링 기능이 필수적인 대규모 엔터프라이즈 및 미션 크리티컬 시스템에 적합하다. 초기 도입 비용이 있지만, 보장된 기술 지원과 시스템의 장기적인 진화 및 유지보수를 지원하는 강력한 기능은 그 가치를 상쇄하고도 남는다.



### 8.2  라우팅 통신의 보안



분산된 네트워크를 연결할 때 보안은 가장 중요한 고려사항 중 하나이다. DDS Security 표준은 이를 위해 인증, 접근 제어, 암호화의 세 가지 핵심 플러그인을 정의한다.4

- **eProsima DDS Router**는 기반이 되는 Fast DDS의 보안 기능을 그대로 활용한다. 보안 관련 설정(인증서 경로, 권한 파일 등)이 포함된 XML 프로파일을 작성하고, 라우터의 YAML 설정 파일에서 이 프로파일을 로드하여 참여자에게 적용하는 방식으로 보안을 활성화할 수 있다.42 또한, WAN 통신 시 TCP 전송 계층 자체를 암호화하는 TLS(Transport Layer Security)를 설정할 수도 있다.44
- **RTI Routing Service**는 RTI Security Plugins와 직접적으로 통합된다. 라우터의 XML 설정 파일 내 `<participant_qos>` 섹션에 보안 관련 프로퍼티를 직접 명시하여 인증, 암호화, 접근 제어 플러그인을 활성화하고 상세하게 설정할 수 있다.34 이는 라우팅 설정과 보안 설정을 하나의 파일에서 일관되게 관리할 수 있다는 장점이 있다.



### 8.3  성능 튜닝 및 최적화



라우터는 데이터 경로의 중간에 위치하므로, 그 자체의 성능이 전체 시스템의 성능에 큰 영향을 미친다.

- **eProsima**의 경우, 고속 데이터 전송 시 운영체제의 소켓 버퍼 크기를 늘려주는 것이 성능 향상에 도움이 될 수 있다. Fast DDS 문서에서는 `send_socket_buffer_size`와 `listen_socket_buffer_size` 파라미터 조정을 권장한다.45 또한, 제로카피 및 공유 메모리(Shared Memory) 전송을 활용하면 프로세스 간 통신 오버헤드를 극적으로 줄일 수 있다.46

- **RTI**의 경우, 성능 문제의 가장 흔한 원인 중 하나는 QoS 불일치이다. 라우터의 DataReader와 DataWriter의 QoS가 원본 발행자 및 최종 구독자와 호환되지 않으면 불필요한 재전송이나 통신 실패가 발생할 수 있으므로, QoS 설정을 신중하게 검토해야 한다.8 RTI 역시 

  `FlatData` 및 `Zero Copy` 전송과 같은 고성능 기능을 지원하며, 이를 활성화하여 성능을 최적화할 수 있다.34

일반적으로 WAN 구간에서는 TCP와 UDP 사이의 트레이드오프를 고려해야 한다. TCP는 신뢰성이 높고 방화벽 통과가 용이하지만 UDP에 비해 지연 시간이 길고 오버헤드가 크다. 반면 UDP는 지연 시간이 짧아 실시간성에 유리하지만, WAN 환경에서는 패킷 손실 가능성이 높다.12 대부분의 라우터는 WAN 구간에 TCP를 사용하는 것을 기본으로 하거나 권장한다.



### 8.4  일반적인 문제 해결



라우터 설정 및 운영 시 발생할 수 있는 일반적인 문제와 해결 방안은 다음과 같다.

- **설정 파일 오류:** YAML이나 XML의 문법 오류는 가장 기본적인 문제이다. 파서가 제공하는 오류 메시지를 통해 쉽게 해결할 수 있다.
- **연결/발견 문제:**
  - WAN 설정 시 방화벽이나 NAT 장비의 포트 포워딩(Port Forwarding) 설정이 잘못된 경우가 많다. 라우터가 수신 대기하는 포트가 외부에서 접근 가능한지 확인해야 한다.
  - RTI Routing Service에서 "non-addressable multicast locator" 오류가 발생하면, 이는 멀티캐스트 설정이 참여자 간에 일치하지 않음을 의미한다. 한쪽에서 멀티캐스트를 비활성화했다면, 연결되는 모든 참여자에서 동일하게 비활성화해야 한다.47
- **QoS 불일치:** 발견은 성공했지만 데이터가 전달되지 않는 가장 흔한 원인이다. 라우터의 내부 Reader/Writer와 양 끝단의 Writer/Reader 간의 QoS 호환성을 `RTI Admin Console`과 같은 도구를 사용해 확인해야 한다.8

더 복잡한 문제에 직면했을 경우, **eProsima DDS-Router의 GitHub 이슈 페이지** 48나 

**RTI의 커뮤니티 포털 및 공식 문서의 알려진 이슈(Known Issues) 목록** 47은 매우 유용한 문제 해결 리소스가 된다.



## 9.  결론: DDS 연결성의 미래



본 보고서는 DDS(Data Distribution Service)의 기본 원리부터 시작하여, 분산 시스템이 확장됨에 따라 발생하는 연결성 문제를 해결하기 위한 DDS 라우터의 필요성과 아키텍처, 그리고 실제 구현 방법을 심도 있게 고찰했다. 마지막으로 시스템 아키텍트를 위한 핵심적인 결론을 종합하고, DDS 연결성의 미래를 조망한다.



### 9.1. 시스템 아키텍트를 위한 핵심 요약



이 보고서의 핵심 논제는 다음과 같이 요약할 수 있다: **표준 DDS는 단일 LAN 환경에서의 강력하고 안정적인 실시간 통신에 매우 뛰어나지만, 복잡한 네트워크 경계를 넘나드는 확장 가능하고, 관리 용이하며, 안전한 분산 시스템을 구축하기 위해서는 DDS 라우터가 필수적인 아키텍처 구성 요소이다.**

라우터는 단순히 네트워크를 연결하는 도구를 넘어, 시스템 전체의 아키텍처를 향상시키는 역할을 수행한다. 라우터를 통해 트래픽을 제어하고, 발견 메커니즘의 범위를 제한하며, 보안을 강화하고, 서로 다른 시스템을 통합할 수 있다.

아키텍트가 어떤 라우터를 선택할지에 대한 결정은 프로젝트의 특성에 따라 명확해진다.

- **eProsima Fast DDS Router**는 오픈 소스 생태계, 특히 ROS 2를 기반으로 하는 시스템, 빠른 프로토타이핑, 비용 효율성이 중요한 프로젝트에 이상적이다. 경량화와 고성능 데이터 전달이라는 명확한 목표를 가지고 있다.
- **RTI Connext Routing Service**는 엔터프라이즈급의 고신뢰성, 고가용성이 요구되는 미션 크리티컬 시스템에 적합하다. 강력한 데이터 변환 기능, 레거시 시스템 통합 능력, 포괄적인 상용 지원 및 툴링은 장기적으로 운영되는 복잡한 시스템의 진화와 유지보수에 결정적인 가치를 제공한다.

궁극적으로 라우터의 도입은 '계층적 데이터버스(Layered Databus)'와 같은 우수한 아키텍처 패턴을 구현할 수 있게 하는 전략적 선택이다.



### 9.1  새로운 동향과 DDS 라우터의 진화하는 역할



DDS 기술과 이를 둘러싼 컴퓨팅 환경은 계속해서 발전하고 있다. 클라우드 네이티브 아키텍처, 엣지 컴퓨팅, 그리고 로보틱스와 자율주행차 같은 자율 시스템의 폭발적인 증가는 DDS 연결성에 새로운 과제와 기회를 제공한다.14

이러한 미래 환경에서 DDS 라우터의 역할은 더욱 중요해질 것이다. 라우터는 더 이상 단순한 네트워크 브리지가 아니다.

- 클라우드와 수많은 엣지 디바이스를 연결하는 **게이트웨이**
- 서로 다른 프로토콜과 데이터 모델을 중재하는 **미디에이터(Mediator)**
- 시스템 경계에서 보안 정책을 강제하는 **보안 강화 지점(Security Enforcement Point)**

으로 진화하고 있다. 시스템이 더욱 분산되고 상호 연결됨에 따라, 지능적인 DDS 라우터는 미래의 복잡한 실시간 분산 시스템을 구축하는 데 있어 그 어떤 때보다 핵심적인 역할을 수행하게 될 것이다.



#### **참고 자료**

1. Data Distribution Service - kjmin's devlog - 티스토리, accessed July 6, 2025, https://leichtjoon.tistory.com/54
2. 멀티 서브네트워크 환경을 위한 DDS Discovery ... - Korea Science, accessed July 6, 2025, https://koreascience.kr/article/CFKO200819463923226.pdf
3. Architectural Overview and OMG Data-Distribution Service - velog, accessed July 6, 2025, https://velog.io/@protocol5/ROS2OMG-Data-Distribution-Service-Architectural-Overview-and-OMG-Data-Distribution-Service-Architectural-Update
4. koreascience.kr, accessed July 6, 2025, https://koreascience.kr/article/JAKO202117457596217.pdf
5. 데이터 분산 서비스 - 위키백과, 우리 모두의 백과사전, accessed July 6, 2025, [https://ko.wikipedia.org/wiki/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EB%B6%84%EC%82%B0_%EC%84%9C%EB%B9%84%EC%8A%A4](https://ko.wikipedia.org/wiki/데이터_분산_서비스)
6. DDS 어플리케이션 개발 및 운영을 위한 플랫폼 설계 방안 - AWS, accessed July 6, 2025, https://journal-home.s3.ap-northeast-2.amazonaws.com/site/2020kics/presentation/0098.pdf
7. DDS(Data Distribution Service)란 무엇인가? - Perbiz, accessed July 6, 2025, [http://perbiz.co.kr/data/DDS(Data%20Distribution%20Service)%EB%9E%80.pdf](http://perbiz.co.kr/data/DDS(Data Distribution Service)란.pdf)
8. RTI Routing Service setup - Stack Overflow, accessed July 6, 2025, https://stackoverflow.com/questions/16207964/rti-routing-service-setup
9. product information - RTI, accessed July 6, 2025, https://www.rti.com/hubfs/docs/RTI_Routing_Service.pdf
10. Introduction to DDS - RTI Community, accessed July 6, 2025, https://d2vkrkwbbxbylk.cloudfront.net/sites/default/files/RTI_DDS_Masterclass_March2010_2day_110slides.pdf
11. eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/
12. 4. WAN Configuration - DDS Router .. documentation - Read the Docs, accessed July 6, 2025, https://eprosima-dds-router.readthedocs.io/en/stable/rst/user_manual/wan_configuration.html
13. Q&A: RTI Introduces Wide Area Routing Service; Scaling To Hundreds of Thousands of Applications - A-Team Insight, accessed July 6, 2025, https://a-teaminsight.com/blog/q-scaling-to-hundreds-of-thousands-of-applications/?brand=ati
14. DDS Router - Read the Docs, accessed July 6, 2025, https://eprosima-dds-router.readthedocs.io/
15. RTI Routing Service - RTI Community, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/6.1.1/doc/manuals/connext_dds_professional/services/routing_service/RTI_RoutingService_UsersManual.pdf
16. rticonnextdds-usecases/DataSubsetWAN/ExampleCode/routing/Routing-TCP-WAN.xml at master - GitHub, accessed July 6, 2025, https://github.com/rticommunity/rticonnextdds-usecases/blob/master/DataSubsetWAN/ExampleCode/routing/Routing-TCP-WAN.xml
17. RTI IS - Routing Service - Real-Time Innovations, accessed July 6, 2025, https://www.rti.com/products/is/routing-service
18. rticommunity/rtiroutingservice-plugins: A collection of plugins for RTI Routing Service - GitHub, accessed July 6, 2025, https://github.com/rticommunity/rtiroutingservice-plugins
19. eProsima/DDS-Router: The DDS Router is an application developed by eProsima that allows, using Fast DDS, to communicate by DDS protocol different networks. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 6, 2025, https://github.com/eProsima/DDS-Router
20. eProsima DDS Router, accessed July 6, 2025, https://www.eprosima.com/middleware/tools/dds-router
21. 3. Configuration - 3.2.0 - DDS Router - Read the Docs, accessed July 6, 2025, https://eprosima-dds-router.readthedocs.io/en/latest/rst/user_manual/configuration.html
22. 3.2.0, accessed July 6, 2025, https://eprosima-dds-router.readthedocs.io/en/latest/
23. Fast-DDS/LICENSE at master - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS/blob/master/LICENSE
24. eProsima/Fast-DDS: The most complete DDS - Proven: Plenty of success cases. Looking for commercial support? Contact info@eprosima.com - GitHub, accessed July 6, 2025, https://github.com/eProsima/Fast-DDS
25. Robotics - eProsima, accessed July 6, 2025, https://www.eprosima.com/robotics
26. eProsima Fast DDS - The middleware powering the ESO ELT, accessed July 6, 2025, https://www.eso.org/sci/meetings/2023/RTC4AO/01_11_martin_losa.pdf
27. 1. Linux installation from sources - 3.2.0 - DDS Router, accessed July 6, 2025, https://eprosima-dds-router.readthedocs.io/en/latest/rst/developer_manual/installation/sources/linux.html
28. 3. Linux installation from sources - 3.2.2 - eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/installation/sources/sources_linux.html
29. 3. Linux installation from sources - Fast DDS 2.6.10 documentation, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/2.6.x/installation/sources/sources_linux.html
30. 1.3. Fast DDS Suite Image - 3.2.2, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/stable/docker/fastdds_suite/fast_dds_suite.html
31. 2. Example of usage - 3.2.0 - DDS Router, accessed July 6, 2025, https://eprosima-dds-router.readthedocs.io/en/latest/rst/getting_started/usage_example.html
32. Green Hills Software's Partners - RTI, accessed July 6, 2025, https://www.ghs.com/partners/rti_partner.html
33. RTI Connext 6 - Cloudfront.net, accessed July 6, 2025, https://d3mm496e6885mw.cloudfront.net/manufacturer_product/5bc5daf83d75cb7178e933de/specsheet/specSheets/original/RTIDatasheet10022WhatsNewConnextV3Web1018.pdf
34. 10. Core Concepts - RTI Routing Service 7.5.0 documentation, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/services/routing_service/core_concepts.html
35. RTI Connext DDS - ROS 2 Documentation, accessed July 6, 2025, https://docs.ros.org/en/iron/Installation/DDS-Implementations/Install-Connext-University-Eval.html
36. RTI Connext - Amazon S3, accessed July 6, 2025, https://s3.amazonaws.com/RTI/RTI_BUNDLES/experimental/rti_prototyper_with_lua/RTI_Connext_Prototyper_GettingStarted.pdf
37. rticonnextdds-usecases/DataSubsetWAN/README.md at master - GitHub, accessed July 6, 2025, https://github.com/rticommunity/rticonnextdds-usecases/blob/master/DataSubsetWAN/README.md
38. 2. Routing Data: Connecting and Scaling Systems - RTI Community, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/services/routing_service/routing_data.html
39. 8. Configuration - RTI Routing Service 7.5.0 documentation - RTI Community, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/services/routing_service/configuration.html
40. user_routing_service_edited.xml - RTI Community, accessed July 6, 2025, https://d2vkrkwbbxbylk.cloudfront.net/sites/default/files/user_routing_service_edited.xml
41. eProsima Fast DDS Monitor, accessed July 6, 2025, https://www.eprosima.com/middleware/tools/fast-dds-monitor
42. 3.12. Secure communications with ROS 2 Router - Vulcanexus 1.0.0 documentation, accessed July 6, 2025, https://docs.vulcanexus.org/en/iron/rst/tutorials/cloud/secure_router/secure_router.html
43. 8. Security - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/security/security.html
44. 6.7. TLS over TCP - 3.2.2 - eProsima Fast DDS, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/latest/fastdds/transport/tcp/tls.html
45. 15.4. Large Data Rates - 3.2.2 - Fast DDS - eProsima, accessed July 6, 2025, https://fast-dds.docs.eprosima.com/en/stable/fastdds/use_cases/large_data/large_data.html
46. eProsima Fast DDS Performance, accessed July 6, 2025, https://www.eprosima.com/developer-resources/performance/eprosima-fast-dds-performance
47. Routing Service Problem | Data Distribution Service (DDS) Community RTI Connext Users, accessed July 6, 2025, https://community.rti.com/forum-topic/routing-service-problem
48. Issues / eProsima/DDS-Router - GitHub, accessed July 6, 2025, https://github.com/eProsima/DDS-Router/issues
49. 14.6. Known Issues - RTI Routing Service 7.5.0 documentation, accessed July 6, 2025, https://community.rti.com/static/documentation/connext-dds/current/doc/manuals/connext_dds_professional/services/routing_service/release_notes/known_issues.html
50. Fast DDS at Full Speed: eProsima Delivers Real-Time Comms for TII Autonomous Vehicles, accessed July 6, 2025, https://www.eprosima.com/news/fast-dds-at-full-speed-eprosima-delivers-real-time-comms-for-tii-autonomous-vehicles
51. RTI in Autonomous Driving - HubSpot, accessed July 6, 2025, https://cdn2.hubspot.net/hubfs/1754418/Collateral_2017/Applications/Autonomous_Driving_20008_v5.pdf


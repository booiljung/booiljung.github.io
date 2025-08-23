# ROS2 Humble Tesseract (Southwest Research Institute의 산업용 모션 플래닝 프레임워크)


현대 산업 자동화는 단순 반복 작업을 넘어, 다품종 소량 생산(high-mix, low-volume) 환경에 대응할 수 있는 지능적이고 유연한 로봇 시스템을 요구한다.1 이러한 변화의 중심에는 로봇이 복잡하고 동적인 환경을 인식하고, 안전하며 효율적인 경로를 스스로 생성하는 모션 플래닝 기술이 자리 잡고 있다. 본 보고서는 Southwest Research Institute (SwRI)가 주도하여 개발한 Tesseract 모션 플래닝 프레임워크에 대한 심층적 고찰을 제공한다. Tesseract는 기존의 범용 플래닝 프레임워크가 가진 한계를 극복하고, 실제 산업 현장의 엄격한 요구사항—견고성, 성능, 정밀도—을 충족시키기 위해 설계되었다. 본 서론에서는 Tesseract의 개발 주체인 SwRI의 역할과 ROS-Industrial 생태계 내에서의 위치, 그리고 Tesseract가 탄생하게 된 기술적 배경과 그 핵심 설계 철학을 탐구한다.


Tesseract를 이해하기 위해서는 먼저 그 개발의 주체인 Southwest Research Institute (SwRI)의 역할과 배경을 파악하는 것이 중요하다. 1947년에 설립된 SwRI는 독립적인 비영리 응용 연구개발 기관으로, 정부 및 산업 고객을 대상으로 광범위한 R&D 서비스를 제공해왔다.3 특히 자동화, 로보틱스, 지능형 시스템 분야에서 수십 년간 축적된 깊이 있는 전문 지식과 경험은 SwRI를 해당 분야의 선도 기관으로 만들었다.3

SwRI는 Robot Operating System (ROS)을 제조 산업에 적용하기 위한 오픈소스 프로젝트인 ROS-Industrial (ROS-I) 컨소시엄의 핵심적인 지원자이자 개발 기여자이다.1 ROS-I의 목표는 전통적으로 폐쇄적인 구조를 가졌던 산업용 로봇 시장에 ROS의 유연성과 상호 운용성을 도입하여, 학계, 정부, 산업계가 공동으로 활용할 수 있는 개방형 솔루션을 제공하는 것이다.1 이러한 생태계 내에서 SwRI는 실제 산업 현장의 문제를 해결하기 위한 핵심 기술과 프레임워크를 개발하고 보급하는 중추적인 역할을 수행하며, Tesseract는 바로 이러한 노력의 가장 중요한 산물 중 하나이다. Tesseract는 순수 학술 연구의 결과물이 아니라, 실제 산업 자동화의 난제들을 해결해 온 기관의 경험과 통찰이 집약된 실용적 프레임워크라는 점에서 그 의의가 크다.


Tesseract의 탄생은 필연의 결과였다. 그 시작은 SwRI가 UC Berkeley에서 개발된 궤적 최적화 기반의 모션 플래너인 TrajOpt를 ROS 생태계에 통합하려는 내부 R&D 프로젝트에서 비롯되었다.8 TrajOpt는 기존의 샘플링 기반 플래너와 달리, 최적화 기법을 통해 매우 제약이 심한 산업 환경 문제에 효과적인 해법을 제시할 잠재력을 가지고 있었다.10

초기 계획은 당시 ROS 생태계의 표준 모션 플래닝 프레임워크였던 MoveIt을 환경 모델로 활용하는 것이었다.8 그러나 통합 과정에서 근본적인 한계에 부딪혔다. TrajOpt는 최적화 계산을 위해 충돌 객체 간의 상세한 거리 정보와 볼록-볼록(convex-to-convex) 충돌 검사 기능을 요구했으나, 당시 MoveIt의 충돌 및 기구학 API는 이러한 정보를 충분히 제공하지 못했다.8 MoveIt의 여러 저장소에 걸친 대대적인 API 변경은 프로젝트의 시간적 제약 하에서 비현실적이었다. 이 기술적 장벽에 직면하여, SwRI는 MoveIt을 수정하는 대신 TrajOpt와 같은 최적화 기반 플래너의 요구사항을 완벽하게 충족하는 새로운 경량 모션 플래닝 환경을 처음부터 구축하기로 결정했다. 이것이 바로 Tesseract의 시작이었다.8 따라서 Tesseract는 단순히 MoveIt의 대안으로 만들어진 것이 아니라, 차세대 플래닝 알고리즘의 잠재력을 산업 현장에서 완전히 발현시키기 위한 필수적인 기반(substrate)으로서 탄생한 것이다.


Tesseract의 핵심 설계 철학은 산업 자동화 애플리케이션에 특화된 '경량성(lightweight)', '모듈성(modularity)', 그리고 '견고성(robustness)'으로 요약할 수 있다.8

첫째, **경량성**을 위해 Tesseract는 외부 라이브러리 의존성을 최소화하고, 주로 Eigen, Boost와 같은 검증된 표준 라이브러리만을 사용한다.13 이를 통해 프레임워크의 복잡성을 낮추고 성능을 극대화하며, 다양한 시스템에 쉽게 이식할 수 있도록 했다.

둘째, **모듈성**은 Tesseract 아키텍처의 핵심이다. 명확하게 정의된 인터페이스, 사용자가 필요에 따라 선택하고 확장할 수 있는 플러그인 구조, 그리고 복잡한 작업을 위한 파이프라인 구성 기능을 제공한다.15 이러한 모듈식 접근은 특정 애플리케이션에 필요한 기능만을 조합하여 최적의 솔루션을 구축할 수 있게 하며, 유지보수와 기능 확장을 용이하게 한다.

셋째, **견고성**은 산업 현장에서의 신뢰성과 직결된다. Tesseract는 결정론적 상태 관리, 정밀한 충돌 검증 등 안정적인 운영을 보장하기 위한 여러 기능을 내장하고 있다. 특히, Tesseract의 핵심 패키지들은 ROS에 독립적(ROS-agnostic)으로 설계되었다.13 이는 특정 미들웨어에 대한 종속성을 제거하여 ROS1, ROS2는 물론 비(非)ROS 환경에서도 Tesseract의 핵심 기능을 활용할 수 있게 하는 전략적 선택이다. 이 설계 철학은 산업 환경의 핵심 요구사항인 고성능, 고신뢰성, 그리고 용이한 시스템 통합을 직접적으로 겨냥하며, Tesseract를 단순한 플래닝 도구를 넘어선 강력한 산업용 소프트웨어 프레임워크로 만든다.


Tesseract의 강력함은 산업 현장의 요구를 반영하여 세심하게 설계된 핵심 아키텍처에서 비롯된다. 이는 단순히 알고리즘의 집합이 아닌, 성능, 안정성, 확장성을 극대화하기 위한 구조적 선택의 결과물이다. 본 장에서는 Tesseract의 심장부 역할을 하는 `Tesseract Environment`와 그 핵심 개념인 '복제(Cloning)' 및 'Command History' 메커니즘을 분석하고, 환경의 연결성을 관리하는 `Scene Graph`와 유연한 `Kinematics` 솔버의 구조를 심층적으로 탐구한다.


`Tesseract Environment`는 Tesseract 프레임워크의 모든 핵심 구성 요소를 총괄하는 중앙 관리자이다.12 이는 로봇의 상태를 해석하는 `State Solver`, 객체 간의 연결 정보를 담은 `Scene Graph`, 충돌을 감지하는 `Contact Managers`, 그리고 환경의 모든 변경 이력을 기록하는 `Command History` 등과 스레드 안전(thread-safe) 방식으로 상호작용한다.12 또한, `Environment`는 씬(scene)이 항상 단일 연결 그래프(single connected graph)를 유지하도록 강제하는 등 시스템의 무결성을 보장하기 위한 제약을 부과하는 역할을 수행한다.12 Tesseract 아키텍처의 차별성은 이 `Environment`가 채택한 두 가지 핵심 개념에서 명확하게 드러난다.


Tesseract의 가장 중요한 아키텍처 결정 중 하나는 스레드 안전성을 확보하고 병렬 처리 성능을 극대화하기 위해 '복제(Cloning)' 메커니즘을 채택한 것이다.12 이는 공유 자원에 대한 접근을 `const` 한정자와 뮤텍스(mutex)로 제어하는 MoveIt의 방식과 근본적인 차이를 보인다.

Tesseract에서는 모션 플래닝과 같은 다중 스레드 작업이 필요할 때, 각 스레드에 `Environment`의 완전한 복사본(deep copy)을 할당한다.12 이 `clone()` 메소드를 통해 생성된 복사본은 원본과 독립적이므로, 각 플래너나 스레드는 복잡한 잠금(locking) 메커니즘 없이도 자신만의 환경 복사본을 자유롭게 수정하며 가상 시나리오를 탐색할 수 있다.12

이러한 접근 방식은 여러 장점을 가진다. 첫째, 교착 상태(deadlock)나 경쟁 상태(race condition)와 같은 공유 메모리 문제의 발생 가능성을 원천적으로 차단하여 코드의 복잡성을 낮추고 안정성을 높인다. 둘째, 각 스레드가 독립적으로 작업을 수행할 수 있어 병렬 플래닝 알고리즘의 성능을 최대한으로 이끌어낼 수 있다. 이는 복잡한 환경에서 여러 가설을 동시에 탐색해야 하는 최적화 기반 플래너에게 특히 유리하다. 복제 메커니즘은 Tesseract가 실시간 장애물 회피와 같은 고성능을 요구하는 산업 애플리케이션에서 안정적으로 작동할 수 있도록 하는 핵심 기반 기술이다.


`Command History`는 Tesseract를 차별화하는 또 다른 독창적인 기능으로, 시스템의 상태를 결정론적으로 추적하고 완벽하게 재현할 수 있는 강력한 메커니즘을 제공한다.12 이는 소프트웨어 디자인 패턴 중 하나인 '커맨드 패턴(Command Pattern)'을 활용하여 `Environment`에 가해지는 모든 변경 사항(예: 링크 추가, 조인트 위치 변경, 충돌 객체 제거 등)을 순차적인 명령어 객체로 기록한다.12

이러한 이력 관리는 산업 현장에서 매우 중요한 가치를 지닌다.

- **디버깅 및 분석**: 만약 시스템에 오류가 발생했을 경우, `Command History`를 재생하여 오류가 발생한 시점의 환경 상태를 100% 정확하게 복원할 수 있다. 이를 통해 문제의 원인을 신속하고 정확하게 파악할 수 있다.
- **상태 복제 및 분산**: `Command History`는 환경의 전체 상태를 담고 있으므로, 이를 다른 노드나 시뮬레이터로 전송하여 완전히 동일한 환경을 손쉽게 구축할 수 있다. 이는 분산 시스템 환경에서 일관된 월드 모델을 유지하는 데 필수적이다.
- **URDF/SRDF 의존성 탈피**: 로봇 모델 파일(URDF/SRDF)은 초기 환경 구성 시에만 필요하다. 한번 파싱된 후에는 모든 정보가 내부 데이터 구조와 `Command History`에 저장되므로, 런타임 중에는 더 이상 원본 파일에 접근할 필요가 없다.12 이는 시스템의 유연성을 높이고 배포를 간소화한다.

`tesseract_command_language` 패키지는 이러한 명령어들을 산업용 티치 펜던트에서 사용하는 것과 유사한 직관적인 언어로 표현하고 실행할 수 있도록 지원한다.13

`Command History`는 Tesseract가 단순한 플래닝 라이브러리를 넘어, 상태 관리가 중요한 장시간 운영 프로덕션 환경에 적합한 견고한 시스템 프레임워크임을 보여주는 핵심 기능이다.


`Tesseract Scene Graph`는 로봇과 작업 환경을 구성하는 모든 객체(링크와 조인트)들의 기구학적 연결 관계를 관리하는 핵심 데이터 구조이다.13 이 컴포넌트의 가장 큰 특징은 검증된 고성능 C++ 라이브러리인 `boost::graph`를 직접 상속하여 구현되었다는 점이다.12

이러한 설계는 개발자에게 여러 이점을 제공한다. `boost::graph`가 제공하는 깊이 우선 탐색(DFS), 너비 우선 탐색(BFS), 최단 경로 찾기(Dijkstra 알고리즘 등)와 같은 강력하고 효율적인 그래프 탐색 및 분석 알고리즘들을 Tesseract의 씬 그래프에 직접 적용할 수 있다.12 예를 들어, 로봇의 베이스 링크에서 엔드 이펙터까지의 기구학적 체인을 찾거나, 특정 조인트에 연결된 모든 하위 링크들을 순회하는 등의 작업을 표준화된 Boost API를 통해 손쉽게 수행할 수 있다.

`Scene Graph`는 링크와 조인트의 추가, 제거, 수정과 같은 기본적인 관리 기능 외에도, 특정 링크 쌍 간의 충돌 검사를 비활성화하는 허용 충돌 매트릭스(Allowed Collision Matrix, ACM) 정보도 관리한다.18 이는 Tesseract가 환경의 물리적 구조와 논리적 제약 조건을 통합적으로 관리할 수 있게 해주는 기반이 된다.


`Tesseract Kinematics` 패키지는 로봇의 순기구학(Forward Kinematics) 및 역기구학(Inverse Kinematics) 계산을 위한 공통 인터페이스와 구현체들을 제공한다.13 Tesseract의 기구학 설계에서 주목할 만한 점은 '환경 비인식(environment-aware-less)' 원칙이다.12 즉, 각각의 기구학 솔버는 충돌 정보와 같은 외부 환경을 고려하지 않고, 순수하게 주어진 조인트 값으로부터 엔드 이펙터의 자세를 계산하거나(FK), 주어진 엔드 이펙터 자세에 해당하는 조인트 값을 찾는(IK) 계산에만 집중한다. 이는 각 솔버의 구현을 단순화하고 독립성을 높여, 새로운 기구학 알고리즘을 쉽게 통합하고 테스트할 수 있게 한다.

Tesseract는 플러그인 기반의 아키텍처를 채택하여 높은 유연성과 확장성을 자랑한다. 모든 기구학 솔버는 YAML 파일에 정의된 설정을 통해 동적으로 로드된다.12 이를 통해 사용자는 애플리케이션에 가장 적합한 솔버를 선택하거나 직접 개발한 커스텀 솔버를 손쉽게 추가할 수 있다.

또한 Tesseract는 산업 현장에서 널리 사용되는 다양한 솔루션에 대한 특화된 지원을 내장하고 있다 12:

- **IKFast**: 자동 생성된 고속 해석적 IK 솔버.
- **OPW Kinematics**: 산업용 로봇에서 흔히 발견되는 평행 손목 구조에 최적화된 솔버.
- **Universal Robots (UR)**: UR 로봇 시리즈를 위한 전용 솔버.
- **복합 시스템**: 로봇이 포지셔너(회전 테이블 등) 위에 있거나, 외부 포지셔너와 함께 작동하는 복잡한 기구학적 체인에 대한 지원.

이러한 유연한 플러그인 구조와 산업 표준에 대한 폭넓은 지원은 Tesseract가 다양한 종류의 로봇과 복잡한 작업 셀 구성에 효과적으로 대응할 수 있게 하는 핵심적인 장점이다.

결론적으로, Tesseract의 핵심 아키텍처는 **분리되고(decoupled), 결정론적이며(deterministic), 분산 가능한(distributable)** 시스템을 지향하는 설계 철학을 명확히 보여준다. **복제(Cloning)** 메커니즘은 플래닝 스레드를 공유 전역 상태로부터 분리하여 병렬 알고리즘 설계를 단순화한다. **Command History**는 환경 상태를 결정론적으로 기록하고 완벽하게 복제하는 방법을 제공하여 디버깅과 분산 시스템에서의 상태 일관성을 보장한다. 마지막으로, **ROS-Agnostic Core**는 핵심 플래닝 로직을 미들웨어로부터 분리하여 Tesseract의 이식성과 미래 확장성을 담보한다. 이 세 가지 요소의 조합은 Tesseract를 단순한 알고리즘 라이브러리를 넘어, 상태 관리, 결정론, 통합 유연성이 필수적인 실제 산업 현장을 위해 설계된 견고한 시스템 엔지니어링의 결과물로 만든다.


산업용 로봇 시스템의 안전성과 신뢰성은 무엇보다 정교하고 완벽한 충돌 검사 능력에 달려있다. 작업물이나 고가의 장비에 대한 경미한 충돌조차 치명적인 손실로 이어질 수 있는 산업 현장에서, '거의' 충돌하지 않는 경로는 용납될 수 없다. Tesseract는 이러한 산업계의 엄격한 요구를 충족시키기 위해, 기존 프레임워크들을 뛰어넘는 진보된 충돌 검사 아키텍처를 갖추고 있다. 본 장에서는 Tesseract의 충돌 검사 패키지 구조를 분석하고, 특히 이산적 검사의 한계를 극복하는 연속 충돌 검사(Continuous Collision Checking)의 원리와 그 중요성을 깊이 있게 탐구한다.


`tesseract_collision` 패키지는 Tesseract의 충돌 검사 기능을 전담하는 핵심 컴포넌트이다.13 이 패키지는 충돌 검사를 위한 공통 인터페이스를 제공하며, 백엔드로는 널리 사용되는 물리 엔진인 Bullet Physics와 FCL(Flexible Collision Library)을 모두 지원하여 유연성을 확보했다.13

`tesseract_collision`의 중요한 설계 원칙 중 하나는 '기구학적 비인식(kinematically-unaware)'이다.12 즉, 이 패키지는 로봇의 링크들이 조인트로 어떻게 연결되어 있는지와 같은 기구학적 구조에 대해서는 전혀 알지 못한다. 단지 공간상에 주어진 위치와 자세(pose)를 가진 여러 기하학적 객체(geometric shapes)들 간의 간섭 여부만을 순수하게 계산한다.18 이러한 책임의 분리는 충돌 검사 로직을 단순화하고 독립성을 높여, 다양한 종류의 객체와 시나리오에 대해 일관된 검사 기능을 제공할 수 있게 한다.


Tesseract가 다른 모션 플래닝 프레임워크와 차별화되는 가장 중요한 장점 중 하나는 연속 충돌 검사(CCC)를 핵심 기능으로 완벽하게 지원한다는 것이다.8

전통적인 이산 충돌 검사(Discrete Collision Checking)는 로봇의 궤적을 구성하는 일련의 특정 지점(waypoint)들에서만 정적인 충돌 여부를 확인한다. 이 방식은 계산이 빠르다는 장점이 있지만, 치명적인 맹점을 가지고 있다. 만약 두 웨이포인트 사이의 거리가 환경 내의 장애물 크기보다 크다면, 로봇은 웨이포인트 자체에서는 충돌하지 않지만 그 사이를 이동하는 과정에서 장애물과 충돌할 수 있다.8 이는 특히 로봇이 고속으로 움직이거나 복잡하고 좁은 공간을 통과할 때 심각한 안전 문제로 이어진다.

연속 충돌 검사는 이러한 문제를 해결한다. 이 기술은 두 웨이포인트 사이에서 로봇의 움직임이 만들어내는 전체 부피, 즉 '스윕 볼륨(swept volume)'을 고려한다. Tesseract의 `BulletCastBVHManager`와 같은 구현체는 두 시점의 링크 위치 사이에 '캐스팅된 볼록 껍질(casted convex hull)'을 생성하고, 이 4차원적(공간+시간) 부피가 다른 장애물과 교차하는지를 검사하는 방식으로 CCC를 수행한다.8

이러한 연속 충돌 검사는 계산 비용이 더 높지만, 궤적의 전 구간에 걸쳐 충돌 안전성을 수학적으로 보장해준다. 이는 용접, 연삭, 도색과 같이 정밀한 경로 제어가 필수적이고, 단 한 번의 충돌도 용납되지 않는 고부가가치 산업 공정에서 자율 로봇 시스템을 안전하게 운용하기 위한 필수 전제 조건이다.19


Tesseract는 단순히 충돌 여부만을 알려주는 것을 넘어, 어떤 종류의 플래너를 사용하느냐에 따라 최적화된 충돌 정보를 제공할 수 있도록 다양한 접촉 테스트 유형을 지원한다.12 이는 Tesseract가 다양한 플래닝 알고리즘의 내부 작동 방식에 대한 깊은 이해를 바탕으로 설계되었음을 보여준다.

- `ContactTestType::FIRST`: 이 모드에서는 충돌 검사기가 가장 먼저 발견된 단 하나의 충돌 쌍에 대한 정보만 반환하고 즉시 계산을 중단한다. 경로의 유효성(valid/invalid) 여부만 빠르게 판단하면 되는 OMPL과 같은 샘플링 기반 플래너에 매우 효율적이다. 불필요한 추가 계산을 생략하여 플래닝 속도를 크게 향상시킬 수 있다.
- `ContactTestType::CLOSEST`: 이 모드는 각 객체 쌍에 대해 가장 가까운 지점의 접촉 정보(거리, 방향, 위치 등)만을 반환한다. 이는 TrajOpt와 같은 최적화 기반 플래ナー에게 필수적인 정보를 제공한다. 플래너는 이 '가장 위험한' 충돌 지점 정보를 바탕으로 충돌을 회피하는 방향으로 궤적을 수정하기 위한 그래디언트(gradient)를 계산할 수 있다.
- `ContactTestType::ALL`: 이 모드는 발견된 모든 충돌 지점 정보를 반환한다. `CLOSEST` 모드보다 더 포괄적인 충돌 상황 정보를 제공하며, 복잡한 충돌 환경을 더 정밀하게 분석하고 회피 전략을 세워야 하는 경우에 TrajOpt 플래너가 활용할 수 있다.

이처럼 Tesseract는 플래너의 요구에 맞춰 충돌 정보의 '양'과 '질'을 조절할 수 있는 기능을 제공함으로써, 전체 모션 플래닝 파이프라인의 성능을 극대화한다.

Tesseract의 정교한 충돌 검사 기능, 특히 연속 충돌 검사와 세분화된 접촉 정보 제공 능력은 임의의 기능이 아니라, TrajOpt와 같은 최적화 기반 플래너를 효과적으로 지원하기 위한 필연적인 설계 결과이다. 최적화 기반 플래너는 충돌을 '벽'으로 인식하는 것이 아니라, 거리에 따라 비용이 점진적으로 증가하는 '언덕'으로 인식해야 효율적으로 작동한다. 즉, 충돌 여부(이진 정보)뿐만 아니라, '얼마나 가까운지'와 '어느 방향으로 피해야 하는지'(연속적인 그래디언트 정보)를 알아야 한다. Tesseract의 `CLOSEST` 접촉 테스트와 연속 충돌 검사는 바로 이러한 풍부하고 연속적인 비용 지형(cost landscape)을 생성하는 데 필요한 핵심 데이터를 제공한다.8 따라서 Tesseract의 진보된 충돌 검사 아키텍처는 TrajOpt와의 강력한 시너지를 창출하는 근본적인 전제 조건이라 할 수 있다.


Tesseract의 진정한 가치는 TrajOpt와 같은 최적화 기반 모션 플래너와 결합될 때 극대화된다. 샘플링 기반 플래너가 무작위 탐색을 통해 해를 찾는다면, 최적화 기반 플래너는 수학적 최적화 기법을 통해 초기 경로를 점진적으로 개선하여 제약 조건을 만족하는 최적의 해를 찾아낸다. 본 장에서는 TrajOpt의 핵심 원리인 순차적 볼록 최적화 기법을 설명하고, 그 수학적 정식화를 통해 비용 함수와 충돌 제약이 어떻게 다루어지는지 분석한다.


TrajOpt는 UC Berkeley에서 개발된 궤적 최적화 프레임워크로, 로봇의 경로를 지역 최적화(local optimization)를 통해 생성한다.8 TrajOpt의 접근 방식은 직관적이다. 먼저 시작점과 목표점을 잇는 단순하고 미숙한(naïve) 초기 궤적(예: 조인트 공간에서의 직선 경로)으로 시작한다. 이 초기 궤적은 장애물과 충돌하거나 다른 제약 조건을 위반할 수 있다.10 그 후, 이 궤적을 반복적으로 수정하여 모든 제약 조건을 만족시키고 비용 함수를 최소화하는 최종 궤적을 찾아낸다.

이 과정의 핵심에는 '순차적 볼록 최적화(Sequential Convex Optimization, SCO)'라는 강력한 기법이 있다.10 모션 플래닝 문제는 본질적으로 비볼록(non-convex) 문제이기 때문에 전역 최적해(global optimum)를 찾기가 매우 어렵다. SCO는 이 비볼록 문제를 직접 푸는 대신, 현재 궤적 주변에서 문제를 볼록(convex) 문제로 근사(approximation)하여 푸는 과정을 반복한다. 볼록 문제는 효율적으로 지역 최적해를 찾을 수 있다는 장점이 있다. 즉, TrajOpt는 풀기 쉬운 일련의 볼록 최적화 문제들을 순차적으로 풀어가면서 원래의 어려운 비볼록 문제의 해에 점근적으로 수렴해 나간다.22

충돌 회피는 이 과정에서 페널티(penalty) 항으로 다루어진다. 힌지 손실(hinge loss) 함수를 사용하여 충돌이 발생한 정도에 비례하는 비용을 부과하고, 외부 루프에서 이 페널티 계수를 점차 증가시킨다. 이 과정을 통해 최적화기는 충돌을 회피하는 방향으로 궤적을 '밀어내어' 결국 충돌 없는 경로를 찾게 된다.10


TrajOpt에서 모션 플래닝 문제는 이산화된 시간 스텝 T개로 구성된 궤적 $X = {q_1,..., q_T}$를 찾는 최적화 문제로 정식화된다. 여기서 qt는 시간 스텝 t에서의 로봇 관절 각도(configuration) 벡터이다. 목표는 전체 비용 함수를 최소화하는 궤적 X를 찾는 것이다.


전체 최적화 문제는 다음과 같이 표현될 수 있다 8:
$$
\min_{X} \sum_{t=1}^{T} f_t(q_t) + \sum_{i=1}^{N_{cnt}} g_i(X)
$$
여기서 각 항의 의미는 다음과 같다:

- $\sum_{t=1}^{T} f_t(q_t)$: 궤적의 각 지점(waypoint)에 개별적으로 적용되는 비용의 합이다. 대표적인 예로 조인트 속도, 가속도, 저크(jerk)를 최소화하여 부드러운 움직임을 유도하는 비용이 있다. 이는 궤적의 평활도(smoothness)를 결정한다.
- $\sum_{i=1}^{N_{cnt}} g_i(X)$: 궤적 전체에 걸쳐 적용되는 비용 또는 제약 조건의 합이다. 가장 중요한 예가 바로 충돌 회피 비용이다. 이 외에도 엔드 이펙터의 자세(orientation)를 특정 방향으로 유지하는 것과 같은 공정 제약 조건(process constraint)도 이 항에 포함될 수 있다.


TrajOpt는 충돌 제약을 엄격한 제약(hard constraint)이 아닌, 비용 함수에 포함되는 페널티 항(soft constraint)으로 다루는 것이 특징이다. 로봇의 몸체 A와 장애물 B 사이의 부호 있는 거리(signed distance)를 $d(q)$라고 하자. 여기서 d(q)>0은 두 물체가 떨어져 있음을, d(q)≤0은 충돌 상태임을 의미한다. 안전을 위해 $\epsilon_{safe}$라는 최소 안전 마진을 설정하면, 충돌 회피 제약은 $d(q) \ge \epsilon_{safe}$로 표현된다.

TrajOpt는 이 제약을 힌지 손실 함수를 사용하여 비용 함수 $g_{coll}$로 변환한다 10:
$$
g_{coll}(q) = w \cdot \max(0, \epsilon_{safe} - d(q))
$$
이 수식의 의미는 다음과 같다.

- 만약 로봇이 안전 마진보다 멀리 떨어져 있다면(d(q)≥ϵsafe), 괄호 안의 값은 음수가 되므로 max 함수에 의해 비용은 0이 된다.
- 만약 로봇이 안전 마진을 침범했다면(d(q)<ϵsafe), 침범한 깊이에 비례하는 양수의 비용이 발생한다.
- w는 이 비용의 가중치, 즉 페널티 계수이다. 최적화 과정의 외부 루프에서 이 w 값을 점진적으로 증가시키면서, 최적화기가 충돌을 더욱 '강력하게' 회피하도록 유도한다.

이러한 정식화는 최적화기가 일시적으로 충돌 상태에 있는 궤적을 탐색할 수 있는 유연성을 제공하면서도, 최종적으로는 충돌이 없는 실행 가능한 해로 수렴하도록 이끄는 강력한 메커니즘이다.


Tesseract는 TrajOpt와 같은 최적화 기반 플래너가 최상의 성능을 발휘할 수 있는 이상적인 환경을 제공하기 위해 설계되었다.8 `tesseract_planning` 패키지는 Tesseract 환경과 TrajOpt, OMPL 등 다양한 외부 플래너를 연결하는 인터페이스 브리지 역할을 한다.8

Tesseract와 TrajOpt의 시너지는 다음과 같은 지점에서 발생한다.

- **상세한 충돌 정보**: TrajOpt가 충돌 비용 $g_{coll}(q)$와 그 그래디언트를 계산하기 위해서는 부호 있는 거리 $d(q)$에 대한 정확한 정보가 필수적이다. Tesseract의 충돌 검사기는 `CLOSEST` 모드를 통해 이 정보를 효율적으로 제공한다.8
- **연속 충돌 검증**: TrajOpt가 생성한 궤적은 이산적인 웨이포인트들로 구성된다. Tesseract의 연속 충돌 검사 기능은 이 웨이포인트들 사이의 구간까지 안전함을 보장하여, 최종 궤적의 신뢰도를 비약적으로 향상시킨다.8

이러한 긴밀한 통합을 통해, Tesseract와 TrajOpt는 단순한 '경로 탐색(motion planning)'을 넘어, 복잡한 산업 공정의 요구사항까지 만족시키는 '공정 계획(process planning)'을 수행할 수 있는 강력한 조합을 이룬다. 예를 들어, 용접 공정에서 '용접 토치의 각도를 항상 표면에 수직으로 유지'하고 'TCP 속도를 일정하게 유지'하는 것과 같은 제약 조건들을 추가적인 비용 함수로 최적화 문제에 포함시킬 수 있다.7 이는 충돌 회피, 경로 평활도, 그리고 공정 제약 조건이라는 다중 목표를 동시에 최적화하는 고차원적인 문제 해결 능력을 의미하며, Tesseract가 산업 현장에서 가지는 독보적인 경쟁력의 원천이다.


Tesseract의 핵심 라이브러리는 ROS에 독립적으로 설계되었지만, 그 진정한 실용성은 ROS2 생태계와의 원활한 통합을 통해 발현된다. 특히 장기 지원(LTS) 버전인 ROS2 Humble Hawksbill은 안정성과 성숙도를 바탕으로 산업계에서 널리 채택되고 있으며, Tesseract는 `tesseract_ros2` 패키지를 통해 Humble 환경을 완벽하게 지원한다.23 본 장에서는 `tesseract_ros2` 패키지의 내부 구조를 분석하고, ROS2 Humble 환경에서의 구체적인 빌드 및 실행 절차를 설명하며, 실제 활용법을 탐구하기 위한 예제 패키지를 살펴본다.


`tesseract_ros2` 저장소는 Tesseract의 강력한 핵심 기능을 ROS2의 표준 통신 및 도구 체계와 연결하는 교량 역할을 하는 패키지들의 집합이다.23 각 패키지는 명확한 역할을 수행하며, 이들의 유기적인 조합을 통해 Tesseract는 ROS2 기반의 로봇 애플리케이션에 통합된다.

**표 1: `tesseract_ros2` 주요 패키지 기능 요약**

| 패키지명                    | 핵심 기능                                                    | 생태계 내 주요 역할                                          |
| --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `tesseract_msgs`            | TesseractState, PlanningRequest 등 Tesseract 전용 ROS2 메시지 정의 | 노드 간 통신을 위한 표준화된 데이터 구조 제공                |
| `tesseract_rosutils`        | 네이티브 Tesseract C++ 타입과 ROS2 메시지 타입 간 변환 유틸리티 함수 제공 | 핵심 라이브러리와 ROS 노드 간의 데이터 상호 운용성을 위한 필수적인 '접착제' 역할 |
| `tesseract_rviz`            | 환경, 로봇 궤적, 충돌 지오메트리 등을 시각화하기 위한 RViz 디스플레이 플러그인 | 개발자를 위한 핵심적인 디버깅 및 시각적 검증 도구            |
| `tesseract_monitoring`      | 주(main) 씬을 관리하고, 업데이트를 처리하며, 현재 상태를 퍼블리시하는 Environment Monitor 노드 | ROS2 생태계 내에서 로봇 월드 모델의 '단일 진실 공급원(single source of truth)' 역할 (MoveIt의 PlanningSceneMonitor와 유사) |
| `tesseract_planning_server` | 비동기 모션 플래닝 요청을 처리하기 위한 ROS2 액션 서버(Action Server) 제공 | 클라이언트 애플리케이션이 블로킹(blocking) 없이 계획을 요청할 수 있는 표준화되고 견고한 인터페이스 제공 |
| `tesseract_ros_examples`    | 다양한 플래닝 기능을 시연하는 런치 파일 및 노드 포함         | 신규 개발자를 위한 주요 학습 자료 및 시작점 역할             |

이 패키지 구조에서 주목할 점은 MoveIt과의 유사성이다. 특히 `tesseract_monitoring` 패키지는 MoveIt의 `PlanningSceneMonitor`와 개념적으로 매우 유사한 역할을 수행한다.8 이는 의도적인 설계 결정으로, 이미 MoveIt에 익숙한 방대한 ROS 개발자 커뮤니티가 Tesseract로 전환하거나 Tesseract를 실험할 때 겪는 학습 곡선을 크게 낮추는 효과가 있다. 개발자들은 기존의 씬 관리 방식에 대한 지식을 활용하면서, Tesseract가 제공하는 연속 충돌 검사나 최적화 기반 플래너와 같은 강력한 백엔드의 이점을 누릴 수 있다. 이는 새로운 기술의 채택을 촉진하기 위한 매우 실용적인 접근 방식이다.


ROS2 Humble 환경에서 Tesseract를 소스 코드로부터 빌드하는 과정은 표준적인 `colcon` 빌드 시스템을 따른다. `tesseract_ros2` GitHub 저장소는 이 과정을 상세히 안내하고 있다.23

빌드 절차는 다음과 같이 요약된다:

1. **`colcon` 워크스페이스 생성**: `mkdir -p tesseract_ws/src`와 같이 새로운 워크스페이스 디렉토리를 생성한다.
2. **소스 코드 클론**: 생성된 `src` 디렉토리로 이동하여 `git clone https://github.com/tesseract-robotics/tesseract_ros2.git` 명령어로 메인 저장소를 클론한다.
3. **의존성 설치**: Tesseract는 여러 개의 개별 저장소로 구성되어 있으므로, 의존 관계에 있는 저장소들을 효율적으로 관리해야 한다.
   - `vcs import < src/tesseract_ros2/dependencies.repos`: 이 명령어는 `dependencies.repos` 파일에 명시된 모든 의존 저장소(예: `tesseract`, `tesseract_planning` 등)를 `src` 디렉토리로 자동으로 클론한다. 이는 복잡한 의존성 관리를 매우 간소화하는 모범 사례이다.23
   - `rosdep install --from-paths src -iry`: `rosdep` 도구를 사용하여 패키지들의 `package.xml` 파일에 명시된 모든 시스템 라이브러리(예: `libbullet-dev`) 의존성을 자동으로 설치한다.
4. **빌드 실행**: 워크스페이스의 최상위 디렉토리로 이동하여 `colcon build` 명령어를 실행한다.
   - `colcon build --symlink-install --cmake-args -DTESSERACT_BUILD_FCL=OFF -DBUILD_RENDERING=OFF -DBUILD_STUDIO=OFF`: 필요에 따라 특정 CMake 인자를 전달하여 FCL 지원이나 시각화 관련 기능 등 선택적 컴포넌트를 비활성화하고 빌드 시간을 단축할 수 있다.23

또한, 개발 및 배포의 일관성과 편의성을 높이기 위해 공식 Docker 이미지가 제공된다.15 Docker를 사용하면 복잡한 의존성 설치 과정을 생략하고, 사전 구성된 개발 환경에서 즉시 Tesseract를 사용하거나 배포할 수 있다.


`tesseract_ros_examples` 패키지는 Tesseract의 기능을 실제로 어떻게 사용하는지 보여주는 가장 좋은 학습 자료이다.23 이 패키지에는 모션 플래닝과 충돌 검사를 시연하는 다양한 예제 코드와 런치 파일이 포함되어 있다.

워크스페이스 빌드가 완료되고 `source install/setup.bash` 명령어를 통해 환경이 설정되면, ROS2의 표준 실행 방식인 `ros2 launch` 또는 `ros2 run` 명령어를 사용하여 예제를 실행할 수 있다. 예를 들어, 특정 플래닝 시나리오를 시각화와 함께 실행하는 런치 파일이 있다면 다음과 같은 명령어로 실행할 수 있다 (예시 명령어):

```sh
$ ros2 launch tesseract_ros_examples freespace_planning_example.launch.py
```

이러한 예제들을 실행하고 코드를 분석함으로써 개발자는 다음과 같은 핵심적인 활용법을 익힐 수 있다.

- 프로그램 코드에서 Tesseract 환경을 설정하고 로봇 모델을 로드하는 방법.
- `tesseract_planning_server` 액션 서버에 플래닝 요청을 보내고 결과를 수신하는 방법.
- 다양한 플래너(TrajOpt, OMPL 등)를 사용하여 자유 공간(freespace) 및 데카르트(Cartesian) 경로를 생성하는 방법.
- RViz 플러그인을 사용하여 Tesseract 환경과 생성된 궤적을 시각적으로 확인하고 디버깅하는 방법.

`tesseract_ros_examples`는 Tesseract의 강력한 기능과 ROS2 생태계의 도구들이 어떻게 결합되어 실제 문제를 해결하는지를 보여주는 실질적인 청사진을 제공한다.


Tesseract의 기술적 우수성은 이론에 머무르지 않고, 실제 산업 현장의 복잡하고 까다로운 문제들을 해결하는 데서 그 가치가 증명된다. SwRI와 ROS-Industrial 컨소시엄은 Tesseract를 핵심 엔진으로 사용하여 다양한 산업 자동화 솔루션을 성공적으로 개발 및 배포해왔다. 본 장에서는 Tesseract의 기능이 어떻게 실질적인 가치로 전환되는지를 보여주는 대표적인 사례들을 연구한다.


'Robotic Blending Milestone 5'는 Tesseract의 산업 적용 능력을 집약적으로 보여주는 대표적인 프로젝트이다.15 미국철강주조협회(SFSA)의 후원으로 진행된 이 프로젝트의 목표는 주물(casting) 부품 표면에 남은 불규칙한 형상(riser, gate)을 로봇이 자율적으로 연삭(grinding)하여 매끄럽게 마감하는 것이었다.24

이 작업은 전통적인 로봇 프로그래밍 방식으로는 자동화하기 매우 어려운 대표적인 '다품종 소량 생산' 시나리오이다. 주물 부품은 생산될 때마다 그 형상과 크기에 미세한 편차가 있어, 미리 정해진 경로를 반복하는 방식으로는 정밀한 작업이 불가능하다. 프로젝트 팀은 'Scan-N-Plan' 프레임워크를 적용하여 이 문제를 해결했다. 작업 흐름은 다음과 같다.

1. **Scan**: 3D 스캐너로 작업할 주물 부품의 정밀한 3차원 모델을 획득한다.
2. **Plan**: 획득한 3D 모델을 기반으로 연삭이 필요한 영역에 대한 최적의 공구 경로(tool path)를 생성하고, Tesseract를 이용하여 이 경로를 따라 움직이는 로봇의 충돌 없는 전체 동작 궤적(motion plan)을 계획한다.
3. **Execute**: 생성된 궤적에 따라 로봇이 실제로 연삭 작업을 수행한다.

이 프로젝트의 성공은 Tesseract의 여러 핵심 기능 덕분이었다.

- **유연한 충돌 모델**: 스캔된 메시(mesh) 데이터를 그대로 사용하거나, 계산 효율을 위해 볼록 껍질(convex hull) 또는 옥트리(octree) 형태로 변환하여 충돌 환경을 구성할 수 있는 유연성을 제공했다.15
- **공정 제약 플래닝**: 단순히 충돌을 피하는 것을 넘어, 연삭 작업의 품질을 보장하기 위해 TCP(Tool Center Point)의 속도를 일정하게 유지하는 시간 파라미터화(time parameterization) 알고리즘이 적용되었다.15
- **모듈성 및 이식성**: 이 시스템은 서로 다른 로봇(Kawasaki, Yaskawa)과 하드웨어 구성을 가진 3개의 다른 장소(SwRI, Iowa State University, Fisher Cast Steel)에 성공적으로 배포되어, Tesseract 기반 솔루션의 높은 모듈성과 이식성을 입증했다.15

이 프로젝트는 위험하고 힘든 수작업 연삭 공정을 자동화하여 작업 환경을 개선하고, 일관된 품질을 확보하며, 상당한 투자수익률(ROI)을 달성할 수 있는 잠재력을 보여주었다.25 이는 Tesseract가 어떻게 고도의 유연성이 요구되는 제조업의 난제를 해결하는지를 보여주는 구체적인 증거이다.


Tesseract의 능력은 단일 로봇 팔에 국한되지 않는다. SwRI는 군수품 장전과 같이 크고 무거운 물체를 다루는 작업을 위해 두 개의 로봇 팔이 협력하는 시스템에 Tesseract를 성공적으로 적용했다.27

이 시나리오에서 두 개의 로봇 팔은 하나의 거대한 객체를 함께 잡고 정밀하게 이동 및 삽입해야 한다. 이는 두 팔을 합쳐 12자유도(DOF)에 달하는 매우 복잡한 기구학적 체인을 제어하는 문제이다. 일반적인 경로 계획기는 이처럼 높은 자유도를 가진 시스템을 효율적으로 다루기 어렵다.27

Tesseract는 이러한 고차원 문제를 해결하기 위한 유연한 프레임워크를 제공했다. 연구팀은 두 로봇 팔 사이의 고정된 거리 유지와 같은 협조 작업을 위한 새로운 제약 조건을 Tesseract 플래닝 프레임워크 내에 추가하여, 12자유도 전체 시스템에 대한 충돌 없는 자유 공간 궤적을 성공적으로 생성했다.27 이 사례는 Tesseract가 표준적인 6-7축 로봇을 넘어, 다중 로봇 시스템이나 모바일 매니퓰레이터와 같은 복잡하고 비정형적인 기구학 구조로 쉽게 확장될 수 있는 뛰어난 확장성을 가지고 있음을 보여준다.


첨단 로봇 기술이 현장에 보급되는 데 가장 큰 장벽 중 하나는 사용의 어려움이다. 복잡한 모션 플래닝을 위해 C++ 코드를 작성하는 것은 로봇 프로그래밍 전문가가 아닌 일반 제조 엔지니어에게는 매우 높은 진입 장벽이다. SwRI는 이 문제를 해결하기 위해 SWORD (SwRI Workbench for Offline Robotics Development)를 개발했다.1

SWORD는 널리 사용되는 오픈소스 CAD 소프트웨어인 FreeCAD의 플러그인 형태로 제공되는 그래픽 기반 툴킷이다.28 사용자는 익숙한 CAD 환경 내에서 다음과 같은 작업을 직관적으로 수행할 수 있다.

- 로봇, 고정 장치(fixture), 엔드 이펙터 등을 포함하는 가상의 작업 셀(workcell)을 모델링한다.
- Tesseract가 지원하는 다양한 경로 계획기(TrajOpt, OMPL 등)를 사용하여 그래픽 인터페이스 상에서 모션 플랜을 생성한다.
- 애플리케이션에 특화된 커스텀 플래닝 파이프라인을 구성하고, 생성된 궤적을 시뮬레이션하며 충돌을 예측하고 회피한다.29

SWORD는 Tesseract의 강력한 플래닝 능력을 코드 한 줄 작성하지 않고도 활용할 수 있게 해주는 '통역사' 역할을 한다. 이는 제품을 설계하는 CAD 엔지니어와 로봇을 프로그래밍하는 로봇 전문가 사이의 간극을 메워, 첨단 모션 플래닝 기술의 민주화를 이끌고 있다.1

이러한 사례들은 Tesseract가 단순히 기술적으로 진보한 프레임워크일 뿐만 아니라, '다품종 소량 생산'이라는 현대 제조업의 핵심 패러다임을 실현하는 핵심 동력임을 명확히 보여준다. 전통적인 로봇이 동일한 작업을 수백만 번 반복하는 데 최적화되어 있다면, Tesseract 기반의 Scan-N-Plan 워크플로우는 매번 다른 부품에 대해 유연하게 대응할 수 있는 능력을 제공한다. CAD 모델 없이 센서 데이터로부터 직접 작업 계획을 수립하고, 복잡한 공정 제약 조건을 만족하는 안전한 경로를 자율적으로 생성하는 Tesseract의 능력은, 수작업 프로그래밍이 경제적으로 불가능했던 영역에 자동화를 도입할 수 있는 길을 열어주고 있다.


ROS2 생태계에서 모션 플래닝을 논할 때, Tesseract와 MoveIt2는 가장 중요하게 고려되는 두 프레임워크이다. 두 시스템은 로봇 매니퓰레이션을 위한 모션 플래닝 프레임워크라는 공통된 목표를 가지고 있지만, 그들의 탄생 배경, 설계 철학, 그리고 강점을 보이는 영역에서 뚜렷한 차이를 보인다.30 본 장에서는 두 프레임워크의 근본적인 차이점을 비교 분석하고, 각각의 강점과 적합한 적용 분야를 고찰하여 사용자가 자신의 애플리케이션에 가장 적합한 도구를 선택할 수 있는 기준을 제시한다.


Tesseract와 MoveIt2의 차이는 단순히 기능의 유무를 넘어, 시스템을 구축하는 근본적인 철학에서 비롯된다. Tesseract는 앞서 언급했듯이, TrajOpt와 같은 최적화 기반 플래너를 지원하기 위해 MoveIt이 제공하지 못했던 특정 기능들(예: 연속 충돌 검사, 상세 거리 정보)을 구현할 필요성에서 탄생했다.8 이로 인해 Tesseract는 산업용 공정 계획에 특화된 고성능, 고신뢰성 시스템으로 발전했다. 반면, MoveIt은 ROS 생태계 전반에 걸쳐 사용되는 범용 조작(general-purpose manipulation) 프레임워크로서, 광범위한 적용 가능성과 ROS와의 깊은 통합에 중점을 둔다.31

이러한 철학의 차이는 아키텍처의 핵심적인 결정들로 이어진다. Tesseract는 병렬 처리 성능과 결정론적 상태 관리를 위해 '복제(Cloning)'와 'Command History'라는 독자적인 메커니즘을 채택했다.12 반면 MoveIt은 ROS의 표준적인 통신 방식과 통합된 'Planning Scene Monitor'를 통해 실시간으로 환경 변화를 관리하며, 스레드 안전성은 전통적인 뮤텍스 잠금 방식으로 확보한다.31 이러한 근본적인 아키텍처의 차이로 인해, 현재 두 프레임워크를 단순 병합하거나 통합하는 것은 매우 어려운 과제이며, 계획되지 않고 있다.30

**표 2: Tesseract vs. MoveIt2 핵심 아키텍처 비교**

| 기능 (Feature)         | Tesseract                                    | MoveIt2                                                      | 시사점 (Implication)                                         |
| ---------------------- | -------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **핵심 철학**          | 산업 공정 계획 (Industrial Process Planning) | 범용 조작 프레임워크 (General Purpose Manipulation Framework) | Tesseract는 특정 산업 문제 해결에, MoveIt2는 광범위한 로봇 연구 및 애플리케이션에 강점을 가짐 |
| **스레드 안전성 모델** | 복제 (스레드별 Deep Copy)                    | Const Correctness 및 뮤텍스 잠금                             | Tesseract는 병렬 알고리즘 설계가 단순하고 잠재적 성능 이점이 있으나 메모리 사용량이 큼. MoveIt2는 메모리 효율적이나 교착 상태 방지를 위한 신중한 구현이 요구됨 |
| **환경 상태 관리**     | Command History (결정론적 복제)              | Planning Scene Monitor (실시간 업데이트)                     | Tesseract는 상태의 완벽한 재현과 디버깅에 유리. MoveIt2는 ROS 토픽을 통한 동적 환경 변화에 즉각적으로 반응하는 데 유리 |
| **충돌 검사**          | 연속 충돌 검사(CCC)가 핵심 기능              | 주로 이산적 검사 (CCC 통합 수준이 낮음)                      | Tesseract는 고속/고정밀 작업에서 수학적으로 보장된 안전 경로 생성에 탁월 |
| **주요 플래너 유형**   | 최적화 기반 (TrajOpt)                        | 샘플링 기반 (OMPL)                                           | Tesseract는 복잡한 제약 조건 하에서 최적 경로를 찾는 데, MoveIt2는 복잡한 공간에서 실행 가능한 경로를 빠르게 찾는 데 강함 |
| **ROS 의존성**         | ROS-Agnostic Core와 ROS 래퍼                 | ROS와 깊게 통합                                              | Tesseract는 비(非)ROS 환경으로의 이식성이 높음. MoveIt2는 ROS 생태계의 모든 도구와 긴밀하게 연동됨 |


두 프레임워크는 서로 다른 강점을 가지고 있으며, 이는 각각이 더 적합한 적용 분야로 이어진다.


MoveIt2의 가장 큰 강점은 ROS 생태계와의 깊고 성숙한 통합이다.31 Planning Scene, RViz 플러그인, 액션 서버 등 모든 구성 요소가 ROS의 표준 방식으로 작동하며, 방대한 사용자 커뮤니티와 풍부한 문서를 보유하고 있다. OMPL을 통해 제공되는 다양한 샘플링 기반 플래너들은 복잡한 환경에서 실행 가능한 경로를 신속하게 찾는 데 매우 효과적이다.32 따라서 MoveIt2는 다음과 같은 분야에 매우 적합하다.

- 학술 연구 및 신속한 프로토타이핑
- 물류 및 서비스 로봇과 같이 자유 공간 이동이 주가 되는 애플리케E이션
- ROS의 다양한 도구(예: `ros2_control`, `Navigation2`)와의 긴밀한 연동이 필수적인 시스템 33


Tesseract는 산업 현장의 특수한 요구사항을 해결하는 데 초점을 맞춘다. 연속 충돌 검사를 통한 높은 수준의 안전성 보장, TrajOpt를 활용한 최적화 및 공정 제약 조건 처리 능력, 그리고 결정론적 상태 관리 기능은 MoveIt2와 차별화되는 Tesseract의 핵심 강점이다.8 따라서 Tesseract는 다음과 같은 고부가가치 산업 애플리케이션에 이상적이다.

- 용접, 도색, 연삭, 디버링 등 정밀한 데카르트 경로와 공정 제약 조건이 중요한 작업
- 안전이 최우선시되는 항공우주, 의료, 원자력 분야의 로봇 작업
- Scan-N-Plan과 같이 비정형적인 환경에서 자율적으로 작업을 계획하고 수행해야 하는 시스템
- ROS가 아닌 다른 시스템 환경에 통합되거나, 실시간 운영체제(RTOS)와의 연동이 필요한 고성능 애플리케이션

결론적으로, '어느 것이 더 좋은가'라는 질문은 적절하지 않다. '어떤 문제를 해결하려 하는가'에 따라 선택이 달라져야 한다. 범용성과 ROS 생태계와의 통합이 중요하다면 MoveIt2가 훌륭한 선택이며, 산업 수준의 정밀도, 안전성, 그리고 최적화가 필요하다면 Tesseract가 더 강력한 솔루션을 제공한다.


앞서 언급했듯이, 근본적인 아키텍처 차이로 인해 두 프레임워크의 완전한 통합은 현실적으로 어렵다.30 하지만 이는 두 시스템이 완전히 고립되어야 함을 의미하지는 않는다. 오히려 로봇 소프트웨어 생태계는 단일 프레임워크가 모든 것을 해결하는 방식에서, 각 분야의 '최고(best-of-breed)' 컴포넌트들을 조합하여 사용하는 방향으로 성숙하고 있다.34

이러한 관점에서 Tesseract와 MoveIt2는 경쟁 관계가 아닌 상호 보완적인 관계가 될 수 있다. 예를 들어, 개발자는 ROS와의 통합이 뛰어난 MoveIt의 Planning Scene Monitor를 사용하여 작업 환경을 관리하고 시각화할 수 있다. 그러다 복잡한 공정 계획이 필요한 시점에는, 현재의 씬 정보를 Tesseract 기반의 플래닝 노드로 전달한다. 이 노드는 Tesseract의 강력한 충돌 검사와 TrajOpt 플래너를 사용하여 최적의 궤적을 생성하고, 그 결과를 다시 ROS 시스템으로 반환하여 `ros2_control` 기반의 컨트롤러가 실행하도록 구성할 수 있다. `tesseract_rosutils`와 같은 변환 유틸리티 패키지는 바로 이러한 컴포넌트 기반의 '조립식(composable)' 시스템 구축을 실질적으로 가능하게 하는 도구이다.23 이러한 접근 방식은 각 프레임워크의 장점만을 취하여, 더 강력하고 유연한 로봇 시스템을 구축할 수 있는 미래를 제시한다.


본 보고서는 Southwest Research Institute가 개발한 산업용 모션 플래닝 프레임워크인 Tesseract에 대해 심층적으로 분석했다. Tesseract는 단순한 MoveIt의 대안이 아니라, 최적화 기반 플래너의 요구사항을 충족하고 실제 산업 현장의 엄격한 기준을 만족시키기 위해 탄생한 목적 지향적 시스템이다. 본 결론에서는 Tesseract의 핵심적인 기여와 기술적 의의를 요약하고, ROS-Industrial 로드맵 내에서의 역할과 향후 발전 방향을 전망하며, 산업 로봇 공학의 미래에 미칠 영향을 고찰한다.


Tesseract의 가장 핵심적인 기여는 최적화 기반 플래너와 복잡한 공정 자동화의 요구에 특화된, 견고하고 고성능의 산업용 모션 플래닝 프레임워크를 제공했다는 점에 있다. Tesseract가 제시한 기술적 혁신들은 기존 프레임워크의 한계를 명확히 겨냥하고 있다.

- **연속 충돌 검사(CCC)**는 이산적 검사의 맹점을 해결하여, 고속·고정밀 작업 환경에서 수학적으로 보장된 수준의 안전성을 제공한다.
- **복제(Cloning) 기반의 동시성 모델**은 복잡한 병렬 플래닝 알고리즘의 설계를 단순화하고 성능을 극대화한다.
- **Command History를 통한 상태 복제**는 산업 현장에서 필수적인 시스템의 결정론적 재현과 신속한 디버깅을 가능하게 한다.

이러한 혁신들은 실제 제조 환경에서 요구되는 안전성, 성능, 신뢰성의 공백을 직접적으로 메우며, Tesseract를 학술적 연구 단계를 넘어 실제 산업 현장에 적용 가능한 강력한 도구로 자리매김하게 했다.


Tesseract는 ROS-Industrial 컨소시엄의 미래 전략에서 핵심적인 위치를 차지하고 있다. ROS-I의 장기 로드맵은 '사용 용이성(Ease of Use)', '첨단 기능(Advanced Capability)', '상호 운용성(Interoperability)'이라는 세 가지 축을 중심으로 전개된다.15 Tesseract는 이 중 '첨단 기능'의 대표적인 사례이며, 그 자체의 발전 방향 또한 이 로드맵과 궤를 같이 한다.

향후 Tesseract는 두 가지 방향으로 발전할 것으로 전망된다. 첫째, SWORD와 같은 그래픽 기반 도구의 발전을 통해 '사용 용이성'을 지속적으로 개선할 것이다. 복잡한 플래닝 기능을 비전문가도 쉽게 활용할 수 있게 함으로써 기술의 보급을 가속화할 것이다. 둘째, 용접, 도색, 조립 등 다양한 산업 공정에 특화된 플래닝 파이프라인과 플러그인 라이브러리를 확장하여 '첨단 기능'을 더욱 심화시킬 것이다.

ROS-I는 명확한 릴리스 목표를 가진 모듈식 프레임워크와 상세한 문서, 그리고 풍부한 애플리케이션 예제를 제공하는 방향으로 나아가고 있으며, Tesseract는 이러한 노력의 중심에 서 있다.15 정기적인 'Tesseract Monthly Check-In' 미팅은 개발자와 사용자 커뮤니티 간의 소통을 촉진하며 개방적인 발전을 이끌고 있다.14


Tesseract와 같은 차세대 모션 플래닝 프레임워크는 산업 로봇 공학의 미래에 지대한 영향을 미칠 것이다. 이 기술은 로봇을 단순한 반복 작업 기계에서, 변화하는 환경과 요구에 지능적으로 적응하는 '스마트 팩토리'의 핵심 요소로 변모시키는 동력이다.

과거에는 프로그래밍의 어려움과 높은 초기 투자 비용으로 인해 대기업의 전유물이었던 첨단 로봇 자동화를, Tesseract는 중소기업(SME)도 접근 가능한 영역으로 이끌고 있다. Scan-N-Plan과 같은 워크플로우를 통해 수작업 프로그래밍 없이도 다품종 소량 생산에 대응할 수 있게 함으로써, 제조업의 유연성과 회복탄력성을 크게 향상시킬 것이다.

결론적으로, Tesseract는 산업 자동화의 새로운 패러다임을 여는 핵심 기술이다. 복잡한 모션 플래닝을 더 안전하고, 더 빠르고, 더 접근하기 쉽게 만듦으로써, Tesseract는 인간과 로봇이 더욱 긴밀하게 협력하는 미래의 생산 현장을 만들어가는 데 결정적인 역할을 수행할 것이다.


1. Intuitive Robotics | Southwest Research Institute, 8월 17, 2025에 액세스, https://www.swri.org/newsroom/technology-today/intuitive-robotics
2. ROS-I Enabling Innovation for Industry through Open Source, 8월 17, 2025에 액세스, https://rosindustrial.squarespace.com/s/11_Matt-Robinson_SwRI.pdf
3. Southwest Research Institute - Wikipedia, 8월 17, 2025에 액세스, https://en.wikipedia.org/wiki/Southwest_Research_Institute
4. History | Southwest Research Institute, 8월 17, 2025에 액세스, https://www.swri.org/about-us/history
5. About Us | Southwest Research Institute, 8월 17, 2025에 액세스, https://www.swri.org/about-us
6. Robot Operating System (ROS) | SwRI, 8월 17, 2025에 액세스, https://www.swri.org/markets/electronics-automation/electronics/electronics-systems-robotics/robot-operating-system-ros
7. ROS-Industrial | SwRI, 8월 17, 2025에 액세스, https://www.swri.org/markets/manufacturing-construction/manufacturing-technologies/industrial-robotics-automation/ros-industrial
8. Optimization Motion Planning with Tesseract and TrajOpt for Industrial Applications, 8월 17, 2025에 액세스, https://rosindustrial.org/news/2018/7/5/optimization-motion-planning-with-tesseract-and-trajopt-for-industrial-applications
9. Trajectory Optimization for Motion Planning in ROS, 10-R8831, 8월 17, 2025에 액세스, https://www.swri.org/what-we-do/internal-research-development/2018/manufacturing-construction/trajectory-optimization-motion-planning-ros-10-r8831
10. Finding Locally Optimal, Collision-Free ... - John Schulman, 8월 17, 2025에 액세스, http://joschu.net/docs/trajopt-paper.pdf
11. Motion Planning with Sequential Convex Optimization and Convex Collision Checking, 8월 17, 2025에 액세스, https://rll.berkeley.edu/trajopt/ijrr/2013-IJRR-TRAJOPT.pdf
12. Overview — Tesseract 3cd485a843e521cb2c2344110d332ea69405f375 documentation, 8월 17, 2025에 액세스, https://tesseract-docs.readthedocs.io/en/latest/_source/core/overview/index.html
13. tesseract-robotics/tesseract: Motion Planning Environment - GitHub, 8월 17, 2025에 액세스, https://github.com/tesseract-robotics/tesseract
14. Tesseract Robotics - ROS-Industrial, 8월 17, 2025에 액세스, https://rosindustrial.org/tesseract-robotics
15. Blog — ROS-Industrial, 8월 17, 2025에 액세스, https://rosindustrial.org/news
16. ROS-Industrial Roadmap Journey and a Path Forward, 8월 17, 2025에 액세스, https://rosindustrial.org/news/2025/4/30/ros-industrial-roadmap-journey-and-a-path-forward
17. Optimization Motion Planning with TrajOpt and Tesseract for Industrial Applications, 8월 17, 2025에 액세스, https://roscon.ros.org/2018/presentations/ROSCon2018_Tesseract_and_TrajOpt.pdf
18. Tesseract, 8월 17, 2025에 액세스, https://ros-industrial-tesseract.readthedocs.io/_/downloads/en/stable/pdf/
19. Using Tesseract and Trajopt for a Real-Time Application - ROS-Industrial, 8월 17, 2025에 액세스, https://rosindustrial.org/news/2022/2/21/using-tesseract-and-trajopt-for-a-real-time-application
20. trajopt: Trajectory Optimization for Motion Planning - UC Berkeley Robot Learning Lab, 8월 17, 2025에 액세스, https://rll.berkeley.edu/trajopt/doc/sphinx_build/html/
21. Motion planning with sequential convex optimization and convex collision checking, 8월 17, 2025에 액세스, https://escholarship.org/uc/item/6km506db
22. Robotics Research The International Journal of - People @EECS, 8월 17, 2025에 액세스, https://people.eecs.berkeley.edu/~pabbeel/papers/2014-IJRR-trajopt.pdf
23. tesseract-robotics/tesseract_ros2 - GitHub, 8월 17, 2025에 액세스, https://github.com/tesseract-robotics/tesseract_ros2
24. Robotic Blending Milestone 5 Wraps Up on the Foundry Shop Floor - ROS-Industrial, 8월 17, 2025에 액세스, https://rosindustrial.org/news/2024/5/17/robotic-blending-milestone-5-wraps-up-on-the-foundry-shop-floor
25. Advance plant-floor robotics three ways with ROS-Industrial tools - Control Engineering, 8월 17, 2025에 액세스, https://www.controleng.com/advance-plant-floor-robotics-three-ways-with-ros-industrial-tools/
26. Enabling Facility-Level Interoperability Between Robot Teams and Machine Cell Devices, 8월 17, 2025에 액세스, https://www.nist.gov/system/files/documents/2018/04/10/5p_interoperability_of_manufacturing_resource_teams-robinson-sobel-langsfeld-mbe2018-04052018b.pdf
27. ARMATRIX: Multi-Arm Robotic Task Integration and Execution, 8월 17, 2025에 액세스, https://www.swri.org/markets/industrial-robotics-automation/blog/armatrix-multi-arm-robotic-task-integration-execution
28. Southwest Research Institute to make robot programming more user friendly with SWORD, 8월 17, 2025에 액세스, https://www.therobotreport.com/southwest-research-institute-makes-robot-programming-more-user-friendly-sword/
29. SwRI Workbench for Offline Robotics Development: Home, 8월 17, 2025에 액세스, https://sword.swri.org/
30. do you have plan to combine the tesseract_ros with moveit ... - GitHub, 8월 17, 2025에 액세스, https://github.com/tesseract-robotics/tesseract_ros/discussions/186
31. Is MoveIt2 the correct/best application to use for simulating a long container-loading process? : r/ROS - Reddit, 8월 17, 2025에 액세스, https://www.reddit.com/r/ROS/comments/1bheoe9/is_moveit2_the_correctbest_application_to_use_for/
32. The Open Motion Planning Library, 8월 17, 2025에 액세스, https://ompl.kavrakilab.org/
33. Is moveit2 ready for industrial robot manipulation? - General - ROS Discourse, 8월 17, 2025에 액세스, https://discourse.ros.org/t/is-moveit2-ready-for-industrial-robot-manipulation/22040
34. Manipulation in ROS2 (2025): What's everyone using these days? - ROS General, 8월 17, 2025에 액세스, https://discourse.openrobotics.org/t/manipulation-in-ros2-2025-what-s-everyone-using-these-days/43683


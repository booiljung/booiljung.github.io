# Autoware Core/Universe
[ROS2 Autoware](./index.md)


Autoware 프로젝트는 세계 최대 규모의 자율주행 오픈소스 소프트웨어(OSS) 프로젝트로서, Robot Operating System(ROS)을 기반으로 연구 개발부터 상용 배포에 이르기까지 광범위한 차량과 애플리케이션을 지원하는 것을 목표로 한다.1 2015년 나고야 대학에서 시작된 이래, Autoware는 전 세계 수많은 기업과 연구 기관의 협력을 통해 자율주행 기술의 민주화를 이끌어왔다.

이 보고서의 핵심 주제는 Autoware의 최신 개발 패러다임인 'Core/Universe' 이원적 구조다. 이 구조는 안정적인 상용 배포를 목표로 하는 `Core`와 최신 기술의 신속한 실험 및 연구를 위한 `Universe`라는 두 개의 축으로 구성된다.3 이는 자율주행이라는 안전 최우선(safety-critical) 분야에서 오픈소스 프로젝트가 필연적으로 마주하는 '안정성'과 '혁신' 사이의 딜레마를 해결하기 위한 전략적 선택이다. 단순히 기술적 구분을 넘어, 서로 다른 목적을 가진 두 개발자 그룹(상용 시스템 통합 전문가와 학계 연구자)을 모두 만족시키려는 거버넌스 철학이 아키텍처를 통해 구현된 것이다.

본 문서는 Autoware의 역사적 진화 과정부터 현재 Core/Universe 아키텍처의 심층 분석, 개발 및 승격 프로세스, 주요 경쟁 플랫폼과의 비교, 그리고 'Autoware 2.0'으로 대표되는 미래 비전까지를 총망라하여 심도 있는 기술적 통찰을 제공하는 것을 목표로 한다.


현재의 Core/Universe 구조를 이해하기 위해서는 그 이전 세대인 Autoware.AI와 Autoware.Auto가 남긴 유산과 교훈을 먼저 살펴봐야 한다. Autoware의 발전사는 ROS라는 미들웨어의 한계와 가능성 속에서 끊임없이 최적의 균형점을 찾아온 과정 그 자체라 할 수 있다.


ROS 1을 기반으로 한 최초의 배포판인 Autoware.AI는 전 세계 100개 이상의 기업과 30종 이상의 차량에 적용되며 Autoware 프로젝트의 가능성을 세상에 알렸다.3 인식, 측위, 계획, 제어 등 자율주행에 필요한 모든 기능을 'all-in-one' 형태로 제공하여 많은 개발자들이 자율주행 기술에 쉽게 접근할 수 있는 길을 열었다.3

하지만 이러한 성공 이면에는 심각한 문제점들이 누적되고 있었다.

- **아키텍처의 부재:** 명확한 아키텍처 설계 없이 기능이 유기적으로 추가되면서 모듈 간의 결합도(tight coupling)가 극심해지고 각 모듈의 책임 소재가 불분명해지는 등 심각한 기술적 부채가 쌓였다.3
- **품질 관리의 한계:** 패키지별로 코딩 표준이 제각각이었고, 테스트 커버리지가 매우 낮아 시스템의 신뢰성을 보장하기 어려웠다.3
- **ROS 1의 근본적 한계:** Autoware.AI의 문제는 근본적으로 ROS 1의 한계와 맞닿아 있었다. ROS 1은 본래 학술 연구용으로 설계되어 실시간성, 메모리 안전성, 보안 등 상용화 및 인증에 필수적인 요소를 보장하기 어려웠다.5

결국 이러한 문제들로 인해 Autoware.AI는 2022년 말 공식적으로 지원이 종료(End-of-Life)되었다.3


Autoware.AI의 교훈을 바탕으로, Autoware Foundation은 ROS 2로의 전환을 결정했다. 이는 단순히 기존 코드를 포팅하는 수준을 넘어, 프로젝트를 처음부터 완전히 재작성하는 방식으로 진행되었다.3 ROS 2가 제공하는 DDS(Data Distribution Service) 기반 통신, Lifecycle Nodes 등 새로운 기능들을 적극적으로 활용하고 과거의 기술적 부채를 완전히 청산하기 위한 결단이었다.

Autoware.Auto는 새로운 개발 철학을 도입했다.

- **ODD (Operational Design Domain) 기반 개발:** '자율 발렛 주차(AVP)', '화물 운송' 등 명확한 사용 사례와 ODD를 먼저 정의하고, 해당 ODD를 만족시키는 데 필요한 최소한의 기능부터 개발해나가는 접근 방식을 채택했다.6
- **엄격한 소프트웨어 공학:** 모든 코드 변경에 대해 설계 문서를 요구하고, 100% 코드 커버리지를 목표로 하며, 철저한 코드 리뷰와 CI/CD 파이프라인을 도입하는 등 현대적인 소프트웨어 공학 기법을 전면적으로 적용했다.1

이러한 노력으로 코드의 품질과 신뢰성은 비약적으로 향상되었지만, 예상치 못한 새로운 문제가 발생했다.

- **높은 진입 장벽:** 지나치게 엄격한 품질 요구사항과 복잡한 병합 절차는 새로운 개발자, 특히 학계 연구자나 학생들이 프로젝트에 기여하는 것을 매우 어렵게 만들었다. 결과적으로 기여자가 소수의 Autoware Foundation 회원사 소속 엔지니어로 제한되면서, 학계의 최신 연구 성과가 프로젝트에 흡수되지 못하는 현상이 발생했다.4
- **아키텍처 변경의 어려움:** 높은 안정성을 유지하면서 대규모 아키텍처 변경을 시도하는 데 막대한 오버헤드가 발생했다. 이는 새로운 아이디어를 실험하고 시스템 구조를 개선하는 데 큰 걸림돌이 되었다.4


Autoware.Auto가 직면한 딜레마, 즉 높은 품질 기준이 오히려 혁신과 커뮤니티 참여를 저해하는 문제를 해결하기 위해, 2020년 TIER IV는 Autoware 기술운영위원회(TSC)에서 '연합 소프트웨어 개발 모델(federated software development model)'이라는 새로운 패러다임을 제안했다.3

이것이 바로 Core/Universe 구조의 시작이었다. 핵심 아이디어는 명확했다. Autoware.Auto의 엄격한 품질 기준과 안정성은 `Core`로 계승하여 상용화의 기반을 다지고, 동시에 기여의 문턱을 낮춘 `Universe`라는 별도의 공간을 만들어 연구와 실험을 촉진하는 것이다. `Universe`는 커뮤니티가 주도하는 혁신의 장(場)이 되고, 여기서 충분히 검증되고 유용성이 입증된 기능들은 엄격한 검토 과정을 거쳐 `Core`로 편입되는 선순환 구조를 만드는 것이 최종 목표였다.1

이러한 역사적 맥락을 살펴보면, Autoware의 아키텍처 변화는 ROS 생태계의 발전과 그 궤를 같이함을 알 수 있다. Autoware.AI의 문제는 ROS 1의 태생적 한계였고, Autoware.Auto로의 재작성은 ROS 2라는 새로운 패러다임에 적응하기 위한 필연적인 과정이었다. 심지어 현재의 Core/Universe 구조에서도 ROS 2의 메시지 구조가 야기하는 성능 문제(예: zero-copy IPC의 어려움)를 해결하기 위해 Agnocast와 같은 외부 미들웨어를 통합하는 등, Autoware는 ROS를 상용 수준으로 끌어올리기 위한 가장 치열한 시험대 역할을 하고 있다.8

| 구분 (Category)         | Autoware.AI             | Autoware.Auto                  | Autoware Core/Universe                 |
| ----------------------- | ----------------------- | ------------------------------ | -------------------------------------- |
| **기반 미들웨어**       | ROS 1                   | ROS 2                          | ROS 2                                  |
| **아키텍처 철학**       | All-in-one, 유기적 성장 | ODD 기반, Top-down 설계        | 이원적 구조 (안정성 + 확장성)          |
| **코드 품질 및 테스트** | 낮음, 비일관적          | 매우 높음 (100% 커버리지 목표) | Core: 매우 높음 / Universe: 완화됨     |
| **주요 기여자**         | 학계, 초기 커뮤니티     | AWF 회원사 중심                | AWF + 글로벌 커뮤니티                  |
| **핵심 문제점**         | 기술 부채, 신뢰성 부족  | 높은 진입 장벽, 낮은 유연성    | 두 저장소 간의 거버넌스 및 통합 복잡성 |
| **현재 상태**           | EOL (2022년 말)         | Core/Universe로 계승           | 현재 주력 개발 버전                    |


Core/Universe 패러다임은 두 개의 독립적인 GitHub 저장소를 중심으로 구현된다. 이 분리는 단순한 코드의 물리적 분리를 넘어, 서로 다른 개발 철학과 목표를 가진 두 개의 생태계를 공존시키기 위한 구조적 장치다.


- **`Autoware Core`**

  - **목표:** `Core`의 존재 이유는 '안정성'과 '신뢰성'이다. 이는 Autoware Foundation(AWF)이 직접 관리하며, 자율 발렛 주차(AVP), 화물 운송 등 모든 공식 ODD를 지원하는 완전한 End-to-End 프레임워크를 제공하는 것을 목표로 한다.3

  - **특징:** `Core`에 포함되기 위한 조건은 매우 까다롭다. 엄격한 코드 및 품질 관리, 100%에 가까운 테스트 커버리지, 포괄적인 문서화, 그리고 명확하게 정의된 개발 및 릴리스 프로세스를 반드시 따라야 한다.1 이러한 높은 기준으로 인해 

    `autoware_core` 저장소는 개념이 도입된 후 거의 3년 가까이 비어 있다가, 최근에서야 최소 기능셋을 구성하기 위한 패키지 포팅 작업이 시작되었다.11

- **`Autoware Universe`**

  - **목표:** `Universe`는 '혁신'과 '실험'의 공간이다. 학계와 커뮤니티의 폭넓은 참여를 촉진하여 최신 기술을 신속하게 실험하고 프로토타이핑하는 것을 목적으로 한다.1 Robo-Taxi나 고속 자율주행 레이싱과 같이 복잡하고 도전적인 사용 사례에 대한 연구가 이곳에서 이루어진다.1

  - **특징:** `Core`에 비해 완화된 코드 품질 요구사항을 적용하여 새로운 아이디어를 가진 개발자들이 쉽게 기여할 수 있도록 진입 장벽을 낮췄다. 하지만 무질서했던 Autoware.AI 시절보다는 훨씬 엄격한 기준을 유지한다. `Universe`의 모든 패키지는 `Core`가 제공하는 기능과 메시지 정의에 완전히 의존하며, 커뮤니티 중심으로 관리된다.3

    `autoware_universe` 저장소는 C++(91.5%)와 Python(5.0%)을 중심으로 매우 다양한 패키지들을 포함하고 있다.14

- **상호작용:** 이 두 저장소는 유기적으로 상호작용한다. 사용자는 자신의 목적에 따라 `Core`의 기본 모듈을 `Universe`에 있는 더 새롭거나 실험적인 모듈로 손쉽게 교체하여 사용할 수 있다. 반대로, `Universe`에서 개발된 기능이 충분한 테스트를 거쳐 그 유용성과 안정성이 입증되면, AWF의 엄격한 검토를 거쳐 `Core`로 승격될 수 있다.3


Autoware는 각 모듈의 역할을 명확히 하고 모듈 간 인터페이스를 단순화하기 위해 계층형 아키텍처(Layered Architecture)를 채택하고 있다. 이는 시스템 내부 처리 과정의 투명성을 높이고, 모듈 간 상호 의존성을 줄여 여러 개발자가 동시에 협업하기 용이하게 만든다.15 이 아키텍처는 크게 7개의 스택으로 구성된다.

1. **Sensing:** LiDAR, 카메라, Radar, GNSS, IMU 등 차량에 탑재된 다양한 센서로부터 원시 데이터를 수집하고, 각 센서 데이터의 특성에 맞는 전처리(필터링, 동기화 등)를 수행하는 역할을 담당한다.1
2. **Localization:** Sensing 스택에서 전달받은 데이터와 Map 스택의 지도 정보를 종합하여, 지도 상에서 차량의 정확한 위치(x, y, z)와 자세(roll, pitch, yaw)를 실시간으로 추정한다.1
3. **Perception:** 주변 환경에 존재하는 객체(다른 차량, 보행자, 자전거, 교통 신호등, 표지판 등)를 탐지하고, 종류를 분류하며, 그 크기와 속도, 미래 이동 경로를 예측한다.1
4. **Planning:** Localization을 통해 파악된 현재 위치와 Perception을 통해 인지된 주변 상황을 바탕으로, 최종 목적지까지의 전역 경로(Global Route)를 생성하고, 교통 법규를 준수하며 장애물을 안전하게 회피하는 상세 주행 경로(Local Trajectory)를 계획한다.1
5. **Control:** Planning 스택에서 생성된 목표 주행 경로를 차량이 정확하게 따라갈 수 있도록 조향각, 가속 및 제동 값을 계산하여 제어 명령을 생성한다.1
6. **Vehicle Interface:** Control 스택에서 생성된 제어 명령을 차량의 물리적인 구동계(스티어링 휠, 액셀, 브레이크)가 이해할 수 있는 신호(예: CAN 메시지)로 변환하여 전달하는 인터페이스 역할을 한다.1
7. **Map:** 차선, 정지선, 신호등 위치 등의 정적 정보를 담고 있는 Lanelet2 기반의 벡터 맵과, 정밀한 위치 추정을 위해 사용되는 3D 점군 맵(Point Cloud Map) 등 자율주행에 필수적인 지도 데이터를 제공하고 관리한다.15

이러한 7-스택 구조는 논리적으로 명확하고 각 기능의 분리가 잘 되어 있다는 장점이 있지만, 동시에 구조적인 한계도 내포한다. 각 스택이 순차적으로 정보를 처리하고 전달하는 과정에서 정보의 손실이나 왜곡이 발생할 수 있다. 예를 들어, Perception 모듈이 객체의 불확실성 정보를 Planning 모듈에 온전히 전달하지 못하면, Planning 모듈은 과도하게 보수적이거나 위험한 경로를 생성할 수 있다. 이러한 정보 병목 현상과 오류 전파 문제는 모듈형 아키텍처의 근본적인 약점이며, 최근 자율주행 업계와 Autoware 커뮤니티가 End-to-End 또는 하이브리드 AI 모델에 주목하는 주된 이유이기도 하다.16


Core/Universe 패러다임의 핵심은 `Universe`에서 탄생한 혁신적인 기능이 체계적인 프로세스를 통해 안정적인 `Core`의 일부가 되는 선순환 구조에 있다. 이 과정은 기술적 요구사항뿐만 아니라 커뮤니티 거버넌스와도 밀접하게 연관되어 있다.


Autoware는 기능 개발을 크게 두 단계로 나누어 진행한다.20

- **Phase 1: Universe에서의 Use-case 기반 개발**
  - **목표:** 이 단계의 핵심은 '신속한 프로토타이핑'이다. 비교적 완화된 규칙 하에서 특정 사용 사례(예: 버스 ODD)나 기술 데모를 중심으로 빠르게 기능을 개발하고 실제 환경 또는 시뮬레이션에서 그 가능성을 검증하는 데 초점을 맞춘다.3
  - **프로세스:** 개발은 `Use case 정의`에서 시작하여, 이를 검증하기 위한 `시나리오 생성`, `시나리오 기반 개발`, 그리고 `데모 시연`으로 이어진다. 데모가 성공적으로 끝나면, 다른 개발자들이 이해하고 활용할 수 있도록 `코드 정리 및 문서화(README 작성, 리팩토링)` 작업을 수행한다. 이 과정에서 축적된 기술적 노하우와 효과적인 테스트 시나리오 셋은 다음 단계인 Core 개발의 중요한 자산이 된다.20
- **Phase 2: Core에서의 고품질 개발**
  - **목표:** 이 단계의 목표는 '상용 수준의 코드 개발'이다. Universe에서 그 가능성을 인정받은 기능을 매우 엄격한 규칙 하에 안정적이고 신뢰성 높은 코드로 재탄생시키는 과정이다.20
  - **프로세스:** `Universe에서 축적된 지식과 시나리오 셋을 활용`하여 `상세 설계 문서를 작성`하는 것으로 시작된다. 이후 `고품질 코드 구현`, `단위/컴포넌트 자동화 테스트 작성`, `시나리오 기반의 CI(Continuous Integration) 통과`, `종합적인 통합 테스트`를 거쳐 최종적으로 `베타 및 안정 버전으로 릴리스`된다.20


`Universe`의 패키지가 `Core`로 승격되는 과정은 Autoware Foundation의 기술적 거버넌스에 따라 체계적으로 관리된다.

- **거버넌스:** Autoware Foundation의 기술운영위원회(TSC, Technical Steering Committee)가 프로젝트의 전반적인 기술 방향성과 거버넌스를 총괄한다.21 실제 승격에 대한 세부적인 기술 논의는 TSC 산하의 각 워킹 그룹(예: Planning & Control WG, Perception & Sensing WG)에서 이루어진다.22
- **승격 제안 및 논의:** 특정 패키지를 `Core`로 승격시키기 위한 제안은 주로 GitHub Discussions를 통해 공개적으로 이루어진다. 예를 들어, 오랫동안 비어 있던 `autoware.core` 저장소를 활성화하고 최소 기능셋을 갖추기 위한 포괄적인 제안과 논의가 'Proposal for implementing Autoware.Core' (#5365) 스레드에서 활발하게 진행되었다.11
- **승격 기준 (Merge Requirements):** 아직 명문화된 단일 문서는 없지만, GitHub 이슈(#88) 및 관련 토론들을 종합해 보면 `Core` 패키지가 되기 위한 기술적 요구사항은 다음과 같이 요약할 수 있다.24

| 영역 (Domain) | 핵심 요구사항 (Key Requirement)                              | 관련 논의 근거 (Source of Discussion) |
| ------------- | ------------------------------------------------------------ | ------------------------------------- |
| **코드 품질** | Autoware.Auto 수준 또는 그 이상의 엄격한 코딩 스타일 가이드 준수. 모든 경고(warning) 제거. | 11                                    |
| **테스트**    | 높은 코드 커버리지(100% 목표)를 달성하는 단위 테스트, 통합 테스트, 시나리오 기반 테스트 필수. | 1                                     |
| **문서화**    | 패키지의 설계 철학, API, 사용법, 파라미터 등을 명확히 설명하는 포괄적인 문서 제공. | 11                                    |
| **CI/CD**     | 모든 CI 테스트(빌드, 정적 분석, 테스트 등)를 통과해야 하며, 통합 테스트를 위한 CI 설계 필요. | 11                                    |
| **의존성**    | `Autoware.Universe`의 다른 패키지에 대한 의존성이 없어야 함. 오직 `Core` 내부 패키지와 외부 라이브러리에만 의존 가능. | 11                                    |
| **거버넌스**  | AWF의 TSC 및 관련 워킹 그룹의 검토와 승인을 받아야 함.       | 21                                    |

- **포팅 절차:** 승격이 결정되면, 실제 코드를 `autoware_universe` 저장소에서 `autoware_core` 저장소로 이전하는 포팅 작업이 진행된다. 이 과정에서 패키지 이름은 `autoware_` 접두사를 사용하도록 표준화되고, 필요한 경우 기능 단순화나 리팩토링이 수행된다. 이후 `autoware_core_launch` 패키지를 통해 `Core`의 일부로서 실행되도록 통합된다.11

  `autoware_motion_utils` 패키지의 포팅 과정이 담긴 GitHub 이슈와 Pull Request는 이러한 절차의 구체적인 사례를 보여준다.25

하지만 이 승격 프로세스는 아직 완전히 정착된 단계라기보다는, 이제 막 구체화되고 있는 단계로 보인다. `autoware_core` 저장소가 개념 도입 후 오랫동안 비어 있었다는 사실과, 최근의 포팅 작업이 커뮤니티의 자발적인 기여보다는 최소 기능셋을 구축하려는 하향식(top-down) 계획에 의해 주도되고 있다는 점이 이를 뒷받침한다.11 패키지 명명 규칙과 같은 기본적인 사안에 대한 논의가 여전히 진행 중이라는 점도 이 프로세스가 아직 성숙 과정에 있음을 시사한다.11


Autoware Core/Universe는 다양한 자율주행 알고리즘을 모듈 형태로 제공하며, 사용자는 필요에 따라 이를 조합하거나 교체할 수 있다.


인식 스택은 `Universe` 저장소를 중심으로 매우 다양한 최신 알고리즘들을 포함하고 있다.

- **알고리즘:** LiDAR 점군 데이터를 개별 객체로 분할하는 전통적인 `euclidean_cluster` 알고리즘 26, 2D 이미지 상에서 객체를 추적하는 고성능 트래커인 `ByteTrack` 28, 그리고 딥러닝 기반의 3D 객체 탐지 모델인 

  `CenterPoint` 등이 대표적이다.29 또한, 카메라, LiDAR, Radar 등 이종 센서의 데이터를 Bird's-Eye-View(BEV) 공간에서 융합하는 `bevfusion`과 같은 최신 센서 융합 기법도 활발히 연구 및 통합되고 있다.29

- **특징:** `Universe`는 Apollo Lidar Segmentation, YOLOX 등 외부에서 성능이 검증된 알고리즘을 적극적으로 통합하여 생태계를 확장하고 있다.29 더 나아가, 대규모 언어 모델(LLM)을 활용하여 "우회해줘"와 같은 인간의 자연어 지시를 해석하고 주행 계획에 반영하는 `Autoware.Flex`와 같은 실험적인 연구도 진행되어, 인식 기술의 새로운 가능성을 모색하고 있다.31


안정적인 측위는 자율주행의 가장 기본이 되는 기술로, `Core`로의 승격이 우선적으로 고려되는 모듈들이 포진해 있다.

- **핵심 알고리즘:**
  - **GNSS 기반 측위:** `gnss_poser`는 위성항법시스템(GNSS)으로부터 수신한 위도, 경도, 고도 정보를 지도 좌표계로 변환하여 차량의 대략적인 위치를 제공한다.11
  - **EKF (Extended Kalman Filter):** `ekf_localizer`는 측위 스택의 핵심으로, GNSS, IMU의 관성 측정치, 차량의 속도(Odometry) 등 여러 센서 정보를 차량의 운동 모델과 함께 융합한다. 칼만 필터를 통해 각 센서의 노이즈를 걸러내고 위치 추정치를 부드럽게 보정하여 훨씬 안정적이고 정확한 위치 정보를 생성한다. `Core` 패키지의 핵심 후보로 논의되고 있다.11
  - **NDT (Normal Distributions Transform):** LiDAR 스캔 데이터와 사전에 제작된 고정밀 점군 지도(Point Cloud Map)를 확률 분포 기반으로 정합(matching)하여 cm 수준의 정밀한 상대 위치를 추정하는 알고리즘이다. 터널이나 도심 빌딩 숲과 같이 GNSS 수신이 어려운 환경에서 핵심적인 역할을 한다. 다만, 초기 위치 추정 오차가 크면 정합에 실패할 수 있는 단점이 있다.32
- **아키텍처:** Autoware의 측위 시스템은 ROS 2의 좌표계 표준인 REP-105를 엄격히 준수한다. 이에 따라 지구 중심의 고정 좌표계인 `earth`부터 지역 지도 좌표계인 `map`, 주행 시작점 기준의 상대 좌표계인 `odom`, 그리고 차량 자체의 중심 좌표계인 `base_link`로 이어지는 명확한 변환(TF) 트리를 구성하여 모든 데이터의 공간적 일관성을 유지한다.32


계획 스택은 가장 복잡한 로직을 담고 있으며, 크게 두 개의 계층으로 나뉘어 역할을 분담한다.35

- **행동 계획 (Behavior Planning):** `behavior_path_planner`가 이 계층의 핵심이다. 차선 유지, 차선 변경, 정적 장애물 회피, 갓길 정차(pull over) 등 교통 법규와 주변 교통 상황을 종합적으로 고려하여 거시적인 주행 경로와 주행 가능 영역(drivable area)을 생성한다. 이 모듈은 'Scene Module'이라는 플러그인 형태로 구현되어, 각 주행 시나리오에 맞는 로직을 독립적으로 개발하고 추가하기 용이한 구조를 가지고 있다.36
- **동작 계획 (Motion Planning):** 행동 계획 계층에서 생성된 경로를 따라 실제로 차량이 부드럽고 안전하게 주행할 수 있도록 미시적인 속도와 경로를 계획한다.
  - `motion_velocity_planner`: 신호등, 정지선, 횡단보도 등 도로의 특정 지점에서의 제약 조건을 고려하여 최대 속도를 계획한다.11
  - `obstacle_stop_planner`: 주행 경로 상에 나타난 동적/정적 장애물에 대해 안전하게 정지하거나, 감속하거나, 혹은 앞차와의 거리를 유지하며 따라가는 적응형 순항(Adaptive Cruise Control)을 수행하도록 속도를 정밀하게 조절한다.37


제어 모듈은 계획된 경로를 차량이 실제로 따라가도록 만드는 마지막 단계를 책임진다. Autoware는 상황과 필요에 따라 선택할 수 있는 두 가지 주요 제어기를 제공한다.

- **PID Controller:**

  - **개요:** 현재 Autoware의 종방향(가속/감속) 제어에 기본적으로 사용되는 방식이다. 목표 속도와 현재 속도의 오차(Proportional), 오차의 누적값(Integral), 오차의 변화율(Derivative)을 조합하여 필요한 가속/감속 명령을 계산한다.

  $$
  u(t)=K_p e(t)+K)_i ∫_0^t e(\tau) d \tau + K_d \frac{de(t)}{dt}
  $$

  - **장단점:** 구현이 간단하고 직관적이며, 개발 및 유지보수 비용이 낮다는 장점이 있어 기본 제어기로 채택되었다.38 하지만 차량의 복잡한 동역학 모델을 고려하지 않기 때문에, 급격한 속도 변화나 외부 외란에 대해 응답이 느리거나 진동(oscillation)하는 등 최적의 성능을 내기 어렵다.40

- **MPC (Model Predictive Control):**

  - **개요:** 차량의 동역학 모델을 기반으로, 현재 상태로부터 미래의 일정 시간(prediction horizon) 동안의 움직임을 예측하고, 주어진 제약 조건(예: 승차감, 조향각 제한)을 만족시키면서 목표 경로와의 오차를 최소화하는 최적의 제어 입력 시퀀스를 계산한다. Autoware에서는 주로 횡방향(조향) 제어에 사용된다.39
  - **장단점:** 시스템의 미래 상태를 예측하고 최적화 과정을 거치므로 PID에 비해 훨씬 부드럽고 정확한 궤적 추종 성능을 보여준다.40 반면, 정확한 차량 모델이 필수적이며, 매 스텝마다 최적화 문제를 풀어야 하므로 PID보다 계산 복잡도가 훨씬 높다는 단점이 있다.39

이러한 알고리즘 선택은 Autoware의 실용주의적 설계 철학을 잘 보여준다. 먼저 PID와 같이 간단하고 신뢰성 있는 기술로 기본 기능을 구현하여 안정적인 `Core`의 기반을 다진 후, MPC와 같이 더 복잡하고 성능이 우수한 알고리즘을 `Universe`에서 개발하고 검증하여 점진적으로 시스템을 고도화해 나가는 전략을 취하고 있다.


자율주행 오픈소스 생태계에서 Autoware의 가장 강력한 경쟁자는 중국의 Baidu가 주도하는 Apollo 프로젝트다. 두 프로젝트는 기술 스택, 설계 철학, 생태계 전략 등 여러 면에서 뚜렷한 차이를 보인다.


- **Autoware (ROS 2):** 산업 표준으로 자리 잡은 DDS(Data Distribution Service)를 미들웨어로 사용한다. 이는 다양한 벤더의 솔루션을 지원하고 넓은 ROS 생태계와의 호환성을 보장하는 장점이 있다. 하지만 Publisher와 Subscriber 간에 데이터를 전송할 때 TCP/UDP 소켓을 생성하고 데이터를 직렬화(serialization)하는 과정을 거친다. 이로 인해 대용량 데이터(예: LiDAR 점군) 전송 시 오버헤드가 발생하여 병목 현상의 원인이 될 수 있다.42
- **Apollo (CyberRT):** Baidu가 자율주행 환경에 최적화하기 위해 자체 개발한 미들웨어다. 가장 큰 특징은 공유 메모리(Shared Memory)를 활용한 통신 방식이다. Publisher가 공유 메모리에 데이터를 직접 쓰면 Subscriber가 이를 복사 없이 읽어가므로, 데이터 직렬화 과정이 생략되어 통신 지연 시간을 획기적으로 줄일 수 있다.43 다만, 여러 프로세스가 동시에 공유 메모리에 접근할 때 발생할 수 있는 데이터 손상(race condition)을 방지하기 위한 정교한 동기화 메커니즘이 필수적이다.


- **Autoware:** '레고 블록'처럼 각 모듈을 쉽게 교체하고 확장할 수 있는 모듈식 접근 방식을 극대화한다. 이는 새로운 알고리즘을 테스트하거나 특정 하드웨어에 맞게 시스템을 수정해야 하는 학계 및 연구 개발 환경에 매우 유리하다.44 하지만 사용자가 직접 각 모듈을 조합하고 최적화해야 하므로 통합의 복잡성이 높을 수 있다.
- **Apollo:** '완성된 가전제품'처럼 즉시 사용 가능한 통합 시스템을 지향한다. 상용 배포를 목표로 잘 최적화된 파라미터와 함께 포괄적인 솔루션을 제공하여 'out-of-the-box' 사용성이 뛰어나다.44 반면, 시스템의 내부 구조가 긴밀하게 통합되어 있어 특정 모듈만 교체하거나 깊이 있는 사용자화를 진행하기에는 제약이 따른다.


- **Localization:** Autoware는 주로 NDT와 EKF의 조합에 의존하는 반면, Apollo는 이동통신망을 통해 GNSS 보정 데이터(RTK)를 수신하여 절대 위치 정확도를 높이는 방식을 적극적으로 활용한다. 또한 LiDAR 측위에서도 Apollo는 점군을 BEV(Bird's-Eye View)로 투영한 뒤 이미지 기반 정합 알고리즘인 Lucas-Kanade를 적용하여 NDT보다 계산 효율성을 높이는 접근을 취한다.43
- **Perception & Planning:** Apollo는 25개 이상의 사전 훈련된 고성능 딥러닝 모델을 제공하지만, 모델을 학습시킨 데이터셋이 공개되지 않아 다른 종류의 차량이나 센서 구성에 그대로 적용하기 어렵다는 한계가 있다. 반면, Autoware는 다양한 오픈소스 모델을 사용자가 직접 학습시키고 통합하는 데 더 개방적인 구조를 가지고 있다.46


두 프로젝트의 가장 근본적인 차이는 생태계를 운영하는 방식에 있다. Autoware는 다양한 기업, 대학, 연구소가 참여하는 Autoware Foundation이라는 컨소시엄을 통해 운영되는 '연합형(federated)' 모델이다.21 의사결정 과정이 분산되어 있으며, 프로젝트의 방향성은 여러 이해관계자의 합의를 통해 결정된다. 반면, Apollo는 Baidu라는 거대 기업이 중심이 되어 프로젝트를 이끌어가는 '중앙 집중형 플랫폼' 모델에 가깝다.46 비록 오픈소스이지만 핵심 기술 개발과 전략적 방향성은 Baidu가 주도하며, 일부 핵심 기능은 외부에서 접근할 수 없는 자사의 클라우드 서비스에 의존하기도 한다.

따라서 어떤 플랫폼을 선택할 것인가는 단순히 기술적 우위를 비교하는 것을 넘어, 어떤 생태계 패러다임에 참여할 것인가에 대한 전략적 결정이라 할 수 있다.

| 항목 (Item)            | Autoware (Core/Universe)                               | Baidu Apollo                                        |
| ---------------------- | ------------------------------------------------------ | --------------------------------------------------- |
| **미들웨어**           | ROS 2 / DDS (표준, 유연성)                             | CyberRT (자체 개발, 고성능)                         |
| **설계 철학**          | 모듈성, 확장성 (연구/개발 친화적)                      | 통합성, 즉시성 (상용 배포 친화적)                   |
| **핵심 측위 알고리즘** | NDT, EKF, GNSS                                         | RTK-GNSS, BEV+Lucas-Kanade, ESKF                    |
| **계획/인식 접근법**   | 다양한 오픈소스 모델 통합 지원                         | 다수의 사전 훈련된 모델 제공 (데이터 비공개)        |
| **거버넌스 및 생태계** | 연합형 (Foundation 주도, 다자 참여)                    | 중앙 집중형 (Baidu 주도)                            |
| **주요 장점**          | 높은 유연성, 커스터마이징 용이, 방대한 ROS 생태계 활용 | 높은 초기 완성도, 강력한 기업 지원, 고성능 미들웨어 |
| **주요 단점**          | 초기 통합 복잡성, 기능 안전 별도 확보 필요             | 낮은 커스터마이징 유연성, 특정 기업에 대한 의존성   |


Autoware는 현재 'Autoware Foundation 2.0'이라는 기치 아래 새로운 도약을 준비하고 있다. 이 변화의 핵심은 'AI-First' 전략과 이를 구현하기 위한 차세대 아키텍처로의 진화다.


Autoware Foundation은 새로운 미션으로 "전 세계적으로 신뢰받는 자율주행 시스템을 위한 오픈소스 소프트웨어 구축"을 내세웠다.17 이는 단순히 기술적 완성도를 넘어, 자율주행 기술이 사회적으로 신뢰받고 수용될 수 있도록 하는 데 기여하겠다는 의지를 표명한 것이다. 이러한 비전을 달성하기 위한 핵심 전략이 바로 'AI-First'다. 이는 기존의 정교한 규칙 기반 시스템이 가진 한계를 데이터 기반의 AI 파운데이션 모델을 통해 극복하려는 시도다.17


기존 Autoware의 계획 모듈은 대부분 수많은 규칙(if-else)으로 구성되어 있어, 예측하지 못한 엣지 케이스에 대응하기 어렵고 새로운 시나리오를 추가할 때마다 복잡한 파라미터 튜닝이 필요하다는 한계가 있었다.35 AI-First 전략은 이러한 문제를 해결하기 위해 스택의 핵심 영역에 AI 모델을 적극적으로 도입하는 것을 목표로 한다.

- **주요 모델:**
  - **SceneSeg & Scene3D:** 비전(카메라) 기반의 AI 인식 모델이다. `SceneSeg`는 이미지 내의 모든 픽셀을 의미론적으로 분할(Semantic Segmentation)하여 도로, 차량, 보행자 등을 구분하고, `Scene3D`는 단일 또는 스테레오 이미지로부터 깊이(Depth)를 추정하여 3차원 구조를 복원한다. 이 모델들은 특히 비나 안개 같은 악천후 상황에서 성능이 저하되는 LiDAR 센서의 한계를 보완하여 시스템의 강건성을 높이는 데 핵심적인 역할을 한다.17
  - **EgoPath & EgoLanes:** 주행 가능 경로를 인식하기 위한 AI 모델이다. `EgoLanes`는 이미지에서 차선과 도로 경계를 정밀하게 탐지하는 역할을 한다. `EgoPath`는 한 걸음 더 나아가, 차선이 없는 비정형 도로에서도 주행 가능한 경로를 End-to-End 방식으로 직접 예측한다. Autoware는 이 두 모델의 예측 결과를 앙상블하여 어떠한 도로 환경에서도 안정적으로 주행 경로를 찾아내는 `Path Finder`를 구현하고자 한다.51

이러한 AI 모델들은 현재 `autoware.privately-owned-vehicles`라는 별도의 GitHub 저장소를 중심으로 고속도로 자율주행 파일럿 시스템 개발 프로젝트의 일환으로 활발히 개발되고 있으며, 다양한 공개 데이터셋을 이용한 학습과 성능 감사가 진행 중이다.51


자율주행 기술 연구의 최전선에서는 전통적인 모듈형 아키텍처가 가진 해석 가능성(interpretability)과 End-to-End 모델이 가진 뛰어난 성능 및 확장성(scalability)을 결합하려는 하이브리드 아키텍처에 대한 논의가 활발하다.18

Autoware의 AI-First 전략은 이러한 흐름과 맥을 같이한다. Autoware는 기존의 견고한 7-스택 모듈형 구조를 버리는 대신, 그 구조를 유지하면서 `EgoPath`와 같이 특정 모듈의 내부 구현을 AI 기반 End-to-End 모델로 대체하는 '모듈형 AI(Modular AI)' 접근법을 취하고 있다.51 이는 시스템 전체의 동작을 예측하고 검증하기 어려운 '블랙박스' 문제를 최소화하면서도, AI 모델의 강력한 성능을 선택적으로 수용하려는 매우 실용적인 전략이다.

물론 고전적인 규칙 기반의 기호적 계획(symbolic planning)과 데이터 기반의 AI 모델을 하나의 시스템 안에서 안정적으로 통합하는 것은 여전히 기술적으로 매우 어려운 과제다. AI 모델의 예측 실패를 감지하고 안전하게 시스템을 제어하는 피드백 루프 설계, 모델의 불확실성을 정량화하고 계획에 반영하는 방법, 그리고 AI 모델의 출력을 실제 세계의 물리적 제약과 연결(grounding)하는 문제 등은 앞으로 해결해야 할 핵심 연구 주제다.56

이 'AI-First' 전략은 Autoware의 미래를 결정할 중대한 전환점이다. Autoware의 정체성과도 같았던 투명하고 해석 가능한 모듈형 구조에 AI라는 강력하지만 예측하기 어려운 요소를 통합하는 것은 높은 잠재력만큼이나 큰 위험을 내포한다. 만약 이 하이브리드 접근법이 성공적으로 안착한다면, Autoware는 해석 가능성과 뛰어난 성능을 모두 갖춘 독보적인 자율주행 플랫폼으로 거듭날 것이다. 하지만 실패할 경우, 모듈형 시스템의 복잡성과 End-to-End 모델의 불확실성이라는 양쪽의 단점만을 가지게 될 위험도 존재한다. 이 전략적 방향성의 성공 여부가 향후 10년간 Autoware의 위상을 결정하게 될 것이다.


Autoware는 세계 최고의 자율주행 OSS 프로젝트로서 괄목할 만한 성과를 이루었지만, 상용 제품으로의 길은 여전히 많은 도전 과제에 직면해 있다.


- **기능 안전 (Functional Safety):** Autoware는 그 자체로 ISO 26262와 같은 기능 안전 표준에 대한 인증을 받지 않았다. 상용 차량에 탑재되기 위해서는, 비결정적(non-deterministic)인 Linux 대신 eMCOS와 같은 실시간 운영체제(RTOS)로의 포팅이 필요하며, 요구사항 분석부터 검증까지 V-모델 개발 프로세스에 따른 엄격한 엔지니어링 활동이 수반되어야 한다. 이는 상당한 추가 비용과 전문 인력을 요구하는 높은 장벽이다.58
- **엣지 케이스 처리:** 현재 Autoware의 계획 모듈은 대부분 사전에 정의된 규칙에 기반하여 동작한다. 이로 인해 도로 공사, 예상치 못한 장애물 출현, 다른 운전자의 돌발 행동 등 예측하기 어려운 엣지 케이스에 대한 대응 능력이 부족하다. 이러한 문제들을 해결하기 위해서는 방대한 실제 주행 데이터를 수집하여 엣지 케이스를 발굴하고, 이를 처리하기 위한 로직을 끊임없이 추가하며, 시뮬레이션과 실차 테스트를 반복해야 하는 고된 과정이 필요하다.35
- **MLOps 및 통합의 복잡성:** AI-First 전략이 본격화되면서 MLOps(Machine Learning Operations)의 중요성이 커지고 있다. 학계 연구를 통해 개발된 프로토타입 수준의 AI 모델을 실제 차량에 안정적으로 배포하고 지속적으로 성능을 관리하기 위해서는 데이터 수집, 가공, 학습, 배포, 모니터링으로 이어지는 전체 파이프라인을 구축해야 한다. 이는 ROS 환경과 기존 시스템과의 통합 문제, 툴체인 차이 등 여러 기술적 장벽을 수반하는 복잡한 작업이다.59


- **아키텍처의 복잡성 및 혼란:**
  - Core/Universe 구조가 처음 도입되었을 때, 많은 개발자들이 `Core`의 실체가 무엇인지, 모든 패키지 개발이 `Universe`에서 시작되어야 하는지, 두 저장소의 관계는 어떻게 되는지에 대해 혼란을 겪었다.60
  - 엄격한 기준에 따라 잘 정돈되었던 `Autoware.Auto`와 달리, `Universe` 저장소에는 다양한 배경을 가진 기여자들이 만든 여러 패키지들이 혼재되어 있어 전체적인 프로젝트 레이아웃의 일관성이 부족하다는 비판도 제기되었다.60
- **유지보수 오버헤드:**
  - `autoware_core`, `autoware_universe`, `autoware_launch` 등 기능별로 여러 저장소가 분리되면서, 하나의 기능을 수정하기 위해 여러 저장소에 걸쳐 Pull Request를 생성하고 이를 조율해야 하는 관리의 복잡성이 증가했다.61
  - 사용자 입장에서는 Autoware를 구동하기 위해 특정 Ubuntu 버전, ROS 배포판, CUDA 버전, Python 버전 등을 정확히 맞춰야 하는 의존성 문제가 큰 부담으로 작용한다. 이러한 호환성 문제는 문서에 명확히 기술되지 않은 경우가 많아, 설치 과정에서 많은 시행착오를 유발한다.63
  - LGSVL과 같이 Autoware 개발에 핵심적으로 사용되던 시뮬레이터의 지원이 갑작스럽게 중단되는 사건은 오픈소스 생태계의 안정성에 대한 우려를 낳았다. 커뮤니티에서는 CARLA 등 대체 시뮬레이터로의 전환에 대한 논의가 활발히 이루어지고 있다.64


- **실시간 성능 보장:** ROS 2의 DDS 통신은 유연성이 높은 만큼 특정 상황에서 오버헤드가 발생하여 실시간 성능을 저해할 수 있다. Autoware는 이를 완화하기 위해 여러 노드를 하나의 프로세스에서 실행하는 `ComponentContainer`를 활용하고, 더 나아가 LiDAR 점군과 같이 데이터 처리량이 매우 큰 토픽에 대해서는 ROS 2 통신을 우회하여 zero-copy에 가까운 성능을 내는 `Agnocast`와 같은 제3의 미들웨어를 도입하는 등 다각적인 노력을 기울이고 있다.8
- **이종 하드웨어 지원:** Autoware는 특정 하드웨어에 종속되지 않는 것을 목표로 하지만, 현실적으로 다양한 종류의 CPU, GPU, 센서 조합을 모두 최적으로 지원하는 것은 매우 어려운 일이다. 특히 리소스가 제한된 임베디드 프로세서에서 안정적인 실시간 성능을 확보하기 위해서는 스레드 우선순위, 스케줄링 정책, 멀티코어 활용 방안 등에 대한 세심한 설정과 최적화가 요구된다.58



Autoware Core/Universe 패러다임은 자율주행 오픈소스 소프트웨어가 직면한 본질적인 딜레마, 즉 '빠른 혁신'과 '상용 수준의 안정성'이라는 상충된 두 가지 요구를 해결하기 위한 영리하고 현실적인 아키텍처적 해법이다. 이는 단순히 코드를 두 개의 저장소로 나눈 기술적 분리를 넘어, 학계와 산업계라는 두 주요 기여자 그룹의 서로 다른 문화와 요구사항을 하나의 프로젝트 안에서 조화롭게 만족시키려는 고도의 거버넌스 전략이기도 하다. `Universe`는 아이디어가 자유롭게 실험되고 경쟁하는 혁신의 장을 제공하고, `Core`는 그 결과물 중 가장 가치 있는 것들을 선별하여 담아내는 신뢰의 보루 역할을 한다.


- 학계 및 연구자 (For Researchers):

  Universe를 최신 알고리즘을 신속하게 구현하고 검증하는 테스트베드로 적극 활용해야 한다. 초기 개발 단계부터 Core로의 승격 기준(높은 테스트 커버리지, 명확한 문서화 등)을 염두에 두고 코드 품질을 관리한다면, 연구 결과물이 실제 시스템에 기여하고 그 영향력을 극대화할 수 있는 가능성이 커질 것이다.

- 상용화 도입 기업 (For Commercial Adopters):

  Core를 안정적인 시스템의 기반으로 삼되, 이것이 곧바로 상용 제품이 될 수 없음을 명확히 인지해야 한다. ISO 26262와 같은 기능 안전 요구사항을 충족시키고, 특정 차량 플랫폼과 ODD에 대한 강건성을 확보하기 위해서는 상당한 추가 엔지니어링 노력이 필수적이다. TIER IV의 Pilot.Auto나 Web.Auto와 같은 상용 지원 플랫폼을 활용하거나, eMCOS와 같은 인증된 RTOS로의 포팅을 초기 단계부터 고려하는 것이 현실적인 상용화 전략이 될 수 있다.58

- 모든 기여자 (For All Contributors):

  Autoware Foundation 2.0이 제시한 'AI-First' 비전은 프로젝트의 장기적인 방향성을 명확히 보여준다. 기존의 견고한 모듈형 아키텍처와 새로운 AI 파운데이션 모델을 어떻게 효과적으로 결합할 것인가가 향후 Autoware 생태계의 핵심 경쟁력을 좌우할 것이다. 이 하이브리드 접근법에 대한 깊은 이해를 바탕으로 기여하는 것이 Autoware의 미래를 함께 만들어가는 가장 효과적인 길이 될 것이다.


1. Autoware Overview, accessed July 24, 2025, https://autoware.org/autoware-overview/
2. Autoware: Home Page, accessed July 24, 2025, https://autoware.org/
3. Past, Present and the Future of Autoware, accessed July 24, 2025, https://autoware.org/past-present-and-the-future-of-autoware/
4. How is Autoware Core/Universe different from Autoware.AI and Autoware.Auto? - Autoware Documentation, accessed July 24, 2025, https://autowarefoundation.github.io/autoware-documentation/main/design/autoware-concepts/difference-from-ai-and-auto/
5. Autoware.Auto: Building reliable open-source software for autonomous driving - AWS, accessed July 24, 2025, https://cdck-file-uploads-us1.s3.dualstack.us-west-2.amazonaws.com/flex022/uploads/ros/original/2X/b/b0782f034d6ac9c89cd0e713545b35e246f0be46.pdf
6. Autoware.Auto, accessed July 24, 2025, https://autowarefoundation.gitlab.io/autoware.auto/AutowareAuto/
7. Autoware Foundation General Assembly 2020, accessed July 24, 2025, https://autoware.org/autoware-foundation-general-assembly-2020/
8. Autoware Tutorial at the IEEE Intelligent Vehicles Symposium (IV 2025), accessed July 24, 2025, https://autoware.org/autoware-tutorial-ieee-iv-2025/
9. Introduce True Zero-Copy Publish/Subscribe IPC to Autoware!! (New Middleware Coexisting with ROS 2) / autowarefoundation / Discussion #5835 - GitHub, accessed July 24, 2025, https://github.com/orgs/autowarefoundation/discussions/5835
10. A brief recap on Autoware and the planning component | by Huawei Zhu | Medium, accessed July 24, 2025, https://medium.com/@huawei.zhu/a-brief-recap-on-autoware-and-the-planning-component-f1d8211f40f0
11. Implementing Autoware Core / autowarefoundation / Discussion #5365 - GitHub, accessed July 24, 2025, https://github.com/orgs/autowarefoundation/discussions/5365
12. autoware_core/README.md at main - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware.core/blob/main/README.md
13. autowarefoundation/autoware_core - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware_core
14. autowarefoundation/autoware_universe - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware_universe
15. Architecture overview - Autoware Universe Documentation, accessed July 24, 2025, https://autowarefoundation.github.io/autoware-documentation/main/design/autoware-architecture/
16. [Discussion] Planning Architecture / Issue #419 / autowarefoundation/autoware_ai - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware/issues/1719
17. Autoware Foundation 2.0: Growing Together Toward Scalable Autonomy, accessed July 24, 2025, https://autoware.org/autoware-foundation-2-0-growing-together-toward-scalable-autonomy/
18. CVPR Tutorial End-to-End Autonomy: A New Era of Self-Driving, accessed July 24, 2025, https://cvpr.thecvf.com/virtual/2024/tutorial/23722
19. End-to-end Autonomous Driving: Challenges and Frontiers - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/371953900_End-to-end_Autonomous_Driving_Challenges_and_Frontiers
20. 2021215-Core/Universe and Development Process - AWS, accessed July 24, 2025, https://cdck-file-uploads-us1.s3.dualstack.us-west-2.amazonaws.com/flex022/uploads/ros/original/2X/b/b0a945ca0e302034b4693237ecbd0ad5cc5e6877.pdf
21. The Autoware Foundation Charter, accessed July 24, 2025, https://autoware.org/governance/
22. Autoware Technical Steering Committee - October 2023, accessed July 24, 2025, https://autoware.org/autoware-technical-steering-committee-october-2023/
23. Monthly Technical Steering Committee Update - September 2023 - Autoware, accessed July 24, 2025, https://autoware.org/monthly-technical-steering-committee-update-september-2023/
24. Issues / autowarefoundation/autoware_core / GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware_core/issues
25. Port 'autoware_motion_utils' from Autoware Universe to Autoware Core / Issue #139 / autowarefoundation/autoware_core - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware_core/issues/139
26. Segmentation, accessed July 24, 2025, https://autowarefoundation.gitlab.io/autoware.auto/AutowareAuto/autoware-perception-segmentation-design.html
27. autoware::perception::segmentation::euclidean_cluster Namespace Reference, accessed July 24, 2025, https://autowarefoundation.gitlab.io/autoware.auto/AutowareAuto/namespaceautoware_1_1perception_1_1segmentation_1_1euclidean__cluster.html
28. autoware_universe/perception/autoware_bytetrack/README.md at main - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware.universe/blob/main/perception/autoware_bytetrack/README.md
29. Autoware Universe Documentation, accessed July 24, 2025, https://autowarefoundation.github.io/autoware_universe/main/
30. Algorithm - Autoware Universe Documentation, accessed July 24, 2025, https://autowarefoundation.github.io/autoware.universe/main/perception/autoware_radar_fusion_to_detected_object/docs/algorithm/
31. Autoware.Flex: Human-Instructed Dynamically Reconfigurable Autonomous Driving Systems - arXiv, accessed July 24, 2025, https://arxiv.org/html/2412.16265v1
32. Localization Design - Sign in / GitLab, accessed July 24, 2025, https://autowarefoundation.gitlab.io/autoware.auto/AutowareAuto/localization-design.html
33. Localization with Autoware. What you need to know about paths... | by yodayoda | Map for Robots | Medium, accessed July 24, 2025, https://medium.com/yodayoda/localization-with-autoware-3e745f1dfe5d
34. Autoware Course Lecture 10: State Estimation for Localization - YouTube, accessed July 24, 2025, https://www.youtube.com/watch?v=g2YURb-d9vY
35. Introduction to obstacle avoidance in Autoware | by TIER IV - Medium, accessed July 24, 2025, https://medium.com/tier-iv-tech-blog/introduction-to-obstacle-avoidance-in-autoware-98fc565a6667
36. Behavior Path Planner - Autoware Universe Documentation, accessed July 24, 2025, https://autowarefoundation.github.io/autoware.universe_planning/main/planning/behavior_path_planner/
37. autoware-documentation/docs/design/autoware-architecture/planning/index.md at main / autowarefoundation/autoware-documentation - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware-documentation/blob/main/docs/design/autoware-architecture/planning/index.md
38. PID (Trajectory Follower), accessed July 24, 2025, https://autowarefoundation.gitlab.io/autoware.auto/AutowareAuto/trajectory_follower-pid-design.html
39. PID vs. Other Control Methods: What's the Best Choice? - RealPars, accessed July 24, 2025, https://www.realpars.com/blog/pid-vs-advanced-control-methods
40. PID vs MPC in Autonomous Vehicles: Which one is better? - YouTube, accessed July 24, 2025, https://www.youtube.com/watch?v=ONlYfX0xxvI
41. Longitudinal (PID) and Lateral (MPC) Motion Control of Autonomous Vehicle - YouTube, accessed July 24, 2025, https://www.youtube.com/watch?v=n6HP66KTkJ8
42. Open-Source Autonomous Driving Software Platforms: Comparison of Autoware and Apollo | Request PDF - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/388633666_Open-Source_Autonomous_Driving_Software_Platforms_Comparison_of_Autoware_and_Apollo
43. Open-Source Autonomous Driving Software Platforms: Comparison of Autoware and Apollo, accessed July 24, 2025, https://arxiv.org/html/2501.18942v1
44. Open-Source Autonomous Driving Software Platforms: Comparison of Autoware and Apollo | AI Research Paper Details - AIModels.fyi, accessed July 24, 2025, https://www.aimodels.fyi/papers/arxiv/open-source-autonomous-driving-software-platforms-comparison
45. Performance of Open Autonomous Vehicle Platforms: Autoware and Apollo - ResearchGate, accessed July 24, 2025, https://www.researchgate.net/publication/339910004_Performance_of_Open_Autonomous_Vehicle_Platforms_Autoware_and_Apollo
46. One Stack to Rule them All: To Drive Automated Vehicles, and Reach for the 4th Level, accessed July 24, 2025, https://arxiv.org/html/2404.02645v1
47. Members - Autoware, accessed July 24, 2025, https://autoware.org/about/members/
48. Companies unite on autonomous car OS to challenge Google : r/SelfDrivingCars - Reddit, accessed July 24, 2025, https://www.reddit.com/r/SelfDrivingCars/comments/a5hi4v/companies_unite_on_autonomous_car_os_to_challenge/
49. Autoware-Challenge-2024 のコピー, accessed July 24, 2025, https://autoware.org/wp-content/uploads/2024/05/Autoware-Challenge-2024.pdf
50. Autoware Foundation Newsletter: January to June Recap Edition!, accessed July 24, 2025, https://autoware.org/autoware-foundation-monthly-newsletter-issue-1/
51. autowarefoundation/autoware.privately-owned-vehicles: An open-source autonomous highway pilot system for privately owned vehicles - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware.privately-owned-vehicles
52. Manual Data Audit - EgoPath Neural Network / Issue #67 / autowarefoundation/autoware.privately-owned-vehicles - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware.privately-owned-vehicles/issues/67
53. Privately Owned Vehicle Work Group Meeting - 2025/04/07 - Slot 1 / autowarefoundation / Discussion #5992 - GitHub, accessed July 24, 2025, https://github.com/orgs/autowarefoundation/discussions/5992
54. Pull requests / autowarefoundation/autoware.privately-owned-vehicles - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware.privately-owned-vehicles/pulls
55. Integrating Modular Pipelines with End-to-End Learning: A Hybrid Approach for Robust and Reliable Autonomous Driving Systems - MDPI, accessed July 24, 2025, https://www.mdpi.com/1424-8220/24/7/2097
56. CoPAL: Corrective Planning of Robot Actions with Large Language Models - arXiv, accessed July 24, 2025, https://arxiv.org/html/2310.07263v3
57. Grounding Classical Task Planners via Vision-Language Models - arXiv, accessed July 24, 2025, https://arxiv.org/pdf/2304.08587
58. Accelerating Autoware Mass Production: Your Path to Autonomous Driving with eMCOS, accessed July 24, 2025, https://autoware.org/accelerating-autoware-mass-production-your-path-to-autonomous-driving-with-emcos/
59. AWML: An Open-Source ML-based Robotics Perception Framework to Deploy for ROS-based Autonomous Driving Software - arXiv, accessed July 24, 2025, https://arxiv.org/html/2506.00645v1
60. [Autoware] Universe & Core vs Auto - ROS Answers archive, accessed July 24, 2025, https://answers.ros.org/question/400746/
61. Autoware - the world's leading open-source software project for autonomous driving - GitHub, accessed July 24, 2025, https://github.com/autowarefoundation/autoware
62. Splitting the Autoware.AI repository and changing the organisation - Page 2, accessed July 24, 2025, https://discourse.ros.org/t/splitting-the-autoware-ai-repository-and-changing-the-organisation/8139?page=2
63. Overcoming Autoware-Ubuntu Incompatibility in Autonomous Driving Systems-Equipped Vehicles: Lessons Learned - arXiv, accessed July 24, 2025, https://arxiv.org/html/2410.06492v1
64. Which simulator for Autoware.auto (Core/Universe) would you recommend (for an autoware-ROS2 interface) for perception and vehicle dynamics simulation now that LGSVL is being dicontinued by LG? - ROS Discourse, accessed July 24, 2025, https://discourse.ros.org/t/which-simulator-for-autoware-auto-core-universe-would-you-recommend-for-an-autoware-ros2-interface-for-perception-and-vehicle-dynamics-simulation-now-that-lgsvl-is-being-dicontinued-by-lg/23983
65. Technical Steering Committee (TSC) Meeting #13 Minutes - Autoware - ROS Discourse, accessed July 24, 2025, https://discourse.ros.org/t/technical-steering-committee-tsc-meeting-13-minutes/12168
66. TIER IV Autoware Partner Program: Fueling collaboration and innovation in autonomous driving - Medium, accessed July 24, 2025, https://medium.com/tier-iv-tech-blog/tier-iv-autoware-partner-program-fueling-collaboration-and-innovation-in-autonomous-driving-e804f6322a35


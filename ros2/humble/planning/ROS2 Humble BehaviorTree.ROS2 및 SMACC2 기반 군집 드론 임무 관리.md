[ROS2 Humble 행동(임무) 계획 및 관리](./index.md)
# ROS2 Humble 기반 BehaviorTree.ROS2 및 SMACC2 군집 드론 임무 관리



현대 산업 사회의 핵심 기반 시설인 풍력 터빈, 교량, 댐, 송전탑 등은 시간의 흐름에 따라 노후화되며, 구조적 안정성을 위협하는 결함이 발생할 수 있다. 이러한 고소(高所) 시설물에 대한 주기적이고 정밀한 점검은 안전 확보와 자산 가치 유지를 위해 필수적이다. 그러나 기존의 유인 점검 방식은 막대한 비용과 시간을 소요할 뿐만 아니라, 작업자가 위험한 환경에 직접 노출되어야 하는 근본적인 한계를 내포한다. 군집 드론(Swarm Drones)은 이러한 문제에 대한 혁신적인 해결책을 제시한다. 다수의 드론을 동시에 운용함으로써 넓은 점검 영역을 신속하게 커버하고, 다양한 각도에서 고해상도 데이터를 취득하여 3차원 모델링 및 정밀 결함 분석의 정확도를 획기적으로 향상시킬 수 있다.

하지만 군집 드론을 이용한 고소 시설물 점검 임무는 수많은 기술적 난제를 동반한다. 첫째, 고고도 비행 환경은 예측 불가능한 돌풍과 급변하는 기상 조건에 노출되어 있어 개별 드론의 안정적인 비행 제어뿐만 아니라, 군집 전체의 대형(Formation)을 강건하게 유지하는 고도의 기술을 요구한다. 둘째, 교량 하부나 댐 구조물 근처와 같은 지역에서는 GPS 신호가 불안정하거나 두절될 수 있어, RTK-GPS와 IMU, 비전 센서 데이터를 융합한 정밀 측위 기술이 필수적이다. 셋째, 시설물의 복잡한 구조를 빠짐없이 스캔하기 위해서는 모든 드론이 사전에 계획된 경로를 밀리초(millisecond) 단위로 동기화하여 비행해야 하며, 이는 드론 간의 신뢰성 높은 저지연 통신에 크게 의존한다.1 마지막으로, 임무 수행 중 발생할 수 있는 개별 드론의 고장, 통신 두절, 배터리 부족 등 예기치 못한 상황에 군집 전체가 유연하고 안전하게 대응할 수 있는 자율적인 의사결정 능력이 반드시 필요하다.


이처럼 복잡하고 동적인 임무를 안정적으로 수행하기 위해서는 단순히 사전에 정의된 스크립트를 순차적으로 실행하는 방식으로는 한계가 명확하다. 임무의 각 단계는 성공, 실패, 혹은 예상치 못한 결과로 이어질 수 있으며, 시스템은 이러한 결과에 따라 다음 행동을 동적으로 결정해야 한다. 예를 들어, 목표 지점으로 이동하던 중 강풍으로 인해 대형이 흐트러졌을 경우, 무작정 다음 단계를 진행하는 것이 아니라 대형을 재정비하는 복구 행동을 우선적으로 수행해야 한다. 이러한 동적 의사결정 로직을 하드코딩된 `if-else` 문이나 중첩된 조건문으로 구현할 경우, 시스템의 복잡도가 증가함에 따라 코드는 기하급수적으로 거대해지고 가독성이 떨어져 유지보수가 거의 불가능한 '스파게티 코드'로 전락하게 된다.3

따라서, 임무의 논리적 흐름과 로봇의 행동을 명확하게 분리하고, 복잡한 의사결정 구조를 체계적으로 모델링할 수 있는 고수준 의사결정 프레임워크, 즉 '숙고 프레임워크(Deliberation Framework)'의 도입이 필수적이다. 이상적인 숙고 프레임워크는 다음과 같은 특성을 만족해야 한다.

- **모듈성(Modularity)**: '이륙', '경로 비행', '스캔'과 같은 개별 행동 단위를 독립적인 모듈로 개발하여 재사용성을 극대화해야 한다.
- **반응성(Reactivity)**: 외부 환경의 변화나 내부 상태의 변동을 실시간으로 감지하고, 이에 맞춰 행동 계획을 즉각적으로 수정할 수 있어야 한다.
- **확장성(Scalability)**: 새로운 임무 요구사항이나 복구 행동을 추가할 때, 기존 시스템의 구조를 최소한으로 변경하면서 쉽게 기능을 확장할 수 있어야 한다.
- **강건성(Robustness)**: 예측하지 못한 오류가 발생하더라도 시스템 전체의 붕괴로 이어지지 않고, 정의된 절차에 따라 안전한 상태로 전환하거나 임무를 부분적으로라도 지속할 수 있는 오류 처리 및 복구 메커니즘을 내장해야 한다.


본 보고서는 로봇 운영체제의 표준으로 자리 잡은 ROS2(Robot Operating System 2)의 최신 LTS(Long-Term Support) 버전인 Humble Hawksbill 환경을 기반으로, 고소 시설물 점검 군집 드론 임무를 위한 소프트웨어 아키텍처를 설계하는 방안을 제안하는 것을 목표로 한다. 특히, ROS2 생태계에서 가장 주목받는 두 가지 숙고 프레임워크인 **BehaviorTree.ROS2**와 **SMACC2**를 심층적으로 분석하고, 각각의 프레임워크를 적용한 구체적인 임무 수행 설계안을 제시한다.

보고서는 다음과 같이 구성된다. 먼저 **II장**에서는 군집 드론 시스템의 전체적인 하드웨어 및 소프트웨어 아키텍처를 개괄하고, 통신 네트워크와 제어 패러다임을 정의하며, 전체 임무를 논리적인 단계로 분해한다. **III장**과 **IV장**에서는 각각 BehaviorTree.ROS2와 SMACC2의 핵심 개념을 설명하고, 이를 바탕으로 분해된 임무를 수행하기 위한 상세한 설계안을 커스텀 노드 라이브러리, 계층적 구조 설계, 오류 처리 메커니즘을 포함하여 기술한다. **V장**에서는 두 프레임워크를 모듈성, 반응성, 동시성 관리 등 다양한 기준에 따라 정량적, 정성적으로 비교 분석하고, 이를 통해 도출된 통찰을 바탕으로 두 프레임워크의 장점을 시너지 있게 결합한 최종적인 **하이브리드 아키텍처**를 제안한다. 마지막으로 **VI장**에서는 전체 내용을 요약하고 제안된 하이브리드 아키텍처의 의의와 향후 전망을 논하며 마무리한다. 본 보고서는 단순한 기술 소개를 넘어, 실제 시스템 개발에 직접 적용 가능한 심층적인 기술 청사진을 제공하고자 한다.




군집 드론 시스템은 크게 공중 세그먼트(Aerial Segment)와 지상 세그먼트(Ground Segment)로 구성된다.

- **공중 세그먼트**: 임무를 직접 수행하는 다수의 동형(homogeneous) 드론으로 구성된다. 각 드론은 다음과 같은 핵심 하드웨어를 탑재한다.
  - **Onboard Computer**: ROS2 Humble이 설치된 고성능 임베디드 컴퓨터(예: NVIDIA Jetson 시리즈)로, 모든 자율 비행 및 임무 로직 연산을 수행하는 두뇌 역할을 한다.
  - **Flight Controller (FC)**: PX4 또는 ArduPilot과 같은 오픈소스 비행 제어 펌웨어가 탑재된 비행 컨트롤러로, Onboard Computer로부터 고수준의 제어 명령(예: 목표 속도, 자세)을 받아 모터를 직접 제어하고 비행 안정성을 확보한다.
  - **고정밀 측위 시스템**: RTK-GPS(Real-Time Kinematic GPS)와 고성능 IMU(Inertial Measurement Unit)를 결합하여 센티미터 수준의 정밀한 위치 및 자세 정보를 제공한다.
  - **임무 페이로드**: 3축 짐벌에 장착된 고해상도 카메라, LiDAR, 또는 열화상 카메라 등으로, 시설물의 상태 데이터를 취득한다.
- **지상 세그먼트**: 지상관제국(Ground Control Station, GCS)으로, 운영자가 임무를 계획, 모니터링하고 비상시 개입하는 역할을 수행한다. GCS는 강력한 연산 능력을 갖춘 워크스테이션으로 구성되며, 군집 전체의 상태를 시각화하는 UI를 제공한다.

각 드론의 Onboard Computer에서 실행되는 소프트웨어는 ROS2 노드(Node)의 집합으로 모듈화된다. 각 모듈은 특정 기능을 담당하며, ROS2의 통신 미들웨어를 통해 유기적으로 상호작용한다.

- **인식(Perception)**: 카메라, LiDAR 등 센서 데이터를 처리하여 장애물을 감지하고, 시설물의 특징점을 추출한다.
- **측위(Localization)**: RTK-GPS, IMU, 비전 센서 데이터를 칼만 필터(Kalman Filter) 기반으로 융합하여 드론의 현재 위치와 자세를 정밀하게 추정한다.
- **제어(Control)**: 상위 기획 모듈에서 생성된 경로점을 추종하도록 FC에 전달할 저수준 제어 명령을 생성한다.
- **기획(Planning)**: 전역 경로 계획, 지역 경로 계획, 그리고 본 보고서의 핵심인 고수준 행동 결정을 담당하는 숙고 프레임워크(BehaviorTree 또는 SMACC2)를 포함한다.
- **통신(Communication)**: 다른 드론 및 GCS와의 데이터 교환을 담당한다.


군집 드론의 협력적 임무 수행을 위해서는 드론 간(inter-drone) 및 드론-GCS(drone-to-GCS) 간의 신뢰성 높은 통신이 전제되어야 한다. 본 시스템은 ROS2의 핵심 통신 시스템인 DDS(Data Distribution Service)를 기반으로 무선 Ad-hoc 네트워크(예: Wi-Fi Mesh)를 구성한다.1 DDS는 중앙 서버 없이 각 노드가 직접 데이터를 교환하는 P2P(Peer-to-Peer) 방식을 사용하므로 분산 시스템에 적합하다. 특히, 다수의 노드가 동적으로 네트워크에 참여하고 이탈하는 군집 환경에서는 DDS의 자동 노드 탐지(Discovery) 기능이 매우 유용하다. 그러나 기본 DDS 설정은 무선 환경의 불안정성에 취약할 수 있으므로, 네트워크 안정성 향상을 위해 **Fast DDS Discovery Server** 사용을 적극적으로 고려한다.6 이 방식은 중앙 Discovery Server를 통해 노드 간의 연결 정보를 관리함으로써, 네트워크 트래픽을 줄이고 대규모 군집에서의 연결 안정성을 높일 수 있다. 또한, 군집 행동에 특화된 메시지 타입과 통신 프로토콜을 효율적으로 관리하기 위해 ROS2swarm 패키지에서 제공하는 `communication_interfaces`를 활용하거나 이를 기반으로 커스텀 인터페이스를 정의할 수 있다.7

제어 패러다임은 중앙 집중식 제어와 분산형 제어의 장점을 결합한 **하이브리드 제어 구조**를 채택한다. 이는 임무의 효율성과 시스템의 강건성을 동시에 확보하기 위한 전략적 선택이다.

- **중앙 집중식 계획 (GCS)**: GCS는 임무의 전역적 목표(Global Goal)를 설정하고 최적의 임무 계획을 수립하는 역할을 한다. 예를 들어, 'A 교량의 1번부터 5번 주탑 케이블을 순서대로 점검하라'와 같은 고수준의 명령을 생성하여 군집에 하달한다. 이는 전체 임무의 최적성을 보장하는 역할을 하지만, GCS가 모든 드론의 실시간 경로를 마이크로매니징하지는 않는다. 이는 통신 병목 현상과 단일 장애점(Single Point of Failure) 문제를 야기할 수 있기 때문이다.8
- **분산형 자율 제어 (드론 군집)**: GCS로부터 고수준 임무를 할당받은 드론 군집은 리더 드론을 중심으로 자율적으로 세부 임무를 수행한다. 각 드론은 주변 드론 및 장애물과의 상대 위치를 실시간으로 인지하며, 충돌 회피, 대형 유지, 동기화 비행 등 지역적(local)인 의사결정을 독립적으로 내린다. 이러한 분산 제어는 개별 드론의 고장이나 일시적인 통신 두절 상황에서도 군집 전체가 임무를 지속할 수 있게 하는 강건성과 확장성을 제공한다.10

이 하이브리드 제어 구조의 핵심에는 각 드론에 탑재된 **숙고 프레임워크**가 있다. 이 프레임워크는 GCS로부터 수신한 고수준의 추상적인 목표를 해석하여, 드론 군집이 수행해야 할 구체적이고 협력적인 행동들의 순서로 분해하는 '두뇌' 역할을 수행한다. 즉, 중앙의 '무엇을 할 것인가(what)'라는 지시를 분산된 '어떻게 할 것인가(how)'의 실행으로 변환하는 중추적인 연결고리인 것이다. 따라서 BehaviorTree 또는 SMACC2의 설계는 이러한 이중 계층 제어 구조를 명시적으로 지원하고, 외부의 고수준 목표를 받아들여 강건한 저수준 자율 행동으로 변환하는 능력을 갖추도록 이루어져야 한다.


성공적인 숙고 프레임워크 설계를 위해서는 전체 임무를 명확하고 독립적인 단위 행동(Behavioral Primitive)으로 분해하는 과정이 선행되어야 한다. 이렇게 분해된 각 단계는 이후 Behavior Tree의 액션 노드(Action Node) 또는 SMACC2의 리프 상태(Leaf State)에 직접적으로 매핑되어 시스템 설계의 근간을 이룬다. 고소 시설물 점검 임무는 다음과 같은 주요 단계로 분해될 수 있다.

- **Phase 1: 전개 및 비행 전 점검 (Deployment & Pre-flight Check)**
  - GCS 운영자가 임무 시작 명령을 내리면, 모든 드론은 이륙하여 사전에 정의된 초기 위치에서 안정적인 호버링 상태에 진입한다.
  - 각 드론은 자체 진단(Self-diagnosis) 절차를 수행하여 GPS 수신 상태, 배터리 전압, 통신 연결, 센서 및 액추에이터의 정상 작동 여부를 확인한다. 모든 드론이 '준비 완료' 상태를 보고하면 다음 단계로 진행한다.
- **Phase 2: 점검 구역으로 이동 (Transit to Inspection Area)**
  - 군집은 리더-팔로워(Leader-Follower) 또는 가상 구조(Virtual Structure)와 같은 대형 제어 알고리즘을 사용하여 안정적인 편대를 유지하며 목표 점검 구역으로 이동한다.12
  - 이동 중 실시간으로 장애물을 감지하고 회피하며, 드론 간의 안전거리를 지속적으로 유지한다.
- **Phase 3: 점검 대형 구성 (Formation Assembly)**
  - 점검 대상 시설물 근처의 안전한 상공에 도달하면, 군집은 정밀 스캔에 최적화된 특정 대형(예: 수평 라인, 수직 라인, 매트릭스)으로 재구성된다.
  - 이 단계에서는 드론 간의 정밀한 상대 위치 제어가 매우 중요하다.
- **Phase 4: 동기화된 점검 스캔 (Synchronized Inspection Scan)**
  - 모든 드론이 동기화된 타이밍에 맞춰 사전에 할당된 경로를 따라 비행하며 시설물을 스캔한다. 예를 들어, 교량 케이블을 점검할 경우, 여러 드론이 케이블을 중심으로 일정한 간격을 유지하며 함께 상승 또는 하강하면서 360도 데이터를 취득한다.
  - 각 드론은 담당 구역의 데이터를 촬영/스캔하는 동시에, 이미지의 초점, 노출, LiDAR 포인트 밀도 등 데이터 품질을 실시간으로 검증한다. 품질 미달 시 해당 구역을 재스캔하는 로직이 포함될 수 있다.
- **Phase 5: 데이터 통합 및 전송 (Data Consolidation & Offload)**
  - 모든 구역의 스캔이 완료되면, 군집은 안전한 위치로 이동하여 수집된 데이터를 통합한다.
  - 데이터는 리더 드론을 통해 GCS로 일괄 전송되거나, 각 드론이 직접 GCS와 통신하여 전송할 수 있다. 대용량 데이터 전송을 위해 통신 링크의 안정성이 확보된 지점에서 수행된다.
- **Phase 6: 기지로 복귀 및 회수 (Return to Launch & Recovery)**
  - 임무를 성공적으로 완료했거나, 임무 수행 중 배터리 부족, 기상 악화, 시스템 오류 등 비상 상황이 발생했을 경우, 모든 드론은 안전하게 이륙 지점으로 복귀(Return to Launch, RTL)하여 순차적으로 착륙한다.

이러한 단계적 분해는 복잡한 전체 임무를 관리 가능한 작은 단위로 나누어, 각 단위의 성공과 실패를 명확하게 정의하고 이에 따른 조건부 로직을 체계적으로 설계할 수 있는 기반을 제공한다.




행동 트리(Behavior Tree, BT)는 비디오 게임 AI에서 시작하여 로봇 공학 분야로 확장된 강력한 임무 계획 및 실행 패러다임이다.14 BT는 복잡한 행동을 '노드(Node)'들의 계층적인 트리 구조로 표현한다. 시스템은 일정한 주기(예: 10Hz)로 트리의 루트(Root) 노드에 '틱(Tick)' 신호를 보내고, 이 틱은 정해진 규칙에 따라 자식 노드로 전파된다. 각 노드는 틱을 받을 때마다 실행되며, 자신의 상태를 `SUCCESS`, `FAILURE`, `RUNNING` 중 하나로 부모에게 반환한다.

- **노드 유형**:
  - **Action Node**: 시스템의 실제 행동을 수행하는 노드. '이륙', '경로 따라가기' 등이 해당된다. 비동기적인 작업의 경우, 작업이 진행 중일 때는 `RUNNING`, 완료되면 `SUCCESS`, 실패하면 `FAILURE`를 반환한다.
  - **Condition Node**: 시스템의 특정 상태를 확인하는 노드. '배터리가 부족한가?', '목표에 도달했는가?' 등이 해당된다. `SUCCESS`(참) 또는 `FAILURE`(거짓)를 즉시 반환한다.
  - **Control Flow Node**: 자식 노드들을 어떤 순서와 조건으로 실행할지 결정하는 노드. 대표적으로 `Sequence` (순차 실행, AND 논리), `Fallback` (또는 `Selector`, 순차 실행, OR 논리), `Parallel` (병렬 실행) 등이 있다.14
  - **Decorator Node**: 자식 노드의 행동을 수정하거나 제어하는 노드. `Inverter` (결과 반전), `Retry` (실패 시 재시도), `RateController` (실행 주기 제어) 등이 있다.15

`BehaviorTree.CPP`는 C++로 구현된 BT 라이브러리의 표준으로, 모듈성과 재사용성을 핵심 철학으로 삼는다.16 각 노드는 독립적인 빌딩 블록으로 구현되어, C++ 코드를 수정하지 않고도 XML 기반의 스크립트를 통해 다양한 행동 트리를 조합하고 동적으로 로드할 수 있다.


`BehaviorTree.ROS2`는 `BehaviorTree.CPP`를 ROS2 환경에서 원활하게 사용할 수 있도록 해주는 핵심적인 래퍼(wrapper) 패키지 모음이다.18 이 패키지는 ROS2의 주요 통신 메커니즘을 BT 노드로 쉽게 캡슐화할 수 있는 템플릿 클래스들을 제공한다.

- **ROS2 Action 연동**: ROS2의 Action은 장시간 실행되는 비동기 작업을 위한 통신 방식으로, 목표 전송, 피드백 수신, 결과 확인, 그리고 **중단(abort)** 기능을 지원한다. 이는 BT의 `Action` 노드 개념과 완벽하게 부합한다. `BehaviorTree.ROS2`의 `RosActionNode` 템플릿을 상속받아 클래스를 구현하면, '목표 설정', '결과 수신 시 처리', '피드백 수신 시 처리', '실패 시 처리' 등의 콜백 함수만 정의하여 손쉽게 ROS2 Action Client 역할을 하는 BT 노드를 만들 수 있다.18 특히 Action의 중단 기능은 상위 제어 노드(예: `Fallback`)가 다른 행동으로 전환할 때 현재 실행 중인 노드를 즉시 멈추게 하는 반응형(reactive) 행동을 구현하는 데 필수적이다.
- **ROS2 Service/Topic 연동**: `RosServiceNode`, `RosTopicSubNode`, `RosTopicPubNode` 등의 템플릿을 통해 ROS2 Service Client, Topic Subscriber, Publisher 기능 또한 간편하게 BT 노드로 구현할 수 있다. 예를 들어, `Condition` 노드는 Topic을 구독하여 특정 조건이 충족되었는지 확인하는 방식으로 구현할 수 있다.


`BehaviorTree.ROS2`를 적용한 시스템 아키텍처는 명확한 역할 분담을 기반으로 한다.18

1. **Coordinator Node**: GCS 또는 군집의 리더 드론에서 실행되는 중앙 집중화된 ROS 노드이다. 이 노드는 전체 임무를 정의하는 마스터 BT XML 파일을 로드하고, 주기적으로 틱을 발생시켜 BT를 실행하는 역할을 한다. 즉, 군집 전체의 고수준 의사결정을 총괄하는 '지휘 본부'이다. `TreeExecutionServer`와 같은 컴포넌트를 활용하면 외부 ROS 노드에서 Action 인터페이스를 통해 특정 BT의 실행을 요청하고 결과를 받을 수 있다.19
2. **Service-Oriented Components**: 나머지 드론들, 또는 리더 드론 내의 저수준 기능 모듈들은 '서비스 지향' 컴포넌트로 동작한다. 이들은 Coordinator의 BT 노드에서 호출하는 ROS2 Action Server, Service Server, Publisher/Subscriber 역할을 수행한다. 예를 들어, BT의 `ArmAndTakeoff` 노드가 `drone_swarm_control/takeoff` Action을 호출하면, 모든 드론에 구현된 해당 Action Server가 그 요청을 받아 실제 이륙 동작을 수행하고 결과를 보고한다. 이 구조는 고수준의 의사결정 로직(BT)과 저수준의 실제 실행 로직(ROS2 노드)을 명확하게 분리하여 시스템의 모듈성과 유지보수성을 극대화한다.


임무 프로파일(II.B)을 효과적으로 수행하기 위해, 재사용 가능한 커스텀 BT 노드 라이브러리를 정의해야 한다. 각 노드는 특정 기능을 수행하는 ROS2 인터페이스를 캡슐화하며, `nav2_behavior_tree` 패키지의 `BtActionNode` 템플릿을 활용하여 효율적으로 구현할 수 있다.21 다음 표는 본 임무를 위해 제안되는 핵심 커스텀 노드 라이브러리를 정의한다. 이 라이브러리는 개발자가 임무 BT를 구성하기 위한 기본적인 빌딩 블록 역할을 한다.

| 노드 이름            | 유형      | 설명                                                         | ROS2 인터페이스                              |
| -------------------- | --------- | ------------------------------------------------------------ | -------------------------------------------- |
| `IsMissionAvailable` | Condition | GCS로부터 새로운 임무가 수신되었는지 블랙보드 또는 특정 토픽을 통해 확인한다. | `mission_goal` Topic Subscriber              |
| `AcceptMission`      | Action    | GCS로부터 수신된 임무 목표(점검 대상, 경로 등)를 파싱하여 BT 내부 데이터 저장소인 블랙보드(Blackboard)에 저장한다. | `accept_mission` Service Client              |
| `ArmAndTakeoff`      | Action    | 군집 내 모든 드론에게 시동(Arm) 및 이륙을 명령하고, 지정된 초기 고도까지 상승하여 호버링하도록 지시한다. | `drone_swarm_control/takeoff` Action Client  |
| `FlyInFormation`     | Action    | 블랙보드에 저장된 목표 지점까지 지정된 대형(예: V자, 라인)을 유지하며 비행한다. 비행 중 피드백으로 현재 위치와 대형 유지 오차를 보고한다. | `drone_swarm_control/navigate` Action Client |
| `CheckFormation`     | Condition | 군집 내 모든 드론이 목표 대형 및 위치에 허용 오차 범위 내로 도달했는지 확인한다. | `swarm_status` Topic Subscriber              |
| `ExecuteSyncScan`    | Action    | 블랙보드에 정의된 동기화된 스캔 패턴을 실행하도록 명령한다. 모든 드론이 동시에 스캔 경로 비행을 시작한다. | `drone_swarm_control/scan` Action Client     |
| `IsBatteryLow`       | Condition | 군집 내 드론 중 하나라도 배터리 잔량이 설정된 임계값(예: 20%) 이하로 떨어졌는지 확인한다. | `swarm_status` Topic Subscriber              |
| `ReturnToLaunch`     | Action    | 임무를 중단하고 모든 드론에게 이륙 지점으로 안전하게 복귀하도록 명령한다. | `drone_swarm_control/rtl` Action Client      |


전체 임무의 복잡성을 관리하고 가독성과 재사용성을 높이기 위해, 여러 개의 서브트리(Subtree)를 활용한 계층적 BT 설계가 필수적이다.24 Groot2와 같은 시각적 편집 도구를 사용하면 이러한 계층적 구조를 직관적으로 설계하고 디버깅할 수 있다.14

- **루트 트리 (Root Tree)**: 전체 BT의 최상위에는 `ReactiveFallback` 제어 노드를 배치한다. 이 노드는 자식 노드를 왼쪽부터 순서대로 틱하며, `RUNNING` 또는 `SUCCESS`를 반환하는 첫 번째 자식을 만나면 나머지 자식은 틱하지 않는다. 이 구조의 첫 번째 자식으로 비상 상황 처리 서브트리를 배치하면, 매 틱마다 비상 상황(예: GCS의 비상 정지 명령, 심각한 시스템 오류)을 최우선으로 확인하고 즉각 대응할 수 있다. 비상 상황이 아닐 경우(`FAILURE` 반환), 두 번째 자식인 주 임무 수행 서브트리로 제어가 넘어간다.

- **주 임무 수행 서브트리 (Mission Execution Subtree)**: 이 서브트리는 `Sequence` 노드를 중심으로 구성되어 임무 프로파일의 전체 흐름, 즉 '임무 수신 → 이륙 → 이동 → 검사 → 복귀'를 순차적으로 제어한다. `Sequence` 노드는 모든 자식이 `SUCCESS`를 반환해야만 `SUCCESS`를 반환하므로, 각 단계가 성공적으로 완료되어야 다음 단계로 넘어가는 논리를 자연스럽게 구현한다.

  ```XML
  <root main_tree_to_execute="MainTree">
    <BehaviorTree ID="MainTree">
      <ReactiveFallback name="Root">
        <SubTree ID="EmergencyHandler"/>
        <Sequence name="MissionExecution">
          <IsMissionAvailable/>
          <AcceptMission/>
          <SubTree ID="TakeoffAndTransit"/>
          <SubTree ID="InspectionPhase"/>
          <SubTree ID="ReturnAndLand"/>
        </Sequence>
      </ReactiveFallback>
    </BehaviorTree>
  </root>
  ```

- **검사 단계 서브트리 (Inspection Subtree)**: 동기화된 스캔 단계는 '검사 대형 유지'와 '스캔 경로 비행'이라는 두 가지 행동이 동시에, 그리고 상호 의존적으로 수행되어야 하는 복잡한 로직을 가진다. Nav2의 `PipelineSequence` 24에서 영감을 받은 커스텀 제어 노드를 사용하거나  `Parallel` 노드를 활용하여 이를 구현할 수 있다. 예를 들어, `Parallel` 노드의 성공 조건을 '모든 자식 성공'으로 설정하고, 첫 번째 자식으로 '스캔 경로 비행'(`ExecuteSyncScan`)을, 두 번째 자식으로 '주기적인 대형 상태 확인'(`RateController(hz=5) -> CheckFormation`)을 배치한다. 이렇게 하면 스캔을 수행하는 내내 5Hz 주기로 대형 유지 여부를 지속적으로 확인하고, 만약 대형이 깨지면 `CheckFormation`이 `FAILURE`를 반환하여 `Parallel` 노드 전체가 실패하게 되어 즉각적인 복구 로직으로 전환할 수 있다.


BT의 계층적 구조는 정교하고 강건한 오류 처리 및 복구 메커니즘을 설계하는 데 매우 강력한 패러다임을 제공한다. 이는 단순한 예외 처리를 넘어, 실패의 맥락(context)에 따라 다른 수준의 복구 전략을 적용하는 다층적(tiered) 접근을 가능하게 한다. ROS2 Nav2 스택의 복구 트리 설계는 이러한 접근법의 우수한 사례이며, 본 임무 설계에 적극적으로 벤치마킹한다.15

이러한 다층적 오류 처리 능력은 BT에 기능을 추가하는 방식이 아니라, BT의 계층적 실행 구조 자체에서 비롯되는 고유한 특성이다. 로봇 공학의 복잡한 문제들은 단순한 성공/실패 이분법으로 모델링하기 어렵다. BT는 `RecoveryNode`와 같은 제어 흐름을 통해 실패를 국소적으로 처리할 기회를 제공함으로써 훨씬 더 정교한 실패 모델을 자연스럽게 구현한다. 예를 들어, `FlyInFormation` 노드가 돌풍으로 인해 일시적으로 실패했을 때, 상위의 `RecoveryNode`는 이 실패를 '잡아서' 전체 임무를 중단시키는 대신, '대형 재정비'라는 국소적 복구 행동을 시도한다. 오직 이 국소적 복구가 반복적으로 실패했을 때에만, 해당 서브트리 전체가 최종적인 `FAILURE`를 반환하여, 더 상위 수준의 시스템 전역 복구 로직을 발동시킨다. 따라서 본 군집 드론 시스템의 BT 설계는 이러한 다층적 복구 구조를 핵심 원칙으로 삼아, 항법, 대형 제어, 데이터 수집 등 모든 핵심 기능에 대해 특화된 국소적 및 전역적 복구 서브트리를 체계적으로 구현해야 한다.

- **국소적 복구 (Contextual Recovery)**: 특정 행동 수행 중에 발생하는 예측 가능하고 해결 가능한 오류를 처리한다. `RecoveryNode` (Nav2에서 제공하는 커스텀 제어 노드) 또는 `Fallback`과 `Sequence`의 조합으로 구현할 수 있다. `RecoveryNode`는 첫 번째 자식(주 행동)을 실행하고, 만약 `FAILURE`를 반환하면 두 번째 자식(복구 행동)을 실행한다.
  - **예시**: `FlyInFormation` 액션이 장애물로 인해 경로 계획에 실패하면, `RecoveryNode`는 '현재 위치에서 호버링하며 경로 재계획'(`HoverAndReplan`)이라는 복구 액션을 시도한다.
  - `number_of_retries` 파라미터를 설정하여, 복구 행동을 지정된 횟수만큼만 시도하고 계속 실패할 경우 최종적으로 `FAILURE`를 반환하여 무한 루프에 빠지는 것을 방지한다.24
- **시스템 전역 복구 (System-Level Recovery)**: 국소적 복구로도 해결되지 않는 심각한 오류가 발생하여 주 임무 서브트리 전체가 `FAILURE`를 반환했을 때 실행된다. 이는 루트 트리의 `ReactiveFallback`에 의해 제어가 넘어가는 별도의 `Recovery` 서브트리로 구현된다.
  - 이 서브트리는 `RoundRobin` (자식을 순환하며 실행) 또는 `Fallback` 노드를 사용하여 단계적인 시스템 전역 복구 절차를 수행한다.24
  - **예시**:
    1. **1단계**: '모든 드론 현재 위치 정지 및 GCS에 경고' (`HoldPositionAndAlert`)
    2. **2단계**: '안전 고도로 상승하여 대형 재구성 시도' (`AscendAndReform`)
    3. **3단계**: '임무 중단 및 기지로 복귀' (`ReturnToLaunch`)
  - 각 복구 단계가 성공하면, 다시 주 임무 서브트리를 시도할지, 아니면 즉시 복귀할지를 결정하는 로직을 추가하여 유연성을 높일 수 있다.




SMACC2(State Machine Asynchronous C++ for ROS2)는 복잡한 다중 컴포넌트 로봇의 행동을 제어하기 위해 설계된 고성능 C++ 상태 머신 라이브러리이다.25 전통적인 유한 상태 머신(Finite State Machine, FSM)과 달리, SMACC2는 다음과 같은 핵심 개념을 통해 현대 로봇 시스템의 요구사항을 충족시킨다.

- **이벤트 기반(Event-Driven)**: SMACC2의 모든 상태 전이(State Transition)는 '이벤트(Event)'에 의해 촉발된다.26 시스템은 특정 상태에 머물러 있다가, 외부(센서 데이터, ROS 메시지 수신) 또는 내부(타이머, 작업 완료)로부터 특정 이벤트가 발생하면, 사전에 정의된 규칙에 따라 다음 상태로 전이된다. 이는 주기적으로 조건을 확인하는 폴링(polling) 방식에 비해 훨씬 효율적이고 반응적이다.
- **비동기(Asynchronous)**: SMACC2는 Boost Statechart 라이브러리를 기반으로 하며, 비동기 스레딩 모델을 지원한다.28 이를 통해 장시간 소요되는 작업(예: 내비게이션, 데이터 처리)이 상태 머신의 주 로직을 차단(blocking)하지 않고 병렬적으로 수행될 수 있다.
- **계층적 구조(Hierarchical)**: 상태 내에 또 다른 상태 머신을 포함하는 계층적 상태 머신(Hierarchical State Machine, HSM)을 지원한다. 이를 통해 복잡한 행동을 '모드 상태(Mode State)', '슈퍼 상태(Super State)', '리프 상태(Leaf State)' 등으로 구조화하여 관리할 수 있다.29
- **핵심 구성 요소**:
  - **State**: 시스템이 가질 수 있는 특정 조건 또는 행동의 집합. 오직 가장 하위 레벨의 '리프 상태'만이 로봇의 하드웨어를 직접 제어하는 로직을 포함한다.
  - **Client**: 상태 머신의 전체 생명주기 동안 유지되는 객체로, 외부 ROS 노드(예: Nav2 서버)와의 통신을 관리하는 등 영속적인 기능을 담당한다. 주요 이벤트 발생원이다.26
  - **Client Behavior**: 특정 상태에 진입했을 때만 활성화되고, 상태를 벗어나면 소멸되는 객체. 상태에 특화된 행동을 구현하는 데 사용된다.
  - **State Reactor**: 이벤트를 입력받아 새로운 이벤트를 출력하는 객체. 여러 개의 저수준 이벤트를 조합하여 하나의 고수준 이벤트를 생성하는 등 복잡한 이벤트 로직을 처리한다.


SMACC2를 다른 상태 머신 라이브러리와 구분 짓는 가장 독창적이고 강력한 개념은 **직교성(Orthogonality)**이다.30 이는 복잡한 로봇이 사실상 여러 독립적인 하위 시스템들의 집합체라는 사실에 착안한 설계 철학이다. 예를 들어, 점검 드론은 '비행 제어 시스템', '짐벌 카메라 시스템', '통신 시스템', '전원 관리 시스템' 등 동시에 독립적으로 작동하는 여러 컴포넌트로 구성된다.

전통적인 FSM으로 이를 모델링하려면, 각 하위 시스템의 모든 상태 조합을 개별적인 상태로 정의해야 한다. 예를 들어, '비행 중-카메라 스캔 중', '호버링 중-카메라 스캔 중', '비행 중-카메라 대기 중' 등 상태의 수가 기하급수적으로 폭발하여 관리가 불가능해진다(State Explosion Problem).3

SMACC2의 직교성은 이 문제를 '직교 컴포넌트(Orthogonal Component)'라는 개념을 통해 해결한다.4 각 하위 시스템을 독립적인 상태 머신으로 간주되는 Orthogonal로 모델링하는 것이다.

- `OrFlightController`: 비행 상태(`Armed`, `Flying`, `Hovering`)를 관리.
- `OrGimbalCamera`: 카메라 상태(`Stowed`, `Scanning`, `Idle`)를 관리.

이 Orthogonal들은 주 임무 상태 머신과 병렬적으로, 그리고 독립적으로 실행된다. 주 임무 상태 머신의 `St_PerformScan` 상태는 비행 시스템이 '어떻게' 비행하는지에 대해 신경 쓸 필요가 없다. 단지 `OrFlightController`가 스캔이 가능한 상태(예: `Flying` 또는 `Hovering`)에 있는지 확인하고, `OrGimbalCamera`에 '스캔 시작' 이벤트를 보내기만 하면 된다. 이처럼 직교성은 로봇 **내부의 복잡성과 동시성**을 체계적으로 관리하는 데 매우 탁월한 메커니즘을 제공하며, 이는 SMACC2의 핵심적인 강점이다.


SMACC2는 ROS2를 위해 특별히 개발되었으며, ROS2의 통신 방식을 네이티브하게 지원한다.25

`smacc2_client_library`는 Nav2, MoveIt2와 같은 주요 ROS2 패키지를 SMACC2의 클라이언트로 즉시 사용할 수 있도록 사전 구현된 라이브러리를 제공하여 통합 개발을 용이하게 한다.25 예를 들어, 

`nav2z_client`를 사용하면, 개발자는 Nav2의 복잡한 Action 인터페이스를 직접 다룰 필요 없이, `Cb_Navigate`와 같은 간단한 클라이언트 행동을 호출하고 `Ev_NavSuccess`, `Ev_NavFailure`와 같은 이벤트를 받아 처리하기만 하면 된다.


SMACC2의 설계 철학에 따라, 점검 드론의 주요 하위 시스템을 다음과 같이 직교 컴포넌트로 분해하여 모델링한다. 각 Orthogonal은 상태 머신 전체 생명주기 동안 영속적으로 존재하며, 관련된 Client와 Client Behavior들을 포함하는 컨테이너 역할을 한다.26

- **`OrFlightController` (비행 제어 Orthogonal)**
  - **역할**: 드론의 물리적인 비행 상태와 이동을 총괄한다.
  - **내부 상태**: `St_Disarmed`, `St_Armed`, `St_TakingOff`, `St_HoldingPosition`, `St_InTrajectory` 등.
  - **포함된 클라이언트**: `nav2z_client` (ROS2 Navigation2 스택과 연동하여 경로 계획 및 추종을 담당).
  - **주요 이벤트**: `Ev_AltitudeReached`, `Ev_NavSuccess`, `Ev_NavFailure` 등을 발생시켜 주 임무 상태 머신에 비행 상태 변화를 알린다.
- **`OrGimbalCamera` (짐벌 카메라 Orthogonal)**
  - **역할**: 짐벌과 카메라 페이로드의 상태를 관리하고 데이터 취득을 제어한다.
  - **내부 상태**: `St_Stowed` (보관 상태), `St_Aiming` (목표 조준), `St_Scanning` (스캔 중), `St_Capturing` (촬영 중).
  - **포함된 클라이언트**: 카메라 드라이버, 짐벌 제어기와 통신하는 ROS2 클라이언트.
  - **주요 이벤트**: `Ev_ScanComplete`, `Ev_CaptureSuccess`, `Ev_GimbalError` 등을 발생시킨다.
- **`OrCommunicationLink` (통신 링크 Orthogonal)**
  - **역할**: 다른 드론 및 GCS와의 무선 통신 링크 상태를 모니터링한다.
  - **내부 상태**: `St_Connected`, `St_Degraded` (신호 약화), `St_Lost` (연결 두절).
  - **포함된 클라이언트**: 네트워크 상태를 모니터링하는 ROS2 노드와 연동.
  - **주요 이벤트**: `Ev_CommsLost`, `Ev_CommsRestored`와 같은 치명적인 이벤트를 발생시켜 비상 대응 로직을 촉발한다.
- **`OrPowerManagement` (전원 관리 Orthogonal)**
  - **역할**: 배터리 상태를 지속적으로 모니터링한다.
  - **내부 상태**: `St_Nominal`, `St_Low`, `St_Critical`.
  - **포함된 클라이언트**: 배터리 상태 토픽(`sensor_msgs/BatteryState`)을 구독하는 클라이언트.
  - **주요 이벤트**: `Ev_LowBattery`, `Ev_CriticalBattery` 이벤트를 발생시켜 복귀 또는 비상 착륙 절차를 시작하도록 한다.


임무 프로파일(II.B)에 따라, 다음과 같은 계층적 상태 머신(HSM)을 설계한다.29

- **최상위 모드 상태 (Superstates / Mode States)**: 임무의 가장 큰 국면을 나타내는 상태들이다.
  - `Ms_Ground`: 이륙 전 대기 및 점검 상태를 포함하는 모드.
  - `Ms_MissionExecution`: 실제 비행 및 점검 임무를 수행하는 모든 상태를 포함하는 모드.
  - `Ms_Emergency`: 비상 상황 발생 시 진입하는 모드로, 모든 정상 임무를 중단하고 안전 조치를 최우선으로 수행한다.
- **리프 상태 (Leaf States)**: `Ms_MissionExecution` 모드 내에서 실제 작업을 수행하는 구체적인 상태들이다. 오직 리프 상태만이 Orthogonal에 접근하여 하드웨어를 제어하는 클라이언트 행동(Client Behavior)을 활성화할 수 있다.
  - `St_Idle`: 새로운 임무를 대기하는 초기 상태.
  - `St_Takeoff`: 이륙을 수행하는 상태. `OrFlightController`의 `Cb_Takeoff` 행동을 활성화한다.
  - `St_FlyToTarget`: 목표 지점으로 비행하는 상태. `OrFlightController`의 `Cb_Navigate` 행동을 활성화한다.
  - `St_PerformScan`: 동기화된 스캔을 수행하는 상태. `OrFlightController`의 `Cb_ExecuteSyncPath`와 `OrGimbalCamera`의 `Cb_StartGimbalScan` 행동을 동시에 활성화한다.
  - `St_ReturnHome`: 기지로 복귀하는 상태. `OrFlightController`의 `Cb_Navigate`를 다시 활성화한다.
- **상태 전이 (State Transitions)**: 상태 간의 전이는 명시적으로 정의된 이벤트에 의해서만 발생한다. 다음 표는 주요 상태 전이 로직을 간략하게 보여준다.

| 현재 상태        | 트리거 이벤트                            | 다음 상태           | 활성화되는 주요 클라이언트 행동                     |
| ---------------- | ---------------------------------------- | ------------------- | --------------------------------------------------- |
| `St_Idle`        | `Ev_MissionReceived` (GCS로부터)         | `St_Takeoff`        | `Cb_AcknowledgeMission`                             |
| `St_Takeoff`     | `Ev_AltitudeReached<OrFlightController>` | `St_FlyToTarget`    | `Cb_StartNavigation<nav2z_client>`                  |
| `St_FlyToTarget` | `Ev_NavSuccess<nav2z_client>`            | `St_PerformScan`    | `Cb_StartGimbalScan`, `Cb_ExecuteSyncPath`          |
| `St_PerformScan` | `Ev_ScanComplete<OrGimbalCamera>`        | `St_ReturnHome`     | `Cb_StowGimbal`, `Cb_StartNavigation<nav2z_client>` |
| `(모든 상태)`    | `Ev_CriticalBattery<OrPowerManagement>`  | `St_EmergencyRTL`   | `Cb_BroadcastEmergency`, `Cb_StartNavigation`       |
| `(모든 상태)`    | `Ev_CommsLost<OrCommunicationLink>`      | `St_EmergencyHover` | `Cb_HoldPosition`, `Cb_AttemptReconnect`            |


SMACC2의 이벤트 기반 아키텍처는 비상 상황 처리에 매우 효과적이다. 시스템의 안전을 위협하는 치명적인 이벤트(예: `Ev_CriticalBattery`, `Ev_CommsLost`, GCS의 `Ev_EmergencyStop`)는 높은 우선순위를 갖도록 설계된다. 이러한 이벤트가 발생하면, 현재 `Ms_MissionExecution` 모드 내에서 어떤 리프 상태가 실행 중이었는지와 관계없이, 상태 머신은 즉시 현재 상태를 탈출(exit)하고 `Ms_Emergency` 모드 상태로 강제 전이된다.

`Ms_Emergency` 모드 상태 내부에는 다양한 비상 시나리오에 대응하기 위한 구체적인 리프 상태들이 존재한다.

- `St_EmergencyHover`: 통신 두절과 같이 원인 파악 및 연결 재시도가 필요한 경우, 현재 위치에서 즉시 정지하고 호버링하며 상황을 대기한다.
- `St_EmergencyRTL`: 배터리 부족과 같이 즉각적인 위험은 없으나 임무 지속이 불가능할 경우, 안전하게 기지로 복귀한다.
- `St_EmergencyLand`: 모터 고장 등 즉각적인 추락 위험이 있을 경우, 현재 위치에서 비상 착륙을 시도한다.

이러한 구조는 임무 로직과 안전 로직을 명확하게 분리하며, 어떤 상황에서도 시스템의 안전을 최우선으로 보장하는 강건한 아키텍처를 제공한다.



BehaviorTree.ROS2와 SMACC2는 모두 ROS2 환경에서 복잡한 로봇 행동을 설계하기 위한 강력한 도구이지만, 근본적인 철학과 구조적 특성에서 차이를 보인다. 따라서 군집 드론 점검 임무의 다양한 요구사항 관점에서 두 프레임워크를 다각도로 비교 분석하는 것은 최적의 아키텍처를 선택하기 위한 필수적인 과정이다.

| 기준                     | BehaviorTree.ROS2                                            | SMACC2                                                       | 분석 및 근거                                                 |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **모듈성 및 재사용성**   | **우수**. BT는 본질적으로 조합적(compositional)이다. 노드와 서브트리는 독립적인 빌딩 블록으로, 쉽게 재사용하고 재배치하여 새로운 행동을 구성할 수 있다.4 | **양호**. 상태(State)와 클라이언트 행동(Client Behavior)은 재사용 가능하지만, 명시적인 상태 전이(transition)를 통해 더 강하게 결합되어 있어 유연성이 다소 떨어진다.3 | BT는 간단하고 재사용 가능한 블록들로부터 복잡하고 계층적인 행동을 조합하는 데 있어 더 뛰어난 유연성을 제공한다. |
| **반응성**               | **우수**. 트리는 높은 주기로 틱(tick)되며, 매 주기마다 조건들을 지속적으로 재평가한다. 이는 동적인 환경 변화에 즉각적으로 대응하는 데 이상적이다.4 | **양호**. 이벤트 기반 아키텍처는 본질적으로 반응적이지만, 반응의 시점은 이벤트 발생에 의존하며 지속적인 폴링 방식이 아니다.25 | BT는 더 지속적이고 '항상 켜져 있는(always-on)' 형태의 반응성을 제공하는 반면, SMACC2의 반응성은 이산적이고 이벤트에 의해 촉발된다. 동적인 임무 로직에는 BT의 모델이 더 적합한 경우가 많다. |
| **동시성 관리**          | **보통/양호**. `Parallel` 노드가 존재하지만, 진정한 의미의 직교성(orthogonality)에 대해서는 논쟁의 여지가 있다. 공유 자원을 관리하는 데 복잡성을 야기할 수 있다.30 | **우수**. `Orthogonal` 개념은 SMACC2의 핵심 기능으로, 병렬적으로 동작하는 하위 시스템을 모델링하고 관리하기 위해 특별히 설계되었다.4 | SMACC2는 복잡한 단일 로봇의 내부적인, 동시적 하드웨어 상태를 관리하는 데 있어 근본적으로 더 강건하고 확장 가능한 솔루션을 제공한다. |
| **확장성**               | **우수**. 새로운 행동을 추가하는 것은 새로운 노드나 서브트리를 추가하는 것으로, 기존 로직에 미치는 부작용(side effect)이 최소화된다.4 | **보통**. 복잡성이 증가함에 따라 상태와 전이의 수가 폭발적으로 증가하는 '상태 폭발(state explosion)' 문제에 취약할 수 있다.3 | 고수준의 임무 로직을 다룰 때, BT는 FSM보다 훨씬 우아하게 확장된다. |
| **가독성 및 유지보수성** | **양호**. Groot과 같은 시각적 도구는 트리 구조의 이해를 돕는다. 로직이 계층적이고 명시적이다.20 | **보통/양호**. 단순한 상태 머신은 매우 가독성이 높지만, 전이가 많은 복잡한 상태 머신은 '스파게티 다이어그램'처럼 얽혀 이해하기 어려워질 수 있다.30 | BT는, 특히 시각화되었을 때, 복잡한 의사결정 로직을 더 명확하게 표현하는 경향이 있다. |
| **오류 처리 설계**       | **우수**. 계층적 구조는 국소적 복구와 시스템 전역 복구를 구분하는 다층적(tiered) 복구 메커니즘을 자연스럽게 지원한다.24 | **양호**. 실패 이벤트를 수신하면 전용 비상/복구 상태로 전이할 수 있다. | BT의 구조는 정교하고 다층적인 복구 전략을 설계하기 위한 더 세분화되고 강력한 패턴을 제공한다. |

분석 결과, 두 프레임워크는 서로의 장단점을 명확하게 보완하는 관계에 있음을 알 수 있다. BehaviorTree.ROS2는 **유연하고 반응적인 임무 흐름 제어**에 탁월하다. 즉, '무엇을, 어떤 순서로, 어떤 조건에서 수행할 것인가'라는 전술적(tactical) 의사결정에 강점을 보인다. 반면, SMACC2는 **단일 로봇 내부의 복잡한 동시성 관리**에 압도적인 강점을 보인다. 즉, '다양한 하위 시스템들을 어떻게 충돌 없이 안전하고 병렬적으로 운영할 것인가'라는 플랫폼 수준(platform-level)의 상태 관리에 최적화되어 있다.


따라서 어느 한 가지 프레임워크를 선택하는 것은 다른 하나의 명백한 장점을 포기하는 결과를 낳는다. 커뮤니티에서 종종 두 패러다임을 경쟁 관계로 보는 시각이 존재하지만 4, 실제 로봇 제어 문제는 서로 다른 계층의 문제들로 구성되어 있다. BT는 '현재 세계 상태에서 다음으로 올바른 행동은 무엇인가?'라는 **임무 계획 및 실행 문제**에 대한 답을 찾는 데 뛰어나다. SMACC2는 '내 모든 하드웨어의 현재 유효한 상태는 무엇이며, 이벤트를 기반으로 어떻게 안전하게 상태를 전환할 것인가?'라는 **강건한 상태 관리 문제**를 해결하는 데 탁월하다. 이 두 가지는 동일한 문제가 아니다.

이에 본 보고서는 두 프레임워크를 상호 배타적인 선택지로 보지 않고, 각각의 강점을 극대화하는 **계층적 하이브리드 아키텍처(Hierarchical Hybrid Architecture)**를 최종적인 솔루션으로 제안한다. 이는 절충안이 아니라, 각 계층의 문제에 가장 적합한 도구를 사용하는 더 우월한 아키텍처 패턴이다. 이 구조는 로봇의 '두뇌'와 '신경계'를 분리하여 설계하는 것과 같다.

1. **드론 간 협력 (군집 레벨): BehaviorTree.ROS2**
   - **역할**: 군집 전체의 최상위 임무 제어, 즉 '두뇌' 역할을 BT가 담당한다. GCS로부터 고수준의 임무 목표를 받아, 이를 '이륙 → 이동 → 스캔 → 복귀'와 같은 구체적인 행동들의 순서로 분해하고 실행을 지휘한다.
   - **구현**: 이 마스터 BT는 군집의 리더 드론에서 실행되거나, GCS의 일부로 실행되어 각 드론에게 고수준의 명령(예: '목표 지점까지 항법 시작', '동기화 스캔 시작')을 하달한다. 동적인 환경 변화(예: 장애물 발견)나 임무 실패(예: 스캔 데이터 품질 미달)에 대한 반응적인 의사결정과 군집 수준의 복구 전략(예: 대형 재구성)을 모두 이 BT가 관리한다.
2. **드론 내부 제어 (개별 드론 레벨): SMACC2**
   - **역할**: 각 개별 드론의 내부 상태 및 하드웨어 제어, 즉 '신경계' 역할을 SMACC2 상태 머신이 담당한다.
   - **구현**: 각 드론은 `OrFlightController`, `OrGimbalCamera`, `OrCommunicationLink` 등의 직교 컴포넌트로 구성된 SMACC2 상태 머신을 실행한다. 이 상태 머신은 비행, 센서 작동, 통신, 전원 관리 등 여러 하위 시스템의 상태를 안전하고 병렬적으로 관리하며, 하드웨어 수준의 오류(예: 모터 이상, 센서 연결 끊김)를 감지하고 처리한다.
3. **인터페이스 계층 (Interface Layer)**
   - 두 계층은 **ROS2 Action 인터페이스**를 통해 명확하게 분리되고 연결된다.
   - **흐름**:
     - 상위의 BT 노드(예: `ExecuteSyncScan`)는 '동기화 스캔 시작'이라는 고수준의 목표를 담아 ROS2 Action Goal을 전송한다.
     - 군집 내 각 드론의 SMACC2 상태 머신에 포함된 `SmaccActionServerClient`가 이 Goal을 수신한다.
     - 클라이언트는 수신된 Goal을 SMACC2 내부 이벤트(예: `Ev_StartScan`)로 변환하여 상태 머신에 게시(post)한다.
     - 이 이벤트는 SMACC2 상태 머신을 적절한 상태(예: `St_PerformScan`)로 전이시켜, 해당 상태에 정의된 클라이언트 행동들이 실제 하드웨어(비행 컨트롤러, 짐벌)를 제어하도록 한다.
     - SMACC2 상태 머신은 작업의 진행 상황(예: 스캔 진행률)을 Action Feedback으로, 최종 성공 또는 실패 여부를 Action Result로 BT에 다시 보고한다. BT는 이 결과를 바탕으로 다음 행동을 결정한다.

이 하이브리드 아키텍처는 BT의 유연한 임무 설계 능력과 SMACC2의 강건한 동시성 관리 능력을 결합하여, 복잡하고 예측 불가능한 실제 환경에서 군집 드론 임무의 성공률과 안전성을 극대화할 수 있는 최적의 솔루션을 제공한다.



본 보고서는 ROS2 Humble을 기반으로 고소 시설물 점검을 위한 군집 드론 임무를 성공적으로 수행하기 위한 소프트웨어 아키텍처를 제안했다. 이를 위해 로봇 행동 및 의사결정을 위한 두 가지 핵심 프레임워크, BehaviorTree.ROS2와 SMACC2에 대한 심층적인 분석과 각각의 설계안을 제시했다.

분석 결과, **BehaviorTree.ROS2**는 모듈화된 노드와 서브트리를 조합하여 복잡한 임무의 흐름을 유연하게 설계하고, 지속적인 환경 재평가를 통해 동적인 상황에 민첩하게 대응하는 **반응형 임무 흐름 제어**에 탁월한 강점을 보였다. 특히, 계층적 구조를 활용한 다층적 오류 복구 메커니즘은 시스템의 강건성을 높이는 데 매우 효과적인 패러다임을 제공함을 확인했다.

반면, **SMACC2**는 '직교성(Orthogonality)'이라는 독창적인 개념을 통해 단일 로봇 내에 존재하는 다수의 병렬적인 하위 시스템(비행, 센서, 통신 등)을 독립적으로 모델링하고 관리하는 **내부 동시성 제어**에 압도적인 우수성을 보였다. 이는 전통적인 상태 머신의 '상태 폭발' 문제를 해결하고, 복잡한 로봇 플랫폼의 내부 상태를 안전하고 체계적으로 관리하는 강력한 기반을 제공함을 확인했다.


두 프레임워크의 강점과 약점이 명확히 상호 보완적이라는 분석에 근거하여, 어느 한 가지를 선택하기보다는 두 기술의 장점을 시너지 있게 결합하는 **계층적 하이브리드 아키텍처**를 최종 솔루션으로 제안한다.

이 아키텍처에서 군집의 전술적 임무 수행, 즉 **'무엇을(What)'과 '언제(When)'에 대한 동적 의사결정은 상위 계층의 BehaviorTree.ROS2가 담당**한다. 그리고 개별 드론의 안정적인 내부 상태 및 하드웨어 제어, 즉 **'어떻게(How)'의 안정적인 실행은 하위 계층의 SMACC2가 담당**한다. 두 계층은 ROS2 Action이라는 표준화된 인터페이스를 통해 통신하며, 고수준의 의사결정과 저수준의 강건한 실행을 명확하게 분리한다.


제안된 하이브리드 아키텍처는 단순히 두 기술의 조합을 넘어, 복잡한 자율 시스템 설계에 대한 새로운 패러다임을 제시한다. 이는 BT의 유연하고 확장 가능한 임무 설계 능력과 SMACC2의 구조적이고 강건한 동시성 관리 능력을 결합함으로써, 예측 불가능하고 위험 요소가 많은 실제 환경에서 군집 드론 임무의 성공률과 안전성을 극대화할 수 있는 최적의 방안이 될 것이다.

나아가, 이러한 '전술적 두뇌(BT)'와 '강건한 신경계(SMACC2)'를 분리하여 설계하는 접근법은 본 보고서에서 다룬 항공 점검 임무를 넘어, 자율 주행, 물류 로봇, 협동 제조 등 다양한 고신뢰성 자율 로봇 시스템의 설계 및 개발에 폭넓게 적용될 수 있는 강력하고 확장 가능한 설계 원칙으로 자리 잡을 잠재력을 가지고 있다. 이는 향후 ROS2 기반 로보틱스 개발의 표준 아키텍처 중 하나로 고려될 가치가 충분하다.


1. Swarm Robot Communications in ROS 2: An Experimental Study - ResearchGate, accessed August 18, 2025, https://www.researchgate.net/publication/384484266_Swarm_Robot_Communications_in_ROS_2_An_Experimental_Study/download
2. Robots that sync and swarm: A proof of concept in ROS 2 - Christian Bettstetter, accessed August 18, 2025, https://bettstetter.com/swarmalatorbots/
3. Behavior Trees for Robotic Applications (ROS2) in C++ | by Markus Buchholz | Medium, accessed August 18, 2025, https://markus-x-buchholz.medium.com/behavior-trees-in-c-for-robotic-applications-ros2-775ec0e97856
4. State Machines vs Behavior Trees ... - Polymath Robotics Blog, accessed August 18, 2025, https://www.polymathrobotics.com/blog/state-machines-vs-behavior-trees
5. (PDF) ROS2SWARM - A ROS 2 Package for Swarm Robot Behaviors - ResearchGate, accessed August 18, 2025, https://www.researchgate.net/publication/361962828_ROS2SWARM_-_A_ROS_2_Package_for_Swarm_Robot_Behaviors
6. Tutorials — ROS 2 Documentation: Humble documentation, accessed August 18, 2025, https://docs.ros.org/en/humble/Tutorials.html
7. ROS2swarm/ROS2swarm: A ROS 2 package providing an easy-to-extend framework for and library of swarm behaviors. - GitHub, accessed August 18, 2025, https://github.com/ROS2swarm/ROS2swarm
8. Centralized vs. decentralized control | Swarm Intelligence and Robotics Class Notes, accessed August 18, 2025, https://library.fiveable.me/swarm-intelligence-and-robotics/unit-4/centralized-vs-decentralized-control/study-guide/GsED5YFNJhZblm8D
9. Advancement Challenges in UAV Swarm Formation Control: A Comprehensive Review, accessed August 18, 2025, https://www.mdpi.com/2504-446X/8/7/320
10. Centralization vs. decentralization in multi-robot coverage: Ground robots under UAV supervision - arXiv, accessed August 18, 2025, https://arxiv.org/html/2408.06553v1
11. Centralization vs. decentralization in multi-robot sweep coverage with ground robots and UAVs - arXiv, accessed August 18, 2025, https://arxiv.org/html/2408.06553v3
12. Intelligent Swarm: Concept, Design and Validation of Self-Organized UAVs Based on Leader–Followers Paradigm for Autonomous Mission Planning - MDPI, accessed August 18, 2025, https://www.mdpi.com/2504-446X/8/10/575
13. artastier/PX4_Swarm_Controller: The aim of this ROS2 package is to facilitate the implementation of a drone swarm controller through simulation on Gazebo. - GitHub, accessed August 18, 2025, https://github.com/artastier/PX4_Swarm_Controller
14. Behavior Trees in Industrial Applications: A Case Study in Underground Explosive Charging This work has been submitted to the IEEE for possible publication. Copyright may be transferred without notice, after which this version may no longer be accessible. - arXiv, accessed August 18, 2025, https://arxiv.org/html/2403.19602v1
15. Behavior-Tree Navigator — Nav2 1.0.0 documentation, accessed August 18, 2025, https://docs.nav2.org/configuration/packages/configuring-bt-navigator.html
16. behaviortree_cpp_v3 3.8.7 documentation, accessed August 18, 2025, https://docs.ros.org/en/iron/p/behaviortree_cpp_v3/
17. behaviortree_cpp 4.7.0 documentation, accessed August 18, 2025, https://docs.ros.org/en/rolling/p/behaviortree_cpp/
18. Integration with ROS2 | BehaviorTree.CPP, accessed August 18, 2025, https://www.behaviortree.dev/docs/ros2_integration/
19. BehaviorTree.CPP utilities to work with ROS2 - GitHub, accessed August 18, 2025, https://github.com/BehaviorTree/BehaviorTree.ROS2
20. TreeExecutionServer - GitHub, accessed August 18, 2025, https://github.com/BehaviorTree/BehaviorTree.ROS2/blob/humble/behaviortree_ros2/tree_execution_server.md
21. nav2_behavior_tree — nav2_behavior_tree: Humble 1.1.18 ..., accessed August 18, 2025, https://docs.ros.org/en/humble/p/nav2_behavior_tree/
22. Nav2 - How to create a custom behavior tree (BT) plugin/node in a separate ros2 package?, accessed August 18, 2025, https://robotics.stackexchange.com/questions/107553/nav2-how-to-create-a-custom-behavior-tree-bt-plugin-node-in-a-separate-ros2
23. nav2_behavior_tree: Jazzy 1.3.7 documentation, accessed August 18, 2025, https://docs.ros.org/en/jazzy/p/nav2_behavior_tree/
24. Detailed Behavior Tree Walkthrough — Nav2 1.0.0 documentation, accessed August 18, 2025, https://docs.nav2.org/behavior_trees/overview/detailed_behavior_tree_walkthrough.html
25. robosoft-ai/SMACC2: An Event-Driven, Asynchronous, Behavioral State Machine Library for ROS2 (Robotic Operating System) applications written in C++ - GitHub, accessed August 18, 2025, https://github.com/robosoft-ai/SMACC2
26. Intro to Substate Objects | SMACC – State Machine Asynchronous C++, accessed August 18, 2025, https://smacc.dev/intro-to-substate-objects/
27. Introducing the SMACC State Machine Library - ROS General - Open Robotics Discourse, accessed August 18, 2025, https://discourse.openrobotics.org/t/introducing-the-smacc-state-machine-library/14963
28. Concepts II - Substate Architecture — SMACC2 0.1 documentation, accessed August 18, 2025, https://smacc2.robosoft.ai/latest/concepts2.html
29. Concepts I - HSM Architecture — SMACC2 0.1 documentation, accessed August 18, 2025, https://smacc2.robosoft.ai/latest/concepts1.html
30. SMACC vs Behavior Trees, accessed August 18, 2025, https://smacc.dev/smacc-vs-behavior-trees/
31. Orthogonals | SMACC – State Machine Asynchronous C++, accessed August 18, 2025, https://smacc.dev/orthogonals/
32. Introducing the SMACC State Machine Library - #9 by facontidavide - ROS General, accessed August 18, 2025, https://discourse.openrobotics.org/t/introducing-the-smacc-state-machine-library/14963/9
33. State Machine Library c++/ROS2 - ROS Answers archive, accessed August 18, 2025, https://answers.ros.org/question/416034/
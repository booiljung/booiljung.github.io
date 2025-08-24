[시뮬레이션을 위한 Unity](./index.md)
# Unity 엔진을 활용한 첨단 로봇 및 드론 시뮬레이션에 대한 종합적 분석


과거 시뮬레이션 기술의 영역은 명확히 구분되어 있었습니다. 한쪽에는 시각적 표현과 실시간 성능에 중점을 둔 게임 엔진이, 다른 한쪽에는 물리적 정확성을 최우선으로 하는 과학적 시뮬레이터가 존재했습니다. 그러나 현대에 이르러 이 두 영역 간의 경계는 점차 허물어지고 있습니다. Unity와 같은 고성능 실시간 3D 엔진이 산업 수준의 정밀한 기능을 통합하기 시작하면서, 이들은 로보틱스 개발을 위한 강력한 도구로 부상하고 있습니다.1 이러한 융합은 전통적인 시뮬레이터가 구현하기 어려웠던 사실적인 센서 시뮬레이션(특히 컴퓨터 비전 AI 학습용)과 확장 가능하며 상호작용이 가능한 환경에 대한 산업계의 요구에 의해 가속화되었습니다.3

본 보고서는 로봇 및 드론 시뮬레이션 분야에서 Unity 플랫폼의 역할과 역량에 대한 종합적인 기술 분석을 제공하는 것을 목표로 합니다. 보고서는 `ArticulationBody` 기반의 물리 엔진부터 시작하여, 로봇 운영 체제(ROS)와의 원활한 통합, 그리고 첨단 인공지능(AI) 및 디지털 트윈 구축 역량에 이르기까지 Unity의 핵심 구성 요소를 심층적으로 해부할 것입니다. 이 분석을 통해 Unity가 단순한 시각화 도구를 넘어, 프로토타이핑, 테스트, AI 훈련, 가상 시운전 등 로보틱스 개발의 전 과정을 가속화할 수 있는 강력하고 포괄적인 엔드투엔드 시뮬레이션 환경으로 진화했음을 입증하고자 합니다. 보고서는 총 4부로 구성되며, 각 부에서는 Unity 로보틱스 생태계의 구조, 물리 시뮬레이션의 핵심 기술, AI 및 센서 시뮬레이션, 그리고 실제 적용 사례 및 비교 분석을 순차적으로 다룰 것입니다.

------


이 파트에서는 Unity의 로보틱스 툴킷을 구성하는 기본 요소들을 살펴보고, 로보틱스 개발자들이 어떻게 구조화되고 효율적인 방식으로 시뮬레이션 환경에 접근할 수 있는지 설명합니다.


Unity는 단순히 게임을 만드는 엔진을 넘어, 제조, 자동차, 건축 등 다양한 산업 분야에서 활용되는 범용 실시간 3D 개발 플랫폼입니다.1 이러한 광범위한 적용 범위는 로보틱스 개발자들에게 성숙하고 안정적이며, 잘 문서화된 핵심 엔진을 제공한다는 점에서 중요한 의미를 가집니다.


Unity의 핵심 강점은 고품질 렌더링 파이프라인(HDRP, URP)을 통한 세계 최고 수준의 시각화 역량에 있습니다.6 이는 단순히 미학적인 만족을 넘어, 사실적인 조명, 그림자, 재질 특성에 의존하는 컴퓨터 비전 모델의 훈련과 검증에 필수적인 기술적 요구사항입니다.4 또한, Unity 에셋 스토어는 방대한 양의 사전 제작된 환경, 모델, 도구 라이브러리를 제공하여, 전통적인 시뮬레이터에서 처음부터 복잡한 시뮬레이션 세계를 구축하는 데 필요한 시간과 노력을 극적으로 줄여줍니다.2 이는 개발 과정에서 '결과물 도출 시간(time to delivery)'을 중시하는 문화를 조성합니다.2


Unity 에디터의 시각적이고 실시간적인 특성은 엔지니어들이 즉석에서 환경을 구축하고 수정하며, 새로운 시나리오(예: 장애물 추가, 작업 공간 레이아웃 변경)를 테스트하고 즉각적인 피드백을 받을 수 있게 합니다. 이는 반복적인 설계 과정을 가속화하는 핵심 요소입니다.2


Unity의 또 다른 핵심 전략적 이점은 시뮬레이션이나 디지털 트윈 애플리케이션을 데스크톱(Windows, Linux, macOS), 모바일, 웹(WebGL), 그리고 증강/가상현실(AR/VR) 헤드셋을 포함한 25개 이상의 플랫폼에 배포할 수 있는 능력입니다.1 이는 서버 기반의 헤드리스(headless) 테스트부터 상호작용이 가능한 HMI(Human-Machine Interface), AR 기반 유지보수 가이드에 이르기까지 매우 다양한 응용 프로그램을 가능하게 합니다.


Unity는 수백만 명의 개발자 커뮤니티를 자랑합니다.2 이들 모두가 로보틱스 전문가는 아니지만, 이 거대한 사용자 기반은 풍부한 튜토리얼, 포럼, 그리고 활용 가능한 서드파티 도구를 보장합니다. 이는 전문화된 소규모 로보틱스 시뮬레이터에 비해 훨씬 더 접근성 높고 지원이 활발한 개발 환경을 만듭니다.2

이러한 요소들이 결합하여 '생태계 선순환(Ecosystem Flywheel)' 효과를 만들어냅니다. Unity의 게임 분야에서의 성공은 고도로 최적화되고 기능이 풍부한 핵심 엔진 개발과 거대한 커뮤니티 육성에 재투자됩니다. 이 강력한 플랫폼과 커뮤니티는 산업 사용자의 진입 장벽을 낮추고, 이들 산업 사용자는 다시 로보틱스와 같은 새로운 활용 사례를 창출하며 `ArticulationBody`와 같은 새로운 기능을 요구하게 됩니다.13 Unity는 이러한 새로운 시장 수요에 대응하여 

`Unity Robotics Hub`와 같은 로보틱스 전용 기능을 개발하고, 이는 다시 플랫폼을 더욱 강력하게 만들어 더 많은 산업 사용자를 유치하는 선순환 구조를 형성합니다.14 이는 특정 목적을 위해 제작된 틈새 시뮬레이터들이 모방하기 어려운 전략적 우위입니다.


Unity Robotics Hub는 Unity에서 로보틱스 시뮬레이션을 위한 모든 필수 도구, 튜토리얼, 문서를 통합 관리하는 중앙 GitHub 저장소입니다.14 이는 개발자들에게 명확한 시작점을 제공하며, 로보틱스 생태계의 관문 역할을 합니다.


URDF(Unified Robot Description Format)는 로봇의 기하학적 구조, 시각적 메시, 운동학 및 동역학적 속성을 정의하는 표준 형식입니다. `URDF-Importer` 패키지는 이 URDF로 정의된 로봇을 Unity 씬으로 직접 가져올 수 있게 해주는 핵심 도구입니다.14 임포터는 URDF 파일의 `<link>`와 `<joint>` 태그를 분석하여, 운동학적 체인을 보존하는 `ArticulationBody` 컴포넌트를 가진 게임오브젝트(GameObject) 계층 구조를 자동으로 생성합니다.16 이 과정에서 다양한 메시 포맷(.dae,.obj,.stl)을 지원하고, 로보틱스에서 주로 사용하는 오른손 좌표계(RHS)와 Unity의 왼손 좌표계(LHS) 간의 변환을 자동으로 처리합니다.16 사용자는 보통 에셋 폴더 내의 `.urdf` 파일을 마우스 오른쪽 버튼으로 클릭하고 "Import"를 선택하는 간단한 과정만으로 로봇을 씬에 추가할 수 있습니다.15 이 패키지는 본래 Siemens의 ROS# 프로젝트에서 파생되었지만, 안정성을 위해 표준 조인트 대신 `ArticulationBody`를 사용하도록 Unity에 의해 대대적으로 개선되었습니다.15


이 시스템은 Unity와 ROS(ROS 1 및 ROS 2 모두 지원) 간의 통신을 담당합니다.14 구조적으로 Unity 내부의 `ROS-TCP-Connector` 패키지와 ROS 환경에서 실행되는 `ROS-TCP-Endpoint` 노드로 구성되며, 이 둘은 TCP 연결을 통해 메시지를 교환합니다.21

이 시스템의 가장 큰 장점은 성능입니다. 기존의 ROS#이 Rosbridge 서버와 JSON 직렬화/역직렬화에 의존하는 것과 달리, `ROS-TCP-Connector`는 ROS의 네이티브 직렬화 포맷으로 메시지를 직접 전송합니다. 이로 인해 JSON 처리 과정에서 발생하는 오버헤드가 제거되어 상당한 성능 향상을 가져옵니다.20 이러한 성능 차이는 단순한 수치적 개선을 넘어 질적인 변화를 의미합니다. 이는 시뮬레이터가 저주파 상태 업데이트만 처리할 수 있는 도구에서, ROS 생태계를 위한 가상의 고성능 센서 스위트 역할을 할 수 있는 도구로 변모시켰기 때문입니다.

Unity의 공식적인 벤치마크 결과는 이러한 성능 우위를 명확히 보여줍니다.

**표 2.1: ROS-Unity 통신 프로토콜 성능 비교**

| 테스트 시나리오                      | 측정 항목     | ROS# + Rosbridge 서버 (JSON 기반) | Unity ROS-TCP-Connector (바이너리) | 성능 향상 | 출처 |
| ------------------------------------ | ------------- | --------------------------------- | ---------------------------------- | --------- | ---- |
| 대용량 이미지 100개 전송 (1036x1698) | 이미지당 시간 | 약 10.0초                         | 약 0.6초                           | >16배     | 20   |
| 대용량 이미지 1000개 전송 (912x1698) | 총 소요 시간  | 직접 비교 불가, 느릴 것으로 추정  | 약 12.0초                          | 해당 없음 | 20   |
| 단순 서비스 호출 (요청/응답)         | 왕복 시간     | 약 2.0초                          | 약 0.17초                          | >11배     | 20   |

이러한 성능은 특히 카메라 피드나 LiDAR 스캔과 같은 고대역폭 센서 데이터를 실시간으로 스트리밍해야 하는 데이터 집약적인 작업에서 Unity를 매우 실용적인 선택지로 만듭니다. 최근 Unity는 `ROS-TCP-Connector`에서 더욱 모듈화된 `Simulation-ROS-Integrations` 패키지로의 전환을 진행하고 있습니다. 이 새로운 패키지는 연결 관리(IConnector)와 메시지 발행/구독을 분리하여 복잡한 프로젝트에서 더 유연한 아키텍처를 제공합니다.23


복잡한 로봇 시스템의 디버깅에는 데이터 스트림의 시각화가 필수적입니다. `Robotics Visualizations Package`는 Unity 씬 내에서 직접 ROS 메시지를 시각화하는, RViz와 유사한 기능을 에디터 내에서 제공합니다.7 이 패키지를 사용하면 TF(Transform) 트리, 점유 격자 지도(occupancy grid), 포인트 클라우드, 레이저 스캔, 이미지 등 일반적인 ROS 메시지 유형을 시각화할 수 있습니다. 이를 통해 개발자는 센서 데이터나 SLAM 알고리즘의 지도 생성 결과와 같은 데이터를 3D 시뮬레이션 환경에 직접 오버레이하여 문제의 원인(예: LiDAR 데이터의 노이즈 문제인지, 매핑 알고리즘의 실패인지)을 훨씬 쉽게 파악할 수 있습니다.25

------


이 파트는 Unity의 물리 엔진이 어떻게 로보틱스에 적합하게 되었는지, 단순한 기능 설명을 넘어 그 기반이 되는 알고리즘을 분석하며 기술적인 핵심을 다룹니다.



과거 Unity에서 연결된 신체(body)를 만드는 전통적인 방법은 `Rigidbody` 컴포넌트를 `HingeJoint`나 `ConfigurableJoint`와 같은 `Joint` 컴포넌트로 연결하는 것이었습니다.13 이 시스템은 각 신체의 위치와 방향을 월드 공간에서 독립적으로 추적하는 **최대 좌표계(maximal-coordinate)** 표현을 사용하며, 조인트는 반복적인 솔버(iterative solver)가 만족시키려고 노력하는 제약 조건으로 취급됩니다.13

이 접근 방식은 로봇 팔과 같이 복잡하게 연결된 체인 구조에서 불안정성을 야기했습니다. 솔버의 반복적인 특성은 특히 질량비가 크거나 제약 조건이 많은 경우 수렴에 실패하여, 조인트가 늘어나거나(stretchy) 스프링처럼 튀는(springy) 현상을 발생시켰습니다. 이러한 불안정하고 비현실적인 떨림(jitter) 현상은 정밀한 로보틱스 시뮬레이션에서는 용납될 수 없는 문제였습니다.13


Unity 2020.1에서 PhysX 4.1로의 업그레이드와 함께 출시된 `ArticulationBody`는 운동학적 체인을 시뮬레이션하기 위해 특별히 설계된 새로운 물리 컴포넌트입니다.13 이 컴포넌트는 로봇에 개별적인 

`Rigidbody`와 `Joint`를 추가할 필요를 없애고, 대신 각자 `ArticulationBody`를 가진 게임오브젝트들의 계층적 트리 구조가 로봇의 구조를 정의합니다.28 즉, 부모-자식 관계가 곧 운동학적 관계를 직접적으로 정의하게 됩니다.26


- **조인트 유형:** `Fixed`(고정), `Prismatic`(직선 운동), `Revolute`(회전 운동), `Spherical`(구형) 조인트를 지원하여 대부분의 로보틱스 요구사항을 충족합니다.28

- **드라이브(Drives):** 움직임은 강력하고 안정적인 스프링처럼 작동하는 드라이브를 통해 제어됩니다. 목표 위치나 속도를 설정하면 드라이브가 필요한 힘을 적용하여 목표에 도달합니다. `Stiffness`(강성), `Damping`(감쇠), `Force Limit`(힘 제한)과 같은 드라이브 속성을 직접 구성할 수 있습니다.28 그 기반이 되는 공식은 

  F=stiffness⋅(currentPosition−target)−damping⋅(currentVelocity−targetVelocity) 입니다.29

- **고정 가능한 루트(Immovable Root):** 관절 체인의 루트(root)는 `immovable`로 설정할 수 있어, 고정된 베이스를 가진 매니퓰레이터 시뮬레이션에 필수적입니다.28

- **지속적인 개선:** Unity는 인스펙터 창에서 속성을 재배치하여 사용성을 개선하고, 조인트 한계를 시각적으로 편집하는 도구를 추가하며, 성능을 향상시키는 등 이 컴포넌트를 지속적으로 개선하고 있습니다.5


물리 모델의 선택은 구현상의 세부 사항이 아니라, 물리 엔진이 고성능 로보틱스에 적합한지를 결정하는 근본적인 요소입니다. 최대 좌표계 시스템의 불안정성은 정밀도가 요구되는 작업이나, 솔버의 허점(예: 관통 오류로부터 에너지를 얻는 현상 32)을 학습할 수 있는 강화학습 정책 훈련에 근본적으로 신뢰할 수 없게 만듭니다. 축소 좌표계 기반의 페더스톤 솔버(Featherstone's solver)의 채택은 Unity를 '게임 물리' 엔진에서 '로보틱스 시뮬레이션' 엔진으로 격상시킨 가장 중요한 단계였습니다.


`Rigidbody`의 최대 좌표계 시스템과 달리, `ArticulationBody`는 **축소 좌표계(reduced-coordinate)** 표현을 사용합니다.26 이 모델에서 각 신체의 상태는 월드 공간에서 독립적으로 정의되지 않고, 계층 구조 내에서 부모에 대한 상대적인 자유도(degrees of freedom)로 정의됩니다. 예를 들어, 회전 조인트의 상태는 단일 각도 값 하나입니다. 이러한 표현 방식은 본질적으로 조인트 제약 조건을 강제합니다. 조인트의 위치가 부모의 위치와 조인트 각도로부터 수학적으로 파생되기 때문에, 조인트가 '늘어나거나' '깨지는' 현상이 원천적으로 불가능합니다. 반복적으로 해결해야 할 제약 조건 위반 자체가 발생하지 않는 것입니다.26


이러한 축소 좌표계 관절 체인의 동역학은 직접적이고 비반복적인 방법인 **페더스톤 알고리즘**을 사용하여 해결됩니다.29 이 알고리즘은 개방형 운동학적 체인의 순동역학(forward dynamics)을 계산하기 위한 로보틱스 분야의 고전적인 방법입니다. 조인트에 가해지는 토크/힘이 주어지면, 링크 수에 비례하는 선형 시간 복잡도 내에서 결과적인 조인트 가속도를 계산합니다. 이는 직접 솔버이므로, 특히 질량비가 크거나 복잡한 체인 구조에서 최대 좌표계 시스템의 반복 솔버가 겪는 수렴 및 안정성 문제를 근본적으로 회피합니다.13


Unity 2022.1부터는 순동역학(힘으로부터 움직임 계산)을 넘어, `ArticulationBody`를 위한 **역동역학** 지원이 추가되었습니다.36 이를 통해 사용자는 원하는 가속도를 달성하거나 중력과 같은 외부 힘을 상쇄하는 데 필요한 힘/토크를 계산할 수 있습니다. 이는 피드포워드 제어나 계산된 토크 제어(computed-torque control)와 같은 고급 제어 기법에 매우 중요하며, 실제 로봇 제어 시스템과의 격차를 더욱 줄여줍니다.

아래 표는 두 물리 모델의 핵심적인 차이점을 요약합니다.

**표 4.1: 물리 모델 비교 분석: `Rigidbody/Joint` vs. `ArticulationBody`**

| 특징               | 레거시 `Rigidbody` + `Joint` 시스템                        | `ArticulationBody` 시스템                                | 로보틱스 시뮬레이션에 미치는 영향                            | 출처 |
| ------------------ | ---------------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------ | ---- |
| **기반 물리 모델** | 최대 좌표계 (Maximal-Coordinate)                           | 축소 좌표계 (Reduced-Coordinate)                         | `ArticulationBody`는 설계상 비물리적인 조인트 늘어남 및 제약 위반을 원천적으로 제거합니다. | 13   |
| **솔버 유형**      | 반복적 제약 솔버 (예: PGS)                                 | 직접 솔버 (페더스톤 알고리즘)                            | 직접 솔버는 더 안정적이고, 수렴 문제를 피하며, 높은 질량비를 더 잘 처리하여 떨림과 불안정성을 방지합니다. | 13   |
| **안정성**         | 긴 체인에서 떨림, "스프링 같은" 조인트, 불안정성 발생 경향 | 안정성 보장; 조인트는 늘어나지 않음                      | 정밀 작업(매니퓰레이션)과 신뢰성 있는 강화학습 훈련에 필수적입니다. | 13   |
| **주요 사용 사례** | 일반 게임 물리, 간단한 메커니즘                            | 로봇 팔, 운동학적 체인, 산업용 시뮬레이션                | `ArticulationBody`는 로보틱스의 핵심적인 과제를 해결하기 위해 특별히 제작되었습니다. | 5    |
| **제어 방식**      | `Rigidbody`에 힘/토크 적용                                 | 힘 적용 또는 통합된 안정적인 `드라이브` (강성/감쇠) 사용 | 드라이브는 목표 위치/속도로 조인트를 제어하는 더 강력하고 직관적인 방법을 제공합니다. | 28   |


Unity는 `ArticulationBody` 외에도 정밀한 시뮬레이션을 지원하기 위한 다양한 고급 물리 기능을 제공합니다.

- **디버깅 및 프로파일링:** Unity는 시뮬레이션의 내부 상태를 검사할 수 있는 강력한 `Physics Debugger`를 제공합니다. 이를 통해 콜라이더, 접촉점, 심지어 레이캐스트와 같은 물리 쿼리까지 시각화할 수 있어 예기치 않은 동작을 해결하는 데 매우 중요합니다.36

  `Physics Profiler`는 활성 관절 바디 수, 동기화된 트랜스폼 수와 같은 상세한 성능 지표를 제공하여 병목 현상을 식별하고 최적화하는 데 도움을 줍니다.32

- **접촉 수정 API (Contact Modification API):** 최신 Unity 버전에서 제공되는 이 강력한 API는 접촉 물리학을 낮은 수준에서 사용자 정의할 수 있게 해줍니다. 개발자는 특정 접촉점을 프로그래밍 방식으로 무시하거나, 마찰 속성을 변경하거나, 솔버의 충격량을 수정할 수 있습니다. 이는 흡입 그리퍼, 끈적한 표면, 또는 콜라이더의 구멍과 같은 복잡한 현상을 시뮬레이션하는 데 사용될 수 있습니다.14

- **연속 충돌 감지 (Continuous Collision Detection, CCD):** 빠르게 움직이는 로봇이나 물체의 경우, 표준 이산 충돌 감지는 실패하여 "터널링" 현상을 유발할 수 있습니다. Unity의 `ArticulationBody`는 스윕 기반(sweep-based) 및 추측성(speculative) CCD 모드를 포함한 여러 CCD 모드를 지원하여 견고한 충돌 처리를 보장합니다. 이는 물리적 정확성뿐만 아니라 AI 에이전트가 물리적 결함을 악용하여 학습하는 것을 방지하는 데에도 중요합니다.14

- **물리 설정 최적화:** 로보틱스 시뮬레이션을 위해서는 특정 물리 설정을 권장합니다. 예를 들어, `Temporal Gauss Seidel` 솔버를 사용하고 솔버 반복 횟수를 늘리면, 특히 `ArticulationBody`가 일반 `Rigidbody` 객체와 상호작용할 때 수렴성과 안정성을 향상시킬 수 있습니다.23

------


이 파트에서는 Unity의 렌더링 및 물리 분야의 핵심 강점이 어떻게 AI 개발을 위한 강력한 플랫폼을 구축하는 데 활용되는지 탐구합니다.


많은 로보틱스 알고리즘(특히 인식 및 항법)의 성능은 센서 데이터의 품질에 직접적으로 좌우됩니다. 따라서 센서를 정확하게 시뮬레이션하는 것은 매우 중요합니다.


Unity의 고급 렌더링 파이프라인(HDRP/URP)은 초점 거리, 피사계 심도, 렌즈 왜곡과 같은 실제 카메라 효과를 시뮬레이션할 수 있는 물리 기반 카메라를 생성할 수 있게 합니다.2 특히 **Perception 패키지**는 특수화된 `PerceptionCamera` 컴포넌트를 제공하여, 단순한 RGB 이미지를 넘어 2D/3D 경계 상자(bounding box), 인스턴스/시맨틱 분할 마스크, 키포인트 등 다양한 종류의 정답 데이터(ground-truth)를 함께 출력할 수 있습니다.9


Unity는 성능과 정확성 사이의 균형을 맞추기 위해 여러 가지 LiDAR 시뮬레이션 접근법을 제공합니다.23

- **레이트레이싱 LiDAR (Raytraced LiDAR):** 가장 정확한 방법으로, Vulkan/DXR을 통한 하드웨어 가속 레이 트레이싱을 사용하여 실제 씬 지오메트리와 광선을 교차시킵니다. 빔 발산(beam divergence)이나 사용자 정의 스캔 패턴과 같은 고급 기능도 시뮬레이션할 수 있습니다.41
- **물리 LiDAR (Physics LiDAR):** 표준 물리 레이캐스트를 사용합니다. 하드웨어 레이 트레이싱보다 정확도는 낮지만 더 넓은 범위의 하드웨어(CPU 기반)에서 실행됩니다.
- **래스터 LiDAR (Raster LiDAR):** 뎁스 버퍼(depth buffer)를 사용하는 더 빠른 카메라 기반 접근법입니다. 복잡한 씬에 유리하지만 광선 기반 방식보다는 정확도가 떨어집니다.
- `RGLUnityPlugin` (Robotec GPU Lidar)과 같은 서드파티 에셋은 CUDA를 사용하여 더욱 전문화되고 고성능의 LiDAR 시뮬레이션을 제공하기도 합니다.43


Unity의 시뮬레이션 프레임워크는 가속도와 각속도를 보고하는 IMU, 뎁스 카메라, 어안 카메라 등 다른 일반적인 로봇 센서의 생성도 지원합니다.23 센서 출력에 노이즈 모델을 적용하여 실제 세계의 불완전성을 더 잘 모방할 수도 있습니다.45


컴퓨터 비전을 위한 딥러닝 모델을 훈련시키는 데는 방대한 양의 레이블링된 데이터가 필요하며, 이를 실제 세계에서 수집하고 주석을 다는 작업은 비용과 시간이 많이 소요됩니다.39 Unity의 시뮬레이션 능력은 대규모의 완벽하게 레이블링된 **합성 데이터셋(synthetic datasets)**을 생성함으로써 이 데이터 병목 현상에 대한 해결책을 제공합니다.1


1. **설정:** 목표 객체와 `PerceptionCamera`를 포함하는 씬을 구성합니다.
2. **레이블링:** 정의된 분류 체계에 따라 객체에 레이블을 할당합니다.40
3. **무작위화 (도메인 무작위화):** 훈련된 모델이 실제 세계에 일반화될 수 있도록, 캡처되는 프레임마다 씬을 무작위로 변경합니다. Perception 패키지에는 객체의 자세, 조명 조건, 텍스처, 카메라 위치 등을 다양하게 변경할 수 있는 "랜더마이저(Randomizers)"가 포함되어 있습니다.9
4. **캡처:** 시뮬레이션을 실행하면 `PerceptionCamera`가 RGB 이미지와 함께 픽셀 단위로 완벽하게 정렬된 경계 상자나 분할 마스크와 같은 정답 데이터를 JSON 형식으로 캡처합니다.9
5. **확장성:** 이 과정을 통해 사람이 몇 주 또는 몇 달이 걸릴 작업을 단 몇 분 만에 수만 장의 다양하고 레이블링된 이미지를 생성할 수 있습니다.9


강화학습(Reinforcement Learning, RL)은 로봇에게 복잡한 행동을 가르치는 강력한 패러다임이지만, 극도로 많은 샘플을 필요로 합니다. 물리적인 로봇에서 직접 훈련하는 것은 종종 느리고 위험하며 비현실적입니다.39


Unity의 ML-Agents 툴킷은 Unity 씬을 강화학습 에이전트를 위한 훈련 환경으로 변환하는 오픈 소스 프로젝트입니다.39

- **핵심 개념:**
  - **에이전트(Agent):** 훈련 대상인 로봇 또는 행위자입니다. 관측을 받고, 행동을 취하며, 보상을 받습니다.
  - **관측(Observations):** 에이전트가 환경을 인식하는 방식입니다. 이는 조인트 각도나 속도와 같은 상태 변수일 수도 있고, 카메라로부터의 시각적 입력일 수도 있습니다.47
  - **행동(Actions):** 로봇을 제어하는 에이전트 정책의 출력입니다 (예: 목표 조인트 회전, 모터 토크).
  - **보상 시스템(Reward System):** 강화학습의 핵심입니다. 에이전트는 바람직한 행동(예: 목표 물체에 성공적으로 닿았을 때 큰 보상)에 대해 보상을 받고, 바람직하지 않은 행동(예: 매 시간 단계마다 작은 패널티)에 대해서는 벌점을 받습니다.39
- **대규모 훈련:** ML-Agents는 여러 시뮬레이션 환경 복사본을 동시에 실행하여 실시간보다 훨씬 빠르게 경험을 수집하는 병렬 훈련을 지원합니다.39


순전히 완벽한 시뮬레이션에서만 훈련된 모델은 실제 로봇에 배포되었을 때 실패하는 경우가 많습니다. 이는 시뮬레이션 세계와 현실 세계 간의 물리적(예: 마찰, 모터 지연, 질량) 및 시각적(예: 텍스처, 조명) 불일치로 인해 발생하는 **"현실 격차(Reality Gap)"** 때문입니다.49


도메인 무작위화(DR)는 이 격차를 해소하는 핵심 기술입니다. 완벽하게 정확한 단 하나의 시뮬레이션을 만들려고 노력하는 대신, DR은 무작위화된 시뮬레이션의 광범위한 분포에 걸쳐 정책을 훈련시킵니다.50

- **시각적 무작위화:** 텍스처, 조명, 카메라 위치 등을 무작위화하면 모델이 시뮬레이션의 특정 픽셀 패턴이 아닌 객체의 본질적인 특징을 학습하도록 강제합니다.50

- **물리적 무작위화:** 질량, 마찰, 감쇠와 같은 물리적 매개변수를 무작위화하면 제어 정책이 실제 로봇 동역학의 변동과 불확실성에 대해 더 강건하고 적응력 있게 만들어집니다.50

  궁극적인 목표는 훈련된 모델에게 실제 세계가 훈련 중에 이미 본 수많은 변형 중 하나로 보이게 하여, 별도의 미세 조정 없이(zero-shot) 또는 최소한의 조정만으로(few-shot) 성공적인 전이를 가능하게 하는 것입니다.51


또 다른 보완적인 접근 방식은 시뮬레이터의 파라미터를 실제 세계와 더 가깝게 조정하는 것입니다. 이는 실제 로봇에서 데이터를 수집하고 최적화 기법을 사용하여 시뮬레이션된 궤적과 실제 궤적 간의 차이를 최소화하는 시뮬레이션 파라미터(예: 마찰, 감쇠)를 찾는 것을 포함합니다.50

이러한 AI 지원 기능들의 결합은 시뮬레이션의 역할을 근본적으로 변화시킵니다. 고성능 물리, 사실적인 센서 시뮬레이션, 대규모 병렬 처리 능력은 Unity를 단순한 테스트 장소를 넘어 AI 개발 파이프라인 자체의 필수적인 구성 요소로 만듭니다. 방대하고 다양한, 완벽하게 레이블링된 합성 데이터를 생성하는 능력9과 안전하고 병렬화된 환경에서 수백만 번의 강화학습 훈련 단계를 수행하는 능력39은 물리적 세계에서는 사실상 불가능한 작업입니다. 이는 로보틱스 R&D의 경제성과 속도를 근본적으로 바꾸고 있으며, 시뮬레이터는 더 이상 설계 과정의 마지막에 사용되는 검증 도구가 아니라, AI 모델 생성 과정 자체의 핵심적인 상류(upstream) 부분이 되었습니다.

------


이 파트에서는 핵심 기술에서 벗어나 상위 수준의 응용 프로그램 및 전략적 위치를 다루며, Unity가 실제 현장에서 어떻게 사용되는지와 대안들과 어떻게 비교되는지에 대한 맥락을 제공합니다.


디지털 트윈은 실제 자산, 프로세스 또는 시스템의 가상 모델로, 데이터 스트림을 통해 실제 세계의 대응물과 연결됩니다. 이는 모니터링, 분석, 예측 및 제어에 사용될 수 있는 살아있는 시뮬레이션입니다.11


1. **가상 복제본(Virtual Replica):** 종종 CAD/BIM 데이터로부터 생성된 Unity 내의 고품질 3D 모델입니다.1 이는 시각적 외형뿐만 아니라 운동학을 위한 

   `ArticulationBody` 및 기타 물리 컴포넌트를 사용하여 시뮬레이션된 행동 및 물리적 속성까지 포함합니다.55

2. **데이터 수집(Data Ingestion):** 트윈은 물리적 세계와 연결됩니다. 이는 IoT 데이터 스트림(MQTT, OPC UA와 같은 프로토콜 사용), 기계용 PLC 컨트롤러, 또는 로봇 제어 박스와의 직접적인 연결을 포함할 수 있습니다.11

3. **연결(The Link):** 로보틱스 맥락에서 이 연결은 종종 ROS를 통해 관리됩니다. Unity 디지털 트윈은 실제 로봇의 상태(예: 조인트 각도)를 발행하는 ROS 토픽을 구독하여 그 움직임을 미러링합니다.

4. **제어 및 상호작용(Control and Interaction):** 이 연결은 양방향이 될 수 있습니다. 가상 환경에서 내린 명령(예: 3D HMI를 통해)은 물리적 로봇으로 다시 전송되어 원격 조작 및 가상 시운전을 가능하게 합니다.11


- **Kauda Studio:** Kauda 로봇 팔의 디지털 트윈 역할을 하는 Unity 애플리케이션이 개발되었습니다.
- **주요 기능:** 역기구학(IK) 제어, 실제 로봇과의 USB/블루투스 연결, 그리고 OAK-D 카메라의 가상 트윈을 포함한 정확한 시뮬레이션을 제공했습니다.
- **응용 분야:**
  - **합성 데이터 생성:** 가상 OAK-D 카메라를 Perception 패키지와 함께 사용하여 훈련 데이터를 생성했습니다.
  - **강화학습:** 물리적 로봇으로는 너무 시간이 많이 걸렸을 "터치" 작업을 위한 제어 정책을 훈련시키기 위해 ML-Agents와 함께 디지털 트윈을 사용했습니다.
  - **AR 유지보수:** 실제 하드웨어 없이 유지보수 훈련을 위해 가상 로봇을 현실 세계에 오버레이하는 데 AR 기능을 사용했습니다.
  - **안전 테스트:** 인간-로봇 협업 시나리오를 안전하게 시뮬레이션하고 테스트하는 데 트윈을 사용했습니다.

이 외에도 Unity의 디지털 트윈은 설계 검증, 공장 라인의 가상 시운전(물리적 설치 전 PLC 로직 테스트), 3D HMI 생성, 품질 검사를 위한 비전 AI 훈련 등 다양한 산업 분야에서 활용됩니다.1


어떤 시뮬레이터가 '최고'인지는 사용 사례에 따라 달라집니다. 단일 '최고'의 시뮬레이터는 존재하지 않으며, 데이터는 명확한 트레이드오프를 시사합니다. Gazebo는 ROS 네이티브, 물리 중심 연구에 최적화되어 있습니다. 반면 Unity는 시각적으로 풍부하고 대규모이며 AI 중심의 시뮬레이션에 최적화되어 있습니다. Unity의 강점(렌더링, 확장성)이 Gazebo의 약점(시각적 품질, 대규모 월드에서의 성능)을 직접적으로 해결한다는 점은 중요한 시사점입니다.4 이는 Unity가 Gazebo를 대체하는 것이 아니라, 인식 및 AI를 중심으로 하는 특정하고 성장하는 현대 로보틱스 문제 클래스에 대해 우월한 선택지임을 의미합니다.


Gazebo는 ROS 커뮤니티의 사실상 표준 시뮬레이터입니다. 오픈 소스이며 ROS와 긴밀하게 통합되어 있고, 기존 로봇 모델과 센서 플러그인의 방대한 라이브러리를 갖추고 있습니다.3

**표 11.1: 기능 및 성능 벤치마크: Unity vs. Gazebo**

| 성능 지표                   | Unity                                             | Gazebo                                                 | 평가자를 위한 결론                                           | 출처 |
| --------------------------- | ------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| **시각적 품질 및 현실성**   | 매우 높음 (고급 렌더링, 물리 기반 재질, 조명)     | 낮음-중간 (기본 렌더링, 덜 사실적)                     | Unity는 비전 기반 AI 훈련 및 사실적인 시각화가 필요한 응용 분야에 우월합니다. | 3    |
| **환경 구축**               | 쉽고 빠름 (직관적인 에디터, 방대한 에셋 스토어)   | 더 복잡함 (주로 SDF/URDF 기반, 제한된 에셋 라이브러리) | Unity는 크고 복잡하며 다양한 세계를 훨씬 빠르게 생성할 수 있습니다. | 2    |
| **확장성 (대규모 환경)**    | 높음 (대규모 게임 월드에 최적화)                  | 낮음 (크고 복잡한 씬에서 성능 저하)                    | Unity는 창고, 공장 또는 야외 지역과 같은 대규모 환경 시뮬레이션에 더 나은 선택입니다. | 4    |
| **실시간 LiDAR 시뮬레이션** | 높은 성능 (하드웨어 레이트레이싱, 다중 모델)      | 더 제한적                                              | Unity는 자율 주행 작업에 중요한 더 정확하고 성능이 뛰어난 LiDAR 시뮬레이션을 제공합니다. | 4    |
| **시각 기반 SLAM 성능**     | Gazebo와 동일하거나 더 우수함                     | 기준선                                                 | Unity의 시각적 출력은 최첨단 인식 알고리즘에 충분히 견고합니다. | 4    |
| **ROS 통합**                | 패키지를 통한 고성능 연결, 설정 필요              | 네이티브, 간소화된 통합. 더 큰 플러그인 생태계         | Gazebo는 즉시 사용 가능한 ROS 통합이 더 쉽고, Unity는 구성 후 더 높은 성능을 제공합니다. | 4    |
| **핵심 강점**               | AI/ML 데이터 생성 및 시각적으로 풍부한 시뮬레이션 | ROS 중심 알고리즘 개발 및 물리 연구                    | 사용 사례에 따라 선택이 달라집니다: AI 및 인식 vs. 전통적인 제어 및 ROS 생태계 깊이. | 3    |


- **NVIDIA Isaac Sim:** Omniverse 기반으로 구축되었으며, 사실적인 레이트레이싱 렌더링과 NVIDIA의 AI 스택(Isaac Gym 등)과의 긴밀한 통합이 핵심 장점입니다. 특히 비전 기반 AI 훈련 분야에서 강력한 경쟁자입니다.57
- **AirSim:** Unreal Engine(과거에는 Unity도 지원)용 오픈 소스 플러그인으로, 주로 드론 및 차량 시뮬레이션에 중점을 둡니다. 고품질 차량 동역학 및 센서 모델로 유명합니다.57

프로젝트의 주요 목표에 따라 시뮬레이터 선택이 달라집니다. ROS 생태계에 깊이 통합된 전통적인 제어 알고리즘 연구에는 Gazebo가 여전히 강력하고 가벼운 선택입니다. 반면, 고품질 시각적 렌더링, 대규모 및 복잡한 환경, 그리고 비전 기반 AI 훈련에 중점을 둔 응용 프로그램에는 Unity가 더 강력하고 유연한 대안을 제시합니다.3

------


본 보고서의 분석 결과를 종합하면, Unity는 고품질 렌더링 엔진, `ArticulationBody`에 기반한 견고하고 안정적인 물리 코어, 그리고 포괄적인 AI/ML 툴킷을 바탕으로 로보틱스 시뮬레이션을 위한 최상위 플랫폼으로 성공적으로 전환했습니다. 특히 시각적으로 풍부하고 데이터 집약적인 응용 분야에서 탁월한 성능을 보입니다.

Unity는 이 분야에 대한 투자를 지속하고 있습니다. 로드맵에는 힘/토크 센서 및 쿼리 프리미티브와 같은 새로운 기능이 포함되어 있어 물리 시뮬레이션 역량을 더욱 강화할 것입니다.14 또한, ROS 통합 패키지의 진화는 더 견고하고 모듈화된 아키텍처에 대한 약속을 보여줍니다.24

로보틱스 시뮬레이션의 미래는 비용과 시간이 많이 소요되는 물리적 하드웨어에 대한 의존도를 줄이기 위해 점점 더 현실적이고 복잡한 시뮬레이션을 만드는 데 있습니다. 물리적 정확성과 사실적인 그래픽, 확장성을 결합한 Unity와 같은 플랫폼은 이러한 추세의 선두에 있습니다. 이들은 단순히 로봇을 테스트하는 도구가 아니라, 로봇을 구동하는 AI를 개발하기 위한 필수 인프라가 되고 있습니다. 궁극적인 목표는 최소한의 미세 조정만으로 정책을 실제 세계에 배포할 수 있을 정도로 '심투리얼(sim-to-real)' 격차를 해소하는 것이며, 도메인 무작위화와 고성능 센싱에 대한 Unity의 집중은 바로 이 목표를 향하고 있습니다.




1. Augmented & Virtual Reality Solutions for Manufacturing | Unity, accessed July 11, 2025, https://unity.com/solutions/manufacturing
2. Robots are hard, game engines are not: why we built our own simulator using Unity, accessed July 11, 2025, https://formant.io/blog/robots-are-hard-game-engines-are-not-why-we-built-our-own-simulator-using-unity/
3. Autonomous Mobile Robots (AMRs) : Which robot simulation software to use?, accessed July 11, 2025, https://www.blackcoffeerobotics.com/blog/autonomous-mobile-robots-which-robot-simulation-software-to-use
4. Comparative Analysis of ROS-Unity3D and ROS-Gazebo for Mobile ..., accessed July 11, 2025, https://www.researchgate.net/publication/366423166_Comparative_Analysis_of_ROS-Unity3D_and_ROS-Gazebo_for_Mobile_Ground_Robot_Simulation
5. Unity 2021.2 베타 물리 업데이트: 더욱 사실적인 로봇 시뮬레이션 제작, accessed July 11, 2025, https://unity.com/kr/blog/engine-platform/simulate-robots-with-more-realism-whats-new-in-physics-for-unity-20212-beta
6. Unity를 이용한 로봇 비전 학습, accessed July 11, 2025, https://unity.com/kr/blog/industry/teaching-robots-to-see-with-unity
7. Unity Robotics Visualizations 패키지 소개, accessed July 11, 2025, https://unity.com/kr/blog/engine-platform/introducing-unity-robotics-visualizations-package
8. AI2-THOR: 홈 어시스턴트 로봇을 훈련하기 위한 가상 환경 - Unity, accessed July 11, 2025, https://unity.com/kr/resources/ai2-thor-session
9. Teaching robots to see with Unity, accessed July 11, 2025, https://unity.com/blog/industry/teaching-robots-to-see-with-unity
10. Robotics simulation in Unity is as easy as 1, 2, 3, accessed July 11, 2025, https://unity.com/blog/engine-platform/robotics-simulation-is-easy-as-1-2-3
11. Digital Twins in the Machinery & Robotics Industry - Unity, accessed July 11, 2025, https://unity.com/blog/digital-twins-machinery-robotics-industry
12. Unity Learn: Learn game development w/ Unity | Courses & tutorials in game design, VR, AR, & Real-time 3D, accessed July 11, 2025, https://learn.unity.com/
13. Use articulation bodies to easily prototype industrial designs with realistic motion and behavior - Unity, accessed July 11, 2025, https://unity.com/blog/industry/use-articulation-bodies-to-easily-prototype-industrial-designs-with-realistic-motion-and
14. Unity-Technologies/Unity-Robotics-Hub: Central repository ... - GitHub, accessed July 11, 2025, https://github.com/Unity-Technologies/Unity-Robotics-Hub
15. Unity-Technologies/URDF-Importer - GitHub, accessed July 11, 2025, https://github.com/Unity-Technologies/URDF-Importer
16. Unity-Robotics-Hub/tutorials/urdf_importer/urdf_appendix.md at main - GitHub, accessed July 11, 2025, https://github.com/Unity-Technologies/Unity-Robotics-Hub/blob/main/tutorials/urdf_importer/urdf_appendix.md
17. URDF Importer - Game Asset Deals, accessed July 11, 2025, https://www.gameassetdeals.com/asset/99316/urdf-importer
18. Import a vehicle model | Unity Simulation, accessed July 11, 2025, https://docs.unity3d.com/Simulation/manual/author/create-a-vehicle-model/import-a-vehicle-model.html
19. Pick-and-Place Workflow in Unity 3D Using ROS - MATLAB & - MathWorks, accessed July 11, 2025, https://www.mathworks.com/help/robotics/ug/pick-and-place-workflow-in-unity-3d-using-ros.html
20. Unity-Robotics-Hub/faq.md at main / Unity-Technologies/Unity ..., accessed July 11, 2025, https://github.com/Unity-Technologies/Unity-Robotics-Hub/blob/main/faq.md
21. Unity-Technologies/ROS-TCP-Endpoint - GitHub, accessed July 11, 2025, https://github.com/Unity-Technologies/ROS-TCP-Endpoint
22. Communication between Unity and ROS using the ROS TCP Connector. Illustration based on [25] - ResearchGate, accessed July 11, 2025, https://www.researchgate.net/figure/Communication-between-Unity-and-ROS-using-the-ROS-TCP-Connector-Illustration-based-on_fig2_386190382
23. Install the authoring tools | Unity Simulation, accessed July 11, 2025, https://docs.unity3d.com/Simulation/manual/author/install-the-authoring-tools.html
24. Migrate from ROS-TCP-Connector to Simulation-ROS-Integrations - Unity - Manual, accessed July 11, 2025, https://docs.unity3d.com/Simulation/manual/author/install-the-authoring-tools/migrate-from-ros-tcp-connector-to-simulation-ros-integrations.html
25. Introducing: Unity Robotics Visualizations Package, accessed July 11, 2025, https://unity.com/blog/engine-platform/introducing-unity-robotics-visualizations-package
26. Introduction to physics articulations - Unity - Manual, accessed July 11, 2025, https://docs.unity3d.com/6000.1/Documentation/Manual/physics-articulations.html
27. Unity Dev Weeks 2023: Unity를 활용한 로봇 시뮬레이션 - YouTube, accessed July 11, 2025, https://www.youtube.com/watch?v=ARoe-YTN6mg
28. articulations-robot-demo/docs/Building-With-Articulations.md at master - GitHub, accessed July 11, 2025, https://github.com/Unity-Technologies/articulations-robot-demo/blob/master/docs/Building-With-Articulations.md
29. Robotics simulation in Unity. Coding physics articulations | by SpicyTech - Medium, accessed July 11, 2025, https://jmake.medium.com/robotics-simulation-in-unity-1986d828abc5
30. Articulation Body - Omniverse Connect - Unity, accessed July 11, 2025, https://docs.omniverse.nvidia.com/connect/latest/unity/manual/physics_articulation.html
31. Articulation Body component reference - Unity - Manual, accessed July 11, 2025, https://docs.unity3d.com/6000.1/Documentation/Manual/class-ArticulationBody.html
32. Simulate robots with more realism: What's new in physics for Unity 2021.2 beta, accessed July 11, 2025, https://unity.com/blog/engine-platform/simulate-robots-with-more-realism-whats-new-in-physics-for-unity-20212-beta
33. Scripting API: ArticulationBody - Unity - Manual, accessed July 11, 2025, https://docs.unity3d.com/6000.1/Documentation/ScriptReference/ArticulationBody.html
34. Featherstone's algorithm - Wikipedia, accessed July 11, 2025, [https://en.wikipedia.org/wiki/Featherstone%27s_algorithm](https://en.wikipedia.org/wiki/Featherstone's_algorithm)
35. Contrived case: large amount of reduced coordinate articulation suffers performance / Issue #71 / NVIDIAGameWorks/PhysX - GitHub, accessed July 11, 2025, https://github.com/NVIDIAGameWorks/PhysX/issues/71
36. 로보틱스 툴박스 확장: Unity 2022.1 물리 개선, accessed July 11, 2025, https://unity.com/kr/blog/engine-platform/expanding-the-robotics-toolbox-physics-changes-in-unity-20221
37. Unity: Articulation Body Overview - Anja's Website, accessed July 11, 2025, https://anja-haumann.de/unity-articulation-body-overview/
38. [Unity Physics] Real-Time 시뮬레이션 유니티의 물리엔진 #2 (Unity 2022.1Alpha) Robotics 튜토리얼 - pnltoen - 티스토리, accessed July 11, 2025, [https://pnltoen.tistory.com/entry/Unity-Physics-Real-Time-%EC%8B%9C%EB%AE%AC%EB%A0%88%EC%9D%B4%EC%85%98-%EC%9C%A0%EB%8B%88%ED%8B%B0%EC%9D%98-%EB%AC%BC%EB%A6%AC%EC%97%94%EC%A7%84-2-Unity-20221Alpha-Robotics-%ED%8A%9C%ED%86%A0%EB%A6%AC%EC%96%BC](https://pnltoen.tistory.com/entry/Unity-Physics-Real-Time-시뮬레이션-유니티의-물리엔진-2-Unity-20221Alpha-Robotics-튜토리얼)
39. Made with Unity: Creating and training a robot digital twin, accessed July 11, 2025, https://unity.com/blog/industry/creating-and-training-a-robot-digital-twin
40. Perception Package | 1.0.0-preview.1 - Unity - Manual, accessed July 11, 2025, https://docs.unity3d.com/Packages/com.unity.perception@1.0/
41. What is a LiDAR? | Unity Simulation, accessed July 11, 2025, https://docs.unity3d.com/Simulation/manual/author/set-up-sensors/configure-a-lidar.html
42. Emulate sensors and mechatronics systems with Unity SystemGraph, accessed July 11, 2025, https://unity.com/blog/industry/emulate-sensors-and-mechatronics-systems-with-unity-systemgraph
43. RGLUnityPlugin - AWSIM Labs Documentation, accessed July 11, 2025, https://autowarefoundation.github.io/AWSIM-Labs/pr-56/Components/Sensors/LiDARSensor/RGLUnityPlugin/
44. Field-Robotics-Japan/UnitySensors: ROS/ROS2 enabled Sensor models (Assets) on Unity, accessed July 11, 2025, https://github.com/Field-Robotics-Japan/UnitySensors
45. Sensor Setup | Unity Simulation - Unity - Manual, accessed July 11, 2025, https://docs.unity3d.com/Simulation/manual/author/set-up-sensors.html
46. Made with Unity: 로봇 디지털 트윈 제작 및 훈련, accessed July 11, 2025, https://unity.com/kr/blog/industry/creating-and-training-a-robot-digital-twin
47. Project - Reinforcement Learning with Unity 3D: G.E.A.R, accessed July 11, 2025, https://dtransposed.github.io/blog/2019/03/10/GEAR/
48. Camera Vision | Unity ML-Agents - YouTube, accessed July 11, 2025, https://www.youtube.com/watch?v=7FHyqzUBzZ0
49. Sim-to-Real Transfer in Deep Reinforcement Learning for Robotics: a Survey - arXiv, accessed July 11, 2025, https://arxiv.org/pdf/2009.13303
50. Domain Randomization for Sim2Real Transfer | Lil'Log, accessed July 11, 2025, https://lilianweng.github.io/posts/2019-05-05-domain-randomization/
51. Domain Randomization for Transferring Deep Neural Networks from Simulation to the Real World | Request PDF - ResearchGate, accessed July 11, 2025, https://www.researchgate.net/publication/315489711_Domain_Randomization_for_Transferring_Deep_Neural_Networks_from_Simulation_to_the_Real_World
52. Few-shot Sim2Real Based on High Fidelity Rendering with Force Feedback Teleoperation, accessed July 11, 2025, https://arxiv.org/html/2503.01301v1
53. What Went Wrong? Closing the Sim-to-Real Gap via Differentiable Causal Discovery, accessed July 11, 2025, https://proceedings.mlr.press/v229/huang23c/huang23c.pdf
54. Introduction to Digital Twins with Unity - Unity Learn, accessed July 11, 2025, https://learn.unity.com/tutorial/introduction-to-digital-twins-with-unity
55. Build Industrial Digital Twins| Free Guide for Robotics & Automation - Unity, accessed July 11, 2025, https://unity.com/resources/industrial-digital-twin-ebook
56. Top 10 Applications & Use Cases for Digital Twins - Unity, accessed July 11, 2025, https://unity.com/topics/digital-twin-applications-and-use-cases
57. Choose a Simulator - Robotics Knowledgebase, accessed July 11, 2025, https://roboticsknowledgebase.com/wiki/robotics-project-guide/choose-a-sim/
58. Best Robot Simulators - Formant, accessed July 11, 2025, https://formant.io/blog/best-robot-simulators/
59. knmcguire/best-of-robot-simulators - GitHub, accessed July 11, 2025, https://github.com/knmcguire/best-of-robot-simulators


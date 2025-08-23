# NVIDIA MobilityGen 로보틱스를 위한 합성 데이터 생성부터 생성형 AI 시뮬레이션



물리 기반 인공지능(Physical AI)의 시대가 도래하며, 디지털 공간에 머물던 AI가 현실 세계와 직접 상호작용하는 자율 로봇으로 구현되고 있다. 이러한 로봇이 복잡하고 예측 불가능한 물리적 환경에서 안전하고 효율적으로 임무를 수행하기 위해서는 방대한 양의 고품질 데이터가 필수적이다. 그러나 실제 환경에서 데이터를 수집하는 전통적인 방식은 여러 근본적인 한계에 직면한다. 첫째, 비용과 시간의 문제다. 실제 로봇을 운영하고, 센서 데이터를 수집하며, 수집된 데이터에 수동으로 주석을 다는(labeling) 과정은 막대한 자원을 소모한다.1 둘째, 안전성 문제다. 특히 위험한 산업 환경이나 인간과의 상호작용이 필요한 시나리오에서 로봇의 초기 학습 단계는 잠재적인 사고 위험을 내포한다. 셋째, 희귀 시나리오(edge case) 재현의 어려움이다. 자율 시스템의 강인성은 예측하기 어려운 드문 상황에 얼마나 잘 대처하는지에 따라 결정되지만, 이러한 희귀 사례는 현실 세계에서 의도적으로 재현하기가 거의 불가능하다.


이러한 현실 세계 데이터 수집의 한계를 극복하기 위한 강력한 대안으로 시뮬레이션을 통해 생성된 합성 데이터(Synthetic Data)가 부상하고 있다. 합성 데이터는 가상의 3D 환경에서 생성되므로, 현실에서는 얻기 힘든 여러 장점을 제공한다. 가장 큰 장점은 완벽한 그라운드 트루스(ground truth)를 자동으로 생성할 수 있다는 점이다. 예를 들어, 시뮬레이션 환경에서는 이미지의 모든 픽셀이 어떤 객체에 속하는지(semantic segmentation), 각 객체의 3D 경계 상자(bounding box)가 어디에 있는지, 카메라로부터의 정확한 거리가 얼마인지(depth) 등의 정보를 오차 없이 얻을 수 있다.2 이는 데이터 레이블링에 소요되는 막대한 비용을 절감시킨다. 또한, 시뮬레이션은 위험하거나 재현하기 어려운 시나리오를 안전하고 반복적으로 테스트할 수 있는 환경을 제공하며, 조명, 날씨, 재질 등 환경 변수를 무작위로 변경(domain randomization)하여 데이터셋의 다양성을 극대화하고 AI 모델의 일반화 성능을 높일 수 있다.3


이러한 배경 속에서 NVIDIA MobilityGen은 로보틱스 개발의 핵심적인 난제를 해결하기 위해 등장했다. MobilityGen은 단순한 소프트웨어 애플리케이션이 아니라, NVIDIA의 로보틱스 시뮬레이션 플랫폼인 Isaac Sim 위에서 동작하는 핵심 툴셋(toolset)이다.4 그 핵심 목표는 자율 이동 로봇(Autonomous Mobile Robots, AMRs), 휴머노이드, 사족보행 로봇 등 다양한 형태의 이동 로봇을 위한 고품질 모션 및 센서 데이터를 손쉽게 생성하고 수집하는 것이다.5 MobilityGen은 물리적으로 정확한 시뮬레이션 환경 내에서 로봇의 궤적을 기록하고, 이를 기반으로 사실적인 센서 데이터를 오프라인으로 렌더링하는 워크플로우를 제공함으로써, AI 모델 훈련에 필요한 대규모 데이터셋 구축 과정을 가속화하고 민주화한다.


본 보고서는 NVIDIA MobilityGen을 기술적, 전략적 관점에서 다각적으로 심층 분석하는 것을 목표로 한다. 이를 위해 먼저 MobilityGen을 구동하는 기술적 기반인 NVIDIA Omniverse와 Isaac Sim 플랫폼의 핵심 요소들을 고찰한다(제1장). 다음으로 MobilityGen의 구체적인 소프트웨어 아키텍처와 데이터 생성 워크플로우를 단계별로 상세히 설명하며 그 확장성을 분석한다(제2장). 제3장에서는 MobilityGen을 통해 생성되는 합성 데이터의 종류와 그 특성을 체계적으로 분류하고, 각 데이터가 로보틱스 AI 모델 훈련에 어떻게 활용될 수 있는지 탐구한다. 제4장에서는 현대 생성형 AI 기술의 정수인 확산 모델(Diffusion Model)의 수학적 원리를 깊이 있게 탐구하며, 이것이 물리 시뮬레이션과 어떻게 연결될 수 있는지 논한다. 이를 바탕으로 제5장에서는 MobilityGen 데이터가 실제 자율 이동 로봇 모델, 특히 모방 학습과 강화 학습 모델 훈련에 어떻게 활용되는지를 NVIDIA의 최신 연구 사례와 함께 살펴본다. 제6장에서는 MobilityGen의 기술이 개별 로봇을 넘어 자율주행차 시뮬레이션 플랫폼인 NVIDIA DRIVE Sim으로 어떻게 확장되고, 생성형 AI 기반 교통 시뮬레이션의 미래를 어떻게 열어가고 있는지 고찰한다. 마지막으로 제7장에서는 Sim-to-Real 격차와 같은 기술적 한계와 도전 과제를 분석하고, 절차적 콘텐츠 생성(PCG) 및 하이브리드 시뮬레이션과 같은 미래 전망을 제시하며 결론을 맺는다.

이 분석 과정에서 MobilityGen이 단순한 데이터 생성 도구를 넘어, NVIDIA가 Isaac Sim과 Omniverse 플랫폼을 중심으로 로보틱스 개발 생태계를 구축하기 위한 전략적 도구(strategic enabler)로서 기능한다는 점이 드러날 것이다. NVIDIA는 MobilityGen을 오픈소스로 제공하고 4, 사용자가 직접 로봇과 시나리오를 확장할 수 있도록 설계했으며 4, 무료 교육 과정까지 제공하고 있다.7 이는 개발자들의 진입 장벽을 낮추고 커뮤니티를 활성화하여 Isaac Sim을 로보틱스 연구 개발의 표준 플랫폼으로 자리매김하게 하려는 의도다. 결국 개발자들이 이 생태계 안에서 활동하며 자연스럽게 NVIDIA의 핵심 자산인 GPU 하드웨어와 클라우드 서비스를 사용하도록 유도하는, 더 큰 그림의 일부인 것이다.8


NVIDIA MobilityGen의 성능과 잠재력을 온전히 이해하기 위해서는 그 근간을 이루는 기술 스택에 대한 깊이 있는 고찰이 선행되어야 한다. MobilityGen은 독립적인 애플리케이션이 아니라, NVIDIA Omniverse라는 거대한 플랫폼 위에서 동작하는 Isaac Sim의 확장 기능이다. 따라서 Omniverse의 협업 철학, OpenUSD의 데이터 구조, PhysX의 물리적 정확성, 그리고 RTX의 시각적 사실성이 결합하여 MobilityGen이 고품질 합성 데이터를 생성할 수 있는 토대를 마련한다.


NVIDIA Omniverse는 3D 워크플로우의 구축과 운영을 위한 개방형 확장 플랫폼으로, 실시간 물리 기반 시뮬레이션과 렌더링을 지원한다.8 이는 단순히 3D 콘텐츠를 제작하는 도구가 아니라, 다양한 산업 분야의 소프트웨어 툴(예: Autodesk Maya, Blender, Unreal Engine 등)과 데이터 포맷을 연결하는 허브 역할을 수행한다. Omniverse의 핵심은 '실시간 협업'과 '단일 진실 공급원(Single Source of Truth)' 개념이다. 여러 명의 개발자와 아티스트가 각자 다른 툴을 사용하더라도, Omniverse Nucleus라는 데이터베이스 서버를 통해 동일한 3D 씬(scene)을 실시간으로 공유하고 수정할 수 있다. 이러한 협업 환경은 복잡한 로봇 시뮬레이션 환경을 구축하고 테스트하는 과정을 극적으로 효율화한다.


Omniverse 플랫폼의 데이터 교환과 협업을 가능하게 하는 핵심 기술은 Pixar Animation Studios에서 개발하고 오픈소스로 공개한 Universal Scene Description (OpenUSD)이다.2 OpenUSD는 복잡한 3D 씬과 에셋을 기술하기 위한 강력하고 확장 가능한 프레임워크다. 가장 중요한 특징은 '비파괴적(non-destructive)' 편집 워크플로우를 지원한다는 점이다. 이는 원본 데이터를 수정하지 않고, 레이어(layer)를 겹겹이 쌓아 올리는 방식으로 씬을 구성하고 수정할 수 있게 해준다. 예를 들어, 기본 환경 레이어 위에 로봇 모델 레이어를 얹고, 그 위에 조명 레이어, 물리 속성 레이어를 추가하는 방식이다. Isaac Sim과 MobilityGen에서 생성되는 모든 환경, 로봇, 센서 데이터는 이 OpenUSD 포맷을 기반으로 기술된다.8 이는 시뮬레이션 에셋의 재사용성, 상호 운용성, 확장성을 극대화하며, MobilityGen이 다양한 로봇과 환경을 유연하게 조합하여 시나리오를 생성할 수 있는 기반이 된다.


로봇 시뮬레이션의 신뢰성은 물리 엔진이 얼마나 현실 세계의 물리 법칙을 정확하게 모사하는지에 달려있다. 부정확한 물리 시뮬레이션에서 생성된 데이터로 학습한 AI 모델은 실제 로봇에서 예기치 않은 오작동을 일으킬 수 있다. NVIDIA Isaac Sim은 최신 버전의 물리 엔진인 PhysX 5를 탑재하고 있으며, 이는 NVIDIA GPU의 병렬 처리 능력을 활용하여 고도로 복잡한 물리 현상을 실시간으로 정확하게 계산한다.2 PhysX 5는 강체 동역학(rigid body dynamics), 연체 동역학(soft body dynamics), 유체 시뮬레이션뿐만 아니라, 로봇 공학에 필수적인 관절 마찰(joint friction), 액추에이터 모델, 충돌 감지(collision detection) 등의 기능을 정교하게 지원한다.8 MobilityGen이 생성하는 로봇의 움직임, 즉 궤적 데이터의 물리적 타당성은 바로 이 PhysX 엔진에 의해 보장된다. 이는 생성된 데이터가 단순한 애니메이션이 아니라, 실제 물리 법칙을 따르는 유의미한 학습 자료가 되도록 만드는 핵심 요소다.


자율 로봇의 의사결정은 대부분 카메라, LiDAR, 레이더와 같은 센서로부터 입력된 데이터를 기반으로 이루어진다. 따라서 시뮬레이션에서 생성되는 센서 데이터가 현실과 얼마나 유사한지는 Sim-to-Real 성능에 결정적인 영향을 미친다. NVIDIA RTX 기술은 하드웨어 가속 레이 트레이싱(Ray Tracing)을 통해 빛과 물질의 상호작용을 물리적으로 시뮬레이션한다. 빛의 직진, 반사, 굴절, 그림자 형성 등을 실시간으로 계산하여 사진처럼 사실적인(photorealistic) 이미지를 생성할 수 있다.8 MobilityGen의 오프라인 렌더링 과정에서 이 RTX 기술이 활용되어, 매우 사실적인 RGB 이미지는 물론, 각 픽셀까지의 거리를 정밀하게 측정한 깊이(Depth) 이미지, 레이저 빔의 반사를 시뮬레이션한 LiDAR 포인트 클라우드 데이터를 생성한다. 이는 시각 기반 AI 모델이 시뮬레이션 환경과 실제 환경을 구분하기 어렵게 만들어 Sim-to-Real 격차를 줄이는 데 결정적인 역할을 한다.


Isaac Sim은 앞서 설명한 Omniverse, OpenUSD, PhysX, RTX 기술을 유기적으로 통합하여 로보틱스 개발에 특화된 기능들을 제공하는 레퍼런스 애플리케이션이다.2 이는 로봇 개발자를 위한 일종의 통합 개발 환경(IDE)이라 할 수 있다. 주요 기능으로는 로봇 운영체제(ROS/ROS2)와의 원활한 연동을 위한 브릿지 기능, 대규모 합성 데이터 생성을 자동화하는 Replicator 프레임워크, 그리고 강화학습 알고리즘 훈련을 위한 Isaac Lab 등이 있다.9 MobilityGen은 바로 이 Isaac Sim의 강력한 기능들을 기반으로, 특히 이동 로봇의 데이터 생성에 초점을 맞춰 개발된 전문 툴셋이다. 사용자는 Isaac Sim이 제공하는 풍부한 3D 에셋과 물리 시뮬레이션 환경 위에서 MobilityGen을 통해 손쉽게 원하는 데이터를 생성할 수 있다.

이처럼 개별 기술들이 수직적으로 통합된 '시뮬레이션 스택'은 NVIDIA의 강력한 기술적 해자(moat)를 형성한다. 경쟁 시뮬레이터들이 특정 기능에서는 우수할 수 있으나, NVIDIA는 하드웨어(GPU)부터 렌더링(RTX), 물리(PhysX), 씬 기술(OpenUSD), 애플리케이션(Isaac Sim), 그리고 데이터 생성 툴킷(MobilityGen)에 이르는 전체 파이프라인을 직접 소유하고 최적화한다. 특히 RTX 레이 트레이싱 기반의 사실적인 센서 데이터 생성 8이나 PhysX의 GPU 가속 물리 시뮬레이션을 통한 대규모 병렬 훈련 11은 경쟁사가 쉽게 모방하기 어려운 하드웨어 종속적인 장점이다. OpenUSD라는 개방형 표준을 채택함으로써 생태계 확장을 꾀하는 동시에, 실제로는 자사의 기술 스택 내에서 최상의 성능을 발휘하도록 설계하여 사용자들이 자연스럽게 NVIDIA 생태계에 머무르도록 유도한다. 결국 MobilityGen의 우수한 성능은 단독으로 존재하는 것이 아니라, 이처럼 긴밀하게 통합된 전체 스택의 시너지 효과에 기인하며, 이는 경쟁사가 넘보기 어려운 높은 기술적 장벽을 구축한다.


MobilityGen은 NVIDIA Isaac Sim의 강력한 시뮬레이션 인프라 위에서 이동 로봇 데이터 생성을 자동화하고 간소화하기 위해 설계된 정교한 툴셋이다. 그 아키텍처는 Isaac Sim의 확장 기능(Extension) 시스템에 깊이 통합되어 있으며, 데이터 생성 과정을 효율성과 확장성을 극대화하는 방향으로 설계한 독특한 워크플로우를 따른다.


MobilityGen은 독립 실행형 프로그램이 아니라 Isaac Sim 내에서 선택적으로 활성화하여 사용하는 확장 기능(Extension) 형태로 제공된다.5 Isaac Sim은 모듈화된 아키텍처를 채택하고 있어, 사용자는 필요한 기능만 확장 기능 형태로 로드하여 시뮬레이터를 맞춤 구성할 수 있다. MobilityGen UI 확장 기능을 활성화하면, Isaac Sim 뷰포트 내에 전용 제어판(UI)이 나타난다.5 이 UI를 통해 사용자는 그래픽 인터페이스 상에서 시뮬레이션 환경(USD 파일)을 지정하고, 다양한 로봇 모델과 데이터 수집 시나리오를 드롭다운 메뉴에서 선택하며, 데이터 기록 시작/중지 등의 모든 과정을 직관적으로 제어할 수 있다.4 이러한 통합 방식은 사용자가 Isaac Sim의 다른 기능들, 예를 들어 씬 편집기나 물리 속성 조절기 등과 MobilityGen을 동시에 원활하게 사용할 수 있게 해준다.


MobilityGen의 데이터 생성 과정은 크게 4단계의 체계적인 워크플로우로 구성된다. 각 단계는 데이터의 품질과 생성 효율성을 보장하기 위해 명확한 역할을 수행한다.


데이터 생성의 첫 단계는 로봇이 활동할 가상 환경을 설정하는 것이다. 사용자는 로컬 파일 시스템이나 Omniverse Nucleus 서버에 저장된 USD(Universal Scene Description) 파일을 로드하여 3D 환경을 불러온다.4 이는 단순한 창고부터 복잡한 사무실 환경까지 다양할 수 있다. 환경이 로드되면, MobilityGen은 '점유 격자 지도(Occupancy Map)' 생성 도구를 제공한다.5 점유 격자 지도는 3D 환경을 2D 그리드로 단순화하여, 로봇이 이동할 수 있는 영역(free space)과 장애물로 막힌 영역(occupied space)을 표시한 지도다. 사용자는 로봇의 높이(예: 2m 높이의 장애물 밑을 통과할 수 있는지), 넘어갈 수 있는 턱의 높이(예: 5cm 높이의 장애물을 넘어갈 수 있는지) 등의 파라미터를 설정하여 지도를 생성한다.5 이렇게 생성된 점유 격자 지도는 로봇의 자율 주행에 필수적인 경로 계획(path planning) 알고리즘을 훈련시키고 검증하는 데 핵심적인 그라운드 트루스 데이터로 사용된다.5


환경 설정이 완료되면, 사용자는 어떤 방식으로 데이터를 수집할지를 정의하는 '시나리오(Scenario)'와 시뮬레이션의 주체가 될 '로봇(Robot)'을 선택한다. MobilityGen은 다양한 사전 정의된 옵션을 제공하여 유연성을 높인다.

- **시나리오 유형:** 데이터 수집 목적에 따라 다양한 시나리오를 선택할 수 있다. 인간 전문가의 조종 데이터를 수집하고 싶다면 `KeyboardTeleoperationScenario`나 `GamepadTeleoperation`을 선택한다. 반면, 대규모 데이터를 자동으로 생성하고 싶다면 `RandomPathFollowingScenario`(점유 격자 지도 내에서 무작위 목표 지점을 설정하고 경로를 따라 이동)나 `RandomAccelerationScenario`(무작위로 가속 및 회전 명령을 주어 다양한 움직임 생성)와 같은 자동화된 시나리오를 활용할 수 있다.4
- **로봇 유형:** MobilityGen은 다양한 이동 메커니즘을 가진 로봇들을 기본적으로 지원한다. 바퀴 두 개로 구동하는 차동 구동(Differential drive) 방식의 Jetbot, Carter부터, 보스턴 다이내믹스의 Spot과 같은 사족보행 로봇, Unitree의 H1과 같은 휴머노이드 로봇까지 포함된다.4 사용자는 자신의 연구 목적에 맞는 로봇을 선택하여 시뮬레이션을 구성할 수 있다.


시나리오와 로봇 선택 후 'Start Recording' 버튼을 누르면 본격적인 데이터 기록이 시작된다. 이 단계에서 MobilityGen은 Isaac Sim의 물리 엔진이 매 스텝(physics timestep) 진행될 때마다 로봇의 핵심 상태 데이터를 메모리에 기록하고 파일로 저장한다.4 기록되는 데이터는 로봇의 위치와 방향(pose), 각 관절의 상태(joint states), 그리고 로봇에 가해진 제어 명령(action) 등이다.4 여기서 중요한 점은, 이 단계에서는 계산 비용이 많이 드는 센서 데이터(예: 카메라 이미지)의 렌더링을 수행하지 않는다는 것이다. 오직 로봇의 물리적 움직임에 대한 '로그'만을 가볍고 빠르게 기록한다. 이 덕분에 실시간 성능을 저해하지 않으면서 길고 복잡한 궤적을 원활하게 수집할 수 있다.


궤적 기록이 완료되면, 수집된 데이터는 아직 '반쪽짜리'다. 시각 정보를 포함하고 있지 않기 때문이다. 완전한 데이터셋을 만들기 위해 MobilityGen은 '오프라인 렌더링' 단계를 거친다. 사용자는 Isaac Sim을 종료한 후, 제공되는 `replay_directory.py`와 같은 파이썬 스크립트를 실행한다.4 이 스크립트는 저장된 궤적 로그 파일을 읽어와, 기록된 매 순간의 로봇 위치와 상태를 시뮬레이션 환경에 그대로 재현(replay)한다. 그리고 이 재현 과정에서 각 타임스텝마다 로봇에 부착된 가상 센서(카메라, LiDAR 등)를 통해 데이터를 렌더링한다. 이 과정을 통해 고품질의 RGB 이미지, 깊이 이미지, 세분화 이미지 등 풍부한 시각적 그라운드 트루스 데이터가 생성되어 기존 궤적 데이터에 추가된다.4


MobilityGen의 아키텍처는 사용자가 자신의 특정 요구에 맞게 기능을 확장할 수 있도록 설계되었다. 이는 MobilityGen을 단순한 툴이 아닌, 강력한 연구 개발 플랫폼으로 만드는 핵심 요소다.

- **커스텀 로봇 추가:** 연구자나 개발자는 자신이 설계한 새로운 로봇을 MobilityGen에 쉽게 통합할 수 있다. 제공되는 `MobilityGenRobot` 파이썬 클래스를 상속받아 새로운 클래스를 정의하고, 몇 가지 필수 메소드를 구현하면 된다. `build()` 메소드에서는 로봇의 3D 모델(USD 파일)을 시뮬레이션 씬에 추가하고 물리적 속성을 설정하는 로직을, `write_action()` 메소드에서는 외부로부터 받은 선속도, 각속도 명령을 로봇의 관절 토크나 바퀴 속도로 변환하여 적용하는 제어 로직을 구현한다. 이렇게 정의된 클래스를 MobilityGen에 등록하면, UI의 로봇 선택 메뉴에 새로운 로봇이 자동으로 나타난다.4
- **커스텀 시나리오 추가:** 마찬가지로 `Scenario` 클래스를 상속받아 자신만의 데이터 수집 시나리오를 만들 수 있다. `reset()` 메소드에는 매 에피소드가 시작될 때 로봇의 위치나 주변 환경을 무작위로 배치하는 등의 초기화 로직을, `step()` 메소드에는 매 물리 스텝마다 실행될 로직(예: 특정 조건 만족 시 로봇에 특정 명령 내리기, 이벤트 발생시키기 등)을 구현한다. 이를 통해 '특정 장애물 앞에서 멈추는' 데이터나 '좁은 문을 통과하는' 데이터와 같이 특정 목적을 가진 데이터셋을 체계적으로 구축할 수 있다.4

이러한 '궤적 기록'과 '오프라인 렌더링'을 분리한 비동기식(asynchronous) 데이터 생성 아키텍처는 단순한 계산 효율성 향상을 넘어선 깊은 의미를 가진다. 이는 대규모 데이터셋 생성을 위한 병렬 처리 파이프라인 구축과 실제 데이터와 합성 데이터를 결합하는 하이브리드 데이터 생성의 기반이 된다. 궤적 기록 과정은 로봇의 동역학 계산만 필요하므로 상대적으로 가볍고 빨라, 한 명의 조종사나 자동화 스크립트가 단시간에 수천 개의 다양한 궤적 '로그'를 생성할 수 있다.4 이렇게 생성된 대량의 궤적 로그들은 클라우드 기반의 다중 GPU 렌더링 팜(render farm)으로 전송되어 병렬적으로 렌더링될 수 있다.8 이는 전체 데이터 생성 파이프라인의 처리량(throughput)을 극대화하는 핵심적인 스케일링 전략이다. 더 나아가, 이 구조는 실제 로봇에서 수집된 궤적 데이터(예: 모션 캡처 데이터)를 replay 시스템의 입력으로 사용하여, '실제' 움직임에 '합성' 센서 데이터를 결합하는 강력한 데이터 증강(data augmentation) 기법을 가능하게 한다. 이는 현실 세계에서는 얻기 불가능한 완벽한 그라운드 트루스를 실제 로봇의 움직임에 부착하는 것과 같다. 따라서 이 아키텍처는 데이터 생성의 규모와 질을 동시에 확보하기 위한 정교한 공학적 설계이며, NVIDIA가 대규모 로보틱스 파운데이션 모델 훈련을 염두에 두고 있음을 강력하게 시사한다.2


MobilityGen은 로봇 AI 모델 훈련에 필요한 풍부하고 다층적인 정보를 제공하기 위해 체계적으로 구조화된 데이터를 생성한다. 생성된 데이터는 기록 시점에 따라 크게 정적 데이터와 상태 데이터로 나눌 수 있으며, 특히 상태 데이터에 포함된 다양한 종류의 그라운드 트루스(Ground Truth)는 MobilityGen의 핵심 가치를 이룬다.


MobilityGen이 생성하는 데이터는 그 성격에 따라 다음과 같이 두 가지 주요 범주로 구분된다.4

- **정적 데이터(Static Data):** 이 데이터는 데이터 수집 세션(recording)이 시작될 때 단 한 번만 기록된다. 해당 세션 전체의 환경과 설정을 정의하는 메타데이터 역할을 한다. 여기에는 다음과 같은 정보가 포함된다.
  - **점유 격자 지도 (Occupancy map):** 제2장에서 생성된 2D 지도로, 경로 계획 알고리즘의 입력으로 사용된다.
  - **설정 정보 (Configuration info):** 데이터 생성에 사용된 로봇의 종류, 시나리오의 유형, 그리고 불러온 3D 환경의 USD 파일 경로 등 실험 조건을 명시한다.
  - **USD Stage:** 시뮬레이션 환경을 구성하는 모든 3D 에셋과 그 속성을 담고 있는 USD 씬 그래프 자체를 저장하여, 데이터가 어떤 환경에서 생성되었는지 완벽하게 재현할 수 있도록 한다.
- **상태 데이터(State Data):** 이 데이터는 시뮬레이션이 진행되는 동안 매 물리 스텝(physics timestep)마다 연속적으로 기록되는 동적인 정보이다. 로봇의 시시각각 변화하는 상태와 주변 환경과의 상호작용을 포착한다.


상태 데이터의 핵심은 AI 모델 훈련의 정답지로 사용될 수 있는 완벽하고 노이즈 없는 그라운드 트루스 데이터다. MobilityGen은 기구학적 정보부터 시각 정보까지 폭넓은 그라운드 트루스를 제공한다.4

- **기구학 및 동역학 데이터 (Kinematic and Dynamic Data):** 로봇의 물리적 움직임과 상태를 나타내는 데이터다.
  - **로봇 포즈(Pose):** 시뮬레이션 월드 좌표계를 기준으로 한 로봇의 3차원 위치(Position, `x, y, z`)와 3차원 방향(Orientation)을 나타내는 쿼터니언(Quaternion, `qx, qy, qz, qw`) 값으로 구성된다. 이는 로봇의 위치 추정(Localization)이나 SLAM 알고리즘의 성능을 평가하는 데 사용된다.
  - **관절 위치/속도(Joint Positions/Velocities):** 휴머노이드나 로봇 팔과 같이 여러 관절을 가진 로봇의 경우, 각 관절의 현재 각도(position)와 각속도(velocity)를 기록한다. 이는 로봇의 정교한 모션 제어 정책을 학습시키는 데 필수적이다.
  - **로봇 액션(Action):** 원격 조종이나 자동화된 시나리오에 의해 로봇에 가해진 제어 명령, 즉 목표 선속도(Linear velocity)와 각속도(Angular velocity) 값을 기록한다. 이는 특정 상태(State)에서 어떤 행동(Action)을 취해야 하는지를 학습하는 모방 학습(Imitation Learning)의 핵심 데이터 쌍(State-Action pair)을 구성한다.
- **시각 센서 데이터 (Visual Sensor Data):** 제2장에서 설명한 오프라인 렌더링 단계를 통해 생성되며, 로봇의 '눈' 역할을 하는 센서들을 시뮬레이션한다.
  - **RGB 이미지:** 사람이 보는 것과 같은 표준 컬러 이미지로, 객체 탐지, 이미지 분류 등 전통적인 컴퓨터 비전 작업에 사용된다.
  - **깊이 이미지(Depth Image):** 각 픽셀이 카메라로부터의 직선거리를 실수 값으로 저장하고 있는 흑백 이미지다. 3D 환경을 재구성하거나 장애물과의 거리를 직접적으로 파악하여 충돌을 회피하는 알고리즘 훈련에 매우 유용하다.
  - **세분화 이미지(Segmentation Image):** MobilityGen은 두 종류의 세분화 맵을 제공한다. **시맨틱 세분화(Semantic Segmentation)**는 이미지의 각 픽셀을 '바닥', '벽', '사람'과 같은 사전 정의된 클래스로 분류하여, 로봇이 주행 가능한 영역을 인식하거나 특정 유형의 객체를 찾는 데 사용된다. **인스턴스 세분화(Instance Segmentation)**는 여기서 더 나아가, 같은 클래스에 속하더라도 서로 다른 객체(예: 사람1, 사람2)를 고유한 ID로 구분한다. 이는 개별 객체를 추적하거나 다수의 객체를 다루는 복잡한 작업을 위해 필요하다.
  - **법선 벡터 이미지(Normals Image):** 이미지의 각 픽셀 위치에서 3D 표면이 향하고 있는 방향, 즉 법선 벡터(normal vector) 정보를 `(nx, ny, nz)` 값으로 저장하고 이를 색상으로 시각화한 이미지다. 이는 조명 변화에 강인한 특징을 추출하거나, 객체의 3D 형상을 더 정교하게 이해하는 데 도움을 준다.


이렇게 생성된 방대한 데이터는 개발자가 쉽게 접근하고 활용할 수 있도록 체계적인 포맷으로 저장된다. MobilityGen GitHub 저장소에는 이 데이터 구조를 파싱하고 읽어들이는 `reader.py`와 같은 헬퍼 클래스가 제공되어, 사용자는 몇 줄의 파이썬 코드만으로 데이터셋을 로드하고 처리할 수 있다.5 또한, 데이터의 내용을 직관적으로 확인하고 분석할 수 있도록 Gradio 라이브러리를 활용한 웹 기반 시각화 예제 스크립트도 함께 제공된다.4 사용자는 이 도구를 통해 기록된 궤적을 재생해보고, 각 시점의 RGB, 깊이, 세분화 이미지를 나란히 비교하며 데이터의 품질을 검증할 수 있다.

이처럼 다양한 종류의 데이터를 체계적으로 생성하고 활용할 수 있는 환경을 제공함으로써, MobilityGen은 로보틱스 연구자들이 자신의 AI 모델에 가장 적합한 데이터를 맞춤형으로 생성하고 실험 과정을 가속화할 수 있도록 지원한다. 아래 표는 MobilityGen이 생성하는 주요 데이터 유형과 그 활용 분야를 요약한 것이다.

**Table 3.1: MobilityGen 생성 데이터 유형 및 활용 분야 (MobilityGen Generated Data Types and Applications)**

| 데이터 유형 (Data Type)                 | 기술적 설명 (Technical Description)                          | 주요 활용 분야 (Primary Application)                         |
| --------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 점유 격자 지도 (Occupancy Map)          | 로봇이 이동 가능한 2D 공간을 격자로 표현한 맵. 장애물 영역은 점유(occupied), 빈 공간은 비점유(free)로 표시. | 전역/지역 경로 계획(Global/Local Path Planning) 알고리즘 훈련 및 검증. |
| 로봇 포즈 (Robot Pose)                  | 월드 좌표계 기준 로봇의 3D 위치(`x, y, z`)와 3D 방향(`qx, qy, qz, qw` 쿼터니언). | 로봇 위치 추정(Localization), SLAM(Simultaneous Localization and Mapping) 알고리즘 평가. |
| 관절 상태 (Joint States)                | 로봇의 각 관절(joint)의 현재 각도(position)와 각속도(velocity). | 로봇 동역학 모델 학습, 모션 제어(Motion Control) 정책 훈련.  |
| RGB 이미지                              | 표준 컬러 카메라 이미지.                                     | 객체 탐지(Object Detection), 이미지 분류(Image Classification), 시각 기반 네비게이션(Visual Navigation). |
| 깊이 이미지 (Depth Image)               | 각 픽셀에 카메라로부터의 거리를 실수 값으로 저장한 흑백 이미지. | 3D 환경 재구성, 장애물 거리 측정 및 충돌 회피(Collision Avoidance). |
| 시맨틱 세분화 (Semantic Segmentation)   | 이미지의 각 픽셀을 클래스(예: 바닥, 벽, 사람)로 분류한 맵.   | 주행 가능 영역(Drivable Area) 탐지, 씬 이해(Scene Understanding). |
| 인스턴스 세분화 (Instance Segmentation) | 시맨틱 세분화에서 더 나아가, 동일 클래스의 다른 객체들을 개별 인스턴스로 구분한 맵. | 개별 객체 추적(Object Tracking), 다중 객체 조작(Multi-object Manipulation) 계획. |
| 법선 벡터 이미지 (Normals Image)        | 각 픽셀 위치에서의 표면 법선 벡터(`nx, ny, nz`)를 색상으로 인코딩한 이미지. | 3D 형상 복원, 표면 디테일 분석, 조명 조건에 강인한 특징 추출. |


MobilityGen이 생성하는 고품질 합성 데이터는 그 자체로도 가치가 있지만, 그 진정한 잠재력은 최신 생성형 AI 모델, 특히 확산 모델(Diffusion Model)과 결합될 때 발현된다. 확산 모델은 최근 몇 년간 AI 분야에서 가장 주목받는 기술 중 하나로, 이미지, 오디오 등 복잡하고 고차원적인 데이터 생성에서 기존의 생성적 적대 신경망(GAN)을 능가하는 성능을 보여주며 지배적인 패러다임으로 자리 잡았다.12 MobilityGen의 데이터로 훈련된 확산 모델은 단순히 기존 데이터를 모방하는 것을 넘어, 새로운 로봇 모션이나 사실적인 센서 데이터를 창발적으로 생성할 수 있다.


과거 생성형 모델 연구는 주로 GAN이 주도했다. GAN은 생성자(Generator)와 판별자(Discriminator)가 서로 경쟁하며 학습하는 구조로, 매우 사실적인 결과물을 만들어냈지만 고질적인 문제점을 안고 있었다. 학습 과정이 매우 불안정하여 최적의 파라미터를 찾기 어렵고, 생성자가 데이터 분포의 일부 모드(mode)에만 집중하여 다양한 결과물을 만들지 못하는 '모드 붕괴(Mode Collapse)' 현상이 자주 발생했다.12 확산 모델은 이러한 GAN의 단점을 극복하는 대안으로 부상했다. 확산 모델은 학습 과정이 훨씬 안정적이며, 데이터 분포의 다양한 모드를 잘 학습하여 결과물의 다양성이 높다는 장점을 가진다.12


확산 모델의 아이디어는 비평형 열역학(non-equilibrium thermodynamics)에서 영감을 받았다.13 핵심 원리는 간단하다. 깨끗한 이미지에 점진적으로 노이즈를 추가하여 완전히 무작위한 노이즈로 만드는 과정(Forward Process)을 정의하고, 이 과정을 거꾸로 되돌려 순수한 노이즈에서부터 원본 이미지를 복원하는 방법(Reverse Process)을 신경망으로 학습시키는 것이다.12


확산 과정은 원본 데이터 $x_0$가 주어졌을 때, 총 $T$ 스텝에 걸쳐 점진적으로 가우시안 노이즈를 추가하는 마르코프 연쇄(Markov Chain)로 정의된다. 이 과정은 미리 정해진 스케줄에 따라 진행되므로 모델이 학습할 필요가 없다.12

$t$번째 스텝에서의 데이터 $x_t$는 이전 스텝의 데이터 $x_{t-1}$로부터 다음과 같은 조건부 확률 분포를 통해 샘플링된다.
$$
q(x_t \rvert x_{t-1}) = \mathcal{N}(x_t; \sqrt{1 - \beta_t} x_{t-1}, \beta_t \mathbf{I})
$$
여기서 $\mathcal{N}$은 가우시안(정규) 분포를, $\mathbf{I}$는 단위 행렬을 의미한다. $\beta_t$는 $t$ 스텝에서 추가되는 노이즈의 양을 조절하는 하이퍼파라미터로, '분산 스케줄(variance schedule)'이라고 불린다. 일반적으로 $t$가 커질수록(즉, 과정이 진행될수록) $\beta_t$ 값도 커지도록 설정하여($0 < \beta_1 < \beta_2 <... < \beta_T < 1$), 점차 더 많은 노이즈가 추가되도록 한다.13

이 과정의 중요한 수학적 속성 중 하나는, 어떤 스텝 $t$에서의 $x_t$를 $x_0$로부터 직접 계산할 수 있다는 점이다. $\alpha_t = 1 - \beta_t$이고, $\bar{\alpha}_t = \prod_{i=1}^{t} \alpha_i$라고 정의하면, 재매개변수화 트릭(reparameterization trick)을 통해 $x_t$는 다음과 같이 표현될 수 있다.
$$
x_t = \sqrt{\bar{\alpha}_t} x_0 + \sqrt{1 - \bar{\alpha}_t} \epsilon, \quad \text{where } \epsilon \sim \mathcal{N}(0, \mathbf{I})
$$
이는 $x_t$가 원본 데이터 $x_0$를 $\sqrt{\bar{\alpha}_t}$만큼 축소시키고, 표준 정규 분포에서 샘플링한 노이즈 $\epsilon$을 $\sqrt{1 - \bar{\alpha}_t}$만큼 확대하여 더한 것과 같다는 의미다. 이 공식을 통해 학습 과정에서 매번 중간 단계를 계산할 필요 없이, 임의의 스텝 $t$에 대한 노이즈 낀 데이터를 효율적으로 생성할 수 있다.


확산 모델 학습의 핵심은 이 확산 과정을 거꾸로 되돌리는 역확산 과정이다. 즉, 순수한 가우시안 노이즈 $x_T \sim \mathcal{N}(0, \mathbf{I})$에서 시작하여, $x_{T-1}, x_{T-2},..., x_0$를 순차적으로 복원해내는 방법을 배우는 것이다.12 이 역 과정 $p_\theta(x_{t-1} \rvert x_t)$를 모델링하기 위해 심층 신경망을 사용한다. 만약 $\beta_t$가 충분히 작다면, 이 역 과정 또한 가우시안 분포를 따른다고 근사할 수 있다.

모델의 목표는 $x_t$와 타임스텝 $t$가 주어졌을 때, $x_{t-1}$의 분포를 예측하는 것이다. 하지만 실제로는 $x_{t-1}$ 자체를 예측하는 대신, $x_t$를 만드는 데 사용된 노이즈 $\epsilon$을 예측하도록 신경망 $\epsilon_\theta(x_t, t)$를 훈련시키는 것이 더 안정적이고 좋은 성능을 보인다. 신경망의 구조로는 이미지의 공간적 구조를 잘 포착하는 U-Net 아키텍처가 널리 사용되며, 최근에는 트랜스포머(Transformer)를 백본으로 사용하여 모델의 확장성과 성능을 더욱 높이는 연구도 활발히 진행되고 있다.12


신경망 $\epsilon_\theta$를 훈련시키기 위한 손실 함수는 이론적으로는 변분 하한(Variational Lower Bound, VLB 또는 Evidence Lower Bound, ELBO)을 최대화하는 것과 관련이 있지만 14, 실제 구현에서는 훨씬 간단하고 직관적인 형태의 손실 함수가 사용된다. 이는 각 스텝 $t$에서 실제 추가된 노이즈 $\epsilon$과 신경망이 예측한 노이즈 $\epsilon_\theta(x_t, t)$ 사이의 평균 제곱 오차(Mean Squared Error, MSE)를 최소화하는 것이다.13
$$
L_{\text{simple}}(\theta) = \mathbb{E}_{t \sim, x_0, \epsilon \sim \mathcal{N}(0, \mathbf{I})} \left[ \left\| \epsilon - \epsilon_\theta(\sqrt{\bar{\alpha}_t} x_0 + \sqrt{1 - \bar{\alpha}_t} \epsilon, t) \right\|^2 \right]
$$
이 손실 함수는 직관적이다. 신경망에게 노이즈가 낀 데이터($x_t = \sqrt{\bar{\alpha}_t} x_0 + \sqrt{1 - \bar{\alpha}_t} \epsilon$)와 현재 타임스텝($t$)을 보여주고, "이 데이터를 만드는 데 사용된 원래 노이즈 $\epsilon$이 무엇이었는지 맞춰보라"고 요구하는 것과 같다. 모델은 역전파(backpropagation)와 경사 하강법(gradient descent)을 통해 이 손실 함수를 최소화하도록 가중치 $\theta$를 반복적으로 업데이트하며, 점차 노이즈를 정확하게 예측하는 능력을 학습하게 된다.13

MobilityGen의 맥락에서 이러한 확산 모델의 원리는 특별한 의미를 가진다. 확산 모델은 단순히 '사실적인 이미지'를 생성하는 것을 넘어, 데이터에 내재된 물리적 제약과 동적인 상호작용을 암묵적으로 학습하는 '데이터 기반 물리 시뮬레이터'로서의 잠재력을 품고 있다. MobilityGen은 PhysX와 RTX를 통해 물리적으로 정확한 데이터를 생성하며, 이 데이터에는 로봇의 동역학, 센서와 환경의 상호작용 등 복잡한 물리 법칙이 이미 인코딩되어 있다. 확산 모델은 이 데이터셋의 분포를 학습함으로써, 픽셀 값의 통계적 분포뿐만 아니라 픽셀들 간의 공간적, 시간적 상관관계, 즉 물리 법칙의 '결과'를 학습하게 된다. 예를 들어, 확산 모델이 로봇의 다음 프레임 이미지를 생성할 때, 광원의 위치에 따라 그림자가 자연스럽게 드리워지거나 로봇의 바퀴가 지면과 올바르게 접촉하는 이미지를 생성한다면, 이는 모델이 빛의 직진성이나 중력과 같은 물리 법칙을 데이터로부터 암묵적으로 학습했음을 의미한다. 이는 NVIDIA가 추구하는 두 가지 시뮬레이션 접근법, 즉 '명시적 물리 기반 시뮬레이션'(PhysX, RTX)과 '데이터 기반 생성형 시뮬레이션'(확산 모델)의 융합 가능성을 시사한다. 미래에는 MobilityGen과 같은 명시적 시뮬레이터로 고품질 데이터를 생성하고, 이 데이터로 확산 모델을 훈련시킨 뒤, 훈련된 확산 모델을 사용하여 훨씬 더 빠르고 다양한 시나리오를 생성하는 하이브리드 방식이 시뮬레이션 기술의 새로운 지평을 열 수 있다.


MobilityGen의 궁극적인 목표는 자율 이동 로봇의 지능을 향상시키는 데 필요한 고품질 데이터를 대규모로 공급하는 것이다. 실제 로봇을 이용한 데이터 수집은 비용, 시간, 안전 문제로 인해 대규모 데이터셋 구축에 한계가 명확하다. MobilityGen은 이러한 데이터 부족 문제(data scarcity)를 해결하고, 모방 학습(Imitation Learning) 및 강화 학습(Reinforcement Learning)과 같은 최신 AI 기법을 로보틱스에 효과적으로 적용할 수 있는 길을 열어준다.6


로봇이 다양한 환경에서 강인하게 동작하기 위해서는 수많은 예시 데이터를 통한 학습이 필수적이다. MobilityGen은 사용자가 원하는 거의 모든 종류의 환경과 시나리오를 가상으로 구축하고, 그 안에서 로봇의 움직임과 센서 데이터를 자동으로 생성 및 레이블링할 수 있다.6 예를 들어, 조명 조건을 새벽부터 한낮, 해질녘까지 다양하게 바꾸거나, 바닥의 재질을 바꾸고, 예기치 않은 동적 장애물을 추가하는 등의 작업을 통해 데이터셋의 다양성을 손쉽게 확보할 수 있다. 이는 실제 세계에서는 수개월이 걸릴 데이터 수집 작업을 단 며칠 만에 완료할 수 있게 하여, 로봇 학습 모델의 개발 주기를 획기적으로 단축시킨다.


모방 학습은 전문가(주로 인간)의 시연 데이터를 학습하여 로봇이 전문가의 행동을 모방하도록 만드는 기법이다. MobilityGen은 모방 학습에 이상적인 데이터셋을 생성하는 데 매우 효과적이다. 사용자는 키보드나 게임패드를 이용한 원격 조종(teleoperation) 시나리오를 통해 특정 과업(예: 창고 내 특정 지점까지 물건 운반)을 수행하는 전문가 궤적 데이터를 쉽게 기록할 수 있다.6 이 데이터에는 각 시간 스텝별 로봇의 관측값(State, 예: 카메라 이미지, LiDAR 스캔)과 그에 해당하는 인간의 조종 명령(Action, 예: 직진, 좌회전)이 쌍으로 기록된다. AI 모델은 이 '상태-행동' 쌍 데이터를 학습하여, 주어진 관측값에 대해 전문가와 유사한 행동을 출력하는 정책(policy)을 학습하게 된다. 또한, MobilityGen으로 생성된 다양한 가상 환경은 이렇게 학습된 정책이 새로운, 보지 못했던 환경에서도 얼마나 잘 일반화되는지를 평가하고 강인성을 높이는 데 사용된다.


강화 학습은 로봇(에이전트)이 환경과 상호작용하며 받은 보상(reward)을 최대화하는 방향으로 행동 정책을 스스로 학습하는 기법이다. 이 과정에서 에이전트는 수백만 번의 시행착오를 거쳐야 하므로, 실제 로봇으로 학습을 진행하는 것은 비효율적이고 위험하다. Isaac Sim의 물리적으로 정확한 시뮬레이션 환경은 강화학습 에이전트가 마음껏 탐험하고 실패하며 배울 수 있는 안전하고 효율적인 '가상 놀이터'를 제공한다.3 MobilityGen은 여기서 한 걸음 더 나아가, 절차적 콘텐츠 생성(Procedural Content Generation, PCG) 기법을 통해 매 학습 에피소드마다 환경의 구조, 장애물의 위치, 목표 지점 등을 무작위로 변경하는 시나리오를 자동으로 생성할 수 있다. 이러한 '영역 무작위화(domain randomization)' 기법은 에이전트가 특정 환경에만 과적합(overfitting)되는 것을 방지하고, 다양한 상황에 대처할 수 있는 일반화된 정책을 학습하도록 돕는다.9


MobilityGen은 단순히 공개된 툴셋에 그치지 않고, NVIDIA Research에서 진행하는 여러 첨단 로보틱스 연구 프로젝트의 핵심적인 기반 기술로 적극 활용되고 있다. 이는 MobilityGen이 실제 연구 현장에서 그 가치를 입증하고 있음을 보여준다.

- **COMPASS (Cross-embOdiment Mobility Policy via ResiduAl RL and Skill Synthesis):** 이 프로젝트의 목표는 특정 로봇 기종(embodiment)을 넘어 다양한 형태의 로봇에 적용될 수 있는 일반화된 이동 정책을 개발하는 것이다.15 연구진은 MobilityGen을 사용하여 특정 로봇(예: 휴머노이드 H1)에 대한 대규모 전문가 시연 데이터를 생성하고, 이를 통해 기본 모방 학습 정책을 사전 훈련시킨다. 그 후, 이 정책을 다른 로봇(예: 사족보행 로봇 Spot)에 적용하고, 각 로봇의 고유한 동역학적 특성에 맞는 미세 조정을 강화 학습(Isaac Lab 활용)을 통해 수행한다. 이 연구는 MobilityGen 데이터가 로봇 개발의 확장성 문제를 해결하는 데 어떻게 기여할 수 있는지를 보여주는 대표적인 사례다.6
- **X-Mobility:** 이 프로젝트는 시뮬레이션에서 훈련된 정책을 현실 세계의 로봇에 별도의 추가 학습 없이 바로 적용하는 'Zero-shot Sim-to-Real' 전이(transfer)를 목표로 한다.15 이를 위해서는 시뮬레이션 데이터가 현실 세계를 매우 정밀하게 모사해야 한다. MobilityGen은 RTX 기반의 사실적인 렌더링과 PhysX 기반의 정확한 물리 시뮬레이션을 통해 Sim-to-Real 격차를 최소화한 데이터를 생성함으로써, X-Mobility와 같은 야심 찬 연구의 성공 가능성을 높인다.6
- **Physical AI 파운데이션 모델 훈련:** 최근 NVIDIA는 인간과 유사한 작업을 수행할 수 있는 휴머노이드 로봇을 위한 파운데이션 모델 프로젝트 'GR00T'을 발표했다. 이러한 거대 모델을 훈련시키기 위해서는 인터넷의 텍스트 데이터에 버금가는 방대한 양의 물리적 상호작용 데이터가 필요하다. MobilityGen은 이러한 파운데이션 모델 훈련에 필요한 대규모 합성 모션 및 센서 데이터를 생성하는 핵심적인 역할을 수행하며, 로보틱스 분야의 'ChatGPT' 순간을 앞당기는 데 기여하고 있다.2

결론적으로, MobilityGen의 핵심 가치는 특정 로봇을 위한 데이터를 생성하는 것을 넘어선다. 이는 다양한 로봇 형태(embodiments), 환경, 작업을 아우르는 일반화된 'Physical AI' 모델의 등장을 촉진하는 촉매제 역할을 한다. 전통적인 로봇 개발이 특정 로봇을 특정 환경에서 특정 작업을 수행하도록 개별 프로그래밍하는 방식이었다면, COMPASS와 같은 연구는 하나의 AI 모델이 바퀴 달린 로봇과 사족보행 로봇을 모두 제어하는 것을 목표로 한다.6 이러한 일반화된 모델을 훈련시키기 위해서는 다양한 로봇, 환경, 조명 조건 등을 포함하는 거대하고 이질적인(heterogeneous) 데이터셋이 필수적이다. MobilityGen은 바로 이러한 데이터셋을 합성적으로, 대규모로, 저비용으로 생성할 수 있는 핵심 도구다.4 따라서 MobilityGen은 로보틱스가 '수작업 프로그래밍'의 시대를 넘어 '데이터 기반 학습'의 시대로, 더 나아가 '파운데이션 모델'의 시대로 나아가는 데 있어 필수적인 기반 시설이라 할 수 있다.


MobilityGen이 개별 로봇의 이동성(mobility)에 초점을 맞추고 있다면, 그 기술적 기반과 철학은 더 넓은 범위의 자율 시스템, 특히 자율주행차(Autonomous Vehicle, AV) 시뮬레이션으로 확장된다. NVIDIA DRIVE Sim은 자율주행차 개발 및 검증을 위해 설계된 플랫폼으로, 단일 에이전트의 움직임을 넘어 다수의 차량, 보행자, 자전거 등이 상호작용하는 복잡하고 동적인 '교통(traffic)' 환경을 시뮬레이션하는 것을 목표로 한다.16 이는 자율주행 시스템의 안전성을 보장하기 위한 필수적인 과정이다.17


자율주행 시뮬레이션의 가장 큰 도전 과제 중 하나는 현실 세계의 교통 흐름을 얼마나 사실적으로 재현하는가이다. 초기의 교통 시뮬레이터들은 주로 '차량은 앞차와 일정한 거리를 유지한다' 또는 '차선 변경 시 특정 규칙을 따른다'와 같은 명시적인 규칙 기반(rule-based) 모델을 사용했다. 그러나 이러한 모델들은 인간 운전자의 예측 불가능하고, 때로는 비합리적이며, 문화적 차이까지 보이는 다양하고 미묘한 행동 양식을 제대로 모사하지 못하는 명백한 한계를 가진다.19 자율주행차가 실제 도로에서 마주칠 무한에 가까운 시나리오에 대비하기 위해서는, 실제 주행 데이터를 기반으로 학습된 데이터 기반(data-driven) 생성형 AI 모델을 통해 교통 참여자들의 행동을 생성하는 것이 필수적이다.20


NVIDIA Research는 이러한 데이터 기반 교통 시뮬레이션의 한계를 극복하기 위해 활발한 연구를 진행하고 있으며, 여러 혁신적인 생성형 AI 모델을 발표했다. 이 모델들은 MobilityGen과 마찬가지로 현실성과 다양성을 높이는 동시에, 개발자가 원하는 특정 시나리오를 생성할 수 있는 '제어 가능성(controllability)'을 확보하는 방향으로 발전하고 있다.

- **BITS (Bi-level Imitation for Traffic Simulation):** BITS는 인간의 운전 행동이 복합적인 의사결정 과정이라는 점에 착안한 모델이다.19 이 모델은 운전 문제를 두 가지 계층으로 분리한다. '상위 수준(high-level)'에서는 운전자의 장기적인 목표나 의도(intent), 예를 들어 '좌회전하겠다' 또는 '현재 차선을 유지하겠다'를 예측한다. '하위 수준(low-level)'에서는 상위 수준에서 결정된 의도를 달성하기 위한 구체적인 조향 및 가감속 제어(control)를 모방 학습한다. 이처럼 문제를 분리함으로써, 모델은 다양한 의도를 조합하여 복잡하고 다채로운 주행 시나리오를 효율적으로 생성할 수 있다. 실험 결과, BITS는 다른 모델에 비해 시나리오의 커버리지와 다양성을 각각 64%, 118% 개선하고, 충돌이나 도로 이탈과 같은 실패율을 36% 낮추는 뛰어난 성능을 보였다.20
- **CTG (Controllable Traffic Generation):** BITS가 실제 데이터 분포를 최대한 사실적으로 재현하는 데 중점을 둔다면, CTG는 개발자가 원하는 특정 조건을 만족하는 시나리오를 생성하는 '제어 가능성'에 더 초점을 맞춘다.22 이 모델은 제4장에서 설명한 확산 모델(Diffusion Model)을 기반으로 한다. CTG의 핵심 아이디어는 확산 모델의 노이즈 제거 과정(reverse process)에 외부적인 '가이드(guidance)'를 제공하는 것이다. 이 가이드는 '시속 80km를 유지하라' 또는 '5초 안에 교차로를 통과하라'와 같은 규칙을 수학적으로 표현한 신호 시간 논리(Signal Temporal Logic, STL)를 사용하여 정의된다. 생성 과정의 매 스텝에서, 모델은 생성 중인 궤적이 주어진 규칙을 만족시키는 방향으로 그래디언트(gradient)를 계산하고, 이를 노이즈 예측에 반영한다. 이를 통해 CTG는 물리적으로 가능하면서도 개발자가 의도한 특정 조건을 만족하는 궤적을 생성할 수 있다.
- **Scenario Diffusion:** 이 모델은 한 단계 더 나아가, 개별 차량의 궤적뿐만 아니라 교통 씬(scene) 전체의 에이전트 분포를 생성하는 것을 목표로 한다.23 Scenario Diffusion은 Stable Diffusion과 같은 이미지 생성 모델에 사용되는 잠재 공간 확산 모델(Latent Diffusion Model) 아키텍처를 활용한다. 모델의 입력으로는 도로망 정보가 담긴 조감도(bird's-eye-view) 맵 이미지와, 생성하고자 하는 시나리오의 핵심 특징을 기술하는 '토큰(token)'이 사용된다. 예를 들어, '자율주행차는 교차로 중앙에 위치하고, 상호작용하는 상대 차량은 왼쪽에서 접근한다'와 같은 정보를 토큰으로 제공하면, 모델은 이 조건을 만족시키면서 주변의 다른 차량들의 위치, 방향, 궤적을 현실적인 분포로 함께 생성해낸다. 이는 소수의 핵심 에이전트만 제어하여 전체 시나리오를 일관성 있고 사실적으로 채우는 강력한 제어 가능 생성 능력을 보여준다.


MobilityGen과 DRIVE Sim은 각각 로봇과 자율주행차라는 다른 도메인을 다루지만, 그 근간에는 NVIDIA Omniverse, PhysX, RTX라는 동일한 기술 스택을 공유하고 있다. MobilityGen을 통해 축적된 물리 기반 합성 데이터 생성 기술, 다양한 센서 모델링 노하우, 그리고 절차적 환경 생성 기법들은 DRIVE Sim의 훨씬 더 복잡한 다중 에이전트 시뮬레이션 환경을 구축하는 데 직접적으로 활용된다. 또한, MobilityGen이 로봇의 모션 데이터를 생성하여 모방 학습 모델을 훈련시키듯, DRIVE Sim은 실제 주행 로그 데이터를 기반으로 BITS나 Scenario Diffusion과 같은 정교한 생성형 AI 교통 모델을 훈련시킨다. 결국 두 플랫폼은 '데이터 기반 물리 AI'라는 NVIDIA의 거대한 비전 아래, 시뮬레이션을 통해 현실 세계를 이해하고 예측하며, 더 나아가 안전하고 지능적인 자율 시스템을 구축하려는 동일한 목표를 향해 나아가고 있다.

아래 표는 앞서 설명한 NVIDIA의 대표적인 AI 기반 교통 시뮬레이션 모델들의 특징을 비교하여 요약한 것이다.

**Table 6.1: AI 기반 교통 시뮬레이션 모델 비교 (Comparison of AI-based Traffic Simulation Models)**

| 모델명 (Model Name)       | 핵심 방법론 (Core Methodology)                 | 제어성 (Controllability)     | 현실성/다양성 (Realism/Diversity)      | 주요 특징 (Key Features)                                     |
| ------------------------- | ---------------------------------------------- | ---------------------------- | -------------------------------------- | ------------------------------------------------------------ |
| **BITS** 19               | 2단계 모방 학습 (Bi-level Imitation Learning)  | 제한적 (의도 기반)           | 높음 (실제 데이터 분포 모사)           | 운전 행동을 '의도'와 '제어'로 분리하여 학습 효율성과 다양성 극대화. |
| **CTG** 22                | 조건부 확산 모델 (Conditional Diffusion Model) | 높음 (STL 기반 가이드)       | 중간 (제어와 현실성 간 트레이드오프)   | 신호 시간 논리(STL)를 통해 '속도 80km/h 유지'와 같은 명시적 규칙을 부여하여 궤적 생성 제어. |
| **Scenario Diffusion** 23 | 잠재 공간 확산 모델 (Latent Diffusion Model)   | 높음 (토큰 기반 조건부 생성) | 높음 (사실적인 다중 에이전트 상호작용) | 지도와 소수 에이전트의 상태(토큰)만으로 전체 씬의 차량 분포와 궤적을 일관성 있게 생성. |


NVIDIA MobilityGen과 그 기반 기술들은 로보틱스 및 자율 시스템 개발에 혁신적인 가능성을 제시하지만, 동시에 기술적 한계와 해결해야 할 과제들도 명확히 존재한다. 이러한 한계를 이해하고 미래 발전 방향을 예측하는 것은 기술을 올바르게 평가하고 활용하기 위해 필수적이다.


가장 근본적이고 오래된 도전 과제는 'Sim-to-Real' 격차다. 아무리 정교한 시뮬레이션이라도 현실 세계를 100% 완벽하게 모사할 수는 없다.24 이 격차는 여러 요인에서 발생한다. 첫째, 물리 엔진의 한계다. PhysX와 같은 최신 물리 엔진도 복잡한 마찰, 연체 변형, 공기 저항과 같은 현상을 근사적으로 계산하므로 미세한 오차가 존재할 수 있다. 둘째, 센서 모델링의 불완전성이다. 실제 센서는 온도, 진동, 렌즈 왜곡 등에 의한 노이즈가 발생하지만, 시뮬레이션에서 이러한 모든 비선형적 노이즈를 완벽하게 모델링하기는 어렵다. 셋째, 3D 에셋의 한계다. 가상 환경을 구성하는 3D 모델과 텍스처의 사실성에는 한계가 있으며, 이는 렌더링 결과물과 실제 카메라 이미지 간의 미묘한 차이(domain gap)를 유발한다.25 이러한 요인들로 인해 시뮬레이션에서 완벽하게 작동하던 AI 모델이 실제 로봇에 탑재되었을 때 예상치 못한 성능 저하를 겪을 수 있다.


MobilityGen이 생성하는 합성 데이터는 많은 장점을 가지지만, 그 자체로 몇 가지 내재적 한계를 지닌다.

- **편향(Bias) 문제:** 생성형 모델은 학습 데이터의 통계적 분포를 모방한다. 만약 시뮬레이션 환경을 구축하는 데 사용된 3D 에셋이나 텍스처가 특정 스타일이나 지역에 편중되어 있다면(예: 서구식 건물 모델만 사용), 생성된 합성 데이터에도 동일한 편향이 전파될 수 있다.26 이러한 편향된 데이터로 학습한 AI 모델은 새로운, 보지 못했던 환경(out-of-distribution)에 대한 일반화 성능이 저하될 수 있다.
- **희귀 사례(Edge Case) 재현의 어려움:** 합성 데이터는 기본적으로 학습된 데이터 분포 내에서 샘플링하는 방식으로 생성된다. 이는 일반적이고 빈번한 시나리오를 대량으로 생성하는 데는 효과적이지만, 현실에서 발생할 수 있는 극히 드물고 예측 불가능한 '꼬리 분포(tail distribution)'의 사건들을 생성하는 데는 본질적인 어려움을 겪는다.28 예를 들어, 도로에 갑자기 소파가 떨어지는 것과 같은 기이한 상황은 사전에 모델링되지 않았다면 생성하기 어렵다.
- **품질 관리의 어려움:** 생성된 데이터가 통계적으로는 실제 데이터와 유사해 보일지라도, 논리적으로나 물리적으로 미묘하지만 치명적인 오류를 포함할 수 있다. 예를 들어, 로봇의 바퀴가 지면을 미세하게 파고드는 현상이나, 물리 법칙에 위배되는 움직임이 포함될 수 있다. 대규모 데이터셋에서 이러한 미세한 오류들을 모두 검증하고 걸러내는 것은 매우 어려운 과제다.30


확산 모델과 같은 생성형 AI를 시뮬레이션에 직접 활용하는 접근법은 새로운 가능성을 열지만, 동시에 새로운 도전 과제를 제시한다.

- **환각(Hallucination):** 생성형 AI 모델이 학습 데이터에 근거하지 않은, 겉보기에는 그럴듯하지만 실제로는 사실이 아니거나 논리적으로 맞지 않는 결과물을 만들어내는 '환각' 현상은 시뮬레이션의 신뢰도를 저해하는 심각한 문제다.31 예를 들어, 생성된 이미지에 그림자가 광원의 위치와 반대 방향으로 드리워지거나, 존재하지 않아야 할 객체가 나타나는 경우가 발생할 수 있다.
- **계산 비용:** 고품질의 합성 데이터를 대규모로 생성하고, 수십억 개의 파라미터를 가진 거대 생성형 모델을 훈련시키는 데에는 막대한 양의 컴퓨팅 자원(고성능 GPU, 대용량 스토리지, 네트워크 대역폭)이 요구된다.32 이는 기술 접근성에 장벽으로 작용할 수 있으며, 지속 가능한 AI 개발을 위한 에너지 효율성 문제와도 직결된다.


이러한 한계와 도전에도 불구하고, MobilityGen과 생성형 AI 기반 시뮬레이션 기술의 미래는 매우 밝다. 앞으로 다음과 같은 방향으로 기술이 발전할 것으로 전망된다.

- **절차적 콘텐츠 생성(Procedural Content Generation, PCG)의 고도화:** 현재의 영역 무작위화(domain randomization)가 주로 파라미터를 변경하는 수준에 머물렀다면, 미래의 PCG는 AI가 스스로 복잡하고 의미론적으로 일관된 시뮬레이션 환경과 시나리오를 창발적으로 생성하는 방향으로 발전할 것이다. 예를 들어, AI가 '위험한 교차로'라는 상위 수준의 목표를 받으면, 그에 맞는 교통량, 신호 체계, 보행자 동선 등을 포함한 시나리오를 자동으로 설계하고 생성하는 것이다.
- **실시간 Sim-to-Real 적응:** Sim-to-Real 격차를 줄이기 위해, 시뮬레이션과 실제 로봇을 분리하지 않고 동시에 운영하는 패러다임이 부상할 것이다. 실제 로봇이 현실 세계에서 동작하며 얻는 데이터와 시뮬레이션 예측 간의 차이를 실시간으로 분석하고, 이 차이를 줄이는 방향으로 시뮬레이션 파라미터를 동적으로 업데이트하는 '온라인 Sim-to-Real 적응(Online Sim-to-Real Adaptation)' 기술이 중요해질 것이다.
- **하이브리드 시뮬레이션:** 제4장에서 제시된 분석과 같이, 미래 시뮬레이션은 단일한 접근법에 의존하지 않을 것이다. 명시적인 물리 법칙 계산에 기반한 물리 기반 시뮬레이션(PhysX)의 '정확성'과, 대규모 데이터로부터 학습된 생성형 시뮬레이션(Diffusion Model)의 '속도 및 다양성'을 결합한 하이브리드 접근법이 주류가 될 것이다. 예를 들어, 시나리오의 핵심적인 물리 상호작용은 PhysX로 정밀하게 계산하고, 배경 환경이나 비주요 에이전트의 행동은 확산 모델로 빠르고 다양하게 생성하는 방식이다.



본 보고서를 통해 다각적으로 분석한 NVIDIA MobilityGen은 단순히 이동 로봇을 위한 합성 데이터를 생성하는 툴을 넘어, 로보틱스 연구 개발의 패러다임을 근본적으로 전환시키는 잠재력을 지닌 핵심 기술이라 평가할 수 있다. MobilityGen은 NVIDIA가 수직적으로 통합한 강력한 '시뮬레이션 스택'—Omniverse의 협업 환경, OpenUSD의 개방형 데이터 구조, PhysX의 물리적 정확성, RTX의 시각적 사실성—위에서 동작하며, 고품질의 레이블링된 데이터를 저비용으로 대량 생산할 수 있게 한다. 이는 로보틱스 개발의 가장 큰 병목이었던 데이터 수집 문제를 해결하고, 개발자들이 실제 하드웨어의 제약에서 벗어나 'Sim-First' 접근법을 통해 더 빠르고 안전하며 창의적으로 AI 모델을 개발하고 검증할 수 있는 환경을 제공한다.


MobilityGen은 기술적 가치를 넘어 NVIDIA의 생태계 전략에서 중요한 역할을 수행한다. 오픈소스 정책과 높은 사용자 정의 확장성은 전 세계 개발자 커뮤니티의 참여를 유도하고, Isaac Sim 플랫폼을 중심으로 한 강력한 기술 생태계를 구축하는 원동력이 된다. 이는 특정 기업이나 연구소에 국한되었던 첨단 로봇 시뮬레이션 기술의 민주화를 이끌고, 더 많은 연구자들이 Physical AI 분야에 진입할 수 있도록 돕는다. 궁극적으로 MobilityGen과 Isaac Sim 생태계는 COMPASS, X-Mobility와 같은 첨단 연구를 촉진하고, GR00T과 같은 Physical AI 파운데이션 모델의 등장을 가속화함으로써, 로보틱스 산업 전체의 발전에 기여할 것이다.


MobilityGen과 그 기반이 되는 생성형 AI 기술의 융합은 미래 시뮬레이션 기술이 나아갈 방향을 명확히 보여준다. 미래의 시뮬레이션은 더 이상 현실 세계를 수동적으로 '모방(imitation)'하는 도구에 머무르지 않을 것이다. 그것은 확산 모델과 같은 생성형 AI의 힘을 빌려, 현실에는 아직 존재하지 않는 무한한 가능성의 시나리오를 능동적으로 '생성(generation)'하고 탐색하는 창의적인 도구로 진화할 것이다. 이러한 가상 세계에서의 무한한 경험을 통해 AI 에이전트의 지능을 현실의 물리적, 시간적 제약을 뛰어넘어 극한까지 끌어올리는 것, 이것이 바로 생성형 시뮬레이션이 추구하는 궁극적인 목표다. NVIDIA MobilityGen은 그 위대한 진화의 여정을 알리는 중요한 첫걸음이라 할 수 있다.


1. A Systematic Review of Synthetic Data Generation Techniques Using Generative AI - MDPI, 8월 20, 2025에 액세스, https://www.mdpi.com/2079-9292/13/17/3509
2. Isaac Sim - Robotics Simulation and Synthetic Data Generation - NVIDIA Developer, 8월 20, 2025에 액세스, https://developer.nvidia.com/isaac/sim
3. Robotics Simulation | Use Case - NVIDIA, 8월 20, 2025에 액세스, https://www.nvidia.com/en-us/use-cases/robotics-simulation/
4. NVlabs/MobilityGen: Data Generation Pipeline for Mobility - GitHub, 8월 20, 2025에 액세스, https://github.com/NVlabs/MobilityGen
5. Data Generation with MobilityGen - Isaac Sim Documentation, 8월 20, 2025에 액세스, https://docs.isaacsim.omniverse.nvidia.com/latest/synthetic_data_generation/tutorial_replicator_mobility_gen.html
6. R²D²: Advancing Robot Mobility and Whole-Body Control with Novel Workflows and AI Foundation Models from NVIDIA Research, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/r2d2-advancing-robot-mobility-whole-body-control-with-ai-from-nvidia-research/
7. Generating High-Quality Motion Data for Robotics With MobilityGen - Course Detail | NVIDIA, 8월 20, 2025에 액세스, https://learn.nvidia.com/courses/course-detail?course_id=course-v1:DLI+S-OV-37+V1
8. Isaac Sim | NVIDIA NGC, 8월 20, 2025에 액세스, https://catalog.ngc.nvidia.com/orgs/nvidia/containers/isaac-sim
9. A Beginner's Guide to Simulating and Testing Robots with ROS 2 and NVIDIA Isaac Sim, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/a-beginners-guide-to-simulating-and-testing-robots-with-ros-2-and-nvidia-isaac-sim/
10. NVIDIA Isaac Sim - GitHub, 8월 20, 2025에 액세스, https://github.com/isaac-sim
11. Robot Learning in Simulation | Use Case - NVIDIA, 8월 20, 2025에 액세스, https://www.nvidia.com/en-us/use-cases/robot-learning/
12. 확산 모델 - 나무위키, 8월 20, 2025에 액세스, [https://namu.wiki/w/%ED%99%95%EC%82%B0%20%EB%AA%A8%EB%8D%B8](https://namu.wiki/w/확산 모델)
13. 확산 모델이란 무엇인가요? - IBM, 8월 20, 2025에 액세스, https://www.ibm.com/kr-ko/think/topics/diffusion-models
14. 새롭게 등장하는 '확산 모델'의 구현 5選, 8월 20, 2025에 액세스, https://turingpost.co.kr/p/5-new-diffusion-models
15. j3soon/nvidia-isaac-summary - GitHub, 8월 20, 2025에 액세스, https://github.com/j3soon/nvidia-isaac-summary
16. Autonomous Vehicle (AV) Simulation - NVIDIA Developer, 8월 20, 2025에 액세스, https://developer.nvidia.com/drive/simulation
17. AI 로봇의 다음 물결 가속화 - NVIDIA, 8월 20, 2025에 액세스, https://www.nvidia.com/ko-kr/industries/robotics/
18. 자율주행 자동차 기술 및 솔루션 - NVIDIA, 8월 20, 2025에 액세스, https://www.nvidia.com/ko-kr/solutions/autonomous-vehicles/
19. Simulating Realistic Traffic Behavior with a Bi-Level Imitation Learning AI Model, 8월 20, 2025에 액세스, https://developer.nvidia.com/blog/simulating-realistic-traffic-behavior-with-a-bi-level-imitation-learning-ai-model/
20. BITS: Bi-level Imitation for Traffic Simulation | Research, 8월 20, 2025에 액세스, https://research.nvidia.com/publication/2023-05_bits-bi-level-imitation-traffic-simulation
21. 양방향 모방 학습 AI 모델로 실제와 같은 교통 행동 시뮬레이션하기 ..., 8월 20, 2025에 액세스, https://developer.nvidia.com/ko-kr/blog/simulating-realistic-traffic-behavior-with-a-bi-level-imitation-learning-ai-model/
22. Guided Conditional Diffusion for Controllable Traffic Simulation - Research at NVIDIA, 8월 20, 2025에 액세스, https://research.nvidia.com/publication/2023-05_guided-conditional-diffusion-controllable-traffic-simulation
23. Scenario Diffusion: Controllable Driving Scenario Generation With ..., 8월 20, 2025에 액세스, https://proceedings.neurips.cc/paper_files/paper/2023/file/d95cb79a3421e6d9b6c9a9008c4d07c5-Paper-Conference.pdf
24. Data-Driven Modeling of Multifaceted Individual Mobility Behavior, 8월 20, 2025에 액세스, https://www.research-collection.ethz.ch/bitstream/handle/20.500.11850/737776/thesis-full-YeHong-upload_A2b.pdf?sequence=1&isAllowed=y
25. What are the limits of synthetic data generation? : r/computervision - Reddit, 8월 20, 2025에 액세스, https://www.reddit.com/r/computervision/comments/1czdyyi/what_are_the_limits_of_synthetic_data_generation/
26. What is ChatGPT, DALL-E, and generative AI? | McKinsey, 8월 20, 2025에 액세스, https://www.mckinsey.com/featured-insights/mckinsey-explainers/what-is-generative-ai
27. The benefits and limitations of generating synthetic data - Syntheticus, 8월 20, 2025에 액세스, https://syntheticus.ai/blog/the-benefits-and-limitations-of-generating-synthetic-data
28. On the Challenges and Opportunities in Generative AI - arXiv, 8월 20, 2025에 액세스, https://arxiv.org/html/2403.00025v3
29. Synthetic Data - what, why and how? - Royal Society, 8월 20, 2025에 액세스, https://royalsociety.org/-/media/policy/projects/privacy-enhancing-technologies/Synthetic_Data_Survey-24.pdf
30. What is Synthetic Data? - AWS, 8월 20, 2025에 액세스, https://aws.amazon.com/what-is/synthetic-data/
31. Opportunities and Challenges of Generative AI in Construction Industry: Focusing on Adoption of Text-Based Models - MDPI, 8월 20, 2025에 액세스, https://www.mdpi.com/2075-5309/14/1/220
32. NVIDIA, AI 클라우드 제공업체를 위한 레퍼런스 아키텍처 공개, 8월 20, 2025에 액세스, https://blogs.nvidia.co.kr/blog/ai-cloud-providers-reference-architecture/
33. What is Generative AI? - IBM, 8월 20, 2025에 액세스, https://www.ibm.com/think/topics/generative-ai
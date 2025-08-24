# NVIDIA Isaac Sim에서의 자연 환경 절차적 생성
[NVidia Omniverse](./index.md)


NVIDIA Isaac Sim은 단일 애플리케이션이 아닌, 모듈화된 플랫폼으로서 환경 생성에 대한 질문은 전체 생태계의 구조를 이해하는 것에서부터 시작해야 합니다. 다양한 환경적 특징들이 왜, 그리고 어떻게 각기 다른 도구와 워크플로우를 통해 생성되는지를 파악하려면 이 아키텍처에 대한 이해가 필수적입니다.


- **NVIDIA Isaac Sim:** 핵심 시뮬레이션 루프, 센서 모델, 로보틱스 유틸리티를 제공하는 중앙 애플리케이션으로, 다른 모든 구성 요소의 통합 지점 역할을 합니다.1
- **NVIDIA Omniverse 및 OpenUSD:** 기술적 근간을 이룹니다. OpenUSD(Universal Scene Description)는 장면 구성을 위한 프레임워크로, 협업적이고 비파괴적인 워크플로우를 가능하게 합니다. 모든 환경, 로봇, 에셋은 USD 프리미티브(prim)로 표현됩니다.1 환경을 생성한다는 것은 본질적으로 USD 스테이지를 프로그래밍 방식으로 저작하는 것이므로 이는 매우 기본적인 개념입니다.
- **NVIDIA PhysX 5:** 고성능의 GPU 가속 물리 엔진입니다. 경사면 위 로봇 발의 접촉부터 흩뿌려진 바위들의 충돌에 이르기까지 모든 동적 상호작용은 PhysX에 의해 관리됩니다.1 API를 통해 힘과 토크를 직접 적용하는 기능은 유체 역학과 같이 기본적으로 모델링되지 않은 현상을 시뮬레이션하는 핵심 열쇠가 됩니다.5


- **Isaac Lab:** 로봇 학습(강화 학습, 모방 학습)을 위한 주요 오픈 소스 프레임워크입니다.4 Isaac Sim 

  *위에* 구축되었으며, **절차적 지형 생성**과 같은 고급 워크플로우를 위해 지정된 도구입니다.4 장면, 이벤트, 행동 관리자 등으로 구성된 관리자 기반 아키텍처는 복잡하고 무작위화된 훈련 시나리오를 구조적으로 처리하는 방법을 제공합니다.8

- **Omniverse Replicator:** 합성 데이터 생성(SDG)을 위한 강력한 SDK입니다. 주된 목표는 인식 모델 훈련을 위한 레이블링된 데이터셋을 만드는 것이지만, 강력한 무작위화 및 에셋 배치 기능은 환경을 채우는 데 필수적입니다.1 숲을 만들기 위해 나무를 흩뿌리는 것과 같은 작업에 가장 적합한 도구입니다.

- **Python 스크립팅:** 모든 것을 연결하는 보편적인 접착제입니다. Isaac Sim은 `SimulationApp`을 통한 앱 초기화부터 개별 객체 조작 및 물리 스텝 실행에 이르기까지 시뮬레이션의 모든 측면을 제어할 수 있는 광범위한 Python API를 제공합니다.10 Isaac Lab이나 Replicator와 같은 상위 레벨 프레임워크에서 다루지 않는 기능은 이러한 Python API를 사용하여 직접 구축하는 것이 해결책입니다.


플랫폼의 구조를 면밀히 살펴보면, 구조화되고 "즉시 사용 가능한(batteries-included)" Isaac Lab 프레임워크와 Replicator 및 핵심 PhysX의 보다 근본적인 "구성 요소(building-block)" API 사이에 명확한 구분이 존재함을 알 수 있습니다.4 Isaac Lab은 다양한 지형에서의 보행 로봇과 같은 일반적인 로보틱스 연구 문제를 즉시 해결하도록 설계되었습니다.6 반면, Replicator와 핵심 API는 사용자 정의 문제를 해결하기 위한 일반화된 도구를 제공합니다.

이러한 구분은 사용자가 해결하려는 문제의 성격에 따라 접근 방식을 달리해야 함을 시사합니다. 예를 들어, "산"과 같은 지형 생성은 Isaac Lab의 핵심 기능인 "절차적 지형 생성"으로 명시되어 있으며, 이는 `AnymalTerrain` 예제와 같이 로보틱스 연구를 직접 지원하는, 플랫폼 내에서 해결된 문제입니다.4 반대로, "강"이나 "도로"는 기본 기능으로 언급되지 않으며, 커뮤니티 포럼에서는 PhysX 힘을 사용한 수동 스크립팅이나 외부에서 제작된 에셋을 가져오는 방식을 제안합니다.5 이는 플랫폼 관점에서 아직 해결되지 않은 문제입니다.

이러한 차이는 우연이 아닙니다. NVIDIA는 로봇 학습 발전에 핵심적인 문제에 대해 상위 레벨 프레임워크(Isaac Lab) 개발에 집중했습니다. 그 외의 모든 것에 대해서는 강력한 하위 레벨 API를 제공하며 "확장성을 특징으로 하는" 철학을 채택했습니다. 따라서 Isaac Sim을 성공적으로 활용하기 위해서는 해결하려는 문제가 어떤 범주에 속하는지 정확히 파악하는 것이 중요합니다. 표준적인 강화 학습 작업이라면 Isaac Lab이 명확한 경로이지만, 강과 같은 사용자 정의 환경 기능을 구현해야 한다면 Python, PhysX API, 그리고 잠재적으로 외부 DCC(디지털 콘텐츠 제작) 도구를 사용한 상당한 수준의 맞춤형 개발을 준비해야 합니다. 이는 필요한 팀의 기술 역량이 단순한 로보틱스를 넘어 3D 그래픽 및 물리 프로그래밍까지 확장되어야 함을 의미합니다.


이 섹션에서는 "산"과 지형적 특징으로서의 "배수로"에 대한 사용자의 질문을 직접적으로 다루며, Isaac Lab을 주요 해결책으로 제시합니다.


`TerrainGenerator`는 생태계 내에서 절차적 지형 생성을 위한 핵심 클래스입니다.4 이 기능이 Isaac Lab의 일부라는 점은 이 기능이 로봇 학습 워크플로우와 밀접하게 연결되어 있음을 강화합니다.4 높이 맵(height field)과 `trimesh` 라이브러리를 사용한 생성을 모두 지원하여 유연성을 제공합니다.7

`TerrainImporter` 클래스는 생성된 지형 메시를 물리적 객체(USD prim)로 시뮬레이션에 가져오는 프로세스를 처리하며 `TerrainGenerator`와 함께 작동합니다.14


생성 과정은 Python 데이터 클래스를 통해 세밀하게 구성할 수 있습니다. `TerrainGeneratorCfg`의 주요 매개변수는 다음과 같습니다.

- `size`: 지형의 (x, y) 차원입니다.
- `horizontal_scale`, `vertical_scale`: 해상도와 높이의 과장 정도를 제어합니다.
- `sub_terrains`: 평평한 지대, 계단, 경사면 또는 노이즈 기반의 무작위 지형과 같은 다양한 유형의 지형을 혼합하여 정의하는 딕셔너리입니다 (`SubTerrainBaseCfg` 사용).14
- `curriculum`: 시간이 지남에 따라 지형의 난이도를 높일 수 있게 하는 불리언 값으로, 강화 학습에 있어 매우 중요한 기능입니다.14


- **산과 언덕:** `SubTerrainBaseCfg`를 노이즈 함수(예: 펄린 노이즈)와 적절한 `vertical_scale`로 구성하여 생성할 수 있습니다. `proportion` 매개변수를 사용하면 언덕 지형과 평지를 혼합할 수 있습니다.14
- **배수로와 도랑:** "배수"를 위한 특정 기능은 없지만, 하위 지형들을 조합하여 만들 수 있습니다. 예를 들어, 대부분 평평한 지형을 생성한 후 프로그래밍 방식으로 음수 높이의 지형을 추가하거나, 맞춤 설계된 높이 맵을 사용할 수 있습니다.
- **AnymalTerrain 예제:** 폐기된 `OmniIsaacGymEnvs`의 `AnymalTerrain` 환경과 Isaac Lab의 후속 버전들은 이러한 기능들을 사용하여 험난하고 절차적으로 생성된 지형에서 보행 로봇을 훈련시키는 표준적인 예시로 사용됩니다.12


실제 지형 데이터(예: 디지털 고도 모델)나 아티스트가 조각한 지형이 필요한 시나리오를 위해, Isaac Sim은 `heightmap` 임포터 확장 기능(`isaacsim.asset.importer.heightmap`)을 제공합니다.17 이 워크플로우는 높이 맵 이미지(PNG 등)를 제공하고 물리적 속성을 구성하는 과정을 포함합니다. 이는 순수하게 절차적인 생성과 실제 세계의 디지털 트윈 사이의 간극을 메우는 강력한 방법입니다.18


`TerrainGeneratorCfg`에 `curriculum` 및 `difficulty_range`와 같은 명시적인 매개변수가 포함되어 있다는 점은 주목할 만합니다.14 이 전체 시스템은 단일의 무작위 지형을 만드는 것뿐만 아니라, 점진적으로 복잡성이 증가하는 일련의 지형을 생성하도록 설계되었습니다. 이러한 기능은 정적인 게임 환경에서는 불필요하지만, 강화 학습에서는 그 가치가 매우 큽니다. 강화 학습 에이전트(예: 사족보행 로봇)는 간단한 과제(평지)에서 시작하여 점차 어려운 과제(가파른 경사, 계단, 험지)로 나아갈 때 더 효과적으로 학습합니다.

`AnymalTerrain` 예제는 이러한 사용 사례를 명확히 보여줍니다.12 목표는 보이지 않는 환경에서도 일반화할 수 있는 견고한 보행 정책을 훈련하는 것입니다. 이는 `TerrainGenerator`의 설계가 강화 학습의 요구에 의해 주도되었음을 보여줍니다. 즉, 이 도구는 Unreal 엔진이나 Houdini와 같은 게임 엔진이나 DCC 도구에서 볼 수 있는 범용적인 조경 도구가 아닙니다.19 Isaac Lab의 지형 도구는 강화 학습 에이전트를 위해 수많은 병렬적이고, 비교적 단순하며, 물리적으로 활성화된 훈련장을 만드는 데 최적화되어 있습니다.20 따라서 사용자는 이 도구가 영화 같은 장면을 위한 단일의 거대하고 시각적으로 멋진 산맥을 만드는 데는 적합하지 않다는 점을 이해해야 합니다. 도구의 목적이 그 아키텍처와 능력을 결정하기 때문입니다.


이 섹션에서는 생성된 지형 위에 나무, 바위, 초목과 같은 에셋을 절차적으로 배치하여 "숲"을 만드는 방법을 다룹니다. 이 워크플로우는 주로 Omniverse Replicator에 의존합니다.


Replicator는 합성 데이터 생성을 위해 장면 요소를 무작위화하는 주요 도구입니다.1 위치, 회전, 크기, 텍스처, 조명 등을 무작위화하는 기능은 자연 환경을 채우는 데 완벽하게 부합합니다. 핵심 워크플로우는 에셋 세트(예: 다양한 나무 모델)와 표면(지형 프리미티브)을 정의한 다음, Replicator를 사용하여 해당 표면에 에셋을 흩뿌리는 것입니다.


이 함수는 이 작업의 핵심입니다. 2D 표면에 프리미티브를 분산시킵니다.21 중요한 것은 

`check_for_collisions` 인수를 포함하고 있어, 흩뿌려진 객체(나무나 바위 등)가 서로 관통하지 않도록 하여 물리적으로 더 그럴듯한 장면을 만듭니다.21 이 함수는 스크립트 편집기나 독립 실행형 애플리케이션에서 Python 스크립트로 사용할 수 있습니다.23 문서는 이 개념을 사용하여 팔레트에 상자를 흩뿌리는 장면 기반 SDG 스크립트의 전체 예제를 제공하며, 이는 지형에 나무를 흩뿌리는 작업에 직접적으로 적용될 수 있습니다.23


- **SimReady 에셋:** 성공적인 숲을 만들려면 좋은 에셋 라이브러리가 필요합니다. Isaac Sim은 1,000개 이상의 SimReady 3D 에셋에 대한 액세스를 제공하며, 이를 시작점으로 사용할 수 있습니다.1
- **사용자 정의 에셋 가져오기:** 더 많은 다양성을 위해 사용자는 자신만의 에셋을 가져와야 합니다. Isaac Lab과 Isaac Sim은 FBX, OBJ, glTF와 같은 표준 형식의 임포터를 제공합니다.25 이는 사용자가 온라인 마켓플레이스에서 초목을 구하거나 직접 만들 수 있음을 의미합니다.
- **외부 DCC 통합 (Houdini/Blender):** 매우 복잡하고 사실적인 생태계(예: 절차적 성장 패턴, 경사 및 고도에 따른 종 분포)를 위해서는 Houdini와 같은 전용 DCC 도구를 사용하는 것이 가장 강력한 워크플로우입니다. NVIDIA 포럼의 한 사용자는 Houdini가 절차적 초목 생성 분야에서 "최고의 목록"에 있으며 Omniverse 커넥터가 있다고 언급했습니다.26 워크플로우는 Houdini에서 복잡한 초목 레이아웃을 생성하고, 이를 USD로 내보낸 후 Isaac Sim 스테이지로 가져오는 방식이 될 것입니다.27


객체를 흩뿌리는 주요 도구인 Replicator는 공식적으로 합성 데이터 생성(SDG)을 위한 것으로 설명되지만 1, 여기서는 환경 *생성*을 위해 활용되고 있습니다. 이 현상은 두 작업의 근본적인 요구 사항이 동일하기 때문에 발생합니다. Replicator의 목적은 인식 모델을 훈련시키기 위해 장면을 무작위화하는 것이며, 이를 위해 객체를 배치하고, 조명을 변경하며, 텍스처를 수정해야 합니다.9 "숲"을 만드는 과정 역시 나무 객체를 배치하고, 회전과 크기를 무작위화하는 등 정확히 동일한 단계를 포함합니다.

핵심은 절차적으로 생성된 환경이 결국 합성 데이터의 한 형태라는 점을 깨닫는 것입니다. 차이점은 의도에 있습니다. SDG에서는 레이블이 지정된 2D/3D 출력 데이터가 목표이지만, 환경 생성에서는 물리 기반 시뮬레이션에 사용될 3D USD 스테이지 자체가 목표입니다. 두 작업 모두 프로그래밍 방식의 장면 조작이라는 근본적인 필요성을 공유하기 때문에, 상업적으로 중요한 SDG 분야를 위해 개발된 강력한 도구(Replicator)가 절차적 환경 생성이라는 연구 지향적 분야의 사실상 표준 해결책이 됩니다.

이는 Isaac Sim에서 환경 생성을 마스터하려면 Replicator SDK를 마스터해야 함을 의미합니다. 최종 목표가 데이터셋 생성이 아니라 강화 학습 에이전트가 살아갈 세계를 만드는 것이라 할지라도, Replicator 튜토리얼을 학습해야 합니다.23 그 기술은 직접적으로 이전 가능하며, `scene_based_sdg.py` 예제는 중요한 시작점이 됩니다.23


이 섹션은 "강"과 물이 채워진 "배수로"에 대한 질문을 다루며, 이는 플랫폼의 한계와 확장성을 동시에 보여주는 주제입니다.


- **내장된 유체 시뮬레이션 부재:** 문서와 커뮤니티 포럼을 통해 Isaac Sim에는 바다나 강과 같은 대규모 동적 수역 또는 유체 시뮬레이션 시스템이 내장되어 있지 않음이 명확합니다.5
- **입자 시스템 대 수역:** 입자 기반 유체 데모가 존재하지만(예: 컵 속의 물) 32, 이는 소규모 효과를 위한 것이며 크고 안정적인 수역을 만드는 데는 적합하지 않습니다. 사용자들은 대규모 입자 시스템이 계산 비용이 많이 들고 불안정하다고 보고합니다.30
- **NVIDIA Flow:** 이는 사실적인 유체 시뮬레이션을 위한 Omniverse 생태계의 별도 구성 요소입니다.34 그러나 로보틱스 사용 사례를 위해 Isaac Sim과 간단하게 통합되는 기능은 아닙니다. "PhysX Flow"에 대한 포럼 토론은 결론에 이르지 못했으며, 별도의 비통합 확장 기능임을 시사합니다.35


물속 객체 시뮬레이션을 위해 공식적으로 권장되는 접근 방식은 Python 스크립트를 통해 수동으로 구현하는 것입니다.5

- **프로세스:**
  1. **시각적 수면:** DCC 도구(예: Blender)에서 물 표면을 나타내는 정적 3D 메시를 만듭니다. 이는 시각적 목적만을 가집니다.
  2. **사용자 정의 물리 스크립트:** 모든 시뮬레이션 단계마다 실행되는 Python 스크립트를 작성합니다.
  3. **힘 계산:** 스크립트 내에서 객체의 형상, 속도, 수면 대비 깊이를 기반으로 부력, 항력, 양력과 같은 유체 역학적 힘을 계산합니다.
  4. **힘 적용:** PhysX API, 특히 `RigidBody.add_force()`와 같은 함수를 사용하여 계산된 힘을 로봇/객체에 적용합니다.


공개된 GitHub 프로젝트 `isaac_underwater`는 이 수동 워크플로우의 완벽한 실제 예시입니다.36 이 프로젝트에는 잠긴 부피를 기반으로 부력을 계산하는 `buoyancy_forces.py` 스크립트가 포함되어 있습니다.36 또한 댐핑 및 제어 스크립트도 포함되어 있어, 단순화되었지만 완전한 수중 ROV 시뮬레이션 구현을 보여줍니다. 저자는 시뮬레이션이 "아직 100% 물리적으로 정확하지 않다"고 명시하며, 이 수동 접근 방식의 어려움을 강조합니다.36


OceanSim과 같은 연구 프로젝트의 존재는 플랫폼의 궁극적인 잠재력을 보여줍니다.37 OceanSim은 Isaac Sim *위에* 구축된 맞춤형 확장 기능으로, 수중 효과를 고려한 소나 및 카메라의 고급 렌더링을 포함한 고품질 수중 시뮬레이션을 제공합니다. 이는 Isaac Sim이 기본 지원은 부족하지만, 확장 가능한 아키텍처를 통해 전담 팀이 매우 전문적이고 사실적인 솔루션을 구축할 수 있음을 증명합니다.37


수중 시뮬레이션 워크플로우는 지형 생성이나 에셋 분산에 비해 훨씬 복잡하고 다릅니다. 이는 사전 구축된 도구를 사용하는 것이 아니라, 물리 원리와 프로그래밍에 대한 깊은 지식을 요구합니다. 지형 생성을 위해서는 사용자가 상위 레벨 클래스(`TerrainGeneratorCfg`)를 구성하고, 숲을 만들기 위해서는 상위 레벨 함수(`rep.scatter_2d`)를 호출하면 됩니다. 그러나 강을 만들기 위해서는 부력에 대한 아르키메데스의 원리나 항력에 대한 유체 역학 방정식을 처음부터 구현해야 합니다.5

이는 요구되는 전문 지식 수준의 상당한 도약을 의미합니다. 강화 학습에 익숙한 로보틱스 연구원은 Isaac Lab을 편안하게 사용할 수 있지만, 계산 유체 역학 전문가는 아닐 수 있습니다. 이러한 높은 진입 장벽은 "확장성을 특징으로 하는" 패러다임의 직접적인 결과입니다. NVIDIA는 핵심 물리 엔진(PhysX)과 힘을 적용할 API를 제공하고, 유체 역학과 같은 특정 분야의 물리 구현은 사용자나 제3자 개발자에게 맡깁니다. 따라서 물, 진흙 또는 복잡한 연체 상호작용과 같은 복잡한 자연 현상을 시뮬레이션하려는 팀은 전문적인 연구 개발을 위한 예산을 책정해야 합니다. `isaac_underwater` 프로젝트는 기본적인 구현에 필요한 노력의 수준을 보여주는 좋은 본보기이며, 고품질 솔루션을 위해서는 OceanSim과 같은 학술 연구 프로젝트에 버금가는 노력이 필요합니다.37


이 섹션은 "도로" 생성을 다루며, 이는 물과 마찬가지로 비-네이티브 워크플로우를 필요로 하고, 모션 플래닝과 환경 생성 간의 차이점을 부각시키는 기능입니다.


문서와 커뮤니티 토론을 철저히 검토한 결과, Isaac Sim에는 절차적으로 도로 메시를 생성하는 내장 도구가 없음이 확인되었습니다.13 이는 게임 엔진에서는 일반적인 기능이지만 40, 여기서는 부재합니다.


Isaac Sim의 Motion Generation 확장 기능에는 로봇 매니퓰레이터나 모바일 베이스를 위한 부드러운 스플라인 기반 궤적을 생성할 수 있는 `LulaTrajectoryGenerator`가 포함되어 있습니다.41 여기서 중요한 차이점은 이 도구가 로봇이 *따라갈 경로*를 생성하는 것이지, 물리적인 도로 *메시 자체*를 생성하는 것이 아니라는 점입니다. 즉, "로봇이 A에서 B로 어떻게 움직여야 하는가?"라는 문제를 해결하는 것이지, "로봇이 움직이는 지면이 어떻게 생겼는가?"라는 문제를 해결하는 것이 아닙니다.41 마찬가지로, NVIDIA cuOpt와 같은 도구는 환경 내 로봇 군집을 위한 상위 레벨의 경로 *계획* 및 최적화를 위한 것이지, 환경의 지오메트리를 생성하기 위한 것이 아닙니다.42


- **1. 외부 DCC 도구에서 가져오기 (권장):**
  - 가장 실용적이고 강력한 워크플로우는 외부 애플리케이션에서 도로망을 설계하고 가져오는 것입니다.
  - **도구:** Blender, Houdini 또는 전문 토목 공학 소프트웨어. Blender와 Houdini는 시뮬레이션을 위한 에셋 제작과 관련하여 자주 언급됩니다.26 포럼 사용자들은 복잡한 트랙을 만들 때 이 접근 방식을 제안합니다.13
  - **프로세스:** 교차로와 차선을 포함하여 도로를 모델링하고, FBX 또는 직접 USD로 내보냅니다. 그런 다음 Mesh Importer를 사용하여 Isaac Sim으로 가져옵니다.25 도로는 장면에 정적 메시 프리미티브가 되어 로봇의 충돌 표면을 제공합니다.
- **2. 프로그래밍 방식의 메시 생성 (고급):**
  - 더 통합적이지만 복잡한 접근 방식은 Isaac Sim 내에서 Python 스크립트를 작성하여 도로 메시를 생성하는 것입니다.
  - **개념:** 이는 3D 스플라인을 따라 2D 도로 단면 프로파일을 "스위핑(sweeping)"하는 것과 같이 게임 개발에서 사용되는 기술을 모방합니다.39
  - **구현:** 이는 Python에서 하위 레벨 USD 및 메시 조작 API를 사용하여 정점, 면, UV 좌표를 정의하는 상당한 양의 사용자 정의 코드를 필요로 합니다. 가능은 하지만, 상당한 소프트웨어 개발 작업입니다.


플랫폼은 로봇 모션을 위한 정교한 도구(`LulaTrajectoryGenerator`, cuOpt)를 갖추고 있지만, 도로 생성기와 같은 기본적인 세계 구축 도구는 부족합니다. 이는 Isaac Sim의 주된 목적이 "AI 기반 로보틱스 솔루션을 시뮬레이션하고 테스트하는 것"이기 때문입니다.1 즉, 초점은 로봇의 인식, 제어, 학습 알고리즘에 맞춰져 있으며, 환경은 중요하지만 종종 로봇이 행동할 "주어진" 무대로 취급됩니다.

NVIDIA는 로봇 중심 도구에 막대하게 투자했습니다: 로봇 임포터(URDF/MJCF) 25, 모션 생성기 41, 강화 학습 프레임워크(Isaac Lab) 6, ROS 통합.1 게임 개발에서 흔히 볼 수 있는 절차적 도로 생성이나 초목 페인팅과 같은 세계 구축 도구는 로봇 알고리즘 개발이라는 핵심 문제에 중심적이지 않기 때문에 부재합니다. 사용자가 단순한 절차적 지형, 사전 구축된 환경(창고 에셋 등) 44, 또는 DCC 도구에서 가져온 사용자 정의 세계를 사용할 것이라는 가정이 깔려 있습니다.

결론적으로, Isaac Sim의 기능 세트는 그 대상 사용자와 문제 영역인 로보틱스 연구원의 요구를 직접적으로 반영합니다. 사용자는 이러한 "로봇 중심적" 사고방식을 채택해야 합니다. 프로젝트를 계획할 때, 도로망, 건물, 상세 인프라와 같은 복잡하고 정적인 환경 부분은 외부 소프트웨어에서 생성하여 가져와야 한다고 가정해야 합니다. Isaac Sim의 절차적 기능은 무작위 지형, 객체 배치, 물리 속성과 같이 로봇의 학습 및 동적 상호작용에 직접적인 영향을 미치는 요소에 대해서만 사용하는 것이 바람직합니다.


이 결론 섹션에서는 분석 결과를 종합하여 일관된 전략을 제시하고, 명확한 요약과 실용적인 통합 워크플로우 예시를 제공합니다.


이 표는 보고서의 결과를 한눈에 볼 수 있도록 요약하여 프로젝트 계획에 귀중한 참고 자료를 제공합니다.

**표: Isaac Sim의 절차적 생성 기능**

| 기능          | 주요 도구/프레임워크                 | 기본 지원 수준  | 핵심 API / 개념                                              | 권장 워크플로우                                              |
| ------------- | ------------------------------------ | --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **산/지형**   | Isaac Lab                            | **높음**        | `TerrainGenerator`, `TerrainImporter`, 높이 맵               | 동적 및 커리큘럼 기반 지형에는 Isaac Lab의 절차적 생성기 사용. 실제 데이터의 경우 높이 맵 가져오기. |
| **숲/초목**   | Omniverse Replicator                 | **중간**        | `rep.scatter_2d`, `rep.modify.pose`, 에셋 라이브러리         | Replicator를 사용하여 SimReady 또는 사용자 정의 초목 에셋을 분산 배치. 고도로 복잡한 생태계는 Houdini에서 생성 후 USD로 가져오기. |
| **강/물**     | 사용자 정의 Python / PhysX           | **낮음 (수동)** | `RigidBody.add_force()`, 사용자 정의 힘 계산                 | Python 스크립트를 통해 강체에 힘을 적용하여 유체 역학 모델링. 수면은 비물리적 시각적 메시. |
| **도로/경로** | 외부 DCC 도구 / 사용자 정의 스크립팅 | **낮음 (수동)** | 에셋 임포터 (FBX, USD), `LulaTrajectoryGenerator` (동작용, 메시 생성용 아님) | Blender/Houdini에서 도로망을 모델링하고 정적 메시로 가져오기. 로봇 궤적 생성과 도로 메시 생성을 혼동하지 말 것. |


이 개념적 스크립트는 이러한 이질적인 기술들을 단일의 통합된 환경 생성 프로세스로 결합하는 방법을 개괄적으로 설명하여 사용자에게 실용적인 가이드를 제공합니다.

```Python
# 개념적 스크립트: 완전한 자연 환경 생성

from isaacsim import SimulationApp
simulation_app = SimulationApp({"headless": True})

from omni.isaac.core import World
import omni.replicator.core as rep
from isaaclab.scene import InteractiveScene # Isaac Lab 컨텍스트 가정
from isaaclab.terrains import TerrainGenerator, TerrainGeneratorCfg

# 1. 설정: World 및 Scene 초기화
world = World()
# Isaac Lab의 InteractiveScene을 사용하면 에셋 관리가 용이
# scene = InteractiveScene(world) # 실제 코드에서는 Isaac Lab 환경 내에서 관리됨

# 2. 도로망: 사전 제작된 도로망 가져오기
# 이 에셋은 Blender/Houdini에서 제작되어 Nucleus에 저장됨
road_network_usd_path = "omniverse://nucleus/Projects/MySim/Assets/road_network.usd"
world.scene.add(XformPrim(prim_path="/World/Roads", name="road_network_ref", usd_path=road_network_usd_path))

# 3. 지형: 도로 주변에 절차적 지형 생성
# 지형은 도로를 위한 '구멍'이나 평탄화된 영역과 함께 생성될 수 있음
terrain_cfg = TerrainGeneratorCfg(
    size=(2000, 2000), 
    #... 기타 매개변수...
    # 도로와 혼합하기 위해 사용자 정의 높이 맵 사용 가능
)
# 실제 Isaac Lab에서는 환경 구성 파일에서 이를 정의
# terrain_generator = TerrainGenerator(terrain_cfg)
# terrain_importer = scene.add_terrain(terrain_generator)

# 4. 수역: 정적 강 메시 추가
# 이 메시는 순전히 시각적. 물리적 상호작용은 사용자 정의 스크립트 필요
river_mesh_usd_path = "omniverse://nucleus/Projects/MySim/Assets/river_surface.usd"
world.scene.add(XformPrim(prim_path="/World/River", name="river_surface_ref", usd_path=river_mesh_usd_path))

# 5. 숲: Replicator를 사용하여 초목 분산 배치
# 분산 배치할 나무 에셋 목록 정의
tree_assets = ["omniverse://.../tree_pine_01.usd", "omniverse://.../tree_oak_02.usd"]

# rep.scatter_2d를 사용하여 지형 프리미티브 위에 나무 배치
with rep.new_layer():
    # terrain_prim = rep.get.prim_at_path(terrain_importer.prim_path) # 지형 프리미티브 경로
    surface_prims = rep.get.prims(path_pattern="/World/Terrain") # 예시 경로
    trees = rep.create.from_usd(tree_assets, count=500)
    with trees:
        rep.randomizer.scatter_2d(
            surface_prims=surface_prims,
            check_for_collisions=True
        )
        rep.modify.pose(
            rotation=rep.distribution.uniform((0,0,0), (0,0,360))
        )

# 6. 시뮬레이션: 로봇 추가 및 시뮬레이션 실행
#... 장면에 로봇 추가...
world.reset()
while simulation_app.is_running():
    world.step(render=True)
    
simulation_app.close()
```


- Isaac Sim은 로보틱스 시뮬레이션을 위한 매우 강력하고 확장 가능한 플랫폼이지만, 범용적인 "세계 구축" 게임 엔진은 아닙니다.
- 복잡한 자연 환경을 생성하려면 **모듈식, 다중 도구 접근 방식**이 필요합니다. 사용자는 지형을 위한 Isaac Lab, 에셋 분산을 위한 Replicator, 비표준 물리를 위한 사용자 정의 Python 스크립트, 도로 및 건물과 같은 구조화된 에셋을 위한 외부 DCC 도구 등 전체 생태계를 활용할 준비가 되어 있어야 합니다.
- 플랫폼의 기능은 **로봇 중심의 설계 철학**을 직접적으로 반영합니다. 가장 잘 개발된 기능은 로봇 AI의 훈련 및 테스트를 직접적으로 가능하게 하는 기능들입니다.
- 복잡한 자연 환경을 포함하는 모든 프로젝트에서는 **에셋 생성/소싱 및 사용자 정의 스크립트 개발**을 위한 시간과 자원을 예산에 책정하는 것이 중요합니다. 성공은 환경 자체를 즉시 사용 가능한 기능이 아닌 핵심 엔지니어링 작업으로 취급하는 데 달려 있습니다.


1. Isaac Sim - Robotics Simulation and Synthetic Data Generation - NVIDIA Developer, accessed July 11, 2025, https://developer.nvidia.com/isaac/sim
2. Environment Setup - Isaac Sim Documentation, accessed July 11, 2025, https://docs.robotsfan.com/isaacsim/4.2.0/gui_tutorials/tutorial_intro_environment_setup.html
3. NVIDIA Isaac Sim™ is an open-source application on NVIDIA Omniverse for developing, simulating, and testing AI-driven robots in realistic virtual environments. - GitHub, accessed July 11, 2025, https://github.com/isaac-sim/IsaacSim
4. Isaac Lab Ecosystem, accessed July 11, 2025, https://isaac-sim.github.io/IsaacLab/main/source/setup/ecosystem.html
5. Underwater basic environment - Isaac Sim - NVIDIA Developer Forums, accessed July 11, 2025, https://forums.developer.nvidia.com/t/underwater-basic-environment/262775
6. Welcome to Isaac Lab!, accessed July 11, 2025, https://isaac-sim.github.io/IsaacLab/
7. Enhancements and upgrades for the release: v0.2 / isaac-sim IsaacLab / Discussion #106, accessed July 11, 2025, https://github.com/isaac-sim/IsaacLab/discussions/106
8. isaaclab.envs - Isaac Lab Documentation, accessed July 11, 2025, https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.envs.html
9. Randomization Snippets - Isaac Sim Documentation, accessed July 11, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/replicator_tutorials/tutorial_replicator_isaac_randomizers.html
10. Python Environment - Isaac Sim Documentation, accessed July 11, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/python_scripting/manual_standalone_python.html
11. Python Environment Installation - Isaac Sim Documentation, accessed July 11, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/installation/install_python.html
12. isaac-sim/IsaacGymEnvs: Isaac Gym Reinforcement Learning Environments - GitHub, accessed July 11, 2025, https://github.com/isaac-sim/IsaacGymEnvs
13. Creating spline based roads, based on a real location. How can I do it? - Reddit, accessed July 11, 2025, https://www.reddit.com/r/gamedev/comments/1l9ulhx/creating_spline_based_roads_based_on_a_real/
14. isaaclab.terrains - Isaac Lab Documentation, accessed July 11, 2025, https://isaac-sim.github.io/IsaacLab/main/source/api/lab/isaaclab.terrains.html
15. [Question] Generating 3D Terrain Maps from Depth Images / isaac-sim IsaacLab / Discussion #2689 - GitHub, accessed July 11, 2025, https://github.com/isaac-sim/IsaacLab/discussions/2689
16. README.md - isaac-sim/OmniIsaacGymEnvs - GitHub, accessed July 11, 2025, https://github.com/isaac-sim/OmniIsaacGymEnvs/blob/main/README.md
17. [isaacsim.asset.importer.heightmap] Height Map extension for Isaac Sim Occupancy Map, accessed July 11, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/py/source/extensions/isaacsim.asset.importer.heightmap/docs/index.html
18. Importing Height Map from Instant Terra - Documentation - Unigine Developer, accessed July 11, 2025, https://developer.unigine.com/en/docs/latest/objects/objects/terrain/landscape_terrain/heightmap_import/heightmap_instant_terra
19. Procedural Content Generation- PCG | Unreal Engine 5 #unrealengine - YouTube, accessed July 11, 2025, https://www.youtube.com/watch?v=I-IoIOCxdRg
20. NVidia Omniverse took over my Computer : r/reinforcementlearning - Reddit, accessed July 11, 2025, https://www.reddit.com/r/reinforcementlearning/comments/1ddkw1g/nvidia_omniverse_took_over_my_computer/
21. Scene Based Synthetic Dataset Generation - Isaac Sim Documentation, accessed July 11, 2025, https://docs.robotsfan.com/isaacsim/4.2.0/replicator_tutorials/tutorial_replicator_scene_based_sdg.html
22. Replicator Core usd files data - NVIDIA Developer Forums, accessed July 11, 2025, https://forums.developer.nvidia.com/t/replicator-core-usd-files-data/305790
23. Scene Based Synthetic Dataset Generation - Isaac Sim Documentation, accessed July 11, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/replicator_tutorials/tutorial_replicator_scene_based_sdg.html
24. Using scatter_2d in Isaac Sim Script Editor - NVIDIA Developer Forums, accessed July 11, 2025, https://forums.developer.nvidia.com/t/using-scatter-2d-in-isaac-sim-script-editor/265631
25. Importing a New Asset - Isaac Lab Documentation, accessed July 11, 2025, https://isaac-sim.github.io/IsaacLab/main/source/how-to/import_new_asset.html
26. Procedural vegetation and nature - Synthetic Data Generation (SDG), accessed July 11, 2025, https://forums.developer.nvidia.com/t/procedural-vegetation-and-nature/272348
27. I'm a Blender user that would like to start using Houdini. Can anyone tell me if they work well together and what kind of projects theyve done using the both of them? - Reddit, accessed July 11, 2025, https://www.reddit.com/r/Houdini/comments/xdcek8/im_a_blender_user_that_would_like_to_start_using/
28. Isaac Sim: Generating Synthetic Data using Replicator Composer - YouTube, accessed July 11, 2025, https://www.youtube.com/watch?v=HHzNIh72B_Y
29. Getting Started Scripts - Isaac Sim Documentation - NVIDIA, accessed July 11, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/replicator_tutorials/tutorial_replicator_getting_started.html
30. Create water with dynamics for underwater robots - Isaac Sim - NVIDIA Developer Forums, accessed July 11, 2025, https://forums.developer.nvidia.com/t/create-water-with-dynamics-for-underwater-robots/266545
31. How to build an ocean environment for marine robots in issac sim - Isaac Sim, accessed July 11, 2025, https://forums.developer.nvidia.com/t/how-to-build-an-ocean-environment-for-marine-robots-in-issac-sim/280374
32. Isaac Sim Deformable Body and Water Physics Simulation - YouTube, accessed July 11, 2025, https://www.youtube.com/watch?v=GHEXrn03cy8
33. Seeking Python Implementation for Fluid Simulation Demo in Isaac Sim 2022.3.3, accessed July 11, 2025, https://forums.developer.nvidia.com/t/seeking-python-implementation-for-fluid-simulation-demo-in-isaac-sim-2022-3-3/324292
34. NIVIDA Omniverse: Realtime fluid simulation with NVIDIA Flow - YouTube, accessed July 11, 2025, https://www.youtube.com/watch?v=m37XVn7N6MQ
35. Isaac Sim with PhysiX Flow - NVIDIA Developer Forums, accessed July 11, 2025, https://forums.developer.nvidia.com/t/isaac-sim-with-physix-flow/219156
36. leonlime/isaac_underwater: Water and underwater tests ... - GitHub, accessed July 11, 2025, https://github.com/leonlime/isaac_underwater
37. OceanSim: A GPU-Accelerated Underwater Robot Perception Simulation Framework - arXiv, accessed July 11, 2025, https://arxiv.org/html/2503.01074v1
38. accessed January 1, 1970, [https.arxiv.org/html/2503.01074v1](http://docs.google.com/https.arxiv.org/html/2503.01074v1)
39. Finding Junctions in Spline-based Road Generation - DiVA portal, accessed July 11, 2025, https://www.diva-portal.org/smash/get/diva2:1675311/FULLTEXT02
40. I created a procedural road network integrated with vehicle AI for obstacle avoidance, using A* for pathfinding. Try the packaged project via this link and let me know if you'd like a tutorial. : r/UnrealEngine5 - Reddit, accessed July 11, 2025, https://www.reddit.com/r/UnrealEngine5/comments/1fcrw4x/i_created_a_procedural_road_network_integrated/
41. Trajectory Generation - Isaac Sim Documentation, accessed July 11, 2025, https://docs.isaacsim.omniverse.nvidia.com/4.5.0/manipulators/concepts/trajectory_interface.html
42. Optimizing Robot Route Planning Using cuOpt for Isaac Sim to Improve ROI - FS Studio, accessed July 11, 2025, https://fsstudio.com/optimizing-robot-route-planning-using-cuopt-for-isaac-sim-to-improve-roi/
43. Using Isaac Sim with ROS2: A Step-by-Step Guide - YouTube, accessed July 11, 2025, https://www.youtube.com/watch?v=L1rpxRm0Q1w&pp=0gcJCfwAo7VqN5tD
44. Environment Assets - Isaac Sim Documentation, accessed July 11, 2025, https://docs.isaacsim.omniverse.nvidia.com/latest/assets/usd_assets_environments.html


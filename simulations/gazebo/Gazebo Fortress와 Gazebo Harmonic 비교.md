# Gazebo Fortress와 Gazebo Harmonic 비교

## **I. 서론: Fortress와 Harmonic, 두 LTS 릴리즈의 맥락**

Gazebo 시뮬레이터는 로보틱스 연구 및 개발에 있어 필수적인 도구로 자리 잡았다. 그중에서도 장기 지원(Long-Term Support, LTS) 릴리즈는 안정성과 신뢰성이 중요한 프로젝트의 기반이 된다. Gazebo Fortress와 Gazebo Harmonic은 이러한 LTS 릴리즈 중에서도 중요한 분기점을 대표하는 두 버전이다. 이 보고서는 두 버전의 릴리즈 배경, 핵심 변화, 그리고 기술적 차이점을 심층적으로 분석하여 사용자가 자신의 프로젝트에 가장 적합한 버전을 선택할 수 있도록 돕는 것을 목표로 한다.

### 0.1 A. 릴리즈 타임라인과 지원 기간(LTS) 비교

Gazebo Fortress는 2021년 9월 말에서 10월 초에 걸쳐 릴리즈된 6번째 메이저 릴리즈이자, Citadel에 이은 두 번째 LTS 버전이다.1 Fortress는 2026년 9월까지 공식 지원이 예정되어 있어, 출시 당시부터 장기 프로젝트를 위한 안정적인 선택지로 주목받았다.3

반면, Gazebo Harmonic은 그로부터 2년 뒤인 2023년 9월 말에서 10월 초에 릴리즈된 8번째 메이저 릴리즈다.6 Harmonic 역시 LTS 버전으로, 2028년 9월까지 지원된다.4

이 두 버전은 모두 안정적인 개발 환경을 제공하는 LTS 릴리즈라는 공통점을 가지지만, Harmonic은 2년 더 최신 기술 스택을 반영하고 있으며 지원 기간 또한 2년 더 길다. 이는 장기적인 프로젝트 관점에서 기술 부채(technical debt)를 줄이고 최신 기술 동향을 따라가는 데 있어 Harmonic이 갖는 명백한 이점이다.

### 0.2 B. 핵심 변화: 'Ignition'에서 'Gazebo'로의 리브랜딩

두 버전을 구분하는 가장 표면적이면서도 중요한 변화는 바로 '이름'이다. 2022년 4월, 개발사인 Open Robotics는 기존에 사용하던 "Ignition"이라는 브랜드 이름에 상표권 문제가 발생함에 따라, 커뮤니티에 친숙한 "Gazebo"라는 이름으로 회귀한다고 발표했다.12

이 결정은 단순한 명칭 변경을 넘어, 라이브러리, 실행 파일, 환경 변수 등 소프트웨어 전반에 영향을 미쳤다.13

- **Gazebo Fortress:** "Ignition Fortress"라는 이름으로 릴리즈되었으며, 모든 라이브러리와 명령어는 ign- 또는 ignition이라는 접두사를 사용한다.3
- **Gazebo Harmonic:** "Gazebo Harmonic"으로 불리며, 모든 구성 요소는 gz- 또는 gazebo 접두사를 사용한다.6

이러한 변화로 인해, Gazebo 11 버전까지를 "Gazebo Classic"으로, Ignition으로 시작된 새로운 아키텍처 기반의 버전을 "Modern Gazebo"로 구분하여 부르게 되었다. Fortress와 Harmonic 모두 Modern Gazebo에 속한다.12

### **C. 핵심 차이점 요약표**

아래 표는 Fortress와 Harmonic의 주요 차이점을 요약하여 보고서의 전체적인 내용을 한눈에 파악할 수 있도록 돕는다.

| 항목                  | Gazebo Fortress                              | Gazebo Harmonic                                              | 주요 차이점 및 시사점                                        |
| --------------------- | -------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **릴리즈 시점**       | 2021년 9월 3                                 | 2023년 10월 7                                                | Harmonic은 2년 더 최신 기술을 반영한다.                      |
| **지원 종료(EOL)**    | 2026년 9월 4                                 | 2028년 9월 4                                                 | 장기 프로젝트의 경우 Harmonic이 더 유리하다.                 |
| **브랜딩/명명법**     | Ignition, ign- 14                            | Gazebo, gz- 13                                               | 마이그레이션 시 코드 전반의 이름 변경이 필요하다.            |
| **핵심 아키텍처**     | 모듈형 라이브러리, ECS, C++ 플러그인 중심 14 | Fortress 아키텍처 계승 및 확장, **Python 플러그인 지원** 7   | Harmonic은 개발 언어의 장벽을 낮춰 접근성을 크게 향상시켰다. |
| **주요 신기능**       | PBR 렌더링 기반 마련, DART 물리 엔진 22      | 고급 렌더링(GI, Lens Flare), 고급 물리(자동 관성 계산, 유체 역학), 신규 센서(DVL, Airspeed), SDF-MJCF 변환기 7 | 시뮬레이션의 현실성과 활용 범위를 비약적으로 확장했다.       |
| **성능**              | 안정적이나 기능 정체 25                      | 성능 최적화를 위한 다양한 도구 제공 (Bullet-Featherstone, Levels 등) 26 | 초기 부하가 클 수 있으나, 최적화 잠재력은 Harmonic이 더 높다. |
| **공식 ROS 2 페어링** | ROS 2 Humble 28                              | ROS 2 Jazzy 28                                               | ROS 2 버전에 따라 권장되는 Gazebo 버전이 명확히 구분된다.    |
| **마이그레이션**      | -                                            | Fortress에서 Harmonic으로의 마이그레이션은 주로 이름 변경 작업 13 | API의 파괴적 변경이 적어 마이그레이션 난이도가 비교적 낮다.  |

### **D. 보고서의 결론 선요약: 어떤 사용자가 어떤 버전을 선택해야 하는가**

이 보고서의 상세 분석을 통해 도출될 결론을 미리 요약하면 다음과 같다.

- **Gazebo Fortress 추천 대상:** ROS 2 Humble 기반의 기존 프로젝트를 안정적으로 유지보수해야 하거나, 바이너리 패키지 의존성이 매우 높아 소스 컴파일을 피하고 싶은 사용자에게 적합하다.
- **Gazebo Harmonic 추천 대상:** 신규 프로젝트를 시작하거나, Python 기반 개발 환경을 선호하며, 최신 물리/렌더링 기능을 적극적으로 활용하고 싶고, ROS 2 Jazzy로의 전환을 계획 중인 사용자에게 강력히 권장된다. 장기적인 관점에서 기술 지원과 커뮤니티 활성도를 고려하는 모든 사용자에게 사실상 표준 선택지다.

## 1.  아키텍처 비교: 이름 너머의 변화

Gazebo Fortress와 Harmonic의 차이를 이해하기 위해서는 먼저 두 버전이 공유하는 견고한 아키텍처 기반을 파악하는 것이 중요하다. 두 버전의 관계는 '단절'이 아닌 '진화'에 가깝다. Fortress가 마련한 혁신적인 아키텍처의 토대 위에서 Harmonic이 어떻게 더 성숙하고 강력하게 발전했는지를 살펴보는 것은 두 버전의 본질적인 차이를 이해하는 핵심이다.

### 1.1 A. 공통 기반: 모듈형 라이브러리와 Entity-Component-System (ECS)

Fortress와 Harmonic이 속한 'Modern Gazebo'는 이전의 'Gazebo Classic'과는 근본적으로 다른 아키텍처 철학을 공유한다. Gazebo Classic이 상대적으로 하나의 큰 덩어리로 구성된 모놀리식(monolithic) 구조에 가까웠다면 29, Modern Gazebo는 물리, 렌더링, 센서, GUI, 통신 등 핵심 기능들이 각각 독립적인 라이브러리로 분리된 모듈형 구조를 채택했다.21

이러한 모듈형 구조의 중심에는 다음과 같은 두 가지 핵심 설계가 자리 잡고 있다.

1. **클라이언트-서버 구조:** 시뮬레이션의 물리 계산과 로직을 처리하는 백엔드 서버(gzserver)와, 이를 시각화하고 사용자와 상호작용하는 프론트엔드 GUI 클라이언트(gzclient)가 명확히 분리되어 있다.21 이 구조 덕분에 GUI 없이 서버만 독립적으로 실행하는 헤드리스(headless) 시뮬레이션이 용이하며, 분산 시뮬레이션의 기반이 된다.
2. **Entity-Component-System (ECS):** 백엔드 서버는 시뮬레이션 세계의 모든 요소를 'Entity'(개체), 'Component'(데이터/속성), 'System'(로직/행동)으로 나누어 관리하는 ECS 아키텍처를 기반으로 동작한다.21 예를 들어, '로봇 팔'이라는 Entity는 '관절 각도'라는 Component를 가지고 있고, '물리 엔진'이라는 System이 이 Component 값을 읽어 로봇 팔을 움직인다. 이 설계는 데이터와 로직을 분리하여 극도의 유연성과 확장성을 제공하며, 플러그인 기반 개발의 핵심 철학이다.

Fortress와 Harmonic 모두 이 동일한 아키텍처 패러다임을 공유한다. 따라서 Fortress 사용자는 이미 Harmonic의 핵심 구조에 익숙하며, 이는 Gazebo Classic에서 Modern Gazebo로 넘어올 때 겪었던 급진적인 변화와는 달리 마이그레이션의 기술적, 심리적 장벽을 크게 낮추는 요인으로 작용한다. Harmonic은 Fortress가 제시한 아키텍처의 완성형에 가깝다고 볼 수 있다.

### 1.2 B. 핵심 라이브러리 스택 비교: ign-\* (Fortress) vs gz-\* (Harmonic)

앞서 언급한 리브랜딩으로 인해, Fortress에서 ign- 접두사를 사용하던 모든 라이브러리는 Harmonic에서 gz- 접두사로 변경되었다.13 이는 마이그레이션 시 개발자가 직접 수정해야 하는 CMakeLists.txt나 소스 코드의 find_package, include 구문과 직접적으로 관련되므로, 아래 표를 통해 주요 라이브러리의 명칭 변경을 숙지하는 것이 중요하다. Gazebo의 각 릴리즈는 특정 버전의 라이브러리 집합으로 구성되며, 각 라이브러리는 시맨틱 버저닝(semantic versioning)을 따른다.4

| Fortress 라이브러리 (ign-) | Harmonic 라이브러리 (gz-) | 주 역할                                  |
| -------------------------- | ------------------------- | ---------------------------------------- |
| ign-cmake                  | gz-cmake                  | CMake 빌드 시스템 지원 유틸리티          |
| ign-common                 | gz-common                 | 파일 시스템, 로깅 등 공통 유틸리티       |
| ign-fuel-tools             | gz-fuel-tools             | 모델 다운로드를 위한 Fuel 서버 통신      |
| ign-gazebo                 | gz-sim                    | 시뮬레이션 서버 및 시스템 플러그인       |
| ign-gui                    | gz-gui                    | GUI 위젯 및 플러그인 기반                |
| ign-launch                 | gz-launch                 | 시뮬레이션 실행 및 설정 관리             |
| ign-math                   | gz-math                   | 벡터, 행렬 등 수학 관련 자료구조 및 함수 |
| ign-msgs                   | gz-msgs                   | Protobuf 기반 메시지 정의                |
| ign-physics                | gz-physics                | 물리 엔진 추상화 계층                    |
| ign-rendering              | gz-rendering              | 렌더링 엔진 추상화 계층                  |
| ign-sensors                | gz-sensors                | 센서 모델 및 노이즈 모델                 |
| ign-transport              | gz-transport              | 비동기 메시지 통신 프레임워크            |
| sdformat                   | sdformat                  | 시뮬레이션 기술 포맷 (이름 변경 없음)    |

### 1.3 C. 물리 엔진(gz-physics): DART, 그리고 Harmonic의 Bullet-Featherstone 도입

두 버전 모두 gz-physics(Fortress에서는 ign-physics)라는 추상화 계층을 통해 다양한 물리 엔진을 플러그인 형태로 지원한다.33 이 덕분에 사용자는 SDF 파일이나 커맨드 라인 인자를 통해 시뮬레이션에 사용할 물리 엔진을 유연하게 선택할 수 있다.36

Fortress는 DART(Dynamic Animation and Robotics Toolkit)를 기본 물리 엔진으로 채택했다.37 DART는 정확성이 높은 것으로 알려져 있지만, 특정 시나리오(예: 다수의 접촉점)에서는 성능 이슈가 보고되기도 한다. 물론 Bullet이나 TPE(Trivial Physics Engine) 같은 다른 엔진도 플러그인으로 사용 가능했다.38

Harmonic 역시 DART를 기본으로 유지하면서, bullet-featherstone 플러그인에 대한 지원을 공식적으로 추가하고 강화했다.26 Bullet은 게임 물리 엔진으로 널리 알려져 속도에 강점이 있으며, Featherstone 알고리즘은 다관절 로봇과 같은 연쇄적인 시스템의 동역학 계산에 매우 효율적이다. 이는 시뮬레이션의 특성에 따라 '정확성' 중심의 DART와 '속도 및 다관절 효율성' 중심의 Bullet-Featherstone 중에서 사용자가 최적의 도구를 선택할 수 있게 되었음을 의미한다. 이는 단순한 엔진 추가를 넘어, 시뮬레이션의 목적에 맞게 성능과 정확도를 트레이드오프할 수 있는 고급 옵션을 제공하는 중요한 구조적 진화다.

### 1.4 D. 렌더링 엔진(gz-rendering): Ogre2 기반의 그래픽 파이프라인과 Harmonic의 기능 확장

물리 엔진과 마찬가지로, 두 버전 모두 gz-rendering(Fortress에서는 ign-rendering) 라이브러리를 통해 렌더링 엔진을 추상화한다. Ogre 1.x와 Ogre 2.x(Ogre-Next)를 모두 플러그인 형태로 지원하지만, 실질적인 주력 엔진은 Ogre 2.x다.39 Ogre 1.x는 레거시 지원의 성격이 강하며, Ogre 2.x는 PBR(Physically Based Rendering)과 같은 현대적인 그래픽 기술을 지원하여 훨씬 사실적인 렌더링 결과를 제공한다.22 다수의 객체가 존재하는 복잡한 환경에서는 Ogre 2.x가 성능 면에서도 우위를 보인다.25

Fortress는 Ogre 2.x를 도입하여 현대적 렌더링의 '기반'을 마련했다. 하지만 이러한 고급 기능을 사용자가 직접 활용하기는 다소 복잡했다. Harmonic은 여기서 한 걸음 더 나아갔다. Global Illumination, Lens Flare, Projector 등 Ogre 2.x의 강력한 기능들을 사용하기 쉬운 시스템 플러그인 형태로 제공함으로써, 렌더링 파이프라인의 잠재력을 일반 사용자도 쉽게 활용할 수 있도록 '제품화'했다.7

결론적으로, Fortress가 Ogre 2.x라는 강력한 '엔진'을 탑재했다면, Harmonic은 그 엔진으로 달릴 수 있는 '슈퍼카'를 만들어냈다고 비유할 수 있다. 이는 시뮬레이션의 시각적 충실도를 한 차원 높은 수준으로 끌어올리는 중요한 발전이다.

## 2.  기능 심층 분석: Harmonic의 결정적 진화

Gazebo Harmonic은 Fortress에 비해 단순히 몇 가지 기능이 추가된 것을 넘어, 개발자의 생산성, 시뮬레이션의 현실성, 그리고 생태계 확장성 측면에서 결정적인 진화를 이루었다. 이 섹션에서는 Harmonic의 핵심 신기능들을 심층적으로 분석하고, 이러한 변화가 로보틱스 개발 워크플로우에 미치는 영향을 탐구한다.

### 2.1 A. 개발자 경험(DX)의 혁신: Python 플러그인 지원

Harmonic에서 가장 주목할 만한 변화는 단연 Python 플러그인 지원이다.7 Fortress에서는 모든 시스템 플러그인을 C++로 작성하고 컴파일해야만 했다.45 이는 C++에 익숙하지 않은 연구자나 개발자에게 높은 진입 장벽으로 작용했다.

Harmonic은 gz-msgs, gz-transport, gz-sim 시스템에 대한 Python 바인딩을 추가함으로써 이 문제를 해결했다.7 이제 개발자들은 C++의 복잡한 빌드 과정 없이 순수 Python 코드로 시뮬레이션의 로직을 제어하는 시스템 플러그인을 작성할 수 있다. 예를 들어,

gz-transport의 Python API를 사용하면 몇 줄의 코드로 간단하게 통신 노드를 생성하고 토픽을 발행(publish)하거나 구독(subscribe)하는 것이 가능하다.46

이 변화는 단순한 편의성 개선을 넘어선 전략적 의미를 갖는다. 전통적인 로보틱스 개발은 C++ 중심이었지만, 최근 인공지능(AI) 및 머신러닝(ML) 연구는 Python이 압도적인 표준 언어로 자리 잡았다. 강화학습, 컴퓨터 비전, 데이터 분석 등 다양한 분야의 연구자들이 시뮬레이션을 활용하고자 할 때, C++ 학습은 큰 부담이었다. Harmonic의 Python 지원은 이 장벽을 허물어, AI/ML 커뮤니티가 Gazebo 생태계로 유입되는 것을 가속화하고, Python 기반의 빠른 프로토타이핑과 실험을 가능하게 한다. 이는 Gazebo의 사용자 기반을 확장하고, 최첨단 AI 연구에서 Gazebo의 채택을 촉진하려는 중요한 전략적 결정으로 해석할 수 있다.

### 2.2 B. 시뮬레이션 현실성 강화: 신규 센서 및 물리 효과

Harmonic은 시뮬레이션의 물리적 현실성을 높이기 위한 다양한 기능을 추가했다.

- **자동 관성 모멘트 계산:** Fortress에서는 사용자가 모델의 관성 텐서(I)를 직접 계산하거나 근사값을 입력해야 했다. 이는 특히 복잡한 형상의 모델에서 매우 번거롭고 오류가 발생하기 쉬운 작업이었다. Harmonic에서는 SDF 파일에 <inertial><auto>true</auto> 태그 하나만 추가하면, 시뮬레이터가 메쉬(mesh) 형상을 기반으로 관성 모멘트를 자동으로 계산해준다.7 이는 모델링 시간을 단축시키고 물리 시뮬레이션의 정확도를 높이는 실용적인 개선이다.
- **Mimic Joint:** Gazebo Classic의 gearbox 조인트 기능을 개선한 것으로, 두 개 이상의 조인트 움직임을 선형적으로 연동시킨다. 예를 들어, 하나의 액추에이터로 여러 개의 손가락을 동시에 움직이는 로봇 그리퍼나, 자동차의 4절 링크 서스펜션과 같은 복잡한 메커니즘을 더 쉽고 직관적으로 모델링할 수 있게 되었다.7
- **유체 역학(Hydrodynamics) 강화:** 해양 로보틱스 커뮤니티의 요구를 반영하여 유체 역학 시뮬레이션 기능이 대폭 강화되었다. 선박이 움직일 때 주변 물을 밀어내며 발생하는 추가적인 저항인 '부가 질량(added mass)' 효과와, CSV 파일로 정의된 해류(ocean currents) 데이터를 시뮬레이션에 적용하는 기능이 추가되었다.7 이는 Fortress에서는 지원되지 않던 기능으로, 수중 로봇의 동역학을 훨씬 더 현실적으로 시뮬레이션할 수 있게 해준다.
- **신규 센서:**

- **Airspeed Sensor:** 항공기나 드론 시뮬레이션에 필수적인 공기 속도 센서가 추가되었다.7 SDF 파일에서
  type="air_speed"로 간단히 정의할 수 있으며 49, PX4와 같은 비행 제어 소프트웨어와의 연동을 위한 커뮤니티의 활발한 논의가 진행되었다.51
- **Doppler Velocity Log (DVL):** 수중 로봇이 음파를 이용해 해저면 대비 상대 속도를 측정하는 DVL 센서가 추가되었다.7 SDF 명세를 통해 음파 빔의 배열, 추적 모드, 노이즈 모델 등 실제 센서와 유사한 상세한 파라미터 설정이 가능하여 54, 고도로 전문적인 해양 시뮬레이션을 지원한다.

이러한 신규 물리 및 센서 기능들은 Gazebo가 범용 시뮬레이터를 넘어, 항공 및 해양과 같은 특정 전문 분야까지 포괄하려는 확장 의지를 명확히 보여준다. 이는 해당 분야의 개발자들에게 Fortress에서 Harmonic으로의 업그레이드를 필수적인 선택으로 만드는 요인이다.

### 2.3 C. 시각적 충실도(Visual Fidelity)의 극대화: 고급 렌더링 기법

Harmonic은 물리적 정확성을 넘어 시각적 현실감을 극대화하는 데에도 큰 발전을 이루었다. 이는 특히 시뮬레이션 환경에서 생성된 이미지 데이터를 학습에 사용하는 Sim-to-Real 연구 분야에서 매우 중요하다.

- **Global Illumination (GI):** 광원으로부터의 직접광뿐만 아니라, 물체 표면에서 반사된 간접광까지 계산하여 그림자와 색상을 훨씬 자연스럽고 사실적으로 표현한다. VCT(Voxel Cone Tracing)와 CIVCT 두 가지 방식을 지원하며, GUI 플러그인을 통해 쉽게 활성화하고 렌더링 품질과 성능 간의 트레이드오프를 조절할 수 있다.7 이는 Fortress에서는 기본적으로 제공되지 않던 고급 기능이다.
- **Lens Flare 및 Projector:** 강한 광원 주변에 생기는 렌즈 플레어 효과를 시뮬레이션하여 카메라 이미지의 현실감을 높이고 7, 장면에 특정 텍스처를 투사하는 프로젝터 기능도 추가되었다.7
- **카메라 기능 강화:** 실제 산업용 카메라와의 격차를 줄이기 위해 넓은 화각의 렌즈(wide-angle lens), 사용자가 직접 정의할 수 있는 카메라 내부 파라미터(intrinsic matrix), 산업용 카메라에서 흔히 사용되는 Bayer 형식 이미지 출력, 그리고 16비트 색심도 등을 지원한다.7

이러한 렌더링 기능의 강화는 '물리적으로 정확한 시뮬레이션'에서 '시각적으로 신뢰할 수 있는 시뮬레이션'으로의 패러다임 전환을 의미한다. Fortress도 물리적으로는 정확했지만, Harmonic은 비전 기반 자율주행, 객체 인식, 강화학습 등을 위한 고품질 합성 데이터(synthetic data) 생성 플랫폼으로서의 가치를 비약적으로 높였다.

### 2.4 D. 상호운용성: SDF ↔ MJCF 변환기의 의미와 활용

Harmonic 시대의 또 다른 중요한 성과는 DeepMind와의 협력을 통해 개발된 sdformat-mjcf 라이브러리의 릴리즈다.23 이 라이브러리는 Gazebo의 표준 포맷인 SDF(Simulation Description Format)와, 강화학습 연구 커뮤니티에서 널리 사용되는 MuJoCo 시뮬레이터의 MJCF(MuJoCo XML Format) 간의 상호 변환을 지원한다.23

이 기능은 Fortress에는 없던 것으로, 두 개의 거대한 시뮬레이션 생태계 사이에 다리를 놓는 역할을 한다. Gazebo 사용자는 방대한 MuJoCo의 로봇 모델 자산을 손쉽게 가져와 Gazebo의 풍부한 센서와 렌더링 환경에서 테스트할 수 있게 된다. 반대로 MuJoCo 사용자는 자신의 모델을 Gazebo 환경으로 가져와 시뮬레이션할 수 있다.

이는 단순한 파일 변환 유틸리티를 넘어, Gazebo가 고립된 생태계에 머무르지 않고 더 넓은 로보틱스 및 AI 연구 커뮤니티의 표준 도구로 자리매김하려는 전략적 방향성을 보여준다. Harmonic 사용자는 이 상호운용성을 통해 더 큰 규모의 개방형 생태계가 제공하는 이점을 누릴 수 있다.

## 3.  성능(Performance) 비교 분석 및 최적화 전략

시뮬레이션의 성능, 특히 실제 시간 대비 시뮬레이션 시간의 비율을 나타내는 RTF(Real-Time Factor)는 사용자가 가장 민감하게 체감하는 부분이다. Fortress와 Harmonic 모두 복잡한 환경에서는 성능 저하 문제가 발생할 수 있지만 25, Harmonic은 성능을 관리하고 최적화할 수 있는 더 다양한 도구를 제공한다는 점에서 차이가 있다.

### 3.1 A. RTF(Real-Time Factor)에 영향을 미치는 공통 요인 분석

두 버전 모두에서 RTF에 영향을 미치는 주요 요인은 다음과 같다.

- **충돌 지오메트리(Collision Geometry):** 시뮬레이션 성능 저하의 가장 큰 원인 중 하나는 복잡한 3D 메쉬(mesh)를 충돌 감지에 그대로 사용하는 것이다. 물리 엔진은 메쉬의 모든 면(face)에 대해 충돌을 계산해야 하므로 연산량이 기하급수적으로 증가한다. 이를 해결하기 위해 충돌용으로는 박스, 구, 실린더와 같은 단순한 기하학적 형상(primitive shapes)을 사용하거나, 메쉬의 폴리곤 수를 줄이는(downscaling) 작업이 강력히 권장된다.58
- **객체 및 조인트 수:** 시뮬레이션 월드 내에 존재하는 모델과 조인트의 수가 많아질수록 물리 계산량이 증가하여 RTF가 감소한다.60 불필요한 모델이나 고정 조인트(fixed joint)를 줄이고 여러 링크를 하나로 통합하는 것이 도움이 될 수 있다.
- **렌더링 및 센서 설정:** GUI에서 그림자(shadows) 효과를 비활성화하거나 광원의 수를 줄이는 것은 렌더링 부하를 크게 줄일 수 있다.58 또한, 카메라의 해상도 및 FPS, LiDAR의 업데이트 속도와 샘플 수를 낮추는 것도 RTF 향상에 효과적이다.58
- **물리 엔진 파라미터:** 물리 엔진의 시간 스텝 크기(max_step_size)를 늘리면 한 번의 계산으로 더 긴 시간을 시뮬레이션하므로 RTF는 향상되지만, 정확도가 떨어지고 시뮬레이션이 불안정해질 수 있다.59

### 3.2 B. Fortress 대비 Harmonic의 성능 개선점

Harmonic은 Fortress에 비해 성능 문제를 해결하기 위한 구체적인 기능들을 추가했다.

- **물리 엔진 최적화:** Harmonic에서 지원이 강화된 bullet-featherstone 물리 엔진은 특정 다관절 로봇 시나리오에서 DART보다 빠른 성능을 보일 수 있다. 또한, 일정 시간 움직임이 없는 객체의 물리 계산을 자동으로 중지하는 '자동 비활성화(auto-deactivation)' 기능이 추가되어 불필요한 연산을 줄이고 RTF를 향상시킨다.26
- **렌더링 센서 효율화:** Harmonic에서는 카메라, LiDAR와 같이 렌더링 파이프라인을 사용하는 센서들의 데이터 처리 효율이 개선되어, 센서로 인한 성능 부하가 감소했다.26
- **대규모 환경 지원:** 거대한 월드를 시뮬레이션할 때 발생하는 성능 저하 문제를 해결하기 위해, Harmonic은 '레벨(Levels)' 기능을 도입했다. 이 기능은 월드를 여러 구역으로 분할하여 현재 로봇이 위치한 구역과 그 주변만 동적으로 로드하는 방식이다.27 또한, 공간 정보를 활용해 시뮬레이션 자산을 동적으로 로드하고 언로드하는 기능도 지원한다.22 이는 Fortress의 빌딩 에디터 기능이 사라진 대신 65, 대규모 환경을 보다 효율적으로 처리하는 방향으로 발전했음을 보여준다.

이러한 개선점들은 Harmonic이 단순히 Fortress보다 '빠르다' 또는 '느리다'로 평가될 수 없음을 시사한다. 오히려 Harmonic은 사용자에게 성능 문제를 진단하고, 시뮬레이션의 특성에 맞게 최적화할 수 있는 더 많은 도구와 선택지를 제공한다. 초기 설정이 더 복잡할 수 있지만, 최적화의 잠재력은 Fortress보다 크다고 할 수 있다.

### 3.3 C. 주요 성능 병목 현상 및 해결 방안

커뮤니티에서는 특정 환경에서 발생하는 공통적인 성능 문제와 그 해결책이 공유되고 있다.

- **가상 머신(VM) 및 Docker 환경:** VM이나 Docker 컨테이너 내부에서 Gazebo GUI를 실행할 때, 호스트 머신의 GPU 가속이 제대로 전달되지 않으면 모든 렌더링을 CPU가 처리하게 되어 CPU 사용량이 급증하고 RTF가 심각하게 저하된다.57 이 문제를 해결하기 위해 nvidia-docker 설정, X11 포워딩, --render-engine ogre 옵션을 통한 Ogre1 엔진 사용, 또는 export OGRE_RTT_MODE=Copy나 export LIBGL_ALWAYS_SOFTWARE=1과 같은 환경 변수 설정이 해결책으로 제시된다.66
- **고해상도 충돌 메쉬:** baylands 월드와 같이 시각적으로 화려하지만 충돌 감지용 메쉬가 매우 복잡한 모델은 RTF를 5% 미만으로 극도로 저하시킬 수 있다.56 이 경우, 모델 제작자가 충돌 감지용 저해상도 메쉬를 별도로 제공하거나 사용자가 직접 생성하여 교체하는 것이 유일한 해결책이다.
- **Global Illumination (GI) 부하:** Harmonic의 GI 기능은 매우 사실적인 그래픽을 제공하지만, GPU에 상당한 계산 부하를 준다. 성능이 우선시되는 상황에서는 GI를 비활성화하거나, GUI 플러그인을 통해 해상도, 빛 반사 횟수 등의 파라미터를 낮추어 성능과 품질 사이의 균형을 맞춰야 한다.55

## 4.  ROS 2 연동: 최적의 조합 찾기

로보틱스 개발에서 Gazebo는 ROS(Robot Operating System)와 함께 사용되는 경우가 대부분이다. 따라서 ROS 2와의 호환성 및 연동 방식은 Gazebo 버전을 선택하는 데 있어 매우 중요한 기준이다.

### 4.1 A. 공식 및 비공식 ROS 2 버전 페어링 가이드

각 ROS 2 배포판은 특정 버전의 Gazebo와 가장 안정적으로 연동되도록 공식 지원 패키지를 제공한다.

- **공식 페어링:**

- **ROS 2 Humble Hawksbill (LTS):** Gazebo **Fortress**와 공식적으로 연동된다. Ubuntu 22.04 환경에서 sudo apt install ros-humble-ros-gz 명령어를 실행하면 Fortress와 연동되는 브릿지 패키지가 설치된다.28
- **ROS 2 Jazzy Jalisco (LTS):** Gazebo **Harmonic**과 공식적으로 연동된다. Ubuntu 24.04 환경에서 ros-jazzy-ros-gz 패키지를 통해 설치되며, 이는 최신 LTS ROS 2와 최신 LTS Gazebo의 조합으로 가장 권장되는 구성이다.28

- **비공식 조합:** 공식 페어링 외의 조합도 가능하지만, 추가적인 설정이나 소스 빌드가 필요하다.

- **ROS 2 Humble + Gazebo Harmonic:** 이 조합은 많은 사용자들이 원하는 구성이다. Humble의 안정성을 유지하면서 Harmonic의 최신 기능을 사용하기 위함이다. OSRF(Open Robotics)에서 제공하는 비공식 바이너리 패키지(ros-humble-ros-gzharmonic)를 설치하거나, ros_gz 패키지를 소스에서 직접 빌드하여 사용할 수 있다.28
- **ROS 2 Iron Irwini:** Gazebo Fortress와 공식 바이너리 지원이 있지만, Harmonic이나 Garden을 사용하려면 소스 빌드가 필요하다.74

아래 표는 ROS 2 버전별 권장 Gazebo 구성 및 연동 방식을 정리한 것이다.

| ROS 2 배포판     | 권장 Gazebo 버전   | 연동 패키지               | 설치 방법                        | 고려사항                                                     |
| ---------------- | ------------------ | ------------------------- | -------------------------------- | ------------------------------------------------------------ |
| **Humble (LTS)** | **Fortress (LTS)** | ros-humble-ros-gz         | 공식 apt 패키지                  | **가장 안정적이고 검증된 조합.** 별도의 설정 없이 바로 사용 가능. |
|                  | Harmonic (LTS)     | ros-humble-ros-gzharmonic | 비공식 apt 패키지 또는 소스 빌드 | Humble 환경에서 Harmonic의 최신 기능 사용 가능. 패키지 충돌 가능성 유의. |
| **Iron**         | Fortress (LTS)     | ros-iron-ros-gz           | 공식 apt 패키지                  | 공식 지원되나, Iron과 Fortress 모두 최신 LTS가 아님.         |
| **Jazzy (LTS)**  | **Harmonic (LTS)** | ros-jazzy-ros-gz          | 공식 apt 패키지                  | **신규 프로젝트에 가장 권장되는 표준 조합.**                 |
| **Rolling**      | Ionic              | ros-rolling-ros-gz        | 공식 apt 패키지                  | 최신 기능을 테스트하기 위한 개발자용 조합. 안정성은 보장되지 않음. |

### 4.2 B. ros_gz_bridge: 버전별 지원 메시지 및 설정 차이

ros_gz_bridge는 ROS 2의 메시지와 Gazebo의 Transport 메시지 간의 통신을 중계하는 핵심 패키지다.15 사용자는 커맨드라인에서

parameter_bridge를 실행하거나, YAML 설정 파일을 통해 여러 토픽에 대한 브릿지를 한 번에 설정할 수 있다.15

Fortress와 Harmonic 간에는 브릿지 설정에 있어 명칭 변경으로 인한 차이가 존재한다.

- **메시지 타입:** Fortress에서는 ignition.msgs.StringMsg와 같이 ignition.msgs 네임스페이스를 사용했지만, Harmonic에서는 gz.msgs.StringMsg와 같이 gz.msgs를 사용한다.15
- **방향 지정:** 브릿지 방향을 YAML 파일에서 지정할 때, Fortress의 IGN_TO_ROS는 Harmonic에서 GZ_TO_ROS로 변경되었다.15
- **지원 메시지 확장:** Harmonic으로 버전이 올라가면서 브릿지가 지원하는 메시지 타입이 확장되었다. 예를 들어, PoseWithCovarianceStamped, Joy 등 더 많은 표준 ROS 메시지들이 브릿지 목록에 추가되었다.32

### 4.3 C. 사용 사례별 권장 구성

- **신규 프로젝트 (ROS 2 Jazzy 기반):** 고민할 필요 없이 **Gazebo Harmonic**을 선택하는 것이 최선이다. 최신 기능, 가장 긴 LTS 지원 기간, 공식 패키지를 통한 간편한 설치 등 모든 면에서 가장 유리하다.28
- **기존 프로젝트 (ROS 2 Humble 기반):**

- **안정성 최우선:** 현재 사용 중인 **Gazebo Fortress**를 그대로 유지하는 것이 좋다. 가장 오랫동안 검증된 조합으로, 예기치 않은 호환성 문제를 피할 수 있다.
- **최신 기능 필요:** **Gazebo Harmonic**으로의 업그레이드를 고려해야 한다. 이 경우, ros_gz를 소스에서 빌드하거나 비공식 패키지를 설치해야 하는 추가 작업이 발생하지만, Python 플러그인이나 고급 렌더링과 같은 Harmonic의 강력한 기능들을 활용할 수 있는 유일한 방법이다.73

## 5.  마이그레이션 가이드: Fortress에서 Harmonic으로

Gazebo Fortress 기반의 프로젝트를 Harmonic으로 이전하는 작업은 'Ignition'에서 'Gazebo'로의 리브랜딩에 따른 이름 변경이 대부분을 차지한다. API의 구조적인 변경보다는 명칭 변경에 가깝기 때문에, 작업량은 많을 수 있으나 로직을 재설계해야 하는 부담은 거의 없다. 이는 Fortress 사용자가 Harmonic으로의 전환을 긍정적으로 고려하게 만드는 중요한 요인이다.

### 5.1 A. 단계별 마이그레이션 절차: CMake, 소스 코드, SDF 파일 수정

마이그레이션의 핵심 원칙은 프로젝트 내의 모든 Ignition 또는 ign 관련 문자열을 해당하는 Gazebo 또는 gz로 변경하는 것이다.13

- **1단계: CMakeLists.txt 수정**

- 패키지 찾기: find_package(ignition-math6)와 같은 구문을 find_package(gz-math7)로 변경한다.
- CMake 함수: ign_find_package()는 gz_find_package()로 변경한다.13
- 타겟 링크 라이브러리: target_link_libraries(my_target ignition-gazebo6::core)는 target_link_libraries(my_target gz-sim8::core)와 같이 라이브러리 이름과 버전을 맞춰 수정한다.

- **2단계: 소스 코드(.cpp,.h,.py) 수정**

- 헤더 포함: #include <ignition/gazebo/System.hh>는 #include <gz/sim/System.hh>로 변경한다.13
- 네임스페이스: ignition::gazebo::는 gz::sim::으로, ignition::math::는 gz::math:: 등으로 변경한다.13
- 로그 매크로: ignerr, ignwarn, igndbg 등은 각각 gzerr, gzwarn, gzdbg로 변경한다.13

- **3단계: SDF 파일(.sdf,.world) 수정**

- 플러그인 경로: <plugin filename="ignition-gazebo-physics-system"...>는 <plugin filename="gz-sim-physics-system"...>으로 수정한다.36
- 플러그인 이름: <plugin name="ignition::gazebo::systems::Physics"...>는 <plugin name="gz::sim::systems::Physics"...>로 네임스페이스를 수정한다.36

- **4. 단계: 환경 변수 및 스크립트 수정**

- 환경 변수: 셸 스크립트나 .bashrc 파일에서 사용되는 IGN_GAZEBO_RESOURCE_PATH는 GZ_SIM_RESOURCE_PATH로, IGN_GAZEBO_PHYSICS_ENGINE_PATH는 GZ_SIM_PHYSICS_ENGINE_PATH로 변경한다.13
- 커맨드 라인 명령어: 셸 스크립트에서 ign gazebo를 사용했다면 gz sim으로 변경한다.78

이러한 변경 작업은 sed나 grep과 같은 커맨드라인 도구나, IDE의 전역 찾기/바꾸기 기능을 활용하면 효율적으로 수행할 수 있다.

### 5.2 B. 주요 API 변경 사항 및 주의점

앞서 강조했듯이, Fortress에서 Harmonic으로의 마이그레이션은 API의 '파괴적 변경'이 아닌 '이름 변경'에 가깝다. 함수의 시그니처나 클래스의 핵심 로직은 대부분 유지된 채, 리브랜딩으로 인한 이름만 바뀌었다. 따라서 마이그레이션 작업은 비교적 위험도가 낮고 예측 가능한 범위 내에서 이루어진다. 컴파일러가 출력하는 경고나 오류 메시지가 대부분의 경우 무엇을 어떻게 바꿔야 할지 알려주는 좋은 가이드가 될 것이다.13

### 5.3 C. 일반적인 마이그레이션 함정과 해결책

- **깨끗한 빌드(Clean Build)의 중요성:** 마이그레이션 후에는 반드시 기존의 build, install, log 디렉토리를 완전히 삭제하고 처음부터 새로 빌드해야 한다. 이전 ign- 라이브러리의 캐시된 정보와 충돌하여 예기치 않은 오류가 발생하는 것을 방지하기 위함이다.13
- **사용자 설정 파일:** GUI 레이아웃이나 플러그인 설정 등은 사용자의 홈 디렉토리 아래 .gz (또는 과거의 .ignition) 폴더에 저장된다. 이 설정 파일들은 소프트웨어 업데이트 시 자동으로 마이그레이션되지 않을 수 있다. 만약 GUI가 비정상적으로 보이거나 로드되지 않는다면, 이 설정 폴더를 수동으로 삭제하거나 내부 파일의 경로를 수정한 후 Gazebo를 다시 실행하는 것이 좋다.13
- **Gazebo Classic 마이그레이션과의 혼동:** Fortress에서 Harmonic으로의 마이그레이션은 Gazebo Classic(예: Gazebo 11)에서 Modern Gazebo로 이전하는 것과는 완전히 다른 차원의 작업이다. Classic에서 Modern으로의 이전은 플러그인 API와 전체 아키텍처가 근본적으로 다르기 때문에 훨씬 더 복잡하고 많은 재작성이 필요하다.79 관련 문서를 참고할 때 이 두 가지 마이그레이션을 혼동하지 않도록 각별히 주의해야 한다.

## 6.  결론: 프로젝트를 위한 최적의 선택

지금까지 Gazebo Fortress와 Harmonic의 아키텍처, 기능, 성능, ROS 2 연동, 그리고 마이그레이션 측면을 다각도로 분석했다. 이를 바탕으로, 어떤 사용자가 어떤 버전을 선택해야 하는지에 대한 명확한 결론과 가이드라인을 제시한다.

### 6.1 A. Fortress와 Harmonic의 장단점 종합 비교

- **Gazebo Fortress:**

- **장점:** ROS 2 Humble과의 공식 연동을 통해 검증된 안정성을 제공하며, 수년간 사용되어 온 만큼 성숙한 생태계와 풍부한 레퍼런스를 보유하고 있다. 기존 사용자에게는 진입 장벽이 낮다.
- **단점:** 2021년 릴리즈 이후 기능적으로 정체되어 있으며, Python 플러그인이나 고급 렌더링과 같은 최신 기능이 부재하다. 2026년 9월 지원이 종료되므로 장기적인 관점에서는 불리하다.4

- **Gazebo Harmonic:**

- **장점:** Python 플러그인 지원, Global Illumination과 같은 고급 렌더링, 유체 역학을 포함한 정교한 물리 효과, 신규 센서 모델 등 압도적인 기능 확장을 제공한다. 개발자 편의성이 크게 향상되었으며, 2028년 9월까지 지원되는 긴 LTS 기간과 활발한 커뮤니티 개발이 강점이다.4
- **단점:** ROS 2 Humble과 공식적으로 연동되지 않아 비공식 패키지 설치나 소스 빌드와 같은 추가 작업이 필요할 수 있다.73 새로 추가된 기능들로 인해 초기 학습 곡선이 Fortress보다 다소 높을 수 있다.

### 6.2 B. 프로젝트 요구사항 기반의 버전 선택 가이드라인

- **"내 프로젝트가 ROS 2 Humble에 강하게 종속되어 있고, 당장 업그레이드할 계획이 없다면?"**

- **답변:** **Gazebo Fortress**를 유지하는 것이 가장 안전한 선택이다. 하지만 만약 Python 플러그인과 같은 Harmonic의 특정 기능이 반드시 필요하다면, 비공식 패키지나 소스 빌드를 통해 Harmonic과 연동하는 방법을 신중하게 검토해볼 수 있다.

- **"새로운 로봇 프로젝트를 시작한다면?"**

- **답변:** 주저 없이 **Gazebo Harmonic**과 **ROS 2 Jazzy** 조합을 선택해야 한다. 이것이 현재와 미래의 표준이며, 최신 기능, 가장 긴 지원 기간, 가장 활발한 커뮤니티 지원을 받을 수 있는 최선의 길이다.28

- **"AI/ML 연구, 특히 강화학습이나 비전 기반 연구를 수행한다면?"**

- **답변:** **Gazebo Harmonic**이 사실상 유일한 선택지다. Python 플러그인을 통한 빠른 프로토타이핑, 시각적으로 신뢰할 수 있는 합성 데이터 생성을 위한 고급 렌더링 기능, 그리고 MuJoCo와의 상호운용성은 AI 연구에 필수적인 요소들이다.7

- **"해양 또는 항공 로봇을 시뮬레이션한다면?"**

- **답변:** DVL, Airspeed 센서, 유체 역학 등 전문 분야를 위한 물리 및 센서 모델을 지원하는 **Gazebo Harmonic**이 반드시 필요하다.7

### 6.3 C. 미래 전망 및 Harmonic으로의 전환 당위성

Gazebo Classic의 지원이 2025년 1월에 종료되는 것을 시작으로 18, 로보틱스 시뮬레이션 생태계는 Modern Gazebo로의 전환이라는 거스를 수 없는 흐름 위에 있다. Fortress 역시 2026년 9월이면 지원이 종료되므로 4, 장기적인 관점에서 모든 신규 개발과 기술 투자는 Harmonic 또는 그 이후 버전(Ionic, Jetty 등)을 중심으로 이루어질 것이다.

최종적으로, Fortress는 안정적인 '현재'를 위한 선택일 수 있지만, **Harmonic은 확장성과 미래를 위한 '투자'**다. Fortress에서 Harmonic으로의 마이그레이션 비용은 API의 파괴적 변경이 적어 상대적으로 낮다. 따라서 특별한 제약 조건이 없다면, 모든 사용자는 가능한 한 빨리 Harmonic으로의 전환을 계획하고 실행하는 것이 장기적인 프로젝트의 성공과 기술 경쟁력 확보를 위해 현명한 결정이 될 것이다.

#### **참고 자료**

1. Gazebo : Distributions - ignition-fortress, accessed July 25, 2025, https://classic.gazebosim.org/distributions/ignition-fortress/releases/
2. gazebosim/gz-fortress: Fortress, Gazebo's 6th named release - GitHub, accessed July 25, 2025, https://github.com/gazebosim/gz-fortress
3. Ignition Fortress Released - Open Robotics, accessed July 25, 2025, https://www.openrobotics.org/blog/2021/10/01/ignition-fortress-released
4. Gazebo Releases - Gazebo ionic documentation, accessed July 25, 2025, https://gazebosim.org/docs/latest/releases/
5. Search - Gazebo fortress documentation - 创客智造, accessed July 25, 2025, https://www.ncnynl.com/en/gazebosim/docs/fortress/search/
6. Gazebo : Distributions - gz-harmonic, accessed July 25, 2025, https://classic.gazebosim.org/distributions/gz-harmonic/releases/
7. Gazebo Harmonic Released! - Open Robotics, accessed July 25, 2025, https://www.openrobotics.org/blog/2023/9/26/gazebo-harmonic-released
8. SAVE THE DATE: Gazebo Harmonic Tutorial Party! - General - ROS Discourse, accessed July 25, 2025, https://discourse.ros.org/t/save-the-date-gazebo-harmonic-tutorial-party/33091
9. Gazebo Harmonic Release - General, accessed July 25, 2025, https://community.gazebosim.org/t/gazebo-harmonic-release/2311
10. Releases / gazebosim/gz-sim - GitHub, accessed July 25, 2025, https://github.com/ignitionrobotics/ign-gazebo/releases
11. Gazebo Development - Gazebo harmonic documentation - 创客智造, accessed July 25, 2025, https://www.ncnynl.com/en/gazebosim/docs/harmonic/development/
12. A new era for Gazebo (cross-post) - General - ROS Discourse, accessed July 25, 2025, https://discourse.ros.org/t/a-new-era-for-gazebo-cross-post/25012
13. Migration Guide - Gazebo ionic documentation, accessed July 25, 2025, https://gazebosim.org/docs/latest/migration_from_ignition/
14. Ignition Fortress - Gazebo fortress documentation, accessed July 25, 2025, https://gazebosim.org/docs/fortress/install/
15. ROS 2 Integration - Gazebo fortress documentation, accessed July 25, 2025, https://gazebosim.org/docs/fortress/ros2_integration/
16. Installing Gazebo Harmonic, accessed July 25, 2025, https://reifxsolutions.com/hjn-harmonic-setup/
17. A new era for Gazebo - ROS Discourse - Open Robotics, accessed July 25, 2025, https://discourse.openrobotics.org/t/a-new-era-for-gazebo/48995
18. Gazebo Classic and Citadel End of Life [x-post Gazebo Sim Community] - ROS Discourse, accessed July 25, 2025, https://discourse.ros.org/t/gazebo-classic-and-citadel-end-of-life-x-post-gazebo-sim-community/40931
19. Gazebo Classic End-of-Life - General, accessed July 25, 2025, https://community.gazebosim.org/t/gazebo-classic-end-of-life/2563
20. Gazebo Classic robotics simulator reaches end of life, accessed July 25, 2025, https://www.therobotreport.com/gazebo-classic-robotics-simulator-reaches-end-of-life/
21. Gazebo Sim Architecture - Gazebo ionic documentation, accessed July 25, 2025, https://gazebosim.org/docs/latest/architecture/
22. Features -- Gazebo, accessed July 25, 2025, https://staging.gazebosim.org/features
23. Gazebo - News - Open Robotics, accessed July 25, 2025, https://www.openrobotics.org/blog/tag/Gazebo
24. Open Robotics releases Gazebo Harmonic simulator - The Robot Report, accessed July 25, 2025, https://www.therobotreport.com/open-robotics-gazebo-harmonic-simulator/
25. Ignition Gazebo (Fortress) has a worse Real Time Factor (RTF) than Gazebo Classic (11), accessed July 25, 2025, https://robotics.stackexchange.com/questions/104985/ignition-gazebo-fortress-has-a-worse-real-time-factor-rtf-than-gazebo-classi
26. State of Gazebo 2024 - ROSCon 2025, accessed July 25, 2025, https://roscon.ros.org/2024/talks/The_State_of_Gazebo.pdf
27. Large model in gazebo slows RTF to 1% - Robotics Stack Exchange, accessed July 25, 2025, https://robotics.stackexchange.com/questions/113316/large-model-in-gazebo-slows-rtf-to-1
28. Installing Gazebo with ROS - Gazebo ionic documentation, accessed July 25, 2025, https://gazebosim.org/docs/latest/ros_installation/
29. Tutorial : Gazebo Architecture - Gazebo, accessed July 25, 2025, https://classic.gazebosim.org/tutorials?tut=architecture
30. Gazebo Tutorials - Gazebo ionic documentation, accessed July 25, 2025, https://gazebosim.org/docs/latest/tutorials/
31. Gazebo, accessed July 25, 2025, https://gazebosim.org/
32. gz-harmonic/release_notes.md at main - GitHub, accessed July 25, 2025, https://github.com/gazebosim/gz-harmonic/blob/main/release_notes.md
33. physics - Gazebo documentation, accessed July 25, 2025, https://gazebosim.org/libs/physics/
34. docs/harmonic/comparison.md at master / gazebosim/docs - GitHub, accessed July 25, 2025, https://github.com/gazebosim/docs/blob/master/harmonic/comparison.md
35. gazebosim/gz-physics: Abstract physics interface designed to support simulation and rapid development of robot applications. - GitHub, accessed July 25, 2025, https://github.com/gazebosim/gz-physics
36. Physics engines - Gazebo Sim, accessed July 25, 2025, https://gazebosim.org/api/sim/9/physics.html
37. Physics engines - Gazebo, accessed July 25, 2025, https://gazebosim.org/api/gazebo/6/physics.html
38. Switching physics engines - Gazebo, accessed July 25, 2025, https://gazebosim.org/api/physics/6/switchphysicsengines.html
39. rendering - Gazebo documentation, accessed July 25, 2025, https://gazebosim.org/libs/rendering/
40. gazebosim/gz-rendering: C++ library designed to provide an abstraction for different rendering engines. It offers unified APIs for creating 3D graphics applications. - GitHub, accessed July 25, 2025, https://github.com/gazebosim/gz-rendering
41. Feature comparison - Gazebo ionic documentation, accessed July 25, 2025, https://gazebosim.org/docs/latest/comparison/
42. Ogre 2 and Ogre 1 merge strategy - Ogre Forums, accessed July 25, 2025, https://forums.ogre3d.org/viewtopic.php?t=94758
43. Ogre v1 vs Ogre v2 - Ogre Forums, accessed July 25, 2025, https://forums.ogre3d.org/viewtopic.php?t=95495
44. Gazebo Release Features - Gazebo ionic documentation, accessed July 25, 2025, https://gazebosim.org/docs/latest/release-features/
45. Tutorial : Plugins 101 - Gazebo, accessed July 25, 2025, https://classic.gazebosim.org/tutorials?tut=plugins_hello_world
46. srmainwaring/gz-python: Python bindings for gz-msgs and gz-transport - GitHub, accessed July 25, 2025, https://github.com/srmainwaring/gz-python
47. Python Support - Gazebo Transport, accessed July 25, 2025, https://gazebosim.org/api/transport/13/python.html
48. transport - Gazebo documentation, accessed July 25, 2025, https://gazebosim.org/libs/transport/
49. sdf::SDF_VERSION_NAMESPACE::Sensor Class Reference - Gazebo, accessed July 25, 2025, https://gazebosim.org/api/sdformat/13/classsdf_1_1SDF__VERSION__NAMESPACE_1_1Sensor.html
50. 
51. Gazebo VTOL and fixed wing plane support / Issue #20836 / PX4/PX4-Autopilot - GitHub, accessed July 25, 2025, https://github.com/PX4/PX4-Autopilot/issues/20836
52. Trouble checking windspeeds set during gazebo simulation - PX4 Discussion Forum, accessed July 25, 2025, https://discuss.px4.io/t/trouble-checking-windspeeds-set-during-gazebo-simulation/25717
53. sensors - Gazebo documentation, accessed July 25, 2025, https://gazebosim.org/libs/sensors/
54. Gazebo Sensors: DopplerVelocityLog Class Reference, accessed July 25, 2025, https://gazebosim.org/api/sensors/9/classgz_1_1sensors_1_1DopplerVelocityLog.html
55. Global Illumination - Gazebo Sim, accessed July 25, 2025, https://gazebosim.org/api/sim/8/global_illumination.html
56. Gazebo Harmonic slow real time factor on baylands world - ArduPilot Discourse, accessed July 25, 2025, https://discuss.ardupilot.org/t/gazebo-harmonic-slow-real-time-factor-on-baylands-world/125284
57. RANT: Gazebo (new) is a terrible piece of software and I don't think we should recommend it to newbies : r/ROS - Reddit, accessed July 25, 2025, https://www.reddit.com/r/ROS/comments/1jqn8wb/rant_gazebo_new_is_a_terrible_piece_of_software/
58. Gazebo Simulator : 5 Ways to Speedup Simulations - Black Coffee Robotics, accessed July 25, 2025, https://www.blackcoffeerobotics.com/blog/gazebo-simulator-5-ways-to-speedup-simulations
59. how to speed up gazebo simulation? - ros - Robotics Stack Exchange, accessed July 25, 2025, https://robotics.stackexchange.com/questions/102469/how-to-speed-up-gazebo-simulation
60. Regarding Gazebo RTF becoming low after increasing joints in the simulation, accessed July 25, 2025, https://answers.gazebosim.org/question/27170/
61. Gazebo Harmonic slow real time factor on baylands world - Robotics Stack Exchange, accessed July 25, 2025, https://robotics.stackexchange.com/questions/113430/gazebo-harmonic-slow-real-time-factor-on-baylands-world
62. Regarding Gazebo RTF becoming low after increasing joints in the simulation, accessed July 25, 2025, http://answers.gazebosim.org/question/27171/
63. Recommendations to improve simulation performance in large building - Gazebo Help, accessed July 25, 2025, https://discourse.openrobotics.org/t/recommendations-to-improve-simulation-performance-in-large-building/48150
64. Recommendations to improve simulation performance in large building - ROS Discourse, accessed July 25, 2025, https://community.gazebosim.org/t/recommendations-to-improve-simulation-performance-in-large-building/2558
65. Alternatives to Gazebo Fortress (Building Editor) Feature? - Robotics Stack Exchange, accessed July 25, 2025, https://robotics.stackexchange.com/questions/104528/alternatives-to-gazebo-fortress-building-editor-feature
66. Very high CPU usage when running Gazebo in a docker container, accessed July 25, 2025, https://robotics.stackexchange.com/questions/107875/very-high-cpu-usage-when-running-gazebo-in-a-docker-container
67. Running Gazebo on a Virtual Machine : r/ROS - Reddit, accessed July 25, 2025, https://www.reddit.com/r/ROS/comments/1f1s3z9/running_gazebo_on_a_virtual_machine/
68. Black screen with 3d acceleration enabled for Gazebo and Rviz (Robotic) | Parallels Forums, accessed July 25, 2025, https://forum.parallels.com/threads/black-screen-with-3d-acceleration-enabled-for-gazebo-and-rviz-robotic.355825/
69. WSLg with GPU support available on latest version of Gazebo Garden and Harmonic!, accessed July 25, 2025, https://community.gazebosim.org/t/wslg-with-gpu-support-available-on-latest-version-of-gazebo-garden-and-harmonic/2360
70. What's up with Global Illumination? : r/pathofexile - Reddit, accessed July 25, 2025, https://www.reddit.com/r/pathofexile/comments/k7xi59/whats_up_with_global_illumination/
71. For ROS2 Beginners - Install ROS 2 and Gazebo on Ubuntu 22.04 Jammy (LTS) Step-by-Step Instruction | by Claudia Yao | Medium, accessed July 25, 2025, https://medium.com/@claudia.yao2012/for-ros2-beginners-install-ros-2-and-gazebo-on-ubuntu-22-04-jammy-lts-step-by-step-instruction-08327c3764a5
72. Installing the default Gazebo/ROS pairing - Robotics Stack Exchange, accessed July 25, 2025, https://robotics.stackexchange.com/questions/114012/installing-the-default-gazebo-ros-pairing
73. Will Gazebo harmonic ever be recommended for ROS2 Humble? - General, accessed July 25, 2025, https://community.gazebosim.org/t/will-gazebo-harmonic-ever-be-recommended-for-ros2-humble/2390
74. Gazebo Classic End-Of-Life & ROS 2 Jazzy - General, accessed July 25, 2025, https://discourse.ros.org/t/gazebo-classic-end-of-life-ros-2-jazzy/36239
75. ROS2 Humble, Gazebo Harmonic, PX4, QGroundControl, Micro XRCE-DDS Agent & Client Installation | by Erdem | Medium, accessed July 25, 2025, https://medium.com/@erdem.ku.3.14/ros2-humble-gazebo-harmonic-px4-ve-micro-xrce-dds-agent-client-installation-aad32d8f5669
76. Use ROS 2 to interact with Gazebo, accessed July 25, 2025, https://gazebosim.org/docs/latest/ros2_integration/
77. ros_gz_bridge: Rolling 3.0.3 documentation - the official ROS docs, accessed July 25, 2025, https://docs.ros.org/en/rolling/p/ros_gz_bridge/
78. Getting Started with Gazebo? - Gazebo ionic documentation, accessed July 25, 2025, https://gazebosim.org/docs/latest/getstarted/
79. Migrating Old Plugins from Gazebo Classic to Gazebo Harmonic, accessed July 25, 2025, https://community.gazebosim.org/t/migrating-old-plugins-from-gazebo-classic-to-gazebo-harmonic/3446
80. Gazebo Classic Migration, accessed July 25, 2025, https://gazebosim.org/docs/latest/gazebo_classic_migration/


# Gazebo

## 1. 장: Gazebo의 기원과 진화

이 장에서는 Gazebo의 역사적 맥락을 설정하고, 학술 프로젝트에서 시작하여 로봇 공학 분야의 필수 도구로 발전하기까지의 과정을 추적한다. 특히 그 궤적을 형성한 주요 이정표와 전략적 변화, 그리고 현대적 정체성을 정의하는 아키텍처 분기를 비판적으로 검토한다.

### 1.1  학술 프로젝트에서 산업 표준으로: 역사적 개요

Gazebo의 개발은 2002년 서던캘리포니아 대학교(USC)에서 앤드류 하워드(Andrew Howard)와 네이트 케이니그(Nate Koenig)에 의해 시작되었다.1 당시 지배적이었던 2D 시뮬레이터와 달리, 다양한 야외 환경에서 로봇을 시뮬레이션해야 할 필요성이 개발의 초기 동기가 되었다.1 이 초기 비전은 로봇이 통제된 실험실을 벗어나 복잡한 실제 환경으로 이동할 것이라는 미래를 예견한 것이었다.

Gazebo의 발전 과정은 로봇 공학 분야 자체의 성숙 과정을 반영한다. 2002년부터 2011년까지 Gazebo는 Player 프로젝트의 구성 요소로 기능했다.4 이후 2011년, ROS(Robot Operating System)를 포함한 로봇 소프트웨어의 핵심 인큐베이터였던 윌로우 개러지(Willow Garage)의 지원을 받으며 독립적인 프로젝트가 되었다.4 이 시기는 Gazebo가 급성장하는 ROS 커뮤니티의 사실상 물리 시뮬레이터로 자리매김하는 중요한 전환점이었다.

2012년에는 거버넌스가 오픈 소스 로보틱스 재단(OSRF, 현 Open Robotics)으로 이전되었다.2 이로써 Gazebo는 특정 기업의 단기 목표에서 벗어나 커뮤니티가 주도하는 진정한 오픈 소스 표준으로서의 역할을 공고히 했다. 이 여정은 전문화된 학술 도구에서 시작하여 더 넓은 프레임워크의 일부가 되고, 주요 로봇 연구소의 지원을 거쳐 최종적으로 커뮤니티 중심의 표준으로 자리 잡는, 로봇 소프트웨어 개발의 표준화 및 협업 증가라는 뚜렷한 경향을 보여준다.

### 1.2  거대한 분기: Gazebo Classic과 Ignition 포크의 근거와 영향

15년 이상 개발이 진행되면서, 현재 "Gazebo Classic"으로 알려진 기존 Gazebo의 코드베이스는 상당한 현대화가 필요한 모놀리식(monolithic) 아키텍처가 되었다.4 모놀리식 아키텍처는 초기에는 관리가 용이하지만 시간이 지남에 따라 유지보수와 확장이 어려워지는 병목 현상을 유발한다. 예를 들어, 렌더링 엔진만 사용하려는 개발자도 전체 Gazebo 생태계를 컴파일하고 링크해야 하는 비효율성이 존재했다.5

이러한 문제를 해결하기 위해 2017년, 개발은 두 갈래로 나뉘었다. 초기에 "Ignition"이라는 브랜드로 알려진 새 버전은 느슨하게 결합된 독립적인 라이브러리 모음으로 재설계되었다.4 이는 단순한 버전 업데이트가 아니라 시뮬레이터가 무엇이어야 하는지에 대한 근본적인 재해석이었다. 단일 코드베이스의 단순함을 포기하는 대신, 라이브러리 기반 생태계의 유연성, 유지보수성, 재사용성을 선택한 전략적 결정이었다.

2022년, "Ignition"이라는 이름의 상표권 문제로 인해 이름이 다시 변경되었다. 현대적인 모듈식 버전은 이제 "Gazebo"로, 기존의 모놀리식 버전은 "Gazebo Classic"으로 불린다.4 Gazebo Classic의 마지막 주요 릴리스는 11 버전이며, 수명 종료 시점까지 지원되지만 모든 새로운 기능 개발은 현대적인 Gazebo에 집중되고 있다.6

이러한 분리는 Gazebo의 유용성을 단순한 시뮬레이터를 넘어 다목적 로봇 소프트웨어 툴킷으로 확장시켰다. 개발자는 이제 `gz-math`, `gz-rendering`과 같은 구성 요소를 자신의 특정 응용 프로그램에 맞게 선택적으로 사용할 수 있게 되었다.5 그러나 이러한 변화는 기존 사용자들에게 익숙했던 패러다임을 바꾸고, 문서의 파편화를 유발하는 등 새로운 과제를 낳기도 했다.

### 1.3  시험의 장: 주요 로봇 경진대회에서의 Gazebo의 역할

Gazebo는 다수의 저명한 로봇 경진대회에서 공식 시뮬레이션 플랫폼으로 채택되며 그 성능과 신뢰성을 입증받았다. 이러한 대회들은 Gazebo의 기술적 성숙을 이끄는 강력한 원동력이 되었다.

- **DARPA 로보틱스 챌린지 (DRC, 2012-2015):** 가상 로보틱스 챌린지(VRC) 단계는 전적으로 Gazebo 내에서 진행되었으며, 참가팀들은 재난 환경에서 휴머노이드 로봇을 제어하는 임무를 수행했다.4
- **NASA 우주 로보틱스 챌린지 (SRC):** 결선 진출자들은 Gazebo에서 시뮬레이션된 발키리(Valkyrie) 휴머노이드를 프로그래밍하여 화성 기지에서 수리 임무를 완수해야 했다.4
- **DARPA 지하 챌린지 (SubT, 2018-2021):** 가상 트랙에서는 지하 환경을 매핑하고 탐색하기 위해 매우 복잡한 Gazebo 기반 시뮬레이션 환경이 사용되었다.4
- **기타 경진대회:** 산업 자동화를 위한 민첩 로봇 경진대회(ARIAC), DARPA 군사 아카데미 스웜 챌린지(SASC), 토요타 프리우스 챌린지, 유럽 로버 챌린지(ERC) 등도 시뮬레이션을 위해 Gazebo를 활용했다.3

수백만 달러 규모의 DARPA 챌린지 공식 플랫폼으로 선정된 것은 Gazebo가 현실 세계를 대리할 만큼 견고하고, 유연하며, 정확하다는 강력한 인증이었다.4 이족보행부터 드론 스웜, 지하 레이저 스캐닝에 이르기까지, 이러한 대회들의 까다로운 요구사항은 오늘날 Gazebo를 강력하게 만드는 다양한 물리 엔진, 고급 센서 모델 등의 핵심 기능 개발에 직접적인 동력을 제공했다. 이는 긍정적인 피드백 루프를 형성했다. 즉, 경진대회가 개발을 촉진하고, 그 결과 향상된 기능이 Gazebo를 미래의 경진대회를 위한 명백한 선택지로 만들었다.

## 2. 장: Gazebo의 아키텍처 핵심

이 장에서는 Gazebo의 핵심 구성 요소를 심층적으로 기술적으로 해부한다. SDF의 구조와 기능, 다양한 물리 엔진, 센서 모델링 기능, 그리고 중추적인 플러그인 아키텍처의 진화를 분석한다.

### 2.1  시뮬레이션 기술 형식(SDF): 가상 세계의 청사진

시뮬레이션 기술 형식(Simulation Description Format, SDF)은 로봇, 정적 및 동적 객체, 조명, 지형, 물리 등 시뮬레이션의 모든 요소를 기술하는 데 사용되는 XML 기반 형식이다.7 SDF의 역할은 단순한 파일 형식을 넘어, 로봇 시스템의 *설명*을 시뮬레이터의 *구현*으로부터 분리하는 보편적인 언어로서 기능한다.

초기에는 Gazebo의 일부였으나, 자체 라이브러리인 `libsdformat`으로 분리되어 독립적인 프로젝트가 되었다.6 이 전략적 분리 덕분에, SDF로 정의된 로봇 모델은 Gazebo뿐만 아니라 `libsdformat`을 사용하는 모든 도구에서 파싱하고 이해할 수 있다. 예를 들어, 모션 플래닝 도구는 전체 Gazebo 물리 시뮬레이션을 실행하지 않고도 로봇의 기구학을 이해하기 위해 `libsdformat`을 사용할 수 있다.8 이러한 관심사의 분리는 성숙한 소프트웨어 엔지니어링의 특징이며 SDF의 강력함의 핵심이다.

SDF 파일의 루트 요소는 `<sdf>`이며, 이는 `<world>`, `<model>`, `<actor>`, `<light>` 등의 자식 요소를 포함할 수 있다.9

`<model>`은 로봇이나 객체를 기술하며, 질량과 관성을 가진 물리적 본체인 `<link>`와 이들을 연결하는 `<joint>`로 구성된다. 각 링크는 외형을 정의하는 `<visual>`과 물리 엔진을 위한 충돌 형태를 정의하는 `<collision>` 속성을 가질 수 있다.11 또한, SDF는 플러그인이나 센서 매개변수와 같은 Gazebo 특정 태그를 포함하도록 확장될 수 있으며, `<gazebo>` 태그를 통해 URDF(Unified Robot Description Format) 파일을 확장하는 데도 사용된다.11

참고로, 전자 설계 분야에서 사용되는 동명의 "Standard Delay Format" 13은 본 보고서에서 다루는 로봇 시뮬레이션용 SDF와는 무관하므로 혼동을 피해야 한다.

### 2.2  물리 시뮬레이션: 현실 모방에 대한 심층 분석

Gazebo는 여러 물리 엔진을 플러그인 형태로 지원하여 사용자가 시뮬레이션 요구에 맞게 선택할 수 있도록 한다. 지원되는 엔진에는 ODE(Open Dynamics Engine), Bullet, Simbody, DART(Dynamic Animation and Robotics Toolkit)가 포함된다.4 Gazebo Classic은 ODE의 포크 버전을 기본으로 사용하는 반면 4, 현대적인 Gazebo는 DART를 기본 엔진으로 채택했다.18

여러 물리 엔진의 가용성은 단순히 선택의 폭을 넓히는 것을 넘어, 모든 로봇 공학 문제에 대한 단일 "최고의" 물리 엔진은 없다는 사실을 인정하는 것이다. 엔진 선택은 시뮬레이션 속도, 안정성, 정확성 사이의 근본적인 트레이드오프를 나타낸다. 엔진은 좌표계 표현 방식에 따라 크게 두 가지로 나눌 수 있다.19

- **최대 좌표계(Maximal Coordinates):** ODE와 Bullet이 사용하는 방식으로, 각 링크의 위치와 방향을 월드 좌표계에서 독립적으로 표현한다 (예: 링크당 6자유도). 조인트와 같은 제약 조건은 수치적으로 해결된다. 이 방식은 계산량이 많고, 제약 조건 오류가 누적되어 "조인트 분리"와 같은 불안정성을 초래할 수 있다.
- **일반화/최소 좌표계(Generalized/Minimal Coordinates):** DART와 Simbody가 사용하는 방식으로, 시스템의 실제 조인트 각도를 상태 변수로 사용하여 표현한다. 이 방식은 공식화하기는 더 복잡하지만, 방정식 시스템의 크기가 작고 휴머노이드와 같이 복잡하고 폐쇄 루프를 가진 다관절 시스템에서 본질적으로 더 안정적이다.

단순한 바퀴형 로봇은 ODE로도 효율적이고 정확하게 시뮬레이션할 수 있지만, 수십 개의 조인트로 구성된 휴머노이드 로봇은 최대 좌표계 시스템에서 불안정해지기 쉽다.20 일반화 좌표계를 사용하는 DART와 Simbody는 제약 조건을 시스템의 수학적 공식에 직접 내장함으로써 이러한 문제를 피한다.19 따라서 Gazebo Classic에서 현대 Gazebo로 넘어오면서 기본 엔진이 ODE에서 DART로 변경된 것은, 시뮬레이션 대상이 되는 로봇의 복잡성이 증가함에 따라 안정성과 정확성을 우선시하는 로봇 커뮤니티의 초점 변화를 직접적으로 반영한다.

SDF 파일 내에서는 `max_step_size`(시뮬레이션 시간 단계), `real_time_update_rate`(실시간 대비 시뮬레이션 속도 조절), 그리고 ODE의 `iters`(반복 횟수)나 `sor`(순차적 과이완)과 같은 엔진별 솔버 설정을 통해 물리 시뮬레이션을 제어할 수 있다.16

| **엔진** | **좌표계**    | **주요 강점**                                                | **주요 약점/한계**                                           | **대표적 사용 사례**            |
| -------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------- |
| ODE      | 최대 좌표계   | 빠르고 간단하며, 단순한 시스템에 적합                        | 복잡한 다관절 시스템에서 제약 조건 오류 및 불안정성 발생 가능 | 바퀴형 로봇, 간단한 조작기      |
| Bullet   | 최대 좌표계   | 빠른 충돌 감지, 게임 물리 분야에서 널리 사용됨               | ODE와 유사하게 복잡한 폐쇄 루프 시스템에서 안정성 문제 발생 가능 | 다수의 동적 객체 상호작용       |
| DART     | 일반화 좌표계 | 복잡한 다관절 및 폐쇄 루프 시스템에서 높은 안정성, 정확한 동역학 | 최대 좌표계 엔진보다 설정이 복잡하고 계산 비용이 높을 수 있음 | 휴머노이드 로봇, 복잡한 로봇 팔 |
| Simbody  | 일반화 좌표계 | 생체역학 및 복잡한 다물체 동역학에 최적화된 높은 정확도      | DART와 유사하게, 특정 유형의 문제에 대해 더 전문화됨         | 생체 모방 로봇, 유연체 동역학   |

표 2.1: Gazebo 물리 엔진 비교

### 2.3  센서 모델링: 로봇의 지각 인터페이스

현대 로봇 공학의 성공은 대부분 지각(perception) 능력에 달려 있으며, 따라서 센서 시뮬레이션의 충실도는 물리 시뮬레이션의 충실도보다 종종 더 중요하다. Gazebo는 이러한 요구를 충족시키기 위해 광범위한 센서 모델링 기능을 제공한다. 시뮬레이션 가능한 센서에는 2D/3D 카메라, 광각 카메라, 레이저 거리 측정기(LiDAR), 키넥트 스타일 센서, 관성 측정 장치(IMU), 접촉 센서, 힘-토크 센서, GPS, 소나 등이 포함된다.4

센서는 SDF 파일 내에서 모델의 특정 `<link>`에 `<sensor>` 태그를 사용하여 추가된다.11 Gazebo를 단순한 물리 시뮬레이터가 아닌 진정한 *로봇 공학* 시뮬레이터로 만드는 핵심 기능은 센서 데이터에 현실적인 노이즈를 추가하는 능력이다. SDF의 `<noise>` 태그를 사용하면 일반적으로 가우시안(Gaussian) 분포를 따르는 노이즈 모델을 지정할 수 있다.23

가우시안 모델의 경우 `<mean>`(평균) 및 `<stddev>`(표준편차)와 같은 매개변수를 설정할 수 있다.23 특히 IMU와 같은 센서의 경우,  `bias_mean`(바이어스 평균), `dynamic_bias_stddev`(동적 바이어스 표준편차), `dynamic_bias_correlation_time`(동적 바이어스 상관 시간) 등 시간에 따라 변하는 드리프트 현상까지 모델링할 수 있는 더 복잡한 노이즈 매개변수가 제공된다.23

이상적인 센서 출력뿐만 아니라 노이즈, 바이어스, 제한된 범위와 같은 실제 세계의 불완전성을 시뮬레이션하는 능력은, 알고리즘이 하드웨어에 배포되기 전에 이러한 불완전성에 대해 강건하게 만들어질 수 있도록 한다. 이는 "심투리얼(sim-to-real)" 패러다임의 핵심이다. 완벽하고 노이즈 없는 LiDAR 데이터로 개발된 알고리즘은 실제 로봇에서 즉시 실패할 것이다. Gazebo의 상세한 노이즈 모델은 개발자가 이러한 현실적인 제약 조건 하에서도 안정적으로 작동하는 알고리즘을 만들 수 있게 해준다. 공식 튜토리얼은 센서 추가 및 노이즈 설정에 대한 단계별 가이드를 제공한다.24

### 2.4  플러그인 아키텍처: 확장성을 위한 프레임워크

플러그인은 사용자가 직접 작성하여 시뮬레이션에 삽입할 수 있는 공유 라이브러리로, Gazebo의 거의 모든 측면을 프로그래밍 방식으로 제어할 수 있게 해주는 강력한 확장 기능이다.7 Gazebo의 역사에 걸쳐 플러그인 아키텍처는 중요한 진화를 겪었으며, 이는 시뮬레이터를 확장하는 방식에 대한 패러다임 전환을 반영한다.

**Gazebo Classic**에서는 `World`, `Model`, `Sensor`, `System`, `Visual`, `GUI`의 6가지 고유한 플러그인 유형이 있었다.28 각 플러그인은 특정 요소(예: 월드, 모델)에 연결되어 해당 API의 특정 부분에만 접근할 수 있었다. 이 방식은 직관적이지만, 예를 들어 센서 판독값이 시각적 속성을 직접 변경하는 동작을 구현하려면 센서 플러그인과 비주얼 플러그인 간의 복잡한 조정이 필요했다.

**현대 Gazebo(구 Ignition)**는 엔티티 컴포넌트 시스템(Entity Component System, ECS) 아키텍처로 전환했다. 여기서 플러그인은 일반적으로 엔티티와 그들의 컴포넌트에 대해 작동하는 "시스템(System)"이다.29 이 접근 방식은 훨씬 더 유연하고 강력하다. 단일 시스템 플러그인이 물리, 시각, 센서 등 모든 유형의 컴포넌트에 접근하고 수정할 수 있기 때문이다.29 이는 "시뮬레이터를 확장하는 것"에서 "컴포넌트로 시뮬레이션을 구축하는 것"으로의 패러다임 전환을 의미한다.

이러한 아키텍처 변화는 ROS 통합 방식에도 근본적인 변화를 가져왔다. Classic에서는 `gazebo_ros_plugins`가 ROS API를 직접 사용하는 플러그인을 제공했다. 반면, 현대 Gazebo에서는 Gazebo Transport와 ROS 2 토픽 간의 메시지를 변환하는 `ros_gz_bridge`가 주요 통합 방법이다.30 이 브리지 방식은 핵심 시뮬레이터를 ROS에 구애받지 않게 만들어 장기적인 건전성과 비(非)ROS 커뮤니티에서의 채택에 유리하지만, 새로운 추상화 계층을 도입하고 고대역폭 데이터에 대한 잠재적 성능 문제를 야기하는 트레이드오프가 존재한다.30

| 표 2.2: Gazebo 플러그인 아키텍처: Classic 대 Modern (ECS) |                                      |                                         |                                                              |
| --------------------------------------------------------- | ------------------------------------ | --------------------------------------- | ------------------------------------------------------------ |
| **특징**                                                  | **Gazebo Classic**                   | **현대 Gazebo (Ignition)**              | **의미 및 트레이드오프**                                     |
| **핵심 철학**                                             | 특정 유형의 객체를 확장              | 엔티티와 컴포넌트에 작동하는 시스템     | 더 데이터 중심적이고 유연한 접근 방식으로 전환               |
| **플러그인 유형**                                         | 6가지 고정 유형 (World, Model 등)    | 주로 단일 'System' 유형                 | 경직된 분류에서 벗어나 기능적 유연성 증대                    |
| **시뮬레이션 상태 접근**                                  | 플러그인 유형에 따라 API 접근 제한   | 단일 시스템에서 모든 컴포넌트 접근 가능 | 복합적인 상호작용 구현이 용이해짐                            |
| **ROS 통합 방식**                                         | `gazebo_ros_plugins` (직접 API 호출) | `ros_gz_bridge` (메시지 브리징)         | 시뮬레이터와 ROS의 분리(decoupling) 강화, 잠재적 성능 오버헤드 발생 |
| **유연성**                                                | 제한적                               | 매우 높음                               | 개발자에게 더 큰 권한을 부여하지만, 복잡성 증가              |
| **학습 곡선**                                             | 상대적으로 낮음                      | 더 가파름                               | 새로운 ECS 패러다임과 브리지 개념에 대한 학습 필요           |

## 3. 장: 실제 사용 환경에서의 Gazebo: 생태계와 사용자 경험

이 장에서는 내부 아키텍처에서 벗어나 Gazebo를 사용하는 외부 경험으로 초점을 전환하여, ROS와의 중요한 관계, 사용 가능한 학습 리소스, 그리고 사용자가 직면하는 일반적인 실제 문제들을 집중적으로 다룬다.

### 3.1  로봇 운영 체제(ROS)와의 공생 관계

Gazebo와 ROS의 성공은 서로 불가분하게 연결되어 있다. Gazebo는 ROS에 강력한 오픈 소스 시뮬레이션 환경을 제공했고, ROS는 Gazebo에 방대하고 활동적인 사용자 기반과 표준화된 통신 프레임워크를 제공했다.31 이 강력한 피드백 루프는 두 기술 모두를 산업 표준으로 확립시켰다.

Gazebo가 ROS를 위한 최고의, 가장 깊이 통합된 시뮬레이터라는 점은 그 인기의 핵심적인 이유이다.34 사용자는 실제 로봇에서 사용할 것과 동일한 토픽을 구독하고 발행하는 ROS 노드를 사용하여 Gazebo라는 가상의 "실제 세계"에서 알고리즘을 테스트할 수 있다.3 이 "심투리얼(sim-to-real)" 워크플로우는 현대 로봇 공학 개발의 초석이며, 개발자는 시뮬레이션에서 실제 로봇으로 전환할 때 코드 변경을 최소화할 수 있다.

앞서 언급했듯이, ROS 2와의 통합 방식은 직접적인 플러그인(`gazebo_ros_pkgs`)에서 전송 브리지(`ros_gz_bridge`)로 진화했다.30 이 변화는 두 주요 프로젝트의 독립적인 발전을 위한 필수적인 단계였지만, 기존 패러다임에 익숙한 사용자에게는 잠재적인 마찰 지점이 될 수 있다. 공생 관계는 계속되지만, 그 기술적 성격은 근본적으로 바뀌었으며 사용자는 이에 적응해야 한다.

### 3.2  학습 곡선과 커뮤니티 지원 인프라

Gazebo는 그 강력함만큼이나 가파른 학습 곡선을 가지고 있으며, 특히 초보자에게는 그 방대한 기능이 부담스러울 수 있다.34 이러한 어려움은 Gazebo Classic과 현대 Gazebo(구 Ignition)로의 분기로 인해 더욱 심화되었다. 이로 인해 온라인에서 찾을 수 있는 많은 정보와 튜토리얼이 오래되었거나 어떤 버전에 해당하는지 불분명하여 문서 환경이 파편화되었다.32

공식적으로는 Gazebo Classic을 위한 초급, 중급, 고급 튜토리얼 26과 현대 Gazebo를 위한 기본, ROS 통합, 라이브러리별 튜토리얼 27이 모두 제공된다. 하지만 사용자가 "Gazebo 튜토리얼"을 검색하면 Gazebo 7, Gazebo 11, 현대 Gazebo 등 서로 다른 API와 통합 방식을 가진 버전에 대한 자료가 혼재되어 나타날 수 있다. 한 커뮤니티 토론에서는 Webots가 더 세련된 UI와 체계적인 문서 덕분에 초보자에게 더 친숙하다고 평가하기도 했다.32

커뮤니티 지원 인프라 역시 변화를 겪었다. 공식 Q&A 포럼이었던 Gazebo Answers는 보존 자료(archive)로 전환되었고 36, 커뮤니티는 새로운 질문을 위해 로보틱스 스택 익스체인지(Robotics Stack Exchange)를 사용하도록 안내받고 있다.37 스택 익스체인지는 강력한 플랫폼이지만, 이는 과거 포럼에 수년간 축적된 맥락 특정적인 지식이 이제 정적인 아카이브에 머물러 검색하고 상호작용하기 더 어려워졌음을 의미한다. 이는 Gazebo 생태계가 직면한 중요한 비기술적 과제이다.

### 3.3  일반적인 문제 해결: 실무자를 위한 가이드

Gazebo 사용자가 겪는 문제들은 종종 그 복잡성과 유연성에서 비롯되는 부수적 속성이다. Gazebo를 강력하게 만드는 바로 그 기능들-상세한 물리 엔진, 유연한 플러그인 시스템-이 가장 흔한 문제의 원인이 되기도 한다. 이러한 문제를 해결하려면 단순한 튜토리얼을 따르는 것을 넘어 깊이 있는 시스템 수준의 이해가 필요하다.

- **설치 및 의존성:** 설치 과정은 복잡할 수 있으며, 특히 비(非)우분투 플랫폼이나 소스에서 컴파일할 때 더욱 그렇다. 사용자들은 의존성, 그래픽 드라이버, 컴파일러 호환성(예: NVCC)과 관련된 문제를 보고한다.2
- **시뮬레이션 불안정성:** 모델이 흔들리거나 "폭발하는" 등 불안정한 동작을 보일 수 있다. 이는 종종 경직된(stiff) 미분 방정식, 부정확한 관성 속성, 또는 큰 물리 시간 단계와 결합된 높은 컨트롤러 게인 때문에 발생한다.20 이는 단순한 "Gazebo 버그"라기보다는 수치 시뮬레이션의 근본적인 문제에 가깝다.
- **성능:** 복잡한 장면, 많은 모델, 또는 고주파 센서가 있을 경우 성능(실시간 계수, Real Time Factor)이 크게 저하될 수 있다. 중요하지 않은 모델을 `<static>`으로 표시하거나, `<collision>` 태그를 제거하고, 객체를 동적으로 생성/삭제하는 등의 전략이 사용된다.41
- **다중 로봇 시뮬레이션:** 한 컴퓨터에서 여러 개의 독립적인 Gazebo 인스턴스를 실행하는 것은 `/gazebo`와 같은 기본 토픽 이름의 충돌로 인해 악명이 높다. 이를 해결하기 위해 종종 `multimaster_fkie` 패키지와 같은 해결 방법이 필요하다.42 이는 Gazebo가 초기에 이러한 사용 사례를 주요 목표로 설계하지 않았기 때문에 발생하는 문제이다.

이러한 사례들은 Gazebo 전문가가 되는 과정이 복잡하고 상호 연결된 시스템에서 숙련된 탐정이 되는 과정과 같다는 것을 보여준다.

## 4. 장: 비판적 분석: 강점, 약점 및 경쟁 환경

이 장에서는 Gazebo의 역량에 대한 균형 잡힌 비판적 평가를 제공하고, 다른 주요 시뮬레이션 도구와의 맥락 속에서 그 위치를 파악하여 사용자가 정보에 입각한 결정을 내릴 수 있도록 돕는다.

### 4.1  내성적 고찰: Gazebo의 본질적인 장점과 한계

Gazebo의 강점과 약점은 동전의 양면과 같으며, 이는 게임 엔진이나 범용 다중 물리 플랫폼이 아닌 *로봇 공학 연구 도구*라는 핵심 설계 철학에서 비롯된다.

**장점:**

- **오픈 소스 및 커뮤니티:** 활기차고 활동적인 커뮤니티와 함께 무료로 제공된다.6
- **ROS 통합:** 동급 최고의 원활한 ROS 통합은 Gazebo의 가장 큰 특징이다.31
- **물리 유연성:** 여러 고성능 물리 엔진을 지원하여 사용자가 특정 요구에 맞게 시뮬레이션을 조정할 수 있다.6
- **확장성:** 플러그인 시스템은 시뮬레이션의 모든 측면을 제어할 수 있는 깊이 있는 프로그래밍 방식의 접근을 제공한다.6
- **센서 충실도:** 구성 가능한 노이즈 모델을 갖춘 다양한 센서를 현실적으로 모델링한다.6

**약점/한계:**

- **렌더링 품질:** OGRE 엔진에 의존하는 그래픽은 기능적이지만, Unreal이나 Unity와 같은 현대 게임 엔진에 비해 사실적이지 않다.4
- **물리 정확도 한계:** 견고하지만, 물리 시뮬레이션은 강체 동역학에 중점을 둔다. 유체 동역학, 상세한 공기역학, 재료의 응력/변형률과 같은 복잡한 현상은 기본적으로 모델링하지 않는다.44 접촉 모델링 또한 부정확성과 불안정성의 원인이 될 수 있다.20
- **계산 비용:** 특히 복잡한 장면이나 많은 센서가 있는 경우 리소스를 많이 사용하여 저사양 하드웨어에서는 시뮬레이션 속도가 느려질 수 있다 (낮은 실시간 계수).34
- **사용 편의성:** 가파른 학습 곡선과 파편화된 문서는 초보자에게 어려움을 준다.32

Gazebo가 모터의 전류 소모량을 시뮬레이션하지 못하는 이유는 MATLAB Simscape와 같은 전기기계 회로 시뮬레이터가 아니라 강체 동역학 시뮬레이터이기 때문이다.44 마찬가지로, 렌더링이 Isaac Sim만큼 사실적이지 않은 이유는 Isaac Sim이 사실적인 렌더링을 위해 설계된 NVIDIA의 Omniverse를 기반으로 하는 반면, Gazebo는 최첨단 시각 효과보다는 오픈 소스 특성과 유용성을 위해 선택된 OGRE 그래픽 엔진을 기반으로 하기 때문이다.35 이러한 설계 경계를 인식하는 것이 중요하다. Gazebo는 이러한 영역에서 "실패"하고 있는 것이 아니라, ROS 기반 로봇 알고리즘 개발을 위한 유연하고 개방적이며 깊이 통합된 시뮬레이션 환경을 제공한다는 본래의 목적을 성공적으로 수행하고 있는 것이다.

### 4.2  현대 로봇 시뮬레이터 비교 분석

로봇 시뮬레이터 환경은 전문화되고 있다. 더 이상 단 하나의 "최고의" 시뮬레이터는 없으며, 특정 작업에 "적합한" 시뮬레이터가 있을 뿐이다. 선택은 프로젝트의 주요 목표에 따라 달라진다. 그것이 물리 기반 제어인가(Gazebo)? 비전 기반 강화학습인가(Isaac Sim)? 교육 및 신속한 프로토타이핑인가(Webots)? 아니면 인간 참여형 VR 실험인가(Unity)?

| **시뮬레이터**           | **주요 물리 엔진**          | **렌더링 품질**   | **ROS 통합**                         | **사용 편의성/학습 곡선**       | **핵심 강점 / 최적 분야**                         | **핵심 한계**                                |
| ------------------------ | --------------------------- | ----------------- | ------------------------------------ | ------------------------------- | ------------------------------------------------- | -------------------------------------------- |
| **Gazebo**               | ODE, Bullet, DART, Simbody  | 기능적 (비사실적) | 최상급 (ROS 2 브리지)                | 가파름                          | ROS 기반 제어 알고리즘 개발, 물리 기반 시뮬레이션 | 렌더링 품질, 초보자 접근성                   |
| **NVIDIA Isaac Sim**     | PhysX 5                     | 사실적 (RTX 가속) | 강력함 (Isaac ROS)                   | 중간 (USD 워크플로우 학습 필요) | 비전 기반 AI/RL 훈련, 합성 데이터 생성            | 고사양 NVIDIA GPU 필요                       |
| **Webots**               | 자체 엔진 (ODE 기반)        | 좋음              | 좋음 (다국어 지원)                   | 쉬움                            | 교육, 빠른 프로토타이핑, 초보자 친화적            | 물리 엔진 커스터마이징 제한                  |
| **CoppeliaSim**          | Bullet, ODE, Vortex, Newton | 보통              | 좋음 (원격 API)                      | 중간 (직관적 GUI)               | 다중 로봇/스웜 시뮬레이션, 산업용 로봇            | 복잡한 환경에서 성능 저하 가능               |
| **Unity (Robotics Hub)** | 자체 엔진 (PhysX/Havok)     | 최상급 (사실적)   | 플러그인/브리지 필요, 복잡할 수 있음 | 가파름 (게임 개발 지식 필요)    | 인간-로봇 상호작용(HRI), VR/AR, 시각화            | 공학적 물리 정확도 부족, 축 규약 불일치 문제 |

표 4.1: 주요 로봇 시뮬레이터 비교 분석

이러한 비교 분석 32을 통해, 프로젝트의 핵심 요구사항에 맞는 도구를 선택하는 것이 중요함을 알 수 있다. 예를 들어, 합성 데이터로 자율주행차의 인식 시스템을 훈련시키는 연구원은 사실적인 렌더링이 최우선이므로 Isaac Sim이 논리적인 선택이다.43 반면, 로봇 공학 입문 과정을 가르치는 교수는 설치가 쉽고 학습 곡선이 완만한 Webots를 선호할 수 있다.32 복잡한 휴머노이드의 새로운 보행 제어기를 개발하는 연구원은 우수한 ROS 지원과 유연한 물리 엔진을 갖춘 Gazebo가 여전히 강력한 선택지가 될 것이다.43

## 5. 장: 응용의 최전선: 현대 로봇 공학 연구에서의 Gazebo

이 장에서는 여러 최첨단 로봇 공학 연구 분야에서 Gazebo의 실제적인 영향을 살펴보고, 그 기능이 실제 문제를 해결하기 위해 어떻게 활용되는지 보여준다.

### 5.1  자율성 시뮬레이션: 자율주행차 및 드론 응용

Gazebo는 자율 시스템 시뮬레이션을 위한 범용 "작업마(workhorse)" 역할을 한다. 특정 영역에 대해 더 높은 충실도를 제공하는 전문 시뮬레이터(예: 자율주행을 위한 CARLA)가 존재하지만, Gazebo의 강점은 이기종 시스템(예: 지상 차량과 상호작용하는 드론)을 시뮬레이션할 수 있는 유연성과 이러한 차량을 제어하는 ROS 기반 소프트웨어 스택과의 깊은 통합에 있다.

- **자율주행:** Gazebo는 차량 동역학, 센서(카메라, LiDAR, GPS), 그리고 현실적인 환경을 모델링하여 Autoware 및 Apollo와 같은 자율주행 소프트웨어를 테스트하고 검증하는 데 사용된다.46 GzScenic과 같은 프로젝트는 상위 수준의 설명에서 Gazebo를 위한 복잡한 주행 시나리오를 자동으로 생성하여, CARLA와 같은 전문 시뮬레이터와의 격차를 줄이려는 시도이다.47
- **드론 (UAV):** Gazebo는 드론 시뮬레이션에 매우 널리 사용되며, 종종 PX4 및 ArduPilot과 같은 비행 제어 소프트웨어와 통합된다.46 드론 동역학, 제어 시스템(PID, MPC), 센서를 모델링하고 안전한 환경에서 행동을 테스트하는 데 이상적이다.46 RotorS와 같은 시뮬레이터는 Gazebo 위에 구축되어 MAV(Micro-Aerial Vehicle) 전용 시뮬레이션 프레임워크를 제공하기도 한다.50 또한 다중 드론 표적 추적과 같은 복잡한 임무에도 활용된다.51

### 5.2  상호작용의 도전: 로봇 조작 및 강화학습

로봇 조작(manipulation)과 강화학습(RL)은 시뮬레이터가 수행해야 하는 능력의 경계를 넓히고 있으며, 특히 접촉 물리와 심투리얼 전환과 관련하여 더욱 그렇다.

- **로봇 조작:** Gazebo는 픽앤플레이스(pick-and-place) 및 조립과 같은 작업을 위해 로봇 팔을 시뮬레이션하는 데 널리 사용된다.52 이를 통해 제어 알고리즘을 테스트하고, 기구학을 검증하며, 필요한 토크를 추정할 수 있다.44 조작 작업의 핵심 과제는 그리퍼가 물체에 닿을 때 발생하는 복잡한 접촉 물리를 정확하게 시뮬레이션하는 것이다. 이 지점에서 시뮬레이터는 종종 어려움을 겪으며, "심투리얼 갭(sim-to-real gap)"이 가장 두드러지게 나타난다.
- **강화학습 (RL):** Gazebo는 RL 에이전트 훈련을 위한 일반적인 환경이다. `gym-gazebo` 툴킷은 Gazebo를 OpenAI Gym 프레임워크와 연결하여 로봇 에이전트 훈련 과정을 표준화하기 위해 만들어졌다.53 RL 에이전트는 시뮬레이션의 부정확성을 "악용"하여 실제 세계에서는 작동하지 않는 해결책을 찾는 경향이 있기 때문에, RL은 물리 시뮬레이션의 정확도에 더욱 민감하다. 이 때문에 NVIDIA는 Isaac Sim의 물리 및 합성 데이터 생성 능력을 RL에 적극적으로 마케팅하고 있다.43
- **고급 상호작용:** Gazebo 커뮤니티는 이러한 한계에 머무르지 않고 혁신을 계속하고 있다. "Gazebo Plants" 플러그인은 코세라 막대(Cosserat rod) 물리를 사용하여 로봇 팔과 식물과 같은 변형 가능한 객체 간의 상호작용을 시뮬레이션한다.55 이는 기본 강체 물리로는 불가능한 작업으로, Gazebo를 특수 물리 플러그인으로 확장하여 차세대 로봇 공학 문제를 해결할 수 있는 길을 보여준다.

결론적으로, Gazebo는 이 분야의 기초적인 도구이지만, 접촉 충실도의 한계와 즉시 사용 가능한 RL 최적화의 부재는 Isaac Sim과 같은 경쟁 시뮬레이터의 개발과 Gazebo 자체의 혁신적인 확장을 모두 촉진하고 있다.

## 6. 장: 미래 궤적: Gazebo 로드맵 및 결론

이 마지막 장에서는 미래를 내다보며 공식 개발 로드맵을 분석하여 Gazebo의 진화를 예측하고, 로봇 공학 분야에서 Gazebo의 지속적인 중요성에 대한 결론적인 생각을 제시한다.

### 6.1  경로 설정: 공식 Gazebo 로드맵 분석

공개된 현대 Gazebo 로드맵은 프로젝트의 미래에 대한 이중 전략을 보여준다: (1) 플랫폼의 장기적인 건전성과 성능을 보장하기 위한 심층적인 기술 현대화, (2) 특히 강화학습(RL) 및 확장성 분야에서 현대 로봇 연구 커뮤니티의 요구와 문제점을 직접적으로 해결하기 위한 목표 지향적 기능 개발이다.56

- **기술 현대화:** Qt5에서 Qt6로의 마이그레이션(UI 프레임워크), Bazel 빌드 시스템 지원, 명령줄 도구를 위한 공유 라이브러리 대신 바이너리 사용 등은 화려한 기능은 아니지만 매우 중요한 기반 시설 투자이다. 이는 향후 몇 년간 성능, 안정성, 개발자 생산성 측면에서 이점을 가져다줄 것이다.56
- **성능 및 상호운용성:** 전송 계층에 Zenoh를 통합하는 계획은 현재 시스템보다 더 높은 성능과 유연한 통신 패턴을 제공할 수 있다.56
- **기능 향상:** RL을 위한 인터페이스 개선, 연합된 서드파티 플러그인 생태계 구축, 그리고 Gazebo Classic의 튜토리얼을 이식하여 문서 개선이라는 명시적인 목표는 커뮤니티의 동향과 요구에 대한 직접적인 응답이다.56 커뮤니티는 RL에 많은 투자를 하고 있으며 32, 더 나은 플러그인 생태계는 Gazebo Plants 55와 같은 확장을 더 쉽게 발견하고 사용할 수 있게 만들 것이다.
- **사용자 경험:** Fuel 서버에서 모델을 다운로드하는 프로세스를 개선하는 것은 사용자 경험을 향상시키기 위한 노력이다.56

이러한 로드맵 항목들은 기술적 기반을 공고히 하면서 사용자의 진화하는 요구에 적극적으로 적응하려는, 성숙하고 장기적인 비전을 가진 프로젝트의 모습을 보여준다.

### 6.2  결론: Gazebo의 지속적인 중요성과 예측된 진화

본 보고서의 분석을 종합하면, Gazebo의 지속적인 유산은 ROS와 깊이 통합된 무료 오픈 소스 도구를 제공함으로써 로봇 공학 시뮬레이션을 대중화하는 데 기여한 역할에 있다. 이는 한 세대의 연구자와 개발자들에게 진입 장벽을 낮춰주었다.

Gazebo의 미래는 모놀리식 애플리케이션이라기보다는 "연합 플랫폼(federated platform)"의 형태가 될 것으로 보인다. 모듈식 아키텍처는 Gazebo가 다양한 응용 프로그램을 위한 핵심 물리 및 센서 엔진 역할을 하도록 할 것이며, 새로운 물리 엔진, 렌더링 프론트엔드 또는 RL 인터페이스와 같은 전문화된 구성 요소는 커뮤니티에 의해 개발되고 통합될 것이다.

앞으로의 가장 큰 과제는 기술적인 것이 아니라 사용자 경험과 생태계 관리일 것이다. 즉, 커뮤니티가 Classic에서 현대 Gazebo로 성공적으로 전환하도록 안내하고, 문서를 통합하며, 강력하지만 복잡한 기능을 차세대 로봇 공학자들이 쉽게 접근할 수 있도록 만드는 것이다. 점점 더 경쟁이 치열해지는 로봇 시뮬레이션 환경에서 이 전환을 성공적으로 이끌어내는 능력이 궁극적으로 Gazebo의 미래 관련성을 결정할 것이다.

#### **참고 자료**

1. Design and use paradigms for Gazebo, an open-source multi-robot simulator, accessed July 10, 2025, https://www.researchgate.net/publication/4122159_Design_and_use_paradigms_for_Gazebo_an_open-source_multi-robot_simulator
2. Mastering Gazebo for Robotics - Number Analytics, accessed July 10, 2025, https://www.numberanalytics.com/blog/mastering-gazebo-for-robotics
3. Leo Rover Blog - Unreal world, real experience, accessed July 10, 2025, https://www.leorover.tech/post/unreal-world-real-experience-gazebo
4. Gazebo (simulator) - Wikipedia, accessed July 10, 2025, https://en.wikipedia.org/wiki/Gazebo_(simulator)
5. About -- Gazebo, accessed July 10, 2025, https://staging.gazebosim.org/about
6. Gazebo, accessed July 10, 2025, https://classic.gazebosim.org/
7. Gazebo Plugins - RoboComp | A simple robotics framework., accessed July 10, 2025, https://robocomp.github.io/web/gsoc/2018/akash_kumar_singh/post3
8. SDFormat Home, accessed July 10, 2025, http://sdformat.org/
9. 
10. SDFormat Specification, accessed July 10, 2025, http://sdformat.org/spec
11. A-Gentle-Introduction-to-Gazebo-Modeling/Tutorial_EN.md at main - GitHub, accessed July 10, 2025, https://github.com/AzulRadio/A-Gentle-Introduction-to-Gazebo-Modeling/blob/main/Tutorial_EN.md
12. SDFormat Specification - Documentation, accessed July 10, 2025, http://sdformat.org/tutorials
13. Standard Delay Format - Wikipedia, accessed July 10, 2025, https://en.wikipedia.org/wiki/Standard_Delay_Format
14. draft-ietf-asdf-sdf-23 - Semantic Definition Format (SDF) for Data and Interactions of Things, accessed July 10, 2025, https://datatracker.ietf.org/doc/draft-ietf-asdf-sdf/
15. ROS Gazebo: Everything You Need To Know - Robotic Simulation Services, accessed July 10, 2025, https://roboticsimulationservices.com/ros-gazebo-everything-you-need-to-know/
16. Gazebo : Tutorial : Physics Parameters - Gazebo Classic, accessed July 10, 2025, https://classic.gazebosim.org/tutorials?tut=physics_params
17. Who Uses DART? - DART: Dynamic Animation and Robotics Toolkit 7.0.0-alpha20230101 documentation, accessed July 10, 2025, https://dart.readthedocs.io/en/latest/community/who_uses_dart.html
18. Physics engines - Gazebo Sim, accessed July 10, 2025, https://gazebosim.org/api/sim/9/physics.html
19. Comparison of Rigid Body Dynamic Simulators for Robotic Simulation in Gazebo - Open Robotics, accessed July 10, 2025, https://www.osrfoundation.org/wordpress2/wp-content/uploads/2015/04/roscon2014_scpeters.pdf
20. [gazebo simulator] is it possible to increase the physics max step size? / robotology / Discussion #161 - GitHub, accessed July 10, 2025, https://github.com/orgs/robotology/discussions/161
21. Gazebo in AI Robotics - Number Analytics, accessed July 10, 2025, https://www.numberanalytics.com/blog/ultimate-guide-gazebo-ai-robotics
22. Simulating with Gazebo - Articulated Robotics, accessed July 10, 2025, https://articulatedrobotics.xyz/tutorials/ready-for-ros/gazebo/
23. Tutorial : Sensor Noise Model - Gazebo, accessed July 10, 2025, https://classic.gazebosim.org/tutorials?tut=sensor_noise
24. gazebo_tutorials/guided_i/tutorial3.md at master - GitHub, accessed July 10, 2025, https://github.com/osrf/gazebo_tutorials/blob/master/guided_i/tutorial3.md
25. 
26. Tutorials - Gazebo, accessed July 10, 2025, https://classic.gazebosim.org/tutorials
27. Gazebo Tutorials - Gazebo ionic documentation, accessed July 10, 2025, https://gazebosim.org/docs/latest/tutorials/
28. Tutorial : Plugins 101 - Gazebo, accessed July 10, 2025, https://classic.gazebosim.org/tutorials?tut=plugins_hello_world
29. Gazebo Sim: Migration from Gazebo Classic: Plugins, accessed July 10, 2025, https://gazebosim.org/api/sim/9/migrationplugins.html
30. Thoughts on Ignition Gazebo Simulator and ROS2 bridge - ROS Discourse, accessed July 10, 2025, https://discourse.ros.org/t/thoughts-on-ignition-gazebo-simulator-and-ros2-bridge/16477
31. Elevating robotics CI with Gazebo simulations: case study - Spyrosoft, accessed July 10, 2025, https://spyro-soft.com/developers/elevating-robotics-ci-with-gazebo-simulations-case-study
32. Why is Gazebo very famous in the ROS community? what about Webots?, accessed July 10, 2025, https://discourse.ros.org/t/why-is-gazebo-very-famous-in-the-ros-community-what-about-webots/42459
33. Best Robot Simulators - Formant, accessed July 10, 2025, https://formant.io/blog/best-robot-simulators/
34. Choose a Simulator - Robotics Knowledgebase, accessed July 10, 2025, https://roboticsknowledgebase.com/wiki/robotics-project-guide/choose-a-sim/
35. Why do we use gazebo instead of unreal or unity - General - ROS Discourse, accessed July 10, 2025, https://discourse.ros.org/t/why-do-we-use-gazebo-instead-of-unreal-or-unity/25890
36. Home - Gazebo Answers archive, accessed July 10, 2025, https://answers.gazebosim.org/
37. Problem using gazebo launch : r/robotics - Reddit, accessed July 10, 2025, https://www.reddit.com/r/robotics/comments/13s3ebm/problem_using_gazebo_launch/
38. Problem with Gazebo - Jetson AGX Xavier - NVIDIA Developer Forums, accessed July 10, 2025, https://forums.developer.nvidia.com/t/problem-with-gazebo/65769
39. Problems compiling gazebo libraries with NVCC - NVIDIA Developer Forums, accessed July 10, 2025, https://forums.developer.nvidia.com/t/problems-compiling-gazebo-libraries-with-nvcc/292719
40. Instability found with gazebo model with dex wrist - Ask - Stretch Forum - Hello Robot, accessed July 10, 2025, https://forum.hello-robot.com/t/instability-found-with-gazebo-model-with-dex-wrist/808
41. Limits of the Simulation - Help - Gazebo Community, accessed July 10, 2025, https://community.gazebosim.org/t/limits-of-the-simulation/54
42. What are the drawbacks of running multiple Gazebo simulations using Multimaster FKIE package, accessed July 10, 2025, https://answers.gazebosim.org/question/22671/
43. Best simulation tools for robotics engineers: A comprehensive ..., accessed July 10, 2025, https://roboticsbiz.com/best-simulation-tools-for-robotics-engineers-a-comprehensive-review/
44. Is Gazebo good for electromechanical design simulation? : r/robotics - Reddit, accessed July 10, 2025, https://www.reddit.com/r/robotics/comments/r2t2gj/is_gazebo_good_for_electromechanical_design/
45. How good is Gazebo? : r/robotics - Reddit, accessed July 10, 2025, https://www.reddit.com/r/robotics/comments/1ku737z/how_good_is_gazebo/
46. Gazebo for Autonomous Systems - Number Analytics, accessed July 10, 2025, https://www.numberanalytics.com/blog/gazebo-autonomous-systems-simulation
47. GzScenic: Automatic Scene Generation for Gazebo Simulator, accessed July 10, 2025, https://arxiv.org/abs/2104.08625
48. Gazebo scenario for "Drone cat and mouse" exercise. - ResearchGate, accessed July 10, 2025, https://www.researchgate.net/figure/Gazebo-scenario-for-Drone-cat-and-mouse-exercise_fig8_346367625
49. ROS/Gazebo based simulation environment for UAV controller performance evaluation - POLITECNICO DI TORINO, accessed July 10, 2025, https://webthesis.biblio.polito.it/26714/1/tesi.pdf
50. Bridging the Gap between Simulation and Real Autonomous UAV Flights in Industrial Applications - MDPI, accessed July 10, 2025, https://www.mdpi.com/2226-4310/10/9/814
51. Integrated Multi-Simulation Environments for Aerial Robotics Research - arXiv, accessed July 10, 2025, https://arxiv.org/html/2502.10218v1
52. Ignition Gazebo: The Future of Robotics Simulation - Number Analytics, accessed July 10, 2025, https://www.numberanalytics.com/blog/ignition-gazebo-future-of-robotics-simulation
53. gym-gazebo2, a toolkit for reinforcement learning using ROS 2 and Gazebo - arXiv, accessed July 10, 2025, https://arxiv.org/pdf/1903.06278
54. [2204.06433] A Systematic Comparison of Simulation Software for Robotic Arm Manipulation using ROS2 - arXiv, accessed July 10, 2025, https://arxiv.org/abs/2204.06433
55. [2402.02570] Gazebo Plants: Simulating Plant-Robot Interaction with Cosserat Rods - arXiv, accessed July 10, 2025, https://arxiv.org/abs/2402.02570
56. Gazebo Roadmap - Gazebo ionic documentation, accessed July 10, 2025, https://gazebosim.org/docs/latest/roadmap/


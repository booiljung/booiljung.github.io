# mav_local_planner에 대한 심층 분석 및 기술적 고찰

## 1.  `mav_local_planner`의 개요 및 아키텍처

### 1.1  핵심 기능 및 설계 철학

`mav_local_planner`는 스위스 취리히 연방 공과대학교(ETH Zurich)의 자율 시스템 연구실(Autonomous Systems Lab, ASL)에서 개발한 `mav_voxblox_planning` 패키지의 핵심 구성 요소이다.1 이 플래너는 마이크로 항공기(Micro Aerial Vehicle, MAV)가 미지의 또는 부분적으로만 알려진 3차원 환경에서 안전하고 동역학적으로 실행 가능한(dynamically feasible) 경로를 실시간으로 생성하도록 설계된 지역 경로 계획기(local planner)이다.1

`mav_local_planner`의 핵심 기능은 크게 두 가지로 구분할 수 있다. 첫째는 단일 목표점(waypoint)을 향해 단기 예측 구간(short-horizon) 내에서 장애물과 충돌을 회피하며 비행하는 기능이며, 둘째는 주어진 목표점 목록으로 구성된 전역 경로를 부드럽게 추종하면서 실시간으로 경로를 수정 및 재계획(replanning)하는 기능이다.1 이러한 기능은 약 4 Hz의 주기, 즉 초당 4회의 재계획을 통해 달성되어 MAV가 변화하는 환경이나 새로운 정보에 신속하게 대응할 수 있도록 한다.1

이 플래너의 근본적인 설계 철학은 '계획과 제어의 분리 및 연계'로 요약된다. 이는 전역 경로 계획기(예: `voxblox_rrt_planner`)가 생성한 순수한 기하학적 경로(geometric path)를 입력으로 받아, 이를 MAV의 동역학적 제약 조건과 주변 환경의 장애물 정보를 통합적으로 고려하여 실행 가능한 시-공간 궤적(timed, dynamically feasible trajectories)으로 변환하는 것이다. 이렇게 생성된 최종 궤적은 MAV의 하위 레벨 제어기 인터페이스로 전송되어 실제 비행을 수행하게 된다.1 이 과정은 MAV의 자율 비행에서 고수준의 임무 계획과 저수준의 모터 제어 사이를 잇는 중요한 가교 역할을 수행한다.

### 1.2  시스템 아키텍처 및 의존성

`mav_local_planner`는 단일 패키지로 독립적으로 동작하는 것이 아니라, 여러 전문화된 모듈이 긴밀하게 결합된 복합적인 시스템 아키텍처를 가진다. 그 핵심 의존성은 환경 표현, 궤적 평활화, 그리고 전역 계획기와의 연동으로 구성된다.

#### 1.2.1 환경 표현 (Map Representation)

`mav_local_planner`의 성능은 환경을 어떻게 표현하고 이해하는가에 절대적으로 의존하며, 이를 위해 `voxblox` 패키지를 기반으로 한다.1

`voxblox`는 3차원 센서(예: 깊이 카메라, LiDAR)로부터 수집된 데이터를 통합하여, 먼저 TSDF(Truncated Signed Distance Field) 볼륨을 생성한다. TSDF는 센서 측정에 포함된 노이즈를 여러 관측에 걸쳐 효과적으로 평활화하는 장점이 있다.3 이후, 이 TSDF로부터 ESDF(Euclidean Signed Distance Field) 맵을 증분적으로(incrementally) 구축한다. ESDF는 공간상의 모든 지점에서 가장 가까운 장애물까지의 부호화된 유클리드 거리를 저장하는 맵으로, 궤적 최적화 과정에서 충돌 비용(collision cost)을 매우 효율적으로 계산하는 데 결정적인 정보를 제공한다. 이 방식은 Octomap과 같은 다른 3차원 맵 표현 방식에 비해 ESDF를 더 빠르고 정확하게 구축할 수 있는 것으로 알려져 있다.3

#### 1.2.2 궤적 평활화 (Trajectory Smoothing)

생성된 경로를 MAV가 실제로 비행할 수 있도록 부드럽게 다듬는 과정은 `mav_path_smoothing` 패키지를 통해 이루어진다. `mav_local_planner`는 설정에 따라 다음과 같은 여러 평활화 모듈을 선택적으로 사용할 수 있다.2

- **`VelocityRampSmoother`**: 경로의 속도 프로파일을 부드럽게 조절하여 급격한 속도 변화를 방지한다.
- **`PolynomialSmoother`**: 경로가 장애물과 충돌하는 지점에 새로운 정점(vertex)을 추가하고, 이 정점들을 통과하는 다항식으로 경로를 재구성하여 충돌을 회피한다.
- **`LocoSmoother`**: `mav_local_planner`의 가장 핵심적인 최적화 엔진으로, LOCO(Local polynomial optimization) 알고리즘을 구현한다. 이 스무서는 주어진 웨이포인트를 엄격한 제약으로 간주하지 않고(unconstraining waypoints), 충돌 비용과 궤적의 평활도를 동시에 최소화하는 최적화 문제를 푼다. 이는 `loco_planner` 패키지와 연계되어 동작하며, 플래너의 유연성과 강인성을 크게 향상시킨다.1

#### 1.2.3 전역 계획기와의 연동

`mav_local_planner`는 단독으로 특정 목표점을 향해 비행할 수도 있지만, 일반적으로는 전역 계획기(global planner)와 함께 계층적 구조로 사용된다. `mav_voxblox_planning` 소프트웨어 스위트에는 OMPL(Open Motion Planning Library)을 기반으로 하는 RRT*(Rapidly-exploring Random Tree Star)나 BIT*(Batch Informed Trees) 같은 샘플링 기반의 `voxblox_rrt_planner`와, 환경의 위상적 구조를 활용하는 `voxblox_skeleton_planner`가 포함되어 있다.1 표준적인 사용 방식은 "global + local planning" 워크플로우로, 전역 계획기가 맵 전체를 고려하여 출발점에서 목적지까지의 기하학적 경로를 생성하면, `mav_local_planner`가 이 경로를 입력받아 주변 환경을 고려하며 안전하게 추종하는 역할을 담당한다.1

### 1.3  ROS 인터페이스 분석

`mav_local_planner`는 ROS(Robot Operating System) 노드로서, 표준적인 ROS 인터페이스인 토픽(Topic), 서비스(Service), 파라미터(Parameter)를 통해 다른 노드들과 통신하고 동작을 제어한다. `mav_local_planner.h` 헤더 파일과 다양한 실행 예제를 통해 그 상세한 인터페이스를 파악할 수 있다.2

- **주요 구독 토픽 (Subscribers):**
  - `odometry_sub_`: MAV의 현재 위치, 자세, 속도 등의 상태 정보를 `nav_msgs/Odometry` 메시지 형태로 수신한다. (예: `/firefly/ground_truth/odometry`).4
  - `waypoint_sub_`: 사용자가 Rviz 플러그인 등을 통해 지정하는 단일 목표 지점을 `geometry_msgs/PoseStamped` 메시지로 수신한다. (예: `/firefly/waypoint`).4
  - `waypoint_list_sub_`: 전역 계획기가 생성한 일련의 경로점 목록을 `geometry_msgs/PoseArray` 메시지로 수신한다.4
  - ESDF/TSDF Map: `voxblox_node`로부터 생성된 맵 데이터를 내부적으로 구독하여 경로 계획에 활용한다. (예: `/firefly/esdf_map`, `voxblox_msgs/Layer`).5
- **주요 발행 토픽 (Publishers):**
  - `command_pub_`: 최적화 과정을 통해 계산된 최종 궤적을 MAV의 하위 레벨 제어기로 전송한다. `trajectory_msgs/MultiDOFJointTrajectory` 메시지 타입을 사용하며, 시간에 따른 위치, 속도, 가속도 정보를 포함한다. (예: `/firefly/command/trajectory`).4
  - `path_marker_pub_`: 계획된 지역 경로를 Rviz와 같은 시각화 도구에서 확인할 수 있도록 `visualization_msgs/MarkerArray` 형태로 발행한다. (예: `/firefly/mav_local_planner/local_path`).4
- **서비스 (Services):**
  - `start_srv_`, `pause_srv_`, `stop_srv_`: `std_srvs/Empty` 타입의 서비스를 통해 외부에서 플래너의 동작을 제어할 수 있다. 각각 경로 발행을 시작, 일시 중지, 그리고 완전히 정지하고 현재 궤적을 초기화하는 기능을 수행한다.4
- **주요 파라미터 (Key Parameters):**
  - `smoother_name_`: 사용할 궤적 평활화기(`loco`, `poly`, `ramp`)를 문자열로 지정한다.4
  - `avoid_collisions_`: 충돌 회피 기능의 활성화 여부를 불리언 값으로 결정한다.4
  - `local_planning_horizon`: 지역 경로 계획의 시간 범위(초 단위)를 설정한다. 이 값이 클수록 더 먼 미래까지 예측하지만 계산량이 증가한다.6
  - `command_publishing_dt`: 제어 명령을 발행하는 시간 간격(초 단위)을 설정한다.6

아래 표는 `mav_local_planner`의 핵심 ROS API를 요약한 것이다.

| 타입 (Type) | 토픽/서비스 이름 (Topic/Service Name)                        | 메시지/서비스 타입 (Message/Service Type) | 설명 (Description)                 |
| ----------- | ------------------------------------------------------------ | ----------------------------------------- | ---------------------------------- |
| 구독자      | `odometry_sub_` (예: `/firefly/ground_truth/odometry`)       | `nav_msgs/Odometry`                       | MAV의 현재 상태(Odometry) 수신     |
| 구독자      | `waypoint_sub_` (예: `/firefly/waypoint`)                    | `geometry_msgs/PoseStamped`               | 단일 목표 지점(waypoint) 수신      |
| 구독자      | `waypoint_list_sub_` (예: `/firefly/voxblox_rrt_planner/path`) | `geometry_msgs/PoseArray`                 | 전역 경로점 목록 수신              |
| 발행자      | `command_pub_` (예: `/firefly/command/trajectory`)           | `trajectory_msgs/MultiDOFJointTrajectory` | MAV 제어기로 전송할 최적 궤적 발행 |
| 발행자      | `path_marker_pub_` (예: `/firefly/mav_local_planner/local_path`) | `visualization_msgs/MarkerArray`          | Rviz 시각화를 위한 지역 경로 발행  |
| 서비스      | `start_srv_`                                                 | `std_srvs/Empty`                          | 경로 발행 시작                     |
| 서비스      | `pause_srv_`                                                 | `std_srvs/Empty`                          | 경로 발행 일시 중지                |
| 서비스      | `stop_srv_`                                                  | `std_srvs/Empty`                          | 경로 발행 정지 및 궤적 초기화      |

`mav_local_planner`의 아키텍처는 몇 가지 중요한 설계적 특징을 시사한다. 첫째, 이 플래너는 단일체(monolithic) 구조가 아닌, `voxblox`(맵핑), `mav_path_smoothing`(최적화), `mav_planning_rviz`(GUI) 등 여러 전문화된 모듈이 긴밀하게 결합된 시스템이다.1 이는 `mav_local_planner.h` 헤더 파일이 다른 패키지의 헤더들을 다수 포함하고 있다는 점에서도 확인된다.4 이러한 모듈식 설계는 `smoother_name_` 파라미터 변경만으로 다른 최적화 알고리즘을 사용하는 등 높은 유연성을 제공하지만, 반대로 사용자가 전체 생태계를 올바르게 이해하고 설정해야 하는 복잡성을 내포한다. 이는 `move_base`에 대한 단일 플러그인 형태로 제공되는 `TEB`나 `DWA`와 같은 플래너들과 대조되는 지점이다.7

둘째, 아키텍처는 전역 경로를 따르는 '경로 추종' 모드와 단일 목표점에 반응하는 '목표 지향' 모드를 명시적으로 분리한다. 이는 `waypointListCallback`과 `waypointCallback`이라는 두 개의 개별 콜백 함수 정의와, 문서에서 설명하는 두 가지 뚜렷한 사용 사례("Local Planning"과 "global + local planning")를 통해 드러난다.2 이러한 이중적 설계는 플래너의 활용도를 높인다. 이미 맵이 잘 알려진 환경에서는 최적의 전역 경로를 효율적으로 추종할 수 있으며, 미지의 환경을 탐사하거나 동적인 상황에 대응해야 할 때는 상위 레벨의 의사결정 모듈(예: 탐사 플래너)이나 인간 조작자가 제공하는 단기 목표점을 따라 반응적으로 움직일 수 있다. 이 유연성은 `mav_local_planner`의 핵심 강점 중 하나이지만, 두 모드 간의 전환과 상태 관리를 신중하게 설계해야 함을 의미하기도 한다.

## 2.  핵심 기반 알고리즘 분석

`mav_local_planner`의 성공적인 동작은 몇 가지 핵심적인 기반 알고리즘에 깊이 의존하고 있다. 이들은 환경을 수학적으로 표현하고, 동역학적으로 실행 가능한 궤적을 생성하며, 이를 실시간으로 최적화하는 역할을 담당한다.

### 2.1  맵 표현: ESDF와 `voxblox`

성공적인 3차원 자율 비행의 가장 근본적인 전제 조건은 주변 환경을 효과적이고 효율적으로 표현하는 것이다. `mav_local_planner`는 이 문제를 해결하기 위해 `voxblox` 패키지를 통해 생성된 ESDF(Euclidean Signed Distance Field)를 그 기반으로 삼는다.1

ESDF는 3차원 공간을 일정한 크기의 복셀(voxel) 그리드로 나누고, 각 복셀의 중심점에 가장 가까운 장애물까지의 유클리드 거리를 저장하는 방식으로 환경을 표현한다. 이때, 자유 공간에 위치한 복셀은 양수 값을, 장애물 내부에 위치한 복셀은 음수 값을 가지며, 장애물 표면은 정확히 0의 값을 갖는다. 이러한 표현 방식은 단순한 점유 격자 맵(occupancy grid map)과 비교하여 궤적 최적화에 필수적인 두 가지 핵심 정보를 제공한다.

1. **충돌 거리 조회:** 특정 지점의 ESDF 값은 그 지점과 가장 가까운 장애물 사이의 거리를 즉시 알려준다. 이는 궤적 상의 한 점이 충돌 위험이 있는지, 그리고 안전 여유(safety margin)가 얼마나 되는지를 상수 시간 복잡도($O(1)$)로 조회할 수 있게 해준다.
2. **충돌 그래디언트:** ESDF의 그래디언트(gradient, 기울기)는 해당 지점에서 가장 가까운 장애물로부터 멀어지는 방향, 즉 가장 가파르게 거리가 증가하는 방향을 가리킨다. 이 그래디언트 벡터는 궤적 최적화 과정에서 장애물을 회피하도록 밀어내는 가상의 '반발력'을 계산하는 데 결정적인 역할을 한다. `mav_local_planner`는 `getMapDistanceAndGradient` 함수를 통해 이 정보를 얻는다.4

`voxblox`는 이러한 ESDF를 효율적으로 구축하고 실시간으로 업데이트하기 위해, 먼저 TSDF(Truncated Signed Distance Field)를 중간 단계로 활용한다.3 TSDF는 센서로부터 일정 거리 내의 정보만을 잘라서 저장하며, 여러 프레임에 걸쳐 들어오는 센서의 측정 노이즈를 평균화하여 부드럽고 일관된 표면을 재구성하는 데 유리하다. 

`voxblox`는 이 TSDF로부터 ESDF를 증분적으로(incrementally) 업데이트함으로써, MAV가 움직이며 새로운 환경을 관측할 때마다 전체 맵을 재계산할 필요 없이 효율적으로 맵을 확장하고 갱신할 수 있다.

### 2.2  궤적 최적화 기법: 다항식 스플라인과 최소 스냅

MAV, 특히 쿼드로터는 고차 미분 방정식으로 모델링되는 복잡한 동역학을 가진다. 따라서 단순히 기하학적인 경로점을 연결하는 것만으로는 부족하며, 시간에 따라 위치, 속도, 가속도 등이 연속적이고 부드럽게 변하는 동역학적으로 실행 가능한 궤적을 생성해야 한다.

#### 2.2.1 미분적 평탄성 (Differential Flatness)

쿼드로터 동역학은 '미분적으로 평탄하다(differentially flat)'는 매우 중요한 수학적 특성을 가지고 있다.9 이는 시스템의 전체 상태(12차원: 위치, 속도, 회전 행렬, 각속도)와 제어 입력(4개의 로터 추력)을 '평탄 출력(flat outputs)'이라 불리는 더 적은 수의 특정 변수들과 그 유한한 수의 시간 미분 값들만으로 완벽하게 표현할 수 있음을 의미한다. 쿼드로터의 경우, 이 평탄 출력은 보통 3차원 위치 ($x, y, z$)와 요(yaw) 각도($\psi$)의 4차원 벡터 $σ(t) = [x(t), y(t), z(t), \psi(t)]^T$로 정의된다.9

이 특성 덕분에, 우리는 복잡하고 비선형적인 전체 상태 공간에서 최적화 문제를 직접 푸는 대신, 상대적으로 다루기 쉬운 4차원 평탄 출력 공간에서 궤적을 계획할 수 있다. 평탄 출력에 대한 부드러운 궤적 $σ(t)$가 일단 생성되면, 이를 여러 번 미분하여 원하는 시점의 속도, 가속도, 저크, 스냅 등을 계산하고, 이를 통해 필요한 추력과 로봇의 자세를 해석적으로(analytically) 유도해낼 수 있다.9 이로 인해 계획 단계에서 반복적인 동역학 시뮬레이션이 필요 없어져 계산 효율성이 극대화된다.

#### 2.2.2 다항식 스플라인 (Polynomial Splines)

평탄 출력 공간에서의 궤적은 주로 구간별 다항식(piecewise polynomials)으로 표현된다.9 즉, 전체 경로는 여러 개의 웨이포인트로 나뉘고, 각 웨이포인트 사이의 구간은 하나의 다항식 함수로 기술된다. 이 다항식 조각들을 부드럽게 이어붙인 전체 궤적을 다항식 스플라인(spline)이라고 한다. 다항식은 무한히 미분 가능하므로, 궤적의 평활도(smoothness)를 원하는 수준까지 보장하는 데 이상적인 수학적 도구이다.9

#### 2.2.3 최소 스냅 (Minimum Snap) 최적화

궤적의 '품질'을 정량화하고 최상의 궤적을 찾기 위해 비용 함수(cost function)를 정의하고 이를 최소화하는 최적화 문제를 푼다. MAV 제어에서는 궤적의 4차 미분인 '스냅(snap)'의 제곱 적분 값을 최소화하는 '최소 스냅' 방식이 표준처럼 널리 사용된다.9 쿼드로터의 모터에 가해지는 명령과 기체의 각가속도는 경로의 스냅에 비례하는 경향이 있다. 따라서 스냅을 최소화하면 MAV의 급격한 움직임과 진동이 줄어들어 에너지가 절약되고, 탑재된 카메라나 센서로 취득하는 데이터의 질이 향상되며, 제어 입력이 포화 상태에 이르지 않도록 하는 데 유리하다.13

#### 2.2.4 최소 스냅 QP(Quadratic Program) 정식화

하나의 1차원 다항식 구간 $p(t)$에 대한 최소 스냅 문제는 다음과 같은 이차 계획법(Quadratic Program, QP) 문제로 정식화할 수 있다.14

- **비용 함수 (Cost Function):** 궤적의 총 스냅 에너지를 최소화하는 것을 목표로 한다.
  $$
  \min_{p(t)} J = \int_{0}^{T} \left( \frac{d^4 p(t)}{dt^4} \right)^2 dt
  $$
  여기서 $p(t) = \sum_{i=0}^{N} c_i t^i$는 N차 다항식이고, $c_i$는 우리가 찾아야 할 계수이다. 이 비용 함수는 다항식 계수 벡터 $\mathbf{c}$에 대한 이차 형식(quadratic form)인 $\frac{1}{2}\mathbf{c}^T \mathbf{Q} \mathbf{c}$ 형태로 정확하게 표현될 수 있어 QP 문제로 변환하기에 매우 적합하다.14

- **제약 조건 (Constraints):** 궤적은 특정 지점에서 특정 조건을 만족해야 한다.

  1. **정점 제약 (Vertex Constraints):** 각 구간의 시작점($t_{start}$)과 끝점($t_{end}$)에서 위치, 속도, 가속도, 저크(jerk) 등의 미분값이 사전에 정의된 값을 갖도록 강제한다. 이는 주로 전체 경로의 시작과 끝, 그리고 중간 경유점(waypoint)에 적용된다.
     $$
     1\begin{cases}
     p^{(k)}(t_{start}) = p_{start}^{(k)} \\
     p^{(k)}(t_{end}) = p_{end}^{(k)}
     \end{cases}
     \quad \text{for } k = 0, 1, 2, 3, \dots
     $$

  2. **연속성 제약 (Continuity Constraints):** 여러 다항식 구간을 이어붙일 때, 연결 지점에서 궤적이 끊어지거나 급격하게 변하지 않도록 위치부터 스냅까지의 모든 미분값이 양쪽 구간에서 동일해야 한다.

  이러한 제약 조건들은 모두 다항식 계수 벡터 $\mathbf{c}$에 대한 선형 등식 제약(linear equality constraints), 즉 $\mathbf{A}\mathbf{c} = \mathbf{b}$ 형태로 표현될 수 있다.

- **최종 QP 문제:** 결과적으로 전체 궤적에 대한 최소 스냅 문제는 다음과 같은 표준적인 QP 형태로 귀결된다. 이는 볼록 최적화(convex optimization) 문제이므로 전역 최적해를 효율적으로 찾을 수 있다.14
  $$
  \begin{aligned}
  & \underset{\mathbf{c}}{\text{minimize}} & & \frac{1}{2}\mathbf{c}^T \mathbf{Q} \mathbf{c} \\
  & \text{subject to} & & \mathbf{A}_{eq}\mathbf{c} = \mathbf{b}_{eq} \quad \text{(Equality constraints)} \\
  & & & \mathbf{A}_{ineq}\mathbf{c} \le \mathbf{b}_{ineq} \quad \text{(Inequality constraints, e.g., corridor constraints)}
  \end{aligned}
  $$
  `mav_local_planner`는 이러한 QP 문제를 효율적인 솔버를 이용해 풀어 최적의 궤적 계수 $\mathbf{c}$를 실시간으로 찾아낸다.

### 2.3  지역 최적화: LOCO 알고리즘

`mav_local_planner`가 전역 경로를 평활화하거나 지역적인 회피 기동을 생성할 때 사용하는 핵심적인 최적화 기법은 `LocoSmoother`이며, 이는 LOCO(Local Trajectory Optimization in Continuous-Time) 알고리즘에 기반한다.1

LOCO 알고리즘은 `mav_path_smoothing` 패키지의 일부로서, 주어진 웨이포인트들을 통과하는 궤적을 최적화하되, 웨이포인트를 반드시 통과해야 하는 엄격한 제약(hard constraint)이 아닌, 비용 함수에 포함되는 부드러운 제약(soft constraint)으로 취급하는 것이 가장 큰 특징이다.1

LOCO의 최적화 비용 함수는 일반적으로 다음과 같은 항들의 가중치 합으로 구성된다:

1. **평활도 비용 (Smoothness Cost):** 궤적의 고차 미분값(예: 가속도, 저크, 스냅)의 제곱 적분을 최소화하여 부드러운 궤적을 유도한다. 이는 앞서 설명한 최소 스냅 문제의 비용 함수와 본질적으로 동일하다.
2. **충돌 비용 (Collision Cost):** 궤적이 장애물에 가까워질수록 기하급수적으로 증가하는 비용이다. 이 비용은 ESDF 맵으로부터 실시간으로 조회한 거리 정보를 사용하여 계산된다.
3. **웨이포인트 비용 (Waypoint Cost):** 궤적이 주어진 웨이포인트로부터 멀어질수록 증가하는 비용이다.

LOCO는 웨이포인트를 '통과해야만 하는 점'이 아니라 '가급적 가까이 지나가야 할 점'으로 취급함으로써(unconstraining waypoints), 최적화 과정에 더 큰 자유도를 부여한다. 이 덕분에 만약 전역 계획기가 생성한 웨이포인트가 장애물에 너무 가깝거나 내부에 위치할 경우, 플래너는 웨이포인트에서 약간 벗어나더라도 충돌을 회피하는 더 안전하고 합리적인 경로를 찾을 수 있다.2 이러한 유연성은 특히 전역 경로의 품질이 완벽하지 않을 수 있는 실제 상황에서 플래너의 강인성을 크게 높여준다.

이 알고리즘의 명칭은 입자 가속기 분야에서 사용되는 LOCO(Linear Optics from Closed Orbits) 알고리즘과 동일하지만, 이는 우연의 일치이며 두 기술 간의 직접적인 관련은 없다.18 MAV 분야에서 LOCO는 지역적인 연속 시간 궤적 최적화 프레임워크를 지칭하는 고유한 용어로 사용된다.

`mav_local_planner`의 핵심 알고리즘들을 분석해 보면, 이 플래너가 "계획 후 최적화(Plan-then-Optimize)"라는 명확한 패러다임을 따르고 있음을 알 수 있다. 먼저 RRT*와 같은 기하학적 경로 탐색 알고리즘을 사용하여 장애물을 고려한 초기 경로를 찾고 1, 그 다음 단계에서 이 경로를 최소 스냅 및 LOCO와 같은 최적화 기법을 통해 동역학적으로 실행 가능하고 안전한 최종 궤적으로 다듬는다.1 이 2단계 접근법은 각 단계에 가장 적합하고 강력한 알고리즘을 적용할 수 있게 하여 계산적으로 구조화되어 있다는 장점이 있다. 그러나 이는 필연적으로 최종 궤적의 품질이 초기 경로의 품질에 크게 의존하게 되는 결과를 낳는다. 만약 초기 경로가 매우 비효율적이거나, 복잡한 장애물 지형을 통과하는 "잘못된" 호모토피 클래스(homotopy class, 위상학적으로 다른 경로군)에 속해 있다면, 지역 최적화 단계에서 이를 근본적으로 "좋은" 궤적으로 수정하는 데는 명백한 한계가 존재한다. 이는 장애물의 양쪽 경로를 동시에 고려하여 최적화하는 `TEB` 플래너의 다중 토폴로지 탐색 전략과는 대조적인 부분이다.19

또한, 이 모든 최적화 프레임워크의 근간에는 미분적 평탄성이라는 속성이 자리 잡고 있다. 이 속성이 없다면, 계획은 훨씬 더 계산 비용이 높은 전체 상태 공간에서 이루어져야 하므로 실시간 적용이 거의 불가능했을 것이다.9 미분적 평탄성은 복잡한 비선형 제어 문제를 다루기 쉬운 저차원 유클리드 공간에서의 궤적 최적화 문제로 변환시켜주는 강력한 도구이다. 이는 `mav_local_planner`가 실시간으로 공격적인 궤적을 생성할 수 있게 하는 핵심 요인이지만, 동시에 이 접근법이 미분적으로 평탄한 시스템에만 직접적으로 적용될 수 있다는 본질적인 한계를 규정하기도 한다.

## 3.  주요 로컬 플래너와의 비교 분석

`mav_local_planner`의 특징과 성능을 객관적으로 평가하기 위해서는, 다른 주요 로컬 플래너들과의 비교 분석이 필수적이다. 비교 대상은 크게 2차원 지상 로봇을 위해 개발된 플래너들과, 동일하게 3차원 공간을 비행하는 MAV를 위해 개발된 플래너들로 나눌 수 있다.

### 3.1  2D 지상 로봇 플래너와의 개념적 차이

ROS `navigation` 스택의 표준 로컬 플래너로 널리 사용되는 `DWA`와 `TEB`는 `mav_local_planner`와 근본적으로 다른 문제 정의와 가정에 기반하고 있다. 이들과의 비교를 통해 `mav_local_planner`가 해결하고자 하는 3차원 MAV 문제의 고유한 특성을 이해할 수 있다.

- **`DWA (Dynamic Window Approach)`:**
  - **핵심 원리:** DWA는 로봇의 현재 속도를 기준으로, 다음 시간 단계에 도달 가능한 속도 영역(dynamic window) 내에서 다수의 (선속도, 각속도) 쌍을 샘플링한다. 각 샘플링된 속도로 짧은 시간 동안 전진했을 때의 예상 궤적을 시뮬레이션하고, 목표 지향성, 장애물과의 거리, 전역 경로와의 일치도 등을 종합적으로 평가하는 비용 함수를 통해 최적의 속도 명령을 단 하나 선택하여 로봇에 전달한다.8
  - **개념적 차이:** DWA는 본질적으로 2차원 평면($SE(2)$)과 2차원 비용 지도(costmap)를 가정하고 동작한다.[8] 예측 호라이즌 동안 일정한 제어 입력을 가정하기 때문에 후진과 같은 복잡한 기동을 예측하기 어렵고, 차와 같은(car-like) 비-홀로노믹(non-holonomic) 로봇의 기구학적 제약을 완벽히 반영하기 어렵다.[19, 22] 반면, `mav_local_planner`는 완전한 3차원 공간($SE(3)$)과 ESDF를 사용하며, 속도 명령이 아닌 전체 궤적 자체를 다항식으로 최적화한다는 점에서 근본적인 차이를 보인다.
- **`TEB (Timed Elastic Band)`:**
  - **핵심 원리:** TEB는 전역 경로의 일부를 시간 정보가 포함된 탄성 밴드(elastic band)로 간주하고, 이 밴드를 구성하는 로봇의 시-공간 상의 자세(pose)들을 직접 최적화 변수로 삼는다. 궤적 실행 시간, 장애물과의 거리, 로봇의 최대 속도 및 가속도와 같은 동역학적 제약 등을 모두 포함하는 다중 목표 최적화 문제를 풀어 밴드의 모양을 실시간으로 변형시켜 최적의 지역 경로를 찾는다.7
  - **개념적 차이:** TEB는 DWA보다 더 복잡한 기동(예: 후진)을 계획할 수 있고, 차와 같은 로봇의 기구학도 잘 지원한다.22 그러나 TEB 역시 기본적으로는 2차원 `base_local_planner`의 플러그인으로 개발되어 2차원 비용 지도 위에서 동작한다.7 이는 3차원 ESDF를 사용하는 `mav_local_planner`와의 가장 큰 차이점이다. 또한, TEB는 장애물의 왼쪽과 오른쪽으로 가는 경로 등 위상학적으로 다른 여러 경로 후보(multiple topologies)를 동시에 최적화하여 지역 최솟값(local minima) 문제에 빠질 위험을 완화하려는 시도를 포함하고 있다.19 이는 단일 경로 후보만을 최적화하는 `mav_local_planner`의 접근 방식과 차별화되는 중요한 특징이다.

이 외에도 `eband_local_planner`는 TEB의 초기 개념과 유사한 탄성 밴드 아이디어를 사용하며 23, `adaptive_local_planner`는 인공적인 포텐셜 필드(potential field)를 이용해 경로를 생성한다.24 이들 모두 2차원 환경을 가정한다는 점에서 `mav_local_planner`와는 적용 영역이 명확히 구분된다.

### 3.2  3D MAV 로컬 플래너와의 성능 비교

`mav_local_planner`는 3차원 MAV 자율 비행 분야의 중요한 선구적 연구 중 하나이며, 그 이후 더 발전된 형태의 플래너들이 등장하여 활발히 연구되고 있다. 대표적으로 `Fast-Planner`와 `EGO-Planner`는 `mav_local_planner`의 철학을 계승하거나 극복하려는 시도 속에서 개발된 중요한 플래너들이다.

- **`Fast-Planner`:**
  - **핵심 원리:** `mav_local_planner`와 마찬가지로 ESDF 맵이 제공하는 풍부한 그래디언트 정보를 활용하는 기울기 기반(gradient-based) 최적화 방식을 사용한다. 가장 큰 차이점은 궤적 표현 방식으로 구간별 다항식 대신 B-스플라인(B-spline)을 채택한 것이다. B-스플라인은 제어점(control points)들이 궤적의 형태를 직관적으로 제어하며, 특히 '볼록 포락선(convex hull)' 특성 덕분에 궤적 전체가 항상 제어점들을 감싸는 볼록 다각형 내부에 존재함이 보장된다. 이 특성을 이용하면 궤적 전체에 대한 충돌 검사를 제어점들에 대한 검사로 근사할 수 있어 계산 효율성을 크게 높일 수 있다.25
  - **비교:** `Fast-Planner`는 B-스플라인의 수학적 장점을 적극적으로 활용하여 `mav_local_planner`의 다항식 기반 최적화보다 더 빠른 계산 속도와 높은 유연성을 목표로 한다. 문제 해결의 기본 패러다임(ESDF 활용, 기울기 기반 최적화)은 유사하지만, 궤적을 표현하고 다루는 수학적 도구에서 중요한 발전을 이루었다.
- **`EGO-Planner (ESDF-free Gradient-based Obstacle-avoidance Planner)`:**
  - **핵심 원리:** 이름에서 명확히 드러나듯이, ESDF 맵을 구축하고 유지하는 데 드는 막대한 계산 비용을 원천적으로 제거하는 것을 목표로 한다.25 EGO-Planner는 ESDF와 같은 사전 정보에 의존하는 대신, 현재 최적화 중인 궤적이 장애물과 충돌했을 때만 해당 충돌 정보를 센서의 원시 데이터(raw point cloud)로부터 직접 사용하여 궤적을 밀어내는 '반발력(gradient)'을 계산한다. 궤적이 자유 공간에 있을 때는 장애물에 대한 계산을 전혀 수행하지 않는다.27
  - **비교:** 이는 `mav_local_planner`의 패러다임을 근본적으로 뒤집는 혁신적인 접근법이다. `mav_local_planner`가 "먼저 정밀한 지도를 만들고, 그 위에서 계획한다"는 신중한 접근법이라면, `EGO-Planner`는 "일단 계획을 시도하고, 실패(충돌)하면 그 때 주변을 인지하여 수정한다"는 대담하고 반응적인 접근법에 가깝다. 이로 인해 `EGO-Planner`는 ESDF 구축 및 유지에 필요한 시간을 완전히 절약하여, 전체 계획 시간을 수 밀리초(ms) 단위로 획기적으로 단축시키는 극적인 성능 향상을 보인다.25 하지만 ESDF가 제공하는 풍부하고 전역적인 그래디언트 정보를 포기하기 때문에, 복잡한 'U'자형 함정(trap)과 같은 어려운 지역 최솟값(local minima)에 빠질 위험이 이론적으로 더 크다.25
- **`Histo-Planner`:**
  - **핵심 원리:** 원격 조종(teleoperation) 보조와 같이 극도로 계산 자원이 제한된 온보드 컴퓨터 환경을 위해 제안된 플래너이다. 이 플래너는 ESDF나 점유 격자 맵과 같은 명시적인 맵 구조를 전혀 사용하지 않고, 대신 센서 데이터로부터 실시간으로 생성된 '장애물 분포 히스토그램(histogram of obstacle distribution)'을 활용한다.29 이 히스토그램을 분석하여 MAV 주변에서 목표 방향으로 향하는 가장 넓고 안전한 '틈(gap)'을 찾고, 그곳에 '유도점(guidance point)'을 생성한다. 이 유도점을 통과하도록 초기 궤적을 설정함으로써 최적화기가 지역 최솟값에 빠지는 문제를 완화한다.30
  - **비교:** `EGO-Planner`보다 한 걸음 더 나아가 맵 표현을 극도로 경량화한 접근법이다. `mav_local_planner`의 정밀한 ESDF, `EGO-Planner`의 충돌점 기반 정보와 달리, `Histo-Planner`는 통계적인 장애물 분포 정보만을 사용하여 계산량을 최소화한다. 이는 궤적의 정밀도나 최적성보다는 실시간 반응성과 계산 효율성을 극대화하는 데 초점을 맞춘 설계 철학을 보여준다.

아래 표는 주요 MAV 로컬 플래너들의 핵심 특징을 비교하여 정리한 것이다.

| 플래너 (Planner)        | 핵심 원리 (Core Principle)  | 환경 표현 (Env. Representation) | 궤적 표현 (Traj. Representation)     | 최적화 방식 (Opt. Method)      | 주요 강점 (Key Strengths)                       | 주요 약점 (Key Weaknesses)                             | 목표 응용 분야 (Target Application)              |
| ----------------------- | --------------------------- | ------------------------------- | ------------------------------------ | ------------------------------ | ----------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------ |
| **`mav_local_planner`** | Plan-then-Optimize, LOCO    | ESDF (`voxblox`)                | 구간별 다항식 (Piecewise Polynomial) | QP 기반 최소 스냅/저크 최적화  | 풍부한 그래디언트 정보 활용, 부드러운 궤적 생성 | ESDF 구축/유지 비용, 초기 경로 의존성                  | 알려진 맵 또는 정적 환경에서의 정밀 비행         |
| **`Fast-Planner`**      | Gradient-based Optimization | ESDF                            | B-스플라인 (B-Spline)                | B-스플라인 제어점 최적화       | 빠른 최적화 속도, 볼록 포락선 특성 활용         | ESDF 구축/유지 비용, `mav_local_planner`와 유사한 한계 | 고속 자율 비행, 동적 환경 대응력 향상            |
| **`EGO-Planner`**       | ESDF-free, Collision-driven | 원시 포인트 클라우드 (충돌 시)  | B-스플라인 (B-Spline)                | 충돌 정보 기반 그래디언트 투영 | 극도로 빠른 계산 속도 (ESDF 불필요)             | 지역 최솟값에 취약, 전역적 그래디언트 정보 부재        | 계산 자원이 제한된 플랫폼, 고속 반응성 요구 환경 |
| **`Histo-Planner`**     | Histogram-based Guidance    | 장애물 분포 히스토그램          | B-스플라인 (B-Spline)                | 유도점 기반 초기화 + 최적화    | 매우 낮은 계산량, 맵 불필요                     | 정밀도 저하 가능성, 복잡한 지형 표현 한계              | 원격 조종 보조, 극히 제한된 온보드 컴퓨터        |
| **`TEB` (참고)**        | Timed Elastic Band          | 2D 비용맵 (Costmap)             | 시-공간 상의 자세 시퀀스             | 다중 목표 최적화 (G2O)         | 2D 지상 로봇 표준, 다중 토폴로지 탐색           | 3D MAV에 직접 적용 불가, 2D 환경 가정                  | 2D 지상 로봇(차동구동, 홀로노믹) 내비게이션      |

이 비교 분석을 통해 MAV 지역 경로 계획 기술의 중요한 발전 방향을 엿볼 수 있다. 특히 "ESDF 기반 대 ESDF-free"의 대립은 단순한 점진적 개선이 아닌 근본적인 패러다임의 전환을 의미한다. `mav_local_planner`와 `Fast-Planner`는 "더 많은 정보가 더 좋은 계획을 만든다"는 철학 아래, ESDF라는 정밀한 세계 모델을 구축하는 데 계산 자원을 투자한다.1 반면, `EGO-Planner`는 ESDF가 "상당한 중복성"을 가지며 궤적이 차지하는 "매우 제한된 부분 공간"에만 정보가 필요하다고 주장하며 이 가정에 정면으로 도전한다.26 즉, `mav_local_planner`가 신중하게 '선-모델링 후-계획' 방식을 취한다면, `EGO-Planner`는 효율성을 위해 '선-계획 후-충돌 시 대응'이라는 과감한 방식을 선택한다. 이 선택은 응용 분야에 따라 달라진다. 정밀성과 평활도가 중요한 임무(예: 항공 촬영, 시설 점검)에는 ESDF 기반 접근법이, 극한의 민첩성과 반응성이 요구되는 임무(예: 드론 레이싱, 재난 현장 탐색)에는 ESDF-free 접근법이 더 적합할 수 있다.

또한, 궤적 표현 방식이 다항식(`mav_local_planner`)에서 B-스플라인(`Fast-Planner`, `EGO-Planner`)으로 진화한 것은 중요한 기술적 발전이다. 다항식 스플라인은 구간 간의 연속성을 보장하기 위해 복잡한 선형 등식 제약 행렬을 구성해야 한다.13 반면, B-스플라인은 수학적으로 C^(k-2) 연속성이 내재되어 있고(k-1 차수일 때), 제어점을 변경해도 궤적의 일부만 국소적으로 변하며, 볼록 포락선 특성을 가지는 등 반복적인 최적화와 충돌 검사에 훨씬 유리한 특성을 지닌다.25 이처럼 더 효율적인 수학적 표현법의 채택이 `Fast-Planner`와 `EGO-Planner`의 성능 향상을 이끈 핵심 요인 중 하나라는 점은, 문제 해결에 있어 적절한 수학적 도구 선택의 중요성을 명백히 보여준다.

## 4.  동적 환경으로의 확장 및 최신 연구 동향

`mav_local_planner`와 같은 전통적인 플래너들은 주로 장애물이 움직이지 않는 정적(static) 또는 준-정적(quasi-static) 환경을 가정한다. 그러나 실제 세계는 움직이는 사람, 차량, 다른 로봇 등 예측 불가능한 동적 장애물로 가득 차 있다. 따라서 MAV 자율 비행 기술의 실용화를 위해서는 동적 환경에 강인하게 대응하는 능력이 필수적이다. 이와 관련된 연구는 예측 기반 접근법과 학습 기반 접근법이라는 두 가지 큰 흐름으로 발전하고 있다.

### 4.1  동적 장애물 회피 기법

동적 장애물을 회피하기 위한 연구는 장애물의 미래 움직임을 예측하여 계획에 반영하는 방식과, 현재 감지된 정보에 즉각적으로 반응하는 방식으로 나뉜다.

- **예측 기반 접근법 (Prediction-based Approaches):** 이 접근법의 핵심은 동적 장애물의 미래 궤적을 예측하고, 예측된 시-공간(spatio-temporal)상의 볼륨을 회피하도록 자신의 궤적을 계획하는 것이다.
  - **모델 예측 제어 (Model Predictive Control, MPC):** 이 분야에서 가장 지배적인 패러다임으로 자리 잡고 있다. MPC는 MAV 자신의 동역학 모델을 기반으로 미래 일정 시간(prediction horizon) 동안의 상태를 예측하고, 그 시간 동안 비용 함수(예: 목표 궤적 추종 오차, 제어 입력 사용량 최소화)를 최소화하는 최적의 제어 입력 시퀀스를 계산한다. 이 과정에서 장애물 회피, 모터 출력 한계 등의 제약 조건을 명시적으로 포함시킬 수 있다.32 동적 장애물은 시간에 따라 변하는 제약 조건으로 MPC 문제에 포함된다. 예를 들어, 특정 시각 $t$에 예측된 장애물의 위치를 중심으로 특정 반경 내에 MAV가 진입하지 않도록 하는 불등식 제약($||p_{mav}(t) - p_{obs}(t)|| \ge d_{safe}$)을 추가할 수 있다.34
  - **확률론적 접근법:** 예측에는 항상 불확실성이 따르므로, 이를 다루기 위한 확률론적 기법들이 MPC와 결합되고 있다. 대표적으로 **CCNMPC(Chance-Constrained Nonlinear MPC)**는 장애물과의 충돌 확률이 사용자가 지정한 작은 임계값(예: 1%) 이하가 되도록 제약 조건을 설정한다. 이를 통해 로봇 자신의 위치 추정 오차나 장애물 예측의 불확실성을 정량적으로 고려하여 더욱 안전한 경로를 계획할 수 있다.36
- **반응형 접근법 (Reactive Approaches):** 장애물의 미래를 명시적으로 예측하기보다는, 현재 감지된 장애물의 위치와 속도 정보에 즉각적으로 반응하는 방식이다.
  - **속도 장애물 (Velocity Obstacles, VO):** 다른 에이전트(장애물)와 미래에 충돌하게 될 자신의 속도 벡터들의 집합을 기하학적으로 계산하고, 그 집합에 속하지 않는 안전한 속도를 선택하는 기법이다. 본래 2차원 평면에서 제안되었으나, 3차원 동적 환경으로 확장된 연구들이 존재한다.37
  - **광학류 기반 (Optical Flow-based):** 단안 카메라에서 얻은 연속적인 이미지에서 픽셀들이 움직이는 패턴, 즉 광학류를 분석하여 장애물의 접근을 감지하고 회피 기동을 생성한다. 이 방법은 추가적인 거리 센서 없이 비전 정보만으로 동작할 수 있고 계산 부하가 적어 소형 MAV에 적합하지만, 3차원의 복잡한 환경에서 정밀한 회피를 수행하는 데는 한계가 있다.38

최근 연구들은 동적 장애물을 단순히 회피하는 것을 넘어, RGB-D 카메라와 칼만 필터(Kalman Filter)와 같은 추정 기법을 사용하여 각 장애물을 개별적으로 탐지하고 추적하며, 마르코프 연쇄(Markov chain) 모델 등으로 그들의 미래 궤적을 예측하여 플래너에 정보를 제공하는 통합 시스템으로 발전하고 있다.39

### 4.2  학습 기반 궤적 생성 및 탐색

전통적인 최적화 기반의 경로 계획은 정교한 수학적 모델링을 요구하며, 복잡한 환경에서 지역 최솟값에 빠지기 쉽고, 모델링되지 않은 예기치 않은 상황에 대한 강인성이 부족할 수 있다. 이러한 한계를 극복하기 위한 대안으로, 데이터로부터 직접 최적의 행동을 학습하는 심층 강화학습(Deep Reinforcement Learning, DRL)이 MAV 자율 비행 분야에서 큰 주목을 받고 있다.40

- **강화학습 (Reinforcement Learning, RL) 기반 접근법:**
  - **핵심 원리:** 에이전트(MAV)가 환경과의 수많은 시행착오(trial-and-error)를 통해 누적 보상(cumulative reward)을 최대화하는 최적의 정책(policy)을 스스로 학습한다.43 정책은 특정 상태(state)에서 어떤 행동(action)을 취해야 하는지를 결정하는 함수(예: 신경망)이다. 예를 들어, 상태는 카메라 이미지와 같은 센서 데이터가 될 수 있고, 행동은 MAV의 다음 속도 명령이 될 수 있다.
  - **장점:** 강화학습은 명시적인 맵핑이나 경로 계획, 최적화 단계를 거치지 않고, 센서 입력으로부터 직접 제어 명령을 출력하는 종단간(end-to-end) 제어 정책을 학습할 수 있다.45 이를 통해 인간 전문가가 수동으로 설계하기 매우 어려운 복잡하고 동적인 환경에서의 최적의 비행 패턴을 데이터로부터 발견할 수 있는 강력한 잠재력을 가진다.
  - **MAV 적용 사례:** 강화학습은 MAV의 특정 곡예비행47, 공격적인 고속 비행48, 움직이는 목표물 포획49, 그리고 GPS 신호가 없는 복잡한 실내 환경 항법45 등 다양한 고난도 임무에 성공적으로 적용되며 그 가능성을 입증하고 있다.
- **전통적 방법과의 결합:** 순수한 강화학습 접근법은 수많은 시행착오를 필요로 하여 학습 시간이 길고(sample inefficiency), 학습된 정책의 안전성을 보장하기 어렵다는 근본적인 한계를 가진다. 따라서 최근 연구는 전통적인 궤적 최적화 방법과 강화학습을 결합하여 각자의 장점을 취하는 하이브리드 방향으로 활발히 진행되고 있다.
  - **궤적 최적화를 통한 RL 학습 보조:** RRT*와 같은 샘플링 기반 경로 계획 알고리즘으로 생성된 양질의 참조 경로를 제공하여 강화학습 에이전트의 탐색 공간을 의미 있는 영역으로 제한하고, 이를 통해 학습 과정을 안정화하고 가속화할 수 있다.52
  - **RL을 통한 최적화 문제 해결:** 전통적인 수치 최적화 방법은 좋은 초기해를 찾기 어렵고 계산 시간이 많이 걸릴 수 있다. 생성 모델(generative model)이나 강화학습을 사용하여 최적화 문제의 좋은 초기해(warm start)를 빠르게 생성하거나, 심지어 최적화 과정 자체를 학습하여 계산 효율성을 높이려는 시도가 있다.43

최신 연구 동향은 여기서 한 걸음 더 나아가, 대형 언어 모델(Large Language Models, LLM) 및 비전-언어 모델(Vision-Language Models, VLM)을 활용하여 "2층 부엌으로 가서 테이블 위의 사과를 찾아오라"와 같은 고수준의 자연어 명령을 이해하고, 임무 수행 중 예상치 못한 문제(예: 닫힌 문)에 직면했을 때 상황을 인지하고 실시간으로 계획을 수정(replanning)하는 연구로 확장되고 있다.54

이러한 연구 동향은 MAV 자율 비행 기술의 패러다임이 변화하고 있음을 시사한다. `mav_local_planner`와 같은 정적 세계 기반 플래너의 한계는 명확해졌으며, 연구의 최전선은 동적 환경과 불확실성을 다루는 방향으로 결정적으로 이동했다. 이 새로운 전선에서는 모델 기반의 예측적 제어(MPC)와 데이터 기반의 학습적 제어(RL)가 양대 산맥을 이루고 있다. MPC는 예측 가능성과 제약 조건 처리 능력에서 강점을 보이지만 정확한 시스템 모델을 요구하며, RL은 명시적인 모델 없이 복잡한 행동을 학습할 수 있지만 안전성 보장과 학습 효율성에서 어려움을 겪는다.35 미래의 가장 유망한 방향은 이 둘을 결합한 하이브리드 시스템, 예를 들어 RL을 사용하여 MPC의 비용 함수나 예측 모델의 오차를 학습하여 보정하는 방식이 될 것이다.56

더 나아가, LLM/VLM의 등장은 자율성의 문제를 "어떻게 움직일 것인가(How to Move)"에서 "무엇을 해야 하는가(What to Do)"로 격상시키고 있다. 이는 기하학적 목표점을 다루는 `mav_local_planner`를 넘어, 고수준의 의미론적 목표를 이해하고 추론하며 계획하는 "추론 후 계획(Reason-then-Plan)" 패러다임의 부상이다.54 이러한 미래의 계층적 자율성 아키텍처에서 `mav_local_planner`와 같은 저수준 모듈은 LLM이라는 고수준 지휘관의 명령을 받아 안정적으로 궤적을 실행하는 '운동 지능'의 핵심 요소로서 그 역할을 계속 수행할 수 있을 것이다.

## 5.  결론 및 향후 전망

### 5.1  `mav_local_planner`의 종합 평가

`mav_local_planner`는 3차원 환경에서의 MAV 자율 비행을 위한 견고하고 이론적으로 잘 정립된 지역 경로 계획기이다. 이 플래너는 ESDF를 통한 풍부하고 정밀한 환경 정보와 최소 스냅 기반의 다항식 궤적 최적화 기법을 효과적으로 결합하여, 동역학적으로 실행 가능하면서도 매우 부드러운 고품질의 궤적을 생성하는 데 탁월한 강점을 보인다.

- **장점:**
  - **고품질 궤적 생성:** 미분적 평탄성 이론과 최소 스냅 최적화 원칙에 기반하여, MAV의 움직임을 물리적으로 부드럽고 에너지 효율적으로 만드는 평활도가 높은 궤적을 보장한다.
  - **견고한 이론적 기반:** ESDF, 이차 계획법(QP) 최적화 등 수학적으로 잘 정립되고 검증된 기법들을 기반으로 하여, 플래너의 동작이 예측 가능하고 안정적이다.
  - **유연한 아키텍처:** 전역 플래너와의 손쉬운 연동, 다양한 궤적 평활화기 선택 기능 등 모듈식 설계를 통해 사용자가 특정 응용에 맞게 시스템을 구성할 수 있는 유연성을 제공한다.
- **단점:**
  - **높은 계산 복잡성:** ESDF 맵을 실시간으로 구축하고 유지하는 과정은 상당한 계산 자원을 소모하므로, 온보드 컴퓨터의 사양이 제한적인 소형 MAV에 적용하기에는 부담이 될 수 있다.
  - **정적 환경 가정:** 플래너의 핵심 설계는 장애물이 움직이지 않는 정적 환경을 가정하므로, 동적 장애물에 대한 명시적인 처리 메커니즘이 부재하여 실제 복잡한 환경에서의 적용에 한계가 있다.
  - **초기 경로 의존성:** "Plan-then-Optimize" 패러다임으로 인해, 초기 전역 경로의 품질이 최종 궤적의 성능을 크게 좌우한다. 초기 경로가 좋지 않으면 지역 최적화만으로는 한계를 극복하기 어렵다.

결론적으로, `mav_local_planner`는 계산 자원이 비교적 풍부하고, 환경이 대부분 정적이거나 사전에 정밀하게 맵핑된 시나리오에서 매우 효과적인 솔루션이다. 예를 들어, 산업 시설의 정밀 구조 점검, 대형 창고 내의 자동화된 물류 운송, 계획된 경로를 따라 비행하는 항공 촬영 등의 응용 분야에서 그 강점을 최대한 발휘할 수 있다.

### 5.2  MAV 자율 비행 기술의 발전 방향 제언

`mav_local_planner`가 제시한 견고한 기반 위에서, 미래의 MAV 자율 비행 기술은 계산 효율성, 동적 환경 대응 능력, 그리고 지능적 의사결정 능력이라는 세 가지 핵심 축을 중심으로 더욱 발전할 것으로 전망된다.

- **하이브리드 플래닝 (Hybrid Planning):** `mav_local_planner`가 보여준 ESDF 기반의 정밀한 최적화 능력과 `EGO-Planner`와 같은 최신 플래너의 빠른 반응성을 결합하는 하이브리드 접근법이 매우 유망하다. 예를 들어, MAV가 넓고 개방된 공간을 비행할 때는 ESDF-free 방식으로 빠르게 이동하다가, 좁은 통로를 통과하거나 정밀한 착륙 기동이 필요한 특정 구간에 진입하면 국소적으로 ESDF를 생성하여 정밀 최적화를 수행하는 방식으로 동적으로 계획 전략을 전환할 수 있다. 이는 상황에 따라 효율성과 정밀성 사이의 균형을 최적으로 조절하는 지능적인 플래너로 이어질 것이다.

- **학습과 모델의 융합 (Fusion of Learning and Model-based Methods):** 순수 강화학습의 안전성 및 데이터 효율성 문제와, 순수 모델 기반 방법의 예측 불가능한 교란에 대한 강인성 문제를 해결하기 위해 두 가지 패러다임을 융합하는 것이 필수적이다. 강화학습을 사용하여 모델 예측 제어(MPC)의 복잡한 비용 함수 가중치를 동적으로 조절하거나, 예측하기 어려운 공기역학적 효과나 외부 바람과 같은 교란을 데이터로부터 학습하여 MPC의 예측 모델을 실시간으로 보정하는 연구가 더욱 중요해질 것이다.56 이는 모델의 정확성에 의존하면서도 학습을 통해 현실 세계의 불확실성에 적응하는, 더욱 강인하고 신뢰성 있는 제어 시스템을 가능하게 할 것이다.

- **계층적/의미론적 자율성 (Hierarchical/Semantic Autonomy):** 미래의 MAV는 단순히 'A 지점에서 B 지점으로 이동하라'는 기하학적 명령을 넘어, "2번 선반에서 빨간색 상자를 찾아 사진을 찍어라"와 같은 고수준의 의미론적 임무를 이해하고 수행해야 한다. 이를 위해서는 대형 언어 모델(LLM)이나 비전-언어 모델(VLM)을 이용한 고수준의 의미론적 이해 및 과업 계획 능력과, `mav_local_planner`와 같은 저수준의 견고한 궤적 생성 및 실행 모듈을 결합한 계층적 아키텍처가 핵심이 될 것이다.54 이러한 미래 시스템에서 

  `mav_local_planner`와 그 후속 기술들은, 복잡한 추론과 계획의 최종 결과를 물리 세계에서 신뢰성 있게 구현하는 '운동 지능(motor intelligence)'의 핵심적인 역할을 계속해서 수행하며 MAV 자율성의 발전에 기여할 것이다.

#### 참고자료

1. ethz-asl/mav_voxblox_planning: MAV planning tools using voxblox as the map representation. - GitHub, accessed August 8, 2025, https://github.com/ethz-asl/mav_voxblox_planning
2. dashenyjn123/mav_voxblox_planning - Gitee, accessed August 8, 2025, https://gitee.com/dashenyjn123/mav_voxblox_planning
3. [1611.03631] Voxblox: Incremental 3D Euclidean Signed Distance Fields for On-Board MAV Planning - arXiv, accessed August 8, 2025, https://arxiv.org/abs/1611.03631
4. mav_voxblox_planning/mav_local_planner/include/mav_local_planner/mav_local_planner.h at master / ethz-asl/mav_voxblox_planning - GitHub, accessed August 8, 2025, https://github.com/ethz-asl/mav_voxblox_planning/blob/master/mav_local_planner/include/mav_local_planner/mav_local_planner.h
5. ethz_asl_UAV_autonomous/README.md at master - GitHub, accessed August 8, 2025, https://github.com/TJUUAVLaboratory/ethz_asl_UAV_autonomous/blob/master/README.md
6. ROS Package | Aerial Robotics IITK, accessed August 8, 2025, https://wiki.ariitk.in/wiki/tutorials/how-to-write-a-ros-package/ros-package
7. teb_local_planner - ROS Wiki, accessed August 8, 2025, http://wiki.ros.org/teb_local_planner
8. dwa_local_planner - ROS Wiki, accessed August 8, 2025, http://wiki.ros.org/dwa_local_planner
9. Polynomial Trajectory Planning for Aggressive Quadrotor Flight in Dense Indoor Environments - Research, accessed August 8, 2025, https://groups.csail.mit.edu/rrg/papers/Richter_ISRR13.pdf
10. Polynomial Trajectory Planning for Quadrotor Flight - Autonomous Robots Lab, accessed August 8, 2025, http://www.autonomousrobotslab.com/uploads/5/8/4/4/58449511/badgerworks_polynomialtrajectoryplanningquadrotorflight.pdf
11. ethz-asl/mav_trajectory_generation: Polynomial trajectory generation and optimization, especially for rotary-wing MAVs. - GitHub, accessed August 8, 2025, https://github.com/ethz-asl/mav_trajectory_generation
12. Minimum Snap Trajectory Planning and Augmented MPC for Morphing Quadrotor Navigation in Confined Spaces - MDPI, accessed August 8, 2025, https://www.mdpi.com/2504-446X/9/4/304
13. MIT Open Access Articles Polynomial Trajectory Planning for Aggressive Quadrotor Flight in Dense Indoor Environments, accessed August 8, 2025, [https://dspace.mit.edu/bitstream/handle/1721.1/106840/Roy_Polynomial%20trajectory.pdf?sequence=1&isAllowed=y](https://dspace.mit.edu/bitstream/handle/1721.1/106840/Roy_Polynomial trajectory.pdf?sequence=1&isAllowed=y)
14. Lecture 10: Trajectory Optimization II 10.1 QP-based ... - VNAV, accessed August 8, 2025, https://vnav.mit.edu/material/10-TrajectoryOptimization2-notes.pdf
15. Lecture 9: Trajectory Optimization I - VNAV, accessed August 8, 2025, https://vnav.mit.edu/material/09-TrajectoryOptimization1-notes.pdf
16. (PDF) Quadrotor Path Planning and Polynomial Trajectory Generation Using Quadratic Programming for Indoor Environments - ResearchGate, accessed August 8, 2025, https://www.researchgate.net/publication/368445707_Quadrotor_Path_Planning_and_Polynomial_Trajectory_Generation_Using_Quadratic_Programming_for_Indoor_Environments
17. Quadrotor Path Planning and Polynomial Trajectory Generation Using Quadratic Programming for Indoor Environments - MDPI, accessed August 8, 2025, https://www.mdpi.com/2504-446X/7/2/122
18. 1.1 Matlab Based LOCO - SciSpace, accessed August 8, 2025, https://scispace.com/pdf/matlab-based-loco-5710gghjj4.pdf
19. Difference between DWA local_planner and TEB local_planner - Robotics Stack Exchange, accessed August 8, 2025, https://robotics.stackexchange.com/questions/83618/difference-between-dwa-local-planner-and-teb-local-planner
20. need some help with minimum snap trajectory generation : r/ControlTheory - Reddit, accessed August 8, 2025, https://www.reddit.com/r/ControlTheory/comments/1207q69/need_some_help_with_minimum_snap_trajectory/
21. Holonomic motion planning - Aditya Kamath, accessed August 8, 2025, https://adityakamath.github.io/2021-10-17-holonomic-motion-planning/
22. Difference Between DWA and TEB Local Planners | PDF | Mathematical Optimization | Computing - Scribd, accessed August 8, 2025, https://www.scribd.com/document/581185898/Difference-between-DWA-and-TEB-local-planners
23. eband_local_planner - ROS Wiki, accessed August 8, 2025, http://wiki.ros.org/eband_local_planner
24. adaptive_local_planner - ROS Wiki, accessed August 8, 2025, http://wiki.ros.org/adaptive_local_planner
25. EGO-Planner: An ESDF-free Gradient-based Local Planner for Quadrotors - ResearchGate, accessed August 8, 2025, https://www.researchgate.net/publication/348028324_EGO-Planner_An_ESDF-free_Gradient-based_Local_Planner_for_Quadrotors
26. ZJU-FAST-Lab/ego-planner - GitHub, accessed August 8, 2025, https://github.com/ZJU-FAST-Lab/ego-planner
27. [2008.08835] EGO-Planner: An ESDF-free Gradient-based Local Planner for Quadrotors, accessed August 8, 2025, https://arxiv.org/abs/2008.08835
28. EGO-Planner: An ESDF-free Gradient-based Local Planner for Quadrotors - YouTube, accessed August 8, 2025, https://www.youtube.com/watch?v=UKoaGW7t7Dk
29. Histo-Planner: A Real-time Local Planner for MAVs Teleoperation based on Histogram of Obstacle Distribution - ResearchGate, accessed August 8, 2025, https://www.researchgate.net/publication/391953591_Histo-Planner_A_Real-time_Local_Planner_for_MAVs_Teleoperation_based_on_Histogram_of_Obstacle_Distribution
30. A Real-time Local Planner for MAVs Teleoperation based on Histogram of Obstacle Distribution - arXiv, accessed August 8, 2025, https://arxiv.org/html/2505.15043v1
31. Real-Time Trajectory Replanning for MAVs using Uniform B-splines and a 3D Circular Buffer - Computer Vision Group, accessed August 8, 2025, https://cvg.cit.tum.de/_media/spezial/bib/usenko17replanning.pdf
32. Obstacle Avoidance for an Unmanned Aerial Vehicle Model using Model Predictive Control - eucass, accessed August 8, 2025, https://www.eucass.eu/component/docindexer/?task=download&id=2818
33. Neural Network Based Model Predictive Control for a Quadrotor UAV - MDPI, accessed August 8, 2025, https://www.mdpi.com/2226-4310/9/8/460
34. Nonlinear Model Predictive Control for Multi-Micro Aerial Vehicle Robust Collision Avoidance - ResearchGate, accessed August 8, 2025, https://www.researchgate.net/publication/314237612_Nonlinear_Model_Predictive_Control_for_Multi-Micro_Aerial_Vehicle_Robust_Collision_Avoidance
35. Robust Collision Avoidance for Multiple Micro Aerial Vehicles Using Nonlinear Model Predictive Control - People | MIT CSAIL, accessed August 8, 2025, https://people.csail.mit.edu/jalonsom/docs/17-kamel-IROS.pdf
36. Chance-Constrained Collision Avoidance for MAVs in Dynamic Environments - TU Delft Research Portal, accessed August 8, 2025, https://research.tudelft.nl/files/101618400/Chance_Constrained_Collision_Avoidance_for_MAVs_in_Dynamic_Environments.pdf
37. Chance-Constrained Collision Avoidance for MAVs in Dynamic Environments, accessed August 8, 2025, https://www.researchgate.net/publication/330432832_Chance-Constrained_Collision_Avoidance_for_MAVs_in_Dynamic_Environments
38. Vision-Based Obstacle Avoidance Strategies for MAVs Using Optical Flows in 3-D Textured Environments, accessed August 8, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC6604071/
39. [2209.08258] A real-time dynamic obstacle tracking and mapping system for UAV navigation and collision avoidance with an RGB-D camera - arXiv, accessed August 8, 2025, https://arxiv.org/abs/2209.08258
40. Aalborg Universitet A Review on Reinforcement Learning for Motion Planning of Robotic Manipulators Elguea-Aguinaco, Íñigo ; In, accessed August 8, 2025, https://vbn.aau.dk/files/761881616/International_Journal_of_Intelligent_Systems_-_2024_-_Elguea-Aguinaco_-_A_Review_on_Reinforcement_Learning_for_Motion.pdf
41. A Comprehensive Review of Deep Learning Techniques in Mobile Robot Path Planning: Categorization and Analysis - MDPI, accessed August 8, 2025, https://www.mdpi.com/2076-3417/15/4/2179
42. A Review of Deep Reinforcement Learning Algorithms for Mobile Robot Path Planning, accessed August 8, 2025, https://www.mdpi.com/2624-8921/5/4/78
43. Local Policy Optimization for Trajectory-Centric Reinforcement Learning - arXiv, accessed August 8, 2025, http://arxiv.org/pdf/2001.08092
44. Deep Reinforcement Learning Based Trajectory Planning Under Uncertain Constraints - Frontiers, accessed August 8, 2025, https://www.frontiersin.org/journals/neurorobotics/articles/10.3389/fnbot.2022.883562/full
45. Deep RL-based Autonomous Navigation of Micro Aerial Vehicles (MAVs) in a complex GPS-denied Indoor Environment - arXiv, accessed August 8, 2025, https://arxiv.org/html/2504.05918v1
46. Multi-UAV simultaneous target assignment and path planning based on deep reinforcement learning in dynamic multiple obstacles environments - Frontiers, accessed August 8, 2025, https://www.frontiersin.org/journals/neurorobotics/articles/10.3389/fnbot.2023.1302898/full
47. Learning-based Trajectory Tracking for Bird-inspired Flapping-Wing Robots, accessed August 8, 2025, https://hybrid-robotics.berkeley.edu/publications/ACC2025_Learning_Flapping_UAV.pdf
48. Vision-Based Learning for Drones: A Survey - arXiv, accessed August 8, 2025, https://arxiv.org/html/2312.05019v2
49. Non-Equilibrium MAV-Capture-MAV via Time-Optimal Planning and Reinforcement Learning - arXiv, accessed August 8, 2025, https://arxiv.org/html/2503.06578v1
50. [2503.06578] Non-Equilibrium MAV-Capture-MAV via Time-Optimal Planning and Reinforcement Learning - arXiv, accessed August 8, 2025, https://arxiv.org/abs/2503.06578
51. [Literature Review] Deep RL-based Autonomous Navigation of Micro Aerial Vehicles (MAVs) in a complex GPS-denied Indoor Environment - Moonlight, accessed August 8, 2025, https://www.themoonlight.io/en/review/deep-rl-based-autonomous-navigation-of-micro-aerial-vehicles-mavs-in-a-complex-gps-denied-indoor-environment
52. [1903.05751] Trajectory Optimization for Unknown Constrained Systems using Reinforcement Learning - arXiv, accessed August 8, 2025, https://arxiv.org/abs/1903.05751
53. Efficient and Guaranteed-Safe Non-Convex Trajectory Optimization with Constrained Diffusion Model - arXiv, accessed August 8, 2025, https://arxiv.org/html/2403.05571v1
54. Robotic Replanning with Perception and Language Models - arXiv, accessed August 8, 2025, https://arxiv.org/html/2401.04157v1
55. ICRA 2025 Program | Thursday May 22, 2025, accessed August 8, 2025, https://ras.papercept.net/conferences/conferences/ICRA25/program/ICRA25_ContentListWeb_3.html
56. A Survey on Learning-Based Model Predictive Control: Toward Path Tracking Control of Mobile Platforms - MDPI, accessed August 8, 2025, https://www.mdpi.com/2076-3417/12/4/1995
57. Learning-Based vs Model-Free Adaptive Control of a MAV Under Wind Gust - ResearchGate, accessed August 8, 2025, https://www.researchgate.net/publication/357512127_Learning-Based_vs_Model-Free_Adaptive_Control_of_a_MAV_Under_Wind_Gust
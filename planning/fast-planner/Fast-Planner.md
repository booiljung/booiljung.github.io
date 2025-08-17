# Fast-Planner

## 서론: Fast-Planner의 의의와 핵심 기여

### 0.1 배경: 미지 환경에서의 고속 자율 비행의 난제

쿼드로터와 같은 무인 항공기(UAV)가 미지의 복잡한 환경에서 고속으로 비행하는 기술은 현대 로보틱스 분야의 가장 도전적인 과제 중 하나로 남아있다. 이 문제의 핵심에는 실시간 경로 재계획(real-time replanning)의 계산 효율성, 기체의 동역학적 한계(kinodynamic constraints)를 만족하는 비행 가능성, 그리고 장애물과의 충돌을 회피하는 안전성이라는 세 가지 요소 간의 근본적인 상충 관계(trade-off)가 존재한다.1 기존의 경로 계획 방법론들은 이러한 요소들을 동시에, 특히 제한된 온보드 컴퓨팅 자원 내에서 강건하게(robustly) 보장하는 데 명백한 한계를 보여왔다.1 예를 들어, 일부 방법은 안전성을 확보하기 위해 과도하게 보수적인 경로를 생성하여 기체의 민첩성을 희생시키거나, 복잡한 최적화 문제로 인해 실시간성을 만족시키지 못하는 경우가 빈번했다.

### 0.2 Fast-Planner의 등장과 목표

이러한 난제를 해결하기 위해 홍콩과기대(HKUST) Aerial Robotics Group과 저장대학(ZJU) FAST Lab의 연구팀은 `Fast-Planner`라는 혁신적인 궤적 계획 시스템을 개발했다.3 Fast-Planner의 핵심 목표는 명확하다: 복잡하고 알려지지 않은 3D 환경에서 쿼드로터의 빠르고(fast) 안전한(safe) 자율 비행을 가능하게 하는 강건하고(robust) 계산적으로 효율적인(computationally efficient) 통합 프레임워크를 제공하는 것이다.1 이는 단순히 하나의 알고리즘을 제안하는 것을 넘어, 환경 인식, 초기 경로 탐색, 그리고 궤적 최적화에 이르는 전 과정을 유기적으로 결합한 완전한 시스템을 지향한다.

### 0.3 핵심 기여 및 학술적 유산

Fast-Planner의 가장 중요한 기여 중 하나는 그것이 단일 프로젝트의 성공을 넘어, 자율 비행 연구 생태계 전반에 미친 지대한 영향력이다. 이 프로젝트는 `EGO-Planner`, `FUEL`, `RACER`와 같이 이후에 등장한 다수의 저명한 오픈소스 드론 프로젝트들의 기반 코드 프레임워크와 핵심 알고리즘을 제공하는 역할을 수행했다.3 이는 Fast-Planner의 소프트웨어 아키텍처가 높은 수준의 모듈성, 확장성, 그리고 실용성을 갖추었음을 명백히 방증한다.

다른 연구 그룹들이 완전히 새로운 시스템을 구축하는 대신 Fast-Planner를 기반으로 자신들의 연구를 확장할 수 있었다는 사실은, Fast-Planner가 환경 맵핑(ESDF), 안정적인 최적화 백엔드, 그리고 잘 정의된 ROS 인터페이스와 같은 공통적이고 어려운 문제들을 매우 효과적으로 해결했음을 의미한다. 예를 들어, EGO-Planner는 Fast-Planner의 핵심 구조에서 ESDF 모듈만을 제거하고 대체하는 방식으로 개발될 수 있었는데, 이는 각 구성 요소가 얼마나 잘 분리되어 설계되었는지를 보여주는 단적인 예이다. 이처럼 Fast-Planner는 새로운 알고리즘 개발의 진입 장벽을 낮추고 연구자들이 자신의 핵심 아이디어에 집중할 수 있는 환경을 조성함으로써, 커뮤니티 전반의 연구를 가속하는 '생태계 조력자(Ecosystem Enabler)'로서의 유산을 남겼다.

## 1.  Fast-Planner의 아키텍처 및 핵심 구성 요소

### 1.1  계층적 계획 파이프라인 (Hierarchical Planning Pipeline)

Fast-Planner는 문제의 복잡성을 효과적으로 관리하기 위해 명확한 2단계 계층적 계획 파이프라인을 채택한다. 이 구조는 거친 전역 탐색과 정교한 지역 최적화를 분리하여, 계산 효율성과 최종 궤적의 품질 사이에서 균형을 맞추는 검증된 전략이다.

- **프론트엔드 (Frontend):** 초기 경로 탐색 단계로, 주어진 시작점과 목표점 사이에서 동역학적 제약을 고려하여 안전하고 대략적으로 최적인 경로를 신속하게 찾아내는 것을 목표로 한다.1 이 단계의 핵심은 전체 탐색 공간을 줄여 해(solution)가 존재할 가능성이 높은 영역을 빠르게 식별하는 것이다. 계산 비용이 높은 최적화를 수행하기 전에 좋은 초기값을 제공하는 중요한 역할을 한다.
- **백엔드 (Backend):** 궤적 최적화 단계로, 프론트엔드에서 생성된 거친 초기 경로를 입력받아 최종적으로 실행될 궤적을 다듬는다. 이 단계에서는 B-스플라인(B-spline)이라는 연속적인 곡선 표현을 사용하며, 경사도 기반 최적화(Gradient-based Optimization)를 통해 궤적을 더욱 매끄럽고(smooth), 안전하며(safe), 동역학적으로 실현 가능하게(dynamically feasible) 만든다.2

### 1.2  코드베이스 구조 분석 (GitHub Repository)

Fast-Planner의 강점은 이러한 계층적 파이프라인이 `fast_planner`라는 단일 ROS 패키지 내에서 모듈화된 구조로 명확하게 구현되어 있다는 점이다. 각 모듈은 독립적인 역할을 수행하면서도 유기적으로 결합하여 전체 시스템을 구성한다.3

- **`plan_env` (환경 인식 및 표현):** 이 모듈은 플래너의 '눈' 역할을 한다. 깊이 센서(Depth Camera)나 라이다(LiDAR)로부터 입력된 포인트 클라우드 데이터와 드론의 위치 추정 정보(Odometry)를 융합하여 주변 환경의 3D 맵을 실시간으로 구축하고 관리한다. 이 모듈의 가장 중요한 출력물은 **유클리드 부호 거리 필드(ESDF, Euclidean Signed Distance Field)**이다. ESDF는 맵 상의 모든 지점에서 가장 가까운 장애물까지의 부호 있는 거리 값을 저장하는 자료구조로, 궤적이 장애물로부터 얼마나 떨어져 있는지와 어느 방향으로 밀어내야 하는지(그래디언트)에 대한 정보를 매우 효율적으로 제공하여 백엔드 최적화 과정에서 핵심적인 역할을 한다.1
- **`path_searching` (프론트엔드 경로 탐색):** 프론트엔드 알고리즘들이 구현된 모듈이다. 쿼드로터의 속도와 가속도 등 동역학을 직접적으로 고려하는 **키노다이내믹 경로 탐색(Kinodynamic Path Searching)** 알고리즘을 포함하여, 동역학적으로 실현 가능한 초기 경로를 생성한다. 또한, 복잡한 환경에서 단순히 하나의 경로가 아닌, 위상학적으로 구별되는 여러 경로(예: 장애물의 왼쪽 길과 오른쪽 길)를 탐색하는 **위상 경로 탐색(Topological Path Searching)** 알고리즘도 포함하여 최적화의 성공률과 궤적의 질을 높인다.3
- **`bspline` (궤적 표현):** 궤적을 수학적으로 표현하고 관리하는 모듈이다. Fast-Planner는 균일 B-스플라인(Uniform B-spline)을 사용하여 궤적을 표현한다. B-스플라인은 소수의 제어점(control points)에 의해 전체 곡선이 정의되며, 제어점의 위치만 조정하면 곡선 전체의 매끄러움(높은 차수의 연속성)이 보장된다. 또한, 하나의 제어점을 수정했을 때 곡선의 일부에만 영향을 미치는 지역적 수정 용이성(locality) 덕분에 실시간 재계획에 매우 적합하다.2
- **`bspline_opt` (백엔드 궤적 최적화):** B-스플라인으로 표현된 궤적을 실제로 최적화하는 핵심 엔진이다. 이 모듈은 `plan_env`로부터 ESDF 정보를 받아 충돌 비용을 계산하고, `bspline`의 제어점들을 조정하여 매끄러움과 동적 제약을 만족시키는 최적의 궤적을 찾는다. 이 비선형 최적화 문제를 풀기 위해 외부 라이브러리인 **NLopt**를 활용한다.3
- **`plan_manage` (고수준 계획 관리자):** 전체 계획 프로세스를 조율하고 스케줄링하는 지휘 본부 역할을 한다. 외부(예: 사용자가 Rviz에서 '2D Nav Goal'로 목표점을 지정)로부터 명령을 받으면, `plan_env`에 맵 업데이트를 요청하고, `path_searching`을 호출하여 초기 경로를 생성한 뒤, `bspline_opt`를 통해 최종 궤적을 얻어낸다. 이 일련의 과정을 상태 머신(State Machine)으로 관리하며, 각 모듈이 적시에 올바르게 작동하도록 제어한다.3
- **`active_perception` (지각 인식 계획):** 단순히 주어진 맵에서 장애물을 피하는 수동적인 계획을 넘어, 불확실성을 줄이기 위해 능동적으로 환경을 관찰하고 탐사하는 고급 전략을 구현하기 위한 모듈이다. 이 기능은 `RAPTOR` 논문과 관련이 있으며, 향후 확장을 위해 예비된 구조이다.3

아래 표는 각 모듈의 역할과 상호작용을 요약하여 보여준다.

| 모듈 (디렉토리)     | 핵심 역할 (Primary Role)      | 주요 입력 (Key Inputs)                             | 주요 출력 (Key Outputs)                         | 관련 기술/클래스                                             |
| ------------------- | ----------------------------- | -------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| `plan_env`          | 온라인 3D 맵핑 및 ESDF 생성   | 깊이 이미지/포인트 클라우드, 카메라 포즈(Odometry) | 확률적 체적 맵 (Probabilistic Voxel Map), ESDF  | Raycasting, Voxel Hashing, `EDFMap`                          |
| `path_searching`    | 프론트엔드 초기 경로 탐색     | 시작/목표 상태, ESDF 맵                            | 안전하고 동역학적으로 실행 가능한 초기 경로(들) | Hybrid A-star, Kinodynamic Search, Topological Path Searching |
| `bspline`           | B-스플라인 궤적 표현          | 제어점(Control Points), 매듭 벡터(Knot Vector)     | 연속적인 궤적(위치, 속도, 가속도)               | `UniformBspline`                                             |
| `bspline_opt`       | 백엔드 B-스플라인 궤적 최적화 | 초기 경로, ESDF 맵, 동적 제약                      | 최적화된 B-스플라인 제어점                      | Gradient-based Optimization, NLopt, Convex Hull Property     |
| `plan_manage`       | 고수준 계획 관리 및 스케줄링  | 사용자 목표 지점(Goal), 현재 상태(State)           | 실행할 궤적 명령(Trajectory Command)            | State Machine, ROS-interfaces, `kino_replan.launch`          |
| `active_perception` | (예정) 능동적 지각 및 탐사    | 맵의 불확실성 정보                                 | 최적의 관측 지점/경로                           | (RAPTOR 논문 관련)                                           |

## 2.  핵심 알고리즘 심층 분석: 이론과 실제

### 2.1  프론트엔드: Kinodynamic Path Searching

Fast-Planner의 프론트엔드는 단순히 기하학적인 최단 경로를 찾는 것을 넘어, 쿼드로터의 동역학적 제약(kinodynamic constraints)을 만족하는 실행 가능한 초기 경로를 찾는 것을 목표로 한다. 이를 위해 이산화된 제어 공간(discretized control space)에서 최소 시간(minimum-time) 궤적을 찾는 휴리스틱 탐색 알고리즘을 사용한다.1

#### 2.1.1 이론적 기반: 하이브리드 A-star (Hybrid A-star)

이 알고리즘의 근간은 고전적인 A-star 탐색 알고리즘을 로봇의 동역학까지 고려하도록 확장한 **하이브리드 A-star** 개념에 있다.1 일반적인 A-star가 2D 또는 3D 그리드 상의 위치만을 상태로 간주하는 반면, 하이브리드 A-star는 로봇의 상태 공간을 위치뿐만 아니라 속도까지 포함하도록 확장한다. 즉, 상태는 단순히 위치 $(p_x, p_y, p_z)$가 아니라, 속도를 포함하는 6차원 벡터 $(p_x, p_y, p_z, v_x, v_y, v_z)$로 정의된다. 이러한 확장된 상태 공간에서의 탐색을 통해 생성된 경로는 시작부터 동역학적으로 실현 가능하다는 큰 장점을 가진다.

#### 2.1.2 목표 함수와 탐색 과정

탐색의 목표는 목표 지점까지 도달하는 데 걸리는 시간과 제어 비용을 최소화하는 것이다. A-star의 비용 함수 $f(n) = g(n) + h(n)$에서 각 항은 다음과 같이 정의된다:

- $g(n)$: 시작점부터 현재 노드 $n$까지 도달하는 데 소요된 실제 누적 시간 비용이다.
- $h(n)$: 현재 노드 $n$에서 목표점까지 도달하는 데 필요할 것으로 예상되는 잔여 시간 비용, 즉 휴리스틱(heuristic)이다. 이 휴리스틱은 계산 효율성을 위해 동역학을 잠시 무시하고 계산한 직선 경로 도달 시간이나, 더 정교하게는 선형-이차(Linear-Quadratic, LQ) 최소 시간 제어 분석을 통해 얻을 수 있다.2

탐색 과정은 다음과 같이 진행된다:

1. **제어 입력 샘플링:** 쿼드로터에 가해질 수 있는 제어 입력(가속도 또는 저크)을 유한한 개수의 이산적인 집합으로 샘플링한다.
2. **상태 전파:** 현재 상태에서 각 이산 제어 입력을 짧은 시간 동안 적용했을 때 도달하게 될 다음 상태(위치, 속도)를 동역학 모델을 통해 예측(forward integration)한다.
3. **유효성 검사:** 예측된 다음 상태가 안전한지(ESDF 맵 상에서 장애물과 충돌하지 않는지), 그리고 동역학적 제약(최대 속도, 최대 가속도)을 만족하는지를 검사한다.
4. **탐색 확장:** 유효한 다음 상태들만을 우선순위 큐(priority queue)에 추가하여 비용이 가장 낮은 노드부터 탐색을 반복한다.

이러한 과정을 개념적으로 수학으로 표현하면 다음과 같다. 상태를 $x =^T \in \mathbb{R}^6$, 제어 입력을 $u \in \mathcal{U}$ (이산 제어 집합)라 할 때, 탐색 목표는 다음과 같은 최적 제어 문제를 푸는 것과 같다.
$$
\min_{u(t)} T \quad \text{s.t.} \quad \dot{x}(t) = f(x(t),u(t)),\\
x(0) = x_{start},\\
x(T) = x_{goal},\\
x(t) \in \mathcal{X}_{free},\\
u(t) \in \mathcal{U}_{admissible}
$$
여기서 $f(x, u) =^T$는 쿼드로터의 동역학 모델, $\mathcal{X}_{free}$는 충돌 없는 자유 공간, $\mathcal{U}_{admissible}$은 허용 가능한 제어 입력 범위를 의미한다.

### 2.2  백엔드: B-Spline 기반 궤적 최적화

프론트엔드에서 생성된 초기 경로는 실행은 가능하지만 거칠고 최적이 아닐 수 있다. 백엔드 최적화 단계는 이 경로를 기반으로 하여 훨씬 더 매끄럽고 안전한 최종 궤적을 생성한다.

#### 2.2.1 B-스플라인 궤적 표현

Fast-Planner는 궤적을 표현하기 위해 $p$차 균일 B-스플라인(uniform B-spline)을 사용한다. B-스플라인 궤적 $\mathbf{p}(t)$는 N+1개의 3차원 제어점 $\mathbf{Q}_0, \mathbf{Q}_1, \dots, \mathbf{Q}_N$의 선형 결합으로 정의된다.
$$
\mathbf{p}(t) = \sum_{i=0}^{N} \mathbf{Q}_i B_{i,p}(t)
$$
여기서 $B_{i,p}(t)$는 B-스플라인 기저 함수(basis function)이다. 이 표현 방식의 장점은 명확하다. 첫째, B-스플라인은 기저 함수의 속성상 제어점의 위치만으로도 궤적의 높은 차수의 연속성(매끄러움)이 자연스럽게 보장된다. 둘째, 지역적 수정 용이성(locality) 덕분에 하나의 제어점 $\mathbf{Q}_i$를 움직이면 궤적의 일부($p+1$개의 구간)만 지역적으로 변형되므로, 실시간으로 궤도를 수정하는 재계획(replanning)에 매우 효율적이다.2

#### 2.2.2 최적화 문제 정식화

백엔드의 목표는 최적의 제어점 집합 $\{\mathbf{Q}_i\}$를 찾는 것이다. 이는 아래와 같은 목표 함수 $J$를 최소화하는 문제로 정식화된다. 목표 함수는 궤적의 매끄러움($J_s$), 충돌 위험($J_c$), 그리고 동적 타당성($J_d$)을 나타내는 비용 항들의 가중치 합으로 구성된다.
$$
\min_{\mathbf{Q}_0, \dots, \mathbf{Q}_N} J = \lambda_s J_s + \lambda_c J_c + \lambda_d J_d
$$
각 비용 함수는 다음과 같이 정의된다:

- **매끄러움 비용 ($J_s$):** 궤적의 불필요한 움직임을 줄이고 부드럽게 만들기 위한 비용이다. 일반적으로 궤적의 고계도 미분(가속도, 저크 등)의 제곱을 시간에 대해 적분한 값으로 정의된다. B-스플라인의 속성을 이용하면, 이 적분 값은 인접한 제어점들 간의 차이를 최소화하는 것으로 근사할 수 있어 계산이 간편해진다.5 예를 들어, 저크(jerk, 가속도의 변화율)를 최소화하는 비용은 다음과 같이 근사된다.
  $$
  J_s = \int_{t_0}^{t_f} \left\| \frac{d^3\mathbf{p}(t)}{dt^3} \right\|^2 dt \approx \sum_{i} \| (\mathbf{Q}_{i+1} - \mathbf{Q}_i) - (\mathbf{Q}_i - \mathbf{Q}_{i-1}) \|^2
  $$

- **충돌 비용 ($J_c$):** 궤적이 장애물에 얼마나 가까운지를 나타내는 비용이다. `plan_env` 모듈에서 계산된 ESDF 값 $d(\mathbf{p})$를 이용하여, 궤적 상의 점 $\mathbf{p}$가 안전 거리 $d_{safe}$보다 장애물에 가까워질 경우 페널티를 부과하는 함수 $F_c$를 정의한다.
  $$
  F_c(\mathbf{p}) = \begin{cases} (d(\mathbf{p}) - d_{safe})^2 & \text{if } d(\mathbf{p}) \le d_{safe} \\ 0 & \text{if } d(\mathbf{p}) > d_{safe} \end{cases}
  $$
  전체 충돌 비용 $J_c$는 이 페널티 함수를 궤적 전체에 대해 적분하여 계산한다.

- **동적 타당성 비용 ($J_d$):** 궤적의 속도와 가속도가 기체의 물리적 한계($v_{max}, a_{max}$)를 초과하는 것을 방지하기 위한 비용이다.
  $$
  J_d = \int_{t_0}^{t_f} \left( F_v(\|\mathbf{v}(t)\|) + F_a(\|\mathbf{a}(t)\|) \right) dt
  $$
  여기서 $F_v$와 $F_a$는 각각 속도와 가속도가 한계를 초과할 경우에만 페널티를 부과하는 함수이다.

#### 2.2.3 핵심 최적화 기법: 볼록 포(Convex Hull) 속성 활용

이러한 비용 함수들을, 특히 적분 형태의 비용들을 매 최적화 단계마다 계산하는 것은 매우 비효율적이다. Fast-Planner는 B-스플라인의 중요한 수학적 성질인 **볼록 포(Convex Hull) 속성**을 활용하여 이 문제를 해결한다.1 이 속성에 따르면, B-스플라인의 한 구간(span)은 항상 그 구간을 정의하는 $p+1$개의 제어점들이 이루는 볼록 포 내부에 존재한다.

따라서, 궤적 전체가 안전하고 동적 제약을 만족하는지 일일이 확인하는 대신, **제어점들 자체가 안전한 영역에 있고, 제어점들로부터 계산된 속도/가속도가 제약을 만족하도록 강제**하는 것만으로도 전체 궤적의 안전성과 타당성을 높은 수준으로 보장할 수 있다. 예를 들어, 충돌 비용의 그래디언트 $\nabla J_c$는 ESDF의 그래디언트 $\nabla d(\mathbf{p})$를 통해 계산되는데, 이 계산을 궤적 상의 무수히 많은 점이 아닌, 유한한 개수의 제어점들에 대해서만 수행하면 되므로 계산 효율성이 극적으로 향상된다.2

### 2.3  실시간 재계획: 위상 경로 탐색과 경로 안내 최적화(PGO)

고속 비행 중에는 이전에 알려지지 않았던 장애물이 나타나므로, 지속적인 궤적 재계획(replanning)이 필수적이다. 이때 경사도 기반 최적화(GTO)의 고질적인 문제인 지역 최솟값(Local Minima) 문제가 발생한다.

#### 2.3.1 문제점: GTO의 지역 최솟값(Local Minima)

GTO는 초기값에 매우 민감하다. 만약 현재 궤적이 장애물의 '잘못된' 쪽에 위치해 있다면, 충돌 비용을 줄이려는 그래디언트는 궤적을 장애물 쪽으로 더욱 밀어붙여 결국 충돌하는 해(infeasible local minimum)에 수렴하게 만들 수 있다. 이는 안전에 치명적인 결과를 초래할 수 있으며, 더 나은 해가 존재함에도 불구하고 찾지 못하게 만든다.6

#### 2.3.2 해결책: PGO와 위상 경로 탐색

Fast-Planner는 이 문제를 체계적으로 해결하기 위해 두 가지 강력한 기법을 도입했다. 이 기법들의 도입은 단순한 지역적 개선(local refinement)에서 벗어나, 해 공간에 대한 전역적 탐색(global exploration)으로 나아가는 철학적 전환을 의미한다.

1. **경로 안내 최적화 (Path-Guided Optimization, PGO):** 지역 최솟값 문제를 완화하기 위한 첫 번째 단계이다. 최적화를 시작하기 전에, 충돌이 없는 순수한 기하학적 경로(guiding path)를 A-star 등으로 빠르게 찾는다. 그리고 최적화 목표 함수에, B-스플라인 궤적이 이 가이드 경로에서 너무 멀리 벗어나지 않도록 유도하는 추가적인 비용 항($J_{guide}$)을 도입한다. 이 가이드 경로는 궤적이 장애물을 통과하려 하지 않고 안전하게 '우회'하도록 이끄는 '족쇄' 역할을 하여, 실행 불가능한 지역 최솟값에 빠질 확률을 크게 줄여준다.6
2. **위상 경로 탐색 (Topological Path Searching):** PGO가 단 하나의 가이드 경로에 의존하는 한계를 극복하기 위해, Fast-Planner는 한 걸음 더 나아간다. 복잡한 환경에는 본질적으로 다른 여러 갈래의 길이 존재할 수 있다. 예를 들어, 기둥 장애물이 있을 때 '왼쪽으로 가는 경로'와 '오른쪽으로 가는 경로'는 위상적으로 구별되는(topologically distinct) 경로이다. Fast-Planner는 샘플링 기반의 경로 탐색 기법(Visibility-PRM 등)을 변형하여, 이러한 위상적으로 다른 경로들을 의도적으로 여러 개 찾아낸다.4

이 두 기법을 통합하여, 찾아낸 여러 개의 위상 경로 각각에 대해 독립적인 PGO를 병렬적으로 수행한다. 그 결과로 여러 개의 후보 궤적들이 생성되고, 그중에서 전체 비용이 가장 낮은(가장 우수한) 궤적을 최종적으로 선택한다. 이 방식은 해 공간(solution space)을 훨씬 더 넓고 포괄적으로 탐색하여, 단일 경로에 갇혔을 때보다 지역 최솟값을 회피하고 더 우수한 품질의 해를 찾을 확률을 극적으로 향상시킨다.6 이는 플래너의 강건성과 성능을 한 차원 높은 수준으로 끌어올리는 핵심적인 혁신이다.

### 2.4  지각 인식 계획: RAPTOR 프레임워크

Fast-Planner의 진화는 여기서 멈추지 않는다. `RAPTOR` 논문에서 제안된 지각 인식 계획(Perception-aware Planning)은 계획의 패러다임을 한 단계 더 발전시킨다.3

#### 2.4.1 개념의 확장: 불확실성의 관리

기존의 계획 방식은 '현재 센서로 관측된 맵'이 세상의 전부라고 가정하고 그 안에서 최적의 경로를 찾는다. 하지만 고속 비행 시에는 센서의 제한된 시야(Limited Field-of-View) 때문에 경로 전방의 '보이지 않는 영역(unknown space)'이 가장 큰 잠재적 위협이다. 지각 인식 계획은 이러한 불확실성을 회피의 대상이 아닌, 관리의 대상으로 보고 계획 과정에 직접 통합한다.

#### 2.4.2 RAPTOR의 전략

RAPTOR 프레임워크는 다음과 같은 전략을 통해 이를 구현한다 13:

1. **능동적 관찰 (Active Observation):** 궤적을 계획할 때, 단순히 장애물을 피하는 것뿐만 아니라, 경로 상의 불확실한 영역을 가장 잘 관찰하여 정보를 최대한 얻을 수 있는 경로에 더 높은 가치를 부여하도록 비용 함수를 설계한다.
2. **위험 인지 궤적 개선 (Risk-aware Trajectory Refinement):** 잠재적으로 위험할 수 있는 미지의 장애물을 가능한 한 더 일찍 관찰하고 제시간에 회피할 수 있도록 궤적을 능동적으로 수정한다.
3. **Yaw 각도 계획 (Yaw Angle Planning):** 드론의 전진 방향과 무관하게 카메라가 향하는 방향(Yaw 각도)을 독립적으로 계획한다. 이를 통해 드론이 직진하면서도 측면의 미탐사 영역을 훑어보는 등, 항법에 중요한 주변 공간을 더욱 적극적으로 탐사할 수 있다.

이러한 접근법은 드론이 수동적으로 환경 데이터를 받아들이는 것을 넘어, 스스로 정보 획득을 극대화하도록 행동하게 만든다. 이는 고속으로 미지의 환경을 비행할 때 안전성과 임무 성공률을 획기적으로 높일 수 있는 잠재력을 가진다.13

## 3.  성능 비교 및 벤치마킹

### 3.1  Fast-Planner vs. EGO-Planner: ESDF의 유무

Fast-Planner의 영향력을 가장 잘 보여주는 사례 중 하나는 바로 그 후속 연구인 `EGO-Planner`이다. EGO-Planner는 Fast-Planner의 프레임워크를 직접적으로 계승했지만, 한 가지 결정적인 차이점을 통해 성능의 새로운 지평을 열었다.5

#### 3.1.1 ESDF-free 접근법

EGO-Planner의 가장 큰 혁신은 **ESDF 맵의 구축 및 유지 과정을 완전히 생략**한 것이다.16 기존 경사도 기반 플래너에서 ESDF 생성은 전체 계획 시간의 상당 부분(최대 70%까지)을 차지하는 주요 계산 병목 구간이었다.5 EGO-Planner는 이 병목을 제거하기 위해, ESDF 맵 대신 센서로부터 직접 들어온 포인트 클라우드 장애물 정보로부터 충돌 비용의 그래디언트를 계산하는 방식을 택했다. 이는 궤적 상의 점과 주변 장애물 포인트들 간의 거리 벡터를 직접 활용하여, 궤적을 밀어내는 힘을 계산하는 방식으로 이루어진다.

#### 3.1.2 성능 트레이드오프

이러한 구조적 차이는 두 플래너 간의 명확한 성능 트레이드오프를 야기한다.

- **EGO-Planner의 장점:** ESDF 생성 과정을 생략함으로써, 전체 계획 시간을 1ms 내외로 극적으로 단축시켰다. 이는 매우 빠른 주기로 경로를 재계획해야 하는 공격적인 고속 비행 시나리오에서 압도적인 장점이 된다.16
- **Fast-Planner(ESDF 기반)의 잠재적 장점:** ESDF는 맵 전체에 대한 전역적인(global) 거리 정보를 제공하므로, 이로부터 계산된 그래디언트 필드는 더 매끄럽고 안정적인 경향이 있다. 반면, EGO-Planner의 직접적인 그래디언트 계산 방식은 센서 데이터의 노이즈에 더 민감하게 반응하거나, 복잡한 형태의 장애물 근처에서 그래디언트가 급격하게 변하는 등 불안정한 동작을 보일 잠재적 가능성이 있다. 즉, EGO-Planner는 속도를 위해 그래디언트 정보의 일부 질을 희생한 것으로 볼 수 있다.

### 3.2  "Speed-First" 벤치마크 심층 분석

"Speed-First: An Aggressive Gradient-Based Local Planner for Quadrotor Faster Flight" 논문은 EGO-Planner와 같은 기존 경사도 기반 플래너들이 여전히 불필요하게 보수적인 궤적을 생성하는 문제를 지적하며, 이를 개선한 새로운 플래너("Ours" 또는 "Speed-First")를 제안하고 EGO-Planner와 직접적인 성능 비교를 수행했다.17

#### 3.2.1 실험 조건 및 분석

실험은 세 가지 다른 장애물 밀도(Low: 30개, Medium: 50개, High: 70개)를 가진 시뮬레이션 환경에서, 다양한 동적 제한($v_{max}, a_{max}$)을 적용하며 진행되었다.17 아래 표는 논문의 표 2, 3, 4에서 가장 까다로운 동적 제한 조건($v_{max}=8 m/s, a_{max}=10 m/s^2$)에서의 성능 데이터를 종합한 것이다.

| 환경 조건         | 플래너      | 계획 시간 (ms) | 비행 시간 (s) | 평균 속도 (m/s) | 최대 속도 (m/s) | 성공 여부 |
| ----------------- | ----------- | -------------- | ------------- | --------------- | --------------- | --------- |
| **저밀도 (30개)** |             |                |               |                 |                 |           |
| v_max=8, a_max=10 | EGO-Planner | 0.93           | 5.14          | 3.70            | 5.92            | 성공      |
|                   | Speed-First | 0.92           | **4.21**      | **4.51**        | **7.13**        | 성공      |
| **중밀도 (50개)** |             |                |               |                 |                 |           |
| v_max=8, a_max=10 | EGO-Planner | 1.78           | 7.34          | 2.59            | 5.21            | 성공      |
|                   | Speed-First | **1.35**       | **5.89**      | **3.23**        | **6.89**        | 성공      |
| **고밀도 (70개)** |             |                |               |                 |                 |           |
| v_max=8, a_max=10 | EGO-Planner | -              | -             | -               | -               | **실패**  |
|                   | Speed-First | **4.37**       | **8.78**      | **2.16**        | **5.58**        | **성공**  |

#### 3.2.2 분석 결과

이 데이터는 몇 가지 중요한 사실을 명확하게 보여준다.

- **계산 시간:** 환경이 복잡해질수록(장애물 밀도 증가) Speed-First 플래너의 계산 시간 우위가 더욱 명확해진다. 중밀도 환경에서 Speed-First는 EGO-Planner보다 약 24% 더 빠른 계획 시간을 보였다.17
- **비행 속도 및 시간:** 모든 조건에서 Speed-First는 EGO-Planner보다 훨씬 짧은 비행 시간과 높은 평균/최대 속도를 기록했다. 이는 Speed-First가 불필요하게 보수적인 계획을 피하고, 동적 한계 내에서 가능한 한 가장 공격적인 궤적을 생성하는 능력 덕분이다. 저밀도 환경에서 비행 시간은 18% 이상 단축되었다.17
- **견고성 (Robustness):** 가장 결정적인 결과는 가장 어려운 조건, 즉 고밀도 장애물 환경에서 높은 동적 한계를 적용했을 때 나타났다. 이 조건에서 EGO-Planner는 실행 가능한 궤적을 생성하는 데 실패했지만, Speed-First는 성공적으로 임무를 완수했다. 이는 Speed-First의 최적화 전략이 극한 상황에서 더 강건하며 안정적임을 시사하는 강력한 증거이다.17

결론적으로, 이 벤치마크는 경사도 기반 플래너의 지속적인 진화를 보여준다. Fast-Planner가 기반을 닦고 EGO-Planner가 속도를 개선했다면, Speed-First와 같은 후속 연구는 다시 강건성과 비행 성능을 한 단계 끌어올리고 있음을 알 수 있다.

## 4.  응용 및 확장

Fast-Planner가 제공하는 강력하고 유연한 궤적 계획 능력은 단순히 한 지점에서 다른 지점으로 이동하는 것을 넘어, 다양한 실제 응용 분야로 확장될 수 있는 잠재력을 가지고 있다.

### 4.1  자율 탐사 (Autonomous Exploration)

미지의 환경을 드론이 스스로 돌아다니며 3D 지도를 완성하는 자율 탐사 임무는 Fast-Planner의 기술이 직접적으로 활용되는 대표적인 분야이다.3 탐사 임무의 핵심은 '어디로 가야 가장 많은 정보를 얻을 수 있는가'를 결정하는 상위 레벨의 탐사 전략과, '결정된 목표 지점까지 어떻게 빠르고 안전하게 갈 것인가'를 해결하는 하위 레벨의 궤적 계획으로 나뉜다. Fast-Planner는 바로 이 하위 레벨의 핵심적인 역할을 수행한다.

`FUEL`과 그 후속 연구인 `FAEP`와 같은 프로젝트들은 Fast-Planner의 프레임워크를 기반으로 개발되었다. 이들은 Fast-Planner의 빠른 재계획 능력을 활용하면서, 정보 획득량을 극대화하기 위한 효율적인 탐사 시퀀스 생성, 불필요한 왕복 이동을 줄이는 전역 경로 계획, 그리고 비행 중 Yaw 각도를 능동적으로 조절하여 탐사 효율을 높이는 등의 상위 레벨 전략을 추가하여 탐사 성능을 극대화했다.3

### 4.2  드론 레이싱 (Drone Racing)

드론 레이싱은 자율 비행 기술의 성능을 극한까지 시험하는 궁극적인 도전 과제이다. 이 분야에서는 단순히 장애물을 피하는 것을 넘어, 정해진 순서대로 게이트를 통과하며 랩 타임을 최소화해야 한다.19 이를 위해서는 기체의 물리적 한계에 근접하는 매우 공격적이면서도 정교한 궤적 생성이 요구된다.

Fast-Planner는 `RACER`와 같은 드론 레이싱 프로젝트의 기반 기술로 언급되며, 그 유연성을 입증했다.3 또한, ZJU FAST Lab의 `Fast-Racing` 프로젝트는 GCOPTER와 MINCO와 같은 최적화 기술을 활용하여 극도로 공격적인 SE(3) 궤적(위치와 자세를 동시에 최적화)을 생성하는 강력한 베이스라인을 제공하는데, 이는 Fast-Planner가 추구하는 고속 비행 철학의 연장선에 있다.20 이는 Fast-Planner의 최적화 프레임워크가 안전한 항법뿐만 아니라, 성능의 한계를 넘나드는 최고 속도 비행에도 성공적으로 적용될 수 있음을 보여준다.

### 4.3  잠재적 산업 응용

Fast-Planner와 그 파생 기술들은 다양한 산업 현장에서 인간의 작업을 더 안전하고 효율적으로 대체할 잠재력을 가지고 있다.

- **인프라 검사 (Infrastructure Inspection):** 전력선, 교량, 통신 타워, 풍력 터빈 등 인간의 접근이 어렵거나 위험한 대형 구조물을 검사하는 데 핵심적인 역할을 할 수 있다. Fast-Planner의 능력은 복잡한 구조물 주변에서 충돌 없이 안전하게 비행하며, 정밀한 웨이포인트 항법을 통해 고품질의 검사 데이터를 자동으로 수집하는 것을 가능하게 한다.21
- **정밀 농업 (Precision Agriculture):** 드론이 넓은 농경지를 자율적으로 비행하며 작물의 건강 상태를 나타내는 다중 스펙트럼 이미지를 수집하고, 이를 분석하여 비료나 농약을 필요한 곳에만 정확한 양으로 살포하는 가변 비율 살포(variable rate application) 맵을 생성하는 데 활용될 수 있다. 이는 생산성을 높이고 환경 영향을 줄이는 데 기여한다.24
- **수색 및 구조 (Search and Rescue):** 지진이나 건물 붕괴 같은 재난 현장은 구조대원의 접근이 매우 위험하다. Fast-Planner 기반의 드론은 이러한 미지의 환경을 신속하고 안전하게 탐색하여 3D 맵을 생성하고, 생존자를 찾거나 위험 요소를 파악하는 데 결정적인 정보를 제공할 수 있다.4

## 5. 결론: Fast-Planner의 유산과 미래 전망

### 핵심 기여 요약

Fast-Planner는 이론적으로 잘 정립된 키노다이내믹 경로 탐색과 강력한 경사도 기반 최적화 기법을 실용적이고 강건한 단일 시스템으로 통합한 선구적인 프로젝트이다. 이 프로젝트가 제시한 계층적 계획 구조, B-스플라인과 ESDF의 효율적인 활용, 그리고 GTO의 고질적인 지역 최솟값 문제에 대한 체계적인 해결책(PGO, Topological Path Searching)은 자율 드론 기술의 발전에 중요한 이정표를 세웠다.

### 5.1 기술적 유산과 영향력

Fast-Planner의 가장 큰 유산은 그 자체의 뛰어난 성능을 넘어, 수많은 후속 연구에 영감과 실제적인 코드베이스를 제공한 '연구 가속기(Research Accelerator)'로서의 역할이다. EGO-Planner, FUEL, RACER 등 자율 비행 분야의 주요 오픈소스 프로젝트들이 모두 Fast-Planner의 어깨 위에서 시작하여 각자의 방향으로 발전했다는 사실은 그 영향력을 명백히 증명한다.3 이는 잘 설계된 오픈소스 플랫폼이 학문적 발전에 얼마나 크게 기여할 수 있는지를 보여주는 모범적인 사례이다.

### 5.2 현재의 한계와 미래 연구 방향

모든 위대한 연구와 마찬가지로, Fast-Planner 역시 새로운 도전 과제와 미래 연구 방향을 제시한다.

- **동적 장애물 회피:** 현재의 Fast-Planner는 대부분 정적인 환경을 가정한다. 실제 환경에는 움직이는 사람, 차량 등 예측 불가능한 동적 장애물이 존재하며, 이를 실시간으로 예측하고 회피하는 기술은 여전히 해결해야 할 큰 과제이다.26
- **다개체 협응 비행 (Multi-agent Coordination):** 단일 드론을 넘어, 여러 대의 드론이 서로 충돌하지 않고 통신 지연과 같은 현실적인 제약 속에서 협력하여 공동의 임무를 수행하는 분산형 궤적 계획은 더욱 복잡한 연구 영역이다.27
- **지각-행동 루프의 심화:** RAPTOR에서 시작된 지각 인식 계획은 더욱 발전하여, 불확실성 하에서의 의사결정(Decision-making under uncertainty) 문제로 확장될 것이다. 이는 강인한 상태 추정(Robust State Estimation)과 계획(Planning)이 더욱 긴밀하게 결합되어, 드론이 '무엇을 모르는지'를 인지하고 이를 해결하기 위해 능동적으로 행동하는 수준까지 나아가야 함을 의미한다.29

최종 평가

Fast-Planner는 쿼드로터의 고속 자율 비행이라는 난제를 '가능성의 영역'에서 '실용성의 영역'으로 한 걸음 더 가깝게 이끈 중추적인 작업이다. 이 프로젝트가 제시한 문제 해결의 패러다임과 그 위에 쌓아 올린 점진적인 개선의 과정은, 앞으로도 오랫동안 자율 로봇 분야의 연구자들에게 중요한 참고 자료이자 새로운 영감의 원천이 될 것이다.

#### 참고 자료

1. Robust and Efficient Quadrotor Trajectory Generation for Fast ... - arXiv, accessed August 11, 2025, https://arxiv.org/pdf/1907.01531
2. Robust and Efficient Quadrotor Trajectory Generation for Fast Autonomous Flight | Shiyu Chen, accessed August 11, 2025, https://www.chenshiyu.top/blog/2020/11/10/Robust-and-Efficient-Quadrotor-Trajectory-Generation-for-Fast-Autonomous-Flight/
3. HKUST-Aerial-Robotics/Fast-Planner: A Robust and Efficient Trajectory Planner for Quadrotors - GitHub, accessed August 11, 2025, https://github.com/HKUST-Aerial-Robotics/Fast-Planner
4. Robust and Efficient Quadrotor Trajectory Generation for Fast Autonomous Flight | Request PDF - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/334191929_Robust_and_Efficient_Quadrotor_Trajectory_Generation_for_Fast_Autonomous_Flight
5. EGO-Planner: An ESDF-free Gradient-based Local Planner for Quadrotors - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/348028324_EGO-Planner_An_ESDF-free_Gradient-based_Local_Planner_for_Quadrotors
6. Robust Real-time UAV Replanning Using Guided Gradient-based ..., accessed August 11, 2025, https://www.researchgate.net/publication/338292412_Robust_Real-time_UAV_Replanning_Using_Guided_Gradient-based_Optimization_and_Topological_Paths
7. [1912.12644] Robust Real-time UAV Replanning Using Guided Gradient-based Optimization and Topological Paths - ar5iv, accessed August 11, 2025, https://ar5iv.labs.arxiv.org/html/1912.12644
8. Topology Graph Aided Gradient-based Trajectory Optimization for Robust UAV Replanning, accessed August 11, 2025, https://arxiv.org/html/2309.13882v3
9. [1912.12644] Robust Real-time UAV Replanning Using Guided Gradient-based Optimization and Topological Paths - arXiv, accessed August 11, 2025, https://arxiv.org/abs/1912.12644
10. An Gradient-based Continuous-Time Trajectory Optimization method for Real-Time Collision Avoidance on collaborative robot - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/361523855_An_Gradient-based_Continuous-Time_Trajectory_Optimization_method_for_Real-Time_Collision_Avoidance_on_collaborative_robot
11. README.md - HKUST-Aerial-Robotics/TopoTraj - GitHub, accessed August 11, 2025, https://github.com/HKUST-Aerial-Robotics/TopoTraj/blob/master/README.md
12. Robust Real-time UAV Replanning Using Guided Gradient-based Optimization and Topological Paths | Request PDF - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/344982637_Robust_Real-time_UAV_Replanning_Using_Guided_Gradient-based_Optimization_and_Topological_Paths
13. [2007.03465] RAPTOR: Robust and Perception-aware Trajectory Replanning for Quadrotor Fast Flight - arXiv, accessed August 11, 2025, https://arxiv.org/abs/2007.03465
14. Robust and Efficient Trajectory Replanning Based on Guiding Path for Quadrotor Fast Autonomous Flight - MDPI, accessed August 11, 2025, https://www.mdpi.com/2072-4292/13/5/972
15. RAPTOR: Robust and Perception-aware Trajectory Replanning for Quadrotor Fast Flight (2007.03465v1) - Emergent Mind, accessed August 11, 2025, https://www.emergentmind.com/articles/2007.03465
16. ZJU-FAST-Lab/ego-planner - GitHub, accessed August 11, 2025, https://github.com/ZJU-FAST-Lab/ego-planner
17. Speed-First: An Aggressive Gradient-Based Local Planner for ..., accessed August 11, 2025, https://www.mdpi.com/2504-446X/7/3/192
18. Zyhlibrary/FAEP - GitHub, accessed August 11, 2025, https://github.com/Zyhlibrary/FAEP
19. [2309.06837] Time-Optimal Gate-Traversing Planner for Autonomous Drone Racing - arXiv, accessed August 11, 2025, https://arxiv.org/abs/2309.06837
20. ZJU-FAST-Lab/Fast-Racing: An Open-source Strong ... - GitHub, accessed August 11, 2025, https://github.com/ZJU-FAST-Lab/Fast-Racing
21. Understanding Drone Flight Path Planning Software: A Complete Guide for Professionals, accessed August 11, 2025, https://www.dslrpros.com/blogs/learning-center/understanding-drone-flight-path-planning-software
22. Drone Powerline Inspection Software With Computer Vision AI - Optelos, accessed August 11, 2025, https://optelos.com/power-utilities/
23. Drone Inspection - Skyline Software Systems, accessed August 11, 2025, https://www.skylinesoft.com/inspection/
24. PIX4Dfields: Drone software for agriculture mapping | Pix4D, accessed August 11, 2025, https://www.pix4d.com/product/pix4dfields
25. abdulkadrtr/ROS2-FrontierBaseExplorationForAutonomousRobot: Our autonomous ground vehicle uses Frontier Based exploration to navigate and map unknown environments. Equipped with sensors, it can avoid obstacles and make real-time decisions. It has potential applications in search and rescue, agriculture, and logistics, and represents an important step forward in autonomous ground vehicle development - GitHub, accessed August 11, 2025, https://github.com/abdulkadrtr/ROS2-FrontierBaseExplorationForAutonomousRobot
26. LudvigWiden/daeplanner: Dynamic Autonomous Exploration Planner (DAEP) - GitHub, accessed August 11, 2025, https://github.com/LudvigWiden/daeplanner
27. EGO-Planner V2.1: Swarm of Heterogeneous Mobile Robots in Habitat - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/385880528_EGO-Planner_V21_Swarm_of_Heterogeneous_Mobile_Robots_in_Habitat
28. Distributed Drone Swarm Trajectory Planner Using Topology Planning Under Communication Delay | Request PDF - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/388295144_Distributed_Drone_Swarm_Trajectory_Planner_Using_Topology_Planning_Under_Communication_Delay
29. [2403.08365] APACE: Agile and Perception-Aware Trajectory Generation for Quadrotor Flights - arXiv, accessed August 11, 2025, https://arxiv.org/abs/2403.08365


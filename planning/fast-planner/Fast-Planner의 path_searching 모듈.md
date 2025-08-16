# `Fast-Planner`의 `path_searching` 모듈

## 서론

`Fast-Planner`는 복잡하고 미지의 환경에서 쿼드콥터의 빠른 자율 비행을 목표로 개발된 강건하고(robust) 효율적인(efficient) 궤적 계획 시스템이다.1 이 시스템은 프론트엔드 경로 탐색, 백엔드 궤적 최적화, 그리고 온라인 매핑 모듈로 구성된 계층적 구조를 채택한다. 

`Fast-Planner`는 EGO-Planner, FUEL, RACER와 같은 여러 유명 오픈소스 드론 프로젝트의 기반 코드 프레임워크를 제공할 정도로 그 완성도와 확장성이 입증되었다.1

`fast_planner` 패키지 내의 `path_searching` 모듈은 전체 플래닝 파이프라인의 프론트엔드(front-end) 역할을 수행한다.1 이 모듈의 핵심 임무는 `plan_env` 모듈로부터 제공된 환경 정보(주로 ESDF 맵)를 바탕으로, 동역학적 제약 조건을 만족하는 안전하고 실행 가능한 초기 궤적(initial trajectory)을 생성하는 것이다. 여기서 생성된 초기 궤적의 품질은 후속 백엔드 최적화 단계(`bspline_opt`)의 성공률과 최종 궤적의 품질에 결정적인 영향을 미치므로, `path_searching`은 시스템 전체 성능의 초석이 된다.

이 모듈은 다양한 상황에 대응하기 위해 여러 탐색 알고리즘을 포함한다.1

**Kinodynamic Path Searching**은 쿼드콥터의 동역학(dynamics)을 존중하는 경로를 탐색하는 핵심 알고리즘으로, 단순한 기하학적 경로가 아닌 속도와 가속도 제약을 만족하는 실행 가능한 궤적을 직접 찾아낸다.2

**Topological Path Searching**은 복잡한 환경에서 국소 최적해(local minima) 문제를 극복하기 위해 도입된 샘플링 기반 알고리즘으로, 장애물과의 위상학적 관계가 다른(topologically distinctive) 여러 경로를 탐색하여 솔루션 공간을 더 넓게 탐색한다.3 마지막으로, 동역학을 고려하지 않고 그리드 맵 상에서 최단 경로를 찾는 **Standard A*** 알고리즘 또한 가이드 경로 생성이나 비교 목적으로 포함되었을 것으로 추정된다.5

`Fast-Planner`의 아키텍처는 복잡한 모션 플래닝 문제를 '실행 가능성 탐색(feasibility search)'과 '품질 최적화(quality optimization)'라는 두 개의 하위 문제로 분리하는 전형적인 "Front-End/Back-End" 설계 패턴을 따른다. 쿼드콥터의 궤적 생성은 비선형적이고 비볼록(non-convex)인 최적화 문제이며, 동역학 제약, 충돌 회피, 평활성 등을 모두 동시에 고려하는 것은 계산적으로 매우 어렵다.2 이 문제를 직접 풀려고 하면 국소 최적해에 빠지기 쉽고 계산 시간이 오래 걸린다.2 따라서 문제를 분할하는 전략이 효과적이다. 프론트엔드인 `path_searching` 모듈은 이산적인 탐색 공간에서 A*와 같은 그래프 탐색 기법을 사용하여 전역적인 경로를 찾아낸다.2 이 단계에서는 평활성보다는 실행 가능성과 안전성에 초점을 맞춘다. 일단 실행 가능한 경로가 확보되면, 백엔드인 `bspline_opt`는 이 경로 주변의 안전한 공간(safe corridor) 내에서 연속적인 B-스플라인 궤적을 최적화한다.1 이 단계에서는 초기 경로가 좋은 시작점을 제공하므로 최적화가 더 빠르고 안정적으로 수렴할 수 있다. 결론적으로, 이 계층적 접근 방식은 각 모듈이 자신의 전문 분야에 집중하게 함으로써 전체 문제의 복잡성을 관리하고, 실시간 환경에서의 강건성과 효율성을 동시에 달성하는 핵심 전략이다.

## 1. `path_searching` 모듈의 아키텍처 분석

### 1.1 디렉토리 및 파일 구조

`path_searching` 모듈은 표준 ROS 패키지 구조를 따르며, 주요 코드는 `include`와 `src` 디렉토리에 나뉘어 배치된다.

- `include/path_searching/`: 클래스 정의, 데이터 구조(예: `PathNode`, `GridNode`), 그리고 외부 모듈에 노출되는 API(Application Programming Interface)를 선언하는 헤더 파일(.h)들이 위치한다. 이 디렉토리의 파일들은 모듈의 '공개 인터페이스'를 정의한다.
- `src/`: 헤더 파일에 선언된 함수와 클래스들의 구체적인 구현을 포함하는 소스 파일(.cpp)들이 위치한다. 알고리즘의 핵심 로직은 이 디렉토리의 파일들에 담겨 있다.6
- `CMakeLists.txt`: 빌드 설정 파일로서, 컴파일 옵션과 다른 ROS 패키지와의 의존성을 명시한다. 특히 `path_searching` 모듈은 충돌 검사를 위해 `plan_env` 모듈이 제공하는 환경 정보에 의존하므로, 이 파일에 해당 의존성이 정의되어 있다.

### 1.2 `path_searching` 모듈의 핵심 파일 및 기능

`path_searching` 모듈은 여러 경로 탐색 알고리즘을 구현한 파일들로 구성된다. 각 파일은 특정 탐색 문제를 해결하는 데 특화되어 있다.

| 파일 (File)                | 주요 클래스 (Main Class)   | 기능 및 역할 (Function and Role)                             | 관련 자료 |
| -------------------------- | -------------------------- | ------------------------------------------------------------ | --------- |
| `kinodynamic_astar.h/.cpp` | `KinodynamicAstar`         | 쿼드콥터의 동역학(위치, 속도)을 고려한 하이브리드 A* 탐색. Motion Primitives를 사용하여 실행 가능한 궤적을 생성. `Fast-Planner`의 핵심 프론트엔드. | 2         |
| `astar.h/.cpp`             | `Astar`                    | 표준 A* 알고리즘. 동역학을 무시하고 3D 그리드 맵 상에서 기하학적인 최단 경로를 탐색. 주로 가이드 경로 생성이나 비교 용도로 사용될 가능성. | 5         |
| (Topological Search Files) | (e.g., `TopoPathSearcher`) | 샘플링 기반으로 위상학적으로 다른 다수의 경로를 생성. `kinodynamic_astar`가 국소 최적해에 빠지는 것을 방지. `TopoTraj`의 핵심 아이디어를 통합. | 3         |

### 1.3 데이터 흐름 및 모듈 간 상호작용

`path_searching` 모듈은 `Fast-Planner` 시스템 내에서 명확한 입력, 처리, 출력 흐름을 가진다.

- **입력 (Input):** 상위 모듈인 `plan_manage`는 ROS 토픽이나 서비스를 통해 수신된 목표 지점과 현재 로봇의 상태(위치, 속도, 가속도)를 `path_searching` 모듈의 탐색 함수(예: `search()`)에 인자로 전달한다.1 동시에, `path_searching`은 `plan_env` 모듈이 실시간으로 생성 및 업데이트하는 ESDF(Euclidean Signed Distance Field) 맵에 직접 접근하여 경로의 충돌 여부를 검사한다.1 ESDF는 특정 지점에서 가장 가까운 장애물까지의 거리를 제공하므로, 충돌 검사를 매우 효율적으로 수행할 수 있게 한다.
- **처리 (Processing):** `KinodynamicAstar` 또는 `TopoPathSearcher`와 같은 핵심 클래스는 입력된 상태와 목표, 그리고 ESDF 맵을 사용하여 탐색 알고리즘을 수행한다. 이 과정에서 수많은 후보 경로(motion primitives)들이 생성되고 평가된다.
- **출력 (Output):** 탐색이 성공하면, 결과로 나온 경로(일련의 점 또는 B-스플라인 제어점)를 반환한다. 이 경로는 `plan_manage` 모듈을 통해 백엔드 최적화 모듈인 `bspline_opt`로 전달된다.1`getKinoTraj()`나 `getSamples()`와 같은 함수가 이 역할을 수행하는 것으로 보인다.8

`path_searching` 모듈의 성능과 효율성은 `plan_env` 모듈이 제공하는 ESDF에 절대적으로 의존한다. ESDF는 단순한 점유 격자 맵(Occupancy Grid Map)을 넘어, 경로 탐색과 최적화에 필수적인 '거리'와 '그래디언트' 정보를 제공하는 핵심 데이터 구조이다. 경로 탐색기는 경로상의 각 점이 안전한지 매우 빈번하게 확인해야 하는데, 점유 격자 맵은 장애물 여부만 알려줄 뿐 안전 여유(clearance)를 알기 어렵다. 반면, ESDF는 맵의 모든 지점에서 가장 가까운 장애물까지의 부호화된 유클리드 거리를 미리 계산해 둔 맵이다.1 따라서 `path_searching` 모듈은 경로상의 한 점에 대해 ESDF 값을 조회하는 것만으로 $O(1)$ 시간에 충돌 여부 및 안전 여유를 즉시 알 수 있다. 이는 수많은 후보 경로를 평가해야 하는 A* 계열 알고리즘의 성능에 결정적이다. 나아가 ESDF의 그래디언트는 장애물로부터 멀어지는 방향을 가리키므로, 백엔드 최적화기(`bspline_opt`)가 궤적을 장애물로부터 밀어내는(push away) 데 사용된다.2 이러한 이유로 `path_searching` 모듈을 ESDF 없이 구현하는 것은 거의 불가능하며, `plan_env`와의 긴밀한 통합은 `Fast-Planner`의 설계상 필연적인 선택이다. 이는 EGO-Planner가 "ESDF-free"를 표방하며 차별점을 내세우는 것에서도 반증된다.9

## 2. Kinodynamic A* 알고리즘 심층 분석

### 2.1 이론적 배경: Kinodynamic Planning

Kinodynamic Planning은 로봇의 기구학적 제약(kinematic constraints, 예: 충돌 회피)과 동역학적 제약(dynamic constraints, 예: 속도/가속도/저크 한계)을 동시에 만족하는 궤적을 찾는 문제이다.11 이는 상태 공간(state space)이 위치뿐만 아니라 속도, 가속도 등을 포함하여 고차원으로 확장되기 때문에 일반적인 경로 탐색보다 훨씬 복잡하다. 

`Fast-Planner`는 이 문제를 해결하기 위해 이산화된 제어 공간에서 탐색을 수행하는 하이브리드 A*(Hybrid A*) 접근법을 채택했다.2

### 2.2 상태 공간, 제어 입력 및 Motion Primitives

`KinodynamicAstar` 알고리즘은 쿼드콥터의 움직임을 수학적으로 정밀하게 모델링한다.

- **상태 공간(State Space):** `Fast-Planner`는 쿼드콥터의 상태를 위치와 속도로 구성된 6차원 벡터로 모델링한다. 이는 논문에서 $n=2$인 경우에 해당하며, 더블 적분기(double integrator) 모델로 볼 수 있다.2
  $$
  x(t) :=^T \in \mathbb{R}^6
  $$
  여기서 $p(t) \in \mathbb{R}^3$는 3차원 위치, $\dot{p}(t) \in \mathbb{R}^3$는 3차원 속도이다.

- **제어 입력(Control Input):** 제어 입력 $u(t)$는 가속도($\ddot{p}(t)$)로 정의되며, 각 축에 대한 최댓값($a_{max}$) 제약을 받는다.2
  $$
  u(t) := \ddot{p}(t) \in U := [-a_{max}, a_{max}]^3 \subset \mathbb{R}^3
  $$
  **Motion Primitives:** 탐색을 위해 연속적인 제어 공간을 이산화한다. 이산화된 제어 입력 집합 $U_d \subset U$에서 하나의 제어 입력 $u_d$를 선택하여 고정된 시간 $\tau$ 동안 적용함으로써 Motion Primitive를 생성한다. 현재 상태 $x(0) =^T$에서 시작하여 $\tau$초 후의 상태 $x(\tau)$는 상태 방정식의 해석적 해(analytic solution)를 통해 다음과 같이 정확하게 계산된다.2
  $$
  p(\tau) = p(0) + \dot{p}(0)\tau + \frac{1}{2}u_d\tau^2
  $$

  $$
  \dot{p}(\tau) = \dot{p}(0) + u_d\tau
  $$

  이렇게 생성된 수많은 Motion Primitive들이 A* 탐색에서 그래프의 간선(edge) 역할을 한다.2

### 2.3 비용 함수: g-cost와 h-cost

`KinodynamicAstar`의 목표는 단순히 도달 가능한 경로가 아닌, 시간과 제어 에너지를 고려한 최적의 궤적을 찾는 것이다. 이는 비용 함수에 잘 나타나 있다.2

- **g-cost (실제 누적 비용):** 시작 노드부터 현재 노드까지의 실제 누적 비용이다. 각 Motion Primitive(간선)를 확장할 때마다 해당 Primitive의 비용이 누적된다. 비용 함수는 제어 입력의 크기(에너지)와 경과 시간을 모두 페널티로 간주한다.
  $$
  J(T) = \int_0^T (||u(t)||^2 + \rho) dt
  $$
  하나의 Primitive에 대한 비용(edge cost)은 $e_c = (||u_d||^2 + \rho)\tau$ 이며, g-cost는 이들의 합이다. 여기서 $\rho$는 시간 페널티 항으로, 제어 에너지가 0이더라도 시간이 흐르면 비용이 증가하게 하여 최단 시간 경로를 선호하도록 유도한다.

- **h-cost (휴리스틱 추정 비용):** 현재 노드에서 목표 노드까지의 남은 비용에 대한 추정치이다. `Fast-Planner`의 가장 독창적인 부분 중 하나로, 단순한 유클리드 거리가 아닌, 동역학을 고려한 최적 제어 문제의 해를 휴리스틱으로 사용한다.2 현재 상태 $x_c$에서 목표 상태 $x_g$까지 위의 비용 함수 $J(T)$를 최소화하는 궤적과 시간을 폰트랴긴의 최소 원리(Pontryagin's Minimum Principle)를 이용해 해석적으로 구한다. 이 문제는 2점 경계값 문제(Two-Point Boundary Value Problem, TPBVP)로 귀결되며, 최적 시간 $T^*$와 그에 따른 최적 비용 $J^*(T^*)$을 계산할 수 있다. 이 $J^*(T^*)$가 바로 휴리스틱 비용 $h_c$가 된다. 이 휴리스틱은 장애물을 무시한 상태에서의 최적 비용이므로, 실제 최적 비용보다 항상 작거나 같아 허용 가능(admissible)하다.

### 2.4 C++ 구현 분석 (`kinodynamic_astar.cpp`)

소스코드 분석을 통해 이론적 개념들이 실제 어떻게 구현되었는지 파악할 수 있다.

- **클래스 및 데이터 구조:**
  - `KinodynamicAstar`: 탐색 알고리즘의 메인 클래스로, `search()`, `setParam()`, `getKinoTraj()` 등의 멤버 함수를 가진다.6
  - `PathNode`: 탐색 트리의 각 노드를 나타내는 구조체. 상태 벡터(`state`, 위치/속도), 부모 노드 포인터(`parent`), 비용(`g_score`, `f_score`), 노드 상태(`IN_OPEN_SET`, `IN_CLOSE_SET`) 등의 정보를 포함한다.6
  - `open_set_`: 방문할 노드들을 담는 우선순위 큐(priority queue). `f_score`가 가장 낮은 노드가 항상 top에 위치하여 탐색의 효율을 높인다.8
  - `expanded_nodes_`: 이미 확장된 노드들을 저장하는 해시맵 또는 이와 유사한 자료구조. 중복 탐색을 방지하여 알고리즘의 종료를 보장한다.6
- **`search()` 함수 핵심 로직:**
  1. **초기화:** 시작 노드를 생성하고 `g_score=0`으로 설정한다. `h_score`를 계산하여 `f_score`를 구한 뒤 `open_set_`에 삽입한다.8
  2. 메인 루프 (while (!open_set_.empty())):
     - a. open_set_에서 f_score가 가장 낮은 노드(cur_node)를 꺼내고, expanded_nodes_(Close Set)에 추가한다.8
     - b. 종료 조건 검사: cur_node가 목표 지점에 충분히 가깝거나(near_end), 탐색 범위를 벗어났거나(reach_horizon), 또는 cur_node에서 목표 지점까지 장애물 없는 분석적 궤적 생성이 가능한지(is_shot_succ_) 등을 종합적으로 판단하여 탐색을 종료한다.6
     - c. 노드 확장 (Node Expansion): getSucc()와 같은 함수를 호출하여 cur_node에 이산화된 제어 입력들을 적용해 여러 후속 노드(pro_node)를 생성한다. 각 pro_node에 대해 충돌 검사, 최대 속도/가속도 검사, 비용 계산을 수행하고, 유효한 노드만 open_set_에 추가한다.6
  3. **경로 역추적 (`retrievePath`):** 탐색이 성공적으로 종료되면, `terminate_node`에서부터 `parent` 포인터를 따라 시작 노드까지 거슬러 올라가 최종 경로를 생성한다.8

### 2.5 `KinodynamicAstar`의 주요 파라미터 및 역할

`KinodynamicAstar`의 성능은 여러 파라미터에 의해 결정되며, 이를 적절히 튜닝하는 것이 중요하다.

| 파라미터 (Parameter)      | 코드/YAML 변수명 (Variable Name)         | 설명 (Description)                                           | 튜닝 영향 (Tuning Impact)                                    | 관련 자료 |
| ------------------------- | ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------- |
| Heuristic Weight          | `lambda_heu_`                            | 휴리스틱 비용(h-cost)에 곱해지는 가중치. 탐색의 탐욕도(greediness)를 조절. | 값이 크면 목표 지향성이 강해져 탐색 속도가 빨라질 수 있으나, 최적성 보장이 약해짐. 값이 작으면 더 넓은 영역을 탐색하여 최적해를 찾을 확률이 높아지나 느려짐. | 8         |
| Max Velocity/Acceleration | `max_vel_`, `max_acc_`                   | 쿼드콥터의 최대 속도 및 가속도. 동역학적 제약 조건.          | 로봇의 물리적 한계에 맞게 설정해야 함. 너무 높게 설정하면 실행 불가능한 궤적이 생성되고, 너무 낮으면 로봇의 성능을 최대로 활용하지 못함. | 6         |
| Search Horizon            | `horizon_`                               | 탐색을 진행할 최대 거리. 이 거리를 넘어서면 탐색을 중단.     | 로컬 리플래닝(local replanning) 시나리오에서 유용. 너무 작으면 목표에 도달하기 전에 탐색이 끝나고, 너무 크면 불필요한 계산을 유발할 수 있음. | 6         |
| Goal Tolerance            | `tolerance`                              | 목표 지점에 얼마나 가까워져야 도달한 것으로 간주할지에 대한 허용 오차. | 값이 너무 작으면 정확한 지점에 도달하기 어려워 탐색이 실패할 수 있음. 너무 크면 목표에서 먼 곳에서 궤적이 끝날 수 있음. | 6         |
| Time Penalty              | `rho` (수식), `tie_breaker_` (코드 추정) | 비용 함수에서 시간에 가해지는 페널티. 동일한 제어 비용을 가질 때 더 짧은 시간의 경로를 선호하도록 함. | 값이 클수록 최단 시간 경로를 찾는 경향이 강해짐. 제어 에너지와 시간 사이의 트레이드오프를 조절. | 2         |
| Discretization            | `ctrl_num`, `tau` (추정)                 | 제어 입력과 시간의 이산화 정도.                              | 이산화가 세밀할수록 더 다양한 Motion Primitive를 생성하여 최적해에 가까워질 수 있으나, 탐색 공간이 커져 계산량이 폭발적으로 증가함. | 2         |

A* 탐색의 효율성은 휴리스틱 함수의 품질에 결정적으로 의존하며, 최적해를 보장하기 위한 조건은 휴리스틱이 실제 비용을 과대평가하지 않는 허용 가능성(admissibility)이다.14 단순한 그리드 기반 경로 탐색에서는 유클리드 거리가 효과적인 휴리스틱이지만, 동역학을 고려하는 문제에서는 기하학적으로 짧은 경로가 동역학적으로는 매우 비싸거나 불가능할 수 있다. 

`Fast-Planner` 개발자들은 이 문제를 인식하고, "현재 상태에서 목표까지 장애물을 무시하고 도달하는 최적 궤적의 비용"을 휴리스틱으로 사용했다.2 이는 허용 가능한 휴리스틱을 생성하기 위한 전형적인 "완화된 문제(relaxed problem)" 접근법이다.16 이 완화된 문제를 풀기 위해 폰트랴긴의 최소 원리와 같은 고등 최적 제어 이론이 필요하며, 이는 노드 하나를 평가하는 데 상당한 계산량을 요구한다. 하지만 그 대가로 얻는 것은 매우 정확한 비용 추정치이며, 이는 탐색을 목표 방향으로 강력하게 이끄는 구동력이 된다. 이러한 설계는 각 노드에서 휴리스틱 계산에 더 많은 비용을 투자하여 전체적으로 탐색해야 할 노드의 수를 극적으로 줄이는 전략적 트레이드오프를 보여준다. 이것이 고차원 상태 공간에서 `KinodynamicAstar`가 빠르고 준최적의 해를 찾을 수 있게 하는 핵심적인 '지능'이다.

## 3. 위상학적 경로 탐색

### 3.1 도입 동기 및 문제 정의

`KinodynamicAstar`와 같은 단일 경로 탐색 알고리즘은 복잡하거나 장애물이 밀집한 환경에서 국소 최적해(local minima)에 빠지기 쉽다. 예를 들어, 큰 장애물의 왼쪽으로 초기 경로가 생성되면, 백엔드 최적화기는 경로를 왼쪽 공간 내에서만 개선할 뿐, 실제로는 더 짧을 수 있는 오른쪽 경로를 발견할 수 없다.4 이 문제를 해결하기 위해, `Fast-Planner`는 위상학적으로 구별되는(topologically distinctive) 여러 경로를 탐색하는 전략을 도입했다.1

### 3.2 핵심 아이디어 및 구현 추정

위상학적으로 구별되는 경로는 수학적으로 호모토피 클래스(homotopy class) 개념과 관련이 있다. 두 경로가 장애물을 통과하지 않고 연속적으로 변형되어 서로 같아질 수 있다면 동일한 호모토피 클래스에 속한다.17 위상학적 경로 탐색의 목표는 서로 다른 호모토피 클래스에 속하는 경로들을 찾는 것이다. 

`Fast-Planner`의 `README`는 이 기능을 "sampling-based topological path searching algorithm"이라고 명시하고 있어 1, RRT(Rapidly-exploring Random Tree)와 같은 샘플링 기반 알고리즘을 변형하여 구현되었을 가능성이 높다.17 구현은 공간에 무작위로 샘플을 생성하고, 가시성 그래프(Visibility Graph) 등을 이용해 연결 관계를 파악한 뒤, 장애물을 기준으로 서로 다른 "쪽"으로 지나가는 경로들을 여러 개 추출하는 방식으로 이루어질 수 있다.19

### 3.3 Path-Guided Optimization (PGO)

위상학적 탐색으로 얻은 각 경로는 백엔드 최적화기(`bspline_opt`)를 위한 '가이드' 역할을 한다. 최적화 과정에서 궤적이 이 가이드 경로에서 너무 멀리 벗어나지 않도록 제약 조건을 추가함으로써, 각 최적화가 해당 위상학적 클래스 내에서 해를 찾도록 강제한다. 이를 PGO(Path-Guided Optimization)라고 한다.4 최종적으로, 여러 위상학적 클래스에서 독립적으로 최적화된 궤적들 중에서 가장 비용이 낮은 궤적을 최종 해로 선택한다.

위상학적 경로 탐색의 통합은 플래너의 철학을 근본적으로 변화시킨다. 이는 단일 가설 접근법("찾아낸 하나의 좋은 경로를 개선하자")에서 다중 가설 접근법("구조적으로 다른 여러 좋은 경로를 찾아서 비교하자")으로의 전환을 의미한다. 이는 마치 인간 운전자가 원형 교차로에 접근할 때 왼쪽으로 갈 경우와 오른쪽으로 갈 경우를 모두 머릿속으로 고려한 후 결정하는 것과 유사하다. 표준적인 `KinodynamicAstar` -> `bspline_opt` 파이프라인은 "A*가 찾은 초기 경로가 근본적으로 옳다"는 단일 가설에 의존한다. 이 가설은 U자형 장애물과 같이 비볼록성이 큰 환경에서는 실패할 수 있다.4 위상학적 탐색기는 이 단계를 변경하여 "가설 A: 이 장애물의 왼쪽이 최선이다. 가설 B: 오른쪽이 최선이다."와 같이 경쟁하는 여러 가설을 생성한다. 각 가설(가이드 경로)은 검증과 개선을 위해 백엔드 최적화기로 전달된다. 최종적으로 가장 비용이 낮은 궤적을 선택하는 것은 가장 성공적으로 검증된 가설을 채택하는 것과 같다. 이처럼 솔루션 공간을 병렬적으로 탐색함으로써 플래너는 국소 최적해에 빠질 가능성이 훨씬 낮아진다. 이는 도전적인 시나리오에서 솔루션의 품질과 강건성을 우선시하는 성숙한 설계 철학을 반영한다.

## 4. 결론 및 종합

`Fast-Planner`의 `path_searching` 모듈은 단순한 경로 탐색기를 넘어, 다양한 환경과 요구사항에 대응하기 위한 다층적이고 정교한 프론트엔드 시스템이다. **기본적**으로는 표준 A*가 연결성 탐색을 제공하고, **핵심**적으로는 `KinodynamicAstar`가 동역학적으로 실행 가능한 궤적을 보장하며 정교한 휴리스틱 설계를 통해 효율성을 극대화했다. 여기에 **강건성**을 더하기 위해 위상학적 경로 탐색을 도입하여 복잡한 환경에서의 국소 최적해 문제를 해결하고 시스템의 신뢰도를 한 차원 높였다.

이 모듈은 '계층적 분할(Hierarchical Decomposition)', '휴리스틱을 통한 탐색 가속(Heuristic-Guided Search)', '다중 가설을 통한 강건성 확보(Robustness through Multiple Hypotheses)'라는 세 가지 핵심 설계 철학을 명확히 보여준다. `path_searching` 모듈은 불확실하고 제약이 많은 탐색 공간에서 양질의 '씨앗'(초기 궤적)을 찾아내는 역할을 성공적으로 수행한다. 이 씨앗의 품질이 후속 `bspline_opt` 최적화의 성공과 효율을 결정하므로, `Fast-Planner`가 이름처럼 '빠르고(Fast)' 안정적인 계획을 수행할 수 있는 근간을 제공한다. `plan_env` → `path_searching` → `bspline_opt`로 이어지는 데이터와 역할의 명확한 흐름은 `Fast-Planner`를 매우 모듈화되고 확장 가능한 강력한 프레임워크로 만든다.

#### 참고 자료

1. HKUST-Aerial-Robotics/Fast-Planner: A Robust and Efficient Trajectory Planner for Quadrotors - GitHub, accessed August 11, 2025, https://github.com/HKUST-Aerial-Robotics/Fast-Planner
2. Robust and Efficient Quadrotor Trajectory Generation for Fast Autonomous Flight - arXiv, accessed August 11, 2025, https://arxiv.org/pdf/1907.01531
3. README.md - HKUST-Aerial-Robotics/TopoTraj - GitHub, accessed August 11, 2025, https://github.com/HKUST-Aerial-Robotics/TopoTraj/blob/master/README.md
4. Robust Real-time UAV Replanning Using Guided Gradient-based Optimization and Topological Paths | Request PDF - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/338292412_Robust_Real-time_UAV_Replanning_Using_Guided_Gradient-based_Optimization_and_Topological_Paths
5. Robust and Efficient Trajectory Replanning Based on Guiding Path for Quadrotor Fast Autonomous Flight - Semantic Scholar, accessed August 11, 2025, https://pdfs.semanticscholar.org/da41/cd8a1d1ab883df3ff58d2c9e05764079c622.pdf
6. Fast-Planner/fast_planner/path_searching/src/kinodynamic_astar.cpp at master - GitHub, accessed August 11, 2025, https://github.com/HKUST-Aerial-Robotics/Fast-Planner/blob/master/fast_planner/path_searching/src/kinodynamic_astar.cpp
7. Easy A* (star) Pathfinding. Today we'll being going over the A*… | by Nicholas Swift, accessed August 11, 2025, https://medium.com/@nicholas.w.swift/easy-a-star-pathfinding-7e6689c7f7b2
8. FAST PLANNER代码阅读笔记原创 - CSDN博客, accessed August 11, 2025, https://blog.csdn.net/m0_62664599/article/details/125495461
9. ZJU-FAST-Lab/ego-planner - GitHub, accessed August 11, 2025, https://github.com/ZJU-FAST-Lab/ego-planner
10. EGO-Planner: An ESDF-free Gradient-based Local Planner for Quadrotors - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/348028324_EGO-Planner_An_ESDF-free_Gradient-based_Local_Planner_for_Quadrotors
11. Kinodynamic Motion Planning1 - Duke Computer Science, accessed August 11, 2025, https://users.cs.duke.edu/~brd/papers/src-papers/jacm-final.pdf
12. (PDF) Kinodynamic Motion Planning. - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/220431834_Kinodynamic_Motion_Planning
13. Robust Planning System for Fast Autonomous Flight in Complex Unknown Environment Using Sparse Directed Frontier Points - MDPI, accessed August 11, 2025, https://www.mdpi.com/2504-446X/7/3/219
14. Admissible heuristic - Wikipedia, accessed August 11, 2025, https://en.wikipedia.org/wiki/Admissible_heuristic
15. Admissibility of A* Algorithm - GeeksforGeeks, accessed August 11, 2025, https://www.geeksforgeeks.org/dsa/a-is-admissible/
16. Lecture 5: The ”animal kingdom” of heuristics: Admissible, Consistent, zero, Relaxed, Dominant, accessed August 11, 2025, https://courses.grainger.illinois.edu/ece448/sp2020/slides/lec05.pdf
17. Topology-Aware Efficient Path Planning in Dynamic Environments - MDPI, accessed August 11, 2025, https://www.mdpi.com/2075-1702/13/1/14
18. Robotic Path Planning - MIT Fab Lab, accessed August 11, 2025, https://fab.cba.mit.edu/classes/865.21/topics/path_planning/robotic.html
19. Topological Path Planning For Crowd Navigation - Carnegie Mellon University Robotics Institute, accessed August 11, 2025, https://www.ri.cmu.edu/app/uploads/2019/05/thesis.pdf
20. FPS: Fast Path Planner Algorithm Based on Sparse Visibility Graph and Bidirectional Breadth-First Search - MDPI, accessed August 11, 2025, https://www.mdpi.com/2072-4292/14/15/3720
21. Robust and Efficient Trajectory Replanning Based on Guiding Path for Quadrotor Fast Autonomous Flight - MDPI, accessed August 11, 2025, https://www.mdpi.com/2072-4292/13/5/972
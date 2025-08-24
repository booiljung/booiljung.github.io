[모션 계획 (Motion Planning)](./index.md)
# ROS2 Humble 기반 CHOMP



로봇 모션 플래닝 분야는 오랫동안 샘플링 기반의 방법론, 특히 OMPL(Open Motion Planning Library)로 대표되는 알고리즘들이 지배해왔다. 이러한 접근법은 로봇의 고차원 설정 공간(Configuration Space, C-Space)을 무작위로 탐색하여 장애물과 충돌하지 않는 연결 가능한 경로를 효율적으로 찾아내는 데 탁월한 성능을 보였다.1 그러나 샘플링 기반 플래너가 생성한 경로는 본질적으로 기하학적 연결성에만 초점을 맞추기 때문에, 종종 로봇이 실행하기에는 부자연스럽고 급격한 움직임(jerky motion)을 포함하는 단점이 있다.3 이로 인해 실제 로봇에 적용하기 위해서는 경로를 평활화(smoothing)하고 최적화하는 별도의 후처리(post-processing) 과정이 필수적으로 요구되었다. 가장 널리 사용되는 후처리 기법 중 하나는 '지름길 휴리스틱(shortcut heuristic)'으로, 경로상의 두 점을 선택하여 그 사이를 더 짧은 직선 경로로 대체하려는 시도를 반복하는 방식이다.4 이와 같은 2단계 접근법(탐색 후 최적화)은 우선 실행 가능한 경로를 찾고, 그 이후에 경로의 품질을 개선하는 방식으로 작동한다.

CHOMP(Covariant Hamiltonian Optimization for Motion Planning)의 등장은 이러한 전통적인 패러다임에 근본적인 질문을 던졌다. CHOMP는 경로 탐색과 최적화를 분리된 단계로 보지 않고, 처음부터 로봇의 동역학, 경로의 평활도, 그리고 장애물 회피와 같은 다양한 요구사항을 단일한 수학적 목적 함수(objective function)에 통합하여 '최적의' 궤적을 직접 생성하는 것을 목표로 한다.2 이는 단순히 '실행 가능한(feasible)' 경로를 찾는 것을 넘어, '최적의(optimal)' 움직임을 직접적으로 구축하려는 철학적 전환을 의미한다.3 CHOMP의 관점에서 로봇의 움직임은 단순한 점들의 나열이 아닌, 시간의 흐름에 따른 연속적인 함수, 즉 궤적(trajectory)으로 정의된다. 따라서 모션 플래닝 문제는 기하학적인 경로 찾기(pathfinding) 문제에서 동적인 궤적 생성(trajectory generation) 문제로 재정의된다. 이러한 접근법은 제어 이론 분야의 최적 제어(optimal control) 문제와 깊은 연관성을 가지며, 로봇의 움직임을 보다 물리적이고 자연스럽게 모델링할 수 있는 길을 열었다.


CHOMP의 핵심 작동 원리는 함수형 경사 하강법(functional gradient techniques)을 이용한 반복적인 궤적 개선이다.4 이 방법론에서 궤적 $\xi$는 시간을 입력받아 로봇의 설정(configuration)을 출력하는 함수 $\xi(t)$로 간주된다. CHOMP는 이러한 궤적 함수 자체를 변수로 취급하는 범함수(functional) $U[\xi]$를 정의하고, 이 범함수의 값을 최소화하는 방향으로 궤적을 점진적으로 수정해 나간다.4 이 범함수는 로봇 궤적의 평활도(smoothness)와 장애물로부터의 안전성(obstacle avoidance)이라는 두 가지 상충하는 목표 사이의 절충안을 나타내도록 설계되었다.4 이 모든 과정의 기저에는 로봇의 기구학 및 환경 정보로부터 경사(gradient)를 효율적으로 계산할 수 있다는 핵심 전제가 깔려 있다.4

CHOMP가 기존의 많은 궤적 최적화 기법들과 구별되는 중요한 특징 중 하나는, 최적화의 시작점으로 반드시 충돌이 없는 '실행 가능한(feasible)' 초기 궤적을 요구하지 않는다는 점이다. 복잡하고 비좁은 환경에서는 실행 가능한 초기 경로를 찾는 것 자체가 어려운 문제일 수 있다.3 CHOMP는 이러한 문제를 시작점과 목표점을 잇는 단순한 직선과 같이, 장애물과 충돌할 가능성이 높은 비현실적인(infeasible) 궤적으로부터 최적화를 시작할 수 있도록 설계되었다.6 이는 CHOMP의 장애물 비용 함수가 작업 공간(workspace)에 정의된 거리 필드(distance field)와 그 경사 정보를 활용하기에 가능하다. 초기 궤적이 장애물과 충돌하더라도, 이 경사 정보는 궤적을 장애물로부터 '밀어내는' 힘으로 작용하여 반복적인 업데이트를 통해 점차 충돌이 없는 안전한 영역으로 궤적을 유도한다.4 이처럼 CHOMP는 최적화 과정 자체가 탐색의 역할을 겸하는 '최적화를 통한 탐색(search via optimization)'이라는 강력하고 실용적인 접근법을 제시한다.


CHOMP라는 이름에 포함된 '공변(Covariant)'이라는 용어는 이 알고리즘의 가장 중요한 수학적 특성 중 하나인 재매개변수화 불변성(reparametrization invariance)을 의미한다.4 이는 궤적을 어떻게 표현하는지(예: 웨이포인트의 개수, 시간 간격 등)에 관계없이 최적화 알고리즘의 동작이 본질적으로 동일해야 한다는 원칙이다. CHOMP의 철학에 따르면, 궤적은 특정 매개변수화에 종속되지 않는 순수한 기하학적 객체 그 자체로 다루어져야 한다.4

표준적인 경사 하강법은 파라미터 공간에서 유클리드 거리(Euclidean distance)를 기준으로 최적화를 수행하기 때문에 이러한 불변성을 만족하지 못한다.4 예를 들어, 궤적의 특정 구간에 웨이포인트를 더 조밀하게 배치하면 해당 구간의 파라미터 수가 늘어나고, 결과적으로 경사 하강법 업데이트가 그 구간에 더 큰 영향을 미치게 된다. 이는 물리적으로 동일한 궤적이라도 이산화 방식에 따라 최적화 결과가 달라질 수 있음을 의미한다.

CHOMP는 이 문제를 해결하기 위해 리만 기하학(Riemannian geometry)의 개념을 도입한다. 궤적 공간에 유클리드 메트릭 대신, 궤적의 동역학적 특성(예: 총 가속도)을 기반으로 하는 리만 메트릭(Riemannian metric)을 정의한다.3 이 메트릭 하에서 계산된 경사를 '공변 경사(covariant gradient)'라 하며, 이는 궤적의 표현 방식에 무관한, 기하학적으로 올바른 업데이트 방향을 제시한다. 결과적으로 CHOMP의 업데이트 규칙은 궤적에 가해지는 업데이트의 '크기'를 파라미터의 변화량이 아닌, 궤적의 평활도 변화량으로 측정하게 된다.3 이 공변성은 알고리즘의 안정성과 예측 가능성을 보장하는 핵심적인 공학적 장치로, 개발자가 궤적의 이산화 방식에 대해 크게 우려하지 않고도 일관된 최적화 결과를 얻을 수 있도록 한다.


대부분의 실제 로봇 모션 플래닝 문제는 장애물 등으로 인해 목적 함수가 비볼록(non-convex) 형태를 띤다. 이는 경사 하강법 기반의 최적화 알고리즘이 전역 최적해(global minimum)가 아닌 지역 최적해(local minimum)에 수렴할 위험이 크다는 것을 의미한다.10 CHOMP는 이러한 근본적인 한계를 완화하기 위한 전략으로 해밀토니안 몬테카를로(Hamiltonian Monte Carlo, HMC) 기법을 채택한다.4

HMC는 경사 하강법을 통해 하나의 지역 최솟값에 도달한 궤적을 그 상태에 머무르게 하지 않고, 물리계의 해밀토니안 동역학을 모사하여 확률적인 탐색을 수행한다. 이 과정에서 현재의 궤적은 위치 에너지(potential energy)가 낮은 상태에 있는 '입자'로 간주되며, 여기에 무작위적인 '운동량(momentum)'을 부여한다. 이 운동량은 입자가 현재 머무르고 있는 에너지 우물(potential well)의 장벽을 넘어 다른, 더 낮은 에너지 상태의 우물로 이동할 수 있는 기회를 제공한다.4 이는 단순히 다른 지점에서 최적화를 재시작하는 무작위 재시작(random restart) 방식보다 훨씬 효율적으로 해 공간을 탐색하는 방법이다. HMC의 도입은 결정론적인 최적화 알고리즘인 CHOMP에 확률적 탐색 능력을 부여하며, 이론적으로는 확률적 완전성(probabilistic completeness)을 만족하게 만든다.4 이는 순수한 최적화 기법의 한계를 인정하고, 샘플링 기반 방법의 장점인 전역적 탐색 능력을 실용적으로 결합하려는 시도로 볼 수 있다.



CHOMP의 핵심은 궤적의 품질을 정량화하는 목적 범함수(objective functional) $U[\xi]$에 있다. 이 범함수는 평활도 비용 범함수 $U_{smooth}[\xi]$와 장애물 비용 범함수 $U_{obs}[\xi]$의 가중합으로 정의된다.12
$$
U[\xi] = U_{obs}[\xi] + \lambda U_{smooth}[\xi]
$$
여기서 $\xi$는 시간에 대한 궤적 함수 $\xi(t)$를, $\lambda$는 평활도와 장애물 회피 사이의 중요도를 조절하는 가중치 매개변수를 나타낸다.


평활도 비용 함수는 궤적의 동역학적 부드러움을 측정하는 역할을 한다. 이는 물리적으로 로봇이 급격한 속도나 가속도 변화 없이 움직이도록 유도한다. 일반적으로 이 비용은 궤적의 시간에 대한 미분값의 $L_2`-norm` 제곱을 적분하여 계산된다.3 CHOMP의 원 논문에서는 주로 궤적의 가속도에 해당하는 2차 미분값을 사용하여 평활도를 정의한다.4
$$
U_{smooth}[\xi] = \frac{1}{2} \int_{0}^{1} \left\| \frac{d^2\xi(t)}{dt^2} \right\|^2 dt
$$
실제 구현에서는 궤적을 이산적인 웨이포인트(waypoint)의 시퀀스 $\xi = [q_1, q_2,..., q_n]^T$로 표현한다. 이 경우, 적분은 합산으로, 미분은 유한 차분(finite differencing)으로 근사된다. 유한 차분 연산은 행렬 $K$로 표현될 수 있으며, 이를 통해 평활도 비용은 궤적 벡터 $\xi$에 대한 2차 형식(quadratic form)으로 간결하게 나타낼 수 있다.3
$$
U_{smooth}(\xi) = \frac{1}{2} \xi^T \mathbf{A} \xi + \mathbf{b}^T \xi + c
$$
여기서 행렬 $\mathbf{A}$는 유한 차분 행렬 $K$로부터 $\mathbf{A} \propto K^T K$와 같이 계산되며, 벡터 $\mathbf{b}$와 스칼라 $c$는 고정된 시작점과 끝점의 영향을 반영하는 항이다. 이렇게 구성된 행렬 $\mathbf{A}$는 항상 대칭 양정부호 행렬(symmetric positive definite)의 특성을 가지며, 이후 설명할 공변 경사 하강법에서 리만 메트릭으로 사용되는 핵심적인 역할을 수행한다.3


장애물 비용 함수는 로봇이 환경 내의 장애물과 충돌하지 않고 안전한 거리를 유지하도록 강제한다. CHOMP는 이 비용을 계산하기 위해 로봇의 고차원 설정 공간(C-space) 대신, 직관적이고 계산적으로 효율적인 3차원 작업 공간(workspace)을 활용하는 혁신적인 접근법을 취한다.4

이 비용 함수는 로봇 몸체를 구성하는 모든 지점($u \in B$)이 궤적을 따라 움직이면서 작업 공간 상의 비용 필드 $c(\mathbf{x})$를 통과할 때 누적되는 총비용으로 정의된다. 특히, 로봇이 위험 지역을 빠르게 통과하여 비용을 인위적으로 낮추는 것을 방지하기 위해, 각 지점의 작업 공간 속도($\left| \frac{d}{dt}\mathbf{x}(\xi(t), u) \right|$)를 비용에 곱해줌으로써 재매개변수화에 불변인 형태로 정식화되었다.4
$$
U_{obs}[\xi] = \int_{0}^{1} \int_{B} c(\mathbf{x}(\xi(t), u)) \left\| \frac{d}{dt}\mathbf{x}(\xi(t), u) \right\| du dt
$$
여기서 $\mathbf{x}(\xi(t), u)$는 시간 $t$에 로봇이 형상 $\xi(t)$에 있을 때, 로봇 몸체 위의 점 $u$가 작업 공간에서 차지하는 위치 벡터이다. 작업 공간 비용 $c(\mathbf{x})$는 일반적으로 장애물까지의 부호 있는 거리 필드(Signed Distance Field, SDF) $d(\mathbf{x})$를 기반으로 정의된다. SDF는 특정 지점이 장애물 내부에 있으면 음수, 외부에 있으면 양수의 거리 값을 가지며, 경계에서는 0이 된다. 비용 함수 $c(\mathbf{x})$는 이 SDF를 이용하여, 장애물 경계로부터 특정 임계 거리 $\epsilon$ 이내에 있을 때만 0보다 큰 값을 갖고, 장애물에 가까워질수록 급격히 증가하는 형태로 설계된다.3
$$
c(\mathbf{x}) =
\begin{cases}
-d(\mathbf{x}) + \frac{1}{2}\epsilon & \text{if } d(\mathbf{x}) < 0 \\
\frac{1}{2\epsilon}(d(\mathbf{x}) - \epsilon)^2 & \text{if } 0 \le d(\mathbf{x}) \le \epsilon \\
0 & \text{otherwise}
\end{cases}
$$
이러한 SDF 기반 접근법은 최근 급격히 발전한 유클리드 거리 변환(Euclidean Distance Transform, EDT) 알고리즘을 활용하여, 복잡한 3D 환경에서도 실시간에 가깝게 거리 필드와 그 경사를 효율적으로 계산할 수 있게 해준다.4

이처럼 평활도 비용과 장애물 비용은 궤적의 서로 다른 측면을 제어하며 상호 보완적으로 작용한다. 장애물 비용 함수는 속도 불변성으로 인해 주로 궤적의 기하학적 '모양'을 결정하는 반면, 평활도 비용 함수는 동역학적 특성을 최소화함으로써 궤적의 '시간적 프로파일'을 제어한다.4 사용자는 가중치 $\lambda$를 조절하여 이 두 요소 간의 균형을 맞춤으로써 원하는 특성의 궤적을 생성할 수 있다.


CHOMP는 위에서 정의된 목적 범함수 $U[\xi]$를 최소화하기 위해 공변 경사 하강법이라는 최적화 기법을 사용한다. 이는 표준적인 경사 하강법을 리만 기하학의 관점에서 일반화한 것으로, 궤적의 표현 방식에 무관한 업데이트를 보장한다. $k$번째 반복에서의 궤적 $\xi_k$가 주어졌을 때, 다음 궤적 $\xi_{k+1}$은 다음과 같은 업데이트 규칙에 따라 계산된다.3
$$
\xi_{k+1} = \xi_k - \frac{1}{\eta} \mathbf{M}^{-1} \nabla U(\xi_k)
$$
여기서 $\eta$는 학습률(learning rate)과 관련된 스텝 사이즈 상수, $\nabla U(\xi_k)$는 $\xi_k$에서의 유클리드 경사(Euclidean gradient), 그리고 $\mathbf{M}$은 궤적 공간에 정의된 리만 메트릭(Riemannian metric)을 나타내는 행렬이다. CHOMP에서는 평활도 비용 함수에서 유도된 행렬 $\mathbf{A}$를 이 메트릭으로 사용한다($\mathbf{M} = \mathbf{A}$).3 따라서, 전체 목적 함수 $U[\xi] = U_{obs}[\xi] + \lambda U_{smooth}[\xi]$를 고려한 최종 업데이트 규칙은 다음과 같이 표현된다.
$$
\xi_{k+1} = \xi_k - \alpha \mathbf{A}^{-1} (\nabla U_{obs}(\xi_k) + \lambda \nabla U_{smooth}(\xi_k))
$$
여기서 $\alpha$는 스텝 사이즈를 나타낸다. 이 수식에서 행렬 $\mathbf{A}$의 역행렬인 $\mathbf{A}^{-1}$의 존재가 이 업데이트 규칙을 표준 경사 하강법과 근본적으로 다르게 만드는 핵심 요소이다.3 행렬 

$\mathbf{A}$는 이산화된 궤적의 크기에 따라 매우 커질 수 있지만, 유한 차분 연산의 특성상 희소(sparse)하고 띠 대각(band diagonal) 구조를 가진다. 이 덕분에 $\mathbf{A}^{-1}\mathbf{g}$를 직접 계산하는 대신, 선형 시스템 $\mathbf{A}\mathbf{x} = \mathbf{g}$를 토마스 알고리즘과 같은 효율적인 방법으로 $O(n)$ 시간 복잡도 내에 풀 수 있다.4


표준 경사 하강법은 리만 메트릭 $\mathbf{M}$이 단위 행렬 $\mathbf{I}$인 특수한 경우에 해당한다. 이 경우 업데이트는 $\xi_{k+1} = \xi_k - \alpha \nabla U(\xi_k)$가 되며, 각 웨이포인트의 수정은 해당 지점의 국소적인 경사 정보에만 의존한다. 반면, CHOMP의 공변 경사 하강법에서 $\mathbf{A}^{-1}$ 항은 특정 지점에서 계산된 경사의 영향을 궤적 전체에 걸쳐 '전파'하고 '분배'하는 역할을 수행한다.3 이는 마치 궤적이라는 탄성 있는 밴드의 한 점을 잡아당겼을 때 밴드 전체가 부드럽게 변형되는 것과 유사하다. 이 '전파' 효과 덕분에 업데이트 이후에도 궤적은 전반적인 평활도를 유지하게 된다.3


리만 메트릭 $\mathbf{A}$는 궤적들의 공간에 내재된 기하학적 구조를 정의한다. 이 공간에서 두 궤적 사이의 '거리'는 단순히 웨이포인트들의 유클리드 거리($|\xi_1 - \xi_2|^2$)가 아니라, 그들의 동역학적 차이를 반영하는 거리 $(\xi_1 - \xi_2)^T \mathbf{A} (\xi_1 - \xi_2)$로 측정된다.3 공변 경사 하강법은 바로 이 기하학적으로 의미 있는 거리 개념 하에서 가장 가파른 하강 방향을 따르는 것으로 해석할 수 있다.14

이러한 접근은 최적화 문제를 평평한 유클리드 공간에서 푸는 것이 아니라, '평활한 궤적들'로 이루어진 휘어진 다양체(manifold) 상에서 푸는 것으로 재정의한다. 행렬 $\mathbf{A}$가 궤적의 평활도(가속도)를 측정하기 때문에, $\mathbf{A}^{-1}$를 경사에 곱하는 연산은 유클리드 경사를 이 다양체의 접평면에 '가장 평활한' 방향으로 투영하는 것과 같다. 이 방향은 자연 경사(Natural Gradient)라고도 불리며 14, 최적화 과정이 항상 평활한 궤적들의 공간을 벗어나지 않도록 강제하는 핵심 메커니즘으로 작용한다. 이는 알고리즘의 수렴 속도와 안정성을 크게 향상시키는 결과를 가져온다.3



ROS2 Humble과 통합된 MoveIt2 프레임워크 내에서 CHOMP는 핵심적인 모션 플래닝 옵션 중 하나로 제공된다. 이는 OMPL, STOMP, Pilz Industrial Motion Planner 등과 함께 플러그인 아키텍처를 통해 동적으로 로드될 수 있는 계획 파이프라인(planning pipeline)으로 구현되어 있다.2 사용자는 MoveIt2의 주 인터페이스 노드인 `move_group`에 모션 계획을 요청할 때, 사용할 플래너로 CHOMP를 지정할 수 있다. CHOMP 플러그인은 `planning_interface::PlannerManager`라는 MoveIt의 표준 인터페이스를 상속받아 구현되며, 구체적으로는 `chomp_interface/CHOMPPlanner`라는 이름으로 시스템에 등록된다.18

MoveIt2는 여러 플래너를 동시에 활용할 수 있는 병렬 계획 인터페이스(parallel planning interface)라는 강력한 기능을 도입했다.16 이 아키텍처 하에서 CHOMP는 단독으로 사용될 뿐만 아니라, OMPL과 같은 다른 플래너들과 병렬로 실행될 수 있다. 각 플래너는 동일한 계획 문제에 대해 독립적으로 해를 탐색하며, 시스템은 사전에 정의된 기준(예: 가장 짧은 경로 길이, 가장 빠른 계획 시간 등)에 따라 최종적으로 사용할 최상의 솔루션을 선택한다. 이는 특정 플래너가 실패하거나 비효율적인 해를 생성하는 경우에 대비한 강건성(robustness)을 높여주는 효과적인 전략이다.


CHOMP 플래너의 모든 동작은 로봇의 MoveIt 설정 패키지 내 `config` 디렉토리에 위치한 `chomp_planning.yaml` 파일을 통해 제어된다.1 이 YAML 파일은 CHOMP의 최적화 알고리즘에 직접적인 영향을 미치는 수십 개의 매개변수를 정의하는 공간이다. 주요 매개변수로는 최적화 시간제한(`planning_time_limit`), 최대 반복 횟수(`max_iterations`), 평활도와 장애물 비용 간의 가중치를 조절하는 `smoothness_cost_weight`와 `obstacle_cost_weight`, 그리고 최적화 스텝의 크기를 결정하는 `learning_rate` 등이 있다.6

이 매개변수들은 로봇의 종류, 작업 환경의 복잡도, 그리고 요구되는 작업의 성격에 따라 신중하게 조정되어야 한다. 예를 들어, 장애물이 밀집한 복잡한 환경에서 기본 매개변수 설정은 CHOMP가 쉽게 지역 최솟값에 빠져 계획에 실패하는 원인이 될 수 있다.6 따라서 성공적인 CHOMP 활용을 위해서는 이러한 매개변수들의 의미를 정확히 이해하고 체계적으로 튜닝하는 과정이 필수적이다. (각 매개변수에 대한 상세한 분석과 튜닝 가이드는 4부에서 심도 있게 다룬다.)


CHOMP는 독립적인 플래너로서의 역할 외에도, MoveIt의 플래닝 요청 어댑터(Planning Request Adapter)라는 강력한 기능을 통해 다른 플래너의 성능을 보완하는 후처리기(post-processor)로 활용될 수 있다.6 이 방식의 가장 대표적이고 효과적인 활용 사례는 OMPL과 CHOMP를 연계하는 것이다.

이 시나리오에서, OMPL은 먼저 복잡한 환경에서 빠르게 충돌 없는 초기 경로를 생성하는 역할을 담당한다. 그 후, 이 경로가 CHOMP 어댑터에 입력으로 전달된다. CHOMP는 OMPL이 생성한 경로를 초기 추측(initial guess)으로 사용하여, 이를 더욱 평활하고 동역학적으로 최적화된 궤적으로 다듬는다.6 이 접근법은 CHOMP의 가장 큰 약점인 지역 최솟값 문제를 완화하는 데 매우 효과적이다. 품질이 보장된 초기 경로에서 최적화를 시작함으로써, CHOMP가 나쁜 초기 추측으로 인해 비효율적인 지역 최솟값에 수렴할 위험을 크게 줄일 수 있다.6

이러한 파이프라인을 구성하기 위해서는 MoveIt 설정에서 몇 가지 변경이 필요하다. `ompl_planning.yaml`과 같은 설정 파일에서 `request_adapters` 목록에 `chomp/OptimizerAdapter`를 추가해야 한다. 또한, CHOMP가 외부에서 제공된 경로를 초기 궤적으로 사용하도록 지시하기 위해 `chomp_planning.yaml` 파일 내의 `trajectory_initialization_method` 매개변수를 `"fillTrajectory"`로 설정해야 한다.6


CHOMP의 장애물 회피 메커니즘은 환경에 대한 정밀한 거리 정보, 특히 부호 있는 거리 필드(Signed Distance Field, SDF)에 깊이 의존한다.4 MoveIt2는 3D 센서 데이터를 처리하고 이를 모션 플래닝에 활용하기 위한 통합된 인식 파이프라인(perception pipeline)을 제공하며, CHOMP는 이 파이프라인과 긴밀하게 연동된다.

이 파이프라인의 핵심은 OctoMap이다. MoveIt의 `PointCloudOctomapUpdater` 플러그인은 로봇에 장착된 깊이 카메라나 라이다와 같은 3D 센서로부터 실시간으로 들어오는 포인트 클라우드 데이터(`sensor_msgs/msg/PointCloud2`)를 수신한다.17 이 플러그인은 수신된 포인트들을 3차원 공간을 재귀적으로 분할하는 트리 구조인 옥트리(octree)에 통합하여, 환경에 대한 3D 점유 격자 지도(occupancy grid map)인 OctoMap을 생성하고 지속적으로 갱신한다.

이렇게 생성된 OctoMap은 CHOMP가 직접 사용하는 형태는 아니다. CHOMP는 각 지점에서의 거리 값과 경사(gradient)를 필요로 한다. MoveIt 프레임워크 내의 `distance_field` 라이브러리가 이 변환을 담당한다. 이 라이브러리는 OctoMap의 점유 정보를 바탕으로 SDF를 계산하고, 이를 통해 CHOMP가 최적화 과정에서 장애물 비용과 그 경사를 효율적으로 쿼리할 수 있도록 지원한다.24 MoveIt의 충돌 검사 환경 구현체 중 하나인 `CollisionWorldHybrid`는 빠른 이진 충돌 검사를 위한 FCL(Flexible Collision Library)과 정밀한 거리 정보 계산을 위한 거리 필드를 함께 사용하여, 다양한 플래너의 요구사항을 동시에 만족시킨다.26

이러한 아키텍처는 CHOMP의 성능이 플래너 자체의 튜닝뿐만 아니라, 상위 스트림인 인식 파이프라인의 설정에 매우 민감하게 의존한다는 중요한 사실을 시사한다. 예를 들어, OctoMap의 해상도(`octomap_resolution`)가 너무 낮거나, 센서 데이터의 노이즈가 심하면 부정확한 SDF가 생성될 수 있다. 부정확한 SDF는 장애물의 경계를 왜곡하거나, 실제로는 비어있는 공간에 '환영' 장애물을 만들어낼 수 있다. 이러한 잘못된 환경 정보는 CHOMP가 최적의 경로를 찾는 것을 방해하거나, 심지어 충돌하는 경로를 생성하게 만드는 직접적인 원인이 될 수 있다. 따라서 CHOMP의 `obstacle_cost_weight`와 같은 매개변수를 튜닝하는 작업은 OctoMap의 해상도나 센서 데이터 필터링과 같은 인식 파이프라인의 매개변수 튜닝과 함께, 시스템 전체적인 관점에서 통합적으로 이루어져야 한다.



CHOMP의 성능과 행동은 `chomp_planning.yaml` 파일에 정의된 여러 매개변수에 의해 결정된다. 성공적인 계획을 위해서는 이들 매개변수의 역할과 상호작용을 이해하는 것이 필수적이다. 아래 표는 가장 핵심적인 매개변수들과 그에 대한 튜닝 가이드를 요약한 것이다.6

**Table 1: `chomp_planning.yaml` 핵심 매개변수 가이드**

| 매개변수 (Parameter)      | 설명 (Description)                                           | 일반적 값/범위 | 튜닝 영향 (Impact of Tuning)                                 | 관련 자료 |
| ------------------------- | ------------------------------------------------------------ | -------------- | ------------------------------------------------------------ | --------- |
| `obstacle_cost_weight`    | 장애물 비용의 가중치. 높을수록 장애물 회피를 우선시한다.     | 0.0 ~ 1.0      | **증가:** 안전 여유 증가, 과도할 경우 해 찾기 실패. **감소:** 충돌 위험 증가. | 6         |
| `smoothness_cost_weight`  | 평활도 비용의 가중치. 높을수록 부드러운 궤적을 생성한다.     | 1e-4 ~ 1.0     | **증가:** 더 부드러운 움직임, 장애물 회피를 위해 경로가 길어질 수 있음. **감소:** 더 짧고 각진 움직임. | 6         |
| `learning_rate`           | 최적화 스텝 크기.                                            | 0.01 ~ 10.0    | **증가:** 빠른 수렴, 발산 위험. **감소:** 안정적 수렴, 느린 속도. | 6         |
| `ridge_factor`            | 비용 행렬에 추가되는 노이즈. 수치 안정성 및 지역 최솟값 탈출에 기여. | 0.0 ~ 0.01     | **증가:** 장애물 회피 능력 향상, 평활도 저하. 0.001 이상 권장.19 | 6         |
| `max_iterations`          | 최대 반복 횟수.                                              | 100 ~ 1000     | **증가:** 더 나은 해를 찾을 가능성 증가, 계획 시간 증가.     | 6         |
| `enable_failure_recovery` | 초기 실패 시 매개변수를 자동으로 변경하여 재시도할지 여부.   | true/false     | **true:** 성공률 향상 가능성. **false:** 예측 가능한 실패.   | 6         |


CHOMP의 가장 근본적인 약점은 경사 하강법에 내재된 지역 최솟값(local minima) 문제에 취약하다는 것이다.29 이 문제는 특히 장애물이 밀집해 있거나 좁은 통로를 통과해야 하는 복잡한(cluttered) 환경에서 두드러지게 나타난다.6 기하학적으로 이는 궤적이 두 장애물 사이에 형성된 '골짜기'에 갇히는 상황으로 비유할 수 있다. 궤적은 이 골짜기 내에서는 최적의 위치에 있을지 모르지만, 장애물을 크게 우회하는 더 나은 전역 최적해가 존재함에도 불구하고 경사 정보만으로는 이 골짜기를 탈출하지 못한다.3

이러한 실패 모드에 대처하기 위한 몇 가지 효과적인 전략은 다음과 같다.

1. **초기 궤적 개선:** 지역 최솟값 문제를 해결하는 가장 강력한 방법은 최적화의 시작점을 좋은 해에 가깝게 설정하는 것이다. OMPL과 같은 샘플링 기반 플래너를 사용하여 충돌 없는 초기 경로를 생성하고 이를 CHOMP의 입력으로 제공하는 것은 매우 효과적인 전략이다.6 이는 CHOMP가 전역적인 탐색을 건너뛰고 국소적인 최적화에만 집중할 수 있게 해준다.
2. **`ridge_factor` 조정:** 이 매개변수는 평활도 비용 행렬의 대각 성분에 작은 양의 값을 더하여, 목적 함수 표면을 미세하게 변화시킨다. 이는 수치적 안정성을 높일 뿐만 아니라, 얕은 지역 최솟값을 '메워' 알고리즘이 이를 넘어설 수 있도록 돕는 역할을 한다. 장애물 회피가 잘 되지 않을 때 `0.001` 정도의 작은 값을 추가하는 것이 권장된다.6
3. **비용 가중치 스케줄링:** 최적화 과정에서 비용 가중치를 동적으로 조절하는 전략도 유용할 수 있다. 초기 반복 단계에서는 `obstacle_cost_weight`를 상대적으로 높게 설정하여 궤적이 신속하게 충돌 영역에서 벗어나도록 유도한다. 일단 충돌 없는 경로가 확보되면, `smoothness_cost_weight`의 비중을 높여 궤적의 품질과 부드러움을 개선하는 데 집중할 수 있다.4
4. **`learning_rate` 조정:** 현재의 지역 최솟값에서 벗어나지 못하는 경우, `learning_rate`를 일시적으로 높여 최적화 스텝의 크기를 키우는 것을 시도해 볼 수 있다. 이는 알고리즘이 현재의 골짜기를 '뛰어넘어' 다른 해 공간을 탐색할 기회를 제공할 수 있다. 하지만 너무 높은 값은 발산으로 이어질 수 있으므로 주의가 필요하다.35


CHOMP의 매개변수 튜닝은 정해진 공식이 없는 시행착오의 과정일 수 있지만, 다음과 같은 체계적인 워크플로우를 따르면 효율성을 높일 수 있다.37

1. **환경 분석:** 먼저 해결하고자 하는 모션 플래닝 문제의 특성을 파악한다. 작업 공간이 개방적이고 장애물이 거의 없는가, 아니면 좁은 통로와 다수의 장애물로 구성된 복잡한 환경인가?.6
2. **초기화 전략 선택:** 환경의 복잡도에 따라 초기 궤적 생성 방법을 결정한다. 단순한 환경에서는 시작점과 목표점을 잇는 직선 또는 3차/5차 스플라인 보간(`linear`, `cubic`, `quintic-spline`)으로 충분할 수 있다. 복잡한 환경에서는 OMPL을 전처리기로 사용하는 `fillTrajectory` 방식이 성공률을 크게 높일 수 있다.6
3. **비용 가중치 초기 설정:** 작업의 우선순위에 따라 비용 가중치를 설정한다. 충돌 회피가 절대적으로 중요하다면 `obstacle_cost_weight`를 `smoothness_cost_weight`보다 훨씬 높게 설정한다 (예: 1.0 대 0.01). 반대로, 매우 부드럽고 예측 가능한 움직임이 필요하다면 `smoothness_cost_weight`의 비중을 높인다.6
4. **실행 및 실패 분석:** 계획을 실행하고 실패할 경우, RViz와 같은 시각화 도구를 통해 최적화가 중단된 마지막 궤적의 형태를 분석한다.
   - **궤적이 장애물 근처에서 진동하는가?** 이는 `learning_rate`가 너무 높거나, 경사가 급변하는 지역에서 불안정함을 의미할 수 있다. `learning_rate`를 낮추거나 `ridge_factor`를 추가하여 안정성을 높여본다.
   - **궤적이 장애물을 전혀 통과하지 못하고 막혀 있는가?** 이는 장애물 비용의 영향력이 부족하거나, 초기 궤적이 너무 나쁜 지역 최솟값에 갇힌 경우일 수 있다. `obstacle_cost_weight`를 높이거나, OMPL을 통해 더 나은 초기 궤적을 제공한다.
5. **실패 복구 기능 활용:** 수동 튜닝이 번거롭다면, `enable_failure_recovery` 옵션을 `true`로 설정한다. 이 기능은 초기 계획이 실패했을 때, MoveIt이 내부적으로 `learning_rate`와 `ridge_factor` 같은 핵심 매개변수들을 휴리스틱하게 변경하며 `max_recovery_attempts`에 지정된 횟수만큼 자동으로 재시도하도록 한다.6


`enable_failure_recovery` 매개변수는 CHOMP의 실용성을 높여주는 중요한 기능이다. 이 기능이 활성화되면, CHOMP 플래너는 첫 번째 계획 시도에서 해를 찾지 못했을 때 즉시 실패를 반환하지 않는다. 대신, 내부적으로 정의된 복구 전략에 따라 최적화 매개변수를 수정한 후 계획을 재시도한다.

`chomp_planner.cpp`의 구현 로직을 살펴보면, 계획 실패 시 `replan_flag`와 같은 내부 상태 변수를 활성화하고, `replan_count`를 증가시키며 루프를 돈다. 이 루프 안에서 기존에 설정되었던 `learning_rate_`나 `ridge_factor_`와 같은 매개변수들을 점진적으로 변경하여 새로운 최적화 조건을 만든다.27 예를 들어, 초기 `ridge_factor`가 0이었다면 점차 값을 증가시키거나, `learning_rate`를 줄여보는 등의 전략을 사용한다. 이 재시도 과정은 `max_recovery_attempts`에 도달하거나 성공적인 해를 찾을 때까지 반복된다.28

이 기능은 CHOMP의 성능이 단일한 '최적'의 매개변수 집합에 의존하는 것이 아니라, 문제의 특성에 따라 유동적으로 변해야 한다는 현실을 인정한 실용적인 해결책이다. 사용자는 모든 상황에 완벽한 단일 `yaml` 파일을 만들기 위해 고심하는 대신, 이 자동 복구 메커니즘을 통해 플래너가 어느 정도의 '지능'을 가지고 스스로 튜닝하도록 할 수 있다. 이는 특히 환경이 동적으로 변하거나 다양한 계획 문제가 주어지는 시나리오에서 CHOMP의 강건성과 전반적인 성공률을 높이는 데 기여한다.



CHOMP와 OMPL은 현대 로봇 모션 플래닝의 두 가지 주요 철학을 대표한다: 최적화 기반 접근법과 샘플링 기반 접근법. 이 둘의 차이는 단순히 알고리즘의 차이를 넘어, 문제를 해결하는 방식과 결과물의 특성에서 근본적인 차이를 보인다.

OMPL은 무작위 샘플링과 그래프 탐색을 통해 고차원 공간에서 충돌 없는 경로를 빠르게 찾아내는 데 특화되어 있다.1 OMPL의 가장 큰 장점은 확률적 완전성(probabilistically complete)이다. 이는 해가 존재하는 한, 충분한 시간이 주어지면 결국 해를 찾아낼 수 있음을 이론적으로 보장한다. 하지만 OMPL은 경로의 품질(예: 평활도, 길이, 에너지 효율)을 직접적으로 고려하지 않는다. 따라서 OMPL이 생성한 경로는 종종 불필요한 움직임을 포함하거나 각진 형태를 띠어, 실제 로봇에서 실행하기 전에 평활화와 같은 후처리 과정이 거의 항상 필요하다.29

반면, CHOMP는 처음부터 경로의 품질을 최적화의 핵심 목표로 삼는다. 목적 함수에 평활도 비용이 명시적으로 포함되어 있기 때문에, CHOMP가 생성하는 궤적은 본질적으로 부드럽고 동역학적으로 우수하다.16 그러나 CHOMP는 결정론적(deterministic) 경사 하강법에 의존하기 때문에, 초기 궤적이 어떤 지역 최솟값의 유인 영역(basin of attraction)에 속하는지에 따라 결과가 크게 달라진다. 이는 전역 최적해를 보장하지 못하며, 복잡한 환경에서는 해를 전혀 찾지 못하고 실패할 수도 있음을 의미한다.


CHOMP와 STOMP(Stochastic Trajectory Optimization for Motion Planning)는 모두 궤적 최적화에 기반한 플래너라는 공통점을 가지지만, 최적화 문제를 푸는 방식에서 결정적인 차이를 보인다. CHOMP가 목적 함수의 해석적 경사(analytic gradient)를 계산하여 궤적을 업데이트하는 반면, STOMP는 경사를 전혀 사용하지 않는 확률적 최적화 기법을 사용한다.2

STOMP의 핵심 아이디어는 현재 궤적 주위에 다수의 무작위 노이즈가 섞인 궤적들(이를 'rollouts'라고 함)을 생성하고, 각 롤아웃의 비용을 평가하는 것이다. 그 후, 비용이 낮은 롤아웃에 더 높은 가중치를 부여하여 이들을 가중 평균함으로써 다음 단계의 궤적을 생성한다.31 이 과정은 정책 개선(Policy Improvement) 알고리즘과 유사하다.

이러한 접근 방식의 차이는 다음과 같은 성능 차이로 이어진다.

- **지역 최솟값 처리:** STOMP의 가장 큰 장점은 확률적 탐색 덕분에 CHOMP보다 지역 최솟값 문제에 훨씬 강건하다는 점이다. 무작위 롤아웃은 현재 궤적이 갇혀 있는 '골짜기'를 벗어날 수 있는 탐색 기회를 제공한다.29 반면, 경사만을 따라 이동하는 CHOMP는 한번 지역 최솟값에 빠지면 탈출하기가 매우 어렵다.43
- **계획 시간:** 두 플래너의 전반적인 계획 시간은 비슷한 수준으로 알려져 있다. 하지만 STOMP는 각 반복에서 다수의 롤아웃 비용을 계산해야 하는 반면, CHOMP는 경사를 한 번만 계산한다. 그럼에도 불구하고, STOMP는 더 안정적으로 큰 폭의 업데이트를 수행할 수 있어, CHOMP보다 적은 반복 횟수로 성공적인 해에 도달하는 경향이 있다.29
- **매개변수 튜닝:** 일반적으로 CHOMP는 성공적인 결과를 얻기 위해 STOMP보다 더 세심하고 많은 매개변수 튜닝을 필요로 한다. STOMP는 확률적 특성 덕분에 매개변수 변화에 대해 상대적으로 덜 민감한 경향을 보인다.29


MoveIt 프레임워크는 `moveit_ros_benchmarks` 패키지를 통해 여러 플래너를 동일한 환경과 문제에 대해 정량적으로 비교, 평가할 수 있는 도구를 제공한다.44 여러 연구에서 이러한 벤치마크를 통해 CHOMP, OMPL, STOMP의 성능을 비교 분석했다.

한 연구45에서는 OMPL을 전처리기로 사용하여 초기 경로를 생성하고, CHOMP와 STOMP를 후처리기(최적화기)로 사용했을 때의 성능을 비교했다. 그 결과, 장애물이 적은 저복잡도 환경에서는 STOMP가 OMPL 경로의 품질을 크게 개선하고 성공률도 높이는 등 매우 우수한 성능을 보였다. 반면, CHOMP 역시 경로 품질을 개선했지만, 일부 경우에는 최적화 과정에서 지역 최솟값에 빠져 오히려 원본 OMPL보다 성공률이 감소하는 현상이 관찰되었다. 장애물이 많은 고복잡도 환경에서는, 두 최적화 플래너 모두 초기 경로를 개선하지 못하고 오히려 성능이 저하되는 경우도 발생했다. 이는 복잡한 제약 조건 하에서 최적화기가 해 공간을 효과적으로 탐색하지 못함을 시사한다.

이러한 분석들을 종합하여 세 플래너의 특성을 요약하면 다음 표와 같다.

**Table 2: 주요 모션 플래너 비교 (CHOMP vs. OMPL vs. STOMP)**

| 특성 (Characteristic)  | CHOMP                        | OMPL                          | STOMP                                |
| ---------------------- | ---------------------------- | ----------------------------- | ------------------------------------ |
| **접근 방식**          | 궤적 최적화 (경사 기반)      | 샘플링 기반 (확률적)          | 궤적 최적화 (확률적, 경사 없음)      |
| **경로 품질 (평활도)** | 매우 높음 (기본적으로 평활)  | 낮음 (후처리 필요)            | 높음 (평활한 경로 생성)              |
| **계획 시간**          | 중간 ~ 김 (반복 횟수에 의존) | 매우 빠름 (최적해 보장 안 함) | 중간 ~ 김 (CHOMP와 유사)             |
| **지역 최솟값 문제**   | 취약함 (주요 단점)           | 해당 없음 (전역적 탐색)       | 강인함 (확률적 탐색으로 회피)        |
| **매개변수 튜닝**      | 민감하고, 많은 튜닝 필요     | 거의 필요 없음                | CHOMP보다 덜 민감함                  |
| **초기 경로 필요성**   | 필요 (비현실적 경로도 가능)  | 필요 없음                     | 필요 (비현실적 경로도 가능)          |
| **주요 강점**          | 고품질, 평활한 궤적 생성     | 복잡한 공간에서 빠른 해 탐색  | 지역 최솟값 회피, 제약조건 처리 용이 |
| **주요 약점**          | 지역 최솟값, 튜닝 복잡성     | 경로 품질이 낮고 비최적       | CHOMP보다 이론적 기반 약함           |

이러한 비교 분석은 '만능' 모션 플래너는 존재하지 않으며, 최적의 전략은 여러 플래너를 하나의 '포트폴리오'로 간주하고 주어진 작업의 요구사항에 맞게 조합하는 것임을 명확히 보여준다. 예를 들어, 복잡한 환경에서 우선적으로 실행 가능한 경로를 찾아야 한다면 OMPL이 최적의 선택이다(탐색, exploration). 반면, 이미 경로가 주어진 상태에서 그 품질을 최대한 끌어올려야 한다면 CHOMP나 STOMP가 적합하다(개선, exploitation). MoveIt2의 플래닝 어댑터와 병렬 계획 인터페이스는 바로 이러한 하이브리드 전략을 시스템 수준에서 구현하기 위해 설계된 아키텍처이다. 이는 현대 모션 플래닝 시스템이 단일 알고리즘의 우월성을 추구하기보다, 여러 알고리즘의 장점을 체계적으로 결합하여 강건성과 성능을 극대화하는 방향으로 발전하고 있음을 보여주는 증거이다.



CHOMP가 제시한 최적화 기반 모션 플래닝의 기본 프레임워크는 이후 다양한 연구를 통해 확장되고 개선되어 왔다. 이러한 변형 알고리즘들은 원본 CHOMP의 한계를 극복하고 새로운 기능을 추가하는 데 중점을 두었다.

- **T-CHOMP (Time-CHOMP):** 원본 CHOMP는 궤적의 기하학적 형태를 최적화하는 데는 뛰어나지만, 궤적상의 시간 매개변수화(timing)를 직접적으로 최적화하지는 못하는 한계가 있었다. T-CHOMP는 이 문제를 해결하기 위해 '시간'을 궤적의 위치와는 독립적인 또 하나의 최적화 변수로 도입했다.9 이를 통해 궤적을 고정된 시공간(space-time)에서 직접 최적화할 수 있게 되었다. 결과적으로 T-CHOMP는 "장애물 근처에서는 속도를 늦추고, 안전한 구간에서는 속도를 높이는" 것과 같이 시간에 따라 변하는 복잡한 비용 함수를 통합하여, 훨씬 더 지능적이고 상황에 맞는 궤적을 생성할 수 있게 되었다.
- **dlCHOMP (Deep Learning CHOMP):** 비교적 정적인 환경에서 반복적인 작업을 수행하는 시나리오에 착안하여, 딥러닝 기술을 CHOMP와 결합한 변형이다.48 dlCHOMP는 신경망을 사전 학습시켜 주어진 환경과 목표에 대한 좋은 초기 궤적을 빠르게 '예측'하도록 한다. 그 후, CHOMP는 이 신경망이 생성한 고품질의 초기 궤적을 입력받아 약간의 미세 조정을 통해 최종 궤적을 완성한다. 이 방식은 반복적인 최적화 계산의 대부분을 빠른 추론 시간의 신경망으로 대체함으로써, 전체적인 계획 시간을 획기적으로 단축시키는 효과를 가져온다.
- **Multigrid CHOMP:** 고해상도 궤적(많은 수의 웨이포인트)을 직접 최적화할 때 발생하는 높은 계산 비용 문제를 해결하기 위한 접근법이다.49 이 알고리즘은 처음에는 매우 낮은 해상도의 궤적으로 최적화를 시작하여 전반적인 경로의 형태를 빠르게 잡는다. 그 후, 점진적으로 궤적의 해상도를 높여가면서 최적화를 세분화하는 다중 그리드(multigrid) 기법을 적용한다. 이는 계산 효율성과 최종 궤적의 정밀도 사이의 균형을 맞추는 효과적인 방법이다.


CHOMP는 샘플링 기반 방법론이 주류를 이루던 모션 플래닝 학계에 최적화 기반 접근법의 잠재력을 다시 한번 각인시킨 기념비적인 알고리즘으로 평가받는다. 특히, 충돌 가능성이 있는 비현실적인 초기 궤적으로부터 시작하여 점진적으로 충돌 없는 경로를 찾아내는 CHOMP의 능력은, 이후 등장한 TrajOpt와 같은 여러 최적화 기반 플래너들에게 큰 영감을 주었다.49

또한, 작업 공간(workspace)에서 정의된 거리 필드와 그 경사 정보를 활용하여 장애물 비용을 계산하는 아이디어는 매우 효율적이고 실용적인 방법으로 인정받아, 많은 현대 플래너들에서 유사한 형태로 채택되고 있다.52 CHOMP가 모션 플래닝 문제를 명시적인 목적 함수와 제약 조건을 갖는 최적 제어(optimal control) 문제로 정식화한 것은, 로봇 공학의 계획(planning) 분야와 제어(control) 분야 사이의 간극을 좁히고 두 분야의 이론적 융합을 촉진하는 계기가 되었다.3


최근 로봇 모션 플래닝 연구에서 가장 두드러지는 흐름은 CHOMP와 같은 고전적인 최적화 방법론과 딥러닝으로 대표되는 학습 기반 방법론을 융합하려는 시도이다.54 이 두 접근법은 상호 보완적인 장단점을 가지고 있어, 이들을 결합함으로써 각각의 한계를 극복하고 시너지를 창출할 수 있다.

- **학습을 통한 초기화(Learning for Initialization):** dlCHOMP에서 볼 수 있듯이, 복잡한 센서 입력(예: 카메라 이미지)으로부터 좋은 초기 궤적을 생성하는 '직관적인' 역할을 신경망에 맡기고, CHOMP와 같은 최적화기는 이 초기 궤적이 물리적 제약(충돌, 동역학)을 엄격하게 만족하도록 빠르고 안정적으로 수정하는 역할을 담당한다.48
- **학습을 통한 비용 함수 정의(Learning the Cost Function):** 어떤 궤적이 '좋은' 궤적인지를 결정하는 비용 함수 자체를 정의하는 것은 어려운 문제일 수 있다. 이 접근법에서는 인간의 선호도나 전문가의 시연 데이터로부터 로봇이 따라야 할 비용 함수를 역강화학습(inverse reinforcement learning) 등의 기법으로 학습한다. 그 후, 최적화기는 이 데이터로부터 학습된, 암묵적인 비용 함수를 최소화하는 궤적을 찾아낸다.57
- **최적화를 신경망의 한 레이어로(Optimization as a Layer):** 가장 진보된 형태의 융합으로, 최적화 과정 자체를 미분 가능한 연산으로 근사하여 신경망의 한 레이어처럼 구성하는 연구가 활발히 진행되고 있다.54 이를 통해 전체 시스템을 종단간(end-to-end)으로 학습시킬 수 있으며, 환경 인식부터 최종 제어 출력까지의 모든 과정이 주어진 목표에 맞게 통합적으로 최적화된다.

이러한 융합의 흐름 속에서 미래의 모션 플래닝 시스템은 순수한 학습 기반 접근법이나 순수한 최적화 기반 접근법 어느 한쪽에 치우치기보다는, 두 방법론의 장점을 결합한 하이브리드 형태가 지배하게 될 것으로 예측된다. 이 패러다임에서 학습은 비정형 데이터로부터 패턴을 인식하고 좋은 초기해를 제공하는 '직관'과 '경험'의 역할을 수행한다. 반면, 최적화는 학습된 결과가 물리적으로 타당하고 안전 제약을 만족하는지를 수학적으로 엄밀하게 보장하는 '이성'과 '검증'의 역할을 담당한다.58 이러한 미래의 하이브리드 모션 플래닝 아키텍처에서 CHOMP는 그 견고한 이론적 기반과 효율성을 바탕으로 핵심적인 '최적화 모듈'로서 계속해서 중요한 역할을 수행할 것이다.


1. Using CHOMP Planner - MoveIt Documentation - PickNik Robotics, accessed August 6, 2025, https://moveit.picknik.ai/main/doc/how_to_guides/chomp_planner/chomp_planner_tutorial.html
2. Planners | MoveIt, accessed August 6, 2025, https://moveit.ai/documentation/planners/
3. CHOMP: Gradient Optimization Techniques for Efficient Motion Planning - Carnegie Mellon University Robotics Institute, accessed August 6, 2025, https://www.ri.cmu.edu/pub_files/2009/5/icra09-chomp.pdf
4. CHOMP: Covariant Hamiltonian Optimization for Motion Planning - Carnegie Mellon University Robotics Institute, accessed August 6, 2025, https://www.ri.cmu.edu/pub_files/2013/5/CHOMP_IJRR.pdf
5. Optimized Robotics - CHOMP - Nathan Ratliff, accessed August 6, 2025, https://www.nathanratliff.com/thesis-research/chomp
6. Using CHOMP Planner - MoveIt Documentation: Humble documentation, accessed August 6, 2025, https://moveit.picknik.ai/humble/doc/how_to_guides/chomp_planner/chomp_planner_tutorial.html
7. CHOMP: Covariant Hamiltonian optimization for motion planning | Request PDF, accessed August 6, 2025, https://www.researchgate.net/publication/258141018_CHOMP_Covariant_Hamiltonian_optimization_for_motion_planning
8. CHOMP: Covariant Hamiltonian Optimization For Motion Planning - Swarthmore College, accessed August 6, 2025, https://works.swarthmore.edu/fac-engineering/58/
9. Space-Time Functional Gradient Optimization for Motion Planning - University of Washington, accessed August 6, 2025, https://homes.cs.washington.edu/~bboots/files/T-CHOMP-icra-2014.pdf
10. Motion Planning by Learning the Solution Manifold in Trajectory Optimization - arXiv, accessed August 6, 2025, https://arxiv.org/abs/2107.05842
11. CHOMP: Covariant Hamiltonian optimization for motion planning - Personal Robotics Lab, accessed August 6, 2025, https://personalrobotics.cs.washington.edu/publications/zucker2013chomp.pdf
12. Covariant Hamiltonian Optimization for Motion ... - Washington, accessed August 6, 2025, https://courses.cs.washington.edu/courses/cse571/19wi/lectures/Feb_6th_CHOMP_Notes.pdf
13. chompSmoothnessOptions - Smoothness options for CHOMP trajectories - MATLAB, accessed August 6, 2025, https://www.mathworks.com/help/robotics/ref/chompsmoothnessoptions.html
14. Lecture Notes: Some notes on gradient descent, accessed August 6, 2025, https://www.cs.virginia.edu/yanjun/teach/2014f/lecture/L4-GD.pdf
15. Natural Gradient Shared Control - Cornell | ARL, accessed August 6, 2025, [https://arl.human.cornell.edu/linked%20docs/Natural%20Gradiant%20Shared%20Control-ROMAN2020.pdf](https://arl.human.cornell.edu/linked docs/Natural Gradiant Shared Control-ROMAN2020.pdf)
16. Parallel Planning with MoveIt 2, accessed August 6, 2025, [https://moveit.ai/moveit%202/parallel%20planning/motion%20planning/2023/02/15/parallel-planning-with-MoveIt-2.html](https://moveit.ai/moveit 2/parallel planning/motion planning/2023/02/15/parallel-planning-with-MoveIt-2.html)
17. Concepts | MoveIt, accessed August 6, 2025, https://moveit.ai/documentation/concepts/
18. Error while Demo launch - on ROS2 Humble and 22.04 Ubuntu / Issue #1814 / moveit/moveit2 - GitHub, accessed August 6, 2025, https://github.com/moveit/moveit2/issues/1814
19. Planning Adapter Tutorials - moveit_tutorials Melodic documentation, accessed August 6, 2025, http://docs.ros.org/en/melodic/api/moveit_tutorials/html/doc/planning_adapters/planning_adapters_tutorial.html
20. CHOMP Planner - moveit_tutorials Kinetic documentation, accessed August 6, 2025, http://docs.ros.org/en/kinetic/api/moveit_tutorials/html/doc/chomp_planner/chomp_planner_tutorial.html
21. Pick-And-Place Workflow Using CHOMP for Manipulators - MATLAB & - MathWorks, accessed August 6, 2025, https://www.mathworks.com/help/robotics/ug/pick-and-place-using-chomp-for-manipulators.html
22. MoveIt for ROS2 - Update - ROS-Industrial, accessed August 6, 2025, https://rosindustrial.squarespace.com/s/ROS-I-Community-Meeting-MoveIt2-June-2020.pdf
23. Perception Pipeline Tutorial - MoveIt Documentation - PickNik Robotics, accessed August 6, 2025, https://moveit.picknik.ai/main/doc/examples/perception_pipeline/perception_pipeline_tutorial.html
24. moveit_core: distance_field::DistanceField Class Reference - ROS documentation, accessed August 6, 2025, http://docs.ros.org/jade/api/moveit_core/html/classdistance__field_1_1DistanceField.html
25. distance_field.cpp File Reference - MoveIt 2 Documentation, accessed August 6, 2025, https://moveit.picknik.ai/main/api/html/distance__field_8cpp.html
26. Namespace List - moveit2, accessed August 6, 2025, https://moveit.picknik.ai/main/api/html/namespaces.html
27. moveit/moveit_planners/chomp/chomp_motion_planner/src/chomp_planner.cpp at master - GitHub, accessed August 6, 2025, https://github.com/ros-planning/moveit/blob/master/moveit_planners/chomp/chomp_motion_planner/src/chomp_planner.cpp
28. moveit2: chomp::ChompParameters Class Reference - MoveIt Documentation, accessed August 6, 2025, https://moveit.picknik.ai/main/api/html/classchomp_1_1ChompParameters.html
29. STOMP Motion Planner - MoveIt Documentation - PickNik Robotics, accessed August 6, 2025, https://moveit.picknik.ai/main/doc/how_to_guides/stomp_planner/stomp_planner.html
30. STOMP Planner - moveit_tutorials Kinetic documentation, accessed August 6, 2025, http://docs.ros.org/en/kinetic/api/moveit_tutorials/html/doc/stomp_planner/stomp_planner_tutorial.html
31. moveit2_tutorials/doc/how_to_guides/stomp_planner/stomp_planner.rst at main / moveit/moveit2_tutorials - GitHub, accessed August 6, 2025, https://github.com/ros-planning/moveit2_tutorials/blob/main/doc/how_to_guides/stomp_planner/stomp_planner.rst
32. Physics-Based Trajectory Optimization for Grasping in Cluttered Environments - People @EECS, accessed August 6, 2025, https://people.eecs.berkeley.edu/~pabbeel/papers/2015-ICRA-clutter.pdf
33. The diagram illustrates the reason for the local minimum problem in the path planning of manipulators - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/figure/The-diagram-illustrates-the-reason-for-the-local-minimum-problem-in-the-path-planning-of_fig1_326268262
34. Adaptive Hybrid Local-Global Sampling for Fast Informed Sampling-Based Optimal Path Planning - arXiv, accessed August 6, 2025, https://arxiv.org/html/2208.09318v2
35. Escaping the Trap of Local Minima: Why Optimization Algorithms Get Stuck and How to Break Free | by Shazan ansar | Medium, accessed August 6, 2025, https://medium.com/@shazanansar/escaping-the-trap-of-local-minima-why-optimization-algorithms-get-stuck-and-how-to-break-free-6e3042a6d9c9
36. The Curse of Local Minima: How to Escape and Find the Global Minimum | by Mohit Mishra, accessed August 6, 2025, https://mohitmishra786687.medium.com/the-curse-of-local-minima-how-to-escape-and-find-the-global-minimum-fdabceb2cd6a
37. ROS 2 Navigation Tuning Guide – Nav2 - Automatic Addison, accessed August 6, 2025, https://automaticaddison.com/ros-2-navigation-tuning-guide-nav2/
38. Optimize trajectory using CHOMP - MATLAB - MathWorks, accessed August 6, 2025, https://www.mathworks.com/help/robotics/ref/manipulatorchomp.optimize.html
39. Class ChompParameters - chomp_motion_planner: Jazzy 2.14.0 documentation - ROS 2, accessed August 6, 2025, https://docs.ros.org/en/ros2_packages/jazzy/api/chomp_motion_planner/generated/classchomp_1_1ChompParameters.html
40. Optimization-based planning with STOMP - Manipulation - ROS Discourse - Open Robotics, accessed August 6, 2025, https://discourse.openrobotics.org/t/optimization-based-planning-with-stomp/31488
41. Benchmarking and optimization of robot motion planning with motion planning pipeline, accessed August 6, 2025, https://d-nb.info/1246182696/34
42. Guided Stochastic Optimization for Motion Planning - PMC - PubMed Central, accessed August 6, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC7805853/
43. Benchmark and Analysis of Path Planning Algorithms of "ROS MoveIt!" for Pick and Place Task in Tomato Harvesting - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/351199972_Benchmark_and_Analysis_of_Path_Planning_Algorithms_of_ROS_MoveIt_for_Pick_and_Place_Task_in_Tomato_Harvesting
44. Benchmarking - MoveIt Documentation: Humble documentation, accessed August 6, 2025, https://moveit.picknik.ai/humble/doc/examples/benchmarking/benchmarking_tutorial.html
45. (PDF) Robot Motion Planning Benchmarking and Optimization using ..., accessed August 6, 2025, https://www.researchgate.net/publication/350713579_Robot_Motion_Planning_Benchmarking_and_Optimization_using_Motion_Planning_Pipeline
46. Benchmarking and optimization of robot motion planning with motion planning pipeline - York Research Database - Please log in to continue, accessed August 6, 2025, https://pure.york.ac.uk/portal/en/publications/benchmarking-and-optimization-of-robot-motion-planning-with-motio
47. (PDF) Benchmarking and optimization of robot motion planning with motion planning pipeline - ResearchGate, accessed August 6, 2025, https://www.researchgate.net/publication/354224017_Benchmarking_and_optimization_of_robot_motion_planning_with_motion_planning_pipeline
48. Train Deep-Learning-Based CHOMP Optimizer for Motion Planning - MATLAB &, accessed August 6, 2025, https://www.mathworks.com/help/robotics/ug/train-deep-learning-based-chomp-optimizer.html
49. Gaussian Process Motion Planning, accessed August 6, 2025, https://homes.cs.washington.edu/~bboots/files/GPMP.pdf
50. Robotics Research The International Journal of, accessed August 6, 2025, https://cal-mr.berkeley.edu/papers/Schulman-IJRR-June-2014.pdf
51. MIT Open Access Articles Improving Trajectory Optimization Using a Roadmap Framework, accessed August 6, 2025, https://dspace.mit.edu/bitstream/handle/1721.1/137335/1811.02044.pdf?sequence=2&isAllowed=y
52. A Comparison of Motion Planners for Robotic Arms - Journal of Control Engineering and Applied Informatics, accessed August 6, 2025, [http://ceai.srait.ro/index.php?journal=ceai&page=article&op=download&path%5B%5D=7290&path%5B%5D=1612](http://ceai.srait.ro/index.php?journal=ceai&page=article&op=download&path[]=7290&path[]=1612)
53. Collision-free and Smooth Trajectory Computation in Cluttered Environments, accessed August 6, 2025, http://gammap.cs.unc.edu/SPATH/paper/ijrr.pdf
54. Technical Report: Learning to Plan Maneuverable and Agile Flight Trajectory with Optimization Embedded Networks - arXiv, accessed August 6, 2025, https://arxiv.org/html/2405.07736v3
55. Combining Optimal Control and Learning for Visual Navigation in Novel Environments, accessed August 6, 2025, http://proceedings.mlr.press/v100/bansal20a/bansal20a.pdf
56. Combining the benefits of function approximation and trajectory optimization - Robotics, accessed August 6, 2025, https://www.roboticsproceedings.org/rss10/p52.pdf
57. Trajectory Improvement and Reward Learning from Comparative Language Feedback, accessed August 6, 2025, https://arxiv.org/html/2410.06401v1
58. Optimizing Motion Planning in Robotics - Number Analytics, accessed August 6, 2025, https://www.numberanalytics.com/blog/optimizing-motion-planning-robotics-trajectory-planning
59. AI-Driven Motion Planning: The Future of Robotics - Number Analytics, accessed August 6, 2025, https://www.numberanalytics.com/blog/ai-driven-motion-planning-future-robotics
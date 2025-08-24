# EGO-Planner 및 드론 시스템 통합
[EGO-Planner](./index.md)



쿼드로터(Quadrotor)와 같은 소형 무인 항공기(MAV, Micro Aerial Vehicle)의 활용 범위가 정찰, 감시, 물류, 인프라 점검 등 다양한 산업 분야로 확장되면서, 미지의 복잡하고 동적인 환경에서 안전하고 효율적으로 임무를 수행할 수 있는 자율 비행 기술의 중요성이 그 어느 때보다 부각되고 있다.1 자율 비행 시스템의 핵심은 경로 계획(Path Planning) 능력에 있으며, 이는 크게 전역 경로 계획(Global Planning)과 지역 경로 계획(Local Planning)으로 나뉜다.

전역 경로 계획은 출발점에서 목표점까지의 전체적인 경로를 사전에 알려진 지도 정보를 바탕으로 생성하는 과정이다. 하지만 실제 비행 환경은 지도에 표시되지 않은 장애물이 존재하거나, 사람이나 다른 이동체와 같은 동적 장애물이 나타나는 등 예측 불가능한 변수로 가득하다. 이러한 상황에서 전역 경로만을 맹목적으로 추종하는 것은 충돌로 이어질 수 있다. 바로 이 지점에서 실시간 지역 경로 계획의 역할이 결정적으로 중요해진다. 지역 경로 계획은 드론에 장착된 센서(카메라, LiDAR 등)를 통해 실시간으로 주변 환경을 인식하고, 전역 경로를 최대한 유지하면서 눈앞의 미지의 장애물을 동적으로 회피하는 안전하고 실행 가능한 짧은 구간의 경로를 수시로 재계획(Replanning)하는 기술이다.1 성공적인 자율 비행은 이 두 가지 계획 단계의 유기적인 결합을 통해 완성된다.


지역 경로 계획 알고리즘 중에서 경사도 기반 최적화(Gradient-based Optimization) 기법은 그 효율성과 생성되는 궤적의 품질 덕분에 널리 사용되어 왔다.1 이 접근법은 궤적의 매끄러움(smoothness), 안전성(safety), 동역학적 타당성(dynamical feasibility) 등을 모두 고려하는 비용 함수(Cost Function)를 정의하고, 이 비용을 최소화하는 궤적을 경사 하강법(Gradient Descent)과 같은 수치 최적화 기법으로 찾는 방식이다.

이러한 경사도 기반 기법의 핵심에는 전통적으로 ESDF(Euclidean Signed Distance Field)가 자리 잡고 있었다.2 ESDF는 맵 상의 모든 지점에서 가장 가까운 장애물까지의 유클리드 거리를 부호(장애물 내부는 음수, 외부는 양수)와 함께 저장한 필드이다. ESDF는 궤적이 장애물에 얼마나 가까운지를 나타내는 비용(collision cost)과, 장애물로부터 멀어지는 방향을 알려주는 그래디언트(gradient)를 연속적으로 제공하므로 최적화 과정에 매우 유용하다.1 FIESTA와 같은 알고리즘은 이러한 ESDF 맵을 점진적으로 효율적으로 구축하기 위해 제안되기도 했다.3

하지만 ESDF는 치명적인 단점을 내포한다. 바로 계산 비용이다. 드론이 탐사하는 환경 전체 또는 매우 넓은 범위에 대해 ESDF를 계산하고 지속적으로 업데이트하는 것은 상당한 계산 부하를 유발한다.2 더욱이, 실제 궤적 최적화 과정에서 고려되는 경로는 전체 ESDF 맵의 극히 제한된 부분 공간(subspace)만을 사용하기 때문에, 나머지 영역에 대한 ESDF 계산은 불필요한 중복 연산(redundant computation)이 된다.1 이러한 계산적 비효율성은 온보드 컴퓨터의 자원이 제한적인 소형 드론에게는 큰 부담으로 작용하며, 실시간 고속 비행을 위한 빠른 재계획을 저해하는 병목 지점이 되어왔다.


이러한 ESDF의 한계를 극복하기 위해, 저우(Zhou) 연구팀은 EGO-Planner(ESDF-free Gradient-based lOcal planner)라는 혁신적인 프레임워크를 제안했다.1 EGO-Planner의 핵심 아이디어는 ESDF라는 무거운 사전 구조물에 의존하는 대신, '필요할 때만' 장애물 정보를 동적으로 활용하는 것이다. 즉, ESDF 맵을 사전에 구축하고 유지하는 과정을 완전히 제거했다.2

이를 위해 EGO-Planner는 충돌 비용 함수를 새롭게 정식화했다. 궤적이 장애물과 충돌했을 때, ESDF 맵을 참조하는 대신 현재 충돌한 궤적과 가상으로 설정된 '충돌 없는 안전한 가이드 경로(collision-free guiding path)'를 비교한다. 그리고 충돌 지점에서 이 안전한 가이드 경로 방향으로 궤적을 밀어내는 힘, 즉 그래디언트를 계산하여 최적화에 사용한다.2 이 방식은 궤적이 장애물에 부딪혔을 때만 충돌에 대한 그래디언트를 계산하므로, 전역적인 ESDF 필드를 유지할 필요가 없어진다. 결과적으로 EGO-Planner는 기존의 ESDF 기반 플래너들에 비해 계산 시간을 수십 배 단축시키면서도, 유사한 품질의 안전하고 매끄러운 궤적을 생성하는 데 성공했다.2


EGO-Planner는 처음에는 단일 드론을 위한 지역 경로 계획기로 발표되었지만, 그 개념의 우수성을 바탕으로 지속적으로 발전했다. 특히, 다수의 드론이 서로 충돌하지 않고 협력 비행하는 군집(Swarm) 환경으로 확장된 `EGO-Swarm` 7과, 강건성 및 성능을 대폭 개선한 `EGO-Planner-v2` 8가 후속으로 공개되었다. 개발팀 스스로도 더 강건하고 안전한 최신 버전인 `EGO-Swarm`의 사용을 공식적으로 권장하고 있으며, 단일 드론 환경에서도 간단한 설정 변경만으로 사용이 가능하다고 밝히고 있다.5

이러한 진화 과정은 중요한 시사점을 제공한다. 고성능 지역 경로 계획기는 단독으로 존재할 때 그 가치를 완전히 발휘할 수 없다. 실제 비행 환경에서 성공적으로 작동하기 위해서는 주변 환경을 정밀하게 인식하는 센서 시스템(Perception), 센서 데이터를 바탕으로 드론의 위치와 자세를 정확하게 추정하는 상태 추정 시스템(State Estimation), 그리고 생성된 경로를 오차 없이 추종하는 저수준 제어 시스템(Control)과의 긴밀한 통합이 필수적이다. 한 연구에서는 `EGO-Planner-v2`를 `VINS-Fusion`이라는 시각-관성 주행 거리 측정(VIO, Visual-Inertial Odometry) 알고리즘과 결합하여 실제 숲 환경에서 자율 비행을 성공시킨 사례를 보고했다.9 이는 EGO-Planner의 성공이 단순히 플래닝 알고리즘 자체의 우수성을 넘어, 인식-추정-계획-제어로 이어지는 전체 자율 비행 시스템의 유기적인 설계에 달려있음을 명확히 보여준다.

따라서 본 보고서는 단순히 초기 EGO-Planner 알고리즘을 분석하는 것에 그치지 않는다. 사용자의 근본적인 질의인 "드론과 조종기에 어떻게 구성하는가"에 답하기 위해, EGO-Planner의 최신 버전(`EGO-Swarm`)을 기준으로 그 핵심 이론을 심도 있게 파헤치고, 이를 실제 드론에 탑재하기 위한 완전한 자율 비행 시스템을 구축하는 전 과정을 이론과 실제를 아우르며 상세히 고찰하고자 한다.



EGO-Planner의 가장 핵심적인 혁신은 기존 경사도 기반 최적화의 패러다임을 바꾼 ESDF-free 접근법에 있다. 이 원리를 이해하기 위해서는 먼저 ESDF의 역할과 그로 인한 계산적 병목 현상을 깊이 있게 살펴볼 필요가 있다.


ESDF는 장애물 회피를 위한 비용 함수를 정의하는 데 이상적인 정보를 제공한다. 맵 상의 임의의 지점 $\mathbf{p}$에 대해 ESDF 값 $d(\mathbf{p})$는 가장 가까운 장애물까지의 거리를 나타낸다. 장애물 바깥에서는 양수, 안쪽에서는 음수 값을 가진다. 이 연속적인 거리 값 자체를 충돌 비용으로 사용할 수 있으며, 더 중요한 것은 ESDF의 그래디언트 $\nabla d(\mathbf{p})$가 해당 지점에서 장애물로부터 가장 빠르게 멀어지는 방향을 가리킨다는 점이다.1 이는 경사 하강법 기반의 최적화 알고리즘이 충돌을 회피하는 방향으로 궤적을 수정하는 데 결정적인 단서를 제공한다.

하지만 이러한 유용성에는 큰 대가가 따른다. ESDF 맵을 생성하고 유지하는 것은 계산적으로 매우 까다로운 작업이다. 전통적인 방식은 센서 데이터로부터 얻은 점유 격자 지도(Occupancy Grid Map)를 기반으로, 각 셀마다 모든 장애물 셀과의 거리를 계산하는 과정을 포함한다. 이를 효율화하기 위해 FIESTA 3와 같은 증분적(incremental) 업데이트 방식이나 복셀 해싱(Voxel Hashing) 기법들이 제안되었지만, 근본적으로 맵의 크기와 해상도에 따라 계산량이 크게 증가하는 문제를 피할 수 없다. 특히, 드론이 새로운 환경을 탐색하며 맵이 계속해서 확장되고 업데이트될 때, ESDF를 실시간으로 유지하는 부담은 더욱 가중된다.

EGO-Planner는 이러한 과정에 근본적인 비효율성이 존재한다고 보았다. 궤적 최적화 과정은 보통 수 미터 내의 가까운 미래에 대한 경로만을 다룬다. 즉, 최적화 알고리즘이 실제로 ESDF 값을 필요로 하는 영역은 드론 주변의 매우 좁은 시공간에 한정된다. 그럼에도 불구하고 전통적인 방식은 드론의 시야에 들어오는 훨씬 더 넓은 영역, 심지어는 궤적이 전혀 지나갈 가능성이 없는 영역에 대해서도 ESDF를 계산하고 있었던 것이다.1 이것이 바로 EGO-Planner가 해결하고자 한 '계산의 중복성' 문제이다.


EGO-Planner는 ESDF라는 거대한 정보 필드를 사전에 구축하는 대신, 충돌이 발생한 바로 그 지점에서 국소적으로 필요한 정보만을 추출하여 사용하는 방식을 택했다. 이 과정은 다음과 같이 이루어진다.2

1. **충돌 지점 식별:** 현재 최적화 중인 궤적 $\mathbf{p}(t)$를 이산적인 샘플 포인트들로 나눈다. 각 샘플 포인트가 점유 격자 지도 상에서 장애물 셀 내부에 위치하는지 검사하여 충돌 지점 집합 $\{\mathbf{p}_{col}\}$을 식별한다.
2. **충돌 없는 가이드 경로 활용:** EGO-Planner는 초기에 A*나 RRT*와 같은 탐색 기반 알고리즘 또는 운동학적으로 가능한 경로 탐색(Kinodynamic Path Searching)을 통해 장애물을 고려한 거친 전역 경로 또는 가이드 경로(Guiding Path)를 확보한다.1 이 가이드 경로는 최적은 아닐지라도 최소한 충돌이 없는 경로이다.
3. **충돌 비용 그래디언트 계산:** 각 충돌 지점 $\mathbf{p}_{col}$에 대해, 사전에 계산된 가이드 경로 상에서 가장 가까운 지점 $\mathbf{p}_{free}$를 찾는다. 그리고 $\mathbf{p}_{col}$에서 $\mathbf{p}_{free}$를 향하는 벡터 $\mathbf{v}_{push} = \mathbf{p}_{free} - \mathbf{p}_{col}$를 계산한다. 이 벡터가 바로 충돌을 회피하도록 밀어내는 힘, 즉 충돌 비용의 그래디언트 역할을 한다. 충돌 비용 함수 $f_{collision}$과 그 그래디언트 $\nabla f_{collision}$은 이 벡터를 기반으로 정식화된다.

이 방식의 장점은 명확하다. 궤적이 장애물과 충돌하지 않는 한, 충돌 비용은 0이며 관련 계산도 전혀 발생하지 않는다. 오직 충돌이 발생했을 때만, 해당 지점 주변의 장애물 정보와 가이드 경로를 참조하여 국소적인 그래디언트를 계산한다. 따라서 전역적인 ESDF 맵을 저장할 메모리도, 이를 업데이트할 계산 시간도 필요 없다. 최적화 과정에서 궤적은 장애물에 부딪히면 가이드 경로 방향으로 '튕겨 나오고(rebound)', 이 과정을 반복하며 점차 안전하고 매끄러운 형태로 수렴하게 된다.6 이는 계산 자원이 제한된 임베디드 시스템에서 경사도 기반 최적화를 매우 가볍고 빠르게 수행할 수 있도록 만드는 핵심적인 발상의 전환이다.


EGO-Planner는 궤적을 표현하고 최적화하기 위한 수학적 도구로 균일 B-스플라인(Uniform B-Spline)을 채택했다. B-스플라인은 그 수학적 특성상 로봇의 동역학적 제약과 매끄러움을 다루는 데 매우 효과적이다.1


$p$차(degree) 균일 B-스플라인 곡선 $\mathbf{p}(t)$는 $N+1$개의 제어점(Control Points) $\mathbf{Q}_0, \mathbf{Q}_1,..., \mathbf{Q}_N$과 기저 함수(Basis Functions) $B_{i,p}(t)$의 선형 결합으로 정의된다.
$$
\mathbf{p}(t) = \sum_{i=0}^{N} \mathbf{Q}_i B_{i,p}(t), \quad t \in [t_{p-1}, t_{N+1}]
$$
여기서 기저 함수 $B_{i,p}(t)$는 콕스-드부어(Cox-de Boor) 재귀 관계식에 의해 정의되며, '균일(uniform)'이라는 말은 노드 벡터(knot vector)의 간격이 일정하다는 것을 의미한다. 이 표현의 중요한 특징은 제어점의 위치만으로 전체 곡선의 형태가 결정되며, $p$차 B-스플라인은 자동적으로 $C^{p-1}$의 연속성(Continuity)을 보장한다는 점이다. 예를 들어 3차(cubic) B-스플라인을 사용하면 궤적은 $C^2$ 연속성을 가지므로, 위치, 속도, 가속도가 모두 연속적이고 매끄러운 궤적이 생성된다. 이는 급격한 방향 전환이나 가속도 변화가 없는, 드론이 물리적으로 안정적으로 추종할 수 있는 경로를 만드는 데 필수적이다.


B-스플라인의 가장 강력하고 유용한 특성 중 하나는 바로 볼록 포체(Convex Hull) 특성이다.2 B-스플라인 곡선의 한 조각(segment), 즉 $t \in [t_j, t_{j+1}]$ 사이의 곡선은 그 조각의 모양에 영향을 주는 $p+1$개의 제어점($\mathbf{Q}_{j-p},..., \mathbf{Q}_j$)들이 만드는 볼록 다각형(볼록 포체) 내부에 항상 존재한다.2

이 특성은 충돌 검사와 동역학적 제약 검사를 매우 효율적으로 만들어준다. 예를 들어, 궤적의 특정 구간이 장애물과 충돌하는지 확인하기 위해 곡선 전체를 촘촘히 샘플링하여 검사하는 대신, 해당 구간에 영향을 주는 제어점들의 볼록 포체가 장애물과 겹치는지 먼저 확인할 수 있다. 만약 볼록 포체 자체가 장애물과 떨어져 있다면, 그 안의 곡선 또한 장애물과 절대 충돌하지 않음을 계산 없이 보장할 수 있다.


볼록 포체 특성은 드론의 최대 속도, 가속도, 저크(jerk)와 같은 동역학적 제약(kinodynamic constraints)을 검사하는 데에도 결정적인 역할을 한다. B-스플라인의 미분(속도, 가속도 등) 또한 원래 제어점들의 차분을 새로운 제어점으로 하는 또 다른 B-스플라인으로 표현된다. 예를 들어, 속도 궤적 $\dot{\mathbf{p}}(t)$는 제어점 $\mathbf{V}_i = (\mathbf{Q}_{i+1} - \mathbf{Q}_i) / \Delta t$로 표현되고, 가속도 궤적 $\ddot{\mathbf{p}}(t)$는 제어점 $\mathbf{A}_i = (\mathbf{V}_{i+1} - \mathbf{V}_i) / \Delta t$로 표현된다.

따라서, 궤적 전체의 속도가 최대 속도 $v_{max}$를 넘지 않는지 확인하기 위해, 모든 속도 제어점 $\mathbf{V}_i$의 크기가 $v_{max}$를 넘지 않는지만 확인하면 된다. 만약 모든 속도 제어점들이 원점을 중심으로 하는 반경 $v_{max}$의 구 안에 있다면, 그들의 볼록 포체 또한 그 구 안에 존재하게 되고, 따라서 실제 속도 궤적 전체가 $v_{max}$를 절대 넘지 않음을 보장할 수 있다.1 이처럼 연속적인 시간 전체에 대한 제약 조건을 이산적인 소수의 제어점에 대한 제약 조건으로 변환함으로써, 최적화 과정에서 반복적으로 수행되어야 하는 제약 검사의 계산 비용을 극적으로 줄일 수 있다.


EGO-Planner의 목표는 단순히 장애물을 피하는 경로를 찾는 것이 아니라, 안전하면서도 매끄럽고, 드론의 물리적 한계를 준수하는 '최적'의 궤적을 찾는 것이다. 이를 위해 여러 상충하는 목표들을 균형 있게 고려하는 다중 목표 비용 함수(Multi-objective Cost Function)를 정의하고 이를 최소화한다.2


전체 비용 함수 $F_{total}$는 매끄러움($f_{smoothness}$), 충돌($f_{collision}$), 동역학적 타당성($f_{dynamics}$)이라는 세 가지 주요 비용 항의 가중치 합으로 구성된다.
$$
F_{total} = \lambda_s f_{smoothness} + \lambda_c f_{collision} + \lambda_d f_{dynamics}
$$
여기서 $\lambda_s, \lambda_c, \lambda_d$는 각 비용 항의 상대적 중요도를 조절하는 가중치 파라미터이다. 최적화의 목표는 이 $F_{total}$을 최소화하는 B-스플라인 제어점 집합 $\{\mathbf{Q}_i\}$를 찾는 것이다.


매끄러움 비용은 궤적의 불필요한 구부러짐이나 급격한 가속 변화를 억제하여, 드론이 부드럽게 비행하고 에너지 소모를 줄이도록 유도한다. 이는 보통 궤적의 고차 미분(주로 가속도 또는 저크)의 제곱을 전체 시간에 대해 적분한 값으로 정의된다.
$$
f_{smoothness} = \int_{t_0}^{t_M} ||\mathbf{p}^{(k)}(t)||^2 dt, \quad (k=2 \text{ for acceleration, } k=3 \text{ for jerk})
$$
B-스플라인의 미분 공식 덕분에 이 적분 값은 제어점 $\mathbf{Q}_i$에 대한 간단한 이차 형식(Quadratic form) $\mathbf{Q}^T \mathbf{M} \mathbf{Q}$로 표현될 수 있다. 이는 비용 함수를 볼록(convex)하게 만들어 최적해를 빠르고 안정적으로 찾는 데 유리하게 작용한다.


충돌 비용은 1.1절에서 설명한 ESDF-free 방식으로 계산된다. 궤적을 따라 샘플링된 포인트 $\mathbf{p}(t_j)$가 장애물 내부에 있을 경우, 이 포인트와 가이드 경로상의 가장 가까운 안전한 포인트 $\mathbf{p}_{free,j}$ 간의 거리 제곱에 비례하는 페널티를 부과한다.
$$
f_{collision} = \sum_{j} w_j \cdot \max(0, ||\mathbf{p}(t_j) - \mathbf{p}_{free,j}||^2 - d_{safe}^2)
$$
여기서 $d_{safe}$는 최소 안전 거리이며, $w_j$는 페널티 가중치이다. 이 비용 함수는 비볼록(non-convex)이지만, 궤적이 장애물과 떨어져 있을 때는 비용이 0이 되어 계산에 영향을 주지 않는다. 그래디언트는 $\mathbf{p}(t_j)$에서 $\mathbf{p}_{free,j}$ 방향으로 궤적을 밀어내는 형태로 계산된다.


동역학적 비용은 궤적의 속도, 가속도 등이 드론의 물리적 한계($v_{max}, a_{max}$)를 초과할 경우에만 페널티를 부과하는 함수이다. 1.2절에서 설명한 볼록 포체 특성을 활용하여, 각 제어점의 속도($\mathbf{V}_i$)와 가속도($\mathbf{A}_i$)가 한계를 초과하는지 검사한다.
$$
f_{dynamics} = \sum_i \max(0, ||\mathbf{V}_i||^2 - v_{max}^2) + \sum_i \max(0, ||\mathbf{A}_i||^2 - a_{max}^2)
$$
이 비용 함수 역시 비볼록이지만, 제약 조건을 만족하는 영역에서는 비용이 0이 된다. 최적화 과정에서 이 비용 항은 제어점의 속도와 가속도를 물리적 한계 내로 수렴시키는 역할을 한다.


경사도 기반 최적화는 강력하지만, 복잡한 환경에서는 지역 최솟값(local minima)에 빠지거나 동역학적으로 불가능한 해를 내놓을 수 있다. EGO-Planner는 이러한 문제를 해결하고 강건성(robustness)을 높이기 위해 최적화 이후의 궤적 재조정(refinement) 단계를 포함한다. 이러한 실패 처리 메커니즘의 도입과 발전은 초기 `EGO-Planner`에서 `EGO-Swarm`으로 진화하는 과정에서 나타난 핵심적인 개선 사항이다.

초기 경사도 기반 최적화는 효율적이지만 근본적인 약점을 가진다. 센서의 시야가 제한적일 때, 장애물 뒤편의 상황을 알지 못해 막다른 길로 들어가는 지역 최솟값에 쉽게 빠질 수 있다.1

`EGO-Swarm` 논문에서는 이러한 지역 최솟값 문제를 해결하고 강건성을 향상시키기 위해 "경량의 위상학적 궤적 생성 방법(lightweight topological trajectory generation method)"을 통합했다고 명시적으로 언급한다.10 이는 단순한 경사 하강법만으로는 실제 환경의 복잡성을 모두 해결할 수 없음을 인정하고, 더 넓은 탐색을 통해 다른 경로의 가능성을 찾는 상위 레벨의 재계획 전략을 추가한 것이다. 즉, EGO 시스템의 발전은 단순히 군집 기능을 추가한 것을 넘어, 그래디언트 기반 최적화라는 핵심 엔진이 실패했을 때 이를 보완하고 시스템 전체의 성공률을 높이는 방향으로 이루어졌다. 이처럼 EGO-Planner 시스템은 효율성을 위한 경사도 기반 최적화와 강건성을 위한 위상학적 재계획이라는 다층적 방어 체계를 갖추고 있다.


최적화가 끝난 후에도 생성된 궤적이 여전히 최대 속도나 가속도 제약을 위반하는 경우가 있다. 이는 대부분 목표 지점까지 할당된 비행 시간이 너무 짧아서 물리적으로 불가능한 속도/가속도를 요구하기 때문이다. 이 경우, EGO-Planner는 궤적의 공간적인 형태는 그대로 유지하면서 전체 비행 시간($\Delta T$)을 점진적으로 늘리는 시간 재할당 기법을 사용한다.2

시간을 늘리면 동일한 거리를 더 오랫동안 비행하게 되므로, 필요한 평균 속도와 가속도가 자연스럽게 감소한다. EGO-Planner는 동역학적 제약이 만족될 때까지 시간을 조금씩 늘려가며 궤적을 재조정한다. 이 과정은 궤적의 기하학적 형태를 크게 바꾸지 않고 동역학적 타당성을 확보하는 효과적인 방법이다.


시간 재할당을 통해 새로운 총 비행 시간이 결정되면, 이 시간에 맞는 새로운 B-spline 제어점들을 계산해야 한다. 이때 목표는 이전 최적화 단계에서 찾은 (동역학적으로는 비현실적이었지만 공간적으로는 최적에 가까웠던) 궤적의 형태를 최대한 유지하는 것이다. 이를 위해 EGO-Planner는 이전 궤적을 '목표'로 삼아 새로운 B-spline 곡선을 피팅(fitting)하는 문제를 푼다.

여기서 '비등방성(anisotropic)'이라는 개념이 도입된다.2 피팅 오차를 계산할 때, 궤적의 진행 방향(axial direction)으로의 오차와 그에 수직인 측면 방향(radial direction)으로의 오차에 서로 다른 가중치를 부여하는 것이다. 일반적으로 측면 방향의 오차에 더 큰 페널티를 부여한다. 이는 드론이 원래의 경로에서 옆으로 벗어나는 것을 최소화하면서, 진행 방향으로는 속도를 조절할 수 있는 유연성을 부여하는 효과를 가진다. 결과적으로 이 비등방성 피팅 과정은 불필요한 경로 이탈을 억제하고, 더욱 매끄러우면서도 동역학적으로 타당한 최종 궤적을 생성하는 데 기여한다.


EGO-Planner를 실제로 구동하기 위해서는 로봇 운영체제(ROS)를 기반으로 하는 복잡한 소프트웨어 시스템을 구성해야 한다. 이 장에서는 EGO-Planner(최신 버전인 `EGO-Swarm` 기준)의 소프트웨어 아키텍처, 개발 환경 구축 방법, 핵심 파라미터 튜닝, 그리고 시뮬레이션 검증까지의 전 과정을 상세히 다룬다.


EGO-Planner는 단일 프로그램이 아닌, 여러 ROS 노드(Node)들이 토픽(Topic)을 통해 메시지를 주고받으며 상호작용하는 분산 시스템으로 동작한다.


실제 드론에 EGO-Planner를 탑재할 때의 일반적인 ROS 노드 구성은 다음과 같다.

- **카메라 드라이버 노드 (`realsense-ros` 등):** Intel RealSense와 같은 깊이 카메라로부터 원시 이미지 데이터(적외선, 깊이, 컬러)를 받아 ROS 토픽으로 발행(publish)한다.
- **상태 추정 노드 (`VINS-Fusion` 등):** 카메라의 이미지 데이터와 IMU(관성 측정 장치) 데이터를 입력받아 시각-관성 주행 거리 측정(VIO)을 수행한다. 이를 통해 드론의 현재 위치, 자세, 속도 등의 상태 정보(Odometry)를 계산하여 `/odometry`와 같은 토픽으로 발행한다.9
- **플래너 노드 (`ego-planner-swarm`):** 본 시스템의 핵심. 상태 추정 노드로부터 드론의 현재 상태를, 카메라 드라이버로부터 환경 정보(포인트 클라우드 또는 깊이 이미지)를, 그리고 사용자 또는 GCS로부터 목표 지점 정보를 입력받는다. 이 정보들을 종합하여 1부에서 설명한 최적화 과정을 통해 안전하고 매끄러운 B-스플라인 궤적을 생성한다.
- **FCU 통신 노드 (`mavros`):** ROS와 비행 컨트롤러(FCU) 펌웨어(PX4, ArduPilot) 사이의 통신을 중계하는 브릿지 역할을 한다. EGO-Planner가 생성한 궤적(setpoints)을 FCU가 이해할 수 있는 MAVLink 프로토콜 메시지로 변환하여 전송하고, FCU로부터 받은 상태 정보(배터리, GPS 등)를 ROS 토픽으로 발행한다.

이 노드들은 서로 독립적으로 실행되지만, 토픽을 통해 긴밀하게 연결되어 하나의 거대한 자율 비행 시스템을 이룬다.


EGO-Planner 시스템의 데이터 흐름을 이해하기 위해서는 주요 토픽의 역할 파악이 필수적이다. `ego-planner-swarm`의 `launch` 파일 11과 소스 코드를 분석하면 다음과 같은 핵심 토픽들을 식별할 수 있다.

- **입력 토픽:**
  - `~odom_world` (remap to $(arg odometry_topic)`): VIO 노드로부터 받는 드론의 전역 좌표계 기준 위치/자세/속도 정보. EGO-Planner가 자신의 위치를 파악하는 데 사용된다.
  - `~grid_map/cloud` (remap to $(arg cloud_topic)`) 또는 `~grid_map/depth` (remap to $(arg depth_topic)`): 카메라로부터 받는 3D 포인트 클라우드 또는 깊이 이미지. 플래너 내부의 로컬 맵을 업데이트하여 장애물 정보를 구축하는 데 사용된다.
  - `/goal` (RViz 등에서 발행): 사용자가 지정하는 목표 지점. 플래너가 나아가야 할 방향을 결정한다.
- **출력 토픽:**
  - `~planning/bspline` (remap to `/drone_$(arg drone_id)_planning/bspline`): EGO-Planner가 최종적으로 계산한 B-스플라인 궤적 정보. 이 정보는 MAVROS를 통해 FCU로 전달된다.
  - `~planning/data_display`: RViz에서의 시각화를 위한 각종 디버깅 정보(예: 로컬 맵, 제어점, 최적화 과정 등)를 발행한다.
- **상호작용 토픽 (군집 비행 시):**
  - `~planning/broadcast_bspline_from_planner` / `~planning/broadcast_bspline_to_planner`: 군집 비행 시, 다른 드론들과 자신의 계획된 궤적을 공유하고 다른 드론의 궤적을 수신하여 서로 충돌을 회피하는 데 사용된다.11


위 구성 요소들을 종합한 전체 데이터 처리 파이프라인은 다음과 같다.9

1. **인식(Perception):** 깊이 카메라가 주변 환경을 촬영하여 깊이 이미지를 생성한다.
2. **맵핑(Mapping):** EGO-Planner 노드 내부의 맵핑 모듈이 깊이 이미지를 3D 포인트 클라우드로 변환하고, 이를 로컬 점유 격자 지도에 통합하여 주변 장애물 지도를 실시간으로 업데이트한다.
3. **상태 추정(State Estimation):** VIO 노드(`VINS-Fusion`)가 카메라 이미지와 IMU 데이터를 융합하여 드론의 현재 6-DOF(위치, 자세) 상태를 정밀하게 추정하고 `/odometry` 토픽으로 발행한다.
4. **경로 계획(Planning):** EGO-Planner 노드는 현재 위치(`/odometry`), 주변 지도, 그리고 목표 지점을 입력받아 충돌 없고 동역학적으로 가능한 B-spline 궤적을 계산한다.
5. **제어(Control):** 계산된 궤적은 MAVROS 노드로 전달된다. MAVROS는 이 궤적을 일정 시간 간격의 위치, 속도, 가속도 목표값(setpoints)으로 변환하여 MAVLink 메시지 형태로 FCU에 전송한다.
6. **실행(Execution):** FCU는 수신한 목표값을 바탕으로 모터의 PWM 신호를 정밀하게 제어하여 드론이 생성된 궤적을 따라 비행하도록 한다.

이 모든 과정이 밀리초(ms) 단위로 끊임없이 반복되며 실시간 자율 비행이 이루어진다.


EGO-Planner를 컴파일하고 실행하기 위해서는 특정 버전의 운영체제와 ROS, 그리고 여러 라이브러리 설치가 필요하다.

- **필수 조건:** EGO-Planner 및 EGO-Swarm은 Ubuntu 16.04, 18.04, 20.04 환경에서 테스트되었다. ROS는 각 버전에 맞는 Melodic (18.04) 또는 Noetic (20.04)의 `ros-desktop-full` 버전 설치가 권장된다.5

- **의존성 패키지 설치:** EGO-Planner가 사용하는 시뮬레이터(`uav_simulator`)는 선형대수 라이브러리인 Armadillo를 필요로 한다. 또한, 실제 하드웨어에서 최대 성능을 끌어내기 위해 CPU 주파수를 수동으로 고정하는 `cpufrequtils` 패키지 설치가 권장된다.5

  ```Bash
  sudo apt-get update
  sudo apt-get install libarmadillo-dev cpufrequtils
  ```

- **소스 코드 클론 및 컴파일:** 최신 버전인 `EGO-Swarm` 저장소를 기준으로 컴파일 과정은 다음과 같다. GPU(NVIDIA)가 장착된 시스템에서는 시뮬레이션의 깊이 이미지 생성을 가속화하기 위해 CUDA를 활성화할 수 있다. 이는 `ego-planner-swarm/src/local_sensing/CMakeLists.txt` 파일 내의 `set(ENABLE_CUDA false)`를 `true`로 변경하여 설정할 수 있다.5

다음 표는 EGO-Swarm을 처음부터 컴파일하는 과정을 단계별로 정리한 것이다.

**Table 1: EGO-Swarm 컴파일 단계별 명령어**

| 단계              | 명령어                                                       | 설명                                                         |
| ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1. ROS 환경 설정  | `source /opt/ros/<distro>/setup.bash`                        | 현재 터미널 세션에 ROS 환경 변수를 로드한다. `<distro>`는 `melodic` 또는 `noetic`이다. |
| 2. 작업 공간 생성 | `mkdir -p ~/catkin_ws/src && cd ~/catkin_ws/src`             | ROS 패키지를 관리할 catkin 작업 공간을 생성하고 해당 디렉토리로 이동한다. |
| 3. 의존성 설치    | `sudo apt-get install libarmadillo-dev`                      | 시뮬레이터에 필요한 Armadillo 라이브러리를 설치한다.7        |
| 4. 소스 코드 클론 | `git clone https://github.com/ZJU-FAST-Lab/ego-planner-swarm.git` | GitHub에서 EGO-Swarm 소스 코드를 다운로드한다.7              |
| 5. 컴파일         | `cd.. && catkin_make -DCMAKE_BUILD_TYPE=Release`             | 작업 공간의 루트로 이동하여 패키지를 최적화된 릴리즈 모드로 컴파일한다.7 |
| 6. 환경 설정 적용 | `source devel/setup.bash`                                    | 컴파일된 패키지를 현재 터미널의 ROS 환경에 등록하여 `roslaunch` 등으로 실행할 수 있게 한다. |


EGO-Planner의 성능과 행동은 `launch` 파일과 `yaml` 설정 파일에 정의된 수많은 파라미터에 의해 결정된다. 이 파라미터들을 드론의 사양과 임무 환경에 맞게 적절히 튜닝하는 것은 성공적인 비행을 위한 필수 과정이다.


`ego-planner-swarm` 패키지의 `launch` 디렉토리에는 다양한 시나리오를 위한 실행 파일들이 있다. 예를 들어, `swarm.launch` 파일은 여러 대의 드론을 시뮬레이션하는 데 사용되며, 내부적으로 각 드론에 대한 `ego-planner-swarm` 노드를 실행한다. `launch` 파일의 주요 역할은 다음과 같다.

- **노드 실행:** `<node>` 태그를 사용하여 `ego_planner_node`와 같은 ROS 노드를 실행한다.
- **파라미터 로딩:** `<param>` 태그를 사용하여 ROS 파라미터 서버에 값을 설정한다. 이 값들은 노드가 실행될 때 읽어들여 내부 변수로 사용한다.11
- **토픽 리매핑(Remapping):** `<remap>` 태그를 사용하여 노드가 구독하거나 발행하는 토픽의 기본 이름을 다른 이름으로 변경한다. 이는 여러 드론이 동일한 노드를 실행할 때 토픽 이름이 충돌하지 않도록 하거나, 다른 패키지(예: VIO)의 출력 토픽 이름에 맞춰 입력 토픽을 연결하는 데 사용된다.11


다음 표는 EGO-Planner의 성능에 큰 영향을 미치는 주요 파라미터들을 정리하고, 그 역할과 튜닝 가이드를 제공한다. 이 값들은 주로 `launch` 파일이나 관련 `yaml` 파일에서 찾을 수 있다.11

**Table 2: EGO-Planner 주요 파라미터, 역할 및 튜닝 가이드**

| 파라미터 이름                     | 설명                                                         | 기본값 예시 | 튜닝 가이드                                                  |
| --------------------------------- | ------------------------------------------------------------ | ----------- | ------------------------------------------------------------ |
| `fsm/planning_horizon`            | 한 번의 재계획으로 생성하는 궤적의 최대 거리(m).             | 5.0         | 센서의 유효 탐지 거리보다 약간 작게 설정하는 것이 일반적이다. 이 값이 너무 크면 먼 곳의 불확실한 정보까지 고려하게 되어 계산량이 증가하고, 너무 작으면 근시안적인 경로를 생성하여 장애물에 갇히기 쉽다. |
| `optimization/max_vel`            | 궤적이 가질 수 있는 최대 속도(m/s).                          | 3.0         | 드론의 모터 추력, 기체 무게, 배터리 성능 등 물리적 한계를 고려하여 설정한다. 값을 높이면 더 공격적인 비행이 가능하지만, 제어 안정성이 떨어질 수 있다. |
| `optimization/max_acc`            | 궤적이 가질 수 있는 최대 가속도(m/s²).                       | 3.0         | 최대 속도와 마찬가지로 드론의 동역학적 성능에 맞춰 설정한다. 높은 가속도는 빠른 회피 기동을 가능하게 하지만, 배터리 소모를 급격히 증가시킨다. |
| `grid_map/resolution`             | 플래너가 사용하는 로컬 맵의 해상도(m/voxel).                 | 0.15        | 값이 작을수록(고해상도) 좁은 틈새를 통과하는 등 정밀한 계획이 가능하지만, 메모리 사용량과 맵 업데이트 계산량이 기하급수적으로 증가한다. 비행 환경의 복잡도와 동반 컴퓨터의 성능 사이에서 절충이 필요하다. |
| `optimization/dist0`              | 충돌 회피를 위한 최소 안전 거리(m). 궤적은 장애물로부터 이 거리 이상 떨어지려고 시도한다. | 0.4         | 드론의 물리적 크기(프로펠러 포함)보다 충분히 큰 값으로 설정해야 한다. 환경이 매우 좁고 복잡하다면 값을 약간 줄여 통과 가능성을 높일 수 있지만, 충돌 위험이 커진다. |
| `manager/control_points_distance` | B-스플라인을 구성하는 제어점들 사이의 공간적 거리(m).        | 0.6         | 이 값이 작을수록 궤적이 더 복잡하고 구불구불한 형태를 가질 수 있게 된다 (자유도 증가). 하지만 최적화해야 할 변수의 수가 늘어나 계산 시간이 길어질 수 있다. |
| `fsm/flight_type`                 | 비행 모드를 지정하는 정수 값. (예: 1: 수동 목표 지정, 2: 사전 정의된 웨이포인트 비행 등). | 1           | 시뮬레이션이나 실제 실험의 목적에 맞게 설정한다. 예를 들어, RViz에서 마우스 클릭으로 목표를 지정하려면 1로 설정한다.11 |


하드웨어에 탑재하기 전, 시뮬레이션 환경에서 알고리즘의 동작을 충분히 검증하는 것은 매우 중요하다. EGO-Planner는 Gazebo 시뮬레이터와 RViz 시각화 도구를 연동한 검증 환경을 제공한다.


시뮬레이션을 실행하기 위해서는 일반적으로 두 개의 터미널이 필요하다.

1. **첫 번째 터미널 (RViz 실행):**

   ```Bash
   source ~/catkin_ws/devel/setup.bash
   roslaunch ego_planner rviz.launch
   ```

   이 명령어는 RViz를 실행하고, EGO-Planner 시뮬레이션에 최적화된 설정 파일을 로드한다. RViz 화면에는 드론 모델, 센서 데이터, 계획된 경로, 로컬 맵 등이 표시된다.5

2. **두 번째 터미널 (시뮬레이터 및 플래너 실행):**

   ```Bash
   source ~/catkin_ws/devel/setup.bash
   roslaunch ego_planner run_in_sim.launch
   ```

   이 명령어는 Gazebo 시뮬레이터를 배경에서 실행하고, 가상의 드론과 센서, 그리고 `ego_planner_node`를 포함한 모든 ROS 노드를 실행시킨다.5 군집 비행을 테스트하려면 `swarm.launch` 파일을 사용한다.


시뮬레이션이 성공적으로 실행되면, RViz 창을 통해 드론과 상호작용하고 플래너의 동작을 분석할 수 있다.

- **목표 지점 설정:** RViz 상단의 '2D Nav Goal' 버튼을 클릭한 후, 맵 상의 원하는 위치를 클릭하고 드래그하여 드론의 목표 지점과 방향을 설정할 수 있다.

- **시각적 확인:** 드론이 목표 지점을 향해 비행하면서 장애물을 회피하는 과정을 실시간으로 확인할 수 있다. RViz의 'Displays' 패널에서 B-스플라인 궤적(주황색 곡선), 제어점(회색 점), 로컬 맵(색깔 있는 복셀) 등의 표시를 켜거나 끌 수 있다.

- **디버깅:** 만약 플래너가 예상대로 동작하지 않는다면, `rqt_graph` 명령어를 사용하여 현재 실행 중인 노드들과 그들 사이의 토픽 연결 상태를 시각적으로 확인할 수 있다. 이를 통해 특정 토픽이 제대로 발행되지 않거나 노드 간 연결이 끊어졌는지 등을 쉽게 파악할 수 있다.5 또한, 

  `rostopic echo <topic_name>` 명령어로 특정 토픽의 메시지 내용을 실시간으로 모니터링하여 데이터의 이상 유무를 확인할 수 있다.


시뮬레이션 검증이 완료되면, EGO-Planner 시스템을 실제 드론 하드웨어에 통합하는 단계로 넘어간다. 이 과정은 동반 컴퓨터, 비행 컨트롤러, 센서 등 여러 하드웨어 구성 요소의 신중한 선택과 이들 간의 정밀한 연동 설정을 요구한다.


성공적인 자율 비행을 위해서는 각 하드웨어 구성 요소가 자신의 역할을 명확히 수행하고 유기적으로 상호작용해야 한다.


EGO-Planner 기반 자율 비행 드론의 핵심 하드웨어 구성 요소와 그 역할은 다음과 같다.9

- **동반 컴퓨터 (Companion Computer):** 시스템의 '두뇌' 역할을 담당한다. Ubuntu와 ROS를 실행하며, EGO-Planner의 경로 계획, VIO를 통한 위치 추정, 맵핑 등 계산 집약적인 고수준 연산을 모두 처리한다.13
- **비행 컨트롤러 (Flight Controller, FCU):** 시스템의 '소뇌'와 같다. Pixhawk, CubePilot 등의 보드가 사용되며, PX4나 ArduPilot과 같은 실시간 운영체제(RTOS) 기반 펌웨어를 실행한다. 동반 컴퓨터로부터 받은 제어 명령을 바탕으로 모터의 RPM을 직접 제어하고, 기체의 자세를 안정적으로 유지하는 저수준 실시간 제어를 담당한다.9
- **센서 (Sensors):** 시스템의 '눈과 귀'이다.
  - **깊이 카메라 (Depth Camera):** Intel RealSense D435i, D455 등이 주로 사용된다. 주변 환경의 3차원 구조를 파악하여 장애물 회피를 위한 맵을 생성하는 데 필수적이다.5 D435i와 같이 IMU가 내장된 모델은 VIO 성능 향상에 유리하다.
  - **IMU (Inertial Measurement Unit):** 가속도계와 자이로스코프로 구성되며, 드론의 각속도와 가속도를 측정한다. FCU와 깊이 카메라에 각각 내장되어 있으며, VIO와 자세 제어에 핵심적인 정보를 제공한다.
- **기타:** 기체 프레임, 모터, ESC(전자 변속기), 프로펠러, 배터리 등 드론의 기본적인 비행을 구성하는 요소들과, GCS와의 통신을 위한 텔레메트리 모듈(Telemetry Radio) 또는 WiFi 동글이 포함된다.


다음 표는 EGO-Planner 구동을 위한 하드웨어 조합을 '연구/개발용'과 '고성능/실증용'으로 나누어 제시한다. 사용자의 예산과 목표 성능에 따라 적절한 조합을 선택할 수 있다.9

**Table 3: EGO-Planner 구동을 위한 표준 및 고성능 하드웨어 구성 예시**

| 구성 요소     | 연구/개발용 (기본)             | 고성능/실증용 (권장)                       | 비고                                                         |
| ------------- | ------------------------------ | ------------------------------------------ | ------------------------------------------------------------ |
| 동반 컴퓨터   | Raspberry Pi 4 (4GB+)          | NVIDIA Jetson Orin Nano / Xavier NX        | EGO-Planner와 VIO를 실시간으로 동시에 구동하려면 강력한 병렬 처리 능력을 갖춘 NVIDIA Jetson 시리즈가 사실상 필수적이다.14 |
| 비행 컨트롤러 | Pixhawk 4 / Holybro Pixhawk 5X | Holybro Pixhawk 6X / CubePilot Cube Orange | 최신 버전의 PX4 또는 ArduPilot 펌웨어를 지원하는 FCU를 선택한다. 커넥터 표준과 처리 성능을 고려하여 선택한다.9 |
| 깊이 카메라   | Intel RealSense D435i          | Intel RealSense D455                       | D455는 D435i에 비해 깊이 인식 범위가 더 넓고 성능이 우수하다. VIO를 위해 IMU가 내장된 'i' 모델 사용이 권장된다. |
| 통신 (GCS)    | 433/915MHz Telemetry Radio     | 동반 컴퓨터를 통한 WiFi                    | 저대역폭의 상태 모니터링은 텔레메트리 모듈로 충분하지만, 영상 스트리밍이나 고속의 파라미터 변경을 위해서는 WiFi 연결이 필요하다. |
| 펌웨어        | PX4                            | PX4                                        | EGO-Planner 및 관련 커뮤니티(ZJU-FAST-Lab)에서 PX4 펌웨어를 사용한 예시가 다수 공개되어 있어 호환성 및 자료 확보에 유리하다.9 |


고수준 연산을 담당하는 동반 컴퓨터와 저수준 제어를 담당하는 FCU를 안정적으로 연결하는 것은 전체 시스템의 성능을 좌우하는 매우 중요한 과정이다.


가장 일반적인 연결 방식은 동반 컴퓨터의 USB 포트 또는 GPIO 핀과 FCU의 `TELEM2` (Telemetry 2) 포트를 UART(Serial) 통신으로 연결하는 것이다.17

- **배선:** FCU의 `TELEM2` 포트 핀아웃을 확인하여, `TX` (송신) 핀을 동반 컴퓨터의 `RX` (수신) 핀에, `RX` 핀을 동반 컴퓨터의 `TX` 핀에 교차하여 연결한다. 또한, `GND` (접지) 핀을 서로 연결하여 전위차를 맞춰야 한다.
- **주의사항:** **절대로 FCU의 `VCC` (+5V) 핀을 동반 컴퓨터의 전원 핀에 연결해서는 안 된다**.17 동반 컴퓨터와 FCU는 각각 독립적인 전원(BEC 또는 전원 모듈)으로부터 전력을 공급받아야 한다. 잘못된 전원 연결은 보드 손상의 주된 원인이 된다.


FCU가 `TELEM2` 포트를 통해 동반 컴퓨터와 MAVLink 프로토콜로 통신하도록 설정해야 한다. 이는 GCS 소프트웨어(예: QGroundControl)를 통해 FCU에 연결하여 파라미터를 변경함으로써 수행한다.17

1. QGroundControl을 실행하고 USB 케이블로 FCU를 컴퓨터에 연결한다.
2. 'Parameters' 탭으로 이동하여 다음 파라미터를 검색하고 값을 변경한다.
   - `MAV_1_CONFIG`: `TELEM 2`로 설정한다. 이는 MAVLink 인스턴스 1번을 `TELEM 2` 포트에 할당하라는 의미이다.
   - `MAV_1_MODE`: `Onboard`로 설정한다. 이는 해당 포트가 동반 컴퓨터와 통신하는 용도임을 지정한다.
   - `SER_TEL2_BAUD`: `921600` 또는 그 이상의 높은 값으로 설정한다. 동반 컴퓨터와의 통신은 대량의 데이터를 빠르게 주고받아야 하므로 높은 전송 속도(baud rate)가 필수적이다.
3. 파라미터 변경 후 FCU를 재부팅하여 설정을 적용한다.


동반 컴퓨터에서는 MAVROS 노드가 FCU와의 통신을 담당한다. MAVROS의 `launch` 파일(예: `px4.launch`)에서 FCU가 연결된 시리얼 포트와 전송 속도를 지정해야 한다.20

```XML
<arg name="fcu_url" default="/dev/ttyTHS1:921600" />
<node pkg="mavros" type="mavros_node" name="mavros" output="screen">
    <param name="fcu_url" value="$(arg fcu_url)" />
  ...
</node>
```

위 예시에서 `fcu_url`은 `/dev/ttyTHS1` 시리얼 포트를 `921600` bps의 속도로 사용하겠다는 의미이다. (NVIDIA Jetson 보드의 UART 포트 이름은 보통 `ttyTHS*` 형태이다). MAVROS는 이 연결을 통해 FCU의 항공우주 표준 좌표계인 NED(North-East-Down)를 ROS 표준 좌표계인 ENU(East-North-Up)로 자동으로 변환해주는 중요한 역할도 수행하므로, 개발자는 좌표계 변환의 복잡함에서 벗어날 수 있다.20


자율 비행 시스템에서 수동 조종기(RC)는 비상 상황 시 제어권을 즉시 가져올 수 있는 최종 안전장치이며, 지상관제시스템(GCS)은 드론의 상태를 실시간으로 모니터링하고 임무를 지시하는 관제탑 역할을 한다. 이 둘을 EGO-Planner 시스템과 효과적으로 통합하는 열쇠는 MAVROS에 있다.

MAVROS는 단순히 동반 컴퓨터와 FCU를 연결하는 것을 넘어, 통신 허브 또는 멀티플렉서(multiplexer)처럼 동작할 수 있다.20 즉, EGO-Planner로부터 받은 고수준의 제어 명령을 FCU로 전달하는(`fcu_url`) 동시에, FCU와 주고받는 모든 MAVLink 데이터를 네트워크를 통해 GCS로 전달(`gcs_url`)하는 것이 가능하다.21 이 아키텍처는 "Man-in-the-Loop" 시스템을 구축하는 가장 효과적인 방법이다. 드론은 EGO-Planner에 의해 자율적으로 비행하고, 지상의 조종사는 GCS 화면을 통해 드론의 현재 상태, 센서 데이터, 계획된 경로 등을 실시간으로 모니터링하며, 위급 상황 시에는 즉시 RC 조종기로 수동 모드로 전환하여 제어권을 확보할 수 있다. 이는 사용자의 "드론과 조종기 구성"에 대한 요구를 안전하고 체계적으로 충족시키는 핵심적인 설계 방식이다.


GCS 연동은 동반 컴퓨터의 MAVROS를 GCS를 위한 프록시(Proxy)로 활용하는 구조를 통해 이루어진다. 데이터 흐름은 다음과 같다: **FCU ↔ (Serial) ↔ MAVROS on Companion Computer ↔ (WiFi/Ethernet) ↔ GCS on Ground Laptop**. 이 구조에서 MAVROS는 FCU로부터 받은 모든 MAVLink 데이터를 UDP 패킷으로 변환하여 GCS가 실행 중인 컴퓨터로 전송한다.22


MAVROS `launch` 파일에서 `gcs_url` 파라미터를 설정하여 GCS로의 데이터 포워딩을 활성화할 수 있다.

```XML
<arg name="fcu_url" default="/dev/ttyTHS1:921600" />
<arg name="gcs_url" default="udp://@192.168.1.100:14550" />
<node pkg="mavros" type="mavros_node" name="mavros" output="screen">
    <param name="fcu_url" value="$(arg fcu_url)" />
    <param name="gcs_url" value="$(arg gcs_url)" />
  ...
</node>
```

위 예시에서 `gcs_url`은 GCS가 실행되고 있는 컴퓨터의 IP 주소(`192.168.1.100`)와 QGroundControl이 기본적으로 사용하는 UDP 포트(`14550`)를 지정한다.21 이렇게 설정하면 MAVROS가 실행됨과 동시에 GCS는 자동으로 드론에 연결되어 각종 정보를 표시하기 시작한다.


RC 조종기는 자율 비행의 안전을 위한 최후의 보루이다. 조종기의 스위치 중 하나를 FCU의 비행 모드(Flight Mode) 변경 기능에 할당해야 한다. 일반적으로 다음과 같은 모드들을 설정한다.

- **Manual/Stabilized Mode:** 조종사가 직접 드론의 자세를 제어하는 완전 수동 모드.
- **Position Control Mode:** 조종사가 드론의 위치를 제어하는 모드. 조종 스틱을 놓으면 드론이 그 자리에 정지 비행(hovering)한다. 이륙 및 착륙, 비상시 위치 고정에 사용된다.
- **Offboard Mode:** 동반 컴퓨터로부터 제어 명령을 받는 자율 비행 모드. EGO-Planner가 생성한 궤적을 따라 비행하기 위해서는 반드시 이 모드로 전환해야 한다.

비행 전에는 항상 Position Control 모드에서 이륙하여 기체의 안정성을 확인한 후, 안전이 확보된 상태에서 스위치를 조작하여 Offboard 모드로 전환한다. 비행 중 어떤 이상이 감지되더라도 즉시 Position Control 모드나 Manual 모드로 전환하여 조종사가 제어권을 되찾을 수 있도록 반복적으로 훈련해야 한다.

결론 및 제언


EGO-Planner는 쿼드로터의 실시간 지역 경로 계획 분야에서 중요한 기술적 진보를 이루었다. 그 핵심 기여는 다음 두 가지로 요약할 수 있다.

1. **ESDF-Free 경사도 기반 최적화:** 전통적인 경사도 기반 플래너들이 의존했던 ESDF의 계산적 병목 현상을, 충돌 시에만 국소적으로 그래디언트를 계산하는 새로운 충돌 비용 함수를 통해 해결했다. 이는 제한된 온보드 컴퓨팅 자원에서도 빠르고 효율적인 궤적 최적화를 가능하게 하여, 고속 자율 비행의 문턱을 낮추었다.1
2. **B-스플라인 기반 궤적 표현 및 최적화:** 궤적을 B-스플라인으로 표현함으로써 매끄러움($C^2$ 연속성)을天然히 보장하고, 볼록 포체 특성을 활용하여 동역학적 제약 조건을 이산적인 제어점에 대한 문제로 변환함으로써 최적화 과정의 계산 효율을 극대화했다.2

이러한 혁신을 통해 EGO-Planner는 계산 시간, 궤적 품질, 안전성 측면에서 높은 성능을 달성했으며, 관련 연구 분야의 발전에 크게 기여했다.


EGO-Planner는 1ms 내외의 빠른 계획 시간을 자랑하며 5, 이론적으로는 경로 계획 단계의 계산 문제를 상당 부분 해결한 것으로 보인다. 하지만 실제 드론 시스템에 이를 성공적으로 적용하는 것은 또 다른 차원의 도전 과제이다. GitHub의 이슈 트래커 24나 실제 구현 사례 연구 9를 깊이 있게 분석해 보면, 실제 시스템의 성능을 좌우하는 병목 지점은 플래너의 계산 속도 자체에서 다른 곳으로 이동했음을 알 수 있다.

성공적인 자율 비행 시스템의 진정한 성능은 가장 약한 연결 고리에 의해 결정된다. EGO-Planner가 아무리 빠르게 최적의 경로를 계산하더라도, 그 입력 데이터가 부정확하다면 결과물은 무용지물이 된다. 새로운 병목 지점은 다음과 같은 시스템 통합 및 상태 추정의 문제들이다.

- **상태 추정(VIO)의 정확성과 강건성:** 플래너는 VIO가 제공하는 현재 위치 정보를 절대적으로 신뢰한다. 만약 VIO가 조명이 급격히 변하거나, 특징점이 없는 환경(예: 흰 벽)에서 위치 추정에 실패하거나 드리프트가 누적되면, 플래너는 잘못된 위치에서 잘못된 계획을 세우게 되어 결국 충돌로 이어진다.
- **센서 데이터의 품질과 캘리브레이션:** 깊이 카메라의 정밀도, IMU의 노이즈 수준, 그리고 이들 간의 시간 동기화 및 외부 파라미터(Extrinsic) 캘리브레이션의 정확성은 VIO와 맵핑 성능에 직접적인 영향을 미친다.
- **시스템 지연 시간(Latency):** 센서 데이터가 동반 컴퓨터로 전달되고, VIO와 플래너가 계산을 마치고, 제어 명령이 MAVROS를 통해 FCU로 전달되기까지의 전체 파이프라인에 존재하는 지연 시간은 시스템의 반응성을 결정한다. 고속으로 비행할수록 이 지연 시간은 치명적인 결과를 초래할 수 있다.

결론적으로, EGO-Planner를 이용한 고성능 자율 비행을 구현하는 것은 단순히 빠른 플래너를 설치하는 문제가 아니라, 정확하고 강건한 상태 추정 시스템을 기반으로 한 저지연(low-latency) 인식-제어 루프를 구축하는 종합적인 시스템 엔지니어링의 과제이다. 사용자는 플래너 자체의 튜닝뿐만 아니라, VIO 성능 최적화, 센서 캘리브레이션, 통신 안정성 확보에 더 많은 노력을 기울여야 한다.


EGO-Planner는 매우 성공적인 프레임워크이지만, 기술은 끊임없이 발전하고 있다. EGO-Planner를 활용하거나 그 이상의 성능을 목표로 하는 개발자들에게 다음과 같은 사항을 제언한다.

- **`EGO-Swarm`으로의 마이그레이션:** 서론에서 언급했듯이, 개발팀 스스로가 `EGO-Swarm`의 사용을 권장하고 있다.5

  `EGO-Swarm`은 단일 드론 환경에서도 기존 `EGO-Planner`보다 개선된 강건성과 지역 최솟값 탈출 메커니즘(위상학적 재계획)을 제공하므로, 특별한 이유가 없다면 `EGO-Swarm`을 기반으로 시스템을 구축하는 것이 바람직하다.10

- **차세대 플래너와의 비교 연구:** 최근 로봇 경로 계획 분야에서는 Diffusion 모델을 이용해 복잡하고 다중 모드(multi-modal)의 궤적 분포를 학습하는 연구 26나, 대규모 언어 모델(LLM)을 이용해 인간의 자연어 지시를 이해하고 행동 계획을 수립하는 연구 27 등 새로운 패러다임이 등장하고 있다. EGO-Planner와 같은 전통적인 최적화 기반 접근법은 그 해의 품질과 제약 조건 만족 측면에서 여전히 강점을 가지지만, 새로운 데이터 기반 접근법들이 제공하는 유연성과 일반화 성능에 주목하고, 이들의 장단점을 비교 분석하여 하이브리드 접근법을 모색하는 것은 의미 있는 연구 방향이 될 것이다.

- **커뮤니티 자원 적극 활용:** 오픈소스로 공개된 프로젝트의 가장 큰 장점 중 하나는 활발한 커뮤니티이다. EGO-Planner 관련 GitHub 저장소의 이슈(Issues) 섹션 24에는 전 세계의 사용자들이 실제 하드웨어에 시스템을 구축하며 겪었던 수많은 문제들과 그에 대한 해결책, 튜닝 노하우가 축적되어 있다. 개발 과정에서 문제에 직면했을 때, 공식 문서와 더불어 이 이슈 트래커를 검색하는 것은 매우 효과적인 문제 해결 전략이 될 수 있다. 비슷한 문제를 겪은 다른 개발자의 경험은 시간과 노력을 크게 절약해 줄 수 있는 귀중한 자산이다.


1. EGO-Planner: An ESDF-free Gradient-based Local Planner for Quadrotors - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/343786586_EGO-Planner_An_ESDF-free_Gradient-based_Local_Planner_for_Quadrotors
2. (PDF) EGO-Planner: An ESDF-free Gradient-based Local Planner ..., accessed August 11, 2025, https://www.researchgate.net/publication/348028324_EGO-Planner_An_ESDF-free_Gradient-based_Local_Planner_for_Quadrotors
3. EGO-Planner: An ESDF-Free Gradient-Based Local Planner for Quadrotors, accessed August 11, 2025, https://www.semanticscholar.org/paper/EGO-Planner%3A-An-ESDF-Free-Gradient-Based-Local-for-Zhou-Wang/6fd8ff3bfee9ba2b57eea07780da868964a47486
4. EGO-Planner: An ESDF-Free Gradient-Based Local Planner for Quadrotors. IEEE Robotics and Automation Letters, 6(2), 478–485 | 10.1109/LRA.2020.3047728 - Sci-Hub, accessed August 11, 2025, https://sci-hub.se/10.1109/LRA.2020.3047728
5. ZJU-FAST-Lab/ego-planner - GitHub, accessed August 11, 2025, https://github.com/ZJU-FAST-Lab/ego-planner
6. EGO-Planner: An ESDF-free Gradient-based Local Planner for Quadrotors - YouTube, accessed August 11, 2025, https://www.youtube.com/watch?v=UKoaGW7t7Dk
7. ZJU-FAST-Lab/ego-planner-swarm: An efficient single/multi-agent trajectory planner for multicopters. - GitHub, accessed August 11, 2025, https://github.com/ZJU-FAST-Lab/ego-planner-swarm
8. ZJU-FAST-Lab/EGO-Planner-v2: Swarm Playground, the codebase of the paper "Swarm of micro flying robots in the wild" - GitHub, accessed August 11, 2025, https://github.com/ZJU-FAST-Lab/EGO-Planner-v2
9. A DRONE SYSTEM FOR AUTONOMOUS MAPPING FLIGHTS INSIDE A FOREST, accessed August 11, 2025, https://isprs-archives.copernicus.org/articles/XLVIII-1-W2-2023/597/2023/isprs-archives-XLVIII-1-W2-2023-597-2023.pdf
10. [2011.04183] EGO-Swarm: A Fully Autonomous and Decentralized Quadrotor Swarm System in Cluttered Environments - arXiv, accessed August 11, 2025, https://arxiv.org/abs/2011.04183
11. Jetson Orin NX 开发指南（7）: EGO-Swarm 的编译与运行原创 - CSDN博客, accessed August 11, 2025, https://blog.csdn.net/qq_44998513/article/details/133791656
12. Forrest-Z/ego-planner-for-ground-robot - GitHub, accessed August 11, 2025, https://github.com/Forrest-Z/ego-planner-for-ground-robot
13. Companion Computers - Dev documentation - ArduPilot, accessed August 11, 2025, https://ardupilot.org/dev/docs/companion-computers.html
14. Top 5 Companion Computers for UAVs | ModalAI, Inc., accessed August 11, 2025, https://www.modalai.com/blogs/blog/top-5-companion-computers-for-uavs
15. A small VIO system using raspberry pi and arducam ov9281 - ArduPilot Discourse, accessed August 11, 2025, https://discuss.ardupilot.org/t/a-small-vio-system-using-raspberry-pi-and-arducam-ov9281/103299
16. Companion Computers | PX4 Guide (main), accessed August 11, 2025, https://docs.px4.io/main/en/companion_computer/
17. Raspberry Pi Companion with Pixhawk | PX4 Guide (main), accessed August 11, 2025, https://docs.px4.io/main/en/companion_computer/pixhawk_rpi
18. PX4-Devguide/en/companion_computer/pixhawk_companion.md at master - GitHub, accessed August 11, 2025, https://github.com/PX4/PX4-Devguide/blob/master/en/companion_computer/pixhawk_companion.md
19. Companion Computer - Wiki, accessed August 11, 2025, https://wiki.hanzheteng.com/quadrotor/companion-computer
20. ROS Package: mavros, accessed August 11, 2025, https://index.ros.org/p/mavros/
21. How to Setup Development Environment using ROS 2 (Jazzy), Mavros, and Gazebo, accessed August 11, 2025, https://satrya.zeroinside.id/blog/how-to-setup-drone-development-environment-using-ros2-mavros-and-gazebo
22. mavros - ROS Wiki, accessed August 11, 2025, http://wiki.ros.org/mavros
23. Sending custom mavros msg to GCS from companion computer. / Issue #632 - GitHub, accessed August 11, 2025, https://github.com/mavlink/mavros/issues/632
24. Issues / ZJU-FAST-Lab/ego-planner-swarm - GitHub, accessed August 11, 2025, https://github.com/ZJU-FAST-Lab/ego-planner-swarm/issues
25. Issues / ZJU-FAST-Lab/EGO-Planner-v2 - GitHub, accessed August 11, 2025, https://github.com/ZJU-FAST-Lab/EGO-Planner-v2/issues
26. [2402.06559] Diffusion-ES: Gradient-free Planning with Diffusion for Autonomous Driving and Zero-Shot Instruction Following - arXiv, accessed August 11, 2025, https://arxiv.org/abs/2402.06559
27. EgoPlan-Bench: Benchmarking Multimodal Large Language Models for Human-Level Planning, accessed August 11, 2025, https://chenyi99.github.io/ego_plan/
28. Automated Generation of Test Scenarios for Autonomous Driving Using LLMs - MDPI, accessed August 11, 2025, https://www.mdpi.com/2079-9292/14/16/3177
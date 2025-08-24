# 드론 자율 비행을 위한 FASTER 경로 계획 알고리즘에 대한 심층 고찰 및 구현 방안 연구
[경로 계획 (Path Planning)](./index.md)



무인 항공기(Unmanned Aerial Vehicle, UAV), 즉 드론의 자율 비행 기술은 현대 로보틱스 분야에서 가장 역동적이고 도전적인 연구 분야 중 하나로 자리매김했다. 특히, 사전에 알려지지 않은 복잡하고 동적인 환경에서 인간의 개입 없이 임무를 수행하는 능력은 드론의 활용 범위를 수색 및 구조, 정찰, 물류 배송, 시설 점검 등 다양한 산업으로 확장하는 핵심 기술이다.1 이러한 자율 비행의 성공은 효과적인 경로 계획(Path Planning) 알고리즘에 달려있다.

자율 비행에서의 경로 계획은 단순히 출발점(A)에서 목표점(B)까지의 기하학적 경로를 찾는 정적인 문제를 넘어선다. 드론은 제한된 센서 정보를 바탕으로 주변 환경을 실시간으로 인식하고, 끊임없이 변화하는 상황에 맞춰 연속적인 의사결정을 내려야 한다.1 이 과정은 "다음 순간 어디로 이동해야 하는가?", "어떤 영역을 우선적으로 탐색하여 지도를 확장할 것인가?", "언제 임무를 중단하거나 경로를 수정해야 하는가?"와 같은 동적인 질문들에 대한 해답을 실시간으로 찾아가는 과정이다.1

이러한 의사결정 기반 경로 계획(Decision-making-based path planning)은 본질적으로 다음과 같은 세 가지 주요 도전 과제로 인해 계산적으로나 개념적으로 매우 어려운 문제로 간주된다 1:

1. **환경의 동적 불확실성 (Dynamic and Uncertain Nature of the Environment):** 환경 내 장애물의 위치가 변하거나, 이전에 없던 장애물이 나타나는 등 환경 정보가 불완전하고 계속해서 변한다.
2. **센서 및 통신 제약 (Sensor and Communication Limitations):** 드론에 탑재된 센서(예: 깊이 카메라, LiDAR)는 탐지 거리, 시야각, 측정 오차 등의 한계를 가지며, 지상국이나 다른 에이전트와의 통신은 지연되거나 두절될 수 있다.
3. **고차원 의사결정 공간 (High-dimensional Decision Space):** 불확실성 하에서의 계획은 환경을 모델링하고, 미래를 예측하며, 수많은 가능한 행동들 중에서 최적의 선택을 해야 하는 고차원의 복잡한 최적화 문제를 포함한다.

이러한 문제들은 드론이 단순히 '실행 가능한(feasible)' 경로를 찾는 것을 넘어, 주어진 임무 목표(예: 정보 획득 극대화, 비행시간 최소화)를 최적화하는 '정보 수집 경로 계획(Informative Path Planning, IPP)'이나 '탐사 경로 계획(Exploratory Path Planning, EPP)'과 같은 고급 패러다임으로 발전하도록 이끌었다.1 결국, 고속 자율 비행의 핵심은 불완전한 정보 속에서 안전을 보장하면서도 빠르고 효율적인 의사결정을 내리는 능력에 있다.


미지 환경에서의 자율 비행을 위해 개발된 기존의 경로 계획 알고리즘들은 크게 샘플링 기반(sampling-based) 기법과 최적화 기반(optimization-based) 기법으로 나눌 수 있다.1 A*, RRT(Rapidly-exploring Random Tree)와 같은 알고리즘들은 알려진 장애물 정보를 바탕으로 충돌 없는 경로를 계산하는 데 효과적이다.3 하지만 이들 전통적인 접근법은 '속도'와 '안전'이라는 두 가지 상충하는 목표 사이에서 근본적인 트레이드오프(trade-off) 문제에 직면한다.

안전을 보장하기 위한 가장 보편적이고 직관적인 전략은 드론이 현재까지 탐색하여 안전하다고 알려진 공간, 즉 '알려진 안전 공간(Free-known space, F)' 내에서만 궤적을 계획하는 것이다.5 더 나아가, 만일의 사태에 대비해 생성된 궤적의 끝 지점에는 항상 '정지(stop)' 조건을 강제하여 드론이 예측 불가능한 상황에 직면했을 때 그 자리에서 멈출 수 있도록 설계한다.5 이러한 접근법은 이론적으로 안전을 보장할 수는 있지만, 드론의 행동을 극도로 보수적(conservative)으로 만드는 치명적인 단점을 가진다.5

이 문제점은 드론이 넓은 미지의 영역(Unknown space, U)을 탐사해야 하는 시나리오에서 특히 두드러진다. 예를 들어, 탐사 초기 단계나 복잡한 미로와 같은 환경에서는 알려진 안전 공간 `F`가 매우 작고, 대부분의 공간이 미지 공간 `U` 또는 장애물 공간 `O`로 채워져 있다.5 이런 상황에서 '안전 우선' 전략을 따르는 드론은 미지 공간으로 진입하는 것을 주저하게 된다. 마치 사람이 어두운 방에 들어갈 때 한 걸음 내딛고 주변을 살핀 후 다시 한 걸음을 내딛는 것처럼, 드론은 짧은 거리만 전진하고 정지하여 주변을 관측한 뒤 다시 짧게 전진하는 과정을 반복한다. 이로 인해 드론의 비행 속도는 현저히 저하되며, 고속 기동성이 요구되는 긴급 수색이나 빠른 정찰 임무에는 부적합하게 된다.8

또한, 안전 공간을 볼록 다면체(convex polyhedra)의 합으로 표현하고 각 궤적 구간을 특정 다면체에 할당하는 최적화 기법들 역시, 시간을 배분하는 방식이 임시방편(ad-hoc)적이거나 경직되어 있어 생성되는 궤적이 최적의 속도를 내지 못하는 경우가 많았다.7 결국, 기존 기법들은 안전을 확보하는 대가로 속도를 희생하는 구조적 한계를 명확히 보여주었다.


이러한 속도와 안전의 근본적인 트레이드오프를 극복하기 위해 MIT 항공우주 제어 연구실(Aerospace Controls Lab)의 Jesus Tordesillas 등에 의해 제안된 알고리즘이 바로 **FASTER (Fast and Safe Trajectory Planner)** 이다.5 FASTER의 이름이 암시하듯, 이 알고리즘의 핵심 철학은 '안전을 희생하지 않으면서도(Safe) 빠른(Fast) 궤적을 생성하는 것'이다.

FASTER는 기존 접근법의 패러다임을 전환하는 혁신적인 아이디어를 제시한다. 바로 지역 플래너(local planner)가 알려진 안전 공간 `F`뿐만 아니라 미지의 공간 `U`까지 포함하는 더 넓은 영역(`F` U `U`)에서 궤적을 최적화하도록 허용하는 것이다.5 이는 드론이 미지의 영역으로 과감하게 진입하는 '공격적인(aggressive)' 궤적을 생성할 수 있게 하여, 기존의 보수적인 플래너들과 비교할 수 없는 높은 비행 속도를 가능하게 한다.

그러나 이러한 공격적인 계획은 자칫 안전을 위협할 수 있다. FASTER는 이 문제를 해결하기 위해 독창적인 이중 궤적(dual-trajectory) 안전 보장 메커니즘을 도입했다.5 매 리플래닝(replanning) 단계마다, FASTER는 두 가지 종류의 궤적을 동시에 계산한다.

1. **전체 궤적 (Whole Trajectory):** `F` U `U` 공간에서 목표점을 향해 계획되는 빠르고 긴 궤적. 드론은 실제로 이 궤적을 따라 비행한다.
2. **안전 백업 궤적 (Safe Back-up Trajectory):** '전체 궤적' 상의 한 지점에서 시작하여, 오직 알려진 안전 공간 `F` 내에서만 존재하며 안전하게 정지할 수 있는 비상 탈출로와 같은 궤적이다.

핵심은 드론이 '전체 궤적'을 따라 비행하는 동안, 항상 실행 가능한 '안전 백업 궤적'의 존재가 수학적으로 보장된다는 점이다.5 만약 미지의 공간에서 예상치 못한 장애물이 나타나거나 계획이 틀어질 경우, 드론은 즉시 이 미리 계산된 안전 백업 궤적으로 전환하여 충돌 없이 

`F` 공간 내에서 안전하게 정지할 수 있다. 이로써 FASTER는 미지의 공간을 향한 고속 비행이라는 '공격성'과 어떠한 상황에서도 안전을 보장하는 '방어성'을 동시에 달성한다. 이 독창적인 접근법은 고속 자율 비행 분야에서 중요한 이정표를 세웠으며, 이후 많은 연구에 영감을 주었다.7


FASTER 알고리즘의 혁신성은 속도와 안전의 조화를 이루는 정교한 수학적 모델링에 기반한다. 이를 이해하기 위해서는 계층적 플래닝 구조, 궤적 표현 방식, 그리고 핵심인 혼합 정수 이차 계획법(MIQP) 최적화 및 안전 보장 메커니즘을 순차적으로 살펴볼 필요가 있다.


FASTER는 효율적인 계산을 위해 계층적 플래닝(Hierarchical Planning) 아키텍처를 채택한다.5 이는 복잡한 경로 계획 문제를 두 개의 하위 문제로 분해하여 처리하는 전략이다.

- **전역 플래너 (Global Planner):** 전역 플래너의 역할은 장기적인 관점에서 시작점에서 목표점까지의 대략적인 경로를 제시하는 것이다. 이 플래너는 현재까지 구축된 맵 정보를 바탕으로 알려진 장애물(`O`, Occupied-known space)을 피해가는 거시적인 경로를 계산한다. FASTER의 원 논문에서는 이 전역 플래너로 **Jump Point Search (JPS)** 알고리즘을 사용했다.5 JPS는 균일 비용 격자(uniform-cost grid) 맵에서 A* 알고리즘의 성능을 크게 개선한 탐색 기법으로, 불필요한 노드 탐색을 건너뛰어 매우 빠른 속도로 경로를 찾아내는 장점이 있다.12 이 전역 경로는 후술할 지역 플래너에게 일종의 '가이드라인' 역할을 한다.
- **지역 플래너 (Local Planner):** 지역 플래너는 FASTER의 핵심 두뇌에 해당한다. 전역 플래너가 제시한 가이드라인을 따라, 드론의 동역학적 제약(kinodynamic constraints)을 만족하는 단기적이고 부드러운 실제 비행 궤적을 생성한다. 이 과정에서 센서를 통해 새로 들어온 환경 정보를 반영하여 실시간으로 궤적을 재계획(replan)한다. FASTER는 이 지역 궤적을 생성하기 위해 **혼합 정수 이차 계획법(Mixed-Integer Quadratic Program, MIQP)** 이라는 강력한 최적화 도구를 사용하며, '전체 궤적'과 '안전 백업 궤적'을 계산하는 주체도 바로 이 지역 플래너다.8

이러한 계층적 구조는 장기적인 목표 지향성(전역 플래너)과 단기적인 환경 변화 대응 및 동역학적 실행 가능성(지역 플래너)을 효과적으로 분리하고 결합하여, 전체 시스템의 계산 효율성과 반응성을 높이는 데 기여한다.


지역 플래너가 생성하는 궤적은 드론이 실제로 따라가야 하는 비행 경로이므로, 수학적으로 다루기 쉽고 드론의 움직임을 잘 표현할 수 있는 형태로 정의되어야 한다. FASTER는 이를 위해 **조각별 3차 다항식(Piecewise Cubic Polynomials)** 을 사용한다.

전체 궤적은 $N$개의 짧은 구간(interval)으로 나뉘며, 각 구간에서의 궤적은 3차 다항식으로 표현된다.9 이는 각 구간에서 저크(jerk, 가속도의 변화율, 즉 가가속도)가 일정하다고 가정하는 것과 수학적으로 동일하다. 3차원 공간에서 $n$번째 구간의 시간 $\tau \in$에 대한 위치 벡터 $\mathbf{p}_n(\tau)$는 다음과 같이 표현된다.
$$
\mathbf{p}_n(\tau) = \mathbf{a}_n \tau^3 + \mathbf{b}_n \tau^2 + \mathbf{c}_n \tau + \mathbf{d}_n
$$
여기서 $\mathbf{a}_n, \mathbf{b}_n, \mathbf{c}_n, \mathbf{d}_n$은 각 축(x, y, z)에 대한 3차, 2차, 1차, 0차 항의 계수 벡터이다. 이 표현 방식은 궤적의 연속성과 미분 가능성을 보장하여 속도, 가속도, 저크를 쉽게 계산할 수 있게 해준다.

여기서 FASTER의 중요한 설계적 통찰이 드러난다. 3차 다항식으로 표현된 궤적은 **3차 베지어 곡선(Cubic Bézier Curve)** 으로 정확하게 변환될 수 있다는 점이다.9 베지어 곡선은 컴퓨터 그래픽스와 기하학 모델링에서 널리 사용되는 곡선 표현 방식으로, 몇 개의 제어점(control points)에 의해 곡선의 형태가 결정된다. 3차 베지어 곡선은 4개의 제어점 $\mathbf{r}_{n0}, \mathbf{r}_{n1}, \mathbf{r}_{n2}, \mathbf{r}_{n3}$으로 정의된다.

FASTER가 베지어 곡선 표현을 도입한 이유는 베지어 곡선이 가진 강력한 속성인 **볼록 포위(Convex Hull) 특성**을 활용하기 위함이다. 이 특성에 따르면, 베지어 곡선은 항상 그 곡선을 정의하는 제어점들을 모두 포함하는 가장 작은 볼록 다각형(convex polygon) 내부에 존재한다. 이 속 덕분에, 궤적 전체가 특정 안전 영역(예: 볼록 다면체) 내에 존재하는지 확인하는 복잡한 문제를, 단 4개의 제어점이 그 영역 내에 존재하는지 확인하는 훨씬 간단한 문제로 치환할 수 있다. 이는 최적화 문제의 제약 조건을 크게 단순화하여 계산 효율성을 높이는 결정적인 역할을 한다.

위에서 정의한 $n$번째 구간의 3차 다항식 계수 $(\mathbf{a}_n, \mathbf{b}_n, \mathbf{c}_n, \mathbf{d}_n)$와 그에 해당하는 베지어 제어점 $(\mathbf{r}_{n0}, \mathbf{r}_{n1}, \mathbf{r}_{n2}, \mathbf{r}_{n3})$ 사이의 관계는 다음과 같이 유도된다.8
$$
\begin{aligned}
\mathbf{r}_{n0} &= \mathbf{d}_n \\
\mathbf{r}_{n1} &= \frac{1}{3} \mathbf{c}_n \Delta t + \mathbf{d}_n \\
\mathbf{r}_{n2} &= \frac{1}{3} \mathbf{b}_n (\Delta t)^2 + \frac{2}{3} \mathbf{c}_n \Delta t + \mathbf{d}_n \\
\mathbf{r}_{n3} &= \mathbf{a}_n (\Delta t)^3 + \mathbf{b}_n (\Delta t)^2 + \mathbf{c}_n \Delta t + \mathbf{d}_n
\end{aligned}
$$
이 관계식을 통해 최적화 문제의 결정 변수(주로 저크 또는 다항식 계수)와 안전 제약 조건에 사용될 기하학적 요소(제어점)가 수학적으로 연결된다.


FASTER 지역 플래너의 심장부는 혼합 정수 이차 계획법(MIQP) 최적화 문제이다.6 MIQP는 목적 함수(최소화하려는 값)가 결정 변수들의 이차식(Quadratic)으로 표현되고, 제약 조건 중에 일부 변수가 정수(Integer) 값을 가져야 하는, 매우 표현력이 높지만 풀기 어려운 최적화 문제 유형이다. FASTER는 이 MIQP 프레임워크를 통해 속도, 안전, 동역학적 제약을 모두 만족하는 최적의 궤적을 찾아낸다.

FASTER가 매 리플래닝 주기마다 푸는 MIQP 문제는 다음과 같이 공식화될 수 있다.8

- 목적 함수 (Objective Function):

  FASTER의 목적은 가능한 한 부드러운 궤적을 생성하는 것이다. 궤적의 부드러움은 물리적으로 기체에 가해지는 힘의 변화율, 즉 저크(jerk)의 크기로 측정할 수 있다. 저크를 최소화하면 드론의 움직임이 매끄러워지고 에너지 소모를 줄이며, 기체의 기계적 스트레스를 완화하는 효과가 있다. 따라서 목적 함수는 전체 궤적에 대한 저크의 제곱의 적분값(또는 이산화된 합)을 최소화하는 것으로 정의된다.
  $$
  \min_{\mathbf{j}_n, b_{np}} \sum_{n=0}^{N-1} \|\mathbf{j}_n\|^2 \Delta t
  $$
  여기서 $\mathbf{j}_n$은 $n$번째 궤적 구간의 일정한 저크 벡터, $b_{np}$는 후술할 이진 결정 변수, 그리고 $\Delta t$는 각 구간에 할당된 시간이다. 이 목적 함수는 결정 변수(저크)에 대한 이차식(Quadratic) 형태를 띤다.

- **제약 조건 (Constraints):**

  1. **동역학적 제약 (Kinodynamic Constraints):** 드론은 물리적 한계를 가진다. 따라서 생성된 궤적은 드론의 최대 속도($v_{max}$), 최대 가속도($a_{max}$), 최대 저크($j_{max}$)를 초과해서는 안 된다. 이러한 제약은 궤적의 모든 지점에서 만족되어야 한다.
     $$
     \begin{aligned}
     \|\mathbf{v}_n(\tau)\|_\infty &\le v_{max} \\
     \|\mathbf{a}_n(\tau)\|_\infty &\le a_{max} \\
     \|\mathbf{j}_n\|_\infty &\le j_{max}
     \end{aligned}
     \quad \forall n, \forall \tau \in
     $$

  2. **연속성 제약 (Continuity Constraints):** 조각별 다항식으로 이루어진 전체 궤적이 물리적으로 의미를 가지려면 각 구간의 경계에서 부드럽게 연결되어야 한다. 즉, $n$번째 구간이 끝나는 지점의 위치, 속도, 가속도는 $n+1$번째 구간이 시작하는 지점의 그것과 동일해야 한다.

     Code snippet

     ```
     $$
     \mathbf{p}_{n+1}(0) = \mathbf{p}_n(\Delta t), \quad \mathbf{v}_{n+1}(0) = \mathbf{v}_n(\Delta t), \quad \mathbf{a}_{n+1}(0) = \mathbf{a}_n(\Delta t) \quad \forall n \in [0, N-2]
     $$
     ```

  3. 안전 회랑 제약 (Safe Corridor Constraints): 이 부분이 MIQP를 사용하는 가장 핵심적인 이유이다. 궤적이 장애물과 충돌하지 않도록 보장하기 위해, FASTER는 알려진 안전 공간 F를 여러 개의 겹치는 볼록 다면체(convex polyhedra)들의 합집합으로 표현한다. 각 다면체 $P_p$는 선형 부등식의 집합, 즉 $A_p \mathbf{x} \le \mathbf{c}_p$ 형태로 정의된다.

     문제는 궤적의 어떤 구간이 어떤 다면체 안에 존재해야 하는지를 미리 알 수 없다는 것이다. FASTER는 이 '할당' 문제 자체를 최적화의 일부로 만든다. 이를 위해 이진 정수 변수(binary integer variable) $b_{np}$를 도입한다.
     $$
     b_{np} = \begin{cases} 1 & \text{if trajectory segment } n \text{ is in polyhedron } p \\ 0 & \text{otherwise} \end{cases}
     $$
     그리고 2.2절에서 설명한 베지어 곡선의 볼록 포위 특성을 이용하여, 궤적 전체가 아닌 4개의 제어점($\mathbf{r}_{nj}, j \in \{0,1,2,3\}$)만 다면체 내에 있도록 제약한다. 이 논리적 조건("만약 $b_{np}=1$이면, 제어점들은 다면체 $p$ 안에 있어야 한다")은 Big-M 기법과 같은 표준적인 방법을 통해 선형 부등식으로 변환되어 MIQP 솔버가 처리할 수 있게 된다.
     $$
     b_{np} = 1 \implies A_p \mathbf{r}_{nj} \le \mathbf{c}_p \quad \forall j \in \{0,1,2,3\}, \forall n, \forall p
     $$
     또한, 각 궤적 구간은 반드시 하나 이상의 다면체에 속해야 하므로 다음 제약이 추가된다.
     $$
     \sum_{p=0}^{P-1} b_{np} \ge 1 \quad \forall n
     $$
     이러한 접근법은 기존의 많은 최적화 기반 플래너들이 각 궤적 구간을 특정 다면체에 미리 할당하고(ad-hoc allocation) 최적화를 수행했던 방식과 근본적인 차이를 보인다.8 FASTER는 MIQP 솔버가 목적 함수를 최소화하는 과정에서 각 구간이 어느 다면체에 속할지를 

     **스스로 결정**하도록 함으로써, 훨씬 더 유연하고 최적에 가까운 해를 탐색할 수 있는 자유도를 부여한다. 이것이 FASTER가 더 빠르고 덜 보수적인 궤적을 생성할 수 있는 핵심적인 이유 중 하나이다.


FASTER가 미지의 공간으로 공격적인 탐색을 하면서도 안전을 보장할 수 있는 비결은 바로 '안전 백업 궤적(Safe Back-up Trajectory)'이라는 독창적인 개념에 있다. FASTER의 지역 플래너는 매 리플래닝 주기마다 하나의 궤적이 아닌, 목적이 다른 두 개의 궤적을 동시에 계획하고 항상 유효한 안전책을 확보한다.5

1. **전체 궤적 (Whole Trajectory):** 이 궤적은 드론의 현재 위치에서 시작하여 전역 경로 상의 먼 목표점을 향한다. 이 궤적을 최적화할 때는 알려진 안전 공간 `F`와 미지 공간 `U`를 모두 통과할 수 있도록 허용된다 (`F` U `U`). 따라서 이 궤적은 미지의 영역을 가로지르는 길고 빠른, 즉 '공격적인' 형태를 띤다. 드론은 실제로 이 '전체 궤적'의 시작 부분을 따라 비행을 시작한다. 이 궤적은 속도(speed) 요구사항을 만족시키는 역할을 한다.8
2. **안전 궤적 (Safe Trajectory):** 이 궤적은 일종의 비상 계획이다. '전체 궤적' 상의 특정 지점(예: 현재 위치에서 일정 시간 후의 지점)에서 시작하여, **오직 알려진 안전 공간 `F` 내에서만** 존재하며, 최종적으로는 속도와 가속도가 0이 되는 '정지(stop)' 상태로 끝난다. 이 궤적은 전역 경로를 따를 필요가 없으며, 가장 안전하게 멈출 수 있는 방향으로 생성된다.

이 이중 궤적 시스템의 작동 원리는 모델 예측 제어(Model Predictive Control, MPC)의 재귀적 실행 가능성(recursive feasibility) 개념과 유사하다.5 드론은 더 빠르고 효율적인 '전체 궤적'을 따라 비행하지만, 매 순간마다 "만약 지금 당장 문제가 생긴다면, 이 '안전 궤적'을 따라 안전하게 멈출 수 있다"는 사실이 수학적으로 보장된다. 즉, '안전 궤적'의 존재 자체가 안전을 증명하는 역할을 한다.5

센서를 통해 새로운 환경 정보가 들어와 맵이 업데이트되면, FASTER는 즉시 새로운 '전체 궤적'과 그에 상응하는 '안전 궤적'을 다시 계산한다. 이 과정이 매우 빠른 주기로 반복되면서 드론은 미지의 환경을 동적으로 탐색해 나간다. 만약 재계획 과정에서 유효한 '안전 궤적'을 찾지 못하면, 시스템은 즉시 비상 정지 절차에 돌입하여 안전을 확보한다. 이처럼 FASTER는 '공격적인 탐색'과 '증명 가능한 안전'이라는 두 마리 토끼를 잡기 위해 정교하게 설계된 프레임워크를 제공한다.


FASTER 알고리즘을 실제 드론에 성공적으로 구현하기 위해서는 하드웨어와 소프트웨어 스택을 올바르게 구성하고, 비행 컨트롤러와의 연동을 정밀하게 설정하는 것이 매우 중요하다. 본 장에서는 시뮬레이션 및 실제 비행을 위한 시스템 아키텍처와 단계별 설정 절차를 상세히 기술한다.


FASTER 시스템은 크게 온보드 컴퓨터(OBC), 센서 시스템, 그리고 비행 컨트롤러(FC)의 세 가지 핵심 하드웨어 구성요소로 이루어진다.

- 온보드 컴퓨터 (On-board Computer, OBC):

  FASTER 시스템의 두뇌 역할을 하는 가장 중요한 하드웨어이다. OBC는 운영체제(Linux), ROS, 그리고 FASTER의 모든 계산 집약적인 알고리즘(매핑, MIQP 최적화 등)을 실시간으로 실행한다. 드론에 탑재되어야 하므로 크기(Size), 무게(Weight), 전력 소모(Power)를 의미하는 SWaP가 매우 중요한 고려사항이다.13

  - **요구사항:** FASTER의 핵심인 MIQP 최적화는 복잡한 수학적 연산을 포함하므로 강력한 **CPU 성능**이 필수적이다.14 동시에, 깊이 카메라로부터 들어오는 데이터를 처리하여 3D 맵을 생성하고, 잠재적으로 머신러닝 기반의 인식(perception) 알고리즘을 구동하기 위해서는 

    **GPU 가속** 능력이 매우 유리하다.16

  - **권장 모델:** 이러한 요구사항 때문에, 강력한 멀티코어 ARM CPU와 내장 NVIDIA GPU(CUDA 코어 및 텐서 코어 포함)를 하나의 소형 모듈에 통합한 **NVIDIA Jetson 시리즈**가 업계 표준처럼 널리 사용된다.18 특히 

    **Jetson Xavier NX**나 최신 **Jetson Orin 시리즈(Nano, NX)**는 뛰어난 AI 연산 성능과 전력 효율성을 제공하여 FASTER와 같은 고성능 자율 비행 알고리즘 구동에 매우 적합하다.20 대안으로 

    **Intel NUC**와 같은 초소형 폼팩터 PC도 고려될 수 있으나, 일반적으로 Jetson 플랫폼에 비해 SWaP 효율성이나 통합된 GPU 성능 면에서 불리할 수 있다.18

  다음 표는 FASTER 구동을 위한 OBC의 최소 및 권장 사양을 정리한 것이다.

  **표 3-1: FASTER 구동을 위한 온보드 컴퓨터 최소 및 권장 사양**

| 항목       | 최소 사양 (시뮬레이션/기초 연구용)                     | 권장 사양 (실시간 고속 비행용)                            | 근거 및 설명                                                 |
| ---------- | ------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| **CPU**    | 4코어 이상 (예: Intel Core i5, ARM Cortex-A7x)         | 6코어 이상 고성능 (예: Intel Core i7, NVIDIA Orin ARM)    | MIQP 솔버 및 다중 ROS 노드 동시 실행을 위해 다중 코어 및 높은 클럭 속도가 필수적임.14 |
| **GPU**    | 내장 그래픽 또는 저전력 외장 GPU (예: NVIDIA GTX 1650) | NVIDIA Jetson Xavier/Orin 내장 GPU 또는 모바일 RTX 시리즈 | 3D 맵핑(깊이 이미지 처리), SLAM, 잠재적 AI 인식 작업의 실시간 처리를 위해 CUDA 지원 GPU가 강력히 권장됨.16 |
| **RAM**    | 8 GB                                                   | 16 GB 이상                                                | OS, ROS, Gazebo 시뮬레이터, FASTER의 여러 노드들을 동시에 실행하려면 충분한 메모리 용량이 필요함.14 |
| **저장소** | 256 GB SSD                                             | 512 GB 이상 NVMe SSD                                      | 빠른 부팅, 맵 데이터 로딩, 비행 로그 기록을 위해 HDD보다 월등히 빠른 SSD, 특히 NVMe SSD가 필수적임.16 |

- **센서 시스템 (Sensor System):**

  - **깊이 카메라 (Depth Camera):** 주변 환경의 3차원 구조를 실시간으로 인지하여 장애물 지도(occupancy map)를 생성하는 데 사용되는 핵심 센서이다. FASTER는 이 정보를 바탕으로 안전 회랑을 계산한다. **Intel RealSense 시리즈(D435, D455 등)**나 **Stereolabs ZED 시리즈**와 같이 ROS 드라이버가 잘 지원되는 카메라가 주로 사용된다.27
  - **관성 측정 장치 (IMU):** 가속도계와 자이로스코프로 구성되어 드론의 자세(attitude)와 각속도를 측정한다. 이는 비행 안정화와 상태 추정(state estimation)의 기본이 된다. 일반적으로 비행 컨트롤러에 내장된 고품질의 IMU를 사용한다.28

- 비행 컨트롤러 (Flight Controller, FC):

  FC는 OBC로부터 상위 레벨의 궤적 명령을 받아, 이를 실제 모터의 RPM 신호로 변환하여 드론의 자세를 안정시키고 저수준 제어를 수행하는 역할을 한다. 오픈소스 오토파일럿 펌웨어인 PX4와 이를 구동하는 Pixhawk 시리즈 하드웨어가 사실상의 표준으로 널리 사용된다.28 OBC와 FC는 보통 UART나 USB를 통해 연결된다.


FASTER는 특정 버전의 운영체제와 ROS, 그리고 여러 핵심 라이브러리에 의존하여 구동된다.

- **운영체제 및 ROS:** FASTER의 공식 GitHub 저장소에 따르면, **Ubuntu 16.04 (ROS Kinetic)** 와 **Ubuntu 18.04 (ROS Melodic)** 환경에서 테스트 및 검증되었다.27 최신 ROS 1 버전인 Noetic이나 ROS 2로 포팅하기 위해서는 일부 코드 수정이 필요할 수 있다.27
- **핵심 의존성 라이브러리:**
  - **Gurobi Optimizer:** FASTER가 사용하는 MIQP 문제를 풀기 위한 고성능 상용 최적화 솔버이다. FASTER는 **Gurobi 8.1, 9.0, 9.1** 버전에서 테스트되었다.27 Gurobi는 학술적 목적의 사용자에게는 무료 라이선스를 제공하지만, 상업적 활용을 위해서는 유료 라이선스가 필요하다는 점을 인지해야 한다.
  - **ROS 패키지:** FASTER 시스템은 여러 ROS 노드들의 집합으로 구성되므로, 다양한 ROS 패키지들이 필요하다. `wstool`과 `catkin`은 빌드 시스템을 위해, `gazebo-ros-pkgs`는 시뮬레이션을 위해, `mavros-msgs`는 PX4와의 통신을 위해, `tf2-sensor-msgs`는 좌표 변환을 위해 필요하다.27
  - **Python 라이브러리:** `pyquaternion`과 같은 라이브러리가 자세 표현 및 계산을 위해 사용된다.27


FASTER와 같은 고수준 플래너를 드론에 통합할 때, OBC와 FC 간의 통신 및 제어권 핸드오버는 매우 중요한 부분이다.

- **연동 방식의 변화:** 과거 PX4는 외부 플래너 연동을 위해 `Path Planning Interface`라는 MAVLink 기반 인터페이스를 제공했다. 이 인터페이스는 `TRAJECTORY_REPRESENTATION_WAYPOINTS`나 `TRAJECTORY_REPRESENTATION_BEZIER`와 같은 메시지를 사용하여 외부에서 계산된 궤적을 PX4에 전달하는 방식이었다.29 그러나 이 방식은 아키텍처의 경직성과 확장성 문제로 인해 

  **PX4 펌웨어 v1.15부터 공식적으로 지원 중단(deprecated)되었으며, 더 이상 사용이 권장되지 않는다**.31

- 현대적 연동 방식: 오프보드 모드 (Offboard Mode):

  현재 FASTER를 포함한 대부분의 외부 플래너는 PX4의 오프보드 모드를 통해 연동된다. 오프보드 모드는 PX4가 외부 컴퓨터(OBC)로부터 위치, 속도, 가속도, 자세 등의 설정값(setpoint)을 직접 받아 기체를 제어하도록 하는 강력한 기능이다.32

  - **통신 프로토콜:** OBC의 ROS 환경과 FC의 PX4 펌웨어 간의 통신은 **MAVLink** 프로토콜을 통해 이루어진다. ROS 사용자는 **MAVROS**라는 ROS 패키지를 사용하여 MAVLink 메시지를 ROS 토픽으로 쉽게 변환하고 통신할 수 있다.27 ROS 2 환경에서는 MAVROS 대신 uXRCE-DDS 클라이언트-에이전트 구조를 통해 PX4의 내부 uORB 토픽에 직접 접근하는 방식이 주로 사용된다.34 FASTER의 공식 코드는 MAVROS를 사용하는 ROS 1 기반으로 작성되었다.

  - **생존 신호 (Heartbeat)의 중요성:** PX4는 안전을 위해 오프보드 제어의 유효성을 지속적으로 확인한다. 오프보드 모드를 활성화하고 유지하기 위해서는 OBC가 PX4로 **최소 2Hz 이상의 빈도**로 특정 설정값 메시지(MAVROS의 경우) 또는 `OffboardControlMode` 메시지(ROS 2의 경우)를 계속해서 보내야 한다.32 이 '생존 신호'가 0.5초 이상(기본값) 끊기면, PX4는 외부 제어 시스템에 문제가 발생했다고 판단하고 자동으로 안전 모드(예: 

    `Hold` 모드)로 전환한다.30 따라서 FASTER의 인터페이스 노드는 계산된 궤적 설정값을 이 빈도에 맞춰 꾸준히 발행해야 한다.

- 주요 PX4 파라미터 설정:

  QGroundControl과 같은 지상 관제 시스템(GCS)을 사용하여 다음 파라미터들을 설정해야 한다.

  - `COM_OBL_RC_ACT`: 오프보드 제어 신호가 손실되었을 때(Offboard Loss) 드론이 취할 비상 행동을 정의한다. 'Position Control', 'Return to Launch (RTL)', 'Land' 등의 옵션이 있다.32
  - `COM_OF_LOSS_T`: 오프보드 신호가 끊긴 후 위에서 설정한 비상 행동을 실행하기까지 대기하는 시간(초)이다.32
  - `COM_RC_IN_MODE`: 비행 중 조종기(RC)의 스틱을 움직였을 때, 오프보드 모드를 즉시 해제하고 수동 제어 모드로 전환할지를 결정한다. 이는 비상 상황 시 조종사가 즉시 제어권을 가져올 수 있도록 하는 매우 중요한 안전 기능이다.32
  - `MPC_*` 파라미터 군: `MPC_XY_VEL_MAX` (최대 수평 속도), `MPC_Z_VEL_MAX` (최대 수직 속도), `MPC_ACC_HOR_MAX` (최대 수평 가속도) 등 PX4의 내부 위치 제어기(MPC) 파라미터들을 FASTER의 `faster.yaml` 파일에 설정된 동역학적 제약 조건과 최대한 일치시켜야 궤적 추종 성능을 극대화할 수 있다.


FASTER 공식 GitHub 저장소 27에 기반한 설정 절차는 다음과 같다.

1. **의존성 설치:** 터미널을 열고 `apt-get`과 `pip`을 사용하여 Gurobi 및 ROS 관련 의존성들을 모두 설치한다. Gurobi는 공식 웹사이트의 가이드에 따라 설치하고 라이선스를 설정해야 한다.
2. **`catkin` 워크스페이스 생성 및 소스 코드 클론:** `mkdir -p ~/ws/src`와 같이 워크스페이스를 생성하고, 해당 `src` 디렉토리에서 `git clone` 명령어로 FASTER 저장소를 복제한다. 그 후 `wstool`을 사용하여 `.rosinstall` 파일에 명시된 추가적인 의존성 패키지들을 다운로드한다.
3. **코드 빌드:** 워크스페이스의 최상위 디렉토리로 이동하여 `catkin config -DCMAKE_BUILD_TYPE=Release` 명령어로 릴리즈 모드 빌드를 설정하고, `catkin build`로 전체 코드를 컴파일한다. 컴파일이 완료되면 `source ~/ws/devel/setup.bash`를 `.bashrc` 파일에 추가하여 환경 변수를 설정한다.
4. **시뮬레이션 실행 (Gazebo):**
   - 총 5개의 터미널이 필요하다. 각 터미널에서 `roslaunch` 명령어를 사용하여 다음 노드들을 순차적으로 실행한다.
     1. `acl_sim start_world.launch`: Gazebo 시뮬레이션 월드를 실행한다.
     2. `acl_sim perfect_tracker_and_sim.launch`: 시뮬레이션된 드론과 완벽한 추종기(perfect tracker)를 실행한다.
     3. `global_mapper_ros global_mapper_node.launch`: 환경 맵을 생성하는 매퍼 노드를 실행한다.
     4. `faster faster_interface.launch`: FASTER 플래너와 시뮬레이션/실제 드론 간의 인터페이스 노드를 실행한다.
     5. `faster faster.launch`: FASTER 플래너의 핵심 로직을 실행한다.
   - 모든 노드가 실행되면, `rviz`를 실행하여 시각화 결과를 확인한다. Rviz 화면에는 주황색(점유 공간), 파란색(미지 공간)으로 표시된 그리드 맵과 드론 모델이 나타난다. 상단의 '2D Nav Goal' 툴을 사용하여 맵 상의 임의의 지점을 클릭하면, FASTER가 해당 지점까지의 궤적을 계산하고 드론이 비행을 시작한다.27
5. **실제 비행 설정:**
   - **파라미터 튜닝:** `faster/params/faster.yaml` 파일을 열어 `drone_radius`, `v_max`, `a_max` 등 실제 사용하는 드론의 물리적 사양과 성능에 맞게 파라미터를 정밀하게 수정한다.
   - **카메라 연동:** 실제 깊이 카메라의 ROS 드라이버(예: `realsense-ros`의 `rs_camera.launch`)를 실행한다. `rostopic list`로 깊이 이미지와 카메라 정보가 발행되는 토픽 이름을 확인하고, `global_mapper_node.launch` 파일 내의 해당 토픽 이름을 실제 토픽 이름으로 수정한다.
   - **상태 추정 연동:** 실제 드론의 위치와 자세를 추정하는 시스템(예: VIO, 모션 캡처 시스템)의 ROS 노드를 실행한다. 이 노드가 발행하는 드론의 상태 정보(주로 `nav_msgs/Odometry` 또는 `geometry_msgs/PoseStamped` 형태)를 FASTER의 인터페이스 노드가 구독하여 `State` 메시지 타입으로 변환 후 플래너에 전달하도록 설정해야 한다.27
   - **제어 명령 전달:** FASTER가 발행하는 궤적 명령(`Goal` 메시지)을 MAVROS를 통해 PX4의 오프보드 설정값 토픽(예: `/mavros/setpoint_raw/local`)으로 전달하는 브릿지 노드를 작성하거나 기존 인터페이스를 활용한다.

이러한 절차를 통해 FASTER는 시뮬레이션 환경에서 검증된 후 실제 드론 플랫폼으로 이식될 수 있다.


알고리즘의 우수성은 이론적 정교함뿐만 아니라, 실제 문제 상황에서의 성능으로 증명된다. FASTER는 기존 알고리즘들의 한계를 극복하기 위해 제안된 만큼, 다른 최신 경로 계획 기법들과의 정량적, 정성적 비교 평가는 그 가치를 이해하는 데 필수적이다. 본 장에서는 주요 성능 지표를 정의하고, 대표적인 알고리즘인 RRT* 및 EGO-Planner와의 비교를 통해 FASTER의 특성과 장단점을 심층 분석한다.


드론 경로 계획 알고리즘의 성능을 객관적으로 평가하기 위해 다음과 같은 지표들이 널리 사용된다.4

- **계산 시간 (Computation Time):** 경로 계획 요청이 들어온 순간부터 실행 가능한 궤적이 생성되기까지 걸리는 시간이다. 단위는 보통 밀리초(ms)를 사용한다. 이 지표는 알고리즘의 실시간 반응성을 직접적으로 나타내며, 동적 환경이나 고속 비행 시나리오에서 가장 중요한 요소 중 하나이다. 계산 시간이 짧을수록 더 높은 빈도로 재계획(replanning)이 가능하여 변화하는 환경에 빠르게 대응할 수 있다.4
- **경로 길이/비용 (Path Length/Cost):** 생성된 궤적의 총 기하학적 길이(m) 또는 궤적의 비용 함수 값(예: 저크의 총합)이다. 경로 길이는 비행 시간 및 에너지 소모와 직결되므로 효율성의 척도가 된다. 일반적으로 더 짧은 경로가 선호되지만, 안전성이나 평활도를 위해 다소 길어지는 것을 감수해야 할 수도 있다.4
- **성공률 (Success Rate):** 주어진 임무(예: 특정 시간 내에 목표 지점 도달)를 충돌이나 실패 없이 완수하는 비율이다. 이는 알고리즘의 강건성(robustness)과 신뢰도를 평가하는 핵심 지표이다.38
- **비행 속도 (Flight Speed):** 실제 비행에서 달성한 평균 속도 및 최대 속도(m/s)이다. 이는 알고리즘이 얼마나 공격적이고 빠른 궤적을 생성할 수 있는지를 보여준다. FASTER와 같은 고속 플래너는 이 지표에서 높은 성능을 보이는 것을 목표로 한다.7
- **경로 평활도 (Path Smoothness):** 생성된 궤적의 곡률(curvature)이나 고차 미분값(저크, 스냅)의 크기로 평가된다. 경로가 부드러울수록 드론의 급격한 기동이 줄어들어 승차감(payload 안정성), 에너지 효율, 제어 안정성이 향상된다.40


FASTER와 RRT*는 각각 최적화 기반 및 샘플링 기반 접근법을 대표하는 알고리즘으로, 근본적인 철학의 차이가 성능 특성으로 이어진다.

- RRT* (Rapidly-exploring Random Tree Star):

  RRT*는 무작위 샘플링을 통해 상태 공간을 탐색하며 점진적으로 경로 트리를 확장하는 알고리즘이다.41 핵심적인 특징은 **점근적 최적성(asymptotic optimality)**이다.42 이는 충분한 시간과 샘플이 주어지면 RRT*가 생성하는 경로의 비용이 점차 감소하여 결국 최적 해에 수렴함을 의미한다. 하지만 이러한 장점은 단점이 되기도 한다. 초기 해를 빠르게 찾을 수는 있지만, 최적 해에 가까워지기까지의 수렴 속도가 매우 느릴 수 있다.43 특히, 복잡한 환경이나 좁은 통로(narrow passage)가 있는 환경에서는 무작위 샘플링의 비효율성으로 인해 해를 찾지 못하거나 오랜 시간이 걸릴 수 있다.42 RRT*는 본질적으로 기하학적 '경로(path)'를 생성하며, 이를 드론이 따라갈 수 있는 동역학적 '궤적(trajectory)'으로 변환하는 후처리 과정이 별도로 필요하다.45

- FASTER:

  FASTER는 주어진 제약 조건 하에서 비용 함수(저크 최소화)를 직접 최적화하여 동역학적으로 실행 가능한 '궤적(trajectory)'을 한 번에 생성한다.45 MIQP라는 강력한 최적화 프레임워크를 사용하므로, 생성된 궤적은 주어진 정보 내에서는 **지역적으로 최적(locally optimal)**이다. FASTER는 점근적 최적성을 보장하지는 않지만, 대신 매우 빠른 계산 시간 내에 고품질의 부드러운 궤적을 생성하는 데 초점을 맞춘다. 실제 비행에서는 환경이 계속 변하기 때문에, 한 번에 완벽한 전역 최적 해를 찾는 것보다, 변화에 맞춰 빠르게 고품질의 지역 최적 해를 반복적으로 재계획하는 것이 더 효과적일 수 있다.4

결론적으로, RRT*가 '탐험(exploration)'을 통해 점진적으로 전역 최적 해를 찾아가는 학술적 이상에 가깝다면, FASTER는 '착취(exploitation)'를 통해 현재 가용한 정보로 최상의 실시간 해를 도출하는 공학적 현실에 더 가깝다. 정적이고 넓은 환경에서는 RRT*의 최적성이 빛을 발할 수 있지만, 장애물이 많고 동적으로 변하는 미지의 환경에서는 FASTER의 빠른 재계획 능력과 동역학적 궤적 생성 능력이 더 큰 실용적 가치를 가진다.


FASTER와 EGO-Planner는 모두 최적화 기반의 고속 지역 플래너라는 공통점을 가지지만, 안전성과 계산 효율성 사이의 미묘한 설계 철학 차이를 보여주는 좋은 예이다.

- EGO-Planner (ESDF-free Gradient-based Local Planner):

  EGO-Planner는 이름에서 알 수 있듯이, ESDF(Euclidean Signed Distance Field) 맵을 사용하지 않는 것이 가장 큰 특징이다.46 ESDF는 맵 상의 모든 지점에서 가장 가까운 장애물까지의 유클리드 거리를 저장하는 맵으로, 궤적과 장애물 간의 충돌 비용 및 그 구배(gradient)를 계산하는 데 매우 유용하다. 하지만 ESDF를 생성하고 실시간으로 유지하는 데에는 상당한 계산 자원이 소모된다. EGO-Planner는 이 ESDF 생성 단계를 과감히 생략하고, 원시 점유 격자 지도(occupancy grid map)로부터 직접 충돌 비용을 계산하는 방식을 택했다. 최적화 방법으로는 B-스플라인으로 표현된 궤적의 제어점을 경사 하강법(gradient-based method)을 이용해 이동시키는 방식을 사용한다.47 이러한 설계 덕분에 EGO-Planner는 **약 1ms**라는 극도로 짧은 계획 시간을 달성할 수 있다.46

- FASTER:

  반면, FASTER는 '증명 가능한 안전(provable safety)'을 최우선 가치 중 하나로 둔다. 이를 위해 알려진 안전 공간을 볼록 다면체로 명시적으로 표현하고, MIQP의 정수 제약($b_{np} \in \{0, 1\}$)을 통해 궤적이 이 안전 회랑 내에 존재함을 **수학적으로 보장(hard constraint)**한다. 이 방식은 경사 하강법처럼 지역 최솟값(local minima)에 빠질 위험이 적고 안전을 확실히 보장하지만, MIQP 문제 자체의 계산 복잡도로 인해 EGO-Planner만큼의 극단적인 속도를 내기는 어렵다. FASTER의 계산 시간은 수십~수백 ms 범위로 보고된다.7

이 두 알고리즘의 비교는 경로 계획 분야의 중요한 연구 동향을 시사한다. 즉, '절대적인 최적성'이나 '수학적으로 증명된 안전성'을 다소 완화하더라도, '극단적인 계산 효율성'을 확보하여 더 높은 주파수(frequency)로 재계획을 수행하는 것이 동적 환경에서 더 강건한 성능을 보일 수 있다는 관점이다.3 EGO-Planner는 이러한 철학을 극명하게 보여주는 사례이며, FASTER는 안전 보장과 속도 사이에서 다른 균형점을 선택한 알고리즘이라 할 수 있다.

다음 표는 세 가지 대표적인 알고리즘의 특성을 정량적으로 비교한 것이다.

**표 4-1: 주요 경로 계획 알고리즘 정량적 성능 비교**

| 구분               | RRT*                           | FASTER                           | EGO-Planner                          |
| ------------------ | ------------------------------ | -------------------------------- | ------------------------------------ |
| **접근 방식**      | 샘플링 기반 (Sampling-based)   | 최적화 기반 (Optimization-based) | 최적화 기반 (Optimization-based)     |
| **궤적 표현**      | 점들의 연결 (Path)             | 조각별 3차 다항식 (Trajectory)   | B-스플라인 (Trajectory)              |
| **안전 보장**      | 점근적 보장                    | MIQP 하드 제약, 안전 백업 궤적   | 충돌 비용 최소화 (소프트 제약)       |
| **평균 계산 시간** | 수백 ~ 수천 ms 4               | 수십 ~ 수백 ms 7                 | ~1 ms 46                             |
| **장점**           | 전역 최적성 보장, 구현 용이    | 증명 가능한 안전성, 고품질 궤적  | 극도로 빠른 계산 속도, ESDF 불필요   |
| **단점**           | 느린 수렴 속도, 좁은 통로 취약 | MIQP 계산 비용, 상용 솔버 의존   | 증명 가능한 안전성 부재, 지역 최솟값 |


- **복잡한 장애물 환경 (Cluttered Environments):** FASTER와 EGO-Planner와 같은 최적화 기반 방법들은 장애물 사이의 공간을 효율적으로 활용하는 경향이 있다. FASTER는 볼록 분해를 통해, EGO-Planner는 충돌 비용 구배를 통해 장애물로부터 안전거리를 유지하면서도 부드러운 경로를 찾아낸다.3 RRT*는 샘플링 운에 따라 비효율적인 경로를 생성할 수 있다.
- **협소 공간 통과 (Narrow Passages):** 좁은 통로를 통과하는 것은 RRT*에게 특히 어려운 과제이다. 무작위 샘플이 좁은 공간에 들어갈 확률이 낮기 때문이다.42 반면, FASTER의 MIQP는 최적화 과정에서 좁은 통로를 통과하는 것이 전체 비용을 줄이는 해라면, 해당 해를 적극적으로 탐색하고 찾아낼 수 있다. EGO-Planner 역시 구배 정보를 이용해 좁은 틈새로 궤적을 밀어 넣을 수 있어, 두 최적화 기반 플래너가 이러한 시나리오에서 더 강건한 성능을 보일 가능성이 높다.


FASTER는 고속 자율 비행 분야에서 중요한 돌파구를 마련했지만, 동시에 몇 가지 명확한 한계점을 가지고 있으며 이는 향후 연구를 위한 중요한 방향을 제시한다. 본 장에서는 FASTER의 핵심적인 한계점들을 분석하고, 이를 극복하기 위한 미래 연구 방향을 논한다.


- 현재 한계:

  FASTER의 핵심 수학적 모델은 기본적으로 환경이 정적(static)이거나, 적어도 지역 플래너가 재계획을 수행하는 짧은 시간 동안에는 변하지 않는다고 가정한다.5 즉, 움직이는 보행자나 다른 차량과 같은 **동적 장애물(dynamic obstacles)**을 명시적으로 고려하는 메커니즘이 내장되어 있지 않다.49 현재의 FASTER 구현으로 동적 장애물에 대응하는 유일한 방법은, 움직이는 장애물을 그 순간의 정적인 장애물로 간주하고 매우 높은 빈도로 재계획을 수행하는 것이다. 그러나 이 접근법은 장애물의 미래 움직임을 예측하지 못하기 때문에 근본적으로 반응적(reactive)일 뿐이며, 장애물이 빠르게 접근하는 경우 충돌을 피하지 못할 수 있는 차선책에 불과하다.51

- 향후 연구 방향:

  이 한계를 극복하기 위해 FASTER를 동적 환경에 맞게 확장하는 연구가 활발히 진행될 수 있다.

  1. **예측 모델 통합 (Integration of Prediction Models):** 칼만 필터(Kalman Filter), LSTM(Long Short-Term Memory)과 같은 시계열 예측 모델을 사용하여 동적 장애물의 미래 궤적을 확률적으로 예측한다.50 이렇게 예측된 미래의 시공간적 점유 영역(spatio-temporal occupancy)을 FASTER의 MIQP 최적화 문제에 새로운 제약 조건으로 추가하는 방식이다. 이를 통해 단순히 현재 위치를 피하는 것을 넘어, 미래의 충돌 가능성까지 미리 회피하는 선제적(proactive) 계획이 가능해진다.54
  2. **위험 인식 계획 (Risk-Aware Planning):** 궤적과 장애물 간의 충돌 확률을 정량적으로 계산하고, 이를 목적 함수에 비용 항으로 추가하는 접근법이다. 이 경우 플래너는 단순히 충돌을 회피하는 이진적인 결정을 넘어, 충돌 '위험' 자체를 최소화하는 더 정교하고 안전한 궤적을 생성하게 된다.50
  3. **다중 가설 궤적 (Multi-Hypothesis Trajectories):** 동적 장애물의 미래 행동에는 여러 가능성(예: 좌회전, 우회전, 직진)이 존재한다. 이러한 불확실성에 강건하게 대응하기 위해, 각 가능성에 대응하는 여러 개의 회피 궤도를 병렬적으로 최적화하고, 실시간으로 관측되는 장애물의 움직임에 따라 가장 가능성이 높고 안전한 궤도를 선택하여 실행하는 프레임워크를 개발할 수 있다.55


- 현재 한계:

  FASTER의 강력한 표현력의 원천인 MIQP는 NP-hard 문제로 알려져 있다.56 이는 문제의 크기, 즉 궤적 구간의 수, 고려해야 할 안전 다면체의 수, 또는 바이너리 변수의 수가 증가함에 따라 최적 해를 찾는 데 필요한 계산 시간이 기하급수적으로 증가할 수 있음을 의미한다. 이로 인해 매우 복잡한 환경에서는 실시간성을 보장하기 어려울 수 있다. 또한, FASTER의 성능은 

  **Gurobi**와 같은 고성능 상용 솔버에 크게 의존하는데, 이는 연구 및 개발 단계에서는 문제가 없지만 상업용 제품에 적용할 때 라이선스 비용과 의존성 문제를 야기한다.27

- **향후 연구 방향:**

  1. **오픈소스 솔버의 활용 및 개발:** **HiGHS**, **OSQP**, **Drake**의 수학적 최적화 도구 모음 등 최근 성능이 크게 향상된 오픈소스 QP/MIQP 솔버들을 FASTER에 통합하고 성능을 벤치마킹하는 연구가 필요하다.57 이는 FASTER의 접근성과 활용성을 크게 높일 수 있다.
  2. **문제 완화 (Problem Relaxation):** MIQP의 계산 복잡도를 낮추기 위해 정수 제약을 완화(relax)하여 연속적인 QP(Quadratic Program) 문제로 근사하여 푸는 기법을 연구할 수 있다. 이 경우 최적성은 일부 손상될 수 있지만, 계산 속도를 수십 배 이상 향상시킬 수 있어 실시간성이 더 중요한 응용에 적합하다.
  3. **하드웨어 가속 (Hardware Acceleration):** MIQP 문제 해결 과정에서 병렬화가 가능한 특정 연산 부분(예: 행렬 연산)을 **FPGA(Field-Programmable Gate Array)**나 맞춤형 반도체(ASIC)를 사용하여 가속하는 연구는 장기적으로 온보드 컴퓨터의 제약을 극복하는 유망한 방향이다.60


- 현재 한계:

  FASTER는 순수한 모델 기반(model-based) 접근법으로, 사전에 정의된 수학적 모델과 제약 조건에 따라 결정론적으로 작동한다. 이는 예측 가능하고 해석이 용이하다는 장점이 있지만, 경험으로부터 학습하여 성능을 개선하는 능력이 없다.

- 향후 연구 방향:

  모델 기반 계획의 견고함과 학습 기반 기법의 적응성을 결합하는 하이브리드 접근법은 매우 유망하다.

  1. **강화학습(RL) 기반 안내 (Reinforcement Learning Guided Planning):** DRL(Deep Reinforcement Learning) 에이전트를 학습시켜, FASTER의 지역 플래너(MIQP)에 더 좋은 초기 해(warm start)를 제공하도록 할 수 있다. 좋은 초기 해는 MIQP 솔버의 수렴 속도를 크게 향상시킬 수 있다.61 더 나아가, DRL 에이전트가 전역 플래너(JPS)를 대체하여, 환경의 의미론적 정보(semantic information)까지 고려한 더 지능적인 장기 경로를 제안하도록 할 수도 있다.3
  2. **모방학습(IL) 활용 (Imitation Learning for Trajectory Generation):** 인간 전문가 조종사의 비행 데이터를 모방 학습(Imitation Learning)하여, 특정 상황(예: 매우 좁은 틈새를 통과하는 곡예 비행)에서 더 자연스럽고 효율적인 궤적 스타일을 생성하도록 FASTER를 유도할 수 있다. 이는 알고리즘만으로는 생성하기 어려운 미묘하고 직관적인 기동을 가능하게 할 수 있다.3
  3. **신경망 기반 동역학 모델 (Neural Network-based Dynamics Model):** 드론의 실제 동역학은 단순한 미분 방정식으로 표현하기 어려운 복잡한 공기역학적 효과(예: 지면 효과, 와류)를 포함한다. 이러한 비선형적이고 불확실한 동역학을 신경망으로 정밀하게 모델링하고, 이를 MIQP의 제약 조건에 통합하면 궤적 생성과 실제 추종 간의 오차(sim-to-real gap)를 줄여 더 정밀한 제어를 달성할 수 있다.64


- 다개체 시스템으로의 확장 (Extension to Multi-Agent Systems):

  FASTER는 단일 드론을 대상으로 설계되었다. 이를 드론 군집(swarm)이 서로 충돌하지 않고 협력적으로 임무를 수행하도록 확장하는 연구가 필요하다. 각 드론이 다른 드론들을 동적 장애물로 간주하고 FASTER를 적용하는 분산형 접근법이 가능하다. 실제로 FASTER의 후속 연구인 MADER는 이러한 다개체 동적 환경에서의 궤적 계획 문제를 다루고 있다.10

- 인식-계획 통합 (Tighter Integration of Perception and Planning):

  현재 FASTER는 '인식(Perception)' 단계에서 생성된 맵을 입력으로 받아 '계획(Planning)'을 수행하는 분리된 구조를 가진다. 차세대 플래너는 이 둘을 더 긴밀하게 통합해야 한다. 예를 들어, 단순히 현재 보이는 맵을 기반으로 최단 경로를 찾는 것을 넘어, 미래에 더 많은 정보를 얻을 수 있는 방향(예: 미지 공간의 경계)으로 궤적을 능동적으로 계획하는 '인식-인지(Perception-Aware)' 플래닝으로 발전해야 한다. 이는 불확실성을 최소화하고 탐사 효율을 극대화하는 데 필수적이다.1


본 보고서는 미지 환경에서의 고속 자율 비행을 위한 핵심적인 경로 계획 알고리즘인 **FASTER (Fast and Safe Trajectory Planner)**에 대한 심층적인 고찰을 제공했다. FASTER는 기존 경로 계획 기법들이 직면했던 '속도'와 '안전' 사이의 근본적인 트레이드오프 문제를 해결하기 위해 등장했으며, 그 핵심 기여는 **안전을 수학적으로 보장하면서도 미지의 영역으로 공격적인 탐색을 가능하게 한 혁신적인 프레임워크**를 제시한 데 있다.

FASTER의 이론적 견고함은 두 가지 핵심 아이디어에서 비롯된다. 첫째, **이중 궤적(dual-trajectory) 시스템**이다. 드론이 실제로 따라가는 빠르고 공격적인 '전체 궤적'과 함께, 어떠한 순간에도 안전하게 정지할 수 있음을 보장하는 '안전 백업 궤적'을 항상 확보함으로써 속도와 안전을 동시에 달성했다.5 둘째, **혼합 정수 이차 계획법(MIQP)의 독창적인 활용**이다. 궤적의 각 구간이 어느 안전 회랑에 속할지를 최적화 과정에서 솔버가 직접 결정하도록 함으로써, 기존의 경직된 할당 방식에서 벗어나 훨씬 더 유연하고 효율적인 궤적을 생성할 수 있었다.6 이러한 이론적 기반은 조각별 3차 다항식 궤적 표현과 계층적 플래닝 아키텍처를 통해 구체화되었다.

실제 구현의 관점에서 FASTER는 ROS(Robot Operating System)를 기반으로 모듈화되어 있어, 시뮬레이션 및 실제 드론으로의 이식성이 비교적 높다.27 그러나 강력한 연산 능력을 갖춘 온보드 컴퓨터(예: NVIDIA Jetson 시리즈)와 고성능 상용 MIQP 솔버(Gurobi)에 대한 의존성은 실제 시스템 구축 시 고려해야 할 중요한 제약 조건이다.16 또한, 최신 PX4 펌웨어와의 연동은 더 이상 지원되지 않는 구형 인터페이스가 아닌, **오프보드 모드(Offboard Mode)**를 통해 MAVROS나 ROS 2-PX4 브릿지를 사용하는 현대적인 방식으로 접근해야 한다는 점을 명확히 인지해야 한다.31

그럼에도 불구하고 FASTER는 명확한 한계를 가진다. 첫째, 원본 알고리즘은 **동적 장애물**에 대한 명시적인 처리 능력이 부재하여, 예측 모델과의 통합이 향후 중요한 연구 과제로 남아있다.49 둘째, MIQP의 **계산 복잡도**는 복잡한 시나리오에서의 실시간성을 위협할 수 있으며, 이는 오픈소스 솔버의 활용이나 문제 완화, 하드웨어 가속 등의 연구를 통해 해결해야 할 과제이다.56

결론적으로, FASTER는 드론 자율 비행 기술의 패러다임을 한 단계 발전시킨 기념비적인 알고리즘이다. 이는 모델 기반 최적화가 어떻게 복잡한 제약 조건 하에서 강건하고 효율적인 해를 제공할 수 있는지를 명확히 보여주었다. 비록 동적 환경 대응과 계산 비용이라는 과제가 남아있지만, FASTER가 제시한 '증명 가능한 안전을 동반한 고속 기동'이라는 철학은 EGO-Planner와 같은 후속 연구에 큰 영향을 미쳤으며, 예측 및 학습 기반 기법과의 융합을 통해 미래의 지능형 자율 항법 시스템으로 발전해 나갈 무한한 가능성을 품고 있다. FASTER에 대한 깊이 있는 이해는 현재와 미래의 고속 자율 비행 시스템을 연구하고 개발하는 모든 이들에게 필수적인 자산이 될 것이다.


1. Decision-Making-Based Path Planning for Autonomous UAVs: A Survey - arXiv, accessed August 14, 2025, https://arxiv.org/html/2508.09304v1
2. Towards Fully Autonomous UAVs: A Survey - Semantic Scholar, accessed August 14, 2025, https://pdfs.semanticscholar.org/7b53/b45f4ec61d43bb2d08660362b15e28a3a846.pdf
3. FM-Planner: Foundation Model Guided Path Planning for Autonomous Drone Navigation, accessed August 14, 2025, https://arxiv.org/html/2505.20783v1
4. Comparison between A* and RRT Algorithms for UAV Path Planning - TU Delft Research Portal, accessed August 14, 2025, https://research.tudelft.nl/files/40690538/Comparison_between_Astar_and_RRT_Algorithms_for_UAV_Path_Planning.pdf
5. FASTER: Fast and Safe Trajectory Planner for Flights in Unknown Environments - DSpace@MIT, accessed August 14, 2025, https://dspace.mit.edu/bitstream/handle/1721.1/137157/1903.03558.pdf?sequence=2&isAllowed=y
6. FASTER: Fast and Safe Trajectory Planner for Flights in Unknown Environments | Request PDF - ResearchGate, accessed August 14, 2025, https://www.researchgate.net/publication/338686573_FASTER_Fast_and_Safe_Trajectory_Planner_for_Flights_in_Unknown_Environments
7. FASTER: Fast and Safe Trajectory Planner for Navigation in Unknown Environments | Request PDF - ResearchGate, accessed August 14, 2025, https://www.researchgate.net/publication/355724804_FASTER_Fast_and_Safe_Trajectory_Planner_for_Navigation_in_Unknown_Environments
8. MIT Open Access Articles FASTER: Fast and Safe Trajectory Planner for Navigation in Unknown Environments, accessed August 14, 2025, https://dspace.mit.edu/bitstream/handle/1721.1/145374/2001.04420.pdf?sequence=2&isAllowed=y
9. FASTER: Fast and Safe Trajectory Planner for Flights in Unknown Environments, accessed August 14, 2025, https://www.researchgate.net/publication/331644791_FASTER_Fast_and_Safe_Trajectory_Planner_for_Flights_in_Unknown_Environments
10. ‪Jesus Tordesillas‬ - ‪Google Scholar‬, accessed August 14, 2025, https://scholar.google.com/citations?user=QKcwqsQAAAAJ&hl=en
11. FASTER: Fast and Safe Trajectory Planner for Navigation in Unknown Environments, accessed August 14, 2025, https://www.researchgate.net/publication/338570042_FASTER_Fast_and_Safe_Trajectory_Planner_for_Navigation_in_Unknown_Environments
12. Quadrotor Path Planning and Polynomial Trajectory Generation Using Quadratic Programming for Indoor Environments - MDPI, accessed August 14, 2025, https://www.mdpi.com/2504-446X/7/2/122
13. Low-SWaP AI Mission Computer | NVIDIA Orin NX | FLYC-300 - Neousys Technology, accessed August 14, 2025, https://www.neousys-tech.com/en/product/product-lines/edge-ai-gpu-computing/nvidia-jetson/flyc-300
14. What are the Minimum Hardware Requirements for Artificial Intelligence?, accessed August 14, 2025, https://www.moontechnolabs.com/qanda/minimum-hardware-requirements-for-artificial-intelligence/
15. Infrastructure: Machine Learning Hardware Requirements - C3 AI, accessed August 14, 2025, https://c3.ai/introduction-what-is-machine-learning/machine-learning-hardware-requirements/
16. how-to-choose-a-computer-for-ai-and-machine-learning-work - ASUS, accessed August 14, 2025, https://www.asus.com/us/content/how-to-choose-a-computer-for-ai-and-machine-learning-work/
17. Hardware Requirements for Machine Learning - GeeksforGeeks, accessed August 14, 2025, https://www.geeksforgeeks.org/machine-learning/hardware-requirements-for-machine-learning/
18. Vision-Based Learning for Drones: A Survey - arXiv, accessed August 14, 2025, https://arxiv.org/html/2312.05019v2
19. Top 5 Companion Computers for UAVs | ModalAI, Inc., accessed August 14, 2025, https://www.modalai.com/blogs/blog/top-5-companion-computers-for-uavs
20. Orin Nano 8GB or Xavier NX 16GB for a Fighter UAV: ? : r/JetsonNano - Reddit, accessed August 14, 2025, https://www.reddit.com/r/JetsonNano/comments/1it4pft/orin_nano_8gb_or_xavier_nx_16gb_for_a_fighter_uav/
21. NVIDIA® Jetson™ Comparison of Modules - Forecr.io, accessed August 14, 2025, https://www.forecr.io/blogs/all/comparison-of-nvidia-jetson-modules
22. NVIDIA Jetson Nano, NVIDIA Jetson TX2 NX, NVIDIA Jetson Xavier NX | Mistral Blog - Mistral Solutions, accessed August 14, 2025, https://www.mistralsolutions.com/blog/nvidia-jetson-modules-use-cases-part-2/
23. CPU Speed Explained: What's a Good Processor Speed? | HP® Tech Takes, accessed August 14, 2025, https://www.hp.com/us-en/shop/tech-takes/what-is-processor-speed
24. PC specs for beginners? : r/UAVmapping - Reddit, accessed August 14, 2025, https://www.reddit.com/r/UAVmapping/comments/yxnf55/pc_specs_for_beginners/
25. Computer Specification Recommendations - Mid-State Technical College, accessed August 14, 2025, https://www.mstc.edu/technology/computer-specification-recommendations
26. How Much Memory & Graphics Card Do I Need for a Fast Desktop Computer? - Lenovo, accessed August 14, 2025, https://www.lenovo.com/us/en/glossary/how-to-build-fast-desktop-computer/
27. mit-acl/faster: 3D Trajectory Planner in Unknown Environments - GitHub, accessed August 14, 2025, https://github.com/mit-acl/faster
28. Recent Advances in Unmanned Aerial Vehicles: A Review - PMC - PubMed Central, accessed August 14, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC9035982/
29. Path Planning Interface | PX4 User Guide, accessed August 14, 2025, https://hamishwillee.github.io/px4_vuepress/en/computer_vision/path_planning_interface.html
30. Path Planning Interface / PX4 User Guide, accessed August 14, 2025, https://docs.px4.io/v1.11/en/computer_vision/path_planning_interface.html
31. Path Planning Interface | PX4 Guide (main) - PX4 docs, accessed August 14, 2025, https://docs.px4.io/main/en/computer_vision/path_planning_interface.html
32. Offboard Mode - PX4 User Guide - GitBook, accessed August 14, 2025, https://px4.gitbook.io/px4-user-guide/flying/flight_modes/offboard
33. mzahana/mavros_trajectory_tracking: Accurate trajectory generation and tracking with interface to PX4 autopilot - GitHub, accessed August 14, 2025, https://github.com/mzahana/mavros_trajectory_tracking
34. Simulation | PX4 Guide (main), accessed August 14, 2025, https://docs.px4.io/main/en/simulation/
35. ROS 2 Offboard Control Example | PX4 User Guide (v1.14), accessed August 14, 2025, https://docs.px4.io/v1.14/en/ros/ros2_offboard_control.html
36. ROS 2 Offboard Control Example | PX4 Guide (main), accessed August 14, 2025, https://docs.px4.io/main/en/ros2/offboard_control
37. UAV Formation Trajectory Planning Algorithms: A Review - MDPI, accessed August 14, 2025, https://www.mdpi.com/2504-446X/7/1/62
38. Real-time 3D UAV Path Planning in Dynamic Environments with Uncertainty | Unmanned Systems - World Scientific Publishing, accessed August 14, 2025, https://www.worldscientific.com/doi/10.1142/S2301385023500073
39. Speed-First: An Aggressive Gradient-Based Local Planner for Quadrotor Faster Flight, accessed August 14, 2025, https://www.mdpi.com/2504-446X/7/3/192
40. A Trajectory Generation Method for Multi-Rotor UAV Based on Adaptive Adjustment Strategy, accessed August 14, 2025, https://www.mdpi.com/2076-3417/13/6/3435
41. A Comparison of RRT, RRT* and RRT*-Smart Path Planning Algorithms | Semantic Scholar, accessed August 14, 2025, https://www.semanticscholar.org/paper/A-Comparison-of-RRT%2C-RRT*-and-RRT*-Smart-Path-Noreen-Khan/a06c3e978e3cb8da7f94079745142520ca930796
42. Fast-RRT: A RRT-Based Optimal Path Finding Method - MDPI, accessed August 14, 2025, https://www.mdpi.com/2076-3417/11/24/11777
43. A Path-Planning Performance Comparison of RRT*-AB with MEA* in a 2-Dimensional Environment - MDPI, accessed August 14, 2025, https://www.mdpi.com/2073-8994/11/7/945
44. An Overview and Comparison of Traditional Motion Planning Based on Rapidly Exploring Random Trees - MDPI, accessed August 14, 2025, https://www.mdpi.com/1424-8220/25/7/2067
45. Difference between trajectory optimization and motion planning and when to use either (both as defined on wikipedia) : r/robotics - Reddit, accessed August 14, 2025, https://www.reddit.com/r/robotics/comments/mcbo0w/difference_between_trajectory_optimization_and/
46. ZJU-FAST-Lab/ego-planner - GitHub, accessed August 14, 2025, https://github.com/ZJU-FAST-Lab/ego-planner
47. Comparison of planned velocity and actual velocity. - ResearchGate, accessed August 14, 2025, https://www.researchgate.net/figure/Comparison-of-planned-velocity-and-actual-velocity_fig15_352869555
48. Approximation Algorithms for the UAV Path Planning with Object Coverage Constraints, accessed August 14, 2025, https://arxiv.org/html/2504.08405v1
49. Comparison of Pathfinding Algorithms in Dynamic and Congested Environments - DiVA, accessed August 14, 2025, https://kth.diva-portal.org/smash/get/diva2:1985736/FULLTEXT01.pdf
50. Real-time Motion Planning with Dynamic Obstacles - Computer Science, accessed August 14, 2025, https://www.cs.unh.edu/~ruml/papers/f-biased-socs-12.pdf
51. Limitation of the obstacle-avoidance algorithm. - ResearchGate, accessed August 14, 2025, https://www.researchgate.net/figure/Limitation-of-the-obstacle-avoidance-algorithm_fig21_50284114
52. Path Planning in Dynamic Environments with Adaptive Dimensionality - Carnegie Mellon University Robotics Institute, accessed August 14, 2025, https://www.ri.cmu.edu/pub_files/2016/7/13951-61058-1-PB.pdf
53. Future-Oriented Navigation: Dynamic Obstacle Avoidance with One-Shot Energy-Based Multimodal Motion Prediction - arXiv, accessed August 14, 2025, https://arxiv.org/html/2505.00237v3
54. ITOMP: Incremental Trajectory Optimization for Real-time Replanning in Dynamic Environments - GAMMA, accessed August 14, 2025, https://gamma.cs.unc.edu/ITOMP/itomp.pdf
55. Topology-Driven Parallel Trajectory Optimization in Dynamic Environments - arXiv, accessed August 14, 2025, https://arxiv.org/html/2401.06021v2
56. [1311.4752] Efficient evaluation of mp-MIQP solutions using lifting - arXiv, accessed August 14, 2025, https://arxiv.org/abs/1311.4752
57. HiGHS - High-performance parallel linear optimization software, accessed August 14, 2025, https://highs.dev/
58. OpEn / Fast and Accurate Nonconvex Optimization - Pantelis Sopasakis, accessed August 14, 2025, https://alphaville.github.io/optimization-engine/
59. Formulating and Solving Optimization Problems - Drake, accessed August 14, 2025, https://drake.mit.edu/doxygen_cxx/group__solvers.html
60. 09 Hardware Acceleration in ROS 2 Faster ROS withFPGAs GPUs and other accelerators, accessed August 14, 2025, https://www.youtube.com/watch?v=celGdC48NzU
61. [2402.06297] Dynamic Q-planning for Online UAV Path Planning in Unknown and Complex Environments - arXiv, accessed August 14, 2025, https://arxiv.org/abs/2402.06297
62. Dynamic Obstacle Avoidance and Path Planning through Reinforcement Learning - MDPI, accessed August 14, 2025, https://www.mdpi.com/2076-3417/13/14/8174
63. AgilePilot: DRL-Based Drone Agent for Real-Time Motion Planning in Dynamic Environments by Leveraging Object Detection - arXiv, accessed August 14, 2025, https://arxiv.org/html/2502.06725v1
64. Agile Drone Flight - Robotics and Perception Group, accessed August 14, 2025, https://rpg.ifi.uzh.ch/aggressive_flight.html
65. Genetic Algorithm Based System for Path Planning with Unmanned Aerial Vehicles Swarms in Cell-Grid Environments - arXiv, accessed August 14, 2025, https://arxiv.org/html/2412.03433v1
66. HKUST-Aerial-Robotics/Fast-Planner: A Robust and Efficient Trajectory Planner for Quadrotors - GitHub, accessed August 14, 2025, https://github.com/HKUST-Aerial-Robotics/Fast-Planner
67. Unified Linear Parametric Map Modeling and Perception-aware Trajectory Planning for Mobile Robotics - arXiv, accessed August 14, 2025, https://arxiv.org/html/2507.09340v1
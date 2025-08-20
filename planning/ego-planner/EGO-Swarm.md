# EGO-Swarm

## 1.  EGO-Swarm: 분산형 자율 비행의 새로운 패러다임

### 1.1  복잡하고 미지의 환경에서의 군집 비행 문제 정의

쿼드로터(quadrotor)로 대표되는 무인 항공기(UAV) 군집은 단일 개체로는 달성하기 어려운 복잡한 임무를 수행할 잠재력을 지닌다. 수색 및 구조, 환경 감시, 정밀 농업, 물류 배송 등 다양한 분야에서 드론 군집은 작업 효율성을 극대화하고, 강건성을 높이며, 임무 범위를 확장하는 혁신적인 해결책으로 주목받고 있다.1 예를 들어, 넓은 재난 지역을 수색할 때 여러 대의 드론을 동시에 투입하면 단일 드론에 비해 훨씬 빠른 시간 안에 임무를 완수할 수 있다.3

그러나 이러한 잠재력을 현실화하는 데에는 수많은 기술적 난제가 존재한다. 특히, GPS 신호가 닿지 않는 실내, 빌딩 숲, 또는 울창한 삼림과 같이 사전 지도 정보가 없고 장애물이 무질서하게 분포하는 미지의 환경에서 드론 군집을 운용하는 것은 가장 어려운 문제 중 하나로 꼽힌다. 이러한 환경에서 군집 시스템은 각 드론의 위치를 정밀하게 추정하고, 주변 환경을 실시간으로 인지하며, 동료 드론 및 장애물과 충돌하지 않도록 안전한 경로를 생성하고, 최종적으로 주어진 임무를 완수해야 한다.

전통적인 군집 제어 방식은 주로 중앙 집중식(centralized) 아키텍처에 의존해왔다. 이 방식은 하나의 강력한 지상 통제 시스템(GCS) 또는 리더 드론이 모든 드론의 상태 정보를 수집하고, 통합된 전역 계획(global plan)을 수립하여 각 드론에 명령을 하달하는 구조이다. 이러한 접근법은 드론 간의 정밀한 협력과 최적의 군집 행동을 유도할 수 있다는 장점이 있지만, 치명적인 단점을 내포한다. 첫째, 중앙 제어 시스템은 단일 고장점(single point of failure)이 된다. 만약 GCS에 문제가 생기거나 리더 드론이 추락하면 군집 전체가 마비될 수 있다.4 둘째, 드론의 수가 증가할수록 중앙 서버로 집중되는 통신량이 기하급수적으로 늘어나 병목 현상을 유발하며, 이는 시스템의 반응성을 저하시키고 확장성을 심각하게 제한한다.1

이러한 한계를 극복하기 위해 분산형(decentralized) 시스템의 필요성이 대두되었다. 분산형 시스템에서는 각 드론이 독립적인 지능을 가진 에이전트(agent)로서, 중앙의 개입 없이 스스로 환경을 인지하고 판단하여 행동을 결정한다. 이는 시스템의 강건성과 확장성을 획기적으로 향상시킬 수 있다. 바로 이 지점에서 EGO-Swarm이 해결하고자 하는 핵심 문제가 명확해진다. EGO-Swarm의 목표는 **"외부의 도움(예: 모션 캡처 시스템, 고성능 서버) 없이, 오직 드론에 탑재된 센서와 컴퓨터 자원만을 사용하여, 사전 지도 정보가 없는 복잡하고 미지의 환경에서 여러 대의 드론이 서로 충돌하지 않고 안전하며 효율적으로 자율 비행하는 완전한 시스템을 구현하는 것"**이다.6 이는 단순한 경로 계획 알고리즘을 넘어, 인지, 계획, 제어를 통합하는 새로운 패러다임을 제시하는 도전적인 목표라 할 수 있다.

### 1.2  EGO-Swarm 솔루션: 분산, 비동기, 온보드 처리의 핵심 원칙

EGO-Swarm은 앞서 정의된 군집 비행의 난제를 해결하기 위해 세 가지 핵심 원칙을 기반으로 설계되었다. 이 원칙들은 시스템의 구조와 동작 방식을 규정하며, EGO-Swarm을 기존 연구와 차별화하는 근간을 이룬다.

첫째, **분산(Decentralized)** 구조이다. EGO-Swarm에는 군집 전체를 통제하는 중앙 서버나 리더 드론이 존재하지 않는다.7 각 드론은 동등한 개체로서, 자신의 센서로 주변 환경과 다른 드론을 인지하고, 탑재된 컴퓨터로 자신의 행동을 독립적으로 결정한다. 드론 간의 상호작용은 오직 로컬 통신을 통해 자신의 상태와 계획된 궤적 정보를 공유하는 방식으로만 이루어진다. 이러한 분산 구조는 한두 대의 드론이 고장 나거나 통신이 두절되더라도 나머지 드론들이 임무를 계속 수행할 수 있게 하여 시스템 전체의 강건성(robustness)을 극대화한다. 또한, 드론의 수가 늘어나도 중앙 서버의 부하가 증가하지 않으므로 이론적으로 무한한 확장성(scalability)을 가진다.9

둘째, **비동기(Asynchronous)** 방식의 통신 및 계획이다. 동기식(synchronous) 시스템에서는 모든 드론이 다음 단계로 나아가기 전에 서로의 계획이 완료되기를 기다리는 '동기화 장벽'이 존재한다. 이는 군집의 일관성을 유지하는 데는 유리하지만, 가장 느린 드론의 속도에 전체 시스템이 맞춰지므로 반응성이 저하된다. 반면, EGO-Swarm은 비동기 방식을 채택하여 각 드론이 다른 드론의 계획 완료 여부와 상관없이, 이웃 드론으로부터 새로운 궤적 정보를 수신하는 즉시 자신의 경로를 재계획한다.7 이는 예측 불가능한 장애물이 나타나거나 환경이 급변할 때 시스템이 매우 신속하게 대응할 수 있도록 한다. 하지만 이러한 설계는 각 드론이 서로 다른 시점의 정보를 바탕으로 계획을 수립하게 되므로, 통신 지연이 심할 경우 정보의 불일치로 인한 충돌 위험을 내포하는 양날의 검과 같다. 이 문제는 후술할 시스템의 한계점에서 더 깊이 다루어진다.11

셋째, **완전한 온보드 시스템(Fully Onboard System)**의 구현이다. 기존의 많은 드론 군집 연구는 Vicon이나 OptiTrack과 같은 고정밀 외부 모션 캡처 시스템으로 드론의 위치를 추정하거나, 연산량이 많은 작업을 강력한 성능의 지상 컴퓨터로 처리(offboard-processing)하는 방식에 의존했다. 이는 실험실 환경에서는 유효하지만, 실제 현장 적용에는 큰 제약이 따른다. EGO-Swarm은 이러한 외부 의존성을 완전히 배제하고, 인지, 위치 추정, 경로 계획, 제어에 필요한 모든 연산을 드론에 탑재된 소형 컴퓨터와 센서만으로 수행한다.6 이는 EGO-Swarm이 실험실을 벗어나 실제 미지의 환경에서 진정한 자율성을 가지고 임무를 수행할 수 있음을 의미하며, 시스템의 실용성을 한 차원 높인 핵심적인 성과이다.

이 세 가지 원칙의 결합은 EGO-Swarm을 단순한 시뮬레이션 연구나 제한된 환경에서의 기술 시연을 넘어, 실제 세계의 복잡성과 불확실성에 대응할 수 있는 강력하고 실용적인 군집 비행 솔루션으로 만들었다.

### 1.3  시스템 아키텍처 개요: 인지, 계획, 제어의 통합

EGO-Swarm의 가장 중요한 학술적 기여 중 하나는 이것이 단편적인 알고리즘의 집합이 아니라, 인지(Perception), 계획(Planning), 제어(Control)라는 로봇 공학의 세 가지 핵심 요소가 온보드 시스템 내에서 유기적으로 통합된 최초의 **'체계적인 솔루션(systematic solution)'**이라는 점이다.7 이전의 연구들은 종종 이 문제들을 분리하여 다루었다. 예를 들어, 어떤 연구는 완벽한 위치 정보와 지도 정보를 가정하고 경로 계획 알고리즘 자체에만 집중했으며, 또 다른 연구는 외부의 도움을 받아 인지와 제어 문제를 해결했다.7

EGO-Swarm은 이러한 단편적인 접근법에서 벗어나, 현실 세계의 드론이 직면하는 전체 문제의 흐름을 온보드에서 해결하는 통합 프레임워크를 제시했다. 시스템의 작동 흐름은 다음과 같다.

1. **인지 (Perception):** 각 드론은 탑재된 뎁스 카메라(depth camera)와 관성 측정 장치(IMU)를 사용하여 자신의 3차원 공간상 위치를 추정(Visual-Inertial Odometry, VIO)하고, 주변의 정적 장애물과 동적인 다른 드론들을 실시간으로 감지하여 로컬 맵(local map)을 생성한다.
2. **계획 (Planning):** 생성된 로컬 맵과 통신을 통해 수신한 다른 드론들의 미래 궤적 정보를 바탕으로, 각 드론은 자신의 다음 목표 지점까지 안전하고, 부드러우며, 동역학적으로 실현 가능한 최적의 궤적을 수 밀리초(milliseconds) 안에 계산해낸다.
3. **제어 (Control):** 생성된 궤적을 비행 컨트롤러(Flight Controller, FC)가 정확하게 추종하도록 모터에 필요한 명령을 실시간으로 전송한다.

이러한 모든 과정이 외부의 개입 없이 각 드론 내부에서 독립적이고 지속적으로 반복된다. 이처럼 복잡한 서브시스템들을 경량의 온보드 컴퓨터에서 실시간으로 통합하여 안정적으로 구동하고, 이를 여러 대의 드론으로 확장하여 성공적으로 군집 비행을 시연한 것은 EGO-Swarm이 이룬 획기적인 성과이다. 이전까지 이론이나 시뮬레이션으로만 가능하다고 여겨졌던, 미지의 복잡한 환경에서의 완전 자율 군집 비행이 실제로 가능하다는 것을 물리적으로 입증한 것이다.

이러한 시스템 통합의 성공은 로봇 공학계에 큰 반향을 일으켰고, 그 중요성을 인정받아 세계적인 과학 저널인 *Science* 매거진의 뉴스 기사로 소개되기도 했다.8 이는 EGO-Swarm이 단순히 특정 기술의 발전을 넘어, 자율 로봇 시스템 연구의 방향성을 '실제 환경에서의 완전한 자율성 구현'으로 한 단계 끌어올린 중요한 이정표임을 시사한다. EGO-Swarm은 자율 군집 비행 연구의 기준을 새롭게 정립했으며, 이후의 많은 연구들이 EGO-Swarm이 제시한 통합 시스템의 개념 위에서 출발하게 되는 계기를 마련했다.

## 2.  EGO-Swarm의 수학적 기반

EGO-Swarm의 혁신적인 성능은 정교한 수학적 모델링에 깊이 뿌리내리고 있다. 특히, 경사도 기반 최적화(gradient-based optimization)를 핵심 도구로 사용하되, 기존 방식의 계산적 비효율성을 극복한 새로운 접근법을 제시했다. 이 섹션에서는 EGO-Swarm의 기반이 된 단일 드론 플래너 EGO-Planner의 핵심 아이디어부터 군집으로 확장되기까지의 수학적 원리를 심층적으로 분석한다.

### 2.1  선행 연구: EGO-Planner의 ESDF-Free 최적화

EGO-Swarm은 단일 쿼드로터의 고속 자율 비행을 위해 개발된 EGO-Planner를 군집 환경으로 확장한 것이다.7 따라서 EGO-Swarm을 이해하기 위해서는 먼저 EGO-Planner의 핵심 철학을 파악해야 한다.

기존의 경사도 기반 지역 경로 계획(local planning) 방법들은 대부분 ESDF(Euclidean Signed Distance Field) 맵에 크게 의존했다. ESDF는 맵 상의 모든 지점에서 가장 가까운 장애물까지의 부호 있는 유클리드 거리를 저장한 필드로, 궤적이 장애물로부터 얼마나 떨어져 있는지와 안전한 방향(경사도)을 즉시 알려주는 강력한 데이터 구조이다. 하지만 이의 생성 및 유지에는 막대한 계산 비용이 수반된다. 한 연구에 따르면, 지역 경로 계획에 소요되는 전체 연산 시간의 약 70%가 ESDF 맵을 계산하는 데 사용될 정도로 심각한 병목 현상을 야기했다.15

EGO-Planner는 이러한 비효율성에 근본적인 의문을 제기했다. 실제로 경로 최적화 과정에서 필요한 정보는 전체 맵 공간이 아니라, 현재 계획 중인 궤적 주변의 매우 제한된 영역에 불과하다. EGO-Planner의 핵심 혁신은 바로 이 지점에서 출발한다. 즉, **ESDF 맵 전체를 미리 계산하는 대신, 최적화 과정에서 궤적이 실제로 장애물과 충돌 위험에 처했을 때만 '필요에 따라(on-demand)' 충돌 관련 정보를 계산하는 'ESDF-Free' 방식을 제안**한 것이다.14 이 접근법은 불필요한 연산을 제거하여 경로 계획 시간을 수십 밀리초에서 단 몇 밀리초 수준으로 극적으로 단축시켰고, 이는 온보드 컴퓨터에서의 실시간 고속 재계획을 가능하게 하는 결정적인 열쇠가 되었다.

#### 2.1.1  B-스플라인을 이용한 궤적 표현

EGO-Planner는 궤적을 표현하기 위한 수학적 도구로 B-스플라인(B-spline)을 사용한다. B-스플라인은 다항식 곡선들을 부드럽게 연결하여 복잡한 형태의 궤적을 표현하는 데 매우 효과적이다. 특히, 궤적의 형태가 소수의 제어점(control points)에 의해 결정되므로 최적화 문제의 변수 수를 줄일 수 있고, 곡선의 볼록 포(convex hull) 성질 덕분에 궤적 전체의 동역학적 제약(최대 속도, 가속도 등)을 제어점의 제약 조건으로 변환하여 쉽게 처리할 수 있다는 장점이 있다.7

$p$차 균일 B-스플라인(uniform B-spline)으로 표현된 궤적 $\mathbf{\Phi}(t)$의 시간 $t$에서의 위치는 다음과 같은 행렬 형태로 간결하게 나타낼 수 있다 7:
$$
\mathbf{\Phi}(t) = \mathbf{s}(t)^T \mathbf{M} \mathbf{q}
$$
여기서 각 항의 의미는 다음과 같다.

- $\mathbf{\Phi}(t) \in \mathbb{R}^3$: 시간 $t$에서의 3차원 위치 벡터.
- $\mathbf{s}(t) =^T$: 시간 $t$가 속한 매듭 구간(knot span) $[t_i, t_{i+1}]$ 내에서 정규화된 시간으로 구성된 기저 함수 벡터.
- $\mathbf{M}$: B-스플라인의 차수 $p$에 의해 유일하게 결정되는 상수 변환 행렬.
- $\mathbf{q} = [\mathbf{Q}_{i-p}, \dots, \mathbf{Q}_i]^T$: 궤적의 모양을 결정하는 제어점(control points)들의 벡터. 최적화 문제에서 우리가 찾아야 할 결정 변수(decision variables)가 바로 이 제어점들이다.

#### 2.1.2  ESDF-Free 충돌 비용 함수

EGO-Planner의 핵심인 ESDF-Free 충돌 비용 함수 $J_c$는 궤적이 장애물과 충돌했을 때, 그 궤적을 장애물로부터 밀어내는 힘(경사도)을 어떻게 계산할 것인가에 대한 해답이다. 이는 충돌이 발생한 궤적의 제어점과, 충돌하지 않는 안전한 '가이드 경로(guiding path)'를 비교하는 방식으로 공식화된다.15

구체적으로, B-스플라인 궤적의 제어점 $\mathbf{Q}_i$가 장애물 영역에 들어갔다고 가정하자. 시스템은 이 제어점 $\mathbf{Q}_i$에 대해 가장 가까운 장애물 표면 위의 점(앵커 포인트) $\mathbf{p}_{ij}$와 그 점에서 바깥쪽을 향하는 단위 법선 벡터(안전 방향 벡터) $\mathbf{v}_{ij}$를 계산한다. 이 두 정보를 이용해 제어점과 장애물 간의 근사적인 거리 $d_{ij}$를 다음과 같이 정의할 수 있다 13:
$$
d_{ij} = (\mathbf{Q}_i - \mathbf{p}_{ij}) \cdot \mathbf{v}_{ij}
$$
이 식은 제어점 $\mathbf{Q}_i$에서 장애물 표면까지의 거리를, 안전 방향 $\mathbf{v}_{ij}$로의 투영(projection)을 통해 근사하는 것이다. 만약 $d_{ij}$가 사용자가 설정한 안전 여유(safety margin)보다 작아지면(즉, 충돌 위험이 발생하면), 이 값을 이용해 페널티 비용과 경사도를 계산하여 최적화 과정에 반영한다. 이 정보($(\mathbf{p}_{ij}, \mathbf{v}_{ij})$ 쌍)는 필요할 때만 계산되고 일시적으로 저장되므로, ESDF 맵 전체를 유지할 필요가 전혀 없다.

#### 2.1.3  단일 드론의 전체 목적 함수

EGO-Planner의 최종 목표는 안전하고(safe), 부드러우며(smooth), 동역학적으로 실현 가능하고(dynamically feasible), 목표 지점에 잘 도달하는(goal-oriented) 궤적을 찾는 것이다. 이는 여러 상충하는 목표들을 동시에 만족시키는 다중 목표 최적화 문제로, 다음과 같은 가중 합 형태의 단일 목적 함수 $J_{\text{EGO}}$를 최소화하는 문제로 공식화된다.13
$$
\min_{\mathbf{Q}} J_{\text{EGO}} = \lambda_s J_s + \lambda_c J_c + \lambda_d J_d + \lambda_t J_t
$$
각 비용 항의 의미는 다음과 같다.

- $J_s$ (Smoothness): 궤적의 평활도 비용. 주로 속도, 가속도, 저크(jerk) 등 고차 미분값의 제곱 적분으로 정의된다. 이는 궤적을 부드럽게 만들고 에너지 소모를 줄이는 역할을 한다.
- $J_c$ (Collision): 위에서 설명한 ESDF-Free 방식의 충돌 비용. 궤적이 장애물에 가까워질수록 페널티를 부과한다.
- $J_d$ (Dynamical Feasibility): 동역학적 실현 가능성 비용. 드론의 최대 속도, 최대 가속도 등 물리적 한계를 초과하지 않도록 페널티를 부과한다.
- $J_t$ (Terminal Goal): 최종 목표점 비용. 궤적의 끝점이 주어진 목표 지점과 가깝도록 유도한다.
- $\lambda_s, \lambda_c, \lambda_d, \lambda_t$: 각 비용 항의 상대적 중요도를 조절하는 가중치 파라미터.

비선형 최적화 솔버(L-BFGS 등)를 이용해 이 목적 함수를 최소화하는 제어점 $\mathbf{Q}$를 찾음으로써, 단일 드론은 미지의 환경에서도 빠르고 안전하게 비행할 수 있는 궤적을 실시간으로 생성할 수 있다.

### 2.2  군집으로의 확장: 상호 충돌 회피

EGO-Planner가 구축한 효율적인 최적화 프레임워크는 군집 환경으로의 확장을 위한 견고한 기반이 된다. EGO-Swarm은 EGO-Planner의 목적 함수에 드론 간 상호 충돌을 방지하기 위한 새로운 페널티 항을 추가하는 직관적인 방식으로 군집 비행 문제를 해결한다. 이러한 접근 방식은 기존 프레임워크의 수정 없이 기능을 확장할 수 있는 모듈식 설계의 우수성을 보여준다.

#### 2.2.1  드론 간 상호 충돌 페널티 모델링

EGO-Swarm은 다른 드론을 일종의 움직이는 동적 장애물로 간주한다. 에이전트 $k$의 궤적 $\mathbf{\Phi}_k(t)$를 최적화할 때, 통신을 통해 수신한 다른 모든 에이전트 $i$의 미래 궤적 $\mathbf{\Phi}_i(t)$를 고려하여 충돌 페널티 $J_w$ (Swarm Penalty)를 계산한다.7

두 드론 $k$와 $i$ 사이의 시간 $t$에서의 거리 $d_{k,i}(t)$가 미리 설정된 최소 안전 거리 $\mathcal{C}$와 버퍼 $\epsilon$의 합보다 작아지면 페널티가 부과되는 소프트 제약(soft constraint)으로 모델링된다. 에이전트 $k$에 대한 전체 군집 충돌 페널티 $J_{w,k}$는 다음과 같이 공식화할 수 있다 7:
$$
J_{w,k} = \sum_{i \neq k} \int_{t_s}^{t_e} \max(0, -d_{k,i}(t))^2 dt
$$
여기서 거리 함수 $d_{k,i}(t)$는 다음과 같이 정의된다.
$$
d_{k,i}(t) = ||\mathbf{E}^{1/2}[\mathbf{\Phi}_k(t) - \mathbf{\Phi}_i(t)]|| - (\mathcal{C} + \epsilon)
$$
이 식에서 주목할 점은 변환 행렬 $\mathbf{E} = \text{diag}(1, 1, 1/c)$ ($c>1$)의 사용이다.7 이는 단순한 유클리드 거리가 아닌 타원체 거리(ellipsoidal distance)를 사용함을 의미한다. 쿼드로터는 프로펠러 회전으로 인해 아래 방향으로 강한 바람(하강풍, downwash)을 생성하는데, 한 드론이 다른 드론의 바로 위를 비행할 경우 이 하강풍이 아래 드론의 비행 안정성을 심각하게 해칠 수 있다. 행렬 $\mathbf{E}$는 수직(z축) 방향의 거리에 더 큰 가중치를 부여하여, 드론들이 수직으로 겹치는 것을 수평으로 가까이 있는 것보다 더 강하게 회피하도록 유도한다. 이는 단순한 기하학적 충돌 모델을 넘어, 실제 쿼드로터의 공기역학적 특성까지 고려한 정교한 설계이며, EGO-Swarm의 실용성을 보여주는 중요한 디테일이다.

최종적으로 각 드론은 자신의 최적화 문제에 이 군집 페널티 항을 추가하여 다음의 목적 함수를 최소화한다.
$$
\min_{\mathbf{Q}_k} J_{\text{total}} = J_{\text{EGO}} + \lambda_w J_{w,k}
$$

#### 2.2.2  비동기적 궤적 공유 및 최적화

EGO-Swarm의 군집 충돌 회피 메커니즘은 비동기적 궤적 공유를 전제로 동작한다. 각 드론은 자신의 최적화된 궤적을 와이파이(Wi-Fi)와 같은 불안정한 통신 네트워크를 통해 주기적으로 주변에 브로드캐스트한다.7 다른 드론의 궤적 정보를 수신하면, 그 즉시 해당 정보를 자신의 최적화 문제에 반영하여 새로운 경로를 계산한다. 이 과정에서 다른 모든 드론과 통신이 성공하기를 기다리거나 계획이 끝날 때까지 대기하지 않는다.10 이러한 비동기적 접근은 시스템 전체의 지연을 최소화하고, 일부 드론과의 통신이 일시적으로 두절되더라도 시스템이 멈추지 않도록 하여 확장성과 반응성을 보장하는 핵심 요소이다.

### 2.3  강건성 향상: 위상학적 경로 계획 및 드리프트 보정

EGO-Swarm은 단순히 효율적인 최적화 프레임워크를 제공하는 데 그치지 않고, 실제 환경에서 발생할 수 있는 두 가지 고질적인 문제, 즉 지역 최솟값(local minima) 문제와 위치 추정 오차 누적(localization drift) 문제를 해결하기 위한 독창적인 메커니즘을 통합하여 시스템의 강건성을 크게 향상시켰다.

#### 2.3.1  지역 최솟값 탈출을 위한 경량 위상학적 경로 생성

경사도 기반 최적화 방법은 계산이 빠르다는 장점이 있지만, 목적 함수가 볼록(convex)하지 않을 경우 전역 최적해(global optimum)가 아닌 지역 최적해(local optimum)에 수렴할 위험이 항상 존재한다. 이는 드론이 더 좋은 경로가 있음에도 불구하고 장애물에 둘러싸인 '함정'에 빠져 움직이지 못하는 상황으로 이어질 수 있다.

이 문제를 해결하기 위해 EGO-Swarm은 위상학적 경로 계획(topological planning) 기법을 경량화하여 통합했다. 이는 장애물을 기준으로 '왼쪽으로 돌아가는 경로'와 '오른쪽으로 돌아가는 경로'처럼 위상학적으로 서로 다른(topologically distinct) 여러 경로 후보를 생성하는 방법이다. EGO-Swarm은 기존 연구들을 발전시켜 거의 추가적인 계산 비용 없이 이러한 다양한 경로들을 생성하고, 그중에서 가장 비용이 낮은 경로를 선택하여 최적화를 수행한다. 이를 통해 드론이 하나의 경로에서 지역 최솟값에 빠지더라도, 위상학적으로 다른 더 나은 경로를 탐색하여 탈출할 기회를 가질 수 있게 된다.7

#### 2.3.2  온보드 인지를 통한 상대 위치 드리프트 보상

장시간 비행 시, VIO(Visual-Inertial Odometry) 시스템은 필연적으로 위치 추정 오차(drift)가 누적된다. 이는 각 드론이 인식하는 자신의 절대 좌표가 실제 위치와 점차 달라짐을 의미한다. 군집 비행에서 각 드론의 드리프트 방향과 크기가 다르다면, 통신으로 공유된 좌표계와 실제 물리적 위치 간의 불일치가 발생하여 충돌 회피 알고리즘이 오작동할 수 있다.

EGO-Swarm은 이 문제를 해결하기 위해 외부 위치 보정 시스템(예: GPS-RTK)에 의존하는 대신, 드론 간의 상대적 관측을 이용한 독창적인 드리프트 보정 메커니즘을 제안했다. 각 드론은 자신의 뎁스 카메라를 이용해 시야에 들어온 다른 드론을 물체로서 감지한다. 그리고 감지된 드론의 실제 측정 위치와, 통신을 통해 수신한 해당 드론의 (드리프트가 포함된) VIO 기반 예측 위치를 비교한다. 이 둘 사이의 오차 벡터를 계산하고, 이를 필터링하여 상대적인 위치 드리프트를 추정하고 자신의 좌표계를 보정한다.8 이 기법은 군집 내 드론들이 서로를 볼 수 있는 한, 외부의 도움 없이도 군집 전체의 위치 일관성을 유지시켜 장시간 임무 수행의 안정성을 보장하는 핵심적인 기능이다.

## 3.  구현 및 구성 가이드

EGO-Swarm의 이론적 우수성을 실제 비행으로 구현하기 위해서는 소프트웨어 환경 설정부터 하드웨어 구성, 파라미터 튜닝에 이르기까지 체계적인 접근이 필요하다. 이 섹션에서는 EGO-Swarm 시스템을 구축하고 운용하는 데 필요한 실질적인 가이드를 제공한다.

### 3.1  소프트웨어 환경 설정

EGO-Swarm은 ROS(Robot Operating System)를 기반으로 개발되었으며, 특정 버전의 운영체제와 라이브러리에 대한 의존성을 가진다.

#### 3.1.1  시스템 요구사항 및 의존성

성공적인 컴파일과 실행을 위해 다음의 환경을 구축해야 한다.

- **운영체제:** Ubuntu 16.04 (ROS Kinetic), 18.04 (ROS Melodic), 또는 20.04 (ROS Noetic)가 공식적으로 지원된다.14

- **미들웨어:** 해당 Ubuntu 버전에 맞는 ROS Desktop-Full 버전을 설치해야 한다. ROS는 패키지 간의 통신, 빌드 시스템, 시각화 도구 등을 제공하는 EGO-Swarm의 근간이다.18

- **필수 라이브러리:** `uav_simulator` 패키지는 고성능 C++ 선형대수 라이브러리인 `Armadillo`를 사용한다. 다음 명령어를 통해 설치할 수 있다 14:

  ```Bash
  sudo apt-get install libarmadillo-dev
  ```

- **GPU 가속 (선택사항):** 시뮬레이션에서 뎁스 카메라를 모사하는 `local_sensing` 패키지는 GPU를 이용한 렌더링을 지원한다. 이 기능을 활성화하려면 시스템에 NVIDIA 그래픽 카드와 해당 드라이버, 그리고 CUDA Toolkit이 설치되어 있어야 한다. GPU를 사용하면 더 현실적인 센서 데이터 생성이 가능하지만, CPU만으로도 기본적인 포인트 클라우드 기반 시뮬레이션은 가능하다.18

#### 3.1.2  소스 코드 컴파일 및 시뮬레이션 실행

환경 설정이 완료되면, GitHub에서 소스 코드를 내려받아 컴파일하고 시뮬레이션을 실행할 수 있다.

1. **소스 코드 클론:** 터미널에서 다음 명령어를 실행하여 EGO-Swarm의 공식 저장소를 클론한다.18

   ```Bash
   git clone https://github.com/ZJU-FAST-Lab/ego-planner-swarm.git
   ```

2. **컴파일:** 클론된 디렉토리로 이동하여 ROS의 빌드 도구인 `catkin_make`를 실행한다. 최적화된 성능을 위해 릴리즈 모드로 컴파일하는 것이 권장된다.18

   ```Bash
   cd ego-planner-swarm
   catkin_make -DCMAKE_BUILD_TYPE=Release
   ```

3. **시뮬레이션 실행:** 컴파일이 성공적으로 완료되면, 두 개의 터미널을 열어 각각 시각화 도구와 플래너를 실행한다. 각 터미널에서 `source devel/setup.bash`를 먼저 실행하여 환경 변수를 설정해야 한다.

   - **터미널 1 (시각화 도구 RViz 실행):**

     ```Bash
     source devel/setup.bash
     roslaunch ego_planner rviz.launch
     ```

     RViz는 계획된 궤적, 생성된 지도, 드론의 현재 상태 등을 3D로 시각화해주며, 사용자가 마우스 클릭으로 목표 지점을 설정하는 인터페이스(`2D Nav Goal`)를 제공한다.18

   - **터미널 2 (군집 플래너 시뮬레이션 실행):**

     ```Bash
     source devel/setup.bash
     roslaunch ego_planner swarm.launch
     ```

     이 명령은 시뮬레이션 환경 내에서 여러 대의 드론을 생성하고, 각 드론이 EGO-Swarm 플래너를 실행하도록 한다.18

만약 단일 드론의 성능을 테스트하고 싶다면, `swarm.launch` 파일 내에서 `drone_id` 파라미터를 0으로 설정하여 간단하게 단일 에이전트 모드로 전환할 수 있다.14

### 3.2  핵심 소프트웨어 모듈 분석

EGO-Swarm은 여러 개의 독립적인 ROS 패키지들이 유기적으로 상호작용하여 구성된 복합 시스템이다. 각 패키지의 역할을 이해하는 것은 시스템을 디버깅하거나 특정 기능을 수정, 확장하는 데 필수적이다.

**EGO-Swarm 소프트웨어 패키지 개요**

| Package Name              | Primary Function                                             | Key Dependencies / Notes                                     |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `ego-planner`             | 핵심 경로 계획 로직을 포함한다. B-스플라인 기반 궤적 최적화, 충돌 검사, 군집 회피 등 EGO-Swarm의 메인 알고리즘이 구현되어 있다. |                                                              |
| `local_sensing`           | 시뮬레이션 환경에서 센서 데이터를 생성하는 역할을 한다. 실제 뎁스 카메라처럼 뎁스 이미지를 렌더링하거나, 간단한 포인트 클라우드를 발행한다. | `CMakeLists.txt` 파일의 `ENABLE_CUDA` 플래그를 `true`로 설정하면 GPU 가속을 사용한다. `false`일 경우 CPU 기반으로 동작한다.18 |
| `uav_simulator`           | 드론의 동역학을 시뮬레이션하고, 외부 환경(장애물 등)을 제공하는 가상 환경이다. | `Armadillo` 라이브러리에 의존한다.18                         |
| `fake_drone`              | 간단한 기구학적 모델(kinematic model)을 사용하여 드론의 움직임을 계산한다. 복잡한 미분 방정식을 풀지 않아 계산 부하가 매우 적다. 대규모 군집(15대 이상) 시뮬레이션 시 CPU 부담을 줄이기 위해 기본으로 사용된다.18 | `simulator.xml` 파일에서 실제 동역학 시뮬레이터로 교체할 수 있다. |
| `quadrotor_simulator_so3` | 더 현실적인 쿼드로터 동역학 모델을 제공하는 고충실도 시뮬레이터이다. | `simulator.xml` 파일에서 해당 노드의 주석을 해제하여 활성화할 수 있다. 이 경우 `fake_drone`은 비활성화해야 한다.18 |
| `so3_control`             | `quadrotor_simulator_so3` 패키지와 함께 사용되는 SO(3) 기반 제어기이다. 고충실도 동역학 시뮬레이션을 위해 필수적이다.18 |                                                              |

이 모듈 구조에서 특히 주목할 점은 시뮬레이션의 충실도(fidelity)와 확장성(scalability) 사이의 트레이드오프를 사용자가 선택할 수 있도록 설계했다는 것이다. 소규모 군집으로 실제와 유사한 동역학적 거동을 테스트하고 싶을 때는 `quadrotor_simulator_so3`를, 수십 대 이상의 대규모 군집에서 알고리즘의 확장성을 검증하고 싶을 때는 계산적으로 가벼운 `fake_drone`을 사용하는 유연성을 제공한다.

### 3.3  실제 드론 구성을 위한 하드웨어 가이드

EGO-Swarm을 시뮬레이션을 넘어 실제 드론에 탑재하기 위해서는 소프트웨어의 요구사항을 충족하는 하드웨어 구성이 필수적이다. 공식 문서에는 구체적인 하드웨어 목록이 명시되어 있지 않지만, 소프트웨어의 기능과 커뮤니티의 일반적인 구성을 종합하여 다음과 같은 권장 사양을 제시할 수 있다.

#### 3.3.1  온보드 컴퓨터 (Onboard Computer)

온보드 컴퓨터는 EGO-Swarm의 두뇌에 해당하며, ROS, VIO, 플래닝 알고리즘 등 모든 핵심 소프트웨어를 실행한다. 따라서 충분한 연산 능력이 가장 중요한 요구사항이다. 특히 `local_sensing` 패키지의 GPU 가속 옵션과 딥러닝 기반의 객체 감지(상대 드론 감지) 등을 효율적으로 사용하기 위해서는 NVIDIA GPU가 탑재된 임베디드 보드가 사실상 필수적이다. **NVIDIA Jetson 시리즈(예: Xavier NX, Orin Nano, AGX Orin)**가 강력하게 권장된다.

#### 3.3.2  비행 컨트롤러(FC) 및 펌웨어 (Flight Controller and Firmware)

비행 컨트롤러는 온보드 컴퓨터로부터 목표 궤적 또는 자세 명령을 받아 모터를 제어하고 비행 안정성을 유지하는 역할을 한다. 업계 표준으로 널리 사용되는 **Pixhawk 시리즈(예: Pixhawk 4, Pixhawk 5X, Pixhawk 6X)**와 같은 고성능 FC가 적합하다. 펌웨어는 **PX4** 또는 **ArduPilot**을 사용할 수 있으며, 이들은 MAVLink 프로토콜을 통해 온보드 컴퓨터의 ROS와 통신한다(MAVROS 패키지 사용).19

#### 3.3.3  센서 스위트 (Sensor Suite)

EGO-Swarm의 성능은 센서 데이터의 품질에 크게 좌우된다.

- **핵심 센서:** 3차원 환경 인지, 장애물 회피, 상대 드론 감지를 위해 **뎁스 카메라**가 필수적이다. VIO를 위한 IMU가 내장된 **Intel RealSense D435i** 또는 더 넓은 시야각과 긴 측정 거리를 제공하는 **D455** 모델이 널리 사용된다.8
- **위치 정확도 향상 (선택사항):** VIO만으로는 장시간 비행 시 절대 위치 오차가 누적될 수 있다. 옥외 환경에서 군집의 절대 위치 정확도를 높여야 하는 임무의 경우, 센티미터 수준의 정밀도를 제공하는 **GPS RTK(Real-Time Kinematic) 모듈(예: Sparkfun ZED-F9P, Holybro H-RTK F9P)**을 추가로 장착할 수 있다.19

**EGO-Swarm 에이전트 권장 하드웨어 구성**

| Component              | Recommended Model(s)                  | Rationale / Connection to EGO-Swarm                          |
| ---------------------- | ------------------------------------- | ------------------------------------------------------------ |
| Onboard Computer       | NVIDIA Jetson Xavier NX / Orin Nano   | ROS, VIO, EGO-Swarm의 동시 실행 및 GPU 가속(CUDA)을 통한 인지 성능 극대화를 위해 필수적이다.18 |
| Flight Controller (FC) | Pixhawk 6X, Holybro Durandal          | 신뢰성 높은 PX4/ArduPilot 펌웨어와 호환되며, MAVROS를 통해 온보드 컴퓨터와 안정적으로 통신한다.19 |
| Depth Camera           | Intel RealSense D435i / D455          | 3D 환경 맵핑, 정적 장애물 인지, 그리고 상대 드론 감지를 통한 드리프트 보정 기능의 핵심 센서이다.8 |
| Communication          | SiK Telemetry Radio, Wi-Fi (2.4/5GHz) | 드론 간 궤적 정보를 비동기적으로 공유하고, GCS와 데이터를 교환한다. EGO-Swarm은 불안정한 네트워크 환경에서도 작동하도록 설계되었다.7 |
| Frame/Motors/ESCs      | 250mm ~ 500mm급 쿼드로터 프레임       | 탑재하는 장비의 총 무게와 요구되는 비행 시간/기동성에 따라 적절한 크기와 추력의 부품을 선택해야 한다. |

### 3.4  GCS 연동 및 파라미터 튜닝

#### 3.4.1  RViz와 QGroundControl의 역할 분담

EGO-Swarm 운용 시, 두 가지 종류의 GCS 소프트웨어가 각각 다른 목적으로 사용된다.

- **RViz:** ROS의 표준 3D 시각화 도구로, 주로 **개발 및 시뮬레이션 단계**에서 사용된다. 플래너가 내부적으로 생성하는 지도, 계산된 궤적, 제어점, 장애물 정보 등 알고리즘의 상세한 상태를 시각적으로 확인하는 데 필수적이다. 또한, 시뮬레이션 환경에서 드론 군집에게 목표 지점을 지정하는 인터페이스 역할도 수행한다 (`2D Nav Goal` 기능).18
- **QGroundControl (QGC):** **실제 비행 운용**을 위한 GCS이다. FC의 펌웨어와 직접 통신하며, 기체의 핵심적인 설정과 모니터링을 담당한다. 주요 기능으로는 센서 캘리브레이션, 라디오(조종기) 설정, 비행 모드(예: Manual, Position, Mission) 변경, 배터리 전압 및 GPS 상태 모니터링, 자율 비행 미션의 업로드 및 실행/중지 명령 등이 있다. QGC는 여러 대의 드론에 동시에 연결하여 각 기체의 상태를 개별적으로 관리할 수 있다.21 EGO-Swarm의 상위 레벨 군집 제어(예: 대형 형성, 목표 분배)는 온보드 컴퓨터의 ROS 노드에서 처리되며, QGC는 이 자율 비행을 시작하고 감독하며, 비상시 수동으로 제어권을 가져오는 안전장치 및 인터페이스 역할을 한다.

#### 3.4.2  주요 계획 파라미터와 성능 튜닝

EGO-Swarm의 성능은 `ego-planner-swarm` 패키지 내의 `launch` 파일과 `yaml` 설정 파일에 정의된 여러 파라미터에 의해 결정된다. 최적의 비행 성능을 얻기 위해서는 드론의 기체 특성과 임무 환경에 맞게 이 값들을 튜닝하는 과정이 매우 중요하다.

- **성능 최적화 팁:** EGO-Planner의 계산 시간은 수 밀리초로 매우 짧기 때문에, 운영체제(OS)가 CPU의 클럭 속도를 동적으로 높일 시간적 여유가 없을 수 있다. 이로 인해 계획 주기가 불안정해질 수 있으므로, `sudo cpufreq-set -g performance` 명령어를 사용하여 CPU를 항상 최대 성능 모드로 고정하는 것이 안정적인 성능 확보에 큰 도움이 된다.18

**EGO-Swarm 핵심 튜닝 파라미터 (YAML 파일 내)**

| Parameter Name       | Description                                     | Likely File Location  | Effect of Increasing Value                                   |
| -------------------- | ----------------------------------------------- | --------------------- | ------------------------------------------------------------ |
| `lambda_smooth`      | 궤적의 평활도($J_s$) 비용에 대한 가중치.        | `planner_config.yaml` | 궤적이 더 부드러워지고 에너지 효율이 높아지지만, 장애물을 회피하는 기동이 다소 둔해질 수 있다. |
| `lambda_collision`   | 정적 장애물과의 충돌($J_c$) 비용에 대한 가중치. | `planner_config.yaml` | 장애물로부터 더 멀리, 더 안전하게 회피하려 하지만, 좁은 공간을 통과하는 데 어려움을 겪을 수 있다. |
| `lambda_swarm`       | 동료 드론과의 충돌($J_w$) 비용에 대한 가중치.   | `planner_config.yaml` | 드론 간의 간격을 더 넓게 유지하여 안전성을 높이지만, 밀집된 대형을 유지하기는 어려워진다. |
| `max_vel`, `max_acc` | 드론의 최대 속도 및 최대 가속도 한계.           | `planner_config.yaml` | 드론의 물리적 기동 한계를 설정한다. 이 값을 높이면 더 공격적이고 빠른 비행이 가능하지만, 제어 안정성이 저하될 수 있다. |
| `drone_id`           | 각 드론의 고유 식별자 (정수).                   | `swarm.launch`        | 군집 내에서 각 드론을 구분하고, ROS 토픽 이름을 생성하는 데 사용된다. |
| `safety_dist_m`      | 드론 간 최소 안전 거리 ($\mathcal{C}$).         | `planner_config.yaml` | 군집 충돌 회피 페널티를 계산하는 기준이 된다. 이 값을 키우면 드론들이 더 멀리 떨어져 비행한다. |

이 파라미터들은 서로 상충 관계(trade-off)에 있으므로, 시뮬레이션과 실제 테스트를 통해 특정 임무와 환경에 가장 적합한 값의 조합을 찾는 경험적인 과정이 반드시 필요하다.

## 4.  성능 분석 및 비교 벤치마킹

EGO-Swarm의 성능을 객관적으로 평가하기 위해서는 다른 최신(State-of-the-art) 경로 계획 방법들과의 정량적인 비교가 필수적이다. 이 섹션에서는 EGO-Swarm의 기반이 되는 EGO-Planner의 시뮬레이션 기반 벤치마크 결과를 분석하고, 실제 환경 실험이 가지는 의미를 고찰한다.

### 4.1  시뮬레이션 기반 평가

EGO-Planner의 성능은 동시대의 대표적인 경로 계획 방법들인 FASTER와 Fast-Planner와의 비교를 통해 검증되었다.24 이 벤치마크는 다양한 장애물 밀도를 가진 무작위 숲(Random Forest) 환경과 복잡한 구조의 사무실(Office) 환경에서 수행되었으며, 주요 평가 지표는 다음과 같다.

- **계산 시간 (Replan Time):** 경로를 한 번 재계획하는 데 걸리는 평균 시간. 시스템의 반응성을 나타낸다.
- **계획 성공률 (Success Rate):** 출발점에서 목표점까지 충돌 없이 성공적으로 도달한 비율. 플래너의 강건성을 나타낸다.
- **비행 거리 (Flight Distance):** 총 비행 거리. 경로의 효율성을 나타낸다.
- **에너지 소모 (Energy):** 궤적의 저크(jerk) 제곱 적분값으로, 궤적의 평활도(smoothness)를 나타내는 척도이다. 값이 낮을수록 부드러운 궤적을 의미한다.

**성능 벤치마크: EGO-Planner vs. 대안들 (무작위 숲, 장애물 밀도 0.4 obs/m²)**

| Method           | Replan Time (ms) | Success Rate (%) | Flight Distance (m) | Energy (m²/s⁵) | 분석 및 트레이드오프                                         |
| ---------------- | ---------------- | ---------------- | ------------------- | -------------- | ------------------------------------------------------------ |
| **EGO-Planner**  | **1.9 - 2.5**    | 66.7             | 35.87               | 464.22         | **속도 중심:** 압도적으로 빠른 계산 속도를 보이지만, 매우 복잡한 환경에서는 성공률이 다소 하락한다. |
| **Fast-Planner** | 3.2 - 3.4        | 33.3             | 37.63               | 675.38         | **강건성 부족:** 계산 속도는 빠르지만, ESDF 기반 최적화로 인해 지역 최솟값에 쉽게 빠져 성공률이 매우 낮다. |
| **FASTER**       | 29.9 - 41.0      | 53.3             | 36.07               | **329.47**     | **품질 중심:** 가장 부드럽고 에너지 효율적인 경로를 생성하지만, 계산 비용이 매우 높아 실시간 고속 재계획에는 불리하다. |

이 벤치마크 결과는 경로 계획 분야의 근본적인 트레이드오프, 즉 **'속도-품질-강건성'** 간의 상충 관계를 명확하게 보여준다.

- **EGO-Planner**는 '속도'에 극단적으로 치중한 설계 철학을 가지고 있다. ESDF-Free 접근법을 통해 달성한 수 밀리초 수준의 계산 시간은 다른 방법들이 따라올 수 없는 압도적인 장점이다. 이는 예측 불가능한 환경에 매우 신속하게 반응할 수 있음을 의미한다. 하지만 복잡도가 극심한 환경에서는 다소 단순한 가이드 경로(guiding path) 휴리스틱에 의존하기 때문에 최적의 경로를 찾지 못하고 계획에 실패하는 경우가 발생한다.24
- **FASTER**는 '품질'을 최우선으로 고려한다. 하드 제약(hard-constraint) 기반의 최적화를 통해 안전을 보장하고 매우 부드러운 경로를 생성하지만, 그 대가로 계산 시간이 수십 배 더 소요된다. 이는 변화가 잦은 동적 환경에 대한 반응성을 떨어뜨릴 수 있다.
- **Fast-Planner**는 ESDF의 경사도 정보를 직접 활용하여 빠른 최적화를 시도하지만, 이로 인해 지역 최솟값 문제에 매우 취약한 모습을 보이며 가장 낮은 성공률을 기록했다.

결론적으로, '모든 면에서 완벽한' 플래너는 존재하지 않으며, 각 방법은 서로 다른 장단점을 가진다. EGO-Planner와 EGO-Swarm의 가장 큰 가치는, 온보드 컴퓨터의 제한된 자원 내에서 실시간 고속 재계획이 가능하다는 '속도'의 한계를 돌파했다는 점에 있다. 이는 일부 극단적인 상황에서 실패할 가능성을 감수하더라도, 대부분의 시나리오에서 매우 민첩하고 공격적인 비행을 가능하게 하는 핵심 동력이다.

### 4.2  실제 환경 실험 검증

시뮬레이션은 알고리즘의 이상적인 성능을 평가하는 데 유용하지만, 실제 세계의 불확실성을 모두 반영하지는 못한다. 센서 노이즈, 제어 오차, 통신 지연, 예상치 못한 환경 변화 등 현실의 복잡성 속에서 시스템이 얼마나 잘 작동하는지를 검증하는 것은 무엇보다 중요하다.

EGO-Swarm은 이 점에서 강력한 증거를 제시한다. 논문과 함께 공개된 영상 자료는 여러 대의 드론이 외부 도움 없이 울창하고 복잡한 숲 환경을 자율적으로 비행하는 모습을 보여준다.8 이 실험에서 드론들은 다음의 과업을 성공적으로 수행했다.

1. **온보드 인지:** 각 드론은 자신의 카메라와 IMU만으로 위치를 추정하고 나무와 같은 장애물을 실시간으로 인지했다.
2. **분산 계획:** 중앙 서버 없이 각자 안전한 경로를 계산했다.
3. **상호 충돌 회피:** 서로의 미래 궤적을 공유하고 예측하여 공중에서 충돌 없이 비행했다.
4. **드리프트 보정:** 서로를 시각적으로 감지하여 누적된 위치 오차를 보정하며 군집의 일관성을 유지했다.

이러한 실제 환경에서의 성공적인 시연은 EGO-Swarm이 단순한 이론이나 시뮬레이션에 머무르지 않고, 인지-계획-제어의 전체 파이프라인이 실제 세계의 불확실성 속에서도 강건하게 작동하는 통합 시스템임을 입증하는 가장 강력한 증거이다. 이는 EGO-Swarm이 제시한 접근법의 실용성과 타당성에 대한 신뢰도를 크게 높여주며, 자율 군집 드론 기술이 실용화에 한 걸음 더 다가섰음을 보여주는 상징적인 성과라 할 수 있다.

## 5.  비판적 평가 및 향후 연구 방향

EGO-Swarm은 자율 군집 비행 분야에서 기념비적인 성과를 이루었지만, 모든 기술이 그러하듯 완벽하지 않으며 명확한 한계점을 가지고 있다. 이러한 한계를 비판적으로 고찰하고, 이를 극복하기 위한 후속 연구의 흐름을 파악하는 것은 EGO-Swarm의 진정한 가치와 유산을 이해하는 데 필수적이다.

### 5.1  식별된 한계점

EGO-Swarm의 설계 철학에서 비롯되는 두 가지 주요 한계점은 통신 지연에 대한 취약성과 센서 인지의 불안정성이다.

#### 5.1.1  통신 지연 및 패킷 손실 문제

EGO-Swarm의 가장 근본적인 약점은 **비동기적(asynchronous) 설계**에서 기인한다. 비동기 방식은 시스템의 반응 속도와 확장성을 높이는 강력한 장점이지만, 동시에 통신 지연(communication delay)에 대한 잠재적 취약성을 내포한다. EGO-Swarm의 충돌 회피 메커니즘은 각 드론이 다른 드론으로부터 수신한 '미래 궤적' 정보를 바탕으로 자신의 계획을 수립하는 것에 의존한다. 하지만 네트워크 지연이나 패킷 손실이 발생하면, 드론 A가 수신한 드론 B의 궤적은 이미 '과거의 정보'가 된다. 드론 A는 이 낡은 정보를 바탕으로 안전하다고 판단하여 계획을 수립하지만, 그 사이 드론 B는 이미 다른 경로로 비행하고 있을 수 있다. 이러한 정보의 불일치가 심각할 경우, 두 드론은 충돌 경로에 들어서게 될 수 있다.11

EGO-Swarm 원 논문에서는 이 문제를 깊이 있게 다루지 않았지만, 이 시스템의 성공에 자극받은 후속 연구 그룹들이 이 약점을 집중적으로 파고들었다. 특히, MADER와 그 후속 연구인 RMADER의 연구진은 실험을 통해 EGO-Swarm이 의도적으로 유발된 통신 지연 상황에서 실제로 충돌을 일으킬 수 있음을 보였다.11 이는 EGO-Swarm이 '이상적인 통신' 또는 '매우 낮은 지연'을 암묵적으로 가정하고 있으며, 실제 필드 환경에서 발생할 수 있는 심각한 네트워크 문제에 대해서는 안전을 보장하지 못함을 시사한다.

#### 5.1.2  센서 노이즈 및 인지 실패에 대한 민감성

EGO-Swarm은 모든 연산을 온보드 센서 데이터에 기반하는 '완전한 온보드 시스템'이다. 이는 시스템의 자율성을 높이는 핵심 요소이지만, 반대로 말하면 시스템 전체의 성능이 인지 모듈의 정확성과 안정성에 크게 의존한다는 의미이기도 하다. 실제 환경은 인지 시스템에 매우 비우호적이다.

- **시각적 도전 과제:** 갑작스러운 조명 변화, 역광, 어두운 환경 등은 카메라 이미지의 품질을 저하시킨다. 드론의 빠른 기동으로 인한 모션 블러(motion blur)는 특징점 추적을 어렵게 하여 VIO의 위치 추정 오차를 급격히 증가시킬 수 있다.
- **인식의 한계:** 빽빽한 나뭇잎이나 복잡한 구조물 뒤에 숨겨진 장애물(occlusion)은 뎁스 카메라로 감지할 수 없다. 또한, 유리나 검은색 표면과 같이 적외선 패턴을 흡수하거나 반사하는 재질은 뎁스 센서의 측정 실패를 유발한다.28

이러한 인지 실패는 치명적인 결과를 초래할 수 있다. VIO의 드리프트가 심해지면 상대 위치 보정 시스템이 오작동할 수 있고, 장애물 감지에 실패하면 충돌로 직결된다. EGO-Swarm은 이러한 센서 레벨의 불확실성을 처리하기 위한 정교한 메커니즘을 포함하고 있지는 않으므로, 인지 모듈의 성능이 전체 시스템의 강건성을 결정하는 중요한 병목 지점이 된다.30

### 5.2  군집 경로 계획의 진화: EGO-Swarm을 넘어서

과학과 기술의 발전은 선행 연구의 성공을 딛고 그 한계를 극복하는 과정에서 이루어진다. EGO-Swarm이 노출한 '통신 지연'이라는 명확한 문제는 후속 연구자들에게 중요한 화두를 던졌고, 이는 군집 경로 계획 기술을 한 단계 더 발전시키는 계기가 되었다.

대표적인 예가 **RMADER (Robust MADER)**이다.11 RMADER는 EGO-Swarm과 같이 분산형 비동기 방식을 유지하면서도 통신 지연 문제를 해결하기 위해 다음과 같은 독창적인 메커니즘을 도입했다.

1. **지연 확인 (Delay Check):** 새로운 궤적을 생성한 후 즉시 실행하지 않고, 예상되는 최대 통신 지연 시간만큼 기다리면서 다른 드론들의 궤적과 충돌이 없는지 반복적으로 확인한다.
2. **2단계 궤적 발행 (Two-step Trajectory Publication):** 충돌이 없음을 확인한 후에야 비로소 새로운 궤적을 주변에 발행하고 실행한다.

이러한 접근법은 약간의 대기 시간을 추가하는 대신, 정보 불일치로 인한 충돌을 원천적으로 방지하여 비동기 시스템의 안전성을 보장한다. RMADER의 등장은 군집 로봇 연구 분야가 '어떻게든 날게 하는 것(가능성 증명)'에서 '어떤 상황에서도 안전하게 날게 하는 것(실용적 강건성 확보)'으로 성숙하고 있음을 보여주는 중요한 사례이다. EGO-Swarm이 '속도'와 '확장성'의 가능성을 열었다면, RMADER와 같은 후속 연구들은 그 위에 '안전'과 '신뢰성'이라는 가치를 더하고 있는 것이다.

### 5.3  결론: EGO-Swarm의 유산과 미래

EGO-Swarm은 **온보드 자원만으로 미지의 복잡한 환경에서 자율 군집 비행이 가능하다는 것을 세계 최초로 물리적으로 입증한 기념비적인 연구**이다. 이 시스템이 제시한 ESDF-Free 최적화 기법은 실시간 지역 경로 계획의 효율성을 새로운 차원으로 끌어올렸으며, Fast-Planner를 비롯한 이후의 많은 연구에 직접적인 영향을 주었다.31 또한, 인지, 계획, 제어를 온보드에서 통합한 체계적인 솔루션의 제시는 자율 로봇 시스템 연구의 지향점을 제시했다.

동시에 EGO-Swarm의 유산은 그 한계점에서도 찾을 수 있다. 시스템의 가장 큰 장점인 비동기적 설계가 통신 지연이라는 가장 큰 약점의 원인이 되는 역설적인 구조는, 분산 시스템 설계의 근본적인 트레이드오프를 명확히 보여주었다. 이로 인해 후속 연구자들은 '강건성(robustness)'이라는 중요한 화두에 집중하게 되었고, 통신 지연, 센서 노이즈, 인지 실패 등 실제 환경의 불확실성을 다루는 다양한 연구들이 촉발되었다.

결론적으로 EGO-Swarm은 단순히 하나의 성공적인 시스템을 만든 것을 넘어, 자율 군집 드론 기술 분야 전체에 중요한 질문을 던지고 새로운 연구 방향을 제시했다. EGO-Swarm은 "자율 군집 비행이 가능한가?"라는 질문에 "그렇다"라고 답했으며, 동시에 "어떻게 하면 더 안전하고 신뢰성 있게 만들 수 있는가?"라는 다음 단계의 질문을 이끌어냈다. 그 성공과 한계 모두가 현재와 미래의 연구에 귀중한 자산이 되고 있으며, 자율 시스템 기술의 발전에 있어 중요한 이정표로 기억될 것이다.

#### 5.3.1 Works cited

1. Deployable Swarms: Decentralized Multi-Agent Path Planning and Control | NIMBUS Lab, accessed August 11, 2025, https://nimbus.unl.edu/deployable-swarms-decentralized-multi-agent-path-planning-and-control/
2. Sensory System for Swarm Drone: A Systematic Review, accessed August 11, 2025, https://vsrp.co.uk/wp-content/uploads/4-IJCI-Vol.-3-No.-6-June-2024-paper3-Amjad.pdf
3. Mutual Cooperation System for Task Execution Between Ground Robots and Drones Using Behavior Tree-Based Action Planning and Dynamic Occupancy Grid Mapping - MDPI, accessed August 11, 2025, https://www.mdpi.com/2504-446X/9/2/95
4. MARLander: A Local Path Planning for Drone Swarms using Multiagent Deep Reinforcement Learning - arXiv, accessed August 11, 2025, https://arxiv.org/html/2406.04159v1
5. An Efficient Real-Time Planning Method for Swarm Robotics Based on an Optimal Virtual Tube - arXiv, accessed August 11, 2025, https://arxiv.org/html/2505.01380v1
6. EGO-Swarm: A Fully Autonomous and Decentralized Quadrotor, accessed August 11, 2025, https://consensus.app/papers/egoswarm-a-fully-autonomous-and-decentralized-quadrotor-zhou-zhu/96ddafc00adf5499a9126316972e0e38
7. EGO-Swarm: A Fully Autonomous and Decentralized Quadrotor Swarm System in Cluttered Environments - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/345653588_EGO-Swarm_A_Fully_Autonomous_and_Decentralized_Quadrotor_Swarm_System_in_Cluttered_Environments
8. FAST Lab's recent work "EGO-Swarm" made frontpage news of Science Magazine, accessed August 11, 2025, http://www.cse.zju.edu.cn/cseenglish/2020/1217/c39728a2664937/page.htm
9. (PDF) Decentralized traffic management of autonomous drones - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/382177629_Decentralized_traffic_management_of_autonomous_drones
10. Decentralized Multi-Agent Planning for Multirotors: a Fully Online and Communication Latency Robust Approach - arXiv, accessed August 11, 2025, https://arxiv.org/pdf/2304.09462
11. Decentralized Multiagent Trajectory Planner Robust to Communication Delay in Dynamic Environments - arXiv, accessed August 11, 2025, https://arxiv.org/html/2303.06222v6
12. [2011.04183] EGO-Swarm: A Fully Autonomous and Decentralized Quadrotor Swarm System in Cluttered Environments - arXiv, accessed August 11, 2025, https://arxiv.org/abs/2011.04183
13. EGO-Swarm原文翻译_ego swarm-CSDN博客, accessed August 11, 2025, https://blog.csdn.net/weixin_44359306/article/details/131895939
14. ZJU-FAST-Lab/ego-planner - GitHub, accessed August 11, 2025, https://github.com/ZJU-FAST-Lab/ego-planner
15. [2008.08835] EGO-Planner: An ESDF-free Gradient-based Local Planner for Quadrotors, accessed August 11, 2025, https://arxiv.org/abs/2008.08835
16. EGO-Planner: An ESDF-Free Gradient-Based Local Planner for Quadrotors - Sci-Hub, accessed August 11, 2025, https://sci-hub.se/downloads/2021-06-05/5f/zhou2021.pdf
17. EGO-Planner: An ESDF-free Gradient-based Local Planner for Quadrotors - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/343786586_EGO-Planner_An_ESDF-free_Gradient-based_Local_Planner_for_Quadrotors
18. ZJU-FAST-Lab/ego-planner-swarm: An efficient single/multi ... - GitHub, accessed August 11, 2025, https://github.com/ZJU-FAST-Lab/ego-planner-swarm
19. Semi Beginner Looking to Test Build A Drone Swarm : r/diydrones - Reddit, accessed August 11, 2025, https://www.reddit.com/r/diydrones/comments/wcjwth/semi_beginner_looking_to_test_build_a_drone_swarm/
20. ZJU-FAST-Lab/Swarm-Formation: Formation Flight in Dense Environments - GitHub, accessed August 11, 2025, https://github.com/ZJU-FAST-Lab/Swarm-Formation
21. QGroundControl Quick Start | QGC Guide (4.3), accessed August 11, 2025, https://docs.qgroundcontrol.com/Stable_V4.3/en/qgc-user-guide/getting_started/quick_start.html
22. Radio Setup | QGC Guide (master), accessed August 11, 2025, https://docs.qgroundcontrol.com/master/en/qgc-user-guide/setup_view/radio.html
23. Swarm flight to using QGC - QGroundControl - PX4 Discussion Forum, accessed August 11, 2025, https://discuss.px4.io/t/swarm-flight-to-using-qgc/15865
24. Robust Planning System for Fast Autonomous Flight in Complex ..., accessed August 11, 2025, https://www.mdpi.com/2504-446X/7/3/219
25. EGO-Swarm: A Fully Autonomous and Decentralized Quadrotor Swarm System in Cluttered Environments - YouTube, accessed August 11, 2025, https://www.youtube.com/watch?v=K5WKg8meb94
26. High-Speed Motion Planning for Aerial Swarms in Unknown and Cluttered Environments - SciSpace, accessed August 11, 2025, https://scispace.com/pdf/high-speed-motion-planning-for-aerial-swarms-in-unknown-and-19051i1j06.pdf
27. Robust MADER: Decentralized and Asynchronous Multiagent Trajectory Planner Robust to Communication Delay - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/372120963_Robust_MADER_Decentralized_and_Asynchronous_Multiagent_Trajectory_Planner_Robust_to_Communication_Delay
28. FlightBench: Benchmarking Learning-based Methods for Ego-vision-based Quadrotors Navigation - arXiv, accessed August 11, 2025, https://arxiv.org/html/2406.05687v3
29. Drone Swarm Navigation in GNSS-Challenged and Cluttered Environments - Medium, accessed August 11, 2025, https://medium.com/@gwrx2005/drone-swarm-navigation-in-gnss-challenged-and-cluttered-environments-d50388bc31b3
30. (PDF) An Autonomous Drone Swarm for Detecting and Tracking Anomalies among Dense Vegetation - ResearchGate, accessed August 11, 2025, https://www.researchgate.net/publication/382271950_An_Autonomous_Drone_Swarm_for_Detecting_and_Tracking_Anomalies_among_Dense_Vegetation
31. HKUST-Aerial-Robotics/Fast-Planner: A Robust and Efficient Trajectory Planner for Quadrotors - GitHub, accessed August 11, 2025, https://github.com/HKUST-Aerial-Robotics/Fast-Planner


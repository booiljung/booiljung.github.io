# ROS2 Humble Drake
[ROS2 Humble 행동(임무) 계획 및 관리](./index.md)


현대 로봇 공학에서 시뮬레이션의 역할은 근본적으로 변화하였다. 과거의 시뮬레이션이 주로 개발된 알고리즘을 테스트하는 가상 검증 도구에 머물렀다면, 오늘날에는 알고리즘 설계, 머신러닝 데이터 생성, 시스템의 수학적 검증까지 포괄하는 개발 프로세스의 핵심 요소로 자리 잡았다.1 Toyota Research Institute(TRI)와 같은 선도적인 산업 연구소는 고품질 시뮬레이션이 로봇 공학 분야에 미칠 막대한 영향을 일찍부터 인지하고, 시뮬레이션과 현실 세계 간의 간극을 줄이는 것을 핵심 목표로 삼았다. 이러한 노력의 중심에 바로 Drake가 있다.2

본 보고서는 모델 기반 설계(Model-Based Design) 철학을 구현한 Drake와, 현대 로봇 시스템의 표준 분산형 미들웨어로 자리 잡은 ROS2(Robot Operating System 2)의 결합을 심층적으로 고찰한다. Drake는 MIT 컴퓨터 과학 및 인공지능 연구소(CSAIL)에서 시작되어 TRI의 전폭적인 지원을 통해 산업 수준의 연구 및 엔지니어링 툴킷으로 발전한 소프트웨어다.2 그 핵심은 다물체 동역학 엔진, 시스템 프레임워크, 그리고 최적화 프레임워크를 단일 애플리케이션 내에 통합하여 로봇의 동역학을 정밀하게 분석하고 제어 시스템을 구축하는 데 있다.2 반면, ROS2는 2007년에 시작된 ROS 1의 경험을 바탕으로 현대 로봇 공학의 요구사항에 맞춰 새롭게 설계된 프레임워크로서, 다양한 소프트웨어 모듈(노드) 간의 안정적이고 효율적인 통신을 지원하는 생태계의 역할을 수행한다.5

이 두 강력한 기술의 결합은 단순한 시뮬레이터 교체를 넘어선다. 이는 물리 법칙에 기반한 정밀한 '분석 및 설계' 능력(Drake)과 표준화된 '통합 및 통신' 능력(ROS2)의 융합을 의미하며, 이를 통해 제어기 설계부터 전체 시스템 통합에 이르는 일관된 개발 파이프라인 구축의 가능성을 연다. Drake와 ROS2의 통합은 각 구성 요소가 서로를 보완하고 향상시키는 공생 관계를 형성한다. ROS2는 복잡한 분산 로봇 시스템을 위한 구조적 뼈대를 제공하며, 인식, 계획, 작동 모듈 간의 통신 '배관(plumbing)'을 처리한다.6 여기에 Drake는 고충실도의 물리 기반 지능을 주입한다. 이로써 시뮬레이션은 수동적인 검증 도구에서 능동적인 

*합성(synthesis)* 도구로 변모한다. 즉, 최적의 제어 정책과 모션 계획이 시뮬레이션의 근간을 이루는 수학적 모델과의 직접적인 상호작용을 통해 생성되는 것이다.3 전통적인 ROS/Gazebo 워크플로우에서는 제어기를 외부에서 설계하고 그 동작을 '블랙박스' 시뮬레이션에서 테스트하는 반면 3, Drake/ROS2 워크플로우에서는 시뮬레이션 모델 자체가 설계 프로세스의 핵심이 된다. Drake의 최적화 프레임워크가 동역학 및 접촉 모델을 직접 탐색하여 제어 전략을 

*발견*하면, ROS2는 이 전략을 배포하고 로봇의 나머지 '두뇌'(예: 센서 데이터를 발행하는 인식 시스템)와 연결하는 수단을 제공한다. 이처럼 두 시스템의 결합은 ROS2가 전체 시스템을 구조화하고 데이터 흐름을 관리하는 동안, Drake가 그 구조 내에서 지능적이고 물리적으로 타당한 행동을 생성하는 '연산 엔진' 역할을 하는 강력한 개발 패러다임을 창출한다.

본 보고서는 먼저 Drake의 핵심 철학인 모델 기반 설계를 정의하고, 다물체 동역학, 궤적 최적화, 접촉 모델링과 같은 핵심 기술을 심층적으로 분석한다. 이어서 ROS2 Humble Hawksbill의 주요 특징과 `drake_ros` 패키지를 통한 통합 방법을 상세히 다루고, 널리 사용되는 시뮬레이터인 Gazebo와의 비교를 통해 장단점을 명확히 제시한다. 마지막으로, 실제 적용 시의 한계와 미래 전망을 논의함으로써 독자에게 포괄적인 통찰을 제공하고자 한다.


Drake의 개발 철학은 기존 로봇 시뮬레이션 도구들의 근본적인 한계에 대한 문제의식에서 출발한다. Drake는 단순한 가상 테스트 환경을 넘어, 로봇 시스템의 설계, 분석, 검증을 위한 통합된 수학적 프레임워크를 지향한다.


대부분의 시뮬레이션 도구는 '블랙박스(black box)'처럼 작동한다. 즉, 사용자는 제어 명령을 입력하고 그 결과로 센서 값을 출력받는다.3 이 방식은 시스템의 전반적인 동작을 직관적으로 확인하는 데는 유용하지만, 왜 특정 결과가 나왔는지에 대한 근본적인 원인을 파악하거나, 제어기의 안정성을 수학적으로 증명하는 데에는 명백한 한계를 가진다.

이에 반해 Drake는 '화이트박스(white-box)' 또는 '글래스박스(glass-box)' 접근 방식을 취한다. 마찰, 접촉, 공기역학과 같이 매우 복잡한 동역학 현상을 시뮬레이션하면서도, 그 기저에 있는 지배 방정식(governing equations)의 수학적 구조를 사용자에게 투명하게 노출하는 데 중점을 둔다.3 여기서 '구조'란 희소성(sparsity), 해석적 그래디언트(analytical gradients), 다항식 구조(polynomial structure) 등을 포함하며, 이러한 정보는 시뮬레이터를 단순한 검증 도구에서 강력한 분석 및 설계 도구로 격상시키는 핵심 요소가 된다.


Drake가 지배 방정식의 구조를 노출함으로써 얻어지는 이점은 다음과 같이 구체화할 수 있다.

- **제어 및 최적화**: 해석적 그래디언트를 직접 계산하여 제공할 수 있으므로, 궤적 최적화와 같은 경사도 기반 최적화 알고리즘의 수렴 속도와 정확성을 극적으로 향상시킬 수 있다.4 이는 복잡한 접촉이 포함된 문제에서도 마찬가지로, 수치적 미분에서 발생하는 오차와 계산 비용을 피할 수 있게 해준다.
- **검증 및 안정성 분석**: 시스템의 수학적 모델에 직접 접근할 수 있다는 것은, 리아푸노프(Lyapunov) 안정성 분석이나 도달 가능성 분석(reachability analysis)과 같은 정교한 형식 검증(formal verification) 기법을 적용할 수 있음을 의미한다.4 이를 통해 특정 조건에서 시스템이 실패하지 않음을 수학적으로 증명할 수 있다.
- **시스템 식별 (System Identification)**: Drake의 기호 연산(symbolic computation) 기능은 물리 엔진으로부터 직접 '집중 파라미터(lumped parameters)'와 같은 시스템의 핵심 파라미터를 자동으로 추출하는 것을 가능하게 한다.4 이는 로봇의 질량, 관성 모멘트 등 시뮬레이션 모델과 실제 로봇 간의 차이를 줄이는 시스템 식별 문제를 구조적으로 해결하는 데 매우 강력한 도구가 된다.

이러한 '화이트박스' 철학은 단순히 학문적 투명성을 추구하는 것을 넘어, 시뮬레이션과 현실 간의 간극(sim-to-real gap)을 줄이는 직접적이고 강력한 촉매제 역할을 한다. 블랙박스 시뮬레이터는 종종 시뮬레이터의 특정하고 비현실적인 물리 현상에 과적합(overfitting)된 정책을 생성하는 경향이 있다. 반면, Drake의 접근 방식은 모델의 구조를 활용하여 민감도 분석, 강인 최적화(robust optimization), 시스템 식별을 가능하게 한다.4 예를 들어, 단일 최적 궤적을 찾는 것을 넘어, 로봇의 질량 파라미터가 ±5% 변동하거나 외부의 무작위 외란이 가해져도 안정성을 유지하는 궤적을 찾도록 최적화 문제를 공식화할 수 있다. 이는 최적화 알고리즘이 모델 파라미터와 외란이 포함된 방정식의 구조에 접근하고 추론할 수 있을 때만 가능하다. 결과적으로, '화이트박스' 접근법은 설계 단계에서부터 sim-to-real 간극을 명시적으로 고려하여 강인한 제어기를 생성하는 것을 촉진하며, 값비싼 하드웨어에서의 시행착오를 줄여준다.


Drake의 강력함은 다음 세 가지 핵심 구성 요소의 유기적인 결합에서 비롯된다.2

1. **다물체 동역학 엔진 (Multibody Dynamics Engine)**: SD/FAST, Simbody와 같은 주류 시뮬레이션 패키지의 수석 아키텍트였던 Michael Sherman과 같은 전문가들이 개발을 주도하여 물리적 정확성과 수치적 안정성을 최고 수준으로 유지한다.4 이 엔진은 단순 시뮬레이션을 넘어, 순방향/역방향 기구학, 자코비안 계산, 심지어 물체 질량 중심의 그래디언트와 같은 복잡한 질의까지 지원하도록 설계되었다.
2. **시스템 프레임워크 (Systems Framework)**: Simulink에서 영감을 받은 다이어그램 기반 모델링 패러다임을 채택하였다.7 사용자는 동역학 모델, 센서, 제어기 등 다양한 구성 요소를 '시스템(System)'이라는 블록으로 정의하고, 이들의 입력(input) 포트와 출력(output) 포트를 연결하여 복잡한 전체 시스템을 구성한다. 이 방식은 시스템 내 정보의 흐름을 명확하게 시각화하고, 각 구성 요소의 재사용성과 모듈성을 극대화한다.7
3. **최적화 프레임워크 (Optimization Framework)**: 수학적 최적화 문제를 기술하기 위한 직관적인 프론트엔드(front-end)를 제공한다. 사용자는 이를 통해 최적화 문제를 쉽게 작성하고, 내부적으로는 SNOPT, IPOPT, MOSEK 등 다양한 고성능 상용 또는 오픈소스 솔버(solver)에 문제를 전달하여 해를 구한다.3 이는 MATLAB의 CVX나 Yalmip, Julia의 JuMP와 유사한 역할을 수행하며, Drake의 동역학 엔진과 긴밀하게 통합되어 있다.


Drake의 강력함은 다물체 동역학, 궤적 최적화, 그리고 접촉 모델링이라는 세 가지 핵심 기능의 정교한 통합에서 비롯된다. 이들은 개별적으로도 강력하지만, 함께 사용될 때 로봇 공학의 가장 어려운 문제들을 해결하는 통일된 계산 프레임워크를 형성한다.


Drake의 동역학 엔진은 로봇의 움직임을 지배하는 물리 법칙을 수학적으로 정밀하게 표현한다.


운동학적 루프(kinematic loop)가 없는 일반적인 로봇 조작기의 운동 방정식은 라그랑주 역학(Lagrangian mechanics)으로부터 유도되며, 다음과 같은 표준적인 형태를 가진다.10
$$
M(q)\ddot{q} + C(q, \dot{q})\dot{q} = \tau_g(q) + Bu
$$
여기서 Drake는 속도를 $\dot{q}$ 대신 `v`로 표현하는 보다 일반적인 1차 미분 방정식 형태를 내부적으로 사용하여, 반드시 2차 시스템이 아닌 경우까지 포괄한다.10
$$
M(q)\dot{v} + C(q, v)v = \tau_g(q) + Bu
$$


- $q, v$: 각각 시스템의 상태를 나타내는 일반화된 위치(generalized positions)와 속도(generalized velocities) 벡터이다.
- $M(q)$: 상태 $q$에 따라 변하는 대칭 양의 정부호(symmetric positive-definite) 관성 행렬(Inertia Matrix)이다. 시스템의 운동 에너지는 $T = \frac{1}{2}\dot{q}^T M(q) \dot{q}$로 간결하게 표현된다.10
- $C(q, v)v$: 코리올리 힘(Coriolis forces)과 구심력(centrifugal forces)을 나타내는 벡터이다. Drake에서는 $\dot{M}(q) - 2C(q, v)$가 반대칭(skew-symmetric) 행렬이 되도록 $C$를 선택하여 에너지 보존과 관련된 수치적 안정성을 높이는 속성을 활용한다.10
- $\tau_g(q)$: 중력에 의해 발생하는 일반화된 힘(generalized forces) 벡터이다.
- $B$: 액추에이터의 제어 입력 $u$를 시스템의 일반화된 힘으로 변환하는 입력 행렬이다.


Drake의 동역학 엔진은 이러한 방정식을 수치적으로 계산하는 것을 넘어, 기호 연산(Symbolic Computation)을 지원한다는 점에서 차별화된다.4 이는 방정식을 숫자 배열이 아닌 수학적 기호(symbol)로 표현하고 조작할 수 있게 해준다. 이 기능은 제어 법칙을 해석적으로 유도하거나, 시스템 파라미터에 대한 민감도를 분석하고, 구조화된 최적화 문제를 공식화하는 데 매우 강력한 도구가 된다.


궤적 최적화는 로봇이 특정 작업을 수행하기 위한 최적의 움직임을 '발견'하는 과정이다.


궤적 최적화는 주어진 시간 동안 특정 비용 함수(예: 에너지 소모, 이동 시간)를 최소화하면서, 시스템의 동역학 방정식과 다양한 제약 조건(예: 관절 한계, 장애물 회피)을 모두 만족하는 상태 $x(t)$ 및 제어 입력 $u(t)$의 시퀀스, 즉 궤적을 찾는 수학적 문제다.4 이는 상태 공간 전체를 탐색하는 대신, 시간이라는 1차원 축을 따라 해를 탐색하므로 고차원 시스템에서도 효율적으로 적용될 수 있다.13


일반적인 궤적 최적화 문제는 다음과 같이 공식화될 수 있다.12

최소화 대상:
$$
J = \ell_f(\mathbf{x}(T)) + \int_{0}^{T} \ell(\mathbf{x}(t), \mathbf{u}(t), t) dt
$$
제약 조건:
$$
\begin{align*}
\dot{\mathbf{x}}(t) &= f(\mathbf{x}(t), \mathbf{u}(t)) && \text{(System Dynamics)} \\
\mathbf{g}(\mathbf{x}(t), \mathbf{u}(t)) &\le 0 && \text{(Path Constraints)} \\
\mathbf{h}(\mathbf{x}(0), \mathbf{x}(T)) &= 0 && \text{(Boundary Constraints)}
\end{align*}
$$


Drake 생태계의 중요한 알고리즘 중 하나인 DIRCON은 Michael Posa 등에 의해 개발된 방법론으로, 접촉을 포함하는 구속된 동역학 시스템의 궤적을 최적화하는 데 특화되어 있다.1 이 방법은 접촉 순서와 타이밍까지도 최적화 변수로 취급하여, 복잡한 접촉 기반 조작 작업을 자동으로 생성할 수 있게 한다.


로봇이 환경과 상호작용하는 모든 작업은 접촉을 포함하며, 이를 정확하고 효율적으로 모델링하는 것은 시뮬레이션의 가장 큰 난제 중 하나다.


강체(rigid body) 가정과 점 접촉(point contact) 모델은 계산이 빠르다는 장점이 있지만, 실제 물리 현상과 상당한 괴리가 있다. 이러한 모델은 비결정성(indeterminate systems)이나 해가 존재하지 않는 경우(예: Painlevé paradox)와 같은 수학적 문제를 야기하여 시뮬레이션을 불안정하게 만들 수 있다.14


Drake는 이러한 한계를 극복하기 위해 Hydroelastic 접촉 모델이라는 혁신적인 접근법을 제시한다. 이는 연속체 역학(continuum mechanics)에 더 근접한 모델로, 다음과 같은 특징을 가진다.14

- **핵심 원리**: 각 물체의 표면을 강체로 보는 대신, 내부에 가상의 압력장(pressure field)을 가진 연성체로 간주한다. 두 물체가 기하학적으로 겹칠 때, 두 압력장의 값이 동일해지는 지점들의 집합으로 3차원 접촉면(contact surface)을 정의한다.14 최종 접촉력은 이 접촉면에 작용하는 압력 분포를 적분하여 계산한다.
- **장점**:
  1. **비볼록(Non-convex) 객체 처리**: 복잡한 기어 맞물림이나 손가락 파지처럼 비볼록한 형상의 객체 간 접촉도 강건하고 안정적으로 처리한다.1
  2. **부드러운 힘 계산**: 접촉 힘이 상태(위치, 자세)에 대해 부드럽게(smooth) 변한다. 이는 접촉이 발생하고 사라지는 순간에도 그래디언트가 잘 정의됨을 의미하며, 경사도 기반 최적화 알고리즘을 적용하는 데 결정적인 이점을 제공한다.18
  3. **물리적 현실성**: 점 접촉 모델이 단일 힘 벡터로 접촉을 근사하는 것과 달리, Hydroelastic 모델은 실제와 유사한 압력 분포와 접촉 패치(contact patch)를 계산하여 더 높은 물리적 현실성을 제공한다.14

이 세 가지 핵심 기능—다물체 동역학, 궤적 최적화, 접촉 모델링—은 단순한 기능의 나열이 아니라, 긴밀하게 통합된 하나의 통일된 계산 프레임워크를 형성한다. Drake의 진정한 힘은 이들의 시너지에서 나온다. **다물체 동역학** 엔진의 기호적, 해석적 특성은 **궤적 최적화** 프레임워크가 요구하는 정확한 수학적 구조를 제공한다. 그리고 진보된 **Hydroelastic 접촉 모델**은 전통적인 방법들이 실패하는 복잡하고 비볼록하며 유연한 접촉 상황에서도 이 수학적 구조가 부드럽고 잘 정의되도록 보장한다. 이 세 요소의 결합은 Drake를 단순한 시뮬레이터에서, 로봇 공학의 가장 어려운 문제 중 하나인 접촉이 풍부한 동작 계획 및 제어 문제를 해결하기 위한 강력한 계산 엔진으로 변모시킨다.


Hydroelastic 모델의 강력한 기능을 활용하기 위해서는 몇 가지 핵심 파라미터를 이해하고 적절히 설정해야 한다.14

| 파라미터 (Parameter)              | 단위 (Unit)  | 설명 (Description)                                           | 튜닝 가이드 (Tuning Guidance)                                |
| --------------------------------- | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`hydroelastic_classification`** | N/A          | 형상을 'compliant'(연성) 또는 'rigid'(강성)로 분류. Hydroelastic 접촉에 참여하려면 반드시 설정해야 함. | 연성-강성 또는 연성-연성 접촉만 가능. 강성-강성 접촉은 지원되지 않으며, `kHydroelasticWithFallback` 모델 사용 시 점 접촉으로 대체됨. |
| **`resolution_hint`**             | m            | 형상을 테셀레이션(tessellation)할 때의 목표 메시 가장자리 길이. 형상의 기하학적 복잡도를 결정함. | 값이 작을수록 접촉면 표현이 정확해지고 안정성이 향상되나, 계산 비용이 기하급수적으로 증가함. 시뮬레이션 속도와 정확도 간의 트레이드오프 관계. |
| **`hydroelastic_modulus`**        | Pa ($N/m^2$) | 재료의 강성(stiffness)을 나타내는 유효 탄성 계수. 침투 깊이에 따른 반발 압력을 결정함. | 값이 클수록 물체가 단단해지지만, 시뮬레이션 시간 간격(time step)을 매우 작게 해야 할 수 있음. 수치적 안정성을 위해 실제 재료의 영률(Young's modulus)보다 낮은 값을 사용하는 경우가 많음. |
| **`hunt_crossley_dissipation`**   | $s/m$        | 접촉 시 발생하는 에너지 손실(감쇠)을 모델링하는 계수. 접촉하는 물체의 반발 계수(coefficient of restitution)에 영향을 줌. | 값이 클수록 접촉이 더 많이 '감쇠'되어 튕김이 줄어듦. 안정적인 접촉을 위해 적절한 값을 설정하는 것이 중요하며, 일반적으로 작은 양수 값을 사용함. |
| **`coulomb_friction`**            | N/A          | 정지 마찰 계수(`static_friction`)와 동적 마찰 계수(`dynamic_friction`)를 포함하는 구조체. | `dynamic_friction` $\le$ `static_friction` 조건을 만족해야 함. 현실적인 마찰 현상을 시뮬레이션하는 데 필수적임. |


ROS2 Humble Hawksbill은 Drake와 통합될 때 시스템의 안정성과 성능을 뒷받침하는 중요한 기반을 제공한다. 특히 장기 지원(LTS) 릴리스로서의 특징과 몇 가지 핵심 기술은 주목할 만하다.


Humble Hawksbill은 2027년 5월까지 공식 지원이 보장되는 LTS 릴리스다.22 이는 장기적인 연구 프로젝트나 상용 제품 개발에 있어 매우 중요한 이점을 제공한다. 개발자는 빈번한 API 변경이나 지원 중단에 대한 걱정 없이 안정적이고 예측 가능한 플랫폼 위에서 개발에 집중할 수 있으며, 지속적인 버그 수정과 보안 업데이트를 받을 수 있다.


Type Adaptation은 ROS2 Humble의 개발자 편의성과 성능을 크게 향상시킨 핵심 기능 중 하나다.

- **개념**: ROS 메시지 타입과 애플리케이션에서 사용하는 네이티브 데이터 구조(예: C++의 `std::string`, OpenCV의 `cv::Mat`) 간의 변환을 프레임워크 수준에서 자동화하는 기능이다.22
- **기술적 설명**: 개발자는 `rclcpp::TypeAdapter` 템플릿을 특수화하여 사용자 정의 타입과 ROS 메시지 타입 간의 변환 규칙(`convert_to_ros_message`, `convert_to_custom`)을 한 번만 정의하면 된다. 이후 퍼블리셔와 서브스크라이버를 생성할 때 이 어댑터를 지정하면, ROS2 시스템이 내부적으로 데이터 변환을 처리해준다. 이는 코드의 가독성을 높이고, 개발자가 반복적인 변환 코드를 작성하는 수고를 덜어주며, 수동 변환 과정에서 발생할 수 있는 잠재적 오류를 원천적으로 차단한다.25 더 중요한 것은, 특히 프로세스 내 통신(Intra-process communication)과 결합될 때 불필요한 데이터 복사를 제거하여 제로 카피(zero-copy)에 가까운 통신을 가능하게 한다는 점이다. 이로 인해 인식(perception)과 같이 데이터 처리량이 많은 애플리케이션에서 CPU 부하를 크게 줄이고 전체 처리량을 극적으로 향상시킬 수 있다.22


CFT는 네트워크 대역폭과 CPU 자원을 효율적으로 사용하기 위한 강력한 기능이다.

- **개념**: 서브스크라이버가 특정 토픽의 모든 메시지를 무조건 수신하는 대신, 메시지의 내용(content)에 기반한 SQL과 유사한 필터링 표현식을 지정하여 원하는 조건에 맞는 메시지만을 선택적으로 수신하는 기능이다.22
- **원리 및 활용**: 이 필터링은 애플리케이션 레벨이 아닌, DDS(Data Distribution Service) 미들웨어 레벨에서 수행된다. 따라서 조건에 맞지 않는 메시지는 애초에 네트워크를 통해 전송되지 않거나, 수신 노드의 ROS 스택에 도달하기 전에 걸러진다. 이는 불필요한 데이터의 네트워크 전송, 직렬화/역직렬화, 애플리케이션 콜백 호출 등의 모든 오버헤드를 줄여준다. 예를 들어, 수십 대의 로봇이 동일한 `/robot_status` 토픽에 자신의 상태를 보고할 때, 특정 로봇의 ID(`robot_id = 'R2D2'`)를 가진 메시지만 구독하거나, 배터리 잔량이 20% 미만(`battery_level < 0.2`)인 로봇의 경고 메시지만 수신하는 등의 시나리오에서 시스템의 효율을 극적으로 향상시킬 수 있다.22


Drake로 설계된 고성능 제어기를 ROS2를 통해 실제 로봇에서 구동하기 위해서는 실시간 시스템의 제약을 이해하는 것이 필수적이다.

- **결정성(Determinism)의 중요성**: 실시간 시스템의 핵심은 단순히 빠른 속도나 낮은 지연 시간(low-latency)이 아니라, 예측 가능한 시간 내에 작업을 반드시 완료하는 결정성(determinism)이다.26 제어 루프가 1ms 주기로 실행되어야 한다면, 매번 1ms 안에 계산을 마치는 것이 보장되어야 한다.
- **주요 방해 요소**:
  1. **동적 메모리 할당 (`malloc`/`new`)**: 힙(heap) 메모리 할당은 운영체제의 내부 상태에 따라 소요 시간이 예측 불가능하며, 반복적인 할당/해제는 메모리 단편화(fragmentation)를 유발하여 장기적인 성능 저하의 원인이 된다.26
  2. **페이지 폴트 (Page Faults)**: 가상 메모리 시스템에서, 코드나 데이터가 물리적 RAM에 없고 디스크(스왑 공간)에 있을 때 페이지 폴트가 발생한다. 이 경우, 디스크 I/O가 발생하여 수 밀리초(ms)에 달하는 예측 불가능한 지연을 초래하며, 이는 하드 실시간(hard real-time) 시스템에 치명적이다.26
- **해결 전략**: 진정한 실시간 성능을 위해서는 실시간 코드 경로에서 동적 메모리 할당을 원천적으로 피해야 한다. 이를 위해 실행 전에 필요한 모든 메모리를 미리 할당(pre-allocation)하고, `mlockall`과 같은 시스템 콜을 사용하여 해당 메모리가 스왑 아웃되지 않도록 RAM에 고정하는 전략이 필요하다. 이는 STL 컨테이너의 무분별한 사용을 제한하고, 실시간 안전(real-time safe) 메모리 할당자를 사용하거나 정적 메모리 풀을 직접 관리하는 등의 특별한 프로그래밍 기법을 요구한다.26

ROS2 Humble의 실시간성 개선과 Drake의 고성능 제어 요구사항 사이에는 중요한 개념적 간극이 존재한다. ROS2의 Type Adaptation이나 향상된 DDS 성능과 같은 기능들은 평균 지연 시간을 줄이고 처리량을 높이는 *소프트 실시간(soft real-time)* 문제를 해결하는 데 중점을 둔다. 그러나 이는 수백, 수천 헤르츠로 실행되는 제어 루프에 필수적인 *하드 실시간(hard real-time)*의 결정론적 실행을 근본적으로 보장하지는 않는다. 일반적인 `rclcpp::Node` 내부에 Drake 컨트롤러를 단순히 배치하는 방식은 미션 크리티컬한 애플리케이션에는 부적합하다. 강인한 아키텍처는 '하이브리드' 접근 방식을 요구한다. 즉, 핵심 Drake 컨트롤러 로직은 별도의 실시간 안전 프로세스나 스레드에서 실행되어야 하며, 이 프로세스는 특정 CPU 코어에 고정되고 미리 할당된 메모리만 사용하도록 엄격하게 관리되어야 한다. 이 실시간 '섬(island)'은 로깅, 시각화, 고수준 명령 등을 처리하는 비실시간 ROS2 시스템과 잠금 없는(lock-free) 메시지 큐나 실시간 안전 DDS 구현을 통해 신중하게 통신해야 한다. ROS2 생태계가 전체 시스템 아키텍처를 제공하더라도, 개발자는 Drake 컨트롤러가 거주할 하드 실시간 영역을 직접 설계하고 보호해야 할 책임이 있다.


Drake의 강력한 모델링 및 최적화 기능과 ROS2의 성숙한 통신 생태계를 연결하는 다리 역할은 `drake_ros` 패키지가 수행한다. 이 패키지는 아직 실험적인 단계에 있지만, 두 세계를 통합하기 위한 구체적인 방법론과 도구를 제공한다.


`drake_ros` 저장소는 Drake와 ROS2 컴포넌트 간의 통합을 위한 API, 예제, 그리고 빌드 시스템 지원을 제공하는 것을 주된 목표로 한다.27 그 구조는 다음과 같은 핵심 요소들로 구성된다.

- `drake_ros`: Drake의 시스템 프레임워크 내에서 ROS2 통신을 처리하는 핵심 클래스와 함수들을 포함하는 API 라이브러리다.
- `drake_ros_examples`: `colcon`을 사용하여 빌드할 수 있는 예제 패키지로, 실제 사용 사례를 통해 API 활용법을 보여준다.
- `bazel_ros2_rules`: Drake의 주 빌드 시스템인 Bazel 환경에서 ROS2 워크스페이스를 통합하기 위한 Starlark 매크로와 툴링을 제공한다.
- `ros2_example_bazel_installed`: Bazel을 사용하여 Drake와 ROS2를 함께 빌드하고 실행하는 예제를 포함한다.


`drake_ros`의 가장 핵심적인 설계 철학은 ROS2 통신을 Drake의 Diagram 패러다임 안으로 자연스럽게 흡수하는 것이다. 이를 위해 특별히 설계된 `LeafSystem` 파생 클래스들을 제공한다.

- **`RosSubscriberSystem`**: 지정된 ROS2 토픽을 구독하고, 수신된 메시지를 Drake Diagram 내에서 사용할 수 있는 출력 포트(output port)로 변환하는 역할을 한다.7 이를 통해 외부 ROS2 노드(예: 카메라 드라이버, 상태 추정기)가 생성하는 데이터를 Drake 시뮬레이션이나 컨트롤러의 입력으로 손쉽게 연결할 수 있다.
- **`RosPublisherSystem`**: Drake Diagram 내의 특정 입력 포트(input port) 값을 구독하여 지정된 ROS2 토픽으로 발행(publish)하는 역할을 한다.29 이를 통해 시뮬레이션된 로봇의 상태나 제어기 출력을 RViz 시각화, 로깅 시스템 등 다른 ROS2 노드로 전달할 수 있다.

이러한 접근 방식은 전체 시뮬레이션 및 제어 시스템을 하나의 일관된 Drake Diagram으로 구성하면서, 외부 ROS2 생태계와의 인터페이스 지점을 명확하게 정의할 수 있게 해준다. 시뮬레이션 내부의 데이터 흐름과 외부 통신이 모두 Diagram의 연결(connection)로 표현되므로 시스템 전체의 구조를 이해하고 디버깅하기 용이하다.7


`drake_ros`의 복잡한 의존성(Drake, ROS2, Bazel 등)을 관리하기 위해 Docker 컨테이너 내에서 개발 환경을 구성하는 방식이 권장된다.7 이는 플랫폼 간의 차이를 없애고 일관되며 재현 가능한 빌드 환경을 제공한다.

`juraph.com` 블로그에 소개된 `kiwi_drake.cpp` 예제는 Gazebo 기반의 시뮬레이션을 Drake로 마이그레이션하는 구체적인 과정을 보여주는 좋은 사례다.7 이 예제에서 단일 C++ 실행 파일은 다음과 같은 역할을 모두 수행한다.

1. Drake의 `MultibodyPlant`와 `Simulator`를 인스턴스화하여 물리 시뮬레이션을 설정한다.
2. 웹 기반 시각화 도구인 Meshcat을 초기화한다.
3. ROS2의 `/cmd_vel` 토픽을 구독하고, 시뮬레이션된 센서 데이터를 `/joint_states`, `/imu` 등의 토픽으로 발행하는 '가상 하드웨어 인터페이스'를 구현한다.


`drake_ros`를 사용할 때 반드시 인지해야 할 가장 중요한 점은 이 프로젝트가 '안정성을 보장하지 않는다(No Stability Commitment)'는 정책이다. 이는 `drake_ros`가 Drake의 공식적인 부분이 아닌 '2자(second-party)' 프로젝트로 간주되기 때문이다.27

- **구체적 의미**:
  - API가 사전 예고나 유예 기간 없이 변경되거나 제거될 수 있다 (e.g., deprecations).
  - 버그 수정이나 새로운 기능이 이전 버전에 적용되지 않을 수 있다 (e.g., backports).
  - 개발 우선순위가 TRI Dexterous Manipulation Group의 내부 요구사항에 맞춰져 있다.27
  - Drake의 공식 릴리스 주기나 지원 플랫폼과 일치하지 않을 수 있다.
- **실무적 영향**: 이 정책은 `drake_ros`를 단순한 외부 라이브러리처럼 사용하는 것을 매우 위험하게 만든다. 이를 사용하는 개발팀은 API 변경에 능동적으로 대응해야 하며, 문제가 발생했을 때 직접 소스 코드를 분석하고 수정하거나, 특정 버전에 의존성을 고정하고 내부적으로 포크(fork)하여 관리할 준비가 되어 있어야 한다. 즉, `drake_ros`는 안정적인 '플러그 앤 플레이' 라이브러리라기보다는, 잘 만들어진 '출발점' 또는 '참조 구현'으로 간주하고 접근하는 것이 현명하다.

`drake_ros`의 'No Stability Commitment' 정책은 더 깊은 의미를 내포한다. 이는 `drake_ros`를 안정적인 라이브러리로 보기보다는, 동기적이고 중앙 집중적인 프레임워크(Drake)와 비동기적이고 분산된 프레임워크(ROS2)를 통합하기 위한 *건축학적 패턴(architectural pattern)*으로 간주해야 함을 시사한다. 이 패키지의 진정한 가치는 두 패러다임 간의 근본적인 불일치를 어떻게 해결하는지를 보여주는 예제와 `System` 구현에 있다. 사용자는 `drake_ros`를 블랙박스로 의존하려 하기보다는, 그 패턴을 학습하여 자신의 특정 애플리케이션 요구에 맞게 조정하고, 종종 ROS2 통신을 위한 자신만의 맞춤형 `LeafSystem` 래퍼를 작성해야 한다. 예를 들어, `RosSubscriberSystem`이 어떻게 ROS2 노드를 관리하고 콜백을 처리하며 데이터를 출력 포트로 내보내는지를 학습함으로써, 사용자는 다른 메시지 타입이나 QoS 정책을 처리하는 자신만의 `MyCustomRosSubscriberSystem`을 구축할 수 있다. 따라서 `drake_ros`의 지속적인 가치는 교육적이고 건축적인 측면에 있으며, 개발자가 특정 구현의 불안정성으로부터 자신을 보호하면서 외부 세계와 인터페이스하는 'Drake 방식'을 가르쳐준다.


로봇 시뮬레이터를 선택하는 것은 프로젝트의 방향과 성공에 큰 영향을 미치는 중요한 결정이다. ROS 생태계의 표준으로 자리 잡은 Gazebo와, 모델 기반 설계에 특화된 Drake는 서로 다른 철학과 강점을 가지고 있다.


- **Drake**: 'White-Box' 또는 'Glass-Box' 접근 방식을 취한다. 사용자는 Drake의 Diagram 아키텍처를 통해 시뮬레이션의 근간을 이루는 동역학 방정식의 내부 구조에 직접 접근할 수 있다.3 이 정보는 궤적 최적화나 모델 기반 제어 알고리즘을 설계하는 데 적극적으로 활용된다. 이는 제어 이론가에게 매우 친숙하고 강력한 패러다임이다.7
- **Gazebo**: 전통적으로 'Black-Box'에 가까운 아키텍처를 가진다. ROS와의 강력한 통합을 통해 센서, 액추에이터, 다중 로봇을 포함하는 복잡한 시스템 수준의 시뮬레이션을 효과적으로 지원한다. 하지만 물리 엔진의 내부 동역학 방정식에 직접 접근하여 최적화 문제에 활용하는 것은 매우 어렵거나 불가능하다.30 ROS-Gazebo 인터페이스는 편리함을 제공하지만, 때로는 느리고 불필요하게 복잡하게 느껴질 수 있으며, 내부 동작을 파악하기 어렵다는 단점이 있다.31


- **동역학 및 접촉**: Drake는 최적 제어와 정확한 접촉 모델링에 강점을 보인다.32 특히 Drake의 핵심 기술인 Hydroelastic 접촉 모델은 복잡한 형상의 객체 간 상호작용이나 연성체 접촉 시나리오에서 Gazebo의 기본 물리 엔진(예: ODE, Bullet)보다 월등히 높은 물리적 정확성과 수치적 안정성을 제공할 수 있다.1
- **그래픽스**: 그래픽 시각화는 두 시뮬레이터의 지향점이 다른 부분이다. Gazebo(특히 최신 버전인 Ignition Gazebo)는 OGRE 렌더링 엔진을 사용하여 물리 기반 렌더링(PBR)을 지원하는 등 고품질의 사실적인 그래픽을 제공한다.30 반면, Drake는 주로 Meshcat과 같은 경량 웹 기반 뷰어를 사용하여 기능적인 시각화에 중점을 둔다. 이는 그래픽적 사실성보다는 동역학 계산의 정확성과 속도를 우선시하는 Drake의 철학을 반영한다.7


- **Drake**: 다음과 같은 분야에 매우 적합하다.
  - **접촉이 풍부한 조작(Contact-rich manipulation)**: 물체를 잡고, 밀고, 조립하는 등 정밀한 접촉력 제어가 필요한 작업.
  - **로코모션(Locomotion)**: 다리 달린 로봇의 보행이나 달리기처럼 동적 안정성이 중요한 작업.
  - **모델 기반 제어 알고리즘 개발**: 모델 예측 제어(MPC), 강인 제어 등 시스템의 수학적 모델을 직접 활용하는 제어기 설계 및 검증.1
- **Gazebo**: 다음과 같은 분야에서 강점을 보인다.
  - **다중 로봇 시뮬레이션**: 여러 로봇 간의 상호작용 및 군집 로봇 알고리즘 테스트.
  - **센서 데이터 기반 알고리즘**: 카메라, 라이다 등 다양한 센서 모델을 활용한 SLAM, 내비게이션 스택 테스트.22
  - **시스템 통합 테스트**: ROS 생태계와의 긴밀한 통합을 바탕으로 전체 로봇 시스템의 각 컴포넌트가 올바르게 상호작용하는지 검증하는 데 적합하다.30


| 항목 (Category)     | Drake                                           | Gazebo (Ignition)                              |
| ------------------- | ----------------------------------------------- | ---------------------------------------------- |
| **핵심 철학**       | 모델 기반 설계 및 검증 ('White-Box')            | 시스템 수준 통합 시뮬레이션 ('Black-Box')      |
| **주요 강점**       | 궤적 최적화, 모델 기반 제어, 정확한 동역학      | ROS 생태계 통합, 다중 로봇, 센서 시뮬레이션    |
| **물리 엔진**       | 자체 고정밀 동역학 엔진                         | DART, Bullet, TPE 등 선택 가능한 플러그인 방식 |
| **접촉 모델**       | Hydroelastic 모델 (비볼록, 연성체 지원)         | 점 접촉 기반 모델 (다양한 엔진에 따라 다름)    |
| **최적화 도구**     | 내장된 강력한 최적화 프레임워크                 | 직접적인 지원 없음, 외부 도구와 연동 필요      |
| **ROS 통합 성숙도** | `drake_ros` (실험적, 'No Stability Commitment') | 매우 성숙함, ROS의 표준 시뮬레이터             |
| **그래픽스**        | Meshcat (웹 기반, 기능적 시각화에 중점)         | OGRE 2 (물리 기반 렌더링 지원, 고품질)         |
| **학습 곡선**       | 높음 (시스템 프레임워크, 제어 이론 지식 요구)   | 중간 (ROS에 익숙하면 용이하나, 내부 구조 복잡) |
| **주요 사용 사례**  | 로코모션, 접촉-풍부 조작, 제어기 설계/검증      | SLAM, 내비게이션, 시스템 통합 테스트, 교육     |


Drake와 ROS2 Humble의 조합은 강력한 잠재력을 지니고 있지만, 실제 프로젝트에 적용하기 위해서는 몇 가지 중요한 현실적 제약과 기술적 난제를 고려해야 한다.


Drake와 ROS2는 각각 독립적으로도 상당한 학습 곡선을 가진다.

- **Drake**: Drake의 Diagram 패러다임과 시스템 프레임워크는 제어 공학이나 모델 기반 설계에 익숙하지 않은 순수 소프트웨어 개발자에게는 생소하고 직관적이지 않을 수 있다.7 시스템을 상태(state), 입력(input), 출력(output)의 정보 흐름으로 추상화하여 사고하는 방식에 대한 적응이 필요하다.
- **ROS2**: ROS2 역시 초심자에게는 설치 및 설정 과정이 복잡하게 느껴질 수 있으며, 분산 시스템의 개념, 빌드 시스템(colcon), 다양한 DDS 미들웨어의 특성 등 이해해야 할 내용이 많다. 특히 "ROS 방식(the ROS way)"이라 불리는 특유의 개발 관행을 따르지 않으면 예기치 않은 문제에 부딪히기 쉽다.6


시뮬레이션에서 성공적으로 검증된 컨트롤러를 실제 로봇 하드웨어에 배포하는 과정은 이 기술 스택의 가장 큰 도전 과제 중 하나다.36 Drake 시뮬레이션 환경의 가장 큰 특징 중 하나는 모든 것이 완벽하게 동기화된 중앙 집중식 모델이라는 점이다. 시뮬레이터가 전역적인 시간(time)을 관리하며, 컨트롤러의 계산(

`CalcOutput`)은 정확한 시간 간격으로 호출되고, 플랜트의 상태는 `Context`를 통해 지연 없이 즉시 접근 가능하다.7

그러나 실제 ROS2 기반 로봇에서는 이러한 이상적인 가정들이 모두 깨진다. 컨트롤러는 독립적인 노드가 되고, 플랜트는 물리적인 로봇이 된다. 로봇의 상태는 센서 토픽을 통해 비동기적으로 전달되며, 컨트롤러의 출력은 명령 토픽을 통해 역시 비동기적으로 액추에이터에 전달된다.6 이 과정에는 예측 불가능하고 가변적인 네트워크 지연 시간이 필연적으로 발생한다.

이러한 "구조적 임피던스 불일치(architectural impedance mismatch)"는 시뮬레이션에서의 우아한 `builder.Connect()` 구문이 현실에서는 복잡하고, 시간 지연이 포함된 비동기 통신 채널로 대체됨을 의미한다. 따라서 개발자의 과제는 단순히 '컨트롤러를 ROS 노드에서 실행'하는 것이 아니라, 비동기성과 지연 시간에 강인하도록 시스템을 재설계하는 것이다. 여기에는 지연된 센서 데이터를 처리하기 위한 상태 추정기(state estimator)를 추가하거나, 타이밍 지터(jitter)에 덜 민감한 제어기를 설계하고, 서로 다른 컴포넌트 간의 타이밍과 동기화를 신중하게 관리하는 분산 시스템 엔지니어링의 문제가 포함된다.37


앞서 논의했듯이, `drake_ros`는 'No Stability Commitment' 정책하에 개발되고 있다.27 이는 API가 예고 없이 변경될 수 있음을 의미하며, 장기 프로젝트의 안정성과 유지보수성에 직접적인 위험 요소로 작용한다. 상용 제품이나 장기 연구 프로젝트에서 `drake_ros`를 사용하고자 하는 팀은, 의존성을 특정 버전에 고정하고 내부적으로 포크하여 관리하거나, 핵심적인 ROS 연동 부분은 `drake_ros`의 구현을 참고하여 자체적으로 개발하는 전략을 심각하게 고려해야 한다.


고주파 동적 제어가 필요한 애플리케이션(예: 보행 로봇)의 경우, 하드 실시간(hard real-time) 제어 루프의 구현이 필수적이다. 그러나 일반적인 ROS2 노드는 표준 리눅스 커널 위에서 실행되므로 실시간성을 보장하지 않는다.26 이 문제를 해결하기 위해서는 `ros2_control`과 같은 실시간 제어 프레임워크를 도입하거나 38, 실시간 패치가 적용된 커널(PREEMPT_RT)을 사용하고, 메모리 잠금, 동적 할당 방지, 스레드 우선순위 관리 등 실시간 프로그래밍의 원칙을 철저히 준수하는 별도의 실시간 프로세스를 설계해야 한다.


Drake와 ROS2 Humble의 조합은 현대 로봇 공학이 직면한 가장 어려운 문제, 특히 접촉이 풍부한 동적 환경에서의 지능적인 상호작용 문제를 해결하기 위한 가장 유망한 기술 스택 중 하나를 제시한다. 그러나 그 잠재력을 완전히 실현하기 위해서는 현재의 한계를 명확히 인식하고 미래 발전 방향을 모색해야 한다.


- **강점**: 물리 법칙에 기반한 정밀한 모델링과 강력한 최적화 도구를 결합하여, 기존의 방법으로는 설계하기 어려웠던 복잡하고 동적인 작업을 위한 고성능 제어기를 체계적으로 설계하고 검증할 수 있는 강력한 환경을 제공한다. ROS2 Humble의 LTS 안정성과 성숙한 통신 인프라는 이러한 고급 알고리즘을 실제 로봇 시스템에 통합하고 확장할 수 있는 견고한 기반이 된다.
- **약점**: 통합의 핵심인 `drake_ros` 패키지가 아직 실험적인 상태에 머물러 있다는 점이 가장 큰 실용적 장벽이다. 또한, Drake의 동기적/중앙집중적 시스템 패러다임과 ROS2의 비동기적/분산적 패러다임 간의 근본적인 차이에서 오는 아키텍처의 복잡성, 그리고 두 시스템 모두에 대한 높은 학습 곡선은 이 기술 스택의 넓은 채택을 가로막는 주요 요인으로 작용한다.


- **`drake_ros`의 안정화 및 공식화**: 가장 시급하고 중요한 과제는 `drake_ros`의 안정성을 높이는 것이다. 이를 위해서는 TRI의 내부 사용을 넘어 커뮤니티의 기여를 적극적으로 유치하고, 명확한 개발 로드맵과 버전 관리 정책을 수립하며, 장기적으로는 Drake의 공식 릴리스 주기와 보조를 맞추는 노력이 필요하다.40
- **생태계와의 융합**: ROS-Industrial과 같은 산업계 컨소시엄은 Tesseract와 같은 고급 모션 플래닝 프레임워크를 중심으로 표준화되고 모듈화된 로봇 애플리케이션 개발을 주도하고 있다.41 Drake 역시 이러한 생태계 내에서 '고급 동역학 및 제어 모듈'로서의 역할을 맡아, 보다 표준화되고 사용하기 쉬운 형태로 통합될 가능성이 있다. 또한, Bazel과 같은 빌드 시스템에 대한 논의는 42 ROS와 Drake 커뮤니티 간의 기술적 융합이 더 깊어질 수 있음을 시사한다.


Drake와 ROS2의 조합은 현재 전문가 수준의 깊은 지식과 상당한 엔지니어링 노력을 요구하는 기술 스택이다. 하지만 이는 접촉-풍부 조작, 동적 로코모션과 같은 차세대 로봇 기술의 핵심 난제를 해결할 수 있는 가장 원칙적이고 강력한 접근법 중 하나임이 분명하다. 이 기술 스택의 도입을 고려하는 개발팀은 `drake_ros`의 불안정성을 감수하고 내부적으로 관련 역량을 강화하는 전략을 취하거나, 커뮤니티에 적극적으로 기여하여 생태계의 성숙을 가속화하는 데 동참하는 전략을 선택해야 할 것이다. 분명한 것은, 시뮬레이션이 단순한 검증 도구를 넘어 설계 및 합성(synthesis)의 핵심 도구로 진화하는 패러다임 전환의 중심에 바로 이 조합이 있다는 사실이다.


1. Drake: Gallery - MIT, accessed August 18, 2025, https://drake.mit.edu/gallery.html
2. Drake: Model-Based Design and Verification for Robotics - CSAIL Alliances, accessed August 18, 2025, https://cap.csail.mit.edu/sites/default/files/2022-08/DrakeCaseStudy_V2_2022.pdf
3. Drake: Model-Based Design and Verification for Robotics, accessed August 18, 2025, https://drake.mit.edu/
4. Drake: Model-based design in the age of robotics and machine learning - Medium, accessed August 18, 2025, https://medium.com/toyotaresearch/drake-model-based-design-in-the-age-of-robotics-and-machine-learning-59938c985515
5. ROS 2 Documentation: Humble documentation, accessed August 18, 2025, https://docs.ros.org/en/humble/index.html
6. Trying to understand why everyone stick to ROS 2 : r/robotics - Reddit, accessed August 18, 2025, https://www.reddit.com/r/robotics/comments/1m38m5c/trying_to_understand_why_everyone_stick_to_ros_2/
7. Migrating from Gazebo to Drake - Juraph, accessed August 18, 2025, https://juraph.com/kiwi/migrating_to_drake/
8. Some Notes on Drake: A Robotic Control ToolBox | Hey There Buddo! - Philip Zucker, accessed August 18, 2025, https://www.philipzucker.com/some-notes-on-drake-a-robotic-control-toolbox/
9. Modules - Drake, accessed August 18, 2025, https://drake.mit.edu/doxygen_cxx/modules.html
10. Ch. 23 - Multi-Body Dynamics - Underactuated Robotics - MIT, accessed August 18, 2025, https://underactuated.mit.edu/multibody.html
11. template
12. Inverse Dynamics Trajectory Optimization for Contact-Implicit Model Predictive Control, accessed August 18, 2025, https://arxiv.org/html/2309.01813v3
13. Ch. 10 - Trajectory Optimization - Underactuated Robotics, accessed August 18, 2025, https://underactuated.mit.edu/trajopt.html
14. Hydroelastic Contact User Guide - Drake, accessed August 18, 2025, https://drake.mit.edu/doxygen_cxx/group__hydroelastic__user__guide.html
15. Contact Modeling in Drake, accessed August 18, 2025, https://drake.mit.edu/doxygen_cxx/group__drake__contacts.html
16. Multibody Kinematics and Dynamics - Drake, accessed August 18, 2025, https://drake.mit.edu/doxygen_cxx/group__multibody.html
17. A Transition-Aware Method for the Simulation of Compliant Contact With Regularized Friction | Request PDF - ResearchGate, accessed August 18, 2025, https://www.researchgate.net/publication/338874807_A_Transition-Aware_Method_for_the_Simulation_of_Compliant_Contact_With_Regularized_Friction
18. HydroelasticTouch: Simulation of Tactile Sensors with Hydroelastic Contact Surfaces, accessed August 18, 2025, https://www.researchgate.net/publication/388029125_HydroelasticTouch_Simulation_of_Tactile_Sensors_with_Hydroelastic_Contact_Surfaces
19. [2501.08077] HydroelasticTouch: Simulation of Tactile Sensors with Hydroelastic Contact Surfaces - arXiv, accessed August 18, 2025, https://arxiv.org/abs/2501.08077
20. [2110.04157] Velocity Level Approximation of Pressure Field Contact Patches - arXiv, accessed August 18, 2025, https://arxiv.org/abs/2110.04157
21. A pressure field model for fast, robust approximation of net contact force and moment between nominally rigid objects - arXiv, accessed August 18, 2025, https://arxiv.org/pdf/1904.11433
22. ROS 2 Humble Hawksbill Release! - Open Robotics, accessed August 18, 2025, https://www.openrobotics.org/blog/2022/5/24/ros-2-humble-hawksbill-release
23. ROS 2 Humble Hawksbill Released! - #15 by Katrin_Kellner - General - ROS Discourse, accessed August 18, 2025, https://discourse.ros.org/t/ros-2-humble-hawksbill-released/25729/15
24. New ROS2 release Humble Hawksbill - The Robot Report, accessed August 18, 2025, https://www.therobotreport.com/new-ros2-release-humble-hawksbill/
25. Humble Hawksbill (humble) — ROS 2 Documentation: Foxy ..., accessed August 18, 2025, https://docs.ros.org/en/foxy/Releases/Release-Humble-Hawksbill.html
26. Introduction to Real-time Systems - ROS 2 Design, accessed August 18, 2025, https://design.ros2.org/articles/realtime_background.html
27. RobotLocomotion/drake-ros: Experimental prototyping (for now) - GitHub, accessed August 18, 2025, https://github.com/RobotLocomotion/drake-ros
28. What is the proper way to conver ROS std_msg types into pydrake compatible values?, accessed August 18, 2025, https://stackoverflow.com/questions/79681113/what-is-the-proper-way-to-conver-ros-std-msg-types-into-pydrake-compatible-value
29. Communicating with PX4 via ROS - drake - Stack Overflow, accessed August 18, 2025, https://stackoverflow.com/questions/76669146/communicating-with-px4-via-ros
30. Robotics simulation – A comparison of two state-of-the-art solutions - ASIM GI, accessed August 18, 2025, https://www.asim-gi.org/fileadmin/user_upload_asim/ASIM_Publikationen_OA/AM180/a2033.arep.20_OA.pdf
31. RANT: Gazebo (new) is a terrible piece of software and I don't think we should recommend it to newbies : r/ROS - Reddit, accessed August 18, 2025, https://www.reddit.com/r/ROS/comments/1jqn8wb/rant_gazebo_new_is_a_terrible_piece_of_software/
32. Help choosing simulators : r/robotics - Reddit, accessed August 18, 2025, https://www.reddit.com/r/robotics/comments/z905og/help_choosing_simulators/
33. ROS-PyBullet Interface: A Framework for Reliable Contact Simulation and Human-Robot Interaction - Edinburgh Research Explorer, accessed August 18, 2025, https://www.research.ed.ac.uk/files/342776818/ROS_PyBullet_Interface_MOWER_DOA10092022_VOR_CC_BY.pdf
34. Top 10 beginner-friendly ROS projects | by Bootcamp AI - Medium, accessed August 18, 2025, https://bootcampai.medium.com/top-10-beginner-friendly-ros-projects-df7b92050257
35. Why ros2 is so frustrating? : r/ROS - Reddit, accessed August 18, 2025, https://www.reddit.com/r/ROS/comments/1horkh8/why_ros2_is_so_frustrating/
36. ros2 - Confusion on deploying Drake based software to a real robot - Stack Overflow, accessed August 18, 2025, https://stackoverflow.com/questions/71592243/confusion-on-deploying-drake-based-software-to-a-real-robot
37. What is the standard workflow for using drake with real robot? - Stack Overflow, accessed August 18, 2025, https://stackoverflow.com/questions/72934179/what-is-the-standard-workflow-for-using-drake-with-real-robot
38. Blog — ROS-Industrial, accessed August 18, 2025, https://rosindustrial.org/news
39. Project Ideas for GSoC 2022 — ROS2_Control: Galactic Aug 2025 documentation, accessed August 18, 2025, https://control.ros.org/galactic/doc/project_ideas.html
40. Should address roslaunch Support Limitations · Issue #344 · RobotLocomotion/drake-ros, accessed August 18, 2025, https://github.com/RobotLocomotion/drake-ros/issues/344
41. ROS-Industrial Roadmap Journey and a Path Forward, accessed August 18, 2025, https://rosindustrial.org/news/2025/4/30/ros-industrial-roadmap-journey-and-a-path-forward
42. Adding Bazel as an official alternative build system for ROS2 Iron? - ROS Discourse, accessed August 18, 2025, https://discourse.ros.org/t/adding-bazel-as-an-official-alternative-build-system-for-ros2-iron/28476?page=2


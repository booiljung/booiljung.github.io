# PX4의 동적 제어 할당(Dynamic Control Allocation)

## 1.  제어 할당의 개념과 필요성

### 1.1  UAV 제어 시스템의 계층적 구조

무인 항공기(Unmanned Aerial Vehicle, UAV)의 자율 비행 제어 시스템은 본질적으로 복잡한 다중 입력 다중 출력(Multi-Input Multi-Output, MIMO) 시스템이며, 그 제어 아키텍처는 일반적으로 계층적 구조(hierarchical structure)를 채택한다.1 이 구조는 전체 제어 문제를 여러 개의 하위 문제로 분해하여 설계의 복잡성을 낮추고 모듈성을 증대시키는 효과적인 접근법이다. 최상위 계층에는 임무 계획(mission planning)이나 경로 생성(path generation)이 위치하며, 그 하위 계층에 외부 루프(outer-loop)와 내부 루프(inner-loop)로 구성된 비행 제어기가 존재한다.

외부 루프 제어기는 주로 기체의 위치 및 속도 제어를 담당한다. 사용자가 설정한 웨이포인트(waypoint)나 목표 궤적을 추종하기 위해 현재 위치와 목표 위치 간의 오차를 기반으로 필요한 가속도나 속도 명령을 생성한다. 이 명령은 다시 내부 루프 제어기를 위한 목표 자세(attitude)와 총 추력(total thrust) 명령으로 변환된다.4

내부 루프 제어기는 외부 루프로부터 전달받은 목표 자세와 현재 기체의 자세(roll, pitch, yaw) 간의 오차를 최소화하는 역할을 수행한다. 이는 기체 안정성의 핵심을 담당하는 부분으로, PID(Proportional-Integral-Derivative) 제어기, LQR(Linear-Quadratic Regulator) 등 다양한 제어 기법이 적용된다.5 내부 루프 제어기의 최종 출력은 기체 전체에 가해져야 할 목표 힘(force)과 토크(torque)의 6차원 벡터, 즉 '렌치(wrench)'다. 이 렌치 벡터는 제어기가 달성하고자 하는 물리적 효과를 나타내는 추상적인 명령이므로 '가상 제어 입력(virtual control input)'이라고도 불린다.6

이러한 가상 제어 입력은 아직 물리적인 액추에이터(actuator) 명령이 아니다. 예를 들어, "양의 롤링 토크를 가하라"는 가상 제어 입력은 쿼드콥터의 경우 특정 모터들의 회전 속도를 조절해야 하고, 고정익 항공기의 경우에는 에일러론(aileron) 서보를 특정 각도로 움직여야 달성할 수 있다. 이처럼 추상적인 가상 제어 입력을 개별 액추에이터(모터, 제어 서피스 등)를 위한 구체적인 물리적 명령(예: PWM 값, RPM)으로 변환하는 과정을 '제어 할당(Control Allocation)'이라 정의한다.8 제어 할당은 제어기 설계와 액추에이터 구동 메커니즘 사이의 다리 역할을 하는 필수적인 구성 요소이다.

### 1.2  과잉 작동 시스템(Over-actuated Systems)과 제어 자유도의 문제

제어 할당 문제는 기체의 액추에이터 구성에 따라 그 성격이 달라진다. 특히 현대의 많은 고성능 UAV는 '과잉 작동 시스템(over-actuated system)'으로 설계된다.5 과잉 작동 시스템이란, 제어하고자 하는 시스템의 자유도(Degrees of Freedom, DoF)보다 독립적으로 구동 가능한 액추에이터의 수가 더 많은 시스템을 의미한다. 예를 들어, 3차원 공간에서 기체의 위치와 자세를 완전히 제어하기 위해서는 6-DoF(x, y, z 위치 및 roll, pitch, yaw 자세) 제어가 필요하다. 4개의 로터를 가진 표준적인 쿼드콥터는 4개의 제어 입력(각 로터의 추력)으로 4-DoF(수직 추력, 롤, 피치, 요)만 직접 제어할 수 있어 과잉 작동 시스템이 아니다. 반면, 6개의 로터를 가진 헥사콥터나 8개의 로터를 가진 옥토콥터는 6-DoF 제어에 대해 각각 6개와 8개의 액추에이터를 가지므로 과잉 작동 시스템에 해당한다. 틸팅이 가능한 로터를 장착한 틸트로터 VTOL(Vertical Take-Off and Landing) 역시 대표적인 과잉 작동 시스템이다.11

과잉 작동 시스템에서 제어 할당 문제는 중요한 특징을 갖는다. 주어진 가상 제어 입력(예: 특정 토크와 추력)을 생성할 수 있는 액추에이터 입력의 조합이 유일하지 않고, 수학적으로 무한히 많은 해의 집합(an infinite number of solutions)이 존재하게 된다.1 이는 제어 설계자에게 단순한 제어 목표 달성을 넘어, 추가적인 최적화 목표를 설정할 수 있는 기회를 제공한다. 예를 들어, 동일한 기동을 수행하면서도 에너지 소비를 최소화하거나, 특정 액추에이터의 포화를 방지하거나, 액추에이터 고장 시 남은 액추에이터를 활용하여 비행을 지속하는 등의 부가적인 목표를 제어 할당 단계에서 달성할 수 있다.

이러한 특성 덕분에 과잉 작동 시스템은 여러 장점을 가진다. 첫째, 입력의 여유(redundancy)를 활용하여 기존의 결합된(coupled) 위치-자세 제어에서 벗어나 독립적인 위치 및 방향 추종이 가능해져 훨씬 더 민첩한 기동(agile maneuver)을 수행할 수 있다.11 셋째, 일부 액추에이터가 고장 나더라도 나머지 액추에이터를 재구성하여 제어력을 유지하는 고장 감내 능력(fault tolerance)이 향상된다.1 셋째, 여러 액추에이터 조합 중 가장 효율적인 것을 선택함으로써 에너지 효율을 증대시킬 수도 있다. 이 모든 장점을 실현하기 위해서는 무한한 해의 집합 중에서 '최적'의 해를 실시간으로 찾아내는 정교한 제어 할당 전략이 필수적이다.

### 1.3  정적 믹싱의 한계와 동적 제어 할당의 등장

전통적인 UAV 제어 시스템에서 제어 할당은 '정적 믹싱(static mixing)' 방식을 통해 구현되었다. 정적 믹싱은 기체의 기하학적 구조에 기반하여 사전에 계산된 고정된 행렬, 즉 '믹서 행렬(mixer matrix)'을 사용하여 제어 입력을 분배하는 방식이다.8 PX4의 레거시 `.mix` 파일 시스템이나 ArduPilot의 소스 코드 내에 하드코딩된 믹서들이 이러한 정적 믹싱의 대표적인 예이다.9 이 방식은 구조가 간단하고 계산 부하가 매우 낮다는 장점이 있어 쿼드콥터와 같은 단순한 기체에 널리 사용되어 왔다.

그러나 UAV 기술이 발전하고 더욱 복잡한 기체가 등장함에 따라 정적 믹싱의 한계가 명확해졌다. 첫째, 정적 믹싱은 액추에이터의 물리적 출력 한계, 즉 포화(saturation) 현상을 체계적으로 다루기 어렵다.5 급격한 기동 시 특정 액추에이터가 최대 또는 최소 출력에 도달하면, 정적 믹서는 단순히 해당 출력을 잘라내는(clipping) 방식을 사용하는데, 이는 의도치 않은 제어 오차를 유발하고 시스템의 안정성을 해칠 수 있다.15 둘째, 정적 믹서는 비행 중 발생하는 동적인 변화에 대응할 수 없다. 예를 들어, VTOL 기체가 호버 모드에서 전진 비행 모드로 전환하는 천이(transition) 과정에서는 모터의 역할이 수직 추력 생성에서 수평 추력 생성으로 점진적으로 변해야 한다. 또한, 비행 중 로터 하나가 고장 나는 경우, 남은 로터들의 역할을 재분배해야 한다. 이러한 동적 상황 변화는 고정된 믹서 행렬로는 효과적으로 표현하고 대응하는 것이 불가능하다.9 셋째, 비대칭적이거나 특이한 구조를 가진 새로운 기체를 설정하는 과정이 매우 복잡하고 직관적이지 않다.9

이러한 한계를 극복하기 위해 등장한 개념이 바로 '동적 제어 할당(Dynamic Control Allocation)'이다. 동적 제어 할당은 비행 상황, 액추에이터 상태, 외부 환경 등 다양한 요소를 고려하여 실시간으로 제어 할당 전략을 최적화하고 변경하는 접근법이다.5 이는 더 이상 단순한 행렬 곱셈 연산에 머무르지 않고, 액추에이터의 제약 조건과 추가적인 최적화 목표를 포함하는 최적화 문제를 실시간으로 해결하는 과정을 포함한다.17

동적 제어 할당으로의 전환은 단순한 기능 개선을 넘어, UAV 설계 패러다임의 근본적인 변화를 의미한다. 초기 드론은 쿼드콥터와 같이 기하학적으로 단순하고 고정된 형태를 가졌으며, 이러한 시스템에는 정적 믹서로 충분했다.8 그러나 기술이 발전함에 따라 틸트로터 VTOL, 고장 감내형 헥사콥터, 그리고 형태가 변하는(morphing) 드론 등 복잡하고 '가변적인' 기체가 등장하기 시작했다.5 이러한 하드웨어의 가변성은 정적 믹서의 근본적인 한계를 명확히 드러냈다. 예를 들어, VTOL의 천이 과정에서 모터의 역할이 동적으로 변하는 것을 고정된 믹서 행렬 하나로 표현할 수는 없다.9 따라서, 이러한 새로운 하드웨어의 잠재력을 소프트웨어적으로 완전히 활용하기 위한 필연적인 아키텍처의 진화가 바로 동적 제어 할당인 것이다. 즉, 하드웨어의 복잡성 증가는 소프트웨어의 동적 할당 필요성을 직접적으로 유발했으며, 이는 '고정된 기체'라는 개념에서 '동적으로 변화하는 기체(dynamic airframe)'라는 개념으로의 확장을 가능하게 하는 핵심 기술이라 할 수 있다.10

## 2.  과잉 작동 시스템을 위한 제어 할당의 수학적 원

### 2.1  제어 효과 행렬(Control Effectiveness Matrix)의 정의

제어 할당 문제를 수학적으로 공식화하기 위한 첫 단계는 액추에이터의 개별 입력이 기체 전체에 미치는 영향을 모델링하는 것이다. 제어기(controller)의 출력은 기체에 가해져야 할 목표 힘과 토크의 조합인 가상 제어 입력 벡터(virtual control input vector) $\boldsymbol{\nu} \in \mathbb{R}^{d}$로 표현된다. 여기서 $d$는 제어하고자 하는 자유도(degree of freedom)의 수이며, 일반적인 6-DoF UAV의 경우 $d=6$이고 $\boldsymbol{\nu} = [\tau_x, \tau_y, \tau_z, F_x, F_y, F_z]^T$ (롤, 피치, 요 토크 및 x, y, z축 힘)가 된다. 반면, 실제 액추에이터에 가해지는 명령은 액추에이터 입력 벡터 $\mathbf{u} \in \mathbb{R}^{n}$으로 표현되며, 여기서 $n$은 독립적인 액추에이터의 총 수이다.6

두 벡터 사이의 관계는 기체의 기하학적 구조와 각 액추에이터의 물리적 특성에 의해 결정된다. 많은 경우, 이 관계는 선형 또는 선형화 가능한 시스템으로 근사할 수 있다. 이 경우, 가상 제어 입력 $\boldsymbol{\nu}$와 액추에이터 입력 $\mathbf{u}$의 관계는 다음과 같은 행렬 방정식으로 표현된다 15:
$$
\boldsymbol{\nu} = B \mathbf{u}
$$
여기서 $B \in \mathbb{R}^{d \times n}$는 '제어 효과 행렬(Control Effectiveness Matrix)'이라 불린다. 이 행렬의 각 열(column) $B_j$는 $j$번째 액추에이터가 단위 입력($u_j=1$)을 가했을 때 발생하는 힘과 토크 벡터를 나타낸다. 즉, 이 행렬은 각 액추에이터가 전체 기체의 각 제어 축(roll, pitch, yaw, thrust 등)에 얼마나 효과적으로 영향을 미치는지를 정량적으로 기술한다.15 PX4 펌웨어에서는 사용자가 QGroundControl 인터페이스를 통해 `CA_*` 접두사를 가진 파라미터들을 설정함으로써 이 제어 효과 행렬 $B$의 요소들을 간접적으로 정의하게 된다.20 예를 들어, `CA_ROTOR0_PX`, `CA_ROTOR0_PY`, `CA_ROTOR0_PZ` 파라미터는 0번 로터의 위치를, `CA_ROTOR0_AX`, `CA_ROTOR0_AY`, `CA_ROTOR0_AZ`는 추력 벡터의 방향을, 그리고 `CA_ROTOR0_KM`은 추력 대비 요 토크의 비율을 결정하여 $B$ 행렬의 해당 열을 구성하는 데 사용된다.

### 2.2  의사 역행렬(Pseudo-inverse)을 이용한 선형 제어 할당

제어 할당의 핵심 문제는 주어진 목표 가상 제어 입력 $\boldsymbol{\nu}$에 대해 $\boldsymbol{\nu} = B \mathbf{u}$를 만족하는 액추에이터 입력 $\mathbf{u}$를 찾는 것이다. 과잉 작동 시스템($n > d$)에서는 이 방정식을 만족하는 해 $\mathbf{u}$가 무한히 많이 존재한다. 이 무한한 해 중에서 하나의 특정 해를 선택하기 위한 가장 간단하고 널리 사용되는 방법은 '무어-펜로즈 의사 역행렬(Moore-Penrose pseudo-inverse)' $B^{+}$를 이용하는 것이다.8

의사 역행렬을 이용한 제어 할당 해 $\mathbf{u}_{alloc}$는 다음과 같이 계산된다 15:
$$
\mathbf{u}_{alloc} = B^{+} \boldsymbol{\nu} \quad \text{where} \quad B^{+} = B^T(BB^T)^{-1}
$$
이 해는 모든 가능한 해 중에서 액추에이터 입력 벡터의 유클리드 노름(Euclidean norm) $\|\mathbf{u}\|_2 = \sqrt{\sum u_i^2}$를 최소화하는 '최소 노름 해(minimum-norm solution)'라는 중요한 특성을 가진다. 이는 물리적으로 각 액추에이터의 출력 제곱의 합을 최소화하는 것과 같으므로, 결과적으로 에너지 소비를 줄이는 경향이 있다.23 이 방법은 선형 대수에 기반하여 해석적인 해를 제공하며, 계산이 빠르고 구현이 간단하다는 큰 장점이 있다. PX4는 내부 수학 라이브러리인 `PX4-Matrix`에 포함된 `matrix::geninv` 함수를 통해 이 의사 역행렬을 효율적으로 계산한다.24

그러나 의사 역행렬 기반의 제어 할당은 명백한 한계를 가진다. 이 방법은 액추에이터가 가질 수 있는 물리적인 제약 조건, 예를 들어 모터의 최소/최대 RPM이나 서보의 최대 각도와 같은 포화(saturation) 한계를 직접적으로 고려하지 않는다.1 따라서 계산된 $\mathbf{u}_{alloc}$의 일부 요소가 물리적으로 불가능한 값을 가질 수 있으며, 이를 단순히 잘라내는(clipping) 후처리 방식은 원치 않는 제어 오차를 발생시켜 시스템의 성능과 안정성을 저해할 수 있다.15

### 2.3  제약 조건을 고려한 최적 제어 할당

의사 역행렬 방식의 한계를 극복하기 위해, 제어 할당 문제를 제약 조건이 있는 최적화 문제(constrained optimization problem)로 공식화하는 접근법이 널리 연구되고 있다. 이 접근법의 대표적인 예가 '이차 계획법(Quadratic Programming, QP)'이다.1

QP 기반 제어 할당에서는 다음과 같은 형태의 최적화 문제를 푼다. 목적 함수(objective function)는 일반적으로 제어 할당 오차($B\mathbf{u} - \boldsymbol{\nu}$)와 제어 입력의 크기($\mathbf{u} - \mathbf{u}_{pref}$)를 최소화하도록 설정되며, 가중치 행렬 $W_{\nu}$와 $W_u$를 통해 각 항목의 중요도를 조절할 수 있다. 여기서 $\mathbf{u}_{pref}$는 선호되는 액추에이터의 기본 설정값(예: 호버 상태의 모터 RPM)이다.
$$
\min_{\mathbf{u}} \quad \| W_u (\mathbf{u} - \mathbf{u}_{pref}) \|^2_2 + \gamma \| W_{\nu} (B\mathbf{u} - \boldsymbol{\nu}) \|^2_2
$$
이 목적 함수에 액추에이터의 물리적인 출력 범위 제약을 부등식 제약 조건(inequality constraints)으로 명시적으로 추가한다:
$$
\text{subject to} \quad \mathbf{u}_{min} \le \mathbf{u} \le \mathbf{u}_{max}
$$
QP는 주어진 제약 조건을 만족하면서 목적 함수를 최소화하는 최적의 해를 보장한다. 이는 액추에이터 포화를 매우 체계적이고 효과적으로 다룰 수 있게 해준다. 그러나 QP는 반복적인 수치 계산을 필요로 하므로 의사 역행렬에 비해 계산 비용이 훨씬 높다. 이 때문에 마이크로컨트롤러와 같이 계산 자원이 제한된 실시간 임베디드 시스템에 적용하기에는 어려움이 따를 수 있다.1

또 다른 정교한 접근법은 의사 역행렬 해의 특성을 활용하는 '영공간(null-space)' 기반 최적화이다. 제어 할당 방정식의 일반해는 의사 역행렬을 이용한 특정 해(particular solution)와 영공간(null-space of B)에 속하는 동차 해(homogeneous solution)의 합으로 표현될 수 있다:
$$
\mathbf{u} = B^{+} \boldsymbol{\nu} + (I - B^{+}B)\mathbf{z}
$$
여기서 $I$는 단위 행렬이며, $\mathbf{z} \in \mathbb{R}^{n}$는 임의의 벡터이다. 영공간 투영 행렬 $(I - B^{+}B)$는 $\mathbf{z}$를 $B$의 영공간으로 투영시킨다. 영공간에 속하는 벡터는 $B$와 곱해졌을 때 0이 되므로($B(I - B^{+}B)\mathbf{z} = \mathbf{0}$), 이 항을 추가해도 최종적으로 생성되는 힘과 토크 $\boldsymbol{\nu}$에는 아무런 영향을 미치지 않는다. 즉, 이 여분의 자유도를 활용하여 추가적인 최적화 목표를 달성할 수 있다. 예를 들어, 벡터 $\mathbf{z}$를 잘 선택함으로써 액추에이터의 포화를 회피하거나, 특정 로터 간의 다운워시(downwash) 효과를 최소화하거나, 전체적인 추력 효율을 극대화하는 등의 부가적인 성능 향상을 꾀할 수 있다.1

이러한 수학적 원리들을 통해 볼 때, 제어 할당 기술의 발전은 단순한 선형 대수 문제 해결에서 복잡한 제약 최적화 문제로 진화해왔음을 알 수 있다. 의사 역행렬은 계산 효율성 덕분에 '기본값(default)' 해법으로서 여전히 중요한 위치를 차지하며, PX4의 현재 구현도 여기에 기반한다. 그러나 실제 고성능 UAV가 요구하는 정밀한 제어와 안정성을 달성하기 위한 제어 할당 기술의 진정한 가치는 '제약 조건'과 '추가적인 최적화 목표'를 어떻게 효과적으로 다루느냐에 있다. 현재 PX4의 구현은 안정성과 구현 용이성을 우선시한 실용적인 선택의 결과이지만, 학계에서 활발히 연구되는 QP나 영공간 최적화와 같은 최첨단 이론을 완전히 통합하기까지는 아직 발전의 여지가 존재한다. 이 이론과 실제 구현 사이의 간극을 메우는 것이 향후 PX4와 같은 오픈소스 오토파일럿의 중요한 기술적 과제가 될 것이다.

## 3.  PX4 동적 제어 할당 아키텍처 심층 분석

### 3.1  레거시 믹서에서 `control_allocator` 모듈로의 진화

PX4 펌웨어 버전 v1.13 이전까지 제어 할당은 정적인 텍스트 파일 기반의 '믹서(mixer)' 시스템을 통해 이루어졌다. 이 시스템은 각 기체 프레임에 대해 `.mix`라는 확장자를 가진 파일을 사용했으며, 이 파일에는 각 액추에이터가 롤, 피치, 요, 추력 제어 입력에 어떻게 반응할지를 정의하는 규칙들이 포함되어 있었다.9 이 레거시 시스템은 강력했지만 몇 가지 본질적인 한계를 가지고 있었다. 첫째, 설정 과정이 사용자 친화적이지 않고 내부 구조를 잘 아는 전문가에게만 용이했다. 둘째, 모든 설정이 비행 전에 고정되는 '정적(static)' 구성에만 국한되어, 비행 중 동적으로 변화하는 상황에 대응할 수 없었다.9

UAV 기술이 발전함에 따라 VTOL(수직이착륙기)이나 틸트로터와 같이 비행 모드에 따라 액추에이터의 역할이 극적으로 변하는 복잡한 기체들이 등장했다. 이러한 기체들은 정적 믹서 시스템으로는 효과적으로 지원하기 어려웠다. 또한, 액추에이터 고장과 같은 예기치 않은 상황에 동적으로 대응하여 안정성을 확보하려는 요구도 증가했다. 이러한 배경 속에서 PX4 v1.13 릴리즈는 제어 할당 시스템에 대한 근본적인 아키텍처 재설계를 단행했으며, 그 결과물이 바로 `control_allocator` 모듈이다.10

새롭게 도입된 동적 제어 할당 아키텍처는 다음과 같은 주요 개선점을 목표로 했다. 첫째, VTOL과 같이 여러 비행 모드를 가지는 복잡한 기체의 설정을 대폭 간소화한다. 사용자는 더 이상 복잡한 믹서 파일을 수동으로 편집할 필요 없이, QGroundControl과 같은 지상 관제 소프트웨어(GCS)를 통해 파라미터 기반으로 설정을 조절할 수 있게 되었다.10 둘째, 비행 중 제어 할당 전략을 동적으로 조정할 수 있는 기반을 마련했다. 이는 로터 손실과 같은 액추에이터 고장 상황에 대응하거나, VTOL의 천이 과정에서 제어권을 부드럽게 전환하는 등의 고급 기능을 구현할 수 있는 가능성을 열었다.9 이로써 PX4는 단순한 믹서를 넘어, 현대적인 제어 이론을 수용할 수 있는 유연하고 확장 가능한 제어 할당 프레임워크를 갖추게 되었다.

### 3.2  핵심 구성요소: `ActuatorEffectiveness`, `ControlAllocation`, `ControlAllocator` 클래스 분석

PX4의 새로운 동적 제어 할당 아키텍처는 C++의 객체 지향 설계 원칙에 따라 세 가지 핵심 클래스로 명확하게 분리되어 구현되었다. 이 세 클래스는 각각 '물리 모델', '할당 알고리즘', '실행 관리'라는 뚜렷한 역할을 담당하며, 이를 통해 코드의 모듈성, 재사용성, 확장성을 극대화한다.26

1. **`ControlAllocator` 클래스:** 이 클래스는 전체 모듈의 실행 흐름을 관리하는 최상위 '접착 코드(glue code)' 역할을 한다.26

   `control_allocator` 모듈의 메인 루프를 구성하며, uORB(PX4의 내부 통신 미들웨어)를 통해 제어기로부터 발행된 `vehicle_torque_setpoint` 및 `vehicle_thrust_setpoint` 토픽을 구독한다. 또한, 사용자가 GCS를 통해 변경하는 `CA_*` 파라미터들의 업데이트를 감지하고, 이에 따라 하위 클래스들의 인스턴스를 생성하거나 재설정하는 역할을 담당한다. 즉, 시스템의 전반적인 상태를 관리하고, 다른 두 클래스를 조율하여 실제 제어 할당 프로세스가 올바르게 실행되도록 보장한다.4

2. **`ActuatorEffectiveness` 클래스:** 이 클래스는 기체의 물리적 모델을 캡슐화하는 역할을 한다. 즉, 각 액추에이터의 기하학적 배치(위치, 방향), 물리적 특성(추력 계수, 토크 계수), 그리고 이들이 기체에 미치는 영향(힘과 토크)을 정의한다. 이 정보를 바탕으로 제2장에서 설명한 제어 효과 행렬($B$)을 계산하는 핵심적인 기능을 수행한다.26

   `ActuatorEffectiveness`는 추상 기반 클래스(abstract base class)이며, 실제 기체 유형에 따라 `ActuatorEffectivenessMultirotor`, `ActuatorEffectivenessFixedWing`, `ActuatorEffectivenessTailsitterVTOL` 등 다양한 파생 클래스가 존재한다. 이러한 상속 구조 덕분에 개발자는 새로운 유형의 기체를 지원하고자 할 때, 해당 기체의 물리적 특성을 기술하는 새로운 `ActuatorEffectiveness` 파생 클래스를 구현하기만 하면 된다.

3. **`ControlAllocation` 클래스:** 이 클래스는 실제 제어 할당 '알고리즘'을 포함한다.26

   `ActuatorEffectiveness`로부터 계산된 제어 효과 행렬 $B$를 입력받아, 이를 기반으로 의사 역행렬을 계산하고, 제어기에서 온 목표 토크와 추력에 적용한다. 또한, 계산된 액추에이터 출력이 물리적 한계를 초과할 경우 이를 처리하는 포화(saturation) 관리 로직을 수행한다. 이 클래스는 순수하게 수학적인 계산에 집중하며, 기체의 구체적인 물리적 형태에 대해서는 알지 못한다. 이러한 역할 분리는 동일한 할당 알고리즘(예: 의사 역행렬 기반)을 다양한 종류의 기체에 재사용할 수 있게 해준다.

이러한 객체 지향적 설계는 소프트웨어 공학적 관점에서 매우 우수하다. '기체의 물리적 특성'과 '할당 알고리즘'을 명확하게 분리함으로써, 한쪽의 변경이 다른 쪽에 미치는 영향을 최소화하고 독립적인 개발과 테스트를 가능하게 한다. 그러나 이러한 구조는 최종 사용자나 제어 이론 연구자에게는 다소 복잡하게 느껴질 수 있다. GitHub의 관련 이슈(issue)에서는 한 개발자가 "왜 이렇게 많은 C++ 클래스를 사용하여 제어 효과 행렬을 만드는지 이해할 수 없다"며, 행렬 요소를 직접 설정하는 더 간단한 인터페이스를 요청하기도 했다.27 이는 PX4의 아키텍처가 플랫폼의 장기적인 확장성과 유지보수성을 우선시하는 '개발자 중심'의 설계 철학을 반영하고 있음을 보여준다. 이로 인해 강력한 모듈성이 확보되었지만, 최종 사용자의 '사용 편의성' 측면에서는 일부 상충 관계(trade-off)가 발생한 것으로 분석된다.

### 3.3  uORB 메시징 파이프라인: 제어기 출력부터 액추에이터까지

PX4의 동적 제어 할당 프로세스는 uORB 메시징 시스템을 통해 여러 모듈 간의 유기적인 데이터 흐름으로 구성된다. 이 파이프라인은 제어기의 추상적인 명령이 실제 액추에이터의 물리적인 움직임으로 변환되는 전 과정을 보여준다.

1. **입력 (가상 제어 명령):** 제어 파이프라인의 시작은 내부 루프 제어기(예: `mc_rate_control`, `fw_att_control`)이다. 이 제어기들은 계산을 마친 후, 기체가 달성해야 할 목표 토크와 목표 추력을 각각 `vehicle_torque_setpoint`와 `vehicle_thrust_setpoint`라는 uORB 토픽으로 발행(publish)한다.20 이 두 토픽이 바로 제어 할당 모듈이 처리해야 할 '가상 제어 입력' 

   $\boldsymbol{\nu}$에 해당한다.

2. **처리 (`control_allocator` 모듈):** `control_allocator` 모듈은 이 두 토픽을 구독(subscribe)하여 원하는 제어 목표를 인지한다. 모듈 내부에서는 앞서 설명한 `ActuatorEffectiveness` 클래스가 현재 기체 설정 파라미터(`CA_*`)를 기반으로 제어 효과 행렬 $B$를 계산한다. 그 후 `ControlAllocation` 클래스가 이 행렬과 구독한 목표 토크/추력을 사용하여 실제 액추에이터 명령 $\mathbf{u}$를 계산한다. 이 과정에는 의사 역행렬 계산과 포화 처리 로직이 포함된다.

3. **출력 (정규화된 액추에이터 명령):** 제어 할당 계산이 완료되면, `control_allocator` 모듈은 최종 결과를 `actuator_outputs`라는 uORB 토픽으로 발행한다.20 이 토픽에 담긴 데이터는 각 액추에이터(모터, 서보 등)에 전달될 정규화된 출력값(normalized value)이다. 예를 들어, 모터의 경우 0에서 1 사이의 값, 양방향 서보의 경우 -1에서 1 사이의 값을 가진다.

4. **상태 피드백:** `control_allocator` 모듈은 할당 과정의 상태에 대한 피드백 정보도 제공한다. `ControlAllocatorStatus`라는 uORB 토픽을 통해 현재 목표 토크와 추력이 성공적으로 할당되었는지 여부, 만약 실패했다면 얼마나 많은 토크/추력이 미할당(unallocated)되었는지, 그리고 각 액추에이터가 포화 상태인지 아닌지에 대한 상세한 정보를 발행한다.20 이 정보는 로깅(logging)되거나 다른 시스템 모듈에서 비행 상태를 진단하는 데 사용될 수 있다.

5. **드라이버 레벨 (물리적 신호 변환):** 마지막으로, 하드웨어와 직접 통신하는 액추에이터 출력 드라이버(예: PWM, DShot, UAVCAN 드라이버)가 `actuator_outputs` 토픽을 구독한다. 이 드라이버는 정규화된 출력값을 실제 물리적 신호(예: 특정 듀티 사이클을 가진 PWM 신호)로 변환하여 ESC나 서보로 전송한다. 이로써 제어기의 추상적인 의도가 기체의 물리적인 움직임으로 최종 변환된다.20

이처럼 PX4의 제어 할당은 명확하게 정의된 uORB 토픽들을 통해 모듈 간의 결합도를 낮추고(decoupling), 데이터의 흐름을 명확하게 하는 현대적인 소프트웨어 아키텍처를 따르고 있다.

### 3.4  QGroundControl을 통한 설정 및 주요 파라미터(`CA_*`)의 역할

동적 제어 할당 시스템의 가장 큰 장점 중 하나는 파라미터 기반의 유연한 설정이다. 사용자는 소스 코드를 직접 수정하거나 복잡한 믹서 파일을 편집할 필요 없이, QGroundControl과 같은 GCS의 그래픽 사용자 인터페이스(GUI)를 통해 기체의 제어 할당 특성을 손쉽게 구성할 수 있다.20 QGroundControl의 'Vehicle Setup' > 'Actuators' 탭은 이러한 설정을 위한 전용 인터페이스를 제공한다.

이 인터페이스에서 설정하는 주요 파라미터들은 `CA_`라는 접두사를 가지며, `ActuatorEffectiveness` 클래스가 제어 효과 행렬 $B$를 구성하는 데 직접 사용된다. 주요 파라미터 그룹과 그 역할은 다음과 같다:

- **`CA_AIRFRAME`**: 제어 할당 시스템이 사용할 기체의 기본 프레임 유형을 지정한다. 이 파라미터 값에 따라 `control_allocator` 모듈은 적절한 `ActuatorEffectiveness` 파생 클래스(예: 멀티로터, 테일시터 VTOL)를 동적으로 로드한다. 이는 전체 제어 할당 로직을 결정하는 가장 기본적인 스위치 역할을 한다.20
- **`CA_ROTOR_COUNT`**: 기체에 장착된 로터의 총 수를 정의한다.
- **`CA_ROTORx_\*`**: 각 개별 로터(`x`는 0부터 시작하는 로터 인덱스)의 물리적 특성을 정의하는 파라미터 그룹이다.
  - `CA_ROTORx_PX`, `CA_ROTORx_PY`, `CA_ROTORx_PZ`: 기체의 무게 중심(Center of Gravity)을 기준으로 한 `x`번 로터의 3차원 위치(position)를 미터 단위로 지정한다. 이 값은 로터 추력이 각 축에 대해 생성하는 토크의 모멘트 암(moment arm)을 결정한다.
  - `CA_ROTORx_AX`, `CA_ROTORx_AY`, `CA_ROTORx_AZ`: `x`번 로터의 추력 벡터(thrust vector) 방향을 단위 벡터로 지정한다. 일반적인 멀티로터는 `[0, 0, -1]`(기체 Z축의 음수 방향, 즉 위쪽) 값을 가지며, 틸팅 로터의 경우 이 값이 동적으로 변할 수 있다.
  - `CA_ROTORx_CT`: `x`번 로터의 추력 계수(thrust coefficient)를 나타낸다. 추력과 로터 회전 속도 관계($Thrust = C_T \cdot \omega^2$)에서 사용되는 상수이다.
  - `CA_ROTORx_KM`: `x`번 로터의 토크 계수(torque coefficient) 또는 추력 대비 요(yaw) 모멘트 비율을 나타낸다. 로터 회전에 따른 반동 토크를 모델링하는 데 사용된다.20
- **`CA_SV_CSx_\*`**: 고정익이나 VTOL의 제어 서피스(control surface, `CS`)에 대한 파라미터 그룹이다.
  - `CA_SV_CSx_ROLL_SCALE`, `CA_SV_CSx_PITCH_SCALE`, `CA_SV_CSx_YAW_SCALE`: `x`번 서보가 롤, 피치, 요 제어에 각각 얼마나 기여하는지를 나타내는 스케일 팩터(scale factor)를 정의한다. 이 값들이 제어 효과 행렬의 해당 요소를 구성한다.15
- **`CA_\**_SLEW_MAX`**: 특정 액추에이터(예: 틸트 서보)의 최대 변화율(slew rate)을 제한하여, 너무 빠른 움직임으로 인한 기체의 불안정성을 방지한다.20

이러한 파라미터들을 통해 사용자는 매우 다양한 형태의 기체를 정의하고, 각 액추에이터가 기체 제어에 미치는 영향을 정밀하게 조정할 수 있다. 이는 PX4의 동적 제어 할당이 제공하는 강력한 유연성과 확장성의 핵심이다.

**Table 3-1: `control_allocator` 관련 주요 uORB 토픽 및 파라미터**

| 항목 이름 (Topic/Parameter) | 유형 (Type)                             | 발행자/구독자 (Publisher/Subscriber) | 설명 및 할당에서의 역할                                      |
| --------------------------- | --------------------------------------- | ------------------------------------ | ------------------------------------------------------------ |
| `vehicle_torque_setpoint`   | uORB Topic (vehicle_torque_setpoint_s)  | 제어기 (예: mc_rate_control)         | **입력:** 제어기가 요구하는 목표 롤, 피치, 요 토크. 제어 할당의 주요 입력 중 하나이다. |
| `vehicle_thrust_setpoint`   | uORB Topic (vehicle_thrust_setpoint_s)  | 제어기 (예: mc_pos_control)          | **입력:** 제어기가 요구하는 목표 3D 추력 벡터. 제어 할당의 또 다른 주요 입력이다. |
| `actuator_outputs`          | uORB Topic (actuator_outputs_s)         | `control_allocator`                  | **출력:** 할당 알고리즘을 통해 계산된 최종 액추에이터 출력 (정규화된 값, -1 ~ 1). |
| `ControlAllocatorStatus`    | uORB Topic (control_allocator_status_s) | `control_allocator`                  | **상태 피드백:** 할당 성공 여부, 미할당 토크/추력, 액추에이터 포화 상태 등 디버깅 및 분석을 위한 상태 정보를 제공한다. |
| `CA_AIRFRAME`               | Parameter (INT32)                       | `control_allocator` (사용)           | **구성:** 기체 유형을 선택하여 적절한 `ActuatorEffectiveness` 클래스를 로드하도록 지시한다. |
| `CA_ROTORx_*`               | Parameter (FLOAT)                       | `control_allocator` (사용)           | **구성:** 각 로터의 위치, 축, 추력/토크 계수를 정의하여 제어 효과 행렬을 구성하는 데 사용된다. |
| `CA_SV_CSx_*`               | Parameter (FLOAT)                       | `control_allocator` (사용)           | **구성:** 각 제어 서피스의 롤/피치/요에 대한 효과를 정의하여 제어 효과 행렬을 구성한다. |

## 4.  PX4 제어 할당의 핵심 알고리즘 구현

### 4.1  제어 효과 행렬의 동적 구성 및 업데이트

PX4의 동적 제어 할당 시스템의 핵심은 제어 효과 행렬 $B$를 비행 상황에 맞게 실시간으로 구성하고 업데이트하는 능력에 있다. 이 과정은 `ActuatorEffectiveness` 파생 클래스들 내부에서 이루어진다. 사용자가 QGroundControl을 통해 설정한 `CA_*` 파라미터들은 펌웨어 내부에 저장되며, `control_allocator` 모듈이 시작되거나 파라미터가 변경될 때마다 `ActuatorEffectiveness` 클래스는 이 값들을 읽어와 제어 효과 행렬을 다시 계산한다.15

예를 들어, `ActuatorEffectivenessMultirotor` 클래스는 `CA_ROTOR_COUNT` 파라미터로 로터의 수를 확인하고, 각 로터 $i$에 대해 `CA_ROTORi_PX`, `CA_ROTORi_PY`, `CA_ROTORi_PZ` 파라미터로 위치 벡터 $\mathbf{r}_i$를, `CA_ROTORi_AX`, `CA_ROTORi_AY`, `CA_ROTORi_AZ` 파라미터로 추력 방향 단위 벡터 $\hat{\mathbf{d}}_i$를 얻는다. 로터의 추력 크기는 정규화된 입력 $u_i$에 비례한다고 가정한다. 이때 $i$번째 로터가 생성하는 힘 벡터 $\mathbf{F}_i$와 토크 벡터 $\boldsymbol{\tau}_i$는 다음과 같이 모델링된다:
$$
\mathbf{F}_i = C_{T,i} \cdot u_i \cdot \hat{\mathbf{d}}_i
$$

$$
\boldsymbol{\tau}_i = \mathbf{r}_i \times \mathbf{F}_i + C_{M,i} \cdot u_i \cdot \hat{\mathbf{d}}_i
$$

여기서 $C_{T,i}$는 추력 계수(`CA_ROTORi_CT`와 관련), $C_{M,i}$는 반동 토크 계수(`CA_ROTORi_KM`과 관련)이다. 모든 로터에 대한 힘과 토크를 합산하여 전체 렌치 벡터 $\boldsymbol{\nu}$를 구성하고, 이 관계로부터 제어 효과 행렬 $B$의 $i$번째 열이 결정된다.

특히 VTOL과 같은 복합 기체에서는 이 동적 구성 능력이 더욱 중요해진다. 테일시터(Tailsitter) VTOL의 경우, 호버링 시에는 로터 추력으로 자세를 제어하지만(멀티로터와 유사), 전진 비행 시에는 동일한 액추에이터(제어 서피스)가 공력(aerodynamic force)을 발생시켜 자세를 제어한다(고정익과 유사). 따라서 비행 모드에 따라 동일한 액추에이터가 제어에 미치는 영향, 즉 제어 효과 행렬의 내용이 완전히 달라져야 한다.27 PX4는 이러한 상황을 처리하기 위해 각 비행 모드에 해당하는 별도의 `ControlAllocation` 인스턴스를 생성하고 관리하는 방식을 사용한다. `flight_mode_manager`가 비행 모드 변경을 감지하면, `control_allocator`는 현재 모드에 적합한 제어 효과 행렬을 가진 인스턴스를 활성화하여 제어 할당을 수행한다.

### 4.2  순차적 역최적화(Sequential Desaturation)를 통한 액추에이터 포화 처리

이론적으로 가장 이상적인 액추에이터 포화 처리 방법은 QP와 같은 제약 최적화 기법을 사용하는 것이지만, 이는 실시간 임베디드 시스템에는 계산 부담이 크다.1 PX4는 이러한 현실적인 제약을 고려하여, 계산적으로 매우 효율적이면서도 합리적인 성능을 제공하는 휴리스틱(heuristic) 알고리즘인 '순차적 역최적화(Sequential Desaturation)' 또는 '우선순위 기반 역최적화(Prioritized Desaturation)' 방식을 채택했다. 이 알고리즘은 중요한 제어 축(예: 롤, 피치)의 성능을 최대한 보장하면서 포화를 처리하는 데 중점을 둔다.

알고리즘의 동작 과정은 다음과 같이 여러 단계로 나눌 수 있다:

1. **초기 할당 (Initial Allocation):** 먼저 액추에이터 포화를 고려하지 않고, 의사 역행렬($B^{+}$)을 사용하여 목표 가상 제어 입력 $\boldsymbol{\nu}_{des}$를 만족시키는 초기 액추에이터 입력 $\mathbf{u}_{initial}$을 계산한다.
   $$
   \mathbf{u}_{initial} = B^{+} \boldsymbol{\nu}_{des}
   $$

2. **포화 검사 및 고정 (Saturation Check and Clipping):** 계산된 $\mathbf{u}_{initial}$의 각 요소가 물리적 한계($\mathbf{u}_{min}, \mathbf{u}_{max}$)를 벗어나는지 검사한다. 만약 한계를 벗어나는 액추에이터가 있다면, 해당 액추에이터의 출력값을 한계값으로 고정(clipping)한다. 예를 들어, $u_j > u_{j,max}$이면 $u_j = u_{j,max}$로 설정한다. 이렇게 포화되어 고정된 액추에이터들의 집합을 $S_{sat}$라고 하자.

3. **잔여 제어 목표 계산 (Residual Control Target Calculation):** 포화된 액추에이터들이 생성하는 힘과 토크 $\boldsymbol{\nu}_{sat}$를 계산한다.
   $$
   \boldsymbol{\nu}_{sat} = \sum_{j \in S_{sat}} B_j u_j
   $$
   원래의 목표 $\boldsymbol{\nu}_{des}$에서 $\boldsymbol{\nu}_{sat}$를 빼서, 아직 포화되지 않은 나머지 액추에이터들이 달성해야 할 새로운 잔여 제어 목표 $\boldsymbol{\nu}_{rem}$를 구한다.
   $$
   \boldsymbol{\nu}_{rem} = \boldsymbol{\nu}_{des} - \boldsymbol{\nu}_{sat}
   $$
   **재할당 (Re-allocation):** 포화되지 않은 액추에이터들($S_{active}$)만 사용하여 잔여 목표 $\boldsymbol{\nu}_{rem}$를 달성하도록 제어를 재할당한다. 이를 위해 $S_{active}$에 해당하는 열들로만 구성된 새로운 제어 효과 행렬 $B_{active}$를 만들고, 이의 의사 역행렬을 사용하여 $S_{active}$에 속한 액추에이터들의 새로운 입력값을 계산한다.

4. **반복 (Iteration):** 재할당된 액추에이터들 중에서 또다시 포화가 발생하는 경우가 있을 수 있다. 이 경우, 2단계부터 4단계까지의 과정을 더 이상 새로운 포화가 발생하지 않을 때까지 반복한다.

이 순차적 방식은 최적의 해를 보장하지는 않지만, 반복 횟수가 최대 액추에이터 수로 제한되어 계산 시간이 예측 가능하고 매우 빠르다. 또한, 제어 축에 우선순위를 부여하여 역최적화를 수행할 수 있다. 예를 들어, 롤/피치 토크를 최우선으로 만족시키고, 만약 포화가 발생하면 요(yaw) 토크나 추력의 일부를 희생하도록 알고리즘을 설계할 수 있다. 이는 비상 상황에서 기체의 자세 안정을 최우선으로 유지하는 데 매우 효과적이다.5 이러한 설계는 비행 제어 시스템에서 '항상 정해진 시간 안에 합리적인 답을 내는 것'이 '최적의 답을 늦게 내는 것'보다 훨씬 중요하다는 임베디드 실시간 시스템의 핵심 원칙을 충실히 따르는 실용적인 선택이라 할 수 있다.

### 4.3  `matrix::geninv`를 통한 의사 역행렬 계산 분석

PX4에서 의사 역행렬 계산의 핵심은 `PX4-Matrix` 라이브러리 내에 구현된 `geninv` 함수이다.24 이 함수의 구현을 분석해보면 PX4의 설계 철학을 엿볼 수 있다. 일반적으로 수치 해석에서 행렬의 의사 역행렬을 안정적으로 계산하기 위해 가장 널리 사용되는 방법은 '특이값 분해(Singular Value Decomposition, SVD)'이다. SVD는 수치적으로 매우 안정적이며 어떤 행렬에 대해서도 적용 가능하다는 장점이 있지만, 계산 비용이 상대적으로 높다.

반면, `matrix::geninv` 함수는 SVD 대신 '촐레스키 분해(Cholesky Decomposition)'를 기반으로 한 방법을 사용한다.24 촐레스키 분해는 대칭 양의 정부호 행렬(symmetric positive-definite matrix)에만 적용할 수 있지만, 이 조건이 만족될 경우 SVD보다 훨씬 빠르게 분해가 가능하다. 

`geninv` 함수는 입력 행렬 $G$에 대해 $G G^T$ 또는 $G^T G$를 계산하여 대칭 양의 준정부호 행렬을 만든 후, 여기에 촐레스키 분해를 적용하는 방식으로 의사 역행렬을 구한다.

이러한 구현 방식은 몇 가지 함의를 가진다. 첫째, 계산 속도를 우선시했다는 점이다. 제어 루프 내에서 반복적으로 호출되는 함수이므로, 계산 효율성은 매우 중요한 요소이다. 둘째, 수치적 안정성 측면에서는 SVD에 비해 다소 취약할 수 있다. 특히 행렬의 조건수(condition number)가 매우 크거나 거의 특이(singular)에 가까운 경우, $G G^T$를 계산하는 과정에서 수치 오차가 증폭될 수 있다. 그럼에도 불구하고 이러한 방식을 채택한 것은, 일반적인 비행 상황에서 제어 효과 행렬이 극단적으로 나쁜 조건을 가질 확률이 낮다는 공학적 판단 하에, 실시간 성능을 극대화하기 위한 트레이드오프(trade-off)의 결과로 해석할 수 있다.

### 4.4  VTOL 등 복합 기체를 위한 다중 제어 할당 인스턴스 관리

VTOL 기체는 멀티로터의 수직 이착륙 능력과 고정익의 고속 순항 능력을 결합한 형태로, 제어 할당 관점에서 매우 도전적인 과제이다. 호버 모드에서는 멀티로터와 같이 로터들의 추력 차이를 이용해 기체를 제어하는 반면, 전진 비행 모드에서는 고정익과 같이 제어 서피스(에일러론, 엘리베이터, 러더)와 주 추력 모터를 사용해 제어한다. 두 모드에서 사용되는 액추에이터 조합과 그 효과는 완전히 다르다.30

PX4의 동적 제어 할당 아키텍처는 이러한 복합 기체를 효과적으로 지원하기 위해 '다중 인스턴스 관리' 방식을 사용한다. `ControlAllocator` 모듈은 기동 시, VTOL 기체로 식별되면 각 비행 모드에 해당하는 별도의 `ControlAllocation` 객체 인스턴스를 생성한다. 예를 들어, 하나는 호버 모드를 위한 인스턴스(`_control_allocation`), 다른 하나는 전진 비행 모드를 위한 인스턴스(`_control_allocation`)를 메모리에 올려둔다.27 각 인스턴스는 해당 모드에 맞는 제어 효과 행렬과 관련 설정을 독립적으로 가진다.

비행 중 `vtol_att_control` 모듈이 기체의 상태(속도, 자세 등)를 기반으로 비행 모드 전환을 결정하면, `control_allocator`는 현재 활성화할 인스턴스를 선택한다. 예를 들어, 호버 모드에서는 `_control_allocation`의 할당 결과를 `actuator_outputs`로 발행하고, 전진 비행 모드에서는 `_control_allocation`의 결과를 발행한다.

가장 중요한 부분은 두 모드 사이의 '천이(transition)' 과정이다. 천이 중에는 두 제어 방식이 부드럽게 혼합되어야 한다. PX4는 `VT_ARSP_BLEND`와 같은 파라미터를 사용하여 천이가 진행됨에 따라 두 인스턴스의 출력에 대한 가중치를 동적으로 조절한다. 예를 들어, 전진 천이(front transition) 시 기체의 전진 속도가 증가함에 따라 호버 모드 할당 결과의 가중치는 점차 줄어들고, 전진 비행 모드 할당 결과의 가중치는 점차 늘어난다. 이 두 가중치가 적용된 결과를 합산하여 최종 액추에이터 출력을 결정함으로써, 제어권이 갑작스럽게 바뀌는 것을 방지하고 부드러운 전환을 구현한다.32 이 다중 인스턴스 관리 및 블렌딩(blending) 메커니즘은 PX4 동적 제어 할당 아키텍처가 제공하는 유연성과 강력함을 보여주는 대표적인 사례이다.

## 5.  고급 적용 사례 및 난제: 포화, 고장 감내 및 최적화

### 5.1  액추에이터 동역학(Actuator Dynamics)을 고려한 제어 할당

표준적인 제어 할당 모델은 대부분 액추에이터가 제어 명령에 즉각적으로, 그리고 무한한 대역폭(bandwidth)으로 반응한다고 가정한다.18 그러나 현실의 액추에이터, 특히 서보 모터나 대형 프로펠러를 구동하는 모터는 고유의 동역학적 특성을 가진다. 여기에는 반응 지연(time delay), 응답 속도의 한계(slew rate limit), 그리고 2차 시스템과 유사한 진동 특성 등이 포함된다.17 기체의 동역학이 상대적으로 느린 대형 항공기에서는 이러한 액추에이터 동역학을 무시할 수 있지만, 기동성이 매우 높은 소형 UAV의 경우 기체의 반응 속도와 액추에이터의 반응 속도가 비슷한 수준이 되어 그 영향을 무시할 수 없게 된다.18

액추에이터 동역학을 무시한 제어 할당은 위상 지연(phase lag)을 발생시켜 제어 루프의 안정성을 저해하고, 특히 고주파수 명령에 대한 추종 성능을 크게 떨어뜨릴 수 있다. PX4는 `CA_**_SLEW_MAX` 파라미터를 통해 액추에이터의 최대 변화율을 제한하는 기능을 제공한다.20 이는 액추에이터가 물리적으로 불가능한 속도로 움직이도록 명령받는 것을 방지하여 시스템을 보호하는 역할을 하지만, 액추에이터의 동역학적 특성을 능동적으로 보상하는 근본적인 해결책은 아니다. 이는 단순한 출력 제한에 가깝다.

이 문제를 해결하기 위한 학술적 연구는 크게 두 방향으로 진행된다. 첫 번째는 '동적 제어 할당기(dynamic control allocator)'를 설계하는 것이다. 이는 할당기 자체에 동적인 요소를 추가하여 액추에이터의 미래 상태를 예측하고, 이를 기반으로 최적의 제어 입력을 계산하는 방식이다. 모델 예측 제어(Model Predictive Control, MPC)나 receding-horizon 최적화 기법이 여기에 해당하며, 액추에이터의 상태가 목표치를 따라가도록 강제하는 추가적인 제어 루프를 할당기 내에 구성한다.17 두 번째는 제어기 설계 단계에서부터 액추에이터 동역학을 시스템 모델에 포함하여 제어 법칙을 유도하는 것이다. 이는 제어기와 할당기를 통합적으로 설계하는 접근법으로, 더 나은 성능을 기대할 수 있지만 시스템의 모듈성은 저해될 수 있다. 현재 PX4에는 이러한 고급 동역학 보상 기능이 내장되어 있지 않으며, 이는 향후 성능 향상을 위해 연구 개발이 필요한 중요한 영역으로 남아 있다.

### 5.2  다운워시(Downwash) 회피 및 추력 효율 최적화

과잉 작동 멀티로터, 특히 로터가 6개 이상인 기체나 로터들이 서로 가까이 배치된 특수 기체에서는 공기역학적 간섭 효과가 중요한 문제로 대두된다. 그중 가장 대표적인 것이 '다운워시(downwash)' 현상이다. 다운워시는 한 로터가 아래로 밀어내는 공기 흐름(하강풍)이 인접한 다른 로터의 유입 공기 흐름에 영향을 주어 해당 로터의 추력 발생 효율을 떨어뜨리는 현상을 말한다.11 이 효과는 기체의 전체적인 추력 효율을 감소시킬 뿐만 아니라, 특정 로터의 제어 효과를 예측과 다르게 만들어 제어 안정성을 심각하게 위협할 수 있다.

기존의 제어 할당 방식은 각 로터가 독립적으로 추력을 생성한다고 가정하므로 이러한 상호 간섭 효과를 전혀 고려하지 않는다. 따라서 다운워시가 심하게 발생하는 액추에이터 조합이 선택될 수 있으며, 이는 불필요한 에너지 소모와 불안정한 비행으로 이어진다.

이 문제를 해결하기 위해, 일부 연구에서는 제어 할당 프레임워크에 다운워시 모델을 통합하고 이를 회피하도록 최적화하는 방안을 제시한다.11 제2장에서 설명한 영공간(null-space) 최적화 기법이 여기서 효과적으로 사용될 수 있다. 동일한 목표 토크와 추력($\boldsymbol{\nu}$)을 생성하는 무한한 액추에이터 조합($\mathbf{u}$) 중에서, 다운워시 효과를 최소화하거나 전체 추력 효율($\eta_f$)을 최대화하는 조합을 영공간 내에서 탐색하는 것이다. 이를 위해 다운워시 발생 정도나 추력 효율을 나타내는 비용 함수(cost function)를 정의하고, 이를 최적화 문제의 목적 함수에 추가한다.
$$
\mathbf{u}_{optimal} = B^{+} \boldsymbol{\nu} + (I - B^{+}B)\mathbf{z}^*
$$

$$
\text{where} \quad \mathbf{z}^* = \arg\min_{\mathbf{z}} J_{downwash}(\mathbf{u}(\mathbf{z}))
$$

이러한 접근법은 과잉 작동 시스템의 여유 자유도를 활용하여 공기역학적 효율까지 고려한 진정한 의미의 최적 제어를 가능하게 한다.11 그러나 이를 위해서는 정확한 다운워시 모델이 필요하며, 이는 계산 유체 역학(CFD) 시뮬레이션이나 정밀한 실험을 통해 얻어야 하는 등 구현의 복잡성이 높다. 현재 PX4의 기본 제어 할당기에는 이러한 다운워시 회피나 추력 효율 최적화 기능이 포함되어 있지 않으며, 이는 고성능 과잉 작동 UAV를 위한 차세대 제어 할당 기술의 핵심 연구 주제 중 하나이다.

### 5.3  액추에이터 고장 감내(Fault Tolerance) 메커니즘 구현 방안

동적 제어 할당의 가장 큰 잠재적 이점 중 하나는 액추에이터 고장에 실시간으로 대응하여 비행 안정성을 유지하는 '고장 감내(fault tolerance)' 능력의 구현이다.1 과잉 작동 시스템은 설계상 여분의 액추에이터를 가지고 있으므로, 일부 액추에이터가 고장(예: 모터 정지, 성능 저하)나더라도 남은 정상적인 액추에이터들의 제어력을 재분배하여 임무를 지속하거나 안전하게 착륙할 수 있는 가능성을 내포하고 있다.

고장 감내 제어 할당 시스템은 일반적으로 다음과 같은 절차로 구성된다:

1. **고장 감지 및 진단 (Fault Detection and Diagnosis, FDD):** 가장 먼저 어떤 액추에이터에 어떤 유형의 고장이 발생했는지를 실시간으로 감지해야 한다. 이는 머신러닝(ML) 기반의 분류 모델을 사용하여 센서 데이터로부터 고장 패턴을 학습하여 감지하거나 34, 각 액추에이터의 명령값 대비 실제 응답(예: RPM, 전류)을 모니터링하여 예상치 못한 편차가 발생했을 때 고장으로 판단하는 분석적 방법으로 구현될 수 있다.13
2. **제어 효과 행렬의 동적 재구성:** 일단 특정 액추에이터에 고장이 발생했다고 판단되면, 제어 할당 시스템은 이 정보를 반영하여 제어 효과 행렬 $B$를 동적으로 수정해야 한다. 예를 들어, $k$번째 모터가 완전히 정지했다면, 행렬 $B$의 $k$번째 열 벡터를 모두 0으로 만들어 새로운 행렬 $B_{fault}$를 생성한다. 만약 성능이 50% 저하되었다면, 해당 열 벡터에 0.5를 곱하는 방식으로 재구성할 수 있다.
3. **제어 재할당 (Control Re-allocation):** 수정된 제어 효과 행렬 $B_{fault}$를 사용하여 남은 정상 액추에이터들만으로 목표 토크와 추력을 달성하도록 제어를 재할당한다.13 이 과정에서 기체가 가진 모든 제어 능력을 발휘하지 못할 수도 있다(예: 최대 요 토크 감소). 이 경우, 제어 가능한 범위(attainable moment set)를 재평가하고, 상위 제어기에게 현재 기체가 수행할 수 있는 기동의 한계를 알려주어 무리한 명령을 내리지 않도록 하는 것이 중요하다.

PX4 포럼의 논의에 따르면, 이러한 고장 감내 기능은 아직 PX4 펌웨어에 기본적으로 내장된 기능이 아니다.34 PX4의 동적 제어 할당 아키텍처는 제어 효과 행렬을 동적으로 변경할 수 있는 '기반'을 제공하지만, 고장을 감지하고 행렬을 어떻게 수정할지에 대한 구체적인 로직은 사용자가 `ControlAllocator.cpp`와 `ActuatorEffectiveness` 관련 코드를 직접 수정하여 구현해야 하는 고급 개발 영역에 속한다.

결론적으로, 다운워시 회피나 고장 감내와 같은 고급 제어 전략들은 동적 제어 할당이라는 프레임워크를 '필요조건'으로 하지만, 그것만으로는 충분하지 않다. 이러한 기능들을 실제로 구현하기 위해서는 제어 할당 프레임워크 위에 정교한 물리 모델(예: 공기역학 모델), 감지 알고리즘(예: FDD 시스템), 그리고 최적화 정책이 추가적으로 구축되어야만 한다. PX4의 동적 제어 할당은 이러한 고급 기능들을 구현할 수 있는 강력한 '기반(infrastructure)'을 제공한 것이며, 이를 활용하여 진정으로 지능적인 제어 시스템을 완성하는 것은 개발자와 연구 커뮤니티의 몫으로 남아있다. 이는 현재 플랫폼이 가진 잠재력과 실제 구현된 기능 간의 중요한 차이를 보여준다.

## 6.  비교 분석: ArduPilot의 제어 믹싱 접근법

### 6.1  `AP_MotorsMatrix`의 구조와 '안정성 패치(Stability Patch)' 로직

ArduPilot의 제어 할당은 `AP_Motors` 라이브러리 내의 `AP_MotorsMatrix`와 같은 C++ 클래스들을 통해 처리된다.3 PX4의 동적 제어 할당이 파라미터와 클래스 상속을 통해 기체 구조를 정의하는 것과 달리, ArduPilot은 다수의 표준적인 멀티콥터 프레임(예: 쿼드콥터의 X, + 형태, 헥사콥터, 옥토콥터의 V, H 형태 등)에 대한 믹서 로직을 C++ 소스 코드 내에 직접 하드코딩(hard-coding)하는 방식을 오랫동안 사용해왔다.37

ArduPilot 믹서의 가장 핵심적이고 독특한 특징은 '안정성 패치(Stability Patch)'라고 불리는 로직이다.3 이는 액추에이터 포화 상황을 다루기 위한 정교한 우선순위 관리 체계이다. 멀티콥터가 최대 추력과 최대 롤/피치 기동을 동시에 명령받는 등 액추에이터의 물리적 한계를 초과하는 제어 요구가 발생했을 때, 안정성 패치는 기체의 자세 안정성을 최우선으로 확보하기 위해 다른 축의 제어량을 지능적으로 희생시킨다.

구체적으로, 롤과 피치 제어에 필요한 추력 변화량이 일부 모터를 포화시키면, 안정성 패치는 요(yaw) 제어에 할당된 추력이나 전체 스로틀(throttle) 레벨을 동적으로 낮춘다. 이를 통해 롤/피치 제어를 위한 '헤드룸(headroom)'을 확보하여 자세 불안정이나 전복(flip-over)을 방지한다.14 이 로직은 단순한 순차적 역최적화를 넘어, 다년간의 비행 테스트와 개발 경험을 통해 축적된 비선형적이고 경험적인 규칙들을 포함하고 있다. 예를 들어, 스로틀이 낮을 때와 높을 때 요 제어의 우선순위를 다르게 처리하거나, 특정 모터가 포화되었을 때 나머지 모터들의 추력을 어떻게 재분배할지에 대한 복잡한 로직이 `AP_MotorsMatrix.cpp` 내에 구현되어 있다.39 이는 ArduPilot이 이론적 일반성보다는 실제 비행 상황에서의 강건성(robustness)을 매우 중요하게 생각하는 설계 철학을 가지고 있음을 보여준다.

### 6.2  Lua 스크립트를 이용한 동적 믹서의 유연성

ArduPilot은 C++ 코드에 하드코딩된 표준 믹서 외에, 사용자에게 극도의 유연성을 제공하는 강력한 도구를 갖추고 있다. 바로 Lua 스크립팅 엔진이다.8 사용자는 C++ 펌웨어를 직접 컴파일하지 않고도, Lua 스크립트 파일을 작성하여 자신만의 맞춤형 모터 믹서를 정의하고 비행 중에 동적으로 변경할 수 있다.41

ArduPilot의 스크립팅 API는 `MotorsMatrix`나 `Motors_dynamic`과 같은 객체를 제공한다. 사용자는 `add_motor_raw`나 `load_factors`와 같은 함수를 호출하여 각 모터가 롤, 피치, 요, 스로틀 제어에 기여하는 정도(즉, 제어 효과 행렬의 요소)를 직접 설정할 수 있다.41 예를 들어, `Motor_mixer_dynamic_setup.lua` 예제 스크립트는 비행 중에 기체의 기하학적 구조가 변하는 상황을 가정하고, `load_factors` 함수를 재호출하여 믹서 행렬을 실시간으로 업데이트하는 방법을 보여준다.42

이러한 스크립팅 기능은 ArduPilot의 제어 할당 접근법에 엄청난 유연성을 부여한다. 연구자나 개발자는 표준 프레임에 해당하지 않는 새로운 형태의 기체(novel frame design)를 신속하게 테스트하거나, 특정 임무에 맞춰 비행 중에 제어 특성을 바꾸는 변형 기체(transforming vehicle)를 구현할 수 있다. 이는 C++ 개발 환경에 익숙하지 않은 사용자도 고수준의 스크립트 언어를 통해 복잡한 동적 제어 할당 로직을 쉽게 구현하고 테스트할 수 있게 해준다는 점에서 매우 강력한 장점이다.

### 6.3  PX4와 ArduPilot의 제어 할당 철학 비교

PX4와 ArduPilot은 현대 UAV가 요구하는 동적 제어 할당이라는 동일한 목표를 추구하지만, 그 목표를 달성하기 위한 접근 방식과 설계 철학에는 뚜렷한 차이가 존재한다.

**PX4의 접근법:** PX4는 C++의 객체 지향 설계(상속, 다형성)를 기반으로 한 매우 구조적이고 체계적인 접근법을 취한다. `ActuatorEffectiveness`라는 추상 클래스를 정의하고, 새로운 기체는 이 클래스를 상속받는 새로운 C++ 클래스를 작성하여 지원한다.26 이 방식은 소프트웨어 공학적 관점에서 코드의 유지보수성과 확장성이 매우 뛰어나다. 제어 할당의 유연성은 `CA_*` 파라미터 설정을 통해 확보되며, 모든 로직은 컴파일된 펌웨어 내에서 결정적으로 실행된다. 이는 플랫폼의 안정성과 예측 가능성을 중시하는 설계 철학을 반영한다. 그러나 새로운 기체 타입을 추가하기 위해서는 C++ 개발 환경 설정과 펌웨어 재컴파일이 필수적이어서, 최종 사용자의 진입 장벽이 상대적으로 높을 수 있다.

**ArduPilot의 접근법:** ArduPilot은 핵심적인 믹서 로직(특히 안정성 패치)은 C++ 코드 내에 최적화하여 구현해두고, 동시에 Lua 스크립팅이라는 고수준 추상화 계층을 통해 사용자에게 극도의 유연성을 제공하는 하이브리드 방식을 채택했다.40 이 접근법은 사용자가 C++ 빌드 과정 없이도 텍스트 기반의 스크립트만으로 매우 복잡한 동적 믹서를 구현하고 신속하게 테스트할 수 있게 해준다. 이는 빠른 프로토타이핑과 실험을 중시하는 DIY 커뮤니티와 연구자들에게 매우 매력적인 특징이다.

**포화 처리 방식의 차이:** 포화 처리에서도 두 플랫폼의 철학 차이가 드러난다. PX4는 일반화된 '순차적 역최적화' 알고리즘을 사용하여 어떤 기체 구성에도 적용될 수 있는 보편적인 해법을 제공한다. 반면, ArduPilot은 다년간 축적된 비행 데이터를 바탕으로 경험적으로 튜닝된 '안정성 패치'라는 비선형 우선순위 로직을 사용하여, 특정 상황(특히 멀티콥터의 한계 기동)에서 검증된 강건성을 제공한다.14

이러한 차이점들은 두 프로젝트의 역사와 지향하는 개발자 커뮤니티의 차이에서 비롯된 결과로 해석할 수 있다. PX4는 ETH 취리히 등 학계와의 긴밀한 협력을 바탕으로 시작되어, 모듈화된 시스템 설계와 이론적 정합성을 중시하는 경향이 있다.2 C++ 클래스 기반의 아키텍처는 이러한 배경과 일치한다. 반면, ArduPilot은 DIY 드론 커뮤니티에서 출발하여 광범위한 사용자층을 지원하는 데 중점을 두어왔다.44 Lua 스크립팅은 이러한 사용자층이 쉽게 접근하고 기여할 수 있도록 하는 강력한 도구이다. 따라서 두 플랫폼의 제어 할당 방식의 차이는 단순한 기술적 선택을 넘어, 각 플랫폼이 지향하는 개발 생태계와 사용자 커뮤니티에 대한 근본적인 철학의 차이를 반영하는 것이다.

**Table 6-1: PX4와 ArduPilot의 제어 할당 방식 비교**

| 특징 (Feature)                       | PX4                                                          | ArduPilot                                                |
| ------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------------- |
| **구성 방식 (Configuration Method)** | 파라미터 기반 (QGroundControl GUI), C++ 클래스 상속          | C++ 하드코딩 (표준 프레임), Lua 스크립팅 (커스텀 프레임) |
| **핵심 로직 (Core Logic)**           | 의사 역행렬 기반 선형 할당                                   | 믹서 계수 기반 선형 할당 + 비선형 '안정성 패치'          |
| **확장성 (Extensibility)**           | `ActuatorEffectiveness` C++ 클래스를 상속하여 새로운 기체 정의 | Lua 스크립트를 통해 새로운 믹서 동적 정의 및 로드        |
| **포화 처리 (Saturation Handling)**  | 일반화된 순차적/우선순위 기반 역최적화                       | 경험적으로 튜닝된 '안정성 패치' (자세 안정성 최우선)     |
| **동적 능력 (Dynamic Capability)**   | 비행 모드에 따른 다중 인스턴스 전환 및 블렌딩                | Lua 스크립트를 통해 비행 중 믹서 행렬 실시간 변경 가능   |
| **주요 대상 사용자 (Target User)**   | 시스템 개발자, 상업용 드론 통합 업체 (구조적 접근)           | DIY 사용자, 연구자, 취미 개발자 (빠른 프로토타이핑)      |

## 7.  결론 및 향후 전망

### 7.1  PX4 동적 제어 할당의 성과와 한계 종합

본 보고서는 PX4 오토파일럿의 동적 제어 할당 시스템에 대해 심층적으로 분석하였다. 분석 결과, PX4의 동적 제어 할당은 기존의 정적 믹서 시스템이 가졌던 근본적인 한계들을 성공적으로 극복하고, 현대적인 UAV 개발에 필수적인 유연성과 확장성을 확보하는 중요한 아키텍처적 진보를 이루었음을 확인하였다.

**주요 성과:**

1. **모듈성 및 확장성 확보:** `ActuatorEffectiveness`, `ControlAllocation`, `ControlAllocator`라는 세 가지 핵심 클래스로 역할을 명확히 분리함으로써, 기체의 물리적 모델과 할당 알고리즘을 독립적으로 개발하고 확장할 수 있는 기반을 마련했다.26
2. **복잡한 기체 지원 간소화:** VTOL과 같이 여러 비행 모드를 가지는 복잡한 기체에 대한 지원을 체계화했다. 다중 인스턴스 관리와 블렌딩 메커니즘을 통해 복잡한 천이 과정을 부드럽게 처리할 수 있게 되었다.10
3. **사용자 편의성 개선:** 파라미터 기반(`CA_*`) 설정을 도입하고 QGroundControl을 통한 GUI를 제공함으로써, 사용자가 소스 코드를 직접 수정하지 않고도 다양한 기체를 구성할 수 있도록 편의성을 향상시켰다.16
4. **고급 기능 구현 기반 마련:** 액추에이터 고장 감내나 비행 중 동적 최적화와 같은 고급 제어 전략을 구현할 수 있는 소프트웨어적 기틀을 제공했다.9

명확한 한계 및 과제:

그럼에도 불구하고, 현재 PX4의 동적 제어 할당 구현은 몇 가지 명확한 한계와 향후 과제를 안고 있다.

1. **단순화된 할당 알고리즘:** 현재 핵심 할당 알고리즘은 계산적으로 효율적인 의사 역행렬과 순차적 역최적화 방식에 의존하고 있다. 이는 실시간성을 보장하는 데는 유리하지만, 액추에이터 제약 조건 하에서 진정한 최적해를 보장하지는 못한다. 학계에서 활발히 논의되는 QP(이차 계획법)나 다른 고급 최적화 기법들은 아직 통합되지 않았다.1
2. **물리 모델의 한계:** 현재의 제어 효과 모델은 다운워시와 같은 로터 간 공기역학적 간섭 효과나 액추에이터의 복잡한 동역학을 고려하지 않는다. 이로 인해 고성능이 요구되는 특정 비행 영역에서 성능 저하가 발생할 수 있다.11
3. **고급 기능의 미완성:** 고장 감내와 같은 기능은 아키텍처 수준에서는 가능성이 열려 있으나, 실제 고장을 감지하고 진단하며 제어를 재구성하는 구체적인 FDD(고장 감지 및 진단) 및 재할당 정책은 프레임워크에 내장되어 있지 않다. 이는 전적으로 사용자와 개발자의 구현 몫으로 남아 있다.34

### 7.2  향후 연구 방향 제언

PX4의 동적 제어 할당 프레임워크가 제공하는 강력한 기반 위에서, 향후 다음과 같은 방향으로 연구 및 개발이 진행된다면 UAV 제어 기술을 한 단계 더 발전시킬 수 있을 것으로 기대된다.

1. **실시간 최적화 솔버의 통합:** 임베디드 환경에 최적화된 경량 QP(Quadratic Programming) 또는 SQP(Sequential Quadratic Programming) 솔버를 제어 할당 모듈에 통합하는 연구가 필요하다. 이를 통해 액추에이터의 포화 및 변화율(slew rate) 제약을 명시적으로 다루면서 최적의 제어 해를 실시간으로 탐색할 수 있게 될 것이다. 이는 특히 한계 성능에 가깝게 기동해야 하는 상황에서 안정성과 성능을 크게 향상시킬 수 있다.
2. **머신러닝 기반 제어 할당 모델 적용:** 다운워시나 지면 효과(ground effect)와 같이 해석적으로 모델링하기 어려운 복잡한 비선형 공기역학적 효과를 고려하기 위해, 머신러닝 기법을 활용할 수 있다. 심층 신경망(Deep Neural Network)을 사용하여 방대한 시뮬레이션 또는 실제 비행 데이터로부터 최적의 제어 할당 맵을 학습시키고, 이를 경량화하여 실시간 추론 엔진 형태로 펌웨어에 탑재하는 방식이다.25 이는 기존 모델의 한계를 뛰어넘는 정밀한 제어 할당을 가능하게 할 것이다.
3. **적응형/자율형 고장 감내 시스템 구축:** 현재의 프레임워크를 확장하여 완전한 고장 감내 시스템을 구축하는 연구가 시급하다. 여기에는 온라인으로 액추에이터의 상태를 추정하고 고장을 진단하는 FDD 모듈, 고장 발생 시 남은 제어 능력을 정량적으로 평가하는 제어 가능성 분석(controllability analysis) 모듈, 그리고 평가된 능력을 바탕으로 안전한 비상 비행 경로를 생성하고 추종하는 상위 레벨의 의사결정 로직이 통합되어야 한다.13
4. **제어 및 할당의 통합 설계(Co-design):** 전통적인 계층적 제어 구조에서 벗어나, 기체 동역학, 액추에이터 동역학, 그리고 제어 할당 문제를 하나의 통합된 시스템으로 보고 제어기와 할당기를 동시에 최적화하는 통합 설계(co-design) 방법론에 대한 연구가 필요하다.17 이는 각 서브시스템 간의 상호작용을 근본적으로 고려함으로써, 전체 시스템의 성능을 극대화할 수 있는 잠재력을 가지고 있다.

요약하자면, PX4의 동적 제어 할당은 성공적인 아키텍처 혁신을 통해 미래 기술을 수용할 수 있는 훌륭한 토대를 마련했다. 이제 이 기반 위에서 더욱 지능적이고, 강건하며, 최적화된 제어 할당 알고리즘을 구현하고 통합해 나가는 것이 PX4 커뮤니티와 UAV 연구 분야 전체에 주어진 중요한 과제라 할 수 있다.

#### 참고 자료

1. Nullspace-Based Control Allocation of Overactuated UAV Platforms - ResearchGate, accessed August 12, 2025, https://www.researchgate.net/publication/353742396_Nullspace-Based_Control_Allocation_of_Overactuated_UAV_Platforms
2. An In-depth Look at the Multicopter Control System Architecture - PX4 Developer Summit Virtual 2020 - YouTube, accessed August 12, 2025, https://www.youtube.com/watch?v=nEo4WGl4Lgc
3. Copter Attitude Control - Dev documentation - ArduPilot, accessed August 12, 2025, https://ardupilot.org/dev/docs/apmcopter-programming-attitude-control-2.html
4. Modules Reference: Controller | PX4 Guide (main), accessed August 12, 2025, https://docs.px4.io/main/en/modules/modules_controller
5. Saturation Resolving Control Allocation Design for an Over-actuated VTOL Drone, accessed August 12, 2025, https://jast.hho.msu.edu.tr/index.php/JAST/article/view/625
6. A Survey of Optimal Control Allocation for Aerial Vehicle Control - ResearchGate, accessed August 12, 2025, https://www.researchgate.net/publication/372327140_A_Survey_of_Optimal_Control_Allocation_for_Aerial_Vehicle_Control
7. A Survey of Optimal Control Allocation for Aerial Vehicle Control - MDPI, accessed August 12, 2025, https://www.mdpi.com/2076-0825/12/7/282
8. On the Control Allocation of Fully-Actuated and Over-Actuated Multirotor UAVs R. (Rens) Werink - University of Twente Student Theses, accessed August 12, 2025, https://essay.utwente.nl/77102/1/Werink_MA_EEMCS.pdf
9. Control Allocation: reworking the PX4 mixing system - PX4 Developer Summit Virtual 2020, accessed August 12, 2025, https://www.youtube.com/watch?v=xjLM9whwjO4
10. The New Dynamic Mixing System of PX4 - Beat Küng & Silvan Fuhrer, Auterion - YouTube, accessed August 12, 2025, https://www.youtube.com/watch?v=NQwepGLMzZ4
11. Downwash-aware Control Allocation for Over-actuated UAV Platforms - Hangxin Liu, accessed August 12, 2025, https://liuhx111.github.io/data/paper/IROS22_Downwash.pdf
12. [2207.09645] Downwash-aware Control Allocation for Over-actuated UAV Platforms - arXiv, accessed August 12, 2025, https://arxiv.org/abs/2207.09645
13. A Physics-Based Fault Tolerance Mechanism for UAVs' Flight Controller - ResearchGate, accessed August 12, 2025, https://www.researchgate.net/publication/379234778_A_Physics-Based_Fault_Tolerance_Mechanism_for_UAVs'_Flight_Controller
14. Using the PX4 mixer infrastructure with ArduPilot - Google Groups, accessed August 12, 2025, https://groups.google.com/g/drones-discuss/c/wV10ZUyLumU
15. Control allocation roll/pitch torque effectiveness actual meaning - PX4 Autopilot, accessed August 12, 2025, https://discuss.px4.io/t/control-allocation-roll-pitch-torque-effectiveness-actual-meaning/37070
16. PX4 Autopilot Release v1.13 Brings Dynamic Control Allocation, Expanded Hardware Support, and More, accessed August 12, 2025, https://px4.io/px4-autopilot-release-v1-13-brings-dynamic-control-allocation-expanded-hardware-support-and-more/
17. On the Actuator Dynamics of Dynamic Control Allocation for a Small Fixed-Wing UAV with Direct Lift Control - PolyU Scholars Hub, accessed August 12, 2025, https://research.polyu.edu.hk/en/publications/on-the-actuator-dynamics-of-dynamic-control-allocation-for-a-smal
18. On the Actuator Dynamics of Dynamic Control ... - UCL Discovery, accessed August 12, 2025, https://discovery.ucl.ac.uk/10188637/1/Yan_Final_extracted.pdf
19. Control allocation roll/pitch torque effectiveness actual meaning - PX4 Discussion Forum, accessed August 12, 2025, https://discuss.px4.io/t/control-allocation-roll-pitch-torque-effectiveness-actual-meaning/37070/4
20. Actuator Configuration and Testing | PX4 Guide (main) - PX4 docs, accessed August 12, 2025, https://docs.px4.io/main/en/config/actuators.html
21. [Bug] Singularities in control allocation's pseudo inverse / Issue #23606 / PX4/PX4-Autopilot, accessed August 12, 2025, https://github.com/PX4/PX4-Autopilot/issues/23606
22. Generic Control Allocation Toolbox for Preliminary Vehicle Design, accessed August 12, 2025, https://ntrs.nasa.gov/api/citations/20190000993/downloads/20190000993.pdf
23. Handling Qualities Considerations in Control Allocation for Multicopters - NASA Technical Reports Server (NTRS), accessed August 12, 2025, [https://ntrs.nasa.gov/api/citations/20220004968/downloads/1548_Malpica%20_Withrow_041422.pdf](https://ntrs.nasa.gov/api/citations/20220004968/downloads/1548_Malpica _Withrow_041422.pdf)
24. PX4-Matrix/matrix/PseudoInverse.hpp at master - GitHub, accessed August 12, 2025, https://github.com/PX4/PX4-Matrix/blob/master/matrix/PseudoInverse.hpp
25. Control Allocation Strategy Based on Min–Max Optimization and Simple Neural Network, accessed August 12, 2025, https://www.mdpi.com/2504-446X/9/5/372
26. Dynamic Control Allocation/Mixer / Issue #18556 / PX4/PX4-Autopilot - GitHub, accessed August 12, 2025, https://github.com/PX4/PX4-Autopilot/issues/18556
27. How to migrate my DFUAV project to v.1.15.4 by config control allocator. #25404 - GitHub, accessed August 12, 2025, https://github.com/PX4/PX4-Autopilot/issues/25404
28. About allocator / Issue #20966 / PX4/PX4-Autopilot - GitHub, accessed August 12, 2025, https://github.com/PX4/PX4-Autopilot/issues/20966
29. Document Control Allocation concept for Developers / Issue #19939 / PX4/PX4-Autopilot, accessed August 12, 2025, https://github.com/PX4/PX4-Autopilot/issues/19939
30. Actuators | PX4 User Guide - GitBook, accessed August 12, 2025, https://px4.gitbook.io/px4-user-guide/config/actuators
31. Understanding `MC_YAW_WEIGHT` - Discussion Forum for PX4, Pixhawk, QGroundControl, MAVSDK, MAVLink, accessed August 12, 2025, https://discuss.px4.io/t/understanding-mc-yaw-weight/43272
32. Generic Standard VTOL (QuadPlane) Configuration & Tuning | PX4 ..., accessed August 12, 2025, https://docs.px4.io/main/en/config_vtol/vtol_quad_configuration
33. Downwash-aware Control Allocation for Over-actuated UAV Platforms - ResearchGate, accessed August 12, 2025, https://www.researchgate.net/publication/363174040_Downwash-aware_Control_Allocation_for_Over-actuated_UAV_Platforms
34. Custom ControlAllocation logic in case of Rotor failure - PX4 Autopilot, accessed August 12, 2025, https://discuss.px4.io/t/custom-controlallocation-logic-in-case-of-rotor-failure/42077
35. Motor Mixers Block - Copter 3.5 - ArduPilot Discourse, accessed August 12, 2025, https://discuss.ardupilot.org/t/motor-mixers-block/27912
36. CoCalc -- AP_MotorsMatrix.cpp, accessed August 12, 2025, https://cocalc.com/github/Ardupilot/ardupilot/blob/master/libraries/AP_Motors/AP_MotorsMatrix.cpp
37. libraries/AP_Motors / d277b6cabd1ad93f78a67d6f765fa20f0010cc59 / ACS / ardupilot - NPS GitLab, accessed August 12, 2025, https://gitlab.nps.edu/acs/ardupilot/-/tree/d277b6cabd1ad93f78a67d6f765fa20f0010cc59/libraries/AP_Motors
38. What setup should be used for this type of drone? - Copter 3.5 - ArduPilot Discourse, accessed August 12, 2025, https://discuss.ardupilot.org/t/what-setup-should-be-used-for-this-type-of-drone/27788
39. ardupilot/libraries/AP_Motors/AP_MotorsMatrix.cpp at master - GitHub, accessed August 12, 2025, https://github.com/ArduPilot/ardupilot/blob/master/libraries/AP_Motors/AP_MotorsMatrix.cpp
40. GSOC 2022 - Custom Controller Implementation Update - Adding New Controller By Hand Coding - Blog - ArduPilot Discourse, accessed August 12, 2025, https://discuss.ardupilot.org/t/gsoc-2022-custom-controller-implementation-update-adding-new-controller-by-hand-coding/89304
41. ardupilot/libraries/AP_Scripting/examples/MotorMatrix_setup.lua at master - GitHub, accessed August 12, 2025, https://github.com/ArduPilot/ardupilot/blob/master/libraries/AP_Scripting/examples/MotorMatrix_setup.lua
42. ardupilot/libraries/AP_Scripting/examples ... - GitHub, accessed August 12, 2025, https://github.com/ArduPilot/ardupilot/blob/master/libraries/AP_Scripting/examples/Motor_mixer_dynamic_setup.lua
43. Custom motor mixing calculator - ArduCopter - ArduPilot Discourse, accessed August 12, 2025, https://discuss.ardupilot.org/t/custom-motor-mixing-calculator/38035
44. Welcome to the ArduPilot Development Site - Dev documentation, accessed August 12, 2025, https://ardupilot.org/dev/


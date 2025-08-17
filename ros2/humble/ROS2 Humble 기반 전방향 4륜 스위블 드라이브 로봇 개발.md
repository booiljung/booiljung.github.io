# ROS2 Humble 기반 전방향 4륜 스위블 드라이브 로봇 개발에 대한 고찰

## 서론: 전방향 이동 로봇과 스위블 드라이브의 이해

### 0.1 홀로노믹(Holonomic) 구동계의 개념과 로보틱스에서의 중요성

홀로노믹 시스템(Holonomic System)은 구속 조건에 얽매이지 않고 시스템이 가진 모든 자유도(Degrees of Freedom, DoF) 방향으로 즉각적인 이동이 가능한 물리 시스템을 지칭한다.1 로봇 공학에서 이는 로봇의 현재 상태(위치 및 방향)가 일반화된 좌표계(generalized coordinates)로 완벽하게 기술될 수 있으며, 로봇이 방향 전환을 위한 별도의 준비 동작 없이 평면상의 모든 방향으로 병진 및 회전 운동을 동시에 수행할 수 있음을 의미한다.1

이러한 특성은 로봇의 기동성을 극대화하여, 특히 창고, 공장, 병원과 같이 복잡하고 좁은 공간에서 운용되는 서비스 로봇이나 물류 로봇에 필수적인 능력으로 간주된다.3 비홀로노믹 시스템, 예를 들어 일반적인 자동차는 전진 및 후진, 그리고 조향을 통한 회전만 가능하며 측면으로 직접 이동할 수 없다. 이와 대조적으로 홀로노믹 로봇은 평행 주차와 같은 복잡한 기동을 즉시 수행할 수 있어 작업 효율성과 공간 활용도를 비약적으로 향상시킨다.1

그러나 이러한 탁월한 유연성은 기술적 과제를 동반한다. 홀로노믹 구동계는 일반적으로 더 많은 액추에이터와 정교한 제어 알고리즘을 요구하며, 이는 시스템의 복잡성, 제작 비용, 그리고 제어 난이도를 높이는 주요 원인이 된다.1 또한, 일부 홀로노믹 구동 방식은 구조적 한계로 인해 기존 비홀로노믹 시스템에 비해 견인력이나 최고 속도에서 손해를 볼 수 있다.2

### 0.2 스위블 드라이브(Swerve Drive)의 작동 원리 및 특성

스위블 드라이브는 각 휠 모듈이 독립적인 구동(driving) 모터와 조향(steering) 모터를 각각 탑재한 구조를 가진다.7 이 설계를 통해 각 휠이 생성하는 힘 벡터의 방향($\phi_i$)과 크기($v_i$)를 개별적으로, 그리고 실시간으로 제어할 수 있다. 로봇은 이 네 개의 힘 벡터를 적절히 조합하여 원하는 방향의 병진 운동과 제자리 회전 운동을 자유롭게 구현할 수 있다.4

스위블 드라이브는 다른 대표적인 전방향 구동계인 메카넘 휠(Mecanum Wheel)이나 옴니 휠(Omni Wheel)과 비교했을 때 뚜렷한 장단점을 가진다. 가장 큰 차이점은 **견인력(Traction)**에 있다. 메카넘 휠과 옴니 휠은 휠의 원주를 따라 배치된 작은 롤러들이 미끄러지며 측면 이동을 구현하는 원리다.10 이 구조는 필연적으로 휠과 지면 사이의 접촉 면적을 감소시키고 슬립을 유발하여 견인력 손실을 야기한다.2 따라서 이러한 로봇들은 다른 로봇과의 물리적 상호작용이 발생하는 상황에서 쉽게 밀려나거나, 고르지 못한 노면에서 성능이 저하되는 경향이 있다.6 반면, 스위블 드라이브는 일반적인 고마찰 타이어를 사용할 수 있기 때문에, 높은 견인력을 유지하면서도 홀로노믹 기동성을 확보할 수 있다.4 이는 기동성과 견인력 사이의 상충 관계를 극복하려는 시도이며, 두 가지 성능 모두가 중요한 고성능 애플리케이션에서 스위블 드라이브가 선택되는 근본적인 이유이다.

물론 이러한 장점은 높은 **기계적 및 제어적 복잡성**을 대가로 한다. 총 8개의 모터(4x 조향, 4x 구동)와 이를 지지하고 동력을 전달하기 위한 복잡한 기어, 벨트, 베어링 구조는 제작 및 유지보수 비용을 크게 증가시킨다.4 또한, 8개의 액추에이터를 실시간으로 동기화하여 제어해야 하므로 제어 알고리즘의 복잡도가 높으며, 특정 조건에서 제어 불능 상태에 빠질 수 있는 기구학적 특이점(kinematic singularity) 문제를 반드시 고려해야 한다.12

### 0.3 ROS2 Humble 및 `ros2_control` 프레임워크의 역할

ROS2 Humble은 차세대 로봇 개발을 위한 오픈소스 프레임워크로, 실시간성, 보안, 다중 플랫폼 지원 등 ROS1에 비해 여러 측면에서 크게 개선되었다.13 특히 산업 표준 미들웨어인 DDS(Data Distribution Service)를 통신 계층의 기본으로 채택하여, 여러 노드로 구성된 분산 시스템의 안정성과 신뢰성을 확보하였다.

이러한 ROS2 환경에서 `ros2_control` 프레임워크는 스위블 드라이브와 같이 복잡한 로봇의 제어 시스템을 구축하는 데 핵심적인 역할을 수행한다.15

`ros2_control`은 고수준의 제어 알고리즘(예: 경로 계획, 자율 주행)과 저수준의 하드웨어(모터, 엔코더) 사이에 표준화된 추상화 계층을 제공한다.17 이는 하드웨어에 대한 직접적인 종속성을 제거하여 소프트웨어의 모듈성과 재사용성을 극대화한다.16

스위블 드라이브의 본질적인 복잡성은 8개의 독립적인 액추에이터를 실시간으로 정밀하게 제어해야 한다는 점에서 비롯된다. 만약 단일 프로그램에서 고수준 로직과 저수준 제어를 모두 처리하려 한다면, 코드는 극도로 복잡해지고 유지보수가 어려워질 것이다.19

`ros2_control`은 이러한 복잡성을 관리하기 위한 필수적인 아키텍처 패턴을 제공한다. '하드웨어 인터페이스'라는 개념을 통해 실제 하드웨어와의 통신 로직을 캡슐화하고, 컨트롤러는 표준화된 '커맨드 인터페이스'와 '상태 인터페이스'를 통해 하드웨어에 접근한다.16 이러한 '관심사의 분리(Separation of Concerns)' 원칙은 개발자가 복잡한 하드웨어 제어의 세부 사항 대신 로봇의 기구학 및 제어 로직 자체에 집중할 수 있도록 하여, 개발 효율성과 시스템의 안정성을 비약적으로 향상시킨다.

## 1.  스위블 드라이브 모듈의 기계적 설계 및 제작

스위블 드라이브 로봇의 성능은 각 휠 모듈의 기계적 완성도에 크게 좌우된다. 정밀하고 강인한 모듈 설계는 제어 안정성의 기반이 되며, 소프트웨어 제어의 복잡도와 직결되는 선행 조건으로 작용한다. 본 장에서는 스위블 모듈의 핵심 설계 방식, 부품 선정 기준, 그리고 모듈화의 이점에 대해 상세히 기술한다.

### 1.1 모듈 구조 설계: 동축(Coaxial) 방식과 비동축 방식

스위블 모듈의 구동 모터와 조향 모터를 배치하는 방식에 따라 크게 동축 방식과 비동축 방식으로 나눌 수 있다.

#### 1.1.1 동축 설계 (Coaxial Design)

동축 설계는 구동 모터의 회전축과 조향 모듈의 회전축이 동일한 중심선을 공유하는 구조다.21 구동 모터는 섀시에 고정되어 있으며, 그 동력은 중공축(hollow shaft) 내부를 통과하는 구동축과 베벨 기어(bevel gear) 세트를 통해 휠로 전달된다.22

- **장점**: 구동 모터가 조향부와 함께 회전하지 않으므로 전체 모듈의 회전 관성이 작다. 이는 더 빠른 조향 응답성을 가능하게 한다. 또한, 모터가 섀시에 고정되어 있어 케이블 관리가 용이하며, 전체적으로 더 컴팩트하고 무게 중심이 낮은 설계를 구현할 수 있다.
- **단점**: 가장 큰 단점은 조향과 구동 간의 기계적 커플링(coupling)이다. 모듈이 조향을 위해 회전할 때, 휠에 동력을 전달하는 베벨 기어도 함께 공전하게 된다. 이로 인해 구동 모터가 정지해 있더라도 휠이 의도치 않게 회전하는 현상이 발생한다.22 이 현상은 제어 소프트웨어 단에서 반드시 보상되어야 하며, 이는 제어 알고리즘의 복잡성을 증가시킨다. 또한, 베벨 기어의 사용은 필연적으로 백래시(backlash) 문제를 야기하며, 이는 조향 정밀도에 부정적인 영향을 줄 수 있다.7

#### 1.1.2 비동축 설계 (Non-coaxial Design)

비동축 설계는 구동 모터가 조향 링크에 직접 부착되어 조향부 전체와 함께 회전하는 구조다. 구동 모터의 동력은 일반적으로 타이밍 벨트나 체인을 통해 휠로 전달된다.9

- **장점**: 조향과 구동 메커니즘이 기계적으로 분리되어 있어 커플링 현상이 발생하지 않는다. 따라서 제어 로직이 더 직관적이고 간단해진다. 베벨 기어를 사용하지 않으므로 백래시 문제에서도 비교적 자유롭다.
- **단점**: 구동 모터 자체가 조향부와 함께 회전하므로 회전 관성이 크다. 이는 조향 속도를 제한하는 요인이 될 수 있다. 또한, 회전하는 모터에 전력과 제어 신호를 전달하기 위해 슬립 링(slip ring)을 사용하거나 케이블이 꼬이지 않도록 회전 범위를 제한해야 하는 등 케이블 관리가 복잡해진다.

이러한 기계적 불완전성은 소프트웨어에서 반드시 보상해야 하는 추가적인 제어 로직을 요구한다. 예를 들어, 동축 설계의 커플링 현상은 역기구학 모델에 보상 항을 추가하거나 피드포워드 제어를 통해 상쇄해야 한다. 백래시는 PID 제어기의 불안정성을 유발하며, 이를 해결하기 위해 데드존(deadzone) 설정이나 복잡한 비선형 제어 기법이 필요할 수 있다. 따라서 모듈 설계 단계에서 백래시가 적은 고품질 부품을 선택하고 정밀하게 조립하는 것은 후속 소프트웨어 개발의 난이도를 결정하는 매우 중요한 요소이다.

### 1.2 핵심 부품 선정 가이드

고성능 스위블 모듈을 제작하기 위해서는 각 부품의 역할을 이해하고 적절한 사양을 선택하는 것이 중요하다.

- **모터**: 각 모듈당 구동용과 조향용으로 총 2개, 로봇 전체로는 8개의 모터가 필요하다.
  - **조향 모터**: 정밀한 각도 제어가 핵심이므로, 높은 분해능의 엔코더가 내장되고 백래시가 적은 고감속비 기어박스가 결합된 BLDC 서보 모터가 이상적이다.
  - **구동 모터**: 로봇의 총 중량, 목표 가속도 및 최고 속도를 고려하여 충분한 토크와 RPM을 제공하는 BLDC 또는 고출력 DC 모터를 선정해야 한다.23
- **감속기**: 특히 조향축에는 백래시가 조향 정밀도에 치명적인 영향을 미치므로, 백래시가 매우 적은 유성기어(Planetary Gear) 감속기가 선호된다.7
- **기어 및 동력 전달**:
  - **동축 방식**: 90도 동력 전달을 위해 베벨 기어가 필수적이다. 이때 기어 사이에 발생하는 축 방향 반발력(axial load)을 견디기 위해 스러스트 베어링(thrust bearing)을 반드시 설계에 포함해야 한다.7
  - **비동축 방식**: 타이밍 벨트와 풀리를 주로 사용하며, 운용 중 벨트의 장력이 느슨해지지 않도록 장력 조절 장치를 마련하는 것이 중요하다.9
- **엔코더**: 각 모듈의 상태를 정확히 파악하기 위해 2개의 엔코더가 필요하다.
  - **조향축**: 전원이 켜졌을 때 즉시 현재 각도를 알 수 있도록 절대 위치 엔코더(absolute encoder)를 사용하는 것이 가장 이상적이다. 비용 문제로 상대 위치 엔코더를 사용할 경우, 초기 구동 시 영점(zero position)을 찾는 절차가 필요하다. 홀 효과 센서(Hall effect sensor)와 자석을 이용해 특정 위치를 감지하는 방식으로 구현할 수 있다.24
  - **구동축**: 휠의 회전 속도와 이동 거리를 측정하기 위해 증분형 엔코더(incremental encoder)가 일반적으로 사용된다.7

### 1.3 모듈화 설계의 이점과 실제 조립 시 고려사항

스위블 드라이브는 4개의 동일한 모듈로 구성하는 것이 표준적인 접근 방식이며, 이는 개발 및 유지보수 측면에서 상당한 이점을 제공한다.

- **신속한 교체 및 유지보수**: 현장에서 특정 모듈에 문제가 발생했을 때, 전체 로봇을 분해할 필요 없이 해당 모듈만 예비 부품으로 신속하게 교체할 수 있다. 이는 로봇의 가동 중단 시간을 최소화하는 데 결정적이다.4
- **개발 프로세스 가속화**: 모듈화는 단순한 유지보수 편의성을 넘어, 개발 프로세스 자체를 가속화하는 핵심 전략이다. 4개의 모듈이 동일하다는 것은 개발팀이 하드웨어 제작, 펌웨어 개발, 단위 테스트를 병렬적으로 진행할 수 있음을 의미한다. 예를 들어, 한 팀원은 1번 모듈을 기계적으로 조립하는 동안, 다른 팀원은 2번 모듈에 연결될 MCU의 펌웨어를 개발하고, 또 다른 팀원은 3번 모듈을 테스트 벤치에 올려 PID 제어기를 튜닝하는 것이 가능하다. 이러한 병렬 개발은 복잡한 시스템의 전체 프로젝트 리드 타임을 획기적으로 단축시킨다.
- **사전 보정(Pre-calibration)**: 각 모듈을 로봇에 장착하기 전에 개별적으로 테스트하고 엔코더 오프셋 등을 보정할 수 있다. 이는 전체 시스템 통합 후 발생할 수 있는 문제를 사전에 파악하고 해결하는 데 도움을 준다.7
- **조립 정밀도**: 설계 시 휠의 지면 접촉점이 조향축의 중심선과 정확히 일치하도록 해야 한다. 만약 오프셋이 존재하면 조향 시 휠이 지면을 끌게 되어(scrubbing) 타이어 마모, 에너지 손실, 오도메트리 오차를 유발한다.7 또한, 모든 기어와 풀리가 정확하게 맞물리도록 하여 백래시를 최소화하는 정밀한 조립이 요구된다.7

**표 1: 전방향 구동 방식 비교**

| 구동 방식           | 기동성 (홀로노믹 여부) | 견인력    | 제어 복잡도 | 기계적 복잡도 | 비용      | 지면 조건 민감도 | 주요 응용 분야                        |
| ------------------- | ---------------------- | --------- | ----------- | ------------- | --------- | ---------------- | ------------------------------------- |
| **스위블 드라이브** | 완전 홀로노믹          | 매우 높음 | 매우 높음   | 매우 높음     | 매우 높음 | 낮음             | 물류 로봇, FRC, 고성능 서비스 로봇    |
| **메카넘 휠**       | 완전 홀로노믹          | 낮음      | 높음        | 중간          | 높음      | 매우 높음        | 실내 서비스 로봇, 교육용 로봇         |
| **옴니 휠**         | 완전 홀로노믹          | 낮음      | 높음        | 중간          | 중간      | 높음             | RoboCup, 컨베이어 시스템, 교육용 로봇 |
| **차동 구동**       | 비홀로노믹             | 높음      | 낮음        | 낮음          | 낮음      | 낮음             | 대부분의 모바일 로봇, 청소 로봇       |

이 표는 각 구동 방식의 상대적인 특성을 요약한 것으로, 스위블 드라이브가 기동성과 견인력을 모두 확보하는 대신 복잡성과 비용이 크게 증가하는 트레이드오프 관계를 명확히 보여준다. 2

## 2.  4륜 스위블 드라이브 기구학 모델링

스위블 드라이브 로봇의 정밀한 제어는 정확한 기구학 모델에 기반한다. 기구학은 로봇의 기하학적 구조로부터 각 관절(joint)의 움직임과 로봇 전체의 움직임 사이의 관계를 수학적으로 기술하는 학문이다. 본 장에서는 역기구학(Inverse Kinematics)과 순기구학(Forward Kinematics) 모델을 상세히 유도한다.

### 2.1 좌표계 설정 및 기구학적 변수 정의

정확한 모델링을 위해 먼저 좌표계와 변수들을 명확히 정의해야 한다.

- **로봇 기준 좌표계 `{B}`**: 로봇의 기하학적 중심에 원점을 두며, 로봇의 움직임을 기술하는 기준이 된다. 일반적으로 $x_B$축은 로봇의 전방, $y_B$축은 좌측, $z_B$축은 상방을 향하도록 설정한다.
- **월드 고정 좌표계 `{W}`**: 외부 환경에 고정된 전역 좌표계로, 로봇의 절대적인 위치와 방향을 나타내는 데 사용된다.
- **휠 모듈 좌표계 `{i}`**: 각 휠 모듈의 조향축 중심에 원점을 둔다. $i \in \{fl, fr, rl, rr\}$ (front-left, front-right, rear-left, rear-right)로 각 휠을 구분한다.

주요 기구학적 변수는 다음과 같다.

- 로봇의 속도 벡터 (로봇 기준 좌표계 {B}):

  $\mathbf{v}_B = [v_x, v_y, \omega_z]^T$

  - $v_x$: 로봇의 전후 방향 선속도
  - $v_y$: 로봇의 좌우 방향(strafe) 선속도
  - $\omega_z$: 로봇의 제자리 회전 각속도

- 휠 $i$의 상태 변수:

  - $\phi_i$: 휠의 조향각(steering angle). 로봇의 $x_B$축을 기준으로 반시계 방향을 양(+)으로 정의한다.
  - $\omega_{w,i}$: 휠의 구동 각속도(wheel angular velocity).
  - $v_{w,i}$: 휠의 구동 선속도.

- 기하학적 파라미터:

  - $\mathbf{r}_i = [r_{ix}, r_{iy}]^T$: 로봇 중심에서 휠 $i$의 조향축 중심까지의 위치 벡터.
  - $R_w$: 휠의 반경.

### 2.2 역기구학(Inverse Kinematics) 모델 유도

역기구학은 로봇 전체에 주어진 목표 속도($\mathbf{v}_B$)를 달성하기 위해 각 휠 모듈이 가져야 할 목표 조향각($\phi_i$)과 목표 구동 속도($v_{w,i}$)를 계산하는 과정이다. 이는 `/cmd_vel` 토픽으로 들어온 Twist 메시지를 실제 모터 명령으로 변환하는 핵심 로직이다.

유도 과정은 강체(rigid body)의 운동학 원리에 기반한다.4 로봇 중심의 병진 운동과 회전 운동이 결합되었을 때, 휠 $i$의 위치에서 발생하는 속도 벡터 $\vec{v}_{w,i}$는 다음과 같이 두 성분의 벡터 합으로 표현된다.4
$$
\vec{v}_{w,i} = \vec{v}_{body} + \vec{\omega_z} \times \vec{r_i}
$$
여기서 $\vec{v}_{body} = [v_x, v_y, 0]^T$ 이고, $\vec{\omega_z} = [0, 0, \omega_z]^T$, $\vec{r_i} = [r_{ix}, r_{iy}, 0]^T$ 이다. 외적($\times$)을 계산하면 다음과 같다.
$$
\vec{\omega_z} \times \vec{r_i} = \begin{vmatrix} \mathbf{i} & \mathbf{j} & \mathbf{k} \\ 0 & 0 & \omega_z \\ r_{ix} & r_{iy} & 0 \end{vmatrix} = [-\omega_z r_{iy}, \omega_z r_{ix}, 0]^T
$$
따라서, 휠 $i$의 속도 벡터 $\vec{v}_{w,i}$의 $x$와 $y$ 성분은 다음과 같이 분해된다.
$$
v_{w,i,x} = v_x - \omega_z r_{iy}
$$

$$
v_{w,i,y} = v_y + \omega_z r_{ix}
$$

이 계산된 속도 벡터 $\vec{v}_{w,i}$가 바로 휠 $i$가 지면을 밀어야 하는 방향과 속도를 나타낸다. 따라서, 목표 조향각 $\phi_i$는 이 벡터의 방향이며, 목표 구동 선속도 $v_{w,i}$는 이 벡터의 크기가 된다.

- **목표 조향각 $\phi_i$**:
  $$
  \phi_i = \operatorname{atan2}(v_{w,i,y}, v_{w,i,x})
  $$

- **목표 구동 선속도 $v_{w,i}$**:
  $$
  v_{w,i} = \sqrt{v_{w,i,x}^2 + v_{w,i,y}^2}
  $$
  **목표 구동 각속도 $\omega_{w,i}$**:
  $$
  \omega_{w,i} = \frac{v_{w,i}}{R_w}
  $$
  이 수식들을 사용하여 4개의 휠 모듈 각각에 대한 8개의 제어 목표값(4개의 조향각, 4개의 구동 속도)을 계산할 수 있다.

### 2.3 순기구학(Forward Kinematics) 모델 유도 (오도메트리)

순기구학은 역기구학의 반대 과정으로, 각 휠 모듈의 엔코더로부터 측정된 실제 조향각($\phi_i$)과 실제 구동 속도($v_{w,i}$)를 이용하여 로봇 전체의 실제 속도($\mathbf{v}_B$)를 추정하는 과정이다. 이는 로봇의 현재 위치를 추정하는 오도메트리(Odometry) 계산의 핵심이다.

여기서 중요한 점은, 순기구학이 단순한 역 연산이 아니라는 것이다. 우리는 4개 휠의 조향각과 구동 속도, 즉 최소 8개의 센서 측정값을 가지고 3개의 미지수($v_x, v_y, \omega_z$)를 구해야 한다. 이는 방정식의 개수가 미지수의 개수보다 많은 과결정 시스템(Overdetermined System)이다.12 이는 모든 휠이 미끄러짐 없이 완벽하게 기구학적 구속 조건을 만족하지 않는 한, 모든 센서 값을 동시에 만족시키는 유일한 해는 존재하지 않음을 의미한다. 따라서 순기구학은 '결정'의 문제가 아닌, 센서 데이터로부터 가장 가능성이 높은 로봇의 움직임을 '추정(Estimation)'하는 문제로 접근해야 한다.

이를 해결하기 위해 최소 자승법(Least Squares Method)에 기반한 의사 역행렬(Pseudo-inverse)을 사용한다. 먼저 역기구학에서 유도한 식을 행렬 형태로 표현한다.25
$$
\begin{bmatrix} v_{w,fl,x} \\ v_{w,fl,y} \\ v_{w,fr,x} \\ v_{w,fr,y} \\ v_{w,rl,x} \\ v_{w,rl,y} \\ v_{w,rr,x} \\ v_{w,rr,y} \end{bmatrix}
=
\begin{bmatrix}
1 & 0 & -r_{fl,y} \\
0 & 1 &  r_{fl,x} \\
1 & 0 & -r_{fr,y} \\
0 & 1 &  r_{fr,x} \\
1 & 0 & -r_{rl,y} \\
0 & 1 &  r_{rl,x} \\
1 & 0 & -r_{rr,y} \\
0 & 1 &  r_{rr,x}
\end{bmatrix}
\begin{bmatrix} v_x \\ v_y \\ \omega_z \end{bmatrix}
$$
위 식을 간략하게 $\mathbf{V}_w = \mathbf{J} \cdot \mathbf{v}_B$로 표현할 수 있다. 여기서 $\mathbf{V}_w$는 8x1 휠 속도 벡터, $\mathbf{J}$는 8x3 자코비안 행렬, $\mathbf{v}_B$는 3x1 로봇 속도 벡터이다. 우리가 구하고자 하는 것은 $\mathbf{v}_B$이므로, $\mathbf{J}$의 의사 역행렬 $\mathbf{J}^+$를 양변의 왼쪽에 곱한다.
$$
\mathbf{v}_B = \mathbf{J}^+ \cdot \mathbf{V}_w
$$
여기서 $\mathbf{J}^+ = (\mathbf{J}^T \mathbf{J})^{-1} \mathbf{J}^T$ 이다. $\mathbf{V}_w$의 각 성분은 엔코더로부터 측정된 실제 휠 속도 $v_{w,i}$와 조향각 $\phi_i$를 사용하여 다음과 같이 계산된다.
$$
v_{w,i,x} = v_{w,i} \cos(\phi_i)
$$

$$
v_{w,i,y} = v_{w,i} \sin(\phi_i)
$$

이 추정된 로봇 속도 $\mathbf{v}_B$를 시간에 대해 적분하면 로봇의 위치와 방향, 즉 오도메트리를 계산할 수 있다. 이 과정은 휠 슬립이나 지면 불균일 등으로 인해 오차가 누적되기 쉬우며, 이것이 IMU와 같은 외부 센서와의 융합이 필수적인 이유이다.26

또한, 기구학 모델의 자코비안 행렬 $\mathbf{J}$는 로봇의 기하학적 파라미터($r_{ix}, r_{iy}$)에 직접적으로 의존한다. 만약 제작 과정에서 이 치수에 오차가 발생하면 모델과 실제 로봇 사이에 불일치가 발생하여, 역기구학에서는 불필요한 휠 슬립과 내부적인 힘의 상쇄(actuator fighting)를 유발하고, 순기구학에서는 오도메트리 추정의 정확도를 심각하게 저하시킨다. 따라서 설계 단계의 정밀한 치수 관리와 제작 후 실제 치수 측정을 통한 모델 파라미터 보정(calibration) 과정은 고성능 스위블 드라이브 로봇 개발에 있어 매우 중요한 단계이다.

## 3.  ROS2 Humble 기반 통합 시스템 아키텍처

성공적인 스위블 드라이브 로봇 개발을 위해서는 강인하고 확장 가능한 시스템 아키텍처를 설계하는 것이 필수적이다. 본 장에서는 하드웨어와 소프트웨어 아키텍처를 계층적으로 나누어 설명하며, 특히 `ros2_control` 프레임워크를 중심으로 각 구성 요소의 역할과 상호작용을 상세히 기술한다.

### 3.1 하드웨어 아키텍처: 계층적 제어 구조

스위블 드라이브 로봇의 하드웨어는 일반적으로 실시간성과 고성능 연산이라는 두 가지 상이한 요구사항을 만족시키기 위해 계층적 구조로 설계된다. 이는 현대 로보틱스 설계의 표준적인 패러다임을 반영한다.

- **고수준 제어 계층 (High-Level Control Layer)**: 이 계층은 복잡한 연산과 비실시간성 작업을 담당한다.
  - **SBC (Single-Board Computer)**: Raspberry Pi 5 또는 NVIDIA Jetson 시리즈와 같은 고성능 SBC가 사용된다.27 이 SBC는 Ubuntu 운영체제 위에서 ROS2 Humble 전체 스택을 실행하며, SLAM(동시적 위치 추정 및 지도 작성), Nav2(경로 계획 및 추종), 객체 인식 등 계산 집약적인 알고리즘을 처리한다.29 리눅스 기반의 풍부한 라이브러리와 개발 환경을 활용할 수 있지만, 커널의 스케줄링 방식으로 인해 엄격한 실시간성을 보장하기는 어렵다.
- **저수준 제어 계층 (Low-Level Control Layer)**: 이 계층은 결정론적(deterministic)이고 빠른 응답이 요구되는 실시간 작업을 전담한다.
  - **MCU (Microcontroller Unit)**: Teensy, ESP32, STM32 등과 같은 MCU가 사용된다.19 MCU는 베어메탈(bare-metal) 환경 또는 실시간 운영체제(RTOS) 위에서 동작하며, 8개의 모터에 대한 정밀한 PWM 신호 생성, 다채널 엔코더 신호의 고속 카운팅, 그리고 각 조인트에 대한 PID(Proportional-Integral-Derivative) 제어 루프와 같은 저지연(low-latency) 작업을 수행한다.
- **인터페이스 및 주변 장치**:
  - **SBC-MCU 통신**: 두 계층 간의 통신은 시스템의 성능과 안정성에 매우 중요하다. 전통적인 USB Serial 통신이 널리 사용되지만, 더 높은 신뢰성과 실시간성이 요구되는 경우 CAN bus가 사용될 수 있다. 특히 `micro-ROS`는 SBC의 ROS2 시스템과 MCU 간의 통신을 DDS 프로토콜 기반으로 효율적으로 통합해주는 강력한 솔루션이다.29
  - **모터 드라이버**: MCU로부터 받은 논리 신호(PWM, 방향)를 실제 모터를 구동할 수 있는 높은 전류와 전압으로 증폭하는 역할을 한다. 각 모듈당 2채널 드라이버 또는 총 8개의 단일 채널 드라이버가 필요하다.
  - **센서**:
    - **IMU (Inertial Measurement Unit)**: 로봇의 각속도와 선형 가속도를 측정하는 필수 센서다. 휠 오도메트리의 드리프트 오차를 보정하는 데 핵심적인 역할을 한다.
    - **LiDAR / Depth Camera**: 자율 주행을 위한 환경 인식 센서로, SLAM과 장애물 회피에 사용된다. 이들 센서는 일반적으로 SBC에 직접 연결되어 ROS2 노드를 통해 데이터를 발행한다.

이러한 SBC-MCU 계층 분리 아키텍처는 각 컴퓨팅 유닛의 장점을 극대화하는 전략이다. 이는 복잡한 자율주행 로봇이 요구하는 상이한 두 가지 요구사항, 즉 '고성능 비실시간 연산'과 '저지연 실시간 제어'를 동시에 만족시키기 위한 필연적인 구조적 선택이라 할 수 있다.

### 3.2 소프트웨어 아키텍처: `ros2_control` 중심의 모듈형 구조

소프트웨어 아키텍처는 `ros2_control` 프레임워크를 중심으로 설계하여 모듈성, 재사용성, 그리고 시뮬레이션-실제 로봇 간의 호환성을 확보한다.

- **드라이버 계층 (Driver Layer - Hardware Interface)**:
  - **`swerve_hardware_interface`**: `ros2_control`의 핵심인 하드웨어 추상화를 담당한다. 이 컴포넌트는 `hardware_interface::SystemInterface`를 상속받는 C++ 플러그인으로 구현된다. SBC와 MCU 간의 통신 프로토콜(예: Serial, micro-ROS)을 내부에 캡슐화하고, 8개 조인트(4x 구동, 4x 조향)의 상태를 읽어 `state_interface`(예: `position`, `velocity`)로 노출하고, 상위 컨트롤러로부터 받은 명령을 `command_interface`를 통해 수신하여 MCU로 전송하는 역할을 한다.
- **컨트롤러 계층 (Controller Layer)**:
  - **`swerve_drive_controller`**: 로봇의 핵심 제어 로직을 담고 있는 컴포넌트다. `controller_interface::ControllerInterface`를 상속받아 `ros2_control`의 컨트롤러로 구현된다.
    1. 외부(예: 조이스틱, Nav2)로부터 `geometry_msgs/msg/Twist` 타입의 속도 명령을 `/cmd_vel` 토픽으로 구독한다.
    2. 수신된 Twist 메시지를 제2장에서 유도한 역기구학 모델에 입력하여 8개 조인트의 목표 값(조향각, 구동속도)을 계산한다.
    3. 계산된 목표 값을 `command_interface`를 통해 `swerve_hardware_interface`로 전달한다.
    4. 주기적으로 `state_interface`를 통해 현재 조인트 상태를 읽어온다.
    5. 읽어온 조인트 상태를 순기구학 모델에 입력하여 로봇의 현재 속도를 추정하고, 이를 적분하여 `nav_msgs/msg/Odometry` 타입의 오도메트리 정보를 `/odom` 토픽으로 발행한다. 또한, `odom` -> `base_link` 좌표 변환(TF) 정보도 발행한다.
  - **`joint_state_broadcaster`**: `ros2_controllers` 패키지에서 제공하는 표준 컨트롤러다. 모든 조인트의 현재 상태(`state_interface`에서 읽어옴)를 `sensor_msgs/msg/JointState` 타입으로 변환하여 `/joint_states` 토픽으로 발행한다. 이 정보는 RViz에서의 로봇 모델 시각화나 다른 분석 도구에서 사용된다.
- **응용 계층 (Application Layer)**:
  - **원격 조작**: `teleop_twist_keyboard` 또는 `teleop_twist_joy` 노드를 사용하여 키보드나 조이스틱 입력으로부터 `/cmd_vel` 메시지를 생성한다.
  - **상태 추정**: `robot_localization` 패키지의 `ekf_node`는 `swerve_drive_controller`가 발행한 `/odom`과 IMU 센서의 `/imu/data`를 입력받아 센서 융합을 수행하고, 더 정밀하게 보정된 오도메트리 정보(`/odometry/filtered`)를 발행한다.
  - **자율 주행**: Nav2 스택과 SLAM Toolbox는 보정된 오도메트리와 LiDAR/카메라 센서 정보를 바탕으로 지도를 작성하고, 목표 지점까지의 경로를 계획하며, 장애물을 회피하는 등의 고수준 자율 주행 기능을 수행한다. Nav2 스택은 계산된 속도 명령을 `/cmd_vel` 토픽으로 발행하여 `swerve_drive_controller`에 전달한다.30

이 아키텍처에서 `swerve_drive_controller`는 단순한 노드가 아니라, 로봇의 특수한 물리적 특성(기구학)과 ROS 생태계의 표준 인터페이스(Twist, Odometry)를 연결하는 핵심적인 **'어댑터(Adapter)'** 역할을 수행한다. 이 컨트롤러만 표준에 맞게 잘 구현된다면, Nav2와 같은 고수준 스택의 관점에서는 스위블 드라이브 로봇이 일반적인 차동 구동 로봇과 전혀 다를 바 없이 취급될 수 있다. 이는 ROS의 강력한 모듈성과 재사용성을 보여주는 대표적인 사례이다.

## 4.  `ros2_control` 프레임워크를 이용한 제어 시스템 구현

본 장에서는 제3장에서 설계한 소프트웨어 아키텍처를 바탕으로, ROS2 Humble 환경에서 `ros2_control`을 사용하여 스위블 드라이브 로봇의 제어 시스템을 실제로 구현하는 과정을 단계별로 상세히 설명한다. 이는 로봇 모델링(URDF), 하드웨어 인터페이스 개발, 컨트롤러 구현, 그리고 Gazebo를 이용한 시뮬레이션 환경 구축을 포함한다.

### 4.1 로봇 모델링: URDF 및 XACRO

URDF(Unified Robot Description Format)는 로봇의 물리적 구조를 XML 형식으로 기술하는 표준이다. 스위블 드라이브 시스템에서 URDF는 단순한 시각적 모델을 넘어, 물리적 로봇, 제어 시스템, 시뮬레이터를 하나로 묶는 **중앙 설정 파일(Central Configuration File)** 역할을 수행한다.

- **구조 및 조인트 정의**: 로봇의 기본 몸체를 나타내는 `base_link`를 최상위 링크로 설정한다. 각 스위블 모듈은 조향을 담당하는 `steering_link`와 바퀴를 나타내는 `wheel_link`로 구성된다. 이들 링크는 조인트(joint)로 연결된다.31

  - `base_link` -->> `steering_link`: 조향 축을 중심으로 회전하므로 `revolute` 타입 조인트로 정의한다. 회전 각도 제한을 설정할 수 있다.
  - `steering_link` -->> `wheel_link`: 바퀴의 구동 축으로, 무한 회전이 가능해야 하므로 `continuous` 타입 조인트로 정의한다.

- **XACRO 활용**: 4개의 스위블 모듈은 동일한 구조를 가지므로, XACRO(XML Macros)를 사용하여 모듈 하나를 매크로로 정의하고 이를 4번 호출하는 방식으로 URDF를 작성한다.31 이는 코드의 중복을 제거하고, 파라미터(예: 모듈의 위치, 이름 접두사) 변경을 용이하게 하여 유지보수성을 크게 향상시킨다.

- **`ros2_control` 연동**: URDF 파일 내에 `<ros2_control>` 태그를 추가하여 제어 시스템의 구성을 명시한다. 이 태그는 컨트롤러 매니저가 하드웨어 인터페이스와 컨트롤러를 올바르게 연결하는 데 필요한 모든 정보를 담고 있다.15

  ```XML
  <ros2_control name="SwerveDriveSystem" type="system">
      <hardware>
          <plugin>swerve_hardware/SwerveHardwareInterface</plugin>
          <param name="serial_port">/dev/ttyACM0</param>
      </hardware>
      <joint name="front_left_steering_joint">
          <command_interface name="position"/>
          <state_interface name="position"/>
          <state_interface name="velocity"/>
      </joint>
      <joint name="front_left_wheel_joint">
          <command_interface name="velocity"/>
          <state_interface name="position"/>
          <state_interface name="velocity"/>
      </joint>
      </ros2_control>
  ```

  위 예시에서 `<plugin>` 태그는 사용할 하드웨어 인터페이스 클래스를 지정하고, 각 `<joint>` 태그는 제어할 조인트의 이름과 사용 가능한 인터페이스(`command_interface`, `state_interface`)를 정의한다.17

### 4.2 하드웨어 인터페이스(Hardware Interface) C++ 플러그인 개발

하드웨어 인터페이스는 `ros2_control` 프레임워크와 실제 로봇 하드웨어(MCU) 사이의 다리 역할을 하는 C++ 플러그인이다. `hardware_interface::SystemInterface`를 상속받아 구현하며, 이는 여러 조인트를 하나의 컴포넌트에서 관리할 때 사용된다.33

- **주요 메서드 구현**:
  - `on_init(info)`: 컨트롤러 매니저에 의해 플러그인이 로드될 때 한 번 호출된다. URDF의 `<param>` 태그로부터 시리얼 포트 이름, 보드레이트 등의 파라미터를 읽고, MCU와의 통신을 초기화하며, 상태 및 커맨드 저장을 위한 멤버 변수(예: `std::vector<double>`)의 크기를 조절한다.
  - `export_state_interfaces()`: 로봇의 현재 상태를 상위 컨트롤러에 보고하기 위한 인터페이스를 등록한다. URDF에 정의된 각 조인트의 `state_interface`에 대해, 해당 값을 저장할 멤버 변수의 주소를 `hardware_interface::StateInterface` 객체로 감싸 반환한다.
  - `export_command_interfaces()`: 상위 컨트롤러로부터 명령을 수신하기 위한 인터페이스를 등록한다. `state_interface`와 유사하게, 명령을 저장할 멤버 변수의 주소를 `hardware_interface::CommandInterface` 객체로 감싸 반환한다.
  - `read(time, period)`: 제어 루프의 시작 부분에서 주기적으로 호출된다. MCU로부터 현재 엔코더 값과 같은 센서 데이터를 수신하고, 이를 파싱하여 `position`, `velocity` 등의 상태를 나타내는 멤버 변수들을 업데이트한다.
  - `write(time, period)`: 제어 루의 끝 부분에서 주기적으로 호출된다. 컨트롤러에 의해 업데이트된 커맨드 멤버 변수의 값을 읽어, 이를 MCU가 이해할 수 있는 프로토콜(예: `[속도1, 각도1, 속도2, 각도2,...]`)로 변환하여 시리얼 포트를 통해 전송한다.

### 4.3 스위블 드라이브 컨트롤러(Swerve Drive Controller) 노드 개발

이 컨트롤러는 `ros2_control`의 일부로 동작하며, `/cmd_vel` 명령을 실제 조인트 명령으로 변환하고 오도메트리를 발행하는 역할을 한다. `controller_interface::ControllerInterface`를 상속받아 구현한다.25

- **주요 메서드 구현**:
  - `on_init()`: 컨트롤러 이름, 조인트 이름 등의 파라미터를 읽고, `/cmd_vel` 구독자와 `/odom` 발행자를 초기화한다.
  - `command_interface_configuration()`: 이 컨트롤러가 제어할 커맨드 인터페이스의 목록을 정의한다. 스위블 드라이브의 경우 4개의 `position` (조향)과 4개의 `velocity` (구동) 인터페이스를 요청한다.
  - `state_interface_configuration()`: 컨트롤러가 상태를 읽어올 스테이트 인터페이스의 목록을 정의한다. 오도메트리 계산을 위해 8개 조인트 모두의 `position`과 `velocity` 인터페이스를 요청한다.
  - `update(time, period)`: 제어 루프마다 호출되는 핵심 함수다.
    1. `/cmd_vel` 토픽에서 최신 Twist 메시지를 읽어온다.
    2. 읽어온 $v_x$, $v_y$, $\omega_z$를 사용하여 역기구학을 계산한다.
    3. 계산된 8개의 목표 조인트 값(조향각, 구동속도)을 `command_interfaces_` 배열의 해당 위치에 기록한다.
    4. `state_interfaces_` 배열에서 현재 조인트 상태를 읽어온다.
    5. 읽어온 상태 값으로 순기구학을 계산하여 로봇의 현재 속도를 추정한다.
    6. 추정된 속도를 시간에 대해 적분하여 로봇의 위치와 방향을 갱신하고, `/odom` 토픽과 관련 TF를 발행한다.

### 4.4 Gazebo 시뮬레이션 환경 구축

`gazebo_ros2_control` 플러그인은 실제 하드웨어 없이도 제어 시스템 전체를 개발하고 테스트할 수 있는 'Sim-to-Real' 워크플로우를 가능하게 하는 핵심 기술이다. 이는 제어 로직의 버그가 고가의 하드웨어에 손상을 입힐 수 있는 스위블 드라이브 개발에서 특히 중요하다.

- **URDF 설정**: 기존 URDF 파일에 `<gazebo>` 태그를 추가하여 각 링크의 물리적 속성(관성, 질량)과 시각적/충돌 속성을 정의한다. 또한, `ros2_control` 태그 내부의 `<plugin>`을 실제 하드웨어 인터페이스 대신 `gazebo_ros2_control/GazeboSystem`으로 지정한다.17

  ```XML
  <ros2_control name="GazeboSystem" type="system">
      <hardware>
          <plugin>gazebo_ros2_control/GazeboSystem</plugin>
      </hardware>
      </ros2_control>
  ```

- **Launch 파일**: Gazebo 시뮬레이션 환경을 실행하기 위한 Launch 파일을 작성한다. 이 파일은 다음 노드들을 실행해야 한다.

  1. Gazebo 서버 및 클라이언트 (`gazebo_ros` 패키지).
  2. `robot_state_publisher`: URDF를 읽어 로봇의 TF 트리를 발행한다.
  3. `spawn_entity.py`: URDF 모델을 Gazebo 월드에 스폰한다.
  4. `joint_state_broadcaster`와 `swerve_drive_controller`를 로드하는 `spawner` 노드.

- **`use_sim_time` 파라미터**: 모든 노드를 실행할 때 `use_sim_time` 파라미터를 `true`로 설정해야 한다. 이는 노드들이 ROS 시스템 시간이 아닌 Gazebo 시뮬레이터의 시간을 기준으로 동작하도록 하여, 시뮬레이션이 실제 시간보다 느리거나 빠르더라도 일관된 동작을 보장한다.25

이러한 Sim-to-Real 접근 방식은 개발 초기 단계의 리스크와 비용을 극적으로 감소시킨다. 개발자는 기구학 모델 검증, 제어 로직 디버깅, Nav2 스택 연동 테스트 등 대부분의 작업을 시뮬레이션 환경에서 안전하고 효율적으로 수행할 수 있다. 개발이 완료되면, URDF의 플러그인 이름과 Launch 파일의 일부 파라미터만 수정하여 거의 동일한 코드를 실제 로봇에서 실행할 수 있다.

**표 2: `ros2_control` URDF 핵심 태그 및 속성**

| 태그 / 속성           | 역할                                                         | 예시 값                                   | 설명                                                         |
| --------------------- | ------------------------------------------------------------ | ----------------------------------------- | ------------------------------------------------------------ |
| `<ros2_control>`      | `ros2_control` 시스템 전체를 정의하는 루트 태그.             | `name="SwerveDriveSystem" type="system"`  | `name`은 시스템 식별자, `type`은 인터페이스 종류(`system`, `actuator`, `sensor`)를 지정한다. |
| `<hardware>`          | 하드웨어 인터페이스 플러그인과 파라미터를 정의한다.          |                                           |                                                              |
| `<plugin>`            | 로드할 하드웨어 인터페이스 플러그인의 클래스 이름을 지정한다. | `swerve_hardware/SwerveHardwareInterface` | `pluginlib`를 통해 동적으로 로드될 라이브러리를 명시한다.    |
| `<param>`             | 하드웨어 인터페이스에 전달할 파라미터를 정의한다.            | `name="serial_port">/dev/ttyACM0`         | `on_init()` 함수에서 이 값을 읽어 통신 포트를 설정할 수 있다. |
| `<joint>`             | 제어할 조인트를 정의하고, 해당 조인트의 인터페이스를 명시한다. | `name="front_left_steering_joint"`        | URDF의 조인트 이름과 일치해야 한다.                          |
| `<command_interface>` | 컨트롤러가 하드웨어에 명령을 내리기 위해 사용하는 인터페이스. | `name="position"`                         | `position`, `velocity`, `effort` 등 표준 인터페이스 또는 사용자 정의 인터페이스를 사용할 수 있다. |
| `<state_interface>`   | 하드웨어가 컨트롤러에 현재 상태를 보고하기 위해 사용하는 인터페이스. | `name="velocity"`                         | 하나의 조인트는 여러 개의 상태 인터페이스를 가질 수 있다 (예: 위치와 속도). |

이 표는 URDF를 통해 `ros2_control` 시스템을 구성하는 핵심 요소들을 요약한다. 개발자는 이 구조를 이해함으로써 자신의 로봇 하드웨어에 맞는 제어 시스템을 체계적으로 기술하고 통합할 수 있다.15

## 5.  위치 추정 정밀도 향상 및 제어 최적화

기본적인 기구학 모델과 제어 시스템이 구현되었다면, 다음 단계는 실제 환경에서 발생할 수 있는 다양한 오차 요인을 극복하고 로봇의 움직임을 최적화하여 성능을 극대화하는 것이다. 본 장에서는 센서 융합을 통한 오도메트리 정밀도 향상, 조향 메커니즘의 최적화, 그리고 기구학적 특이점 문제 해결 방안에 대해 심도 있게 다룬다.

### 5.1 `robot_localization` 패키지를 이용한 EKF 센서 융합

휠 엔코더 기반의 오도메트리는 바퀴의 미끄러짐(slip)이나 지면의 요철로 인해 필연적으로 오차가 누적되는 한계를 가진다.26 반면, IMU는 자이로스코프와 가속도계를 통해 로봇의 각속도와 가속도를 직접 측정하므로 단기적인 움직임 변화에 강인하지만, 시간에 따라 적분 오차(drift)가 누적된다. 확장 칼만 필터(Extended Kalman Filter, EKF)는 이러한 서로 다른 특성을 가진 센서 데이터들을 통계적으로 융합하여, 각 센서의 단점을 상호 보완하고 더 정확하며 강인한 상태 추정치를 제공하는 강력한 기법이다.37

ROS2에서는 `robot_localization` 패키지가 EKF를 구현한 `ekf_node`를 제공하며, YAML 설정 파일을 통해 복잡한 센서 융합 로직을 손쉽게 구성할 수 있다.38

- **`ekf.yaml` 핵심 파라미터 설정**:
  - `frequency`: 필터가 상태 추정치를 계산하고 발행하는 주기(Hz). 일반적으로 30-50 Hz로 설정한다.
  - `two_d_mode: true`: 스위블 드라이브 로봇은 주로 평평한 2D 평면에서 운용되므로 이 옵션을 활성화하는 것이 매우 중요하다. 이 설정은 상태 벡터의 Z, roll, pitch 관련 변수들을 강제로 0으로 고정하여, IMU의 미세한 노이즈나 로봇의 가감속 시 발생하는 관성력으로 인해 상태 추정치가 3차원으로 발산하는 것을 방지한다.38 이는 운용 환경에 대한 강력한 사전 지식을 필터에 주입하여 안정성을 높이는 전략적 결정이다.
  - `odom_frame`, `base_link_frame`, `world_frame`: ROS REP-105 표준에 따라 좌표계를 설정한다. 일반적으로 `world_frame`은 `odom`, `odom_frame`은 `odom`, `base_link_frame`은 `base_link`로 설정하여 연속적인 오도메트리를 생성한다.
  - `odom0`: 첫 번째 오도메트리 입력 소스로, `swerve_drive_controller`가 발행하는 `/odom` 토픽을 지정한다.
  - `odom0_config`: `odom0`에서 어떤 변수를 필터에 사용할지 결정하는 15차원 불리언 배열이다. 스위블 오도메트리는 X, Y 위치와 Yaw, X, Y 선속도, Yaw 각속도 정보를 제공하므로, 이에 해당하는 인덱스를 `true`로 설정한다.
  - `imu0`: 첫 번째 IMU 입력 소스로, IMU 드라이버가 발행하는 `/imu/data` 토픽을 지정한다.
  - `imu0_config`: IMU에서 Yaw, 각속도, 선형 가속도 정보를 사용하도록 설정한다. `two_d_mode`가 `true`이므로 Roll과 Pitch는 사용하도록 설정해도 필터 내부에서 무시된다.
  - `imu0_remove_gravitational_acceleration: true`: IMU의 선형 가속도 측정값에서 중력 가속도 성분을 제거하도록 설정한다. 이를 위해서는 IMU가 신뢰할 수 있는 방향(orientation) 정보를 함께 제공해야 한다.

### 5.2 조향 최적화 알고리즘

스위블 드라이브의 독특한 메커니즘은 효율적이고 안정적인 움직임을 위해 몇 가지 제어 최적화를 요구한다. 이러한 기법들은 단순히 성능을 개선하는 것을 넘어, 물리적으로 실현 가능하고 예측 가능한 움직임을 만들기 위한 필수 과정이다.

- **최단 경로 회전 (Shortest Path Rotation)**: 조향 모터가 목표 각도로 회전할 때, 현재 각도에서 가장 가까운 방향으로 회전하도록 하는 알고리즘이다. 예를 들어, 현재 각도가 10도이고 목표 각도가 350도일 때, 시계 방향으로 20도 회전하는 것이 반시계 방향으로 340도 회전하는 것보다 훨씬 효율적이다. 더 나아가, 회전 각도가 90도를 초과하는 경우에는 목표 각도를 180도 반전시키고 구동 휠의 회전 방향을 반대로 하는 것이 더 빠르다.8 예를 들어, 현재 각이 -89도에서 목표 각이 90도로 주어졌을 때, 179도를 회전하는 대신 목표 각을 -90도로 변경하고(-1도 회전) 휠 속도를 음수로 바꾸는 것이 훨씬 효율적이다.39

- **저속/정지 시 떨림(Jittering) 문제 해결**: 로봇이 정지하거나 매우 느린 속도로 움직일 때, 휠 속도는 거의 0이지만 조향각은 특정 방향을 유지해야 한다. 이때 PID 제어 오차나 최단 경로 회전 알고리즘의 경계 조건(±90도 근처)에서의 불안정성으로 인해 조향 모터가 특정 각도 주변에서 빠르게 진동하는 '떨림(jittering)' 현상이 발생할 수 있다.40 이를 해결하기 위해, 로봇의 전체 속도가 특정 임계값 이하일 때는 조향각을 변경하지 않도록 하거나, 조향 PID 제어기의 게인을 속도에 비례하여 동적으로 조절하는 기법을 적용할 수 있다.

- **코사인 보상 (Cosine Compensation)**: 조향각이 목표 각도에 아직 도달하지 않았을 때 구동 모터가 먼저 회전하면, 로봇은 의도치 않은 방향으로 미끄러지게 된다. 코사인 보상은 이러한 현상을 방지하기 위해, 실제 조향각과 목표 조향각 사이의 오차($\Delta\phi$)의 코사인 값($\cos(\Delta\phi)$)을 목표 구동 속도에 곱해주는 기법이다.39 각도 오차가 크면(

  $\Delta\phi \to 90^\circ$), 코사인 값은 0에 가까워져 구동 속도가 줄어들고, 오차가 없으면($\Delta\phi \to 0^\circ$), 코사인 값은 1이 되어 원래 속도로 구동된다. 이는 조향 동작과 구동 동작의 동기화를 통해 로봇의 움직임을 더 부드럽고 예측 가능하게 만든다.39

### 5.3 기구학적 특이점(Kinematic Singularity) 문제

- **정의 및 문제점**: 특이점은 로봇이 특정 자세에서 자유도를 잃어버리는 현상을 의미한다.41 스위블 드라이브에서는 모든 휠의 조향 방향이 로봇의 기하학적 중심과 같은 한 점을 향할 때 발생할 수 있다. 이 상태에서는 로봇이 제자리 회전은 쉽게 할 수 있지만, 특정 방향으로의 병진 운동을 생성하기 위해 무한대의 휠 속도가 필요하게 되어 사실상 해당 방향으로의 이동이 불가능해진다.42 특이점 근처에서 역기구학의 해가 불안정해지며, 매우 큰 조향 속도나 구동 속도를 요구하여 시스템에 무리를 주거나 제어 불능 상태에 빠질 수 있다.41
- **회피 전략**:
  - **알고리즘적 접근**: 제어 소프트웨어에서 순기구학의 자코비안 행렬($\mathbf{J}$)의 조작성 지수(manipulability index), 예를 들어 $\sqrt{\det(\mathbf{J}^T \mathbf{J})}$를 실시간으로 계산한다. 이 값이 특정 임계값 이하로 떨어지면 특이점에 가까워졌다고 판단하고, 감쇠 최소 자승법(Damped Least Squares)과 같은 기법을 사용하여 역기구학의 해가 발산하지 않도록 안정화시킨다.43
  - **경로 계획**: 상위 레벨의 경로 계획기(path planner)가 로봇의 기구학적 모델을 인지하고, 특이점을 유발하는 자세(예: 제자리에서 정지한 상태로 90도 회전)를 피하는 경로를 생성하도록 설계할 수 있다.

------

**표 3: `ekf.yaml` 구성 파라미터 상세 설명 (스위블 드라이브 기준)**

| 파라미터                   | 데이터 타입 | 예시 값     | 설명 및 스위블 드라이브 설정 시 고려사항                     |
| -------------------------- | ----------- | ----------- | ------------------------------------------------------------ |
| `frequency`                | double      | 30.0        | 필터의 업데이트 주기(Hz). 제어 루프 주기와 비슷하거나 약간 낮게 설정한다. |
| `two_d_mode`               | bool        | true        | **(필수)** 로봇이 2D 평면에서 작동한다고 가정한다. IMU의 roll/pitch 노이즈가 Z축 위치 추정에 미치는 악영향을 원천적으로 차단하여 안정성을 크게 향상시킨다.38 |
| `odom_frame`               | string      | "odom"      | 오도메트리 좌표계의 이름.                                    |
| `base_link_frame`          | string      | "base_link" | 로봇의 기준 좌표계 이름.                                     |
| `world_frame`              | string      | "odom"      | `map` 좌표계 없이 연속적인 위치 추정만 수행할 경우 `odom_frame`과 동일하게 설정한다. |
| `odom0`                    | string      | "/odom"     | `swerve_drive_controller`가 발행하는 오도메트리 토픽 이름.   |
| `odom0_config`             | bool        | ``          | X,Y 위치, Yaw, X,Y 선속도, Yaw 각속도만 사용. (X,Y,Z,R,P,Y, $\dot{X}$,$\dot{Y}$,$\dot{Z}$,$\dot{R}$,$\dot{P}$,$\dot{Y}$,$\ddot{X}$,$\ddot{Y}$,$\ddot{Z}$) |
| `odom0_differential`       | bool        | false       | `odom0`의 위치 정보를 절대값으로 사용할지, 차분(속도)으로 사용할지 결정. `false`로 설정하여 위치 정보를 직접 사용한다. |
| `imu0`                     | string      | "/imu/data" | IMU 센서가 발행하는 토픽 이름.                               |
| `imu0_config`              | bool        | ``          | 방향, 각속도, 선형 가속도 정보를 사용. (`two_d_mode`로 인해 R,P는 무시됨) |
| `imu0_differential`        | bool        | false       | IMU의 방향 정보를 절대값으로 사용. 만약 다른 절대 방향 센서와 충돌 시 `true`로 설정하여 차분(각속도)만 사용하도록 변경할 수 있다. |
| `process_noise_covariance` | double      | (대각 행렬) | 모델의 불확실성을 나타내는 공분산 행렬. 이 값을 조절하여 필터가 예측 모델과 센서 측정 중 어느 쪽을 더 신뢰할지 튜닝한다. |

 이 표는 `robot_localization` 패키지를 스위블 드라이브 로봇에 맞게 설정하는 데 필요한 핵심 파라미터와 그 의미를 상세히 설명한다. 올바른 설정은 위치 추정의 정확성과 안정성에 결정적인 영향을 미친다. 36

## 6. 결론: 종합 고찰 및 향후 연구 방향

### 개발 과정의 핵심 요약 및 기술적 기여

본 보고서는 ROS2 Humble 프레임워크를 기반으로 하는 전방향 4륜 스위블 드라이브 로봇의 개발 과정을 심층적으로 고찰하였다. 이 과정은 단순히 특정 기술의 나열이 아닌, 정밀한 기계 설계, 엄밀한 기구학적 모델링, 체계적인 소프트웨어 아키텍처 설계, 그리고 강인한 제어 알고리즘 구현이 유기적으로 결합되어야 하는 복합적인 시스템 엔지니어링 과제임을 명확히 보여주었다.

기술적 기여는 다음과 같이 요약할 수 있다. 첫째, 스위블 드라이브의 기계적 특성(동축/비동축, 백래시)이 소프트웨어 제어의 복잡도에 미치는 영향을 분석하고, 모듈화 설계가 개발 효율성과 유지보수성을 어떻게 향상시키는지 밝혔다. 둘째, 역기구학과 순기구학 모델을 상세히 유도하며, 특히 순기구학이 과결정 시스템으로서 의사 역행렬을 이용한 '추정'의 문제임을 수학적으로 규명하였다. 셋째, 현대 로보틱스 설계 패러다임을 반영하는 SBC-MCU 계층적 하드웨어 아키텍처와 `ros2_control`을 중심으로 한 모듈형 소프트웨어 아키텍처를 제시하였다. 이는 실시간성과 고성능 연산이라는 상충되는 요구사항을 효과적으로 분리하고, ROS 생태계와의 표준화된 연동을 가능하게 한다. 마지막으로, EKF 센서 융합을 통한 위치 추정 정밀도 향상, 최단 경로 회전 및 코사인 보상과 같은 실용적인 제어 최적화 기법, 그리고 기구학적 특이점 회피 전략을 제시하여 실제 운용 환경에서의 안정성과 성능을 확보하는 방안을 구체화하였다.

특히, 본 고찰 전반에 걸쳐 `ros2_control` 프레임워크가 스위블 드라이브의 본질적인 복잡성을 체계적으로 관리하고, 시뮬레이션과 실제 하드웨어 간의 간극을 최소화하는 'Sim-to-Real' 워크플로우를 가능하게 하는 핵심 기술임을 강조하였다. 이는 복잡한 로봇 시스템을 개발하는 데 있어 현대적인 개발 방법론의 중요성을 시사한다.

### 6.1 향후 과제 제시

본 보고서에서 제시된 개발 내용은 강인하고 정밀한 스위블 드라이브 플랫폼을 위한 견고한 기반을 제공한다. 이를 바탕으로 다음과 같은 후속 연구 및 개발을 통해 시스템을 더욱 고도화할 수 있다.

- **Nav2 스택과의 완전한 통합 및 최적화**: 개발된 `swerve_drive_controller`를 ROS2의 표준 내비게이션 스택인 Nav2와 완전히 통합하여 완전한 자율 주행 시스템을 구축하는 것이 다음 목표가 될 것이다. 특히, 스위블 드라이브의 홀로노믹 기동성을 최대한 활용하기 위해 SMAC Planner(격자 기반의 다방향 탐색 가능)나 MPPI Controller(모델 예측 기반의 최적 제어)와 같은 최신 플래너 및 컨트롤러 플러그인을 적용하고, 그 파라미터를 스위블 드라이브의 특성에 맞게 튜닝하는 연구가 필요하다.30
- **동역학 모델 기반 제어 (Dynamics-based Control)**: 현재의 기구학 모델은 로봇의 움직임을 기하학적으로만 다루며, 질량, 관성, 마찰, 모터 토크 한계와 같은 물리적 특성은 고려하지 않는다. 더 높은 속도와 가속도에서 정밀한 제어를 달성하기 위해서는 로봇의 동역학 모델을 수립하고, 이를 기반으로 한 고급 제어 기법(예: 모델 예측 제어(MPC), 슬라이딩 모드 제어)을 적용해야 한다.45 이는 외부 힘에 대한 강인성을 높이고, 에너지 효율을 최적화하는 데에도 기여할 것이다.
- **하드웨어 시스템의 고도화**: 현재의 프로토타입 수준을 넘어, 3D 프린팅 부품을 CNC로 가공된 알루미늄 부품으로 대체하여 시스템 전체의 강성과 정밀도를 향상시킬 수 있다. 또한, 각 모듈의 모터 드라이버, 엔코더 처리 회로, MCU를 하나의 맞춤형 PCB(Printed Circuit Board)로 통합 설계하면 시스템의 크기와 무게를 줄이고, 배선의 복잡성을 낮추어 전체적인 신뢰성을 크게 향상시킬 수 있다.

이러한 향후 과제들은 스위블 드라이브 로봇을 단순한 연구 플랫폼을 넘어, 실제 산업 및 서비스 현장에서 신뢰성 있게 운용될 수 있는 완성도 높은 제품으로 발전시키는 데 필수적인 단계가 될 것이다.

#### **참고 자료**

1. A brief overview of holonomic systems and their usability in robotics, 8월 17, 2025에 액세스, https://www.wevolver.com/article/holonomic-robot
2. Pros and cons for different types of drive selection - Robohub, 8월 17, 2025에 액세스, https://robohub.org/pros-and-cons-for-different-types-of-drive-selection/
3. International Journal of Applied Science-Research and Review Commentary Advantage and Application of Holonomic Vehicles and Cons - Prime Scholars, 8월 17, 2025에 액세스, https://www.primescholars.com/articles/advantage--and-application-of-holonomic-vehicles-and-constantfactor-transmission-cvt-in-the-vehicle-concept.pdf
4. How To Build a Swerve-Drive Robot - Fresh Consulting, 8월 17, 2025에 액세스, https://www.freshconsulting.com/insights/blog/how-to-build-a-swerve-drive-robot/
5. Holonomic Drivetrains - Game Manual 0, 8월 17, 2025에 액세스, https://gm0.org/en/latest/docs/common-mechanisms/drivetrains/holonomic.html
6. The advantage of holonomic drivetrains - Technical Discussion - Chief Delphi, 8월 17, 2025에 액세스, https://www.chiefdelphi.com/t/the-advantage-of-holonomic-drivetrains/71171
7. Swerve Central - DEW Robotics, 8월 17, 2025에 액세스, http://team1640.com/wiki/index.php/Swerve_Central
8. Using Inverse Kinematics to become a Master-Swerver | by Abhinav Peri | Medium, 8월 17, 2025에 액세스, https://abhinavwastaken.medium.com/using-inverse-kinematics-to-become-a-master-swerver-1026759d81b0
9. Compact Shaft-Rotating Swerve Drive with Prong Structure for Highly-Maneuverable and Agile Robots - Athens Journal, 8월 17, 2025에 액세스, https://www.athensjournals.gr/technology/2022-9-1-2-Vranas.pdf
10. Wheel Arrangement of Four Omni Wheel Mobile Robot for Compactness - MDPI, 8월 17, 2025에 액세스, https://www.mdpi.com/2076-3417/12/12/5798
11. james-yoo/swerve_drive: ROS based independent steering and drive implementation with Dynamixel - GitHub, 8월 17, 2025에 액세스, https://github.com/james-yoo/swerve_drive
12. Swerve drive - Kinematics simulation - Mind vortex, 8월 17, 2025에 액세스, https://www.petrikvandervelde.nl/posts/Swerve-drive-kinematics-simulation
13. Features Status - ROS 2 Documentation: Humble documentation, 8월 17, 2025에 액세스, https://docs.ros.org/en/humble/The-ROS2-Project/Features.html
14. 로봇공학자, 8월 17, 2025에 액세스, https://rosmaster.tistory.com/
15. Demos - ROS2_Control: Humble Aug 2025 documentation, 8월 17, 2025에 액세스, https://control.ros.org/humble/doc/ros2_control_demos/doc/index.html
16. Getting Started - ROS2_Control: Humble Aug 2025 documentation, 8월 17, 2025에 액세스, https://control.ros.org/humble/doc/getting_started/getting_started.html
17. ros2_control Concepts & Simulation - Articulated Robotics, 8월 17, 2025에 액세스, https://articulatedrobotics.xyz/tutorials/mobile-robot/applications/ros2_control-concepts/
18. Welcome to the ros2_control documentation! - ROS2_Control: Rolling Aug 2025 documentation, 8월 17, 2025에 액세스, https://control.ros.org/
19. ROS 2 Control, Robot Control the Right Way | by Jiayi Hoffman - Medium, 8월 17, 2025에 액세스, https://medium.com/@jiayi.hoffman/ros-2-control-robot-control-the-right-way-d0e72e7f1b6c
20. Example 7: Full tutorial with a 6DOF robot - ROS2_Control, 8월 17, 2025에 액세스, https://control.ros.org/rolling/doc/ros2_control_demos/example_7/doc/userdoc.html
21. Coaxial Swerve Explanation - CAD - Chief Delphi, 8월 17, 2025에 액세스, https://www.chiefdelphi.com/t/coaxial-swerve-explanation/439591
22. Swerve module physically unable to drive and rotate at the same time - Chief Delphi, 8월 17, 2025에 액세스, https://www.chiefdelphi.com/t/swerve-module-physically-unable-to-drive-and-rotate-at-the-same-time/482808
23. Building Cheapest ROS2 Robot using ESP32 - Part 1 - Hardware Build | by RoboFoundry, 8월 17, 2025에 액세스, https://robofoundry.medium.com/building-cheapest-ros2-robot-using-esp32-part-1-hardware-build-af0044de68ce
24. Swerve Robotic Platform, 8월 17, 2025에 액세스, https://swerveroboticsystems.github.io/DDR/Software/Software.pdf
25. pvandervelde/zinger_swerve_controller: The swerve controller code for the zinger robot - GitHub, 8월 17, 2025에 액세스, https://github.com/pvandervelde/zinger_swerve_controller
26. Explicit Steering (4 Wheel Steering / Drive) Analysis - Projects - Open Robotics Discourse, 8월 17, 2025에 액세스, https://discourse.openrobotics.org/t/explicit-steering-4-wheel-steering-drive-analysis/15264
27. 4 & 6 Wheel Robots - RobotShop, 8월 17, 2025에 액세스, https://www.robotshop.com/collections/4-wheeled-development-platforms
28. ROSbot 3 (and previous 2x versions ) - Husarion, 8월 17, 2025에 액세스, https://husarion.com/manuals/rosbot/
29. linorobot/linorobot2: Autonomous mobile robots (2WD, 4WD, Mecanum Drive) - GitHub, 8월 17, 2025에 액세스, https://github.com/linorobot/linorobot2
30. Swerve drive - Using Nav2 - Mind vortex, 8월 17, 2025에 액세스, https://www.petrikvandervelde.nl/posts/Swerve-drive-ros2-nav
31. Robotics - Making URDF models - Mind vortex, 8월 17, 2025에 액세스, https://www.petrikvandervelde.nl/posts/Robotics-making-urdf-models
32. 1월 1, 1970에 액세스, [https.petrikvandervelde.nl/posts/Robotics-making-urdf-models](http://docs.google.com/https.petrikvandervelde.nl/posts/Robotics-making-urdf-models)
33. ros-controls/ros2_control_demos: This repository aims at providing examples to illustrate ros2_control and ros2_controllers - GitHub, 8월 17, 2025에 액세스, https://github.com/ros-controls/ros2_control_demos
34. ROS2 Control with the JetBot Part 2: Building a ros2_control System | Mike Likes Robots, 8월 17, 2025에 액세스, https://mikelikesrobots.github.io/blog/jetbot-motors-pt2/
35. ros-controls/gazebo_ros2_control: Wrappers, tools and additional API's for using ros2_control with Gazebo Classic - GitHub, 8월 17, 2025에 액세스, https://github.com/ros-controls/gazebo_ros2_control
36. Sharpen Your Robot's Eyes – Mastering ROS2 Sensor Fusion with EKF & AMCL - Robotisim, 8월 17, 2025에 액세스, https://robotisim.com/ros2-sensor-fusion-ekf-amcl/
37. norsechurros/Ekf-Fusion: ekfFusion is a ROS package for sensor fusion using the Extended Kalman Filter (EKF). It integrates IMU, GPS, and odometry data to estimate the pose of robots or vehicles. With ROS integration and support for various sensors, ekfFusion provides reliable localization for robotic applications. - GitHub, 8월 17, 2025에 액세스, https://github.com/norsechurros/Ekf-Fusion
38. Sensor Fusion Using the Robot Localization Package – ROS 2, 8월 17, 2025에 액세스, https://automaticaddison.com/sensor-fusion-using-the-robot-localization-package-ros-2/
39. Swerve Drive Kinematics - FIRST Robotics Competition ..., 8월 17, 2025에 액세스, https://docs.wpilib.org/en/stable/docs/software/kinematics-and-odometry/swerve-drive-kinematics.html
40. Swerve Drive Optimization Jittering - Java - Chief Delphi, 8월 17, 2025에 액세스, https://www.chiefdelphi.com/t/swerve-drive-optimization-jittering/452323
41. Robot Singularities: What Are They and How to Beat Them - RoboDK blog, 8월 17, 2025에 액세스, https://robodk.com/blog/robot-singularities/
42. Singularity avoidance for over-actuated, pseudo-omnidirectional, wheeled mobile robots | Request PDF - ResearchGate, 8월 17, 2025에 액세스, https://www.researchgate.net/publication/221075729_Singularity_avoidance_for_over-actuated_pseudo-omnidirectional_wheeled_mobile_robots
43. How to deal with robotic singularities - Reddit, 8월 17, 2025에 액세스, https://www.reddit.com/r/robotics/comments/9hrrxx/how_to_deal_with_robotic_singularities/
44. Four wheel independent steering robot using Nav2 and ros2-control - Projects, 8월 17, 2025에 액세스, https://discourse.openrobotics.org/t/four-wheel-independent-steering-robot-using-nav2-and-ros2-control/34587
45. Motion Planning and Control of an Omnidirectional Mobile Robot in Dynamic Environments, 8월 17, 2025에 액세스, https://www.mdpi.com/2218-6581/10/1/48
46. Omni-directional mobile robot controller based on trajectory linearization - Ohio University, 8월 17, 2025에 액세스, https://people.ohio.edu/williams/html/PDF/JRAS08.pdf


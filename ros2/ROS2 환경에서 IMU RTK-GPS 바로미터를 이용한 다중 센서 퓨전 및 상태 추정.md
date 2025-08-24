[ROS2 (Robot Operating System 2)](./index.md)
# ROS2 환경에서 IMU, RTK-GPS, 바로미터를 이용한 다중 센서 퓨전 및 상태 추정



이동 로봇 공학의 근본적인 도전 과제는 로봇이 자신의 상태(state), 즉 "자신이 어디에 있고 어떻게 움직이는가"를 정확하게 인지하는 것입니다. 이 상태 정보 없이는 자율 주행, 물체 조작, 경로 계획 등 의미 있는 고차원적 작업을 수행하는 것이 불가능합니다. 로봇의 상태는 위치, 자세, 속도, 각속도 등 다양한 변수로 구성되며, 이러한 변수들을 실시간으로 정밀하게 추정하는 것이 모든 자율 시스템의 핵심 기반 기술입니다.


개별 센서는 본질적인 한계를 가집니다. 관성 측정 장치(IMU)는 높은 주기로 상대적인 움직임을 측정하지만, 시간이 지남에 따라 오차가 누적되어 드리프트(drift)가 발생합니다. 반면, 실시간 이동 측위(RTK-GPS)와 같은 위성 항법 시스템은 전역 좌표계에서 절대 위치를 제공하지만, 신호 수신이 불가능한 환경(예: 실내, 터널, 도심 협곡)이 존재하고 데이터 주기가 낮으며 불연속적인 값 점프(jump)가 발생할 수 있습니다. 이러한 개별 센서의 단점을 상호 보완하고 장점을 극대화하기 위한 유일하고 강력한 해결책이 바로 센서 퓨전(sensor fusion)입니다. 센서 퓨전은 여러 센서로부터의 불완전하고 노이즈가 섞인 데이터를 통계적으로 결합하여, 단일 센서만으로는 얻을 수 없는 더 정확하고 강건하며 신뢰성 높은 상태 추정치를 생성하는 과정입니다.


로봇 운영 체제(ROS)는 로봇 소프트웨어 개발을 위한 사실상의 표준 프레임워크로 자리 잡았으며, ROS2는 실시간성, 보안, 멀티 로봇 시스템 지원 등을 강화한 차세대 버전입니다.1 이 방대한 ROS2 생태계 내에서 상태 추정을 위한 핵심 패키지는 `robot_localization`입니다.2 이 패키지는 비선형 상태 추정기(확장 칼만 필터 및 무향 칼만 필터)를 구현한 노드들을 제공하며, 임의의 개수의 센서 입력을 융합하여 로봇의 3차원 공간상의 상태를 지속적으로 추정하는 강력한 기능을 제공합니다.2


본 보고서는 IMU, RTK-GPS, 그리고 바로미터(barometer)를 사용하는 로봇 시스템에서 `robot_localization` 패키지를 중심으로 정확한 위치, 자세, 속도, 각속도를 추정하는 방법에 대한 포괄적이고 전문적인 가이드를 제공하는 것을 목표로 합니다. 독자는 기본적인 ROS2 지식을 갖춘 로봇 공학자 또는 대학원생으로 가정하며, 보고서는 단순한 사용법 나열을 넘어 이론적 토대부터 실제 구현, 튜닝, 검증에 이르는 전 과정을 심도 있게 다룰 것입니다. 보고서는 다음과 같은 구조로 구성됩니다: 상태 추정의 기초 개념 정립, `robot_localization` 패키지의 상세 분석, GPS 통합을 위한 이중 EKF(Dual-EKF) 아키텍처 설계, 각 센서의 구체적인 통합 방법론, 가장 어려운 과제인 공분산 튜닝 기법, 그리고 마지막으로 전체 시스템의 구현 및 검증 절차를 제시합니다.

------


성공적인 상태 추정 시스템을 구축하기 위해서는 먼저 ROS에서 사용되는 "공간의 언어"와 상태 추정의 기본 이론을 명확히 이해해야 합니다. 이 섹션에서는 그 필수적인 이론적, 규약적 토대를 마련합니다.


`robot_localization` 패키지는 로봇의 상태를 총 15개의 변수로 구성된 벡터로 추적합니다. 이 상태 벡터는 다음과 같이 구성됩니다 5:

- **위치 (Position):** X,Y,Z
- **자세 (Orientation):** Roll, Pitch, Yaw ($ \phi, \theta, \psi $)
- **선형 속도 (Linear Velocity):** vX,vY,vZ
- **각속도 (Angular Velocity):** vRoll,vPitch,vYaw
- **선형 가속도 (Linear Acceleration):** aX,aY,aZ

이처럼 포괄적인 상태를 추적하는 이유는 단순히 현재 위치와 자세를 아는 것을 넘어, 로봇의 동역학적 상태를 완벽하게 모델링하기 위함입니다. 특히 속도와 가속도를 상태 벡터에 포함함으로써, 필터는 센서 데이터가 일시적으로 지연되거나 수신되지 않는 상황에서도 내부 모션 모델을 통해 로봇의 상태를 예측(predict)할 수 있습니다.2 이는 센서 데이터의 공백 기간 동안에도 추정치가 연속성을 유지하게 하여 시스템의 강건성을 크게 향상시킵니다.


ROS에서 좌표계(coordinate frame)에 대한 이해 부족은 시스템 실패의 가장 흔한 원인 중 하나입니다. REP-105는 이동 로봇을 위한 표준 좌표계를 정의하며, `robot_localization`은 이를 엄격히 따릅니다. 핵심적인 세 가지 좌표계는 다음과 같습니다.

- **`base_link`:** 로봇의 몸체에 단단히 고정된, 로봇 자체의 기준 좌표계입니다. 모든 지역적 움직임과 센서 측정값은 궁극적으로 이 좌표계를 기준으로 표현됩니다.8

- **`odom`:** 월드 고정(world-fixed) 좌표계이지만, 시간이 지남에 따라 드리프트가 누적될 수 있습니다. 이 좌표계의 목적은 점프(jump)가 허용되지 않는 지역적 경로 계획이나 시각 서보(visual servoing)와 같은 작업에 연속적이고 단기적으로 정확한 기준을 제공하는 것입니다.9

  `odom` -> `base_link` 변환(transform)은 로봇의 주행 거리계(odometry) 기반 자세를 나타냅니다.

- **`map`:** `odom`과 마찬가지로 월드 고정 좌표계이지만, 드리프트가 없어야 하며 전역적으로 정확해야 합니다. 대신 불연속적인 점프가 발생할 수 있습니다. 이 좌표계의 목적은 알려진 환경에서 장기적인 항법을 위한 드리프트 없는 전역 기준을 제공하는 것입니다. 이 프레임은 `odom` 프레임에 누적된 드리프트를 보정하는 역할을 합니다.9

이 세 좌표계는 `map` -> `odom` -> `base_link` 형태의 TF(Transform) 트리를 구성합니다. 이 구조는 직관적이지 않을 수 있지만, 드리프트 보정을 효율적으로 처리하기 위한 의도적인 설계입니다.12

`map` -> `odom` 변환은 주행 거리계의 누적된 드리프트를 나타내며, 이 드리프트는 GPS와 같은 전역 센서에 의해 보정됩니다. 따라서 RViz와 같은 시각화 도구에서 `map`과 `odom` 좌표계의 분리 정도를 관찰하는 것은 전체 측위 시스템의 성능을 진단하는 핵심적인 방법론이 됩니다. 드리프트가 심한 주행 거리계 소스를 사용하거나 전역 보정이 부정확하면 두 프레임은 빠르게 멀어질 것입니다.11


상태 추정에는 크게 두 가지 접근 방식이 존재합니다.

- **필터링 (Filtering, 칼만 필터):** 필터링의 핵심 원리는 과거의 모든 측정값을 기반으로 *현재* 상태를 추정하는 것입니다. 이는 새로운 측정값이 들어올 때마다 상태를 갱신하고, 이전 측정값은 버리는 재귀적인(recursive) 프로세스입니다. `robot_localization` 패키지는 이 필터링 접근법을 사용합니다.13
  - **장점:** 계산적으로 효율적이어서 실시간 애플리케이션에 매우 적합합니다.
  - **단점:** 루프 클로저(loop closure)와 같이 미래의 정보를 사용하여 과거의 상태를 다시 수정할 수 없습니다. 추정치는 오직 과거 정보에 대해서만 최적입니다.
- **스무딩 (Smoothing, 팩터 그래프):** 스무딩의 원리는 특정 시간 창(window) 내의 과거, 현재, 미래의 모든 측정값을 기반으로 *전체 궤적*을 추정하는 것입니다. 이는 모든 제약 조건을 하나의 거대한 최적화 문제로 구성하여 해결합니다. GTSAM이나 `fuse`와 같은 라이브러리가 이 접근법을 사용합니다.14
  - **장점:** 모든 가용 정보를 사용하여 전체 경로를 최적화하므로 일반적으로 더 높은 정확도를 보입니다. 순서가 뒤바뀐 데이터나 루프 클로저를 자연스럽게 처리할 수 있습니다.
  - **단점:** 계산 복잡도가 높고, 실시간 시스템에 구현하기 더 복잡할 수 있습니다.

본 보고서는 널리 사용되는 `robot_localization` (필터링)에 초점을 맞추지만, ROS 커뮤니티가 점차 고정밀 추정을 위한 미래 표준으로 `fuse`와 같은 스무딩 기반 접근법을 모색하고 있다는 점을 인지하는 것이 중요합니다.15 이는 사용자에게 현재 기술의 위치와 미래 발전 방향에 대한 중요한 맥락을 제공합니다.

`robot_localization` 내에서도 확장 칼만 필터(EKF)와 무향 칼만 필터(UKF) 중 하나를 선택해야 합니다. 이 선택은 단순히 정확도 문제가 아니라, 시스템의 비선형성 처리 능력과 설정의 복잡성 사이의 트레이드오프입니다. EKF는 1차 테일러 급수 전개(자코비안 행렬)를 통해 모션/센서 모델을 선형화하는데, 이는 로봇의 고속 기동이나 복잡한 센서 모델과 같이 비선형성이 강한 시스템에서 큰 오차를 유발할 수 있습니다.13 반면, UKF는 결정론적 샘플링 기법(무향 변환)을 사용하여 시그마 포인트들을 실제 비선형 모델에 직접 통과시키므로, 비선형 시스템에서 일반적으로 더 나은 정확도를 보입니다.13 하지만 UKF는 

`alpha`, `kappa`, `beta`와 같은 추가적인 튜닝 파라미터를 가지며, 이를 올바르게 조정하려면 필터의 수학적 원리에 대한 깊은 이해가 필요합니다.17 따라서, 단순한 기구학을 가진 대부분의 표준 지상 로봇에게는 EKF가 충분하고 설정이 더 용이하며, 드론이나 고속 차량과 같이 비선형성이 두드러지는 동적인 시스템에는 UKF가 더 선호되는 선택입니다.

------


이 섹션에서는 일반적인 이론에서 벗어나 사용자가 실제로 사용할 도구인 `robot_localization` 패키지를 심층적으로 분석합니다.


앞서 언급했듯이, 이 두 노드는 각각 확장 칼만 필터(EKF)와 무향 칼만 필터(UKF)의 구현체입니다.2 두 노드는 공통된 파라미터 집합과 인터페이스를 공유하므로, 사용자는 비교적 쉽게 두 필터 사이를 전환하며 테스트할 수 있습니다.2 이 패키지의 핵심적인 특징은 다음과 같습니다:

- 임의의 개수 센서 융합 지원
- 다양한 ROS 메시지 타입 지원 (`nav_msgs/Odometry`, `sensor_msgs/Imu`, `geometry_msgs/PoseWithCovarianceStamped`, `geometry_msgs/TwistWithCovarianceStamped`)
- 센서별 데이터 선택적 퓨전 기능 2


이 YAML 설정 파일은 패키지의 심장부와 같습니다. 가장 중요한 파라미터들을 단계별로 살펴보겠습니다.

- **핵심 파라미터:**
  - `frequency`: 필터가 상태 추정치를 출력하는 주파수(Hz)입니다.
  - `sensor_timeout`: 특정 센서로부터 데이터가 이 시간(초) 동안 수신되지 않으면 타임아웃으로 간주합니다.
  - `two_d_mode`: `true`로 설정하면 Z, roll, pitch 및 그 도함수들을 0으로 강제하여 2D 평면 환경에 최적화합니다. 본 보고서의 3D 문제에서는 반드시 `false`로 설정해야 합니다.13
  - `publish_tf`: `true`일 경우, `world_frame`에서 `base_link_frame`으로의 TF 변환을 발행합니다.
  - `publish_acceleration`: `true`일 경우, 추정된 가속도 상태를 `accel/filtered` 토픽으로 발행합니다.17
- **좌표계 파라미터:**
  - `map_frame`, `odom_frame`, `base_link_frame`: 시스템의 TF 트리에 맞는 좌표계 이름을 지정합니다.
  - `world_frame`: 이 파라미터는 필터가 어떤 "모드"로 동작할지 결정하는 핵심입니다. `odom`으로 설정하면 지역적이고 연속적인 필터로, `map`으로 설정하면 전역적이고 불연속적인 필터로 동작합니다.9 이 파라미터가 바로 이중 EKF 아키텍처를 구현하는 열쇠입니다.
- **센서 입력 파라미터:**
  - `odomN`, `imuN`, `poseN`, `twistN`: `N`을 0부터 증가시키며 여러 센서 입력을 추가할 수 있습니다. 값으로는 해당 센서 데이터가 발행되는 토픽 이름을 지정합니다.6


이 파라미터는 `robot_localization`에서 가장 중요하면서도 가장 자주 잘못 설정되는 부분입니다.

- **15개 요소의 불리언 벡터:** 이 벡터는 상태 벡터의 15개 변수 각각에 대해 해당 센서의 측정값을 퓨전할지 여부를 결정합니다. 순서는 다음과 같습니다 5:

  1. **X** (위치)
  2. **Y** (위치)
  3. **Z** (위치)
  4. **Roll** (자세)
  5. **Pitch** (자세)
  6. **Yaw** (자세)
  7. **vX** (선형 속도)
  8. **vY** (선형 속도)
  9. **vZ** (선형 속도)
  10. **vRoll** (각속도)
  11. **vPitch** (각속도)
  12. **vYaw** (각속도)
  13. **aX** (선형 가속도)
  14. **aY** (선형 가속도)
  15. **aZ** (선형 가속도) 5

- 구체적인 예시: 바로미터와 같은 pose0 소스로부터 Z축 위치 정보만 퓨전하고 싶다면, pose0_config는 다음과 같이 설정합니다:

  [false, false, true, false, false, false, false, false, false, false, false, false, false, false, false].6

- **`_differential` 및 `_relative` 파라미터:**

  - `_differential`: 이 값을 `true`로 설정하면, 해당 센서의 절대 위치/자세 측정값은 연속된 두 메시지 간의 차이를 계산하여 속도로 변환된 후 퓨전됩니다. 이는 두 개의 IMU로부터 절대 자세를 입력받는 경우와 같이, 여러 개의 절대 자세 소스를 퓨전할 때 필터가 두 값 사이에서 진동하는 것을 방지하는 강력한 도구입니다. 경험적으로, 특정 자세 변수에 대해 하나의 비-미분(non-differential) 소스만 두는 것이 좋습니다.13
  - `_relative`: 이 파라미터는 시작 시점의 센서 측정값을 0으로 만들어, 이후의 모든 측정값이 시작점 기준의 상대값으로 처리되도록 합니다. `_differential`과의 미묘하지만 중요한 차이점은, `relative`는 측정값을 속도로 변환하지 않는다는 것입니다.6

`_config` 행렬은 단순한 스위치가 아니라, 센서 측정값의 어떤 부분을 신뢰할 것인지에 대한 선언입니다. 무엇을 퓨전할지 결정하는 것은 공분산을 튜닝하는 것만큼이나 중요합니다. 흔한 실수는 단일 센서 소스로부터 중복된 정보를 퓨전하는 것입니다. 예를 들어, 휠 오도메트리 메시지는 위치(X, Y, Yaw)와 속도(vX, vYaw)를 모두 제공할 수 있지만, 위치는 보통 속도를 단순히 적분한 값입니다.13 두 값을 모두 퓨전하는 것은 필터에 상관된 노이즈와 중복된 정보를 주입하는 셈입니다. 가장 좋은 방법은 가장 "직접적인" 측정값을 퓨전하는 것입니다. 즉, 엔코더로부터는 속도를, IMU로부터는 자세를 퓨전하는 것이 일반적입니다.21 따라서 

`_config` 행렬은 사용자가 각 메시지 필드의 물리적 출처와 신뢰도에 대해 의식적인 결정을 내리도록 강제하며, 이는 단순히 숫자를 튜닝하는 것보다 더 근본적인 단계입니다.

또한 `_differential` 파라미터는 여러 절대 자세 센서 간의 충돌을 해결하는 핵심 메커니즘이지만, 해당 센서 데이터에 대한 적분 드리프트를 유발하는 대가를 치릅니다. 예를 들어, 두 IMU가 절대 자세를 제공할 때, 두 센서의 측정값이 약간 다르면 필터가 진동할 수 있습니다.13 한 센서에 

`_differential: true`를 설정하면 이 문제가 해결되지만, 이는 해당 센서의 데이터가 이제 시간에 따라 적분됨을 의미합니다. 만약 그 센서에 약간의 바이어스가 있었다면, 그 바이어스는 이제 필터 상태에서 계속해서 커지는 자세 오차로 누적될 것입니다.13 이는 중요한 트레이드오프를 만듭니다: 진동을 멈추기 위해 `_differential`을 사용하되, 이제 장기적인 드리프트 보정을 다른 절대 센서에 전적으로 의존하게 된다는 점을 받아들여야 합니다. 이 선택은 장기적으로 어떤 센서가 더 신뢰할 수 있는지에 따라 내려져야 합니다.


이 표는 YAML 파일의 가장 중요한 파라미터들에 대한 빠른 참조 가이드 역할을 합니다.

| 파라미터 이름                 | 설명                                                      | 기본값       | 주요 고려사항                                                |
| ----------------------------- | --------------------------------------------------------- | ------------ | ------------------------------------------------------------ |
| `frequency`                   | 필터 출력 주파수(Hz).13                                   | 30.0         | 시스템의 실시간 요구사항과 계산 부하에 맞춰 설정합니다.      |
| `two_d_mode`                  | `true`일 경우 3D 변수(Z, Roll, Pitch 등)를 0으로 강제.13  | `false`      | 3D 환경에서는 반드시 `false`로 설정해야 합니다.              |
| `publish_tf`                  | `world_frame` -> `base_link_frame` TF 발행 여부.13        | `true`       | 이중 EKF 설정에서는 각 필터의 역할에 맞게 설정해야 합니다.   |
| `world_frame`                 | 필터의 기준 월드 좌표계. `odom` 또는 `map`.9              | `odom`       | 지역 필터는 `odom`, 전역 필터는 `map`으로 설정합니다.        |
| `[sensor]N`                   | 센서 입력 토픽 이름. 예: `odom0`, `imu0`.6                | 없음         | 사용하는 모든 센서에 대해 지정해야 합니다.                   |
| `[sensor]N_config`            | 15개 상태 변수에 대한 퓨전 여부를 결정하는 불리언 행렬.6  | 모두 `false` | 각 센서에서 신뢰할 수 있는 측정값만 `true`로 설정합니다.     |
| `[sensor]N_differential`      | `true`일 경우, 위치/자세 데이터를 속도로 변환하여 퓨전.13 | `false`      | 여러 개의 절대 자세 소스를 퓨전할 때 진동을 막기 위해 사용합니다. |
| `[sensor]N_relative`          | `true`일 경우, 첫 측정값을 기준으로 상대적인 값만 퓨전.6  | `false`      | 절대값이 아닌 상대적인 변화량이 중요할 때 사용합니다.        |
| `process_noise_covariance`    | 예측 모델의 불확실성을 나타내는 공분산 행렬(Q).22         | 작은 값      | 필터의 반응성과 안정성 사이의 트레이드오프를 결정하는 핵심 튜닝 파라미터입니다. |
| `initial_estimate_covariance` | 초기 상태 추정치의 불확실성을 나타내는 공분산 행렬(P).22  | 큰 값        | 초기 수렴 속도에 영향을 미칩니다.                            |

------


이 섹션에서는 GPS와 같은 전역 센서를 통합하기 위한 표준 아키텍처인 이중 EKF(Dual-EKF)의 "왜"와 "어떻게"를 상세히 설명합니다.


핵심 문제는 센서의 특성 차이에서 비롯됩니다. GPS 데이터는 전역적으로 정확하고 드리프트가 없지만, 주파수가 낮고 지연 시간이 길며, 수신 환경에 따라 값이 갑자기 튀는 불연속적인 특성을 가집니다.9 반면 IMU나 휠 오도메트리와 같은 연속적인 센서들은 주파수가 높고 부드러운 데이터를 제공하지만, 시간이 지남에 따라 오차가 누적되어 드리프트가 발생합니다.9

만약 단일 필터에서 이 두 종류의 데이터를 모두 융합하려 한다면, GPS의 불연속적인 점프가 필터의 상태 추정치 전체에 영향을 미쳐, 최종 출력이 "들쭉날쭉(jagged)"하게 됩니다. 이러한 출력은 부드럽고 연속적인 자세 정보를 요구하는 지역 경로 계획기(local planner)에는 적합하지 않습니다.10

이중 EKF 설정은 이러한 문제를 우아하게 해결합니다. 역할을 분리하여, 하나는 부드러운 지역 오도메트리를, 다른 하나는 전역적으로 정확한 자세를 담당하게 합니다.10


- **역할:** 부드럽고, 연속적이며, 높은 주기의 지역 오도메트리를 제공하는 것이 주 목적입니다. 이 추정치는 단기적으로는 정확하지만 장기적으로는 드리프트됩니다.
- **입력:** 모든 **연속적인** 센서 데이터가 입력됩니다. 본 보고서의 경우, IMU의 각속도와 선형 가속도, 그리고 가능하다면 휠 오도메트리 데이터가 여기에 해당합니다. **결정적으로, GPS나 바로미터 데이터는 이 필터에 절대로 입력되지 않습니다**.10
- **설정:** YAML 파일에서 `world_frame: odom`으로 설정합니다. 이 노드는 `odom` -> `base_link` TF 변환을 발행할 책임이 있습니다.24
- **출력:** `nav_msgs/Odometry` 메시지를 `/odometry/filtered/local`과 같은 토픽으로 발행합니다.


- **역할:** 전역적으로 정확하고 드리프트가 없는 상태 추정치를 제공하는 것이 주 목적입니다. 이 추정치는 GPS 데이터로 인해 불연속적인 점프를 포함할 수 있습니다.
- **입력:** **모든** 센서 데이터가 입력됩니다. 여기에는 연속 센서(IMU 등)뿐만 아니라, `navsat_transform_node`를 통해 변환된 GPS 데이터와 바로미터의 고도 데이터와 같은 절대 측정값이 포함됩니다.10
- **설정:** YAML 파일에서 `world_frame: map`으로 설정합니다. 이 노드는 `map` -> `base_link`를 직접 발행하지 않습니다. 대신, 지역 EKF가 발행하는 `odom` -> `base_link` TF를 구독(lookup)하여, 그 둘 사이의 차이, 즉 누적된 드리프트를 보정하는 `map` -> `odom` TF 변환을 발행합니다.9
- **출력:** `nav_msgs/Odometry` 메시지를 `/odometry/filtered/global`과 같은 토픽으로 발행합니다.


전체 시스템의 데이터 흐름과 TF 트리가 어떻게 합성되는지 단계별로 설명하면 다음과 같습니다.

1. **연속 센서 -> 지역 EKF:** IMU와 같은 연속 센서 데이터가 지역 EKF(`ekf_odom`)로 입력됩니다.
2. **지역 EKF -> TF 및 오도메트리 발행:** 지역 EKF는 `/odometry/filtered/local` 토픽과 `odom` -> `base_link` TF 변환을 발행합니다.
3. **GPS/IMU -> `navsat_transform_node`:** GPS 수신기의 `/gps/fix` 토픽과 IMU의 `/imu/data`(헤딩 정보용) 토픽이 `navsat_transform_node`로 입력됩니다. 이때, 이 노드는 초기 변환을 설정하기 위해 전역 EKF의 출력(`/odometry/filtered/global`)도 필요로 합니다. 이는 `datum` 메커니즘에 의해 해결되는 미묘한 순환 의존성입니다.25
4. **`navsat_transform_node` -> GPS 오도메트리 발행:** `navsat_transform_node`는 변환된 GPS 위치를 담은 `/odometry/gps` 토픽을 발행합니다.
5. **바로미터 -> 고도 자세 발행:** 바로미터 데이터를 처리하는 별도의 노드가 고도 정보를 담은 `/pose/altitude` 토픽(`geometry_msgs/PoseWithCovarianceStamped`)을 발행합니다.
6. **모든 센서 -> 전역 EKF:** IMU, `/odometry/gps`, `/pose/altitude` 등 모든 센서 데이터가 전역 EKF(`ekf_map`)로 입력됩니다.
7. **전역 EKF -> TF 보정 발행:** 전역 EKF는 2단계에서 발행된 `odom` -> `base_link` TF를 조회하고, 이를 기반으로 드리프트를 보정하는 `map` -> `odom` TF를 발행합니다. 이로써 최종적으로 `map` -> `odom` -> `base_link` 체인이 완성됩니다.

이 구조에서 전역 측위 루프에는 미묘하지만 중요한 순환 의존성이 존재합니다. `navsat_transform_node`는 첫 GPS 측정값을 올바르게 배치하기 위해 전역 자세가 필요하고, 전역 EKF는 전역 자세를 생성하기 위해 변환된 GPS 측정값이 필요합니다.25 이 닭과 달걀 문제는 `datum`에 의해 해결됩니다. `datum`(파라미터나 서비스 호출로 설정)은 로봇의 `map` 프레임 원점(0,0,0)에 해당하는 초기 지리 좌표를 제공합니다.26 이를 통해 `navsat_transform_node`는 EKF의 출력 없이도 초기 `utm` -> `map` 변환을 계산할 수 있게 되어, 순환 루프를 끊고 시스템이 정상적으로 초기화될 수 있도록 합니다. 이 메커니즘을 이해하는 것은 초기화 실패를 디버깅하는 데 매우 중요합니다.

또한, 이중 EKF 설정은 두 필터 간의 진정한 의미의 퓨전이 아닙니다. 두 필터는 서로 독립적인 추정기이며, TF 트리를 통해 협력적으로 작동합니다. 한 사용자는 두 EKF 간에 "퓨전"이 어떻게 일어나는지에 대해 혼란을 표현했는데, 답변은 "두 EKF는 어떠한 상호작용도 하지 않는다"고 명확히 합니다.10 이는 매우 중요한 사실입니다. 지역 EKF의 유일한 임무는 `odom`->`base_link` 변환을 생성하는 것이고, 전역 EKF의 유일한 임무는 `map`->`odom` 변환을 생성하는 것입니다. 최종적으로 전역적으로 정확한 자세(`map`->`base_link`)는 단일 노드의 출력이 아니라, TF 시스템이 이 두 변환을 연결한 결과물입니다. 이는 디버깅 노력을 분리해야 함을 의미합니다. 지역적인 움직임이 불안정하면 지역 EKF를, 전역 위치가 틀리면 전역 EKF와 `navsat_transform_node`를 디버깅해야 합니다.


이 표는 두 EKF 인스턴스의 역할을 명확하게 비교하여, 이중 EKF 아키텍처를 이해하고 올바르게 구현하기 위한 핵심 참조 자료를 제공합니다.

| 구분                       | 지역 EKF (`ekf_odom`)                  | 전역 EKF (`ekf_map`)                   |
| -------------------------- | -------------------------------------- | -------------------------------------- |
| **주요 역할**              | 부드럽고 연속적인 지역 오도메트리 생성 | 드리프트 없는 전역 자세 추정 및 보정   |
| **`world_frame` 파라미터** | `odom`                                 | `map`                                  |
| **입력 센서**              | 연속적인 센서 (IMU, 휠 오도메트리 등)  | 모든 센서 (IMU, GPS, 바로미터 등)      |
| **발행 TF**                | `odom` -> `base_link`                  | `map` -> `odom`                        |
| **발행 오도메트리 토픽**   | `/odometry/filtered/local` (예시)      | `/odometry/filtered/global` (예시)     |
| **핵심 특징**              | 드리프트가 있지만 연속적이고 부드러움  | 불연속적일 수 있지만 전역적으로 정확함 |

------


이 섹션에서는 각 센서를 시스템에 통합하는 구체적인 방법을 제공합니다.


- **데이터 준비:** 센서 데이터가 REP-103 표준을 준수하는 것이 매우 중요합니다. 월드 기준 데이터는 ENU(X-동, Y-북, Z-상), 바디 기준 데이터는 (X-전방, Y-좌, Z-상) 규칙을 따라야 합니다.21
- **설정:** YAML 파일에서 자세(roll, pitch, yaw)와 각속도를 퓨전하도록 설정하는 예시를 보입니다. `imuN_remove_gravitational_acceleration` 파라미터는 `true`로 설정할 경우, 중력 가속도를 제거하기 위해 정확한 절대 자세 정보가 반드시 필요하다는 점을 유의해야 합니다.13
- **모범 사례:** 자세 정보가 신뢰할 수 있다면 각속도보다 자세를 퓨전하고, 그 반대의 경우도 마찬가지입니다.21 선형 가속도를 퓨전할 경우, 이중 적분으로 인한 빠른 드리프트가 발생하므로 GPS나 바로미터와 같은 다른 센서로 오차를 구속하지 않으면 상태 추정치가 발산할 수 있음을 경고해야 합니다.30


- **역할과 기능:** 이 노드의 유일한 목적은 `sensor_msgs/NavSatFix` 메시지(위도, 경도, 고도)를 로봇의 `map` 또는 `odom` 프레임 기준의 `nav_msgs/Odometry` 메시지로 변환하는 것입니다.2 이 과정은 먼저 WGS84 좌표를 지역 데카르트 좌표계인 UTM(Universal Transverse Mercator) 그리드로 변환하는 것으로 시작됩니다.23
- **입력:** 세 가지 필수 입력이 있습니다: GPS의 `/gps/fix`, IMU의 `/imu/data`(헤딩용), 그리고 전역 EKF 출력인 `/odometry/filtered`(초기 자세용).27
- **중요 파라미터:**
  - `magnetic_declination_radians`: IMU의 자력계가 가리키는 자북(magnetic north)과 UTM 그리드가 사용하는 진북(true north) 사이의 차이(자기 편각)를 보정하는 역할을 합니다.26
  - `yaw_offset`: IMU의 0도 헤딩 방향을 REP-103 표준인 동쪽(East)으로 정렬하는 역할을 합니다. 예를 들어, IMU가 북쪽을 향할 때 0도를 출력한다면, `yaw_offset`은 반드시 $ \pi/2 $로 설정해야 합니다.26
  - `zero_altitude`: `true`로 설정하면 출력되는 오도메트리의 Z 위치를 0으로 강제합니다. 이는 EKF에서 Z 위치를 퓨전하는 설정과 함께 사용되어야 합니다.26
  - `wait_for_datum` 및 `datum` 서비스: `utm` -> `map` 변환을 초기화하는 데 사용되는 이 파라미터들의 역할을 다시 한번 강조합니다.26
- **출력:** 이 노드는 `/odometry/gps` 토픽을 발행하며, 이 데이터는 전역 EKF 인스턴스에 `pose` 입력으로 제공됩니다.27

`navsat_transform_node`는 단순한 변환기가 아니라, 중요한 정적 변환인 `utm` -> `map`을 설정하는 작은 상태 추정기 자체로 이해해야 합니다. 이 노드의 주된 출력은 `/odometry/gps`라는 자세 스트림이지만, 가장 중요한 일회성 계산은 전역 UTM 그리드와 로봇의 시작 `map` 프레임 간의 고정된 관계를 결정하는 것입니다.10 이 계산은 첫 유효 GPS  `datum`, IMU의 헤딩, 그리고 EKF 출력의 자세를 사용합니다. 이 변환이 한번 설정되면, 이후의 모든 GPS 포인트는 단순히 이 변환을 통과하게 됩니다. 따라서, 잘못된 `yaw_offset`이나 `magnetic_declination`으로 인한 초기 헤딩 오차나 초기 자세 오차는 미래의 모든 GPS 기반 위치에 영구적인 회전 또는 평행 이동 오프셋을 유발합니다. 이 초기 변환을 디버깅하는 것이 무엇보다 중요합니다.


- **도전 과제:** 바로미터는 고도를 제공하지만, `robot_localization`은 전용 `Barometer` 메시지 타입을 지원하지 않습니다. 따라서 데이터를 지원되는 형식으로 변환하여 발행해야 합니다.

- **해결책:** 바로미터의 고도 측정값을 `geometry_msgs/PoseWithCovarianceStamped` 메시지의 Z 위치 컴포넌트로 발행합니다.21

- **발행기 노드:** 원시 바로미터 데이터(예: `sensor_msgs/FluidPressure`)를 구독하고, 압력을 고도로 변환한 후, `PoseWithCovarianceStamped` 메시지를 발행하는 파이썬 또는 C++ 노드 예제를 구성할 수 있습니다. 메시지의 `header.frame_id`는 `map` 또는 `odom`이어야 하며, Z 위치를 제외한 다른 모든 자세 필드는 0으로 채워야 합니다.

- **YAML 설정:** 전역 EKF를 위한 정확한 YAML 설정은 다음과 같습니다. 이 예시는 사용자의 핵심 질문에 직접적으로 답하며, 관련 자료들의 논리에 의해 뒷받침됩니다.6

  ```YAML
  # ekf_global.yaml
  pose0: /your_barometer/pose_topic
  pose0_config: [false, false, true,   # Z 위치만 퓨전
                 false, false, false,
                 false, false, false,
                 false, false, false,
                 false, false, false]
  pose0_differential: false
  pose0_queue_size: 5
  ```

- **공분산:** 발행되는 메시지의 Z 위치에 대한 공분산은 바로미터의 데이터시트에 명시된 정확도를 반영해야 합니다.

IMU의 Z축 선형 가속도를 퓨전할 때 발생하는 Z축 드리프트를 막을 수 있는 유일한 실용적인 방법은 바로미터 데이터를 퓨전하는 것입니다. IMU는 중력을 포함한 가속도를 측정하며, `imuN_remove_gravitational_acceleration: true`로 설정하더라도 Z축 가속도계의 잔여 바이어스는 EKF에 의해 이중 적분되어 시간에 따라 제곱에 비례하는(오차=0.5×바이어스×t2) 위치 오차를 유발합니다.30 이는 Z축 드리프트가 무한정 커지는 결과로 이어집니다. 바로미터는 절대적인(비록 노이즈가 있지만) 고도(Z-위치) 측정값을 제공합니다. 이 절대 Z-위치 측정값을 (높은 공분산으로라도) 퓨전함으로써, IMU의 적분된 가속도 바이어스로 인해 필터가 발산하는 것을 방지하는 "구속" 또는 "앵커" 역할을 합니다. 즉, 바로미터는 IMU를 길들이는 역할을 합니다.

------


이 섹션은 구현 과정에서 가장 어렵고 시간이 많이 소요되는 튜닝 문제를 다룹니다.


공분산 행렬은 임의의 숫자가 아니라, 필터가 특정 측정값을 얼마나 "신뢰"하는지를 나타내는 통계적 표현입니다. 낮은 분산(variance)은 높은 신뢰를, 높은 분산은 낮은 신뢰를 의미합니다.21 이 행렬은 반드시 올바르게 채워져야 하며, 특정 변수를 "무시"하기 위해 매우 큰 값을 사용하는 대신, `_config` 행렬을 `false`로 설정하는 것이 올바른 방법입니다.21


- **데이터시트에서 시작:** 센서 제조사가 제공하는 노이즈/정확도 사양을 공분산 행렬의 대각 성분(분산)의 초기값으로 사용합니다.
- **경험적 튜닝:** 데이터시트는 이상적인 조건에서의 성능을 나타냅니다. 실제 환경에서는 진동, 온도, 전원 노이즈 등이 성능에 영향을 미칩니다.34 튜닝 과정은 필터의 동작을 관찰하고 값을 조정하는 반복적인 작업입니다.
- **A/B 테스트:** 실용적인 방법 중 하나는 한 번에 하나의 센서만 활성화하여(예: IMU만) 로봇을 구동하고 출력을 그래프로 그려보는 것입니다. 출력의 노이즈와 떨림을 관찰하면 해당 센서의 실제 환경에서의 분산을 질적으로 파악할 수 있습니다. 다른 센서에 대해서도 이 과정을 반복합니다.
- **"진동" 테스트:** 만약 특정 상태(예: yaw)의 필터 출력이 두 값 사이에서 심하게 진동한다면, 이는 해당 상태를 제공하는 두 센서의 공분산이 모두 너무 낮게 설정되어(필터가 두 센서를 모두 과도하게 신뢰함) 측정값이 서로 충돌하고 있음을 의미합니다.13 덜 신뢰하는 센서의 공분산을 진동이 멈출 때까지 점진적으로 증가시킵니다.


- **역할:** 이 행렬은 필터의 *예측 모델* 자체의 불확실성을 모델링합니다. 이는 휠 슬립, 돌풍 등 모델링되지 않은 동역학적 요인으로 인해 측정 간격 사이에 상태가 얼마나 변할 수 있는지를 나타냅니다.22
- **튜닝 원리:** `Q` 값이 클수록 필터는 자신의 예측을 덜 신뢰하고 들어오는 센서 측정값에 기반하여 더 공격적으로 상태를 보정합니다. `Q` 값이 작을수록 필터는 자신의 예측을 더 신뢰하고 노이즈가 섞인 측정값을 평활화(smoothing)하는 "뻣뻣한" 특성을 보입니다.22
- **실용적 접근법:** 느리게 변할 것으로 예상되는 변수(예: 지상 로봇의 Z축 속도)에 대해서는 대각 성분을 작은 값으로 시작하고, 빠르게 변할 것으로 예상되는 변수(예: X축 속도, Yaw 각속도)에 대해서는 더 큰 값으로 시작합니다. 필터의 반응성을 관찰하며 튜닝합니다. 만약 추정치가 실제 움직임보다 뒤처진다면(lag), 관련된 `Q` 값을 증가시킵니다. 만약 추정치가 너무 노이즈가 많고 불안정하다면, `Q` 값을 감소시킵니다.

프로세스 노이즈(`Q`)와 측정 노이즈(`R`)의 절대적인 값보다 그들의 *비율*이 더 중요합니다. 이 비율은 칼만 게인(Kalman Gain)을 결정하며, 이는 예측과 보정 사이의 균형을 조절합니다. 칼만 필터 업데이트 단계의 핵심은 `새로운_추정치 = 이전_추정치 + K * (측정값 - 예측된_측정값)` 입니다. 칼만 게인 `K`는 대략적으로 $Q / (Q + R)$에 비례합니다. 만약 `R`에 비해 `Q`가 크면, `K`는 1에 가까워지고 새로운 추정치는 새로운 측정값을 강력하게 반영합니다. 만약 `Q`에 비해 `R`이 크면, `K`는 0에 가까워지고 새로운 추정치는 예측을 더 신뢰하게 됩니다. 따라서 튜닝은 `Q`와 `R`을 독립적으로 완벽하게 찾는 것이 아니라, 모션 모델과 센서에 대한 상대적인 신뢰도를 반영하는 올바른 *균형*을 찾는 과정입니다.38

잘못 설정된 공분산 값 중에서는 너무 낮은 값이 너무 높은 값보다 훨씬 더 위험합니다. 공분산을 너무 높게 설정하면(낮은 신뢰도) 필터가 좋은 센서를 충분히 활용하지 못해 성능이 저하되고 드리프트가 발생할 수 있지만, 필터 자체는 안정성을 유지합니다.34 반면, 노이즈가 많거나 편향된 센서에 대해 공분산을 너무 낮게 설정하면(과신), 필터는 잘못된 데이터에 집착하게 되어 전체 상태 추정치를 오염시키고 잠재적으로 완전히 발산하게 만들 수 있습니다.34 이는 필터 내부 상태의 공분산 값이 "폭발"하고 측위 시스템이 완전히 실패하는 결과로 이어질 수 있습니다.22 튜닝의 황금률은 "확신이 없다면 보수적으로 접근하여 공분산을 높여라"입니다.

------


이 섹션에서는 시스템을 구축하고 테스트하기 위한 구체적인 코드와 도구를 제공합니다.


앞서 논의된 원칙에 따라, 완전하고 주석이 달린 예제 파일들을 제공합니다. 사용자는 공개된 GitHub 저장소를 시작점으로 활용할 수 있습니다.40

- **`dual_ekf.launch.py`:** 두 개의 `ekf_localization_node` 인스턴스(지역 및 전역)와 `navsat_transform_node`를 시작하는 파이썬 런치 파일입니다. 이 파일은 각 노드에 해당하는 YAML 파일을 로드하고 필요한 토픽 리매핑(remapping)을 처리합니다.24
- **`ekf_local.yaml`:** `odom` 프레임 EKF를 위한 설정 파일입니다.
- **`ekf_global.yaml`:** `map` 프레임 EKF를 위한 설정 파일입니다.
- **`navsat.yaml`:** `navsat_transform_node`를 위한 설정 파일입니다.


측위 시스템을 위한 RViz2 대시보드를 설정하는 단계별 가이드는 다음과 같습니다.

- **Fixed Frame 설정:** Global Options에서 `Fixed Frame`을 설정하는 것은 매우 중요합니다. `map`으로 설정하면 전역적인 맥락에서 로봇의 움직임을 볼 수 있습니다. `base_link`로 설정하면 로봇을 중심으로 주변 세계가 움직이는 것을 볼 수 있습니다.43
- **필수 디스플레이 추가:**
  - **TF:** 모든 좌표계(`map`, `odom`, `base_link`, 센서 프레임)를 시각화합니다. 이는 좌표계 관련 문제를 디버깅하는 데 가장 중요한 디스플레이입니다.43
  - **Odometry (`nav_msgs/Odometry`):** 원시 오도메트리, 지역 EKF 출력(`/odometry/filtered/local`), 전역 EKF 출력(`/odometry/filtered/global`)에 대한 디스플레이를 추가합니다. 서로 다른 색상을 사용하여 비교하면 드리프트와 보정 과정을 시각적으로 평가할 수 있습니다.11
  - **Pose (`geometry_msgs/PoseWithCovarianceStamped`):** 바로미터의 자세 토픽에 대한 디스플레이를 추가하여 고도 측정값을 확인합니다.
- **`map`과 `odom` 프레임 드리프트 해석:** 로봇이 움직임에 따라, `odom` 프레임은 `map` 프레임으로부터 서서히 멀어지며 드리프트되는 것이 보여야 합니다. GPS 보정 신호가 도착하면, `map` -> `odom` 변환은 순간적으로 점프하지만 `odom` -> `base_link` 변환은 부드럽게 유지됩니다. 이러한 시각적 확인은 이중 EKF 시스템이 의도한 대로 작동하고 있다는 증거입니다.11

RViz를 실행하기 전에, `rqt_graph`와 `tf2_tools view_frames`를 이용한 "사전 점검"은 필수적입니다. `rqt_graph`는 노드와 토픽 연결을 보여주어, 의도한 대로 구독/발행이 이루어지는지 즉시 확인할 수 있게 해줍니다.44

`ros2 run tf2_tools view_frames`는 TF 트리의 PDF를 생성하여, 연결이 끊어진 프레임이나 잘못된 부모-자식 관계를 명확하게 보여줍니다.44 TF 트리가 깨져 있다면 RViz의 어떤 시각화도 제대로 작동하지 않으므로, 이 두 명령어를 먼저 실행하는 것은 수많은 디버깅 시간을 절약해 줄 수 있습니다.


- **공분산 폭발 (Covariance Explosion):** 필터 출력이 멈추거나 `NaN`이 됩니다. **원인:** 노이즈가 심한 센서에 대한 과신(너무 낮은 공분산), 또는 지연되거나 잘못된 데이터의 고주파 업데이트.22

  **해결책:** 공분산을 높이고, 센서 타임스탬프를 확인하며, `process_noise_covariance`를 튜닝합니다.

- **"Could not find transform" 오류:** **원인:** `base_link`와 센서의 `frame_id` 사이에 정적(static) 변환이 없거나, 시간 동기화 문제(`use_sim_time`).46

  **해결책:** 모든 센서가 TF 트리에서 `base_link`로의 경로를 가지도록 하고, `use_sim_time` 파라미터를 올바르게 설정합니다.

- **전역 자세가 회전/오프셋됨:** **원인:** `navsat_transform_node`의 `yaw_offset` 또는 `magnetic_declination` 값이 부정확함. **해결책:** 로봇의 특정 위치와 IMU에 맞게 이 값들을 신중하게 보정합니다.

- **필터가 실제 움직임을 뒤따라감 (Lag):** **원인:** `process_noise_covariance`가 너무 낮음. 필터가 자신의 (오래된) 예측을 너무 신뢰함. **해결책:** 관련된 속도/가속도 상태에 대한 `Q` 값을 증가시킵니다.

초보자들이 겪는 가장 흔한 실패 모드는 잘못 설정된 TF 트리입니다. 필터 노드는 퓨전 전에 모든 들어오는 센서 데이터를 `base_link` 프레임으로 변환해야 합니다.21 예를 들어, IMU 메시지가 

`frame_id: "imu_link"`로 발행된다면, 로봇에 IMU가 물리적으로 장착된 위치와 방향을 설명하는 `base_link`에서 `imu_link`로의 정적 변환이 반드시 TF 시스템에 발행되어야 합니다. 이 변환이 없으면, 필터는 "Could not transform..." 오류를 쏟아내며 해당 센서의 데이터를 전혀 퓨전하지 않을 것입니다.46 이는 사용자들이 패키지가 센서의 위치를 "알아서 알 것"이라고 가정하면서 흔히 간과하는 부분입니다. 보고서는 URDF 파일이나 

`static_transform_publisher` 노드를 통해 이러한 정적 변환을 정의할 필요성을 강력히 강조해야 합니다.

------



본 보고서는 ROS2 환경에서 IMU, RTK-GPS, 바로미터를 사용하여 강건한 상태 추정 시스템을 구축하는 과정을 심도 있게 탐구했습니다. 핵심적인 발견 사항들을 요약하면 다음과 같습니다. 첫째, GPS와 같은 불연속적인 전역 센서와 IMU와 같은 연속적인 지역 센서를 효과적으로 통합하기 위해서는 이중 EKF 아키텍처가 필수적입니다. 둘째, ROS의 표준 좌표계(REP-105)에 대한 명확한 이해와 이에 따른 데이터 준비는 시스템 성공의 전제 조건입니다. 셋째, 공분산 튜닝은 단순히 숫자를 맞추는 작업이 아니라, 센서와 모션 모델에 대한 신뢰도를 통계적으로 표현하는 과학과 경험의 결합입니다.


가장 중요한 설계 패턴은 `map` 프레임과 `odom` 프레임의 역할을 명확히 분리하는 것입니다. `map` 프레임은 전역적이고 절대적인 정확성을 담당하며, `odom` 프레임은 지역적이고 연속적인 부드러움을 담당합니다. 이 두 세계를 `map` -> `odom` 변환으로 연결함으로써, 시스템은 장기적인 드리프트 보정과 단기적인 부드러운 움직임 제어라는 두 마리 토끼를 모두 잡을 수 있습니다. 이 아키텍처는 고성능 자율 항법 시스템을 구축하기 위한 검증된 청사진입니다.


`robot_localization` 패키지는 현재 ROS 생태계에서 가장 강력하고 널리 사용되는 필터링 기반 솔루션이지만, 로봇 공학 커뮤니티의 관심은 점차 팩터 그래프(factor graph) 기반의 스무딩(smoothing) 기법으로 이동하고 있습니다.15 GTSAM, Fuse와 같은 라이브러리는 전체 궤적을 최적화함으로써 더 높은 정확도를 달성하고, 루프 클로저나 순서가 뒤바뀐 데이터 처리에 있어 개념적으로 더 우월한 유연성을 제공합니다. 본 보고서에서 다룬 지식은 견고한 필터링 시스템을 구축하는 데 필수적이며, 동시에 사용자가 향후 더 높은 정확도를 추구하며 스무딩 기반의 차세대 상태 추정 기술로 나아갈 수 있는 튼튼한 발판이 될 것입니다.


1. ROS 2 Documentation: Humble documentation, accessed July 30, 2025, https://docs.ros.org/en/humble/index.html
2. robot_localization wiki - ROS documentation, accessed July 30, 2025, http://docs.ros.org/en/melodic/api/robot_localization/html/index.html
3. ROS Package: robot_localization, accessed July 30, 2025, https://index.ros.org/p/robot_localization/
4. robot_localization/README.md at ros2 - GitHub, accessed July 30, 2025, https://github.com/cra-ros-pkg/robot_localization/blob/ros2/README.md
5. ROS2 Navigation: Setting up Robot, accessed July 30, 2025, https://robotics.snowcron.com/robotics_ros2/nav_basics_localization.htm
6. Multiple Odometry data for robot_localization package in ros2 - Robotics Stack Exchange, accessed July 30, 2025, https://robotics.stackexchange.com/questions/105095/multiple-odometry-data-for-robot-localization-package-in-ros2
7. Wrong Initial Orientation / Issue #141 / CCNYRoboticsLab/imu_tools - GitHub, accessed July 30, 2025, https://github.com/ccny-ros-pkg/imu_tools/issues/141
8. How to use the ROS robot_localization package | by Zillur - Medium, accessed July 30, 2025, https://medium.com/@zillur-rahman/how-to-use-the-ros-robot-localization-package-534fe04014d3
9. State Estimation Nodes - robot_localization 2.7.7 documentation, accessed July 30, 2025, http://docs.ros.org/en/noetic/api/robot_localization/html/state_estimation_nodes.html
10. robot_localization dual-EKF:How the two ekf nodes work together, accessed July 30, 2025, https://robotics.stackexchange.com/questions/90869/robot-localization-dual-ekfhow-the-two-ekf-nodes-work-together
11. Robot localization: Odom tf is far offset from Map tf - Robotics Stack Exchange, accessed July 30, 2025, https://robotics.stackexchange.com/questions/103423/robot-localization-odom-tf-is-far-offset-from-map-tf
12. Map->base->odom as alternative for REP-105 recommended frame order - ROS Discourse, accessed July 30, 2025, https://discourse.openrobotics.org/t/map-base-odom-as-alternative-for-rep-105-recommended-frame-order/25095
13. Configuring robot_localization - robot_localization 2.6.12 ..., accessed July 30, 2025, http://docs.ros.org/en/melodic/api/robot_localization/html/configuring_robot_localization.html
14. Factor Graphs and GTSAM | GTSAM, accessed July 30, 2025, https://gtsam.org/tutorials/intro.html
15. Future of ROS 2 GPS Support - ROS General - ROS Discourse - Open Robotics, accessed July 30, 2025, https://discourse.openrobotics.org/t/future-of-ros-2-gps-support/33297
16. The Unreasonable Power of The Unscented Kalman Filter with ROS 2 | by Carlos Argueta, accessed July 30, 2025, https://soulhackerslabs.com/the-unreasonable-power-of-the-unscented-kalman-filter-with-ros-2-d4c97d4b4bb9
17. State Estimation Nodes - robot_localization 2.6.12 documentation, accessed July 30, 2025, http://docs.ros.org/en/melodic/api/robot_localization/html/state_estimation_nodes.html
18. State Estimation Nodes - robot_localization 2.3.1 documentation, accessed July 30, 2025, http://docs.ros.org/jade/api/robot_localization/html/state_estimation_nodes.html
19. Working with the robot_localization Package, accessed July 30, 2025, https://roscon.ros.org/2015/presentations/robot_localization.pdf
20. Sensor Fusion Using the Robot Localization Package – ROS 2 - Automatic Addison, accessed July 30, 2025, https://automaticaddison.com/sensor-fusion-using-the-robot-localization-package-ros-2/
21. Preparing Your Data for Use with robot_localization - ROS documentation, accessed July 30, 2025, http://docs.ros.org/jade/api/robot_localization/html/preparing_sensor_data.html
22. ros2 - robot_localization EKF covariance exploding with absolute ..., accessed July 30, 2025, https://robotics.stackexchange.com/questions/110754/robot-localization-ekf-covariance-exploding-with-absolute-pose-input
23. Integrating GPS Data - robot_localization - ROS documentation, accessed July 30, 2025, http://docs.ros.org/en/noetic/api/robot_localization/html/integrating_gps.html
24. Smoothing Odometry using Robot Localization - Nav2 1.0.0 documentation, accessed July 30, 2025, https://docs.nav2.org/setup_guides/odom/setup_robot_localization.html
25. navsat_transform documentation diagram / Issue #550 / cra-ros-pkg/robot_localization, accessed July 30, 2025, https://github.com/cra-ros-pkg/robot_localization/issues/550
26. Integrating GPS Data - robot_localization - ROS documentation, accessed July 30, 2025, https://docs.ros.org/en/indigo/api/robot_localization/html/integrating_gps.html
27. navsat_transform_node - robot_localization 2.6.12 documentation, accessed July 30, 2025, http://docs.ros.org/en/melodic/api/robot_localization/html/navsat_transform_node.html
28. IMU convention for robot_localization - Robotics Stack Exchange, accessed July 30, 2025, https://robotics.stackexchange.com/questions/82375/imu-convention-for-robot-localization
29. State Estimation Nodes - robot_localization 2.5.6 documentation, accessed July 30, 2025, http://docs.ros.org/lunar/api/robot_localization/html/state_estimation_nodes.html
30. navigation - robot_localization: map -> odom drifts in z-direction with ..., accessed July 30, 2025, https://robotics.stackexchange.com/questions/77777/robot-localization-map-odom-drifts-in-z-direction-with-zero-altitude-set-to
31. Declination usage in navsat_transform - Robotics Stack Exchange, accessed July 30, 2025, https://robotics.stackexchange.com/questions/102824/declination-usage-in-navsat-transform
32. Random Heading Robot Localization - ROS Answers archive, accessed July 30, 2025, https://answers.ros.org/question/375741
33. how do i add barometer and gps data to robot localization? - ROS Answers archive, accessed July 30, 2025, https://answers.ros.org/question/362799/
34. AI-Driven Dynamic Covariance for ROS 2 Mobile Robot Localization ..., accessed July 30, 2025, https://pmc.ncbi.nlm.nih.gov/articles/PMC12114818/
35. Setting covarince matrix for Robot Localization - ROS Answers archive, accessed July 30, 2025, https://answers.ros.org/question/351488/
36. how to use process_noise_covariance - Robotics Stack Exchange, accessed July 30, 2025, https://robotics.stackexchange.com/questions/81578/how-to-use-process-noise-covariance
37. Setup Process Noise Covariance of robot_localization with enabled ~use_control parameter, accessed July 30, 2025, https://answers.ros.org/question/405136/
38. "process_noise_covariance" and "initial_estimate_covariance" in EKF-global and EKF-local - Robotics Stack Exchange, accessed July 30, 2025, https://robotics.stackexchange.com/questions/100642/process-noise-covariance-and-initial-estimate-covariance-in-ekf-global-and-e
39. Ros2 Robot Localization package problems at high frequencies / Issue #798 / cra-ros-pkg/robot_localization - GitHub, accessed July 30, 2025, https://github.com/cra-ros-pkg/robot_localization/issues/798
40. cocodmdr/turtlebot3_robot_localization_ws: tutorial with ... - GitHub, accessed July 30, 2025, https://github.com/cocodmdr/turtlebot3_robot_localization_ws
41. gyiptgyipt/loc_diff: robot_localization , GPS , Mapviz and Nav2 gps ( ros2 humble) - GitHub, accessed July 30, 2025, https://github.com/gyiptgyipt/loc_diff
42. robot_localization/launch/dual_ekf_navsat_example.launch.py at ..., accessed July 30, 2025, https://github.com/cra-ros-pkg/robot_localization/blob/ros2/launch/dual_ekf_navsat_example.launch.py
43. Visualizing with RViz2 | Clearpath Robotics Documentation, accessed July 30, 2025, https://docs.clearpathrobotics.com/docs/ros/tutorials/rviz/
44. Sensor Fusion and Robot Localization Using ROS 2 Jazzy - YouTube, accessed July 30, 2025, https://www.youtube.com/watch?v=XOQTF38lmtE
45. Sensor Fusion and Localization in ROS2: Understanding and Implementing the Extended Kalman Filter (EKF) | by SHANMUGARAJA PANDIAN | Jul, 2025 | Medium, accessed July 30, 2025, https://medium.com/@shanmugaraja9919/sensor-fusion-and-localization-in-ros2-understanding-and-implementing-the-extended-kalman-filter-e4c861ddc1f2
46. How to set ekf_node in robot_localization? - Robotics Stack Exchange, accessed July 30, 2025, https://robotics.stackexchange.com/questions/103921/how-to-set-ekf-node-in-robot-localization
47. Sensor Fusion and Robot Localization Using ROS 2 Jazzy, accessed July 30, 2025, https://automaticaddison.com/sensor-fusion-and-robot-localization-using-ros-2-jazzy/


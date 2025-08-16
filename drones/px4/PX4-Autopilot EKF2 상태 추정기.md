# PX4-Autopilot EKF2 상태 추정기

## 1.  EKF2 아키텍처 개요

PX4-Autopilot의 핵심은 기체의 상태, 즉 자세(각도), 속도, 위치를 정확하게 추정하는 능력에 있다. 이 중추적인 역할을 수행하는 것이 바로 EKF2(Extended Kalman Filter 2) 모듈이다. EKF2는 단순히 센서 데이터를 필터링하는 것을 넘어, 실제 비행 환경에서 마주치는 다양한 비이상적인 조건들을 극복하도록 설계된 정교한 상태 추정 시스템이다.

### 1.1  확장 칼만 필터(EKF)의 역할과 PX4에서의 중요성

EKF2는 PX4 펌웨어의 기본 상태 추정기로, 특히 GNSS/GPS와 같은 절대 위치 센서가 장착된 기체에 사용이 강력히 권장된다.1 그 이유는 EKF2가 관성 측정 장치(IMU)의 상대적인 측정값에서 발생하는 누적 오차(drift)를 GPS의 절대적인 위치 정보로 주기적으로 보정하여 장시간 비행에서도 안정적인 항법 성능을 보장하도록 설계되었기 때문이다.

EKF2는 3축 자세(attitude), 3차원 위치 및 속도(3D position/velocity), 그리고 비행에 영향을 미치는 바람의 상태(wind states)까지 포괄적으로 추정한다.1 이는 단순히 기체의 현재 위치를 아는 것을 넘어, 바람의 영향을 예측하고 제어기가 이를 보상하는 고급 비행 제어를 가능하게 하는 필수 정보를 생성함을 의미한다. 과거에는 LPE(Local Position Estimator)와 같은 다른 추정기도 사용되었으나 현재는 폐기(deprecated)되었으며, GPS가 없는 특수한 경우(예: 실내 레이싱 드론)를 위해 Q-Estimator가 존재하지만, 표준적인 활용 사례에서는 EKF2가 독보적인 위치를 차지하고 있다.1 이는 PX4 개발 커뮤니티가 EKF2의 성능과 안정성 향상에 리소스를 집중하고 있음을 보여준다.

### 1.2  지연 퓨전 시간축 (Delayed Fusion Horizon): 센서 지연 문제 해결법

EKF2 아키텍처의 가장 중요한 설계 특징 중 하나는 '지연된 퓨전 시간축(delayed fusion time horizon)' 개념의 도입이다.2 실제 드론에 장착된 여러 센서(IMU, GPS, 기압계, 지자기 센서 등)는 각각 고유의 데이터 처리 및 전송 지연(latency)을 가진다. 만약 이 지연을 무시하고 데이터가 도착하는 즉시 퓨전(fusion)을 시도한다면, 시간적으로 서로 맞지 않는 정보가 결합되어 특히 급격한 기동 중에 추정 오차를 유발하고 심한 경우 필터가 발산하게 된다.

이 문제를 해결하기 위해, EKF2는 모든 센서 데이터를 수신 즉시 처리하지 않는다. 대신, 각 데이터는 고유의 타임스탬프와 함께 FIFO(First-In, First-Out) 버퍼에 저장된다.3 필터의 실제 계산은 현재 시간이 아닌, 시스템에서 가장 긴 센서 지연 시간을 고려한 과거의 특정 시점, 즉 '퓨전 시간축'에서 수행된다. EKF는 이 퓨전 시간축에 맞는 시점의 센서 데이터들을 각 버퍼에서 꺼내어 사용함으로써 모든 입력 데이터의 시간적 정합성을 보장한다.4 각 센서의 지연 시간은 `EKF2_*_DELAY` 파라미터를 통해 설정되며, 전체적인 퓨전 시간축의 지연 길이는 `EKF2_DELAY_MAX` 파라미터에 의해 결정된다.3

퓨전 시간축에서 계산된 과거의 상태는 현재 시간까지 보상 필터(complementary filter)를 통해 전파(propagate)된다. 이 과정에서는 버퍼에 저장된 고주파의 IMU 데이터를 사용하여 상태를 빠르게 예측하며, 이 보상 필터의 특성은 `EKF2_TAU_VEL` 및 `EKF2_TAU_POS` 파라미터로 제어된다.2 이 아키텍처는 동적 기동 상황에서 발생하는 가속도와 GPS 위치 측정값 간의 시간 불일치로 인한 혁신(innovation) 급증 문제를 근본적으로 해결하여 동적 안정성을 크게 향상시킨다.4

### 1.3  예측-수정 사이클: IMU 예측과 보조 센서 수정의 분리

EKF2는 칼만 필터의 고전적인 예측-수정(predict-correct) 구조를 명확하게 따른다. 여기서 중요한 점은 IMU 데이터가 **오직 상태 예측(state prediction) 단계에서만 사용된다**는 것이다.3 즉, IMU의 각속도와 가속도 측정값은 필터의 '관측(observation)'으로 사용되지 않고, 이전 상태로부터 다음 상태를 추정하는 운동학 모델의 입력으로만 활용된다.

이러한 역할 분리는 시스템의 견고성을 높이는 데 기여한다. 고주파(수백 Hz)로 들어오는 IMU 데이터를 사용해 상태를 빠르게 전파(예측)하고, 상대적으로 저주파(수 Hz ~ 수십 Hz)로 들어오는 GPS, 기압계, 지자기 센서와 같은 보조 센서들의 측정값을 사용하여 예측 과정에서 누적된 오차를 수정한다. 이 구조 덕분에 만약 GPS 신호가 일시적으로 끊기거나 다른 보조 센서 데이터가 유실되더라도, 기체는 IMU 기반의 예측을 통해 단기적으로 항법을 지속할 수 있다. 물론 시간이 지남에 따라 오차는 누적되겠지만, 시스템이 즉시 붕괴되는 것을 막아준다.

### 1.4  다중 인스턴스 및 페일오버 (Multi-Instance and Failover)

높은 신뢰성이 요구되는 임무를 위해, PX4는 여러 개의 EKF 인스턴스를 동시에 실행하는 기능을 제공한다. 생성되는 인스턴스의 총 수는 `EKF2_MULTI_IMU`와 `EKF2_MULTI_MAG` 파라미터에 의해 결정되며, 각각 다른 IMU와 지자기 센서 조합을 사용하도록 설정할 수 있다.3

이 기능은 센서 이중화(redundancy)를 통한 결함 허용(fault tolerance) 메커니즘의 핵심이다. 각 EKF 인스턴스는 독립적으로 자신에게 할당된 센서 데이터를 사용하여 기체의 상태를 추정한다. 시스템에는 'EKF 선택기(selector)'가 존재하여 각 인스턴스의 상태(건강도, 신뢰도)를 지속적으로 모니터링한다. 만약 특정 IMU에서 과도한 자이로 오차가 감지되는 등 하나의 인스턴스가 비정상적인 동작을 보이면, EKF 선택기는 해당 인스턴스의 추정치를 배제하고 가장 신뢰도가 높은 다른 인스턴스의 결과로 출력을 신속하게 전환(failover)한다.3 특히 3개 이상의 IMU를 사용하여 다중 인스턴스를 운영할 경우, 선택기는 중간값 선택(median select) 전략을 적용하여 고장난 IMU를 더 빠르게 식별하고 격리할 수 있다.3

이 기능을 올바르게 사용하기 위해서는 `SENS_IMU_MODE`와 `SENS_MAG_MODE` 파라미터를 1(기본값)에서 0으로 변경해야 한다. 기본 설정에서는 센서 모듈이 자체적으로 최상의 센서를 선택하여 단일 데이터 스트림을 제공하지만, 0으로 설정하면 각 EKF 인스턴스가 지정된 센서 데이터에 직접 접근하여 독립적인 추정을 수행할 수 있게 된다.3 이는 단순한 데이터 유실 방지를 넘어, 비정상적인 데이터를 생성하는 센서 자체의 고장에 대응하기 위한 필수적인 설정이다.

## 2.  핵심 코드 구조 분석

EKF2의 정교한 아키텍처는 소스 코드 구조에도 그대로 반영되어 있다. 코드는 크게 PX4 운영체제와 통합을 담당하는 '모듈 래퍼'와, 순수한 수학적 알고리즘을 담고 있는 '핵심 라이브러리'로 명확하게 분리되어 있다. 이러한 설계는 코드의 이식성, 테스트 용이성, 그리고 유지보수성을 극대화한다.

### 2.1  모듈 래퍼: `src/modules/ekf2/EKF2.cpp`

`src/modules/ekf2/EKF2.cpp` 파일은 EKF2 알고리즘을 PX4 시스템 내에서 하나의 독립적인 프로세스(태스크 또는 모듈)로 실행시키는 래퍼(wrapper) 역할을 한다.6 이 파일의 코드는 EKF 알고리즘 자체보다는 PX4의 생태계와 상호작용하는 데 초점을 맞춘다.

주요 책임은 다음과 같다:

- **uORB 메시지 구독 및 발행**: EKF 연산에 필요한 모든 센서 데이터를 uORB 메시징 시스템을 통해 구독한다. 여기에는 `sensor_combined`(IMU), `vehicle_gps_position`(GPS), `vehicle_magnetometer`(지자기), `vehicle_air_data`(기압계 및 풍속계) 등이 포함된다. EKF가 추정한 최종 상태는 `vehicle_odometry`(자세, 위치, 속도 포함), `estimator_status`(필터 상태), `wind_estimate`(바람 추정치) 등의 토픽으로 다시 발행되어 제어기(controller)나 다른 모듈에서 사용할 수 있도록 한다.
- **태스크 생성 및 관리**: `EKF2::task_spawn()` 함수를 통해 EKF의 메인 루프를 별도의 스레드(thread)로 생성하고 관리한다.6 이는 EKF의 복잡한 연산이 다른 중요한 비행 제어 루프를 방해하지 않도록 보장한다.
- **인스턴스 관리**: 다중 EKF 인스턴스 모드가 활성화된 경우, 여러 `Ekf` 클래스 객체를 생성하고 관리한다. 또한, `select_instance` 명령어를 통해 외부에서 특정 인스턴스로의 전환을 요청하는 인터페이스를 제공한다.6
- **데이터 및 파라미터 전달**: 구독한 uORB 메시지에서 데이터를 추출하여 핵심 라이브러리인 `Ekf` 클래스의 인스턴스로 전달하는 다리 역할을 수행한다. 또한, 사용자가 QGroundControl 등을 통해 설정한 `EKF2_*` 파라미터들을 읽어와 `Ekf` 인스턴스의 동작을 설정하는 역할을 한다.6

### 2.2  핵심 라이브러리: `src/lib/ecl/EKF/`

ECL(Estimation and Control Library)은 EKF2의 모든 핵심 수학 및 알고리즘 로직을 포함하는 독립 라이브러리이다.7 이 라이브러리는 PX4-Autopilot 메인 저장소와는 별개로 개발 및 테스트가 가능하며, 이론적으로는 PX4가 아닌 다른 로보틱스 프로젝트에서도 재사용할 수 있도록 설계되었다.7 이러한 분리는 순수 알고리즘 개발과 시스템 통합을 분리하여 개발 효율성을 높인다.

ECL 내의 `EKF` 디렉토리는 다음과 같은 구조를 가진다:

- **`ekf.h` / `ekf.cpp`**: `Ekf` 클래스의 선언과 핵심 구현을 담고 있다. 이 클래스가 EKF 알고리즘의 심장부로, 상태 벡터, 공분산 행렬을 멤버 변수로 가지며, 예측 및 수정 로직의 최상위 호출이 이 파일들에서 이루어진다.8

- **관심사 분리 (Separation of Concerns)**: `Ekf` 클래스의 기능은 매우 방대하지만, 코드의 가독성과 유지보수성을 높이기 위해 기능별로 여러 파일에 나뉘어 구현되어 있다.9

  - `control.cpp`: 어떤 센서 데이터를 언제 퓨전할지 결정하는 상위 레벨의 제어 로직을 담당한다.10 예를 들어, GPS 수신 상태가 특정 품질 기준을 만족하고, 충분한 수의 위성이 확보되었을 때만 GPS 퓨전을 활성화하는 결정 로직이 여기에 포함된다.

  - `covariance.cpp`: 공분산 행렬의 예측(`predictCovariance`) 및 수정과 관련된 복잡한 수학적 계산을 전담한다. 특히, 필터의 수치적 안정성을 보장하기 위한 공분산 행렬의 제약 조건(constraining) 로직이 포함되어 있다.11

  - **센서별 퓨전 파일**: 각 센서의 측정 모델과 퓨전 로직은 별도의 파일로 완벽하게 분리되어 있다. 예를 들어, `gps_fusion.cpp`, `mag_fusion.cpp` 12, `baro_fusion.cpp`, `range_fusion.cpp`, `flow_fusion.cpp` 등이 존재한다.13 새로운 센서를 추가하고자 할 때, 개발자는 다른 코드에 미치는 영향을 최소화하면서 새로운 퓨전 파일만 생성하여 통합할 수 있다.

이러한 모듈식 구조는 PX4/EKF2가 대규모 오픈소스 프로젝트로서 성공적으로 유지보수될 수 있는 기반을 제공한다. 알고리즘 전문가는 ECL 라이브러리에 집중하여 수학적 모델을 개선하고 단위 테스트를 수행할 수 있으며, 시스템 통합 전문가는 `modules/ekf2`에서 uORB 파이프라인과 태스크 관리에 집중할 수 있다.

## 3.  상태 벡터 및 운동학 모델

EKF2가 드론의 상태를 어떻게 표현하고 예측하는지 이해하기 위해서는 먼저 EKF의 '상태 벡터(state vector)'와 그 움직임을 기술하는 '운동학 모델(kinematic model)'을 알아야 한다. 이는 필터의 모든 연산의 수학적 기초가 된다.

### 3.1  EKF2 상태 벡터 정의

EKF2는 총 24개의 변수로 구성된 상태 벡터를 사용하여 드론의 운동 상태와 주요 센서의 오차를 동시에 추정한다. 각 상태 변수의 정의는 다음 표와 같다.

**Table 1: EKF2 24-State Vector Definition**

| 인덱스 (Index) | 변수 (Variable)                 | 차원 | 설명 (Description)                                           |
| -------------- | ------------------------------- | ---- | ------------------------------------------------------------ |
| 0-3            | Quaternion (‘q‘)                | 4    | NED(North-East-Down) 좌표계에서 기체(Body) 좌표계로의 회전을 나타내는 쿼터니언. [qw,qx,qy,qz] |
| 4-6            | Velocity (‘v‘)                  | 3    | NED 좌표계 기준의 속도 (North, East, Down) [m/s].            |
| 7-9            | Position (‘p‘)                  | 3    | NED 좌표계 기준의 위치 (North, East, Down) [m].              |
| 10-12          | Gyroscope Bias (‘bg‘)           | 3    | 자이로스코프 센서의 3축 바이어스 오차 [rad/s].               |
| 13-15          | Accelerometer Bias (‘ba‘)       | 3    | 가속도계 센서의 3축 바이어스 오차 [m/s²].                    |
| 16-18          | Earth Magnetic Field (‘me‘)     | 3    | NED 좌표계 기준의 지구 자기장 벡터.                          |
| 19-21          | Body Magnetic Field Bias (‘mb‘) | 3    | 기체에 고정된 자기장 소스(예: 모터, 전력선)에 의한 3축 자기장 바이어스. |
| 22-23          | Wind Velocity (‘vw‘)            | 2    | NED 좌표계 기준의 수평 바람 속도 (North, East) [m/s].        |

상태 벡터에 센서 바이어스(‘bg‘, ‘ba‘)와 환경 변수(‘me‘, ‘mb‘, ‘vw‘)가 포함되어 있다는 점은 매우 중요하다. 이는 EKF2가 단순히 기체의 운동만을 추정하는 필터가 아니라, 센서 자체의 결함과 주변 환경까지 능동적으로 모델링하고 보상하는 정교한 시스템임을 보여준다. 예를 들어, 자이로 바이어스를 추정하고 보상함으로써 시간이 지나도 자세가 크게 틀어지는 현상을 방지하고, 기체 자기장 바이어스를 추정함으로써 모터 전류 변화에 따른 자기장 간섭의 영향을 줄여 더 정확한 Yaw 각도 추정을 가능하게 한다.

### 3.2  기체 운동학: 상태 전파의 수학적 기초

EKF의 예측 단계는 IMU 측정값(각속도, 가속도)을 사용하여 현재 상태로부터 다음 시간 단계(‘Δt‘ 후)의 상태를 예측하는 과정이며, 이는 기체의 운동학 모델에 기반한다.

- 자세(쿼터니언) 전파:

  기체 좌표계에서 측정된 각속도 $\boldsymbol{\omega} = [\omega_x, \omega_y, \omega_z]^T$는 쿼터니언 미분 방정식 ‘q˙=21q⊗ωquat‘ (여기서 ‘ωquat=[0,ωx,ωy,ωz]T‘)을 통해 쿼터니언을 시간에 따라 변화시킨다. 이산 시간(discrete time)에서는 다음 시간의 쿼터니언 $\mathbf{q}_{k+1}$을 현재 쿼터니언 $\mathbf{q}_k$에 작은 회전 변화량 $\Delta \mathbf{q}$를 곱하여 계산할 수 있다.

  $$
  \mathbf{q}_{k+1} = \mathbf{q}_k \otimes \Delta \mathbf{q}(\boldsymbol{\omega}_k, \Delta t)
  $$
  여기서 $\otimes$는 쿼터니언 곱셈 연산을 의미한다.
  
- 속도 전파:

  기체 좌표계에서 측정된 가속도 $\mathbf{a}_{body}$에서 먼저 추정된 가속도계 바이어스 $\mathbf{b}_a$를 뺀 순수 가속도를 구한다. 이 가속도는 현재 추정된 자세를 나타내는 회전 행렬 $\mathbf{R}(\mathbf{q}_k)$를 통해 NED 좌표계로 변환된다. 마지막으로 중력 가속도 벡터 $\mathbf{g} = [0, 0, 9.80665]^T$의 영향을 더해준 후 시간에 대해 적분하여 속도를 업데이트한다.

  $$
  \mathbf{v}_{k+1} = \mathbf{v}_k + (\mathbf{R}(\mathbf{q}_k)(\mathbf{a}_{body} - \mathbf{b}_{a,k}) + \mathbf{g}) \Delta t
  $$
  
- 위치 전파:

  업데이트된 속도를 다시 시간에 대해 적분하여 위치를 전파한다. 단순화를 위해 1차 적분을 사용하거나, 더 정확한 계산을 위해 2차 적분을 사용할 수 있다.

  $$
  \mathbf{p}_{k+1} = \mathbf{p}_k + \mathbf{v}_k \Delta t + \frac{1}{2}(\mathbf{R}(\mathbf{q}_k)(\mathbf{a}_{body} - \mathbf{b}_{a,k}) + \mathbf{g}) \Delta t^2
  $$
  이러한 운동학 방정식들은 `Ekf::predictState()` 함수 내에서 실제 코드로 구현되어 EKF 예측 단계의 근간을 이룬다.

## 4.  예측 단계: IMU 데이터 기반 상태 전파

EKF의 예측 단계는 고주파로 수신되는 IMU 데이터를 사용하여 상태 벡터와 그 불확실성(공분산)을 다음 시간 단계로 전파하는 과정이다. 이 단계는 외부 센서의 도움 없이 오직 관성 정보만으로 단기적인 움직임을 예측하는 역할을 한다.

### 4.1  `Ekf::predictState()` 분석

`Ekf::predictState()` 함수는 새로운 IMU 샘플이 처리될 때마다 호출된다. 이 함수는 이전 IMU 샘플링 시점부터 현재까지 누적된 각도 변화량(`delta_ang`)과 속도 변화량(`delta_vel`)을 입력으로 받는다. 이 누적된 값을 사용하는 것은 IMU의 높은 샘플링 속도와 EKF의 상대적으로 낮은 업데이트 속도 사이의 불일치를 처리하고 적분 오차를 줄이는 효과적인 방법이다.

- **자세(쿼터니언) 예측**: 입력으로 받은 `delta_ang` 벡터(x, y, z축 회전각)를 축-각(axis-angle) 표현으로 간주하고, 이를 작은 회전을 나타내는 델타 쿼터니언(`dq`)으로 변환한다. 이 델타 쿼터니언을 현재의 공칭 쿼터니언(`_state.quat_nominal`)에 곱하여 자세를 업데이트한다. 마지막으로, 누적된 수치 오차를 방지하기 위해 쿼터니언을 정규화(normalize)한다. 이 과정은 3.2절에서 설명한 수학적 모델 $\mathbf{q}_{k+1} = \mathbf{q}_k \otimes \Delta \mathbf{q}$를 효율적으로 구현한 것이다.
- **속도 및 위치 예측**: `delta_vel`은 IMU의 기체 좌표계 기준 속도 변화량이므로, 이를 NED 좌표계로 변환해야 한다. 이 변환에는 현재 추정된 자세(`_state.quat_nominal`)로부터 계산된 회전 행렬(DCM)이 사용된다. 변환된 속도 변화량 `delta_vel_earth`를 현재 속도(`_state.vel`)에 더한다. 이와 동시에 중력의 영향(‘gravitymss∗dt‘)을 속도에 반영한다. 마지막으로, 업데이트된 속도를 사용하여 위치(`_state.pos`)를 적분한다. 이 로직은 14와 8에서 암시하는 바와 같이, IMU의 적분된 측정치를 사용하여 상태를 예측하는 EKF의 핵심적인 부분이다.
- **센서 바이어스 예측**: 가장 기본적인 모델에서 센서 바이어스는 시간에 따라 변하지 않는다고 가정(랜덤 상수)하거나, 매우 느리게 변하는 랜덤 워크(random walk) 프로세스로 모델링된다. 예측 단계에서는 일단 이전 상태의 바이어스 값을 그대로 유지한다(‘bk+1=bk‘). 바이어스의 실제 변화는 예측 모델의 불확실성을 나타내는 프로세스 노이즈(`Q` 행렬)에 의해 모델링되며, 실제 값의 업데이트는 수정 단계에서 이루어진다.

### 4.2  `Ekf::predictCovariance()` 분석

상태 벡터를 예측하는 것만으로는 충분하지 않다. 그 예측이 얼마나 불확실한지를 나타내는 공분산 행렬(`P`) 또한 함께 전파되어야 한다. `Ekf::predictCovariance()` 함수가 이 역할을 수행하며, EKF의 핵심 방정식 중 하나를 구현한다.

$$
\mathbf{P}_{k+1|k} = \mathbf{F}_k \mathbf{P}_{k|k} \mathbf{F}_k^T + \mathbf{Q}_k
$$

- $\mathbf{P}_{k|k}$는 이전 수정 단계에서 계산된 공분산 행렬이다.
- $\mathbf{F}_k$는 상태 전이 행렬(State Transition Matrix)의 야코비안(Jacobian)으로, 비선형적인 운동학 모델을 현재 상태에 대해 선형화한 행렬이다. 상태 벡터의 각 요소가 다른 요소에 시간에 따라 어떻게 영향을 미치는지를 나타낸다.
- $\mathbf{Q}_k$는 프로세스 노이즈 공분산 행렬이다. 이는 운동학 모델 자체의 불완전성이나 IMU 측정에 포함된 노이즈로 인해 발생하는 불확실성의 증가를 나타낸다. 이 행렬의 대각 성분들은 `EKF2_GYR_P_NOISE`, `EKF2_ACC_P_NOISE`와 같은 튜닝 파라미터에 의해 결정된다.

24x24 크기의 야코비안 행렬 $\mathbf{F}_k$를 실시간으로 계산하는 것은 임베디드 프로세서에 상당한 계산 부하를 유발한다. 이 문제를 해결하기 위해, PX4 개발팀은 SymForce와 같은 심볼릭 수학 도구를 사용하여 야코비안 행렬의 각 원소를 구성하는 복잡한 대수 방정식을 오프라인에서 미리 유도했다.3 그 결과로 생성된 C++ 코드가 

`covariance.cpp` 파일에 포함되어 있다.11 따라서 런타임 환경에서는 복잡한 편미분 계산 대신, 사전에 유도된 비교적 간단한 수식에 현재 상태 값을 대입하여 야코비안을 효율적으로 계산할 수 있다. 이는 제한된 연산 자원을 가진 비행 컨트롤러에서도 EKF2가 안정적으로 동작할 수 있게 하는 핵심적인 최적화 기법이다. 또한, 수치적 안정성을 높이기 위해 공분산 업데이트 시 표준적인 형태 대신 Joseph 안정화 형태(Joseph stabilized form)를 사용한다.3

## 5.  수정 단계: 센서 퓨전

예측 단계에서 IMU 데이터만으로 전파된 상태는 시간이 지남에 따라 오차가 필연적으로 누적된다. 수정 단계는 GPS, 지자기 센서, 기압계 등 다양한 보조 센서의 측정값을 사용하여 이 누적 오차를 보정하고 상태 추정치를 실제 세계에 맞게 재정렬하는 과정이다. EKF2는 각 센서의 특성에 맞는 정교한 퓨전 로직을 갖추고 있다.

### 5.1  자세 추정 (Attitude Estimation)

- Ekf::fuseMag(): 지자기 센서를 이용한 Yaw 각도 보정

  지자기 센서는 지구 자기장을 측정하여 절대적인 방향(자북)에 대한 정보를 제공하므로, 주로 방위각, 즉 Yaw 각도의 드리프트를 보정하는 데 결정적인 역할을 한다. EKF2_MAG_TYPE 파라미터를 통해 사용자는 비행 환경에 따라 두 가지 주요 퓨전 모드 중 하나를 선택할 수 있다.2

  1. **자기장 방향(Magnetic Heading) 퓨전**: 이 모드에서는 측정된 3축 자기장 벡터와 현재 EKF가 추정하고 있는 Roll/Pitch 각도를 사용하여 기체의 Yaw 각도를 직접 계산한다. 그리고 이 계산된 단일 Yaw 각도를 EKF의 측정값(observation)으로 사용한다. 이 방식은 자기장 이상 현상(magnetic anomaly)에 상대적으로 강건하지만, 정확도는 다소 떨어진다. 주로 이륙 전 초기 정렬이나 자기장 환경이 매우 불안정한 곳에서 유용하다.
  2. **3축 자기장(3-Axis Magnetometer) 퓨전**: 이 모드에서는 자기장 벡터의 X, Y, Z 성분을 3개의 독립적인 측정값으로 간주하여 EKF에 직접 퓨전한다. 이 방식은 더 높은 정확도를 제공하며, 상태 벡터에 포함된 지구 자기장(‘me‘)과 기체 고정 자기장 바이어스(‘mb‘)까지 함께 추정할 수 있다. 하지만 주변 자기장 환경이 안정적이라는 가정이 필요하다. `mag_fusion.cpp` 12 파일에는 이 두 모드에 대한 측정 모델 야코비안과 칼만 이득 계산 로직이 구현되어 있다.

- 중력 벡터를 이용한 Roll/Pitch 각도 암시적 보정

  코드베이스를 살펴보면 fuseAccelerometer()와 같은 명시적인 가속도계 퓨전 함수는 존재하지 않는다. 이는 가속도계가 Roll/Pitch 각도를 보정하는 방식이 암시적(implicit)이기 때문이다. 가속도계는 기체가 정지 또는 등속 운동을 할 때, 중력 벡터만을 측정해야 한다는 물리적 제약 조건을 제공한다. EKF는 이 원리를 다른 센서의 퓨전 과정에 통합하여 Roll과 Pitch를 보정한다.

  예를 들어, 호버링 상태에서 GPS 속도 퓨전(fuseGps())이 일어나는 경우를 생각해보자. GPS는 수평 속도가 0이라고 보고한다. EKF는 IMU 가속도 측정값에서 중력 성분을 제거하여 예측된 수평 속도를 계산하는데, 만약 Roll/Pitch 추정치가 틀렸다면 중력 성분이 잘못 제거되어 0이 아닌 수평 속도 예측값을 생성할 것이다. EKF는 GPS 측정값(0)과 예측값 사이의 차이(혁신)를 최소화하는 과정에서, 이 오차의 원인이 Roll/Pitch 자세 오차와 가속도계 바이어스에 있다고 판단하고 관련 상태 변수(쿼터니언, 가속도계 바이어스)를 수정한다. 이처럼 가속도계는 다른 센서와의 협력을 통해 간접적으로, 하지만 매우 효과적으로 수평 자세를 유지하는 역할을 수행한다.

### 5.2  위치 및 속도 추정 (Position and Velocity Estimation)

- Ekf::fuseGps(): GPS 데이터를 이용한 위치 및 속도 보정

  GPS는 EKF2의 위치 및 속도 드리프트를 막는 가장 중요한 외부 센서이다. EKF2_AID_MASK 파라미터의 첫 번째 비트를 설정하여 GPS 퓨전을 활성화할 수 있다.2 GPS 수신기는 위도, 경도, 고도 및 NED 기준 속도를 제공하며, EKF는 이 데이터를 내부의 로컬 NED 좌표계 위치/속도 추정치와 비교하여 혁신(innovation) 벡터를 계산하고 상태를 보정한다.

  GPS 퓨전이 성공적으로 이루어지기 위한 핵심적인 전제 조건은 Yaw 각도가 안정적으로 정렬(`yaw_align`)되어야 한다는 것이다.16 EKF의 내부 상태는 비행 시작점을 원점으로 하는 로컬 NED 좌표계에서 관리되지만, GPS 측정값은 지구 전체를 기준으로 하는 글로벌 좌표계(WGS84)로 제공된다. 이 두 좌표계 간의 데이터를 일치시키려면 둘 사이의 회전 관계, 즉 Yaw 각도를 정확히 알아야 한다. 만약 Yaw 정렬이 이루어지지 않았다면, '북쪽으로 1m/s 이동'이라는 GPS 정보를 로컬 좌표계의 어느 방향 속도로 반영해야 할지 알 수 없기 때문이다. 따라서 EKF는 지자기 센서나 다른 수단을 통해 Yaw를 먼저 정렬하고 

  `_control_status.flags.yaw_align` 플래그를 참으로 설정한 후에야 GPS 데이터 퓨전을 시작한다.16

  만약 지자기 센서가 없거나 신뢰할 수 없는 환경이라면, EKF는 EKF-GSF(Gaussian Sum Filter)라는 보조 필터를 내부적으로 실행하여 기체의 움직임으로부터 Yaw를 추정한다.2 기체가 한 방향으로 충분히 가속하면, 그 가속 방향(IMU로 측정)과 속도 벡터 방향(GPS로 측정)이 일치해야 한다는 원리를 이용하여 Yaw 각도를 추정할 수 있다.

- Ekf::fuseVelPosHeight(): 퓨전 로직 통합

  이 함수는 GPS, 외부 비전 시스템(VIO), 또는 기타 위치/속도 센서로부터 데이터를 받아 실제 칼만 필터 수정 단계를 수행하는 저수준 핵심 함수이다. 이 과정은 다음과 같은 표준적인 EKF 수정 절차를 따른다.

  1. **혁신 계산 (Compute Innovation)**: ‘yinnov=z−h(xk+1∣k)‘. 센서 측정값(`z`)과 예측된 상태(`x`)로부터 계산된 예측 측정값(`h(x)`)의 차이를 구한다.
  2. **측정 야코비안 계산 (Compute Jacobian)**: ‘H=∂x∂h‘. 비선형 측정 모델 `h`를 현재 상태에 대해 선형화한다.
  3. **혁신 공분산 계산 (Compute Innovation Covariance)**: ‘S=HPk+1∣kHT+R‘. 여기서 `R`은 센서 측정 노이즈 공분산이다 (예: `EKF2_GPS_P_NOISE`).
  4. **칼만 이득 계산 (Compute Kalman Gain)**: ‘K=Pk+1∣kHTS−1‘.
  5. **상태 수정 (Update State)**: ‘xk+1∣k+1=xk+1∣k+Kyinnov‘.
  6. **공분산 수정 (Update Covariance)**: ‘Pk+1∣k+1=(I−KH)Pk+1∣k‘.

### 5.3  고도 추정 (Altitude Estimation)

EKF2는 단일 고도 센서에 의존하지 않고, 여러 소스를 조합하여 더 정확하고 안정적인 고도 추정을 수행한다. `EKF2_HGT_REF` 파라미터를 통해 어떤 센서를 주 고도 참조 소스로 사용할지 선택할 수 있다.3

- **기압계 (Barometer)**: `EKF2_BARO_CTRL`로 활성화. 대기압을 측정하여 고도를 추정한다. 장기적인 고도 유지에 안정적이지만, 날씨 변화나 프로펠러 후류(propwash)에 의한 공기역학적 오차에 민감하다.3
- **레인지파인더 (Rangefinder)**: `EKF2_RNG_CTRL`로 활성화. 레이저나 초음파를 이용해 지면까지의 거리를 직접 측정하므로 지형 고도(AGL) 추정에 매우 정확하다. 하지만 측정 거리가 제한적이고 지형의 고저차에 직접적인 영향을 받는다.15
- **GPS 고도**: `EKF2_GPS_CTRL`로 활성화. 해수면 기준 절대 고도(AMSL)를 제공하지만, 일반적으로 수직 정확도가 수평 정확도보다 현저히 낮다.15
- **외부 비전 (External Vision)**: `EKF2_HGT_REF`를 "Vision"으로 설정하여 활성화. VIO 시스템으로부터 고도 정보를 받아 퓨전한다.15

EKF2는 이러한 센서들을 상황에 맞게 조합한다. 예를 들어, 저고도 정밀 착륙 시에는 레인지파인더를, 일반 순항 비행 시에는 기압계를 주로 사용하며, GPS 고도를 장기적인 기압계 드리프트 보정에 활용하는 등 유연한 퓨전 전략을 구사한다.

### 5.4  각속도 추정 (Angular Velocity Estimation)

한 가지 명확히 해야 할 점은, EKF2의 상태 벡터는 각속도를 직접 포함하지 않는다는 것이다. 각속도는 자이로스코프 센서에서 직접 측정되는 값으로 간주된다. 대신 EKF가 추정하는 것은 자이로스코프 센서 자체의 **바이어스 오차**(‘bg‘, 상태 벡터 인덱스 10-12)이다.

추정 원리는 다음과 같다.

1. 자이로스코프의 원시 측정값(‘ωmeas‘)은 실제 각속도(‘ωtrue‘)에 바이어스(‘bg‘)와 노이즈(‘ng‘)가 더해진 값이다.

2. EKF는 예측 단계에서 이 측정값을 사용하여 자세를 전파한다. 만약 바이어스가 존재한다면, 예측된 자세는 실제 자세와 점점 멀어질 것이다.

3. 수정 단계에서 지자기 센서나 GPS 등 다른 센서의 정보를 사용하여 이 누적된 자세 오차를 보정한다.

4. 칼만 필터는 이 보정 과정에서 "이러한 자세 오차를 지속적으로 유발하는 원인은 자이로스코프의 체계적인 오차, 즉 바이어스일 것이다"라고 추론하고, 상태 벡터 내의 자이로 바이어스(‘b^g‘) 값을 업데이트하여 오차를 줄이는 방향으로 조정한다.17

5. 최종적으로 시스템의 다른 모듈(예: 제어기)에 제공되는 '추정된 각속도'는 원시 측정값에서 이 추정된 바이어스를 뺀 값이다.

   $$
   \boldsymbol{\omega}_{est} = \boldsymbol{\omega}_{meas} - \hat{\mathbf{b}}_g
   $$

결론적으로, EKF는 비행 중에 실시간으로 자이로스코프 센서를 '온라인 캘리브레이션'하는 역할을 수행하여, 더 깨끗하고 정확한 각속도 정보를 생성한다.

## 6.  결론 및 개발자 권장 사항

본 분석을 통해 PX4-Autopilot의 EKF2가 단순한 칼만 필터 구현체를 넘어, 실제 비행 환경의 다양한 제약과 비이상적인 조건들을 극복하기 위해 설계된 견고하고 신뢰성 높은 상태 추정 시스템임이 드러났다. 개발자가 EKF2의 성능을 최대한 활용하고 문제를 해결하기 위해서는 그 설계 철학을 이해하고 핵심 파라미터를 적절히 튜닝하는 것이 중요하다.

### 6.1. EKF2 설계 철학 요약

EKF2의 설계는 **분리(Separation)**, **적응(Adaptation)**, **이중화(Redundancy)**라는 세 가지 핵심 철학으로 요약될 수 있다.

- **분리**: 순수 알고리즘(ECL)과 플랫폼 통합(module)의 명확한 분리는 코드의 재사용성과 테스트 용이성을 보장한다. 또한, IMU 기반의 고주파 예측과 보조 센서 기반의 저주파 수정을 분리함으로써 시스템의 동적 응답성과 안정성을 동시에 확보한다.
- **적응**: 비행 조건과 센서 상태에 따라 지자기 퓨전 모드를 자동으로 전환하고, 주 고도 소스를 유연하게 선택하는 기능은 다양한 환경에 대한 적응력을 높인다.
- **이중화**: 다중 EKF 인스턴스와 페일오버 메커니즘은 단일 센서의 고장이 전체 시스템의 실패로 이어지는 것을 방지하여 임무 수행의 신뢰성을 극대화한다.

이러한 설계 철학은 EKF2가 이론적 최적성을 넘어 실제 세계에서의 실용성과 강인함(robustness)을 목표로 개발되었음을 명확히 보여준다.

### 6.1  튜닝을 위한 핵심 파라미터

EKF2의 성능은 사용되는 기체의 물리적 특성과 센서 품질에 맞춰 파라미터를 정밀하게 튜닝하는 것에 크게 좌우된다. 다음 표는 성능에 가장 큰 영향을 미치는 핵심 파라미터와 그 튜닝 가이드를 요약한 것이다.

**Table 2: Key EKF2 Tuning Parameters and Their Impact**

| 파라미터 (Parameter)                                        | 설명 (Description)                                           | 튜닝 가이드 (Tuning Guide)                                   | 관련 자료 |
| ----------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | --------- |
| `EKF2_*_NOISE` (e.g., `EKF2_GPS_P_NOISE`, `EKF2_ACC_NOISE`) | 각 센서 측정값의 노이즈(불확실성) 수준을 필터에 알려준다. 측정 노이즈 공분산 ‘R‘ 행렬을 구성한다. | 값을 **낮추면** 해당 센서를 더 신뢰하게 되어 추정치가 측정값에 더 가깝게 따라간다. 값을 **높이면** 센서를 덜 신뢰하게 되어 추정치가 더 부드러워지고 IMU 예측에 더 의존한다. 너무 낮추면 노이즈가 심해지고, 너무 높이면 센서 데이터가 무시되어 드리프트가 발생할 수 있다. | 18        |
| `EKF2_AID_MASK`                                             | 어떤 보조 센서를 퓨전에 사용할지 결정하는 비트마스크. (GPS, Optical Flow, External Vision 등) | 특정 센서를 사용하려면 해당 비트를 활성화해야 한다. 예를 들어, GPS만 사용하려면 1, Optical Flow만 사용하려면 2로 설정한다. | 2         |
| `EKF2_HGT_REF`                                              | 주 고도 참조 센서를 선택한다. (Baro, GPS, Rangefinder, Vision) | 비행 환경에 가장 적합한 센서를 선택한다. 저고도 정밀 호버링에는 Rangefinder, 일반적인 비행에는 Baro가 적합하다. | 3         |
| `EKF2_MAG_TYPE`                                             | 지자기 퓨전 방식을 선택한다. (Automatic, Heading, 3-axis, None) | 일반적으로 'Automatic'이 권장된다. 자기장 간섭이 심한 환경에서는 'Heading' 모드를 수동으로 선택하여 안정성을 높일 수 있다. | 2         |
| `EKF2_*_DELAY`                                              | 각 센서의 지연 시간을 보상한다 (ms).                         | 정확한 지연 시간 값을 설정하면 '지연 퓨전 시간축'이 효과적으로 동작하여 동적 성능이 향상된다. 로그 분석을 통해 측정할 수 있다. | 2         |
| `EKF2_GYR_P_NOISE`, `EKF2_ACC_P_NOISE`                      | 프로세스 노이즈. IMU 모델의 불확실성을 나타낸다.             | 이 값을 높이면 필터가 IMU 예측을 덜 신뢰하게 되어, 보조 센서의 수정에 더 민감하게 반응한다. 바이어스 추정 속도에 영향을 준다. | -         |

### 6.2  일반적인 문제 및 디버깅 팁 (로그 분석 포함)

- **위치 추정치 발산 (Position Divergence)**:

  - **원인**: 불량한 GPS 신호(예: 빌딩 밀집 지역2), 과도한 진동으로 인한 가속도계 클리핑, 심각한 지자기 센서 간섭, 또는 부정확한 센서 지연 시간(`EKF2_*_DELAY`) 설정.

  - **해결**: GPS 안테나의 위치와 방향을 개선하고, 기체의 진동을 댐핑으로 줄인다. 지자기 센서를 재보정하고 전력선으로부터 멀리 이격시킨다. 로그 분석을 통해 센서 지연 시간을 확인하고 파라미터를 조정한다.

- **"화장실 물 내림" 현상 (Toilet Bowl Effect)**:

  - **원인**: 주로 지자기 센서의 부정확한 캘리브레이션이나 심한 자기장 간섭으로 인해 Yaw 추정치가 불안정할 때 발생한다. 잘못된 Yaw 정보로 인해 위치 제어기가 기체를 잘못된 방향으로 밀어내면서, 기체가 제자리에서 빙글빙글 도는 현상이다.
  - **해결**: 지자기 센서를 재보정하고, 주변에 자기장을 발생시키는 부품(모터, 배터리, 전력선)이 없는지 확인한다. 임시방편으로 `EKF2_MAG_TYPE`을 'Heading'으로 설정하여 안정성을 테스트해볼 수 있다.

- 로그 분석의 중요성:

  EKF2의 문제를 진단하는 가장 강력한 도구는 비행 로그(.ulg 파일) 분석이다. QGroundControl과 같은 도구를 사용하여 다음 메시지들을 중점적으로 확인해야 한다.

  - **`estimator_status`**: `filter_fault_flags` 필드를 통해 EKF 내부에서 수치적 오류나 심각한 문제가 발생했는지 확인할 수 있다.2 또한 `hagl_test_ratio`, `vel_test_ratio`, `pos_test_ratio`와 같은 '혁신 테스트 비율'을 통해 각 센서 퓨전이 얼마나 일관성 있게 이루어지고 있는지(즉, 측정값이 예측 범위 내에 있는지) 확인할 수 있다.

  - **`ekf2_innovations`**: 각 센서의 '혁신(innovation)' 값을 시간 축에 따라 플로팅하는 것은 매우 중요하다. 혁신은 센서 측정값과 필터의 예측값 사이의 차이를 의미한다. 만약 특정 센서의 혁신 값이 지속적으로 크거나 한쪽으로 치우쳐 있다면, 해당 센서의 노이즈 파라미터(`EKF2_*_NOISE`)가 너무 작게 설정되었거나, 센서 정렬 또는 캘리브레이션에 문제가 있을 가능성이 높다. 특히4에서 지적된 바와 같이, 기체가 가속할 때만 혁신 값이 급증한다면 이는 센서 시간 동기화 문제(`EKF2_*_DELAY` 파라미터)일 가능성이 매우 크다.

  - **`yaw_estimator_status`**: 지자기 센서 없이 비행하는 경우, EKF-GSF의 동작 상태와 각 하위 필터의 Yaw 추정치를 이 메시지를 통해 확인할 수 있다.2

  #### **참고 자료**

  1. Switching State Estimators | PX4 User Guide (v1.14) - PX4 docs, accessed July 29, 2025, https://docs.px4.io/v1.14/en/advanced/switching_state_estimators.html
  2. ECL/EKF Overview & Tuning - PX4 docs, accessed July 29, 2025, https://docs.px4.io/v1.11/en/advanced_config/tuning_the_ecl_ekf.html
  3. Using PX4's Navigation Filter (EKF2) | PX4 Guide (main), accessed July 29, 2025, https://docs.px4.io/main/en/advanced_config/tuning_the_ecl_ekf
  4. Feeding mocap pose as vision pose into EKF2 does not wrok · Issue #8520 · PX4/PX4-Autopilot - GitHub, accessed July 29, 2025, https://github.com/PX4/PX4-Autopilot/issues/8520
  5. Using PX4's Navigation Filter (EKF2) - GitHub, accessed July 29, 2025, https://github.com/PX4/PX4-user_guide/blob/main/tr/advanced_config/tuning_the_ecl_ekf.md
  6. PX4-Autopilot/src/modules/ekf2/EKF2.cpp at main - GitHub, accessed July 29, 2025, https://github.com/PX4/PX4-Autopilot/blob/main/src/modules/ekf2/EKF2.cpp
  7. PX4/PX4-ECL: Estimation & Control Library for Guidance, Navigation and Control Applications - GitHub, accessed July 29, 2025, https://github.com/PX4/PX4-ECL
  8. PX4-ECL/EKF/ekf.cpp at master - GitHub, accessed July 29, 2025, https://github.com/PX4/PX4-ECL/blob/master/EKF/ekf.cpp
  9. px4/PX4-Autopilot - The Dispatch Report: OSS Watchlist, accessed July 29, 2025, https://thedispatch.ai/reports/1382/
  10. PX4-ECL/EKF/control.cpp at master - GitHub, accessed July 29, 2025, https://github.com/PX4/ecl/blob/master/EKF/control.cpp
  11. PX4-ECL/EKF/covariance.cpp at master - GitHub, accessed July 29, 2025, https://github.com/PX4/ecl/blob/master/EKF/covariance.cpp
  12. PX4-ECL/EKF/mag_fusion.cpp at master - GitHub, accessed July 29, 2025, https://github.com/PX4/ecl/blob/master/EKF/mag_fusion.cpp
  13. PX4 Firmware: src/lib/ecl/EKF/terrain_estimator.cpp Source File - GitHub Pages, accessed July 29, 2025, https://px4.github.io/Firmware-Doxygen/da/d47/ecl_2_e_k_f_2terrain__estimator_8cpp_source.html
  14. ekf.cpp - inertialsense/nav-ecl - GitHub, accessed July 29, 2025, https://github.com/inertialsense/nav-ecl/blob/master/EKF/ekf.cpp
  15. Using PX4's Navigation Filter (EKF2) | PX4 Guide (main), accessed July 29, 2025, https://docs.px4.io/main/zh/advanced_config/tuning_the_ecl_ekf.html
  16. EKF2 GPS fusion without heading source - PX4 Autopilot, accessed July 29, 2025, https://discuss.px4.io/t/ekf2-gps-fusion-without-heading-source/33587
  17. PX4-ECL/EKF/ekf_helper.cpp at master - GitHub, accessed July 29, 2025, https://github.com/PX4/ecl/blob/master/EKF/ekf_helper.cpp
  18. EKF2 tuning for MOCAP tracking - PX4 Discussion Forum, accessed July 29, 2025, https://discuss.px4.io/t/ekf2-tuning-for-mocap-tracking/20618
  
  
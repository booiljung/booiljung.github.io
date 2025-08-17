# Jetson Orin 기반 고성능 드론의 상태 추정을 위한 고주파 IMU 선정 및 ROS2 Humble 통합

## 1. 서론

### 0.1  고성능 자율 비행의 전제 조건: 정밀 상태 추정

현대 무인 항공기(Unmanned Aerial Vehicle, UAV), 즉 드론 기술은 단순한 소비자용 촬영 장비를 넘어, 정밀 농업, 인프라 감시, 군사 정찰, 물류 운송 등 고도의 정확성과 신뢰성을 요구하는 산업 및 군사 영역으로 빠르게 확장되고 있다.1 이러한 고부가가치 임무 환경에서 드론의 핵심 경쟁력은 '정밀성(Precision)'과 '안정성(Stability)'으로 귀결된다. 이 두 가지 요소를 충족시키기 위한 가장 근본적인 기술적 전제 조건이 바로 '정밀 상태 추정(Precise State Estimation)'이다.

상태 추정이란 비행 중인 기체가 현재 어떠한 자세(Attitude), 속도(Velocity), 그리고 위치(Position)에 있는지를 실시간으로 정확하게 추적하는 과정을 의미한다.4 이는 자율 비행을 위한 모든 후속 단계, 즉 경로 계획(Path Planning), 장애물 회피(Obstacle Avoidance), 임무 수행(Mission Execution) 등의 기반이 되는 가장 원초적인 정보이다. 부정확한 상태 추정 정보는 잘못된 제어 명령으로 이어져 임무 실패는 물론 기체 손실이라는 치명적인 결과를 초래할 수 있다. 따라서, 어떠한 비행 환경에서도 신뢰할 수 있는 상태 추정 파이프라인을 구축하는 것은 고성능 자율 비행 드론 개발의 성패를 가르는 첫 번째 관문이라 할 수 있다.

### 0.2  관성 측정 장치(IMU): 폐쇄 루프 제어의 심장

드론의 안정적인 비행 제어는 본질적으로 폐쇄 루프 제어(Closed-Loop Control) 시스템에 의존한다. 이 시스템의 심장부에서 가장 빠르고 중요한 피드백 신호를 제공하는 센서가 바로 관성 측정 장치(Inertial Measurement Unit, IMU)이다.1 IMU는 내장된 자이로스코프(Gyroscope)와 가속도계(Accelerometer)를 통해 드론의 3차원 공간 내 운동 상태, 즉 각속도(Angular Velocity)와 가속도(Acceleration)를 수백에서 수천 헤르츠(Hz)에 달하는 매우 높은 주파수로 직접 측정한다.5

비행 제어 시스템(Flight Control System, FCS)은 IMU로부터 실시간으로 전달받은 이 관성 데이터를 바탕으로 현재 기체의 미세한 기울어짐이나 흔들림을 즉각적으로 인지한다. 이후, 제어 알고리즘을 통해 목표 상태를 유지하기 위해 필요한 모터의 출력 변화량을 계산하고, 각 모터에 명령을 전달하여 기체의 자세를 바로잡는다. 수정된 기체의 움직임은 다시 IMU에 의해 측정되어 FCS로 피드백되며, 이 과정이 끊임없이 반복되면서 안정적인 비행이 유지되는 폐쇄 루프가 형성된다.1

GPS나 기압계(Barometer)와 같은 다른 센서들도 상태 추정에 중요한 정보를 제공하지만, 이들의 데이터 업데이트 속도는 보통 5-10 Hz 수준으로 매우 느리다.7 따라서 급격한 바람이나 조종 명령에 대응해야 하는 드론의 고기동(High-dynamic) 자세 제어에는 단독으로 사용될 수 없다.1 오직 IMU만이 제어 루프가 요구하는 실시간성과 고주파 피드백을 만족시킬 수 있으며, 이것이 IMU를 드론 안정성 제어의 핵심 코어로 부르는 이유이다.

### 0.3  보고서의 목표 및 구성

본 보고서는 NVIDIA Jetson Orin 플랫폼을 기반으로 고성능 자율 비행 드론을 개발하는 엔지니어 및 연구자를 대상으로 한다. Jetson Orin의 강력한 AI 연산 능력을 최대한 활용하여 정밀하고 강인한(robust) 상태 추정 시스템을 구축하는 데 필요한 포괄적인 기술 가이드를 제공하는 것을 목표로 한다.

이를 위해 본 보고서는 다음과 같이 구성된다.

- **2장**에서는 관성 항법의 원리와 오차 모델, 고주파 샘플링의 필요성, 그리고 확장 칼만 필터(EKF) 기반 센서 융합의 수학적 원리를 포함한 이론적 배경을 심도 있게 다룬다.
- **3장**에서는 고성능 IMU를 선택하기 위한 핵심 성능 지표(KPI)를 분석하고, 실제 시장에서 구할 수 있는 유력한 MEMS IMU 모델들의 데이터시트를 기반으로 정량적인 비교 분석을 수행하여 프로젝트 목표에 맞는 최적의 센서를 선정하는 기준을 제시한다.
- **4장**에서는 선정된 IMU를 Jetson Orin의 40핀 확장 헤더에 물리적으로 연결하고, 고속 데이터 통신을 위한 SPI 인터페이스를 활성화하는 하드웨어 설정 과정을 상세히 안내한다.
- **5장**에서는 ROS2 Humble 환경에서 IMU 드라이버를 설치하고, 표준 상태 추정 패키지인 `robot_localization`을 사용하여 EKF 파이프라인을 구축, 실행, 검증하는 전 과정을 단계별로 설명한다.

이러한 유기적인 구성을 통해, 독자는 드론 상태 추정에 대한 깊이 있는 이론적 이해를 바탕으로 실제 하드웨어를 선정하고, 이를 Jetson Orin과 ROS2라는 표준 개발 환경에 성공적으로 통합하는 실무 역량을 확보하게 될 것이다.

## 1.  드론 상태 추정의 이론적 배경

### 1.1  관성 항법(Inertial Navigation)과 오차 누적

관성 항법 시스템(Inertial Navigation System, INS)은 외부의 도움 없이 오직 관성 센서의 측정값만을 이용해 이동체의 위치, 속도, 자세를 추정하는 기술이다. 그 기본 원리는 뉴턴의 운동 법칙에 근거한 추측 항법(Dead Reckoning)이다.8 IMU의 가속도계는 기체의 비관성 가속도(Specific Force)를, 자이로스코프는 각속도를 측정한다. 이 측정값들을 시간에 대해 적분함으로써 속도와 자세를, 그리고 속도를 다시 한번 적분하여 위치를 계산할 수 있다.8

하지만 이 방식은 치명적인 약점을 내포하고 있다. 바로 '오차 누적(Error Accumulation)'이다. IMU 센서의 측정값에는 필연적으로 미세한 오차가 포함되어 있다. 이 오차는 바이어스(Bias), 노이즈(Noise), 축 부정렬(Misalignment), 스케일 팩터 오류(Scale Factor Error) 등 다양한 형태로 나타난다.9 이 미세한 오차들이 적분 과정을 거치면서 시간에 따라 눈덩이처럼 불어난다. 예를 들어, 가속도 측정값에 작은 양(+)의 바이어스가 존재한다면, 이를 적분한 속도는 시간에 비례하여 선형적으로 증가하고, 다시 적분한 위치는 시간에 제곱하여 포물선 형태로 발산하게 된다.8 이로 인해 INS는 단독으로 사용할 경우, 시간이 지남에 따라 실제 상태와 추정 상태 간의 차이, 즉 드리프트(Drift)가 걷잡을 수 없이 커져 결국 항법 정보를 신뢰할 수 없게 된다.11

### 1.2  IMU 측정 모델 및 확률적 오차

이러한 오차를 분석하고 보정하기 위해 IMU의 측정 과정을 수학적 모델로 표현할 필요가 있다. 특정 축에 대한 자이로스코프의 측정값 $\tilde{\omega}(t)$는 실제 각속도 $\omega(t)$에 두 가지 주요 확률적 오차(Stochastic Errors)가 더해진 형태로 모델링할 수 있다.14
$$
\tilde{\omega}(t) = \omega(t) + b(t) + n(t)
$$
여기서 각 항은 다음과 같은 의미를 가진다.

- $\omega(t)$: 참(True) 각속도. 우리가 알고자 하는 실제 물리량이다.
- $b(t)$: 시간에 따라 천천히 변하는 바이어스(Bias). 이는 다시 두 가지로 나뉜다. 하나는 센서 전원을 켤 때마다 미세하게 달라지는 '턴온 바이어스(Turn-on Bias)'이고, 다른 하나는 작동 중에 온도 변화나 부품 노후화 등으로 인해 불규칙하게 변동하는 '바이어스 불안정성(Bias Instability)'이다.9 이 바이어스 변동은 주로 '랜덤 워크(Random Walk)' 프로세스로 모델링된다.14
- $n(t)$: 평균이 0이고 매우 빠르게 변동하는 가우시안 백색 잡음(White Noise). 이 잡음은 센서의 물리적, 전자적 특성에서 기인하며, 측정값에 단기적인 떨림(Jitter)을 유발한다. 이 백색 잡음이 적분 과정에 미치는 영향을 정량화한 지표가 각각 자이로스코프의 '각속도 랜덤 워크(Angular Random Walk, ARW)'와 가속도계의 '속도 랜덤 워크(Velocity Random Walk, VRW)'이며, 이는 데이터시트 상의 노이즈 밀도(Noise Density) 사양과 직접적인 관련이 있다.10

이 모델은 상태 추정 필터(예: 칼만 필터)가 센서의 오차 특성을 이해하고, 이를 바탕으로 오차를 추정하고 보정하는 데 필수적인 수학적 기초를 제공한다.

### 1.3  고주파 샘플링의 필요성: 에일리어싱과 위상 지연

드론과 같이 동적인 시스템에서 IMU의 데이터 출력 속도(Output Data Rate, ODR), 즉 샘플링 주파수는 상태 추정의 성능과 안정성에 결정적인 영향을 미친다. 고주파 샘플링이 필요한 이유는 크게 두 가지, 에일리어싱 방지와 위상 지연 최소화이다.

- **에일리어싱(Aliasing) 방지:** 드론은 모터와 프로펠러의 회전으로 인해 수백 Hz에 달하는 고주파 진동에 항상 노출된다.7 나이퀴스트-섀넌 샘플링 이론에 따르면, 어떤 신호를 왜곡 없이 디지털로 표현하기 위해서는 해당 신호에 포함된 가장 높은 주파수 성분의 최소 2배 이상의 속도로 샘플링해야 한다. 만약 IMU의 샘플링 주파수가 드론의 진동 주파수보다 낮으면, 이 고주파 진동이 실제로는 존재하지 않는 저주파 움직임으로 잘못 해석되는 '에일리어싱' 현상이 발생한다. 비행 제어기는 이 '유령' 신호를 실제 기체의 움직임으로 착각하고 불필요한 보정 명령을 내리게 되며, 이는 제어 시스템의 불안정성을 야기하고 심각한 경우 기체를 추락시키는 원인이 될 수 있다.7 따라서 높은 샘플링 주파수는 드론의 진동 성분을 올바르게 필터링하고 제어 시스템이 '진실된' 정보만을 사용하도록 보장하는 역할을 한다.
- **위상 지연(Phase Lag) 최소화:** 제어 루프는 '측정 -->> 계산 -->> 작동'의 순서로 동작하며, 각 단계에는 미세한 시간 지연이 존재한다. IMU의 데이터 출력 속도가 낮다는 것은 제어기가 더 '오래된' 과거의 상태 정보를 기반으로 현재의 제어량을 결정한다는 의미이다. 이 시간 지연을 '위상 지연'이라고 하며, 시스템의 반응성을 떨어뜨리고 안정성을 저해하는 주된 요인이다.1 특히 드론이 급격하게 기동할 때, 위상 지연이 크면 제어 명령이 실제 기체의 움직임을 따라가지 못해 진동이 증폭되거나 제어가 발산할 수 있다. 고주파 IMU는 이 지연 시간을 최소화하여 제어기가 가능한 한 '현재'에 가까운, 즉 '시기적절한' 데이터를 바탕으로 판단하게 함으로써 제어 루프의 안정성과 응답성을 극대화한다.1

결론적으로, 고주파 IMU를 사용하는 것은 단순히 더 많은 데이터를 얻는 차원을 넘어, 제어 시스템의 '진실성(에일리어싱 방지)'과 '적시성(위상 지연 최소화)'을 보장함으로써 드론의 근본적인 비행 안정성을 확보하는 핵심 전략이다.

### 1.4  확장 칼만 필터(EKF) 기반 센서 융합

IMU의 누적 오차 문제를 해결하기 위한 가장 효과적인 방법은 다른 종류의 센서와 데이터를 융합(Sensor Fusion)하는 것이다. GPS, 비전 센서, LiDAR 등은 IMU와 달리 드리프트 오차가 없거나 매우 적어 절대적인 위치나 자세 정보를 제공할 수 있다. 이러한 서로 다른 특성을 가진 센서들의 데이터를 통계적으로 최적의 방식으로 결합하는 데 널리 사용되는 알고리즘이 바로 칼만 필터(Kalman Filter)이다.4

드론의 운동 모델은 회전 변환 등으로 인해 비선형적(non-linear)이기 때문에, 표준 칼만 필터를 직접 적용할 수 없다. 확장 칼만 필터(Extended Kalman Filter, EKF)는 이러한 비선형 시스템에 칼만 필터를 적용하기 위해, 현재 추정값 주변에서 테일러 급수 전개를 통해 시스템을 선형 근사하는 기법이다.12 EKF는 재귀적으로 두 단계를 반복하며 상태를 추정한다.

- **예측 단계 (Prediction Step):** 이 단계에서는 외부 센서의 도움 없이, 이전 시간 단계($t-1$)에서 추정된 상태($x_{t-1}$)와 현재 IMU의 고주파 측정값($u_t$, 각속도 및 가속도)만을 사용하여 현재 시간 단계($t$)의 상태($\hat{x}_t$)를 예측한다. 이 과정은 관성 항법과 동일하며, IMU의 오차가 그대로 반영되어 불확실성(공분산 행렬 $P$)이 증가한다.17
  $$
  \hat{x}_t = f(x_{t-1}, u_t)
  $$

  $$
  \hat{P}_t = F_{t-1} P_{t-1} F_{t-1}^T + Q_{t-1}
  $$

  여기서 $f$는 비선형 상태 전이 모델, $F$는 $f$의 자코비안 행렬, $Q$는 프로세스 노이즈 공분산이다.

- **보정 단계 (Correction/Update Step):** GPS나 비전 센서와 같은 외부 센서로부터 새로운 측정값($z_t$)이 들어오면, 이 측정값과 예측된 상태($\hat{x}_t$)를 비교하여 그 차이, 즉 '이노베이션(Innovation)'($v_t$)을 계산한다. 칼만 이득(Kalman Gain, $K_t$)은 예측의 불확실성($\hat{P}_t$)과 측정의 불확실성($R_t$)을 고려하여 이노베이션을 얼마나 신뢰할지 결정하는 가중치 역할을 한다. 최종적으로 이노베이션에 칼만 이득을 곱한 값을 예측된 상태에 더하여 상태를 보정($x_t$)하고, 불확실성을 감소시킨다. 이 과정을 통해 IMU의 누적 오차가 주기적으로 초기화되고, 전체 시스템의 추정 정확도가 유지된다.11
  $$
  v_t = z_t - h(\hat{x}_t) \\
  K_t = \hat{P}_t H_t^T (H_t \hat{P}_t H_t^T + R_t)^{-1} \\
  x_t = \hat{x}_t + K_t v_t \\
  P_t = (\mathbb{I} - K_t H_t) \hat{P}_t \\
  $$
  여기서 $h$는 비선형 측정 모델, $H$는 $h$의 자코비안 행렬, $R$은 측정 노이즈 공분산이다.

이처럼 EKF는 IMU의 고주파 데이터를 통해 부드럽고 연속적인 상태 변화를 예측하고, 외부 센서의 저주파 데이터를 통해 누적 오차를 보정하는 상호보완적인 융합 메커니즘을 제공한다.

## 2.  고성능 IMU 선정 기준 및 모델 비교

Jetson Orin 기반 고성능 드론에 적합한 IMU를 선정하기 위해서는 데이터시트에 명시된 여러 성능 지표를 종합적으로 이해하고 프로젝트의 요구사항과 비교 분석하는 과정이 필수적이다.

### 2.1  핵심 성능 지표(KPI) 심층 분석

- **바이어스 안정성 (Bias Instability, 단위: °/h 또는 mg):** 센서가 일정한 온도에서 장시간 작동할 때, 출력값의 평균(바이어스)이 얼마나 안정적으로 유지되는지를 나타내는 지표이다. 이 값이 낮을수록 시간이 지나도 바이어스의 변동이 적어 장시간 비행 시 자세 및 위치 추정의 드리프트가 감소한다. 이는 IMU의 장기적인 정밀도를 결정하는 가장 중요한 지표 중 하나로, 특히 GPS 신호가 없는 환경에서 항법 성능을 좌우한다.2
- **노이즈 밀도 (Noise Density, 단위: °/s/√Hz 또는 g/√Hz):** 센서 출력에 포함된 고주파 랜덤 노이즈의 크기를 나타낸다. 데이터시트에서는 종종 '각속도 랜덤 워크(Angular Random Walk, ARW)' 또는 '속도 랜덤 워크(Velocity Random Walk, VRW)'라는 이름으로 표현되기도 한다. 이 값이 낮을수록 단기적인 측정값의 떨림(jitter)이 적고, 상태 추정 결과가 부드러워져 제어 시스템의 안정성에 기여한다.10
- **데이터 출력 속도 (ODR, Hz) 및 대역폭 (Bandwidth, Hz):** ODR(Output Data Rate)은 초당 데이터가 출력되는 횟수를, 대역폭은 센서가 감지할 수 있는 움직임의 최대 주파수를 의미한다. 드론 레이싱이나 공격적인 기동과 같이 매우 빠른 자세 변화를 정확하게 추적하고 제어하기 위해서는 최소 500 Hz, 권장 1 kHz 이상의 높은 ODR이 필수적이다. 이는 2.3절에서 설명한 에일리어싱 방지 및 위상 지연 최소화와 직결된다.13
- **SWaP-C (Size, Weight, and Power, and Cost):** 크기, 무게, 전력 소모, 그리고 비용은 드론 시스템 설계에서 항상 고려해야 할 현실적인 제약 조건이다. IMU의 무게와 크기는 드론의 전체 중량과 페이로드 용량에 영향을 미치고, 전력 소모는 비행 시간을 결정하는 중요한 요소이다. 고성능 IMU는 일반적으로 더 비싸고 전력 소모가 크므로, 임무 요구사항과 예산에 맞는 최적의 균형점을 찾아야 한다.2
- **내환경성 (Environmental Resilience):** 드론은 비행 중 모터에서 발생하는 강한 진동, 급격한 온도 변화, 착륙 시의 충격 등 가혹한 환경에 노출된다. 데이터시트에 명시된 작동 온도 범위, 충격 저항(g), 그리고 진동에 대한 강건성(vibration robustness)은 센서의 신뢰성과 직결된다. 특히 진동에 강하게 설계된 IMU는 별도의 댐핑 구조 없이도 깨끗한 데이터를 제공하여 제어 안정성을 크게 향상시킬 수 있다.2

### 2.2  추천 IMU 모델 비교 분석

Jetson Orin 기반 고성능 드론 프로젝트에 적용할 수 있는 대표적인 고주파 MEMS IMU 세 가지 모델을 선정하여 핵심 사양을 비교 분석하였다. 이 표는 개발자가 각 센서의 강점과 약점을 명확히 파악하고, 자신의 프로젝트 요구사항과 개발 우선순위에 기반하여 정보에 입각한 결정을 내리는 데 결정적인 정보를 제공한다. 선택된 모델들은 각각 '최고의 원시 성능', '최상의 소프트웨어 생태계', 그리고 '온보드 퓨전'이라는 뚜렷한 특징을 대표한다.

**Table 1: 주요 고주파 MEMS IMU 성능 비교**

| 지표 (Indicator)           | TDK ICM-42688-P         | Bosch BMI088                   | CEVA BNO085                   | 근거 (Source) |
| -------------------------- | ----------------------- | ------------------------------ | ----------------------------- | ------------- |
| **자이로 노이즈 밀도**     | **2.8 mdps/√Hz**        | 14 mdps/√Hz                    | N/A (Fusion Engine)           | 22            |
| **가속도계 노이즈 밀도**   | **70 µg/√Hz**           | 175-230 µg/√Hz                 | N/A (Fusion Engine)           | 22            |
| **자이로 바이어스 안정성** | N/A (±5 mdps/°C TempCo) | **< 2 °/h**                    | N/A (Fusion Engine)           | 21            |
| **최대 ODR (Gyro/Accel)**  | **8 kHz / 8 kHz**       | 2 kHz / 1.6 kHz                | 200 Hz (Fused) / 1 kHz (Raw)  | 21            |
| **인터페이스**             | I2C, SPI (24MHz)        | I2C, SPI                       | I2C, SPI, UART                | 21            |
| **온보드 퓨전 엔진**       | 아니요                  | 아니요                         | **예 (SH-2)**                 | 25            |
| **ROS2 드라이버 지원**     | 커뮤니티/직접 개발 필요 | **NVIDIA Isaac ROS 공식 지원** | 커뮤니티 (Python 기반)        | 26            |
| **단가 (1개 기준, USD)**   | ~$4.80                  | ~$5.23                         | ~$13.05 (칩) / ~$24.95 (보드) | 29            |

이 비교표는 단순한 스펙 나열을 넘어, 엔지니어링 의사결정을 직접적으로 지원하는 도구로서의 가치를 지닌다. 예를 들어, 제어 루프의 성능을 한계까지 끌어올리기 위해 가장 깨끗하고 빠른 원시 데이터가 필요한 연구 프로젝트라면 ICM-42688-P의 압도적인 노이즈 성능과 ODR에 주목할 것이다. 반면, 정해진 기간 내에 안정적인 제품을 출시해야 하는 상업 프로젝트라면 BMI088이 제공하는 NVIDIA의 공식 ROS2 드라이버 지원이 주는 개발 시간 단축과 신뢰성 확보라는 이점을 더 높게 평가할 것이다. BNO085는 메인 프로세서의 부하를 줄이고자 할 때 온보드 퓨전 엔진이라는 대안을 제시한다. 이처럼 각 센서는 서로 다른 개발 철학과 우선순위를 만족시키는 고유한 가치를 제공한다.

### 2.3  최종 모델 선정 및 근거

분석 결과를 바탕으로, 프로젝트의 목표에 따라 다음과 같이 최종 모델을 추천할 수 있다.

- 최고 성능 추구 시: TDK ICM-42688-P

  가장 낮은 노이즈 밀도와 8 kHz에 달하는 압도적인 ODR은 EKF 알고리즘의 성능을 이론적으로 극대화할 수 있는 최상의 원시 데이터를 제공한다.22 이는 제어 알고리즘 연구나 최고 수준의 기동 성능을 요구하는 특수 목적 드론 개발에 가장 적합한 선택이다. 단, 공식 ROS2 드라이버가 존재하지 않아26, 개발자가 직접 SPI 통신 프로토콜을 구현하고 ROS2 노드를 작성해야 하는 상당한 연구개발(R&D) 비용이 발생한다는 점을 반드시 고려해야 한다.

- 개발 속도 및 안정성 중시 시: Bosch BMI088

  본 보고서에서 가장 강력하게 추천하는 모델이다. 원시 데이터 성능은 ICM-42688-P에 비해 다소 낮지만, 드론 및 로보틱스 애플리케이션을 위해 특별히 설계되어 높은 진동 내성과 뛰어난 온도 안정성을 갖추고 있다.21 그러나 이 센서의 가장 큰 장점은 소프트웨어 생태계에 있다. 

  **NVIDIA Isaac ROS에서 공식적으로 `isaac_ros_imu_bmi088`이라는 이름의 최적화된 ROS2 Humble 드라이버를 제공한다**.27 이는 Jetson Orin 플랫폼과의 완벽한 호환성, 지속적인 유지보수, 그리고 검증된 안정성을 보장한다. 이는 개발 시간을 몇 주에서 몇 달까지 단축시킬 수 있는 막대한 이점으로, 대부분의 상업 및 학술 프로젝트에서 가장 합리적이고 효율적인 선택이 될 것이다.

이러한 분석은 '절대적으로 가장 좋은 센서'는 없으며, '프로젝트의 목표와 제약 조건에 가장 적합한 센서'가 존재할 뿐이라는 중요한 사실을 시사한다. 하드웨어의 원시 성능과 그것을 지원하는 소프트웨어 생태계 사이에는 근본적인 트레이드오프 관계가 존재한다. 이 보고서는 개발 속도와 안정성을 우선시하는 관점에서 BMI088을 기준으로 후속 통합 가이드를 진행하지만, ICM-42688-P가 제공하는 극한의 성능 잠재력 또한 명확히 인지하고, 개발팀의 역량과 프로젝트의 우선순위에 따라 현명한 선택을 내릴 것을 권장한다.

## 3.  Jetson Orin 하드웨어 인터페이스 및 설정

선정된 IMU를 Jetson Orin에 성공적으로 통합하기 위한 첫 단계는 하드웨어 인터페이스를 올바르게 구성하고 물리적으로 연결하는 것이다. 고주파 데이터 전송을 위해 SPI(Serial Peripheral Interface) 통신 방식을 사용하는 것을 기준으로 설명한다.

### 3.1  Jetson Orin 40핀 확장 헤더 분석

NVIDIA Jetson Orin Developer Kit는 라즈베리 파이와 유사한 40핀 확장 헤더를 제공하여 다양한 외부 센서 및 주변 장치와의 연결을 지원한다. AGX Orin 모델의 경우 J30, Orin Nano 모델의 경우 J12 헤더가 이에 해당한다.32 고주파 IMU와의 통신에는 I2C(Inter-Integrated Circuit)보다 훨씬 높은 데이터 전송률을 제공하는 SPI 인터페이스 사용이 권장된다. 40핀 헤더에서 SPI1 버스에 할당된 핀은 다음과 같다.

**Table 2: Jetson Orin과 BMI088 간 SPI 연결 핀맵**

| Jetson Orin 핀 (J30/J12) | 신호 (Signal) | BMI088 핀                 | 설명 (Description)               |
| ------------------------ | ------------- | ------------------------- | -------------------------------- |
| 17                       | 3.3V          | VDD, VDDIO                | 센서 전원 공급 (3.3V)            |
| 19                       | SPI1_MOSI     | SDI/SDA                   | Master Out, Slave In 데이터 라인 |
| 21                       | SPI1_MISO     | SDO1 (Accel), SDO2 (Gyro) | Master In, Slave Out 데이터 라인 |
| 23                       | SPI1_SCK      | SCx                       | 직렬 클럭 라인                   |
| 24                       | SPI1_CS0_N    | CSB1 (Accel)              | 칩 선택 (가속도계)               |
| 26                       | SPI1_CS1_N    | CSB2 (Gyro)               | 칩 선택 (자이로스코프)           |
| 25                       | GND           | GND                       | 공통 접지                        |

BMI088은 가속도계와 자이로스코프가 내부적으로 별개의 칩으로 구성되어 있어, 각각을 제어하기 위한 두 개의 칩 선택(Chip Select, CS) 라인이 필요하다. Jetson Orin의 SPI1 버스는 기본적으로 두 개의 CS 라인(CS0, CS1)을 지원하므로 BMI088과 완벽하게 호환된다.21

### 3.2  고속 데이터 전송을 위한 SPI 인터페이스 활성화

Jetson Orin의 40핀 헤더 핀들은 기본적으로 대부분 범용 입출력(GPIO)으로 설정되어 있다. 따라서 SPI 통신을 사용하기 위해서는 해당 핀들의 기능을 SPI로 변경하는 과정이 필요하다. 과거 Jetson Nano에서는 디바이스 트리 블롭(DTB) 파일을 수동으로 수정하고 커널을 재컴파일하는 복잡한 과정이 필요했지만 35, Jetson Orin에서는 `jetson-io`라는 편리한 유틸리티를 통해 이 과정을 안전하고 간단하게 수행할 수 있다.36

SPI 활성화 절차는 다음과 같다.

1. **Jetson-IO 실행:** Jetson Orin의 터미널을 열고 다음 명령어를 입력하여 `jetson-io` 유틸리티를 실행한다. 터미널 창은 전체 화면으로 설정하는 것이 메뉴를 확인하기에 용이하다.

   ```Bash
   sudo /opt/nvidia/jetson-io/jetson-io.py
   ```

2. **헤더 설정 메뉴 진입:** 실행된 화면에서 키보드 방향키를 사용하여 'Configure 40-pin header' 옵션을 선택하고 엔터를 누른다.

3. **SPI1 활성화:** 새로운 화면에 40핀 헤더에 설정 가능한 기능 목록이 나타난다. 여기서 'spi1' (또는 사용하고자 하는 다른 SPI 버스) 항목으로 이동하여 스페이스바를 눌러 선택(*)한다.

4. **설정 저장 및 재부팅:** 'Back'을 눌러 메인 메뉴로 돌아온 뒤, 'Save and reboot to apply changes'를 선택하여 변경 사항을 저장하고 시스템을 재부팅한다.

재부팅이 완료되면 19, 21, 23, 24, 26번 핀은 SPI1 버스의 전용 핀으로 기능하게 된다.

### 3.3  IMU 물리적 연결

하드웨어 설정이 완료되었으면, 이제 IMU 센서를 Jetson Orin에 물리적으로 연결한다. 모든 작업은 반드시 Jetson Orin의 전원이 완전히 차단된 상태에서 진행해야 한다.

1. **연결 준비:** BMI088 센서가 장착된 브레이크아웃 보드와 6-7개의 암-수 점퍼 와이어를 준비한다.

2. **핀 연결:** Table 2의 핀맵을 참조하여 점퍼 와이어를 사용하여 BMI088 보드의 각 핀을 Jetson Orin의 40핀 헤더에 정확하게 연결한다. 핀 번호는 헤더 근처 PCB에 인쇄된 표시를 기준으로 확인한다.

3. **연결 확인:** 모든 연결이 올바르게 되었는지 다시 한번 확인한다. 특히 VDD(3.3V)와 GND(접지)가 바뀌지 않도록 주의한다.

4. **SPI 장치 확인:** 연결을 마친 후 Jetson Orin의 전원을 켠다. 부팅이 완료되면 터미널을 열고 다음 명령어를 실행한다.

   ```Bash
   ls /dev/spidev*
   ```

   만약 `/dev/spidev0.0` 와 `/dev/spidev0.1` 과 같은 장치 파일이 목록에 나타난다면, SPI 버스가 커널 레벨에서 성공적으로 활성화되고 인식되었음을 의미한다. 이로써 하드웨어 통합 단계가 완료된다.

## 4.  ROS2 Humble 통합 및 상태 추정 파이프라인 구축

하드웨어 연결이 완료된 후, ROS2 Humble 환경에서 IMU 데이터를 수신하고 이를 EKF와 융합하여 최종적인 상태 추정 파이프라인을 구축하는 소프트웨어 통합 과정을 진행한다.

### 4.1  ROS2 개발 환경 구축

본 가이드는 Jetson Orin에 NVIDIA JetPack 6.x 버전(Ubuntu 22.04 기반)이 설치된 것을 전제로 한다. 이 환경에서 ROS2 Humble Hawksbill을 설치하는 절차는 다음과 같다.

1. **ROS2 APT 저장소 설정:** ROS2 패키지를 다운로드할 수 있도록 공식 APT 저장소를 시스템에 추가한다.39

   ```Bash
   sudo apt install software-properties-common
   sudo add-apt-repository universe
   sudo apt update && sudo apt install curl -y
   sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
   ```

2. **ROS2 Humble 설치:** `ros-humble-desktop` 또는 필요한 최소 기능만 설치하는 `ros-humble-ros-base`를 설치한다. 개발 환경에서는 일반적으로 데스크톱 버전을 설치하는 것이 편리하다.41

   ```Bash
   sudo apt update
   sudo apt install ros-humble-desktop
   ```

3. **개발 도구 설치 및 환경 설정:** `colcon` 빌드 도구와 `rosdep` 의존성 관리 도구를 설치한다.39

   ```Bash
   sudo apt install ros-dev-tools
   sudo rosdep init
   rosdep update
   ```

4. **환경 변수 설정:** 새로운 터미널을 열 때마다 ROS2 환경이 자동으로 설정되도록 `setup.bash` 파일을 `.bashrc`에 추가한다.42

   ```Bash
   echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
   source ~/.bashrc
   ```

   이제 터미널에서 `ros2` 명령어를 사용할 수 있다.

### 4.2  IMU ROS2 드라이버 설치 및 검증

3.3절에서 선정한 Bosch BMI088은 NVIDIA Isaac ROS에서 공식 드라이버를 제공하므로, 설치 및 사용이 매우 간편하다.

1. **Isaac ROS APT 저장소 추가:** NVIDIA에서 제공하는 Isaac ROS용 APT 저장소를 시스템에 추가한다. 이를 통해 사전 빌드된 최적화된 ROS2 패키지를 손쉽게 설치할 수 있다.41

   ```Bash
   sudo apt-key adv --fetch-keys https://isaac.download.nvidia.com/isaac-ros/repos.key
   echo 'deb https://isaac.download.nvidia.com/isaac-ros/ubuntu/main jammy main' | sudo tee -a /etc/apt/sources.list.d/isaac-ros.list
   sudo apt update
   ```

2. **BMI088 드라이버 패키지 설치:** `apt`를 사용하여 `ros-humble-isaac-ros-imu-bmi088` 패키지를 설치한다.27

   ```Bash
   sudo apt-get install ros-humble-isaac-ros-imu-bmi088
   ```

3. **드라이버 실행 및 데이터 검증:** 설치가 완료되면, 제공된 런치 파일을 사용하여 IMU 드라이버 노드를 실행한다.27

   ```Bash
   ros2 launch isaac_ros_imu_bmi088 bmi088.launch.py
   ```

   노드가 성공적으로 실행되면, 새로운 터미널을 열고 `ros2 topic echo` 명령을 사용하여 `/imu` 토픽에 `sensor_msgs/msg/Imu` 타입의 메시지가 발행되는지 확인한다.

   ```Bash
   ros2 topic echo /imu
   ```

   출력되는 메시지에서 `angular_velocity`와 `linear_acceleration` 필드에 데이터가 지속적으로 업데이트된다면, IMU 센서와 ROS2 시스템 간의 통신이 성공적으로 이루어진 것이다.

### 4.3  `robot_localization` 패키지를 이용한 EKF 설정

`robot_localization`은 IMU, Odometry, GPS 등 다양한 센서 데이터를 융합하여 로봇의 상태를 추정하는 EKF 및 UKF(Unscented Kalman Filter)를 제공하는 ROS의 표준 패키지이다.

1. **패키지 설치:** `apt`를 통해 `robot_localization` 패키지를 설치한다.

   ```Bash
   sudo apt install ros-humble-robot-localization
   ```

2. **EKF 설정 파일 작성:** `ekf_node`의 동작을 정의하는 YAML 설정 파일(`ekf.yaml`)을 작성한다. 이 파일은 어떤 센서의 어떤 데이터를 융합할지, 좌표계는 어떻게 설정할지 등을 정의하는 핵심적인 부분이다.44

   ```YAML
   ekf_filter_node:
       ros__parameters:
           frequency: 200.0
           sensor_timeout: 0.1
           two_d_mode: false
           publish_tf: true
           map_frame: map
           odom_frame: odom
           base_link_frame: base_link
           world_frame: odom
   
           imu0: /imu
           imu0_config: # dddX, dddY, dddZ
           imu0_differential: false
           imu0_relative: false
           imu0_remove_gravitational_acceleration: true
   ```

   - `frequency`: 필터의 업데이트 주기. IMU의 ODR보다는 낮게, 제어 주기보다는 높게 설정하는 것이 일반적이다.
   - `two_d_mode`: 3D 공간을 비행하는 드론이므로 `false`로 설정한다.
   - `imu0`: 융합할 IMU 데이터가 발행되는 토픽 이름.
   - `imu0_config`: 15개의 boolean 값으로 구성된 행렬로, EKF에 어떤 IMU 데이터를 융합할지 결정하는 가장 중요한 매개변수이다.
   - `imu0_remove_gravitational_acceleration`: `true`로 설정하여 가속도계 측정값에서 중력 가속도 성분을 제거하고 순수한 선형 가속도만 융합하도록 한다.

**Table 3: `imu0_config` 매개변수 설정 가이드**

| 변수 (Variable)                      | 위치 (Position) | 융합 여부 (Fuse?) | 설명 (Description)                                           |
| ------------------------------------ | --------------- | ----------------- | ------------------------------------------------------------ |
| Roll, Pitch, Yaw                     | 1-3             | No                | IMU의 절대 자세는 드리프트가 심하므로, 자력계나 외부 센서(GPS Compass 등) 없이는 융합하지 않는 것이 일반적이다. |
| $\dot{Roll}, \dot{Pitch}, \dot{Yaw}$ | 4-6             | **Yes**           | 자이로스코프에서 직접 측정되는 각속도. IMU가 제공하는 가장 신뢰성 높은 정보이므로 반드시 `true`로 설정하여 융합한다. |
| $\dot{X}, \dot{Y}, \dot{Z}$          | 7-9             | **Yes**           | 가속도계에서 측정되는 선형 가속도. `remove_gravitational_acceleration` 옵션과 함께 사용하여 기체의 동적인 움직임을 융합한다. |
| $X, Y, Z$                            | 10-12           | No                | IMU만으로는 절대 위치를 알 수 없으므로 `false`로 설정한다. 위치 정보는 GPS나 비전 센서를 통해 융합해야 한다. |
| $\ddot{X}, \ddot{Y}, \ddot{Z}$       | 13-15           | No                | 속도의 변화율(가속도)은 이미 7-9번 위치에서 사용되었으므로 중복으로 융합하지 않는다. |

### 4.4  전체 시스템 실행 및 시각화

마지막으로, IMU 드라이버와 EKF 노드를 함께 실행하고 그 결과를 시각화하여 전체 상태 추정 파이프라인이 정상적으로 동작하는지 검증한다.

1. **런치 파일 작성:** 두 노드를 한 번에 실행하기 위한 ROS2 런치 파일(`state_estimation.launch.py`)을 작성한다.

   ```Python
   from launch import LaunchDescription
   from launch_ros.actions import Node
   from ament_index_python.packages import get_package_share_directory
   import os
   
   def generate_launch_description():
       pkg_share = get_package_share_directory('your_package_name')
       ekf_config_path = os.path.join(pkg_share, 'config/ekf.yaml')
   
       return LaunchDescription([
           # Launch the IMU driver
           Node(
               package='isaac_ros_imu_bmi088',
               executable='isaac_ros_imu_bmi088_node',
               name='imu_node',
               output='screen'
           ),
           # Launch the EKF node
           Node(
               package='robot_localization',
               executable='ekf_node',
               name='ekf_filter_node',
               output='screen',
               parameters=[ekf_config_path]
           )
       ])
   ```

2. **시스템 실행:** 작성한 런치 파일을 빌드하고 실행한다.

   ```Bash
   ros2 launch your_package_name state_estimation.launch.py
   ```

3. **RViz2 시각화 및 검증:** RViz2를 실행하여 상태 추정 결과를 시각적으로 확인한다.

   ```Bash
   rviz2
   ```

   - **TF 확인:** RViz2 좌측 하단의 'Add' 버튼을 클릭하여 'TF' 디스플레이를 추가한다. `odom` 프레임과 `base_link` 프레임이 나타나고, IMU를 움직였을 때 `base_link`가 부드럽게 따라 움직인다면 `ekf_node`가 TF를 정상적으로 발행하고 있는 것이다.
   - **Odometry 확인:** 'Add' 버튼을 통해 'Odometry' 디스플레이를 추가하고, Topic을 `/odometry/filtered`로 설정한다. 기체를 움직일 때 해당 위치와 방향을 나타내는 화살표가 TF와 일치하여 표시되면, EKF가 추정한 최종 상태가 정상적으로 발행되고 있음을 의미한다.

이 과정을 통해 Jetson Orin 기반 드론의 고주파 IMU 데이터가 ROS2 Humble 환경에서 성공적으로 EKF와 융합되어 안정적인 상태 추정 결과를 출력하는 전체 파이프라인이 완성된다.

## 5.  결론

### 6.1. 핵심 요약

본 보고서는 NVIDIA Jetson Orin 플랫폼을 기반으로 하는 고성능 드론의 핵심 기술인 정밀 상태 추정 시스템 구축을 위한 포괄적인 가이드를 제시했다. 이를 위해, 먼저 관성 항법의 근본적인 오차 누적 문제와 이를 극복하기 위한 고주파 IMU의 이론적 중요성을 에일리어싱, 위상 지연, 확장 칼만 필터의 관점에서 심도 있게 분석했다. 다음으로, 바이어스 안정성, 노이즈 밀도, ODR 등 핵심 성능 지표를 기준으로 실제 시장의 유력 IMU 모델(TDK ICM-42688-P, Bosch BMI088 등)을 정량적으로 비교하여, 프로젝트 목표에 따른 최적의 센서 선정 전략을 제안했다. 마지막으로, 선정된 IMU(Bosch BMI088)를 Jetson Orin에 물리적으로 연결하고 SPI 인터페이스를 활성화하는 하드웨어 통합 절차와, ROS2 Humble 환경에서 NVIDIA Isaac ROS 드라이버 및 `robot_localization` 패키지를 사용하여 EKF 기반의 완전한 상태 추정 파이프라인을 구축하고 검증하는 단계별 소프트웨어 통합 가이드를 제공했다.

### 5.1  최종 권장 사항

본 보고서의 분석과 실험 과정을 통해 도출된 핵심 권장 사항은 다음과 같다.

- **이론과 실제의 균형:** 성공적인 상태 추정 시스템은 단순히 고가의 센서를 사용하는 것만으로 이루어지지 않는다. 에일리어싱, 위상 지연과 같은 제어 이론에 대한 깊은 이해가 올바른 하드웨어(고주파 IMU) 선택의 근거가 되며, 이는 시스템 전체의 안정성을 보장하는 첫걸음이다.
- **성능과 생태계의 트레이드오프 인지:** 최고의 원시 성능을 가진 센서(예: TDK ICM-42688-P)와 최고의 소프트웨어 생태계 지원을 받는 센서(예: Bosch BMI088) 사이에는 명백한 트레이드오프가 존재한다. 프로젝트의 기간, 예산, 개발팀의 역량을 종합적으로 고려하여 '가장 좋은 센서'가 아닌 '가장 적합한 센서'를 선택하는 전략적 판단이 필수적이다. 대부분의 경우, 검증된 공식 드라이버가 제공하는 개발 효율성과 안정성은 약간의 하드웨어 성능 차이를 상쇄하고도 남는다.
- **점진적 융합 및 튜닝:** EKF와 같은 복잡한 필터를 설정할 때는 점진적인 접근법이 매우 유효하다. 처음에는 가장 신뢰도 높은 데이터(IMU의 각속도 및 선형 가속도)만으로 시스템을 구성하여 안정성을 확보한 뒤, GPS, 비전 등 다른 센서를 하나씩 추가하며 파라미터를 튜닝하는 방식이 디버깅을 용이하게 하고 전체 시스템의 강인성을 높이는 지름길이다.

### 5.2  미래 전망

본 보고서에서 다룬 IMU 기반의 EKF 상태 추정은 모든 자율 비행 시스템의 근간을 이룬다. Jetson Orin의 강력한 병렬 처리 능력을 활용하면, 여기서 한 걸음 더 나아가 IMU 데이터와 카메라의 시각 정보를 결합하는 시각-관성 주행 거리 측정(Visual-Inertial Odometry, VIO) 또는 LiDAR의 공간 정보와 결합하는 LiDAR-관성 주행 거리 측정(LiDAR-Inertial Odometry, LIO) 기술로 확장할 수 있다. 이러한 진보된 센서 융합 기술은 GPS가 완전히 사용 불가능한 실내 환경이나 도심 협곡에서도 강인하고 정밀한 항법을 가능하게 하여, 미래 드론 애플리케이션의 가능성을 무한히 확장시킬 것이다. 본 보고서에서 제시한 고주파 IMU의 선정 및 통합 방법론은 이러한 차세대 상태 추정 기술을 구현하기 위한 견고한 초석이 될 것이다.

#### 참고 자료

1. Why is IMU the core of drone stability control? - Ericco Inertial ..., accessed August 17, 2025, https://www.ericcointernational.com/info/why-is-imu-the-core-of-drone-stability-control.html
2. How to Choose the Right IMU for UAVs? - GuideNav: The Global Standard in Inertial Navigation Excellence, accessed August 17, 2025, https://guidenav.com/how-to-choose-the-right-imu-for-uavs/
3. How to Select the Right IMU for UAV Applications | Unmanned Systems Technology, accessed August 17, 2025, https://www.unmannedsystemstechnology.com/feature/how-to-select-the-right-imu-for-uav-applications/
4. Airspeed-Aided State Estimation Algorithm of Small Fixed-Wing UAVs in GNSS-Denied Environments - MDPI, accessed August 17, 2025, https://www.mdpi.com/1424-8220/22/9/3156
5. A Complete Guide to Inertial Measurement Unit (IMU) - JOUAV, accessed August 17, 2025, https://www.jouav.com/blog/inertial-measurement-unit.html
6. IMU - Inertial Measurement Unit - SBG Systems, accessed August 17, 2025, https://www.sbg-systems.com/glossary/inertial-measurement-unit-imu-sensor/
7. MARS: Defending Unmanned Aerial Vehicles From Attacks on Inertial Sensors with Model-based Anomaly Detection and Recovery - arXiv, accessed August 17, 2025, https://arxiv.org/html/2505.00924v1
8. Inertial measurement unit - Wikipedia, accessed August 17, 2025, https://en.wikipedia.org/wiki/Inertial_measurement_unit
9. IMU Error Modeling Tutorial: INS state estimation with real-time sensor calibration - SciSpace, accessed August 17, 2025, https://scispace.com/pdf/imu-error-state-modeling-for-state-estimation-and-sensor-4pefuknce3.pdf
10. Inertial Measurement Unit (IMU) | An Introduction, accessed August 17, 2025, https://www.advancednavigation.com/tech-articles/inertial-measurement-unit-imu-an-introduction/
11. Quadcopter State Estimation : r/AerospaceEngineering - Reddit, accessed August 17, 2025, https://www.reddit.com/r/AerospaceEngineering/comments/175747t/quadcopter_state_estimation/
12. Position Estimation Method for Small Drones Based on the Fusion of Multisource, Multimodal Data and Digital Twins - MDPI, accessed August 17, 2025, https://www.mdpi.com/2079-9292/13/11/2218
13. How to Choose the Right Inertial Measurement Unit for UAVs ..., accessed August 17, 2025, https://bavovna.ai/inertial-measurement-unit/
14. IMU Noise Model / ethz-asl/kalibr Wiki - GitHub, accessed August 17, 2025, https://github.com/ethz-asl/kalibr/wiki/IMU-Noise-Model
15. How to Choose the Right IMU for UAVs - Embention, accessed August 17, 2025, https://www.embention.com/news/the-right-imu-for-an-uav/
16. Using PX4's Navigation Filter (EKF2) | PX4 Guide (main), accessed August 17, 2025, https://docs.px4.io/main/en/advanced_config/tuning_the_ecl_ekf
17. Extended Kalman Filter - AHRS 0.4.0 documentation, accessed August 17, 2025, https://ahrs.readthedocs.io/en/latest/filters/ekf.html
18. Extended Kalman Filter Navigation Overview and Tuning - Dev documentation - ArduPilot, accessed August 17, 2025, https://ardupilot.org/dev/docs/extended-kalman-filter.html
19. MEMS IMU for FPV: The Ultimate Precision Flight Control Solution for Drones - GuideNav, accessed August 17, 2025, https://guidenav.com/mems-imu-for-fpv-the-perfect-solution-for-precision-flight-control/
20. IMUs and LiDARs - Not Uncommon Pitfalls, accessed August 17, 2025, https://msadowski.github.io/basic-sensors-for-mobile-robots/
21. BMI088 High-performance Inertial Measurement Unit (IMU) - Mouser Electronics, accessed August 17, 2025, https://www.mouser.com/datasheet/2/783/Bosch_Sensortec_Product_flyer_BMI088-1358231.pdf
22. ICM-42688-P Datasheet - TDK Product Center, accessed August 17, 2025, https://product.tdk.com/system/files/dam/doc/product/sensor/mortion-inertial/imu/data_sheet/ds-000347-icm-42688-p-v1.6.pdf
23. Inertial Measurement Unit BMI088 - Bosch Sensortec, accessed August 17, 2025, https://www.bosch-sensortec.com/products/motion-sensors/imus/bmi088/
24. How to Use Adafruit BNO085 9-DOF Orientation IMU Fusion: Examples, Pinouts, and Specs, accessed August 17, 2025, https://docs.cirkitdesigner.com/component/c5efd92e-381f-a894-e2c8-7da4f19fb0f8/adafruit-bno085-9-dof-orientation-imu-fusion
25. Adafruit 9-DOF Orientation IMU Fusion Breakout - BNO085 (BNO080) - STEMMA QT / Qwiic, accessed August 17, 2025, https://www.kiwi-electronics.com/en/adafruit-9-dof-orientation-imu-fusion-breakout-bno085-bno080-stemma-qt-qwiic-11273
26. Arduino library for communicating with the InvenSense ICM42688 nine-axis IMUs. - GitHub, accessed August 17, 2025, https://github.com/finani/ICM42688
27. isaac_ros_imu_bmi088 - isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 17, 2025, https://nvidia-isaac-ros.github.io/repositories_and_packages/isaac_ros_nova/isaac_ros_imu_bmi088/index.html
28. robert-stevenson-1/BNO085-ROS2-Node - GitHub, accessed August 17, 2025, https://github.com/robert-stevenson-1/BNO085-ROS2-Node
29. DK-42688-P TDK InvenSense | Development Boards, Kits, Programmers | DigiKey, accessed August 17, 2025, https://www.digikey.com/en/products/detail/tdk-invensense/DK-42688-P/11681011
30. BMI088 Bosch Sensortec - Mouser Electronics, accessed August 17, 2025, https://www.mouser.com/ProductDetail/Bosch-Sensortec/BMI088?qs=f9yNj16SXrIMFspTV6RB6Q%3D%3D
31. Search results for: BNO085 - Mouser Electronics, accessed August 17, 2025, https://www.mouser.com/c/?q=BNO085
32. NVIDIA Jetson AGX Orin GPIO Header Pinout - JetsonHacks, accessed August 17, 2025, https://jetsonhacks.com/nvidia-jetson-agx-orin-gpio-header-pinout/
33. NVIDIA Jetson Orin Nano GPIO Header Pinout - JetsonHacks, accessed August 17, 2025, https://jetsonhacks.com/nvidia-jetson-orin-nano-gpio-header-pinout/
34. JETGPIO/README.md at main - GitHub, accessed August 17, 2025, https://github.com/Rubberazer/JETGPIO/blob/main/README.md
35. Enable SPI interface on Jetson-Nano | Seeed Studio Wiki, accessed August 17, 2025, https://wiki.seeedstudio.com/enable_spi_interface_on_jetsonnano/
36. README.md - JetsonHacksNano/SPI-Playground - GitHub, accessed August 17, 2025, https://github.com/JetsonHacksNano/SPI-Playground/blob/master/README.md
37. How to set gpio for spi? - Jetson AGX Orin - NVIDIA Developer Forums, accessed August 17, 2025, https://forums.developer.nvidia.com/t/how-to-set-gpio-for-spi/238990
38. Connecting SPI Display to GPIO JETSON ORIN : r/JetsonNano - Reddit, accessed August 17, 2025, https://www.reddit.com/r/JetsonNano/comments/1j0lf4g/connecting_spi_display_to_gpio_jetson_orin/
39. How to install ROS 2 Humble Hawksbill - Forecr.io, accessed August 17, 2025, https://www.forecr.io/blogs/installation/how-to-install-ros-2-humble-hawksbill
40. Install the ROS2 Humble | Seeed Studio Wiki, accessed August 17, 2025, https://wiki.seeedstudio.com/install_ros2_humble/
41. Installing ROS2 on Jetson Xavier - Isaac ROS - NVIDIA Developer Forums, accessed August 17, 2025, https://forums.developer.nvidia.com/t/installing-ros2-on-jetson-xavier/289580
42. Configuring environment - ROS 2 Documentation: Humble documentation, accessed August 17, 2025, https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Configuring-ROS2-Environment.html
43. Getting Started - isaac_ros_docs documentation - NVIDIA Isaac ROS, accessed August 17, 2025, https://nvidia-isaac-ros.github.io/getting_started/index.html
44. Sensor Fusion and Robot Localization Using ROS 2 Jazzy - YouTube, accessed August 17, 2025, https://www.youtube.com/watch?v=XOQTF38lmtE
45. Configuring robot_localization - ROS documentation, accessed August 17, 2025, http://docs.ros.org/en/melodic/api/robot_localization/html/configuring_robot_localization.html
46. Sharpen Your Robot's Eyes – Mastering ROS2 Sensor Fusion with EKF & AMCL - Robotisim, accessed August 17, 2025, https://robotisim.com/ros2-sensor-fusion-ekf-amcl/

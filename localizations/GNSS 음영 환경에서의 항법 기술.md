# GNSS 음영 환경에서의 항법 기술
[위치 추정 (Localization)](./index.md)


현대 문명은 위성항법시스템(Global Navigation Satellite System, GNSS)이 제공하는 위치(Positioning), 항법(Navigation), 시각(Timing) 정보, 즉 PNT에 깊이 의존하고 있다. 자율주행차, 드론, 정밀 농업부터 금융 거래, 통신망 동기화에 이르기까지, GNSS는 4차 산업혁명의 기반 기술로서 사회 전반에 걸쳐 핵심적인 역할을 수행한다.1 GNSS는 지구 궤도를 도는 다수의 위성에서 송신하는 신호를 수신하여, 신호의 도달 시간(Time of Arrival) 차이를 기반으로 삼변측량(Trilateration) 원리를 이용해 수신기의 위치를 계산한다.

그러나 이 위성 신호는 약 20,000 km 상공에서부터 지상에 도달하기 때문에 매우 미약하며, 다양한 요인에 의해 신호의 수신이 방해받거나 정보가 왜곡될 수 있다.2 이러한 문제로 인해 GNSS의 사용이 불가능하거나 신뢰도가 현저히 떨어지는 'GNSS 음영(denied)' 또는 'GNSS 열화(degraded)' 환경이 발생한다. 이 두 환경은 기술적 대응 관점에서 명확히 구분될 필요가 있다. 신호가 전혀 도달하지 않는 터널이나 실내와 같은 '음영' 환경에서는 관성항법장치(INS)와 같은 추측 항법 기술이 주력이 되어야 하며, 외부 센서를 통한 보정이 필수적이다. 반면, 신호는 존재하지만 오염되거나 신뢰도가 낮은 도심 협곡과 같은 '열화' 환경에서는 오염된 신호를 식별하고 배제하며, 제한된 수의 위성 신호라도 최대한 활용하는 고도의 센서 융합 알고리즘이 요구된다. 이러한 구분은 항법 시스템의 설계 철학에 직접적인 영향을 미치며, 본 자습서에서는 각 시나리오에 최적화된 기술적 접근법을 제시하고자 한다.

GNSS 음영 및 열화 환경을 유발하는 원인은 복합적이다.

- **물리적 차단:** 고층 빌딩이 밀집한 도심 협곡(Urban Canyon), 실내, 터널, 수중, 울창한 숲 등은 위성 신호를 물리적으로 차단하여 수신을 불가능하게 만든다.5
- **신호 간섭 및 다중 경로:** 신호가 건물이나 지형지물에 반사되어 여러 경로를 통해 수신기에 도달하는 다중 경로(Multipath) 현상은 신호의 도달 시간을 왜곡시켜 심각한 위치 오차를 유발한다. 특히, 직접 신호(Line-of-Sight, LOS) 없이 반사된 신호(Non-LOS, NLOS)만 수신되는 경우, 위치 오차는 수백 미터에 달할 수 있다.4
- **의도적 공격 (Jamming & Spoofing):** 재밍(Jamming)은 GNSS 주파수 대역에 강력한 방해 전파를 송출하여 수신기를 마비시키는 행위이다. 스푸핑(Spoofing)은 더 나아가 위조된 위성 신호를 생성 및 송신하여 수신기가 잘못된 위치와 시각 정보를 계산하도록 기만하는 지능적인 공격이다. 이러한 전자전(Electronic Warfare) 위협은 군사 분야뿐만 아니라 자율주행차, 드론 등 민간 시스템의 안전에도 심각한 위협이 되고 있다.3
- **기타 요인:** 이 외에도 태양 흑점 폭발과 같은 자연 현상, 전리층과 대류층의 변화로 인한 대기 지연 오차, 위성 자체의 시계 및 궤도 오차 등도 GNSS 정확도를 저하시키는 요인으로 작용한다.11

결론적으로, GNSS에만 전적으로 의존하는 항법 시스템은 신뢰성과 강인성 측면에서 명백한 한계를 가진다. 따라서 GNSS 신호를 사용할 수 없거나 신뢰할 수 없는 상황에서도 독립적으로 정확한 위치, 속도, 자세 정보를 제공할 수 있는 자율 항법 기술의 확보는 현대 사회의 안전과 기술 발전을 위해 필수적인 과제가 되었다. 본 자습서는 이러한 GNSS 음영 환경에서의 항법 기술을 심도 있게 탐구하는 것을 목표로 한다.




관성 항법 시스템(Inertial Navigation System, INS)은 추측 항법(Dead Reckoning) 원리에 기반한 대표적인 자율 항법 장치이다. 추측 항법이란, 외부의 도움 없이 시스템의 초기 위치, 속도, 자세를 기준으로 자체 탑재된 센서로 측정한 이동체의 움직임 정보를 연속적으로 누적하여 현재 위치를 추정하는 방식이다.14 INS는 이 원리를 뉴턴의 운동 법칙과 관성 센서를 이용하여 구현한다.16


INS의 핵심 센서 모듈은 관성 측정 장치(Inertial Measurement Unit, IMU)로, 일반적으로 3축 가속도계와 3축 자이로스코프로 구성된다.15

- **가속도계(Accelerometer):** 가속도계는 뉴턴의 관성 법칙($F=ma$)에 기반하여 이동체의 비중력(specific force)을 측정한다. 비중력이란 중력을 제외하고 이동체에 가해지는 모든 외부 힘에 의한 가속도를 의미한다. 3축 가속도계는 서로 직교하는 세 축 방향의 선형 가속도를 측정한다. 이동체가 정지 상태에 있거나 등속 운동을 할 경우, 가속도계는 중력 가속도만을 감지하게 되는데, 이 정보를 이용하여 이동체의 기울어진 자세, 즉 피치(Pitch)와 롤(Roll) 각도를 계산할 수 있다.18
- **자이로스코프(Gyroscope):** 자이로스코프는 각운동량 보존 법칙이나 코리올리 효과(Coriolis effect)와 같은 물리적 원리를 이용하여 이동체의 각속도(회전율)를 측정하는 센서이다. 3축 자이로스코프는 동체의 세 축(Roll, Pitch, Yaw)을 기준으로 한 회전 각속도를 측정하여 이동체의 자세 변화를 감지한다.20


INS는 IMU로부터 얻은 측정값을 바탕으로 다음과 같은 적분 기반의 계산 과정을 거쳐 항법 정보를 산출한다.

1. **자세 계산:** 자이로스코프가 측정한 3축 각속도를 시간에 대해 적분하여 이동체의 자세(Attitude: Roll, Pitch, Yaw)를 계산한다.17
2. **속도 계산:** 계산된 자세 정보를 이용하여, 동체 좌표계(Body Frame)에서 측정된 가속도 값을 항법 좌표계(Navigation Frame)로 변환한다. 이 변환된 가속도 값에서 중력 가속도 성분을 제거한 후, 시간에 대해 적분하여 속도(Velocity)를 계산한다.14
3. **위치 계산:** 계산된 속도를 다시 한번 시간에 대해 적분하여 최종적으로 위치(Position)를 계산한다.24


INS는 외부 정보 없이 독립적으로 항법 정보를 제공할 수 있다는 큰 장점을 가지지만, 치명적인 단점 또한 내포하고 있다.14 바로 센서 자체의 미세한 오차(바이어스, 노이즈, 스케일 팩터 오차 등)가 적분 과정을 거치면서 시간에 따라 계속 누적되어 오차가 기하급수적으로 커지는 현상, 즉 드리프트(Drift)이다.16

특히 위치 정보는 가속도를 두 번 적분하여 얻어지므로 오차의 발산이 매우 심각하다.27 예를 들어, 일반적인 상용 MEMS 센서를 사용한 INS는 수 분만 작동해도 수백 미터의 위치 오차가 발생할 수 있으며, 고정밀 INS라 할지라도 10시간 정도 비행하면 오차 범위가 수 킬로미터에 달할 수 있다.18 이처럼 시간이 지남에 따라 항법 정보의 신뢰도가 급격히 저하되기 때문에, 순수한 INS 단독 항법은 장시간 운용이 불가능하다. 따라서 신뢰성 있는 항법 정보를 유지하기 위해서는 GNSS나 카메라 등 다른 외부 센서 정보를 활용하여 주기적으로 누적된 오차를 보정해주는 과정이 반드시 필요하다.26



관성 항법을 수학적으로 정확하게 기술하기 위해서는 여러 좌표계(Coordinate Frame)에 대한 이해가 필수적이다.

- **관성 좌표계 (Inertial Frame, i-frame):** 뉴턴의 운동 법칙이 성립하는 비가속, 비회전 좌표계. 일반적으로 지구 중심에 원점을 두고 우주 공간의 특정 별을 기준으로 축이 고정된 좌표계로 가정한다.
- **지구중심고정 좌표계 (Earth-Centered, Earth-Fixed, ECEF, e-frame):** 지구 중심에 원점을 두고 지구의 자전과 함께 회전하는 좌표계.
- **항법 좌표계 (Navigation Frame, n-frame):** 이동체의 위치를 기준으로 정의되는 지역 좌표계. 일반적으로 북-동-하늘(North-East-Down, NED) 좌표계가 널리 사용된다.
- **동체 좌표계 (Body Frame, b-frame):** 이동체에 고정된 좌표계. 일반적으로 전방-우측-하늘(Forward-Right-Down) 방향으로 축을 설정한다. IMU 센서는 이 동체 좌표계를 기준으로 가속도와 각속도를 측정한다.


과거에는 기계적인 짐벌(Gimbal)을 이용해 관성 센서 플랫폼을 항법 좌표계에 정렬시켰으나, 현대에는 센서를 동체에 직접 부착(Strap-down)하고 컴퓨터의 연산을 통해 좌표 변환을 수행하는 스트랩다운 방식이 주를 이룬다.25 스트랩다운 INS의 운동 방정식은 다음과 같이 유도된다.

- **자세 업데이트 (Attitude Update):** 동체 좌표계에서 항법 좌표계로의 좌표 변환을 나타내는 방향 코사인 행렬(Direction Cosine Matrix, DCM) $\mathbf{C}_b^n$의 시간에 대한 변화율은 다음과 같은 미분 방정식으로 표현된다.30
  $$
  \dot{\mathbf{C}}_b^n = \mathbf{C}_b^n [\boldsymbol{\omega}_{nb}^b \times]
  $$
  여기서 $\boldsymbol{\omega}_{nb}^b$는 항법 좌표계에 대한 동체 좌표계의 상대 각속도를 동체 좌표계에서 표현한 벡터이며, $[\mathbf{a} \times]$는 벡터 $\mathbf{a}$의 외적을 행렬 곱으로 표현한 반대칭 행렬(Skew-symmetric matrix)이다. 이 방정식을 수치 적분하여 매시간 자세를 갱신한다. 계산 효율성과 특이점(Singularity) 문제 때문에 실제 구현에서는 쿼터니언(Quaternion)을 사용하는 경우가 많다.30

- **속도 미분 방정식 (Velocity Differential Equation):** 항법 좌표계에서 관측되는 이동체의 속도 변화율($\dot{\mathbf{v}}^n$)은 동체 좌표계에서 가속도계가 측정한 비중력($\mathbf{f}^b$)을 항법 좌표계로 변환한 값에, 코리올리 효과와 지구 자전에 의한 구심 가속도($-(2\boldsymbol{\omega}_{ie}^n + \boldsymbol{\omega}_{en}^n) \times \mathbf{v}^n$), 그리고 중력 가속도($\mathbf{g}^n$)를 더한 값으로 나타낼 수 있다.31
  $$
  \dot{\mathbf{v}}^n = \mathbf{C}_b^n \mathbf{f}^b - (2\boldsymbol{\omega}_{ie}^n + \boldsymbol{\omega}_{en}^n) \times \mathbf{v}^n + \mathbf{g}^n
  $$
  여기서 $\boldsymbol{\omega}_{ie}^n$는 관성 좌표계에 대한 지구 좌표계의 각속도(지구 자전 속도)를, $\boldsymbol{\omega}_{en}^n$는 지구 좌표계에 대한 항법 좌표계의 각속도(이동체의 움직임에 의한 각속도)를 나타낸다.

- **위치 미분 방정식 (Position Differential Equation):** 이동체의 위도($\lambda_{lat}`), 경도($\lambda_{lon}`), 고도($h$)의 시간에 대한 변화율은 항법 좌표계에서의 속도 성분($v_N, v_E, v_D$)과 지구의 곡률 반경($R_{mr}, R_{tr}$)을 이용하여 다음과 같이 표현된다.31
  $$
  \dot{\lambda}_{lat} = \frac{v_N}{R_{mr} + h}
  $$

  $$
  \dot{\lambda}_{lon} = \frac{v_E}{(R_{tr} + h)\cos\lambda_{lat}}
  $$

  $$
  \dot{h} = -v_D
  $$


INS의 오차 전파 특성을 분석하고 칼만 필터를 통해 보정하기 위해서는, 실제 참값과 시스템이 계산한 값 사이의 오차에 대한 동역학 모델, 즉 오차 방정식이 필요하다. 이는 섭동(perturbation) 방법을 통해 유도할 수 있다.32

- **자세 오차 방정식:** 자세 오차를 작은 각도 오차 벡터 $\delta\boldsymbol{\Phi}$로 정의하면, 이 오차의 시간에 대한 변화율은 다음과 같은 선형 미분 방정식으로 근사화할 수 있다.
  $$
  \delta\dot{\boldsymbol{\Phi}} = -[\boldsymbol{\omega}_{in}^n \times] \delta\boldsymbol{\Phi} + \delta\boldsymbol{\omega}_{in}^n - \mathbf{C}_b^n \delta\boldsymbol{\omega}_{ib}^b
  $$
  이 방정식은 자이로스코프의 측정 오차($\delta\boldsymbol{\omega}_{ib}^b$)가 어떻게 자세 오차로 전파되는지를 보여준다.32

- **속도 오차 방정식:** 속도 오차 벡터 $\delta\mathbf{v}^n$의 동역학 모델은 다음과 같다.
  $$
  \delta\dot{\mathbf{v}}^n \approx [\mathbf{f}^n \times] \delta\boldsymbol{\Phi} - (2\boldsymbol{\omega}_{ie}^n + \boldsymbol{\omega}_{en}^n) \times \delta\mathbf{v}^n + \mathbf{v}^n \times (2\delta\boldsymbol{\omega}_{ie}^n + \delta\boldsymbol{\omega}_{en}^n) + \mathbf{C}_b^n \delta\mathbf{f}^b
  $$
  이 식은 자세 오차($\delta\boldsymbol{\Phi}$)가 속도 오차에 영향을 미치고(오차 커플링), 가속도계의 측정 오차($\delta\mathbf{f}^b$)가 속도 오차를 유발하는 과정을 설명한다.32

- **위치 오차 방정식:** 위치 오차는 속도 오차를 적분한 형태로 나타나며, 속도 오차와의 선형 관계로 모델링된다.32

이러한 오차 상태 방정식들은 INS의 오차를 상태 변수로 하는 선형 시스템을 구성하며, 이는 칼만 필터를 이용한 최적의 오차 추정 및 보정 알고리즘 설계의 수학적 기반이 된다.




현실 세계의 모든 센서 측정값과 시스템 모델에는 필연적으로 불확실성이 포함된다. 이러한 불확실성을 수학적으로 다루기 위해 확률론적 접근법이 사용된다. 상태 변수(위치, 속도 등)를 단일한 값이 아닌 확률 분포(Probability Distribution)로 표현함으로써, 추정치가 얼마나 신뢰할 수 있는지를 정량적으로 나타낼 수 있다. 많은 경우, 계산의 편의성과 중심 극한 정리(Central Limit Theorem)에 근거하여 불확실성을 평균(mean)과 공분산(covariance)으로 표현되는 가우시안 분포(Gaussian Distribution)로 가정한다.


베이즈 필터는 베이즈 정리에 기반하여 상태를 추정하는 모든 확률적 필터의 이론적 근간을 이룬다. 베이즈 정리는 새로운 관측 정보가 주어졌을 때, 기존의 믿음(사전 확률)을 어떻게 갱신하여 새로운 믿음(사후 확률)을 형성해야 하는지에 대한 합리적인 추론 규칙을 제공한다.
$$
$P(x|z) = \frac{P(z|x)P(x)}{P(z)}$
$$

- $P(x)$: **사전 확률(Prior)** - 측정 이전에 가지고 있던 상태 $x$에 대한 믿음(확률 분포).
- $P(z|x)$: **우도(Likelihood)** - 실제 상태가 $x$일 때 측정값 $z$가 관측될 확률. 측정 모델에 해당한다.
- $P(x|z)$: **사후 확률(Posterior)** - 측정값 $z$를 관측한 후 갱신된 상태 $x$에 대한 믿음. 이것이 필터가 구하고자 하는 최종 결과이다.
- $P(z)$: 정규화 상수로, 사후 확률의 총합이 1이 되도록 만든다.


베이즈 필터는 새로운 측정값이 들어올 때마다 이전 단계의 사후 확률을 현재 단계의 사전 확률로 사용하여 상태를 순차적으로 갱신하는 재귀적 구조를 가진다. 이 과정은 크게 예측(Prediction)과 업데이트(Update) 두 단계로 나뉜다.

- **예측(Prediction) 단계:** 이 단계에서는 시스템의 동역학 모델을 사용하여 이전 시간($k-1$)의 상태 추정치로부터 현재 시간($k$)의 상태를 예측한다.
  $$
  p(x_k | z_{1:k-1}) = \int p(x_k | x_{k-1}) p(x_{k-1} | z_{1:k-1}) dx_{k-1}
  $$
  이 과정에서 시스템 자체의 불확실성(프로세스 노이즈)으로 인해 예측된 상태의 불확실성(공분산)은 일반적으로 증가한다.

- **업데이트(Update) 단계:** 예측 단계에서 구한 사전 확률 분포에 새로운 측정값($z_k$)의 정보를 베이즈 정리를 이용해 반영하여 최종적인 사후 확률 분포를 계산한다.
  $$
  p(x_k | z_{1:k}) = \frac{p(z_k | x_k) p(x_k | z_{1:k-1})}{p(z_k | z_{1:k-1})}
  $$
  이 단계에서는 새로운 측정 정보를 통해 불확실성이 감소하며, 예측된 상태가 실제 측정값에 가깝게 보정된다.

이러한 예측-업데이트 사이클을 반복함으로써, 베이즈 필터는 시간에 따라 변하는 시스템의 상태를 지속적으로 추적하고 추정한다.


칼만 필터와 그 변형들은 베이즈 필터의 재귀적 추정 프레임워크를 구체적인 수학적 형태로 구현한 알고리즘들이다. 어떤 형태의 칼만 필터를 사용할지는 대상 시스템의 선형성(linearity)과 노이즈의 분포 특성에 따라 결정된다.


칼만 필터는 시스템의 동역학 모델과 측정 모델이 모두 선형 방정식으로 표현되고, 모든 노이즈(프로세스 노이즈, 측정 노이즈)가 평균이 0인 가우시안 분포를 따른다고 가정할 때, 평균 제곱 오차(Mean Squared Error)를 최소화하는 최적의(optimal) 재귀 필터이다.33 이 필터는 상태의 확률 분포를 평균 벡터와 공분산 행렬로 완벽하게 표현하며, 예측과 업데이트 단계를 5개의 핵심 행렬 방정식으로 수행한다.33

- **예측 단계**

  1. **상태 예측:**
     $$
     \hat{\mathbf{x}}_{k|k-1} = \mathbf{F}_k \hat{\mathbf{x}}_{k-1|k-1} + \mathbf{B}_k \mathbf{u}_k
     $$
     **오차 공분산 예측:**
     $$
     \mathbf{P}_{k|k-1} = \mathbf{F}_k \mathbf{P}_{k-1|k-1} \mathbf{F}_k^T + \mathbf{Q}_k
     $$
     업데이트 단계

  3. 칼만 이득 계산:
     $$
     \mathbf{K}_k = \mathbf{P}_{k|k-1} \mathbf{H}_k^T (\mathbf{H}_k \mathbf{P}_{k|k-1} \mathbf{H}_k^T + \mathbf{R}_k)^{-1}
     $$

  4. 상태 업데이트:
     $$
     \hat{\mathbf{x}}_{k|k} = \hat{\mathbf{x}}_{k-1|k-1} + \mathbf{K}_k (\mathbf{z}_k - \mathbf{H}_k \hat{\mathbf{x}}_{k|k-1})
     $$

  5. 오차 공분산 업데이트:
     $$
     \mathbf{P}_{k|k} = (\mathbf{I} - \mathbf{K}_k \mathbf{H}_k) \mathbf{P}_{k|k-1}
     $$


실제 세계의 많은 시스템, 특히 관성 항법 시스템은 비선형 동역학 모델을 가진다. 확장 칼만 필터(EKF)는 이러한 비선형 시스템에 칼만 필터를 적용하기 위한 가장 기본적인 접근법이다. EKF의 핵심 아이디어는 비선형 함수를 현재 상태 추정치 주변에서 테일러 급수 전개를 통해 1차 항까지만 근사하여 선형화(Linearization)하는 것이다.37 이 선형화 과정에서 상태 전이 행렬($\mathbf{F}$)과 측정 행렬($\mathbf{H}$)의 역할을 편미분 행렬인 자코비안 행렬(Jacobian Matrix)이 대신하게 된다.37 하지만 EKF는 비선형성이 강한 시스템에서는 선형화 오차가 커져 필터의 성능이 저하되거나 심한 경우 발산(Diverge)할 수 있다는 근본적인 한계를 가진다.37


무향 칼만 필터(UKF)는 EKF의 선형화 오차 문제를 해결하기 위한 효과적인 대안이다. UKF는 비선형 함수 자체를 근사하는 대신, 상태의 확률 분포를 직접 근사하는 방식을 취한다. 이는 '무향 변환(Unscented Transform)'이라는 기법을 통해 이루어진다. 무향 변환은 현재 상태의 평균과 공분산 정보를 정확히 대표하는 소수의 샘플 포인트, 즉 '시그마 포인트(Sigma Points)'를 결정론적으로 생성한다. 그 다음, 이 시그마 포인트들을 비선형 함수에 직접 통과시킨 후, 변환된 포인트들의 가중 평균과 가중 공분산을 계산하여 결과 분포를 근사한다. 이 방식은 자코비안 행렬을 계산할 필요가 없으며, EKF가 무시하는 고차항의 정보를 통계적으로 반영하므로 일반적으로 EKF보다 더 높은 정확도를 제공한다.40


파티클 필터는 비선형, 비가우시안(Non-Gaussian) 시스템에 모두 적용할 수 있는 가장 일반적이고 강력한 베이즈 필터이다. 파티클 필터는 확률 분포를 다수의 무작위 샘플, 즉 '파티클(Particle)'의 집합으로 근사한다. 각 파티클은 시스템의 가능한 상태 가설을 나타내며, 가중치(Weight)를 가진다. 예측 단계에서는 모든 파티클을 시스템 모델에 따라 전파시키고, 업데이트 단계에서는 새로운 측정값에 대한 각 파티클의 우도(Likelihood)를 계산하여 가중치를 갱신한다. 이 과정에서 가중치가 낮은 파티클은 사라지고 높은 파티클이 복제되는 재표본추출(Resampling) 과정을 통해 중요한 상태 영역에 파티클을 집중시킨다.42 파티클 필터는 다중 모드(Multi-modal) 분포와 같이 복잡한 형태의 확률 분포도 효과적으로 표현할 수 있지만, 상태 공간의 차원이 커지거나 필요한 파티클 수가 많아질수록 계산량이 급격히 증가하는 단점이 있다.42

다음 표는 비선형 시스템에 사용되는 주요 칼만 필터 계열 알고리즘의 특성을 비교한다. 이 표는 특정 응용 분야에 가장 적합한 필터를 선택하는 데 유용한 기준을 제공한다. 문제의 비선형성 정도, 실시간 처리 요구사항, 노이즈의 통계적 특성 등을 고려하여 각 필터의 장단점을 파악할 수 있다. 이는 단순히 알고리즘을 나열하는 것을 넘어, 문제 해결 전략의 스펙트럼을 보여줌으로써 각 접근 방식의 철학적 차이를 명확히 이해하는 데 도움을 준다.

| 특징            | 확장 칼만 필터 (EKF)                        | 무향 칼만 필터 (UKF)                            | 파티클 필터 (PF)                     |
| --------------- | ------------------------------------------- | ----------------------------------------------- | ------------------------------------ |
| **기본 원리**   | 비선형 모델을 선형화 (테일러 급수 1차 근사) | 무향 변환을 통한 확률 분포 근사 (시그마 포인트) | 순차적 몬테카를로 방법 (가중 파티클) |
| **적용 대상**   | 미분 가능한 비선형 시스템                   | 비선형 시스템                                   | 모든 비선형/비가우시안 시스템        |
| **정확도**      | 비선형성이 약할 때 양호                     | EKF보다 우수 (고차항 정보 반영)                 | 파티클 수에 비례하여 이론상 최적     |
| **계산 복잡도** | 낮음 (자코비안 계산 필요)                   | 중간 (시그마 포인트 전파)                       | 높음 (파티클 수에 비례)              |
| **가정**        | 가우시안 노이즈                             | 가우시안 노이즈 (근사적으로)                    | 임의의 노이즈 분포 가능              |
| **구현 난이도** | 중간 (자코비안 해석적 유도 필요)            | 비교적 용이 (자코비안 불필요)                   | 높음 (재표본추출 등)                 |



카메라는 저렴하고 가벼우며 풍부한 환경 정보를 제공하기 때문에 GNSS 음영 환경에서 강력한 항법 센서로 활용된다. 비전 기반 항법은 크게 Visual Odometry(VO)와 SLAM(Simultaneous Localization and Mapping)으로 나뉜다.


Visual Odometry는 연속적인 카메라 이미지 간의 시각적 변화를 분석하여 카메라 자체의 상대적인 움직임(6-DOF: 3차원 위치 및 3차원 자세)을 추정하는 기술이다.43 이는 바퀴의 회전 수를 세어 이동 거리를 추정하는 휠 오도메트리와 원리적으로 유사하지만, 바퀴 대신 시각 정보를 이용한다.46

VO 알고리즘은 일반적으로 다음과 같은 파이프라인을 따른다.48

1. **특징점 추출 및 기술(Feature Extraction & Description):** 이미지에서 모서리나 질감의 변화가 뚜렷한 지점과 같이 추적하기 용이한 특징점(Keypoint)을 검출한다. SIFT, SURF, ORB와 같은 알고리즘이 널리 사용된다. 검출된 각 특징점 주변의 픽셀 정보를 고유한 벡터 형태의 기술자(Descriptor)로 변환하여 다른 이미지의 특징점과 비교할 수 있도록 한다.45
2. **특징점 매칭(Feature Matching):** 연속된 두 이미지 프레임 간에 동일한 물리적 지점에 해당하는 특징점 쌍을 기술자를 비교하여 찾는다.
3. **움직임 추정(Motion Estimation):** 매칭된 특징점 쌍들의 기하학적 관계, 즉 에피폴라 기하(Epipolar Geometry)를 이용하여 두 프레임 간의 상대적인 회전 행렬($R$)과 변위 벡터($t$)를 계산한다. 이 과정에서 잘못된 매칭(Outlier)을 제거하기 위해 RANSAC(Random Sample Consensus)과 같은 강인한 추정 기법이 필수적으로 사용된다.43

VO는 단기적으로는 비교적 정확한 움직임 추정이 가능하지만, 각 단계에서 발생하는 미세한 오차가 누적되어 시간이 지남에 따라 실제 궤적으로부터 멀어지는 드리프트(Drift) 문제가 발생한다. 이는 외부의 절대적인 기준 없이 상대적인 변화만을 측정하기 때문에 발생하는 본질적인 한계이다.53


SLAM은 로봇이나 이동체가 사전에 정보가 없는 미지의 환경을 탐색하면서, 주변 환경의 지도를 실시간으로 작성함과 동시에 그 지도상에서 자신의 위치를 추정하는 기술이다.55 이는 "위치를 정확히 알려면 지도가 필요하고, 정확한 지도를 만들려면 자신의 위치를 알아야 한다"는 상호 의존적인 문제를 동시에 해결하는 복잡하고 중요한 과제이다.57

SLAM 시스템은 일반적으로 다음과 같은 파이프라인으로 구성된다.50

1. **프론트엔드(Front-end):** VO와 유사한 역할을 수행한다. 연속적인 센서 데이터를 입력받아 단기적인 움직임을 추정하고, 지도 작성을 위한 환경의 특징(랜드마크)을 추출한다.
2. **백엔드(Back-end):** 프론트엔드에서 누적된 모든 관측 정보(이동체의 궤적, 랜드마크 위치)를 이용하여 전역적으로 가장 일관성 있는 궤적과 지도를 추정하는 최적화(Optimization)를 수행한다. 주로 그래프 최적화(Graph Optimization) 기법이 사용된다.
3. **루프 폐쇄(Loop Closure):** SLAM의 핵심 요소로, 이동체가 과거에 방문했던 장소를 다시 인식하는 것을 의미한다. 루프가 감지되면, 현재 위치와 과거 위치가 동일하다는 강력한 제약 조건이 생성된다. 이 제약 조건을 백엔드 최적화에 활용하여 그동안 누적된 드리프트 오차를 전체 궤적에 걸쳐 보정한다. 이 과정을 통해 전역적으로 일관된(Globally Consistent) 지도와 궤적을 얻을 수 있다.46

VO가 단순히 "얼마나 움직였는가"에 대한 답을 찾는 기술이라면, SLAM은 "얼마나 움직였으며, 현재 어디에 있고, 주변 환경은 어떻게 생겼는가"라는 더 포괄적인 질문에 답하는 기술이다. 루프 폐쇄를 통한 드리프트 보정 능력 유무가 VO와 SLAM을 구분하는 가장 핵심적인 차이점이다.50


능동 센서는 스스로 신호(레이저, 전파 등)를 방출하고, 반사된 신호를 수신하여 환경 정보를 획득하는 센서를 말한다. 조명 조건에 독립적이라는 장점 때문에 비전 센서의 한계를 보완하는 데 널리 사용된다.


LiDAR(Light Detection and Ranging)는 레이저 펄스를 주변 환경에 방출하고, 물체에 반사되어 돌아오는 빛의 비행시간(Time-of-Flight, ToF)을 측정하여 거리를 계산하는 센서이다. 이를 360도 회전하며 반복하여 주변 환경을 정밀한 3차원 점군(Point Cloud) 데이터로 표현할 수 있다.62

LiDAR 기반 항법은 연속적으로 획득되는 점군 데이터를 정합(Align)하는 스캔 매칭(Scan Matching) 과정을 통해 이동체의 움직임을 추정한다. 대표적인 스캔 매칭 알고리즘으로는 점과 점 사이의 거리를 최소화하는 ICP(Iterative Closest Point)와 점군을 확률 분포로 모델링하여 정합하는 NDT(Normal Distributions Transform)가 있다.64

LiDAR는 카메라에 비해 조명 변화나 텍스처가 부족한 환경에 매우 강인하며, 센티미터 수준의 정밀한 3D 지도를 생성할 수 있다는 장점이 있다. 그러나 센서 가격이 비싸고, 비, 눈, 안개, 먼지와 같은 악천후 조건에서는 레이저가 산란되거나 흡수되어 성능이 크게 저하될 수 있다.56


레이더(Radar, Radio Detection and Ranging)는 전자기파(Radio Wave)를 이용하여 물체까지의 거리와 상대 속도(도플러 효과)를 측정하는 센서이다.68 특히 차량용 레이더에서는 주파수가 시간에 따라 선형적으로 변하는 처프 신호를 사용하는 FMCW(Frequency-Modulated Continuous-Wave) 방식이 널리 사용된다.69

레이더의 가장 큰 장점은 LiDAR나 카메라와 달리 파장이 긴 전자기파를 사용하기 때문에 비, 눈, 안개, 먼지 등 악천후와 저조도 환경에 거의 영향을 받지 않는다는 점이다.68 이러한 특성 덕분에 전천후 자율주행을 위한 핵심 센서로 주목받고 있다. 하지만, 파장이 긴 만큼 해상도가 낮아 정밀한 형태 인식이 어렵고, 데이터가 희소(Sparse)하며, 전파가 여러 번 반사되어 돌아오는 다중 경로 반사로 인한 노이즈(Clutter)가 심하다는 단점이 있다.69

LiDAR와 레이더는 단순히 서로를 대체하는 경쟁 관계가 아니다. 오히려, 두 센서는 각자의 장단점이 명확하여 상호 보완적인 특성을 가진다. LiDAR는 맑은 날씨에 정밀한 기하학적 정보를 제공하는 데 탁월하고, 레이더는 악천후 상황에서 강인한 감지 능력과 속도 정보를 제공한다. 예를 들어, 긴 터널과 같이 기하학적 특징이 반복되어 LiDAR의 성능이 저하되는(degeneracy) 환경에서, 레이더가 제공하는 속도 정보는 INS의 드리프트를 억제하는 데 중요한 역할을 할 수 있다.73 따라서, 이 두 센서의 측정치를 상황에 따라 가중치를 조절하며 융합하는 동적 센서 융합 전략은 자율주행 시스템의 전반적인 강인성과 신뢰성을 극대화하는 핵심 기술이다. '강결합 LiDAR-레이더-관성 항법(Tightly-coupled LiDAR-Radar-Inertial fusion)'과 같은 최신 연구들은 이러한 상보성을 극대화하려는 노력의 일환이다.73


INS의 드리프트 오차를 보정하기 위해 비전, LiDAR, 레이더 외에도 다양한 보조 센서들이 활용된다.


INS 오차 중 특히 방위각(Yaw) 오차는 수평 방향의 움직임을 계산하는 데 직접적인 영향을 미치지만, 가속도계로는 보정할 수 없어 드리프트가 가장 심각한 문제 중 하나이다. 지자기 센서(Magnetometer)는 지구 자기장을 측정하여 절대적인 자기 북쪽 방향에 대한 정보를 제공함으로써, 자이로스코프의 Yaw 축 드리프트를 효과적으로 보정할 수 있다.74

AHRS(Attitude and Heading Reference System)는 가속도계(Pitch/Roll 자세 정보), 자이로스코프(단기적인 자세 변화), 지자기 센서(장기적인 Yaw 안정성)의 측정값을 칼만 필터와 같은 알고리즘으로 융합하여 빠르고 안정적인 3축 자세 정보를 제공하는 시스템이다.75 하지만 지자기 센서는 주변의 강자성체(Hard Iron)나 전류가 흐르는 전선(Soft Iron)에 의해 발생하는 자기장 왜곡에 매우 취약하다. 따라서 차량, 드론, 실내와 같이 자기장 교란이 심한 환경에서 사용하기 위해서는 정밀한 사전 보정(Calibration) 과정이 필수적이다.75


INS의 고도 채널은 중력 가속도 보상의 부정확성 등으로 인해 오차가 시간에 따라 기하급수적으로 발산하는 특성이 있다.78 기압계(Barometer)는 고도가 높아질수록 대기압이 낮아지는 원리를 이용하여 절대 고도에 대한 정보를 제공한다. 이 기압 고도 정보를 INS의 고도 추정치에 대한 보정값으로 활용함으로써, 수직 채널 오차의 발산을 효과적으로 억제할 수 있다.80 그러나 대기압은 날씨 변화에 따라 계속 변동하므로, 정밀한 고도 유지를 위해서는 외부 기상 정보(예: 공항의 QNH 값)를 이용한 주기적인 보정이 필요하다.80




실내는 GNSS 신호가 완전히 차단되는 대표적인 음영 환경이다. 그러나 스마트 빌딩, 대규모 쇼핑몰, 공항, 병원, 물류 창고 등에서 자산 추적, 길안내, 위치 기반 마케팅과 같은 다양한 위치 기반 서비스(LBS)에 대한 수요가 폭발적으로 증가하고 있다.2 이를 위해 GNSS를 대체할 다양한 실내 측위 기술(IPS)이 개발되고 있다.

- **주요 기술** 7:
  - **Wi-Fi 핑거프린팅(Fingerprinting):** 실내 공간을 격자로 나누고 각 지점에서 수신되는 여러 Wi-Fi AP(Access Point)의 신호 세기(RSSI) 패턴을 미리 측정하여 데이터베이스, 즉 '전파 지도(Radio Map)'를 구축한다. 이후 사용자의 현재 위치에서 측정된 RSSI 패턴을 이 지도와 비교하여 가장 유사한 패턴을 가진 지점을 현재 위치로 추정하는 방식이다. 기존에 설치된 Wi-Fi 인프라를 그대로 활용할 수 있다는 큰 장점이 있지만, AP의 위치가 바뀌거나 가구 배치 변경 등 환경이 변하면 전파 지도를 다시 구축해야 하는 유지보수의 어려움이 있다.83
  - **BLE (Bluetooth Low Energy) 비콘:** 저전력 블루투스 신호를 주기적으로 발신하는 작은 장치인 비콘을 실내 곳곳에 설치하고, 스마트폰 등 수신기가 감지한 비콘 신호의 RSSI를 기반으로 거리를 추정하여 위치를 계산한다. 설치가 용이하고 비콘의 배터리 수명이 길다는 장점이 있다.7
  - **UWB (Ultra-Wideband):** 매우 넓은 주파수 대역에 걸쳐 나노초(ns) 단위의 매우 짧은 펄스 신호를 사용하여 신호의 도달 시간(Time of Arrival, ToA) 또는 도달 시간차(Time Difference of Arrival, TDoA)를 정밀하게 측정한다. 이를 통해 다중 경로 오차에 강인하며, 수십 센티미터 수준의 매우 높은 측위 정확도를 제공할 수 있다. 다만, 전용 인프라 구축 비용이 상대적으로 높다.2
  - **기타 기술:** 이 외에도 건물 구조물에 의해 미세하게 변하는 지구 자기장 패턴을 이용하는 **지자기 측위**, 스마트폰 카메라로 주변 환경을 인식하는 **컴퓨터 비전 기반 측위**, 그리고 사용자의 걸음 수와 방향을 추정하는 **PDR(Pedestrian Dead Reckoning)** 등 다양한 기술들이 단독으로 또는 서로 융합되어 사용되고 있다.7


수중은 전자기파가 급격히 감쇠되어 GNSS 신호가 전혀 도달하지 못하는 대표적인 극한 환경이다.87 따라서 무인 잠수정(AUV), 어뢰, 다이버 등의 수중 항법을 위해서는 특수한 기술이 요구된다.

- **주요 기술:**
  - **관성 항법 (INS):** 외부 정보 없이 독립적으로 항해가 가능한 거의 유일한 수단으로, 특히 은밀한 기동이 필수적인 군용 잠수정이나 장거리 탐사 AUV의 핵심 항법 장치로 사용된다. 하지만 장시간 운용 시 드리프트 오차 보정이 반드시 필요하다.89
  - **음향 측위 시스템(Acoustic Positioning):** 수중에서 잘 전파되는 음파를 이용해 위치를 결정한다. 해저에 음향 송수신기(Transponder)를 미리 설치하고 거리를 측정하는 LBL(Long Baseline) 방식은 정밀도가 높지만 인프라 설치가 필요하다. 반면, 선박이나 모선에 설치된 장비와 수중 이동체 간의 거리와 방위를 측정하는 USBL(Ultra-Short Baseline) 방식은 운용이 편리하여 널리 사용된다.87
  - **지구물리 항법(Geophysical Navigation):** 사전에 정밀하게 제작된 해저 지형, 지자기장, 또는 중력장 이상(anomaly) 지도와 이동체가 실시간으로 측정한 센서 값을 비교(Map Matching)하여 위치 오차를 보정하는 기술이다. 이는 INS의 드리프트를 효과적으로 억제할 수 있는 강력한 보조 항법 수단이다.88


도심 협곡은 고층 빌딩으로 인해 위성 신호가 차단되거나 반사되어 다중 경로 오차가 심각하게 발생하는 대표적인 GNSS 열화 환경이다.6

- **대응 기술:**
  - **다중 경로 신호 완화:** 수신된 위성 신호의 품질 지표(예: 신호 대 잡음비 C/N0)나 의사거리 측정치의 일관성 등을 분석하여, 직접 경로 신호(LOS)와 다중 경로 오염 신호(NLOS)를 식별한다. 이후 NLOS 신호를 위치 계산에서 제외하거나 가중치를 낮추어 오차의 영향을 최소화한다. 최근에는 신호의 시간-주파수 특성을 분석하여 딥러닝 모델로 NLOS 신호를 분류하는 연구가 활발히 진행되고 있다.9
  - **강결합 GNSS/INS:** 가시 위성의 수가 4개 미만으로 줄어들어 단독으로 위치를 계산할 수 없는 상황에서도, 수신 가능한 소수의 위성 정보(의사거리, 도플러 등)를 INS와 강결합 방식으로 융합하여 INS의 오차를 지속적으로 보정한다.
  - **3D 지도 및 비전 센서 활용:** 정밀 3D 빌딩 모델을 이용하여 위성 신호가 차단되거나 반사될 경로를 미리 예측하여 다중 경로 오차를 모델링하거나, 차량에 장착된 카메라로 주변 건물을 인식하여 SLAM 기술을 통해 위치를 보정하는 등 보조 센서를 적극적으로 활용하는 접근법이 연구되고 있다.


GNSS와 INS를 융합하여 상호 단점을 보완하는 통합 항법 시스템은 구현 방식에 따라 약결합과 강결합 아키텍처로 나뉜다.


약결합 방식은 GNSS 수신기와 INS가 각각 독립적으로 자신의 항법 해(위치, 속도)를 계산한 후, 이 두 개의 독립적인 해를 외부의 칼만 필터에서 융합하는 구조이다.91 즉, 칼만 필터의 측정값으로 GNSS가 출력한 위치 및 속도 정보가 사용된다.

이 방식은 각 시스템이 모듈화되어 있어 구현이 비교적 간단하고 계산 부하가 적다는 장점이 있다. 하지만, GNSS 수신기가 가시 위성 수 부족(4개 미만) 등의 이유로 자체적으로 항법 해를 산출하지 못하면 칼만 필터에 제공할 측정값이 없어지게 된다. 이 경우, 통합 시스템은 GNSS 정보가 완전히 단절(outage)된 것으로 간주하고 INS 단독 항법 모드로 전환되어 오차가 빠르게 누적되기 시작한다.93


강결합 방식은 GNSS 수신기가 계산한 최종적인 위치/속도 해를 사용하는 대신, 각 위성으로부터 수신한 원시 측정치(Raw Measurements)인 의사거리(Pseudorange)와 의사거리 변화율(Doppler)을 칼만 필터의 측정값으로 직접 사용하는 구조이다.91 칼만 필터는 INS로부터 예측된 위치를 바탕으로 각 위성까지의 예상 의사거리를 계산하고, 이를 실제 측정된 의사거리와 비교하여 INS의 오차를 보정한다.

이 방식은 가시 위성이 1~3개에 불과하여 GNSS 단독으로 위치를 결정할 수 없는 상황에서도, 수신 가능한 위성들의 원시 측정치 정보를 최대한 활용하여 INS 오차를 보정할 수 있다는 강력한 장점을 가진다. 따라서 도심 협곡과 같이 위성 신호 수신이 간헐적으로 이루어지는 환경에서 약결합 방식보다 훨씬 강인하고 연속적인 고정밀 항법 성능을 제공한다.92 다만, GNSS 원시 측정치를 직접 다루어야 하므로 구현이 복잡하고 계산량이 많다는 단점이 있다.

다음 표는 약결합과 강결합 방식의 핵심적인 차이점을 요약한다. 이 표는 '어떤 수준의 정보를 융합하는가'라는 근본적인 차이점을 명확히 보여주며, 이는 시스템 설계의 핵심적인 결정 사항이다. 특히, GNSS 음영/열화 환경에서 왜 강결합 방식이 우월한지에 대한 이유를 직관적으로 설명하여, 사용자가 자신의 응용 분야 요구사항에 맞는 아키텍처를 선택하는 데 직접적인 도움을 준다.

| 구분                | 약결합 (Loosely Coupled)                   | 강결합 (Tightly Coupled)                        |
| ------------------- | ------------------------------------------ | ----------------------------------------------- |
| **융합 정보**       | GNSS 위치/속도 해(Solution)                | GNSS 원시 측정치 (의사거리, 도플러 등)          |
| **아키텍처**        | GNSS 수신기와 INS가 독립적, 최종 해를 융합 | GNSS 원시 측정치를 INS 예측치와 직접 융합       |
| **구현 복잡도**     | 낮음                                       | 높음                                            |
| **계산량**          | 적음                                       | 많음                                            |
| **성능 (위성 > 4)** | 강결합과 유사                              | 약결합과 유사                                   |
| **성능 (위성 < 4)** | GNSS 단절, INS 단독 항법으로 전환          | 가용 위성 정보로 INS 오차 부분적 보정 가능      |
| **강인성**          | 낮음 (GNSS 해에 의존적)                    | 높음 (개별 위성 정보 활용, 불량 위성 식별 용이) |
| **주요 적용 분야**  | 일반적인 환경, 저가형 시스템               | 도심 협곡, 재밍/스푸핑 환경, 고정밀 시스템      |


GNSS 음영 환경에서의 항법 기술은 인공지능과 차세대 센서 기술의 발전에 힘입어 새로운 패러다임으로 진화하고 있다.


전통적인 항법 알고리즘은 정교한 물리 모델과 기하학적 제약 조건에 기반하지만, 복잡하고 동적인 실제 환경의 모든 변수를 모델링하는 데 한계가 있다. 딥러닝은 방대한 데이터로부터 시스템의 복잡한 특성을 직접 학습함으로써 이러한 한계를 극복할 잠재력을 보여주고 있다.

- **End-to-End Visual-Inertial Odometry (VIO):** 초기 연구들은 CNN(Convolutional Neural Network)과 RNN(Recurrent Neural Network)을 결합하여, 카메라 이미지와 IMU 데이터 시퀀스를 입력받아 이동체의 궤적을 직접 출력하는 End-to-End 학습 방식을 제안했다. VINet과 같은 모델은 수동적인 센서 교정이나 동기화 과정 없이도 데이터로부터 직접 융합 방법을 학습할 수 있는 가능성을 제시했다.96
- **하이브리드 접근법:** 최근에는 순수 End-to-End 방식의 낮은 해석 가능성과 일반화 성능을 보완하기 위해, 딥러닝과 전통적인 최적화 기법을 결합하는 하이브리드 방식이 주목받고 있다. 예를 들어, 딥러닝 모델이 특징점 추출, 데이터 연관 분석, IMU 바이어스 예측 등과 같이 모델링이 어려운 부분을 담당하고, 전체적인 궤적 추정은 기하학적 제약에 기반한 번들 조정(Bundle Adjustment)과 같은 전통적인 최적화 기법으로 수행하여 두 방식의 장점을 모두 취하는 연구가 활발히 진행 중이다.99


현재의 관성 센서는 아무리 정밀해도 미세한 드리프트를 피할 수 없다. 양자 관성 센서는 이러한 한계를 근본적으로 극복할 수 있는 차세대 기술로 기대를 모으고 있다.

- **원리:** 원자 간섭계(Atom Interferometry)와 같은 양자 현상을 이용하여 가속도와 각속도를 측정한다. 원자의 양자 상태는 매우 안정적이고 외부 환경에 둔감하므로, 이론적으로 기존의 MEMS나 광섬유 자이로(FOG) 센서보다 훨씬 낮은 드리프트와 높은 장기 안정성을 달성할 수 있다.101
- **전망:** 아직은 실험실 수준의 연구가 주를 이루고 있지만, 기술이 성숙하면 외부의 도움 없이도 장시간 동안 정밀한 항법이 가능한 '진정한 의미의 관성 항법' 시대를 열 수 있을 것으로 기대된다. 상용화를 위해서는 센서의 소형화, 비용 절감, 그리고 실제 운용 환경에서의 강인성 확보가 핵심 과제로 남아있다.102


GNSS 음영 환경이라 할지라도, 우리 주변에는 Wi-Fi, 셀룰러(LTE/5G), 디지털 방송, FM 라디오 등 수많은 종류의 인공적인 무선 신호가 존재한다. 기회 신호(SoP) 항법은 이러한 기존의 통신 및 방송 인프라를 항법에 '기회적으로' 활용하는 기술이다. 이는 별도의 항법 인프라를 구축할 필요 없이 주변에 존재하는 신호를 이용하여 위치 정보를 얻을 수 있다는 점에서 확장성과 경제성이 매우 높다.82


본 자습서는 현대 사회의 필수 기술인 항법 시스템이 GNSS 신호에 대한 의존성에서 벗어나, 신호가 차단되거나 왜곡되는 극한 환경에서도 신뢰성을 유지하기 위한 다양한 기술적 접근법을 포괄적으로 탐구했다.

관성 항법 시스템(INS)은 외부 정보 없이 자율적으로 항법 정보를 제공하는 유일한 수단이지만, 시간에 따라 오차가 누적되는 드리프트라는 본질적인 한계를 가진다. 이 한계를 극복하기 위해, 칼만 필터를 필두로 한 확률적 추정 이론은 다양한 센서 정보를 수학적으로 최적 융합하는 강력한 프레임워크를 제공한다.

GNSS를 대체하는 보조 항법 기술은 카메라, LiDAR, 레이더와 같은 외부 환경 인식 센서부터 지자기, 기압계와 같은 보조 센서에 이르기까지 매우 다양하다. 특히, Visual SLAM과 LiDAR/Radar Odometry 기술은 정밀한 지도 생성과 움직임 추정을 통해 GNSS 음영 환경에서 항법의 연속성을 보장하는 핵심 기술로 자리 잡았다. 더 나아가, 실내, 수중, 도심 협곡과 같은 특수 환경의 고유한 문제들을 해결하기 위한 맞춤형 솔루션들이 개발되고 있으며, 약결합과 강결합과 같은 시스템 통합 아키텍처는 환경의 특성과 시스템 요구사항에 따라 최적의 성능을 발휘하도록 설계된다.

미래의 항법 기술은 딥러닝을 통한 데이터 기반 학습, 양자 센서를 통한 물리적 한계 돌파, 그리고 주변의 모든 신호를 활용하는 기회 신호 항법으로 나아가고 있다. 이는 GNSS 음영 환경에서의 항법이 더 이상 단일 기술의 문제가 아니라, 다양한 센서와 정교한 알고리즘, 그리고 인공지능이 유기적으로 융합된 복합 지능 시스템의 영역임을 명확히 보여준다. 결국, 미래 자율 항법 시스템의 성공은 얼마나 정밀한 센서를 사용하는가뿐만 아니라, 다양한 정보를 어떻게 지능적으로 융합하고, 실시간으로 변화하는 환경에 얼마나 강인하게 대처할 수 있는가에 달려 있을 것이다.


1. 인공지능 기반 실내 측위 기술 동향 및 전망 - Korea Science, 8월 18, 2025에 액세스, https://koreascience.kr/article/JAKO202013661038026.pdf
2. IPS란 무엇인가요? - 실내 위치 확인 시스템 소개 - MOKOSmart, 8월 18, 2025에 액세스, https://www.mokosmart.com/ko/introduction-to-indoor-positioning-system/
3. 5.1 GNSS-Challenged Environments - VectorNav Technologies, 8월 18, 2025에 액세스, https://www.vectornav.com/resources/inertial-navigation-primer/alternative-navigation/altnav-gnsschallenged
4. What are the limitations of GNSS? - Inertial Labs, 8월 18, 2025에 액세스, https://inertiallabs.com/what-are-the-limitations-of-gnss/
5. 스마트폰을 활용한 GNSS 정밀 측위의 현재와 미래 - 세종대학교, 8월 18, 2025에 액세스, [https://www.sejong.ac.kr/kor/research/research-focus.do?mode=view&articleNo=717845&title=%EC%86%90+%EC%95%88%EC%9D%98+%EC%A0%95%EB%B0%80+%EC%9C%84%EC%B9%98+%EA%B8%B0%EC%88%A0+%3A+%EC%8A%A4%EB%A7%88%ED%8A%B8%ED%8F%B0%EC%9D%84+%ED%99%9C%EC%9A%A9%ED%95%9C+GNSS+%EC%A0%95%EB%B0%80+%EC%B8%A1%EC%9C%84%EC%9D%98+%ED%98%84%EC%9E%AC%EC%99%80+%EB%AF%B8%EB%9E%98](https://www.sejong.ac.kr/kor/research/research-focus.do?mode=view&articleNo=717845&title=손+안의+정밀+위치+기술+:+스마트폰을+활용한+GNSS+정밀+측위의+현재와+미래)
6. [기고] GNSS를 보완하는 셀룰러 및 와이파이 기반 위치확인 기술은, 8월 18, 2025에 액세스, https://elec4.co.kr/article/articleView.asp?idx=32232
7. 실내 공간에서 GPS를 통한 위치 측위의 한계와 대안 - 프리그로우, 8월 18, 2025에 액세스, https://grow-space.io/gps/
8. Navigating GPS-Denied Environments: Solutions and Challenges - GNSS Jamming, 8월 18, 2025에 액세스, https://www.gnssjamming.com/post/gps-denied-environments
9. Time-Frequency 특성과 인공신경망을 이용한 도심 지역에서의 GNSS 다중경로오차 완화기법 > 2024proc - 사단법인 항법시스템학회, 8월 18, 2025에 액세스, https://ipnt.or.kr/2024proc/165
10. GNSS Denied? | NovAtel, 8월 18, 2025에 액세스, https://novatel.com/tech-talk/velocity-magazine/velocity-2014/gnss-denied
11. GNSS가 무엇이고 어떻게 작동하는지 알아보세요. - SINSMART, 8월 18, 2025에 액세스, https://www.sinsmarts.com/ko/blog/discover-what-gnss-is-and-how-it-works/
12. GNSS-Denied Navigation Kit | UAV Navigation, 8월 18, 2025에 액세스, https://www.uavnavigation.com/products/navigation-systems/gnss-denied-navigation-kit
13. GNSS 및 오류 원인, 8월 18, 2025에 액세스, https://www.sbg-systems.com/ko/technology/gnss-and-their-error-sources/
14. 관성 항법 시스템 소개 - 뉴스, 8월 18, 2025에 액세스, http://ko.liocrebifx.com/news/inertial-navigation-system-introduction-68260514.html
15. 관성 내비게이션 시스템이란 무엇인가요? - OXTS, 8월 18, 2025에 액세스, https://www.oxts.com/ko/what-is-an-inertial-navigation-system/
16. Inertial navigation system - Wikipedia, 8월 18, 2025에 액세스, https://en.wikipedia.org/wiki/Inertial_navigation_system
17. [IMU] IMU의 개념 및 활용법, 8월 18, 2025에 액세스, https://winterbloooom.github.io/robotics/perception/2021/11/17/imu.html
18. 관성항법장치(INS: Inertial Navigation System) - 클라우드의 데일리 리포트 - 티스토리, 8월 18, 2025에 액세스, https://clouds-daily.tistory.com/219
19. IMU 는 어디에 사용되는 물건일까요?, 8월 18, 2025에 액세스, https://dulidungsil.tistory.com/entry/IMU-Inertial-Measurement-Unit
20. 마이크로 관정 센서의 기술현황과 전망, 8월 18, 2025에 액세스, https://koreascience.kr/article/JAKO200011920693097.pdf
21. 프로젝트 일지 5. IMU 센서, 8월 18, 2025에 액세스, https://k-min-algorithm.tistory.com/55
22. 자유도 10의 관성 센서, 어떻게 구현할까? - 바이라인네트워크, 8월 18, 2025에 액세스, https://byline.network/2021/08/12-148/
23. 관성항법시스템(INS)에 대하여 설명하시오. - 클라우드의 데일리 리포트, 8월 18, 2025에 액세스, https://clouds-daily.tistory.com/78
24. www.joet.org, 8월 18, 2025에 액세스, [https://www.joet.org/m/journal/view.php?number=25#:~:text=%EA%B4%80%EC%84%B1%ED%95%AD%EB%B2%95%20%EC%8B%9C%EC%8A%A4%ED%85%9C%EC%9D%80%20%EC%9E%A0%EC%88%98%ED%95%A8,%ED%95%9C%20%EA%B1%B0%EB%A6%AC%EB%A5%BC%20%EA%B5%AC%ED%95%98%EB%8A%94%20%EA%B2%83%EC%9D%B4%EB%8B%A4.](https://www.joet.org/m/journal/view.php?number=25#:~:text=관성항법 시스템은 잠수함,한 거리를 구하는 것이다.)
25. Chapter 1 - Strapdown Associates, 8월 18, 2025에 액세스, [https://www.strapdownassociates.com/Chapter%201.pdf](https://www.strapdownassociates.com/Chapter 1.pdf)
26. Implementation of ZUPT on RPA Navigation System for GNSS Denied Ground Test, 8월 18, 2025에 액세스, http://www.jpnt.org/wp-content/uploads/2024/03/JPNT-0902-10.pdf
27. www.jksqm.org, 8월 18, 2025에 액세스, https://www.jksqm.org/upload/pdf/jksqm-51-4-619.pdf
28. Journal of Ocean Engineering and Technology, 8월 18, 2025에 액세스, https://www.joet.org/m/journal/view.php?number=294
29. 관성항법장치 - 위키백과, 우리 모두의 백과사전, 8월 18, 2025에 액세스, [https://ko.wikipedia.org/wiki/%EA%B4%80%EC%84%B1%ED%95%AD%EB%B2%95%EC%9E%A5%EC%B9%98](https://ko.wikipedia.org/wiki/관성항법장치)
30. Strapdown Inertial Navigation Systems: An Algorithm for Attitude and Navigation Computations, - DTIC, 8월 18, 2025에 액세스, https://apps.dtic.mil/sti/tr/pdf/ADA112256.pdf
31. [INS] 관성항법 방정식 (Kinematics of Inertial Navigation), 8월 18, 2025에 액세스, https://pasus.tistory.com/323
32. [INS] 관성항법시스템 오차 방정식 (INS Error Equations), 8월 18, 2025에 액세스, https://pasus.tistory.com/324
33. Tutorial: The Kalman Filter - MIT, 8월 18, 2025에 액세스, [https://web.mit.edu/kirtley/kirtley/binlustuff/literature/control/Kalman%20filter.pdf](https://web.mit.edu/kirtley/kirtley/binlustuff/literature/control/Kalman filter.pdf)
34. Understanding and Applying Kalman Filtering, 8월 18, 2025에 액세스, http://biorobotics.ri.cmu.edu/papers/sbp_papers/integrated3/kleeman_kalman_basics.pdf
35. Kalman and Extended Kalman Filters: Concept, Derivation and Properties - CiteSeerX, 8월 18, 2025에 액세스, https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=b689ad56d935ceb053e24bf4de26f1466b911263
36. Kalman Filter Equations - Kalman Filter Tutorial, 8월 18, 2025에 액세스, https://www.kalmanfilter.net/multiSummary.html
37. Extended Kalman filter - Wikipedia, 8월 18, 2025에 액세스, https://en.wikipedia.org/wiki/Extended_Kalman_filter
38. 2.9 Nonlinear Kalman Filter - VectorNav Technologies, 8월 18, 2025에 액세스, https://www.vectornav.com/resources/inertial-navigation-primer/math-fundamentals/math-nonlinearkalman
39. Extended Kalman Filters - MATLAB & Simulink - MathWorks, 8월 18, 2025에 액세스, https://www.mathworks.com/help/fusion/ug/extended-kalman-filters.html
40. Unscented Kalman Filter for Non-Linear Systems - Number Analytics, 8월 18, 2025에 액세스, https://www.numberanalytics.com/blog/unscented-kalman-filter-for-non-linear-systems
41. Introduction to Unscented Kalman Filter 1 Introdution, 8월 18, 2025에 액세스, https://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/AV0809/qi.pdf
42. Particle Filters: A Hands-On Tutorial - PMC, 8월 18, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC7826670/
43. Visual Odometry 요약 - Donghoon Note - 티스토리, 8월 18, 2025에 액세스, https://dhpark1212.tistory.com/entry/Visual-Odometry
44. 8. Visual Odometry 기술과 Instant Tracker | by MAXST_Tech | Medium, 8월 18, 2025에 액세스, [https://medium.com/@maxst_tech/8-visual-odometry-%EA%B8%B0%EC%88%A0%EA%B3%BC-instant-tracker-65929e72d355](https://medium.com/@maxst_tech/8-visual-odometry-기술과-instant-tracker-65929e72d355)
45. Visual Odometry: Part II Matching, Robustness, Optimization, and Applications, 8월 18, 2025에 액세스, https://www.zora.uzh.ch/71030/1/Fraundorfer_Scaramuzza_Visual_odometry.pdf
46. 모바일 로봇공학에서의 시각적 오도메트리와 시각적 SLAM 개요 - MRDVS, 8월 18, 2025에 액세스, https://mrdvs.com/ko/visual-odometry-and-visual-slam-in-mobile-robotics/
47. Visual odometry - Wikipedia, 8월 18, 2025에 액세스, https://en.wikipedia.org/wiki/Visual_odometry
48. Visual Odometry Tutorial | John Lambert, 8월 18, 2025에 액세스, https://johnwlambert.github.io/vo/
49. Implementing Visual SLAM - A Step-by-Step Guide - Rama Prashanth, 8월 18, 2025에 액세스, https://ramaprashanth.com/blog/visual-slam-walkthrough
50. The 6 Components of a Visual SLAM Algorithm - Think Autonomous, 8월 18, 2025에 액세스, https://www.thinkautonomous.ai/blog/visual-slam/
51. 20190224 Visual Odometry 1 (part 1) - YouTube, 8월 18, 2025에 액세스, https://www.youtube.com/watch?v=gC-bnvrleno
52. Introduction to Visual Odometry - Medium, 8월 18, 2025에 액세스, https://medium.com/@3502.stkabirdin/introduction-to-visual-odometry-47bcab3aa213
53. SLAM(동시적 위치추정 및 지도작성)이란 – MATLAB 및 Simulink - 매스웍스, 8월 18, 2025에 액세스, https://kr.mathworks.com/discovery/slam.html
54. SLAM의 이해와 구현 PART 01 요약, 8월 18, 2025에 액세스, https://roytravel.tistory.com/405
55. SLAM 기술: 공간 지능의 핵심 동력 - 브런치, 8월 18, 2025에 액세스, https://brunch.co.kr/@donghyungshin/148
56. 스스로 만드는 위치 지도 SLAM 기술의 원리 - LG디스커버리랩, 8월 18, 2025에 액세스, https://www.lgdlab.or.kr/contents/6
57. 3D LiDAR SLAM - 개념, 원리, 구조 - 개발in개발자 - 티스토리, 8월 18, 2025에 액세스, https://hjdevelop.tistory.com/15
58. Introduction to SLAM (Simultaneous Localization and Mapping) | Ouster, 8월 18, 2025에 액세스, https://ouster.com/insights/blog/introduction-to-slam-simultaneous-localization-and-mapping
59. Simultaneous Localization and Mapping - GeeksforGeeks, 8월 18, 2025에 액세스, https://www.geeksforgeeks.org/machine-learning/simultaneous-localization-and-mapping/
60. SLAM과 시각 오도메트리 접근 방식 비교 : r/computervision - Reddit, 8월 18, 2025에 액세스, https://www.reddit.com/r/computervision/comments/s863pj/slam_vs_visual_odometry_approaches/?tl=ko
61. From Pixels to Precision: A Survey of Monocular Visual Odometry in Digital Twin Applications - PubMed Central, 8월 18, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC10891866/
62. SLAM과 3D LiDAR로 AMR 기술을 해결하는 방법 - 유진로봇, 8월 18, 2025에 액세스, https://yujinrobot.com/resource/blog/how-slam-and-3d-lidar-solve-for-amr-technology
63. 자율주행 로봇 기술 완전 정리: SLAM, LiDAR, 딥러닝 - hwangdolsun 님의 블로그, 8월 18, 2025에 액세스, https://hwangdolsun.com/12
64. 3D LiDAR SLAM – Scan Matching Explained - Ignitarium, 8월 18, 2025에 액세스, https://ignitarium.com/3d-lidar-slam-scan-matching-explained/
65. Lidar Slam vs. Visual Slam : 더 똑똑한 매핑 결정의 주요 차이점, 8월 18, 2025에 액세스, https://korean.geosunlidar.com/news/lidar-slam-vs-visual-slam-an-in-depth-comparison-190902.html
66. 이동로봇을 위한 SLAM 기술, 8월 18, 2025에 액세스, [http://icros.org/Newsletter/202201/5.%EB%A1%9C%EB%B4%87%EA%B8%B0%EC%88%A0%EB%A6%AC%EB%B7%B0.pdf](http://icros.org/Newsletter/202201/5.로봇기술리뷰.pdf)
67. SLAM이란 무엇일까요? : 티쓰리솔루션 블로그, 8월 18, 2025에 액세스, https://www.t3solution.co.kr/24/?bmode=view&idx=95337001
68. LiDAR vs. RADAR : A comprehensive comparison - YellowScan, 8월 18, 2025에 액세스, https://www.yellowscan.com/knowledge/lidar-vs-radar/
69. Radar-Only Odometry and Mapping for Autonomous Vehicles, 8월 18, 2025에 액세스, https://www.ipb.uni-bonn.de/wp-content/papercite-data/pdf/casado-herraez2024icra.pdf
70. Frequency-Modulated Continuous-Wave Radar (FMCW Radar) - Radartutorial.eu, 8월 18, 2025에 액세스, [https://www.radartutorial.eu/02.basics/Frequency%20Modulated%20Continuous%20Wave%20Radar.en.html](https://www.radartutorial.eu/02.basics/Frequency Modulated Continuous Wave Radar.en.html)
71. Radar odometry based on Fuzzy-NDT scan registration - DiVA portal, 8월 18, 2025에 액세스, https://www.diva-portal.org/smash/get/diva2:1596455/FULLTEXT01.pdf
72. A LOW-COST VISUAL RADAR-BASED ODOMETRY FRAMEWORK WITH MMWAVE RADAR AND MONOCULAR CAMERA - Semantic Scholar, 8월 18, 2025에 액세스, https://pdfs.semanticscholar.org/23ae/a6d99b48a2b7e9f29ed5721092ea8df2824d.pdf
73. Degradation Resilient LiDAR-Radar-Inertial Odometry - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2403.05332v1
74. 관성 항법 시스템에서 자력계의 중요한 역할 - 지식, 8월 18, 2025에 액세스, http://ko.liocrebifx.com/info/crucial-role-of-magnetometers-88621038.html
75. Learn how an Attitude and Heading Reference System (AHRS) works, 8월 18, 2025에 액세스, https://www.vectornav.com/resources/inertial-navigation-primer/theory-of-operation/theory-ahrs
76. Drift Reduction in Pedestrian Navigation System by Exploiting Motion Constraints and Magnetic Field - MDPI, 8월 18, 2025에 액세스, https://www.mdpi.com/1424-8220/16/9/1455
77. Correcting Current-Induced Magnetometer Errors on UAVs: An Online Model-Based Approach - Kamran Mohseni - University of Florida, 8월 18, 2025에 액세스, https://enstrophy.mae.ufl.edu/publications/MyPapers/2019_Silic_IEEEJSens.pdf
78. KR101107219B1 - 비행체의 항법 방법 및 이를 이용한 관성항법장치 필터 및 항법 시스템, 8월 18, 2025에 액세스, https://patents.google.com/patent/KR101107219B1/ko
79. 기압고도계 오차 추정 - Korea Science, 8월 18, 2025에 액세스, https://koreascience.kr/article/JAKO200800557082683.pdf
80. 비행기의 고도는 어떻게 측정할까? 고도계의 원리 - 캡틴닭의 비행LOG, 8월 18, 2025에 액세스, [https://captaindark.tistory.com/entry/%EB%B9%84%ED%96%89%EA%B8%B0%EC%9D%98-%EA%B3%A0%EB%8F%84%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%B8%A1%EC%A0%95%ED%95%A0%EA%B9%8C-%EA%B3%A0%EB%8F%84%EA%B3%84%EC%9D%98-%EC%9B%90%EB%A6%AC](https://captaindark.tistory.com/entry/비행기의-고도는-어떻게-측정할까-고도계의-원리)
81. 고도는 어떻게 재는 걸까? - YouTube, 8월 18, 2025에 액세스, https://m.youtube.com/watch?v=MbCKy3zgYfk
82. 실내 위치 기반 서비스 기술개발 및 표준화 동향 - ETRI 지식공유플랫폼, 8월 18, 2025에 액세스, https://ksp.etri.re.kr/ksp/article/file/13640.pdf
83. RTLS와 커져가는 실내 측위 기술의 중요성 - IPIN LABS | 아이핀랩스, 8월 18, 2025에 액세스, https://home.ipinlabs.com/ko/feeds/blog/rtls-and-the-growing-importance-of-indoor-positioning-technology
84. Wi-Fi Fingerprint 기반 실내 측위 기법 성능 예측에 관한 연구 - MMLAB - 서울대학교, 8월 18, 2025에 액세스, https://mmlab.snu.ac.kr/wp-content/uploads/2025/05/2015kics_djkim.pdf
85. Location fingerprinting - what is it, and should you choose it as your IPS technology? - Pointr, 8월 18, 2025에 액세스, https://www.pointr.tech/blog/location-fingerprinting-what-is-it-should-you-choose-it
86. Static vs. Dynamic Databases for Indoor Localization based on Wi-Fi Fingerprinting: A Discussion from a Data Perspective - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2402.12756v1
87. 수중 GPS의 기능과 한계 탐구: 혁신과 실제 적용 - MesidaTech, 8월 18, 2025에 액세스, https://mesidatech.com/ko/exploring-the-capabilities-and-limitations-of-underwater-gps-innovations-and-practical-applications/
88. 지구 물리정보를 이용한 무인잠수정의 복합 항법 기술, 8월 18, 2025에 액세스, https://jkros.org/xml/26017/26017.pdf
89. 수중자율주행용 항법기술 개발현황 및 분석(2편), 8월 18, 2025에 액세스, http://ko.liocrebifx.com/info/development-status-and-analysis-of-navigation-60509221.html
90. 수중운동체의 제어장치 - THE SSEN LIG, 8월 18, 2025에 액세스, https://www.thessen-lig.com:10010/svcShowWebZinDetail.do?contentsSeq=299
91. Inertial Navigation System (INS) - SBG Support Center, 8월 18, 2025에 액세스, https://support.sbg-systems.com/sc/kb/latest/integrated-motion-navigation-sensors/inertial-navigation-system-ins
92. Loosely vs Tightly Coupled INS & GNSS | Point One Nav, 8월 18, 2025에 액세스, https://pointonenav.com/news/loose-vs-tight-coupling-gnss/
93. Learn how a GNSS-Aided Inertial Navigation System (GNSS/INS ..., 8월 18, 2025에 액세스, https://www.vectornav.com/resources/inertial-navigation-primer/theory-of-operation/theory-gpsins
94. Loose and Tight GNSS/INS Integrations: Comparison of ..., 8월 18, 2025에 액세스, https://pmc.ncbi.nlm.nih.gov/articles/PMC5335985/
95. GNSS/INS Integration Methods - University of Calgary, 8월 18, 2025에 액세스, [https://www.ucalgary.ca/engo_webdocs/other/Angrisano_PhD%20Thesis_ENG2.pdf](https://www.ucalgary.ca/engo_webdocs/other/Angrisano_PhD Thesis_ENG2.pdf)
96. VINet: Visual-Inertial Odometry as a Sequence-to-Sequence Learning Problem | Request PDF - ResearchGate, 8월 18, 2025에 액세스, https://www.researchgate.net/publication/313096356_VINet_Visual-Inertial_Odometry_as_a_Sequence-to-Sequence_Learning_Problem
97. Efficient Deep Visual and Inertial Odometry with Adaptive Visual Modality Selection, 8월 18, 2025에 액세스, https://www.ecva.net/papers/eccv_2022/papers_ECCV/papers/136980229.pdf
98. VINet: Visual-Inertial Odometry as a Sequence-to-Sequence Learning Problem - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/pdf/1701.08376
99. Deep Learning for Visual Localization and Mapping: A Survey - PolyU Scholars Hub, 8월 18, 2025에 액세스, https://research.polyu.edu.hk/en/publications/deep-learning-for-visual-localization-and-mapping-a-survey
100. Adaptive VIO: Deep Visual-Inertial Odometry with Online Continual Learning - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2405.16754v1
101. Quantum Inertial Sensors for Gravimetry and Inertial Navigation in the Field | Technical Program - IEEE/ION PLANS 2025, 8월 18, 2025에 액세스, https://www.ion.org/plans/abstracts.cfm?paperID=15240
102. Navigation in the future: Review of quantum sensing in navigation - SciEngine, 8월 18, 2025에 액세스, https://www.sciengine.com/doi/10.1007/s11433-025-2699-1
103. Quantum Artificial Intelligence for Secure Autonomous Vehicle Navigation: An Architectural Proposal - arXiv, 8월 18, 2025에 액세스, https://arxiv.org/html/2506.16000v1
104. Quantum vs. Classical Complementary PNT - MITRE Corporation, 8월 18, 2025에 액세스, https://www.mitre.org/sites/default/files/2024-06/PR-23-0577-Quantum-vs-Classical-Complementary-PNT.pdf

